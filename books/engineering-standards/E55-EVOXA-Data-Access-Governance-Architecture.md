E55 — EVOXA Data Access Governance Architecture
Engineering Specification

Pertenece a: Engineering Specification
Capítulo: E55 — Data Access Governance Architecture
Anterior: E54 — Data Access Abstraction Architecture
Siguiente: E56 — Data Access Security Architecture

1. Propósito

E55 define el modelo mediante el cual EVOXA gobierna quién puede acceder a qué datos, mediante qué contrato, bajo qué condiciones, con qué propósito, durante cuánto tiempo y bajo qué controles operativos y de cumplimiento.

E54 responde:

¿Cómo desacoplamos al consumidor de la implementación del acceso?

E55 responde:

¿Bajo qué reglas está permitido ese acceso y cómo demostramos que dichas reglas se cumplen?

La relación fundamental es:

E54 — Data Access Abstraction
        │
        ▼
Access Contract
        │
        ▼
E55 — Data Access Governance
        │
        ├── Identity
        ├── Authorization
        ├── Purpose
        ├── Tenant
        ├── Data Classification
        ├── Policy
        ├── Approval
        ├── Audit
        ├── Monitoring
        └── Lifecycle
2. Objetivo Arquitectónico

EVOXA debe evitar que el acceso a datos sea una capacidad implícita.

No:

Service
   │
   ▼
Database

Sino:

Service
   │
   ▼
Access Contract
   │
   ▼
Governance Evaluation
   │
   ├── Identity
   ├── Tenant
   ├── Authorization
   ├── Purpose
   ├── Classification
   ├── Policy
   └── Context
          │
          ▼
     Access Decision
          │
     ┌────┴────┐
     ▼         ▼
   Allow      Deny
     │
     ▼
Implementation
3. Principios Fundamentales
3.1 Least Privilege

Cada consumidor obtiene únicamente el acceso necesario.

Required Access
      │
      ▼
Minimum Privilege
3.2 Explicit Authorization

El acceso debe ser autorizado explícitamente.

Authenticated
    ≠
Authorized
3.3 Policy Driven

Las decisiones deben derivarse de políticas:

Access Request
      │
      ▼
Policy Evaluation
      │
      ▼
Decision
3.4 Tenant Isolation

Los datos de un tenant no deben quedar accesibles por otro tenant por accidente o configuración incorrecta.

3.5 Purpose Limitation

El acceso debe estar asociado, cuando corresponda, a un propósito legítimo.

3.6 Traceability

Toda operación gobernada debe poder ser reconstruida posteriormente.

3.7 Accountability

Debe ser posible determinar:

who
what
when
where
why
which policy
which decision
3.8 Default Deny

Si una política no concede acceso:

NO POLICY
    ↓
DENY
4. Scope

E55 gobierna:

Data Access Policies
Access Permissions
Access Roles
Access Scopes
Data Classification
Purpose
Tenant Boundaries
Delegation
Approvals
Access Requests
Temporary Access
Break-Glass Access
Access Reviews
Audit
Compliance
Policy Versioning
Access Certification
Policy Enforcement
Governance Metrics
Access Lifecycle
5. Non-Goals

E55 no reemplaza:

E04 Authentication
E05 Authorization
E06 Policy Architecture
E20 Runtime Policy
E43 Data Integrity
E46 Data Retention
E47 Data Disposal

E55 coordina la gobernanza del acceso a datos utilizando esos mecanismos.

6. Governance Layers
┌──────────────────────────────────────┐
│ Data Access Governance               │
├──────────────────────────────────────┤
│ Governance Policy                    │
├──────────────────────────────────────┤
│ Access Classification                │
├──────────────────────────────────────┤
│ Authorization                       │
├──────────────────────────────────────┤
│ Purpose                              │
├──────────────────────────────────────┤
│ Tenant / Scope                       │
├──────────────────────────────────────┤
│ Runtime Enforcement                  │
├──────────────────────────────────────┤
│ Audit & Evidence                     │
└──────────────────────────────────────┘
7. Data Access Governance Model

El modelo lógico:

Principal
   │
   ▼
Access Request
   │
   ├── Resource
   ├── Operation
   ├── Tenant
   ├── Purpose
   ├── Context
   └── Risk
          │
          ▼
   Policy Evaluation
          │
          ▼
    Governance Decision
          │
    ┌─────┼─────┐
    ▼     ▼     ▼
  Allow Deny Conditional
8. Governance Entities

E55 define como mínimo:

Principal
Data Resource
Access Contract
Access Policy
Access Scope
Purpose
Access Request
Access Decision
Approval
Access Grant
Access Review
Audit Record
Governance Exception
9. Principal

Un principal puede ser:

User
Service
Application
Agent
Workflow
Job
Administrator
External Identity

No debe asumirse que sólo los usuarios humanos acceden a datos.

10. Data Resource

Un recurso gobernable puede ser:

Database
Table
Collection
Dataset
Read Model
Projection
API Resource
Data Service
File
Object
Search Index
Analytics Dataset

La gobernanza debe operar preferentemente sobre recursos lógicos, no únicamente sobre tablas físicas.

11. Access Contract as Governance Boundary

E54 proporciona:

CustomerReader
OrderReader
AccountReader

E55 asigna políticas a esos contratos:

CustomerReader
      │
      ▼
Governance Policy

Esto permite gobernar:

logical capability

sin depender de:

physical storage
12. Access Operation

Cada contrato puede exponer operaciones:

READ
SEARCH
CREATE
UPDATE
DELETE
EXPORT
BULK_READ
BULK_EXPORT
ADMIN

Cada operación puede tener una política diferente.

13. Access Scope

El scope define hasta dónde llega el acceso.

Ejemplos:

TENANT
USER
TEAM
REGION
RESOURCE
RECORD
FIELD
DATASET
GLOBAL
14. Scope Hierarchy
GLOBAL
  │
  ▼
REGION
  │
  ▼
TENANT
  │
  ▼
DOMAIN
  │
  ▼
RESOURCE
  │
  ▼
RECORD
  │
  ▼
FIELD

Las políticas más específicas deben poder restringir las más generales.

15. Data Classification

Los datos deben clasificarse.

Ejemplo:

PUBLIC
INTERNAL
CONFIDENTIAL
RESTRICTED
HIGHLY_RESTRICTED

La taxonomía concreta debe ser configurable por gobernanza empresarial.

16. Classification Drives Governance

Ejemplo:

PUBLIC
   → normal access

CONFIDENTIAL
   → authorization required

RESTRICTED
   → authorization + purpose + audit

HIGHLY_RESTRICTED
   → explicit approval + enhanced audit
17. Classification Must Propagate

Cuando los datos pasan:

Source
  ↓
Projection
  ↓
Cache
  ↓
API
  ↓
Export

su clasificación no debe desaparecer.

18. Data Ownership

Cada recurso gobernado debe tener:

Data Owner
Technical Owner
Steward
Custodian

cuando el modelo organizativo lo requiera.

19. Data Owner

El Data Owner define:

allowed usage
classification
access requirements
retention expectations
approved consumers
20. Data Steward

El steward ayuda a mantener:

metadata
classification
quality
access definitions
governance rules
21. Access Request

Una solicitud debe contener conceptualmente:

requestId
principal
resource
operation
tenant
purpose
requestedScope
requestedDuration
context
risk
22. Access Decision

Una decisión debe ser explícita:

ALLOW
DENY
ALLOW_WITH_CONDITIONS
REQUIRE_APPROVAL
23. Conditional Access

Ejemplo:

ALLOW
IF:
  tenant = currentTenant
  purpose = customer_support
  classification <= CONFIDENTIAL
  export = false
24. Policy Evaluation
Access Request
      │
      ▼
Identity Context
      │
      ▼
Resource Metadata
      │
      ▼
Applicable Policies
      │
      ▼
Policy Evaluation
      │
      ▼
Risk Evaluation
      │
      ▼
Decision
25. Policy Precedence

Cuando varias políticas aplican:

Global Policy
     │
     ▼
Tenant Policy
     │
     ▼
Domain Policy
     │
     ▼
Resource Policy
     │
     ▼
Record/Field Policy

Las restricciones específicas no deben ampliar privilegios accidentalmente.

26. Deny Precedence

En conflictos críticos:

ALLOW
+
DENY
=
DENY

Esto protege contra configuraciones permisivas accidentalmente.

27. Explicit Grant

Un grant debe especificar:

principal
resource
operation
scope
conditions
expiration
policy
28. Temporary Access

El acceso temporal debe tener:

startAt
expiresAt
reason
approver
scope

Nunca debe existir:

temporary access
    +
no expiration
29. Just-in-Time Access

Para recursos sensibles:

Request
   │
   ▼
Approval
   │
   ▼
Temporary Grant
   │
   ▼
Access
   │
   ▼
Automatic Expiration
30. Break-Glass Access

Debe existir únicamente cuando sea necesario.

Normal Access
     │
     X
     ▼
Break-Glass
     │
     ▼
Emergency Access

Debe exigir:

reason
identity
time
scope
enhanced audit
post-review
31. Delegated Access

Un principal puede actuar en nombre de otro:

User
  │
  ▼
Delegated Principal
  │
  ▼
Access

La delegación debe registrar:

originalPrincipal
delegatedPrincipal
delegationReason
scope
expiration
32. Service-to-Service Access

Los servicios deben usar identidades de workload:

Service A
   │
   ▼
Workload Identity
   │
   ▼
CustomerReader

No:

shared_database_password
33. Agent Access

Los agentes de IA deben recibir acceso limitado:

Agent
  │
  ▼
Approved Data Tool
  │
  ▼
Governance
  │
  ▼
Data

Nunca deben recibir acceso arbitrario a la infraestructura.

34. Human vs Machine Access

E55 debe distinguir:

Human Access
Machine Access
Agent Access
Emergency Access
Automated Job Access

porque sus controles pueden diferir.

35. Purpose Binding

El propósito puede ser:

CUSTOMER_SUPPORT
OPERATIONS
BILLING
SECURITY
ANALYTICS
REPORTING
AUDIT
LEGAL
MODEL_TRAINING

La taxonomía debe ser gobernada.

36. Purpose Enforcement

Ejemplo:

Customer Data
     │
     ├── support → ALLOW
     ├── billing → ALLOW
     ├── analytics → CONDITIONAL
     └── unrelated purpose → DENY
37. Purpose Non-Repurposing

Que un servicio tenga acceso a un dato para:

billing

no implica automáticamente permiso para:

marketing
38. Data Minimization

La gobernanza debe favorecer:

minimum data
minimum scope
minimum duration
minimum privilege
39. Field-Level Governance

Ejemplo:

Customer
├── id
├── name
├── email
├── phone
└── paymentData

Un actor puede recibir:

id
name
email

pero no:

paymentData
40. Record-Level Governance

Puede aplicarse:

tenantId = currentTenant

o:

region = authorizedRegion
41. Dataset-Level Governance

Un dataset completo puede requerir:

approved role
approved purpose
approved environment
42. Export Governance

Exportar datos es distinto de leerlos.

READ
   ≠
EXPORT

Por tanto:

CustomerReader.read()

no concede automáticamente:

CustomerExport.export()
43. Bulk Access Governance

Una operación:

getCustomer(id)

puede estar permitida mientras:

getAllCustomers()

requiera privilegios adicionales.

44. Search Governance

Buscar por:

email
phone
identity attributes

puede requerir controles más estrictos que leer un registro ya identificado.

45. Aggregation Governance

Una consulta agregada:

COUNT customers

puede ser menos sensible que:

customer-level export

pero debe evaluarse riesgo de reidentificación.

46. Analytics Governance

El acceso analítico debe distinguir:

raw data
pseudonymized data
aggregated data
anonymized data
47. Pseudonymization

Cuando sea posible:

Raw Identifier
      │
      ▼
Pseudonymous Identifier

reduce exposición directa.

48. Anonymization

Cuando los datos deben salir del ámbito identificable:

Raw Data
   │
   ▼
Anonymization
   │
   ▼
Governed Dataset

La gobernanza debe verificar que la transformación realmente cumple el nivel requerido.

49. Environment Governance

Los datos pueden tener reglas distintas:

PRODUCTION
STAGING
TEST
DEVELOPMENT

Ejemplo:

Production Restricted Data
        │
        X
        ▼
Development

salvo mecanismos autorizados de sanitización.

50. Non-Production Data

Debe preferirse:

synthetic data
masked data
anonymized data

para entornos no productivos.

51. Cross-Tenant Access

Debe estar explícitamente gobernado.

Tenant A
   │
   X
   ▼
Tenant B

Sólo debe existir si una política lo permite.

52. Cross-Region Access

También debe gobernarse:

EU Tenant
    │
    X
    ▼
US Data Store

si las políticas de residencia no lo permiten.

53. Cross-Domain Access

Un dominio no debería acceder directamente a los stores de otro dominio.

Preferido:

Domain A
   │
   ▼
Approved Access Contract
   │
   ▼
Domain B Data
54. Direct Database Access

El acceso directo debe ser excepcional:

Application
    │
    X
    ▼
Database

Preferido:

Application
    │
    ▼
Governed Access Contract
    │
    ▼
Database
55. Governance Exceptions

Cuando sea necesario romper una regla:

Exception Request
      │
      ▼
Risk Review
      │
      ▼
Approval
      │
      ▼
Time-Bounded Exception
56. Exception Requirements

Toda excepción debe incluir:

reason
owner
scope
risk
mitigation
expiration
approval
review date
57. No Permanent Exceptions

Una excepción sin expiración se convierte de facto en política.

Por tanto:

Las excepciones deben expirar automáticamente salvo renovación explícita.

58. Access Approval

Los recursos críticos pueden requerir:

single approval
dual approval
owner approval
security approval
compliance approval

según clasificación y riesgo.

59. Separation of Duties

Una persona no debería poder:

request
approve
execute
audit

la misma operación sensible sin controles adicionales.

60. Four-Eyes Principle

Para acceso crítico:

Requester
    │
    ▼
Approver A
    │
    ▼
Approver B
    │
    ▼
Grant
61. Access Review

Los grants permanentes deben revisarse periódicamente.

Grant
  │
  ▼
Review
  │
  ├── retain
  ├── reduce
  └── revoke
62. Certification

Los owners deben poder certificar:

"This principal still requires this access."
63. Automatic Revocation

Debe revocarse cuando:

user disabled
service retired
role removed
tenant closed
contract retired
grant expired
policy revoked
64. Access Lifecycle
REQUESTED
   ↓
EVALUATED
   ↓
APPROVED
   ↓
GRANTED
   ↓
ACTIVE
   ↓
REVIEWED
   ↓
RENEWED / REDUCED / REVOKED
   ↓
REVOKED
65. Policy Lifecycle
DRAFT
  ↓
REVIEW
  ↓
APPROVED
  ↓
ACTIVE
  ↓
DEPRECATED
  ↓
RETIRED
66. Policy Change Governance

Los cambios críticos deben producir:

changeId
author
reviewer
reason
diff
effectiveAt
version
67. Policy Versioning

Nunca debe sobrescribirse silenciosamente una política activa.

Policy v1
Policy v2
Policy v3

Cada decisión debe poder identificar qué versión se evaluó.

68. Policy Rollback

Debe ser posible revertir:

v3
 ↓
v2

si una política nueva genera acceso incorrecto.

69. Policy Simulation

Antes de activar una política:

Historical Requests
       │
       ▼
New Policy
       │
       ▼
Simulated Decisions

Esto permite detectar:

unexpected allows
unexpected denies
70. Shadow Policy

Una política nueva puede ejecutarse sin enforcement:

Request
 ├── Current Policy → enforce
 └── New Policy → observe

Después:

compare
71. Governance Decision Record

Cada decisión relevante debe poder registrar:

decisionId
requestId
principal
resource
operation
scope
purpose
policyVersion
decision
conditions
timestamp
72. Audit Record

El audit trail debe responder:

Who accessed?
What was accessed?
When?
From where?
For what purpose?
Under which policy?
What decision was made?
What was the result?
73. Audit Integrity

Los registros de auditoría deben estar protegidos contra:

tampering
unauthorized deletion
unauthorized modification
74. Audit Retention

La retención del audit trail debe seguir:

legal requirements
compliance requirements
security requirements
operational requirements

y alinearse con E46.

75. Audit vs Operational Logs

No deben confundirse:

Operational Log
    → debugging / operations

Audit Record
    → accountability / compliance
76. Governance Monitoring

Métricas:

access_requests
access_allowed
access_denied
approval_requests
approval_latency
temporary_grants
expired_grants
exceptions
policy_violations
cross_tenant_attempts
sensitive_access
bulk_exports
77. Governance Risk Signals

Debe detectarse:

unusual access volume
unusual tenant access
unusual time
unusual geography
unusual resource
repeated denial
bulk access
rapid privilege escalation
78. Anomaly Detection
Normal Access Pattern
        │
        ▼
Behavioral Baseline
        │
        ▼
Deviation
        │
        ▼
Risk Signal
79. Risk-Based Governance

La decisión puede incorporar:

principal risk
resource sensitivity
operation risk
volume
purpose
environment
location
time
80. Risk Levels

Ejemplo:

LOW
MEDIUM
HIGH
CRITICAL
81. Adaptive Controls

Mayor riesgo puede activar:

additional authentication
approval
reduced scope
shorter duration
enhanced logging
blocking
82. Governance Enforcement Point
Consumer
   │
   ▼
Access Request
   │
   ▼
Policy Enforcement Point
   │
   ▼
Policy Decision Point
   │
   ▼
Decision
83. PEP / PDP
Policy Enforcement Point

Aplica la decisión.

Policy Decision Point

Evalúa las políticas.

PEP
 │
 ▼
PDP
 │
 ▼
Decision
 │
 ▼
PEP
84. Policy Information Point

El PDP puede consultar:

Principal attributes
Resource metadata
Tenant state
Purpose
Risk
Environment
PIP
 │
 ├── identity
 ├── tenant
 ├── classification
 └── context
85. Policy Administration Point

Las políticas son administradas mediante:

PAP
 │
 ▼
Policy Repository
 │
 ▼
Policy Versions
86. Governance Architecture
                   ┌─────────────────────┐
                   │ Governance Admin    │
                   └──────────┬──────────┘
                              │
                              ▼
                    ┌──────────────────┐
                    │ Policy Repository│
                    └────────┬─────────┘
                             │
                             ▼
                    ┌──────────────────┐
                    │ Policy Decision  │
                    │ Point             │
                    └────────┬─────────┘
                             │
Access Request ──────────────┤
                             ▼
                    ┌──────────────────┐
                    │ Enforcement Point│
                    └────────┬─────────┘
                             │
                             ▼
                    E54 Access Contract
                             │
                             ▼
                        Data Source
87. Governance Registry

EVOXA debe mantener un registry lógico de:

data resources
access contracts
owners
classifications
policies
grants
exceptions
approvals
88. Resource Registration

Un recurso no debería entrar en producción sin metadata mínima:

resourceId
owner
classification
tenantScope
allowedOperations
governancePolicy
89. Unregistered Data

Principio:

Unknown Resource
      ↓
DENY or restricted quarantine

No debe existir acceso productivo completamente desconocido para governance.

90. Access Contract Registration

Cada contrato debe registrar:

contractId
version
owner
operations
dataResources
classification
policies
implementations
91. Ownership Transfer

Cuando cambia el owner:

Old Owner
    │
    ▼
Transfer
    │
    ▼
New Owner

debe quedar auditado.

92. Access Inventory

EVOXA debe poder responder:

Who can access Customer Data?
Which services can access Restricted Data?
Which grants expire soon?
Which data resources have no owner?
93. Governance Queries

Ejemplos:

listAccessByPrincipal()
listPrincipalsByResource()
listExpiredGrants()
listUnreviewedAccess()
listSensitiveAccess()
listCrossTenantAccess()
listExceptions()
94. Governance Dashboard

Debe proporcionar:

Access Health
Policy Health
Exception Health
Certification Health
Audit Health
Risk Signals
95. Access Review Dashboard

Debe mostrar:

principal
resource
scope
purpose
lastUsed
grantedAt
expiresAt
owner
risk
96. Dormant Access

Debe detectarse:

Granted Access
      │
      ▼
No Usage
      │
      ▼
Review

Puede recomendarse revocación.

97. Privilege Creep

Debe detectarse:

Initial Role
   │
   ├── grant A
   ├── grant B
   ├── grant C
   └── grant D

sin justificación continua.

98. Access Right-Sizing

Governance puede recomendar:

Current:
READ + WRITE + EXPORT

Observed:
READ only

Recommendation:
READ
99. Data Access Governance for APIs

Los APIs que exponen datos deben registrar:

API
operation
resource
classification
scope
consumer
policy
100. Data Access Governance for Events

La publicación de eventos con datos también es acceso/disclosure.

Producer
   │
   ▼
Event
   │
   ▼
Consumer

Debe gobernarse:

event classification
allowed consumers
field exposure
tenant scope
purpose
101. Event Data Minimization

No:

CustomerUpdated
{
   fullCustomerRecord
}

si el consumer sólo necesita:

customerId
status
102. Cache Governance

Las caches deben heredar:

classification
tenant boundary
authorization assumptions
retention constraints

Una cache no debe convertirse en bypass de governance.

103. Search Governance

Los índices de búsqueda deben considerarse recursos gobernados.

Search Index
   │
   ▼
Governance Policy

No deben exponer campos que el usuario no puede leer.

104. Read Model Governance

Un read model puede tener clasificación propia derivada de sus fuentes.

Sensitive Source
       │
       ▼
Read Model
       │
       ▼
Governed Access
105. Export Governance

Todo export debe registrar:

exportId
principal
resource
purpose
scope
format
destination
timestamp
policy
106. External Sharing

Compartir datos fuera de EVOXA requiere:

approved destination
approved purpose
approved classification
approved scope
expiration
audit
107. Governance for Data Pipelines

Los pipelines también deben estar gobernados:

Source
  │
  ▼
Pipeline
  │
  ▼
Destination

Cada transferencia debe tener:

authorized source
authorized destination
authorized purpose
108. Governance for Jobs

Un job debe poseer sólo los permisos necesarios:

Job
  │
  ▼
Job Identity
  │
  ▼
Governed Access
109. Governance for Workflows

Cada workflow step debe respetar:

workflow identity
step identity
resource policy
tenant
purpose
110. Governance for Scheduled Access

Los jobs programados no deben recibir permisos indefinidos si sólo necesitan:

daily read

Preferido:

time-scoped permission

o identidad de servicio con permisos estrictamente limitados.

111. Governance for AI Training

Si EVOXA utiliza datos para entrenamiento:

Data
 │
 ▼
Training Eligibility Policy
 │
 ├── classification
 ├── consent
 ├── purpose
 ├── residency
 └── retention

Sólo después:

Training Dataset
112. Governance for AI Inference

En inferencia:

Agent
 │
 ▼
Data Access Tool
 │
 ▼
Governance
 │
 ▼
Approved Data

El modelo no debe recibir datos fuera del scope autorizado.

113. Governance for Human-in-the-Loop

Cuando un proceso requiere aprobación humana:

AI / Service
      │
      ▼
Sensitive Access Request
      │
      ▼
Human Approval
      │
      ▼
Temporary Grant
114. Governance Evidence

EVOXA debe poder producir evidencia de:

policy version
access decision
approval
grant
usage
revocation
exception
review
115. Compliance Mapping

Las políticas deben poder mapearse a:

internal controls
security controls
privacy requirements
contractual obligations
regulatory obligations

sin codificar necesariamente una regulación concreta en el access layer.

116. Governance Control Framework

Cada control debe definir:

controlId
objective
scope
owner
policy
enforcement
evidence
reviewFrequency
status
117. Control Effectiveness

Un control no se considera suficiente sólo porque exista una política.

Debe comprobarse:

Policy Exists
      ↓
Policy Active
      ↓
Policy Enforced
      ↓
Policy Observed
      ↓
Policy Effective
118. Governance Drift

Debe detectarse divergencia entre:

Declared Policy
      ≠
Runtime Configuration

o:

Governance Registry
      ≠
Actual Access
119. Configuration Drift

Ejemplo:

Policy:
READ only

Runtime:
READ + EXPORT

Esto debe generar:

Governance Violation
120. Continuous Governance

La gobernanza no debe ser únicamente:

design-time

Debe existir:

Design
   ↓
Deploy
   ↓
Runtime
   ↓
Observe
   ↓
Review
   ↓
Improve
121. Governance Feedback Loop
Access
  │
  ▼
Telemetry
  │
  ▼
Risk Analysis
  │
  ▼
Policy Review
  │
  ▼
Policy Update
  │
  ▼
New Enforcement
122. Governance Failure Modes

Debe manejar:

Policy unavailable
Policy repository unavailable
Identity unavailable
Metadata unavailable
Audit unavailable
Registry unavailable
Approval service unavailable
123. Fail-Closed vs Fail-Open

Para datos sensibles:

Governance Dependency Failure
          │
          ▼
        DENY

Para recursos de bajo riesgo pueden existir políticas explícitas de degradación.

Nunca debe depender de un comportamiento implícito.

124. Governance Availability

El sistema de governance es crítico.

Debe proporcionar:

high availability
low latency
policy caching
versioned policies
safe fallback
125. Policy Cache

Las políticas pueden cachearse:

Policy Store
    │
    ▼
Policy Cache
    │
    ▼
Decision Engine

Pero:

revocation
critical policy update

debe poder invalidar rápidamente el cache.

126. Governance Decision Cache

Una decisión puede cachearse sólo si sus inputs siguen siendo válidos:

principal
resource
operation
scope
policyVersion
context
expiration
127. Governance Consistency

Para decisiones críticas:

latest policy

debe tener precedencia sobre caches potencialmente obsoletos.

128. Governance Latency

El control de acceso no debe convertirse en un cuello de botella.

Objetivo:

Access Request
   │
   ├── Identity
   ├── Policy
   └── Context
          │
          ▼
      Decision

con latencia predecible.

129. Governance Observability

Métricas mínimas:

policy_evaluations
policy_denials
policy_allows
policy_errors
policy_latency
approval_latency
grant_count
revocation_count
exception_count
review_overdue
130. Governance Alerts

Alertar ante:

unexpected privilege escalation
mass export
cross-tenant attempt
repeated denied sensitive access
policy evaluation failures
expired approval still used
unowned resource
unreviewed sensitive grant
131. Security Event Integration

Los eventos de governance pueden alimentar:

Security Monitoring
SIEM
Risk Engine
Incident Management
132. Governance Incident

Cuando ocurre una violación:

Detection
   │
   ▼
Governance Incident
   │
   ├── investigate
   ├── contain
   ├── revoke
   ├── notify
   └── remediate
133. Automatic Containment

Para ciertos eventos:

Risk Threshold Exceeded
        │
        ▼
Temporary Block
        │
        ▼
Investigation
134. Governance Runbooks

Debe existir procedimiento para:

unexpected access
policy misconfiguration
mass export
tenant isolation violation
expired grant
broken audit
policy rollback
emergency access
135. Testing Governance

Debe probarse:

ALLOW cases
DENY cases
boundary cases
tenant isolation
policy precedence
expiration
revocation
delegation
break-glass
136. Negative Testing

Especial importancia:

Tenant A → Tenant B
User → Admin data
Read → Export
Expired grant → Access
Revoked grant → Access
Wrong purpose → Access

Todos deben fallar correctamente.

137. Policy Unit Tests

Cada política crítica debe tener:

positive tests
negative tests
boundary tests
regression tests
138. Policy Integration Tests

Debe probarse:

Identity
+
Policy
+
Resource Metadata
+
Enforcement

como sistema.

139. Governance Chaos Testing

Debe probarse:

policy service unavailable
stale policy cache
identity delay
audit unavailable
registry inconsistency

para verificar que el sistema no abre accesos accidentalmente.

140. Governance Deployment

Las políticas deben desplegarse como artefactos versionados:

Policy Source
    │
    ▼
Validation
    │
    ▼
Simulation
    │
    ▼
Approval
    │
    ▼
Deployment
    │
    ▼
Monitoring
141. Policy CI/CD

Pipeline:

Commit
  ↓
Lint
  ↓
Static Validation
  ↓
Unit Tests
  ↓
Simulation
  ↓
Review
  ↓
Deploy
  ↓
Observe
142. Policy Rollout

Puede utilizarse:

canary
shadow
tenant-by-tenant
region-by-region
percentage rollout

para minimizar riesgo.

143. Policy Rollback Trigger

Rollback automático o manual cuando:

denials spike
unexpected allows
latency spike
security anomaly
policy evaluation errors
144. Governance Ownership

Responsabilidades:

Data Owner
    → data usage rules

Security
    → security controls

Platform
    → enforcement infrastructure

Application Owner
    → consumer justification

Compliance
    → regulatory controls

Engineering
    → implementation correctness
145. RACI Conceptual
Actividad	Data Owner	Security	Platform	App Owner
Classification	A	C	I	C
Policy Definition	A	A/C	C	C
Enforcement	I	C	A	C
Access Request	I	C	I	A
Approval	A	C	I	C
Audit	C	A	A	C
Review	A	A/C	C	A
146. Governance APIs

Conceptualmente:

POST   /access-requests
GET    /access-requests/{id}

POST   /access-grants
DELETE /access-grants/{id}

GET    /access-policies
POST   /access-policies

POST   /access-reviews
POST   /access-exceptions

GET    /access-audit
GET    /governance-resources

Los endpoints concretos pertenecen a E03.

147. Governance Domain Model
Principal
    │
    ├── AccessGrant
    │       │
    │       └── AccessScope
    │
    └── AccessRequest
            │
            ▼
        AccessDecision
            │
            ▼
        AccessPolicy
            │
            ▼
        DataResource
148. Access Grant Model
AccessGrant
├── grantId
├── principal
├── resource
├── operation
├── scope
├── purpose
├── conditions
├── grantedBy
├── grantedAt
├── expiresAt
├── policyVersion
└── status
149. Access Request Model
AccessRequest
├── requestId
├── requester
├── principal
├── resource
├── operation
├── scope
├── purpose
├── duration
├── justification
├── risk
└── status
150. Access Decision Model
AccessDecision
├── decisionId
├── requestId
├── result
├── policyVersion
├── conditions
├── evaluatedAt
├── expiresAt
└── reason
151. Governance Resource Model
GovernedResource
├── resourceId
├── type
├── owner
├── classification
├── tenantScope
├── allowedOperations
├── policyRefs
└── lifecycleState
152. Governance Policy Model
AccessPolicy
├── policyId
├── version
├── resource
├── principalConditions
├── operationConditions
├── scopeConditions
├── purposeConditions
├── environmentConditions
├── riskConditions
├── effect
└── lifecycle
153. Policy Decision Example
Principal:
  support-agent

Resource:
  Customer

Operation:
  READ

Tenant:
  tenant-123

Purpose:
  CUSTOMER_SUPPORT

Classification:
  CONFIDENTIAL

Decision:
  ALLOW

Conditions:
  currentTenantOnly
  noExport
  auditRequired
154. Policy Violation Example
Principal:
  support-agent

Resource:
  Customer

Operation:
  EXPORT

Policy:
  EXPORT prohibited

Decision:
  DENY

Debe producir:

AccessDenied
+
AuditRecord
+
SecuritySignal

cuando corresponda.

155. Governance with E54

La integración principal:

                E55
       Data Access Governance
                  │
                  ▼
           Policy Decision
                  │
                  ▼
                E54
       Data Access Abstraction
                  │
                  ▼
          Access Contract
                  │
                  ▼
            Implementation
156. Governance with E05

E05 define:

who is authorized

E55 añade:

why
what data
under which policy
for how long
under which conditions
with which evidence
157. Governance with E06

E06 define el framework general de políticas.

E55 especializa ese framework para:

data access governance
158. Governance with E20

E20 controla políticas en runtime.

E55 define las necesidades de gobernanza:

policy
ownership
approval
lifecycle
audit
review
159. Governance with E46–E48

Access governance debe respetar:

retention
disposal
archival

Por ejemplo:

Archived Data
     │
     ▼
Different Access Policy
160. Governance with E43–E44

Data access no debe comprometer:

integrity
consistency

Un access path alternativo no puede convertirse en bypass de controles de integridad.

161. Governance with E49–E54
Migration
Synchronization
Replication
Federation
Virtualization
Access Abstraction
        │
        ▼
Access Governance

E55 se encuentra por encima de las distintas formas de acceso.

162. End-to-End Architecture
                         ┌─────────────────────┐
                         │ Governance Admin    │
                         └──────────┬──────────┘
                                    │
                                    ▼
                         ┌─────────────────────┐
                         │ Policy Repository   │
                         └──────────┬──────────┘
                                    │
                                    ▼
                         ┌─────────────────────┐
                         │ Policy Decision     │
                         │ Point               │
                         └──────────┬──────────┘
                                    │
Request ────────────────────────────┤
                                    ▼
                         ┌─────────────────────┐
                         │ Enforcement Point   │
                         └──────────┬──────────┘
                                    │
                                    ▼
                         ┌─────────────────────┐
                         │ E54 Access Contract │
                         └──────────┬──────────┘
                                    │
                 ┌──────────────────┼──────────────────┐
                 ▼                  ▼                  ▼
             Database             API             Virtual Layer
                 │                  │                  │
                 └──────────────────┼──────────────────┘
                                    ▼
                              Governed Data
163. Core Invariants
Invariant 1 — Default Deny

La ausencia de una autorización explícita no concede acceso.

Invariant 2 — Least Privilege

Ningún consumidor debe obtener más acceso del necesario.

Invariant 3 — Tenant Isolation

Un principal no puede cruzar fronteras de tenant sin una política explícita.

Invariant 4 — Purpose Limitation

El acceso concedido para un propósito no implica autorización para otro.

Invariant 5 — Classification Preservation

La clasificación de los datos debe conservarse a través de las capas de acceso.

Invariant 6 — No Governance Bypass

Cache, search, replicas, read models, APIs y adapters no pueden convertirse en rutas alternativas que evadan governance.

Invariant 7 — Time-Bounded Privilege

El acceso temporal debe expirar automáticamente.

Invariant 8 — Auditability

Todo acceso sensible debe producir evidencia suficiente para reconstruir la decisión.

Invariant 9 — Policy Versioning

Toda decisión debe poder asociarse a la versión de política utilizada.

Invariant 10 — Separation of Duties

Las operaciones de alto riesgo deben impedir combinaciones indebidas de solicitud, aprobación y ejecución.

Invariant 11 — Revocation

Los permisos revocados no deben continuar siendo efectivos por caches o configuraciones obsoletas más allá del límite permitido.

Invariant 12 — Fail Secure

Los fallos de governance no deben convertirse accidentalmente en permisos adicionales.

Invariant 13 — Ownership

Todo recurso gobernado debe tener un responsable identificable.

Invariant 14 — Continuous Review

Los privilegios deben poder revisarse y retirarse cuando dejen de ser necesarios.

164. Completion Criteria

E55 se considera completo cuando EVOXA dispone de:

✓ Governed data resource registry
✓ Data classification
✓ Resource ownership
✓ Access contract governance
✓ Access operation governance
✓ Access scopes
✓ Principal governance
✓ Tenant governance
✓ Purpose binding
✓ Least privilege
✓ Default deny
✓ Policy evaluation
✓ Policy precedence
✓ Conditional access
✓ Temporary access
✓ JIT access
✓ Break-glass access
✓ Delegated access
✓ Service identity governance
✓ Agent access governance
✓ Export governance
✓ Bulk access governance
✓ Field-level governance
✓ Record-level governance
✓ Dataset-level governance
✓ Environment governance
✓ Cross-tenant controls
✓ Cross-region controls
✓ Cross-domain controls
✓ Exception management
✓ Approval workflows
✓ Separation of duties
✓ Access certification
✓ Access reviews
✓ Automatic revocation
✓ Policy lifecycle
✓ Policy versioning
✓ Policy simulation
✓ Shadow policies
✓ Policy rollback
✓ Governance registry
✓ Access inventory
✓ Dormant access detection
✓ Privilege creep detection
✓ Access right-sizing
✓ Audit records
✓ Audit integrity
✓ Governance metrics
✓ Risk signals
✓ Anomaly detection
✓ PEP/PDP architecture
✓ Policy caching
✓ Fail-secure behavior
✓ Governance observability
✓ Incident integration
✓ CI/CD for policies
✓ Canary rollout
✓ Governance testing
✓ Negative testing
✓ Chaos testing
✓ Governance ownership
✓ Compliance evidence
165. Principio Rector de E55

Data Access Governance garantiza que todo acceso a datos en EVOXA sea explícito, autorizado, limitado, contextual, trazable y revisable, independientemente de si el acceso se realiza mediante una base de datos, API, cache, búsqueda, réplica, read model, dataset virtual o cualquier otra implementación de E54.

La secuencia conceptual queda:

E49 — DATA MIGRATION
        │
        ▼
E50 — DATA SYNCHRONIZATION
        │
        ▼
E51 — DATA REPLICATION
        │
        ▼
E52 — DATA FEDERATION
        │
        ▼
E53 — DATA VIRTUALIZATION
        │
        ▼
E54 — DATA ACCESS ABSTRACTION
        │
        ▼
E55 — DATA ACCESS GOVERNANCE

La distinción esencial:

E53
→ ¿Cómo presentamos datos distribuidos como una superficie lógica?

E54
→ ¿Cómo accede EVOXA a esa superficie sin acoplarse
  a su implementación?

E55
→ ¿Quién puede acceder, a qué, para qué, bajo qué
  condiciones, durante cuánto tiempo y con qué evidencia?

E55 convierte el acceso a datos de una simple capacidad técnica en una capacidad gobernada de EVOXA.
