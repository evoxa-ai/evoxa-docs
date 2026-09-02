IS-IDENTITY-009 — Refresh Token Lifecycle

Estado: Draft
Prioridad: Critical
Dominio: Identity
Capability: Refresh Token Management
Dependencias: IS-IDENTITY-001, 002, 003, 004, 005, 006, 007, 008
Siguiente historia: IS-IDENTITY-010 — Multi-Factor Authentication

1. Propósito

Definir el ciclo de vida completo de los Refresh Tokens utilizados por EVOXA para mantener una sesión autenticada sin requerir que el usuario vuelva a introducir sus credenciales cuando expire el Access Token.

El diseño debe proporcionar:

Refresh Tokens persistentes y administrables.
Rotación segura.
Token Families.
Detección de reutilización.
Revocación.
Expiración.
Asociación con una Session.
Protección frente a robo de tokens.
Control de concurrencia.
Auditoría.
Eventos de seguridad.

El Refresh Token no debe considerarse simplemente otro JWT de larga duración. Su ciclo de vida debe ser stateful y controlado por Identity.

2. Business Goal

Como plataforma EVOXA, necesitamos renovar credenciales de acceso de corta duración sin obligar al usuario a autenticarse nuevamente constantemente, manteniendo al mismo tiempo la capacidad de detectar y contener el robo o reutilización de Refresh Tokens.

El sistema debe permitir:

Access Token expira
        ↓
Refresh Token válido
        ↓
Validación
        ↓
Rotación
        ↓
Nuevo Access Token
        ↓
Nuevo Refresh Token

La arquitectura debe permitir detectar situaciones como:

Token legítimo
      ↓
Rotado
      ↓
Token anterior vuelve a utilizarse
      ↓
REUSE DETECTION
      ↓
Security Response
      ↓
Revocar Token Family
      ↓
Revocar Session
3. Persona
Primary Persona

Authenticated User

Secondary Personas
Coach
Nutritionist
Physiotherapist
Team Manager
Admin
Security Administrator
System Service
4. User Journey
4.1 Login
User
 ↓
Authentication
 ↓
Session Created
 ↓
Access Token
 ↓
Refresh Token

El login inicial es responsabilidad de IS-IDENTITY-005.

4.2 Refresh normal
Access Token
     ↓
   EXPIRED
     ↓
Refresh Request
     ↓
Validate Refresh Token
     ↓
Validate Session
     ↓
Validate User
     ↓
Validate Organization/Membership
     ↓
Validate Security Policy
     ↓
Rotate Refresh Token
     ↓
Issue new Access Token
     ↓
Issue new Refresh Token
4.3 Refresh Token reutilizado
Refresh Token A
       ↓
     USED
       ↓
Rotated → Token B
       ↓
Token A utilizado nuevamente
       ↓
REUSE DETECTION
       ↓
Revoke Token Family
       ↓
Revoke Session
       ↓
Security Event
       ↓
Audit Event

Este comportamiento es una pieza central de seguridad del diseño.

5. Feature Pack

Esta Implementation Story cubre:

Token Lifecycle
Creación.
Activación.
Rotación.
Expiración.
Revocación.
Reutilización.
Token Security
Hash/storage seguro.
Token Families.
Token lineage.
Reuse Detection.
Revocación de family.
Asociación con Session.
Session Integration
Session validation.
Session expiration.
Session revocation.
User suspension.
Organization suspension.
Membership changes.
Operational
Concurrencia.
Idempotencia.
Auditoría.
Eventos.
Métricas.
Logging.
Tracing.
6. Scope
In Scope
Refresh Token entity.
Refresh Token ID.
Token Family.
Token rotation.
Token lineage.
Token expiration.
Token revocation.
Reuse detection.
Session association.
Persistence.
Concurrency control.
Security events.
Audit.
Observability.
Authentication integration.
Session integration.
Out of Scope

No se implementa aquí:

Password authentication.
MFA.
Access Token authorization.
Role management.
Permission management.
Security Policy Engine completo.
OAuth/OIDC provider completo.
API Token management.
7. Arquitectura conceptual

La arquitectura definida por los ADR de Identity establece:

Session
   │
   │ 1:N
   ↓
Refresh Token Family
   │
   ├── Refresh Token 1
   ├── Refresh Token 2
   ├── Refresh Token 3
   └── Refresh Token N

Cada rotación genera un nuevo Refresh Token.

RT-001
  ↓ rotate
RT-002
  ↓ rotate
RT-003
  ↓ rotate
RT-004

El token anterior queda invalidado.

8. RefreshToken Entity

Entidad conceptual:

RefreshToken
├── id
├── session_id
├── family_id
├── parent_token_id
├── status
├── token_hash
├── issued_at
├── expires_at
├── used_at
├── revoked_at
├── revoke_reason
├── created_at
└── metadata

Los nombres definitivos de persistencia se establecerán en IS-IDENTITY-015.

9. RefreshTokenId

Value Object:

RefreshTokenId

Debe:

Identificar el token internamente.
Validar formato.
Evitar que el dominio dependa de strings arbitrarios.
10. Token Family

Una Token Family representa la cadena de Refresh Tokens derivados de una misma sesión/flujo de autenticación.

Ejemplo:

Family F1

RT1
 │
 └── RT2
       │
       └── RT3
             │
             └── RT4

Todos pertenecen a:

family_id = F1

La family permite responder ante un compromiso de token.

11. Token Lineage

Cada token debe poder mantener una relación con el token anterior.

Conceptualmente:

RT1
 ↓ parent
RT2
 ↓ parent
RT3
 ↓ parent
RT4

Esto permite reconstruir la cadena de rotación y detectar comportamientos anómalos.

12. Refresh Token Lifecycle

Estados conceptuales:

ACTIVE
USED
EXPIRED
REVOKED

Puede existir además un estado interno de procesamiento dependiendo de la estrategia de concurrencia, pero no debe introducirse sin justificación arquitectónica.

13. Estados
ACTIVE

El Refresh Token puede utilizarse para realizar una operación de refresh si todas las condiciones de seguridad son válidas.

USED

El token ya fue utilizado exitosamente y fue sustituido por otro token.

Un token USED no puede utilizarse nuevamente.

EXPIRED

El token alcanzó su fecha de expiración.

No puede utilizarse para obtener nuevos tokens.

REVOKED

El token fue invalidado explícitamente.

Puede ocurrir por:

Logout.
Session revoke.
User suspension.
Security event.
Token reuse.
Organization suspension.
Policy decision.
14. Rotation

La rotación es obligatoria en el modelo.

Flujo:

Request
   │
   ↓
Validate RT1
   │
   ↓
Mark RT1 USED
   │
   ↓
Create RT2
   │
   ↓
Issue Access Token
   │
   ↓
Return RT2

El proceso debe ser atómico.

No debe existir una ventana en la que dos solicitudes concurrentes puedan utilizar legítimamente el mismo Refresh Token.

15. Atomic Rotation

Una rotación debe garantizar:

RT1 ACTIVE
     ↓
transaction
     ↓
RT1 USED
     +
RT2 CREATED
     +
association established
     ↓
commit

Si la transacción falla:

rollback

y no debe producirse una rotación parcialmente aplicada.

16. Concurrency

Este es uno de los puntos más críticos de IS-009.

Escenario:

Request A ── RT1 ──┐
                   ├── concurrent
Request B ── RT1 ──┘

Solamente una solicitud debe poder ganar la rotación legítima.

La segunda solicitud debe detectar que:

RT1.status != ACTIVE

y tratarlo según las reglas de reuse/concurrency definidas.

La implementación deberá diferenciar cuidadosamente entre:

verdadera reutilización maliciosa;
retransmisión legítima;
retry de red;
solicitudes concurrentes.

La política exacta de tolerancia deberá definirse antes de producción.

17. Reuse Detection

Una reutilización ocurre cuando un Refresh Token que ya fue utilizado o invalidado vuelve a presentarse como si fuera válido.

Ejemplo:

RT1
 ↓
ROTATED
 ↓
RT2
 ↓
RT1 presented again
 ↓
REUSE DETECTED

Este evento debe considerarse una condición de seguridad crítica.

18. Response to Reuse

Ante una reutilización confirmada:

Refresh Token
      ↓
REUSE DETECTED
      ↓
Revoke Token Family
      ↓
Revoke associated Session
      ↓
Security Event
      ↓
Audit Event

La respuesta exacta puede ser refinada por Security Policy, pero el diseño base debe permitir contención de toda la family.

19. Why Revoke the Family?

Supongamos:

RT1 → RT2 → RT3 → RT4

Si RT2 fue robado y posteriormente se detecta reutilización:

RT2 reused

No es suficiente revocar solamente RT2.

El atacante podría intentar utilizar:

RT3
RT4

Por ello, el modelo debe permitir revocar la Token Family completa.

20. Session Relationship

Cada Refresh Token pertenece a una Session.

User
 ↓
Organization
 ↓
Membership
 ↓
Session
 ↓
Token Family
 ↓
Refresh Tokens

Esto permite que:

Revoke Session

pueda provocar:

Revoke Refresh Token Family

según las reglas establecidas.

21. User Suspension

Si el usuario pasa a:

SUSPENDED

las capacidades de refresh deben quedar bloqueadas.

El sistema debe evitar:

User suspended
     ↓
Refresh Token
     ↓
New Access Token

La suspensión del usuario debe prevalecer.

22. User Disabled

Un usuario:

DISABLED

no debe poder utilizar Refresh Tokens para recuperar acceso.

23. Organization Suspension

Si el Refresh Token pertenece a una sesión dentro de una organización suspendida:

Organization
    ↓
SUSPENDED

el refresh debe ser rechazado para ese contexto.

24. Membership Removal

Si la membership asociada a la sesión es eliminada:

Membership
    ↓
REMOVED

el Refresh Token no debe permitir recuperar acceso al contexto de esa organización.

25. Session Expiration

Si:

Session = EXPIRED

entonces:

Refresh Token
      ↓
DENY

Aunque el Refresh Token individual todavía no haya alcanzado su propia fecha expires_at.

La sesión establece un límite superior.

26. TTL Relationship

El ADR establece conceptualmente:

Access Token TTL
        <
Refresh Token TTL
        ≤
Session Maximum Lifetime

Los valores concretos quedan abiertos.

No se deben inventar en esta Implementation Story.

27. Logout

Logout de sesión actual:

POST /logout
       ↓
Resolve Session
       ↓
Revoke Session
       ↓
Revoke associated refresh capability

La implementación exacta de revocación de tokens debe mantener consistencia con la Session.

28. Logout All
User
 ↓
Logout All
 ↓
Find active sessions
 ↓
Revoke sessions
 ↓
Revoke associated token families

Esto debe ejecutarse de forma segura e idempotente.

29. Idempotency

Las operaciones de revocación deben ser idempotentes.

Por ejemplo:

Revoke RT1
Revoke RT1
Revoke RT1

no debe producir estados inconsistentes.

El resultado final debe seguir siendo:

REVOKED
30. Application Use Cases

La capa Application deberá contemplar:

CreateRefreshToken
RefreshAccessToken
RotateRefreshToken
RevokeRefreshToken
RevokeTokenFamily
GetRefreshToken
DetectRefreshTokenReuse

También podrán existir:

GetTokenFamily
RevokeSessionRefreshTokens

según el diseño final.

31. Domain Service
RefreshTokenService

Responsabilidades:

Validar Refresh Token.
Verificar lifecycle.
Verificar Session.
Validar expiración.
Rotar tokens.
Crear Token Family.
Detectar reuse.
Revocar family.
Emitir eventos.
Coordinar seguridad.

No debe depender directamente de:

FastAPI.
HTTP.
PostgreSQL.
Redis.
JWT libraries.
32. Token Generation

El Refresh Token presentado al cliente debe ser un secreto de alta entropía.

La persistencia no debe requerir almacenar el token en plaintext.

Conceptualmente:

Client
   ↓
Refresh Token
   ↓
Hash
   ↓
Database

La base de datos debe almacenar una representación que permita verificar el token sin exponer innecesariamente el secreto.

El algoritmo exacto de hashing/storage deberá quedar especificado durante la implementación de seguridad.

33. Refresh Token ≠ Access Token

Debe mantenerse una separación explícita:

Característica	Access Token	Refresh Token
Vida	Corta	Mayor
Estado	Principalmente stateless	Stateful
Persistencia	No necesariamente	Sí
Rotación	No	Sí
Family	No	Sí
Reuse detection	No	Sí
Revocación individual	Limitada por TTL	Sí
Asociado a Session	Sí	Sí
34. APIs Conceptuales

Los contratos definitivos se definirán posteriormente en IS-IDENTITY-014.

Refresh
POST /api/v1/auth/refresh
Revoke Token
POST /api/v1/identity/refresh-tokens/{token_id}/revoke
Revoke Family
POST /api/v1/identity/refresh-token-families/{family_id}/revoke
Logout
POST /api/v1/auth/logout

El cliente no debería enviar el Refresh Token en una URL.

35. Refresh Request

Conceptualmente:

RefreshRequest
├── refresh_token
└── context

El Refresh Token debe transmitirse mediante el mecanismo seguro definido por plataforma:

Web

Cookie:

HttpOnly
Secure
Mobile

Almacenamiento seguro del sistema operativo.

No debe utilizarse:

localStorage
URL query parameter
URL path
logs

para almacenar/transmitir Refresh Tokens.

36. Refresh Response

Conceptualmente:

RefreshResponse
├── access_token
├── refresh_token
├── token_type
└── expires_in

El contrato definitivo queda para IS-014.

37. Security Events

Debe existir al menos:

RefreshTokenCreated
RefreshTokenRotated
RefreshTokenRevoked
RefreshTokenExpired
RefreshTokenReuseDetected
TokenFamilyRevoked

El evento más importante desde el punto de vista de detección es:

RefreshTokenReuseDetected
38. Audit

Las siguientes acciones deben poder auditarse:

REFRESH_TOKEN_CREATED
REFRESH_TOKEN_ROTATED
REFRESH_TOKEN_REVOKED
REFRESH_TOKEN_REUSE_DETECTED
TOKEN_FAMILY_REVOKED

En particular, un REUSE_DETECTED debe contener suficiente información para investigación de seguridad.

Nunca:

refresh_token
access_token
password
secret
private_key

en audit logs.

39. Observability
Metrics
identity.refresh.success
identity.refresh.failure
identity.refresh.rotation
identity.refresh.expired
identity.refresh.revoked
identity.refresh.reuse_detected
identity.refresh.family_revoked

Métricas adicionales podrán incluir:

refresh latency
rotation latency
reuse detection rate
concurrent rotation conflicts
40. Logging

Los logs deben permitir investigar:

correlation_id
trace_id
user_id
session_id
organization_id
token_id
family_id
operation
result
reason

pero nunca deben registrar:

raw refresh token
raw access token
password
MFA secrets
private keys
41. Failure Modes
Database Failure

Si PostgreSQL no está disponible:

Refresh
   ↓
FAIL CLOSED

No se debe emitir un nuevo Access Token basándose en información de sesión obsoleta si no puede verificarse la fuente de verdad requerida.

Token Already Used
RT1 = USED
      ↓
Present RT1
      ↓
Reuse detection

Debe aplicarse la respuesta de seguridad correspondiente.

Token Expired
RT1 = EXPIRED
      ↓
DENY

No debe generarse nuevo Access Token.

Session Revoked
Session = REVOKED
      ↓
DENY REFRESH
Organization Suspended
Organization = SUSPENDED
      ↓
DENY REFRESH
Membership Removed
Membership = REMOVED
      ↓
DENY REFRESH
42. Concurrency Model

La implementación debe garantizar atomicidad en:

Validate token
      +
Check ACTIVE
      +
Mark USED
      +
Create replacement
      +
Commit

Debe estudiarse el uso de mecanismos como:

Transaction.
Row-level locking.
Optimistic concurrency.
Unique constraints.
Atomic compare-and-set.

La elección concreta queda para la especificación técnica.

43. Persistence

PostgreSQL será la fuente de verdad.

Conceptualmente:

refresh_tokens
--------------
id
session_id
family_id
parent_token_id
token_hash
status
issued_at
expires_at
used_at
revoked_at
revoke_reason
created_at
metadata

Puede existir una entidad/estructura adicional para:

refresh_token_families

si el diseño final lo considera conveniente.

44. Índices

Deben estudiarse índices sobre:

id
session_id
family_id
parent_token_id
status
expires_at
token_hash

Los índices definitivos deberán definirse en IS-015.

45. Retention / Cleanup

Los tokens:

EXPIRED
USED
REVOKED

pueden requerir políticas de retención y limpieza.

Sin embargo, la eliminación física no debe destruir información necesaria para:

auditoría;
investigación de seguridad;
detección;
compliance;
correlación de eventos.

La política definitiva de retention queda pendiente.

46. Business Rules
BR-009-001 — Stateful Refresh

Los Refresh Tokens deben poder validarse contra estado persistente.

BR-009-002 — Session Association

Todo Refresh Token debe estar asociado a una Session válida.

BR-009-003 — Rotation

Un Refresh Token utilizado exitosamente debe quedar invalidado y ser reemplazado.

BR-009-004 — Single Use

Un Refresh Token no debe poder utilizarse exitosamente más de una vez.

BR-009-005 — Family

Los tokens rotados pertenecen a una misma Token Family.

BR-009-006 — Reuse Detection

La reutilización de un token previamente usado debe generar una condición de seguridad.

BR-009-007 — Family Revocation

Ante reuse confirmado, la plataforma debe poder revocar la Token Family.

BR-009-008 — Session Revocation

Un compromiso confirmado puede provocar la revocación de la Session asociada.

BR-009-009 — Expiration

Un token expirado no puede generar nuevos tokens.

BR-009-010 — User State

Un usuario suspendido o deshabilitado no puede utilizar Refresh Tokens para recuperar acceso.

BR-009-011 — Tenant Isolation

El refresh nunca puede utilizarse para cambiar de contexto organizacional sin autorización válida.

BR-009-012 — No Secrets in Logs

Los Refresh Tokens nunca deben registrarse en plaintext.

BR-009-013 — Fail Closed

Cuando no pueda determinarse de forma segura si el refresh es válido, se debe rechazar.

BR-009-014 — Atomic Rotation

La rotación debe ser transaccional y resistente a concurrencia.

47. Acceptance Criteria
AC-009-01 — Refresh válido

Given una Session ACTIVE y un Refresh Token ACTIVE
When el cliente solicita refresh
Then debe recibir un nuevo Access Token y un nuevo Refresh Token.

AC-009-02 — Rotación

Given un Refresh Token utilizado exitosamente
When termina la operación
Then el token anterior debe quedar USED y el nuevo token debe pertenecer a la misma family.

AC-009-03 — Reutilización

Given un Refresh Token USED
When vuelve a presentarse
Then debe detectarse una posible reutilización.

AC-009-04 — Reuse Response

Given una reutilización confirmada
When se procesa el incidente
Then la Token Family debe poder ser revocada y debe generarse un Security Event.

AC-009-05 — Session Revoked

Given una Session REVOKED
When se presenta un Refresh Token asociado
Then el refresh debe ser rechazado.

AC-009-06 — Session Expired

Given una Session EXPIRED
When se utiliza un Refresh Token
Then debe rechazarse.

AC-009-07 — Expired Token

Given un Refresh Token expirado
When se solicita refresh
Then debe rechazarse.

AC-009-08 — Suspended User

Given un usuario SUSPENDED
When intenta utilizar un Refresh Token
Then debe rechazarse.

AC-009-09 — Organization Suspended

Given una organización SUSPENDED
When se intenta refresh dentro de ese contexto
Then debe rechazarse.

AC-009-10 — Concurrent Refresh

Given dos solicitudes concurrentes utilizando el mismo Refresh Token
When ambas intentan rotarlo
Then el sistema debe mantener consistencia y solamente una rotación debe resultar válida.

AC-009-11 — Logout

Given una sesión activa
When el usuario ejecuta logout
Then la capacidad de refresh asociada debe quedar invalidada.

AC-009-12 — Logout All

Given múltiples sesiones activas
When el usuario ejecuta logout-all
Then las sesiones y capacidades de refresh correspondientes deben quedar revocadas.

48. Technical Tasks
TSK-009-01

Crear RefreshToken Domain Entity.

TSK-009-02

Crear RefreshTokenId Value Object.

TSK-009-03

Definir Token Family.

TSK-009-04

Definir Token Lineage.

TSK-009-05

Definir lifecycle de Refresh Token.

TSK-009-06

Implementar reglas de transición.

TSK-009-07

Crear RefreshTokenRepository.

TSK-009-08

Crear RefreshTokenService.

TSK-009-09

Implementar CreateRefreshToken.

TSK-009-10

Implementar RefreshAccessToken.

TSK-009-11

Implementar RotateRefreshToken.

TSK-009-12

Implementar RevokeRefreshToken.

TSK-009-13

Implementar RevokeTokenFamily.

TSK-009-14

Implementar DetectRefreshTokenReuse.

TSK-009-15

Implementar hashing/storage seguro del token.

TSK-009-16

Implementar asociación Session ↔ Token Family.

TSK-009-17

Implementar asociación Parent Token → Child Token.

TSK-009-18

Implementar expiración.

TSK-009-19

Implementar atomic rotation.

TSK-009-20

Implementar concurrency controls.

TSK-009-21

Integrar con Authentication.

TSK-009-22

Integrar con Session Management.

TSK-009-23

Integrar con User lifecycle.

TSK-009-24

Integrar con Organization lifecycle.

TSK-009-25

Integrar con Membership lifecycle.

TSK-009-26

Implementar Logout.

TSK-009-27

Implementar Logout All.

TSK-009-28

Implementar Security Events.

TSK-009-29

Implementar Audit Events.

TSK-009-30

Implementar métricas.

TSK-009-31

Implementar logging seguro.

TSK-009-32

Implementar tracing.

TSK-009-33

Crear persistence model PostgreSQL.

TSK-009-34

Crear índices.

TSK-009-35

Crear transaction boundaries.

TSK-009-36

Implementar cleanup/retention strategy.

TSK-009-37

Unit tests.

TSK-009-38

Integration tests.

TSK-009-39

Concurrency tests.

TSK-009-40

Security tests.

TSK-009-41

Reuse Detection tests.

TSK-009-42

Token Family revocation tests.

TSK-009-43

Cross-tenant tests.

TSK-009-44

API integration tests.

TSK-009-45

Documentar lifecycle y security model.

49. Testing Strategy
Unit

Cubrir:

Token lifecycle.
Rotation.
Family.
Lineage.
Expiration.
Revocation.
Reuse Detection.
Integration

Validar:

PostgreSQL
Session
User
Organization
Membership
RefreshToken
Security

Debe probarse:

Token theft.
Token replay.
Token reuse.
Family revocation.
Session revocation.
Cross-tenant access.
User suspension.
Organization suspension.
Membership removal.
Concurrent refresh.
Concurrency

Escenario obligatorio:

             ┌── Request A
RT1 ─────────┤
             └── Request B

Debe garantizarse que no se generen dos rotaciones válidas independientes.

50. Definition of Done
 RefreshToken Entity implementada.
 RefreshTokenId implementado.
 Token Family implementada.
 Token Lineage implementado.
 Lifecycle implementado.
 Rotation implementada.
 Single-use implementado.
 Reuse Detection implementado.
 Family Revocation implementada.
 Session integration implementada.
 User lifecycle integration implementada.
 Organization lifecycle integration implementada.
 Membership integration implementada.
 PostgreSQL persistence implementada.
 Token storage seguro implementado.
 Atomic rotation implementada.
 Concurrency controls implementados.
 Logout implementado.
 Logout-all implementado.
 Security Events implementados.
 Audit implementado.
 Metrics implementadas.
 Logging seguro implementado.
 Tracing implementado.
 Unit tests implementados.
 Integration tests implementados.
 Security tests implementados.
 Concurrency tests implementados.
 Cross-tenant tests implementados.
 API integration preparada para IS-014.
 No se registran tokens en plaintext.
 No existen bypasses de Session/Organization/Membership.
 Documentación actualizada.
51. Traceability
Elemento	Referencia
Blueprint	Identity / Session & Authentication
ESP	ESP-0001 — Identity
ADR	ADR-IDENTITY-002
ADR	ADR-IDENTITY-003
ADR	ADR-IDENTITY-005
ADR	ADR-IDENTITY-006
ADR	ADR-IDENTITY-007
ADR	ADR-IDENTITY-008
ADR	ADR-IDENTITY-009
ADR	ADR-IDENTITY-010
ADR	ADR-IDENTITY-011
Previous IS	IS-IDENTITY-008 — Session Management
Authentication	IS-IDENTITY-005
Authorization	IS-IDENTITY-006
API	IS-IDENTITY-014
Persistence	IS-IDENTITY-015
Testing	IS-IDENTITY-016
Runtime	IS-IDENTITY-017

La trazabilidad mantiene la relación ESP → IS → Technical Tasks → Source Code, conforme al lifecycle establecido por el Blueprint.

52. Architecture Model
                         ┌───────────────────┐
                         │    Client         │
                         │ Web / Mobile      │
                         └─────────┬─────────┘
                                   │
                              Refresh
                                   │
                                   ↓
                         ┌───────────────────┐
                         │   Identity API    │
                         └─────────┬─────────┘
                                   ↓
                         ┌───────────────────┐
                         │ Application       │
                         │                   │
                         │ RefreshAccess     │
                         │ RotateToken       │
                         │ RevokeFamily      │
                         └─────────┬─────────┘
                                   ↓
                         ┌───────────────────┐
                         │ Domain            │
                         │                   │
                         │ RefreshToken     │
                         │ TokenFamily      │
                         │ RefreshService   │
                         └─────────┬─────────┘
                                   ↓
                ┌──────────────────┼──────────────────┐
                ↓                  ↓                  ↓
         ┌─────────────┐    ┌─────────────┐    ┌──────────────┐
         │ PostgreSQL  │    │   Redis     │    │ Event        │
         │ Source      │    │ Temporary   │    │ Platform     │
         │ of Truth    │    │ / Cache     │    │ / Outbox     │
         └─────────────┘    └─────────────┘    └──────────────┘
53. Security Model

El modelo completo queda:

                 Authentication
                       │
                       ↓
                    Session
                       │
                       ↓
                Token Family
                       │
             ┌─────────┴─────────┐
             ↓                   ↓
       Refresh Token 1     Refresh Token N
             │
             ↓
          Rotation
             │
             ↓
       Reuse Detection
             │
             ↓
      Family Revocation
             │
             ↓
       Session Revocation
             │
             ↓
       Security / Audit

Esto convierte al Refresh Token Lifecycle en una barrera de seguridad, no solamente en un mecanismo de renovación.

54. Architectural Decisions Pending

Antes de marcar esta IS como Approved, deben definirse explícitamente:

TTL exacto de Refresh Token.
Session Maximum Lifetime.
Inactivity timeout.
Estrategia exacta ante concurrencia.
Política de retry/retransmisión.
Algoritmo de hashing de Refresh Tokens.
Estructura definitiva de Token Family.
Retention de tokens históricos.
Política de cleanup.
Estrategia de detección de reutilización.
Severidad de RefreshTokenReuseDetected.
Estrategia de revocación inmediata de Access Tokens.
Cookie policy definitiva para Web.
Secure Storage abstraction definitiva para Mobile.

Estos valores no deben inventarse mientras no estén aprobados arquitectónicamente.

55. Dependency Flow

La secuencia ahora queda:

IS-IDENTITY-001
Identity Foundation
       ↓
IS-IDENTITY-002
User
       ↓
IS-IDENTITY-003
Organization
       ↓
IS-IDENTITY-004
Membership
       ↓
IS-IDENTITY-005
Authentication
       ↓
IS-IDENTITY-006
Authorization
       ↓
IS-IDENTITY-007
Roles & Permissions
       ↓
IS-IDENTITY-008
Session Management
       ↓
IS-IDENTITY-009
Refresh Token Lifecycle
       ↓
IS-IDENTITY-010
MFA
       ↓
IS-IDENTITY-011
Security Policy Engine
       ↓
IS-IDENTITY-012
Audit & Security Events
       ↓
IS-IDENTITY-013
Domain Events & Outbox
       ↓
IS-IDENTITY-014
Identity API Contracts
       ↓
IS-IDENTITY-015
Identity Persistence
       ↓
IS-IDENTITY-016
Identity Testing & Quality
       ↓
IS-IDENTITY-017
Identity Runtime & Deployment
Estado

IS-IDENTITY-009 — Refresh Token Lifecycle queda definida como Draft — Critical.

Esta historia deja preparada una de las piezas más sensibles de seguridad de EVOXA: Session → Refresh Token → Rotation → Token Family → Reuse Detection → Revocation.

El siguiente bloque lógico es IS-IDENTITY-010 — Multi-Factor Authentication, donde podremos definir MFA sin duplicar la lógica ya establecida para Authentication, Session y Refresh Token.
