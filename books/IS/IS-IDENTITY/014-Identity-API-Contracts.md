IS-IDENTITY-014 — Identity API Contracts

Estado: Draft
Prioridad: Critical
Dominio: Identity
Capability: API Contracts / Identity API
ADR principal: ADR-IDENTITY-001
ADRs relacionados: ADR-IDENTITY-002, 003, 005, 006, 007, 008, 009, 010, 011
ESP: ESP-0001
Dependencies: IS-IDENTITY-001 → IS-IDENTITY-013
Arquitectura: Presentation → API → Application → Domain → Infrastructure

1. Propósito

Definir los contratos API formales del dominio Identity de EVOXA, estableciendo una interfaz estable entre clientes y el dominio Identity.

Esta IS convierte las capacidades definidas previamente en contratos HTTP/API consistentes para:

autenticación;
usuarios;
organizaciones;
membresías;
roles;
permisos;
autorización;
sesiones;
refresh tokens;
MFA;
políticas de seguridad;
auditoría;
eventos de seguridad.

El objetivo es que la implementación posterior pueda desarrollarse contra contratos definidos y versionados, evitando que el código determine unilateralmente el diseño de la API.

2. Business Goal

Proporcionar una API Identity:

consistente;
segura;
versionada;
multi-tenant;
idempotente;
observable;
documentada mediante OpenAPI;
preparada para Web, Mobile, AI y servicios internos.

La API debe actuar como contrato estable entre EVOXA y sus consumidores.

3. Fundamento arquitectónico

El Blueprint establece una arquitectura API-First, con:

/api/v1

como base de versionado.

La especificación API debe utilizar OpenAPI 3.1 y mantener un modelo de errores estándar:

{
  "error": {
    "code": "ERROR_CODE",
    "message": "Human readable message",
    "correlation_id": "..."
  }
}

También se establece el uso de:

Idempotency-Key

para operaciones de escritura donde corresponda.

4. Persona
Primary Personas
Web Application
Mobile Application
Internal Services
AI Services
Administrative interfaces
Secondary
Integration Partners
Platform Operations
Security Operations
5. Journey general
Client
  │
  ▼
HTTP Request
  │
  ├── Authentication
  │
  ├── Correlation ID
  │
  ├── Tenant Context
  │
  ├── Input Validation
  │
  ▼
API Layer
  │
  ▼
Application Use Case
  │
  ▼
Domain
  │
  ▼
Authorization / Policy
  │
  ▼
Persistence / Events
  │
  ▼
API Response
6. Principios API

La Identity API deberá respetar:

6.1 API First

El contrato debe definirse antes de la implementación.

6.2 Versioning
/api/v1

Las modificaciones incompatibles deberán utilizar una nueva versión.

6.3 Stateless HTTP

La API HTTP no debe depender de estado de sesión almacenado en memoria del proceso.

La sesión de Identity, sin embargo, continúa siendo stateful a nivel de dominio.

7. Tenant Context

Las operaciones que requieren contexto organizacional deberán ejecutarse dentro del tenant correspondiente.

Conceptualmente:

Request
 │
 ├── actor
 ├── organization
 ├── membership
 └── authorization context

El tenant no debe confiarse exclusivamente a un parámetro enviado por el cliente.

Debe derivarse y validarse mediante el contexto autenticado/autorizado.

8. Authentication API
8.1 Login
POST /api/v1/auth/login
Request conceptual
{
  "email": "user@example.com",
  "password": "********",
  "organization_id": "uuid"
}

organization_id será requerido solamente cuando el flujo necesite selección/contexto explícito de organización.

Response conceptual
{
  "user": {
    "id": "uuid",
    "email": "user@example.com"
  },
  "session": {
    "id": "uuid"
  },
  "access_token": "..."
}

El tratamiento del refresh token dependerá del cliente:

Web → HttpOnly + Secure cookie.
Mobile → secure storage del sistema operativo.

No deberá aparecer en URLs ni logs.

9. Authentication Outcomes

El endpoint debe poder representar diferentes resultados:

AUTHENTICATED
FAILED
MFA_REQUIRED
CHALLENGE_REQUIRED
BLOCKED

Por ejemplo:

{
  "status": "MFA_REQUIRED",
  "challenge_id": "uuid"
}

No se debe utilizar una respuesta genérica que permita al cliente asumir que la autenticación fue completada cuando todavía existe un challenge obligatorio.

10. Refresh Token API
POST /api/v1/auth/refresh

El endpoint deberá:

validar refresh token;
validar sesión;
validar User;
validar Membership/Organization cuando corresponda;
evaluar políticas;
detectar reuse;
rotar token;
generar nuevo access token;
registrar eventos correspondientes.

El refresh token debe ser stateful y rotatorio, conforme a IS-IDENTITY-009.

11. Logout
Logout actual
POST /api/v1/auth/logout

Revoca la sesión actual y su capacidad de refresh.

Logout global
POST /api/v1/auth/logout-all

Revoca las sesiones del usuario según la política aplicable.

La operación debe generar audit/security events cuando corresponda.

12. User API
Crear usuario
POST /api/v1/identity/users
Obtener usuario
GET /api/v1/identity/users/{user_id}
Actualizar usuario
PATCH /api/v1/identity/users/{user_id}
Activar usuario
POST /api/v1/identity/users/{user_id}/activate
Suspender usuario
POST /api/v1/identity/users/{user_id}/suspend
Deshabilitar usuario
POST /api/v1/identity/users/{user_id}/disable
13. User Response

Los endpoints de usuario no deben devolver:

password;
password hash;
reset tokens;
MFA secrets;
recovery codes;
refresh tokens;
información criptográfica privada.

Ejemplo:

{
  "id": "uuid",
  "email": "user@example.com",
  "status": "ACTIVE",
  "created_at": "2026-09-02T10:00:00Z",
  "updated_at": "2026-09-02T10:00:00Z"
}
14. Organization API
Crear
POST /api/v1/identity/organizations
Obtener
GET /api/v1/identity/organizations/{organization_id}
Actualizar
PATCH /api/v1/identity/organizations/{organization_id}
Suspender
POST /api/v1/identity/organizations/{organization_id}/suspend
Reactivar
POST /api/v1/identity/organizations/{organization_id}/reactivate

La organización constituye el tenant boundary.

15. Membership API
Crear Membership
POST /api/v1/identity/organizations/{organization_id}/memberships
Obtener
GET /api/v1/identity/memberships/{membership_id}
Activar
POST /api/v1/identity/memberships/{membership_id}/activate
Suspender
POST /api/v1/identity/memberships/{membership_id}/suspend
Reactivar
POST /api/v1/identity/memberships/{membership_id}/reactivate
Eliminar
POST /api/v1/identity/memberships/{membership_id}/remove

La relación User ↔ Organization se mantiene mediante Membership.

16. Role API
Crear Role
POST /api/v1/identity/roles
Listar Roles
GET /api/v1/identity/roles
Obtener Role
GET /api/v1/identity/roles/{role_id}
Actualizar
PATCH /api/v1/identity/roles/{role_id}
Desactivar
POST /api/v1/identity/roles/{role_id}/deactivate
17. Permission API
Crear Permission
POST /api/v1/identity/permissions
Listar
GET /api/v1/identity/permissions
Obtener
GET /api/v1/identity/permissions/{permission_id}
Actualizar
PATCH /api/v1/identity/permissions/{permission_id}
Desactivar
POST /api/v1/identity/permissions/{permission_id}/deactivate

El formato conceptual será:

<resource>.<action>

Ejemplos:

users.read
users.create
users.update
users.delete
audit.read
audit.export

El catálogo definitivo de permisos deberá mantenerse como configuración/datos de Identity y no hardcodearse arbitrariamente en cada endpoint.

18. Role ↔ Permission
Asignar Permission
POST /api/v1/identity/roles/{role_id}/permissions
{
  "permission_id": "uuid"
}
Revocar Permission
DELETE /api/v1/identity/roles/{role_id}/permissions/{permission_id}

La relación debe ser explícita.

19. Membership ↔ Role
Asignar Role
POST /api/v1/identity/memberships/{membership_id}/roles
{
  "role_id": "uuid"
}
Revocar Role
DELETE /api/v1/identity/memberships/{membership_id}/roles/{role_id}

Esto materializa:

User
 │
 ▼
Membership
 │
 ▼
Role
 │
 ▼
Permission
20. Authorization API

Endpoint conceptual:

POST /api/v1/identity/authorization/check
Request
{
  "resource": {
    "type": "workout",
    "id": "uuid"
  },
  "operation": "update"
}

El contexto del actor no debe depender de datos enviados libremente por el cliente.

El servidor construirá:

AuthorizationRequest

con:

user;
organization;
membership;
roles;
permissions;
session;
authentication method;
MFA;
device;
request;
resource;
operation;
risk/context.
21. Authorization Response
{
  "decision": "ALLOW",
  "reason": "PERMISSION_GRANTED",
  "policy_id": "uuid",
  "evaluated_at": "2026-09-02T10:00:00Z"
}

Posibles decisiones:

ALLOW
DENY
CHALLENGE
REAUTHENTICATE
REQUIRE_MFA
REVOKE

El consumidor debe respetar cualquier decisión distinta de ALLOW.

22. Session API
Sesión actual
GET /api/v1/identity/sessions/current
Listar sesiones
GET /api/v1/identity/sessions
Obtener sesión
GET /api/v1/identity/sessions/{session_id}
Revocar sesión
POST /api/v1/identity/sessions/{session_id}/revoke
Revocar todas
POST /api/v1/identity/sessions/revoke-all
23. Session Response

Nunca deberá incluir secretos.

Ejemplo:

{
  "id": "uuid",
  "status": "ACTIVE",
  "created_at": "2026-09-02T10:00:00Z",
  "last_activity_at": "2026-09-02T10:15:00Z",
  "expires_at": "2026-09-03T10:00:00Z",
  "auth_method": "PASSWORD",
  "mfa_status": "VERIFIED"
}
24. MFA API

Base:

/api/v1/identity/mfa
Factores
GET /api/v1/identity/mfa/factors
Enrollment
POST /api/v1/identity/mfa/factors
Verificación
POST /api/v1/identity/mfa/challenges/{challenge_id}/verify
Revocación
POST /api/v1/identity/mfa/factors/{factor_id}/revoke
Challenges
POST /api/v1/identity/mfa/challenges
25. Recovery Codes
POST /api/v1/identity/mfa/recovery-codes/generate
POST /api/v1/identity/mfa/recovery-codes/verify
POST /api/v1/identity/mfa/recovery-codes/revoke

Los códigos de recuperación nunca deberán almacenarse ni exponerse en logs en texto plano.

26. Trusted Devices
GET /api/v1/identity/mfa/trusted-devices
POST /api/v1/identity/mfa/trusted-devices
POST /api/v1/identity/mfa/trusted-devices/{device_id}/revoke

Las reglas definitivas de TTL y binding quedan pendientes de decisión específica de MFA.

27. Security Policy API

La administración de políticas deberá estar protegida mediante autorización de alto privilegio.

Conceptualmente:

POST /api/v1/identity/security-policies
GET /api/v1/identity/security-policies
GET /api/v1/identity/security-policies/{policy_id}
PATCH /api/v1/identity/security-policies/{policy_id}
POST /api/v1/identity/security-policies/{policy_id}/activate
POST /api/v1/identity/security-policies/{policy_id}/suspend
POST /api/v1/identity/security-policies/{policy_id}/retire

La API deberá soportar versionado de políticas.

28. Audit API

Base:

/api/v1/identity/audit-events
Listar
GET /api/v1/identity/audit-events
Detalle
GET /api/v1/identity/audit-events/{event_id}

La búsqueda deberá poder filtrar, según autorización:

tenant;
actor;
event type;
resource;
operation;
result;
date/time;
correlation ID;
trace ID.

El acceso a audit es una operación privilegiada.

29. Security Events API
GET /api/v1/identity/security-events
GET /api/v1/identity/security-events/{event_id}

El endpoint deberá aplicar autorización estricta y tenant isolation.

30. Pagination

Las APIs de colección deberán soportar paginación.

Conceptualmente:

GET /users?page=1&page_size=25

La forma definitiva:

offset;
cursor;
continuation token;

queda pendiente de la especificación API global.

Para grandes colecciones, se recomienda evaluar cursor pagination, pero esto es una decisión derivada, no una regla normativa del Blueprint.

31. Filtering

Las colecciones podrán soportar filtros:

status
organization_id
role
created_at
updated_at

No debe permitirse que el cliente construya consultas arbitrarias.

Los filtros deben ser parte del contrato.

32. Sorting

Debe definirse un conjunto explícito de campos ordenables.

Ejemplo:

created_at
updated_at
email
status

El API debe rechazar campos no permitidos.

33. Idempotency

Las operaciones de escritura susceptibles a repetición deben soportar:

Idempotency-Key: <unique-key>

Ejemplo:

POST /api/v1/identity/memberships
Idempotency-Key: 123456

Esto resulta especialmente importante para:

creación;
asignaciones;
revocaciones;
operaciones MFA;
operaciones sensibles.

La estrategia exacta de almacenamiento de idempotency keys queda pendiente de la arquitectura de plataforma.

34. HTTP Status Codes

Se propone utilizar semántica HTTP consistente.

Status	Uso
200	Operación exitosa
201	Recurso creado
202	Operación aceptada/asíncrona
204	Operación exitosa sin body
400	Request inválido
401	No autenticado
403	No autorizado
404	Recurso no encontrado
409	Conflicto
422	Validación semántica
429	Rate limit
500	Error interno
503	Servicio temporalmente no disponible

Los códigos exactos deberán alinearse con el API Architecture global.

35. Error Contract

Todos los errores deben mantener el formato estándar:

{
  "error": {
    "code": "IDENTITY_USER_NOT_FOUND",
    "message": "User not found",
    "correlation_id": "uuid"
  }
}

No deberán devolverse:

stack traces;
SQL;
secretos;
detalles criptográficos;
información interna innecesaria.
36. Error Taxonomy

Se deberán establecer códigos estables.

Ejemplos:

AUTH_INVALID_CREDENTIALS
AUTH_MFA_REQUIRED
AUTH_SESSION_REVOKED
AUTH_REFRESH_TOKEN_INVALID
AUTH_REFRESH_TOKEN_REUSE_DETECTED

IDENTITY_USER_NOT_FOUND
IDENTITY_USER_SUSPENDED
IDENTITY_USER_DISABLED

IDENTITY_ORGANIZATION_NOT_FOUND
IDENTITY_ORGANIZATION_SUSPENDED

IDENTITY_MEMBERSHIP_NOT_FOUND
IDENTITY_MEMBERSHIP_SUSPENDED

IDENTITY_PERMISSION_DENIED
IDENTITY_TENANT_ACCESS_DENIED

IDENTITY_POLICY_DENIED
IDENTITY_POLICY_REQUIRES_MFA

IDENTITY_RESOURCE_CONFLICT
IDENTITY_VALIDATION_ERROR

El catálogo definitivo será parte del contrato OpenAPI.

37. Authentication Headers

Las APIs protegidas utilizarán:

Authorization: Bearer <access_token>

El access token será un JWT de corta duración, firmado asimétricamente.

La API deberá validar como mínimo los elementos establecidos por la política de JWT:

issuer;
audience;
expiration;
signature;
token identifier;
session context cuando corresponda.
38. JWT Response Contract

El API no debe devolver un catálogo completo de permisos dentro del token.

Los claims conceptuales son:

sub
iss
aud
iat
exp
jti
session_id

La información sensible no debe introducirse en el JWT.

39. Security Headers

La implementación deberá alinearse con las políticas de seguridad de plataforma para headers HTTP.

Como mínimo deberán contemplarse:

protección de transporte;
políticas de contenido;
protección contra framing;
cache control en respuestas sensibles.

Los valores exactos deben definirse en Security Standards, no duplicarse arbitrariamente aquí.

40. Rate Limiting

Los endpoints especialmente sensibles deberán estar sujetos a rate limiting:

/login
/refresh
/mfa/*
/password/*
/authorization/*

Los valores exactos no se inventan en esta IS.

Deberán ser definidos por política de seguridad/plataforma.

41. Correlation ID

Cada request deberá poder correlacionarse mediante:

X-Correlation-ID: <id>

o el mecanismo estándar que adopte API Platform.

El mismo contexto deberá propagarse hacia:

logs;
audit events;
security events;
domain/integration events;
tracing.
42. Traceability

La API deberá conservar:

Request
 ↓
Correlation ID
 ↓
Application
 ↓
Domain
 ↓
Outbox
 ↓
Event Platform

Esto permite investigar una operación completa.

43. API Security

Todas las operaciones deberán pasar por:

Authentication
        ↓
Tenant Context
        ↓
Authorization
        ↓
Security Policy
        ↓
Application Use Case

No deberá existir:

HTTP endpoint
   ↓
Repository

saltándose Application/Domain/Authorization.

44. Authorization Enforcement

La protección de endpoints no debe depender exclusivamente de:

if user.role == "admin"

La autorización deberá utilizar el modelo establecido:

RBAC
+
ABAC
+
Tenant Isolation
+
Ownership
+
Security Policy Engine

Esto corresponde a IS-IDENTITY-006 e IS-IDENTITY-011.

45. Ownership

Cuando un recurso tenga propietario, la API deberá permitir que Authorization determine:

actor == owner

o cualquier regla de ownership definida por el dominio.

No debe asumirse que tener una permission implica automáticamente acceso a cualquier recurso.

46. Sensitive Operations

Las operaciones sensibles podrán requerir:

REAUTHENTICATE
REQUIRE_MFA
CHALLENGE

Ejemplos:

modificar MFA;
eliminar/revocar factores;
cambiar permisos;
asignar roles;
modificar políticas;
revocar todas las sesiones;
operaciones administrativas.

La decisión concreta dependerá de Security Policy Engine.

47. API y AI Agents

Los AI Agents que consuman Identity API deberán autenticarse y autorizarse como actores explícitos.

No se debe permitir:

AI Agent
   ↓
bypass Identity

Un AI Agent deberá operar con:

actor identity
+
tenant
+
authorization context
+
policy

y únicamente sobre el contexto autorizado.

48. OpenAPI

El contrato definitivo deberá expresarse mediante:

OpenAPI 3.1

Deberá documentar:

endpoints;
parameters;
request bodies;
response schemas;
errors;
authentication;
authorization;
security schemes;
pagination;
idempotency;
examples;
event-related responses cuando corresponda.
49. Schema Organization

Se recomienda organizar schemas conceptualmente:

Identity
├── User
├── Organization
├── Membership
├── Role
├── Permission
├── Session
├── RefreshToken
├── MFAFactor
├── MFAChallenge
├── RecoveryCode
├── TrustedDevice
├── SecurityPolicy
├── AuditEvent
├── SecurityEvent
└── Error

Los secretos nunca deben formar parte de schemas de respuesta.

50. API Compatibility

Una modificación compatible puede incluir:

nuevo campo opcional

Una modificación potencialmente incompatible:

renombrar campo
eliminar campo
cambiar tipo
cambiar semántica

deberá gestionarse mediante versionado apropiado.

51. API Contract Testing

Antes de implementación final deberá existir:

Contract tests

Verificar:

Request
 ↓
Expected schema
 ↓
Response

Incluyendo:

status codes;
required fields;
types;
enum values;
security requirements;
errors.
52. Technical Tasks
API Foundation

T01 — Definir API Identity base.
T02 — Definir /api/v1.
T03 — Definir OpenAPI 3.1.
T04 — Definir security schemes.
T05 — Definir correlation strategy.
T06 — Definir API conventions.

Authentication

T07 — Definir login contract.
T08 — Definir MFA_REQUIRED response.
T09 — Definir refresh contract.
T10 — Definir logout contract.
T11 — Definir logout-all contract.
T12 — Definir authentication errors.

Users

T13 — User create schema.
T14 — User read schema.
T15 — User update schema.
T16 — User lifecycle operations.
T17 — User validation errors.

Organizations

T18 — Organization schemas.
T19 — Organization lifecycle APIs.
T20 — Tenant context contract.

Membership

T21 — Membership schemas.
T22 — Membership lifecycle APIs.
T23 — Membership role assignment APIs.

Roles & Permissions

T24 — Role schemas.
T25 — Permission schemas.
T26 — Role CRUD contracts.
T27 — Permission CRUD contracts.
T28 — Role-Permission contracts.
T29 — Membership-Role contracts.

Authorization

T30 — Authorization request schema.
T31 — Authorization response schema.
T32 — Decision enum.
T33 — Authorization error contracts.

Sessions

T34 — Current session API.
T35 — Session listing.
T36 — Session detail.
T37 — Session revoke.
T38 — Session revoke-all.

MFA

T39 — MFA factor schema.
T40 — Challenge schema.
T41 — Enrollment contracts.
T42 — Verification contracts.
T43 — Recovery code contracts.
T44 — Trusted device contracts.

Security Policies

T45 — Security policy schema.
T46 — Policy lifecycle APIs.
T47 — Policy version contract.

Audit/Security

T48 — Audit Event schema.
T49 — Security Event schema.
T50 — Search/filter contract.
T51 — Authorization for audit endpoints.

Cross-cutting

T52 — Pagination.
T53 — Filtering.
T54 — Sorting.
T55 — Idempotency-Key.
T56 — Error taxonomy.
T57 — Rate limiting contract.
T58 — Correlation IDs.
T59 — Security headers.
T60 — OpenAPI documentation.

Testing

T61 — OpenAPI validation tests.
T62 — Contract tests.
T63 — Authentication contract tests.
T64 — Authorization contract tests.
T65 — Tenant isolation tests.
T66 — Error contract tests.
T67 — Idempotency tests.
T68 — Security tests.
T69 — API performance tests.
T70 — Backward compatibility tests.

53. Acceptance Criteria
AC-001 — Versioning

Given la Identity API
When un cliente consume la API
Then debe utilizar un contrato versionado bajo /api/v1.

AC-002 — OpenAPI

Given la Identity API
When se genera la especificación
Then debe cumplir OpenAPI 3.1.

AC-003 — Authentication

Given un usuario válido
When realiza login correctamente
Then debe recibir un resultado AUTHENTICATED y el contexto de sesión correspondiente.

AC-004 — MFA

Given un usuario cuya política exige MFA
When proporciona credenciales válidas
Then la API debe indicar MFA_REQUIRED y no considerar completada la autenticación.

AC-005 — Authorization

Given un usuario sin autorización suficiente
When intenta acceder a un recurso
Then la API debe rechazar la operación.

AC-006 — Tenant Isolation

Given un usuario perteneciente al tenant A
When intenta acceder directamente a un recurso del tenant B
Then la API debe rechazar la operación independientemente de otras permissions.

AC-007 — Errors

Given cualquier error API
When se devuelve la respuesta
Then debe utilizar el contrato estándar error.code, error.message y correlation_id.

AC-008 — Secrets

Given cualquier endpoint Identity
When devuelve información al cliente
Then nunca debe exponer secretos de autenticación, MFA o criptografía.

AC-009 — Idempotency

Given una operación soportada por Idempotency-Key
When el mismo request se repite con la misma key
Then no debe generar efectos de negocio duplicados.

AC-010 — Correlation

Given una solicitud Identity
When se procesa
Then debe poder correlacionarse con logs, audit/security events y eventos publicados.

AC-011 — Session

Given una sesión revocada
When se intenta utilizar un refresh token asociado
Then la operación debe ser rechazada.

AC-012 — Security Policy

Given una operación que requiere MFA
When el usuario no ha satisfecho el requisito
Then la API debe devolver el resultado correspondiente (REQUIRE_MFA, CHALLENGE o equivalente contractual).

54. Definition of Done
 API version /api/v1 definida.
 OpenAPI 3.1 definido.
 Authentication contracts definidos.
 User contracts definidos.
 Organization contracts definidos.
 Membership contracts definidos.
 Role contracts definidos.
 Permission contracts definidos.
 Authorization contracts definidos.
 Session contracts definidos.
 MFA contracts definidos.
 Security Policy contracts definidos.
 Audit contracts definidos.
 Security Event contracts definidos.
 Error contract definido.
 Pagination definida.
 Filtering definido.
 Idempotency definido.
 Correlation definido.
 Security requirements documentados.
 Tenant isolation documentado.
 OpenAPI validation implementado.
 Contract tests implementados.
 Security tests implementados.
 Compatibility tests implementados.
 Documentation publicada.
 Traceability completa.
55. Dependencias
Upstream
ESP-0001
ADR-IDENTITY-001
ADR-IDENTITY-002
ADR-IDENTITY-003
ADR-IDENTITY-005
ADR-IDENTITY-006
ADR-IDENTITY-007
ADR-IDENTITY-008
ADR-IDENTITY-009
ADR-IDENTITY-010
ADR-IDENTITY-011

IS-IDENTITY-001
IS-IDENTITY-002
IS-IDENTITY-003
IS-IDENTITY-004
IS-IDENTITY-005
IS-IDENTITY-006
IS-IDENTITY-007
IS-IDENTITY-008
IS-IDENTITY-009
IS-IDENTITY-010
IS-IDENTITY-011
IS-IDENTITY-012
IS-IDENTITY-013
56. Downstream

IS-IDENTITY-014 habilita directamente:

Source Code
    ↓
API Controllers
    ↓
Application Use Cases
    ↓
OpenAPI
    ↓
API Client SDKs
    ↓
Flutter / Mobile
    ↓
Angular / Web
    ↓
Integration Testing

Por eso esta IS es un punto de control importante antes de comenzar la implementación.

57. Traceability
Nivel	Referencia
Blueprint	API-First Architecture
ESP	ESP-0001
API Architecture	API Contracts
ADR	ADR-IDENTITY-001
Security	ADR-IDENTITY-005
JWT	ADR-IDENTITY-002
Sessions	ADR-IDENTITY-003
Persistence	ADR-IDENTITY-006
Events	ADR-IDENTITY-007
Cryptography	ADR-IDENTITY-008
Token Storage	ADR-IDENTITY-009
Testing	ADR-IDENTITY-010
Runtime	ADR-IDENTITY-011
IS	IS-IDENTITY-014
Technical Tasks	T01–T70
Source Code	Pendiente
Tests	Pendiente

La estructura mantiene la trazabilidad establecida por el Blueprint: ESP → IS → Technical Tasks → Source Code → Testing → Deployment.

58. Decisiones todavía pendientes

Hay varios puntos que deliberadamente no debemos cerrar artificialmente en esta IS:

API-001

Pagination definitiva:

offset vs cursor
API-002

Formato definitivo de filtros y sorting.

API-003

Política exacta de Idempotency-Key.

API-004

Catálogo definitivo de error codes.

API-005

Política definitiva de rate limits.

API-006

Contrato exacto de refresh token para Web y Mobile.

API-007

Contrato definitivo de MFA factors.

API-008

Formato definitivo de Security Policy.

API-009

Versionado mayor de API.

API-010

Estrategia de API Gateway/BFF.

Estas decisiones deben coordinarse con la arquitectura global y no resolverse arbitrariamente dentro de un endpoint.

59. Estado

IS-IDENTITY-014 — Draft

Esta IS completa una pieza especialmente importante del bloque Identity:

001 Foundation
      ↓
002 Users
      ↓
003 Organizations
      ↓
004 Membership
      ↓
005 Authentication
      ↓
006 Authorization
      ↓
007 Roles & Permissions
      ↓
008 Sessions
      ↓
009 Refresh Tokens
      ↓
010 MFA
      ↓
011 Security Policy
      ↓
012 Audit & Security Events
      ↓
013 Domain Events & Outbox
      ↓
👉 014 Identity API Contracts

Con IS-IDENTITY-014 ya tenemos definida la superficie API conceptual de prácticamente todo Identity. El siguiente paso natural del Blueprint es IS-IDENTITY-015 — Identity Persistence, donde se debe convertir el modelo conceptual de entidades y relaciones en la especificación formal de persistencia, PostgreSQL, índices, constraints, transacciones y estrategia de migraciones, sin comenzar todavía a escribir el código de los modelos.
