TASK-001-17 — Architecture Boundary Tests

Implementation Story: IS-001 — Identity Domain Foundation
ESP: ESP-0001/14 — Identity Implementation Requirements
Related ESP: ESP-0001/01, ESP-0001/02, ESP-0001/03, ESP-0001/05, ESP-0001/07, ESP-0001/09, ESP-0001/10, ESP-0001/11, ESP-0001/12, ESP-0001/13
Related ADRs: ADR-IDENTITY-001, 005, 006, 007, 027, 045, 047, 049, 067, 069
Priority: 🔴 Critical
Status: 🟢 Ready for Implementation
Task Type: Architecture Governance / Automated Quality

1. Objetivo

Crear una suite de Architecture Boundary Tests que compruebe automáticamente que la implementación de EVOXA Identity respeta las reglas arquitectónicas definidas en el Blueprint, ESP y ADRs.

La finalidad no es probar comportamiento funcional de usuarios, login, MFA, etc.

La finalidad es comprobar:

Que el código no pueda evolucionar accidentalmente rompiendo la arquitectura de EVOXA.

Por ejemplo, si mañana alguien implementa:

from app.core.infrastructure.database import ...

dentro de:

domains/identity/account/domain/

el código podría incluso funcionar, pero la arquitectura estaría rota.

El test debe detectarlo y hacer fallar el pipeline.

2. Principio fundamental

Los Architecture Boundary Tests son un Architecture Gate.

Developer
    │
    ▼
Source Code
    │
    ▼
Architecture Tests
    │
    ├── PASS → CI/CD continúa
    │
    └── FAIL → CI/CD bloqueado

Esto convierte las reglas arquitectónicas en restricciones ejecutables.

3. Arquitectura que debe protegerse

La arquitectura definida para Identity es:

                    ┌───────────────┐
                    │      API      │
                    └───────┬───────┘
                            │
                            ▼
                    ┌───────────────┐
                    │ Application   │
                    └───────┬───────┘
                            │
                            ▼
                    ┌───────────────┐
                    │    Domain     │
                    └───────────────┘
                            ▲
                            │
                    ┌───────┴───────┐
                    │ Infrastructure │
                    └───────────────┘

Importante:

La flecha de Infrastructure hacia Domain representa implementación de contratos definidos por capas internas.

No significa que Domain pueda importar Infrastructure.

4. Reglas arquitectónicas
AB-001 — Domain debe ser independiente

Domain NO puede importar:

FastAPI
SQLAlchemy
Redis
HTTP clients
PostgreSQL
Event Broker
Docker
Kubernetes
OpenTelemetry SDK

Ni ningún framework de infraestructura.

5. Regla AB-002 — Domain no depende de API

Debe fallar algo como:

from app.api.v1 import router

dentro de:

identity/*/domain/

La dependencia permitida es:

Domain
   ↓
Domain

y conceptos compartidos del Shared Kernel.

6. Regla AB-003 — Domain no depende de Infrastructure

Debe fallar:

from app.core.infrastructure.database import session

o:

from app.domains.identity.account.infrastructure import ...

desde Domain.

7. Regla AB-004 — Application no depende de Infrastructure concreta

Application puede depender de contratos:

from ...domain.repositories import UserRepository

pero no:

from ...infrastructure.repositories import PostgreSQLUserRepository

Esto protege el Dependency Inversion Principle.

8. Regla AB-005 — API no accede directamente a Infrastructure

Incorrecto:

API
 │
 └── PostgreSQLRepository

Correcto:

API
 │
 ▼
Application
 │
 ▼
Contract
 │
 ▼
Infrastructure

El API debe ejecutar casos de uso/servicios de Application.

9. Regla AB-006 — API no contiene lógica de negocio

Los Architecture Tests deberán detectar patrones arquitectónicos incorrectos cuando sea posible.

Por ejemplo:

@router.post("/users")
async def create_user(...):
    user = User(...)
    ...
    db.commit()

El endpoint no debería contener la implementación del negocio.

Debe ser conceptualmente:

@router.post("/users")
async def create_user(
    command: CreateUserCommand,
    use_case: CreateUserUseCase = Depends(...),
):
    return await use_case.execute(command)

El test estructural puede verificar que los routers no importen directamente ORM/repositories concretos.

10. Regla AB-007 — Infrastructure implementa contracts

La relación válida es:

Domain/Application Contract
            ▲
            │
            │ implements
            │
Infrastructure Adapter

Ejemplo:

class PostgreSQLUserRepository(UserRepository):
    ...

Esto debe ser permitido.

11. Regla AB-008 — Shared Kernel debe permanecer pequeño

identity/shared/ no puede convertirse en un depósito de funcionalidades.

No debe contener:

User
Organization
Membership
Role
Permission
Session
RefreshToken
MFA
Policy
Delegation
Audit
SecurityEvent

ni implementaciones de:

PostgreSQL
Redis
FastAPI
Event Broker

El Shared Kernel debe contener únicamente conceptos realmente transversales.

12. Regla AB-009 — No Cross-Module Infrastructure Access

Los módulos de Identity son:

account
organization
membership
authentication
authorization
role_permission
policy
session
token
mfa
recovery
federation
provisioning
audit
security_events
integration

Un módulo no puede saltarse sus contracts para acceder directamente a Infrastructure de otro módulo.

Incorrecto:

account/application
       │
       ▼
membership/infrastructure

Correcto:

account/application
       │
       ▼
Membership Contract
13. Regla AB-010 — No Cross-Module Database Access

Ningún módulo puede acceder directamente a las tablas/repositorios internos de otro módulo.

Por ejemplo:

Account
   │
   └── SELECT membership table

está prohibido.

Debe utilizarse un contrato:

Account
   │
   ▼
MembershipQuery / Contract
   │
   ▼
Membership

Esto sigue directamente las reglas de límites de módulos de Identity.

14. Regla AB-011 — Authorization centralizada

Los módulos de negocio no pueden implementar sus propios mecanismos independientes de autorización.

No queremos:

if user.role == "admin":
    ...

disperso por los módulos.

La autorización debe pasar por:

Authorization Runtime
        │
        ▼
Policy / RBAC / ABAC / Ownership / Delegation
        │
        ▼
Decision

Los Architecture Tests deberán empezar protegiendo esta frontera aunque el Runtime completo todavía no exista.

15. Regla AB-012 — No Authorization Bypass

Debe bloquearse cualquier patrón que intente establecer:

ALLOW

directamente desde:

API
Application
Domain
AI Agent

sin pasar por el mecanismo de autorización definido.

Esto no significa buscar literalmente la palabra ALLOW en todo el código.

El test debe evolucionar hacia reglas estructurales conforme se implementen los módulos.

16. Regla AB-013 — Tenant Isolation

Los componentes tenant-aware deben recibir explícitamente el contexto necesario.

Debe evitarse un patrón como:

repository.get_all_users()

cuando el recurso es tenant-scoped.

El diseño debe favorecer:

repository.list_for_organization(
    organization_id
)

Los Architecture Tests iniciales podrán validar que los contratos de los recursos tenant-scoped exijan el contexto apropiado.

17. Regla AB-014 — Security Context ≠ Tenant Context

No se deben fusionar artificialmente:

ActorContext
TenantContext
TraceContext
SecurityContext
IdempotencyContext

Cada uno tiene una responsabilidad diferente.

Esto evita que un contexto transversal termine convirtiéndose en un mecanismo oculto de autorización.

18. Regla AB-015 — Domain no conoce FastAPI

No debe aparecer:

from fastapi import Depends

en Domain.

FastAPI pertenece a API/Presentation.

19. Regla AB-016 — Domain no conoce SQLAlchemy

No debe aparecer:

from sqlalchemy import ...

en Domain.

Tampoco:

from sqlalchemy.ext.asyncio import ...
20. Regla AB-017 — Domain no conoce Redis

No:

import redis

dentro de Domain.

Redis pertenece a Infrastructure.

21. Regla AB-018 — No secretos en Domain

Architecture Tests pueden incluir búsquedas estáticas para detectar:

password=
JWT_SECRET=
PRIVATE_KEY=
API_KEY=
REDIS_PASSWORD=
DATABASE_PASSWORD=

No se debe intentar resolver secretos dentro de Domain.

22. Regla AB-019 — No estado global mutable

Debe evitarse:

GLOBAL_SESSION = ...

o:

GLOBAL_TENANT = ...

o:

GLOBAL_ACTOR = ...

Esto es especialmente importante para una API async/concurrente.

23. Regla AB-020 — No Service Locator

Debe bloquearse el patrón:

container.get("user_repository")

desde Domain/Application como mecanismo oculto de dependencia.

Las dependencias deben aparecer explícitamente:

def __init__(
    self,
    user_repository: UserRepository,
):
24. Estructura de tests

Propuesta:

apps/api/tests/
└── architecture/
    ├── __init__.py
    ├── test_layer_boundaries.py
    ├── test_domain_dependencies.py
    ├── test_application_dependencies.py
    ├── test_api_boundaries.py
    ├── test_module_boundaries.py
    ├── test_shared_kernel.py
    ├── test_infrastructure_boundaries.py
    ├── test_security_boundaries.py
    ├── test_tenant_boundaries.py
    └── test_dependency_injection_boundaries.py
25. Herramienta de análisis

La implementación puede utilizar una combinación de:

Python AST
importlib
package metadata
pytest

La recomendación es utilizar AST/static analysis para las reglas de importación.

No debemos depender de ejecutar toda la aplicación para detectar una violación arquitectónica.

26. Ejemplo de test de imports

Conceptualmente:

from pathlib import Path
import ast


def collect_imports(path: Path):
    tree = ast.parse(path.read_text())

    imports = []

    for node in ast.walk(tree):
        if isinstance(node, ast.Import):
            imports.extend(
                alias.name
                for alias in node.names
            )

        elif isinstance(node, ast.ImportFrom):
            if node.module:
                imports.append(node.module)

    return imports

Después:

FORBIDDEN_DOMAIN_IMPORTS = {
    "fastapi",
    "sqlalchemy",
    "redis",
}

Y se verifica que ningún archivo Domain importe esos paquetes.

27. Architecture Rule Engine

A medida que crezca el proyecto, conviene evitar tests completamente dispersos.

Podemos establecer:

tests/architecture/
│
├── rules/
│   ├── layers.py
│   ├── modules.py
│   ├── security.py
│   └── dependencies.py
│
├── fixtures/
│
└── test_architecture.py

Conceptualmente:

ArchitectureRule(
    id="AB-003",
    name="Domain cannot import Infrastructure",
    source="domain",
    forbidden={"infrastructure"},
)

Esto nos permitirá hacer crecer las reglas sin convertir cada test en código aislado.

28. Matriz de dependencias

La matriz inicial será:

Desde	Puede depender de	Prohibido
Domain	Domain / Shared	API / Application / Infrastructure
Application	Domain / Shared / Contracts	API / Infrastructure concreta
API	Application / Domain DTOs / Shared	DB / Redis / Infrastructure directa
Infrastructure	Domain / Application contracts / Shared	API como dependencia arquitectónica
Shared	Solo conceptos transversales	módulos concretos

Esta matriz se convierte en una regla automatizada.

29. Module Dependency Matrix

También debemos proteger la regla establecida en ADR-069:

Module A
   │
   ├── SYNC
   ├── QUERY
   ├── EVENT
   └── PUBLISH

pero no:

Module A
   │
   └── direct DB access
             │
             ▼
        Module B tables

Los tipos de interacción permitidos serán posteriormente:

SYNC
QUERY
EVENT
PUBLISH

y los prohibidos:

FORBIDDEN
DIRECT_TABLE_ACCESS
DIRECT_INFRASTRUCTURE_ACCESS
30. Architecture Test: Layer Boundary

Ejemplo conceptual:

def test_domain_does_not_import_infrastructure():
    violations = find_import_violations(
        domain_path,
        forbidden=["infrastructure"],
    )

    assert not violations, violations

Resultado esperado:

PASSED

Si alguien rompe la arquitectura:

FAILED

AB-003
Domain module imported Infrastructure

File:
identity/account/domain/entities.py

Import:
app.core.infrastructure.database

Esto es exactamente el tipo de feedback que queremos.

31. Architecture Test: API Boundary
def test_api_does_not_import_database():
    violations = find_import_violations(
        api_path,
        forbidden=[
            "sqlalchemy",
            "asyncpg",
            "redis",
        ],
    )

    assert not violations
32. Architecture Test: Shared Kernel

Podremos definir una allowlist.

ALLOWED_SHARED_MODULES = {
    "entity",
    "aggregate",
    "value_object",
    "identifiers",
    "events",
    "exceptions",
    "repositories",
    "ports",
    "result",
}

Si aparece:

identity/shared/domain/user.py

el architecture test puede fallar.

33. Architecture Test: Infrastructure Direction

Debe permitirse:

Infrastructure → Domain contracts

pero no:

Domain → Infrastructure

El test debe distinguir la dirección.

34. Architecture Test: Circular Dependencies

También debemos detectar ciclos:

A → B
B → C
C → A

Especialmente:

Domain ↔ Application
Application ↔ Infrastructure
Module A ↔ Module B

Los ciclos deben considerarse una violación arquitectónica salvo excepciones explícitamente documentadas.

35. Architecture Test: Cross-Module Imports

Inicialmente debemos permitir imports hacia contratos públicos.

Por ejemplo:

membership.application
        ↓
account.application.contracts

pero no:

membership
   ↓
account.infrastructure.repositories

La frontera debe estar en el contrato público del módulo.

36. Architecture Test: ORM Boundary

El ORM debe estar confinado a Infrastructure.

Inicialmente:

SQLAlchemy
    ↓
Infrastructure

No:

SQLAlchemy
   ↓
Domain

ni:

SQLAlchemy
   ↓
Application
37. Architecture Test: Redis Boundary

Igualmente:

Redis client
     ↓
Infrastructure

Los módulos superiores reciben ports/interfaces.

38. Architecture Test: Event Platform Boundary

El Domain puede crear Domain Events.

Pero no debe:

broker.publish(...)

El publisher pertenece a Infrastructure/Application Integration.

La arquitectura:

Domain Event
    ↓
UnitOfWork
    ↓
Outbox
    ↓
Publisher
    ↓
Event Platform

debe permanecer intacta.

39. Architecture Test: DI Boundary

Como acabamos de definir en TASK-001-16:

Debe fallar:

Domain → Container
Application → Global Container

Debe ser:

Composition Root
       ↓
Dependency Injection
       ↓
Application
40. Architecture Test: Observability Boundary

Domain no debería importar directamente:

opentelemetry

o:

structlog

si esas tecnologías pertenecen a Infrastructure/Core.

El Domain puede emitir conceptos propios; la infraestructura decide cómo observarlos.

41. Architecture Test: Error Boundary

De acuerdo con TASK-001-15:

Domain no debe importar:

from fastapi import HTTPException

Los errores deben seguir:

Domain Error
      ↓
Application
      ↓
API Mapper
      ↓
HTTP Response
42. Architecture Test: Configuration Boundary

Domain no debe leer directamente:

os.getenv(...)

ni:

settings.database_password

La configuración debe ser inyectada donde sea necesaria.

43. Architecture Test: Tenant Boundary

Los tests deben verificar progresivamente que las operaciones tenant-scoped no introduzcan APIs ambiguas como:

get_user(...)

cuando existe riesgo de confundir:

global user
vs
tenant membership

No significa que todos los métodos tengan que recibir tenant_id; el modelo global User + Membership requiere distinguir claramente ambos conceptos.

44. Architecture Test: Security Boundary

Nunca debe permitirse que un módulo introduzca:

local authorization engine
local RBAC
local permission evaluator

que compita con Identity Authorization.

La regla será:

Identity Authorization es el único punto central de evaluación de autorización.

45. Tests negativos

Esta suite debe contener deliberadamente ejemplos de violaciones.

Por ejemplo, fixtures:

tests/architecture/fixtures/
└── invalid/
    ├── domain_imports_fastapi.py
    ├── domain_imports_infrastructure.py
    ├── application_imports_postgres.py
    ├── api_imports_sqlalchemy.py
    ├── module_cross_infrastructure.py
    └── global_service_locator.py

Esto permite probar que el Architecture Test realmente detecta una violación.

46. Golden Architecture Rules

Las reglas críticas deben considerarse Golden Architecture Rules.

Por ejemplo:

GAR-001 Domain independence
GAR-002 Layer direction
GAR-003 No cross-module DB access
GAR-004 Centralized authorization
GAR-005 Tenant isolation
GAR-006 No secret leakage
GAR-007 No Infrastructure in Application
GAR-008 No Infrastructure in Domain
GAR-009 No global mutable security context
GAR-010 No Service Locator

Estas reglas no deben modificarse silenciosamente.

47. CI/CD Integration

Los Architecture Tests deberán ejecutarse automáticamente:

Pull Request
     │
     ├── Unit Tests
     ├── Architecture Tests
     ├── Security Tests
     └── Contract Tests
             │
             ▼
        Quality Gate

Si falla:

❌ Architecture Gate FAILED

el PR no debe poder considerarse válido.

48. Severity

Clasificación inicial:

P0 — Critical
cross-tenant boundary
authorization bypass
Domain → Infrastructure
secret exposure
security boundary violation
P1 — High
Application → Infrastructure concrete
cross-module DB access
circular dependency
API → DB
P2 — Medium
shared kernel pollution
observability boundary
configuration boundary
49. Qué NO hará esta tarea

TASK-001-17 no implementará:

Authorization Runtime;
Policy Engine;
RBAC;
ABAC;
MFA;
JWT;
Sessions;
Refresh Tokens;
PostgreSQL repositories de negocio;
Redis cache;
Event Broker;
microservices.

Esta tarea crea los guardianes arquitectónicos para que esas implementaciones futuras respeten el diseño.

50. Acceptance Criteria
AC-01

Given el código Domain,
When se ejecutan los Architecture Tests,
Then no puede importar Infrastructure.

AC-02

Given el código Domain,
When se ejecutan los tests,
Then no puede importar FastAPI, ORM ni Redis.

AC-03

Given Application,
When se analiza su árbol de imports,
Then no puede importar implementaciones concretas de Infrastructure.

AC-04

Given API,
When se ejecutan los tests,
Then no puede acceder directamente a PostgreSQL/Redis.

AC-05

Given Infrastructure,
When implementa un contract,
Then la dependencia es considerada válida.

AC-06

Given dos módulos Identity,
When uno intenta acceder directamente a Infrastructure del otro,
Then el Architecture Test falla.

AC-07

Given un módulo,
When intenta acceder directamente a tablas internas de otro módulo,
Then el Architecture Test falla.

AC-08

Given Shared Kernel,
When se agrega una dependencia de un módulo concreto,
Then el test falla.

AC-09

Given una dependencia circular,
When se ejecuta el análisis arquitectónico,
Then el test falla.

AC-10

Given Application,
When intenta utilizar un Service Locator global,
Then el Architecture Test falla.

AC-11

Given Domain,
When intenta utilizar FastAPI HTTPException,
Then el test falla.

AC-12

Given Domain,
When intenta acceder a configuración mediante os.getenv,
Then el test falla.

AC-13

Given una violación deliberada en una fixture,
When se ejecutan los tests,
Then la violación es detectada.

AC-14

Given una modificación arquitectónicamente válida,
When se ejecutan los tests,
Then el pipeline continúa.

AC-15

Given un Pull Request,
When existe una violación P0/P1,
Then el Architecture Gate debe bloquear el cambio.

51. Definition of Done
 Architecture test framework creado.
 Layer boundaries automatizados.
 Domain dependency rules implementadas.
 Application dependency rules implementadas.
 API boundaries implementados.
 Cross-module boundaries implementados.
 Shared Kernel rules implementadas.
 Infrastructure boundaries implementados.
 DI boundaries implementados.
 Security boundaries iniciales implementados.
 Tenant boundaries iniciales implementados.
 Circular dependency detection implementada.
 Negative fixtures implementadas.
 Golden Architecture Rules documentadas.
 Severity definida.
 CI/CD integration preparada.
 Tests deterministas.
 Tests reproducibles.
 Documentación actualizada.
 Traceability registrada.
52. Traceability
Blueprint
    │
    ▼
Architecture Map
    │
    ▼
ESP-0001
    │
    ├── ESP-0001/01 Domain Model
    ├── ESP-0001/02 Architecture
    ├── ESP-0001/03 Authentication & Authorization
    ├── ESP-0001/05 Roles & Permissions
    ├── ESP-0001/10 API Contracts
    ├── ESP-0001/11 Persistence
    ├── ESP-0001/12 Events
    ├── ESP-0001/13 Testing
    └── ESP-0001/14 Implementation
    │
    ▼
ADR-067 Identity Module Boundaries
ADR-069 Module Dependency Matrix
    │
    ▼
IS-001
    │
    ▼
TASK-001-17
    │
    ▼
Architecture Boundary Tests
    │
    ▼
CI/CD Architecture Gate
53. Resultado esperado

Al terminar esta tarea tendremos algo muy importante para EVOXA:

                 EVOXA Architecture
                         │
                         ▼
                Architecture Rules
                         │
                         ▼
                Automated Tests
                         │
             ┌───────────┴───────────┐
             ▼                       ▼
          VALID                    INVALID
             │                       │
             ▼                       ▼
        CI CONTINUES              CI BLOCKED

Es decir, la arquitectura deja de ser solamente documentación y pasa a ser código ejecutable.

Esto será especialmente importante cuando empecemos a implementar Authentication, Authorization, MFA, Policies, Sessions y Tokens, porque evitaremos que cada nuevo módulo cree su propia interpretación de la arquitectura.

Estado de IS-001
TASK-001-01  API Runtime Foundation             ✅
TASK-001-02  Identity Domain Package             ✅
TASK-001-03  Layer Architecture                  ✅
TASK-001-04  Identity Shared Kernel              ✅
TASK-001-05  Domain Events Foundation             ✅
TASK-001-06  Repository Contracts                ✅
TASK-001-07  Unit of Work                        ✅
TASK-001-08  PostgreSQL Infrastructure            ✅
TASK-001-09  Redis Infrastructure                 ✅
TASK-001-10  Configuration & Secrets              ✅
TASK-001-11  Security Foundation                  ✅
TASK-001-12  Tenant Context Foundation            ✅
TASK-001-13  Actor Context                        ✅
TASK-001-14  Correlation & Trace Context          ✅
TASK-001-15  Exception Model                      ✅
TASK-001-16  Dependency Injection                 ✅
TASK-001-17  Architecture Boundary Tests          🟢
Siguiente tarea

TASK-001-18 — Initial Observability

Aquí estableceremos la primera capa real de logs estructurados, métricas, trazabilidad operacional y correlation/trace context, manteniendo estrictamente separadas Application Logs, Audit Events, Security Events y Distributed Tracing.
