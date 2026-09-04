91 — Roadmap Foundation
91.1 — Definición

Roadmap Foundation establece las bases conceptuales, estructurales, semánticas y de gobierno que permiten a EVOXA definir, representar, administrar y evolucionar su Roadmap como una entidad de primera clase.

El Foundation no ejecuta iniciativas ni administra directamente proyectos.

Su responsabilidad es definir qué es un Roadmap, qué elementos lo componen, cuáles son sus límites, qué reglas lo gobiernan, cómo se relaciona con el resto de EVOXA y cuáles son las condiciones necesarias para que pueda evolucionar de forma controlada.

91 — Roadmap Foundation = Defines Roadmap

91.2 — Propósito

El propósito de Roadmap Foundation es establecer una estructura común para representar:

VISION
   ↓
STRATEGY
   ↓
OBJECTIVES
   ↓
ROADMAP
   ↓
PHASES
   ↓
INITIATIVES
   ↓
EPICS
   ↓
FEATURES
   ↓
CAPABILITIES
   ↓
DELIVERABLES
   ↓
RELEASES
   ↓
OUTCOMES

El Foundation debe garantizar que esta estructura sea:

coherente;
trazable;
gobernada;
medible;
versionable;
auditable;
interoperable;
segura;
extensible;
AI-readable;
Agent-ready;
evolutiva.
91.3 — Roadmap como entidad de EVOXA

Dentro de EVOXA, el Roadmap es una entidad estratégica de primera clase.

Roadmap
├── Identity
├── Vision
├── Strategy
├── Objectives
├── Scope
├── Current State
├── Target State
├── Gaps
├── Phases
├── Stages
├── Initiatives
├── Programs
├── Projects
├── Epics
├── Features
├── Capabilities
├── Requirements
├── Dependencies
├── Milestones
├── Deliverables
├── Releases
├── Outcomes
├── Risks
├── Resources
├── Budget
├── Costs
├── Teams
├── Owners
├── Metrics
├── KPIs
├── Policies
├── Contracts
├── AI
├── Agents
├── Governance
└── Lifecycle

Esto significa que el Roadmap no debe tratarse simplemente como:

documento

o:

lista de tareas

sino como un modelo estructurado de evolución.

91.4 — Principio fundamental

El Roadmap debe responder cinco preguntas fundamentales:

1. ¿Dónde estamos?
2. ¿Dónde queremos llegar?
3. ¿Qué necesitamos cambiar?
4. ¿En qué orden debemos hacerlo?
5. ¿Cómo sabemos que llegamos?

Por lo tanto:

CURRENT STATE
      ↓
TARGET STATE
      ↓
GAP
      ↓
ROADMAP
      ↓
EXECUTION
      ↓
VALIDATION
      ↓
OUTCOME
91.5 — Roadmap vs Plan

EVOXA debe diferenciar claramente ambos conceptos.

Roadmap

Define:

dirección
prioridades
evolución
dependencias
resultados
Plan

Define:

acciones
fechas
responsables
recursos
ejecución

Por lo tanto:

ROADMAP
   ↓
WHAT + WHY + ORDER
   ↓
PLAN
   ↓
HOW + WHEN + WHO

Un Roadmap puede producir múltiples planes.

91.6 — Roadmap vs Project

Tampoco deben confundirse.

ROADMAP
   │
   ├── Initiative
   │      ├── Project A
   │      └── Project B
   │
   ├── Initiative
   │      └── Project C
   │
   └── Initiative
          └── Project D

El Project ejecuta un objetivo delimitado.

El Roadmap coordina la evolución estratégica que puede requerir muchos proyectos.

91.7 — Roadmap vs Architecture

La arquitectura define el sistema.

El Roadmap define su evolución.

ARCHITECTURE
     ↓
TARGET ARCHITECTURE
     ↓
GAP
     ↓
ROADMAP
     ↓
ARCHITECTURAL EVOLUTION

Por lo tanto:

Architecture defines structure. Roadmap defines the path to evolve that structure.

91.8 — Roadmap Scope

El Roadmap puede existir en diferentes niveles.

Enterprise Roadmap
       ↓
Organization Roadmap
       ↓
Portfolio Roadmap
       ↓
Product Roadmap
       ↓
Application Roadmap
       ↓
Platform Roadmap
       ↓
Engineering Roadmap
       ↓
Team Roadmap
       ↓
Capability Roadmap

Cada nivel puede tener su propio contexto, pero todos deben poder relacionarse.

91.9 — Roadmap Hierarchy

La jerarquía principal será:

VISION
   ↓
STRATEGY
   ↓
OBJECTIVE
   ↓
ROADMAP
   ↓
PHASE
   ↓
INITIATIVE
   ↓
PROGRAM
   ↓
PROJECT
   ↓
EPIC
   ↓
FEATURE
   ↓
TASK

Pero también existe una dimensión basada en capacidades:

OBJECTIVE
   ↓
CAPABILITY
   ↓
SERVICE
   ↓
COMPONENT
   ↓
IMPLEMENTATION

Ambas estructuras deben poder conectarse.

91.10 — Roadmap Anatomy

La anatomía fundamental:

Roadmap
├── Identity
├── Purpose
├── Vision
├── Strategy
├── Objectives
├── Scope
├── Context
├── Current State
├── Target State
├── Gap Analysis
├── Phases
├── Initiatives
├── Programs
├── Projects
├── Milestones
├── Deliverables
├── Capabilities
├── Requirements
├── Dependencies
├── Constraints
├── Priorities
├── Risks
├── Resources
├── Teams
├── Budget
├── Cost
├── Metrics
├── KPIs
├── Outcomes
├── Policies
├── Contracts
├── AI Metadata
├── Agent Metadata
├── Version
├── Governance
└── Lifecycle
91.11 — Roadmap Identity

Cada Roadmap debe poseer identidad única.

Roadmap Identity
├── ID
├── Name
├── Key
├── Version
├── Organization
├── Tenant
├── Owner
├── Created At
├── Updated At
└── Status

La identidad debe permitir diferenciar:

Roadmap
vs
Roadmap Version
vs
Roadmap Snapshot
vs
Roadmap Scenario
91.12 — Roadmap Purpose

Cada Roadmap debe declarar explícitamente su propósito.

Ejemplo:

Purpose:
Define the evolutionary path required to transform
the current EVOXA platform into the target autonomous
AI-native platform.

El propósito permite evaluar si una iniciativa realmente pertenece al Roadmap.

91.13 — Roadmap Vision

La Vision define el estado futuro deseado.

VISION
   ↓
TARGET STATE

Debe ser estable en comparación con las iniciativas.

Una iniciativa puede cambiar.

La visión no debería cambiar continuamente.

91.14 — Strategy

Strategy traduce la visión en principios de acción.

VISION
   ↓
STRATEGY
   ↓
OBJECTIVES
   ↓
ROADMAP

Ejemplo:

Vision:
Self-evolving EVOXA

Strategy:
Build foundational capabilities first,
then intelligence, AI, agents and autonomy.
91.15 — Objectives

Los Objectives representan resultados estratégicos.

Objective
├── Identity
├── Description
├── Value
├── Scope
├── Success Criteria
├── Metrics
├── Dependencies
├── Owner
└── Lifecycle

Un Objective debe poder responder:

¿Qué cambio queremos conseguir?

91.16 — Current State

El Foundation debe definir una representación formal del estado actual.

CURRENT STATE
├── Platform
├── Applications
├── Users
├── Security
├── Operations
├── Engineering
├── AI
├── Agents
├── Infrastructure
├── Data
├── Capabilities
├── Architecture
└── Technical Debt
91.17 — Target State

El Target State representa el resultado esperado.

TARGET STATE
├── Platform
├── Applications
├── Users
├── Security
├── Operations
├── Engineering
├── AI
├── Agents
├── Intelligence
├── Infrastructure
├── Data
├── Capabilities
└── Autonomy
91.18 — Gap

El Gap representa la diferencia entre Current State y Target State.

CURRENT STATE
      ↓
     GAP
      ↓
TARGET STATE

Tipos:

Capability Gap
Architecture Gap
Technology Gap
Security Gap
Operational Gap
Engineering Gap
Data Gap
AI Gap
Agent Gap
Resource Gap
Organizational Gap
Compliance Gap
91.19 — Roadmap Elements

Los elementos principales:

Phase
Stage
Initiative
Program
Project
Epic
Feature
Milestone
Deliverable
Capability
Requirement
Release
Outcome

Cada elemento tiene una función distinta.

91.20 — Phase

Una Phase representa una gran etapa evolutiva.

Ejemplo:

PHASE 1
Foundation

PHASE 2
Platform

PHASE 3
Applications

PHASE 4
Intelligence

PHASE 5
Autonomy

Una Phase puede contener múltiples Initiatives.

91.21 — Stage

Un Stage representa un estado más específico dentro de una Phase.

Phase
   ↓
Stage
   ↓
Initiatives

Esto permite mayor granularidad sin convertir el Roadmap en una simple lista de tareas.

91.22 — Initiative

Una Initiative representa una transformación estratégica.

Initiative
├── Identity
├── Purpose
├── Objective
├── Scope
├── Value
├── Dependencies
├── Risks
├── Resources
├── Milestones
├── Deliverables
├── Capabilities
├── Projects
└── Lifecycle
91.23 — Milestone

Un Milestone representa un punto verificable.

Milestone
├── Identity
├── Objective
├── Acceptance Criteria
├── Dependencies
├── Deliverables
├── Owner
├── Status
├── Risk
├── Metrics
└── Evidence

La finalización debe poder demostrarse.

91.24 — Deliverable

Un Deliverable es un resultado concreto.

Puede ser:

Architecture
Component
Service
Capability
Application
API
Documentation
Infrastructure
AI Model
Agent
Release
91.25 — Outcome

El Outcome representa el resultado real producido.

Esta distinción es fundamental:

DELIVERABLE
= Lo que construimos

OUTCOME
= Lo que conseguimos

Ejemplo:

Deliverable:
Authorization Engine

Outcome:
Reduced unauthorized access risk
91.26 — Roadmap Dependencies

Las dependencias son elementos fundamentales.

Initiative A
      ↓
Capability B
      ↓
Platform Service C
      ↓
Application D

Tipos:

Technical
Architectural
Functional
Security
Operational
Organizational
Financial
Data
AI
Agent
Regulatory
91.27 — Constraints

El Roadmap debe representar restricciones.

Constraints
├── Budget
├── Resources
├── Technology
├── Security
├── Compliance
├── Infrastructure
├── Time
├── Availability
├── Dependencies
├── Skills
└── Capacity
91.28 — Priorities

La prioridad debe ser un concepto explícito.

Priority
├── Strategic Value
├── Business Value
├── User Value
├── Risk Reduction
├── Dependency Criticality
├── Cost
├── Complexity
├── Urgency
└── Confidence

No debe depender únicamente de una prioridad manual.

91.29 — Roadmap Resources

Cada elemento puede requerir:

People
Teams
Infrastructure
Cloud
Compute
Storage
Data
AI
GPU
Budget
External Services

El Foundation debe permitir asociar estos recursos al Roadmap.

91.30 — Roadmap Ownership

Cada elemento importante debe tener ownership.

Roadmap Owner
      ↓
Phase Owner
      ↓
Initiative Owner
      ↓
Milestone Owner
      ↓
Deliverable Owner

Esto evita Roadmaps sin responsabilidad clara.

91.31 — Roadmap Governance

Governance define:

Who can create
Who can modify
Who can approve
Who can reprioritize
Who can defer
Who can cancel
Who can complete
Who can archive

También define:

Approval Policies
Change Policies
Risk Policies
Budget Policies
Security Policies
AI Policies
Agent Policies
91.32 — Roadmap Policies

Ejemplos:

Priority Policy
Dependency Policy
Risk Policy
Budget Policy
Change Policy
Release Policy
Security Policy
AI Policy
Agent Policy
Governance Policy

Las Policies no deben confundirse con Roadmap status.

91.33 — Roadmap State

El estado operacional del Roadmap puede ser:

PLANNED
IN_PROGRESS
ON_TRACK
AT_RISK
BLOCKED
COMPLETED
CANCELLED
DEFERRED

Esto es diferente del Lifecycle.

91.34 — Roadmap Lifecycle

El Lifecycle define evolución:

PROPOSED
   ↓
DESIGNED
   ↓
VALIDATED
   ↓
APPROVED
   ↓
ACTIVE
   ↓
EVOLVING
   ↓
COMPLETED
   ↓
ARCHIVED
91.35 — Roadmap Versioning

Cada modificación importante puede generar una nueva versión.

Roadmap v1.0
     ↓
Roadmap v1.1
     ↓
Roadmap v1.2
     ↓
Roadmap v2.0

Debe ser posible conocer:

What changed?
Why?
Who changed it?
When?
What was the impact?
What was approved?
91.36 — Roadmap Snapshot

EVOXA debe poder congelar un estado:

ROADMAP
   ↓
SNAPSHOT

Ejemplo:

Roadmap Snapshot
2026-Q3

Esto permite comparar:

Planned
vs
Actual
91.37 — Roadmap Scenarios

El Foundation debe soportar escenarios.

BASELINE
OPTIMISTIC
CONSERVATIVE
ACCELERATED
COST_OPTIMIZED
RISK_MINIMIZED
AI_ACCELERATED

Cada escenario puede tener:

different priorities
different dependencies
different costs
different timelines
different risks
91.38 — Roadmap Traceability

Debe existir trazabilidad:

Vision
 ↓
Strategy
 ↓
Objective
 ↓
Initiative
 ↓
Requirement
 ↓
Capability
 ↓
Component
 ↓
Release
 ↓
Deployment
 ↓
Outcome

Esta trazabilidad será crítica para Engineering y AI.

91.39 — Roadmap Relationship Graph

El Foundation debe definir un Roadmap Graph.

                    VISION
                      │
                      ▼
                   OBJECTIVE
                      │
                      ▼
                  INITIATIVE
                 /    │     \
                ▼     ▼      ▼
          CAPABILITY PROJECT MILESTONE
              │        │        │
              ▼        ▼        ▼
          COMPONENT  RELEASE  DELIVERABLE
              │        │
              └────┬───┘
                   ▼
                 OUTCOME
91.40 — Roadmap Digital Twin

El Roadmap puede poseer un Digital Twin:

ROADMAP DIGITAL TWIN
├── Current State
├── Target State
├── Dependencies
├── Resources
├── Costs
├── Risks
├── Progress
├── Scenarios
├── Outcomes
└── Predictions

Esto permitirá posteriormente simulaciones.

91.41 — Roadmap Metrics

El Foundation debe definir métricas estándar.

Progress
Velocity
Completion Rate
Milestone Success Rate
Schedule Variance
Budget Variance
Risk Exposure
Dependency Delay
Value Delivered
Outcome Achievement
91.42 — Roadmap KPIs

Ejemplos:

% Roadmap Completed
% Objectives Achieved
% Initiatives On Track
% Milestones Completed
Average Delay
Budget Utilization
Risk Exposure
Dependency Blockers
Outcome Success Rate
91.43 — Roadmap Security

El Roadmap contiene información estratégica, por lo que debe estar protegido.

Roadmap Access
      ↓
Identity
      ↓
Authentication
      ↓
Authorization
      ↓
Policy
      ↓
Scope
      ↓
Audit

Debe integrarse con Volume 07 — Security.

91.44 — Roadmap Audit

Toda modificación relevante debe poder auditarse.

Actor
Action
Entity
Previous State
New State
Reason
Timestamp
Approval
Risk
Impact

Esto conecta Roadmap Foundation con Security y Operations.

91.45 — Roadmap Data Ownership

Cada elemento debe tener un propietario de datos.

Roadmap Owner
Initiative Owner
Project Owner
Capability Owner
Outcome Owner

Debe existir claridad sobre quién mantiene la información actualizada.

91.46 — AI Metadata

Cada elemento del Roadmap puede contener metadata para AI:

AI Metadata
├── Semantic Description
├── Intent
├── Context
├── Dependencies
├── Constraints
├── Risk
├── Expected Value
├── Cost
├── Confidence
├── Predictability
└── Recommendations

Esto hará que el Roadmap sea AI-readable.

91.47 — Agent Metadata

Para Agents:

Agent Metadata
├── Allowed Actions
├── Capabilities
├── Tools
├── Scope
├── Permissions
├── Policies
├── Risk Limits
├── Budget
├── Approval
└── Audit

Un Agent nunca debe poder modificar libremente el Roadmap.

91.48 — AI Governance

AI puede:

Analyze
Recommend
Predict
Simulate
Prioritize
Detect Risk
Detect Delay
Suggest Replanning

Pero:

RECOMMENDATION
      ≠
AUTHORIZATION

La autoridad debe permanecer gobernada.

91.49 — Agent Governance

Un Agent podría recibir una delegación:

Agent
 ↓
Roadmap Capability
 ↓
Contract
 ↓
Policy
 ↓
Permission
 ↓
Risk
 ↓
Approval
 ↓
Execution

Por ejemplo:

"Analizar iniciativas retrasadas"

puede estar autorizado.

Pero:

"Cancelar una iniciativa estratégica"

podría requerir aprobación humana.

91.50 — Roadmap Interoperability

Roadmap Foundation debe permitir integración con:

Product Management
Project Management
Engineering
Architecture
Security
Operations
Finance
AI
Agents
Applications
Platform
91.51 — Roadmap Contracts

El Foundation debe establecer que las interacciones críticas puedan formalizarse mediante Contracts.

Roadmap
   ↓
Initiative Contract
   ↓
Milestone Contract
   ↓
Deliverable Contract
   ↓
Acceptance Criteria

Esto conecta directamente con la lógica de Contracts definida en los volúmenes anteriores.

91.52 — Roadmap Boundaries

El Roadmap no debe absorber:

Project Management
Task Management
Source Code Management
Deployment
Operations
Security Enforcement

Puede referenciarlos, pero no reemplazarlos.

Roadmap
   ↓
defines direction
      ↓
Engineering executes
      ↓
Operations operates
      ↓
Security governs
91.53 — Roadmap Foundation Principles

Principios fundamentales:

01. Strategy Aligned
02. Outcome Oriented
03. Dependency Aware
04. Value Driven
05. Risk Aware
06. Governed
07. Traceable
08. Versioned
09. Observable
10. Measurable
11. Secure
12. Auditable
13. AI Readable
14. Agent Ready
15. Evolvable
91.54 — Foundation Maturity

Roadmap Foundation puede evolucionar:

LEVEL 1
STATIC
    ↓
LEVEL 2
DOCUMENTED
    ↓
LEVEL 3
STRUCTURED
    ↓
LEVEL 4
GOVERNED
    ↓
LEVEL 5
TRACEABLE
    ↓
LEVEL 6
MEASURABLE
    ↓
LEVEL 7
INTELLIGENT
    ↓
LEVEL 8
PREDICTIVE
    ↓
LEVEL 9
AUTONOMOUS
    ↓
LEVEL 10
ADAPTIVE
    ↓
LEVEL 11
SELF-EVOLVING
91.55 — Roadmap Foundation Integration

La posición del Foundation dentro de EVOXA:

                 EVOXA
                   │
             ┌─────┴─────┐
             ▼           ▼
        ARCHITECTURE   STRATEGY
             │           │
             └─────┬─────┘
                   ▼
              ROADMAP
                   │
           ┌───────┼───────┐
           ▼       ▼       ▼
        CURRENT  TARGET    GAP
         STATE    STATE
           │       │       │
           └───────┼───────┘
                   ▼
                PHASES
                   ▼
              INITIATIVES
                   ▼
              EXECUTION
91.56 — Relación con los otros volúmenes
FOUNDATION
   │
   ├── Platform Foundation
   ├── Application Foundation
   ├── User Foundation
   ├── Security Foundation
   ├── Operations Foundation
   ├── Engineering Foundation
   │
   └── Roadmap Foundation
             │
             ▼
       EVOLUTION MODEL

Cada Foundation define su propia realidad.

Roadmap Foundation define cómo esas realidades evolucionan.

91.57 — Roadmap como capa transversal
                       ROADMAP
                          │
        ┌─────────────────┼─────────────────┐
        ▼                 ▼                 ▼
     PLATFORM       APPLICATIONS        USERS
        │                 │                 │
        └─────────────────┼─────────────────┘
                          ▼
                       SECURITY
                          │
                       OPERATIONS
                          │
                      ENGINEERING
                          │
                          AI
                          │
                       AGENTS

Por esto Roadmap no pertenece exclusivamente a Engineering o Product.

Es una capa transversal de evolución.

91.58 — Roadmap Evolution Principle

El principio fundamental será:

CURRENT STATE
      ↓
UNDERSTAND
      ↓
TARGET STATE
      ↓
IDENTIFY GAP
      ↓
DEFINE ROADMAP
      ↓
EXECUTE
      ↓
MEASURE
      ↓
LEARN
      ↓
UPDATE CURRENT STATE
      ↓
DEFINE NEW TARGET
      ↓
EVOLVE ROADMAP

Esto convierte al Roadmap en un sistema continuo.

91.59 — Roadmap Foundation Final Model
ROADMAP FOUNDATION
│
├── Identity
├── Vision
├── Strategy
├── Objectives
├── Scope
│
├── Current State
├── Target State
├── Gap Analysis
│
├── Phases
├── Stages
├── Initiatives
├── Programs
├── Projects
├── Epics
├── Features
├── Capabilities
├── Requirements
├── Milestones
├── Deliverables
├── Releases
├── Outcomes
│
├── Dependencies
├── Constraints
├── Priorities
├── Risks
├── Resources
├── Budget
├── Costs
│
├── Metrics
├── KPIs
├── Ownership
├── Governance
├── Policies
├── Contracts
├── Security
├── Audit
│
├── AI Metadata
├── Agent Metadata
├── Digital Twin
├── Versioning
└── Lifecycle
91.60 — Responsabilidad final

La responsabilidad de 91 — Roadmap Foundation queda definida como:

Defines Roadmap.

El Foundation establece:

WHAT IS A ROADMAP
        ↓
WHAT DOES IT CONTAIN
        ↓
WHAT IS ITS PURPOSE
        ↓
WHAT ARE ITS BOUNDARIES
        ↓
HOW DOES IT RELATE TO STRATEGY
        ↓
HOW DOES IT REPRESENT EVOLUTION
        ↓
HOW IS IT GOVERNED
        ↓
HOW IS IT MEASURED
        ↓
HOW IS IT CONNECTED TO AI / AGENTS

Y establece la base para los siguientes niveles:

91 — ROADMAP FOUNDATION
          ↓
92 — ROADMAP CORE
          ↓
93 — ROADMAP RUNTIME
          ↓
94 — ROADMAP SERVICES
          ↓
95 — ROADMAP MODULES
          ↓
96 — ROADMAP DOMAINS
          ↓
97 — ROADMAP COMPONENTS
          ↓
98 — ROADMAP CAPABILITIES
          ↓
99 — ROADMAP CONTRACTS
          ↓
100 — ROADMAP LIFECYCLE

91 — Roadmap Foundation queda así como la capa que define el lenguaje, estructura, límites, principios, gobierno e identidad del sistema de evolución de EVOXA.
