E04 — EVOXA Authentication Architecture
Architecture & Engineering Specification

Pertenece a: Engineering Specification

Depende de:

A01 — EVOXA Master Architecture
A02 — EVOXA System Architecture
A05 — Security Architecture
A06 — EVOXA API Architecture
A13 — EVOXA Multi-Tenant Architecture
E01 — EVOXA Backend Architecture
E02 — EVOXA Database Architecture
E03 — EVOXA API Architecture

Siguiente: E05 — EVOXA Authorization Architecture

1. Propósito

E04 define la arquitectura completa de Authentication de EVOXA.

Authentication responde:

¿Quién eres?

Authorization, que será definida en E05, responde:

¿Qué puedes hacer?

La arquitectura de autenticación debe permitir que usuarios, aplicaciones, dispositivos, integraciones y agentes puedan identificarse de forma segura ante EVOXA.

2. Principio Fundamental

La autenticación debe seguir:

Identity
   ↓
Authentication
   ↓
Session / Token
   ↓
Tenant Context
   ↓
Authorization
   ↓
Application

Nunca:

Client
   ↓
Resource

sin pasar por el mecanismo de identidad correspondiente.

3. Objetivos

E04 debe proporcionar:

Secure Authentication
Identity Verification
Session Management
Token Management
Credential Management
MFA
Account Recovery
Email Verification
OAuth / OIDC
Device Management
Session Revocation
Auditability
Tenant Awareness
4. Authentication Architecture

Arquitectura general:

                    EVOXA AUTHENTICATION
                            │
        ┌───────────────────┼───────────────────┐
        │                   │                   │
    Credentials          OAuth/OIDC           MFA
        │                   │                   │
        └───────────────────┼───────────────────┘
                            │
                       Identity
                            │
                       Auth Service
                            │
              ┌─────────────┼─────────────┐
              │             │             │
          Sessions       Tokens        Devices
              │             │             │
              └─────────────┼─────────────┘
                            │
                     Tenant Context
                            │
                     Authorization
5. Identity vs Authentication

EVOXA debe separar:

Identity

Representa:

User
Identity Provider
External Identity
Credentials
Email
Phone
Authentication

Representa:

Login
Verification
Tokens
Sessions
MFA
Recovery
Logout
6. User Identity

Un usuario de EVOXA posee una identidad interna:

User
 ├── id
 ├── email
 ├── status
 ├── identity metadata
 ├── createdAt
 └── updatedAt

La identidad interna no debe depender directamente del proveedor OAuth.

7. Authentication Factors

EVOXA podrá soportar múltiples factores:

Something You Know
 └── Password

Something You Have
 ├── Authenticator App
 ├── Passkey
 └── Trusted Device

Something You Are
 └── Future biometric integrations

La arquitectura debe permitir agregar factores sin rediseñar todo Authentication.

8. Primary Authentication

El mecanismo inicial será:

Email
+
Password

Flujo:

Email
 ↓
Password Verification
 ↓
Identity
 ↓
Account Status
 ↓
MFA Check
 ↓
Session
 ↓
Tokens
9. Registration

Endpoint:

POST /api/v1/auth/register

Request:

{
  "email": "user@example.com",
  "password": "********",
  "firstName": "User",
  "lastName": "Example"
}

Proceso:

Validate
 ↓
Check Existing Identity
 ↓
Hash Password
 ↓
Create User
 ↓
Create Identity
 ↓
Create Verification Token
 ↓
Send Verification Email
10. Password Storage

Nunca almacenar:

plain password

Debe almacenarse:

password_hash

utilizando un algoritmo moderno de password hashing.

Preferencia arquitectónica:

Argon2id

con parámetros configurables.

11. Password Policy

La política debe establecer:

Minimum Length
Password Strength
Common Password Detection
Breached Password Detection
Reuse Policy
Reset Policy

La longitud mínima debe ser configurable por política de seguridad.

12. Password Hashing

Flujo:

Password
   ↓
Argon2id
   ↓
Salt
   ↓
Password Hash
   ↓
Database

Nunca:

Password
 ↓
Encryption
 ↓
Database

Las contraseñas no necesitan ser recuperables.

13. Login

Endpoint:

POST /api/v1/auth/login

Request:

{
  "email": "user@example.com",
  "password": "********"
}

Flujo:

Request
 ↓
Rate Limit
 ↓
Find Identity
 ↓
Verify Password
 ↓
Check Account
 ↓
Check MFA
 ↓
Create Session
 ↓
Issue Tokens
14. Login Success

Response conceptual:

{
  "data": {
    "accessToken": "...",
    "refreshToken": "...",
    "expiresIn": 900,
    "tokenType": "Bearer"
  }
}

La implementación puede evolucionar hacia un modelo donde el refresh token se gestione mediante cookie segura para determinados clientes.

15. Access Token

El Access Token permite acceder a recursos protegidos.

Características:

Short-lived
Scoped
Signed
Tenant-aware
Revocable indirectly through session controls

Ejemplo conceptual:

15 minutes

El valor definitivo será configurable.

16. Refresh Token

El Refresh Token permite obtener un nuevo Access Token sin volver a solicitar credenciales.

Características:

Long-lived
Rotating
Revocable
Bound to session
Stored securely
17. Token Architecture
Login
 ↓
Session
 ↓
Access Token
 +
Refresh Token

Posteriormente:

Refresh Token
 ↓
Validate Session
 ↓
Rotate Refresh Token
 ↓
Issue New Access Token
 ↓
Issue New Refresh Token
18. Refresh Token Rotation

EVOXA utilizará refresh token rotation.

RT1
 ↓
Refresh
 ↓
RT2

El token anterior:

RT1

queda invalidado.

19. Refresh Token Reuse Detection

Si un refresh token ya utilizado vuelve a presentarse:

RT1
 ↓
Reuse Detected
 ↓
Security Event
 ↓
Invalidate Token Family
 ↓
Invalidate Session

Esto reduce el impacto de robo de refresh tokens.

20. Token Family

Los refresh tokens pertenecen a una familia:

Family A
 ├── RT1
 ├── RT2
 ├── RT3
 └── RT4

Una detección de reutilización puede invalidar la familia completa.

21. JWT

Los Access Tokens pueden utilizar JWT.

Conceptualmente:

{
  "sub": "user-id",
  "sid": "session-id",
  "tid": "tenant-id",
  "iat": 123,
  "exp": 456,
  "scope": ["profile:read"]
}

No incluir información sensible innecesaria.

22. JWT Claims

Claims principales:

sub
sid
tid
iat
exp
jti
scope
iss
aud

Donde:

sub → user
sid → session
tid → tenant
jti → token identifier
23. JWT Issuer

El token debe identificar su emisor:

iss = EVOXA Authentication Service

Los consumidores deben validar:

issuer
audience
signature
expiration
24. JWT Audience

Los tokens deben poder limitarse mediante:

aud

Ejemplo:

evoxa-api

Un token destinado a una audiencia no válida debe rechazarse.

25. Token Signing

Las claves criptográficas deben estar separadas de la aplicación.

Preferencia:

Key Management System
+
Key Rotation

Nunca almacenar claves privadas directamente en el código.

26. Signing Key Rotation

Arquitectura:

Key A
 ↓
Key B
 ↓
Key C

Durante la transición pueden existir múltiples claves activas para validación.

Los JWT deben incluir un identificador de clave:

kid
27. Session Architecture

Una sesión representa:

Una instancia autenticada de un usuario en un dispositivo o cliente.

Modelo conceptual:

Session
 ├── id
 ├── userId
 ├── tenantId
 ├── deviceId
 ├── createdAt
 ├── lastActivityAt
 ├── expiresAt
 ├── revokedAt
 └── metadata
28. Session Lifecycle
CREATED
   ↓
ACTIVE
   ↓
EXPIRED

o:

ACTIVE
   ↓
REVOKED
29. Session States

Estados posibles:

PENDING
ACTIVE
EXPIRED
REVOKED
LOCKED

No todos deben necesariamente persistirse como estados independientes si pueden derivarse.

30. Session Revocation

Un usuario puede revocar:

Current Session
Specific Session
All Sessions
All Other Sessions

Ejemplo:

POST /api/v1/auth/logout
31. Logout

Logout debe invalidar la sesión.

Client
 ↓
Logout
 ↓
Revoke Session
 ↓
Invalidate Refresh Token Family

El Access Token de corta duración puede continuar siendo criptográficamente válido hasta expirar, por lo que los recursos extremadamente sensibles pueden requerir controles adicionales.

32. Logout All

Endpoint:

POST /api/v1/auth/logout-all

Flujo:

User
 ↓
Revoke All Sessions
 ↓
Revoke Token Families
33. Device Management

EVOXA debe reconocer dispositivos/sesiones.

Ejemplo:

Devices
 ├── Chrome / Windows
 ├── iPhone
 └── Android

El usuario podrá revisar:

Device
IP
Location approximation
Last activity
Created date
Session status

No almacenar más información de la necesaria.

34. Device Fingerprinting

No depender exclusivamente de fingerprinting.

Puede utilizarse como señal adicional:

Device metadata
+
Session
+
Risk Signals

No como identidad criptográfica única.

35. Trusted Devices

EVOXA podrá soportar:

Trusted Device

para reducir desafíos MFA.

El mecanismo deberá utilizar credenciales/tokenización segura y expiración.

36. Email Verification

Después del registro:

REGISTER
 ↓
Verification Token
 ↓
Email
 ↓
User Clicks Link
 ↓
Verify Identity
 ↓
Email Verified

Endpoint:

POST /api/v1/auth/email/verify

o equivalente según implementación.

37. Verification Token

Debe ser:

Random
High Entropy
Single-use
Time-limited
Stored Hashed

Nunca guardar tokens de verificación en texto plano si no es necesario.

38. Resend Verification
POST /api/v1/auth/email/resend

Debe existir rate limiting.

No permitir abuso para convertirlo en un mecanismo de spam.

39. Password Reset

Endpoint:

POST /api/v1/auth/password/forgot

Flujo:

Email
 ↓
Generate Reset Token
 ↓
Send Email
 ↓
User Opens Link
 ↓
Set New Password
 ↓
Invalidate Existing Sessions
40. Password Reset Security

La respuesta para una cuenta existente o inexistente debe ser equivalente.

Ejemplo:

"If the account exists, instructions have been sent."

Esto evita:

User Enumeration
41. Reset Token

Características:

Single-use
Short-lived
Random
High entropy
Hashed at rest
42. Password Change

Usuario autenticado:

POST /api/v1/auth/password/change

Puede requerir:

Current Password
+
New Password

Después del cambio:

Security Event

y opcionalmente:

Revoke Other Sessions
43. Account Locking

La cuenta puede bloquearse temporalmente después de múltiples intentos sospechosos.

Failed Attempts
 ↓
Risk Evaluation
 ↓
Temporary Lock

Debe evitarse un mecanismo que permita fácilmente bloquear cuentas de terceros mediante solicitudes maliciosas.

44. Brute Force Protection

Controles:

IP Rate Limit
Account Rate Limit
Device Signals
Progressive Delay
CAPTCHA / Challenge
Temporary Lock

según nivel de riesgo.

45. Credential Stuffing

EVOXA debe contemplar ataques con credenciales filtradas.

Controles:

Rate Limiting
Password Breach Detection
MFA
Risk Detection
Session Monitoring
46. MFA

EVOXA debe soportar MFA como segundo factor.

Arquitectura:

Password
 ↓
MFA Challenge
 ↓
Second Factor
 ↓
Authenticated Session
47. TOTP

Una opción inicial:

TOTP

compatible con aplicaciones autenticadoras.

Proceso:

Enroll
 ↓
Generate Secret
 ↓
QR
 ↓
User Confirms Code
 ↓
MFA Enabled
48. MFA Secret

El secreto TOTP debe protegerse cuidadosamente.

Nunca:

log(secret)

Debe almacenarse cifrado o protegido mediante mecanismos equivalentes de gestión de secretos.

49. MFA Recovery Codes

Al activar MFA:

Recovery Codes

deben generarse.

Características:

One-time use
High entropy
Hashed
50. Recovery Code Usage
Login
 ↓
Password
 ↓
MFA Challenge
 ↓
Recovery Code
 ↓
Authenticated

Después del uso:

Recovery Code
 ↓
Consumed
51. MFA Disable

Deshabilitar MFA debe requerir autenticación reforzada.

Puede requerir:

Current Password
+
MFA

o:

Recovery Process

según el caso.

52. Step-Up Authentication

Para operaciones sensibles:

Authenticated
       ↓
Sensitive Operation
       ↓
Step-Up Authentication
       ↓
Operation

Ejemplos:

Change email
Disable MFA
Delete account
Rotate API credentials
Billing changes
Security changes
53. Passkeys

La arquitectura debe permitir incorporar:

WebAuthn
FIDO2
Passkeys

como mecanismo de autenticación moderno.

No debe requerir rediseñar User/Session.

54. OAuth 2.0 / OIDC

EVOXA podrá integrarse con proveedores externos:

Google
Microsoft
Apple
Enterprise Identity Providers

mediante:

OAuth 2.0
OpenID Connect
55. External Identity

Una identidad externa debe mapearse:

Provider
+
Provider Subject
+
EVOXA User

Ejemplo:

google
+
123456789
+
user_abc

Nunca utilizar únicamente el email como identificador externo.

56. OAuth Login
EVOXA
 ↓
Identity Provider
 ↓
Authorization
 ↓
Callback
 ↓
Validate ID Token
 ↓
Map External Identity
 ↓
Create / Login User
 ↓
Create Session
57. OIDC Validation

Validar:

issuer
audience
signature
nonce
state
expiration
subject
58. OAuth Account Linking

Un usuario puede vincular:

EVOXA Password
+
Google
+
Microsoft

pero el linking debe requerir autenticación suficiente.

Nunca vincular automáticamente una identidad externa solamente porque coincide el email sin un flujo seguro de verificación.

59. Enterprise SSO

Para clientes Enterprise:

Tenant
 ↓
Identity Provider
 ↓
SAML / OIDC
 ↓
EVOXA

El tenant podrá definir políticas de autenticación.

60. Authentication Policy

La configuración puede existir por tenant:

Tenant Authentication Policy
 ├── Password Required
 ├── MFA Required
 ├── SSO Required
 ├── Session Lifetime
 ├── Password Policy
 └── Allowed Identity Providers
61. Tenant Authentication

Un usuario puede pertenecer a múltiples tenants:

User
 ├── Tenant A
 ├── Tenant B
 └── Tenant C

Authentication identifica al usuario.

Tenant context determina:

current tenant
62. Tenant Switching

Si un usuario pertenece a múltiples tenants:

User Authenticated
 ↓
Tenant Memberships
 ↓
Select Tenant
 ↓
Tenant Context
 ↓
Access Token / Session Context

La selección de tenant debe validarse contra memberships reales.

63. Tenant Isolation

Nunca aceptar simplemente:

tenantId = request.body.tenantId

sin validar que el usuario pertenece al tenant.

64. Authentication Context

El contexto autenticado puede contener:

userId
sessionId
tenantId
authenticationMethod
mfaLevel
deviceId
scopes
65. Authentication Assurance Level

EVOXA puede distinguir:

AAL1
Basic Authentication

AAL2
Password + MFA

AAL3
Hardware-backed / Strong Auth

Esto permite exigir mayor seguridad para operaciones críticas.

66. Authentication Middleware

Pipeline:

HTTP Request
 ↓
Extract Credentials
 ↓
Validate Token
 ↓
Resolve Session
 ↓
Resolve User
 ↓
Resolve Tenant
 ↓
Build AuthContext
 ↓
Application
67. Auth Context

Conceptualmente:

AuthContext {
  userId: string;
  sessionId: string;
  tenantId?: string;
  authenticationMethod: string;
  assuranceLevel: string;
  scopes: string[];
}
68. Authentication Service

Responsabilidades:

register()
login()
refresh()
logout()
logoutAll()
verifyEmail()
forgotPassword()
resetPassword()
changePassword()
enableMfa()
disableMfa()
createSession()
revokeSession()

No debe contener autorización de negocio.

69. Authentication Repository

Persistencia:

User
Identity
Credential
Session
RefreshToken
MfaCredential
RecoveryCode
ExternalIdentity
VerificationToken
PasswordResetToken
70. Authentication Data Model

Modelo conceptual:

users
   │
   ├── identities
   │
   ├── credentials
   │
   ├── sessions
   │      │
   │      └── refresh_token_families
   │
   ├── mfa_credentials
   │
   ├── recovery_codes
   │
   └── external_identities
71. Credential Model

No mezclar todos los tipos de credenciales en una sola estructura rígida.

Conceptualmente:

Credential
 ├── PASSWORD
 ├── TOTP
 ├── PASSKEY
 └── EXTERNAL_IDENTITY

Esto permite extensibilidad.

72. Session Storage

Las sesiones pueden persistirse en PostgreSQL.

Redis puede utilizarse para:

Rate Limits
Temporary Challenges
Short-lived State
Caching
Distributed Session Signals

La decisión concreta depende de la necesidad de revocación y escala.

73. Authentication Cache

No cachear indiscriminadamente información sensible.

Puede cachearse:

Public Identity Metadata
JWKS
Provider Configuration
Rate Limit Counters
74. Security Events

Authentication debe producir eventos:

UserRegistered
UserLoggedIn
LoginFailed
MfaChallengeCreated
MfaVerified
PasswordChanged
PasswordResetRequested
PasswordResetCompleted
SessionCreated
SessionRevoked
RefreshTokenRotated
RefreshTokenReuseDetected
EmailVerified
ExternalIdentityLinked
75. Authentication Audit

Registrar eventos de seguridad:

timestamp
userId
tenantId
event
sessionId
ip
userAgent
requestId
result

Evitar secretos.

76. Suspicious Login

Un login sospechoso puede producir:

Security Alert

Ejemplos:

New device
Unusual location
Impossible travel
Repeated failures
Compromised credentials

El sistema debe tratar estos indicadores como señales, no necesariamente como prueba definitiva.

77. Security Notifications

Eventos importantes pueden notificar al usuario:

New Login
Password Changed
MFA Enabled
MFA Disabled
New External Login
Session Revoked
78. Email Security

Emails de autenticación deben incluir:

Verification
Password Reset
Security Alert
MFA Changes

Los enlaces deben:

Expire
Be Single-use
Use HTTPS
Avoid Sensitive Data in URL where possible
79. Authentication API

Endpoints principales:

POST /api/v1/auth/register
POST /api/v1/auth/login
POST /api/v1/auth/refresh
POST /api/v1/auth/logout
POST /api/v1/auth/logout-all

POST /api/v1/auth/email/verify
POST /api/v1/auth/email/resend

POST /api/v1/auth/password/forgot
POST /api/v1/auth/password/reset
POST /api/v1/auth/password/change

GET  /api/v1/auth/sessions
DELETE /api/v1/auth/sessions/{id}

POST /api/v1/auth/mfa/enroll
POST /api/v1/auth/mfa/verify
POST /api/v1/auth/mfa/disable

Los nombres definitivos podrán ajustarse durante el API contract review.

80. Current User

Endpoint:

GET /api/v1/auth/me

Respuesta conceptual:

{
  "data": {
    "id": "user_123",
    "email": "user@example.com",
    "emailVerified": true
  }
}

No incluir credenciales.

81. Current Sessions
GET /api/v1/auth/sessions

Respuesta:

{
  "data": [
    {
      "id": "session_123",
      "device": "Chrome",
      "lastActivityAt": "2026-10-01T15:00:00Z",
      "current": true
    }
  ]
}
82. Revoke Session
DELETE /api/v1/auth/sessions/{sessionId}

Debe validar:

session belongs to authenticated user

y, cuando corresponda, tenant context.

83. API Keys

Authentication de usuarios y API keys deben mantenerse separadas.

User Authentication
      │
      └── Human identity

API Key
      │
      └── Machine identity

Las API Keys se definirán con mayor profundidad en la arquitectura de Integrations/Platform Security.

84. Service-to-Service Authentication

Para servicios internos:

Service
 ↓
Service Identity
 ↓
Credential
 ↓
Authenticated Request

No utilizar credenciales de usuarios humanos para comunicación interna.

85. Machine Identity

EVOXA podrá utilizar:

OAuth Client Credentials
Service Accounts
Workload Identity
Signed Requests

según el entorno.

86. Agent Identity

Los agentes deben tener identidad propia:

Agent
 ↓
Agent Identity
 ↓
Agent Credentials
 ↓
Agent Session / Execution

Nunca deben recibir automáticamente el refresh token del usuario.

87. User Impersonation

La impersonación administrativa, si existe, debe ser:

Explicit
Authorized
Audited
Time-limited
Highly Restricted

Ejemplo:

Admin
 ↓
Impersonation Request
 ↓
Approval / Policy
 ↓
Temporary Session
 ↓
Audit
88. Authentication and Billing

Operaciones de billing sensibles pueden requerir:

Authentication
+
Step-Up
+
Authorization

Authentication por sí sola no concede permisos financieros.

89. Authentication and Security

E04 debe integrarse con:

Security Architecture
      │
      ├── Threat Detection
      ├── Audit
      ├── Secrets
      ├── Key Management
      └── Incident Response
90. Authentication and Observability

Métricas:

login_success_total
login_failure_total
mfa_success_total
mfa_failure_total
token_refresh_total
token_refresh_failure_total
session_created_total
session_revoked_total
password_reset_total
91. Authentication Alerts

Alertas potenciales:

Excessive Login Failures
Refresh Token Reuse
Mass Session Revocation
MFA Disabled
Suspicious Login Pattern
Credential Attack
92. Authentication Rate Limits

Ejemplos:

/login
/register
/password/forgot
/password/reset
/mfa/verify
/token/refresh

deben tener límites específicos.

Los valores deben ser configurables y adaptables por riesgo.

93. Enumeration Protection

Los endpoints sensibles no deben revelar:

Email Exists
User Exists
Tenant Exists

cuando esto permita enumeración.

Especialmente:

Forgot Password
Registration
Login
External Identity
94. Timing Protection

La comparación de credenciales debe utilizar mecanismos seguros.

No revelar mediante tiempos claramente diferenciables:

User does not exist

vs:

Password incorrect
95. Secrets Management

Secretos como:

JWT signing keys
OAuth client secrets
SMTP credentials
TOTP encryption keys
Database credentials

deben vivir en:

Secrets Manager
Environment-secured configuration
KMS

según infraestructura.

Nunca:

Git repository
Source code
Docker image
Logs
96. Key Management

La arquitectura debe separar:

Application
      ↓
Key Management
      ↓
Cryptographic Keys

Las aplicaciones no deben administrar manualmente claves maestras cuando exista una infraestructura apropiada.

97. Authentication Failure Model

Cuando Authentication falle:

Client
 ↓
Safe Error

Nunca devolver:

Stack Trace
SQL Error
Hash
Internal Provider Details
Secret Information
98. Authentication Resilience

Si un proveedor externo está temporalmente caído:

OAuth Provider
 ↓
Unavailable

EVOXA debe:

Fail Gracefully
Provide Safe Error
Avoid Session Corruption
Retry Where Appropriate
99. Authentication Availability

Authentication es un componente crítico.

Debe diseñarse para:

High Availability
Horizontal Scaling
Stateless API Nodes
Shared Session State
Distributed Rate Limits
Key Availability
100. Stateless Access Tokens

Los API nodes deben poder validar Access Tokens sin almacenar una sesión local.

API Node A
API Node B
API Node C
      ↓
Same Token Validation Rules

La información necesaria para validación debe estar disponible de forma distribuida.

101. Stateful Refresh Sessions

Aunque Access Tokens puedan ser stateless, refresh sessions deben mantener estado suficiente para:

Revocation
Rotation
Reuse Detection
Logout
Security Controls
102. Authentication Deployment
                    Internet
                       │
                     WAF
                       │
                 API Gateway
                       │
          ┌────────────┼────────────┐
          │            │            │
       API Node     API Node     API Node
          │            │            │
          └────────────┼────────────┘
                       │
                Authentication
                       │
              ┌────────┴────────┐
              │                 │
           PostgreSQL         Redis
              │                 │
              └────────┬────────┘
                       │
                  Secrets/KMS
103. Authentication Testing

Debe probarse:

Registration
Login
Invalid Login
Refresh
Refresh Rotation
Refresh Reuse
Logout
Logout All
Email Verification
Password Reset
Password Change
MFA Enrollment
MFA Verification
MFA Recovery
Session Revocation
OAuth Login
Tenant Switching
104. Security Testing

Además:

Brute Force
Credential Stuffing
Token Replay
Token Tampering
JWT Algorithm Confusion
Expired Tokens
Invalid Audience
Invalid Issuer
Session Hijacking
CSRF where applicable
IDOR
User Enumeration
Rate Limit Bypass
105. Authentication Contract Testing

Debe verificarse:

Status Codes
Response Schema
Token Schema
Error Schema
Expiration
Claims
Refresh Behavior
Session Behavior
106. Authentication Lifecycle
                    REGISTER
                       │
                       ▼
                  EMAIL VERIFY
                       │
                       ▼
                   ACTIVE USER
                       │
                       ▼
                      LOGIN
                       │
                       ▼
                 MFA CHALLENGE
                       │
                       ▼
                    SESSION
                       │
              ┌────────┴────────┐
              │                 │
          ACCESS TOKEN     REFRESH TOKEN
              │                 │
              │              ROTATION
              │                 │
              └────────┬────────┘
                       │
                    LOGOUT
                       │
                       ▼
                  SESSION REVOKED
107. Password Lifecycle
Password Created
       ↓
Password Hash
       ↓
Password Used
       ↓
Password Change
       ↓
Password Hash Replaced
       ↓
Sessions Reviewed

Reset:

Forgot Password
 ↓
Reset Token
 ↓
New Password
 ↓
Invalidate Existing Sessions
108. MFA Lifecycle
MFA Disabled
      ↓
Enroll
      ↓
Verify
      ↓
MFA Enabled
      ↓
Challenge
      ↓
Verify
      ↓
Authenticated

Disable:

Step-Up Authentication
 ↓
Disable MFA
 ↓
Security Event
109. Session Lifecycle
Created
  ↓
Active
  ├── Refresh
  ├── Activity
  └── MFA Step-Up
  ↓
Expired / Revoked
110. Authentication Architecture Principles
01 — Authentication Must Be Centralized
02 — Identity and Authentication Must Be Separated
03 — Passwords Must Never Be Stored in Plaintext
04 — Access Tokens Must Be Short-Lived
05 — Refresh Tokens Must Rotate
06 — Refresh Token Reuse Must Be Detected
07 — Sessions Must Be Revocable
08 — MFA Must Be Supported
09 — Sensitive Operations Require Step-Up Authentication
10 — Authentication Must Be Tenant-Aware
11 — External Identities Must Be Explicitly Linked
12 — Authentication Secrets Must Be Protected
13 — Security Events Must Be Audited
14 — Authentication Must Be Rate-Limited
15 — Authentication Must Resist Enumeration
16 — Authentication Failures Must Not Leak Internals
17 — Machine Identities Must Be Separate From Human Identities
18 — Agents Must Have Independent Identity
19 — Authentication Must Be Observable
20 — Authentication Must Be Horizontally Scalable
111. Definition of Done

E04 queda definido cuando EVOXA posee una arquitectura para:

✓ User Registration
✓ Login
✓ Logout
✓ Logout All
✓ Access Tokens
✓ Refresh Tokens
✓ Token Rotation
✓ Token Reuse Detection
✓ Session Management
✓ Session Revocation
✓ Device Management
✓ Email Verification
✓ Password Reset
✓ Password Change
✓ Password Security
✓ MFA
✓ TOTP
✓ Recovery Codes
✓ Step-Up Authentication
✓ OAuth/OIDC
✓ External Identities
✓ Enterprise SSO
✓ Tenant Authentication
✓ Service Authentication
✓ Machine Identities
✓ Agent Identity
✓ Security Events
✓ Audit
✓ Rate Limiting
✓ Enumeration Protection
✓ Secrets Management
✓ Key Rotation
✓ Observability
✓ High Availability
✓ Authentication Testing
112. Relación E01 → E04

La especificación de Engineering queda evolucionando:

E01
Backend Architecture
        │
        ▼
E02
Database Architecture
        │
        ▼
E03
API Architecture
        │
        ▼
E04
Authentication Architecture
        │
        ▼
E05
Authorization Architecture

La separación fundamental será:

             EVOXA SECURITY
                    │
          ┌─────────┴─────────┐
          │                   │
   AUTHENTICATION        AUTHORIZATION
          │                   │
     "Who are you?"      "What can you do?"
          │                   │
         E04                 E05

E04 define la identidad y la autenticación; E05 deberá definir RBAC, roles, permisos, policies, scopes, ABAC, resource-level authorization, tenant permissions y enforcement de autorización dentro de EVOXA.
