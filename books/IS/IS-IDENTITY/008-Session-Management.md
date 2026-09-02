IS-IDENTITY-008 — Session Management

Estado: Draft
Prioridad: Critical
Dominio: Identity
Capability: Session Management
Dependencias principales: IS-IDENTITY-001, 002, 003, 004, 005, 006, 007
Siguiente historia: IS-IDENTITY-009 — Refresh Token Lifecycle

1. Propósito

Definir e implementar el ciclo de vida de las sesiones autenticadas de EVOXA, proporcionando un mecanismo persistente y controlable para representar una sesión activa de un usuario dentro de un contexto de organización/tenant.

La sesión será el vínculo entre:

Usuario autenticado.
Organización/tenant.
Membership.
Método de autenticación.
Estado de autenticación.
Dispositivo/contexto.
Access Token.
Refresh Token.
Políticas de seguridad.
Revocación.
Auditoría.

La sesión debe permitir que EVOXA pueda crear, consultar, expirar, revocar y administrar sesiones, incluyendo escenarios multi-dispositivo.

Importante: esta historia define la sesión. El ciclo de vida específico de los Refresh Tokens será desarrollado en IS-IDENTITY-009.

2. Business Goal

Como plataforma EVOXA, necesitamos administrar sesiones autenticadas de manera persistente y segura para poder:

Controlar sesiones activas.
Revocar sesiones comprometidas.
Permitir múltiples dispositivos.
Cerrar la sesión actual.
Cerrar todas las sesiones.
Detectar sesiones expiradas.
Asociar tokens a una sesión.
Aplicar políticas de seguridad.
Reaccionar ante suspensión de usuarios.
Reaccionar ante suspensión de organizaciones.
Reaccionar ante cambios de membership.
Mantener trazabilidad de seguridad.

El objetivo es evitar que la autenticación dependa exclusivamente de un JWT stateless y disponer de un estado de sesión administrable por la plataforma.

3. Persona
Primary Persona

Authenticated User

Usuario que ha completado correctamente el proceso de autenticación.

Secondary Personas
Coach
Nutritionist
Physiotherapist
Team Manager
Admin
Security Administrator
System Service
4. User Journey
4.1 Inicio de sesión
User
 ↓
Authentication
 ↓
Validate User
 ↓
Validate Organization/Membership
 ↓
Security Policy
 ↓
MFA if required
 ↓
Create Session
 ↓
Create Access Token
 ↓
Create Refresh Token
 ↓
Authenticated

La creación de sesión ocurre como parte del proceso definido por IS-IDENTITY-005 — Authentication.

4.2 Uso de una sesión
Session ACTIVE
      ↓
Access Token
      ↓
API Request
      ↓
Authorization
      ↓
ALLOW / DENY / ...

La sesión proporciona contexto para las decisiones posteriores de autorización.

4.3 Logout
User
 ↓
Logout
 ↓
Revoke Session
 ↓
Revoke / invalidate associated refresh capability
 ↓
Audit
 ↓
Security Event

El detalle de Refresh Token pertenece a IS-IDENTITY-009.

5. Feature Pack

Esta Implementation Story cubre:

Session Core
Creación de sesiones.
Consulta de sesiones.
Estado de sesión.
Actualización controlada.
Expiración.
Revocación.
Session Security
Asociación con usuario.
Asociación con membership.
Asociación con organización.
Método de autenticación.
Estado MFA.
Información del dispositivo/contexto.
IP y metadata de seguridad cuando corresponda.
Session Administration
Sesión actual.
Sesiones del usuario.
Revocar sesión.
Revocar todas las sesiones.
Gestión multi-dispositivo.
Security Integration
User suspension.
Organization suspension.
Membership removal.
Security policies.
Audit events.
Domain events.
6. Scope
In Scope
Entidad Session.
SessionId.
Lifecycle de sesión.
Session Repository.
Session Service.
Creación de sesión.
Activación.
Expiración.
Revocación.
Consulta de sesiones.
Logout.
Logout all.
Multi-device sessions.
Integración con Authentication.
Integración con Authorization.
Integración con Security Policy.
Auditoría.
Domain Events.
Observabilidad.
Persistencia.
Out of Scope

No se define aquí:

Implementación completa de Refresh Tokens.
Rotación de Refresh Tokens.
Refresh Token Families.
Refresh Token Reuse Detection.
MFA completo.
Password authentication.
Role management.
Permission management.
Security Policy Engine completo.

Estos corresponden a otras IS.

7. Modelo de Session

La entidad conceptual Session debe representar una sesión autenticada persistente.

Atributos principales
Session
├── id
├── user_id
├── organization_id
├── membership_id
├── status
├── created_at
├── activated_at
├── last_activity_at
├── expires_at
├── revoked_at
├── revoke_reason
├── auth_method
├── mfa_status
├── device_context
├── client_context
└── lifecycle metadata

Algunos atributos específicos podrán ser refinados durante la especificación de persistencia.

8. SessionId

Value Object:

SessionId

Responsabilidades:

Identificar inequívocamente una sesión.
Validar formato.
Evitar uso directo de strings sin validación en el dominio.
9. Session Lifecycle

El Blueprint/ADR establece el siguiente modelo conceptual:

CREATED
   ↓
 ACTIVE
  ↙   ↘
EXPIRED  REVOKED
CREATED

La sesión ha sido creada pero aún no está completamente activa.

ACTIVE

Sesión autenticada y válida.

EXPIRED

La sesión alcanzó su límite temporal.

REVOKED

La sesión fue invalidada explícitamente por seguridad, logout, administración u otra política.

10. Reglas de transición
CREATED → ACTIVE

Permitido cuando:

Authentication terminó correctamente.
User es válido.
Contexto organizacional es válido cuando corresponde.
Security Policy permite la sesión.
MFA requerido fue completado.
CREATED → REVOKED

Permitido si:

Authentication es cancelada.
MFA falla de manera definitiva.
Security Policy determina revocación.
Se detecta una condición de seguridad.
ACTIVE → EXPIRED

Permitido cuando:

Se alcanza el límite absoluto de sesión.
Se alcanza otro límite temporal configurado.
ACTIVE → REVOKED

Permitido por:

Logout.
Logout all.
User suspension.
User disable.
Organization suspension.
Membership removal.
Security policy.
Compromiso de seguridad.
Administración explícita.
Detección de actividad sospechosa.
EXPIRED → ACTIVE

No permitido.

Debe crearse una nueva sesión mediante autenticación.

REVOKED → ACTIVE

No permitido.

Debe crearse una nueva sesión.

11. Session Context

Cada sesión debe mantener suficiente contexto para permitir decisiones de seguridad.

Conceptualmente:

SessionContext
├── user_id
├── organization_id
├── membership_id
├── authentication_method
├── mfa_status
├── device
├── client
├── network/context metadata
└── security metadata

La información debe limitarse a lo necesario para seguridad, autorización, auditoría y operación.

No se deben almacenar secretos innecesarios.

12. Authentication Method

La sesión debe registrar el método mediante el cual fue autenticada.

Ejemplos conceptuales:

PASSWORD
MFA
OIDC
OAUTH
API_TOKEN
SERVICE_ACCOUNT

El catálogo definitivo de métodos deberá ser normalizado posteriormente.

13. MFA Status

La sesión debe poder representar el nivel de autenticación alcanzado.

Ejemplos:

NOT_REQUIRED
PENDING
VERIFIED
FAILED

La implementación completa de MFA pertenece a IS-IDENTITY-010.

14. Multi-Device Sessions

EVOXA debe soportar múltiples sesiones simultáneas.

Ejemplo:

User
 ├── Session A → Web Browser
 ├── Session B → Mobile
 ├── Session C → Tablet
 └── Session D → Other Device

Cada sesión posee su propio:

Session ID.
Lifecycle.
Contexto.
Refresh capability.
Estado.
Metadata.

La existencia de una nueva sesión no debe invalidar automáticamente las demás, salvo que una Security Policy establezca dicha condición.

15. Session Limits

El sistema debe estar preparado para soportar:

Maximum concurrent sessions.
Session absolute lifetime.
Sliding session lifetime.
Device limits.
Risk-based session restrictions.

Sin embargo, los valores concretos no deben inventarse en esta IS.

Deben quedar como configuración/policy pendiente de definición.

16. Absolute vs Sliding Lifetime

La arquitectura contempla ambos conceptos.

Absolute Lifetime

La sesión tiene una fecha máxima de expiración:

created_at + maximum_session_lifetime

La actividad del usuario no extiende ese límite.

Sliding Lifetime

La expiración puede desplazarse en función de actividad válida.

last_activity + inactivity_timeout

La política exacta debe ser definida posteriormente.

17. Relación con Access Token

La relación conceptual es:

Session
   │
   ├── Access Token
   │
   └── Refresh Token

El Access Token es de corta duración y puede continuar siendo válido hasta su expiración debido a su naturaleza stateless.

Por lo tanto:

Revocar una sesión no implica necesariamente que un Access Token ya emitido quede inmediatamente inválido.

La estrategia de revocación inmediata de Access Tokens queda abierta para una decisión arquitectónica posterior.

18. Relación con Refresh Token

Cada Refresh Token debe estar asociado a una sesión.

Session 1
   │
   ├── Refresh Token Family
   │      ├── Token 1
   │      ├── Token 2
   │      └── Token 3
   │
   └── Access Tokens

La implementación de:

rotation,
token families,
reuse detection,
atomic rotation,
revocation,

será desarrollada en:

IS-IDENTITY-009 — Refresh Token Lifecycle.

19. Application Use Cases

La capa Application deberá proporcionar, como mínimo:

CreateSession
GetSession
GetCurrentSession
ListUserSessions
ActivateSession
ExpireSession
RevokeSession
RevokeAllUserSessions

Opcionalmente podrán existir:

ValidateSession
TouchSessionActivity
GetOrganizationSessions

según las necesidades finales de API y runtime.

20. Domain Service
SessionService

Responsabilidades:

Crear sesiones.
Validar transiciones.
Activar sesiones.
Expirar sesiones.
Revocar sesiones.
Aplicar reglas de lifecycle.
Coordinar políticas de sesión.
Generar eventos de dominio.

No debe depender directamente de:

FastAPI.
PostgreSQL.
Redis.
HTTP.
JWT libraries.
21. Repository Contract

Debe existir un contrato:

SessionRepository

Con operaciones conceptuales:

create()
get_by_id()
list_by_user()
list_active_by_user()
list_by_organization()
update()
activate()
expire()
revoke()
revoke_all_for_user()

La implementación concreta pertenece a Infrastructure.

22. Persistence

La persistencia debe utilizar PostgreSQL como source of truth transaccional.

Conceptualmente:

sessions
---------
id
user_id
organization_id
membership_id
status
created_at
activated_at
last_activity_at
expires_at
revoked_at
revoke_reason
auth_method
mfa_status
device_context
client_context
metadata

Los nombres definitivos de columnas corresponden a la especificación de persistencia.

Redis podrá utilizarse para:

Cache.
Datos temporales.
High-frequency operations.

Pero Redis no sustituye PostgreSQL como fuente transaccional de identidad.

23. Business Rules
BR-001 — User Required

Una sesión debe pertenecer a un User válido.

BR-002 — Organization Context

Cuando una sesión opera dentro de una organización, debe existir un contexto organizacional válido.

BR-003 — Membership

Cuando el acceso depende de una organización, la membership debe ser válida.

BR-004 — User State

Un usuario:

SUSPENDED
DISABLED

no debe poder mantener sesiones efectivamente utilizables.

BR-005 — Organization State

Una organización suspendida debe impedir el acceso efectivo al contexto organizacional.

BR-006 — Membership Removal

Una membership eliminada debe impedir la utilización efectiva de la sesión dentro de esa organización.

BR-007 — Revoked Sessions

Una sesión REVOKED no puede volver a ACTIVE.

BR-008 — Expired Sessions

Una sesión EXPIRED no puede volver a ACTIVE.

BR-009 — Tenant Isolation

Una sesión de una organización no puede utilizarse para acceder a recursos de otra organización sin autorización explícita válida.

BR-010 — No Secrets

La entidad Session no debe almacenar:

Password.
Password hash.
Refresh Token plaintext.
TOTP secret.
Private keys.
Recovery codes.
BR-011 — Auditability

Las operaciones de seguridad sobre sesiones deben poder auditarse.

BR-012 — Fail Closed

Ante una condición de seguridad no resoluble, la sesión debe tratarse como no autorizada/no utilizable.

24. User Stories
US-008-01 — Create Session

Como sistema de autenticación
Quiero crear una sesión después de una autenticación válida
Para mantener un contexto persistente de autenticación.

Acceptance Criteria

Given un usuario autenticado correctamente
When Authentication solicita crear una sesión
Then debe crearse una sesión válida.

US-008-02 — Activate Session

Given una sesión CREATED
When la autenticación finaliza correctamente
Then la sesión debe pasar a ACTIVE.

US-008-03 — Get Current Session

Given un access token válido asociado a una sesión
When se solicita el contexto actual
Then debe poder resolverse la sesión correspondiente.

US-008-04 — Revoke Session

Given una sesión ACTIVE
When el usuario ejecuta logout
Then la sesión debe pasar a REVOKED.

US-008-05 — Revoke All Sessions

Given un usuario autenticado
When solicita cerrar todas sus sesiones
Then todas las sesiones activas aplicables deben ser revocadas.

US-008-06 — Session Expiration

Given una sesión activa
When alcanza su límite temporal
Then debe pasar a EXPIRED.

US-008-07 — Suspended User

Given un usuario con sesiones activas
When el usuario pasa a SUSPENDED
Then sus sesiones deben quedar sujetas a revocación/inutilización conforme a la política de seguridad.

US-008-08 — Organization Suspension

Given una sesión dentro de una organización
When la organización pasa a SUSPENDED
Then el acceso efectivo de la sesión a esa organización debe quedar bloqueado.

US-008-09 — Membership Removal

Given una sesión asociada a una membership
When la membership es eliminada
Then la sesión no debe poder continuar accediendo al contexto de esa organización.

US-008-10 — Multi-Device

Given un usuario con una sesión Web activa
When inicia sesión desde Mobile
Then debe poder existir una segunda sesión independiente.

25. APIs Conceptuales

Los contratos definitivos se especificarán en:

IS-IDENTITY-014 — Identity API Contracts.

Se contemplan conceptualmente:

Current Session
GET /api/v1/identity/sessions/current
User Sessions
GET /api/v1/identity/sessions
Session Detail
GET /api/v1/identity/sessions/{session_id}
Revoke Session
POST /api/v1/identity/sessions/{session_id}/revoke
Revoke All
POST /api/v1/identity/sessions/revoke-all

No se considera este bloque como contrato API definitivo todavía.

26. Events

La sesión debe generar eventos de dominio.

SessionCreated
session.created
SessionActivated
session.activated
SessionRevoked
session.revoked
SessionExpired
session.expired
SessionSecurityViolation

Podrá utilizarse cuando corresponda a una condición de seguridad.

27. Event Payload

Los eventos deben respetar el envelope estándar de EVOXA:

event_id
event_type
event_version
timestamp
tenant_id
actor
resource
payload
metadata
correlation_id
causation_id
trace_id

Nunca deben incluir:

password
password_hash
refresh_token
access_token
TOTP secret
private_key
recovery_code
28. Audit

Las operaciones relevantes deben generar Audit Events.

Ejemplos:

SESSION_CREATED
SESSION_ACTIVATED
SESSION_REVOKED
SESSION_EXPIRED
SESSION_REVOKE_ALL
SESSION_SECURITY_VIOLATION

El audit debe registrar suficiente información para investigar:

quién;
qué;
cuándo;
desde qué contexto;
sobre qué organización;
sobre qué sesión;
resultado;
correlation ID.
29. Security

La implementación debe contemplar:

Session Hijacking

La sesión debe estar vinculada a un contexto de autenticación verificable.

Token Theft

La revocación de sesión debe impedir la continuidad mediante mecanismos de refresh asociados.

Session Fixation

No debe reutilizarse una sesión creada antes de una autenticación válida cuando el flujo requiera una nueva sesión.

Concurrent Access

Las operaciones de revocación y actualización deben ser seguras ante concurrencia.

Cross-Tenant Access

La sesión no debe utilizarse como mecanismo para saltarse tenant isolation.

30. Authorization Integration

La existencia de una sesión ACTIVE no implica autorización automática.

El flujo debe ser:

Authenticated Session
        ↓
Authorization Context
        ↓
RBAC
        +
ABAC
        +
Tenant Isolation
        +
Ownership
        +
Security Policy
        ↓
Authorization Decision

Por tanto:

Session ≠ Permission

La sesión proporciona contexto de autenticación.

La autorización determina qué puede hacer ese usuario.

31. Observability

Debe existir observabilidad sobre:

Metrics
identity.sessions.created
identity.sessions.active
identity.sessions.revoked
identity.sessions.expired
identity.sessions.revoke_all
identity.sessions.security_events
Logs

Los logs deben incluir:

correlation_id
trace_id
session_id
user_id
organization_id
event_type
result

Nunca secretos.

Tracing

Las operaciones de sesión deben integrarse con el sistema distribuido de tracing de EVOXA.

32. Performance

Las operaciones normales de sesión deben diseñarse para cumplir los objetivos generales de Identity.

Se debe evitar:

Consultas innecesarias.
N+1.
Bloqueos prolongados.
Dependencia sincrónica innecesaria de servicios externos.

La optimización debe mantener PostgreSQL como fuente de verdad.

33. Failure Modes
Database Unavailable

La operación que requiere persistencia no debe asumir éxito.

Redis Unavailable

Las operaciones críticas no deben perder integridad de sesión por una dependencia exclusiva de Redis.

Concurrent Revoke

Dos solicitudes simultáneas de revocación deben producir un estado consistente.

Expiration Race

Una solicitud concurrente no debe reactivar una sesión ya expirada o revocada.

User Suspension Race

La suspensión de usuario debe prevalecer sobre operaciones concurrentes que intenten mantener acceso.

34. Technical Tasks
TSK-008-01 — Session Domain Entity

Crear entidad de dominio Session.

TSK-008-02 — SessionId Value Object

Implementar SessionId.

TSK-008-03 — Session Lifecycle

Implementar:

CREATED
ACTIVE
EXPIRED
REVOKED

y sus transiciones válidas.

TSK-008-04 — Session Context

Definir el modelo de contexto de sesión.

TSK-008-05 — Session Repository Contract

Definir interfaz:

SessionRepository
TSK-008-06 — Session Service

Implementar reglas de dominio mediante SessionService.

TSK-008-07 — Create Session Use Case

Implementar CreateSession.

TSK-008-08 — Activate Session Use Case

Implementar ActivateSession.

TSK-008-09 — Get Session Use Case

Implementar consulta individual.

TSK-008-10 — Current Session Use Case

Implementar resolución de sesión actual.

TSK-008-11 — List Sessions Use Case

Permitir listar sesiones del usuario según autorización.

TSK-008-12 — Revoke Session

Implementar revocación individual.

TSK-008-13 — Revoke All Sessions

Implementar revocación masiva.

TSK-008-14 — Session Expiration

Implementar mecanismo de expiración.

TSK-008-15 — Persistence Model

Crear modelo de persistencia PostgreSQL.

TSK-008-16 — Session Indexes

Definir índices para:

user_id
organization_id
membership_id
status
expires_at

según el diseño final.

TSK-008-17 — Authentication Integration

Integrar SessionService con IS-IDENTITY-005.

TSK-008-18 — Authorization Integration

Integrar sesión con Authorization Context.

TSK-008-19 — User Suspension Integration

Revocar/inutilizar sesiones según la política establecida al suspender usuarios.

TSK-008-20 — Organization Suspension Integration

Aplicar suspensión organizacional a las sesiones afectadas.

TSK-008-21 — Membership Integration

Revalidar contexto de membership.

TSK-008-22 — Session Events

Implementar:

SessionCreated
SessionActivated
SessionRevoked
SessionExpired
TSK-008-23 — Audit Integration

Integrar Session Management con AuditService.

TSK-008-24 — Observability

Implementar métricas, logs y tracing.

TSK-008-25 — Concurrency Controls

Validar race conditions en:

revoke;
expire;
activate;
revoke-all.
TSK-008-26 — Security Tests

Implementar pruebas contra:

session hijacking;
session fixation;
cross-tenant access;
unauthorized session revocation;
privilege escalation.
TSK-008-27 — Multi-Device Tests

Validar múltiples sesiones simultáneas.

TSK-008-28 — Lifecycle Tests

Cubrir todas las transiciones válidas e inválidas.

TSK-008-29 — API Integration

Preparar implementación para los contratos definitivos de IS-014.

TSK-008-30 — Documentation

Documentar:

lifecycle;
API behavior;
events;
persistence;
security;
operational behavior.
35. Testing Strategy
Unit Tests

Cobertura de:

Session.
SessionId.
Lifecycle.
State transitions.
Business Rules.
SessionService.
Integration Tests

Validar:

PostgreSQL.
Repository.
Authentication.
Authorization.
Membership.
Organization.
API Tests

Validar:

GET current
GET sessions
GET session
POST revoke
POST revoke-all

cuando los contratos de IS-014 estén definidos.

Security Tests

Especialmente:

Cross-tenant.
Unauthorized revoke.
Revoked session reuse.
Expired session reuse.
Concurrent revocation.
Session fixation.
Token/session mismatch.
Performance Tests

Validar operaciones de:

creación;
consulta;
revocación;
listado.
36. Definition of Done

IS-IDENTITY-008 se considerará terminada cuando:

 Session Domain Entity implementada.
 SessionId implementado.
 Lifecycle implementado.
 State transitions validadas.
 SessionRepository definido.
 SessionService implementado.
 Use cases implementados.
 PostgreSQL persistence implementada.
 Índices definidos.
 Authentication integrada.
 Authorization integrada.
 User suspension integrada.
 Organization suspension integrada.
 Membership validation integrada.
 Multi-device soportado.
 Session revoke implementado.
 Revoke-all implementado.
 Expiration implementada.
 Events implementados.
 Audit implementado.
 Metrics implementadas.
 Logging seguro implementado.
 Tracing integrado.
 Unit tests implementados.
 Integration tests implementados.
 Security tests implementados.
 Concurrency tests implementados.
 API tests preparados/implementados según IS-014.
 No existen secretos en logs/events.
 No existen bypasses de tenant isolation.
 No existen transiciones inválidas de lifecycle.
 Documentación actualizada.
37. Traceability
Elemento	Referencia
Blueprint	Identity / Session Management
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
Previous IS	IS-IDENTITY-001
Previous IS	IS-IDENTITY-002
Previous IS	IS-IDENTITY-003
Previous IS	IS-IDENTITY-004
Previous IS	IS-IDENTITY-005
Previous IS	IS-IDENTITY-006
Previous IS	IS-IDENTITY-007
Next IS	IS-IDENTITY-009
API Contract	IS-IDENTITY-014
Persistence	IS-IDENTITY-015
Testing	IS-IDENTITY-016
Runtime	IS-IDENTITY-017

La trazabilidad sigue la regla del Blueprint de mantener la relación ESP → IS → Technical Tasks → Source Code, sin saltar etapas.

38. Architecture Model

La implementación debe respetar la separación:

Presentation
      ↓
API
      ↓
Application
      ↓
Domain
      ↓
Infrastructure

Para Session Management:

                    ┌──────────────────────┐
                    │      Identity API    │
                    └──────────┬───────────┘
                               ↓
                    ┌──────────────────────┐
                    │ Application Layer    │
                    │                      │
                    │ CreateSession        │
                    │ RevokeSession        │
                    │ ListSessions          │
                    └──────────┬───────────┘
                               ↓
                    ┌──────────────────────┐
                    │ Domain Layer         │
                    │                      │
                    │ Session              │
                    │ SessionService       │
                    │ SessionId            │
                    └──────────┬───────────┘
                               ↓
                    ┌──────────────────────┐
                    │ Infrastructure       │
                    │                      │
                    │ PostgreSQL            │
                    │ Redis                 │
                    │ Event Platform        │
                    └──────────────────────┘

Esto mantiene la arquitectura por capas definida para Identity y evita que Session Management quede acoplado directamente a HTTP o infraestructura.

39. Architectural Decisions Pending

Esta IS deja deliberadamente abiertos algunos parámetros que no están definidos normativamente en el Blueprint:

Session maximum lifetime.
Inactivity timeout.
Sliding session policy.
Maximum concurrent sessions.
Device limits.
Immediate access-token revocation strategy.
Exact session metadata.
Exact authentication-method enum.
Exact MFA status enum.
Política de limpieza de sesiones expiradas.

Estos puntos deben resolverse mediante decisiones arquitectónicas/configuración antes de cerrar la implementación.

40. Dependency Flow

La relación entre las historias queda:

IS-001 Identity Foundation
          │
          ├── IS-002 User
          │
          ├── IS-003 Organization
          │
          ├── IS-004 Membership
          │
          ├── IS-005 Authentication
          │          │
          │          ↓
          │     IS-008 Session
          │          │
          │          ↓
          │     IS-009 Refresh Token
          │
          ├── IS-006 Authorization
          │
          └── IS-007 Roles & Permissions

Y posteriormente:

IS-008 Session
      ↓
IS-009 Refresh Token
      ↓
IS-010 MFA
      ↓
IS-011 Security Policy
      ↓
IS-012 Audit
      ↓
IS-013 Domain Events & Outbox
      ↓
IS-014 API Contracts
      ↓
IS-015 Persistence
      ↓
IS-016 Testing
      ↓
IS-017 Runtime & Deployment
Estado de la especificación

IS-IDENTITY-008 queda definida como Draft — Critical.

El siguiente bloque lógico es:

IS-IDENTITY-009 — Refresh Token Lifecycle

Ahí debemos especificar con mucho detalle Refresh Token Rotation, Token Families, Reuse Detection, revocación, concurrencia, expiración y relación Session ↔ Refresh Token, porque es una de las piezas de seguridad más críticas de Identity.
