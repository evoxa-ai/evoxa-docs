IS-IDENTITY-013 — Domain Events & Transactional Outbox

Estado: Draft
Prioridad: Critical
Dominio: Identity
Capability: Domain Events / Event-Driven Architecture
Dependencias principales: IS-IDENTITY-001, 002, 003, 004, 005, 006, 007, 008, 009, 010, 011, 012
ADR principal: ADR-IDENTITY-007
Capa arquitectónica: Domain → Application → Infrastructure
Tipo: Implementation Story

1. Propósito

Definir e implementar la capacidad de Domain Events y Transactional Outbox para el dominio Identity de EVOXA, garantizando que los cambios de estado del dominio puedan convertirse en eventos publicados hacia la plataforma de eventos sin utilizar un patrón de dual-write inseguro.

La implementación debe garantizar que:

Un cambio de estado confirmado en el dominio y la generación de su evento sean una misma operación transaccional.

La publicación hacia el Event Platform podrá fallar temporalmente sin perder el evento, porque el evento permanecerá persistido en el Outbox hasta que pueda ser publicado.

Esta Implementation Story materializa principalmente ADR-IDENTITY-007 — Transactional Outbox & Event Platform.

2. Business Goal

Proporcionar a Identity una infraestructura confiable para comunicar cambios importantes a otros componentes de EVOXA.

Esto permitirá desacoplar Identity de otros dominios y servicios que necesiten reaccionar ante eventos como:

creación de usuarios;
activación de usuarios;
suspensión de usuarios;
creación de organizaciones;
cambios de membresías;
cambios de roles;
creación y revocación de sesiones;
rotación de Refresh Tokens;
eventos MFA;
cambios de políticas de seguridad;
eventos de autorización;
eventos de seguridad.

El objetivo no es simplemente enviar mensajes, sino establecer una infraestructura de eventos:

transaccional → durable → versionada → observable → recuperable → idempotente.

3. Persona
Primary Persona

Identity Platform

Componente responsable de generar hechos de negocio y seguridad derivados de cambios en Identity.

Secondary Personas
otros dominios EVOXA;
servicios de plataforma;
Audit Service;
Security/Event Processing;
AI Services;
sistemas de notificación;
analytics;
observabilidad;
integraciones externas.
4. Journey

Flujo conceptual:

Identity Use Case
      │
      ▼
Domain Aggregate
      │
      ├── cambia estado
      │
      └── genera Domain Event
              │
              ▼
        Application Layer
              │
              ▼
       Database Transaction
        ┌───────────────┐
        │ Domain State  │
        │      +        │
        │ Outbox Event  │
        └───────────────┘
              │
              ▼
           COMMIT
              │
              ▼
       Outbox Relay/Publisher
              │
              ▼
        Event Platform
              │
       ┌──────┴───────┐
       ▼              ▼
   Consumer A      Consumer B
       │              │
       ▼              ▼
   Process         Process
       │
       ▼
   Idempotency
5. Feature Pack

Esta IS incorpora:

Domain Event model.
Integration Event model.
Event Envelope.
Transactional Outbox.
Outbox persistence.
Outbox publisher/relay.
At-least-once delivery.
Consumer idempotency.
Retry.
Exponential backoff.
Dead Letter Queue.
Event replay.
Event versioning.
Event schema governance.
Correlation/Causation/Trace propagation.
Tenant isolation.
Event security.
Event observability.
Failure recovery.
Event lifecycle management.
6. Alcance
6.1 In Scope

Esta IS cubre:

generación de Domain Events;
captura de eventos;
persistencia Transactional Outbox;
publicación hacia Event Platform;
event envelope;
versionado;
retries;
DLQ;
replay;
idempotencia;
trazabilidad;
observabilidad;
seguridad de eventos;
integración con Audit/Security Events.
6.2 Out of Scope

No define todavía:

proveedor específico del Event Platform;
Kafka vs RabbitMQ vs otro broker;
configuración definitiva de particiones;
retención definitiva;
schema registry específico;
infraestructura Kubernetes concreta;
mecanismo definitivo de encryption-at-rest del broker;
contratos definitivos de cada consumidor;
política legal definitiva de retención de eventos.

Esos puntos deberán quedar en ADRs posteriores.

7. Distinción de tipos de eventos

Es fundamental no mezclar diferentes categorías.

7.1 Domain Event

Representa un hecho ocurrido dentro del dominio.

Ejemplo:

UserCreated

Significa:

Un usuario fue creado.

No representa una orden ni una intención.

7.2 Integration Event

Es la representación de un hecho que se publica fuera del bounded context.

Ejemplo:

identity.user.created.v1

Puede derivarse de:

UserCreated

El Domain Event y el Integration Event no necesariamente tienen que ser idénticos.

7.3 Audit Event

Representa una acción que debe quedar registrada para trazabilidad.

Ejemplo:

RolePermissionAssigned
7.4 Security Event

Representa un hecho relevante desde el punto de vista de seguridad.

Ejemplo:

RefreshTokenReuseDetected

Un mismo acontecimiento puede producir:

Domain Event
       │
       ├── Integration Event
       │
       ├── Audit Event
       │
       └── Security Event

pero conceptualmente siguen siendo responsabilidades diferentes.

8. Event Envelope

Todos los eventos publicados deberán utilizar un envelope estándar.

Conceptualmente:

{
  "event_id": "uuid",
  "event_type": "identity.user.created",
  "event_version": 1,
  "timestamp": "2026-09-02T00:00:00Z",

  "tenant_id": "uuid",

  "actor": {
    "type": "USER",
    "id": "uuid"
  },

  "resource": {
    "type": "USER",
    "id": "uuid"
  },

  "payload": {},

  "metadata": {
    "correlation_id": "uuid",
    "causation_id": "uuid",
    "trace_id": "string"
  }
}

Los nombres definitivos y el schema formal deberán consolidarse en el contrato de eventos de plataforma.

9. Requisitos del Event Envelope

Cada evento deberá permitir identificar:

event_id;
event_type;
event_version;
timestamp;
tenant;
actor;
resource;
payload;
metadata;
correlation ID;
causation ID;
trace ID cuando corresponda.
Regla

event_id debe ser único.

Esto permitirá que consumidores puedan detectar eventos duplicados.

10. Transactional Outbox

El principio fundamental será:

BEGIN TRANSACTION

    modificar aggregate

    persistir domain state

    registrar outbox event

COMMIT

Nunca:

UPDATE database
COMMIT

publish event

como dos operaciones independientes.

11. Problema que resuelve

Sin Outbox:

Database
   │
   ├── SUCCESS
   │
   ▼
Application
   │
   └── Event Broker
          │
          └── FAILURE

Resultado:

Estado actualizado pero evento perdido.

Con Transactional Outbox:

Database Transaction
       │
       ├── Domain State
       │
       └── Outbox Event
              │
              ▼
           COMMIT
              │
              ▼
         Outbox Relay
              │
              ▼
        Event Platform

Si el broker está temporalmente fuera de servicio:

Outbox Event = PENDING

y podrá ser publicado posteriormente.

12. OutboxEvent

La entidad conceptual será:

OutboxEvent

Campos mínimos propuestos:

Campo	Propósito
id	Identificador interno
event_id	Identificador público único del evento
event_type	Tipo de evento
event_version	Versión del contrato
occurred_at	Momento del acontecimiento
tenant_id	Tenant asociado
aggregate_type	Tipo de aggregate
aggregate_id	Aggregate afectado
actor	Actor responsable
correlation_id	Correlación
causation_id	Causalidad
trace_id	Trazabilidad
payload	Datos del evento
metadata	Metadata adicional
status	Estado de publicación
attempt_count	Cantidad de intentos
next_attempt_at	Próximo intento
published_at	Momento de publicación
created_at	Creación

La estructura exacta de persistencia queda pendiente de la especificación técnica de Data Model.

13. Outbox Lifecycle

Se propone:

PENDING
   │
   ▼
PROCESSING
   │
   ├───────────────┐
   │               │
 SUCCESS         FAILURE
   │               │
   ▼               ▼
PUBLISHED       RETRY
                   │
                   ▼
                 PENDING
                   │
             max attempts
                   │
                   ▼
                  DLQ

Los estados exactos y su persistencia deberán consolidarse durante la implementación.

14. Outbox Publisher / Relay

Se deberá implementar un componente responsable de:

detectar eventos pendientes;
reclamarlos de manera segura;
publicarlos;
manejar errores;
actualizar estado;
registrar métricas;
ejecutar retries;
enviar eventos irrecuperables a DLQ.

Conceptualmente:

Outbox
  │
  ▼
Publisher
  │
  ├── Claim
  │
  ├── Publish
  │
  ├── Confirm
  │
  └── Retry
15. Concurrencia del Publisher

Debe evitarse que múltiples publishers procesen simultáneamente el mismo evento de manera descontrolada.

La estrategia concreta podrá utilizar mecanismos como:

row locking;
leasing;
status transitions;
worker ownership;
partitioning.

La estrategia definitiva queda pendiente de ADR de infraestructura de eventos.

16. At-Least-Once Delivery

La arquitectura adoptará:

At-least-once delivery

No se asumirá entrega exactly-once.

Esto significa que un evento puede llegar más de una vez.

Ejemplo:

Publisher
   │
   ├── publish()
   │
   ▼
Broker recibe evento
   │
   X
   │
Publisher no recibe ACK

El publisher puede volver a intentar:

publish()

El consumidor recibirá:

event_id = ABC
event_id = ABC

Por lo tanto:

Los consumidores deben ser idempotentes.
17. Consumer Idempotency

Cada consumidor debe poder determinar si ya procesó un evento.

Conceptualmente:

event_id
   │
   ▼
Deduplication Store
   │
   ├── already processed → ignore
   │
   └── new → process

El procesamiento debe garantizar:

check
+
business operation
+
mark processed

con garantías transaccionales apropiadas al consumidor.

18. Retry Policy

Los errores transitorios deberán generar retry.

Ejemplo conceptual:

Attempt 1
   ↓
failure
   ↓
backoff
   ↓
Attempt 2
   ↓
failure
   ↓
backoff
   ↓
Attempt 3

Se deberá utilizar backoff progresivo, potencialmente exponencial.

Los valores exactos de:

máximo de intentos;
intervalo inicial;
intervalo máximo;
jitter;

quedan pendientes de definición de plataforma.

19. Dead Letter Queue

Los eventos que no puedan procesarse después de la política de retry deberán ser enviados a:

DLQ

La DLQ debe permitir:

identificación del evento;
causa del fallo;
cantidad de intentos;
timestamps;
diagnóstico;
reintento manual/automatizado;
replay controlado.
20. Event Replay

La arquitectura deberá permitir replay controlado de eventos.

Casos:

recuperación de un consumidor;
reconstrucción de proyecciones;
reparación de procesamiento;
migración;
incorporación de un nuevo consumidor.

El replay debe conservar:

event_id original
event_type
event_version
timestamp original
tenant
resource

No debe generar eventos falsamente atribuidos a una nueva operación de negocio.

21. Event Ordering

No se deberá asumir orden global de todos los eventos.

Cuando el orden sea necesario:

Aggregate
   │
   ▼
Partition Key
   │
   ▼
Ordered Stream

La estrategia preferida será preservar orden por aggregate o partición, cuando el Event Platform lo soporte.

El orden global queda fuera del alcance salvo que un caso de negocio específico lo requiera.

22. Event Versioning

Los eventos deben ser:

inmutables y versionados.

Ejemplo:

identity.user.created.v1
identity.user.created.v2

Una modificación incompatible no deberá cambiar silenciosamente el significado de v1.

23. Compatibilidad

Los contratos de eventos deberán evolucionar siguiendo compatibilidad controlada.

Se debe evitar:

v1
↓
cambio incompatible
↓
sigue llamándose v1

En su lugar:

v1
↓
v2

cuando exista una modificación incompatible.

24. Eventos iniciales de Identity

La infraestructura debe soportar eventos como:

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
Role / Permission
RoleCreated
RoleUpdated
RoleDeleted
RoleAssigned
RoleRemoved

PermissionCreated
PermissionUpdated
PermissionAssigned
PermissionRevoked
Session
SessionCreated
SessionActivated
SessionRevoked
SessionExpired
Refresh Token
RefreshTokenCreated
RefreshTokenRotated
RefreshTokenRevoked
RefreshTokenExpired
RefreshTokenReuseDetected
TokenFamilyRevoked
MFA
MFAFactorEnrolled
MFAFactorActivated
MFAFactorRevoked
MFAChallengeCreated
MFAChallengeVerified
MFAChallengeFailed
MFAChallengeExpired
RecoveryCodesGenerated
RecoveryCodeUsed
RecoveryCodesRevoked
TrustedDeviceRegistered
TrustedDeviceRevoked
Security Policy
PolicyCreated
PolicyUpdated
PolicyActivated
PolicySuspended
PolicyRetired

AuthorizationDecisionMade
AuthorizationDenied
AuthorizationChallengeRequired
PolicyViolationDetected
PrivilegeEscalationDetected

Estos eventos deben considerarse catálogo inicial derivado de las IS anteriores, no necesariamente un catálogo definitivo de plataforma.

25. Tenant Isolation

Todo evento asociado a una organización deberá transportar:

tenant_id

Los consumidores deberán respetar el aislamiento de tenant.

Regla crítica:

Un consumidor nunca debe utilizar un evento de un tenant para modificar o exponer recursos de otro tenant.

Los eventos SYSTEM pueden no tener tenant cuando corresponda.

26. Seguridad de Eventos

Está prohibido colocar en eventos:

passwords;
password hashes;
access tokens;
refresh tokens;
TOTP secrets;
OTPs;
private keys;
recovery codes;
secretos de infraestructura.

Además:

Los eventos deben transportar el mínimo de información necesario para cumplir su propósito.

Cuando sea suficiente, se deberán utilizar referencias:

{
  "user_id": "...",
  "organization_id": "..."
}

en lugar de copiar información sensible.

27. AI y Event Platform

Los eventos que puedan ser consumidos por AI Services deberán respetar las mismas reglas de autorización y privacidad del dominio.

Un evento no implica automáticamente:

AI can access everything

La información entregada a un AI Agent deberá continuar limitada por:

tenant;
autorización;
ownership;
policy;
privacidad;
contexto permitido.
28. Integración con Audit & Security Events

IS-013 no reemplaza IS-IDENTITY-012.

La relación será:

Domain Operation
       │
       ▼
Domain Event
       │
       ├── Integration Event
       │
       ├── Audit Event
       │
       └── Security Event

Cuando un audit/security event sea obligatorio para una operación crítica, su persistencia deberá integrarse correctamente con la estrategia transaccional/outbox.

Esto evita:

Business operation SUCCESS
Audit operation FAILURE

sin una estrategia de recuperación.

29. Correlation / Causation / Trace

Los eventos deberán permitir reconstruir una cadena causal.

Ejemplo:

HTTP Request
 correlation_id = A
       │
       ▼
User Login
       │
       └── UserAuthenticated
              correlation_id = A
              causation_id = B
              trace_id = C

Esto permitirá investigar:

Request
 → Authentication
 → Session
 → Token
 → Authorization
 → Business Operation
30. Observabilidad

El sistema debe exponer métricas como mínimo para:

Outbox
pending events;
processing events;
published events;
failed events;
DLQ events;
oldest pending event;
publishing latency;
retry count;
publication failures.
Consumers
processed events;
duplicated events;
processing failures;
retry count;
DLQ count;
processing latency.
Event Platform
publish latency;
unavailable broker;
connection failures;
throughput;
backlog.
31. Logging

Los logs deben incluir:

event_id
event_type
event_version
tenant_id
aggregate_id
correlation_id
causation_id
trace_id

pero nunca secretos.

El payload sensible no deberá aparecer automáticamente en logs.

32. Performance

La solución deberá ser compatible con los objetivos globales de EVOXA.

El mecanismo Outbox no debe bloquear innecesariamente la transacción principal.

Debe minimizar:

locks prolongados;
polling excesivo;
consultas full-table;
serialización innecesaria;
payloads excesivos.

El objetivo de latencia global de API permanece:

API P95 < 300 ms

La publicación asíncrona no deberá convertirse en requisito síncrono para una operación de negocio salvo que una especificación posterior lo justifique.

33. Failure Modes
Caso 1 — DB transaction falla
Domain state = rollback
Outbox = rollback

Resultado correcto.

Caso 2 — DB commit correcto / broker caído
Domain state = committed
Outbox = PENDING
Broker = unavailable

Resultado:

evento no perdido.

Caso 3 — Broker recibe evento / ACK perdido

Puede producir:

duplicate event

El consumidor debe ser idempotente.

Caso 4 — Consumer falla
event
 ↓
consumer
 ↓
failure
 ↓
retry
Caso 5 — Poison Event

Si falla repetidamente:

retry
 ↓
retry
 ↓
retry
 ↓
DLQ
Caso 6 — Replay

Debe poder reprocesarse sin producir efectos duplicados.

34. Application Services

Se deberán establecer responsabilidades como:

DomainEventCollector

Recolectar eventos generados por aggregates.

OutboxService

Registrar eventos en la transacción.

OutboxPublisher

Publicar eventos pendientes.

EventRetryService

Gestionar retries.

EventReplayService

Gestionar replay controlado.

EventSchemaService

Gestionar validación/versionado de contratos cuando corresponda.

Los nombres concretos pueden variar durante implementación.

35. Repository Contracts

Se propone:

OutboxEventRepository

con capacidades conceptuales:

append()
get_pending()
claim()
mark_published()
mark_failed()
schedule_retry()
move_to_dlq()
get_by_event_id()

La implementación concreta pertenece a Infrastructure.

36. Domain Boundary

El Domain Layer no debe conocer:

Kafka;
RabbitMQ;
HTTP;
Redis;
PostgreSQL;
Kubernetes.

El dominio solamente conoce:

Domain Event

La infraestructura conoce:

Event Platform
Outbox persistence
Publisher
37. Architecture

La arquitectura resultante:

┌─────────────────────────────┐
│       Presentation/API      │
└──────────────┬──────────────┘
               │
┌──────────────▼──────────────┐
│        Application          │
│       Use Cases             │
└──────────────┬──────────────┘
               │
┌──────────────▼──────────────┐
│           Domain            │
│ Aggregates + Domain Events  │
└──────────────┬──────────────┘
               │
               ▼
       Transaction Boundary
               │
       ┌───────┴────────┐
       ▼                ▼
 Domain State       OutboxEvent
       │                │
       └───────┬────────┘
               ▼
          PostgreSQL
               │
               ▼
       Outbox Publisher
               │
               ▼
        Event Platform
               │
       ┌───────┼────────┐
       ▼       ▼        ▼
    Domain   Audit     AI
   Consumer Consumer Consumer
38. API

IS-013 no requiere necesariamente APIs públicas de negocio.

La comunicación principal será:

Domain → Outbox → Event Platform

Podrán existir APIs operacionales internas para:

GET /health/events
GET /metrics/events
GET /outbox/status

pero los contratos definitivos deberán ser establecidos por la arquitectura API.

No se deben considerar todavía contratos públicos finales.

39. Technical Tasks
Event Model

T01 — Definir DomainEvent base.
T02 — Definir IntegrationEvent.
T03 — Definir EventEnvelope.
T04 — Definir EventVersion.
T05 — Definir EventIdentity/EventId.
T06 — Definir metadata de eventos.

Domain Integration

T07 — Integrar eventos con aggregates.
T08 — Implementar event collection.
T09 — Definir publicación desde Application Layer.
T10 — Evitar dependencias del Domain respecto del broker.

Outbox

T11 — Definir OutboxEvent.
T12 — Definir OutboxRepository.
T13 — Crear persistence model.
T14 — Crear índices necesarios.
T15 — Implementar append transaccional.
T16 — Garantizar atomicidad Domain State + Outbox.
T17 — Implementar estados Outbox.
T18 — Implementar event lookup.

Publisher

T19 — Implementar OutboxPublisher.
T20 — Implementar event claiming.
T21 — Implementar publicación al Event Platform.
T22 — Implementar confirmación de publicación.
T23 — Implementar concurrencia segura.
T24 — Implementar publisher recovery.

Retry

T25 — Implementar retry policy.
T26 — Implementar exponential backoff.
T27 — Implementar jitter.
T28 — Persistir attempt count.
T29 — Persistir next_attempt_at.

DLQ

T30 — Definir DLQ contract.
T31 — Implementar movimiento a DLQ.
T32 — Registrar causa de fallo.
T33 — Permitir inspección operacional.
T34 — Definir replay desde DLQ.

Consumer Idempotency

T35 — Definir consumer idempotency contract.
T36 — Definir deduplication strategy.
T37 — Persistir processed event IDs.
T38 — Probar duplicate delivery.

Versioning

T39 — Definir event naming convention.
T40 — Definir versioning convention.
T41 — Definir compatibility rules.
T42 — Implementar schema validation.

Security

T43 — Implementar payload sanitization.
T44 — Prohibir secrets en eventos.
T45 — Aplicar tenant isolation.
T46 — Implementar actor attribution.
T47 — Implementar correlation/causation/trace.

Observability

T48 — Implementar métricas Outbox.
T49 — Implementar métricas Publisher.
T50 — Implementar métricas Consumer.
T51 — Implementar logs estructurados.
T52 — Implementar tracing.

Testing

T53 — Unit tests Domain Events.
T54 — Unit tests Outbox.
T55 — Transaction tests.
T56 — Publisher integration tests.
T57 — Retry tests.
T58 — DLQ tests.
T59 — Duplicate delivery tests.
T60 — Replay tests.
T61 — Failure recovery tests.
T62 — Tenant isolation tests.
T63 — Security tests.
T64 — Performance/load tests.

40. Acceptance Criteria
AC-001 — Atomicidad

Given una operación que modifica un aggregate
When la operación es confirmada
Then el cambio de dominio y su Outbox Event deben confirmarse dentro de la misma transacción.

AC-002 — Rollback

Given una operación que falla
When la transacción hace rollback
Then no debe existir un Outbox Event correspondiente a una operación no confirmada.

AC-003 — Broker unavailable

Given una transacción confirmada
And el Event Platform está indisponible
When el publisher intenta publicar
Then el evento debe permanecer durable en Outbox.

AC-004 — At-least-once

Given un evento publicado
When el ACK de publicación se pierde
Then el sistema puede volver a publicar el evento.

AC-005 — Idempotencia

Given un consumidor recibe dos veces el mismo event_id
When procesa ambos mensajes
Then el efecto de negocio debe ejecutarse una sola vez.

AC-006 — Retry

Given un error transitorio
When el procesamiento falla
Then debe ejecutarse la política de retry configurada.

AC-007 — DLQ

Given un evento que supera la política de retry
When no puede procesarse correctamente
Then debe ser enviado a DLQ.

AC-008 — Replay

Given un evento almacenado en DLQ
When se solicita un replay autorizado
Then el evento debe poder reprocesarse conservando su identidad original.

AC-009 — Versioning

Given un cambio incompatible en un evento
When se publica el nuevo contrato
Then debe utilizar una nueva versión.

AC-010 — Tenant Isolation

Given un evento perteneciente al tenant A
When es procesado
Then no debe permitir acceso/modificación sobre recursos del tenant B.

AC-011 — Secrets

Given cualquier evento de Identity
When es serializado
Then no debe contener passwords, tokens, MFA secrets, private keys ni recovery codes.

AC-012 — Observability

Given un evento procesado
When se consulta observabilidad
Then debe ser posible correlacionarlo mediante event_id y los identificadores de tracing disponibles.

41. Definition of Done

La IS podrá considerarse Implemented cuando:

 Domain Event base implementado.
 Event Envelope definido.
 Event versioning implementado.
 Outbox persistente implementado.
 Atomicidad DB + Outbox garantizada.
 Publisher implementado.
 Retry implementado.
 DLQ implementada.
 Idempotency contract definido.
 Tenant isolation validado.
 Security filtering implementado.
 Correlation/Causation/Trace implementados.
 Métricas disponibles.
 Logs estructurados disponibles.
 Tests unitarios implementados.
 Tests de integración implementados.
 Tests de fallos implementados.
 Tests de duplicación implementados.
 Tests de replay implementados.
 Security tests aprobados.
 Performance tests aprobados.
 Documentación actualizada.
 Traceability completa.
 Code review aprobado.
 Quality gates aprobados.
42. Dependencias
Upstream
IS-IDENTITY-001
IS-IDENTITY-002
IS-IDENTITY-003
IS-IDENTITY-004
IS-IDENTITY-005
IS-IDENTITY-006
IS-IDENTITY-007
IS-IDENTITY-008
IS-IDENTITY-009
IS-IDENTITY-010
IS-IDENTITY-011
IS-IDENTITY-012
Downstream

Esta IS habilita especialmente:

Identity integrations
Audit processing
Security detection
Notifications
Analytics
AI Services
Other Domains
Event-driven workflows
43. Traceability
Nivel	Referencia
Architecture	Event-Driven Architecture
Domain	Identity
ESP	ESP-0001
ADR principal	ADR-IDENTITY-007
ADR relacionado	ADR-IDENTITY-006
ADR relacionado	ADR-IDENTITY-010
ADR relacionado	ADR-IDENTITY-011
Implementation Story	IS-IDENTITY-013
Technical Tasks	T01–T64
Source Code	Pendiente
Tests	Pendiente
Deployment	Pendiente

Esto mantiene la trazabilidad definida por el Blueprint:

ESP
 ↓
IS
 ↓
Technical Tasks
 ↓
Source Code
 ↓
Testing
 ↓
Deployment

44. ADRs que quedan pendientes

IS-013 identifica varias decisiones que no debemos inventar todavía.

ADR-IDENTITY-012

Event Platform Technology & Topology

Definir:

broker;
topology;
topics/streams;
partitions;
consumers;
delivery guarantees.
ADR-IDENTITY-013

Event Retention & Replay

Definir:

retention;
archival;
replay windows;
legal requirements;
deletion/anonymization.
ADR-IDENTITY-014

Event Schema Governance

Definir:

schema registry;
compatibility;
versioning;
validation;
evolution.
ADR-IDENTITY-015

Outbox Relay & Concurrency Strategy

Definir:

polling;
locking;
leasing;
workers;
batching;
concurrency.
ADR-IDENTITY-016

Consumer Idempotency Strategy

Definir:

deduplication store;
retention;
transactional strategy;
exactly-once business effects.
ADR-IDENTITY-017

Event Security & Privacy

Definir:

encryption;
PII minimization;
sensitive-event handling;
tenant boundaries;
access to event streams.
45. Estado de la especificación

IS-IDENTITY-013 queda en estado Draft.

No recomiendo pasar todavía a implementación porque aún faltan decisiones de infraestructura que afectan directamente el diseño técnico del Outbox Publisher, especialmente:

Event Platform
      ↓
Topology
      ↓
Partitioning
      ↓
Delivery
      ↓
Retry
      ↓
DLQ
      ↓
Replay
      ↓
Consumer Idempotency

Esto es consistente con la regla del Blueprint de no saltarse el ciclo:

Blueprint → Architecture → Standards → ESP → IS → Technical Tasks → Source Code → Testing → Deployment.

Siguiente paso recomendado

Después de IS-IDENTITY-013, el siguiente bloque lógico es IS-IDENTITY-014 — Identity API Contracts, porque permitirá convertir todas las capacidades Identity que ya definimos en contratos API formales y trazables antes de comenzar a programar.
