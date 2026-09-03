54 — User Services
54.1 — Introducción

En los capítulos anteriores establecimos:

51 — User Foundation
        ↓
52 — User Core
        ↓
53 — User Runtime

Con esto ya sabemos:

Foundation define quién es el usuario.
Core coordina su estado, contexto, intención y comportamiento.
Runtime ejecuta sus interacciones.

Ahora necesitamos una capa que proporcione funcionalidades reutilizables alrededor del usuario.

Esta es la responsabilidad de:

User Services

User Services constituye la capa de servicios reutilizables que encapsula las operaciones relacionadas con:

identidad;
perfiles;
sesiones;
preferencias;
privacidad;
consentimiento;
comunicación;
personalización;
búsqueda;
recomendaciones;
actividad;
objetivos;
AI;
Agents;
dispositivos;
relaciones;
seguridad.

El principio fundamental es:

User Services proporciona capacidades reutilizables para trabajar con el usuario sin obligar a cada aplicación o módulo a implementar nuevamente la misma lógica.

54.2 — Definición

User Services es la capa de servicios reutilizables y gobernados que implementa operaciones relacionadas con identidad, perfil, contexto, preferencias, privacidad, consentimiento, comunicación, personalización, actividad, objetivos, dispositivos, relaciones, AI y Agents, utilizando User Foundation, User Core y User Runtime como base.

La arquitectura queda:

USER FOUNDATION
      │
      ▼
USER CORE
      │
      ▼
USER RUNTIME
      │
      ▼
USER SERVICES
      │
      ├── Identity
      ├── Profile
      ├── Session
      ├── Preference
      ├── Privacy
      ├── Consent
      ├── Communication
      ├── Personalization
      ├── Recommendation
      ├── Activity
      ├── Goal
      ├── Device
      ├── Relationship
      ├── AI
      └── Agent
54.3 — User Services vs Platform Services

La distinción es crítica.

Platform Services

Proporcionan capacidades generales para todo EVOXA.

Ejemplos:

Identity Infrastructure
Storage
Messaging
Events
Search
Observability
Secrets
AI Infrastructure
User Services

Proporcionan comportamiento específico relacionado con usuarios.

Ejemplos:

User Profile
User Preferences
User Consent
User Personalization
User Recommendations
User Goals
User Activity
User Relationships

Por lo tanto:

Platform Services proporcionan infraestructura reutilizable; User Services proporcionan comportamiento reutilizable centrado en el usuario.

54.4 — User Services vs User Core

También debemos separar:

User Core

Coordina.

User Services

Implementan servicios reutilizables.

User Core
    │
    ├── calls
    ▼
User Services
    │
    ├── Profile Service
    ├── Preference Service
    ├── Recommendation Service
    └── Notification Service

User Core decide qué debe ocurrir.

User Services proporcionan la funcionalidad necesaria para hacerlo.

54.5 — Service Anatomy

Todo User Service debe tener una estructura consistente.

User Service
├── Identity
├── Purpose
├── Interface
├── Operations
├── Inputs
├── Outputs
├── Dependencies
├── Policies
├── Permissions
├── Data
├── Events
├── Errors
├── Security
├── Observability
├── Performance
├── Cost
├── Version
└── Lifecycle

Esto permite que todos los servicios sean:

descubribles;
observables;
gobernables;
versionables;
reutilizables.
54.6 — User Service Registry

EVOXA debe mantener un:

User Service Registry

El registro puede contener:

Service ID
Service Name
Service Type
Purpose
Owner
Domain
Operations
Dependencies
Contracts
Permissions
Policies
Version
Health
SLO
Cost
Security
Lifecycle
AI Metadata
Agent Metadata

Esto permite descubrir servicios dinámicamente.

54.7 — Categorías de User Services

Una clasificación inicial:

User Services
│
├── Identity Services
├── Profile Services
├── Session Services
├── Preference Services
├── Privacy Services
├── Consent Services
├── Security Services
├── Device Services
├── Communication Services
├── Personalization Services
├── Recommendation Services
├── Activity Services
├── Goal Services
├── Relationship Services
├── Search Services
├── AI Services
├── Agent Services
└── Experience Services
54.8 — User Identity Service

El Identity Service administra las operaciones relacionadas con identidad.

Puede proporcionar:

Create Identity
Resolve Identity
Verify Identity
Link Identity
Unlink Identity
Update Identity
Disable Identity
Restore Identity

No necesariamente implementa directamente toda la autenticación.

Puede coordinarse con Platform Identity Services.

User Identity Service
        ↓
Platform Identity Service
        ↓
Identity Infrastructure
54.9 — User Profile Service

Administra el perfil.

Operaciones:

Get Profile
Create Profile
Update Profile
Delete Profile
Merge Profile
Export Profile

Puede gestionar:

nombre;
avatar;
idioma;
timezone;
información de contacto;
atributos de experiencia.
54.10 — Profile Separation

El Profile Service no debe convertirse en un contenedor universal.

Debe mantenerse la separación:

Identity
   ≠
Profile
   ≠
Preferences
   ≠
Activity
   ≠
Goals

Esto permite evolucionar cada área independientemente.

54.11 — User Context Service

El User Context Service ayuda a construir y resolver el contexto del usuario.

User
 ↓
Context Service
 ↓
Identity
Profile
Tenant
Organization
Session
Device
Preferences
Permissions
 ↓
Resolved Context

Puede ser utilizado por:

Applications;
AI;
Agents;
workflows;
recommendations.
54.12 — User Preference Service

Gestiona las preferencias.

Ejemplos:

Language
Timezone
Theme
Notifications
Communication
AI Behavior
Experience
Accessibility

Operaciones:

Get Preferences
Update Preference
Reset Preference
Resolve Effective Preference
54.13 — Effective Preferences

Una preferencia puede provenir de múltiples niveles.

Platform Default
       ↓
Organization
       ↓
Tenant
       ↓
Application
       ↓
User
       ↓
Effective Preference

Esto requiere resolución de prioridad.

Por ejemplo:

Default = English
Tenant = Spanish
User = Spanish

Effective = Spanish
54.14 — User Privacy Service

El Privacy Service gestiona las preferencias y políticas relacionadas con privacidad.

Puede administrar:

visibility;
data sharing;
processing preferences;
AI data usage;
personalization;
external integrations.

Arquitectura:

User
 ↓
Privacy Service
 ↓
Privacy Policies
 ↓
Data Access
54.15 — User Consent Service

El Consent Service administra:

Consent
├── Purpose
├── Scope
├── Data
├── Recipient
├── Version
├── Status
├── Granted At
└── Revoked At

Operaciones:

Request Consent
Grant Consent
Reject Consent
Revoke Consent
Check Consent
54.16 — Consent Resolution

Antes de una operación sensible:

Request
 ↓
Consent Service
 ↓
Consent Valid?
 ├── YES → Continue
 └── NO  → Block / Request Consent

Esto será especialmente importante para:

AI;
personalization;
data sharing;
Agents.
54.17 — User Security Service

Este servicio puede encapsular operaciones relacionadas con:

account security;
credential state;
MFA;
trusted devices;
sessions;
security events;
account recovery.

Ejemplo:

Security Event
 ↓
User Security Service
 ↓
Risk / Policy
 ↓
Action
54.18 — User Device Service

Gestiona dispositivos asociados al usuario.

Operaciones:

Register Device
Verify Device
List Devices
Rename Device
Revoke Device
Trust Device
Block Device

Ejemplo:

User
 ↓
Device Service
 ↓
Mobile Device
 ↓
Trust State
54.19 — User Session Service

Administra sesiones desde la perspectiva del usuario.

Puede proporcionar:

Create Session
Resolve Session
Refresh Session
Revoke Session
List Sessions
Terminate Session

El runtime ejecuta la sesión.

El Session Service proporciona la lógica reutilizable para administrarla.

54.20 — Communication Services

La comunicación con usuarios puede dividirse:

Communication Services
│
├── Notification
├── Email
├── Push
├── SMS
├── In-App
├── Messaging
└── Voice
54.21 — Notification Service

Administra notificaciones.

Event
 ↓
Notification Service
 ↓
User Preferences
 ↓
Channel Selection
 ↓
Delivery

Puede determinar:

canal;
prioridad;
idioma;
contenido;
horario;
preferencias.
54.22 — Notification Preferences

La decisión puede combinar:

User Preferences
+
Application Rules
+
Notification Policy
+
Channel Availability

para producir:

Delivery Decision
54.23 — Messaging Service

Proporciona comunicación persistente.

Puede soportar:

conversations;
threads;
messages;
attachments;
participants;
read state;
delivery state.

Arquitectura:

User
 ↕
Messaging Service
 ↕
User / AI / Agent / Application
54.24 — User Personalization Service

Uno de los servicios más importantes.

Su responsabilidad es adaptar la experiencia al usuario.

Puede utilizar:

Profile
Preferences
Context
History
Goals
Behavior
Consent
AI

para producir:

Personalization Result
54.25 — Personalization Engine

Puede existir un componente especializado:

Personalization Engine

Arquitectura:

User Context
     ↓
Personalization Engine
     ↓
Rules
+
Preferences
+
Behavior
+
AI
     ↓
Experience Decision
54.26 — Personalization Levels

La personalización puede evolucionar:

Level 1

Preferencias básicas.

Level 2

Segmentación.

Level 3

Contextual.

Level 4

Behavioral.

Level 5

Predictive.

Level 6

AI-Personalized.

Level 7

Adaptive.

54.27 — Recommendation Service

El Recommendation Service proporciona recomendaciones.

Puede utilizar:

rules;
history;
preferences;
context;
goals;
AI models.

Ejemplo:

User
 ↓
Context
 ↓
Recommendation Service
 ↓
Recommendation

Las recomendaciones no deben convertirse automáticamente en acciones.

54.28 — Recommendation Explainability

Cuando sea relevante, la recomendación puede incluir:

Recommendation
├── Result
├── Reason
├── Confidence
├── Source
└── Timestamp

Esto permite explicar por qué algo fue recomendado.

54.29 — User Activity Service

Registra actividades relevantes.

Ejemplo:

Activity
├── User
├── Type
├── Source
├── Context
├── Timestamp
├── Resource
└── Result

Puede utilizarse para:

historial;
analytics;
personalization;
AI.
54.30 — User Goal Service

Administra objetivos.

Operaciones:

Create Goal
Update Goal
Pause Goal
Resume Goal
Complete Goal
Abandon Goal
Track Progress

Arquitectura:

User
 ↓
Goal Service
 ↓
Goal
 ↓
Progress
54.31 — Goal and AI

AI puede ayudar a convertir objetivos generales en acciones.

User Goal
 ↓
AI Analysis
 ↓
Recommended Actions
 ↓
User Core
 ↓
Capabilities

El Goal Service mantiene el estado del objetivo.

54.32 — User Relationship Service

Gestiona relaciones entre usuarios.

Puede soportar:

Follow
Friend
Manager
Coach
Member
Collaborator
Contact

Las relaciones deben ser explícitas y gobernadas.

54.33 — Relationship Permissions

Una relación no implica automáticamente acceso.

Por ejemplo:

User A
   │
   └── follows → User B

no significa:

User A
   ↓
Access all User B data

La relación y el permiso son conceptos independientes.

54.34 — User Search Service

Permite localizar usuarios o recursos relacionados.

Puede soportar:

identity search;
profile search;
organization search;
relationship search.

Debe respetar:

Privacy
Permissions
Tenant
Visibility
54.35 — User Experience Service

Este servicio puede actuar como una capa de composición.

User Context
       ↓
Experience Service
       │
 ┌─────┼─────┐
 ▼     ▼     ▼
Profile AI   Recommendations
       │
       ▼
Experience

Su función es ensamblar información necesaria para una experiencia concreta.

54.36 — User AI Service

Este servicio encapsula interacciones de AI específicas del usuario.

Puede proporcionar:

Ask AI
Summarize User Context
Generate Recommendation
Analyze User Activity
Explain Recommendation
Personalize Experience

Arquitectura:

User
 ↓
User AI Service
 ↓
Platform AI Services
 ↓
AI Runtime
 ↓
Model
54.37 — User AI Context

Antes de utilizar AI:

User AI Request
      ↓
Context Resolution
      ↓
Permission
      ↓
Privacy
      ↓
Consent
      ↓
AI Context
      ↓
Model

Esto evita enviar contexto innecesario.

54.38 — User AI Memory Service

Puede existir un servicio específico para memoria relacionada con AI.

Debe diferenciar:

Conversation Context
User Memory
Application Memory
Agent Memory

Y controlar:

storage;
retrieval;
retention;
privacy;
consent;
deletion.
54.39 — User Agent Service

Este servicio gestiona la relación entre usuarios y Agents.

Puede proporcionar:

List Agents
Assign Agent
Delegate Task
Approve Action
Reject Action
Revoke Delegation
View Agent Activity
54.40 — Delegation Service

La delegación puede representarse:

User
 ↓
Delegation Service
 ↓
Agent
 ↓
Capabilities

Debe controlar:

scope;
permissions;
duration;
budget;
risk;
approval.
54.41 — Agent Approval Service

Cuando un Agent necesita autorización:

Agent
 ↓
Approval Request
 ↓
Approval Service
 ↓
User
 ↓
Approve / Reject
 ↓
Agent

La decisión debe quedar registrada.

54.42 — User Subscription Service

Puede existir un servicio para administrar las relaciones del usuario con productos o servicios.

User
 ↓
Subscription
 ↓
Application / Service

Puede manejar:

plans;
status;
renewal;
cancellation;
entitlements.

Este servicio podrá integrarse posteriormente con Billing.

54.43 — User Entitlement Service

Los entitlements determinan qué obtiene el usuario.

Subscription
 ↓
Entitlements
 ↓
Capabilities
 ↓
Application

Por ejemplo:

Basic User
 → Capability A

Premium User
 → Capability A
 → Capability B
 → Capability C
54.44 — User Data Service

Puede proporcionar una abstracción segura para acceder a información del usuario.

Application
 ↓
User Data Service
 ↓
Policies
 ↓
User Data

No debería permitir acceso indiscriminado.

54.45 — User Export Service

El usuario puede solicitar sus datos.

User
 ↓
Export Request
 ↓
Authorization
 ↓
Privacy
 ↓
Data Collection
 ↓
Export

Puede generar:

profile;
preferences;
activity;
relationships;
selected data.
54.46 — User Deletion Service

El borrado debe ser gobernado.

Deletion Request
 ↓
Verification
 ↓
Impact Analysis
 ↓
Retention Rules
 ↓
Data Deletion / Anonymization
 ↓
Audit

No todos los datos necesariamente pueden eliminarse inmediatamente si existen obligaciones de retención.

54.47 — User Data Portability

El usuario puede necesitar trasladar información.

User
 ↓
Export
 ↓
Portable Format
 ↓
External System

Esto será especialmente importante para ecosistemas abiertos.

54.48 — User Service Composition

Los servicios pueden combinarse.

Ejemplo:

User Registration
       │
       ├── Identity Service
       ├── Profile Service
       ├── Preference Service
       ├── Consent Service
       └── Notification Service

User Core puede coordinar el proceso.

54.49 — Service Orchestration

Un workflow complejo puede ser:

Create User
     ↓
Create Profile
     ↓
Set Preferences
     ↓
Request Consent
     ↓
Create Session
     ↓
Send Welcome Notification

Si una operación falla:

Failure
 ↓
Retry
 ↓
Compensation
54.50 — Service Choreography

También puede utilizar eventos.

UserCreated
    ↓
Profile Service
    ↓
Preferences Service
    ↓
Notification Service
    ↓
Analytics Service

Esto reduce el acoplamiento.

54.51 — Service-to-Service Security

Todos los User Services deben autenticarse y autorizarse entre sí.

User Core
 ↓
Service Identity
 ↓
User Service
 ↓
Authorization

No se debe confiar únicamente en que una llamada venga de otro servicio interno.

54.52 — Service Permissions

Cada operación puede requerir permissions.

Ejemplo:

user.profile.read
user.profile.update
user.preferences.update
user.consent.manage
user.data.export
user.agent.delegate
user.agent.approve
54.53 — Service Policies

Las policies pueden agregar condiciones.

Permission
+
Policy
+
Context
=
Allowed Operation
54.54 — Service Contracts

Cada User Service debe exponer contratos.

User Service
      ↓
Service Contract
      ↓
Consumer

Los contratos deben definir:

operations;
inputs;
outputs;
errors;
permissions;
events;
SLO;
version.
54.55 — Service Versioning

Los servicios deben evolucionar sin romper consumidores.

User Profile Service v1
          ↓
User Profile Service v2

Debe existir:

compatibility;
migration;
deprecation;
version routing.
54.56 — Service Reliability

Cada User Service debe tener objetivos de confiabilidad.

Ejemplo:

Availability
Latency
Error Rate
Throughput
Recovery Time
54.57 — Service Health

Cada servicio debe exponer:

Health
Readiness
Dependencies
Capacity
Errors
Latency

Ejemplo:

Profile Service
   ↓
Healthy

Recommendation Service
   ↓
Degraded

User Core puede reaccionar adecuadamente.

54.58 — Service Observability

Debe existir:

Logs
Metrics
Traces
Events
Audit

y una relación:

User Interaction
 ↓
User Core
 ↓
User Service
 ↓
Capability
 ↓
Component
54.59 — Service Dependency Graph

EVOXA debe conocer dependencias:

Personalization Service
       │
       ├── Profile Service
       ├── Preference Service
       ├── Activity Service
       └── AI Service

Esto permite:

impact analysis;
failure analysis;
optimization;
lifecycle management.
54.60 — User Services and AI

Los servicios pueden utilizar AI internamente.

Por ejemplo:

Recommendation Service
        ↓
AI Recommendation Engine
        ↓
Model

Pero el contrato del servicio permanece estable.

Esto permite cambiar el modelo sin cambiar necesariamente la Application.

54.61 — AI Service Selection

EVOXA puede seleccionar modelos según:

Quality
Latency
Cost
Availability
Privacy
Region
Risk

Esto conecta User Services con Platform AI Services.

54.62 — User Services and Agents

Los Agents pueden consumir User Services mediante contratos.

Agent
 ↓
Tool Contract
 ↓
User Service
 ↓
Capability

Esto evita que el Agent acceda directamente a las estructuras internas.

54.63 — Agent-Safe Services

Los servicios utilizados por Agents deben poder declarar:

Agent Allowed?
Risk Level
Required Approval
Allowed Scope
Maximum Cost
Rate Limit
Data Access

Esto crea servicios Agent-Ready.

54.64 — User Service Marketplace

En una plataforma madura, los User Services pueden descubrirse.

Service Registry
      ↓
Search
      ↓
Service
      ↓
Contract
      ↓
Capability

Las Applications pueden reutilizar servicios existentes en lugar de recrearlos.

54.65 — Service Certification

EVOXA puede certificar servicios según:

Security
Reliability
Performance
Compatibility
Documentation
Observability
AI Safety
Agent Safety

Esto permite construir un catálogo de servicios confiables.

54.66 — Service Quality Score

Cada servicio puede tener un score:

Reliability
Security
Performance
Cost
Adoption
Quality
Risk

La plataforma puede utilizarlo para seleccionar servicios automáticamente.

54.67 — Intelligent Service Routing

Una Application podría requerir:

Recommendation Service

y EVOXA seleccionar:

Provider A
Quality: 95
Latency: 80ms
Cost: Low

en lugar de:

Provider B
Quality: 80
Latency: 200ms
Cost: High

Esto conecta Service Registry con AI y Runtime.

54.68 — Service Cost

Cada User Service debe poder medir su costo.

Service
 ↓
Usage
 ↓
Resources
 ↓
Cost

Puede calcularse:

cost/request;
cost/user;
cost/tenant;
cost/application;
cost/AI interaction.
54.69 — User Service FinOps

EVOXA puede detectar:

High Usage
 ↓
High Cost
 ↓
Optimization Opportunity

Por ejemplo:

Recommendation Service
 ↓
AI Model A
 ↓
High Cost

Alternative:
Model B
 ↓
Lower Cost
 ↓
Similar Quality
54.70 — Service Lifecycle

Cada User Service debe tener lifecycle:

PROPOSED
   ↓
DESIGNED
   ↓
REGISTERED
   ↓
VALIDATED
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

Esto conecta directamente con:

60 — User Lifecycle.

54.71 — Service Self-Healing

Un servicio puede detectar problemas:

Health Degraded
      ↓
Detection
      ↓
Recovery Strategy
      ↓
Restart / Failover / Scale
      ↓
Validation
54.72 — Service Intelligence

En etapas avanzadas:

Metrics
+
Logs
+
Traces
+
Usage
+
Cost
+
User Feedback
        ↓
AI
        ↓
Service Intelligence

Puede identificar:

degradación;
problemas;
costos;
anomalías;
oportunidades de mejora.
54.73 — Autonomous User Services

Un servicio avanzado puede participar en su propia optimización.

Observe
 ↓
Analyze
 ↓
Predict
 ↓
Recommend
 ↓
Policy
 ↓
Optimize
 ↓
Validate

La autonomía debe estar limitada por governance.

54.74 — User Service Architecture

La arquitectura completa:

                         USER
                           │
                    USER FOUNDATION
                           │
                       USER CORE
                           │
                     USER RUNTIME
                           │
                    USER SERVICES
                           │
       ┌───────────────────┼────────────────────┐
       │                   │                    │
       ▼                   ▼                    ▼
    IDENTITY            PROFILE              CONTEXT
       │                   │                    │
       ▼                   ▼                    ▼
  PREFERENCES           PRIVACY             CONSENT
       │                   │                    │
       └───────────────────┼────────────────────┘
                           ▼
                    COMMUNICATION
                           │
             ┌─────────────┼─────────────┐
             ▼             ▼             ▼
        NOTIFICATION     MESSAGE        EMAIL
                           │
                           ▼
                    PERSONALIZATION
                           │
             ┌─────────────┼─────────────┐
             ▼             ▼             ▼
       RECOMMENDATION     ACTIVITY       GOALS
             │             │             │
             └─────────────┼─────────────┘
                           ▼
                       AI SERVICES
                           │
                           ▼
                      AGENT SERVICES
                           │
                           ▼
                       EXPERIENCE
54.75 — User Services Fabric

A partir de esta arquitectura aparece una abstracción importante:

User Services Fabric

Esta fabric proporciona un conjunto común de servicios reutilizables alrededor del usuario.

                 USER SERVICES FABRIC
                         │
     ┌───────────────────┼───────────────────┐
     │                   │                   │
 Identity             Context          Communication
     │                   │                   │
 Profile            Preferences        Notifications
     │                   │                   │
 Privacy             Consent            Messaging
     │                   │                   │
 Personalization     Recommendations    Goals
     │                   │                   │
 AI Services         Agent Services     Experience
54.76 — Principios de User Services
1. Reusable

Los servicios deben poder utilizarse desde múltiples Applications.

2. Governed

Todos deben estar sujetos a políticas.

3. Contract First

Cada servicio debe tener un contrato explícito.

4. Secure by Default

La seguridad debe formar parte del servicio.

5. Privacy Aware

Los servicios deben respetar privacidad y consentimiento.

6. Observable

Los servicios deben ser observables.

7. Versioned

Los servicios deben poder evolucionar.

8. Composable

Los servicios deben poder combinarse.

9. AI Ready

Los servicios deben poder integrar AI.

10. Agent Ready

Los servicios deben poder ser utilizados por Agents de forma controlada.

11. Resilient

Los servicios deben tolerar fallos.

12. Cost-Aware

El costo debe poder medirse.

13. Discoverable

Los servicios deben poder descubrirse.

14. Interoperable

Deben funcionar mediante contratos comunes.

54.77 — User Services Maturity

La madurez puede evolucionar:

Level 1 — Basic Services

Servicios simples.

Level 2 — Structured Services

Servicios claramente separados.

Level 3 — Governed Services

Políticas, permisos y governance.

Level 4 — Observable Services

Métricas, logs y tracing.

Level 5 — Composable Services

Servicios combinables.

Level 6 — Intelligent Services

AI integrada.

Level 7 — Agent-Enabled Services

Agents pueden consumirlos bajo contratos.

Level 8 — Autonomous Services

Servicios capaces de optimizarse y recuperarse automáticamente.

Level 9 — Adaptive Services

Servicios que evolucionan dinámicamente bajo governance.

54.78 — Responsabilidad dentro de Volume 06

Con este capítulo la separación queda:

User Foundation
        ↓
Defines the User

User Core
        ↓
Coordinates the User

User Runtime
        ↓
Executes User Interaction

User Services
        ↓
Provides User Services

User Modules
        ↓
Organizes User Functionality

User Domains
        ↓
Contextualizes User Behavior

User Components
        ↓
Implements User Functionality

User Capabilities
        ↓
Exposes User Abilities

User Contracts
        ↓
Standardizes User Interaction

User Lifecycle
        ↓
Evolves the User
54.79 — Relación con Applications

La arquitectura de interacción queda:

USER
 │
 ▼
USER FOUNDATION
 │
 ▼
USER CORE
 │
 ▼
USER RUNTIME
 │
 ▼
USER SERVICES
 │
 ▼
APPLICATION
 │
 ├── Application Services
 ├── Modules
 ├── Domains
 ├── Components
 └── Capabilities

Los User Services proporcionan una capa transversal que puede ser consumida por múltiples Applications.

54.80 — Relación con AI y Agents

La arquitectura avanzada queda:

                         USER
                           │
                     USER FOUNDATION
                           │
                       USER CORE
                           │
                     USER RUNTIME
                           │
                    USER SERVICES
                           │
             ┌─────────────┴─────────────┐
             ▼                           ▼
            AI                         AGENTS
             │                           │
             └─────────────┬─────────────┘
                           ▼
                       APPLICATION
                           │
                       EXPERIENCE

Y el principio:

AI y Agents no deberían acceder directamente al User Model sin pasar por servicios, contratos, permisos, policies y contexto.

54.81 — Definición final

User Services son la capa de servicios reutilizables, gobernados y observables de EVOXA que proporciona operaciones relacionadas con identidad, perfil, contexto, sesiones, preferencias, privacidad, consentimiento, seguridad, dispositivos, comunicación, personalización, recomendaciones, actividad, objetivos, relaciones, AI, Agents y experiencia, permitiendo que múltiples Applications utilicen capacidades comunes de usuario de forma segura, versionable, composable, resiliente e inteligente.

54.82 — Visión final

La evolución de User Services:

Basic Services
      ↓
Structured Services
      ↓
Governed Services
      ↓
Observable Services
      ↓
Composable Services
      ↓
Intelligent Services
      ↓
Agent-Enabled Services
      ↓
Autonomous Services
      ↓
Adaptive Services

Y la evolución conceptual:

                   USER
                     │
               FOUNDATION
                     │
                   CORE
                     │
                 RUNTIME
                     │
                 SERVICES
                     │
        ┌────────────┼────────────┐
        ▼            ▼            ▼
     IDENTITY     CONTEXT    COMMUNICATION
        │            │            │
        ▼            ▼            ▼
     PROFILE     PREFERENCES   NOTIFICATIONS
        │            │            │
        └────────────┼────────────┘
                     ▼
              PERSONALIZATION
                     │
            ┌────────┼────────┐
            ▼        ▼        ▼
       RECOMMENDATION GOALS ACTIVITY
            │        │        │
            └────────┼────────┘
                     ▼
                    AI
                     │
                   AGENTS
                     │
                     ▼
                EXPERIENCE
                     │
                     ▼
                 ADAPTATION

Con 54 — User Services, quedan establecidas las cuatro primeras capas del Volume 06 — Users & Experience:

51 — User Foundation
        ↓
52 — User Core
        ↓
53 — User Runtime
        ↓
54 — User Services

El siguiente capítulo será:

55 — User Modules

donde organizaremos estas capacidades y servicios en módulos funcionales completos del usuario, estableciendo sus límites, responsabilidades, dependencias, datos, capacidades, componentes, AI y Agents.
