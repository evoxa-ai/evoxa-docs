E67 — EVOXA MASTER APPLICATION BLUEPRINT
1. Propósito

E67 convierte toda la arquitectura conceptual desarrollada hasta E66 en un blueprint técnico único para construir EVOXA.

La función de E67 no es todavía implementar código.

Su función es responder:

¿Qué aplicación vamos a construir, qué componentes la forman, cómo se relacionan, qué responsabilidades tiene cada uno y en qué orden deben implementarse?

Por tanto:

E01–E66
   ↓
Architectural Knowledge
   ↓
E67 — MASTER APPLICATION BLUEPRINT
   ↓
Implementable System Design
   ↓
E68+
   ↓
Implementation
2. Objetivo principal

E67 debe eliminar la distancia entre:

ARCHITECTURE

y:

SOFTWARE SYSTEM

Es decir:

Business Intent
       ↓
Architecture
       ↓
Components
       ↓
Modules
       ↓
Interfaces
       ↓
Data
       ↓
Runtime
       ↓
Code
3. Regla fundamental

E67 debe convertirse en el documento de referencia principal para la implementación de EVOXA.

Esto significa que cuando posteriormente aparezca una decisión como:

¿Dónde vive esta lógica?
¿Quién es dueño de esta entidad?
¿Quién publica este evento?
¿Quién accede a este repositorio?
¿Dónde se aplica esta policy?
¿Quién ejecuta este workflow?

la respuesta debe poder encontrarse en el blueprint.

4. Qué NO es E67

E67 no es:

❌ Código
❌ Base de datos definitiva
❌ API specification completa
❌ Deployment guide
❌ Manual de DevOps
❌ Test plan detallado

Es la estructura maestra que conecta todos esos elementos.

5. EVOXA como sistema

La aplicación puede modelarse inicialmente como:

                         EVOXA
                           │
        ┌──────────────────┼──────────────────┐
        │                  │                  │
        ▼                  ▼                  ▼
      DOMAIN          APPLICATION       GOVERNANCE
        │                  │                  │
        └──────────────┬───┴──────────────────┘
                       ▼
                  INFRASTRUCTURE
                       │
          ┌────────────┼────────────┐
          ▼            ▼            ▼
       DATA         MESSAGING      EXTERNAL
                                    SYSTEMS
                       │
                       ▼
                    RUNTIME
6. Architectural Layers

La arquitectura maestra queda organizada en capas.

┌─────────────────────────────────────────────┐
│                 INTERFACES                   │
│ API / UI / CLI / Events / Integrations      │
├─────────────────────────────────────────────┤
│              APPLICATION                    │
│ Use Cases / Services / Commands / Queries   │
├─────────────────────────────────────────────┤
│                 DOMAIN                      │
│ Entities / Rules / Policies / Decisions     │
├─────────────────────────────────────────────┤
│              INFRASTRUCTURE                 │
│ DB / Messaging / Cache / External Services  │
├─────────────────────────────────────────────┤
│                  RUNTIME                    │
│ Execution / Jobs / Scheduling / Resources   │
└─────────────────────────────────────────────┘

Governance y security actúan transversalmente.

7. Domain Layer

El Domain representa el conocimiento propio de EVOXA.

Debe contener:

Entities
Aggregates
Value Objects
Domain Services
Domain Events
Business Rules
Domain Policies
Domain Decisions

La regla:

El dominio no debe depender de detalles técnicos de infraestructura.

8. Application Layer

La Application Layer coordina casos de uso.

Application
├── Commands
├── Queries
├── Application Services
├── Handlers
├── DTOs
├── Transactions
└── Orchestration

Responsabilidad:

Receive Request
      ↓
Validate
      ↓
Load Domain
      ↓
Execute Use Case
      ↓
Persist
      ↓
Publish Effects
      ↓
Return Result
9. Infrastructure Layer

Infrastructure implementa capacidades técnicas.

Infrastructure
├── Persistence
├── Repositories
├── Messaging
├── Event Store
├── Cache
├── Search
├── Serialization
├── External Integrations
├── File Storage
└── Runtime Adapters
10. Interfaces Layer

Interfaces representa las puertas de entrada y salida.

Interfaces
├── HTTP API
├── REST
├── GraphQL (si aplica)
├── CLI
├── Event Consumers
├── Message Consumers
├── Webhooks
└── Administrative Interfaces

No debe contener reglas de negocio profundas.

11. EVOXA Core

El núcleo de EVOXA debe permanecer pequeño y estable.

EVOXA CORE
│
├── Identity
├── Domain Primitives
├── Result / Error Model
├── Contracts
├── Policies
├── Events
├── Commands
└── Common Infrastructure Abstractions

La regla es:

Core debe contener abstracciones realmente compartidas, no convertirse en un contenedor de utilidades arbitrarias.

12. Component Model

E67 establece un modelo de componentes:

Component
{
    componentId
    responsibility
    inputs
    outputs
    dependencies
    contracts
    owner
}

Cada componente debe tener una responsabilidad identificable.

13. Module Model

Un módulo agrupa componentes relacionados.

Module
{
    moduleId
    purpose
    boundedContext
    components
    publicContracts
    dependencies
}
14. Bounded Contexts

No todo EVOXA debe formar un único modelo conceptual.

Los dominios deben separarse cuando tengan:

Different Language
Different Ownership
Different Rules
Different Lifecycle
Different Data Model
15. Context Map

La arquitectura debe documentar relaciones:

Context A
   │
   │ contract
   ▼
Context B

Tipos posibles:

Provider
Consumer
Upstream
Downstream
Published Language
Anti-Corruption Layer
16. Application Components

El mapa inicial:

                    APPLICATION
                         │
        ┌────────────────┼────────────────┐
        ▼                ▼                ▼
     Commands         Queries          Workflows
        │                │                │
        ▼                ▼                ▼
 Application        Read Services      Orchestrators
 Services
        │                │                │
        └────────────────┼────────────────┘
                         ▼
                       DOMAIN
17. Command Architecture
Command
   ↓
Command Handler
   ↓
Application Service
   ↓
Domain
   ↓
Repository
   ↓
Events

Un Command representa intención.

Ejemplo conceptual:

CreateDataProduct
UpdatePolicy
GrantAccess
ExecuteWorkflow
ArchiveDataset
18. Query Architecture
Query
 ↓
Query Handler
 ↓
Read Model
 ↓
Projection / Search
 ↓
Result

Las queries no deben modificar el estado del dominio salvo que exista una razón arquitectónica explícita.

19. Event Architecture
Domain Event
      ↓
Event Publisher
      ↓
Message Infrastructure
      ↓
Consumers
      ↓
Side Effects

Los eventos representan hechos:

Something happened.

No deben utilizarse simplemente como comandos disfrazados.

20. Messaging

EVOXA necesita separar:

Command
Event
Message
Notification

Conceptualmente:

Command
"What should happen?"

Event
"What happened?"

Message
"Information transported between components."
21. Workflow Architecture

Los workflows coordinan múltiples pasos.

Workflow
   ↓
Step 1
   ↓
Step 2
   ↓
Decision
 ┌─┴─┐
 ▼   ▼
A     B
 └─┬─┘
   ▼
Completion

Los workflows no deben absorber reglas de dominio que pertenecen al Domain.

22. Job Architecture

Los Jobs representan trabajo ejecutable.

Job
 ↓
Queue
 ↓
Worker
 ↓
Execution
 ↓
Result
23. Scheduling

Scheduling determina cuándo ejecutar.

Schedule
    ↓
Trigger
    ↓
Job
    ↓
Worker

Scheduling no debe confundirse con Job execution.

24. Repository Architecture

Los repositorios se modelan como abstracciones del dominio/application layer:

Domain
   ↓
Repository Interface
   ↓
Infrastructure Implementation
   ↓
Database

El dominio no conoce:

SQL
ORM
Database Driver
Connection Pool
25. Data Architecture

E67 establece categorías de datos:

Operational Data
Domain Data
Configuration Data
Audit Data
Governance Data
Read Models
Search Indexes
Cache Data
Event Data
Telemetry

No todo debe almacenarse utilizando el mismo mecanismo.

26. Read Models

Los read models sirven para consultas optimizadas.

Domain State
     ↓
Projection
     ↓
Read Model
     ↓
Query

Esto permite separar:

Write Model

de:

Read Model

cuando el caso lo justifique.

27. Search

Search constituye una capacidad especializada:

Data
 ↓
Indexing
 ↓
Search Index
 ↓
Search Query
 ↓
Result

No debe convertirse automáticamente en la fuente autoritativa de datos.

28. Cache

Cache:

Application
    ↓
Cache
    ↓
Source of Truth

La regla fundamental:

Cache no es la fuente autoritativa salvo que un diseño específico lo establezca explícitamente.

29. Serialization

Los límites del sistema requieren transformación:

Domain Object
     ↓
DTO
     ↓
Serializer
     ↓
Wire Format

Los formatos deben estar definidos por contratos.

30. Transformation & Mapping

EVOXA debe distinguir:

Transformation

de:

Mapping

Mapping:

A → B

Transformation:

A → transformed representation

Esto evita introducir conversiones arbitrarias en cualquier capa.

31. Projection

Projection:

Events / State
      ↓
Projection Logic
      ↓
Read Model

Las projections deben poder reconstruirse cuando la arquitectura lo requiera.

32. Governance Plane

EVOXA incorpora un plano transversal:

                 GOVERNANCE
                     │
      ┌──────────────┼──────────────┐
      ▼              ▼              ▼
    Policy         Control       Compliance
      │              │              │
      ▼              ▼              ▼
 Enforcement     Monitoring        Audit
33. Governance Control Plane

Basado en E62:

Policy
  ↓
Control Definition
  ↓
Control Evaluation
  ↓
Decision
  ↓
Enforcement
  ↓
Evidence
34. Governance Evidence

Toda capacidad gobernada importante debe poder producir:

Evidence
   ↓
Provenance
   ↓
Integrity
   ↓
Retention
   ↓
Audit
35. Security Plane

Security debe actuar transversalmente:

Identity
Authentication
Authorization
Secrets
Encryption
Privacy
Audit

No debe implementarse únicamente en la API.

36. Observability Plane

También transversal:

Logs
Metrics
Traces
Events
Health
Audit Signals

Distinción importante:

Observability
      ≠
Audit

pero pueden compartir infraestructura de captura y correlación.

37. Configuration Plane
Configuration
      ↓
Resolution
      ↓
Validation
      ↓
Runtime

Debe soportar:

Defaults
Environment Overrides
Secure Values
Versioning
Validation
38. Feature Flags
Feature
   ↓
Flag Evaluation
   ↓
Enabled / Disabled
   ↓
Runtime Behavior

Los feature flags no deben convertirse en un sustituto de configuración general.

39. Runtime Policy
Request
   ↓
Policy Evaluation
   ↓
Decision
   ↓
Allow / Deny / Modify

Las policies deben poder auditarse cuando afecten decisiones relevantes.

40. Rules Engine

Las reglas permiten separar:

Rule Definition
      ↓
Evaluation
      ↓
Result

No todas las reglas deben convertirse en código rígido.

41. Validation

La validación se divide conceptualmente en:

Input Validation
Domain Validation
Policy Validation
Configuration Validation
Persistence Validation

Cada capa valida aquello de lo que es responsable.

42. Decision Architecture

Una decisión debe poder representarse como:

Decision
{
    decisionId
    subject
    inputs
    rules
    result
    timestamp
}

Cuando sea necesario:

Decision
   ↓
Evidence
43. Action Architecture

Una decisión puede generar una acción:

Decision
   ↓
Action
   ↓
Execution
   ↓
Outcome

Separar:

Decision

de:

Action

permite auditar ambas etapas.

44. Execution Architecture
Action
  ↓
Execution Engine
  ↓
Resource
  ↓
Execution
  ↓
Result

La ejecución debe tener:

Status
Start
End
Actor
Correlation
Result
Failure
45. Resource Architecture

Los recursos ejecutables deben abstraerse:

Resource
{
    resourceId
    type
    capacity
    state
    availability
}

El execution layer consume recursos; no debería asumir detalles concretos innecesarios.

46. Capacity Architecture

EVOXA debe poder determinar:

Demand
   ↓
Capacity
   ↓
Allocation
   ↓
Execution

Esto conecta con E37.

47. Resilience

Todo componente crítico debe definir:

Failure Mode
Retry
Timeout
Fallback
Recovery
Circuit Breaking

cuando corresponda.

48. Data Integrity

Toda operación relevante debe preservar:

Identity
Constraints
Validation
Transactions
Consistency
Auditability
49. Consistency

La aplicación debe especificar explícitamente cuándo necesita:

Strong Consistency

o:

Eventual Consistency

No asumir consistencia fuerte global por defecto.

50. Data Lifecycle

Los datos siguen:

Created
 ↓
Active
 ↓
Modified
 ↓
Retained
 ↓
Archived
 ↓
Disposed

El lifecycle debe estar gobernado.

51. Data Access

La aplicación debe separar:

Domain Access
Application Access
Administrative Access
Governance Access
Audit Access

y aplicar autorización apropiada.

52. Integration Architecture

Los sistemas externos se integran mediante adapters:

EVOXA
  ↓
Integration Port
  ↓
Adapter
  ↓
External System

No debe propagarse la API de un proveedor externo por todo el dominio.

53. Anti-Corruption Layer

Cuando un sistema externo utiliza un modelo incompatible:

External Model
      ↓
ACL
      ↓
EVOXA Model

Esto protege el dominio.

54. API Boundary

La API es un contrato:

Client
  ↓
API
  ↓
Application

La API no debe acceder directamente al repositorio saltándose el Application Layer.

55. Dependency Direction

Regla fundamental:

Interfaces
     ↓
Application
     ↓
Domain

Infrastructure implementa abstracciones requeridas por las capas internas.

Una representación más precisa:

             Interfaces
                 │
                 ▼
            Application
                 │
                 ▼
              Domain
                 ▲
                 │
          Infrastructure

Infrastructure depende de contratos internos.

56. Dependency Rule

El dominio no debe depender de:

HTTP
Database
Message Broker
Cloud Provider
UI
Framework

salvo una decisión arquitectónica explícita y justificada.

57. Transaction Boundary

Las transacciones deben definirse alrededor de casos de uso apropiados.

Use Case
   ↓
Transaction
   ↓
Domain Changes
   ↓
Commit

No hacer de cada función una transacción arbitraria.

58. Outbox Pattern

Cuando una operación modifica estado y publica un evento:

Transaction
 ├── State Change
 └── Outbox Event
        ↓
      Commit
        ↓
 Outbox Publisher
        ↓
     Broker

Esto reduce el riesgo de inconsistencia entre DB y messaging.

59. Idempotency

Las operaciones que puedan repetirse deben definir:

Idempotency Key
Request Identity
Deduplication

Especialmente:

Messages
Jobs
Commands
Integrations
60. Error Architecture

Los errores deben clasificarse.

Domain Error
Application Error
Validation Error
Infrastructure Error
Integration Error
Security Error
Concurrency Error

No devolver excepciones internas arbitrarias directamente al cliente.

61. Result Model

Una operación puede producir:

Success
Failure
Validation Failure
Not Found
Conflict
Unauthorized
Forbidden
Infrastructure Failure

Esto permite contratos consistentes.

62. Correlation Architecture

Cada operación distribuida importante debería poder seguirse:

Request
  ↓
Command
  ↓
Domain
  ↓
Event
  ↓
Consumer
  ↓
Job
  ↓
External Call

mediante:

correlationId
causationId
63. Audit Architecture dentro de E67

E66 se materializa como capacidad transversal:

                    APPLICATION
                         │
      ┌──────────────────┼──────────────────┐
      ▼                  ▼                  ▼
   Commands           Events            Actions
      │                  │                  │
      └──────────────────┼──────────────────┘
                         ▼
                    AUDIT PIPELINE
                         │
                  Evidence Store
                         │
                       Audit
64. Observability vs Audit

Debe mantenerse:

Telemetry
    ↓
Operational Visibility

mientras:

Audit Evidence
    ↓
Governance / Accountability

pueden compartir:

Correlation
Collection
Transport
Storage Technology

pero no necesariamente las mismas políticas de retención.

65. Component Ownership

Cada componente debe tener:

Owner
Responsibility
Public Contract
Dependencies
Data Ownership
Operational Responsibility
66. Data Ownership

Una entidad debe tener un sistema o módulo autoritativo.

Entity
  ↓
Owner
  ↓
Source of Truth

Otros componentes pueden mantener:

Projection
Cache
Replica
Search Index

pero deben reconocer la autoridad original.

67. Source of Truth

EVOXA debe evitar múltiples autoridades simultáneas.

❌ DB A
❌ Cache B
❌ Search C
❌ External D

todos considerados "truth"

En su lugar:

Authoritative Source
        ↓
Replicas / Projections / Caches
68. Module Communication

Preferencia:

Direct Contract

o:

Application Interface

y cuando exista desacoplamiento temporal:

Event / Message

Evitar acceso directo a tablas internas de otro módulo.

69. Internal API

Incluso dentro del mismo proceso deben definirse límites cuando la complejidad lo justifique.

Module A
   ↓
Public Contract
   ↓
Module B

No:

Module A
   ↓
B.internal.database.table
70. Deployment Unit

Un módulo arquitectónico no tiene por qué ser un microservicio.

Puede desplegarse como:

Same Process
Separate Process
Worker
Service
Serverless Function

dependiendo de las necesidades.

71. Monolith vs Microservices

E67 no debe asumir automáticamente:

Microservices = better

La decisión debe depender de:

Boundaries
Scaling
Ownership
Failure Isolation
Deployment Independence
Operational Complexity
72. Recommended Initial Strategy

Para el desarrollo inicial de EVOXA:

Modular Architecture
        +
Strong Internal Boundaries
        +
Clear Contracts
        +
Selective Asynchronous Processing

Esto permite evolucionar posteriormente hacia servicios independientes sin destruir los límites internos.

73. Runtime Topology

Modelo conceptual:

                    CLIENTS
                       │
                       ▼
                  API GATEWAY
                       │
                       ▼
                EVOXA APPLICATION
                       │
        ┌──────────────┼──────────────┐
        ▼              ▼              ▼
     Domain        Workers       Schedulers
        │              │              │
        └──────────────┼──────────────┘
                       ▼
              Infrastructure Layer
             ┌─────────┼─────────┐
             ▼         ▼         ▼
            DB      Messaging   Cache
                       │
                       ▼
                 External Systems
74. Security Boundary
Internet / Clients
        │
   Authentication
        │
   Authorization
        │
      API
        │
   Application
        │
      Domain
        │
 Infrastructure

Cada boundary debe aplicar los controles correspondientes.

75. Implementation Units

El desarrollo se dividirá en:

Foundation
Core
Domain Modules
Application Modules
Infrastructure
Interfaces
Governance
Observability
Testing
Deployment
76. Implementation Order

No conviene implementar E01 → E66 literalmente.

El orden técnico recomendado es:

1. Foundation
2. Core
3. Domain Model
4. Application Layer
5. Persistence
6. API
7. Messaging
8. Events
9. Workflows
10. Jobs
11. Scheduling
12. Governance
13. Audit
14. Observability
15. Resilience
16. Deployment
77. Vertical Slice Strategy

Cada capacidad importante debería implementarse como vertical slice:

API
 ↓
Application
 ↓
Domain
 ↓
Repository
 ↓
Database
 ↓
Event
 ↓
Audit
 ↓
Tests

Esto permite obtener funcionalidades reales temprano.

78. First Vertical Slice

Antes de construir toda la plataforma, necesitamos escoger una capacidad núcleo de EVOXA.

Esa capacidad debe atravesar:

API
Application
Domain
Persistence
Validation
Authorization
Audit
Testing

y convertirse en el primer walking skeleton.

79. Walking Skeleton

El primer sistema ejecutable debe demostrar:

Client
 ↓
API
 ↓
Use Case
 ↓
Domain
 ↓
Repository
 ↓
Database
 ↓
Response

y además:

Audit
Logging
Correlation
Tests
80. Project Structure — Logical

La estructura conceptual inicial:

EVOXA/
│
├── src/
│   ├── core/
│   ├── domain/
│   ├── application/
│   ├── infrastructure/
│   ├── interfaces/
│   └── governance/
│
├── tests/
│   ├── unit/
│   ├── integration/
│   ├── architecture/
│   └── end_to_end/
│
├── docs/
│
├── scripts/
│
└── deployment/

El lenguaje y framework concretos quedan para E68.

81. Test Architecture

E67 requiere:

Unit Tests
Integration Tests
Contract Tests
Architecture Tests
End-to-End Tests
Security Tests
Migration Tests
Resilience Tests
82. Architecture Tests

Los límites arquitectónicos deben ser testeables.

Ejemplo:

Domain
MUST NOT depend on
Infrastructure

El sistema de tests debe poder detectar una violación.

83. Contract Tests

Los contratos deben probarse independientemente:

API Contract
Event Contract
Message Contract
Integration Contract
Repository Contract
84. Definition of Done

Una funcionalidad no está terminada únicamente porque:

Code compiles

Debe cumplir:

Code
+ Tests
+ Contract
+ Security
+ Observability
+ Audit
+ Documentation

según corresponda.

85. Architecture Decision Records

Las decisiones importantes deben registrarse:

ADR
{
    decision
    context
    alternatives
    consequences
}

Esto evita que decisiones críticas desaparezcan del conocimiento del proyecto.

86. Change Management

Cuando cambie la arquitectura:

Change Request
      ↓
Impact Analysis
      ↓
ADR
      ↓
Architecture Update
      ↓
Implementation
87. Traceability Matrix

E67 debe mantener una relación:

Architecture Requirement
        ↓
Architectural Component
        ↓
Module
        ↓
Implementation Unit
        ↓
Test

Esto será extremadamente importante cuando EVOXA crezca.

88. E01–E66 → E67 Mapping

La transformación general:

E01–E07
Foundation / EVOXA Core
        ↓
E08–E10
Application / Domain / Repository
        ↓
E11–E18
Integration / Messaging / Runtime Support
        ↓
E19–E26
Policies / Rules / Transformation / Projection
        ↓
E27–E37
Query / Read / Search / Analytics / Execution / Resources
        ↓
E38–E44
Resilience / Recovery / Integrity / Consistency
        ↓
E45–E58
Data Lifecycle / Access / Protection / Privacy
        ↓
E59–E66
Governance / Compliance / Assurance / Audit
        ↓
E67
MASTER APPLICATION BLUEPRINT
89. E67 como mapa de implementación

El resultado final debe poder expresarse así:

                         EVOXA
                           │
       ┌───────────────────┼───────────────────┐
       │                   │                   │
       ▼                   ▼                   ▼
     DOMAIN          APPLICATION          GOVERNANCE
       │                   │                   │
       └───────────────────┼───────────────────┘
                           │
                           ▼
                    INFRASTRUCTURE
                           │
       ┌───────────┬───────┼───────┬───────────┐
       ▼           ▼       ▼       ▼           ▼
      DATA       CACHE   EVENTS   SEARCH    EXTERNAL
       │           │       │       │           │
       └───────────┴───────┼───────┴───────────┘
                           ▼
                        RUNTIME
                           │
                           ▼
                     OBSERVABILITY
                           │
                           ▼
                         AUDIT
90. Qué produce E67

Al terminar E67 debemos tener definidos conceptualmente:

✓ System Boundaries
✓ Architectural Layers
✓ Bounded Contexts
✓ Modules
✓ Components
✓ Responsibilities
✓ Dependency Rules
✓ Data Ownership
✓ Contracts
✓ Commands
✓ Queries
✓ Events
✓ Workflows
✓ Jobs
✓ Scheduling
✓ Integration Boundaries
✓ Governance Boundaries
✓ Security Boundaries
✓ Audit Boundaries
✓ Runtime Model
✓ Testing Model
✓ Implementation Sequence
91. Qué queda para E68+

Ahora sí podemos pasar de:

¿Qué es EVOXA?

a:

¿Con qué tecnología y estructura concreta vamos a construir EVOXA?

Por eso la siguiente secuencia queda:

E67 — MASTER APPLICATION BLUEPRINT
             ↓
E68 — EVOXA TECHNICAL STACK & PLATFORM ARCHITECTURE
             ↓
E69 — EVOXA PROJECT / REPOSITORY ARCHITECTURE
             ↓
E70 — EVOXA MODULE & PACKAGE ARCHITECTURE
             ↓
E71 — EVOXA DATABASE ARCHITECTURE
             ↓
E72 — EVOXA API ARCHITECTURE
             ↓
E73 — EVOXA TESTING ARCHITECTURE
             ↓
E74 — EVOXA DEPLOYMENT ARCHITECTURE
             ↓
           CODE
92. Punto de transición

E67 es, por tanto, el puente entre arquitectura y desarrollo.

Hasta E66 estábamos definiendo cómo debe comportarse y gobernarse EVOXA.

A partir de E67 estamos definiendo cómo se materializa EVOXA como producto de software.

Y hay una decisión importante: todavía no conviene empezar a escribir código de producción hasta cerrar E68–E74, porque ahí fijaremos stack, estructura del repositorio, módulos, persistencia, APIs, testing y deployment.

Después de eso sí podemos abrir el proyecto y comenzar por el Foundation / Walking Skeleton de EVOXA, implementando la primera vertical slice de extremo a extremo.
