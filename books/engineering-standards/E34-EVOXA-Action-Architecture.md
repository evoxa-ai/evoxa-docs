E34 — EVOXA Action Architecture
Engineering Specification

Pertenece a: Engineering Specification
Capítulo: E34 — Action Architecture
Anterior: E33 — Decision Architecture
Siguiente: E35 — Execution Architecture

1. Propósito

E34 define la arquitectura de Action de EVOXA.

Su responsabilidad es transformar una decisión válida y autorizada en una acción ejecutable, expresando:

qué hacer
sobre qué
con qué parámetros
bajo qué condiciones
con qué autoridad
y con qué garantías

La frontera fundamental es:

Decision
   ↓
Action
   ↓
Execution

Por tanto:

Decision selecciona qué debe hacerse; Action define la operación que debe ejecutarse; Execution realiza físicamente esa operación.

2. Action ≠ Execution

Esta separación es crítica.

Una Action es una intención operacional formalizada.

Una Execution es el proceso mediante el cual esa intención se lleva a cabo.

Ejemplo:

Decision:
Approve customer refund

Action:
Issue refund of $75 to customer X

Execution:
Payment provider API call
3. Action Architecture
                         ACTION
                            │
              ┌─────────────┼─────────────┐
              ▼             ▼             ▼
           Decision       Policy       Context
              │             │             │
              └─────────────┼─────────────┘
                            ▼
                     Action Resolution
                            │
                            ▼
                     Action Validation
                            │
                            ▼
                    Authorization Check
                            │
                            ▼
                    Action Preparation
                            │
                            ▼
                       Action
                            │
              ┌─────────────┼─────────────┐
              ▼             ▼             ▼
          Execution       Workflow       Event
4. Action Boundary

La arquitectura debe mantener:

Intelligence
      ↓
Decision
      ↓
Action
      ↓
Execution

No debe mezclarse:

Decision logic

con:

Execution mechanics
5. Action Definition

Una Action representa:

"Perform operation X on subject Y with parameters Z."

Modelo conceptual:

Action
├── id
├── type
├── subject
├── operation
├── parameters
├── decision
├── actor
├── authority
├── policy
├── constraints
├── preconditions
├── postconditions
├── status
├── priority
├── deadline
└── version
6. Action Types

EVOXA puede clasificar acciones como:

CREATE
UPDATE
DELETE
APPROVE
REJECT
NOTIFY
SEND
TRANSFER
ALLOCATE
ASSIGN
ROUTE
SCHEDULE
CANCEL
ESCALATE
TRIGGER
INVOKE
PUBLISH

Los dominios pueden extender estos tipos.

7. Action Intent

Toda acción debe expresar claramente:

intent

Ejemplo:

Intent:
Notify customer about approved refund.

El intent debe permanecer independiente del mecanismo concreto de ejecución.

8. Action Parameters

Una acción puede contener:

ActionParameters
├── input
├── target
├── operation
├── options
└── metadata

Los parámetros deben validarse antes de la ejecución.

9. Action Subject

Toda Action debe identificar el objeto sobre el que opera cuando corresponda:

customer
order
invoice
subscription
account
workflow
resource
document
tenant
10. Action Target

El target identifica el recurso o sistema receptor:

customer
internal service
external API
database
queue
workflow
agent
human
11. Action Actor

Debe registrarse quién origina la acción:

user
agent
workflow
system
service
policy
scheduled process
12. Action Authority

La autoridad debe propagarse desde Decision:

Decision Authority
        ↓
Action Authority

Una Action no debe obtener implícitamente más autoridad que la Decision que la origina.

13. Authority Inheritance

Regla:

ActionAuthority
≤
DecisionAuthority

salvo una delegación explícita.

14. Action Authorization

Authorization responde:

Can this actor perform this action?

Mientras Decision Authority responde:

Was this actor allowed to choose this action?

Ambos controles deben mantenerse.

15. Action Preconditions

Una Action puede requerir:

Preconditions
├── resource available
├── state valid
├── authorization valid
├── policy valid
├── decision valid
└── context fresh

Si una precondition falla, la Action no debe ejecutarse.

16. Action Postconditions

Debe definirse qué debe ser cierto después de una ejecución exitosa:

Postconditions
├── state changed
├── event emitted
├── resource updated
└── outcome recorded
17. Action Lifecycle
REQUESTED
    ↓
RESOLVED
    ↓
VALIDATED
    ↓
AUTHORIZED
    ↓
PREPARED
    ↓
READY
    ↓
DISPATCHED
    ↓
EXECUTING
    ↓
COMPLETED

Estados alternativos:

REJECTED
CANCELLED
EXPIRED
FAILED
BLOCKED
COMPENSATED
18. Action Request

Una Action Request representa:

"Perform this operation."

Modelo:

ActionRequest
├── id
├── decisionId
├── actor
├── actionType
├── subject
├── parameters
├── priority
├── deadline
└── idempotencyKey
19. Action Resolution

La resolución transforma:

Decision

en:

Concrete Action

Ejemplo:

Decision:
Approve refund

Action:
POST Refund(customerId, amount)
20. Action Planning

Una Decision puede producir una única Action:

Decision
   ↓
Action

o una secuencia:

Decision
   ↓
Action A
   ↓
Action B
   ↓
Action C

Cuando existe una secuencia compleja, debe delegarse al Workflow/Orchestration layer.

21. Atomic Action

Una Action puede ser atómica:

Update Customer Status

No debería contener lógica de negocio extensa.

22. Composite Action

Puede representar:

CompositeAction
├── Action A
├── Action B
└── Action C

pero la coordinación debe pertenecer a Workflow cuando exista dependencia compleja entre pasos.

23. Action vs Workflow
Action
→ operation

Workflow
→ coordination of operations

Ejemplo:

Action:
Send notification

Workflow:
Validate customer
→ update account
→ send notification
→ schedule follow-up
24. Action Command

La Action puede materializarse como un command:

ActionCommand
├── commandId
├── actionType
├── subject
├── parameters
├── actor
├── authorization
└── correlationId
25. Command Boundary

El Command representa:

what should happen

El executor representa:

how it happens
26. Action Executor

El executor debe:

receive action
validate executable state
invoke target
capture result
publish outcome

No debe decidir si la acción era estratégicamente correcta.

27. Action Handlers

Puede existir un handler por tipo:

CreateHandler
UpdateHandler
NotifyHandler
TransferHandler
AssignHandler
ScheduleHandler
CancelHandler
28. Action Adapter

Cuando el target es externo:

Action
 ↓
Adapter
 ↓
External System

El Adapter traduce el contrato interno al contrato externo.

29. Action Gateway

Puede utilizarse un gateway para:

authentication
authorization
routing
rate limiting
resilience
observability

antes de acceder a sistemas externos.

30. Action Validation

Debe validar:

schema
parameters
subject
state
authority
policy
constraints
preconditions
freshness
31. Validation Layers
Syntax
   ↓
Schema
   ↓
Domain
   ↓
Policy
   ↓
Authorization
   ↓
Runtime Preconditions
32. Action Policy

Policy puede definir:

whether an action is allowed

Ejemplo:

Refund > $1,000
→ manager approval required
33. Action Constraints

Las restricciones pueden incluir:

amount
frequency
time
resource
tenant
geography
risk
rate
34. Action Limits

Una Action puede estar limitada por:

maxAmount
maxRetries
maxDuration
maxFrequency
allowedTargets
35. Action Expiration

Una Action puede dejar de ser válida:

validUntil

Ejemplo:

Approve transfer
valid for 10 minutes

Una Action expirada debe pasar a:

EXPIRED
36. Action Freshness

Antes de ejecutar puede ser necesario comprobar:

Decision still valid?
Context still valid?
Policy still valid?
Authority still valid?
37. Pre-Execution Validation

Para acciones críticas:

Action
 ↓
Revalidate
 ↓
Authorize
 ↓
Execute

Esto evita ejecutar una intención que ya quedó obsoleta.

38. Action Idempotency

Toda Action con efectos externos debe considerar:

idempotencyKey

Ejemplo:

refund:customer123:invoice456

para evitar duplicaciones.

39. Exactly-Once Illusion

La arquitectura no debe asumir que los sistemas externos proporcionan exactamente-once.

Debe diseñarse alrededor de:

at-least-once delivery
+
idempotent execution

cuando corresponda.

40. Action Retry

Los retries deben distinguir:

transient failure

de:

permanent failure
41. Retry Policy

Puede definir:

maxAttempts
backoff
jitter
retryableErrors
deadline
42. Retry Safety

No todas las acciones pueden repetirse.

Ejemplo:

Send email

puede tolerar cierto retry.

Charge credit card

requiere idempotency y controles adicionales.

43. Action Timeout

Toda acción remota debería tener:

timeout

adecuado a su naturaleza.

44. Action Cancellation

Una Action puede cancelarse si:

not started
still cancellable
policy permits cancellation

No toda Action puede cancelarse una vez iniciada.

45. Cancellation States
CANCEL_REQUESTED
CANCELLED
CANCEL_REJECTED
46. Action Failure

Una acción puede fallar por:

VALIDATION_FAILURE
AUTHORIZATION_FAILURE
POLICY_FAILURE
DEPENDENCY_FAILURE
TIMEOUT
RATE_LIMIT
NETWORK_FAILURE
BUSINESS_FAILURE
UNKNOWN_FAILURE
47. Failure Classification

Debe conservarse:

failureCode
failureCategory
retryable
recoverable
compensatable
48. Compensation

Cuando una acción ya ocurrió pero debe revertirse:

Action
 ↓
Compensation Action

Ejemplo:

Reserve inventory
       ↓
Payment fails
       ↓
Release inventory
49. Compensation ≠ Rollback

No todos los sistemas soportan rollback transaccional.

Por eso:

Compensation

es una nueva Action diseñada para devolver el sistema a un estado aceptable.

50. Action Saga

Para procesos distribuidos:

Action A
 ↓
Action B
 ↓
Action C

pueden existir compensaciones:

C → Compensate C
B → Compensate B
A → Compensate A

La coordinación pertenece a Workflow/Orchestration.

51. Action Result

Una ejecución debe producir:

ActionResult
├── actionId
├── status
├── output
├── target
├── executionId
├── timestamps
├── errors
└── metadata
52. Action Outcome

El ActionResult describe ejecución.

El Outcome describe impacto:

ActionResult
→ Did execution succeed?

Outcome
→ What happened in reality?
53. Action Events

Eventos principales:

ActionRequested
ActionResolved
ActionValidated
ActionAuthorized
ActionPrepared
ActionReady
ActionDispatched
ActionStarted
ActionCompleted
ActionFailed
ActionCancelled
ActionExpired
ActionCompensated
54. Event Correlation

Todos los eventos deben poder relacionarse:

decisionId
actionId
executionId
workflowId
correlationId
causationId
55. Causation Chain
DecisionCommitted
       ↓
ActionRequested
       ↓
ExecutionStarted
       ↓
ExecutionCompleted
       ↓
OutcomeRecorded

Esto proporciona trazabilidad end-to-end.

56. Action → Execution

La frontera:

Action
   ↓
Execution Request

debe ser explícita.

Action no debe contener detalles específicos de:

thread
container
HTTP connection
worker
process

salvo que formen parte del contrato operacional.

57. Execution Independence

Una misma Action puede ejecutarse mediante:

synchronous executor
asynchronous worker
workflow engine
external integration
human task
agent

sin modificar necesariamente la definición de Action.

58. Action Routing

El Action Router determina:

Which executor should handle this action?

Ejemplo:

SEND_EMAIL
→ Notification Service

TRANSFER_FUNDS
→ Payment Service

UPDATE_CUSTOMER
→ Customer Service
59. Action Registry

Puede existir un registro:

Action Registry
├── action types
├── schemas
├── handlers
├── policies
├── permissions
├── retry strategies
└── execution targets
60. Action Capability

Un executor anuncia capacidades:

Capability
├── actionType
├── version
├── supportedParameters
├── limits
└── availability

Esto permite routing basado en capability.

61. Action Discovery

El sistema puede consultar:

What actions are available for this context?

Esto es especialmente útil para Agents.

62. Agent Action Boundary

Un Agent puede:

discover
propose
request
execute

pero sus permisos deben limitarse explícitamente.

63. Agent Tool Invocation

Cuando un Agent llama una herramienta:

Agent
 ↓
Action
 ↓
Tool
 ↓
Execution

La llamada debe quedar registrada como Action.

64. Action and Human Task

Una Action puede requerir intervención humana:

Action
 ↓
Human Task
 ↓
Completion

Ejemplo:

Review high-risk transaction
65. Human Action Authority

Las acciones humanas también deben registrar:

actor
role
authority
timestamp
result
66. Action Security

Controles:

authentication
authorization
least privilege
tenant isolation
input validation
secret management
audit
non-repudiation
67. Least Privilege

Un Action Executor solo debe recibir:

minimum permissions

necesarios para ejecutar esa Action.

68. Credential Isolation

Los secretos del target externo no deben formar parte arbitrariamente del objeto Action.

Deben resolverse mediante:

credential provider
secret manager
secure runtime context
69. Tenant Isolation

Una Action de tenant A no puede operar sobre recursos de tenant B salvo una política explícita de cross-tenant operation.

70. Action Data Classification

Los parámetros pueden contener:

public
internal
confidential
restricted

y deben respetar las políticas correspondientes.

71. Action Audit

Debe registrarse:

who requested
who authorized
what action
on what subject
with what parameters
when
where
which decision
which policy
which executor
what result

Los parámetros sensibles deben estar protegidos o redactados.

72. Action Observability

Métricas:

actions_requested_total
actions_completed_total
actions_failed_total
actions_cancelled_total
action_latency
action_retry_total
action_timeout_total
action_compensation_total
73. Action Tracing

Una traza debe conectar:

Decision
 ↓
Action
 ↓
Executor
 ↓
External Target
 ↓
Result
74. Action Logging

Logs estructurados deberían incluir:

actionId
actionType
decisionId
tenantId
actor
target
status
correlationId
timestamp

sin incluir secretos.

75. Action Metrics by Type

Permite detectar:

which action types fail most
which targets are slow
which actions require retries
76. Action Reliability

La confiabilidad debe medirse por tipo:

successRate
failureRate
timeoutRate
compensationRate
77. Action SLO

Cada familia puede definir:

availability
latency
completion rate
failure rate
freshness
78. Action Capacity

El sistema debe considerar:

executor capacity
target capacity
rate limits
queue depth
concurrency limits
79. Backpressure

Cuando la capacidad disminuye:

Action Requests
       ↓
Queue
       ↓
Controlled Execution

Debe evitarse saturar targets downstream.

80. Rate Limiting

Las acciones pueden estar limitadas por:

tenant
actor
actionType
target
global capacity
81. Priority

Las Actions pueden tener:

LOW
NORMAL
HIGH
CRITICAL

pero la prioridad nunca debe saltarse restricciones de seguridad o autoridad.

82. Deadline

Una Action puede tener:

deadline

Después de ese momento:

EXPIRED

si no se ejecutó.

83. Scheduling Boundary

Cuando una Action debe ejecutarse posteriormente:

Action
 ↓
Scheduler
 ↓
Execution

Action expresa qué.

Scheduler expresa cuándo.

84. Queue Boundary

Para ejecución asíncrona:

Action
 ↓
Command/Event
 ↓
Queue
 ↓
Worker
85. Transaction Boundary

Si Action modifica varios recursos:

Action
 ↓
Transaction

cuando están dentro del mismo boundary transaccional.

Para sistemas distribuidos debe utilizarse el mecanismo adecuado:

Saga
Outbox
Compensation

según el caso.

86. Outbox Pattern

Cuando una Action modifica estado y debe emitir un evento:

State Change
+
Outbox Event

pueden persistirse atómicamente.

Después:

Outbox
 ↓
Publisher
87. Action Event Reliability

El sistema no debe asumir que:

database update
+
event publish

son atómicos si utilizan infraestructuras independientes.

88. External Side Effects

Las acciones con side effects externos deben tener:

idempotency
timeout
retry policy
audit
result capture
compensation strategy

cuando corresponda.

89. Action Ordering

Algunas acciones requieren:

A before B

La dependencia debe ser explícita.

90. Action Dependency
Action A
   ↓
Action B

B no puede comenzar hasta que A satisfaga su condición de finalización.

91. Parallel Actions

Cuando no existe dependencia:

        Action A
       ↗
Decision
       ↘
        Action B

pueden ejecutarse en paralelo.

La coordinación corresponde a Workflow.

92. Action Atomicity

Una Action debe tener una semántica clara:

completed
or
not completed

cuando el dominio lo permita.

93. Partial Success

Para acciones compuestas:

A → success
B → success
C → failure

el sistema debe expresar:

PARTIAL_SUCCESS

cuando corresponda.

94. Action Recovery

Después de un fallo:

retry
resume
compensate
manual intervention
abandon

La estrategia debe ser explícita.

95. Action Dead Letter

Actions que no pueden procesarse después de retries pueden pasar a:

Dead Letter Queue

con:

reason
attempts
lastError
originalAction
96. Action Replay

Una Action puede ser reintentada desde su representación persistida cuando:

safe
authorized
idempotent

Replay no debe implicar automáticamente repetir side effects peligrosos.

97. Action Versioning

Los contratos pueden evolucionar:

Action V1
Action V2

Los executors deben declarar versiones soportadas.

98. Backward Compatibility

Un executor nuevo debería soportar versiones anteriores cuando sea necesario o existir un adapter.

99. Action Schema

Cada Action Type debe definir:

input schema
output schema
required permissions
constraints
preconditions
postconditions
failure model
100. Action Contract

Contrato conceptual:

ActionContract
├── type
├── version
├── inputSchema
├── outputSchema
├── authority
├── permissions
├── preconditions
├── postconditions
├── retryPolicy
├── timeout
└── failurePolicy
101. Action Registry Governance

No cualquier servicio debería poder registrar arbitrariamente una Action crítica.

Debe existir:

ownership
review
versioning
approval
security classification
102. Action Ownership

Cada Action Type debe tener:

owner
domain
service
support responsibility
103. Action Deprecation

Una Action Type puede pasar por:

ACTIVE
DEPRECATED
DISABLED
REMOVED
104. Action Compatibility Matrix

EVOXA puede mantener:

Action Type
 ×
Executor Version
 ×
Policy Version

para evitar incompatibilidades.

105. Action Testing

Debe cubrir:

schema
authorization
policy
preconditions
execution
retry
timeout
idempotency
concurrency
compensation
audit
security
106. Action Contract Testing

Los consumidores y executors deben verificar:

input compatibility
output compatibility
error compatibility
version compatibility
107. Action Simulation

Puede utilizar:

dry-run
simulation
sandbox
shadow execution

para comprobar qué ocurriría sin producir side effects reales.

108. Dry Run
Action
 ↓
Validate
 ↓
Simulate
 ↓
Return predicted result

No se ejecuta el side effect.

109. Shadow Execution

Puede compararse:

Real Executor
+
Candidate Executor

sin aplicar el resultado candidato.

110. Action Rollback

Debe distinguirse:

rollback execution

de:

compensating action

No todas las acciones soportan rollback.

111. Action Safety

Las Actions críticas deben tener:

explicit authorization
precondition checks
bounded parameters
audit
human approval where required
emergency controls
112. Dangerous Actions

Una clasificación puede identificar:

LOW IMPACT
MEDIUM IMPACT
HIGH IMPACT
CRITICAL

y aplicar controles progresivos.

113. High-Impact Action

Puede requerir:

dual approval
strong authentication
fresh context
explicit confirmation
enhanced audit

según Governance.

114. Action Confirmation

Para ciertas acciones:

Preview
 ↓
Confirm
 ↓
Execute

La confirmación debe registrar:

actor
timestamp
action snapshot
115. Action Preview

Debe poder mostrar:

what will happen
target
parameters
expected impact
risk

antes de ejecutar cuando sea apropiado.

116. Action Explainability

La Action puede proporcionar:

originating decision
reason
expected outcome
side effects
117. Action Provenance

La cadena completa debe ser:

Evidence
 ↓
Intelligence
 ↓
Decision
 ↓
Action
 ↓
Execution
 ↓
Outcome
118. Action Traceability

Debe ser posible navegar desde:

Outcome

hasta:

Decision

y desde:

Decision

hasta:

Action

y finalmente:

Execution
119. Action Outcome Feedback

Después de ejecutar:

ActionResult
       ↓
Outcome
       ↓
Decision Evaluation

Esto permite comparar intención contra realidad.

120. Action Learning

Los resultados pueden utilizarse para:

improve policies
improve decision strategies
improve action routing
improve reliability

siempre mediante los mecanismos de Governance correspondientes.

121. Action Architecture and AI

AI puede generar una Action Proposal:

AI
 ↓
Action Proposal

pero la Action ejecutable debe pasar por:

validation
authorization
policy
execution controls
122. AI Action Guardrail

Nunca debe asumirse:

AI generated
=
authorized

La arquitectura debe mantener:

AI
 ↓
Proposal
 ↓
Policy
 ↓
Authority
 ↓
Action
123. Agentic Action

Para Agents:

Agent
 ↓
Reasoning
 ↓
Decision
 ↓
Action
 ↓
Tool
 ↓
Result
 ↓
Agent

Cada Action debe quedar registrada.

124. Agent Action Limits

Puede definirse:

allowed actions
max frequency
max value
allowed targets
approval thresholds
125. Action Policy Evaluation

Antes de ejecutar:

Action
 ↓
Policy Engine
 ↓
ALLOW
DENY
REVIEW
126. Policy Outcomes
ALLOW
→ execute

DENY
→ reject

REVIEW
→ human / approval workflow
127. Action Approval

Cuando una Action supera un threshold:

Action
 ↓
Approval Required
 ↓
Human / Authority
 ↓
Execution
128. Action Revocation

Una Action preparada pero no ejecutada puede revocarse.

READY
 ↓
REVOKED
129. Action Supersession

Una nueva Decision puede invalidar una Action pendiente:

Action A
   ↓
Decision B
   ↓
Action A superseded
130. Action Conflict

Dos Actions pueden competir por el mismo recurso:

Action A
   ↘
    Resource
   ↗
Action B

Debe existir una estrategia de:

serialization
priority
rejection
coordination
131. Resource Locking

Cuando sea necesario:

Action
 ↓
Acquire Resource
 ↓
Execute
 ↓
Release Resource

El locking debe ser corto y controlado.

132. Action Scheduling

Una Action puede contener:

executeAfter
deadline

pero la responsabilidad de scheduling pertenece a E16.

133. Action Queueing

Las acciones pueden almacenarse en:

priority queue
work queue
tenant queue
target queue

según necesidades operativas.

134. Multi-Tenant Action Execution

Cada Action debe conservar:

tenantId

y el executor debe respetar aislamiento.

135. Cross-Tenant Actions

Solo pueden existir cuando:

explicit policy
explicit authority
explicit audit

lo permitan.

136. Action API

Conceptualmente:

POST /actions
GET /actions/{id}
POST /actions/{id}/validate
POST /actions/{id}/authorize
POST /actions/{id}/execute
POST /actions/{id}/cancel
POST /actions/{id}/retry
POST /actions/{id}/compensate
GET /actions/{id}/result
GET /actions/{id}/history

Los contratos definitivos pertenecen a E03.

137. Action Application Service

Responsabilidades:

receive request
resolve action
validate
authorize
evaluate policy
prepare
dispatch
capture result
publish events
138. Action Domain Services

Ejemplos:

ActionResolver
ActionValidator
ActionAuthorizer
ActionPolicyEvaluator
ActionDispatcher
ActionRetryManager
ActionCompensationService
ActionResultProcessor
139. Action Repository

Debe persistir:

action
state
parameters
authority
decision reference
execution references
attempts
results
history
140. Action State Machine

Una implementación conceptual:

REQUESTED
    │
    ▼
RESOLVED
    │
    ▼
VALIDATED
    │
    ▼
AUTHORIZED
    │
    ▼
READY
    │
    ▼
DISPATCHED
    │
    ▼
EXECUTING
    │
 ┌──┴─────┐
 ▼        ▼
SUCCESS  FAILURE
 │        │
 ▼        ▼
COMPLETED RETRY
           │
        ┌──┴──┐
        ▼     ▼
     EXECUTE  FAILED
141. Action Failure State Machine
FAILED
  │
  ├── retryable → RETRY
  │
  ├── compensatable → COMPENSATION
  │
  ├── recoverable → MANUAL_REVIEW
  │
  └── permanent → ABANDONED
142. Action Reliability Patterns

Puede utilizar:

Timeout
Retry
Circuit Breaker
Bulkhead
Rate Limit
Idempotency
Outbox
Dead Letter
Compensation

según el tipo de integración.

143. Circuit Breaker

Si un target falla repetidamente:

CLOSED
 ↓
OPEN
 ↓
HALF_OPEN

evitando saturar el sistema downstream.

144. Bulkhead

Separar capacidad por:

tenant
action type
target
criticality

puede evitar que una clase de acciones degrade todo el runtime.

145. Action Priority Isolation

Las Actions críticas pueden tener capacidad reservada.

146. Action Cost Control

Puede limitarse:

expensive external calls
AI tool calls
high-volume actions

para proteger recursos.

147. Action Security Boundary

La ejecución debe considerarse un security boundary porque aquí aparecen side effects reales.

Por ello:

Decision
→ logical commitment

Action
→ authorized intent

Execution
→ real-world side effect
148. Action Governance

Governance debe definir:

which actions exist
who can invoke them
who can execute them
which require approval
which require audit
which are prohibited
149. Action Catalog

El catálogo debe permitir conocer:

Action Type
Owner
Risk
Permissions
Executor
Policy
Version
SLO
150. Definition of Done

E34 queda definido cuando EVOXA dispone de:

✓ Action boundary
✓ Action definition
✓ Action types
✓ Action intent
✓ Parameters
✓ Subject
✓ Target
✓ Actor
✓ Authority
✓ Authority inheritance
✓ Authorization
✓ Preconditions
✓ Postconditions
✓ Lifecycle
✓ Action request
✓ Action resolution
✓ Action planning
✓ Atomic actions
✓ Composite actions
✓ Workflow boundary
✓ Commands
✓ Executors
✓ Handlers
✓ Adapters
✓ Gateways
✓ Validation
✓ Policy integration
✓ Constraints
✓ Limits
✓ Expiration
✓ Freshness
✓ Pre-execution validation
✓ Idempotency
✓ Retry
✓ Timeout
✓ Cancellation
✓ Failure classification
✓ Compensation
✓ Saga boundary
✓ Action result
✓ Action outcome
✓ Action events
✓ Correlation
✓ Causation
✓ Execution boundary
✓ Executor independence
✓ Routing
✓ Registry
✓ Capability discovery
✓ Agent integration
✓ Human tasks
✓ Security
✓ Least privilege
✓ Credential isolation
✓ Tenant isolation
✓ Audit
✓ Observability
✓ Tracing
✓ Reliability
✓ Capacity
✓ Backpressure
✓ Rate limiting
✓ Priority
✓ Deadline
✓ Scheduling boundary
✓ Queue boundary
✓ Transaction boundary
✓ Outbox
✓ External side effects
✓ Ordering
✓ Dependencies
✓ Parallel execution
✓ Partial success
✓ Recovery
✓ Dead letter
✓ Replay
✓ Versioning
✓ Contract testing
✓ Simulation
✓ Dry run
✓ Rollback/compensation
✓ Safety controls
✓ High-impact controls
✓ Confirmation
✓ Explainability
✓ Provenance
✓ AI guardrails
✓ Agent action limits
✓ Policy evaluation
✓ Approval
✓ Revocation
✓ Supersession
✓ Conflict handling
✓ Resource locking
✓ Multi-tenant execution
✓ API boundary
✓ Application services
✓ Domain services
✓ Repository
✓ State machine
✓ Reliability patterns
✓ Circuit breaker
✓ Bulkhead
✓ Governance
✓ Action catalog
151. Position in Engineering Specification

La cadena queda ahora:

E29 — Search
        ↓
E30 — Reporting
        ↓
E31 — Analytics
        ↓
E32 — Intelligence
        ↓
E33 — Decision
        ↓
E34 — Action
        ↓
E35 — Execution

Y la frontera completa:

┌───────────────────────────────────────────────────────┐
│                  EVOXA INTELLIGENCE LOOP              │
│                                                       │
│  Data → Search → Reporting → Analytics → Intelligence│
│                                              │        │
│                                              ▼        │
│                                          Decision     │
│                                              │        │
│                                              ▼        │
│                                           Action      │
│                                              │        │
│                                              ▼        │
│                                          Execution    │
│                                              │        │
│                                              ▼        │
│                                           Outcome     │
│                                              │        │
│                                              ▼        │
│                                           Learning    │
│                                              │        │
│                                              └─────────┤
│                                                        │
└────────────────────────────────────────────────────────┘

La regla arquitectónica que queda establecida es:

E32 — Intelligence comprende. E33 — Decision elige. E34 — Action formaliza la intención operacional. E35 — Execution realizará el efecto.

Siguiente: E35 — EVOXA Execution Architecture.
