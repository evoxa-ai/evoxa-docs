E26 — EVOXA Projection Architecture
Engineering Specification

Pertenece a: Engineering Specification
Volumen: volume-09-engineering
Capítulo: E26 — Projection Architecture
Anterior: E25 — Mapping Architecture
Siguiente: E27 — Query Architecture

1. Propósito

E26 define la arquitectura de Projection de EVOXA.

Projection establece cómo EVOXA selecciona, organiza y presenta una vista derivada de un modelo fuente, orientada a un propósito concreto, sin convertir esa vista en el modelo canónico ni introducir lógica de negocio indebida.

La pregunta fundamental de Projection es:

¿Qué parte del modelo necesitamos exponer para este consumidor, consulta, pantalla, proceso o boundary?

2. Concepto Fundamental

Projection:

Source Model
      │
      ▼
   Projection
      │
      ▼
Purpose-specific View

Ejemplo:

Customer
├── id
├── name
├── email
├── address
├── billing
├── security
└── metadata

        ↓ Projection

CustomerSummary
├── id
└── name

La projection selecciona y organiza información existente.

3. Projection vs Mapping

Mapping:

firstName → first_name

Projection:

Customer
   ↓
[id, name]

Por tanto:

E25 — Mapping
    correspondence

E26 — Projection
    selection / view

Una projection puede utilizar mappings para construir su resultado.

4. Projection vs Transformation

Transformation:

Order
 ↓
OrderSummary

puede implicar:

normalization
calculation
conversion
enrichment

Projection:

Order
 ↓
select:
id
status
total

Su objetivo primario es seleccionar una representación derivada.

5. Projection Architecture
                         Projection Control Plane
                                  │
              ┌───────────────────┼───────────────────┐
              ▼                   ▼                   ▼
           Schemas             Profiles            Versions
              │                   │                   │
              └───────────────────┼───────────────────┘
                                  ▼
                         Projection Runtime
                                  │
            ┌─────────────────────┼─────────────────────┐
            ▼                     ▼                     ▼
        Selector               Builder               Projector
            │                     │                     │
            └─────────────────────┼─────────────────────┘
                                  ▼
                          Projected Model
6. Projection Boundary

El boundary:

Source Model
      ↓
Projection Definition
      ↓
Projected Model

debe declarar:

source
target
fields
nested projections
profile
version
context
7. Projection Contract

Conceptualmente:

project(source, context)

produce:

ProjectedModel

o:

ProjectionError
8. Projection Characteristics

Una projection puede declararse:

lossy
deterministic
read_only
purpose_specific
versioned
cacheable

Normalmente es:

one-way

No debe asumirse:

Projection → Source

como operación reversible.

9. Read Model

Una projection puede constituir un:

Read Model

especialmente cuando está optimizada para:

Queries
Dashboards
Search
Lists
Reports
APIs
10. Projection Types

EVOXA debe contemplar:

Entity Projection
Summary Projection
Detail Projection
List Projection
Search Projection
Dashboard Projection
API Projection
Event Projection
Persistence Projection
Analytics Projection
Security Projection
Tenant Projection
11. Entity Projection
Customer
 ↓
CustomerView

Puede contener los campos necesarios para representar una entidad.

12. Summary Projection
Customer
 ↓
CustomerSummary

Ejemplo:

id
displayName
status

Está diseñada para reducir payload.

13. Detail Projection
Customer
 ↓
CustomerDetail

Puede incluir:

profile
address
preferences
status

pero no debe incluir automáticamente todos los campos internos.

14. List Projection

Para colecciones:

Customers
 ↓
CustomerListItem[]

Normalmente contiene sólo:

id
name
status
15. Search Projection

Optimizada para:

Search
Filtering
Sorting
Autocomplete
Indexing

Puede tener una estructura diferente del modelo de dominio.

16. Dashboard Projection

Puede combinar información necesaria para una vista:

Customer
Orders
Payments
Risk

→

CustomerDashboard

Si requiere múltiples fuentes, la orquestación debe permanecer fuera del projector puro.

17. API Projection
Domain Result
      ↓
API Projection
      ↓
Response DTO

Protege el dominio frente al contrato público.

18. Event Projection
Domain Event
      ↓
Public Event Projection
      ↓
Integration Event

Permite controlar qué información se publica.

19. Persistence Projection

Puede existir una proyección específica para almacenamiento:

Domain Model
      ↓
Persistence Projection
      ↓
Storage Model

Aunque el acceso a almacenamiento sigue siendo responsabilidad del Repository.

20. Analytics Projection

Puede seleccionar datos para:

Reporting
Analytics
Data Warehouse
Metrics
BI

Debe aplicar minimización y políticas de privacidad.

21. Security Projection

Una projection puede producir una vista segura:

Internal User
      ↓
Public User Projection

Excluyendo:

password
tokens
internal permissions
security metadata
22. Tenant Projection

Debe preservar el aislamiento:

Tenant A Data
      ↓
Tenant A Projection

Nunca:

Tenant A + Tenant B

sin una operación explícitamente autorizada.

23. Projection Field Selection

La unidad básica:

source.field
      ↓
projected.field

Ejemplo:

customer.id
customer.name
customer.status
24. Projection Allowlist

Para boundaries externos:

Allowed Fields

debe ser el mecanismo preferido.

Evitar:

Expose Everything
25. Projection Denylist

Una denylist:

exclude:
password
token
secret

puede ser útil como defensa adicional, pero no debería ser el único mecanismo de seguridad.

26. Explicit Projection

Preferido:

CustomerPublicProjection

con:

id
displayName
status

en lugar de:

Customer → serialize all
27. Nested Projection
Customer
 ├── id
 ├── name
 └── address
      ├── city
      └── country

→

CustomerView
 ├── id
 ├── name
 └── address
      ├── city
      └── country

Puede componerse mediante:

AddressProjection
28. Nested Projection Reuse

Ejemplo:

Address
 ↓
AddressSummary

puede reutilizarse en:

CustomerSummary
OrderSummary
SupplierSummary

si el contrato es realmente compartido.

29. Projection Composition
CustomerProjection
      +
OrderProjection
      ↓
CustomerOrdersProjection

Debe evitarse la composición indiscriminada.

30. Projection Aggregation

Una projection puede combinar datos:

Customer
+
OrderSummary[]

pero si requiere consultas a múltiples agregados, la coordinación debe pertenecer a:

Application Service
Query Service
Read Model Builder

según el diseño.

31. Projection and Query

La relación natural es:

Query
 ↓
Data Retrieval
 ↓
Projection
 ↓
Result

No:

Projection
 ↓
arbitrary database access

salvo que sea explícitamente una Query Projection especializada.

32. Projection and Repository

Repository:

retrieve
persist

Projection:

shape result

Pueden colaborar:

Repository
    ↓
Source Data
    ↓
Projection
    ↓
Read Model
33. Query Projection

Para consultas de alto rendimiento:

Query
 ↓
Repository / Query Service
 ↓
Projection
 ↓
DTO

La consulta puede seleccionar directamente los campos necesarios.

34. Database Projection

Cuando el storage lo permita:

SELECT
    id,
    name,
    status

puede evitar cargar:

metadata
history
security_context

Esto mejora:

I/O
Memory
Latency
35. Projection Pushdown

Una projection puede ejecutarse:

Application Layer

o ser empujada hacia:

Database
Search Engine
Data Warehouse

si el backend soporta la operación sin alterar la semántica.

36. Projection Pushdown Rule

Debe mantenerse:

same semantic result

entre:

source → application projection

y:

storage → pushed-down projection
37. Projection vs View

Un View puede ser:

Database View
API View
Domain Read View
UI View

Projection es el proceso arquitectónico que construye una vista derivada.

38. Materialized Projection

Una projection puede persistirse:

Source Events
      ↓
Projection Processor
      ↓
Materialized Read Model

Esto es especialmente útil en arquitecturas event-driven.

39. Materialized View

Ejemplo:

Order Events
    │
    ▼
Order Projection
    │
    ▼
OrderReadModel

La proyección puede reconstruirse a partir del historial de eventos.

40. Projection Rebuild

Debe ser posible:

Events
 ↓
Replay
 ↓
Projection
 ↓
New Read Model

cuando la arquitectura lo requiera.

41. Projection Versioning

Debe soportar:

CustomerSummaryV1
CustomerSummaryV2

cuando los consumidores tengan ciclos de evolución diferentes.

42. Projection Compatibility

Debe definirse:

backward compatible
forward compatible
breaking

según el contrato.

43. Projection Evolution

Cambiar:

CustomerSummary

de:

id
name

a:

id
name
email

puede ser compatible para consumidores tolerantes a campos adicionales.

Eliminar:

name

puede ser breaking.

44. Projection Lifecycle
DEFINE
 ↓
IMPLEMENT
 ↓
TEST
 ↓
REGISTER
 ↓
DEPLOY
 ↓
OBSERVE
 ↓
VERSION
 ↓
DEPRECATE
 ↓
REMOVE
45. Projection Registry

Puede existir:

ProjectionRegistry

para resolver:

source
target
profile
version
46. Projection Profiles

Ejemplos:

Public
Internal
Admin
Summary
Detail
List
Search
Analytics
Event

Los perfiles deben estar ligados a un propósito claro.

47. No Generic Projection

Evitar:

UniversalProjection

que exponga arbitrariamente cualquier campo.

Una projection debe tener:

purpose
owner
contract
48. Projection Security

Una projection es un security boundary potencial.

Debe impedir:

Data Leakage
Privilege Leakage
Tenant Leakage
Secret Exposure
Internal Metadata Exposure
49. Sensitive Field Protection

Nunca proyectar automáticamente:

password
passwordHash
accessToken
refreshToken
privateKey
secret
securityContext

salvo un contrato específico y autorizado.

50. Authorization vs Projection

Projection decide:

what data is represented

Authorization decide:

whether the caller may access it

Por tanto:

Authorization
      ↓
Projection

no:

Projection = Authorization
51. Field-Level Authorization

Si existen permisos por campo:

user
 ↓
Authorization Policy
 ↓
Allowed Fields
 ↓
Projection

El projector no debe inventar las reglas de autorización.

52. Tenant Isolation

Una projection debe conservar:

tenant_id

cuando forme parte del modelo contractual.

Debe impedir:

cross-tenant projection

accidental.

53. Projection Context

Puede contener:

tenant
locale
timezone
permissions
profile
version
environment

Debe ser explícito.

54. Context-Dependent Projection

Ejemplo:

Customer
 ↓
Projection
 ↓
locale = es

puede producir:

displayStatus = "Activo"

mientras:

locale = en

produce:

displayStatus = "Active"

La lógica de traducción compleja debe permanecer en la capa apropiada.

55. Deterministic Projection

Preferible:

same source
+
same context
+
same version
=
same projection
56. Time-Dependent Projection

Si una projection contiene:

age
relativeTime
expiresIn

debe utilizar un Clock explícito.

57. Projection Purity

Un projector puro:

project(source)

no debería:

save()
publish()
delete()
authorize()
58. Projection Side Effects

Por defecto:

Projection = read-only

Los efectos secundarios deben quedar fuera.

59. Projection Enrichment

Si se requiere:

Customer
+
RiskScore

el enriquecimiento debe estar explícitamente separado:

Query / Application Service
      ↓
Customer + RiskScore
      ↓
Projection

o:

Enrichment
      ↓
Projection

según la arquitectura.

No debe esconderse un network call dentro de:

project()
60. N+1 Protection

Una projection no debe provocar:

100 customers
 ↓
100 database calls

para obtener un campo relacionado.

Preferir:

Batch Query
Join
Preloaded Data
Materialized Read Model

cuando corresponda.

61. Projection Performance

Métricas:

projection_total
projection_latency
projection_errors_total
projection_input_size
projection_output_size
62. Projection Caching

Puede cachearse:

Projected View

cuando:

source stable
context stable
version stable
63. Cache Key

Puede incluir:

entity_id
projection_name
projection_version
tenant_id
context_hash
64. Cache Invalidation

Debe coordinarse con:

E17 — Caching Architecture

No debe existir una estrategia de caching paralela incompatible.

65. Projection Observability

Debe registrar:

projection_name
projection_version
source_type
target_type
duration

sin exponer:

PII
Secrets
Tokens
Sensitive Payloads
66. Projection Tracing

Puede utilizar:

trace_id
projection_id
projection_version

para seguir:

Query
 ↓
Repository
 ↓
Projection
 ↓
API
67. Projection Errors
PROJECTION_ERROR
├── SOURCE_UNAVAILABLE
├── FIELD_NOT_AVAILABLE
├── REQUIRED_FIELD_MISSING
├── UNSUPPORTED_VERSION
├── INVALID_CONTEXT
├── NESTED_PROJECTION_ERROR
└── PROJECTION_LIMIT_EXCEEDED
68. Projection vs Validation

Projection:

select / shape

Validation:

verify correctness

Una projection puede ejecutarse antes o después de una validación dependiendo del boundary.

69. Projection vs Mapping

Puede existir:

Source
 ↓
Projection
 ↓
Mapping
 ↓
DTO

o:

Source
 ↓
Mapping
 ↓
Projection

La elección depende del contrato.

La responsabilidad no debe mezclarse.

70. Projection vs Transformation

Una projection puede ser parte de una transformación:

E24 Transformation
       │
       ├── E25 Mapping
       └── E26 Projection

pero una projection no debe asumir automáticamente todas las capacidades de E24.

71. Projection and Events

En event sourcing:

Event Store
    ↓
Event Replay
    ↓
Projection
    ↓
Read Model

La projection representa el estado derivado.

72. Projection Consistency

Una projection materializada puede ser:

Eventually Consistent

Debe declararse cuando los consumidores dependan de ella.

73. Projection Lag

Debe monitorizarse:

projection_lag

especialmente en:

event-driven
CQRS
materialized views
74. Projection Failure Recovery

Debe soportar:

Retry
Replay
Rebuild
Checkpoint
Dead Letter

según el tipo de projection.

75. Projection Checkpoints

Una projection durable puede almacenar:

last_event_id
last_sequence
last_offset

para recuperación incremental.

76. Idempotency

Aplicar el mismo evento dos veces no debería producir:

duplicate state

cuando la projection sea diseñada para procesamiento idempotente.

77. Ordering

Si el orden de eventos es importante:

event sequence

debe respetarse o gestionarse explícitamente.

78. Projection Reconciliation

Puede compararse:

Source of Truth
       vs
Projected Read Model

para detectar:

drift
missing events
corrupted projections
79. Projection Drift

Debe detectarse cuando:

Expected Projection
       ≠
Actual Projection
80. Projection Testing

Debe cubrir:

Happy Path
Empty Source
Null Fields
Missing Fields
Nested Objects
Collections
Security
Tenant Isolation
Versioning
Compatibility
Rebuild
Idempotency
Ordering
81. Golden Projection Tests

Ejemplo:

source.json
     ↓
projection
     ↓
expected-view.json

Esto permite controlar cambios contractuales.

82. Snapshot Testing

Puede utilizarse:

Source
 ↓
Projection
 ↓
Snapshot

especialmente para:

API responses
UI models
Read models

Debe evitarse que snapshots oculten cambios semánticos importantes.

83. Property Testing

Invariantes:

projection never exposes secret
projection preserves tenant
projection contains required identifier
projection does not mutate source
84. Security Testing

Debe probar:

unauthorized fields
cross-tenant data
sensitive fields
internal metadata
privileged fields
85. Performance Testing

Debe probar:

large collections
deep nesting
high-cardinality queries
materialized rebuilds
projection cache
86. Projection Rebuild Testing

Para projections derivadas de eventos:

Events
 ↓
Projection V1

y:

Events
 ↓
Projection V2

debe verificarse que las diferencias sean intencionales.

87. Schema Compatibility Testing

Debe comprobar:

Producer
 ↓
Projection
 ↓
Consumer

para cada versión contractual.

88. Projection Ownership

Cada projection debe declarar:

owner
source_owner
consumer
purpose
version
89. Projection Consumers

Debe conocerse quién consume:

API
UI
Search
Analytics
Integration
Internal Service

Una projection sin consumidores identificados debería revisarse antes de mantenerse.

90. Projection Governance

Los cambios deben documentar:

Purpose
Source
Target
Fields
Consumers
Security
Version
Compatibility
Performance
Lifecycle
91. Projection Deprecation

Una projection antigua debe indicar:

deprecated_since
replacement
consumer_migration_path
removal_target
92. Projection Registry Governance

Debe evitarse:

CustomerSummary1
CustomerSummary2
CustomerSummaryNew
CustomerSummaryFinal

sin ownership y versionado formal.

Preferible:

CustomerSummary v1
CustomerSummary v2
93. Projection Naming

Los nombres deben expresar propósito:

CustomerSummary
CustomerDetail
CustomerSearch
CustomerPublic
CustomerDashboard

Evitar:

CustomerData
CustomerDTO2
CustomerViewThing

sin significado arquitectónico.

94. Projection Contract Stability

Una projection pública debe tratarse como contrato.

Cambios breaking requieren:

new version
migration
consumer coordination
95. Projection and API Architecture

Flujo recomendado:

HTTP Request
      ↓
Authentication
      ↓
Authorization
      ↓
Application Service
      ↓
Query
      ↓
Projection
      ↓
Response DTO
      ↓
Serialization
96. Projection and Domain Architecture
Domain Model
      ↓
Projection
      ↓
Read Model

El dominio no debe depender del projection model.

97. Projection and Application Architecture

Application Services pueden coordinar:

retrieve
 ↓
authorize
 ↓
enrich
 ↓
project
 ↓
return
98. Projection and Repository Architecture
Repository / Query Repository
          ↓
       Source Data
          ↓
       Projection
          ↓
       Read Model
99. Projection and Messaging
Message
 ↓
Deserialize
 ↓
Query / Processing
 ↓
Projection
 ↓
Message Response
100. Projection and Serialization

La cadena externa puede ser:

Domain / Read Model
       ↓
E26 Projection
       ↓
Response Model
       ↓
E25 Mapping
       ↓
DTO
       ↓
E23 Serialization
       ↓
Wire Format
101. Projection and Transformation

Una composición completa:

Source
  │
  ▼
E26 Projection
  │
  ▼
Selected Model
  │
  ▼
E25 Mapping
  │
  ▼
Target Structure
  │
  ▼
E24 Transformation
  │
  ▼
Target Semantic Model

El orden exacto depende del boundary, pero las responsabilidades permanecen separadas.

102. CQRS Integration

E26 puede proporcionar la arquitectura de:

Command Model
      │
      ▼
Domain Events
      │
      ▼
Projection Processor
      │
      ▼
Read Model
      │
      ▼
Query
103. Event-Sourced Projection
Event 1
Event 2
Event 3
   │
   ▼
Projection Engine
   │
   ▼
Current Read State
104. Projection Processor

Debe soportar:

consume
validate
apply
checkpoint
observe
recover

pero no debería asumir responsabilidades generales de workflow.

105. Projection Handler

Conceptualmente:

handle(event, state)
    ↓
new_state

Debe ser:

deterministic
idempotent
testable

cuando el modelo lo permita.

106. Projection State

Puede almacenarse:

ReadModel
Checkpoint
Version
Metadata

La estructura debe ser independiente del Domain Model cuando optimice consultas.

107. Projection Schema

Debe versionarse:

ProjectionSchemaV1
ProjectionSchemaV2

especialmente cuando exista persistencia del read model.

108. Projection Migration

Una projection persistida puede migrarse:

ReadModel V1
      ↓
Migration
      ↓
ReadModel V2

o reconstruirse:

Event History
      ↓
Projection V2

La segunda estrategia puede ser preferible cuando el coste de replay sea aceptable.

109. Rebuild Strategy

Debe definirse:

full rebuild
incremental rebuild
parallel rebuild
blue/green projection

cuando el sistema lo requiera.

110. Blue/Green Projection

Puede ejecutarse:

Projection V1 → ReadModel V1
Projection V2 → ReadModel V2

y cambiar el consumer después de validar V2.

111. Shadow Projection

Puede ejecutarse V2 sin servir tráfico:

Events
 ├── Projection V1 → Production
 └── Projection V2 → Shadow

y comparar resultados.

112. Projection Backfill

Puede requerirse:

Historical Data
      ↓
Backfill
      ↓
Projection

Debe ser controlado para evitar impactos de carga.

113. Projection Rate Control

Debe soportar:

batch size
throttling
checkpoint interval
retry policy

cuando exista procesamiento masivo.

114. Projection Failure Isolation

Un error en:

Projection A

no debería bloquear necesariamente:

Projection B

si son independientes.

115. Projection Dead Letter

Eventos que no pueden proyectarse pueden dirigirse a:

Dead Letter

para:

inspection
repair
replay
116. Projection Observability Dashboard

Debe poder observarse:

Projection Health
Projection Lag
Processing Rate
Error Rate
Rebuild Progress
Checkpoint
Read Model Freshness
117. Projection Health

Indicadores:

healthy
degraded
stalled
rebuilding
failed
118. Projection Freshness

Debe poder medirse:

source_timestamp
projection_timestamp
lag
119. Projection SLA

Cuando una projection sea parte de una experiencia crítica:

freshness SLA
latency SLA
availability SLA
rebuild SLA

debe estar definido.

120. Projection Cost

Debe considerarse:

storage
CPU
memory
network
rebuild time
query performance

antes de crear una nueva projection materializada.

121. Projection Duplication

No crear una nueva projection si una existente puede satisfacer el caso sin comprometer:

performance
security
contract isolation
122. Projection Specialization

Una nueva projection está justificada cuando:

different consumer
different security boundary
different performance profile
different lifecycle
different schema
123. Projection Anti-Pattern: God View

Evitar:

UniversalCustomerView

que contiene:

API
Admin
Analytics
Security
Billing
Internal

en una sola estructura.

124. Projection Anti-Pattern: Domain Leakage

Evitar exponer directamente:

Domain Entity

como:

Public API Response

cuando el contrato externo tenga necesidades diferentes.

125. Projection Anti-Pattern: Hidden Query

Evitar:

project(customer)

que internamente ejecuta:

database
network

sin que el contrato lo indique.

126. Projection Anti-Pattern: Authorization by Projection

Nunca considerar:

field absent

como sustituto de:

authorization

La autorización debe ocurrir antes de confiar en la projection como boundary de seguridad.

127. Projection Anti-Pattern: Business Rules

Evitar:

project(order)

que determine:

isPremium
shouldApprove
canCancel

cuando esas decisiones pertenezcan a:

Rules
Domain Services
Policy
128. Projection Anti-Pattern: Persistence Coupling

Evitar que:

API Projection

dependa directamente de:

Database schema

si el contrato de API no coincide con el modelo de persistencia.

129. Projection Design Principle

Una projection debe ser diseñada desde el consumidor y el propósito, no desde la estructura accidental del modelo fuente.

130. Projection Boundary Principle

Una projection puede perder información deliberadamente; lo importante es que la pérdida sea conocida, intencional y compatible con el propósito del consumidor.

131. Projection Security Principle

Toda projection hacia un boundary externo debe considerarse una operación de exposición de datos y, por tanto, debe usar una allowlist explícita y respetar autorización y aislamiento de tenant.

132. Projection Performance Principle

Una projection debe seleccionar únicamente los datos necesarios para el caso de uso y evitar cargas, consultas o expansiones que el consumidor no necesita.

133. Projection Evolution Principle

Una projection consumida externamente es un contrato versionado, no un detalle interno de implementación.

134. Definition of Done

E26 queda definido cuando EVOXA dispone de:

✓ Projection Architecture
✓ Projection Boundary
✓ Projection Contract
✓ Projection Characteristics
✓ Projection Types
✓ Entity Projection
✓ Summary Projection
✓ Detail Projection
✓ List Projection
✓ Search Projection
✓ Dashboard Projection
✓ API Projection
✓ Event Projection
✓ Persistence Projection
✓ Analytics Projection
✓ Security Projection
✓ Tenant Projection
✓ Field Selection
✓ Allowlist Strategy
✓ Denylist Defense
✓ Explicit Projection
✓ Nested Projection
✓ Projection Composition
✓ Aggregated Projection
✓ Query Integration
✓ Repository Integration
✓ Database Projection
✓ Projection Pushdown
✓ View Boundary
✓ Materialized Projection
✓ Materialized View
✓ Projection Rebuild
✓ Projection Versioning
✓ Projection Compatibility
✓ Projection Evolution
✓ Projection Lifecycle
✓ Projection Registry
✓ Projection Profiles
✓ No Generic Projection
✓ Security Boundary
✓ Sensitive Field Protection
✓ Authorization Separation
✓ Field-Level Authorization Boundary
✓ Tenant Isolation
✓ Projection Context
✓ Context-Dependent Projection
✓ Deterministic Projection
✓ Time Dependency
✓ Projection Purity
✓ Side-Effect Isolation
✓ Enrichment Boundary
✓ N+1 Protection
✓ Performance
✓ Projection Caching
✓ Cache Integration
✓ Observability
✓ Tracing
✓ Error Taxonomy
✓ Validation Boundary
✓ Mapping Integration
✓ Transformation Integration
✓ CQRS Integration
✓ Event-Sourced Projection
✓ Projection Processor
✓ Projection Handler
✓ Projection State
✓ Projection Schema
✓ Projection Migration
✓ Rebuild Strategies
✓ Blue/Green Projection
✓ Shadow Projection
✓ Backfill
✓ Rate Control
✓ Failure Isolation
✓ Dead Letter
✓ Health Monitoring
✓ Freshness Monitoring
✓ SLA
✓ Cost Management
✓ Duplication Control
✓ Specialization Rules
✓ Anti-Patterns
✓ Design Principles
✓ Security Principles
✓ Performance Principles
✓ Evolution Principles
135. Position in Engineering Specification

La secuencia queda:

E18 — Configuration Architecture
        ↓
E19 — Feature Flag Architecture
        ↓
E20 — Runtime Policy Architecture
        ↓
E21 — Rules Engine Architecture
        ↓
E22 — Validation Architecture
        ↓
E23 — Serialization Architecture
        ↓
E24 — Transformation Architecture
        ↓
E25 — Mapping Architecture
        ↓
E26 — Projection Architecture
        ↓
E27 — Query Architecture

La separación conceptual ahora queda especialmente limpia:

E23 — SERIALIZATION
        │
        │ represent
        ▼
E25 — MAPPING
        │
        │ correspond
        ▼
E24 — TRANSFORMATION
        │
        │ convert
        ▼
E26 — PROJECTION
        │
        │ select / expose
        ▼
E27 — QUERY
        │
        │ retrieve
        ▼
       DATA

Y para la arquitectura de lectura:

                    QUERY
                      │
                      ▼
              Data Retrieval
                      │
                      ▼
                 PROJECTION
                      │
             ┌────────┴────────┐
             ▼                 ▼
          Mapping         Transformation
             │                 │
             └────────┬────────┘
                      ▼
                 Read Model
                      │
                      ▼
                 Serialization
                      │
                      ▼
                External Client

Principio central de E26:
Projection selecciona y construye una vista orientada a un propósito. No es autorización, no es una regla de negocio, no es un serializer y no debe convertirse en un mecanismo oculto de acceso a datos. Una projection debe tener un consumidor, un contrato, un owner, una política de exposición y un ciclo de vida explícitos.

Siguiente: E27 — EVOXA Query Architecture.
