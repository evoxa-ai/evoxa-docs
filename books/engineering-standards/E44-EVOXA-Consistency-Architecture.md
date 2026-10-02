E44 — EVOXA Consistency Architecture
Engineering Specification

Pertenece a: Engineering Specification
Capítulo: E44 — Consistency Architecture
Anterior: E43 — Data Integrity Architecture
Siguiente: E45 — EVOXA Data Lifecycle Architecture

1. Propósito

E44 define cómo EVOXA mantiene, coordina, detecta y recupera la consistencia de los datos y estados distribuidos entre:

Transactions
Aggregates
Services
Databases
Events
Message Brokers
Read Models
Projections
Caches
Search Indexes
Analytics Models
External Systems
AI / Agents

El principio central es:

EVOXA no debe asumir que todos los componentes comparten el mismo estado al mismo tiempo. Debe definir explícitamente qué significa "consistente" para cada frontera, cuánto retraso es aceptable y cómo se detectan y resuelven las divergencias.

2. Consistency Boundary

E44 cubre:

Consistency Models
Consistency Domains
Strong Consistency
Eventual Consistency
Causal Consistency
Read-Your-Writes
Monotonic Reads
Monotonic Writes
Session Consistency
Transactional Consistency
Distributed Consistency
Cross-Service Consistency
Eventual Reconciliation
Conflict Detection
Conflict Resolution
Ordering
Concurrency
Staleness
Consistency Windows
Consistency Guarantees

E44 se relaciona directamente con:

E02 → Database Architecture
E07 → Event Architecture
E12 → Messaging Architecture
E13 → Event Processing Architecture
E14 → Workflow & Orchestration
E17 → Caching
E26 → Projection
E28 → Read Models
E29 → Search
E31 → Analytics
E38 → Resilience
E40 → Recovery
E43 → Data Integrity
3. Integrity vs Consistency

La distinción debe ser explícita.

E43 — Integrity:

"¿El dato es correcto y confiable?"

E44 — Consistency:

"¿Los diferentes estados y representaciones concuerdan
según las reglas del sistema?"

Por ejemplo:

Authoritative Order = PAID

Read Model = PENDING

Ambos estados pueden ser individualmente válidos.

Pero juntos:

INCONSISTENT
4. Consistency Model

Cada dominio de EVOXA debe declarar su modelo de consistencia.

Como mínimo:

STRONG
EVENTUAL
CAUSAL
SESSION
READ-YOUR-WRITES
MONOTONIC-READ
MONOTONIC-WRITE

No debe existir un supuesto global de que:

"all EVOXA is strongly consistent"

ni:

"all EVOXA is eventually consistent"
5. Consistency Domains

La consistencia debe definirse por dominio o frontera:

Domain
  │
  ├── Aggregate
  ├── Database
  ├── Service
  ├── Event Stream
  ├── Projection
  ├── Cache
  └── External Integration

Cada uno puede tener garantías diferentes.

6. Strong Consistency

Strong consistency significa que una lectura observa el estado correcto conforme al contrato después de una escritura confirmada.

Modelo:

WRITE
  ↓
COMMIT
  ↓
READ
  ↓
NEW STATE

Debe utilizarse cuando una divergencia temporal no sea aceptable.

Ejemplos típicos:

financial balance
critical authorization state
unique resource allocation
inventory reservation
security policy state
7. Eventual Consistency

En eventual consistency:

Source
  │
  ├── immediate → New State
  │
  └── delayed → Derived State

Durante una ventana temporal:

Source != Projection

Esto no constituye necesariamente una falla.

Es válido si:

convergence is guaranteed

y el tiempo de convergencia cumple el contrato.

8. Consistency Window

Cada estado eventualmente consistente debe definir:

consistencyWindow

Ejemplo conceptual:

T0 → source updated
T1 → event published
T2 → consumer processes event
T3 → projection updated

Entonces:

T3 - T0

representa la latencia de convergencia.

9. Consistency SLA

Para cada frontera eventualmente consistente:

Expected convergence
Maximum acceptable lag
Detection threshold
Recovery strategy

deben estar definidos.

10. Causal Consistency

Si:

A → B

entonces un consumidor no debería observar:

B before A

cuando la relación causal sea parte del contrato.

Ejemplo:

Customer Created
      ↓
Order Created

Un sistema downstream no debería interpretar:

Order Created

antes de poder establecer el contexto requerido de:

Customer Created
11. Causal Chain

EVOXA debe poder representar:

Event A
  ↓
Correlation
  ↓
Event B
  ↓
Event C

mediante mecanismos como:

causationId
correlationId
aggregateId
sequence
version

según corresponda.

12. Read-Your-Writes

Después de que un usuario complete:

WRITE

su siguiente lectura puede requerir observar inmediatamente:

READ → same or newer state

Este requisito es especialmente importante para:

interactive APIs
user sessions
command/query flows
13. Monotonic Reads

Un consumidor no debería observar:

Version 10
↓
Version 8

después de haber observado una versión más reciente.

Por tanto:

observedVersion(n+1) >= observedVersion(n)

cuando el contrato exige monotonicidad.

14. Monotonic Writes

Las operaciones de un actor deben preservar el ordering requerido:

Write A
   ↓
Write B

No debe producirse:

B applied before A

si existe una dependencia causal.

15. Session Consistency

Una sesión puede requerir:

Read-Your-Writes
+
Monotonic Reads
+
Monotonic Writes

Esto es especialmente útil en:

web applications
mobile clients
interactive agents
long-running workflows
16. Transactional Consistency

Dentro de un boundary transaccional:

Transaction
   ↓
Atomic Commit

las invariantes deben mantenerse.

Pero el commit transaccional de una base de datos no implica automáticamente consistencia global entre servicios.

17. Distributed Consistency

Para:

Service A
Service B
Service C

no debe asumirse:

one global transaction

salvo que exista explícitamente.

La consistencia distribuida se obtiene mediante mecanismos como:

Saga
Outbox
Event Processing
Compensation
Reconciliation
18. Consistency Strategy Matrix
Boundary	Modelo recomendado
Aggregate transaction	Strong
Critical authorization	Strong
Unique resource allocation	Strong
Domain event stream	Ordered / causal
Read model	Eventual
Search index	Eventual
Cache	Eventual
Analytics	Eventual
External integration	Eventual / contract-dependent
Cross-service workflow	Saga / causal
Reporting	Snapshot / eventual

La tabla representa patrones de diseño, no una imposición universal. Cada dominio debe declarar su garantía real.

19. Consistency Hierarchy
                    CONSISTENCY
                         │
        ┌────────────────┼────────────────┐
        ▼                ▼                ▼
    LOCAL             DOMAIN          DISTRIBUTED
        │                │                │
        ▼                ▼                ▼
   Transaction       Aggregate          Services
   Database          Invariants         Events
                                      Projections
                                      External Systems
20. Consistency Ownership

Cada estado debe tener un owner claro:

Source of Truth
      ↓
Consistency Owner
      ↓
Derived Consumers

El consumidor derivado no debe convertirse accidentalmente en autoridad.

21. Source of Truth

Debe definirse explícitamente:

authoritativeState

Ejemplo:

Order Database
      ↓
Source of Truth

Order Read Model
      ↓
Derived State

Order Search Index
      ↓
Derived State

Order Cache
      ↓
Derived State
22. Consistency Graph

EVOXA debe poder representar:

              ┌──────────────┐
              │ Source Truth │
              └──────┬───────┘
                     │
              ┌──────▼──────┐
              │ Event Stream│
              └───┬────┬────┘
                  │    │
          ┌───────▼┐  ┌▼────────┐
          │ Read   │  │ Search  │
          │ Model  │  │ Index   │
          └────┬───┘  └─────────┘
               │
          ┌────▼─────┐
          │  Cache   │
          └───────────┘

Cada flecha debe representar una relación de sincronización.

23. Consistency Contract

Cada frontera debe definir:

consistencyModel
source
consumer
ordering
maximumLag
freshnessRequirement
failureBehavior
reconciliation
24. Example Contract
Projection: OrderReadModel

Source:
  Order Aggregate

Model:
  Eventual

Ordering:
  Aggregate sequence

Maximum Lag:
  5 seconds

Failure:
  Retry

Repair:
  Replay

Reconciliation:
  Daily + on-demand
25. Freshness

Consistency no es exactamente lo mismo que freshness.

Un modelo puede ser:

consistent

pero:

stale

respecto a una fuente más reciente.

Por ello EVOXA debe distinguir:

Correctness
Consistency
Freshness
Availability
26. Staleness

Una lectura debe poder clasificarse:

CURRENT
RECENT
STALE
UNKNOWN

cuando el consumidor necesite conocer la antigüedad del estado.

27. Read Consistency Metadata

Cuando sea necesario, las respuestas pueden incluir:

version
sequence
sourceTimestamp
projectionTimestamp
lag

Esto permite a consumidores tomar decisiones informadas.

28. Consistency Tokens

EVOXA puede utilizar:

version token
sequence token
watermark
LSN
cursor

para indicar hasta qué punto un consumidor ha observado el estado.

29. Read Barrier

Un cliente puede requerir:

"I need state >= version 42"

El sistema puede aplicar:

Read Barrier

antes de responder.

30. Write Barrier

Una nueva escritura puede requerir:

stateVersion == expectedVersion

para evitar modificaciones sobre estado obsoleto.

31. Optimistic Concurrency

Modelo:

Current Version = 10

Client A → write(version=10)
Client B → write(version=10)

Solo una escritura debe avanzar si el contrato exige:

compare-and-set

La otra debe recibir:

CONFLICT
32. Conflict

Un conflicto ocurre cuando:

State A
   +
State B
   ↓
Cannot both be accepted

Los conflictos deben detectarse explícitamente.

33. Conflict Types
Concurrent Update
Version Conflict
Duplicate Event
Ordering Conflict
Business Conflict
Source Conflict
Merge Conflict
External Conflict
34. Conflict Detection

Puede utilizar:

version
sequence
timestamp
hash
business key
causation

según el tipo de conflicto.

35. Conflict Resolution

Las estrategias incluyen:

Reject
Last-Write-Wins
First-Write-Wins
Merge
Priority
Domain Rule
Manual Review
Compensation

No debe utilizarse:

Last-Write-Wins

como default universal.

36. Domain Conflict Rules

Cada dominio crítico debe definir:

What constitutes conflict?
Who wins?
Can states be merged?
Is manual resolution required?
37. Event Ordering

En sistemas event-driven, EVOXA debe definir ordering por:

Aggregate
Entity
Partition
Workflow
Tenant
Global

Solo debe requerirse global ordering cuando sea realmente necesario.

38. Aggregate Ordering

Un aggregate puede utilizar:

sequenceNumber

Ejemplo:

Order-123
  Event 1
  Event 2
  Event 3

El consumer puede detectar:

received 1
received 3
missing 2
39. Eventual Consistency Pipeline
COMMAND
  ↓
AUTHORITATIVE WRITE
  ↓
EVENT
  ↓
BROKER
  ↓
CONSUMER
  ↓
PROJECTION
  ↓
READ

Cada etapa puede introducir:

latency
retry
duplication
reordering
failure

La arquitectura debe compensarlo explícitamente.

40. Outbox Consistency

Para evitar:

DB COMMIT ✓
EVENT PUBLISH ✗

EVOXA puede utilizar:

Transaction
 ├── Domain State
 └── Outbox Event

ambos dentro de la misma transacción local.

Después:

Outbox
 ↓
Publisher
 ↓
Broker
41. Inbox Consistency

Consumers críticos pueden utilizar:

Inbox

para registrar eventos procesados.

Esto permite detectar:

duplicate delivery

y mantener:

logical idempotency
42. Exactly-Once Semantics

Debe distinguirse:

Exactly Once Delivery

de:

Exactly Once Logical Effect

En sistemas distribuidos, EVOXA debe diseñar principalmente para:

at-least-once delivery
+
idempotent processing

cuando corresponda.

43. Idempotency

Una operación idempotente:

f(f(x)) = f(x)

desde el punto de vista de su efecto lógico.

Esto reduce el impacto de:

retries
duplicate messages
network failures
consumer restarts
44. Retry and Consistency

Un retry puede producir:

duplicate command
duplicate event
duplicate side effect

Por eso los retries deben combinarse con:

idempotency key
deduplication
transaction boundaries

cuando corresponda.

45. Saga Consistency

Una saga no proporciona atomicidad global.

Proporciona:

sequence of local transactions
+
compensation

Modelo:

Step A
 ↓
Step B
 ↓
Step C

Si C falla:

Compensate B
 ↓
Compensate A
46. Saga Consistency State

El workflow debe distinguir:

COMPLETED
IN_PROGRESS
FAILED
COMPENSATING
COMPENSATED
REQUIRES_MANUAL_INTERVENTION
47. Workflow Consistency

Los workflows largos deben persistir:

workflowId
state
version
step
correlation
lastSuccessfulStep

Esto permite recuperación y continuación sin perder coherencia.

48. Cache Consistency

La caché introduce:

Source
  ≠
Cache

temporalmente.

Por ello debe definirse:

TTL
Invalidation
Refresh
Versioning
Stale-read policy
49. Cache Invalidation

Estrategias:

Write-through
Write-behind
Cache-aside
Event invalidation
TTL
Versioned keys

La elección depende del dominio.

50. Search Consistency

Un índice de búsqueda puede quedar temporalmente desactualizado:

Database
  ↓
Event
  ↓
Indexer
  ↓
Search Index

El contrato debe definir:

acceptable search lag

y estrategia de reindexación.

51. Projection Consistency

Una proyección debe mantener:

projectionVersion
sourceVersion
lastProcessedSequence

cuando sea necesario.

52. Projection Gap

Si:

lastProcessed = 100
incoming = 103

y:

101, 102

no están disponibles, el projection engine debe:

wait
recover
replay
or quarantine

según el contrato.

53. Reconciliation

La reconciliación es el mecanismo que transforma:

Observed State

en:

Expected State

mediante:

compare
detect
explain
repair
verify
54. Reconciliation Types
Online
Periodic
Scheduled
Triggered
On-demand
Post-recovery
55. Online Reconciliation

Puede ejecutarse cerca de tiempo real:

Source update
 ↓
Derived state
 ↓
Immediate consistency check

Útil para dominios críticos.

56. Periodic Reconciliation

Ejemplo:

Every 15 minutes

comparar:

Source
vs
Projection
57. Deep Reconciliation

Para datasets críticos:

record count
checksums
business totals
relationships
versions

pueden compararse.

58. Consistency Checkpoint

Un checkpoint representa:

"All state up to X has been successfully processed."

Ejemplo:

aggregateVersion = 42
projectionVersion = 42
59. Watermarks

Un watermark indica:

latest known safe processing boundary

Ejemplo:

Event Stream
0 ─────────────── 100
                ↑
             watermark
60. Consistency Monitoring

Métricas esenciales:

consistencyLag
projectionLag
reconciliationFailures
conflictRate
staleReadRate
outOfOrderEvents
duplicateEvents
missingEvents
repairRate
61. Consistency SLOs

Para cada sistema eventual:

P50 convergence
P95 convergence
P99 convergence
maximum lag
unresolved divergence
62. Consistency Alerts

Alertar cuando:

lag > threshold
sequence gap detected
projection stops advancing
reconciliation fails
conflict rate spikes
stale data exceeds SLA
63. Consistency Incident

Proceso:

DETECT
  ↓
CLASSIFY
  ↓
CONTAIN
  ↓
IDENTIFY SOURCE OF TRUTH
  ↓
MEASURE DIVERGENCE
  ↓
REPAIR
  ↓
RECONCILE
  ↓
VERIFY
64. Consistency Failure Modes

EVOXA debe contemplar:

Lost Event
Duplicate Event
Out-of-Order Event
Stale Projection
Stale Cache
Partial Commit
Split Brain
Concurrent Update
Replication Lag
Network Partition
Consumer Failure
Producer Failure
External System Divergence
65. Network Partition

Ante una partición:

Node A
   X
Node B

EVOXA debe seguir una política explícita:

Reject Writes
Allow Local Writes
Degrade
Queue

No debe ocurrir accidentalmente.

66. Split Brain

Dos componentes no deben poder convertirse simultáneamente en:

authoritative writer

si el dominio requiere single-writer semantics.

Deben existir mecanismos de:

leader election
fencing
lease
epoch

cuando sean necesarios.

67. Fencing

Un writer antiguo debe ser incapaz de continuar escribiendo después de perder autoridad.

Modelo:

Epoch 10 → old writer
Epoch 11 → new writer

Las operaciones del epoch 10 deben rechazarse.

68. Multi-Tenant Consistency

La consistencia debe respetar el aislamiento:

Tenant A state
        ≠
Tenant B state

Las operaciones de:

replication
projection
cache
reconciliation
restore

deben preservar tenantId.

69. Cross-Tenant Consistency Failure

Un estado perteneciente a Tenant A nunca debe ser usado para satisfacer una lectura de Tenant B.

Esto incluye:

cache key
search index
projection
analytics
batch
reconciliation
70. External Consistency

Los sistemas externos pueden divergir:

EVOXA State
     ≠
External State

EVOXA debe definir:

EVOXA authoritative
External authoritative
Shared authority

por integración.

71. Integration Reconciliation
EVOXA
  │
  ├── State A
  │
  ▼
External
  │
  └── State B

Si:

A != B

debe existir una política explícita:

repair EVOXA
repair external
manual resolution
ignore
72. AI Consistency

Los sistemas AI pueden observar datos:

at different versions

Un agente puede estar razonando sobre:

State 41

mientras el sistema ya está en:

State 44

Por tanto, las acciones AI críticas deben validar el estado actual antes de ejecutar.

73. Agent Read-Then-Act

Patrón:

READ STATE
   ↓
REASON
   ↓
VALIDATE VERSION
   ↓
ACT

No:

READ
 ↓
LONG REASONING
 ↓
BLIND ACT
74. Stale Decision Protection

Una acción puede incluir:

expectedVersion

Ejemplo:

Approve Order
if version == 12

Si el estado actual es:

version == 14

la acción debe rechazarse o reevaluarse.

75. Consistency and Governance

Las políticas deben declarar:

which consistency model applies
who owns the state
acceptable lag
who resolves conflicts
76. Consistency Policy
consistencyPolicy
├── model
├── sourceOfTruth
├── freshness
├── ordering
├── concurrency
├── conflictResolution
├── reconciliation
├── repair
└── monitoring
77. Consistency Registry

EVOXA puede mantener un catálogo:

Domain
State
Source
Consistency Model
Maximum Lag
Owner
Repair Strategy

Esto evita que las garantías de consistencia existan solamente como conocimiento tribal.

78. Consistency Matrix

Ejemplo:

State	Source	Model	Max Lag	Repair
Order	Order DB	Strong	0	Transaction
Order View	Events	Eventual	5s	Replay
Search	Search Index	Eventual	30s	Reindex
Cache	DB	Eventual	60s	Invalidate
Analytics	Warehouse	Eventual	24h	Reload
Workflow	Workflow Store	Strong	0	Resume
79. Consistency Testing

Debe probarse:

concurrent writes
duplicate events
out-of-order events
lost messages
consumer restart
producer restart
network partition
cache failure
projection rebuild
database failover
restore
replay
80. Jepsen-Style Thinking

Sin asumir una herramienta concreta, EVOXA debe validar explícitamente:

What happens under concurrency?
What happens under partition?
What happens under retry?
What happens under failure?

La consistencia no debe considerarse demostrada únicamente porque el sistema funciona bajo condiciones normales.

81. Property-Based Consistency Testing

Se pueden probar propiedades como:

P1:
No aggregate version decreases.

P2:
No committed write disappears.

P3:
Derived state eventually converges.

P4:
Invalid conflicts are never silently accepted.
82. Replay Testing

Para sistemas event-driven:

Events
  ↓
Replay
  ↓
Expected State

debe producir un estado equivalente al authoritative state cuando el modelo sea reproducible.

83. Projection Rebuild

Una proyección debe poder:

DELETE
REPLAY
REBUILD
VERIFY
PUBLISH

sin modificar la fuente de verdad.

84. Consistency During Recovery

Después de recovery:

Restore
 ↓
Replay
 ↓
Reconcile
 ↓
Verify

No debe declararse el sistema consistente simplemente porque los servicios están "up".

85. Consistency During Deployment

Deployments pueden introducir:

old producer
new consumer

o:

new producer
old consumer

Por ello los contratos deben soportar:

version compatibility

durante las transiciones.

86. Rolling Deployment Consistency

Durante una transición:

v1
 +
v2

deben coexistir cuando sea necesario.

El cambio debe evitar:

producer v2
→
consumer v1
→
data corruption
87. Backward Compatibility

Los eventos y contratos críticos deben evolucionar de manera compatible cuando exista procesamiento asíncrono.

88. Consistency and Schema Evolution

Un schema nuevo no debe crear automáticamente:

inconsistent derived state

Los consumidores deben poder determinar:

schema version
projection compatibility
migration status
89. Consistency Dashboard

Debe poder visualizar:

Source
 ↓
Events
 ↓
Consumers
 ↓
Projection
 ↓
Search
 ↓
Cache

y para cada componente:

version
lag
health
last processed
errors
90. Consistency Observability

Logs deben incluir, cuando corresponda:

entityId
aggregateId
tenantId
version
sequence
eventId
causationId
correlationId
sourceTimestamp
processingTimestamp

Esto permite reconstruir divergencias.

91. Distributed Trace

Una operación distribuida puede seguir:

Command
 ↓
DB Commit
 ↓
Outbox
 ↓
Event
 ↓
Consumer
 ↓
Projection
 ↓
Read

La trazabilidad debe permitir localizar dónde apareció la divergencia.

92. Consistency Error Budget

Un sistema eventual puede definir un presupuesto:

Maximum acceptable stale-read duration
+
Maximum unresolved divergence

Si se excede:

SLO violation
93. Consistency Levels

Una API puede exponer explícitamente:

CONSISTENCY=STRONG
CONSISTENCY=SESSION
CONSISTENCY=EVENTUAL

solo si realmente soporta esas garantías.

94. Client-Specified Consistency

Cuando sea necesario:

GET /resource
Consistency: strong

o:

Consistency: version >= 42

La interfaz exacta dependerá de E03/API Architecture.

95. Default Consistency

Nunca debe dejarse ambiguo.

Cada API y servicio debe declarar:

default consistency

y cualquier excepción debe ser explícita.

96. Consistency and Availability

Existe un trade-off:

Consistency
      ↕
Availability

Durante determinadas fallas, EVOXA debe decidir si:

rejects stale operation

o:

serves stale data

La decisión pertenece al contrato del dominio.

97. Business-Critical Consistency

Los estados que afectan directamente:

money
authorization
ownership
inventory
legal state
security

deben recibir garantías superiores a:

analytics
search
recommendations
cache

cuando corresponda.

98. Consistency Classification

EVOXA puede clasificar estados:

C0 — Strong
C1 — Bounded Eventual
C2 — Eventual
C3 — Best Effort

Cada nivel debe tener un contrato medible.

99. Consistency Invariants
CI1 — Every authoritative state must have a defined consistency owner.

CI2 — Every distributed state must have an explicit consistency model.

CI3 — Eventual consistency must have a convergence expectation.

CI4 — Strong consistency must not be assumed outside its transaction boundary.

CI5 — Derived state must never silently become authoritative.

CI6 — Consumers must not regress to an older state when monotonic reads are required.

CI7 — Causal dependencies must preserve required causal ordering.

CI8 — Concurrent updates must either serialize, merge, or produce an explicit conflict.

CI9 — Conflicts must never be silently discarded when they can affect correctness.

CI10 — Every eventually consistent projection must have a recovery path.

CI11 — Reconciliation must be possible for critical distributed state.

CI12 — Consistency failures must be observable.

CI13 — Staleness must be measurable when freshness matters.

CI14 — Read models must expose or internally track the source version required for reconciliation.

CI15 — Cache state must never override authoritative state unless explicitly designed as part of the authority model.

CI16 — External-system divergence must have an explicit ownership rule.

CI17 — AI/Agent actions based on mutable state must protect against stale decisions.

CI18 — Recovery must restore consistency, not merely availability.

CI19 — Multi-tenant consistency boundaries must preserve tenant isolation.

CI20 — Consistency guarantees must be explicit rather than inferred.
100. Consistency Control Plane
                     CONSISTENCY CONTROL PLANE
                              │
          ┌───────────────────┼──────────────────┐
          ▼                   ▼                  ▼
     Policies             Registry           Monitoring
          │                   │                  │
          ▼                   ▼                  ▼
     Consistency          Source-of-Truth     Lag/SLO
       Models                Ownership        Detection
          │                   │                  │
          └───────────────────┼──────────────────┘
                              ▼
                       Reconciliation
                              │
                    ┌─────────┴─────────┐
                    ▼                   ▼
                  Repair              Conflict
                                      Resolution
101. Consistency Data Plane
Command
  ↓
Transaction
  ↓
Event
  ↓
Broker
  ↓
Consumer
  ↓
Projection
  ↓
Cache/Search

El data plane ejecuta las reglas definidas por el control plane.

102. Consistency Lifecycle
WRITE
  ↓
COMMIT
  ↓
PUBLISH
  ↓
PROPAGATE
  ↓
PROCESS
  ↓
PROJECT
  ↓
CONVERGE
  ↓
VERIFY

Si algo falla:

DETECT
  ↓
RETRY
  ↓
REPLAY
  ↓
RECONCILE
  ↓
REPAIR
103. Reference Architecture
                         ┌─────────────────────┐
                         │   AUTHORITATIVE     │
                         │       STATE         │
                         └──────────┬──────────┘
                                    │
                              TRANSACTION
                                    │
                                    ▼
                         ┌─────────────────────┐
                         │       OUTBOX        │
                         └──────────┬──────────┘
                                    │
                                    ▼
                         ┌─────────────────────┐
                         │    EVENT STREAM     │
                         └───────┬─────┬───────┘
                                 │     │
                    ┌────────────┘     └────────────┐
                    ▼                               ▼
           ┌────────────────┐              ┌────────────────┐
           │ READ MODEL     │              │ SEARCH INDEX   │
           └───────┬────────┘              └────────────────┘
                   │
                   ▼
             ┌───────────┐
             │   CACHE   │
             └───────────┘

                    ALL DERIVED STATES
                           │
                           ▼
                  ┌────────────────┐
                  │ RECONCILIATOR  │
                  └───────┬────────┘
                          │
                 ┌────────┴────────┐
                 ▼                 ▼
             CONSISTENT         DIVERGED
                                   │
                                   ▼
                                REPAIR
                                   │
                                   ▼
                                VERIFY
104. Engineering Completion Criteria

E44 queda completo cuando EVOXA posee:

✓ Consistency model
✓ Consistency domains
✓ Strong consistency definition
✓ Eventual consistency definition
✓ Causal consistency
✓ Session consistency
✓ Read-your-writes
✓ Monotonic reads
✓ Monotonic writes
✓ Transactional consistency
✓ Distributed consistency strategy
✓ Consistency ownership
✓ Source-of-truth model
✓ Consistency graph
✓ Consistency contracts
✓ Consistency windows
✓ Freshness model
✓ Staleness model
✓ Consistency tokens
✓ Read barriers
✓ Write barriers
✓ Optimistic concurrency
✓ Conflict detection
✓ Conflict resolution
✓ Event ordering
✓ Aggregate ordering
✓ Outbox strategy
✓ Inbox strategy
✓ Idempotency
✓ Saga consistency
✓ Workflow consistency
✓ Cache consistency
✓ Search consistency
✓ Projection consistency
✓ Projection gap handling
✓ Reconciliation
✓ Watermarks
✓ Checkpoints
✓ Consistency monitoring
✓ Consistency SLOs
✓ Consistency alerts
✓ Consistency incident process
✓ Failure modes
✓ Partition strategy
✓ Split-brain protection
✓ Fencing
✓ Multi-tenant consistency
✓ External-system reconciliation
✓ AI stale-decision protection
✓ Governance policies
✓ Consistency registry
✓ Testing strategy
✓ Replay testing
✓ Projection rebuild
✓ Recovery consistency
✓ Deployment consistency
✓ Schema evolution consistency
✓ Observability
✓ Error budgets
✓ Consistency classification
✓ Consistency invariants
✓ Control plane
✓ Data plane
✓ Lifecycle
✓ Reference architecture
105. Principio Rector de E44

La consistencia en EVOXA no significa que todos los componentes tengan el mismo estado simultáneamente; significa que cada frontera del sistema tiene una garantía explícita, medible y verificable sobre cómo sus estados pueden diferir, cuánto tiempo pueden diferir, qué relaciones deben preservarse y cómo el sistema detecta, reconcilia y corrige las divergencias.

La progresión queda entonces:

E42 — Backup & Restore
       │
       │ "Can we recover it?"
       ▼
E43 — Data Integrity
       │
       │ "Can we trust it?"
       ▼
E44 — Consistency
       │
       │ "Do distributed states agree
       │  according to their contract?"
       ▼
E45 — Data Lifecycle
       │
       │ "How does data live, evolve,
       │  age and eventually leave?"
       ▼
E46 ...

E44 queda así establecido como la capa que conecta la integridad individual de E43 con el comportamiento distribuido y evolutivo de los datos dentro de EVOXA.
