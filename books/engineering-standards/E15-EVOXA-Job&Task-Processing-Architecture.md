E15 — EVOXA Job & Task Processing Architecture
Engineering Specification

Pertenece a: Engineering Specification
Volumen: volume-09-engineering
Capítulo: E15 — Job & Task Processing Architecture
Anterior: E14 — Workflow & Orchestration Architecture
Siguiente: E16 — EVOXA Scheduling Architecture

1. Propósito

E15 define la arquitectura responsable de ejecutar trabajos, tareas y unidades de procesamiento asíncrono dentro de EVOXA.

E14 responde:

¿Cómo coordinamos un proceso compuesto por múltiples pasos?

E15 responde:

¿Cómo ejecutamos de forma fiable cada trabajo o tarea que necesita ejecutarse ahora, después o en segundo plano?

La arquitectura debe soportar:

Jobs
Tasks
Workers
Queues
Retries
Priorities
Concurrency
Scheduling Integration
Dead Letters
Timeouts
Cancellation
Idempotency
Persistence
Observability
Horizontal Scaling
Multi-Tenancy
2. Objetivos

El sistema debe permitir ejecutar:

Background Jobs
Async Tasks
Deferred Tasks
Integration Jobs
Processing Jobs
Maintenance Jobs
AI Jobs
Analytics Jobs
Notification Jobs
Data Jobs
System Jobs

de forma:

Reliable
Scalable
Observable
Recoverable
Idempotent
Tenant-aware
Policy-aware
3. Job vs Task

Aunque están relacionados, no son exactamente lo mismo.

Job

Representa una unidad de trabajo gestionada por el sistema:

GenerateWeeklyReport
Task

Representa una operación ejecutable dentro de un job o workflow:

LoadData
GenerateReport
StoreReport

Conceptualmente:

Job
 │
 ├── Task A
 ├── Task B
 └── Task C
4. Job vs Workflow

Un Workflow:

Coordina un proceso empresarial.

Un Job:

Ejecuta trabajo.

Ejemplo:

Workflow
   │
   ├── Create Report Job
   │       ↓
   │    Worker
   │
   ├── Notify User Job
   │       ↓
   │    Worker
   │
   └── Archive Job
           ↓
        Worker
5. Job vs Event

Un Event significa:

Something happened

Un Job significa:

Something must be executed

Ejemplo:

WorkoutCompleted
       ↓
Event

puede producir:

GenerateProgressAnalysis
       ↓
Job
6. Arquitectura General
                  Trigger
                     │
          ┌──────────┼──────────┐
          ▼          ▼          ▼
        Event      Workflow   Schedule
          │          │          │
          └──────────┼──────────┘
                     ▼
                Job Creation
                     │
                     ▼
                Job Queue
                     │
          ┌──────────┼──────────┐
          ▼          ▼          ▼
       Worker A   Worker B   Worker C
          │          │          │
          └──────────┼──────────┘
                     ▼
               Task Execution
                     │
              ┌──────┴──────┐
              ▼             ▼
           Success        Failure
                            │
                    ┌───────┴───────┐
                    ▼               ▼
                  Retry            DLQ
7. Job Lifecycle

Un Job puede atravesar:

CREATED
QUEUED
CLAIMED
RUNNING
COMPLETED
FAILED
RETRYING
CANCELLED
DEAD_LETTER
8. Task Lifecycle

Una Task puede utilizar:

PENDING
READY
RUNNING
COMPLETED
FAILED
RETRYING
CANCELLED
SKIPPED
9. Job Definition

Una definición de Job debe especificar:

job_type
version
handler
input_schema
timeout
retry_policy
priority
concurrency_policy
tenant_scope
10. Job Instance

Una ejecución concreta:

Job Definition
       │
       ▼
Job Instance

Ejemplo:

GenerateReport.v1
       │
       ├── Job #001
       ├── Job #002
       └── Job #003
11. Job Identity

Cada Job debe tener:

job_id
job_type
job_version

Esto permite:

Tracking
Idempotency
Retries
Audit
Recovery
12. Job Context

El contexto debe incluir cuando corresponda:

job_id
job_type
job_version
tenant_id
correlation_id
causation_id
trace_id
created_at
scheduled_at
started_at
completed_at
13. Job Payload

El payload debe ser:

Typed
Validated
Versioned
Minimal

No debe contener secretos innecesarios.

14. Job Queue

La cola desacopla:

Job Producer
      │
      ▼
   Queue
      │
      ▼
Job Worker

Esto permite absorber picos de carga.

15. Queue Responsibilities

La cola proporciona conceptualmente:

Buffering
Delivery
Visibility
Backpressure
Load Distribution

La cola no debe contener lógica empresarial.

16. Worker

El Worker es responsable de:

Claim Job
Validate Context
Execute Handler
Report Result
Handle Failure
17. Worker Model
                 Job Queue
                     │
       ┌─────────────┼─────────────┐
       ▼             ▼             ▼
    Worker A      Worker B      Worker C
       │             │             │
       ▼             ▼             ▼
    Handler       Handler       Handler

Los workers deben poder escalar horizontalmente.

18. Worker Registration

Conceptualmente:

Job Type
   ↓
Handler
   ↓
Worker Capability

Ejemplo:

GenerateProgressAnalysis
        ↓
ProgressAnalysisHandler
19. Handler

El Handler contiene la ejecución específica:

Job
 ↓
Handler
 ↓
Application Service

No debería convertirse en una segunda capa de dominio.

20. Application Service Integration

Preferible:

Worker
  ↓
Job Handler
  ↓
Application Service
  ↓
Domain

No:

Worker
  ↓
Direct Database Mutation
21. Job Creation

Un Job puede crearse desde:

API
Event
Workflow
Schedule
System Trigger
Admin Operation
Another Job
22. Event-Triggered Job
Event
  ↓
Event Processor
  ↓
Create Job
  ↓
Queue
  ↓
Worker

Esto mantiene separados E13 y E15.

23. Workflow-Triggered Job
Workflow
  ↓
Create Job
  ↓
Queue
  ↓
Worker
  ↓
Result
  ↓
Workflow
24. Scheduled Job

E15 puede recibir instrucciones desde E16:

Scheduler
   ↓
Job
   ↓
Queue
   ↓
Worker

El Scheduler decide cuándo.

El Job system decide cómo ejecutar.

25. Job Priority

Los Jobs pueden tener:

CRITICAL
HIGH
NORMAL
LOW

Ejemplo:

PaymentRecovery → HIGH
AnalyticsRebuild → LOW
26. Priority Queues

Puede existir:

High Priority Queue
Normal Queue
Low Priority Queue

o una única cola con prioridad lógica.

La implementación concreta queda abierta.

27. Concurrency

Cada Job puede tener restricciones:

max_concurrency

Ejemplo:

GenerateLargeReport
max_concurrency = 2
28. Global Concurrency

También pueden existir límites globales:

Worker Pool
       │
       ▼
Maximum Concurrent Jobs

Esto protege recursos compartidos.

29. Tenant Concurrency

Los límites pueden aplicarse por tenant:

Tenant A → 10 workers
Tenant B → 5 workers
Tenant C → 5 workers

Esto evita que un tenant monopolice capacidad.

30. Fairness

El sistema debe evitar:

Tenant A
████████████████████

bloqueando:

Tenant B
Tenant C

Puede utilizar:

Fair Scheduling
Per-Tenant Quotas
Weighted Queues
31. Idempotency

Un Job puede ejecutarse más de una vez debido a:

Retry
Worker Crash
Network Failure
Queue Redelivery
Manual Retry

Por tanto:

Los Jobs críticos deben ser idempotentes.

32. Idempotency Key

Puede utilizarse:

idempotency_key

para impedir efectos duplicados.

Ejemplo:

SendWelcomeEmail
idempotency_key = user-123-welcome
33. Job Claiming

Un worker debe reclamar el Job:

QUEUED
   ↓
CLAIMED
   ↓
RUNNING

El claim debe ser seguro frente a concurrencia.

34. Visibility Timeout

Si un worker desaparece:

Worker
  ↓
Claim
  ↓
Crash

el Job debe poder volver a estar disponible después de un timeout controlado.

35. Lease

El claim puede utilizar:

Lease

con:

lease_started_at
lease_expires_at

Esto evita Jobs permanentemente bloqueados.

36. Heartbeat

Para trabajos largos:

Worker
  ↓
Heartbeat
  ↓
Heartbeat
  ↓
Heartbeat

permite indicar que el Job sigue vivo.

37. Long-Running Jobs

Los Jobs largos deben evitar:

Infinite Execution

Deben disponer de:

Timeout
Heartbeat
Cancellation
Recovery
Progress
38. Job Progress

Puede registrarse:

progress = 65%

o:

items_processed = 650
items_total = 1000
39. Retry

Un Job fallido puede reintentarse:

Attempt 1
   ↓
Failure
   ↓
Attempt 2
   ↓
Failure
   ↓
Attempt 3
40. Retry Policy

Debe definirse:

max_attempts
backoff
max_delay
jitter
retryable_errors
41. Exponential Backoff

Ejemplo:

1s
2s
4s
8s
16s

evita bombardear sistemas externos que están fallando.

42. Retry Classification
Retryable
Timeout
Connection Error
503
Temporary Database Failure
Rate Limit
Permanent
Invalid Input
Authorization Failure
Business Rule Violation
Unknown Job Type
43. Dead Letter Queue

Después de agotar retries:

Job
 ↓
Retry
 ↓
Retry
 ↓
Retry Exhausted
 ↓
DLQ

Debe conservarse información suficiente para recuperación.

44. Dead Letter Metadata
job_id
job_type
tenant_id
attempt_count
failure_reason
first_failed_at
last_failed_at
worker
45. Job Cancellation

Un Job puede cancelarse:

QUEUED
   ↓
CANCELLED

o:

RUNNING
   ↓
CANCEL_REQUESTED
   ↓
CANCELLED
46. Cooperative Cancellation

Los workers deben poder comprobar:

Cancellation Requested?

y detener operaciones seguras.

47. Forced Termination

No debe asumirse que todos los Jobs pueden detenerse inmediatamente.

Para operaciones no interrumpibles:

Cancel Requested
      ↓
Current Step Completes
      ↓
Job Stops
48. Job Timeout

Cada Job puede tener:

execution_timeout

Si se supera:

RUNNING
   ↓
TIMEOUT
   ↓
Retry / Fail / DLQ

según política.

49. Task Processing

Un Job puede contener múltiples Tasks:

Job
 │
 ├── Load
 ├── Transform
 ├── Validate
 └── Persist
50. Task Dependencies

Las tareas pueden ser:

Sequential
Parallel
Conditional
Dependent
Optional

Para workflows complejos, la coordinación debe delegarse a E14.

51. Job Composition

E15 debe evitar convertirse en un Workflow Engine.

Regla:

Simple execution
→ Job

Multi-step business process
→ Workflow
52. Job Batch

Un Job puede procesar múltiples elementos:

Batch Job
 ├── Item 1
 ├── Item 2
 ├── Item 3
 └── Item N
53. Batch Failure

Debe definirse si el batch:

Fails Entirely

o permite:

Partial Success

Ejemplo:

100 items
95 success
5 failed
54. Item-Level Retry

Los batches pueden reintentar únicamente los elementos fallidos:

Batch
 ├── 95 ✓
 └── 5 ✗
       ↓
     Retry
55. Chunking

Para datasets grandes:

1,000,000 records
        ↓
Chunks
        ↓
10,000
10,000
10,000
...

Cada chunk puede ser un Job independiente.

56. Fan-Out
Parent Job
    │
    ├── Child Job A
    ├── Child Job B
    ├── Child Job C
    └── Child Job D
57. Fan-In

Después:

A ──┐
B ──┤
C ──┼──> Aggregator
D ──┘

Si el patrón requiere coordinación empresarial compleja, E14 debe controlar el proceso.

58. Job Dependencies

Un Job puede depender de otro:

Job A
 ↓
Job B
 ↓
Job C

Pero dependencias complejas deben modelarse como Workflow.

59. Job Result

El resultado puede ser:

SUCCESS
FAILURE
PARTIAL_SUCCESS
CANCELLED
60. Job Output

El resultado debe evitar almacenar grandes payloads directamente en la cola.

Preferible:

Job
 ↓
Artifact Reference
 ↓
Object Storage / Database
61. Job Artifacts

Un Job puede generar:

Report
File
Dataset
Model
Export
Snapshot

Los artifacts deben gestionarse fuera de la cola.

62. Job Persistence

Debe persistirse información suficiente para:

Recovery
Audit
Retry
Debugging
Observability
63. Job History

Debe registrarse:

Created
Queued
Claimed
Started
Retry
Completed
Failed
Cancelled
64. Job Retention

No todos los Jobs deben conservarse indefinidamente.

Puede definirse:

Short Retention
Long Retention
Audit Retention

según criticidad y compliance.

65. Job Replay

Un Job puede volver a ejecutarse.

Debe diferenciarse:

Retry

de:

Replay

Retry:

misma ejecución que falló.

Replay:

nueva ejecución basada en una ejecución anterior.

66. Safe Replay

Debe evaluarse:

External Side Effects
Payments
Notifications
Integrations
Data Mutations

antes de permitir replay.

67. Dry Run

Puede soportarse:

dry_run = true

para Jobs administrativos o de migración.

68. Job Security

Los Jobs pueden contener información sensible.

Debe protegerse:

Payload
State
Results
Artifacts
Logs
69. Secret Handling

Nunca incluir directamente:

Passwords
API Keys
Access Tokens
Private Keys

en Job Payload.

Usar:

Secret Reference
70. Job Authorization

Debe controlarse quién puede:

Create
Execute
Cancel
Retry
Replay
Inspect
Delete

un Job.

71. Administrative Jobs

Ejemplos:

RebuildProjection
RecalculateProgress
ReprocessEvents
RepairData
GenerateExport

Estos requieren permisos elevados.

72. AI Jobs

Los procesos de IA que puedan tardar o consumir recursos significativos pueden ejecutarse como Jobs:

GenerateTrainingPlan
       ↓
AI Job
       ↓
Worker
       ↓
AI Provider
73. AI Job Controls

Deben controlarse:

Model
Token Budget
Timeout
Retry
Provider
Cost
Output Validation
74. AI Failure

Ejemplos:

Provider Timeout
Rate Limit
Invalid Response
Safety Rejection
Malformed Output

Cada uno debe tener una estrategia específica.

75. Integration Jobs

Los Jobs pueden encapsular integraciones:

SyncExternalCustomer
       ↓
Integration Service
       ↓
External API

El Worker no debe conocer detalles del proveedor.

76. Notification Jobs

Ejemplo:

SendNotification
       ↓
Notification Service
       ↓
Email / Push / SMS

Esto permite retry sin bloquear la operación principal.

77. Analytics Jobs

Ejemplo:

RebuildAnalytics
       ↓
Analytics Worker
       ↓
Analytics Store

Estos Jobs pueden tener prioridad baja.

78. Maintenance Jobs

Ejemplos:

CleanupExpiredData
RebuildIndexes
ArchiveRecords
ValidateIntegrity
79. Job and Multi-Tenancy

Cada Job tenant-scoped debe transportar:

tenant_id

durante todo el ciclo.

80. Tenant Isolation

Nunca permitir:

Tenant A Job
      ↓
Tenant B Data

Las referencias deben verificarse antes de ejecutar operaciones.

81. Tenant Quotas

Puede definirse:

Max Jobs
Max Concurrent Jobs
Max Runtime
Max Storage
Max Priority Jobs

por tenant.

82. Resource Governance

Los workers deben controlar:

CPU
Memory
Network
Database Connections
External API Rate
AI Tokens
83. Worker Isolation

Jobs costosos pueden ejecutarse en pools separados:

General Workers
AI Workers
Integration Workers
Analytics Workers
Maintenance Workers

Esto evita que un tipo de carga degrade todo EVOXA.

84. Queue Isolation

De forma equivalente:

critical.queue
default.queue
analytics.queue
ai.queue
maintenance.queue

La partición concreta depende de la infraestructura.

85. Backpressure

Cuando la capacidad disminuye:

Incoming Jobs
██████████████████
        ↓
Queue
████████████████
        ↓
Workers
████

el sistema debe:

Queue
Throttle
Prioritize
Scale
Reject

según política.

86. Queue Lag

Debe medirse:

created_at
      ↓
started_at

La diferencia representa:

Queue Lag
87. Processing Metrics

Métricas principales:

jobs_created_total
jobs_queued_total
jobs_started_total
jobs_completed_total
jobs_failed_total
jobs_retried_total
jobs_cancelled_total
jobs_dead_lettered_total
job_duration
queue_lag
88. Worker Metrics
worker_active
worker_idle
worker_capacity
worker_failures
worker_utilization
89. Job Tracing

Cada ejecución debe poder rastrearse:

trace_id
job_id
job_type
tenant_id
worker_id
90. Structured Logging

Ejemplo:

job_id=job-123
job_type=GenerateReport
worker_id=worker-7
tenant_id=tenant-01
status=completed
duration=842ms
91. Job Alerts

Alertas recomendadas:

Queue Lag High
Failure Rate High
DLQ Growth
Worker Saturation
Retry Storm
Long Running Jobs
Tenant Quota Exhaustion
92. Retry Storm Protection

Si un sistema externo falla:

Job
 ↓
Retry
 ↓
Retry
 ↓
Retry
 ↓
Retry

puede generarse una tormenta.

Debe existir:

Backoff
Jitter
Circuit Breaker
Rate Limit
Retry Budget

cuando sea necesario.

93. Circuit Breaker

Para integraciones externas:

Worker
  ↓
External API
  ↓
Repeated Failure
  ↓
Circuit Open

Los Jobs pueden esperar o fallar temporalmente en lugar de continuar atacando al proveedor.

94. Worker Health

Un Worker debe exponer:

Alive
Ready
Capacity
Current Jobs
Failure State
95. Graceful Shutdown

Antes de detener un Worker:

Stop accepting new jobs
        ↓
Finish / release current jobs
        ↓
Persist state
        ↓
Shutdown
96. Worker Autoscaling

El número de workers puede depender de:

Queue Depth
Queue Lag
CPU
Memory
Job Duration
Priority
97. Job Registry

Conceptualmente:

Job Registry
├── job_type
├── version
├── handler
├── timeout
├── retry_policy
├── priority
├── tenant_scope
└── status
98. Job Definition Lifecycle
Draft
 ↓
Validated
 ↓
Active
 ↓
Deprecated
 ↓
Retired
99. Job Versioning

Un Job debe asociarse a una versión:

GenerateReport.v1
GenerateReport.v2

Los Jobs ya creados deben conservar su versión.

100. Job Compatibility

Cuando cambia el payload:

v1 → v2

debe existir una estrategia:

Backward Compatibility
Migration
Translation
Dual Support
101. Job Contract

Cada Job debe definir:

Input Schema
Output Schema
Timeout
Retry Policy
Failure Policy
Authorization
Tenant Scope
Resource Requirements
102. Job Ownership

Cada Job debe tener:

Owner
Technical Owner
Business Owner
Criticality
SLA
103. SLA

Jobs críticos pueden definir:

Start SLA
Completion SLA
Recovery SLA

Ejemplo:

PaymentRecovery
→ completion target < 5 min
104. Job Dependency Management

Las dependencias externas deben registrarse:

Database
Integration
Queue
Service
External Provider
105. Failure Matrix
Failure	Strategy
Worker crash	Redelivery
Timeout	Retry / Fail
Network error	Retry
External 503	Retry
Rate limit	Backoff
Invalid payload	Fail
Business rejection	Fail
Retry exhaustion	DLQ
Cancellation	Stop safely
Resource exhaustion	Backpressure
106. Relationship with E13
E13 Event Processing
        │
        ▼
Event Handler
        │
        ▼
Create Job
        │
        ▼
E15 Job Processing

E13 procesa el evento.

E15 ejecuta el trabajo derivado.

107. Relationship with E14
E14 Workflow
      │
      ▼
Task
      │
      ▼
E15 Job
      │
      ▼
Worker

E14 coordina.

E15 ejecuta.

108. Relationship with E16

La separación prevista es:

E16 Scheduler
      │
      │ When?
      ▼
E15 Job Processing
      │
      │ How?
      ▼
Worker

Por tanto:

Scheduling determina cuándo debe ejecutarse un Job; Job Processing determina cómo se ejecuta.

109. Relationship with E12
E12 Messaging
      │
      ▼
Transport
      │
      ▼
E15 Job Queue
      │
      ▼
Worker

El mecanismo concreto de transporte puede variar sin cambiar la semántica del Job.

110. Relationship with E10
Worker
  ↓
Application Service
  ↓
Repository
  ↓
Database

E15 no sustituye la arquitectura de persistencia de E10.

111. Relationship with E11
Worker
  ↓
Application Service
  ↓
Integration Interface
  ↓
Adapter
  ↓
External System

Los Workers no deben acoplarse directamente a proveedores externos.

112. Canonical Architecture
                         EVOXA
                           │
        ┌──────────────────┼──────────────────┐
        │                  │                  │
        ▼                  ▼                  ▼
      Event             Workflow           Scheduler
      E13                 E14                E16
        │                  │                  │
        └──────────────────┼──────────────────┘
                           ▼
                      Job Creation
                           │
                           ▼
                      Job Registry
                           │
                           ▼
                       Job Queue
                           │
             ┌─────────────┼─────────────┐
             ▼             ▼             ▼
          Worker A      Worker B      Worker C
             │             │             │
             └─────────────┼─────────────┘
                           ▼
                       Job Handler
                           │
                           ▼
                  Application Service
                           │
              ┌────────────┼────────────┐
              ▼            ▼            ▼
           Domain      Repository   Integration
              │            │            │
              ▼            ▼            ▼
          Business      Database    External
113. Architectural Rules
Rule 1

Jobs execute work; Workflows coordinate business processes.

Rule 2

Workers do not own business rules.

Rule 3

Job handlers delegate business operations to Application Services.

Rule 4

Every retryable Job must define idempotency behavior.

Rule 5

External calls must pass through Integration Architecture.

Rule 6

Long-running business coordination belongs to E14.

Rule 7

Scheduling belongs to E16.

Rule 8

Queue infrastructure must remain replaceable.

Rule 9

Tenant context must travel with tenant-scoped Jobs.

Rule 10

Operational failures must never silently disappear.

114. Definition of Done

E15 queda definido cuando EVOXA dispone de:

✓ Job Definition
✓ Job Instance
✓ Job Identity
✓ Job Context
✓ Job Payload
✓ Job Queue
✓ Worker Architecture
✓ Worker Registration
✓ Job Handlers
✓ Application Service Integration
✓ Job Creation
✓ Event-triggered Jobs
✓ Workflow-triggered Jobs
✓ Scheduled Jobs
✓ Job Lifecycle
✓ Task Lifecycle
✓ Priority
✓ Concurrency
✓ Tenant Concurrency
✓ Fairness
✓ Idempotency
✓ Idempotency Keys
✓ Job Claiming
✓ Visibility Timeout
✓ Lease
✓ Heartbeat
✓ Long-running Jobs
✓ Progress Tracking
✓ Retry
✓ Retry Policies
✓ Backoff
✓ Retry Classification
✓ Dead Letter Queue
✓ Cancellation
✓ Cooperative Cancellation
✓ Timeouts
✓ Task Processing
✓ Batch Jobs
✓ Partial Batch Success
✓ Item-level Retry
✓ Chunking
✓ Fan-out
✓ Fan-in
✓ Job Results
✓ Job Artifacts
✓ Persistence
✓ Job History
✓ Retention
✓ Replay
✓ Safe Replay
✓ Dry Run
✓ Security
✓ Secret Protection
✓ Authorization
✓ Administrative Jobs
✓ AI Jobs
✓ Integration Jobs
✓ Notification Jobs
✓ Analytics Jobs
✓ Maintenance Jobs
✓ Tenant Isolation
✓ Tenant Quotas
✓ Resource Governance
✓ Worker Isolation
✓ Queue Isolation
✓ Backpressure
✓ Queue Lag
✓ Processing Metrics
✓ Worker Metrics
✓ Distributed Tracing
✓ Structured Logging
✓ Alerts
✓ Retry Storm Protection
✓ Circuit Breaker Compatibility
✓ Worker Health
✓ Graceful Shutdown
✓ Autoscaling
✓ Job Registry
✓ Definition Lifecycle
✓ Versioning
✓ Compatibility
✓ Job Contracts
✓ Ownership
✓ SLA
✓ Dependency Management
✓ Failure Matrix
✓ E13 Integration
✓ E14 Integration
✓ E16 Integration
✓ E10 Integration
✓ E11 Integration
✓ E12 Integration
115. Engineering Specification Progress

La secuencia queda:

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
E14 — Workflow & Orchestration Architecture
E15 — Job & Task Processing Architecture

La cadena de ejecución queda ahora claramente separada:

                       EVOXA
                         │
        ┌────────────────┼────────────────┐
        ▼                ▼                ▼
      Event           Workflow         Scheduler
       E13               E14              E16
        │                │                │
        └────────────────┼────────────────┘
                         ▼
                    Job Creation
                         │
                         ▼
                    Job Queue
                         │
                         ▼
                      Worker
                         │
                         ▼
                    Job Handler
                         │
                         ▼
                Application Service
                         │
              ┌──────────┴──────────┐
              ▼                     ▼
           Domain              Integration
              │                     │
              ▼                     ▼
         Repository           External World
              │
              ▼
           Database

E15 establece la capa de ejecución asíncrona de EVOXA: transforma eventos, workflows y schedules en unidades de trabajo ejecutables, distribuye esas unidades entre workers, controla concurrencia, retries, timeouts, prioridades, aislamiento por tenant, recuperación y observabilidad, sin absorber las responsabilidades de Event Processing, Workflow Orchestration o Scheduling.

Siguiente capítulo: E16 — EVOXA Scheduling Architecture.
