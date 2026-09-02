ADR-IDENTITY-003 — Refresh Token Security & Session Management

Proyecto: EVOXA Platform
Dominio: Identity
ADR: ADR-IDENTITY-003
Título: Refresh Token Security & Session Management
Estado: Proposed
Versión: 1.0.0
Relacionado con: ESP-0001 — Identity
Relacionado con: ADR-IDENTITY-001, ADR-IDENTITY-002
Tipo: Architecture Decision Record

1. Contexto

EVOXA Identity utiliza un modelo híbrido de autenticación:

Access Token
    ↓
JWT / stateless

Refresh Token
    ↓
Stateful

Session
    ↓
Stateful

El objetivo es combinar la eficiencia de JWT para las solicitudes normales con el control de seguridad de sesiones y refresh tokens persistidos.

ESP-0001 establece que las sesiones y refresh tokens deben permitir:

Revocación.
Rotation.
Detección de reuse.
Logout.
Logout all.
Control por dispositivo.
Protección ante compromiso.
Auditabilidad.
Observabilidad.

Además, el Blueprint define que los servicios deben ser seguros por defecto, utilizar least privilege, Zero Trust, auditabilidad y fail-closed.

2. Problema

Necesitamos determinar exactamente cómo funcionará el ciclo de vida de:

Session
Refresh Token
Refresh Token Family

y qué ocurrirá ante:

Refresh normal.
Refresh concurrente.
Refresh token expirado.
Refresh token revocado.
Reutilización de un token.
Logout.
Logout all.
Suspensión de usuario.
Eliminación de membership.
Suspensión de organización.
Robo de refresh token.
Revocación administrativa.
Rotación de claves.
Recuperación ante incidentes.
3. Objetivos

La solución deberá:

Evitar reutilización de refresh tokens.
Detectar replay.
Permitir revocación individual.
Permitir revocación por sesión.
Permitir revocación global.
Soportar múltiples dispositivos.
Evitar race conditions.
Mantener aislamiento de tenants.
Permitir auditoría.
Permitir investigación de incidentes.
Mantener escalabilidad horizontal.
No depender de una instancia concreta del servicio.
4. Decisión principal

Se utilizará:

Stateful Sessions + Stateful Refresh Tokens + Refresh Token Rotation + Token Families + Reuse Detection.

Arquitectura:

                         USER
                           │
                           ▼
                       SESSION
                           │
                           ▼
                    TOKEN FAMILY
                           │
              ┌────────────┼────────────┐
              ▼            ▼            ▼
           Token A      Token B      Token C
              │            │            │
              └──── rotation ──────────┘

Cada nuevo refresh reemplaza al anterior.

5. Session como Security Boundary

La Session será la principal unidad stateful de control.

Session
├── User
├── Organization
├── Device
├── Authentication context
├── Refresh token family
├── Created
├── Last activity
├── Expiration
└── Revocation

Esto permitirá revocar un dispositivo sin afectar necesariamente a los demás.

6. Session Lifecycle

Estados:

CREATED
   ↓
ACTIVE
   ↓
EXPIRED
   ↓
REVOKED
CREATED

La sesión ha sido creada pero todavía está en proceso de activación.

ACTIVE

La sesión puede utilizarse para mantener autenticación.

EXPIRED

Ha superado su tiempo de vida.

REVOKED

Ha sido invalidada antes de su expiración natural.

7. Refresh Token Lifecycle

Cada refresh token seguirá:

ISSUED
   ↓
ACTIVE
   ↓
USED
   ↓
ROTATED

o:

ISSUED
   ↓
REVOKED

o:

ISSUED
   ↓
EXPIRED

Un refresh token utilizado exitosamente no podrá volver a utilizarse.

8. Token Family

Los refresh tokens estarán agrupados por:

family_id

Ejemplo:

Family F1

Token A
   ↓
Token B
   ↓
Token C
   ↓
Token D

Esto permitirá identificar una posible reutilización dentro de una cadena.

9. Refresh Normal

El flujo será:

Client
  │
  │ Refresh Token A
  ▼
Identity
  │
  ├── Validate token
  ├── Validate session
  ├── Validate user
  ├── Validate membership
  ├── Validate organization
  ├── Validate security policy
  │
  ▼
Rotate
  │
  ├── Token A → USED
  └── Token B → ACTIVE
  │
  ▼
Access Token B
+
Refresh Token B
10. Atomic Rotation

La operación de rotation deberá ser atómica.

Conceptualmente:

BEGIN TRANSACTION

Validate Token A

Mark Token A = USED

Create Token B

Commit

No deberá existir una ventana en la que dos procesos puedan utilizar el mismo refresh token y ambos obtener una nueva cadena válida.

11. Concurrency Control

Caso:

Request A ── Token A
Request B ── Token A

El sistema deberá garantizar que solamente una solicitud pueda consumir exitosamente el token.

Resultado esperado:

Request A → SUCCESS
Request B → REUSE / INVALID

o el resultado equivalente según cuál gane la operación atómica.

12. Reuse Detection

Si se presenta un token que ya fue utilizado:

Token A
   ↓
USED
   ↓
Token A presented again
   ↓
REUSE DETECTED

Identity deberá:

Rechazar el refresh.
Registrar security event.
Crear audit event.
Marcar el incidente.
Revocar la familia según la política definida.
13. Family Revocation
Decisión

Ante un refresh token reuse confirmado, la respuesta por defecto será:

Revocar toda la refresh token family asociada.

Ejemplo:

Family F1

Token A → USED
Token B → USED
Token C → ACTIVE
Token A → reused
             │
             ▼
       REUSE DETECTED
             │
             ▼
      Revoke Family F1

Esto protege contra un escenario en el que una copia antigua del token esté siendo utilizada por un atacante.

14. Session Revocation

Cuando la familia sea revocada por un incidente de seguridad:

Token Reuse
    ↓
Family Revoked
    ↓
Session Revoked

Esto convierte la sesión en la unidad de contención.

15. User Security Reset

Para incidentes de mayor severidad:

Security Incident
       ↓
User Security Reset
       ↓
Revoke all sessions
       ↓
Revoke all refresh families
       ↓
Require reauthentication

Esto deberá ser una operación explícita y auditada.

16. Logout

El logout normal deberá revocar la sesión actual:

POST /auth/logout
        ↓
Current Session
        ↓
REVOKED
        ↓
Refresh Family
        ↓
REVOKED

Esto no requiere necesariamente invalidar inmediatamente todos los access tokens JWT ya emitidos.

17. Logout All
POST /auth/logout-all
        ↓
User
        ↓
All Sessions
        ↓
REVOKED

También se invalidarán las refresh token families correspondientes.

18. Access Token After Logout

Debido al modelo stateless definido en ADR-IDENTITY-002:

Logout
 ↓
Session revoked
 ↓
Refresh revoked

pero un access JWT previamente emitido podría permanecer técnicamente válido hasta su expiración.

Decisión

Esto se acepta como comportamiento normal del modelo.

Para operaciones altamente sensibles podrá utilizarse una estrategia adicional de revocación inmediata.

19. Refresh After Logout

Si el cliente intenta:

Refresh
 ↓
Revoked Session

resultado:

DENY

El sistema no deberá emitir nuevos tokens.

20. Refresh After User Suspension

Si:

User ACTIVE
       ↓
SUSPENDED

se deberán revocar las sesiones activas del usuario.

Además:

Refresh attempt
       ↓
User SUSPENDED
       ↓
DENY
21. Refresh After Organization Suspension

Si:

Organization ACTIVE
       ↓
SUSPENDED

las sesiones relacionadas con esa organización deberán quedar impedidas de realizar nuevos refresh.

Las sesiones podrán ser revocadas según la política de tenant.

22. Refresh After Membership Removal

Si:

Membership ACTIVE
       ↓
REMOVED

un refresh posterior deberá volver a evaluar la membership.

Resultado:

Membership REMOVED
       ↓
DENY

Esto evita que una sesión antigua mantenga acceso indefinidamente.

23. Refresh Validation Order

Cada refresh deberá validar, conceptualmente:

1. Token exists
       ↓
2. Token status
       ↓
3. Token expiration
       ↓
4. Session status
       ↓
5. User status
       ↓
6. Organization status
       ↓
7. Membership status
       ↓
8. Security policies
       ↓
9. Token rotation

La operación deberá fallar inmediatamente cuando una condición crítica no sea válida.

24. Fail Closed

Si Identity no puede determinar de manera confiable si una sesión o token es válido:

Unknown
 ↓
DENY

Nunca:

Unknown
 ↓
ALLOW
25. Refresh Token Storage

La base de datos almacenará:

token_hash

y metadata como:

id
session_id
family_id
status
created_at
used_at
expires_at
revoked_at

El token secreto original no deberá ser recuperable desde la base de datos.

26. Refresh Token Status

Estados propuestos:

ACTIVE
USED
EXPIRED
REVOKED
REUSE_DETECTED

La transición deberá estar controlada.

27. Session Revocation Reasons

La sesión deberá poder registrar un motivo de revocación.

Ejemplos:

USER_LOGOUT
LOGOUT_ALL
USER_SUSPENDED
ORG_SUSPENDED
MEMBERSHIP_REMOVED
TOKEN_REUSE
SECURITY_INCIDENT
ADMIN_REVOCATION
EXPIRED

Esto facilita auditoría y análisis operacional.

28. Device Sessions

Cada sesión podrá asociarse a información de dispositivo:

device_id
device_type
platform
user_agent
ip

La información deberá estar limitada a lo necesario y cumplir políticas de privacidad.

29. Multiple Devices

Se permitirá:

User
│
├── Desktop
├── Mobile
├── Tablet
└── Browser

Cada sesión podrá ser revocada individualmente.

30. Concurrent Session Limit
Decisión

No se establecerá inicialmente un límite fijo obligatorio.

Se permitirá configurar:

MAX_SESSIONS_PER_USER

si una política de seguridad futura lo requiere.

31. Session Lifetime

La sesión deberá tener:

created_at
expires_at

y potencialmente:

last_activity_at
Decisión

La sesión tendrá un maximum lifetime configurable.

El valor concreto deberá definirse posteriormente mediante política de seguridad.

32. Sliding Session
Decisión

No se implementará inicialmente una extensión ilimitada de sesión.

La sesión deberá respetar un límite máximo absoluto.

Si posteriormente se habilita sliding expiration:

Activity
 ↓
Extend
 ↓
Maximum absolute lifetime

deberá mantenerse siempre un límite máximo.

33. Refresh Token Lifetime

El refresh token tendrá una duración configurable.

Debe cumplir:

Access Token TTL
       <
Refresh Token TTL
       ≤
Session Maximum Lifetime

Los valores exactos quedan sujetos a la política de seguridad final.

34. Refresh Rotation Frequency

Cada refresh exitoso deberá producir:

New Access Token
+
New Refresh Token

No se reutilizará indefinidamente el mismo refresh token.

35. Token Reuse Security Event

Deberá generarse:

TokenReuseDetected

con información segura:

event_id
timestamp
tenant_id
user_id
session_id
family_id
token_id
correlation_id
trace_id

Nunca:

refresh_token

en texto plano.

36. Audit

Cada operación crítica deberá producir audit:

SessionCreated
SessionRevoked
RefreshTokenRotated
RefreshTokenRevoked
TokenReuseDetected
Logout
LogoutAll

Esto es consistente con los requisitos de Audit & Compliance de ESP-0001.

37. Observability

Deberán existir métricas como:

refresh_success_total
refresh_failure_total
refresh_reuse_total
session_created_total
session_revoked_total
logout_total
logout_all_total

Y métricas de latencia:

refresh_latency
session_operation_latency
38. Alerting

Una tasa anormal de:

TokenReuseDetected

deberá generar una alerta.

También:

Authentication failures.
Refresh failures.
Suspicious session creation.
Massive logout/revocation.
Cross-tenant attempts.
39. Database Concurrency

La implementación deberá garantizar atomicidad mediante mecanismos adecuados como:

Database transaction.
Row-level locking cuando corresponda.
Unique constraints.
Compare-and-set/update conditions.
Isolation apropiada.

La tecnología concreta será responsabilidad de Infrastructure.

40. Idempotency

El refresh no se tratará como una operación idempotente simple, porque cada refresh genera una nueva credencial.

Por ello:

Same Refresh Token
+
Repeated Request

no deberá producir dos refreshes exitosos.

El primer consumo gana; el siguiente debe detectarse como reuse o invalidación.

41. API

Los endpoints principales serán:

POST /api/v1/identity/auth/login
POST /api/v1/identity/auth/refresh
POST /api/v1/identity/auth/logout
POST /api/v1/identity/auth/logout-all

Y para administración de sesiones:

GET  /api/v1/identity/sessions
POST /api/v1/identity/sessions/{id}/revoke
POST /api/v1/identity/sessions/revoke-all

El contrato final deberá mantenerse alineado con ESP-0001 / 10.

42. Security Response

Cuando un refresh falle, la respuesta no deberá revelar información que permita al atacante determinar:

Si el usuario existe.
Si el token pertenece a otro usuario.
Detalles internos de la session.
Estado interno de la token family.
Información de seguridad no necesaria.
43. Incident Response

Ante reuse confirmado:

TokenReuseDetected
       ↓
Reject Request
       ↓
Revoke Token Family
       ↓
Revoke Session
       ↓
Audit
       ↓
Security Alert

Para incidentes de mayor severidad:

User Security Reset
       ↓
Revoke All Sessions
44. Administrative Revocation

Un administrador autorizado podrá revocar una sesión.

Admin
 ↓
Authorize
 ↓
Revoke Session
 ↓
Audit
 ↓
Event

Esta operación deberá estar protegida por RBAC + ABAC + Security Policy.

45. Tenant Isolation

Todas las operaciones de sesiones deberán respetar:

tenant_id

Un administrador de:

Organization A

no podrá revocar una sesión perteneciente a:

Organization B

salvo que posea una autorización explícita de plataforma.

46. Security Hierarchy

La autorización deberá seguir:

Authentication
 ↓
Tenant Context
 ↓
RBAC
 ↓
ABAC
 ↓
Ownership
 ↓
Security Policy
 ↓
Decision

Esto mantiene el modelo de autorización establecido para Identity.

47. Backup Considerations

Las tablas de:

sessions
refresh_tokens

deberán formar parte de la estrategia de backup según corresponda.

Sin embargo, la recuperación de sesiones deberá considerar las implicaciones de seguridad.

No se debe asumir que restaurar un backup antiguo automáticamente hace válidas todas las sesiones.

48. Disaster Recovery

En un escenario DR:

Production Failure
      ↓
Restore
      ↓
Validate Sessions
      ↓
Validate Refresh Tokens
      ↓
Validate Security State
      ↓
Resume

Si existe incertidumbre sobre el estado de seguridad de refresh tokens, podrá ser necesario forzar reauthentication.

49. Key Rotation Interaction

La rotación de signing keys de JWT no deberá romper innecesariamente las sesiones.

Key K1
   ↓
Key K2

Los tokens existentes deberán manejarse según la política de key rotation.

Este tema queda relacionado con:

ADR-IDENTITY-008 — Identity Cryptographic Key Management

50. Testing Requirements

Deberán existir pruebas para:

Normal
Login.
Session creation.
Refresh.
Rotation.
Logout.
Security
Reuse.
Replay.
Theft.
Revocation.
User suspension.
Organization suspension.
Membership removal.
Concurrency
Two refreshes
same token
same time
Recovery
Database restore.
Session restore.
Refresh family integrity.
Tenant
Tenant A → Session A
Tenant B → Session B

y verificar que no exista acceso cruzado.

51. Security Test Example
Given:
    Refresh Token A = USED

When:
    Client sends Refresh Token A

Then:
    Request DENIED

And:
    TokenReuseDetected emitted

And:
    AuditEvent created

And:
    Family F1 revoked

And:
    Session S1 revoked
52. Operational Test Example
Given:
    User has 3 active sessions

When:
    User executes logout-all

Then:
    Sessions = 0 active

And:
    Refresh token families revoked

And:
    Audit event created
53. Decision Summary
Área	Decisión
Session	Stateful
Refresh Token	Stateful
Token Rotation	✅ Required
Token Family	✅ Required
Reuse Detection	✅ Required
Family Revocation	✅ On confirmed reuse
Session Revocation	✅ On confirmed reuse
Logout	Current session
Logout All	All sessions
Multi-device	✅ Supported
Concurrent sessions	Allowed initially
Maximum sessions	Configurable
Session maximum lifetime	Configurable
Sliding session	Limited / not unlimited
Access token	JWT/stateless
Refresh storage	Secure hash
Refresh replay	Denied
Tenant isolation	Mandatory
Audit	Mandatory
Security Events	Mandatory
Fail Closed	Mandatory
54. Decisiones que quedan cerradas

Este ADR permite considerar como decidido:

✅ Stateful Session
✅ Stateful Refresh Token
✅ Refresh Rotation
✅ Refresh Token Family
✅ Reuse Detection
✅ Family Revocation
✅ Session Revocation on confirmed reuse
✅ Multi-device sessions
✅ Individual session revocation
✅ Logout All
✅ Atomic refresh
✅ Concurrency protection
✅ Secure token hashing
55. Decisiones todavía abiertas

No debemos cerrar artificialmente:

🟡 Access Token TTL
🟡 Refresh Token TTL
🟡 Absolute Session Lifetime
🟡 Maximum concurrent sessions
🟡 Sliding session exact behavior
🟡 Immediate access-token revocation strategy
🟡 JWT signing algorithm
🟡 Key rotation period
🟡 Browser token storage

Estas decisiones deben quedar en sus ADR correspondientes.

56. Consecuencias
Positivas

La arquitectura proporciona:

Protección fuerte contra refresh-token replay.
Revocación por dispositivo.
Revocación global.
Detección de robo.
Multi-device.
Auditabilidad.
Control de sesiones.
Escalabilidad.
Compatibilidad con JWT stateless.
Negativas

Introduce:

Mayor complejidad de persistencia.
Gestión de token families.
Concurrency control.
Necesidad de transacciones.
Mayor volumen de audit/events.
Necesidad de monitorear reuse.

La complejidad es aceptable porque Identity es un componente de seguridad crítico.

57. Relación con los ADR anteriores
ADR-IDENTITY-001
Architecture & Technology
        │
        ▼
ADR-IDENTITY-002
JWT & Token Lifecycle
        │
        ▼
ADR-IDENTITY-003
Refresh Token Security
& Session Management

El modelo completo queda:

             Identity
                │
        ┌───────┴────────┐
        ▼                ▼
 Access Token       Session
   JWT                │
    │                 ▼
 Stateless        Token Family
                     │
             ┌───────┼───────┐
             ▼       ▼       ▼
           A → B → C → D
             │
             ▼
        Reuse Detection
             │
             ▼
       Family Revocation
             │
             ▼
        Session Revoked
58. Próximos ADR

Después de este ADR, el orden lógico es:

ADR-IDENTITY-004 — MFA Architecture

Definiremos:

TOTP.
WebAuthn/Passkeys.
OTP.
Recovery codes.
Trusted devices.
Step-up authentication.
MFA enrollment.
MFA enforcement.
MFA recovery.

Después:

ADR-IDENTITY-005 — Authorization & Policy Engine

y posteriormente:

006 Persistence
007 Events & Outbox
008 Cryptographic Keys
009 Client Token Storage
010 Testing
011 Deployment
59. Estado

Status: Proposed

Este ADR ya deja cerrada la arquitectura fundamental de Refresh Tokens + Sessions, pero no debemos marcarlo como Accepted hasta que las decisiones relacionadas de seguridad —especialmente TTL, cryptographic keys y client storage— estén formalizadas.

Con esto, ESP-0001 ya tiene una cadena de decisiones mucho más sólida para pasar posteriormente a Implementation Stories, sin tener que tomar decisiones críticas de autenticación directamente mientras escribimos el código.
