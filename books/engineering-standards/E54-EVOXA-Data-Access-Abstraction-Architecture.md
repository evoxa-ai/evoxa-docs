E54 — EVOXA Data Access Abstraction Architecture
Engineering Specification

Pertenece a: Engineering Specification
Capítulo: E54 — Data Access Abstraction Architecture
Anterior: E53 — Data Virtualization Architecture
Siguiente: E55 — Data Access Governance Architecture

1. Propósito

E54 define la arquitectura mediante la cual EVOXA desacopla a los consumidores de las implementaciones concretas utilizadas para acceder a datos.

El principio fundamental es:

Data Access Abstraction proporciona contratos de acceso estables y orientados al dominio o caso de uso, ocultando detalles de persistencia, transporte, consultas, proveedores y mecanismos físicos de acceso.

E53 y E54 están relacionados, pero tienen responsabilidades distintas.

E53 — DATA VIRTUALIZATION
        │
        │ hides physical data topology
        ▼
Logical Data Surface
        │
        ▼
E54 — DATA ACCESS ABSTRACTION
        │
        │ hides access implementation
        ▼
Application / Domain Consumer

En términos simples:

Virtualization  → ¿Qué datos existen lógicamente?
Abstraction     → ¿Cómo accede el consumidor a ellos?
2. Objetivos

E54 debe permitir:

Consumer
   │
   ▼
Stable Access Contract
   │
   ▼
Abstract Data Access
   │
   ├── Repository
   ├── Query Service
   ├── Virtual Data Layer
   ├── API
   ├── Cache
   └── Physical Store

Los consumidores no deben depender directamente de:

SQL
ORM
HTTP
SDK
database driver
cache implementation
vendor API
storage engine
3. Problema que Resuelve

Sin abstracción:

Application
    │
    ├── SQL
    ├── ORM
    ├── Redis
    ├── REST
    └── vendor SDK

Esto produce:

high coupling
low testability
vendor lock-in
leaky persistence concerns
difficult migration
duplicated access logic

Con E54:

Application
    │
    ▼
Data Access Contract
    │
    ▼
Implementation
4. Principio de Dependency Inversion

La dirección correcta es:

Domain / Application
        │
        ▼
Abstract Data Contract
        ▲
        │
Concrete Implementation

No:

Domain
   │
   ▼
PostgreSQL Repository

sino:

Domain
   │
   ▼
CustomerReader
   ▲
   │
PostgresCustomerReader
5. Scope

E54 cubre:

Data Access Contracts
Access Interfaces
Repositories
Query Interfaces
Command Interfaces
Read Access
Write Access
Data Providers
Adapters
Access Strategies
Implementation Binding
Dependency Injection
Unit of Work Boundaries
Transaction Abstraction
Pagination
Streaming
Caching Integration
Retry Semantics
Timeouts
Consistency
Concurrency
Authorization Context
Tenant Context
Observability
Testing
Lifecycle
Versioning
6. Non-Goals

E54 no define por sí mismo:

database schema
physical storage
replication
data federation
data virtualization
business rules
workflow orchestration
domain decisions

Esas responsabilidades pertenecen a otras arquitecturas.

7. Relationship with E53

E53:

Consumer
   │
   ▼
Virtual Data Surface
   │
   ▼
Multiple Physical Sources

E54:

Application Service
   │
   ▼
Access Contract
   │
   ▼
Data Access Implementation

Una implementación E54 puede utilizar E53:

Application Service
       │
       ▼
CustomerReader
       │
       ▼
Virtual Data Layer
       │
       ▼
Physical Sources

Por tanto:

E53 abstrae la estructura/topología de los datos; E54 abstrae el mecanismo mediante el cual el software accede a ellos.

8. Core Access Contract

Un contrato debe expresar intención:

CustomerReader
├── getById()
├── findByEmail()
└── search()

y no detalles físicos:

executeSQL()
prepareStatement()
openConnection()
9. Intent-Oriented Access

Preferido:

getCustomer(customerId)

sobre:

execute(
    "SELECT * FROM customers WHERE id = ?"
)

La primera expresión representa una capacidad del sistema.

La segunda expone una implementación.

10. Access Contract Layers
                    Consumer
                       │
                       ▼
              ┌──────────────────┐
              │ Access Contract   │
              └────────┬─────────┘
                       │
          ┌────────────┼────────────┐
          ▼            ▼            ▼
       Reader        Writer       Query
          │            │            │
          └────────────┼────────────┘
                       ▼
                Access Adapter
                       │
          ┌────────────┼────────────┐
          ▼            ▼            ▼
       Database        API         Virtual
11. Reader Abstraction

Un reader representa acceso de lectura:

CustomerReader
OrderReader
ProductReader
AccountReader

Ejemplo conceptual:

CustomerReader.getById(id)

No debe revelar:

Postgres
Mongo
REST
GraphQL
12. Writer Abstraction

Un writer representa operaciones de escritura:

CustomerWriter
OrderWriter
AccountWriter

Ejemplo:

CustomerWriter.save(customer)

La implementación puede ser:

SQL
API
event-backed
service-backed
13. Query Abstraction

Cuando las consultas son complejas:

CustomerQuery
OrderQuery
TransactionQuery

pueden exponer operaciones especializadas.

Ejemplo:

findActiveCustomers(criteria)

en lugar de:

find(
    table,
    filters,
    joins,
    projections
)
14. Repository Abstraction

Un Repository proporciona una interfaz de acceso coherente para un agregado o entidad cuando corresponde al modelo.

CustomerRepository
├── get()
├── add()
├── update()
└── remove()

No toda lectura debe convertirse automáticamente en Repository.

15. Repository vs Data Access Service
Repository

Orientado a:

domain entity
aggregate
Data Access Service

Orientado a:

query
integration
projection
external dataset

Ejemplo:

CustomerRepository
CustomerSearchService
CustomerProjectionReader

No deben mezclarse arbitrariamente.

16. Query-Specific Access

Para reporting o read models:

CustomerSummaryReader
OrderHistoryReader
AccountBalanceReader

puede ser mejor que un repository genérico.

17. Generic Repository

EVOXA debe evitar abusar de:

GenericRepository<T>

porque suele ocultar diferencias semánticas importantes.

Un contrato explícito:

CustomerRepository

puede comunicar mucho más que:

Repository<Customer>
18. Access Adapter

El adapter traduce el contrato abstracto a una implementación concreta:

CustomerReader
      │
      ▼
PostgresCustomerReader
      │
      ▼
PostgreSQL

o:

CustomerReader
      │
      ▼
ApiCustomerReader
      │
      ▼
Customer API
19. Adapter Responsibilities

Un adapter puede manejar:

protocol translation
schema mapping
serialization
connection management
error translation
pagination
retry
timeout
authentication

No debería contener reglas de negocio del dominio.

20. Data Access Boundary

La frontera debe ser explícita:

┌─────────────────────────────┐
│ Application / Domain        │
│                             │
│ Access Contract             │
└──────────────┬──────────────┘
               │
───────────────┼────────────────
               │ Access Boundary
───────────────┼────────────────
               │
┌──────────────▼──────────────┐
│ Infrastructure              │
│                             │
│ Adapter / Repository Impl.  │
└─────────────────────────────┘
21. Dependency Injection

Las implementaciones deben resolverse mediante DI:

CustomerReader
       ▲
       │
PostgresCustomerReader

El consumidor solicita:

CustomerReader

no:

PostgresCustomerReader
22. Runtime Binding

El binding puede variar:

Development
   → MockCustomerReader

Test
   → InMemoryCustomerReader

Production
   → PostgresCustomerReader
23. Multi-Provider Binding

Un mismo contrato puede tener:

CustomerReader
   ├── PrimaryReader
   ├── ReplicaReader
   ├── CacheReader
   └── VirtualReader

La selección debe estar gobernada por política.

24. Context-Aware Access

La implementación puede depender de:

tenant
region
environment
consistency mode
security context
workload

Ejemplo:

CustomerReader
      │
      ▼
Tenant Routing
      │
      ├── Tenant A → DB A
      └── Tenant B → DB B
25. Tenant Isolation

La abstracción nunca debe eliminar el contexto de tenant.

Toda operación debe poder determinar:

tenantId

cuando los datos sean tenant-scoped.

26. Access Context

Conceptualmente:

AccessContext
├── tenantId
├── principal
├── authorization
├── correlationId
├── consistency
├── deadline
└── locale

No todos los contratos necesitan exponer todos estos elementos directamente.

27. Authorization

Data access no debe asumir que:

authenticated = authorized

La autorización puede ocurrir:

Application Layer
      │
      ▼
Authorization Policy
      │
      ▼
Data Access

y, cuando corresponda:

Data Access
      │
      ▼
Source Authorization
28. Row-Level Security

Un reader debe preservar filtros de seguridad:

CustomerReader
      │
      ▼
tenantId = currentTenant

Nunca debe depender únicamente de que el caller haya agregado el filtro manualmente.

29. Field-Level Security

La abstracción puede devolver:

CustomerPublicView

en lugar de:

CustomerInternalModel

cuando exista información restringida.

30. Domain Object vs Data Object

No debe asumirse que:

DatabaseRow = DomainEntity

Puede existir:

Database Row
    │
    ▼
Data Model
    │
    ▼
Mapper
    │
    ▼
Domain Entity
31. Mapping Boundary
Physical Data
      │
      ▼
Persistence Model
      │
      ▼
Mapper
      │
      ▼
Domain/Application Model

El mapper evita que detalles físicos penetren en el dominio.

32. DTO Access

Para consultas especializadas:

CustomerSummaryDto
OrderHistoryDto
AccountBalanceDto

pueden evitar cargar entidades completas.

33. Projection Access

Un reader puede solicitar sólo:

id
name
status

en lugar de:

full customer record

Esto reduce:

IO
network
memory
serialization
34. Query Specification

Para consultas dinámicas:

CustomerQuery
├── status
├── createdAfter
├── segment
├── limit
└── cursor

El contrato debe controlar las capacidades permitidas.

No debe convertirse en:

raw SQL passthrough
35. Query Object

Una query puede representar:

FindActiveCustomers

y transportar criterios:

segment
region
limit
cursor

La implementación decide cómo ejecutarla.

36. Query Translation
Abstract Query
      │
      ▼
Query Translator
      │
      ▼
SQL / API / Search / Virtual Query
37. Query Capability

Cada implementation puede declarar:

supportsFiltering
supportsSorting
supportsPagination
supportsProjection
supportsStreaming

El contrato no debe prometer capacidades que la implementación no puede cumplir.

38. Pagination

E54 debe abstraer:

offset
page
cursor
continuation token

cuando sea necesario.

Preferencia general:

cursor-based pagination

para grandes datasets y APIs distribuidas, cuando el source lo permita.

39. Pagination Contract

Ejemplo conceptual:

Page<T>
├── items
├── nextCursor
└── hasMore

La representación física del cursor debe permanecer interna cuando sea posible.

40. Streaming

Para datasets grandes:

Reader
  │
  ▼
Stream<T>

permite:

bounded memory
incremental processing
backpressure
41. Batch Access

Debe existir soporte para operaciones como:

getMany(ids)

cuando el workload lo requiere.

Esto evita:

N + 1 access
42. Bulk Operations

Operaciones como:

bulkCreate
bulkUpdate
bulkRead

deben tener contratos explícitos.

No deben ocultar costes operativos elevados.

43. Transaction Abstraction

Cuando una operación requiere atomicidad:

Transaction
   │
   ├── read
   ├── write
   └── commit

debe abstraerse del mecanismo concreto.

44. Unit of Work

Cuando el modelo lo requiere:

UnitOfWork
├── track
├── flush
└── commit

puede encapsular la coordinación de cambios.

45. Transaction Boundary

La transacción debe pertenecer al límite apropiado:

Application Service
       │
       ▼
Transaction Boundary
       │
       ├── Repository
       ├── Repository
       └── Repository

No debe abrirse una transacción global por defecto para cada operación.

46. Distributed Transactions

E54 no debe asumir que puede crear:

Transaction
   │
   ├── DB A
   ├── API B
   └── DB C

como una transacción ACID única.

Cuando no sea posible:

workflow
saga
outbox
compensation

pertenecen a otras arquitecturas.

47. Consistency Modes

Un contrato puede declarar:

STRONG
READ_YOUR_WRITES
EVENTUAL
BOUNDED_STALENESS
BEST_EFFORT

según lo soportado.

48. Read-After-Write

Cuando una operación requiere:

write()
   │
   ▼
read()

el access layer debe permitir seleccionar una ruta que garantice la semántica requerida.

49. Replica Awareness

Si existen replicas:

Writer → Primary
Reader → Replica

pero después de escribir:

Reader → Primary

cuando sea necesario para read-after-write.

50. Caching

El acceso puede utilizar:

Application Cache
Data Access Cache
Virtualization Cache
Source Cache

pero la cache no debe cambiar silenciosamente la semántica de consistencia.

51. Cache-aside

Patrón:

Reader
  │
  ▼
Cache
  │
  ├── Hit → return
  │
  └── Miss
       │
       ▼
      Source
52. Cache Key

Debe incorporar los elementos necesarios:

tenant
entity
identifier
projection
security scope
version

para evitar data leakage.

53. Cache Invalidation

Debe soportar:

TTL
explicit invalidation
event-driven invalidation
version checking

según la fuente de verdad.

54. Error Abstraction

Los consumidores no deberían recibir errores específicos del driver:

SQLException
RedisError
AxiosError
VendorSDKException

sino errores semánticos:

DataNotFound
DataConflict
AccessDenied
DataUnavailable
DataTimeout
DataConstraintViolation
55. Error Translation
Physical Error
      │
      ▼
Adapter
      │
      ▼
EVOXA Access Error
      │
      ▼
Application
56. Error Categories

Mínimo:

NOT_FOUND
CONFLICT
INVALID_REQUEST
UNAUTHORIZED
FORBIDDEN
TIMEOUT
UNAVAILABLE
RATE_LIMITED
DEPENDENCY_FAILURE
DATA_CORRUPTION
UNKNOWN
57. Retry Semantics

El contrato debe distinguir:

retryable
non-retryable
unknown

No todos los errores deben reintentarse.

58. Timeout Propagation

El consumer puede establecer:

deadline = T

y la implementación debe respetarlo.

Application
   │ deadline
   ▼
Access Layer
   │
   ▼
Source
59. Cancellation

Si el consumer cancela:

Request
   X
Access
   X
Source operation

la cancelación debe propagarse cuando el protocolo lo permita.

60. Concurrency

Debe controlarse:

max concurrent reads
max concurrent writes
max concurrent source calls

por:

application
tenant
source
operation

cuando sea necesario.

61. Backpressure

Para streaming:

Consumer
   │ slow
   ▼
Access Layer
   │
   ▼
Source

debe evitarse acumulación ilimitada.

62. Connection Management

La infraestructura de acceso puede administrar:

connection pool
connection lifetime
idle timeout
max connections
health checks

sin exponer esos detalles al dominio.

63. Health Checks

Los adapters pueden proporcionar:

health
readiness
connectivity
dependency status

pero health checks no deben ejecutar queries costosas indiscriminadamente.

64. Access Policy

Cada access contract puede estar sujeto a:

authorization
rate limit
timeout
consistency
cache
data classification
65. Access Policy Resolution
Request
   │
   ▼
Policy Resolver
   │
   ├── authorization
   ├── timeout
   ├── consistency
   ├── cache
   └── rate limit
66. Rate Limiting

Debe evitarse que una aplicación consuma ilimitadamente:

CustomerReader

La limitación puede ser:

tenant
application
user
source
operation
67. Observability

Cada access operation debe generar contexto suficiente:

operation
entity
implementation
source
tenant
latency
result
error
68. Metrics

Mínimo:

access_requests
access_success
access_failures
access_latency
timeouts
retries
cache_hits
cache_misses
not_found
conflicts
69. Tracing

Trace:

API
 │
 ▼
Application Service
 │
 ▼
CustomerReader
 │
 ▼
Repository
 │
 ▼
Database

debe poder correlacionarse.

70. Logging

No deben registrarse indiscriminadamente:

PII
credentials
tokens
secrets
full query payloads
sensitive data

Los logs deben respetar clasificación y masking.

71. Audit

Debe registrarse cuando corresponda:

who accessed
what logical data
when
tenant
purpose
result

especialmente para datos sensibles.

72. Performance Observability

Debe ser posible separar:

contract latency
mapping latency
adapter latency
source latency
serialization latency
73. Data Access Cost

Para workloads relevantes:

read count
write count
bytes transferred
source compute
cache usage

pueden medirse por:

tenant
application
service
operation
74. Testability

Una de las principales razones de E54 es:

Domain
   │
   ▼
Interface
   ▲
   │
Mock / Fake

El dominio no necesita levantar:

database
API
cache

para pruebas unitarias.

75. Test Double Types

Se pueden utilizar:

Mock
Stub
Fake
InMemory
Contract Test Adapter

según el objetivo.

76. Contract Testing

Cada implementation debe demostrar:

implements contract
preserves semantics
maps errors correctly
respects pagination
respects authorization
respects consistency
77. Integration Testing

Debe probarse contra la dependencia real:

Access Contract
      │
      ▼
Real Adapter
      │
      ▼
Real Dependency
78. Mutation Testing

Las pruebas deben detectar errores como:

wrong tenant
missing filter
incorrect mapping
incorrect pagination
lost authorization
79. Access Contract Versioning

Un contrato puede evolucionar:

CustomerReader v1
CustomerReader v2

Los cambios incompatibles deben tener estrategia de migración.

80. Backward Compatibility

No se deben eliminar silenciosamente:

fields
operations
semantics
error behavior
pagination contract

que consumidores todavía utilicen.

81. Consumer-Driven Contracts

Cuando el access contract tiene múltiples consumers:

Consumer A
Consumer B
Consumer C
      │
      ▼
Access Contract

los cambios deben considerar sus expectativas.

82. Lifecycle

Un access contract puede evolucionar:

DRAFT
   ↓
ACTIVE
   ↓
DEPRECATED
   ↓
RETIRED
83. Implementation Lifecycle

Una implementación concreta puede tener:

REGISTERED
VALIDATING
ACTIVE
DEGRADED
DRAINING
DISABLED
RETIRED

Esto permite migraciones controladas.

84. Implementation Swapping

Una ventaja crítica:

CustomerReader
     │
     ├──► PostgreSQL
     │
     └──► New Data Platform

La aplicación permanece estable.

85. Migration Pattern
Old Implementation
       │
       ├──────────────┐
       ▼              ▼
Old Reader        New Reader
       │              │
       └──────┬───────┘
              ▼
       Contract Validation
              │
              ▼
        Traffic Shift
              │
              ▼
       New Implementation
86. Shadow Reads

Durante migraciones:

Primary Reader → Result A
Shadow Reader  → Result B

se pueden comparar:

A == B

sin cambiar el resultado visible.

87. Dual Read

Puede utilizarse temporalmente:

Read Old
Read New
Compare
Return Primary

Debe existir control de coste.

88. Access Failover
Primary Adapter
      │
      X
      ▼
Secondary Adapter

Debe preservar:

security
consistency
freshness
semantics
89. Fallback Constraints

No debe hacerse:

Strong Source unavailable
       │
       ▼
Stale Cache
       │
       ▼
Pretend Strong

El resultado debe indicar la degradación cuando sea relevante.

90. Data Freshness Contract

Un reader puede declarar:

freshness = LIVE

o:

freshness = <= 5 minutes

según su implementación.

91. Provenance

Cuando el consumer necesita trazabilidad:

Access Result
├── source
├── retrievedAt
├── version
└── origin

debe estar disponible.

92. Access Result Metadata

Una respuesta puede conceptualmente incluir:

Data
Metadata
├── source
├── freshness
├── consistency
├── completeness
└── correlationId
93. Completeness

Especialmente en acceso compuesto:

complete
partial
unknown

debe poder representarse.

94. Data Quality

Cuando corresponda:

qualityStatus
validationWarnings
sourceQuality

pueden formar parte de metadata.

95. Security Context

La implementación debe poder recibir el contexto necesario:

tenant
principal
roles
permissions
purpose

pero no debe permitir que el caller lo falsifique arbitrariamente.

96. Trusted Context

El runtime debe establecer:

Authenticated Principal

antes de ejecutar acceso sensible.

No debe confiar simplemente en:

request.body.userId

para determinar autorización.

97. Tenant Context Propagation
Request
   │
   ▼
Tenant Context
   │
   ▼
Access Contract
   │
   ▼
Adapter
   │
   ▼
Source
98. Data Residency Routing

El access implementation puede resolver:

Tenant EU → EU Source
Tenant US → US Source

cuando lo requieran las políticas.

99. Regional Failover

El failover regional no debe romper:

tenant isolation
residency
consistency
authorization
100. Access Abstraction and AI

Los agentes de EVOXA deben preferir:

CustomerReader
OrderReader
AccountReader

sobre:

raw SQL
raw HTTP
vendor SDK

Esto reduce:

tool complexity
security surface
prompt complexity
implementation coupling
101. Agent Tool Boundary
AI Agent
   │
   ▼
Typed Data Access Tool
   │
   ▼
Access Contract
   │
   ▼
Authorized Implementation

El agente no debe recibir credenciales de infraestructura.

102. Query Safety for AI

Un agente no debería poder producir:

SELECT *
FROM arbitrary_table

si el access layer sólo necesita permitir:

CustomerReader.search(criteria)
103. Application Service Integration
Application Service
       │
       ▼
CustomerReader
       │
       ▼
Data Access Implementation

La aplicación expresa intención; infraestructura resuelve mecanismo.

104. Domain Service Integration

Un Domain Service puede depender de:

CustomerLookup

si ese acceso es necesario para una regla de dominio.

Debe evitar depender de:

DatabaseConnection
105. Repository and Domain Model

Cuando exista un Aggregate:

Domain Service
      │
      ▼
CustomerRepository
      │
      ▼
Persistence Adapter

La entidad de dominio permanece independiente del persistence technology.

106. Access and Commands

No todo acceso de escritura debe pasar por un repository.

Para acciones:

SuspendCustomer
CloseAccount
ApproveOrder

puede ser mejor:

Application Command
      │
      ▼
Domain / Application Service
      │
      ▼
Authoritative Write Mechanism

E54 debe evitar convertirse en una capa genérica de CRUD que ignore el dominio.

107. Read/Write Separation

Puede existir:

CustomerReader
CustomerWriter

con implementaciones diferentes:

Reader → Replica / Read Model
Writer → Primary
108. CQRS Compatibility

E54 puede soportar:

Command
   │
   ▼
Write Model

Query
   │
   ▼
Read Model

sin imponer CQRS a todos los dominios.

109. Access to Read Models

Para consultas optimizadas:

Application
   │
   ▼
CustomerSummaryReader
   │
   ▼
Read Model

Esto evita reconstruir entidades completas innecesariamente.

110. Access to External Systems
Application
   │
   ▼
ExternalCustomerReader
   │
   ▼
Integration Adapter
   │
   ▼
External System

El external protocol permanece fuera del dominio.

111. Anti-Corruption Layer

E54 puede servir como una frontera:

External System
       │
       ▼
Adapter
       │
       ▼
EVOXA Access Contract
       │
       ▼
EVOXA Application

El modelo externo no invade el modelo interno.

112. Access Composition

Un access service puede componer:

CustomerReader
AccountReader
SubscriptionReader

pero la composición debe estar controlada.

No debe crear accidentalmente:

unbounded fan-out
113. Access Orchestration

Si la composición requiere:

multiple calls
retry
compensation
state
long-running execution

debe pasar a:

Workflow / Orchestration Architecture

y no crecer indefinidamente dentro del data access layer.

114. Access Boundaries

Regla:

Simple data retrieval
        → E54

Complex business workflow
        → E14

Business decision
        → E33

Action
        → E34

Execution
        → E35
115. Serialization Boundary

E54 puede transformar:

External JSON

en:

Access Model

pero la serialización general pertenece a E23.

116. Transformation Boundary

Transformaciones necesarias para acceso:

physical → access model

son válidas.

Transformaciones de negocio complejas:

business transformation

no deben esconderse dentro del adapter.

117. Mapping Boundary

E25 puede proporcionar:

mapping framework

mientras E54 consume esos mappings para implementar contratos de acceso.

118. Projection Boundary

E26 puede definir:

projection semantics

E54 puede exponer:

CustomerSummaryReader

basado en esas proyecciones.

119. Query Boundary

E27 define query semantics.

E54 define:

how consumers access those queries
120. Read Model Boundary

E28 define read models.

E54 proporciona contratos para acceder a ellos:

OrderHistoryReader

sin exponer:

read_model_table
121. Search Boundary

E29 puede proporcionar:

ProductSearch

E54 puede envolverlo mediante:

ProductSearchReader

cuando el consumidor necesite una abstracción estable.

122. Reporting Boundary

E30 puede exponer:

ReportDataReader

sin que las aplicaciones conozcan:

warehouse schema
123. Analytics Boundary

E31 puede proporcionar datasets analíticos.

E54 puede abstraer su acceso para consumidores internos autorizados.

124. Core Access Model
                     ┌───────────────────┐
                     │ Consumer          │
                     └─────────┬─────────┘
                               │
                               ▼
                    ┌────────────────────┐
                    │ Access Contract    │
                    └─────────┬──────────┘
                              │
                   ┌──────────┼──────────┐
                   ▼          ▼          ▼
                Reader      Query      Writer
                   │          │          │
                   └──────────┼──────────┘
                              ▼
                    ┌────────────────────┐
                    │ Access Policy      │
                    ├────────────────────┤
                    │ Auth               │
                    │ Tenant             │
                    │ Consistency        │
                    │ Timeout            │
                    │ Cache              │
                    └─────────┬──────────┘
                              ▼
                    ┌────────────────────┐
                    │ Adapter            │
                    └─────────┬──────────┘
                              │
                ┌─────────────┼─────────────┐
                ▼             ▼             ▼
             Database        API          Virtual
125. Runtime Access Flow
Request
   │
   ▼
Authenticate
   │
   ▼
Resolve Tenant
   │
   ▼
Authorize
   │
   ▼
Resolve Access Contract
   │
   ▼
Resolve Implementation
   │
   ▼
Apply Policy
   │
   ▼
Execute
   │
   ▼
Map Result
   │
   ▼
Attach Metadata
   │
   ▼
Return
126. Access Event Model

Eventos relevantes:

AccessContractCreated
AccessContractUpdated
AccessContractDeprecated
AccessContractRetired

AccessImplementationRegistered
AccessImplementationActivated
AccessImplementationDeactivated

DataAccessStarted
DataAccessCompleted
DataAccessFailed
DataAccessTimedOut
DataAccessCancelled

DataAccessDenied
DataAccessRateLimited
DataAccessDegraded

DataAccessCacheHit
DataAccessCacheMiss

DataAccessFallbackActivated
127. Operational States
Contract
DRAFT
ACTIVE
DEPRECATED
RETIRED
Implementation
REGISTERED
VALIDATING
ACTIVE
DEGRADED
DRAINING
DISABLED
RETIRED
Operation
RECEIVED
AUTHORIZED
RESOLVED
EXECUTING
COMPLETED
FAILED
TIMED_OUT
CANCELLED
128. Core Invariants
Invariant 1 — Contract First

Los consumidores dependen de contratos de acceso, no de implementaciones concretas.

Invariant 2 — No Persistence Leakage

Los detalles de persistencia no deben filtrarse al dominio o aplicación salvo decisión explícita.

Invariant 3 — Explicit Semantics

El contrato debe definir claramente las garantías de lectura, escritura, consistencia y errores.

Invariant 4 — Security Preservation

La abstracción nunca puede ampliar privilegios.

Invariant 5 — Tenant Isolation

Todo acceso tenant-scoped debe preservar el contexto de tenant.

Invariant 6 — Bounded Access

Toda operación debe estar limitada por tiempo, concurrencia, cardinalidad y recursos.

Invariant 7 — Error Translation

Los errores físicos deben traducirse a errores semánticos apropiados.

Invariant 8 — Implementation Replaceability

Cuando el contrato lo permita, una implementación debe poder reemplazarse sin modificar consumidores.

Invariant 9 — No Hidden Business Logic

Los adapters no deben convertirse en contenedores de reglas de negocio.

Invariant 10 — Explicit Consistency

El access layer no puede prometer garantías que la implementación no soporte.

Invariant 11 — Observable Access

Las operaciones deben poder observarse y diagnosticarse sin exponer datos sensibles.

Invariant 12 — Testable Boundary

Los contratos de acceso deben poder probarse independientemente de las dependencias físicas.

129. Reference Example
                    Application Service
                           │
                           ▼
                    CustomerReader
                           │
                           ▼
                    Access Policy
                           │
              ┌────────────┼────────────┐
              ▼            ▼            ▼
          Tenant       Authorization   Deadline
              │            │            │
              └────────────┼────────────┘
                           ▼
                  CustomerReaderImpl
                           │
                ┌──────────┼──────────┐
                ▼          ▼          ▼
             Cache      Virtual      API
                         Layer
                           │
                           ▼
                       CRM / DB
130. Completion Criteria

E54 se considera completo cuando EVOXA dispone de:

✓ Stable access contracts
✓ Reader abstractions
✓ Writer abstractions
✓ Query abstractions
✓ Repository boundaries
✓ Data access services
✓ Adapter architecture
✓ Dependency inversion
✓ Dependency injection
✓ Runtime implementation binding
✓ Tenant-aware access
✓ Authorization integration
✓ Field-level security
✓ Row-level security
✓ Persistence isolation
✓ Data-to-domain mapping
✓ DTO/projection access
✓ Query specification
✓ Pagination abstraction
✓ Streaming
✓ Batch access
✓ Bulk operations
✓ Transaction abstraction
✓ Unit of Work where required
✓ Consistency modes
✓ Read-after-write semantics
✓ Replica awareness
✓ Cache integration
✓ Cache isolation
✓ Error abstraction
✓ Retry semantics
✓ Timeout propagation
✓ Cancellation
✓ Concurrency controls
✓ Backpressure
✓ Connection management
✓ Health checks
✓ Access policies
✓ Rate limiting
✓ Observability
✓ Metrics
✓ Distributed tracing
✓ Secure logging
✓ Audit
✓ Performance monitoring
✓ Test doubles
✓ Contract testing
✓ Integration testing
✓ Migration support
✓ Implementation swapping
✓ Shadow reads where required
✓ Failover
✓ Freshness metadata
✓ Provenance
✓ AI access boundary
✓ Application integration
✓ Domain integration
✓ External system adapters
✓ Anti-corruption boundary
✓ Core invariants
131. Principio Rector de E54

Data Access Abstraction es la frontera que permite a EVOXA expresar acceso a datos mediante contratos estables, semánticos y testeables, mientras las implementaciones concretas —bases de datos, APIs, caches, servicios, modelos virtuales o proveedores externos— permanecen intercambiables y aisladas de los consumidores.

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
       │
       ▼
E54 — DATA ACCESS ABSTRACTION

La distinción esencial:

MIGRATION       → move data
SYNCHRONIZATION → align data
REPLICATION     → duplicate data
FEDERATION      → compose data
VIRTUALIZATION  → abstract data topology
ACCESS ABSTRACTION
                → abstract data access

E53 hace que los datos parezcan una superficie lógica unificada; E54 hace que el software pueda acceder a esa superficie mediante contratos estables sin conocer cómo se implementa físicamente el acceso.
