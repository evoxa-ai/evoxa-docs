95 — ROADMAP MODULES
95.1 — Definición

Roadmap Modules = Organizes Roadmap Functionality

Los Roadmap Modules constituyen la capa encargada de organizar la funcionalidad del sistema Roadmap en unidades cohesivas, autónomas, gobernadas, reutilizables y composables.

Mientras:

91 — ROADMAP FOUNDATION
↓
Defines Roadmap

92 — ROADMAP CORE
↓
Coordinates Roadmap

93 — ROADMAP RUNTIME
↓
Executes Roadmap

94 — ROADMAP SERVICES
↓
Provides Roadmap Services

95 — ROADMAP MODULES
↓
Organizes Roadmap Functionality

La distinción fundamental es:

DOMAIN     = ¿En qué contexto existe?
MODULE     = ¿Qué funcionalidad agrupamos?
COMPONENT  = ¿Cómo la implementamos?
CAPABILITY = ¿Qué puede hacer?
CONTRACT   = ¿Cómo acordamos la interacción?
LIFECYCLE  = ¿Cómo evoluciona?

Por lo tanto, un Module no es simplemente una carpeta de código.

Es una unidad funcional gobernada dentro de EVOXA.

95.2 — Propósito

El propósito de Roadmap Modules es dividir el enorme espacio funcional de Roadmap en unidades que puedan:

desarrollarse;
desplegarse;
configurarse;
activarse;
desactivarse;
versionarse;
observarse;
asegurarse;
evolucionar;
reutilizarse;
componerse;
gobernarse.

La arquitectura debe evitar un único bloque monolítico:

ROADMAP
└── EVERYTHING

y evolucionar hacia:

ROADMAP
├── Strategy
├── Planning
├── Objectives
├── Initiatives
├── Dependencies
├── Resources
├── Risk
├── Execution
├── Outcomes
├── Intelligence
├── AI
└── Agents
95.3 — Roadmap Module como entidad

Cada Module debe ser una entidad de primera clase.

Roadmap Module
├── Identity
├── Name
├── Purpose
├── Description
├── Type
├── Scope
├── Ownership
├── Domain
├── Capabilities
├── Services
├── Components
├── Data
├── Policies
├── Permissions
├── Rules
├── Commands
├── Queries
├── Events
├── Workflows
├── Dependencies
├── Integrations
├── Configuration
├── Security
├── Risk
├── Observability
├── Health
├── Reliability
├── SLO
├── Performance
├── Resources
├── Cost
├── AI Metadata
├── Agent Metadata
├── Version
├── Compatibility
├── Contracts
├── Activation State
├── Lifecycle
└── Governance
95.4 — Module vs Service

Es importante no confundir ambos conceptos.

Module

Agrupa una funcionalidad completa y coherente.

Service

Proporciona una función reutilizable.

Ejemplo:

PLANNING MODULE
│
├── Planning Service
├── Schedule Service
├── Dependency Service
├── Resource Planning Service
│
├── Planning Components
└── Planning Capabilities

Por lo tanto:

MODULE
    ↓
contains
    ↓
SERVICES
    ↓
implemented by
    ↓
COMPONENTS
95.5 — Module vs Domain

También debemos mantener la separación conceptual:

DOMAIN
=
Contexto y significado

MODULE
=
Agrupación funcional

Por ejemplo:

Planning Domain
        │
        ├── Planning Module
        ├── Resource Planning Module
        └── Scenario Planning Module

El Domain define el contexto semántico.

El Module organiza la funcionalidad necesaria dentro de ese contexto.

95.6 — Arquitectura de Roadmap Modules
ROADMAP MODULES
│
├── Strategy Module
├── Vision Module
├── Objectives Module
├── Roadmap Management Module
├── Planning Module
├── Phase Management Module
├── Initiative Management Module
├── Program Management Module
├── Project Management Module
├── Milestone Management Module
├── Dependency Management Module
├── Priority Management Module
├── Risk Management Module
├── Resource Management Module
├── Budget & Cost Management Module
├── Value Management Module
├── Progress Management Module
├── Outcome Management Module
├── Scenario Planning Module
├── Forecasting Module
├── Decision Management Module
├── Change Management Module
├── Execution Management Module
├── Release & Deployment Planning Module
├── Governance Module
├── Reporting & Analytics Module
├── AI Roadmap Module
├── Agent Roadmap Module
└── Roadmap Intelligence Module
95.7 — 29 módulos oficiales

Para mantener la arquitectura completa y consistente, proponemos 29 Roadmap Modules:

01 Strategy Management
02 Vision Management
03 Objective Management
04 Roadmap Management
05 Planning Management
06 Phase Management
07 Initiative Management
08 Program Management
09 Project Management
10 Milestone Management
11 Dependency Management
12 Priority Management
13 Risk Management
14 Resource Management
15 Budget & Cost Management
16 Value Management
17 Progress Management
18 Outcome Management
19 Scenario Planning
20 Forecasting
21 Decision Management
22 Change Management
23 Execution Management
24 Release & Deployment Planning
25 Governance
26 Reporting & Analytics
27 AI Roadmap
28 Agent Roadmap
29 Roadmap Intelligence
95.8 — Module 01 — Strategy Management

Responsabilidad:

Conectar el Roadmap con la estrategia de EVOXA.

Funciones:

Strategy
├── Vision
├── Strategic Goals
├── Strategic Objectives
├── Strategic Priorities
├── Strategic Initiatives
└── Strategic Outcomes

Capacidades:

definir estrategia;
mapear estrategia;
alinear Roadmaps;
medir alineación;
detectar desalineación.

Flujo:

VISION
↓
STRATEGY
↓
OBJECTIVES
↓
ROADMAP
95.9 — Module 02 — Vision Management

Administra la visión futura.

Permite:

crear visión;
definir target state;
actualizar visión;
comparar visión;
versionar visión;
analizar impacto.

Modelo:

Current State
        ↓
Vision
        ↓
Target State
95.10 — Module 03 — Objective Management

Administra objetivos.

Objective
├── Identity
├── Purpose
├── Owner
├── Priority
├── Metrics
├── Target
├── Deadline
├── Initiatives
├── Outcomes
└── Status

Funciones:

crear;
priorizar;
medir;
alinear;
dividir;
completar;
cancelar.
95.11 — Module 04 — Roadmap Management

Es el módulo central de administración.

Funciones:

Create Roadmap
Update Roadmap
Clone Roadmap
Version Roadmap
Publish Roadmap
Archive Roadmap
Compare Roadmaps
Validate Roadmap

Puede administrar:

metadata;
ownership;
versiones;
estados;
snapshots;
escenarios.
95.12 — Module 05 — Planning Management

Transforma objetivos en planes.

OBJECTIVE
↓
CHANGE
↓
INITIATIVE
↓
PROGRAM
↓
PROJECT
↓
MILESTONE
↓
DELIVERABLE

Incluye:

schedule planning;
dependency planning;
resource planning;
capacity planning;
delivery planning.
95.13 — Module 06 — Phase Management

Administra fases del Roadmap.

Phase
├── Objectives
├── Initiatives
├── Milestones
├── Dependencies
├── Resources
├── Budget
├── Risks
└── Outcomes

Permite:

crear;
iniciar;
pausar;
reanudar;
completar;
cerrar;
evaluar.
95.14 — Module 07 — Initiative Management

Administra iniciativas estratégicas.

Initiative
├── Objective
├── Priority
├── Projects
├── Dependencies
├── Resources
├── Budget
├── Risks
├── Milestones
├── Deliverables
└── Outcomes

Estados:

PROPOSED
READY
ACTIVE
BLOCKED
AT_RISK
PAUSED
COMPLETED
CANCELLED
95.15 — Module 08 — Program Management

Agrupa proyectos relacionados.

Program
↓
Initiatives
↓
Projects
↓
Milestones
↓
Outcomes

Permite:

coordinación;
seguimiento;
presupuesto;
recursos;
riesgos;
dependencias.
95.16 — Module 09 — Project Management

Conecta el Roadmap con proyectos concretos.

Project
├── Scope
├── Team
├── Resources
├── Budget
├── Schedule
├── Risks
├── Dependencies
├── Milestones
└── Deliverables

El módulo no necesita reemplazar una herramienta completa de gestión de proyectos; proporciona la capa estratégica y de coordinación.

95.17 — Module 10 — Milestone Management

Administra puntos verificables.

Milestone
├── Target
├── Deadline
├── Criteria
├── Evidence
├── Validation
├── Dependencies
└── Outcome

Permite detectar:

milestone achieved;
delayed;
blocked;
at risk.
95.18 — Module 11 — Dependency Management

Administra el grafo de dependencias.

Dependency
├── Source
├── Target
├── Type
├── Criticality
├── Status
├── Owner
├── Risk
└── Impact

Funciones:

Discover
Create
Analyze
Validate
Monitor
Resolve
Remove
95.19 — Module 12 — Priority Management

Administra prioridades.

Strategic Value
+
Business Value
+
User Value
+
Urgency
+
Risk Reduction
+
Dependency Criticality
-
Cost
-
Complexity
-
Risk

Funciones:

rank;
reprioritize;
compare;
simulate;
optimize.
95.20 — Module 13 — Risk Management

Administra riesgos.

Risk
├── Type
├── Probability
├── Impact
├── Score
├── Mitigation
├── Owner
├── Status
└── Escalation

Puede detectar:

Technical
Architecture
Security
Operational
Financial
Schedule
Dependency
Data
Compliance
AI
Agent
Organizational
95.21 — Module 14 — Resource Management

Administra recursos.

Resources
├── People
├── Teams
├── Compute
├── GPU
├── Storage
├── Network
├── Cloud
├── Data
├── AI Models
└── Budget

Funciones:

allocate;
reserve;
forecast;
rebalance;
release;
optimize.
95.22 — Module 15 — Budget & Cost Management

Gestiona costos.

Roadmap
↓
Initiative
↓
Project
↓
Resources
↓
Infrastructure
↓
AI
↓
Operations
↓
Total Cost

Permite:

budget;
cost estimation;
actual cost;
forecast;
variance;
optimization.
95.23 — Module 16 — Value Management

Conecta inversión con valor.

Expected Value
↓
Investment
↓
Execution
↓
Delivered Value
↓
Measured Outcome

Puede calcular:

business value;
user value;
technical value;
strategic value;
security value;
operational value;
ROI.
95.24 — Module 17 — Progress Management

Gestiona progreso.

Expected
   vs
Actual

Incluye:

progress;
variance;
velocity;
blockers;
delays;
milestone completion;
deliverables.
95.25 — Module 18 — Outcome Management

Gestiona resultados reales.

Diferencia:

DELIVERABLE
=
Lo que construimos

OUTCOME
=
Lo que conseguimos

Ejemplo:

Deliverable:
Nueva aplicación.

Outcome:
Mayor adopción y productividad.
95.26 — Module 19 — Scenario Planning

Permite explorar futuros posibles.

Current State
↓
Scenario Engine
├── Baseline
├── Accelerated
├── Cost Optimized
├── Risk Minimized
└── AI Accelerated

Cada escenario puede tener:

schedule;
cost;
resources;
risk;
value;
outcomes.
95.27 — Module 20 — Forecasting

Predice evolución futura.

Historical Data
+
Current State
+
Progress
+
Dependencies
+
Resources
+
Risk
↓
Forecast

Predicciones:

completion date;
delay;
cost;
resource demand;
risk;
outcome probability.
95.28 — Module 21 — Decision Management

Gestiona decisiones estratégicas.

Decision
├── Context
├── Options
├── Criteria
├── Risk
├── Cost
├── Value
├── Recommendation
├── Approval
└── Result

Decisiones:

Prioritize
Deprioritize
Accelerate
Defer
Block
Unblock
Replan
Cancel
Replace
Merge
Split
Approve
Reject
95.29 — Module 22 — Change Management

Administra cambios.

CHANGE
↓
CLASSIFY
↓
IMPACT
↓
RISK
↓
APPROVAL
↓
EXECUTION
↓
VERIFY

Tipos:

routine;
minor;
major;
security;
emergency;
regulatory;
breaking;
architecture;
data migration;
infrastructure;
AI;
Agent.
95.30 — Module 23 — Execution Management

Es la conexión funcional con Roadmap Runtime.

Roadmap
↓
Execution Management
↓
Roadmap Runtime
↓
Execution

Permite:

iniciar;
detener;
pausar;
reanudar;
cancelar;
reintentar;
rollback;
monitorizar.
95.31 — Module 24 — Release & Deployment Planning

Conecta Roadmap con Engineering y Operations.

Roadmap
↓
Initiative
↓
Release
↓
Deployment
↓
Production

Permite:

release planning;
deployment planning;
rollout strategy;
release dependencies;
release risk.
95.32 — Module 25 — Governance

Administra las reglas del Roadmap.

Governance
├── Policies
├── Permissions
├── Approvals
├── Risk
├── Compliance
├── Audit
├── Ownership
├── Controls
└── Evidence

Todo cambio crítico debe pasar por Governance.

95.33 — Module 26 — Reporting & Analytics

Transforma información del Roadmap en información consumible.

Roadmap Data
↓
Analytics
↓
Metrics
↓
Reports
↓
Insights

Reportes:

executive;
strategic;
progress;
risk;
cost;
resource;
dependency;
outcome;
AI;
Agent.
95.34 — Module 27 — AI Roadmap

Este módulo introduce inteligencia artificial especializada.

AI Roadmap
├── Analysis
├── Planning
├── Prioritization
├── Dependency Analysis
├── Risk Analysis
├── Scenario Generation
├── Forecasting
├── Optimization
├── Progress Analysis
├── Outcome Analysis
└── Recommendations

AI puede:

analizar;
recomendar;
predecir;
explicar;
optimizar.

Pero:

AI no obtiene autoridad automáticamente por producir una recomendación.

95.35 — Module 28 — Agent Roadmap

Este módulo permite incorporar Agents.

Agent Roadmap
├── Agent Identity
├── Goals
├── Capabilities
├── Tools
├── Permissions
├── Policies
├── Scope
├── Risk
├── Budget
├── Approval
├── Memory
└── Audit

Ejemplos:

Roadmap Planning Agent
Dependency Analysis Agent
Risk Agent
Resource Optimization Agent
Cost Optimization Agent
Execution Agent
Outcome Analysis Agent
95.36 — Module 29 — Roadmap Intelligence

Representa la capa avanzada de inteligencia.

Roadmap Intelligence
├── Strategic Intelligence
├── Execution Intelligence
├── Risk Intelligence
├── Dependency Intelligence
├── Resource Intelligence
├── Cost Intelligence
├── Value Intelligence
├── Schedule Intelligence
├── AI Intelligence
└── Agent Intelligence
95.37 — Module Anatomy

Todos los módulos deben seguir una estructura común.

Roadmap Module
│
├── Identity
├── Purpose
├── Scope
├── Context
├── Ownership
│
├── Domains
├── Services
├── Components
├── Capabilities
├── Contracts
│
├── Data
├── Commands
├── Queries
├── Events
├── Workflows
│
├── Policies
├── Permissions
├── Rules
├── Security
├── Risk
│
├── Dependencies
├── Integrations
├── Configuration
│
├── Observability
├── Health
├── Reliability
├── SLO
├── Performance
├── Cost
│
├── AI Metadata
├── Agent Metadata
│
├── Version
├── Compatibility
├── Activation
├── Lifecycle
└── Governance
95.38 — Module Communication

Los módulos deben comunicarse mediante mecanismos explícitos:

API
Events
Messages
Commands
Queries
Capabilities
Contracts
Workflows

Evitar:

Module A
   ↓
direct internal database access
   ↓
Module B

Preferir:

Module A
   ↓
Contract
   ↓
Service / Capability
   ↓
Module B
95.39 — Module Dependencies

Los módulos deben declarar sus dependencias.

Ejemplo:

Planning Module
├── Objective Module
├── Dependency Module
├── Resource Module
├── Risk Module
└── Cost Module

El grafo debe permitir detectar:

ciclos;
dependencias críticas;
módulos aislados;
puntos únicos de fallo;
impactos de cambio.
95.40 — Module Registry

EVOXA debe disponer de un:

Roadmap Module Registry

Module Registry
├── Module Identity
├── Name
├── Description
├── Type
├── Owner
├── Domain
├── Services
├── Capabilities
├── Components
├── Contracts
├── Dependencies
├── Version
├── Compatibility
├── Permissions
├── Policies
├── Health
├── SLO
├── Cost
├── AI Metadata
├── Agent Metadata
└── Lifecycle

Esto permite discovery dinámico.

95.41 — Module Activation

Un módulo puede activarse dependiendo del contexto.

Estados:

INSTALLED
↓
ENABLED
↓
ACTIVE

También:

DISABLED
DEGRADED
SUSPENDED
DEPRECATED
RETIRED

Esto permite que diferentes tenants o aplicaciones utilicen diferentes módulos.

95.42 — Module Health

Estado de salud:

HEALTHY
DEGRADED
AT_RISK
FAILED
UNKNOWN

Debe existir separación entre:

Activation State
Operational State
Health State
Lifecycle State
Security State
Risk State
95.43 — Module Configuration

Cada módulo debe tener configuración gobernada.

Module Configuration
├── Defaults
├── Tenant Settings
├── Organization Settings
├── Feature Flags
├── Policies
├── Limits
└── AI Settings

Jerarquía:

Platform
↓
Organization
↓
Tenant
↓
Application
↓
Roadmap
↓
Module
95.44 — Module Security

Todo módulo debe estar protegido.

User / Application / Agent
↓
Identity
↓
Authentication
↓
Authorization
↓
Policy
↓
Risk
↓
Module

Los permisos deben poder definirse a nivel de:

módulo;
servicio;
capability;
operación;
recurso.
95.45 — Module Events

Eventos principales:

ModuleRegistered
ModuleEnabled
ModuleDisabled
ModuleActivated
ModuleSuspended
ModuleDegraded
ModuleRecovered
ModuleVersionChanged
ModuleDependencyChanged
ModulePolicyChanged
ModuleDeprecated
ModuleRetired
95.46 — Module Commands
RegisterModule
EnableModule
DisableModule
ActivateModule
SuspendModule
UpdateModule
UpgradeModule
ValidateModule
DeprecateModule
RetireModule
95.47 — Module Queries
GetModule
GetModules
SearchModules
GetModuleHealth
GetModuleDependencies
GetModuleServices
GetModuleCapabilities
GetModuleContracts
GetModuleVersion
GetModuleCost
GetModuleUsage
GetModuleMetrics
95.48 — Module Composition

Los módulos pueden combinarse para crear funcionalidades mayores.

Ejemplo:

STRATEGIC PLANNING
│
├── Strategy Module
├── Objective Module
├── Planning Module
├── Dependency Module
├── Risk Module
├── Resource Module
├── Cost Module
└── Scenario Module

Esto produce una composición funcional:

STRATEGIC PLANNING CAPABILITY
95.49 — Module Digital Twin

Cada módulo puede tener representación digital.

MODULE DIGITAL TWIN
├── Current State
├── Desired State
├── Dependencies
├── Resources
├── Health
├── Risk
├── Cost
├── Usage
├── Performance
├── Capabilities
└── Lifecycle

Esto permite simulación e impacto.

95.50 — Module Impact Analysis

Cuando se cambia un módulo:

MODULE CHANGE
↓
SERVICES
↓
CAPABILITIES
↓
COMPONENTS
↓
CONTRACTS
↓
DEPENDENCIES
↓
INITIATIVES
↓
PROJECTS
↓
ROADMAPS

EVOXA puede conocer el impacto antes de ejecutar el cambio.

95.51 — AI Module Discovery

AI puede consultar:

"What module handles roadmap risk?"

El sistema responde mediante:

Intent
↓
Capability
↓
Module Registry
↓
Risk Management Module
↓
Compatible Service
↓
Contract
↓
Execution
95.52 — Agent Module Discovery

Un Agent puede descubrir módulos de forma controlada:

Agent
↓
Intent
↓
Capability
↓
Module
↓
Service
↓
Contract
↓
Permission
↓
Policy
↓
Execution

Esto evita que los Agents dependan de implementaciones concretas.

95.53 — Module Marketplace

EVOXA puede evolucionar hacia un marketplace interno o federado.

Module Marketplace
├── Discover
├── Evaluate
├── Compare
├── Install
├── Validate
├── Enable
├── Upgrade
├── Deprecate
└── Retire

Los módulos podrían ser:

core;
optional;
tenant-specific;
industry-specific;
partner-provided;
AI-powered.
95.54 — Module Certification

Antes de entrar en producción:

MODULE
↓
SECURITY VALIDATION
↓
QUALITY
↓
CONTRACT VALIDATION
↓
PERFORMANCE
↓
RELIABILITY
↓
COMPATIBILITY
↓
CERTIFICATION
↓
PRODUCTION
95.55 — Module Governance

Todo módulo debe tener:

Owner
Policy
Permissions
Risk
Compliance
Audit
SLO
Security
Lifecycle

Esto evita módulos huérfanos.

95.56 — Module FinOps

Los módulos deben ser económicamente observables.

Module
↓
Usage
↓
Resources
↓
Cost
↓
Value

Esto permite:

Cost per Module
Cost per Tenant
Cost per Roadmap
Cost per Initiative
Cost per Capability
95.57 — Module AI Metadata

Para que AI pueda comprender módulos:

AI Metadata
├── Purpose
├── Semantic Description
├── Capabilities
├── Inputs
├── Outputs
├── Constraints
├── Risks
├── Cost
├── Performance
└── Examples

Esto convierte los módulos en entidades AI-readable.

95.58 — Module Agent Metadata

Para Agents:

Agent Metadata
├── Allowed Agents
├── Allowed Capabilities
├── Required Permissions
├── Policies
├── Risk Level
├── Scope
├── Approval Rules
├── Cost Limits
├── Resource Limits
└── Audit Requirements
95.59 — Module Lifecycle

El lifecycle estándar:

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

La implementación concreta del lifecycle será responsabilidad de:

100 — Roadmap Lifecycle.

95.60 — Intelligent Modules

Los módulos pueden evolucionar:

Static Module
↓
Configured Module
↓
Observable Module
↓
Composable Module
↓
Discoverable Module
↓
AI-Assisted Module
↓
Intelligent Module
↓
Predictive Module
↓
Autonomous Module
↓
Adaptive Module
↓
Self-Evolving Module
95.61 — Roadmap Module Intelligence

En una arquitectura avanzada:

MODULE
↓
OBSERVE
↓
ANALYZE
↓
DETECT PATTERN
↓
PREDICT
↓
RECOMMEND
↓
OPTIMIZE

Ejemplo:

Risk Module
↓
detects increasing risk
↓
AI analyzes cause
↓
Scenario Module simulates options
↓
Decision Module evaluates
↓
Governance approves
↓
Execution Module acts
95.62 — Autonomous Module Coordination

Los módulos pueden coordinarse mediante Agents:

Roadmap Agent
      ↓
Objective Module
      ↓
Planning Module
      ↓
Dependency Module
      ↓
Risk Module
      ↓
Resource Module
      ↓
Cost Module
      ↓
Scenario Module
      ↓
Decision Module
      ↓
Governance Module
      ↓
Execution Module

Esto crea una arquitectura de modular autonomous planning.

95.63 — Module Evolution Loop
OBSERVE MODULE
        ↓
ANALYZE USAGE
        ↓
ANALYZE PERFORMANCE
        ↓
ANALYZE COST
        ↓
ANALYZE RISK
        ↓
IDENTIFY IMPROVEMENT
        ↓
SIMULATE
        ↓
VALIDATE
        ↓
APPROVE
        ↓
UPGRADE
        ↓
MEASURE
        ↓
LEARN
95.64 — Roadmap Modules Maturity
Nivel 1 — Basic

Módulos funcionales.

Nivel 2 — Structured

Interfaces y límites definidos.

Nivel 3 — Governed

Permisos, políticas y ownership.

Nivel 4 — Observable

Health, métricas y tracing.

Nivel 5 — Reusable

Módulos reutilizables.

Nivel 6 — Composable

Módulos componibles.

Nivel 7 — Discoverable

Registry y discovery.

Nivel 8 — Intelligent

AI-assisted.

Nivel 9 — Predictive

Predicción y optimización.

Nivel 10 — Autonomous

Agents pueden operar módulos bajo gobernanza.

Nivel 11 — Adaptive

Módulos adaptan su comportamiento.

Nivel 12 — Self-Evolving

Módulos participan en su propia evolución controlada.

95.65 — Arquitectura Final
95 — ROADMAP MODULES
│
├── 01 Strategy Management
├── 02 Vision Management
├── 03 Objective Management
├── 04 Roadmap Management
├── 05 Planning Management
├── 06 Phase Management
├── 07 Initiative Management
├── 08 Program Management
├── 09 Project Management
├── 10 Milestone Management
├── 11 Dependency Management
├── 12 Priority Management
├── 13 Risk Management
├── 14 Resource Management
├── 15 Budget & Cost Management
├── 16 Value Management
├── 17 Progress Management
├── 18 Outcome Management
├── 19 Scenario Planning
├── 20 Forecasting
├── 21 Decision Management
├── 22 Change Management
├── 23 Execution Management
├── 24 Release & Deployment Planning
├── 25 Governance
├── 26 Reporting & Analytics
├── 27 AI Roadmap
├── 28 Agent Roadmap
└── 29 Roadmap Intelligence
95.66 — Arquitectura jerárquica

La posición de Modules dentro de Roadmap queda:

ROADMAP FOUNDATION
        ↓
ROADMAP CORE
        ↓
ROADMAP RUNTIME
        ↓
ROADMAP SERVICES
        ↓
ROADMAP MODULES
        ↓
ROADMAP DOMAINS
        ↓
ROADMAP COMPONENTS
        ↓
ROADMAP CAPABILITIES
        ↓
ROADMAP CONTRACTS
        ↓
ROADMAP LIFECYCLE

Pero conceptualmente los módulos se relacionan internamente así:

MODULE
├── Domains
├── Services
├── Components
├── Capabilities
└── Contracts
95.67 — Relación con el resto de EVOXA
                         EVOXA
                           │
                    ROADMAP MODULES
                           │
        ┌──────────────────┼──────────────────┐
        ↓                  ↓                  ↓
    STRATEGY           PLANNING           EXECUTION
        │                  │                  │
        └──────────────────┼──────────────────┘
                           ↓
                      ENGINEERING
                           ↓
                       SECURITY
                           ↓
                      OPERATIONS
                           ↓
                        PLATFORM
                           ↓
                     APPLICATIONS
                           ↓
                         USERS
                           ↓
                       OUTCOMES

Así, los Roadmap Modules se convierten en una de las principales capas de organización funcional de la evolución de EVOXA.

95.68 — Principio fundamental

Roadmap Modules organizan la funcionalidad del sistema Roadmap en unidades cohesivas, autónomas, gobernadas, reutilizables y componibles, permitiendo que estrategia, planificación, ejecución, inteligencia, AI y Agents evolucionen sin convertir el Roadmap en un sistema monolítico.

95.69 — Responsabilidad oficial
95 — ROADMAP MODULES
        ↓
ORGANIZES ROADMAP FUNCTIONALITY

La separación oficial queda:

91 — ROADMAP FOUNDATION
↓
Defines Roadmap

92 — ROADMAP CORE
↓
Coordinates Roadmap

93 — ROADMAP RUNTIME
↓
Executes Roadmap

94 — ROADMAP SERVICES
↓
Provides Roadmap Services

95 — ROADMAP MODULES
↓
Organizes Roadmap Functionality

96 — ROADMAP DOMAINS
↓
Contextualizes Roadmap

97 — ROADMAP COMPONENTS
↓
Implements Roadmap Functionality

98 — ROADMAP CAPABILITIES
↓
Exposes Roadmap Abilities

99 — ROADMAP CONTRACTS
↓
Standardizes Roadmap Interaction

100 — ROADMAP LIFECYCLE
↓
Evolves Roadmap

Con esto, 95 — Roadmap Modules queda establecido como la capa que organiza funcionalmente Roadmap y prepara la transición hacia 96 — Roadmap Domains, donde dejaremos de preguntar “qué funcionalidad agrupamos” para pasar a definir “en qué contexto semántico y funcional existe esa capacidad dentro de EVOXA”.
