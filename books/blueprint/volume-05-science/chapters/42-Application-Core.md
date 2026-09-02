42 — Application Core
42.1 — Overview

En el capítulo 41 — Application Foundation definimos qué constituye una aplicación dentro de EVOXA:

identidad;
propósito;
contexto;
límites;
configuración;
seguridad;
políticas;
datos;
dependencias;
observabilidad;
AI;
Agents;
lifecycle.

Ahora debemos definir cómo funciona internamente una Application.

Por eso:

Application Core es el núcleo de coordinación, estado, comportamiento, reglas y ejecución lógica de una aplicación EVOXA.

La relación es:

EVOXA
  │
  ▼
PLATFORM
  │
  ▼
APPLICATION FOUNDATION
  │
  ▼
APPLICATION CORE
  │
  ├── State
  ├── Commands
  ├── Events
  ├── Queries
  ├── Rules
  ├── Policies
  ├── Orchestration
  ├── Coordination
  └── Consistency

Application Core no es simplemente el backend.

Es el cerebro operacional de la aplicación.

42.2 — Purpose of Application Core

Application Core tiene cinco responsabilidades principales:

                 APPLICATION CORE
                        │
        ┌───────────────┼───────────────┐
        ▼               ▼               ▼
      STATE          BEHAVIOR       COORDINATION
        │               │               │
        └───────────────┼───────────────┘
                        ▼
                    CONSISTENCY
                        │
                        ▼
                  ORCHESTRATION

Debe determinar:

qué estado tiene la aplicación;
cómo cambia ese estado;
qué operaciones puede realizar;
qué reglas deben cumplirse;
cómo interactúan sus módulos;
cómo se coordinan sus componentes;
cómo se procesan comandos y eventos;
cómo se manejan errores;
cómo se mantiene la consistencia.
42.3 — Application Core vs Platform Core

Es importante no confundir ambos.

Platform Core

Coordina:

Platform
Infrastructure
Services
Resources
Global State
Application Core

Coordina:

Application
Business State
Application Behavior
Modules
Domains
Capabilities

La relación:

                 PLATFORM CORE
                       │
                 coordinates
                       ▼
              APPLICATION CORE
                       │
                 coordinates
                       ▼
               APPLICATION

Por lo tanto:

Platform Core coordina la plataforma; Application Core coordina la aplicación.

42.4 — Application Core Model

La arquitectura conceptual:

┌──────────────────────────────────────────┐
│              APPLICATION CORE            │
├──────────────────────────────────────────┤
│                                          │
│ Identity Context                         │
│ Application Context                      │
│ State Management                         │
│ Command Processing                       │
│ Query Processing                         │
│ Event Processing                         │
│ Business Rules                           │
│ Policies                                 │
│ Orchestration                            │
│ Transactions                             │
│ Consistency                              │
│ Error Handling                           │
│ Coordination                             │
│ AI / Agent Coordination                  │
│                                          │
└──────────────────────────────────────────┘

El Core no debería contener toda la lógica de la aplicación.

Debe coordinarla.

42.5 — Application State

El estado es uno de los elementos centrales.

Una aplicación puede tener:

Application State
│
├── System State
├── Configuration State
├── User State
├── Session State
├── Business State
├── Workflow State
├── Integration State
├── AI State
└── Agent State

El Core debe controlar las transiciones de estado.

42.6 — State Transition

El cambio de estado debe ser explícito.

Current State
      │
      ▼
    Command
      │
      ▼
Validation
      │
      ▼
Business Rules
      │
      ▼
State Transition
      │
      ▼
New State
      │
      ▼
Event

Por ejemplo:

Workout
  │
  ▼
PLANNED
  │
  │ StartWorkout
  ▼
ACTIVE
  │
  │ CompleteWorkout
  ▼
COMPLETED
42.7 — State Machine

Las entidades importantes pueden utilizar máquinas de estado.

                 CREATED
                    │
                    ▼
                 ACTIVE
                /      \
               ▼        ▼
           SUSPENDED   COMPLETED
               │
               ▼
             ACTIVE
               │
               ▼
            ARCHIVED

Esto evita estados ambiguos como:

status = "maybe"

El Core debe conocer:

estados válidos;
transiciones válidas;
condiciones;
acciones;
eventos generados.
42.8 — Desired State vs Actual State

Siguiendo el modelo establecido en Platform Core:

Desired State
      │
      ▼
Application Core
      │
      ▼
Application Runtime
      │
      ▼
Actual State

Ejemplo:

Desired:
Workflow = ACTIVE

Actual:
Workflow = STOPPED

Application Core puede detectar:

State Drift

y solicitar reconciliación.

42.9 — Application Reconciliation

El proceso:

Desired State
      │
      ▼
Compare
      │
      ▼
Actual State
      │
      ▼
Drift?
   /     \
 No       Yes
 │         │
 ▼         ▼
Continue Reconcile
             │
             ▼
          Validate
             │
             ▼
           Apply
             │
             ▼
           Verify

Esto permite construir aplicaciones más resistentes.

42.10 — Commands

Los Commands representan una intención de cambiar algo.

Command
│
├── Identity
├── Actor
├── Target
├── Action
├── Parameters
├── Context
├── Permissions
└── Correlation ID

Ejemplo:

CreateWorkout
UpdateWorkout
StartWorkout
CompleteWorkout
CancelWorkout
GeneratePlan
ApprovePlan

Un Command expresa:

Quiero que ocurra esta acción.

42.11 — Command Processing

El Core procesa Commands mediante:

Command
   │
   ▼
Authentication
   │
   ▼
Authorization
   │
   ▼
Validation
   │
   ▼
Policy Evaluation
   │
   ▼
Business Rules
   │
   ▼
Execution
   │
   ▼
State Change
   │
   ▼
Events

Esto establece un flujo consistente para las operaciones.

42.12 — Queries

Las Queries representan solicitudes de información.

Query
   │
   ▼
Application Core
   │
   ▼
Read Model / Data
   │
   ▼
Response

Ejemplos:

GetWorkout
GetUserProgress
GetNutritionPlan
GetDashboard
GetTrainingHistory

Una Query idealmente no cambia el estado.

42.13 — Commands vs Queries

La separación conceptual:

COMMAND
"I want to change something."
        │
        ▼
     STATE CHANGE


QUERY
"I want to know something."
        │
        ▼
     READ DATA

Esto permite separar:

Write Model

de:

Read Model

cuando la arquitectura lo requiera.

42.14 — Events

Los Events representan hechos ocurridos.

Event
│
├── Identity
├── Type
├── Timestamp
├── Actor
├── Source
├── Entity
├── Payload
└── Correlation

Ejemplos:

UserCreated
WorkoutStarted
WorkoutCompleted
PaymentProcessed
PlanGenerated
AgentActionCompleted

Un Event expresa:

Esto ocurrió.

42.15 — Command → State → Event

Uno de los patrones centrales será:

Command
   │
   ▼
Application Core
   │
   ▼
State Transition
   │
   ▼
Event

Por ejemplo:

CompleteWorkout
       │
       ▼
Application Core
       │
       ▼
Workout = COMPLETED
       │
       ▼
WorkoutCompleted

El evento puede activar otros procesos:

WorkoutCompleted
      │
      ├── Analytics
      ├── Notifications
      ├── AI
      ├── Rewards
      └── Progress
42.16 — Application Event Model

Los eventos pueden clasificarse:

Domain Events

Representan hechos del dominio.

WorkoutCompleted
PaymentConfirmed
GoalAchieved
Application Events

Representan hechos de la aplicación.

ApplicationStarted
ModuleActivated
ConfigurationChanged
Integration Events

Comunican con otros sistemas.

CustomerCreated
InvoiceIssued
WebhookReceived
System Events

Relacionados con infraestructura o Platform.

ServiceUnavailable
ResourceLimitReached
DeploymentCompleted
42.17 — Event-Driven Application

Una Application puede construirse alrededor de eventos:

                 EVENT BUS
                    │
       ┌────────────┼────────────┐
       ▼            ▼            ▼
   Module A      Module B     Module C
       │            │            │
       ▼            ▼            ▼
   Component     Service       AI

Esto reduce acoplamiento directo entre módulos.

42.18 — Business Rules

Application Core es responsable de coordinar las reglas de negocio, pero no necesariamente de almacenarlas todas directamente.

Application Core
       │
       ▼
Business Rules
       │
       ├── Validation
       ├── Constraints
       ├── Eligibility
       ├── Calculations
       └── Decisions

Ejemplo:

IF user completed required assessment
THEN allow personalized plan

Las reglas deben ser:

explícitas;
versionables;
testeables;
auditables.
42.19 — Policies vs Business Rules

Hay que mantener una distinción.

Business Rule

Define cómo funciona el negocio.

A completed workout contributes to progress.
Policy

Define qué está permitido.

Only authorized coaches can approve this plan.

Por lo tanto:

Business Rules
       +
Policies
       ↓
Application Decision
42.20 — Decision Engine

Las decisiones complejas pueden delegarse a un Decision Engine.

Context
   │
   ▼
Rules
   │
   ▼
Policies
   │
   ▼
AI / Decision Logic
   │
   ▼
Decision

Ejemplo:

User Context
      │
      ▼
Training Rules
      │
      ▼
Safety Policies
      │
      ▼
AI Analysis
      │
      ▼
Training Recommendation
42.21 — Application Context

Toda operación debe ejecutarse dentro de un contexto.

Application Context
│
├── User
├── Organization
├── Tenant
├── Application
├── Session
├── Device
├── Locale
├── Permissions
├── Policies
└── Correlation

El Core utiliza este contexto para tomar decisiones correctas.

42.22 — Request Context

Un Request puede contener:

Request
│
├── Request ID
├── Correlation ID
├── Actor
├── Application
├── Tenant
├── Locale
├── Permissions
└── Metadata

Esto permite trazabilidad de extremo a extremo.

42.23 — Correlation

Una operación compleja puede atravesar múltiples módulos:

Request
 │
 ▼
Application Core
 │
 ├── Module A
 │     │
 │     ▼
 │   Service
 │
 ├── Module B
 │
 └── AI

Todos deben conservar un:

Correlation ID

para poder reconstruir la operación completa.

42.24 — Application Orchestration

Application Core puede orquestar operaciones complejas.

Orchestration
│
├── Step 1
├── Step 2
├── Step 3
├── Step 4
└── Completion

Ejemplo:

Create Personalized Plan
        │
        ▼
Load User Profile
        │
        ▼
Load Training History
        │
        ▼
Load Nutrition Data
        │
        ▼
AI Analysis
        │
        ▼
Generate Plan
        │
        ▼
Validate
        │
        ▼
Save
        │
        ▼
Notify User
42.25 — Orchestration vs Choreography

EVOXA debe soportar ambos modelos.

Orchestration

Un coordinador controla el proceso:

          Core
       /    |    \
      ▼     ▼     ▼
     A      B      C
Choreography

Los componentes reaccionan a eventos:

A
│
▼
Event
│
├── B
│
└── C

La elección depende del problema.

42.26 — Transactions

Las operaciones que requieren consistencia pueden utilizar transacciones.

BEGIN
   │
   ├── Operation A
   ├── Operation B
   └── Operation C
   │
   ▼
COMMIT

Si existe un error:

ROLLBACK

Sin embargo, las transacciones distribuidas deben utilizarse con cuidado.

42.27 — Distributed Consistency

En aplicaciones distribuidas puede no existir una única transacción global.

EVOXA debe soportar patrones como:

Saga
Compensation
Eventual Consistency
Idempotency
Retry

Ejemplo:

Step A
  │
  ▼
Step B
  │
  ▼
Step C
  │
  X failure
  │
  ▼
Compensation
42.28 — Idempotency

Las operaciones críticas deben poder ser repetidas de manera segura cuando sea necesario.

Request #123
    │
    ▼
Operation
    │
    ▼
Success

Retry #123
    │
    ▼
No duplicate effect

Esto es fundamental para:

pagos;
órdenes;
workflows;
integraciones;
Agents.
42.29 — Retry Strategy

El Core debe diferenciar errores recuperables de errores definitivos.

Error
 │
 ├── Recoverable
 │       │
 │       ▼
 │     Retry
 │
 └── Permanent
         │
         ▼
       Fail

Las estrategias pueden incluir:

Immediate Retry
Exponential Backoff
Scheduled Retry
Dead Letter
Manual Recovery
42.30 — Error Model

Los errores deben ser estructurados.

Application Error
│
├── Code
├── Category
├── Message
├── Severity
├── Retryable
├── Context
├── Correlation ID
└── Recovery Action

Categorías:

Validation
Authorization
Business
Integration
Infrastructure
Data
AI
Agent
Timeout
Conflict
42.31 — Application Failure Handling

El Core debe poder decidir:

Failure
 │
 ├── Retry
 ├── Fallback
 ├── Compensate
 ├── Degrade
 ├── Queue
 ├── Escalate
 └── Fail

Esto evita que cualquier error termine automáticamente en un fallo completo de la aplicación.

42.32 — Application Coordination

El Core conecta los distintos elementos internos.

                APPLICATION CORE
                       │
       ┌───────────────┼───────────────┐
       ▼               ▼               ▼
    MODULE A        MODULE B        MODULE C
       │               │               │
       └───────────────┼───────────────┘
                       ▼
                    SERVICES
                       │
                       ▼
                  PLATFORM

La aplicación obtiene una estructura coordinada sin que todos los módulos deban conocerse entre sí.

42.33 — Module Isolation

El Core debe favorecer límites claros:

Module A
   │
   X── Direct internal access → Module B
   │
   ▼
Contract / Event
   │
   ▼
Module B

Esto permite evolucionar módulos independientemente.

42.34 — Application Capability Execution

Las capacidades definidas en el capítulo 48 deben ejecutarse a través del Core.

Capability
     │
     ▼
Application Core
     │
     ▼
Rules / Policies
     │
     ▼
Modules
     │
     ▼
Components
     │
     ▼
Services

Esto proporciona una ruta controlada desde la intención hasta la ejecución.

42.35 — Application Core and Platform

La relación completa:

                       EVOXA
                         │
                      PLATFORM
                         │
             ┌───────────┼───────────┐
             ▼           ▼           ▼
          Services   Capabilities Contracts
             │           │           │
             └───────────┼───────────┘
                         ▼
                APPLICATION FOUNDATION
                         │
                         ▼
                  APPLICATION CORE
                         │
             ┌───────────┼───────────┐
             ▼           ▼           ▼
          Modules      Domains     Policies
             │           │           │
             └───────────┼───────────┘
                         ▼
                  Application Runtime
42.36 — AI Integration

AI puede participar en Application Core como:

Advisor
Predictor
Classifier
Decision Support
Generator
Analyzer
Optimizer

Pero el Core debe controlar:

Permissions
Context
Policies
Validation
Audit

AI no debería modificar directamente el estado crítico sin pasar por los mecanismos de control.

42.37 — AI Decision Flow

Un flujo seguro:

Application Event
       │
       ▼
Application Core
       │
       ▼
AI Analysis
       │
       ▼
Recommendation
       │
       ▼
Policy Evaluation
       │
       ▼
Business Rules
       │
       ▼
Decision
       │
       ▼
State Change

Esto mantiene a AI dentro de la arquitectura gobernada.

42.38 — Agent Coordination

Los Agents pueden enviar Commands al Application Core.

Agent
 │
 ▼
Capability
 │
 ▼
Contract
 │
 ▼
Application Core
 │
 ▼
Authorization
 │
 ▼
Policy
 │
 ▼
Command
 │
 ▼
State Change

El Agent no debería tener acceso directo e irrestricto al estado interno.

42.39 — Human-in-the-Loop

Para acciones críticas:

Agent
  │
  ▼
Recommendation
  │
  ▼
Application Core
  │
  ▼
Human Approval
  │
  ▼
Command
  │
  ▼
State Change

Esto permite distintos niveles de autonomía.

42.40 — Application Core as Control Center

Podemos visualizarlo así:

                    APPLICATION
                         │
                         ▼
                ┌─────────────────┐
                │ APPLICATION CORE│
                ├─────────────────┤
                │                 │
                │ State           │
                │ Commands        │
                │ Queries         │
                │ Events          │
                │ Rules           │
                │ Policies        │
                │ Decisions       │
                │ Orchestration   │
                │ Transactions    │
                │ Errors          │
                │ AI              │
                │ Agents          │
                │                 │
                └────────┬────────┘
                         │
              ┌──────────┼──────────┐
              ▼          ▼          ▼
           Modules     Services   Runtime
42.41 — Application Core Event Flow

Un flujo completo puede ser:

User / AI / Agent
       │
       ▼
     Request
       │
       ▼
   Application
       │
       ▼
 Application Core
       │
       ├── Authenticate
       ├── Authorize
       ├── Validate
       ├── Evaluate Policy
       ├── Apply Rules
       ├── Execute
       ├── Change State
       └── Emit Event
                    │
                    ▼
              Event Consumers
                    │
          ┌─────────┼─────────┐
          ▼         ▼         ▼
         AI      Analytics   Notification
42.42 — Application Core Observability

El Core debe producir información sobre su comportamiento.

Core
 │
 ├── Command Metrics
 ├── Query Metrics
 ├── Event Metrics
 ├── State Transitions
 ├── Rule Decisions
 ├── Policy Decisions
 ├── Errors
 ├── Latency
 └── Correlation

Esto permite conocer no solamente:

qué hizo la aplicación

sino también:

por qué lo hizo.

42.43 — Decision Observability

Las decisiones importantes deben poder explicarse.

Decision
│
├── Input Context
├── Rules Evaluated
├── Policies Evaluated
├── AI Recommendation
├── Final Decision
├── Actor
└── Timestamp

Esto será especialmente importante en aplicaciones que utilicen AI.

42.44 — Application Core Audit

Las operaciones críticas deben generar auditoría.

Actor
 │
 ▼
Command
 │
 ▼
Decision
 │
 ▼
State Change
 │
 ▼
Audit Event

Debe ser posible responder:

Who?
What?
Why?
When?
Under which policy?
Which version?
What changed?
42.45 — Application Core Security

El Core debe actuar como una frontera de seguridad lógica.

External Input
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
Application Core

Ninguna operación crítica debería saltarse este flujo.

42.46 — Application Core Performance

El Core debe minimizar la latencia de coordinación.

Métricas importantes:

Command Latency
Query Latency
Event Processing Latency
Decision Latency
State Transition Latency
Orchestration Duration

Esto permite identificar cuellos de botella.

42.47 — Application Core Scalability

El Core debe poder escalar horizontalmente cuando sea necesario.

                Application Core
                      │
          ┌───────────┼───────────┐
          ▼           ▼           ▼
       Core #1     Core #2     Core #3
          │           │           │
          └───────────┼───────────┘
                      ▼
                 Shared State

Esto requiere mecanismos adecuados para:

concurrencia;
locking;
idempotencia;
eventos;
consistencia.
42.48 — Concurrency

Dos operaciones pueden intentar cambiar el mismo estado.

Command A ──┐
            ├──► Entity
Command B ──┘

El Core debe controlar:

Optimistic Concurrency
Pessimistic Locking
Version Checks
Conflict Detection
Conflict Resolution

Ejemplo:

Entity Version = 4

Command A → expected v4 → success → v5
Command B → expected v4 → conflict
42.49 — Application Core Resilience

El Core debe resistir fallos de:

Database
Services
Network
AI
External APIs
Message Broker
Modules
Runtime

La estrategia:

Detect
  │
  ▼
Isolate
  │
  ▼
Recover
  │
  ▼
Verify
  │
  ▼
Resume
42.50 — Application Core Principles
Principle 1 — Core Coordinates

El Core coordina; no absorbe toda la lógica.

Principle 2 — State Must Be Explicit

El estado importante debe ser explícito.

Principle 3 — Commands Express Intent

Los Commands expresan intención.

Principle 4 — Events Represent Facts

Los Events representan hechos ocurridos.

Principle 5 — Policies Govern Actions

Las Policies controlan acciones.

Principle 6 — Rules Define Behavior

Las Business Rules definen comportamiento.

Principle 7 — Contracts Protect Boundaries

Los Contracts protegen las fronteras.

Principle 8 — Decisions Must Be Observable

Las decisiones importantes deben poder rastrearse.

Principle 9 — AI Must Operate Within Governance

AI debe operar dentro de reglas y políticas.

Principle 10 — Agents Must Use Controlled Interfaces

Los Agents deben utilizar capacidades y contratos, no acceso directo indiscriminado.

42.51 — Application Core Maturity Model
Level 1 — Basic
State
Commands
Basic Logic
Level 2 — Structured
Modules
Rules
Events
Level 3 — Governed
Policies
Authorization
Audit
Level 4 — Distributed
Events
Async Processing
Distributed State
Level 5 — Resilient
Retries
Compensation
Recovery
Reconciliation
Level 6 — Intelligent
AI Decisions
Prediction
Optimization
Level 7 — Agentic
Agents
Tools
Autonomous Decisions
Human Oversight
Level 8 — Adaptive
Observe
Analyze
Learn
Optimize
Evolve
42.52 — Application Core and Self-Management

En una aplicación avanzada:

Application
      │
      ▼
Observe
      │
      ▼
Application Core
      │
      ▼
Analyze
      │
      ▼
Decide
      │
      ▼
Act
      │
      ▼
Verify

Esto permite que algunas aplicaciones puedan:

detectar problemas;
ajustar configuraciones;
optimizar workflows;
recomendar cambios;
recuperar servicios;
escalar recursos.

Siempre dentro de los límites establecidos por Platform.

42.53 — Final Application Core Architecture
                         APPLICATION
                              │
                    APPLICATION FOUNDATION
                              │
                              ▼
                    ┌─────────────────────┐
                    │   APPLICATION CORE  │
                    ├─────────────────────┤
                    │                     │
                    │ Application Context │
                    │ State Management    │
                    │ Commands            │
                    │ Queries             │
                    │ Events              │
                    │ Business Rules      │
                    │ Policies            │
                    │ Decisions           │
                    │ Orchestration       │
                    │ Transactions        │
                    │ Consistency         │
                    │ Error Handling      │
                    │ AI Coordination     │
                    │ Agent Coordination  │
                    │ Audit               │
                    │ Observability       │
                    │                     │
                    └──────────┬──────────┘
                               │
                ┌──────────────┼──────────────┐
                ▼              ▼              ▼
             MODULES        SERVICES        DATA
                │              │              │
                └──────────────┼──────────────┘
                               ▼
                      APPLICATION RUNTIME
                               │
                               ▼
                           PLATFORM
42.54 — Final Responsibility Model

Con la incorporación de Application Core, la arquitectura EVOXA queda:

Foundation
    = Defines

Core
    = Coordinates

Runtime
    = Executes

Services
    = Provides

Modules
    = Organizes

Domains
    = Contextualizes

Components
    = Implements

Capabilities
    = Exposes

Contracts
    = Standardizes

Lifecycle
    = Evolves

A nivel de Application:

Application Foundation
    = Defines the Application

Application Core
    = Coordinates the Application

Application Runtime
    = Executes the Application

Application Services
    = Provides reusable Application services

Application Modules
    = Organizes Application functionality

Application Domains
    = Contextualizes Application behavior

Application Components
    = Implements Application logic

Application Capabilities
    = Exposes Application abilities

Application Contracts
    = Standardizes Application interaction

Application Lifecycle
    = Evolves the Application
42.55 — Final Definition

EVOXA Application Core is the central coordination layer responsible for managing application state, processing commands and queries, emitting and consuming events, enforcing business rules and policies, coordinating modules and services, maintaining consistency, orchestrating complex operations, integrating AI and Agents, handling failures, and providing observable and auditable application behavior.

42.56 — Final Vision

El Application Core representa el paso desde una aplicación simplemente ejecutable hacia una aplicación coordinada e inteligente.

La evolución es:

Basic Application
        │
        ▼
Structured Application
        │
        ▼
Event-Driven Application
        │
        ▼
Distributed Application
        │
        ▼
Resilient Application
        │
        ▼
AI-Native Application
        │
        ▼
Agentic Application
        │
        ▼
Adaptive Application

Y la arquitectura futura puede resumirse:

                    USER
                     │
                     ▼
              APPLICATION
                     │
                     ▼
            APPLICATION CORE
                     │
       ┌─────────────┼─────────────┐
       ▼             ▼             ▼
     STATE        DECISIONS      EVENTS
       │             │             │
       │            AI/AGENTS      │
       │             │             │
       └─────────────┼─────────────┘
                     ▼
                 MODULES
                     │
                     ▼
                SERVICES
                     │
                     ▼
                 PLATFORM

Application Foundation defines what an application is. Application Core defines how that application thinks, coordinates, decides and changes its state.

El siguiente paso natural es 43 — Application Runtime, donde definiremos cómo todo lo coordinado por Application Core pasa de la lógica interna a la ejecución real de la aplicación: procesos, workers, sesiones, requests, recursos, networking, scaling, resiliencia, aislamiento y runtime AI/Agent.
