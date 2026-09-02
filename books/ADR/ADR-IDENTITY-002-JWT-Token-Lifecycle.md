ADR-IDENTITY-002 — JWT & Token Lifecycle

Proyecto: EVOXA Platform
Dominio: Identity
ADR: ADR-IDENTITY-002
Título: JWT & Token Lifecycle
Estado: Proposed
Versión: 1.0.0
Relacionado con: ESP-0001 — Identity, capítulos 03 y 06
Relacionado con: ADR-IDENTITY-001
Tipo: Architecture Decision Record

1. Contexto

El dominio Identity necesita un mecanismo seguro para mantener la autenticación de los usuarios después del login y permitir acceso a las APIs de EVOXA.

Los capítulos ESP-0001 / 03 — Authentication & Authorization y ESP-0001 / 06 — Sessions & Token Management establecen conceptualmente:

Authentication
      ↓
Session
      ↓
Access Token
      +
Refresh Token

El Blueprint define además APIs versionadas, autenticación mediante Bearer/JWT, refresh tokens, sesiones, seguridad, auditabilidad y trazabilidad.

La arquitectura propuesta para Identity utiliza:

JWT para access tokens.
Sesiones persistentes.
Refresh tokens persistentes y revocables.
Refresh token rotation.
Detección de reuse.
Logout y revocación.
Tenant/security context.
Audit y eventos.

Sin embargo, los capítulos anteriores dejan abiertas varias decisiones concretas sobre el lifecycle de los tokens.

Este ADR formaliza esas decisiones.

2. Problema

Necesitamos definir:

Qué representa un access token.
Qué representa un refresh token.
Cuánto duran.
Cómo se generan.
Cómo se validan.
Cómo se revocan.
Cómo se rotan.
Cómo detectar reutilización.
Cómo se relacionan con las sesiones.
Qué ocurre durante logout.
Qué ocurre ante suspensión de un usuario.
Qué ocurre ante compromiso de tokens.
Qué información puede contener un JWT.
Cómo manejar múltiples dispositivos.
Cómo realizar key rotation.
3. Objetivos

La estrategia deberá proporcionar:

Seguridad.
Revocación controlada.
Protección contra replay.
Stateless validation para access tokens.
Stateful lifecycle para sesiones/refresh tokens.
Soporte multi-device.
Tenant isolation.
Auditabilidad.
Observabilidad.
Compatibilidad con APIs de EVOXA.
Escalabilidad horizontal.
Capacidad de evolución.
4. Decisión general

Se utilizará un modelo híbrido:

                 Authentication
                       │
                       ▼
                    Session
                  /         \
                 /           \
                ▼             ▼
        Access Token     Refresh Token
            JWT             Stateful
             │                 │
             ▼                 ▼
       Stateless API       Persistence
       validation          + Rotation
                              + Revocation
Decisión

Access Token: JWT stateless

Refresh Token: stateful y persistido

Session: stateful y persistida

Esta combinación permite validar rápidamente access tokens sin consultar la base de datos en cada request, mientras mantiene control sobre sesiones y refresh tokens.

5. Access Token
Decisión

Los access tokens serán:

JWT Bearer Tokens

y se enviarán mediante:

Authorization: Bearer <access_token>

La API utilizará:

/api/v1/...

como esquema de versionamiento.

6. Características del Access Token

El access token deberá ser:

Firmado.
De corta duración.
No modificable por el cliente.
Validable independientemente.
Asociado a un usuario.
Asociado a una sesión.
Limitado al contexto de seguridad necesario.

No deberá contener secretos ni información innecesaria.

7. JWT Claims

El JWT deberá utilizar, como mínimo cuando corresponda:

{
  "sub": "user-id",
  "iss": "evoxa-identity",
  "aud": "evoxa-api",
  "iat": 0,
  "exp": 0,
  "jti": "token-id",
  "session_id": "session-id"
}
Significado
Claim	Propósito
sub	Usuario
iss	Emisor
aud	Audiencia
iat	Fecha de emisión
exp	Expiración
jti	Identificador único
session_id	Sesión asociada

El Blueprint establece conceptualmente estos elementos para el contexto de autenticación, pero no fija un conjunto definitivo de claims ni sus nombres exactos. Por lo tanto, la lista anterior es una decisión derivada de implementación.

8. Información que NO deberá estar en el JWT

No deberán incluirse:

password
password_hash
refresh_token
MFA_secret
recovery_codes
private_keys
API secrets
sensitive health information

Tampoco deberá incluirse todo el catálogo de permisos si esto produce tokens excesivamente grandes o genera problemas de revocación/evolución.

9. Access Token TTL
Decisión propuesta

El access token será short-lived.

Sin embargo:

El Blueprint no define un valor concreto para el TTL.

Por lo tanto, este ADR no debe inventar un valor definitivo.

Se recomienda que el TTL sea una configuración centralizada:

ACCESS_TOKEN_TTL

y que el valor inicial sea decidido mediante configuración de seguridad/ADR de hardening.

Estado
Decision: Short-lived
Exact TTL: OPEN
10. Refresh Token

El refresh token tendrá una función diferente al access token.

Access Token
→ acceder a APIs

Refresh Token
→ obtener nuevo Access Token

El refresh token no deberá utilizarse directamente para acceder a recursos de negocio.

11. Refresh Token Persistence

Los refresh tokens deberán estar asociados a una sesión.

Modelo conceptual:

User
  │
  └── Session
         │
         └── Refresh Token Family
                │
                ├── Token A
                ├── Token B
                └── Token C

La base de datos almacenará una representación segura del token, no el secreto recuperable en texto plano.

12. Refresh Token Hashing

Se deberá persistir un hash/representación no reversible del refresh token.

Conceptualmente:

Refresh Token
      ↓
Secure Hash
      ↓
Database

El token original solamente estará disponible para el cliente.

13. Refresh Token Rotation
Decisión

Se utilizará Refresh Token Rotation.

Flujo:

Refresh Token A
      ↓
     use
      ↓
Refresh Token B
      ↓
Token A invalidated

Cada refresh exitoso genera una nueva credencial de refresh.

14. Refresh Token Family

Los tokens deberán poder relacionarse con una familia de rotación.

Ejemplo:

Family F1

Token A
  ↓
Token B
  ↓
Token C
  ↓
Token D

Esto permitirá detectar comportamientos anómalos y revocar la familia cuando corresponda.

15. Refresh Token Reuse Detection

Si un refresh token ya utilizado vuelve a presentarse:

Token A
 ↓
used
 ↓
Token B generated
 ↓
Token A presented again
 ↓
REUSE DETECTED

El sistema deberá:

Rechazar la operación.
Registrar evento de seguridad.
Generar audit event.
Aplicar la política de revocación correspondiente.

La respuesta exacta —por ejemplo revocar toda la familia o toda la sesión— deberá quedar alineada con la política de seguridad final.

16. Session Binding

Cada refresh token deberá estar asociado a una sesión.

Session S1
   │
   └── Refresh Family F1

Esto permite:

Logout.
Revocación remota.
Revocación por dispositivo.
Revocación global.
Investigación de incidentes.
17. Session Lifecycle

Las sesiones utilizarán:

CREATED
   ↓
ACTIVE
   ↓
EXPIRED
   ↓
REVOKED

La sesión será el elemento de control stateful principal.

18. Login

Flujo definitivo:

POST /api/v1/identity/auth/login
                  │
                  ▼
          Validate Credentials
                  │
                  ▼
           Validate Account
                  │
                  ▼
             Validate MFA
                  │
                  ▼
            Create Session
                  │
          ┌───────┴────────┐
          ▼                ▼
   Access Token      Refresh Token
          │                │
          └───────┬────────┘
                  ▼
             Client
19. Token Refresh

Flujo:

Client
  │
  │ Refresh Token
  ▼
Identity
  │
  ├── Validate token
  ├── Validate session
  ├── Check revocation
  ├── Detect reuse
  └── Rotate token
  │
  ▼
New Access Token
+
New Refresh Token
20. Logout
Logout actual
Client
 ↓
POST /auth/logout
 ↓
Current Session revoked
 ↓
Refresh Token invalidated

El access token existente podrá permanecer técnicamente válido hasta su expiración si se mantiene el modelo stateless.

Esto es precisamente una consecuencia importante del diseño.

21. Logout All

Se deberá proporcionar:

POST /api/v1/identity/auth/logout-all

Resultado:

User
 │
 ├── Session A → REVOKED
 ├── Session B → REVOKED
 ├── Session C → REVOKED
 └── Session D → REVOKED

Todos los refresh tokens asociados deberán quedar invalidados.

22. Access Token Revocation
Decisión

Los access tokens serán principalmente stateless.

Por lo tanto, no se consultará obligatoriamente PostgreSQL en cada request para verificar si un JWT sigue vigente.

Esto proporciona mejor escalabilidad.

Pero introduce una consecuencia:

Un access token válido puede continuar siendo técnicamente válido hasta su expiración, incluso después de logout, salvo que se implemente una estrategia adicional de revocación.

Esta limitación deberá ser aceptada explícitamente.

23. Immediate Revocation

Para situaciones críticas podrá existir una estrategia adicional:

Critical Revocation
       ↓
Session Revoked
       ↓
Security State
       ↓
Token rejected

Pero no se implementará automáticamente una consulta persistente por cada API request si esto contradice el objetivo de stateless validation.

24. User Suspension

Cuando un usuario sea suspendido:

User ACTIVE
    ↓
SUSPENDED

Identity deberá aplicar la política definida para sesiones.

Decisión recomendada

Revocar las sesiones activas y refresh tokens del usuario.

Esto evita que una suspensión deje sesiones renovables activas.

El tratamiento de access tokens ya emitidos dependerá de su TTL o de la estrategia de revocación inmediata.

25. Organization Suspension

Si una organización pasa a:

SUSPENDED

las sesiones relacionadas deberán quedar sujetas a revocación según la política de tenant.

La implementación deberá impedir nuevos refreshes para el tenant suspendido.

26. Membership Removal

Cuando una membership sea:

REMOVED

el usuario no deberá poder continuar utilizando esa membership.

El refresh deberá volver a evaluar:

User status.
Organization status.
Membership status.
Security policies.

Esto evita que un refresh antiguo mantenga acceso indefinidamente.

27. Multi-Device

Identity deberá soportar múltiples sesiones simultáneas.

Ejemplo:

User
 │
 ├── Session Desktop
 ├── Session Mobile
 ├── Session Tablet
 └── Session Browser

Cada sesión deberá poder revocarse individualmente.

28. Session Metadata

Una sesión podrá mantener metadata operacional como:

device_id
device_type
user_agent
ip
created_at
last_activity_at
expires_at
revoked_at

La recopilación deberá respetar privacidad y minimización de datos.

29. Maximum Sessions
Estado

OPEN

El Blueprint no define un número máximo de sesiones simultáneas.

Por lo tanto, deberá convertirse en configuración:

MAX_SESSIONS_PER_USER

si finalmente se decide aplicar ese límite.

30. Refresh Token Lifetime

El refresh token deberá tener una duración superior al access token.

Sin embargo:

El Blueprint no define un TTL concreto.

Por lo tanto:

Refresh Token TTL = configurable
Exact value = OPEN

La duración definitiva deberá establecerse en una decisión de seguridad posterior.

31. Absolute Session Lifetime

Deberá existir conceptualmente:

Session Maximum Lifetime

para evitar sesiones infinitas.

Pero:

Exact value = OPEN

hasta completar la política de seguridad.

32. Sliding Session
Estado

OPEN

Debe decidirse si una sesión puede extenderse automáticamente mediante refresh.

Posibles modelos:

Fixed
Session created
      ↓
Maximum lifetime fixed
Sliding
Active refresh
      ↓
Session expiration extended

La decisión final deberá estar en el ADR de session security.

33. JWT Issuer

Identity será el emisor de los tokens:

iss = EVOXA Identity

El valor exacto de issuer deberá ser configurable por ambiente.

34. JWT Audience

El access token deberá estar limitado a la audiencia para la que fue emitido.

Conceptualmente:

aud = EVOXA API

Los servicios deberán rechazar tokens emitidos para otra audiencia.

35. JWT Algorithm

La arquitectura requiere firma asimétrica.

Opciones contempladas en el Blueprint:

Ed25519
RSA-4096

Estado
Asymmetric signing: DECIDED
Exact algorithm: OPEN

No debemos cerrar Ed25519 o RSA-4096 sin el ADR de Cryptographic Key Management.

36. Key Rotation

La arquitectura deberá permitir:

Key K1
  ↓
Key K2
  ↓
K1 retirement

Los tokens emitidos con la clave anterior deberán poder validarse durante el período de transición definido.

Esto requiere soporte para key identifiers (kid) cuando corresponda.

37. JWKS

Cuando se utilicen claves asimétricas, Identity deberá poder exponer una representación pública de las claves para los servicios autorizados que necesiten validar JWT.

Conceptualmente:

Identity
   ↓
Public Keys
   ↓
JWKS
   ↓
API Services

La URL exacta será definida por la implementación.

38. Token Validation

La validación deberá comprobar como mínimo:

Signature
Issuer
Audience
Expiration
Issued At
Subject
JWT ID
Session context

cuando correspondan al contrato definitivo.

Nunca se deberá aceptar:

Algoritmos no permitidos.
Firma inválida.
Token expirado.
Issuer incorrecto.
Audience incorrecta.
39. Token Theft

Ante sospecha de robo de un refresh token:

Detection
 ↓
Reject token
 ↓
Revoke family
 ↓
Revoke session
 ↓
Audit
 ↓
Security Event

La política exacta deberá determinarse según severidad.

40. Replay Protection

Los refresh tokens deberán tener protección contra replay.

Los access tokens dependen principalmente de:

Short TTL.
Signature.
Expiration.
Session context.
Revocation strategy cuando corresponda.
41. Browser Storage
Estado

OPEN

El Blueprint no especifica una estrategia definitiva para almacenar access/refresh tokens en browser.

Opciones:

HttpOnly Secure SameSite Cookies

o:

Authorization Header
+
Secure client-side storage

La decisión deberá quedar en un ADR de client/token storage.

42. Mobile Storage

Para aplicaciones móviles, los tokens deberán almacenarse utilizando mecanismos seguros del sistema operativo.

La implementación deberá evitar almacenamiento en:

Plain Shared Preferences
Unencrypted files
Logs
Local databases without encryption

La implementación concreta dependerá del SDK móvil de EVOXA.

43. Token Size

Los JWT deberán mantenerse pequeños.

No deberán utilizarse como base de datos de permisos.

En lugar de:

JWT
 └── 500 permissions

se recomienda:

JWT
 └── Identity Context
       ↓
Authorization Service
       ↓
Permissions / Policies

Esto facilita evolución y revocación.

44. Authorization Context

El JWT proporcionará identidad y contexto mínimo.

Identity Authorization podrá posteriormente resolver:

User
Organization
Membership
Role
Permission
Policy
Ownership

Los dominios de negocio seguirán siendo responsables de reglas específicas de sus recursos.

45. Correlation & Traceability

Cada operación de token deberá poder correlacionarse con:

trace_id
correlation_id
session_id
user_id
tenant_id
jti

No se deberá registrar el token completo.

46. Audit Events

Deberán generarse eventos de audit para:

UserAuthenticated
AuthenticationFailed
SessionCreated
SessionRevoked
RefreshTokenRotated
RefreshTokenRevoked
TokenReuseDetected
Logout
LogoutAll

Los eventos de Identity deberán seguir el envelope definido en la arquitectura de eventos de EVOXA.

47. Security Events

Algunos eventos deberán clasificarse además como security events:

TokenReuseDetected
AuthenticationAttackDetected
SuspiciousRefresh
SessionAnomaly
CredentialAttack

Estos podrán alimentar posteriormente Security Operations y Analytics.

48. Error Responses

Las APIs nunca deberán revelar información sensible sobre el estado interno del token.

Ejemplo:

{
  "error": {
    "code": "INVALID_REFRESH_TOKEN",
    "message": "Refresh token is invalid.",
    "correlation_id": "..."
  }
}

No se deberá devolver información como:

"Token belongs to user X"
"Token expired because..."
"User exists but..."

cuando dicha información facilite enumeración o ataques.

49. Rate Limiting

Se deberá aplicar rate limiting especialmente a:

POST /auth/login
POST /auth/refresh
POST /mfa/*
POST /auth/password-reset

Los thresholds serán configurables.

50. Concurrency

El refresh deberá ser seguro frente a solicitudes simultáneas:

Request A ── Token A
Request B ── Token A

No deberán generarse dos cadenas válidas inconsistentes.

La operación de rotation deberá ser atómica o protegida mediante mecanismos de concurrencia apropiados.

51. Persistence Model

El modelo mínimo deberá permitir:

Session
├── id
├── user_id
├── organization_id
├── status
├── created_at
├── expires_at
├── revoked_at
└── metadata

RefreshToken
├── id
├── session_id
├── family_id
├── token_hash
├── status
├── created_at
├── expires_at
├── used_at
└── revoked_at

Los nombres exactos de columnas serán definidos durante la implementación/migrations.

52. Revocation Model

La revocación deberá soportar:

Token-level
Session-level
User-level
Organization-level
Security-incident-level

Ejemplo:

Token revoked
        ↓
Only token

Session revoked
        ↓
All refresh tokens in session

User security reset
        ↓
All sessions

Organization suspended
        ↓
Affected tenant sessions
53. Caching

La validación normal de access tokens deberá permanecer stateless.

Podrán utilizarse caches para:

Public keys.
Security policy.
Authorization context.
Revocation state crítica.

Pero cualquier cache de seguridad deberá tener una estrategia de invalidación clara.

54. Failure Behavior

Si Identity no puede verificar una condición de seguridad crítica:

Unknown security state
        ↓
DENY

Nunca:

Unknown
 ↓
ALLOW

Esto es obligatorio por el principio de fail-closed.

55. Testing Requirements

Deberán existir pruebas para:

Access token
Valid.
Expired.
Invalid signature.
Wrong issuer.
Wrong audience.
Invalid claims.
Manipulated token.
Refresh token
Valid refresh.
Rotation.
Reuse detection.
Revocation.
Expiration.
Concurrent refresh.
Family revocation.
Session
Creation.
Logout.
Logout all.
Expiration.
Revocation.
User suspension.
Security
Replay.
Token theft scenario.
Privilege escalation.
Tenant escape.

Estos requisitos están alineados con ESP-0001 / 13 — Identity Testing & Quality Requirements.

56. Operational Requirements

Identity deberá monitorizar:

Authentication success
Authentication failures
Token refresh rate
Token refresh failures
Token reuse detection
Session creation
Session revocation
MFA failures
Latency
Error rate

Un aumento anormal de TokenReuseDetected deberá generar una alerta de seguridad.

57. Performance

La estrategia deberá preservar:

Stateless Access Token Validation

para evitar una consulta a PostgreSQL en cada request.

El objetivo arquitectónico general es:

Authentication P95 < 500 ms
API P95 < 300 ms
58. Decision Matrix
Decisión	Estado
JWT Access Token	✅ Decided
Bearer Authentication	✅ Decided
Stateless Access Validation	✅ Decided
Stateful Sessions	✅ Decided
Stateful Refresh Tokens	✅ Decided
Refresh Rotation	✅ Decided
Reuse Detection	✅ Decided
Token Family	✅ Decided
Session Binding	✅ Decided
Asymmetric Signing	✅ Decided
Access Token Short TTL	✅ Decided
Exact Access TTL	🟡 Open
Exact Refresh TTL	🟡 Open
Max Session Lifetime	🟡 Open
Sliding Sessions	🟡 Open
Max Concurrent Sessions	🟡 Open
Ed25519 vs RSA-4096	🟡 Open
Browser Storage	🟡 Open
Immediate JWT Revocation	🟡 Open
Key Rotation Period	🟡 Open
59. Consequences
Positivas

El diseño proporciona:

Escalabilidad horizontal.
Menos consultas a PostgreSQL.
Revocación de refresh tokens.
Control por sesión.
Multi-device.
Protección contra refresh replay.
Seguridad mejorada.
Auditabilidad.
Compatibilidad con microservicios.
Evolución futura.
Negativas

Existe una complejidad inherente:

Gestión de sesiones.
Gestión de refresh token families.
Revocation.
Key rotation.
Concurrency control.
Event handling.
Client storage.
Security monitoring.
60. Flujo definitivo

El lifecycle principal queda:

                         LOGIN
                           │
                           ▼
                    Authenticate User
                           │
                           ▼
                       MFA Check
                           │
                           ▼
                    Create Session
                           │
                ┌──────────┴──────────┐
                ▼                     ▼
          Access Token          Refresh Token
             JWT                  Stateful
                │                     │
                ▼                     ▼
          API Requests          Token Rotation
                │                     │
                │                ┌────┴────┐
                │                ▼         ▼
                │             Valid      Reuse
                │                │         │
                │                ▼         ▼
                │             Rotate    Security
                │                         Incident
                │
                ▼
             Expiration
61. Decisión arquitectónica resultante

La decisión central de este ADR es:

EVOXA Identity utilizará JWT Bearer Access Tokens de corta duración para acceso stateless a APIs, combinados con sesiones persistentes y refresh tokens stateful con rotation, revocación y detección de reutilización.

Esta decisión queda alineada con la arquitectura API-First, Security by Design, Stateless APIs y el modelo de sesiones/tokens definido en ESP-0001.

62. ADRs derivados

Este ADR deja decisiones especializadas para documentos posteriores:

ADR-IDENTITY-003
Refresh Token Security & Session Management

ADR-IDENTITY-004
MFA Architecture

ADR-IDENTITY-005
Authorization & Policy Engine

ADR-IDENTITY-006
Identity Persistence Strategy

ADR-IDENTITY-007
Identity Event & Outbox Strategy

ADR-IDENTITY-008
Identity Cryptographic Key Management

ADR-IDENTITY-009
Identity Client Token Storage

ADR-IDENTITY-010
Identity Testing Strategy

ADR-IDENTITY-011
Identity Deployment & Infrastructure
63. Estado del ADR

Status: Proposed

No lo marcaría todavía como Accepted porque todavía existen decisiones de seguridad que afectan directamente al comportamiento final:

TTL exactos.
Algoritmo criptográfico.
Key rotation.
Browser storage.
Session lifetime.
Sliding sessions.
Revocación inmediata.

Eso es deliberado: no debemos inventar estos valores solo para poder programar.

64. Relación con ESP-0001
ESP-0001 Identity
│
├── 01 Domain Model
├── 02 Architecture
├── 03 Authentication & Authorization
│        └── ADR-IDENTITY-002
├── 04 User & Organization
├── 05 Roles & Permissions
├── 06 Sessions & Tokens
│        └── ADR-IDENTITY-002
├── 07 MFA
├── 08 Security Policies
├── 09 Audit
├── 10 API
├── 11 Persistence
├── 12 Events
├── 13 Testing
├── 14 Implementation
└── 15 Deployment

Este ADR ya nos permite fijar la columna vertebral del sistema de autenticación, pero todavía no conviene pasar a código. El siguiente ADR crítico sería ADR-IDENTITY-003 — Refresh Token Security & Session Management, donde podemos cerrar las decisiones que quedaron abiertas aquí: TTL, lifetime de sesión, concurrencia, límites de sesiones, revocación, logout y comportamiento ante suspensión/compromiso.
