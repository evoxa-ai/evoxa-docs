E36 — EVOXA Resource Architecture
Engineering Specification

Pertenece a: Engineering Specification
Capítulo: E36 — Resource Architecture
Anterior: E35 — Execution Architecture
Siguiente: E37 — EVOXA Runtime Capacity Architecture

1. Propósito

E36 define cómo EVOXA modela, descubre, asigna, reserva, consume, limita, comparte, monitorea y libera recursos necesarios para ejecutar operaciones.

La relación fundamental es:

Action
   ↓
Execution
   ↓
Resource Requirements
   ↓
Resource Allocation
   ↓
Execution
   ↓
Resource Release

La regla central:

Resources are finite capabilities consumed by executions under explicit ownership, quota, policy, and lifecycle rules.

2. Resource Boundary

Resource Architecture responde:

¿Qué recurso existe?
¿Quién lo posee?
¿Quién puede usarlo?
¿Cuánto hay disponible?
¿Cuánto puede consumir una ejecución?
¿Cómo se reserva?
¿Cómo se libera?
¿Qué ocurre cuando no hay capacidad?

No decide qué Action debe ejecutarse.

Decision
  → qué hacer

Action
  → intención

Execution
  → realizarlo

Resource
  → con qué capacidad realizarlo
3. Resource Model

Un recurso EVOXA puede representarse como:

Resource
├── resourceId
├── resourceType
├── owner
├── tenant
├── scope
├── capacity
├── availability
├── state
├── policy
├── allocation
└── metadata
4. Resource Categories

EVOXA debe poder modelar diferentes categorías:

COMPUTE
MEMORY
STORAGE
NETWORK
DATABASE
CONNECTION
QUEUE
WORKER
API_QUOTA
RATE_LIMIT
CREDENTIAL
SECRET
LICENSE
FINANCIAL
HUMAN
EXTERNAL_SERVICE
5. Resource as a First-Class Concept

Un recurso no debe estar implícito exclusivamente dentro de un Executor.

Debe poder existir independientemente:

Resource
   ↓
Capability
   ↓
Executor
   ↓
Execution
6. Resource Identity

Cada recurso debe tener:

resourceId
resourceType
scope
owner

Ejemplo:

resourceId = db-primary-01
resourceType = DATABASE
scope = production
7. Resource Ownership

Un recurso puede pertenecer a:

SYSTEM
TENANT
TEAM
SERVICE
APPLICATION
EXTERNAL_PROVIDER
USER

El ownership determina responsabilidad, no necesariamente acceso.

8. Resource Scope

El alcance puede ser:

GLOBAL
REGION
ENVIRONMENT
TENANT
DOMAIN
SERVICE
WORKER
EXECUTION
9. Resource Lifecycle
DISCOVERED
    ↓
REGISTERED
    ↓
AVAILABLE
    ↓
ALLOCATED
    ↓
IN_USE
    ↓
RELEASED
    ↓
AVAILABLE

Estados alternativos:

DEGRADED
EXHAUSTED
QUARANTINED
DISABLED
RETIRED
10. Resource Registration

Todo recurso administrado por EVOXA debe poder registrarse:

Resource Registry
      ↓
Resource Definition
      ↓
Resource Metadata

El Registry proporciona identidad y descubrimiento.

11. Resource Discovery

Una Execution puede solicitar:

Resource Requirement

y resolverlo contra:

Resource Registry

Ejemplo:

DATABASE
capacity >= X
region = EU
tenant = T1
12. Resource Requirement

Una Execution puede declarar:

ResourceRequirement
├── resourceType
├── quantity
├── unit
├── scope
├── constraints
├── priority
├── duration
└── exclusivity
13. Resource Quantity

La cantidad debe tener una unidad explícita:

2 workers
4 GB memory
100 API requests/min
1 database connection
10 GB storage
14. Resource Capacity

La capacidad representa cuánto puede ofrecer un recurso.

Resource Capacity
├── total
├── allocated
├── consumed
└── available

Relación:

available =
total - allocated

cuando el modelo sea puramente reservacional.

15. Allocatable vs Consumable

Debe distinguirse:

Allocatable

Puede reservarse.

worker slot
CPU
memory
connection
Consumable

Se consume irreversiblemente o por ventana.

API quota
credits
financial budget
storage allowance
16. Shareable vs Exclusive

Un recurso puede ser:

SHAREABLE
EXCLUSIVE
PARTITIONABLE

Ejemplo:

Database → SHAREABLE
GPU → EXCLUSIVE
CPU → PARTITIONABLE
17. Resource Allocation

Allocation representa:

quién
qué
cuánto
por cuánto tiempo

consume o reserva un recurso.

Allocation
├── allocationId
├── resourceId
├── consumer
├── quantity
├── start
├── expiry
├── state
└── executionId
18. Resource Consumer

El consumidor puede ser:

EXECUTION
WORKER
JOB
WORKFLOW
SERVICE
TENANT
19. Allocation Lifecycle
REQUESTED
   ↓
APPROVED
   ↓
RESERVED
   ↓
ACTIVE
   ↓
RELEASED

Alternativas:

REJECTED
EXPIRED
REVOKED
FAILED
20. Reservation

Una Reservation garantiza capacidad futura:

Resource
   ↓
Reservation
   ↓
Execution

Esto es importante para operaciones con recursos escasos.

21. Reservation vs Allocation
Reservation
→ capacidad prometida

Allocation
→ capacidad asignada

Consumption
→ capacidad efectivamente utilizada
22. Resource Consumption

El consumo real puede diferir de la asignación:

Allocated: 4 GB
Consumed: 2.7 GB

Debe poder medirse cuando sea relevante.

23. Resource Release

Al finalizar:

Execution
   ↓
Release
   ↓
Resource Available

La liberación debe ser segura incluso si la Execution falla.

24. Resource Leak

Un leak ocurre cuando:

Execution finished
        ↓
Allocation remains active

Resource Architecture debe prevenir y detectar estos casos.

25. Allocation Lease

Para recursos temporales:

Lease
├── owner
├── acquiredAt
├── expiresAt
└── renewal

Si expira:

Allocation
   ↓
EXPIRED
   ↓
Resource Reclaimed
26. Resource Lock

Para recursos no compartibles:

Resource
   ↓
Lock
   ↓
Execution A

Otra Execution debe:

wait
reject
or defer

según política.

27. Lock Ownership

Todo lock debe identificar:

lockId
resourceId
owner
executionId
expiresAt
28. Deadlock Prevention

Cuando una Execution requiere múltiples recursos:

Resource A
Resource B
Resource C

debe existir una estrategia para evitar deadlocks.

Opciones:

ordered acquisition
timeouts
deadlock detection
rollback
29. Resource Dependencies

Un recurso puede depender de otros:

Execution
   ↓
Application
   ↓
Database
   ↓
Storage

La disponibilidad efectiva depende de la cadena.

30. Resource Health

Cada recurso puede tener:

HEALTHY
DEGRADED
UNAVAILABLE
UNKNOWN

Health no equivale necesariamente a allocation availability.

31. Resource Availability

Disponibilidad debe considerar:

capacity
health
policy
quota
maintenance
location
tenant
32. Resource Selection

Cuando existen múltiples recursos compatibles:

Resource A
Resource B
Resource C

puede utilizarse:

capacity
latency
cost
region
health
load
policy

para seleccionar.

33. Resource Resolver

El Resolver transforma:

Resource Requirement

en:

Concrete Resource

Ejemplo:

"database, production, EU"
       ↓
db-eu-prod-02
34. Resource Pool

Recursos homogéneos pueden agruparse:

Worker Pool
├── Worker 1
├── Worker 2
├── Worker 3
└── Worker 4

El Pool administra capacidad agregada.

35. Pool Allocation
Execution
   ↓
Worker Pool
   ↓
Worker 3

La selección concreta puede ser dinámica.

36. Resource Partition

Un recurso puede dividirse:

Resource
├── Partition A
├── Partition B
└── Partition C

Esto permite aislamiento y control de capacidad.

37. Resource Hierarchy

Los recursos pueden formar jerarquías:

Cluster
 ├── Node
 │    ├── CPU
 │    └── Memory
 └── Node
      ├── CPU
      └── Memory
38. Parent-Child Capacity

La capacidad de un hijo no debe superar la capacidad efectiva del padre.

Cluster
  ↓
Node
  ↓
Worker
  ↓
Execution
39. Resource Quota

Una cuota limita cuánto puede consumir un sujeto:

Quota
├── subject
├── resourceType
├── limit
├── window
└── policy

Ejemplo:

Tenant A
API Calls
10,000 / hour
40. Quota Dimensions

Las cuotas pueden aplicarse por:

tenant
user
service
actionType
executor
region
environment
41. Quota Enforcement

Antes de allocation:

Request
 ↓
Quota Check
 ↓
Allow / Reject / Defer
42. Quota Consumption

Debe registrarse:

limit
used
remaining
resetAt
43. Rate Limit

Rate limiting controla velocidad:

requests / second
requests / minute
requests / hour

Quota controla volumen acumulado.

No son equivalentes.

44. Resource Budget

Algunos recursos necesitan presupuesto:

Financial Budget
Compute Budget
API Credit Budget
Storage Budget

Modelo:

Budget
├── allocated
├── consumed
├── remaining
└── period
45. Cost-Aware Allocation

Cuando existen recursos equivalentes:

Resource A → $1
Resource B → $0.20

el resolver puede considerar coste.

La optimización de coste nunca debe saltarse:

security
policy
availability
correctness
46. Priority Allocation

En escasez:

Critical Execution
High Execution
Normal Execution
Low Execution

pueden recibir distinta prioridad.

47. Fairness

La asignación no debe permitir que un tenant monopolice recursos compartidos.

Puede utilizar:

fair scheduling
weighted allocation
tenant quotas
burst limits
48. Burst Capacity

Un tenant puede superar temporalmente su nivel normal:

baseline
   ↓
burst
   ↓
throttle

si la política lo permite.

49. Resource Preemption

En recursos escasos:

Low Priority Execution
        ↓
PREEMPTED
        ↓
High Priority Execution

La preemption debe ser explícita.

50. Preemption Safety

No todas las ejecuciones pueden interrumpirse.

Debe declararse:

preemptible = true | false
51. Resource Reclamation

Cuando una allocation queda huérfana:

Allocation
   ↓
Lease Expired
   ↓
Reclamation
   ↓
Resource Available
52. Resource Garbage Collection

Resources temporales pueden necesitar:

detect
verify
release
cleanup

automáticamente.

53. Resource Cleanup

Cleanup puede incluir:

connections
temporary files
locks
workers
memory
leases
temporary credentials
54. Resource Lifecycle Events

Eventos mínimos:

ResourceRegistered
ResourceAvailable
ResourceDegraded
ResourceUnavailable
ResourceAllocated
ResourceReleased
ResourceExhausted
ResourceReclaimed
ResourceRetired
55. Allocation Events
AllocationRequested
AllocationApproved
AllocationReserved
AllocationActivated
AllocationReleased
AllocationExpired
AllocationRevoked
56. Resource Observability

Debe poder observarse:

capacity
utilization
allocation
consumption
availability
health
saturation
57. Resource Metrics

Métricas:

resource_capacity
resource_available
resource_allocated
resource_utilization
resource_saturation
allocation_count
allocation_failures
allocation_latency
resource_leaks
resource_reclaims
quota_exceeded
58. Utilization

Utilización:

used / available capacity

debe distinguirse de:

reserved capacity

Ejemplo:

Capacity = 100
Reserved = 80
Used = 50

No significa que haya 50 disponibles para nuevas allocations si las 80 están comprometidas.

59. Saturation

Un recurso puede estar:

high utilization

sin estar técnicamente exhausted.

Saturation representa incapacidad práctica de aceptar trabajo adicional.

60. Resource Health vs Capacity

Un recurso puede tener:

capacity = 100
health = degraded

Por tanto capacity sola no determina disponibilidad.

61. Resource Dependency Health

La disponibilidad efectiva puede ser:

Resource
 AND
Dependency A
 AND
Dependency B

Si una dependencia crítica está caída, el recurso puede considerarse no utilizable.

62. Resource Isolation

Los recursos deben aislarse por:

tenant
environment
security boundary
data classification

cuando corresponda.

63. Shared Resources

Para recursos compartidos:

Tenant A ─┐
Tenant B ─┼── Shared Resource
Tenant C ─┘

deben existir límites claros.

64. Dedicated Resources

Para cargas sensibles:

Tenant A
   ↓
Dedicated Resource

puede ofrecer aislamiento superior.

65. Hybrid Allocation

Un tenant puede tener:

Dedicated baseline
+
Shared burst capacity
66. Resource Security

Resource access debe requerir:

identity
authorization
tenant context
policy
credential
67. Resource Credentials

Credentials no deben ser consideradas Resources ordinarios desde el punto de vista de exposición.

Deben gestionarse mediante:

Secret Management
Credential Provider
Identity

Resource Architecture define el consumo; Security Architecture define protección.

68. Resource Policy

Una Resource Policy puede definir:

who
can allocate
what
how much
where
when
for how long
69. Resource Admission

Antes de allocation:

Requirement
 ↓
Policy
 ↓
Quota
 ↓
Capacity
 ↓
Health
 ↓
Allocation
70. Resource Allocation Failure

Razones:

NO_CAPACITY
QUOTA_EXCEEDED
POLICY_DENIED
RESOURCE_UNAVAILABLE
RESOURCE_LOCKED
INVALID_REQUIREMENT
DEADLINE_EXCEEDED
71. Deferred Allocation

Si no existe capacidad:

REQUEST
 ↓
DEFERRED
 ↓
WAIT
 ↓
RETRY

en lugar de fallar inmediatamente, cuando la política lo permita.

72. Allocation Deadline

Una petición puede tener:

allocationDeadline

Después:

DEADLINE_EXCEEDED
73. Resource Reservation Priority

Reservations pueden tener prioridad:

CRITICAL
HIGH
NORMAL
LOW

Esto es útil para capacidad futura.

74. Resource Reservation Conflict

Si dos reservations compiten:

Reservation A
Reservation B

la decisión debe considerar:

priority
policy
tenant
deadline
fairness
75. Resource Transaction

Cuando una Execution requiere varios recursos:

Allocate A
Allocate B
Allocate C

debe evitarse quedar en un estado parcial.

Puede utilizar:

all-or-nothing allocation

cuando sea viable.

76. Partial Allocation

Si no puede garantizarse atomicidad:

A allocated
B allocated
C failed

debe existir compensación:

release A
release B
77. Resource Reservation Saga

Para asignaciones distribuidas:

Reserve A
 ↓
Reserve B
 ↓
Reserve C

con compensaciones si falla una etapa.

78. Resource Registry

Conceptualmente:

Resource Registry
├── Resources
├── Resource Types
├── Capabilities
├── Pools
├── Policies
└── Metadata
79. Resource Discovery API

Conceptualmente:

GET /resources
GET /resources/{id}
GET /resources/{id}/capacity
GET /resources/{id}/health
POST /resources/{id}/reservations
DELETE /resources/{id}/reservations/{reservationId}

Los contratos definitivos pertenecen a la API Architecture.

80. Resource Repository

Debe soportar:

register
update
find
allocate
release
reserve
reconcile
81. Resource State Store

Para estado operativo:

current capacity
current allocations
leases
locks
health
82. Resource History

Debe conservarse cuando sea relevante:

allocation history
capacity changes
health changes
policy changes
ownership changes
83. Resource Reconciliation

El sistema debe comparar:

Expected State
      vs
Observed State

Ejemplo:

Expected allocations = 10
Actual allocations = 8

Esto puede revelar leaks o fallos de sincronización.

84. External Resource Reconciliation

Para recursos externos:

EVOXA Registry
      vs
Provider State

debe poder reconciliarse.

85. Resource Drift

Drift ocurre cuando:

Declared Resource
        ≠
Actual Resource

Debe detectarse y resolverse.

86. Resource Provisioning

Resource Architecture puede solicitar creación:

Resource Requirement
 ↓
Provisioning
 ↓
Resource Registered
 ↓
Available

La implementación concreta puede pertenecer a Deployment/Infrastructure Architecture.

87. Dynamic Resources

Algunos recursos existen solo durante una Execution:

Execution
 ↓
Provision Worker
 ↓
Execute
 ↓
Release Worker
88. Ephemeral Resources

Ejemplos:

temporary container
temporary worker
temporary connection
temporary storage
temporary credential

Deben tener lifecycle explícito.

89. Resource Lifecycle Ownership

Debe existir un responsable de:

creation
allocation
monitoring
release
retirement
90. Resource Retirement

Un recurso puede pasar:

AVAILABLE
 ↓
RETIRING
 ↓
RETIRED

Durante RETIRING no deben aceptarse nuevas allocations, salvo excepción explícita.

91. Resource Drain

Antes de retirar:

Resource
 ↓
DRAINING
 ↓
Active allocations finish
 ↓
RETIRED
92. Resource Maintenance

Maintenance puede marcar:

MAINTENANCE

y evitar nuevas allocations.

93. Resource Failover

Cuando un recurso falla:

Resource A
   X
Resource B
   ↓
Execution

El failover debe respetar:

compatibility
state
data consistency
policy
94. Resource Affinity

Una Execution puede requerir:

same region
same node
same tenant pool
same database

La selección debe soportar affinity.

95. Resource Anti-Affinity

También puede requerirse evitar concentración:

Execution A → Node 1
Execution B → Node 2

para resiliencia.

96. Resource Locality

La selección puede optimizar:

network distance
region
zone
data locality
latency
97. Resource Capability Matching

No basta con buscar:

resourceType = WORKER

Puede requerirse:

capabilities:
  GPU
  language = Python
  version >= X
98. Capability vs Resource

Distinción:

Resource
→ capacidad física/lógica disponible

Capability
→ qué puede hacer ese recurso

Ejemplo:

GPU Worker
Resource = worker-42
Capability = GPU inference
99. Resource Pool Selection

La selección puede seguir:

Requirement
 ↓
Filter
 ↓
Policy
 ↓
Quota
 ↓
Health
 ↓
Capacity
 ↓
Cost
 ↓
Affinity
 ↓
Selected Resource
100. Resource Allocation Algorithm

Una implementación de referencia:

1. Validate requirement
2. Resolve tenant/context
3. Evaluate policy
4. Check quota
5. Discover candidates
6. Filter unhealthy resources
7. Check capacity
8. Apply affinity constraints
9. Rank candidates
10. Reserve
11. Confirm allocation
12. Return allocation
101. Resource Release Algorithm
1. Validate ownership
2. Locate allocation
3. Stop consumption
4. Release locks/leases
5. Update capacity
6. Persist release
7. Emit event
8. Verify consistency
102. Resource Allocation with Execution

La relación principal:

Execution
    │
    ▼
Resource Requirements
    │
    ▼
Admission
    │
    ▼
Allocation
    │
    ▼
Executor
    │
    ▼
Execution Result
    │
    ▼
Release
103. Execution Failure + Resource Release

Incluso ante:

FAILED
TIMED_OUT
CANCELLED

debe ejecutarse:

cleanup
release
reconciliation

cuando corresponda.

104. Resource Failure + Execution

Si un recurso falla durante ejecución:

Execution
   ↓
Resource Failure
   ↓
Recovery Policy
   ├── retry
   ├── failover
   ├── pause
   ├── compensate
   └── terminate
105. Resource State Machine
              ┌──────────────┐
              │  REGISTERED  │
              └──────┬───────┘
                     ▼
              ┌──────────────┐
              │   AVAILABLE  │
              └──────┬───────┘
                     │
            ┌────────┼────────┐
            ▼        ▼        ▼
       ALLOCATED  DEGRADED  DISABLED
            │
            ▼
         IN_USE
            │
       ┌────┴─────┐
       ▼          ▼
   RELEASED     FAILED
       │          │
       ▼          ▼
   AVAILABLE   RECOVERY
                     │
                     ▼
                 AVAILABLE
106. Resource Invariants

Reglas:

R1 — Every managed resource has an identity.
R2 — Allocation requires an identifiable consumer.
R3 — Allocation cannot exceed allocatable capacity.
R4 — Consumption cannot exceed permitted allocation.
R5 — Resource ownership must be explicit.
R6 — Tenant boundaries must be enforced.
R7 — Leases must expire or be renewed.
R8 — Released resources must become reclaimable.
R9 — Resource state must be observable.
R10 — Resource failures must be recoverable or terminally classified.
R11 — Resource allocation must be policy-controlled.
R12 — Quotas must be enforced independently of application intent.
R13 — Resource leaks must be detectable.
R14 — External resource state must be reconcilable.
R15 — Execution must never bypass resource controls.
107. Reference Architecture
                         EXECUTION
                             │
                             ▼
                  ┌────────────────────┐
                  │ RESOURCE REQUIREMENT│
                  └──────────┬─────────┘
                             │
                             ▼
                  ┌────────────────────┐
                  │ RESOURCE ADMISSION │
                  ├────────────────────┤
                  │ Policy             │
                  │ Quota              │
                  │ Capacity           │
                  │ Health             │
                  │ Security           │
                  └──────────┬─────────┘
                             │
                             ▼
                  ┌────────────────────┐
                  │ RESOURCE RESOLVER  │
                  └──────────┬─────────┘
                             │
                             ▼
                  ┌────────────────────┐
                  │ RESOURCE POOL      │
                  └──────────┬─────────┘
                             │
                 ┌───────────┼───────────┐
                 ▼           ▼           ▼
             Resource A  Resource B  Resource C
                 │           │           │
                 └───────────┼───────────┘
                             ▼
                  ┌────────────────────┐
                  │    ALLOCATION      │
                  └──────────┬─────────┘
                             │
                             ▼
                  ┌────────────────────┐
                  │     EXECUTOR       │
                  └──────────┬─────────┘
                             │
                             ▼
                  ┌────────────────────┐
                  │    CONSUMPTION     │
                  └──────────┬─────────┘
                             │
                             ▼
                  ┌────────────────────┐
                  │ RELEASE / RECLAIM  │
                  └──────────┬─────────┘
                             │
                             ▼
                  ┌────────────────────┐
                  │ RESOURCE REGISTRY  │
                  └────────────────────┘
108. Relationship with E35

La frontera debe quedar inequívoca:

E35 Execution
      │
      │ requests
      ▼
E36 Resource
      │
      │ allocates
      ▼
Resource
      │
      │ enables
      ▼
Execution

E35 responde:

¿Cómo realizamos la operación?

E36 responde:

¿Qué capacidad necesitamos y cómo obtenemos el derecho a consumirla?

109. Relationship with Runtime
Resource
   ↓
Runtime
   ↓
Execution

Runtime proporciona el entorno operativo.

Resource Architecture controla la capacidad disponible dentro de ese entorno.

110. Relationship with Infrastructure

Infrastructure puede crear:

compute
storage
network
databases

Resource Architecture los convierte en:

managed resources

que pueden ser descubiertos y asignados por EVOXA.

111. Relationship with Governance

Governance establece:

who may use resources
how much
under which conditions

Resource Architecture implementa:

allocation
quota
reservation
consumption
release
112. Relationship with Security

Security protege:

identity
credentials
resource access
tenant boundaries

Resource Architecture gestiona:

capacity
allocation
ownership
consumption
113. Relationship with Scheduling

Scheduler:

WHEN

Resource:

WITH WHAT CAPACITY

Execution:

HOW

Por tanto:

Scheduler
    ↓
Execution
    ↓
Resource Allocation
114. Relationship with Workflow

Workflow puede coordinar:

Execution A
 ↓
Resource allocation
 ↓
Execution B
 ↓
Resource allocation

pero no debe convertirse en Resource Manager.

115. Relationship with Multi-Tenancy

Cada allocation debe poder responder:

Which tenant owns it?
Which tenant consumes it?
Which quota applies?
Which isolation boundary applies?
116. Relationship with Observability

Observability debe poder mostrar:

Resource
 ↓
Allocation
 ↓
Execution
 ↓
Outcome

Esto permite explicar:

"La ejecución falló porque el recurso requerido estaba agotado."

117. Resource Architecture Completion Criteria

E36 queda arquitectónicamente completo cuando EVOXA posee:

✓ Resource model
✓ Resource identity
✓ Resource ownership
✓ Resource scope
✓ Resource categories
✓ Resource lifecycle
✓ Resource registry
✓ Resource discovery
✓ Resource requirements
✓ Capacity model
✓ Allocatable resources
✓ Consumable resources
✓ Shareable resources
✓ Exclusive resources
✓ Resource allocation
✓ Reservation
✓ Consumption
✓ Release
✓ Lease
✓ Lock
✓ Deadlock prevention
✓ Resource pools
✓ Resource partitions
✓ Resource hierarchy
✓ Quotas
✓ Rate limits
✓ Budgets
✓ Cost-aware allocation
✓ Fairness
✓ Burst capacity
✓ Preemption
✓ Reclamation
✓ Garbage collection
✓ Health
✓ Availability
✓ Resource resolution
✓ Capability matching
✓ Affinity
✓ Anti-affinity
✓ Locality
✓ Provisioning boundary
✓ Ephemeral resources
✓ Retirement
✓ Maintenance
✓ Drain
✓ Failover
✓ Reconciliation
✓ Drift detection
✓ Resource security
✓ Resource policy
✓ Admission control
✓ Deferred allocation
✓ Allocation deadlines
✓ Multi-resource allocation
✓ Partial allocation handling
✓ Resource events
✓ Metrics
✓ Observability
✓ Execution integration
✓ Runtime integration
✓ Governance integration
✓ Security integration
✓ Scheduler integration
✓ Workflow integration
✓ Multi-tenant isolation
✓ Resource invariants
118. Cadena arquitectónica actual

Con E35 y E36, el núcleo operativo queda:

E32 Intelligence
       ↓
E33 Decision
       ↓
E34 Action
       ↓
E35 Execution
       ↓
E36 Resource
       ↓
Actual Capacity
       ↓
Execution Result
       ↓
Outcome

Pero operacionalmente la relación real es:

                  ┌───────────────┐
                  │    ACTION     │
                  └───────┬───────┘
                          ▼
                  ┌───────────────┐
                  │   EXECUTION   │
                  └───────┬───────┘
                          │
                  requires│
                          ▼
                  ┌───────────────┐
                  │    RESOURCE   │
                  └───────┬───────┘
                          │
                    allocates
                          ▼
                  ┌───────────────┐
                  │   CAPACITY    │
                  └───────┬───────┘
                          │
                       enables
                          ▼
                  ┌───────────────┐
                  │    EFFECT     │
                  └───────────────┘

La distinción clave queda establecida:

Execution realiza; Resource habilita.

El siguiente capítulo natural es E37 — EVOXA Runtime Capacity Architecture, donde se baja un nivel más: cómo Runtime administra workers, pools, CPU, memoria, concurrencia, autoscaling, saturation, backpressure y capacity planning sobre los Resources definidos aquí.
