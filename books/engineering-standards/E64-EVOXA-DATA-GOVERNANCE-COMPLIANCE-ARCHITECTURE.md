E64 — EVOXA DATA GOVERNANCE COMPLIANCE ARCHITECTURE
Engineering Specification

Pertenece a: Engineering Specification
Capítulo: E64
Anterior: E63 — Data Governance Enforcement Architecture
Siguiente: E65 — Data Governance Monitoring & Assurance Architecture

1. Propósito

E64 define la arquitectura mediante la cual EVOXA determina, mide, demuestra y mantiene el cumplimiento de Data Governance.

E62 estableció:

Policy
   ↓
Decision

E63 estableció:

Decision
   ↓
Enforcement
   ↓
Execution

E64 añade:

Enforcement
   ↓
Compliance Evaluation
   ↓
Finding
   ↓
Evidence
   ↓
Remediation
   ↓
Verification

El objetivo es que EVOXA pueda responder de forma determinista:

¿Se está cumpliendo la política?
¿Qué activos cumplen?
¿Cuáles no?
¿Por qué?
Desde cuándo?
Quién es responsable?
Qué evidencia lo demuestra?
Qué riesgo existe?
Qué remediación está pendiente?
2. Principio Fundamental

Compliance no significa simplemente que una operación haya sido permitida; significa que EVOXA puede demostrar que las obligaciones de governance aplicables se cumplen de manera verificable y auditable.

Por tanto:

Policy
   ↓
Requirement
   ↓
Control
   ↓
Evidence
   ↓
Assessment
   ↓
Compliance Status
3. Compliance Control Plane

E64 extiende el Control Plane definido en E62.

                 GOVERNANCE CONTROL PLANE
                          │
          ┌───────────────┼────────────────┐
          ▼               ▼                ▼
       Policy          Decision         Enforcement
          │               │                │
          └───────────────┼────────────────┘
                          ▼
                  Compliance Engine
                          │
             ┌────────────┼────────────┐
             ▼            ▼            ▼
        Assessment      Findings     Evidence
             │            │            │
             └────────────┼────────────┘
                          ▼
                    Remediation
                          │
                          ▼
                    Verification
4. Compliance vs Governance

Debe existir una separación conceptual:

Governance
→ define cómo deben gestionarse los datos.

Compliance
→ determina si esas obligaciones se están cumpliendo.

Por tanto:

Governance = normative
Compliance = evaluative
5. Compliance vs Enforcement

E63 responde:

"¿Qué hacemos cuando ocurre esta operación?"

E64 responde:

"¿Estamos cumpliendo de manera sostenida?"

Ejemplo:

Policy
  ↓
DENY unauthorized access
  ↓
E63 blocks access

E64:
  ↓
Are unauthorized attempts occurring?
  ↓
Are controls working?
  ↓
Is evidence complete?
6. Compliance Domains

El Compliance Architecture debe poder evaluar:

Access Governance
Data Classification
Data Quality
Data Protection
Data Privacy
Data Retention
Data Lifecycle
Data Ownership
Data Stewardship
Data Contracts
Data Lineage
Data Security
Data Usage
Data Sharing
Data Export
Data Residency

según las políticas aplicables a cada dominio.

7. Compliance Model

El modelo conceptual es:

Requirement
    ↓
Control
    ↓
Control Implementation
    ↓
Evidence
    ↓
Assessment
    ↓
Finding
    ↓
Compliance Status
8. Compliance Requirement

Un requirement expresa qué debe cumplirse.

Ejemplo conceptual:

Requirement:
Every governed dataset must have an accountable owner.

Debe poseer identidad estable:

requirementId
requirementVersion
9. Requirement Types
Mandatory
Conditional
Advisory
Risk-Based
Temporal
Operational
10. Compliance Control

Un control representa el mecanismo utilizado para cumplir un requirement.

Requirement
     ↓
Control
     ↓
Implementation

Ejemplo:

Requirement:
Dataset must have an owner.

Control:
Ownership validation.
11. Control Identity
controlId
controlVersion
controlType
controlStatus

La versión debe quedar registrada en cada assessment relevante.

12. Control Types
Preventive
Detective
Corrective
Automated
Manual
Hybrid
13. Automated Controls

Ejemplo:

IF owner IS NULL
THEN compliance = FAIL
14. Manual Controls

Algunos controles pueden requerir revisión humana:

Policy Review
Business Approval
Certification Review
Risk Acceptance
Exception Review
15. Hybrid Controls
Automated Check
       ↓
Human Review
       ↓
Final Assessment
16. Compliance Assessment

Un assessment evalúa el estado de cumplimiento.

ComplianceAssessment
{
    assessmentId,
    subject,
    requirement,
    control,
    status,
    evidence,
    assessor,
    assessedAt,
    validUntil
}
17. Compliance Status

Estados mínimos:

COMPLIANT
NON_COMPLIANT
PARTIALLY_COMPLIANT
NOT_APPLICABLE
UNKNOWN
PENDING_REVIEW
18. COMPLIANT

Indica que existe evidencia suficiente de cumplimiento.

No significa que el activo sea perfecto.

Significa que satisface el requirement evaluado bajo las condiciones definidas.

19. NON_COMPLIANT

Existe evidencia suficiente de que el requirement no se cumple.

Debe generar:

Finding

cuando corresponda.

20. PARTIALLY_COMPLIANT

Se utiliza cuando el control cumple solo parcialmente.

Ejemplo:

100 datasets
80 compliant
20 non-compliant

El resultado agregado puede ser:

PARTIALLY_COMPLIANT

según la política de evaluación.

21. UNKNOWN

Debe utilizarse cuando no existe evidencia suficiente.

Importante:

UNKNOWN
≠
COMPLIANT

La ausencia de evidencia no debe convertirse automáticamente en cumplimiento.

22. NOT_APPLICABLE

Un requirement puede no aplicar a un activo.

Debe existir una justificación o regla que explique por qué.

23. Compliance Scope

La evaluación puede aplicarse a:

Tenant
Organization
Domain
Data Product
Dataset
Table
Column
Pipeline
Application
Service
API
Workflow
Event Stream
Storage
24. Compliance Hierarchy
Organization
      ↓
Domain
      ↓
Data Product
      ↓
Dataset
      ↓
Table
      ↓
Column

Un estado agregado debe poder derivarse de estados inferiores.

25. Compliance Inheritance

Ejemplo:

Domain = NON_COMPLIANT

no implica automáticamente:

Every Asset = NON_COMPLIANT

La herencia debe estar explícitamente definida por el requirement.

26. Applicability Engine

Antes de evaluar un control:

Asset
 ↓
Applicability Rules
 ↓
Applicable Requirements

Esto evita evaluar controles irrelevantes.

27. Applicability Factors
classification
domain
asset type
data sensitivity
jurisdiction
purpose
lifecycle state
business criticality
28. Compliance Evaluation Pipeline
Asset
 ↓
Applicability
 ↓
Requirements
 ↓
Controls
 ↓
Evidence Collection
 ↓
Assessment
 ↓
Finding
 ↓
Status
29. Evidence-Driven Compliance

El estado debe fundamentarse en evidencia.

Control
 ↓
Evidence
 ↓
Assessment

No simplemente:

Control
 ↓
ASSUME COMPLIANT
30. Evidence Types
Policy Evidence
Configuration Evidence
Runtime Evidence
Access Evidence
Audit Evidence
Quality Evidence
Certification Evidence
Approval Evidence
Execution Evidence
Monitoring Evidence
31. Evidence Freshness

La evidencia puede caducar.

capturedAt
validFrom
validUntil

Una evidencia demasiado antigua puede dejar de demostrar compliance.

32. Evidence Integrity

La evidencia crítica debe ser:

Attributable
Timestamped
Versioned
Tamper-Evident
Traceable
33. Evidence Provenance

Debe poder responderse:

Where did this evidence come from?
When was it collected?
Which system produced it?
Which control generated it?
Which policy version applied?
34. Evidence Chain
Requirement
 ↓
Control
 ↓
Evidence
 ↓
Assessment
 ↓
Finding
 ↓
Remediation
35. Compliance Finding

Cuando un control falla:

Compliance Finding

debe contener:

findingId
requirementId
controlId
subject
severity
description
evidence
detectedAt
owner
status
36. Finding Severity
CRITICAL
HIGH
MEDIUM
LOW
INFORMATIONAL

La clasificación exacta debe derivarse de riesgo y policy.

37. Finding Lifecycle
DETECTED
 ↓
TRIAGED
 ↓
ASSIGNED
 ↓
REMEDIATION
 ↓
VERIFICATION
 ↓
RESOLVED

También:

FALSE_POSITIVE
ACCEPTED_RISK
WAIVED

cuando estén autorizados.

38. Finding Ownership

Todo finding accionable debe tener:

owner
domain
dueDate
priority
39. Compliance Remediation
Finding
 ↓
Remediation Plan
 ↓
Action
 ↓
Verification
 ↓
Closure
40. Remediation Types
Automatic
Semi-Automatic
Manual
Preventive
Corrective
Compensating
41. Automated Remediation

Ejemplo:

Missing classification
 ↓
Classification workflow
 ↓
Apply default classification
 ↓
Verify

Solo debe automatizarse cuando la acción sea segura y determinista.

42. Compliance Verification

Resolver un finding no significa automáticamente cerrarlo.

Remediation
 ↓
Verification
 ↓
PASS
 ↓
Resolve
43. Verification Evidence

Debe registrarse:

verificationId
verifiedBy
verifiedAt
evidence
controlVersion
44. Compliance Exceptions

Una excepción no convierte automáticamente:

NON_COMPLIANT

en:

COMPLIANT

Puede representar:

ACCEPTED_RISK
EXEMPTED
CONDITIONALLY_COMPLIANT

según el modelo de EVOXA.

45. Exception Relationship
Finding
 ↓
Exception Request
 ↓
Risk Review
 ↓
Approval
 ↓
Temporary Exception

La excepción debe tener expiración.

46. Risk Acceptance

Un riesgo aceptado debe permanecer visible.

NON_COMPLIANCE
      ↓
RISK ACCEPTED

no:

NON_COMPLIANCE
      ↓
DELETE FINDING
47. Compliance Score

EVOXA puede calcular indicadores agregados.

Ejemplo conceptual:

Compliance Score =
weighted compliant controls
/
weighted applicable controls

Pero el score no debe sustituir los findings individuales.

48. Weighted Compliance

Los controles críticos deben tener mayor peso.

Critical Control → weight 10
High → weight 5
Medium → weight 2
Low → weight 1

Los pesos deben ser configurables por governance.

49. Score Anti-Gaming

Un sistema no debe poder parecer compliant simplemente acumulando muchos controles triviales.

Por ello:

Critical failure

puede imponer un límite superior al score.

Ejemplo:

Critical Control Failed
→ Overall status cannot be COMPLIANT

cuando la policy lo establezca.

50. Compliance State

El estado agregado puede ser:

COMPLIANT
PARTIALLY_COMPLIANT
NON_COMPLIANT
UNKNOWN
51. Compliance State Machine
UNKNOWN
   ↓
ASSESSED
   ↓
COMPLIANT

o:

ASSESSED
   ↓
NON_COMPLIANT
   ↓
REMEDIATION
   ↓
VERIFICATION
   ↓
COMPLIANT
52. Continuous Compliance

Compliance no debe ser únicamente una auditoría puntual.

Policy
 ↓
Continuous Controls
 ↓
Continuous Evidence
 ↓
Continuous Assessment
 ↓
Current Compliance State
53. Compliance Monitoring

Debe detectar cambios en:

policy
asset
configuration
ownership
classification
access
quality
lineage
contracts
exceptions
certifications
54. Compliance Drift
COMPLIANT
   ↓
Asset Changes
   ↓
Control Re-evaluation
   ↓
NON_COMPLIANT
55. Compliance Reassessment Triggers

Una reevaluación puede producirse por:

Policy Change
Asset Change
Classification Change
Configuration Change
Access Change
Exception Expiration
Certification Expiration
Scheduled Assessment
Detected Incident
56. Scheduled Assessments

Algunos controles requieren evaluación periódica:

Daily
Weekly
Monthly
Quarterly
Annually

La frecuencia debe ser definida por riesgo y policy.

57. Event-Driven Assessment

En lugar de esperar al próximo ciclo:

AssetChanged
 ↓
Relevant Controls
 ↓
Reassessment

Esto reduce ventanas de incumplimiento no detectado.

58. Compliance Event Architecture

Eventos:

ComplianceAssessmentCompleted
ComplianceStatusChanged
ComplianceFindingCreated
ComplianceFindingAssigned
ComplianceFindingResolved
ComplianceExceptionApproved
ComplianceExceptionExpired
ComplianceCertificationGranted
ComplianceCertificationExpired
59. Compliance Event Flow
Change
 ↓
Control Evaluation
 ↓
Assessment
 ↓
Status Change
 ↓
Event
 ↓
Notification / Workflow / Analytics
60. Compliance Certification

Una certificación representa una afirmación formal de compliance durante un período determinado.

Certification
{
    certificationId,
    subject,
    scope,
    status,
    issuedAt,
    expiresAt,
    evidence,
    issuer
}
61. Certification States
CANDIDATE
CERTIFIED
EXPIRING
EXPIRED
REVOKED
62. Certification Expiration

Una certificación no debe permanecer válida indefinidamente.

expiresAt

debe ser obligatorio cuando la naturaleza del certification lo requiera.

63. Certification Revocation

Puede producirse por:

policy change
finding
evidence invalidation
control failure
risk escalation
64. Compliance Dashboard

El sistema debe poder representar:

Overall Compliance
Compliance by Domain
Compliance by Asset
Open Findings
Critical Findings
Expired Exceptions
Expiring Certifications
Control Failures
Evidence Gaps
65. Compliance Query Model

Consultas fundamentales:

Which assets are non-compliant?

Which controls are failing?

Which findings are critical?

Which policies have the highest failure rate?

Which domains have declining compliance?

Which evidence is missing?

Which exceptions expire soon?
66. Compliance Read Model

Puede materializar:

Asset
Requirement
Control
Assessment
Status
Finding
Risk
Exception
Certification

para consultas rápidas.

67. Compliance Reporting

Debe soportar:

Operational Reports
Management Reports
Risk Reports
Audit Reports
Regulatory Reports
Domain Reports
Control Reports
68. Audit Report

Un auditor debe poder obtener:

Scope
Requirements
Controls
Assessments
Evidence
Findings
Exceptions
Remediation
Certifications
Audit Trail
69. Compliance Snapshot

Para una fecha determinada:

ComplianceSnapshot
{
    scope,
    capturedAt,
    policyVersions,
    controlVersions,
    statuses,
    findings,
    evidenceReferences
}

Esto permite reconstruir el estado histórico.

70. Historical Compliance

Debe ser posible responder:

What was the compliance status on date X?

sin sobrescribir la historia.

71. Temporal Model

Los elementos relevantes deben conservar:

effectiveFrom
effectiveTo
createdAt
updatedAt

cuando aplique.

72. Compliance Versioning

Un assessment debe saber:

Policy version
Control version
Rule version
Evidence version

para ser reproducible.

73. Reproducibility

Idealmente:

same subject
+
same policy version
+
same control version
+
same evidence
=
same assessment
74. Compliance Determinism

Los controles automatizados críticos deben producir resultados deterministas.

Las excepciones deben quedar explícitamente modeladas.

75. Manual Assessment Governance

Las evaluaciones humanas deben registrar:

assessor
role
reason
evidence
timestamp
decision
76. Separation of Duties

Para controles críticos:

Requester
   ≠
Assessor

cuando el riesgo lo requiera.

77. Compliance Authorization

No todos los usuarios pueden:

close finding
accept risk
approve exception
issue certification

Estos privilegios deben estar gobernados.

78. Compliance Security

La información de compliance puede ser sensible.

Debe proteger:

findings
vulnerabilities
control weaknesses
risk acceptance
exceptions
audit evidence
79. Compliance Data Classification

Los datos del sistema de compliance deben clasificarse según:

sensitivity
business impact
security impact
regulatory impact
80. Compliance Retention

Evidence y audit data deben tener políticas de retención específicas.

Assessment
Finding
Evidence
Certification
Audit

no necesariamente tienen la misma retención.

81. Compliance Immutability

Los registros históricos críticos no deben poder alterarse silenciosamente.

Assessment
Evidence
Certification
Audit

deben tener controles de integridad.

82. Compliance Integrity

El sistema debe detectar:

missing evidence
modified evidence
orphan assessment
invalid certification
expired control
stale assessment
83. Compliance Reconciliation

Debe compararse periódicamente:

Governance Registry
      ↕
Control Registry
      ↕
Runtime State
      ↕
Compliance State
84. Compliance Drift Detection

Ejemplo:

Certification = ACTIVE
Control = FAILED

Esto debe generar una inconsistencia que provoque:

reassessment
revocation
or investigation

según policy.

85. Compliance Dependency Graph

Los controles pueden depender de otros:

Access Control
      ↓
Classification
      ↓
Ownership
      ↓
Data Inventory

Si falla un control fundamental, otros controles pueden quedar:

UNKNOWN

en vez de marcarse falsamente como compliant.

86. Control Dependency

Ejemplo:

Control A:
Asset must be classified.

Control B:
Access must depend on classification.

Si A falla:

B = UNKNOWN

puede ser más correcto que:

B = COMPLIANT
87. Compliance Engine

El Compliance Engine debe realizar:

Applicability
Requirement Resolution
Control Resolution
Evidence Collection
Evaluation
Scoring
Finding Creation
Status Calculation
88. Compliance Engine Pipeline
Subject
 ↓
Applicability Engine
 ↓
Requirement Resolver
 ↓
Control Resolver
 ↓
Evidence Resolver
 ↓
Assessment Engine
 ↓
Finding Engine
 ↓
Status Aggregator
89. Evidence Collector

Puede obtener evidencia desde:

Governance Repository
Data Catalog
Audit Store
Runtime Logs
Security Systems
Quality Systems
Workflow Systems
Configuration
Access Systems
90. Evidence Collection Failure

Si una fuente no está disponible:

Evidence unavailable

no debe convertirse automáticamente en:

COMPLIANT

Puede resultar:

UNKNOWN

o:

NON_COMPLIANT

según el control.

91. Compliance Workflow

Para un finding:

Finding
 ↓
Triage
 ↓
Assign
 ↓
Remediate
 ↓
Verify
 ↓
Close
92. SLA

Los findings pueden tener:

targetResolutionTime
dueDate
escalationPolicy

según severidad.

93. Escalation
Finding
 ↓
Due Date Approaches
 ↓
Owner Notification
 ↓
Manager Escalation
 ↓
Governance Escalation
94. Aging

Debe poder medirse:

findingAge
timeToRemediation
timeToVerification
timeToClosure
95. Compliance Risk

El riesgo puede modelarse:

Risk =
Likelihood × Impact

pero la fórmula concreta debe ser configurable.

96. Risk-Based Compliance

No todos los incumplimientos tienen la misma prioridad.

Critical + High Impact
        ↓
Immediate Remediation

mientras:

Low + Low Impact
        ↓
Scheduled Remediation
97. Compliance Prioritization

La priorización puede considerar:

severity
business criticality
data sensitivity
exposure
likelihood
impact
age
98. Compliance Automation

Puede automatizar:

assessment
evidence collection
finding creation
notification
assignment
remediation
verification

cuando sea seguro.

99. Compliance Human-in-the-Loop

Debe existir para:

ambiguous controls
risk acceptance
exceptions
critical findings
certification
policy interpretation
100. Compliance API

Modelo conceptual:

POST /compliance/assessments
GET  /compliance/assessments/{id}

GET  /compliance/status
GET  /compliance/findings
POST /compliance/findings/{id}/remediate

GET  /compliance/evidence
GET  /compliance/certifications
101. Compliance Command Model
RunAssessment
CreateFinding
AssignFinding
AcceptRisk
RequestException
ApproveException
RemediateFinding
VerifyFinding
IssueCertification
RevokeCertification
102. Idempotency

Los comandos críticos deben soportar:

commandId

para evitar duplicación.

103. Compliance Event Outbox

Cambios críticos deben seguir:

Assessment
 +
Outbox Event

en la misma unidad transaccional cuando sea necesario.

104. Compliance Resilience

El sistema debe tolerar:

Evidence source unavailable
Control engine unavailable
Event bus unavailable
Reporting unavailable
Notification unavailable

sin perder evidencia crítica.

105. Compliance Availability

Debe distinguirse:

Assessment Path
Evidence Path
Reporting Path
Administration Path

No todos necesitan la misma disponibilidad.

106. Fail-Safe Compliance

Cuando no pueda evaluarse un control crítico:

UNKNOWN

debe ser preferible a inventar:

COMPLIANT
107. Compliance Observability

Métricas:

assessment_count
compliant_count
non_compliant_count
unknown_count
finding_count
critical_finding_count
evidence_failure_count
remediation_time
verification_time
certification_count
108. Compliance SLOs

Ejemplos:

Assessment completion latency
Evidence collection success
Finding detection latency
Remediation workflow availability
Certification processing latency
109. Compliance Tracing

Debe seguirse:

Assessment
 ↓
Requirement
 ↓
Control
 ↓
Evidence
 ↓
Evaluation
 ↓
Finding
 ↓
Remediation
 ↓
Verification

mediante correlationId / traceId.

110. Compliance Architecture
                         ┌──────────────────────┐
                         │ Governance Policies  │
                         └──────────┬───────────┘
                                    │
                                    ▼
                         ┌──────────────────────┐
                         │ Applicability Engine │
                         └──────────┬───────────┘
                                    │
                                    ▼
                         ┌──────────────────────┐
                         │ Requirement Resolver │
                         └──────────┬───────────┘
                                    │
                                    ▼
                         ┌──────────────────────┐
                         │   Control Resolver   │
                         └──────────┬───────────┘
                                    │
                                    ▼
                         ┌──────────────────────┐
                         │   Evidence Collector │
                         └──────────┬───────────┘
                                    │
                                    ▼
                         ┌──────────────────────┐
                         │  Compliance Engine   │
                         └──────────┬───────────┘
                                    │
                      ┌─────────────┼─────────────┐
                      ▼             ▼             ▼
                 Assessment      Finding       Status
                      │             │             │
                      └─────────────┼─────────────┘
                                    ▼
                         ┌──────────────────────┐
                         │ Remediation Workflow │
                         └──────────┬───────────┘
                                    │
                                    ▼
                         ┌──────────────────────┐
                         │     Verification     │
                         └──────────┬───────────┘
                                    │
                                    ▼
                         ┌──────────────────────┐
                         │ Certification /      │
                         │ Compliance State     │
                         └──────────────────────┘
111. Compliance Integration With Previous Architecture

E64 depende directamente de:

E06  — Policy Architecture
E20  — Runtime Policy Architecture
E21  — Rules Engine Architecture
E22  — Validation Architecture
E26  — Projection Architecture
E28  — Read Model Architecture
E30  — Reporting Architecture
E31  — Analytics Architecture
E38  — Resilience Architecture
E39  — Fault Tolerance Architecture
E40  — Recovery Architecture
E43  — Data Integrity Architecture
E44  — Consistency Architecture
E45  — Data Lifecycle
E46  — Data Retention
E48  — Data Archival
E55  — Data Access Governance
E56  — Data Access Security
E57  — Data Protection
E58  — Data Privacy
E59  — Data Governance
E60  — Data Governance Operating Model
E61  — Data Governance Implementation Architecture
E62  — Data Governance Control Plane Architecture
E63  — Data Governance Enforcement Architecture
112. Architectural Chain

La secuencia ahora es:

E59
DATA GOVERNANCE
      ↓
E60
OPERATING MODEL
      ↓
E61
IMPLEMENTATION
      ↓
E62
CONTROL PLANE
      ↓
E63
ENFORCEMENT
      ↓
E64
COMPLIANCE

La semántica es:

E59 → Define
E60 → Operates
E61 → Implements
E62 → Decides
E63 → Enforces
E64 → Verifies
113. Compliance Acceptance Criteria

E64 está correctamente implementado cuando:

✓ Requirements tienen identidad y versión
✓ Controls tienen identidad y versión
✓ Existe Applicability Engine
✓ Existe Compliance Engine
✓ Existe Evidence Collection
✓ Evidence tiene provenance
✓ Evidence tiene integridad
✓ Assessments son reproducibles
✓ Existe estado COMPLIANT
✓ Existe estado NON_COMPLIANT
✓ Existe estado UNKNOWN
✓ Existen Findings
✓ Findings tienen owner
✓ Findings tienen severity
✓ Existe Remediation Workflow
✓ Existe Verification
✓ Existen Exceptions controladas
✓ Existe Risk Acceptance
✓ Existen Certifications
✓ Certifications pueden expirar
✓ Certifications pueden revocarse
✓ Existe Continuous Compliance
✓ Existe Drift Detection
✓ Existe Historical Compliance
✓ Existe Compliance Reporting
✓ Existe Audit Trail
✓ Existe observabilidad
✓ Existe fail-safe behavior
114. Invariantes Fundamentales
Invariant 1
No evidence
≠
COMPLIANT
Invariant 2
Expired evidence
≠
valid evidence
Invariant 3
Expired certification
≠
active certification
Invariant 4
Open critical finding
→
cannot produce COMPLIANT overall status

cuando la policy lo determine.

Invariant 5
Risk acceptance
≠
remediation
Invariant 6
Exception
must be time-bound when required.
Invariant 7
Compliance status
must be traceable to controls and evidence.
Invariant 8
Historical compliance
must not be silently rewritten.
115. Principio Rector

EVOXA Data Governance Compliance Architecture convierte las políticas, controles, evidencias y resultados de enforcement en un sistema continuo, verificable y auditable de evaluación del cumplimiento, capaz de detectar desviaciones, generar findings, coordinar remediaciones, verificar resultados y mantener una representación histórica y reproducible del estado de governance.

La cadena queda:

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
              ↓
E65 — DATA GOVERNANCE MONITORING & ASSURANCE ARCHITECTURE

E62 = decide.
E63 = hace cumplir.
E64 = verifica.
E65 deberá cerrar el ciclo operativo mediante monitorización continua, assurance, detección temprana de desviaciones y evidencia de que todo el sistema de governance continúa funcionando correctamente.
