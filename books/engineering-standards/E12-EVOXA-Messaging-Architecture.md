E12 — EVOXA Messaging Architecture
Engineering Specification

Pertenece a: Engineering Specification
Volumen: volume-09-engineering
Capítulo: E12 — Messaging Architecture
Anterior: E11 — Integration Architecture
Siguiente: E13 — Event Processing Architecture

1. Propósito

E12 define la arquitectura de mensajería de EVOXA.

Mientras E11 establece cómo EVOXA se integra con sistemas externos, E12 establece cómo los componentes de EVOXA intercambian información de manera:

asíncrona,
desacoplada,
confiable,
escalable,
observable,
resiliente.

La pregunta central es:

¿Cómo puede EVOXA transportar comandos, eventos y mensajes entre sus componentes sin crear dependencias directas entre productores y consumidores?

2. Objetivos

Messaging Architecture debe proporcionar:

Asynchronous Communication
Message Delivery
Event Distribution
Command Delivery
Queueing
Pub/Sub
Retry
Dead Letter Handling
Ordering
Idempotency
Delivery Guarantees
Backpressure
Message Persistence
Schema Management
Observability
Security
Scalability
3. Principio Fundamental

Los componentes deben comunicarse mediante contratos de mensajería.

Producer
    │
    ▼
Message Contract
    │
    ▼
Broker
    │
    ├──────────────┐
    ▼              ▼
Consumer A      Consumer B

El productor no necesita conocer la implementación interna del consumidor.

4. Messaging Boundary
┌─────────────────────────────┐
│ EVOXA Application           │
│                             │
│ Commands / Events           │
└──────────────┬──────────────┘
               │
═══════════════╪══════════════
      Messaging Boundary
═══════════════╪══════════════
               │
               ▼
        Message Broker
               │
       ┌───────┼───────┐
       ▼       ▼       ▼
   Consumer Consumer Consumer
5. Messaging vs Integration

E11:

Integration Architecture
→ Comunicación con sistemas externos

E12:

Messaging Architecture
→ Comunicación mediante mensajes

Ambas pueden coexistir:

EVOXA
  │
  ▼
Messaging
  │
  ▼
Integration Worker
  │
  ▼
External System
6. Tipos de Mensajes

EVOXA debe distinguir:

Command
Event
Notification
Request
Response
7. Command

Un Command expresa una intención:

Do X

Ejemplos:

GenerateTrainingPlan
ProcessPayment
SendNotification
CalculateNutrition

Normalmente tiene:

One Producer
One Logical Consumer
8. Event

Un Event expresa algo que ya ocurrió:

Something happened

Ejemplos:

WorkoutCompleted
PaymentCaptured
TrainingPlanCreated
SubscriptionActivated

Puede tener:

One Producer
Many Consumers
9. Notification

Una Notification informa sobre un cambio que puede no requerir procesamiento complejo.

Ejemplo:

TrainingReminder

Puede utilizar canales como:

Email
Push
SMS
In-App
10. Request / Response

Aunque Messaging es principalmente asíncrono, algunos brokers pueden soportar:

Request
   ↓
Consumer
   ↓
Response

Debe utilizarse con cuidado porque introduce mayor acoplamiento temporal.

11. Event-Driven Architecture

EVOXA puede utilizar un modelo:

Domain Event
      ↓
Application Event
      ↓
Message Broker
      ↓
Subscribers

Esto permite construir capacidades desacopladas.

12. Event Example

Cuando un usuario completa un entrenamiento:

WorkoutCompleted

puede producir:

Workout Service
       │
       ▼
Event Broker
       │
 ┌─────┼────────────┐
 ▼     ▼            ▼
Progress Analytics Notification
13. Producer

El Producer es responsable de:

Create Message
Validate Contract
Assign Metadata
Publish Message

No debe conocer la implementación de los consumidores.

14. Consumer

Un Consumer es responsable de:

Receive Message
Validate Message
Process Message
Acknowledge
Retry when appropriate
Handle Failure
15. Broker

El Message Broker proporciona:

Routing
Delivery
Queueing
Persistence
Fan-out
Acknowledgement
Retry Support

El broker es infraestructura.

16. Broker Abstraction

EVOXA no debería acoplar el dominio a un broker concreto.

Puede existir:

MessageBus

o:

EventBus
CommandBus

como abstracción.

17. Broker Technologies

La arquitectura debe permitir utilizar tecnologías como:

Redis Streams
RabbitMQ
Kafka
Cloud Messaging
Managed Queue Services

La elección concreta pertenece a Infrastructure/Deployment Architecture.

18. Redis Messaging

Redis puede utilizarse para determinados escenarios:

Redis Streams
Pub/Sub
Queues

pero debe distinguirse:

Cache

de:

Durable Messaging

Redis Pub/Sub, por ejemplo, no debe tratarse automáticamente como un sistema durable de eventos.

19. Kafka-style Messaging

Para grandes volúmenes:

Producer
   ↓
Topic
   ↓
Partition
   ↓
Consumer Group

puede utilizarse un modelo de streaming.

20. Queue

Una Queue representa trabajo pendiente.

Producer
   ↓
Queue
   ↓
Worker

Normalmente cada mensaje es procesado por un consumidor lógico.

21. Topic

Un Topic representa un flujo de eventos.

Producer
   ↓
Topic
 ┌─┼─────────┐
 ▼ ▼         ▼
A B          C

Permite múltiples consumidores.

22. Queue vs Topic
Queue
One logical consumer
Topic
Multiple subscribers
23. Consumer Groups

Para escalar:

Topic
   │
   ▼
Consumer Group
 ┌────┬────┬────┐
 ▼    ▼    ▼
W1   W2   W3

El trabajo se distribuye entre workers.

24. Message Contract

Cada mensaje debe poseer un contrato explícito.

Ejemplo conceptual:

Message
├── id
├── type
├── version
├── timestamp
├── producer
├── correlation_id
├── causation_id
└── payload
25. Message ID

Cada mensaje debe tener un identificador único:

message_id

Sirve para:

Deduplication
Tracing
Auditing
Troubleshooting
26. Correlation ID

El:

correlation_id

permite seguir una operación completa.

Ejemplo:

API Request
   │
   ▼
Command
   │
   ▼
Event
   │
   ▼
Notification

Todos pueden compartir el mismo correlation ID.

27. Causation ID

El:

causation_id

identifica el mensaje que originó otro mensaje.

Ejemplo:

Command A
   ↓
Event B
   ↓
Command C

Entonces:

C.causation_id = B.message_id
28. Message Metadata

Metadata recomendada:

message_id
message_type
schema_version
timestamp
producer
tenant_id
correlation_id
causation_id
trace_id
29. Tenant Context

En EVOXA multi-tenant, los mensajes deben transportar contexto de tenant cuando corresponda:

tenant_id

Esto permite mantener aislamiento durante procesamiento asíncrono.

30. Tenant Isolation

Nunca debe ocurrir:

Tenant A Event
      ↓
Consumer
      ↓
Tenant B Data

Los consumers deben validar el contexto antes de procesar.

31. Message Schema

Los schemas deben ser explícitos.

Ejemplo:

WorkoutCompleted.v1

con:

workout_id
user_id
completed_at
duration
32. Schema Versioning

Los mensajes deben poder evolucionar:

WorkoutCompleted.v1
WorkoutCompleted.v2

sin romper consumidores existentes.

33. Backward Compatibility

Cuando sea posible:

v2

debe mantener compatibilidad con consumidores de:

v1

durante un período de transición.

34. Schema Registry

Para arquitecturas grandes puede utilizarse:

Schema Registry

para administrar:

Schemas
Versions
Compatibility
Validation
35. Event Naming

Los eventos deben utilizar lenguaje de negocio.

Preferible:

WorkoutCompleted
SubscriptionActivated
PaymentCaptured

Evitar:

WorkoutRowUpdated
SubscriptionTableChanged
PaymentRecordInserted
36. Command Naming

Los Commands deben expresar intención:

CompleteWorkout
GenerateTrainingPlan
ActivateSubscription
ProcessPayment
37. Message Ownership

Cada mensaje debe tener un productor claramente definido.

Workout Service
→ WorkoutCompleted

No múltiples productores generando el mismo evento semántico sin una razón clara.

38. Event Ownership

El dominio que posee el estado debe publicar los eventos relacionados con ese estado.

Ejemplo:

Training Domain
→ TrainingPlanCreated
39. Consumer Ownership

Cada consumer debe tener:

Business Owner
Technical Owner
Failure Policy
Retry Policy
40. Delivery Semantics

EVOXA debe considerar:

At-most-once
At-least-once
Exactly-once
41. At-Most-Once

El mensaje puede perderse, pero no procesarse dos veces.

Deliver
  ↓
ACK
  ↓
Process

Útil únicamente para determinadas notificaciones no críticas.

42. At-Least-Once

El mensaje puede procesarse más de una vez.

Deliver
  ↓
Process
  ↓
ACK

Si falla el ACK:

Redelivery

Este modelo es frecuentemente preferible porque favorece la durabilidad.

43. Exactly-Once

Debe tratarse con mucha cautela.

En sistemas distribuidos, "exactly once" suele requerir garantías adicionales.

EVOXA no debe asumir exactly-once simplemente porque el broker lo declare.

44. Idempotent Consumer

Los consumidores deben ser idempotentes.

Event A
Event A
Event A

debe producir:

One Business Effect
45. Inbox Pattern

Un Consumer puede mantener:

Inbox

para registrar mensajes procesados.

Message
  ↓
Inbox
  ↓
Already processed?
 ├── Yes → Ignore
 └── No  → Process
46. Transactional Inbox

Puede combinarse:

BEGIN
  Save message_id
  Apply business change
COMMIT

Esto evita procesar nuevamente el mismo mensaje después de un crash.

47. Outbox Pattern

EVOXA debe utilizar Outbox para publicar eventos relacionados con cambios transaccionales.

BEGIN
 ├── Domain Change
 └── Outbox Message
COMMIT

Luego:

Outbox Worker
      ↓
Message Broker
48. Outbox + Repository

La relación con E10:

Application Service
       │
       ├── Repository
       │
       └── Outbox Repository
               │
               ▼
             Commit

Ambos cambios deben pertenecer a la misma transacción cuando corresponda.

49. Reliable Publishing

Sin Outbox:

Database Commit
      ↓
Broker Publish
      ↓
Failure

Resultado:

Database changed
Event lost

Con Outbox:

Database Change
+
Outbox Event
      ↓
Atomic Commit
      ↓
Publisher
50. Message Retry

Los mensajes fallidos deben clasificarse.

Transient
Retry
Permanent
Dead Letter
51. Retry Backoff

Ejemplo:

Attempt 1 → 1s
Attempt 2 → 5s
Attempt 3 → 30s
Attempt 4 → 5m

Los valores reales dependen del caso.

52. Dead Letter Queue

Después de agotar retries:

Queue
  ↓
Retry
  ↓
Retry
  ↓
DLQ
53. Dead Letter Management

Una DLQ debe permitir:

Inspect
Diagnose
Replay
Discard
Repair
Audit
54. Poison Message

Un mensaje que siempre falla se denomina:

Poison Message

No debe bloquear indefinidamente toda la cola.

55. Message Ordering

Algunos dominios requieren orden.

Ejemplo:

SubscriptionCreated
       ↓
SubscriptionActivated
       ↓
SubscriptionCancelled

No debería procesarse:

Cancelled

antes de:

Created
56. Ordering Strategy

Puede utilizarse:

Partition Key
Aggregate ID
Sequence Number

Ejemplo:

partition_key = subscription_id
57. Ordering Scope

No intentar garantizar orden global si no es necesario.

Preferible:

Order per Aggregate

en lugar de:

Global Ordering

porque escala mejor.

58. Message Deduplication

La deduplicación puede utilizar:

message_id
event_id
provider_event_id
business_key

según el origen.

59. Message Expiration

Algunos mensajes pueden tener TTL.

Ejemplo:

WorkoutReminder

Si llega tres días tarde:

Discard
60. Priority

No todos los mensajes tienen la misma prioridad.

Ejemplo:

CRITICAL
HIGH
NORMAL
LOW

Debe utilizarse solamente cuando realmente sea necesario.

61. Backpressure

Si los consumidores procesan más lentamente que los productores:

Producer Rate
      ↓
████████████
      ↓
Consumer Rate
      ↓
████

la cola crecerá.

EVOXA debe controlar:

Concurrency
Queue Size
Rate
Autoscaling
62. Consumer Concurrency

Puede escalarse horizontalmente:

Queue
 ├── Worker 1
 ├── Worker 2
 ├── Worker 3
 └── Worker 4
63. Autoscaling

El número de workers puede basarse en:

Queue Depth
Message Age
Processing Latency
CPU
64. Message Processing Time

Los consumers deben medir:

processing_duration

para detectar degradación.

65. Long-Running Jobs

Trabajos largos deben evitar bloquear consumers generales.

Ejemplos:

AI Batch Processing
Report Generation
Video Processing
Large Data Import

Pueden utilizar:

Dedicated Queue
Dedicated Workers
66. Message Routing

El broker puede enrutar por:

Topic
Routing Key
Event Type
Tenant
Priority
Capability
67. Routing Example
WorkoutCompleted
      │
      ▼
training.events
      │
 ┌────┼─────┐
 ▼    ▼     ▼
Progress Analytics Coaching
68. Broadcast

Cuando muchos consumidores necesitan el mismo evento:

Event
 ↓
Topic
 ├── Progress
 ├── Analytics
 ├── Coaching
 └── Notifications
69. Point-to-Point

Cuando existe una única responsabilidad:

Command
 ↓
Queue
 ↓
TrainingWorker
70. Message Security

Los mensajes deben proteger:

Confidentiality
Integrity
Authentication
Authorization
71. Sensitive Messages

No colocar innecesariamente en mensajes:

Passwords
API Keys
Tokens
Payment Credentials
Sensitive Personal Data
72. Message Encryption

Cuando corresponda:

Encryption in Transit
Encryption at Rest

debe ser responsabilidad de la infraestructura.

73. Message Access Control

No todos los consumidores deben poder consumir todos los topics.

TrainingConsumer
→ training.events

BillingConsumer
→ billing.events
74. Tenant-Aware Messaging

Puede utilizarse:

tenant_id

en metadata o payload según el diseño.

El sistema debe impedir que un consumidor procese accidentalmente mensajes de otro tenant cuando el aislamiento sea obligatorio.

75. Messaging Observability

Cada mensaje debe poder rastrearse.

Producer
   ↓
message_id
   ↓
Broker
   ↓
Consumer
   ↓
processing
76. Messaging Metrics

Métricas:

messages_published_total
messages_consumed_total
messages_failed_total
messages_retried_total
messages_dead_lettered_total
message_processing_duration
queue_depth
message_age
77. Distributed Tracing

Los mensajes deben propagar:

trace_id
correlation_id
causation_id

para conectar:

HTTP
→ Command
→ Event
→ Consumer
→ Database
78. Logging

Los logs deben registrar:

message_id
message_type
consumer
status
duration
correlation_id

pero no payloads sensibles completos.

79. Message Audit

Para mensajes críticos puede mantenerse:

Message Audit Record

con:

message_id
type
producer
timestamp
status
attempts
80. Messaging Health

La plataforma debe observar:

Broker Connectivity
Queue Depth
Consumer Health
DLQ Size
Message Latency
Processing Failures
81. Broker Failure

Si el broker está temporalmente indisponible:

Producer
   ↓
Publish Failure

EVOXA debe definir:

Retry
Local Buffer
Outbox
Fail Fast

según criticidad.

82. Consumer Failure

Si un consumer falla:

Message
 ↓
Consumer
 ↓
Crash

el sistema debe permitir:

Redelivery
Retry
Recovery

cuando la semántica lo requiera.

83. Consumer Deployment

Los consumers deben poder desplegarse independientemente cuando sea conveniente:

API
Worker
Event Consumer
Scheduler
84. Worker Architecture

Una arquitectura típica:

Application
     │
     ▼
Message Queue
     │
     ▼
Worker
     │
     ├── Application Service
     ├── Repository
     └── Integration
85. Messaging and Domain Events

El Domain Event:

WorkoutCompleted

puede convertirse en un Integration/Event message:

WorkoutCompleted.v1

La transformación debe ser controlada.

86. Internal vs External Events
Internal Event
Dentro de EVOXA
Integration Event
Diseñado para cruzar una frontera

No asumir que ambos contratos deben ser idénticos.

87. Event Publication

Flujo:

Domain Event
     ↓
Application Layer
     ↓
Integration Event
     ↓
Outbox
     ↓
Broker
88. Message Translation

Un evento interno puede:

WorkoutCompleted

convertirse en:

training.workout.completed.v1

para consumidores externos.

89. Messaging and APIs

API y Messaging son complementarios.

API
→ Immediate interaction

Messaging
→ Asynchronous interaction

Ejemplo:

POST /training-plans
       ↓
202 Accepted
       ↓
GenerateTrainingPlan command
       ↓
Worker
       ↓
TrainingPlanGenerated
90. Asynchronous API Pattern

Cuando un proceso tarda:

Request
  ↓
Command
  ↓
202 Accepted
  ↓
Job ID

Después:

GET /jobs/{id}

o:

Webhook

o:

Push Notification
91. Messaging and AI

EVOXA puede utilizar mensajería para trabajos AI largos:

GenerateTrainingPlan
       ↓
AI Queue
       ↓
AI Worker
       ↓
AI Provider
       ↓
TrainingPlanGenerated

Esto evita bloquear requests HTTP.

92. Messaging and Notifications
WorkoutCompleted
       ↓
Notification Event
       ↓
Notification Queue
       ↓
Email / Push / SMS

Cada canal puede tener su propio worker.

93. Messaging and Billing
SubscriptionCreated
       ↓
Billing Queue
       ↓
Payment Worker
       ↓
Payment Provider
       ↓
PaymentCaptured
94. Messaging and Analytics

Los eventos operacionales pueden alimentar analytics:

WorkoutCompleted
NutritionLogged
GoalUpdated
SubscriptionChanged

hacia:

Analytics Consumer

Esto evita consultas analíticas pesadas sobre las tablas transaccionales.

95. Event Retention

La retención dependerá del tipo de mensaje:

Transient Command
→ Short retention

Operational Event
→ Medium retention

Audit / Compliance Event
→ Long retention
96. Replay

Los eventos persistentes pueden permitir:

Replay

para:

Rebuild Read Models
Recover Consumers
Reprocess Analytics
Repair Derived State
97. Replay Safety

Los consumers deben diseñarse para que replay no produzca efectos duplicados.

Esto requiere:

Idempotency
Versioning
Business Keys
98. Message Lifecycle
Created
   ↓
Validated
   ↓
Published
   ↓
Delivered
   ↓
Processing
   ↓
Processed

En caso de error:

Processing
   ↓
Retrying
   ↓
Dead Letter
99. Messaging Architecture

Arquitectura consolidada:

                         EVOXA
                           │
                           ▼
                  Application Services
                           │
             ┌─────────────┴─────────────┐
             ▼                           ▼
        Commands                      Events
             │                           │
             └─────────────┬─────────────┘
                           ▼
                     Message Bus
                           │
             ┌─────────────┼─────────────┐
             ▼             ▼             ▼
          Queue          Topic        Stream
             │             │             │
             ▼             ▼             ▼
         Workers       Consumers      Analytics
             │
       ┌─────┼──────┐
       ▼     ▼      ▼
    Domain  Repo  Integration
100. Relationship E10–E12

La secuencia queda:

E10 — Repository Architecture
          │
          ▼
     Persistence
          │
          ▼
E11 — Integration Architecture
          │
          ▼
    External Systems
          │
          ▼
E12 — Messaging Architecture
          │
          ▼
 Asynchronous Communication

Las responsabilidades quedan separadas:

E10
→ Persistir estado

E11
→ Integrarse con sistemas externos

E12
→ Transportar mensajes y eventos
101. Definition of Done

E12 queda definido cuando EVOXA dispone de:

✓ Messaging Boundary
✓ Message Types
✓ Commands
✓ Events
✓ Notifications
✓ Producer Model
✓ Consumer Model
✓ Broker Abstraction
✓ Queue Model
✓ Topic Model
✓ Consumer Groups
✓ Message Contracts
✓ Message Metadata
✓ Correlation IDs
✓ Causation IDs
✓ Tenant Context
✓ Schema Versioning
✓ Schema Registry Strategy
✓ Event Naming
✓ Command Naming
✓ Delivery Semantics
✓ At-Least-Once Strategy
✓ Idempotent Consumers
✓ Inbox Pattern
✓ Outbox Pattern
✓ Reliable Publishing
✓ Retry Strategy
✓ Dead Letter Queue
✓ Poison Message Handling
✓ Ordering
✓ Deduplication
✓ TTL
✓ Priority
✓ Backpressure
✓ Consumer Scaling
✓ Autoscaling
✓ Routing
✓ Broadcast
✓ Point-to-Point
✓ Message Security
✓ Sensitive Data Protection
✓ Tenant Isolation
✓ Observability
✓ Metrics
✓ Distributed Tracing
✓ Audit
✓ Broker Failure Handling
✓ Consumer Failure Handling
✓ Worker Architecture
✓ Internal Events
✓ Integration Events
✓ API/Messaging Integration
✓ Async API Pattern
✓ AI Messaging
✓ Notification Messaging
✓ Billing Messaging
✓ Analytics Messaging
✓ Retention
✓ Replay
✓ Replay Safety
✓ Message Lifecycle
✓ Governance
102. Engineering Specification Progress

La secuencia de Engineering Specification queda ahora:

E01 — EVOXA Backend Architecture
E02 — EVOXA Database Architecture
E03 — EVOXA API Architecture
E04 — EVOXA Authentication Architecture
E05 — Authorization Architecture
E06 — Policy Architecture
E07 — EVOXA Service Architecture
E08 — Domain Services Architecture
E09 — Application Services Architecture
E10 — Repository Architecture
E11 — Integration Architecture
E12 — Messaging Architecture

Y la arquitectura completa comienza a verse así:

                         EVOXA
                           │
                    ┌──────┴──────┐
                    ▼             ▼
                  API         Application
                    │             │
                    ▼             ▼
              Authentication   Services
                    │             │
                    ▼             ▼
              Authorization    Domain
                    │             │
                    ▼        ┌────┴────┐
                  Policy     ▼         ▼
                            Repo    Integration
                              │         │
                              ▼         ▼
                          Database   External
                                      Systems

                           │
                           ▼
                     Messaging Layer
                           │
             ┌─────────────┼─────────────┐
             ▼             ▼             ▼
          Commands       Events       Workers

E12 establece la columna vertebral de comunicación asíncrona de EVOXA. A partir de aquí, los siguientes capítulos pueden profundizar en el procesamiento de esos mensajes, incluyendo consumidores, workflows, procesamiento de eventos, jobs y ejecución distribuida.

Siguiente capítulo: E13 — EVOXA Event Processing Architecture.
