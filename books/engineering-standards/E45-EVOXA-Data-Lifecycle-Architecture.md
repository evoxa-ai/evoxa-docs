E45 — EVOXA Data Lifecycle Architecture
Engineering Specification

Pertenece a: Engineering Specification
Capítulo: E45 — Data Lifecycle Architecture
Anterior: E44 — Consistency Architecture
Siguiente: E46 — EVOXA Data Retention Architecture

1. Propósito

E45 define cómo los datos de EVOXA nacen, se activan, evolucionan, se utilizan, envejecen, archivan, recuperan y finalmente se eliminan.

El principio central es:

Todo dato de EVOXA debe tener un lifecycle explícito desde su creación hasta su disposición final.

El lifecycle no debe depender de que cada servicio implemente su propia interpretación de:

create
update
read
archive
restore
expire
delete

Debe existir un modelo arquitectónico común.

2. Data Lifecycle Boundary

E45 cubre:

Data Creation
Data Activation
Data Usage
Data Mutation
Data Versioning
Data State Transitions
Data Aging
Data Expiration
Data Archival
Data Restoration
Data Retention Interaction
Data Deletion
Data Purging
Data Destruction
Lifecycle Automation
Lifecycle Policies
Lifecycle Events
Lifecycle Metadata
Lifecycle Observability

E45 se relaciona directamente con:

E02 → Database Architecture
E17 → Caching
E18 → Configuration
E19 → Feature Flags
E23 → Serialization
E26 → Projection
E28 → Read Models
E29 → Search
E30 → Reporting
E31 → Analytics
E40 → Recovery
E41 → Disaster Recovery
E42 → Backup & Restore
E43 → Data Integrity
E44 → Consistency
3. Data Lifecycle Principle

Un dato no debe considerarse simplemente:

EXISTS

Debe tener un estado de lifecycle.

Modelo base:

CREATED
   ↓
ACTIVE
   ↓
UPDATED
   ↓
AGED
   ↓
EXPIRED
   ↓
ARCHIVED
   ↓
PURGED

No todos los datos recorrerán exactamente todos los estados.

4. Lifecycle State Machine

Modelo de referencia:

                  ┌─────────────┐
                  │   CREATED   │
                  └──────┬──────┘
                         ▼
                  ┌─────────────┐
                  │   ACTIVE    │
                  └──────┬──────┘
                         │
                 ┌───────┴────────┐
                 ▼                ▼
             UPDATED            AGING
                 │                │
                 └───────┬────────┘
                         ▼
                  ┌─────────────┐
                  │   EXPIRED   │
                  └──────┬──────┘
                         │
               ┌─────────┴─────────┐
               ▼                   ▼
          ARCHIVED              PURGED
               │
               ▼
           RESTORED
               │
               ▼
             ACTIVE

La máquina real depende del tipo de dato.

5. Data Lifecycle Classes

EVOXA debe clasificar los datos antes de definir su lifecycle.

Ejemplo:

Operational Data
Transactional Data
Reference Data
Configuration Data
Event Data
Audit Data
Security Data
Analytical Data
Derived Data
Temporary Data
Cache Data
Search Data
AI Context Data
Agent State
Workflow State

Cada clase puede tener lifecycle diferente.

6. Lifecycle Ownership

Cada tipo de dato debe tener:

Lifecycle Owner

El owner es responsable de:

creation policy
mutation policy
retention interaction
expiration
archival
deletion
recovery
compliance behavior
7. Lifecycle Metadata

Los datos lifecycle-aware deberían poder asociarse con:

createdAt
updatedAt
activatedAt
lastAccessedAt
expiresAt
archivedAt
purgedAt
version
lifecycleState
lifecyclePolicy
owner
tenantId

No todos los campos son obligatorios para todos los datos.

8. Created State

CREATED representa que el objeto existe, pero todavía puede no estar disponible para uso normal.

Ejemplo:

CREATED
   ↓
validation
   ↓
activation

Esto permite separar:

physical creation

de:

business activation
9. Active State

ACTIVE significa:

data is valid
data is usable
data participates in normal operations

Es normalmente el estado principal del dato operativo.

10. Updated State

Actualizar datos no implica necesariamente cambiar su lifecycle state.

Ejemplo:

ACTIVE
  ↓
UPDATE
  ↓
ACTIVE

El lifecycle debe distinguir:

state transition

de:

content mutation
11. Aging

Un dato puede entrar en una etapa de envejecimiento:

ACTIVE
   ↓
AGING

cuando:

time elapsed
usage decreased
business relevance decreased
retention threshold approaching

Esto no significa todavía que deba eliminarse.

12. Expired

EXPIRED significa que el dato ya no debe considerarse válido para su propósito operativo normal.

Ejemplo:

Temporary Token
ACTIVE
   ↓
expiration time
   ↓
EXPIRED
13. Expired ≠ Deleted

Un dato puede estar:

EXPIRED

y todavía existir físicamente.

Esto permite:

audit
recovery
verification
cleanup

antes del purge definitivo.

14. Archived

ARCHIVED significa:

not part of hot operational state

pero:

still retained

Puede conservarse para:

historical access
audit
analytics
legal requirements
business history
15. Archive Characteristics

Los datos archivados pueden pasar de:

hot storage

a:

cold storage

o a otro storage class.

El cambio físico no debe alterar el significado lógico del dato.

16. Restore

Un dato archivado puede:

ARCHIVED
   ↓
RESTORED
   ↓
ACTIVE

pero solamente si el dominio permite reactivación.

En otros dominios:

ARCHIVED
   ↓
READ-ONLY

puede ser la única transición válida.

17. Purged

PURGED representa la eliminación física o lógica irreversible según el contrato.

EXPIRED
   ↓
PURGED

o:

ARCHIVED
   ↓
PURGED
18. Deletion vs Purge

Debe distinguirse:

DELETE

de:

PURGE

DELETE puede significar que el objeto deja de estar disponible operacionalmente.

PURGE significa que el almacenamiento y las representaciones correspondientes han sido eliminados conforme a la política.

19. Tombstone

En sistemas distribuidos, eliminar físicamente un registro inmediatamente puede provocar problemas de consistencia.

Puede utilizarse:

TOMBSTONE

para indicar:

this entity was deleted

y permitir que:

projections
replicas
indexes
caches

converjan correctamente.

20. Tombstone Lifecycle
ACTIVE
  ↓
DELETE REQUEST
  ↓
TOMBSTONE
  ↓
PROPAGATE
  ↓
RECONCILE
  ↓
PURGE
21. Lifecycle Policy

Cada clase de dato debe tener una política equivalente a:

lifecyclePolicy
├── creation
├── activation
├── mutation
├── aging
├── expiration
├── archival
├── restoration
├── deletion
├── purge
└── audit
22. Lifecycle Policy Example
Order

Creation:
  immediate

Active:
  until business completion

Aging:
  after 90 days

Archive:
  after 1 year

Restore:
  read-only

Purge:
  according to retention policy

Los tiempos concretos pertenecen a las políticas del dominio y cumplimiento aplicable.

23. Lifecycle Automation

Los cambios lifecycle deben poder ejecutarse automáticamente:

Scheduler
   ↓
Lifecycle Engine
   ↓
Policy Evaluation
   ↓
State Transition
   ↓
Event
   ↓
Consumers
24. Lifecycle Engine

El lifecycle engine es responsable de:

discover eligible records
evaluate policy
validate transition
execute transition
emit lifecycle event
record result
retry failure
25. Lifecycle Eligibility

Un objeto puede ser elegible para transición por:

age
lastAccess
businessState
expirationDate
workflowCompletion
externalSignal
policyChange
manualCommand
26. Time-Based Lifecycle

Ejemplo:

createdAt = T0

T0 + 30d → AGING
T0 + 90d → EXPIRED
T0 + 180d → ARCHIVED
T0 + 365d → PURGED

Los intervalos son ilustrativos.

27. Event-Based Lifecycle

No todos los lifecycle transitions deben depender del tiempo.

Ejemplo:

Workflow Completed
      ↓
Document
      ↓
ARCHIVE

Otro ejemplo:

Subscription Cancelled
      ↓
Account Data
      ↓
Lifecycle Policy
28. Business-State Lifecycle

Puede depender del estado de negocio:

ORDER
  ACTIVE
    ↓
  COMPLETED
    ↓
  HISTORICAL
    ↓
  ARCHIVED

El lifecycle técnico debe respetar las invariantes del dominio.

29. Lifecycle Event

Cada transición importante puede generar:

DataCreated
DataActivated
DataUpdated
DataExpired
DataArchived
DataRestored
DataDeleted
DataPurged
30. Lifecycle Event Contract

Un evento lifecycle debería poder identificar:

eventId
entityId
entityType
tenantId
previousState
newState
occurredAt
effectiveAt
actor
reason
policy
version
correlationId
31. Effective Time

Debe distinguirse:

eventOccurredAt

de:

effectiveAt

porque una transición puede ejecutarse después de que conceptualmente debía producirse.

32. Lifecycle Idempotency

Las transiciones deben ser idempotentes cuando sea posible.

Ejemplo:

ARCHIVE(entity-123)
ARCHIVE(entity-123)

no debería producir:

corruption
duplicate archive
33. Lifecycle Concurrency

Dos procesos podrían intentar:

ARCHIVE

y:

RESTORE

simultáneamente.

El sistema debe proteger la transición mediante:

version check
locking
state transition validation

según el dominio.

34. Valid State Transitions

No toda transición debe permitirse.

Ejemplo:

ACTIVE → ARCHIVED       ✓
ARCHIVED → RESTORED     ✓
ACTIVE → PURGED         ✗
PURGED → ACTIVE         ✗

Las reglas deben ser explícitas.

35. Lifecycle State Machine per Domain

No debe existir necesariamente una única máquina universal.

Por ejemplo:

Document:
DRAFT → ACTIVE → ARCHIVED → PURGED

mientras:

Session:
CREATED → ACTIVE → EXPIRED → PURGED

y:

Financial Record:
CREATED → POSTED → CLOSED → ARCHIVED
36. Lifecycle and Multi-Tenancy

Todo lifecycle operation debe preservar:

tenantId

El lifecycle engine nunca debe ejecutar accidentalmente:

global delete

sobre datos multi-tenant sin aislamiento explícito.

37. Tenant Lifecycle Policy

Los tenants pueden tener políticas diferentes cuando el modelo de negocio lo permita:

Tenant
   ↓
Lifecycle Policy

Pero nunca deben poder superar restricciones superiores de:

security
governance
compliance
platform policy
38. Lifecycle and Data Integrity

Antes de una transición destructiva:

PURGE

puede requerirse:

integrity verification

para asegurar que no se está eliminando el estado equivocado.

39. Lifecycle and Consistency

E44 exige que los estados derivados converjan.

Por tanto:

DELETE SOURCE

no debe implicar automáticamente:

DELETE ONLY SOURCE

Debe propagarse:

source
 ↓
event
 ↓
projection
 ↓
search
 ↓
cache
 ↓
analytics

según las necesidades de cada sistema.

40. Lifecycle Propagation
Lifecycle Transition
        ↓
Lifecycle Event
        ↓
Consumers
   ┌────┼─────┐
   ▼    ▼     ▼
Read  Search Cache
Model

Cada consumidor debe aplicar su propia transición compatible.

41. Derived Data Lifecycle

Los datos derivados no deben vivir necesariamente tanto como el source.

Ejemplo:

Source Record
  retained 7 years

Search Index
  retained while source exists

Cache
  retained minutes

Analytics Projection
  retained 3 years
42. Lifecycle Dependency Graph
Source
  │
  ├── Read Model
  ├── Search Index
  ├── Cache
  ├── Analytics
  └── AI Index

Cada nodo debe declarar:

dependency
lifecycle
rebuild capability
43. Rebuildable Data

Los datos derivados deberían clasificarse:

REBUILDABLE
NON-REBUILDABLE
PARTIALLY_REBUILDABLE

Esto afecta:

retention
backup
archive
recovery
purge
44. Rebuildable Data Strategy

Si un dato puede reconstruirse desde:

source + events

no necesariamente necesita el mismo lifecycle que el authoritative data.

45. Non-Rebuildable Data

Los datos no reconstruibles requieren mayor protección.

Ejemplo conceptual:

Authoritative business record

Su lifecycle debe coordinarse con:

backup
retention
recovery
integrity
46. Lifecycle and Backup

No debe purgarse un dato solamente porque:

"it is old"

si todavía existe una dependencia legítima de:

backup
restore
audit
recovery

E42 debe coordinarse con E45.

47. Backup Dependency

Conceptualmente:

DATA
 ↓
RETENTION
 ↓
BACKUP
 ↓
ARCHIVE
 ↓
PURGE

La eliminación debe respetar las dependencias definidas por las políticas correspondientes.

48. Lifecycle and Disaster Recovery

Después de un disaster recovery:

restored data

puede tener lifecycle states distintos a los esperados.

Por ello:

restore
 ↓
lifecycle validation
 ↓
reconciliation

debe formar parte del proceso.

49. Lifecycle and Cache

Cuando una entidad pasa a:

DELETED

el cache debe recibir:

invalidation

o un mecanismo equivalente.

Nunca debe continuar sirviendo indefinidamente:

deleted entity
50. Lifecycle and Search

Cuando una entidad se elimina:

source deleted
 ↓
delete event
 ↓
search index
 ↓
remove document

Si el índice es eventualmente consistente, la ventana debe estar dentro de su contrato.

51. Lifecycle and Analytics

Los datos históricos pueden permanecer en analytics incluso después de abandonar el operational store.

Debe existir una regla explícita para:

source deletion
vs
analytical retention
52. Lifecycle and AI Data

Los datos utilizados por AI pueden existir en:

vector indexes
embeddings
prompt context
feature stores
agent memory
training datasets
evaluation datasets

Cada representación debe tener lifecycle propio.

No debe asumirse:

delete source
=
delete all derived AI representations

sin una política explícita de propagación.

53. AI Data Lineage

Para datos AI derivados debe existir, cuando sea necesario:

sourceId
derivedArtifactId
modelVersion
createdAt
lifecyclePolicy

Esto permite identificar qué derivados deben ser actualizados o eliminados.

54. Lifecycle and Agents

Un agente puede mantener:

short-term context
long-term memory
workflow state
task state
execution artifacts

Cada uno necesita lifecycle independiente.

55. Temporary Data

Los datos temporales deben tener:

expiresAt

desde su creación cuando sea posible.

Ejemplo:

temporary session
temporary upload
temporary execution artifact
temporary cache
56. TTL

TTL puede utilizarse para:

temporary data
cache
sessions
locks
ephemeral artifacts

pero no debe utilizarse como sustituto de una política de lifecycle de datos críticos.

57. Soft Delete

Soft delete:

deletedAt != null

permite:

recovery
audit
reconciliation

pero introduce complejidad.

Debe existir una política para cuándo pasar de:

SOFT_DELETED

a:

PURGED
58. Hard Delete

Hard delete elimina físicamente el registro.

Debe reservarse para casos donde:

irreversible deletion

sea permitido y correctamente coordinado.

59. Secure Deletion

Para datos sensibles, el lifecycle puede requerir:

logical deletion
+
physical deletion
+
derived-data deletion

según la política aplicable.

60. Lifecycle Audit

Las transiciones críticas deben auditar:

who
what
when
why
fromState
toState
policy
result
61. Lifecycle Actor

El actor puede ser:

USER
SERVICE
SYSTEM
SCHEDULER
POLICY_ENGINE
ADMIN
AGENT
RECOVERY_PROCESS

Debe poder distinguirse acción automática de acción humana.

62. Lifecycle Reason

Cada transición significativa debería poder explicar:

reason

Ejemplo:

"expiration policy reached"
"manual archival"
"tenant closure"
"workflow completed"
"retention policy"
63. Lifecycle Observability

Métricas:

recordsByLifecycleState
transitionsPerMinute
expirationRate
archiveRate
restoreRate
purgeRate
transitionFailures
transitionRetries
staleLifecycleRecords
64. Lifecycle SLO

Puede medirse:

Expiration latency
Archive latency
Deletion propagation latency
Purge completion latency
Lifecycle reconciliation latency
65. Lifecycle Backlog

Un sistema lifecycle puede acumular:

eligible records

Debe observarse:

eligibleCount
processingRate
backlogAge
oldestPendingTransition
66. Lifecycle Failure

Si una transición falla:

ACTIVE
   ↓
ARCHIVE FAILED

no debe dejarse un estado ambiguo.

Debe existir:

retry
quarantine
manual intervention

según el caso.

67. Transition Failure State

Puede utilizarse:

TRANSITION_PENDING
TRANSITION_FAILED

cuando el dominio necesite representar explícitamente el proceso.

68. Lifecycle Retry

Debe respetar:

idempotency
backoff
maxAttempts
dead-letter handling
observability
69. Lifecycle Dead Letter

Los casos que no pueden procesarse automáticamente pueden ir a:

Lifecycle DLQ

para:

inspection
repair
replay
manual resolution
70. Lifecycle Reconciliation

Debe poder responder:

"Which records are in the wrong lifecycle state?"

Ejemplo:

Policy:
archive after 365 days

Actual:
ACTIVE for 500 days

Resultado:

LIFECYCLE DIVERGENCE
71. Lifecycle Repair

La reparación puede ser:

automatic
semi-automatic
manual

Nunca debe ejecutar una acción destructiva automáticamente si el riesgo no está claramente definido.

72. Lifecycle Safety

Las operaciones irreversibles deben incorporar:

authorization
policy validation
scope validation
tenant validation
dependency validation
audit
73. Purge Safety Gate

Antes de PURGE:

verify ownership
      ↓
verify eligibility
      ↓
verify retention
      ↓
verify legal/compliance state
      ↓
verify dependencies
      ↓
execute purge
      ↓
verify
74. Lifecycle Dependency Check

Antes de eliminar una entidad:

Entity
 ├── References
 ├── Projections
 ├── Search
 ├── Cache
 ├── Workflows
 ├── Jobs
 └── External systems

deben evaluarse las dependencias relevantes.

75. Orphan Prevention

Después de lifecycle transitions no deberían quedar:

orphaned records
orphaned projections
orphaned files
orphaned indexes
orphaned workflows

sin una política explícita.

76. Cascade Lifecycle

Una entidad puede producir cascadas:

Parent
 ↓
Child
 ↓
Derived

Pero las cascadas deben ser:

explicit
bounded
observable
recoverable

No debe permitirse un cascade delete accidental a través de servicios.

77. Lifecycle Boundaries

El lifecycle engine no debe violar los boundaries de dominio.

Domain A
   │
   │ lifecycle contract
   ▼
Domain B

El dominio B debe aplicar su propia transición según su contrato.

78. Lifecycle API

Conceptualmente:

GET    /resource/{id}/lifecycle
POST   /resource/{id}/archive
POST   /resource/{id}/restore
DELETE /resource/{id}

La API exacta pertenece a E03.

79. Lifecycle Commands

Comandos internos:

ActivateData
ExpireData
ArchiveData
RestoreData
DeleteData
PurgeData
ReconcileLifecycle
80. Lifecycle Event Flow
Lifecycle Command
       ↓
Policy Validation
       ↓
State Transition
       ↓
Transaction
       ↓
Lifecycle Event
       ↓
Propagation
       ↓
Verification
81. Lifecycle Versioning

Las políticas pueden evolucionar.

Por tanto:

lifecyclePolicyVersion

puede ser necesario.

Esto permite saber:

"Why was this record archived?"

años después.

82. Policy Change

Cambiar:

archiveAfter = 365d

a:

archiveAfter = 180d

no debería automáticamente modificar millones de registros sin una estrategia de migration/reconciliation.

83. Lifecycle Migration

Cuando cambia una política:

New Policy
   ↓
Eligibility Scan
   ↓
Impact Analysis
   ↓
Controlled Execution
   ↓
Reconciliation
84. Lifecycle Batch Processing

Para grandes volúmenes:

scan
 ↓
partition
 ↓
batch
 ↓
transition
 ↓
checkpoint

Debe soportar:

pause
resume
retry
rate limiting
85. Lifecycle Throttling

Las operaciones lifecycle masivas no deben saturar:

database
event broker
storage
search
external integrations

Debe existir control de capacidad.

86. Lifecycle Priority

Puede existir:

critical
normal
background

para diferenciar:

security expiration

de:

cold archival
87. Lifecycle and Capacity

Archival y purge pueden afectar directamente:

storage capacity
IOPS
network
CPU
database locks

Por ello deben integrarse con E37 — Runtime Capacity Architecture.

88. Lifecycle and Resilience

Una operación lifecycle masiva debe sobrevivir:

worker restart
node failure
network failure
database failover
partial batch completion

Debe poder reanudarse sin duplicar efectos.

89. Lifecycle Checkpoint

Ejemplo:

batchId = 123

processedUntil = entity-500000

Permite:

resume

sin reiniciar todo el proceso.

90. Lifecycle Data Lineage

Debe poder responderse:

Where did this data come from?
What derived data exists?
Which lifecycle policy governs it?
What happened to it?
91. Lifecycle Graph
Source Record
     │
     ├── Projection
     ├── Search
     ├── Cache
     ├── Analytics
     └── AI Artifact

El lifecycle system debe conocer dependencias suficientes para coordinar transiciones críticas.

92. Lifecycle and Data Classification

El lifecycle depende de la clasificación:

PUBLIC
INTERNAL
CONFIDENTIAL
SENSITIVE
CRITICAL

La clasificación puede modificar:

retention
archive
encryption
access
purge
audit
93. Lifecycle and Governance

E14 Governance y las políticas superiores pueden imponer:

minimum retention
maximum retention
deletion requirements
audit requirements

E45 implementa el lifecycle; no redefine unilateralmente esas políticas.

94. Lifecycle and Legal Hold

Un dato que normalmente sería elegible para purge puede quedar bloqueado:

PURGE ELIGIBLE
       ↓
LEGAL HOLD
       ↓
PURGE BLOCKED

El lifecycle engine debe poder representar esta excepción.

95. Lifecycle Hold

Los holds pueden incluir:

legal
security
investigation
business
recovery
manual
96. Lifecycle Hold Release

Al eliminarse el hold:

HOLD RELEASED
      ↓
RE-EVALUATE POLICY
      ↓
PURGE / ARCHIVE / ACTIVE

No debe asumirse que el dato debe eliminarse automáticamente sin reevaluación.

97. Lifecycle and Tenant Deletion

Cuando un tenant termina:

Tenant Closure
      ↓
Lifecycle Plan
      ↓
Operational Data
Derived Data
Files
Indexes
Caches
Backups
Analytics
AI Artifacts

deben tratarse según sus respectivas políticas.

98. Tenant Deletion Workflow
REQUEST
  ↓
VALIDATE
  ↓
FREEZE
  ↓
DEPENDENCY DISCOVERY
  ↓
DELETE/ARCHIVE
  ↓
PROPAGATE
  ↓
RECONCILE
  ↓
PURGE
  ↓
VERIFY
99. Lifecycle Completion

Una operación lifecycle destructiva no debe considerarse completada simplemente porque:

primary database = deleted

Debe verificarse el alcance definido:

primary
replicas
projections
search
cache
files
derived artifacts
100. Lifecycle Invariants
LI1 — Every data class must have an explicit lifecycle.

LI2 — Every lifecycle state must have defined semantics.

LI3 — Every lifecycle transition must be explicitly permitted.

LI4 — Lifecycle ownership must be unambiguous.

LI5 — Lifecycle policy must be versionable when policy evolution matters.

LI6 — Expiration must not be confused with physical deletion.

LI7 — Derived data must have lifecycle rules compatible with its source.

LI8 — Destructive lifecycle operations must be idempotent or safely retryable.

LI9 — Lifecycle transitions must preserve tenant isolation.

LI10 — Lifecycle operations must respect consistency boundaries.

LI11 — Lifecycle propagation must reach required derived representations.

LI12 — Critical lifecycle transitions must be auditable.

LI13 — Purge operations must pass safety gates.

LI14 — Lifecycle processing must be observable.

LI15 — Lifecycle backlogs must be measurable.

LI16 — Lifecycle failures must be recoverable.

LI17 — Lifecycle operations must support controlled replay or retry.

LI18 — Lifecycle state must not be inferred solely from physical storage location.

LI19 — Recovery must restore lifecycle semantics, not merely bytes.

LI20 — AI-derived data must have an explicit lifecycle.

LI21 — Legal or governance holds must be able to block destructive transitions.

LI22 — Tenant closure must execute a complete lifecycle plan.

LI23 — Rebuildable data may have a different lifecycle from authoritative data.

LI24 — Lifecycle state changes must not silently bypass domain invariants.

LI25 — Irreversible operations must have explicit authorization and policy validation.
101. Lifecycle Control Plane
                     DATA LIFECYCLE CONTROL PLANE
                                  │
          ┌───────────────────────┼──────────────────────┐
          ▼                       ▼                      ▼
      Policies                 Registry              Governance
          │                       │                      │
          ▼                       ▼                      ▼
   State Machines          Data Classification      Holds
          │                       │                      │
          └───────────────────────┼──────────────────────┘
                                  ▼
                         Lifecycle Engine
                                  │
             ┌────────────────────┼────────────────────┐
             ▼                    ▼                    ▼
         Scheduler             Events             Reconciliation
             │                    │                    │
             └────────────────────┼────────────────────┘
                                  ▼
                              Storage
102. Lifecycle Data Plane
Create
  ↓
Activate
  ↓
Use
  ↓
Update
  ↓
Age
  ↓
Expire
  ↓
Archive
  ↓
Restore
  ↓
Delete
  ↓
Purge

No todos los datos recorren todos los estados.

103. Reference Architecture
                         ┌─────────────────────┐
                         │ LIFECYCLE POLICIES  │
                         └──────────┬──────────┘
                                    │
                                    ▼
                         ┌─────────────────────┐
                         │ LIFECYCLE ENGINE    │
                         └──────────┬──────────┘
                                    │
                ┌───────────────────┼───────────────────┐
                ▼                   ▼                   ▼
           Scheduler             Events            Commands
                │                   │                   │
                └───────────────────┼───────────────────┘
                                    ▼
                         ┌─────────────────────┐
                         │  DOMAIN / STORAGE   │
                         └──────────┬──────────┘
                                    │
              ┌─────────────────────┼─────────────────────┐
              ▼                     ▼                     ▼
          Operational            Archive              Derived
             Data                Storage               Data
              │                     │                     │
              └─────────────────────┼─────────────────────┘
                                    ▼
                         ┌─────────────────────┐
                         │   RECONCILIATION    │
                         └──────────┬──────────┘
                                    ▼
                         ┌─────────────────────┐
                         │     OBSERVABILITY    │
                         └─────────────────────┘
104. Lifecycle Completion Criteria

E45 queda completo cuando EVOXA posee:

✓ Lifecycle state model
✓ Lifecycle state machines
✓ Data classification
✓ Lifecycle ownership
✓ Lifecycle metadata
✓ Creation model
✓ Activation model
✓ Aging model
✓ Expiration model
✓ Archive model
✓ Restore model
✓ Delete model
✓ Purge model
✓ Tombstone strategy
✓ Lifecycle policies
✓ Lifecycle automation
✓ Lifecycle engine
✓ Eligibility rules
✓ Time-based transitions
✓ Event-based transitions
✓ Business-state transitions
✓ Lifecycle events
✓ Lifecycle event contract
✓ Idempotent transitions
✓ Concurrency protection
✓ Derived-data lifecycle
✓ Rebuildability classification
✓ Backup interaction
✓ Recovery interaction
✓ Cache lifecycle
✓ Search lifecycle
✓ Analytics lifecycle
✓ AI lifecycle
✓ Agent lifecycle
✓ Temporary-data lifecycle
✓ TTL strategy
✓ Soft delete
✓ Hard delete
✓ Secure deletion
✓ Lifecycle audit
✓ Lifecycle actors
✓ Lifecycle reasons
✓ Lifecycle observability
✓ Lifecycle SLOs
✓ Lifecycle backlog management
✓ Lifecycle failure handling
✓ Retry strategy
✓ Lifecycle DLQ
✓ Lifecycle reconciliation
✓ Lifecycle repair
✓ Purge safety gates
✓ Dependency checks
✓ Cascade controls
✓ Policy versioning
✓ Policy migration
✓ Batch processing
✓ Checkpointing
✓ Throttling
✓ Capacity integration
✓ Resilience integration
✓ Data lineage
✓ Data classification integration
✓ Governance integration
✓ Legal hold support
✓ Tenant deletion lifecycle
✓ Lifecycle invariants
✓ Control plane
✓ Data plane
✓ Reference architecture
105. Principio Rector de E45

El lifecycle de EVOXA debe tratar cada dato como un recurso con una trayectoria explícita: nace bajo una política, adquiere un estado válido, evoluciona dentro de invariantes, envejece según reglas, puede ser archivado o restaurado cuando corresponda y finalmente puede ser eliminado de manera controlada, observable, verificable y segura.

La secuencia arquitectónica queda:

E42 — Backup & Restore
       │
       ▼
E43 — Data Integrity
       │
       │ "Can we trust the data?"
       ▼
E44 — Consistency
       │
       │ "Do distributed representations
       │  converge correctly?"
       ▼
E45 — Data Lifecycle
       │
       │ "How does data live, evolve,
       │  age, archive and disappear?"
       ▼
E46 — Data Retention
       │
       │ "How long must data remain
       │  available and under what rules?"
       ▼
E47 ...

E45 establece así la trayectoria completa del dato dentro de EVOXA; E46 deberá separar y formalizar específicamente la dimensión de retention, evitando mezclarla con lifecycle, archival o purge.
