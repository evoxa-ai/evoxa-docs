E52 — EVOXA Data Federation Architecture
Engineering Specification

Pertenece a: Engineering Specification
Capítulo: E52 — Data Federation Architecture
Anterior: E51 — Data Replication Architecture
Siguiente: E53 — Data Virtualization Architecture

1. Propósito

E52 define la arquitectura mediante la cual EVOXA puede acceder, consultar, combinar y presentar datos distribuidos entre múltiples sistemas y dominios sin requerir que esos datos sean físicamente consolidados en un único datastore.

El principio fundamental es:

Data Federation unifica el acceso lógico a datos distribuidos sin exigir que exista una única ubicación física para esos datos.

Mientras E51 mantiene copias redundantes:

Primary
   │
   ├──► Replica A
   ├──► Replica B
   └──► Replica C

E52 compone fuentes independientes:

Source A ──┐
Source B ──┼──► Federation Layer ──► Unified Result
Source C ──┘
2. Federation vs Replication

Esta distinción es crítica.

Replication
Dataset
   │
   ├──► Copy A
   ├──► Copy B
   └──► Copy C

Objetivo:

redundancy
availability
durability
read scaling
Federation
System A ──┐
System B ──┼──► Federated Access
System C ──┘

Objetivo:

unified access
cross-source queries
composition
logical integration

Por tanto:

Replication duplica datos; Federation compone datos.

3. Federation vs Synchronization
Synchronization
A ─────► B

produce alineamiento entre estados.

Federation
A ──┐
B ──┼──► Query / Composition
C ──┘

no requiere necesariamente copiar datos.

4. Federation Boundary

E52 cubre:

Federation Gateway
Source Connectors
Federated Query
Source Discovery
Schema Discovery
Logical Schema
Source Routing
Query Planning
Query Decomposition
Pushdown
Join Strategies
Aggregation
Filtering
Projection
Pagination
Federated Transactions
Consistency
Failure Handling
Source Isolation
Security
Authorization
Data Residency
Caching
Observability
Performance
Cost Control

No sustituye:

E25 — Mapping
E26 — Projection
E27 — Query
E28 — Read Model
E29 — Search
E30 — Reporting
E31 — Analytics
E50 — Data Synchronization
E51 — Data Replication
5. Core Model

La arquitectura lógica:

                         ┌─────────────────────┐
                         │   Client / Service  │
                         └──────────┬──────────┘
                                    │
                                    ▼
                         ┌─────────────────────┐
                         │ Federation Gateway  │
                         └──────────┬──────────┘
                                    │
                         ┌──────────▼──────────┐
                         │ Federated Query     │
                         │ Planner / Executor  │
                         └──────────┬──────────┘
                                    │
              ┌─────────────────────┼─────────────────────┐
              ▼                     ▼                     ▼
        ┌───────────┐         ┌───────────┐         ┌───────────┐
        │ Source A  │         │ Source B  │         │ Source C  │
        │ Database  │         │ API       │         │ Service   │
        └───────────┘         └───────────┘         └───────────┘
6. Federation Source

Una fuente federada es cualquier sistema autorizado que expone datos consumibles por EVOXA.

Ejemplos conceptuales:

SQL Database
NoSQL Database
External API
Internal Service
Data Warehouse
Object Storage
Search Engine
Event Store
Partner System
7. Source Registration

Cada fuente debe registrarse mediante una definición explícita:

FederationSource
├── sourceId
├── name
├── type
├── endpoint
├── capabilities
├── schema
├── ownership
├── securityPolicy
├── consistencyPolicy
├── availabilityPolicy
└── status
8. Source Identity

Cada source debe tener un identificador estable:

sourceId

Ejemplo:

CUSTOMER_DB
ORDER_SERVICE
PAYMENT_SYSTEM
ANALYTICS_WAREHOUSE
PARTNER_CRM
9. Source Ownership

Debe conocerse:

owner
domain
responsibleTeam

La federation layer no debe convertirse en un sistema sin ownership claro.

10. Source Capabilities

Cada source debe declarar sus capacidades:

supportsFiltering
supportsProjection
supportsSorting
supportsPagination
supportsAggregation
supportsJoin
supportsTransactions
supportsStreaming
supportsPushdown

El planner utilizará estas capacidades para construir el plan.

11. Logical Federation Schema

EVOXA debe proporcionar un modelo lógico independiente de la ubicación física:

Customer
Order
Payment
Product
Subscription

aunque físicamente:

Customer → CRM
Order → Order DB
Payment → Payment Service
Product → Catalog DB
12. Logical-to-Physical Mapping
Logical Entity
      │
      ▼
Federation Mapping
      │
 ┌────┼────┐
 ▼    ▼    ▼
Source A B Source C

El mapping define:

logical field
source field
conversion
identity
availability
13. Federated Query

Ejemplo conceptual:

Find customers
who have orders
and successful payments

Puede implicar:

Customer DB
      │
      ├── Customer ID
      │
Order DB
      │
      ├── Customer ID
      │
Payment System
      │
      └── Customer ID
14. Query Decomposition

Una consulta federada:

Q

puede descomponerse:

Q
├── Q1 → Customer DB
├── Q2 → Order DB
└── Q3 → Payment Service

Después:

Q1
Q2
Q3
 │
 ▼
Federation Join
 │
 ▼
Final Result
15. Query Planner

El planner determina:

source selection
execution order
filter pushdown
projection pushdown
join strategy
parallelism
aggregation strategy
timeouts
retry policy
16. Query Plan

Conceptualmente:

Federated Query
      │
      ▼
Parse
      │
      ▼
Logical Plan
      │
      ▼
Source Resolution
      │
      ▼
Optimization
      │
      ▼
Physical Plan
      │
      ▼
Execution
      │
      ▼
Merge
17. Logical Plan

Ejemplo:

Join
├── Filter(Customer)
└── Filter(Order)
18. Physical Plan

Puede convertirse en:

Customer DB
   │
Filter country = ES
   │
   ▼
Customer IDs
   │
   └──────────────┐
                  ▼
              Order DB
                  │
             customer_id IN (...)
                  │
                  ▼
                Orders
19. Filter Pushdown

Siempre que sea posible:

Federation Layer
       │
       ▼
Filter
       │
       ▼
Source

en lugar de:

Source
  │
  ▼
All Data
  │
  ▼
Federation Filter

Esto reduce:

network
memory
latency
source load
20. Projection Pushdown

Si sólo se necesitan:

customerId
name

no debe solicitarse:

customerId
name
address
phone
history
preferences
...

si la fuente permite projection.

21. Aggregation Pushdown

Una operación:

COUNT(orders)

puede ejecutarse directamente en el source:

Order DB
   │
   ▼
COUNT
   │
   ▼
Federation

en lugar de transportar todos los orders.

22. Join Strategies

EVOXA puede utilizar:

Nested Loop
Hash Join
Merge Join
Broadcast Join
Semi Join
Source-side Join

según las capacidades y cardinalidades.

23. Local Join
Source A ──► Result A
Source B ──► Result B
                    │
                    ▼
                Local Join

La federation layer realiza el join.

24. Source-Side Join

Si una fuente puede resolver la relación:

Source A
   │
   └── JOIN Source B

el planner puede delegar la operación.

25. Broadcast Join

Cuando un dataset es pequeño:

Small Dataset
     │
     ├──► Source A
     ├──► Source B
     └──► Source C

puede utilizarse para reducir transferencias.

26. Semi-Join

Primero se obtienen claves:

Source A
   │
   ▼
IDs
   │
   ▼
Source B

y sólo se recuperan registros relevantes.

27. Cardinality Estimation

El planner debe estimar:

rows
bytes
selectivity
join cardinality
latency

para elegir un plan eficiente.

28. Query Cost

Cada source puede tener un coste:

CPU
network
API calls
database load
rate limits
monetary cost

El planner debe considerar estos costes cuando sean conocidos.

29. API Federation

Una fuente puede ser un API:

Federation
    │
    ▼
GET /customers
    │
    ▼
Partner System

El federation layer debe gestionar:

authentication
rate limits
pagination
timeouts
retries
schema conversion
30. Service Federation

Una fuente puede ser un microservicio:

Federation
   │
   ▼
Customer Service

Debe respetarse su contrato público.

La federation layer no debe depender de tablas internas del servicio si el contrato arquitectónico establece APIs como boundary.

31. Database Federation

Puede acceder a:

PostgreSQL
MySQL
SQL Server
NoSQL
Warehouse

mediante adapters apropiados.

32. Object Storage Federation

Puede exponer datasets desde:

Object Storage
    │
    ▼
Parquet / CSV / JSON

cuando el volumen y el patrón de acceso lo justifiquen.

33. Search Federation

Una fuente puede ser un search engine:

Federated Query
      │
      ├── Database
      └── Search Index

La arquitectura debe distinguir:

authoritative data
derived search data
34. Authority Model

Cada dato debe tener una fuente de autoridad:

Customer.email
      │
      ▼
CRM = authoritative

mientras que:

Search Index

puede ser sólo una proyección.

35. Source-of-Truth

Federation no debe ocultar:

source of truth

El resultado debe poder rastrearse conceptualmente hasta su origen.

36. Data Lineage

Cada campo federado debe poder asociarse a:

source
source field
transformation
timestamp/version

cuando la trazabilidad sea necesaria.

37. Provenance

Un resultado puede llevar:

provenance
├── sourceId
├── sourceVersion
├── retrievedAt
└── transformation

Esto es especialmente importante para:

analytics
compliance
AI
decisioning
audit
38. Consistency

Una federated query puede leer:

Customer @ T1
Order    @ T2
Payment  @ T3

Por tanto:

Federated reads no deben asumir automáticamente snapshot consistency global.

39. Consistency Modes

EVOXA puede definir:

BEST_EFFORT
EVENTUAL
BOUNDED_STALENESS
SOURCE_CONSISTENT
COORDINATED

según el caso de uso.

40. Temporal Consistency

Cuando sea necesario:

asOfTimestamp

puede solicitarse para obtener datos conceptualmente alineados a un instante.

No todas las fuentes podrán soportarlo.

41. Cross-Source Transactions

Federation no implica automáticamente transacciones distribuidas.

Una consulta:

A + B + C

no significa que:

A
B
C

formen una única transacción ACID.

42. Federated Writes

Los writes federados son considerablemente más complejos.

Preferencia arquitectónica:

Federation
   │
   └──► Read / Composition

y:

Domain Service
   │
   └──► Authoritative Write
43. Write Ownership

Si un request modifica:

Customer
Order
Payment

cada dominio debe conservar ownership sobre su propia mutación.

44. Distributed Transactions

Si son estrictamente necesarias:

Coordinator
   │
   ├──► Source A
   ├──► Source B
   └──► Source C

debe existir una estrategia explícita:

2PC
Saga
Compensation
Outbox
Workflow

La federation layer no debe asumir responsabilidad transaccional implícita.

45. Failure Isolation

Si:

Source A ✓
Source B ✓
Source C ✗

EVOXA debe poder decidir:

fail entire query
return partial result
fallback
use cache

según la política.

46. Partial Results

No debe devolverse silenciosamente un resultado incompleto como si fuese completo.

Si se permiten partial results:

result
├── data
├── sourceStatus
├── completeness
└── warnings
47. Source Availability

Cada source puede encontrarse:

AVAILABLE
DEGRADED
TIMEOUT
UNAVAILABLE
UNKNOWN
48. Timeout Budget

Una federated query debe tener:

global timeout
per-source timeout
join timeout

Ejemplo:

Global = 2s

Source A = 500ms
Source B = 700ms
Source C = 500ms
49. Cancellation

Si el cliente cancela:

Client
  X
  │
  ▼
Federation Query
  │
  ├──► cancel A
  ├──► cancel B
  └──► cancel C

los trabajos downstream deben cancelarse cuando sea posible.

50. Retry Policy

No todas las operaciones deben reintentarse.

El retry debe considerar:

idempotency
error type
timeout
rate limit
source health
query cost
51. Retry Storm Prevention

Debe evitarse:

Source failure
   ↓
Many federated queries
   ↓
Automatic retries
   ↓
More source load
   ↓
Source collapse

Utilizando:

backoff
jitter
circuit breakers
bulkheads
52. Circuit Breaker
Source
  │
  ▼
Failures ↑
  │
  ▼
OPEN
  │
  X
  │
Recovery
  │
  ▼
HALF_OPEN
  │
  ▼
CLOSED
53. Bulkhead Isolation

Los sources deben estar aislados:

Source A Pool
Source B Pool
Source C Pool

para evitar que un source lento consuma todos los recursos de federation.

54. Concurrency Limits

Cada source puede tener:

maxConcurrentQueries
maxConnections
maxRequests

para proteger tanto EVOXA como el sistema remoto.

55. Rate Limits

Para APIs externas:

requests / second
requests / minute

deben formar parte de la política del adapter.

56. Caching

Federation puede utilizar:

source metadata cache
schema cache
query result cache
reference data cache

pero la política de freshness debe ser explícita.

57. Cache Correctness

No debe asumirse:

cache = source of truth

La cache representa:

derived state

salvo que la arquitectura defina lo contrario.

58. Schema Registry

Una federation layer puede mantener:

Federated Schema Registry

conteniendo:

entities
fields
types
source mappings
capabilities
versions
59. Schema Evolution

Si Source A cambia:

customer_name

a:

full_name

la federation mapping puede mantener temporalmente:

logical.name → source.full_name

sin cambiar inmediatamente el contrato lógico.

60. Schema Versioning

Debe soportarse:

Schema V1
Schema V2

cuando múltiples consumers no puedan migrarse simultáneamente.

61. Type Normalization

Diferentes sources pueden representar:

DATE
TIMESTAMP
STRING
UUID
DECIMAL
BOOLEAN

de forma diferente.

La federation layer debe normalizar al modelo lógico.

62. Null Semantics

Debe definirse cómo se interpreta:

NULL
missing
empty
unknown
not applicable

entre fuentes heterogéneas.

63. Identifier Mapping

Puede existir:

CRM Customer ID = C123
Order Customer ID = 84721

El federation mapping debe resolver:

Logical Customer ID

sin confundir identidades.

64. Cross-Source Joins

Los joins entre fuentes requieren:

join key
identity mapping
type compatibility
cardinality

La ausencia de una clave fiable debe ser una condición explícita.

65. Data Quality

Federation no debe ocultar diferencias como:

duplicate customers
invalid identifiers
inconsistent currencies
missing values
stale records

Puede exponerlas mediante metadata de calidad.

66. Currency and Units

Cuando distintas fuentes utilizan:

USD
EUR
GBP

o:

kg
lb

la normalización debe ser explícita.

67. Time Zones

Las fuentes pueden utilizar:

UTC
local timezone
offset timestamps

La federated model debe definir una convención.

Preferencia:

UTC internally

cuando sea compatible con el dominio.

68. Security Boundary

Federation no debe convertirse en un bypass de seguridad.

Cada query debe respetar:

authentication
authorization
tenant isolation
field security
source policy
data classification
69. Authorization

La autorización debe poder evaluarse en:

Federation Layer
       +
Source

No debe asumirse que autorizar al usuario para EVOXA automáticamente autoriza todos los sources.

70. Row-Level Security

Puede existir:

User A → Customer rows 1-100
User B → Customer rows 101-200

La federation engine debe preservar la política.

71. Field-Level Security

Ejemplo:

Customer
├── id ✓
├── name ✓
├── email ✓
└── ssn ✗

La federated result no debe filtrar el campo prohibido sólo después de haberlo expuesto internamente a un componente no autorizado.

72. Tenant Isolation
Tenant A
   │
   ▼
Federation Query
   │
   X Tenant B

El tenant boundary debe propagarse a todos los source queries.

73. Query Context

Una consulta federada debe transportar contexto:

tenantId
userId
roles
permissions
correlationId
consistencyRequirement
deadline
dataResidency

según las necesidades.

74. Data Residency Routing

Si un tenant sólo puede consultar datos en:

EU

el planner debe evitar:

EU → US Source

cuando ello viole la política.

75. Sensitive Data

Los datos sensibles deben tener políticas explícitas sobre:

source access
transport
logging
caching
result composition
audit
76. Observability

Cada federated query debe generar:

queryId
correlationId
source timings
source status
rows returned
bytes transferred
plan
cache hits
retries
errors

según clasificación y seguridad.

77. Query Trace

Conceptualmente:

Query Q123
│
├── Parse          5ms
├── Plan           8ms
├── Source A      30ms
├── Source B     120ms
├── Source C      40ms
├── Join           8ms
└── Serialize      3ms

Esto permite identificar el bottleneck real.

78. Source Metrics

Por source:

requestCount
successRate
errorRate
p95Latency
p99Latency
timeoutRate
retryRate
rowsRead
bytesRead
79. Federation Metrics

Globalmente:

federatedQueries
queryLatency
queryFailures
partialResults
sourceFailures
joinFailures
plannerFailures
cacheHitRate
80. Query Logging

Debe evitarse registrar indiscriminadamente:

PII
secrets
credentials
sensitive query parameters
raw sensitive results

Los logs deben seguir E14/E06/E43 y las políticas de seguridad correspondientes.

81. Query Explain

EVOXA debe poder generar un plan explicable:

SOURCE A
  Filter: country = ES
  Projection: id,name

SOURCE B
  Filter: customer_id IN (...)

LOCAL JOIN
  customer.id = order.customer_id

Esto facilita:

debugging
optimization
capacity planning
82. Performance Optimization

Las principales optimizaciones:

filter pushdown
projection pushdown
aggregation pushdown
join reordering
parallel execution
result caching
metadata caching
connection pooling
batching
83. Parallel Execution

Cuando no existen dependencias:

Q1 ─────► Source A
Q2 ─────► Source B
Q3 ─────► Source C

deben ejecutarse en paralelo.

84. Dependency Graph

El planner puede representar:

Q1 ──────┐
         ├──► Join ───► Result
Q2 ──────┘

y ejecutar nodos independientes simultáneamente.

85. Pagination

La paginación federada es compleja.

No debe suponerse que:

LIMIT 20

en cada source equivale a:

global LIMIT 20

después de un join o sort.

86. Global Ordering

Para:

ORDER BY createdAt
LIMIT 20

el planner puede necesitar obtener suficientes candidatos de cada source antes de producir el resultado final.

87. Streaming Federation

Para grandes resultados:

Source A ──┐
Source B ──┼──► Streaming Merge
Source C ──┘

evita cargar todo el dataset en memoria.

88. Backpressure

Si el consumer procesa lentamente:

Federation
   │
   ▼
Consumer
   │
   ▼
Slow

debe existir backpressure hacia los source adapters.

89. Memory Protection

Las queries federadas deben tener límites:

maxRows
maxBytes
maxJoinMemory
maxResultSize
maxExecutionTime
90. Query Admission Control

Una query excesivamente costosa puede:

reject
queue
throttle
degrade

en lugar de poner en riesgo todo el sistema.

91. Federation Query Classes

Puede clasificarse:

INTERACTIVE
BACKGROUND
ANALYTICS
REPORTING
AI
ADMINISTRATIVE

Cada clase puede tener:

timeout
priority
resource limits

distintos.

92. AI Federation

Para EVOXA AI:

AI Agent
   │
   ▼
Federation Layer
   │
   ├── Customer
   ├── Orders
   ├── Inventory
   └── Knowledge

La federation layer debe controlar:

authorization
source provenance
freshness
PII
query scope
93. Decision Federation

Para Decision Architecture:

Decision
   │
   ▼
Federated Query
   │
   ├── Customer State
   ├── Account State
   ├── Risk State
   └── Transaction State

Debe conservarse provenance suficiente para justificar la decisión.

94. Federation and Intelligence

E32 Intelligence puede consumir:

Federated Data

pero no debe asumir que los datos tienen:

same freshness
same authority
same consistency
95. Federation and Analytics

Para analytics:

Source A
Source B
Source C
   │
   ▼
Federated Dataset
   │
   ▼
Analytics

Para cargas masivas y repetitivas puede ser preferible materializar un dataset en:

Data Warehouse
Data Lake
Read Model

en lugar de ejecutar federation continuamente.

96. Federation vs Materialization
Federation
Query-time composition

Ventaja:

freshness
no duplication

Coste:

runtime latency
source dependency
Materialization
Build-time composition

Ventaja:

fast reads
predictable performance

Coste:

storage
staleness
pipeline complexity
97. Hybrid Architecture

EVOXA puede utilizar:

                   ┌── Federation ──► Fresh operational data
Client ────────────┤
                   └── Read Model ───► High-performance queries

La elección debe depender del workload.

98. Federation Cache

La cache puede existir por:

source
entity
query
reference data
schema
metadata

Debe existir una política explícita de:

TTL
invalidation
staleness
scope
tenant
99. Source Discovery

La federation layer debe conocer:

available sources
source capabilities
health
schema
version
ownership

Esto puede gestionarse mediante registry.

100. Source Lifecycle
REGISTERED
    ↓
VALIDATING
    ↓
ACTIVE
    ↓
DEGRADED
    ↓
DRAINING
    ↓
DISABLED
    ↓
REMOVED
101. Source Registration Validation

Antes de activar un source:

connectivity
authentication
schema
capabilities
latency
authorization
data classification

deben validarse.

102. Adapter Architecture
                  Federation Engine
                         │
              ┌──────────┼──────────┐
              ▼          ▼          ▼
          SQL Adapter API Adapter Object Adapter
              │          │          │
              ▼          ▼          ▼
          Database     Service     Storage
103. Adapter Contract

Cada adapter debe proporcionar conceptualmente:

connect()
describeSchema()
getCapabilities()
execute()
stream()
health()
close()

según el tipo de source.

104. Adapter Isolation

Un fallo de un adapter no debe:

crash federation engine

Debe estar aislado mediante:

timeouts
bulkheads
circuit breakers
resource limits
105. Query Result Model

Un resultado federado puede tener:

FederatedResult
├── data
├── sourceMetadata
├── completeness
├── warnings
├── executionMetadata
└── provenance
106. Error Model

Los errores deben distinguir:

SOURCE_UNAVAILABLE
SOURCE_TIMEOUT
SOURCE_UNAUTHORIZED
SCHEMA_MISMATCH
QUERY_UNSUPPORTED
QUERY_TIMEOUT
JOIN_FAILURE
RESOURCE_LIMIT
DATA_QUALITY_FAILURE
POLICY_DENIED
107. Unsupported Operations

Si un source no soporta:

aggregation
join
sorting

el planner debe decidir si:

execute locally
rewrite query
use another source
reject query
108. Capability Negotiation

Antes de ejecutar:

Planner
   │
   ▼
Capabilities
   │
   ├── supportsFilter ✓
   ├── supportsSort ✓
   ├── supportsJoin ✗
   └── supportsAggregate ✓

el plan se adapta.

109. Schema Drift

Si la fuente cambia inesperadamente:

Expected:
customer_id

Actual:
customerId

la query no debe fallar silenciosamente ni devolver datos incorrectos.

Debe generarse:

schema drift event

y activar la política correspondiente.

110. Federation Testing

Debe cubrir:

source availability
schema drift
latency
partial failure
timeouts
authorization
tenant isolation
large joins
source throttling
duplicate data
missing data
inconsistent data
111. Chaos Testing

Escenarios:

kill source
network partition
high latency
rate limit
schema change
partial response
connection exhaustion
corrupt metadata
112. Contract Testing

Cada adapter debe probar:

schema contract
capability contract
authentication contract
error contract
pagination contract
consistency contract
113. Performance Testing

Benchmarks:

1 source
2 sources
5 sources
10 sources
100 sources

con:

small result
medium result
large result
high cardinality joins
114. Security Testing

Debe comprobarse:

tenant escape
field leakage
source privilege escalation
cross-region violation
PII leakage
cache isolation
log leakage
115. Federation Audit

Auditar:

source registration
source removal
mapping changes
schema changes
policy changes
query access
privileged queries
administrative operations
116. Federation Governance

Cada federated source debe tener:

owner
classification
allowed consumers
allowed regions
retention policy
availability expectation
cost boundary
117. Cost Governance

Una federated query puede generar costes en múltiples sistemas:

Source A cost
Source B cost
Source C cost
Network cost
Compute cost

El sistema debe poder identificar el consumo por:

tenant
application
query
team

cuando sea necesario.

118. Query Quotas

Puede definirse:

maxQueriesPerMinute
maxBytesRead
maxSourceCalls
maxConcurrentFederatedQueries

por:

tenant
application
user
workload class
119. Federation Isolation

Una query de un tenant no debe consumir ilimitadamente los recursos compartidos.

Por ello:

Tenant A
   │
   ▼
Quota / Scheduler
   │
   ▼
Federation

debe existir cuando la escala lo requiera.

120. Reference Architecture
                         ┌─────────────────────────┐
                         │       Consumers         │
                         │ API / UI / AI / Rules   │
                         └────────────┬────────────┘
                                      │
                                      ▼
                         ┌─────────────────────────┐
                         │   Federation Gateway    │
                         ├─────────────────────────┤
                         │ Auth                    │
                         │ Tenant Context          │
                         │ Policy                  │
                         │ Quotas                  │
                         └────────────┬────────────┘
                                      │
                                      ▼
                         ┌─────────────────────────┐
                         │ Federation Query Engine │
                         ├─────────────────────────┤
                         │ Parser                  │
                         │ Logical Planner         │
                         │ Optimizer               │
                         │ Physical Planner        │
                         │ Executor                │
                         │ Result Merger           │
                         └────────────┬────────────┘
                                      │
              ┌───────────────────────┼───────────────────────┐
              │                       │                       │
              ▼                       ▼                       ▼
       ┌─────────────┐        ┌─────────────┐        ┌─────────────┐
       │ SQL Adapter │        │ API Adapter │        │ Data Adapter│
       └──────┬──────┘        └──────┬──────┘        └──────┬──────┘
              │                      │                      │
              ▼                      ▼                      ▼
          Database                Service               Warehouse
              │                      │                      │
              └──────────────────────┼──────────────────────┘
                                     │
                                     ▼
                         ┌─────────────────────────┐
                         │ Provenance / Observability│
                         │ Metrics / Tracing / Audit│
                         └─────────────────────────┘
121. Federation Domain Model
Federation
├── federationId
├── sources
├── logicalSchema
├── mappings
├── policies
├── capabilities
├── queryPolicies
└── status
122. Source Model
FederationSource
├── sourceId
├── type
├── endpoint
├── owner
├── schema
├── capabilities
├── health
├── securityPolicy
└── residencyPolicy
123. Query Model
FederatedQuery
├── queryId
├── logicalQuery
├── tenantContext
├── consistencyRequirement
├── deadline
├── plan
├── sources
├── status
└── result
124. Query Execution Model
Query
  │
  ▼
Validate
  │
  ▼
Authorize
  │
  ▼
Resolve Sources
  │
  ▼
Build Logical Plan
  │
  ▼
Optimize
  │
  ▼
Build Physical Plan
  │
  ▼
Execute
  │
  ├──► Source A
  ├──► Source B
  └──► Source C
  │
  ▼
Merge
  │
  ▼
Validate Result
  │
  ▼
Return
125. Federated Events
FederationSourceRegistered
FederationSourceValidated
FederationSourceActivated
FederationSourceDegraded
FederationSourceDisabled
FederationSchemaChanged
FederationMappingChanged
FederatedQueryStarted
FederatedQueryCompleted
FederatedQueryFailed
FederatedQueryTimedOut
FederatedPartialResultProduced
FederatedPolicyDenied
FederatedSourceCircuitOpened
FederatedSourceCircuitClosed
126. Operational States
Federation
INITIALIZING
ACTIVE
DEGRADED
DRAINING
DISABLED
Source
REGISTERED
VALIDATING
ACTIVE
DEGRADED
UNAVAILABLE
DRAINING
DISABLED
Query
RECEIVED
VALIDATING
PLANNING
EXECUTING
MERGING
COMPLETED
PARTIAL
FAILED
CANCELLED
TIMED_OUT
127. Core Invariants
Invariant 1 — No Hidden Source

Todo dato federado debe tener una fuente identificable.

Invariant 2 — Explicit Authority

Cada dato debe tener una fuente de autoridad claramente definida.

Invariant 3 — No Silent Partial Results

Un resultado incompleto nunca debe presentarse como completo.

Invariant 4 — Security Propagation

Las políticas de seguridad deben propagarse a todos los sources involucrados.

Invariant 5 — Tenant Isolation

Una federated query no puede cruzar boundaries de tenant no autorizados.

Invariant 6 — Explicit Consistency

La federation layer debe declarar o comunicar las garantías de consistencia del resultado.

Invariant 7 — Bounded Resources

Una query federada no puede consumir recursos ilimitados de EVOXA ni de sus sources.

Invariant 8 — Failure Isolation

El fallo de un source no debe provocar automáticamente la caída de toda la federation platform.

Invariant 9 — Traceability

Los resultados que requieran auditabilidad deben conservar provenance suficiente para determinar de dónde proceden.

Invariant 10 — No Implicit Distributed Transaction

Una federated query no implica una transacción distribuida.

Invariant 11 — Explicit Mapping

Las relaciones entre modelo lógico y modelo físico deben estar definidas explícitamente.

Invariant 12 — Capability Awareness

El planner no puede asumir capacidades que un source no declara.

128. Relationship with E50
E50 — DATA SYNCHRONIZATION
        │
        │ copies / aligns state
        ▼
Aligned Data

mientras:

E52 — DATA FEDERATION
        │
        │ queries / composes
        ▼
Unified Logical View

Federation puede incluso consultar:

independent systems

sin sincronizarlos previamente.

129. Relationship with E51
E51 — DATA REPLICATION
        │
        ├── Primary
        ├── Replica A
        └── Replica B

Federation puede tratar una de esas réplicas como source:

Federation
    │
    └──► Read Replica

pero federation y replication siguen siendo responsabilidades diferentes.

130. Relationship with E27
E27 — QUERY ARCHITECTURE
        │
        ▼
Query semantics

E52 añade:

multiple physical sources
source planning
cross-source execution

Por tanto:

E27 define cómo EVOXA consulta; E52 define cómo una consulta puede cruzar múltiples sistemas.

131. Relationship with E28
E28 — READ MODEL
        │
        ▼
Materialized representation

frente a:

E52 — FEDERATION
        │
        ▼
Query-time representation
132. Relationship with E29

Search puede utilizar:

Federation → Search Source

pero Search Architecture continúa siendo responsable de:

indexing
ranking
retrieval
search semantics
133. Relationship with E30/E31
Federation
    │
    ▼
Reporting / Analytics

pero para cargas analíticas intensivas puede ser preferible:

Federation
    │
    ▼
Materialized Analytics Dataset
134. Relationship with E32 Intelligence
AI / Intelligence
       │
       ▼
Federation
       │
 ┌─────┼─────┐
 ▼     ▼     ▼
CRM  Orders  Payments

La federation layer proporciona acceso compuesto, mientras Intelligence determina cómo utilizar ese contexto.

135. Relationship with E33 Decision
Decision
   │
   ▼
Federated Context
   │
   ├── Customer
   ├── Account
   ├── Risk
   └── Transaction

La decisión debe conservar suficiente provenance cuando requiera explicabilidad.

136. Relationship with E34/E35

Federation debe permanecer principalmente en:

READ / COMPOSITION

mientras:

E34 Action
E35 Execution

controlan:

intent
action
execution

Esto evita que una consulta federada se convierta accidentalmente en una operación distribuida con efectos laterales.

137. Testing Matrix
                     Source A  Source B  Source C
Healthy                 ✓         ✓         ✓
Slow                    ✓         ~         ✓
Unavailable             ✓         X         ✓
Schema Changed          ~         ✓         ✓
Unauthorized            X         ✓         ✓
Partial Result          ✓         X         ✓
High Cardinality        ✓         ✓         ✓
Tenant Isolation        ✓         ✓         ✓
Rate Limited            ~         ~         ✓
Network Partition       ✓         X         ✓
138. Completion Criteria

E52 se considera arquitectónicamente completo cuando EVOXA dispone de:

✓ Federation boundary
✓ Source registration
✓ Source identity
✓ Source ownership
✓ Source capabilities
✓ Logical schema
✓ Physical mappings
✓ Federated query model
✓ Query decomposition
✓ Query planning
✓ Query optimization
✓ Filter pushdown
✓ Projection pushdown
✓ Aggregation pushdown
✓ Join strategies
✓ API federation
✓ Database federation
✓ Service federation
✓ Object/data federation
✓ Source-of-truth model
✓ Provenance
✓ Consistency model
✓ Partial-result policy
✓ Timeout policy
✓ Retry policy
✓ Circuit breakers
✓ Bulkheads
✓ Rate limits
✓ Query quotas
✓ Resource limits
✓ Tenant isolation
✓ Authorization
✓ Data residency
✓ Sensitive-data controls
✓ Schema evolution
✓ Schema drift handling
✓ Type normalization
✓ Identity mapping
✓ Query tracing
✓ Source observability
✓ Cost governance
✓ Federation caching
✓ Source lifecycle
✓ Adapter architecture
✓ Failure isolation
✓ Testing
✓ Chaos testing
✓ Security testing
✓ Audit
✓ Reference architecture
✓ Core invariants
139. Principio Rector de E52

Data Federation es la capacidad de EVOXA para presentar una superficie lógica unificada sobre datos distribuidos y heterogéneos, resolviendo en tiempo de consulta la localización, autorización, planificación, ejecución, composición, consistencia, trazabilidad y aislamiento de múltiples fuentes sin exigir su consolidación física.

La secuencia queda:

E49 — DATA MIGRATION
       │
       ▼
E50 — DATA SYNCHRONIZATION
       │
       ▼
E51 — DATA REPLICATION
       │
       ▼
E52 — DATA FEDERATION
       │
       ▼
E53 — DATA VIRTUALIZATION

La diferencia conceptual queda establecida:

MIGRATION       → move data
SYNCHRONIZATION → align data
REPLICATION     → duplicate data
FEDERATION      → compose data
VIRTUALIZATION  → abstract data access

E52 permite que EVOXA trate múltiples sistemas como un espacio lógico de datos, sin borrar sus boundaries físicos, ownerships, políticas de seguridad o responsabilidades de dominio.
