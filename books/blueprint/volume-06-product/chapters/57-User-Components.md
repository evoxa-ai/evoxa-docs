57 — User Components
57.1 — Introducción

Los User Components representan la unidad concreta de implementación dentro de la arquitectura Users & Experience de EVOXA.

En los capítulos anteriores hemos construido progresivamente la arquitectura:

User Foundation define al User.
User Core coordina al User.
User Runtime ejecuta la interacción.
User Services proporcionan servicios reutilizables.
User Modules organizan la funcionalidad.
User Domains contextualizan el comportamiento.
User Components implementan esa funcionalidad.
User Capabilities expondrán lo que el sistema puede hacer.
User Contracts estandarizarán cómo interactúan las partes.
User Lifecycle gobernará cómo todo evoluciona.

Por lo tanto:

User Components = Implementan la funcionalidad del User.

El Component es el punto donde las definiciones arquitectónicas anteriores comienzan a convertirse en comportamiento ejecutable.

57.2 — Definición de User Component

Un User Component es una unidad cohesiva, reutilizable, encapsulada, testeable, observable y gobernada de implementación que ejecuta una responsabilidad específica dentro de un User Module y User Domain.

Conceptualmente:

User Component
├── Identity
├── Purpose
├── Responsibility
├── Scope
├── Ownership
├── Interface
├── Inputs
├── Outputs
├── State
├── Logic
├── Dependencies
├── Policies
├── Permissions
├── Events
├── Contracts
├── Configuration
├── Security
├── Observability
├── Testing
├── Performance
├── Cost
├── AI Metadata
├── Agent Metadata
└── Lifecycle

El Component debe representar una responsabilidad suficientemente pequeña para ser comprendida y suficientemente completa para ser reutilizada.

57.3 — Principio fundamental

La regla fundamental es:

Un Component debe hacer una cosa bien, dentro de un contexto claramente definido.

Por ejemplo:

Profile Module
│
├── Profile Reader
├── Profile Validator
├── Profile Writer
├── Avatar Manager
└── Profile Mapper

Cada componente tiene una responsabilidad específica.

No deberíamos crear:

UserEverythingComponent

que gestione:

Profile
Billing
Goals
Notifications
AI
Security

Eso destruiría la modularidad.

57.4 — Component vs Module

Es importante mantener la separación.

Module

Organiza funcionalidad:

Profile Module
Component

Implementa una parte de esa funcionalidad:

Profile Validator
Profile Reader
Profile Writer

Por lo tanto:

Module
   ↓
organiza
   ↓
Components
   ↓
implementan
57.5 — Component vs Service

Un Service representa una funcionalidad reutilizable.

Un Component representa una unidad de implementación.

Ejemplo:

Profile Service
      ↓
Profile Reader
Profile Validator
Profile Writer

El Service puede utilizar varios Components.

Service
   ↓
Components
57.6 — Component vs Capability

Una Capability expresa lo que el sistema puede hacer.

Un Component expresa cómo se implementa.

Ejemplo:

Capability:
profile.update

puede ser implementada mediante:

Profile Update Component

La separación permite cambiar la implementación sin cambiar necesariamente la capacidad.

57.7 — Component Anatomy

Todo User Component debería tener una identidad estructurada.

Component ID
Name
Namespace
Version
Type
Owner
Module
Domain
Status

Ejemplo:

Component ID:
user.profile.validator

Name:
Profile Validator

Type:
Business Component

Module:
Profile

Domain:
User
57.8 — Component Purpose

Debe existir una definición clara de propósito.

Ejemplo:

Purpose:
Validate profile updates before
they are persisted.

El propósito debe permitir determinar:

qué hace;
qué no hace;
qué datos necesita;
qué dependencias posee;
qué resultados produce.
57.9 — Responsibility

La responsabilidad debe ser explícita.

Ejemplo:

Responsibility:

Validate:
- Required fields
- Data formats
- Business constraints
- Privacy constraints
- User permissions

El componente no debería asumir responsabilidades que pertenecen a otro componente.

57.10 — Scope

El Scope determina dónde puede operar.

Puede incluir:

Global
Organization
Tenant
Application
Domain
Module
User
Session
Resource

Por ejemplo:

Profile Component
Scope:
User

mientras que:

Tenant Preference Component
Scope:
Tenant
57.11 — Ownership

Cada componente debe tener un propietario.

Owner
Team
Organization
Application
Domain

Esto permite determinar:

Who maintains it?
Who approves changes?
Who owns incidents?
Who owns security?
Who owns cost?
57.12 — Component Interface

Todo Component debe exponer una interfaz clara.

Ejemplo conceptual:

ProfileValidator
│
├── validate(input)
│
└── returns ValidationResult

La interfaz debe evitar revelar detalles internos.

57.13 — Inputs

Los Inputs deben estar formalmente definidos.

Ejemplo:

ProfileUpdateInput

userId
name
lastName
phone
language
timezone

Debe definirse:

Type
Required
Optional
Format
Constraints
Sensitivity
Source
57.14 — Outputs

Los Outputs también deben ser explícitos.

ValidationResult

valid
errors
warnings
metadata

Un componente no debería devolver estructuras ambiguas.

57.15 — Component State

Algunos componentes son stateless:

Validator
Mapper
Formatter
Calculator

Otros necesitan estado:

Session Manager
Workflow State Manager
Recommendation Cache
Conversation Context

El estado debe clasificarse como:

Persistent
Transient
Cached
Session
Derived
External
57.16 — Component Logic

La lógica debe estar encapsulada.

Ejemplo:

Input
 ↓
Validation
 ↓
Business Rules
 ↓
Policy Check
 ↓
Transformation
 ↓
Output

La lógica del Component debe respetar las reglas del Domain.

57.17 — Domain Rules dentro de Components

Un componente puede implementar una regla del Domain.

Ejemplo:

Fitness Domain

Rule:
A workout cannot be completed
without all mandatory exercises.

Puede existir:

WorkoutCompletionValidator

que implemente esta regla.

La arquitectura queda:

Domain Rule
      ↓
Component
      ↓
Implementation
57.18 — Component Dependencies

Los componentes pueden depender de:

Other Components
Services
Capabilities
Contracts
Data Stores
External APIs
AI Models
Platform Services

Pero las dependencias deben ser explícitas.

57.19 — Dependency Injection

EVOXA debe favorecer componentes desacoplados mediante inyección de dependencias.

Conceptualmente:

Component
   ↓
Interface
   ↓
Implementation

Esto permite sustituir implementaciones.

Ejemplo:

RecommendationEngine
        ↓
Interface
   ┌────┴────┐
   ▼         ▼
RuleEngine  AIEngine
57.20 — Component Events

Los componentes pueden producir eventos.

Ejemplo:

ProfileUpdated
ValidationFailed
GoalEvaluated
WorkoutCompleted
RecommendationGenerated

Los eventos permiten integrar componentes sin crear dependencias directas innecesarias.

57.21 — Component Commands

Un componente puede ejecutar comandos internos.

ValidateProfile
CalculateProgress
GenerateRecommendation
UpdatePreference

Debe mantenerse la separación entre:

Command = Intent
Event = Fact
57.22 — Component Contracts

Las interfaces externas del componente deben estar protegidas por contratos.

Component
      ↓
Contract
      ↓
Consumer

El contrato define:

Inputs
Outputs
Errors
Security
Permissions
Version
Compatibility
Performance
57.23 — Component Error Model

Los errores deben ser estructurados.

Ejemplo:

VALIDATION_ERROR
AUTHORIZATION_ERROR
POLICY_ERROR
DEPENDENCY_ERROR
TIMEOUT
CONFLICT
RATE_LIMIT
EXTERNAL_ERROR
INTERNAL_ERROR

Cada error debería incluir:

Code
Message
Category
Retryable
Correlation ID
Context
57.24 — Idempotency

Los componentes que procesan operaciones repetibles deben soportar idempotencia cuando corresponda.

Ejemplo:

UpdateProfile

Si la misma solicitud llega dos veces:

Request A
Request A

el sistema debe evitar efectos duplicados cuando la operación lo requiera.

57.25 — Transactions

Cuando un componente modifica múltiples recursos, puede necesitar una transacción.

Operation
 ↓
Validate
 ↓
Execute
 ↓
Commit

Si existe una operación distribuida:

Component
 ↓
Service A
 ↓
Service B

pueden utilizarse:

Saga
Compensation
Eventual Consistency
57.26 — Component Resilience

Los componentes deben poder soportar fallos.

Mecanismos:

Timeout
Retry
Circuit Breaker
Fallback
Bulkhead
Rate Limit
Backpressure

No todos los componentes necesitan todos los mecanismos.

La política depende del riesgo y criticidad.

57.27 — Component Security

Security debe formar parte del diseño del componente.

Debe considerar:

Authentication
Authorization
Permissions
Policies
Data Access
Secrets
Encryption
Tenant Isolation
Privacy
Audit
57.28 — Component Permissions

Un componente puede requerir permisos.

Ejemplo:

profile.read
profile.update
profile.delete

Pero los permisos no deben estar codificados rígidamente cuando puedan ser gobernados externamente.

57.29 — Component Policy Evaluation

Antes de ejecutar una operación:

Request
 ↓
Identity
 ↓
Permission
 ↓
Policy
 ↓
Component

Esto permite autorización contextual.

57.30 — Component Observability

Cada componente importante debe ser observable.

Debe registrar:

Logs
Metrics
Traces
Events
Errors
Latency
Usage
Cost
Security Signals
57.31 — Component Metrics

Ejemplos:

Invocations
Success Rate
Error Rate
Latency
Throughput
Retries
Timeouts
Cache Hit Rate
Resource Usage

En componentes de IA:

Tokens
Model Latency
Model Cost
Confidence
Quality
Safety Events
57.32 — Component Health

Los componentes críticos deben reportar:

HEALTHY
DEGRADED
UNHEALTHY
UNKNOWN

La salud puede depender también de sus dependencias.

Component
   ↓
Dependency
   ↓
UNHEALTHY

puede provocar:

Component
   ↓
DEGRADED
57.33 — Component Performance

Debe medirse:

Latency
Throughput
CPU
Memory
I/O
Network
Cache
Queue Time

Esto permite detectar cuellos de botella.

57.34 — Component Testing

Los componentes deben poder probarse independientemente.

Tipos de pruebas:

Unit Tests
Integration Tests
Contract Tests
Security Tests
Performance Tests
Failure Tests
Property Tests

Para componentes de IA:

AI Evaluation
Safety Evaluation
Quality Evaluation
Regression Evaluation
57.35 — Component Testability

Un componente bien diseñado debe permitir:

Mock Dependencies
Inject Test Data
Simulate Failures
Replay Inputs
Validate Outputs
Measure Behavior

Esto reduce el costo de mantenimiento.

57.36 — Component Configuration

La configuración debe estar separada de la implementación.

Ejemplo:

enabled
timeout
retryPolicy
cacheTTL
limits
featureFlags
AIModel
AIParameters

La configuración puede depender de:

Platform
Organization
Tenant
Application
User
Environment
57.37 — Component Types

EVOXA puede clasificar User Components.

Core Components

Implementan coordinación o lógica central.

Business Components

Implementan reglas de negocio.

Data Components

Gestionan acceso o transformación de datos.

Integration Components

Conectan sistemas externos.

Communication Components

Gestionan canales de comunicación.

AI Components

Implementan capacidades de inteligencia.

Agent Components

Implementan herramientas o comportamientos para Agents.

Security Components

Implementan funciones de protección.

Workflow Components

Implementan pasos de procesos.

UI Components

Implementan elementos de experiencia.

57.38 — UI Components

Aunque la arquitectura Users & Experience no debe reducirse al frontend, los componentes de interfaz también forman parte del modelo.

Ejemplo:

Profile Screen
├── Profile Header
├── Avatar Component
├── Personal Information
├── Preferences
└── Save Action

Estos componentes deben consumir:

Capabilities
Contracts
Services

en lugar de acoplarse directamente a la base de datos.

57.39 — AI Components

Un AI Component puede representar:

Recommendation Generator
Intent Classifier
Context Summarizer
Goal Analyzer
Personalization Engine
Conversation Processor

Debe definir:

Model
Context
Input
Output
Tokens
Latency
Cost
Confidence
Safety
Privacy
57.40 — AI Component Governance

Todo componente de IA debe respetar:

Privacy
Consent
Data Boundaries
Model Policy
Safety
Risk
Cost
Explainability
Audit

El componente no debe utilizar datos simplemente porque técnicamente puede acceder a ellos.

57.41 — Agent Components

Los Agent Components pueden funcionar como herramientas controladas.

Ejemplo:

Workout Creation Tool
Goal Evaluation Tool
Profile Update Tool
Notification Tool

Deben declarar:

Agent Allowed
Risk Level
Permissions
Scope
Input Schema
Output Schema
Approval
Rate Limit
Cost Limit
Audit
57.42 — Agent Execution Boundary

La arquitectura debe ser:

Agent
  ↓
Tool Contract
  ↓
Permission
  ↓
Policy
  ↓
Capability
  ↓
Component
  ↓
Result

No:

Agent
  ↓
Direct Component Access

El Contract y Capability actúan como fronteras de control.

57.43 — Component Registry

EVOXA debe mantener un:

User Component Registry

con información como:

Component ID
Name
Version
Type
Purpose
Owner
Domain
Module
Interfaces
Dependencies
Capabilities
Contracts
Permissions
Policies
Events
Security
Observability
SLO
Cost
AI Metadata
Agent Metadata
Lifecycle
57.44 — Component Discovery

Los componentes pueden ser descubiertos por:

Type
Capability
Domain
Module
Version
Performance
Security
Cost
AI Readiness
Agent Readiness

Por ejemplo:

Find components capable of:
"calculate user progress"

La resolución puede devolver:

Progress Calculator v2
57.45 — Component Versioning

Los componentes deben tener versiones independientes.

Profile Validator
1.0.0
1.1.0
2.0.0

La evolución debe considerar:

Interface Compatibility
Contract Compatibility
Data Compatibility
Behavior Compatibility
57.46 — Component Replacement

Una de las ventajas de la arquitectura es poder reemplazar componentes.

Recommendation Component A
           ↓
       replaced by
           ↓
Recommendation Component B

Siempre que:

Contract
Capabilities
Policies
Security
Compatibility

se mantengan.

57.47 — Component Dependency Graph

EVOXA debe mantener un grafo:

Component Dependency Graph

Ejemplo:

Profile Writer
      ↓
Profile Validator
      ↓
Policy Engine
      ↓
Privacy Service

Esto permite realizar análisis de impacto.

57.48 — Component Impact Analysis

Antes de cambiar un componente:

Component Change
      ↓
Dependency Graph
      ↓
Affected Capabilities
      ↓
Affected Contracts
      ↓
Affected Modules
      ↓
Affected Domains
      ↓
Affected Applications
      ↓
Affected Agents

EVOXA puede identificar el impacto antes de ejecutar el cambio.

57.49 — Component Digital Twin

Los componentes críticos pueden disponer de una representación digital:

Component Digital Twin

con:

Version
Health
Dependencies
Usage
Performance
Cost
Errors
Security
Capabilities
Contracts
AI Usage
Agent Usage
Lifecycle

Esto proporciona una visión operacional en tiempo real.

57.50 — Intelligent Components

Un componente puede incorporar inteligencia.

Por ejemplo:

Recommendation Component

puede observar:

Usage
Quality
Latency
Cost
User Feedback

y determinar:

Which strategy performs better?
Which model is better?
Which provider is cheaper?
57.51 — Self-Optimizing Components

En etapas avanzadas:

Observe
   ↓
Analyze
   ↓
Predict
   ↓
Optimize
   ↓
Verify

Ejemplo:

AI Component

detecta que:

Model A
Cost = High
Latency = High
Quality = Similar

y puede recomendar:

Model B

La decisión final debe respetar las políticas del sistema.

57.52 — Autonomous Components

Los componentes pueden alcanzar cierto nivel de autonomía.

Ejemplo:

Component detects:
dependency failure

Puede ejecutar:

Fallback
Retry
Failover
Cache

sin intervención humana.

Pero las acciones de alto riesgo deben requerir aprobación.

57.53 — Component Lifecycle

Cada componente debe seguir un lifecycle.

PROPOSED
    ↓
DESIGNED
    ↓
REGISTERED
    ↓
VALIDATED
    ↓
APPROVED
    ↓
DEPLOYED
    ↓
ACTIVE
    ↓
EVOLVING
    ↓
DEPRECATED
    ↓
MIGRATING
    ↓
RETIRED
57.54 — Component Deprecation

Cuando un componente deja de ser recomendado:

ACTIVE
  ↓
DEPRECATED
  ↓
MIGRATING
  ↓
RETIRED

EVOXA debe conocer:

Who uses it?
What replaces it?
When does support end?
Which contracts change?
57.55 — Safe Component Retirement

Retirar un componente debe ser controlado.

Detect Dependencies
      ↓
Notify Consumers
      ↓
Prepare Replacement
      ↓
Migrate
      ↓
Validate
      ↓
Disable
      ↓
Retire

Esto evita eliminar componentes todavía utilizados.

57.56 — Component Governance

Los componentes deben estar sujetos a:

Architecture Governance
Security Governance
Data Governance
AI Governance
Agent Governance
Cost Governance
Lifecycle Governance
57.57 — Component Quality Score

EVOXA puede calcular un indicador:

Component Quality Score

basado en:

Reliability
Security
Performance
Test Coverage
Observability
Documentation
Compatibility
Cost
Maintainability
AI Readiness
Agent Readiness

Esto ayuda a identificar componentes que requieren modernización.

57.58 — Component Technical Debt

Puede medirse:

Technical Debt
├── Old Dependencies
├── Deprecated Contracts
├── Poor Tests
├── High Complexity
├── Security Findings
├── Performance Problems
├── High Cost
└── Architecture Violations

Esto permite que EVOXA priorice mejoras.

57.59 — Component Reuse

Una de las principales ventajas de los Components es la reutilización.

Ejemplo:

Profile Validator

puede ser utilizado por:

Web Application
Mobile Application
Trainer Application
Admin Application
AI Assistant
Agent

Siempre mediante contratos y capacidades apropiadas.

57.60 — Component Composition

Los componentes pueden combinarse.

Component A
     ↓
Component B
     ↓
Component C

formando una funcionalidad superior:

Composite Component

Por ejemplo:

Profile Update
├── Validator
├── Permission Checker
├── Privacy Checker
├── Writer
└── Event Publisher
57.61 — Component Composition Engine

EVOXA puede incorporar un motor capaz de:

Discover
Select
Validate
Compose
Configure
Execute
Monitor
Replace

componentes dinámicamente.

La selección puede considerar:

Compatibility
Performance
Cost
Security
Availability
Quality
Tenant
Region
Risk
57.62 — AI Component Selection

La IA puede resolver:

"I need a component
that calculates progress."

y buscar:

Capability
   ↓
Component Registry
   ↓
Compatible Components
   ↓
Evaluate
   ↓
Select

Esto permite una arquitectura descubrible por máquinas.

57.63 — Component Federation

Componentes externos podrían integrarse:

External Component
       ↓
Adapter
       ↓
Contract
       ↓
EVOXA Component Fabric

Debe verificarse:

Trust
Security
Compatibility
Privacy
Performance
Compliance
57.64 — Multi-Tenant Components

Los componentes deben poder definir su modelo de aislamiento.

Global
Organization
Tenant
Application
User

Un componente puede ser:

Shared

o:

Tenant-Isolated

dependiendo de su función.

57.65 — Component Cost Attribution

El costo debe poder asociarse a:

Component
↓
Capability
↓
Module
↓
Domain
↓
Application
↓
Tenant
↓
User

Esto permite comprender el costo real de una operación.

57.66 — User Component Fabric

Todos estos conceptos forman el:

USER COMPONENT FABRIC

que conecta:

Domains
   ↓
Modules
   ↓
Components
   ↓
Capabilities
   ↓
Contracts
   ↓
Runtime

y transversalmente:

Security
Governance
Observability
AI
Agents
Cost
Lifecycle
57.67 — Arquitectura completa

La arquitectura Users & Experience hasta este punto:

USER FOUNDATION
Defines the User
        ↓
USER CORE
Coordinates the User
        ↓
USER RUNTIME
Executes User Interaction
        ↓
USER SERVICES
Provides User Services
        ↓
USER MODULES
Organizes User Functionality
        ↓
USER DOMAINS
Contextualizes User Behavior
        ↓
USER COMPONENTS
Implements User Functionality
        ↓
USER CAPABILITIES
Exposes User Abilities
        ↓
USER CONTRACTS
Standardizes User Interaction
        ↓
USER LIFECYCLE
Evolves the User

La progresión ahora es:

DEFINE
   ↓
COORDINATE
   ↓
EXECUTE
   ↓
PROVIDE
   ↓
ORGANIZE
   ↓
CONTEXTUALIZE
   ↓
IMPLEMENT
   ↓
EXPOSE
   ↓
STANDARDIZE
   ↓
EVOLVE
57.68 — Maturity Model

Los User Components evolucionan mediante:

Level 1 — Basic

Componentes funcionales básicos.

Implementation
Interface
Tests
Level 2 — Structured

Se incorporan:

Ownership
Dependencies
Configuration
Boundaries
Level 3 — Governed

Se incorporan:

Security
Policies
Permissions
Contracts
Lifecycle
Level 4 — Observable

Se incorporan:

Metrics
Logs
Tracing
Health
SLO
Cost
Level 5 — Composable

Los componentes pueden:

Discover
Compose
Reuse
Replace
Version
Level 6 — Intelligent

Incorporan:

AI
Prediction
Optimization
Anomaly Detection
Level 7 — Agent-Enabled

Pueden ser utilizados de forma segura por Agents.

Tool Contracts
Permissions
Risk
Approval
Audit
Level 8 — Autonomous

Pueden:

Detect
Decide
Act
Verify
Recover

dentro de límites.

Level 9 — Adaptive

Pueden:

Observe
Learn
Predict
Adapt
Optimize
Evolve

manteniendo:

Security
Governance
Contracts
Compatibility
Human Control
57.69 — Principios fundamentales

Los User Components de EVOXA deben seguir:

1. Cohesion

Una responsabilidad clara.

2. Encapsulation

Los detalles internos permanecen protegidos.

3. Reusability

Los componentes deben poder reutilizarse.

4. Testability

Deben poder probarse independientemente.

5. Observability

Deben ser observables.

6. Security

Deben estar protegidos por diseño.

7. Contract First

La interacción debe utilizar contratos.

8. Capability Driven

La funcionalidad debe exponerse mediante capacidades.

9. AI Ready

Los componentes deben poder ser comprendidos por IA.

10. Agent Ready

Los componentes deben poder operar mediante Agents gobernados.

11. Lifecycle Managed

Deben tener lifecycle.

12. Cost Aware

Deben poder medir su costo.

13. Replaceable

Las implementaciones deben poder sustituirse.

14. Composable

Deben poder combinarse.

15. Evolvable

Deben poder evolucionar sin romper el ecosistema.

57.70 — Fórmula conceptual

Podemos resumir un User Component como:

USER COMPONENT
=
Responsibility
+
Interface
+
Inputs
+
Outputs
+
Logic
+
State
+
Dependencies
+
Policies
+
Permissions
+
Events
+
Contracts
+
Security
+
Observability
+
Testing
+
AI
+
Agents
+
Lifecycle
57.71 — Visión final

El objetivo de User Components no es simplemente dividir código en piezas pequeñas.

El objetivo es crear unidades de comportamiento inteligentes y gobernables que puedan formar parte de diferentes aplicaciones, dominios, capacidades y experiencias.

La visión final:

                     USER
                       │
                       ▼
                 USER DOMAIN
                       │
                       ▼
                 USER MODULE
                       │
              ┌────────┼────────┐
              ▼        ▼        ▼
          Component Component Component
              │        │        │
              └────────┼────────┘
                       ▼
                  CAPABILITIES
                       │
                       ▼
                   CONTRACTS
                       │
              ┌────────┴────────┐
              ▼                 ▼
             AI              AGENTS
              │                 │
              └────────┬────────┘
                       ▼
              INTELLIGENT COMPONENTS
                       │
                       ▼
              AUTONOMOUS COMPONENTS
                       │
                       ▼
                ADAPTIVE EVOXA
Definición oficial

User Components son unidades de implementación cohesivas, reutilizables, encapsuladas, testeables, observables y gobernadas que implementan la funcionalidad definida por User Modules y User Domains, exponiendo comportamiento mediante capacidades y contratos y estando preparadas para integración con IA y Agents.

Responsabilidad oficial

User Components = Implements User Functionality

Con esto queda establecida una separación muy importante:

DOMAIN
   = ¿Qué significa?

MODULE
   = ¿Qué funcionalidad agrupamos?

COMPONENT
   = ¿Cómo la implementamos?

CAPABILITY
   = ¿Qué podemos hacer?

CONTRACT
   = ¿Cómo interactuamos?

El siguiente capítulo lógico es 58 — User Capabilities, donde dejaremos de enfocarnos en cómo está implementada una funcionalidad y pasaremos a definir qué habilidades concretas EVOXA puede ofrecer al User, a las Applications, a la IA y a los Agents, independientemente de qué Components las implementen.
