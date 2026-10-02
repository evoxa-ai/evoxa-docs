E49 — EVOXA Data Migration Architecture
Engineering Specification

Pertenece a: Engineering Specification
Capítulo: E49 — Data Migration Architecture
Anterior: E48 — Data Archival Architecture
Siguiente: E50 — EVOXA Data Synchronization Architecture

1. Propósito

E49 define cómo EVOXA mueve datos de manera controlada entre estructuras, sistemas, versiones, almacenes, regiones o modelos de datos, preservando integridad, semántica, seguridad, trazabilidad y continuidad operacional.

El principio fundamental es:

Data Migration no es simplemente copiar datos. Es transformar un conjunto de datos desde un estado origen hacia un estado destino verificando que el significado y las invariantes del sistema permanezcan correctos.

Conceptualmente:

SOURCE
  │
  ▼
DISCOVERY
  │
  ▼
MAPPING
  │
  ▼
TRANSFORMATION
  │
  ▼
VALIDATION
  │
  ▼
TRANSFER
  │
  ▼
VERIFICATION
  │
  ▼
CUTOVER
  │
  ▼
TARGET
2. Boundary

E49 cubre:

Migration Planning
Migration Discovery
Source Assessment
Target Assessment
Schema Migration
Data Transformation
Data Mapping
Data Validation
Data Transfer
Bulk Migration
Incremental Migration
Online Migration
Offline Migration
Dual Write Migration
Change Capture
Backfill
Cutover
Rollback
Reconciliation
Migration Checkpointing
Migration Resume
Migration Audit
Migration Observability
Migration Security
Migration Governance
Migration Testing
Migration Completion

No sustituye:

E23 — Serialization Architecture
E24 — Transformation Architecture
E25 — Mapping Architecture
E26 — Projection Architecture
E27 — Query Architecture
E43 — Data Integrity Architecture
E44 — Data Consistency Architecture
E48 — Data Archival Architecture
3. Migration vs Archival

E48:

Operational Data
      ↓
Long-Term Storage

E49:

Source Data
      ↓
Target Data

Por tanto:

ARCHIVAL
Source → Long-Term Preservation

MIGRATION
Source → New Operational/Logical Representation
4. Migration Objectives

Una migración puede perseguir:

schema upgrade
database replacement
platform replacement
service decomposition
tenant migration
region migration
storage migration
domain migration
application modernization
data model redesign
vendor replacement
system consolidation
system split
5. Migration Classes

EVOXA debe distinguir:

SCHEMA MIGRATION
DATA STORE MIGRATION
APPLICATION MIGRATION
SERVICE MIGRATION
TENANT MIGRATION
REGION MIGRATION
PLATFORM MIGRATION
DOMAIN MIGRATION
6. Migration Topology

Modelo general:

┌──────────────┐
│ SOURCE       │
│ SYSTEM       │
└──────┬───────┘
       │
       ▼
┌──────────────┐
│ MIGRATION    │
│ PIPELINE     │
└──────┬───────┘
       │
       ▼
┌──────────────┐
│ TARGET       │
│ SYSTEM       │
└──────────────┘
7. Migration Control Plane

El control plane administra:

Migration Definition
Migration Plan
Migration State
Migration Policy
Migration Checkpoints
Migration Progress
Migration Validation
Migration Cutover
Migration Rollback
Migration Audit
8. Migration Data Plane

El data plane ejecuta:

Extract
Transform
Load
Validate
Reconcile

Conceptualmente:

Source
  │
  ▼
Extractor
  │
  ▼
Transformation
  │
  ▼
Validator
  │
  ▼
Loader
  │
  ▼
Target
9. Migration Definition

Cada migración debe poseer una definición formal:

MigrationDefinition
├── migrationId
├── source
├── target
├── scope
├── strategy
├── mappingVersion
├── transformationVersion
├── validationPolicy
├── cutoverStrategy
├── rollbackStrategy
└── owner
10. Migration Scope

El scope debe ser explícito:

tenant
dataset
entity
dateRange
partition
region
service
schema

Ejemplo:

Tenant A
  └── Transactions
       └── 2022–2024
11. Migration Inventory

Antes de migrar:

Source Inventory
      ↓
Data Classification
      ↓
Dependency Analysis
      ↓
Migration Scope

Debe conocerse:

record count
data volume
schema
dependencies
relationships
indexes
constraints
events
external references
12. Source Assessment

El origen debe evaluarse respecto a:

data quality
schema stability
volume
growth rate
consistency
availability
access constraints
dependencies
13. Target Assessment

El destino debe evaluarse respecto a:

capacity
schema compatibility
constraints
performance
availability
security
tenant model
indexes
partitioning
14. Migration Readiness

Una migración no debe comenzar simplemente porque existe un script.

Debe existir:

Source Ready
Target Ready
Mapping Ready
Transformation Ready
Validation Ready
Rollback Ready
Observability Ready
15. Migration State Machine
PLANNED
   ↓
DISCOVERING
   ↓
READY
   ↓
MIGRATING
   ↓
VALIDATING
   ↓
CUTOVER_READY
   ↓
CUTOVER
   ↓
COMPLETED

Estados alternativos:

BLOCKED
FAILED
PAUSED
ROLLED_BACK
CANCELLED
16. Migration Strategies

EVOXA debe soportar conceptualmente:

BIG_BANG
BATCH
INCREMENTAL
ONLINE
OFFLINE
DUAL_WRITE
CHANGE_CAPTURE
BLUE_GREEN
SHADOW
PHASED

La estrategia se selecciona según:

downtime tolerance
data volume
change rate
rollback requirements
system criticality
17. Big Bang Migration
Source
  │
  ▼
Migration Window
  │
  ▼
Full Migration
  │
  ▼
Cutover

Ventaja:

simple state transition

Desventaja:

large downtime
large rollback surface
18. Batch Migration
Dataset
  ↓
Batch 1
Batch 2
Batch 3
...
Batch N

Permite controlar:

memory
IO
network
transaction size
retry scope
19. Incremental Migration
Initial Load
     ↓
Delta 1
     ↓
Delta 2
     ↓
Delta 3
     ↓
Cutover

Reduce el tiempo de downtime.

20. Online Migration

Durante la migración:

Users
  │
  ├────► Source
  │
  └────► Migration Process

El sistema permanece operativo.

Requiere controlar:

concurrent writes
change capture
consistency
cutover
21. Offline Migration
Stop Writes
    ↓
Extract
    ↓
Transform
    ↓
Load
    ↓
Validate
    ↓
Start Target

Es más simple pero puede requerir downtime.

22. Dual Write

Durante una transición:

Application
    │
    ├────► Source
    │
    └────► Target

El objetivo es mantener ambos sistemas actualizados.

Debe controlarse:

write ordering
failure handling
idempotency
reconciliation
23. Change Capture

Para migraciones online:

Source
  │
  ├── Initial Snapshot
  │
  └── Change Stream
          │
          ▼
       Target

Los cambios posteriores al snapshot deben aplicarse al target.

24. Migration Watermark

El sistema debe conocer hasta dónde se ha migrado:

watermark

Ejemplo:

Snapshot → T0
Changes → T0...T1000
Current Source → T1200

Estado:

Migrated Through T1000
25. Migration Lag

Debe medirse:

Source Current Position
        -
Migration Position

Ejemplo:

Current = 1200
Migrated = 1180

Lag = 20
26. Snapshot Migration

La migración inicial puede comenzar con:

consistent snapshot

para obtener un punto de referencia estable.

27. Snapshot Consistency

El snapshot debe representar una vista coherente según las garantías del origen.

No debe combinar arbitrariamente:

Entity A @ T1
Entity B @ T2
Entity C @ T3

si eso viola las invariantes del dominio.

28. Extract

El extractor debe:

read source
respect source limits
capture metadata
capture ordering
capture checkpoints
29. Extract Metadata

Debe conservarse:

sourceVersion
sourceTimestamp
extractTimestamp
partition
batchId
checkpoint
30. Transformation

El pipeline puede transformar:

Source Model
      ↓
Transformation
      ↓
Target Model

Ejemplos:

column rename
type conversion
normalization
denormalization
field split
field merge
enum conversion
identifier conversion
31. Mapping

La relación debe ser explícita:

Source Field
      ↓
Mapping Rule
      ↓
Target Field

Ejemplo:

customer_id
      ↓
CustomerIdentifier
      ↓
customerId
32. Mapping Versioning

Cada migración debe conocer:

mappingVersion

para que los resultados puedan reproducirse.

33. Transformation Versioning

Igualmente:

transformationVersion

Debe poder determinarse exactamente qué lógica produjo un dato migrado.

34. Deterministic Transformation

Siempre que sea posible:

same source
+
same transformation version
=
same target result

Esto simplifica:

testing
replay
verification
rollback
35. Data Type Conversion

Debe controlarse:

string → UUID
integer → bigint
timestamp → timestamp with timezone
legacy enum → canonical enum

Toda conversión potencialmente destructiva debe detectarse.

36. Lossy Transformation

Si:

Source Information > Target Information

la migración es potencialmente lossful.

Debe declararse explícitamente.

Ejemplo:

Source:
full precision decimal

Target:
rounded decimal

Debe existir una política que autorice la pérdida.

37. Null Semantics

Debe definirse cómo mapear:

NULL
empty string
missing field
default value
unknown

No deben confundirse automáticamente.

38. Identifier Migration

Los IDs pueden:

remain stable
be remapped
be namespaced
be regenerated

Si cambian:

Old ID → New ID

debe existir un mapping persistente.

39. Identity Mapping
IdentityMap
├── sourceId
├── targetId
├── entityType
├── migrationId
└── createdAt

Esto es fundamental para relaciones y referencias externas.

40. Relationship Migration

Las relaciones deben migrarse después de resolver identidades:

Source A → Source B

A → Target A
B → Target B

Target A → Target B
41. Foreign Keys

Las foreign keys requieren:

identity resolution
ordering
referential validation
42. Migration Ordering

Dependencias típicas:

Reference Data
      ↓
Parent Entities
      ↓
Child Entities
      ↓
Relationships
      ↓
Derived Data
43. Dependency Graph

La migración puede representarse:

Customer
   │
   ├── Account
   │      └── Transaction
   │
   └── Address

El orden debe respetar dependencias.

44. Cyclic Dependencies

Si existe:

A → B
B → A

la estrategia puede requerir:

deferred constraints
staging records
two-phase linking
45. Staging Layer

Para migraciones complejas:

Source
  ↓
Staging
  ↓
Validation
  ↓
Target

El staging permite:

inspection
replay
validation
transformation
46. Migration Validation

La validación debe existir en múltiples niveles:

structural
schema
record
referential
semantic
aggregate
business
47. Structural Validation

Validar:

row count
file count
partition count
object count
byte count
48. Record Validation

Comparar:

source record
vs
target record

según las reglas de transformación.

49. Referential Validation

Debe comprobar:

foreign keys
references
parent-child relationships
cross-entity links
50. Semantic Validation

No basta con que el schema sea válido.

Debe verificarse que:

business meaning

se preserve.

Ejemplo:

Source status = ACTIVE
Target status = ACTIVE

y no:

ACTIVE → UNKNOWN

por un mapping incompleto.

51. Aggregate Validation

Comparar invariantes:

Source total transactions
=
Target total transactions

o:

Source balance
≈
Target balance

según reglas explícitas.

52. Business Validation

Ejemplos:

customer count
account count
open order count
transaction totals

deben coincidir según el scope migrado.

53. Checksum Validation

Puede utilizarse:

source checksum
target checksum

cuando la transformación permita comparación directa.

Cuando no:

canonicalized checksum
54. Reconciliation

La reconciliación compara:

Source
  vs
Migration State
  vs
Target

Ejemplo:

Source = 10,000
Target = 9,997

Difference = 3

La migración no debe declararse completa sin explicar la diferencia.

55. Reconciliation Report

Debe producir:

MigrationReconciliationReport
├── migrationId
├── sourceCount
├── targetCount
├── matched
├── missing
├── extra
├── transformed
├── failed
└── exceptions
56. Migration Exceptions

Los registros problemáticos deben separarse:

Successful
Failed
Skipped
Deferred
Invalid

Nunca deben desaparecer silenciosamente.

57. Dead Letter Migration Records

Puede existir:

Migration DLQ

para registros que no pudieron migrarse.

Debe conservar:

sourceReference
error
attempts
timestamp
migrationVersion
58. Retry

Errores transitorios:

network timeout
temporary storage failure
rate limit

pueden reintentarse.

Errores permanentes:

invalid data
unsupported type
mapping violation
constraint violation

requieren tratamiento explícito.

59. Idempotency

Reprocesar un batch debe ser seguro.

same migrationId
+
same source identity
+
same version

no debe generar duplicados.

60. Migration Checkpoint

Cada unidad migrable debe poder tener:

checkpoint

Ejemplo:

Partition 1 ✓
Partition 2 ✓
Partition 3 ✓
Partition 4 ← checkpoint
61. Resume

Si el proceso falla:

Failure
 ↓
Load Checkpoint
 ↓
Resume

No debe ser necesario reiniciar siempre desde cero.

62. Parallel Migration

Los datos pueden dividirse:

Partition A ──► Worker 1
Partition B ──► Worker 2
Partition C ──► Worker 3
Partition D ──► Worker 4

Debe existir control de:

ordering
dependencies
duplicate processing
resource limits
63. Migration Concurrency

Debe definirse:

maxWorkers
batchSize
transactionSize
sourceConcurrency
targetConcurrency
64. Backpressure

Si el target no soporta la velocidad:

Extractor
    ↓
Queue
    ↓
Transformer
    ↓
Loader

La cola permite desacoplar velocidades.

65. Target Protection

Una migración nunca debe degradar el target hasta comprometer producción.

Debe existir:

rate limiting
load shedding
throttling
maintenance windows
66. Migration Isolation

El workload migratorio debe poder aislar:

CPU
memory
IO
connections
network

cuando sea necesario.

67. Cutover

El cutover es el momento en que:

Source

deja de ser el sistema autoritativo y:

Target

pasa a serlo.

68. Cutover Preconditions

Antes del cutover:

migration complete
validation passed
reconciliation passed
lag acceptable
target healthy
rollback available
monitoring active
69. Cutover State
PRE_CUTOVER
     ↓
CUTOVER_STARTED
     ↓
TARGET_AUTHORITY
     ↓
POST_CUTOVER_VALIDATION
     ↓
CUTOVER_COMPLETE
70. Freeze Window

Puede utilizarse:

Write Freeze

durante el cutover para garantizar consistencia.

71. Read Cutover

La lectura puede cambiar:

Source Reads
    ↓
Dual Read
    ↓
Target Reads

permitiendo una transición gradual.

72. Write Cutover

La escritura puede cambiar:

Source Writes
    ↓
Dual Write
    ↓
Target Only
73. Shadow Migration

El target puede recibir datos pero no servir tráfico:

Production
   ↓
Source

Production
   ↓
Target (shadow)

Permite validar comportamiento.

74. Dual Read Validation

Durante shadow:

Read Source
Read Target
Compare

Las diferencias deben registrarse.

75. Rollback

El rollback debe estar diseñado antes del cutover.

Target
  ↓
Failure
  ↓
Rollback Decision
  ↓
Source Authority Restored
76. Rollback Conditions

Ejemplos:

validation failure
critical error rate
data mismatch
performance regression
security violation
target instability
77. Rollback Complexity

Cuanto más tarde ocurre el rollback:

Rollback Cost ↑

Por eso debe existir una ventana explícita:

Rollback Window
78. Post-Cutover Rollback

Después de que el target haya recibido nuevas escrituras:

Target
   ↓
New Writes
   ↓
Rollback

puede requerir:

reverse migration
change replay
write reconciliation

Por tanto, rollback no significa simplemente "volver a apuntar el DNS".

79. Migration Completion

Una migración sólo termina cuando:

Data Loaded
AND
Validation Passed
AND
Reconciliation Passed
AND
Cutover Passed
AND
Exceptions Resolved
80. Migration Certification

Para migraciones críticas:

Migration Certification

debe registrar:

migrationId
scope
source
target
versions
validation results
exceptions
approvals
cutover timestamp
81. Migration Audit

Eventos:

MigrationCreated
MigrationStarted
BatchStarted
BatchCompleted
TransformationFailed
ValidationStarted
ValidationFailed
ReconciliationCompleted
CutoverStarted
CutoverCompleted
RollbackStarted
RollbackCompleted
MigrationCompleted
82. Migration Observability

Métricas:

recordsRead
recordsWritten
recordsFailed
recordsSkipped
bytesRead
bytesWritten
migrationRate
migrationLag
validationFailures
reconciliationDifferences
retryCount
workerUtilization
83. Progress

Debe poder calcularse:

progress =
processed_records / total_records

cuando el total sea conocido.

Para streams:

watermark
lag

pueden ser métricas más útiles.

84. Migration Health

Estados:

HEALTHY
DEGRADED
BLOCKED
FAILING
FAILED
ROLLING_BACK
85. Migration Security

Principios:

Least Privilege
Encryption in Transit
Encryption at Rest
Tenant Isolation
Credential Isolation
Auditability
Controlled Access
86. Source Credentials

Los credenciales del origen deben estar separados de:

target credentials

y no deben almacenarse en el migration payload.

87. Target Credentials

El migration worker debe recibir sólo los privilegios necesarios:

read source
write target
validate target

No necesariamente:

drop database
change security policy
88. Sensitive Data

La migración no debe producir copias adicionales innecesarias de:

PII
credentials
tokens
secrets
financial data
89. Temporary Storage

Si se usa staging:

Source
 ↓
Temporary Storage
 ↓
Target

el staging debe tener:

encryption
access control
expiration
disposal
audit
90. Temporary Data Disposal

Una vez finalizada:

Migration Complete
      ↓
Temporary Artifacts
      ↓
Disposal

Debe integrarse con E47.

91. Multi-Tenant Migration

Debe soportarse:

tenant-by-tenant
batch-of-tenants
all-tenants

según el caso.

92. Tenant Isolation

Nunca debe mezclarse accidentalmente:

Tenant A data
+
Tenant B target scope

El migration scope debe validar:

tenantId

en origen y destino.

93. Tenant Migration

Una migración de tenant puede ser:

Tenant A
Source Region
      ↓
Migration
      ↓
Target Region

Debe preservar:

tenant identity
authorization boundary
data ownership
references
configuration
94. Cross-Region Migration

Debe considerar:

latency
egress
residency
encryption
replication
regional availability
95. Region Cutover
Region A
  ↓
Migration
  ↓
Region B
  ↓
Validation
  ↓
Traffic Shift
96. Domain Migration

Cuando una entidad cambia de bounded context:

Old Domain
    ↓
Mapping
    ↓
New Domain

La migración debe preservar invariantes de dominio, no simplemente columnas.

97. Service Decomposition Migration

Ejemplo:

Monolith
   │
   ├── Customer
   ├── Billing
   └── Orders

hacia:

Customer Service
Billing Service
Order Service

Cada dominio puede requerir una migración independiente.

98. Data Ownership

Después del cutover debe existir un único owner lógico.

Before:
Legacy Service owns Data

After:
New Service owns Data

No debe existir ownership ambiguo indefinidamente.

99. Migration Dependencies

Las migraciones pueden depender de:

schema
application version
event version
API version
configuration
feature flag
100. Feature Flags

Una migración puede utilizar:

read_from_target
write_to_target
enable_new_model
enable_new_region

Pero los feature flags no sustituyen las garantías de consistencia.

101. Migration Compatibility Layer

Puede existir:

Legacy API
    ↓
Compatibility Layer
    ↓
New Data Model

Esto permite migrar internamente sin romper inmediatamente consumidores externos.

102. API Contract Preservation

Si la migración cambia el backend pero no el contrato:

Client
  ↓
Same API
  ↓
New Data Store

la migración puede ser transparente al cliente.

103. Event Compatibility

Si existe event-driven architecture:

Source
  ↓
Events
  ↓
Migration
  ↓
Target

Debe evitarse producir eventos duplicados o semánticamente incompatibles.

104. Event Replay

Puede utilizarse:

Historical Events
      ↓
Replay
      ↓
Target State

cuando el modelo lo permita.

Debe garantizar:

ordering
idempotency
version compatibility
105. Derived Data

Después de migrar el estado primario:

Primary Data
   ↓
Projection Rebuild
   ↓
Read Models

Las proyecciones no necesariamente deben migrarse directamente si pueden reconstruirse.

106. Migration vs Projection

Si un read model puede regenerarse:

Source of Truth
      ↓
Migration
      ↓
Target Source of Truth
      ↓
Projection Rebuild

puede ser preferible a migrar el read model directamente.

107. Cache Handling

Las caches normalmente deben:

invalidate
rebuild

en lugar de tratarse como fuente primaria de migración.

108. Search Index Migration

Puede utilizarse:

Target Data
    ↓
Reindex
    ↓
Search Index

en lugar de copiar directamente índices antiguos.

109. Analytics Migration

Los datos analíticos pueden requerir:

historical backfill
schema mapping
partition migration
aggregate validation
110. Migration Backfill

Un backfill puede ejecutarse después del cutover:

Target Current
      ↓
Historical Backfill

Debe ser:

idempotent
throttled
observable
reconcilable
111. Migration Performance

Debe optimizarse:

throughput
latency
batch size
parallelism
IO
compression
network utilization

sin comprometer la estabilidad.

112. Adaptive Throttling

El worker puede ajustar su velocidad:

Target Load ↑
      ↓
Migration Rate ↓

Target Load ↓
      ↓
Migration Rate ↑
113. Migration Cost

Debe poder medirse:

compute
storage
network
egress
temporary storage
engineering execution
114. Migration Failure Domains

Una migración debe limitar el blast radius:

Migration
  ├── Tenant A
  ├── Tenant B
  ├── Tenant C

Un fallo en Tenant A no debería invalidar automáticamente todos los demás.

115. Partial Completion

Debe poder existir:

Tenant A → COMPLETE
Tenant B → COMPLETE
Tenant C → FAILED
Tenant D → PENDING

El estado global debe reflejarlo.

116. Migration Cancellation

Cancelar:

MIGRATING

no significa necesariamente:

DELETE TARGET DATA

Debe existir una política explícita:

pause
stop
rollback
retain partial result
117. Migration Pause

Un migration job puede pausarse:

MIGRATING
   ↓
PAUSED
   ↓
MIGRATING

sin perder checkpoint.

118. Migration Approval

Migraciones críticas pueden requerir:

technical approval
business approval
security approval
operations approval

según riesgo.

119. Migration Risk Classification
LOW
MEDIUM
HIGH
CRITICAL

Factores:

data volume
data sensitivity
downtime
business criticality
rollback complexity
cross-region impact
120. Migration Runbook

Toda migración crítica debe poseer:

pre-checks
execution steps
validation steps
cutover steps
rollback steps
post-checks
incident contacts
121. Dry Run

Antes de producción:

Source Snapshot
      ↓
Migration
      ↓
Validation
      ↓
Measure

El dry run debe revelar:

errors
duration
throughput
data differences
resource consumption
122. Replay Testing

Debe poder repetirse:

same snapshot
same migration version

para verificar determinismo.

123. Canary Migration

En migraciones grandes:

5 tenants
    ↓
Validate
    ↓
50 tenants
    ↓
Validate
    ↓
500 tenants

reduce el riesgo.

124. Migration Rollout
Canary
  ↓
Small Batch
  ↓
Medium Batch
  ↓
Large Batch
  ↓
Full Migration
125. Migration Completion Criteria

Una migración debe considerarse completada sólo si:

✓ All scoped data processed
✓ Validation passed
✓ Referential integrity passed
✓ Semantic validation passed
✓ Reconciliation passed
✓ Exceptions resolved
✓ Cutover completed
✓ Target healthy
✓ Source retirement approved
126. Source Retirement

Después de una migración:

Target Stable
      ↓
Observation Window
      ↓
Source Read-Only
      ↓
Source Retirement

El source no debe eliminarse inmediatamente después del cutover.

127. Observation Window

Debe existir un período durante el cual:

target

se monitoriza intensamente antes de retirar definitivamente:

source
128. Legacy Read-Only

Una estrategia segura:

Source
  ↓
READ_ONLY

durante la ventana de observación.

Esto permite:

audit
comparison
emergency retrieval
rollback support
129. Legacy Disposal

Cuando el source deja de ser necesario:

Legacy Source
      ↓
Retention Evaluation
      ↓
Archive / Disposal

E48 y E47 pueden intervenir.

130. Migration Auditability

Debe ser posible responder:

What migrated?
From where?
To where?
When?
By which version?
Using which mapping?
What failed?
What was skipped?
Who approved?
Who performed cutover?
131. Migration Provenance

Cada resultado debe poder rastrearse:

Target Record
      ↓
Migration ID
      ↓
Source Record
      ↓
Transformation Version
      ↓
Migration Run
132. Migration Lineage

Modelo:

Source
  │
  ▼
Extract
  │
  ▼
Transform
  │
  ▼
Load
  │
  ▼
Target

Cada etapa debe ser identificable.

133. Migration Data Contract

Debe existir un contrato conceptual:

Source Schema
      ↓
Migration Contract
      ↓
Target Schema

El contrato define:

required fields
optional fields
transformations
constraints
null semantics
identity rules
134. Contract Validation

Antes de ejecutar:

Source
  ↓
Contract Validator

debe detectar incompatibilidades.

135. Contract Drift

Si el source cambia durante la migración:

Source Schema v1
      ↓
Source Schema v2

el sistema debe detectar:

SCHEMA_DRIFT

y aplicar:

pause
adapt
or fail

según política.

136. Migration Freeze

Para migraciones sensibles puede requerirse:

Schema Freeze

o:

Change Freeze

durante el proceso.

137. Migration Security Boundary
Source Security Domain
        │
        ▼
Migration Boundary
        │
        ▼
Target Security Domain

Los permisos no deben heredarse implícitamente de un sistema al otro.

138. Secrets

Nunca deben incluirse en:

migration logs
migration reports
migration payloads
temporary files
139. Encryption in Transit

Todo tráfico entre:

source
staging
target

debe utilizar transporte seguro cuando los datos sean sensibles o la plataforma lo requiera.

140. Migration Data Retention

Los artefactos temporales de una migración deben tener una política:

migration logs
staging files
failed batches
snapshots
mapping artifacts
validation reports

No deben conservarse indefinidamente por defecto.

141. Migration Artifact Disposal
Migration Complete
       ↓
Artifact Inventory
       ↓
Retention Evaluation
       ↓
Dispose Temporary Artifacts
142. Disaster During Migration

Si ocurre un desastre:

Migration
   ↓
Failure
   ↓
Recovery

debe existir información suficiente para saber:

last checkpoint
last valid batch
target state
source state
143. Migration Recovery
Restore Migration State
        ↓
Load Checkpoint
        ↓
Reconcile
        ↓
Resume / Rollback
144. Migration Invariants
Invariant 1 — No Silent Loss

Ningún registro dentro del scope de migración puede desaparecer silenciosamente.

Invariant 2 — No Unauthorized Duplication

La migración no debe crear duplicados lógicos no autorizados.

Invariant 3 — Referential Integrity

Las relaciones migradas deben continuar apuntando a entidades válidas.

Invariant 4 — Tenant Isolation

Los datos de un tenant no pueden terminar dentro del scope de otro tenant.

Invariant 5 — Traceability

Todo dato migrado debe poder rastrearse hasta su origen.

Invariant 6 — Idempotency

Reprocesar una unidad migratoria no debe producir corrupción o duplicación.

Invariant 7 — Explicit Loss

Cualquier pérdida de información debe ser explícita, autorizada y verificable.

Invariant 8 — Cutover Safety

El target no puede convertirse en autoridad hasta superar las validaciones requeridas.

145. Reference Architecture
                         ┌───────────────────────┐
                         │ MIGRATION CONTROL      │
                         │ PLANE                  │
                         ├───────────────────────┤
                         │ Definition             │
                         │ Policy                 │
                         │ Planning               │
                         │ Checkpoints            │
                         │ Validation             │
                         │ Cutover               │
                         │ Rollback              │
                         │ Audit                 │
                         └───────────┬───────────┘
                                     │
                                     ▼
┌──────────────┐          ┌───────────────────────┐
│ SOURCE       │─────────►│ MIGRATION DATA PLANE  │
└──────────────┘          ├───────────────────────┤
                          │ Extract               │
                          │ Transform             │
                          │ Map                   │
                          │ Validate              │
                          │ Load                  │
                          │ Reconcile             │
                          └───────────┬───────────┘
                                      │
                                      ▼
                             ┌────────────────┐
                             │ TARGET         │
                             └───────┬────────┘
                                     │
                                     ▼
                             ┌────────────────┐
                             │ CUTOVER        │
                             └───────┬────────┘
                                     │
                    ┌────────────────┴────────────────┐
                    ▼                                 ▼
              TARGET AUTHORITY                   ROLLBACK
146. Migration Domain Model
Migration
├── migrationId
├── source
├── target
├── scope
├── strategy
├── status
├── mappingVersion
├── transformationVersion
├── validationPolicy
├── cutoverPolicy
├── rollbackPolicy
├── startedAt
├── completedAt
└── owner
147. Migration Batch
MigrationBatch
├── batchId
├── migrationId
├── partition
├── sourceCheckpoint
├── targetCheckpoint
├── recordCount
├── byteCount
├── status
├── attemptCount
├── startedAt
└── completedAt
148. Migration Error
MigrationError
├── errorId
├── migrationId
├── batchId
├── sourceReference
├── errorType
├── message
├── retryable
├── attemptCount
└── occurredAt
149. Migration Reconciliation
MigrationReconciliation
├── migrationId
├── scope
├── sourceCount
├── targetCount
├── matchedCount
├── missingCount
├── extraCount
├── invalidCount
├── transformedCount
└── status
150. Migration Events
MigrationCreated
MigrationStarted
MigrationBatchStarted
MigrationBatchCompleted
MigrationBatchFailed
MigrationPaused
MigrationResumed
MigrationValidationStarted
MigrationValidationCompleted
MigrationReconciliationCompleted
MigrationCutoverStarted
MigrationCutoverCompleted
MigrationRollbackStarted
MigrationRollbackCompleted
MigrationCompleted
MigrationFailed
151. Relationship to E48

E48 y E49 se complementan:

                 DATA MOVEMENT
                      │
          ┌───────────┴───────────┐
          ▼                       ▼
       ARCHIVE                 MIGRATE
          │                       │
          ▼                       ▼
Long-Term Storage           Target System
          │                       │
          ▼                       ▼
     E48 Lifecycle             E49 Lifecycle

Un dato archivado puede posteriormente necesitar migración:

Archive
  ↓
Extract
  ↓
Transform
  ↓
Target

pero eso debe tratarse como una migración controlada, no como una simple restauración.

152. Relationship to E43/E44

E43:

Data Integrity

define la integridad que la migración debe preservar.

E44:

Data Consistency

define las garantías de consistencia.

E49 implementa:

Source
  ↓
Migration
  ↓
Target

sin violar esas garantías.

153. Relationship to E24/E25
E24 Transformation
        │
        ▼
Transformation Rules

E25 Mapping
        │
        ▼
Field / Entity Mapping

E49 Migration
        │
        ▼
Execution of the migration

E49 consume esos mecanismos; no debe duplicar innecesariamente su definición.

154. End-to-End Migration
                    E49 DATA MIGRATION

Source
  │
  ▼
Discovery
  │
  ▼
Inventory
  │
  ▼
Scope
  │
  ▼
Mapping
  │
  ▼
Transformation
  │
  ▼
Snapshot
  │
  ▼
Initial Load
  │
  ▼
Incremental Changes
  │
  ▼
Validation
  │
  ▼
Reconciliation
  │
  ▼
Cutover
  │
  ├────────────► Rollback
  │
  ▼
Target Authority
  │
  ▼
Observation
  │
  ▼
Legacy Retirement
155. Completion Criteria

E49 se considera arquitectónicamente completo cuando EVOXA dispone de:

✓ Migration boundary
✓ Migration classification
✓ Source assessment
✓ Target assessment
✓ Migration definition
✓ Migration scope
✓ Migration state machine
✓ Migration strategies
✓ Snapshot strategy
✓ Extract architecture
✓ Transformation integration
✓ Mapping integration
✓ Identity mapping
✓ Relationship migration
✓ Dependency ordering
✓ Staging
✓ Validation
✓ Reconciliation
✓ Error handling
✓ Retry
✓ Idempotency
✓ Checkpointing
✓ Resume
✓ Parallel execution
✓ Backpressure
✓ Target protection
✓ Cutover
✓ Rollback
✓ Shadow migration
✓ Dual write
✓ Change capture
✓ Backfill
✓ Tenant migration
✓ Cross-region migration
✓ Domain migration
✓ Service migration
✓ Security
✓ Temporary artifact handling
✓ Audit
✓ Observability
✓ Provenance
✓ Data lineage
✓ Schema drift handling
✓ Migration certification
✓ Source retirement
✓ Architectural invariants
✓ Reference architecture
156. Principio Rector de E49

Data Migration es un proceso controlado de transformación y transferencia de datos desde un sistema de origen hacia un sistema de destino, preservando identidad, integridad, relaciones, semántica, seguridad y trazabilidad, y permitiendo validación, reconciliación, cutover y rollback antes de declarar el destino como autoridad.

La relación con los capítulos anteriores queda:

E45 — Data Lifecycle
        │
        ├──────────────┐
        ▼              ▼
E46 — Retention     E49 — Migration
        │              │
        ▼              ▼
E48 — Archival     New Target
        │
        ▼
E47 — Disposal

E48 mueve datos fuera del plano operativo para conservarlos. E49 mueve datos desde un origen hacia un nuevo estado o sistema, transformándolos y verificándolos hasta que el destino pueda convertirse en la nueva fuente de verdad.
