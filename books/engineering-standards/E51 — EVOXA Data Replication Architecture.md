E51 — EVOXA Data Replication Architecture
Engineering Specification

Pertenece a: Engineering Specification
Capítulo: E51 — Data Replication Architecture
Anterior: E50 — Data Synchronization Architecture
Siguiente: E52 — Data Federation Architecture

1. Propósito

E51 define cómo EVOXA mantiene copias redundantes y operacionalmente utilizables de datos en múltiples nodos, almacenes, regiones o sistemas, con garantías explícitas sobre:

disponibilidad,
durabilidad,
latencia de replicación,
consistencia,
recuperación,
failover,
failback,
integridad,
aislamiento,
observabilidad.

El principio fundamental es:

Replication mantiene múltiples copias de un mismo estado; Synchronization mantiene alineados estados que pueden pertenecer a sistemas o modelos distintos.

2. Replication vs Synchronization

Esta distinción es fundamental.

Synchronization
System A
   │
   ▼
Transform / Map
   │
   ▼
System B

Puede existir:

different models
different schemas
different ownership
different semantics
Replication
Primary
   │
   ├────────► Replica A
   ├────────► Replica B
   └────────► Replica C

La intención principal es mantener:

same logical dataset
multiple copies

Por tanto:

Replication es una estrategia de redundancia de datos; synchronization es una estrategia de alineamiento de estados.

3. Boundary

E51 cubre:

Replication Topologies
Primary / Replica Models
Single-Leader Replication
Multi-Leader Replication
Leaderless Replication
Synchronous Replication
Asynchronous Replication
Semi-Synchronous Replication
Log-Based Replication
Storage Replication
Database Replication
Cross-Region Replication
Cross-Zone Replication
Read Replicas
Failover
Failback
Replica Promotion
Replication Lag
Consistency
Quorum
Conflict Handling
Replication Integrity
Replica Health
Recovery
Rebuild
Reseed
Observability

No reemplaza:

E43 — Data Integrity Architecture
E44 — Data Consistency Architecture
E49 — Data Migration Architecture
E50 — Data Synchronization Architecture
E40 — Recovery Architecture
E41 — Disaster Recovery Architecture
E42 — Backup & Restore Architecture
4. Replication Objectives

EVOXA debe poder utilizar replication para:

high availability
read scaling
disaster recovery
regional resilience
fault isolation
latency reduction
durability
operational continuity
5. Replication Model

Conceptualmente:

                 ┌──────────────┐
                 │   PRIMARY    │
                 └──────┬───────┘
                        │
             Replication Stream
                        │
          ┌─────────────┼─────────────┐
          ▼             ▼             ▼
     ┌─────────┐   ┌─────────┐   ┌─────────┐
     │Replica A│   │Replica B│   │Replica C│
     └─────────┘   └─────────┘   └─────────┘
6. Replica

Una réplica es:

Una representación persistente de un dataset mantenida a partir de otra representación considerada fuente de replicación.

Una réplica puede ser:

read-only
read-write
hot
warm
cold
synchronous
asynchronous
regional
cross-regional
7. Replica Roles

Estados típicos:

PRIMARY
SECONDARY
REPLICA
CANDIDATE
PROMOTING
PROMOTED
DEMOTING
FAILED
RECOVERING
REBUILDING
8. Primary

El primary es el nodo que, bajo un modelo single-leader:

accepts authoritative writes

Ejemplo:

Client
  │
  ▼
Primary
  │
  ├──► Replica A
  ├──► Replica B
  └──► Replica C
9. Secondary / Replica

Una replica normalmente:

receives replicated changes
applies changes
serves reads

dependiendo de la política.

10. Read Replica

Una read replica puede utilizarse para:

read scaling
reporting
analytics
regional reads
query isolation

Ejemplo:

                 Primary
                /       \
               ▼         ▼
          Read Replica  Read Replica
11. Hot Replica

Una hot replica:

continuously receives changes
is operationally ready

Puede convertirse rápidamente en primary.

12. Warm Replica

Una warm replica:

receives data
but may require additional preparation

antes de asumir tráfico completo.

13. Cold Replica

Una cold replica puede mantenerse:

offline
infrequently updated

y no es apropiada para failover rápido.

14. Replication Topologies

EVOXA debe soportar conceptualmente:

Single-Leader
Multi-Leader
Leaderless
Chain
Tree
Star
Hub-and-Spoke
Cross-Region

La implementación concreta depende del storage engine.

15. Single-Leader Replication
                 ┌──► Replica A
                 │
Primary ─────────┼──► Replica B
                 │
                 └──► Replica C

Ventajas:

simple ownership
simple ordering
simple conflict model

Es el modelo preferido cuando no se necesita multi-master.

16. Multi-Leader Replication
Region A ◄────────► Region B
   │                    │
   ▼                    ▼
Replica                Replica

Permite:

local writes
regional autonomy

pero introduce:

conflicts
causal ordering
write convergence
17. Leaderless Replication
        ┌── Node A
Write ──┼── Node B
        └── Node C

La escritura puede dirigirse a múltiples réplicas sin un primary único.

Requiere una política explícita de:

quorum
read repair
conflict resolution
18. Chain Replication
Primary
   ↓
Replica A
   ↓
Replica B
   ↓
Replica C

Puede reducir ciertas cargas de coordinación, pero aumenta la dependencia de la cadena.

19. Tree Replication
               Primary
              /       \
             A         B
           /  \       / \
          C    D     E   F

Puede utilizarse para:

global distribution
regional fan-out
hierarchical replication
20. Synchronous Replication

En términos conceptuales:

Write
  │
  ├──► Primary
  │
  └──► Replica
          │
          ▼
       ACK
          │
          ▼
       Client

La operación sólo se confirma cuando se cumple el requisito de durabilidad definido.

Ventajas:

stronger durability
lower RPO

Costes:

higher latency
network dependency
21. Asynchronous Replication
Write
  │
  ▼
Primary
  │
  └────► Replication Stream ────► Replica

El primary puede confirmar antes de que la replica haya aplicado el cambio.

Ventajas:

lower write latency
better geographic scalability

Coste:

replication lag
possible data loss on primary failure
22. Semi-Synchronous Replication

Combina ambos modelos:

Primary
   │
   ├──► synchronous acknowledgement
   │
   └──► asynchronous replicas

Permite controlar:

durability
latency

según la política.

23. Replication Contract

Cada grupo de replicación debe declarar:

ReplicationDefinition
├── replicationId
├── source
├── replicas
├── topology
├── consistency
├── durability
├── lagPolicy
├── failoverPolicy
├── recoveryPolicy
└── ownership
24. Replication Scope

Debe especificarse qué se replica:

database
schema
table
collection
partition
tenant
entity
dataset
storage volume
25. Full vs Partial Replication
Full
Dataset
  │
  └──► Complete Replica
Partial
Dataset
  │
  ├──► Region A subset
  └──► Region B subset

La replicación parcial requiere reglas claras de ownership.

26. Tenant-Aware Replication

Puede replicarse:

Tenant A → Region 1
Tenant B → Region 2
Tenant C → Region 3

o:

All Tenants
     ↓
Regional Replica

La elección debe respetar:

isolation
data residency
capacity
availability
27. Replication Stream

El mecanismo lógico es:

Source State
     │
     ▼
Change Log
     │
     ▼
Replication Stream
     │
     ▼
Replica
28. Log-Based Replication

Modelo:

Database
   │
   ▼
Transaction Log
   │
   ▼
Replication Consumer
   │
   ▼
Replica

El log permite:

ordering
replay
checkpointing
recovery
29. Replication Position

Cada replica debe poder representar su posición:

replicationPosition

Ejemplos conceptuales:

offset
LSN
sequence
log position
commit index
30. Replication Checkpoint
Primary Log
     │
     ├── 100
     ├── 101
     ├── 102
     └── 103
              ▲
              │
        Replica checkpoint

Permite reanudar desde un punto conocido.

31. Replica Lag

La métrica central:

replicationLag =
sourcePosition - replicaPosition

También puede medirse temporalmente:

latestPrimaryCommit
-
latestReplicaApply
32. Lag States
HEALTHY
WARNING
CRITICAL
STALE
BROKEN

según políticas de cada workload.

33. Replication SLA

Una definición puede incluir:

Maximum Lag
Maximum RPO
Maximum Failover Time
Minimum Replica Count
Required Regions
34. RPO

Recovery Point Objective determina:

Cuánto dato puede perderse si la fuente primaria falla.

Ejemplo conceptual:

RPO = 0

requiere garantías de durabilidad mucho más fuertes que:

RPO = 5 minutes
35. RTO

Recovery Time Objective determina:

Cuánto tiempo puede tardar EVOXA en recuperar capacidad operacional después del fallo.

Replication puede reducir RTO porque una réplica ya está disponible.

36. Replication vs Backup

No son equivalentes.

Replication
Primary ───► Replica

mantiene copias operativas.

Mientras:

Primary ───► Backup

proporciona un punto recuperable.

Una corrupción lógica puede replicarse:

Bad Data
   │
   ├──► Replica A
   ├──► Replica B
   └──► Replica C

Por eso:

Replication no sustituye Backup.

37. Replication vs Disaster Recovery

Replication puede ser un mecanismo de DR:

Region A
   │
   ▼
Region B

pero DR incluye además:

failover
restore
validation
operations
DNS / routing
application recovery
38. Replica Health

Cada replica debe exponer:

health
replication position
lag
apply rate
error rate
storage capacity
connectivity
39. Replica State Machine
PROVISIONING
     ↓
CATCHING_UP
     ↓
IN_SYNC
     ↓
SERVING

Failure:

IN_SYNC
   ↓
DEGRADED
   ↓
RECOVERING
   ↓
IN_SYNC
40. Catch-Up

Una replica nueva puede realizar:

Snapshot
   ↓
Install Snapshot
   ↓
Replay Log
   ↓
Catch Up
   ↓
Serving
41. Replica Bootstrap

Proceso:

Source Snapshot
       ↓
Replica Initialization
       ↓
Log Position
       ↓
Incremental Replay
       ↓
Validation
       ↓
Activation
42. Replica Rebuild

Si una replica está corrupta:

Replica
  ↓
Invalidate
  ↓
Rebuild
  ↓
Catch Up
  ↓
Validate
  ↓
Return to Pool
43. Reseeding

Puede utilizarse:

Healthy Replica
      ↓
Snapshot
      ↓
Failed Replica

para evitar cargar excesivamente al primary.

44. Failover

Ante pérdida del primary:

Primary
   X
   │
   ▼
Replica A
   │
   ▼
PROMOTION
   │
   ▼
New Primary
45. Failover Preconditions

Antes de promover una replica debe comprobarse:

replication health
lag
data integrity
role eligibility
connectivity
fencing
46. Split-Brain Prevention

Riesgo:

Node A thinks it is Primary
Node B thinks it is Primary

Ambos aceptan writes.

Esto puede producir:

conflicts
data divergence
corruption
47. Fencing

EVOXA debe disponer de un mecanismo para impedir que el antiguo primary continúe escribiendo después de la promoción de otro nodo.

Conceptualmente:

Old Primary
     │
     ▼
FENCED
     X
     │
New Primary
     │
     ▼
Writes
48. Leader Election

En modelos dinámicos puede existir:

Candidate
   ↓
Election
   ↓
Leader

La elección debe evitar:

multiple leaders
49. Quorum

Una decisión puede requerir:

N = total replicas
W = write acknowledgements
R = read acknowledgements

El sistema debe documentar sus propias garantías; no debe asumirse que cualquier combinación produce strong consistency.

50. Read Quorum

Una lectura puede consultar múltiples réplicas:

Read
 │
 ├──► A
 ├──► B
 └──► C

y combinar resultados según la política de consistencia.

51. Write Quorum

Una escritura puede requerir:

Write
 │
 ├──► A
 ├──► B
 └──► C

con un mínimo de acknowledgements.

52. Strong Consistency

Objetivo:

Successful Write
      ↓
Subsequent Read
      ↓
Latest Value

Normalmente requiere mayor coordinación.

53. Eventual Consistency

Puede existir:

Write
  ↓
Primary updated
  ↓
Replica lag
  ↓
Replica catches up

Durante el intervalo puede observarse un valor anterior.

54. Bounded Staleness

La replica puede garantizar:

staleness ≤ configured threshold

por ejemplo:

seconds

según el sistema.

55. Read Routing

EVOXA debe decidir:

Read
 │
 ├──► Primary
 │
 └──► Replica

según:

consistency requirement
latency
region
freshness
56. Read-After-Write

Después de:

Write → Primary

un request posterior puede necesitar:

Read → Primary

hasta que la replica alcance la posición requerida.

57. Session Consistency

Una sesión puede conservar:

lastObservedVersion

y evitar lecturas de una replica que todavía no haya alcanzado esa versión.

58. Causal Consistency

Cuando los cambios tienen relaciones causales:

A
 ↓
B
 ↓
C

una replica no debería observar:

C

sin poder observar las dependencias necesarias, cuando la garantía lo requiera.

59. Conflict Detection

Especialmente importante en:

multi-leader
active-active
leaderless

Los conflictos pueden ser:

update/update
delete/update
insert/insert
schema conflict
identity conflict
60. Conflict Resolution

EVOXA puede definir:

SOURCE_WINS
REGION_WINS
LATEST_VERSION
LATEST_TIMESTAMP
MERGE
DOMAIN_RULE
MANUAL

La política debe pertenecer al dominio de datos correspondiente.

61. Tombstones

Cuando se elimina un registro:

Delete

no siempre puede simplemente desaparecer del stream.

Puede necesitarse:

Tombstone

para informar a las réplicas:

Entity X = deleted
62. Tombstone Retention

Los tombstones deben conservarse el tiempo suficiente para que las réplicas atrasadas puedan procesarlos.

De lo contrario:

Replica stale
   ↓
Misses Delete
   ↓
Resurrects Data
63. Data Resurrection

Es un riesgo importante en sistemas replicados:

Delete @ Primary
      ↓
Replica offline
      ↓
Replica returns
      ↓
Old Data replayed

La solución requiere:

versions
tombstones
generation metadata

según el modelo.

64. Identity

Todas las réplicas deben preservar una identidad estable:

entityId

La generación de IDs debe evitar colisiones entre escritores cuando exista multi-leader.

65. Global Identity

En multi-region puede utilizarse una estrategia de identidad que garantice:

globally unique ID

para evitar colisiones entre regiones.

66. Schema Replication

Además de datos, ciertos sistemas necesitan replicar:

schema
indexes
constraints
metadata

Debe distinguirse:

data replication

de:

schema replication
67. Schema Compatibility

Durante un rolling upgrade:

Replica A → Schema V1
Replica B → Schema V2

deben coexistir temporalmente si la plataforma lo requiere.

68. Replication and Transactions

Una transacción puede modificar:

A
B
C

La replicación debe preservar la semántica transaccional requerida.

No siempre es suficiente replicar:

A
B
C

como cambios independientes.

69. Transaction Boundary

Cuando sea necesario:

Transaction
 ├── Change A
 ├── Change B
 └── Change C
        │
        ▼
Replica applies transaction

para evitar estados intermedios inválidos.

70. Partial Apply

Un fallo puede ocurrir después de aplicar:

A
B

pero antes de:

C

El sistema debe disponer de:

transaction rollback
resume
replay
idempotency

según las capacidades del storage.

71. Replication Integrity

Debe verificarse:

record count
checksums
versions
sequence continuity
transaction completeness

cuando corresponda.

72. Gap Detection

Si el primary produce:

100
101
102
103

y la replica recibe:

100
101
103

debe detectar:

missing 102

en lugar de asumir que está sincronizada.

73. Duplicate Detection

Si recibe:

100
101
101
102

debe reconocer el duplicado.

74. Out-of-Order Detection

Si recibe:

100
102
101

debe aplicar una política explícita.

No debe asumir que el transporte preserva siempre el orden.

75. Replication Backpressure

Si la replica no puede aplicar cambios:

Primary
  │
  ▼
Replication Queue
  │
  ▼
Replica

la cola puede crecer.

Debe controlarse:

queue size
storage
consumer rate
network bandwidth
76. Flow Control

El sistema puede:

pause producer
slow producer
batch
compress
scale consumer

según capacidades.

77. Compression

Para replicación cross-region puede utilizarse:

compression
batching
deduplication

para reducir:

bandwidth
cost
latency
78. Network Partitions

Durante:

Region A X Region B

el sistema debe seguir una política conocida:

stop writes
continue local writes
queue writes
failover

No debe emerger accidentalmente.

79. CAP Trade-off

En presencia de una partición de red, EVOXA debe conocer qué prioriza el sistema:

Consistency
Availability
Partition Tolerance

La decisión debe estar definida por workload.

80. Regional Failover
Region A
   X
   │
   ▼
Region B
   │
   ▼
Promote Replica
   │
   ▼
Route Traffic

Debe existir coordinación con:

application routing
service discovery
DNS / traffic management
81. Failback

Cuando la región primaria original vuelve:

New Primary
    │
    ▼
Original Region
    │
    ▼
Rebuild / Catch-up
    │
    ▼
Eligible Replica

No debe reasumir automáticamente el rol primary sin validación.

82. Promotion Safety

Antes de promover:

1. Fence old primary
2. Validate replica
3. Determine latest safe position
4. Promote
5. Route writes
6. Monitor
83. Failover Data Loss

Si la replicación era asíncrona:

Primary committed:
A B C D

Replica received:
A B C

tras el failover puede perderse:

D

si no existe otra copia.

Esto define el RPO real.

84. Replication Monitoring

Métricas:

replicationLag
applyRate
sendRate
queueDepth
replicationErrors
reconnects
bytesReplicated
transactionsReplicated
conflicts
gaps
duplicateChanges
85. Replica Health Dashboard

Debe poder visualizarse:

                 Replication Health

Primary
  │
  ├── Replica A   HEALTHY   Lag: low
  ├── Replica B   WARNING   Lag: medium
  └── Replica C   CRITICAL  Lag: high
86. Alerts

Alertas mínimas:

replica lag high
replica disconnected
replication stopped
replication gap detected
storage near capacity
conflict rate high
checksum mismatch
replica unhealthy
87. Auditability

Las operaciones administrativas deben auditar:

promote replica
demote primary
pause replication
resume replication
rebuild replica
change topology
change policy
force failover
88. Security

Replication channels deben utilizar:

authentication
authorization
encryption in transit
credential isolation
least privilege
89. Data Residency

Cross-region replication debe considerar:

data residency
jurisdiction
tenant policy
regulatory constraints

No todo dataset debe poder replicarse libremente entre regiones.

90. Encryption

La réplica debe mantener las garantías de:

encryption at rest
encryption in transit
key management

según la clasificación del dato.

91. Key Management

Si una réplica utiliza claves distintas:

Region A → Key A
Region B → Key B

debe existir un mecanismo controlado de:

key rotation
key availability
recovery
92. Replication Priority

No todas las tablas/datasets requieren igual prioridad.

Puede definirse:

CRITICAL
HIGH
NORMAL
LOW

para asignar:

bandwidth
workers
replication frequency
93. Capacity Planning

Debe estimarse:

write throughput
replication throughput
peak burst
storage growth
network bandwidth
replica count
rebuild bandwidth
94. N+1 Replication

Para sistemas críticos:

Primary
 ├── Replica A
 ├── Replica B
 └── Replica C

puede evitar depender de una única réplica.

95. Replica Diversity

Cuando sea necesario, las réplicas pueden distribuirse:

Zone A
Zone B
Zone C

para evitar que un único failure domain destruya todas las copias.

96. Failure Domain

Las réplicas deben evitar compartir innecesariamente:

same host
same rack
same zone
same power domain
same region

si el objetivo es resiliencia frente a ese fallo.

97. Replication Placement

Una política puede exigir:

Replica 1 → Zone A
Replica 2 → Zone B
Replica 3 → Zone C

y para DR:

Replica 4 → Region B
98. Rebuild Storm

Después de un fallo:

Replica A lost
Replica B rebuilding
Replica C rebuilding

puede generarse carga excesiva.

El rebuild debe estar:

rate-limited
prioritized
observable
99. Repair vs Rebuild
Repair
small divergence
Rebuild
large divergence
corruption
untrusted replica

La selección debe ser explícita.

100. Replication Validation

Una replica recién creada debe pasar:

schema validation
data validation
position validation
checksum validation
application-read validation

antes de entrar en producción.

101. Replica Promotion Eligibility

Una réplica sólo puede ser promovida si:

healthy
authorized
sufficiently caught up
validated
fenced-safe

según la política.

102. Replication Generation

Cada primary epoch puede tener:

generationId

Esto ayuda a distinguir:

old primary writes

de:

new primary writes

y prevenir escrituras obsoletas.

103. Epoch

Conceptualmente:

Epoch 1
Primary A

Failover

Epoch 2
Primary B

Los cambios deben estar asociados al epoch cuando el sistema lo necesite.

104. Old Primary Protection

Después de failover:

Old Primary
    │
    └──► Reject Writes

hasta que sea:

reconfigured
reseeded
demoted
105. Replication State Store

El sistema debe conservar de forma durable:

replicationId
role
position
epoch
lag
health
lastError
checkpoint

cuando corresponda.

106. Reference Architecture
                         ┌───────────────────────┐
                         │ REPLICATION CONTROL   │
                         │        PLANE          │
                         ├───────────────────────┤
                         │ Topology              │
                         │ Policies              │
                         │ Roles                 │
                         │ Failover              │
                         │ Placement             │
                         │ Health                │
                         └───────────┬───────────┘
                                     │
                                     ▼

                         ┌───────────────────────┐
                         │       PRIMARY         │
                         │                       │
                         │  Source of Writes     │
                         └───────────┬───────────┘
                                     │
                              Change / Log
                                     │
                  ┌──────────────────┼──────────────────┐
                  │                  │                  │
                  ▼                  ▼                  ▼
           ┌────────────┐     ┌────────────┐     ┌────────────┐
           │ Replica A  │     │ Replica B  │     │ Replica C  │
           │   Zone A   │     │   Zone B   │     │   Region B │
           └─────┬──────┘     └─────┬──────┘     └─────┬──────┘
                 │                  │                  │
                 └──────────────────┼──────────────────┘
                                    ▼
                         ┌───────────────────────┐
                         │ REPLICATION HEALTH    │
                         │ LAG / INTEGRITY /     │
                         │ RECOVERY / FAILOVER   │
                         └───────────────────────┘
107. Replication Domain Model
Replication
├── replicationId
├── source
├── replicas
├── topology
├── consistencyPolicy
├── durabilityPolicy
├── placementPolicy
├── failoverPolicy
├── status
└── health
108. Replica Model
Replica
├── replicaId
├── replicationId
├── node
├── region
├── zone
├── role
├── generation
├── position
├── lag
├── status
└── lastError
109. Replication Position Model
ReplicationPosition
├── streamId
├── epoch
├── sequence
├── offset
├── timestamp
└── checkpoint

No todos los campos necesitan existir en cada implementation.

110. Failover Model
FailoverOperation
├── failoverId
├── replicationId
├── failedPrimary
├── candidateReplica
├── oldGeneration
├── newGeneration
├── initiatedAt
├── completedAt
├── reason
├── status
└── operator
111. Replication Events
ReplicationCreated
ReplicaProvisioned
ReplicaCatchingUp
ReplicaReady
ReplicationStarted
ReplicationLagDetected
ReplicationGapDetected
ReplicationPaused
ReplicationResumed
ReplicationFailed
ReplicaPromoted
PrimaryDemoted
FailoverStarted
FailoverCompleted
FailbackStarted
FailbackCompleted
ReplicaRebuilt
ReplicaValidated
ReplicationRecovered
112. Operational Flow
                     WRITE
                       │
                       ▼
                  ┌─────────┐
                  │ PRIMARY │
                  └────┬────┘
                       │
                  Commit Log
                       │
                       ▼
                Replication Stream
                       │
            ┌──────────┼──────────┐
            ▼          ▼          ▼
         Replica A  Replica B  Replica C
            │          │          │
            ▼          ▼          ▼
         Apply       Apply      Apply
            │          │          │
            └──────────┼──────────┘
                       ▼
                 Health Check
                       │
                 ┌─────┴─────┐
                 ▼           ▼
              Healthy      Lagging
                             │
                             ▼
                          Recovery
113. Relationship with E50

La relación debe mantenerse explícita:

E50 — DATA SYNCHRONIZATION
        │
        │ aligns data states
        ▼
E51 — DATA REPLICATION
        │
        │ maintains redundant copies
        ▼
Multiple Operational Copies

Ejemplo:

CRM ──────► EVOXA
      E50 Synchronization

mientras:

EVOXA Primary ──────► EVOXA Replica
                  E51 Replication
114. Relationship with E49
E49
Migration
   │
   ▼
Initial Dataset
   │
   ├──► E50 Synchronization
   │
   └──► E51 Replication

Migration puede crear el estado inicial de una replica, pero no sustituye el mecanismo continuo de replication.

115. Relationship with E40/E41/E42
E51 Replication
      │
      ├──► Availability
      │
      └──► DR Support
              │
              ├── E40 Recovery
              ├── E41 Disaster Recovery
              └── E42 Backup & Restore

Cada uno resuelve una dimensión diferente.

116. Core Invariants
Invariant 1 — Replica Identity

Cada réplica debe tener una identidad única y estable.

Invariant 2 — Position

Una réplica debe conocer de forma durable hasta qué posición de replicación ha aplicado datos.

Invariant 3 — No Silent Gaps

Los gaps de replicación deben detectarse explícitamente.

Invariant 4 — No Split-Brain

Nunca deben existir simultáneamente dos writers autorizados para el mismo dataset cuando el modelo exige single-leader.

Invariant 5 — Promotion Safety

Una réplica no debe promocionarse sin cumplir las condiciones de elegibilidad.

Invariant 6 — Integrity

Una réplica no debe declararse healthy únicamente porque esté conectada; debe cumplir las verificaciones de integridad requeridas.

Invariant 7 — Bounded Lag

Las réplicas sujetas a SLA deben mantener su lag dentro del límite definido.

Invariant 8 — Recoverability

Una réplica fallida debe poder reconstruirse o recuperarse sin intervención destructiva sobre la fuente sana.

Invariant 9 — Failure-Domain Separation

Las réplicas destinadas a resiliencia deben distribuirse entre failure domains adecuados.

Invariant 10 — Explicit Consistency

La consistencia de lectura y escritura debe ser una propiedad declarada, no una suposición.

117. Completion Criteria

E51 se considera arquitectónicamente completo cuando EVOXA dispone de:

✓ Replication boundary
✓ Primary/replica model
✓ Single-leader replication
✓ Multi-leader replication
✓ Leaderless model
✓ Synchronous replication
✓ Asynchronous replication
✓ Semi-synchronous model
✓ Replication topology
✓ Replication stream
✓ Log-based replication
✓ Checkpoints
✓ Positions
✓ Lag management
✓ Read replicas
✓ Quorum model
✓ Consistency model
✓ Conflict model
✓ Tombstones
✓ Gap detection
✓ Duplicate detection
✓ Out-of-order handling
✓ Failover
✓ Failback
✓ Replica promotion
✓ Fencing
✓ Split-brain prevention
✓ Replica rebuild
✓ Reseeding
✓ Integrity validation
✓ Cross-zone replication
✓ Cross-region replication
✓ Tenant-aware replication
✓ Data residency controls
✓ Security
✓ Observability
✓ Capacity management
✓ Failure-domain placement
✓ RPO/RTO integration
✓ Disaster recovery integration
✓ Backup distinction
✓ Operational audit
✓ Testing strategy
✓ Chaos testing
✓ Reference architecture
✓ Architectural invariants
118. Principio Rector de E51

Data Replication es la capacidad de mantener múltiples copias operacionalmente válidas de un mismo estado de datos, coordinando su propagación, consistencia, integridad, posición, disponibilidad, recuperación y promoción bajo políticas explícitas de durabilidad y fallo.

La secuencia arquitectónica queda:

E49 — DATA MIGRATION
       │
       │ establishes initial state
       ▼
E50 — DATA SYNCHRONIZATION
       │
       │ aligns independent states
       ▼
E51 — DATA REPLICATION
       │
       │ maintains redundant copies
       ▼
E52 — DATA FEDERATION
       │
       │ provides unified access
       ▼
Distributed Data Access

E50 mantiene estados alineados.
E51 mantiene copias redundantes.
E52 permitirá consultar y componer datos distribuidos sin exigir necesariamente su consolidación física.
