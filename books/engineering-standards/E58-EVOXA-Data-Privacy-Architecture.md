E58 — EVOXA Data Privacy Architecture
Engineering Specification

Pertenece a: Engineering Specification
Capítulo: E58 — Data Privacy Architecture
Anterior: E57 — Data Protection Architecture
Siguiente: E59 — Data Governance Architecture

1. Propósito

E58 define la arquitectura mediante la cual EVOXA garantiza que los datos personales y otros datos sujetos a requisitos de privacidad sean:

recopilados con una finalidad legítima y definida;
utilizados únicamente dentro de los propósitos autorizados;
minimizados;
procesados de forma transparente;
accesibles y controlables por las partes autorizadas;
protegidos durante todo su lifecycle;
transferidos únicamente bajo condiciones permitidas;
conservados durante el tiempo necesario;
eliminados o anonimizados cuando corresponda;
auditables y demostrables.

La diferencia fundamental respecto de E57 es:

E56 — Data Access Security
        ↓
¿Quién puede acceder?

E57 — Data Protection
        ↓
¿Cómo protegemos el dato?

E58 — Data Privacy
        ↓
¿Por qué podemos procesarlo,
para qué podemos usarlo,
qué derechos aplican
y cuándo debemos dejar de procesarlo?
2. Objetivo Arquitectónico

La privacidad debe tratarse como una propiedad arquitectónica del sistema, no como una funcionalidad aislada.

Data Subject
      │
      ▼
Collection
      │
      ▼
Purpose
      │
      ▼
Processing
      │
      ├── Access
      ├── Sharing
      ├── Transfer
      ├── Retention
      └── Deletion
      │
      ▼
Privacy Controls
      │
      ▼
Accountability
3. Principios Fundamentales
3.1 Privacy by Design

La privacidad debe incorporarse durante el diseño de:

domains
APIs
databases
events
workflows
integrations
analytics
AI systems

y no añadirse posteriormente.

3.2 Privacy by Default

La configuración inicial debe minimizar el procesamiento de datos personales.

Default
   ↓
minimum collection
minimum exposure
minimum retention
minimum sharing
3.3 Purpose Limitation

Los datos deben estar vinculados a una finalidad explícita.

Data
 ↓
Purpose
 ↓
Permitted Processing

No debe asumirse:

Collected
   ≠
usable for everything
3.4 Data Minimization

Recopilar:

únicamente los datos adecuados, relevantes y necesarios para la finalidad definida.

3.5 Transparency

Debe poder explicarse:

what
why
how
where
with whom
how long

se procesan los datos.

3.6 Accountability

EVOXA debe poder demostrar que sus controles de privacidad existen y funcionan.

Policy
 ↓
Control
 ↓
Evidence
 ↓
Auditability
4. Scope

E58 cubre:

Privacy Classification
Data Subject
Purpose Management
Processing Purposes
Privacy Policies
Consent Management
Lawful Processing Basis
Notice
Transparency
Data Minimization
Purpose Enforcement
Privacy Preferences
Data Subject Rights
Access Requests
Correction Requests
Deletion Requests
Restriction Requests
Portability
Objection
Consent Withdrawal
Privacy Requests
Privacy Workflows
Privacy-aware APIs
Privacy-aware Events
Privacy-aware Integrations
Privacy-aware Analytics
Privacy-aware AI
Cross-border Privacy
Privacy Retention
Anonymization
Pseudonymization
Privacy Audit
Privacy Monitoring
Privacy Compliance Evidence
5. Non-Goals

E58 no reemplaza:

E43 — Data Integrity
E45 — Data Lifecycle
E46 — Data Retention
E47 — Data Disposal
E48 — Data Archival
E55 — Data Access Governance
E56 — Data Access Security
E57 — Data Protection

E58 consume estas capacidades para construir el modelo de privacidad.

6. Privacy Architecture
                         ┌─────────────────────┐
                         │   Data Subject      │
                         └──────────┬──────────┘
                                    │
                                    ▼
                         ┌─────────────────────┐
                         │ Privacy Identity    │
                         └──────────┬──────────┘
                                    │
                                    ▼
                         ┌─────────────────────┐
                         │ Purpose / Policy    │
                         └──────────┬──────────┘
                                    │
                  ┌─────────────────┼─────────────────┐
                  ▼                 ▼                 ▼
             Collection         Processing        Sharing
                  │                 │                 │
                  └─────────────────┼─────────────────┘
                                    ▼
                         ┌─────────────────────┐
                         │ Privacy Enforcement │
                         └──────────┬──────────┘
                                    │
              ┌──────────────┬──────┼──────┬──────────────┐
              ▼              ▼      ▼      ▼              ▼
           Rights         Consent  DSR   Retention    Transfer
              │              │      │      │              │
              └──────────────┴──────┼──────┴──────────────┘
                                    ▼
                         ┌─────────────────────┐
                         │ Privacy Audit       │
                         └─────────────────────┘
7. Data Subject

EVOXA debe poder representar la entidad sobre la cual existen datos personales.

DataSubject
├── subjectId
├── tenantId
├── identifiers
├── privacyStatus
├── preferences
└── rightsState
8. Data Subject Identity

La identidad de privacidad debe ser independiente de cualquier identificador operativo cuando sea necesario.

Operational Customer ID
        │
        ▼
Privacy Subject ID

Esto facilita:

identity resolution
privacy requests
cross-system correlation
9. Identity Resolution

EVOXA debe poder resolver diferentes identificadores que representan al mismo Data Subject.

email
customerId
externalId
accountId
deviceId
       │
       ▼
Privacy Subject

La resolución debe estar estrictamente controlada para evitar correlaciones indebidas.

10. Tenant Privacy Boundary

En arquitectura multi-tenant:

Tenant A
 └── Subjects A

Tenant B
 └── Subjects B

Los mecanismos de privacidad no deben permitir correlacionar o revelar datos entre tenants sin una autorización explícita.

11. Personal Data Classification

Los datos pueden clasificarse:

NON_PERSONAL
PERSONAL
SENSITIVE_PERSONAL
SPECIAL_CATEGORY / RESTRICTED

La taxonomía concreta debe definirse mediante governance.

12. Privacy Metadata

Cada dataset relevante puede tener:

PrivacyMetadata
├── personalData
├── subjectType
├── purpose
├── processingBasis
├── retention
├── residency
├── sharingPolicy
└── rightsApplicable
13. Purpose Architecture

La finalidad debe ser una entidad explícita.

Purpose
├── purposeId
├── description
├── scope
├── allowedData
├── allowedProcessing
├── retention
└── status
14. Purpose Binding

Los datos personales deben vincularse a uno o más propósitos.

Dataset
   │
   ▼
Purpose
   │
   ▼
Allowed Processing
15. Purpose Registry

EVOXA debería disponer de un registro centralizado de finalidades:

Purpose Registry
├── account_management
├── billing
├── security
├── service_delivery
├── analytics
├── communications
└── other approved purposes

Las finalidades concretas deben ser gobernadas por el contexto legal y operativo aplicable.

16. Purpose Limitation Enforcement

Antes de utilizar un dato:

Data
 ↓
Requested Processing
 ↓
Purpose Check
 ↓
ALLOW / DENY
17. Secondary Use

Un uso secundario no debe asumirse automáticamente como permitido.

Original Purpose
      ↓
New Purpose
      ↓
Privacy Assessment
      ↓
Allow / Require Consent / Deny
18. Processing Basis

El sistema debe poder registrar la base que autoriza determinado procesamiento.

Conceptualmente:

Processing
   ↓
Processing Basis

Ejemplos de categorías posibles pueden incluir:

consent
contract
legal_obligation
legitimate_interest
vital_interest
public_task

La disponibilidad y utilización de cada categoría dependerá del marco jurídico aplicable.

19. Processing Basis Registry
ProcessingBasis
├── basisId
├── purpose
├── scope
├── jurisdiction
├── effectiveFrom
├── effectiveTo
└── evidence
20. Jurisdiction-Aware Privacy

La misma operación puede estar sujeta a diferentes reglas según:

Data Subject
+
Tenant
+
Processing Location
+
Service
+
Jurisdiction

Por ello:

Privacy Policy

no debe tratarse necesariamente como una única regla global.

21. Privacy Policy

Una política puede definir:

Policy
├── jurisdiction
├── dataCategories
├── purposes
├── processingBasis
├── retention
├── rights
├── sharing
└── transferRules
22. Policy Resolution
Subject
   │
   ▼
Jurisdiction
   │
   ▼
Applicable Privacy Policy
   │
   ▼
Processing Decision
23. Consent Architecture

Cuando el procesamiento dependa de consentimiento:

Consent
├── subjectId
├── purpose
├── version
├── timestamp
├── source
├── status
└── evidence
24. Consent Granularity

El consentimiento debe poder gestionarse por finalidad cuando sea necesario.

Subject
 ├── Product Communications → granted
 ├── Marketing → denied
 └── Analytics → granted
25. Consent Versioning

Los cambios relevantes de finalidad o información proporcionada pueden requerir una nueva versión:

Consent v1
   ↓
Policy Change
   ↓
Consent v2
26. Consent Evidence

Debe conservarse evidencia suficiente para demostrar:

who
what
when
purpose
version
source

sin conservar datos innecesarios.

27. Consent Withdrawal

El retiro debe poder expresarse:

Granted
   ↓
Withdrawn

y activar las acciones correspondientes.

28. Withdrawal Propagation
Consent Withdrawal
       │
       ├── API
       ├── Database
       ├── Messaging
       ├── Marketing
       ├── Analytics
       └── Integrations

La propagación debe ser controlable y auditable.

29. Consent ≠ Universal Permission

Tener consentimiento para una finalidad no autoriza automáticamente:

unrelated purpose
unrelated sharing
unrelated processing
30. Privacy Preferences

Las preferencias pueden incluir:

communications
marketing
analytics
personalization
data sharing
tracking

y deben mantenerse separadas de los permisos técnicos de acceso.

31. Privacy Notice

EVOXA debe poder producir información sobre:

data collected
purpose
processing
retention
sharing
transfers
rights

según el contexto aplicable.

32. Notice Versioning

Los notices deben versionarse:

Notice v1
Notice v2
Notice v3

para poder determinar qué información se proporcionó en un momento determinado.

33. Transparency Architecture

La transparencia debe estar soportada por metadata estructurada.

Data
 ↓
Metadata
 ↓
Privacy Explanation
34. Data Inventory

EVOXA debería mantener un inventario de:

data categories
systems
purposes
processors
destinations
retention
35. Processing Inventory

Para cada operación:

Processing Activity
├── source
├── data
├── purpose
├── basis
├── destination
├── retention
└── security controls
36. Data Flow Mapping
Collection
   ↓
Service
   ↓
Database
   ↓
Analytics
   ↓
External Processor

Debe ser posible representar qué datos personales recorren cada frontera.

37. Privacy Data Lineage

E57 protege el dato.

E58 debe además conocer:

where did it come from?
why was it collected?
where did it go?
why was it sent there?
38. Data Subject Rights

La arquitectura debe soportar solicitudes de privacidad cuando resulten aplicables:

Access
Correction
Deletion
Restriction
Portability
Objection
Consent Withdrawal

La disponibilidad exacta de cada derecho depende de la jurisdicción y contexto.

39. Data Subject Request

Modelo:

PrivacyRequest
├── requestId
├── subjectId
├── type
├── jurisdiction
├── submittedAt
├── status
├── verification
├── deadline
└── result
40. Privacy Request Lifecycle
SUBMITTED
   ↓
IDENTITY_VERIFICATION
   ↓
SCOPING
   ↓
DISCOVERY
   ↓
REVIEW
   ↓
EXECUTION
   ↓
VALIDATION
   ↓
COMPLETED
41. Request Identity Verification

Una solicitud de derechos no debe permitir a una persona obtener los datos de otra.

Request
   ↓
Identity Verification
   ↓
Subject Resolution
42. Request Scoping

Determinar:

which tenant
which systems
which datasets
which data categories
which jurisdiction
43. Privacy Discovery

El sistema debe localizar:

primary data
replicas
indexes
caches
derived data
exports
archives

cuando estén dentro del alcance de la solicitud.

44. Privacy Request Orchestration
Privacy Request
      │
      ▼
Orchestrator
      │
 ┌────┼─────────────┐
 ▼    ▼             ▼
DB   Search      Integrations
 │     │             │
 └─────┼─────────────┘
       ▼
   Consolidation
45. Access Request

Una solicitud de acceso debe producir una vista consolidada y apropiadamente protegida.

Systems
  ↓
Data Collection
  ↓
Normalization
  ↓
Redaction
  ↓
Subject Report
46. Access Request Security

El resultado de una solicitud de acceso contiene datos personales y debe estar protegido mediante E57.

47. Correction Request

La corrección debe:

validate request
identify authoritative source
update source
propagate where required
audit result
48. Deletion Request

La eliminación debe integrarse con:

E45 Lifecycle
E46 Retention
E47 Disposal
E57 Data Protection
49. Deletion Decision

No toda solicitud puede traducirse automáticamente en eliminación inmediata.

Deletion Request
      ↓
Retention / Legal Obligation Check
      ↓
Delete / Restrict / Retain
50. Restriction

Cuando corresponda, EVOXA puede pasar el procesamiento a un estado restringido:

ACTIVE
  ↓
RESTRICTED_PROCESSING
51. Restriction Enforcement
Restricted Subject Data
       │
       ├── Required processing → allowed
       ├── prohibited processing → denied
       └── required retention → preserved
52. Portability

Cuando sea aplicable, los datos portables deben producirse en un formato:

structured
commonly used
machine-readable

y mantenerse protegidos.

53. Portability Export
Subject Request
      ↓
Data Discovery
      ↓
Portable Projection
      ↓
Protection
      ↓
Secure Delivery
54. Objection

Cuando un sujeto pueda oponerse a determinado procesamiento:

Objection
   ↓
Purpose
   ↓
Policy Evaluation
   ↓
Stop / Restrict / Continue
55. Automated Decisioning

Si EVOXA realiza decisiones automatizadas relevantes para personas:

Subject
   ↓
Decision Engine
   ↓
Outcome

deben existir mecanismos para determinar:

decision type
purpose
input data
applicable policy
review requirements

según el contexto aplicable.

56. Profiling

Los procesos de profiling deben estar identificados explícitamente:

Data
 ↓
Profile
 ↓
Decision / Personalization

y sujetos a la política de privacidad correspondiente.

57. Privacy-Aware APIs

Las APIs deben poder aplicar:

purpose
subject
privacy preference
processing basis
field minimization
58. Privacy-Aware Queries

Una consulta no debería devolver automáticamente todos los datos personales disponibles.

Query
 ↓
Privacy Context
 ↓
Allowed Projection
 ↓
Result
59. Privacy-Aware Projection
Customer
├── id             → allowed
├── displayName    → allowed
├── privateEmail   → restricted
├── identityData   → restricted
└── internalNotes  → denied
60. Privacy-Aware Events

Los eventos deben transportar sólo los datos necesarios.

CustomerUpdated
{
  subjectId
  changedFields
}

preferido frente a transportar el registro completo.

61. Privacy Event Propagation

Un cambio de privacidad puede requerir eventos:

ConsentWithdrawn
PrivacyRestrictionApplied
DeletionRequested
SubjectDeleted
SubjectDataCorrected
62. Privacy Workflow
Privacy Trigger
      ↓
Policy Evaluation
      ↓
Workflow
      ├── discover
      ├── validate
      ├── transform
      ├── propagate
      └── verify
63. Privacy Jobs

Operaciones de gran volumen pueden ejecutarse como jobs:

PrivacyRequestJob
DeletionPropagationJob
RetentionEnforcementJob
ConsentPropagationJob
64. Privacy Scheduling

Los controles periódicos pueden incluir:

retention checks
consent reconciliation
privacy drift scans
data inventory refresh
processor reconciliation
65. Privacy Cache

Las preferencias y decisiones de privacidad pueden cachearse cuando sea seguro.

Pero:

Privacy Change
      ↓
Cache Invalidation

debe ser rápido y fiable.

66. Privacy Decision Cache

No debe existir una cache que mantenga una autorización de privacidad obsoleta indefinidamente.

67. Privacy Search

Los índices deben respetar:

subject privacy
deletion state
restriction state
tenant boundary
purpose
68. Deletion Propagation to Search
Deletion
   ↓
Primary DB
   ↓
Search Index
   ↓
Cache
   ↓
Derived Models
69. Analytics Privacy

Analytics debe operar con el mínimo nivel de granularidad necesario.

Raw Personal Data
       ↓
Aggregation / Pseudonymization
       ↓
Analytics Dataset
70. Statistical Privacy

Cuando sea apropiado pueden utilizarse:

aggregation
thresholding
suppression
noise

para reducir riesgos de reidentificación.

71. Privacy in Reporting

Los reportes deben controlar:

row-level access
field visibility
aggregation level
export capability
72. AI Privacy

Los sistemas de inteligencia deben conocer:

data purpose
data sensitivity
subject restrictions
consent / basis
retention
training eligibility
73. Training Data Eligibility

No todo dato disponible para EVOXA debe ser elegible para:

model training
fine-tuning
evaluation
prompt examples
74. AI Data Boundary
Operational Data
       ↓
Privacy Eligibility
       ↓
Approved AI Dataset
       ↓
Model
75. AI Forgetting / Deletion

Cuando un requerimiento de privacidad afecte datos utilizados por sistemas de IA, debe existir una política explícita sobre:

source data
training datasets
embeddings
indexes
derived artifacts

No se debe asumir que eliminar el registro operacional elimina automáticamente todas sus derivaciones.

76. Embedding Privacy

Los embeddings derivados de datos personales deben tratarse según la sensibilidad y posibilidad de reconstrucción o asociación.

Personal Data
   ↓
Embedding
   ↓
Protected Derived Data
77. External Processor

Antes de enviar datos personales a un tercero:

Data
 ↓
Purpose
 ↓
Processor / Recipient
 ↓
Allowed Data
 ↓
Transfer Policy
78. Processor Registry
Processor
├── processorId
├── purpose
├── dataCategories
├── jurisdictions
├── retention
├── transferMechanism
└── status
79. Third-Party Sharing

La arquitectura debe distinguir:

internal processing
processor processing
independent recipient

porque pueden tener obligaciones diferentes.

80. Cross-Border Transfer
Source Region
      ↓
Transfer Assessment
      ↓
Destination Region
      ↓
Allowed / Restricted / Denied
81. Transfer Metadata

Debe poder registrarse:

origin
destination
data category
purpose
recipient
transfer basis
protection controls
82. Privacy Residency

La residencia de datos puede estar determinada por:

tenant
subject
contract
jurisdiction
policy

y no únicamente por la ubicación del servidor.

83. Retention Integration

E58 utiliza E46:

Purpose
   ↓
Retention Requirement
   ↓
Retention Policy

La finalidad debe influir en cuánto tiempo es necesario conservar los datos.

84. Purpose-Based Retention
Purpose A → 30 days
Purpose B → 7 years
Purpose C → until withdrawal

Los valores son ilustrativos; los períodos reales deben venir de governance/legal policy.

85. Retention Conflict

Si existen múltiples finalidades:

Data
├── Purpose A
└── Purpose B

el sistema debe resolver qué obligación de retención prevalece según la política aplicable.

86. Privacy Deletion

Cuando finaliza la necesidad legítima de conservar datos:

No Longer Needed
       ↓
Deletion / Anonymization
87. Anonymization

Cuando la finalidad requiere conservar información estadística pero no identificar sujetos:

Personal Dataset
       ↓
Anonymization
       ↓
Non-personal / Lower-risk Dataset

Debe existir una evaluación apropiada de reidentificación.

88. Pseudonymization vs Anonymization
Pseudonymization
    ↓
Identity can potentially be restored

Anonymization
    ↓
Identity should no longer be reasonably recoverable
89. Privacy-Preserving Derived Data

Un dataset derivado puede conservarse si:

purpose remains valid
privacy risk is acceptable
identification is controlled
retention is justified
90. Privacy Audit

Debe registrarse:

purpose decisions
consent changes
privacy requests
data sharing
transfer decisions
deletion actions
policy changes
91. Privacy Evidence

EVOXA debe poder demostrar:

what policy applied
what decision was made
when
for whom
by which component
with what result
92. Privacy Audit Integrity

Los registros de privacidad deben estar protegidos contra manipulación.

E57 proporciona:

integrity
encryption
access control

E58 define:

what privacy evidence must exist
93. Privacy Monitoring

Indicadores:

open privacy requests
overdue requests
consent mismatches
purpose violations
unclassified personal data
retention violations
cross-border violations
deletion propagation failures
processor violations
94. Privacy Drift

Ejemplo:

Policy:
Analytics requires consent

Runtime:
Analytics receives subjects without consent

Esto constituye:

Privacy Drift
95. Privacy Compliance State

Cada dataset o processing activity puede tener:

COMPLIANT
NON_COMPLIANT
UNKNOWN
EXCEPTION
REMEDIATION
96. Privacy Exceptions

Una excepción debe registrar:

reason
scope
owner
approval
expiration
compensating controls

Las excepciones indefinidas deben evitarse.

97. Privacy Incident

Un incidente de privacidad puede producirse cuando:

purpose violation
unauthorized disclosure
wrong recipient
wrong retention
failed deletion
incorrect subject resolution
consent violation
98. Privacy Incident Flow
Detection
   ↓
Containment
   ↓
Scope
   ↓
Impact Assessment
   ↓
Remediation
   ↓
Evidence
   ↓
Required Notification

Los requisitos concretos de notificación dependen de jurisdicción y contexto.

99. Privacy Risk Assessment

Para nuevos tratamientos de alto riesgo, EVOXA debe poder representar:

Processing
   ↓
Risk Assessment
   ├── data sensitivity
   ├── scale
   ├── purpose
   ├── subjects
   ├── transfers
   └── safeguards
100. Privacy Impact Assessment

La plataforma debe poder soportar un artefacto de evaluación:

PrivacyImpactAssessment
├── processing
├── risks
├── controls
├── residualRisk
├── approval
└── reviewDate
101. Privacy Review Lifecycle
DRAFT
 ↓
ASSESSMENT
 ↓
REVIEW
 ↓
APPROVED
 ↓
ACTIVE
 ↓
REVIEW_REQUIRED
 ↓
RETIRED
102. Privacy Policy Enforcement Engine
                   ┌──────────────────┐
                   │ Privacy Context  │
                   └────────┬─────────┘
                            │
                            ▼
                   ┌──────────────────┐
                   │ Policy Resolver  │
                   └────────┬─────────┘
                            │
                            ▼
                   ┌──────────────────┐
                   │ Purpose Check    │
                   └────────┬─────────┘
                            │
              ┌─────────────┼──────────────┐
              ▼             ▼              ▼
           Consent        Basis        Restriction
              │             │              │
              └─────────────┼──────────────┘
                            ▼
                   ┌──────────────────┐
                   │ Decision         │
                   └────────┬─────────┘
                            │
                     ALLOW / DENY
103. Privacy Context

Una decisión puede requerir:

subject
tenant
purpose
dataCategory
jurisdiction
processingType
destination
consentState
restrictionState
104. Privacy Decision
PrivacyDecision
{
  subject,
  purpose,
  basis,
  policy,
  decision,
  reason,
  timestamp
}
105. Decision Determinism

Para una misma combinación de contexto y política:

same input
+
same policy version
=
same decision

salvo factores explícitamente dinámicos.

106. Privacy Policy Versioning

Las decisiones deben poder asociarse a:

policyVersion

para permitir auditoría histórica.

107. Privacy Policy Evaluation
Request
   ↓
Subject
   ↓
Data Category
   ↓
Purpose
   ↓
Jurisdiction
   ↓
Policy
   ↓
Decision
108. Privacy-Aware Domain Model

Los dominios que manejan datos personales deberían expresar explícitamente:

PersonalData
Purpose
PrivacyState

en lugar de tratar la privacidad como metadata puramente externa cuando esto pueda producir errores.

109. Privacy-Aware Repository
Repository
   ↓
Subject Scope
   ↓
Privacy Filter
   ↓
Data Access
110. Privacy-Aware Read Model

Los read models pueden contener datos personales y deben mantener:

subject identity
privacy state
deletion state
purpose
111. Privacy-Aware Projection

Cuando se proyectan datos:

Source
 ↓
Privacy Evaluation
 ↓
Projection
 ↓
Protected Read Model
112. Privacy-Aware Search Index

Los índices deben poder responder a:

subject deleted?
subject restricted?
tenant valid?
purpose valid?
113. Privacy-Aware Cache Invalidation

Cambios de privacidad deben disparar invalidaciones:

Deletion
   ↓
Cache Invalidation
Restriction
   ↓
Sensitive Cache Invalidation
114. Privacy-Aware Messaging

Los cambios relevantes deben poder propagarse mediante eventos:

PrivacyStateChanged
ConsentChanged
SubjectDeleted
SubjectRestricted
PurposeChanged
115. Privacy-Aware Integration

Un conector externo debe poder responder:

Can this data be sent?
Why?
Which fields?
For how long?

antes de realizar el envío.

116. Privacy-Aware Workflow
Business Workflow
       │
       ▼
Privacy Check
       │
   ┌───┴───┐
   ▼       ▼
ALLOW     BLOCK

La privacidad debe ser un guardrail del workflow.

117. Privacy-Aware Job Processing

Jobs batch deben:

resolve purpose
resolve policy
filter subjects
process allowed data
record evidence
118. Privacy-Aware Scheduling

Las tareas programadas que procesen datos personales deben poder invalidarse o modificarse cuando cambie el estado de privacidad.

119. Privacy-Aware Configuration

Configuraciones como:

analyticsEnabled
marketingEnabled
personalizationEnabled
externalSharingEnabled

no deben sustituir el motor de privacidad.

Son inputs de configuración, no autorización universal.

120. Privacy Feature Flags

Un feature flag que habilita una nueva finalidad de procesamiento debe requerir evaluación de privacidad.

Feature Enabled
      ↓
Privacy Policy
      ↓
Purpose Valid?
      ↓
Proceed
121. Privacy and Rules Engine

E21 puede expresar reglas como:

IF
  subject.jurisdiction == X
AND
  purpose == Y
AND
  consent != granted
THEN
  DENY
122. Privacy and Validation

E22 debe validar:

purpose exists
basis exists
subject exists
policy applicable
required privacy metadata exists
123. Privacy and Serialization

E23 debe evitar serializar campos personales innecesarios.

Entity
 ↓
Privacy-aware DTO
 ↓
Serialized Output
124. Privacy and Transformation

E24 puede implementar:

masking
redaction
aggregation
pseudonymization
anonymization
125. Privacy and Mapping

E25 debe evitar mapear automáticamente todos los campos personales entre modelos.

126. Privacy and Projection

E26 debe producir vistas compatibles con:

purpose
subject rights
privacy state
127. Privacy and Query

E27 debe aplicar filtros de privacidad antes de ejecutar o completar una consulta cuando sea necesario.

128. Privacy and Reporting

E30 debe garantizar que:

report
=
privacy-approved projection
129. Privacy and Analytics

E31 debe controlar:

raw data access
aggregation
retention
re-identification
purpose
130. Privacy and Intelligence

E32 debe conocer las restricciones sobre los datos utilizados para inferencia.

131. Privacy and Decision

E33 debe registrar si una decisión utiliza datos personales y bajo qué finalidad.

132. Privacy and Action

E34 debe evitar que una acción produzca tratamiento de datos fuera del propósito permitido.

133. Privacy and Execution

E35 debe transportar el contexto de privacidad hasta el punto de ejecución.

134. Privacy and Resources

E36 debe considerar que ciertos recursos pueden estar restringidos por:

jurisdiction
data residency
privacy classification
135. Privacy and Capacity

E37 debe considerar cargas producidas por:

privacy requests
deletion jobs
data exports
subject discovery
136. Privacy and Resilience

E38 debe garantizar que los mecanismos de privacidad sobrevivan a fallos.

Un sistema no debe perder:

consent state
restriction state
deletion state
privacy policy state

por una caída parcial.

137. Privacy and Recovery

E40 debe restaurar también:

privacy metadata
consent records
deletion state
request state
policy versions
138. Privacy and Disaster Recovery

Los entornos DR deben mantener las mismas obligaciones de privacidad.

Production Privacy Policy
        =
DR Privacy Policy
139. Privacy and Backup

Los backups deben preservar los estados necesarios para cumplir derechos y obligaciones.

Al mismo tiempo:

Backup
≠
excuse for indefinite retention
140. Privacy and Consistency

E44 debe evitar inconsistencias como:

Primary → deleted
Replica → active
Cache → active

durante períodos superiores a los aceptables definidos por la arquitectura.

141. Privacy and Data Lifecycle

La privacidad está integrada en:

Creation
 ↓
Collection
 ↓
Use
 ↓
Sharing
 ↓
Retention
 ↓
Archive
 ↓
Deletion
142. Privacy Lifecycle State
COLLECTED
   ↓
ACTIVE
   ↓
RESTRICTED
   ↓
RETENTION
   ↓
ARCHIVED
   ↓
DELETED / ANONYMIZED
143. Privacy State Invariants
Invariant 1

Todo dato personal debe tener una finalidad identificable.

Invariant 2

Todo procesamiento debe tener una base/política aplicable.

Invariant 3

Un consentimiento retirado no debe continuar autorizando procesamiento dependiente de dicho consentimiento.

Invariant 4

Una solicitud de privacidad debe poder rastrearse hasta su resultado.

Invariant 5

La eliminación debe propagarse a los sistemas incluidos en el alcance definido.

Invariant 6

Los datos personales no deben compartirse fuera de los destinos autorizados.

Invariant 7

La privacidad debe mantenerse independientemente del sistema de almacenamiento.

Invariant 8

Los datos personales no deben conservarse indefinidamente sin una justificación válida.

Invariant 9

Los cambios de política deben ser versionados.

Invariant 10

Las decisiones de privacidad deben ser auditables.

Invariant 11

El contexto de privacidad debe acompañar al procesamiento distribuido.

Invariant 12

Una falla de privacidad crítica debe producir un comportamiento seguro: DENY, RESTRICT, QUARANTINE o equivalente.

144. Privacy Request Architecture
                  ┌──────────────────┐
                  │   Data Subject   │
                  └────────┬─────────┘
                           │
                           ▼
                  ┌──────────────────┐
                  │ Privacy Request │
                  └────────┬─────────┘
                           │
                           ▼
                  ┌──────────────────┐
                  │ Identity Verify  │
                  └────────┬─────────┘
                           │
                           ▼
                  ┌──────────────────┐
                  │ Request Scope    │
                  └────────┬─────────┘
                           │
              ┌────────────┼────────────┐
              ▼            ▼            ▼
             DB         Search        Events
              │            │            │
              └────────────┼────────────┘
                           ▼
                  ┌──────────────────┐
                  │ Consolidation    │
                  └────────┬─────────┘
                           │
                           ▼
                  ┌──────────────────┐
                  │ Privacy Action   │
                  └────────┬─────────┘
                           │
                           ▼
                  ┌──────────────────┐
                  │ Verification     │
                  └────────┬─────────┘
                           │
                           ▼
                       COMPLETE
145. Privacy Control Plane vs Data Plane

Una separación clara:

                 EVOXA
                   │
        ┌──────────┴──────────┐
        ▼                     ▼
 Privacy Control Plane    Data Plane
        │                     │
        ├── Policies          ├── Data
        ├── Purposes          ├── Queries
        ├── Consent           ├── Processing
        ├── Rights            ├── Events
        ├── Requests          └── Storage
        └── Evidence

El Control Plane determina qué tratamiento está permitido.

El Data Plane ejecuta el tratamiento permitido.

146. Privacy Enforcement Boundary

La enforcement debe existir lo más cerca posible del procesamiento:

Request
   ↓
Application
   ↓
Privacy Enforcement
   ↓
Domain Operation
   ↓
Repository

pero también debe existir defense-in-depth en las capas inferiores apropiadas.

147. Fail-Closed Privacy

Para controles críticos:

Cannot determine privacy policy
        ↓
DENY

en lugar de:

Cannot determine policy
        ↓
ALLOW
148. Privacy Observability

Debe poder observarse:

privacy decision latency
privacy denials
consent changes
DSR volume
DSR completion
deletion propagation
policy conflicts
privacy violations
149. Privacy Metrics

Métricas recomendadas:

privacy_requests_open
privacy_requests_overdue
privacy_requests_completed
consent_grants
consent_withdrawals
purpose_denials
privacy_policy_denials
deletion_propagation_failures
privacy_drift_events
cross_border_denials
personal_data_inventory_coverage
150. Privacy SLOs

EVOXA debe definir objetivos para:

request acknowledgement
request processing
consent propagation
restriction propagation
deletion propagation
privacy decision availability
privacy policy availability

Los valores concretos deben definirse por governance y requisitos aplicables.

151. Privacy Testing

Debe probarse:

purpose enforcement
consent enforcement
withdrawal propagation
subject resolution
access requests
deletion requests
restriction requests
portability
cross-tenant isolation
cross-border restrictions
retention
anonymization
processor sharing
AI eligibility
152. Negative Testing

Ejemplos:

No Purpose
    → DENY

No Applicable Basis
    → DENY

Consent Withdrawn
    → DENY dependent processing

Wrong Tenant
    → DENY

Wrong Jurisdiction
    → DENY / RESTRICT

Expired Retention
    → DELETE / ANONYMIZE

Deleted Subject
    → Exclude from active processing

Unauthorized Processor
    → DENY
153. Privacy Chaos Testing

Debe probarse que fallos de infraestructura no provoquen:

consent loss
privacy-state loss
deletion-state loss
cross-tenant leakage
policy bypass
154. Privacy Disaster Recovery Testing

Debe verificarse que después de restauración:

privacy policies
consent state
subject rights state
deletion state
processing records
audit evidence

sean coherentes.

155. Privacy Security Boundary

La relación completa queda:

E55
Data Access Governance
        │
        ▼
E56
Data Access Security
        │
        ▼
E57
Data Protection
        │
        ▼
E58
Data Privacy
        │
        ▼
Privacy-Compliant Processing
156. Privacy vs Security

La distinción arquitectónica fundamental:

Área	Pregunta
Access Governance	¿Quién debería poder acceder?
Access Security	¿Cómo verificamos y hacemos cumplir ese acceso?
Data Protection	¿Cómo protegemos el dato?
Data Privacy	¿Por qué, para qué y bajo qué condiciones podemos tratarlo?
157. Privacy Architecture Invariant

El principio central de E58:

La existencia de un dato en EVOXA no implica autorización para utilizarlo. Todo procesamiento de datos personales debe estar vinculado a una finalidad, una política aplicable y las condiciones de privacidad correspondientes.

158. Completion Criteria

E58 se considera arquitectónicamente completo cuando EVOXA dispone de:

✓ Data Subject model
✓ Privacy identity
✓ Tenant privacy boundary
✓ Personal data classification
✓ Privacy metadata
✓ Purpose registry
✓ Purpose enforcement
✓ Processing basis model
✓ Jurisdiction-aware policies
✓ Consent management
✓ Consent versioning
✓ Consent withdrawal
✓ Preference management
✓ Privacy notice support
✓ Transparency metadata
✓ Data inventory
✓ Processing inventory
✓ Privacy lineage
✓ Data Subject Requests
✓ Identity verification
✓ Access requests
✓ Correction requests
✓ Deletion requests
✓ Restriction requests
✓ Portability
✓ Objection handling
✓ Automated decisioning controls
✓ Profiling controls
✓ Privacy-aware APIs
✓ Privacy-aware queries
✓ Privacy-aware projections
✓ Privacy-aware events
✓ Privacy workflows
✓ Privacy jobs
✓ Privacy scheduling
✓ Privacy cache invalidation
✓ Privacy search
✓ Analytics privacy
✓ Reporting privacy
✓ AI privacy
✓ Training-data eligibility
✓ Embedding protection
✓ Processor registry
✓ Third-party sharing controls
✓ Cross-border transfer controls
✓ Residency controls
✓ Purpose-based retention
✓ Anonymization
✓ Pseudonymization
✓ Privacy audit
✓ Privacy evidence
✓ Privacy monitoring
✓ Privacy drift detection
✓ Privacy exceptions
✓ Privacy incidents
✓ Privacy risk assessment
✓ Privacy impact assessment
✓ Policy versioning
✓ Fail-closed enforcement
✓ Privacy observability
✓ Privacy metrics
✓ Privacy SLOs
✓ Privacy testing
✓ Privacy disaster recovery testing
159. Principio Rector de E58

EVOXA Data Privacy Architecture garantiza que los datos personales sean tratados únicamente de manera justificada, transparente, limitada por finalidad, minimizada, controlable y auditable, manteniendo las obligaciones de privacidad desde la recopilación hasta la eliminación, independientemente del servicio, almacenamiento, región, integración, workflow, analytics o sistema de inteligencia que participe en su procesamiento.

La secuencia arquitectónica queda:

E54 — DATA ACCESS ABSTRACTION
        ↓
E55 — DATA ACCESS GOVERNANCE
        ↓
E56 — DATA ACCESS SECURITY
        ↓
E57 — DATA PROTECTION
        ↓
E58 — DATA PRIVACY
        ↓
E59 — DATA GOVERNANCE

E57 responde “¿cómo protegemos el dato?”.
E58 responde “¿por qué, para qué, bajo qué condiciones y durante cuánto tiempo podemos tratarlo?”.
