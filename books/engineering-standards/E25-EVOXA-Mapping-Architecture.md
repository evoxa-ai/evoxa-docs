E25 — EVOXA Mapping Architecture
Engineering Specification

Pertenece a: Engineering Specification
Volumen: volume-09-engineering
Capítulo: E25 — Mapping Architecture
Anterior: E24 — Transformation Architecture
Siguiente: E26 — Projection Architecture

1. Propósito

E25 define la arquitectura de Mapping de EVOXA.

Mapping establece cómo se define y ejecuta la correspondencia entre campos, propiedades, estructuras y tipos de dos modelos compatibles, sin asumir por sí mismo reglas de negocio, persistencia, transporte o serialización.

La distinción central es:

E23 — Serialization
    ↓
Represent

E24 — Transformation
    ↓
Convert

E25 — Mapping
    ↓
Correspond

Un mapping responde:

¿Qué elemento del modelo origen corresponde con qué elemento del modelo destino y cómo se obtiene su valor?

2. Objetivos

E25 debe proporcionar:

Field Mapping
Property Mapping
Type Mapping
Nested Mapping
Collection Mapping
Map Mapping
Enum Mapping
Identifier Mapping
Date/Time Mapping
Money Mapping
Nullable Mapping
Optional Mapping
Default Mapping
Rename Mapping
Flattening Mapping
Nesting Mapping
Conditional Mapping
Computed Mapping
Bidirectional Mapping
One-Way Mapping
Mapping Profiles
Mapping Registry
Mapping Resolution
Mapping Composition
Mapping Versioning
Mapping Validation
Mapping Testing
Mapping Observability
Mapping Security
3. Concepto Fundamental

Mapping:

Source Model
     │
     │ correspondence
     ▼
Target Model

Ejemplo:

Customer
├── firstName
├── lastName
└── email

        ↓ Mapping

CustomerDTO
├── first_name
├── last_name
└── email

El mapping expresa:

firstName → first_name
lastName  → last_name
email     → email
4. Mapping vs Transformation

Mapping:

firstName → first_name

Transformation:

firstName + lastName
        ↓
displayName

Por tanto:

Mapping
= Correspondence

Transformation
= Conversion

Una transformación puede utilizar mappings internamente, pero un mapping no debe convertirse silenciosamente en una transformación arbitraria.

5. Mapping Architecture
                         Mapping Control Plane
                                  │
             ┌────────────────────┼────────────────────┐
             ▼                    ▼                    ▼
         Profiles             Schemas              Versions
             │                    │                    │
             └────────────────────┼────────────────────┘
                                  ▼
                         Mapping Runtime
                                  │
              ┌───────────────────┼───────────────────┐
              ▼                   ▼                   ▼
          Resolver             Mapper             Converter
              │                   │                   │
              └───────────────────┼───────────────────┘
                                  ▼
                         Mapping Execution
                                  │
                    ┌─────────────┼─────────────┐
                    ▼             ▼             ▼
                 Source        Mapping        Target
6. Mapping Boundary

El boundary debe ser explícito:

Source Type
     ↓
Mapping Definition
     ↓
Target Type

El mapping debe conocer:

source_type
target_type
field mappings
nested mappings
conversion rules
version
context
7. Mapping Contract

Conceptualmente:

map(source, context)

produce:

target

o:

MappingError
8. Mapping Definition

Una definición puede contener:

Mapping
├── source
├── target
├── fields
├── nested
├── conversions
├── defaults
├── conditions
└── metadata

Ejemplo conceptual:

Customer.firstName
        ↓
CustomerDTO.first_name
9. Field Mapping

Es la unidad básica:

source.field
      ↓
target.field

Ejemplo:

userId → id
10. Identity Mapping

Cuando los nombres y tipos coinciden:

id → id
email → email
status → status

puede existir mapping implícito.

Sin embargo, para contratos críticos debe poder hacerse explícito.

11. Explicit Mapping

Preferible cuando existe riesgo de ambigüedad:

source:
customer.identifier

target:
customer.id

La correspondencia debe quedar documentada.

12. Rename Mapping
firstName
    ↓
first_name

El valor permanece semánticamente equivalente.

13. Type Mapping

Puede existir:

source: Integer
target: String

pero el cambio de tipo debe tener una conversión definida.

Mapping identifica la correspondencia.

El converter ejecuta la conversión.

14. Mapping + Conversion
Source Field
     ↓
Mapping
     ↓
Converter
     ↓
Target Field

Ejemplo:

amount: Decimal
       ↓
Money Converter
       ↓
amount: String
15. Mapping vs Converter

Mapping:

amount → amount

Converter:

Decimal → String

Separar ambos conceptos evita que un mapper acumule lógica de conversión arbitraria.

16. Nested Mapping

Ejemplo:

Customer
 └── Address
      ├── city
      └── country

a:

CustomerDTO
 └── AddressDTO
      ├── city
      └── country

El mapping debe poder delegar:

Address → AddressDTO
17. Nested Mapping Registry

Puede existir:

Address → AddressDTO
Customer → CustomerDTO
Order → OrderDTO

El mapper principal reutiliza mappings registrados.

18. Collection Mapping

Ejemplo:

List<Order>
      ↓
List<OrderDTO>

El mapping de colección puede componerse:

Order → OrderDTO

aplicado sobre cada elemento.

19. Collection Semantics

Debe definirse:

null collection
empty collection
ordered collection
unordered collection

No deben tratarse automáticamente como equivalentes.

20. Map Mapping

Para:

Map<K,V>

debe definirse:

K_source → K_target
V_source → V_target

Ejemplo:

Map<CustomerId, Order>

→

Map<String, OrderDTO>
21. Enum Mapping

Ejemplo:

DomainStatus.PENDING
        ↓
ApiStatus.PENDING

Puede requerir:

PENDING → pending
APPROVED → approved
REJECTED → rejected
22. Unknown Enum Values

Debe definirse una estrategia:

REJECT
DEFAULT
UNKNOWN
IGNORE

No debe existir comportamiento implícito para enums contractuales.

23. Identifier Mapping

Puede existir:

DomainId
   ↓
String

o:

UUID
   ↓
ExternalId

El mapping debe preservar la identidad semántica.

24. Identifier Transformation

Si:

InternalId
 ↓
ExternalId

requiere generación, hashing o lookup, ya no es un mapping puramente estructural.

Debe delegarse a:

E24 Transformation

o al servicio correspondiente.

25. Date Mapping

Puede mapear:

createdAt → created_at

sin cambiar semántica.

Si además convierte:

UTC timestamp
 ↓
Local date

ya existe una transformación semántica.

26. Timezone Mapping

Debe especificarse:

source timezone
target timezone

cuando el mapping altere la representación temporal.

27. Money Mapping

Ejemplo:

Money
├── amount
└── currency

        ↓

MoneyDTO
├── amount
└── currency

La correspondencia debe preservar:

amount
currency
precision
28. Decimal Mapping

Debe evitarse una conversión implícita:

Decimal → Float

si puede producir pérdida de precisión.

29. Nullable Mapping

Debe distinguir:

null

de:

absent

cuando el target tenga semánticas diferentes.

30. Optional Mapping

Debe definirse:

Optional<T>

→

T?

o:

field absent

según el contrato.

31. Default Mapping

Puede existir:

source.field missing
        ↓
target.field = default

Pero los defaults contractuales deben estar definidos por el schema o contrato apropiado.

El mapper no debe inventar semántica.

32. Null-to-Default

Una política puede definir:

null → default

pero debe ser explícita.

No debe aplicarse globalmente.

33. Missing-to-Default

Diferente de:

null → default

porque:

missing

puede significar:

not provided

mientras:

null

significa:

explicitly empty
34. Flattening Mapping
Customer.Address.city
        ↓
CustomerDTO.address_city

El mapping debe expresar el path explícitamente.

35. Nesting Mapping
address_city
address_country

→

Address
├── city
└── country

Debe existir una definición clara de creación del objeto nested.

36. Conditional Mapping

Puede existir:

if source.type == "business"
    businessName → name

Pero la condición debe permanecer simple.

Si representa una decisión de negocio compleja:

Rule Engine

es el boundary correcto.

37. Computed Mapping

Ejemplo:

firstName + lastName
        ↓
displayName

Esto puede ser considerado mapping computado, pero arquitectónicamente debe tratarse como una transformación cuando la operación tenga semántica significativa.

38. Mapping Purity

Preferido:

target = map(source)

sin:

database access
network access
publishing
persistence
39. No Hidden I/O

Un mapper no debería ejecutar:

repository.find()
api.call()
event.publish()

para resolver campos.

Eso rompe:

determinism
performance
testability
40. Enrichment Boundary

Si falta un dato:

source.customerId
        ↓
lookup customer
        ↓
target.customer

esto debe pertenecer a:

Application Service

o:

E24 Transformation / Enrichment

según el caso.

El mapper puro no debería ocultar ese lookup.

41. Bidirectional Mapping

Puede existir:

A → B
B → A

pero no debe asumirse que ambos mappings son automáticamente inversos.

42. Reverse Mapping

Para:

Domain → DTO

puede existir:

DTO → Domain

pero cada dirección debe tener contrato propio.

43. Round Trip Mapping

Cuando sea posible:

A
 ↓
map A→B
 ↓
B
 ↓
map B→A
 ↓
A'

debe cumplirse:

A ≈ A'

si el mapping se declara reversible.

44. Lossy Mapping

Puede ocurrir:

DomainEntity
├── id
├── name
├── email
├── metadata
└── securityContext

        ↓

PublicDTO
├── id
└── name

Esto es deliberadamente lossless=false.

45. Mapping Metadata

Debe poder declararse:

lossless
reversible
deterministic
pure
version
owner
46. Mapping Profiles

EVOXA puede definir profiles:

DomainToApi
DomainToPersistence
DomainToEvent
ApiToCommand
EventToDomain
LegacyToCanonical
CanonicalToLegacy

Cada profile representa un boundary específico.

47. Profile Isolation

No debe reutilizarse automáticamente:

DomainToPersistence

para:

DomainToApi

aunque los campos parezcan similares.

Los boundaries tienen objetivos distintos.

48. API Mapping
Request DTO
     ↓
Command

y:

Domain Result
     ↓
Response DTO
49. Persistence Mapping
Persistence Model
     ↓
Domain Model

y:

Domain Model
     ↓
Persistence Model

El Repository sigue siendo responsable del acceso a almacenamiento.

50. Event Mapping
Domain Event
     ↓
Integration Event

Permite proteger el contrato interno del dominio.

51. Message Mapping
External Message
      ↓
Internal Command

El mapping debe estar asociado al contrato del mensaje.

52. Legacy Mapping
Legacy Model
      ↓
Canonical EVOXA Model

Este boundary puede formar parte de un Anti-Corruption Layer.

53. Mapping Registry

Debe poder resolverse:

source_type
target_type
profile
version

→

Mapper
54. Registry Example

Conceptualmente:

MappingRegistry
├── Customer → CustomerDTO
├── Order → OrderDTO
├── Order → OrderEvent
├── CustomerV1 → CustomerV2
└── LegacyCustomer → Customer
55. Ambiguous Mapping

No debe existir:

A → B

con múltiples mappings igualmente válidos sin:

profile
version
context
priority
56. Mapping Resolution

Resolución:

Source
 +
Target
 +
Profile
 +
Version
 +
Context

debe producir un mapper único.

57. Mapping Versioning

Un cambio contractual puede producir:

Customer → CustomerDTO v1
Customer → CustomerDTO v2

Los consumers deben poder seleccionar explícitamente la versión.

58. Compatibility

Un mapping debe declarar si soporta:

source v1 → target v1
source v1 → target v2
source v2 → target v1
source v2 → target v2
59. Mapping Evolution

El ciclo:

Define
 ↓
Implement
 ↓
Test
 ↓
Register
 ↓
Deploy
 ↓
Observe
 ↓
Deprecate
 ↓
Remove

debe estar gobernado.

60. Mapping Validation

Antes de registrar un mapping se debe comprobar:

source exists
target exists
fields resolvable
types compatible
converters available
nested mappings available
61. Static Mapping Validation

Cuando sea posible:

compile-time validation

es preferible a descubrir errores durante runtime.

62. Runtime Mapping Validation

Debe existir cuando:

dynamic schemas
plugins
runtime contracts
external integrations

hagan imposible conocer todo durante compilación.

63. Mapping Errors
MAPPING_ERROR
├── SOURCE_FIELD_NOT_FOUND
├── TARGET_FIELD_NOT_FOUND
├── TYPE_MISMATCH
├── CONVERTER_NOT_FOUND
├── NESTED_MAPPING_NOT_FOUND
├── AMBIGUOUS_MAPPING
├── UNSUPPORTED_VERSION
└── REQUIRED_VALUE_MISSING
64. Mapping vs Validation Errors
Mapping Error
→ no sabemos cómo establecer la correspondencia

Validation Error
→ sabemos qué corresponde, pero el valor no cumple el contrato
65. Mapping vs Transformation Errors
Mapping Error
→ correspondence failure

Transformation Error
→ conversion failure
66. Mapping Security

Mapping debe proteger contra:

Data Leakage
Privilege Field Exposure
Tenant Leakage
Sensitive Field Exposure
Unexpected Field Propagation
67. Explicit Field Exposure

Para boundaries externos:

source fields
      ↓
allowlisted target fields

es preferible a:

auto-map everything
68. Sensitive Fields

Nunca deben mapearse automáticamente:

password
secret
token
private_key
security_metadata

salvo contrato explícito y autorizado.

69. Tenant Fields

Debe verificarse:

source.tenantId

→

target.tenantId

y evitar que un mapper permita cambiarlo arbitrariamente.

70. Authorization Fields

Campos como:

roles
permissions
securityContext

no deben copiarse automáticamente entre boundaries.

71. Mapping Performance

Métricas:

mapping_total
mapping_errors_total
mapping_latency
mapping_input_size
mapping_output_size
72. Hot Path

En hot paths:

Reflection
Dynamic Lookup
Repeated Registry Resolution

debe minimizarse.

Puede utilizarse:

Generated Mapper
Compiled Mapping
Cached Mapping Plan
73. Mapping Plan

Un mapping puede compilarse en:

Mapping Plan
├── source accessor
├── converter
├── target setter
└── nested plan

Esto reduce overhead.

74. Generated Mapping

Cuando sea apropiado:

Schema
 ↓
Code Generation
 ↓
Mapper

puede proporcionar:

Performance
Type Safety
Compile-Time Errors
75. Reflection Mapping

Puede utilizarse para:

Dynamic Models
Plugins
Generic Infrastructure

pero debe controlarse por:

Performance
Security
Type Safety
76. Mapping Cache

Puede cachearse:

Resolved Mapping Plan

si:

source type
target type
profile
version

son estables.

77. Mapping Observability

Una trace puede registrar:

source_type
target_type
profile
version
duration

No debe registrar automáticamente todo el objeto.

78. Mapping Drift

Debe detectarse:

Mapping Definition
        ≠
Actual Output

especialmente cuando existen mappings generados.

79. Mapping Testing

Cada mapping importante debe cubrir:

Happy Path
Nulls
Missing Fields
Nested Objects
Collections
Type Conversion
Defaults
Unknown Fields
Versioning
Security
80. Golden Mapping Tests

Ejemplo conceptual:

source.json
   ↓
mapper
   ↓
expected-target.json

Esto permite detectar cambios accidentales.

81. Property Tests

Invariantes:

identifier preserved
tenant preserved
required fields populated
sensitive fields excluded

pueden comprobarse automáticamente.

82. Security Tests

Debe comprobarse que:

password
token
private metadata

no aparezcan en mappings públicos.

83. Compatibility Tests

Debe probarse:

Source V1 → Target V1
Source V2 → Target V1
Source V1 → Target V2
Source V2 → Target V2

cuando aplique.

84. Performance Tests

Debe medirse:

Throughput
Latency
Memory
Allocation
Nested Mapping Cost
Collection Mapping Cost
85. Mapping Lifecycle
PROPOSE
 ↓
DESIGN
 ↓
IMPLEMENT
 ↓
VALIDATE
 ↓
TEST
 ↓
REGISTER
 ↓
DEPLOY
 ↓
OBSERVE
 ↓
DEPRECATE
 ↓
REMOVE
86. Mapping Ownership

Cada mapping debe tener:

owner
source_owner
target_owner
contract_owner

cuando diferentes equipos sean responsables de los modelos.

87. Cross-Team Mapping

Para:

Team A Model
     ↓
Team B Contract

debe existir ownership explícito del boundary.

88. Contract Change Impact

Cuando cambia:

Source Model

debe evaluarse:

Mapping Impact
Transformation Impact
Serialization Impact
Consumer Impact
89. Mapping Dependency Graph

Puede representarse:

Customer
   ↓
CustomerDTO
   ↓
API Response

y:

Customer
   ↓
CustomerEvent
   ↓
Integration Event

Esto permite análisis de impacto.

90. Circular Mapping Dependencies

Debe evitarse:

A → B
B → C
C → A

cuando genere dependencia arquitectónica circular.

91. Mapping Composition

Puede componerse:

A → B
B → C

pero debe evaluarse si es mejor:

A → C

para evitar overhead y pérdida acumulada.

92. Mapping Chaining

Cadena:

External DTO
 ↓
Canonical DTO
 ↓
Domain

puede ser válida.

Pero cada paso debe tener una responsabilidad clara.

93. Direct Mapping

Cuando los modelos son compatibles:

A → C

puede ser preferible a:

A → B → C

si no existe una razón arquitectónica para mantener el modelo intermedio.

94. Mapping Intermediate Models

Los modelos intermedios pueden ser útiles para:

Canonicalization
Legacy Isolation
Protocol Independence
Version Migration
95. No Universal Mapper

EVOXA no debe intentar crear:

UniversalMapper

capaz de transformar cualquier objeto arbitrariamente.

Esto reduce:

Type Safety
Predictability
Security
Maintainability
96. Explicit Mapping over Magic

Preferible:

CustomerMapper

sobre:

magic auto-mapper

cuando el boundary sea contractual o sensible.

97. Convention over Configuration

Puede utilizarse auto-mapping para casos triviales:

id → id
name → name

pero con escape explícito hacia mappings manuales.

98. Manual Override

Si auto-mapping produce:

incorrect correspondence

debe poder definirse:

Explicit Override
99. Mapping Profiles and Policies

Un profile puede imponer:

No Sensitive Fields
No Unknown Fields
Explicit IDs
Explicit Enums

Esto conecta mapping con policy sin convertirlo en policy engine.

100. Architectural Flow

La cadena completa queda:

                  External Representation
                           │
                           ▼
                    E23 Serialization
                           │
                           ▼
                     Source Model
                           │
                           ▼
                    E25 Mapping
                           │
                  ┌────────┴────────┐
                  ▼                 ▼
              Converter         Nested Mapper
                  │                 │
                  └────────┬────────┘
                           ▼
                      Target Model
                           │
                           ▼
                  E24 Transformation
                           │
                           ▼
                  Canonical Model

En la práctica, E25 puede ser utilizado dentro de E24, cuando la transformación necesita correspondencia estructural:

E24 Transformation
       │
       ├── E25 Mapping
       ├── Conversion
       ├── Normalization
       └── Enrichment
101. Position Relative to E23 and E24

La separación conceptual definitiva:

E23 — SERIALIZATION
"What is the wire representation?"
             │
             ▼
E25 — MAPPING
"Which source field corresponds to which target field?"
             │
             ▼
E24 — TRANSFORMATION
"How does the source model become a different target model?"

Aunque el orden conceptual pueda variar en una implementación concreta, la responsabilidad arquitectónica permanece separada.

102. Definition of Done

E25 queda definido cuando EVOXA dispone de:

✓ Mapping Architecture
✓ Mapping Boundary
✓ Mapping Contract
✓ Mapping Definitions
✓ Field Mapping
✓ Identity Mapping
✓ Explicit Mapping
✓ Rename Mapping
✓ Type Mapping
✓ Conversion Boundary
✓ Nested Mapping
✓ Nested Mapping Registry
✓ Collection Mapping
✓ Collection Semantics
✓ Map Mapping
✓ Enum Mapping
✓ Unknown Enum Policy
✓ Identifier Mapping
✓ Date Mapping
✓ Timezone Mapping
✓ Money Mapping
✓ Decimal Mapping
✓ Nullable Mapping
✓ Optional Mapping
✓ Default Mapping
✓ Null-to-Default Policy
✓ Missing-to-Default Policy
✓ Flattening Mapping
✓ Nesting Mapping
✓ Conditional Mapping
✓ Computed Mapping Boundary
✓ Mapping Purity
✓ No Hidden I/O
✓ Enrichment Boundary
✓ Bidirectional Mapping
✓ Reverse Mapping
✓ Round-Trip Mapping
✓ Lossy Mapping
✓ Mapping Metadata
✓ Mapping Profiles
✓ Profile Isolation
✓ API Mapping
✓ Persistence Mapping
✓ Event Mapping
✓ Message Mapping
✓ Legacy Mapping
✓ Mapping Registry
✓ Mapping Resolution
✓ Ambiguity Handling
✓ Mapping Versioning
✓ Compatibility
✓ Mapping Evolution
✓ Mapping Validation
✓ Static Validation
✓ Runtime Validation
✓ Mapping Error Taxonomy
✓ Security
✓ Field Exposure Controls
✓ Sensitive Field Protection
✓ Tenant Isolation
✓ Authorization Field Protection
✓ Performance
✓ Mapping Plans
✓ Generated Mapping
✓ Reflection Boundary
✓ Mapping Cache
✓ Observability
✓ Mapping Drift Detection
✓ Golden Tests
✓ Property Tests
✓ Security Tests
✓ Compatibility Tests
✓ Performance Tests
✓ Mapping Lifecycle
✓ Mapping Ownership
✓ Cross-Team Ownership
✓ Impact Analysis
✓ Dependency Graph
✓ Circular Dependency Protection
✓ Mapping Composition
✓ Mapping Chaining
✓ Direct Mapping
✓ Intermediate Models
✓ Explicit Mapping Strategy
✓ Convention-Based Mapping
✓ Manual Overrides
✓ Mapping Profiles and Policies
103. Position in Engineering Specification

La secuencia queda ahora:

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

La relación fundamental entre los tres últimos capítulos:

E23
Serialization
    │
    │ represent
    ▼
E25
Mapping
    │
    │ correspond
    ▼
E24
Transformation
    │
    │ convert
    ▼
E26
Projection
    │
    │ select / expose
    ▼
Target Boundary

Y una regla arquitectónica importante para EVOXA:

E25 debe ser deliberadamente más pequeño que E24. Mapping establece correspondencias; Transformation realiza conversiones semánticas. Cuando un mapper empieza a tomar decisiones, consultar sistemas externos, ejecutar reglas o producir modelos con significado nuevo, la responsabilidad debe migrar hacia E24, Application Services, Domain Services o Rules, según corresponda.

Siguiente: E26 — EVOXA Projection Architecture.
