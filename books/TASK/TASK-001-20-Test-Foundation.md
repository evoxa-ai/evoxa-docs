TASK-001-20 — Test Foundation

Identity Implementation Story: IS-001 — Identity Domain Foundation
ESP: ESP-0001/13 — Identity Testing & Quality Requirements
ESP relacionado: ESP-0001/14 — Identity Implementation Requirements
Dependencias: TASK-001-01 → TASK-001-19
Prioridad: 🔴 Critical
Estado: 🟢 Ready for Implementation

1. Objetivo

Establecer la infraestructura base de testing de EVOXA Identity antes de comenzar a implementar funcionalidades de negocio.

Este Task no implementa todavía las pruebas completas de Authentication, Authorization, MFA, Sessions, etc.; establece el framework, convenciones, fixtures, factories, mocks, configuración y quality gates que utilizarán posteriormente todas esas pruebas.

La idea es que desde este punto:

ningún módulo nuevo de Identity debería comenzar a implementarse sin disponer de una estructura de pruebas correspondiente.

Esto materializa el principio:

Source Code
     │
     ├── Unit Tests
     ├── Integration Tests
     ├── Contract Tests
     └── Architecture Tests

y posteriormente se extenderá hacia:

Unit
  ↓
Component
  ↓
Integration
  ↓
API
  ↓
E2E
  ↓
Security
  ↓
Performance
  ↓
Chaos / Recovery
2. Objetivos específicos

TASK-001-20 debe dejar preparada la plataforma para:

Ejecutar tests unitarios.
Ejecutar tests de integración.
Crear fixtures reutilizables.
Crear factories de entidades.
Mockear infraestructura.
Probar PostgreSQL.
Probar Redis.
Probar eventos/outbox posteriormente.
Probar límites arquitectónicos.
Medir coverage.
Ejecutar tests de forma determinista.
Separar tests rápidos de tests de infraestructura.
Integrar testing con CI/CD.
Preparar los Quality Gates definidos por ESP-0001/13.
3. Principio fundamental

El framework de testing debe respetar la arquitectura:

Tests
 │
 ├── Domain
 │     └── no infrastructure required
 │
 ├── Application
 │     └── ports can be mocked
 │
 ├── Infrastructure
 │     └── real PostgreSQL / Redis
 │
 └── API
       └── HTTP contract

Los tests no deben crear dependencias artificiales que violen la arquitectura.

Por ejemplo:

❌ Domain Test
      ↓
   PostgreSQL

no debería ser necesario.

Mientras que:

✅ Infrastructure Test
      ↓
   PostgreSQL

sí corresponde.

4. Stack inicial de testing

Siguiendo el baseline de implementación de ESP-0001/14:

Python
FastAPI
PostgreSQL
Redis

La propuesta inicial para testing es:

pytest
pytest-asyncio
httpx
pytest-cov

Y posteriormente, cuando corresponda:

testcontainers
hypothesis

La incorporación de herramientas adicionales deberá justificarse por necesidad real.

5. Estructura propuesta

La estructura inicial:

apps/api/
│
├── app/
│   ├── ...
│
└── tests/
    │
    ├── conftest.py
    │
    ├── unit/
    │   └── identity/
    │       ├── account/
    │       ├── organization/
    │       ├── membership/
    │       ├── authentication/
    │       ├── authorization/
    │       ├── role_permission/
    │       ├── policy/
    │       ├── session/
    │       ├── token/
    │       ├── mfa/
    │       ├── recovery/
    │       ├── federation/
    │       ├── provisioning/
    │       ├── audit/
    │       └── security_events/
    │
    ├── integration/
    │   ├── database/
    │   ├── redis/
    │   ├── events/
    │   └── identity/
    │
    ├── api/
    │   └── identity/
    │
    ├── architecture/
    │   └── identity/
    │
    ├── contracts/
    │   └── identity/
    │
    ├── fixtures/
    │   ├── users.py
    │   ├── organizations.py
    │   ├── memberships.py
    │   └── security.py
    │
    ├── factories/
    │   ├── user_factory.py
    │   ├── organization_factory.py
    │   └── membership_factory.py
    │
    └── helpers/
        ├── assertions.py
        └── async_helpers.py

No debemos crear todas estas pruebas ahora.

La estructura prepara el terreno para las futuras IS.

6. Configuración de pytest

Propuesta:

apps/api/pytest.ini

o preferentemente una configuración centralizada en:

pyproject.toml

Ejemplo conceptual:

[tool.pytest.ini_options]
asyncio_mode = "auto"

testpaths = [
    "tests"
]

addopts = [
    "-ra",
    "--strict-markers"
]

markers = [
    "unit: Unit tests",
    "integration: Integration tests",
    "api: API tests",
    "architecture: Architecture tests",
    "security: Security tests",
    "slow: Slow tests"
]

La configuración definitiva dependerá de la estructura real del proyecto.

7. Test Categories

Desde el comienzo debemos poder clasificar los tests.

Unit

Prueban una unidad aislada:

Entity
Value Object
Aggregate
Domain Service
Policy evaluator
Mapper
Validator

Características:

rápidos
deterministas
sin DB
sin Redis
sin red
Integration

Prueban integración real entre componentes.

Ejemplos:

Repository ↔ PostgreSQL

Redis Adapter ↔ Redis

Outbox ↔ PostgreSQL

Application ↔ Repository
API

Prueban el contrato HTTP:

HTTP Request
      ↓
FastAPI
      ↓
Application
      ↓
Response

Incluyen posteriormente:

status code
response schema
error envelope
correlation ID
authentication
authorization
idempotency
Architecture

Validan que el código respete las reglas arquitectónicas.

Ejemplo:

Domain
  ❌ → FastAPI

Domain
  ❌ → SQLAlchemy

Domain
  ❌ → Redis

Application
  ❌ → concrete infrastructure implementation

Estos tests serán especialmente importantes para mantener el Modular Monolith limpio.

8. Fixtures

Las fixtures deben ser reutilizables y pequeñas.

Ejemplo conceptual:

@pytest.fixture
def user_id():
    return uuid4()

Posteriormente:

@pytest.fixture
def user():
    return User(...)

Y para infraestructura:

@pytest.fixture
async def db_session():
    ...

La fixture no debe esconder comportamiento importante.

Debe ser evidente qué dependencia está utilizando el test.

9. Factories

Las factories permiten crear objetos de prueba consistentes.

Por ejemplo:

class UserFactory:

    @staticmethod
    def create(
        email: str = "test@example.com",
        status: str = "ACTIVE",
    ):
        ...

Posteriormente podremos crear:

user = UserFactory.create()

o:

user = UserFactory.create(
    status="SUSPENDED"
)

Esto será particularmente importante para:

Account
Organization
Membership
Session
MFA
Delegation
Authorization
10. No utilizar datos de producción

Los tests jamás deben depender de:

❌ producción
❌ usuarios reales
❌ emails reales
❌ tokens reales
❌ MFA secrets reales
❌ API keys reales
❌ passwords reales
❌ credenciales reales

Todo debe ser generado específicamente para testing.

11. Determinismo

Un test debe producir el mismo resultado con el mismo estado inicial.

Evitar:

datetime.now()

directamente dentro de lógica que se quiera testear.

Preferir posteriormente una abstracción:

Clock

que permita:

fake_clock

Esto será especialmente importante para:

JWT
Sessions
Refresh Tokens
MFA challenges
Recovery tokens
Policy effective dates
TTL
Cache freshness.
12. UUIDs

Los identificadores generados aleatoriamente pueden dificultar ciertos tests.

Para tests donde importe reproducibilidad:

UUID conocido

o factories controladas.

Ejemplo:

USER_ID = UUID("00000000-0000-0000-0000-000000000001")

No debemos depender de UUIDs aleatorios para assertions específicas.

13. Async Testing

Identity utilizará infraestructura asíncrona.

Por lo tanto:

async def test_something():
    ...

debe ser soportado directamente.

Por ejemplo:

@pytest.mark.asyncio
async def test_repository_returns_user():
    user = await repository.get_by_id(user_id)

    assert user is not None

La configuración debe evitar mezclar innecesariamente múltiples event loops.

14. Mocking

Los mocks deben utilizarse en los límites de infraestructura.

Ejemplo:

Application
     │
     ↓
Repository Port
     │
     └── FakeRepository

No debemos hacer:

Domain
  ↓
Mock de todo

porque eso puede producir tests que pasan aunque el diseño real sea incorrecto.

15. Fakes vs Mocks

Regla propuesta:

Preferir Fake cuando:

Existe una abstracción relativamente estable.

Ejemplo:

FakeUserRepository
FakeUnitOfWork
FakeEventPublisher
Mock cuando:

Necesitamos verificar una interacción específica.

Ejemplo:

publisher.publish(...)
16. Test de Unit of Work

TASK-001-20 debe preparar pruebas para comprobar el contrato definido en TASK-001-07.

Conceptualmente:

BEGIN
   ↓
operation
   ↓
COMMIT

y:

BEGIN
   ↓
operation fails
   ↓
ROLLBACK

Posteriormente debemos comprobar además:

Business State
      +
Outbox
      ↓
same transaction
17. Test de excepciones

TASK-001-15 define el modelo de errores.

Debemos preparar tests que garanticen que:

DomainError
     ↓
Application Mapper
     ↓
HTTP Error

no exponga información interna.

Por ejemplo:

❌ SQL query
❌ stack trace
❌ database hostname
❌ password
❌ token
❌ internal filesystem
18. Test de correlation ID

Debe existir una prueba que compruebe:

Request
  ↓
X-Correlation-ID
  ↓
Application
  ↓
Log
  ↓
Response

Si el cliente no envía uno:

Request
  ↓
generate correlation ID
  ↓
Response header

Esto conecta TASK-001-14 con TASK-001-18.

19. Test de Tenant Context

Aunque todavía no exista Membership completo, debemos dejar preparada la estructura para validar:

tenant A
   ↓
resource A
   ✅ allowed

tenant A
   ↓
resource B
   ❌ denied

La regla fundamental:

La incertidumbre del tenant nunca debe producir un ALLOW.

Esto será crítico cuando implementemos Authorization.

20. Test de Actor Context

Debemos poder representar:

USER
SERVICE
SYSTEM
AI_AGENT

y comprobar que el actor se mantiene correctamente en:

Application
↓
Audit
↓
Event metadata
↓
Observability

El hecho de ser:

AI_AGENT

no debe otorgar privilegios adicionales.

21. Architecture Tests

Este será uno de los elementos más importantes de TASK-001-20.

Debemos comprobar automáticamente reglas como:

Domain
  ↓
must not import
  ├── FastAPI
  ├── SQLAlchemy
  ├── Redis
  └── infrastructure

También:

Identity Module A
  ❌
direct import
  ↓
Identity Module B infrastructure

Y:

API
 ↓
Application

en lugar de:

API
 ↓
Repository
22. Tests de Shared Kernel

Debemos verificar que Shared Kernel permanezca pequeño.

Por ejemplo:

shared/
   ↓
NO imports
   ↓
account
organization
membership
authorization
mfa

Esto evita que Shared Kernel termine convirtiéndose en un God Module.

23. Coverage

ESP-0001/13 establece los objetivos globales de calidad.

Baseline:

Overall Coverage
≥ 85%

Y para componentes críticos:

≥ 95%

Especialmente:

Authorization Runtime
Criticality
Authentication
MFA
Token Lifecycle
Policy Engine
Security-sensitive logic

Pero TASK-001-20 no exige alcanzar todavía esos porcentajes para funcionalidades que aún no existen.

Su responsabilidad es dejar configurada la medición.

24. Coverage no significa calidad

No debemos aceptar:

100% coverage

como evidencia suficiente.

Un test puede ejecutar una línea sin comprobar su comportamiento.

Por eso posteriormente combinaremos:

Coverage
+
Golden Scenarios
+
Security Invariants
+
Mutation Testing
+
Property Testing
+
Integration

Esto está alineado con ESP-0001/13 y ADR-062/063.

25. Security Testing Foundation

TASK-001-20 debe dejar preparada la clasificación:

@pytest.mark.security

para pruebas futuras.

Ejemplos:

tenant isolation
authentication bypass
authorization bypass
MFA bypass
token reuse
privilege escalation
secret leakage
cross-tenant access

No implementaremos todas aquí.

26. Negative Testing

Desde el principio debemos aceptar una filosofía:

Los errores y accesos inválidos también son comportamientos que debemos probar.

Ejemplos futuros:

missing tenant
invalid actor
expired session
revoked token
invalid permission
unknown policy
invalid resource
invalid organization

Y especialmente:

UNKNOWN
ERROR
INDETERMINATE

no deben convertirse en:

ALLOW
27. Database Testing

Cuando se implementen repositories:

tests/integration/database/

deberá utilizar una base de datos específica para tests.

Nunca:

production DB

La infraestructura deberá permitir:

setup
 ↓
test
 ↓
cleanup

de manera controlada.

28. Redis Testing

Posteriormente:

tests/integration/redis/

validará:

conexión
serialization
TTL
namespaces
errores
timeouts
recuperación.

Pero debemos mantener la regla:

Redis failure
      ↓
NO
      ↓
Authorization ALLOW
29. Test de Health Endpoints

Como TASK-001-01 ya creó la base de health checks, debemos probar:

GET /api/v1/health/live

y:

GET /api/v1/health/ready

Debe existir diferencia entre:

Liveness

y:

Readiness

Un fallo de una dependencia no debe hacer que /live falle automáticamente.

30. API Contract Foundation

Los tests API deberán comprobar el contrato estándar:

{
  "error": {
    "code": "...",
    "message": "...",
    "correlation_id": "..."
  }
}

Y posteriormente:

HTTP status
+
error code
+
response schema
+
headers

deberán formar parte del contrato.

31. Test isolation

Cada test debe ser independiente.

No debe existir:

test A
  ↓
modifica global
  ↓
test B depende de A

Todos los tests deben poder ejecutarse:

individualmente

y:

en cualquier orden

cuando sea técnicamente posible.

32. Parallel Testing

La estructura debe quedar preparada para ejecutar:

pytest

y posteriormente:

pytest -n auto

si el proyecto lo requiere.

Para ello:

evitar estado global mutable;
aislar fixtures;
aislar DB;
evitar puertos compartidos innecesarios;
generar datos independientes.
33. Test naming

Convención:

test_<behavior>()

Ejemplos:

def test_user_id_is_immutable():
    ...
async def test_uow_rolls_back_when_operation_fails():
    ...
async def test_correlation_id_is_returned_in_response():
    ...

Evitar:

test_user_1()
test_case_2()
test_something()
34. Test structure

Preferencia:

Arrange
Act
Assert

Ejemplo:

async def test_uow_rolls_back_when_operation_fails():
    # Arrange
    ...

    # Act
    ...

    # Assert
    ...

Esto mantiene las pruebas legibles.

35. Tests para TASK-001-01 → 19

TASK-001-20 debe validar la fundación creada hasta ahora.

Runtime
API starts
Health works
Error handler works
Correlation works
Shared Kernel
Entity
ValueObject
AggregateRoot
DomainEvent
Result
Repository
UnitOfWork
Events
Event immutability
Envelope mapping
Metadata propagation
No secret leakage
Infrastructure
PostgreSQL configuration
Redis configuration
health checks
failure handling
Context
TenantContext
ActorContext
TraceContext
Errors
mapping
sanitization
HTTP translation
Migration
migration configuration
revision discovery
database compatibility
36. Primer conjunto mínimo de tests

Al terminar TASK-001-20 deberíamos tener como mínimo tests para:

TEST-001
FastAPI application starts

TEST-002
/api/v1 route exists

TEST-003
live health endpoint works

TEST-004
ready health endpoint exists

TEST-005
correlation ID is generated

TEST-006
correlation ID is preserved

TEST-007
unexpected exception is sanitized

TEST-008
Entity equality works

TEST-009
Value Object is immutable

TEST-010
Aggregate records domain events

TEST-011
DomainEvent is immutable

TEST-012
Event envelope preserves correlation

TEST-013
Event envelope does not expose secrets

TEST-014
Repository contract is importable

TEST-015
UnitOfWork rollback works

TEST-016
Tenant context is isolated

TEST-017
Actor context is isolated

TEST-018
Trace context is isolated

TEST-019
Domain cannot import Infrastructure

TEST-020
Shared Kernel cannot import feature modules

Estos son foundation tests, no todavía los tests funcionales completos de Identity.

37. Test Context

Conviene establecer un concepto común:

@dataclass
class TestContext:
    tenant_id: UUID | None
    actor_id: UUID | None
    correlation_id: str
    trace_id: str

Sin embargo, no debemos mezclarlo con:

SecurityContext

ni convertirlo en un objeto global mutable.

38. Test configuration

Debe existir separación entre:

Development
Testing
Production

Por ejemplo:

.env
.env.test

pero los secretos reales nunca deben estar en Git.

Para tests:

TEST_DATABASE_URL
TEST_REDIS_URL

deben apuntar a infraestructura exclusiva de testing.

39. CI/CD

TASK-001-20 debe dejar preparado el comando:

pytest

y posteriormente:

pytest --cov=app

CI deberá poder ejecutar al menos:

install
  ↓
lint/type checks
  ↓
unit tests
  ↓
coverage

Los integration tests pueden ejecutarse en un job separado cuando la infraestructura esté disponible.

40. Quality Gate inicial

Para esta etapa:

❌ Test failure → pipeline failure

No debemos permitir:

pytest failure
     ↓
"continuemos igual"

Especialmente para:

architecture tests
security tests
critical tests
41. No implementar todavía

TASK-001-20 NO debe implementar:

❌ User CRUD
❌ Authentication
❌ Login
❌ JWT lifecycle completo
❌ Refresh Tokens
❌ Authorization Runtime
❌ RBAC
❌ ABAC
❌ MFA
❌ Policy Engine
❌ Delegation
❌ Membership business logic
❌ Audit business logic

Esas funcionalidades tendrán sus propios tests en las IS correspondientes.

42. Criterios de aceptación
AC-01

pytest puede ejecutarse desde el proyecto.

AC-02

Existe estructura separada para unit, integration, API y architecture tests.

AC-03

Los tests async funcionan correctamente.

AC-04

Existe configuración centralizada de pytest.

AC-05

Existen fixtures base.

AC-06

Existen factories base.

AC-07

Los tests pueden ejecutarse individualmente.

AC-08

Los tests foundation son deterministas.

AC-09

No se utilizan credenciales de producción.

AC-10

Existe configuración de coverage.

AC-11

Los tests de arquitectura pueden detectar dependencias prohibidas.

AC-12

Existe validación del Shared Kernel.

AC-13

Existe validación básica de Domain Events.

AC-14

Existe validación básica de UnitOfWork.

AC-15

Existe validación básica de Trace/Correlation Context.

AC-16

Existe validación básica de Tenant Context.

AC-17

Existe validación básica de Actor Context.

AC-18

Los errores inesperados no exponen información sensible.

AC-19

La base de testing está preparada para CI/CD.

AC-20

Un fallo de test no puede ser ignorado silenciosamente por el quality gate.

43. Definition of Done

TASK-001-20 estará terminado cuando:

[ ] pytest configurado
[ ] async testing configurado
[ ] estructura tests creada
[ ] fixtures base creadas
[ ] factories base creadas
[ ] unit tests foundation
[ ] integration test foundation
[ ] API test foundation
[ ] architecture tests
[ ] coverage configurado
[ ] secret-leak tests básicos
[ ] context tests
[ ] event tests
[ ] UoW tests
[ ] error mapping tests
[ ] CI compatible
[ ] documentación de testing

Y además:

[ ] pytest pasa
[ ] architecture tests pasan
[ ] foundation coverage cumple objetivo definido para esta etapa
[ ] no existen secretos de producción
[ ] no existen dependencias arquitectónicas prohibidas
44. Traceability
Blueprint
   ↓
ARCHITECTURE-MAP
   ↓
ESP-0001/13
   ↓
ESP-0001/14
   ↓
IS-001
   ↓
TASK-001-20
   ↓
Test Foundation

ADRs relacionados:

ADR-IDENTITY-006  PostgreSQL
ADR-IDENTITY-007  Events / Outbox
ADR-IDENTITY-009  Token Storage
ADR-IDENTITY-010  Testing & Quality
ADR-IDENTITY-014  Encryption
ADR-IDENTITY-016  Concurrency
ADR-IDENTITY-017  Migrations
ADR-IDENTITY-022  Consumer Idempotency
ADR-IDENTITY-028  Observability
ADR-IDENTITY-030  DR Testing
ADR-IDENTITY-045  Authorization Runtime
ADR-IDENTITY-049  Authorization Evaluation
ADR-IDENTITY-062  Golden Scenarios & Invariants
ADR-IDENTITY-063  Test Execution Engine
ADR-IDENTITY-064  Test Data & Evidence
ADR-IDENTITY-065  Schema Versioning
ADR-IDENTITY-066  Test Schema Registry
45. Resultado esperado

Al finalizar este Task tendremos algo muy importante:

                 EVOXA Identity
                       │
                ┌──────┴──────┐
                │ Test Engine │
                └──────┬──────┘
                       │
       ┌───────────────┼────────────────┐
       ↓               ↓                ↓
     Unit          Integration          API
       │               │                │
       └───────────────┼────────────────┘
                       ↓
                Architecture
                       │
                       ↓
                 Security

Y sobre esta base podremos implementar las siguientes funcionalidades sin tener que crear el sistema de testing desde cero.

Estado de IS-001
Task	Descripción	Estado
001-01	API Runtime Foundation	✅
001-02	Identity Domain Package	✅
001-03	Layer Architecture	✅
001-04	Shared Kernel	✅
001-05	Domain Events Foundation	✅
001-06	Repository Contracts	✅
001-07	Unit of Work	✅
001-08	PostgreSQL Infrastructure	✅
001-09	Redis Infrastructure	✅
001-10	Configuration & Secrets	✅
001-11	Security Foundation	✅
001-12	Tenant Context	✅
001-13	Actor Context	✅
001-14	Correlation & Trace Context	✅
001-15	Exception Model	✅
001-16	Dependency Injection	✅
001-17	Architecture Boundary Tests	⏳/integración con 20
001-18	Initial Observability	✅
001-19	Migration Framework	✅
001-20	Test Foundation	🟢 Ready
Próximo paso

TASK-001-21 — CI Foundation

Después de 001-21 todavía quedarán las tareas finales de infraestructura/gobernanza de IS-001, antes de cerrar formalmente Identity Domain Foundation y pasar a la siguiente Implementation Story.
