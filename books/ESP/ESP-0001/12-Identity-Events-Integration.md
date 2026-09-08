ESP-0001/12 — Identity Events & Integration

Document ID: ESP-0001/12
Module: Identity
Version: 1.1.0
Status: Draft — Consolidated
Parent Specification: ESP-0001 — Identity Engineering Specification
Target Repository: evoxa-platform
Target Path: apps/api/app/domains/identity/

1. Purpose

Este documento define cómo Identity:

publica eventos;
consume eventos;
integra otros dominios;
utiliza Event Platform;
garantiza entrega At-Least-Once;
maneja retries;
utiliza DLQ;
permite replay controlado;
mantiene idempotencia;
preserva tenant isolation;
mantiene trazabilidad;
versiona eventos;
protege información sensible;
utiliza Transactional Outbox;
integra proveedores externos;
expone webhooks;
mantiene contratos de integración.

Identity no debe convertirse en un dominio aislado.

Los cambios relevantes deben poder propagarse hacia:

Human Digital Twin
Training
Nutrition
Recovery
Recommendation
AI Orchestrator
Notifications
Analytics
Administration
Security
Observability
Integrations

El documento original ya establece esta intención de integración Event-Driven.

2. Event Architecture Principles

Identity Events deberá cumplir:

Events are facts, not commands.
Immutability.
Explicit versioning.
At-Least-Once delivery.
Idempotent consumers.
Tenant awareness.
Correlation.
Causation.
Scoped ordering.
Retry.
DLQ.
Controlled replay.
Schema governance.
Security.
Privacy.
Observability.
No secrets in events.
Transactional consistency.
Explicit ownership.
Evolvability.
3. Identity Event Architecture

Modelo:

                  Identity
                     │
              State Change
                     │
                     ▼
              Domain Event
                     │
                     ▼
             Transactional
                Outbox
                     │
                     ▼
              Event Publisher
                     │
                     ▼
              Event Platform
                     │
        ┌────────────┼────────────┐
        ▼            ▼            ▼
   Notifications  Analytics    Security
        │
        ├───────────────┐
        ▼               ▼
  Other Domains      AI Platform

El evento se produce como consecuencia de un cambio real de estado.

No deberá generarse un evento de negocio afirmando que algo ocurrió si la transacción correspondiente no quedó confirmada.

4. Domain Event vs Integration Event

Se mantiene la distinción del documento original:

Domain Event

Representa un hecho dentro de Identity.

UserActivated
Integration Event

Representa el contrato publicado hacia otros módulos:

identity.user.activated.v1

Esta separación permite que el modelo interno evolucione sin obligar a los consumidores externos a conocer las estructuras internas de Identity.

5. Event Ownership

Cada evento debe tener un owner.

Ejemplo:

Identity
   │
   └── UserCreated

Identity es responsable de:

significado;
schema;
versionado;
compatibilidad;
lifecycle;
documentación;
seguridad del evento.

El consumidor es responsable de su propia reacción.

6. Event Envelope

Todos los Integration Events deberán utilizar un envelope estándar.

{
  "event_id": "...",
  "event_type": "identity.user.created",
  "event_version": 1,
  "timestamp": "...",
  "tenant_id": "...",
  "actor": {
    "type": "USER",
    "id": "..."
  },
  "resource": {
    "type": "User",
    "id": "..."
  },
  "payload": {},
  "metadata": {
    "correlation_id": "...",
    "causation_id": "...",
    "trace_id": "..."
  }
}

Los campos exactos se congelarán en el Event Contract definitivo.

7. Event ID

Todo evento tendrá:

event_id

Debe ser único.

Se utilizará para:

deduplicación;
trazabilidad;
auditoría;
debugging;
replay control;
correlación.

Un consumidor puede utilizar event_id como una de sus claves de deduplicación.

8. Event Type

Los nombres deben representar hechos de negocio.

Correcto:

identity.user.created
identity.user.activated
identity.membership.removed
identity.role.assigned

Incorrecto:

identity.user.table.updated
identity.database.row.changed

El evento no debe revelar la implementación de persistencia.

9. Event Version

Todos los eventos son versionados.

Ejemplo:

identity.user.created.v1

Una evolución incompatible podrá requerir:

identity.user.created.v2

La versión del evento no es lo mismo que:

schema version;
application version;
policy version;
runtime version.

Cada uno deberá mantenerse identificable cuando sea necesario para reproducibilidad.

10. Timestamp

Todo evento debe incluir timestamp.

Se utilizará una representación temporal consistente, preferentemente UTC.

Debe distinguirse entre:

occurred_at

y timestamps de infraestructura como:

published_at
processed_at

El momento en que ocurrió el hecho no debe confundirse con el momento en que fue publicado o procesado.

11. Tenant ID

Los eventos organization-scoped deberán incluir:

tenant_id

El consumidor deberá validar que el evento pertenece al tenant/contexto que está procesando.

Nunca debe asumirse:

event tenant = consumer tenant

sin validación.

Cross-tenant processing deberá fallar de manera segura.

12. Actor

Los eventos deberán identificar el origen cuando corresponda.

Tipos conceptuales:

USER
SERVICE
SYSTEM
AI_AGENT

Puede existir ADMIN como clasificación de actor a nivel de contrato, pero cuando un administrador actúa como usuario autenticado debe conservarse la identidad suficiente para trazabilidad.

Ejemplo:

{
  "type": "USER",
  "id": "..."
}

Para automatización:

{
  "type": "SYSTEM",
  "id": "identity-service"
}
13. AI Actor

Cuando una operación sea realizada por un AI Agent, el evento deberá conservar esa atribución.

Ejemplo conceptual:

{
  "type": "AI_AGENT",
  "id": "..."
}

No se debe ocultar la participación de AI detrás de un actor genérico SYSTEM.

Esto permite:

auditoría;
seguridad;
debugging;
análisis de comportamiento;
governance de AI.
14. Resource

Los eventos deberán identificar el recurso afectado.

{
  "type": "Membership",
  "id": "..."
}

Esto permite relacionar:

Event
 ↓
Resource
 ↓
Audit
 ↓
Security Investigation
15. Correlation ID

Los eventos deberán conservar:

correlation_id

Ejemplo:

POST /users
      │
      ├── correlation_id
      │
      ├── UserCreated
      ├── MembershipCreated
      └── NotificationRequested

Esto permite reconstruir una operación distribuida.

16. Causation ID

Cuando un evento origine otro evento:

causation_id

deberá permitir conservar la relación causal.

Ejemplo:

UserSuspended
      │
      ▼
SessionsRevoked
      │
      ▼
RefreshTokensRevoked

Cada evento posterior puede identificar el evento que provocó su creación.

17. Event Catalog

El catálogo inicial se mantiene como base del documento original, pero se considera un catálogo gobernado y evolutivo.

User
UserCreated
UserUpdated
UserActivated
UserSuspended
UserDisabled
Organization
OrganizationCreated
OrganizationUpdated
OrganizationSuspended
OrganizationReactivated
Membership
MembershipCreated
MembershipActivated
MembershipSuspended
MembershipRemoved
MembershipRoleChanged
Roles
RoleCreated
RoleUpdated
RoleDeleted
RoleAssigned
RoleRemoved
Permissions
PermissionCreated
PermissionUpdated
PermissionAssigned
PermissionRevoked
Authentication
UserAuthenticated
UserAuthenticationFailed
UserLoggedOut
Sessions
SessionCreated
SessionRevoked
SessionExpired
Tokens
RefreshTokenRotated
RefreshTokenRevoked
TokenReuseDetected
MFA
MFAFactorCreated
MFAFactorActivated
MFAFactorRevoked
MFAChallengeVerified
MFAChallengeFailed
MFARecoveryCompleted
Security Policy
SecurityPolicyCreated
SecurityPolicyUpdated
SecurityPolicyActivated
SecurityPolicyDeactivated
SecurityPolicyViolation

El catálogo definitivo deberá evolucionar durante las Implementation Stories sin romper consumidores existentes.

18. Security Event Stream

No todos los eventos deben distribuirse de la misma manera.

Identity podrá separar:

Identity Business Integration Stream

de:

Security Event Stream

Ejemplo:

UserCreated
    ↓
Notifications / Analytics

mientras:

TokenReuseDetected
    ↓
Security Operations
    ↓
Risk Engine
    ↓
Observability

Esto reduce exposición innecesaria y permite diferentes controles de acceso.

19. Sensitive Event Filtering

Puede existir una separación entre:

Internal Security Event

y:

External Integration Event

El evento interno puede contener información adicional necesaria para Security Operations.

El evento externo debe contener solamente la información necesaria para el consumidor.

20. Payload Design

Los payloads deben ser:

mínimos;
semánticos;
estables;
versionables;
tenant-aware;
libres de secretos.

Ejemplo:

{
  "user_id": "...",
  "organization_id": "...",
  "status": "ACTIVE"
}

No:

{
  "password": "...",
  "password_hash": "...",
  "refresh_token": "...",
  "mfa_secret": "..."
}
21. Security Data Prohibition

Nunca deberán incluirse:

passwords;
password hashes;
refresh tokens;
access tokens;
API secrets;
MFA secrets;
OTP;
recovery codes;
private keys;
KMS secrets;
credentials de proveedores.

Esto aplica a:

payload;
metadata;
headers persistidos;
retry records;
DLQ;
telemetry.
22. Transactional Outbox

Aquí realizamos una actualización importante respecto del documento original.

Transactional Outbox queda establecido como arquitectura obligatoria para los cambios transaccionales de Identity que publiquen eventos.

El patrón será:

                PostgreSQL Transaction
                       │
             ┌─────────┴─────────┐
             │                   │
             ▼                   ▼
       Identity State       Outbox Event
             │                   │
             └─────────┬─────────┘
                       │
                    COMMIT
                       │
                       ▼
                Outbox Publisher
                       │
                       ▼
                 Event Platform

Esto evita el problema:

DB COMMIT
   +
EVENT LOST
23. Outbox Guarantees

El Outbox deberá proporcionar:

transactional consistency;
retry;
publication tracking;
idempotency;
observability;
failure recovery.

El publisher puede publicar un mismo evento más de una vez.

Por ello:

At-Least-Once

es el modelo esperado.

24. Event Publication

El flujo será:

Domain Operation
      ↓
Domain Event
      ↓
Outbox
      ↓
Publisher
      ↓
Event Platform

El Domain Event no debe publicarse directamente desde el controller antes de que la transacción de negocio sea confirmada.

25. Event Delivery

Identity utilizará:

At-Least-Once Delivery

No se asumirá Exactly-Once.

Consecuencia:

Event A
   ↓
Delivery 1
   ↓
Delivery 2

debe producir el mismo resultado lógico que una única entrega.

26. Consumer Idempotency

Cada consumer debe implementar deduplicación/idempotencia.

Posibles mecanismos:

event_id

o una clave idempotente basada en:

event_type
+
aggregate/resource
+
version

La estrategia exacta pertenece al consumer y a las reglas generales de Event Platform.

27. Scoped Ordering

No existe un orden global de todos los eventos.

El orden solamente se garantiza donde el dominio lo requiera.

Ejemplo:

UserCreated
      ↓
UserActivated
      ↓
UserSuspended

Para otro usuario:

UserCreated(B)

puede procesarse independientemente.

Esto permite escalabilidad horizontal.

28. Partitioning

Cuando Event Platform requiera particionado, el partition key deberá mantener juntos los eventos cuyo orden sea relevante.

Ejemplos conceptuales:

user_id
membership_id
organization_id
session_id

La elección concreta depende del stream/event contract.

No se utilizará una partición global única simplemente para obtener ordering.

29. Event Retry

Los errores transitorios deberán poder reintentarse.

Attempt 1
   ↓
Failure
   ↓
Retry
   ↓
Attempt 2
   ↓
Retry
   ↓
Attempt 3
   ↓
Success

La estrategia exacta de:

cantidad de retries;
backoff;
jitter;
maximum retry duration;

será parte de la configuración de Event Platform.

30. Permanent vs Transient Failure

Los errores deberán clasificarse.

Transient

Ejemplos:

timeout;
temporary unavailable;
connection failure;
temporary downstream failure.

→ Retry.

Permanent

Ejemplos:

invalid schema;
unsupported event version;
invalid mandatory field;
tenant contract violation.

→ DLQ o tratamiento equivalente.

31. Dead Letter Queue

Cuando un evento no pueda procesarse después del retry policy:

Event
 ↓
Retry
 ↓
Retry
 ↓
Retry
 ↓
DLQ

La DLQ debe permitir:

inspección;
diagnóstico;
clasificación;
corrección;
replay controlado;
auditoría.
32. DLQ Security

Una DLQ no debe convertirse en una copia insegura de los eventos.

Debe mantener las mismas restricciones:

encryption;
access control;
tenant isolation;
retention;
no secrets.
33. Replay

Identity debe soportar replay controlado cuando sea necesario para:

recovery;
reconstrucción;
analytics;
debugging;
integración;
validación.

Pero replay no significa volver a ejecutar ciegamente una operación de negocio.

34. Replay Safety

Un replay debe ser:

explícito;
autorizado;
auditable;
idempotente;
tenant-aware;
version-aware;
observable.

Debe evitar duplicar outcomes de negocio.

Ejemplo:

UserCreated

reprocesado no debe crear un segundo User.

35. Replay ≠ Source of Truth

El modelo continúa siendo:

PostgreSQL
     +
Events
     +
Audit

y no Event Sourcing completo por defecto.

Los eventos sirven para integración, propagación, reconstrucción de determinados procesos y recovery cuando corresponda.

36. Event Schema Registry

Todos los Integration Events deberán estar gobernados mediante un Schema Registry.

Conceptualmente:

Event Schema Registry
       │
       ├── identity.user.created.v1
       ├── identity.user.activated.v1
       ├── identity.membership.removed.v1
       ├── identity.role.assigned.v1
       └── identity.mfa.factor.activated.v1

El Registry será la referencia de:

schema;
version;
compatibility;
lifecycle.
37. Schema Compatibility

Los cambios deberán clasificarse.

BACKWARD_COMPATIBLE
FORWARD_COMPATIBLE
FULLY_COMPATIBLE
INCOMPATIBLE

Los cambios incompatibles requieren:

new event version

o una estrategia explícita de migration.

38. Event Schema Evolution

Ejemplo:

identity.user.created.v1

puede evolucionar a:

identity.user.created.v2

Los consumidores no deben ser forzados a actualizar instantáneamente.

Durante una transición podrán coexistir versiones.

39. Event Integration Contract

Cada evento deberá definir como mínimo:

Event ID
Event Type
Event Version
Owner
Tenant Scope
Actor
Resource
Payload Schema
Metadata Schema
Security Classification
Retention
Ordering Requirement
Idempotency Requirements
Compatibility Policy
40. Event Consumer Access

No todos los consumidores deben recibir todos los eventos.

Ejemplo:

Notifications
     │
     └── UserCreated

Analytics
     │
     └── Aggregated Authentication Events

Security
     │
     ├── AuthenticationFailed
     ├── TokenReuseDetected
     └── MFAFactorRevoked

Administration
     │
     ├── UserSuspended
     └── RoleChanged

El acceso deberá basarse en least privilege.

41. Event Subscription

Las subscriptions deberán ser:

explícitas;
autorizadas;
version-aware;
tenant-aware;
observables.

Un consumer no debe suscribirse indiscriminadamente a todos los streams de Identity.

42. Identity Consuming External Events

Identity también puede consumir eventos externos.

Ejemplos:

ExternalIdentityLinked
EnterpriseUserProvisioned
OrganizationProvisioningRequested

Pero la recepción de un evento externo no significa automáticamente que Identity deba ejecutar una modificación privilegiada.

Debe pasar por:

Event Validation
      ↓
Authentication / Service Identity
      ↓
Authorization
      ↓
Tenant Validation
      ↓
Policy
      ↓
Application Use Case
      ↓
Transaction
43. External Identity Providers

Identity puede integrar:

OIDC
SAML
Entra ID
Google
Okta
Auth0

La implementación concreta deberá estar encapsulada detrás de adapters.

External Provider
       ↓
Identity Adapter
       ↓
Identity Application
       ↓
Identity Domain

El dominio no debe depender directamente del SDK de un proveedor.

44. Federation Events

Eventos federados deberán evitar propagar tokens externos.

Por ejemplo:

ExternalIdentityLinked

puede incluir:

provider
external_subject
user_id

pero nunca:

external_access_token
external_refresh_token
client_secret
45. Provisioning Integration

La integración con provisioning deberá respetar:

idempotency;
at-least-once;
tenant isolation;
scoped ordering;
stale event handling;
source-of-truth rules.

Esto es especialmente importante para futuros flujos SCIM/JIT/provisioning empresarial.

46. Webhooks

Identity podrá utilizar webhooks para integraciones externas.

Identity Event
      ↓
Webhook Dispatcher
      ↓
External System

Los webhooks deberán incluir:

event ID;
event type;
version;
timestamp;
signature;
replay protection;
retry support.
47. Webhook Security

Los endpoints externos no deberán recibir eventos sin protección.

El mecanismo deberá soportar:

Signature
Timestamp
Replay Protection
Secret Rotation
Retry
Delivery Tracking

Los secrets utilizados para firmar webhooks deberán estar gestionados por el mecanismo central de secrets/KMS.

48. Webhook Delivery

Los webhooks deben asumir que el endpoint externo puede:

fallar;
responder lentamente;
devolver 5xx;
procesar y no responder;
recibir duplicados.

Por lo tanto:

Webhook = At-Least-Once

y el consumidor externo debe ser idempotente.

49. Event Security

Event Platform deberá proteger:

Authentication
Authorization
Encryption in Transit
Encryption at Rest
Tenant Isolation
Schema Validation
Access Control

Los consumers utilizarán identidades de servicio/workload.

50. Service Identity

Cada consumer sensible deberá tener una identidad explícita.

Identity Event
      ↓
Consumer
      ↓
Service Identity
      ↓
Authorized Subscription

No deberán utilizarse credenciales compartidas entre múltiples consumidores.

51. Tenant-Aware Event Processing

Un consumer deberá validar:

event.tenant_id

contra el contexto que corresponde.

Si el tenant es:

missing
invalid
unexpected
ambiguous

el procesamiento deberá fallar de forma segura.

52. Event Failure Isolation

Un error de un consumer no deberá impedir necesariamente la operación principal de Identity.

Por eso:

Identity Transaction
        ↓
Outbox
        ↓
Event Platform
        ↓
Consumer A
Consumer B
Consumer C

permite aislar fallos.

Un problema en Notifications no debe impedir que Identity cree correctamente un User.

53. Event Platform Failure

Si Event Platform está temporalmente indisponible:

Identity
   ↓
PostgreSQL
   ↓
Outbox

preserva el evento.

Cuando Event Platform vuelva:

Outbox
   ↓
Publisher
   ↓
Event Platform

continúa la publicación.

Esto es una de las razones principales por las que Transactional Outbox es obligatorio.

54. Outbox Failure

Si el publisher falla:

Outbox Event
     ↓
Retry

El evento permanece persistido hasta completar la política de publicación o ser enviado al mecanismo de failure handling correspondiente.

No se debe eliminar un evento simplemente porque el primer intento falló.

55. Event Ordering Failure

Los consumidores deberán tolerar:

duplicates;
delayed events;
retries;
out-of-order events cuando el scope no garantice ordering.

Para eventos versionados se deberá evitar regresar a una versión de estado anterior.

Ejemplo:

MembershipUpdated v5
        ↓
MembershipUpdated v4

El consumer no debe regresar el estado de v5 a v4.

56. Event Version Validation

El consumer debe validar:

event_type
event_version
schema_version

Un evento desconocido no debe procesarse como si fuera una versión conocida.

57. Security Event Priority

Eventos de seguridad críticos pueden requerir un tratamiento diferente al de eventos informativos.

Ejemplo:

TokenReuseDetected

puede requerir:

Security Operations
Risk Evaluation
Session/Token Response
Alerting
Audit

Mientras:

UserUpdated

puede tener una prioridad operativa inferior.

58. Cache Invalidation Events

Los cambios de autorización relevantes también producen propagación de invalidación.

Flujo:

Role Revoked
      ↓
Identity State Change
      ↓
Security Epoch / Version
      ↓
Outbox
      ↓
Event Platform
      ↓
Cache Invalidation Consumer
      ↓
L1 / L2 Invalidation

Importante:

el evento representa el hecho de que el estado cambió, no una instrucción específica de Redis.

Esto mantiene Event Platform desacoplada de la implementación del cache.

59. Cache Invalidation Scopes

Podrán existir scopes conceptuales:

PLATFORM
ORGANIZATION
MEMBERSHIP
ROLE
PERMISSION
RESOURCE
SESSION
DELEGATION
SECURITY

El consumer deberá invalidar de forma selectiva cuando sea posible.

60. Security Epoch Propagation

Cuando un cambio de seguridad invalide autorización previamente cacheada:

Authoritative Epoch
       >
Cached Epoch

el resultado cacheado será considerado obsoleto.

Nunca se debe utilizar un cache antiguo para crear una autorización insegura.

61. Observability

El flujo completo debe ser observable:

HTTP Request
   ↓
Application
   ↓
Database
   ↓
Outbox
   ↓
Publisher
   ↓
Event Platform
   ↓
Consumer
   ↓
Downstream Action

Se utilizarán:

trace_id
span_id
correlation_id
causation_id
event_id
62. Event Metrics

Métricas mínimas:

events_published
events_failed
events_retried
events_dlq
consumer_processing_latency
consumer_failure_rate
duplicate_events
out_of_order_events
publication_latency
outbox_backlog
replay_count
schema_validation_failures
event_propagation_lag
63. Auditability

Las operaciones relevantes del sistema de eventos deberán poder auditarse.

Especialmente:

replay;
DLQ operations;
schema changes;
subscription changes;
security event handling;
consumer failures;
administrative event actions.
64. Replay Governance

Un replay administrativo deberá requerir:

Authorized Actor
       ↓
Reason
       ↓
Scope
       ↓
Event Selection
       ↓
Approval where required
       ↓
Execution
       ↓
Audit

AI no puede ejecutar un replay privilegiado fuera de sus permisos.

65. AI Integration

AI puede:

consumir Identity Events autorizados;
analizar eventos;
detectar anomalías;
generar recomendaciones;
proponer acciones.

AI no puede:

alterar eventos históricos;
eliminar audit evidence;
modificar Event Platform;
saltarse tenant isolation;
publicar eventos falsos;
utilizar eventos como mecanismo de escalación de privilegios.
66. AI Event Attribution

Los eventos producidos por AI deberán identificar:

AI_AGENT
Agent Identity
Tenant
Actor / Delegation Context
Correlation
Causation

cuando corresponda.

La acción del AI debe permanecer atribuible.

67. Privacy

Los eventos deben aplicar:

Data Minimization
Purpose Limitation
Least Privilege
Tenant Isolation
Retention
Access Control

No se debe incluir información personal simplemente porque esté disponible en Identity.

El consumer recibe únicamente aquello que necesita.

68. Event Retention

No todos los eventos tendrán la misma retención.

Se diferenciará:

Operational Events
Security Events
Audit Events
Integration Events
DLQ Events
Replay Evidence

La política concreta deberá alinearse con los requisitos de retention definidos para Identity.

69. Disaster Recovery

Event infrastructure deberá formar parte del modelo de recuperación.

Debe ser posible recuperar:

PostgreSQL
      +
Outbox
      +
Event Platform state
      +
Consumer processing

Un recovery no debe producir:

duplicate business outcome
70. Recovery Sequence

Conceptualmente:

Restore PostgreSQL
       ↓
Validate Identity State
       ↓
Recover Outbox
       ↓
Recover Event Platform
       ↓
Reconcile Consumers
       ↓
Replay Required Events
       ↓
Validate Idempotency
       ↓
Resume Normal Operation
71. Event Contract Testing

Cada evento deberá tener Contract Tests para:

envelope;
schema;
required fields;
version;
tenant;
actor;
resource;
compatibility;
sensitive-data rules.

También deberán probarse consumers.

72. Event Security Testing

Como mínimo:

Cross-tenant event
Unknown event
Unknown version
Malformed schema
Duplicate event
Out-of-order event
Replay
DLQ
Consumer failure
Publisher failure
Event Platform outage
Invalid signature
Unauthorized consumer
Sensitive data leakage
73. Golden Scenarios

Los eventos críticos de Identity deberán integrarse con los Golden Scenarios definidos en los ADR de testing.

Especialmente:

Tenant Isolation
User Suspension
Membership Revocation
Role Revocation
Permission Revocation
Session Revocation
Refresh Token Reuse
MFA Revocation
Policy Change
Cache Invalidation
74. Event Security Invariants

EVENT-001 — Events represent facts, not commands.

EVENT-002 — Events are versioned.

EVENT-003 — Events are tenant-aware where required.

EVENT-004 — Consumers are idempotent.

EVENT-005 — Delivery is assumed At-Least-Once.

EVENT-006 — No secrets appear in events.

EVENT-007 — Outbox state is transactionally consistent.

EVENT-008 — Event versions cannot silently change semantics.

EVENT-009 — Cross-tenant event processing is denied.

EVENT-010 — Replay cannot duplicate business outcomes.

EVENT-011 — Out-of-order events cannot regress security state.

EVENT-012 — DLQ access is controlled and audited.

EVENT-013 — AI cannot forge privileged Identity events.

EVENT-014 — Security events receive appropriate protection.

EVENT-015 — Event consumers have least privilege.

EVENT-016 — Cache invalidation events cannot directly grant authorization.

EVENT-017 — Event Platform failure cannot create authorization bypass.

EVENT-018 — Event schema changes require compatibility governance.

EVENT-019 — Historical events remain immutable.

EVENT-020 — Event replay is explicit, controlled and auditable.

75. Implementation Structure

La implementación podrá seguir conceptualmente:

apps/api/app/domains/identity/
│
├── domain/
│   └── events/
│
├── application/
│   └── event_handlers/
│
├── infrastructure/
│   ├── outbox/
│   ├── event_publisher/
│   ├── event_consumers/
│   ├── schemas/
│   └── integrations/
│
└── api/

La ubicación exacta de los adapters de Event Platform dependerá de la arquitectura final del runtime.

76. Traceability
Artifact	Relación
ESP-0001/01	Identity Domain Model
ESP-0001/03	Authentication & Authorization
ESP-0001/05	Roles & Permissions
ESP-0001/06	Sessions & Tokens
ESP-0001/07	MFA
ESP-0001/08	Security Policies
ESP-0001/09	Audit & Compliance
ESP-0001/10	API Contracts
ESP-0001/11	Persistence & Data Model
ESP-0001/13	Testing & Quality
ESP-0001/14	Implementation
ESP-0001/15	Deployment
ADR-007	Transactional Outbox
ADR-018	Central Event Broker
ADR-019	Partitioning / Routing / Ordering
ADR-020	Schema Registry
ADR-021	Controlled Replay
ADR-022	Consumer Idempotency
ADR-028	Observability
ADR-029/030	DR
ADR-053	Authorization Cache Invalidation
ADR-054/055	Cache Failure / Recovery
ADR-056/057	Freshness
ADR-062–067	Test Catalog / Execution / Evidence / Contracts
77. Implementation Readiness
Capability	Estado
Event-driven model	🟢
Domain vs Integration Events	🟢
Event Envelope	🟢
Event ID	🟢
Versioning	🟢
Tenant awareness	🟢
Actor attribution	🟢
Correlation/Causation	🟢
Event catalog	🟢
At-Least-Once	🟢
Consumer idempotency	🟢
Scoped ordering	🟢
Retry	🟢
DLQ	🟢
Replay	🟢
Transactional Outbox	🟢
Schema Registry	🟢
Schema compatibility	🟢
Event security	🟢
Consumer authorization	🟢
Cache invalidation	🟢
AI integration	🟢
Observability	🟢
Disaster recovery	🟢
Contract testing	🟢
Exact Event Platform technology	🟡
Exact topic/stream names	🟡
Exact schemas	🟡
Exact partition keys	🟡
Exact retry configuration	🟡
Exact DLQ retention	🟡
Exact webhook implementation	🟡
78. Remaining Decisions

Quedan deliberadamente para la etapa de implementación:

tecnología concreta de Event Platform;
topic/stream topology;
nombres definitivos de topics;
partition keys;
consumer groups;
retry/backoff exactos;
DLQ retention;
schema serialization format;
Schema Registry implementation;
webhook infrastructure;
exact event payload schemas;
event retention por categoría;
cross-region event strategy;
exact observability dashboards.

No considero necesario crear otro ADR simplemente para volver a decidir principios que ya están establecidos.

79. Final Assessment
ESP-0001/12 original

🟡 — Buena base, pero parcialmente desactualizado.

El documento original ya tenía prácticamente todos los conceptos fundamentales: envelope, event catalog, versioning, tenant awareness, correlation, causation, retries, DLQ, replay, consumers, external providers y webhooks.

El principal problema era que algunos elementos posteriormente decididos por los ADR todavía aparecían como pendientes.

ESP-0001/12 v1.1.0

🟢 — Arquitecturalmente consolidado.

Ahora la arquitectura queda:

                 IDENTITY
                    │
              Domain Change
                    │
                    ▼
             Domain Event
                    │
                    ▼
        TRANSACTIONAL OUTBOX
                    │
                    ▼
            EVENT PLATFORM
                    │
       ┌────────────┼────────────┐
       ▼            ▼            ▼
   Security     Analytics    Integration
       │                         │
       ▼                         ▼
    AI / Risk                Other Domains

Y, sobre todo, queda establecida una regla fundamental para EVOXA:

Identity no publica eventos directamente desde la lógica HTTP ni depende de que un consumer esté disponible para completar una transacción.

Primero se garantiza el estado transaccional y el Outbox; después se propaga el hecho.

Siguiente paso

Con esto ya tenemos consolidada la cadena:

01 Domain Model
03 Authentication & Authorization
05 Roles & Permissions
07 MFA
09 Audit & Compliance
10 API Contracts
11 Persistence & Data Model
12 Events & Integration

El siguiente documento lógico es ESP-0001/13 — Identity Testing & Quality Requirements.

Y este será particularmente importante, porque aquí vamos a incorporar formalmente todo lo que construimos en ADR-062 → ADR-067: Golden Scenarios, Security Invariants, Test Catalog, Test Execution Engine, machine-readable schemas, regression, mutation testing, differential testing, evidence, schema/runtime versioning y los contratos entre Test Runtime y Authorization Runtime.
