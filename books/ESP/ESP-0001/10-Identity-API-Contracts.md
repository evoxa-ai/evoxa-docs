ESP-0001/10 — Identity API Contracts

Document ID: ESP-0001/10
Module: Identity
Version: 1.1.0
Status: Draft — Consolidated
Parent Specification: ESP-0001 — Identity
Target Repository: evoxa-platform
Target Path: apps/api/app/domains/identity/

1. Purpose

ESP-0001/10 define los contratos API públicos del Identity Domain de EVOXA.

El objetivo es establecer una interfaz estable entre Identity y:

Web Platform
Mobile Platform
Coach Platform
Administration
AI Orchestrator
otros dominios de negocio
servicios internos
integraciones externas autorizadas

Identity expone contratos estables y versionados; ningún dominio externo debe acceder directamente a su persistencia interna. Esto es consistente con el modelo de plataforma, donde los módulos colaboran mediante APIs y eventos manteniendo autonomía.

2. API Principles

Las APIs de Identity deberán cumplir:

API-first.
Versionado explícito.
OpenAPI 3.1.
Contratos backward-compatible cuando sea posible.
Seguridad por defecto.
Tenant isolation.
Authorization centralizada.
Idempotencia para operaciones retryable.
Correlation IDs.
Observabilidad.
Rate limiting.
Errores estandarizados.
No exposición de secretos.
No acceso directo a base de datos desde otros dominios.
Compatibilidad con Web, Mobile, Service Accounts y AI Agents.

El Blueprint define que los módulos deben exponer contratos estables y evolucionar independientemente.

3. API Versioning

La versión pública inicial será:

/api/v1

La versión forma parte del contrato público.

Ejemplo:

/api/v1/auth/login

Una nueva versión mayor será necesaria cuando exista un cambio incompatible.

Ejemplo:

/api/v2/...

Los cambios compatibles deberán preferentemente mantenerse dentro de la misma versión.

4. Canonical API Namespace

La API de Identity utilizará el namespace general:

/api/v1

Los recursos se organizarán por capability.

Authentication
/api/v1/auth/*
Users
/api/v1/users/*
Organizations
/api/v1/organizations/*
Memberships
/api/v1/memberships/*
Roles
/api/v1/roles/*
Permissions
/api/v1/permissions/*
Sessions
/api/v1/sessions/*
MFA
/api/v1/mfa/*
Authorization
/api/v1/authorization/*
Policies
/api/v1/policies/*
Audit
/api/v1/audit/*

La nomenclatura concreta de cada endpoint deberá quedar congelada en el contrato OpenAPI definitivo.

5. Authentication API
5.1 Login
POST /api/v1/auth/login

Request conceptual:

{
  "email": "user@example.com",
  "password": "********"
}

El servicio deberá:

validar request;
localizar identidad;
validar estado de cuenta;
verificar credenciales;
evaluar Security Policy;
determinar si MFA es requerido;
crear sesión;
emitir access credential;
emitir/establecer refresh credential;
generar audit/security events cuando corresponda.
6. Authentication Response

La respuesta de autenticación deberá diferenciar entre:

Authentication completed
{
  "user": {},
  "session": {},
  "access_token": "...",
  "token_type": "Bearer",
  "expires_in": 0
}

Los valores exactos de expiración serán definidos en configuración de runtime.

MFA required

La respuesta deberá indicar que el proceso de autenticación requiere un challenge adicional, sin considerar al usuario completamente autenticado para operaciones que requieren MFA.

Conceptualmente:

{
  "status": "MFA_REQUIRED",
  "challenge_id": "...",
  "session_id": "..."
}

El challenge real y sus detalles deberán estar sujetos a las reglas definidas en ESP-0001/07.

7. Refresh Token Contract

El refresh token será tratado como una credencial stateful y rotativa.

POST /api/v1/auth/refresh

La API deberá:

validar refresh credential;
validar sesión;
validar usuario;
validar membership/tenant cuando corresponda;
validar estado de seguridad;
detectar reutilización;
rotar el token;
emitir nuevo access token;
actualizar la familia correspondiente.

La reutilización de un refresh token usado deberá activar el flujo de seguridad definido por Identity.

8. Web vs Mobile Token Handling

Este punto queda explícitamente congelado por seguridad.

Web

El refresh token no deberá entregarse como parte normal del JSON de respuesta.

Deberá utilizarse un mecanismo de almacenamiento seguro basado en cookie:

HttpOnly
Secure
appropriate SameSite policy

El access token podrá mantenerse en memoria del cliente.

Mobile

El refresh credential deberá almacenarse mediante almacenamiento seguro proporcionado por el sistema operativo.

No deberá utilizarse almacenamiento inseguro del dispositivo.

No permitido
localStorage
URL query parameters
URL fragments
logs
analytics payloads
9. Logout
Current Session
POST /api/v1/auth/logout

Revoca la sesión/refresh credentials correspondientes.

All Sessions
POST /api/v1/auth/logout-all

Revoca las sesiones y familias de refresh correspondientes al usuario, sujeto a autorización y políticas de seguridad.

10. Current User
GET /api/v1/users/me

Devuelve el contexto básico del usuario autenticado.

No deberá devolver:

password hash;
refresh tokens;
MFA secrets;
recovery codes;
API secrets;
internal security credentials.
11. Organizations

Identity mantiene organizaciones como parte de su modelo de identidad.

Endpoints conceptuales:

GET    /api/v1/organizations
GET    /api/v1/organizations/{organization_id}
POST   /api/v1/organizations
PATCH  /api/v1/organizations/{organization_id}

La autorización se evaluará siempre dentro del contexto correspondiente.

Un usuario puede tener múltiples memberships y diferentes roles por organización.

Esto deriva del modelo de Identity donde Organization/Tenant constituye una frontera de seguridad y los dominios externos consumen Identity mediante contratos estables.

12. Membership API

Endpoints conceptuales:

GET    /api/v1/organizations/{organization_id}/memberships
POST   /api/v1/organizations/{organization_id}/memberships
GET    /api/v1/memberships/{membership_id}
PATCH  /api/v1/memberships/{membership_id}
DELETE /api/v1/memberships/{membership_id}

Membership constituye el boundary de acceso de un usuario dentro de una organización.

Toda operación deberá validar:

Authenticated Actor
        ↓
Organization
        ↓
Membership
        ↓
Authorization
13. Role API
GET    /api/v1/roles
GET    /api/v1/roles/{role_id}
POST   /api/v1/roles
PATCH  /api/v1/roles/{role_id}
DELETE /api/v1/roles/{role_id}

Asignación:

POST /api/v1/memberships/{membership_id}/roles

Revocación:

DELETE /api/v1/memberships/{membership_id}/roles/{role_id}

La API no debe asumir jerarquías implícitas entre roles.

14. Permission API
GET /api/v1/permissions
GET /api/v1/permissions/{permission_id}

Las permissions utilizarán el modelo conceptual:

<resource>.<action>

Ejemplos:

users.read
users.update
users.delete

organizations.read
organizations.update

sessions.read
sessions.revoke

El catálogo definitivo de resources/actions quedará congelado posteriormente.

15. Authorization API

Identity debe proporcionar un contrato explícito para evaluar autorización.

POST /api/v1/authorization/check

Request conceptual:

{
  "organization_id": "...",
  "resource": {
    "type": "workout",
    "id": "..."
  },
  "action": "read"
}

La evaluación deberá construir el contexto completo del actor.

16. Authorization Response

Conceptualmente:

{
  "decision": "ALLOW",
  "reason_code": "...",
  "criticality": "C2",
  "freshness": "F2-SHORT",
  "cacheable": true,
  "policy": {
    "id": "...",
    "version": "..."
  },
  "obligations": []
}

Decisions válidas:

ALLOW
DENY
CHALLENGE
REAUTHENTICATE
REQUIRE_MFA
REVOKE

La respuesta de autorización no debe interpretarse como una simple comprobación RBAC.

Debe representar el resultado del Authorization Runtime completo.

17. Authorization Runtime Contract

La API deberá respetar el pipeline establecido:

Request Validation
        ↓
Authentication Validation
        ↓
Tenant Guard
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
Authorization Decision

Por lo tanto:

RBAC ≠ Authorization completa.

18. Session API

Endpoints conceptuales:

GET  /api/v1/sessions
GET  /api/v1/sessions/{session_id}
POST /api/v1/sessions/{session_id}/revoke
POST /api/v1/sessions/revoke-all

La información expuesta deberá ser mínima.

No se expondrán:

refresh token;
token hashes;
secrets;
MFA secrets.
19. MFA API

Basado en ESP-0001/07.

Factors
GET  /api/v1/mfa/factors
POST /api/v1/mfa/factors
DELETE /api/v1/mfa/factors/{factor_id}
Challenges
POST /api/v1/mfa/challenges
POST /api/v1/mfa/challenges/{challenge_id}/verify
Recovery
POST /api/v1/mfa/recovery
POST /api/v1/mfa/recovery-codes/regenerate

Los secretos MFA nunca deberán aparecer en logs, eventos, métricas o respuestas API innecesarias.

20. Security Policy API

Las políticas deberán administrarse mediante Identity.

Conceptualmente:

GET  /api/v1/policies
GET  /api/v1/policies/{policy_id}
POST /api/v1/policies
PATCH /api/v1/policies/{policy_id}
POST /api/v1/policies/{policy_id}/validate
POST /api/v1/policies/{policy_id}/activate
POST /api/v1/policies/{policy_id}/retire

La activación de políticas de seguridad deberá pasar por los controles de gobernanza correspondientes.

AI puede asistir en análisis o generación de propuestas, pero no puede activar directamente una política de seguridad.

21. Audit API

Conceptualmente:

GET /api/v1/audit/events
GET /api/v1/audit/events/{event_id}

El acceso a audit data deberá estar autorizado mediante Identity.

La API deberá respetar:

Tenant Isolation
Least Privilege
Data Minimization
Audit Access Auditing
22. Pagination

Para colecciones grandes se utilizará preferentemente cursor pagination.

Ejemplo:

GET /api/v1/users?limit=50&cursor=...

El cursor deberá ser:

opaco;
no manipulable;
asociado al contexto de seguridad;
tenant-aware;
compatible con el ordering utilizado.

Para colecciones pequeñas y estables podrá utilizarse offset pagination.

23. Sorting

Los campos de ordenamiento deberán estar explícitamente allowlisted.

Ejemplo:

sort=created_at
sort=-created_at

No se permitirá que el cliente suministre expresiones arbitrarias de SQL.

24. Filtering

Los filtros deberán estar definidos por contrato.

Ejemplo:

GET /api/v1/users?status=ACTIVE

No se permitirá convertir parámetros de API directamente en expresiones SQL arbitrarias.

25. Idempotency

Las operaciones retryable deberán soportar:

Idempotency-Key: <opaque-key>

Especialmente:

creación;
operaciones financieras futuras relacionadas con Identity;
cambios sensibles;
provisioning;
operaciones que puedan ser reintentadas por clientes.

El mismo key con la misma operación deberá producir el mismo resultado lógico.

El mismo key con un request diferente deberá producir:

IDEMPOTENCY_CONFLICT

La idempotencia API es independiente de la idempotencia de consumidores de eventos.

26. Error Contract

Todas las APIs deberán utilizar un envelope uniforme:

{
  "error": {
    "code": "IDENTITY_ERROR",
    "message": "Request could not be processed.",
    "correlation_id": "..."
  }
}

Los errores deberán tener códigos estables.

Ejemplos conceptuales:

AUTHENTICATION_FAILED
AUTHORIZATION_DENIED
MFA_REQUIRED
MFA_INVALID
SESSION_REVOKED
TOKEN_EXPIRED
TOKEN_REUSE_DETECTED
TENANT_ACCESS_DENIED
RESOURCE_NOT_FOUND
VALIDATION_ERROR
IDEMPOTENCY_CONFLICT
RATE_LIMITED
POLICY_DENIED

No se deberán exponer detalles internos que faciliten:

credential enumeration;
policy discovery;
database discovery;
secret discovery;
infrastructure discovery.
27. HTTP Status Codes

Se utilizarán códigos HTTP estándar.

Ejemplo conceptual:

Status	Uso
200	Successful query/update
201	Resource created
202	Accepted asynchronous operation
204	Successful operation without body
400	Invalid request
401	Authentication required/failed
403	Authorization denied
404	Resource unavailable
409	Conflict/idempotency conflict
422	Validation error
429	Rate limited
500	Internal error
503	Service unavailable

La semántica exacta de cada endpoint se congelará en OpenAPI.

28. Correlation & Traceability

Las requests deberán soportar correlation/trace identifiers.

Conceptualmente:

X-Correlation-ID: ...
X-Request-ID: ...

Cuando exista distributed tracing deberán integrarse los identificadores correspondientes.

Los identificadores deberán aparecer en:

API telemetry;
audit;
security events;
logs;
traces;
event metadata cuando corresponda.
29. Security Headers

La API deberá aplicar los controles definidos por los estándares de seguridad de EVOXA.

Entre ellos:

TLS;
secure cookies cuando correspondan;
appropriate CORS;
security headers;
rate limiting;
request validation;
authentication;
authorization.
30. Rate Limiting

Las APIs sensibles deberán aplicar rate limiting multidimensional.

Dimensiones posibles:

IP
User
Organization
Endpoint
Operation
Session
Device
Risk

Las operaciones especialmente sensibles incluyen:

login;
MFA;
recovery;
password reset;
token refresh;
API token operations;
administrative operations.
31. Tenant Isolation

Todo endpoint que opere sobre recursos tenant-scoped deberá resolver explícitamente:

Actor
Organization
Membership
Tenant
Resource

Una ausencia o ambigüedad de tenant deberá producir comportamiento fail-closed.

Cross-tenant access está prohibido por defecto.

32. AI API Access

Los AI Agents deberán utilizar los mismos contratos de Identity.

No existirán:

AI bypass
AI superuser implicit
AI authorization shortcut

Un AI Agent tendrá:

Identity
Service Identity
Scopes
Roles
Permissions
Tenant Context
Policy Context
Audit Identity

La autorización del AI utilizará el mismo Authorization Runtime.

33. Service-to-Service API

Las comunicaciones internas deberán soportar identidades de workload.

Modelos posibles definidos por los ADR:

OAuth2.1 Client Credentials;
signed JWT assertions;
mTLS;
workload identity.

Las credenciales deberán:

tener audiencia restringida;
tener privilegio mínimo;
ser de corta duración cuando corresponda;
ser auditables.
34. OAuth2.1 / OIDC

Para aplicaciones Web/Mobile:

Authorization Code
        +
PKCE

Para service-to-service:

Client Credentials

OIDC se utilizará para autenticación federada.

Un ID Token no será tratado como API access token.

35. API Security Model

Todas las APIs deberán respetar:

Authentication
      ↓
Tenant Context
      ↓
Authorization
      ↓
Policy
      ↓
Criticality
      ↓
Freshness
      ↓
Execution
      ↓
Audit

No deberá existir lógica de autorización duplicada de manera independiente en cada dominio.

Identity es responsable de authentication y authorization centralizadas. El Blueprint establece explícitamente que Authentication y Authorization pertenecen exclusivamente al Identity Domain.

36. Events vs API

Las APIs representan:

Commands
Queries
Synchronous interactions

Los eventos representan:

Facts
Domain changes
Integration notifications

No deberán utilizarse eventos como sustituto de una API síncrona cuando el consumidor necesita una decisión inmediata.

El Blueprint define explícitamente APIs y eventos como mecanismos de integración separados.

37. API Contract Governance

Cada contrato deberá contener:

API ID
Version
Operation
HTTP Method
Path
Authentication
Authorization
Request Schema
Response Schema
Error Schema
Idempotency
Rate Limit
Audit Requirement
Security Classification
Tenant Scope

Los contratos serán mantenidos en OpenAPI.

38. Breaking Changes

Cambios incompatibles incluyen, entre otros:

eliminar endpoint;
cambiar semántica de endpoint;
eliminar required field;
cambiar tipo de campo;
modificar significado de un campo;
eliminar response property obligatoria;
cambiar autorización de manera incompatible;
cambiar security semantics.

Estos cambios requieren nueva versión mayor o mecanismo explícito de migración.

39. Contract Testing

Las APIs deberán tener Contract Tests.

Deberán validar:

Request
schema;
required fields;
types;
validation;
authentication.
Response
schema;
status;
headers;
security properties.
Errors
status;
error code;
message;
correlation ID.

El Blueprint de testing establece Contract Testing específicamente para API compatibility, request schema, response schema y error models.

40. Security Contract Tests

Identity deberá probar específicamente:

unauthenticated → DENY
wrong tenant → DENY
revoked session → DENY
revoked membership → DENY
revoked role → DENY
revoked permission → DENY
revoked MFA → DENY/required security action
expired token → DENY
refresh reuse → security response
unknown policy state → fail closed
authorization error → fail closed

También deberán cubrirse los invariantes definidos por los ADR de autorización y criticality.

41. Performance

Los contratos deberán respetar los objetivos establecidos para Identity.

Objetivos conceptuales:

Authorization < 50 ms
Permission Resolution < 30 ms

Estos objetivos aparecen también en los estándares de seguridad del Blueprint.

El API global mantiene además los objetivos de performance definidos por la arquitectura.

42. Observability

Cada API deberá generar telemetry suficiente para determinar:

request
actor
tenant
operation
resource
result
latency
correlation
trace
policy
criticality
freshness
security events

Nunca deberán registrarse:

passwords
refresh tokens
MFA secrets
recovery codes
private keys
API secrets
43. Privacy

Las APIs deberán aplicar:

data minimization;
least privilege;
tenant isolation;
explicit authorization;
no secrets in URLs;
no sensitive data in logs;
controlled audit access.
44. API Layer Architecture

La implementación seguirá:

Presentation
      ↓
API
      ↓
Application
      ↓
Domain
      ↓
Infrastructure

El API layer no deberá contener business rules ni acceder directamente a repositories de otros módulos. Esta separación corresponde al modelo arquitectónico oficial de EVOXA.

45. API Responsibilities
API Layer

Responsable de:

HTTP;
serialization;
validation;
authentication extraction;
correlation;
response mapping;
error mapping.
Application Layer

Responsable de:

use cases;
orchestration;
transactions;
authorization invocation.
Domain

Responsable de:

invariants;
domain rules;
domain entities;
domain services.
Infrastructure

Responsable de:

PostgreSQL;
Redis;
Event Platform;
external IdPs;
email/SMS/push providers;
KMS;
secret management.
46. Non-Negotiable API Security Rules

API-001 — All Identity APIs are versioned.

API-002 — Authentication is mandatory where required by operation.

API-003 — Authorization is centralized.

API-004 — Tenant isolation cannot be bypassed.

API-005 — Cross-tenant access is denied by default.

API-006 — Unknown authorization state cannot produce ALLOW.

API-007 — Authorization errors fail closed.

API-008 — Refresh tokens are never exposed through unsafe channels.

API-009 — Secrets are never returned unnecessarily.

API-010 — Secrets are never logged.

API-011 — Critical write operations support idempotency where retryable.

API-012 — Pagination/filtering/sorting are allowlisted.

API-013 — AI Agents use the same Identity contracts.

API-014 — AI cannot bypass authorization.

API-015 — API changes are contract-tested.

API-016 — Security-sensitive breaking changes require governance.

API-017 — Audit requirements are part of the API contract.

API-018 — Correlation is preserved across synchronous and asynchronous flows.

API-019 — API availability failure cannot create authorization bypass.

API-020 — Identity persistence is never accessed directly by external domains.

47. Traceability
Artifact	Relationship
BP-0002	Reference Architecture
ESP-0001/01	Domain Model
ESP-0001/03	Authentication & Authorization
ESP-0001/05	Roles & Permissions
ESP-0001/06	Sessions & Tokens
ESP-0001/07	MFA
ESP-0001/08	Security Policies
ESP-0001/09	Audit & Compliance
ESP-0001/10	API Contracts
ESP-0001/11	Persistence
ESP-0001/12	Events
ESP-0001/13	Testing
ESP-0001/14	Implementation
ESP-0001/15	Deployment

Las APIs son además uno de los contratos públicos definidos por el Identity Domain, junto con User, Organization, Tenant, Role, Permission y Session APIs.

48. Implementation Readiness
Capability	Estado
API versioning	🟢
Authentication API	🟢
Refresh contract	🟢
Web/Mobile token strategy	🟢
Session API	🟢
MFA API	🟢
Authorization API	🟢
Tenant isolation	🟢
RBAC	🟢
ABAC	🟢
Policy Engine	🟢
Criticality	🟢
Freshness	🟢
Idempotency	🟢
Pagination	🟢
Error model	🟢
Correlation	🟢
Rate limiting	🟢
AI authorization	🟢
Contract testing	🟢
Exact OpenAPI schemas	🟡
Exact endpoint catalog	🟡
Exact HTTP semantics per endpoint	🟡
Exact request/response DTOs	🟡
Exact authentication provider configuration	🟡
49. Decisions Deliberately Deferred

Este ESP no inventa detalles que todavía no están congelados.

Quedan para la siguiente fase:

OpenAPI machine-readable definitivo.
DTOs completos.
catálogo definitivo de endpoints.
catálogo definitivo de resources/actions.
parámetros exactos de tokens.
parámetros exactos de rate limiting.
MFA provider configuration.
exact pagination limits.
exact error code registry.
exact API permission mapping.
webhook contracts, si aplican.
SDK contracts.
API Gateway configuration.

Estos puntos deberán resolverse mediante los siguientes ESP/IS/Technical Tasks, no mediante decisiones improvisadas durante la programación.

50. Conclusión

ESP-0001/10 v1.1.0 queda conceptualmente alineado con el modelo actual de EVOXA Identity.

El cambio importante respecto del documento original es que ahora el API Contract no se limita a:

HTTP → Controller → Response

sino que formaliza:

Client
  ↓
API Contract
  ↓
Authentication
  ↓
Tenant Context
  ↓
Authorization Runtime
  ↓
Policy
  ↓
Criticality
  ↓
Freshness
  ↓
Execution
  ↓
Audit / Events

Esto es importante porque Identity no es simplemente un CRUD de usuarios: es el Core Domain de confianza y control de acceso de toda la plataforma. El Blueprint establece precisamente que Identity es consumido por los demás dominios, pero permanece independiente de ellos.

Estado

ESP-0001/10 — 🟢 Arquitecturalmente consolidado

No veo necesario crear otro ADR solamente para completar este ESP. Los ADR existentes ya cubren las decisiones de seguridad, autorización, sesiones, tokens, MFA, políticas, cache, idempotencia y contratos.

El siguiente paso natural sería ESP-0001/11 — Identity Persistence & Data Model, donde podemos transformar todo esto en el modelo PostgreSQL real: tablas, aggregates, relaciones, constraints, índices, versionado, outbox, security epochs y estrategia de migraciones.
