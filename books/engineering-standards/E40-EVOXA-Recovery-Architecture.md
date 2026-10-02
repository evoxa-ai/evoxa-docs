E40 — EVOXA Recovery Architecture
Engineering Specification

Pertenece a: Engineering Specification
Capítulo: E40 — Recovery Architecture
Anterior: E39 — Fault Tolerance Architecture
Siguiente: E41 — EVOXA Disaster Recovery Architecture

1. Propósito

E40 define cómo EVOXA recupera, reconstruye, reconcilia y devuelve a operación normal sus componentes, estados, ejecuciones y servicios después de un fallo.

La distinción fundamental:

E39 permite que EVOXA continúe funcionando durante el fallo. E40 define cómo EVOXA vuelve a un estado operativo correcto después del fallo.

Por tanto:

E38 → Resilience
E39 → Fault Tolerance
E40 → Recovery
E41 → Disaster Recovery
2. Recovery Boundary

E40 cubre:

Failure Detection
Recovery Decision
Recovery Orchestration
State Restoration
Checkpoint Recovery
Replay
Reconciliation
Repair
Rebuild
Rehydration
Failback
Recovery Verification
Service Reintegration
Return to Normal

No sustituye:

E39 → Fault Tolerance
E41 → Disaster Recovery
E42 → Backup & Restore
3. Recovery Model

El modelo base:

                FAILURE
                   │
                   ▼
              STABILIZE
                   │
                   ▼
               ASSESS
                   │
                   ▼
              RECOVER
                   │
                   ▼
             RECONCILE
                   │
                   ▼
              VALIDATE
                   │
                   ▼
             REINTEGRATE
                   │
                   ▼
            NORMAL OPERATION
4. Recovery Objective

Recovery debe conseguir:

Correct State
+
Operational Continuity
+
Consistency
+
Integrity
+
Controlled Reintegration

No basta con:

process restarted

El proceso puede reiniciar correctamente mientras su estado permanece incorrecto.

5. Recovery Dimensions

EVOXA debe poder recuperar:

Process
Worker
Service
Execution
Workflow
Job
Queue
Event Consumer
Projection
Cache
Database Connection
Runtime State
Agent State
Configuration
Domain State
6. Recovery Classes

Cada componente debe declarar su estrategia:

RESTART
RESUME
RESTORE
REPLAY
REBUILD
RECONCILE
FAILBACK
MANUAL_RECOVERY
7. Restart Recovery

La estrategia más simple:

Process
  ↓
Failure
  ↓
Restart
  ↓
Health Check
  ↓
Ready

Es apropiada cuando el estado es:

stateless
ephemeral
reconstructable
8. Resume Recovery

Para procesos largos:

Execution
   ↓
Checkpoint 42
   ↓
Failure
   ↓
Resume from 42

Evita repetir todo el trabajo.

9. Restore Recovery

Cuando el estado debe recuperarse desde una fuente persistente:

Failure
  ↓
Load Durable State
  ↓
Validate
  ↓
Restore
  ↓
Continue
10. Replay Recovery

Cuando existe un historial durable:

Event Log
   ↓
Replay
   ↓
Reconstruct State

Aplicable a:

projections
read models
derived state
event consumers
11. Rebuild Recovery

Para estado derivado:

Source of Truth
      ↓
Rebuild
      ↓
Derived State

Ejemplos:

Search Index
Projection
Cache
Analytics Model
12. Reconciliation Recovery

Cuando existe incertidumbre:

Unknown
   ↓
Compare Sources
   ↓
Determine Actual State
   ↓
Resolve Difference
13. Failback

Después de utilizar un sistema secundario:

Primary
   ↓
Failure
   ↓
Secondary
   ↓
Recovery
   ↓
Primary restored
   ↓
Synchronize
   ↓
Failback

El failback nunca debe ocurrir simplemente porque el primary volvió a estar online.

14. Recovery State Machine

Cada componente recuperable debe tener estados:

HEALTHY
   ↓
FAILED
   ↓
RECOVERING
   ↓
SYNCING
   ↓
VALIDATING
   ↓
READY
   ↓
REINTEGRATED

Con posibles estados:

DEGRADED
QUARANTINED
RECOVERY_FAILED
MANUAL_INTERVENTION
15. Recovery Lifecycle
1. Detect
2. Stabilize
3. Classify
4. Assess
5. Select Strategy
6. Recover
7. Synchronize
8. Reconcile
9. Validate
10. Reintegrate
11. Observe
12. Close Recovery
16. Stabilization

Antes de recuperar:

stop cascading failure
stop duplicate ownership
protect durable state
preserve evidence

Ejemplo:

Failed Worker
     ↓
revoke lease
     ↓
prevent duplicate execution
     ↓
recover execution
17. Recovery Assessment

Debe determinar:

failureScope
affectedComponents
affectedState
dataIntegrity
replicationStatus
checkpointAvailable
replayAvailable
externalEffects
18. Recovery Decision

EVOXA debe seleccionar una estrategia:

if stateless:
    restart

if checkpoint:
    resume

if durable state:
    restore

if event history:
    replay

if derived:
    rebuild

if uncertain:
    reconcile
19. Recovery Policy

Cada recovery debe tener:

recoveryPolicy
├── strategy
├── timeout
├── retryLimit
├── validation
├── rollback
├── escalation
└── manualOverride
20. Recovery Orchestrator

Debe existir una capacidad de coordinación:

Recovery Orchestrator
        │
        ├── detect scope
        ├── select strategy
        ├── order operations
        ├── track progress
        ├── validate
        └── finalize
21. Recovery Plan

Cada incidente debe producir un plan:

Recovery Plan
├── Incident
├── Scope
├── Dependencies
├── State Sources
├── Recovery Steps
├── Validation Steps
├── Rollback Steps
└── Completion Criteria
22. Recovery Dependency Graph

La recuperación debe respetar dependencias.

Ejemplo:

Database
    ↓
Domain Services
    ↓
Event Processing
    ↓
Projections
    ↓
Search
    ↓
Analytics

No reconstruir Search antes de disponer de su source of truth.

23. Recovery Ordering

Regla:

Recover authoritative state before derived state.

Por tanto:

Source of Truth
      ↓
Domain State
      ↓
Events
      ↓
Projections
      ↓
Search
      ↓
Analytics
24. Recovery Waves

Para sistemas grandes:

Wave 1
Critical Infrastructure

Wave 2
Core Data

Wave 3
Core Services

Wave 4
Processing

Wave 5
Derived Systems

Wave 6
Optional Systems
25. Criticality Levels

Cada componente debe clasificarse:

CRITICAL
HIGH
NORMAL
LOW
OPTIONAL

La prioridad de recovery depende de esta clasificación.

26. Recovery Priority

Ejemplo:

P0 → Identity / Authorization / Core Data
P1 → Domain Services
P2 → Execution / Messaging
P3 → Search / Analytics
P4 → Optional integrations
27. Recovery Dependencies

Cada componente debe declarar:

dependsOn
requiredFor
recoveryOrder

Esto permite construir automáticamente un DAG de recuperación.

28. Recovery Deadline

Cada recovery debe tener:

recoveryDeadline

Ejemplo conceptual:

Critical Service
RTO = 5 min

E40 implementa cómo conseguir ese objetivo.

29. Recovery Point

Debe conocerse:

lastKnownGoodState

y, cuando corresponda:

recoveryPoint
30. Recovery Point Selection

Puede basarse en:

latest checkpoint
latest committed transaction
latest replicated state
latest verified snapshot

No utilizar un estado simplemente porque sea el más reciente si no está validado.

31. Checkpoint Recovery

Modelo:

Execution
   │
   ├── CP1
   ├── CP2
   ├── CP3
   └── CP4
          ↓
       Failure
          ↓
      Restore CP4
          ↓
        Resume
32. Checkpoint Integrity

Antes de usar un checkpoint:

checksum
schema compatibility
version
ownership
timestamp
integrity

deben ser validados.

33. Stale Checkpoint

Un checkpoint puede ser demasiado antiguo:

Current = v100
Checkpoint = v70

Debe calcularse:

recovery gap

antes de utilizarlo.

34. Incremental Recovery

Cuando sea posible:

Checkpoint
   +
Event Delta
   ↓
Current State

reduce el coste de recovery.

35. Full Recovery

Si no existe delta utilizable:

Snapshot
   ↓
Restore
   ↓
Replay
   ↓
Validate
36. State Restoration

Debe distinguir:

persisted state
derived state
temporary state

Solo restaurar aquello que corresponda.

37. State Rehydration

Un componente recuperado puede necesitar:

load state
load configuration
load ownership
load subscriptions
load leases
load runtime metadata

antes de estar listo.

38. Rehydration Sequence
START
 ↓
Load Configuration
 ↓
Load Durable State
 ↓
Validate Schema
 ↓
Load Runtime Metadata
 ↓
Synchronize
 ↓
Health Check
 ↓
READY
39. Database Recovery

El recovery de base de datos puede incluir:

reconnect
replica promotion
transaction recovery
log replay
snapshot restore
consistency validation
40. Database Reconciliation

Después de recuperar:

Primary State
      ↕
Replica State
      ↕
Event State

debe verificarse que las fuentes sean coherentes.

41. Transaction Recovery

Una transacción puede estar:

COMMITTED
ABORTED
UNKNOWN

UNKNOWN requiere reconciliación.

42. Recovery of Unknown Transactions
Unknown
   ↓
Query authoritative source
   ↓
Committed?
 ┌──┴──┐
Yes    No
 ↓      ↓
Apply   Retry/Abort
43. Event Recovery

Un consumer recuperado debe conocer:

lastProcessedOffset
lastCommittedOffset
currentPartition
consumerGeneration
44. Event Replay
Event Store
     ↓
Offset N
     ↓
Replay N → Current
45. Replay Safety

Replay debe soportar:

duplicate events
idempotent handlers
version checks
side-effect protection
46. Projection Recovery
Projection Failure
       ↓
Load latest valid checkpoint
       ↓
Replay events
       ↓
Rebuild projection
       ↓
Validate
       ↓
Publish Ready
47. Search Recovery

Search index:

FAILED
  ↓
EMPTY / PARTIAL
  ↓
REBUILDING
  ↓
SYNCING
  ↓
VALIDATED
  ↓
READY

El dominio principal puede continuar durante el proceso si la arquitectura lo permite.

48. Cache Recovery

Cache recovery normalmente debe ser:

invalidate
warm
repopulate

No debe requerir restaurar manualmente cada entry salvo que exista una necesidad específica.

49. Queue Recovery

Después de un broker/consumer failure:

Durable Queue
      ↓
Recover Offsets
      ↓
Reclaim Leases
      ↓
Resume Consumption
50. Job Recovery

Un job puede estar:

QUEUED
RUNNING
FAILED
UNKNOWN
COMPLETED

Tras recovery:

RUNNING

puede requerir reconciliación para determinar si realmente terminó.

51. Workflow Recovery

Workflow recovery:

Workflow State
      ↓
Restore
      ↓
Identify last completed activity
      ↓
Resume next safe activity
52. Activity Recovery

Una activity debe tener:

activityId
attempt
status
result
sideEffectState

para evitar repetir efectos externos incorrectamente.

53. Agent Recovery

Agent runtime:

Agent Session
      ↓
Checkpoint
      ↓
Failure
      ↓
Restore Context
      ↓
Validate Pending Action
      ↓
Resume

No reemitir automáticamente una acción externa cuya ejecución anterior sea UNKNOWN.

54. AI Recovery

Si falla un modelo:

Model A
  ↓
Failure
  ↓
Model B

Pero el nuevo modelo debe recibir el contexto correcto:

state
policy
conversation/execution context
constraints
previous outputs
55. Recovery of External Integrations

Una integración puede requerir:

retry
reconciliation
resubmission
manual confirmation

No todas las operaciones externas pueden simplemente repetirse.

56. Side-Effect Recovery

Ejemplo:

Send Email
   ↓
Timeout
   ↓
Unknown

Antes de enviar nuevamente:

query provider

Si ya fue enviado:

do not duplicate
57. Recovery Idempotency

Cada operación recuperable debe declarar:

idempotent = true/false

Si es false:

requires reconciliation

antes de retry.

58. Recovery Reconciliation

La reconciliación debe comparar:

Expected State
vs
Observed State

y producir:

MATCH
MISMATCH
UNKNOWN
59. Reconciliation Sources

Las fuentes pueden incluir:

Database
Event Log
Provider API
Execution Log
Audit Log
External Confirmation
60. Reconciliation Priority

Preferencia:

Authoritative Source
        ↓
Durable State
        ↓
Committed Event
        ↓
Derived State
        ↓
Cache
61. Repair

Cuando el estado es incorrecto:

detect
 ↓
repair
 ↓
validate

Repair debe ser:

audited
authorized
idempotent

cuando sea posible.

62. Automatic vs Manual Recovery
Automatic
→ predictable, low-risk, reversible

Manual
→ ambiguous, high-impact, irreversible
63. Recovery Escalation
Auto Recovery
      ↓ failure
Retry Recovery
      ↓ failure
Alternative Strategy
      ↓ failure
Quarantine
      ↓
Manual Intervention
64. Recovery Retry

Recovery itself puede fallar.

Por tanto:

Recovery Attempt 1
Recovery Attempt 2
Recovery Attempt 3

debe estar limitado.

65. Recovery Backoff

Evitar:

failure
↓
recovery
↓
failure
↓
recovery
↓
...

utilizando:

exponential backoff
jitter
maximum attempts
66. Recovery Storm

Muchos componentes pueden intentar recuperarse simultáneamente.

Esto puede generar:

CPU spike
database overload
network overload
queue overload

Por ello recovery debe tener:

concurrency limits
priority
backpressure
67. Recovery Admission Control

Durante recuperación:

Recovery Manager
     ↓
admit only N recoveries

No recuperar todo simultáneamente.

68. Recovery Isolation

Un componente fallido puede ponerse en:

QUARANTINED

para evitar que una recuperación defectuosa contamine el sistema.

69. Quarantine
Failed Node
   ↓
Quarantine
   ↓
Diagnostics
   ↓
Repair
   ↓
Validation
   ↓
Rejoin
70. Recovery Validation

Recovery no termina con:

process = running

Debe verificarse:

health
state
consistency
dependencies
ownership
version
policy
71. Readiness Validation

Un servicio recuperado debe pasar:

Liveness
Readiness
Dependency Health
State Validation
72. Recovery Verification Levels
L1 — Process healthy
L2 — Service healthy
L3 — State valid
L4 — Dependencies valid
L5 — Functional verification
L6 — Production ready
73. Functional Recovery Test

Ejemplo:

write test
read test
event test
dependency test

según criticidad.

74. Reintegration

Un componente validado vuelve progresivamente:

RECOVERED
   ↓
CANARY
   ↓
PARTIAL TRAFFIC
   ↓
FULL TRAFFIC
75. Canary Reintegration

Primero:

1%

después:

10%
30%
50%
100%

según el sistema.

76. Reintegration Guardrails

Cada fase debe comprobar:

error rate
latency
consistency
resource usage
77. Failback Validation

Antes de volver al primary:

Primary recovered
      ↓
State synchronized
      ↓
Validated
      ↓
Canary
      ↓
Failback
78. Recovery Completion

Recovery termina cuando:

All required components
+
state valid
+
dependencies healthy
+
ownership correct
+
traffic restored
79. Recovery Closure

Debe generarse:

Recovery Record
├── incidentId
├── startTime
├── endTime
├── strategy
├── affectedComponents
├── stateRecovered
├── dataLoss
├── reconciliation
├── validation
└── finalStatus
80. Recovery Audit

Todas las acciones críticas deben registrar:

who
what
when
why
before
after
result
81. Recovery Observability

Métricas mínimas:

recoveryAttempts
recoverySuccesses
recoveryFailures
recoveryDuration
recoveryLag
recoveryQueueDepth
reconciliationFailures
repairCount
manualInterventions
failbacks
82. Recovery SLOs

Ejemplos:

Recovery Success Rate
Mean Recovery Time
P95 Recovery Time
Recovery Validation Success
Reconciliation Success
83. MTTR

E40 debe operacionalizar:

MTTR
=
Mean Time To Recovery

Pero debe distinguir:

detection time
decision time
repair time
validation time
reintegration time
84. Recovery Timeline
Failure
  │
  ├── Detection
  ├── Stabilization
  ├── Assessment
  ├── Recovery Start
  ├── State Restore
  ├── Reconciliation
  ├── Validation
  ├── Reintegration
  └── Recovery Complete
85. Recovery Cost

Recovery strategy debe considerar:

time
compute
storage
network
operator effort
data replay
external API cost
86. Recovery vs Availability

No todo recovery debe intentar maximizar availability.

En ciertos casos:

correctness > availability

Por ejemplo:

state corruption suspected

puede requerir:

pause writes
87. Recovery Safety Principle

Never restore a state that has not been validated.

88. Recovery Integrity Principle

Recovery must restore correctness, not merely process availability.

89. Recovery Authority

Solo fuentes autorizadas pueden determinar:

canonical state

Esto evita que un:

cache
stale replica
old worker

se convierta accidentalmente en source of truth.

90. Recovery Version Compatibility

Antes de restaurar:

State Schema
Runtime Version
Event Version
Configuration Version

deben ser compatibles.

91. Schema Migration Recovery

Si el estado antiguo requiere migración:

Old State
   ↓
Validate
   ↓
Migrate
   ↓
Validate
   ↓
Activate

Nunca migrar directamente sin validación intermedia.

92. Recovery Rollback

Si recovery produce un estado inválido:

Recovery
   ↓
Validation Failure
   ↓
Rollback
   ↓
Previous Known Good State
93. Recovery Checkpoints

El propio recovery puede tener checkpoints:

Recovery Step 1 ✓
Recovery Step 2 ✓
Recovery Step 3 ✗

Permite continuar o revertir sin repetir todo.

94. Recovery Plan Versioning

Los recovery plans deben ser versionados:

Recovery Plan v1
Recovery Plan v2

porque el sistema puede cambiar mientras evoluciona.

95. Recovery Automation

Las acciones repetitivas y deterministas deben automatizarse:

restart
restore
replay
rebuild
sync
validate
reintegrate

Las acciones ambiguas deben escalar.

96. Recovery Runbooks

Cada componente crítico debe disponer de:

Detection Runbook
Recovery Runbook
Validation Runbook
Rollback Runbook
Escalation Runbook
97. Recovery Simulation

EVOXA debe probar recovery mediante:

failure injection
node termination
dependency outage
network partition
storage failure
worker crash
98. Recovery Testing

Las pruebas deben validar:

can recover
can recover correctly
can recover within target
can recover repeatedly
99. Recovery Drill

Periódicamente:

simulate failure
 ↓
execute recovery
 ↓
measure
 ↓
identify gaps
 ↓
improve
100. Recovery Game Day

Un Game Day puede probar:

database failure
worker fleet failure
event broker failure
provider failure
zone failure
101. Recovery Chaos

Chaos testing puede validar:

kill
delay
partition
corrupt
disconnect
restart

pero siempre dentro de límites controlados.

102. Recovery Dependencies on E39

E39 proporciona:

redundancy
replication
leases
quorum
fencing
checkpoint
idempotency

E40 los utiliza para:

restore
resume
reconcile
reintegrate
103. Recovery Dependencies on E38

E38 define:

degradation
fallback
continuity

E40 define:

return to normal
104. Recovery Boundary with E41

E40:

component/service/system recovery

E41:

site/region/disaster recovery

Ejemplo:

Worker failure
→ E40

Database node failure
→ E40

Entire region unavailable
→ E41
105. Recovery Architecture
                    FAILURE
                       │
                       ▼
              ┌────────────────┐
              │    ASSESS      │
              └───────┬────────┘
                      ▼
              ┌────────────────┐
              │ RECOVERY PLAN  │
              └───────┬────────┘
                      ▼
        ┌─────────────┼─────────────┐
        ▼             ▼             ▼
     RESTART        RESUME        RESTORE
        │             │             │
        └─────────────┼─────────────┘
                      ▼
                  REPLAY /
                  REBUILD
                      │
                      ▼
                RECONCILIATION
                      │
                      ▼
                  VALIDATION
                      │
                      ▼
                REINTEGRATION
                      │
                      ▼
               NORMAL OPERATION
106. Recovery Control Plane
                Recovery Controller
                       │
       ┌───────────────┼───────────────┐
       ▼               ▼               ▼
   Assessment       Planning        Execution
       │               │               │
       └───────────────┼───────────────┘
                       ▼
                 Validation
                       │
                       ▼
                  Reintegration
107. Recovery State Model
HEALTHY
   │
   ▼
FAILED
   │
   ▼
STABILIZING
   │
   ▼
RECOVERING
   │
   ▼
SYNCING
   │
   ▼
VALIDATING
   │
   ├──── fail ───► RECOVERY_FAILED
   │
   ▼
CANARY
   │
   ▼
REINTEGRATED
   │
   ▼
HEALTHY
108. Recovery Contract

Cada componente crítico:

recoveryContract
├── recoveryClass
├── recoverySource
├── checkpointPolicy
├── replayPolicy
├── reconciliationPolicy
├── validationPolicy
├── rollbackPolicy
├── recoveryDeadline
├── reintegrationPolicy
└── escalationPolicy
109. Recovery Invariants
RC1 — Recovery must not create a second authoritative state.

RC2 — Recovery must preserve data integrity.

RC3 — Recovery must validate restored state before serving traffic.

RC4 — Recovery must respect dependency ordering.

RC5 — Recovery must not replay unsafe external side effects blindly.

RC6 — Unknown outcomes must be reconciled before retrying non-idempotent operations.

RC7 — Recovery must be bounded by explicit retry and timeout policies.

RC8 — Recovery must be observable.

RC9 — Recovery must be auditable.

RC10 — Recovery must preserve tenant isolation.

RC11 — Recovery must preserve authorization and policy enforcement.

RC12 — Recovery must be reversible where technically possible.

RC13 — Recovery must support degraded operation when full restoration is not yet possible.

RC14 — Recovery must distinguish authoritative state from derived state.

RC15 — Recovered nodes must synchronize before full reintegration.

RC16 — Recovery must prevent stale ownership from becoming authoritative.

RC17 — Recovery must not amplify the original failure.

RC18 — Recovery itself must tolerate partial failure.

RC19 — Recovery completion requires functional validation, not merely process health.

RC20 — Every critical recovery path must be testable.
110. Engineering Completion Criteria

E40 queda completo cuando EVOXA posee:

✓ Recovery taxonomy
✓ Restart recovery
✓ Resume recovery
✓ Restore recovery
✓ Replay recovery
✓ Rebuild recovery
✓ Reconciliation recovery
✓ Failback
✓ Recovery state machine
✓ Recovery lifecycle
✓ Stabilization
✓ Recovery assessment
✓ Recovery decision engine
✓ Recovery policies
✓ Recovery orchestrator
✓ Recovery plans
✓ Dependency graph
✓ Recovery ordering
✓ Recovery waves
✓ Criticality model
✓ Recovery priorities
✓ Recovery deadlines
✓ Recovery points
✓ Checkpoint recovery
✓ Checkpoint integrity
✓ Incremental recovery
✓ Full recovery
✓ State restoration
✓ State rehydration
✓ Database recovery
✓ Transaction recovery
✓ Event recovery
✓ Projection recovery
✓ Search recovery
✓ Cache recovery
✓ Queue recovery
✓ Job recovery
✓ Workflow recovery
✓ Agent recovery
✓ AI recovery
✓ External integration recovery
✓ Side-effect reconciliation
✓ Recovery idempotency
✓ Repair
✓ Automatic/manual recovery
✓ Escalation
✓ Recovery retry
✓ Recovery backoff
✓ Recovery storm protection
✓ Recovery admission control
✓ Quarantine
✓ Recovery validation
✓ Functional verification
✓ Reintegration
✓ Canary reintegration
✓ Failback validation
✓ Recovery closure
✓ Recovery audit
✓ Recovery observability
✓ Recovery SLOs
✓ MTTR decomposition
✓ Recovery safety
✓ Recovery integrity
✓ Recovery authority
✓ Version compatibility
✓ Migration recovery
✓ Recovery rollback
✓ Recovery checkpoints
✓ Recovery automation
✓ Recovery runbooks
✓ Recovery simulation
✓ Recovery testing
✓ Recovery drills
✓ Chaos recovery testing
✓ E38/E39/E41 boundaries
✓ Recovery contract
✓ Recovery invariants
111. Runtime Chain Consolidada

Con E37–E40, la arquitectura empieza a cerrar un ciclo operativo completo:

                         WORKLOAD
                            │
                            ▼
                    ┌───────────────┐
                    │     E37       │
                    │   CAPACITY    │
                    └───────┬───────┘
                            │
                            ▼
                    ┌───────────────┐
                    │     E38       │
                    │  RESILIENCE   │
                    └───────┬───────┘
                            │
                     failure occurs
                            ▼
                    ┌───────────────┐
                    │     E39       │
                    │FAULT TOLERANCE│
                    └───────┬───────┘
                            │
                   continue / contain
                            ▼
                    ┌───────────────┐
                    │     E40       │
                    │   RECOVERY    │
                    └───────┬───────┘
                            │
              restore / replay / reconcile
                            │
                            ▼
                    ┌───────────────┐
                    │   VALIDATED   │
                    │    STATE      │
                    └───────┬───────┘
                            │
                            ▼
                     REINTEGRATION
                            │
                            ▼
                    NORMAL OPERATION
                            │
                            ▼
                    ┌───────────────┐
                    │     E41       │
                    │   DISASTER    │
                    │   RECOVERY    │
                    └───────────────┘
Frontera conceptual

E38 — Resilience: cómo EVOXA absorbe condiciones adversas.
E39 — Fault Tolerance: cómo continúa funcionando pese al fallo.
E40 — Recovery: cómo reconstruye, reconcilia, valida y reintegra el sistema después del fallo.
E41 — Disaster Recovery: cómo recupera EVOXA frente a fallos de mayor escala, incluyendo pérdida de infraestructura, zona o región.

El siguiente capítulo lógico es E41 — EVOXA Disaster Recovery Architecture, donde la unidad de recuperación deja de ser únicamente el componente/servicio y pasa a ser la plataforma EVOXA completa frente a un desastre mayor.
