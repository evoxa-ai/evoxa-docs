ESP-0001/14 — Identity Implementation Requirements

Document ID: ESP-0001/14
Domain: Identity
Version: 1.1.0
Status: Draft — Consolidated
Parent: ESP-0001 — Identity Engineering Specification
Repository: evoxa-platform
Implementation Path: apps/api/app/domains/identity/

Este documento es especialmente importante porque ya no define solamente qué debe hacer Identity, sino cómo debe organizarse la implementación sin saltarse la arquitectura.

El Architecture Map establece que EVOXA debe seguir:

Vision
  ↓
Blueprint
  ↓
Reference Architecture
  ↓
Engineering Standards
  ↓
ESP
  ↓
Implementation Stories
  ↓
Technical Tasks
  ↓
Source Code
  ↓
Testing
  ↓
Deployment

y explícitamente indica que ninguna implementación debe saltarse este lifecycle.

Además, el código de Identity está destinado a apps/api/app/domains/identity/.

1. Purpose

ESP-0001/14 define los requisitos técnicos que deben cumplirse antes y durante la implementación de Identity.

Su objetivo es convertir las decisiones de:

Blueprint;
Reference Architecture;
Engineering Standards;
ESP-0001;
ADR-001 → ADR-069;
Implementation Stories;

en una estructura concreta de software.

La implementación debe preservar:

Domain-Driven Design;
Modular Architecture;
separación de capas;
seguridad;
tenant isolation;
testability;
observability;
evolvability;
traceability.

Identity es una Platform Capability, y las capacidades de plataforma deben exponer contratos estables y mantenerse independientes de la lógica de negocio de los dominios consumidores.

2. Implementation Principles

La implementación debe seguir:

IMP-001 — Architecture First

No implementar directamente desde una idea o requerimiento informal.

Requirement
    ↓
ESP
    ↓
IS
    ↓
Technical Task
    ↓
Code
IMP-002 — Domain Ownership

Cada módulo debe ser propietario de su lógica y datos.

IMP-003 — Explicit Contracts

Los módulos se comunican mediante contratos explícitos.

IMP-004 — No Cross-Module Database Access

Un módulo no debe acceder directamente a tablas/repositorios internos de otro módulo.

IMP-005 — Authorization Centralization

La autorización debe pasar por el Authorization Runtime central.

IMP-006 — Fail Closed

Los errores de seguridad nunca deben convertirse en ALLOW.

IMP-007 — Tenant Isolation

Toda operación relevante debe tener contexto de tenant explícito.

IMP-008 — Testability

Todo componente crítico debe poder probarse aisladamente.

IMP-009 — Observability

Las operaciones críticas deben producir señales operacionales y de seguridad.

IMP-010 — Traceability

Todo código importante debe poder relacionarse con una Implementation Story y su ESP.

Esto sigue la regla general del repositorio: source code shall always trace back to an ESP.

3. Technology Baseline

El baseline actual de la arquitectura define:

Área	Tecnología
Backend	Python / FastAPI
Database	PostgreSQL
Cache	Redis
Messaging	RabbitMQ / Kafka — futuro
Containers	Docker
Orchestration	Kubernetes
CI/CD	GitHub Actions

Importante:

estas tecnologías son el baseline actual, no deben interpretarse como decisiones arquitectónicas inmutables.

La arquitectura permite evolucionarlas sin alterar los principios arquitectónicos.

4. Architectural Style

Identity deberá comenzar como:

Modular Monolith.

No se dividirá inicialmente en múltiples microservicios.

                    Identity
                       │
        ┌──────────────┼──────────────┐
        │              │              │
      Account      Organization    Membership
        │              │              │
        ├──────────────┼──────────────┤
        │              │              │
 Authentication    Authorization    Policy
        │              │              │
      Session       Token            MFA
        │              │              │
        └──────────────┼──────────────┘
                       │
                    Events

La extracción posterior a servicios independientes deberá basarse en evidencia técnica, operacional y de escala; no simplemente en una preferencia arquitectónica.

5. Layered Architecture

La implementación deberá respetar:

Presentation
      ↓
API
      ↓
Application
      ↓
Domain
      ↓
Infrastructure

Esta es la estructura oficial de capas definida en Architecture Map.

6. Presentation Layer

Identity no debe contener lógica de dominio en Presentation.

Responsabilidades:

serialización;
request handling;
response formatting;
API models;
dependency injection;
authentication extraction;
correlation context.

No debe:

evaluar permisos directamente;
acceder a PostgreSQL directamente;
modificar entidades de dominio;
implementar políticas de seguridad.
7. API Layer

La API debe contener:

routers
controllers
request schemas
response schemas
dependencies
exception mapping

Debe convertir:

HTTP
 ↓
Application Command / Query

y posteriormente:

Application Result
 ↓
HTTP Response
8. Application Layer

La Application Layer implementará los casos de uso.

Ejemplos:

CreateUser
AuthenticateUser
CreateOrganization
CreateMembership
AssignRole
RemovePermission
AuthorizeRequest
CreateSession
RefreshSession
RevokeSession
EnrollMFA
VerifyMFA
ActivatePolicy

La Application Layer coordina el caso de uso.

No debe convertirse en un segundo Domain Layer.

9. Domain Layer

Aquí reside la lógica de negocio y seguridad propia de Identity.

Debe contener:

entities
aggregates
value_objects
domain_services
domain_events
policies
rules
repositories interfaces

El Domain Layer no deberá depender de:

FastAPI;
PostgreSQL;
Redis;
HTTP;
Kafka/RabbitMQ;
proveedores externos.
10. Infrastructure Layer

Implementará:

PostgreSQL
Redis
Event Platform
KMS / Secret Manager
Email providers
MFA providers
Identity providers
repositories
publishers
external adapters

Infrastructure implementa interfaces definidas por capas superiores.

11. Identity Module Structure

La estructura inicial propuesta:

apps/api/app/domains/identity/
│
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
│
├── shared/
│
└── __init__.py

Los módulos corresponden al modelo consolidado de Identity.

12. Module Responsibilities
Account

Responsable de:

User;
account lifecycle;
credentials;
account security state.
Organization

Responsable de:

Organization;
tenant boundary;
organization lifecycle.
Membership

Responsable de:

user ↔ organization;
membership lifecycle;
membership roles.
Authentication

Responsable de:

login;
credential verification;
authentication methods;
authentication context.
Authorization

Responsable de:

authorization requests;
Authorization Runtime;
RBAC;
ABAC;
ownership;
delegation;
policy composition.
Role & Permission

Responsable de:

roles;
permissions;
assignments;
effective permissions.
Policy

Responsable de:

Security Policies;
policy lifecycle;
policy validation;
policy versions.
Session

Responsable de:

sessions;
revocation;
lifecycle.
Token

Responsable de:

access tokens;
refresh tokens;
token families;
reuse detection.
MFA

Responsable de:

factors;
challenges;
recovery codes;
trusted devices;
step-up evidence.
Recovery

Responsable de:

credential recovery;
account recovery;
controlled recovery workflows.
Federation

Responsable de:

OIDC;
SAML;
external identities.
Provisioning

Responsable de:

JIT;
invitation;
provisioning;
deprovisioning.
Audit

Responsable de audit evidence.

Security Events

Responsable de security event lifecycle.

Integration

Responsable de external/public integration contracts.

13. Domain Aggregates

La implementación deberá respetar los aggregates definidos por la arquitectura.

Principales:

User
Organization
Membership
Session
RefreshTokenFamily
MFAFactor
MFAChallenge
SecurityPolicy
Role
Permission
Delegation
AuditEvent
SecurityEvent
OutboxEvent

No se deberá crear una entidad simplemente porque exista una tabla.

Database tables no definen automáticamente Domain Aggregates.

14. Repository Pattern

Los repositories deberán definirse desde Domain/Application mediante interfaces.

Ejemplo conceptual:

Domain
  │
  └── UserRepository
          ▲
          │ implements
          │
Infrastructure
  │
  └── PostgreSQLUserRepository

El Domain no debe importar:

SQLAlchemy
asyncpg
Redis
FastAPI

directamente.

15. Persistence Boundary

PostgreSQL es la autoridad transaccional.

Redis:

cache
temporary state
rate limiting
performance optimization

No será fuente de verdad para:

users;
memberships;
roles;
permissions;
security policies;
sessions críticas;
revocations críticas.
16. Transaction Boundaries

Las operaciones críticas deberán definir explícitamente su frontera transaccional.

Ejemplo:

Role Revocation
      │
      ├── Update PostgreSQL
      ├── Increment security version/epoch
      └── Create Outbox Event
             │
             └── SAME TRANSACTION

Después:

Outbox
  ↓
Event Platform
  ↓
Consumers
17. Authorization Runtime

El Authorization Runtime será un componente central.

Debe implementar conceptualmente:

Authorization Request
        ↓
Request Validation
        ↓
Actor/Auth Validation
        ↓
Tenant Validation
        ↓
Context Construction
        ↓
Policy Selection
        ↓
Policy Evaluation
        ↓
Mandatory Security Guards
        ↓
RBAC
        ↓
Ownership
        ↓
ABAC
        ↓
Delegation
        ↓
Policy Composition
        ↓
Conflict Resolution
        ↓
Criticality
        ↓
Freshness
        ↓
Cache Eligibility
        ↓
Decision

No se permite implementar autorización de forma independiente dentro de cada dominio.

18. Authorization API Boundary

Los dominios consumidores deberán utilizar:

Identity.Authorize(...)

o el contrato equivalente definido por Identity.

No deberán implementar:

if user.role == ...

como mecanismo de autorización propio.

Tampoco:

if user.organization_id == ...

como sustituto del Authorization Runtime.

19. Criticality Engine

Criticality deberá ejecutarse antes de determinar cache eligibility.

Conceptualmente:

Base Criticality
        +
Risk
        +
Context
        +
Security State
        +
Tenant
        +
Delegation
        +
MFA
        +
Platform Minimum
        ↓
Effective Criticality

con:

MAX(...)

La implementación no podrá reducir una criticality superior por una condición secundaria.

20. Freshness Engine

La implementación deberá soportar:

F0
F1
F2
F3

y validar:

security epoch;
policy version;
entity version;
context fingerprint;
TTL;
invalidation state;
cache generation.

Critical operations deberán poder exigir estado fresco.

21. Cache Integration

El cache será una optimización.

PostgreSQL
    ↓
Authoritative State
    ↓
Version / Epoch
    ↓
Cache

Nunca:

Redis
 ↓
Authorization Authority

El Authorization Runtime seguirá siendo el punto de decisión.

22. Security Epochs & Versions

Los cambios de seguridad deberán poder incrementar versiones/epochs relevantes.

Ejemplos:

Membership revoked
      ↓
Security version/epoch change
      ↓
Cached authorization becomes invalid

Lo mismo aplica a:

policy;
role;
permission;
delegation;
session;
MFA/security state.
23. Token Implementation

Access Token:

short-lived;
JWT;
asymmetric signing;
kid;
JWKS;
claims mínimos.

Refresh Token:

opaque;
stateful;
cryptographically random;
protected storage;
rotation;
token family;
reuse detection.

La rotación debe ser concurrency-safe.

24. MFA Implementation

Debe existir separación entre:

MFAFactor
MFAChallenge

Ejemplo:

Factor
   ↓
Challenge
   ↓
Verification
   ↓
Evidence
   ↓
Authentication / Authorization Context

El Policy Engine determina cuándo MFA es requerido.

MFA no decide por sí mismo la autorización.

25. Security Policy Implementation

Policy deberá implementarse como estructura declarativa.

No se permitirá:

eval(user_supplied_code)

ni:

dynamic SQL

ni scripts arbitrarios para authorization.

Las policies deben ser:

typed;
structured;
versioned;
deterministic;
auditable.
26. API Contract Implementation

La API pública debe estar bajo:

/api/v1

Los contratos de Identity deberán implementar los endpoints establecidos en ESP-0001/10.

Ejemplo:

/api/v1/auth/*
/api/v1/users/*
/api/v1/organizations/*
/api/v1/memberships/*
/api/v1/roles/*
/api/v1/permissions/*
/api/v1/sessions/*
/api/v1/mfa/*
/api/v1/authorization/*
/api/v1/policies/*

No deben aparecer rutas internas de infraestructura como API pública.

27. Error Handling

Los errores deberán utilizar el contrato estándar:

{
  "error": {
    "code": "...",
    "message": "...",
    "correlation_id": "..."
  }
}

No exponer:

stack traces;
SQL;
secrets;
token values;
internal security state innecesario.
28. Idempotency

Las operaciones retryable deberán soportar:

Idempotency-Key

Debe cumplirse:

same key
+
same request
=
same logical result

pero:

same key
+
different request
=
IDEMPOTENCY_CONFLICT

La implementación debe soportar concurrencia sobre la misma key.

29. Event Implementation

Los eventos deberán ser:

immutable;
versioned;
tenant-aware;
idempotent-consumer friendly.

Envelope conceptual:

event_id
event_type
event_version
timestamp
tenant_id
actor
resource
payload
metadata
correlation_id
causation_id
trace_id

Nunca incluir:

password
refresh token
MFA secret
private key
API secret
30. Outbox Implementation

El patrón obligatorio:

Domain Change
     ↓
Database Transaction
     ├── State Change
     └── Outbox Event
             ↓
          Commit
             ↓
      Publisher Worker
             ↓
       Event Platform

Nunca depender de:

DB commit
   ↓
HTTP call
   ↓
Publish

como mecanismo de consistencia.

31. Event Consumers

Los consumers deberán ser:

idempotent;
retry-safe;
tenant-aware;
version-aware.

Deben soportar:

duplicate
retry
out-of-order
replay
consumer crash
DLQ
32. Cache Invalidation

Los eventos de cambio de seguridad serán hechos de dominio/integración.

No deben convertirse en comandos como:

DELETE_REDIS_KEY

La lógica será:

Security Fact
     ↓
Event Platform
     ↓
Invalidation Consumer
     ↓
Version / Epoch
     ↓
Cache Reconciliation

El consumer no será el Authorization Engine.

33. Security Boundaries

Los componentes críticos deben tener permisos mínimos.

Por ejemplo:

API
 ↓
Application
 ↓
Domain

y no:

API
 ↓
PostgreSQL superuser

Los service accounts deberán utilizar identidad S2S y scopes mínimos.

34. Secrets

Secrets deberán obtenerse desde infraestructura externa:

Secret Manager
KMS/HSM
Workload Identity

No deberán almacenarse en:

Git
.env committed
Docker image
Source code
logs
events
test artifacts

La configuración debe ser externa al código, alineada con las dependencias de arquitectura.

35. Observability

Cada operación importante debe soportar:

correlation_id
trace_id
causation_id

Logs estructurados.

Debe existir observabilidad para:

authentication;
authorization;
policy evaluation;
MFA;
refresh reuse;
security events;
cache invalidation;
event publication;
failures.
36. Audit

Las acciones críticas deberán producir Audit Evidence.

Ejemplos:

User created
Membership revoked
Role assigned
Permission revoked
Policy activated
Session revoked
Refresh reuse detected
MFA factor revoked
Delegation created
Cross-tenant access denied

Audit no debe depender exclusivamente de application logs.

37. Test Requirements

Cada componente implementado deberá incluir pruebas.

Mínimo:

Unit
Component
Integration
API
Security

Los componentes críticos adicionalmente:

Property
Mutation
Golden
Regression
Chaos

Los requisitos detallados se encuentran en ESP-0001/13.

38. Implementation Story Traceability

Cada implementación debe indicar:

ESP
 ↓
ADR
 ↓
IS
 ↓
Technical Task
 ↓
Code
 ↓
Tests

Ejemplo:

ESP-0001/03
    ↓
ADR-049
    ↓
IS-006
    ↓
TASK-AUTHZ-001
    ↓
authorization/runtime/
    ↓
TST-AUTHZ-001
39. Source Code Traceability

Se recomienda que cada módulo mantenga metadata/documentación que permita localizar:

Implementation Story
Technical Task
ADR
ESP

No significa llenar el código de comentarios innecesarios.

La trazabilidad debe mantenerse principalmente mediante:

estructura;
documentación;
naming;
commits;
PRs;
tests;
CI metadata.
40. Proposed Source Structure

La implementación inicial puede seguir:

apps/api/
│
├── app/
│   ├── main.py
│   │
│   ├── api/
│   │   └── v1/
│   │
│   ├── core/
│   │   ├── config/
│   │   ├── security/
│   │   ├── observability/
│   │   └── infrastructure/
│   │
│   └── domains/
│       └── identity/
│           │
│           ├── account/
│           ├── organization/
│           ├── membership/
│           ├── authentication/
│           ├── authorization/
│           ├── role_permission/
│           ├── policy/
│           ├── session/
│           ├── token/
│           ├── mfa/
│           ├── recovery/
│           ├── federation/
│           ├── provisioning/
│           ├── audit/
│           ├── security_events/
│           ├── integration/
│           └── shared/
│
└── tests/
    └── domains/
        └── identity/

Esta estructura es una implementación propuesta derivada de la arquitectura, no una ruta que el Blueprint actual haya congelado literalmente.

41. Module Internal Structure

Cada módulo puede utilizar:

module/
├── domain/
│   ├── entities/
│   ├── value_objects/
│   ├── services/
│   ├── events/
│   ├── policies/
│   └── repositories/
│
├── application/
│   ├── commands/
│   ├── queries/
│   ├── services/
│   └── dto/
│
├── infrastructure/
│   ├── persistence/
│   ├── cache/
│   └── adapters/
│
└── api/
    ├── routes/
    ├── schemas/
    └── dependencies/
42. Shared Kernel

identity/shared/ debe mantenerse pequeño.

Puede contener únicamente conceptos verdaderamente compartidos:

TenantContext
ActorContext
CorrelationContext
Domain primitives
Common errors
Common identifiers

No debe convertirse en:

misc/
utils/
everything/
43. Dependency Rules

Regla:

API
 ↓
Application
 ↓
Domain

Infrastructure implementa interfaces.

Nunca:

Domain → API
Domain → FastAPI
Domain → PostgreSQL
Domain → Redis

Y tampoco:

Account → Membership database

directamente.

Debe utilizar contratos.

44. Inter-Module Communication

Comunicación síncrona:

Public Application Contract

Comunicación asíncrona:

Domain Event
Integration Event

No utilizar eventos como sustituto de una transacción que realmente necesita consistencia inmediata.

45. Forbidden Patterns

No se permitirá:

❌ SQL dentro de Domain
❌ Redis como source of truth
❌ Authorization duplicada
❌ Cross-module table access
❌ Global mutable authorization state
❌ eval() para policies
❌ secrets en código
❌ secrets en logs
❌ tokens en URLs
❌ refresh token en localStorage
❌ fail-open
❌ cross-tenant queries sin guard
❌ AI bypass
46. Database Access

Todas las consultas deberán incluir contexto de tenant cuando corresponda.

Conceptualmente:

Query
+
Tenant Context
+
Authorization Context

El tenant no debe depender solamente de un parámetro enviado por el cliente.

Debe derivarse/validarse desde el contexto autenticado y membership correspondiente.

47. Security-Sensitive Operations

Antes de implementar una operación crítica se deberá identificar:

Criticality
Freshness
Authorization
MFA
Session
Tenant
Audit
Event
Idempotency

Ejemplo:

Revoke Membership

requiere:

Authorization
→ C0
→ F0
→ transaction
→ security version
→ outbox
→ audit
→ invalidation
48. Implementation Order

La implementación no deberá comenzar por endpoints arbitrarios.

Orden recomendado:

1. Project / Runtime Foundation
        ↓
2. Identity Shared Kernel
        ↓
3. Account
        ↓
4. Organization
        ↓
5. Membership
        ↓
6. Role & Permission
        ↓
7. Session / Token
        ↓
8. Authentication
        ↓
9. Policy
        ↓
10. Authorization Runtime
        ↓
11. MFA
        ↓
12. Recovery
        ↓
13. Audit / Security Events
        ↓
14. Outbox / Events
        ↓
15. Cache / Invalidation
        ↓
16. Federation / Provisioning
        ↓
17. API completion
        ↓
18. Full Testing

Este orden es una secuencia de implementación propuesta, no una nueva decisión arquitectónica.

49. Implementation Story Mapping

Los IS existentes se pueden mapear:

IS	Implementation Area
IS-001	Foundation
IS-002	Account
IS-003	Organization
IS-004	Membership
IS-005	Authentication
IS-006	Authorization
IS-007	Role & Permission
IS-008	Session
IS-009	Refresh Token
IS-010	MFA
IS-011	Security Policy
IS-012	Audit/Security
IS-013	Events/Outbox
IS-014	API
IS-015	Persistence
IS-016	Testing
IS-017	Deployment

Así mantenemos el catálogo existente en lugar de crear historias duplicadas.

50. Technical Task Requirements

Cada IS deberá descomponerse en Technical Tasks antes de escribir código.

Ejemplo:

IS-006 Authorization
        │
        ├── TASK-AUTHZ-001
        │     Context Model
        │
        ├── TASK-AUTHZ-002
        │     Tenant Guard
        │
        ├── TASK-AUTHZ-003
        │     RBAC
        │
        ├── TASK-AUTHZ-004
        │     ABAC
        │
        ├── TASK-AUTHZ-005
        │     Ownership
        │
        ├── TASK-AUTHZ-006
        │     Delegation
        │
        ├── TASK-AUTHZ-007
        │     Policy Runtime
        │
        ├── TASK-AUTHZ-008
        │     Criticality
        │
        ├── TASK-AUTHZ-009
        │     Freshness
        │
        └── TASK-AUTHZ-010
              Tests
51. Definition of Done

Una Technical Task de Identity no está terminada solamente porque compile.

Debe cumplir:

Code
+
Unit Tests
+
Integration Tests
+
Security Tests
+
Traceability
+
Documentation
+
Observability
+
Quality Gate

Para operaciones críticas:

+
Golden Scenario
+
Security Invariant
52. Local Development

El entorno local debe poder levantar como mínimo:

API
PostgreSQL
Redis

y, cuando sea necesario:

Event infrastructure

Los servicios deben poder ejecutarse mediante infraestructura reproducible.

Docker forma parte del baseline tecnológico actual.

53. Configuration

La configuración debe separarse en:

Application Config
Environment Config
Secrets

Nunca:

hardcoded credentials

Debe existir validación de configuración al startup.

54. Database Migrations

Todas las modificaciones del modelo de Identity deben realizarse mediante migrations versionadas.

No depender de:

automatic schema creation in production

La migration debe validarse mediante:

forward migration
data integrity
constraints
indexes
rollback/recovery strategy

cuando corresponda.

55. Deployment Readiness

Antes de deployment:

Build
 ↓
Tests
 ↓
Security
 ↓
Migration validation
 ↓
Configuration validation
 ↓
Health checks
 ↓
Observability
 ↓
Quality Gates

La arquitectura exige que los cambios sean trazables hasta Source Code y que posteriormente pasen por Testing y Deployment.

56. Health Checks

Identity deberá proporcionar al menos:

liveness
readiness
dependency health

Pero:

health check no debe exponer secrets ni información interna sensible.

57. Graceful Failure

Cuando una dependencia no crítica falle:

System
 ↓
Controlled degradation

Cuando falle una dependencia crítica de seguridad:

System
 ↓
Fail Closed

Nunca:

Dependency Failure
 ↓
Security Disabled
58. AI Integration

AI será consumidor de Identity, no autoridad independiente.

AI Agent
   ↓
Identity
   ↓
Authentication
   ↓
Authorization Runtime
   ↓
Policy
   ↓
Decision

AI no podrá:

modificar directamente permisos;
saltarse MFA;
manipular security epochs;
modificar cache para obtener ALLOW;
activar policies;
acceder cross-tenant.
59. Implementation Quality Gates

Antes de considerar un módulo terminado:

✓ Architecture compliant
✓ Dependency compliant
✓ Unit tests
✓ Integration tests
✓ Security tests
✓ Tenant isolation tests
✓ API tests
✓ Observability
✓ Audit
✓ Events where required
✓ Migration
✓ Documentation
✓ Traceability
60. Implementation Readiness Checklist

Identity estará listo para entrar en programación cuando:

Architecture
 Bounded Context definido.
 Modular Monolith definido.
 Layers definidos.
 Module boundaries definidos.
 Authorization centralizado.
 Tenant boundary definido.
Security
 Authentication.
 Authorization.
 MFA.
 Sessions.
 Refresh tokens.
 Policies.
 Delegation.
 Criticality.
 Freshness.
 Cache security.
Persistence
 PostgreSQL authority.
 Redis cache.
 Outbox.
 Versioning.
 Security epochs.
Integration
 Event model.
 Idempotency.
 Replay.
 DLQ.
 Schema governance.
Quality
 Testing strategy.
 Golden scenarios.
 Security invariants.
 Regression.
 Mutation.
 Chaos.
 Quality gates.
Implementation
 Repository target.
 Domain path.
 Layer structure.
 Module structure.
 Implementation sequence.
 Traceability model.
61. Traceability

ESP-0001/14 queda relacionado con:

Blueprint
   ↓
Reference Architecture
   ↓
Engineering Standards
   ↓
ESP-0001/01–15
   ↓
ADR-001–069
   ↓
IS-001–017
   ↓
Technical Tasks
   ↓
Source Code

Esto respeta la dependencia formal establecida en DEPENDENCIES.md: ESP → IS → Technical Tasks → Source Code.

62. Non-Negotiable Implementation Rules

IMP-001 — No implementación fuera del lifecycle arquitectónico.

IMP-002 — No cross-module database access.

IMP-003 — No duplicated authorization engines.

IMP-004 — No fail-open.

IMP-005 — No cross-tenant access.

IMP-006 — No secrets in source code.

IMP-007 — No secrets in logs/events/tests.

IMP-008 — No Redis as authorization source of truth.

IMP-009 — No arbitrary policy code.

IMP-010 — No AI authorization bypass.

IMP-011 — No critical feature without automated security tests.

IMP-012 — No untraceable source code.

IMP-013 — No production schema changes outside migration process.

IMP-014 — No direct infrastructure dependency from Domain.

IMP-015 — No security decision outside Identity Authorization Runtime.

63. Final Assessment
ESP-0001/14 original

🔴 — Requería actualización importante.

El documento original fue escrito cuando varias decisiones de implementación todavía estaban abiertas.

Después de ADR-001 → ADR-069, muchas de esas decisiones ya quedaron establecidas.

ESP-0001/14 v1.1.0

🟢 — Consolidado y listo para pasar a Implementation Stories → Technical Tasks.

La arquitectura queda ahora claramente separada:

WHAT
 ↓
ESP-0001

WHY / ARCHITECTURAL DECISIONS
 ↓
ADR

WHAT TO BUILD
 ↓
Implementation Stories

HOW TO BREAK IT DOWN
 ↓
Technical Tasks

IMPLEMENT
 ↓
Source Code

PROVE IT WORKS
 ↓
Testing

RUN IT
 ↓
Deployment

Y esto es exactamente lo que necesitamos antes de tocar código: el repositorio evoxa-platform es el destino del Source Code y evoxa-docs mantiene la documentación, mientras que la trazabilidad entre ambos debe permanecer explícita.

Estado de ESP-0001

Con esto ya tenemos:

01 Domain Model                    🟢
02 Architecture                    🟢
03 Authentication & Authorization  🟢
04 User & Organization             🟢
05 Roles & Permissions             🟢
06 Sessions & Tokens               🟢
07 MFA                             🟢
08 Security Policies               🟢
09 Audit & Compliance              🟢
10 API Contracts                   🟢
11 Persistence & Data Model        🟢
12 Events & Integration            🟢
13 Testing & Quality               🟢
14 Implementation Requirements     🟢
15 Deployment & Operations         ⏳

El único documento que nos falta para cerrar la especificación técnica de Identity es ESP-0001/15 — Identity Deployment & Operational Requirements.

Después de ese documento, el paso lógico ya no sería inventar más arquitectura: sería tomar IS-001 Identity Domain Foundation, convertirla en Technical Tasks y comenzar la implementación real siguiendo exactamente esta especificación.
