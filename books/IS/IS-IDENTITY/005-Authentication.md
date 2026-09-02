IS-IDENTITY-005 — Authentication

Implementation Story ID: IS-IDENTITY-005
Título: Authentication
Dominio: Identity
ESP: ESP-0001 — Identity
Prioridad: Critical
Estado: Draft
Versión: 1.0.0
Repositorio documental: evoxa-docs
Repositorio de implementación: evoxa-platform

1. Propósito

Esta Implementation Story define la capacidad de autenticar de forma segura a los usuarios de EVOXA y establecer un contexto de identidad confiable para las operaciones posteriores de autorización, sesiones, MFA y emisión de tokens.

Authentication debe responder fundamentalmente:

¿Quién es el actor que intenta acceder al sistema?

No debe decidir por sí sola:

¿Qué puede hacer ese actor?

Esa segunda responsabilidad pertenece a Authorization & Policy Engine, definida posteriormente en IS-IDENTITY-006.

La arquitectura de EVOXA separa explícitamente Authentication de Authorization y establece Identity como responsable de las capacidades de autenticación, sesiones, tokens y seguridad.

2. Business Goal

EVOXA necesita un mecanismo centralizado que permita:

Autenticar usuarios.
Validar credenciales.
Detectar cuentas inválidas.
Detectar cuentas suspendidas/deshabilitadas.
Integrar MFA.
Crear sesiones autenticadas.
Emitir tokens.
Registrar eventos de autenticación.
Aplicar políticas de seguridad.
Proteger contra ataques de fuerza bruta.
Mantener trazabilidad.
Proporcionar un Security Context confiable a los demás dominios.
3. Pregunta fundamental

Authentication establece:

Credentials
     ↓
Authentication
     ↓
Authenticated Identity

Mientras que posteriormente:

Authenticated Identity
     ↓
Authorization
     ↓
Allowed / Denied

Por tanto:

Authentication ≠ Authorization

Esta separación es una decisión arquitectónica fundamental de Identity.

4. Persona
Actores principales
Usuario final.
Coach.
Nutritionist.
Physiotherapist.
Team Manager.
Organization Administrator.
Platform Administrator.
Service Account.

Los permisos de cada actor serán evaluados por Authorization.

5. Journey

Flujo principal:

                    Login Request
                         │
                         ▼
                  Validate Request
                         │
                         ▼
                  Identify User
                         │
                         ▼
                 Validate Account
                         │
                         ▼
                Validate Credentials
                         │
                         ▼
                  Security Policy
                         │
              ┌──────────┴──────────┐
              │                     │
          MFA required          No MFA
              │                     │
              ▼                     │
        MFA Challenge               │
              │                     │
              ▼                     │
       Verify MFA                  │
              │                     │
              └──────────┬──────────┘
                         ▼
                  Create Session
                         │
                         ▼
                Issue Access Token
                         │
                         ▼
               Issue Refresh Token
                         │
                         ▼
                   Authenticated
6. Feature Pack

Identity → Authentication

Incluye:

Credential authentication.
Account validation.
Authentication policy evaluation.
MFA integration.
Session creation.
Access token issuance.
Refresh token issuance.
Logout initiation.
Authentication failure handling.
Security event generation.
Audit integration.
Rate limiting.
Authentication context creation.
7. Alcance
Incluido

Esta historia cubre:

Authentication domain/application flow.
Credential validation.
User lookup.
Account-state validation.
Organization/Membership context validation.
Security policy integration.
MFA integration boundary.
Session creation boundary.
Access token issuance boundary.
Refresh token issuance boundary.
Authentication events.
Failed authentication handling.
Rate limiting boundary.
Audit integration.
Security tests.
Authentication tests.
8. Fuera de alcance

No se implementa completamente aquí:

Authorization engine.
Role management.
Permission management.
Full session lifecycle.
Refresh token rotation.
MFA factor management.
Password reset workflow.
OAuth provider implementation.
External Identity Providers completos.
UI.
Mobile UI.
Kubernetes.

Estas capacidades se desarrollan mediante sus correspondientes IS/ADRs.

9. Authentication Model

El modelo conceptual será:

User
  ↓
Credentials
  ↓
Authentication Service
  ↓
Authentication Result
  ↓
Session
  ↓
Tokens

El resultado exitoso de Authentication no debe ser simplemente:

true

Debe producir un contexto suficientemente rico para crear la sesión y continuar el flujo.

10. Authentication Result

Resultado conceptual:

AuthenticationResult
├── user_id
├── authentication_status
├── authentication_method
├── session_context
├── mfa_status
├── security_context
└── token_context

No debe incluir secretos innecesarios.

11. Authentication Status

Estados conceptuales:

AUTHENTICATED
FAILED
MFA_REQUIRED
CHALLENGE_REQUIRED
BLOCKED
AUTHENTICATED

Las credenciales y políticas requeridas fueron satisfechas.

FAILED

Las credenciales no pudieron validarse.

MFA_REQUIRED

Las credenciales primarias son correctas pero se requiere segundo factor.

CHALLENGE_REQUIRED

Una política de seguridad exige un challenge adicional.

BLOCKED

La operación debe ser bloqueada por política, estado de cuenta o controles de seguridad.

12. User Account Validation

Antes de considerar autenticado a un usuario, el sistema debe validar el estado de su cuenta.

Ejemplo:

User = ACTIVE
       ↓
Continue

Mientras:

User = SUSPENDED
       ↓
Authentication denied

y:

User = DISABLED
       ↓
Authentication denied

Esto conecta directamente con IS-IDENTITY-002 — User & Account Management.

13. Organization Validation

La autenticación identifica al usuario globalmente.

La organización se determinará mediante el contexto correspondiente.

User
 ↓
Membership
 ↓
Organization

La existencia de un usuario autenticado no significa automáticamente que tenga acceso a cualquier organización.

14. Membership Validation

Cuando una autenticación se realiza en un contexto organizacional, deben considerarse:

User Status
+
Membership Status
+
Organization Status

Ejemplo:

User       = ACTIVE
Membership = ACTIVE
Organization = ACTIVE

                 ↓

             AUTHENTICATED

Pero:

User       = ACTIVE
Membership = SUSPENDED
Organization = ACTIVE

                 ↓

             BLOCKED

y:

User       = ACTIVE
Membership = ACTIVE
Organization = SUSPENDED

                 ↓

             BLOCKED

La autorización final continuará siendo responsabilidad de Authorization.

15. Credential Authentication

La autenticación primaria podrá utilizar credenciales de usuario.

La arquitectura de Identity establece Argon2id para protección de contraseñas.

Por tanto:

Submitted Password
       ↓
Password Verification
       ↓
Argon2id Hash Verification

El sistema nunca debe:

almacenar passwords en plaintext;
devolver passwords;
registrar passwords;
incluir passwords en eventos;
incluir hashes en respuestas API.
16. Password Verification

Conceptualmente:

User enters password
        ↓
Retrieve password verifier
        ↓
Argon2id verification
        ↓
 ┌──────┴──────┐
 │             │
Valid        Invalid
 │             │
 ▼             ▼
Continue      Failure

La implementación concreta del password hashing pertenece a Infrastructure/Security Services.

El Domain Layer no debe depender directamente de una librería concreta de hashing.

17. Timing & Enumeration Protection

Authentication debe evitar revelar información innecesaria.

No debería responder públicamente con diferencias como:

"Email does not exist"

versus:

"Password incorrect"

cuando hacerlo facilite enumeración de cuentas.

La respuesta pública debe utilizar un error de autenticación suficientemente genérico, mientras que el sistema mantiene información detallada internamente para seguridad y auditoría.

18. Authentication Policy

Antes de crear una sesión:

Credentials Valid
       ↓
Security Policy Engine
       ↓
Decision

El Policy Engine puede producir:

ALLOW
DENY
CHALLENGE
REAUTHENTICATE
REQUIRE_MFA
REVOKE

Estos valores son consistentes con la decisión definida en ADR-IDENTITY-005.

19. MFA Integration

Authentication debe estar preparado para MFA.

Flujo:

Primary Credentials
       ↓
Valid
       ↓
MFA Policy
       ↓
 ┌─────┴─────┐
 │           │
No MFA      MFA
 │           │
 ▼           ▼
Session    Challenge
              ↓
          Verification
              ↓
           Success
              ↓
           Session

La administración completa de factores MFA pertenece a:

IS-IDENTITY-010 — Multi-Factor Authentication.

Authentication debe consumir la capacidad, no duplicarla.

20. Session Creation

Una autenticación exitosa debe crear una sesión.

Authentication
      ↓
SessionService
      ↓
SessionCreated
      ↓
Token issuance

La sesión será persistente según ADR-IDENTITY-002 y ADR-IDENTITY-003.

Authentication no debe mantener la sesión dentro de memoria local del proceso.

21. Access Token

La autenticación exitosa permitirá emitir un JWT Access Token.

Características establecidas por ADR-IDENTITY-002:

JWT.
Short-lived.
Asymmetric signing.
Stateless validation.
jti.
sub.
iss.
aud.
iat.
exp.
session_id.

La selección definitiva entre Ed25519 y RSA-4096 permanece como decisión abierta de los ADR existentes.

22. Access Token Claims

Claims conceptuales:

sub
iss
aud
iat
exp
jti
session_id

No debe incluir:

password;
password hash;
refresh token;
MFA secret;
recovery codes;
health data;
información completa del Human Digital Twin;
catálogo completo de permisos;
información sensible innecesaria.
23. Refresh Token

Authentication puede emitir un Refresh Token asociado a la sesión.

El refresh token debe:

ser stateful;
persistirse;
almacenarse de forma segura;
utilizar rotación;
pertenecer a una token family;
permitir reuse detection.

La implementación detallada pertenece a:

IS-IDENTITY-009 — Refresh Token Lifecycle.

La arquitectura de refresh tokens ya definida en ADR-IDENTITY-003 establece rotación, token families y detección de reutilización.

24. Authentication + Session + Tokens

La relación completa:

                   Authentication
                         │
                         ▼
                      Session
                    /         \
                   /           \
                  ▼             ▼
           Access Token    Refresh Token
              JWT            Stateful
           short-lived      rotated

Esto evita tratar al JWT como si fuera la sesión.

JWT ≠ Session

25. Login Flow

Flujo normativo de alto nivel:

1. Receive credentials
2. Validate request
3. Identify user
4. Validate user state
5. Validate credentials
6. Evaluate security policies
7. Evaluate MFA requirement
8. Complete MFA if required
9. Resolve authentication context
10. Create session
11. Issue access token
12. Issue refresh token
13. Publish authentication event
14. Record audit
15. Return authentication response
26. Authentication Failure

Si las credenciales fallan:

Credentials
    ↓
Invalid
    ↓
AuthenticationFailed

El sistema debe:

rechazar la autenticación;
no crear una sesión;
no emitir tokens;
registrar evento de seguridad;
registrar auditoría cuando corresponda;
actualizar rate-limit/security counters;
evitar revelar información sensible.
27. Failed Authentication Events

Evento:

UserAuthenticationFailed

Payload conceptual:

{
    user_id?,
    authentication_method,
    reason_category,
    timestamp,
    security_context
}

Cuando el usuario no pueda identificarse de forma confiable, user_id puede no estar disponible.

Nunca debe incluir:

password
password_hash
tokens
MFA secret
28. Successful Authentication Event

Evento:

UserAuthenticated

Payload conceptual:

{
    user_id,
    session_id,
    authentication_method,
    mfa_used,
    timestamp
}
29. Session Event

Después de Authentication:

SessionCreated

debe ser producido por Session Management.

Authentication coordina el proceso, pero la propiedad del lifecycle de sesión permanece en Session Management.

30. Security Context

Authentication debe generar el contexto inicial:

SecurityContext
├── user_id
├── session_id
├── authentication_method
├── authenticated_at
├── mfa_state
├── organization_context
├── membership_context
└── security metadata

Posteriormente Authorization podrá enriquecerlo con:

roles
permissions
policies
resource context
risk context
31. Rate Limiting

Authentication debe estar protegido contra:

brute force;
credential stuffing;
password spraying;
automated login abuse.

El sistema debe poder aplicar límites sobre:

User
IP
Client
Device
Organization
Authentication endpoint

La estrategia concreta de Redis/rate limiting pertenece a Infrastructure y Runtime.

32. Account Locking / Blocking

La arquitectura permite políticas de seguridad que pueden producir:

BLOCKED

No se debe inventar aquí una regla fija como:

"Después de 5 intentos bloquear exactamente 30 minutos."

Esos valores no están definidos en el Blueprint disponible y deben ser configurables mediante Security Policy.

33. Logout

Authentication debe exponer el punto de entrada para logout, pero el lifecycle completo pertenece a Session Management.

Conceptualmente:

Logout
 ↓
SessionService
 ↓
Revoke Session
 ↓
Revoke Refresh Token Family
 ↓
Audit
 ↓
Security Event

El JWT access token puede permanecer técnicamente válido hasta su expiración debido a su naturaleza stateless, salvo que posteriormente se implemente una estrategia explícita de revocación inmediata.

Esto es consistente con ADR-IDENTITY-002/003.

34. Application Services

Casos de uso principales:

AuthenticateUser
CompleteAuthenticationChallenge
LogoutUser

La creación de sesión y emisión de tokens serán delegadas a los servicios correspondientes.

35. Domain Services

Se utilizarán límites conceptuales:

AuthenticationService
SessionService
SecurityPolicyService
AuditService

y posteriormente:

MFAService
TokenService

La responsabilidad debe mantenerse separada.

36. Repository Dependencies

Authentication requerirá acceso abstracto a:

UserRepository
MembershipRepository
OrganizationRepository
SessionRepository

y posteriormente:

RefreshTokenRepository
MFA repositories

No deberá acceder directamente a tablas PostgreSQL.

37. API Requirements

Namespace:

/api/v1/identity

Endpoint principal:

POST /auth/login

Conceptualmente:

Request
{
  "email": "user@example.com",
  "password": "********"
}
Successful response conceptual
{
  "user": {
    "id": "..."
  },
  "session": {
    "id": "..."
  },
  "access_token": "...",
  "refresh_token": "...",
  "token_type": "Bearer"
}

La forma final del contrato será definida en:

IS-IDENTITY-014 — Identity API Contracts.

No se considera este ejemplo un contrato final de OpenAPI.

38. MFA Challenge Response

Cuando MFA sea requerido, la respuesta deberá indicar que la autenticación primaria fue insuficiente para completar el proceso:

MFA_REQUIRED

sin emitir una sesión plenamente autenticada ni tokens finales de acceso antes de completar el challenge.

La forma exacta de la respuesta será definida en la IS de MFA y contratos API.

39. Standard Error Model

Authentication debe utilizar el envelope EVOXA:

{
  "error": {
    "code": "...",
    "message": "...",
    "correlation_id": "..."
  }
}

Errores conceptuales:

InvalidCredentials
AuthenticationFailed
AccountSuspended
AccountDisabled
MembershipSuspended
OrganizationSuspended
MFARequired
AuthenticationChallengeRequired
RateLimitExceeded
Unauthorized
SecurityPolicyViolation
40. Security Requirements

Authentication debe cumplir:

Argon2id.
Fail closed.
Least privilege.
No plaintext passwords.
No passwords en logs.
No tokens en logs.
No secrets en eventos.
Rate limiting.
Brute-force protection.
Account state validation.
Membership validation cuando aplique.
Organization state validation.
MFA policy integration.
Security Policy Engine.
Auditability.
Correlation IDs.
Secure token issuance.
Secure session creation.
No account enumeration innecesaria.
41. Authentication Events

Eventos principales:

UserAuthenticated
UserAuthenticationFailed

Eventos relacionados generados por otros componentes:

SessionCreated
RefreshTokenRotated
MFAChallengeVerified
SecurityPolicyViolation
42. Audit

Las siguientes acciones deben ser auditables:

Login success
Login failure
MFA required
MFA success/failure
Logout
Security policy block
Authentication challenge

La auditoría nunca debe almacenar:

password
password_hash
access_token
refresh_token
MFA secret
recovery code
private key
43. Observability

Authentication debe producir métricas como:

authentication_requests_total
authentication_success_total
authentication_failure_total
authentication_latency
mfa_required_total
mfa_failure_total
rate_limit_events_total
security_policy_denials_total

Y trazas correlacionadas mediante:

correlation_id
trace_id
44. Performance Requirements

Debe respetar los objetivos establecidos para Identity:

Authentication P95 < 500ms
API P95 < 300ms
Availability ≥ 99.9%

Las operaciones lentas no deben bloquear innecesariamente el request.

El sistema debe utilizar:

connection pooling;
Redis cuando corresponda;
caching seguro;
timeouts;
bounded retries.
45. External Identity Providers

La arquitectura contempla integración futura con:

OAuth/OIDC.
External Identity Providers.
Enterprise Identity.

Pero esta IS no implementa proveedores externos.

Debe existir una abstracción que permita posteriormente:

AuthenticationProvider
├── LocalCredentials
├── OIDC
├── OAuth
└── EnterpriseIdentity

La implementación concreta se definirá mediante futuras especificaciones/ADRs.

46. Service Accounts

Authentication debe distinguir entre:

Human User

y:

Service Account

Los mecanismos de autenticación machine-to-machine no deben reutilizar automáticamente el login humano.

Los detalles corresponden a la arquitectura de API/service accounts.

47. Acceptance Criteria Consolidated
ID	Criterio
AC-001	Se pueden validar credenciales válidas
AC-002	Credenciales inválidas son rechazadas
AC-003	No se crean sesiones en autenticaciones fallidas
AC-004	No se emiten tokens ante autenticación fallida
AC-005	Usuarios SUSPENDED son bloqueados
AC-006	Usuarios DISABLED son bloqueados
AC-007	Membership inválido bloquea el contexto organizacional
AC-008	Organization SUSPENDED bloquea acceso organizacional
AC-009	Se integra Security Policy Engine
AC-010	Se integra MFA cuando sea requerido
AC-011	Se crea una sesión tras autenticación completa
AC-012	Se emite Access Token correctamente
AC-013	Se emite Refresh Token mediante el mecanismo definido
AC-014	Se generan eventos de autenticación
AC-015	Las operaciones relevantes son auditables
AC-016	Passwords y secretos nunca se registran
AC-017	Existe protección contra brute force
AC-018	Se evita información innecesaria de enumeración
AC-019	Authentication permanece separada de Authorization
AC-020	El flujo soporta correlation/trace IDs
AC-021	Las operaciones críticas fallan cerradas
AC-022	Se mantiene la separación de capas arquitectónicas
48. Technical Tasks
TASK-IDENTITY-005-01 — Definir Authentication Service

Crear el contrato conceptual:

AuthenticationService
TASK-IDENTITY-005-02 — Definir AuthenticationResult

Crear el modelo de resultado de autenticación.

TASK-IDENTITY-005-03 — Implementar Credential Validation

Implementar la validación de credenciales mediante la abstracción correspondiente.

TASK-IDENTITY-005-04 — Integrar Argon2id

Implementar el password verification adapter sin acoplar Domain directamente a la librería.

TASK-IDENTITY-005-05 — Implementar Account State Validation

Validar:

PENDING
ACTIVE
SUSPENDED
DISABLED
TASK-IDENTITY-005-06 — Implementar Membership Context Validation

Validar Membership cuando la operación requiera contexto organizacional.

TASK-IDENTITY-005-07 — Implementar Organization State Validation

Validar estado de la Organization.

TASK-IDENTITY-005-08 — Integrar SecurityPolicyService

Evaluar políticas antes de completar la autenticación.

TASK-IDENTITY-005-09 — Definir MFA Integration Boundary

Preparar el punto de integración con MFAService.

TASK-IDENTITY-005-10 — Integrar SessionService

Crear sesión únicamente después de autenticación completa.

TASK-IDENTITY-005-11 — Integrar Access Token Service

Preparar emisión de JWT Access Token.

TASK-IDENTITY-005-12 — Integrar Refresh Token Service

Delegar emisión/gestión de Refresh Token al componente correspondiente.

TASK-IDENTITY-005-13 — Implementar AuthenticateUser

Crear el caso de uso principal:

AuthenticateUser
TASK-IDENTITY-005-14 — Implementar Authentication Failure Handling

Gestionar:

invalid credentials;
blocked accounts;
policy denial;
rate limit;
MFA failure.
TASK-IDENTITY-005-15 — Implementar Rate Limiting Boundary

Definir el contrato para:

RateLimitService
TASK-IDENTITY-005-16 — Implementar Authentication Events

Crear:

UserAuthenticated
UserAuthenticationFailed
TASK-IDENTITY-005-17 — Integrar AuditService

Registrar:

login;
failure;
MFA challenge;
policy denial;
logout.
TASK-IDENTITY-005-18 — Implementar Logout Boundary

Delegar revocación de sesión y token family a los servicios correspondientes.

TASK-IDENTITY-005-19 — Definir Security Context

Crear el contexto inicial de identidad autenticada.

TASK-IDENTITY-005-20 — Implementar API Contract

Preparar endpoint:

POST /auth/login

La definición final pertenece a IS-IDENTITY-014.

TASK-IDENTITY-005-21 — Unit Tests

Probar:

credential verification;
account state;
authentication result;
errors;
policy decisions.
TASK-IDENTITY-005-22 — Integration Tests

Probar:

User
+
Membership
+
Organization
+
Authentication
+
Session
TASK-IDENTITY-005-23 — Security Tests

Cubrir:

brute force;
credential stuffing;
enumeration;
suspended account;
disabled account;
suspended membership;
suspended organization;
privilege escalation;
secret leakage.
TASK-IDENTITY-005-24 — Token Tests

Validar:

JWT claims;
expiry;
signature;
session association;
token issuance conditions.
TASK-IDENTITY-005-25 — MFA Tests

Validar:

Credentials valid
       ↓
MFA required
       ↓
No token yet
       ↓
MFA valid
       ↓
Token issuance
TASK-IDENTITY-005-26 — Concurrency Tests

Probar múltiples solicitudes simultáneas de login y creación de sesión.

TASK-IDENTITY-005-27 — Performance Tests

Validar:

Authentication P95 < 500ms

bajo carga representativa.

TASK-IDENTITY-005-28 — Observability Tests

Verificar:

metrics;
traces;
correlation IDs;
security events;
audit events.
TASK-IDENTITY-005-29 — Failure Mode Tests

Probar:

PostgreSQL unavailable
Redis unavailable
KMS unavailable
Session service unavailable
MFA service unavailable

y verificar comportamiento fail-closed/controlado según la dependencia.

TASK-IDENTITY-005-30 — Traceability

Actualizar:

ESP-0001
    ↓
IS-IDENTITY-005
    ↓
TASK-IDENTITY-005-01 ... 030
    ↓
Source Code
    ↓
Tests
49. Definition of Done
 Authentication Service definido.
 AuthenticationResult implementado.
 Credential validation implementada.
 Argon2id integrado.
 Account state validation implementada.
 Membership validation integrada.
 Organization validation integrada.
 Security Policy Engine integrado.
 MFA boundary definido.
 SessionService integrado.
 Access Token Service integrado.
 Refresh Token Service integrado.
 AuthenticateUser implementado.
 Failure handling implementado.
 Rate limiting integrado.
 Authentication events implementados.
 Audit integrado.
 Logout boundary implementado.
 Security Context definido.
 API contract preparado.
 Unit tests implementados.
 Integration tests implementados.
 Security tests implementados.
 Token tests implementados.
 MFA tests implementados.
 Concurrency tests implementados.
 Performance tests implementados.
 Observability tests implementados.
 Failure mode tests implementados.
 Coverage ≥85% general.
 Coverage ≥95% en áreas críticas de Identity.
 Traceability actualizada.
 Architecture Review aprobado.
 QA Review aprobado.
 Security Review aprobado.
50. Traceability Matrix
Elemento	Referencia
Blueprint	Architecture Map
Domain	Identity
ESP	ESP-0001
Predecessor	IS-IDENTITY-001
Predecessor	IS-IDENTITY-002
Predecessor	IS-IDENTITY-003
Predecessor	IS-IDENTITY-004
Current IS	IS-IDENTITY-005
ADR	ADR-IDENTITY-001
ADR	ADR-IDENTITY-002
ADR	ADR-IDENTITY-003
ADR	ADR-IDENTITY-005
ADR	ADR-IDENTITY-006
ADR	ADR-IDENTITY-007
ADR	ADR-IDENTITY-008
ADR	ADR-IDENTITY-010
ADR	ADR-IDENTITY-011
Source	apps/api/app/domains/identity/
Tests	apps/api/tests/identity/
51. Dependencies
Depends on
IS-IDENTITY-001 — Identity Domain Foundation
IS-IDENTITY-002 — User & Account Management
IS-IDENTITY-003 — Organization & Tenant Management
IS-IDENTITY-004 — Membership Management

ADR-IDENTITY-001 — Identity Architecture
ADR-IDENTITY-002 — JWT & Token Lifecycle
ADR-IDENTITY-003 — Refresh Token Security & Session Management
ADR-IDENTITY-005 — Authorization & Policy Engine
ADR-IDENTITY-006 — Identity Persistence
ADR-IDENTITY-007 — Identity Event & Outbox
ADR-IDENTITY-008 — Cryptographic Key Management
ADR-IDENTITY-010 — Identity Testing
ADR-IDENTITY-011 — Deployment & Runtime
Required by
IS-IDENTITY-006 — Authorization & Permission Evaluation
IS-IDENTITY-008 — Session Management
IS-IDENTITY-009 — Refresh Token Lifecycle
IS-IDENTITY-010 — Multi-Factor Authentication
IS-IDENTITY-011 — Security Policy Engine
IS-IDENTITY-012 — Audit & Security Events
IS-IDENTITY-014 — Identity API Contracts
52. Architectural Model

Con esta IS, el modelo de Identity queda:

                         USER
                          │
                          │
                     Membership
                          │
                          ▼
                    Organization
                          │
                          │
                  Authentication
                          │
             ┌────────────┴────────────┐
             │                         │
       Credentials                  MFA
             │                         │
             └────────────┬────────────┘
                          ▼
                       Session
                      /       \
                     ▼         ▼
              Access Token   Refresh Token
                  JWT          Stateful
                     │
                     ▼
                Authorization
                     │
                     ▼
              Policy Evaluation
                     │
                     ▼
               Business Domain

La idea fundamental es:

Authentication
    = demostrar quién eres

Membership
    = demostrar en qué tenant tienes relación

Authorization
    = determinar qué puedes hacer

Policy
    = determinar bajo qué condiciones puedes hacerlo

Session
    = mantener el estado de autenticación

Access Token
    = representar autenticación de corta duración

Refresh Token
    = renovar la autenticación de forma controlada
53. Estado de la Implementation Story
IS-IDENTITY-005 — Authentication

Estado: Draft

Con esto ya tenemos las primeras cinco piezas de Identity:

IS-IDENTITY-001
Identity Domain Foundation
        ↓
IS-IDENTITY-002
User & Account Management
        ↓
IS-IDENTITY-003
Organization & Tenant Management
        ↓
IS-IDENTITY-004
Membership Management
        ↓
IS-IDENTITY-005
Authentication

Y la siguiente pieza lógica es:

IS-IDENTITY-006 — Authorization & Permission Evaluation

Ahí definiremos formalmente cómo EVOXA pasa de:

"Ya sé quién eres"

a:

"Ahora determino qué puedes hacer,
en qué organización,
sobre qué recurso,
bajo qué condiciones
y según qué política."

Esto será especialmente importante porque el Blueprint exige separación de Authentication, Authorization, RBAC, ABAC y Tenant Isolation, evitando que las reglas de permisos queden mezcladas dentro de Authentication.
