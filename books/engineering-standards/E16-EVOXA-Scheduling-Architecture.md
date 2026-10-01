E16 — EVOXA Scheduling Architecture
Engineering Specification

Pertenece a: Engineering Specification
Volumen: volume-09-engineering
Capítulo: E16 — Scheduling Architecture
Anterior: E15 — Job & Task Processing Architecture
Siguiente: E17 — EVOXA Caching Architecture

1. Propósito

E16 define la arquitectura responsable de determinar cuándo debe ejecutarse una operación dentro de EVOXA.

E15 responde:

¿Cómo ejecutamos un trabajo?

E16 responde:

¿Cuándo debe crearse o activarse ese trabajo?

La separación fundamental es:

E16 — Scheduling
        │
        │ When?
        ▼
E15 — Job Processing
        │
        │ How?
        ▼
     Worker
2. Objetivos

La arquitectura de Scheduling debe soportar:

Schedules
Timers
Delays
Intervals
Cron Expressions
Calendar Rules
One-Time Execution
Recurring Execution
Deferred Execution
Deadlines
Time Windows
Time Zones
Misfire Handling
Schedule Persistence
Schedule Recovery
Distributed Scheduling
Concurrency
Deduplication
Tenant Isolation
Observability
3. Principio Fundamental

Scheduling no ejecuta directamente la lógica empresarial.

Su responsabilidad es:

Determine Time
       ↓
Trigger Execution
       ↓
Create / Release Job

No:

Scheduler
   ↓
Business Logic
4. Scheduling vs Job Processing

La separación canónica:

                 Schedule
                    │
                    │ WHEN?
                    ▼
              Scheduler
                    │
                    ▼
                 Job
                    │
                    │ HOW?
                    ▼
                Worker
                    │
                    ▼
             Application Service
5. Scheduling vs Workflow

Un Workflow puede contener un timer:

Workflow
   ↓
WAIT 7 days
   ↓
Continue

E14 mantiene el estado del Workflow.

Scheduling proporciona la capacidad temporal necesaria para activar esa transición.

6. Scheduling vs Event

Un Event describe:

Something happened

Un Schedule describe:

Something should happen at time X

Ejemplo:

SubscriptionCreated

vs:

Every Monday at 08:00
7. Tipos de Scheduling

EVOXA debe soportar conceptualmente:

One-Time
Recurring
Delayed
Interval
Calendar-Based
Cron-Based
Event-Relative
Deadline-Based
Window-Based
8. One-Time Schedule

Ejemplo:

ExecuteAt:
2027-01-15 09:00

Flujo:

Schedule
   ↓
Time Reached
   ↓
Create Job
   ↓
Completed
9. Delayed Schedule

Ejemplo:

Now
 ↓
Delay 30 minutes
 ↓
Job

Esto es diferente de una fecha absoluta.

10. Interval Schedule

Ejemplo:

Every 30 minutes

Conceptualmente:

T0
 ↓
T0 + 30m
 ↓
T0 + 60m
 ↓
T0 + 90m
11. Recurring Schedule

Ejemplo:

Every Monday

o:

Every day at 08:00

El scheduler debe generar las siguientes ejecuciones según la definición.

12. Cron Schedule

Puede utilizarse una expresión cron:

0 8 * * 1

que representa conceptualmente:

Every Monday at 08:00

La semántica exacta debe estar definida por el motor utilizado.

13. Calendar-Based Scheduling

No todas las reglas temporales se expresan adecuadamente con cron.

Ejemplos:

Business Days
Weekdays
Last Day of Month
First Business Day
Holiday-Aware
Fiscal Calendar
14. Time Zone

Los schedules deben definir explícitamente su timezone cuando el horario tenga significado local.

Ejemplo:

08:00 America/New_York

No debe asumirse automáticamente UTC para todos los schedules.

15. UTC

Los timestamps operacionales deben almacenarse de forma consistente, preferentemente en UTC.

Pero la definición del schedule puede conservar:

timezone = America/New_York

para calcular correctamente la próxima ejecución.

16. Daylight Saving Time

Los schedules basados en timezone deben contemplar cambios DST.

Ejemplo:

08:00 local time

debe continuar significando:

08:00 en la zona configurada.

No simplemente:

el mismo offset UTC para siempre.

17. Ambiguous Time

Durante el cambio de horario pueden existir horas ambiguas.

El scheduler debe tener una política explícita:

First Occurrence
Second Occurrence
Skip
Reject

según la semántica elegida.

18. Nonexistent Time

Durante el cambio de horario puede desaparecer una hora.

Ejemplo conceptual:

02:30

puede no existir en una transición DST.

Debe existir una política:

Skip
Shift Forward
Shift Backward
Fail
19. Schedule Definition

Un Schedule debe definir conceptualmente:

schedule_id
schedule_type
expression
timezone
start_at
end_at
status
target
tenant_scope
misfire_policy
concurrency_policy
20. Schedule Instance

Una definición puede generar múltiples ejecuciones.

Schedule Definition
        │
        ├── Occurrence 001
        ├── Occurrence 002
        ├── Occurrence 003
        └── ...
21. Next Run

Cada Schedule debe poder determinar:

next_run_at

Ejemplo:

Now
 ↓
Calculate Next Occurrence
 ↓
next_run_at
22. Previous Run

Puede mantenerse:

last_run_at

para observabilidad y control operacional.

23. Schedule Lifecycle
DRAFT
   ↓
ACTIVE
   ↓
PAUSED
   ↓
ACTIVE
   ↓
DISABLED
   ↓
RETIRED
24. Schedule Creation

Un Schedule puede crearse desde:

API
Admin Interface
Workflow
System Configuration
Tenant Configuration
Application Service
25. Schedule Trigger

Cuando llega el momento:

Schedule
   ↓
Trigger
   ↓
Create Job

El Job pasa entonces a E15.

26. Scheduler Responsibilities

El Scheduler es responsable de:

Calculate Next Execution
Detect Due Schedules
Apply Time Rules
Apply Misfire Policy
Trigger Jobs
Persist Schedule State
Recover After Failure
Prevent Duplicate Triggering
27. Scheduler Non-Responsibilities

No debe ser responsable de:

Business Logic
Database Business Rules
External API Calls
Long Business Processes
Job Handler Execution
Domain Decisions
28. Scheduler Architecture
                 Schedule Store
                      │
                      ▼
               Scheduler Engine
                      │
             ┌────────┼────────┐
             ▼        ▼        ▼
          Timer A  Timer B  Timer C
             │        │        │
             └────────┼────────┘
                      ▼
                  Job Trigger
                      │
                      ▼
                  Job Queue
                      │
                      ▼
                    E15
29. Schedule Store

Debe persistirse:

Schedule Definition
Schedule State
Next Run
Last Run
Status
Execution Metadata

Esto permite recuperación después de reinicios.

30. Durable Scheduling

Un scheduler no debe depender únicamente de memoria:

Scheduler Process
      │
      ▼
In-Memory Timer

porque un restart podría perder ejecuciones.

Debe existir una fuente durable de verdad:

Scheduler
   ↓
Persistent Schedule Store
31. Distributed Scheduler

EVOXA puede ejecutar múltiples scheduler nodes:

Scheduler A
Scheduler B
Scheduler C

Todos pueden consultar el mismo Schedule Store.

Pero una ejecución debe ser reclamada por un único scheduler.

32. Schedule Claiming

Conceptualmente:

Schedule
   ↓
Due
   ↓
Claim
   ↓
Trigger

El claim debe ser atómico.

33. Duplicate Trigger Prevention

Debe evitarse:

Scheduler A ──┐
              ├──> Same Job
Scheduler B ──┘

Para ello pueden utilizarse:

Locks
Leases
Compare-and-Swap
Unique Constraints
Idempotency Keys
34. Schedule Idempotency

Cada ocurrencia puede generar una clave:

schedule_id
+
scheduled_at

por ejemplo:

schedule-123:2027-01-15T08:00:00Z

Esto permite detectar duplicados.

35. Exactly Once

No debe asumirse que el scheduler proporciona exactamente una entrega física.

La arquitectura debe buscar:

At-Least-Once Triggering
+
Idempotent Job Creation

cuando la infraestructura lo requiera.

36. At-Least-Once Scheduling

Puede ocurrir:

Trigger
 ↓
Network Failure
 ↓
Scheduler Unsure
 ↓
Trigger Again

Por ello:

La creación del Job debe ser idempotente.

37. Schedule Concurrency

Un Schedule recurrente puede tener:

Previous Run
      │
      └── Still Running

cuando llega la siguiente ocurrencia.

Debe existir una política.

38. Concurrency Policies

Opciones conceptuales:

ALLOW
SKIP
QUEUE
REPLACE
COALESCE
39. Allow Policy

Permite ejecuciones simultáneas:

Run 1 ───────────────
Run 2       ───────────────
Run 3             ───────────────

Adecuado cuando las ejecuciones son independientes.

40. Skip Policy

Si una ejecución está activa:

Run 1 ─────────────
Run 2 → SKIPPED

Útil para tareas de mantenimiento donde ejecutar dos veces no aporta valor.

41. Queue Policy

La siguiente ejecución espera:

Run 1 ─────────
Run 2         ─────────
Run 3                   ─────────
42. Replace Policy

Una ejecución nueva puede reemplazar una anterior cuando sea seguro hacerlo.

Run 1 ──────X
Run 2       ─────────

No debe utilizarse para Jobs con side effects no reversibles sin una estrategia explícita.

43. Coalesce Policy

Múltiples ocurrencias pueden consolidarse:

Run 1 ─┐
Run 2 ─┼──> One Job
Run 3 ─┘

Útil para procesamiento periódico de cambios acumulados.

44. Misfire

Un Schedule puede no ejecutarse a tiempo:

Expected:
08:00

Scheduler unavailable

Actual:
08:17

Esto se denomina conceptualmente:

Misfire
45. Misfire Policies

Debe soportarse una política explícita:

FIRE_NOW
SKIP
FIRE_ONCE
CATCH_UP
FAIL
46. Fire Now

Si el schedule perdió su hora:

08:00 missed
08:17 scheduler recovers
       ↓
Execute immediately
47. Skip

La ocurrencia perdida no se ejecuta:

08:00 missed
       ↓
Skip
       ↓
Next occurrence
48. Fire Once

Si se perdieron múltiples ocurrencias:

08:00
09:00
10:00

puede ejecutarse una sola vez al recuperarse.

49. Catch Up

Puede ejecutarse cada ocurrencia perdida:

08:00 → execute
09:00 → execute
10:00 → execute

Debe utilizarse con cuidado para no producir una tormenta de Jobs.

50. Misfire Window

Puede existir un límite:

misfire_grace_period

Ejemplo:

Missed < 15 min
→ Fire

Missed > 15 min
→ Skip
51. Schedule Start

Puede definirse:

start_at

antes del cual no se debe generar ninguna ejecución.

52. Schedule End

Puede definirse:

end_at

después del cual el Schedule deja de generar nuevas ejecuciones.

53. Maximum Executions

Un Schedule puede limitarse:

max_executions = 10

Después:

ACTIVE
  ↓
COMPLETED / RETIRED
54. Schedule Duration

Puede definirse:

valid_from
valid_until

para limitar temporalmente la existencia del Schedule.

55. Time Window

Un Schedule puede ejecutarse únicamente dentro de una ventana:

09:00 — 17:00

Ejemplo:

Business Hours
56. Blackout Window

Puede definirse una ventana donde no se deben crear Jobs:

00:00 — 02:00

por ejemplo durante mantenimiento.

57. Holiday Calendar

Un Schedule puede respetar:

Holiday Calendar

Ejemplo:

Every business day

requiere conocer qué días son laborales.

58. Tenant Calendar

Un tenant puede tener:

Timezone
Working Days
Holidays
Business Hours

El Scheduler debe aplicar estas reglas cuando el Schedule sea tenant-aware.

59. User Time Zone

Los schedules creados por usuarios pueden guardar:

user_timezone

en lugar de depender de la timezone del servidor.

60. Schedule Ownership

Cada Schedule debe tener:

owner
tenant_id
scope

cuando corresponda.

61. Schedule Authorization

Las operaciones:

Create
Update
Pause
Resume
Disable
Delete
Trigger Now
Inspect

deben estar protegidas por autorización.

62. Trigger Now

Puede existir una operación administrativa:

Trigger Now

que genere una ejecución inmediatamente.

Debe diferenciarse de:

Scheduled Execution

para auditoría.

63. Manual Trigger

Un trigger manual debe conservar:

trigger_type = MANUAL
triggered_by
triggered_at
64. Schedule Audit

Debe registrarse:

Created
Updated
Paused
Resumed
Triggered
Skipped
Misfired
Disabled
Retired
65. Schedule History

Debe poder responderse:

When should it have run?
When did it actually run?
Was it late?
Was it skipped?
Which Job was created?
Why?
66. Schedule-to-Job Correlation

Debe existir una relación:

Schedule
   │
   └── schedule_execution_id
              │
              ▼
            Job

Esto permite rastrear:

Schedule
 ↓
Occurrence
 ↓
Job
 ↓
Worker
 ↓
Result
67. Schedule Execution

Cada ocurrencia puede tener:

execution_id
scheduled_at
triggered_at
job_id
status
68. Schedule Status

Una ejecución puede estar:

DUE
CLAIMED
TRIGGERED
SKIPPED
MISFIRED
FAILED
COMPLETED
69. Scheduler Failure

Si el Scheduler falla:

Scheduler
   ↓
Crash

al reiniciarse:

Restart
   ↓
Load Schedule Store
   ↓
Detect Due Schedules
   ↓
Apply Misfire Policy
   ↓
Resume
70. Clock Failure

El sistema debe evitar depender de relojes inconsistentes entre nodos.

Debe utilizar:

Consistent Time Source

y timestamps normalizados.

71. Clock Skew

En un cluster:

Node A → 10:00:00
Node B → 10:00:03

puede generar condiciones de carrera.

El diseño debe minimizar el impacto del clock skew.

72. Scheduler Heartbeat

Los nodos pueden registrar:

scheduler_id
last_heartbeat
status

para detectar nodos inactivos.

73. Scheduler Leader

Puede utilizarse un modelo:

Leader
  │
  ├── Schedule Evaluation
  └── Trigger Creation

con otros nodos como standby.

Esto simplifica ciertas condiciones de concurrencia.

74. Leader Election

Si existe leader election:

Leader A
   ↓
Failure
   ↓
Leader B

la transición debe minimizar:

Duplicate Triggers
Missed Executions
75. Sharded Scheduling

Para grandes volúmenes:

Schedules
   │
   ├── Shard A
   ├── Shard B
   ├── Shard C
   └── Shard D

cada scheduler puede procesar una partición.

76. Scheduler Scaling

La capacidad debe escalar según:

Number of Schedules
Due Schedule Rate
Trigger Rate
Schedule Evaluation Cost
Tenant Count
77. Near-Term Scheduling

No es necesario mantener todos los timers en memoria.

Puede utilizarse una estrategia:

Persistent Store
      ↓
Next Due Window
      ↓
In-Memory Timer
      ↓
Trigger
78. Schedule Indexing

Para grandes volúmenes debe ser eficiente consultar:

next_run_at <= now

El Schedule Store debe permitir búsquedas eficientes por tiempo.

79. Schedule Partitioning

Puede particionarse por:

Tenant
Time Range
Schedule Type
Shard
Region

según escala.

80. Regional Scheduling

En arquitecturas multi-región puede existir:

Region A Scheduler
Region B Scheduler
Region C Scheduler

Debe definirse quién tiene autoridad sobre cada Schedule.

81. Regional Ownership

Un Schedule puede tener:

home_region

para evitar ejecuciones duplicadas entre regiones.

82. Failover

Si una región falla:

Region A
   ↓
Failure
   ↓
Region B
   ↓
Take Ownership

Debe aplicarse una estrategia explícita de recuperación.

83. Active-Active

En Active-Active:

Region A ─┐
          ├── Shared Schedule State
Region B ─┘

se requiere coordinación fuerte para evitar doble ejecución.

84. Active-Passive

En Active-Passive:

Primary Scheduler
      │
      ▼
Standby Scheduler

la implementación puede ser más simple, aunque con failover potencialmente más lento.

85. Multi-Tenant Scheduling

Los Schedules pueden pertenecer a:

Platform
Tenant
Organization
User
System

Cada scope debe tener reglas claras.

86. Tenant Quotas

Puede limitarse:

Schedules per Tenant
Executions per Minute
Concurrent Executions
87. Noisy Neighbor Protection

Un tenant que crea:

1,000,000 schedules

no debe degradar el Scheduler completo.

Deben existir:

Quotas
Rate Limits
Isolation
Fair Scheduling
88. Schedule Validation

Antes de activar un Schedule:

Expression Valid?
Timezone Valid?
Target Valid?
Permissions Valid?
Tenant Valid?
Start/End Valid?
89. Invalid Schedule

Un Schedule inválido debe quedar:

INVALID

o rechazarse antes de persistirse como activo.

No debe producir ejecuciones impredecibles.

90. Schedule Update

Modificar un Schedule activo requiere una semántica clara.

Ejemplo:

Every hour

cambiado a:

Every 2 hours

Debe definirse qué ocurre con la próxima ejecución ya calculada.

91. Schedule Versioning

Los cambios importantes pueden generar:

Schedule v1
Schedule v2

Esto resulta especialmente útil para auditoría.

92. Schedule Deletion

Eliminar un Schedule no debe necesariamente eliminar Jobs ya generados.

Schedule
   ↓
Delete
   ↓
Existing Job
   ↓
Can still execute

La semántica debe ser explícita.

93. Pause Semantics

Pausar significa:

Do not generate new executions

No necesariamente:

Cancel existing Jobs
94. Resume Semantics

Al reanudar:

PAUSED
   ↓
RESUME

debe aplicarse la política de misfire correspondiente.

95. Schedule Dependencies

Un Schedule puede depender de:

System State
Tenant State
External Calendar
Feature Flag

pero las dependencias complejas deberían resolverse en Jobs o Workflows.

96. Feature Flags

Puede utilizarse:

Schedule
   ↓
Feature Enabled?

antes de crear el Job.

Debe quedar auditado si una ejecución fue suprimida por una política.

97. Rate Limiting

Puede limitarse la creación de Jobs:

100 executions/minute

para evitar overload.

98. Burst Handling

Si llegan muchas ejecuciones simultáneamente:

00:00
████████████████████ Jobs

el Scheduler debe:

Queue
Throttle
Batch
Prioritize

en lugar de intentar ejecutar todo inmediatamente.

99. Scheduler Backpressure

E15 gestiona backpressure de Jobs.

E16 debe evitar crear una cantidad ilimitada de Jobs si el downstream no tiene capacidad.

Scheduler
   ↓
Capacity Check
   ↓
Job Creation

cuando sea apropiado.

100. Schedule Observability

Métricas:

schedules_total
schedules_active
schedules_paused
schedule_triggers_total
schedule_skips_total
schedule_misfires_total
schedule_trigger_latency
schedule_evaluation_latency
101. Schedule Lag

Debe medirse:

triggered_at - scheduled_at

Esto proporciona:

Schedule Lag
102. Scheduler Health

Indicadores:

Due Schedule Backlog
Trigger Rate
Evaluation Latency
Misfire Rate
Duplicate Rate
Scheduler Heartbeat
103. Alerts

Alertas importantes:

Scheduler Down
Schedule Backlog High
Trigger Lag High
Misfire Rate High
Duplicate Trigger Detected
Schedule Store Failure
Leader Election Failure
Tenant Quota Exhaustion
104. Logging

Logs estructurados:

schedule_id
execution_id
tenant_id
scheduled_at
triggered_at
job_id
scheduler_id
status
105. Distributed Tracing

Debe poder rastrearse:

Schedule
   ↓
Schedule Execution
   ↓
Job
   ↓
Worker
   ↓
Application Service

mediante:

trace_id
correlation_id
106. Scheduler Security

El Scheduler debe proteger:

Schedule Definitions
Tenant Configuration
Execution Metadata
Administrative Controls
107. Scheduler Secrets

El Scheduler no debe almacenar:

API Keys
Passwords
Access Tokens

Los Jobs o servicios correspondientes deben resolver referencias seguras cuando sea necesario.

108. Scheduler Administration

Operaciones administrativas:

Pause Schedule
Resume Schedule
Trigger Now
Disable
Recalculate Next Run
Inspect Execution
Retry Failed Trigger

deben estar autorizadas y auditadas.

109. Schedule Recovery

Una estrategia de recuperación:

Scheduler Failure
       ↓
Restart
       ↓
Load Persistent State
       ↓
Find Due Schedules
       ↓
Evaluate Misfire
       ↓
Claim
       ↓
Create Job
110. Recovery Idempotency

Si el scheduler no sabe si creó el Job:

Schedule
 ↓
Create Job
 ↓
Network failure

al recuperar debe poder comprobar:

Was execution already materialized?

antes de crear otro.

111. Schedule Execution Key

Una clave canónica:

(schedule_id, scheduled_at)

puede actuar como identidad única de la ocurrencia.

112. Job Creation Contract

E16 debe entregar a E15 como mínimo:

job_type
job_version
payload
tenant_id
correlation_id
causation_id
scheduled_at
schedule_execution_id
idempotency_key
113. Schedule → Job Contract
┌─────────────────────┐
│ Schedule             │
│                      │
│ schedule_id          │
│ execution_id         │
│ scheduled_at         │
│ timezone             │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│ Job                  │
│                      │
│ job_id               │
│ job_type             │
│ tenant_id            │
│ idempotency_key      │
│ scheduled_at         │
└─────────────────────┘
114. Workflow Timer Integration

E14 puede utilizar E16 conceptualmente:

Workflow
   │
   ▼
Timer
   │
   ▼
Scheduler
   │
   ▼
Resume Workflow

Pero el Scheduler no debe convertirse en propietario del Workflow State.

115. Reminder Example
User
 ↓
Create Reminder
 ↓
Schedule
 ↓
09:00
 ↓
Create Notification Job
 ↓
E15
 ↓
Notification Service
116. Subscription Renewal Example
Subscription
      ↓
Renewal Schedule
      ↓
Renewal Date
      ↓
Create Renewal Job
      ↓
E15
      ↓
Payment Service
117. Daily Analytics Example
Schedule
Every day 02:00
       ↓
Scheduler
       ↓
Analytics Job
       ↓
E15
       ↓
Analytics Worker
118. Business-Day Example
Schedule
Every Business Day
       ↓
Calendar
       ↓
Is Business Day?
       ↓
YES
       ↓
Create Job
119. Deadline Example
Workflow
   ↓
Deadline = T+24h
   ↓
Scheduler
   ↓
Deadline Event / Job
   ↓
Workflow
120. Canonical Architecture
                         EVOXA
                           │
          ┌────────────────┼────────────────┐
          ▼                ▼                ▼
       Workflow           API             Event
         E14               E03              E13
          │                │                │
          └────────────────┼────────────────┘
                           ▼
                    Schedule Creation
                           │
                           ▼
                    Schedule Registry
                           │
                           ▼
                    Persistent Store
                           │
                           ▼
                    Scheduler Engine
                           │
             ┌─────────────┼─────────────┐
             ▼             ▼             ▼
          Timer A       Timer B       Timer C
             │             │             │
             └─────────────┼─────────────┘
                           ▼
                    Schedule Execution
                           │
                           ▼
                      Job Creation
                           │
                           ▼
                         E15
                           │
                           ▼
                       Job Queue
                           │
                           ▼
                        Worker
121. Architectural Rules
Rule 1

Scheduling determina cuándo ejecutar; Job Processing determina cómo ejecutar.

Rule 2

Scheduler nunca debe contener lógica empresarial.

Rule 3

Todo Schedule debe tener una semántica temporal explícita.

Rule 4

Las ejecuciones deben ser idempotentes frente a redelivery.

Rule 5

Los timestamps operacionales deben manejarse consistentemente en UTC.

Rule 6

Las definiciones basadas en horario local deben conservar timezone explícita.

Rule 7

DST debe tener una política definida.

Rule 8

Un Scheduler distribuido debe evitar doble triggering.

Rule 9

Los misfires deben tener una política explícita.

Rule 10

Pausar un Schedule no implica automáticamente cancelar Jobs existentes.

Rule 11

El Schedule Store debe ser durable.

Rule 12

Las ejecuciones deben correlacionarse con los Jobs generados.

Rule 13

Los límites por tenant deben evitar noisy neighbors.

Rule 14

Scheduling complejo de procesos empresariales pertenece a E14.

Rule 15

La ejecución física del trabajo pertenece a E15.

122. Definition of Done

E16 queda definido cuando EVOXA dispone de:

✓ Schedule Definition
✓ Schedule Instance
✓ Schedule Execution
✓ One-Time Scheduling
✓ Delayed Scheduling
✓ Interval Scheduling
✓ Recurring Scheduling
✓ Cron Scheduling
✓ Calendar Scheduling
✓ Event-Relative Scheduling
✓ Deadline Scheduling
✓ Time Windows
✓ Time Zones
✓ UTC Handling
✓ DST Handling
✓ Ambiguous Time Policy
✓ Nonexistent Time Policy
✓ Schedule Lifecycle
✓ Schedule Creation
✓ Schedule Triggering
✓ Next Run
✓ Previous Run
✓ Persistent Schedule Store
✓ Durable Scheduling
✓ Distributed Scheduler
✓ Schedule Claiming
✓ Duplicate Prevention
✓ Idempotency
✓ At-Least-Once Semantics
✓ Concurrency Policies
✓ Allow Policy
✓ Skip Policy
✓ Queue Policy
✓ Replace Policy
✓ Coalesce Policy
✓ Misfire Detection
✓ Misfire Policies
✓ Misfire Window
✓ Start / End Constraints
✓ Maximum Executions
✓ Holiday Calendars
✓ Tenant Calendars
✓ User Time Zones
✓ Schedule Ownership
✓ Schedule Authorization
✓ Manual Triggering
✓ Schedule Audit
✓ Schedule History
✓ Schedule-to-Job Correlation
✓ Scheduler Recovery
✓ Clock Management
✓ Clock Skew Handling
✓ Scheduler Heartbeat
✓ Leader Election Compatibility
✓ Sharded Scheduling
✓ Scheduler Scaling
✓ Schedule Indexing
✓ Regional Scheduling
✓ Regional Ownership
✓ Failover
✓ Active-Active Compatibility
✓ Active-Passive Compatibility
✓ Multi-Tenant Scheduling
✓ Tenant Quotas
✓ Noisy Neighbor Protection
✓ Schedule Validation
✓ Schedule Update Semantics
✓ Schedule Versioning
✓ Pause / Resume
✓ Rate Limiting
✓ Burst Handling
✓ Scheduler Backpressure
✓ Observability
✓ Schedule Lag
✓ Health Monitoring
✓ Alerts
✓ Structured Logging
✓ Distributed Tracing
✓ Security
✓ Administrative Controls
✓ Recovery Idempotency
✓ Schedule Execution Keys
✓ Schedule → Job Contract
✓ E14 Integration
✓ E15 Integration
✓ E13 Integration
123. Engineering Specification Progress

La secuencia queda:

E01 — EVOXA Backend Architecture
E02 — EVOXA Database Architecture
E03 — EVOXA API Architecture
E04 — EVOXA Authentication Architecture
E05 — Authorization Architecture
E06 — Policy Architecture
E07 — EVOXA Service Architecture
E08 — Domain Services Architecture
E09 — Application Services Architecture
E10 — Repository Architecture
E11 — Integration Architecture
E12 — Messaging Architecture
E13 — Event Processing Architecture
E14 — Workflow & Orchestration Architecture
E15 — Job & Task Processing Architecture
E16 — Scheduling Architecture

La separación temporal y operacional de EVOXA queda:

                         EVOXA
                           │
          ┌────────────────┼─────────────────┐
          ▼                ▼                 ▼
       Event            Workflow           Schedule
        E13               E14                E16
          │                │                  │
          │                │                  │
          └────────────────┼──────────────────┘
                           ▼
                      Job Creation
                           │
                           ▼
                    E15 Job Processing
                           │
                           ▼
                        Queue
                           │
                           ▼
                        Worker
                           │
                           ▼
                  Application Service
                           │
              ┌────────────┴────────────┐
              ▼                         ▼
           Domain                  Integration
              │                         │
              ▼                         ▼
         Repository               External World
              │
              ▼
           Database

La regla arquitectónica central queda:

E16 decide cuándo ocurre una ejecución; E15 ejecuta el trabajo; E14 coordina procesos; E13 procesa eventos.

Siguiente capítulo: E17 — EVOXA Caching Architecture.
