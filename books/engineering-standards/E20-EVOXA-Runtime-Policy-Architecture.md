E20 — EVOXA Runtime Policy Architecture
Engineering Specification

Pertenece a: Engineering Specification
Volumen: volume-09-engineering
Capítulo: E20 — Runtime Policy Architecture
Anterior: E19 — Feature Flag Architecture
Siguiente: E21 — EVOXA Rules Engine Architecture

1. Propósito

E20 define la arquitectura mediante la cual EVOXA aplica políticas durante la ejecución del sistema.

Su responsabilidad es determinar:

qué decisiones deben estar sujetas a policy;
cómo se carga una policy;
cómo se evalúa;
qué contexto utiliza;
cómo se combinan múltiples policies;
cómo se versionan;
cómo se actualizan;
cómo se aplican en runtime;
cómo se auditan;
cómo se observan;
cómo se comportan ante errores;
cómo se integran con autorización, configuración, feature flags y dominios.

La separación fundamental es:

E18 — Configuration
    ↓
¿Cómo está parametrizado el sistema?

E19 — Feature Flags
    ↓
¿Qué capacidad está expuesta?

E20 — Runtime Policies
    ↓
¿Qué reglas deben cumplirse durante la ejecución?

Por tanto:

Runtime Policy es la capa que transforma reglas de gobierno y operación en decisiones aplicables al comportamiento runtime.

2. Objetivos

EVOXA debe soportar:

Policy Definition
Policy Registry
Policy Schema
Policy Versioning
Policy Evaluation
Policy Context
Policy Composition
Policy Precedence
Policy Enforcement
Policy Overrides
Policy Exceptions
Policy Simulation
Policy Validation
Policy Activation
Policy Rollback
Policy Audit
Policy Observability
Policy Caching
Policy Distribution
Policy Lifecycle
Policy Conflict Resolution
Policy Multi-Tenancy
Policy Environment Scope
Policy Runtime Decisions
3. Policy Concept

Una policy representa una restricción o condición:

Context
   ↓
Policy
   ↓
Decision

Ejemplo:

User
+
Action
+
Resource
+
Context
   ↓
Policy
   ↓
ALLOW / DENY

Pero las policies pueden producir también:

ALLOW
DENY
REQUIRE_APPROVAL
REQUIRE_STEP_UP
LIMIT
REDACT
ROUTE

según el dominio.

4. Policy vs Authorization

Authorization responde:

¿Puede este actor realizar esta acción?

Runtime Policy puede responder:

¿Bajo qué condiciones puede realizarse esta acción?

Por ejemplo:

Authorization
    ↓
User may refund

Runtime Policy
    ↓
Refund > $10,000
    ↓
Requires approval

Por tanto:

E05 — Authorization
        ↓
E20 — Runtime Policy

pueden colaborar sin ser equivalentes.

5. Policy vs Configuration

Configuration:

timeout = 30
max_retry = 3

Policy:

IF risk_score > threshold
THEN require additional verification

Configuration proporciona parámetros.

Policy proporciona decisiones o restricciones.

6. Policy vs Feature Flag

Feature Flag:

new_checkout = ON

Policy:

new_checkout
    allowed only for
    eligible tenants

Por tanto:

E19
Feature Exposure
     ↓
E20
Runtime Constraint
7. Policy Architecture
                         Policy Control Plane
                                  │
              ┌───────────────────┼───────────────────┐
              ▼                   ▼                   ▼
          Registry             Versions            Schemas
              │                   │                   │
              └───────────────────┼───────────────────┘
                                  ▼
                           Policy Distribution
                                  │
                                  ▼
                         Runtime Policy Engine
                                  │
                         ┌────────┴────────┐
                         ▼                 ▼
                    Evaluation         Enforcement
                         │                 │
                         └────────┬────────┘
                                  ▼
                              Runtime
8. Policy Definition

Una policy debe tener como mínimo:

id
key
name
description
type
scope
version
status
owner
rules
default_decision
9. Policy Key

Las keys deben ser:

Stable
Unique
Readable
Namespaced
Version-independent

Ejemplo:

billing.refund.approval
identity.password.requirements
orders.cancellation.policy
10. Policy Ownership

Cada policy debe tener:

Owner
Domain
Responsible Team
Lifecycle
Classification

Esto evita policies huérfanas.

11. Policy Types

EVOXA puede soportar:

Authorization Policy
Security Policy
Operational Policy
Compliance Policy
Data Policy
Resource Policy
Rate Policy
Approval Policy
Routing Policy
Transformation Policy
Tenant Policy
12. Policy Decision Model

La salida de evaluación puede ser:

ALLOW
DENY
CONDITIONAL
REQUIRE_APPROVAL
REQUIRE_STEP_UP
LIMITED

No todas las policies necesitan todos los resultados.

13. Policy Context

La evaluación puede recibir:

Actor
Action
Resource
Tenant
Environment
Region
Time
Request
Application
Risk
Attributes
14. Context Minimization

Sólo deben proporcionarse los atributos necesarios.

Esto reduce:

Latency
Privacy Risk
Complexity
Data Exposure
15. Policy Input

Conceptualmente:

Policy
+
Context
+
Action
+
Resource
      ↓
Evaluation
16. Policy Output

Ejemplo:

decision = REQUIRE_APPROVAL
reason = "amount_threshold"
policy_version = 12

La respuesta debe poder explicar la decisión sin exponer información sensible innecesaria.

17. Explainability

Una policy debe poder responder:

Which policy?
Which version?
Which rule?
Which context?
Which decision?
Why?

Esto es fundamental para debugging y auditoría.

18. Policy Rule

Una policy puede componerse de reglas:

Rule 1
Rule 2
Rule 3
Default

Cada regla debe tener prioridad determinista.

19. Rule Precedence

Ejemplo:

Explicit Deny
      ↓
Security Rule
      ↓
Tenant Restriction
      ↓
Domain Rule
      ↓
Default

La precedencia exacta debe ser definida por el tipo de policy.

20. Explicit Deny

Para policies de acceso:

DENY

debe poder prevalecer sobre:

ALLOW

cuando el modelo de seguridad así lo requiera.

21. Policy Composition

Varias policies pueden aplicarse:

Authorization Policy
        +
Security Policy
        +
Tenant Policy
        +
Domain Policy

produciendo una decisión final.

22. Policy Combination

La combinación puede utilizar:

AND
OR
DENY-OVERRIDES
ALLOW-OVERRIDES
FIRST-MATCH
PRIORITY

La estrategia debe ser explícita.

23. Conflict Resolution

Si dos policies producen:

ALLOW
DENY

el motor debe resolverlo de manera determinista.

Nunca:

undefined
24. Policy Hierarchy

Las policies pueden organizarse:

Global
  ↓
Environment
  ↓
Region
  ↓
Tenant
  ↓
Domain
  ↓
Resource

pero debe evitarse una jerarquía excesivamente profunda.

25. Policy Scope

Una policy puede tener scope:

Global
Environment
Region
Service
Module
Domain
Tenant
Resource
26. Tenant Policies

En un entorno multi-tenant:

Global Policy
      ↓
Tenant Policy
      ↓
Effective Policy

La configuración de un tenant nunca debe modificar accidentalmente la policy de otro.

27. Tenant Isolation

La evaluación debe mantener:

Tenant A Context
        ✕
Tenant B Policy State

La resolución de policies debe ser tenant-aware cuando corresponda.

28. Environment Policies

Puede existir:

Development
Staging
Production

con políticas distintas.

Sin embargo, production debe tener reglas explícitas y no depender de defaults de desarrollo.

29. Region Policies

Puede ser necesario aplicar reglas regionales:

Region
  ↓
Applicable Policy

especialmente para:

Compliance
Data Residency
Operational Constraints
30. Runtime Enforcement

La policy puede aplicarse:

Before Operation
During Operation
After Operation

según el caso.

31. Pre-Action Enforcement

Ejemplo:

Request
 ↓
Policy Evaluation
 ↓
ALLOW / DENY
 ↓
Operation

Adecuado para:

Authorization
Validation
Resource Limits
32. In-Flight Enforcement

Durante una operación:

Long-Running Process
       ↓
Policy Check
       ↓
Continue / Stop

Necesario cuando policy puede cambiar durante el ciclo de ejecución.

33. Post-Action Enforcement

Después:

Operation
 ↓
Policy Validation
 ↓
Audit / Remediation

Puede ser útil para:

Compliance
Data Governance
Audit
34. Policy Enforcement Point

Debe existir un punto claro donde se aplica una policy:

Application
    ↓
Policy Enforcement Point
    ↓
Policy Engine
    ↓
Decision
35. Policy Decision Point

Separación conceptual:

PEP
Policy Enforcement Point
      │
      ▼
PDP
Policy Decision Point

El PEP aplica la decisión.

El PDP produce la decisión.

36. Policy Administration Point

También puede existir:

PAP
Policy Administration Point

responsable de:

Create
Modify
Version
Approve
Activate
37. Canonical Model
PAP
 │
 ▼
Policy Registry
 │
 ▼
Policy Distribution
 │
 ▼
PDP
 │
 ▼
Decision
 │
 ▼
PEP
 │
 ▼
Runtime Action
38. Policy Control Plane

Debe manejar:

Policy Registry
Policy Versions
Policy Schemas
Policy Lifecycle
Approval
Audit
Distribution
39. Policy Data Plane

Debe encargarse de:

Load Policy
Evaluate Policy
Return Decision
Enforce Decision

El data plane debe estar optimizado para runtime.

40. Local Evaluation

Preferentemente:

Policy Control Plane
        ↓
Policy Snapshot
        ↓
Local PDP
        ↓
Application

Esto evita una dependencia de red en cada operación.

41. Remote Evaluation

También puede existir:

Application
    ↓
Policy Service
    ↓
Decision

pero añade:

Latency
Network Dependency
Availability Dependency
42. Hybrid Policy Evaluation

Una arquitectura robusta puede utilizar:

Central Policy Management
          ↓
Distributed Policy Snapshots
          ↓
Local Evaluation
43. Policy Snapshot

Cada runtime puede mantener:

Policy Snapshot v42

Esto permite:

Deterministic Evaluation
Fast Access
Offline Continuity
Rollback
Debugging
44. Snapshot Consistency

Si varias policies deben cambiar juntas:

Policy A v10
Policy B v10
Policy C v10

puede requerirse una activación atómica.

45. Policy Bundles

Policies relacionadas pueden distribuirse como bundle:

Security Bundle
   ├── Authentication
   ├── Session
   └── Password

Billing Bundle
   ├── Refund
   ├── Invoice
   └── Payment
46. Policy Dependencies

Una policy puede depender de otra:

Policy B
   requires
Policy A

Las dependencias deben validarse.

47. Circular Dependencies

Debe rechazarse:

A → B
B → C
C → A
48. Policy Validation

Antes de activarse:

Parse
 ↓
Schema Validation
 ↓
Semantic Validation
 ↓
Dependency Validation
 ↓
Conflict Detection
 ↓
Security Validation
 ↓
Activate
49. Syntax Validation

Debe comprobar:

Structure
Types
Required Fields
Expression Syntax
50. Semantic Validation

Debe detectar:

Contradictory Rules
Impossible Conditions
Invalid Thresholds
Unsupported Actions
51. Conflict Detection

El sistema debería detectar conflictos conocidos antes de activar:

ALLOW rule
+
DENY rule

cuando ambos puedan aplicarse al mismo contexto.

52. Policy Simulation

Una capacidad importante:

Policy Draft
      ↓
Simulation
      ↓
Sample Contexts
      ↓
Expected Decisions

Esto permite probar una policy sin activarla.

53. Dry Run

Una policy puede evaluarse en modo:

DRY_RUN

produciendo:

would_allow
would_deny
would_require_approval

sin modificar el comportamiento real.

54. Shadow Policy

Una policy nueva puede ejecutarse en paralelo:

Active Policy
      ↓
Real Decision

Shadow Policy
      ↓
Shadow Decision

y compararse.

55. Policy Comparison

Debe poder medirse:

Old Decision
vs
New Decision

antes de un rollout completo.

56. Progressive Policy Rollout

Puede desplegarse:

5%
 ↓
25%
 ↓
50%
 ↓
100%

por:

Tenant
Region
Environment
Service

si el tipo de policy lo permite.

57. Policy Activation

Estados posibles:

DRAFT
VALIDATED
APPROVED
STAGED
ACTIVE
SUPERSEDED
ROLLED_BACK
DEPRECATED
58. Policy Lifecycle
CREATE
  ↓
VALIDATE
  ↓
SIMULATE
  ↓
APPROVE
  ↓
STAGE
  ↓
ACTIVATE
  ↓
OBSERVE
  ↓
SUPERSEDE
  ↓
ARCHIVE
59. Policy Versioning

Cada cambio importante genera:

v1
 ↓
v2
 ↓
v3

Debe conservarse el historial.

60. Policy Diff

Debe ser posible comparar:

v12
vs
v13

mostrando:

Added Rules
Removed Rules
Changed Conditions
Changed Decisions
Changed Scope
61. Policy Rollback

Debe ser posible:

v13
 ↓
Rollback
 ↓
v12

especialmente cuando una nueva policy produce decisiones incorrectas.

62. Last Known Good

Si una nueva policy es inválida:

v12 = valid
v13 = invalid

el runtime debe conservar:

Last Known Good = v12
63. Policy Availability

Si el policy control plane está caído:

Control Plane Failure
        ↓
Last Known Good Snapshot
        ↓
Continue

cuando la seguridad del caso lo permita.

64. Fail-Open vs Fail-Closed

La estrategia debe definirse por policy.

Fail-Closed
Policy unavailable
       ↓
DENY

adecuado para controles de seguridad críticos.

Fail-Open
Policy unavailable
       ↓
ALLOW / continue

puede ser apropiado para determinadas policies operacionales.

La elección nunca debe quedar implícita.

65. Policy Freshness

Debe existir una noción de:

policy_snapshot_age

Para policies críticas puede existir:

max_allowed_staleness
66. Policy Propagation

Cuando cambia una policy:

Policy Update
      ↓
Distribution
      ↓
Runtime
      ↓
New Snapshot

Debe medirse:

propagation_latency
67. Policy Event

E13 puede distribuir:

PolicyCreated
PolicyUpdated
PolicyActivated
PolicyRolledBack
PolicyDeprecated

El evento debe identificar la versión.

68. Policy Messaging

E12 puede transportar eventos de distribución:

PolicyChanged

cuando la infraestructura de mensajería sea utilizada.

69. Policy Cache

E17 puede cachear:

Policy Snapshot
Compiled Policy
Evaluation Metadata

Pero:

E20 mantiene la semántica de policy; E17 proporciona el mecanismo de cache.

70. Compiled Policies

Si las expresiones son costosas:

Policy Definition
      ↓
Compile
      ↓
Compiled Policy
      ↓
Cache
      ↓
Evaluate

Esto reduce latencia en runtime.

71. Policy Evaluation Performance

La evaluación debe ser:

Fast
Predictable
Bounded
Non-Blocking
Observable
72. Policy Evaluation API

Conceptualmente:

evaluate(
    policy_key,
    context
)

Resultado:

decision
policy_version
rule
reason
73. Typed Decisions

En lugar de strings arbitrarios:

ALLOW
DENY
REQUIRE_APPROVAL

deben existir tipos explícitos.

74. Decision Metadata

Puede incluir:

policy_id
policy_version
rule_id
decision
reason
timestamp
75. Decision Explainability

Ejemplo:

Policy:
billing.refund.approval

Version:
v12

Rule:
amount-over-threshold

Decision:
REQUIRE_APPROVAL

Esto facilita soporte y debugging.

76. Decision Logging

No todas las decisiones necesitan persistirse.

Debe utilizarse:

Metrics
Sampling
Tracing
Audit Events

según criticidad.

77. Policy Audit

Cambios deben registrar:

Who
What
When
Previous Version
New Version
Reason
Approval
78. Runtime Decision Audit

Para operaciones sensibles:

Actor
Action
Resource
Policy
Decision
Timestamp

puede formar parte del audit trail.

79. Sensitive Context

El audit no debe almacenar innecesariamente:

Passwords
Tokens
Secrets
Sensitive Payloads
80. Policy Observability

Métricas:

policy_evaluations_total
policy_evaluation_errors
policy_evaluation_latency
policy_denials_total
policy_allows_total
policy_snapshot_version
policy_snapshot_age
policy_propagation_latency
81. Policy Decision Metrics

Por policy:

ALLOW count
DENY count
CONDITIONAL count
ERROR count

Esto ayuda a detectar cambios inesperados.

82. Policy Anomaly Detection

Puede observarse:

DENY rate

antes y después de una actualización.

Ejemplo:

v12 → 2% denied
v13 → 48% denied

Esto puede indicar una regresión.

83. Automatic Safety Controls

Una futura integración puede permitir:

Policy Activation
      ↓
Monitor
      ↓
Anomaly
      ↓
Pause / Rollback

Debe estar gobernada por políticas explícitas.

84. Policy and Scheduling

E16 puede programar:

Activate Policy v13
Deactivate Policy v13

en un instante determinado.

85. Policy and Jobs

E15 puede ejecutar:

Policy Validation Job
Policy Simulation Job
Policy Drift Detection
Policy Cleanup
86. Policy and Workflow

E14 puede orquestar:

Draft
 ↓
Validate
 ↓
Simulate
 ↓
Approve
 ↓
Stage
 ↓
Activate
 ↓
Observe
87. Policy and Configuration

E18 proporciona parámetros:

refund_limit = 10000

E20 puede utilizar ese valor:

IF refund_amount > refund_limit
THEN require approval

Por tanto:

E18 → Policy Parameters
E20 → Policy Decision
88. Policy and Feature Flags

E19:

Feature Enabled?

E20:

Feature Allowed Under These Conditions?

Ejemplo:

Feature Flag = ON
       ↓
Policy = allowed for tenant
       ↓
Action
89. Policy and Authorization

Flujo:

Request
  ↓
Authentication
  ↓
Authorization
  ↓
Runtime Policy
  ↓
Business Operation

No necesariamente todas las operaciones requieren el mismo orden, pero el sistema debe definirlo explícitamente.

90. Policy and Domain Services

Los Domain Services pueden invocar policies:

Domain Service
      ↓
Policy Evaluation
      ↓
Decision
      ↓
Domain Action

La policy no debe contener arbitrariamente toda la lógica de dominio.

91. Policy Boundary

Una policy debe responder:

¿Qué condición gobierna este comportamiento?

El dominio debe responder:

¿Qué significa esta operación?

La separación evita convertir el policy engine en un dominio paralelo.

92. Policy and Repository

Los repositories no deberían normalmente decidir policies de negocio complejas.

Preferible:

Application / Domain Layer
       ↓
Policy Evaluation
       ↓
Repository
93. Policy Enforcement Location

La policy debe aplicarse lo más cerca posible del boundary relevante:

API Boundary
Service Boundary
Domain Boundary
Resource Boundary

según la responsabilidad.

94. Defense in Depth

Una operación crítica puede tener:

API Policy
 +
Authorization
 +
Domain Policy
 +
Resource Policy

pero deben evitarse duplicaciones contradictorias.

95. Policy Duplication

No debería existir:

Controller:
  amount > 10000

Service:
  amount > 10000

Policy:
  amount > 10000

sin una razón clara.

La policy debe convertirse en la fuente normativa cuando corresponda.

96. Policy Source of Truth

Cada regla normativa debe tener una fuente autoritativa.

Policy Registry
      ↓
Canonical Policy

El código no debería contener copias divergentes de esa misma regla.

97. Policy Drift

Puede existir:

Desired Policy
        ≠
Runtime Policy

Debe detectarse.

98. Policy Reconciliation

Un reconciler puede verificar:

Desired Version
        ↓
Observed Version

y corregir:

Runtime ≠ Desired
99. Policy Integrity

Policies críticas pueden requerir:

Signature
Checksum
Version Verification
Source Verification

antes de activarse.

100. Policy Security

Debe protegerse contra:

Unauthorized Modification
Policy Injection
Tampering
Privilege Escalation
Unsafe Rollback
101. Policy Injection

Las reglas recibidas externamente nunca deben ejecutarse como código arbitrario.

Preferible:

Structured Policy Language
+
Validated Expression Engine

en lugar de:

eval(user_input)
102. Policy Language

El lenguaje de policies debe ser:

Declarative
Bounded
Deterministic
Auditable
Testable
103. Policy Runtime Safety

Una policy no debería poder:

Execute arbitrary code
Access unrestricted filesystem
Perform uncontrolled network calls
Mutate arbitrary state

La evaluación debe estar limitada.

104. Policy Timeout

El motor debe tener límites:

max_evaluation_time

para evitar que una policy bloquee el runtime.

105. Policy Resource Limits

También:

max_rules
max_expression_depth
max_context_size
max_evaluation_steps

cuando sea necesario.

106. Policy Determinism

Con:

same policy version
+
same input context

debe producirse:

same decision

salvo policies explícitamente dependientes de tiempo o fuentes externas controladas.

107. External Data

Una policy puede necesitar información externa:

Risk Score
Account State
Resource State

pero las dependencias externas deben estar explícitamente modeladas.

108. External Dependency Failure

Si una dependencia falla:

Risk Service unavailable

la policy debe tener:

Defined fallback

por ejemplo:

DENY
REQUIRE_APPROVAL
DEGRADE

según criticidad.

109. Policy Evaluation Context Version

El contexto también puede versionarse:

Context Schema v3
Policy v12

Esto ayuda a evitar incompatibilidades.

110. Backward Compatibility

Una nueva versión de policy debe ser compatible con los contextos soportados o declarar explícitamente el cambio.

111. Policy Testing

Cada policy crítica debe tener:

Unit Tests
Scenario Tests
Negative Tests
Boundary Tests
Regression Tests
112. Policy Test Cases

Ejemplo:

amount = 9999
→ ALLOW

amount = 10000
→ REQUIRE_APPROVAL

amount = 10001
→ REQUIRE_APPROVAL
113. Boundary Testing

Los thresholds deben probarse explícitamente:

N-1
N
N+1

para evitar errores de comparación.

114. Policy Regression Testing

Antes de activar una nueva versión:

Historical Inputs
      ↓
Old Policy
      ↓
New Policy
      ↓
Compare
115. Policy Coverage

Debe conocerse qué reglas tienen:

Covered Scenarios
Uncovered Scenarios
116. Policy Simulation Environment

Debe existir un entorno donde puedan probarse:

Policy
+
Context

sin afectar producción.

117. Policy Replay

Para incidentes:

Historical Request
      ↓
Policy Version
      ↓
Replay
      ↓
Decision

Esto facilita análisis retrospectivo.

118. Policy Time Travel

Debe poder evaluarse:

Policy v10
at historical timestamp

cuando la auditoría lo requiera.

119. Policy Lifecycle Management

Las policies deben poder:

Create
Validate
Test
Approve
Activate
Monitor
Supersede
Rollback
Deprecate
Archive
120. Policy Retirement

Una policy obsoleta debe:

Mark Deprecated
 ↓
Migrate Consumers
 ↓
Disable
 ↓
Archive
121. Policy Ownership Transfer

Debe ser posible transferir ownership:

Team A
 ↓
Team B

manteniendo el historial.

122. Policy Documentation

Cada policy debe documentar:

Purpose
Scope
Inputs
Outputs
Rules
Dependencies
Failure Behavior
Owner
Lifecycle
123. Policy Contract

Ejemplo:

Policy:
billing.refund.approval

Purpose:
Determine whether refund requires approval.

Inputs:
actor
tenant
amount
currency

Output:
ALLOW
REQUIRE_APPROVAL
DENY

Default:
REQUIRE_APPROVAL

Owner:
Billing Domain

Failure:
REQUIRE_APPROVAL
124. Multi-Tenant Policy Architecture
                    Global Policy
                         │
              ┌──────────┴──────────┐
              ▼                     ▼
         Tenant A Policy       Tenant B Policy
              │                     │
              ▼                     ▼
       Effective Policy A     Effective Policy B
              │                     │
              ▼                     ▼
        Runtime A              Runtime B
125. Global Policy Override

Una policy global puede imponer:

Global DENY

sobre todos los tenants.

Esto debe estar definido como una regla explícita de precedence.

126. Tenant Restriction

Un tenant puede ser más restrictivo:

Global:
ALLOW

Tenant:
DENY

El resultado dependerá del modelo de combinación, pero para políticas de seguridad normalmente debe existir una estrategia de restricción segura.

127. Policy Configuration

E18 puede suministrar:

Thresholds
Timeouts
Limits
Feature Parameters

E20 utiliza esos parámetros dentro de reglas.

128. Policy and Feature Rollout

E19 puede habilitar una feature:

Feature ON

pero E20 puede restringirla:

Feature ON
+
Policy DENY
=
DENY
129. Canonical Runtime Decision Flow
Request
   │
   ▼
Authentication
   │
   ▼
Authorization
   │
   ▼
Feature Flag Evaluation
   │
   ▼
Runtime Policy Evaluation
   │
   ▼
Domain Validation
   │
   ▼
Operation
   │
   ▼
Audit / Events

El orden concreto puede variar por boundary, pero las responsabilidades deben mantenerse separadas.

130. Architectural Rules
Rule 1

Runtime Policy debe ser declarativa siempre que sea posible.

Rule 2

Una policy debe tener una fuente de verdad identificable.

Rule 3

Las policies deben estar versionadas.

Rule 4

Las decisiones deben ser deterministas cuando sea posible.

Rule 5

Toda policy crítica debe tener un comportamiento explícito ante fallos.

Rule 6

El runtime debe poder operar con un Last Known Good cuando sea seguro hacerlo.

Rule 7

Las policies deben poder auditarse.

Rule 8

Las policies deben poder simularse antes de activarse.

Rule 9

El motor no debe ejecutar código arbitrario.

Rule 10

Las policies no deben convertirse en sustituto de la lógica de dominio.

Rule 11

Authorization y Policy deben permanecer conceptualmente separados.

Rule 12

Feature Flags y Policies deben permanecer conceptualmente separados.

Rule 13

Configuration y Policy deben permanecer conceptualmente separados.

Rule 14

La evaluación debe tener límites de recursos y tiempo.

Rule 15

E17 proporciona caching; E20 mantiene la semántica de runtime policy.

131. Definition of Done

E20 queda definido cuando EVOXA dispone de:

✓ Policy Definition
✓ Policy Registry
✓ Policy Schema
✓ Policy Types
✓ Policy Keys
✓ Policy Ownership
✓ Policy Context
✓ Context Minimization
✓ Policy Input Model
✓ Policy Output Model
✓ Policy Explainability
✓ Policy Rules
✓ Rule Precedence
✓ Policy Composition
✓ Policy Combination
✓ Conflict Resolution
✓ Policy Hierarchy
✓ Policy Scope
✓ Tenant Policies
✓ Tenant Isolation
✓ Environment Policies
✓ Region Policies
✓ Runtime Enforcement
✓ Pre-Action Enforcement
✓ In-Flight Enforcement
✓ Post-Action Enforcement
✓ Policy Enforcement Point
✓ Policy Decision Point
✓ Policy Administration Point
✓ Control Plane
✓ Data Plane
✓ Local Evaluation
✓ Remote Evaluation
✓ Hybrid Evaluation
✓ Policy Snapshots
✓ Snapshot Consistency
✓ Policy Bundles
✓ Policy Dependencies
✓ Dependency Validation
✓ Policy Validation
✓ Syntax Validation
✓ Semantic Validation
✓ Conflict Detection
✓ Policy Simulation
✓ Dry Run
✓ Shadow Policy
✓ Policy Comparison
✓ Progressive Rollout
✓ Policy Activation
✓ Policy Lifecycle
✓ Policy Versioning
✓ Policy Diff
✓ Policy Rollback
✓ Last Known Good
✓ Policy Availability
✓ Fail-Open / Fail-Closed
✓ Policy Freshness
✓ Policy Propagation
✓ Policy Events
✓ Policy Messaging
✓ Policy Cache
✓ Compiled Policies
✓ Evaluation Performance
✓ Evaluation API
✓ Typed Decisions
✓ Decision Metadata
✓ Decision Explainability
✓ Decision Logging
✓ Policy Audit
✓ Runtime Decision Audit
✓ Policy Observability
✓ Policy Metrics
✓ Policy Anomaly Detection
✓ Automatic Safety Controls
✓ Scheduling Integration
✓ Job Integration
✓ Workflow Integration
✓ Configuration Integration
✓ Feature Flag Integration
✓ Authorization Integration
✓ Domain Integration
✓ Repository Boundary
✓ Policy Source of Truth
✓ Policy Drift Detection
✓ Policy Reconciliation
✓ Policy Integrity
✓ Policy Security
✓ Policy Injection Protection
✓ Policy Language Constraints
✓ Runtime Safety
✓ Evaluation Timeouts
✓ Resource Limits
✓ Policy Determinism
✓ External Dependency Handling
✓ Context Versioning
✓ Backward Compatibility
✓ Policy Testing
✓ Boundary Testing
✓ Regression Testing
✓ Policy Replay
✓ Policy Time Travel
✓ Policy Retirement
✓ Policy Documentation
✓ Policy Contracts
✓ Multi-Tenant Policy Architecture
132. Position in Engineering Specification

La secuencia queda:

E14 — Workflow & Orchestration
        ↓
E15 — Job & Task Processing
        ↓
E16 — Scheduling
        ↓
E17 — Caching
        ↓
E18 — Configuration
        ↓
E19 — Feature Flags
        ↓
E20 — Runtime Policy

Y la relación conceptual central:

                 E18 Configuration
                         │
                         ▼
                 E19 Feature Flags
                         │
                         ▼
                  E20 Runtime Policy
                         │
              ┌──────────┴──────────┐
              ▼                     ▼
        E05 Authorization      Domain Rules
              │                     │
              └──────────┬──────────┘
                         ▼
                    Runtime Action

La frontera queda definida así:

E18 parametriza.
E19 expone.
E20 restringe y decide bajo reglas.
E05 autoriza.
El Domain Layer ejecuta la semántica del negocio.

Siguiente capítulo: E21 — EVOXA Rules Engine Architecture.
