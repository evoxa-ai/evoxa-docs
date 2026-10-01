E08 — EVOXA Domain Services Architecture
Engineering Specification

Pertenece a: Engineering Specification
Volumen: volume-09-engineering
Capítulo: E08 — Domain Services Architecture
Anterior: E07 — Service Architecture
Siguiente: E09 — Application Services Architecture

1. Propósito

E08 define la arquitectura de los Domain Services de EVOXA.

Mientras E07 establece la arquitectura general de servicios, este capítulo se concentra en una pregunta específica:

¿Dónde vive y cómo se ejecuta la lógica de negocio que no pertenece naturalmente a una sola entidad del dominio?

Los Domain Services constituyen una pieza fundamental de la arquitectura porque permiten mantener las reglas de negocio dentro del dominio sin convertir las entidades, controllers o application services en componentes excesivamente complejos.

2. Objetivos

Domain Services debe proporcionar:

Domain Business Logic
Domain Operations
Cross-Entity Rules
Domain Calculations
Domain Decisions
Domain Validation
Domain Coordination
Domain Policies
Domain Invariants

Debe mantener una separación clara entre:

Domain Logic
Application Orchestration
Infrastructure
3. Principio Fundamental

Un Domain Service representa una operación significativa del dominio que no pertenece naturalmente a una única entidad.

Domain Service
      │
      ├── Domain Rules
      ├── Domain Entities
      ├── Value Objects
      └── Domain Policies

No debe convertirse simplemente en:

utils/
helpers/
common/
misc/
4. Domain Service vs Application Service

La diferencia fundamental:

Application Service
    → Orquesta un caso de uso

Domain Service
    → Ejecuta una regla u operación del dominio

Ejemplo:

CompleteWorkoutApplicationService
             │
             ▼
WorkoutCompletionDomainService
             │
             ├── Workout
             ├── Exercise
             └── Progression Rules
5. Domain Service Responsibilities

Un Domain Service puede encargarse de:

Calculate
Evaluate
Determine
Validate
Recommend
Plan
Classify
Compare
Apply Domain Rule

Siempre que la operación represente conocimiento propio del dominio.

6. Cuándo Crear un Domain Service

Debe crearse cuando una operación:

No pertenece claramente a una sola entidad.
Utiliza múltiples entidades.
Representa una regla de negocio importante.
Debe ser reutilizada por varios casos de uso.
Requiere una decisión específica del dominio.
7. Cuándo NO Crear un Domain Service

No debe crearse para:

Simple CRUD
Database Queries
HTTP Requests
Serialization
Logging
Authentication
Generic Utilities

Ejemplo incorrecto:

UserDomainService.getUserById()

si simplemente consulta una base de datos.

8. Domain Service Model
                  Domain
                     │
        ┌────────────┼────────────┐
        │            │            │
     Entities    Value Objects   Policies
        │            │            │
        └────────────┼────────────┘
                     ▼
              Domain Services
                     │
                     ▼
              Domain Decisions
9. Domain Service Purity

Siempre que sea posible, un Domain Service debe ser:

Deterministic
Predictable
Testable
Side-effect controlled

Ejemplo:

calculateTrainingLoad()

debería producir el mismo resultado para los mismos datos.

10. Domain Services and Side Effects

Debe evitarse que un Domain Service directamente:

SendEmail
CallHTTP
WriteLogs
PublishInfrastructure Message

Esas responsabilidades pertenecen a otras capas.

El Domain Service puede producir una decisión o resultado que posteriormente será utilizado por Application Services.

11. Domain Service Inputs

Los inputs deben representar conceptos del dominio.

Preferencia:

Workout
AthleteProfile
TrainingHistory
TrainingConstraints

sobre estructuras técnicas como:

HttpRequest
DatabaseRow
JSON
RequestDTO
12. Domain Service Outputs

Los resultados deben representar decisiones o conceptos de negocio:

TrainingRecommendation
ProgressionDecision
NutritionEvaluation
EligibilityResult
WorkoutPlan

No deberían devolver directamente:

HTTP Response
ORM Model
Database Record
13. Domain Service Dependencies

Un Domain Service puede depender de:

Entities
Value Objects
Domain Policies
Domain Specifications
Domain Repositories Interfaces
Domain Events

Debe evitar depender directamente de:

FastAPI
PostgreSQL
Redis
HTTP clients
Cloud SDKs
14. Repository Dependency

Cuando el dominio necesita consultar información persistida, puede depender de una abstracción:

TrainingHistoryRepository

y no:

PostgresTrainingHistoryRepository

Arquitectura:

Domain Service
      │
      ▼
Repository Interface
      ▲
      │
Infrastructure Implementation
15. Domain Services and Entities

Los Domain Services no deben reemplazar a las entidades.

Regla:

Entity
→ Owns its own state and invariants

Domain Service
→ Owns domain operation involving multiple concepts
16. Example

Una entidad:

Workout

puede controlar:

addExercise()
removeExercise()
complete()

Mientras un Domain Service puede controlar:

calculateWorkoutLoad()

porque necesita:

Workout
+
Exercises
+
Athlete Profile
+
Training History
17. Domain Services and Value Objects

Los Value Objects deben utilizarse para representar conceptos importantes:

Weight
Distance
Duration
Intensity
Calories
MacroDistribution
TrainingLoad

El Domain Service puede trabajar con estos conceptos en lugar de tipos primitivos dispersos.

18. Domain Service Categories

EVOXA puede clasificar Domain Services como:

Calculation Services
Evaluation Services
Decision Services
Planning Services
Validation Services
Recommendation Services
Transformation Services
Coordination Services
19. Calculation Services

Realizan cálculos de dominio.

Ejemplos:

TrainingLoadCalculationService
CalorieRequirementService
ProgressScoreService
RecoveryScoreService
NutritionMacroService
20. Evaluation Services

Evalúan estados.

Ejemplos:

TrainingReadinessService
GoalProgressEvaluationService
NutritionComplianceService
WorkoutQualityEvaluationService
21. Decision Services

Toman decisiones basadas en reglas.

Ejemplos:

WorkoutProgressionDecisionService
PlanAdjustmentDecisionService
EligibilityDecisionService
22. Planning Services

Generan planes de dominio.

Ejemplos:

TrainingPlanGenerationService
NutritionPlanGenerationService
RecoveryPlanService
23. Validation Services

Validan reglas complejas del dominio.

Ejemplos:

WorkoutValidationService
TrainingPlanValidationService
NutritionConstraintValidationService
GoalValidationService
24. Recommendation Services

Generan recomendaciones.

Ejemplos:

ExerciseRecommendationService
TrainingRecommendationService
NutritionRecommendationService
RecoveryRecommendationService
25. Transformation Services

Transforman conceptos de dominio.

Ejemplo:

TrainingHistory
      ↓
ProgressProfile

o:

NutritionProfile
      ↓
MacroTarget
26. Coordination Services

Coordinan múltiples conceptos de dominio cuando la operación sigue siendo una responsabilidad de dominio.

Ejemplo:

WorkoutCompletionDomainService

puede coordinar:

Workout
Exercise Performance
Training Load
Progress

Pero la orquestación técnica de persistencia y eventos debe permanecer fuera del dominio.

27. Domain Service Boundaries

Cada Domain Service debe tener una responsabilidad clara.

Incorrecto:

FitnessDomainService

con cientos de operaciones.

Preferible:

TrainingLoadService
WorkoutProgressionService
RecoveryEvaluationService
ExerciseRecommendationService
28. Domain Service Granularity

No se debe crear un Domain Service para cada método.

Incorrecto:

CalculateSetsService
CalculateRepsService
CalculateWeightService

si todos forman una única decisión de negocio.

Preferible:

WorkoutProgressionService
29. Domain Service Cohesion

Los servicios deben agrupar comportamiento relacionado.

High Cohesion
Low Coupling
30. Domain Service Dependencies

Ejemplo:

WorkoutProgressionService
       │
       ├── TrainingHistory
       ├── AthleteProfile
       ├── ProgressionPolicy
       └── ExerciseConstraints

No debería depender directamente de:

HTTP
Database Driver
FastAPI
Redis
31. Domain Policies

Los Domain Services pueden utilizar Policies definidas en E06.

Domain Service
      │
      ▼
Policy
      │
      ▼
Decision

Ejemplo:

WorkoutProgressionService
          │
          ▼
ProgressionPolicy
          │
          ▼
Increase / Maintain / Deload
32. Domain Specifications

Cuando una regla puede expresarse como una condición reusable:

Specification

Ejemplo:

IsEligibleForAdvancedProgram

Puede ser utilizada por múltiples Domain Services.

33. Domain Service Composition

Los Domain Services pueden colaborar:

WorkoutPlanningService
        │
        ├── ExerciseSelectionService
        ├── TrainingLoadService
        └── RecoveryEvaluationService

Debe evitarse una cadena excesivamente profunda.

34. Domain Service Calls

Preferencia:

Service A
   ↓
Service B

solo cuando existe una relación de dominio real.

No utilizar Domain Services como una capa genérica de utilidades.

35. Domain Events

Un Domain Service puede generar un resultado que produzca un Domain Event.

Ejemplo:

WorkoutCompletionService
          │
          ▼
WorkoutCompleted

El Application Layer puede encargarse posteriormente de publicar el evento.

36. Domain Event Separation

Debe distinguirse:

Domain Event
≠
Integration Event

Domain Event:

Dentro del modelo de dominio

Integration Event:

Comunicación entre bounded contexts/services
37. Domain Service Transactions

El Domain Service no debería controlar directamente:

BEGIN TRANSACTION
COMMIT
ROLLBACK

La frontera transaccional normalmente pertenece al Application Layer / Unit of Work.

38. Domain Service Error Model

Los errores deben representar conceptos del dominio.

Ejemplos:

InvalidWorkoutState
InsufficientRecovery
GoalConstraintViolation
TrainingLimitExceeded
NutritionConstraintViolation

No:

SQLException
HTTPException
RedisError
39. Domain Exceptions

Ejemplo conceptual:

WorkoutCannotBeCompleted

puede ocurrir cuando:

Workout state != InProgress

La API posteriormente transforma ese error en una respuesta HTTP adecuada.

40. Domain Service Determinism

Cuando sea posible:

Input
 ↓
Domain Service
 ↓
Deterministic Output

Esto facilita:

Testing
Auditing
Reproducibility
AI Evaluation
41. Time Dependency

Cuando un Domain Service depende del tiempo:

CurrentDate
CurrentTime

debe utilizar una abstracción de reloj:

Clock

en lugar de acceder directamente al sistema.

42. Randomness

Cuando exista aleatoriedad:

Recommendation
Selection
AI Sampling

debe existir una estrategia explícita para poder reproducir pruebas.

43. External Information

Un Domain Service no debe acceder directamente a servicios externos.

Incorrecto:

NutritionService
    ↓
OpenAI API

Preferible:

Application Service
    ↓
AI Integration
    ↓
AI Result
    ↓
Domain Service
44. AI and Domain Services

La IA no debe sustituir las reglas fundamentales del dominio.

Arquitectura:

AI
 ↓
Recommendation
 ↓
Domain Policy
 ↓
Domain Service
 ↓
Validated Decision

Esto es especialmente importante en EVOXA.

45. AI Recommendation Validation

Ejemplo:

AI recommends:
Heavy training tomorrow

El dominio puede determinar:

Recovery insufficient

Resultado:

Recommendation rejected

La autoridad final sobre las reglas del producto permanece en el dominio.

46. Domain Service Security

Los Domain Services no deben asumir que los datos recibidos son confiables.

La validación de:

Identity
Authorization
Tenant Context

ocurre antes de llegar al dominio.

El dominio, sin embargo, debe proteger sus invariantes.

47. Tenant-Aware Domain Services

Cuando una regla depende del tenant:

Tenant Configuration
Tenant Policy
Tenant Limits

debe incorporarse explícitamente al modelo de dominio.

48. Multi-Tenant Isolation

Un Domain Service nunca debe mezclar datos de diferentes tenants.

Tenant A
  ↓
Domain Service
  ↓
Tenant A Data

y:

Tenant B
  ↓
Domain Service
  ↓
Tenant B Data
49. Domain Service Testing

Cada Domain Service debe poder probarse sin infraestructura.

Unit Test
    ↓
Domain Service
    ↓
In-memory domain objects

No debería necesitar:

PostgreSQL
Redis
HTTP
Docker

para las pruebas unitarias.

50. Test Categories

Se deben considerar:

Happy Path
Boundary Conditions
Invalid State
Business Rule Violations
Multi-Entity Scenarios
Tenant Isolation
Policy Evaluation
Deterministic Calculations
51. Example — Training Load
TrainingLoadService

Input:

Workout
ExercisePerformance
AthleteProfile

Output:

TrainingLoad

Proceso:

Workout
   │
   ▼
Volume
   │
   ▼
Intensity
   │
   ▼
Exercise Factors
   │
   ▼
Training Load
52. Example — Workout Progression
WorkoutProgressionService

Input:

CurrentWorkout
TrainingHistory
RecoveryState
ProgressionPolicy

Output:

ProgressionDecision

Ejemplo:

Increase
Maintain
Reduce
Deload
53. Example — Nutrition
MacroTargetService

Input:

UserProfile
Goal
ActivityLevel
NutritionPolicy

Output:

MacroTarget

No debería devolver directamente un HTTP response.

54. Example — Goal Evaluation
GoalProgressService

puede evaluar:

Goal
CurrentMetrics
HistoricalMetrics
TimePeriod

y producir:

ProgressStatus
ProgressPercentage
Trend
Recommendation
55. Domain Service Interfaces

Los contratos deben ser simples:

interface WorkoutProgressionService {
    evaluate(context): ProgressionDecision
}

El contrato debe expresar lenguaje del dominio.

56. Domain Service Naming

Preferir nombres:

TrainingLoadService
WorkoutProgressionService
RecoveryEvaluationService
GoalProgressService
NutritionPlanningService

Evitar:

GenericService
BusinessService
HelperService
Manager
Processor
Utils

cuando no expresan claramente el dominio.

57. Domain Service Organization

Estructura conceptual:

domain/
├── entities/
├── value_objects/
├── policies/
├── specifications/
├── services/
│   ├── training/
│   ├── nutrition/
│   ├── progress/
│   └── coaching/
├── events/
└── exceptions/
58. Domain Service Module

Ejemplo:

domain/services/training/
├── workout_progression.py
├── training_load.py
├── recovery_evaluation.py
└── exercise_recommendation.py

La estructura física puede variar según el lenguaje y framework.

59. Domain Service Documentation

Cada servicio debería documentar:

Purpose
Domain Responsibility
Inputs
Outputs
Rules
Dependencies
Policies
Errors
Events
Examples
Tests
60. Domain Service Contract

Formato recomendado:

Service:
WorkoutProgressionService

Purpose:
Determine the next progression action.

Inputs:
Workout
TrainingHistory
RecoveryState
ProgressionPolicy

Output:
ProgressionDecision

Rules:
- Respect progression limits
- Respect recovery constraints
- Respect exercise constraints

Errors:
InvalidWorkoutState
InsufficientData
61. Domain Service Observability

El dominio no debería depender directamente de logging infrastructure.

Si se necesita trazabilidad:

Application Layer
       ↓
Domain Execution
       ↓
Result
       ↓
Telemetry
62. Auditability

Las decisiones importantes deben poder explicar:

Input
Rules
Policy
Decision
Timestamp
Actor
Tenant

Esto será especialmente importante para:

AI
Coaching
Nutrition
Billing
Security
63. Explainable Decisions

Los Domain Services pueden devolver:

Decision
+
Reason
+
AppliedRules

Ejemplo:

Decision:
Maintain

Reason:
Recovery below progression threshold

Rules:
RecoveryPolicy.v2
ProgressionPolicy.v3
64. Domain Service Versioning

Cuando una regla cambia significativamente:

ProgressionPolicy v1
ProgressionPolicy v2

el Domain Service debe poder determinar qué versión de reglas aplica cuando sea necesario.

65. Backward Compatibility

Cambios en Domain Services deben evitar romper:

Existing Workflows
Existing APIs
Existing Events
Existing Data

cuando sea posible.

66. Domain Service Performance

Los Domain Services deben ser eficientes, pero la optimización no debe destruir claridad.

Prioridad:

Correctness
→ Maintainability
→ Observability
→ Performance
67. Heavy Domain Calculations

Procesos costosos pueden delegarse a workers:

Application Service
      ↓
Job
      ↓
Domain Calculation
      ↓
Result

Ejemplos:

Large Progress Analysis
Historical Training Analysis
AI-assisted Planning
Population Analytics
68. Domain Service Concurrency

Cuando varias operaciones puedan modificar el mismo concepto:

Concurrency Control

debe definirse en el Application/Infrastructure layer.

El dominio debe proteger sus invariantes independientemente.

69. Domain Service Anti-Patterns

EVOXA debe evitar:

God Domain Service
Anemic Domain Model
Infrastructure Leakage
Database-driven Domain Logic
HTTP-aware Domain Logic
Global Mutable State
Hidden Dependencies
Generic Service Objects
70. God Service

Incorrecto:

EvoxaDomainService

que contiene:

Training
Nutrition
Billing
Users
AI
Notifications

Debe dividirse por bounded context y responsabilidad.

71. Anemic Domain Model

No trasladar toda la lógica a Application Services:

Application Service
   └── 500 lines of business rules

Las reglas deben vivir en:

Entities
Value Objects
Policies
Specifications
Domain Services

según corresponda.

72. Infrastructure Leakage

Incorrecto:

TrainingDomainService
   ↓
SQLAlchemy

Preferible:

TrainingDomainService
   ↓
Repository Interface
73. Database-Driven Domain Logic

No debe dependerse de stored procedures como mecanismo principal para expresar toda la lógica del dominio.

La base de datos proporciona:

Persistence
Constraints
Indexes
Queries

El dominio proporciona:

Business Rules
Decisions
Behavior
74. Domain Service Governance

Los Domain Services deben estar sujetos a:

Architecture Standards
Security Policies
Coding Standards
Testing Standards
Observability Standards
Documentation Standards
75. Domain Service Lifecycle
Identify Rule
     ↓
Model Domain Concept
     ↓
Define Service
     ↓
Implement
     ↓
Test
     ↓
Integrate
     ↓
Observe
     ↓
Evolve
     ↓
Deprecate
76. Relationship with E07
E07 Service Architecture
          │
          ▼
Service Boundary
          │
          ▼
Application Service
          │
          ▼
E08 Domain Services
          │
          ├── Entities
          ├── Value Objects
          ├── Policies
          └── Specifications
77. Relationship with E06

E06 define:

Policy

E08 define:

Domain Service

Relación:

Policy
  ↓
Rule / Constraint
  ↓
Domain Service
  ↓
Decision
78. Relationship with E09

La evolución de la especificación será:

E07 Service Architecture
        ↓
E08 Domain Services Architecture
        ↓
E09 Application Services Architecture
        ↓
E10 Repository Architecture
        ↓
E11 Integration Architecture

Esto mantiene una separación clara entre:

Business Logic
Application Orchestration
Persistence
External Integrations
79. EVOXA Domain Service Map

La arquitectura conceptual queda:

                         EVOXA DOMAIN
                              │
        ┌─────────────────────┼─────────────────────┐
        │                     │                     │
     Training             Nutrition             Progress
        │                     │                     │
        ▼                     ▼                     ▼
  Domain Services       Domain Services       Domain Services
        │                     │                     │
        └─────────────────────┼─────────────────────┘
                              │
                         Domain Policies
                              │
                         Domain Entities
                              │
                         Value Objects
80. Definition of Done

E08 queda definido cuando EVOXA dispone de:

✓ Domain Service Definition
✓ Domain Service Responsibilities
✓ Domain Service Boundaries
✓ Domain Service Categories
✓ Domain Service Naming
✓ Domain Service Dependencies
✓ Entity Integration
✓ Value Object Integration
✓ Policy Integration
✓ Specification Integration
✓ Domain Events
✓ Domain Exceptions
✓ Repository Abstractions
✓ Transaction Separation
✓ AI Interaction Rules
✓ Tenant Isolation
✓ Deterministic Execution
✓ Decision Explainability
✓ Domain Service Testing
✓ Performance Strategy
✓ Background Execution
✓ Concurrency Strategy
✓ Anti-Patterns
✓ Governance
✓ Documentation
✓ Lifecycle
✓ Service Evolution
81. Arquitectura resultante

Con E07 y E08, la arquitectura de ejecución de EVOXA queda establecida de esta forma:

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
                 Application Service
                           │
                           ▼
                  ┌────────────────┐
                  │ Domain Service │
                  └────────────────┘
                     │     │     │
                     ▼     ▼     ▼
                  Entity Policy Value
                           │
                           ▼
                    Domain Decision
                           │
                           ▼
                 Repository / Events
                           │
                           ▼
                  Infrastructure

E08 establece así la capa donde reside el conocimiento operativo del dominio de EVOXA: las decisiones, cálculos, evaluaciones y reglas que requieren colaboración entre múltiples conceptos del dominio, manteniéndolas independientes de HTTP, bases de datos, frameworks e infraestructura.
