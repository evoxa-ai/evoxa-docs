E62 — EVOXA DATA GOVERNANCE CONTROL PLANE ARCHITECTURE
Engineering Specification

Pertenece a: Engineering Specification
Capítulo: E62
Anterior: E61 — Data Governance Implementation Architecture
Siguiente: E63 — Data Governance Enforcement Architecture

1. Propósito

E62 define la arquitectura interna del EVOXA Data Governance Control Plane.

E61 estableció cómo se implementa técnicamente Data Governance.

E62 profundiza en el núcleo de control que permite que las decisiones de governance sean:

centralizadas cuando corresponde,
consistentes,
versionadas,
auditables,
reproducibles,
distribuibles,
ejecutables,
y separadas del Data Plane.

La arquitectura fundamental es:

                 DATA GOVERNANCE CONTROL PLANE

        ┌─────────────────────────────────────────┐
        │ Governance API / Control Interfaces      │
        └───────────────────┬─────────────────────┘
                            │
        ┌───────────────────▼─────────────────────┐
        │        Governance Control Core          │
        │                                         │
        │ Policy │ Rules │ Decision │ Context     │
        │ State  │ Workflow │ Evidence │ Audit    │
        └───────────────────┬─────────────────────┘
                            │
             ┌──────────────┼──────────────┐
             ▼              ▼              ▼
        Metadata        Governance      External
        Registry        Repository      Systems
             │              │              │
             └──────────────┼──────────────┘
                            ▼
                     Governance Events
                            │
                            ▼
                    EVOXA Data Plane
2. Objetivo Arquitectónico

El Control Plane debe responder cinco preguntas fundamentales:

1. ¿Qué debe cumplirse?
2. ¿Qué regla aplica?
3. ¿Qué decisión corresponde?
4. ¿Cómo se ejecuta esa decisión?
5. ¿Cómo demostramos posteriormente que ocurrió?

Por tanto:

Policy
   ↓
Rule
   ↓
Decision
   ↓
Enforcement
   ↓
Evidence
3. Principio Fundamental

El Governance Control Plane decide y coordina; el Data Plane ejecuta las operaciones sobre los datos.

Esto evita convertir el sistema de governance en un componente acoplado a cada sistema de datos.

4. Control Plane vs Data Plane
┌──────────────────────────────────────────────┐
│             GOVERNANCE CONTROL PLANE         │
│                                              │
│ Policies                                     │
│ Rules                                        │
│ Decisions                                    │
│ Governance Context                           │
│ Ownership                                    │
│ Classification                               │
│ Certifications                               │
│ Exceptions                                   │
│ Evidence                                     │
│ Audit                                        │
└──────────────────────┬───────────────────────┘
                       │
                Governance Contract
                       │
                       ▼
┌──────────────────────────────────────────────┐
│                 DATA PLANE                   │
│                                              │
│ APIs                                         │
│ Databases                                    │
│ Data Products                                │
│ Pipelines                                    │
│ Events                                       │
│ Analytics                                    │
│ Applications                                │
└──────────────────────────────────────────────┘
5. Componentes Principales

El Control Plane estará compuesto por:

Governance Gateway
Governance Context Service
Policy Registry
Policy Engine
Rules Engine
Decision Engine
Governance State Manager
Workflow Coordinator
Approval Service
Exception Service
Certification Service
Metadata Registry
Governance Relationship Graph
Evidence Service
Audit Service
Event Publisher
Governance Query Service
Governance Administration Service
6. Governance Gateway

Es la entrada principal al Control Plane.

Responsabilidades:

authentication
authorization
request normalization
context creation
correlation
routing
rate limiting
audit initiation
7. Governance Context Service

Construye el contexto necesario para evaluar una decisión.

GovernanceContext
{
    actor,
    tenant,
    domain,
    asset,
    operation,
    purpose,
    environment,
    classification,
    policyContext,
    correlationId
}
8. Context Resolution

El contexto puede obtenerse de:

request
identity
asset registry
metadata
classification
contract
lineage
runtime environment

Proceso:

Request
 ↓
Identity Resolution
 ↓
Asset Resolution
 ↓
Metadata Resolution
 ↓
Policy Context
 ↓
Governance Context
9. Policy Registry

Es la fuente de verdad para las políticas.

Debe soportar:

create
update
version
approve
activate
deprecate
retire
retrieve
10. Policy Identity

Una política debe tener identidad estable:

policyId
policyVersion
policyStatus

Nunca debe depender únicamente del nombre humano.

11. Policy State
DRAFT
REVIEW
APPROVED
ACTIVE
SUSPENDED
DEPRECATED
RETIRED
12. Policy Activation

Una política solamente puede pasar a ACTIVE cuando:

syntax valid
semantics valid
dependencies valid
approval completed
deployment successful

según el tipo de política.

13. Policy Engine

El Policy Engine determina:

which policies apply

No necesariamente toma por sí solo la decisión final.

Context
   ↓
Policy Resolution
   ↓
Applicable Policies
14. Policy Resolution

Puede considerar:

tenant
domain
asset
classification
operation
actor
environment
risk
15. Policy Precedence

Cuando existen múltiples políticas:

Global
  ↓
Organization
  ↓
Domain
  ↓
Asset
  ↓
Operation

debe existir una regla explícita de precedencia.

No se debe depender de un orden accidental.

16. Policy Conflict Resolution

Si dos políticas entran en conflicto:

Policy A → ALLOW
Policy B → DENY

el sistema debe aplicar una estrategia definida.

Para controles de seguridad y governance críticos, el comportamiento recomendado es:

DENY / REVIEW

cuando no pueda resolverse el conflicto de forma determinista.

17. Rules Engine

El Rules Engine evalúa las condiciones concretas.

Policy
   ↓
Rules
   ↓
Conditions
   ↓
Results

Ejemplo:

IF
    asset.classification = CONFIDENTIAL
AND
    actor.clearance < REQUIRED
THEN
    DENY
18. Rule Types

Las reglas pueden clasificarse como:

Eligibility
Classification
Quality
Ownership
Lifecycle
Access
Compliance
Contract
Certification
Exception
Operational
19. Rule Evaluation

Cada evaluación debe producir información suficiente para explicar el resultado:

ruleId
ruleVersion
inputs
result
reason
timestamp
20. Decision Engine

El Decision Engine consolida los resultados de las reglas.

Applicable Policies
        ↓
Rule Evaluation
        ↓
Decision Aggregation
        ↓
Governance Decision
21. Decision Types
ALLOW
DENY
BLOCK
WARN
REVIEW
ESCALATE
CONDITIONAL
22. Decision Object
GovernanceDecision
{
    decisionId,
    decision,
    subject,
    asset,
    operation,
    policySet,
    ruleResults,
    reason,
    issuedAt,
    expiresAt,
    correlationId
}
23. Decision Explainability

Una decisión no debe ser solamente:

DENY

Debe poder explicar:

DENY
because:
  Policy P-104
  Rule R-104-3
  Classification = RESTRICTED
  Actor clearance = insufficient
24. Decision Determinism

Para decisiones críticas:

same input
+
same policy version
+
same rule version
=
same decision

salvo dependencias dinámicas explícitamente declaradas.

25. Decision Expiration

Algunas decisiones tienen duración limitada:

issuedAt
expiresAt

Una decisión expirada no debe continuar siendo válida.

26. Governance State Manager

Mantiene el estado operacional del governance.

Asset State
Policy State
Certification State
Exception State
Workflow State
Compliance State
27. State Machine

Los estados deben ser explícitos.

Ejemplo:

UNREGISTERED
     ↓
REGISTERED
     ↓
GOVERNED
     ↓
CERTIFIED
     ↓
RETIRED
28. State Transition Validation

No deben permitirse transiciones arbitrarias.

DRAFT → ACTIVE

puede ser inválido si requiere:

DRAFT
 ↓
REVIEW
 ↓
APPROVED
 ↓
ACTIVE
29. Governance Workflow Coordinator

Coordina procesos de larga duración.

Ejemplos:

Policy Approval
Certification
Exception Approval
Asset Onboarding
Governance Remediation
30. Workflow vs Decision

Debe mantenerse la diferencia:

Decision
→ resultado de evaluación

Workflow
→ proceso para llegar o responder a ese resultado
31. Approval Service

Gestiona las aprobaciones humanas.

Request
 ↓
Approver Resolution
 ↓
Approval
 ↓
Decision
32. Approval Evidence

Cada aprobación debe contener:

approver
role
decision
reason
timestamp
requestId
policyVersion
33. Exception Service

Administra desviaciones temporales respecto de las políticas.

Exception
├── exceptionId
├── policyId
├── scope
├── justification
├── risk
├── owner
├── approver
├── effectiveFrom
└── expiresAt
34. Exception Lifecycle
REQUESTED
 ↓
RISK_ASSESSED
 ↓
APPROVED
 ↓
ACTIVE
 ↓
EXPIRED / REVOKED
35. Exception Safety

Una excepción nunca debe:

exist indefinitely

Debe tener:

owner
justification
approval
expiration

cuando el riesgo lo requiera.

36. Certification Service

Determina si un activo cumple las condiciones necesarias para obtener una certificación.

Asset
 ↓
Governance Checks
 ↓
Certification Decision
 ↓
Certification
37. Certification State
CANDIDATE
UNDER_REVIEW
CERTIFIED
EXPIRED
REVOKED
38. Metadata Registry

El Control Plane necesita conocer el objeto sobre el cual aplica una política.

Asset
Domain
Owner
Steward
Classification
Lifecycle
Quality
Lineage
Contract
39. Governance Relationship Graph

El Control Plane debe representar relaciones:

Domain
 ↓
owns
 ↓
Asset
 ↓
governed-by
 ↓
Policy
 ↓
contains
 ↓
Rule
 ↓
produces
 ↓
Decision

Esto permite análisis de impacto.

40. Impact Analysis

Ejemplo:

Policy P-100 changes
       ↓
Rules affected
       ↓
Assets affected
       ↓
Data Products affected
       ↓
Consumers affected
41. Evidence Service

El Evidence Service conserva la evidencia que demuestra que governance ocurrió.

Policy Evidence
Decision Evidence
Approval Evidence
Certification Evidence
Exception Evidence
Validation Evidence
42. Evidence Chain
Request
 ↓
Context
 ↓
Policy
 ↓
Rules
 ↓
Decision
 ↓
Action
 ↓
Result

Cada paso debe poder reconstruirse cuando el nivel de criticidad lo requiera.

43. Evidence Integrity

La evidencia crítica debe ser:

immutable
timestamped
attributable
versioned
tamper-evident
44. Audit Service

El Audit Service registra cambios y acciones administrativas.

CREATE
UPDATE
APPROVE
REJECT
ACTIVATE
SUSPEND
RETIRE
REVOKE
45. Audit Separation

La auditoría no debe depender exclusivamente del mismo mecanismo que puede ser modificado por el operador auditado.

Debe existir una separación lógica y, cuando sea necesario, física.

46. Governance Event Publisher

Los cambios importantes producen eventos:

PolicyActivated
PolicyRetired
DecisionIssued
ExceptionApproved
ExceptionExpired
CertificationGranted
CertificationRevoked
AssetGovernanceChanged
47. Event Flow
Governance Action
       ↓
State Change
       ↓
Outbox
       ↓
Event Bus
       ↓
Consumers
48. Event Consumers

Pueden consumirlos:

Audit
Notifications
Analytics
Reporting
Automation
Data Platform
Security
Compliance
49. Governance Query Service

Proporciona una vista optimizada del estado del Control Plane.

Consultas:

getAssetGovernance()
getPolicy()
getDecision()
getCertification()
getExceptions()
getComplianceStatus()
getEvidence()
50. Governance Read Model

Puede combinar:

Ownership
Classification
Quality
Certification
Policy
Exceptions
Lineage

en una respuesta consolidada:

Asset Governance Status
51. Governance Control Plane API

Modelo conceptual:

/governance/assets
/governance/policies
/governance/rules
/governance/decisions
/governance/workflows
/governance/approvals
/governance/exceptions
/governance/certifications
/governance/evidence
/governance/audit
52. Internal Service APIs

Los servicios internos deben comunicarse mediante contratos explícitos:

PolicyResolver
RuleEvaluator
DecisionEvaluator
ContextResolver
EvidenceWriter
AuditWriter
WorkflowCoordinator
53. Policy Evaluation Pipeline
                 Request
                    │
                    ▼
           ┌─────────────────┐
           │ Context Builder │
           └────────┬────────┘
                    ▼
           ┌─────────────────┐
           │ Policy Resolver │
           └────────┬────────┘
                    ▼
           ┌─────────────────┐
           │  Rules Engine   │
           └────────┬────────┘
                    ▼
           ┌─────────────────┐
           │ Decision Engine │
           └────────┬────────┘
                    ▼
             Governance
               Decision
                    │
          ┌─────────┴─────────┐
          ▼                   ▼
      Enforcement          Evidence
54. Synchronous Path

Para decisiones runtime:

Request
 ↓
Context
 ↓
Policy
 ↓
Rules
 ↓
Decision
 ↓
Allow / Deny

Debe ser optimizado para baja latencia.

55. Asynchronous Path

Para procesos administrativos:

Request
 ↓
Create Workflow
 ↓
Human Review
 ↓
Approval
 ↓
Decision
 ↓
Event
56. Governance Command Bus

Las operaciones mutables pueden representarse como comandos:

RegisterAsset
ActivatePolicy
ApproveException
CertifyAsset
RevokeCertification
RetireAsset
57. Command Processing
Command
 ↓
Authorization
 ↓
Validation
 ↓
Policy Evaluation
 ↓
State Transition
 ↓
Evidence
 ↓
Event
58. Idempotency

Las operaciones críticas deben soportar idempotencia.

commandId

permite evitar que:

same command

produzca múltiples efectos.

59. Concurrency Control

El Control Plane debe protegerse contra modificaciones concurrentes.

Puede utilizar:

optimistic locking
version numbers
compare-and-swap

según el componente.

60. Policy Concurrency

No deben coexistir dos activaciones incompatibles de la misma política sin una versión explícita.

Policy P
 v4 ACTIVE

debe ser una condición inequívoca.

61. Control Plane Consistency

Los componentes críticos requieren garantías fuertes sobre:

policy state
decision state
approval state
exception state
certification state

Otros datos pueden ser eventualmente consistentes:

search index
analytics
reporting
62. Control Plane Transactions

Cuando una operación requiere múltiples cambios críticos:

State Change
+
Evidence
+
Audit

deben diseñarse con garantías de consistencia apropiadas.

63. Outbox Pattern

Para eventos críticos:

┌───────────────┐
│ Transaction   │
│ State Change  │
│ Outbox Event  │
└───────┬───────┘
        │
        ▼
    Publisher
        │
        ▼
    Event Bus
64. Failure Handling

Si el Event Bus está caído:

State
  +
Outbox

deben persistir.

El evento se publica posteriormente.

65. Governance Reconciliation

El Control Plane debe reconciliar periódicamente su estado con los sistemas externos.

Control Plane
      ↕
External Platform
66. Drift Detection

Debe detectar:

ownership drift
classification drift
policy drift
configuration drift
contract drift
certification drift
67. Drift Resolution
Drift
 ↓
Finding
 ↓
Risk Evaluation
 ↓
Remediation Workflow
 ↓
Verification
68. Runtime Decision Cache

Para políticas de alta frecuencia puede existir cache.

Decision Cache

Debe incluir:

policyVersion
contextVersion
TTL
decision
69. Cache Invalidation

Cuando cambia una política:

Policy v3 → v4

deben invalidarse decisiones incompatibles con v3.

70. Fail-Closed vs Fail-Open

El comportamiento debe definirse por política.

Critical controls
FAIL CLOSED
Advisory controls
FAIL OPEN / WARN

Nunca debe quedar implícito.

71. Control Plane Availability

Debe existir clasificación:

Critical Decision Path
High Availability

Administrative Path
Normal Availability

Analytics Path
Eventual Availability
72. Scaling

Componentes stateless:

API
Context Resolver
Policy Evaluator
Rules Engine
Decision Engine
Query Service

pueden escalar horizontalmente.

73. Stateful Components

Componentes que requieren especial protección:

Governance Database
Policy Repository
Evidence Store
Audit Store
Event Outbox
74. Security Boundary

El Control Plane constituye un security boundary lógico.

Debe proteger:

policy definitions
governance decisions
ownership
classification
exceptions
audit evidence
75. Identity Integration

Debe integrarse con el sistema de identidad de EVOXA.

Identity
 ↓
Actor
 ↓
Roles
 ↓
Permissions
 ↓
Governance Decision
76. Authorization

Debe soportar:

RBAC
ABAC
domain scoping
tenant scoping
resource scoping

cuando sean necesarios.

77. Separation of Duties

Controles sensibles pueden exigir:

Requester
    ≠
Approver

y:

Operator
    ≠
Auditor
78. Control Plane Observability

Debe emitir:

metrics
logs
traces
events
audit
79. Core Metrics
policy_evaluation_count
policy_denial_count
decision_latency
workflow_latency
approval_latency
exception_count
certification_count
governance_errors
reconciliation_failures
80. Governance SLO

Ejemplos:

Policy evaluation availability
Decision latency
Evidence persistence success
Audit persistence success
Governance event delivery
81. Distributed Tracing

Una operación debe poder seguirse:

API
 ↓
Context
 ↓
Policy
 ↓
Rule
 ↓
Decision
 ↓
Workflow
 ↓
Evidence

mediante correlationId y traceId.

82. Governance Health

El Control Plane debe poder determinar:

HEALTHY
DEGRADED
NON_COMPLIANT
BLOCKED
UNKNOWN
83. Health vs Compliance

No deben confundirse:

System HEALTH

con:

Governance COMPLIANCE

Un sistema puede estar técnicamente saludable pero no cumplir governance.

84. Control Plane Administration

Administradores autorizados deben poder:

manage policies
manage rules
manage workflows
manage roles
manage integrations
inspect decisions
inspect evidence
manage exceptions

Todo cambio debe quedar auditado.

85. Policy-as-Code

Cuando sea apropiado, las políticas deben representarse como código o artefactos versionados.

Policy
 ↓
Repository
 ↓
Review
 ↓
Test
 ↓
Deploy
86. Policy Testing

Cada política crítica debe disponer de:

positive cases
negative cases
edge cases
conflict cases
regression cases
87. Policy Simulation

Antes de activar una política:

Policy Candidate
      ↓
Simulation
      ↓
Affected Assets
      ↓
Affected Consumers
      ↓
Risk
88. Observe-Only Mode

Una política puede desplegarse primero como:

OBSERVE_ONLY

para medir impacto sin bloquear operaciones.

89. Enforcement Transition
DRAFT
 ↓
SIMULATE
 ↓
OBSERVE
 ↓
LIMITED
 ↓
ENFORCE
90. Governance Control Plane Events

Catálogo mínimo:

PolicyCreated
PolicyUpdated
PolicyApproved
PolicyActivated
PolicySuspended
PolicyRetired

RuleCreated
RuleUpdated

DecisionIssued
DecisionExpired

ExceptionCreated
ExceptionApproved
ExceptionExpired
ExceptionRevoked

CertificationGranted
CertificationExpired
CertificationRevoked

AssetRegistered
AssetUpdated
AssetRetired

GovernanceDriftDetected
GovernanceRemediated
91. Event Versioning

Los eventos deben tener:

eventType
eventVersion
eventId
occurredAt
producer
correlationId
payload
92. Event Compatibility

Los consumidores no deben romperse automáticamente cuando evoluciona un evento.

Debe existir:

backward compatibility
versioning
schema validation
93. Governance Data Model

El núcleo relacional puede modelarse como:

policy
policy_version
rule
rule_version
governance_asset
governance_domain
governance_decision
governance_workflow
governance_approval
governance_exception
governance_certification
governance_evidence
governance_audit
governance_event
94. Policy Relationships
Policy
  │
  ├── versions
  ├── contains rules
  ├── governs assets
  ├── produces decisions
  └── generates evidence
95. Asset Relationships
Asset
 │
 ├── owned-by
 ├── classified-as
 ├── governed-by
 ├── certified-by
 ├── affected-by
 └── connected-to
96. Governance Graph Query

Preguntas posibles:

Which policies govern this asset?

Which assets are affected by this policy?

Who owns all assets affected by this rule?

Which consumers depend on a non-compliant asset?
97. Evidence Retrieval

Una auditoría debe poder reconstruir:

What happened?
Who did it?
Which policy applied?
Which version?
Which rules?
What decision?
What evidence?
What action followed?
98. Governance Audit Reconstruction
Correlation ID
      ↓
Request
      ↓
Decision
      ↓
Policy Version
      ↓
Approval
      ↓
Action
      ↓
Evidence
99. Disaster Recovery

Debe poder recuperarse:

Policy State
Rule State
Decision Records
Workflow State
Exceptions
Certifications
Evidence
Audit
100. Recovery Invariant

Después de una recuperación:

No critical governance decision
may become unverifiable.
101. Implementation Layers

El Control Plane queda dividido en:

Layer 1 — Interface
Layer 2 — Context
Layer 3 — Policy
Layer 4 — Rules
Layer 5 — Decision
Layer 6 — State
Layer 7 — Workflow
Layer 8 — Evidence
Layer 9 — Audit
Layer 10 — Events
Layer 11 — Persistence
Layer 12 — Integration
102. Reference Architecture
                         ┌───────────────────────────┐
                         │     GOVERNANCE CLIENTS    │
                         └─────────────┬─────────────┘
                                       │
                                       ▼
                         ┌───────────────────────────┐
                         │    GOVERNANCE GATEWAY     │
                         └─────────────┬─────────────┘
                                       │
                                       ▼
                         ┌───────────────────────────┐
                         │ GOVERNANCE CONTEXT SERVICE│
                         └─────────────┬─────────────┘
                                       │
             ┌─────────────────────────┼─────────────────────────┐
             ▼                         ▼                         ▼
      ┌─────────────┐           ┌─────────────┐           ┌─────────────┐
      │Policy Engine│           │ Rules Engine│           │Decision     │
      │             │           │             │           │Engine       │
      └──────┬──────┘           └──────┬──────┘           └──────┬──────┘
             └─────────────────────────┼─────────────────────────┘
                                       ▼
                           ┌───────────────────────┐
                           │ Governance State      │
                           │ Manager               │
                           └───────────┬───────────┘
                                       │
                ┌──────────────────────┼──────────────────────┐
                ▼                      ▼                      ▼
         ┌────────────┐        ┌─────────────┐        ┌─────────────┐
         │ Workflow   │        │ Certification│       │ Exception   │
         │ Coordinator│        │ Service      │       │ Service     │
         └──────┬─────┘        └──────┬──────┘        └──────┬──────┘
                └─────────────────────┼──────────────────────┘
                                      ▼
                         ┌──────────────────────────┐
                         │ Governance Repository    │
                         └────────────┬─────────────┘
                                      │
                    ┌─────────────────┼─────────────────┐
                    ▼                 ▼                 ▼
              Evidence Store     Audit Store       Event Outbox
                                                        │
                                                        ▼
                                                   Event Bus
103. Integration With EVOXA

E62 debe integrarse con:

E03 API Architecture
E04 Authentication Architecture
E05 Authorization Architecture
E06 Policy Architecture
E08 Domain Services
E09 Application Services
E10 Repository Architecture
E12 Messaging Architecture
E13 Event Processing
E14 Workflow & Orchestration
E15 Job & Task Processing
E16 Scheduling
E17 Caching
E18 Configuration
E19 Feature Flags
E20 Runtime Policy
E21 Rules Engine
E22 Validation
E26 Projection
E27 Query
E28 Read Models
E29 Search
E31 Analytics
E38 Resilience
E39 Fault Tolerance
E40 Recovery
E43 Data Integrity
E44 Consistency
E55 Data Access Governance
E56 Data Access Security
E57 Data Protection
E58 Data Privacy
E59 Data Governance
E60 Governance Operating Model
E61 Governance Implementation
104. Architectural Dependency

La dependencia principal queda:

E59
Governance Architecture
        ↓
E60
Operating Model
        ↓
E61
Implementation Architecture
        ↓
E62
Control Plane
        ↓
E63
Enforcement
105. E62 Acceptance Criteria

E62 está correctamente implementado cuando:

✓ Existe un Governance Control Plane explícito
✓ Policies están versionadas
✓ Rules están versionadas
✓ Existe resolución de políticas
✓ Existe evaluación de reglas
✓ Existe Decision Engine
✓ Las decisiones son explicables
✓ Existe Governance Context
✓ Existe estado gobernado
✓ Existen workflows
✓ Existen approvals
✓ Existen exceptions
✓ Existen certifications
✓ Existe Evidence Service
✓ Existe Audit Service
✓ Existe Event Bus integration
✓ Existe idempotencia
✓ Existe control de concurrencia
✓ Existe reconciliación
✓ Existe drift detection
✓ Existe observabilidad
✓ Existe fail-safe behavior
✓ Existe integración con Data Plane
✓ Las decisiones críticas son reconstruibles
106. Principio Rector

El EVOXA Data Governance Control Plane es la autoridad técnica que transforma políticas y reglas de gobierno en decisiones deterministas, estados gobernados, workflows, evidencias y eventos auditables, manteniendo una separación explícita entre la lógica de control y la ejecución sobre el Data Plane.

La cadena queda ahora:

E59 — DATA GOVERNANCE ARCHITECTURE
              ↓
E60 — DATA GOVERNANCE OPERATING MODEL
              ↓
E61 — DATA GOVERNANCE IMPLEMENTATION ARCHITECTURE
              ↓
E62 — DATA GOVERNANCE CONTROL PLANE ARCHITECTURE
              ↓
E63 — DATA GOVERNANCE ENFORCEMENT ARCHITECTURE

E62 define el cerebro de control.
E63 deberá definir cómo las decisiones del Control Plane se hacen cumplir efectivamente sobre APIs, datos, servicios, pipelines, eventos, aplicaciones y runtime.
