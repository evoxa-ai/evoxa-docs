58 — User Capabilities
58.1 — Introducción

Los User Capabilities representan la capa que transforma toda la arquitectura anterior en habilidades concretas que EVOXA puede ofrecer, ejecutar y exponer en relación con un User.

Hasta este punto hemos construido:

User Foundation → define al User.
User Core → coordina al User.
User Runtime → ejecuta la interacción.
User Services → proporcionan servicios reutilizables.
User Modules → organizan la funcionalidad.
User Domains → contextualizan el comportamiento.
User Components → implementan la funcionalidad.
User Capabilities → exponen las habilidades.
User Contracts → estandarizan la interacción.
User Lifecycle → evoluciona todo el sistema.

Por lo tanto:

User Capabilities = Exponen las habilidades del User y del ecosistema para actuar sobre el User.

La diferencia fundamental es que un Capability no describe una implementación.

Describe una posibilidad de acción o comportamiento.

58.2 — Definición de User Capability

Una User Capability es una capacidad funcional explícita, identificable, gobernada, descubrible, medible y componible que permite realizar una acción o producir un resultado relacionado con un User.

Conceptualmente:

User Capability
├── Identity
├── Purpose
├── Inputs
├── Outputs
├── Preconditions
├── Postconditions
├── Rules
├── Policies
├── Permissions
├── Implementation
├── Components
├── Services
├── Contracts
├── Events
├── Security
├── Observability
├── Performance
├── Cost
├── AI Metadata
├── Agent Metadata
└── Lifecycle

Una Capability representa:

Qué puede hacer EVOXA.

No:

Cómo está construido EVOXA.

58.3 — Component vs Capability

La diferencia debe quedar completamente definida.

Component
¿Cómo se implementa?
Capability
¿Qué puede hacer?

Ejemplo:

Component:
Profile Update Component

implementa:

Capability:
profile.update

Otro componente podría reemplazarlo:

Profile Update Component v2

sin cambiar necesariamente:

profile.update

Por lo tanto:

Capability
      ↓
Implementation
      ↓
Component
58.4 — Capability como abstracción funcional

La Capability debe abstraer la implementación.

Por ejemplo:

Capability:
user.profile.update

puede ser implementada mediante:

Profile Module
    ↓
Profile Service
    ↓
Profile Validator
    ↓
Profile Writer

Pero el consumidor no necesita conocer esos detalles.

Solo necesita conocer:

user.profile.update
58.5 — Principio fundamental

La regla principal es:

Las Capabilities deben describir habilidades, no componentes.

Por ejemplo, es incorrecto exponer:

ProfileWriterComponent

como concepto de negocio.

Es mejor:

user.profile.update

Esto permite que las capacidades sean consumidas por:

Users
Applications
Services
Workflows
AI
Agents
External Systems
58.6 — Anatomía de una Capability

Toda Capability debe tener una definición formal.

Capability ID
Name
Namespace
Version
Type
Purpose
Owner
Status

Ejemplo:

Capability ID:
user.profile.update

Name:
Update User Profile

Type:
Business Capability

Version:
1.0.0

Status:
ACTIVE
58.7 — Purpose

El propósito explica qué hace la Capability.

Ejemplo:

Purpose:
Allow an authorized actor to update
the user's profile information.

Debe ser:

claro;
verificable;
independiente de implementación;
interpretable por humanos;
interpretable por IA.
58.8 — Inputs

Toda Capability debe definir sus entradas.

Ejemplo:

user.profile.update

Inputs:
userId
name
lastName
phone
language
timezone

Cada input debe definir:

Type
Required
Optional
Format
Constraints
Sensitivity
Source
58.9 — Outputs

La Capability debe declarar qué resultado produce.

Output:
UpdatedProfile

Puede incluir:

Entity
Status
Metadata
Warnings
Errors
58.10 — Preconditions

Las precondiciones determinan qué debe cumplirse antes de ejecutar la Capability.

Ejemplo:

Preconditions:

User exists
User is active
Actor is authenticated
Permission is valid
Policy allows operation
Input is valid
58.11 — Postconditions

Las postcondiciones determinan qué debe ser cierto después de la ejecución.

Ejemplo:

Postconditions:

Profile contains updated values
ProfileUpdated event emitted
Audit record created

Las precondiciones y postcondiciones permiten validar el comportamiento.

58.12 — Capability Rules

Las reglas definen el comportamiento funcional.

Ejemplo:

A profile update cannot modify
immutable identity attributes.

La Capability debe respetar las reglas definidas por su Domain.

58.13 — Capability Policies

Las políticas determinan qué está permitido.

Ejemplo:

Profile Update Policy
Privacy Policy
Tenant Policy
AI Data Policy
Agent Policy

Por lo tanto:

Rule
=
What must be true

Policy
=
What is allowed
58.14 — Permissions

Una Capability puede requerir permisos específicos.

Ejemplo:

user.profile.read
user.profile.update
user.profile.delete

La autorización debe evaluarse antes de ejecutar.

Identity
   ↓
Permission
   ↓
Policy
   ↓
Capability
58.15 — Capability Implementation

Una Capability puede tener múltiples implementaciones.

Capability
    │
    ├── Component A
    ├── Component B
    └── External Provider

Esto permite seleccionar la mejor implementación según:

Quality
Latency
Cost
Availability
Security
Region
Tenant
Risk
58.16 — Atomic Capabilities

Una Capability puede ser atómica.

Ejemplo:

profile.read
profile.update
goal.create
goal.complete
activity.record

Representan una acción concreta.

58.17 — Composite Capabilities

También puede existir una Capability compuesta.

Ejemplo:

user.onboarding.complete

que puede utilizar:

profile.create
preferences.initialize
consent.capture
goal.create
notification.enable

Conceptualmente:

Composite Capability
        ↓
Capability A
Capability B
Capability C
Capability D
58.18 — Capability Graph

EVOXA debe mantener un:

User Capability Graph

Ejemplo:

user.onboarding.complete
        │
        ├── profile.create
        ├── preferences.initialize
        ├── consent.capture
        └── goal.create

Esto permite conocer:

dependencias;
composición;
impacto;
orden de ejecución;
alternativas.
58.19 — Capability Categories

EVOXA puede clasificar las User Capabilities.

Identity
user.identity.resolve
user.identity.link
user.identity.verify
Profile
user.profile.create
user.profile.read
user.profile.update
user.profile.delete
user.profile.export
Preferences
user.preference.read
user.preference.update
user.preference.reset
Privacy
user.privacy.view
user.privacy.update
user.consent.grant
user.consent.revoke
user.data.export
user.data.delete
Security
user.session.manage
user.device.register
user.security.update
Communication
user.notification.send
user.message.send
user.email.send
user.push.send
Personalization
user.personalize
user.recommend
user.preference.infer
Activity
user.activity.record
user.activity.read
user.activity.analyze
Goals
user.goal.create
user.goal.update
user.goal.complete
user.goal.evaluate
Relationships
user.relationship.create
user.relationship.remove
user.relationship.list
AI
user.ai.ask
user.ai.recommend
user.ai.summarize
user.ai.analyze
Agents
user.agent.delegate
user.agent.approve
user.agent.revoke
user.agent.execute
58.20 — Business Capabilities

Las capacidades también pueden representar acciones de negocio.

Ejemplo:

fitness.goal.create
fitness.workout.plan
fitness.workout.start
fitness.workout.complete
fitness.progress.calculate

Estas capacidades pertenecen al contexto del Domain correspondiente.

58.21 — Analytical Capabilities

Pueden representar análisis.

user.activity.analyze
user.progress.analyze
user.behavior.analyze
user.performance.analyze

Pueden devolver:

Metrics
Insights
Patterns
Trends
Predictions
58.22 — AI Capabilities

Las AI Capabilities deben ser explícitas.

Ejemplo:

user.ai.recommend
user.ai.summarize
user.ai.classify
user.ai.predict
user.ai.generate
user.ai.explain

Cada una debe declarar:

Model Requirements
Context Requirements
Input
Output
Tokens
Latency
Cost
Confidence
Safety
Privacy
58.23 — Agent Capabilities

Los Agents deben consumir capacidades, no acceder directamente a componentes internos.

Ejemplo:

Agent
   ↓
user.goal.create
   ↓
Goal Module
   ↓
Goal Component

El Agent conoce:

Capability
Contract
Permission
Policy

pero no necesita conocer toda la implementación.

58.24 — Agent Permission Boundary

Una Agent Capability debe declarar:

Agent Allowed
Risk Level
Required Permission
Allowed Scope
Approval Required
Maximum Cost
Rate Limit
Data Access
Audit Required

Por ejemplo:

user.profile.update

Agent Allowed:
YES

Risk:
MEDIUM

Approval:
Required for sensitive fields
58.25 — Capability Discovery

EVOXA debe permitir descubrir capacidades.

Por ejemplo:

Find capabilities that:
- modify user goals
- are available to this tenant
- support agents
- require low risk

El sistema podría devolver:

user.goal.update
user.goal.pause
user.goal.complete
58.26 — Capability Registry

Debe existir un:

User Capability Registry

con:

Capability ID
Name
Version
Type
Purpose
Inputs
Outputs
Preconditions
Postconditions
Policies
Permissions
Implementations
Components
Services
Contracts
Events
Security
SLO
Cost
AI Metadata
Agent Metadata
Lifecycle

Este Registry será una fuente de verdad.

58.27 — Capability Resolution

Cuando alguien solicita una Capability:

Capability Request
        ↓
Capability Registry
        ↓
Candidate Implementations
        ↓
Compatibility
        ↓
Policy
        ↓
Security
        ↓
Cost
        ↓
Availability
        ↓
Selection

La plataforma puede seleccionar la implementación más apropiada.

58.28 — Intelligent Capability Routing

EVOXA puede realizar routing inteligente.

Por ejemplo:

Capability:
user.ai.recommend

Puede existir:

Model A
Model B
Model C

El sistema puede considerar:

Quality
Latency
Cost
Availability
Privacy
Region
Risk

y seleccionar dinámicamente.

58.29 — Capability Negotiation

Dos sistemas pueden negociar capacidades.

Ejemplo:

Consumer:
Need user.ai.recommend

Requirements:
Quality > 90
Latency < 500ms
Privacy = strict
Cost < X

EVOXA busca una implementación compatible.

Esto crea:

Capability Negotiation.

58.30 — Capability Preconditions and Invariants

Las capacidades deben declarar invariantes.

Ejemplo:

user.goal.complete

puede requerir:

Goal exists
Goal is active
Completion criteria satisfied
User has permission

Estas condiciones pueden ser verificadas automáticamente.

58.31 — Capability Errors

Las capacidades deben tener errores estructurados.

INVALID_INPUT
PRECONDITION_FAILED
PERMISSION_DENIED
POLICY_DENIED
RESOURCE_NOT_FOUND
CONFLICT
TIMEOUT
DEPENDENCY_FAILURE
RATE_LIMIT
INTERNAL_ERROR

Debe declararse:

Retryable
Recoverable
User Action Required
Approval Required
58.32 — Capability Idempotency

Las capacidades que soportan operaciones repetibles deben declarar si son idempotentes.

Idempotent:
YES

o:

Idempotent:
NO

Esto es especialmente importante para:

Payments
Notifications
Commands
External Integrations
Agent Actions
58.33 — Capability Events

Una Capability puede generar eventos.

Ejemplo:

user.goal.create
        ↓
GoalCreated

O:

user.profile.update
        ↓
ProfileUpdated

Esto permite construir flujos reactivos.

58.34 — Capability Security

Toda Capability debe estar protegida.

Identity
 ↓
Authentication
 ↓
Authorization
 ↓
Permission
 ↓
Policy
 ↓
Capability

Las capacidades de mayor riesgo deben incorporar controles adicionales.

58.35 — Capability Risk

Cada Capability puede tener un nivel de riesgo.

LOW
MEDIUM
HIGH
CRITICAL

Ejemplo:

profile.read → LOW
profile.update → MEDIUM
profile.delete → HIGH
user.data.export → HIGH
user.data.delete → CRITICAL

El nivel puede afectar:

permisos;
aprobación;
auditoría;
límites;
Agent access.
58.36 — Human-in-the-Loop

Las capacidades de alto riesgo pueden requerir intervención humana.

Agent
 ↓
Capability Request
 ↓
Risk Evaluation
 ↓
Human Approval
 ↓
Execution

Esto permite combinar autonomía con control.

58.37 — Capability Observability

Cada invocación debe poder observarse.

Capability Invocation
├── Timestamp
├── User
├── Tenant
├── Application
├── Actor
├── Agent
├── Implementation
├── Duration
├── Result
├── Error
├── Cost
└── Correlation ID

Esto crea trazabilidad completa.

58.38 — Capability Metrics

EVOXA puede medir:

Invocation Count
Success Rate
Error Rate
Latency
Throughput
Cost
Usage
Quality

Para IA:

Tokens
Model Cost
Confidence
Quality
Safety
58.39 — Capability SLO

Una Capability crítica puede tener:

Availability
Latency
Error Rate
Quality
Freshness

Por ejemplo:

user.profile.read
Availability: 99.99%

o:

user.ai.recommend
Latency: < 2 sec
Quality: > defined threshold
58.40 — Capability Cost

Cada Capability debe poder medir su costo.

Capability Cost
├── Compute
├── Storage
├── Network
├── AI Tokens
├── External API
└── Operational Cost

Esto permite:

Cost per Capability
Cost per User
Cost per Tenant
Cost per Application
Cost per Agent
58.41 — Capability Metering

EVOXA puede contabilizar utilización.

Ejemplo:

user.ai.ask
    1,000 invocations
    250,000 tokens
    $X cost

Esto puede alimentar:

Billing
FinOps
Quotas
Analytics
Usage Plans
58.42 — Capability Quotas

Las capacidades pueden tener límites.

Requests per minute
Requests per day
Tokens per day
Storage
Execution time
Cost budget

Estos límites pueden ser:

Global
Organization
Tenant
Application
User
Agent
58.43 — Capability Rate Limiting

Las capacidades críticas pueden tener rate limits.

Ejemplo:

user.ai.generate
100 requests / hour / user

Esto protege:

seguridad;
costos;
infraestructura;
proveedores externos.
58.44 — Capability Composition

Las capacidades pueden combinarse para construir operaciones superiores.

Ejemplo:

user.onboarding.complete

puede componerse de:

user.profile.create
user.preference.initialize
user.consent.capture
user.goal.create
user.notification.enable

La composición debe respetar:

Dependencies
Permissions
Policies
Transactions
Compensation
Contracts
58.45 — Workflow + Capabilities

Los workflows pueden consumir capacidades.

Workflow
    ↓
Capability A
    ↓
Capability B
    ↓
Capability C

Esto desacopla los workflows de las implementaciones.

58.46 — Capability Orchestration

Una Capability compuesta puede utilizar:

Sequential
Parallel
Conditional
Event-driven
Compensating

estrategias.

Ejemplo:

Onboarding
   ├── Create Profile
   ├── Initialize Preferences
   ├── Capture Consent
   └── Create Goal

Algunas operaciones pueden ejecutarse en paralelo.

58.47 — Capability Versioning

Las capacidades deben versionarse.

user.profile.update
v1
v2
v3

La evolución debe considerar:

Input Compatibility
Output Compatibility
Behavior Compatibility
Policy Compatibility
Contract Compatibility
58.48 — Capability Deprecation

Una Capability puede ser retirada.

ACTIVE
 ↓
DEPRECATED
 ↓
MIGRATING
 ↓
RETIRED

Antes de retirarla EVOXA debe saber:

Who uses it?
Which applications?
Which modules?
Which agents?
Which workflows?
58.49 — Capability Impact Analysis

El impacto puede representarse:

Capability Change
      ↓
Contracts
      ↓
Components
      ↓
Modules
      ↓
Domains
      ↓
Applications
      ↓
AI
      ↓
Agents

Esto permite evaluar riesgos antes de cambios.

58.50 — Capability Marketplace

En el futuro EVOXA puede ofrecer un:

Capability Marketplace

donde capacidades puedan:

Discover
Evaluate
Subscribe
Use
Compose
Monitor
Upgrade

Los criterios pueden incluir:

Quality
Security
Latency
Availability
Cost
Certification
Compatibility
58.51 — Capability Certification

Una Capability puede certificarse.

Security Certified
Privacy Certified
AI Certified
Agent Certified
Performance Certified
Compliance Certified
EVOXA Certified

Esto ayuda a seleccionar capacidades confiables.

58.52 — Capability Trust

EVOXA puede establecer un nivel de confianza.

Trust Score

basado en:

Reliability
Security
Quality
Usage
Certification
Incident History
Performance

Los Agents pueden utilizar esta información para seleccionar capacidades.

58.53 — AI Capability Discovery

La IA puede consultar:

What capabilities can help this User?

Por ejemplo:

User Intent:
"I want to improve my fitness."

La IA puede resolver:

fitness.goal.create
fitness.progress.analyze
fitness.workout.plan
fitness.recommendation.generate
58.54 — Agent Capability Discovery

Un Agent puede descubrir:

Available Capabilities

filtradas por:

Permissions
Risk
Scope
Tenant
Cost
Policies

El Agent no recibe acceso indiscriminado.

58.55 — Capability Selection

La selección puede utilizar:

Intent
Context
Requirements
Permissions
Policies
Quality
Cost
Latency
Availability
Risk

Conceptualmente:

Intent
  +
Context
  +
Constraints
       ↓
Capability Discovery
       ↓
Candidate Capabilities
       ↓
Policy Evaluation
       ↓
Selection
       ↓
Execution
58.56 — Capability Negotiation entre Agents

Dos Agents pueden interactuar mediante capacidades.

Agent A
   ↓
Required Capability
   ↓
Capability Registry
   ↓
Agent B
   ↓
Capability Contract

La interacción debe mantener:

Identity
Trust
Permissions
Policies
Contracts
Audit
58.57 — Capability Graph como Knowledge Graph

El Capability Graph puede conectarse con:

Domains
Modules
Components
Services
Contracts
Users
Applications
Agents

Por ejemplo:

User
 ↓
Intent
 ↓
Capability
 ↓
Domain
 ↓
Module
 ↓
Component

Esto proporciona una representación semántica de lo que EVOXA puede hacer.

58.58 — Capability Digital Twin

Una Capability crítica puede tener una representación digital:

Capability Digital Twin

que conozca:

Current Version
Health
Usage
Cost
Performance
Implementations
Dependencies
Consumers
Contracts
Security
AI Usage
Agent Usage
Lifecycle
58.59 — Intelligent Capabilities

Una Capability inteligente puede analizar su propio comportamiento.

Observe
 ↓
Analyze
 ↓
Predict
 ↓
Optimize

Puede detectar:

Performance Degradation
Cost Anomalies
Low Quality
Provider Failure
Unused Capacity
Security Risk
58.60 — Autonomous Capabilities

En una arquitectura avanzada una Capability puede seleccionar automáticamente cómo ejecutarse.

Capability Request
      ↓
Evaluate Implementations
      ↓
Select Best Option
      ↓
Execute
      ↓
Verify Result
      ↓
Learn

Por ejemplo:

AI Recommendation Capability

puede elegir dinámicamente:

Model A
Model B
Model C

según políticas.

58.61 — Adaptive Capabilities

Una Capability adaptativa puede cambiar su estrategia.

Observe Usage
      ↓
Detect Pattern
      ↓
Predict Demand
      ↓
Adapt Strategy
      ↓
Verify

Pero nunca debe cambiar arbitrariamente:

Security
Permissions
Core Policies
Contracts
Governance
58.62 — Capability Lifecycle

Las capacidades siguen el lifecycle de EVOXA:

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
58.63 — Capability Governance

Cada Capability debe estar gobernada por:

Security
Privacy
Permissions
Policies
Contracts
Quality
Performance
Cost
AI Governance
Agent Governance
Lifecycle
58.64 — Capability Quality

EVOXA puede calcular:

Capability Quality Score

considerando:

Reliability
Performance
Availability
Security
Quality
Compatibility
Cost Efficiency
Documentation
Observability
Certification
58.65 — Capability Technical Debt

También puede medirse:

Deprecated Implementations
Old Contracts
High Cost
Poor Reliability
Security Findings
Low Usage
Redundant Capabilities

Esto permite detectar capacidades que deben:

Optimize
Merge
Replace
Deprecate
Retire
58.66 — User Capability Fabric

Todos estos elementos forman:

USER CAPABILITY FABRIC

que conecta:

User
   ↓
Intent
   ↓
Capabilities
   ↓
Domains
   ↓
Modules
   ↓
Components
   ↓
Contracts

y transversalmente:

Security
Governance
Observability
AI
Agents
Cost
Lifecycle
58.67 — Arquitectura completa hasta el capítulo 58
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

La cadena conceptual queda:

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
58.68 — Fórmula conceptual

Un User Capability puede resumirse como:

USER CAPABILITY
=
Purpose
+
Inputs
+
Outputs
+
Preconditions
+
Postconditions
+
Rules
+
Policies
+
Permissions
+
Implementation
+
Contracts
+
Events
+
Security
+
Observability
+
Performance
+
Cost
+
AI
+
Agents
+
Lifecycle
58.69 — Principios fundamentales

Los User Capabilities de EVOXA deben seguir:

1. Capability First

La funcionalidad debe expresarse como capacidad.

2. Implementation Independent

La Capability no debe depender de una implementación específica.

3. Discoverable

Debe poder descubrirse.

4. Composable

Debe poder combinarse.

5. Governed

Debe estar gobernada.

6. Secure

Debe estar protegida.

7. Observable

Debe poder medirse.

8. Measurable

Debe tener métricas de uso, calidad y costo.

9. Versioned

Debe evolucionar mediante versiones.

10. AI-Readable

Debe ser interpretable por IA.

11. Agent-Ready

Debe definir claramente cómo puede ser utilizada por Agents.

12. Risk-Aware

Debe tener clasificación de riesgo.

13. Cost-Aware

Debe conocer su costo.

14. Replaceable

Su implementación debe poder cambiar.

15. Evolvable

Debe poder evolucionar sin romper consumidores.

58.70 — Visión final

El verdadero objetivo de User Capabilities es convertir EVOXA en una plataforma donde las funcionalidades puedan ser tratadas como habilidades descubribles y composables.

La evolución es:

Component
   ↓
Capability
   ↓
Capability Graph
   ↓
Capability Discovery
   ↓
Capability Composition
   ↓
AI Capability Selection
   ↓
Agent Capability Selection
   ↓
Autonomous Capability Execution

La visión final:

                         USER
                           │
                           ▼
                        INTENT
                           │
                           ▼
                   CAPABILITY DISCOVERY
                           │
              ┌────────────┼────────────┐
              ▼            ▼            ▼
          Profile        Goals       Activity
         Capability    Capability   Capability
              │            │            │
              └────────────┼────────────┘
                           ▼
                    CAPABILITY GRAPH
                           │
                           ▼
                     IMPLEMENTATION
                           │
                     ┌─────┴─────┐
                     ▼           ▼
                 COMPONENTS    SERVICES
                     │           │
                     └─────┬─────┘
                           ▼
                       EXECUTION
                           │
                  ┌────────┴────────┐
                  ▼                 ▼
                 AI              AGENTS
                  │                 │
                  └────────┬────────┘
                           ▼
               INTELLIGENT CAPABILITIES
                           │
                           ▼
                AUTONOMOUS CAPABILITIES
                           │
                           ▼
                 ADAPTIVE EVOXA
Definición oficial

User Capabilities son capacidades funcionales explícitas, gobernadas, descubribles, medibles y componibles que representan las habilidades que EVOXA puede ofrecer o ejecutar en relación con un User, independientemente de su implementación, y que pueden ser consumidas de forma segura por Users, Applications, Workflows, IA y Agents.

Responsabilidad oficial

User Capabilities = Exposes User Abilities

Y la separación arquitectónica queda ahora completamente clara:

DOMAIN
= ¿En qué contexto?

MODULE
= ¿Qué funcionalidad agrupamos?

COMPONENT
= ¿Cómo la implementamos?

CAPABILITY
= ¿Qué podemos hacer?

CONTRACT
= ¿Cómo acordamos hacerlo?

El siguiente capítulo será 59 — User Contracts, donde estableceremos el acuerdo formal entre Capabilities, Components, Services, Modules, Domains, Applications, Users, IA y Agents, definiendo interfaces, datos, errores, seguridad, compatibilidad, rendimiento, eventos y reglas de interacción.
