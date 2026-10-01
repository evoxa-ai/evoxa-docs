E27 — EVOXA Query Architecture
Engineering Specification

Pertenece a: Engineering Specification
Volumen: volume-09-engineering
Capítulo: E27 — Query Architecture
Anterior: E26 — Projection Architecture
Siguiente: E28 — Read Model Architecture

1. Propósito

E27 define la arquitectura de Query de EVOXA.

Query establece cómo el sistema:

solicita información;
define criterios de búsqueda;
recupera datos;
filtra;
ordena;
pagina;
compone resultados;
aplica límites;
selecciona projections;
mantiene separación entre lectura y modificación.

La pregunta fundamental de Query es:

¿Qué información necesita el consumidor y bajo qué criterios debe recuperarse?

2. Concepto Fundamental
Query
  │
  ▼
Query Handler
  │
  ▼
Data Source
  │
  ▼
Projection
  │
  ▼
Query Result

Una Query describe una intención de lectura.

No debe modificar el estado del sistema salvo efectos técnicos estrictamente controlados.

3. Query vs Command
Command
    │
    ▼
Change State

Query
    │
    ▼
Read State

Principio:

Command → mutation
Query   → retrieval
4. Query Boundary

El boundary principal:

Query Request
      ↓
Query Handler
      ↓
Query Execution
      ↓
Projection
      ↓
Query Result
5. Query Contract

Conceptualmente:

execute(query, context)

produce:

QueryResult

o:

QueryError
6. Query Characteristics

Una Query debería ser:

read_only
deterministic
bounded
observable
versionable
cacheable
composable

cuando el caso de uso lo permita.

7. Query Types

EVOXA debe contemplar:

Entity Query
Collection Query
Search Query
Lookup Query
Detail Query
Summary Query
List Query
Aggregation Query
Reporting Query
Analytics Query
Cross-Domain Query
Temporal Query
Historical Query
8. Entity Query

Obtiene una entidad concreta:

GetCustomerById
CustomerId
    ↓
Customer
9. Collection Query

Obtiene una colección:

ListCustomers
Criteria
   ↓
Customer[]
10. Search Query

Permite:

text
filters
sorting
pagination

Ejemplo:

SearchCustomers
11. Lookup Query

Busca una correspondencia específica:

FindCustomerByEmail

Puede devolver:

Customer?
12. Detail Query

Obtiene información detallada:

GetCustomerDetail

Normalmente utiliza:

CustomerDetailProjection
13. Summary Query

Obtiene una vista reducida:

GetCustomerSummary

con:

CustomerSummaryProjection
14. List Query

Está optimizada para colecciones:

CustomerListQuery

Resultado:

items
page
total

cuando corresponda.

15. Aggregation Query

Calcula resultados agregados:

CountOrders
SumRevenue
AverageOrderValue

Debe diferenciarse entre:

query aggregation

y:

domain business rule
16. Reporting Query

Puede proporcionar:

reports
KPIs
period comparisons
historical data

No debe contaminar el modelo transaccional.

17. Analytics Query

Está orientada a:

analytics
BI
data warehouse
data lake
metrics

Puede utilizar fuentes especializadas.

18. Cross-Domain Query

Puede requerir información de varios dominios:

Customer
+
Orders
+
Billing

La coordinación debe pertenecer a una capa explícita.

19. Temporal Query

Permite consultas como:

CustomerAtTime
OrdersDuringPeriod
StateAtDate

Debe definir claramente la semántica temporal.

20. Historical Query

Consulta:

history
events
audit records
previous states

No debe confundirse con obtener el estado actual.

21. Query Object

Una Query debe representar explícitamente sus parámetros.

Ejemplo conceptual:

SearchCustomersQuery
├── tenantId
├── filters
├── sort
├── page
└── pageSize
22. Query Handler

Responsabilidad:

receive query
validate query
resolve data source
execute read
apply projection
return result

No debería contener lógica de negocio extensa.

23. Query Handler Isolation
Query A
   ↓
Handler A

Query B
   ↓
Handler B

Esto permite:

testing
authorization
observability
performance tuning

por caso de uso.

24. Query Service

Un Query Service puede agrupar operaciones relacionadas:

CustomerQueryService

pero no debe convertirse en:

UniversalQueryService

sin boundaries claros.

25. Query Repository

Puede existir:

CustomerQueryRepository

orientado exclusivamente a lectura.

Esto permite optimizar:

SELECT
JOIN
INDEX
FILTER
SORT

sin afectar al Repository transaccional.

26. Repository vs Query Repository
Repository
    ↓
Domain persistence

Query Repository
    ↓
Read optimization

La segunda puede retornar:

DTO
Projection
Read Model

sin reconstruir necesariamente entidades de dominio.

27. Query Execution

Flujo:

Query
 ↓
Authorization
 ↓
Validation
 ↓
Query Handler
 ↓
Data Source
 ↓
Projection
 ↓
Result
28. Authorization Boundary

Authorization debe preceder a la exposición de datos:

Caller
 ↓
Authorization
 ↓
Query
 ↓
Projection

Query no sustituye autorización.

29. Query Validation

Debe validar:

required parameters
parameter types
range limits
pagination limits
sorting fields
filter operators
30. Query vs Validation Architecture

E22 define validación.

E27 consume esa capacidad:

E22 Validation
      ↓
E27 Query

Query no debe duplicar el framework general de validación.

31. Query Criteria

Los criterios pueden incluir:

equal
notEqual
greaterThan
lessThan
between
in
contains
startsWith
exists

Debe existir una allowlist de operadores soportados.

32. Filter Object

Conceptualmente:

Filter
├── field
├── operator
└── value

Ejemplo:

status = ACTIVE
33. Filter Security

Nunca debe permitirse que el consumidor construya arbitrariamente:

raw SQL
raw database expressions

desde un Query API público.

Preferir:

structured filters
34. Query Injection Protection

Debe prevenir:

SQL injection
NoSQL injection
search injection
expression injection

mediante:

parameterization
allowlists
typed criteria
query builders
35. Sorting

Una Query puede definir:

sort:
  - field: createdAt
    direction: DESC

Los campos ordenables deben estar explícitamente permitidos.

36. Multi-Sort

Puede soportar:

status ASC
createdAt DESC
id ASC

para obtener resultados deterministas.

37. Stable Sorting

Toda paginación debe utilizar un orden estable.

Preferible:

createdAt DESC
id DESC

en lugar de únicamente:

createdAt DESC

cuando existan empates.

38. Pagination

EVOXA debe soportar:

offset pagination
cursor pagination
keyset pagination

según el caso.

39. Offset Pagination
page=2
pageSize=50

Ventajas:

simple
familiar

Problemas:

large offsets
data mutation between pages
performance
40. Cursor Pagination
after=cursor
limit=50

Adecuada para:

large datasets
infinite scrolling
high-volume APIs
41. Keyset Pagination

Utiliza valores de orden:

createdAt
id

para localizar el siguiente conjunto.

Es especialmente útil en consultas de gran volumen.

42. Pagination Contract

Debe definir:

items
nextCursor
hasNext
pageSize

o el equivalente según el modelo.

43. Pagination Limits

Debe existir:

defaultPageSize
maximumPageSize

Nunca permitir:

pageSize = unlimited

en un endpoint general.

44. Query Result

Puede adoptar:

SingleResult<T>
CollectionResult<T>
PagedResult<T>
CursorResult<T>
AggregationResult<T>
45. Empty Result

Debe distinguirse:

not found

de:

empty collection

Ejemplo:

GetCustomer(id)
→ NotFound

mientras:

ListCustomers(filter)
→ []
46. Query Errors
QUERY_INVALID
QUERY_UNAUTHORIZED
QUERY_NOT_FOUND
QUERY_UNSUPPORTED
QUERY_TIMEOUT
QUERY_LIMIT_EXCEEDED
QUERY_SOURCE_UNAVAILABLE
QUERY_EXECUTION_FAILED
47. Query Timeout

Toda Query externa o costosa debería tener:

timeout

explícito.

Una Query nunca debe bloquear indefinidamente un worker.

48. Query Cancellation

Debe poder cancelarse:

client disconnect
request timeout
workflow cancellation
system shutdown

cuando el runtime lo permita.

49. Query Idempotency

Las Queries deben ser repetibles sin cambiar el estado observable.

Esto permite:

retry
cache
deduplication

cuando sean apropiados.

50. Query Side Effects

Por defecto:

Query = no business side effects

Evitar:

query → update
query → publish event
query → create entity
51. Technical Side Effects

Pueden existir efectos técnicos:

metrics
tracing
cache population
read-model refresh

pero deben ser transparentes para la semántica de lectura.

52. Query Caching

Puede utilizarse:

Query
 ↓
Cache
 ↓
Data Source

o:

Query
 ↓
Data Source
 ↓
Cache

según la estrategia.

53. Cache Key

Debe incorporar los elementos relevantes:

tenant
query
parameters
projection
version
authorization context

cuando sean necesarios para evitar data leakage.

54. Cache Security

Nunca compartir accidentalmente:

Tenant A Query Result

con:

Tenant B

ni:

User A

con:

User B

cuando el resultado dependa de autorización.

55. Query and Projection

La relación principal:

Query
 ↓
Projection
 ↓
Result

Query responde:

qué datos recuperar.

Projection responde:

qué representación devolver.

56. Query and Mapping

Puede existir:

Query
 ↓
Source
 ↓
Projection
 ↓
Mapping
 ↓
DTO

Mapping no debe utilizarse para ocultar una Query.

57. Query and Transformation

Transformation puede aparecer después de Query:

Query
 ↓
Result
 ↓
Transformation
 ↓
Consumer Model

pero debe mantenerse la separación de responsabilidades de E24.

58. Query and Serialization
Query
 ↓
Result
 ↓
Serialization
 ↓
JSON / XML / MessagePack / etc.

E23 controla la serialización.

59. Query and Projection Architecture

E26 y E27 forman una pareja fundamental:

E27 Query
   │
   │ retrieves
   ▼
E26 Projection
   │
   │ shapes
   ▼
Consumer Result
60. Query Data Sources

Una Query puede utilizar:

Relational Database
Document Store
Search Engine
Cache
Read Model
Event Store
Data Warehouse
External API

La elección debe ser explícita.

61. Primary Query Source

Cada Query debe identificar:

source of truth

cuando exista.

62. Read Model Query

Para sistemas CQRS:

Query
 ↓
Read Model
 ↓
Projection
 ↓
Result

El Query Model puede estar completamente separado del Domain Model.

63. CQRS Query Architecture
             COMMAND SIDE
                 │
                 ▼
             Domain Model
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
                 ▼
             QUERY SIDE
64. Query Consistency

Debe declararse si una Query utiliza:

strong consistency
read-after-write
eventual consistency
snapshot consistency
65. Read-After-Write

Cuando un usuario crea:

Order

y posteriormente consulta:

GetOrder

la arquitectura debe definir cuándo la Query garantiza observar el nuevo estado.

66. Consistency Token

Puede utilizarse:

version
sequence
event position

para coordinar:

write
 ↓
projection
 ↓
query
67. Query Freshness

Para read models eventual-consistent:

sourceVersion
projectedVersion
lag

pueden formar parte de la observabilidad.

68. Query Federation

Una Query puede federar:

Domain A
Domain B
Domain C

pero debe evitarse que la federación genere:

distributed monolith
69. Cross-Domain Query Boundary

Preferible:

Application / Query Layer
        │
   ┌────┼────┐
   ▼    ▼    ▼
   A    B    C
        │
        ▼
    Projection

en lugar de:

Domain A
   ↓
directly access
Domain B database
70. Database Boundary

Nunca debe permitirse:

Domain A
    ↓
Domain B tables

como mecanismo normal de integración.

71. Query Composition

Una Query compuesta puede ser:

CustomerDashboardQuery

que combina:

CustomerSummary
OrderSummary
PaymentSummary

La composición debe estar explícitamente modelada.

72. Query Fan-Out

Debe controlarse:

1 Query
 ↓
100 downstream queries

Esto puede causar:

latency amplification
failure amplification
cost amplification
73. Batch Query

Cuando sea posible:

100 individual queries

deben convertirse en:

1 batch query
74. Query Optimization

Debe optimizarse mediante:

indexes
query plans
projection pushdown
batching
caching
read replicas
materialized views
specialized read models
75. Index Strategy

Cada Query crítica debe analizar:

filter fields
sort fields
join fields
cardinality
selectivity

antes de introducir índices.

76. Query Plan

Las Queries críticas deben poder inspeccionarse mediante:

execution plan
latency
rows scanned
rows returned
index usage
77. Query Performance Metrics

Mínimo:

query_total
query_errors_total
query_duration
query_timeout_total
query_rows_read
query_rows_returned
78. Query Observability

Debe incluir:

query_name
query_version
source
projection
tenant_context
duration
result_size

sin registrar datos sensibles.

79. Query Tracing

Ejemplo:

HTTP Request
   │
   └── Query: GetCustomer
          │
          ├── Repository
          │
          └── Projection
80. Query Logging

Debe evitar:

password
tokens
secrets
full PII payload

en logs.

81. Query Auditing

Queries sensibles pueden requerir:

who
what
when
tenant
purpose
result scope

especialmente para:

financial
security
administrative
regulated

data.

82. Query Rate Limits

Queries públicas pueden necesitar:

requests/sec
requests/min
concurrency limit
cost limit
83. Query Complexity

Especialmente en APIs flexibles, debe existir:

max depth
max fields
max joins
max result size
max execution cost
84. Graph Query Protection

Si EVOXA expone consultas estructuradas como GraphQL o equivalentes:

depth limiting
complexity analysis
field allowlists
pagination enforcement

deben formar parte del boundary.

85. Query Cost Model

Una Query puede tener un coste estimado:

cost =
  fields
+ joins
+ cardinality
+ depth
+ fanout

Queries sobre un umbral pueden:

reject
throttle
require async execution
86. Async Query

Las Queries extremadamente costosas pueden convertirse en:

Query Request
 ↓
Job
 ↓
Processing
 ↓
Result

Esto conecta E27 con:

E15 — Job & Task Processing
E14 — Workflow & Orchestration
87. Query Scheduling

Una consulta periódica puede ser disparada por:

E16 — Scheduling

pero Query sigue siendo la operación de lectura.

88. Query Authorization Profiles

Puede existir:

PublicQuery
UserQuery
AdminQuery
SystemQuery
TenantQuery

pero deben mapearse a políticas reales.

89. Query Tenant Context

Toda Query multi-tenant debe resolver explícitamente:

tenantId

y aplicarlo en el boundary de datos.

90. Tenant Filter Enforcement

Nunca confiar exclusivamente en:

caller supplied tenantId

Debe derivarse o verificarse mediante:

authenticated context
authorization policy
tenant scope
91. Tenant Query Isolation

Debe garantizar:

Tenant A Query
    ↓
Tenant A Data

y nunca:

Tenant A Query
    ↓
all tenants

por defecto.

92. Query Context

Puede incluir:

tenant
user
roles
permissions
locale
timezone
requestId
traceId
consistencyRequirement
93. Query Context Discipline

No convertir el context en:

Global Bag

con dependencias implícitas.

Cada Query debe consumir sólo lo necesario.

94. Query Versioning

Puede requerirse:

GetCustomerV1
GetCustomerV2

cuando cambie el contrato de forma incompatible.

95. Query Compatibility

Cambios deben clasificarse:

non-breaking
behavior-changing
breaking
96. Query Evolution

Agregar un filtro opcional puede ser compatible:

status?

Cambiar la semántica de:

status

puede no serlo.

97. Query Deprecation

Debe indicar:

deprecatedSince
replacement
migrationPath
removalDate

cuando exista una API contractual.

98. Query Registry

Puede existir:

QueryRegistry

que resuelva:

queryType
handler
version
permissions
projection
99. Query Ownership

Cada Query crítica debe tener:

owner
domain
consumers
source
projection
SLA
100. Query Documentation

Debe documentar:

purpose
parameters
filters
sorting
pagination
authorization
source
projection
consistency
errors
limits
101. Query Testing

Debe cubrir:

valid query
invalid query
empty result
not found
authorization
tenant isolation
filters
sorting
pagination
timeouts
cancellation
performance
caching
consistency
versioning
102. Query Contract Tests

Los consumidores deben poder verificar:

request
response
error contract
pagination
field availability

sin depender de la implementación interna.

103. Query Integration Tests

Deben probar:

Query
 ↓
Repository
 ↓
Database / Read Model
 ↓
Projection
 ↓
Result
104. Query Performance Tests

Las Queries críticas deben tener objetivos:

P50
P95
P99
throughput
concurrency

según su SLA.

105. Query Failure Testing

Debe probar:

database unavailable
timeout
partial dependency failure
cache failure
read model stale
projection failure
106. Query Resilience

Puede utilizar:

timeouts
retry
circuit breakers
fallbacks
cache
read replicas

pero sólo cuando su semántica sea segura.

107. Retry Rules

Una Query read-only puede ser candidata a retry, pero:

retry × expensive query

puede amplificar la carga.

Debe existir:

retry budget
backoff
maximum attempts
108. Query Fallback

Un fallback puede utilizar:

cache
replica
materialized read model
previous snapshot

pero el resultado debe indicar o respetar la semántica de freshness.

109. Stale Data

Cuando un fallback devuelve datos antiguos:

stale=true

puede ser necesario según el contrato.

110. Query Security Principle

Una Query nunca debe ampliar el conjunto de datos que el consumidor está autorizado a consultar.

111. Query Performance Principle

Una Query debe recuperar sólo la información necesaria para producir su resultado.

112. Query Isolation Principle

Una Query no debe acceder directamente a datos pertenecientes a otro boundary sin un contrato explícito.

113. Query Determinism Principle

Dados los mismos datos, parámetros y contexto, una Query debe producir un resultado semánticamente equivalente.

114. Query Read-Only Principle

Una Query representa intención de lectura y no debe contener mutaciones de negocio ocultas.

115. Query Boundary Principle

La Query define qué se solicita; la Projection define qué representación se devuelve; la fuente de datos define de dónde se recupera.

116. Query Architecture Summary
                         QUERY ARCHITECTURE
                                │
             ┌──────────────────┼──────────────────┐
             ▼                  ▼                  ▼
          Contract           Context            Policy
             │                  │                  │
             └──────────────────┼──────────────────┘
                                ▼
                         Query Handler
                                │
                    ┌───────────┼───────────┐
                    ▼           ▼           ▼
                Repository   Read Model   Search
                    │           │           │
                    └───────────┼───────────┘
                                ▼
                           Projection
                                │
                                ▼
                           Query Result
117. Position in Engineering Specification

La secuencia queda:

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

La relación fundamental:

E23 Serialization
        │
        ▼
E24 Transformation
        │
        ▼
E25 Mapping
        │
        ▼
E26 Projection
        │
        ▼
E27 Query
        │
        ▼
E28 Read Model
        │
        ▼
E29 Search

Y conceptualmente:

                    CONSUMER
                       │
                       ▼
                    QUERY
                       │
             "what do I need?"
                       │
                       ▼
                  DATA SOURCE
                       │
                       ▼
                 PROJECTION
                       │
               "what do I expose?"
                       │
                       ▼
                 READ MODEL
                       │
                       ▼
                 RESULT MODEL
                       │
                       ▼
                SERIALIZATION

Principio central de E27:
Query expresa una intención de lectura, recupera únicamente los datos necesarios y entrega el resultado a través de una Projection apropiada. Debe permanecer separada de Commands, reglas de negocio, autorización, serialización y acceso arbitrario a datos.

Siguiente: E28 — EVOXA Read Model Architecture.
