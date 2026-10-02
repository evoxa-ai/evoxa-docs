E53 — EVOXA Data Virtualization Architecture
Engineering Specification

Pertenece a: Engineering Specification
Capítulo: E53 — Data Virtualization Architecture
Anterior: E52 — Data Federation Architecture
Siguiente: E54 — Data Access Abstraction Architecture

1. Propósito

E53 define la arquitectura mediante la cual EVOXA proporciona una capa de acceso lógico y uniforme a datos distribuidos, heterogéneos y físicamente independientes, ocultando al consumidor los detalles innecesarios de almacenamiento, ubicación, protocolo y tecnología.

El principio fundamental es:

Data Virtualization abstrae dónde y cómo viven los datos, proporcionando una superficie lógica coherente para acceder a ellos sin exigir necesariamente su movimiento o materialización.

La relación conceptual con E52 es:

E52 — Federation
    │
    │ compose multiple sources
    ▼
Unified Federated Query

mientras:

E53 — Virtualization
    │
    │ abstract physical data access
    ▼
Logical Data Access Layer
2. Virtualization vs Federation

Son conceptos estrechamente relacionados, pero no equivalentes.

Federation

Se centra en:

Source A ──┐
Source B ──┼──► Federated Query
Source C ──┘

Su problema principal es:

¿Cómo ejecuto una operación sobre múltiples fuentes?

Virtualization

Se centra en:

Consumer
   │
   ▼
Logical Data Interface
   │
   ├── Database
   ├── API
   ├── Service
   ├── Warehouse
   └── Object Store

Su problema principal es:

¿Cómo accede el consumidor a los datos sin depender directamente de su implementación física?

3. Virtualization vs Replication

Replication:

Source
  │
  ├──► Copy A
  ├──► Copy B
  └──► Copy C

Virtualization:

Source
  │
  ▼
Virtual Access

Por tanto:

Replication mueve o duplica datos; Virtualization abstrae el acceso a los datos.

4. Virtualization vs Materialization
Materialized
Source
   │
   ▼
ETL / Pipeline
   │
   ▼
Materialized Dataset
Virtualized
Consumer
   │
   ▼
Virtual Data Layer
   │
   ▼
Source

La virtualización evita, cuando es apropiado, crear otra copia física del dataset.

5. Scope

E53 cubre:

Virtual Data Layer
Logical Data Interfaces
Source Abstraction
Data Access Abstraction
Logical Schema
Physical Schema Hiding
Source Resolution
Virtual Entities
Virtual Views
Virtual Queries
Source Adapters
Query Pushdown
Data Composition
Metadata
Lineage
Caching
Freshness
Security
Tenant Isolation
Data Residency
Performance
Failure Handling
Observability
Governance
6. Architectural Position
                    Consumers
                        │
          ┌─────────────┼─────────────┐
          ▼             ▼             ▼
         API            AI          Analytics
          │             │             │
          └─────────────┼─────────────┘
                        ▼
              ┌────────────────────┐
              │ Virtual Data Layer │
              └──────────┬─────────┘
                         │
              ┌──────────┼──────────┐
              ▼          ▼          ▼
          Database      API       Service
              │          │          │
              ▼          ▼          ▼
           Physical Data Sources
7. Core Principle

El consumidor debe trabajar contra:

Logical Customer
Logical Order
Logical Product
Logical Account

y no necesariamente contra:

customer_db.customer_table
orders-api/v3/orders
legacy_customer_system.CUST_01

La capa virtual oculta esas implementaciones.

8. Logical Data Interface

Una entidad virtual puede definirse:

VirtualCustomer
├── id
├── name
├── email
├── status
└── createdAt

sin imponer al consumer conocimiento de dónde proceden esos campos.

9. Virtual Entity

Una Virtual Entity representa una entidad lógica:

Virtual Entity
      │
      ├── Logical Schema
      ├── Mapping
      ├── Source
      ├── Policies
      └── Capabilities
10. Virtual View

Una Virtual View puede combinar:

Customer
+
Account
+
Subscription

y exponer:

Customer360

sin requerir necesariamente una tabla física Customer360.

11. Virtual Dataset

Puede definirse un dataset lógico:

CustomerTransactions

con datos procedentes de:

Customer DB
Order Service
Payment Service
12. Logical Schema

La capa de virtualización mantiene:

Logical Schema
      │
      ├── entities
      ├── attributes
      ├── relationships
      ├── types
      └── constraints

Este esquema es independiente del storage físico.

13. Physical Schema Hiding

El consumidor no debería necesitar conocer:

PostgreSQL
MongoDB
REST
GraphQL
Kafka
Parquet
Legacy SOAP

si todos ellos están correctamente encapsulados detrás del virtual data layer.

14. Source Abstraction
Logical Data
     │
     ▼
Virtualization Layer
     │
     ├── SQL Adapter
     ├── API Adapter
     ├── Service Adapter
     ├── File Adapter
     └── Search Adapter
15. Source Resolution

Una petición:

GET VirtualCustomer

se resuelve internamente:

VirtualCustomer
       │
       ▼
Source Resolver
       │
       ▼
CRM Customer API

El consumer permanece desacoplado.

16. Multiple Physical Implementations

La misma entidad lógica puede tener diferentes implementaciones:

VirtualCustomer
      │
      ├── Production → CRM
      ├── Test       → Mock DB
      └── Analytics  → Warehouse

La selección depende del contexto.

17. Environment Abstraction

Puede utilizarse:

VirtualCustomer

en:

Development
Testing
Staging
Production

sin cambiar el contrato lógico.

18. Tenant-aware Resolution

Para multi-tenancy:

VirtualCustomer
       │
       ▼
Tenant Resolver
       │
 ┌─────┼─────┐
 ▼     ▼     ▼
Tenant A B   C

Cada tenant puede utilizar una fuente o configuración diferente cuando la arquitectura lo permita.

19. Data Access Abstraction

El consumer utiliza:

Logical Query

mientras el virtualization layer genera:

Physical Query

Ejemplo:

Logical:
customer.status = ACTIVE

Physical:
CRM API filter=status:eq:ACTIVE
20. Query Translation

Pipeline:

Logical Query
      │
      ▼
Parser
      │
      ▼
Logical Plan
      │
      ▼
Mapping Resolver
      │
      ▼
Physical Query
      │
      ▼
Source
21. Query Pushdown

Cuando la fuente soporta la operación:

Virtual Layer
      │
      ▼
Filter
      │
      ▼
Source

El objetivo es minimizar:

network traffic
memory usage
latency
source-independent processing
22. Query Rewriting

La capa puede transformar:

Logical Query

en:

Source-specific Query

sin exponer el dialecto físico al consumer.

23. Source Capability Model

Cada adapter debe declarar:

supportsFilter
supportsProjection
supportsSort
supportsPagination
supportsAggregation
supportsJoin
supportsStreaming

El virtualization engine no debe asumir capacidades inexistentes.

24. Unsupported Operations

Si el source no soporta:

ORDER BY

la virtual layer puede:

local sort

si el dataset es suficientemente pequeño.

Alternativamente:

reject

si la operación excede los límites de seguridad o capacidad.

25. Virtual Joins

Una virtual view puede representar:

Customer
   │
   └── Account

aunque físicamente estén en:

CRM
Billing DB

El engine puede delegar o ejecutar el join localmente.

26. Virtual Relationships

El modelo lógico puede definir:

Customer 1 ─── N Orders

aunque no exista una foreign key física.

La relación puede basarse en:

logical identity
mapping
join key
service contract
27. Identity Resolution

Diferentes sistemas pueden utilizar:

CRM_ID
CUSTOMER_ID
ACCOUNT_ID
LEGACY_ID

La virtual layer debe definir cómo se relacionan.

28. Virtual Identity

Puede existir:

VirtualCustomerId

que abstrae:

CRM_ID
+
LegacyCustomerId
+
ExternalCustomerId

cuando el dominio lo requiere.

29. Schema Mapping
Logical Field      Physical Field
─────────────────────────────────
customer.id   →   crm.customer_id
customer.name →   crm.full_name
customer.status → crm.lifecycle_status
30. Type Mapping
Logical UUID
     │
     ├── PostgreSQL UUID
     ├── API String
     └── Legacy Numeric ID

La conversión debe ser determinista.

31. Null and Missing Semantics

La virtual layer debe distinguir cuando sea necesario:

NULL
MISSING
UNKNOWN
NOT_APPLICABLE
EMPTY

Esto evita inconsistencias entre fuentes.

32. Virtualization and Data Freshness

La virtualización puede proporcionar acceso a datos relativamente frescos:

Consumer
   │
   ▼
Virtual Layer
   │
   ▼
Live Source

pero la freshness real depende del source.

33. Freshness Metadata

Cuando sea relevante:

retrievedAt
sourceTimestamp
sourceVersion
freshnessGuarantee

debe acompañar al resultado.

34. Cache

La virtualización puede utilizar cache:

Virtual Layer
      │
      ├── Cache Hit → Result
      │
      └── Cache Miss → Source

La cache debe tener una política explícita.

35. Cache Scope

Puede ser:

Global
Tenant
User
Application
Query
Entity

La selección depende de la sensibilidad y semántica de los datos.

36. Virtualization Cache Invalidation

Puede utilizar:

TTL
Event-driven invalidation
Version validation
Explicit invalidation

No debe asumirse que una cache es siempre válida.

37. Security

Virtualization no debe convertirse en un mecanismo para saltarse controles de acceso.

Debe respetar:

Authentication
Authorization
Tenant Isolation
Field Security
Row Security
Data Classification
Residency
Audit
38. Security Context Propagation

La solicitud debe transportar:

tenantId
userId
roles
permissions
purpose
correlationId

cuando sea necesario.

39. Delegated Authorization

Cuando el source soporta autorización propia:

Virtual Layer
      │
      ▼
Source Authorization

puede complementarse con:

EVOXA Authorization
40. Field-Level Security

La virtual entity:

Customer
├── id
├── name
├── email
└── secretData

puede exponer:

id
name
email

pero ocultar:

secretData

según policy.

41. Row-Level Security

Ejemplo:

Virtual Orders
       │
       ▼
tenantId = T1

debe traducirse al source cuando sea posible.

42. Tenant Isolation

La resolución virtual nunca debe permitir:

Tenant A
   │
   X
Tenant B data

por una mala configuración de mapping o cache.

43. Data Residency

La virtual layer debe respetar:

EU tenant
   │
   ▼
EU sources

cuando la política de residencia lo requiera.

44. Sensitive Data Handling

La virtualización debe controlar:

retrieval
transformation
cache
logging
tracing
debugging
export

de datos sensibles.

45. Provenance

Cada virtual field importante puede tener:

source
physical field
mapping
transformation
retrievedAt

Esto permite responder:

¿De dónde salió este dato?

46. Lineage
VirtualCustomer.email
        │
        ▼
CRM.customer_email
        │
        ▼
CRM Customer Service

La lineage debe ser navegable cuando el caso de uso lo requiera.

47. Virtualization Metadata

Debe existir metadata sobre:

Virtual Entity
├── logical schema
├── source mappings
├── capabilities
├── security policy
├── freshness
├── cache policy
├── ownership
└── lifecycle
48. Metadata Registry

Arquitectura:

              ┌─────────────────────┐
              │ Virtual Metadata    │
              │ Registry             │
              └──────────┬──────────┘
                         │
            ┌────────────┼────────────┐
            ▼            ▼            ▼
        Schemas       Mappings     Policies
49. Virtualization Catalog

El catálogo puede contener:

VirtualCustomer
VirtualOrder
VirtualProduct
Customer360
AccountBalance
TransactionHistory

Cada objeto debe tener owner y lifecycle.

50. Lifecycle

Una virtual entity puede evolucionar:

DRAFT
   ↓
VALIDATING
   ↓
ACTIVE
   ↓
DEPRECATED
   ↓
RETIRED
51. Versioning

Debe soportarse:

VirtualCustomer v1
VirtualCustomer v2

cuando existan cambios incompatibles.

52. Backward Compatibility

Cambios como:

rename field
change type
remove field
change semantics

requieren una estrategia de compatibilidad.

53. Schema Evolution

Una modificación física:

CRM.full_name

no debería romper automáticamente:

VirtualCustomer.name

si el mapping puede absorber el cambio.

54. Decoupling

Uno de los objetivos principales:

Consumer
   │
   X
Physical implementation

en lugar de:

Consumer
   │
   ▼
Physical Database Schema
55. Anti-Corruption Boundary

Virtualization puede actuar como una capa de protección frente a:

legacy systems
external APIs
poor schemas
inconsistent naming
legacy identifiers

El modelo lógico de EVOXA no tiene que adoptar sus imperfecciones.

56. Legacy Integration

Ejemplo:

Legacy Customer System
        │
        ▼
Legacy Adapter
        │
        ▼
Virtual Customer

El legacy model queda encapsulado.

57. API Integration
Virtual Order
      │
      ▼
Order API
      │
      ▼
JSON

El consumer no necesita conocer:

HTTP
endpoint
pagination
authentication protocol
API version

si la virtual layer los abstrae.

58. Database Integration
Virtual Account
      │
      ▼
SQL Adapter
      │
      ▼
Account DB
59. File/Data Lake Integration
Virtual Dataset
      │
      ▼
Object Adapter
      │
      ▼
Parquet / JSON / CSV

La virtualización puede convertir el formato físico en un modelo lógico.

60. Search Integration
Virtual Product
      │
      ▼
Search Adapter
      │
      ▼
Index

Debe distinguirse entre:

authoritative source
derived index
61. Event-backed Data

Una virtual view puede derivarse de:

Event Store

pero debe quedar explícito que:

event-derived state

puede tener diferentes garantías de consistencia.

62. Virtualization and Eventual Consistency

Si la fuente es eventualmente consistente:

Virtual Layer
      │
      ▼
Stale Result

no debe presentarse como:

strongly consistent
63. Query Consistency Modes

EVOXA puede soportar:

BEST_EFFORT
SOURCE_CONSISTENT
BOUNDED_STALENESS
STRONG_WHERE_SUPPORTED

La capacidad real depende de cada source.

64. Transactions

La virtualización no implica automáticamente:

distributed transaction

Una virtual entity debe distinguir:

read abstraction

de:

write transaction
65. Virtual Writes

Los writes deben seguir ownership del dominio:

Virtual Customer
      │
      ▼
Customer Domain Service
      │
      ▼
Authoritative Store

La virtual layer no debería convertirse en un bypass de domain services.

66. Write Routing

Si una operación permite write:

Virtual Update
      │
      ▼
Write Policy
      │
      ▼
Authoritative Source

debe existir una única autoridad clara.

67. Side Effects

Un acceso virtual debe ser preferentemente:

read-oriented

Los efectos laterales deben gestionarse mediante:

Action
Execution
Workflow
Domain Service
68. Performance Model

Las variables principales:

source latency
translation latency
network latency
join cost
serialization
deserialization
cache hit rate
result size
69. Latency Budget

Ejemplo:

Virtual Request
│
├── Authorization    5ms
├── Resolution       5ms
├── Translation      5ms
├── Source          80ms
├── Mapping          5ms
└── Serialization    5ms
                    ─────
                    105ms

El presupuesto debe definirse por workload.

70. Connection Pooling

Los adapters deben evitar:

connect
query
disconnect

en cada request cuando el protocolo lo permita.

Debe utilizarse:

connection pooling

con límites adecuados.

71. Concurrency Control

Debe existir:

max concurrent source calls
max concurrent virtual queries
max connections

para evitar saturación.

72. Circuit Breakers
Source
  │
  ▼
Repeated failures
  │
  ▼
Circuit OPEN
  │
  X
  │
Recovery
  │
  ▼
HALF_OPEN
73. Timeouts

Debe existir:

request timeout
source timeout
mapping timeout
global deadline

No debe permitirse que un source lento bloquee indefinidamente la virtual layer.

74. Retry

Retry sólo cuando:

operation safe
error transient
deadline remains
source policy allows

No deben reintentarse indiscriminadamente operaciones no idempotentes.

75. Partial Failure

Una virtual entity compuesta puede recibir:

Customer ✓
Account ✗

La política debe decidir:

fail
partial
fallback
cached value
76. Partial Result Semantics

Si se permite:

Customer360

con componentes ausentes:

Customer ✓
Account ✗
Orders ✓

el resultado debe indicar explícitamente:

completeness = PARTIAL
77. Fallback

Un source puede tener:

Primary
Secondary
Cache

pero el fallback debe preservar las garantías declaradas.

78. Observability

Cada virtual request debe poder rastrearse:

virtualRequestId
correlationId
tenantId
virtualEntity
source
latency
cache
status
79. Metrics

Métricas mínimas:

virtualRequests
successRate
errorRate
p50
p95
p99
cacheHitRate
sourceFailures
translationFailures
mappingFailures
partialResults
80. Source Metrics

Por source:

requestCount
latency
timeouts
errors
retries
bytesRead
rowsRead
81. Explainability

La plataforma debería poder mostrar:

VirtualCustomer
       │
       ▼
CRM Adapter
       │
       ▼
GET /customers/{id}

cuando sea seguro y útil para operaciones.

82. Explain Plan

Para consultas complejas:

Virtual Query
      │
      ▼
Logical Plan
      │
      ▼
Physical Plan
      │
      ├── CRM
      ├── Billing
      └── Orders
83. Cost Governance

La virtualización puede trasladar costes al runtime.

Debe medirse:

source calls
network
compute
cache
query frequency

por:

tenant
application
workload
team

cuando sea necesario.

84. Query Admission

Una consulta que exceda límites puede:

REJECT
THROTTLE
QUEUE
DEGRADE

en lugar de afectar a toda la plataforma.

85. Resource Limits

Debe haber límites como:

maxRows
maxBytes
maxExecutionTime
maxJoinMemory
maxSourceCalls
maxFanout
86. Fan-out Protection

Una consulta virtual no debería provocar accidentalmente:

1 request
   │
   ├── 1,000 source calls
   ├── 1,000 source calls
   └── 1,000 source calls

Debe existir control de fan-out.

87. N+1 Protection

Debe evitarse:

Get 100 customers
     │
     ├──► API call order customer 1
     ├──► API call order customer 2
     ├──► ...
     └──► API call order customer 100

cuando sea posible mediante:

batching
bulk APIs
joins
prefetching
88. Batching

En lugar de:

get(id1)
get(id2)
get(id3)

usar:

get(ids=[id1,id2,id3])

cuando el source lo soporte.

89. Pagination Abstraction

La virtual layer puede normalizar:

offset pagination
cursor pagination
token pagination
page-based pagination

a un contrato lógico.

90. Global Pagination

En views compuestas:

Customer360

la paginación debe respetar la semántica global, no simplemente la paginación de cada source.

91. Sorting

La virtual layer debe determinar si el sort puede:

push down

o necesita:

local sort
92. Aggregation

Para:

COUNT
SUM
AVG
MIN
MAX

debe priorizarse pushdown cuando sea seguro.

93. Streaming

Para grandes resultados:

Source
  │
  ▼
Virtual Stream
  │
  ▼
Consumer

permite evitar cargar todo en memoria.

94. Backpressure

Si el consumer es lento:

Consumer slow
      │
      ▼
Virtual Layer
      │
      ▼
Source

la presión debe propagarse de manera controlada.

95. Memory Safety

Debe limitarse:

buffer size
result size
join memory
cache size
concurrent streams
96. Schema Registry Integration

El virtualization catalog puede integrarse con:

Schema Registry

para detectar:

type changes
field additions
field removals
version mismatches
97. Schema Drift

Un cambio físico:

CRM.customer_status

→

CRM.lifecycle_status

debe activar:

schema drift detection

antes de producir resultados incorrectos.

98. Contract Testing

Cada virtual entity debe probar:

schema
mapping
source connectivity
authorization
query semantics
error handling
pagination
freshness
99. Integration Testing

Debe validarse:

Virtual Entity
      │
      ▼
Adapter
      │
      ▼
Real Source

con casos reales de:

success
timeout
schema drift
authorization failure
partial data
large data
100. Chaos Testing

Escenarios:

source unavailable
network partition
high latency
rate limit
schema change
malformed response
connection exhaustion
cache corruption
101. Security Testing

Debe probar:

tenant escape
field leakage
row leakage
source privilege escalation
cache cross-tenant leakage
PII logging
unauthorized source access
102. Audit

Deben poder auditarse:

virtual entity registration
mapping changes
source changes
policy changes
privileged access
schema changes
103. Governance

Cada virtual object necesita:

owner
domain
classification
source
SLA
freshness
security policy
retirement policy
104. Ownership

Debe evitarse:

VirtualCustomer
      │
      X
No owner

Debe existir:

VirtualCustomer
      │
      ▼
Customer Domain Owner
105. SLA

Una virtual entity puede declarar:

availability
latency
freshness
completeness

pero no debe prometer garantías superiores a sus sources.

106. Data Quality

La virtual layer puede exponer:

qualityStatus
completeness
freshness
validationWarnings

cuando el consumer necesite conocer la calidad del resultado.

107. Virtualization and Data Lineage
Consumer
   │
   ▼
VirtualCustomer.email
   │
   ▼
Mapping
   │
   ▼
CRM.customer_email
   │
   ▼
CRM Service

La lineage conecta:

logical
→ physical
→ source
108. Virtualization and AI

EVOXA AI puede consultar:

VirtualCustomer
VirtualOrder
VirtualAccount
VirtualProduct

sin conocer las implementaciones físicas.

Esto reduce el acoplamiento de los agentes.

109. AI Security

Un agente no debe poder usar:

Virtual Customer

para acceder indirectamente a campos que no tendría autorización para consultar directamente.

La autorización debe aplicarse antes de producir el resultado.

110. Decision Systems

Una decisión puede consultar:

Virtual Customer Profile

pero el decision engine debe conocer, cuando sea relevante:

freshness
provenance
confidence
source authority
111. Workflow Integration

Un workflow puede:

Get Virtual Customer
       │
       ▼
Evaluate Rule
       │
       ▼
Execute Action

La virtualización proporciona datos; no debe apropiarse de la responsabilidad del workflow.

112. Application Services

Application Services pueden utilizar:

Virtual Repository

cuando necesitan una abstracción de lectura sobre fuentes heterogéneas.

113. Repository Relationship

Conceptualmente:

Application Service
       │
       ▼
Repository / Data Access Interface
       │
       ▼
Virtualization Layer
       │
       ▼
Physical Source
114. Domain Boundary

La virtualización no debe convertir entidades externas en entidades del dominio sin una decisión explícita.

Por ejemplo:

ExternalCustomer

no necesariamente equivale a:

DomainCustomer
115. Anti-Corruption Layer

Para sistemas legacy:

Legacy Model
     │
     ▼
Virtualization / ACL
     │
     ▼
EVOXA Logical Model

Esto permite proteger el modelo interno.

116. Virtualization Catalog Example
VirtualCustomer
────────────────────────────
Owner: Customer Domain
Source: CRM
Freshness: Live
Security: CustomerPolicy
Status: ACTIVE

Fields:
id
name
email
status
createdAt
117. Virtual View Example
Customer360
────────────────────────────
Customer
+
Account
+
Subscription
+
RecentOrders

Physicalmente:

CRM
Billing
Subscription Service
Order DB
118. Reference Architecture
                         ┌───────────────────────┐
                         │      Consumers        │
                         │ API / AI / Apps / BI  │
                         └───────────┬───────────┘
                                     │
                                     ▼
                      ┌──────────────────────────┐
                      │  Virtual Data Interface  │
                      └────────────┬─────────────┘
                                   │
                      ┌────────────▼─────────────┐
                      │ Virtualization Engine    │
                      ├──────────────────────────┤
                      │ Authorization            │
                      │ Tenant Resolution        │
                      │ Schema Resolution        │
                      │ Query Translation        │
                      │ Mapping                  │
                      │ Optimization             │
                      │ Cache                    │
                      │ Result Normalization     │
                      └────────────┬─────────────┘
                                   │
             ┌─────────────────────┼─────────────────────┐
             │                     │                     │
             ▼                     ▼                     ▼
        ┌─────────┐          ┌─────────┐          ┌──────────┐
        │ SQL     │          │ API     │          │ Service  │
        │ Adapter │          │ Adapter │          │ Adapter  │
        └────┬────┘          └────┬────┘          └────┬─────┘
             │                    │                    │
             ▼                    ▼                    ▼
         Database              External API        Domain Service
119. Metadata Architecture
                  ┌─────────────────────┐
                  │ Virtual Data Catalog│
                  └──────────┬──────────┘
                             │
        ┌────────────────────┼────────────────────┐
        ▼                    ▼                    ▼
   Logical Schema        Mappings             Policies
        │                    │                    │
        └────────────────────┼────────────────────┘
                             ▼
                  Virtualization Engine
120. Runtime Architecture
Request
   │
   ▼
Authenticate
   │
   ▼
Authorize
   │
   ▼
Resolve Tenant
   │
   ▼
Resolve Virtual Entity
   │
   ▼
Resolve Source
   │
   ▼
Translate Query
   │
   ▼
Optimize
   │
   ▼
Execute
   │
   ▼
Normalize
   │
   ▼
Apply Policy
   │
   ▼
Attach Provenance
   │
   ▼
Return
121. Virtualization Events
VirtualEntityCreated
VirtualEntityUpdated
VirtualEntityDeprecated
VirtualEntityRetired

VirtualSourceRegistered
VirtualSourceChanged
VirtualSourceDisabled

VirtualMappingCreated
VirtualMappingChanged
VirtualMappingInvalidated

VirtualSchemaChanged
VirtualSchemaDriftDetected

VirtualQueryStarted
VirtualQueryCompleted
VirtualQueryFailed
VirtualQueryTimedOut
VirtualPartialResultProduced

VirtualPolicyChanged
VirtualCacheInvalidated
122. Operational States
Virtual Entity
DRAFT
VALIDATING
ACTIVE
DEGRADED
DEPRECATED
RETIRED
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
AUTHORIZED
RESOLVING
PLANNING
EXECUTING
NORMALIZING
COMPLETED
PARTIAL
FAILED
CANCELLED
TIMED_OUT
123. Core Invariants
Invariant 1 — Logical Independence

Los consumidores no deben depender innecesariamente del esquema físico del source.

Invariant 2 — Explicit Mapping

Todo dato virtualizado debe tener un mapping físico explícito.

Invariant 3 — Source Authority

La virtualización no modifica silenciosamente la autoridad del dato.

Invariant 4 — Security Preservation

La abstracción virtual nunca puede ampliar los privilegios efectivos del consumidor.

Invariant 5 — Tenant Isolation

Una virtual entity no puede exponer datos de otro tenant.

Invariant 6 — Explicit Freshness

La virtualización no puede prometer una freshness superior a la proporcionada por la fuente.

Invariant 7 — Explicit Consistency

La virtualización debe preservar o declarar las garantías reales de consistencia.

Invariant 8 — Bounded Execution

Toda operación virtual debe estar limitada por tiempo, memoria, cardinalidad y fan-out apropiados.

Invariant 9 — No Hidden Side Effects

El acceso virtual de lectura no debe producir efectos laterales implícitos.

Invariant 10 — Traceability

Los datos virtualizados deben poder rastrearse hasta su origen cuando el caso de uso lo requiera.

Invariant 11 — No Physical Leakage

Los detalles internos de una fuente no deben formar parte del contrato lógico salvo decisión arquitectónica explícita.

Invariant 12 — Capability Awareness

El virtual engine sólo puede delegar operaciones soportadas por el source.

124. Relationship with E52

La relación debe mantenerse explícita:

E52 — DATA FEDERATION
        │
        │ cross-source composition
        ▼
Federated Query
        │
        ▼
E53 — DATA VIRTUALIZATION
        │
        │ abstraction of data access
        ▼
Logical Data Surface

E52 responde:

¿Cómo componemos datos distribuidos?

E53 responde:

¿Cómo ocultamos la complejidad física de esos datos al consumidor?

Una implementación de E53 puede utilizar internamente capacidades de E52.

125. Relationship with E51
E51 — REPLICATION
      │
      ▼
Physical copies

mientras:

E53 — VIRTUALIZATION
      │
      ▼
Logical abstraction

No deben confundirse.

Una virtual entity puede apuntar a:

Primary
Replica
Cache
Warehouse

pero la elección debe estar gobernada por políticas explícitas.

126. Relationship with E50

Synchronization:

A ─────► B

Virtualization:

Consumer
   │
   ▼
Virtual Layer
   │
   ├──► A
   └──► B

La virtualización no requiere que A y B estén sincronizados.

127. Relationship with E27

E27 define:

query semantics

E53 añade:

logical-to-physical translation
source abstraction
virtual data contracts
128. Relationship with E28
E28 Read Model
       │
       ▼
Materialized representation

frente a:

E53 Virtualization
       │
       ▼
Logical runtime representation
129. Relationship with E29

Search puede ser un source virtualizado:

Virtual Product
      │
      ▼
Search Adapter

pero la semántica de ranking y retrieval continúa perteneciendo a Search Architecture.

130. Relationship with E30/E31

Reporting y Analytics pueden consumir:

Virtual Dataset

pero si el workload es masivo y recurrente:

Virtualization
      │
      ▼
Materialized Analytics Model

puede ser arquitectónicamente superior.

131. Relationship with E32
AI Agent
   │
   ▼
Virtual Data Interface
   │
   ├── Customer
   ├── Orders
   ├── Products
   └── Accounts

Esto reduce la necesidad de que el agente conozca múltiples APIs y schemas físicos.

132. Relationship with E33

Las decisiones pueden utilizar entidades virtuales, pero deben conocer:

source authority
freshness
provenance
consistency

cuando estos factores afectan al resultado.

133. Relationship with E34/E35

La virtualización debe permanecer principalmente como:

DATA ACCESS

mientras:

E34 → ACTION
E35 → EXECUTION

gestionan efectos y operaciones.

134. Testing Matrix
                         Expected
Source available             ✓
Source unavailable           ✓
Schema drift                 ✓
Mapping invalid              ✓
Tenant isolation             ✓
Field security               ✓
Cache isolation              ✓
Large dataset                ✓
High fan-out                 ✓
N+1 prevention               ✓
Timeout                      ✓
Partial source               ✓
Rate limit                   ✓
Legacy source                ✓
API source                   ✓
Database source              ✓
135. Completion Criteria

E53 se considera arquitectónicamente completo cuando EVOXA dispone de:

✓ Virtual data boundary
✓ Logical data interfaces
✓ Virtual entities
✓ Virtual views
✓ Virtual datasets
✓ Logical schema
✓ Physical schema hiding
✓ Source abstraction
✓ Source resolution
✓ Query translation
✓ Query rewriting
✓ Capability negotiation
✓ Query pushdown
✓ Virtual joins
✓ Identity resolution
✓ Schema mapping
✓ Type mapping
✓ Null semantics
✓ Freshness model
✓ Consistency model
✓ Cache architecture
✓ Security propagation
✓ Tenant isolation
✓ Field-level security
✓ Row-level security
✓ Data residency
✓ Provenance
✓ Lineage
✓ Metadata registry
✓ Virtualization catalog
✓ Lifecycle management
✓ Versioning
✓ Backward compatibility
✓ Legacy isolation
✓ API abstraction
✓ Database abstraction
✓ Object/data abstraction
✓ Search abstraction
✓ Performance controls
✓ Connection pooling
✓ Concurrency limits
✓ Circuit breakers
✓ Timeout controls
✓ Retry controls
✓ Fan-out protection
✓ N+1 protection
✓ Batching
✓ Pagination abstraction
✓ Streaming
✓ Backpressure
✓ Resource limits
✓ Schema drift detection
✓ Contract testing
✓ Chaos testing
✓ Security testing
✓ Audit
✓ Governance
✓ Ownership
✓ SLA model
✓ AI integration
✓ Decision integration
✓ Workflow integration
✓ Core invariants
136. Principio Rector de E53

Data Virtualization es la capacidad de EVOXA para presentar una superficie lógica, segura y gobernada sobre datos físicamente distribuidos, ocultando las particularidades de almacenamiento, protocolo, esquema y ubicación mientras preserva explícitamente ownership, autoridad, consistencia, freshness, seguridad, provenance y límites operativos.

La secuencia completa queda:

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
       │
       ▼
E54 — DATA ACCESS ABSTRACTION

Y la distinción fundamental:

MIGRATION       → move
SYNCHRONIZATION → align
REPLICATION     → duplicate
FEDERATION      → compose
VIRTUALIZATION  → abstract

E53 convierte la heterogeneidad física de los datos en una superficie lógica estable para EVOXA, permitiendo que aplicaciones, agentes, decisiones, workflows y servicios consuman información sin acoplarse directamente a la infraestructura que la almacena.
