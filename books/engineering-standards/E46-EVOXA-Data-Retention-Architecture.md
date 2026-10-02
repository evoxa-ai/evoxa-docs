E46 — EVOXA Data Retention Architecture
Engineering Specification

Pertenece a: Engineering Specification
Capítulo: E46 — Data Retention Architecture
Anterior: E45 — Data Lifecycle Architecture
Siguiente: E47 — EVOXA Data Disposal Architecture

1. Propósito

E46 define cuánto tiempo debe conservarse cada clase de dato de EVOXA, bajo qué condiciones, en qué estado y hasta qué momento puede ser eliminado.

El principio fundamental es:

Retention determina la obligación o necesidad de conservar un dato; lifecycle determina cómo evoluciona ese dato durante su existencia.

Por tanto:

Lifecycle ≠ Retention

Una política de lifecycle puede decir:

ACTIVE → ARCHIVED

mientras una política de retention determina:

ARCHIVED → retain until T
2. Retention Boundary

E46 cubre:

Retention Classification
Retention Policies
Retention Periods
Retention Start Events
Retention End Events
Minimum Retention
Maximum Retention
Indefinite Retention
Retention Exceptions
Legal Holds
Compliance Holds
Business Holds
Retention Extensions
Retention Overrides
Retention Calculation
Retention Enforcement
Retention Evaluation
Retention Monitoring
Retention Audit
Retention Reconciliation
Retention and Disposal Coordination

No define por sí mismo:

physical deletion mechanics
backup implementation
database implementation
storage implementation

Esas responsabilidades pertenecen a otras arquitecturas.

3. Retention vs Lifecycle

Modelo conceptual:

                   DATA LIFECYCLE
                        │
        ┌───────────────┼────────────────┐
        ▼               ▼                ▼
     ACTIVE          ARCHIVED          EXPIRED
        │               │                │
        └───────────────┼────────────────┘
                        │
                        ▼
                  RETENTION RULE
                        │
                        ▼
                 RETENTION MET
                        │
                        ▼
                  DISPOSAL ELIGIBLE

La existencia de DISPOSAL ELIGIBLE no significa necesariamente que el dato deba eliminarse inmediatamente.

4. Retention Principle

Todo dato sujeto a una política de conservación debe poder responder:

What is being retained?
Why?
For how long?
Starting from when?
Until when?
Under which policy?
Who owns the policy?
Can retention be extended?
Can retention be shortened?
What prevents disposal?
5. Retention Object

Conceptualmente:

RetentionRecord
├── entityId
├── entityType
├── tenantId
├── retentionPolicy
├── policyVersion
├── retentionClass
├── retentionStart
├── retentionEnd
├── minimumRetention
├── maximumRetention
├── holdState
├── extensionState
└── disposalEligibility
6. Retention Classes

EVOXA debe clasificar los datos por requisitos de conservación.

Ejemplo:

Operational
Transactional
Financial
Security
Audit
Compliance
Legal
Analytical
Historical
Temporary
Derived
AI
Agent

Las clases son conceptuales; cada dominio puede definir categorías adicionales.

7. Retention Policy

Una política debe definir como mínimo:

retentionClass
duration
startTrigger
endCondition
scope
owner
priority
exceptions
holdBehavior
disposalEligibility
8. Retention Period

El período puede expresarse como:

30 days
90 days
1 year
7 years
10 years
indefinite

La duración concreta debe proceder de la política correspondiente, no de una suposición del servicio.

9. Retention Start

Uno de los aspectos más importantes es determinar cuándo empieza el reloj.

Posibles triggers:

creation
activation
transaction completion
account closure
workflow completion
contract termination
last activity
event occurrence
business state transition
tenant closure
10. Retention Start ≠ Creation

Ejemplo:

Record created
    ↓
Transaction completed
    ↓
Retention clock starts

Por tanto:

createdAt != retentionStart

cuando la política lo determine así.

11. Retention End

Conceptualmente:

retentionEnd =
retentionStart + retentionPeriod

pero puede existir:

hold
extension
policy override
legal requirement
business requirement

que modifique la fecha efectiva de elegibilidad.

12. Minimum Retention

minimumRetention significa:

El dato no puede ser dispuesto antes de que se alcance el período mínimo establecido.

Modelo:

NOW < retentionEnd
      ↓
DISPOSAL BLOCKED
13. Maximum Retention

Algunas clases pueden requerir también:

maximumRetention

Esto significa que, salvo una excepción explícita válida, el dato no debería conservarse indefinidamente.

Modelo:

retentionEnd reached
        ↓
disposal required
14. Minimum vs Maximum

No deben confundirse:

Concepto	Significado
Minimum retention	No eliminar antes
Maximum retention	No conservar después
Target retention	Duración operacional esperada
Indefinite retention	Sin fecha automática de finalización
15. Indefinite Retention

Algunos datos pueden tener:

retention = INDEFINITE

Esto no significa:

ignore lifecycle

Significa que no existe una fecha automática de disposición basada exclusivamente en esa política.

16. Retention Clock

EVOXA debe representar explícitamente el reloj:

Retention Start
      │
      ▼
T0 ──────────────────────── T1
                            │
                            ▼
                     Retention End

El reloj debe ser determinista y auditable.

17. Retention Calculation

Debe existir una única interpretación autorizada de:

retentionStart
retentionDuration
retentionEnd

para cada política.

No debe permitirse que cada servicio calcule:

retentionEnd

de manera diferente.

18. Retention Policy Versioning

Las políticas deben ser versionables:

Policy v1
Policy v2
Policy v3

Esto permite responder:

Which retention rule applied when this record was created?
19. Historical Policy Preservation

Cuando una política cambia:

Old Policy
   ↓
Existing Data

no debe asumirse automáticamente que todos los datos adoptan la nueva política.

Debe existir una estrategia explícita:

grandfather
migrate
recalculate
preserve
20. Policy Migration

Una migración de retention debe seguir:

New Policy
    ↓
Impact Analysis
    ↓
Affected Records
    ↓
Migration Strategy
    ↓
Controlled Update
    ↓
Reconciliation
21. Retention Priority

Puede haber múltiples reglas aplicables:

Platform Policy
Domain Policy
Tenant Policy
Business Policy
Legal Hold
Compliance Rule

Debe existir un mecanismo determinista de precedencia.

22. Retention Precedence

Modelo conceptual:

Higher-order constraint
        ↓
Governance
        ↓
Legal / Compliance
        ↓
Platform
        ↓
Domain
        ↓
Tenant
        ↓
Operational preference

Las reglas exactas deben definirse en E06/E14 según corresponda.

23. Retention Conflict

Ejemplo:

Domain:
retain 1 year

Compliance:
retain 7 years

Resultado:

Effective Retention = 7 years

si la regla superior exige ese período.

24. Retention Cannot Arbitrarily Shorten Higher Constraints

Un servicio no debe poder hacer:

retain 7 years

→

retain 30 days

simplemente cambiando una configuración local.

25. Retention Extension

Puede existir:

retentionEnd
      ↓
extension
      ↓
newRetentionEnd

Toda extensión debe registrar:

reason
actor
policy
timestamp
previousEnd
newEnd
26. Retention Hold

Un hold suspende la elegibilidad para disposal.

RETENTION MET
      ↓
HOLD ACTIVE
      ↓
DISPOSAL BLOCKED

El dato continúa conservándose.

27. Hold Types
Legal Hold
Compliance Hold
Security Hold
Investigation Hold
Business Hold
Recovery Hold
Administrative Hold
28. Hold Semantics

Un hold no debería alterar necesariamente:

retentionStart

ni:

originalRetentionEnd

Puede simplemente bloquear:

disposal eligibility

hasta que sea liberado.

29. Hold Release

Cuando se libera:

HOLD RELEASED
      ↓
RE-EVALUATE RETENTION
      ↓
DISPOSAL ELIGIBLE?

No debe asumirse automáticamente que el dato se elimina inmediatamente.

30. Retention State Machine
             ┌─────────────┐
             │   RETAINED  │
             └──────┬──────┘
                    │
                    ▼
             ┌─────────────┐
             │  EXPIRING   │
             └──────┬──────┘
                    │
              retention end
                    │
                    ▼
          ┌───────────────────┐
          │ DISPOSAL ELIGIBLE │
          └─────────┬─────────┘
                    │
          ┌─────────┴─────────┐
          ▼                   ▼
       HOLD                DISPOSE
          │
          ▼
    RETENTION BLOCKED
          │
          ▼
    HOLD RELEASED
          │
          └──────────────► RE-EVALUATE
31. Disposal Eligible

DISPOSAL_ELIGIBLE significa:

retention requirement satisfied

No significa necesariamente:

data physically deleted

La disposición pertenece al proceso posterior.

32. Retention and Lifecycle

La relación correcta es:

Lifecycle
   │
   ├── ACTIVE
   ├── ARCHIVED
   └── EXPIRED
          │
          ▼
Retention Evaluation
          │
          ▼
Disposal Eligibility

Lifecycle no debe inferir retention.

33. Retention and Archive

Archiving no implica que retention haya terminado.

Ejemplo:

ACTIVE
  ↓
ARCHIVED
  ↓
retained 7 years
  ↓
DISPOSAL ELIGIBLE
34. Retention and Expiration

Un dato puede estar:

EXPIRED

pero seguir retenido.

Por ejemplo:

Session expired

no implica:

session data immediately deleted
35. Retention and Deletion

El orden conceptual es:

Lifecycle
    ↓
Retention evaluation
    ↓
Retention satisfied
    ↓
Disposal eligibility
    ↓
Deletion / Purge
36. Retention and Backup

La existencia de backups no debe interpretarse automáticamente como una nueva retención funcional.

Debe distinguirse:

Business retention

de:

Backup retention

E42 define cómo se conserva el backup.

E46 define cuánto tiempo debe conservarse el dato como requisito de retención.

37. Retention and Recovery

Antes de eliminar definitivamente un dato, EVOXA debe comprender:

recovery requirements

para evitar que una política de retention cree una ventana de recuperación incompatible con las necesidades del sistema.

38. Retention and Replicas

Una entidad puede existir en:

Primary
Replica
Read Model
Search
Cache
Archive
Backup
Analytics

La retención lógica debe distinguirse de las copias técnicas.

39. Retention of Derived Data

Los datos derivados pueden tener políticas diferentes:

Source:
7 years

Search index:
until source disposal

Cache:
minutes

Analytics:
3 years

AI artifact:
policy-specific

La política debe ser explícita para cada clase.

40. Retention Dependency

Si:

Derived Data

es necesario para reconstruir:

Authoritative Data

su retención puede necesitar coordinarse con la del source.

41. Rebuildable Data

Si un artefacto puede reconstruirse completamente:

source
   +
events

puede tener una retención independiente.

Esto debe documentarse.

42. Non-Rebuildable Data

Si no puede reconstruirse:

non-rebuildable artifact

su retention debe evaluarse como dato independiente.

43. Retention and AI

Las representaciones AI pueden incluir:

Embeddings
Vector records
Agent memory
Prompt artifacts
Evaluation records
AI-generated derivatives

Cada una debe tener:

retentionPolicy

cuando corresponda.

44. AI Deletion Dependency

Eliminar:

Source Data

puede requerir reevaluar:

Embedding
Vector Index
Agent Memory
Derived AI Artifact

según la política de datos derivada.

45. Retention and Multi-Tenancy

La política debe aplicarse dentro del tenant boundary:

tenantId
+
retentionPolicy
+
entity scope

Nunca debe existir una consulta de retention que accidentalmente mezcle tenants.

46. Tenant-Specific Retention

Puede existir:

Tenant A → 1 year
Tenant B → 3 years

cuando la arquitectura y la política superior lo permitan.

47. Tenant Closure

La terminación de un tenant debe activar un retention evaluation plan:

Tenant Closure
      ↓
Inventory
      ↓
Retention Evaluation
      ↓
Holds
      ↓
Retention Constraints
      ↓
Disposal Eligibility
48. Retention Inventory

EVOXA debe poder producir:

Retention Inventory

con:

entity
tenant
class
policy
start
end
hold
status
disposal eligibility
49. Retention Registry

Puede existir un:

Retention Policy Registry

que mantenga:

policyId
policyVersion
scope
classification
duration
startTrigger
priority
exceptions
status
50. Retention Evaluation Engine

Responsabilidades:

load policy
identify applicable policy
calculate retention window
evaluate holds
evaluate exceptions
calculate eligibility
emit decision
51. Retention Decision

El resultado debe ser explícito:

RETAIN
EXTEND
BLOCKED
DISPOSAL_ELIGIBLE
INDEFINITE
POLICY_ERROR
52. Retention Decision Example
Entity:
invoice-123

Policy:
FINANCIAL-7Y

Retention Start:
2025-04-01

Retention End:
2032-04-01

Hold:
NONE

Decision:
RETAIN
53. Retention Error State

Si falta una política:

NO_RETENTION_POLICY

no debería interpretarse como:

DELETE

El comportamiento seguro por defecto es:

retain
+
alert
+
policy resolution

cuando el dominio lo requiera.

54. Fail-Safe Retention

Para datos críticos:

La incertidumbre de retention debe favorecer la conservación, no la destrucción.

Especialmente cuando:

policy missing
policy conflict
hold status unknown
dependency unknown
55. Retention Enforcement

La enforcement puede producir:

retention metadata
retention locks
policy-controlled disposal queues
hold records

No debe depender únicamente de disciplina operacional.

56. Retention Scheduler

Los registros próximos a cumplir su período pueden ser procesados mediante:

Scheduler
   ↓
Retention Evaluation
   ↓
Eligible Queue
   ↓
Disposal Workflow
57. Retention Nearing Expiry

Puede existir una ventana:

EXPIRING_SOON

para anticipar:

review
hold
extension
disposal planning
58. Retention Monitoring

Métricas:

recordsRetained
recordsExpiringSoon
recordsEligibleForDisposal
retentionExtensions
activeHolds
blockedDisposals
policyConflicts
missingPolicies
retentionEvaluationFailures
59. Retention Compliance Metrics

También:

Retention Compliance Rate
Over-retention Count
Under-retention Count
Policy Coverage
Hold Coverage
Disposal SLA
Retention Decision Latency
60. Over-Retention

Un dato puede permanecer demasiado tiempo:

retentionEnd passed
+
no valid hold
+
not disposed

Resultado:

OVER_RETENTION

Esto debe ser detectable.

61. Under-Retention

Más grave:

data disposed
before retentionEnd

Resultado:

UNDER_RETENTION

Debe tratarse como una violación crítica cuando el dato esté sujeto a una obligación de conservación.

62. Retention Reconciliation

El sistema debe poder encontrar:

record
actual state
policy expected state
difference

Ejemplo:

Expected:
RETAIN

Actual:
DISPOSED

Result:
RETENTION VIOLATION
63. Retention Audit

Cada decisión relevante debería registrar:

entityId
policyId
policyVersion
evaluationTime
retentionStart
retentionEnd
holdState
decision
reason
evaluator
64. Retention Explainability

Debe ser posible responder:

¿Por qué este dato todavía se conserva?

Y obtener:

Policy: FINANCIAL-7Y
Start: 2025-04-01
End: 2032-04-01
Hold: none
Decision: RETAIN
65. Disposal Explainability

También:

¿Por qué este dato puede eliminarse?

Ejemplo:

Retention:
satisfied

Hold:
none

Policy:
FINANCIAL-7Y

Decision:
DISPOSAL_ELIGIBLE
66. Retention Exceptions

Las excepciones deben ser explícitas.

Retention Policy
      │
      ├── Normal Rule
      │
      └── Exception
             ├── Hold
             ├── Extension
             └── Override
67. Retention Override

Un override manual debe ser:

authorized
audited
time-bounded
reasoned

cuando sea permitido.

Nunca debe convertirse en:

permanent hidden exception
68. Retention Override Metadata
overrideId
entityId
previousPolicy
newPolicy
actor
reason
createdAt
expiresAt
approval
69. Retention Policy Lifecycle

Las propias políticas también tienen lifecycle:

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
70. Policy Activation

Una política nueva no debería afectar datos simplemente porque existe en una tabla.

Debe pasar por:

definition
validation
approval
activation

según governance.

71. Policy Deprecation

Una política deprecated puede dejar de aplicarse a nuevos datos mientras continúa gobernando datos existentes.

Esto debe ser explícito.

72. Retention and Configuration

Retention no debe depender exclusivamente de:

environment variable

o:

application config

para reglas críticas.

Debe existir una fuente de autoridad controlada.

73. Retention and Feature Flags

Un feature flag no debe alterar silenciosamente:

retention period

sin un control de governance adecuado.

Retention es policy, no una simple feature toggle.

74. Retention API

Conceptualmente:

GET /retention/{entityId}
GET /retention/{entityId}/decision
POST /retention/{entityId}/hold
DELETE /retention/{entityId}/hold
POST /retention/{entityId}/extension

La API exacta pertenece a E03.

75. Retention Commands

Internamente:

EvaluateRetention
PlaceRetentionHold
ReleaseRetentionHold
ExtendRetention
RecalculateRetention
MarkDisposalEligible
ReconcileRetention
76. Retention Events

Eventos posibles:

RetentionStarted
RetentionUpdated
RetentionExtended
RetentionHoldPlaced
RetentionHoldReleased
RetentionExpired
DisposalEligibilityGranted
RetentionViolationDetected
77. Event Contract

Cada evento debe incluir cuando sea necesario:

eventId
entityId
tenantId
retentionPolicy
policyVersion
retentionStart
retentionEnd
holdState
decision
occurredAt
correlationId
78. Idempotency

Operaciones como:

PlaceHold
ReleaseHold
ExtendRetention
EvaluateRetention

deben soportar reintentos sin producir estados corruptos.

79. Concurrency

Dos procesos pueden intentar:

Release Hold

y:

Dispose

simultáneamente.

El sistema debe garantizar que:

DISPOSAL

no se ejecute mientras el hold válido siga aplicando.

80. Atomicity Boundary

Cuando sea posible:

Retention Decision
+
Disposal Eligibility

deben producirse de forma consistente dentro del boundary apropiado.

81. Retention and Distributed Systems

La evaluación distribuida debe evitar decisiones basadas en información stale.

Ejemplo:

Worker A:
hold = NONE

Worker B:
creates HOLD

Worker A:
disposes record

Este escenario debe estar protegido mediante mecanismos de concurrencia y autoridad de decisión.

82. Retention Lock

Para operaciones críticas puede existir un:

Retention Lock

que impida:

disposal

durante una ventana determinada.

83. Retention Batch Processing

Para millones de registros:

Scan
 ↓
Partition
 ↓
Evaluate
 ↓
Mark Eligible
 ↓
Queue Disposal

Debe soportar:

checkpoint
resume
retry
rate limiting
84. Retention Backlog

Debe medirse:

eligibleCount
evaluationBacklog
disposalBacklog
oldestEligibleRecord
85. Retention SLA

Puede definirse:

"Eligible data must enter disposal workflow
within X hours after retention expiry."

El SLA exacto pertenece a la política operacional.

86. Retention and Capacity

La sobre-retención genera:

storage growth
backup growth
index growth
analytics growth
cost growth

Por ello E46 debe alimentar las previsiones de capacidad.

87. Retention Forecasting

Puede calcularse:

Current retained volume
+
Expected incoming volume
-
Expected disposal volume
=
Projected storage requirement
88. Retention Cost

El sistema puede atribuir:

retention cost

por:

tenant
data class
storage tier
region
retention policy

cuando el modelo comercial lo requiera.

89. Retention and Data Residency

Si existen restricciones geográficas:

retention policy
+
data residency

deben coordinarse.

Mover datos a archive storage no debe violar las restricciones de ubicación aplicables.

90. Retention and Encryption

Retained data puede requerir:

encryption at rest
key lifecycle
access controls

La retención no elimina los requisitos de seguridad.

91. Retention and Key Lifecycle

Si los datos retenidos están cifrados, la disponibilidad durante el período de retention puede depender del lifecycle de las claves.

Debe evitarse:

data retained
+
key destroyed prematurely

cuando eso haga imposible cumplir la retención.

92. Retention and Access

Retener no significa mantener acceso operativo completo.

Puede existir:

ACTIVE + full access
ARCHIVED + restricted access
RETAINED + audit access
93. Retention and Privacy

Una política de retención debe equilibrar:

retain when required

con:

dispose when no longer required

No debe interpretarse retention como justificación para conservar datos indefinidamente.

94. Data Minimization

Principio:

Retain only what is required, for only as long as required.

Esto reduce:

risk
cost
attack surface
operational complexity
95. Retention and Data Minimization

La arquitectura debe evitar:

"keep everything forever"

como estrategia por defecto.

96. Retention Registry Example
RETENTION_POLICY
-----------------------------------------
policy_id
policy_version
data_class
scope
duration
start_trigger
minimum_retention
maximum_retention
hold_allowed
extension_allowed
priority
status
owner
created_at
activated_at
retired_at
97. Retention Evaluation Record
RETENTION_EVALUATION
-----------------------------------------
evaluation_id
entity_id
tenant_id
policy_id
policy_version
retention_start
retention_end
hold_state
decision
reason
evaluated_at
next_evaluation_at
98. Retention Hold Record
RETENTION_HOLD
-----------------------------------------
hold_id
entity_id
tenant_id
hold_type
reason
created_by
created_at
released_by
released_at
status
99. Reference Architecture
                 ┌─────────────────────────┐
                 │ GOVERNANCE / POLICIES   │
                 └────────────┬────────────┘
                              │
                              ▼
                 ┌─────────────────────────┐
                 │ RETENTION POLICY        │
                 │ REGISTRY                │
                 └────────────┬────────────┘
                              │
                              ▼
                 ┌─────────────────────────┐
                 │ RETENTION ENGINE        │
                 └────────────┬────────────┘
                              │
          ┌───────────────────┼──────────────────┐
          ▼                   ▼                  ▼
      Scheduler             Holds            Events
          │                   │                  │
          └───────────────────┼──────────────────┘
                              ▼
                 ┌─────────────────────────┐
                 │ RETENTION DECISION      │
                 └────────────┬────────────┘
                              │
                    ┌─────────┴─────────┐
                    ▼                   ▼
                 RETAIN          DISPOSAL ELIGIBLE
                    │                   │
                    │                   ▼
                    │             E47 Disposal
                    │
                    ▼
               Monitoring
100. Retention Decision Flow
             Entity
                │
                ▼
       Identify Data Class
                │
                ▼
       Resolve Retention Policy
                │
                ▼
       Calculate Retention Window
                │
                ▼
          Check Holds
                │
                ▼
       Check Exceptions
                │
                ▼
       Evaluate Retention
          ┌─────┴─────┐
          ▼           ▼
       RETAIN     ELIGIBLE
          │           │
          │           ▼
          │       Disposal Flow
          │
          ▼
      Re-evaluate
101. Retention Control Plane
                 RETENTION CONTROL PLANE

       ┌──────────┐  ┌──────────┐  ┌──────────┐
       │ Policies │  │  Holds   │  │ Overrides│
       └────┬─────┘  └────┬─────┘  └────┬─────┘
            │             │             │
            └─────────────┼─────────────┘
                          ▼
                 ┌─────────────────┐
                 │ Retention Engine│
                 └────────┬────────┘
                          │
             ┌────────────┼────────────┐
             ▼            ▼            ▼
         Evaluation     Events     Reconciliation
102. Retention Data Plane
Authoritative Data
       │
       ▼
Retention Metadata
       │
       ▼
Retention Decision
       │
       ├──────────────► RETAIN
       │
       └──────────────► DISPOSAL ELIGIBLE
103. Retention Invariants
RI1 — Every governed data class must have an explicit retention policy.

RI2 — Retention start must be deterministically defined.

RI3 — Retention duration must be policy-controlled.

RI4 — Retention end must be calculable and auditable.

RI5 — Minimum retention must prevent premature disposal.

RI6 — Maximum retention must prevent unauthorized indefinite retention.

RI7 — Retention policy versions must be traceable.

RI8 — Policy changes must have explicit migration semantics.

RI9 — Higher-order retention constraints cannot be silently overridden.

RI10 — Holds must be able to block disposal.

RI11 — Hold creation and release must be auditable.

RI12 — Disposal eligibility must be distinct from physical deletion.

RI13 — Retention uncertainty must fail toward preservation for protected data.

RI14 — Derived data must have explicit retention semantics.

RI15 — AI-derived data must have explicit retention semantics when applicable.

RI16 — Tenant boundaries must be preserved during retention evaluation.

RI17 — Retention decisions must be observable.

RI18 — Over-retention must be detectable.

RI19 — Under-retention must be detectable.

RI20 — Retention decisions must be reproducible.

RI21 — Retention evaluation must be idempotent.

RI22 — Retention and disposal must be concurrency-safe.

RI23 — Retention policies must not be implemented solely as uncontrolled local configuration.

RI24 — Retained data must remain recoverable for the required retention period.

RI25 — Retention must not be used as justification for unnecessary indefinite data storage.
104. Retention Completion Criteria

E46 queda completo cuando EVOXA dispone de:

✓ Retention boundary
✓ Retention classification
✓ Retention policy model
✓ Retention periods
✓ Retention start triggers
✓ Retention end calculation
✓ Minimum retention
✓ Maximum retention
✓ Indefinite retention
✓ Retention clocks
✓ Policy versioning
✓ Policy migration
✓ Policy precedence
✓ Conflict resolution
✓ Retention extensions
✓ Retention holds
✓ Hold types
✓ Hold lifecycle
✓ Disposal eligibility
✓ Retention state machine
✓ Retention engine
✓ Retention registry
✓ Evaluation records
✓ Retention decisions
✓ Fail-safe behavior
✓ Retention enforcement
✓ Scheduler integration
✓ Retention monitoring
✓ Compliance metrics
✓ Over-retention detection
✓ Under-retention detection
✓ Reconciliation
✓ Auditability
✓ Explainability
✓ Exceptions
✓ Overrides
✓ Distributed consistency
✓ Concurrency control
✓ Batch processing
✓ Backlog management
✓ SLA model
✓ Capacity integration
✓ Residency integration
✓ Encryption/key dependency
✓ Access model
✓ Privacy/minimization principles
✓ Multi-tenant support
✓ AI retention
✓ Derived-data retention
✓ Backup interaction
✓ Recovery interaction
✓ Reference architecture
✓ Retention invariants
105. Principio Rector de E46

Retention es la política que determina cuánto tiempo EVOXA debe conservar un dato y cuándo puede considerarse elegible para disposición. No define por sí misma cómo se elimina el dato; establece el límite temporal y las restricciones que deben cumplirse antes de que la disposición sea permitida.

La relación arquitectónica queda:

E43 — Data Integrity
        │
        ▼
E44 — Consistency
        │
        ▼
E45 — Data Lifecycle
        │
        │ "How does data evolve?"
        ▼
E46 — Data Retention
        │
        │ "How long must it remain?"
        ▼
E47 — Data Disposal
        │
        │ "How is it safely removed?"
        ▼
E48 — ...

E45 define la vida del dato.
E46 define cuánto debe conservarse.
E47 deberá definir cómo se ejecuta de forma segura, verificable e irreversible la disposición cuando E46 determine que el dato es elegible.
