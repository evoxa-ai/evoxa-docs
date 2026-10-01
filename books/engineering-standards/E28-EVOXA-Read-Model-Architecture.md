E28 — EVOXA Read Model Architecture
Engineering Specification

Pertenece a: Engineering Specification
Volumen: volume-09-engineering
Capítulo: E28 — Read Model Architecture
Anterior: E27 — Query Architecture
Siguiente: E29 — Search Architecture

1. Propósito

E28 define la arquitectura de los Read Models de EVOXA.

Un Read Model es una representación de datos optimizada para lectura, construida para satisfacer las necesidades de uno o más consumidores concretos.

La pregunta fundamental de E28 es:

¿Cómo representamos y almacenamos los datos para que puedan ser consultados de forma eficiente, segura y consistente con el propósito de lectura?

2. Concepto Fundamental
                    SOURCE
                       │
                       ▼
                 Domain State
                       │
                       ▼
                    Events
                       │
                       ▼
               Projection Engine
                       │
                       ▼
                  READ MODEL
                       │
             ┌─────────┼─────────┐
             ▼         ▼         ▼
           Query     Search    Reporting
             │         │         │
             └─────────┼─────────┘
                       ▼
                   Consumer

El Read Model no necesita reproducir la estructura del Domain Model.

Su estructura debe estar optimizada para:

query shape
access pattern
performance
consumer needs
3. Read Model vs Domain Model
Domain Model
    ↓
represents business behavior

Read Model
    ↓
represents read requirements

Por tanto:

Domain Model ≠ Read Model
4. Read Model vs Projection

Projection:

process / derive

Read Model:

store / expose derived read state

Relación:

Source
  ↓
Projection
  ↓
Read Model

Una projection puede existir sin persistencia.

Un Read Model normalmente representa un estado preparado para lectura.

5. Read Model vs Query

Query:

request data

Read Model:

store data optimized for queries

Flujo:

Query
  ↓
Read Model
  ↓
Projection
  ↓
Result
6. Read Model Architecture
                         READ MODEL PLATFORM
                                  │
          ┌───────────────────────┼───────────────────────┐
          ▼                       ▼                       ▼
    Source Events            Source State            External Data
          │                       │                       │
          └───────────────────────┼───────────────────────┘
                                  ▼
                         Projection Engine
                                  │
                                  ▼
                           Read Model Store
                                  │
              ┌───────────────────┼───────────────────┐
              ▼                   ▼                   ▼
            Query              Search              Reports
7. Read Model Characteristics

Un Read Model puede ser:

derived
denormalized
optimized
materialized
versioned
rebuildable
eventually_consistent
consumer_specific

según el caso.

8. Read Model Types

EVOXA debe contemplar:

Entity Read Model
List Read Model
Detail Read Model
Summary Read Model
Search Read Model
Dashboard Read Model
Reporting Read Model
Analytics Read Model
Integration Read Model
Materialized Read Model
Temporal Read Model
9. Entity Read Model

Representa una entidad para lectura:

CustomerReadModel
├── id
├── name
├── status
└── createdAt
10. Summary Read Model

Optimizado para listados o referencias:

CustomerSummary
├── id
├── name
└── status
11. Detail Read Model

Puede contener una representación más completa:

CustomerDetail
├── identity
├── profile
├── address
├── preferences
└── status
12. List Read Model

Optimizado para:

tables
lists
pagination
sorting
filtering

Ejemplo:

CustomerListItem
├── id
├── displayName
├── status
└── updatedAt
13. Search Read Model

Diseñado para:

full-text search
filters
facets
ranking
autocomplete

Puede residir en un motor especializado.

14. Dashboard Read Model

Puede combinar información:

CustomerDashboard
├── customer
├── orderMetrics
├── paymentMetrics
└── activityMetrics

Debe construirse explícitamente para ese caso de lectura.

15. Reporting Read Model

Puede estar optimizado para:

aggregations
historical analysis
grouping
period comparisons
16. Analytics Read Model

Puede residir en:

data warehouse
data lake
analytical database
columnar store

según los requisitos.

17. Integration Read Model

Puede preparar datos para:

external systems
partners
integration APIs
exports

Debe mantener separado el contrato externo del modelo interno cuando corresponda.

18. Materialized Read Model

Un Read Model materializado almacena físicamente el resultado:

Events
  ↓
Projection
  ↓
Materialized Read Model

Esto evita reconstruir el resultado para cada Query.

19. Read Model Store

Puede utilizar:

Relational Database
Document Database
Key-Value Store
Search Engine
Columnar Database
Cache
Data Warehouse

La elección depende del patrón de lectura.

20. Read Store Selection

Debe analizarse:

query pattern
volume
latency
consistency
filtering
sorting
aggregation
retention
rebuild cost
21. Read Model Schema

Cada Read Model debe tener un schema explícito:

ReadModelSchema
├── version
├── fields
├── types
├── indexes
└── constraints
22. Schema Ownership

Debe definirse:

owner
version
consumers
source
projection

para cada Read Model.

23. Read Model Versioning

Ejemplo:

CustomerReadModelV1
CustomerReadModelV2

La versión debe representar cambios contractuales o estructurales significativos.

24. Schema Evolution

Debe soportar:

add field
rename field
remove field
change representation
rebuild
migration

según la compatibilidad requerida.

25. Backward Compatibility

Una nueva versión debe mantener compatibilidad cuando sea posible:

V1 consumer
      ↓
V2 Read Model

Si no es posible:

V1 Read Model
V2 Read Model

pueden coexistir durante la migración.

26. Read Model Rebuild

Uno de los principios centrales:

Source of Truth
      ↓
Replay / Rebuild
      ↓
Read Model

El Read Model no debería convertirse accidentalmente en la única fuente de verdad cuando la arquitectura depende de una fuente primaria distinta.

27. Rebuild Sources

Puede reconstruirse desde:

Domain State
Events
Change Data Capture
External Source
Historical Dataset

Debe documentarse cuál aplica.

28. Event-Based Read Model

En arquitectura event-driven:

Domain
  ↓
Event
  ↓
Projection
  ↓
Read Model

Cada evento puede actualizar el estado derivado.

29. Event Handler

Conceptualmente:

handle(event, readModel)
        ↓
updatedReadModel

Debe ser:

deterministic
testable
idempotent

cuando sea posible.

30. Idempotent Projection

Procesar dos veces:

Event A
Event A

no debe producir:

duplicated state

cuando el modelo requiere idempotencia.

31. Ordering

Cuando el orden sea relevante:

Event 1
Event 2
Event 3

debe preservarse o gestionarse explícitamente.

32. Sequence Tracking

Un Read Model puede almacenar:

lastSequence
lastEventId
lastOffset

para saber hasta dónde ha procesado.

33. Projection Checkpoint
Read Model
├── state
├── projectionVersion
└── checkpoint

El checkpoint permite recuperación incremental.

34. Read Model Freshness

Debe poder determinarse:

sourceVersion
readModelVersion
lag
lastUpdatedAt
35. Eventual Consistency

Muchos Read Models serán:

eventually consistent

Esto debe formar parte explícita del contrato.

36. Strong Consistency

Un Read Model puede requerir consistencia fuerte cuando:

critical read
transactional decision
immediate visibility

lo justifique.

No debe asumirse que todos los Read Models son eventualmente consistentes.

37. Read-After-Write

Debe definirse qué sucede:

Write
 ↓
Event
 ↓
Projection
 ↓
Read

si la proyección aún no ha procesado el evento.

38. Consistency Token

Puede utilizarse:

writeVersion
eventSequence
projectionVersion

para permitir al consumidor solicitar:

read at least version X
39. Read Model Lag

Debe monitorizarse:

projection_lag_seconds

especialmente en sistemas críticos.

40. Read Model Freshness SLA

Puede definirse:

P95 freshness < 2s
P99 freshness < 10s

según el caso de uso.

41. Read Model Storage

Debe separar:

write storage
read storage

cuando la arquitectura CQRS lo requiera.

42. CQRS

E28 es un componente central del patrón CQRS:

             COMMAND
                │
                ▼
          Domain Model
                │
                ▼
             Events
                │
                ▼
          Read Projection
                │
                ▼
           READ MODEL
                │
                ▼
              QUERY
43. Read Model Denormalization

Puede utilizar:

Customer
+
Orders
+
Payments

en una estructura:

CustomerDashboard

para evitar múltiples joins durante la lectura.

44. Denormalization Principle

La desnormalización debe responder a:

known access pattern

No debe hacerse simplemente para copiar estructuras.

45. Read Model Duplication

Es aceptable tener:

CustomerList
CustomerDetail
CustomerSearch
CustomerDashboard

si cada uno responde a un patrón diferente.

46. Read Model Specialization

Una nueva proyección/read model está justificada cuando existe:

different query pattern
different performance requirement
different consumer
different security boundary
different lifecycle
different consistency requirement
47. Read Model Aggregation

Puede almacenar agregados precalculados:

OrderMetrics
├── totalOrders
├── totalRevenue
├── averageOrderValue
└── lastOrderAt

Esto evita recalcularlos en cada Query.

48. Precomputed Data

Debe distinguirse:

derived data

de:

business source of truth

Los datos precalculados no sustituyen automáticamente al modelo canónico.

49. Read Model Indexing

Debe optimizar:

filter
sort
join
lookup
search
aggregation

según el patrón real.

50. Index Design

No crear índices indiscriminadamente.

Cada índice tiene coste de:

storage
write/update
memory
maintenance

aunque el Read Model esté orientado a lectura.

51. Query-Driven Schema

Un Read Model puede diseñarse desde la Query:

Query
 ↓
Access Pattern
 ↓
Read Model Schema
 ↓
Indexes

Este enfoque es preferible a adaptar siempre el modelo de escritura.

52. Read Model Query Optimization

Debe minimizar:

joins
scans
network calls
object hydration
serialization

cuando sea apropiado.

53. Read Model Partitioning

Para grandes volúmenes puede particionarse por:

tenant
region
date
entity range

según el patrón de acceso.

54. Tenant Partitioning

En multi-tenant:

Tenant A → partition A
Tenant B → partition B

puede mejorar:

isolation
performance
operations

pero no sustituye la autorización.

55. Tenant Isolation

Debe garantizarse:

Read Model Query
      ↓
Tenant Scope
      ↓
Authorized Data
56. Read Model Security

Debe evitar:

data leakage
cross-tenant access
sensitive field exposure
internal metadata exposure
57. Field-Level Security

Puede utilizar:

secure projection
field filtering
authorization policy

antes de exponer el Read Model.

58. Read Model vs Authorization
Authorization
    ↓
decides access

Read Model
    ↓
stores / shapes accessible data

El Read Model no sustituye las políticas de autorización.

59. Read Model Caching

Puede existir:

Query
 ↓
Cache
 ↓
Read Model

La estrategia debe alinearse con E17.

60. Cache Invalidation

Cuando cambia el Read Model:

Source Event
 ↓
Projection
 ↓
Read Model Update
 ↓
Cache Invalidation

La invalidación debe ser consistente con el modelo de frescura.

61. Read Model Snapshot

Puede almacenarse un snapshot:

Read Model
   ↓
Snapshot

para acelerar recuperación o rebuild.

62. Snapshot Strategy

Debe definirse:

frequency
retention
format
version
recovery strategy
63. Read Model Recovery

Ante corrupción:

Detect
 ↓
Stop / isolate
 ↓
Rebuild
 ↓
Validate
 ↓
Resume
64. Read Model Reconciliation

Debe ser posible comparar:

Source
   vs
Read Model

cuando la arquitectura requiera detectar drift.

65. Read Model Drift

Puede ocurrir:

Source state ≠ Read state

por:

missed event
processing failure
ordering error
projection bug
manual corruption
66. Drift Detection

Puede utilizar:

counts
checksums
versions
sequence numbers
sample comparison
full reconciliation
67. Read Model Repair

Las estrategias incluyen:

replay
backfill
rebuild
manual correction
migration

La corrección manual debe ser excepcional y auditada.

68. Projection Failure Isolation

Si existe:

Projection A → Read Model A
Projection B → Read Model B

un fallo en A no debería inutilizar B.

69. Read Model Failure Modes
STORE_UNAVAILABLE
PROJECTION_FAILED
SCHEMA_MISMATCH
STALE_DATA
CHECKPOINT_CORRUPTED
REBUILD_FAILED
INDEX_FAILURE
CONSISTENCY_LAG
70. Read Model Resilience

Debe considerar:

replication
failover
retry
backoff
checkpointing
rebuild
fallback

según criticidad.

71. Read Replicas

Para cargas de lectura:

Primary
  │
  ├── Replica 1
  ├── Replica 2
  └── Replica 3

puede mejorar:

read throughput
availability

pero introduce posibles problemas de lag.

72. Read Replica Semantics

Debe conocerse:

replica lag
consistency guarantee
failover behavior
73. Read Model Scaling

Puede escalar:

vertically
horizontally
by partition
by tenant
by workload

según la tecnología.

74. Read Model Capacity

Debe monitorizar:

storage
CPU
memory
IOPS
connections
query throughput
projection throughput
75. Read Model Observability

Métricas mínimas:

read_model_updates_total
read_model_update_errors
read_model_lag
read_model_freshness
read_model_size
read_model_query_latency
read_model_rebuild_duration
76. Read Model Tracing

Debe poder seguirse:

Event
 ↓
Projection
 ↓
Read Model Update
 ↓
Query
 ↓
Consumer

mediante tracing distribuido.

77. Read Model Logging

Debe evitar registrar:

secrets
tokens
passwords
unnecessary PII
full payloads
78. Read Model Audit

Read Models que contienen información sensible pueden requerir:

access audit
query audit
data lineage
retention controls
79. Data Lineage

Debe poder determinarse:

Source
 ↓
Event
 ↓
Projection
 ↓
Read Model
 ↓
Query
 ↓
Consumer

Esto es especialmente importante para:

compliance
debugging
governance
80. Data Retention

Cada Read Model debe declarar:

retention
deletion policy
archival policy
rebuild policy

cuando corresponda.

81. GDPR / Data Deletion Consideration

Cuando se solicita eliminación de datos:

Source
 ↓
Derived Models
 ↓
Caches
 ↓
Search
 ↓
Analytics

debe existir una estrategia para propagar el cambio a las representaciones derivadas aplicables.

82. Read Model Deletion

Eliminar una entidad del Source no implica automáticamente que desaparezca de:

Read Model
Cache
Search Index
Analytics

Debe definirse el flujo de propagación.

83. Tombstones

En arquitecturas event-driven puede utilizarse:

EntityDeleted

o un tombstone para indicar:

remove from read model
84. Soft Delete

Si se utiliza:

deletedAt

debe quedar claro si el Read Model:

hides deleted records

o:

exposes historical deletion state
85. Temporal Read Models

Puede conservar:

current state
historical state
valid-from
valid-to

para consultas temporales.

86. Historical Read Model

Puede construirse a partir de:

events
snapshots
audit records
temporal tables
87. Read Model for Reporting

Puede priorizar:

throughput
aggregation
historical retention
analytical scans

sobre:

transactional latency
88. Read Model for APIs

Puede priorizar:

latency
payload size
contract stability
authorization
89. Read Model for UI

Puede priorizar:

screen-specific shape
low round trips
pagination
sorting
responsive latency
90. Read Model for Search

Puede priorizar:

text indexing
ranking
facets
autocomplete

La arquitectura completa de Search pertenece a E29.

91. Read Model for AI

Cuando EVOXA utilice Read Models para AI/Agents:

Source
 ↓
Read Model
 ↓
AI Query
 ↓
Context Projection

debe aplicar:

authorization
tenant isolation
data minimization
freshness
lineage
92. AI Context Read Model

Puede existir un modelo especializado:

CustomerAIContext

que sólo contiene información necesaria para una operación de IA.

No debe convertirse en una copia indiscriminada del dominio.

93. Read Model and Agent Architecture

Un Agent puede consumir:

Query
 ↓
Read Model
 ↓
Context Projection

pero no debe acceder directamente a storage interno salvo mediante los boundaries definidos.

94. Read Model and API Architecture
API
 ↓
Query
 ↓
Read Model
 ↓
Projection
 ↓
Response

Esto permite desacoplar API y persistencia.

95. Read Model and Integration Architecture

Para integraciones:

Domain
 ↓
Event
 ↓
Integration Read Model
 ↓
External API / Export

El modelo externo debe tener su propio contrato cuando sea necesario.

96. Read Model and Serialization
Read Model
 ↓
Projection
 ↓
DTO
 ↓
Serialization

E23 sigue siendo responsable del formato wire.

97. Read Model and Mapping

Mapping puede convertir:

Read Model
 ↓
External DTO

pero no debe ser utilizado para ocultar diferencias arquitectónicas entre:

read schema
external contract
98. Read Model and Transformation

Transformation puede adaptar:

Read Model
 ↓
Consumer-specific structure

si existe una necesidad semántica real.

99. Read Model and Query Architecture

La relación queda:

E27 QUERY
    │
    ▼
"What do I need?"
    │
    ▼
E28 READ MODEL
    │
    ▼
"How should the data be stored for that need?"
    │
    ▼
E26 PROJECTION
    │
    ▼
"What representation should be exposed?"
100. Read Model Lifecycle
DEFINE
   ↓
DESIGN
   ↓
BUILD
   ↓
BACKFILL
   ↓
VALIDATE
   ↓
SERVE
   ↓
OBSERVE
   ↓
EVOLVE
   ↓
MIGRATE
   ↓
DEPRECATE
   ↓
REMOVE
101. Read Model Creation Criteria

Crear un nuevo Read Model cuando exista:

different access pattern
high query cost
specific latency requirement
specialized filtering
specialized aggregation
different consistency requirement
102. Read Model Avoidance Criteria

No crear un Read Model si:

simple query suffices
cost is negligible
existing model is adequate
maintenance cost exceeds benefit
103. Read Model Cost

Debe evaluarse:

storage
projection compute
rebuild time
operational complexity
monitoring
schema evolution
data duplication
104. Read Model Duplication Tradeoff

La duplicación es aceptable si produce:

lower latency
simpler queries
better scalability
consumer isolation

pero debe controlarse mediante:

ownership
lineage
rebuildability
105. Read Model Governance

Cada modelo debe documentar:

name
purpose
owner
source
projection
store
schema
consumers
consistency
freshness
retention
security
rebuild strategy
106. Read Model Naming

Preferir:

CustomerSummaryReadModel
CustomerDetailReadModel
CustomerSearchReadModel
CustomerDashboardReadModel

o una convención equivalente.

Evitar:

CustomerData2
CustomerViewFinal
TempCustomerRead
107. Read Model Ownership

El owner debe responder por:

schema
performance
freshness
security
rebuild
consumer compatibility
108. Read Model Contract

Cuando un Read Model es consumido directamente, debe tratarse como:

contract

No como:

internal implementation detail
109. Read Model Consumer Registration

Debe conocerse:

consumer
purpose
query
version
SLA

para Read Models críticos.

110. Read Model Migration

Estrategia:

V1
 ↓
V2
 ↓
Backfill
 ↓
Validate
 ↓
Switch consumers
 ↓
Retire V1
111. Dual Write Warning

Evitar:

Write
 ├── Read Model V1
 └── Read Model V2

mediante dos operaciones independientes sin una estrategia de consistencia.

Preferir:

Source Event
 ├── Projection V1
 └── Projection V2

cuando la arquitectura sea event-driven.

112. Shadow Read Model

Puede utilizarse:

Production V1
      │
      └── consumer traffic

Shadow V2
      │
      └── validation only

para comparar:

latency
schema
results
freshness
113. Read Model Blue/Green
Read Model V1
       │
       ├── active
       │
Read Model V2
       │
       └── rebuilding

Después:

V1 → inactive
V2 → active
114. Read Model Canary

Una nueva versión puede exponerse a:

small percentage
specific tenant
internal users
test consumers

antes de generalizarla.

115. Read Model Rebuild Safety

Antes de activar un rebuild:

validate schema
validate source
validate projection version
estimate duration
estimate load
116. Rebuild Isolation

Un rebuild no debería degradar innecesariamente:

production queries
source systems
event processing
117. Rebuild Progress

Debe ser observable mediante:

eventsProcessed
eventsRemaining
percentage
throughput
ETA
errors
checkpoint
118. Read Model Operational States
CREATED
BUILDING
ACTIVE
DEGRADED
STALE
REBUILDING
FAILED
DEPRECATED
RETIRED
119. Read Model Health

Health debe evaluar:

availability
freshness
projection lag
schema compatibility
storage
query latency
120. Definition of Done

E28 queda definido cuando EVOXA dispone de:

✓ Read Model definition
✓ Read Model boundary
✓ Domain vs Read Model separation
✓ Projection relationship
✓ Query relationship
✓ Read Model types
✓ Entity Read Model
✓ Summary Read Model
✓ Detail Read Model
✓ List Read Model
✓ Search Read Model
✓ Dashboard Read Model
✓ Reporting Read Model
✓ Analytics Read Model
✓ Integration Read Model
✓ Materialized Read Model
✓ Read Model Store
✓ Store selection criteria
✓ Schema definition
✓ Ownership
✓ Versioning
✓ Schema evolution
✓ Compatibility
✓ Rebuild
✓ Rebuild sources
✓ Event-based projection
✓ Event handlers
✓ Idempotency
✓ Ordering
✓ Checkpoints
✓ Freshness
✓ Eventual consistency
✓ Strong consistency
✓ Read-after-write
✓ Consistency tokens
✓ Freshness SLA
✓ CQRS integration
✓ Denormalization
✓ Aggregation
✓ Precomputed data
✓ Indexing
✓ Query-driven schema
✓ Partitioning
✓ Tenant isolation
✓ Security
✓ Field-level protection
✓ Caching
✓ Snapshot strategy
✓ Recovery
✓ Reconciliation
✓ Drift detection
✓ Repair
✓ Failure isolation
✓ Resilience
✓ Replicas
✓ Scaling
✓ Capacity management
✓ Observability
✓ Tracing
✓ Logging
✓ Auditing
✓ Data lineage
✓ Retention
✓ Deletion propagation
✓ Tombstones
✓ Soft delete semantics
✓ Temporal models
✓ Historical models
✓ API integration
✓ Search integration
✓ AI integration
✓ Agent integration
✓ Lifecycle
✓ Creation criteria
✓ Avoidance criteria
✓ Cost management
✓ Governance
✓ Naming
✓ Consumer registration
✓ Migration
✓ Shadow models
✓ Blue/green deployment
✓ Canary strategy
✓ Rebuild safety
✓ Operational states
✓ Health model
121. Position in Engineering Specification

La secuencia continúa:

E23 — Serialization Architecture
        ↓
E24 — Transformation Architecture
        ↓
E25 — Mapping Architecture
        ↓
E26 — Projection Architecture
        ↓
E27 — Query Architecture
        ↓
E28 — Read Model Architecture
        ↓
E29 — Search Architecture
        ↓
E30 — Reporting Architecture

La separación queda:

             WRITE SIDE
                 │
                 ▼
             DOMAIN
                 │
                 ▼
              EVENTS
                 │
                 ▼
          ┌──────────────┐
          │  PROJECTION  │
          └──────┬───────┘
                 │
                 ▼
          ┌──────────────┐
          │  READ MODEL  │
          └──────┬───────┘
                 │
          ┌──────┼──────┐
          ▼      ▼      ▼
        QUERY  SEARCH REPORTING
          │      │      │
          └──────┼──────┘
                 ▼
             CONSUMER

Y el mapa conceptual de E24–E29:

E24 Transformation
       │
       │ changes semantic representation
       ▼
E25 Mapping
       │
       │ establishes correspondence
       ▼
E26 Projection
       │
       │ selects / shapes
       ▼
E27 Query
       │
       │ requests / retrieves
       ▼
E28 Read Model
       │
       │ stores optimized read state
       ▼
E29 Search
       │
       │ indexes / retrieves by search semantics
       ▼
   SEARCH CONSUMER

Principio central de E28:
El Read Model es una representación derivada y optimizada para lectura. Debe diseñarse desde los patrones reales de acceso, puede estar desnormalizado y especializado, debe tener una fuente de reconstrucción claramente definida y nunca debe confundirse con la fuente canónica de verdad.

Siguiente: E29 — EVOXA Search Architecture.
