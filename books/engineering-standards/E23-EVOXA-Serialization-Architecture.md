E23 — EVOXA Serialization Architecture
Engineering Specification

Pertenece a: Engineering Specification
Volumen: volume-09-engineering
Capítulo: E23 — Serialization Architecture
Anterior: E22 — Validation Architecture
Siguiente: E24 — Transformation Architecture

1. Propósito

E23 define la arquitectura de Serialization de EVOXA: cómo los objetos, valores y contratos internos se transforman en representaciones transportables y cómo esas representaciones se reconstruyen de forma segura y determinista.

La distinción fundamental es:

E22 — Validation
    ↓
¿La representación es válida?

E23 — Serialization
    ↓
¿Cómo representamos el objeto/valor?

E24 — Transformation
    ↓
¿Cómo transformamos una representación/modelo en otro?

Serialization no debe convertirse en:

Domain Logic;
Validation;
Mapping arbitrario;
Business Transformation;
Persistence Logic;
API orchestration.
2. Objetivos

E23 debe proporcionar:

Object Serialization
Object Deserialization
Schema-Aware Serialization
Format Management
Encoding
Decoding
Canonical Representation
Versioning
Compatibility
Type Mapping
Null Handling
Optional Fields
Collections
Nested Objects
Date/Time Serialization
Money Serialization
Identifier Serialization
Enum Serialization
Binary Serialization
Compression Boundary
Encryption Boundary
Streaming Serialization
Message Serialization
Event Serialization
API Serialization
Persistence Serialization
Error Handling
Security
Testing
Observability
3. Concepto Fundamental

Serialization:

Object
  ↓
Serializer
  ↓
Representation

Deserialization:

Representation
  ↓
Deserializer
  ↓
Object

Ejemplo:

Order
  ↓
JSON Serializer
  ↓
{
  "id": "...",
  "status": "pending",
  "total": 150
}
4. Serialization Architecture
                         Serialization Control Plane
                                    │
                 ┌──────────────────┼──────────────────┐
                 ▼                  ▼                  ▼
              Schemas            Formats           Versions
                 │                  │                  │
                 └──────────────────┼──────────────────┘
                                    ▼
                           Serialization Runtime
                                    │
              ┌─────────────────────┼─────────────────────┐
              ▼                     ▼                     ▼
         Serializer            Deserializer          Codec Registry
              │                     │                     │
              └─────────────────────┼─────────────────────┘
                                    ▼
                             Representation
                                    │
             ┌──────────────────────┼──────────────────────┐
             ▼                      ▼                      ▼
            API                   Events                Messages
5. Serialization Boundary

El boundary debe ser explícito:

Domain Model
      │
      ▼
Serialization Boundary
      │
      ▼
External Representation

El modelo de dominio no debe depender de detalles específicos de:

JSON
XML
MessagePack
Protobuf
Database Wire Format
6. Domain Independence

Preferible:

Domain Object
      ↓
DTO / Contract
      ↓
Serializer

y no:

Domain Object
      ↓
JSON annotations everywhere

cuando esto genere acoplamiento innecesario.

7. Serialization Formats

EVOXA puede soportar diferentes formatos:

JSON
Protobuf
MessagePack
Avro
CSV
Binary
Custom Formats

La arquitectura debe separar:

Serialization Semantics

de:

Encoding Format
8. Format Registry

Debe existir un mecanismo para registrar:

format
media_type
serializer
deserializer
version
capabilities

Ejemplo:

application/json
application/protobuf
application/msgpack
9. Serializer Contract

Conceptualmente:

serialize(value, context)

produce:

SerializedRepresentation
10. Deserializer Contract

Conceptualmente:

deserialize(data, target_type, context)

produce:

Object

o:

DeserializationError
11. Serializer Context

Puede contener:

format
version
locale
timezone
tenant
schema
options

Debe evitarse un contexto global mutable.

12. Representation

Una representación debe poder describirse mediante:

data
format
media_type
schema
version
encoding
13. Schema

El schema define:

Fields
Types
Requiredness
Formats
Nested Structures
Allowed Values
Version

Serialization utiliza el schema.

Validation determina si la representación cumple el schema.

14. Serialization vs Validation

La diferencia debe permanecer clara:

Serializer
→ convierte

Validator
→ verifica

Ejemplo:

Order
 ↓
JSON

es serialization.

JSON
 ↓
"total must be >= 0"

es validation.

15. Serialization vs Transformation

Serialization:

Order
 ↓
JSON representation

Transformation:

Order
 ↓
OrderSummary

La segunda cambia el modelo semántico.

16. Serialization vs Mapping

Mapping:

DomainOrder
 ↓
OrderDTO

Serialization:

OrderDTO
 ↓
JSON

La arquitectura debe poder distinguir ambos pasos.

17. Canonical Representation

Para determinados contratos debe existir una representación canónica:

Object
 ↓
Canonical Serializer
 ↓
Canonical Representation

Esto facilita:

Hashing
Signing
Caching
Comparison
Deduplication
Auditing
18. Deterministic Serialization

Para el mismo:

Object
+
Schema
+
Version
+
Serialization Options

debe producirse la misma representación cuando el formato lo permita.

19. Field Ordering

Si la representación necesita determinismo:

Field ordering

debe estar definida.

Esto es especialmente importante para:

Hash
Signature
Canonical Payload
20. Null Semantics

Debe definirse la diferencia entre:

null

y:

field absent

Ejemplo:

{
  "middle_name": null
}

vs:

{}

No son necesariamente equivalentes.

21. Optional Fields

Los contratos deben definir:

Required
Optional
Nullable
Defaulted

como conceptos distintos.

22. Default Values

La serialización no debe introducir defaults semánticos silenciosamente.

Preferible:

Schema
 ↓
Default explicitly defined

en lugar de asumir:

missing → false

sin contrato.

23. Unknown Fields

Durante deserialización puede existir:

Known fields
+
Unknown fields

La estrategia debe ser explícita:

REJECT
IGNORE
PRESERVE
24. Forward Compatibility

Un consumidor antiguo debería poder manejar nuevos campos cuando el contrato lo permita.

Ejemplo:

v1 consumer
       ↑
v2 producer adds optional field

Debe definirse si el consumidor:

ignores

o:

rejects

el campo.

25. Backward Compatibility

Un nuevo consumidor debe poder leer representaciones antiguas cuando el contrato lo soporte.

26. Breaking Changes

Son potencialmente breaking:

Field removal
Type change
Required field addition
Meaning change
Enum removal
Structural change

Deben pasar por versioning y compatibility review.

27. Serialization Versioning

Un contrato puede utilizar:

Order.v1
Order.v2
Order.v3

La versión debe ser detectable.

28. Version Location

La versión puede expresarse mediante:

Media Type
Schema Identifier
Message Metadata
Envelope
URL
Header

Debe elegirse una estrategia coherente por boundary.

29. Envelope

Para eventos y mensajes:

Envelope
├── message_id
├── message_type
├── schema_version
├── timestamp
├── tenant
└── payload

El payload contiene la representación serializada.

30. API Serialization

En API:

Request
 ↓
Deserializer
 ↓
DTO
 ↓
Validation
 ↓
Application

Respuesta:

Application
 ↓
DTO
 ↓
Serializer
 ↓
Response
31. Event Serialization

Eventos:

Domain Event
 ↓
Event Contract
 ↓
Serializer
 ↓
Event Envelope
 ↓
Message Broker
32. Message Serialization

Mensajes:

Command/Event
 ↓
Message Serializer
 ↓
Transport

El serializer no debe asumir semántica de negocio.

33. Persistence Serialization

Si se requiere serializar objetos para persistencia:

Domain Model
 ↓
Persistence Representation
 ↓
Repository

La persistencia debe utilizar su propio contrato cuando sea necesario.

34. Repository Boundary

E10 define Repository Architecture.

E23 proporciona serialization cuando un repository lo necesita, pero:

Repository
≠
Serializer
35. Date Serialization

Las fechas deben utilizar formatos explícitos.

Ejemplo conceptual:

2026-10-01

para date.

Y:

2026-10-01T14:30:00Z

para timestamp.

No deben confundirse.

36. Timezone

Un timestamp debe tener timezone/offset definido cuando represente un instante.

Evitar:

2026-10-01 14:30:00

si la semántica requiere un instante global.

37. Money Serialization

Money debe conservar:

amount
currency

Ejemplo:

{
  "amount": "150.00",
  "currency": "EUR"
}

La representación debe evitar pérdida de precisión.

38. Decimal Serialization

Los valores financieros no deberían serializarse utilizando representación binaria de floating point si puede provocar pérdida de precisión.

Preferible:

Decimal String

cuando el contrato lo permita.

39. Identifier Serialization

IDs deben tener representación estable:

UUID
ULID
String
Integer

según el contrato.

No debe cambiarse su semántica entre layers.

40. Enum Serialization

Los enums deberían serializarse mediante valores contractuales estables:

PENDING
APPROVED
REJECTED

No depender de:

internal ordinal = 2
41. Boolean Serialization

Debe evitarse representar booleanos inconsistentes:

true
false

en lugar de variantes ambiguas:

"yes"
"no"
"1"
"0"

salvo que el contrato explícitamente las defina.

42. Collection Serialization

Debe definirse:

Array
Set
Map

y su representación.

Ejemplo:

items: []

Debe distinguirse, cuando importe, de:

items: null
43. Map Serialization

Los mapas requieren claves serializables y semántica definida.

Si el formato sólo soporta claves string:

Map<UUID, Value>

requiere una estrategia explícita.

44. Nested Objects

La serialización debe soportar estructuras:

Order
 ├── Customer
 ├── Address
 └── Items
       ├── Item
       └── Item

respetando límites de profundidad.

45. Circular References

Debe evitarse serialización infinita:

Customer
 ↓
Order
 ↓
Customer
 ↓
Order

Estrategias posibles:

REJECT
REFERENCE
IDENTIFIER
CUSTOM PROJECTION
46. Object Identity

Cuando importe preservar identidad:

object_id
reference_id

debe definirse explícitamente.

47. Binary Data

Binary debe tener una estrategia:

Base64
Binary Protocol
External Reference

según el boundary.

No debe introducirse Base64 automáticamente si el protocolo soporta binary nativo.

48. Large Payloads

Para payloads grandes:

Streaming
Chunking
External Object Reference
Compression

pueden ser preferibles.

49. Streaming Serialization

Debe soportarse cuando el volumen lo requiera:

Producer
 ↓
Streaming Serializer
 ↓
Consumer

sin cargar todo el objeto en memoria.

50. Streaming Safety

Debe existir:

Max Stream Size
Timeout
Backpressure
Cancellation
51. Compression

Compression es una preocupación distinta:

Serialization
 ↓
Compression
 ↓
Transport

No debe mezclarse semánticamente con serialization.

52. Encryption

Encryption también debe ser una capa separada:

Serialize
 ↓
Encrypt
 ↓
Transport

o:

Serialize
 ↓
Compress
 ↓
Encrypt

según protocolo.

53. Serialization Security

Debe proteger contra:

Unsafe Deserialization
Object Injection
Resource Exhaustion
Recursive Payloads
Oversized Payloads
Malformed Encodings
Schema Abuse
54. Unsafe Deserialization

Nunca debe permitirse:

Arbitrary Class Instantiation

a partir de input no confiable.

Debe utilizarse:

Explicit Type Registry

o:

Schema-Based Deserialization
55. Type Allowlist

Los tipos deserializables deben estar restringidos:

Allowed Types

No:

Any Type
56. Payload Limits

Debe haber límites para:

Max Payload Size
Max Field Size
Max Array Length
Max Nesting Depth
Max String Length
57. Parser Security

Los parsers deben proteger contra:

Entity Expansion
Recursive Structures
Parser Bombs
Malformed Unicode

según el formato utilizado.

58. Schema Security

Schemas externos no deben poder causar:

Unbounded Resolution
Remote Code Execution
Unexpected Network Access
59. Serialization Errors

Debe distinguirse:

SerializationError
DeserializationError
SchemaError
EncodingError
UnsupportedFormatError
60. Error Taxonomy
SERIALIZATION_ERROR
 ├── INVALID_VALUE
 ├── UNSUPPORTED_TYPE
 ├── UNSUPPORTED_FORMAT
 ├── ENCODING_ERROR
 └── SIZE_LIMIT_EXCEEDED

DESERIALIZATION_ERROR
 ├── MALFORMED_INPUT
 ├── INVALID_TYPE
 ├── UNKNOWN_FIELD
 ├── INVALID_SCHEMA
 └── UNSUPPORTED_VERSION
61. Partial Deserialization

Debe definirse si el sistema permite:

Partial Object

cuando algunos campos son inválidos.

Para contratos críticos, normalmente:

all-or-nothing

es preferible.

62. Atomic Deserialization

La deserialización debería producir:

Complete Valid Representation

o:

Error

evitando objetos parcialmente inicializados.

63. Serialization Registry

Debe existir un registry para:

Type
 ↓
Serializer
Deserializer
Schema
Version
64. Codec Registry

Un codec puede representar:

encode
decode

para un formato.

Ejemplo:

JSONCodec
ProtobufCodec
MessagePackCodec
65. Serializer Selection

La selección puede depender de:

Type
Format
Media Type
Version
Context
Boundary
66. Content Negotiation

En API:

Accept
Content-Type

pueden determinar el serializer.

Ejemplo:

Accept: application/json
67. Explicit Format Selection

Para mensajes internos puede ser preferible:

Message Type
+
Schema
+
Serializer

sin negociación dinámica.

68. Serialization Profiles

Puede existir:

API Profile
Event Profile
Message Profile
Persistence Profile
Audit Profile

Cada uno con necesidades diferentes.

69. Projection

No siempre debe serializarse el objeto completo.

Puede utilizarse una proyección:

Order
 ↓
OrderSummary
 ↓
Serializer

Esto reduce:

Payload Size
Data Exposure
Coupling
70. Data Minimization

Los serializers deben evitar exponer:

Internal IDs
Secrets
Credentials
Internal Metadata
Sensitive Fields

salvo que el contrato lo requiera.

71. Serialization Boundary Security

La pregunta debe ser:

What data is allowed to leave this boundary?

no simplemente:

How do we serialize the object?
72. Sensitive Field Policies

Puede declararse:

public
internal
sensitive
secret

y el serializer debe respetar estas categorías cuando corresponda.

73. Redaction

Para logs/audits:

password
token
secret

deben serializarse como:

[REDACTED]

o excluirse.

74. Logging

Nunca debe registrarse automáticamente el payload serializado completo si contiene información sensible.

75. Serialization Observability

Métricas:

serialization_total
deserialization_total
serialization_errors_total
deserialization_errors_total
serialization_latency
deserialization_latency
payload_size
76. Format Metrics

Debe poder observarse:

serialization_total{format="json"}
serialization_total{format="protobuf"}

cuando la infraestructura de métricas lo permita.

77. Payload Size Metrics

Medir:

P50
P95
P99
max

puede revelar:

Payload Growth
Unexpected Fields
Serialization Regression
78. Serialization Tracing

Una trace puede registrar:

format
schema_version
payload_size
serialization_duration

sin almacenar necesariamente el payload.

79. Serialization Testing

Cada serializer debe tener:

Round Trip Tests
Compatibility Tests
Schema Tests
Boundary Tests
Security Tests
Performance Tests
80. Round Trip

Para representaciones compatibles:

Object
 ↓ serialize
Representation
 ↓ deserialize
Object'

debe cumplirse:

Object ≈ Object'

según la semántica definida.

81. Canonical Round Trip

Cuando existe representación canónica:

serialize(
    deserialize(
        canonical_representation
    )
)

debe producir la misma representación canónica.

82. Compatibility Testing

Debe probarse:

Producer v1 → Consumer v1
Producer v2 → Consumer v1
Producer v1 → Consumer v2
Producer v2 → Consumer v2

cuando el contrato requiera compatibilidad cruzada.

83. Golden Payloads

Deben mantenerse ejemplos contractuales:

fixtures/
 ├── order-v1.json
 ├── order-v2.json
 └── event-v3.json

para detectar cambios accidentales.

84. Schema Regression

Una modificación del serializer no debe cambiar silenciosamente:

Field Names
Types
Formats
Null Semantics
Enum Values

de un contrato estable.

85. Fuzz Testing

Los deserializers críticos deberían probarse con:

Malformed Inputs
Random Inputs
Oversized Inputs
Deeply Nested Inputs
Unexpected Types
86. Performance Testing

Debe medirse:

Serialization Throughput
Deserialization Throughput
Latency
Memory
Payload Size
87. Version Migration

Cuando se elimina un serializer antiguo:

v1
 ↓
Migration
 ↓
v2

debe existir una estrategia explícita.

88. Dual Serialization

Durante una migración puede utilizarse:

Object
 ├── Serializer v1
 └── Serializer v2

para validar compatibilidad.

89. Shadow Serialization

Puede ejecutarse un nuevo serializer sin cambiar el output principal:

Production Serializer
        +
Shadow Serializer
        ↓
Compare

Esto permite detectar diferencias antes del rollout.

90. Serialization Drift

Debe detectarse:

Expected Representation
        ≠
Observed Representation

especialmente en contratos compartidos.

91. Contract Ownership

Cada formato/schema debe tener:

Owner
Version
Lifecycle
Compatibility Policy
92. Schema Registry

Para eventos y mensajes puede existir:

Schema Registry

responsable de:

Schema Storage
Versioning
Compatibility
Lookup
93. Schema Registry vs Serializer Registry

No son lo mismo:

Schema Registry
→ describe data

Serializer Registry
→ describes how data is encoded
94. API Contract Integration

E06/E03 pueden definir contracts.

E23 materializa esos contratos:

Contract
 ↓
Serializer
 ↓
Wire Representation
95. Event Contract Integration

E07/A07 y E12/E13 utilizan serialization para transportar eventos.

Event
 ↓
Contract
 ↓
Serializer
 ↓
Broker
96. Messaging Integration

Mensajes:

Command
Event
Reply

pueden compartir infraestructura de serialization, pero no necesariamente los mismos schemas.

97. Serialization and Caching

E17 puede almacenar:

Serialized Representation

cuando sea eficiente.

Pero debe conocerse:

format
version
schema

para evitar cache poisoning semántico.

98. Serialization and Configuration

E18 puede utilizar serialization para transportar configuration:

Config
 ↓
Serialize
 ↓
Storage / Distribution

Pero la semántica de configuration permanece en E18.

99. Serialization and Feature Flags

E19 puede serializar snapshots:

Feature Flags
 ↓
Serialized Snapshot

sin convertir serialization en feature-flag logic.

100. Serialization and Rules

E21 puede serializar:

Rule Definitions
Rule Results
Evaluation Contexts

para persistencia, distribución o debugging.

101. Serialization and Validation

La secuencia típica:

Wire Representation
        ↓
Deserialize
        ↓
Structural Validation
        ↓
Semantic Validation
        ↓
Application

En algunos sistemas puede validarse parcialmente antes de deserializar completamente, pero la estrategia debe ser explícita.

102. Serialization and Transformation

La frontera:

Object A
 ↓
Transformation
 ↓
Object B
 ↓
Serialization
 ↓
Representation

es preferible a esconder transformaciones dentro del serializer.

103. Serialization Control Plane

Debe gobernar:

Schemas
Formats
Versions
Compatibility
Registries
Lifecycle
104. Serialization Data Plane

Debe ejecutar:

Serialize
Deserialize
Encode
Decode

de forma eficiente.

105. Runtime Architecture

El runtime debería cargar:

Serializer Registry
Schema Registry
Codec Registry

y utilizar representaciones versionadas.

106. Locality

Para hot paths:

Registry
 ↓
Local Cache
 ↓
Serializer

debe evitarse una llamada remota por cada operación.

107. Bounded Serialization

Deben existir límites para:

Depth
Payload Size
Collection Size
String Size
Processing Time
108. Cancellation

Operaciones de serialization/serialization streaming deberían soportar:

Cancellation
Timeout
Backpressure

cuando el runtime lo requiera.

109. Idempotence

Serialization debe ser funcionalmente idempotente en el sentido de:

same object
+
same serialization contract
→
same representation

cuando el contrato sea determinista.

110. Failure Isolation

Un payload inválido no debe provocar:

Process-wide failure

si el boundary permite aislar el error.

111. Deserialization Quarantine

Para mensajes no confiables puede existir:

Input
 ↓
Deserialize
 ↓
Validation
 ↓
Accept

o:

Invalid
 ↓
Dead Letter / Quarantine

según Messaging Architecture.

112. Dead Letter Boundary

El serializer puede clasificar:

Malformed Message
Unsupported Version
Schema Error

para que el sistema de messaging determine su tratamiento.

113. Serialization Policy

Las decisiones sobre:

Allowed Formats
Allowed Versions
Maximum Payload Size
Unknown Field Policy

pueden gobernarse mediante policy/configuration.

114. Serialization Governance

Cambios críticos deben registrar:

Who
What
Why
Version
Compatibility
Migration
Tests
115. Engineering Principles
Principle 1

Serialization converts representations; it does not own business semantics.

Principle 2

Domain models should remain independent from wire formats whenever practical.

Principle 3

Every external representation must have an explicit contract.

Principle 4

Serialization versions must be identifiable.

Principle 5

Unknown-field behavior must be explicit.

Principle 6

Null and absent-field semantics must be explicit.

Principle 7

Serialization must be deterministic where canonicalization is required.

Principle 8

Deserialization of untrusted input must be type-safe.

Principle 9

Arbitrary class instantiation must never be allowed from untrusted data.

Principle 10

Payload limits are mandatory for untrusted boundaries.

Principle 11

Serialization must not silently perform business transformations.

Principle 12

Sensitive data must not leak through serialization.

Principle 13

Compression and encryption remain separate concerns.

Principle 14

Schemas and serializers have different responsibilities.

Principle 15

Contract compatibility must be tested.

Principle 16

Serialization failures must be distinguishable from validation failures.

Principle 17

Serialization should be local and efficient on hot runtime paths.

Principle 18

Version migrations must be explicit.

Principle 19

Serialization must preserve semantic precision.

Principle 20

The representation leaving a boundary must contain only data authorized for that boundary.

116. Definition of Done

E23 queda definido cuando EVOXA dispone de:

✓ Serialization Architecture
✓ Deserialization Architecture
✓ Serializer Contract
✓ Deserializer Contract
✓ Serialization Context
✓ Representation Model
✓ Schema Integration
✓ Format Registry
✓ Codec Registry
✓ Serializer Registry
✓ JSON Support
✓ Binary Format Boundary
✓ Format Versioning
✓ Schema Versioning
✓ Canonical Representation
✓ Deterministic Serialization
✓ Field Ordering
✓ Null Semantics
✓ Optional Fields
✓ Default Semantics
✓ Unknown Field Strategy
✓ Forward Compatibility
✓ Backward Compatibility
✓ Breaking Change Policy
✓ Version Identification
✓ Envelope Model
✓ API Serialization
✓ Event Serialization
✓ Message Serialization
✓ Persistence Serialization
✓ Date Serialization
✓ Timezone Semantics
✓ Money Serialization
✓ Decimal Serialization
✓ Identifier Serialization
✓ Enum Serialization
✓ Collection Serialization
✓ Map Serialization
✓ Nested Object Serialization
✓ Circular Reference Handling
✓ Object Identity
✓ Binary Data Handling
✓ Large Payload Handling
✓ Streaming Serialization
✓ Compression Boundary
✓ Encryption Boundary
✓ Serialization Security
✓ Safe Deserialization
✓ Type Allowlisting
✓ Payload Limits
✓ Parser Security
✓ Serialization Errors
✓ Deserialization Errors
✓ Partial/Atomic Deserialization Policy
✓ Projection Support
✓ Data Minimization
✓ Sensitive Field Handling
✓ Redaction
✓ Observability
✓ Payload Metrics
✓ Serialization Tracing
✓ Round-Trip Testing
✓ Compatibility Testing
✓ Golden Payloads
✓ Schema Regression Testing
✓ Fuzz Testing
✓ Performance Testing
✓ Version Migration
✓ Dual Serialization
✓ Shadow Serialization
✓ Serialization Drift Detection
✓ Contract Ownership
✓ Schema Registry
✓ Registry Separation
✓ Cache Integration
✓ Runtime Integration
✓ Local Serializer Caching
✓ Bounded Serialization
✓ Cancellation
✓ Failure Isolation
✓ Quarantine Boundary
✓ Dead-Letter Integration
✓ Serialization Governance
117. Position in Engineering Specification

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

Y aparece una cadena especialmente importante:

                  EVOXA Runtime
                       │
                       ▼
              External Representation
                       │
                       ▼
                E23 Serialization
                       │
                       ▼
                E22 Validation
                       │
                       ▼
              Application Boundary
                       │
                       ▼
                E21 Rules Engine
                       │
                       ▼
                E20 Runtime Policy
                       │
                       ▼
                  Domain Logic

La separación esencial de E21–E23 es:

E21 — Rules
        │
        │  Decide
        ▼

E22 — Validation
        │
        │  Verify
        ▼

E23 — Serialization
        │
        │  Represent
        ▼

External / Internal Boundary

E21 decide mediante reglas; E22 verifica validez; E23 representa y reconstruye datos. Ninguna de las tres capas debe absorber las responsabilidades de las otras.

Siguiente capítulo: E24 — EVOXA Transformation Architecture.
