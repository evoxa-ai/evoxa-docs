E37 — EVOXA Runtime Capacity Architecture
Engineering Specification

Pertenece a: Engineering Specification
Capítulo: E37 — Runtime Capacity Architecture
Anterior: E36 — Resource Architecture
Siguiente: E38 — EVOXA Resilience Architecture

1. Propósito

E37 define cómo EVOXA determina, administra y adapta la capacidad operativa del Runtime necesaria para ejecutar trabajo de forma segura, predecible y eficiente.

La idea central es:

Runtime Capacity is the controlled ability of EVOXA to accept, schedule, execute, absorb, and recover workload within defined resource and operational limits.

E36 define Resources.

E37 define cómo el Runtime convierte esos Resources en capacidad efectiva de ejecución.

Resource
   ↓
Capacity
   ↓
Concurrency
   ↓
Work Admission
   ↓
Execution
2. Capacity Boundary

E37 responde:

¿Cuánto trabajo puede aceptar EVOXA?
¿Cuánto trabajo puede ejecutar simultáneamente?
¿Cuándo debe dejar de aceptar trabajo?
¿Cuándo debe escalar?
¿Cuándo debe aplicar backpressure?
¿Cuándo debe degradar?
¿Cuándo debe rechazar?

No decide:

qué Action ejecutar
qué Decision tomar
qué Resource posee el sistema

Su responsabilidad es:

capacity
concurrency
admission
saturation
scaling
backpressure
load management
3. Capacity Model

La capacidad efectiva puede representarse como:

Runtime Capacity
├── Compute Capacity
├── Memory Capacity
├── Concurrency Capacity
├── Queue Capacity
├── Network Capacity
├── I/O Capacity
├── External Dependency Capacity
└── Policy Capacity

La capacidad efectiva de una operación está limitada por el cuello de botella más restrictivo.

Effective Capacity
=
min(
    compute,
    memory,
    concurrency,
    queue,
    network,
    dependency,
    policy
)
4. Capacity vs Resource

Distinción fundamental:

Resource
→ capacidad física/lógica disponible

Capacity
→ cantidad de trabajo que puede soportarse

Runtime Capacity
→ capacidad utilizable para ejecutar workload

Ejemplo:

Resource:
8 CPU

Runtime Capacity:
40 concurrent executions

No existe necesariamente una relación 1:1.

5. Capacity Dimensions

EVOXA debe poder medir capacidad en varias dimensiones:

CPU
MEMORY
CONCURRENCY
THROUGHPUT
LATENCY
QUEUE_DEPTH
NETWORK
I/O
CONNECTIONS
WORKERS
EXTERNAL_QUOTA
6. Capacity Unit

Toda capacidad debe tener una unidad explícita.

Ejemplos:

8 CPU cores
32 GB memory
100 concurrent executions
1,000 jobs/min
500 requests/sec
200 database connections
7. Runtime Capacity Envelope

Cada Runtime opera dentro de un envelope:

Capacity Envelope
├── Minimum Capacity
├── Target Capacity
├── Maximum Capacity
├── Burst Capacity
└── Emergency Capacity

Ejemplo:

minimum = 2 workers
target  = 8 workers
maximum = 20 workers
burst   = 30 workers
8. Minimum Capacity

Minimum Capacity define el nivel mínimo operativo.

Permite:

availability
warm state
baseline throughput

Debe evitar que el sistema reduzca capacidad por debajo de un límite funcional.

9. Target Capacity

Representa el nivel esperado para operar normalmente.

Target Capacity
≈
normal workload capacity

No es necesariamente un valor fijo.

Puede adaptarse dinámicamente.

10. Maximum Capacity

Límite superior:

Runtime
   ↓
Maximum Capacity
   ↓
No additional scaling

Cuando se alcanza:

queue
backpressure
throttle
reject

según política.

11. Burst Capacity

Permite absorber picos temporales:

Baseline
   ↓
Burst
   ↓
Normalize
   ↓
Scale Down

Burst no debe convertirse accidentalmente en capacidad permanente.

12. Capacity Headroom

Debe mantenerse margen operativo:

Headroom
=
Available Capacity
-
Expected Demand

Ejemplo:

capacity = 100
expected = 80
headroom = 20

El headroom protege contra variaciones repentinas.

13. Capacity Utilization
Utilization
=
Used Capacity / Total Capacity

Pero utilización no debe interpretarse sola.

Debe analizarse junto con:

latency
queue depth
error rate
saturation
14. Saturation

Saturation representa el grado en que el Runtime está cerca de no poder aceptar trabajo adicional.

Ejemplo:

CPU = 65%
Queue = 95%
Concurrency = 100%

Aunque CPU no esté saturada, el Runtime sí puede estar saturado.

15. Bottleneck

El Runtime debe identificar el recurso limitante:

CPU
Memory
Worker Slots
Database Connections
Network
External API
Queue

Ejemplo:

Workers available = 100
DB connections = 20

Effective concurrency <= 20
16. Capacity Admission

Toda carga debe pasar por un Admission Layer:

Incoming Work
      ↓
Capacity Check
      ↓
Admission Decision

Resultado:

ACCEPT
QUEUE
DEFER
THROTTLE
REJECT
17. Admission Controller

El Admission Controller evalúa:

current load
available capacity
priority
tenant quota
resource availability
execution cost
deadline
system health
18. Admission States
OPEN
LIMITED
THROTTLED
QUEUE_ONLY
REJECTING
DRAINING
19. Open State
Incoming Work
      ↓
Accepted

El Runtime opera dentro de capacidad normal.

20. Limited State

Acepta trabajo pero reduce la tasa:

Requests
   ↓
Rate Control
   ↓
Execution
21. Throttled State

Reduce deliberadamente throughput para proteger el sistema.

Demand
  ↓
Throttle
  ↓
Controlled Throughput
22. Queue-Only State

Cuando no existe capacidad inmediata:

Incoming Work
      ↓
Queue
      ↓
Wait
      ↓
Execution
23. Rejecting State

Si no puede absorber más trabajo:

Incoming Work
      ↓
Rejected

Debe producir una razón explícita.

CAPACITY_EXCEEDED
SYSTEM_SATURATED
QUOTA_EXCEEDED
DEPENDENCY_SATURATED
24. Draining State

Durante shutdown o maintenance:

DRAINING
   ↓
No New Work
   ↓
Existing Work Completes
   ↓
Runtime Empty
25. Workload Model

El Runtime debe modelar:

Arrival Rate
Service Rate
Concurrency
Queue Depth
Execution Duration

Conceptualmente:

Arrival Rate > Service Rate
        ↓
Queue grows
        ↓
Saturation
26. Throughput

Throughput representa:

completed work / unit of time

Ejemplos:

500 executions/min
20 workflows/sec
100 jobs/min
27. Arrival Rate
λ = incoming work / time

El Runtime debe comparar:

arrival rate
vs
service rate
28. Service Rate
μ = completed work / time

Si:

λ < μ

el sistema puede absorber carga.

Si:

λ > μ

la cola tenderá a crecer.

29. Concurrency

Concurrency representa trabajo simultáneamente activo:

Concurrency
=
active executions

Debe existir un límite:

maxConcurrency
30. Concurrency Pools

La concurrencia puede controlarse por:

global
tenant
domain
service
action
executor
resource
workflow
31. Global Concurrency
Runtime
   ↓
Global Concurrency Limit

Protege el Runtime completo.

32. Tenant Concurrency
Tenant A → 20
Tenant B → 10
Tenant C → 5

Evita monopolización.

33. Action Concurrency

Puede limitarse una Action específica:

Action: GenerateReport
maxConcurrency = 3

Esto puede ser necesario si la Action tiene una dependencia costosa.

34. Executor Concurrency

Cada Executor puede tener:

maxConcurrentExecutions

Ejemplo:

Executor A → 10
Executor B → 50
35. Resource-Bound Concurrency

La concurrencia efectiva puede estar limitada por Resource.

100 workers
10 database connections

Por tanto:

DB-bound concurrency = 10
36. Concurrency Admission

Antes de iniciar una Execution:

Concurrency Check
        ↓
slot available?
   ┌────┴────┐
  yes        no
   ↓          ↓
start       queue
37. Worker Capacity

Workers representan unidades de ejecución.

Worker Pool
├── Worker 1
├── Worker 2
├── Worker 3
└── Worker N

Cada worker puede tener:

capacity
state
load
health
current execution
38. Worker States
STARTING
READY
BUSY
DRAINING
UNHEALTHY
STOPPING
STOPPED
39. Worker Pool
Worker Pool
├── Minimum Workers
├── Target Workers
├── Maximum Workers
├── Available Workers
├── Busy Workers
└── Unhealthy Workers
40. Worker Allocation
Execution
   ↓
Worker Pool
   ↓
Available Worker
   ↓
Execution

El Resource Architecture controla el recurso.

Runtime Capacity controla la capacidad del pool.

41. Autoscaling

Autoscaling adapta capacidad a demanda.

Demand
  ↓
Scaling Signal
  ↓
Scaling Decision
  ↓
Provision / Remove Workers
42. Scaling Signals

Se pueden utilizar:

CPU utilization
memory utilization
queue depth
queue age
concurrency
throughput
latency
execution backlog
custom workload metric
43. Scaling Policy

Una política puede definir:

scaleUpThreshold
scaleDownThreshold
minCapacity
maxCapacity
cooldown
step
44. Scale Up
Demand ↑
   ↓
Capacity Pressure ↑
   ↓
Scale Up
   ↓
Capacity ↑
45. Scale Down
Demand ↓
   ↓
Low Utilization
   ↓
Cooldown
   ↓
Scale Down

Debe evitarse el oscillation loop.

46. Scaling Hysteresis

Debe existir diferencia entre:

scale-up threshold
scale-down threshold

Ejemplo:

scale up  > 70%
scale down < 30%

Esto evita:

up
down
up
down

continuamente.

47. Cooldown

Después de escalar:

Scale
 ↓
Cooldown
 ↓
Observe
 ↓
Scale Again

evita decisiones prematuras.

48. Scaling Step

Puede escalarse:

+1 worker
+5 workers
+25%

según política.

49. Predictive Scaling

Además de señales actuales:

historical demand
scheduled workloads
known peaks
seasonality

puede anticiparse capacidad.

50. Scheduled Capacity

EVOXA puede conocer:

08:00 → expected load 20%
12:00 → expected load 70%
18:00 → expected load 30%

y preparar capacidad.

51. Capacity Reservation

Puede reservarse capacidad para:

critical workflows
scheduled workloads
priority tenants
maintenance windows
52. Priority Capacity

Puede dividirse:

Critical Pool
High Pool
Normal Pool
Best Effort Pool
53. Capacity Partitioning
Runtime
├── Critical 20%
├── High     30%
├── Normal   40%
└── BestEffort 10%

Esto evita que workloads secundarios consuman todo el Runtime.

54. Elastic Capacity

La capacidad puede crecer:

Baseline
   ↓
Elastic Expansion
   ↓
Peak
   ↓
Elastic Contraction
55. Hard Capacity

Algunos límites no pueden superarse:

maximum workers
maximum memory
maximum connections
maximum tenant quota
56. Soft Capacity

Otros límites pueden superarse temporalmente:

target workers
preferred utilization
soft queue threshold
57. Capacity Guardrails

Guardrails protegen contra decisiones de scaling incorrectas:

minimum capacity
maximum capacity
maximum scale step
maximum growth rate
maximum cost
58. Backpressure

Backpressure transmite la presión desde el Runtime hacia upstream.

Runtime Saturated
      ↓
Backpressure
      ↓
Producer slows
59. Backpressure Strategies
slow producer
queue
delay
throttle
reject
shed load
60. Load Shedding

Cuando la capacidad es insuficiente:

Critical Work
     ↓
Preserve

Best Effort Work
     ↓
Drop / Defer
61. Load Shedding Policy

Debe ser explícita:

priority
tenant
workload type
deadline
cost
importance

Nunca debe depender de comportamiento accidental.

62. Queue Architecture

Queues absorben diferencias entre:

arrival rate
service rate

Modelo:

Producer
   ↓
Queue
   ↓
Consumer
63. Queue Capacity

Toda queue debe tener:

maxDepth
retention
priority
overflowPolicy
64. Queue Overflow

Cuando la queue alcanza capacidad:

QUEUE_FULL

Posibles acciones:

reject
drop low priority
spillover
defer upstream
65. Queue Age

No basta con medir profundidad.

Debe medirse:

oldest message age

Porque:

queue depth = 100

puede ser normal si los trabajos son rápidos.

Mientras:

queue depth = 10
age = 30 minutes

indica problema.

66. Queue Drain Rate

Debe medirse:

enqueue rate
dequeue rate
completion rate
67. Capacity Forecast

Puede estimarse:

Time to Saturation

basándose en:

current queue
arrival rate
service rate
available capacity
68. Capacity Exhaustion

Cuando:

available capacity = 0

el Runtime debe pasar a una estrategia explícita:

queue
throttle
reject
shed
scale
69. Dependency Capacity

El Runtime no opera aisladamente.

Puede estar limitado por:

Database
External API
Message Broker
Storage
Network
Identity Provider
70. Dependency-Aware Capacity

Ejemplo:

Workers = 100
Database connections = 20

No significa:

100 database-heavy executions

La capacity planner debe considerar la dependencia.

71. Capacity Propagation

Si una dependencia se satura:

Database Saturation
       ↓
Execution latency ↑
       ↓
Workers occupied longer
       ↓
Concurrency ↑
       ↓
Runtime saturation

EVOXA debe detectar esta cadena.

72. Resource-Aware Scaling

No debe escalar workers si el verdadero bottleneck es externo.

Incorrecto:

DB saturated
 ↓
Add 100 workers

Correcto:

DB saturated
 ↓
Limit DB-bound concurrency
 ↓
Backpressure
 ↓
Protect DB
73. Capacity Profiles

Una workload puede tener un perfil:

CPU_BOUND
MEMORY_BOUND
IO_BOUND
NETWORK_BOUND
DATABASE_BOUND
EXTERNAL_API_BOUND

El Runtime puede utilizarlo para optimizar scaling.

74. Capacity Class

Execution puede declarar:

capacityClass = CPU_HEAVY

o:

capacityClass = IO_HEAVY

Esto permite selección de pools apropiados.

75. Runtime Admission Pipeline
Incoming Execution
        ↓
Identity / Tenant
        ↓
Policy
        ↓
Quota
        ↓
Capacity Check
        ↓
Dependency Check
        ↓
Concurrency Check
        ↓
Priority
        ↓
Admission
76. Capacity Decision

El resultado puede ser:

ACCEPT_NOW
ACCEPT_QUEUE
DEFER
THROTTLE
REJECT
77. Capacity Decision Record

Debe poder registrarse:

capacityDecision
├── executionId
├── timestamp
├── availableCapacity
├── requestedCapacity
├── limitingDimension
├── decision
└── reason
78. Capacity Accounting

El Runtime debe saber:

allocated
active
queued
reserved
available

Ejemplo:

Workers = 20

Active      = 12
Reserved    = 3
Queued      = 10
Available   = 5
79. Capacity Reservation Accounting

Reservation no debe confundirse con active usage.

Total Capacity
   ├── Active
   ├── Reserved
   └── Free
80. Capacity Reconciliation

Debe compararse:

Expected Capacity
vs
Observed Capacity

Ejemplo:

Expected workers = 10
Observed workers = 8

Debe disparar reconciliación.

81. Capacity Drift

Drift:

Declared capacity ≠ actual capacity

Puede producir:

over-admission
under-utilization
unexpected saturation
82. Capacity Health

Un Runtime puede tener:

HEALTHY
DEGRADED
SATURATED
CRITICAL
DRAINING
83. Capacity Health Signals
queue growth
latency
error rate
worker availability
resource utilization
dependency saturation
84. Capacity SLO

Debe poder definirse:

Admission latency
Execution start latency
Queue age
Throughput
Capacity availability

Ejemplo:

99% of accepted work starts within 5 seconds.
85. Capacity Alerting

Alertas importantes:

capacity_low
capacity_exhausted
queue_growth
queue_age_high
scaling_failed
scaling_stuck
worker_shortage
dependency_saturation
capacity_drift
86. Capacity Failure Modes
NO_WORKERS
NO_MEMORY
CPU_SATURATION
QUEUE_FULL
DEPENDENCY_LIMIT
QUOTA_LIMIT
SCALING_FAILURE
RESOURCE_UNAVAILABLE
87. Scaling Failure

Si scaling falla:

Demand ↑
 ↓
Scale Request
 ↓
Failure

El Runtime debe aplicar:

retry
fallback
throttle
queue
reject

No asumir que la capacidad aparecerá.

88. Capacity Degradation

En condiciones extremas:

Normal
 ↓
Degraded
 ↓
Critical

pueden deshabilitarse:

non-critical workloads
background jobs
optional AI operations
analytics
precomputation

para preservar funciones críticas.

89. Graceful Degradation

La degradación debe ser:

explicit
policy-driven
observable
reversible
90. Capacity Priority Model

Una jerarquía recomendada:

P0 Critical
P1 High
P2 Normal
P3 Background
P4 Best Effort
91. Capacity Fairness

Dentro de la misma prioridad:

Tenant A
Tenant B
Tenant C

deben recibir capacidad según:

quota
weight
fairness policy
92. Weighted Capacity

Ejemplo:

Tenant A → weight 5
Tenant B → weight 3
Tenant C → weight 2

Si existe capacidad limitada:

A ≈ 50%
B ≈ 30%
C ≈ 20%

subject to limits and workload.

93. Capacity Starvation

Starvation ocurre cuando una clase nunca obtiene capacidad.

Debe detectarse:

queued too long

y mitigarse mediante:

aging
priority adjustment
fair scheduling
94. Priority Aging

Un workload que espera demasiado puede aumentar temporalmente su prioridad.

P3
 ↓
wait
 ↓
P2
 ↓
wait
 ↓
P1

Debe estar limitado por política.

95. Capacity Deadline

Cada Execution puede tener:

deadline

Si la capacidad no estará disponible antes:

deadline

puede ser mejor:

reject
cancel
defer

en lugar de ejecutar tarde.

96. Capacity-Aware Scheduling

Scheduler debe conocer:

capacity state

para evitar programar trabajos que no podrán ejecutarse.

97. Capacity-Aware Workflow

Workflow Engine puede pausar:

waiting for capacity

en vez de generar executions que inmediatamente fallen.

98. Capacity-Aware Agent

Agents deben conocer:

available capacity

para evitar generar acciones imposibles de ejecutar.

Esto es especialmente importante en workloads AI.

99. AI Capacity

EVOXA AI puede consumir:

GPU
CPU
memory
model quota
token quota
provider rate limit
inference concurrency

E37 debe tratar estos límites como capacidad operativa.

100. AI Capacity Example
Model A
max concurrency = 20
provider quota = 10 req/sec

Aunque existan:

100 workers

la capacidad efectiva de inferencia sigue estando limitada por:

10 req/sec
20 concurrent requests
101. Capacity Cost

La capacidad puede tener coste:

worker-hour
GPU-second
API-call
database operation
storage
network

Scaling debe considerar:

capacity
+
cost

cuando governance lo permita.

102. Capacity Budget

Un Runtime puede tener:

daily budget
hourly budget
tenant budget
workflow budget

Al alcanzar el presupuesto:

throttle
degrade
reject

según policy.

103. Capacity Governance

Governance define límites estratégicos.

Runtime Capacity ejecuta:

enforcement
measurement
adaptation
104. Capacity Observability Model
Runtime
  ↓
Capacity
  ├── utilization
  ├── saturation
  ├── queue
  ├── concurrency
  ├── workers
  ├── scaling
  └── dependencies
105. Core Metrics
runtime_capacity_total
runtime_capacity_available
runtime_capacity_allocated
runtime_capacity_utilization
runtime_capacity_saturation
runtime_concurrency
runtime_max_concurrency
runtime_queue_depth
runtime_queue_age
runtime_throughput
runtime_arrival_rate
runtime_service_rate
runtime_scale_up
runtime_scale_down
runtime_worker_count
runtime_worker_available
runtime_worker_busy
runtime_admission_rate
runtime_rejection_rate
runtime_throttle_rate
106. Capacity Events

Eventos mínimos:

CapacityThresholdReached
CapacityExhausted
CapacityRestored
ScalingRequested
ScalingCompleted
ScalingFailed
WorkAdmitted
WorkQueued
WorkThrottled
WorkRejected
CapacityDegraded
CapacityRecovered
RuntimeDraining
107. Capacity State Machine
                         ┌───────────┐
                         │   OPEN    │
                         └─────┬─────┘
                               │
                       pressure increases
                               ▼
                       ┌──────────────┐
                       │   LIMITED    │
                       └──────┬───────┘
                              │
                         saturation
                              ▼
                       ┌──────────────┐
                       │  THROTTLED   │
                       └──────┬───────┘
                              │
                        capacity full
                              ▼
                       ┌──────────────┐
                       │ QUEUE_ONLY   │
                       └──────┬───────┘
                              │
                       no capacity
                              ▼
                       ┌──────────────┐
                       │  REJECTING   │
                       └──────────────┘

Recovery:
REJECTING → QUEUE_ONLY → THROTTLED → LIMITED → OPEN
108. Capacity Control Loop

El mecanismo fundamental:

             ┌───────────────┐
             │    Demand     │
             └───────┬───────┘
                     ▼
             ┌───────────────┐
             │   Measure     │
             └───────┬───────┘
                     ▼
             ┌───────────────┐
             │ Analyze Load  │
             └───────┬───────┘
                     ▼
             ┌───────────────┐
             │ Decide        │
             │ Capacity      │
             └───────┬───────┘
                     ▼
             ┌───────────────┐
             │ Scale/Throttle│
             └───────┬───────┘
                     ▼
             ┌───────────────┐
             │ Observe       │
             └───────┬───────┘
                     │
                     └───────────►
109. Capacity Controller

El Controller coordina:

Demand
Capacity
Scaling
Admission
Backpressure

No ejecuta la workload directamente.

110. Reference Runtime Capacity Architecture
                         ┌──────────────────────┐
                         │      WORKLOAD        │
                         └──────────┬───────────┘
                                    ▼
                         ┌──────────────────────┐
                         │ ADMISSION CONTROLLER │
                         └──────────┬───────────┘
                                    │
                ┌───────────────────┼──────────────────┐
                ▼                   ▼                  ▼
          Policy/Quota        Capacity State       Priority
                │                   │                  │
                └───────────────────┼──────────────────┘
                                    ▼
                         ┌──────────────────────┐
                         │   QUEUE / SCHEDULER  │
                         └──────────┬───────────┘
                                    ▼
                         ┌──────────────────────┐
                         │    WORKER POOLS      │
                         ├──────────────────────┤
                         │ Critical             │
                         │ High                 │
                         │ Normal               │
                         │ Background           │
                         └──────────┬───────────┘
                                    ▼
                         ┌──────────────────────┐
                         │      EXECUTION       │
                         └──────────┬───────────┘
                                    │
                     ┌──────────────┼──────────────┐
                     ▼              ▼              ▼
                  Resource       Dependency      Runtime
                  Capacity        Capacity       Capacity
                     │              │              │
                     └──────────────┼──────────────┘
                                    ▼
                         ┌──────────────────────┐
                         │   CAPACITY CONTROL   │
                         └──────────┬───────────┘
                                    ▼
                         ┌──────────────────────┐
                         │ AUTOSCALER / CONTROL │
                         └──────────────────────┘
111. Relationship with E36

La separación queda:

E36 Resource Architecture
        ↓
"What resources exist and how are they allocated?"

E37 Runtime Capacity Architecture
        ↓
"How much workload can the runtime safely execute?"

Por tanto:

Resource
   ↓
Allocation
   ↓
Runtime Capacity
   ↓
Admission
   ↓
Execution
112. Relationship with E35
E35 Execution
      │
      ▼
Needs capacity
      │
      ▼
E37 Capacity
      │
      ▼
Admission
      │
      ▼
E35 Execution starts

Execution no debe iniciar si el Runtime no puede garantizar la capacidad requerida, salvo que la política permita ejecución degradada.

113. Relationship with Scheduling
Scheduling
→ cuándo ejecutar

Capacity
→ si puede ejecutarse

Execution
→ ejecutar

La decisión final puede ser:

Scheduled
   ↓
Capacity unavailable
   ↓
Deferred
114. Relationship with Resource Architecture
E36 Resource
   ↓
available capacity
   ↓
E37 Runtime Capacity
   ↓
effective execution capacity

E36 administra Resources.

E37 administra capacidad operativa agregada.

115. Relationship with Resilience

La capacidad es parte de la resiliencia.

Cuando existe overload:

Demand ↑
 ↓
Capacity pressure
 ↓
Backpressure
 ↓
Load shedding
 ↓
Protected core functionality

El siguiente capítulo debe formalizar cómo EVOXA tolera fallos, no simplemente exceso de carga.

116. Capacity Invariants
R1 — Runtime must never knowingly exceed hard capacity limits.

R2 — Every accepted execution must have a capacity path.

R3 — Capacity decisions must be observable.

R4 — Capacity limits must be policy-controlled.

R5 — Tenant capacity must respect isolation boundaries.

R6 — Scaling must respect minimum and maximum limits.

R7 — Scaling failure must not imply infinite capacity.

R8 — Queue growth must be observable.

R9 — Saturation must trigger an explicit control strategy.

R10 — External dependency capacity must be considered.

R11 — Backpressure must propagate toward workload producers.

R12 — Load shedding must be policy-driven.

R13 — Capacity accounting must distinguish reserved, active and available capacity.

R14 — Capacity state must be reconcilable with actual runtime state.

R15 — Draining runtimes must not admit new work.

R16 — Critical workloads must be protectable from lower-priority workloads.

R17 — Capacity must be measurable across tenants and workload classes.

R18 — Scaling must not bypass security, policy or resource constraints.
117. Engineering Completion Criteria

E37 queda completo cuando EVOXA posee:

✓ Capacity model
✓ Capacity dimensions
✓ Capacity units
✓ Capacity envelope
✓ Minimum capacity
✓ Target capacity
✓ Maximum capacity
✓ Burst capacity
✓ Headroom
✓ Utilization
✓ Saturation
✓ Bottleneck detection
✓ Admission control
✓ Admission states
✓ Workload model
✓ Throughput model
✓ Arrival rate
✓ Service rate
✓ Concurrency model
✓ Concurrency limits
✓ Worker model
✓ Worker pools
✓ Worker lifecycle
✓ Autoscaling
✓ Scaling signals
✓ Scaling policies
✓ Hysteresis
✓ Cooldown
✓ Predictive scaling
✓ Scheduled capacity
✓ Capacity reservation
✓ Priority capacity
✓ Capacity partitioning
✓ Elastic capacity
✓ Hard/soft limits
✓ Guardrails
✓ Backpressure
✓ Load shedding
✓ Queue capacity
✓ Queue overflow
✓ Queue age
✓ Queue drain rate
✓ Dependency capacity
✓ Resource-aware scaling
✓ Capacity profiles
✓ Capacity classes
✓ Admission pipeline
✓ Capacity accounting
✓ Capacity reconciliation
✓ Capacity drift
✓ Capacity health
✓ Capacity SLO
✓ Capacity alerting
✓ Failure modes
✓ Scaling failure handling
✓ Graceful degradation
✓ Priority model
✓ Fairness
✓ Starvation prevention
✓ Priority aging
✓ Capacity deadlines
✓ Capacity-aware scheduling
✓ Capacity-aware workflows
✓ Capacity-aware agents
✓ AI capacity
✓ Capacity cost
✓ Capacity budgets
✓ Governance integration
✓ Observability
✓ Capacity events
✓ Capacity state machine
✓ Capacity control loop
✓ Capacity controller
✓ Runtime integration
✓ Resource integration
✓ Execution integration
✓ Resilience boundary
✓ Capacity invariants
118. Cadena operativa consolidada

Con E36 + E37, la arquitectura empieza a formar un Runtime realmente controlable:

                         DECISION
                            │
                            ▼
                          ACTION
                            │
                            ▼
                        EXECUTION
                            │
                            ▼
                    RESOURCE REQUIREMENT
                            │
                            ▼
                      RESOURCE E36
                            │
                     allocation
                            ▼
                  RUNTIME CAPACITY E37
                            │
                 ┌──────────┼──────────┐
                 ▼          ▼          ▼
             Admission   Queue     Scaling
                 │          │          │
                 └──────────┼──────────┘
                            ▼
                         WORKER
                            │
                            ▼
                        EXECUTION
                            │
                            ▼
                          RESULT

La regla arquitectónica queda:

E36 determina qué recursos pueden utilizarse. E37 determina cuánta carga puede absorber el Runtime con esos recursos. E35 ejecuta el trabajo.

Y el siguiente paso natural es E38 — EVOXA Resilience Architecture, donde debemos formalizar failure domains, fault isolation, retries, circuit breakers, timeouts, bulkheads, failover, recovery, degradation y disaster recovery, conectándolos con la capacidad que acabamos de definir.
