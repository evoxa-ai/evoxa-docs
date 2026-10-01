E24 — EVOXA Transformation Architecture
Engineering Specification

Pertenece a: Engineering Specification
Volumen: volume-09-engineering
Capítulo: E24 — Transformation Architecture
Anterior: E23 — Serialization Architecture
Siguiente: E25 — Mapping Architecture

1. Propósito

E24 define la arquitectura de Transformation de EVOXA.

Transformation establece cómo EVOXA transforma una representación, estructura, modelo, colección o estado en otro modelo o representación semánticamente diferente, preservando las reglas de negocio y los límites de responsabilidad de cada capa.

La distinción fundamental es:

E21 — Rules
    ↓
Decide

E22 — Validation
    ↓
Verify

E23 — Serialization
    ↓
Represent / Reconstruct

E24 — Transformation
    ↓
Convert one model into another
2. Objetivos

E24 debe proporcionar una arquitectura para:

Model Transformation
Data Transformation
Representation Transformation
DTO Transformation
Domain Transformation
Input Transformation
Output Transformation
Projection
Normalization
Denormalization
Aggregation
Flattening
Expansion
Reshaping
Enrichment
Reduction
Filtering
Collection Transformation
Version Transformation
Schema Transformation
Compatibility Transformation
Legacy Transformation
Pipeline Transformation
Transformation Composition
Transformation Testing
Transformation Observability
3. Concepto Fundamental

Transformation responde:

¿Cómo convertimos una estructura o modelo en otro que tiene una forma o propósito diferente?

Ejemplo:

Order
   ↓
OrderSummary

o:

LegacyCustomer
   ↓
CanonicalCustomer

o:

Raw Event
   ↓
Normalized Event

Esto no es simplemente serialization.

4. Serialization vs Transformation
Serialization
Order
 ↓
JSON

La semántica sigue siendo:

Order

Transformation:

Order
 ↓
OrderSummary

La semántica cambia.

5. Transformation Architecture
                    Transformation Control Plane
                              │
             ┌────────────────┼────────────────┐
             ▼                ▼                ▼
          Schemas          Versions          Policies
             │                │                │
             └────────────────┼────────────────┘
                              ▼
                    Transformation Runtime
                              │
        ┌─────────────────────┼─────────────────────┐
        ▼                     ▼                     ▼
     Adapters              Mappers              Pipelines
        │                     │                     │
        └─────────────────────┼─────────────────────┘
                              ▼
                       Transformation
                              │
              ┌───────────────┼───────────────┐
              ▼               ▼               ▼
           Input            Domain           Output
        Transformation   Transformation   Transformation
6. Transformation Boundary

El boundary canónico:

Source Model
      ↓
Transformation
      ↓
Target Model

Debe existir una definición explícita de:

Source
Target
Transformation Contract
Version
Context
Constraints
7. Transformation Contract

Cada transformación debe declarar:

source_type
target_type
version
direction
required_context
supported_variants
failure_model

Ejemplo:

CustomerV1
    ↓
CustomerV2
8. Transformation Direction

Una transformación puede ser:

Forward
Reverse
Bidirectional
One-way

No debe asumirse que:

A → B

implica automáticamente:

B → A
9. Lossless Transformation

Una transformación puede preservar toda la información:

A
 ↓
B
 ↓
A'

donde:

A' ≈ A
10. Lossy Transformation

Puede eliminar información:

Customer
 ├── id
 ├── name
 ├── email
 ├── phone
 └── address
        ↓
CustomerSummary
 ├── id
 └── name

La pérdida debe ser intencional y conocida.

11. Transformation Metadata

Debe poder declararse:

lossless
lossy
deterministic
context_dependent
reversible
non_reversible
12. Transformation vs Mapping

Mapping:

A
 ↓
B

normalmente expresa correspondencia estructural:

first_name → firstName

Transformation puede incluir:

Aggregation
Calculation
Normalization
Filtering
Enrichment

Por tanto:

Mapping ⊂ Transformation

cuando la arquitectura utilice esta relación.

13. Transformation vs Business Logic

Una transformación puede reorganizar datos:

firstName + lastName
→ fullName

pero no debería esconder decisiones de negocio complejas:

customer.balance > 10000
→ premium_customer

si esa clasificación pertenece al dominio o a Rules.

14. Transformation vs Rules

Rules:

Input
 ↓
Decision

Transformation:

Input Model
 ↓
Output Model

Ejemplo:

Rule:
amount > 1000 → requiresApproval

Transformation:
Order → OrderDTO
15. Transformation vs Validation

Transformation:

A → B

Validation:

Is B valid?

La transformación puede producir un resultado que posteriormente debe validarse.

16. Transformation vs Serialization
Domain
 ↓
Transformation
 ↓
DTO
 ↓
Serialization
 ↓
JSON

Esta separación debe mantenerse cuando los boundaries lo requieran.

17. Transformation Categories

EVOXA debe reconocer al menos:

Structural Transformation
Semantic Transformation
Normalization
Projection
Aggregation
Enrichment
Reduction
Expansion
Version Transformation
Schema Transformation
Compatibility Transformation
18. Structural Transformation

Cambia la forma:

{
  firstName,
  lastName
}

a:

{
  name: {
    first,
    last
  }
}

La información puede mantenerse intacta.

19. Semantic Transformation

Cambia la representación semántica:

status = 1

a:

status = "ACTIVE"

cuando ambos representan el mismo concepto bajo contratos diferentes.

20. Normalization

Ejemplo:

"  John   Smith "

→

"John Smith"

Debe distinguirse:

Normalization

de:

Validation

Normalizar cambia el dato.

Validar decide si el dato cumple el contrato.

21. Canonicalization

Puede producir una representación única:

Input Variants
      ↓
Canonical Form

Ejemplo conceptual:

john@example.com
JOHN@EXAMPLE.COM
 John@example.com
       ↓
john@example.com

cuando el contrato permita esa normalización.

22. Projection

Una proyección selecciona información:

Customer
 ↓
CustomerPublicProfile

Debe respetar data minimization.

23. Aggregation

Combina múltiples estructuras:

Customer
+
Orders
+
Payments
       ↓
CustomerDashboard

Este tipo de transformación puede requerir Application Services.

No debe convertir el transformer en un orquestador general.

24. Enrichment

Añade información:

Customer
+
RiskScore
↓
CustomerRiskProfile

Debe distinguirse entre:

Pure Transformation

y:

External Data Enrichment
25. Pure Transformation

Preferida cuando sea posible:

output = transform(input)

sin:

I/O
Database
Network
Side Effects
26. Enrichment Transformation

Cuando requiere datos externos:

Input
 ↓
Data Retrieval
 ↓
Transformation

La recuperación debería pertenecer al servicio apropiado, no quedar oculta dentro de un transformer genérico.

27. Reduction

Reduce información:

OrderItems
 ↓
Total

Debe tener cuidado cuando el cálculo represente una regla de negocio.

28. Expansion

Convierte:

Compact Representation

en:

Expanded Representation

Ejemplo:

customer_id
 ↓
Customer Object

Si requiere lookup externo, ya no es una transformación puramente estructural.

29. Flattening

Ejemplo:

Customer
 └── Address
      ├── city
      └── country

→

customer_city
customer_country
30. Nesting

La operación inversa:

customer_city
customer_country

→

Customer.Address

Debe definir claramente qué ocurre cuando faltan campos.

31. Collection Transformation

Debe soportar:

Map
Filter
Reduce
Sort
Group
Flatten
Expand
Partition

pero evitando esconder reglas de negocio complejas dentro de una colección helper.

32. Transformation Pipeline

Una transformación puede componerse:

Input
 ↓
Normalize
 ↓
Map
 ↓
Enrich
 ↓
Project
 ↓
Validate
 ↓
Output
33. Pipeline Responsibility

Cada etapa debe tener una única responsabilidad.

Evitar:

Normalize
 + Validate
 + Authorize
 + Persist
 + Publish

dentro de un único transformer.

34. Transformation Composition

Las transformaciones pueden componerse:

T1: A → B
T2: B → C
T3: C → D

A
 ↓ T1
B
 ↓ T2
C
 ↓ T3
D
35. Composition Compatibility

Para componer:

T1: A → B
T2: B → C

deben ser compatibles:

Output(T1) ≈ Input(T2)
36. Transformation Context

Puede contener:

tenant
locale
timezone
version
environment
feature_flags
request_context

Debe ser explícito.

37. Context-Dependent Transformation

Ejemplo:

Product
 ↓
localized representation

depende de:

locale

Por tanto:

same input

no necesariamente produce:

same output

sin el mismo contexto.

38. Deterministic Transformation

Una transformación es determinista cuando:

same input
+
same context
+
same version
=
same output

Debe preferirse en pipelines críticos.

39. Non-Deterministic Transformation

Puede depender de:

Current Time
Randomness
External State
External Services

Debe declararse explícitamente.

40. Time Dependency

Cuando una transformación depende del tiempo:

Clock

debe inyectarse.

Evitar:

System.currentTimeMillis()

directamente dentro de lógica de transformación.

41. Version Transformation

Uno de los usos principales:

Model V1
 ↓
Transformation
 ↓
Model V2

Esto permite evolución de contratos.

42. Schema Migration

Debe distinguirse:

Schema Migration

de:

Runtime Transformation

Migration modifica datos persistidos.

Transformation adapta datos durante ejecución.

43. Compatibility Transformation

Puede utilizarse para mantener consumidores antiguos:

Canonical V3
 ↓
Transform
 ↓
Legacy V1
44. Legacy Adapter

Un sistema legado puede conectarse:

EVOXA Canonical Model
        ↓
Legacy Transformation
        ↓
Legacy Representation

El código legacy debe quedar aislado detrás de este boundary.

45. Anti-Corruption Layer

Transformation puede formar parte de una:

Anti-Corruption Layer

para evitar que modelos externos contaminen el dominio EVOXA.

46. External Model Isolation
External Model
      ↓
Adapter / Transformer
      ↓
Canonical EVOXA Model

No:

External Model
      ↓
Domain

cuando existan incompatibilidades semánticas.

47. Canonical Model

EVOXA puede definir modelos canónicos para:

Customer
Order
Identity
Tenant
Payment
Event
Resource

Los sistemas externos se transforman hacia/desde ellos.

48. Transformation Ownership

Cada transformación debe tener un owner:

Domain Team
Application Team
Integration Team
Platform Team

según su boundary.

49. Transformation Registry

Puede existir:

TransformationRegistry

que resuelva:

source_type
target_type
version
context

hacia una transformación.

50. Transformer Contract

Conceptualmente:

transform(source, context)

produce:

target

o:

TransformationError
51. Transformer Metadata

Cada transformer debería declarar:

name
version
source
target
reversible
lossless
deterministic
dependencies
52. Error Model

Debe distinguir:

TransformationError
ValidationError
SerializationError
BusinessRuleError
ExternalDependencyError
53. Transformation Error Taxonomy
TRANSFORMATION_ERROR
 ├── UNSUPPORTED_SOURCE
 ├── UNSUPPORTED_TARGET
 ├── MISSING_REQUIRED_DATA
 ├── INVALID_MAPPING
 ├── INCOMPATIBLE_VERSION
 ├── CONTEXT_REQUIRED
 ├── TRANSFORMATION_LIMIT
 └── EXTERNAL_DEPENDENCY_FAILURE
54. Partial Transformation

Por defecto:

all-or-nothing

cuando una transformación representa una operación contractual.

No producir objetos parcialmente transformados sin un contrato explícito.

55. Optional Transformation

Puede existir:

tryTransform()

para transformaciones opcionales.

Debe distinguir:

No transformation available

de:

Transformation failed
56. Transformation Validation

El resultado de una transformación crítica puede validarse:

Input
 ↓
Transform
 ↓
Validate Output
 ↓
Continue

Especialmente en:

Version Migration
External Integration
Contract Conversion
57. Precondition

Una transformación puede requerir:

source schema
source version
required fields
context

Estas precondiciones deben ser explícitas.

58. Postcondition

Debe definirse qué garantiza el resultado:

Target schema
Target version
Required invariants
59. Transformation Contract Example
Source:
CustomerV1

Target:
CustomerV2

Preconditions:
- source is CustomerV1

Transformation:
- rename full_name → display_name
- normalize contact structure

Postconditions:
- output conforms to CustomerV2

Loss:
- none
60. Business Logic Boundary

Si una transformación necesita decidir:

Should customer receive premium status?

debe delegar:

Rule Engine

o:

Domain Service

según el caso.

El transformer consume la decisión.

61. Transformation and Domain

Un Domain Model puede transformarse hacia:

Domain DTO
Persistence Model
Event Model
API Model

pero el dominio no debe depender de esos modelos.

62. Transformation and Application Services

Application Services pueden orquestar:

Fetch
 ↓
Transform
 ↓
Validate
 ↓
Authorize
 ↓
Execute

El transformer no debe convertirse en Application Service.

63. Transformation and Repository

Repository:

Persistence

Transformer:

Model Conversion

Puede existir:

Persistence Model
 ↓
Domain Model

pero la responsabilidad del acceso a datos sigue perteneciendo al Repository.

64. Transformation and API
HTTP Request
 ↓
Deserialize
 ↓
Validate
 ↓
Transform DTO → Command
 ↓
Application Service

Respuesta:

Domain Result
 ↓
Transform → Response DTO
 ↓
Validate Contract
 ↓
Serialize
65. Transformation and Events
Domain Event
 ↓
Transform
 ↓
Integration Event
 ↓
Serialize
 ↓
Publish

Esto permite separar:

Internal Event Model

de:

External Event Contract
66. Transformation and Messaging

Commands externos pueden transformarse:

External Command
 ↓
Canonical Command

antes de llegar al Application Layer.

67. Transformation and Workflows

Workflow payloads pueden transformarse entre etapas:

Stage A Output
 ↓
Transformation
 ↓
Stage B Input

La transformación debe ser explícita y versionada cuando el workflow sea durable.

68. Transformation and Jobs

Job payload:

V1
 ↓
Transformation
 ↓
Current V2

permite procesar trabajos antiguos después de deployments.

69. Transformation and Scheduling

Schedule definitions antiguas pueden transformarse:

ScheduleV1
 ↓
ScheduleV2

antes de ejecución.

70. Transformation and Configuration

Configuraciones históricas pueden transformarse:

ConfigV1
 ↓
ConfigV2

durante migration o bootstrap.

71. Transformation and Feature Flags

Snapshots de feature flags pueden transformarse cuando cambia su schema.

72. Transformation and Policy

Policy representations pueden transformarse entre versiones:

PolicyV1
 ↓
PolicyV2

pero las decisiones de policy no pertenecen al transformer.

73. Transformation and Rules

Rule definitions pueden migrarse:

RuleSchemaV1
 ↓
RuleSchemaV2

La evaluación continúa perteneciendo a E21.

74. Transformation and Serialization

La cadena completa puede ser:

External Bytes
 ↓
E23 Deserialize
 ↓
External Model
 ↓
E24 Transform
 ↓
Canonical Model
 ↓
E22 Validate
 ↓
Application

Salida:

Canonical Model
 ↓
E24 Transform
 ↓
External Model
 ↓
E22 Validate
 ↓
E23 Serialize
75. Transformation Ordering

Una secuencia común:

Deserialize
 ↓
Transform
 ↓
Validate

o:

Deserialize
 ↓
Validate Source
 ↓
Transform
 ↓
Validate Target

Debe decidirse según el boundary.

76. Normalize Before Validate

Puede existir:

Input
 ↓
Canonicalization
 ↓
Validation

cuando la validación debe aplicarse sobre la forma canónica.

77. Validate Before Transform

Puede ser necesario:

Input
 ↓
Validate
 ↓
Transform

cuando el transformer sólo acepta entradas contractualmente válidas.

78. Transformation Security

Transformation no debe permitir:

Privilege Escalation
Data Leakage
Tenant Crossing
Unsafe Expansion
Resource Exhaustion
79. Tenant Isolation

Transformaciones multi-tenant deben preservar:

tenant_id

y nunca permitir que:

source tenant
→
target different tenant

sin una operación autorizada explícitamente.

80. Data Leakage

Una proyección externa debe excluir:

Internal Metadata
Secrets
Security Attributes
Private Notes
Internal IDs

cuando no formen parte del contrato.

81. Field Allowlist

Para boundaries sensibles:

Allowed Fields

es preferible a:

Serialize Everything
82. Transformation Limits

Debe existir protección contra:

Deep Expansion
Large Collections
Recursive Transformation
Large External Enrichment
83. Recursive Transformation

Si existen estructuras cíclicas:

A
 ↓
B
 ↓
A

debe aplicarse:

Cycle Detection
Depth Limit
Reference Strategy
84. Enrichment Limits

Una transformación no debe generar:

N+1 External Calls

sin control.

Preferible:

Batch Enrichment
Caching
Preloaded Context

cuando sea apropiado.

85. Transformation Performance

Métricas:

transformation_total
transformation_latency
transformation_errors_total
transformation_input_size
transformation_output_size
86. Transformation Tracing

Debe poder observarse:

source_type
target_type
version
duration

sin registrar datos sensibles.

87. Transformation Cache

Puede utilizarse cuando:

Transformation is Pure
Input stable
Context stable
Version stable
88. Cache Key

Debe considerar:

source hash
target version
transformer version
context version

cuando corresponda.

89. Transformation Testing

Cada transformación debe tener:

Valid Input Tests
Boundary Tests
Null Tests
Missing Field Tests
Version Tests
Compatibility Tests
Loss Tests
Round Trip Tests
Security Tests
Performance Tests

según su naturaleza.

90. Golden Fixtures

Debe haber fixtures:

fixtures/
 ├── source-v1
 ├── target-v1
 ├── source-v2
 └── target-v2

para detectar cambios semánticos accidentales.

91. Property Testing

Puede comprobarse:

transform(x)

contra invariantes:

required fields preserved
identifier preserved
tenant preserved
92. Loss Testing

Para transformaciones declaradas lossless:

A
 ↓
B
 ↓
A'

debe comprobarse:

A ≈ A'
93. Compatibility Testing

Debe probarse:

V1 → V2
V2 → V1

cuando ambas direcciones estén soportadas.

94. Migration Testing

Antes de migrar datos:

Sample Dataset
 ↓
Transformation
 ↓
Validation
 ↓
Comparison
95. Shadow Transformation

Puede ejecutarse una nueva transformación en paralelo:

Current Transformer
       +
New Transformer
       ↓
Compare Results

sin modificar el resultado productivo.

96. Transformation Versioning

Cada cambio semántico significativo debe producir:

Transformer v1
Transformer v2

o una estrategia equivalente de versionado.

97. Semantic Versioning

Cuando sea apropiado:

MAJOR
MINOR
PATCH

puede comunicar:

Breaking
Compatible Feature
Bug Fix
98. Transformation Lifecycle
DEFINE
 ↓
IMPLEMENT
 ↓
TEST
 ↓
REGISTER
 ↓
PUBLISH
 ↓
OBSERVE
 ↓
DEPRECATE
 ↓
MIGRATE
 ↓
REMOVE
99. Deprecation

Una transformación antigua debe indicar:

deprecated_since
replacement
migration_path
removal_target

cuando sea necesario.

100. Transformation Registry Governance

No debe permitirse registrar transformaciones ambiguas:

A → B
A → B

sin resolver:

version
priority
context
101. Single Responsibility

Un transformer debe tener una responsabilidad semántica clara:

Order → OrderSummary

no:

Order
→ Summary
→ Authorization
→ Persistence
→ Notification
102. No Hidden Side Effects

Preferiblemente:

transform()

no debe:

save()
publish()
delete()
authorize()
103. Pure Transformation Contract

Cuando sea posible:

output = f(input, context)

sin modificar:

input

ni producir side effects.

104. Immutability

Preferible:

Input Object
      ↓
New Output Object

en lugar de:

Input Object
      ↓
Mutate In Place

especialmente en pipelines concurrentes.

105. Input Preservation

Los transformers no deberían modificar silenciosamente el objeto fuente salvo que el contrato lo establezca.

106. Output Ownership

El resultado de una transformación debe tener ownership claro.

Ejemplo:

Domain Object
 ↓
API DTO

el DTO pertenece al boundary API.

107. Contract Stability

Una transformación debe preservar los campos contractualmente definidos.

Cambiar:

customer_id

por:

id

es un cambio de contrato si el target lo considera breaking.

108. Transformation Drift

Debe detectarse cuando:

Expected Mapping
        ≠
Actual Mapping

por cambios de código.

109. Schema Drift

Debe coordinarse con E23:

Schema Change
 ↓
Transformation Impact Analysis
 ↓
Serializer Impact
110. Transformation Governance

Los cambios críticos deben documentar:

Source
Target
Reason
Version
Compatibility
Data Loss
Security Impact
Performance Impact
Migration
111. Architectural Flow

La integración canónica queda:

                 External Boundary
                        │
                        ▼
                 E23 Serialization
                        │
                        ▼
                 External Model
                        │
                        ▼
              E24 Transformation
                        │
                        ▼
                Canonical Model
                        │
                        ▼
                 E22 Validation
                        │
                        ▼
               Application Layer
                        │
                        ▼
                 Domain Model

Salida:

Domain Model
      │
      ▼
E24 Transformation
      │
      ▼
External Model
      │
      ▼
E22 Validation
      │
      ▼
E23 Serialization
      │
      ▼
External Boundary
112. Definition of Done

E24 queda definido cuando EVOXA dispone de:

✓ Transformation Architecture
✓ Transformation Boundaries
✓ Transformation Contracts
✓ Transformation Direction
✓ Lossless Transformation
✓ Lossy Transformation
✓ Transformation Metadata
✓ Structural Transformation
✓ Semantic Transformation
✓ Normalization
✓ Canonicalization
✓ Projection
✓ Aggregation
✓ Enrichment
✓ Reduction
✓ Expansion
✓ Flattening
✓ Nesting
✓ Collection Transformation
✓ Transformation Pipelines
✓ Transformation Composition
✓ Transformation Context
✓ Deterministic Transformation
✓ Context-Dependent Transformation
✓ Time Dependency
✓ Version Transformation
✓ Schema Migration Boundary
✓ Compatibility Transformation
✓ Legacy Transformation
✓ Anti-Corruption Layer
✓ Canonical Model Integration
✓ Transformation Ownership
✓ Transformation Registry
✓ Transformer Contract
✓ Transformer Metadata
✓ Error Model
✓ Error Taxonomy
✓ Partial Transformation Policy
✓ Preconditions
✓ Postconditions
✓ Business Logic Boundary
✓ Domain Integration
✓ Application Integration
✓ Repository Boundary
✓ API Integration
✓ Event Integration
✓ Messaging Integration
✓ Workflow Integration
✓ Job Integration
✓ Scheduling Integration
✓ Configuration Integration
✓ Feature Flag Integration
✓ Policy Integration
✓ Rules Integration
✓ Serialization Integration
✓ Validation Integration
✓ Security
✓ Tenant Isolation
✓ Data Leakage Prevention
✓ Field Allowlists
✓ Transformation Limits
✓ Cycle Detection
✓ Enrichment Limits
✓ Performance
✓ Observability
✓ Tracing
✓ Transformation Cache
✓ Cache Key Strategy
✓ Testing
✓ Golden Fixtures
✓ Property Testing
✓ Loss Testing
✓ Compatibility Testing
✓ Migration Testing
✓ Shadow Transformation
✓ Versioning
✓ Deprecation
✓ Lifecycle
✓ Registry Governance
✓ Single Responsibility
✓ No Hidden Side Effects
✓ Purity
✓ Immutability
✓ Input Preservation
✓ Output Ownership
✓ Contract Stability
✓ Transformation Drift Detection
✓ Schema Drift Coordination
✓ Governance
113. Position in Engineering Specification

La secuencia ahora queda:

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

La distinción crítica entre E21–E24:

E21 — RULES
       Decide
         ↓
E22 — VALIDATION
       Verify
         ↓
E23 — SERIALIZATION
       Represent
         ↓
E24 — TRANSFORMATION
       Convert

Y el pipeline arquitectónico recomendado:

External Representation
        │
        ▼
E23 — Deserialize
        │
        ▼
External Model
        │
        ▼
E24 — Transform
        │
        ▼
Canonical / Domain-facing Model
        │
        ▼
E22 — Validate
        │
        ▼
Application / Domain
        │
        ▼
E21 — Rules
        │
        ▼
Decision / Execution

Principio central de E24:
Transformation convierte modelos y representaciones entre boundaries sin absorber validación, reglas de negocio, autorización, persistencia, serialización ni orquestación. Una transformación debe tener un origen, un destino, un contrato y una semántica de evolución explícitos.

Siguiente capítulo: E25 — EVOXA Mapping Architecture.
