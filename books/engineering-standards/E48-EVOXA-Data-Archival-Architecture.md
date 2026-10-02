E48 — EVOXA Data Archival Architecture
Engineering Specification

Pertenece a: Engineering Specification
Capítulo: E48 — Data Archival Architecture
Anterior: E47 — Data Disposal
Siguiente: E49 — EVOXA Data Migration Architecture

1. Propósito

E48 define cómo EVOXA mueve datos desde almacenamiento operativo hacia almacenamiento de archivo, preservando su integridad, contexto, trazabilidad y capacidad de recuperación durante el período en que esos datos deben conservarse pero ya no necesitan permanecer en el plano operativo.

El principio fundamental es:

Archiving no es deletion, y tampoco es simplemente mover un registro a otro storage. Es una transición controlada del dato desde un estado operativo hacia un estado de conservación de largo plazo.

Por tanto:

Operational Data
      │
      ▼
Archive Eligibility
      │
      ▼
Archive Planning
      │
      ▼
Archive Preparation
      │
      ▼
Archive Transfer
      │
      ▼
Archive Verification
      │
      ▼
Archived
      │
      ├──────────────► Restore / Retrieval
      │
      └──────────────► Disposal when retention expires
2. Boundary

E48 cubre:

Archive Eligibility
Archive Policy
Archive Classification
Archive Planning
Archive Packaging
Archive Serialization
Archive Transfer
Archive Storage
Archive Metadata
Archive Indexing
Archive Integrity
Archive Encryption
Archive Immutability
Archive Retrieval
Archive Restore
Archive Rehydration
Archive Lifecycle
Archive Expiration
Archive Verification
Archive Reconciliation
Archive Failures
Archive Retry
Archive Replication
Archive Audit
Archive Observability

No sustituye:

E42 — Backup & Restore
E45 — Data Lifecycle
E46 — Data Retention
E47 — Data Disposal
E49 — Data Migration
3. Archival vs Backup

La distinción debe ser explícita.

BACKUP
────────────────────────────
Objetivo:
Recovery operacional

Pregunta:
"¿Cómo recuperamos el sistema?"

ARCHIVE
────────────────────────────
Objetivo:
Conservación de datos

Pregunta:
"¿Cómo conservamos datos que ya no son operacionales?"

Por tanto:

Backup ≠ Archive

Un archive puede existir durante años aunque el sistema operativo haya dejado de necesitar esos datos diariamente.

4. Archival vs Disposal

La relación con E47 es:

Operational
    │
    ├────► Archive
    │          │
    │          ▼
    │       Retained
    │          │
    │          ▼
    │       Disposal
    │
    └────► Disposal

Archiving preserva.

Disposal elimina o transforma para impedir su conservación/acceso posterior, según la estrategia definida.

5. Archive Eligibility

Un dato puede ser elegible para archivo cuando:

OperationalNeed = LOW
AND
RetentionRequired = TRUE
AND
ArchivePolicy = ENABLED
AND
NoActiveHoldConflict

Ejemplo:

Customer Transaction
      │
      ├── operationally inactive
      ├── retention still required
      └── archive policy applies
                │
                ▼
          ARCHIVE_ELIGIBLE
6. Eligibility ≠ Archival

Al igual que en E47:

ELIGIBLE

no significa:

ARCHIVED

Existe una transición controlada:

Eligible
   ↓
Planned
   ↓
Approved
   ↓
Archiving
   ↓
Verified
   ↓
Archived
7. Archive State Machine
                  ┌──────────────┐
                  │  OPERATIONAL │
                  └──────┬───────┘
                         │
                         ▼
                ┌─────────────────┐
                │ ARCHIVE ELIGIBLE│
                └────────┬────────┘
                         │
                         ▼
                ┌─────────────────┐
                │ ARCHIVE PLANNED │
                └────────┬────────┘
                         │
                         ▼
                ┌─────────────────┐
                │   ARCHIVING     │
                └────────┬────────┘
                         │
                  ┌──────┴───────┐
                  ▼              ▼
              VERIFIED          FAILED
                  │              │
                  ▼              ▼
              ARCHIVED         RETRY

Estados adicionales:

BLOCKED
CANCELLED
VERIFICATION_FAILED
RESTORING
RESTORED
8. Archive Policy

Toda clase de datos archivables debe estar asociada a una política:

ArchivePolicy
├── policyId
├── policyVersion
├── dataClass
├── eligibilityRule
├── archiveStrategy
├── storageClass
├── encryptionPolicy
├── retentionPolicy
├── retrievalPolicy
├── verificationPolicy
└── expirationPolicy
9. Archive Strategy

EVOXA puede soportar distintas estrategias:

FULL_ARCHIVE
PARTIAL_ARCHIVE
COLD_ARCHIVE
DEEP_ARCHIVE
IMMUTABLE_ARCHIVE
COMPRESSED_ARCHIVE
ENCRYPTED_ARCHIVE

La estrategia concreta depende de:

accessFrequency
retentionPeriod
dataSensitivity
retrievalLatency
cost
compliance
10. Archive Unit

El sistema debe definir qué constituye una unidad archivada.

Puede ser:

record
aggregate
dataset
tenant
time partition
event stream
document
object
snapshot

Ejemplo:

Customer Aggregate
      │
      ├── profile
      ├── transactions
      ├── events
      └── documents
             │
             ▼
        Archive Package
11. Archive Package

Los datos deben poder empaquetarse en una estructura autocontenida:

ArchivePackage
├── archiveId
├── schemaVersion
├── data
├── metadata
├── manifest
├── integrityHash
├── encryptionMetadata
└── provenance
12. Archive Manifest

El manifest debe describir el contenido:

ArchiveManifest
├── archiveId
├── tenantId
├── dataClass
├── entityCount
├── objectCount
├── createdAt
├── archivedAt
├── schemaVersion
├── policyVersion
├── checksum
└── dependencies
13. Provenance

El archivo debe conservar información suficiente para responder:

¿De dónde vino?
¿Cuándo fue archivado?
¿Por qué fue archivado?
¿Con qué política?
¿Con qué versión del schema?

Modelo:

Source
  ↓
Archive
  ↓
Provenance
14. Source Reference

Cada archive debe mantener una referencia al origen lógico:

sourceSystem
sourceEntity
sourceVersion
sourceTimestamp

pero evitando almacenar datos redundantes innecesarios.

15. Schema Versioning

Un archivo debe conservar:

schemaVersion

porque el sistema operativo puede cambiar después del archival.

Operational Schema v8
        ↓
Archive
        ↓
Operational Schema v15

El archive debe seguir siendo interpretable.

16. Archive Format

El formato debe favorecer:

durability
portability
self-description
versionability
integrity
recoverability

El formato concreto pertenece a la implementación tecnológica, pero E48 debe exigir que sea suficientemente estable para el horizonte de retención.

17. Serialization

Antes de archivar:

Domain Object
      ↓
Archive DTO
      ↓
Serialization
      ↓
Archive Package

No debe depender directamente de una representación interna efímera del modelo operativo.

18. Compression

Los archives pueden comprimirse:

Data
 ↓
Serialize
 ↓
Compress
 ↓
Encrypt
 ↓
Store

La compresión debe ser compatible con la estrategia de recuperación.

19. Encryption

Los archives sensibles deben poder cifrarse:

Archive Data
     ↓
Encryption
     ↓
Archive Storage

El archive debe conservar metadatos suficientes para saber:

keyId
algorithm
keyVersion
encryptionVersion

sin exponer claves.

20. Key Lifecycle

El ciclo de vida del archive depende también del ciclo de vida criptográfico:

Archive
  ↓
Encryption Key
  ↓
Key Rotation
  ↓
Key Preservation
  ↓
Archive Expiration
  ↓
Key Destruction

La destrucción prematura de una clave no debe hacer ilegible un archive que todavía deba conservarse.

21. Immutable Archive

Para determinadas clases puede requerirse:

WORM
Immutable Storage
Object Lock
Append-only

El objetivo es:

El archivo no puede modificarse durante el período protegido.

22. Immutability ≠ Encryption

Son controles diferentes:

Encryption
→ protects confidentiality

Immutability
→ protects against modification/deletion

Pueden utilizarse simultáneamente.

23. Archive Storage Tiers

EVOXA puede clasificar almacenamiento:

HOT
WARM
COLD
DEEP_COLD

según:

access frequency
retrieval latency
cost
retention
24. Storage Selection

La selección puede representarse:

High Access
    ↓
WARM

Low Access
    ↓
COLD

Rare Access
    ↓
DEEP_COLD

La política debe controlar la transición.

25. Archive Lifecycle

Un archive puede evolucionar:

Created
  ↓
Warm Archive
  ↓
Cold Archive
  ↓
Deep Archive
  ↓
Expiration
  ↓
Disposal
26. Archive Tiering

El movimiento entre tiers no debe cambiar el significado lógico del archive.

Archive ID
   │
   ├── storage tier A
   ├── storage tier B
   └── storage tier C

El identificador lógico permanece estable.

27. Archive Identity

Cada archive debe poseer un identificador único:

archiveId

y, cuando corresponda:

archiveVersion

Esto permite rastrear movimientos y reemplazos sin perder provenance.

28. Deduplication

Puede existir deduplicación para evitar almacenar múltiples copias idénticas.

Pero debe preservar:

tenant isolation
integrity
reference counting
disposal semantics
29. Multi-Tenant Archiving

Un archive debe mantener aislamiento por tenant.

Tenant A
   └── Archive A

Tenant B
   └── Archive B

Nunca debe permitirse que una recuperación de Tenant A acceda a datos de Tenant B.

30. Tenant Archive Boundary

El scope mínimo debe contener:

tenantId
archiveId
dataClass

La autorización de recuperación debe validar esos límites.

31. Cross-Tenant Archive

Un archive compartido puede existir sólo si la política lo permite y la separación lógica es demostrable.

La optimización de almacenamiento nunca debe degradar el aislamiento.

32. Archive Transfer

El movimiento hacia archive debe ser transaccional a nivel de workflow, aunque no necesariamente ACID entre sistemas.

Prepare
  ↓
Package
  ↓
Upload
  ↓
Verify
  ↓
Commit Archive State
33. Source Deletion Ordering

El source operacional no debe eliminarse antes de confirmar que el archive es válido.

Incorrecto:

Delete Source
     ↓
Create Archive
     ↓
Archive Fails

Correcto:

Create Archive
     ↓
Verify Archive
     ↓
Mark Archived
     ↓
Remove/compact Source if policy allows
34. Archive Commit

El estado lógico debe cambiar a:

ARCHIVED

sólo después de una verificación suficiente.

35. Archive Verification

Debe verificarse al menos:

package exists
manifest exists
checksum matches
metadata is valid
schema is supported
encryption metadata is valid
36. Integrity Hash

Puede utilizarse:

SHA-256

u otro mecanismo criptográfico equivalente definido por la plataforma.

Conceptualmente:

Archive
   ↓
Hash
   ↓
Manifest
37. Integrity Verification

Durante recuperación:

Stored Archive
      ↓
Recalculate Hash
      ↓
Compare Manifest
      ↓
VALID / CORRUPTED
38. Corruption Handling

Si un archive falla integridad:

ARCHIVE_CORRUPTED

Debe buscarse una copia válida:

Replica
Backup
Secondary Archive

según la política.

39. Archive Replication

Los archives críticos pueden replicarse:

Primary Archive
      │
      ├────► Replica A
      └────► Replica B

La replicación debe preservar:

integrity
immutability
encryption
metadata
40. Geographic Replication

Para resiliencia:

Region A
   │
   ▼
Archive
   │
   ├────► Region B
   └────► Region C

Las restricciones regulatorias de localización de datos deben respetarse.

41. Archive Residency

La política puede especificar:

allowedRegions
forbiddenRegions
dataResidency
replicationPolicy
42. Archive Retrieval

La recuperación debe tener dos modos:

READ_ARCHIVE
RESTORE_TO_OPERATIONAL
43. Direct Archive Read

Cuando no sea necesario reintroducir el dato al sistema operativo:

User
 ↓
Archive Query
 ↓
Archive Store
 ↓
Result

Esto evita una restauración completa.

44. Rehydration

Cuando el dato debe volver al plano operativo:

Archive
   ↓
Retrieve
   ↓
Decrypt
   ↓
Decompress
   ↓
Deserialize
   ↓
Validate
   ↓
Transform
   ↓
Rehydrate
45. Restore ≠ Rehydrate

Restore normalmente se asocia a recuperación de estado/sistema.

Rehydrate significa reconstruir una representación operativa a partir del archive.

Archive
   ↓
Rehydration
   ↓
Operational Representation
46. Rehydration Safety

Antes de rehidratar:

validate schema
validate tenant
validate authorization
validate integrity
validate policy
validate conflicts
47. Schema Evolution

Puede existir:

Archived Schema v4
Current Schema v12

La rehidratación requiere:

v4
 ↓
Migration / Transformation
 ↓
v12

La transformación debe ser explícita.

48. Archive Compatibility

Debe existir una matriz conceptual:

Archive Schema
       │
       ▼
Compatibility Layer
       │
       ▼
Current Domain Model
49. Backward Compatibility

El sistema debe poder interpretar archives antiguos durante todo el período en que deban conservarse.

No debe depender exclusivamente de:

current production schema
50. Archive Catalog

EVOXA debe mantener un catálogo lógico:

ArchiveCatalog
├── archiveId
├── tenantId
├── dataClass
├── dateRange
├── schemaVersion
├── storageLocation
├── storageTier
├── status
├── checksum
└── policy
51. Archive Metadata vs Archive Data

Debe distinguirse:

Archive Metadata

de:

Archive Payload

Esto permite localizar un archive sin leer todo su contenido.

52. Archive Index

Puede indexarse:

tenantId
entityId
dataClass
timeRange
archiveId
status

para permitir recuperación eficiente.

53. Metadata Retention

El catálogo del archive puede tener una política de retención diferente al payload.

Por ejemplo:

Payload disposed
Metadata audit retained

si la política lo permite.

54. Archive Search

La búsqueda debe soportar:

tenant
entity
date range
data class
archive status
archive id

pero nunca debe revelar contenido fuera del scope autorizado.

55. Archive Access Control

Operaciones sensibles:

list archive
read archive
download archive
restore archive
delete archive
change archive policy

deben estar separadas por permisos.

56. Archive Download

Cuando se permita exportar un archive:

Authorization
 ↓
Generate Export
 ↓
Audit
 ↓
Time-limited Access

El acceso debe evitar enlaces permanentes.

57. Archive Audit

Debe registrarse:

archiveCreated
archiveMoved
archiveVerified
archiveRetrieved
archiveRestored
archiveFailed
archiveExpired
archiveDisposed
58. Audit Minimization

Los logs no deben copiar innecesariamente el contenido archivado.

Debe conservarse:

metadata
reference
action
actor
timestamp
result
59. Archive Expiration

Cuando termina el período de conservación:

Archive
   ↓
Retention Expired
   ↓
Disposal Eligible
   ↓
E47

La transición es:

E48 Archive
      ↓
E46 Retention
      ↓
E47 Disposal
60. Archive Disposal

El archive mismo debe estar sujeto a disposición.

Archived
   ↓
Retention Ends
   ↓
Disposal
   ↓
Destroyed
61. Archive Hold

Un archive puede estar sujeto a:

LEGAL_HOLD
COMPLIANCE_HOLD
SECURITY_HOLD
INVESTIGATION_HOLD

Mientras exista un hold válido:

Archive Disposal = BLOCKED
62. Hold Release

Cuando el hold desaparece:

Hold Released
      ↓
Reevaluate Retention
      ↓
Disposal Eligibility

No debe asumirse que el archive se elimina automáticamente sin reevaluación.

63. Archive Failure Model

Errores típicos:

ARCHIVE_NOT_ELIGIBLE
ARCHIVE_POLICY_CONFLICT
ARCHIVE_PACKAGE_FAILURE
ARCHIVE_STORAGE_FAILURE
ARCHIVE_TRANSFER_FAILURE
ARCHIVE_VERIFICATION_FAILURE
ARCHIVE_CORRUPTED
ARCHIVE_SCHEMA_UNSUPPORTED
ARCHIVE_KEY_UNAVAILABLE
ARCHIVE_ACCESS_DENIED
ARCHIVE_REHYDRATION_FAILURE
ARCHIVE_RESIDENCY_VIOLATION
64. Retry

Los fallos transitorios:

timeout
network failure
storage unavailable
rate limit

pueden reintentarse.

Los fallos permanentes:

invalid policy
schema unsupported
authorization denied
residency violation

deben bloquear la operación.

65. Archive Idempotency

Si el mismo proceso se ejecuta dos veces:

Archive Request
     ↓
same idempotency key

no debe producir dos archives lógicamente equivalentes.

66. Duplicate Archive Detection

Puede utilizarse:

source identity
source version
policy version
content hash

para detectar duplicados.

67. Partial Archive Failure

Ejemplo:

10 partitions

P1 ✓
P2 ✓
P3 ✓
P4 ✓
P5 ✗
P6 ✓
...

El proceso debe conservar checkpoints.

68. Checkpointing
Archive Job
   │
   ├── Partition 1 ✓
   ├── Partition 2 ✓
   ├── Partition 3 ✓
   └── Partition 4 pending

Permite reanudar sin repetir todo el trabajo.

69. Large Dataset Archiving

Para datasets grandes:

Dataset
  ↓
Partition
  ↓
Batch
  ↓
Package
  ↓
Upload
  ↓
Verify

Debe evitarse cargar todo el dataset en memoria.

70. Streaming Archive

Puede utilizarse:

Source Stream
    ↓
Serializer
    ↓
Compressor
    ↓
Encryptor
    ↓
Archive Writer

para grandes volúmenes.

71. Archive Concurrency

Debe controlarse:

max concurrent archive jobs
max concurrent uploads
max source read rate
max verification rate

para evitar impacto sobre workloads operacionales.

72. Backpressure

Si el archive storage está degradado:

Archive Producer
      ↓
Backpressure
      ↓
Queue

La operación operacional no debe colapsar por una degradación del archive.

73. Archive Queue

Modelo:

Archive Queue
├── priority
├── archiveId
├── tenantId
├── attemptCount
├── scheduledAt
├── status
└── nextRetryAt
74. Priority

Puede existir prioridad:

CRITICAL
HIGH
NORMAL
LOW

según requisitos de compliance y negocio.

75. Archive SLA

Debe definirse:

Eligibility → Archive Completion SLA
Retrieval → Availability SLA
Rehydration → Completion SLA
Verification → Completion SLA
76. Archive Observability

Métricas:

archiveJobs
archivesCompleted
archivesFailed
archivesBlocked
archiveBacklog
archiveLatency
archiveBytes
archiveObjects
verificationFailures
retrievalRequests
retrievalLatency
rehydrationFailures
77. Cost Observability

Debe medirse:

storageCost
retrievalCost
egressCost
compressionRatio
replicationCost

cuando la plataforma lo permita.

78. Archive Health

Un archive puede tener estado:

HEALTHY
DEGRADED
CORRUPTED
UNAVAILABLE
EXPIRED
DISPOSED
79. Archive Reconciliation

Periódicamente:

Archive Catalog
      vs
Archive Storage
      vs
Source State

deben reconciliarse.

Ejemplo:

Catalog says:
ARCHIVED

Storage says:
NOT FOUND

Result:
ARCHIVE_RECONCILIATION_FAILURE
80. Orphan Archives

Si existe un archive sin referencia válida:

Archive
  ↓
No source/catalog reference

debe pasar a:

ORPHAN

y entrar en un workflow de revisión.

Nunca debe eliminarse automáticamente sin aplicar la política correspondiente.

81. Missing Archive

Si el catálogo apunta a un archive inexistente:

MISSING_ARCHIVE

Debe generar una alerta de integridad.

82. Archive Catalog Consistency

El catálogo debe mantener consistencia con:

archive storage
archive status
archive metadata
archive checksum
83. Source Cleanup

Después de archivar correctamente, el source puede:

remain
compact
delete
partition-drop

según E45/E46/E47 y la política de almacenamiento operativo.

84. Archive vs Operational Performance

Una razón principal para archivar es reducir:

table size
index size
query latency
storage cost
operational complexity

Pero el archival nunca debe comprometer la capacidad de recuperar datos retenidos.

85. Partition Archival

Para datasets temporales:

2024 partition → archive
2025 partition → archive
2026 partition → operational

Esto puede ser más eficiente que archivar registro por registro.

86. Time-Based Archiving

Una política puede definirse:

IF record.age > 365 days
AND operationalNeed = LOW
THEN archive

Pero la regla debe evaluar también:

holds
dependencies
policy
tenant
87. Archive Triggers

Los triggers pueden ser:

age
status transition
tenant closure
dataset closure
business lifecycle
storage threshold
compliance rule
manual request
88. Manual Archiving

Debe permitirse cuando sea necesario:

Archive Request

pero siempre sujeto a:

policy
authorization
verification
audit
89. Bulk Tenant Archival

Para cerrar un tenant:

Tenant Closure
      ↓
Inventory
      ↓
Archive Eligible
      ↓
Archive
      ↓
Verify
      ↓
Operational Cleanup
      ↓
Retained Archive
90. Archive Security

Principios:

Least Privilege
Encryption
Tenant Isolation
Integrity
Immutability
Auditability
Controlled Retrieval
Key Management
91. Archive Threat Model

Amenazas principales:

unauthorized retrieval
unauthorized deletion
tampering
corruption
key loss
cross-tenant access
archive resurrection
catalog inconsistency
storage loss
schema obsolescence
92. Archive Integrity Invariant

Un archive marcado como válido debe corresponder exactamente al contenido cuya integridad fue verificada.

93. Archive Immutability Invariant

Si una política exige inmutabilidad, ningún actor ni proceso operativo debe poder modificar o eliminar el contenido durante el período protegido.

94. Archive Retrieval Invariant

Una solicitud de recuperación nunca puede ampliar el scope de autorización del solicitante.

95. Archive Tenant Invariant

Un archive perteneciente a Tenant A nunca puede ser recuperado como parte de una operación autorizada únicamente para Tenant B.

96. Archive Lifecycle Invariant
Operational
    ↓
Archived
    ↓
Retained
    ↓
Disposal Eligible
    ↓
Disposed

No debe existir:

Operational
   ↓
Disposed

si la política exige conservación mediante archive.

97. Archive Recovery Invariant

La recuperación debe ser capaz de detectar un archive corrupto antes de incorporarlo nuevamente al plano operativo.

98. Archive Schema Invariant

La conservación a largo plazo no puede depender de que el schema operativo actual permanezca sin cambios.

99. Archive Reconciliation Invariant

Toda discrepancia entre catálogo, storage y estado lógico del archive debe permanecer visible hasta su resolución.

100. Reference Architecture
                         ┌───────────────────────┐
                         │ E45 DATA LIFECYCLE     │
                         └───────────┬───────────┘
                                     │
                                     ▼
                         ┌───────────────────────┐
                         │ E46 RETENTION         │
                         └───────────┬───────────┘
                                     │
                              Archive Eligible
                                     │
                                     ▼
                    ┌────────────────────────────────┐
                    │      ARCHIVE CONTROLLER        │
                    └───────────────┬────────────────┘
                                    │
                ┌───────────────────┼───────────────────┐
                ▼                   ▼                   ▼
            Policy              Planning           Authorization
                │                   │                   │
                └───────────────────┼───────────────────┘
                                    ▼
                    ┌────────────────────────────────┐
                    │       ARCHIVE ORCHESTRATOR      │
                    └───────────────┬────────────────┘
                                    │
               ┌────────────────────┼────────────────────┐
               ▼                    ▼                    ▼
           Packaging            Storage              Metadata
               │                    │                    │
               └────────────────────┼────────────────────┘
                                    ▼
                         ┌───────────────────────┐
                         │ VERIFICATION ENGINE   │
                         └───────────┬───────────┘
                                     │
                            ┌────────┴────────┐
                            ▼                 ▼
                         SUCCESS            FAILURE
                            │                 │
                            ▼                 ▼
                         ARCHIVED            RETRY
                            │
                            ▼
                     Retrieval / Restore
                            │
                            ▼
                     Retention Expiry
                            │
                            ▼
                         E47 DISPOSAL
101. Archive Control Plane
              ARCHIVE CONTROL PLANE

Policies ───────────┐
Retention ──────────┤
Governance ─────────┤
Authorization ──────┤
Residency ──────────┤
                    ▼
             Archive Controller
                    │
         ┌──────────┴──────────┐
         ▼                     ▼
      Planning             Verification
102. Archive Data Plane
              ARCHIVE DATA PLANE

Operational Data
      │
      ▼
Serialization
      │
      ▼
Compression
      │
      ▼
Encryption
      │
      ▼
Archive Storage
      │
      ├── Replica
      ├── Cold Tier
      └── Deep Tier
103. Archive Retrieval Plane
Request
   │
   ▼
Authorization
   │
   ▼
Archive Catalog
   │
   ▼
Archive Storage
   │
   ▼
Integrity Check
   │
   ▼
Decrypt / Decompress
   │
   ▼
Deserialize
   │
   ├────────────► Direct Read
   │
   └────────────► Rehydrate
104. Archive Domain Model

Conceptualmente:

Archive
├── archiveId
├── tenantId
├── dataClass
├── sourceReference
├── archiveStatus
├── archiveStrategy
├── storageTier
├── schemaVersion
├── policyVersion
├── createdAt
├── archivedAt
├── expiresAt
├── checksum
├── encryptionMetadata
├── residency
└── provenance
105. Archive Job
ArchiveJob
├── jobId
├── archiveId
├── tenantId
├── status
├── priority
├── attemptCount
├── checkpoint
├── startedAt
├── completedAt
└── lastError
106. Archive Retrieval Request
ArchiveRetrieval
├── requestId
├── archiveId
├── tenantId
├── requester
├── purpose
├── accessMode
├── authorization
├── requestedAt
├── status
└── result
107. Archive Events

Eventos principales:

ArchiveRequested
ArchiveEligible
ArchivePlanned
ArchiveStarted
ArchiveStored
ArchiveVerified
ArchiveCompleted
ArchiveFailed
ArchiveRetrieved
ArchiveRehydrated
ArchiveCorrupted
ArchiveExpired
ArchiveDisposed
108. Event Contract
eventId
archiveId
tenantId
dataClass
policyId
policyVersion
schemaVersion
occurredAt
correlationId
result

Debe evitarse incluir payload sensible innecesario.

109. API Surface

Conceptualmente:

POST /archives
GET  /archives/{archiveId}
GET  /archives
POST /archives/{archiveId}/retrieve
POST /archives/{archiveId}/rehydrate
POST /archives/{archiveId}/verify
POST /archives/{archiveId}/retry

Las APIs concretas pertenecen a E03.

110. Archive Governance

Debe definirse:

who can archive
who can retrieve
who can rehydrate
who can modify policy
who can expire
who can dispose
who audits
111. Separation of Duties

Para archives de alto impacto:

Archive Requester
       ≠
Archive Approver

cuando la política lo requiera.

Asimismo:

Archive Retrieval
       ≠
Archive Disposal

puede requerir privilegios separados.

112. Emergency Retrieval

Puede existir recuperación de emergencia:

Emergency Request
      ↓
Elevated Authorization
      ↓
Audit
      ↓
Retrieval
      ↓
Review

Debe mantenerse completamente auditable.

113. Disaster Recovery

El archive puede formar parte de la estrategia de recuperación a largo plazo, pero:

Archive ≠ Disaster Recovery

E41 define DR.

E48 define la conservación y recuperación de datos archivados.

114. Archive Monitoring

Debe existir monitorización de:

archive failures
archive corruption
missing archives
retrieval failures
key failures
catalog drift
replication lag
storage availability
115. Periodic Integrity Scan

Los archives de larga duración pueden requerir:

Periodic Integrity Verification

para detectar:

bit rot
storage corruption
metadata corruption
replica divergence
116. Integrity Repair

Si existe una replica válida:

Corrupted Archive
       ↓
Healthy Replica
       ↓
Restore
       ↓
Verify

El repair debe quedar auditado.

117. Archive Cost Management

La arquitectura debe permitir optimizar:

storage tier
compression
deduplication
replication count
retrieval frequency

sin violar:

retention
availability
integrity
security
residency
118. Operational Isolation

El archive workload no debe consumir recursos críticos del plano operacional.

Debe existir aislamiento de:

CPU
memory
IO
network
database connections
queues
workers

cuando el volumen lo justifique.

119. Archive Batch Scheduling

Los trabajos pueden ejecutarse durante:

off-peak windows
scheduled windows
low-load periods

si la política operacional lo requiere.

120. Archive Backpressure

Si el source produce datos archivables más rápido que la capacidad de archive:

Eligible Data
     ↓
Archive Queue
     ↓
Backlog

Debe existir capacidad para:

throttling
prioritization
capacity scaling
alerting
121. Completion Criteria

E48 se considera completo cuando EVOXA dispone de:

✓ Archive boundary
✓ Archive eligibility
✓ Archive policy
✓ Archive state machine
✓ Archive strategies
✓ Archive packaging
✓ Archive manifest
✓ Provenance
✓ Schema versioning
✓ Serialization
✓ Compression
✓ Encryption
✓ Key lifecycle
✓ Immutability
✓ Storage tiers
✓ Archive lifecycle
✓ Tiering
✓ Archive identity
✓ Replication
✓ Residency
✓ Multi-tenant isolation
✓ Transfer workflow
✓ Archive commit
✓ Integrity verification
✓ Corruption handling
✓ Archive catalog
✓ Archive indexing
✓ Retrieval
✓ Rehydration
✓ Schema evolution
✓ Compatibility
✓ Archive expiration
✓ Archive disposal
✓ Hold handling
✓ Failure model
✓ Retry
✓ Idempotency
✓ Checkpointing
✓ Large dataset support
✓ Streaming
✓ Backpressure
✓ SLA
✓ Observability
✓ Reconciliation
✓ Security
✓ Governance
✓ Audit
✓ Reference architecture
✓ Domain model
✓ Event model
✓ API model
✓ Architectural invariants
122. Relación con E45–E47

La cadena completa debe quedar:

E45 — DATA LIFECYCLE
        │
        ├── Operational
        │
        ▼
E46 — DATA RETENTION
        │
        ├── retain operationally
        │
        ├── archive eligible
        │
        ▼
E48 — DATA ARCHIVAL
        │
        ├── archived
        │
        ├── retrieved / rehydrated
        │
        ▼
E46 — RETENTION EXPIRY
        │
        ▼
E47 — DATA DISPOSAL

Pero también existe un camino directo:

E46
 │
 └── no archive required
          │
          ▼
        E47

Por tanto:

Retention
    │
    ├──────────────► Disposal
    │
    └──────────────► Archive
                         │
                         ▼
                      Disposal
123. Principio Rector de E48

Archiving es la transición controlada de datos fuera del plano operativo hacia un medio de conservación de largo plazo, manteniendo identidad, provenance, integridad, seguridad, contexto semántico y capacidad de recuperación durante todo el período de retención.

La arquitectura resultante:

E45 — Data Lifecycle
        ↓
E46 — Data Retention
        ↓
   ┌────┴────┐
   │         │
   ▼         ▼
 E48       E47
Archive   Disposal
   │
   ▼
Long-Term Retention
   │
   ▼
Retention Expiry
   │
   ▼
E47 — Disposal

E47 responde “cómo dejamos de conservarlo”; E48 responde “cómo lo conservamos fuera del plano operativo hasta que llegue ese momento”.
