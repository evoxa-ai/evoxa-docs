E14 — EVOXA Workflow & Orchestration Architecture
Engineering Specification

Pertenece a: Engineering Specification
Volumen: volume-09-engineering
Capítulo: E14 — Workflow & Orchestration Architecture
Anterior: E13 — Event Processing Architecture
Siguiente: E15 — EVOXA Job & Task Processing Architecture

1. Propósito

E14 define la arquitectura mediante la cual EVOXA coordina procesos de negocio compuestos por múltiples pasos, servicios, eventos y decisiones.

E13 establece cómo EVOXA procesa eventos.

E14 establece cómo EVOXA coordina procesos que requieren:

Multiple Steps
State
Conditions
Branches
Retries
Timeouts
Compensation
Human Intervention
External Integrations
Long-Running Execution

La pregunta central es:

¿Cómo coordina EVOXA procesos complejos sin convertir los servicios individuales en un sistema fuertemente acoplado?

2. Objetivos

La arquitectura de Workflow & Orchestration debe proporcionar:

Workflow Definition
Workflow Execution
State Management
Step Execution
Branching
Conditions
Parallel Execution
Sequential Execution
Retries
Timeouts
Compensation
Suspension
Resume
Cancellation
Human Tasks
External Tasks
Event Waiting
Workflow Versioning
Observability
Recovery
Idempotency
Multi-Tenant Execution
3. Principio Fundamental

Un Workflow representa:

Una secuencia controlada de actividades necesarias para alcanzar un resultado empresarial.

Ejemplo:

Subscription Creation
        │
        ▼
Validate Customer
        │
        ▼
Authorize Payment
        │
        ▼
Create Subscription
        │
        ▼
Provision Entitlements
        │
        ▼
Send Confirmation
4. Workflow vs Event

No deben confundirse.

Event

Describe:

Something happened

Ejemplo:

PaymentCaptured
Workflow

Describe:

What must happen next

Ejemplo:

ActivateSubscriptionWorkflow
5. Workflow vs Application Service

Un Application Service ejecuta una operación.

Un Workflow coordina múltiples operaciones y estados a través del tiempo.

Application Service
       │
       ▼
One Business Operation

mientras:

Workflow
   │
   ├── Service A
   ├── Service B
   ├── Event
   ├── External API
   ├── Wait
   └── Service C
6. Orchestration

Orchestration significa que existe un componente coordinador:

              Workflow
                  │
       ┌──────────┼──────────┐
       ▼          ▼          ▼
   Service A  Service B  Service C

El Workflow conoce el orden y las condiciones.

7. Choreography

En Choreography los servicios reaccionan a eventos:

Service A
   │
   ▼
Event
   │
   ▼
Service B
   │
   ▼
Event
   │
   ▼
Service C

Ningún componente central coordina necesariamente todo el proceso.

8. Orchestration vs Choreography

EVOXA debe permitir ambos modelos.

Orchestration

Adecuado cuando:

Complex Business Process
Long Running Process
Compensation
Explicit State
Human Approval
External Dependencies
Choreography

Adecuado cuando:

Loose Coupling
Simple Reactions
Independent Consumers
Event Notifications
9. Workflow Boundary
┌─────────────────────────────────┐
│ EVOXA Application               │
│                                 │
│ Workflow Definition             │
└────────────────┬────────────────┘
                 │
═════════════════╪═════════════════
     Workflow Boundary
═════════════════╪═════════════════
                 │
                 ▼
        Workflow Runtime
                 │
       ┌─────────┼─────────┐
       ▼         ▼         ▼
    Services   Events   Tasks
10. Workflow Definition

Un Workflow debe describir:

Workflow ID
Version
Steps
Transitions
Conditions
Timeouts
Retries
Compensation
Inputs
Outputs
Policies
11. Workflow Instance

Una definición es reutilizable.

Una ejecución concreta es una:

Workflow Instance

Ejemplo:

Workflow Definition
ActivateSubscriptionWorkflow.v1
             │
      ┌──────┼──────┐
      ▼      ▼      ▼
 Instance  Instance Instance
   #001      #002     #003
12. Workflow Instance State

Una instancia puede estar:

CREATED
RUNNING
WAITING
PAUSED
COMPLETED
FAILED
CANCELLED
COMPENSATING
COMPENSATED
13. Workflow Lifecycle
Created
  ↓
Started
  ↓
Running
  ↓
Waiting
  ↓
Resumed
  ↓
Completed

Con errores:

Running
  ↓
Failed
  ↓
Retry
  ↓
Compensation
  ↓
Compensated
14. Workflow Context

Cada instancia debe disponer de contexto:

workflow_id
workflow_version
instance_id
tenant_id
correlation_id
started_at
updated_at
current_step
status
input
state
15. Tenant Context

Todos los workflows tenant-scoped deben conservar:

tenant_id

durante toda la ejecución.

Workflow
   ↓
Tenant Context
   ↓
Every Step
16. Workflow Input

Ejemplo:

ActivateSubscriptionWorkflow

Input:

customer_id
subscription_id
plan_id
payment_method
17. Workflow State

Durante la ejecución:

Workflow State
├── customer
├── payment
├── subscription
├── provisioning
└── notifications

El estado debe mantenerse de forma durable cuando el workflow pueda sobrevivir a reinicios.

18. Workflow Output

Una ejecución completada puede producir:

subscription_id
activation_status
entitlements

El output debe estar versionado cuando forme parte de un contrato externo.

19. Workflow Steps

Los pasos representan unidades ejecutables:

Step A
  ↓
Step B
  ↓
Step C

Tipos posibles:

Service Task
Event Task
Wait Task
Human Task
Decision Task
Integration Task
Compensation Task
20. Service Task

Ejecuta una operación interna:

Workflow
   ↓
Application Service

Ejemplo:

CreateSubscription
21. Integration Task

Ejecuta una operación externa:

Workflow
   ↓
Integration Service
   ↓
External Provider

Ejemplo:

AuthorizePayment
22. Event Task

Publica o espera un evento.

Workflow
   ↓
Publish Event

o:

Workflow
   ↓
Wait for Event
23. Wait Task

Un workflow puede detenerse temporalmente:

Step A
  ↓
WAIT
  ↓
Event / Time
  ↓
Step B

Esto es fundamental para workflows de larga duración.

24. Human Task

Algunos procesos requieren intervención humana.

Workflow
   ↓
Human Approval
   ↓
Approved
   ↓
Continue

Ejemplos:

Manual Review
Refund Approval
Coach Approval
Account Verification
Compliance Review
25. Decision Task

Permite branching:

                 Decision
                    │
          ┌─────────┼─────────┐
          ▼         ▼         ▼
        Path A    Path B    Path C
26. Conditional Branching

Ejemplo:

Payment Authorized?
       │
   ┌───┴───┐
   ▼       ▼
  YES      NO
   │       │
   ▼       ▼
Continue  Retry
27. Parallel Execution

Un workflow puede ejecutar pasos simultáneamente:

             Start
               │
        ┌──────┼──────┐
        ▼      ▼      ▼
      Task A Task B Task C
        │      │      │
        └──────┼──────┘
               ▼
             Join
28. Sequential Execution

La ejecución secuencial:

A
↓
B
↓
C
↓
D

debe utilizarse cuando exista dependencia entre pasos.

29. Parallel Join

Cuando varios caminos terminan:

A ──┐
B ──┼──> Join
C ──┘

el Workflow Runtime debe conocer:

All Completed
Any Completed
N of M Completed

según la definición.

30. Workflow Conditions

Las condiciones deben estar expresadas mediante políticas controladas.

Ejemplo:

if payment.status == AUTHORIZED

No debería permitirse ejecutar código arbitrario dentro de una definición de workflow.

31. Policy Integration

Los workflows deben respetar:

Authorization
Tenant Policies
Business Rules
Security Policies
Compliance Policies
32. Workflow State Machine

Conceptualmente:

CREATED
   ↓
RUNNING
   ↓
WAITING
   ↓
RUNNING
   ↓
COMPLETED

Errores:

RUNNING
   ↓
FAILED
   ↓
RETRYING
   ↓
RUNNING
33. Workflow Persistence

El estado del workflow debe persistirse cuando sea necesario.

Workflow Runtime
       │
       ▼
Workflow State Store

Debe soportar recuperación después de:

Process Restart
Worker Failure
Server Failure
Deployment
Network Failure
34. Durable Execution

Los workflows long-running deben ser durables.

Ejemplo:

Day 1
Create Application
      ↓
WAIT

Day 3
Approval Received
      ↓
Continue

El proceso no debe depender de mantener un proceso en memoria durante tres días.

35. Workflow Checkpoints

El runtime puede crear checkpoints:

Step A completed
       ↓
Checkpoint
       ↓
Step B

Si ocurre un fallo:

Restart
   ↓
Load Checkpoint
   ↓
Continue
36. Retry

Cada step puede tener:

max_attempts
backoff
retryable_errors

Ejemplo:

Payment API
   ↓
Timeout
   ↓
Retry
37. Retry Policy

No todos los errores deben reintentarse.

Timeout
503
Network Failure

pueden ser retryable.

Mientras:

Invalid Card
Unauthorized
Business Rule Violation

pueden ser permanent failures.

38. Timeout

Cada step puede tener timeout:

Step A
 └── timeout = 30s

Un workflow completo puede tener:

Workflow Timeout

además de los timeouts individuales.

39. Workflow Cancellation

Una instancia puede cancelarse:

RUNNING
   ↓
CANCEL_REQUESTED
   ↓
CANCELLED

Debe definirse qué sucede con los pasos ya ejecutados.

40. Workflow Pause

Puede pausarse:

RUNNING
   ↓
PAUSED

y posteriormente:

PAUSED
   ↓
RESUMED
41. Compensation

Cuando no existe rollback técnico, debe utilizarse compensación empresarial.

Ejemplo:

Authorize Payment
       ↓
Provision Account
       ↓
Provision Failed

Compensación:

Refund Payment
42. Saga Pattern

Los workflows distribuidos pueden utilizar Saga:

Step A
 ↓
Step B
 ↓
Step C

con:

Compensation C
Compensation B
Compensation A

cuando corresponda.

43. Compensation Order

Normalmente:

A
 ↓
B
 ↓
C

si C falla:

Compensate B
      ↓
Compensate A

Debe definirse explícitamente por workflow.

44. Compensation State

El workflow puede entrar en:

COMPENSATING

y posteriormente:

COMPENSATED

o:

COMPENSATION_FAILED
45. Compensation Failure

Si la compensación falla:

Business Operation
       ↓
Failure
       ↓
Compensation
       ↓
Compensation Failure

debe existir:

Alert
Manual Recovery
Audit
46. Workflow Recovery

Después de un crash:

Runtime Failure
      ↓
Restart
      ↓
Load Workflow State
      ↓
Resume
47. Workflow Idempotency

Cada step que pueda ejecutarse nuevamente debe ser idempotente.

Step
 ↓
Failure
 ↓
Retry

no debe producir efectos duplicados.

48. Workflow Correlation

Todas las operaciones de una instancia deben compartir:

workflow_instance_id
correlation_id

Esto permite reconstruir la ejecución completa.

49. Workflow Causation

Cuando un evento inicia o avanza un workflow:

event_id

puede almacenarse como:

causation_id
50. Workflow Events

El runtime puede producir:

WorkflowStarted
WorkflowStepStarted
WorkflowStepCompleted
WorkflowWaiting
WorkflowFailed
WorkflowCompleted
WorkflowCancelled
WorkflowCompensating
WorkflowCompensated
51. Workflow Audit

Debe poder responderse:

Who started it?
When?
Which version?
Which steps executed?
Which failed?
How many retries?
What external systems were called?
What was the final result?
52. Workflow History

Una instancia debe mantener historial suficiente:

Step
Status
Started At
Completed At
Attempts
Error
Result
53. Workflow Observability

Métricas:

workflow_started_total
workflow_completed_total
workflow_failed_total
workflow_cancelled_total
workflow_duration
workflow_step_duration
workflow_retry_total
workflow_compensation_total
54. Workflow Tracing

Debe existir:

workflow_instance_id
trace_id
correlation_id
step_id

para conectar:

API
 ↓
Workflow
 ↓
Service
 ↓
Database
 ↓
External API
 ↓
Event
55. Workflow Logging

Logs estructurados:

workflow_id
workflow_version
instance_id
step_id
tenant_id
status
duration
error

No deben contener secretos ni datos sensibles innecesarios.

56. Workflow Definition Versioning

Una instancia debe quedar asociada a una versión:

ActivateSubscriptionWorkflow.v1

Una nueva versión:

ActivateSubscriptionWorkflow.v2

no debería alterar inesperadamente las instancias existentes.

57. Version Migration

Los workflows long-running requieren una estrategia para:

Old Instance
       ↓
Old Version

mientras nuevas instancias utilizan:

New Version
58. Workflow Deployment

Un nuevo workflow puede pasar por:

Draft
 ↓
Validated
 ↓
Test
 ↓
Active
 ↓
Deprecated
 ↓
Retired
59. Workflow Validation

Antes de activar un workflow:

Schema Validation
Step Validation
Transition Validation
Policy Validation
Dependency Validation
60. Circular Workflow Detection

Debe detectarse:

A → B → C → A

cuando no esté explícitamente permitido.

Los loops controlados sí pueden existir, pero deben tener:

Exit Condition
Maximum Iterations
Timeout
61. Workflow Loop

Ejemplo:

Process Item
    ↓
More Items?
 ┌──┴──┐
YES    NO
 │      │
 └─→ Process
        ↓
      Finish
62. Workflow Variables

Las variables pueden representar:

Input
Intermediate State
Step Output
Decision Result

Deben tener tipos y límites definidos.

63. Workflow Data Minimization

No almacenar todo el payload de cada paso indefinidamente.

Debe distinguirse:

Operational State
Audit Data
Sensitive Data
Temporary Data
64. Workflow Secrets

Nunca almacenar directamente:

API Keys
Passwords
Tokens

en workflow state.

Debe utilizarse una referencia segura:

secret_reference
65. External Waiting

Un workflow puede esperar una respuesta externa:

Start
 ↓
Send Request
 ↓
WAIT
 ↓
Webhook
 ↓
Continue
66. Timer Events

Puede esperar hasta:

Specific Date
Delay
Deadline
Timeout
Scheduled Time

Ejemplo:

Subscription Created
       ↓
WAIT 7 days
       ↓
Send Reminder
67. Deadline Handling

Un workflow puede tener:

Deadline

y una acción:

Deadline reached
       ↓
Escalate
Cancel
Notify
Compensate
68. Human Approval Workflow

Ejemplo:

Application
    ↓
Risk Evaluation
    ↓
Risk High?
    ↓
Human Review
    ↓
Approved?
 ┌──┴──┐
YES    NO
 │      │
Continue Reject
69. Human Task State
CREATED
ASSIGNED
IN_PROGRESS
APPROVED
REJECTED
EXPIRED
CANCELLED
70. Workflow Assignment

Las tareas humanas pueden asignarse por:

User
Role
Team
Organization
Queue
Capability
71. Workflow Escalation

Si una tarea humana no se completa:

Task
 ↓
Deadline
 ↓
Escalation
 ↓
Supervisor
72. Workflow Notifications

Los workflows pueden producir:

NotificationRequested

en lugar de enviar directamente:

Email
SMS
Push

Esto mantiene separación con E11/E12.

73. Workflow and Messaging
Workflow
   │
   ├── Publish Command
   │
   ├── Wait for Event
   │
   └── Publish Event

Por tanto:

E12
→ Transport

E13
→ Event Processing

E14
→ Process Coordination
74. Workflow and Integration
Workflow
   ↓
Integration Interface
   ↓
Adapter
   ↓
External Provider

Un workflow no debe conocer directamente SDKs externos.

75. Workflow and Repository

El Workflow Runtime necesita persistir:

Workflow State
Workflow History
Step State
Timers
Pending Tasks

Puede utilizar componentes definidos por E10.

76. Workflow and Domain

El workflow coordina.

El dominio decide las reglas de negocio.

Workflow
   ↓
Application Service
   ↓
Domain

No:

Workflow
   ↓
Direct Database Manipulation
77. Workflow Engine

EVOXA puede utilizar un:

Workflow Engine

responsable de:

Execution
Persistence
Timers
Retries
State
Scheduling
Recovery

La implementación concreta queda fuera de este capítulo.

78. Workflow Runtime

El runtime ejecuta:

Workflow Instance

y administra:

Current State
Current Step
Transitions
Timers
Failures
Recovery
79. Workflow Scheduler

Los workflows pueden requerir:

Delayed Execution
Timers
Deadlines
Scheduled Steps

El scheduler debe integrarse con el runtime.

80. Workflow Worker

Los steps pueden ser ejecutados por workers:

Workflow Runtime
       ↓
Task Queue
       ↓
Worker
       ↓
Application Service
81. Workflow Scalability

Las instancias deben poder distribuirse:

Workflow Runtime
 ├── Node A
 ├── Node B
 └── Node C

sin perder consistencia.

82. Workflow Concurrency

Dos workers no deben ejecutar accidentalmente el mismo step simultáneamente.

Debe utilizarse:

Lease
Lock
Optimistic Concurrency
Atomic State Transition

según la infraestructura.

83. Workflow State Transition

Las transiciones deben ser atómicas:

RUNNING
   ↓
WAITING

No debe quedar un estado intermedio inconsistente.

84. Workflow Backpressure

Si existen demasiadas instancias:

Workflow Requests
████████████████
        ↓
Workers
████

debe aplicarse:

Queueing
Concurrency Limits
Priorities
Autoscaling
85. Workflow Priority

Las instancias pueden clasificarse:

CRITICAL
HIGH
NORMAL
LOW

Ejemplo:

Payment Recovery
→ HIGH

Analytics Rebuild
→ LOW
86. Workflow Rate Limits

Puede limitarse por:

Tenant
Workflow Type
Provider
Resource
Global System

Esto protege tanto EVOXA como sistemas externos.

87. Tenant Fairness

Un tenant con alto volumen no debería consumir todos los workers.

Tenant A → 80%
Tenant B → 10%
Tenant C → 10%

La arquitectura debe permitir cuotas y límites cuando sea necesario.

88. Workflow Security

Debe protegerse:

Workflow Definition
Workflow State
Human Tasks
Execution Commands
External Credentials
Sensitive Data
89. Workflow Authorization

No todos los usuarios pueden:

Start
Cancel
Pause
Resume
Retry
Replay
Inspect

un workflow.

Estas operaciones deben estar protegidas por políticas.

90. Workflow Administrative Operations

Las operaciones administrativas pueden incluir:

Start
Pause
Resume
Cancel
Retry
Skip
Compensate
Replay
Terminate

Algunas deben estar restringidas a operadores autorizados.

91. Workflow Failure Modes

Deben contemplarse:

Step Failure
Worker Failure
Database Failure
Broker Failure
External API Failure
Timeout
Invalid Input
Policy Rejection
Compensation Failure
Version Conflict
92. Workflow Recovery Matrix

Ejemplo conceptual:

Error	Estrategia
Timeout	Retry
External 503	Retry
Invalid Input	Fail
Business Rejection	Compensate / Fail
Worker Crash	Resume
Broker Failure	Retry
Compensation Failure	Manual Recovery
93. Workflow Replay

Debe diferenciarse:

Replay Event

de:

Replay Workflow

Un workflow replay debe evitar efectos externos duplicados.

94. Workflow Simulation

Puede existir:

Simulation Mode

para validar una definición antes de producción.

95. Workflow Testing

Debe probarse:

Happy Path
Failure Path
Retry
Timeout
Cancellation
Compensation
Parallel Execution
Human Approval
External Failure
Recovery
Version Migration
96. Workflow Contract Testing

Las dependencias de un workflow deben validarse:

Application Services
Events
Commands
Integrations
Human Tasks
Policies
97. Workflow Governance

Cada workflow debe tener:

Owner
Business Purpose
Version
Criticality
SLA
Dependencies
Failure Strategy
Security Classification
Data Classification
98. Workflow Registry

EVOXA debe disponer conceptualmente de un:

Workflow Registry

con:

workflow_id
name
version
status
owner
criticality
tenant_scope
definition
created_at
updated_at
99. Workflow Lifecycle
Draft
  ↓
Validated
  ↓
Tested
  ↓
Published
  ↓
Active
  ↓
Deprecated
  ↓
Retired

Las instancias existentes deben gestionarse independientemente de este lifecycle.

100. Workflow Architecture

La arquitectura consolidada:

                         EVOXA
                           │
                           ▼
                    Workflow Trigger
                           │
                           ▼
                  Workflow Definition
                           │
                           ▼
                   Workflow Runtime
                           │
              ┌────────────┼────────────┐
              ▼            ▼            ▼
           Service       Decision      Event
             Task          Task         Task
              │             │            │
              ▼             ▼            ▼
        Application      Policy       Message
         Service         Engine        Bus
              │
              ▼
            Domain
              │
       ┌──────┴──────┐
       ▼             ▼
 Repository      Integration
       │             │
       ▼             ▼
   Database     External World
101. Long-Running Workflow

Ejemplo EVOXA:

User Starts Program
        │
        ▼
Create Training Plan
        │
        ▼
Assign Coach
        │
        ▼
WAIT
        │
        ▼
Workout Completed
        │
        ▼
Evaluate Progress
        │
        ▼
Goal Achieved?
     ┌──┴──┐
    YES    NO
     │      │
     ▼      ▼
Reward   Adjust Plan
             │
             ▼
           WAIT

Este tipo de proceso demuestra por qué EVOXA necesita una arquitectura de workflows independiente del simple procesamiento de eventos.

102. Subscription Workflow Example
Create Subscription
        │
        ▼
Validate Plan
        │
        ▼
Authorize Payment
        │
        ▼
Payment Success?
      ┌─┴─┐
     YES  NO
      │    │
      ▼    ▼
Provision Retry / Fail
      │
      ▼
Activate Subscription
      │
      ▼
Publish Event
      │
      ▼
Send Notification

Si provisioning falla después del pago:

Payment Captured
       ↓
Provision Failed
       ↓
Compensation
       ↓
Refund
103. Relationship E11–E14

La arquitectura queda:

E11 — Integration
        │
        ▼
External Capabilities
        │
        ▼
E12 — Messaging
        │
        ▼
Message Transport
        │
        ▼
E13 — Event Processing
        │
        ▼
Event Execution
        │
        ▼
E14 — Workflow & Orchestration
        │
        ▼
Business Process Coordination
104. Responsibility Boundaries

La separación fundamental:

E10
→ Persistence

E11
→ External Integration

E12
→ Message Transport

E13
→ Event Processing

E14
→ Workflow Coordination

Esto evita que un único componente termine manejando:

Database
HTTP
Events
Retries
Business Process
Scheduling

al mismo tiempo.

105. Definition of Done

E14 queda definido cuando EVOXA dispone de:

✓ Workflow Definition
✓ Workflow Instance
✓ Workflow State
✓ Workflow Context
✓ Workflow Inputs
✓ Workflow Outputs
✓ Workflow Steps
✓ Service Tasks
✓ Integration Tasks
✓ Event Tasks
✓ Wait Tasks
✓ Human Tasks
✓ Decision Tasks
✓ Sequential Execution
✓ Parallel Execution
✓ Branching
✓ Conditions
✓ Workflow State Machine
✓ Durable Execution
✓ Checkpoints
✓ Retry
✓ Timeout
✓ Cancellation
✓ Pause / Resume
✓ Compensation
✓ Saga Pattern
✓ Compensation Failure Handling
✓ Recovery
✓ Idempotency
✓ Correlation
✓ Causation
✓ Workflow Events
✓ Workflow Audit
✓ Workflow History
✓ Observability
✓ Distributed Tracing
✓ Versioning
✓ Version Migration
✓ Definition Validation
✓ Circular Dependency Detection
✓ Loop Protection
✓ Workflow Variables
✓ Data Minimization
✓ Secret Protection
✓ External Waiting
✓ Timers
✓ Deadlines
✓ Human Approval
✓ Escalation
✓ Messaging Integration
✓ Integration Architecture Integration
✓ Repository Integration
✓ Domain Integration
✓ Workflow Runtime
✓ Workflow Scheduler
✓ Workflow Workers
✓ Horizontal Scaling
✓ Concurrency Control
✓ Backpressure
✓ Priority
✓ Rate Limiting
✓ Tenant Fairness
✓ Security
✓ Authorization
✓ Failure Handling
✓ Recovery Matrix
✓ Replay
✓ Simulation
✓ Testing
✓ Contract Testing
✓ Governance
✓ Workflow Registry
✓ Workflow Lifecycle
106. Engineering Specification Progress

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

Y la arquitectura empieza a formar una cadena especialmente importante:

                         EVOXA
                           │
                           ▼
                    Application Layer
                           │
          ┌────────────────┼────────────────┐
          ▼                ▼                ▼
      Repository      Integration       Workflow
          │                │                │
          ▼                ▼                ▼
      Database       External World    Orchestration
                           │                │
                           └───────┬────────┘
                                   ▼
                              Messaging
                                   │
                                   ▼
                           Event Processing
                                   │
                                   ▼
                                Domain

E14 establece la capa de coordinación de procesos de negocio de EVOXA. Con esto, EVOXA ya posee una separación clara entre persistencia, integraciones, mensajería, procesamiento de eventos y workflows de larga duración.

Siguiente capítulo: E15 — EVOXA Job & Task Processing Architecture.
