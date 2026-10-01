no debe crear dos veces:

PaymentTransaction
17. Event Inbox

Puede utilizarse un Inbox:

Event
 ↓
Inbox
 ↓
Check event_id

Estados:

RECEIVED
PROCESSING
PROCESSED
FAILED
18. Transactional Event Processing

Cuando sea posible:

BEGIN
   │
   ├── Mark event processed
   ├── Apply business change
   └── Create outgoing event
COMMIT

Esto reduce inconsistencias.

19. Event Routing

Una vez validado:

Event
  ↓
Router

decide qué processor debe ejecutarlo.

Ejemplo:

WorkoutCompleted
       │
       ├── ProgressProcessor
       ├── AnalyticsProcessor
       └── NotificationProcessor
20. Routing Rules

Las reglas pueden basarse en:

Event Type
Tenant
Domain
Priority
Version
Source
Business Context
21. Event Filtering

No todos los consumidores necesitan todos los eventos.

Ejemplo:

WorkoutCompleted

puede interesar a:

Progress
Analytics
Coaching

pero no necesariamente a:

Billing
22. Event Transformation

Un evento puede transformarse:

External Event
      ↓
Normalizer
      ↓
EVOXA Event

Esto es especialmente importante para eventos externos.

23. Event Normalization

Diferentes proveedores pueden utilizar:

status = "paid"
status = "completed"
status = "captured"

EVOXA puede normalizarlos a:

PaymentCaptured
24. Event Enrichment

Un evento puede requerir información adicional:

WorkoutCompleted
      ↓
Enrichment
      ↓
User Context
Training Context
Tenant Context

Debe evitarse enriquecer indiscriminadamente con datos innecesarios.

25. Enrichment Sources

Los datos pueden provenir de:

Database
Cache
Repository
External Provider
Identity Context
Tenant Configuration
26. Enrichment Risk

El enriquecimiento introduce dependencias.

Ejemplo:

Event
 ↓
Database lookup
 ↓
External API
 ↓
Processing

Esto puede aumentar:

Latency
Failure Rate
Cost

Por eso debe aplicarse solamente cuando sea necesario.

27. Event Processor

El Processor es responsable de:

Receive Event
Validate Context
Execute Business Logic
Handle Result
Emit Events

No debe convertirse en un "God Object".

28. Processor Isolation

Preferible:

WorkoutCompletedProcessor
PaymentCapturedProcessor
SubscriptionActivatedProcessor

en lugar de:

GenericEventProcessor

que contenga toda la lógica de negocio.

29. Application Service Integration

El processor debe delegar la lógica empresarial:

Event
 ↓
Processor
 ↓
Application Service
 ↓
Domain

No debe duplicar reglas del dominio.

30. Event Processing Example
WorkoutCompleted
       │
       ▼
WorkoutCompletedProcessor
       │
       ▼
ProgressApplicationService
       │
       ▼
Progress Domain
       │
       ▼
ProgressUpdated
31. Domain Events

El procesamiento puede generar nuevos eventos:

Event A
  ↓
Processing
  ↓
Domain Change
  ↓
Event B

Ejemplo:

WorkoutCompleted
       ↓
GoalEvaluation
       ↓
GoalAchieved
32. Event Chains

Las cadenas deben estar controladas:

A
 ↓
B
 ↓
C
 ↓
D

Evitar cadenas circulares:

A → B → C → A
33. Event Loop Protection

EVOXA debe detectar posibles loops mediante:

causation_id
correlation_id
event lineage
maximum depth
34. Event Lineage

Debe ser posible conocer:

What caused this event?

Ejemplo:

UserAction
    ↓
WorkoutCompleted
    ↓
ProgressUpdated
    ↓
GoalAchieved
    ↓
AchievementGranted
35. Event Correlation

Una operación completa puede compartir:

correlation_id

permitiendo reconstruir:

Request
 ↓
Command
 ↓
Event
 ↓
Processing
 ↓
Notification
36. Event Causation

Cada evento derivado puede registrar:

causation_id

para identificar su evento origen.

37. Event Aggregation

Algunos procesos requieren agrupar eventos.

Ejemplo:

WorkoutCompleted
WorkoutCompleted
NutritionLogged
SleepRecorded

pueden alimentar:

DailyUserActivity
38. Event Windowing

El procesamiento puede utilizar ventanas:

1 minute
5 minutes
1 hour
1 day

según el caso.

Ejemplo:

Events during day
       ↓
Daily Activity Summary
39. Temporal Processing

Algunos eventos deben procesarse según tiempo:

Occurred At
Received At
Scheduled At
Expiration At

No deben confundirse.

40. Late Events

Un evento puede llegar tarde:

Occurred: 10:00
Received: 10:30

EVOXA debe definir si:

Process
Ignore
Recalculate
Compensate

según el dominio.

41. Out-of-Order Events

Ejemplo:

Event 2
Event 1
Event 3

El processor debe soportar orden fuera de secuencia cuando sea posible.

42. Sequence Numbers

Para dominios sensibles al orden:

sequence_number

puede utilizarse.

Ejemplo:

1 → Created
2 → Activated
3 → Cancelled
43. Event State

El procesamiento puede registrar:

RECEIVED
VALIDATED
ROUTED
PROCESSING
PROCESSED
FAILED
RETRYING
DEAD_LETTER
44. Processing Failure

No todos los errores deben generar retry.

Transient
Database unavailable
Network timeout
External 503

→ Retry.

Permanent
Invalid schema
Business rule violation
Unknown event version

→ DLQ / manual intervention.

45. Retry Policy

Cada processor puede definir:

max_attempts
backoff
jitter
retryable_errors
46. Retry Isolation

Un evento que falla repetidamente no debe bloquear todos los demás.

Event A → FAILED
Event B → SUCCESS
Event C → SUCCESS

La cola debe permitir aislamiento cuando la tecnología lo soporte.

47. Dead Letter

Después de agotar retries:

Processor
   ↓
Retry
   ↓
Retry
   ↓
DLQ

Debe conservarse información suficiente para diagnóstico.

48. Dead Letter Metadata
event_id
event_type
original_queue
failure_reason
attempt_count
first_failed_at
last_failed_at
processor
49. Event Replay

Los eventos persistidos pueden ser reprocesados:

Historical Events
       ↓
Replay
       ↓
Processor

Usos:

Recovery
Read Model Rebuild
Analytics Reprocessing
Bug Fix
Migration
50. Replay Controls

El replay debe permitir:

By Event Type
By Tenant
By Time Range
By Aggregate
By Event ID
51. Replay Safety

Nunca asumir que replay es seguro automáticamente.

Debe verificarse:

Idempotency
Side Effects
External Calls
Notifications
Payments
52. Side Effect Protection

Durante replay, puede ser necesario:

Disable Notifications
Disable External Calls
Use Simulation Mode

según el caso.

53. Dry Run

El Event Processor puede soportar:

dry_run = true

para analizar qué ocurriría sin ejecutar efectos irreversibles.

54. Event Projection

Los eventos pueden construir modelos de lectura:

Events
  ↓
Projection Processor
  ↓
Read Model

Ejemplo:

WorkoutCompleted
NutritionLogged
GoalUpdated
       ↓
Progress Dashboard
55. Projection Rebuild

Un read model puede reconstruirse:

Events
   ↓
Replay
   ↓
Projection
   ↓
New Read Model

Esto permite corregir proyecciones sin modificar el historial original.

56. Event Sourcing Compatibility

EVOXA no necesita convertir todo su sistema a Event Sourcing.

Pero la arquitectura debe permitir utilizar eventos como fuente histórica en dominios seleccionados cuando exista una razón arquitectónica.

57. Event Store

Cuando sea necesario puede existir:

Event Store

para:

Immutable Events
Versioning
Replay
Audit
Reconstruction

No todos los eventos necesitan almacenarse indefinidamente.

58. Event Retention

Las políticas pueden ser:

Short
Medium
Long
Permanent

dependiendo de:

Business
Compliance
Analytics
Recovery
59. Event Compaction

Para algunos flujos puede utilizarse:

Event Compaction

cuando el historial completo no sea necesario para consumidores determinados.

60. Event Processing and Multi-Tenancy

Todo processor debe conocer el contexto:

tenant_id

cuando el evento sea tenant-scoped.

Event
 ↓
Tenant Context
 ↓
Tenant Authorization
 ↓
Processing
61. Cross-Tenant Protection

Nunca permitir:

Tenant A Event
      ↓
Processor
      ↓
Tenant B Aggregate

Las referencias deben verificarse antes de modificar estado.

62. Event Authorization

Los processors deben aplicar las políticas correspondientes.

Event
 ↓
Policy
 ↓
Allowed?

No asumir que un evento válido es automáticamente autorizado.

63. Event Processing Security

Debe protegerse:

Event Integrity
Event Authenticity
Tenant Isolation
Payload Confidentiality
Replay Abuse
Injection
64. Event Injection Protection

Los eventos externos deben validarse contra schemas estrictos.

No ejecutar directamente contenido arbitrario:

External Payload
       ↓
Validation
       ↓
Safe Internal Model
65. AI Event Processing

Eventos generados por IA deben tratarse como datos no confiables.

Ejemplo:

AIRecommendationGenerated
       ↓
Schema Validation
       ↓
Policy Validation
       ↓
Domain Validation
66. Event Processing and AI

Un flujo:

GenerateTrainingPlan
       ↓
AI Worker
       ↓
AI Provider
       ↓
AIResponseReceived
       ↓
Event Processor
       ↓
Validation
       ↓
TrainingPlanCreated
67. Event Processing and Notifications
GoalAchieved
      ↓
NotificationProcessor
      ↓
NotificationApplicationService
      ↓
Notification Queue

El processor no debe enviar directamente email/SMS si la arquitectura requiere desacoplamiento.

68. Event Processing and Billing
PaymentCaptured
       ↓
BillingProcessor
       ↓
SubscriptionApplicationService
       ↓
SubscriptionActivated
69. Event Processing and Analytics
WorkoutCompleted
       ↓
AnalyticsProcessor
       ↓
Analytics Model

Esto permite separar:

Transactional Processing

de:

Analytical Processing
70. Event Processing and Integration

E11 y E13 se conectan:

External Provider
       ↓
Integration Adapter
       ↓
Normalized Event
       ↓
Event Processing
71. Event Processing and Messaging

E12 y E13 forman:

Messaging
    │
    ▼
Transport
    │
    ▼
Event Processing
    │
    ▼
Business Execution
72. Processing Concurrency

Los processors deben controlar:

Concurrency
Locks
Partitions
Aggregate Boundaries

para evitar condiciones de carrera.

73. Optimistic Concurrency

Cuando corresponda:

Aggregate Version

puede utilizarse:

Expected Version
       ↓
Actual Version
       ↓
Match?
74. Event Processing Lock

Para procesos que requieren exclusividad:

Event
 ↓
Distributed Lock
 ↓
Process

Debe evitarse utilizar locks distribuidos innecesariamente.

75. Event Ordering per Aggregate

Una estrategia preferida:

Aggregate ID
      ↓
Partition
      ↓
Ordered Events

Esto permite escalar manteniendo orden local.

76. Batch Processing

Los eventos pueden procesarse individualmente o en batch:

Event 1
Event 2
Event 3
      ↓
Batch Processor

El batch puede mejorar throughput, pero debe preservar correctamente las garantías de procesamiento.

77. Bulk Processing

Para grandes volúmenes:

10
100
1,000
10,000

eventos pueden procesarse mediante workers especializados.

78. Backpressure

Cuando aumenta la carga:

Event Arrival
████████████████
       ↓
Processor
████

EVOXA debe controlar:

Queue Depth
Concurrency
Rate
Worker Scaling
79. Processing Priorities

Los eventos pueden clasificarse:

CRITICAL
HIGH
NORMAL
LOW

Ejemplo:

PaymentCaptured
→ HIGH

AnalyticsUpdate
→ LOW
80. Event Processing Metrics

Debe medirse:

events_received_total
events_processed_total
events_failed_total
events_retried_total
events_dead_lettered_total
processing_duration
processing_latency
queue_lag
replay_count
81. Processor Metrics

Cada processor puede exponer:

processor_invocations
processor_success
processor_failure
processor_duration
processor_retry
82. Event Lag

Debe medirse:

occurred_at
        ↓
processed_at

La diferencia representa:

Event Processing Lag
83. Observability

Cada procesamiento debe poder seguirse:

trace_id
correlation_id
event_id
processor
tenant_id
84. Structured Logging

Ejemplo conceptual:

event_id=abc
event_type=WorkoutCompleted
processor=ProgressProcessor
tenant_id=tenant-01
status=processed
duration=42ms
85. Event Processing Alerts

Alertas recomendadas:

High Processing Failure
High Queue Lag
Growing DLQ
High Retry Rate
Processor Down
Unexpected Event Volume
Schema Validation Failures
86. Event Processing Health

El sistema debe poder responder:

Are events being consumed?
Are processors healthy?
Are queues growing?
Are failures increasing?
Are events delayed?
87. Event Processing Governance

Cada processor debe tener:

Owner
Contract
Schema
Failure Policy
Retry Policy
SLA
Observability
Security Policy
88. Processor Registry

Puede existir conceptualmente:

EventProcessorRegistry

que mantenga:

Event Type
Processor
Version
Status
Owner

Ejemplo:

WorkoutCompleted
→ ProgressProcessor
→ AnalyticsProcessor
→ NotificationProcessor
89. Event Compatibility

Cuando cambia un evento:

v1
 ↓
v2

los processors deben tener una estrategia:

Upgrade
Dual Support
Translation
Deprecation
90. Event Deprecation

Un evento antiguo puede pasar por:

Active
 ↓
Deprecated
 ↓
Migration
 ↓
Retired
91. Event Processing Lifecycle
Event Created
      ↓
Published
      ↓
Received
      ↓
Validated
      ↓
Deduplicated
      ↓
Routed
      ↓
Enriched
      ↓
Authorized
      ↓
Processed
      ↓
Persisted
      ↓
Acknowledged

Error:

Processing
    ↓
Retry
    ↓
Retry Exhausted
    ↓
Dead Letter
92. Event Processing Architecture

La arquitectura consolidada:

                         EVENT
                           │
                           ▼
                    Message Broker
                           │
                           ▼
                  Event Ingestion
                           │
                           ▼
                    Authentication
                           │
                           ▼
                      Validation
                           │
                           ▼
                     Deduplication
                           │
                           ▼
                      Normalizer
                           │
                           ▼
                        Router
                           │
             ┌─────────────┼─────────────┐
             ▼             ▼             ▼
         Processor      Processor      Processor
             │             │             │
             └─────────────┼─────────────┘
                           ▼
                   Application Service
                           │
                           ▼
                         Domain
                           │
                 ┌─────────┴─────────┐
                 ▼                   ▼
             Repository           Outbox
                 │                   │
                 ▼                   ▼
             Database             Events
93. Relationship E10–E13

La cadena de Engineering Specification ahora queda:

E10 Repository
       │
       ▼
Persistence
       │
       ▼
E11 Integration
       │
       ▼
External Systems
       │
       ▼
E12 Messaging
       │
       ▼
Message Transport
       │
       ▼
E13 Event Processing
       │
       ▼
Business Processing
       │
       ▼
Domain / Application
94. Architectural Principle

La regla fundamental de E13 es:

El transporte de un evento y el procesamiento de un evento son responsabilidades diferentes.

Por lo tanto:

E12
→ Transport

E13
→ Processing

Esto permite evolucionar independientemente:

Broker Technology

sin tener que reconstruir:

Event Processing Logic
95. Definition of Done

E13 queda definido cuando EVOXA dispone de:

✓ Event Processing Boundary
✓ Event Ingestion
✓ Event Envelope
✓ Event Validation
✓ Event Authentication
✓ Event Authorization
✓ Event Deduplication
✓ Inbox Pattern
✓ Transactional Processing
✓ Event Routing
✓ Event Filtering
✓ Event Transformation
✓ Event Normalization
✓ Event Enrichment
✓ Processor Architecture
✓ Application Service Integration
✓ Domain Event Integration
✓ Event Chains
✓ Event Loop Protection
✓ Event Lineage
✓ Correlation
✓ Causation
✓ Event Aggregation
✓ Windowing
✓ Temporal Processing
✓ Late Event Handling
✓ Out-of-Order Handling
✓ Sequence Management
✓ Processing States
✓ Retry Strategy
✓ Failure Classification
✓ Dead Letter Handling
✓ Event Replay
✓ Replay Controls
✓ Replay Safety
✓ Side Effect Protection
✓ Dry Run
✓ Event Projection
✓ Projection Rebuild
✓ Event Store Compatibility
✓ Event Retention
✓ Event Compaction
✓ Multi-Tenant Processing
✓ Tenant Isolation
✓ Security
✓ AI Event Processing
✓ Notification Processing
✓ Billing Processing
✓ Analytics Processing
✓ Integration Processing
✓ Concurrency Control
✓ Ordering per Aggregate
✓ Batch Processing
✓ Backpressure
✓ Processing Priority
✓ Metrics
✓ Event Lag Monitoring
✓ Distributed Tracing
✓ Structured Logging
✓ Alerting
✓ Processor Health
✓ Processor Registry
✓ Event Compatibility
✓ Event Deprecation
✓ Event Lifecycle
✓ Governance
96. Engineering Specification Progress
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
E13 — Event Processing Architecture

La evolución arquitectónica ahora es:

             EVOXA ENGINEERING
                    │
                    ▼
             Application Layer
                    │
          ┌─────────┴─────────┐
          ▼                   ▼
     Repositories        Integrations
          │                   │
          ▼                   ▼
       Database          External World
                             
                    │
                    ▼
                Messaging
                    │
                    ▼
            Event Processing
                    │
                    ▼
             Business Logic
                    │
          ┌─────────┴─────────┐
          ▼                   ▼
        Domain            Application

E13 establece el motor de procesamiento reactivo de EVOXA: recibe eventos, valida su contexto, elimina duplicados, enruta, enriquece, ejecuta la lógica correspondiente, controla errores y produce nuevos eventos de manera observable, resiliente y compatible con el modelo multi-tenant.

Siguiente capítulo: E14 — EVOXA Workflow & Orchestration Architecture.
