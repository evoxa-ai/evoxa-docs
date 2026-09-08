TASK-001-08 — PostgreSQL Infrastructure

IS: IS-001 — Identity Domain Foundation
ESP: ESP-0001/14 — Identity Implementation Requirements
Dependencias: TASK-001-07 — Unit of Work / Transaction Boundary
Prioridad: 🔴 Critical
Estado: Ready for Implementation

1. Objetivo

Implementar la primera infraestructura concreta de persistencia de Identity utilizando PostgreSQL como fuente transaccional de verdad, manteniendo completamente separada la infraestructura de los contratos del Domain.

Esta tarea conecta:

Identity Application
        │
        ▼
   Unit of Work
        │
        ▼
 PostgreSQL Infrastructure
        │
        ▼
    PostgreSQL

Pero todavía no implementaremos los modelos completos de User, Organization, Membership, etc.

La finalidad de esta tarea es dejar PostgreSQL correctamente preparado para las siguientes tareas.

2. Alcance

Esta tarea incluye:

configuración de PostgreSQL;
configuración externa mediante settings;
connection pool;
database session;
Unit of Work concreta;
transaction lifecycle;
manejo inicial de errores PostgreSQL;
health/readiness check;
integración con FastAPI;
lifecycle de conexiones;
aislamiento de credenciales;
pruebas de infraestructura;
preparación para migrations.

No incluye todavía:

User table
Organization table
Membership table
Role table
Permission table

Eso pertenece a las tareas de Persistence/Data Model posteriores.

3. Fuente de verdad

La arquitectura queda:

PostgreSQL
    │
    ├── Identity state
    ├── transactional data
    ├── security state
    └── Outbox

PostgreSQL será la fuente transaccional de verdad.

Redis no reemplaza PostgreSQL.

Event Platform tampoco reemplaza PostgreSQL.

4. Arquitectura
┌──────────────────────────────────────────┐
│                FastAPI                   │
└────────────────────┬─────────────────────┘
                     │
                     ▼
┌──────────────────────────────────────────┐
│              Application                 │
│                                          │
│              UnitOfWork                  │
└────────────────────┬─────────────────────┘
                     │
                     ▼
┌──────────────────────────────────────────┐
│             Infrastructure               │
│                                          │
│ PostgreSQLUnitOfWork                     │
│ PostgreSQLSession                        │
│ PostgreSQLRepository implementations     │
│ PostgreSQL error mapping                 │
└────────────────────┬─────────────────────┘
                     │
                     ▼
┌──────────────────────────────────────────┐
│                PostgreSQL                │
└──────────────────────────────────────────┘
5. Tecnología

La especificación actual establece:

Database: PostgreSQL

La elección concreta del driver/ORM debe mantenerse como decisión de implementación.

La estructura deberá permitir utilizar:

SQLAlchemy
asyncpg
psycopg

sin contaminar Domain.

Para la primera implementación podemos utilizar SQLAlchemy async + PostgreSQL driver, pero esto queda como implementación técnica, no como cambio arquitectónico.

6. Dependencias

Conceptualmente:

fastapi
pydantic-settings
sqlalchemy
asyncpg
alembic
pytest
pytest-asyncio

El conjunto exacto de versiones deberá quedar fijado en el proyecto mediante el mecanismo de dependencias elegido.

7. Configuración

La configuración de PostgreSQL no debe estar escrita directamente en el código.

Por ejemplo:

DATABASE_HOST
DATABASE_PORT
DATABASE_NAME
DATABASE_USER
DATABASE_PASSWORD
DATABASE_POOL_SIZE
DATABASE_MAX_OVERFLOW
DATABASE_POOL_TIMEOUT
DATABASE_POOL_RECYCLE
DATABASE_ECHO

Nunca:

DATABASE_PASSWORD = "password123"
8. Settings

Podemos extender:

apps/api/app/core/config.py

con una configuración equivalente a:

from pydantic_settings import BaseSettings, SettingsConfigDict


class Settings(BaseSettings):

    database_host: str
    database_port: int = 5432
    database_name: str
    database_user: str
    database_password: str

    database_pool_size: int = 10
    database_max_overflow: int = 20
    database_pool_timeout: int = 30
    database_pool_recycle: int = 1800
    database_echo: bool = False

    model_config = SettingsConfigDict(
        env_file=".env",
        extra="ignore",
    )

Los valores exactos de producción no deben quedar hardcodeados.

9. Database URL

Infrastructure puede construir internamente:

postgresql+asyncpg://user:password@host:port/database

pero esa URL no debe aparecer en logs.

Especialmente no:

postgresql://user:password@...

en:

logs;
traces;
exceptions;
métricas;
audit;
events.
10. Engine

La infraestructura podrá crear un engine:

from sqlalchemy.ext.asyncio import (
    AsyncEngine,
    create_async_engine,
)


def create_database_engine(settings) -> AsyncEngine:

    url = (
        "postgresql+asyncpg://"
        f"{settings.database_user}:"
        f"{settings.database_password}@"
        f"{settings.database_host}:"
        f"{settings.database_port}/"
        f"{settings.database_name}"
    )

    return create_async_engine(
        url,
        pool_size=settings.database_pool_size,
        max_overflow=settings.database_max_overflow,
        pool_timeout=settings.database_pool_timeout,
        pool_recycle=settings.database_pool_recycle,
        echo=settings.database_echo,
    )

Esto es Infrastructure.

Nunca debe ser importado por:

identity/.../domain/
11. Connection Pool

El engine administrará un pool.

Application
     │
     ▼
UnitOfWork
     │
     ▼
Connection Pool
 ┌───┼───┬───┐
 │   │   │   │
 ▼   ▼   ▼   ▼
 DB  DB  DB  DB

Esto evita abrir/cerrar conexiones físicas constantemente.

12. Pool Configuration

Los valores:

pool_size
max_overflow
pool_timeout
pool_recycle

serán configurables.

No debemos asumir que:

pool_size = 100

es automáticamente mejor.

La capacidad deberá dimensionarse posteriormente de acuerdo con:

número de réplicas;
conexiones permitidas por PostgreSQL;
carga;
workers;
Kubernetes;
operaciones concurrentes.
13. Session Factory

Crearemos:

apps/api/app/core/infrastructure/database/
├── engine.py
├── session.py
└── health.py

La sesión será creada por Infrastructure.

Conceptualmente:

from sqlalchemy.ext.asyncio import (
    AsyncSession,
    async_sessionmaker,
)


def create_session_factory(engine):

    return async_sessionmaker(
        bind=engine,
        class_=AsyncSession,
        expire_on_commit=False,
    )
14. PostgreSQL Unit of Work

Ahora TASK-001-07 tendrá su implementación concreta.

UnitOfWork
     ▲
     │ implements
     │
PostgreSQLUnitOfWork

Ejemplo conceptual:

class PostgreSQLUnitOfWork(UnitOfWork):

    def __init__(self, session_factory):
        self.session_factory = session_factory
        self.session = None

    async def begin(self):
        self.session = self.session_factory()
        await self.session.begin()

    async def commit(self):
        await self.session.commit()

    async def rollback(self):
        await self.session.rollback()

La implementación final deberá además garantizar correctamente el lifecycle de la sesión.

15. Context Manager

La utilización será:

async with uow:

    ...

Conceptualmente:

ENTER
  │
  ▼
BEGIN
  │
  ▼
operations
  │
  ├── success → COMMIT
  │
  └── error   → ROLLBACK
  │
  ▼
CLOSE
16. Cierre de sesión

Una de las reglas importantes:

Una sesión PostgreSQL no debe quedar abierta después de finalizar la Unit of Work.

Por lo tanto:

BEGIN
 ↓
operations
 ↓
COMMIT / ROLLBACK
 ↓
CLOSE

La conexión vuelve al pool.

17. Rollback seguro

Ante cualquier excepción:

try:
    ...
    await session.commit()

except Exception:
    await session.rollback()
    raise

finally:
    await session.close()

La implementación concreta deberá evitar también errores secundarios durante el cleanup.

18. Transaction Isolation

No vamos a fijar todavía una estrategia global agresiva de isolation level.

La configuración inicial deberá utilizar el comportamiento estándar soportado por PostgreSQL y permitir que operaciones específicas puedan requerir controles adicionales.

Esto evita introducir prematuramente:

SERIALIZABLE everywhere

que podría tener impacto innecesario en throughput.

19. Concurrency

La infraestructura debe soportar el mecanismo definido en TASK-001-06:

optimistic concurrency

Posteriormente, una operación podrá utilizar una condición conceptual:

UPDATE aggregate
SET version = version + 1
WHERE id = ?
  AND version = expected_version

Si no se actualiza ninguna fila:

ConcurrencyConflict

Pero esta lógica pertenece a los Repository implementations concretos, no al engine global.

20. Error Mapping

Infrastructure no debe exponer excepciones internas de PostgreSQL directamente a API.

Por ejemplo:

PostgreSQL exception
       ↓
Infrastructure mapping
       ↓
Application/domain exception
       ↓
API error

Podemos crear:

database/exceptions.py

con excepciones como:

class DatabaseError(Exception):
    pass


class DatabaseUnavailable(DatabaseError):
    pass


class DatabaseIntegrityError(DatabaseError):
    pass

Y posteriormente mapear conflictos concretos.

21. Integrity Errors

Un error PostgreSQL como:

UNIQUE violation

no debería devolver al usuario:

psycopg.errors.UniqueViolation
DETAIL: Key (...)

Debe transformarse posteriormente a un error de aplicación estable.

Por ejemplo:

{
  "error": {
    "code": "RESOURCE_CONFLICT",
    "message": "The resource already exists.",
    "correlation_id": "..."
  }
}

La API seguirá el contrato establecido en ESP-0001/10.

22. Database Availability

Necesitamos un health check.

Agregar:

GET /api/v1/health/ready

ya creado conceptualmente en TASK-001-01.

Ahora podrá comprobar:

API
 +
PostgreSQL
23. Liveness vs Readiness
Liveness
GET /health/live

responde si el proceso está vivo.

No debería depender de PostgreSQL.

Readiness
GET /health/ready

comprueba dependencias necesarias para recibir tráfico.

Por ejemplo:

PostgreSQL → OK

Resultado:

{
  "status": "ok"
}

Si PostgreSQL no está disponible:

HTTP 503
24. Health Query

La comprobación puede ser mínima:

SELECT 1;

No debemos ejecutar:

SELECT * FROM users

para comprobar disponibilidad.

La health check debe ser:

rápida;
barata;
segura;
sin efectos secundarios.
25. Readiness y Kubernetes

Posteriormente Kubernetes podrá utilizar:

readinessProbe

para decidir:

Traffic → API

Si PostgreSQL está temporalmente caído:

API instance
    │
    └── Not Ready

en lugar de aceptar tráfico que inevitablemente fallará.

26. Startup

Al arrancar:

Application startup
      │
      ├── load config
      ├── create DB engine
      ├── create session factory
      └── initialize infrastructure

No debemos ejecutar automáticamente:

create all tables

mediante:

Base.metadata.create_all(...)

en producción.

La creación/evolución del schema será responsabilidad de migrations.

27. Migrations

Prepararemos:

Alembic

como mecanismo de migrations.

La dirección será:

Migration
   ↓
PostgreSQL Schema

No:

Application startup
   ↓
automatically alter database

Esto será particularmente importante cuando lleguemos a producción y múltiples replicas ejecuten simultáneamente.

28. Migration Ownership

Las migrations deberán vivir fuera del Domain:

apps/api/
├── app/
└── migrations/
    ├── versions/
    └── env.py

El Domain no importará Alembic.

29. Database Lifecycle

El proceso de aplicación deberá soportar:

START
 │
 ▼
Create Engine
 │
 ▼
Create Pool
 │
 ▼
Application Ready
 │
 ▼
Requests
 │
 ▼
Shutdown
 │
 ▼
Dispose Engine
 │
 ▼
Exit

En shutdown:

await engine.dispose()

debe liberar correctamente los recursos.

30. Dependency Injection

FastAPI podrá recibir la UoW mediante dependency injection.

Conceptualmente:

async def get_uow():

    uow = PostgreSQLUnitOfWork(
        session_factory
    )

    yield uow

Y el Use Case:

API
 ↓
Dependency
 ↓
PostgreSQLUnitOfWork
 ↓
Application

La lógica de negocio no construirá directamente el engine.

31. Configuración global

Debe existir una única fuente de configuración:

Settings
   │
   ├── Database
   ├── Redis
   ├── Security
   ├── Event Platform
   └── Application

No:

os.getenv(...)

disperso por todo el proyecto.

32. Secret Management

En desarrollo podremos utilizar .env:

DATABASE_PASSWORD=...

pero:

.env

debe quedar fuera del repositorio.

Producción:

Application
   ↓
Secret Manager
   ↓
DATABASE_PASSWORD

Nunca:

Git
Docker Image
Source Code
Logs
33. Database Credentials

El usuario de PostgreSQL utilizado por la aplicación deberá tener solamente los permisos necesarios.

No utilizar:

postgres superuser

como identidad normal de la aplicación.

La separación futura será:

Migration Identity
       ≠
Application Identity

Esto permite aplicar least privilege.

34. SSL / TLS

La configuración deberá soportar conexiones PostgreSQL protegidas mediante TLS.

Conceptualmente:

Application
    │
    │ TLS
    ▼
PostgreSQL

El requisito exacto de:

sslmode
CA certificate
certificate verification

se fijará según el entorno de despliegue.

No debemos desactivar la validación TLS en producción como solución de troubleshooting.

35. Logging

Los logs de infraestructura podrán indicar:

database connection initialized
database connection failed
database health check failed
transaction committed
transaction rolled back

Pero nunca:

password
connection URL
credentials
tokens

Además, evitar logging excesivo de SQL en producción.

36. Observabilidad

Se deberán preparar métricas como:

db_connections_active
db_connections_idle
db_transaction_duration
db_transaction_commit_total
db_transaction_rollback_total
db_errors_total
db_health_check
db_pool_exhaustion

El detalle final dependerá del sistema de observabilidad implementado en tareas posteriores.

37. Correlation ID

Si una transacción falla:

HTTP Request
 correlation_id
      │
      ▼
Application
      │
      ▼
UnitOfWork
      │
      ▼
Database Error

la correlación debe mantenerse.

Pero nunca debemos añadir:

password
token
SQL credentials

al contexto de observabilidad.

38. Testing
Unit Tests

Probar:

Settings
Database URL construction
UnitOfWork lifecycle
Exception mapping

sin necesidad de PostgreSQL real cuando no sea necesario.

Integration Tests

Con PostgreSQL real:

BEGIN
INSERT
COMMIT

y:

BEGIN
INSERT
ROLLBACK
39. Test de Atomicidad

Una prueba crítica:

BEGIN
    business state
    outbox state
    ERROR
ROLLBACK

Después:

business state → absent
outbox state    → absent

Este test conecta directamente con TASK-001-05 y TASK-001-07.

40. Test de Database Failure

Simular:

PostgreSQL unavailable

y comprobar:

readiness = false
HTTP = 503

sin que la aplicación:

crash loop

de forma incontrolada.

41. Test de Connection Pool

Debemos probar:

normal load
pool exhaustion
connection timeout
recovery

Esto será especialmente importante antes de producción.

42. Test de Shutdown

Verificar:

Application shutdown
       ↓
engine.dispose()
       ↓
pool closed

sin conexiones huérfanas.

43. Test de Domain Isolation

Debe continuar pasando:

Domain
  ✗ SQLAlchemy
  ✗ asyncpg
  ✗ PostgreSQL

mientras:

Infrastructure
  ✓ SQLAlchemy
  ✓ asyncpg
  ✓ PostgreSQL
44. Estructura resultante

Después de esta tarea:

apps/api/
│
├── app/
│   ├── main.py
│   │
│   ├── core/
│   │   ├── config.py
│   │   │
│   │   └── infrastructure/
│   │       └── database/
│   │           ├── engine.py
│   │           ├── session.py
│   │           ├── health.py
│   │           └── exceptions.py
│   │
│   └── domains/
│       └── identity/
│           └── shared/
│               ├── application/
│               │   └── ports.py
│               │
│               └── infrastructure/
│                   └── persistence/
│                       └── postgres/
│                           └── unit_of_work.py
│
├── migrations/
│   ├── env.py
│   └── versions/
│
└── tests/
    ├── unit/
    └── integration/
45. Flujo completo

Ya podemos visualizar:

                   HTTP Request
                         │
                         ▼
                    FastAPI API
                         │
                         ▼
                   Application
                         │
                         ▼
                   UnitOfWork
                         │
                         ▼
              PostgreSQL Infrastructure
                         │
                  ┌──────┴──────┐
                  ▼             ▼
             Connection      Transaction
                Pool              │
                                  ▼
                             PostgreSQL
                                  │
                    ┌─────────────┴─────────────┐
                    ▼                           ▼
             Business State                 Outbox
                    │                           │
                    └─────────────┬─────────────┘
                                  ▼
                               COMMIT
                                  │
                                  ▼
                            Outbox Worker
                                  │
                                  ▼
                           Event Platform
46. Qué NO hacemos en TASK-001-08

Para mantener el lifecycle limpio, no debemos adelantarnos.

No implementaremos todavía:

❌ User model
❌ Organization model
❌ Membership model
❌ Role model
❌ Permission model
❌ MFA tables
❌ RefreshToken tables
❌ Authorization tables
❌ SecurityPolicy tables

Tampoco:

❌ Redis implementation
❌ Event broker implementation
❌ Authentication
❌ JWT
❌ MFA
❌ Authorization Runtime

Cada uno tendrá su propia tarea.

47. Definition of Done
[ ] PostgreSQL configuration
[ ] Externalized DB settings
[ ] Database engine
[ ] Connection pool
[ ] Async session factory
[ ] PostgreSQL UnitOfWork
[ ] Transaction lifecycle
[ ] Commit
[ ] Rollback
[ ] Session cleanup
[ ] Database exception mapping
[ ] PostgreSQL readiness check
[ ] Engine shutdown/disposal
[ ] Migration framework initialized
[ ] TLS configuration capability
[ ] Least-privilege DB configuration documented
[ ] No credentials in source
[ ] No credentials in logs
[ ] Unit tests
[ ] Integration tests
[ ] Architecture tests
48. Criterios de aceptación
ID	Criterio
AC-01	PostgreSQL es accesible únicamente desde Infrastructure
AC-02	Las credenciales son externas al código
AC-03	Existe connection pool
AC-04	Existe session factory
AC-05	PostgreSQLUnitOfWork implementa UnitOfWork
AC-06	Commit y rollback funcionan correctamente
AC-07	Las sesiones se cierran correctamente
AC-08	PostgreSQL failures no exponen detalles internos a API
AC-09	/health/ready detecta PostgreSQL
AC-10	/health/live no depende de PostgreSQL
AC-11	El schema se gestiona mediante migrations
AC-12	No se crean tablas automáticamente en startup
AC-13	Business state y Outbox pueden compartir transacción
AC-14	ConcurrencyConflict puede propagarse correctamente
AC-15	No existen credenciales en logs
AC-16	Domain permanece independiente de PostgreSQL
AC-17	La infraestructura soporta shutdown limpio
AC-18	Existe cobertura de tests de infraestructura
49. Trazabilidad
IS-001
 │
 └── TASK-001-08
       │
       ├── ESP-0001/11
       │     Identity Persistence
       │
       ├── ESP-0001/14
       │     Implementation Requirements
       │
       ├── ESP-0001/15
       │     Deployment & Operations
       │
       ├── ADR-006
       │     PostgreSQL Source of Truth
       │
       ├── ADR-012
       │     PostgreSQL Schema
       │
       ├── ADR-016
       │     Concurrency
       │
       ├── ADR-017
       │     Migrations
       │
       ├── ADR-029
       │     Disaster Recovery
       │
       └── ADR-031
             High Availability
50. Resultado

Con esta tarea, EVOXA pasa de tener solamente contratos abstractos a disponer de una primera infraestructura real de persistencia:

TASK-001-01  API Runtime             ✅
TASK-001-02  Identity Domain         ✅
TASK-001-03  Layer Architecture      ✅
TASK-001-04  Shared Kernel           ✅
TASK-001-05  Domain Events           ✅
TASK-001-06  Repository Contracts    ✅
TASK-001-07  Unit of Work            ✅
TASK-001-08  PostgreSQL              ✅

La separación queda:

DOMAIN
  │
  │ contracts
  ▼
APPLICATION
  │
  │ UnitOfWork
  ▼
INFRASTRUCTURE
  │
  │ PostgreSQL
  ▼
DATABASE

Y, muy importante, todavía no estamos creando tablas de negocio. Primero estamos construyendo correctamente los cimientos sobre los cuales después podrán vivir User, Organization, Membership, Roles, Sessions, MFA, Authorization, etc.

Siguiente: TASK-001-09 — Redis Infrastructure, donde estableceremos Redis como infraestructura de cache, estado temporal, rate limiting y otros usos no autoritativos, manteniendo PostgreSQL como fuente de verdad y evitando que una caída de Redis pueda convertirse en un bypass de seguridad.
