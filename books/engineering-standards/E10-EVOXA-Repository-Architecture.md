E10 — EVOXA Repository Architecture
Engineering Specification

Pertenece a: Engineering Specification
Volumen: volume-09-engineering
Capítulo: E10 — Repository Architecture
Anterior: E09 — Application Services Architecture
Siguiente: E11 — Integration Architecture

1. Propósito

E10 define la arquitectura de Repositories de EVOXA.

Los repositories constituyen la frontera entre el modelo de aplicación/dominio y los mecanismos concretos de persistencia.

La pregunta central es:

¿Cómo puede EVOXA almacenar y recuperar información sin acoplar su dominio a PostgreSQL, SQLAlchemy, Redis, APIs externas u otras tecnologías de infraestructura?

El principio fundamental es:

Application / Domain
        │
        ▼
Repository Interface
        │
        ▼
Infrastructure Implementation
        │
        ▼
Database / Storage
2. Objetivos

Repository Architecture debe proporcionar:

Persistence Abstraction
Aggregate Persistence
Entity Retrieval
Query Abstraction
Transaction Integration
Tenant Isolation
Concurrency Control
Consistency
Caching Strategy
Persistence Error Translation
Testability
3. Principio Fundamental

El dominio debe conocer qué necesita persistir, pero no cómo se persiste.

Domain
   │
   ▼
Repository Contract
   │
   ▼
PostgreSQL Repository

No:

Domain
   │
   ▼
SQLAlchemy
   │
   ▼
PostgreSQL
4. Repository Definition

Un Repository representa una colección persistente de objetos del dominio.

Conceptualmente:

Repository
=
Domain Persistence Boundary

Ejemplos:

UserRepository
WorkoutRepository
TrainingPlanRepository
GoalRepository
NutritionPlanRepository
SubscriptionRepository
5. Repository Responsibilities

Un Repository puede encargarse de:

Create
Get
Find
Update
Delete
List
Existence Checks
Aggregate Retrieval
Persistence Mapping

Debe evitar incorporar reglas de negocio.

6. Repository vs Service

La diferencia:

Application Service
→ Orquesta el caso de uso

Domain Service
→ Ejecuta lógica de dominio

Repository
→ Persiste y recupera datos

Ejemplo:

CompleteWorkout
      │
      ▼
WorkoutRepository.get()
      │
      ▼
WorkoutCompletionService
      │
      ▼
WorkoutRepository.save()
7. Repository Boundary

La frontera queda:

┌─────────────────────────────┐
│ Domain / Application        │
│                             │
│ Repository Interface        │
└──────────────┬──────────────┘
               │
═══════════════│═══════════════
       Persistence Boundary
═══════════════│═══════════════
               │
┌──────────────▼──────────────┐
│ Infrastructure              │
│                             │
│ Repository Implementation   │
└──────────────┬──────────────┘
               │
               ▼
          PostgreSQL
8. Repository Interfaces

Los contratos deben expresarse en lenguaje del dominio.

Ejemplo:

interface WorkoutRepository:

    get(workout_id)
    save(workout)
    delete(workout)

No deberían exponer detalles como:

execute_sql()
create_cursor()
get_session()
9. Repository Implementations

La implementación concreta pertenece a Infrastructure.

Ejemplo:

WorkoutRepository
       ▲
       │
PostgresWorkoutRepository

La aplicación depende del primero, no del segundo.

10. Dependency Inversion

La arquitectura debe seguir:

High-Level Modules
        │
        ▼
Repository Interface
        ▲
        │
Low-Level Implementation

Así:

Application
    ↓
Interface
    ↑
PostgreSQL

y no:

Application
    ↓
PostgreSQL
11. Aggregate Persistence

Los Repositories deben centrarse preferentemente en Aggregates.

Ejemplo:

WorkoutAggregate
      │
      ├── Workout
      ├── WorkoutExercise
      └── Performance

El Repository puede persistir el agregado completo.

WorkoutRepository.save(workoutAggregate)
12. Aggregate Root

Cada agregado debe tener una raíz.

Ejemplo:

TrainingPlan
   │
   ├── TrainingWeek
   ├── TrainingDay
   └── Workout

El acceso debería producirse mediante:

TrainingPlanRepository

en lugar de manipular arbitrariamente entidades internas.

13. Repository Scope

Un Repository debe tener un límite claro.

Preferible:

WorkoutRepository
TrainingPlanRepository
NutritionPlanRepository
ProgressRepository

Evitar:

EvoxaRepository

que termine manejando todos los dominios.

14. Repository Methods

Las operaciones deben ser intencionales.

Ejemplo:

findById()
findByAthlete()
save()
delete()
exists()

Para consultas de negocio complejas:

getActiveTrainingPlan()
getCurrentWorkout()

puede ser apropiado si representa una necesidad real del dominio.

15. Generic Repository

EVOXA no debe depender excesivamente de un:

GenericRepository<T>

como única abstracción.

Aunque puede ser útil para operaciones CRUD simples, los contratos específicos del dominio son preferibles cuando existen necesidades particulares.

16. Repository and Queries

Debe distinguirse:

Command-side Repository

de:

Query-side Data Access

Una consulta como:

GetTrainingDashboard

puede no necesitar reconstruir un agregado.

Puede utilizar:

TrainingDashboardQuery

o:

TrainingDashboardReadRepository
17. Read Repositories

Para consultas complejas:

ProgressReadRepository
DashboardReadRepository
AnalyticsReadRepository

pueden devolver modelos de lectura.

Ejemplo:

TrainingDashboardView

en lugar de:

TrainingPlanAggregate
18. CQRS Compatibility

La arquitectura de repositories debe permitir evolucionar hacia CQRS:

                Application
                     │
          ┌──────────┴──────────┐
          ▼                     ▼
       Commands              Queries
          │                     │
          ▼                     ▼
    Domain Repository      Read Repository
          │                     │
          ▼                     ▼
      Write Model           Read Model

No es obligatorio separar físicamente las bases de datos inicialmente.

19. Repository Transactions

Los Repositories no deberían controlar por sí solos la transacción completa.

Preferible:

Application Service
       │
       ▼
Unit of Work
       │
       ├── Repository A
       ├── Repository B
       └── Repository C
       │
       ▼
Commit
20. Unit of Work

El Unit of Work representa la unidad transaccional de un caso de uso.

Ejemplo:

with unit_of_work:

    workout = workout_repository.get(id)

    workout.complete()

    workout_repository.save(workout)

    progress_repository.save(progress)

    unit_of_work.commit()
21. Repository Atomicity

Si una operación modifica:

Workout
+
Progress
+
Training Metrics

el Application Service puede utilizar una única unidad transaccional.

BEGIN
   ↓
Workout
   ↓
Progress
   ↓
Metrics
   ↓
COMMIT
22. Repository Mapping

La infraestructura puede requerir modelos específicos:

Domain Entity
      ↕
Persistence Model

Ejemplo:

Workout
   ↕
WorkoutRecord

El mapping evita que los modelos ORM contaminen el dominio.

23. ORM Isolation

EVOXA puede utilizar un ORM como SQLAlchemy.

Pero:

SQLAlchemy Model

no debe convertirse automáticamente en:

Domain Entity

La separación recomendada es:

Domain Model
      │
      ▼
Repository Mapper
      │
      ▼
ORM Model
24. PostgreSQL

En la arquitectura actual de EVOXA, PostgreSQL puede actuar como almacenamiento principal.

Repository
    ↓
PostgreSQL Adapter
    ↓
PostgreSQL

El dominio continúa sin conocer PostgreSQL.

25. Database Schema Evolution

Los cambios de schema pertenecen a Infrastructure/Database Engineering.

Ejemplo:

Alembic Migration

no debe modificar directamente la lógica de dominio.

26. Repository and Migrations

Flujo:

Domain Change
     ↓
Repository Contract Change
     ↓
Infrastructure Implementation
     ↓
Database Migration

cuando sea necesario.

27. Repository Error Translation

Los errores de infraestructura no deben propagarse directamente al dominio.

Ejemplo:

UniqueViolation
      ↓
Repository
      ↓
PersistenceConflict
      ↓
Application
      ↓
Conflict
28. Database Exceptions

No deberían aparecer en:

Domain Services
Entities
Value Objects

Ejemplo incorrecto:

except IntegrityError:

dentro del dominio.

29. Not Found

Un repository puede devolver:

None

o lanzar una excepción específica de infraestructura/aplicación según el contrato.

La decisión debe ser consistente en EVOXA.

30. Repository Concurrency

Los repositories deben soportar estrategias como:

Optimistic Locking
Pessimistic Locking
Version Fields

cuando el dominio lo requiera.

31. Optimistic Locking

Ejemplo:

Workout
version = 7

Una actualización espera:

WHERE id = X
AND version = 7

Si no existe la fila:

ConcurrencyConflict
32. Pessimistic Locking

En operaciones críticas puede utilizarse:

SELECT ... FOR UPDATE

pero debe permanecer encapsulado en Infrastructure.

El Application Service solamente expresa la necesidad funcional.

33. Tenant Isolation

Los Repositories son una frontera crítica para multi-tenancy.

Toda consulta debe respetar:

tenant_id

cuando corresponda.

Ejemplo:

findWorkout(
    tenant_id,
    workout_id
)
34. Tenant Safety

Nunca debe ocurrir:

Tenant A
   ↓
WorkoutRepository
   ↓
Workout belonging to Tenant B

El Repository debe aplicar aislamiento de tenant de forma consistente.

35. Tenant Context

Puede utilizarse un contexto:

TenantContext

para evitar que cada capa tenga que reconstruir manualmente el tenant.

Sin embargo, las fronteras críticas deben continuar validándose.

36. Row-Level Security

Para escenarios de alta seguridad, PostgreSQL puede complementar la arquitectura con:

Row-Level Security

La defensa puede ser:

Application Authorization
+
Repository Tenant Filtering
+
Database RLS
37. Repository Security

Un Repository debe proteger contra:

Cross-Tenant Access
Unauthorized Data Access
Accidental Mass Updates
Unsafe Deletes
SQL Injection
38. Parameterized Queries

Las consultas deben utilizar parámetros.

Nunca:

SQL = "SELECT ... WHERE id = " + user_input

Preferible:

Parameterized Query

o mecanismos seguros del ORM.

39. Repository and Authorization

El Repository no sustituye Authorization.

La separación:

Authorization
→ Can the actor perform this operation?

Repository
→ Can the system retrieve/store this data?

Ambos pueden proporcionar defensa complementaria.

40. Repository and Policies

Las políticas de negocio no deben convertirse en filtros SQL arbitrarios.

Ejemplo:

TrainingPlanPolicy

debe permanecer en la capa apropiada.

41. Repository and Domain Events

El Repository normalmente persiste cambios.

El evento puede gestionarse mediante:

Application Service
+
Unit of Work
+
Outbox

Ejemplo:

Repository.save()
        ↓
Outbox.save()
        ↓
Commit
42. Transactional Outbox

Para garantizar consistencia:

BEGIN
 ├── Domain Changes
 └── Outbox Event
COMMIT

Después:

Outbox Worker
      ↓
Broker

Esto evita perder eventos después de un commit exitoso.

43. Repository Caching

Los Repositories pueden integrarse con cache cuando tenga sentido.

Application
    ↓
Repository
    ↓
Cache
    ↓
Database

Pero el cache no debe cambiar la semántica del dominio.

44. Cache Invalidation

Después de un cambio:

Update
 ↓
Persist
 ↓
Invalidate Cache

Debe existir una estrategia explícita.

45. Redis

Redis puede utilizarse como:

Cache
Session Store
Distributed Lock
Rate Limit Store
Temporary Data

pero no debe convertirse automáticamente en fuente de verdad para los datos transaccionales del dominio.

46. Source of Truth

Por defecto:

PostgreSQL
→ System of Record

mientras:

Redis
→ Acceleration / Temporary State

cuando corresponda.

47. Repository and Files

Para datos documentales o archivos:

FileRepository
ObjectStorageRepository
DocumentRepository

pueden abstraer:

S3
Local Storage
Azure Blob
Other Object Storage

El dominio no debería conocer el proveedor concreto.

48. Repository and External Data

Los datos externos pueden tener una abstracción diferente:

ExternalDataProvider

No todo acceso externo debe modelarse como Repository.

Regla:

Repository
→ Persistence of application/domain state

Integration Adapter
→ External system interaction
49. Repository vs Integration

Ejemplo:

UserRepository
→ EVOXA Users

PaymentGateway
→ Stripe / Payment Provider

No convertir:

Stripe

en:

PaymentRepository

si representa una integración externa.

50. Repository Interfaces

Las interfaces deben ser pequeñas y específicas.

Ejemplo:

interface TrainingPlanRepository:

    get_by_id(id)
    get_active_for_user(user_id)
    save(plan)

No:

interface GenericRepository:

    query_anything()
    execute_sql()
    update_any_table()
51. Repository Methods and Domain Language

Preferir:

get_active_plan()

sobre:

select_where_status_active()

La primera expresa el lenguaje de EVOXA.

52. Pagination

Las consultas grandes deben soportar:

Pagination
Cursor Pagination
Limits
Sorting
Filtering

según el caso de uso.

53. Cursor Pagination

Para datasets grandes:

GET page
   ↓
cursor
   ↓
next page

puede ser preferible a offsets gigantes.

54. Repository Filtering

Los filtros deben ser explícitos:

tenant_id
status
date_range
user_id
goal_id

y no permitir que el cliente construya SQL libremente.

55. Repository Aggregation

Para analytics:

SUM
COUNT
AVG
GROUP BY

pueden ejecutarse directamente en la base de datos cuando sea apropiado.

No siempre conviene cargar millones de registros al dominio.

56. Read Model Optimization

Ejemplo:

Progress Dashboard

puede requerir:

Aggregated Query

en lugar de:

Load every workout
Load every exercise
Load every metric
Calculate everything in Python
57. Bulk Operations

Repositories pueden ofrecer operaciones batch:

bulk_insert()
bulk_update()
bulk_delete()

cuando el caso de uso lo requiere.

Deben aplicarse con cuidado para mantener invariantes.

58. Repository and Domain Invariants

Una operación bulk que evita entidades puede saltarse reglas de dominio.

Por ello:

Bulk Operation

solo debe utilizarse cuando:

Business Rules
+
Consistency

estén correctamente consideradas.

59. Soft Delete

Cuando el negocio requiere conservar información:

deleted_at

puede utilizarse.

El Repository debe definir claramente:

Active records
Deleted records
Restore
Permanent delete
60. Audit Fields

Los modelos persistentes pueden incluir:

created_at
updated_at
created_by
updated_by
deleted_at
version

según las necesidades del dominio.

61. Repository and Audit

Los Repositories pueden proporcionar información técnica de persistencia.

Pero la auditoría de negocio puede requerir una capa separada:

Audit Service

Ejemplo:

Subscription changed
Role changed
Security policy changed
62. Repository Observability

Debe medirse:

query_duration
query_count
errors
timeouts
connection_pool_usage
cache_hits
cache_misses

sin exponer datos sensibles.

63. Slow Queries

Las consultas lentas deben poder identificarse mediante:

Tracing
Metrics
Database Monitoring
Query Logs
64. Connection Pooling

La infraestructura debe administrar:

Connection Pool
Max Connections
Timeouts
Recycle
Health Checks

El Application Layer no debería administrar conexiones manualmente.

65. Repository Resilience

Ante problemas de infraestructura:

Timeout
Connection Failure
Deadlock
Transient Error

pueden aplicarse estrategias apropiadas:

Retry
Backoff
Circuit Breaker
Fail Fast

según el tipo de operación.

66. Retry Safety

No todas las operaciones son seguras para retry.

Debe evaluarse:

Idempotency
Transaction State
Side Effects

antes de aplicar reintentos automáticos.

67. Repository Testing

Cada Repository debe tener:

Unit Tests
Integration Tests
Tenant Isolation Tests
Concurrency Tests
Mapping Tests
Migration Compatibility Tests
68. Repository Unit Tests

Los contracts pueden probarse con:

Fake Repository
In-Memory Repository
Mock Repository

cuando sea útil.

69. Repository Integration Tests

Las implementaciones PostgreSQL deben probarse contra una base real o entorno equivalente.

Application
 ↓
Repository
 ↓
PostgreSQL

Esto detecta problemas que mocks no encuentran.

70. Test Containers

EVOXA puede utilizar entornos aislados para:

PostgreSQL
Redis
Message Broker

durante integration testing.

71. Repository Contract Tests

Si existen varias implementaciones:

PostgresRepository
InMemoryRepository

ambas deben respetar el mismo contrato.

72. Repository Contract Example
Repository Contract:

save(entity)
get(id)
delete(id)
exists(id)

Todas las implementaciones deben mantener la misma semántica.

73. Repository Anti-Patterns

EVOXA debe evitar:

God Repository
Generic Everything Repository
Business Logic in Repository
HTTP Calls in Repository
Hidden Transactions
Cross-Tenant Queries
Raw SQL Everywhere
ORM Leakage
Unbounded Queries
Implicit Cache Semantics
74. God Repository

Incorrecto:

EvoxaRepository

que maneje:

Users
Training
Nutrition
Billing
AI
Security

Debe dividirse por bounded context.

75. Business Logic in Repository

Incorrecto:

WorkoutRepository
   ↓
if recovery < threshold:
    reduce_training()

El repository persiste.

La decisión pertenece al dominio.

76. HTTP in Repository

Incorrecto:

UserRepository
    ↓
HTTP GET external-service

Los repositories deben representar persistencia.

Las integraciones externas deben utilizar adapters/client interfaces.

77. Raw SQL

SQL directo puede utilizarse cuando aporta valor:

Complex Analytics
Performance Critical Query
PostGIS
Advanced PostgreSQL Features

pero debe permanecer encapsulado en Infrastructure.

78. PostGIS

Dado que EVOXA puede requerir funcionalidades geoespaciales en determinados módulos, operaciones como:

Distance
Radius
Geospatial Filtering
Spatial Aggregation

pueden utilizar capacidades de PostgreSQL/PostGIS.

El dominio recibe conceptos como:

Distance
Location
Area

y no expresiones SQL/PostGIS.

79. Repository Architecture

Estructura recomendada:

infrastructure/
└── persistence/
    ├── postgres/
    │   ├── models/
    │   ├── repositories/
    │   ├── mappers/
    │   ├── queries/
    │   └── migrations/
    ├── redis/
    └── object_storage/

Mientras:

domain/
└── repositories/
    ├── workout_repository.py
    ├── training_plan_repository.py
    └── progress_repository.py

contiene los contratos, si se adopta esa organización.

80. Repository Lifecycle
Define Aggregate Boundary
        ↓
Define Repository Contract
        ↓
Implement Infrastructure Adapter
        ↓
Implement Mapping
        ↓
Add Transactions
        ↓
Add Tenant Isolation
        ↓
Add Tests
        ↓
Add Observability
        ↓
Optimize
        ↓
Evolve
        ↓
Deprecate
81. Repository Documentation

Cada Repository debe documentar:

Purpose
Aggregate
Ownership
Methods
Queries
Transactions
Tenant Rules
Concurrency
Caching
Errors
Performance
Events
Tests
82. Repository Contract Example
Repository:
TrainingPlanRepository

Aggregate:
TrainingPlan

Operations:
get_by_id()
get_active_for_user()
save()
delete()

Tenant:
Required

Concurrency:
Optimistic locking

Transaction:
Managed by Unit of Work

Errors:
NotFound
ConcurrencyConflict
PersistenceConflict
83. Application + Repository

El flujo estándar:

Command
   ↓
Application Service
   ↓
Authorization
   ↓
Policy
   ↓
Repository
   ↓
Domain
   ↓
Repository
   ↓
Unit of Work
   ↓
Commit
84. Repository + Domain Service

Ejemplo:

CompleteWorkout
       │
       ▼
WorkoutRepository.get()
       │
       ▼
WorkoutCompletionService
       │
       ▼
WorkoutRepository.save()
       │
       ▼
ProgressRepository.save()

La orquestación permanece en Application Service.

85. Repository + Events

Flujo recomendado:

Application Service
       │
       ├── Repository.save()
       │
       └── Outbox.save()
                │
                ▼
             Commit
                │
                ▼
          Event Publisher
86. Repository + Multi-Tenancy

La cadena completa:

Request
   ↓
Tenant Context
   ↓
Application Service
   ↓
Repository
   ↓
Tenant Filter / RLS
   ↓
Database

Esto establece una defensa de múltiples niveles.

87. Repository + Security

Seguridad:

Authentication
       ↓
Authorization
       ↓
Application Service
       ↓
Tenant Isolation
       ↓
Repository
       ↓
Database Security

Ninguna capa individual debe considerarse suficiente para todos los controles.

88. Repository Architecture Principles

Los principios oficiales de EVOXA son:

1. Domain independence
2. Explicit contracts
3. Aggregate-oriented persistence
4. Dependency inversion
5. Transaction consistency
6. Tenant isolation
7. Concurrency awareness
8. Infrastructure encapsulation
9. Query optimization
10. Testability
11. Observability
12. Security by boundary
89. Definition of Done

E10 queda definido cuando EVOXA dispone de:

✓ Repository Definition
✓ Repository Responsibilities
✓ Repository Boundaries
✓ Repository Interfaces
✓ Repository Implementations
✓ Aggregate Persistence
✓ Aggregate Root Strategy
✓ Domain Mapping
✓ ORM Isolation
✓ PostgreSQL Strategy
✓ Query Repositories
✓ CQRS Compatibility
✓ Unit of Work
✓ Transaction Strategy
✓ Error Translation
✓ Concurrency Strategy
✓ Tenant Isolation
✓ Row-Level Security Strategy
✓ Security Controls
✓ Cache Strategy
✓ Redis Strategy
✓ File/Object Storage Strategy
✓ External Data Boundary
✓ Pagination
✓ Filtering
✓ Aggregations
✓ Bulk Operations
✓ Soft Delete
✓ Audit Fields
✓ Observability
✓ Connection Pooling
✓ Resilience
✓ Retry Safety
✓ Repository Testing
✓ Contract Testing
✓ Anti-Patterns
✓ Documentation
✓ Lifecycle
90. Arquitectura consolidada E07–E10

Con E10, la arquitectura de Engineering Specification queda:

                    EVOXA
                      │
                      ▼
               Interface / API
                      │
                      ▼
              Application Services
                      │
             ┌────────┴────────┐
             ▼                 ▼
       Domain Services       Queries
             │                 │
             ▼                 ▼
        Domain Model      Read Models
             │                 │
             ▼                 ▼
      Repository Interfaces
             │
═════════════╪════════════════════
       Persistence Boundary
═════════════╪════════════════════
             │
      Repository Adapters
             │
       ┌─────┼─────┐
       ▼     ▼     ▼
   PostgreSQL Redis Object Storage

La responsabilidad queda claramente separada:

E07
→ Service Architecture

E08
→ Domain Business Operations

E09
→ Use Case Orchestration

E10
→ Persistence Abstraction

Y el siguiente paso natural del Engineering Specification es:

E11 — EVOXA Integration Architecture

donde definiremos la frontera entre EVOXA y sistemas externos: APIs, webhooks, proveedores, adapters, clients, eventos de integración, retries, circuit breakers, contratos externos y patrones de integración.
