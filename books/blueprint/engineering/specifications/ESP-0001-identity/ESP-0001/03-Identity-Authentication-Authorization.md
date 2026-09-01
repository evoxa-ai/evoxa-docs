ESP-0001 / 03 — Identity Authentication & Authorization

Document ID: ESP-0001/03
Module: Identity
Version: 1.0.0
Status: Draft
Parent: ESP-0001 — Identity
Previous Specifications:

ESP-0001/01 — Identity Domain Model
ESP-0001/02 — Identity Architecture

Architecture Parent: BP-0002 — Reference Architecture
Repository: evoxa-platform
Target Path: apps/api/app/domains/identity/

1. Purpose

Este documento define cómo EVOXA debe realizar:

Authentication — determinar quién es el usuario.
Session establishment — establecer una sesión autenticada.
Token issuance — emitir credenciales de acceso.
Token refresh — renovar el acceso.
Token revocation — invalidar sesiones/credenciales.
MFA — aplicar autenticación multifactor cuando corresponda.
Authorization — determinar qué puede hacer el usuario.
Tenant authorization — determinar en qué organización puede actuar.
RBAC — autorización basada en roles.
ABAC — autorización basada en atributos y contexto.
Audit — registrar operaciones de seguridad relevantes.

La especificación se deriva de los principios de seguridad, API y capacidades definidos en el Blueprint de EVOXA.

La arquitectura establece OAuth2.1/OIDC, JWT, API keys y service accounts como mecanismos contemplados, además de RBAC + ABAC, MFA, tenant isolation y least privilege.

2. Fundamental Security Model

La diferencia fundamental será:

AUTHENTICATION
      │
      │ ¿Quién eres?
      ▼
   IDENTITY
      │
      ▼
AUTHORIZATION
      │
      │ ¿Qué puedes hacer?
      ▼
 ALLOW / DENY

Nunca debemos confundir:

Usuario autenticado

con:

Usuario autorizado

Un usuario puede estar correctamente autenticado y aun así no tener permiso para realizar una operación.

3. Authentication Flow

Flujo general:

┌─────────────┐
│    Client   │
└──────┬──────┘
       │
       │ Credentials
       ▼
┌─────────────────────┐
│    Identity API     │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│ Authentication      │
│ Service             │
└──────────┬──────────┘
           │
           ├── Validate identity
           ├── Validate credentials
           ├── Validate account
           ├── Check MFA
           │
           ▼
      ┌─────────┐
      │ Session │
      └────┬────┘
           │
           ├── Access Token
           └── Refresh Token
4. Authentication Mechanisms

EVOXA deberá soportar una arquitectura extensible de autenticación.

Conceptualmente:

                  Authentication
                        │
          ┌─────────────┼─────────────┐
          ▼             ▼             ▼
       Password        OIDC          SAML
          │             │             │
          ▼             ▼             ▼
      Local Auth    External IdP   Enterprise IdP
          │             │             │
          └─────────────┼─────────────┘
                        ▼
                 EVOXA Identity

La arquitectura de seguridad contempla EVOXA Identity, OIDC, SAML2 y proveedores como Entra ID, Google, Okta y Auth0.

Estado

La especificación no fija todavía cuál será el proveedor primario.

Eso deberá resolverse mediante ADR.

5. Credential Validation

Para autenticación local:

Email
+
Password
      │
      ▼
Normalize Email
      │
      ▼
Find User
      │
      ▼
Check Account Status
      │
      ▼
Verify Password
      │
      ▼
MFA?

La contraseña nunca debe almacenarse en texto plano.

La arquitectura establece Argon2id como mecanismo de password hashing.

6. Password Verification

Conceptualmente:

Submitted Password
        │
        ▼
   Argon2id Verify
        │
    ┌───┴───┐
    │       │
  Valid   Invalid
    │       │
    ▼       ▼
Continue    DENY

No se debe devolver al cliente información que permita distinguir innecesariamente entre:

email inexistente

y

password incorrecta

para reducir exposición de información de identidad.

7. Account Status Validation

Antes de establecer una sesión:

User
 │
 ▼
Account Status
 │
 ├── ACTIVE → continue
 │
 ├── PENDING → policy
 │
 ├── SUSPENDED → DENY
 │
 └── DISABLED → DENY

La definición exacta de los estados deberá permanecer alineada con el modelo de dominio de ESP-0001/01.

8. MFA

Cuando MFA sea requerido:

Credentials
    │
    ▼
Valid
    │
    ▼
MFA Required
    │
    ▼
Challenge
    │
    ▼
Verification
    │
 ┌──┴──┐
 ▼     ▼
Valid Invalid
 │       │
 ▼       ▼
Session  DENY

MFA es una capability explícita de Identity.

Los métodos concretos de MFA aún deben definirse.

Por tanto, esta especificación no debe asumir todavía TOTP, WebAuthn, SMS u otro método como mecanismo definitivo.

9. Authentication Result

Una autenticación exitosa produce un Authentication Context.

Conceptualmente:

AuthenticationContext
│
├── user_id
├── session_id
├── tenant context
├── authentication method
├── authentication time
├── MFA status
└── security claims

Este contexto será utilizado posteriormente por Authorization.

10. Session Creation

Después de una autenticación válida:

Authentication
      │
      ▼
Create Session
      │
      ├── user_id
      ├── organization context
      ├── created_at
      ├── expiration
      └── security metadata
      │
      ▼
Issue Credentials

La sesión debe poder ser posteriormente revocada.

11. Access Token

El Access Token representa la credencial temporal utilizada para acceder a APIs protegidas.

Flujo:

Client
  │
  │ Authorization: Bearer <token>
  ▼
API Gateway / API
  │
  ▼
Token Validation
  │
  ▼
Identity Context

La API Architecture establece JWT Bearer como mecanismo contemplado para autenticación de APIs.

12. JWT Claims

El conjunto exacto de claims deberá definirse en la especificación de API, pero conceptualmente deberá permitir identificar:

subject / user
session
issuer
issued_at
expiration
tenant context
security claims

No debemos introducir en el JWT información excesiva o sensible.

El principio debe ser:

El token contiene suficiente información para establecer el contexto de seguridad, pero no debe convertirse en una copia de la base de datos del usuario.

13. Refresh Token

El sistema utilizará el concepto:

Access Token
+
Refresh Token

Flujo:

Access Token expired
        │
        ▼
Refresh Token
        │
        ▼
Validate Session
        │
        ▼
Validate Refresh Credential
        │
        ▼
Issue New Access Token

La API Architecture contempla refresh tokens.

14. Refresh Token Rotation

La estrategia recomendada para EVOXA será utilizar rotación de refresh tokens, pero esta decisión debe formalizarse como ADR antes de convertirla en requisito definitivo.

Conceptualmente:

Refresh Token A
       │
       ▼
Refresh
       │
       ├── revoke A
       │
       └── issue B

Si se detecta reutilización de un token previamente revocado:

Refresh Token A
       │
       ▼
Reuse Detected
       │
       ▼
Security Event
       │
       ▼
Revoke Session / Token Family

Esto es Derived Design, no una regla literal del Blueprint.

15. Logout

Logout debe permitir revocar la sesión correspondiente.

Client
  │
  ▼
POST /auth/logout
  │
  ▼
Identity
  │
  ▼
Revoke Session
  │
  ▼
Audit Event

Después de la revocación:

Session = REVOKED

y las credenciales asociadas dejan de considerarse válidas según las políticas de sesión.

16. Global Logout

Como capacidad futura, Identity debe poder soportar:

Logout Current Session
Logout All Sessions
Revoke All Sessions

Ejemplo:

User
 │
 ├── Session A
 ├── Session B
 └── Session C

Logout All
 │
 ├── A → revoked
 ├── B → revoked
 └── C → revoked

La API exacta debe definirse posteriormente.

17. Authorization Flow

Una request autorizada deberá seguir:

Request
   │
   ▼
Authentication
   │
   ▼
Identity Context
   │
   ▼
Tenant Context
   │
   ▼
RBAC
   │
   ▼
ABAC / Policies
   │
   ▼
Ownership
   │
   ▼
ALLOW / DENY
18. Authorization Decision

La decisión debe ser explícita:

AuthorizationDecision
│
├── allowed
├── reason
├── policy
└── context

Ejemplo conceptual:

allowed = false

reason = "RESOURCE_NOT_OWNED"

Sin embargo, los motivos internos de seguridad no deben exponerse necesariamente al usuario final.

19. RBAC Authorization

RBAC:

User
 ↓
Membership
 ↓
Role
 ↓
Permission

Ejemplo:

Coach
 │
 ├── clients.read
 ├── clients.update
 ├── training.read
 └── training.create

La autorización se basa en permisos, no simplemente en nombres de roles.

Esto permite modificar los permisos de un rol sin tener que modificar el código de cada dominio.

20. Permission Model

La propuesta inicial continúa siendo:

resource.action

Ejemplos:

clients.read
clients.create
clients.update
clients.delete

training.read
training.create
training.update
training.delete

nutrition.read
nutrition.create

Status: Derived Design.

El formato definitivo deberá quedar establecido en el ADR correspondiente.

21. ABAC Authorization

ABAC permite incorporar contexto:

User
+
Role
+
Organization
+
Resource
+
Resource Owner
+
Resource State
+
Policy

Ejemplo:

User:
   Coach

Role:
   training.update

Organization:
   Organization A

Resource:
   Training Program 123

Owner:
   Coach belongs to Organization A

Result:
   ALLOW
22. Ownership

La pertenencia a una organización no implica automáticamente acceso a todos los recursos.

Ejemplo:

Coach A
   │
   └── Organization X

Client 1
   └── assigned to Coach A

Client 2
   └── assigned to Coach B

Coach A:

Client 1 → ALLOW
Client 2 → POLICY CHECK / DENY

La arquitectura contempla explícitamente ownership como parte de las restricciones de autorización.

23. Tenant Authorization

Toda autorización de recursos tenant-scoped debe verificar:

Authenticated User
       │
       ▼
Membership
       │
       ▼
Organization
       │
       ▼
Resource Tenant
       │
       ▼
Same Tenant?

Si:

User Tenant ≠ Resource Tenant

resultado:

DENY

La protección cross-tenant es una exigencia de Security Architecture.

24. Authorization Policy Engine

La arquitectura propuesta:

AuthorizationService
        │
        ├── RBAC
        │
        ├── ABAC
        │
        ├── TenantPolicy
        │
        ├── OwnershipPolicy
        │
        └── ResourcePolicy

Esto permitirá que dominios como Training o Nutrition aporten políticas específicas sin duplicar todo el sistema de Identity.

25. Domain-Owned Authorization

Hay una distinción importante:

Identity

Determina:

Who is the user?
What roles/permissions does the user have?
What organizations can the user access?
Business Domain

Determina:

Can this specific user perform this business operation
on this specific resource?

Ejemplo:

Identity
   ↓
training.update = ALLOWED
   ↓
Training Domain
   ↓
Does this program belong to the coach?
   ↓
ALLOW / DENY

Esto evita convertir Identity en el propietario de todas las reglas de negocio de EVOXA.

26. Authorization Middleware

En la API:

Request
  │
  ▼
Authentication Middleware
  │
  ▼
Authorization Middleware
  │
  ▼
Controller

Pero el middleware no debe contener todas las reglas.

Debe delegar:

AuthenticationMiddleware
       ↓
AuthenticationService

AuthorizationMiddleware
       ↓
AuthorizationService
27. Authentication Context Propagation

El contexto autenticado debe propagarse internamente:

API
 │
 ▼
Application Service
 │
 ▼
Domain Policy

Conceptualmente:

IdentityContext
├── user_id
├── session_id
├── tenant_id
├── roles
├── permissions
└── authentication metadata

No debe confiarse en un user_id proporcionado libremente por el frontend.

28. API Key Authentication

La arquitectura de API contempla API keys.

Estas estarán destinadas principalmente a:

External integrations
Server-to-server integrations
Developer integrations

y no deberían sustituir automáticamente el modelo de sesión de usuarios humanos.

Las API keys deberán disponer de:

Owner
Scope
Expiration
Status
Audit
Revocation

La implementación concreta será definida en una futura especificación de API Tokens.

29. Service Account Authentication

EVOXA deberá poder representar identidades no humanas.

Conceptualmente:

Human User
      │
      └── User Identity

Service
      │
      └── Service Account

Las service accounts deberán utilizar permisos mínimos.

Ejemplo:

analytics-service
    ↓
analytics.events.read

No:

analytics-service
    ↓
admin.*
30. Authentication Events

Eventos derivados:

UserAuthenticated
AuthenticationFailed
MFAChallengeCreated
MFACompleted
SessionCreated
SessionRevoked
TokenRefreshed
TokenRevoked
LogoutCompleted

Los nombres son una propuesta de diseño.

La arquitectura de eventos exige que los eventos representen hechos ocurridos, no comandos.

31. Security Events

Además de eventos de negocio:

AuthenticationFailed
AuthorizationDenied
MFAFailed
RefreshTokenReuseDetected
SuspiciousAuthentication

pueden alimentar:

Security Monitoring
Audit
Analytics
Alerting
32. Audit Flow
Authentication
      │
      ├──────────► Result
      │
      └──────────► Audit
                       │
                       ▼
                  Audit Store

Ejemplo conceptual:

{
  actor: "user-123",
  action: "LOGIN",
  result: "SUCCESS",
  timestamp: "...",
  correlation_id: "...",
  tenant_id: "org-123"
}

Las operaciones críticas deben ser auditables según la arquitectura de seguridad.

33. Failed Authentication

Un intento fallido debe:

1. Deny authentication
2. Avoid leaking identity information
3. Generate security telemetry
4. Generate audit record when applicable
5. Apply rate limiting / abuse controls

No debe responder:

"El usuario existe pero la contraseña es incorrecta"

si esa información no es necesaria.

34. Brute Force Protection

Especial atención a:

/login
/refresh
/mfa
/password-reset

La arquitectura de API contempla rate limiting como control de seguridad.

Los umbrales concretos no se fijan todavía en este documento.

35. Token Revocation

Debe ser posible revocar:

Session
Refresh Token
API Key
Service Credential

Ejemplo:

Security Incident
       │
       ▼
Revoke User Sessions
       │
       ▼
Invalidate Refresh Credentials
       │
       ▼
Audit
36. Password Change

Flujo:

Authenticated User
        │
        ▼
Verify Current Credential
        │
        ▼
Validate New Password
        │
        ▼
Hash with Argon2id
        │
        ▼
Persist
        │
        ▼
Security Event

Dependiendo de la política definitiva, un cambio de contraseña puede provocar revocación de sesiones existentes.

Esto debe quedar definido mediante una política de seguridad/ADR.

37. Password Reset

La recuperación de contraseña debe ser tratada como un flujo de autenticación de alta sensibilidad:

Request Reset
      │
      ▼
Identity Verification
      │
      ▼
Reset Challenge
      │
      ▼
New Password
      │
      ▼
Argon2id
      │
      ▼
Credential Updated

Debe evitarse revelar si una dirección de correo pertenece o no a EVOXA.

38. Secure Transport

Toda comunicación de autenticación/autorización debe utilizar transporte seguro.

La arquitectura de seguridad establece:

TLS 1.3

para transporte.

No se deben permitir credenciales mediante canales inseguros.

39. Sensitive Data

Identity maneja información especialmente sensible.

Por ello:

Passwords
Refresh Tokens
MFA Secrets
Authentication Secrets
Private Credentials

no deben aparecer en:

Application Logs
Error Responses
Analytics Events
Audit Payloads
Tracing Attributes

salvo que una especificación de seguridad explícita determine un tratamiento seguro.

40. AI Interaction

Identity también debe proteger las operaciones relacionadas con AI.

Por ejemplo:

User
 ↓
AI Request
 ↓
Identity
 ↓
Authorization
 ↓
AI Orchestrator

Identity debe determinar:

Who?
Which tenant?
Which permissions?

Mientras que AI determina:

What AI operation?
What model?
What context?
What recommendation?

La arquitectura AI exige aislamiento de tenant, context filtering y controles de seguridad.

41. Security Decision Flow

El modelo completo:

                     REQUEST
                        │
                        ▼
               ┌────────────────┐
               │ Authentication │
               └───────┬────────┘
                       │
                 authenticated?
                   ┌───┴───┐
                   │       │
                  NO      YES
                   │       │
                 DENY      ▼
                       Identity
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
                      Ownership
                           │
                           ▼
                       Resource
                         Policy
                           │
                      ┌────┴────┐
                      │         │
                    ALLOW      DENY
42. End-to-End Example

Supongamos:

Coach
Organization A
Client 123
Training Program 456

El coach solicita:

PATCH /api/v1/training-programs/456

Flujo:

1. Receive Request
       ↓
2. Validate Access Token
       ↓
3. Identify User
       ↓
4. Resolve Organization
       ↓
5. Check training.update
       ↓
6. Load Training Program
       ↓
7. Validate Tenant
       ↓
8. Validate Ownership
       ↓
9. Validate Business Policy
       ↓
10. Execute Update
       ↓
11. Publish Domain Event
       ↓
12. Audit

Este flujo mantiene separadas:

Identity
Security
Training Business Logic
Events
Audit
43. Performance Requirements

Authentication y authorization son capacidades transversales.

Por ello:

Authentication P95 < 500 ms
General API P95 < 300 ms

son los objetivos de arquitectura disponibles actualmente.

Las decisiones de caching no pueden comprometer seguridad.

44. Failure Behavior

Una regla fundamental:

Ante incertidumbre de autorización, Identity debe negar el acceso.

Ejemplo:

Authorization service unavailable
        │
        ▼
Cannot determine permission
        │
        ▼
DENY

No:

Cannot verify
    ↓
ALLOW

Esto es coherente con Zero Trust y Secure by Default.

45. Testing Requirements

Authentication debe probar:

Valid credentials
Invalid credentials
Inactive account
Suspended account
Expired credentials
MFA required
MFA success
MFA failure
Session creation
Session revocation
Refresh
Token expiration
Token reuse
Logout

Authorization debe probar:

Correct role
Missing permission
Tenant mismatch
Ownership mismatch
ABAC policy
RBAC policy
Cross-tenant access
Service account scopes
API key scopes

La estrategia de testing de EVOXA contempla específicamente autenticación, autorización, seguridad, API contracts y escenarios de AI.

46. Acceptance Criteria

ESP-0001/03 estará listo para implementación cuando:

Authentication
 El flujo de autenticación esté definido.
 Credential validation esté definido.
 Account status esté definido.
 MFA esté definido a nivel de arquitectura.
 Session establishment esté definido.
 Access token esté definido.
 Refresh token esté definido.
 Revocation esté definido.
Authorization
 RBAC esté definido.
 ABAC esté definido.
 Tenant isolation esté definido.
 Ownership esté definido.
 Resource policies estén definidas.
 Fail-closed esté definido.
Security
 Argon2id.
 TLS 1.3.
 Rate limiting.
 Audit.
 Security events.
 Sensitive-data protection.
Integration
 API Gateway integration.
 Event Platform integration.
 PostgreSQL persistence.
 Redis strategy.
 External Identity Provider abstraction.
47. Open ADRs

Después de ESP-0001/03 quedan especialmente importantes:

ADR	Decisión
ADR-IDENTITY-005	JWT como mecanismo principal
ADR-IDENTITY-006	Refresh Token y rotación
ADR-IDENTITY-007	Identity Provider principal
ADR-IDENTITY-008	MFA
ADR-IDENTITY-009	Persistence schema
ADR-IDENTITY-010	Redis
ADR-IDENTITY-011	RBAC + ABAC implementation
ADR-IDENTITY-012	Session lifecycle
ADR-IDENTITY-013	Password policy
ADR-IDENTITY-014	API Keys
ADR-IDENTITY-015	Service Accounts
48. Implementation Readiness

Con los tres documentos actuales:

ESP-0001
Identity
│
├── 01 — Identity Domain Model
│
├── 02 — Identity Architecture
│
└── 03 — Identity Authentication & Authorization

tenemos ahora:

                    IDENTITY
                       │
       ┌───────────────┼────────────────┐
       ▼               ▼                ▼
     MODEL        ARCHITECTURE    AUTHENTICATION
       │               │                │
       │               │                ▼
       │               │          AUTHORIZATION
       │               │                │
       └───────────────┼────────────────┘
                       ▼
                 API CONTRACTS
                       │
                       ▼
                  PERSISTENCE
                       │
                       ▼
                     EVENTS
                       │
                       ▼
                    TESTING
                       │
                       ▼
                    CODE
Estado actual

ESP-0001/03: 🟡 DRAFT — Architecture Defined

Todavía no recomiendo comenzar el código.

El siguiente documento lógico sería:

ESP-0001 / 04 — Identity User & Organization Management

Ahí definiríamos en detalle Users, Organizations, Memberships, lifecycle, creación, activación, suspensión, invitaciones, asignación organizacional y administración de identidades, antes de pasar a Roles & Permissions y posteriormente a los contratos API.

Y cuando terminemos el conjunto de ESP-0001, podremos hacer una revisión cruzada de todo Identity para detectar contradicciones antes de escribir una sola línea del backend.
