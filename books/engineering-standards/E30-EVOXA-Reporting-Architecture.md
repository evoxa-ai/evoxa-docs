E30 — EVOXA Reporting Architecture
Engineering Specification

Pertenece a: Engineering Specification
Volumen: volume-09-engineering
Capítulo: E30 — Reporting Architecture
Anterior: E29 — Search Architecture
Siguiente: E31 — Analytics Architecture

1. Propósito

E30 define la arquitectura de Reporting de EVOXA.

Reporting establece cómo el sistema:

genera reportes;
consolida información;
transforma datos operacionales en información presentable;
produce métricas y KPIs;
soporta reportes operativos y administrativos;
genera reportes bajo demanda y programados;
controla acceso a información reportada;
exporta resultados;
mantiene trazabilidad y reproducibilidad;
gestiona versiones de reportes;
separa reporting de operaciones transaccionales y analytics.

La pregunta fundamental es:

¿Cómo convierte EVOXA datos confiables en reportes consistentes, auditables y útiles para la toma de decisiones?

2. Principio Fundamental

Reporting no debe convertirse en una segunda base de datos operacional.

Operational Systems
        │
        ▼
Reporting Read / Reporting Model
        │
        ▼
Report Definition
        │
        ▼
Report Execution
        │
        ▼
Report Result

El reporte es una representación derivada de información existente.

3. Reporting vs Query
Query
 ↓
retrieve data

mientras:

Report
 ↓
retrieve
+
aggregate
+
calculate
+
format
+
present

Un Query puede devolver:

Customer records

Un Report puede devolver:

Customer Activity Report
├── total customers
├── active customers
├── inactive customers
├── trends
└── detailed rows
4. Reporting vs Search
Search
 ↓
find relevant information
Reporting
 ↓
summarize and present information

Search optimiza:

discovery
relevance

Reporting optimiza:

accuracy
aggregation
consistency
presentation
reproducibility
5. Reporting vs Analytics
Reporting
    ↓
"What happened?"
Analytics
    ↓
"Why?"
"What might happen?"
"What should we do?"

Reporting debe proporcionar una base confiable para Analytics, pero no reemplazarla.

6. Reporting Architecture
                     REPORTING ARCHITECTURE
                              │
              ┌───────────────┼────────────────┐
              ▼               ▼                ▼
        Operational       Read Models       Analytical
           Sources                              Sources
              │               │                │
              └───────────────┼────────────────┘
                              ▼
                     Reporting Data Model
                              │
                              ▼
                       Report Definition
                              │
                     ┌────────┴────────┐
                     ▼                 ▼
                Report Engine      Scheduler
                     │                 │
                     └────────┬────────┘
                              ▼
                       Report Execution
                              │
               ┌──────────────┼──────────────┐
               ▼              ▼              ▼
             JSON           CSV/PDF        Dashboard
               │              │              │
               └──────────────┼──────────────┘
                              ▼
                           Consumer
7. Reporting Capabilities

EVOXA debe soportar conceptualmente:

Operational Reports
Management Reports
Compliance Reports
Audit Reports
Scheduled Reports
Ad-hoc Reports
Parameterized Reports
Aggregated Reports
Detail Reports
Exportable Reports
Snapshot Reports
8. Report Definition

Un reporte debe ser una definición declarativa:

ReportDefinition
├── id
├── name
├── version
├── source
├── parameters
├── dimensions
├── measures
├── filters
├── calculations
├── ordering
├── formatting
└── security
9. Report Definition vs Report Result

Debe existir una separación explícita:

ReportDefinition
        │
        ▼
ReportExecution
        │
        ▼
ReportResult

La definición describe qué producir.

El resultado representa qué se produjo.

10. Report Versioning

Un reporte puede evolucionar:

Sales Report V1
       ↓
Sales Report V2
       ↓
Sales Report V3

Las versiones deben ser identificables.

Un resultado histórico debe conservar la versión utilizada para generarlo.

11. Report Parameters

Los reportes pueden aceptar:

dateFrom
dateTo
tenant
department
status
region
customer
category

Los parámetros deben estar tipados y validados.

12. Parameter Validation

Debe validar:

type
range
allowed values
required fields
cross-field constraints
authorization scope

Ejemplo:

dateFrom <= dateTo
13. Report Filters

Los filtros pueden incluir:

equals
in
range
contains
status
date
tenant
domain

Pero deben estar restringidos al contrato del reporte.

14. Dimensions

Las dimensiones representan agrupaciones:

date
region
customer
product
department
status

Ejemplo:

Sales
 ├── Region
 │    ├── North
 │    ├── South
 │    └── West
15. Measures

Las medidas representan valores cuantificables:

count
sum
average
minimum
maximum
percentage
rate

Ejemplo:

Orders
Revenue
Average Order Value
16. Calculated Measures

Puede definir:

Revenue / Orders

como:

AverageOrderValue

Las fórmulas deben estar versionadas y ser deterministas.

17. KPI

Un KPI puede definirse:

KPI
├── name
├── value
├── target
├── threshold
├── period
└── calculationDefinition
18. KPI Semantics

Debe existir una definición canónica.

Por ejemplo:

Active Customer

no debe significar una cosa en un reporte y otra diferente en otro.

La semántica debe pertenecer a una capa común de métricas.

19. Metric Definition

Conceptualmente:

MetricDefinition
├── id
├── name
├── description
├── formula
├── source
├── dimensions
├── filters
├── unit
└── version
20. Semantic Layer

Para evitar duplicación:

Sources
   ↓
Semantic / Metric Layer
   ↓
Reports
   ↓
Dashboards
   ↓
Analytics

La semantic layer proporciona definiciones reutilizables.

21. Reporting Data Model

El modelo puede estar optimizado para:

report queries
aggregation
joins
historical analysis

No necesariamente coincide con el modelo transaccional.

22. Reporting Read Model

Puede existir:

Operational Data
       ↓
Reporting Projection
       ↓
Reporting Read Model

Esto evita ejecutar reportes pesados directamente sobre tablas operacionales.

23. Reporting Projection

La proyección puede consolidar:

Orders
Customers
Products
Payments

en una estructura específica para reporting.

24. Reporting Source

Las fuentes pueden ser:

Operational DB
Read Models
Event Streams
Data Warehouse
Data Lake
Analytics Store
External Systems

La fuente debe declararse explícitamente.

25. Source of Truth

El reporting model normalmente es derivado:

Canonical Source
      ↓
Reporting Projection
      ↓
Report

No debe convertirse en autoridad transaccional.

26. Snapshot Reporting

Algunos reportes requieren una fotografía consistente:

Snapshot
   ↓
Report

Esto permite reproducir:

"What did the system report at time T?"
27. As-Of Reporting

Debe soportarse conceptualmente:

Report as of:
2026-10-01 23:59:59

cuando el dominio requiere histórico.

28. Historical Reporting

Debe diferenciar:

current state

de:

historical state

Ejemplo:

Customer Status Today

vs.

Customer Status on June 30
29. Temporal Semantics

Los reportes deben declarar si trabajan con:

event time
transaction time
processing time
effective time
30. Time Zone

Las fechas deben considerar:

tenant timezone
user timezone
report timezone
UTC

La zona utilizada debe formar parte del contexto del reporte.

31. Period Definitions

Debe evitarse ambigüedad entre:

day
week
month
quarter
year

Ejemplo:

Fiscal Month

puede diferir de:

Calendar Month
32. Fiscal Calendars

Si el dominio lo requiere:

FiscalYear
FiscalQuarter
FiscalMonth
FiscalWeek

deben formar parte del modelo temporal.

33. Report Execution

Flujo:

Request
  ↓
Validate
  ↓
Authorize
  ↓
Resolve Definition
  ↓
Resolve Data Scope
  ↓
Execute
  ↓
Aggregate
  ↓
Format
  ↓
Validate Result
  ↓
Publish
34. Synchronous Reports

Adecuados para:

small datasets
simple queries
low latency
interactive usage
35. Asynchronous Reports

Adecuados para:

large datasets
complex aggregation
PDF generation
large exports
scheduled execution

Flujo:

Request
 ↓
Job Created
 ↓
Queued
 ↓
Executing
 ↓
Artifact Generated
 ↓
Available
36. Report Job

Puede existir:

ReportJob
├── id
├── reportDefinition
├── parameters
├── requestedBy
├── status
├── startedAt
├── completedAt
└── artifact
37. Report Job States
PENDING
QUEUED
RUNNING
COMPLETED
FAILED
CANCELLED
EXPIRED
38. Report Scheduling

Los reportes pueden ejecutarse:

hourly
daily
weekly
monthly
cron-like
event-triggered

El Scheduling Architecture definido anteriormente debe controlar la ejecución temporal.

39. Scheduled Report
Schedule
   ↓
Report Job
   ↓
Report Execution
   ↓
Artifact
   ↓
Delivery
40. Report Delivery

Puede entregar mediante:

API
UI
Email
Object Storage
Webhook
Download
Integration

Cada mecanismo debe respetar autorización.

41. Report Artifacts

Los resultados grandes pueden producir:

CSV
JSON
PDF
XLSX
Parquet

según los consumidores.

42. Artifact Storage

Para resultados grandes:

Report Engine
      ↓
Object Storage
      ↓
Artifact Reference

La API puede devolver una referencia en lugar del contenido completo.

43. Artifact Lifecycle
GENERATED
   ↓
AVAILABLE
   ↓
EXPIRED
   ↓
DELETED
44. Report Retention

Debe declararse:

artifact retention
snapshot retention
execution metadata retention
audit retention
45. Report Security

Todo reporte debe evaluarse bajo:

identity
tenant
role
permissions
data scope
field restrictions
46. Row-Level Security

Ejemplo:

Manager A
   ↓
Region A rows

Manager B
   ↓
Region B rows

La seguridad debe aplicarse en la fuente o en una capa confiable antes de la presentación.

47. Field-Level Security

Puede ocultar:

salary
internalCost
personalData
securityMetadata

según permisos.

48. Tenant Isolation

Un reporte debe tener un contexto explícito:

ReportExecution
      ↓
TenantContext
      ↓
DataScope

Nunca debe depender exclusivamente de un filtro enviado por el usuario.

49. Report Authorization

La autorización debe comprobar:

canViewReport
canExecuteReport
canExportReport
canScheduleReport
canShareReport
canViewSensitiveFields
50. Report Sharing

Compartir un reporte no debe implicar compartir automáticamente los permisos de la fuente.

Debe existir una política explícita.

51. Report Export Security

Exportar puede incrementar el riesgo.

Debe poder aplicar:

export permission
field redaction
watermarking
size limits
audit logging
52. Report Audit

Registrar:

who
what report
version
parameters
when
tenant
result status
export
delivery
53. Sensitive Parameters

Los logs no deben almacenar indiscriminadamente:

PII
secrets
tokens
credentials
sensitive filter values
54. Report Reproducibility

Un reporte importante debe poder reconstruirse con:

definition version
data source version
parameters
time context
timezone
metric versions
execution metadata
55. Deterministic Reports

Cuando el dominio lo requiera:

same inputs
+
same data snapshot
+
same definition version

debe producir:

same result
56. Report Integrity

Los resultados críticos pueden incluir:

checksum
artifact hash
definition version
generation timestamp
57. Report Metadata

Cada ejecución puede registrar:

ReportExecutionMetadata
├── executionId
├── definitionId
├── definitionVersion
├── sourceVersion
├── parametersHash
├── generatedAt
├── generatedBy
├── duration
└── resultHash
58. Report Errors

Debe distinguir:

INVALID_PARAMETERS
UNAUTHORIZED
SOURCE_UNAVAILABLE
QUERY_TIMEOUT
PROCESSING_FAILURE
ARTIFACT_FAILURE
DELIVERY_FAILURE
59. Report Timeout

Debe existir:

execution timeout
query timeout
export timeout
delivery timeout

según la etapa.

60. Report Cancellation

Los reportes asíncronos deben poder cancelarse:

QUEUED → CANCELLED
RUNNING → CANCELLED

cuando sea seguro hacerlo.

61. Report Resource Limits

Limitar:

maximum rows
maximum execution time
maximum memory
maximum export size
maximum concurrent jobs
62. Report Concurrency

Debe existir control sobre:

same report
same tenant
same user
global capacity

para evitar sobrecarga.

63. Report Queue
Report Request
      ↓
Queue
      ↓
Worker
      ↓
Report Engine

Esto desacopla solicitudes de ejecuciones pesadas.

64. Worker Isolation

Los reportes costosos pueden ejecutarse en workers separados de:

API workers
transaction workers
event processors

para evitar interferencia.

65. Reporting Resource Pools

Puede existir:

Interactive Pool
Batch Pool
Export Pool
Scheduled Pool

según escala.

66. Query Optimization

El reporting debe optimizar:

joins
aggregations
grouping
sorting
partition pruning
materialized views
precomputed metrics
67. Materialized Reporting Views

Cuando sea necesario:

Source
 ↓
Materialized View
 ↓
Report

Esto puede reducir costes de consultas repetitivas.

68. Precomputed Aggregations

Ejemplo:

Daily Sales Aggregate

en lugar de recalcular millones de eventos para cada consulta.

69. Incremental Aggregation

Puede mantener:

dailyRevenue
monthlyRevenue
customerOrderCount

mediante procesamiento incremental.

Debe existir una estrategia clara para correcciones y backfills.

70. Late Arriving Data

Si un evento llega tarde:

Event Date = yesterday
Processing Date = today

el reporting debe decidir cómo corregir el período histórico.

71. Backfill

Debe soportar:

Historical Data
      ↓
Reprocessing
      ↓
Reporting Model
      ↓
Corrected Report
72. Data Corrections

Una corrección de datos debe poder propagarse:

Source Correction
 ↓
Projection
 ↓
Aggregate
 ↓
Report
73. Report Cache

Puede cachearse:

report result
metric
aggregation
dashboard query

pero la clave debe incluir:

definition version
parameters
authorization scope
data freshness
74. Cache Invalidation

Invalidar cuando cambian:

source data
report definition
metric definition
authorization
time context

según las garantías requeridas.

75. Reporting Freshness

Debe declararse:

real-time
near-real-time
hourly
daily
monthly
76. Freshness Metadata

El reporte puede incluir:

dataAsOf
generatedAt
sourceUpdatedAt

Esto evita confundir:

report generated now

con:

data current now
77. Reporting Observability

Métricas mínimas:

report_requests_total
report_executions_total
report_failures_total
report_execution_duration
report_queue_depth
report_timeout_total
report_export_size
report_artifact_generation_time
78. Reporting Data Quality

Debe monitorizar:

row counts
null rates
duplicate rates
missing periods
unexpected values
reconciliation totals
79. Reconciliation

Para reportes financieros o críticos:

Source Total
      =
Reporting Total
      =
Report Total

cuando las definiciones son equivalentes.

80. Report Quality Gates

Antes de publicar ciertos reportes:

data quality
completeness
freshness
reconciliation
authorization

deben pasar los thresholds establecidos.

81. Report Templates

La presentación puede estar basada en:

template
layout
sections
tables
charts
headers
footers

pero la lógica de datos debe permanecer separada de la presentación.

82. Presentation Layer
Report Data
    ↓
Presentation Model
    ↓
Renderer
    ↓
PDF / HTML / XLSX / UI
83. Report Renderer

Debe ser independiente del motor de datos:

Report Engine
     ↓
Canonical Report Model
     ↓
Renderer
84. Canonical Report Model

Puede contener:

ReportModel
├── metadata
├── sections
├── tables
├── metrics
├── charts
└── narrative

Esto permite múltiples formatos.

85. Charts

Los gráficos deben derivarse de datos definidos:

Chart
├── type
├── dimensions
├── measures
└── configuration

No deben contener lógica duplicada de negocio.

86. PDF Generation

PDF es una representación:

Report Model
 ↓
PDF Renderer
 ↓
PDF Artifact

No debe modificar los datos del reporte.

87. CSV Export

CSV es adecuado para:

flat tabular data
machine processing
large exports

Debe especificar:

encoding
delimiter
header behavior
escaping
locale
88. XLSX Export

Puede soportar:

multiple sheets
formatting
formulas
tables

pero los límites del formato deben considerarse.

89. API Report Response

Una respuesta interactiva puede ser:

{
  reportId,
  version,
  generatedAt,
  dataAsOf,
  metrics,
  rows,
  metadata
}

El contrato exacto pertenece a API Architecture.

90. Pagination in Reports

Los reportes de detalle pueden necesitar:

cursor
page
offset
streaming

Los reportes agregados normalmente tienen requisitos distintos.

91. Streaming Reports

Para grandes datasets:

Source
 ↓
Streaming Query
 ↓
Streaming Renderer
 ↓
Artifact

evitando cargar todo en memoria.

92. Report Streaming Limits

Debe controlar:

max bytes
max rows
max duration
backpressure
consumer cancellation
93. Report Sharing

Puede soportar:

private
team
role-based
tenant-wide
public-with-token

siempre con políticas explícitas.

94. Public Report Links

Si existen enlaces públicos:

opaque token
expiration
scope
revocation
audit

son obligatorios.

95. Report Notifications

Una ejecución puede producir:

ReportReady
ReportFailed
ReportExpired

que pueden integrarse con Notification Architecture.

96. Reporting Events

Eventos posibles:

ReportRequested
ReportQueued
ReportStarted
ReportCompleted
ReportFailed
ReportCancelled
ReportPublished
ReportExpired
97. Reporting and Workflow

Un Workflow puede:

request report
wait for completion
consume artifact
deliver report

La ejecución de reporting debe permanecer desacoplada del workflow.

98. Reporting and Scheduling
Scheduler
   ↓
Report Job
   ↓
Reporting Engine

Scheduling decide cuándo.

Reporting decide qué generar.

99. Reporting and Jobs
Job System
   ↓
execution infrastructure

Reporting
   ↓
report-specific semantics

No deben confundirse.

100. Reporting and Search

Search puede alimentar reportes de descubrimiento, pero:

Search Index

no debe utilizarse como sustituto automático de una fuente analítica.

101. Reporting and Read Models

Una relación habitual:

Read Model
   ↓
Reporting Projection
   ↓
Report

pero un Read Model operacional no debe cargarse con reporting analítico pesado sin justificación.

102. Reporting and Data Architecture

Reporting depende de:

data ownership
data contracts
data quality
retention
lineage
classification

definidos en Data Architecture.

103. Data Lineage

Debe poder responder:

Report
 ↓
Metric
 ↓
Reporting Model
 ↓
Source
104. Report Lineage

Ejemplo:

Revenue KPI
 ↓
Revenue Metric V4
 ↓
Daily Revenue Aggregate
 ↓
Order Events

Esto facilita auditoría.

105. Report Governance

Cada reporte debe declarar:

owner
purpose
audience
source
metrics
security
freshness
retention
SLA
version
106. Report Ownership

El owner responde por:

definition
semantics
data quality
security
performance
lifecycle
107. Report Deprecation

Un reporte debe poder marcarse:

ACTIVE
DEPRECATED
RETIRED

La deprecación debe incluir:

replacement
migration guidance
retirement date
consumer notification
108. Report Lifecycle
DRAFT
  ↓
VALIDATED
  ↓
PUBLISHED
  ↓
ACTIVE
  ↓
DEPRECATED
  ↓
RETIRED
109. Report Template Lifecycle

Los templates también deben versionarse:

Template V1
Template V2

sin alterar retroactivamente artefactos históricos.

110. Report Compatibility

Cambiar:

metric definition
field meaning
parameter
column

puede ser un breaking change.

Debe existir una política de compatibilidad.

111. Report Contract

El contrato debe definir:

inputs
outputs
semantics
freshness
security
errors
limits
version
112. Report API Boundary
Consumer
   ↓
Report API
   ↓
Reporting Application Service
   ↓
Report Engine
   ↓
Reporting Data Model
113. Report Application Service

Responsabilidades:

validate request
authorize
resolve definition
create execution
coordinate job
return result

No debe contener detalles del renderer.

114. Report Domain Service

Cuando exista lógica de dominio:

Metric Calculation
KPI Evaluation
Business Aggregation

debe residir en servicios apropiados.

115. Report Repository

Puede almacenar:

report definitions
versions
schedules
execution metadata
artifacts

No necesariamente los datos analíticos.

116. Reporting Storage

Puede dividirse:

Definition Store
Execution Store
Reporting Data Store
Artifact Store

Cada uno tiene responsabilidades distintas.

117. Reporting Security Boundary
API
 ↓
Authorization
 ↓
Reporting Service
 ↓
Data Scope
 ↓
Reporting Store

La seguridad no debe depender únicamente del frontend.

118. Multi-Tenant Reporting

Opciones:

shared reporting model + tenant key
tenant partitions
tenant-specific reporting stores

La elección depende de:

scale
isolation
compliance
cost
119. Tenant Aggregation

Un reporte cross-tenant debe ser una capacidad explícita y altamente restringida.

Por defecto:

tenant A
≠
tenant B
120. Cross-Tenant Reporting

Cuando sea legítimo:

Authorized Scope
       ↓
Tenant Aggregation
       ↓
Report

Debe existir una política específica para ello.

121. Compliance Reporting

Los reportes regulatorios deben tener:

immutable definition
versioning
audit trail
reproducibility
data lineage
retention
122. Financial Reporting

Puede requerir:

period locking
reconciliation
adjustments
auditability
approval
123. Approval Workflow

Para reportes críticos:

Generate
   ↓
Validate
   ↓
Approve
   ↓
Publish

El reporte no debe considerarse oficial antes de la aprobación requerida.

124. Report Certification

Un reporte puede tener:

Certified
Not Certified
Deprecated

La certificación debe indicar:

certifier
date
version
scope
125. Report Quality Classification

Puede utilizar:

Operational
Informational
Management
Certified
Regulatory

para establecer diferentes controles.

126. Report Performance Tiers

Por ejemplo:

Interactive
 < 2 sec

Standard
 < 30 sec

Batch
 minutes/hours

Los valores reales deben establecerse según producto.

127. Reporting Capacity

Debe dimensionarse:

reports/day
concurrent reports
rows/report
bytes/report
scheduled jobs
peak execution windows
128. Peak Scheduling

Evitar que todos los reportes diarios comiencen simultáneamente:

00:00
00:00
00:00
00:00

Puede utilizarse distribución de schedules y capacidad controlada.

129. Backpressure

Cuando la capacidad esté saturada:

Requests
   ↓
Queue
   ↓
Controlled execution

No ejecutar ilimitadamente.

130. Reporting Resilience

Debe contemplar:

source unavailable
worker failure
queue failure
renderer failure
storage failure
delivery failure
131. Retry

Retries deben ser:

bounded
backoff
idempotent
failure-aware

No todo error debe reintentarse.

132. Report Idempotency

Una misma solicitud lógica puede generar:

same execution

cuando se utilice un idempotency key.

133. Duplicate Delivery Protection

Si un reporte se entrega por email/webhook:

deliveryId

debe evitar duplicaciones accidentales.

134. Report Artifact Security

Los artefactos deben:

encrypted at rest
access controlled
expiring where appropriate
audited
135. Encryption

Debe proteger:

report data
artifacts
sensitive parameters
stored definitions where required
136. Data Classification

Los reportes deben heredar o declarar:

public
internal
confidential
restricted

según la clasificación de los datos.

137. Report Watermarking

Para determinados reportes:

CONFIDENTIAL
Tenant
GeneratedAt
User

puede aparecer en el artefacto.

138. Report Download Authorization

Cada descarga debe volver a validar:

identity
authorization
artifact ownership
expiration

No asumir que haber generado el reporte garantiza acceso permanente.

139. Reporting Observability Trace

Una ejecución puede rastrearse:

Request
 ↓
Authorization
 ↓
Job
 ↓
Query
 ↓
Aggregation
 ↓
Renderer
 ↓
Storage
 ↓
Delivery

con un correlation/trace ID.

140. Reporting Alerts

Alertar por:

execution failures
latency spikes
queue backlog
freshness violations
data quality failures
storage exhaustion
141. Reporting SLO

Ejemplo:

Interactive reports:
P95 < 2s

Scheduled report completion:
99% before delivery deadline

Freshness:
< 15 min

Los valores finales deben definirse por producto.

142. Reporting Testing

Debe cubrir:

unit
integration
contract
security
performance
data quality
snapshot
reproducibility
143. Golden Reports

Debe existir un conjunto de reportes de referencia:

Golden Dataset
     ↓
Golden Report

para detectar regresiones.

144. Snapshot Testing

Especialmente útil para:

PDF
XLSX
HTML
formatted reports

La estructura debe poder compararse de manera estable.

145. Metric Testing

Cada métrica debe probar:

normal
empty
null
boundary
large values
negative values
late data
duplicate data

cuando corresponda.

146. Security Test Matrix

Debe probar:

Tenant A → Tenant A data
Tenant A ↛ Tenant B data

User A → permitted fields
User A ↛ restricted fields
147. Performance Testing

Debe medir:

small report
medium report
large report
peak concurrency
long-running aggregation
large export
148. Failure Testing

Debe simular:

worker crash
source timeout
storage unavailable
queue delay
partial data
renderer failure
149. Disaster Recovery

Debe definir:

RPO
RTO
report definition backup
execution metadata backup
artifact backup
rebuild strategy
150. Reporting Architecture Principles

EVOXA debe mantener:

1. Reporting is derived.
2. Report definitions are versioned.
3. Metrics have canonical semantics.
4. Security applies before presentation.
5. Heavy reports are asynchronous.
6. Reports must be observable.
7. Critical reports must be reproducible.
8. Historical semantics must be explicit.
9. Artifacts have controlled lifecycle.
10. Reporting must not overload transactional systems.
11. Reporting models should be optimized for reporting.
12. Data lineage must be traceable.
13. Tenant isolation is mandatory.
14. Export is a privileged capability.
15. Report lifecycle must be governed.
151. Definition of Done

E30 queda definido cuando EVOXA dispone de:

✓ Reporting definition
✓ Reporting boundary
✓ Reporting vs Query
✓ Reporting vs Search
✓ Reporting vs Analytics
✓ Reporting architecture
✓ Report definitions
✓ Report results
✓ Versioning
✓ Parameters
✓ Validation
✓ Filters
✓ Dimensions
✓ Measures
✓ Calculated measures
✓ KPIs
✓ Metric definitions
✓ Semantic layer
✓ Reporting data model
✓ Reporting read model
✓ Reporting projections
✓ Source definition
✓ Snapshot reporting
✓ As-of reporting
✓ Historical reporting
✓ Temporal semantics
✓ Time zones
✓ Fiscal calendars
✓ Report execution
✓ Synchronous reports
✓ Asynchronous reports
✓ Report jobs
✓ Scheduling
✓ Delivery
✓ Artifacts
✓ Artifact storage
✓ Artifact lifecycle
✓ Retention
✓ Security
✓ Row-level security
✓ Field-level security
✓ Tenant isolation
✓ Authorization
✓ Export security
✓ Audit
✓ Reproducibility
✓ Deterministic reporting
✓ Integrity
✓ Metadata
✓ Errors
✓ Timeouts
✓ Cancellation
✓ Resource limits
✓ Concurrency
✓ Queues
✓ Worker isolation
✓ Query optimization
✓ Materialized views
✓ Precomputed aggregates
✓ Incremental aggregation
✓ Late-arriving data
✓ Backfills
✓ Corrections
✓ Caching
✓ Freshness
✓ Observability
✓ Data quality
✓ Reconciliation
✓ Templates
✓ Presentation model
✓ Renderers
✓ PDF
✓ CSV
✓ XLSX
✓ Streaming
✓ Sharing
✓ Notifications
✓ Reporting events
✓ Workflow integration
✓ Search integration
✓ Read Model integration
✓ Data lineage
✓ Governance
✓ Ownership
✓ Deprecation
✓ Compatibility
✓ API boundary
✓ Application services
✓ Domain services
✓ Repositories
✓ Storage separation
✓ Multi-tenancy
✓ Cross-tenant controls
✓ Compliance reporting
✓ Financial reporting
✓ Approval
✓ Certification
✓ Capacity planning
✓ Resilience
✓ Retry
✓ Idempotency
✓ Artifact protection
✓ Encryption
✓ Classification
✓ Watermarking
✓ Traceability
✓ Alerts
✓ SLOs
✓ Testing
✓ Golden reports
✓ Snapshot testing
✓ Metric testing
✓ Security testing
✓ Performance testing
✓ Failure testing
✓ Disaster recovery
152. Position in Engineering Specification

La cadena queda ahora:

E23 — Serialization Architecture
        ↓
E24 — Transformation Architecture
        ↓
E25 — Mapping Architecture
        ↓
E26 — Projection Architecture
        ↓
E27 — Query Architecture
        ↓
E28 — Read Model Architecture
        ↓
E29 — Search Architecture
        ↓
E30 — Reporting Architecture
        ↓
E31 — Analytics Architecture

Y conceptualmente:

                         DATA
                          │
             ┌────────────┼────────────┐
             ▼            ▼            ▼
          QUERY         SEARCH      REPORTING
             │            │            │
        retrieval      discovery    aggregation
             │            │            │
             ▼            ▼            ▼
        Read Model     Indexes      Report Models
                                      │
                                      ▼
                                  REPORT RESULT
                                      │
                         ┌────────────┼────────────┐
                         ▼            ▼            ▼
                       API          Export       Dashboard

La frontera con el siguiente capítulo queda:

E29 Search
    │
    │ "Find information"
    ▼
E30 Reporting
    │
    │ "Summarize and present information"
    ▼
E31 Analytics
    │
    │ "Understand, explain and derive insights"
    ▼

Principio central de E30:
Reporting es una capacidad derivada, gobernada y reproducible que transforma datos confiables en información agregada y presentable, manteniendo separación entre definición, ejecución, datos, seguridad y presentación. Los reportes críticos deben ser versionables, auditables, trazables y reproducibles.

Siguiente: E31 — EVOXA Analytics Architecture.
