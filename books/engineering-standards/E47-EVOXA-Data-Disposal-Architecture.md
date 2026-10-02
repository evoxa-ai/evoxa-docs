E47 — EVOXA Data Disposal Architecture
Engineering Specification

Pertenece a: Engineering Specification
Capítulo: E47 — Data Disposal Architecture
Anterior: E46 — Data Retention
Siguiente: E48 — EVOXA Data Archival Architecture

1. Propósito

E47 define cómo EVOXA determina, autoriza, ejecuta, verifica y audita la disposición de datos cuando éstos dejan de estar sujetos a retención.

El principio fundamental es:

Retention determina cuándo un dato puede dejar de conservarse; Disposal determina cómo se elimina, destruye, anonimiza o invalida de forma controlada.

Por tanto:

Retention ≠ Disposal

La relación es:

Retention Policy
      │
      ▼
Retention Requirement Satisfied
      │
      ▼
Disposal Eligibility
      │
      ▼
Disposal Authorization
      │
      ▼
Disposal Execution
      │
      ▼
Verification
      │
      ▼
Disposal Completed
2. Disposal Boundary

E47 cubre:

Disposal Eligibility
Disposal Planning
Disposal Authorization
Disposal Requests
Disposal Jobs
Disposal Strategies
Logical Deletion
Physical Deletion
Permanent Purge
Anonymization
Redaction
Cryptographic Destruction
Derived Data Disposal
Replica Disposal
Search Index Disposal
Cache Disposal
AI Artifact Disposal
Disposal Verification
Disposal Audit
Disposal Reconciliation
Disposal Failures
Disposal Retry
Disposal Idempotency
Disposal Safety

No define por sí mismo:

Retention policy definition
Backup implementation
Storage engine implementation
Database implementation
Infrastructure provisioning
3. Core Principle

Un dato no debe eliminarse simplemente porque:

createdAt + N days < now

Debe cumplirse:

RetentionSatisfied
AND
NoActiveHold
AND
NoHigherOrderConstraint
AND
DisposalAuthorized
AND
DependenciesResolved
4. Disposal Eligibility

E46 produce:

DISPOSAL_ELIGIBLE

E47 consume esa decisión.

Modelo:

             E46
              │
              ▼
     ┌───────────────────┐
     │ Disposal Eligible │
     └─────────┬─────────┘
               │
               ▼
             E47
               │
       ┌───────┴────────┐
       ▼                ▼
   DISPOSE           BLOCK
5. Eligibility ≠ Authorization

Es importante distinguir:

Eligible

de:

Authorized

Un registro puede ser elegible para disposición pero requerir:

approval
review
dependency resolution
legal verification

antes de ejecutarse.

6. Disposal State Machine
                  ┌──────────────┐
                  │   RETAINED   │
                  └──────┬───────┘
                         │
                         ▼
               ┌───────────────────┐
               │ DISPOSAL ELIGIBLE │
               └─────────┬─────────┘
                         │
                         ▼
                ┌──────────────────┐
                │ PENDING APPROVAL │
                └────────┬─────────┘
                         │
                    approved
                         │
                         ▼
                ┌──────────────────┐
                │   DISPOSING      │
                └────────┬─────────┘
                         │
              ┌──────────┴──────────┐
              ▼                     ▼
          COMPLETED                FAILED
                                    │
                                    ▼
                                  RETRY

También puede existir:

BLOCKED
CANCELLED
REQUIRES_REVIEW
VERIFICATION_FAILED
7. Disposal Strategies

No todos los datos deben eliminarse de la misma manera.

EVOXA puede utilizar:

DELETE
PURGE
ANONYMIZE
REDACT
CRYPTO_ERASE
INVALIDATE
DETACH

La estrategia debe venir determinada por la clase de dato y su política.

8. Logical Deletion

Logical deletion significa que el dato deja de estar disponible para las operaciones normales, por ejemplo:

deleted = true

o:

status = DISPOSED

Esto no equivale necesariamente a destrucción.

9. Physical Deletion

Physical deletion implica eliminar físicamente el contenido de su almacenamiento primario.

Logical Delete
      ↓
Physical Delete

Puede ser necesario ejecutar ambos.

10. Permanent Purge

PURGE significa eliminar definitivamente los datos que ya no deben permanecer accesibles.

Debe ser una operación explícita y controlada.

11. Anonymization

Cuando la política lo permita, un registro puede transformarse:

PII
 ↓
Anonymized Data

La anonimización debe impedir razonablemente la recuperación de la identidad original.

No debe confundirse con:

masking
pseudonymization
encryption
12. Redaction

Redaction elimina únicamente determinados campos:

Customer
├── id
├── name       → removed
├── email      → removed
└── aggregate  → retained

Debe utilizarse sólo cuando la política permita conservar el resto del registro.

13. Cryptographic Destruction

Para determinados datos cifrados puede utilizarse:

Data
  +
Encryption Key
      ↓
Key Destruction
      ↓
Data Becomes Unrecoverable

Esto requiere garantías específicas sobre:

key ownership
key replicas
key backups
key versions
key recovery
14. Disposal Strategy Selection

La estrategia puede depender de:

dataClass
sensitivity
storageType
retentionPolicy
legalRequirement
tenantPolicy
dependencyModel

Ejemplo:

Operational Record → PURGE
PII → REDACT / PURGE
Analytics Aggregate → ANONYMIZE
Encrypted Archive → CRYPTO_ERASE
Cache → INVALIDATE
15. Disposal Scope

Una disposición debe tener un scope explícito:

entity
record
aggregate
tenant
data class
dataset
artifact

Nunca debe asumirse que:

delete entity

significa:

delete everything related to entity

sin una política de dependencias.

16. Disposal Graph

Los datos relacionados pueden representarse como:

                 Entity
                    │
        ┌───────────┼───────────┐
        ▼           ▼           ▼
    Read Model   Search       Cache
        │           │           │
        ▼           ▼           ▼
    Analytics    Vector      Derived AI

La disposición debe recorrer las dependencias correspondientes.

17. Authoritative Data

Debe identificarse la fuente de autoridad:

Authoritative Record

antes de ejecutar disposal.

La eliminación de una proyección no debe eliminar accidentalmente el source of truth.

18. Derived Data

Las copias derivadas deben evaluarse explícitamente:

Read Model
Search Index
Cache
Analytics Dataset
Materialized View
Embedding
Agent Memory
AI Artifact
19. Derived Data Disposal

Modelo:

Authoritative Data
       │
       ├────► Read Model
       ├────► Search
       ├────► Cache
       ├────► Analytics
       └────► AI Artifacts

Disposal debe determinar cuáles deben eliminarse, invalidarse o conservarse bajo una política independiente.

20. Search Disposal

Si un registro se elimina:

Source Record
     ↓
Search Index

el índice debe dejar de exponerlo.

Puede utilizar:

delete
tombstone
reindex
segment cleanup

según la implementación.

21. Cache Disposal

Una cache puede requerir:

invalidate
evict
expire

No debe considerarse suficiente eliminar únicamente el registro primario si el dato sigue accesible mediante cache.

22. Read Model Disposal

Los read models deben mantenerse coherentes con la disposición del source.

Modelo:

Source Deleted
      ↓
Deletion Event
      ↓
Read Model Removal
23. Analytics Disposal

Los datos analíticos requieren especial cuidado porque pueden haber sido:

aggregated
denormalized
joined
derived

La disposición debe respetar la política correspondiente.

24. AI Artifact Disposal

EVOXA debe considerar:

Embeddings
Vector Store Entries
Agent Memory
Prompt Artifacts
Generated Documents
Evaluation Artifacts
Model Feedback
AI-derived Profiles

cuando estén vinculados al dato eliminado.

25. AI Memory Disposal

Si un dato alimentó memoria persistente de un agente:

Source Data
    ↓
Agent Memory

el disposal puede requerir eliminar o invalidar también:

memory entry
embedding
retrieval metadata
references

cuando así lo determine la política.

26. Tenant Disposal

La eliminación de un tenant debe utilizar un proceso controlado:

Tenant Closure
      ↓
Inventory
      ↓
Retention Evaluation
      ↓
Hold Evaluation
      ↓
Dependency Discovery
      ↓
Disposal Plan
      ↓
Execution
      ↓
Verification
27. Disposal Plan

Antes de ejecutar operaciones masivas debe generarse un plan:

DisposalPlan
├── planId
├── tenantId
├── scope
├── policy
├── strategy
├── dependencies
├── estimatedRecords
├── estimatedArtifacts
├── approval
└── executionWindow
28. Dry Run

Debe existir capacidad de:

DRY_RUN

para responder:

What would be deleted?

sin ejecutar la eliminación.

29. Disposal Preview

Ejemplo:

Records:
12,481

Search entries:
12,481

Cache entries:
8,204

AI embeddings:
12,481

Analytics records:
3,100

Blocked:
17

Esto permite detectar errores antes de ejecutar.

30. Approval Model

Para determinadas categorías:

Disposal Request
      ↓
Review
      ↓
Approval
      ↓
Execution

No todos los datos requieren aprobación humana; el nivel debe depender del riesgo.

31. High-Risk Disposal

Puede requerir aprobación explícita:

tenant-wide disposal
regulated data
financial records
security evidence
legal data
large datasets
irreversible destruction
32. Disposal Authorization

Una autorización debería contener:

authorizationId
scope
strategy
policy
requester
approver
reason
createdAt
expiresAt
33. Authorization Expiry

Una autorización debe poder expirar:

Authorization
    ↓
expiresAt
    ↓
No longer executable

Esto evita que autorizaciones antiguas permanezcan reutilizables indefinidamente.

34. Disposal Command

Conceptualmente:

DisposeData

con:

disposalId
entityId
scope
strategy
authorization
idempotencyKey
requestedAt
35. Idempotency

El mismo disposal puede ser reintentado:

attempt 1 → timeout
attempt 2 → retry
attempt 3 → success

El resultado final debe seguir siendo:

DISPOSED

y no producir corrupción.

36. Partial Failure

Un disposal distribuido puede fallar parcialmente:

Primary      ✓
Search       ✓
Cache        ✓
Analytics    ✗
AI Memory    ✗

El sistema debe registrar el estado por dependencia.

37. Disposal Transaction Boundary

No debe asumirse que todo el disposal puede ejecutarse en una única transacción ACID.

Para sistemas distribuidos:

Orchestration
+
Idempotency
+
Compensation
+
Verification

son necesarios.

38. Disposal Workflow
Request
  ↓
Validate
  ↓
Authorize
  ↓
Discover Dependencies
  ↓
Create Plan
  ↓
Execute
  ↓
Verify
  ↓
Reconcile
  ↓
Complete
39. Disposal Verification

La eliminación debe verificarse.

No basta con:

DELETE returned 200

Debe comprobarse:

primary absent
derived copies absent
search unavailable
cache invalidated
AI artifacts handled

según el scope.

40. Verification Levels

Puede existir:

LEVEL_1 — Primary Store
LEVEL_2 — Derived Stores
LEVEL_3 — Search / Cache
LEVEL_4 — AI Artifacts
LEVEL_5 — Backup / Archive Constraints
41. Disposal Completion

Un disposal puede marcarse:

COMPLETED

sólo cuando se hayan satisfecho los requisitos de verificación definidos por su política.

42. Verification Failure

Si falla:

Verification

el estado debe ser:

VERIFICATION_FAILED

y no:

COMPLETED
43. Disposal Reconciliation

Debe poder compararse:

Expected Disposal State
        vs
Actual Data State

Ejemplo:

Expected:
No search record

Actual:
Search record exists

Result:
DISPOSAL_RECONCILIATION_FAILURE
44. Tombstones

En sistemas event-driven puede ser necesario publicar:

EntityDeleted

o:

EntityDisposed

para permitir que las proyecciones eliminen sus copias.

45. Tombstone Semantics

Un tombstone no es necesariamente el dato eliminado.

Es una señal:

"This entity must no longer exist in this projection."

Debe tener lifecycle propio.

46. Eventual Consistency

Después del disposal puede existir una ventana:

Primary → deleted
Search → pending deletion
Cache → pending invalidation

La arquitectura debe definir el SLA máximo aceptable para esa convergencia.

47. Disposal Consistency

El objetivo no es necesariamente:

instantaneous global deletion

sino:

bounded eventual disposal consistency

cuando el sistema sea distribuido.

48. Disposal Queue

Puede existir:

Disposal Queue

con:

priority
scheduledAt
attemptCount
lastError
nextRetryAt
status
49. Retry Policy

Los fallos transitorios pueden reintentarse:

retry
backoff
jitter
maxAttempts
dead-letter

Los errores permanentes deben detener el proceso.

50. Dead-Letter Disposal

Un disposal que no puede ejecutarse después de los retries debe pasar a:

DISPOSAL_DLQ

y generar una alerta.

Nunca debe simplemente desaparecer de la cola.

51. Disposal Safety

Antes de eliminar:

Validate Identity
Validate Tenant
Validate Scope
Validate Policy
Validate Authorization
Validate Hold
Validate Dependencies
Validate Idempotency
52. Tenant Isolation

Toda operación debe comprobar:

requestedTenantId
actualEntityTenantId
authorizationTenantScope

para evitar cross-tenant disposal.

53. Authorization Bypass Prevention

No debe existir un endpoint genérico que permita:

DELETE /anything

sin evaluación de:

policy
scope
authorization
tenant
54. Bulk Disposal

Para grandes volúmenes:

Bulk Request
   ↓
Partition
   ↓
Batch
   ↓
Execute
   ↓
Checkpoint
   ↓
Verify

Debe limitarse el tamaño de cada batch.

55. Rate Limiting

El disposal no debe saturar:

database
search
object storage
event bus
AI systems

Debe soportar:

rate limit
concurrency limit
backpressure
56. Disposal Ordering

Cuando existen dependencias:

Derived
   ↓
References
   ↓
Authoritative

o el orden inverso, según la implementación.

El orden debe estar definido por el dependency graph.

57. Referential Integrity

Antes de eliminar una entidad debe evaluarse:

foreign references
active workflows
pending jobs
events
subscriptions
projections
58. Active Workflow Dependency

Si un workflow todavía referencia un dato:

Entity
  ↑
Workflow

el disposal debe determinar si:

cancel
detach
migrate
retain

es la acción correcta.

59. Pending Jobs

Los jobs pendientes pueden contener referencias al dato.

Debe existir:

Job Cancellation

o:

Reference Invalidated

antes o durante el disposal.

60. Eventual Event Consumers

Consumers que reciban eventos antiguos pueden intentar recrear datos eliminados.

La arquitectura debe prevenir:

Delete
   ↓
Late Event
   ↓
Recreate Deleted Record

mediante:

tombstones
version checks
event ordering
disposal markers
61. Disposal Marker

Puede existir:

Disposal Marker

que indique:

entityId
disposedAt
disposalVersion

y permita rechazar recreaciones inválidas.

62. Resurrection Prevention

Principio:

Un dato disposed no debe reaparecer por un evento, retry, replay o reconstrucción accidental.

Esto es especialmente importante en sistemas event-sourced.

63. Event Sourcing

Si EVOXA utiliza event sourcing:

Entity State
     ↑
Event Stream

el disposal debe definir cómo tratar:

historical events
snapshots
projections
indexes

No basta con borrar el current state.

64. Event Stream Disposal

Puede requerirse:

event tombstone
redaction
crypto erasure
stream retirement

según la política.

65. Snapshot Disposal

Los snapshots que contengan datos eliminados deben:

delete
rebuild
invalidate

según corresponda.

66. Audit Data

Un punto crítico:

Business Data

no necesariamente tiene la misma política que:

Audit Data

El disposal debe respetar la política de auditoría aplicable.

67. Disposal Audit Record

Incluso después de eliminar un dato, debe poder conservarse evidencia mínima de que la eliminación ocurrió:

disposalId
entityReference
policy
strategy
timestamp
result
operator

sin conservar innecesariamente el contenido eliminado.

68. Minimal Disposal Evidence

Principio:

La evidencia de disposición debe demostrar que ocurrió sin convertirse en una copia del dato dispuesto.

69. Security Evidence

Los registros de seguridad pueden estar sujetos a retention independiente.

No deben eliminarse automáticamente sólo porque el objeto operativo asociado fue disposed.

70. Backup Interaction

La eliminación del primary store no implica automáticamente que el dato desaparezca de backups existentes.

E47 debe distinguir:

Primary Disposal

de:

Backup Expiration

E42 gobierna el segundo.

71. Backup Recovery Problem

Debe evitarse:

Primary disposed
      ↓
Backup restored
      ↓
Disposed data returns

Si esto puede ocurrir, el proceso de restore debe incluir mecanismos de:

disposal tombstones
reconciliation
post-restore disposal
72. Archive Interaction

Si un registro está archivado:

Archive
   ↓
Disposal Eligible

el disposal debe alcanzar también el archive cuando éste esté dentro del scope.

73. Object Storage Disposal

Los objetos pueden requerir:

object delete
version delete
multipart cleanup
metadata delete
index removal

especialmente en sistemas con versionado.

74. Storage Versioning

Un simple:

DELETE object

puede no ser suficiente si existen versiones históricas.

La política debe especificar cómo tratar:

object versions
snapshots
soft deletes
replicas
75. Search and Vector Stores

La disposición de AI/search puede requerir:

document deletion
vector deletion
metadata deletion
index refresh
cache invalidation
76. Disposal Observability

Métricas:

disposalRequests
disposalsCompleted
disposalsFailed
disposalsBlocked
disposalsRetried
verificationFailures
disposalLatency
disposalBacklog
disposalAge
77. Disposal Compliance Metrics
Eligible-to-Disposed SLA
Disposal Completion Rate
Verification Success Rate
Overdue Disposal Count
Failed Disposal Count
Resurrection Count
78. Alerts

Alertas importantes:

disposal backlog high
verification failure
cross-tenant mismatch
retention/disposal conflict
repeated failure
resurrection detected
backup restoration reintroduced disposed data
79. Disposal Logging

Cada operación crítica debe generar logs estructurados:

disposalId
entityId
tenantId
strategy
stage
attempt
result
errorCode
timestamp
correlationId
80. Error Taxonomy

Ejemplos:

DISPOSAL_NOT_ELIGIBLE
DISPOSAL_NOT_AUTHORIZED
ACTIVE_HOLD
DEPENDENCY_BLOCKED
TENANT_SCOPE_MISMATCH
POLICY_CONFLICT
STORAGE_FAILURE
SEARCH_FAILURE
CACHE_FAILURE
AI_ARTIFACT_FAILURE
VERIFICATION_FAILURE
RECONCILIATION_FAILURE
81. Permanent vs Transient Errors

Transient:

timeout
temporary unavailable
network failure
rate limit

Permanent:

not authorized
active hold
invalid scope
policy conflict
unknown entity

El retry debe depender de esta clasificación.

82. Disposal API

Conceptualmente:

POST /disposals
GET  /disposals/{disposalId}
POST /disposals/{disposalId}/approve
POST /disposals/{disposalId}/cancel
POST /disposals/{disposalId}/retry
GET  /disposals/{disposalId}/verification

La API definitiva pertenece a E03.

83. Disposal Events

Eventos:

DisposalRequested
DisposalAuthorized
DisposalStarted
DisposalCompleted
DisposalFailed
DisposalBlocked
DisposalVerificationFailed
DisposalReconciled
84. Disposal Event Contract
eventId
disposalId
entityId
tenantId
scope
strategy
policyId
policyVersion
occurredAt
correlationId
result

No debe incluir contenido sensible innecesario.

85. Disposal Security

Disposal es una operación de alto impacto.

Debe aplicar:

least privilege
separation of duties
strong authorization
auditability
tenant isolation
approval controls
86. Separation of Duties

Para operaciones críticas:

Requester ≠ Approver

cuando governance lo requiera.

87. Emergency Disposal

Puede existir un mecanismo de emergencia para:

security incident
data exposure
critical compliance requirement

pero debe mantener:

authorization
audit
scope
verification

incluso bajo presión operacional.

88. Disposal Governance

Las políticas de disposal deben definir:

who may request
who may approve
who may execute
who may verify
who audits
89. Disposal Plan Example
DISPOSAL PLAN
────────────────────────────────
Plan: DP-2026-00123
Tenant: T-001
Scope: Customer-8472

Strategy:
PURGE

Dependencies:
✓ Primary
✓ Search
✓ Cache
✓ Read Model
✓ AI Memory

Blocked:
0

Approval:
APPROVED

Execution:
2026-10-01T10:00Z
90. Disposal Result Example
DISPOSAL RESULT
────────────────────────────────
Disposal: D-99128

Primary:       COMPLETED
Read Model:    COMPLETED
Search:        COMPLETED
Cache:         COMPLETED
AI Memory:     COMPLETED
Analytics:     COMPLETED

Verification:  PASSED

Status:
COMPLETED
91. Disposal Failure Example
Primary:       COMPLETED
Search:        COMPLETED
Cache:         COMPLETED
Analytics:     FAILED
AI Memory:     PENDING

Status:
PARTIAL_FAILURE

Action:
RETRY
92. Disposal Reconciliation Example
Expected:
Search record absent

Observed:
Search record present

Result:
RECONCILIATION_FAILURE

Action:
REINDEX / DELETE
93. Reference Architecture
                         ┌───────────────────────┐
                         │ E46 RETENTION         │
                         │ DISPOSAL ELIGIBILITY  │
                         └───────────┬───────────┘
                                     │
                                     ▼
                         ┌───────────────────────┐
                         │ DISPOSAL CONTROLLER   │
                         └───────────┬───────────┘
                                     │
                   ┌─────────────────┼─────────────────┐
                   ▼                 ▼                 ▼
              Authorization      Planning          Holds
                   │                 │                 │
                   └─────────────────┼─────────────────┘
                                     ▼
                         ┌───────────────────────┐
                         │ DISPOSAL ORCHESTRATOR │
                         └───────────┬───────────┘
                                     │
          ┌──────────────┬───────────┼───────────┬─────────────┐
          ▼              ▼           ▼           ▼             ▼
       Primary         Search       Cache      Analytics       AI
          │              │           │           │             │
          └──────────────┴───────────┼───────────┴─────────────┘
                                     ▼
                         ┌───────────────────────┐
                         │ VERIFICATION ENGINE   │
                         └───────────┬───────────┘
                                     ▼
                         ┌───────────────────────┐
                         │ RECONCILIATION        │
                         └───────────┬───────────┘
                                     ▼
                              COMPLETED / FAILED
94. Control Plane
                 DISPOSAL CONTROL PLANE

 Policies ────────┐
 Governance ──────┤
 Authorization ───┤
 Holds ───────────┤
                  ▼
          ┌───────────────┐
          │ Disposal      │
          │ Controller    │
          └───────┬───────┘
                  │
          ┌───────┴────────┐
          ▼                ▼
      Planning        Verification
95. Data Plane
                 DISPOSAL DATA PLANE

                 Disposal Command
                        │
                        ▼
                  Target Dataset
                        │
         ┌──────────────┼──────────────┐
         ▼              ▼              ▼
      Primary        Derived        Artifacts
         │              │              │
         └──────────────┼──────────────┘
                        ▼
                  Disposed State
96. Disposal Invariants
DI1 — Disposal cannot occur before retention requirements are satisfied.

DI2 — Active holds must block disposal.

DI3 — Disposal authorization must be explicit where required.

DI4 — Disposal scope must be deterministic.

DI5 — Tenant boundaries must never be crossed.

DI6 — Disposal operations must be idempotent.

DI7 — Partial failure must be recoverable.

DI8 — Disposal must be observable.

DI9 — Disposal must be auditable.

DI10 — Disposal completion must require verification.

DI11 — Derived data must be handled according to explicit policy.

DI12 — Search indexes must not continue exposing disposed data.

DI13 — Caches must not continue exposing disposed data beyond the permitted window.

DI14 — AI artifacts must be handled when they are within disposal scope.

DI15 — Disposal must prevent accidental data resurrection.

DI16 — Event replay must not recreate disposed entities.

DI17 — Backup restoration must not silently bypass disposal requirements.

DI18 — Logical deletion must not be represented as physical destruction unless it actually is.

DI19 — Disposal failures must remain visible until resolved.

DI20 — Retry must not create inconsistent disposal state.

DI21 — High-risk disposal must support stronger authorization.

DI22 — Disposal evidence must not unnecessarily recreate disposed content.

DI23 — Bulk disposal must support checkpointing and recovery.

DI24 — Disposal execution must respect dependency ordering.

DI25 — Verification failure must prevent false completion.

DI26 — Emergency disposal must remain auditable.

DI27 — Disposal must fail safely when scope or policy is ambiguous.

DI28 — A disposed entity must not be resurrected by stale asynchronous operations.

DI29 — Disposal policy changes must be versioned.

DI30 — Disposal completion must be independently reconcilable.
97. Completion Criteria

E47 queda completo cuando EVOXA dispone de:

✓ Disposal boundary
✓ Disposal eligibility
✓ Authorization model
✓ Disposal planning
✓ Dry-run capability
✓ Disposal strategies
✓ Logical deletion
✓ Physical deletion
✓ Permanent purge
✓ Anonymization
✓ Redaction
✓ Cryptographic destruction
✓ Dependency graph
✓ Derived-data disposal
✓ Search disposal
✓ Cache disposal
✓ Read-model disposal
✓ Analytics disposal
✓ AI artifact disposal
✓ Tenant disposal
✓ Bulk disposal
✓ Disposal orchestration
✓ Idempotency
✓ Retry
✓ Partial-failure handling
✓ Dead-letter handling
✓ Verification
✓ Reconciliation
✓ Tombstones
✓ Resurrection prevention
✓ Event-sourcing handling
✓ Backup interaction
✓ Archive interaction
✓ Storage version handling
✓ Security controls
✓ Tenant isolation
✓ Governance
✓ Audit trail
✓ Observability
✓ Metrics
✓ Alerts
✓ Error taxonomy
✓ API model
✓ Event model
✓ Reference architecture
✓ Disposal invariants
98. Relación con E46

La frontera entre ambos documentos debe mantenerse estricta:

E46 — DATA RETENTION
────────────────────────────
¿Debe seguir conservándose?
¿Cuánto tiempo?
¿Existe un hold?
¿Se alcanzó el retention end?
¿Es disposal eligible?
E47 — DATA DISPOSAL
────────────────────────────
¿Está autorizado eliminarlo?
¿Qué estrategia usar?
¿Qué dependencias existen?
¿Cómo ejecutar la disposición?
¿Cómo verificarla?
¿Cómo evitar su resurrección?
99. Relación con E42
E42 — Backup & Restore
        │
        │ backup retention
        │ restore behavior
        ▼
E47 — Data Disposal
        │
        │ disposed state
        │ tombstones
        │ post-restore reconciliation
        ▼
Restored System

El restore nunca debe ignorar disposals previamente autorizados.

100. Relación con E43/E44
E43 Data Integrity
        │
        ▼
E44 Consistency
        │
        ▼
E47 Disposal
        │
        ├── integrity preserved
        ├── references reconciled
        └── distributed state converges
101. Relación con E45/E46
E45 — Data Lifecycle
        │
        │ data evolution
        ▼
E46 — Data Retention
        │
        │ retention requirement
        ▼
E47 — Data Disposal
        │
        │ controlled destruction
        ▼
Disposed
102. Principio Rector de E47

La disposición de datos es una operación irreversible o potencialmente irreversible de alto impacto. EVOXA sólo debe ejecutarla cuando la elegibilidad, autorización, scope, dependencias y restricciones hayan sido verificadas; debe ejecutarla de forma idempotente, observable y resistente a fallos, y debe demostrar posteriormente que el dato y sus derivados incluidos en el scope dejaron de estar disponibles.

La secuencia arquitectónica queda:

E43 — Data Integrity
        ↓
E44 — Data Consistency
        ↓
E45 — Data Lifecycle
        ↓
E46 — Data Retention
        ↓
E47 — Data Disposal
        ↓
E48 — Data Archival

E46 responde “¿cuándo puede dejar de conservarse?”; E47 responde “¿cómo lo eliminamos de manera segura y verificable?”.
