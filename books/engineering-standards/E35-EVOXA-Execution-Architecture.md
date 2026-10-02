E35 — EVOXA Execution Architecture
Engineering Specification

Pertenece a: Engineering Specification
Capítulo: E35 — Execution Architecture
Anterior: E34 — Action Architecture
Siguiente: E36 — EVOXA Resource Architecture

1. Propósito

E35 define la arquitectura responsable de ejecutar físicamente una Action.

La separación fundamental de EVOXA queda:

Intelligence
     ↓
Decision
     ↓
Action
     ↓
Execution
     ↓
Outcome

Donde:

Decision determina qué debería ocurrir.
Action expresa la intención operacional.
Execution realiza la operación.
Outcome registra lo que realmente ocurrió.

La regla central:

Execution is the runtime realization of an authorized Action.

2. Execution Boundary

Execution comienza cuando existe una Action válida y autorizada:

Action
  ↓
Execution Request
  ↓
Execution

Y termina cuando existe un resultado:

Execution
  ↓
Execution Result
  ↓
Outcome

Execution no debe convertirse en un segundo Decision Engine.

3. Arquitectura General
                         ACTION
                           │
                           ▼
                  Execution Dispatcher
                           │
                           ▼
                  Execution Resolver
                           │
              ┌────────────┼────────────┐
              ▼            ▼            ▼
          Sync Executor  Async Worker  Workflow
              │            │            │
              └────────────┼────────────┘
                           ▼
                    Execution Runtime
                           │
              ┌────────────┼────────────┐
              ▼            ▼            ▼
          Internal      External      Human
           Target        Target       Target
              │            │            │
              └────────────┼────────────┘
                           ▼
                    Execution Result
                           │
              ┌────────────┼────────────┐
              ▼            ▼            ▼
           Outcome       Event        Audit
4. Execution Definition

Una Execution representa:

la realización concreta de una Action

Modelo conceptual:

Execution
├── executionId
├── actionId
├── decisionId
├── executor
├── target
├── input
├── context
├── attempt
├── status
├── startedAt
├── completedAt
├── result
├── error
└── metadata
5. Execution Identity

Toda ejecución debe poseer un identificador único:

executionId

Debe poder relacionarse con:

actionId
decisionId
workflowId
jobId
correlationId
causationId
6. Execution Lifecycle

Ciclo principal:

REQUESTED
    ↓
ACCEPTED
    ↓
DISPATCHED
    ↓
STARTED
    ↓
RUNNING
    ↓
COMPLETED

Estados alternativos:

REJECTED
CANCELLED
TIMED_OUT
FAILED
RETRYING
ABORTED
COMPENSATING
COMPENSATED
7. Execution State Machine
                 ┌──────────────┐
                 │   REQUESTED  │
                 └──────┬───────┘
                        ▼
                 ┌──────────────┐
                 │   ACCEPTED   │
                 └──────┬───────┘
                        ▼
                 ┌──────────────┐
                 │  DISPATCHED  │
                 └──────┬───────┘
                        ▼
                 ┌──────────────┐
                 │    STARTED   │
                 └──────┬───────┘
                        ▼
                 ┌──────────────┐
                 │    RUNNING   │
                 └──────┬───────┘
                        │
              ┌─────────┼─────────┐
              ▼         ▼         ▼
         COMPLETED    FAILED    TIMEOUT
                          │
                   ┌──────┴──────┐
                   ▼             ▼
                RETRY       TERMINAL FAILURE
8. Execution Request

El sistema debe crear una representación explícita:

ExecutionRequest
├── executionId
├── actionId
├── actionType
├── actionVersion
├── actor
├── target
├── input
├── context
├── priority
├── deadline
└── idempotencyKey
9. Execution Dispatcher

El Dispatcher recibe una Execution Request y determina:

where
how
when
by whom

debe ejecutarse.

No debe modificar la intención de la Action.

10. Execution Resolver

El Resolver determina el mecanismo concreto:

Action Type
      ↓
Capability
      ↓
Executor
      ↓
Target

Ejemplo:

TRANSFER_FUNDS
      ↓
Payment Capability
      ↓
Payment Executor
      ↓
Payment Provider
11. Executor

Un Executor es el componente que realiza físicamente la operación.

Ejemplos:

NotificationExecutor
PaymentExecutor
CustomerExecutor
DocumentExecutor
StorageExecutor
IntegrationExecutor
HumanTaskExecutor
12. Executor Contract

Cada Executor debe declarar:

ExecutorContract
├── supportedActions
├── supportedVersions
├── capabilities
├── inputSchema
├── outputSchema
├── limits
├── timeout
└── failureModel
13. Executor Independence

La Action debe permanecer independiente del Executor concreto.

Action
   ↓
Execution Contract
   ↓
Executor A

puede cambiarse por:

Executor B

sin modificar necesariamente la Action.

14. Execution Modes

EVOXA debe soportar al menos:

SYNCHRONOUS
ASYNCHRONOUS
SCHEDULED
BATCH
STREAMING
HUMAN
REMOTE
15. Synchronous Execution
Caller
  ↓
Action
  ↓
Executor
  ↓
Result

Adecuado para operaciones:

rápidas
deterministas
con timeout corto
16. Asynchronous Execution
Action
  ↓
Execution Request
  ↓
Queue
  ↓
Worker
  ↓
Executor
  ↓
Result

El caller no necesita esperar el resultado.

17. Scheduled Execution
Action
  ↓
Scheduler
  ↓
Execution Request
  ↓
Executor

El Scheduler determina cuándo.

Execution determina cómo ejecutar.

18. Batch Execution

Puede agrupar múltiples acciones:

Batch
├── Execution A
├── Execution B
├── Execution C
└── Execution D

Debe conservarse la identidad individual de cada ejecución.

19. Streaming Execution

Para operaciones de larga duración:

Execution
   ↓
Progress
   ↓
Progress
   ↓
Progress
   ↓
Completed

Los eventos parciales no deben confundirse con completion.

20. Human Execution

Algunas Actions requieren:

Execution
   ↓
Human Task
   ↓
Human Decision/Input
   ↓
Completion

Debe tratarse como una forma legítima de Execution.

21. Remote Execution

Una Execution puede delegarse a otro runtime:

EVOXA Runtime
      ↓
Execution Gateway
      ↓
Remote Runtime
      ↓
Remote Executor
22. Internal Execution

Cuando target pertenece al propio sistema:

Action
 ↓
Application Service
 ↓
Domain Service
 ↓
State Change
23. External Execution

Cuando target es externo:

Action
 ↓
Executor
 ↓
Adapter
 ↓
Gateway
 ↓
External System
24. Execution Adapter

El Adapter traduce:

EVOXA Execution Contract

a:

External API Contract

y viceversa.

25. Execution Gateway

El Gateway puede aplicar:

authentication
authorization
rate limiting
timeouts
routing
circuit breaking
observability
26. Execution Context

Toda ejecución necesita un contexto controlado:

ExecutionContext
├── tenant
├── actor
├── authority
├── correlation
├── locale
├── security context
├── deadline
└── runtime metadata
27. Context Propagation

Debe propagarse:

tenantId
correlationId
causationId
actorId
traceId
executionId

sin perder su integridad.

28. Security Context

Execution debe respetar:

identity
permissions
roles
scopes
tenant isolation
policy decisions

La existencia de una Action autorizada no elimina las verificaciones necesarias en el punto de ejecución.

29. Execution Isolation

Las ejecuciones deben aislarse según:

tenant
security domain
resource
executor
risk class

cuando sea necesario.

30. Resource Boundary

Una Execution consume recursos:

CPU
memory
network
database connections
external API quota
storage
workers

Por tanto Execution debe integrarse con Resource Architecture.

31. Execution Allocation

Antes de comenzar una ejecución puede ser necesario reservar:

worker
capacity
connection
memory
quota
32. Concurrency

El sistema debe controlar:

max concurrent executions

por:

tenant
executor
actionType
target
resource
33. Execution Queue

Las ejecuciones asíncronas pueden pasar por:

Execution Queue

con:

priority
visibility timeout
retry count
deadline
tenant
34. Worker

El Worker:

receive
claim
execute
report

una Execution.

Debe evitar que múltiples workers ejecuten accidentalmente la misma ejecución.

35. Work Claiming

Un worker debe adquirir ownership temporal:

Execution
 ↓
CLAIMED
 ↓
Worker A

El sistema debe manejar la pérdida de ese ownership.

36. Visibility Timeout

Para ejecuciones asíncronas puede utilizarse:

visibilityTimeout

Si el worker desaparece:

Execution
 ↓
visible again
 ↓
another worker
37. Execution Lease

Una alternativa es:

Execution Lease
├── owner
├── acquiredAt
├── expiresAt
└── renewal
38. Execution Heartbeat

Para tareas largas:

STARTED
 ↓
heartbeat
 ↓
heartbeat
 ↓
heartbeat
 ↓
COMPLETED

permite distinguir una tarea viva de una tarea perdida.

39. Execution Timeout

Debe existir un límite:

executionTimeout

Al superarlo:

RUNNING
 ↓
TIMED_OUT
40. Cancellation

Una ejecución puede recibir:

cancel request

pero la capacidad real de cancelación depende del executor.

41. Cooperative Cancellation

Para operaciones largas:

Executor
 ↓
Cancellation Token

permite detener la ejecución de forma segura.

42. Hard Cancellation

Algunas ejecuciones requieren terminar forzosamente:

process termination
worker termination
connection termination

Debe utilizarse solo cuando sea seguro.

43. Execution Result

Modelo:

ExecutionResult
├── executionId
├── status
├── output
├── metadata
├── startedAt
├── completedAt
├── duration
└── error
44. Result Classification

Un resultado puede ser:

SUCCESS
PARTIAL_SUCCESS
FAILURE
TIMEOUT
CANCELLED
UNKNOWN
45. Execution Error

Modelo:

ExecutionError
├── code
├── category
├── message
├── retryable
├── recoverable
├── externalCode
└── details

Los detalles sensibles deben protegerse.

46. Failure Categories
VALIDATION
AUTHORIZATION
POLICY
RESOURCE
DEPENDENCY
NETWORK
TIMEOUT
RATE_LIMIT
CONCURRENCY
BUSINESS
SYSTEM
UNKNOWN
47. Retry

Execution debe implementar retries solamente cuando sea seguro.

Failure
 ↓
Retry Policy
 ↓
Retry
48. Retry Attempt

Cada intento debe identificarse:

executionId
attemptNumber

Ejemplo:

Execution E100
Attempt 1
Attempt 2
Attempt 3
49. Retry Backoff

Puede utilizar:

fixed
linear
exponential
exponential + jitter

según el target.

50. Retry Budget

Debe limitarse:

maxAttempts
maxElapsedTime
maxRetryCost
51. Idempotent Execution

Execution debe soportar idempotencia cuando el target pueda producir side effects.

Action
 ↓
Execution
 ↓
Idempotency Key
 ↓
Target
52. Duplicate Execution

Si llega dos veces:

ExecutionRequest(E100)
ExecutionRequest(E100)

el sistema debe detectar la duplicación y evitar efectos duplicados cuando el contrato lo requiera.

53. Exactly Once

No debe asumirse exactamente-once a nivel distribuido.

La arquitectura debe preferir:

at-least-once delivery
+
idempotent side effects

cuando sea apropiado.

54. Execution Transaction

Cuando existe un único boundary transaccional:

Execution
 ↓
Transaction
 ↓
Commit
55. Distributed Execution

Para múltiples sistemas:

Execution
 ↓
Service A
 ↓
Service B
 ↓
External System

debe utilizarse una estrategia explícita de consistencia.

56. Saga Execution

Cuando corresponda:

Execution A
   ↓
Execution B
   ↓
Execution C

con compensaciones asociadas.

La coordinación compleja pertenece a Workflow/Orchestration.

57. Execution Compensation

Cuando una ejecución produjo un side effect irreversible o parcialmente reversible:

Execution
 ↓
Compensation Execution
58. Execution Journal

Debe existir historial suficiente:

ExecutionJournal
├── requested
├── accepted
├── dispatched
├── started
├── retries
├── completed
└── failed

Esto permite reconstruir el lifecycle.

59. Execution Event Model

Eventos:

ExecutionRequested
ExecutionAccepted
ExecutionDispatched
ExecutionStarted
ExecutionProgressed
ExecutionRetried
ExecutionCompleted
ExecutionFailed
ExecutionTimedOut
ExecutionCancelled
ExecutionCompensationStarted
ExecutionCompensated
60. Event Ordering

Los eventos deben conservar:

sequence
timestamp
causation
correlation

para permitir reconstrucción del estado.

61. Execution Outcome

Execution produce:

Execution Result

que posteriormente alimenta:

Outcome

La diferencia:

Execution Result
→ technical execution status

Outcome
→ business/system effect
62. Execution → Outcome
Execution
   ↓
Result
   ↓
Outcome Processor
   ↓
Outcome
63. Execution Observability

Debe proporcionar:

logs
metrics
traces
events
audit
64. Execution Metrics

Métricas esenciales:

executions_total
executions_success_total
executions_failed_total
execution_latency
execution_duration
execution_timeout_total
execution_retry_total
execution_cancel_total
execution_queue_depth
execution_concurrency
65. Executor Metrics

Por executor:

success rate
failure rate
latency
throughput
saturation
timeouts
retry rate
66. Target Metrics

Por target:

availability
latency
error rate
rate-limit events
connection failures
67. Distributed Tracing

La traza debe cubrir:

Decision
 ↓
Action
 ↓
Execution
 ↓
Executor
 ↓
Adapter
 ↓
External Target
68. Correlation

Cada Execution debe mantener:

correlationId

durante todo el lifecycle.

69. Causation

Debe conservar:

causationId

para conocer qué evento o acción originó una ejecución.

70. Audit

Las ejecuciones sensibles deben registrar:

who
what
when
where
why
with which authority
result
71. Secret Protection

Nunca deben almacenarse directamente en logs:

passwords
tokens
API keys
credentials
private keys
72. Execution Security

Controles mínimos:

least privilege
identity verification
authorization
tenant isolation
credential isolation
input validation
output validation
audit
73. Credential Resolution

El Executor puede obtener credenciales mediante:

Secret Manager
Credential Provider
Workload Identity
Managed Identity

sin que la Action las contenga directamente.

74. External API Execution

Patrón:

Action
 ↓
Execution
 ↓
Executor
 ↓
Adapter
 ↓
Gateway
 ↓
External API
75. External Dependency Failure

Debe distinguirse:

target unavailable
target rejected request
network failure
timeout
rate limit
authentication failure

porque cada uno puede requerir una estrategia distinta.

76. Circuit Breaker

Cuando un target falla repetidamente:

CLOSED
 ↓
OPEN
 ↓
HALF_OPEN

Execution debe evitar continuar enviando tráfico al target degradado.

77. Bulkhead

Execution puede aislar capacidad:

Payment Executor
Notification Executor
Search Executor
Integration Executor

para evitar fallos en cascada.

78. Rate Limiting

Puede aplicarse:

per tenant
per executor
per action
per target
per credential
79. Backpressure

Cuando los workers están saturados:

Requests
 ↓
Queue
 ↓
Controlled Consumption

en lugar de aceptar trabajo ilimitadamente.

80. Priority Execution

Las ejecuciones pueden clasificarse:

CRITICAL
HIGH
NORMAL
LOW

pero prioridad no debe saltarse:

security
authorization
policy
resource limits
81. Deadline Propagation

Si Action contiene deadline:

Action Deadline
      ↓
Execution Deadline
      ↓
Network Timeout
      ↓
External Request Timeout

El deadline debe propagarse.

82. Execution Budget

Una ejecución puede consumir:

time budget
CPU budget
memory budget
network budget
API quota
financial budget
83. Resource Exhaustion

Si no existen recursos suficientes:

Execution
 ↓
RESOURCE_UNAVAILABLE

Debe evitarse iniciar una operación que probablemente no pueda completarse.

84. Execution Admission Control

Antes de aceptar:

Execution Request

puede evaluarse:

capacity
quota
priority
deadline
risk
85. Admission Control

Resultado:

ACCEPT
DEFER
REJECT
86. Execution Queue Fairness

La cola debe evitar starvation.

Puede implementar:

weighted fairness
tenant quotas
priority aging

según necesidades.

87. Multi-Tenant Execution

Cada Execution debe estar asociada a:

tenantId

y la infraestructura debe impedir acceso cruzado.

88. Tenant Quotas

Puede existir:

max concurrent executions
max requests/sec
max queue depth
max compute

por tenant.

89. Cross-Tenant Execution

Debe requerir:

explicit authority
explicit policy
explicit audit
90. Execution Scheduling Boundary

La separación:

Scheduler
→ determines when

Execution
→ determines how

debe mantenerse.

91. Execution and Jobs

Un Job representa:

durable unit of work

Una Execution representa:

one concrete attempt/realization

Por tanto:

Job
 ├── Execution 1
 ├── Execution 2
 └── Execution 3

puede existir cuando hay retries o re-ejecuciones.

92. Execution and Workflow

Workflow coordina:

Execution A
 ↓
Execution B
 ↓
Execution C

Execution realiza cada operación individual.

93. Execution and Event Processing

Un evento puede disparar una Execution:

Event
 ↓
Action
 ↓
Execution
94. Execution and Messaging

Una Execution asíncrona puede viajar mediante:

Command Message

hasta un Worker.

95. Execution and API

Una API puede:

POST Action

y obtener:

202 Accepted
executionId

cuando la ejecución sea asíncrona.

96. Execution API

Conceptualmente:

POST /executions
GET /executions/{id}
POST /executions/{id}/cancel
POST /executions/{id}/retry
GET /executions/{id}/result
GET /executions/{id}/history

Los contratos concretos pertenecen a E03.

97. Execution Repository

Debe persistir:

executionId
actionId
status
attempt
executor
target
timestamps
result
error
history
98. Execution Persistence

No todas las ejecuciones necesitan la misma retención.

Puede distinguirse:

short-lived runtime state
durable execution history
audit records
business outcome
99. Execution State Store

Para ejecuciones activas:

Execution State Store

debe permitir:

fast lookup
concurrency control
lease management
heartbeat
state transition
100. Execution History Store

Para histórico:

Execution History

puede almacenar:

events
attempts
results
failures
timings
101. Execution State Transition

Las transiciones deben ser válidas:

REQUESTED → ACCEPTED
ACCEPTED → DISPATCHED
DISPATCHED → STARTED
STARTED → RUNNING
RUNNING → COMPLETED
RUNNING → FAILED
RUNNING → TIMED_OUT

No debería ser posible:

COMPLETED → RUNNING

sin un mecanismo explícito de replay/re-execution.

102. Optimistic Concurrency

Las transiciones pueden protegerse mediante:

version
etag
compare-and-swap

para evitar que dos workers actualicen simultáneamente el mismo estado.

103. Execution Ownership

Debe existir un concepto de:

currentExecutor

o:

executionLease

cuando la ejecución es distribuida.

104. Worker Failure

Si un worker muere:

Worker A
   ↓
Execution RUNNING
   X
Worker failure
   ↓
Lease expires
   ↓
Worker B

La arquitectura debe decidir si:

resume
retry
reconcile
manual intervention
105. Side-Effect Ambiguity

El caso peligroso:

External system accepted request
       ↓
Network timeout
       ↓
EVOXA does not know result

No debe asumirse automáticamente que falló.

Debe existir:

reconciliation
status query
idempotency
manual review

según el target.

106. Reconciliation

Una Execution con estado desconocido puede pasar a:

UNKNOWN
 ↓
RECONCILIATION
 ↓
CONFIRMED_SUCCESS

o:

CONFIRMED_FAILURE
107. Unknown State

UNKNOWN no debe traducirse automáticamente a FAILED.

Especialmente para side effects externos.

108. Execution Recovery

Recovery puede utilizar:

retry
resume
reconcile
compensate
manual review
109. Execution Replay

Replay debe crear una nueva realización cuando sea necesario:

Original Execution E1
        ↓
Replay
        ↓
Execution E2

No debe destruir el histórico de E1.

110. Execution Lineage

Debe poder observarse:

Action A
 ├── Execution E1
 │    └── Attempt 1
 ├── Execution E2
 │    └── Replay
 └── Execution E3
      └── Compensation
111. Execution Versioning

Debe conservarse:

actionVersion
executorVersion
adapterVersion
policyVersion

cuando sea relevante para reproducibilidad.

112. Deterministic Execution

Cuando sea posible:

same input
+
same context
+
same version
=
same result

Para operaciones no deterministas debe registrarse suficiente contexto para explicar el resultado.

113. Execution Reproducibility

Debe poder reconstruirse:

what was executed
with which input
under which policy
using which executor

sin necesariamente repetir el side effect.

114. Dry Run

Execution puede soportar:

DRY_RUN

donde:

validate
resolve
simulate

pero no se ejecutan side effects reales.

115. Simulation

Puede utilizarse un target simulado:

Production Action
       ↓
Simulation Executor

para pruebas y validación.

116. Shadow Execution

Un nuevo Executor puede ejecutarse en paralelo:

Production Executor
Candidate Executor

pero solo el resultado autorizado produce side effects.

117. Human Approval Boundary

Si Execution requiere aprobación:

Execution
 ↓
WAITING_APPROVAL
 ↓
APPROVED
 ↓
RUNNING

o:

REJECTED
118. Execution Approval

La aprobación debe registrar:

approver
authority
timestamp
execution snapshot
decision
119. Execution Safety

Las ejecuciones de alto impacto deben poder incorporar:

approval
rate limits
resource limits
kill switch
audit
reconciliation
120. Kill Switch

Governance/Operations puede detener una clase de executions:

Action Type
 ↓
Execution Block

sin eliminar necesariamente las Actions históricas.

121. Emergency Stop

Para situaciones críticas:

Executor
 ↓
DISABLED

Las nuevas ejecuciones deben rechazarse o ponerse en espera según política.

122. Execution Governance

Governance controla:

allowed executors
allowed targets
risk levels
approval rules
execution limits
retention
audit requirements
123. Execution Policy

Execution Policy puede definir:

maxDuration
maxAttempts
allowedHours
allowedRegions
allowedExecutors
requiredApprovals
124. Policy Enforcement Point

El enforcement puede aparecer:

before dispatch
before execution
during execution
after execution

dependiendo del control.

125. Execution Preflight

Antes de iniciar:

Action valid?
Authority valid?
Policy valid?
Resources available?
Executor available?
Target available?
Deadline valid?

Si todo es válido:

START
126. Execution Postflight

Después:

result captured
state persisted
events published
audit recorded
outcome generated
127. Execution Completion

Una Execution solo debe marcarse COMPLETED cuando:

required side effect
+
required persistence
+
required result handling

hayan alcanzado el criterio de completion definido por su contrato.

128. Completion Semantics

Cada Executor debe definir qué significa:

COMPLETED

No debe depender de una interpretación genérica.

129. Partial Completion

Cuando solo una parte de la operación se completó:

PARTIAL_SUCCESS

debe preservar:

completed portions
failed portions
remaining work
compensation options
130. Execution Contract with Outcome
Action
   ↓
Execution
   ↓
ExecutionResult
   ↓
Outcome

Esta separación permite que el resultado técnico no sea confundido con el impacto de negocio.

131. Execution Architecture Rules

Reglas normativas:

R1 — Every execution has an identity.
R2 — Execution requires a valid Action.
R3 — Execution cannot silently expand Action authority.
R4 — Executors must be replaceable.
R5 — External side effects must consider idempotency.
R6 — Distributed execution must assume failure.
R7 — Unknown external outcomes must not be treated as failure automatically.
R8 — Retries must be policy-controlled.
R9 — Execution state must be observable.
R10 — Execution must preserve tenant isolation.
R11 — Execution must preserve provenance.
R12 — Execution must produce an explicit result.
R13 — Workflow coordinates; Executor executes.
R14 — Scheduler schedules; Executor executes.
R15 — Policy governs; Executor enforces.
R16 — Execution must not silently become Decision.
132. Execution Failure Matrix
Failure	Default Response
Validation	Reject
Authorization	Reject
Policy	Reject / Review
Resource unavailable	Defer / Retry
Network transient	Retry
Rate limit	Backoff
Timeout	Reconcile / Retry
External unknown	Reconcile
Permanent business error	Fail
Worker crash	Recover / Retry
Duplicate request	Idempotent handling
Executor disabled	Reject / Defer
133. Execution Reference Architecture
                       ┌──────────────────┐
                       │      ACTION      │
                       └────────┬─────────┘
                                │
                                ▼
                       ┌──────────────────┐
                       │    PREFLIGHT     │
                       └────────┬─────────┘
                                │
                                ▼
                       ┌──────────────────┐
                       │   DISPATCHER     │
                       └────────┬─────────┘
                                │
                    ┌───────────┼───────────┐
                    ▼           ▼           ▼
                Sync        Queue       Scheduler
                 │            │             │
                 │            ▼             │
                 │         Worker           │
                 │            │             │
                 └────────────┼─────────────┘
                              ▼
                       ┌───────────────┐
                       │    EXECUTOR   │
                       └───────┬───────┘
                               │
                   ┌───────────┼───────────┐
                   ▼           ▼           ▼
               Internal     Adapter      Human
                   │           │           │
                   ▼           ▼           ▼
               Services     External      Task
                            Systems
                   │           │           │
                   └───────────┼───────────┘
                               ▼
                       ┌───────────────┐
                       │ EXECUTION     │
                       │ RESULT        │
                       └───────┬───────┘
                               │
             ┌─────────────────┼────────────────┐
             ▼                 ▼                ▼
          Outcome            Event            Audit
134. Relationship with Previous Architecture

La secuencia E32–E35 queda cerrada:

E32 Intelligence
      │
      │ understands
      ▼
E33 Decision
      │
      │ chooses
      ▼
E34 Action
      │
      │ expresses authorized intent
      ▼
E35 Execution
      │
      │ performs
      ▼
Outcome

Y alrededor:

Policy
  ├── governs Decision
  ├── governs Action
  └── governs Execution

Workflow
  └── coordinates Executions

Scheduler
  └── determines timing

Messaging
  └── transports execution requests

Runtime
  └── provides execution environment

Observability
  └── observes execution

Governance
  └── controls execution boundaries
135. Definition of Done

E35 queda arquitectónicamente definido cuando EVOXA dispone de:

✓ Execution boundary
✓ Execution identity
✓ Execution request
✓ Execution lifecycle
✓ Execution state machine
✓ Dispatcher
✓ Resolver
✓ Executor contract
✓ Executor abstraction
✓ Synchronous execution
✓ Asynchronous execution
✓ Scheduled execution
✓ Batch execution
✓ Streaming execution
✓ Human execution
✓ Remote execution
✓ Internal execution
✓ External execution
✓ Adapter boundary
✓ Gateway boundary
✓ Execution context
✓ Context propagation
✓ Security context
✓ Isolation
✓ Resource allocation
✓ Concurrency
✓ Queue
✓ Worker
✓ Work claiming
✓ Lease
✓ Heartbeat
✓ Timeout
✓ Cancellation
✓ Retry
✓ Retry attempts
✓ Backoff
✓ Retry budget
✓ Idempotency
✓ Duplicate handling
✓ Transaction boundary
✓ Distributed execution
✓ Saga integration
✓ Compensation
✓ Execution journal
✓ Execution events
✓ Result model
✓ Error model
✓ Failure classification
✓ Outcome boundary
✓ Observability
✓ Metrics
✓ Tracing
✓ Audit
✓ Secret protection
✓ Circuit breaker
✓ Bulkhead
✓ Rate limiting
✓ Backpressure
✓ Priority
✓ Deadline
✓ Execution budget
✓ Admission control
✓ Multi-tenant isolation
✓ Job boundary
✓ Workflow boundary
✓ Scheduler boundary
✓ Messaging boundary
✓ API boundary
✓ Repository
✓ State store
✓ History store
✓ Concurrency control
✓ Worker recovery
✓ Unknown outcome handling
✓ Reconciliation
✓ Replay
✓ Lineage
✓ Versioning
✓ Reproducibility
✓ Dry run
✓ Simulation
✓ Shadow execution
✓ Human approval
✓ Kill switch
✓ Governance
✓ Policy enforcement
✓ Preflight
✓ Postflight
✓ Completion semantics
✓ Partial success
✓ Execution rules
Posición actual del Engineering Specification
E29 Search
  ↓
E30 Reporting
  ↓
E31 Analytics
  ↓
E32 Intelligence
  ↓
E33 Decision
  ↓
E34 Action
  ↓
E35 Execution
  ↓
E36 Resource

La frontera conceptual ya queda muy sólida:

E33 decide. E34 formaliza la intención. E35 ejecuta el efecto.

El siguiente capítulo natural es E36 — EVOXA Resource Architecture, porque una vez definido cómo se ejecuta una Action, hay que definir sobre qué recursos se ejecuta, cómo se asignan, limitan, reservan, comparten y liberan.
