E38 — EVOXA Resilience Architecture
Engineering Specification

Pertenece a: Engineering Specification
Capítulo: E38 — Resilience Architecture
Anterior: E37 — Runtime Capacity Architecture
Siguiente: E39 — EVOXA Fault Tolerance Architecture

1. Propósito

E38 define cómo EVOXA mantiene la continuidad operativa, aislamiento de fallos, degradación controlada y recuperación cuando componentes, dependencias, recursos o condiciones operativas fallan.

La idea central es:

Resilience is the ability of EVOXA to continue providing acceptable behavior under failure, overload, dependency degradation, partial outage, and recovery conditions.

E37 responde:

¿Cuánto trabajo puede soportar el Runtime?

E38 responde:

¿Qué ocurre cuando esa capacidad, un componente o una dependencia falla?

2. Resilience Boundary

E38 controla:

failure
fault isolation
degradation
recovery
redundancy
failover
retry
timeout
circuit breaking
bulkheads
health
recovery coordination

No debe confundirse con:

E37 → capacity
E35 → execution
E20 → runtime policy
E16 → scheduling
3. Resilience Model
Normal Operation
       ↓
Stress
       ↓
Failure Detection
       ↓
Containment
       ↓
Degradation / Failover
       ↓
Recovery
       ↓
Reintegration
       ↓
Normal Operation
4. Resilience Objectives

EVOXA debe intentar:

1. Prevent failure propagation
2. Detect failures quickly
3. Isolate affected components
4. Preserve critical functionality
5. Degrade non-critical functionality
6. Recover automatically where safe
7. Avoid recovery storms
8. Preserve data correctness
9. Maintain tenant isolation
10. Make resilience behavior observable
5. Failure Taxonomy

Los fallos se clasifican como:

Component Failure
Dependency Failure
Network Failure
Resource Failure
Capacity Failure
Process Failure
Data Failure
Configuration Failure
Deployment Failure
Infrastructure Failure
Regional Failure
Operator-Induced Failure
6. Component Failure

Ejemplos:

API service unavailable
worker crashes
scheduler unavailable
event processor stops
repository unavailable
agent runtime crashes

El fallo no debe propagarse automáticamente al resto del sistema.

7. Dependency Failure
EVOXA
  ↓
External Dependency
  ↓
Unavailable

Ejemplos:

database
identity provider
payment provider
AI provider
message broker
external API
storage

Debe existir una estrategia específica.

8. Network Failure

Incluye:

timeout
packet loss
connection reset
DNS failure
partition
high latency
intermittent connectivity

Una red degradada debe tratarse como una condición de resiliencia, no únicamente como un error técnico.

9. Resource Failure

Puede incluir:

CPU exhaustion
memory exhaustion
disk exhaustion
connection pool exhaustion
worker exhaustion
GPU exhaustion

Aquí E37 proporciona las señales de capacity.

10. Capacity Failure
Demand
   ↓
Capacity
   ↓
Exhausted

La resiliencia debe activar:

backpressure
queueing
throttling
load shedding
degradation

según política.

11. Data Failure

Incluye:

corrupted state
inconsistent state
missing data
stale data
partial write
failed transaction

La resiliencia de datos debe priorizar:

correctness > availability

cuando la operación no pueda garantizar consistencia.

12. Configuration Failure

Ejemplos:

invalid configuration
missing secret
incorrect endpoint
invalid policy
incompatible feature flag

La configuración inválida no debe provocar una cascada de fallos.

13. Deployment Failure

Incluye:

bad release
incompatible version
failed migration
failed startup
partial rollout

Debe soportarse:

rollback
roll-forward
traffic isolation
version coexistence
14. Failure Domain

EVOXA debe reconocer diferentes failure domains:

Process
Worker
Service
Host
Availability Zone
Region
Provider
Tenant
Domain
Dependency
15. Failure Domain Isolation

La regla:

A failure should remain within the smallest practical failure domain.

Ejemplo:

Worker A crashes
      ↓
Worker A affected
      ↓
Worker Pool continues

No:

Worker A crashes
      ↓
Entire Runtime crashes
16. Fault Containment
Fault
 ↓
Detect
 ↓
Contain
 ↓
Protect
 ↓
Recover

El containment layer debe ejecutarse antes de permitir propagación.

17. Fault Isolation

Mecanismos:

bulkheads
process isolation
resource quotas
tenant isolation
connection pools
worker pools
queue isolation
dependency isolation
18. Bulkhead Pattern

Los workloads se separan:

Runtime
├── Critical Pool
├── Interactive Pool
├── Background Pool
└── AI Pool

Si:

Background Pool

falla o se satura:

Critical Pool

debe continuar operativo.

19. Tenant Isolation

Un tenant no debe consumir accidentalmente toda la capacidad de otro.

Tenant A overload
       ↓
Tenant A degraded
       ↓
Tenant B continues

Esto conecta directamente con E13 Multi-Tenant Architecture y E37 Capacity.

20. Dependency Isolation

Cada dependencia importante debe disponer de:

timeout
retry policy
circuit breaker
connection limit
bulkhead
fallback
health state
21. Timeout Architecture

Toda operación externa debe tener un límite temporal.

Request
  ↓
Timeout Boundary
  ↓
Dependency

Si excede:

TIMEOUT

la operación debe finalizar o cambiar de estrategia.

22. Timeout Types
Connection Timeout
Read Timeout
Write Timeout
Execution Timeout
Queue Timeout
Workflow Timeout
Dependency Timeout
Global Deadline
23. Deadline Propagation

Una operación puede recibir:

deadline = T

Las llamadas internas deben respetarla.

Request
deadline = 10s
    ↓
Service A
remaining = 8s
    ↓
Service B
remaining = 5s
    ↓
Dependency
remaining = 2s

No debe reiniciarse el timeout arbitrariamente en cada hop.

24. Retry Architecture

Los retries deben utilizarse únicamente cuando el error sea potencialmente recuperable.

Transient Failure
      ↓
Retry

No:

Permanent Failure
      ↓
Retry forever
25. Retryable Errors

Ejemplos:

temporary network failure
connection reset
temporary dependency unavailable
rate limit
transient infrastructure failure
26. Non-Retryable Errors

Ejemplos:

validation failure
authorization failure
invalid request
business rule violation
schema incompatibility
permanent configuration failure
27. Retry Limit

Toda retry policy debe definir:

maxAttempts
maxElapsedTime
backoff
jitter
retryableErrors
28. Exponential Backoff

Conceptualmente:

Attempt 1 → immediate
Attempt 2 → short delay
Attempt 3 → longer delay
Attempt 4 → longer delay

Evita saturar una dependencia que ya está fallando.

29. Jitter

Los retries deben incorporar jitter para evitar:

Retry Storm

Cuando muchos workers fallan simultáneamente:

1000 workers
   ↓
retry simultaneously
   ↓
dependency overload

Con jitter:

retry
 ├── 1.2s
 ├── 1.7s
 ├── 2.4s
 ├── 3.1s
 └── ...
30. Retry Budget

EVOXA debe poder limitar retries globalmente.

Normal requests = 1000
Retries allowed = 100

No permitir:

1000 requests
×
10 retries
=
10000 requests

sin control.

31. Retry Amplification

Debe evitarse:

Service A retry
   ↓
Service B retry
   ↓
Service C retry

porque una sola operación puede multiplicarse exponencialmente.

32. Retry Ownership

Debe existir un responsable principal del retry.

Preferentemente:

one logical retry layer

por operación.

33. Circuit Breaker

Cuando una dependencia falla repetidamente:

Healthy
   ↓
Failures
   ↓
Threshold
   ↓
OPEN

El Circuit Breaker evita llamadas adicionales innecesarias.

34. Circuit States
CLOSED
OPEN
HALF_OPEN
35. CLOSED
Request
  ↓
Dependency

Las llamadas están permitidas.

36. OPEN
Request
  ↓
Circuit Breaker
  ↓
Fast Failure / Fallback

La dependencia no recibe tráfico.

37. HALF_OPEN

Después de cooldown:

OPEN
 ↓
cooldown
 ↓
HALF_OPEN
 ↓
probe request

Si funciona:

CLOSED

Si falla:

OPEN
38. Circuit Breaker Threshold

Debe definirse mediante política:

failure count
failure ratio
latency threshold
time window
39. Circuit Breaker Scope

Puede existir por:

dependency
tenant
operation
region
provider
resource

Evitar un breaker global cuando un fallo local es suficiente.

40. Fallback

Cuando una dependencia no está disponible:

Primary
  ↓
Failure
  ↓
Fallback

Ejemplos:

cache
secondary provider
stale read model
degraded response
queued operation
41. Fallback Safety

Fallback no debe ocultar errores críticos.

Ejemplo:

Read unavailable
→ stale data acceptable

puede ser válido.

Pero:

Payment confirmation unavailable
→ assume success

puede ser peligroso.

42. Graceful Degradation

EVOXA debe poder reducir funcionalidad:

Full Mode
   ↓
Reduced Mode
   ↓
Critical Mode
43. Degradation Classes
FULL
REDUCED
MINIMAL
CRITICAL_ONLY
READ_ONLY
QUEUE_ONLY
OFFLINE
44. Feature Degradation

En overload:

Optional AI enrichment
        ↓
disabled

mientras:

Core transaction
        ↓
continues
45. Dependency Degradation
Primary Provider
      ↓
Unavailable
      ↓
Secondary Provider

Debe estar gobernado por policy.

46. Read Degradation

Cuando write path está afectado:

WRITE unavailable
READ available

EVOXA puede operar temporalmente como:

READ_ONLY

si es seguro.

47. Write Protection

Ante incertidumbre de consistencia:

Unknown write state
      ↓
Protect further writes

Esto evita corrupción.

48. Idempotency

Toda operación recuperable debe declarar si es:

idempotent
non-idempotent
conditionally idempotent
49. Idempotency Keys

Para operaciones críticas:

idempotencyKey

permite:

retry
without duplicate side effect
50. Duplicate Execution Protection

Especialmente importante en:

payments
notifications
commands
external side effects
workflow actions
agent actions
51. Exactly-Once Illusion

EVOXA no debe asumir que:

network retry
+
distributed system
=
exactly once

Debe diseñarse alrededor de:

at-least-once delivery
+
idempotent processing
+
deduplication

cuando aplique.

52. Failure Detection

Detection mechanisms:

health checks
heartbeats
timeouts
error rates
latency
queue growth
missing events
worker state
dependency probes
53. Health Model
HEALTHY
DEGRADED
UNHEALTHY
UNKNOWN
DRAINING
RECOVERING
54. Liveness

Liveness responde:

¿El componente está vivo?

Ejemplo:

process responds
55. Readiness

Readiness responde:

¿Puede recibir trabajo?

Un proceso puede estar:

alive = true
ready = false

durante:

startup
recovery
draining
dependency initialization
56. Startup Resilience

Startup debe soportar:

dependency unavailable
configuration delay
partial initialization
temporary network failure

sin crear una cascada innecesaria.

57. Recovery State
FAILED
 ↓
RECOVERING
 ↓
HEALTH_CHECK
 ↓
READY

No debe pasar directamente de failure a full traffic sin validación.

58. Traffic Reintroduction

Después de recuperación:

0%
 ↓
5%
 ↓
25%
 ↓
50%
 ↓
100%

según política.

Esto reduce el riesgo de recovery overload.

59. Recovery Storm

Problema:

dependency recovers
 ↓
all clients retry
 ↓
traffic spike
 ↓
dependency fails again

Mitigación:

jitter
rate limiting
gradual recovery
connection ramp-up
60. Failover

Failover:

Primary
  ↓
Failure
  ↓
Secondary

Puede ser:

active-passive
active-active
warm standby
cold standby
61. Active-Passive
Primary → traffic
Secondary → standby

Tras fallo:

Secondary → traffic
62. Active-Active
Region A → traffic
Region B → traffic

El tráfico puede redistribuirse cuando una región falla.

63. Failover Criteria

Debe definirse:

failure threshold
health state
timeout
operator override
data safety condition

No debe depender de una única señal.

64. Failback

Después de recuperar primary:

Secondary
   ↓
Primary recovery
   ↓
Data validation
   ↓
Traffic migration

No realizar failback inmediatamente sin validación.

65. Data Resilience

Debe existir:

backup
replication
consistency
recovery point
recovery time
validation
66. RPO

Recovery Point Objective:

How much data may be lost?

Ejemplo:

RPO = 5 minutes
67. RTO

Recovery Time Objective:

How long may recovery take?

Ejemplo:

RTO = 10 minutes
68. Resilience Tiers

Diferentes workloads pueden tener distintos objetivos:

Tier 0 → critical
Tier 1 → high
Tier 2 → standard
Tier 3 → best effort
69. Resilience Policy

Una política puede declarar:

failureStrategy
retryPolicy
timeoutPolicy
fallbackPolicy
degradationMode
recoveryPolicy
70. Resilience Policy Example
failureStrategy:
    retry = 3
    backoff = exponential
    jitter = enabled

timeout:
    5 seconds

circuit:
    failureRatio = 50%

fallback:
    cache

degradation:
    REDUCED
71. Resilience Controller

Responsabilidades:

detect
classify
contain
degrade
recover
restore

No debe ejecutar directamente el workload.

72. Resilience Decision

Toda decisión importante debe tener:

condition
signal
policy
decision
scope
duration
reason
73. Resilience Event Model

Eventos:

FailureDetected
DependencyDegraded
CircuitOpened
CircuitClosed
RetryScheduled
RetryExhausted
FallbackActivated
DegradationActivated
DegradationRecovered
FailoverStarted
FailoverCompleted
RecoveryStarted
RecoveryCompleted
HealthChanged
74. Failure Correlation

Debe poder correlacionarse:

Root Failure
   ↓
Dependent Failures
   ↓
User Impact

Ejemplo:

Database failure
 ↓
Repository failures
 ↓
Domain service latency
 ↓
Workflow delays
 ↓
API errors
75. Root Failure vs Symptom

Observability debe diferenciar:

ROOT_CAUSE

de:

SECONDARY_FAILURE

No alertar 500 veces como si fueran 500 fallos independientes.

76. Failure Storm Control

Durante incidentes:

failure rate ↑

EVOXA debe limitar:

logs
events
retries
alerts
notifications

para evitar secondary overload.

77. Resilience Observability

Métricas mínimas:

failure_rate
timeout_rate
retry_rate
retry_exhaustion_rate
circuit_open_rate
fallback_rate
degradation_rate
recovery_time
failover_count
health_state
dependency_latency
dependency_error_rate
78. Resilience SLO

Ejemplos:

99.9% critical operations remain available
99% dependency failures contained within 5 seconds
Recovery within defined RTO
No uncontrolled retry amplification
79. Resilience Testing

La arquitectura debe ser verificable mediante:

failure injection
dependency outage
worker termination
network latency
packet loss
capacity exhaustion
database failover
provider failure
region failure
80. Chaos Testing

Chaos Engineering puede validar:

fault containment
recovery
failover
degradation
capacity behavior
81. Fault Injection

Ejemplos:

kill worker
block dependency
delay database
drop messages
inject timeout
exhaust connections
82. Recovery Testing

No basta probar:

failure

Debe probarse:

failure
→ detection
→ containment
→ recovery
→ reintegration
83. Resilience Test Matrix
Failure
   │
   ├── Detect?
   ├── Contain?
   ├── Degrade?
   ├── Recover?
   ├── Restore?
   └── Observe?

Cada componente crítico debe tener respuestas verificadas.

84. Resilience and E37 Capacity

La relación fundamental:

E37 Capacity
       │
       ▼
Capacity Pressure
       │
       ▼
E38 Resilience
       │
 ┌─────┼─────┐
 ▼     ▼     ▼
Queue Throttle Shed
       │
       ▼
Protected Runtime
85. Resilience and E35 Execution
Execution
   ↓
Timeout
   ↓
Failure
   ↓
Retry / Fallback / Abort
   ↓
Recovery

Execution debe ser consciente de los límites de resiliencia.

86. Resilience and E14 Workflow

Workflow failures pueden requerir:

retry activity
compensation
pause
resume
checkpoint
manual intervention
87. Resilience and E15 Jobs

Jobs pueden utilizar:

retry
dead-letter
backoff
pause
resume
requeue
88. Resilience and E16 Scheduling

Scheduler debe evitar programar repetidamente workloads cuando:

dependency unavailable
capacity exhausted
system degraded

Puede utilizar:

deferred scheduling
89. Resilience and E17 Cache

Cache puede funcionar como:

fallback
read degradation
dependency shielding

Pero nunca debe ocultar permanentemente inconsistencias críticas.

90. Resilience and E18 Configuration

Configuration debe soportar:

safe defaults
validation
versioning
rollback

Una configuración defectuosa no debe destruir resilience controls.

91. Resilience and E19 Feature Flags

Feature flags permiten:

disable failing feature
reduce functionality
activate fallback

durante incidentes.

92. Resilience and E20 Runtime Policy

E20 determina:

what is allowed

E38 determina:

how the Runtime behaves under failure
93. Resilience and E21 Rules

Rules pueden definir:

retry eligibility
fallback eligibility
degradation conditions
94. Resilience and E30 Reporting

Reporting debe distinguir:

successful
degraded
failed
partial
recovered
95. Resilience and E32 Intelligence

Intelligence puede ayudar a:

detect anomalies
predict failures
identify bottlenecks
recommend mitigation

Pero los mecanismos críticos de protección no deben depender exclusivamente de AI.

96. Resilience and E33 Decision

Decision puede seleccionar:

retry
fallback
failover
degrade
abort

cuando la política lo permita.

97. Resilience and E34 Action

Actions deben declarar:

retryable
idempotent
timeout
fallback
compensation

cuando corresponda.

98. Resilience and E36 Resources

Resource failure debe traducirse en:

Resource unavailable
       ↓
Capacity reduced
       ↓
Resilience strategy
99. Resilience Control Loop
             ┌───────────────┐
             │    Runtime    │
             └───────┬───────┘
                     ▼
             ┌───────────────┐
             │    Detect     │
             └───────┬───────┘
                     ▼
             ┌───────────────┐
             │    Classify   │
             └───────┬───────┘
                     ▼
             ┌───────────────┐
             │    Contain    │
             └───────┬───────┘
                     ▼
             ┌───────────────┐
             │    Protect    │
             └───────┬───────┘
                     ▼
             ┌───────────────┐
             │    Recover    │
             └───────┬───────┘
                     ▼
             ┌───────────────┐
             │ Reintegrate   │
             └───────┬───────┘
                     │
                     └──────────────► Observe
100. Reference Resilience Architecture
                         ┌──────────────────────┐
                         │       WORKLOAD       │
                         └──────────┬───────────┘
                                    ▼
                         ┌──────────────────────┐
                         │      RUNTIME         │
                         └──────────┬───────────┘
                                    │
                     ┌──────────────┼──────────────┐
                     ▼              ▼              ▼
                Admission       Execution      Dependencies
                     │              │              │
                     └──────────────┼──────────────┘
                                    ▼
                         ┌──────────────────────┐
                         │ RESILIENCE CONTROL   │
                         ├──────────────────────┤
                         │ Timeout              │
                         │ Retry                │
                         │ Circuit Breaker      │
                         │ Bulkhead             │
                         │ Fallback             │
                         │ Degradation          │
                         │ Failover              │
                         │ Recovery             │
                         └──────────┬───────────┘
                                    │
                    ┌───────────────┼───────────────┐
                    ▼               ▼               ▼
                 Detect          Protect         Recover
                    │               │               │
                    └───────────────┼───────────────┘
                                    ▼
                         ┌──────────────────────┐
                         │    OBSERVABILITY     │
                         └──────────────────────┘
101. Resilience State Model
                 ┌───────────┐
                 │  HEALTHY  │
                 └─────┬─────┘
                       │
                  degradation
                       ▼
                 ┌───────────┐
                 │ DEGRADED  │
                 └─────┬─────┘
                       │
                     failure
                       ▼
                 ┌───────────┐
                 │ UNHEALTHY │
                 └─────┬─────┘
                       │
                    recovery
                       ▼
                 ┌───────────┐
                 │ RECOVERING│
                 └─────┬─────┘
                       │
                   validation
                       ▼
                 ┌───────────┐
                 │  HEALTHY  │
                 └───────────┘
102. Resilience Invariants
R1 — Critical failures must be isolated whenever technically possible.

R2 — No retry loop may be unbounded.

R3 — Every retry policy must define termination conditions.

R4 — External dependencies must have explicit timeout boundaries.

R5 — Circuit breakers must prevent repeated calls to known-failing dependencies.

R6 — Recovery must not automatically create uncontrolled traffic spikes.

R7 — Degradation must be explicit and observable.

R8 — Critical functionality must be protectable from non-critical workload.

R9 — Tenant failures must not automatically propagate to other tenants.

R10 — Failure handling must preserve data correctness.

R11 — Non-idempotent operations must not be blindly retried.

R12 — Failover must validate the target before full traffic restoration.

R13 — Recovery must be gradual where recovery storms are possible.

R14 — Resilience decisions must be policy-controlled.

R15 — Resilience state must be observable.

R16 — Root causes must be distinguishable from secondary failures.

R17 — Recovery mechanisms must themselves have capacity limits.

R18 — Resilience mechanisms must not become a larger source of load than the original workload.

R19 — A degraded system must have a defined path toward recovery.

R20 — A component marked unhealthy must not continue receiving normal traffic.

R21 — Shutdown and draining must preserve in-flight work according to policy.

R22 — Resilience behavior must be testable through controlled failure injection.

R23 — Critical protection mechanisms must not depend exclusively on AI decisions.

R24 — Recovery must not bypass security, authorization, policy, or tenant boundaries.
103. Engineering Completion Criteria

E38 queda completo cuando EVOXA posee:

✓ Failure taxonomy
✓ Failure domains
✓ Fault containment
✓ Fault isolation
✓ Bulkheads
✓ Tenant isolation
✓ Dependency isolation
✓ Timeout architecture
✓ Deadline propagation
✓ Retry architecture
✓ Retry classification
✓ Backoff
✓ Jitter
✓ Retry budgets
✓ Retry amplification protection
✓ Circuit breakers
✓ Circuit states
✓ Circuit scopes
✓ Fallback
✓ Graceful degradation
✓ Degradation modes
✓ Read-only degradation
✓ Write protection
✓ Idempotency
✓ Duplicate execution protection
✓ Failure detection
✓ Health model
✓ Liveness
✓ Readiness
✓ Recovery state
✓ Traffic reintroduction
✓ Recovery storm protection
✓ Failover
✓ Failback
✓ Data resilience
✓ RPO
✓ RTO
✓ Resilience tiers
✓ Resilience policies
✓ Resilience controller
✓ Resilience decisions
✓ Resilience events
✓ Failure correlation
✓ Root-cause distinction
✓ Failure storm control
✓ Resilience observability
✓ Resilience SLOs
✓ Chaos testing
✓ Fault injection
✓ Recovery testing
✓ Capacity integration
✓ Execution integration
✓ Workflow integration
✓ Job integration
✓ Scheduling integration
✓ Cache integration
✓ Configuration integration
✓ Feature-flag integration
✓ Policy integration
✓ Intelligence integration
✓ Decision integration
✓ Action integration
✓ Resource integration
✓ Resilience state machine
✓ Resilience control loop
✓ Resilience invariants
104. Cadena operativa consolidada

Con E37 + E38, el Runtime de EVOXA queda estructurado así:

                         WORKLOAD
                            │
                            ▼
                    ┌───────────────┐
                    │   CAPACITY    │
                    │     E37       │
                    └───────┬───────┘
                            │
                       admission
                            ▼
                    ┌───────────────┐
                    │   EXECUTION   │
                    │      E35      │
                    └───────┬───────┘
                            │
                     failure/stress
                            ▼
                    ┌───────────────┐
                    │  RESILIENCE   │
                    │      E38      │
                    └───────┬───────┘
                            │
             ┌──────────────┼──────────────┐
             ▼              ▼              ▼
          RETRY          FALLBACK       FAILOVER
             │              │              │
             └──────────────┼──────────────┘
                            ▼
                       RECOVERY
                            │
                            ▼
                       NORMAL STATE

La separación arquitectónica queda:

E36 — Resource: qué recursos existen y cómo se asignan.
E37 — Capacity: cuánto workload puede soportar el Runtime.
E38 — Resilience: cómo EVOXA contiene fallos, degrada funcionalidad y recupera el sistema.
E35 — Execution: cómo se ejecuta efectivamente el trabajo.

El siguiente capítulo natural es E39 — EVOXA Fault Tolerance Architecture, que debe bajar un nivel más: cómo se construyen los mecanismos concretos que permiten que una falla no produzca pérdida de servicio o pérdida de estado, incluyendo fault domains, redundancy, replication, state recovery, quorum, leader election, failover coordination y recovery guarantees.
