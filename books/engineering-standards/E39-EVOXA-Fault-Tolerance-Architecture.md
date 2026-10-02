E39 — EVOXA Fault Tolerance Architecture
Engineering Specification

Pertenece a: Engineering Specification
Capítulo: E39 — Fault Tolerance Architecture
Anterior: E38 — Resilience Architecture
Siguiente: E40 — EVOXA Recovery Architecture

1. Propósito

E39 define los mecanismos mediante los cuales EVOXA continúa operando correctamente cuando uno o más componentes fallan, reduciendo la probabilidad de pérdida de servicio, pérdida de estado, corrupción de datos o interrupción de operaciones críticas.

La distinción fundamental es:

Resilience defines how EVOXA behaves under failure. Fault Tolerance defines how EVOXA continues functioning despite failure.

E38 establece la estrategia general de resiliencia.

E39 establece los mecanismos estructurales que hacen posible esa resiliencia.

2. Fault Tolerance Boundary

E39 cubre:

Fault Detection
Fault Isolation
Redundancy
Replication
Failover
Quorum
Leader Election
State Protection
Checkpointing
Duplicate Protection
Consistency Protection
Availability Preservation
Failure Containment

No reemplaza:

E37 → Capacity
E38 → Resilience
E40 → Recovery
E41 → Disaster Recovery
3. Fault Tolerance Model

El modelo fundamental es:

                NORMAL
                   │
                   ▼
                 FAULT
                   │
                   ▼
             DETECTION
                   │
                   ▼
              ISOLATION
                   │
                   ▼
          REDUNDANT PATH
                   │
                   ▼
              CONTINUE
                   │
                   ▼
              REPAIR
                   │
                   ▼
              REINTEGRATE
4. Fault Tolerance Objective

EVOXA debe poder mantener:

Availability
+
Correctness
+
Consistency
+
Isolation
+
Continuity

cuando un componente individual o un subconjunto limitado de componentes falla.

5. Fault Model

EVOXA debe reconocer diferentes tipos de faults:

Crash Fault
Transient Fault
Omission Fault
Timing Fault
Network Fault
Storage Fault
Dependency Fault
Byzantine-like Fault
Configuration Fault
Resource Fault
6. Crash Fault

Un componente deja de responder:

Worker
   ↓
Crash

El sistema debe detectar:

Worker unhealthy

y retirar el componente del tráfico.

7. Transient Fault

Fallo temporal:

Network
   ↓
temporary failure
   ↓
recovery

Debe evitarse una respuesta excesivamente agresiva.

8. Omission Fault

Un componente:

does not receive

o:

does not send

un mensaje esperado.

Puede detectarse mediante:

timeout
heartbeat
sequence
acknowledgement
9. Timing Fault

El componente responde:

too late

Aunque eventualmente responda correctamente, puede haber violado el deadline operativo.

10. Network Fault

Incluye:

partition
latency
packet loss
connection reset
DNS failure
routing failure

Debe tratarse como una posible pérdida parcial de comunicación, no necesariamente como una caída completa del sistema.

11. Storage Fault

Puede producir:

read failure
write failure
partial write
stale replica
corruption
unavailable volume

La estrategia debe priorizar integridad.

12. Dependency Fault
External Provider
       ↓
Failure
       ↓
EVOXA

El sistema debe poder continuar cuando exista una alternativa segura.

13. Resource Fault

Ejemplos:

memory exhausted
CPU unavailable
worker unavailable
connection pool exhausted
GPU failure
disk unavailable
14. Fault Domain

EVOXA debe separar faults por dominio:

Process
Worker
Service
Host
Cluster
Zone
Region
Provider
Tenant
Domain
Data Store
15. Fault Domain Principle

Regla:

Una falla debe afectar el menor dominio posible.

Ejemplo:

Worker A
   ↓
failure
   ↓
Worker A removed
   ↓
Worker Pool continues

No:

Worker A
   ↓
failure
   ↓
Entire Runtime unavailable
16. Redundancy

La tolerancia a fallos depende de redundancia:

Primary
   +
Secondary

o:

Node A
Node B
Node C

La redundancia puede aplicarse a:

Compute
Storage
Network
Services
Workers
Data
Providers
Regions
17. Active-Passive Redundancy
Primary → ACTIVE
Secondary → STANDBY

Si Primary falla:

Secondary → ACTIVE

Ventajas:

simple
predictable

Desventaja:

standby capacity
18. Active-Active Redundancy
Node A → traffic
Node B → traffic
Node C → traffic

Permite:

load distribution
fault tolerance
capacity utilization

pero requiere coordinación más compleja.

19. N+1 Redundancy

Ejemplo:

Required = 4
Available = 5

Un componente puede fallar sin perder capacidad mínima.

20. N+2 Redundancy
Required = 4
Available = 6

Permite tolerar dos fallos dentro del modelo definido.

21. Redundancy Level

Cada componente crítico debe declarar:

redundancyLevel
failureTolerance
minimumOperationalInstances
22. Replication

Replication mantiene múltiples copias:

Primary
  │
  ├── Replica A
  ├── Replica B
  └── Replica C

Puede utilizarse para:

availability
read scaling
failover
data protection
23. Replication Types
Synchronous
Asynchronous
Semi-synchronous
24. Synchronous Replication

La operación espera confirmación de múltiples replicas.

Ventaja:

stronger consistency

Coste:

higher latency
25. Asynchronous Replication
Primary
   ↓
ack
   ↓
Replica later

Ventaja:

lower latency

Riesgo:

replication lag
26. Replication Lag

Debe medirse:

replicaLag

Una replica demasiado atrasada puede no ser válida para failover.

27. Failover Eligibility

No toda replica puede convertirse automáticamente en primary.

Debe verificarse:

healthy
sufficiently synchronized
authorized
reachable
correct version
28. Failover
Primary
   ↓
Failure
   ↓
Detect
   ↓
Select Replica
   ↓
Promote
   ↓
Route Traffic
29. Failover Safety

Nunca debe producirse:

Primary A
     +
Primary B

sin coordinación.

Esto puede generar:

split brain
30. Split Brain

Ocurre cuando dos nodos creen simultáneamente ser primary.

      Network Partition
       /             \
      A               B
  "I am primary"  "I am primary"

Debe prevenirse mediante:

quorum
leader election
fencing
leases
31. Fencing

Fencing impide que un nodo antiguo continúe modificando estado después de perder liderazgo.

Ejemplo:

Old Primary
   ↓
Leadership lost
   ↓
Fenced
   ↓
Cannot write
32. Leader Election

Cuando existe un rol único:

Leader

debe existir un mecanismo para seleccionar uno.

Node A
Node B
Node C
   ↓
Election
   ↓
Leader B
33. Leader Lease

El liderazgo puede estar limitado temporalmente:

Lease
=
Leadership valid until T

Si no se renueva:

Leader
   ↓
Lease expires
   ↓
Leadership lost
34. Quorum

Para decisiones distribuidas:

N nodes

puede requerirse:

quorum > N/2

Ejemplo:

3 nodes
quorum = 2
35. Quorum Purpose

Quorum evita:

two independent majorities

y ayuda a garantizar:

single authoritative state
36. Read Quorum

Para lecturas distribuidas puede definirse:

R

número de replicas requeridas.

37. Write Quorum

Para writes:

W

número de replicas necesarias.

En ciertos modelos:

R + W > N

ayuda a garantizar intersección entre lecturas y escrituras.

38. Quorum Availability

Debe existir una política para cuando no existe quorum:

NO_QUORUM

Posibles respuestas:

reject writes
read degraded
enter safe mode
wait
39. Safe Mode

Cuando la integridad no puede garantizarse:

NORMAL
  ↓
FAULT
  ↓
SAFE MODE

Safe Mode puede limitar:

writes
commands
external side effects
configuration changes
40. State Preservation

Fault tolerance debe proteger:

runtime state
workflow state
job state
execution state
transaction state
agent state
domain state
41. State Categories
Ephemeral State
Durable State
Derived State
Reconstructable State
Critical State
42. Ephemeral State

Puede perderse sin corrupción:

cache
temporary buffers
worker-local state

Debe poder reconstruirse.

43. Durable State

Debe sobrevivir al proceso:

business state
workflow state
audit records
configuration
44. Derived State

Puede reconstruirse desde source of truth:

read models
search indexes
analytics projections
45. Checkpointing

Para operaciones largas:

Execution
   ↓
Checkpoint
   ↓
Continue

Si ocurre un fallo:

Failure
   ↓
Restore checkpoint
   ↓
Resume
46. Checkpoint Frequency

Debe balancearse:

checkpoint cost
vs
recovery work
47. Execution Recovery
Execution
   ↓
Running
   ↓
Failure
   ↓
Checkpoint
   ↓
Resume

Cuando no exista checkpoint:

restart

o:

abort

según policy.

48. Workflow Fault Tolerance

Un Workflow debe soportar:

activity failure
worker failure
dependency failure
orchestrator failure

mediante:

checkpoint
retry
resume
compensation
49. Job Fault Tolerance

Jobs pueden utilizar:

persistent queue
acknowledgement
lease
retry
dead-letter
checkpoint
50. Lease-Based Execution

Un worker puede adquirir:

Execution Lease

Si el worker desaparece:

Lease expires
   ↓
Execution becomes recoverable
51. Lease Renewal

Mientras el worker está activo:

Lease
 ↓
Renew
 ↓
Renew
 ↓
Renew

Si no puede renovarse:

Execution ownership lost
52. Duplicate Execution

Tras una lease expiry puede ocurrir:

Worker A
   ↓
slow

mientras:

Worker B
   ↓
reclaims execution

Puede producirse ejecución duplicada.

Por ello:

idempotency
fencing
deduplication

son esenciales.

53. Execution Ownership

Toda ejecución distribuida debe tener:

executionId
ownerId
leaseId
version
54. Fencing Token

Una ejecución puede utilizar un token monotónico:

token 41
token 42
token 43

Solo el token vigente puede modificar estado.

Esto protege contra workers antiguos.

55. Optimistic Concurrency

Para estado compartido:

version = 10

Update:

WHERE version = 10

Si otro actor ya cambió el estado:

version = 11

el update falla.

56. Pessimistic Coordination

Cuando sea necesario:

lock
lease
reservation

pueden impedir modificaciones simultáneas.

Debe evitarse mantener locks demasiado tiempo.

57. State Conflict

Si dos actores producen:

State A
State B

EVOXA debe tener una estrategia:

reject
merge
last-writer
version conflict
manual resolution

según el dominio.

58. Message Durability

Para fault tolerance:

Message
   ↓
Durable Broker
   ↓
Consumer

El mensaje no debe desaparecer simplemente porque el consumer falle.

59. Acknowledgement

El consumer debe confirmar:

ACK

solo cuando el procesamiento haya alcanzado el estado requerido.

60. At-Least-Once Processing

El modelo recomendado para muchas operaciones:

delivery = at least once
processing = idempotent

Esto favorece:

availability
recovery

sin asumir exactly-once imposible de garantizar de forma global.

61. Dead Letter

Mensajes que no pueden procesarse:

Queue
  ↓
Retries
  ↓
Dead Letter

No deben bloquear indefinidamente la cola principal.

62. Poison Message

Un mensaje puede fallar siempre.

Message X
 ↓
retry
 ↓
retry
 ↓
retry

Debe terminar en:

dead-letter
quarantine
manual review

según policy.

63. Storage Fault Tolerance

Storage crítico debe contemplar:

replication
backup
checksums
versioning
transactionality
failover
64. Database Fault Tolerance

Debe poder soportar:

primary failure
connection failure
replica failure
network partition

mediante arquitectura apropiada.

65. Connection Failover

Los clientes no deben mantener indefinidamente conexiones hacia un nodo fallido.

Debe existir:

connection health
reconnect policy
endpoint discovery
66. Network Fault Tolerance

Debe utilizar:

timeouts
retries
alternate routes
connection pools
health checks

pero evitando retry storms.

67. Service Redundancy

Servicios críticos:

API
Scheduler
Workflow Engine
Event Processor
Agent Runtime
Execution Engine

deben poder tener múltiples instancias cuando el SLA lo requiera.

68. Stateless Service Tolerance

Servicios stateless son más fáciles de reemplazar:

Instance A
Instance B
Instance C

Si A falla:

Traffic → B/C
69. Stateful Service Tolerance

Stateful components requieren:

replication
ownership
leader
quorum
state recovery
70. Control Plane vs Data Plane

EVOXA debe separar:

Control Plane
Data Plane
71. Control Plane Fault

Si falla el Control Plane:

configuration
scheduling
coordination

pueden degradarse.

Pero el Data Plane debería continuar durante una ventana segura cuando sea posible.

72. Data Plane Fault

Un fallo de execution no debería necesariamente destruir:

governance
observability
control
73. Control Plane Resilience

Debe disponer de:

redundancy
leader election
persistent state
safe defaults
74. Data Plane Resilience

Debe disponer de:

worker redundancy
queue durability
execution recovery
capacity controls
75. Fault Tolerance Across Tenants

La redundancia debe respetar:

tenant isolation
tenant quota
tenant data boundary
tenant encryption boundary

No utilizar un failover que accidentalmente mezcle estados de tenants.

76. Fault Tolerance for Agents

Agents pueden fallar durante:

planning
reasoning
tool execution
action execution

Debe conservarse:

agent session
execution state
action history
decision context

según las políticas de privacidad y persistencia.

77. Agent Recovery
Agent
 ↓
Failure
 ↓
Recover state
 ↓
Resume

Pero no debe reemitir automáticamente acciones no idempotentes.

78. Fault Tolerance for AI

AI provider failure:

Provider A
   ↓
failure

posibles estrategias:

Provider B
cached result
smaller model
degraded mode
queue
79. AI Safety Boundary

La tolerancia a fallos nunca debe provocar:

unsafe action
unauthorized action
duplicate external action

La disponibilidad no puede saltarse:

authorization
policy
validation
80. External Side Effects

Para:

payment
email
notification
command
webhook
device action

debe existir:

idempotency
deduplication
execution record
confirmation
81. Transactional Outbox

Cuando un cambio de estado y un evento deben mantenerse coordinados:

Transaction
├── Domain State
└── Outbox Event

Después:

Outbox
 ↓
Publisher

Esto reduce pérdida de eventos.

82. Inbox Pattern

Para evitar procesamiento duplicado:

Incoming Event
   ↓
Inbox
   ↓
Already processed?
   ├── yes → ignore
   └── no → process
83. Exactly-Once Business Effect

Aunque el transporte pueda ser:

at-least-once

el efecto de negocio puede diseñarse para comportarse como:

effectively once

mediante:

idempotency key
deduplication
transaction
state version
84. Fault-Tolerant Transactions

Una operación crítica debe tener:

begin
 ↓
state transition
 ↓
durable commit
 ↓
event publication

con garantías explícitas sobre cada frontera.

85. Partial Failure

El caso más peligroso:

A succeeded
B failed
C unknown

EVOXA debe registrar el estado intermedio.

Nunca asumir:

unknown = failed

ni:

unknown = succeeded

sin evidencia.

86. Unknown Outcome

Una operación puede terminar en:

SUCCESS
FAILURE
UNKNOWN

UNKNOWN requiere reconciliación.

87. Reconciliation
Unknown
  ↓
Query source
  ↓
Determine actual state
  ↓
Reconcile

Esto es fundamental para efectos externos.

88. Fault-Tolerant Coordination

Componentes coordinados deben utilizar:

leases
epochs
versions
quorum
fencing

para evitar decisiones simultáneas incompatibles.

89. Epoch

Cada generación de liderazgo puede tener:

epoch = 17

Un actor con:

epoch = 16

ya no puede modificar estado.

90. Versioned State

El estado crítico debe ser versionable:

State v41
State v42
State v43

Permite:

conflict detection
recovery
audit
reconciliation
91. Fault-Tolerant Configuration

Configuration debe soportar:

version
validation
rollback
safe default

Una nueva configuración inválida no debe dejar todos los nodos inutilizables.

92. Rolling Fault Tolerance

Durante deployment:

Version A
 ↓
A A A A

Deploy B

A A A B
A A B B
A B B B
B B B B

Si B falla:

rollback
93. Compatibility Window

Versiones simultáneas deben ser compatibles durante transición:

A ↔ B

especialmente en:

events
API
schemas
database
messages
94. Fault-Tolerant Events

Event consumers deben tolerar:

duplicate
out-of-order
delayed
missing
replayed

cuando el modelo de eventos lo permita.

95. Event Replay

Si un projection o consumer falla:

Event Log
   ↓
Replay
   ↓
Rebuild State

Esto convierte eventos durables en mecanismo de recuperación.

96. Projection Fault Tolerance

Una projection caída:

Projection
 ↓
failure

puede reconstruirse desde:

source events

si la arquitectura lo permite.

97. Search Fault Tolerance

Search puede ser tratado como:

derived state

y reconstruirse desde source of truth.

Durante reconstrucción:

search degraded

pero:

core domain state

permanece disponible.

98. Cache Fault Tolerance

Cache failure debe causar:

cache miss

no:

system failure

siempre que el backend primario siga disponible.

99. Observability Fault Tolerance

Observability no debe convertirse en dependency crítica para ejecutar workload.

Si falla:

metrics backend
log collector
trace collector

el Runtime debe continuar dentro de límites seguros.

100. Fault-Tolerant Logging

Logs críticos pueden requerir:

local buffering
durable forwarding
sampling
backpressure
101. Fault-Tolerant Metrics

Metrics pueden degradarse mediante:

sampling
aggregation
local buffering

sin bloquear execution.

102. Fault-Tolerant Control

En caso de pérdida del Control Plane:

last known safe configuration

puede mantenerse durante un período definido.

103. Safe Defaults

Cada componente crítico debe tener:

safe failure behavior

Ejemplo:

authorization unavailable
→ deny

capacity controller unavailable
→ conservative admission

policy unavailable
→ deny critical operation
104. Fail-Open vs Fail-Closed

Cada boundary debe declarar explícitamente:

FAIL_OPEN
FAIL_CLOSED
105. Fail-Closed

Preferible para:

authorization
security policy
critical writes
unsafe external actions
106. Fail-Open

Puede ser válido para:

optional analytics
non-critical telemetry
cache
decorative enrichment

solo cuando policy lo permita.

107. Fault-Tolerant Shutdown

Durante shutdown:

stop admission
 ↓
drain
 ↓
persist state
 ↓
release ownership
 ↓
stop
108. Fault-Tolerant Startup

Durante startup:

load state
 ↓
validate state
 ↓
recover leases
 ↓
health check
 ↓
join cluster
 ↓
ready
109. Rejoining Cluster

Un nodo recuperado no debe recibir inmediatamente tráfico completo.

RECOVERED
   ↓
SYNCING
   ↓
VALIDATING
   ↓
READY
   ↓
TRAFFIC
110. Recovery Lag

Debe medirse:

time to sync
replication lag
state reconstruction duration
111. Fault-Tolerant Scaling

Scaling debe conservar tolerancia a fallos:

minimum replicas >= fault tolerance requirement

No permitir:

scale down
 ↓
single instance
 ↓
single point of failure

si el componente requiere redundancy.

112. Capacity + Fault Tolerance

E37 define:

capacity

E39 define:

capacity after failure

Ejemplo:

Normal:
10 workers

One failure:
9 workers

Required minimum:
8 workers

Entonces:

fault tolerated = 1
113. Fault-Tolerant Capacity Envelope
Normal Capacity
      │
      ▼
Failure
      │
      ▼
Reduced Capacity
      │
      ▼
Minimum Safe Capacity

Si cae por debajo:

degradation

debe activarse.

114. Fault Tolerance Budget

Cada servicio crítico debe declarar:

maximum tolerated failures

Ejemplo:

tolerate 1 worker failure
tolerate 1 zone failure
115. Fault Budget vs Error Budget

No son idénticos.

Error Budget
→ cuánto error operativo es aceptable

Fault Budget
→ qué fallos estructurales puede soportar el sistema
116. Fault Tolerance Matrix
Component
│
├── Failure Domain
├── Redundancy
├── Detection
├── Isolation
├── Failover
├── State Recovery
├── Data Protection
├── Maximum Faults
└── Degradation Strategy
117. Fault Tolerance Contract

Cada componente crítico debe declarar:

faultToleranceContract
├── toleratedFaults
├── redundancyModel
├── healthModel
├── failoverStrategy
├── stateStrategy
├── recoveryStrategy
├── consistencyGuarantee
└── degradationMode
118. Reference Fault-Tolerant Runtime
                       ┌───────────────────┐
                       │     WORKLOAD      │
                       └─────────┬─────────┘
                                 ▼
                       ┌───────────────────┐
                       │     CAPACITY      │
                       │       E37         │
                       └─────────┬─────────┘
                                 ▼
                       ┌───────────────────┐
                       │    EXECUTION      │
                       └─────────┬─────────┘
                                 │
               ┌─────────────────┼─────────────────┐
               ▼                 ▼                 ▼
          Worker Pool       Queue Layer       Dependency
          A B C D           Durable Queue      A/B
               │                 │                 │
               └─────────────────┼─────────────────┘
                                 ▼
                       ┌───────────────────┐
                       │ FAULT TOLERANCE   │
                       │       E39         │
                       ├───────────────────┤
                       │ Detection         │
                       │ Isolation         │
                       │ Replication       │
                       │ Quorum            │
                       │ Election          │
                       │ Fencing           │
                       │ Failover          │
                       │ Checkpoint        │
                       │ Reconciliation    │
                       └─────────┬─────────┘
                                 ▼
                       ┌───────────────────┐
                       │    SAFE STATE     │
                       └───────────────────┘
119. Fault-Tolerance Control Loop
              ┌───────────────┐
              │    OPERATE    │
              └───────┬───────┘
                      ▼
              ┌───────────────┐
              │    DETECT     │
              └───────┬───────┘
                      ▼
              ┌───────────────┐
              │    ISOLATE    │
              └───────┬───────┘
                      ▼
              ┌───────────────┐
              │    FAILOVER   │
              └───────┬───────┘
                      ▼
              ┌───────────────┐
              │   VALIDATE    │
              └───────┬───────┘
                      ▼
              ┌───────────────┐
              │   REINTEGRATE │
              └───────┬───────┘
                      │
                      └──────────────► OPERATE
120. Relationship with E38

La separación queda:

E38 Resilience
────────────────────────────
What should EVOXA do when failure occurs?

E39 Fault Tolerance
────────────────────────────
How does EVOXA structurally continue operating despite that failure?

Ejemplo:

Dependency fails

E38:
→ activate fallback

E39:
→ redundant provider
→ circuit isolation
→ failover
→ state protection
121. Relationship with E37
E37
Capacity under normal conditions
        ↓
E39
Capacity after fault
        ↓
E38
Degradation strategy
122. Relationship with E40

E39 termina cuando el sistema consigue:

continue
protect
fail over
maintain state

E40 comenzará con:

recover
restore
rebuild
reconcile
return to normal
123. Fault-Tolerance Invariants
FT1 — No critical component may depend on a single unprotected failure point.

FT2 — Every critical failure domain must have an explicit tolerance level.

FT3 — Redundancy must not create conflicting authorities.

FT4 — Only one valid leader may control a single-writer role at a time.

FT5 — Stale leaders must be fenced.

FT6 — Failover targets must be validated before promotion.

FT7 — Replication state must be observable.

FT8 — Replica lag must be considered before failover.

FT9 — Critical state must be durable or reconstructable.

FT10 — Long-running executions must have a recovery strategy.

FT11 — Distributed execution ownership must expire safely.

FT12 — Duplicate execution must be controlled.

FT13 — Unknown operation outcomes must be reconciled.

FT14 — Messages must not be lost solely because a consumer fails.

FT15 — Poison messages must not block healthy workload indefinitely.

FT16 — Control-plane failure must not automatically destroy data-plane operation where safe.

FT17 — Observability failure must not automatically terminate workload execution.

FT18 — Recovery nodes must synchronize before receiving full traffic.

FT19 — Scaling must preserve minimum fault tolerance.

FT20 — Fail-open and fail-closed behavior must be explicit.

FT21 — Fault tolerance mechanisms must respect security and tenant boundaries.

FT22 — Fault tolerance must preserve data correctness over superficial availability.

FT23 — A fault-tolerant system must have explicit behavior for loss of quorum.

FT24 — Every critical state transition must have an authoritative owner.

FT25 — Fault tolerance mechanisms themselves must be observable and testable.
124. Engineering Completion Criteria

E39 queda completo cuando EVOXA posee:

✓ Fault model
✓ Fault taxonomy
✓ Failure domains
✓ Fault isolation
✓ Redundancy
✓ Active-passive
✓ Active-active
✓ N+1/N+2 models
✓ Replication
✓ Replication modes
✓ Replication lag
✓ Failover eligibility
✓ Failover
✓ Failover safety
✓ Split-brain protection
✓ Fencing
✓ Leader election
✓ Leader lease
✓ Quorum
✓ Safe mode
✓ State preservation
✓ State classification
✓ Checkpointing
✓ Execution recovery
✓ Workflow tolerance
✓ Job tolerance
✓ Lease-based execution
✓ Execution ownership
✓ Fencing tokens
✓ Optimistic concurrency
✓ State conflict handling
✓ Durable messaging
✓ Acknowledgements
✓ At-least-once processing
✓ Dead-letter handling
✓ Poison message handling
✓ Storage tolerance
✓ Database tolerance
✓ Network tolerance
✓ Service redundancy
✓ Stateless tolerance
✓ Stateful tolerance
✓ Control-plane tolerance
✓ Data-plane tolerance
✓ Tenant fault isolation
✓ Agent fault tolerance
✓ AI provider failover
✓ External side-effect protection
✓ Transactional outbox
✓ Inbox/deduplication
✓ Unknown outcome reconciliation
✓ Versioned state
✓ Epochs
✓ Event replay
✓ Projection reconstruction
✓ Cache tolerance
✓ Observability tolerance
✓ Safe defaults
✓ Fail-open / fail-closed policies
✓ Fault-tolerant startup
✓ Fault-tolerant shutdown
✓ Cluster rejoin
✓ Fault-tolerant scaling
✓ Fault-tolerant capacity envelope
✓ Fault budget
✓ Fault tolerance matrix
✓ Fault tolerance contract
✓ Fault tolerance control loop
✓ Fault tolerance invariants
125. Cadena Runtime consolidada

Con E37, E38 y E39:

                         WORKLOAD
                            │
                            ▼
                    ┌───────────────┐
                    │   E37        │
                    │   CAPACITY   │
                    └───────┬───────┘
                            │
                            ▼
                    ┌───────────────┐
                    │   E35        │
                    │   EXECUTION   │
                    └───────┬───────┘
                            │
                       fault/stress
                            ▼
                    ┌───────────────┐
                    │   E38        │
                    │  RESILIENCE   │
                    └───────┬───────┘
                            │
                    resilience strategy
                            ▼
                    ┌───────────────┐
                    │   E39        │
                    │ FAULT TOLERANCE│
                    └───────┬───────┘
                            │
          ┌─────────────────┼─────────────────┐
          ▼                 ▼                 ▼
      REDUNDANCY        FAILOVER          STATE PROTECTION
          │                 │                 │
          └─────────────────┼─────────────────┘
                            ▼
                     CONTINUED SERVICE
                            │
                            ▼
                    ┌───────────────┐
                    │   E40        │
                    │   RECOVERY    │
                    └───────────────┘

La frontera arquitectónica queda así:

E37 — Capacity: cuánto puede soportar EVOXA.
E38 — Resilience: cómo debe comportarse frente a condiciones adversas.
E39 — Fault Tolerance: qué redundancia, coordinación, replicación y protección de estado permiten continuar operando pese al fallo.
E40 — Recovery: cómo se reconstruye y devuelve el sistema a su estado normal después del fallo.

E40 — EVOXA Recovery Architecture es, por tanto, el siguiente capítulo natural: debe formalizar recovery orchestration, state restoration, checkpoint recovery, replay, reconciliation, repair, failback, recovery sequencing, recovery verification y retorno controlado a operación normal.
