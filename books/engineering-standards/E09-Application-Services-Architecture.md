E09 — EVOXA Application Services Architecture
Engineering Specification

Pertenece a: Engineering Specification
Volumen: volume-09-engineering
Capítulo: E09 — Application Services Architecture
Anterior: E08 — Domain Services Architecture
Siguiente: E10 — Repository Architecture

1. Propósito

E09 define la arquitectura de los Application Services de EVOXA.

Mientras:

E07 — Service Architecture define la arquitectura general de servicios.
E08 — Domain Services Architecture define dónde reside la lógica de negocio del dominio.

E09 define la capa responsable de coordinar los casos de uso de EVOXA.

La pregunta central es:

¿Cómo ejecuta EVOXA un caso de uso completo, coordinando autenticación, autorización, políticas, dominio, persistencia, eventos e integraciones sin colocar esa lógica en los controllers o en el dominio?

2. Objetivos

Application Services debe proporcionar:

Use Case Orchestration
Transaction Coordination
Authorization Coordination
Policy Coordination
Domain Service Invocation
Repository Coordination
Event Coordination
Integration Coordination
Idempotency
Error Translation
Execution Context
3. Principio Fundamental

Un Application Service representa una acción que el sistema puede ejecutar.

User Intent
     ↓
Application Service
     ↓
Domain
     ↓
Result

Ejemplos:

CreateWorkout
CompleteWorkout
GenerateTrainingPlan
UpdateNutritionPlan
RecordProgress
CreateSubscription
4. Application Service vs Domain Service

La distinción es fundamental:

Application Service
→ "¿Qué debe hacer el sistema?"

Domain Service
→ "¿Cómo se determina la decisión de negocio?"

Ejemplo:

CompleteWorkoutApplicationService
             │
             ├── Authenticate Context
             ├── Authorize
             ├── Load Workout
             ├── Invoke Domain Service
             ├── Persist Result
             └── Publish Event

Mientras:

WorkoutCompletionDomainService
             │
             ├── Validate State
             ├── Calculate Load
             ├── Apply Rules
             └── Produce Decision
5. Application Service Responsibilities

Un Application Service puede:

Receive Command
Validate Application Input
Load Domain Objects
Check Authorization
Evaluate Policies
Invoke Domain Services
Persist Changes
Publish Events
Call Integrations
Return Result

No debe convertirse en el lugar donde viven todas las reglas de negocio.

6. Application Service Structure

Arquitectura:

                    API
                     │
                     ▼
              Application Service
                     │
       ┌─────────────┼─────────────┐
       ▼             ▼             ▼
 Authorization     Policy       Domain
       │             │             │
       └─────────────┼─────────────┘
                     ▼
                Persistence
                     │
                     ▼
                  Events
7. Use Case Orientation

Los Application Services deben organizarse alrededor de casos de uso, no solamente de tablas.

Preferible:

CompleteWorkout
GenerateNutritionPlan
ChangeSubscription

en lugar de:

WorkoutCRUDService
NutritionCRUDService
SubscriptionCRUDService

cuando el comportamiento requiere una operación empresarial real.

8. Use Case

Un caso de uso representa:

Actor
+
Intent
+
Business Context
+
Expected Result

Ejemplo:

Actor:
Trainer

Intent:
Create training plan

Result:
Training plan created and activated
9. Application Service Input

Los inputs deben representar la intención de la operación.

Ejemplo:

CreateWorkoutCommand

con:

trainingPlanId
date
exercises

No debería recibir directamente:

HttpRequest
10. Commands

Los Commands representan una intención:

CreateWorkout
CompleteWorkout
CancelWorkout
GenerateTrainingPlan
UpdateNutritionPlan

Características:

Explicit
Immutable
Validated
Traceable
11. Queries

Las Queries representan consultas sin intención de modificar el estado.

Ejemplos:

GetWorkout
GetTrainingPlan
GetProgress
GetNutritionSummary

Arquitectura:

Command
→ Changes State

Query
→ Reads State
12. Command/Query Separation

EVOXA puede utilizar un enfoque conceptual CQRS:

             Application Layer
                    │
             ┌──────┴──────┐
             ▼             ▼
          Commands       Queries
             │             │
             ▼             ▼
         Write Path     Read Path

No implica necesariamente implementar dos bases de datos desde el comienzo.

13. Application Service Lifecycle

Cada ejecución puede seguir:

Receive
  ↓
Validate
  ↓
Authorize
  ↓
Policy Check
  ↓
Load
  ↓
Execute
  ↓
Persist
  ↓
Publish
  ↓
Return
14. Execution Context

Cada Application Service debe ejecutarse dentro de un contexto:

ApplicationContext

que puede contener:

requestId
traceId
tenantId
subjectId
serviceIdentity
authorizationContext
correlationId
15. Tenant Context

En EVOXA multi-tenant:

Request
   ↓
Tenant Resolution
   ↓
Application Service
   ↓
Tenant-aware Domain

El tenantId no debe confiarse ciegamente desde el payload del usuario.

16. Authentication Context

La autenticación debe resolverse antes del caso de uso.

Authentication
      ↓
Identity
      ↓
Application Context
      ↓
Use Case

El Application Service no debería implementar el mecanismo de login.

17. Authorization

El Application Service debe garantizar que el actor pueda ejecutar la operación.

Actor
  ↓
Authorization
  ↓
Allowed?
  ├── No → Reject
  └── Yes
       ↓
   Execute Use Case
18. Policy Evaluation

Las policies definidas en E06 pueden intervenir antes o durante la ejecución:

Application Service
        │
        ▼
Policy Evaluation
        │
   ┌────┴────┐
   ▼         ▼
Allowed    Denied
19. Domain Invocation

Después de preparar el contexto:

Application Service
       ↓
Load Domain Objects
       ↓
Domain Service
       ↓
Domain Decision
20. Repository Invocation

La persistencia se coordina mediante abstracciones:

Application Service
       ↓
Repository Interface
       ↓
Infrastructure

El Application Service no debería conocer detalles de SQL.

21. Transaction Boundary

La transacción normalmente se inicia alrededor del caso de uso:

BEGIN
   ↓
Load
   ↓
Execute
   ↓
Persist
   ↓
Commit

Si ocurre un error:

ROLLBACK
22. Unit of Work

EVOXA puede utilizar un patrón Unit of Work:

Application Service
       ↓
Unit of Work
       │
       ├── Repository A
       ├── Repository B
       └── Repository C
       ↓
Commit

Esto mantiene la consistencia del caso de uso.

23. Domain Service Composition

Un Application Service puede coordinar varios Domain Services:

GenerateTrainingPlan
        │
        ├── GoalAnalysisService
        ├── ExerciseRecommendationService
        ├── TrainingLoadService
        └── RecoveryEvaluationService

La coordinación pertenece a Application Layer.

24. Application Service Orchestration

El Application Service debe responder:

What happens first?
What happens next?
What must be persisted?
What event must be emitted?
What integration must execute?

No:

What is the business rule itself?

Esa última pregunta pertenece al dominio.

25. Example — Complete Workout
CompleteWorkout
      │
      ▼
Authenticate Context
      │
      ▼
Authorize
      │
      ▼
Load Workout
      │
      ▼
Validate Policy
      │
      ▼
WorkoutCompletionDomainService
      │
      ▼
Persist Workout
      │
      ▼
Persist Progress
      │
      ▼
Publish WorkoutCompleted
      │
      ▼
Return Result
26. Application Result

Los resultados deben representar el resultado del caso de uso.

Ejemplo:

CompleteWorkoutResult

puede contener:

workoutId
completedAt
trainingLoad
progressImpact

No debería devolver directamente una respuesta HTTP.

27. DTO Separation

Se recomienda separar:

API DTO
Application Command
Domain Model
Application Result
API Response DTO

Arquitectura:

HTTP Request
   ↓
Request DTO
   ↓
Command
   ↓
Domain
   ↓
Result
   ↓
Response DTO
   ↓
HTTP Response
28. Mapping

Los mappings deben ser explícitos:

Request DTO
      ↓
Command

y:

Application Result
      ↓
Response DTO

Esto evita que los modelos internos se conviertan accidentalmente en contratos públicos.

29. Validation

Debe distinguirse:

Input Validation
Required fields
Format
Type
Range
Domain Validation
Business rules
Invariants
Domain constraints

Ejemplo:

age must be integer

es input validation.

Mientras:

athlete cannot start advanced program without prerequisites

es domain validation.

30. Application Errors

Los errores del Application Layer pueden representar:

Unauthorized
Forbidden
ResourceNotFound
Conflict
InvalidCommand
PolicyViolation

El Domain Layer puede producir:

InvalidWorkoutState
GoalConstraintViolation
ProgressionLimitExceeded
31. Error Translation

Arquitectura:

Infrastructure Error
       ↓
Repository / Integration
       ↓
Application Error
       ↓
API Error

Los detalles internos no deben filtrarse al cliente.

32. Not Found

Ejemplo:

Workout not found

El Application Service determina cómo manejar la ausencia.

La API posteriormente puede producir:

404 Not Found
33. Conflict

Ejemplo:

Workout already completed

puede traducirse a:

409 Conflict

según el contrato API.

34. Idempotency

Los Commands críticos deben soportar idempotencia cuando corresponda.

Ejemplos:

CreatePayment
CompleteWorkout
CreateSubscription
ProcessWebhook

Una solicitud repetida no debería generar efectos duplicados.

35. Idempotency Key

Puede utilizarse:

idempotencyKey

junto con:

tenantId
+
actorId
+
operation

para identificar una operación única.

36. Application Events

Después de completar correctamente una operación:

Application Service
       ↓
Domain Event
       ↓
Integration Event

La publicación debe ocurrir de forma consistente con la transacción.

37. Transactional Outbox

Para evitar:

Database Commit
      ✓

Event Publish
      ✗

EVOXA puede utilizar:

Transactional Outbox

Flujo:

BEGIN
  │
  ├── Persist Domain Changes
  │
  └── Persist Outbox Event
  │
COMMIT
  │
  ▼
Outbox Worker
  │
  ▼
Message Broker
38. Application Events vs Domain Events

El Domain Layer puede producir:

WorkoutCompleted

El Application Layer puede determinar cuándo debe publicarse hacia otros sistemas.

39. Integration Coordination

Cuando un caso de uso necesita un sistema externo:

Application Service
       ↓
Integration Interface
       ↓
External Provider

Ejemplo:

CreateSubscription
       ↓
Payment Gateway
40. External Integrations

No deben llamarse directamente desde entidades.

Incorrecto:

Workout.complete()
   ↓
sendEmail()

Preferible:

Application Service
   ↓
Workout.complete()
   ↓
Persist
   ↓
Notification Integration
41. Application Service and AI

Los casos de uso que requieren IA pueden seguir:

Application Service
       ↓
AI Application Adapter
       ↓
AI Service
       ↓
AI Result
       ↓
Domain Validation
       ↓
Persist

La IA no obtiene autoridad automática para modificar el dominio.

42. AI Decision Boundary

Ejemplo:

GenerateTrainingPlan
        │
        ▼
AI Recommendation
        │
        ▼
Domain Validation
        │
   ┌────┴────┐
   ▼         ▼
Valid      Invalid
   │
   ▼
Persist
43. Authorization Before AI

Las operaciones AI costosas deben verificar autorización antes de consumir recursos:

Authenticate
    ↓
Authorize
    ↓
Policy
    ↓
AI Operation
44. Application Service and Billing

Una operación premium puede requerir:

Authorization
       ↓
Entitlement
       ↓
Policy
       ↓
Execute

Ejemplo:

GenerateAdvancedPlan

puede comprobar:

User Subscription
+
Feature Entitlement
45. Application Service and Entitlements

El flujo:

User
 ↓
Authentication
 ↓
Authorization
 ↓
Entitlement
 ↓
Policy
 ↓
Use Case

permite separar:

Permission

de:

Product Entitlement
46. Query Services

Las consultas pueden utilizar servicios especializados:

GetTrainingDashboard
GetProgressSummary
GetNutritionSummary
GetWorkoutHistory

Su objetivo es optimizar lectura y composición.

47. Query Optimization

Los Query Services pueden utilizar:

Read Models
Projections
Aggregations
Caching
Specialized Queries

sin contaminar el modelo de dominio.

48. Query vs Domain Model

No todas las consultas necesitan construir todo el agregado de dominio.

Ejemplo:

GetDashboard

puede obtener directamente una proyección optimizada.

Database
   ↓
Projection
   ↓
Query Service
   ↓
Response
49. CQRS Evolution

EVOXA puede evolucionar:

Simple CRUD
   ↓
Application Commands / Queries
   ↓
Separated Read Models
   ↓
Selective CQRS

No es necesario implementar CQRS completo desde el inicio.

50. Application Service Dependencies

Un Application Service puede depender de:

Repository Interfaces
Domain Services
Policy Services
Authorization Interfaces
Integration Interfaces
Unit of Work
Event Dispatcher
Clock
Idempotency Store
51. Dependency Injection

Las dependencias deben inyectarse:

Application Service
       │
       ├── Repository
       ├── Domain Service
       ├── Policy
       └── Event Publisher

Evitar instanciaciones globales ocultas.

52. Application Service Interface

Ejemplo:

interface CompleteWorkout {
    execute(command): CompleteWorkoutResult
}

El contrato debe expresar el caso de uso.

53. Naming

Preferir:

CreateWorkout
CompleteWorkout
GenerateTrainingPlan
RecordProgress
CancelSubscription

o:

CreateWorkoutService
CompleteWorkoutService
GenerateTrainingPlanService

Evitar:

GenericService
Manager
HandlerManager
BusinessService

si no aportan significado.

54. Application Service Organization

Estructura conceptual:

application/
├── commands/
│   ├── training/
│   ├── nutrition/
│   ├── progress/
│   └── billing/
├── queries/
├── results/
├── mappers/
├── workflows/
└── errors/
55. Command Handler Pattern

Una alternativa:

Command
   ↓
Command Handler
   ↓
Domain

Ejemplo:

CompleteWorkoutCommand
        ↓
CompleteWorkoutHandler
        ↓
WorkoutCompletionService

Esto puede facilitar CQRS.

56. Workflow Services

Para procesos complejos:

Application Workflow

puede coordinar múltiples pasos.

Ejemplo:

OnboardingWorkflow
Create User
 ↓
Create Profile
 ↓
Create Goals
 ↓
Initialize Training
 ↓
Initialize Nutrition
 ↓
Publish OnboardingCompleted
57. Long-Running Workflows

Procesos largos no deberían mantener una transacción abierta.

Preferir:

Command
 ↓
State
 ↓
Event
 ↓
Next Step
58. Saga Coordination

Ejemplo:

CreateSubscription
       ↓
Payment
       ↓
Provision Entitlement
       ↓
Activate Subscription

Si falla un paso:

Compensation
59. Background Application Services

Algunos casos de uso pueden ejecutarse en workers:

GenerateLargeProgressReport
GenerateNutritionAnalysis
ProcessImport
GenerateAIPlan

Arquitectura:

API
 ↓
Create Job
 ↓
Queue
 ↓
Worker
 ↓
Application Service
60. Application Service Observability

Cada ejecución debería registrar métricas como:

use_case
duration
success
failure
tenant
actor
traceId

sin registrar información sensible innecesaria.

61. Distributed Tracing

Un caso de uso puede generar:

Trace
 │
 ├── API
 ├── Authorization
 ├── Policy
 ├── Application Service
 ├── Domain Service
 ├── Database
 └── Event Publishing
62. Audit

Las operaciones importantes deben poder auditarse:

Who
What
When
Tenant
Result
Correlation

Ejemplos:

SubscriptionChanged
RoleChanged
WorkoutCompleted
AIPlanGenerated
63. Security Boundaries

El Application Service es una frontera crítica.

Debe impedir:

Unauthorized Use Case
Cross-Tenant Access
Privilege Escalation
Policy Bypass
Invalid State Transition
64. Multi-Tenant Execution

Ejemplo:

Request
  ↓
Tenant Context
  ↓
Authorization
  ↓
Application Service
  ↓
Repository

El repository debe utilizar el contexto apropiado para evitar acceso cruzado.

65. Application Service Isolation

Un Application Service nunca debe asumir:

"el controller ya validó todo"

Las garantías críticas deben permanecer en las capas apropiadas.

66. Transaction + Authorization

La autorización debe producirse antes de ejecutar efectos importantes:

Authorize
   ↓
Begin Transaction
   ↓
Execute
   ↓
Commit
67. Concurrency

Casos como:

CompleteWorkout
ChangeSubscription
UpdateGoal

pueden requerir:

Optimistic Locking
Version Fields
Conflict Detection
68. Optimistic Concurrency

Ejemplo:

Workout version = 5

Si otro proceso lo cambia:

Workout version = 6

la primera operación debe detectar el conflicto.

69. Application Service Performance

Prioridad:

Correctness
Security
Consistency
Observability
Performance

Después se pueden aplicar:

Caching
Batching
Async Processing
Read Models
70. Application Service Caching

Las Queries pueden utilizar cache:

Query Service
    ↓
Cache
    ↓
Database

Los Commands deben invalidar o actualizar caches cuando corresponda.

71. Batch Operations

Operaciones masivas:

ImportUsers
ProcessWorkoutHistory
GenerateProgressReports

deben utilizar procesamiento batch en lugar de ejecutar miles de operaciones HTTP individuales.

72. Application Service Testing

Cada Application Service debe tener:

Unit Tests
Integration Tests
Authorization Tests
Policy Tests
Transaction Tests
Idempotency Tests
Contract Tests
73. Application Unit Tests

Se pueden mockear:

Repository
Domain Service
Policy
Event Publisher
Integration

para probar la orquestación.

74. Application Integration Tests

Validan:

Application Service
+
Real Database
+
Repositories
+
Policies
+
Events

según el nivel de integración requerido.

75. Security Tests

Debe comprobarse:

Unauthorized
Forbidden
Wrong Tenant
Wrong Role
Policy Denial
Expired Identity
Invalid Context
76. Idempotency Tests

Ejemplo:

Command
 ↓
Execute
 ↓
Execute same command

Resultado esperado:

One Business Effect

cuando la operación requiere idempotencia.

77. Application Anti-Patterns

EVOXA debe evitar:

God Application Service
Fat Controller
Anemic Domain Logic
Infrastructure Leakage
Transaction Leakage
Hidden Side Effects
Unbounded Orchestration
Direct SQL
Direct HTTP in Use Case Logic
78. Fat Controller

Incorrecto:

Controller
 ├── Authorization
 ├── Business Rules
 ├── Database
 ├── Events
 └── External APIs

Correcto:

Controller
   ↓
Application Service
79. God Application Service

Incorrecto:

EvoxaApplicationService

con cientos de casos de uso.

Preferible:

Training Application Services
Nutrition Application Services
Progress Application Services
Billing Application Services
80. Infrastructure Leakage

Incorrecto:

CreateWorkoutService
   ↓
SQLAlchemy Session
   ↓
PostgreSQL

Preferible:

CreateWorkoutService
   ↓
WorkoutRepository
   ↓
Infrastructure
81. Hidden Side Effects

Un método como:

completeWorkout()

no debería enviar silenciosamente:

Email
Push
Webhook
Payment
AI request

sin que el flujo de aplicación lo haga explícito.

82. Application Service Documentation

Cada caso de uso debe documentar:

Purpose
Actor
Command
Preconditions
Authorization
Policies
Domain Services
Repositories
Transaction
Events
Integrations
Errors
Idempotency
Result
83. Use Case Contract

Ejemplo:

Use Case:
Complete Workout

Actor:
Athlete

Preconditions:
Workout exists
Workout belongs to tenant
Workout is in progress

Authorization:
CompleteWorkout permission

Policies:
WorkoutCompletionPolicy

Domain Services:
WorkoutCompletionService
TrainingLoadService

Persistence:
WorkoutRepository
ProgressRepository

Events:
WorkoutCompleted

Result:
WorkoutCompletionResult
84. Application Service Governance

Los Application Services deben cumplir:

Architecture Rules
Security Rules
API Contracts
Policy Rules
Transaction Rules
Observability Standards
Testing Standards
Documentation Standards
85. Application Service Lifecycle
Identify Use Case
       ↓
Define Contract
       ↓
Define Authorization
       ↓
Define Policies
       ↓
Define Domain Operations
       ↓
Implement Orchestration
       ↓
Persist
       ↓
Publish Events
       ↓
Test
       ↓
Deploy
       ↓
Observe
       ↓
Evolve
86. Relationship with E07
E07 Service Architecture
          │
          ▼
Service Boundary
          │
          ▼
Application Layer
          │
          ▼
E09 Application Services

E07 define qué es un service.

E09 define cómo un service ejecuta casos de uso.

87. Relationship with E08
Application Service
       │
       ▼
Domain Service
       │
       ▼
Domain Model

La responsabilidad queda claramente separada:

Application
→ Orchestration

Domain
→ Business Logic
88. Relationship with E10

La siguiente capa será:

E09 Application Services
          │
          ▼
E10 Repository Architecture

E10 definirá cómo los Application Services acceden a la persistencia mediante repositories y abstracciones de datos.

89. EVOXA Application Architecture

La arquitectura consolidada queda:

                         CLIENT
                           │
                           ▼
                     API / Interface
                           │
                           ▼
                    Authentication
                           │
                           ▼
                     Authorization
                           │
                           ▼
                        Policy
                           │
                           ▼
                ┌──────────────────────┐
                │ Application Service  │
                └──────────────────────┘
                           │
                ┌──────────┼──────────┐
                ▼          ▼          ▼
             Domain     Repository   Event
             Service    Interface    Interface
                │          │          │
                ▼          ▼          ▼
             Domain    Infrastructure  Broker
             Model
90. Example — EVOXA End-to-End

Caso:

Generate Training Plan

Client
  │
  ▼
POST /training-plans
  │
  ▼
Authentication
  │
  ▼
Authorization
  │
  ▼
Entitlement
  │
  ▼
GenerateTrainingPlanApplicationService
  │
  ├── Load Athlete Profile
  ├── Load Goals
  ├── Load Training History
  ├── Evaluate Policies
  │
  ▼
TrainingPlanDomainService
  │
  ├── Analyze Goal
  ├── Evaluate Training Load
  ├── Evaluate Recovery
  ├── Select Exercises
  └── Build Plan
  │
  ▼
Validate Domain Result
  │
  ▼
Persist Training Plan
  │
  ▼
Create Domain Event
  │
  ▼
Outbox
  │
  ▼
TrainingPlanGenerated
  │
  ▼
Application Result
  │
  ▼
API Response

Este flujo constituye el patrón de referencia para los casos de uso complejos de EVOXA.

91. Definition of Done

E09 queda definido cuando EVOXA dispone de:

✓ Application Service Definition
✓ Use Case Model
✓ Command Model
✓ Query Model
✓ Application Context
✓ Authentication Integration
✓ Authorization Integration
✓ Policy Integration
✓ Domain Service Integration
✓ Repository Integration
✓ Transaction Boundaries
✓ Unit of Work
✓ DTO Separation
✓ Result Model
✓ Error Translation
✓ Idempotency
✓ Domain Events
✓ Integration Events
✓ Transactional Outbox
✓ External Integration Coordination
✓ AI Integration Boundary
✓ Entitlement Checks
✓ Multi-Tenant Execution
✓ Concurrency Control
✓ Query Optimization
✓ CQRS Evolution Strategy
✓ Background Workflows
✓ Saga Coordination
✓ Observability
✓ Auditability
✓ Security Testing
✓ Application Testing
✓ Anti-Patterns
✓ Governance
✓ Documentation
✓ Lifecycle
92. Arquitectura final E07–E09

Con los tres capítulos terminados:

E07 — Service Architecture
          │
          ▼
Define Service Boundaries
          │
          ▼
E08 — Domain Services Architecture
          │
          ▼
Defines Business Operations
          │
          ▼
E09 — Application Services Architecture
          │
          ▼
Orchestrates Use Cases
          │
          ▼
E10 — Repository Architecture
          │
          ▼
Defines Persistence Access

La separación fundamental de EVOXA queda:

┌─────────────────────────────────────┐
│ API / Interface                     │
├─────────────────────────────────────┤
│ Application Services                │
│ → Use Case Orchestration            │
├─────────────────────────────────────┤
│ Domain Services                     │
│ → Business Decisions                │
├─────────────────────────────────────┤
│ Domain Model                        │
│ → Entities / Value Objects / Rules  │
├─────────────────────────────────────┤
│ Repository Interfaces               │
│ → Persistence Abstraction           │
├─────────────────────────────────────┤
│ Infrastructure                     │
│ → Database / Cache / External APIs  │
└─────────────────────────────────────┘

E09 establece la capa de orquestación de EVOXA: convierte una intención del usuario o de otro sistema en un caso de uso ejecutable, coordina seguridad, políticas, dominio, persistencia, eventos e integraciones, y mantiene esas responsabilidades fuera de los controllers y del modelo de dominio.
