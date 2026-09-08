ESP-0001 / 06 — Identity Sessions & Token Management

Document ID: ESP-0001/06
Module: Identity
Version: 1.1.0
Status: Draft — Consolidated
Parent: ESP-0001 — Identity

Target Repository: evoxa-platform
Target Path: apps/api/app/domains/identity/

1. Purpose

Esta especificación define el modelo de:

Sessions
Access Tokens
Refresh Tokens
JWT
Token validation
Token rotation
Token families
Token reuse detection
Session revocation
Token revocation
Logout
Global logout
Concurrent sessions
Device sessions
Security context
Token/session lifecycle
Security events
Audit
Authorization integration
Tenant context
Account and membership security state

El objetivo es proporcionar un mecanismo seguro para mantener una autenticación activa sin convertir los Access Tokens en credenciales permanentes.

2. Fundamental Model

El modelo consolidado es:

USER
 │
 ▼
SESSION
 │
 ├───────────────┐
 ▼               ▼
ACCESS TOKEN   REFRESH TOKEN
                   │
                   ▼
              TOKEN FAMILY

Las responsabilidades son diferentes:

Session
    = authenticated security context

Access Token
    = short-lived API credential

Refresh Token
    = credential used to obtain new Access Token

Una sesión no debe confundirse con un token.

3. Core Architectural Decision

EVOXA utiliza un modelo híbrido:

Access Token
    ↓
Stateless JWT validation

Session
    ↓
Stateful

Refresh Token
    ↓
Stateful

Refresh Token Family
    ↓
Stateful

Esto permite:

validación local rápida de Access Tokens;
revocación controlada;
detección de reutilización;
gestión de múltiples dispositivos;
seguridad frente a compromiso de credenciales.

Esta decisión consolida ADR-002 y ADR-003.

4. Session Definition

Una Session representa una instancia autenticada de un usuario.

Ejemplo:

User
 │
 ├── Session A → Web
 ├── Session B → Mobile
 └── Session C → Tablet

Cada sesión puede tener:

user
organization context
membership
authentication method
MFA state
device context
security metadata
timestamps
status
5. Session Lifecycle

El lifecycle consolidado es:

             CREATED
                │
                ▼
              ACTIVE
             /      \
            /        \
       EXPIRE       REVOKE
          │             │
          ▼             ▼
       EXPIRED       REVOKED

Una sesión en estado terminal no puede volver a ACTIVE.

Estados conceptuales:

CREATED
ACTIVE
EXPIRED
REVOKED

El modelo físico definitivo se especificará en ESP-0001/11.

6. Session Attributes

El agregado Session puede contener conceptualmente:

Session
├── id
├── user_id
├── organization_id
├── membership_id
├── status
├── authentication_method
├── created_at
├── expires_at
├── last_activity_at
├── revoked_at
├── revocation_reason
├── device_id
├── client_metadata
├── security_context
└── version

La especificación actual también contempla ip_address y user_agent como metadata de seguridad, pero deben tratarse como información potencialmente sensible.

7. Session Context

La sesión mantiene el contexto necesario para relacionar la autenticación con Identity.

Session
├── User
├── Organization
├── Membership
├── Authentication Method
├── MFA Evidence
├── Device Context
└── Security State

Esto es especialmente importante para el modelo multi-organización.

8. Organization Context

Una sesión puede establecerse dentro de una Organization concreta:

User
   │
   ▼
Membership
   │
   ▼
Organization
   │
   ▼
Session

Pero:

Session
    ≠
permanent authorization

La autorización continúa siendo evaluada por Authorization Runtime.

9. Authentication → Session

El flujo consolidado es:

Authentication Request
        │
        ▼
Credential Validation
        │
        ▼
Account Validation
        │
        ▼
Organization/Membership Context
        │
        ▼
Security Policy
        │
        ▼
MFA if required
        │
        ▼
Create Session
        │
        ▼
Issue Access Token
        │
        ▼
Issue Refresh Token

Una sesión sólo debe crearse después de una autenticación válida.

10. Access Token

El Access Token es una credencial temporal utilizada para acceder a APIs protegidas.

EVOXA utiliza JWT como mecanismo para Access Tokens.

Client
   │
   │ Authorization: Bearer <token>
   ▼
API
   │
   ▼
JWT Validation
   │
   ▼
Security Context
   │
   ▼
Authorization Runtime
11. JWT

El Access Token JWT debe validarse criptográficamente y semánticamente.

Validaciones conceptuales:

Signature
Issuer
Audience
Expiration
Not Before
Required Claims
Algorithm
Key ID

La firma debe utilizar criptografía asimétrica y claves administradas mediante el mecanismo definido por ADR-008.

12. JWT Signing

El modelo consolidado utiliza:

Asymmetric Signing
        +
JWKS
        +
kid
        +
Key Rotation

El servicio que valida el JWT no debe necesitar conocer una clave privada de firma.

Private Key
     ↓
Token Issuer

Public Key
     ↓
Token Validators

La rotación de claves debe permitir coexistencia controlada de claves durante la transición.

13. JWT Claims

Los claims conceptuales incluyen:

sub
iss
aud
iat
exp
jti
session_id
tenant/context
security context

El JWT debe contener solamente la información necesaria.

No debe contener:

password
MFA secrets
refresh tokens
health data
sensitive profile data
full permission catalog
internal secrets

Esto mantiene coherencia con ADR-002 y ADR-008.

14. JWT ≠ Session

Un JWT válido criptográficamente no significa necesariamente:

Session = ACTIVE

ni:

User = ACTIVE

ni:

Membership = ACTIVE

ni:

Authorization = ALLOW

Por tanto:

Valid JWT
   ≠
Active Session
   ≠
Authorized Request

Esta separación es fundamental.

15. Access Token Lifetime

La arquitectura establece que el Access Token debe ser de corta duración.

La relación normativa es:

Access Token Lifetime
        <
Refresh Token Lifetime
        ≤
Session Maximum Lifetime

Los valores concretos de TTL deben ser parámetros de configuración/gobierno y no deben inventarse dentro de esta ESP.

16. Refresh Token

El Refresh Token permite obtener un nuevo Access Token sin repetir toda la autenticación.

Flujo:

Refresh Token
      │
      ▼
Validate Token
      │
      ▼
Validate Session
      │
      ▼
Validate User
      │
      ▼
Validate Membership / Context
      │
      ▼
Validate Security Policy
      │
      ▼
Rotate Refresh Token
      │
      ▼
Issue New Access Token
17. Refresh Token Characteristics

El Refresh Token debe ser:

Opaque
Cryptographically Random
Stateful
Short-lived
Rotatable
Revocable
Family-bound
Protected at Rest

No debe utilizarse como JWT público con información de usuario.

18. Refresh Token Storage

El token presentado por el cliente no debe almacenarse en texto plano.

El modelo conceptual:

Client
   │
   │ Refresh Token
   ▼
Identity
   │
   ▼
Secure Hash/HMAC Verification
   │
   ▼
Stored Token Record

El mecanismo criptográfico exacto queda gobernado por ADR-025 y las decisiones de Security/Infrastructure.

19. Refresh Token Entity

Conceptualmente:

RefreshToken
├── id
├── session_id
├── family_id
├── parent_token_id
├── token_hash
├── status
├── created_at
├── expires_at
├── used_at
├── revoked_at
└── revocation_reason

Estados conceptuales:

ACTIVE
USED
REVOKED
EXPIRED

La representación física definitiva corresponde a ESP-0001/11.

20. Refresh Token Rotation

La rotación es obligatoria dentro del modelo consolidado.

Token A
   │
   │ refresh
   ▼
Token B

Resultado:

A = USED
B = ACTIVE

Un Refresh Token utilizado exitosamente no debe volver a ser reutilizable.

21. Token Family

Los Refresh Tokens pertenecen a una familia:

Family F1
│
├── Token A
│
├── Token B
│
├── Token C
│
└── Token D

Relación:

A → B → C → D

Esto permite detectar reutilización de un token antiguo.

22. Refresh Token Reuse Detection

Caso:

A → refresh → B

Posteriormente:

A → refresh

El sistema detecta:

A = already USED

Resultado mínimo:

DENY
+
Security Event
+
Audit

Y, bajo la política definida por ADR-003/025:

Revoke Token Family
        +
Revoke Session
23. Reuse Is a Security Signal

La reutilización de un Refresh Token previamente consumido puede indicar:

Credential Theft
Replay
Token Copy
Client Compromise
Race/Retry Anomaly

Por ello no debe tratarse simplemente como:

Invalid Token

sin registrar el evento de seguridad correspondiente.

24. Concurrency

Debe protegerse el siguiente escenario:

Client A ──┐
           ├── Refresh Token A
Client B ──┘

Ambos requests llegan simultáneamente.

No se debe producir:

A → B
A → C

de forma válida.

La rotación debe ser:

Atomic
Concurrency Safe
Idempotency-aware

y debe existir un único resultado válido.

25. Network Retry vs Malicious Replay

La implementación debe distinguir, cuando sea posible, entre:

Legitimate Network Retry

y:

Suspicious Token Reuse

La política exacta de:

grace window;
retry handling;
idempotency;
concurrent refresh;

debe implementarse conforme a ADR-025 y quedar documentada antes de producción.

No se debe interpretar automáticamente toda repetición como ataque sin considerar el modelo de concurrencia definido.

26. Session Revocation

Revocar una Session significa:

Session
   ↓
REVOKED

y las credenciales asociadas deben quedar impedidas de continuar utilizando esa sesión según la estrategia de validación vigente.

Debe revocarse también el Refresh Token/Family asociado cuando corresponda.

27. Token Revocation Causes

Una credencial puede revocarse por:

Logout
Session Revocation
Password Security Event
Account Suspension
Membership Revocation
MFA Security Change
Token Reuse
Administrative Action
Security Incident

La acción exacta depende del tipo de incidente.

28. Security State Changes

Eventos de seguridad importantes pueden requerir revocación:

Account Suspended
Account Disabled
Password Compromised
Membership Removed
MFA Factor Revoked
Refresh Token Reuse
Security Incident

El principio consolidado es:

Un cambio de estado de seguridad no debe quedar neutralizado por un Access Token o cache antiguo.

29. Account Suspension

Cuando una cuenta pasa a:

SUSPENDED

la política de seguridad puede revocar sus sesiones activas.

Para eventos de seguridad críticos, el comportamiento recomendado es:

Account Suspension
       ↓
Revoke Sessions
       ↓
Revoke Refresh Credentials
       ↓
Security Event
       ↓
Audit
30. Membership Revocation

El acceso a una Organization depende de Membership.

Si:

Membership → REVOKED

entonces el usuario no puede continuar actuando dentro de esa Organization.

User
 ↓
Session
 ↓
Organization A
 ↓
Membership REVOKED
 ↓
DENY

No debe asumirse necesariamente que todas las sesiones globales del usuario deben ser destruidas.

La revocación debe respetar el alcance del contexto.

31. Organization Suspension

Si:

Organization A
      ↓
SUSPENDED

el acceso contextual a esa Organization debe ser rechazado:

User
 ↓
Membership
 ↓
Organization SUSPENDED
 ↓
DENY

La identidad global del usuario puede permanecer activa para otras Organizations.

32. Logout Current Session

Logout de la sesión actual:

POST /api/v1/auth/logout

Conceptualmente:

Authenticate Context
        ↓
Identify Session
        ↓
Revoke Session
        ↓
Revoke Refresh Credentials
        ↓
Audit
        ↓
Security Event

La ruta exacta será congelada en ESP-0001/10.

33. Logout All Sessions

Debe existir la capacidad conceptual de:

Logout Current Session
Logout All Sessions

Ejemplo:

User
 ├── Session A
 ├── Session B
 └── Session C

Logout All
 ├── A → REVOKED
 ├── B → REVOKED
 └── C → REVOKED

Debe ser una operación autorizada y auditable.

34. Session Management

El usuario debe poder visualizar sus sesiones cuando la UX/API lo permita:

Session
├── Device
├── Client
├── Created
├── Last Activity
├── Organization Context
└── Status

Esto permite detectar sesiones desconocidas.

35. Remote Session Revocation

Conceptualmente:

DELETE /api/v1/auth/sessions/{session_id}

El actor sólo puede revocar:

own sessions

o aquellas para las que tenga una Permission administrativa explícita.

Nunca:

arbitrary session
36. Concurrent Sessions

EVOXA debe soportar múltiples sesiones simultáneas:

User
 ├── Web
 ├── Mobile
 ├── Tablet
 └── Other Device

La arquitectura no debe asumir una única sesión por usuario.

Los límites concretos son una política de seguridad/configuración y no deben inventarse aquí.

37. Device Context

Una Session puede incluir:

device_id
device_type
platform
user_agent
client

Esto sirve para:

session management;
security telemetry;
user-facing session display;
incident investigation.

Pero:

Device Identity
    ≠
Authentication Proof

No debe utilizarse como único mecanismo de autenticación.

38. IP Address

La IP puede almacenarse como metadata de seguridad.

Debe considerarse potencialmente sensible.

No debe aparecer innecesariamente en:

API Responses
Logs
Events
Analytics
AI Context

El acceso debe estar gobernado por Security/Privacy.

39. Stateless Access Validation

El modelo preferido para requests normales es:

Request
  ↓
Bearer JWT
  ↓
Local Signature Validation
  ↓
Claims Validation
  ↓
Authorization Runtime

Esto permite escalar horizontalmente sin consultar Session en cada request.

40. Stateful Security Validation

Hay operaciones donde puede ser necesaria una validación de estado:

Critical Security Operation
Session Revocation
Refresh
Security Incident
Account State Change
MFA Change
Privilege Change

En estos casos puede ser necesario consultar el estado actual.

41. Important Security Rule

Un JWT válido no debe convertirse en una garantía de autorización permanente.

El sistema debe considerar:

JWT validity
+
Current security state
+
Tenant context
+
Membership
+
Authorization
+
Criticality
+
Freshness

según la operación.

42. Session Revocation vs JWT Expiration

Revocar una Session no necesariamente significa que el JWT desaparezca inmediatamente del cliente.

Un Access Token ya emitido puede continuar siendo criptográficamente válido hasta su expiración si la arquitectura no realiza introspección online.

Por eso:

Revoked Session

y:

Expired JWT

son mecanismos diferentes.

Las operaciones críticas deben respetar las garantías de freshness definidas por Authorization.

43. Refresh Security Boundary

El Refresh Token siempre debe validar:

Token
Session
User
Account State
Membership / Context
Expiration
Revocation
Family
Security Policy

No basta con:

token exists
44. Password Security Events

Un cambio de contraseña puede provocar medidas como:

Revoke Current Session
Revoke Other Sessions
Revoke Refresh Families
Require Reauthentication
Require MFA

dependiendo de si se trata de:

Normal Password Change

o:

Security Recovery / Compromise

La política debe ser explícita.

45. MFA Integration

MFA no pertenece al núcleo de esta ESP, pero Session debe almacenar suficiente contexto para saber:

authentication_method
mfa_status
mfa_evidence
assurance context

Una sesión creada después de MFA debe poder distinguirse de una sesión sin MFA cuando la Policy Engine lo necesite.

46. Step-Up Authentication

Una sesión existente puede requerir:

REAUTHENTICATE

o:

REQUIRE_MFA

antes de una operación sensible.

Flujo:

Active Session
      ↓
Sensitive Operation
      ↓
Authorization Runtime
      ↓
REQUIRE_MFA
      ↓
MFA Challenge
      ↓
Fresh Security Evidence
      ↓
Continue

La sesión original no se convierte automáticamente en una sesión de mayor assurance permanente.

47. Session and Authorization

Session Management no decide permisos.

La separación es:

Session
    ↓
Authentication Context

Authorization Runtime
    ↓
Authorization Decision

Por ejemplo:

Session = ACTIVE

no significa:

training-programs.update = ALLOW
48. Criticality Integration

Las operaciones relacionadas con Sessions/Token pueden tener diferentes niveles de Criticality.

Ejemplos:

C0
Session revoke-all
Refresh family revoke
Security reset
Critical credential operation

C1
Session security modification
Sensitive reauthentication

C2
Normal session read

C3
Low-risk session metadata

La Criticality efectiva sigue:

MAX(
    Base,
    Dynamic Risk,
    Context,
    Security State,
    Tenant,
    Delegation,
    MFA,
    Platform Minimum
)
49. Freshness Integration

La operación puede requerir:

F0
F1
F2
F3

según Criticality.

Especialmente:

Session Revocation
Refresh Family Revocation
Account Security State
MFA Security State
Membership Revocation

requieren garantías fuertes.

50. Cache

Redis puede utilizarse para optimización y estado temporal, pero:

Redis
    ≠
Source of Truth

El estado durable de Sessions/Refresh Tokens pertenece al sistema persistente definido en ESP-0001/11.

51. Cache Invalidation

Los cambios de seguridad relevantes deben propagarse mediante:

Security State Change
        ↓
PostgreSQL Transaction
        +
Transactional Outbox
        ↓
Event Platform
        ↓
Invalidation
        ↓
L1/L2 Cache

Esto evita que una entrada cacheada mantenga indefinidamente un estado de seguridad obsoleto.

52. Security Epoch

Los cambios críticos pueden incrementar un:

Security Epoch

Una cache entry asociada a un epoch anterior debe considerarse inválida aunque su TTL todavía no haya terminado.

Por tanto:

Security Revocation
      >
Security Epoch
      >
Policy Version
      >
Entity Version
      >
TTL

en la jerarquía de seguridad.

53. Refresh Token Revocation Priority

Una revocación de Refresh Token debe tener prioridad sobre:

TTL
Cache
Previous Authorization Result
Client State

No debe existir:

Revoked Refresh Token
      +
Valid TTL
      =
ALLOW
54. Session Repository

La interfaz conceptual puede incluir:

SessionRepository
├── create()
├── get_by_id()
├── list_by_user()
├── revoke()
├── revoke_all_for_user()
├── revoke_for_organization()
└── update_activity()

Estos son límites de aplicación/dominio, no contratos físicos definitivos.

55. Refresh Token Repository

Conceptualmente:

RefreshTokenRepository
├── create()
├── get()
├── rotate()
├── revoke()
├── detect_reuse()
└── revoke_family()

rotate() debe ser una operación transaccional y concurrency-safe.

56. Transactional Rotation

El modelo debe garantizar:

Validate A
   +
Mark A USED
   +
Create B
   +
Persist Security State
   +
Create Outbox Event

dentro de una operación transaccional cuando corresponda.

No debe existir un estado donde:

A = USED

pero:

B

no exista debido a una transacción parcialmente aplicada.

57. Event Integration

Eventos conceptuales:

SessionCreated
SessionRevoked
SessionExpired

RefreshTokenIssued
RefreshTokenRotated
RefreshTokenRevoked
RefreshTokenReuseDetected

LogoutCompleted
GlobalLogoutCompleted

Los eventos deben:

ser versionados;
ser inmutables;
ser idempotentemente consumibles;
no contener secretos;
respetar tenant context;
utilizar Transactional Outbox.
58. Security Event

RefreshTokenReuseDetected es particularmente importante.

Conceptualmente:

RefreshTokenReuseDetected
├── event_id
├── session_id
├── token_family_id
├── actor/context
├── tenant_id
├── timestamp
├── correlation_id
└── security metadata

Nunca:

raw_refresh_token
59. Audit

Las acciones sensibles deben quedar auditadas:

Session Created
Session Revoked
Global Logout
Refresh Rotation
Refresh Reuse
Token Revocation
Security Session Reset

El audit debe registrar:

Who
What
When
Where/context
Why
Result
Correlation

sin secretos.

60. API Boundary

Las APIs conceptuales incluyen:

POST   /api/v1/auth/logout

GET    /api/v1/auth/sessions

GET    /api/v1/auth/sessions/{session_id}

DELETE /api/v1/auth/sessions/{session_id}

POST   /api/v1/auth/sessions/revoke-all

POST   /api/v1/auth/refresh

Pero las rutas definitivas pertenecen a:

ESP-0001/10 — Identity API Contracts

No debemos congelar aquí detalles que corresponden a API Contracts.

61. Refresh Token Client Storage

Consolidando ADR-009:

Web

El Refresh Token debe utilizar:

HttpOnly
Secure
Cookie

y no debe almacenarse en:

localStorage
Mobile

Debe almacenarse mediante:

OS Secure Storage

El SDK de cliente debe abstraer el mecanismo de almacenamiento.

62. Access Token Client Storage

El Access Token web debe mantenerse preferentemente en memoria.

No se debe utilizar el mismo mecanismo de almacenamiento del Refresh Token.

Modelo:

WEB

Access Token
    ↓
Memory

Refresh Token
    ↓
HttpOnly Secure Cookie
63. Mobile

Conceptualmente:

Mobile
 │
 ├── Access Token → protected application memory
 │
 └── Refresh Token → OS Secure Storage

La implementación concreta dependerá del SDK móvil.

64. API Keys & Service Accounts

API Keys y Service Accounts pertenecen a capacidades de Identity relacionadas pero no deben confundirse con Sessions de usuarios.

Separación:

User Session
    ≠
API Token
    ≠
Service Account

Su modelo está gobernado por:

ADR-037
ADR-038
65. OAuth2.1 / OIDC

Las sesiones pueden originarse mediante:

Local Authentication
OAuth2.1
OIDC
Federation

pero una vez autenticado el actor, EVOXA establece su propio contexto de Identity.

Los tokens externos no deben convertirse automáticamente en Access Tokens internos.

66. Federation

Para identidad federada:

External Identity
      ↓
Authentication
      ↓
Local EVOXA Identity
      ↓
Membership
      ↓
Session
      ↓
EVOXA Access Token

La autorización sigue siendo responsabilidad de EVOXA.

67. Rate Limiting

Deben existir controles para:

Login
Refresh
Logout
Session Management
Token Reuse
Security Operations

especialmente:

Refresh Endpoint

para impedir abuso automatizado.

Los detalles de rate limiting corresponden a ADR-032.

68. Failure Model

Ante fallos de componentes de seguridad:

Authorization Failure
Session State Uncertainty
Token Validation Failure
Security State Uncertainty

no se debe convertir automáticamente la incertidumbre en:

ALLOW

El principio es:

Unknown
   ↓
Fail Closed

especialmente para operaciones críticas.

69. AI Agents

Los AI Agents pueden tener identidad propia.

Pero:

AI Agent
   ↓
Session / Credential
   ↓
Authorization

debe seguir exactamente las mismas garantías de seguridad.

Un AI Agent no puede:

bypass session security
extend token lifetime
reuse revoked tokens
disable revocation
ignore MFA
reduce criticality
manipulate cache
70. Security Invariants

Este ESP debe preservar como mínimo:

INV-SESSION-001
Revoked Session Cannot Become Active

INV-SESSION-002
Expired Session Cannot Become Active

INV-TOKEN-001
Used Refresh Token Cannot Be Reused

INV-TOKEN-002
Revoked Refresh Token Cannot Refresh

INV-TOKEN-003
Refresh Rotation Is Atomic

INV-TOKEN-004
Refresh Reuse Cannot Produce Successful Authorization

INV-TOKEN-005
Refresh Token Family Revocation Is Enforced

INV-SESSION-003
Cross-Tenant Session Context Cannot Be Used

INV-SESSION-004
Security Revocation Overrides TTL

INV-SESSION-005
Cache Cannot Override Authoritative Security State

INV-SESSION-006
Unknown Security State Cannot Produce Unsafe ALLOW
71. Testing
Session
Create
Activate
Expire
Revoke
Revoke all
Concurrent sessions
Remote revoke
Access Token
Valid JWT
Expired JWT
Invalid signature
Wrong issuer
Wrong audience
Missing claims
Invalid algorithm
Unknown kid
Refresh
Valid refresh
Expired refresh
Revoked refresh
Used refresh
Rotation
Concurrent refresh
Family revocation
Reuse detection
Security
Account suspension
Membership revocation
MFA revocation
Password security event
Tenant suspension
Cache
Fresh
Stale
Invalidated
Out-of-order
Duplicate event
Event delay
Redis failure
72. Performance

El Access Token JWT debe poder validarse localmente para evitar dependencia de red en cada request.

Objetivos generales:

API P95 < 300ms
Authentication P95 < 500ms
Authorization < 50ms objective

La implementación debe validar que Session/Token operations no introduzcan una dependencia central innecesaria en el request path normal.

73. Traceability
Área	ADR / ESP
Identity Domain	ESP-0001/01
Authentication	ESP-0001/03
Roles/Permissions	ESP-0001/05
Session	ADR-003
Access JWT	ADR-002
Refresh Tokens	ADR-003 / ADR-025
Token Cryptography	ADR-008 / ADR-025
Token Storage	ADR-009
MFA	ADR-023 / ADR-024
Account Lifecycle	ADR-033
Password Security	ADR-034
Recovery	ADR-035
OAuth/OIDC	ADR-039
Federation	ADR-040
Provisioning	ADR-041
Runtime Security	ADR-027–032
Cache/Freshness	ADR-050–059
Test Contracts	ADR-062–067
Module Boundaries	ADR-068–069
API	ESP-0001/10
Persistence	ESP-0001/11
Events	ESP-0001/12
Testing	ESP-0001/13
74. Qué eliminamos del ESP original

Aquí hay una corrección importante.

El documento original todavía decía que varias decisiones debían resolverse mediante futuros ADR, por ejemplo:

ADR-IDENTITY-034 — Token & Session Lifetimes
ADR-IDENTITY-035 — Concurrent Session Policy
ADR-IDENTITY-036 — Session Revocation on Account Suspension

Además, indicaba que Rotation y Token Family eran todavía recomendaciones que debían formalizarse.

Eso ya no debe permanecer así.

Las decisiones fundamentales fueron posteriormente consolidadas en:

ADR-002
ADR-003
ADR-008
ADR-009
ADR-025
ADR-027
ADR-032
ADR-033
ADR-034
ADR-035
ADR-037
ADR-038
ADR-039
ADR-040
ADR-043
ADR-049
ADR-050–059
ADR-067

Por lo tanto, la ESP no debe volver a presentar esas decisiones como si estuvieran abiertas.

75. Decisiones que siguen abiertas

Después de consolidar los ADR, las cuestiones que realmente quedan para las siguientes especificaciones son principalmente de implementación, no de arquitectura fundamental:

Persistence
schema físico;
índices;
constraints;
estrategia de hashing/HMAC;
transacciones;
migrations.

→ ESP-0001/11

API
request/response schemas;
status codes;
cookie contract;
refresh endpoint;
session endpoints.

→ ESP-0001/10

Events
nombres definitivos;
payloads;
versiones;
partition keys.

→ ESP-0001/12

Configuration
valores concretos de TTL;
session lifetime;
idle timeout;
límites de sesiones;
grace/idempotency window.

Estos valores deben establecerse como configuración gobernada, no inventarse dentro de esta ESP.

76. Implementation Readiness
Área	Estado
Session model	🟢
Session lifecycle	🟢
Access JWT	🟢
Asymmetric signing	🟢
JWKS / kid	🟢
Refresh Token	🟢
Refresh Rotation	🟢
Token Family	🟢
Reuse Detection	🟢
Atomic Rotation	🟢
Session Revocation	🟢
Global Logout	🟢
Concurrent Sessions	🟢
Tenant Context	🟢
MFA integration	🟢
Security State integration	🟢
Authorization integration	🟢
Criticality	🟢
Freshness	🟢
Cache	🟢
Audit	🟢
Security Events	🟢
API implementation	⏳ ESP-10
Persistence implementation	⏳ ESP-11
Event implementation	⏳ ESP-12
Test implementation	⏳ ESP-13
Runtime implementation	⏳ ESP-15
77. Resultado final

Con este /06, el modelo de credenciales de EVOXA queda conceptualmente así:

                         USER
                           │
                           ▼
                    AUTHENTICATION
                           │
                           ▼
                        SESSION
                           │
              ┌────────────┴────────────┐
              │                         │
              ▼                         ▼
        ACCESS TOKEN              REFRESH TOKEN
              │                         │
              │                         ▼
              │                   TOKEN FAMILY
              │                         │
              │                    ROTATION
              │                         │
              │                  REUSE DETECTION
              │                         │
              │                    REVOCATION
              │                         │
              └────────────┬────────────┘
                           ▼
                    SECURITY CONTEXT
                           │
                           ▼
                 AUTHORIZATION RUNTIME
                           │
             ┌─────────────┼─────────────┐
             ▼             ▼             ▼
           TENANT       CRITICALITY    FRESHNESS
             │             │             │
             └─────────────┼─────────────┘
                           ▼
                 AUTHORIZATION DECISION
Evaluación

ESP-0001/06 original: 🟡 Draft con decisiones pendientes/desactualizadas

ESP-0001/06 v1.1.0 consolidado: 🟢 Arquitectónicamente alineado con ADR-001 → ADR-069

Y hay algo especialmente importante: ya no necesitamos crear ADR-070 para Sessions & Tokens. Los ADR existentes cubren las decisiones arquitectónicas principales; ahora corresponde continuar consolidando las ESP y después pasar a implementación.

El siguiente documento lógico es ESP-0001/07 — Identity Multi-Factor Authentication (MFA), donde debemos consolidar especialmente ADR-023 y 024 junto con el impacto de ADR-045–059 y 067 sobre MFA, step-up, assurance, freshness, recovery y trusted devices.
