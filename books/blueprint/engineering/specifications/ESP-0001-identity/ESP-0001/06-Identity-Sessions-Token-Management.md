ESP-0001 / 06 — Identity Sessions & Token Management

Document ID: ESP-0001/06
Module: Identity
Version: 1.0.0
Status: Draft
Parent: ESP-0001 — Identity
Previous Specifications:

ESP-0001/01 — Identity Domain Model
ESP-0001/02 — Identity Architecture
ESP-0001/03 — Identity Authentication & Authorization
ESP-0001/04 — User & Organization Management
ESP-0001/05 — Roles & Permissions

Architecture Parent: BP-0002 — Reference Architecture
Repository: evoxa-platform
Target Path: apps/api/app/domains/identity/

1. Purpose

Este documento define el modelo de Sessions & Token Management de EVOXA.

Su objetivo es establecer cómo la plataforma:

crea sesiones autenticadas;
administra sesiones activas;
emite Access Tokens;
emite Refresh Tokens;
valida tokens;
renueva credenciales;
revoca tokens;
revoca sesiones;
detecta reutilización de Refresh Tokens;
controla sesiones concurrentes;
administra logout;
protege credenciales;
registra eventos de seguridad;
integra sesiones con el contexto de tenant y autorización.

La arquitectura API de EVOXA contempla JWT Bearer, refresh tokens, API keys y service accounts, mientras que la arquitectura de Identity establece Session Management como capability de plataforma.

2. Scope

Este documento cubre:

Session
Access Token
Refresh Token
JWT
Token Validation
Token Rotation
Token Revocation
Session Revocation
Logout
Global Logout
Concurrent Sessions
Device Sessions
Token Security
Session Security
Token Events
Session Audit

No define todavía en profundidad:

MFA
API Keys
Service Accounts
External Identity Providers
Password Reset

Estos tendrán especificaciones propias.

3. Fundamental Model

Debemos distinguir:

USER
  │
  ▼
SESSION
  │
  ├── ACCESS TOKEN
  │
  └── REFRESH TOKEN

Una sesión representa el contexto autenticado.

Los tokens son las credenciales que permiten mantener ese contexto.

4. Session Definition

Una Session representa una instancia autenticada de un usuario.

Ejemplo:

User
 │
 ├── Session A → Web
 ├── Session B → Mobile
 └── Session C → Tablet

Una sesión puede ser:

ACTIVE
EXPIRED
REVOKED

Los estados exactos deben quedar consolidados en el modelo definitivo.

5. Session Attributes

Propuesta:

Campo	Descripción
id	Identificador único
user_id	Usuario propietario
organization_id	Contexto organizacional
status	Estado
created_at	Fecha de creación
expires_at	Expiración
last_activity_at	Última actividad
revoked_at	Fecha de revocación
revocation_reason	Motivo
device_id	Identificador de dispositivo, si aplica
ip_address	Información de origen protegida
user_agent	Información del cliente
version	Versión

Estos campos son Derived Design. El Blueprint no proporciona un esquema físico completo de Session.

6. Session Lifecycle

Propuesta:

             ┌─────────┐
             │ CREATED │
             └────┬────┘
                  │
                  ▼
             ┌─────────┐
             │ ACTIVE  │
             └────┬────┘
                  │
          ┌───────┼────────┐
          │       │        │
       logout   expire   revoke
          │       │        │
          ▼       ▼        ▼
      REVOKED  EXPIRED  REVOKED

Una sesión terminada no debe volver a convertirse en una sesión activa.

7. Session Creation

Una sesión se crea después de una autenticación válida:

Credentials
    │
    ▼
Authentication
    │
    ▼
Account Validation
    │
    ▼
MFA if required
    │
    ▼
Create Session
    │
    ▼
Issue Tokens

Esto conecta directamente con ESP-0001/03.

8. Session Context

La sesión debe mantener asociación con:

User
Organization
Authentication Context
Security Metadata

Conceptualmente:

Session
├── user_id
├── organization_id
├── authentication_method
├── created_at
├── expires_at
└── security_context
9. Access Token

El Access Token representa una credencial temporal para acceder a APIs protegidas.

Flujo:

Client
   │
   │ Authorization: Bearer TOKEN
   ▼
API Gateway / API
   │
   ▼
Token Validation
   │
   ▼
Authentication Context

La API Architecture establece JWT Bearer como mecanismo contemplado.

10. JWT

JWT será tratado como un formato posible para Access Tokens dentro de la arquitectura definida.

Conceptualmente:

JWT
├── Header
├── Payload
└── Signature

La plataforma debe validar:

Signature
Issuer
Audience
Expiration
Not Before
Required Claims

Los valores concretos de issuer/audience y claims deben definirse en API Contracts y Security Configuration.

11. JWT Claims

El token deberá contener solamente la información necesaria para establecer el contexto de seguridad.

Conceptualmente:

sub
iss
aud
iat
exp
jti
session_id
tenant/context claims
security claims

No se debe convertir el JWT en un almacén de datos del usuario.

No deberían incluirse innecesariamente:

Password
MFA secrets
Sensitive profile information
Private health data
Full permissions catalog
12. Token Lifetime

La arquitectura exige credenciales con duración limitada, pero el Blueprint revisado no fija valores concretos para:

Access Token TTL
Refresh Token TTL
Session TTL

Por lo tanto, no vamos a inventar esos valores en esta especificación.

Deberán definirse mediante:

ADR-IDENTITY-034 — Token & Session Lifetimes

13. Refresh Token

El Refresh Token permite obtener nuevas credenciales de acceso.

Flujo:

Access Token
    │
    ▼
Expired
    │
    ▼
Refresh Token
    │
    ▼
Validate
    │
    ▼
Validate Session
    │
    ▼
Issue New Access Token

La existencia de refresh tokens forma parte del contrato arquitectónico de APIs.

14. Refresh Token Storage

No se debe almacenar necesariamente el Refresh Token en texto plano.

La estrategia recomendada es almacenar una representación segura que permita verificarlo y revocarlo.

Conceptualmente:

Client
  │
  │ Refresh Token
  ▼
Identity
  │
  ▼
Secure Verification
  │
  ▼
Stored Token Record

El mecanismo criptográfico exacto debe definirse en Security/Persistence Specification.

15. Refresh Token Rotation

Se propone Refresh Token Rotation.

Flujo:

Refresh Token A
      │
      ▼
   /refresh
      │
      ├── invalidate A
      │
      └── issue B

Resultado:

A = REVOKED
B = ACTIVE

Esta es una recomendación de diseño derivada y debe formalizarse mediante ADR.

16. Token Family

Para detectar reutilización podemos agrupar Refresh Tokens:

Token Family
│
├── Token A
├── Token B
├── Token C
└── Token D

Si aparece nuevamente:

Token A

después de haber sido revocado:

Reuse Detected
      │
      ▼
Security Incident
      │
      ▼
Revoke Token Family
      │
      ▼
Revoke Session

Esta estrategia es Derived Design, no una exigencia textual del Blueprint.

17. Refresh Token Reuse Detection

Caso:

A → refresh → B

Después:

A → refresh

El sistema detecta:

A already revoked

Resultado:

DENY
+
Security Event
+
Session Revocation

El comportamiento exacto debe ser fijado en ADR.

18. Token Revocation

Un token puede ser revocado por:

Logout
Session Revocation
Password Change
Security Incident
Account Suspension
Membership Removal
Token Reuse Detection
Administrative Action

No todas estas causas deben necesariamente invalidar todas las sesiones; la política exacta debe definirse por caso.

19. Session Revocation

Revocar una sesión significa:

Session
   ↓
REVOKED

y todas las credenciales asociadas deben dejar de ser válidas según la estrategia de validación.

20. Logout

Logout:

POST /api/v1/auth/logout

Flujo:

Request
  │
  ▼
Authenticate Session
  │
  ▼
Revoke Session
  │
  ▼
Revoke Refresh Credentials
  │
  ▼
Audit
  │
  ▼
LogoutCompleted

El endpoint es una propuesta y deberá consolidarse en API Contracts.

21. Logout All

EVOXA debería soportar conceptualmente:

Logout Current Session
Logout All Sessions

Ejemplo:

User
 ├── Session A
 ├── Session B
 └── Session C

Logout All
 ├── A → revoked
 ├── B → revoked
 └── C → revoked

La disponibilidad de esta capacidad debe ser definida en API Contracts.

22. Session Listing

El usuario podría consultar sus sesiones activas:

GET /api/v1/auth/sessions

Ejemplo conceptual:

Current session
Chrome / Windows
Last active: ...

Mobile
iPhone
Last active: ...

Tablet
...

Esta capacidad mejora:

seguridad;
control del usuario;
detección de sesiones desconocidas.

El endpoint es una propuesta.

23. Remote Session Revocation

El usuario debería poder revocar una sesión concreta:

DELETE /api/v1/auth/sessions/{session_id}

Flujo:

User
 │
 ▼
Select Session
 │
 ▼
Revoke
 │
 ▼
Session REVOKED

Esto requiere controles para impedir revocar arbitrariamente sesiones de otros usuarios.

24. Concurrent Sessions

La arquitectura debe soportar múltiples sesiones.

User
 │
 ├── Web
 ├── Mobile
 ├── Tablet
 └── Other

El Blueprint revisado no define un número máximo de sesiones.

Por tanto:

ADR-IDENTITY-035 — Concurrent Session Policy

debe determinar si existe:

Unlimited
Fixed Maximum
Configurable per Organization
Configurable per User
25. Device Sessions

Una sesión puede asociarse opcionalmente a un dispositivo:

Session
├── device_id
├── device_type
├── platform
└── user_agent

Esto es útil para:

Session Management
Security Monitoring
User Session UI

No debemos convertir la identificación de dispositivo en un mecanismo de seguridad único.

26. IP Address

La IP puede almacenarse como metadata de seguridad:

Session
 └── ip_address

pero debe tratarse como dato potencialmente sensible.

No debería aparecer innecesariamente en:

Logs
Analytics
Events
API Responses

La política definitiva corresponde a Privacy/Security.

27. User Agent

El User Agent puede utilizarse para:

Security Telemetry
Session Display
Incident Investigation

pero tampoco debe considerarse una prueba absoluta de identidad.

28. Session Expiration

Una sesión puede expirar por:

Absolute expiration
Idle expiration
Security revocation
Account suspension

La estrategia exacta deberá ser configurable.

29. Absolute Expiration

Una sesión puede tener una fecha máxima:

created_at
     +
maximum lifetime
     =
expires_at

Al llegar a expires_at:

Session → EXPIRED
30. Idle Expiration

Opcionalmente:

last_activity_at

puede utilizarse para determinar inactividad.

Conceptualmente:

Now - last_activity_at > idle_timeout

→

Session EXPIRED

Esta estrategia debe ser evaluada por Security y UX.

31. Session Renewal

Una sesión puede permanecer activa mientras se utilice un Refresh Token válido.

Pero:

Session Lifetime
≠
Access Token Lifetime

Es importante mantener esta separación.

32. Account Suspension

Cuando un usuario pasa a:

SUSPENDED

las sesiones activas deben quedar sujetas a una política de revocación.

La recomendación de seguridad es:

User Suspended
       │
       ▼
Revoke Active Sessions

pero debe registrarse como decisión formal:

ADR-IDENTITY-036 — Session Revocation on Account Suspension

33. Membership Removal

Si:

User
 └── Organization A

pierde su membership:

Membership → REMOVED

las credenciales que permiten actuar en Organization A deben dejar de proporcionar acceso a ese tenant.

Esto es obligatorio desde la perspectiva de tenant isolation.

La revocación completa de la sesión global debe determinarse por política.

34. Organization Suspension

Si:

Organization A

es suspendida:

Organization → SUSPENDED

el acceso contextual de sus usuarios deberá ser rechazado.

User
 ↓
Membership
 ↓
Organization SUSPENDED
 ↓
DENY

No implica necesariamente eliminar las sesiones o identidades globales.

35. Token Validation

Cada request protegida debe validar el Access Token.

Request
  │
  ▼
Extract Bearer Token
  │
  ▼
Validate JWT
  │
  ├── Signature
  ├── Issuer
  ├── Audience
  ├── Expiration
  └── Required Claims
  │
  ▼
Authentication Context
36. Session Validation

Cuando la arquitectura requiera validación de sesión:

JWT
 │
 ▼
session_id
 │
 ▼
Session
 │
 ├── ACTIVE → Continue
 │
 ├── EXPIRED → DENY
 │
 └── REVOKED → DENY

Esto permite invalidar sesiones incluso cuando el JWT todavía no ha alcanzado su exp.

La necesidad exacta de introspección online debe resolverse mediante ADR.

37. Stateless vs Stateful

La arquitectura de EVOXA favorece servicios stateless para escalabilidad.

Por ello:

Access Token

puede validarse localmente.

Pero:

Session Revocation
Refresh Token
Security Incidents

pueden requerir estado centralizado.

Esto produce un modelo híbrido:

API
 │
 ├── Stateless Access Token Validation
 │
 └── Stateful Session / Refresh Management

Esta es una decisión arquitectónica recomendada.

38. Redis Role

Redis puede utilizarse para información temporal de alta frecuencia:

Session state
Token metadata
Revocation state
Rate limiting
Temporary security state

La arquitectura contempla Redis como componente de infraestructura.

La decisión exacta de qué datos deben residir en Redis pertenece a Persistence/Infrastructure.

39. PostgreSQL Role

PostgreSQL puede ser el almacenamiento durable para:

Sessions
Refresh Token Records
Security Audit

El esquema físico definitivo se definirá posteriormente.

40. Session Repository

Interfaz propuesta:

SessionRepository
├── create()
├── get_by_id()
├── list_by_user()
├── revoke()
├── revoke_all_for_user()
├── revoke_for_organization()
└── update_activity()

Estos métodos son una propuesta y no contratos definitivos.

41. Refresh Token Repository

Propuesta:

RefreshTokenRepository
├── create()
├── get()
├── revoke()
├── rotate()
├── detect_reuse()
└── revoke_family()

La implementación debe garantizar concurrencia segura.

42. Race Conditions

Un escenario importante:

Client A → Refresh Token A
Client B → Refresh Token A

ambos llegan simultáneamente.

No debemos permitir:

A → B
A → C

como dos refresh válidos.

La operación de rotación debe ser atómica.

Conceptualmente:

BEGIN
   Validate A
   Mark A consumed
   Create B
COMMIT

Esto es un requisito técnico derivado de la estrategia de rotation propuesta.

43. Token Theft

Si un Refresh Token es robado:

Attacker
   │
   ▼
Refresh Token
   │
   ▼
/refresh

el sistema debe poder:

Detect
Reject
Revoke
Audit
Alert

si la política de seguridad determina que existe reutilización o comportamiento sospechoso.

44. Security Incident Flow
Suspicious Token
      │
      ▼
Detection
      │
      ▼
Security Event
      │
      ▼
Token Revocation
      │
      ▼
Session Revocation
      │
      ▼
Audit / Monitoring

Esto conecta Identity con Security y Observability.

45. Token Leakage Protection

Nunca se deben incluir tokens en:

Application Logs
Error Messages
Analytics
Tracing
Audit Payloads
Domain Events

Debe registrarse únicamente información segura como:

token_id / jti
session_id
user_id
tenant_id
event type
correlation_id
46. Transport Security

Todas las operaciones relacionadas con sesiones y tokens requieren transporte seguro.

La arquitectura de seguridad establece:

TLS 1.3

para comunicaciones.

47. Token Storage on Clients

Este documento define el contrato de seguridad, no la implementación de cada cliente.

Pero el principio será:

Tokens
 ↓
Secure Storage

Nunca:

Tokens
 ↓
Plain Text Logs

ni almacenamiento inseguro sin una justificación arquitectónica.

La implementación concreta para:

Web
Mobile
Desktop

se definirá en las respectivas especificaciones de plataforma.

48. Browser Security

Para aplicaciones web, la estrategia definitiva deberá decidir:

HttpOnly Cookies
vs
Authorization Header

y mecanismos complementarios:

SameSite
Secure
CSRF Protection

No se fija todavía una única estrategia porque el Blueprint no define esta implementación concreta.

49. Mobile Security

En mobile:

Access Token
Refresh Token
      ↓
Platform Secure Storage

La implementación concreta corresponde a Mobile Architecture.

50. Events

Eventos propuestos:

SessionCreated
SessionRevoked
SessionExpired

TokenIssued
TokenRefreshed
TokenRevoked
TokenReuseDetected

LogoutCompleted
LogoutAllCompleted

Estos nombres son Derived Design.

Event Architecture establece que los eventos deben representar hechos ocurridos y seguir un envelope común.

51. SessionCreated

Conceptualmente:

{
  "event_type": "SessionCreated",
  "version": 1,
  "resource": {
    "session_id": "..."
  },
  "actor": "...",
  "tenant_id": "...",
  "timestamp": "...",
  "metadata": {
    "correlation_id": "..."
  }
}

No debe incluir tokens.

52. TokenRefreshed

Debe registrar el hecho de renovación sin revelar credenciales:

TokenRefreshed
├── user_id
├── session_id
├── token_id
├── timestamp
└── correlation_id

Nunca:

refresh_token = "..."
53. TokenReuseDetected

Este es un evento de seguridad importante:

TokenReuseDetected
      │
      ├── user
      ├── session
      ├── token identifier
      ├── timestamp
      └── correlation

Puede alimentar Security Monitoring.

54. Audit

Operaciones críticas:

Login
Logout
Session creation
Session revocation
Token refresh
Token reuse
Global logout
Administrative revocation

deben ser auditables.

La arquitectura de seguridad exige auditabilidad y la arquitectura de observabilidad contempla audit logs.

55. Authorization Interaction

Session Management no reemplaza Authorization.

Flujo completo:

Access Token
    │
    ▼
Authentication
    │
    ▼
Session Context
    │
    ▼
Tenant Context
    │
    ▼
RBAC
    │
    ▼
ABAC
    │
    ▼
Resource Policy
    │
    ▼
ALLOW / DENY

Esto conecta directamente ESP-0001/03 con ESP-0001/05.

56. API Operations

Propuesta inicial:

POST   /api/v1/auth/login
POST   /api/v1/auth/refresh
POST   /api/v1/auth/logout

GET    /api/v1/auth/sessions
DELETE /api/v1/auth/sessions/{id}

POST   /api/v1/auth/logout-all

No son todavía contratos oficiales.

La definición formal corresponde a:

ESP-0001 / 13 — Identity API Contracts

siguiendo API-First.

57. Error Codes

Propuesta:

INVALID_ACCESS_TOKEN
EXPIRED_ACCESS_TOKEN
INVALID_REFRESH_TOKEN
EXPIRED_REFRESH_TOKEN
REVOKED_REFRESH_TOKEN
SESSION_NOT_FOUND
SESSION_EXPIRED
SESSION_REVOKED
TOKEN_REUSE_DETECTED
SESSION_ACCESS_DENIED

Los códigos definitivos deberán consolidarse en API Contracts.

58. Performance

La arquitectura de EVOXA establece objetivos como:

Authentication P95 < 500 ms
General API P95 < 300 ms

La implementación debe evitar que cada request requiera operaciones costosas innecesarias.

59. Scalability

El modelo debe funcionar con múltiples instancias:

                    Load Balancer
                         │
             ┌───────────┼───────────┐
             ▼           ▼           ▼
         API Node 1  API Node 2  API Node 3
             │           │           │
             └───────────┼───────────┘
                         │
                  Shared Session State
                    PostgreSQL/Redis

Esto permite escalar horizontalmente.

60. Security Principles

Este componente debe respetar:

Zero Trust
Least Privilege
Secure by Default
Defense in Depth
Privacy by Design
Continuous Verification
Auditability

Estos principios están definidos en Security Architecture.

61. Testing Requirements
Session
Create
Expire
Revoke
Revoke all
Concurrent sessions
Session listing
Unauthorized access
Access Token
Valid
Expired
Invalid signature
Wrong issuer
Wrong audience
Missing claim
Tampered token
Refresh Token
Valid refresh
Expired refresh
Revoked refresh
Rotation
Reuse detection
Concurrent refresh
Token family revocation
Security
Token theft
Session hijacking
Cross-tenant
Privilege escalation
Revoked session access
62. Acceptance Criteria

ESP-0001/06 estará preparado para implementación cuando:

Sessions
 Session model definido.
 Session lifecycle definido.
 Session revocation definido.
 Concurrent sessions definido.
 Session expiration definido.
Tokens
 Access Token definido.
 JWT validation definido.
 Refresh Token definido.
 Rotation definida.
 Revocation definida.
 Reuse detection definida.
Security
 Secure storage definido.
 Token leakage protection definido.
 TLS definido.
 Fail-closed definido.
 Audit definido.
Infrastructure
 PostgreSQL strategy.
 Redis strategy.
 Stateless validation strategy.
 Concurrency strategy.
63. ADR Candidates

Este documento deja los siguientes ADR:

ADR	Decisión
ADR-IDENTITY-034	Access/Refresh/Session lifetimes
ADR-IDENTITY-035	Concurrent session policy
ADR-IDENTITY-036	Session revocation on account suspension
ADR-IDENTITY-037	Refresh Token rotation
ADR-IDENTITY-038	Refresh Token reuse response
ADR-IDENTITY-039	Token storage strategy
ADR-IDENTITY-040	Session state: PostgreSQL vs Redis
ADR-IDENTITY-041	JWT claims
ADR-IDENTITY-042	JWT issuer/audience
ADR-IDENTITY-043	Stateless vs stateful validation
ADR-IDENTITY-044	Browser token storage
ADR-IDENTITY-045	Mobile token storage
ADR-IDENTITY-046	Device/session identification
64. Traceability
BP-0002
   │
   ▼
Identity
   │
   ▼
ESP-0001
   │
   ├── 01 Domain Model
   ├── 02 Architecture
   ├── 03 Authentication & Authorization
   ├── 04 User & Organization Management
   ├── 05 Roles & Permissions
   └── 06 Sessions & Token Management
           │
           ├── Session
           ├── Access Token
           ├── Refresh Token
           ├── Revocation
           ├── Rotation
           └── Security

La trazabilidad entre arquitectura, ESP, implementación, testing y deployment forma parte del lifecycle oficial de EVOXA.

65. Implementation Readiness

Actualmente:

ESP-0001 — Identity
│
├── 01 — Domain Model                    ✅
├── 02 — Architecture                   ✅
├── 03 — Authentication & Authorization  ✅
├── 04 — User & Organization Management  ✅
├── 05 — Roles & Permissions             ✅
└── 06 — Sessions & Token Management     🟡
Resultado conceptual
                         IDENTITY
                            │
          ┌─────────────────┼─────────────────┐
          ▼                 ▼                 ▼
         USER        ORGANIZATION          SESSION
          │                 │                 │
          └────────┬────────┘                 │
                   ▼                          │
              MEMBERSHIP                     │
                   │                          │
                   ▼                          │
                  ROLE                        │
                   │                          │
                   ▼                          │
              PERMISSION                      │
                   │                          │
                   ▼                          │
              RBAC + ABAC                     │
                   │                          │
                   └──────────┬───────────────┘
                              ▼
                       AUTHORIZATION
                              ▲
                              │
                         AUTHENTICATION
                              │
                              ▼
                           SESSION
                         ┌────┴────┐
                         ▼         ▼
                  ACCESS TOKEN  REFRESH TOKEN
                                    │
                                    ▼
                                 ROTATION
                                    │
                                    ▼
                                REVOCATION

Estado: 🟡 DRAFT — Architecture Defined.

Todavía no debemos programar estas piezas.

El siguiente documento lógico sería:

ESP-0001 / 07 — Identity Multi-Factor Authentication (MFA)

Ahí definiremos MFA enrollment, factores, challenges, verification, recovery, backup codes, trusted devices, step-up authentication y eventos de seguridad, manteniendo separada la autenticación multifactor del sistema general de sesiones y tokens.
