E70 — EVOXA MODULE & PACKAGE ARCHITECTURE
1. Propósito

E70 convierte la estructura física definida en E69 — Project / Repository Architecture en una arquitectura concreta de módulos, paquetes, bounded contexts, dependencias y ownership.

La secuencia ahora es:

E67 — Master Application Blueprint
          ↓
E68 — Technical Stack & Platform
          ↓
E69 — Project / Repository Architecture
          ↓
E70 — Module & Package Architecture
          ↓
E71 — Database Architecture
          ↓
E72 — API Architecture
          ↓
E73 — Testing Architecture
          ↓
E74 — Deployment Architecture
          ↓
       IMPLEMENTATION

La pregunta central de E70 es:

¿Cuáles son exactamente los módulos que forman EVOXA y cómo pueden comunicarse entre ellos sin destruir los límites arquitectónicos?

2. Principio fundamental

EVOXA no será diseñado como una colección arbitraria de carpetas.

La unidad arquitectónica fundamental será:

Bounded Context
      ↓
Module
      ↓
Package
      ↓
Component
      ↓
Implementation

Por tanto:

Folder ≠ Module
Module ≠ Service
Service ≠ Deployment Unit

Esta distinción será fundamental durante toda la evolución de EVOXA.

3. Arquitectura modular general

La primera división de EVOXA será:

EVOXA
│
├── Core
│
├── Identity & Access
│
├── Domain
│
├── Application
│
├── Integration
│
├── Messaging
│
├── Event Processing
│
├── Workflow
│
├── Job & Task Processing
│
├── Scheduling
│
├── Data
│
├── Intelligence
│
├── Governance
│
└── Runtime Platform

Pero esto debe refinarse.

No todos estos elementos son bounded contexts independientes.

4. Clasificación de componentes

EVOXA tendrá cuatro grandes categorías:

1. Core Modules
2. Business / Domain Modules
3. Platform Modules
4. Cross-Cutting Modules
Core

Capacidades fundamentales.

Domain

Conocimiento y comportamiento de EVOXA.

Platform

Capacidades técnicas reutilizables.

Cross-Cutting

Capacidades transversales.

5. Core Modules
core/
├── primitives
├── identity
├── errors
├── result
├── contracts
├── commands
├── queries
├── events
├── policies
└── types

Core debe mantenerse deliberadamente pequeño.

6. Core Primitives

Contendrá conceptos básicos como:

EntityId
CorrelationId
CausationId
Timestamp
Version
Status
Result
Error

Estos objetos no deben contener lógica específica de un dominio concreto.

7. Identity Module

Identity representa identidades dentro del sistema.

Conceptualmente:

Identity
├── User
├── Service Identity
├── Machine Identity
└── Principal

Debe distinguirse de Authentication.

8. Authentication vs Identity
Identity
"What is this actor?"

Authentication
"How do we know who they are?"

La implementación concreta de authentication puede vivir en Infrastructure.

9. Authorization Module

Authorization determina:

Can subject perform action X
on resource Y
under context Z?

Puede evolucionar desde:

RBAC

hacia:

ABAC
Policy-Based Authorization
Resource-Level Authorization
10. Domain Architecture

El dominio debe organizarse por capacidad y contexto, no por tipo técnico.

Evitar:

domain/
├── entities/
├── services/
├── repositories/
└── events/

como única organización.

Es mejor:

domain/
├── resource/
├── execution/
├── workflow/
├── policy/
└── governance/

y dentro de cada módulo:

entities
value_objects
rules
services
events
repositories
11. Domain Modules

La propuesta inicial de EVOXA:

domain/
├── resource/
├── execution/
├── workflow/
├── policy/
├── decision/
├── action/
├── governance/
└── data/

Estos son candidatos arquitectónicos, y algunos podrán fusionarse o dividirse cuando el dominio real lo requiera.

12. Resource Domain

Resource representa recursos administrados por EVOXA.

resource/
├── entities/
├── value_objects/
├── rules/
├── services/
├── events/
└── repositories/

Responsabilidades:

Resource Identity
Resource State
Resource Lifecycle
Resource Constraints
Resource Ownership
13. Execution Domain

Execution representa la ejecución de acciones.

execution/
├── entities/
├── value_objects/
├── rules/
├── services/
├── events/
└── repositories/

Conceptualmente:

Action
 ↓
Execution
 ↓
Outcome
14. Workflow Domain

Workflow representa procesos coordinados.

workflow/
├── entities/
├── value_objects/
├── steps/
├── rules/
├── events/
└── repositories/

Debe mantenerse separado de Job Processing.

15. Workflow vs Job

Un Workflow representa:

qué proceso debe ejecutarse.

Un Job representa:

trabajo concreto pendiente de ejecución.

Por tanto:

Workflow
   ↓
Job
   ↓
Worker
16. Policy Domain

Policy representa reglas formales de comportamiento.

policy/
├── entities/
├── definitions/
├── rules/
├── evaluation/
├── decisions/
└── repositories/

Debe distinguir:

Policy Definition

de:

Policy Evaluation
17. Decision Domain

Decision representa resultados de evaluación.

decision/
├── entities/
├── inputs/
├── evaluation/
├── outcomes/
└── events/

Modelo:

Inputs
  ↓
Rules / Policies
  ↓
Decision
  ↓
Outcome
18. Action Domain

Action representa una intención ejecutable derivada de una decisión o workflow.

action/
├── entities/
├── definitions/
├── parameters/
├── validation/
└── events/

Separación:

Decision
   ↓
Action
   ↓
Execution
19. Data Domain

El dominio de datos representa conceptos propios de EVOXA relacionados con la gestión de datos.

data/
├── datasets/
├── data_products/
├── classifications/
├── lineage/
├── lifecycle/
└── ownership/

Esto conecta directamente con los capítulos E45–E58.

20. Governance Domain

Governance tendrá una frontera explícita.

governance/
├── policy/
├── control/
├── compliance/
├── audit/
├── evidence/
└── assurance/

Pero no todo Governance debe convertirse en Domain Entity.

Algunas capacidades son claramente infrastructure/application concerns.

21. Application Modules

La Application Layer seguirá los mismos límites conceptuales del dominio.

application/
├── resource/
├── execution/
├── workflow/
├── policy/
├── decision/
├── action/
├── data/
└── governance/

Esto permite:

Domain Context
      ↕
Application Context

sin crear una capa application gigantesca.

22. Application Module Structure

Cada módulo podrá tener:

resource/
├── commands/
├── queries/
├── handlers/
├── services/
├── dto/
└── workflows/

No todos los módulos necesitarán todas estas carpetas.

23. Infrastructure Modules

Infrastructure se organizará por capacidad técnica.

infrastructure/
├── persistence/
├── messaging/
├── cache/
├── search/
├── storage/
├── integrations/
├── serialization/
├── configuration/
├── observability/
└── runtime/
24. Persistence Modules

Persistence puede organizarse:

persistence/
├── postgres/
│   ├── connection/
│   ├── models/
│   ├── repositories/
│   ├── mappers/
│   └── transactions/
└── migrations/

Las implementaciones específicas de cada bounded context permanecerán separadas.

25. Messaging Modules
messaging/
├── nats/
├── publishers/
├── consumers/
├── serializers/
├── delivery/
└── dead_letter/

Pero debe existir una separación conceptual entre:

Message Contract

y:

NATS Implementation
26. Cache Modules
cache/
├── redis/
├── policies/
├── serialization/
└── locks/

El dominio nunca importará Redis directamente.

27. Search Modules
search/
├── opensearch/
├── indexes/
├── projections/
└── queries/

La arquitectura será:

Domain State
     ↓
Projection
     ↓
Search Index
28. Storage Modules
storage/
├── object_storage/
├── s3/
├── artifacts/
└── archives/

El dominio utilizará una abstracción:

ObjectStorage

no directamente el SDK del proveedor.

29. Integration Modules

Las integraciones externas estarán aisladas:

integrations/
├── identity/
├── notifications/
├── external_apis/
├── data_sources/
└── providers/

Cada integración tendrá:

Port
Adapter
Mapper
Client
Error Mapping
30. Anti-Corruption Layer

Cuando un proveedor tenga un modelo diferente:

External System
       ↓
External Adapter
       ↓
Mapper / ACL
       ↓
EVOXA Model

Nunca:

External API Object
       ↓
Domain
31. Serialization Modules
serialization/
├── json/
├── events/
├── messages/
└── external/

Debe evitarse que los DTO externos se conviertan automáticamente en entidades de dominio.

32. Configuration Modules
configuration/
├── settings/
├── loaders/
├── validation/
└── sources/

Flujo:

Environment
   ↓
Loader
   ↓
Validation
   ↓
Settings
   ↓
Application
33. Runtime Modules
runtime/
├── api/
├── workers/
├── scheduler/
├── consumers/
└── lifecycle/

Cada runtime debe arrancar componentes concretos sin introducir lógica de dominio.

34. Governance Modules

La estructura propuesta:

governance/
├── policies/
├── controls/
├── compliance/
├── audit/
├── evidence/
├── monitoring/
└── assurance/
35. Policy Module

Debe existir una única abstracción conceptual para policy.

Policy
   ↓
Evaluation
   ↓
Decision

Evitar que cada módulo invente su propio motor de policies.

36. Control Module

Controls representan controles verificables:

Control
   ↓
Evaluation
   ↓
Result
   ↓
Evidence
37. Compliance Module

Compliance consume:

Policies
Controls
Evidence
Audit Data

y produce:

Compliance State
Findings
Reports
38. Audit Module

Audit captura hechos relevantes:

Actor
Action
Resource
Timestamp
Result
Context
Evidence

Debe poder relacionarse con:

correlationId
causationId
39. Evidence Module

Evidence representa evidencia de cumplimiento o ejecución.

Evidence
{
    evidenceId
    source
    type
    timestamp
    integrity
    retention
}
40. Monitoring Module

Monitoring observa:

Controls
Policies
Data Access
Security Signals
Runtime Signals

No debe confundirse con observability operacional.

41. Assurance Module

Assurance agrega:

Monitoring
Evidence
Controls
Audit

para determinar:

Control Effectiveness
Compliance Confidence
Risk Signals
42. Package-Level Architecture

Cada módulo deberá tener una estructura interna consistente.

Ejemplo:

resource/
├── domain/
│   ├── entities/
│   ├── value_objects/
│   ├── rules/
│   ├── events/
│   └── repositories/
│
├── application/
│   ├── commands/
│   ├── queries/
│   ├── handlers/
│   └── dto/
│
└── infrastructure/
    ├── persistence/
    └── integrations/

Sin embargo, E70 debe decidir entre dos estilos.

43. Style A — Layer-First
src/evoxa/
├── domain/
├── application/
├── infrastructure/
└── interfaces/

y dentro:

resource/
execution/
workflow/

Ventaja:

Clear global layers

Desventaja:

Feature code becomes distributed
44. Style B — Module-First
src/evoxa/
├── resource/
├── execution/
├── workflow/
├── policy/
└── governance/

y dentro de cada módulo:

domain/
application/
infrastructure/
interfaces/

Ventaja:

Feature cohesion

Desventaja:

Global architectural layers less obvious
45. Decisión para EVOXA

Para EVOXA recomiendo un enfoque Module-First dentro de los bounded contexts, manteniendo explícitas las capas internas.

Es decir:

src/evoxa/
├── core/
├── modules/
│   ├── resource/
│   ├── execution/
│   ├── workflow/
│   ├── policy/
│   ├── decision/
│   ├── action/
│   └── data/
│
├── governance/
├── platform/
└── interfaces/

Esto prepara mejor EVOXA para crecer sin crear una capa domain/ gigantesca.

46. Estructura recomendada

La estructura refinada:

src/evoxa/
│
├── core/
│
├── modules/
│   ├── resource/
│   ├── execution/
│   ├── workflow/
│   ├── policy/
│   ├── decision/
│   ├── action/
│   └── data/
│
├── governance/
│
├── platform/
│   ├── persistence/
│   ├── messaging/
│   ├── cache/
│   ├── search/
│   ├── storage/
│   ├── integrations/
│   ├── serialization/
│   ├── configuration/
│   ├── observability/
│   └── runtime/
│
└── interfaces/
    ├── http/
    ├── messaging/
    ├── cli/
    └── webhooks/

Esto refina E69, no lo contradice.

47. Module Internal Structure

Cada módulo de negocio podrá seguir:

module/
├── domain/
├── application/
├── infrastructure/
└── interfaces/

Ejemplo:

modules/resource/
├── domain/
│   ├── entities/
│   ├── value_objects/
│   ├── rules/
│   ├── events/
│   └── repositories/
│
├── application/
│   ├── commands/
│   ├── queries/
│   ├── handlers/
│   └── dto/
│
├── infrastructure/
│   └── persistence/
│
└── interfaces/
    └── http/
48. Why Module-First

Porque permite localizar una funcionalidad completa:

resource/

en lugar de:

domain/resource
application/resource
infrastructure/resource
interfaces/resource

dispersos por todo el árbol.

Para un sistema del tamaño esperado de EVOXA, esto mejora:

Discoverability
Ownership
Refactoring
Testing
Future Extraction
49. Module Public API

Cada módulo tendrá una superficie pública controlada.

Conceptualmente:

resource/
├── public/
└── internal/

No necesariamente como carpetas físicas.

Python permite establecer esta convención mediante:

__init__.py
Explicit Imports
__all__
Architecture Tests
50. Public vs Internal

Otros módulos podrán utilizar:

resource.public

pero no:

resource.internal.persistence.models

Esto es una regla arquitectónica, no una garantía de seguridad.

51. Module Contracts

Cada módulo debe declarar:

Public Commands
Public Queries
Public Events
Public Services
Public Types
52. Module Dependencies

Ejemplo:

workflow
   ↓
action
   ↓
execution
   ↓
resource

Pero:

resource
   ✕
workflow

si no existe una necesidad arquitectónica real.

Esto evita ciclos.

53. Dependency Graph

Inicialmente:

             ┌──────────┐
             │ Workflow │
             └────┬─────┘
                  ↓
             ┌──────────┐
             │  Action  │
             └────┬─────┘
                  ↓
             ┌──────────┐
             │Execution │
             └────┬─────┘
                  ↓
             ┌──────────┐
             │ Resource │
             └──────────┘

Mientras:

Policy
  ↓
Decision
  ↓
Action
54. Policy / Decision Relationship
Policy
   ↓
Evaluation
   ↓
Decision

No:

Policy
   ↔
Decision

si esto crea un ciclo conceptual.

55. Data Module

El módulo data debe mantenerse separado porque las capacidades de datos atraviesan múltiples dominios.

data/
├── dataset/
├── data_product/
├── classification/
├── lineage/
└── lifecycle/

Otros módulos pueden consumir contratos de Data.

56. Data Ownership

Cada entidad debe tener un propietario.

Ejemplo:

Resource → Resource Module
Workflow → Workflow Module
Execution → Execution Module
Policy → Policy Module
Decision → Decision Module
Dataset → Data Module
Control → Governance
57. Shared Entities

Debe evitarse:

shared/entities/

con entidades utilizadas por todo el sistema.

En su lugar:

Identity
Value Object
Contract
Reference

cuando realmente sean compartidos.

58. Shared Kernel

EVOXA tendrá un Shared Kernel mínimo en core.

Puede contener:

IDs
Errors
Result
Correlation
Common Contracts
Primitive Value Objects

No:

Business Entities
59. Domain Events

Cada módulo será dueño de sus eventos.

Ejemplo:

resource/
└── domain/
    └── events/
        └── resource_created.py

Otros módulos consumen el contrato público.

60. Event Ownership

Regla:

El módulo que posee el estado es dueño del evento que anuncia cambios sobre ese estado.

Por ejemplo:

Resource
    ↓
ResourceCreated

es responsabilidad de Resource.

61. Commands Ownership

El módulo que posee la operación debe ser propietario del Command.

CreateResource

pertenece a:

Resource

No a un módulo commands/ global.

62. Queries Ownership

Igualmente:

GetResource
SearchResources

pertenecen al contexto de Resource.

Pero las consultas cross-context pueden tener un read model específico.

63. Cross-Module Queries

Evitar:

Workflow
 ↓
direct SQL
 ↓
Resource tables

Preferir:

Workflow
 ↓
Resource Contract
 ↓
Resource

o:

Workflow
 ↓
Read Model

cuando sea apropiado.

64. Cross-Module Transactions

Evitar transacciones que atraviesen arbitrariamente múltiples módulos.

Preferir:

Module A
  ↓
Event
  ↓
Module B

cuando la consistencia eventual sea aceptable.

65. Strong Consistency

Cuando sea indispensable consistencia fuerte:

Application Service
   ↓
Explicit Transaction Boundary

pero debe ser una decisión explícita.

66. Circular Dependencies

Están prohibidas:

A → B → A

y especialmente:

A → B → C → A
67. Dependency Inversion

Cuando A necesita capacidades de B:

A
 ↓
Port / Contract
 ↑
B Adapter

en lugar de acoplamiento directo.

68. Infrastructure Dependency

Los módulos de negocio no deben conocer:

PostgreSQL
Redis
NATS
OpenSearch
S3

directamente.

Deben utilizar:

Repository
MessageBus
Cache
SearchPort
ObjectStorage
69. Platform Layer

La carpeta:

platform/

será la implementación compartida de capacidades técnicas.

platform/
├── persistence/
├── messaging/
├── cache/
├── search/
├── storage/
├── integrations/
├── serialization/
├── configuration/
├── observability/
└── runtime/
70. Platform vs Module Infrastructure

No todo infrastructure debe ser global.

Por ejemplo:

Resource Repository

pertenece al módulo Resource.

Mientras:

PostgreSQL Connection Pool

pertenece a Platform.

Por tanto:

Module
   ↓
Repository Interface
   ↓
Module Adapter
   ↓
Platform PostgreSQL
71. Interfaces

La API HTTP puede descubrir módulos mediante:

interfaces/http/
├── resource/
├── workflow/
├── execution/
├── policy/
└── governance/

pero la lógica permanece en cada módulo.

72. HTTP Boundary
HTTP Route
   ↓
Application Command / Query
   ↓
Module

Nunca:

HTTP Route
   ↓
Repository
73. Messaging Boundary
Message Consumer
   ↓
Application Command
   ↓
Module

Esto permite que HTTP y Messaging utilicen el mismo caso de uso.

HTTP ─────┐
          ├──→ Application
Message ──┘
74. CLI Boundary

El CLI también será un adapter:

CLI
 ↓
Application
 ↓
Module

No debe contener lógica de negocio.

75. Webhook Boundary
Webhook
   ↓
Integration Adapter
   ↓
Application

con validación de:

Signature
Identity
Payload
Replay
Idempotency
76. Module Template

Cada módulo nuevo deberá seguir un patrón:

module/
│
├── domain/
│   ├── entities/
│   ├── value_objects/
│   ├── rules/
│   ├── events/
│   └── repositories/
│
├── application/
│   ├── commands/
│   ├── queries/
│   ├── handlers/
│   └── dto/
│
├── infrastructure/
│   └── persistence/
│
└── interfaces/

Pero se crearán solamente los directorios necesarios.

77. Empty Package Rule

No crear:

commands/
queries/
events/
services/

si el módulo todavía no necesita alguno.

La arquitectura debe reflejar necesidades reales.

78. Module Lifecycle

Un módulo pasa por:

Proposed
   ↓
Defined
   ↓
Implemented
   ↓
Integrated
   ↓
Operational
   ↓
Mature
79. New Module Process

Para introducir un módulo:

Need
 ↓
Bounded Context Analysis
 ↓
Ownership
 ↓
Public Contract
 ↓
Dependencies
 ↓
ADR
 ↓
Implementation
80. Module Extraction

Si un módulo necesita convertirse en servicio:

Module
 ↓
Contract Stabilization
 ↓
Infrastructure Isolation
 ↓
Deployment Isolation
 ↓
Service

La arquitectura Module-First facilita esta evolución.

81. Future Service Topology

Podríamos evolucionar:

                EVOXA
                  │
       ┌──────────┼──────────┐
       ▼          ▼          ▼
    Resource   Workflow   Governance
       │          │          │
       └──────────┼──────────┘
                  │
              Messaging

hacia:

Resource Service
Workflow Service
Governance Service
Execution Service

sin modificar radicalmente los contratos internos.

82. Architecture Tests

E70 exige tests que comprueben:

Module A cannot import Module B internal package
Module cannot import platform implementation directly
Domain cannot import HTTP
Domain cannot import ORM
No circular dependencies
Public contracts remain stable
83. Import Rules

Una regla conceptual:

PUBLIC
   ↑
PRIVATE

Los consumidores solamente deben utilizar APIs públicas.

84. Module Manifest

Podemos establecer opcionalmente un descriptor conceptual:

module.yaml

con:

module
owner
public_contracts
dependencies
events
data_ownership

No es obligatorio implementarlo en la primera versión; puede convertirse en una capacidad posterior de Governance.

85. Package Naming

Convención:

snake_case

Ejemplos:

data_product
event_processing
runtime_policy
access_control

Evitar nombres ambiguos.

86. Class Naming

Convención estándar:

PascalCase

Ejemplos:

Resource
ResourceRepository
CreateResource
CreateResourceHandler
ResourceCreated
87. Command Naming
Verb + Noun

Ejemplo:

CreateResource
UpdatePolicy
ExecuteWorkflow
ArchiveDataset
88. Event Naming
Noun + Past Tense

Ejemplo:

ResourceCreated
WorkflowStarted
ExecutionCompleted
PolicyEvaluated

El evento describe un hecho pasado.

89. Query Naming
GetResource
ListResources
SearchResources
FindResource
90. Repository Naming
ResourceRepository
WorkflowRepository
ExecutionRepository

No:

DatabaseManager
GenericRepository
UniversalRepository

como sustitutos de ownership.

91. Service Naming

Los servicios deben indicar intención.

Preferir:

WorkflowOrchestrator
PolicyEvaluator
ExecutionCoordinator

sobre:

WorkflowManager
PolicyManager
SystemService

cuando estos no expresen responsabilidad concreta.

92. Domain Service Rule

Un Domain Service solamente existe cuando la lógica:

Is domain-specific

pero:

Does not naturally belong to one entity

No utilizar Domain Services para esconder application orchestration.

93. Application Service Rule

Application Services coordinan:

Use Case
Transactions
Repositories
Domain Calls
Events

No deberían convertirse en un segundo dominio.

94. Infrastructure Service Rule

Infrastructure Services implementan capacidades técnicas:

NATS Publisher
Redis Cache
S3 Storage
OpenSearch Client
95. Interface Service Rule

Interface components traducen:

HTTP
Message
CLI
Webhook

a contratos de aplicación.

96. Four-Service Distinction

EVOXA debe distinguir:

Domain Service
Application Service
Infrastructure Service
Interface Adapter

Esto evitará una gran cantidad de confusión arquitectónica.

97. Data Access Rule

Solo el módulo propietario puede escribir directamente su estado transaccional.

Resource
   ↓
Resource Repository
   ↓
Resource Data

Otro módulo:

Workflow
   ✕
Resource DB
98. Read Access

Puede existir:

Read Model

compartido cuando sea necesario.

Ejemplo:

Workflow Read Model

en lugar de consultar directamente múltiples tablas internas.

99. Search Ownership

El módulo propietario del dato debe controlar su indexación:

Resource
 ↓
Resource Projection
 ↓
Search Index
100. Cache Ownership

El cache debe estar asociado a la capacidad que lo utiliza.

Evitar un:

global_cache.py

que almacene cualquier cosa.

101. Event Ownership

Cada event debe tener:

Owner
Schema
Version
Producer
Consumers
102. Module Communication Matrix

La matriz conceptual inicial:

Módulo	Puede consumir
Resource	Core
Execution	Core, Resource
Workflow	Core, Action, Execution
Policy	Core
Decision	Core, Policy
Action	Core, Decision
Data	Core
Governance	Core, Policies, Data, Audit
Interfaces	Application
Platform	Contracts / Ports

Debe revisarse cuando definamos los dominios reales.

103. Core Dependency Graph
Core
 ↑
 ├── Resource
 ├── Execution
 ├── Workflow
 ├── Policy
 ├── Decision
 ├── Action
 └── Data

Core no depende de ellos.

104. Governance Dependency Graph

Governance consume información de:

Domain
Application
Data
Runtime
Security
Audit

pero no debe convertirse en una dependencia obligatoria de todas las operaciones internas.

105. Governance as Side Plane

Conceptualmente:

              EVOXA
                 │
       ┌─────────┼─────────┐
       ▼         ▼         ▼
    Domain   Application  Runtime
       │         │         │
       └─────────┼─────────┘
                 │
          Governance Plane
106. Security as Cross-Cutting

Security no debe ser solamente un módulo:

security/

Debe atravesar:

Interfaces
Application
Domain Policies
Infrastructure
Runtime
Audit
107. Observability as Cross-Cutting

Igualmente:

Observability

debe poder instrumentar:

HTTP
Application
Database
Messaging
Workers
External Calls
108. Audit as Cross-Cutting

Audit también cruza módulos:

Command
Decision
Action
Execution
Data Access
Governance

pero su implementación debe estar centralizada para mantener consistencia.

109. Module Boundary Checklist

Antes de crear un módulo:

Does it own a concept?
Does it have clear responsibility?
Does it have lifecycle?
Does it own data?
Does it expose contracts?
Can its dependencies be defined?
Can it be tested independently?

Si la respuesta es no, quizá no sea un módulo real.

110. Anti-Patterns

No permitir:

god_module
god_service
god_repository
global_utils
shared_entities
cross_module_database_access
circular_dependencies
framework_in_domain
111. God Module

Un módulo que contiene:

Everything

es una violación arquitectónica.

Si un módulo crece demasiado:

Analyze Boundaries
      ↓
Split Context
112. God Service

Evitar:

EvoxaService

con cientos de métodos.

Cada servicio debe tener una responsabilidad coherente.

113. God Repository

Evitar:

GenericRepository

que conoce todas las entidades.

Cada bounded context controla sus repositorios.

114. Global Utilities

Evitar:

utils/
helpers/
common/
misc/

como destinos genéricos.

115. Architecture Enforcement

La estructura de E70 deberá ser comprobada automáticamente.

Conceptualmente:

CI
 ↓
Architecture Test
 ↓
Import Graph
 ↓
Validate Rules
116. E70 Target Tree

La estructura objetivo queda:

src/evoxa/
│
├── core/
│   ├── contracts/
│   ├── errors/
│   ├── events/
│   ├── commands/
│   ├── queries/
│   ├── policies/
│   ├── identifiers/
│   └── result/
│
├── modules/
│   ├── resource/
│   │   ├── domain/
│   │   ├── application/
│   │   ├── infrastructure/
│   │   └── interfaces/
│   │
│   ├── execution/
│   ├── workflow/
│   ├── policy/
│   ├── decision/
│   ├── action/
│   └── data/
│
├── governance/
│   ├── policies/
│   ├── controls/
│   ├── compliance/
│   ├── audit/
│   ├── evidence/
│   ├── monitoring/
│   └── assurance/
│
├── platform/
│   ├── persistence/
│   ├── messaging/
│   ├── cache/
│   ├── search/
│   ├── storage/
│   ├── integrations/
│   ├── serialization/
│   ├── configuration/
│   ├── observability/
│   └── runtime/
│
└── interfaces/
    ├── http/
    ├── messaging/
    ├── cli/
    └── webhooks/
117. What E70 Establishes

E70 establece oficialmente:

✓ Module boundaries
✓ Package boundaries
✓ Bounded-context candidates
✓ Domain ownership
✓ Data ownership
✓ Public contracts
✓ Dependency direction
✓ Module communication
✓ Internal package structure
✓ Naming conventions
✓ Infrastructure boundaries
✓ Governance boundaries
✓ Cross-cutting responsibilities
✓ Architecture enforcement
118. What E70 Does NOT Establish

Todavía no fijamos definitivamente:

❌ Database schema
❌ Tables
❌ Indexes
❌ API endpoints
❌ OpenAPI contracts
❌ Deployment topology
❌ Kubernetes
❌ Production infrastructure

Eso llegará en:

E71
E72
E74
119. Relación E69 → E70

E69 decía:

src/evoxa/
├── core/
├── domain/
├── application/
├── infrastructure/
├── interfaces/
└── governance/

E70 refina esa estructura hacia:

src/evoxa/
├── core/
├── modules/
│   ├── resource/
│   ├── execution/
│   ├── workflow/
│   ├── policy/
│   ├── decision/
│   ├── action/
│   └── data/
├── governance/
├── platform/
└── interfaces/

La razón es que EVOXA debe crecer alrededor de sus capacidades de negocio, no alrededor de carpetas técnicas.

120. Decisión arquitectónica principal

La decisión de E70 es:

EVOXA utilizará una arquitectura modular orientada a bounded contexts, organizada Module-First, con capas internas Domain → Application → Infrastructure → Interfaces, un Core mínimo compartido, una Platform técnica común y Governance como plano transversal.

La representación final:

                         EVOXA
                           │
             ┌─────────────┼─────────────┐
             │             │             │
             ▼             ▼             ▼
           CORE         MODULES       GOVERNANCE
                           │
          ┌────────────────┼────────────────┐
          ▼                ▼                ▼
       Resource         Workflow         Execution
          │                │                │
          ├────────────┬───┴────┬───────────┤
          ▼            ▼        ▼           ▼
       Policy       Decision   Action      Data
                           │
                           ▼
                       PLATFORM
                           │
        ┌──────────────────┼──────────────────┐
        ▼                  ▼                  ▼
    Persistence         Messaging          Storage
        │                  │                  │
        └──────────────────┼──────────────────┘
                           ▼
                       INTERFACES
121. El siguiente paso: E71

Ahora ya sabemos:

E67 → qué construimos
E68 → con qué tecnologías
E69 → cómo se organiza el repositorio
E70 → cuáles son los módulos y sus fronteras

El siguiente paso es E71 — EVOXA DATABASE ARCHITECTURE.

Ahí pasaremos de los módulos a sus datos:

Module
   ↓
Aggregate
   ↓
Entity
   ↓
Persistence Model
   ↓
Table
   ↓
Column
   ↓
Constraint
   ↓
Index

y definiremos además:

PostgreSQL
Schema Strategy
Data Ownership
Transactions
Keys
Relationships
Indexes
Audit Tables
Outbox
Read Models
Migrations
Data Lifecycle

E71 será especialmente importante porque, por primera vez, podremos diseñar la estructura concreta de persistencia que posteriormente se convertirá en las primeras migrations y tablas reales de EVOXA.
