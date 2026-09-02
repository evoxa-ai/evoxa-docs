ESP-0001 / 10 — Identity API Contracts

Estado: Draft
Dominio: Identity
Código: ESP-0001
Sección: 10 — API Contracts
Parent Specification: ESP-0001 — Identity Engineering Specification

1. Propósito

Este documento define los contratos API del dominio Identity de EVOXA.

Su objetivo es transformar las capacidades definidas en ESP-0001/01–09 en interfaces API explícitas, versionadas, consistentes, seguras y verificables.

La arquitectura de EVOXA establece que:

Las APIs son contratos, no detalles de implementación.

El enfoque es API-First: el contrato debe definirse antes de la implementación, servir como fuente autoritativa para la documentación y ser validado mediante pruebas de contrato.

2. Alcance

Este capítulo cubre APIs para:

Authentication
Users
Organizations
Memberships
Roles
Permissions
Sessions
Tokens
MFA
Security Policies
Audit

No define todavía:

código Python;
código TypeScript;
modelos ORM;
SQL;
infraestructura;
UI.

Esos elementos pertenecen a las fases posteriores del lifecycle de EVOXA.

3. Principios API

Las APIs de Identity deben ser:

RESTful.
Resource-oriented.
Versionadas.
Stateless cuando corresponda.
Secure by default.
Tenant-aware.
Observable.
Idempotentes cuando corresponda.
Documentadas mediante OpenAPI.
Compatibles hacia atrás cuando sea posible.
Consistentes entre dominios.

Esto coincide con la arquitectura API definida para EVOXA.

4. Base URL

La versión inicial será:

/api/v1

Por tanto, Identity utilizará conceptualmente:

/api/v1/identity

Ejemplo:

GET /api/v1/identity/users

La arquitectura de EVOXA establece versionamiento mediante /api/v1 y recomienda una nueva versión mayor para cambios incompatibles.

5. API Categories

Identity puede exponer diferentes categorías:

Public APIs
Internal APIs
Administrative APIs
Security APIs

No todos los endpoints deben ser accesibles por todos los clientes.

6. Authentication API
6.1 Login
POST /api/v1/identity/auth/login

Request conceptual:

{
  "email": "user@example.com",
  "password": "********"
}

Respuesta cuando no se requiere MFA:

{
  "access_token": "...",
  "refresh_token": "...",
  "token_type": "Bearer",
  "expires_in": 900
}

Cuando MFA es requerido:

{
  "mfa_required": true,
  "challenge_id": "...",
  "expires_in": 300
}
7. Refresh Token
POST /api/v1/identity/auth/refresh

Debe permitir obtener un nuevo access token.

Conceptualmente:

Refresh Token
      │
      ▼
Validate
      │
      ▼
Check Revocation
      │
      ▼
Check Rotation
      │
      ▼
Issue Access Token

La arquitectura contempla refresh tokens y recomienda mecanismos de rotación y control de reutilización.

8. Logout
POST /api/v1/identity/auth/logout

Debe invalidar la sesión actual y los tokens asociados según la política definida.

9. Global Logout
POST /api/v1/identity/auth/logout-all

Permite cerrar todas las sesiones activas del usuario.

Debe considerarse una operación sensible y auditable.

10. Authentication Status
GET /api/v1/identity/auth/me

Devuelve información contextual de la identidad autenticada.

Ejemplo:

{
  "user": {
    "id": "...",
    "email": "...",
    "status": "ACTIVE"
  },
  "organization": {
    "id": "...",
    "name": "..."
  },
  "roles": [],
  "permissions": []
}

La respuesta definitiva deberá evitar exponer información que no sea necesaria para el cliente.

11. User API
11.1 Create User
POST /api/v1/identity/users

Request conceptual:

{
  "email": "user@example.com",
  "first_name": "Juan",
  "last_name": "Perez"
}

La creación debe respetar:

Authentication
Authorization
Tenant Policy
Validation
Audit
12. List Users
GET /api/v1/identity/users

Debe soportar:

page
page_size
sort
filter
search
status
organization_id
role

Ejemplo:

GET /api/v1/identity/users?page=1&page_size=25&status=ACTIVE
13. Get User
GET /api/v1/identity/users/{user_id}

Debe validar:

Actor
Tenant
Permission
Ownership / Policy
14. Update User
PATCH /api/v1/identity/users/{user_id}

Solo deben modificarse atributos permitidos.

Cambios sensibles deben activar las políticas correspondientes.

15. Activate User
POST /api/v1/identity/users/{user_id}/activate
16. Suspend User
POST /api/v1/identity/users/{user_id}/suspend
17. Disable User
POST /api/v1/identity/users/{user_id}/disable

Las operaciones de cambio de estado deben generar eventos y auditoría.

18. Organization API
Create Organization
POST /api/v1/identity/organizations
List Organizations
GET /api/v1/identity/organizations
Get Organization
GET /api/v1/identity/organizations/{organization_id}
Update Organization
PATCH /api/v1/identity/organizations/{organization_id}
Suspend Organization
POST /api/v1/identity/organizations/{organization_id}/suspend
Reactivate Organization
POST /api/v1/identity/organizations/{organization_id}/reactivate
19. Membership API

Membership representa la relación:

User
  │
  ▼
Organization
Create Membership
POST /api/v1/identity/organizations/{organization_id}/memberships

Request:

{
  "user_id": "...",
  "role_ids": []
}
List Memberships
GET /api/v1/identity/organizations/{organization_id}/memberships
Get Membership
GET /api/v1/identity/memberships/{membership_id}
Suspend Membership
POST /api/v1/identity/memberships/{membership_id}/suspend
Remove Membership
POST /api/v1/identity/memberships/{membership_id}/remove
20. Role API
Create Role
POST /api/v1/identity/roles

Request:

{
  "name": "Coach",
  "description": "Coach role",
  "scope": "ORGANIZATION"
}
List Roles
GET /api/v1/identity/roles
Get Role
GET /api/v1/identity/roles/{role_id}
Update Role
PATCH /api/v1/identity/roles/{role_id}
Delete Role
DELETE /api/v1/identity/roles/{role_id}

La eliminación de roles críticos puede requerir restricciones adicionales.

21. Permission API
List Permissions
GET /api/v1/identity/permissions
Create Permission
POST /api/v1/identity/permissions

Conceptualmente:

{
  "resource": "clients",
  "action": "read"
}

El modelo de permisos propuesto anteriormente utiliza una convención:

resource.action

por ejemplo:

clients.read
training.update
nutrition.publish
22. Role Permissions
Assign Permission
POST /api/v1/identity/roles/{role_id}/permissions

Request:

{
  "permission_id": "..."
}
Remove Permission
DELETE /api/v1/identity/roles/{role_id}/permissions/{permission_id}

Estas operaciones deben generar auditoría.

23. User Roles
Assign Role
POST /api/v1/identity/memberships/{membership_id}/roles

Request:

{
  "role_id": "..."
}
Remove Role
DELETE /api/v1/identity/memberships/{membership_id}/roles/{role_id}

Esto es preferible a asignar directamente un rol global al User cuando el modelo permite que una persona pertenezca a múltiples organizaciones.

24. Session API
List Sessions
GET /api/v1/identity/sessions

Respuesta conceptual:

{
  "items": [
    {
      "id": "...",
      "device": "...",
      "created_at": "...",
      "last_activity_at": "...",
      "status": "ACTIVE"
    }
  ]
}

Nunca debe devolver tokens completos.

25. Get Session
GET /api/v1/identity/sessions/{session_id}
26. Revoke Session
POST /api/v1/identity/sessions/{session_id}/revoke
27. Revoke All Sessions
POST /api/v1/identity/sessions/revoke-all

Debe requerir autenticación y autorización apropiadas.

28. MFA API

Basado en ESP-0001/07.

List Factors
GET /api/v1/identity/mfa/factors
Begin Enrollment
POST /api/v1/identity/mfa/factors

Request:

{
  "type": "TOTP"
}
Verify Factor
POST /api/v1/identity/mfa/factors/{factor_id}/verify

Request:

{
  "code": "123456"
}

Después de una verificación correcta:

PENDING
   ↓
ACTIVE
29. Revoke MFA Factor
DELETE /api/v1/identity/mfa/factors/{factor_id}

Debe tratarse como operación sensible.

Puede requerir:

Step-Up Authentication

según Security Policy.

30. MFA Challenge
POST /api/v1/identity/mfa/challenges

Request conceptual:

{
  "factor_id": "...",
  "purpose": "LOGIN"
}
31. Verify MFA Challenge
POST /api/v1/identity/mfa/challenges/{challenge_id}/verify

Request:

{
  "code": "123456"
}
32. Recovery Codes
Generate
POST /api/v1/identity/mfa/recovery-codes
Use Recovery Code
POST /api/v1/identity/mfa/recovery

Los códigos nunca deben devolverse después de su creación inicial.

33. Trusted Devices
List
GET /api/v1/identity/mfa/trusted-devices
Revoke
DELETE /api/v1/identity/mfa/trusted-devices/{device_id}
34. Security Policies API

Estas APIs deben estar restringidas a administradores autorizados.

List Policies
GET /api/v1/identity/security-policies
Create Policy
POST /api/v1/identity/security-policies
Get Policy
GET /api/v1/identity/security-policies/{policy_id}
Update Policy
PATCH /api/v1/identity/security-policies/{policy_id}
Activate Policy
POST /api/v1/identity/security-policies/{policy_id}/activate
Deactivate Policy
POST /api/v1/identity/security-policies/{policy_id}/deactivate
35. Audit API
Query Audit
GET /api/v1/identity/audit

Filtros:

actor_id
organization_id
user_id
event_type
action
result
target_type
target_id
date_from
date_to
correlation_id
36. Audit Detail
GET /api/v1/identity/audit/{audit_event_id}

El acceso debe estar fuertemente protegido.

37. Audit Export
POST /api/v1/identity/audit/export

Request conceptual:

{
  "format": "JSON",
  "filters": {
    "date_from": "...",
    "date_to": "..."
  }
}

La exportación debe generar su propio evento de auditoría.

38. Standard Response

EVOXA debe mantener respuestas consistentes.

Ejemplo:

{
  "data": {
    "id": "...",
    "name": "..."
  }
}

Para colecciones:

{
  "data": [],
  "pagination": {
    "page": 1,
    "page_size": 25,
    "total": 100
  }
}

La arquitectura de API contempla paginación, filtrado, ordenamiento y búsqueda.

39. Standard Error

Todos los errores deben seguir el formato definido por EVOXA:

{
  "error": {
    "code": "IDENTITY_ACCESS_DENIED",
    "message": "Access denied",
    "correlation_id": "..."
  }
}

La arquitectura API define explícitamente este formato estándar.

40. Error Codes

Se propone una nomenclatura:

IDENTITY_AUTHENTICATION_FAILED
IDENTITY_INVALID_TOKEN
IDENTITY_TOKEN_EXPIRED
IDENTITY_ACCESS_DENIED
IDENTITY_TENANT_ACCESS_DENIED
IDENTITY_USER_NOT_FOUND
IDENTITY_ORGANIZATION_NOT_FOUND
IDENTITY_MEMBERSHIP_NOT_FOUND
IDENTITY_ROLE_NOT_FOUND
IDENTITY_PERMISSION_NOT_FOUND
IDENTITY_MFA_REQUIRED
IDENTITY_MFA_FAILED
IDENTITY_MFA_CHALLENGE_EXPIRED
IDENTITY_SESSION_NOT_FOUND
IDENTITY_SESSION_REVOKED
IDENTITY_POLICY_DENIED
IDENTITY_RATE_LIMITED

El catálogo definitivo debe establecerse antes de implementar.

41. HTTP Status Codes

Uso conceptual:

Código	Uso
200	Operación exitosa
201	Recurso creado
202	Operación aceptada/asíncrona
204	Operación exitosa sin contenido
400	Request inválido
401	No autenticado
403	No autorizado
404	Recurso no encontrado
409	Conflicto
422	Validación semántica
429	Rate limit
500	Error interno
42. Idempotency

Las operaciones POST susceptibles a reintentos deben soportar:

Idempotency-Key: <unique-key>

Esto es especialmente importante para:

creación de usuarios;
creación de organizaciones;
memberships;
MFA enrollment;
operaciones administrativas;
recovery.

La arquitectura API establece el uso de Idempotency-Key para operaciones de escritura donde corresponda.

43. Authentication

Las APIs protegidas utilizarán:

Authorization: Bearer <access_token>

La arquitectura contempla OAuth2.1/OIDC, JWT Bearer, refresh tokens, API keys y service accounts según el tipo de integración.

44. Authorization

Una petición protegida debe pasar conceptualmente por:

Bearer Token
     ↓
Authentication
     ↓
Tenant Context
     ↓
RBAC
     ↓
ABAC
     ↓
Ownership / Resource Policy
     ↓
Security Policy
     ↓
ALLOW

El endpoint no debe confiar únicamente en que el token sea válido.

45. Tenant Context

Cuando corresponda, el contexto debe derivarse de la identidad/membership autenticada.

No se debe confiar ciegamente en un:

X-Tenant-ID

enviado por el cliente.

Si se utiliza un header de contexto, debe validarse contra las memberships reales del usuario.

46. Correlation ID

Cada request debe tener un identificador de correlación.

Ejemplo:

X-Correlation-ID: 01H...

Si el cliente no lo proporciona, la plataforma puede generarlo.

Debe aparecer en:

Response
Logs
Audit
Events
Tracing
47. OpenAPI

Identity debe documentarse mediante:

OpenAPI 3.1

La arquitectura API de EVOXA establece OpenAPI 3.1 como estándar de contrato/documentación.

Conceptualmente:

evoxa-docs
   │
   ▼
Identity API Contract
   │
   ▼
OpenAPI 3.1
   │
   ├── Documentation
   ├── Client Generation
   ├── Contract Tests
   └── Validation
48. API Contract First

El proceso correcto será:

ESP-0001
   ↓
API Contract
   ↓
OpenAPI
   ↓
Contract Review
   ↓
Implementation
   ↓
Contract Tests

No:

Code
 ↓
Guess API
 ↓
Documentation

Esto es especialmente importante para evitar que frontend, mobile y backend evolucionen con contratos incompatibles.

49. API Security

Todos los endpoints deben considerar:

HTTPS.
Authentication.
Authorization.
Input validation.
Rate limiting.
Audit.
CORS donde corresponda.
CSRF protection cuando corresponda.
Tenant isolation.
Secure error handling.

Estos controles están alineados con la arquitectura API y Security de EVOXA.

50. API → Events

Las operaciones de escritura relevantes deben generar eventos.

Ejemplo:

POST /users
      │
      ▼
UserCreated
      │
      ├── Audit
      ├── Analytics
      └── Other Consumers

Esto sigue el enfoque Event-Driven de EVOXA, donde acciones de negocio significativas generan eventos inmutables.

51. API → Audit

Ejemplo:

PATCH /users/{id}
       │
       ▼
Authorization
       │
       ▼
Business Operation
       │
       ├── Domain Event
       │
       └── Audit Event

Esto permite mantener trazabilidad completa.

52. API → MFA → Session

Login con MFA:

POST /auth/login
        │
        ▼
Authentication
        │
        ▼
MFA Required
        │
        ▼
POST /mfa/challenges
        │
        ▼
POST /mfa/challenges/{id}/verify
        │
        ▼
Session Created
        │
        ▼
Tokens Issued
53. API → Security Policy

Operación sensible:

PATCH /users/{id}/security
            │
            ▼
      Security Policy
            │
       ┌────┴────┐
       ▼         ▼
     ALLOW    REQUIRE_MFA
                 │
                 ▼
              Challenge
                 │
                 ▼
              Continue
54. API Versioning

La primera versión:

/api/v1

Cambios compatibles:

v1

Cambios incompatibles:

/api/v2

No debe utilizarse:

/api/v1.1
/api/v1.2

como mecanismo de versionamiento público.

55. Backward Compatibility

Cuando sea posible:

Additive Changes

son preferibles a:

Breaking Changes

Ejemplo aceptable:

Agregar campo opcional

Ejemplo potencialmente incompatible:

Eliminar campo existente
Cambiar significado
Cambiar tipo
Cambiar estructura
56. Rate Limiting por API

Especial atención en:

/auth/login
/auth/refresh
/mfa/challenges
/mfa/.../verify
/auth/password-reset

Las APIs de seguridad deben tener límites específicos.

57. API Testing

Cada endpoint deberá validarse mediante:

Contract Tests
OpenAPI
   ↓
Request validation
Response validation
Status validation
Security Tests
Authentication
Authorization
Tenant Isolation
Privilege Escalation
Integration Tests
API
 ↓
Application
 ↓
Domain
 ↓
Infrastructure
E2E

Flujos completos de usuario.

La arquitectura de testing exige validación de schema, status codes, authentication, authorization, validation, errors e idempotency.

58. Matriz inicial de endpoints
Área	Endpoint	Método
Auth	/auth/login	POST
Auth	/auth/refresh	POST
Auth	/auth/logout	POST
Auth	/auth/logout-all	POST
Auth	/auth/me	GET
Users	/users	GET/POST
Users	/users/{id}	GET/PATCH
Users	/users/{id}/activate	POST
Users	/users/{id}/suspend	POST
Users	/users/{id}/disable	POST
Organizations	/organizations	GET/POST
Organizations	/organizations/{id}	GET/PATCH
Memberships	/organizations/{id}/memberships	GET/POST
Memberships	/memberships/{id}	GET
Roles	/roles	GET/POST
Roles	/roles/{id}	GET/PATCH/DELETE
Permissions	/permissions	GET/POST
Sessions	/sessions	GET
Sessions	/sessions/{id}/revoke	POST
MFA	/mfa/factors	GET/POST
MFA	/mfa/factors/{id}/verify	POST
MFA	/mfa/challenges	POST
MFA	/mfa/challenges/{id}/verify	POST
MFA	/mfa/recovery	POST
Security	/security-policies	GET/POST
Audit	/audit	GET
Audit	/audit/{id}	GET
Audit	/audit/export	POST

Esta es una matriz conceptual inicial, no todavía el contrato OpenAPI definitivo.

59. ADRs necesarios
ADR-IDENTITY-027

Identity API Resource Model

Definir definitivamente la jerarquía de recursos.

ADR-IDENTITY-028

Authentication API Contract

Definir:

login;
MFA;
refresh;
logout;
recovery.
ADR-IDENTITY-029

User & Membership API Model

Resolver definitivamente si roles pertenecen al User, Membership u otra abstracción.

ADR-IDENTITY-030

API Error Catalog

Definir códigos oficiales.

ADR-IDENTITY-031

API Pagination & Filtering

Definir formato común.

ADR-IDENTITY-032

API Idempotency

Definir operaciones obligatorias y estrategia de almacenamiento.

ADR-IDENTITY-033

API Authentication Mechanisms

Definir:

OAuth2.1
OIDC
JWT
API Keys
Service Accounts

y cuándo utilizar cada uno.

ADR-IDENTITY-034

API Tenant Context

Definir cómo se determina y propaga el tenant.

ADR-IDENTITY-035

API Versioning

Formalizar estrategia de evolución y compatibilidad.

60. Criterios de aceptación

ESP-0001/10 estará listo para implementación cuando:

 Todos los recursos de Identity estén identificados.
 Todos los endpoints estén definidos.
 Request schemas estén definidos.
 Response schemas estén definidos.
 Error schemas estén definidos.
 HTTP status codes estén definidos.
 Authentication esté definida.
 Authorization esté definida.
 Tenant context esté definido.
 Pagination esté definida.
 Filtering esté definido.
 Sorting esté definido.
 Search esté definido.
 Idempotency esté definida.
 Correlation IDs estén definidos.
 Audit esté definido.
 Events estén definidos.
 Rate limiting esté definido.
 OpenAPI 3.1 esté definido.
 ADRs estén aprobados.
 Contract tests estén definidos.
61. Trazabilidad
BP-0002 Reference Architecture
          │
          ▼
20 — API Architecture
          │
          ▼
19 — API Contracts
          │
          ▼
ESP-0001 Identity
          │
 ┌────────┼─────────────┐
 ▼        ▼             ▼
Auth    Users        Organizations
 │        │             │
 ▼        ▼             ▼
MFA     Roles       Memberships
 │        │             │
 └────────┼─────────────┘
          ▼
    Security Policies
          │
          ▼
        Audit
          │
          ▼
    OpenAPI 3.1
          │
          ▼
 Implementation Stories
          │
          ▼
      Technical Tasks
          │
          ▼
        Source Code
          │
          ▼
         Tests

Esto respeta el lifecycle oficial definido por el Architecture Map: Blueprint → ESP → Implementation Stories → Technical Tasks → Source Code → Testing → Deployment → Monitoring → Continuous Evolution.

62. Estado completo de ESP-0001
#	Sección	Estado
01	Identity Domain Model	Draft
02	Identity Architecture	Draft
03	Authentication & Authorization	Draft
04	User & Organization Management	Draft
05	Roles & Permissions	Draft
06	Sessions & Token Management	Draft
07	Multi-Factor Authentication	Draft
08	Security Policies	Draft
09	Audit & Compliance	Draft
10	Identity API Contracts	Draft
Punto importante

Con 01–10 ya tenemos prácticamente definido el diseño conceptual completo de Identity. Pero todavía no debemos saltar directamente al código.

El siguiente paso lógico es continuar con:

ESP-0001 / 11 — Identity Persistence & Data Model

Ahí podemos transformar todo lo anterior en el modelo de persistencia definitivo:

User
Organization
Membership
Role
Permission
RolePermission
Session
RefreshToken
MFAFactor
MFAChallenge
RecoveryCode
TrustedDevice
SecurityPolicy
AuditEvent

y definir:

PK / FK
Indexes
Unique Constraints
Tenant Isolation
Lifecycle States
Soft Delete / Retention
Versioning
Audit Metadata
Relationships
Transaction Boundaries

Después de eso podremos seguir con Events, Integration, Testing y finalmente Implementation Stories, dejando Identity listo para comenzar la programación de evoxa-platform de manera ordenada y trazable.
