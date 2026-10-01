9. Rule Lifecycle
CREATE
   ↓
VALIDATE
   ↓
TEST
   ↓
APPROVE
   ↓
STAGE
   ↓
ACTIVATE
   ↓
EXECUTE
   ↓
OBSERVE
   ↓
SUPERSEDE
   ↓
ARCHIVE
10. Declarative Rules

Las reglas deben ser preferentemente declarativas:

WHEN order.status == "pending"
AND order.age_days > 30
THEN cancel_order

Esto permite:

Versioning
Testing
Auditing
Simulation
Explainability

sin modificar código de aplicación.

11. Rule Language

El lenguaje de reglas debe ser:

Declarative
Deterministic
Bounded
Typed
Auditable
Versionable
Testable

No debe permitir ejecución arbitraria de código.

12. Rule Expressions

Las expresiones pueden soportar:

Equality
Inequality
Comparison
Boolean Logic
Set Membership
Ranges
Null Checks
String Operations
Date Operations
Collection Operations

Ejemplo:

customer.country IN ["ES", "PT", "FR"]
13. Rule Conditions

Una regla puede tener múltiples condiciones:

Condition A
AND
Condition B
AND
Condition C

o:

Condition A
OR
Condition B

La semántica debe ser explícita.

14. Condition Tree

Internamente:

AND
├── customer.active == true
├── order.total > 1000
└── order.currency == "EUR"

Esto permite evaluación estructurada.

15. Rule Actions

Una regla puede producir:

ALLOW
DENY
REQUIRE_APPROVAL
SET_VALUE
EMIT_EVENT
CREATE_TASK
ROUTE
NOTIFY
BLOCK

Las acciones deben estar limitadas por el contrato del engine.

16. Rules vs Actions

La regla decide:

IF X
THEN Y

pero el engine no necesariamente ejecuta directamente toda la operación.

Preferible:

Rule
 ↓
Decision / Command
 ↓
Application Service
 ↓
Domain Operation

Esto mantiene separación de responsabilidades.

17. Rule Facts

Los facts representan información disponible:

Order
Customer
Account
Transaction
Tenant
RiskScore
Time
Environment

Ejemplo:

facts.order.total
facts.customer.segment
18. Fact Sources

Los facts pueden proceder de:

Request Context
Domain State
Application Context
Configuration
External Services
Computed Values
19. Fact Immutability

Durante una evaluación, los facts deberían ser inmutables salvo que el modelo explícitamente soporte stateful rules.

Preferible:

Facts
 ↓
Evaluation
 ↓
Result

en lugar de:

Rule A modifies Facts
Rule B modifies Facts
Rule C modifies Facts

sin control.

20. Stateful Rules

Si EVOXA requiere reglas stateful:

Event A
+
Event B
+
Time Window
 ↓
Rule

debe existir un modelo separado y explícito.

No debe mezclarse accidentalmente con reglas stateless.

21. Stateless Evaluation

El modelo principal debe ser:

Rule Set
+
Facts
 ↓
Decision

Esto facilita:

Caching
Replay
Testing
Determinism
Horizontal Scaling
22. Stateful Evaluation

Cuando sea necesario:

State
+
Events
+
Rules
 ↓
New State / Action

Debe depender de infraestructura explícita de state management.

23. Rule Matching

El engine debe determinar:

Which rules match?

Ejemplo:

Rules:
R1 → total > 1000
R2 → customer.vip
R3 → country == ES

Facts:
total = 5000
vip = true
country = ES

Matched:
R1
R2
R3
24. Rule Evaluation Pipeline
Input
 ↓
Context Validation
 ↓
Fact Resolution
 ↓
Rule Selection
 ↓
Condition Evaluation
 ↓
Match Collection
 ↓
Priority Resolution
 ↓
Action Resolution
 ↓
Decision
25. Rule Selection

No todas las reglas deben evaluarse siempre.

Puede utilizarse:

Domain
Type
Tags
Scope
Priority
Fact Index

para reducir trabajo.

26. Rule Indexing

Ejemplo:

event_type = "refund"

permite seleccionar únicamente reglas relacionadas con refund.

Esto evita:

Evaluate 10,000 rules

cuando sólo:

Evaluate 20 relevant rules
27. Rule Priority

Las reglas pueden tener:

priority = 100
priority = 50
priority = 10

La semántica debe estar definida.

28. Priority Semantics

Una opción:

Higher Priority
      ↓
Evaluated First

Otra:

Higher Priority
      ↓
Overrides Lower Priority

El engine debe diferenciar ambos conceptos si ambos existen.

29. Rule Conflict

Puede ocurrir:

Rule A → ALLOW
Rule B → DENY

El engine necesita una estrategia:

DENY_OVERRIDES
ALLOW_OVERRIDES
FIRST_MATCH
LAST_MATCH
PRIORITY
ALL_MATCH
30. Default Behavior

Cuando ninguna regla coincide:

NO_MATCH

debe existir una decisión predeterminada:

DEFAULT_ALLOW
DEFAULT_DENY
DEFAULT_NOOP
DEFAULT_ACTION

según el rule set.

31. No-Rule Case

Nunca debe quedar implícito qué ocurre cuando:

0 rules matched

La política debe declararlo.

32. Rule Sets

Las reglas relacionadas deben agruparse:

Billing Rules
   ├── Refund
   ├── Payment
   └── Invoice

El runtime evalúa un RuleSet.

33. Rule Set

Un RuleSet contiene:

id
key
version
rules
evaluation_strategy
default_behavior
34. Rule Set Version

Debe ser posible tener:

billing:v10
billing:v11

y evaluar una versión concreta.

35. Rule Dependencies

Una regla puede depender de:

Fact
Rule
RuleSet
Configuration
Policy
Feature

Las dependencias deben declararse.

36. Circular Dependencies

Debe rechazarse:

Rule A
 ↓
Rule B
 ↓
Rule C
 ↓
Rule A

antes de activar el RuleSet.

37. Rule Composition

Las reglas pueden componerse:

RuleSet A
+
RuleSet B

pero la composición debe mantener:

Deterministic Ordering
Explicit Precedence
Conflict Resolution
38. Rule Hierarchy

Puede existir:

Global Rules
    ↓
Domain Rules
    ↓
Tenant Rules
    ↓
Resource Rules

siempre que el modelo de precedencia sea explícito.

39. Multi-Tenant Rules

En multi-tenancy:

Global Rules
      ↓
Tenant Rules
      ↓
Effective RuleSet

Debe garantizarse:

Tenant A
   ✕
Tenant B Facts
40. Tenant Isolation

La resolución del RuleSet debe incorporar el tenant cuando la regla tenga scope tenant.

Nunca debe depender únicamente de filtros aplicados posteriormente.

41. Rule Context

El contexto puede contener:

actor
tenant
resource
action
environment
region
request
timestamp
facts
42. Context Schema

El engine debe validar el contexto antes de evaluar:

Context
 ↓
Schema Validation
 ↓
Rule Evaluation
43. Context Versioning

Los schemas pueden evolucionar:

Context v1
Context v2
Context v3

El RuleSet debe declarar qué versión acepta.

44. Rule Inputs

Cada RuleSet debería definir explícitamente:

Required Facts
Optional Facts
Types
Defaults
Constraints
45. Missing Facts

Si falta un fact requerido:

customer.risk_score

no debe convertirse silenciosamente en:

0

salvo que el schema lo defina.

Debe existir:

MISSING_INPUT

o comportamiento equivalente.

46. Type Safety

Debe evitarse:

"100" > 50

si el lenguaje no define conversión explícita.

Los tipos deben ser conocidos.

47. Null Semantics

Debe definirse claramente:

null == null
null > 10
null IN [...]

La semántica no puede quedar al runtime del lenguaje anfitrión.

48. Time Semantics

Las reglas temporales deben utilizar:

Explicit Timezone
Explicit Clock
Explicit Timestamp

No:

system local time

implícito.

49. Deterministic Time

Para testing:

EvaluationContext
    timestamp = fixed_timestamp

permite reproducir resultados.

50. External Facts

Si una regla requiere:

fraud_score

obtenido de otro servicio:

Rules Engine
      ↓
Fact Provider
      ↓
Risk Service

La dependencia debe estar explícitamente controlada.

51. External Fact Timeout

Cada proveedor externo debe tener:

Timeout
Retry Policy
Fallback
Circuit Breaker

cuando corresponda.

52. Rule Engine Isolation

El Rules Engine no debería convertirse en un cliente libre de todos los servicios de EVOXA.

Preferible:

Fact Provider Interface

con dependencias controladas.

53. Rule Evaluation API

Conceptualmente:

evaluate(
    ruleset,
    context
)

Resultado:

matches
decisions
actions
metadata
54. Evaluation Result

Ejemplo:

{
  ruleset: "billing.refund",
  version: 12,
  matched_rules: [
    "refund.high_value"
  ],
  decision: "REQUIRE_APPROVAL"
}
55. Rule Trace

Para debugging:

Rule A → evaluated → false
Rule B → evaluated → true
Rule C → skipped

Esto permite entender la evaluación.

56. Trace Levels

No todos los entornos necesitan el mismo detalle:

NONE
ERROR
SUMMARY
DETAILED
DEBUG

Production debería evitar traces excesivamente voluminosos.

57. Rule Explainability

Una evaluación debe poder responder:

Why did this rule match?
Why did another rule not match?
Which facts were used?
Which version was active?
58. Rule Audit

Cambios administrativos:

Created
Modified
Approved
Activated
Disabled
Deleted

deben auditarse.

59. Evaluation Audit

Para operaciones críticas:

RuleSet
Version
Matched Rule
Decision
Timestamp
Tenant

puede registrarse.

60. Sensitive Data

El trace nunca debe exponer innecesariamente:

Secrets
Tokens
Credentials
Sensitive Payloads
61. Rule Validation

Antes de activarse:

Syntax
 ↓
Schema
 ↓
Semantic
 ↓
Dependency
 ↓
Conflict
 ↓
Security
 ↓
Test
62. Syntax Validation

Debe detectar:

Invalid operators
Malformed expressions
Unknown functions
Invalid action syntax
63. Semantic Validation

Debe detectar:

Impossible conditions
Unknown facts
Invalid types
Unsupported actions
64. Dead Rules

Debe detectarse potencialmente:

Rule A
condition = x > 10

Rule B
condition = x > 100

si la prioridad hace que B nunca pueda ejecutarse.

65. Unreachable Rules

Una regla que nunca puede coincidir debe generar:

WARNING

o impedir activación según criticidad.

66. Contradictory Rules

Ejemplo:

IF customer.vip
THEN discount = 10%

IF customer.vip
THEN discount = 20%

El engine debe requerir:

Priority
Precedence
Aggregation

para resolverlo.

67. Rule Testing

Cada RuleSet crítico debe tener:

Positive Cases
Negative Cases
Boundary Cases
Conflict Cases
Missing Input Cases
Failure Cases
68. Golden Tests

Debe ser posible definir:

Input
Expected Output

como fixtures.

Ejemplo:

amount = 1000
expected = ALLOW
69. Regression Testing

Cuando se cambia una regla:

Old RuleSet
      ↓
Historical Fixtures
      ↓
New RuleSet
      ↓
Diff
70. Rule Simulation

Debe poder ejecutarse:

Draft RuleSet
+
Production-like Context

sin modificar producción.

71. Dry Run

Modo:

DRY_RUN

debe producir:

would_match
would_not_match
would_execute

sin aplicar acciones reales.

72. Shadow Rules

Puede ejecutarse:

Active RuleSet
+
Shadow RuleSet

para comparar:

Decision A
vs
Decision B
73. Rule Rollout

Puede hacerse progresivamente:

Tenant subset
Region subset
Service subset

cuando el caso lo permita.

74. Rule Rollback

Debe ser posible volver a:

Last Known Good RuleSet

rápidamente.

75. Rule Snapshot

El runtime puede cargar:

RuleSet v42

en memoria.

Esto permite:

Fast Evaluation
Deterministic Behavior
Reduced Network Dependency
76. Rule Distribution
Rule Registry
     ↓
Distribution
     ↓
Runtime Nodes
     ↓
Rule Snapshot
77. Rule Propagation

Debe medirse:

rule_propagation_latency

desde:

Activation

hasta:

Runtime Adoption
78. Rule Cache

E17 puede almacenar:

Compiled RuleSet
Rule Metadata
Evaluation Artifacts

E21 define cuándo y cómo se utilizan semánticamente.

79. Rule Compilation

Pipeline:

Rule Definition
      ↓
Parse
      ↓
Validate
      ↓
Compile
      ↓
Optimized Representation
      ↓
Evaluate
80. Compilation Cache

Puede reutilizarse:

RuleSet v42
Compiled Artifact

hasta que cambie la versión.

81. Rule Performance

El engine debe optimizar:

Evaluation Latency
Memory
Rule Matching
Context Processing
Compilation
82. Performance Targets

Los límites concretos deben definirse por dominio, pero deben existir objetivos para:

P50
P95
P99

de evaluación.

83. Bounded Evaluation

Una regla no debe poder provocar:

Infinite Loop
Unbounded Recursion
Unbounded Collection Scan
84. Rule Complexity Limits

Puede establecerse:

max_expression_depth
max_rules_per_set
max_actions_per_rule
max_evaluation_steps
max_context_size
85. Rule Engine Failure

Si el engine falla:

Evaluation Error

debe existir un comportamiento definido por RuleSet:

FAIL_CLOSED
FAIL_OPEN
NOOP
REQUIRE_APPROVAL
86. Engine Availability

El runtime no debería depender obligatoriamente de una llamada remota por cada evaluación.

Preferible:

Central Registry
      ↓
Local Rule Snapshot
      ↓
Local Evaluation
87. Rule Control Plane vs Data Plane
Control Plane
Definition
Validation
Approval
Versioning
Distribution
Lifecycle
Data Plane
Load
Match
Evaluate
Return
Execute
88. Rule Engine Security

Debe proteger contra:

Rule Injection
Unauthorized Rule Changes
Tampered Rule Artifacts
Privilege Escalation
Unsafe Actions
89. Rule Signing

Para reglas críticas:

Rule Artifact
   ↓
Signature
   ↓
Verification
   ↓
Activation
90. Rule Authorization

Sólo usuarios o sistemas autorizados deben poder:

Create
Modify
Approve
Activate
Rollback
91. Four-Eyes Approval

Para reglas de alto impacto:

Author
 +
Approver

puede ser obligatorio.

92. Rule Ownership

Cada RuleSet debe tener:

Owner
Domain
Responsible Team
Lifecycle
93. Domain Ownership

Ejemplo:

billing.refund
    → Billing Domain

identity.session
    → Identity Domain

risk.transaction
    → Risk Domain
94. Domain Boundary

El Rules Engine no debe absorber el dominio.

Debe evaluar reglas sobre conceptos del dominio, no convertirse en el lugar donde reside toda la lógica de negocio.

95. Rule vs Domain Logic
Rule
IF total > threshold
THEN require_approval
Domain Logic
RefundService.calculateRefund()

La segunda representa comportamiento semántico del dominio.

96. Rule vs Workflow

Rule:

IF condition
THEN require approval

Workflow:

Create approval
 ↓
Assign reviewer
 ↓
Wait
 ↓
Approve
 ↓
Continue

El Rules Engine decide.

El Workflow Engine ejecuta el proceso.

97. Rule vs Policy

La relación recomendada:

Policy
   ↓
references / uses
RuleSet
   ↓
evaluates
Rules

Por ejemplo:

E20 Runtime Policy
        ↓
billing.refund.approval
        ↓
E21 RuleSet
        ↓
refund.high_value
98. Rule vs Feature Flag

Feature Flag:

feature.enabled

Rule:

customer.segment == enterprise

La combinación puede producir:

Feature Enabled
+
Rule Match
=
Feature Available
99. Rule vs Configuration

Configuration:

refund_threshold = 10000

Rule:

refund_amount > refund_threshold
    → require approval
100. Rule Events

El engine puede producir:

RuleMatched
RuleExecuted
RuleFailed
RuleSetActivated
RuleSetRolledBack

E13 puede procesar estos eventos cuando corresponda.

101. Rule Messaging

E12 puede transportar:

RuleSetUpdated
RuleSetActivated
RuleSetInvalidated

para distribución y coordinación.

102. Rule Jobs

E15 puede ejecutar:

Rule Validation
Rule Simulation
Rule Regression
Rule Compilation
Rule Drift Detection
103. Rule Scheduling

E16 puede controlar:

Rule activation at timestamp
Rule expiration
Temporary rule
104. Rule Expiration

Una regla temporal puede definir:

valid_from
valid_until

Ejemplo:

Black Friday Rule
valid_until = 2026-11-30T23:59:59Z
105. Temporal Rules

Las reglas temporales deben utilizar timestamps explícitos y timezone definido.

106. Rule Drift

Debe detectarse:

Desired RuleSet v50
        ≠
Observed RuleSet v49
107. Rule Reconciliation

Un reconciler puede restaurar:

Observed
    ↓
Desired

cuando exista divergencia.

108. Rule Observability

Métricas:

rule_evaluations_total
rule_matches_total
rule_execution_total
rule_errors_total
rule_evaluation_latency
ruleset_version
ruleset_age
rule_propagation_latency
109. Rule Match Metrics

Por regla:

matches
non_matches
errors
execution_count

Esto permite detectar reglas:

Never Used
Overused
Unexpectedly Triggered
110. Rule Anomaly Detection

Ejemplo:

Before:
refund.high_value → 1.2%

After:
refund.high_value → 35%

Debe investigarse antes de asumir que el cambio es correcto.

111. Rule Evaluation Tracing

Una trace distribuida puede incluir:

ruleset
ruleset_version
matched_rules
decision
evaluation_latency

sin incluir datos sensibles.

112. Rule Replay

Para debugging:

Historical Context
+
Historical RuleSet
 ↓
Replay

Esto permite reproducir una decisión.

113. Rule Time Travel

Debe poder evaluarse:

RuleSet v12
+
Historical Context
+
Historical Timestamp

para auditoría.

114. Rule Determinism

Para:

same RuleSet version
+
same facts
+
same timestamp

debe producirse el mismo resultado.

115. External Side Effects

La evaluación debe preferentemente ser libre de side effects.

En lugar de:

Rule
 ↓
Direct DB mutation

preferible:

Rule
 ↓
Action / Command
 ↓
Application Service
 ↓
Mutation
116. Idempotency

Si una acción puede ejecutarse más de una vez:

Rule Evaluation
 ↓
Action

la capa ejecutora debe garantizar idempotencia cuando corresponda.

117. Action Registry

Las acciones permitidas deben estar registradas:

require_approval
emit_event
block_transaction
route_to_queue

No deben existir acciones arbitrarias.

118. Action Authorization

Incluso si una regla produce:

delete_resource

la ejecución debe pasar por las fronteras de autorización y dominio apropiadas.

119. Rule Engine API Boundary

El acceso debe estar centralizado:

Application Service
      ↓
Rules Engine Interface

y no mediante acceso directo a internals.

120. Canonical Rule Flow
Request
  ↓
Context
  ↓
Fact Resolution
  ↓
RuleSet Selection
  ↓
Rule Matching
  ↓
Condition Evaluation
  ↓
Conflict Resolution
  ↓
Decision
  ↓
Action / Command
  ↓
Application / Domain
  ↓
Audit + Events
121. Engineering Principles
Principle 1

Rules must be declarative whenever possible.

Principle 2

Rule evaluation must be deterministic.

Principle 3

Rule versions must be immutable.

Principle 4

RuleSets must have explicit default behavior.

Principle 5

Missing facts must never be silently invented.

Principle 6

Rule conflicts must have deterministic resolution.

Principle 7

Rules must not execute arbitrary code.

Principle 8

Rule evaluation must be bounded.

Principle 9

Actions must be explicitly registered.

Principle 10

Rules should produce decisions or commands rather than directly mutating arbitrary state.

Principle 11

Domain semantics remain in the Domain Layer.

Principle 12

Workflow semantics remain in the Workflow Engine.

Principle 13

Policy governance remains in E20.

Principle 14

Configuration remains in E18.

Principle 15

Feature exposure remains in E19.

Principle 16

Rule lifecycle must be auditable.

Principle 17

Critical RuleSets must support simulation and rollback.

Principle 18

Runtime should use immutable rule snapshots.

Principle 19

Tenant isolation must be enforced at rule resolution.

Principle 20

The same input against the same rule version must produce the same result.

122. Definition of Done

E21 queda definido cuando EVOXA dispone de:

✓ Rule Definition
✓ Rule Registry
✓ Rule Schema
✓ Rule Keys
✓ Rule Versioning
✓ Rule Lifecycle
✓ Declarative Rule Language
✓ Rule Expressions
✓ Rule Conditions
✓ Rule Actions
✓ Rule Facts
✓ Fact Sources
✓ Stateless Evaluation
✓ Stateful Evaluation Model
✓ Rule Matching
✓ Rule Selection
✓ Rule Indexing
✓ Rule Priority
✓ Conflict Resolution
✓ Default Behavior
✓ Rule Sets
✓ Rule Set Versioning
✓ Rule Dependencies
✓ Circular Dependency Detection
✓ Rule Composition
✓ Rule Hierarchy
✓ Tenant Rules
✓ Tenant Isolation
✓ Rule Context
✓ Context Schema
✓ Context Versioning
✓ Missing Fact Handling
✓ Type Safety
✓ Null Semantics
✓ Temporal Semantics
✓ External Fact Providers
✓ Fact Timeouts
✓ Evaluation API
✓ Evaluation Result
✓ Rule Trace
✓ Explainability
✓ Rule Audit
✓ Evaluation Audit
✓ Validation
✓ Semantic Validation
✓ Dead Rule Detection
✓ Contradiction Detection
✓ Testing
✓ Golden Tests
✓ Regression Testing
✓ Simulation
✓ Dry Run
✓ Shadow Rules
✓ Progressive Rollout
✓ Rollback
✓ Rule Snapshots
✓ Rule Distribution
✓ Rule Cache
✓ Rule Compilation
✓ Performance Controls
✓ Bounded Evaluation
✓ Complexity Limits
✓ Failure Strategy
✓ Control Plane
✓ Data Plane
✓ Rule Security
✓ Rule Signing
✓ Rule Authorization
✓ Approval Workflow
✓ Rule Ownership
✓ Domain Boundaries
✓ Workflow Boundaries
✓ Policy Integration
✓ Feature Flag Integration
✓ Configuration Integration
✓ Event Integration
✓ Messaging Integration
✓ Job Integration
✓ Scheduling Integration
✓ Rule Expiration
✓ Drift Detection
✓ Reconciliation
✓ Observability
✓ Match Metrics
✓ Anomaly Detection
✓ Distributed Tracing
✓ Replay
✓ Time Travel
✓ Determinism
✓ Side-Effect Isolation
✓ Idempotency
✓ Action Registry
✓ Action Authorization
✓ Canonical Runtime Flow
123. Position in Engineering Specification

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

Y la separación fundamental de estas cuatro capas queda:

┌─────────────────────────────────────────┐
│ E18 — Configuration                     │
│ Parameters / Settings                   │
└────────────────────┬────────────────────┘
                     ↓
┌─────────────────────────────────────────┐
│ E19 — Feature Flags                     │
│ Capability Exposure                     │
└────────────────────┬────────────────────┘
                     ↓
┌─────────────────────────────────────────┐
│ E20 — Runtime Policies                  │
│ Governance / Constraints / Decisions    │
└────────────────────┬────────────────────┘
                     ↓
┌─────────────────────────────────────────┐
│ E21 — Rules Engine                      │
│ Rule Evaluation / Matching / Actions    │
└────────────────────┬────────────────────┘
                     ↓
┌─────────────────────────────────────────┐
│ Domain / Application Runtime            │
│ Business Semantics / Execution          │
└─────────────────────────────────────────┘

La idea clave para el blueprint es:

E20 define qué comportamiento está gobernado por policy; E21 proporciona el motor determinista que evalúa las reglas que materializan ese gobierno.

Siguiente capítulo: E22 — EVOXA Validation Architecture.
