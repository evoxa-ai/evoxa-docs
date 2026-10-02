E63 — EVOXA DATA GOVERNANCE ENFORCEMENT ARCHITECTURE
Engineering Specification

Pertenece a: Engineering Specification
Capítulo: E63
Anterior: E62 — Data Governance Control Plane Architecture
Siguiente: E64 — Data Governance Compliance Architecture

1. Propósito

E63 define cómo las decisiones producidas por el Data Governance Control Plane se convierten en controles efectivos sobre el Data Plane.

E62 estableció:

Policy
   ↓
Rule
   ↓
Decision

E63 establece:

Decision
   ↓
Enforcement Point
   ↓
Allow / Deny / Block / Restrict / Review
   ↓
Actual Operation

El objetivo es evitar que Governance sea únicamente declarativo.

Una política que puede decidir pero no puede hacerse cumplir no constituye enforcement completo.

2. Objetivo Arquitectónico

La arquitectura debe permitir que EVOXA aplique governance:

Before execution
During execution
After execution

y en diferentes superficies:

API
Database
Data Product
Pipeline
Event
Message
Application
Workflow
Job
File
Storage
Analytics
3. Enforcement Architecture
                  GOVERNANCE CONTROL PLANE
                           │
                           │ Decision
                           ▼
                  ┌─────────────────┐
                  │ Decision Router │
                  └────────┬────────┘
                           │
       ┌───────────────────┼────────────────────┐
       ▼                   ▼                    ▼
   API PEP             Data PEP             Event PEP
       │                   │                    │
       ▼                   ▼                    ▼
   API Gateway        Data Service        Event Consumer
       │                   │                    │
       └───────────────────┼────────────────────┘
                           ▼
                     DATA PLANE
4. Control Plane vs Enforcement Plane

E63 introduce una separación conceptual adicional:

Control Plane
─────────────
Decides

Enforcement Plane
─────────────────
Applies

Data Plane
───────────
Executes

Por tanto:

Policy
  ↓
Decision
  ↓
Enforcement
  ↓
Execution
5. Policy Enforcement Point

El componente que aplica una decisión se denomina:

PEP
Policy Enforcement Point

Responsabilidades:

receive request
resolve decision
apply decision
block unauthorized operation
apply conditions
emit enforcement evidence
6. Policy Decision Point

El PDP permanece en el Control Plane:

PDP
Policy Decision Point

El PEP consume su decisión.

Request
   ↓
PEP
   ↓
PDP
   ↓
Decision
   ↓
PEP
   ↓
Execute / Block
7. Separación PDP / PEP

Esta separación es fundamental.

PDP
────
"What should happen?"

PEP
────
"Make it happen."

Esto permite que una misma política se aplique sobre múltiples superficies.

8. Enforcement Decision

Las decisiones mínimas son:

ALLOW
DENY
BLOCK
WARN
REVIEW
CONDITIONAL
9. ALLOW
Decision = ALLOW

El PEP permite la operación.

Puede incluir condiciones:

ALLOW
conditions:
    read_only
    masked_fields
    limited_scope
10. DENY
Decision = DENY

La operación no debe ejecutarse.

Debe devolverse una respuesta controlada.

11. BLOCK

BLOCK representa una operación detenida por enforcement.

Puede implicar:

transaction rollback
pipeline stop
message rejection
job cancellation
API rejection
12. WARN

WARN permite la operación pero registra una violación o condición relevante.

Request
 ↓
Policy
 ↓
WARN
 ↓
Execute
 ↓
Evidence

Debe utilizarse únicamente cuando la política lo permita.

13. REVIEW

La operación pasa a revisión.

Request
 ↓
PEP
 ↓
REVIEW
 ↓
Workflow
 ↓
Human Decision
 ↓
ALLOW / DENY
14. CONDITIONAL

La operación se permite bajo condiciones explícitas.

Ejemplo:

ALLOW
IF:
    only aggregated data
    no raw PII
    maximum 1000 records

El PEP debe poder aplicar esas condiciones.

15. Enforcement Context

Cada enforcement debe transportar:

EnforcementContext
{
    decisionId,
    actor,
    asset,
    operation,
    policyVersion,
    ruleVersion,
    conditions,
    correlationId,
    expiresAt
}
16. Enforcement Flow
Incoming Operation
        ↓
Identity
        ↓
Context
        ↓
Policy Evaluation
        ↓
Governance Decision
        ↓
PEP
        ↓
Enforcement
        ↓
Execution
        ↓
Evidence
17. Enforcement Modes

EVOXA debe soportar:

Preventive
Detective
Corrective
Compensating
18. Preventive Enforcement

Impide que ocurra una operación no permitida.

Request
 ↓
PEP
 ↓
DENY
 ↓
STOP

Ejemplo:

Unauthorized data access
19. Detective Enforcement

Permite detectar después del evento.

Operation
 ↓
Execute
 ↓
Detection
 ↓
Violation

Útil para:

monitoring
analytics
legacy systems
migration
20. Corrective Enforcement

Corrige una violación.

Violation
 ↓
Remediation
 ↓
Correct State

Ejemplo:

Asset without owner
 ↓
Remediation Workflow
 ↓
Owner Assigned
21. Compensating Enforcement

Cuando el control principal no puede aplicarse, se utiliza otro mecanismo equivalente o mitigador.

Primary Control
      ↓
Unavailable
      ↓
Compensating Control

Debe existir una política explícita.

22. Enforcement Locations

Los PEPs pueden existir en:

API Gateway
Application Service
Domain Service
Database Layer
Data Access Layer
Pipeline
Workflow
Job Worker
Message Consumer
Event Processor
Storage
Data Product
Analytics Layer
23. API Enforcement
Client
  ↓
API Gateway
  ↓
PEP
  ↓
Governance Decision
  ↓
Service

Puede controlar:

endpoint
operation
resource
tenant
purpose
classification
24. Application Enforcement

Los Application Services pueden aplicar governance antes de ejecutar comandos.

Application Service
       ↓
Governance Check
       ↓
Decision
       ↓
Domain Operation
25. Domain Enforcement

Los invariantes críticos del dominio deben permanecer protegidos aunque una llamada evite capas superiores.

Application
     ↓
Domain
     ↓
Domain Governance Invariant

No debe confiarse exclusivamente en UI o API Gateway.

26. Data Access Enforcement

El Data Access Layer puede aplicar:

row filtering
column filtering
masking
tenant isolation
purpose restrictions
27. Database Enforcement

Cuando la tecnología lo permita:

Database
├── Row-Level Security
├── Views
├── Roles
├── Permissions
└── Stored Controls

Pero las reglas de negocio complejas no deben quedar acopladas exclusivamente a la base de datos.

28. Data Product Enforcement

Un Data Product debe publicar su estado de governance:

AVAILABLE
RESTRICTED
CERTIFIED
DEPRECATED
BLOCKED

El consumidor no debería descubrir la restricción únicamente después de intentar consumirlo.

29. Pipeline Enforcement

Antes de procesar:

Pipeline
 ↓
Governance Check
 ↓
Policy
 ↓
Decision
 ↓
Process / Block
30. Pipeline Gates

Los pipelines críticos pueden tener:

Schema Gate
Quality Gate
Classification Gate
Contract Gate
Policy Gate
Security Gate
31. Pipeline Example
Source
 ↓
Schema Validation
 ↓
Classification
 ↓
Quality Check
 ↓
Governance Policy
 ↓
ALLOW
 ↓
Transformation
 ↓
Publish
32. Event Enforcement

Para eventos:

Producer
 ↓
Event
 ↓
Governance Check
 ↓
Consumer

o:

Consumer
 ↓
PEP
 ↓
Decision
 ↓
Process / Reject
33. Message Enforcement

Puede controlar:

topic
message type
producer
consumer
payload classification
operation
34. Job Enforcement

Antes de ejecutar un Job:

Job Scheduler
 ↓
Governance Check
 ↓
Decision
 ↓
Worker
35. Workflow Enforcement

Cada transición crítica puede tener una policy gate:

Workflow
   ↓
Transition
   ↓
Governance Check
   ↓
ALLOW
   ↓
Next State
36. Storage Enforcement

Para almacenamiento:

Object
 ↓
Classification
 ↓
Retention Policy
 ↓
Access Policy
 ↓
Storage Operation
37. Analytics Enforcement

Debe poder controlar:

dataset access
metric visibility
field masking
aggregation requirements
export permissions
38. Search Enforcement

Los resultados de búsqueda también deben estar gobernados.

Search Query
 ↓
Search PEP
 ↓
Governance Filter
 ↓
Allowed Results

No debe ser posible eludir una restricción simplemente usando Search.

39. Read Model Enforcement

Los Read Models deben respetar las restricciones del activo original.

Source Data
 ↓
Governance
 ↓
Projection
 ↓
Read Model

La proyección no debe convertirse en un canal de bypass.

40. Projection Enforcement

Antes de generar una proyección:

Source
 ↓
Policy Check
 ↓
Transformation
 ↓
Projection
41. Export Enforcement

Los exports requieren especial atención.

Export Request
 ↓
Governance Check
 ↓
Classification
 ↓
Purpose
 ↓
Decision
 ↓
Export
42. Bulk Access

Las operaciones masivas deben poder tener políticas específicas:

bulk download
bulk export
bulk delete
bulk transformation
43. Rate-Based Enforcement

Puede limitar:

records
requests
exports
queries
time

según policy.

44. Purpose-Based Enforcement

El PEP puede verificar:

purpose

antes de permitir acceso.

Ejemplo:

Purpose = Analytics

puede permitir únicamente:

aggregated data
45. Attribute-Based Enforcement

La decisión puede utilizar:

actor
role
department
domain
classification
asset
purpose
environment
risk
46. Context-Based Enforcement

El mismo usuario puede recibir decisiones distintas según:

environment
time
location
asset
purpose
risk

si la política lo establece.

47. Dynamic Enforcement
Context
 ↓
Policy
 ↓
Decision
 ↓
Dynamic Constraint
 ↓
Operation

Ejemplo:

ALLOW
until 18:00
48. Static Enforcement

Para reglas estables:

role → permission

puede aplicarse directamente mediante mecanismos de autorización.

49. Enforcement Granularity

Debe soportar diferentes niveles:

System
Service
API
Resource
Dataset
Table
Column
Row
Field
Record
Event
Message
Operation
50. Fine-Grained Enforcement

Para activos sensibles:

Dataset
 ↓
Table
 ↓
Column
 ↓
Row

pueden existir diferentes controles.

51. Masking

Cuando la policy lo indique:

Raw value
 ↓
Masking PEP
 ↓
Masked value

Ejemplo conceptual:

123456789
↓
******789
52. Filtering
Query
 ↓
Governance Filter
 ↓
Permitted Rows

El filtro debe ser aplicado de forma consistente y no únicamente en la interfaz.

53. Transformation Enforcement

Puede requerirse transformar:

PII
 ↓
Tokenization

o:

Sensitive Data
 ↓
Aggregation

antes de permitir consumo.

54. Conditional Access

Ejemplo:

ALLOW
IF:
    actor = authorized
    AND
    purpose = approved
    AND
    data is masked
55. Enforcement Chain

Los controles pueden encadenarse:

Authentication
 ↓
Authorization
 ↓
Governance
 ↓
Quality
 ↓
Contract
 ↓
Execution
56. Ordering

El orden debe ser explícito.

Por ejemplo:

Identity
  ↓
Access
  ↓
Governance
  ↓
Execution

No debe depender del orden accidental de middleware.

57. Enforcement Priority

Cuando múltiples controles bloquean:

Security
    >
Governance
    >
Quality
    >
Advisory

La precedencia concreta debe estar definida por EVOXA.

58. No Bypass Principle

Todo canal hacia un activo gobernado debe pasar por un enforcement apropiado.

API ────────┐
Pipeline ───┤
Event ──────┤
Job ────────┼──→ Governed Asset
Search ─────┤
Export ─────┘
59. Bypass Detection

El sistema debe detectar accesos que eviten PEPs obligatorios.

Expected Access Path
        ≠
Actual Access Path
        ↓
Governance Finding
60. Legacy Systems

Cuando un sistema antiguo no puede incorporar un PEP:

Legacy System
      ↓
Gateway / Adapter
      ↓
Governance Enforcement

Debe evitarse confiar únicamente en que el sistema legacy "respete" la policy.

61. Enforcement Gateway

Puede actuar como punto de control:

Consumer
 ↓
Governance Gateway
 ↓
Legacy System
62. Enforcement Adapter

Para sistemas no compatibles:

Governance PEP
      ↓
Adapter
      ↓
Legacy Interface
63. Enforcement SDK

EVOXA puede proporcionar un SDK para servicios internos:

governance.check()
governance.authorize()
governance.enforce()
governance.requireCertification()

Esto reduce implementaciones inconsistentes.

64. Enforcement Middleware

Para APIs:

Request
 ↓
Middleware
 ↓
Governance Check
 ↓
Handler
65. Enforcement Interceptor

Para servicios:

Service Call
 ↓
Interceptor
 ↓
Governance Decision
 ↓
Service Method
66. Enforcement Sidecar

En entornos distribuidos puede utilizarse:

Application
   │
   ├── Sidecar PEP
   │
   └── Governance PDP

Esto desacopla el enforcement de la aplicación.

67. Sidecar Considerations

Ventajas:

centralized enforcement
consistent implementation
low application coupling

Costes:

network hop
operational complexity
debugging
latency
68. Local PDP Cache

Cuando la latencia sea crítica:

PEP
 ↓
Local Policy Cache
 ↓
Decision

pero debe respetarse la versión de policy.

69. Decision Cache

Debe invalidarse ante:

policy change
rule change
classification change
ownership change
exception change
certification revocation
70. Fail-Closed Enforcement

Para controles críticos:

PEP unavailable
 ↓
DENY / BLOCK

Debe evitarse:

PEP unavailable
 ↓
ALLOW

sin autorización explícita.

71. Fail-Open Enforcement

Solo puede utilizarse cuando:

risk acceptable
policy explicitly permits
operation non-critical

y debe registrarse.

72. Graceful Degradation

Puede existir:

FULL
 ↓
DEGRADED
 ↓
RESTRICTED
 ↓
BLOCKED

según el tipo de fallo.

73. Enforcement Evidence

Cada enforcement relevante debe registrar:

decisionId
pepId
action
result
timestamp
resource
actor
policyVersion
correlationId
74. Enforcement Audit

Debe poder responderse:

Who attempted access?
What asset?
What operation?
Which policy?
Which decision?
Which PEP?
What happened?
75. Enforcement Telemetry

Métricas:

enforcement_requests
enforcement_allows
enforcement_denies
enforcement_blocks
enforcement_reviews
enforcement_failures
enforcement_latency
bypass_attempts
76. Enforcement Tracing
Request
 ↓
PEP
 ↓
PDP
 ↓
Policy
 ↓
Rules
 ↓
Decision
 ↓
PEP
 ↓
Execution

Todo debe compartir traceId.

77. Enforcement Errors

Los errores deben ser explícitos:

GOVERNANCE_DENIED
GOVERNANCE_BLOCKED
GOVERNANCE_REVIEW_REQUIRED
GOVERNANCE_UNAVAILABLE
GOVERNANCE_CONTEXT_INVALID

No:

500 Internal Error

como única explicación.

78. User-Facing Errors

El sistema debe revelar únicamente la información necesaria.

Access denied by governance policy.

No debe exponer:

internal rule definitions
sensitive classifications
security internals

cuando no corresponda.

79. Enforcement API

Conceptualmente:

POST /governance/evaluate
POST /governance/enforce
GET  /governance/decisions/{id}
80. Enforcement Contract

Request:

{
    actor,
    resource,
    operation,
    purpose,
    context
}

Response:

{
    decision,
    conditions,
    policyVersion,
    decisionId,
    expiresAt
}
81. Enforcement Conditions

Las condiciones pueden incluir:

maskFields
allowedRows
maxRecords
allowedOperations
expiration
requiredApproval
requiredPurpose
82. Enforcement Composition

Varias decisiones pueden combinarse:

Authorization
    +
Governance
    +
Quality
    +
Contract

Resultado:

FINAL ENFORCEMENT DECISION
83. Enforcement Decision Matrix

Ejemplo:

Authorization	Governance	Quality	Resultado
ALLOW	ALLOW	PASS	ALLOW
ALLOW	DENY	PASS	DENY
ALLOW	ALLOW	FAIL	BLOCK
DENY	ALLOW	PASS	DENY
ALLOW	REVIEW	PASS	REVIEW
ALLOW	ALLOW	UNKNOWN	POLICY-DEPENDENT
84. Governance vs Authorization

No deben confundirse:

Authorization
→ "¿Puede este actor realizar esta operación?"

Governance
→ "¿Debe esta operación permitirse dadas las reglas de gobierno?"

Una operación requiere cumplir ambas cuando ambas sean aplicables.

85. Governance vs Security

Security puede decir:

actor authenticated

Governance puede decir:

purpose not permitted

Resultado:

DENY
86. Governance vs Quality

Un usuario puede tener autorización y aun así no poder consumir un dataset porque:

quality = CRITICAL_FAILURE

si la policy lo establece.

87. Governance vs Contract

Una operación puede estar autorizada pero ser inválida porque:

contract incompatible

Resultado:

BLOCK
88. Enforcement Pipeline
Authentication
       ↓
Authorization
       ↓
Governance
       ↓
Contract
       ↓
Quality
       ↓
Execution

El orden final debe ajustarse al tipo de operación.

89. Enforcement Transactions

Cuando enforcement bloquea una operación transaccional:

Request
 ↓
PEP
 ↓
DENY
 ↓
No side effect

El control debe ocurrir antes del efecto irreversible cuando sea posible.

90. Irreversible Operations

Para:

delete
publish
export
share
transfer
destroy

se recomienda enforcement preventivo.

91. Post-Execution Controls

Para operaciones que no pueden interceptarse:

Execute
 ↓
Detect
 ↓
Remediate

La arquitectura debe registrar explícitamente que el control es detective/correctivo.

92. Enforcement Windows

Algunas políticas pueden definir:

effectiveFrom
effectiveTo

El PEP debe verificar la vigencia.

93. Emergency Enforcement

Debe existir un mecanismo controlado para:

EMERGENCY_BLOCK

pero con:

authorization
expiration
audit
94. Emergency Override

Un override nunca debe ser:

permanent
silent
unaudited

Debe ser:

explicit
time-bound
attributable
reviewable
95. Enforcement Rollback

Cuando una policy nueva produce efectos incorrectos:

Policy v4
 ↓
Problem
 ↓
Disable / Rollback
 ↓
Policy v3

El cambio debe quedar auditado.

96. Canary Enforcement

Puede activarse en una parte del sistema:

10% traffic
 ↓
Observe
 ↓
Validate
 ↓
50%
 ↓
100%
97. Enforcement Shadow Mode

La policy evalúa pero no bloquea:

Request
 ↓
Policy
 ↓
Decision = DENY
 ↓
Shadow Mode
 ↓
Request still executes
 ↓
Telemetry

Útil para medir impacto.

98. Enforcement Testing

Debe probarse:

ALLOW
DENY
BLOCK
REVIEW
CONDITIONAL
TIMEOUT
PDP unavailable
PEP unavailable
policy expired
exception expired
99. Negative Testing

Debe intentarse explícitamente:

bypass
missing context
invalid identity
expired decision
stale cache
duplicate request
direct database access
alternate API
bulk endpoint
export path
100. Enforcement Security Invariants
Invariant 1
A governed operation cannot bypass its required PEP.
Invariant 2
A DENY decision cannot result in execution.
Invariant 3
An expired decision cannot authorize an operation.
Invariant 4
An expired exception cannot override policy.
Invariant 5
Critical enforcement failures fail safely.
Invariant 6
Every critical enforcement decision is attributable.
Invariant 7
Policy version is known at enforcement time.
Invariant 8
Conditional decisions cannot be treated as unconditional ALLOW.
101. Enforcement Architecture by Layer
Layer 1 — Gateway Enforcement
Layer 2 — Service Enforcement
Layer 3 — Domain Enforcement
Layer 4 — Data Access Enforcement
Layer 5 — Pipeline Enforcement
Layer 6 — Event Enforcement
Layer 7 — Workflow Enforcement
Layer 8 — Storage Enforcement
Layer 9 — Analytics Enforcement
Layer 10 — Detection & Remediation
102. Reference Architecture
                         ┌───────────────────────┐
                         │       CLIENTS         │
                         └───────────┬───────────┘
                                     │
                                     ▼
                         ┌───────────────────────┐
                         │   API GATEWAY / PEP   │
                         └───────────┬───────────┘
                                     │
                                     ▼
                         ┌───────────────────────┐
                         │ GOVERNANCE PDP        │
                         │                       │
                         │ Context               │
                         │ Policy                │
                         │ Rules                 │
                         │ Decision              │
                         └───────────┬───────────┘
                                     │
                               Decision
                                     │
                                     ▼
                         ┌───────────────────────┐
                         │      PEP ROUTER       │
                         └───────────┬───────────┘
                                     │
          ┌──────────────┬───────────┼────────────┬──────────────┐
          ▼              ▼           ▼            ▼              ▼
       Service         Data       Pipeline       Event         Job
         PEP            PEP          PEP           PEP          PEP
          │              │           │             │             │
          └──────────────┴───────────┼─────────────┴─────────────┘
                                     ▼
                              ┌───────────────┐
                              │   DATA PLANE  │
                              └───────┬───────┘
                                      │
                                      ▼
                              Evidence / Audit
103. Implementation Strategy
Phase 1 — Core Enforcement
PEP interface
PDP integration
Decision contract
Evidence
Audit
Phase 2 — API Enforcement
Gateway
Middleware
Service enforcement
Phase 3 — Data Enforcement
Data access
Masking
Filtering
Export controls
Phase 4 — Processing Enforcement
Pipelines
Jobs
Events
Workflows
Phase 5 — Advanced Enforcement
Conditional access
Dynamic policies
Shadow mode
Canary rollout
Drift detection
Bypass detection
104. Minimum Viable Enforcement Platform

Debe contener:

✓ PDP integration
✓ PEP abstraction
✓ Decision contract
✓ Allow/Deny
✓ Policy version propagation
✓ Enforcement audit
✓ Enforcement evidence
✓ Fail-safe behavior
✓ API enforcement
✓ Service enforcement
✓ Basic data enforcement
105. Production-Grade Enforcement Platform

Añade:

✓ Fine-grained controls
✓ Conditional enforcement
✓ Masking
✓ Row filtering
✓ Pipeline gates
✓ Event enforcement
✓ Workflow enforcement
✓ Export controls
✓ Shadow mode
✓ Canary rollout
✓ Bypass detection
✓ Emergency controls
✓ Automated remediation
✓ Full distributed tracing
106. Acceptance Criteria

E63 está implementado cuando:

✓ Existe una abstracción PEP
✓ PDP y PEP están separados
✓ Las decisiones pueden aplicarse en runtime
✓ ALLOW/DENY funcionan correctamente
✓ CONDITIONAL puede hacerse cumplir
✓ REVIEW puede iniciar workflow
✓ Los controles críticos fallan de forma segura
✓ Los datos pueden filtrarse cuando la policy lo exige
✓ Los datos pueden enmascararse cuando la policy lo exige
✓ APIs están protegidas
✓ Pipelines pueden detenerse
✓ Jobs pueden bloquearse
✓ Eventos pueden rechazarse
✓ Workflows pueden detener transiciones
✓ Exports pueden controlarse
✓ Existe evidencia de enforcement
✓ Existe audit trail
✓ Existe detección de bypass
✓ Existe observabilidad
✓ Existe rollback
✓ Existe testing de enforcement
107. Principio Rector

EVOXA Data Governance Enforcement Architecture convierte las decisiones del Governance Control Plane en controles técnicos efectivos sobre cada superficie relevante del Data Plane, utilizando Policy Enforcement Points explícitos, condiciones verificables, mecanismos preventivos/detectivos/correctivos, evidencia auditable y comportamiento fail-safe para impedir que las políticas de governance sean meramente declarativas.

La cadena arquitectónica queda:

E59 — DATA GOVERNANCE ARCHITECTURE
              ↓
E60 — DATA GOVERNANCE OPERATING MODEL
              ↓
E61 — DATA GOVERNANCE IMPLEMENTATION ARCHITECTURE
              ↓
E62 — DATA GOVERNANCE CONTROL PLANE ARCHITECTURE
              ↓
E63 — DATA GOVERNANCE ENFORCEMENT ARCHITECTURE
              ↓
E64 — DATA GOVERNANCE COMPLIANCE ARCHITECTURE

E62 = decide.
E63 = hace cumplir.
E64 = verifica y demuestra el cumplimiento.
