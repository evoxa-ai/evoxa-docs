E22 — EVOXA Validation Architecture
Engineering Specification

Pertenece a: Engineering Specification
Volumen: volume-09-engineering
Capítulo: E22 — Validation Architecture
Anterior: E21 — Rules Engine Architecture
Siguiente: E23 — Serialization Architecture

1. Propósito

E22 define la arquitectura de Validation de EVOXA.

Validation es la capacidad responsable de determinar si una entrada, estructura, estado, transición, configuración, regla, comando o contrato satisface las restricciones necesarias para continuar dentro del sistema.

La distinción fundamental es:

E21 — Rules Engine
        ↓
Evalúa reglas y produce decisiones

E22 — Validation
        ↓
Determina si algo es válido

Validation no debe convertirse en un segundo Rules Engine ni en una duplicación de Domain Logic.

2. Objetivos

La arquitectura debe proporcionar:

Input Validation
Schema Validation
Type Validation
Structural Validation
Semantic Validation
Business Validation
Domain Validation
Command Validation
Request Validation
Configuration Validation
Rule Validation
State Validation
Transition Validation
Cross-Field Validation
Cross-Entity Validation
Validation Composition
Validation Reporting
Validation Severity
Validation Lifecycle
Validation Versioning
Validation Testing
3. Concepto fundamental

Validation responde:

¿Es válido?

mientras que:

Rule Engine

responde:

¿Qué decisión corresponde dadas estas condiciones?

Ejemplo:

Input:
amount = -50

Validation:
INVALID

Rule:
amount > 1000
    → require_approval

La regla no debería sustituir la validación de que amount tenga un valor válido.

4. Validation Architecture
                    Validation Control Plane
                             │
              ┌──────────────┼──────────────┐
              ▼              ▼              ▼
           Schemas        Validators      Versions
              │              │              │
              └──────────────┼──────────────┘
                             ▼
                    Validation Runtime
                             │
          ┌──────────────────┼──────────────────┐
          ▼                  ▼                  ▼
       Structural         Semantic            Domain
       Validation         Validation          Validation
          │                  │                  │
          └──────────────────┼──────────────────┘
                             ▼
                    Validation Result
                             │
              ┌──────────────┼──────────────┐
              ▼              ▼              ▼
           Errors         Warnings        Metadata
5. Validation Layers

La validación debe organizarse por niveles:

Transport
   ↓
Schema
   ↓
Type
   ↓
Structure
   ↓
Semantic
   ↓
Domain
   ↓
State
   ↓
Authorization / Policy

No todos los objetos necesitan todas las capas.

6. Transport Validation

Comprueba aspectos básicos del mensaje recibido:

Content-Type
Encoding
Payload presence
Request shape
Required headers
Protocol constraints

No debe contener reglas de negocio.

7. Schema Validation

Comprueba:

Required fields
Allowed fields
Field types
Nested structures
Array structures
Formats
Constraints

Ejemplo:

name: string
age: integer
email: email
8. Type Validation

Debe impedir inconsistencias como:

age = "twenty"

cuando se requiere:

age: integer

Los tipos deben validarse antes de ejecutar lógica dependiente de ellos.

9. Structural Validation

Comprueba relaciones estructurales:

Object hierarchy
Required relationships
Collection constraints
Nested object consistency
10. Semantic Validation

Determina si una estructura tiene significado válido.

Ejemplo:

start_date = 2027-01-10
end_date   = 2027-01-01

Ambos valores son válidos individualmente.

La combinación es inválida.

11. Cross-Field Validation

Permite:

password == password_confirmation

o:

start_date < end_date
12. Domain Validation

La validación de dominio protege invariantes:

Account.balance >= 0
Order cannot be shipped before confirmation
Refund cannot exceed captured amount

Esta validación pertenece al dominio cuando expresa una invariancia fundamental del modelo.

13. Domain Invariants

Una invariant:

Debe ser verdadera siempre que el agregado esté en un estado válido.

Por ejemplo:

Order.status == SHIPPED
→ shipment must exist
14. Application Validation

Application Services pueden validar:

Command completeness
Use-case prerequisites
Input combinations
Workflow prerequisites

pero no deben duplicar invariantes profundas del dominio.

15. Command Validation

Antes de ejecutar un command:

Command
   ↓
Validation
   ↓
Application Service

Ejemplo:

CreateOrderCommand

debe comprobar que la entrada tiene la forma necesaria.

16. Query Validation

Las queries también pueden validarse:

pagination.limit
pagination.offset
filters
sort fields
date ranges
17. Configuration Validation

E18 puede proporcionar configuration.

E22 valida:

Required configuration
Type correctness
Allowed ranges
Cross-setting constraints
Environment requirements

Ejemplo:

timeout = -5

debe rechazarse.

18. Feature Flag Validation

E19 puede definir flags.

E22 puede comprobar:

Flag schema
Allowed values
Targeting configuration
Dependency consistency
19. Policy Validation

E20 puede definir policies.

E22 puede comprobar:

Policy schema
Required parameters
Referenced resources
Policy consistency
20. Rule Validation

E21 define reglas.

E22 puede proporcionar mecanismos para:

Rule syntax validation
Rule schema validation
Rule input validation
Rule action validation
Rule dependency validation

La evaluación sigue perteneciendo a E21.

21. Validation vs Rules

La separación debe ser estricta:

Validation:
"amount debe ser >= 0"

Rule:
"si amount > 10000 entonces require_approval"

Validation determina validez.

Rules determinan decisión.

22. Validation vs Authorization

Authorization responde:

¿Puede este actor realizar esta operación?

Validation responde:

¿La entrada/estado es válido?

Una petición puede ser:

Valid + Unauthorized

o:

Authorized + Invalid
23. Validation vs Policy

Policy:

¿Qué restricciones aplican?

Validation:

¿La entrada cumple las restricciones validables?
24. Validation Contract

Cada validator debe definir:

Input Type
Validation Scope
Rules
Output
Errors
Severity
Dependencies
Version
25. Validator

Conceptualmente:

validate(input, context)

produce:

ValidationResult
26. Validation Result

Debe representar:

valid
errors
warnings
metadata

Ejemplo:

{
  valid: false,
  errors: [
    {
      code: "INVALID_AMOUNT",
      field: "amount"
    }
  ]
}
27. Validation Error

Cada error debe tener un código estable:

INVALID_EMAIL
INVALID_AMOUNT
MISSING_CUSTOMER
INVALID_STATE
DATE_RANGE_INVALID

Los códigos no deben depender del texto mostrado al usuario.

28. Error Message

Debe separarse:

error_code
message
details

Esto permite internacionalización y evolución de mensajes.

29. Error Path

Los errores deben poder identificar dónde ocurre el problema:

customer.email
order.items[3].quantity
payment.card.expiry
30. Validation Severity

Como mínimo:

ERROR
WARNING
INFO

Pero:

ERROR

es lo que determina normalmente que una operación no pueda continuar.

31. Fail Fast

Puede utilizarse cuando:

Una validación inicial invalida completamente la operación.

Ejemplo:

Malformed JSON
32. Collect All Errors

Para formularios y APIs puede ser preferible:

Validate all fields
      ↓
Return all validation errors

en lugar de devolver solamente el primero.

33. Validation Strategy

Cada caso debe declarar si utiliza:

FAIL_FAST
COLLECT_ALL

No debe quedar implícito.

34. Validation Composition

Los validators pueden componerse:

UserValidator
 ├── IdentityValidator
 ├── ContactValidator
 └── AddressValidator
35. Composite Validation
Input
 ↓
Validator A
 ↓
Validator B
 ↓
Validator C
 ↓
Combined Result
36. Validation Ordering

El orden puede importar:

Schema
 ↓
Type
 ↓
Structure
 ↓
Semantic
 ↓
Domain

Un validator posterior no debería asumir que los invariantes anteriores ya se cumplen sin un contrato explícito.

37. Validation Dependencies

Un validator puede requerir:

Repository
Service
Configuration
Clock
Context

pero las dependencias deben mantenerse controladas.

38. External Validation

Ejemplo:

Validate postal code
      ↓
External Address Service

Debe definirse:

Timeout
Retry
Fallback
Failure Semantics
39. Validation Availability

La validación local no debería depender innecesariamente de servicios externos.

Preferible:

Local Validation
      ↓
External Validation
only when necessary
40. Validation Context

Puede contener:

tenant
actor
operation
resource
timestamp
locale
environment
41. Context Isolation

Los validators deben recibir únicamente el contexto necesario.

No deben depender de un contexto global mutable.

42. Deterministic Validation

Idealmente:

same input
+
same context
+
same validator version
=
same result
43. Time-Based Validation

Cuando depende del tiempo:

Clock

debe inyectarse.

No debe utilizarse directamente el reloj global del sistema en lógica testeable.

44. Validation Versioning

Los schemas y validators pueden evolucionar:

UserValidator:v1
UserValidator:v2

Debe ser posible identificar qué versión produjo un resultado.

45. Backward Compatibility

Cuando evoluciona un contrato:

v1
→
v2

debe determinarse:

Compatible
Breaking
Migration Required
46. Validation Schema Evolution

Cambios potencialmente breaking:

Remove required field
Change type
Narrow allowed values
Change semantic constraints

deben pasar por revisión.

47. Validation Profiles

Un mismo objeto puede tener perfiles:

CREATE
UPDATE
PATCH
IMPORT
PUBLISH
ACTIVATE

Cada perfil puede tener requisitos distintos.

48. Partial Validation

Para PATCH:

Only provided fields
+
cross-field constraints when applicable

No debe tratarse automáticamente como un CREATE completo.

49. State Validation

Debe validarse el estado actual:

Entity
   ↓
State Validator

Ejemplo:

Order cannot be shipped if CANCELLED
50. Transition Validation

Una transición:

PENDING → SHIPPED

debe validarse explícitamente.

Modelo:

Current State
+
Requested Transition
+
Context
↓
Validation
51. State Machine Integration

Cuando exista una máquina de estados:

State Machine
     ↓
Transition Validation

La validación debe respetar el modelo de estados canónico.

52. Entity Validation

Una entidad debe mantenerse en estado válido después de una operación.

53. Aggregate Validation

Los aggregates pueden validar invariantes que involucran múltiples objetos internos.

54. Cross-Entity Validation

Algunas reglas requieren múltiples entidades:

Order
+
Customer
+
Account

Estas validaciones deben pertenecer a Application/Domain Services cuando corresponda, evitando hacer del validator un servicio omnisciente.

55. Database Validation

La base de datos debe proteger invariantes estructurales mediante:

NOT NULL
UNIQUE
FOREIGN KEY
CHECK

cuando corresponda.

Validation de aplicación no sustituye constraints de persistencia.

56. Defense in Depth

La integridad debe protegerse en varias capas:

API
 ↓
Application
 ↓
Domain
 ↓
Persistence

No confiar exclusivamente en una única capa.

57. Validation and Transactions

Una validación crítica relacionada con estado mutable puede necesitar ejecutarse dentro del boundary transaccional apropiado.

Ejemplo:

Validate
+
Persist

no siempre es suficiente si otro proceso puede modificar el estado entre ambos pasos.

58. TOCTOU

Debe evitarse:

Check
 ↓
State changes
 ↓
Use old assumption

Cuando corresponda, utilizar:

Transaction
Optimistic Lock
Constraint
Atomic Operation
59. Validation and Concurrency

Validators que dependen de estado compartido deben considerar:

Concurrent Updates
Race Conditions
Stale Reads
Version Conflicts
60. Validation Caching

Puede cachearse validación cuando sea:

Pure
Stable
Versioned
Context-independent

No debe cachearse ciegamente validación dependiente de estado mutable.

61. Validation Cache Key

Debe incorporar cuando corresponda:

input hash
validator version
context version
relevant state version
62. Validation Performance

Deben medirse:

validation_latency
validation_errors
validation_count
validation_external_calls
63. Validation Limits

Debe haber límites para:

Payload Size
Collection Size
Nesting Depth
Validation Steps
External Calls
64. Recursive Validation

Debe evitar:

Infinite Recursive Validation

mediante:

Depth Limit
Cycle Detection
Visited Set
65. Validation Security

La validación también es una frontera de seguridad.

Debe ayudar a prevenir:

Malformed Input
Injection Payloads
Oversized Payloads
Unexpected Structures
Unsafe Encodings

Pero no sustituye controles especializados de seguridad.

66. Validation Sanitization

Debe diferenciarse:

Validation

de:

Sanitization

Validar:

"input is invalid"

no significa necesariamente transformar:

input → sanitized input
67. Canonicalization

Cuando sea necesario:

Normalize
Canonicalize
Then Validate

La estrategia debe ser explícita.

68. Unicode Validation

Los contratos de texto deben considerar:

Encoding
Normalization
Length Semantics
Allowed Characters

cuando corresponda.

69. Format Validation

Formatos típicos:

UUID
Email
URL
Date
DateTime
Currency
Country Code
Phone

deben utilizar validadores consistentes.

70. Domain-Specific Validators

Ejemplos:

MoneyValidator
CurrencyValidator
OrderValidator
CustomerValidator
TenantValidator
PermissionValidator

Deben vivir cerca del dominio que conocen.

71. Generic Validators

Ejemplos:

Required
Min
Max
Length
Pattern
Enum
Format

pueden formar parte del framework de validation.

72. Custom Validators

Cuando un requisito es específico del dominio:

RefundAmountValidator

debe poder implementarse sin modificar el núcleo genérico.

73. Validator Registry

Puede existir:

ValidatorRegistry

que resuelva:

type + validation_profile

hacia el validator correspondiente.

74. Validation Metadata

Los validators pueden declarar:

name
version
owner
scope
dependencies
severity
75. Validation Lifecycle
DEFINE
 ↓
REGISTER
 ↓
VALIDATE
 ↓
TEST
 ↓
APPROVE
 ↓
PUBLISH
 ↓
EXECUTE
 ↓
OBSERVE
 ↓
DEPRECATE
 ↓
ARCHIVE
76. Validation Testing

Cada validator debe tener:

Valid Cases
Invalid Cases
Boundary Cases
Null Cases
Missing Cases
Type Cases
Concurrency Cases
External Failure Cases

según corresponda.

77. Property-Based Testing

Para validadores adecuados:

Generated Input
 ↓
Validator
 ↓
Invariant

puede complementar los tests tradicionales.

78. Regression Testing

Cambiar un validator debe ejecutar fixtures históricos.

79. Contract Testing

Los validators de contratos compartidos deben comprobar compatibilidad entre:

Producer
Consumer
Schema
80. Validation Explainability

El sistema debe poder responder:

Why is this invalid?
Which constraint failed?
Which validator detected it?
Which version was used?
81. Validation Trace

Ejemplo:

RequestValidator → PASS
SchemaValidator  → PASS
OrderValidator   → FAIL
  └── quantity must be > 0
82. Validation Audit

Para operaciones críticas:

validator
version
input reference
result
timestamp
actor
tenant

puede auditarse.

Nunca se debe almacenar automáticamente el payload sensible completo.

83. Validation Observability

Métricas:

validation_total
validation_success_total
validation_failure_total
validation_latency
validation_error_code_total
84. Validation Error Rate

Un aumento repentino:

1% → 18%

puede indicar:

Client Regression
Contract Change
Deployment Error
Schema Drift
85. Schema Drift

Debe detectarse cuando:

Expected Schema
        ≠
Observed Payload
86. Validation and API Architecture

E03 proporciona contratos API.

E22 valida:

Request
Response
Parameters
Payload

pero no define la API.

87. Validation and Database Architecture

E02 proporciona constraints de persistencia.

E22 proporciona validación de aplicación.

Ambos deben estar alineados.

88. Validation and Repository Architecture

E10 proporciona acceso a persistencia.

Un validator no debería convertirse en una capa de acceso arbitrario a repositories.

Cuando necesita existencia o estado, la dependencia debe ser explícita.

89. Validation and Domain Services

E08 contiene Domain Services.

Cuando una validación representa conocimiento de dominio complejo, el Domain Service puede ser el lugar correcto.

90. Validation and Application Services

E09 coordina casos de uso.

Su responsabilidad incluye orquestar:

Validate
+
Authorize
+
Execute

en el orden correcto.

91. Validation and Rules Engine

La relación canónica:

Input
 ↓
Validation
 ↓
Rules Evaluation
 ↓
Decision
 ↓
Execution

No:

Input
 ↓
Rules Engine
 ↓
Hope it's valid
92. Validation and Policy Engine
Policy
 ↓
Constraints
 ↓
Validation

cuando una policy se materializa como restricciones verificables.

93. Validation and Workflow

Cada transición crítica del workflow puede requerir:

Precondition Validation

y eventualmente:

Postcondition Validation
94. Validation and Events

Antes de publicar un evento:

Domain Event
 ↓
Event Contract Validation
 ↓
Publish

Esto reduce eventos inválidos.

95. Validation and Messaging

Los mensajes entrantes deben validarse:

Message
 ↓
Schema Validation
 ↓
Semantic Validation
 ↓
Handler
96. Validation and Jobs

Jobs deben validar:

Job Payload
Job Parameters
Execution Context

antes de procesarse.

97. Validation and Scheduling

Schedules deben validar:

Cron / Schedule Syntax
Timezone
Start / End
Frequency
Constraints
98. Validation and Serialization

La serialización transforma:

Object ↔ Representation

Validation determina:

Representation / Object válido

Las responsabilidades deben permanecer separadas.

99. Validation Architecture Boundary

La frontera canónica:

              Validation
                   │
      ┌────────────┼────────────┐
      ▼            ▼            ▼
    Schema       Domain       State
      │            │            │
      └────────────┼────────────┘
                   ▼
             Valid Result
                   │
                   ▼
           Application Flow
100. Canonical Validation Flow
Input
  ↓
Canonicalization (if required)
  ↓
Schema Validation
  ↓
Type Validation
  ↓
Structural Validation
  ↓
Semantic Validation
  ↓
Domain Validation
  ↓
State / Transition Validation
  ↓
Validation Result
  ↓
Application Execution
101. Failure Model

Validation failure debe producir:

ValidationError

y no:

NullPointerException
Generic Exception
Database Error

cuando el problema sea realmente una entrada inválida.

102. Error Taxonomy
VALIDATION_ERROR
 ├── REQUIRED_FIELD
 ├── INVALID_TYPE
 ├── INVALID_FORMAT
 ├── INVALID_VALUE
 ├── INVALID_RELATION
 ├── INVALID_STATE
 ├── INVALID_TRANSITION
 ├── INVALID_DOMAIN_STATE
 └── VALIDATION_DEPENDENCY_FAILURE
103. Dependency Failure

Si una validación externa falla:

Validation unavailable

no necesariamente significa:

Input invalid

Debe distinguirse:

INVALID

de:

UNABLE_TO_VALIDATE
104. Validation Availability Semantics

Resultado posible:

VALID
INVALID
UNKNOWN

Esto es especialmente importante para validaciones dependientes de sistemas externos.

105. Fail-Closed Validation

Para controles críticos:

UNKNOWN
→ BLOCK

cuando no sea seguro continuar.

106. Fail-Open Validation

Sólo cuando el dominio explícitamente lo permita:

UNKNOWN
→ CONTINUE

Debe ser una decisión arquitectónica, no un accidente.

107. Validation Security Boundary

Las validaciones de seguridad críticas deben ejecutarse:

Server-side

Nunca confiar exclusivamente en:

Client-side Validation
108. Client Validation

Puede mejorar UX:

Client
 ↓
Fast Feedback

pero:

Server
 ↓
Authoritative Validation

es la fuente definitiva.

109. Validation Contract Ownership

Cada contrato debe tener un owner:

API Team
Domain Team
Platform Team
Security Team

según corresponda.

110. Validation Governance

Los cambios en validaciones críticas deben registrar:

Who
What
Why
Version
Impact
Tests
Approval
111. Validation Drift

Debe evitarse que:

API schema
≠
Application validator
≠
Domain invariant
≠
Database constraint

sin una razón explícita.

112. Single Source of Truth

No siempre significa un único archivo.

Significa:

Cada constraint debe tener una autoridad claramente definida.

Ejemplo:

Database uniqueness
→ Database

Domain invariant
→ Domain

API shape
→ API Contract

Runtime policy
→ Policy

Decision rule
→ Rules Engine
113. Validation Architecture Principle

La validación debe estar distribuida según responsabilidad, no centralizada artificialmente.

Transport → Transport Validation
API       → Contract Validation
Application → Use-case Validation
Domain    → Invariant Validation
Database  → Persistence Constraints
114. Definition of Done

E22 queda definido cuando EVOXA dispone de:

✓ Validation Architecture
✓ Validation Layers
✓ Transport Validation
✓ Schema Validation
✓ Type Validation
✓ Structural Validation
✓ Semantic Validation
✓ Domain Validation
✓ Application Validation
✓ Command Validation
✓ Query Validation
✓ Configuration Validation
✓ Feature Flag Validation
✓ Policy Validation
✓ Rule Validation
✓ Cross-Field Validation
✓ State Validation
✓ Transition Validation
✓ Entity Validation
✓ Aggregate Validation
✓ Cross-Entity Validation
✓ Database Constraint Alignment
✓ Validation Contracts
✓ Validators
✓ Validation Results
✓ Validation Errors
✓ Error Codes
✓ Error Paths
✓ Severity
✓ Fail-Fast Strategy
✓ Collect-All Strategy
✓ Validation Composition
✓ Ordering
✓ Dependencies
✓ External Validation
✓ Context
✓ Context Isolation
✓ Determinism
✓ Time Injection
✓ Versioning
✓ Compatibility
✓ Validation Profiles
✓ Partial Validation
✓ State Validation
✓ State Machine Integration
✓ Validation Caching
✓ Performance Controls
✓ Recursive Validation Protection
✓ Security Validation
✓ Sanitization Boundary
✓ Canonicalization
✓ Unicode Handling
✓ Format Validation
✓ Generic Validators
✓ Domain Validators
✓ Custom Validators
✓ Validator Registry
✓ Validation Metadata
✓ Lifecycle
✓ Testing
✓ Regression Testing
✓ Property-Based Testing
✓ Contract Testing
✓ Explainability
✓ Traceability
✓ Audit
✓ Observability
✓ Schema Drift Detection
✓ API Integration
✓ Database Integration
✓ Repository Boundary
✓ Domain Service Integration
✓ Application Service Integration
✓ Rules Engine Integration
✓ Policy Integration
✓ Workflow Integration
✓ Event Integration
✓ Messaging Integration
✓ Job Integration
✓ Scheduling Integration
✓ Serialization Boundary
✓ Failure Model
✓ Error Taxonomy
✓ Dependency Failure Semantics
✓ UNKNOWN State
✓ Fail-Closed Strategy
✓ Fail-Open Strategy
✓ Server-Side Authority
✓ Governance
✓ Constraint Ownership
✓ Drift Prevention
✓ Single Source of Truth
115. Position in Engineering Specification

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

Y la separación conceptual:

E18
Configuration
     ↓
E19
Feature Exposure
     ↓
E20
Runtime Governance
     ↓
E21
Decision Rules
     ↓
E22
Validation
     ↓
E23
Serialization
     ↓
Runtime / APIs / Messaging / Persistence

Principio central de E22:

Validation determina si una representación, entrada, estado o transición satisface un contrato o invariant; no debe convertirse en un sustituto de Policy, Rules, Domain Logic, Authorization ni Workflow.
