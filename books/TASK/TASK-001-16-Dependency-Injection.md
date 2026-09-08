TASK-001-16 — Dependency Injection

Implementation Story: IS-001 — Identity Domain Foundation
ESP: ESP-0001/14 — Identity Implementation Requirements
Related ESP: ESP-0001/02, ESP-0001/03, ESP-0001/10, ESP-0001/11, ESP-0001/12, ESP-0001/13
Priority: 🔴 Critical
Status: 🟢 Ready for Implementation
Task Type: Architecture / Application Infrastructure

1. Objetivo

Establecer el mecanismo oficial de Dependency Injection (DI) para EVOXA Identity, permitiendo que las capas superiores reciban sus dependencias sin crear directamente implementaciones concretas.

El objetivo es conseguir:

desacoplamiento entre capas;
testabilidad;
sustitución de implementaciones;
configuración centralizada;
composición explícita de dependencias;
evitar estado global mutable;
facilitar evolución futura del Modular Monolith hacia servicios independientes.

La regla fundamental será:

Las dependencias se construyen en la composición de la aplicación y se inyectan hacia los consumidores.

Esto evita que los casos de uso, dominios y controladores creen directamente bases de datos, Redis, publishers, repositories u otros servicios.

2. Contexto arquitectónico

La arquitectura establecida para Identity es:

Presentation
     ↓
API
     ↓
Application
     ↓
Domain
     ↑
Infrastructure

DI debe respetar esta arquitectura.

Por ejemplo:

FastAPI Endpoint
      │
      ▼
Application Use Case
      │
      ├── UserRepository
      ├── UnitOfWork
      ├── AuthorizationService
      └── EventPublisher
              │
              ▼
       Infrastructure

El caso de uso conoce interfaces/ports, no implementaciones concretas.

3. Principio de inversión de dependencias

La dependencia debe apuntar hacia abstracciones.

Incorrecto
class CreateUserUseCase:

    def __init__(self):
        self.repository = PostgreSQLUserRepository()

Esto acopla Application directamente con Infrastructure.

Correcto
class CreateUserUseCase:

    def __init__(
        self,
        repository: UserRepository,
        unit_of_work: UnitOfWork,
    ):
        self.repository = repository
        self.unit_of_work = unit_of_work

La composición externa decide qué implementación utilizar:

UserRepository
      ↑
      │
PostgreSQLUserRepository
4. Responsabilidades de DI

El sistema de Dependency Injection deberá encargarse de:

construir dependencias;
resolver dependencias;
controlar lifetime;
proporcionar repositories;
proporcionar Unit of Work;
proporcionar servicios de aplicación;
proporcionar ports;
conectar Infrastructure con Application;
integrar dependencias con FastAPI;
facilitar overrides durante testing.

No debe encargarse de lógica de negocio.

5. Dependency Injection vs Service Locator

EVOXA no utilizará Service Locator como mecanismo principal.

Evitar
container.resolve("user_repository")

desde cualquier parte de la aplicación.

Esto oculta las dependencias.

Preferir
class UserApplicationService:

    def __init__(
        self,
        user_repository: UserRepository,
        uow: UnitOfWork,
    ):
        ...

Las dependencias deben ser visibles en el constructor.

6. Composición de la aplicación

Se establecerá un Composition Root.

Conceptualmente:

                    Composition Root
                           │
          ┌────────────────┼────────────────┐
          ▼                ▼                ▼
     PostgreSQL          Redis        Event Platform
          │                │                │
          ▼                ▼                ▼
    Repositories       Cache Ports      Publishers
          │                │                │
          └────────────────┼────────────────┘
                           ▼
                    Application Services
                           │
                           ▼
                        API

El Composition Root será el lugar autorizado para ensamblar implementaciones concretas.

7. Estructura propuesta

A partir de la estructura establecida en TASK-001-01 a TASK-001-15:

apps/api/app/
├── main.py
│
├── core/
│   ├── config/
│   ├── security/
│   ├── observability/
│   ├── infrastructure/
│   │   ├── database/
│   │   └── redis/
│   │
│   └── dependencies/
│       ├── __init__.py
│       ├── container.py
│       ├── identity.py
│       └── request.py
│
├── api/
│   └── v1/
│
└── domains/
    └── identity/
        ├── account/
        ├── organization/
        ├── membership/
        ├── authentication/
        ├── authorization/
        ├── role_permission/
        ├── policy/
        ├── session/
        ├── token/
        ├── mfa/
        ├── recovery/
        ├── federation/
        ├── provisioning/
        ├── audit/
        ├── security_events/
        ├── integration/
        └── shared/
8. Composition Container

Se puede establecer un container de composición, pero debe ser simple y explícito.

Conceptualmente:

class IdentityContainer:

    def __init__(self, settings):
        self.settings = settings

    def user_repository(self) -> UserRepository:
        ...

    def unit_of_work(self) -> UnitOfWork:
        ...

    def event_publisher(self) -> EventPublisher:
        ...

Sin embargo, el container no debe transformarse en un objeto global accesible desde todo el sistema.

9. FastAPI Dependency Injection

FastAPI podrá utilizar su mecanismo de Depends como mecanismo de integración en la capa API.

Ejemplo conceptual:

@router.post("/users")
async def create_user(
    command: CreateUserCommand,
    use_case: CreateUserUseCase = Depends(
        get_create_user_use_case
    ),
):
    return await use_case.execute(command)

La función:

def get_create_user_use_case(
    repository=Depends(get_user_repository),
    uow=Depends(get_unit_of_work),
):
    return CreateUserUseCase(
        repository=repository,
        uow=uow,
    )

Esto mantiene el ensamblaje fuera del caso de uso.

10. Dependency Lifetimes

Las dependencias deberán tener un lifetime explícito.

Se contemplan inicialmente:

Application lifetime

Para recursos compartidos y seguros:

Database Engine
Redis Connection Pool
Configuration
HTTP clients
Event Platform clients

Normalmente viven durante el proceso.

Request lifetime

Para objetos asociados a una operación HTTP:

Database Session
UnitOfWork
Request Context
Actor Context
Tenant Context
Trace Context

Ejemplo:

HTTP Request
     │
     ▼
Database Session
     │
     ▼
UnitOfWork
     │
     ▼
Use Case
     │
     ▼
Commit / Rollback
Transient

Objetos sin estado persistente:

Use Cases
Domain Services
Mappers
Validators
Decision Builders

Cuando corresponda.

11. Unit of Work y DI

La integración con UnitOfWork es especialmente importante.

El UoW debe ser creado para el contexto correcto de la operación.

Ejemplo:

async def get_unit_of_work() -> UnitOfWork:
    return PostgreSQLUnitOfWork(...)

Y posteriormente:

class CreateUserUseCase:

    def __init__(
        self,
        user_repository: UserRepository,
        uow: UnitOfWork,
    ):
        self.user_repository = user_repository
        self.uow = uow

    async def execute(self, command):
        async with self.uow:
            ...

La implementación concreta continúa perteneciendo a Infrastructure.

12. Repository Injection

Los repositories se inyectan mediante sus contratos.

Ejemplo:

class UserRepository(ABC):

    @abstractmethod
    async def get_by_id(
        self,
        user_id: UUID,
    ) -> User | None:
        ...

Infrastructure:

class PostgreSQLUserRepository(UserRepository):

    def __init__(self, session):
        self.session = session

Composition Root:

UserRepository
       ▲
       │
PostgreSQLUserRepository
       ▲
       │
DI

El Application Layer nunca debe hacer:

PostgreSQLUserRepository(...)
13. Event Publisher Injection

La misma regla aplica al Event Publisher.

Application:

class EventPublisher(ABC):

    @abstractmethod
    async def publish(self, event):
        ...

Infrastructure:

class EventPlatformPublisher(EventPublisher):
    ...

Application recibe:

class SomeUseCase:

    def __init__(
        self,
        event_publisher: EventPublisher,
    ):
        self.event_publisher = event_publisher

Pero para eventos transaccionales:

La publicación no sustituye al Transactional Outbox.

El UoW debe mantener la atomicidad entre:

Business State
      +
Outbox Event

La publicación posterior seguirá siendo responsabilidad del mecanismo de integración definido en TASK-001-05.

14. Tenant Context Injection

El Tenant Context será explícito.

Conceptualmente:

@dataclass(frozen=True)
class TenantContext:
    organization_id: UUID | None

Un componente de API/Application podrá recibirlo:

class SomeUseCase:

    def __init__(
        self,
        tenant_context: TenantContext,
    ):
        self.tenant_context = tenant_context

Pero hay una regla crítica:

DI no determina si el usuario tiene acceso al tenant.

DI únicamente transporta el contexto.

La autorización y validación de membership corresponden al sistema de Identity Authorization.

15. Actor Context Injection

Igualmente:

@dataclass(frozen=True)
class ActorContext:
    actor_type: ActorType
    actor_id: UUID | None

Posibles actores:

USER
SERVICE
SYSTEM
AI_AGENT

El actor context tampoco otorga permisos.

Actor Context
      │
      ▼
Authorization Runtime
      │
      ▼
Decision
16. Trace Context

El TraceContext desarrollado en TASK-001-14 podrá estar disponible mediante DI.

Ejemplo:

class SomeUseCase:

    def __init__(
        self,
        trace_context: TraceContext,
    ):
        self.trace_context = trace_context

Debe mantenerse separado de:

SecurityContext
TenantContext
ActorContext
IdempotencyContext

No se deben fusionar estos conceptos solamente para simplificar DI.

17. Request Context

Puede existir un contexto transversal:

@dataclass(frozen=True)
class RequestContext:
    request_id: str
    correlation_id: str
    trace_id: str

Pero no debe contener automáticamente:

password
JWT
refresh token
cookies
MFA secrets
API secrets
18. Security Boundaries

DI no puede utilizarse para saltarse controles de seguridad.

Por ejemplo, nunca debería existir:

get_authorized_user_without_auth()

para facilitar tests o desarrollo.

Los overrides de testing deben ser explícitos y aislados.

19. Testing mediante DI

Una de las principales ventajas será poder reemplazar Infrastructure.

Por ejemplo:

class InMemoryUserRepository(UserRepository):
    ...

Testing:

CreateUserUseCase
      │
      ├── InMemoryUserRepository
      └── FakeUnitOfWork

Producción:

CreateUserUseCase
      │
      ├── PostgreSQLUserRepository
      └── PostgreSQLUnitOfWork

La lógica del caso de uso permanece idéntica.

20. FastAPI Dependency Overrides

Para tests de API se podrán utilizar overrides.

Conceptualmente:

app.dependency_overrides[
    get_user_repository
] = get_test_user_repository

Esto permite:

tests aislados;
evitar dependencias externas;
simular errores;
controlar datos;
probar autorización;
probar escenarios de fallo.

Los overrides deben existir únicamente dentro del contexto de testing.

21. Prohibiciones

Quedan prohibidos:

❌ Instanciar Infrastructure dentro de Domain
PostgreSQLUserRepository(...)
❌ Instanciar Infrastructure dentro de Application
Redis(...)
❌ Acceder directamente al container global
container.get(...)

desde cualquier módulo.

❌ Singleton global mutable
GLOBAL_DATABASE_SESSION
❌ Dependencias ocultas
class Service:

    def execute(self):
        redis = get_global_redis()
❌ DI para lógica de negocio

El container no decide:

ALLOW
DENY
MFA_REQUIRED
REAUTHENTICATE

Eso pertenece al dominio/autorización.

22. DI y Authorization Runtime

La futura implementación del Authorization Runtime tendrá múltiples componentes:

AuthorizationRuntime
RequestValidator
ActorValidator
TenantGuard
ContextBuilder
PolicyMatcher
PolicyEvaluator
RBACEvaluator
OwnershipEvaluator
ABACEvaluator
DelegationEvaluator
ConflictResolver
CriticalityEvaluator
FreshnessEvaluator
CacheEvaluator
DecisionBuilder
AuditEmitter

DI permitirá construir este grafo.

Conceptualmente:

AuthorizationRuntime
        │
        ├── RequestValidator
        ├── TenantGuard
        ├── PolicyEvaluator
        ├── RBACEvaluator
        ├── ABACEvaluator
        ├── OwnershipEvaluator
        ├── DelegationEvaluator
        ├── CriticalityEvaluator
        ├── FreshnessEvaluator
        └── DecisionBuilder

Pero TASK-001-16 no implementa todavía Authorization Runtime.

23. DI y AI Agents

Los AI Agents también utilizarán el mismo mecanismo de dependencias.

AI Agent
   │
   ▼
Application Service
   │
   ▼
Authorization Runtime

Nunca:

AI Agent
   │
   └── bypass Authorization

La inyección de una dependencia no significa otorgar privilegios.

24. Manejo de configuración

Las dependencias deben construirse utilizando la configuración central.

Ejemplo conceptual:

settings = get_settings()

database = create_database(
    settings.database
)

redis = create_redis(
    settings.redis
)

No:

database = create_database(
    host="localhost",
    password="..."
)

Las credenciales deben provenir del mecanismo de configuración/secrets definido en TASK-001-10.

25. Gestión de recursos

DI debe garantizar correctamente:

startup
   │
   ├── Database Engine
   ├── Redis Pool
   └── Event Clients
   │
   ▼
runtime
   │
   ▼
shutdown
   │
   ├── close sessions
   ├── close Redis
   ├── dispose DB
   └── close external clients

No deben quedar conexiones abiertas.

26. Manejo de errores

Si una dependencia crítica no puede inicializarse:

Database unavailable
Redis configuration invalid
Secret unavailable
KMS unavailable

la aplicación debe aplicar las reglas de seguridad correspondientes.

Particularmente:

DI no debe convertir un fallo de infraestructura en una autorización ALLOW.

27. Observabilidad

La creación y utilización de dependencias debe ser observable cuando sea operacionalmente relevante.

Pero nunca registrar:

password
token
secret
private key
MFA secret
cookie
API credential

Se podrán registrar métricas como:

dependency_initialization_failures
database_pool_exhaustion
redis_connection_failures
dependency_resolution_errors
28. Scope de implementación
Incluido
Composition Root;
container básico;
integración FastAPI DI;
repository injection;
UnitOfWork injection;
configuration injection;
TenantContext;
ActorContext;
TraceContext;
infraestructura lifecycle;
testing overrides;
dependency boundary tests.
No incluido
Authorization Runtime;
MFA;
Session;
Refresh Token;
Policy Engine;
Redis Authorization Cache;
Event Broker;
Service Mesh;
microservices;
service discovery distribuido.
29. Archivos propuestos
apps/api/app/core/dependencies/
├── __init__.py
├── container.py
├── identity.py
└── request.py

Y potencialmente:

apps/api/tests/
└── core/
    └── dependencies/
        ├── test_container.py
        ├── test_request_dependencies.py
        └── test_dependency_boundaries.py

Además, se modificarán los puntos de composición de:

core/infrastructure/database/
core/infrastructure/redis/
domains/identity/shared/
api/v1/

cuando corresponda.

30. Pruebas requeridas
Unitarias
DI-UNIT-001

El container construye una dependencia válida.

DI-UNIT-002

Las implementaciones concretas cumplen sus contracts.

DI-UNIT-003

Los lifetimes son correctos.

DI-UNIT-004

Las dependencias no contienen estado global mutable.

Integración
DI-INT-001

FastAPI resuelve correctamente un caso de uso.

DI-INT-002

Repository real se conecta mediante DI.

DI-INT-003

UnitOfWork se crea correctamente por operación.

DI-INT-004

Shutdown libera recursos.

Testing
DI-TEST-001

Repository puede reemplazarse por fake.

DI-TEST-002

UoW puede reemplazarse por fake.

DI-TEST-003

Event publisher puede reemplazarse por fake.

DI-TEST-004

Los overrides de FastAPI no persisten fuera del test.

Architecture tests
DI-ARCH-001

Domain no importa Infrastructure.

DI-ARCH-002

Application no importa implementaciones PostgreSQL.

DI-ARCH-003

Application no importa cliente Redis concreto.

DI-ARCH-004

Infrastructure puede implementar contracts.

DI-ARCH-005

API utiliza Application.

DI-ARCH-006

No existe Service Locator global.

31. Acceptance Criteria
AC-01

Given que la aplicación inicia,
When se construye el Composition Root,
Then las dependencias principales quedan registradas.

AC-02

Given un Application Service,
When recibe sus dependencias,
Then no crea Infrastructure directamente.

AC-03

Given un repository contract,
When se ejecuta producción,
Then se utiliza su implementación PostgreSQL correspondiente.

AC-04

Given un test,
When se sustituye un repository,
Then el Application Service funciona sin PostgreSQL real.

AC-05

Given un HTTP request,
When se resuelve un use case,
Then FastAPI proporciona sus dependencias correctamente.

AC-06

Given un UnitOfWork request-scoped,
When termina la operación,
Then su recurso se libera correctamente.

AC-07

Given TenantContext,
When se inyecta,
Then DI no interpreta ni concede autorización.

AC-08

Given ActorContext,
When se inyecta,
Then el actor no recibe privilegios implícitos.

AC-09

Given una dependencia crítica que falla,
When se intenta utilizar,
Then no se produce un fail-open de seguridad.

AC-10

Given un test con dependency override,
When termina el test,
Then el override no afecta otros tests.

AC-11

Given Domain code,
When se ejecutan architecture tests,
Then no puede importar Infrastructure.

AC-12

Given Application code,
When se ejecutan architecture tests,
Then no puede importar implementaciones concretas de Infrastructure.

AC-13

Given múltiples requests concurrentes,
When cada request recibe sus dependencias,
Then no existe contaminación de estado entre requests.

AC-14

Given un AI Agent,
When utiliza Application Services,
Then sigue exactamente el mismo modelo de autorización.

AC-15

Given shutdown de la aplicación,
When se cierran los recursos,
Then DB/Redis/clientes se liberan correctamente.

32. Definition of Done

TASK-001-16 estará completa cuando:

 Composition Root definido.
 DI container básico implementado.
 FastAPI integrado.
 Repository injection implementado.
 UnitOfWork injection implementado.
 Configuration injection implementado.
 Tenant Context integrado.
 Actor Context integrado.
 Trace Context integrado.
 Lifetimes definidos.
 Startup/shutdown integrado.
 Testing overrides funcionando.
 Architecture tests implementados.
 No Service Locator global.
 No Infrastructure dentro de Domain/Application.
 No secrets expuestos por DI.
 Tests unitarios e integración pasando.
 Documentación actualizada.
 Traceability registrada.
33. Traceability
Blueprint
   │
   ▼
ARCHITECTURE-MAP
   │
   ▼
ESP-0001/14
   │
   ├── ADR-001 Identity Architecture
   ├── ADR-006 PostgreSQL Source of Truth
   ├── ADR-007 Transactional Outbox
   ├── ADR-027 Configuration & Secrets
   ├── ADR-028 Observability
   ├── ADR-031 HA & Failover
   ├── ADR-045 Authorization Policy
   ├── ADR-049 Authorization Runtime
   └── ADR-067/069 Module Boundaries
   │
   ▼
IS-001
   │
   ▼
TASK-001-16
   │
   ▼
Source Code
34. Decisiones importantes para EVOXA

Hay cuatro reglas que quiero que mantengamos desde este punto y durante toda la programación:

1. Constructor Injection como patrón principal
Dependency → Constructor → Service
2. FastAPI Depends solamente como mecanismo de integración

No queremos que FastAPI contamine Domain.

3. Composition Root centralizado
Infrastructure
      ↓
Composition Root
      ↓
Application
      ↓
API
4. DI jamás decide seguridad
DI
 ≠
Authorization

La autorización seguirá siendo responsabilidad exclusiva del Authorization Runtime.

35. Estado de IS-001

Con TASK-001-16 cerramos:

TASK-001-01  API Runtime Foundation             ✅
TASK-001-02  Identity Domain Package            ✅
TASK-001-03  Layer Architecture                 ✅
TASK-001-04  Identity Shared Kernel             ✅
TASK-001-05  Domain Events Foundation            ✅
TASK-001-06  Repository Contracts               ✅
TASK-001-07  Unit of Work                       ✅
TASK-001-08  PostgreSQL Infrastructure           ✅
TASK-001-09  Redis Infrastructure                ✅
TASK-001-10  Configuration & Secrets             ✅
TASK-001-11  Security Foundation                 ✅
TASK-001-12  Tenant Context Foundation           ✅
TASK-001-13  Actor Context                       ✅
TASK-001-14  Correlation & Trace Context         ✅
TASK-001-15  Exception Model                     ✅
TASK-001-16  Dependency Injection                🟢
Siguiente

TASK-001-17 — Architecture Boundary Tests

Esta será especialmente importante porque no solo vamos a probar que el código funciona, sino que vamos a crear tests automatizados que impidan que en el futuro un desarrollador rompa las reglas arquitectónicas de EVOXA.
