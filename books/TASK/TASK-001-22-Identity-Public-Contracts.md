TASK-001-22 — Identity Public Contracts

Identity Implementation Story: IS-001 — Identity Domain Foundation
ESP: ESP-0001/10 — Identity API Contracts
ESP relacionado: ESP-0001/14 — Identity Implementation Requirements
Dependencias: TASK-001-14, TASK-001-15, TASK-001-16, TASK-001-20, TASK-001-21
Prioridad: 🔴 Critical
Estado: 🟢 Ready for Implementation

1. Objetivo

Establecer los contratos públicos de Identity que permiten que el resto de EVOXA interactúe con Identity sin depender de sus entidades internas, tablas, repositories o implementación tecnológica.

La regla fundamental será:

Los consumidores conocen contratos; no conocen la implementación interna de Identity.

La arquitectura queda:

                         EVOXA Identity
                              │
                 ┌────────────┴────────────┐
                 │ Public Contracts Layer  │
                 └────────────┬────────────┘
                              │
       ┌──────────────────────┼──────────────────────┐
       ↓                      ↓                      ↓
      Web                   Mobile                Other Domains
       │                      │                      │
       └──────────────────────┼──────────────────────┘
                              ↓
                         Identity API
                              ↓
                         Application
                              ↓
                            Domain

Esto prepara Identity para ser utilizado posteriormente por:

Web Angular
Flutter
otros dominios EVOXA
AI Agents
integraciones externas
servicios internos
2. Principio de encapsulación

Está prohibido que otro módulo haga:

❌ Identity DB
❌ Identity Repository
❌ Identity ORM Model
❌ Identity internal service
❌ Identity internal table

Por ejemplo:

Nutrition Domain
       │
       ❌
       ↓
identity.membership_repository

Debe hacer:

Nutrition Domain
       │
       ↓
Membership Contract
       │
       ↓
Identity
3. Tipos de contratos

Identity tendrá varios niveles de contratos:

Public Contracts
│
├── HTTP/API Contracts
│
├── Application Contracts
│
├── Domain Integration Contracts
│
├── Event Contracts
│
├── Authorization Contracts
│
└── Service-to-Service Contracts

No todos tienen el mismo nivel de exposición.

4. API pública

El baseline será:

/api/v1

Identity utilizará:

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
/api/v1/audit/*

Los contratos concretos de cada endpoint deberán mantenerse alineados con ESP-0001/10.

5. Versionamiento

La API pública deberá ser versionada:

/api/v1

Un cambio breaking no debe modificar silenciosamente el contrato existente.

Ejemplo:

v1
 ↓
compatible evolution

pero si existe un breaking change:

v1
 ↓
v2

La versión forma parte del contrato público.

6. OpenAPI

El contrato HTTP debe poder representarse mediante:

OpenAPI 3.1

Debe describir:

endpoints;
métodos;
request schemas;
response schemas;
errores;
authentication;
authorization requirements;
headers;
pagination;
idempotency;
versioning.

La especificación OpenAPI será un artefacto de contrato, no una sustitución de las reglas de negocio.

7. Request / Response DTOs

No debemos devolver directamente entidades Domain:

return user

El contrato debe utilizar DTOs:

HTTP Request
     ↓
Request DTO
     ↓
Application
     ↓
Domain
     ↓
Response DTO
     ↓
HTTP Response

Esto protege el Domain de cambios en la API.

8. User Contract

Conceptualmente:

class UserResponse:
    id: UUID
    email: str
    status: UserStatus
    created_at: datetime
    updated_at: datetime

No debemos exponer automáticamente:

❌ password_hash
❌ password_reset_token
❌ MFA secret
❌ recovery codes
❌ refresh token
❌ internal security fields

La representación pública debe contener únicamente atributos autorizados.

9. Organization Contract

Conceptualmente:

class OrganizationResponse:
    id: UUID
    name: str
    status: OrganizationStatus
    created_at: datetime
    updated_at: datetime

La organización es además una frontera fundamental de tenant.

10. Membership Contract

El contrato de Membership debe representar la relación:

User
  │
  │ membership
  ↓
Organization

y posteriormente:

Membership
   ↓
Roles

Nunca debemos obligar al consumidor a conocer las tablas:

users
memberships
membership_roles
organizations
11. Authentication Contracts

Los contratos de Authentication deben permitir posteriormente:

Login
Refresh
Logout
Logout All

Por ejemplo:

POST /api/v1/auth/login

Request conceptual:

{
  "identifier": "...",
  "password": "..."
}

Response exitoso conceptualmente:

{
  "access_token": "...",
  "token_type": "Bearer",
  "expires_in": 900
}

Nota importante: la representación exacta de tokens y sus campos deberá quedar alineada con ESP-0001/10 y ADR-002/003/009/025.

12. MFA_REQUIRED

Authentication no siempre produce un access token inmediatamente.

Puede producir:

MFA_REQUIRED

Conceptualmente:

{
  "status": "MFA_REQUIRED",
  "challenge_id": "...",
  "available_factors": []
}

No se debe devolver:

❌ MFA secret
❌ OTP
❌ recovery code
13. Refresh Contract

Endpoint:

POST /api/v1/auth/refresh

El contrato debe respetar la estrategia definida:

Web
 ↓
HttpOnly + Secure Cookie

y:

Mobile
 ↓
OS Secure Storage

El API contract no debe obligar al cliente a manipular directamente un refresh token cuando la arquitectura de almacenamiento lo abstrae.

14. Logout Contracts

Identity debe soportar conceptualmente:

POST /api/v1/auth/logout

y:

POST /api/v1/auth/logout-all

El segundo representa una operación de mayor sensibilidad porque afecta múltiples sesiones/familias de tokens.

Esto será importante posteriormente para:

Criticality
Freshness
Cache
Audit
15. Authorization Contract

Uno de los contratos más importantes será:

POST /api/v1/authorization/check

Conceptualmente:

{
  "resource": "training_plan",
  "action": "read",
  "resource_id": "..."
}

El resultado no debe reducirse a:

{
  "allowed": true
}

porque el modelo de Identity contempla:

ALLOW
DENY
CHALLENGE
REAUTHENTICATE
REQUIRE_MFA
REVOKE

Por lo tanto, el contrato debe poder representar una Authorization Decision completa.

16. Authorization Decision

Conceptualmente:

{
  "decision": "ALLOW",
  "reason_code": "...",
  "policy_id": "...",
  "policy_version": "...",
  "criticality": "C2",
  "freshness": "F1"
}

Los campos exactos deben congelarse en el contrato definitivo.

No debemos exponer innecesariamente:

❌ reglas internas completas
❌ atributos sensibles
❌ información confidencial de políticas
❌ secretos
17. Application Contracts

Además de HTTP, otros módulos podrán necesitar contratos internos.

Ejemplo:

class MembershipQuery(Protocol):

    async def get_membership(
        self,
        user_id: UUID,
        organization_id: UUID,
    ) -> MembershipView | None:
        ...

La idea es:

Domain A
   ↓
Public Application Contract
   ↓
Identity Application

y no:

Domain A
   ↓
Identity Repository
18. Contract Types

Podemos separar:

Commands
Queries
Decisions
Events

Por ejemplo:

Commands
Authenticate
CreateMembership
AssignRole
RevokeSession
EnrollMFA
Queries
GetUser
GetMembership
GetOrganization
GetEffectivePermissions
Decisions
Authorize
Events
UserCreated
MembershipCreated
RoleAssigned
SessionRevoked

Los nombres concretos de eventos se definirán cuando las IS funcionales correspondientes sean implementadas.

19. No exponer Domain Entities

Regla:

Domain Entity
      ≠
Public Contract

Aunque inicialmente puedan parecer iguales, deben permanecer conceptualmente separados.

Esto permite que mañana:

Domain User

cambie internamente sin romper:

UserResponse
20. Error Contract

Debe utilizarse el contrato establecido anteriormente:

{
  "error": {
    "code": "...",
    "message": "...",
    "correlation_id": "..."
  }
}

Esto conecta directamente con:

TASK-001-15 — Exception Model

21. HTTP Status

Baseline:

Situación	HTTP
Validation Error	400
Authentication Required/Failed	401
Authorization Denied	403
Tenant Access Denied	403
Not Found	404
Conflict	409
Concurrency Conflict	409
Rate Limited	429
Dependency unavailable	503
Unexpected Error	500

La tabla definitiva deberá permanecer alineada con ESP-0001/10.

22. Correlation Contract

Todas las operaciones HTTP deben poder relacionarse mediante:

X-Correlation-ID

El flujo:

Client
 ↓
X-Correlation-ID
 ↓
API
 ↓
Application
 ↓
Domain
 ↓
Outbox
 ↓
Event

Esto conecta:

TASK-001-14
TASK-001-18
TASK-001-22
23. Trace Contract

Cuando exista tracing distribuido:

Trace ID

debe atravesar los límites correspondientes.

Pero:

Traceability nunca constituye autorización.

Un actor no obtiene privilegios por proporcionar un trace_id.

24. Idempotency Contract

Las operaciones retryable deberán soportar:

Idempotency-Key

cuando corresponda.

Reglas:

same key
+
same logical request
=
same logical result

Mientras:

same key
+
different request
=
IDEMPOTENCY_CONFLICT

No se debe confundir:

API Idempotency

con:

Event Consumer Idempotency
25. Pagination Contract

Para endpoints de colección:

users
organizations
memberships
roles
permissions
sessions
audit

se utilizará el modelo definido en ESP-0001/10:

cursor pagination

preferentemente para colecciones grandes/dinámicas.

El cursor debe ser:

opaque
tenant-bound
non-sensitive
deterministic
26. Filtering & Sorting

Los filtros deben utilizar allowlists.

Nunca:

?sort=raw_sql

ni:

?filter=<arbitrary expression>

El contrato deberá definir:

allowed filters
allowed sorting
allowed fields
27. Tenant Context

Toda operación tenant-scoped debe identificar explícitamente su contexto.

Conceptualmente:

Request
  ↓
Authenticated Actor
  ↓
Active Organization
  ↓
Membership validation
  ↓
Authorization

Nunca debemos confiar únicamente en:

organization_id enviado por cliente

para establecer autorización.

28. AI Agent Contracts

Los AI Agents también consumirán Identity mediante contratos.

AI Agent
   ↓
Identity Contract
   ↓
Authentication
   ↓
Authorization

Nunca:

AI Agent
   ↓
direct DB

ni:

AI Agent
   ↓
internal repository

Y:

AI no recibe privilegios especiales por ser AI.

29. Service-to-Service Contracts

Los servicios internos podrán utilizar:

OAuth2.1 Client Credentials

o mecanismos de workload identity establecidos en ADR-038.

La identidad del servicio deberá formar parte del contexto:

actor_type = SERVICE
30. Event Contracts

Identity publicará eventos mediante el modelo definido anteriormente.

Arquitectura:

Identity
   ↓
Domain Event
   ↓
Outbox
   ↓
Event Platform
   ↓
Integration Event
   ↓
Consumers

El evento debe utilizar el envelope estándar.

Conceptualmente:

{
  "event_id": "...",
  "event_type": "...",
  "event_version": 1,
  "occurred_at": "...",
  "tenant_id": "...",
  "actor": {},
  "resource": {},
  "payload": {},
  "metadata": {}
}
31. Event Versioning

Los eventos son contratos.

Por tanto:

Event v1

no debe cambiar semánticamente sin control.

Para breaking change:

UserCreated.v1
UserCreated.v2

según la estrategia definitiva de schema governance.

Esto conecta con:

ADR-020
ADR-064
ADR-065
ADR-066
32. Public Event Payload

Un evento debe contener únicamente la información necesaria para sus consumidores.

Nunca:

❌ password
❌ password_hash
❌ access token
❌ refresh token
❌ MFA secret
❌ recovery code
❌ private key
❌ API secret
33. Contract Registry

La evolución del proyecto debería terminar teniendo un catálogo:

contracts/
├── api/
├── application/
├── events/
├── authorization/
└── schemas/

Pero debemos evitar crear un segundo sistema de gobierno desconectado del ESP.

El Identity Contract Registry deberá integrarse posteriormente con el gobierno de schemas definido por ADR-066.

34. Estructura propuesta
apps/api/app/
│
├── api/
│   └── v1/
│       ├── auth/
│       ├── users/
│       ├── organizations/
│       ├── memberships/
│       ├── roles/
│       ├── permissions/
│       ├── sessions/
│       ├── mfa/
│       ├── authorization/
│       ├── policies/
│       └── audit/
│
└── domains/
    └── identity/
        ├── account/
        ├── organization/
        ├── membership/
        └── ...

Los schemas públicos pueden organizarse como:

app/api/v1/schemas/
└── identity/
    ├── auth.py
    ├── users.py
    ├── organizations.py
    ├── memberships.py
    ├── authorization.py
    ├── sessions.py
    └── errors.py
35. Application Contracts

Los contratos internos podrían ubicarse:

identity/
└── shared/
    └── application/
        └── contracts/

o dentro de cada módulo:

identity/
├── account/
│   └── application/
│       └── contracts.py
│
├── membership/
│   └── application/
│       └── contracts.py

La decisión definitiva dependerá de cuánto contrato sea verdaderamente compartido.

36. Contract Tests

TASK-001-20 debe extenderse con pruebas de contrato.

Ejemplo:

Client expectation
       ↓
Identity API
       ↓
Response schema

Debemos comprobar:

request schema
response schema
status codes
error schema
headers
versioning
37. Backward Compatibility

Una modificación compatible:

v1
 ↓
new optional response field

puede ser posible dependiendo del consumidor.

Una modificación breaking:

remove field
rename field
change type
change semantics

requiere tratamiento de versionamiento/gobierno.

38. Contract Evolution

El ciclo será:

Draft
 ↓
Validate
 ↓
Test
 ↓
Approve
 ↓
Publish
 ↓
Observe
 ↓
Evolve

No debemos cambiar contratos públicos directamente desde una implementación.

39. Contract Ownership

Identity será responsable de sus contratos.

Un consumidor puede:

consume
validate
request evolution

pero no modificar unilateralmente el contrato.

40. Forbidden Patterns

Quedan prohibidos:

❌ Exponer ORM models
❌ Exponer DB rows
❌ Exponer repositories
❌ Exponer password hashes
❌ Exponer tokens internos
❌ Exponer MFA secrets
❌ Permitir SQL arbitrario
❌ Permitir filtros arbitrarios
❌ Bypass de Authorization
❌ Bypass de Tenant Guard
❌ Contratos distintos para AI
41. Scope
Incluye
✓ API contract structure
✓ DTOs
✓ Error contract
✓ Authentication contracts
✓ Authorization contract
✓ Tenant contract
✓ Application contracts
✓ Event contract foundation
✓ S2S contract foundation
✓ Pagination
✓ Filtering
✓ Sorting
✓ Idempotency
✓ Versioning
✓ Contract tests
✓ OpenAPI foundation
No incluye
❌ Implementar Login
❌ Implementar JWT completo
❌ Implementar MFA
❌ Implementar Authorization Runtime
❌ Implementar Policy Engine
❌ Implementar User CRUD completo
❌ Implementar Event Consumers
❌ Implementar API Gateway
42. Acceptance Criteria
AC-01

Existe una estructura formal para Identity Public Contracts.

AC-02

Los contratos HTTP utilizan /api/v1.

AC-03

Los contratos se representan mediante DTOs y no Domain Entities.

AC-04

Existe contrato estándar de errores.

AC-05

Correlation ID forma parte del modelo de interacción.

AC-06

Idempotency-Key puede representarse en contratos retryable.

AC-07

Existe estructura para pagination.

AC-08

Existe estructura para filtering y sorting allowlisted.

AC-09

Tenant context forma parte de las operaciones tenant-scoped.

AC-10

Existe contrato conceptual de Authentication.

AC-11

Existe contrato conceptual de Authorization Decision.

AC-12

El contrato soporta decisiones distintas de ALLOW/DENY.

AC-13

Existe estructura para Event Contracts.

AC-14

Los eventos son versionables.

AC-15

Los contratos no exponen secretos.

AC-16

Existe soporte conceptual para Service-to-Service.

AC-17

AI Agents utilizan los mismos contratos y controles.

AC-18

Existe estrategia para backward compatibility.

AC-19

Los contratos pueden validarse mediante tests.

AC-20

La estructura está preparada para OpenAPI 3.1.

43. Definition of Done
[ ] API contract structure creada
[ ] DTO foundation
[ ] Error contract
[ ] Auth contract foundation
[ ] Authorization contract foundation
[ ] Tenant contract foundation
[ ] Event contract foundation
[ ] S2S contract foundation
[ ] Pagination contract
[ ] Filtering contract
[ ] Sorting contract
[ ] Idempotency contract
[ ] Versioning rules
[ ] OpenAPI foundation
[ ] Contract tests
[ ] Secret exposure tests
[ ] Architecture tests
[ ] Documentation
[ ] CI validation

Además:

[ ] ningún contrato depende directamente del ORM
[ ] ningún contrato expone secretos
[ ] ningún consumidor necesita acceder a Identity DB
[ ] AI utiliza el mismo modelo de seguridad
44. Traceability
Blueprint
   ↓
API Architecture
   ↓
ESP-0001/10
   ↓
ESP-0001/14
   ↓
IS-001
   ↓
TASK-001-22
   ↓
Identity Public Contracts

ADRs principales:

ADR-IDENTITY-002  Access Token
ADR-IDENTITY-003  Sessions / Refresh
ADR-IDENTITY-007  Events / Outbox
ADR-IDENTITY-020  Event Schema Governance
ADR-IDENTITY-026  Pagination / Idempotency
ADR-IDENTITY-038  S2S Authentication
ADR-IDENTITY-039  OAuth2.1 / OIDC
ADR-IDENTITY-045  Authorization
ADR-IDENTITY-049  Authorization Runtime
ADR-IDENTITY-064  Test Contracts / Evidence
ADR-IDENTITY-065  Schema Versioning
ADR-IDENTITY-066  Contract Governance
45. Relación con la arquitectura completa

Con TASK-001-22 completamos una pieza fundamental:

                    Identity
                       │
        ┌──────────────┼──────────────┐
        ↓              ↓              ↓
      Domain       Application        API
        │              │              │
        │              │              │
        └──────────────┼──────────────┘
                       ↓
               Public Contracts
                       │
       ┌───────────────┼────────────────┐
       ↓               ↓                ↓
      Web            Mobile          Other Domains
       │               │                │
       └───────────────┼────────────────┘
                       ↓
                  AI Agents

Esto es precisamente lo que queremos evitar:

                    ❌
                   /|\
                  / | \
              Web DB ORM AI
                  \ | /
                   \|/
                Identity DB
46. Estado actualizado de IS-001
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
001-17	Architecture Boundary Tests	🟢
001-18	Initial Observability	✅
001-19	Migration Framework	✅
001-20	Test Foundation	✅
001-21	CI Foundation	✅
001-22	Identity Public Contracts	🟢 Ready
Próximo paso
TASK-001-23 — Architecture Documentation

Será el último Task documental importante antes de la validación final de IS-001.

Ahí vamos a consolidar cómo queda documentada la arquitectura realmente implementable de Identity: estructura, dependencias, boundaries, contratos, decisiones ADR, diagramas, convenciones y reglas para que un desarrollador pueda entrar al repositorio y entender exactamente dónde debe colocar cada pieza de código y qué cosas tiene prohibido hacer.

Después de TASK-001-23, tendremos:

TASK-001-24 — IS-001 Validation

y ahí podremos cerrar formalmente IS-001 — Identity Domain Foundation antes de pasar a la siguiente Implementation Story.
