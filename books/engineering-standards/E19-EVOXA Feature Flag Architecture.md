E19 — EVOXA Feature Flag Architecture
Engineering Specification

Pertenece a: Engineering Specification
Volumen: volume-09-engineering
Capítulo: E19 — Feature Flag Architecture
Anterior: E18 — Configuration Architecture
Siguiente: E20 — EVOXA Runtime Policy Architecture

1. Propósito

E19 define la arquitectura de Feature Flags de EVOXA.

Su responsabilidad es controlar de forma segura y observable qué capacidades están activas, para quién, dónde y bajo qué condiciones, sin necesidad de modificar o redeployar el código para cada cambio de exposición.

La distinción fundamental es:

E18 — Configuration
    ↓
Configura el comportamiento

E19 — Feature Flags
    ↓
Controla la exposición del comportamiento

Por tanto:

Una Feature Flag controla la disponibilidad o exposición de una capacidad; no debe convertirse en un sustituto general de configuración ni de autorización.

2. Objetivos

EVOXA debe soportar:

Feature Enablement
Feature Disablement
Progressive Rollout
Tenant Rollout
Environment Rollout
Percentage Rollout
Targeted Rollout
User Targeting
Role Targeting
Region Targeting
Segment Targeting
Emergency Disable
Experimentation
Kill Switch
Flag Scheduling
Flag Versioning
Flag Auditing
Flag Evaluation
Flag Caching
Flag Propagation
Flag Lifecycle
3. Feature Flag Concept

Una Feature Flag representa una decisión:

Feature X
   ↓
Should this capability be active?
   ↓
YES / NO

Pero puede ser más compleja:

Feature X
   ↓
Who?
Where?
When?
Percentage?
Environment?
Tenant?
Context?
   ↓
Decision
4. Feature Flag vs Configuration

Debe mantenerse una separación estricta.

Configuration
timeout = 30
max_retries = 3

Responde:

¿Cómo funciona?

Feature Flag
new_checkout = enabled

Responde:

¿Está habilitada esta capacidad?

5. Feature Flag vs Authorization

Una Feature Flag no sustituye autorización.

Incorrecto:

if feature_enabled:
    allow_user()

Correcto:

if feature_enabled:
    if authorized(user):
        allow_user()

Las responsabilidades son diferentes:

E19 → Feature Exposure
E05 → Authorization
6. Feature Flag vs Policy

Una policy determina reglas:

Can actor perform action?

Una flag determina exposición:

Is capability currently exposed?

Pueden combinarse:

Feature Flag
      ↓
Capability Available
      ↓
Policy
      ↓
Authorization
7. Feature Flag Architecture
                         Feature Control Plane
                                  │
             ┌────────────────────┼────────────────────┐
             ▼                    ▼                    ▼
          Flags                Rules                Segments
             │                    │                    │
             └────────────────────┼────────────────────┘
                                  ▼
                         Flag Evaluation Engine
                                  │
                                  ▼
                           Evaluation Context
                                  │
                    ┌─────────────┼─────────────┐
                    ▼             ▼             ▼
                 Tenant         User          Runtime
                    │             │             │
                    └─────────────┼─────────────┘
                                  ▼
                           Evaluation Result
                                  │
                                  ▼
                              Application
8. Flag Definition

Una flag debería representar conceptualmente:

id
key
name
description
type
state
scope
rules
default
version
owner
created_at
updated_at
9. Flag Key

Las keys deben ser:

Stable
Unique
Deterministic
Readable
Namespaced

Ejemplo:

checkout.new_flow

o:

billing.invoices_v2
10. Flag Naming

Debe evitarse:

flag1
test123
new_feature
temporary_flag_2

si no describen claramente la capacidad.

Preferible:

checkout.new_flow
11. Flag Types

EVOXA puede soportar conceptualmente:

Boolean Flag
Multivariate Flag
Percentage Flag
Targeting Flag
Experiment Flag
Kill Switch
Release Flag
Operational Flag
Permission-adjacent Flag
12. Boolean Flag

La forma más simple:

feature = true

o:

feature = false

Adecuada para:

Enable
Disable
13. Multivariate Flag

Una flag puede devolver variantes:

variant = A
variant = B
variant = C

Ejemplo:

checkout_ui:
    control
    redesigned

Esto requiere que el consumidor conozca las variantes válidas.

14. Percentage Rollout

Una flag puede activarse para un porcentaje:

0%
 ↓
5%
 ↓
25%
 ↓
50%
 ↓
100%

Esto permite despliegues progresivos.

15. Deterministic Percentage Evaluation

El porcentaje no debe cambiar aleatoriamente en cada request.

Debe utilizarse una función determinista:

hash(subject_id + flag_key)
        ↓
bucket
        ↓
0–99

Así un usuario permanece estable en el mismo bucket durante el rollout.

16. Targeted Rollout

Puede dirigirse una flag a:

Tenant
User
Organization
Role
Region
Environment
Segment

Ejemplo:

Tenant A → enabled
Tenant B → disabled
17. Evaluation Context

La evaluación puede recibir:

tenant_id
user_id
organization_id
role
region
environment
device
application_version
runtime_version

No todos los flags necesitan todos estos atributos.

18. Context Minimization

Sólo deben utilizarse los atributos necesarios para evaluar la flag.

Esto reduce:

Privacy Risk
Complexity
Latency
Data Exposure
19. Evaluation Engine

El Evaluation Engine recibe:

Flag
+
Context

y devuelve:

Evaluation Result

Ejemplo:

enabled = true
variant = "new"
reason = "tenant-target"
version = 42
20. Evaluation Result

Una respuesta debe poder explicar:

value
variant
flag_version
rule
reason

cuando la observabilidad lo requiera.

21. Evaluation Reasons

Ejemplos:

DEFAULT
ENVIRONMENT_RULE
TENANT_RULE
USER_RULE
SEGMENT_RULE
PERCENTAGE_ROLLOUT
OVERRIDE
KILL_SWITCH

Esto facilita debugging.

22. Rule Evaluation

Una flag puede tener reglas ordenadas:

Rule 1
   ↓
Rule 2
   ↓
Rule 3
   ↓
Default

Debe existir una prioridad determinista.

23. Rule Precedence

Ejemplo:

Global
  ↓
Environment
  ↓
Tenant
  ↓
User
  ↓
Percentage
  ↓
Default

La precedencia debe estar documentada.

No debe existir ambigüedad sobre qué regla gana.

24. Default Value

Toda flag debe tener un default seguro:

default = false

especialmente para nuevas capacidades.

25. Fail-Safe Defaults

Si el sistema de flags está indisponible:

Flag Service unavailable
        ↓
Use Local Safe Default

Por ejemplo:

new_payment_flow = false
26. Fail-Open vs Fail-Closed

Debe definirse por flag.

Fail-Closed
Unavailable
    ↓
Feature OFF

Adecuado para:

Experimental Features
High-Risk Features
Incomplete Features
Fail-Open
Unavailable
    ↓
Feature ON

Puede ser apropiado para determinadas capacidades operacionales.

La decisión debe ser explícita.

27. Kill Switch

Una flag puede funcionar como emergencia:

Feature Active
      ↓
Incident
      ↓
Kill Switch
      ↓
Feature Disabled

Debe tener un camino de evaluación extremadamente sencillo y fiable.

28. Emergency Disable

Un kill switch debería poder activarse sin:

Code Deployment

y preferiblemente sin depender del componente que está siendo deshabilitado.

29. Control Plane vs Data Plane

Debe separarse:

Control Plane
    ↓
Defines Flags

Data Plane
    ↓
Evaluates Flags

Esto permite que el runtime continúe evaluando flags aunque el control plane tenga una interrupción temporal.

30. Local Evaluation

Una estrategia recomendada:

Control Plane
      ↓
Flag Configuration
      ↓
Local SDK / Evaluator
      ↓
Application

La evaluación no necesita hacer una llamada remota en cada request.

31. Remote Evaluation

Alternativamente:

Application
    ↓
Flag Service
    ↓
Evaluation

Es simple conceptualmente, pero añade:

Latency
Network Dependency
Availability Risk
32. Hybrid Evaluation

Puede utilizarse:

Remote Control Plane
        ↓
Local Flag Cache
        ↓
Local Evaluation

Esto combina:

Central Management
+
Low Runtime Latency
33. Flag Cache

E17 puede almacenar:

Flag Definitions
Evaluation Metadata
Segments
Rules

La relación es:

E19
  ↓
Semantic Ownership

E17
  ↓
Caching Mechanism
34. Flag Cache Invalidation

Cuando una flag cambia:

Flag Updated
    ↓
Configuration/Event
    ↓
Invalidate/Refresh
    ↓
Local Evaluator

La propagación debe ser observable.

35. Flag Propagation

Opciones:

Polling
Push
Event
Streaming
WebSocket
Pub/Sub

La implementación puede variar según las necesidades de EVOXA.

36. Propagation Latency

Debe medirse:

Control Plane Change
        ↓
Runtime Activation

La diferencia temporal debe ser observable:

propagation_latency
37. Stale Flags

Durante una interrupción puede existir:

Control Plane = v43
Runtime = v42

Debe conocerse la antigüedad:

flag_snapshot_age
38. Flag Snapshot

Cada runtime puede trabajar con:

Flag Snapshot v42

Esto facilita:

Consistency
Debugging
Rollback
Recovery
39. Snapshot Atomicity

Un grupo de flags relacionadas puede requerir activación atómica:

Flag A = v10
Flag B = v10
Flag C = v10

No debería observarse accidentalmente:

A = v10
B = v9
C = v10

si eso produce una combinación inválida.

40. Flag Bundle

Las flags relacionadas pueden agruparse:

checkout-v2
 ├── checkout.api_v2
 ├── checkout.ui_v2
 └── checkout.validation_v2

Un bundle puede facilitar activación coordinada.

41. Flag Dependency

Una flag puede depender de otra:

feature_B
    requires
feature_A

Pero las dependencias deben minimizarse.

42. Dependency Validation

Debe rechazarse:

A → B
B → A

para evitar ciclos.

43. Flag Composition

Puede existir una condición compuesta:

A = true
AND
B = true

pero la complejidad debe mantenerse limitada.

44. Flag Evaluation Complexity

Una flag no debería convertirse en:

if A
 and B
 and C
 and D
 and tenant
 and user
 and region
 and version
 and ...

porque termina funcionando como un motor de reglas no gobernado.

Si la lógica crece demasiado:

debe trasladarse a Policy Architecture.

45. Feature Flag Lifecycle
PROPOSED
   ↓
CREATED
   ↓
TESTING
   ↓
ROLLOUT
   ↓
ACTIVE
   ↓
DEPRECATION
   ↓
REMOVED
46. Temporary Flags

Muchas flags existen sólo durante un rollout.

Debe registrarse:

created_at
expected_removal_at
owner
purpose
47. Flag Debt

Una flag olvidada genera:

Dead Code Paths
Complexity
Testing Burden
Operational Confusion

Por eso EVOXA debe tratar las flags temporales como deuda técnica explícita.

48. Flag Expiration

Una flag puede tener:

expiration_date

Al alcanzar esa fecha:

Warning

o:

Automatic Disable

según la política.

49. Flag Cleanup

Cuando una flag deja de ser necesaria:

Flag
 ↓
Code Cleanup
 ↓
Flag Removed
 ↓
Rules Removed
 ↓
Tests Updated
50. Flag Ownership

Cada flag debe tener:

Owner
Team
Domain
Purpose
Lifecycle Status
51. Flag Audit

Los cambios deben registrar:

Who
What
When
Previous Value
New Value
Reason
Version
52. Change Reason

Cambiar:

checkout.new_flow = true

debería poder asociarse a:

reason = "production rollout"

Esto facilita incident response y governance.

53. Approval

Las flags críticas pueden requerir:

Create
 ↓
Review
 ↓
Approve
 ↓
Activate
54. Environment Flags

Las flags pueden variar por entorno:

Development → true
Staging     → true
Production  → false

Esto debe ser explícito.

55. Production Safety

Una flag activada en desarrollo no debe implicar automáticamente:

Production = enabled
56. Tenant Rollout

Un rollout puede realizarse:

Tenant A → ON
Tenant B → OFF
Tenant C → OFF

y posteriormente:

Tenant A/B → ON
57. Tenant Cohorts

Puede definirse:

Early Adopters
Beta Customers
Internal Tenants
Enterprise Tenants

como segmentos.

58. User Targeting

Puede dirigirse a usuarios específicos:

user_id ∈ allowlist

Esto es útil para:

Internal Testing
Support Cases
Early Access

pero requiere especial cuidado con privacidad y seguridad.

59. Role Targeting

Puede utilizarse:

role = internal_admin

pero:

El role targeting de una Feature Flag no sustituye al sistema de autorización.

60. Region Targeting

Puede activarse por región:

region = EU

por motivos de:

Deployment
Regulation
Infrastructure
Rollout
61. Application Version Targeting

Una flag puede aplicarse a:

application_version >= 5.2

Esto facilita migraciones graduales.

62. Client Compatibility

Una nueva feature puede requerir:

Backend
+
Frontend
+
API Version

La flag debe evitar activar una capacidad incompatible con versiones antiguas.

63. Migration Flags

Durante migraciones:

Old Path
    │
    ├── Flag OFF → Old
    │
    └── Flag ON  → New

Esto permite migraciones progresivas.

64. Dual-Run

En migraciones críticas puede ejecutarse:

Old Implementation
        │
        ├────────► Result A
        │
        └────────► Result B

y comparar resultados antes de activar la nueva implementación.

65. Experiment Flags

Las flags pueden soportar experimentos:

Control
Variant A
Variant B

pero el sistema experimental debe mantener separadas:

Exposure
Assignment
Measurement
66. Experiment Assignment

La asignación debe ser determinista cuando sea necesario:

hash(subject_id + experiment_id)

Esto evita que un usuario cambie constantemente de variante.

67. Experiment Exposure

Debe registrarse:

subject
experiment
variant
timestamp

cuando las métricas de experimentación lo requieran.

68. Experiment vs Feature Flag

Una feature flag responde:

Is feature enabled?

Un experimento responde:

Which variant should this subject receive?

No deben tratarse como conceptos idénticos.

69. Flag Evaluation API

Conceptualmente:

evaluate(
    flag_key,
    context
)

Resultado:

{
    enabled,
    variant,
    version,
    reason
}
70. Typed Evaluation

Para boolean:

is_enabled(...)

Para variantes:

get_variant(...)

El consumidor debería utilizar APIs tipadas en lugar de interpretar strings arbitrarios.

71. Default Evaluation

La API debe aceptar un default seguro:

is_enabled(
    "checkout.new_flow",
    default=false
)

Esto permite comportamiento seguro ante errores.

72. Evaluation Performance

La evaluación debe ser:

Low Latency
Predictable
Non-Blocking
Highly Available

Especialmente si se ejecuta en paths de alta frecuencia.

73. Evaluation Path

Idealmente:

Request
 ↓
Local Evaluator
 ↓
Result

y no:

Request
 ↓
Network
 ↓
Flag Service
 ↓
Database
 ↓
Response

para cada request.

74. Bulk Evaluation

Una request puede necesitar varias flags:

Flag A
Flag B
Flag C

Debe existir la posibilidad de obtenerlas eficientemente sin multiplicar innecesariamente operaciones de red.

75. Flag Evaluation Metrics

Métricas:

flag_evaluations_total
flag_evaluation_errors
flag_evaluation_latency
flag_cache_hits
flag_cache_misses
flag_snapshot_version
76. Flag Change Metrics

También:

flag_changes_total
flag_rollbacks_total
flag_activation_total
flag_deactivation_total
flag_propagation_latency
77. Observability

Cada decisión importante debe poder rastrearse:

Request
 ↓
Flag Evaluation
 ↓
Flag Version
 ↓
Rule
 ↓
Result

sin registrar innecesariamente datos personales.

78. Debug Mode

Puede existir un modo administrativo que muestre:

Flag:
checkout.new_flow

Result:
true

Reason:
tenant-rule

Version:
42

pero nunca secretos.

79. Evaluation Logging

No se recomienda registrar cada evaluación en producción si el volumen es alto.

Preferible:

Metrics
Sampling
Tracing
Targeted Debugging
80. Privacy

Los atributos usados en evaluación pueden contener:

User ID
Tenant ID
Region
Role

Deben manejarse según las políticas de privacidad correspondientes.

81. Flag Security

Sólo actores autorizados pueden:

Create Flag
Modify Flag
Activate Flag
Deactivate Flag
Delete Flag
Rollback Flag
82. Emergency Access

Los kill switches pueden requerir un camino administrativo especial:

Incident
 ↓
Authorized Operator
 ↓
Kill Switch
 ↓
Audit

La velocidad no debe eliminar la trazabilidad.

83. Flag Integrity

El runtime debe verificar que el snapshot:

Exists
Is Valid
Is Compatible
Is Authorized

antes de aplicarlo.

84. Flag Versioning

Una modificación genera:

v1
 ↓
v2
 ↓
v3

Esto permite:

Diff
Audit
Rollback
Debugging
85. Atomic Updates

Un cambio de rollout puede necesitar:

Old:
10%

New:
25%

La actualización debe ser atómica dentro del control plane.

86. Configuration Relationship

E18 y E19 pueden integrarse:

E18
Configuration
   │
   └── Flag system configuration

E19
Feature Flag State
   │
   └── Feature exposure decisions

La flag no debe quedar escondida dentro de una configuración arbitraria.

87. Cache Relationship
E19
 ↓
Flag Definition
 ↓
E17 Cache
 ↓
Local Evaluation

El cache contiene datos de flags; E19 mantiene su semántica.

88. Event Relationship
FlagChanged
    ↓
E13
    ↓
Propagation
    ↓
Flag Evaluators

Esto permite actualización rápida.

89. Messaging Relationship

E12 puede transportar eventos de distribución:

FlagChanged
FlagActivated
FlagRolledBack

cuando la topología lo requiera.

90. Job Relationship

E15 puede ejecutar:

Flag Expiration Job
Flag Cleanup Job
Flag Consistency Job
Flag Audit Job
91. Scheduling Relationship

E16 puede programar:

Activate at 09:00
Deactivate at 18:00
Start rollout tomorrow
92. Workflow Relationship

E14 puede orquestar:

Create Flag
 ↓
Validate
 ↓
Approve
 ↓
Deploy
 ↓
Observe
 ↓
Increase Rollout
 ↓
Finalize
93. Policy Relationship

E06 puede definir:

Who may modify flags?
Which flags require approval?
Which flags require dual control?
94. Authorization Relationship

E05 determina:

Can this actor modify this flag?

E19 determina:

Should this feature be exposed?
95. Multi-Tenant Architecture
Global Flag
     │
     ├── Tenant A → enabled
     ├── Tenant B → disabled
     └── Tenant C → enabled

El resultado debe respetar aislamiento de tenant.

96. Global Kill Switch

Una flag global puede sobrescribir rollout:

Global Kill Switch = OFF
        ↓
All Tenants = OFF

Debe existir una precedencia claramente definida.

97. Local Overrides

Los overrides administrativos deben estar:

Explicit
Audited
Time-Bounded
Reversible

cuando sea posible.

98. Temporary Override

Ejemplo:

Feature = OFF
Temporary Override = ON
Expires = 30 minutes

Esto evita que un workaround operacional se convierta en configuración permanente.

99. Flag Drift

Puede ocurrir:

Control Plane = 50%
Instance A = 50%
Instance B = 25%
Instance C = 50%

Debe detectarse.

100. Flag Reconciliation

El sistema puede comparar:

Desired Flag State
        ↓
Observed Runtime State

y reconciliar divergencias.

101. Disaster Recovery

El sistema debe poder reconstruir:

Flag Definitions
Rules
Versions
Segments
Assignments

desde un almacenamiento duradero.

102. Last Known Good

Si el control plane deja de estar disponible:

Current Snapshot
       ↓
Last Known Good
       ↓
Continue Evaluation

hasta que pueda recuperarse una versión nueva.

103. Stale Snapshot Policy

Debe existir una política sobre cuánto tiempo puede utilizarse un snapshot:

max_snapshot_age

Para flags críticas puede requerirse:

strict freshness

mientras que otras pueden tolerar mayor antigüedad.

104. Flag Availability

El fallo del sistema de flags no debe necesariamente derribar toda la aplicación.

Preferible:

Flag Service Failure
       ↓
Safe Local Snapshot
       ↓
Application Continues

cuando el caso de uso lo permita.

105. Flag Control Plane

Debe proporcionar:

Flag Registry
Rule Management
Segment Management
Versioning
Audit
Rollout
Rollback
Approval
106. Flag Registry

El registry mantiene:

Flag Key
Owner
Description
Status
Type
Version
Lifecycle
107. Segment Registry

Los segmentos pueden representar:

Beta Users
Enterprise Tenants
Internal Users
Region EU
Application Version >= X

Los segmentos deben ser reutilizables cuando sea apropiado.

108. Segment Evaluation
Context
  ↓
Segment Engine
  ↓
Membership
  ↓
Flag Rule
109. Segment Complexity

Los segmentos deben evitar depender de lógica arbitrariamente compleja.

Si la decisión requiere reglas de negocio complejas:

Policy Engine

puede ser más apropiado.

110. Flag Dependency Boundaries

Una flag puede depender de:

Context
Segment
Environment
Tenant

pero debería evitar depender directamente de:

Database Queries
External APIs
Long-Running Jobs

durante cada evaluación.

111. Deterministic Evaluation

Dados:

same flag version
+
same context

el resultado debe ser:

same result

salvo que la flag explícitamente dependa de tiempo o de una condición dinámica.

112. Time-Based Flags

Puede existir:

start_at
end_at

Ejemplo:

2026-10-10 09:00
        ↓
ON
        ↓
2026-10-20 18:00
        ↓
OFF
113. Time Zone

Los flags programados deben almacenar:

UTC instant

o una semántica de timezone explícita.

Nunca debe depender implícitamente del timezone del servidor.

114. Flag Evaluation During Clock Skew

Los runtimes distribuidos pueden tener relojes ligeramente diferentes.

Las flags temporales deben tolerar o controlar:

Clock Skew

cuando la precisión sea importante.

115. Percentage Rollout Stability

El bucket debe permanecer estable durante el rollout.

Por ejemplo:

User A → bucket 17
User B → bucket 73
User C → bucket 91

Si rollout = 25%:

A → ON
B → OFF
C → OFF

Al pasar a 50%:

A → ON
B → OFF
C → ON

sin reasignaciones arbitrarias.

116. Rollout Safety

El rollout debe poder avanzar:

5%
 ↓
observe
 ↓
10%
 ↓
observe
 ↓
25%

en lugar de activar inmediatamente al 100%.

117. Automated Rollout

En el futuro puede integrarse con observabilidad:

Rollout
   ↓
Metrics
   ↓
Error Rate
   ↓
Threshold
   ↓
Continue / Pause / Rollback

Esto debe estar gobernado por políticas explícitas.

118. Automatic Rollback

Una política puede definir:

IF error_rate > threshold
THEN disable feature

Pero debe existir:

Audit
Reason
Operator Visibility
Safety Limits
119. Feature Flag and Incident Response

Durante un incidente:

Incident
 ↓
Identify Feature
 ↓
Kill Switch
 ↓
Observe Recovery
 ↓
Investigate
 ↓
Rollback / Fix

Las flags son una herramienta operacional, no sólo de desarrollo.

120. Flag Contract

Cada flag debería documentar:

Flag Key
Purpose
Type
Owner
Default
Scope
Rules
Dependencies
TTL / Expiration
Fail-Safe Behavior
Security Classification
Audit Policy
Removal Plan
121. Example Contract
Feature:
checkout.new_flow

Type:
Boolean

Default:
false

Owner:
Checkout Domain

Scope:
Environment + Tenant

Rollout:
Percentage

Fail-Safe:
false

Expiration:
2026-12-31

Kill Switch:
true

Dependencies:
checkout.api_v2

Audit:
Required
122. Architectural Rules
Rule 1

Feature Flags control feature exposure, not authorization.

Rule 2

Feature Flags control feature exposure, not arbitrary configuration.

Rule 3

Every flag must have an owner.

Rule 4

Every flag must have a safe default.

Rule 5

Flag evaluation must be deterministic whenever possible.

Rule 6

Runtime evaluation should not depend on a network request per application request.

Rule 7

Flag definitions must be versioned.

Rule 8

Flag changes must be auditable.

Rule 9

Temporary flags must have an explicit removal lifecycle.

Rule 10

Kill switches must remain available during partial system failures.

Rule 11

Tenant targeting must preserve tenant isolation.

Rule 12

Flag propagation must be observable.

Rule 13

Complex business decisions belong in Policy Architecture, not hidden inside flags.

Rule 14

Flag evaluation must have explicit failure behavior.

Rule 15

E17 provides caching; E19 owns Feature Flag semantics.

123. Definition of Done

E19 queda definido cuando EVOXA dispone de:

✓ Feature Flag Model
✓ Flag Keys
✓ Flag Types
✓ Boolean Flags
✓ Multivariate Flags
✓ Percentage Rollouts
✓ Deterministic Evaluation
✓ Targeted Rollouts
✓ Evaluation Context
✓ Context Minimization
✓ Evaluation Engine
✓ Evaluation Result
✓ Evaluation Reasons
✓ Rule Evaluation
✓ Rule Precedence
✓ Safe Defaults
✓ Fail-Open / Fail-Closed
✓ Kill Switches
✓ Emergency Disable
✓ Control Plane
✓ Data Plane
✓ Local Evaluation
✓ Remote Evaluation
✓ Hybrid Evaluation
✓ Flag Cache
✓ Cache Invalidation
✓ Flag Propagation
✓ Snapshot Management
✓ Snapshot Atomicity
✓ Flag Bundles
✓ Flag Dependencies
✓ Dependency Validation
✓ Feature Flag Lifecycle
✓ Temporary Flags
✓ Flag Debt Management
✓ Flag Expiration
✓ Flag Cleanup
✓ Flag Ownership
✓ Flag Audit
✓ Approval
✓ Environment Targeting
✓ Tenant Targeting
✓ User Targeting
✓ Role Targeting
✓ Region Targeting
✓ Application Version Targeting
✓ Migration Flags
✓ Dual-Run
✓ Experiment Flags
✓ Experiment Assignment
✓ Exposure Tracking
✓ Typed Evaluation APIs
✓ Evaluation Performance
✓ Bulk Evaluation
✓ Evaluation Metrics
✓ Change Metrics
✓ Observability
✓ Privacy Controls
✓ Security Controls
✓ Flag Versioning
✓ Atomic Updates
✓ Configuration Integration
✓ Event Integration
✓ Messaging Integration
✓ Job Integration
✓ Scheduling Integration
✓ Workflow Integration
✓ Policy Integration
✓ Authorization Integration
✓ Multi-Tenant Isolation
✓ Global Kill Switch
✓ Temporary Overrides
✓ Flag Drift Detection
✓ Flag Reconciliation
✓ Disaster Recovery
✓ Last Known Good
✓ Stale Snapshot Policy
✓ Flag Registry
✓ Segment Registry
✓ Segment Evaluation
✓ Time-Based Flags
✓ Timezone Handling
✓ Clock Skew Handling
✓ Rollout Safety
✓ Automated Rollout Compatibility
✓ Automatic Rollback Compatibility
✓ Incident Response Integration
✓ Flag Contracts
124. Position in Engineering Specification

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

Y conceptualmente:

                 E18 Configuration
                         │
                         ▼
                 E19 Feature Flags
                         │
             ┌───────────┴───────────┐
             ▼                       ▼
       Feature Exposure          Rollout
             │                       │
             └───────────┬───────────┘
                         ▼
                    Application
                         │
                         ▼
                    E05 Authorization
                         │
                         ▼
                      E06 Policy

La separación queda así:

E17 decide cómo acelerar el acceso.
E18 decide cómo parametrizar el sistema.
E19 decide qué capacidades se exponen.
E05 decide quién puede utilizarlas.
E06 decide bajo qué reglas pueden utilizarse.

Siguiente capítulo: E20 — EVOXA Runtime Policy Architecture.
