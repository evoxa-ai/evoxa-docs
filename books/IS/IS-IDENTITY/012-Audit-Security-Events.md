IS-IDENTITY-012 — Audit & Security Events

Estado: Draft
Prioridad: Critical
Dominio: Identity
Capability: Audit & Security Events
Dependencias: IS-IDENTITY-001 → IS-IDENTITY-011
Siguiente historia: IS-IDENTITY-013 — Domain Events & Outbox

1. Propósito

Definir el sistema de Audit & Security Events de EVOXA para registrar, preservar y permitir la trazabilidad de las operaciones relevantes de identidad y seguridad.

Esta capacidad debe permitir responder de forma confiable:

Quién hizo qué, cuándo, sobre qué recurso, dentro de qué organización, desde qué contexto, con qué resultado y bajo qué decisión de seguridad.

El sistema debe proporcionar una separación clara entre:

Audit Events
      +
Security Events
      +
Domain Events

No deben tratarse como conceptos idénticos.

Los Audit Events representan evidencia operacional y de seguridad que debe poder investigarse posteriormente.

Los Security Events representan hechos relevantes para seguridad y pueden desencadenar detección, respuesta o automatización.

Los Domain Events & Outbox serán especificados formalmente en IS-IDENTITY-013.

2. Business Goal

Como plataforma EVOXA, necesitamos una capacidad centralizada de auditoría que permita:

Investigar actividades de usuarios.
Detectar comportamientos sospechosos.
Registrar cambios de seguridad.
Mantener trazabilidad de autenticación.
Registrar autorización.
Registrar cambios de roles y permisos.
Registrar cambios de MFA.
Registrar sesiones.
Registrar Refresh Token security events.
Registrar cambios de Security Policies.
Cumplir requisitos de seguridad y compliance.
Correlacionar eventos distribuidos.
Proporcionar evidencia para incident response.

La auditoría no debe depender de logs de aplicación.

Application Logs
      ≠
Audit Trail
3. Principio Fundamental

El Audit System debe diseñarse como una capacidad de seguridad independiente de los logs técnicos.

Application
     │
     ├──────────────→ Application Logs
     │
     └──────────────→ Audit Event
                              │
                              ↓
                         Audit Store

Los logs sirven principalmente para debugging y observabilidad.

Los Audit Events sirven como registro de seguridad y trazabilidad.

4. Personas
Primary
Security Administrator
Organization Administrator
Admin
Audited Actors
Coach
Nutritionist
Physiotherapist
Team Manager
Admin
System Service
AI Agent, cuando posteriormente corresponda
5. User Journey
5.1 Operación normal
User
 ↓
Request
 ↓
Authentication
 ↓
Authorization
 ↓
Operation
 ↓
Audit Event
5.2 Operación rechazada
Request
 ↓
Authorization
 ↓
DENY
 ↓
Security Event
 ↓
Audit Event
5.3 Cambio de privilegios
Admin
 ↓
Assign Role
 ↓
Authorization
 ↓
Role Changed
 ↓
Audit Event
 ↓
Security Event
 ↓
Cache Invalidation
5.4 Incidente de Refresh Token
Refresh Token Reuse
       ↓
Security Event
       ↓
Audit Event
       ↓
Token Family Revoked
       ↓
Session Revoked
6. Feature Pack

Esta IS cubre:

Audit
Audit Event.
Actor.
Resource.
Operation.
Result.
Organization.
Timestamp.
Correlation.
Traceability.
Security Events
Authentication events.
Authorization events.
Session events.
Refresh Token events.
MFA events.
Role/Permission events.
Policy events.
Security
Immutable audit records.
Tenant isolation.
Sensitive-data protection.
Secret exclusion.
Correlation.
Retention.
Operations
Search.
Filtering.
Investigation.
Metrics.
Monitoring.
Alert integration.
7. Scope
In Scope
AuditEvent.
SecurityEvent.
Audit Event schema.
Security Event schema.
Actor identification.
Resource identification.
Tenant context.
Correlation.
Audit persistence.
Event categorization.
Security severity.
Immutable records.
Retention.
Access control.
Observability.
Integration with Identity capabilities.
Out of Scope

No se implementa aquí:

Full Event Platform.
Transactional Outbox.
Message broker implementation.
SIEM específico.
Security Policy Engine.
Authentication.
MFA.
Session.
Refresh Token.
Role Management.
8. AuditEvent

Entidad conceptual:

AuditEvent
├── id
├── event_type
├── event_version
├── timestamp
├── tenant_id
├── actor
├── resource
├── operation
├── result
├── reason
├── correlation_id
├── causation_id
├── trace_id
├── source
└── metadata
9. Actor

El actor representa quién o qué produjo la operación.

Conceptualmente:

Actor
├── type
├── id
└── authentication_context

Tipos posibles:

USER
SERVICE
SYSTEM
AI_AGENT

El catálogo definitivo debe ser normalizado posteriormente.

10. Resource

Representa el objeto afectado.

Ejemplos:

User
Organization
Membership
Role
Permission
Session
RefreshToken
MFAFactor
SecurityPolicy

Conceptualmente:

Resource
├── type
├── id
└── organization_id

No debe incluirse información sensible innecesaria.

11. Operation

Representa la acción realizada.

Ejemplos:

CREATE
READ
UPDATE
DELETE
LOGIN
LOGOUT
REVOKE
ACTIVATE
SUSPEND
ASSIGN
REMOVE
VERIFY
REFRESH

El catálogo definitivo deberá ser centralizado.

12. Result

El resultado debe ser explícito.

SUCCESS
FAILURE
DENIED
CHALLENGED

Puede ampliarse posteriormente.

13. Security Severity

Los Security Events deben soportar niveles de severidad.

Conceptualmente:

INFO
LOW
MEDIUM
HIGH
CRITICAL

Ejemplo:

Successful login
→ INFO

Repeated MFA failures
→ MEDIUM

Privilege escalation attempt
→ HIGH

Refresh Token reuse
→ CRITICAL

La clasificación definitiva debe ser gobernada por la arquitectura de seguridad.

14. Security Event

Un Security Event representa un hecho relevante para seguridad.

Ejemplos:

UserAuthenticationFailed
UserAuthenticated
AuthorizationDenied
TenantAccessDenied
PermissionDenied
PolicyViolationDetected
PrivilegeEscalationDetected
RefreshTokenReuseDetected
MFAChallengeFailed
SessionRevoked
15. Audit vs Security Event
Característica	Audit Event	Security Event
Trazabilidad	Sí	Sí
Investigación	Sí	Sí
Seguridad	Sí	Sí
Puede generar alerta	Posible	Sí
Debe preservarse	Sí	Sí
Puede ser operacional	Sí	Principalmente seguridad
Trigger automático	Opcional	Frecuente

Un mismo hecho puede generar ambos:

RefreshTokenReuseDetected
       │
       ├── Security Event
       │
       └── Audit Event
16. Event Categories

El sistema debe poder categorizar eventos.

Authentication
LOGIN_SUCCESS
LOGIN_FAILURE
LOGOUT
Authorization
AUTHORIZATION_ALLOWED
AUTHORIZATION_DENIED
TENANT_ACCESS_DENIED
Session
SESSION_CREATED
SESSION_REVOKED
SESSION_EXPIRED
Refresh Token
TOKEN_CREATED
TOKEN_ROTATED
TOKEN_REVOKED
TOKEN_REUSE_DETECTED
MFA
MFA_ENROLLED
MFA_VERIFIED
MFA_FAILED
MFA_REVOKED
Authorization Management
ROLE_CREATED
ROLE_CHANGED
ROLE_ASSIGNED
ROLE_REMOVED
PERMISSION_ASSIGNED
PERMISSION_REVOKED
Policy
POLICY_CREATED
POLICY_CHANGED
POLICY_ACTIVATED
POLICY_SUSPENDED
17. Immutable Audit

Una vez creado un Audit Event:

CREATE
  ↓
IMMUTABLE

No debe existir:

UPDATE AUDIT EVENT

ni:

DELETE AUDIT EVENT

desde las operaciones normales de la aplicación.

Las correcciones deben realizarse mediante nuevos eventos, manteniendo trazabilidad.

18. Audit Integrity

El sistema debe estar diseñado para evitar:

Alteración silenciosa.
Eliminación silenciosa.
Manipulación por usuarios auditados.
Cross-tenant visibility.
Inserción de eventos falsos por clientes.

El actor no debe poder determinar arbitrariamente el contenido de seguridad del Audit Event.

19. Tenant Isolation

Cada evento que pertenezca a una organización debe incluir:

tenant_id

Un usuario de:

Organization A

no puede consultar:

Organization B

sin autorización explícita.

20. System Events

No todos los eventos tienen un actor humano.

Ejemplo:

Actor:
type = SYSTEM
id = identity-service

Esto permite registrar:

Expiraciones automáticas.
Revocaciones automáticas.
Policy actions.
Cleanup.
Security response.
21. AI Events

El modelo debe prepararse para eventos producidos por AI Agents.

Ejemplo:

Actor:
type = AI_AGENT
id = agent-xyz

La auditoría debe permitir responder:

Qué agente actuó.
En nombre de quién.
Qué autorización tenía.
Qué operación realizó.
Qué policy permitió la acción.

Esto será fundamental para la evolución AI-Native de EVOXA.

22. Correlation

Cada operación debe poder correlacionarse.

Conceptualmente:

correlation_id
causation_id
trace_id

Ejemplo:

Login Request
     │
     ├── UserAuthenticated
     ├── SessionCreated
     ├── RefreshTokenCreated
     └── AuditEvent

Todos pueden compartir:

correlation_id = C123
23. Correlation vs Causation
correlation_id

Relaciona eventos pertenecientes al mismo flujo.

causation_id

Indica qué evento/acción originó el evento actual.

Ejemplo:

AuthorizationDenied
        ↓
PolicyViolationDetected

El segundo evento puede tener:

causation_id = AuthorizationDenied.event_id
24. Trace ID

El trace_id permite relacionar:

API
 ↓
Application
 ↓
Domain
 ↓
Database
 ↓
Event Platform

con la misma operación distribuida.

25. Audit Access

Consultar Audit Events también es una operación protegida.

User
 ↓
Authorization
 ↓
audit.read
 ↓
Policy
 ↓
ALLOW / DENY

No todos los usuarios pueden acceder a toda la auditoría.

26. Audit Permissions

El modelo de permisos debe permitir diferenciar:

audit.read
audit.read_sensitive
audit.export
audit.admin

Los nombres son conceptuales y deberán integrarse al catálogo de permisos definitivo.

27. Sensitive Audit Data

La auditoría debe aplicar minimización.

No almacenar:

password
password_hash
access_token
refresh_token
TOTP secret
OTP value
private_key
recovery_code

Tampoco debe registrarse información personal innecesaria.

28. Health Data Protection

Dado que EVOXA eventualmente manejará información relacionada con usuarios, entrenamiento, nutrición y otras áreas potencialmente sensibles:

El Audit Event debe registrar referencias a recursos y operaciones, no copiar indiscriminadamente el contenido completo del recurso auditado.

Ejemplo preferido:

resource:
  type: training_plan
  id: TP-123

operation:
  UPDATE

en lugar de copiar todo el contenido del plan.

29. Audit Metadata

Metadata puede incluir:

ip
user_agent
device_id
request_id
service
environment
region

cuando sea necesario y permitido.

Debe aplicarse minimización y política de privacidad.

30. Retention

Los Audit Events pueden requerir una retención superior a los logs normales.

Conceptualmente:

Application Logs
     ↓
Short/Medium Retention

Audit Events
     ↓
Longer Security Retention

El período exacto no debe inventarse en esta IS.

Debe definirse según:

Compliance.
Política organizacional.
Requerimientos legales.
Seguridad.
Capacidad de almacenamiento.
31. Audit Search

El sistema debe permitir posteriormente búsquedas por:

event_type
actor_id
organization_id
resource_type
resource_id
operation
result
severity
timestamp
correlation_id
trace_id

La API exacta será definida en IS-014.

32. Security Event Processing

Los Security Events pueden activar respuestas posteriores:

Security Event
      ↓
Detection
      ↓
Policy / Security Automation
      ↓
Response

Ejemplo:

RefreshTokenReuseDetected
        ↓
Revoke Family
        ↓
Revoke Session
        ↓
Alert Security

El procesamiento asíncrono completo será desarrollado en IS-013.

33. Domain Events vs Audit Events

Debe mantenerse esta separación:

Domain Event
    =
Hecho ocurrido en el dominio

Audit Event
    =
Registro de trazabilidad de una operación

Security Event
    =
Hecho relevante para seguridad

Un Domain Event puede generar un Audit Event, pero no son necesariamente equivalentes.

34. AuditService

Debe existir un servicio conceptual:

AuditService

Responsable de:

Crear Audit Events.
Validar estructura.
Sanitizar datos.
Garantizar tenant context.
Correlacionar eventos.
Persistir.
Integrarse con eventos de seguridad.

No debe depender directamente de HTTP.

35. SecurityEventService

Puede existir un servicio separado:

SecurityEventService

Responsable de:

Clasificar eventos.
Determinar severidad.
Publicar Security Events.
Integrar mecanismos de respuesta.
Generar métricas.
36. Repository Contracts

Conceptualmente:

AuditEventRepository
SecurityEventRepository

Operaciones:

append()
get_by_id()
search()
count()

El método append() debe enfatizar que la escritura es append-only.

37. Persistence

PostgreSQL puede actuar como almacenamiento transaccional del Audit Trail.

Conceptualmente:

audit_events
------------
id
event_type
event_version
timestamp
tenant_id
actor_type
actor_id
resource_type
resource_id
operation
result
severity
reason
correlation_id
causation_id
trace_id
source
metadata
created_at

La estructura final será especificada en IS-IDENTITY-015.

38. Index Strategy

Deben estudiarse índices sobre:

tenant_id
timestamp
actor_id
event_type
resource_type
resource_id
correlation_id
trace_id
severity

Debe evitarse crear índices innecesarios que impacten el volumen de escritura.

39. API Conceptual

Los contratos definitivos corresponden a IS-014.

Search Audit
GET /api/v1/identity/audit-events
Audit Detail
GET /api/v1/identity/audit-events/{event_id}
Security Events
GET /api/v1/identity/security-events
Security Event Detail
GET /api/v1/identity/security-events/{event_id}

No deben existir endpoints normales para modificar Audit Events.

40. Security Rules
BR-012-001 — Append Only

Los Audit Events son append-only.

BR-012-002 — Immutability

Un evento persistido no debe modificarse.

BR-012-003 — No Secrets

Nunca almacenar secretos.

BR-012-004 — Tenant Isolation

Los eventos deben respetar tenant isolation.

BR-012-005 — Authorization

La lectura de auditoría requiere autorización.

BR-012-006 — Correlation

Los eventos relevantes deben ser correlacionables.

BR-012-007 — Accurate Actor

El actor debe provenir del contexto de identidad confiable.

BR-012-008 — System Actor

Operaciones automáticas deben identificar al sistema responsable.

BR-012-009 — AI Actor

Las acciones de AI Agents deben poder atribuirse al agente y al contexto de autorización.

BR-012-010 — Sensitive Data Minimization

No copiar innecesariamente contenido sensible al Audit Trail.

BR-012-011 — Security Events

Los eventos de seguridad críticos deben poder desencadenar mecanismos de respuesta.

41. Acceptance Criteria
AC-012-01 — Audit Creation

Given una operación auditable
When finaliza
Then debe generarse un Audit Event apropiado.

AC-012-02 — Actor

Given una operación ejecutada por un usuario
When se genera el Audit Event
Then debe identificarse el actor.

AC-012-03 — Tenant

Given una operación dentro de una organización
When se registra
Then debe quedar asociado el tenant_id.

AC-012-04 — Immutable

Given un Audit Event persistido
When un usuario intenta modificarlo
Then la operación debe ser rechazada.

AC-012-05 — Secret Protection

Given cualquier evento de Identity
When se persiste
Then no debe contener password, tokens, MFA secrets ni private keys.

AC-012-06 — Security Event

Given un evento de seguridad relevante
When ocurre
Then debe generarse el Security Event correspondiente.

AC-012-07 — Correlation

Given varias operaciones pertenecientes a un mismo flujo
When se registran
Then deben poder correlacionarse mediante correlation_id.

AC-012-08 — Causation

Given un Security Event derivado de otra operación
When se registra
Then debe poder identificarse su causation cuando corresponda.

AC-012-09 — Tenant Access

Given un usuario perteneciente a Organization A
When solicita Audit Events de Organization B
Then la consulta debe ser rechazada.

AC-012-10 — System Event

Given una operación automática
When se registra
Then debe identificarse un actor de tipo SYSTEM.

AC-012-11 — AI Event

Given una acción ejecutada por un AI Agent
When se audita
Then debe poder determinarse el agente y su contexto de autorización.

AC-012-12 — Audit Search

Given un usuario con permisos de auditoría
When realiza una búsqueda
Then debe poder filtrar eventos autorizados por los criterios soportados.

42. Technical Tasks
Domain

TSK-012-01 — Crear AuditEvent.

TSK-012-02 — Crear SecurityEvent.

TSK-012-03 — Crear Actor model.

TSK-012-04 — Crear Resource model.

TSK-012-05 — Crear AuditContext.

TSK-012-06 — Definir EventType.

TSK-012-07 — Definir Operation.

TSK-012-08 — Definir Result.

TSK-012-09 — Definir SecuritySeverity.

Services

TSK-012-10 — Crear AuditService.

TSK-012-11 — Crear SecurityEventService.

TSK-012-12 — Implementar event sanitization.

TSK-012-13 — Implementar actor resolution.

TSK-012-14 — Implementar correlation.

Persistence

TSK-012-15 — Crear Audit Repository.

TSK-012-16 — Crear Security Event Repository.

TSK-012-17 — Crear persistence models.

TSK-012-18 — Crear índices.

TSK-012-19 — Implementar append-only semantics.

TSK-012-20 — Implementar retention strategy.

Integration

TSK-012-21 — Integrar Authentication.

TSK-012-22 — Integrar Authorization.

TSK-012-23 — Integrar User.

TSK-012-24 — Integrar Organization.

TSK-012-25 — Integrar Membership.

TSK-012-26 — Integrar Role/Permission.

TSK-012-27 — Integrar Session.

TSK-012-28 — Integrar Refresh Token.

TSK-012-29 — Integrar MFA.

TSK-012-30 — Integrar Security Policy.

Events

TSK-012-31 — Definir Security Event Catalog.

TSK-012-32 — Implementar Security Event generation.

TSK-012-33 — Implementar event severity.

TSK-012-34 — Preparar integración con Outbox.

API

TSK-012-35 — Preparar Audit API.

TSK-012-36 — Preparar Security Events API.

TSK-012-37 — Implementar authorization para lectura.

Observability

TSK-012-38 — Metrics.

TSK-012-39 — Logging.

TSK-012-40 — Tracing.

Security

TSK-012-41 — Secret filtering.

TSK-012-42 — PII minimization.

TSK-012-43 — Tenant isolation.

TSK-012-44 — Audit access controls.

TSK-012-45 — Tamper protection.

Testing

TSK-012-46 — Unit tests.

TSK-012-47 — Integration tests.

TSK-012-48 — Security tests.

TSK-012-49 — Tenant isolation tests.

TSK-012-50 — Immutability tests.

TSK-012-51 — Secret leakage tests.

TSK-012-52 — Correlation tests.

TSK-012-53 — Retention tests.

TSK-012-54 — AI audit tests.

43. Testing Strategy
Unit

Cubrir:

Event creation.
Event classification.
Actor.
Resource.
Severity.
Sanitization.
Correlation.
Integration

Validar:

Authentication
Authorization
Session
MFA
Refresh Token
Policy
Audit
Security

Especial atención a:

Audit Tampering
Cross-Tenant Access
Secret Leakage
Unauthorized Audit Read
False Actor Attribution
Privilege Escalation
AI Attribution
44. Observability
Metrics
identity.audit.events
identity.audit.security_events
identity.audit.write_failures
identity.audit.search
identity.audit.access_denied
identity.security.critical_events
identity.security.policy_violations
identity.security.privilege_escalation

Debe poder observarse especialmente:

critical security events
audit write failures
unusual denial patterns
45. Audit Failure Strategy

Este punto requiere especial atención.

Si una operación de seguridad ocurre pero el Audit Event no puede persistirse:

Security Operation
       ↓
Audit Write Failure

La plataforma no debe simplemente ignorar el error.

Debe definirse una estrategia de durabilidad y atomicidad.

Esto conecta directamente con:

IS-IDENTITY-013 — Domain Events & Outbox.

La solución debe permitir evitar:

Business operation SUCCESS
+
Audit LOST

cuando la auditoría sea obligatoria.

46. Transactional Consistency

Para operaciones críticas puede requerirse:

Business Transaction
       +
Audit/Event Intent
       ↓
Atomic Commit

La implementación exacta se definirá con Transactional Outbox.

47. Performance

Audit no debe bloquear innecesariamente las operaciones normales.

Se deberá estudiar:

Escritura asíncrona.
Transactional Outbox.
Batching.
Index strategy.
Partitioning.
Retention.
Archival.

Pero las operaciones que requieran garantía transaccional deben mantener dicha garantía.

48. Definition of Done
 AuditEvent implementado.
 SecurityEvent implementado.
 Actor model implementado.
 Resource model implementado.
 Correlation implementado.
 Severity implementada.
 AuditService implementado.
 SecurityEventService implementado.
 Append-only implementado.
 Immutability implementada.
 Secret filtering implementado.
 PII minimization implementada.
 Tenant isolation implementado.
 Audit access control implementado.
 Authentication integrado.
 Authorization integrado.
 User integrado.
 Organization integrado.
 Membership integrado.
 Roles/Permissions integrados.
 Session integrado.
 Refresh Token integrado.
 MFA integrado.
 Security Policy integrado.
 PostgreSQL persistence implementada.
 Índices definidos.
 Retention strategy definida.
 Events preparados para Outbox.
 Metrics implementadas.
 Logging seguro implementado.
 Tracing implementado.
 Unit tests implementados.
 Integration tests implementados.
 Security tests implementados.
 Cross-tenant tests implementados.
 Immutability tests implementados.
 Secret leakage tests implementados.
 AI audit tests implementados.
 Documentación actualizada.
49. Traceability
Elemento	Referencia
Blueprint	Identity / Audit / Security
ESP	ESP-0001 — Identity
ADR	ADR-IDENTITY-005
ADR	ADR-IDENTITY-006
ADR	ADR-IDENTITY-007
ADR	ADR-IDENTITY-008
ADR	ADR-IDENTITY-010
Previous IS	IS-IDENTITY-005 — Authentication
Previous IS	IS-IDENTITY-006 — Authorization
Previous IS	IS-IDENTITY-007 — Roles & Permissions
Previous IS	IS-IDENTITY-008 — Session
Previous IS	IS-IDENTITY-009 — Refresh Token
Previous IS	IS-IDENTITY-010 — MFA
Previous IS	IS-IDENTITY-011 — Security Policy
Next IS	IS-IDENTITY-013 — Domain Events & Outbox
API	IS-IDENTITY-014
Persistence	IS-IDENTITY-015
Testing	IS-IDENTITY-016
Runtime	IS-IDENTITY-017

La trazabilidad sigue el modelo ESP → IS → Technical Tasks → Source Code establecido por el Blueprint.

50. Architecture Model
                         ┌──────────────────┐
                         │     Request      │
                         └────────┬─────────┘
                                  ↓
                     ┌────────────────────────┐
                     │ Authentication         │
                     └───────────┬────────────┘
                                 ↓
                     ┌────────────────────────┐
                     │ Authorization / Policy  │
                     └───────────┬────────────┘
                                 ↓
                         Business Operation
                                 │
                    ┌────────────┴────────────┐
                    ↓                         ↓
             Audit Event                Security Event
                    │                         │
                    └────────────┬────────────┘
                                 ↓
                         Transactional
                            Outbox
                                 ↓
                         Event Platform
                                 ↓
               ┌─────────────────┼─────────────────┐
               ↓                 ↓                 ↓
            Audit Store      Detection          Automation

La parte de Transactional Outbox + Event Platform se desarrollará formalmente en la siguiente IS.

51. Security Architecture

El modelo acumulado queda:

                         USER
                           │
                           ↓
                    AUTHENTICATION
                           │
                           ↓
                         MFA
                           │
                           ↓
                        SESSION
                           │
                           ↓
                    AUTHORIZATION
                           │
                           ↓
                   POLICY ENGINE
                           │
                 ┌─────────┴─────────┐
                 ↓                   ↓
              ALLOW                  DENY
                 │                   │
                 ↓                   ↓
           OPERATION          SECURITY EVENT
                 │                   │
                 └─────────┬─────────┘
                           ↓
                      AUDIT EVENT
                           │
                           ↓
                   TRANSACTIONAL
                      OUTBOX

Esto proporciona la trazabilidad completa desde una acción del usuario hasta su consecuencia de seguridad.

52. AI-Native Audit

Para EVOXA será especialmente importante que las acciones realizadas por agentes de IA sean auditables.

Conceptualmente:

User
 ↓
AI Agent
 ↓
Authorization
 ↓
Policy Engine
 ↓
ALLOW
 ↓
Action
 ↓
Audit Event

El Audit Event debería permitir reconstruir:

Human Actor
     ↓
AI Agent
     ↓
Authorization Context
     ↓
Policy
     ↓
Action
     ↓
Result

Esto será fundamental cuando EVOXA avance hacia AI Agents y Autonomous Operations.

53. Architectural Decisions Pending

Antes de pasar esta IS a Approved, deberán definirse:

Audit storage definitivo.
Retention periods.
Archival strategy.
Partitioning strategy.
Security Event catalog definitivo.
Severity taxonomy definitiva.
Audit immutability mechanism.
Tamper-evidence strategy.
SIEM integration.
Alerting integration.
Audit write failure behavior.
Transactional consistency.
Event delivery guarantees.
Search architecture.
Cross-tenant audit administration.
PII masking/anonymization.
Compliance-specific retention.
AI Agent attribution model.
Audit export format.
Legal hold requirements.
54. Dependency Flow

La secuencia de Identity queda:

001 Identity Foundation
        ↓
002 User
        ↓
003 Organization
        ↓
004 Membership
        ↓
005 Authentication
        ↓
006 Authorization
        ↓
007 Roles & Permissions
        ↓
008 Session
        ↓
009 Refresh Token
        ↓
010 MFA
        ↓
011 Security Policy
        ↓
012 Audit & Security Events
        ↓
013 Domain Events & Outbox
        ↓
014 Identity API Contracts
        ↓
015 Identity Persistence
        ↓
016 Identity Testing & Quality
        ↓
017 Identity Runtime & Deployment
55. Estado

IS-IDENTITY-012 — Audit & Security Events queda definida como Draft — Critical.

Con esta historia queda establecida la trazabilidad de seguridad de Identity:

Authentication → Authorization → Policy → Operation → Security Event → Audit Event

y queda preparada la transición hacia:

IS-IDENTITY-013 — Domain Events & Outbox

Esta siguiente IS será especialmente importante porque conectará los eventos de Identity con el modelo Event-Driven de EVOXA, definiendo Transactional Outbox, Event Envelope, versionado, idempotencia, at-least-once delivery, retries, DLQ, replay, correlation/causation y garantías de publicación, sin perder la consistencia entre PostgreSQL y la plataforma de eventos.
