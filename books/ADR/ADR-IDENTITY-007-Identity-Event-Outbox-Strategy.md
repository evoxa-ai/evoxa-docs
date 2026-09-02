ADR-IDENTITY-007 — Identity Event & Outbox Strategy

Status: Proposed
Version: 1.0.0
Domain: Identity
Decision Type: Architecture / Integration / Reliability
Scope: Domain Events, Integration Events, Transactional Outbox, Event Publishing, Retry, Idempotency, DLQ, Replay
Related: ESP-0001 — Identity
Depends on:

ADR-IDENTITY-001 — Identity Architecture & Technology Decisions
ADR-IDENTITY-005 — Authorization & Policy Engine
ADR-IDENTITY-006 — Identity Persistence Strategy
1. Context

EVOXA está diseñada como una plataforma Domain-Driven, Modular, API-First y Event-Driven. Las acciones relevantes de negocio deben poder generar eventos que permitan desacoplar los dominios y habilitar integraciones con:

Analytics;
Notifications;
Security;
Administration;
AI Platform;
otros dominios actuales y futuros.

La arquitectura de eventos definida para EVOXA establece que los eventos representan hechos de negocio, no comandos, y que deben ser inmutables, versionados, trazables e idempotentemente procesables.

El principal problema a resolver es la consistencia entre:

Database Transaction
        +
Event Publication

Sin una estrategia adecuada podría producirse:

DB = COMMITTED
Event = NOT PUBLISHED

o:

Event = PUBLISHED
DB = ROLLBACK

Ambos escenarios son problemáticos.

2. Problem Statement

Consideremos:

UserCreated

La aplicación realiza:

1. INSERT User
2. Publish UserCreated

Si el paso 1 funciona:

Database → SUCCESS

pero el broker falla:

Event → FAILURE

el usuario existe, pero los consumidores nunca reciben el evento.

El problema inverso también es posible:

Publish Event → SUCCESS
Database Transaction → ROLLBACK

Los consumidores recibirían un evento correspondiente a un estado que finalmente no existe.

EVOXA necesita una estrategia que reduzca ambos riesgos.

3. Decision

Se adopta:

Transactional Outbox + Event Platform + At-Least-Once Delivery + Idempotent Consumers + Retry + Dead Letter Queue.

La arquitectura será:

Application Command
        │
        ▼
Domain Logic
        │
        ▼
PostgreSQL Transaction
   ┌────┴────┐
   │         │
   ▼         ▼
Aggregate  Outbox Event
   │         │
   └────┬────┘
        │
     COMMIT
        │
        ▼
 Outbox Publisher
        │
        ▼
 Event Platform
        │
   ┌────┼──────────────┐
   ▼    ▼              ▼
Security Analytics Notifications
        │
        ▼
       AI
4. Core Principle

La regla principal será:

El cambio de estado y la creación del evento deben pertenecer a la misma transacción cuando el evento representa ese cambio transaccional.

Ejemplo:

BEGIN TRANSACTION

Create User

Create UserCreated Outbox Event

COMMIT

Después del COMMIT:

Outbox Publisher
       ↓
Event Platform

De esta manera, el evento no depende de que el broker esté disponible durante la transacción de negocio.

5. Domain Events vs Integration Events

Se mantendrá la distinción conceptual:

Domain Event
      ↓
Internal business fact

y:

Integration Event
      ↓
Externalized event contract

Un dominio puede producir un evento interno y posteriormente transformarlo en un evento de integración.

No se debe asumir que todas las estructuras internas del dominio deben exponerse externamente.

6. Events Are Facts

Los eventos deben representar algo que ya ocurrió.

Correcto:

UserCreated
MembershipActivated
RoleAssigned
SessionRevoked
MFAFactorActivated

Incorrecto como evento:

CreateUser
ActivateMembership
AssignRole

Estos últimos son comandos/intenciones, no hechos.

La arquitectura de EVOXA establece explícitamente esta separación.

7. Event Immutability

Una vez publicado:

UserCreated

el evento no deberá modificarse.

Si existe un cambio posterior:

UserUpdated

se genera otro evento.

Por tanto:

UserCreated
      ↓
UserUpdated
      ↓
UserSuspended
      ↓
UserActivated

Los eventos forman una secuencia histórica de hechos.

8. Event Envelope

Los eventos deberán utilizar un envelope estándar.

Conceptualmente:

{
  "event_id": "evt_123",
  "event_type": "identity.user.created",
  "event_version": 1,
  "timestamp": "2026-09-02T00:00:00Z",
  "tenant_id": "org_123",
  "actor": {
    "type": "user",
    "id": "usr_123"
  },
  "resource": {
    "type": "user",
    "id": "usr_123"
  },
  "payload": {},
  "metadata": {
    "correlation_id": "corr_123",
    "causation_id": "cmd_123",
    "trace_id": "trace_123"
  }
}

Los nombres y tipos definitivos deberán formalizarse en el contrato de eventos.

La arquitectura de eventos de EVOXA ya establece event_id, event_type, event_version, timestamp, tenant_id, actor, resource, payload, metadata y correlation/causation IDs como elementos del envelope.

9. Event ID

Cada evento debe tener un identificador único:

event_id

Ejemplo:

evt_01J...

Este identificador permite:

deduplicación;
trazabilidad;
auditoría;
debugging;
correlación;
idempotencia.
10. Event Version

Todos los eventos deberán ser versionados.

Ejemplo:

identity.user.created.v1

Posteriormente:

identity.user.created.v2

El cambio de versión deberá respetar reglas de compatibilidad.

No se debe modificar silenciosamente el significado de un evento ya publicado.

11. Event Naming

Se recomienda una nomenclatura consistente:

<domain>.<aggregate>.<event>

Ejemplos:

identity.user.created
identity.user.updated
identity.user.suspended

identity.organization.created
identity.organization.suspended

identity.membership.created
identity.membership.removed

identity.session.created
identity.session.revoked

identity.mfa.factor.created
identity.mfa.factor.revoked

El catálogo definitivo deberá mantenerse como contrato versionado.

12. Identity Event Catalog

El catálogo inicial de Identity contempla eventos como:

Users
UserCreated
UserUpdated
UserActivated
UserSuspended
UserDisabled
Organizations
OrganizationCreated
OrganizationUpdated
OrganizationSuspended
OrganizationReactivated
Memberships
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
Security Policies
SecurityPolicyCreated
SecurityPolicyUpdated
SecurityPolicyActivated
SecurityPolicyDeactivated
SecurityPolicyViolation

Este catálogo se deriva del modelo de eventos definido para Identity en ESP-0001.

13. Events That Require Outbox

No todo evento necesariamente requiere exactamente el mismo nivel de garantía.

Los eventos que representan cambios persistentes críticos deberán utilizar:

Transactional Outbox

Especialmente:

UserCreated
UserSuspended
UserDisabled

MembershipCreated
MembershipRemoved
MembershipRoleChanged

RoleAssigned
RoleRemoved

SessionRevoked

RefreshTokenRevoked
TokenReuseDetected

MFAFactorActivated
MFAFactorRevoked

SecurityPolicyUpdated

La lista definitiva debe quedar en el catálogo de eventos.

14. Outbox Table

Se utilizará una tabla conceptual:

outbox_events

Estructura propuesta:

id
event_id
event_type
event_version
aggregate_type
aggregate_id
tenant_id
payload
metadata
created_at
published_at
status
retry_count
last_error

Estados posibles:

PENDING
PUBLISHED
FAILED
DEAD_LETTER

La estructura física será definida posteriormente en la especificación de persistencia.

15. Outbox Transaction

Ejemplo:

BEGIN

UPDATE users
SET status = 'SUSPENDED'
WHERE id = :user_id;

INSERT INTO outbox_events (...)
VALUES (
    'evt_123',
    'identity.user.suspended',
    ...
);

COMMIT

Si la transacción falla:

ROLLBACK

y no debe existir:

UserSuspended

en la outbox.

16. Outbox Publisher

Un proceso independiente leerá:

outbox_events
WHERE status = PENDING

y publicará los eventos.

Flujo:

Outbox
   ↓
Claim Event
   ↓
Publish
   ↓
Success?
 ┌─┴─┐
Yes  No
 │    │
 ▼    ▼
Mark Retry
Published

El mecanismo exacto de locking/claiming deberá garantizar que múltiples publishers puedan trabajar de manera segura.

17. At-Least-Once Delivery

La estrategia utilizará:

At-Least-Once Delivery

Esto significa que un evento puede llegar más de una vez.

Ejemplo:

Event #123
   ↓
Consumer
   ↓
Process
   ↓
ACK lost
   ↓
Event #123 delivered again

Por ello:

Los consumidores deben ser idempotentes.

18. Idempotent Consumers

Cada consumidor debe poder reconocer eventos ya procesados.

Ejemplo conceptual:

processed_events

o mediante un mecanismo equivalente.

Flujo:

Event
 ↓
event_id exists?
 ├── YES → Ignore duplicate
 └── NO  → Process
             ↓
        Store event_id

La implementación concreta dependerá del Event Platform.

19. Retry Strategy

Cuando la publicación falla:

Attempt 1
   ↓
Failure
   ↓
Retry
   ↓
Attempt 2
   ↓
Failure
   ↓
Retry

Se recomienda utilizar backoff progresivo.

Conceptualmente:

1
2
4
8
16
...

Los valores definitivos quedan fuera de este ADR.

20. Dead Letter Queue

Después de superar el límite de reintentos:

Outbox
   ↓
Retry
   ↓
Retry
   ↓
Retry
   ↓
DLQ

La Dead Letter Queue permite:

investigar;
corregir;
reprocesar;
evitar pérdida silenciosa.

Un evento enviado a DLQ no debe considerarse procesado correctamente.

21. Replay

La arquitectura deberá permitir replay controlado cuando la infraestructura de eventos lo soporte.

Ejemplo:

Historical Events
       ↓
Replay
       ↓
Analytics

El replay debe ser:

controlado;
auditable;
idempotente;
limitado al tenant correspondiente cuando aplique;
protegido contra efectos secundarios no deseados.

No todos los consumidores necesariamente deben permitir replay automático.

22. Event Ordering

No se requiere orden global de todos los eventos.

La regla será:

Ordering only where required by the aggregate.

Ejemplo:

UserCreated
     ↓
UserSuspended

El consumidor podría necesitar mantener ese orden.

Pero no necesariamente:

UserCreated
OrganizationCreated
RoleCreated

en un orden global.

La arquitectura de eventos de EVOXA establece que el ordering debe garantizarse únicamente cuando sea necesario, particularmente dentro de un aggregate.

23. Concurrency

La outbox debe soportar múltiples workers.

Ejemplo:

Worker A ─┐
Worker B ─┼──> Outbox
Worker C ─┘

No debe ocurrir:

Worker A → Publish Event
Worker B → Publish same Event

sin que el sistema tenga mecanismos para tolerar esa posibilidad.

La solución deberá combinar:

locking/claiming;
event IDs;
idempotencia;
estado de publicación.
24. Failure Scenarios
Database failure
DB Transaction
     ↓
FAIL
     ↓
ROLLBACK

Resultado:

No aggregate change
No outbox event
Event Platform failure
DB
 ↓
COMMIT
 ↓
Outbox
 ↓
Event Platform unavailable

Resultado:

Event remains PENDING

y posteriormente:

Retry
Publisher crash
Publisher
    ↓
Publish
    ↓
CRASH before marking published

Puede ocurrir:

Duplicate delivery

Los consumidores deben tolerarlo mediante idempotencia.

25. Security

Los eventos no deben contener:

password
password_hash
access_token
refresh_token
MFA_secret
recovery_code
private_key

Tampoco deben incluir información sensible que no sea necesaria para el consumidor.

Especialmente en Identity:

TokenReuseDetected

debe indicar que ocurrió una reutilización, pero nunca incluir el token comprometido.

26. Tenant Isolation

Todo evento relacionado con un recurso de organización deberá transportar el contexto de tenant cuando corresponda:

tenant_id

Los consumidores deben validar dicho contexto.

No se debe permitir:

Tenant A Event
      ↓
Tenant B Data

La seguridad multi-tenant debe mantenerse también dentro de la plataforma de eventos.

27. Correlation and Causation

Los eventos deberán permitir reconstruir cadenas de operaciones.

Ejemplo:

HTTP Request
     │
     │ correlation_id = C1
     ▼
Suspend User
     │
     │ causation_id = CMD1
     ▼
UserSuspended
     │
     └── correlation_id = C1

Esto permite seguir una operación completa entre:

API
Application
Domain
Database
Outbox
Event Platform
Consumers
AI
28. Observability

El sistema deberá medir:

outbox_pending_events
outbox_publish_latency
outbox_publish_failures
outbox_retry_count
events_published
events_failed
events_dlq
consumer_processing_latency
consumer_failures
duplicate_events

Esto permite detectar:

Outbox backlog
Publisher failure
Broker problems
Consumer failures
29. Audit

La publicación y los problemas de seguridad relevantes deberán quedar trazables.

Ejemplos:

TokenReuseDetected
SecurityPolicyViolation
AuthorizationDenied
MFAChallengeFailed

No todos los eventos de dominio deben convertirse automáticamente en registros de auditoría.

La distinción será:

Domain Event
    ≠
Audit Event

aunque una misma operación pueda producir ambos.

30. Integration Consumers

Los principales consumidores previstos incluyen:

Identity Events
       │
       ├── Security
       ├── Notifications
       ├── Analytics
       ├── Administration
       └── AI Platform

Por ejemplo:

UserSuspended
      ↓
Security
      ↓
Revoke Security Context

o:

MembershipCreated
      ↓
Notifications
      ↓
Send Invitation / Welcome Notification
31. AI Integration

Los eventos de Identity podrán alimentar componentes de AI cuando exista una necesidad legítima.

Por ejemplo:

Security Events
       ↓
AI Security Analysis
       ↓
Risk Detection

Sin embargo:

La IA no debe recibir automáticamente todos los eventos de Identity.

El acceso deberá respetar:

autorización;
tenant isolation;
minimización de datos;
privacidad;
propósito;
políticas de seguridad.
32. Event Schema Governance

Los eventos deberán tener:

Owner
Schema
Version
Compatibility Rules
Documentation
Lifecycle

La evolución debe ser controlada.

Ejemplo:

identity.user.created.v1

No se debe modificar arbitrariamente el contrato de v1.

Si el cambio es incompatible:

v1 → v2

deberá existir una estrategia de migración/compatibilidad.

33. Event Lifecycle

Conceptualmente:

CREATED
   ↓
PENDING
   ↓
PUBLISHED
   ↓
CONSUMED

En caso de errores:

PENDING
   ↓
FAILED
   ↓
RETRY
   ↓
FAILED
   ↓
DEAD_LETTER

El evento original permanece conceptualmente inmutable.

34. Data Retention

La retención de:

Outbox
Event Platform
Audit

no necesariamente debe ser idéntica.

Por ejemplo:

Outbox
→ temporal hasta publicación

Event Platform
→ según estrategia de eventos

Audit
→ retención prolongada

Los períodos exactos no se fijan en este ADR.

35. Transactional Boundaries

La regla será:

Business State Change
        +
Outbox Event
        ↓
Same Transaction

pero:

Event Publication

ocurrirá fuera de la transacción de negocio.

Esto evita mantener transacciones abiertas esperando a infraestructura externa.

36. Alternatives Considered
A. Dual Write
Database
   +
Event Broker

en operaciones independientes.

Rechazada

Puede producir inconsistencias:

DB success
Broker failure

o:

Broker success
DB failure
B. Publish Before Database Commit
Rechazada

Podría publicar eventos de operaciones que posteriormente hacen rollback.

C. Database Polling Without Outbox
Rechazada

No proporciona una frontera clara y confiable entre el cambio de dominio y el evento que representa ese cambio.

D. Transactional Outbox
Seleccionada

Permite:

atomicidad entre cambio y evento;
retry;
recuperación;
trazabilidad;
desacoplamiento;
at-least-once delivery.
37. Consequences
Positivas
Reduce el riesgo de pérdida de eventos.
Evita dual writes inseguros.
Permite retry.
Permite DLQ.
Facilita observabilidad.
Permite consumidores desacoplados.
Mantiene los dominios independientes.
Facilita integración con AI y Analytics.
Permite evolución del Event Platform.
Negativas
Introduce una tabla adicional.
Requiere publisher.
Requiere mecanismos de retry.
Requiere DLQ.
Requiere consumidores idempotentes.
Puede producir duplicados.
Añade complejidad operacional.
Existe consistencia eventual entre DB y consumidores.
38. Implementation Direction

La estructura conceptual en Identity podría ser:

apps/api/app/domains/identity/

├── domain/
│   └── events/
│
├── application/
│   └── event_handlers/
│
└── infrastructure/
    └── events/
        ├── outbox/
        ├── publisher/
        ├── schemas/
        └── consumers/

Y conceptualmente:

Domain Event
      ↓
Application
      ↓
Outbox Repository
      ↓
PostgreSQL
      ↓
Outbox Publisher
      ↓
Event Platform

Esta estructura es una dirección de implementación derivada, no una prescripción literal del Blueprint.

39. Testing Requirements

Se deberán probar:

Transaction
domain change + outbox commit;
rollback;
failed transaction.
Publisher
successful publish;
broker unavailable;
retry;
crash recovery.
Idempotency
duplicate event;
same event_id;
concurrent delivery.
Ordering
aggregate event sequence;
out-of-order events.
DLQ
maximum retries;
DLQ insertion;
replay.
Security
tenant isolation;
secret redaction;
unauthorized consumer;
event tampering.
Observability
correlation ID;
trace ID;
metrics;
failure alerts.
40. Traceability
BP-0002 Reference Architecture
        ↓
ESP-0001 Identity
        ↓
ESP-0001 / 02 Identity Architecture
        ↓
ESP-0001 / 11 Persistence & Data Model
        ↓
ESP-0001 / 12 Events & Integration
        ↓
ESP-0001 / 13 Testing & Quality
        ↓
ESP-0001 / 15 Deployment & Operations
        ↓
ADR-IDENTITY-007

La arquitectura de EVOXA establece Event-Driven Architecture, eventos inmutables, Event Platform centralizada, consumidores idempotentes, at-least-once delivery, retry, DLQ, replay y transactional outbox como elementos de la estrategia de eventos.

41. Follow-up ADRs

Este ADR deja como decisiones posteriores:

ADR-IDENTITY-007.1

Event Transport Technology

Definir el mecanismo concreto:

Kafka
RabbitMQ
Cloud Event Bus
u otro
ADR-IDENTITY-007.2

Event Schema Registry

Definir:

registry;
compatibility;
versioning;
governance.
ADR-IDENTITY-007.3

Outbox Publisher Architecture

Definir:

worker;
locking;
batching;
polling;
CDC si corresponde.
ADR-IDENTITY-007.4

Retry & Dead Letter Policy

Definir:

retry count;
backoff;
DLQ;
replay.
ADR-IDENTITY-007.5

Event Retention & Replay

Definir:

retention;
archival;
replay windows;
consumer replay policies.
42. Final Decision Summary

La arquitectura queda:

                         IDENTITY
                            │
                            ▼
                    Business Operation
                            │
                            ▼
                    ┌───────────────┐
                    │   PostgreSQL  │
                    │               │
                    │ Aggregate     │
                    │      +        │
                    │ Outbox Event  │
                    └───────┬───────┘
                            │
                          COMMIT
                            │
                            ▼
                    Outbox Publisher
                            │
                    ┌───────┴───────┐
                    │               │
                  Retry           Publish
                    │               │
                    └───────┬───────┘
                            ▼
                     Event Platform
                            │
             ┌──────────────┼──────────────┐
             ▼              ▼              ▼
          Security       Analytics    Notifications
             │
             ▼
             AI
Decisión principal

EVOXA Identity utilizará Transactional Outbox para garantizar la consistencia entre cambios persistentes y eventos de dominio/integración. Los eventos serán inmutables, versionados y trazables; la entrega utilizará un modelo At-Least-Once, por lo que los consumidores deberán ser idempotentes. Se utilizarán mecanismos de retry y Dead Letter Queue para eventos que no puedan publicarse o procesarse correctamente, manteniendo aislamiento multi-tenant, seguridad, observabilidad y capacidad de replay controlado.

Estado: PROPOSED.

Con esto quedan cubiertas las decisiones principales de persistencia (ADR-006) y eventos/outbox (ADR-007). El siguiente ADR natural de la secuencia es ADR-IDENTITY-008 — Identity Cryptographic Key Management, antes de entrar a decisiones más concretas de implementación.
