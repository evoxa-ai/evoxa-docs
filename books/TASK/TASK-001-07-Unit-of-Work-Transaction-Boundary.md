TASK-001-07 — Unit of Work / Transaction Boundary

IS: IS-001 — Identity Domain Foundation
ESP: ESP-0001/14 — Identity Implementation Requirements
Dependencias: TASK-001-05, TASK-001-06
Prioridad: 🔴 Critical
Estado: Ready for Implementation

1. Objetivo

Definir e implementar la Unit of Work (UoW) de Identity como la frontera que garantiza que los cambios de negocio y sus eventos/outbox sean tratados dentro de una misma transacción.

La regla fundamental será:

Business State
      +
Domain Events
      +
Outbox Records
      │
      ▼
   COMMIT

o, ante cualquier error:

Business State
      +
Domain Events
      +
Outbox Records
      │
      ▼
  ROLLBACK

Nunca debemos terminar con:

Usuario guardado
      +
Evento no guardado

ni:

Evento publicado
      +
Transacción de negocio rollback
2. Principio arquitectónico

La Unit of Work pertenece conceptualmente a Application, mientras que su implementación concreta pertenece a Infrastructure.

Application
│
├── Use Case
│
└── UnitOfWork Contract
          │
          ▼
Infrastructure
│
└── PostgreSQLUnitOfWork

El Domain no conoce:

PostgreSQL
SQLAlchemy
asyncpg
transactions
connection pools
3. Frontera transaccional

Una operación de negocio completa deberá seguir:

API
 │
 ▼
Application Use Case
 │
 ▼
UnitOfWork BEGIN
 │
 ├── Repository.load()
 │
 ├── Aggregate operation
 │
 ├── Repository.save()
 │
 ├── collect Domain Events
 │
 ├── OutboxWriter.append()
 │
 ▼
UnitOfWork COMMIT
 │
 ▼
Response

Si algo falla:

BEGIN
 │
 ├── state change
 ├── event
 ├── outbox
 │
 └── ERROR
       │
       ▼
    ROLLBACK
4. Regla crítica: publicar ≠ persistir

La Unit of Work no debe publicar directamente el evento al Event Platform.

No:

save()
  ↓
publish()
  ↓
commit()

porque podríamos terminar con:

Event Platform:
    UserCreated

PostgreSQL:
    ROLLBACK

Eso generaría un evento de una operación que realmente no ocurrió.

La secuencia correcta es:

BEGIN
  │
  ├── persist User
  │
  ├── persist OutboxEvent
  │
COMMIT
  │
  ▼
Outbox Publisher
  │
  ▼
Event Platform

Esto implementa el patrón Transactional Outbox establecido previamente.

5. Contrato de Unit of Work

Ya existe:

identity/shared/application/ports.py

Lo vamos a convertir en el contrato oficial.

from abc import ABC, abstractmethod


class UnitOfWork(ABC):

    async def __aenter__(self):
        await self.begin()
        return self

    async def __aexit__(
        self,
        exc_type,
        exc_value,
        traceback,
    ):
        if exc_type is not None:
            await self.rollback()
            return False

        await self.commit()

    @abstractmethod
    async def begin(self) -> None:
        raise NotImplementedError

    @abstractmethod
    async def commit(self) -> None:
        raise NotImplementedError

    @abstractmethod
    async def rollback(self) -> None:
        raise NotImplementedError
6. ¿Por qué begin()?

Aunque algunas implementaciones puedan iniciar una transacción automáticamente al ejecutar la primera operación, conceptualmente queremos que la frontera sea explícita:

BEGIN
...
COMMIT

Esto facilita:

testing;
observabilidad;
debugging;
control de concurrencia;
futuras implementaciones;
comprensión del lifecycle.
7. Uso desde Application

Un Use Case podrá tener esta estructura:

class CreateUserUseCase:

    def __init__(
        self,
        uow: UnitOfWork,
    ):
        self.uow = uow

    async def execute(self, command):

        async with self.uow:

            user = User.create(
                ...
            )

            await self.uow.users.save(user)

            events = user.pull_domain_events()

            for event in events:
                await self.uow.outbox.append(event)

Al salir del contexto:

async with self.uow
        │
        ▼
     COMMIT

Si aparece una excepción:

exception
   │
   ▼
ROLLBACK
8. Problema que debemos evitar

No queremos que cada Use Case implemente su propia transacción:

await db.begin()

...

await db.commit()

Eso produciría:

UseCase A → DB
UseCase B → DB
UseCase C → DB

con lógica transaccional duplicada.

La aplicación utilizará:

UnitOfWork

como abstracción única.

9. Repositories dentro de Unit of Work

La UoW será el punto de acceso a los repositorios que participen en la transacción.

Conceptualmente:

class UnitOfWork(ABC):

    users: UserRepository
    organizations: OrganizationRepository
    memberships: MembershipRepository

    outbox: OutboxWriter

Sin embargo, no vamos a declarar todavía todos esos atributos definitivos, porque los módulos y sus aggregates aún no han sido implementados.

Por ahora podemos mantener el contrato mínimo y extenderlo progresivamente.

10. Unit of Work ≠ Repository

Esta separación es fundamental.

Repository

Se ocupa de:

Load Aggregate
Save Aggregate
Delete Aggregate
Unit of Work

Se ocupa de:

BEGIN
COMMIT
ROLLBACK
transaction lifecycle
Outbox

Se ocupa de:

persist event for later publication

Por lo tanto:

Repository
    │
    ▼
Aggregate persistence

UnitOfWork
    │
    ▼
Transaction boundary

Outbox
    │
    ▼
Reliable event publication
11. Transactional Outbox

La operación completa será:

BEGIN
 │
 ├── UserRepository.save(user)
 │
 ├── user.pull_domain_events()
 │
 ├── OutboxWriter.append(event)
 │
COMMIT

Después:

Outbox Worker
      │
      ▼
Event Publisher
      │
      ▼
Event Platform

El Publisher nunca deberá depender de que el Use Case siga ejecutándose.

12. Domain Events dentro de la UoW

El Aggregate mantiene sus eventos:

user.record_event(
    UserCreated(...)
)

La UoW/application boundary deberá garantizar que esos eventos sean llevados al Outbox antes del commit.

La dirección correcta es:

Aggregate
   │
   └── DomainEvent
          │
          ▼
Application
          │
          ▼
Outbox
          │
          ▼
Commit

El Aggregate no conoce Outbox.

13. Regla de ownership

Incorrecto:

class User:

    def create(self):
        ...
        outbox.save(...)

Correcto:

class User:

    def create(self):
        ...
        self.record_event(...)

La infraestructura de eventos es responsabilidad de Application/Infrastructure.

14. Rollback

Si cualquier parte de la operación falla:

Repository
      ↓
Domain
      ↓
Outbox
      ↓
Commit

la UoW ejecutará:

ROLLBACK

Ejemplo:

Create User
    │
    ├── User persisted
    │
    ├── Outbox persisted
    │
    └── Database error
             │
             ▼
          ROLLBACK

Resultado:

User → no existe
Outbox → no existe
15. Excepciones de dominio

Si ocurre:

raise DomainRuleViolation(...)

no debemos convertirla en un commit parcial.

Debe producir:

Domain Exception
      ↓
Application
      ↓
UnitOfWork.rollback()
      ↓
API error
16. Concurrency Conflict

TASK-001-06 definió:

class ConcurrencyConflict(DomainError):
    pass

Ahora la UoW debe permitir que este error provoque rollback.

Ejemplo:

User version = 7

Process A:
    load 7
    save → 8

Process B:
    load 7
    save → CONFLICT

Proceso B:

ConcurrencyConflict
        ↓
ROLLBACK

Nunca debe convertirse silenciosamente en:

last write wins

para aggregates donde el control de concurrencia sea requerido.

17. Idempotencia

La Unit of Work no sustituye el mecanismo de idempotencia de API.

Son conceptos diferentes:

API Idempotency
       │
       ▼
Evitar ejecutar dos veces una misma operación solicitada

mientras:

Unit of Work
       │
       ▼
Garantizar atomicidad de una operación

Y:

Consumer Idempotency
       │
       ▼
Evitar efectos duplicados al procesar eventos

Tenemos entonces tres niveles:

Request Idempotency
        +
Transaction Atomicity
        +
Consumer Idempotency
18. Nested Transactions

No permitiremos que cada Repository abra su propia transacción.

Incorrecto:

UoW BEGIN
   │
   ├── UserRepository
   │       └── BEGIN/COMMIT
   │
   ├── MembershipRepository
   │       └── BEGIN/COMMIT
   │
   └── Outbox
           └── BEGIN/COMMIT

Correcto:

UoW BEGIN
   │
   ├── UserRepository
   ├── MembershipRepository
   ├── Outbox
   │
UoW COMMIT

Una única frontera.

19. PostgreSQL Implementation

La implementación concreta vendrá en Infrastructure.

Propuesta:

identity/shared/infrastructure/
└── persistence/
    └── postgres/
        ├── unit_of_work.py
        ├── session.py
        └── transaction.py

El nombre exacto puede ajustarse cuando implementemos TASK-001-08.

20. PostgreSQL UoW — estructura conceptual
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

Esto es estructura conceptual, no la implementación final de PostgreSQL.

No debemos fijar todavía si utilizaremos:

SQLAlchemy
asyncpg
psycopg

como decisión definitiva de esta tarea.

21. Connection Pool

La UoW no debe crear una conexión PostgreSQL nueva manualmente para cada operación.

Infrastructure manejará:

Connection Pool
      │
      ▼
Unit of Work
      │
      ▼
Transaction

El pool será responsabilidad de la infraestructura PostgreSQL de TASK-001-08.

22. Transaction Context

Cada request que ejecute una operación transaccional deberá tener una frontera claramente observable.

Conceptualmente:

correlation_id
trace_id
transaction
actor
tenant

Esto permitirá correlacionar:

API request
   ↓
Use Case
   ↓
DB transaction
   ↓
Outbox event
   ↓
Event Platform
23. Tenant Context

La UoW no debe permitir accidentalmente que un Repository tenant-aware opere fuera del contexto apropiado.

Conceptualmente:

Request
  ↓
TenantContext
  ↓
Application
  ↓
UnitOfWork
  ↓
Tenant-aware Repository

Una incertidumbre de tenant para una operación que requiere tenant deberá producir:

fail closed

y no:

query all tenants
24. Cross-Tenant Transaction

No debemos interpretar:

UnitOfWork

como permiso para modificar múltiples tenants.

Una transacción que involucre más de una organización debe ser explícitamente autorizada por el caso de uso y por las políticas correspondientes.

Por defecto:

Tenant A
   │
   └── transaction → Tenant A

No:

Tenant A
   │
   └── transaction → Tenant B
25. Outbox Atomicity

El caso crítico será:

User update
+
Security Event

La operación:

BEGIN

UPDATE user

INSERT security_event

INSERT outbox

COMMIT

debe quedar atómicamente persistida.

Si falla:

ROLLBACK

Todo vuelve al estado anterior.

26. Publicación posterior

Una vez hecho:

COMMIT

un proceso independiente podrá hacer:

Outbox
  │
  ▼
Publisher
  │
  ▼
Event Platform

Esto significa que la publicación puede fallar temporalmente sin perder el hecho de negocio:

PostgreSQL:
    committed = YES

Outbox:
    pending = YES

Event Platform:
    unavailable

Cuando vuelva:

Outbox
  ↓
retry
  ↓
Event Platform
27. At-Least-Once

El diseño no busca:

exactly once delivery

como garantía global.

La garantía será:

At-Least-Once
+
Idempotent Consumers

Por eso el Outbox puede ser publicado nuevamente si existe incertidumbre sobre el resultado de una publicación.

28. Transaction State

Podemos modelar conceptualmente:

NEW
 ↓
ACTIVE
 ├───────────────┐
 ↓               ↓
COMMITTED      ROLLED_BACK

Una UoW no debería permitir:

COMMIT
   ↓
COMMIT

ni:

ROLLBACK
   ↓
COMMIT

sin una nueva transacción.

29. Protección contra errores

La implementación debe garantizar:

Exception before commit
        ↓
rollback

Y:

Commit successful
        ↓
do not rollback

Los errores de rollback deben registrarse de forma segura para observabilidad, sin exponer información sensible.

30. Tests

Esta tarea tendrá pruebas unitarias e integración posteriormente.

Test 1 — Commit
BEGIN
save entity
append outbox
COMMIT

Resultado:

entity persisted
outbox persisted
Test 2 — Rollback
BEGIN
save entity
append outbox
exception
ROLLBACK

Resultado:

entity absent
outbox absent
Test 3 — Domain Error
DomainRuleViolation

debe producir:

ROLLBACK
Test 4 — Concurrency
ConcurrencyConflict

debe producir:

ROLLBACK
Test 5 — Outbox Atomicity

Verificar:

business state = committed
outbox = committed

o:

business state = rolled back
outbox = rolled back

Nunca:

business state = committed
outbox = rolled back
31. Architecture Tests

Debemos garantizar:

Domain
  ✗ PostgreSQL
  ✗ SQLAlchemy
  ✗ Redis

y:

Repository Contract
  ✗ SQL
  ✗ HTTP

Mientras:

Infrastructure
  ✓ PostgreSQL
  ✓ ORM/driver
  ✓ transaction implementation
32. Observabilidad

La UoW debe producir métricas/eventos operacionales conceptuales como:

transactions_started
transactions_committed
transactions_rolled_back
transaction_duration
concurrency_conflicts
commit_failures
rollback_failures

Nunca debemos registrar:

password
access token
refresh token
MFA secret
recovery code
private key
33. Performance

La UoW no debe introducir latencia innecesaria.

Debemos evitar:

BEGIN
   ↓
HTTP request externo
   ↓
AI call
   ↓
event platform
   ↓
COMMIT

Una transacción de base de datos no debe mantenerse abierta esperando servicios externos.

Correcto:

BEGIN
   ↓
DB operations
   ↓
Outbox
   ↓
COMMIT
   ↓
external processing

Esto será particularmente importante para EVOXA porque los componentes de IA nunca deben convertirse en parte innecesaria de una transacción PostgreSQL.

34. AI Boundary

Un AI Agent puede iniciar una operación autorizada:

AI Agent
   ↓
Authorization
   ↓
Application Use Case
   ↓
UnitOfWork

Pero:

AI Agent
   ✗
direct DB transaction

no está permitido.

La IA utiliza los mismos contratos y controles que cualquier otro actor.

35. Archivos de esta tarea

La primera versión debería afectar principalmente:

apps/api/app/domains/identity/
│
├── shared/
│   ├── application/
│   │   └── ports.py
│   │
│   └── domain/
│       └── exceptions.py
│
└── tests/
    └── ...

Y preparar la futura infraestructura:

shared/infrastructure/
└── persistence/
    └── postgres/
        └── unit_of_work.py

Pero la implementación PostgreSQL real queda para:

TASK-001-08 — PostgreSQL Infrastructure

36. Definition of Done
[ ] UnitOfWork contract definido
[ ] begin() definido
[ ] commit() definido
[ ] rollback() definido
[ ] async context manager definido
[ ] Repository/UoW separation validada
[ ] Outbox compatible con UoW
[ ] Domain Events compatibles
[ ] ConcurrencyConflict integrado
[ ] Rollback ante DomainError
[ ] Tenant boundary documentado
[ ] No nested transactions
[ ] No external calls dentro de DB transaction
[ ] Tests de commit/rollback
[ ] Tests de atomicidad Outbox
[ ] Architecture boundary tests
[ ] Observability definida
[ ] Sin dependencia PostgreSQL en Domain
37. Criterios de aceptación
ID	Criterio
AC-01	Existe un contrato único de Unit of Work
AC-02	La UoW define BEGIN/COMMIT/ROLLBACK
AC-03	Repository no controla la transacción global
AC-04	Application no depende de PostgreSQL
AC-05	Domain no conoce UoW concreta
AC-06	Business state + Outbox son atómicos
AC-07	Error antes de commit produce rollback
AC-08	Concurrency conflict produce rollback
AC-09	No existen nested transactions por Repository
AC-10	Tenant context no puede quedar implícitamente global
AC-11	No se realizan llamadas externas dentro de la transacción
AC-12	La publicación de eventos ocurre después del commit
AC-13	El diseño soporta At-Least-Once
AC-14	La UoW es testeable sin PostgreSQL
AC-15	La implementación concreta queda aislada en Infrastructure
38. Resultado arquitectónico

Con TASK-001-07 completamos una pieza fundamental:

             Application
                  │
                  ▼
           ┌──────────────┐
           │ UnitOfWork   │
           └──────┬───────┘
                  │
       ┌──────────┼──────────┐
       ▼          ▼          ▼
 Repository   Repository   Outbox
       │          │          │
       └──────────┼──────────┘
                  ▼
             PostgreSQL
                  │
                COMMIT
                  │
                  ▼
            Outbox Worker
                  │
                  ▼
           Event Platform

Y la regla de EVOXA queda clara:

Una operación de negocio no se considera persistida hasta que su estado y los hechos que deben publicarse hayan sido confirmados en la misma frontera transaccional.

Estado del bloque Foundation
TASK-001-01  API Runtime              ✅
TASK-001-02  Identity Domain Package  ✅
TASK-001-03  Layer Architecture      ✅
TASK-001-04  Shared Kernel            ✅
TASK-001-05  Domain Events             ✅
TASK-001-06  Repository Contracts      ✅
TASK-001-07  Unit of Work              ✅

Siguiente: TASK-001-08 — PostgreSQL Infrastructure, donde ya podremos pasar de los contratos abstractos a la infraestructura real: configuración de PostgreSQL, connection pool, sesiones/transacciones, health check, manejo de errores de DB y preparación para migrations, sin crear todavía los modelos de negocio completos.
