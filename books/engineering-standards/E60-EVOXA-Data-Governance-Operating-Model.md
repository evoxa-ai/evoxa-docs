E60 — EVOXA Data Governance Operating Model
Engineering Specification

Pertenece a: Engineering Specification
Capítulo: E60 — Data Governance Operating Model
Anterior: E59 — Data Governance Architecture
Siguiente: E61 — Data Governance Implementation Architecture

1. Propósito

E60 define cómo EVOXA opera, mantiene, supervisa y evoluciona el gobierno de datos en la práctica.

E59 estableció la arquitectura de gobierno:

ownership
policies
classification
quality
lineage
lifecycle
certification

E60 establece el mecanismo operativo:

roles
responsibilities
decision rights
processes
workflows
cadences
committees
escalation
approvals
reviews
evidence
metrics
continuous improvement

La diferencia fundamental es:

E59
¿Qué debe gobernarse?

E60
¿Cómo se gobierna continuamente?
2. Objetivo Arquitectónico

El modelo operativo debe transformar Data Governance de una colección de políticas en un sistema operativo permanente de accountability.

Governance Policy
       ↓
Operating Process
       ↓
Assigned Responsibility
       ↓
Workflow
       ↓
Decision
       ↓
Evidence
       ↓
Measurement
       ↓
Improvement
3. Scope

E60 cubre:

Governance Operating Model
Governance Organization
Governance Roles
Decision Rights
RACI
Governance Forums
Governance Committees
Data Owners
Data Stewards
Data Custodians
Data Consumers
Governance Processes
Approval Workflows
Data Issue Management
Data Quality Operations
Metadata Operations
Classification Operations
Data Certification
Data Contract Governance
Data Change Governance
Data Sharing Governance
Data Lifecycle Governance
Exception Management
Escalation
Governance Cadence
Governance KPIs
Governance Reporting
Governance Evidence
Governance Audit Support
Continuous Improvement
4. Non-Goals

E60 no reemplaza:

E59 — Data Governance Architecture
E58 — Data Privacy
E57 — Data Protection
E56 — Data Access Security
E55 — Data Access Governance
E45–E48 — Data Lifecycle / Retention / Disposal / Archival

E60 define cómo estas capacidades son operadas organizacionalmente.

5. Operating Model Principle

El gobierno debe ser:

Distributed
but
Federated

Es decir:

Central Governance
        │
        ├───────────────┐
        ▼               ▼
 Domain A           Domain B
 Stewardship        Stewardship
        │               │
        └───────┬───────┘
                ▼
        Shared Standards

El gobierno central establece coherencia.

Los dominios mantienen accountability sobre sus datos.

6. Target Operating Model
                    ┌─────────────────────┐
                    │ Governance Authority│
                    └──────────┬──────────┘
                               │
                    ┌──────────▼──────────┐
                    │ Data Governance      │
                    │ Office / Function    │
                    └──────────┬──────────┘
                               │
          ┌────────────────────┼────────────────────┐
          ▼                    ▼                    ▼
   Domain Governance     Platform Governance   Security/Privacy
          │                    │                    │
          ▼                    ▼                    ▼
     Data Owners         Custodians           Specialists
          │
          ▼
     Data Stewards
          │
          ▼
     Data Consumers
7. Governance Organizational Layers

EVOXA debe operar governance en cuatro niveles:

Level 1 — Enterprise
Level 2 — Domain
Level 3 — Asset / Product
Level 4 — Operational / Runtime
8. Level 1 — Enterprise Governance

Responsable de:

enterprise standards
global policies
cross-domain conflicts
risk posture
governance strategy
critical decisions
9. Level 2 — Domain Governance

Responsable de:

domain definitions
domain ownership
quality
classification
data products
domain policies
domain issues
10. Level 3 — Asset / Product Governance

Responsable de:

specific datasets
data products
schemas
contracts
quality rules
lineage
certification
consumer relationships
11. Level 4 — Operational Governance

Responsable de:

runtime policy enforcement
quality monitoring
contract violations
data incidents
metadata synchronization
automated controls
12. Governance Authority

Debe existir una autoridad formal para resolver cuestiones que atraviesan dominios.

Responsabilidades:

approve global policies
resolve conflicts
define mandatory standards
approve exceptions
review governance health
13. Data Governance Office

El Data Governance Office —o función equivalente— actúa como mecanismo coordinador.

Funciones:

framework maintenance
process ownership
governance metrics
training
standards
coordination
audit support
issue escalation

No debe convertirse en propietario operativo de todos los datos.

14. Data Owner

El Data Owner posee accountability empresarial sobre un activo o dominio.

Responsabilidades:

approve definitions
approve classification
define acceptable quality
approve usage
approve sharing
approve lifecycle
resolve escalated issues
15. Data Steward

El Steward ejecuta el gobierno operativo.

Responsabilidades:

maintain metadata
monitor quality
coordinate issues
maintain glossary
support certification
coordinate lineage
prepare reviews
16. Data Custodian

El Custodian mantiene la infraestructura técnica.

Responsabilidades:

storage
pipelines
technical controls
backup
availability
technical metadata
runtime enforcement
17. Data Consumer

El Consumer:

requests access
declares purpose
respects usage constraints
reports data issues
respects contracts
18. Governance Role Separation

Debe evitarse:

Owner = Steward = Custodian

cuando esto genere conflictos de interés.

Modelo preferido:

Owner
  ↓ accountability

Steward
  ↓ governance operations

Custodian
  ↓ technical operation
19. RACI Model

Cada proceso crítico debe tener:

Responsible
Accountable
Consulted
Informed

Ejemplo:

Proceso	Owner	Steward	Custodian	Governance
Definition	A	R	C	C
Classification	A	R	C	C
Quality	A	R	R	C
Schema Change	A	R	R	C
Sharing	A	R	C	C
Certification	A	R	C	R
Policy	C	C	C	A/R
20. Decision Rights

El operating model debe responder explícitamente:

Who decides?
Who executes?
Who approves?
Who can override?
Who escalates?

No debe existir una decisión crítica basada únicamente en "consenso informal".

21. Governance Decision Matrix
Decision
   ↓
Domain?
   ├── YES → Domain Owner
   └── NO  → Enterprise Governance

Para conflictos:

Domain Decision
      ↓
Conflict
      ↓
Governance Authority
22. Governance Forums

Se recomiendan tres tipos:

Strategic Forum
Operational Forum
Technical Forum
23. Strategic Governance Forum

Responsable de:

strategy
risk
major exceptions
cross-domain conflicts
priority
governance maturity

Cadencia típica:

monthly / quarterly
24. Operational Governance Forum

Responsable de:

data quality
open issues
certification
metadata
sharing requests
lifecycle
upcoming changes

Cadencia:

weekly / biweekly
25. Technical Governance Forum

Responsable de:

schemas
contracts
lineage
pipelines
platform controls
integration
technical standards

Cadencia:

weekly
26. Governance Meeting Principle

Una reunión de governance debe producir:

decision
owner
action
deadline
evidence

No simplemente discusión.

27. Governance Workflow

Modelo general:

Request
  ↓
Triage
  ↓
Validation
  ↓
Impact Analysis
  ↓
Decision
  ↓
Approval
  ↓
Execution
  ↓
Verification
  ↓
Evidence
  ↓
Closure
28. Governance Request

Todo workflow debe comenzar con una solicitud estructurada.

GovernanceRequest
├── requestId
├── requester
├── asset
├── domain
├── purpose
├── requestedChange
├── risk
├── evidence
└── status
29. Request Triage

El triage determina:

scope
severity
risk
owner
workflow
required approvals
30. Governance Workflow Classes

EVOXA debe distinguir:

Access Request
Classification Request
Definition Change
Schema Change
Data Sharing
Data Contract Change
Quality Exception
Lifecycle Change
Certification
Policy Exception
Data Issue
31. Approval Matrix

Cada workflow debe tener requisitos claros:

Low Risk
   → Owner

Medium Risk
   → Owner + Steward

High Risk
   → Owner + Governance

Critical
   → Governance Authority

La clasificación concreta de riesgo debe configurarse mediante política.

32. Approval Evidence

Toda aprobación debe registrar:

actor
role
decision
timestamp
reason
policy version
request version
33. Governance State Machine
DRAFT
  ↓
SUBMITTED
  ↓
TRIAGED
  ↓
UNDER_REVIEW
  ↓
APPROVED
  ↓
EXECUTING
  ↓
VERIFIED
  ↓
CLOSED

Estados alternativos:

REJECTED
CANCELLED
EXPIRED
ESCALATED
34. Data Issue Operating Process
Detect
  ↓
Register
  ↓
Classify
  ↓
Assign
  ↓
Investigate
  ↓
Remediate
  ↓
Validate
  ↓
Close
35. Data Issue Severity

Modelo:

P1 — Critical
P2 — High
P3 — Medium
P4 — Low

La prioridad debe considerar:

business impact
regulatory impact
security impact
privacy impact
consumer impact
data criticality
36. Critical Data Issue

Debe activar:

immediate owner notification
incident workflow
impact assessment
containment
remediation
executive escalation

cuando corresponda.

37. Data Quality Operating Model
Rules
 ↓
Monitoring
 ↓
Detection
 ↓
Issue
 ↓
Root Cause
 ↓
Remediation
 ↓
Validation
 ↓
Trend Analysis
38. Quality Ownership

El equipo técnico puede operar la medición.

Pero:

Data Owner
   ↓
defines acceptable quality

El Steward coordina.

39. Quality Review

Debe revisarse periódicamente:

quality score
failed rules
trend
top issues
root causes
SLA breaches
consumer impact
40. Metadata Operating Process
Discover
 ↓
Register
 ↓
Validate
 ↓
Publish
 ↓
Maintain
 ↓
Review
 ↓
Retire
41. Metadata Stewardship

El Steward debe garantizar que metadata crítica no quede obsoleta.

Indicadores:

metadata freshness
metadata completeness
metadata accuracy
42. Classification Operating Process
Identify
 ↓
Classify
 ↓
Review
 ↓
Approve
 ↓
Enforce
 ↓
Reassess
43. Classification Reassessment

Debe activarse ante:

new data type
new business use
new regulation
new consumer
schema change
data aggregation
44. Data Certification Process
Candidate
   ↓
Metadata Check
   ↓
Ownership Check
   ↓
Quality Check
   ↓
Lineage Check
   ↓
Policy Check
   ↓
Certification
45. Certification Review

Las certificaciones deben tener cadencia.

CERTIFIED
    ↓
REVIEW_DUE
    ↓
REVALIDATE
    ↓
CERTIFIED / REVOKED
46. Data Contract Operating Process
Proposal
 ↓
Impact Analysis
 ↓
Compatibility Check
 ↓
Consumer Review
 ↓
Approval
 ↓
Migration
 ↓
Deployment
 ↓
Verification
47. Breaking Change

Un breaking change debe exigir:

consumer identification
migration plan
deprecation period
owner approval
verification
48. Data Sharing Process
Request
 ↓
Purpose Validation
 ↓
Data Classification
 ↓
Privacy Review
 ↓
Security Review
 ↓
Owner Approval
 ↓
Technical Enablement
 ↓
Monitoring
49. Lifecycle Governance Process
Create
 ↓
Activate
 ↓
Review
 ↓
Deprecate
 ↓
Retire
 ↓
Archive / Dispose

Cada transición debe tener:

owner
criteria
approval
evidence
50. Exception Management

No debe existir una excepción informal.

Exception Request
 ↓
Risk Assessment
 ↓
Owner Approval
 ↓
Governance Approval
 ↓
Expiration
 ↓
Review
51. Exception Record
GovernanceException
├── exceptionId
├── policy
├── asset
├── reason
├── risk
├── compensatingControls
├── approver
├── createdAt
├── expiresAt
└── status
52. Exception Expiration

Debe evitarse:

temporary exception
        ↓
permanent undocumented behavior

Por ello:

expiresAt

es obligatorio para excepciones temporales.

53. Escalation Model
Steward
   ↓
Data Owner
   ↓
Domain Governance
   ↓
Enterprise Governance
   ↓
Executive Authority

Solo debe escalarse cuando el nivel inferior no pueda resolver.

54. Escalation Triggers
SLA breach
cross-domain conflict
regulatory concern
critical quality failure
security concern
privacy concern
unresolved ownership
policy conflict
55. Governance SLA

Los procesos deben tener tiempos objetivo.

Ejemplo:

Proceso	SLA conceptual
Critical issue	Immediate
High issue	Short
Standard request	Defined business SLA
Certification	Defined review SLA
Exception	Defined approval SLA

Los valores exactos deben establecerse por política operativa.

56. Governance Cadence

El modelo debe tener una cadencia explícita:

Continuous
Daily
Weekly
Monthly
Quarterly
Annual
57. Continuous

Automatizado:

quality monitoring
contract monitoring
classification controls
policy enforcement
metadata synchronization
58. Daily
critical issues
quality failures
policy violations
pipeline failures
governance alerts
59. Weekly
open issues
quality trends
contract changes
metadata problems
certification work
60. Monthly
domain governance
KPIs
exceptions
certifications
quality posture
cross-domain issues
61. Quarterly
governance maturity
policy review
ownership review
critical asset review
risk assessment
strategic priorities
62. Annual
governance framework review
role review
policy baseline
standards refresh
operating model assessment
training
63. Governance Calendar

EVOXA debe mantener un calendario operativo:

Governance Calendar
├── Reviews
├── Certifications
├── Policy expirations
├── Exception expirations
├── Contract deadlines
├── Lifecycle milestones
└── Audits
64. Governance Work Queue

Las actividades deben centralizarse en una cola:

Governance Work
├── Issues
├── Requests
├── Reviews
├── Approvals
├── Exceptions
├── Certifications
└── Changes
65. Governance Prioritization

La prioridad debe considerar:

business criticality
risk
regulatory exposure
consumer impact
data sensitivity
deadline
66. Governance Backlog

Debe distinguirse:

urgent remediation
mandatory compliance
quality improvement
metadata debt
technical debt
strategic improvement
67. Governance Debt

Los gaps acumulados constituyen deuda de governance:

missing owner
missing lineage
missing definition
stale metadata
unresolved quality issues
expired certification

Debe medirse explícitamente.

68. Governance Debt Management
Identify
 ↓
Quantify
 ↓
Prioritize
 ↓
Remediate
 ↓
Measure Reduction
69. Governance Metrics

KPIs principales:

Ownership Coverage
Metadata Coverage
Classification Coverage
Lineage Coverage
Certification Coverage
Quality Compliance
Issue Resolution Time
Exception Aging
Policy Compliance
Contract Compliance
70. Ownership Coverage
Assets With Owner
──────────────────
Governed Assets
71. Issue Resolution Time
ResolvedAt - DetectedAt

Debe medirse por severidad.

72. Exception Aging

Debe detectarse:

active exceptions
near-expiry exceptions
expired exceptions
long-running exceptions
73. Governance Health Score

Puede agregarse:

Governance Health
 =
Ownership
+ Metadata
+ Quality
+ Lineage
+ Certification
+ Compliance

No debe ocultar métricas individuales.

74. Governance Reporting

Un reporte ejecutivo debe responder:

Are we governed?
Where are the risks?
What is degrading?
What decisions are blocked?
What requires leadership?
75. Domain Governance Dashboard

Cada dominio debería observar:

assets
owners
quality
issues
certification
lineage
contracts
exceptions
76. Operational Governance Dashboard

Debe mostrar:

open work
critical alerts
SLA breaches
policy violations
quality failures
pending approvals
77. Governance Audit Support

El operating model debe poder entregar:

policy
owner
approval
decision
evidence
execution
verification

para cualquier control relevante.

78. Audit Request Workflow
Audit Request
 ↓
Scope
 ↓
Evidence Collection
 ↓
Validation
 ↓
Response
 ↓
Finding
 ↓
Remediation
79. Evidence Retention

La evidencia de governance debe conservarse conforme a:

retention policy
regulatory requirements
audit requirements
business requirements
80. Governance Training

Todos los roles deben recibir formación adecuada.

Owner
 → accountability

Steward
 → operational governance

Custodian
 → technical controls

Consumer
 → usage obligations
81. Governance Onboarding

Un nuevo dominio debe pasar por:

Domain Registration
 ↓
Owner Assignment
 ↓
Steward Assignment
 ↓
Asset Inventory
 ↓
Classification
 ↓
Policies
 ↓
Quality
 ↓
Lineage
 ↓
Certification
82. New Data Asset Onboarding
Proposal
 ↓
Owner
 ↓
Definition
 ↓
Classification
 ↓
Policy
 ↓
Quality
 ↓
Lineage
 ↓
Lifecycle
 ↓
Publish
83. New Data Product Onboarding

Debe añadir:

contract
consumers
SLA
support model
versioning
84. Governance Offboarding

Cuando un activo se retira:

Consumer Notification
 ↓
Deprecation
 ↓
Access Review
 ↓
Archival / Disposal
 ↓
Catalog Update
 ↓
Certification Revocation
85. Change Management

Governance debe integrarse con change management.

Change
 ↓
Governance Impact
 ↓
Approval
 ↓
Implementation
 ↓
Validation
86. Governance Automation

La automatización debe cubrir progresivamente:

metadata collection
classification detection
quality monitoring
lineage extraction
policy validation
contract checks
certification checks
expiration detection
87. Human-in-the-Loop

La automatización no elimina la responsabilidad humana en decisiones de alto impacto.

Automated Detection
       ↓
Human Review
       ↓
Decision
88. Governance as Code

Cuando sea viable:

Policy
 ↓
Machine-readable representation
 ↓
Automated validation
 ↓
CI/CD
 ↓
Runtime enforcement
89. CI/CD Governance

Los cambios de datos pueden incorporar:

schema compatibility
policy validation
classification validation
contract tests
quality tests
migration checks

antes de deployment.

90. Preventive Governance

La mejor intervención ocurre antes del problema.

Design
 ↓
Governance Validation
 ↓
Implementation

en lugar de:

Production
 ↓
Problem
 ↓
Governance Investigation
91. Detective Governance

Cuando la prevención no es suficiente:

Runtime Monitoring
 ↓
Violation
 ↓
Alert
 ↓
Remediation
92. Corrective Governance
Finding
 ↓
Root Cause
 ↓
Corrective Action
 ↓
Verification
93. Governance Control Loop
        ┌───────────────┐
        │    Govern     │
        └───────┬───────┘
                ↓
        ┌───────────────┐
        │   Operate     │
        └───────┬───────┘
                ↓
        ┌───────────────┐
        │   Measure     │
        └───────┬───────┘
                ↓
        ┌───────────────┐
        │    Improve    │
        └───────┬───────┘
                │
                └──────────────→ Govern
94. Operating Model Maturity
Level 0
Ad hoc

Level 1
Defined roles

Level 2
Documented processes

Level 3
Measured governance

Level 4
Automated controls

Level 5
Adaptive continuous governance
95. Level 0 — Ad Hoc
ownership unclear
manual decisions
informal approvals
limited evidence
96. Level 1 — Defined
roles exist
owners assigned
basic policies defined
97. Level 2 — Managed
workflows
cadences
KPIs
escalation
evidence
98. Level 3 — Measured
quality metrics
coverage metrics
SLA metrics
risk metrics
99. Level 4 — Automated
policy-as-code
automated validation
automated classification
automated quality
automated expiration
100. Level 5 — Adaptive
Continuous Signals
        ↓
Risk Detection
        ↓
Governance Adjustment
        ↓
Automated Control
        ↓
Feedback
101. Governance Operating Principles
Principle 1

Every critical data asset has accountability.

Principle 2

Decision rights are explicit.

Principle 3

Governance work is workflow-driven.

Principle 4

Exceptions are temporary unless explicitly renewed.

Principle 5

Governance evidence is part of the operation, not an afterthought.

Principle 6

Domain ownership and enterprise standards coexist.

Principle 7

Critical governance controls should be automated.

Principle 8

Human approval remains available for high-impact decisions.

Principle 9

Governance metrics measure outcomes, not meeting volume.

Principle 10

Every governance process has an owner, SLA and escalation path.

102. Governance Operating Model Reference
                         GOVERNANCE AUTHORITY
                                  │
                                  ▼
                       ┌────────────────────┐
                       │ Governance Office  │
                       └─────────┬──────────┘
                                 │
             ┌───────────────────┼───────────────────┐
             ▼                   ▼                   ▼
       Enterprise          Domain Governance   Technical Governance
             │                   │                   │
             ▼                   ▼                   ▼
         Policies             Owners              Custodians
             │                   │                   │
             │                   ▼                   │
             │               Stewards               │
             │                   │                   │
             └───────────────────┼───────────────────┘
                                 ▼
                        Governance Workflows
                                 │
       ┌──────────────┬──────────┼───────────┬──────────────┐
       ▼              ▼          ▼           ▼              ▼
    Quality       Metadata   Contracts   Lifecycle      Sharing
       │              │          │           │              │
       └──────────────┴──────────┼───────────┴──────────────┘
                                 ▼
                           Measurements
                                 │
                                 ▼
                              Evidence
                                 │
                                 ▼
                           Improvement
103. Critical Operating Invariants
Invariant 1

Every governance process has an accountable owner.

Invariant 2

Every critical decision has explicit decision rights.

Invariant 3

Every approval produces evidence.

Invariant 4

Every exception has an expiration or explicit permanent status.

Invariant 5

Every critical issue has an escalation path.

Invariant 6

Every critical asset participates in a governance cadence.

Invariant 7

Governance KPIs are measurable.

Invariant 8

Governance processes are reproducible.

Invariant 9

Governance decisions are auditable.

Invariant 10

Governance responsibilities remain distinct from technical custody where separation is required.

104. Completion Criteria

E60 se considera completo cuando EVOXA dispone de:

✓ Governance operating model
✓ Governance organizational layers
✓ Governance authority
✓ Governance office/function
✓ Data owners
✓ Data stewards
✓ Data custodians
✓ Data consumers
✓ Decision rights
✓ RACI
✓ Governance forums
✓ Strategic governance
✓ Operational governance
✓ Technical governance
✓ Governance workflows
✓ Request management
✓ Triage
✓ Approval model
✓ Approval evidence
✓ Data issue process
✓ Quality operating process
✓ Metadata operating process
✓ Classification process
✓ Certification process
✓ Contract governance
✓ Sharing governance
✓ Lifecycle governance
✓ Exception management
✓ Escalation model
✓ Governance SLAs
✓ Governance cadence
✓ Governance calendar
✓ Governance work queue
✓ Governance backlog
✓ Governance debt management
✓ Governance KPIs
✓ Governance dashboards
✓ Audit support
✓ Evidence management
✓ Training
✓ Onboarding
✓ Offboarding
✓ Change management
✓ Governance automation
✓ Governance-as-code
✓ CI/CD governance
✓ Preventive controls
✓ Detective controls
✓ Corrective controls
✓ Maturity model
✓ Continuous improvement
105. Principio Rector de E60

EVOXA Data Governance Operating Model convierte la arquitectura de gobierno en una capacidad operativa permanente, distribuyendo accountability entre autoridad empresarial, dominios, owners, stewards y custodios; formalizando decisiones mediante workflows, aprobaciones, SLAs, escalaciones y evidencia; y cerrando continuamente el ciclo entre política, operación, medición, remediación y mejora.

La evolución queda:

E57 — DATA PROTECTION
        ↓
E58 — DATA PRIVACY
        ↓
E59 — DATA GOVERNANCE ARCHITECTURE
        ↓
E60 — DATA GOVERNANCE OPERATING MODEL
        ↓
E61 — DATA GOVERNANCE IMPLEMENTATION ARCHITECTURE

E59 define el sistema de gobierno.
E60 define cómo funciona organizacionalmente.
E61 deberá definir cómo ese modelo se materializa técnicamente en servicios, componentes, repositorios, automatizaciones, workflows y controles ejecutables de EVOXA.
