52 — User Core
52.1 — Introducción

User Core es la capa central de coordinación del comportamiento, estado, intención e interacción del usuario dentro de EVOXA.

En 51 — User Foundation definimos:

quién es el usuario;
su identidad;
su perfil;
su contexto;
sus organizaciones;
sus tenants;
sus permisos;
sus preferencias;
su privacidad;
su consentimiento;
sus relaciones;
su relación con AI y Agents.

Ahora debemos responder una pregunta diferente:

¿Cómo EVOXA comprende, coordina y transforma lo que el usuario hace, quiere y necesita en comportamiento dentro del ecosistema?

Esa es la responsabilidad de User Core.

52.2 — Definición

User Core es la capa central responsable de coordinar el estado, contexto, intención, decisiones, comandos, consultas, eventos, preferencias, interacciones y procesos relacionados con el usuario.

User Core recibe el contexto definido por User Foundation y lo convierte en comportamiento coordinado.

User Foundation
       │
       │ Identity
       │ Profile
       │ Context
       │ Permissions
       │ Preferences
       │ Consent
       ▼
   USER CORE
       │
       ├── State
       ├── Intent
       ├── Decisions
       ├── Commands
       ├── Queries
       ├── Events
       ├── Interaction
       ├── Personalization
       ├── AI
       └── Agents

La diferencia fundamental es:

Foundation define al User. Core coordina al User.

52.3 — User Core como cerebro de coordinación

User Core puede considerarse el centro lógico que conecta:

User
 │
 ▼
Context
 │
 ▼
Intent
 │
 ▼
Decision
 │
 ▼
Action
 │
 ▼
Experience
 │
 ▼
Feedback
 │
 ▼
Learning

No significa que User Core deba contener toda la lógica de la aplicación.

Su responsabilidad es coordinar la interacción del usuario con las diferentes capacidades de EVOXA.

52.4 — User Core y Application Core

Debemos mantener una separación clara.

User Core

Responde:

¿Qué está intentando hacer el usuario?

Application Core

Responde:

¿Cómo debe comportarse la aplicación para realizar esa acción?

Por ejemplo:

User
 ↓
User Core
 ↓
Intent
 ↓
Application Core
 ↓
Capability
 ↓
Execution

Esta separación permite evitar que el usuario quede acoplado directamente a la implementación interna de una Application.

52.5 — User State

User Core administra el estado lógico del usuario.

No debe confundirse con los datos permanentes del perfil.

Podemos distinguir:

Identity State
Profile State
Session State
Interaction State
Preference State
Activity State
Goal State
Workflow State
AI State
Agent State
Experience State

Por ejemplo:

User Profile:
Language = Spanish

Current Session:
Application = Training

Current Intent:
Create Workout

Current Experience:
Workout Builder

AI Context:
Personalized recommendations enabled
52.6 — Persistent State vs Dynamic State

User Core debe distinguir entre:

Persistent State

Información que permanece.

Ejemplos:

preferencias;
configuración;
objetivos;
relaciones;
historial.
Dynamic State

Información temporal.

Ejemplos:

sesión;
actividad actual;
intent;
workflow;
interacción;
contexto temporal.
USER
 │
 ├── Persistent State
 │
 └── Dynamic State

Esto evita convertir todo el contexto del usuario en información permanente.

52.7 — Desired User State vs Actual User State

Siguiendo el modelo establecido en Platform y Application Lifecycle, User Core también puede trabajar con:

Desired User State
        │
        ▼
    User Core
        │
        ▼
Actual User State

Ejemplo:

Desired:
Notifications = Enabled

Actual:
Notifications = Disabled

User Core puede detectar la diferencia y coordinar la acción correspondiente.

52.8 — User Intent

Uno de los conceptos centrales de User Core es:

Intent

El Intent representa aquello que el usuario quiere conseguir.

Puede originarse mediante:

acción explícita;
texto;
voz;
navegación;
selección;
API;
AI;
Agent.

Ejemplo:

User says:
"Quiero crear un entrenamiento"

        ↓

Intent:
CreateWorkout
52.9 — Intent Structure

Un Intent puede contener:

Intent
├── Intent ID
├── User
├── Type
├── Goal
├── Context
├── Inputs
├── Constraints
├── Preferences
├── Permissions
├── Priority
├── Deadline
├── Confidence
├── Expected Outcome
└── Status

Esto permite que EVOXA transforme una interacción humana en una representación estructurada.

52.10 — Intent Sources

Los intents pueden provenir de diferentes canales:

User Action
     │
Voice
     │
Chat
     │
API
     │
AI
     │
Agent
     │
External System
     ▼
   INTENT

Esto permite una arquitectura omnicanal.

52.11 — Intent Resolution

El User Core debe determinar qué significa una interacción.

Por ejemplo:

Input:
"Necesito entrenar mañana"

       ↓

Interpretation

       ↓

Intent:
ScheduleWorkout

       ↓

Context:
User + Calendar + Preferences

       ↓

Decision

La interpretación puede ser realizada por:

reglas;
servicios;
NLP;
AI;
Agents.
52.12 — Intent vs Command

Deben mantenerse separados.

Intent

Lo que el usuario quiere conseguir.

Command

Una instrucción concreta para cambiar el estado.

Ejemplo:

Intent:
Complete Workout

        ↓

Command:
CompleteWorkoutCommand

Esta separación permite que un mismo Intent pueda generar diferentes estrategias de ejecución.

52.13 — User Commands

Los Commands representan acciones solicitadas.

Ejemplos:

UpdateProfile
ChangePreference
CreateGoal
StartWorkout
CompleteWorkout
CancelWorkout
AskAI
DelegateTask
ApproveAgentAction

Un command debería pasar por:

Authentication
      ↓
Authorization
      ↓
Validation
      ↓
Policy
      ↓
Execution
      ↓
State Change
      ↓
Event
52.14 — User Queries

Las Queries representan solicitudes de información.

Ejemplos:

GetProfile
GetPreferences
GetGoals
GetHistory
GetProgress
GetRecommendations
GetNotifications
GetAvailableCapabilities

Una query no debería modificar el estado.

Query
 ↓
Read
 ↓
Response
52.15 — User Events

Los Events representan hechos ocurridos.

Ejemplos:

UserRegistered
ProfileUpdated
PreferenceChanged
GoalCreated
GoalCompleted
WorkoutStarted
WorkoutCompleted
RecommendationViewed
AIInteractionCompleted
AgentActionApproved

La diferencia:

Command → Intent to change
Event   → Fact that changed
Query   → Request for information
52.16 — User Interaction Model

Toda interacción puede seguir:

INPUT
  ↓
UNDERSTAND
  ↓
CONTEXTUALIZE
  ↓
INTERPRET
  ↓
DECIDE
  ↓
EXECUTE
  ↓
RESPOND
  ↓
OBSERVE

Esto permite una experiencia coherente entre:

Web;
Mobile;
Voice;
Chat;
API;
AI;
Agents.
52.17 — User Context Engine

User Core debe disponer conceptualmente de un:

User Context Engine

Su responsabilidad es construir el contexto necesario para cada interacción.

User
 │
 ├── Identity
 ├── Profile
 ├── Tenant
 ├── Organization
 ├── Session
 ├── Device
 ├── Preferences
 ├── Permissions
 ├── Policies
 ├── History
 ├── Goals
 └── Current Intent
        │
        ▼
 User Context Engine
        │
        ▼
 Interaction Context
52.18 — Context Resolution

No toda información debe utilizarse en cada interacción.

El Context Engine debe resolver:

Required Context
        ↓
Available Context
        ↓
Permission Check
        ↓
Privacy Check
        ↓
Policy Check
        ↓
Resolved Context

Esto es especialmente importante cuando interviene AI.

52.19 — User Decision Engine

User Core puede contener conceptualmente un:

User Decision Engine

Su función es coordinar decisiones relacionadas con la experiencia.

Intent
 ↓
Context
 ↓
Preferences
 ↓
Policies
 ↓
History
 ↓
Available Capabilities
 ↓
Decision

La decisión puede ser:

ejecutar;
pedir información;
recomendar;
bloquear;
solicitar aprobación;
delegar;
utilizar AI.
52.20 — Rules vs Policies

La separación establecida en Application Core debe mantenerse.

Rule

Describe cómo debe comportarse el sistema.

Policy

Define qué está permitido.

Ejemplo:

Rule:
Recommend next workout based on progress.

Policy:
Do not expose private data without authorization.

User Core coordina ambas.

52.21 — Personalization

User Core es uno de los puntos centrales de personalización.

Puede combinar:

User Profile
+
Preferences
+
Context
+
History
+
Goals
+
Behavior
+
Permissions
+
AI

para generar:

Personalized Experience
52.22 — Personalization Context

La personalización debe ser contextual.

El mismo usuario puede recibir diferentes experiencias:

Morning
 ↓
Training Experience

Work Hours
 ↓
Business Experience

Evening
 ↓
Personal Experience

La personalización no debería significar solamente:

“mostrar contenido diferente”.

Debe poder modificar:

recomendaciones;
workflows;
interfaces;
notifications;
AI behavior;
agent suggestions.
52.23 — User Preferences

User Core consume las preferencias definidas en Foundation.

Ejemplo:

Preference
 ↓
User Core
 ↓
Decision
 ↓
Experience

Una preferencia puede influir en:

canal;
frecuencia;
idioma;
presentación;
recomendaciones;
comportamiento de AI.
52.24 — User Goals

Los objetivos permiten que EVOXA deje de reaccionar solamente a acciones.

Ejemplo:

User Goal:
Improve fitness

        ↓

User Core

        ↓

Recommended Actions

        ↓

Application Capabilities

Esto permite experiencias orientadas a resultados.

52.25 — Goal State

Los objetivos pueden tener estados:

Created
 ↓
Active
 ↓
Progressing
 ↓
Completed

También:

Paused
Abandoned
Replaced
Expired

User Core puede coordinar las transiciones.

52.26 — User History

User Core puede consumir historial para comprender contexto.

Ejemplo:

User History
├── Interactions
├── Actions
├── Decisions
├── Goals
├── Experiences
├── Recommendations
├── AI Conversations
└── Agent Actions

Pero debe mantenerse la separación entre:

historial operacional

y:

memoria de AI.

52.27 — User Memory

La memoria relacionada con AI debe estar gobernada.

Podemos distinguir:

Short-Term Context
Long-Term User Memory
Application Memory
AI Memory
Agent Memory

No todo debe compartirse automáticamente.

El acceso debe depender de:

Permission
+
Policy
+
Consent
+
Context
52.28 — User Core and AI

AI puede participar en User Core para:

interpretar intents;
clasificar solicitudes;
recomendar acciones;
personalizar experiencias;
resumir contexto;
detectar patrones;
anticipar necesidades.

Arquitectura:

User
 ↓
User Core
 ↓
AI Service
 ↓
AI Runtime
 ↓
Model
 ↓
Result
 ↓
User Core

AI no debe reemplazar al Core.

AI aporta inteligencia.

User Core conserva la coordinación y governance.

52.29 — AI Decision Support

AI puede generar una recomendación:

AI
 ↓
Recommendation
 ↓
User Core
 ↓
Policy Validation
 ↓
Decision

La decisión final puede seguir siendo determinista.

52.30 — AI Autonomous Decision

En escenarios permitidos:

AI
 ↓
Decision
 ↓
Policy
 ↓
Risk
 ↓
Automatic Execution

El nivel de autonomía depende de la política.

52.31 — User Core and Agents

Los Agents deben interactuar con User Core mediante un modelo controlado.

User
 ↓
Intent
 ↓
User Core
 ↓
Delegation
 ↓
Agent
 ↓
Capability
 ↓
Application

Esto evita:

Agent
 ↓
Direct User Database Access

El Agent debe recibir el contexto autorizado.

52.32 — Agent Delegation

User Core puede administrar la delegación:

User Intent
      ↓
Delegation
      ↓
Agent
      ↓
Allowed Capabilities
      ↓
Execution

La delegación puede tener:

scope;
duration;
permissions;
budget;
risk;
approval;
expiration.
52.33 — Human-in-the-Loop

Cuando una acción es sensible:

Agent
 ↓
Action Proposal
 ↓
User Core
 ↓
Human Approval
 ↓
Execution

User Core actúa como una de las capas que mantiene la relación entre:

intención humana y acción automatizada.

52.34 — User Interaction Orchestration

Las interacciones complejas pueden requerir múltiples pasos.

Ejemplo:

User Intent
      ↓
Check Context
      ↓
Check Permissions
      ↓
Resolve Capability
      ↓
Execute
      ↓
Wait
      ↓
Receive Event
      ↓
Update State
      ↓
Respond

Esto convierte User Core en un coordinador de experiencias complejas.

52.35 — Interaction State Machine

Una interacción puede representarse como:

RECEIVED
   ↓
INTERPRETED
   ↓
VALIDATED
   ↓
AUTHORIZED
   ↓
DECIDED
   ↓
EXECUTING
   ↓
COMPLETED

Estados alternativos:

WAITING
REQUIRES_APPROVAL
FAILED
CANCELLED
REJECTED
52.36 — User Core Transactions

User Core puede coordinar operaciones que afecten múltiples elementos.

Por ejemplo:

Create Goal
 ↓
Create Workflow
 ↓
Schedule Action
 ↓
Notify User

Si una parte falla, puede requerirse compensación.

Action
 ↓
Failure
 ↓
Compensation
52.37 — Idempotency

Las acciones de usuario deben poder protegerse contra duplicación.

Ejemplo:

User clicks twice
        ↓
Same Command ID
        ↓
Single Execution

Esto es importante para:

pagos;
reservas;
creación;
delegaciones;
workflows.
52.38 — User Core Error Model

Los errores deben ser estructurados.

Por ejemplo:

INVALID_INPUT
UNAUTHORIZED
FORBIDDEN
CONTEXT_INVALID
POLICY_BLOCKED
CAPABILITY_UNAVAILABLE
DEPENDENCY_FAILURE
TIMEOUT
REQUIRES_APPROVAL
CONFLICT

El usuario no debería recibir únicamente errores técnicos.

Debe existir una traducción hacia una experiencia comprensible.

52.39 — User Experience Response

User Core debe separar:

Technical Result
        ↓
User Meaning
        ↓
Experience Response

Por ejemplo:

Technical:
CAPABILITY_UNAVAILABLE

User Experience:
"No podemos completar esta acción en este momento."
52.40 — User Feedback

La interacción no termina con la respuesta.

Puede continuar con:

Action
 ↓
Result
 ↓
User Feedback
 ↓
Event
 ↓
Learning

El feedback puede ser:

explícito;
implícito;
comportamiento posterior;
rating;
correction;
rejection.
52.41 — User Learning Context

EVOXA puede aprender patrones de interacción.

Por ejemplo:

User repeatedly rejects recommendation
        ↓
Pattern detected
        ↓
Personalization adjustment

Pero los cambios deben estar gobernados y respetar privacidad y consentimiento.

52.42 — User Core Observability

Toda interacción importante debe ser observable.

Debe poder responder:

What did the user request?
What context was used?
What decision was made?
What policy applied?
What capability executed?
What was the result?
What AI participated?
What Agent participated?

Esto crea:

User Interaction Observability
52.43 — Correlation Context

Cada interacción debe poder mantener un identificador de correlación.

User
 ↓
Intent ID
 ↓
Correlation ID
 ↓
Application
 ↓
Service
 ↓
Capability
 ↓
Component

Esto permite reconstruir una interacción completa.

52.44 — User Audit

Las acciones relevantes deben quedar auditadas.

Ejemplo:

User:
U123

Action:
Delegated Task

Agent:
A45

Capability:
CreateWorkout

Approval:
Required

Result:
Approved

Timestamp:
...

Esto será fundamental para Agents y AI.

52.45 — User Core Security

User Core debe aplicar:

Identity
 ↓
Authentication
 ↓
Authorization
 ↓
Policy
 ↓
Context
 ↓
Action

Nunca debería asumir que el hecho de que una solicitud provenga de un usuario autenticado significa que puede ejecutar cualquier acción.

52.46 — User Contextual Authorization

La autorización puede depender del contexto.

User
+
Tenant
+
Application
+
Resource
+
Action
+
Device
+
Risk
+
Policy

Esto permite decisiones más precisas.

52.47 — User Core Resilience

User Core debe tolerar fallos.

Puede utilizar:

retries;
timeout;
fallback;
circuit breaker;
cached context;
graceful degradation.

Ejemplo:

Recommendation Service
        ↓
Unavailable
        ↓
User Core
        ↓
Fallback Recommendation

La aplicación puede continuar funcionando.

52.48 — User Core Performance

La experiencia del usuario requiere baja latencia.

User Core debe optimizar:

context resolution;
permission checks;
intent processing;
decision latency;
AI calls;
agent coordination.

Una arquitectura inteligente no debe convertirse en una arquitectura lenta.

52.49 — User Core Caching

Puede utilizarse caching para:

profile;
preferences;
permissions;
context;
recommendations;
capabilities.

Pero debe controlarse:

Cache
 ↓
Freshness
 ↓
Invalidation
 ↓
Consistency
52.50 — User Core Events Architecture

User Core puede adoptar un modelo event-driven.

User Action
    ↓
Command
    ↓
Core
    ↓
State Change
    ↓
Event
    ↓
Subscribers

Esto permite desacoplar:

notifications;
analytics;
recommendations;
audit;
AI;
Agents.
52.51 — User Core and Eventual Consistency

No todas las actualizaciones necesitan ser instantáneamente consistentes.

Por ejemplo:

Profile Update
 ↓
Event
 ↓
Analytics
 ↓
Recommendation Model

La recomendación puede actualizarse posteriormente.

52.52 — User Core Composition

User Core puede coordinar múltiples servicios:

User Core
 │
 ├── Identity Service
 ├── Profile Service
 ├── Preference Service
 ├── Permission Service
 ├── Context Service
 ├── Recommendation Service
 ├── AI Service
 ├── Agent Service
 └── Notification Service

El Core no debería duplicar toda la lógica de estos servicios.

Debe coordinarlos.

52.53 — User Core Dependency Graph

Las dependencias deben ser visibles:

User Core
 │
 ├── Identity
 ├── Context
 ├── Permissions
 ├── Preferences
 ├── AI
 └── Agents

Esto permite detectar:

dependencias críticas;
latencias;
fallos;
ciclos;
costos.
52.54 — User Core Lifecycle

Aunque el capítulo 60 será el lifecycle completo, User Core debe mantener estados propios.

INITIALIZING
    ↓
READY
    ↓
ACTIVE
    ↓
DEGRADED
    ↓
RECOVERING
    ↓
READY

Esto representa el estado operacional del Core.

52.55 — User Core Configuration

Debe poder configurarse:

policies;
feature flags;
AI behavior;
personalization;
thresholds;
timeout;
fallback;
autonomy levels.

Ejemplo:

Agent Autonomy:
LOW

AI Personalization:
HIGH

Approval Threshold:
CRITICAL

Recommendation:
ENABLED
52.56 — User Core Feature Flags

Las capacidades pueden habilitarse progresivamente.

User
 ↓
Tenant
 ↓
Application
 ↓
Feature Flag
 ↓
Feature

Esto permite pruebas controladas.

52.57 — Multi-Tenant User Core

El Core debe operar dentro del contexto correcto:

User
 ↓
Tenant
 ↓
Application
 ↓
User Core

No debe existir contaminación de contexto entre tenants.

52.58 — User Core Intelligence

La inteligencia puede utilizar:

Context
+
History
+
Goals
+
Preferences
+
AI
+
Feedback

para mejorar la experiencia.

Esto permite pasar de:

Reactive User Experience

a:

Context-Aware Experience

y posteriormente:

Predictive Experience
52.59 — Predictive User Core

El Core puede anticipar necesidades.

Ejemplo conceptual:

User Behavior
      ↓
Pattern Detection
      ↓
Prediction
      ↓
Recommended Action
      ↓
User Experience

La predicción no debería convertirse automáticamente en acción.

Puede primero producir:

Suggestion
52.60 — Proactive User Experience

La evolución puede ser:

User asks
 ↓
System responds

hacia:

System understands context
 ↓
Predicts likely need
 ↓
Offers action

Y posteriormente:

Policy allows
 ↓
Agent prepares action
 ↓
User approves
 ↓
Execution
52.61 — Autonomous User Core

En escenarios de baja criticidad:

Observe
 ↓
Understand
 ↓
Predict
 ↓
Decide
 ↓
Act
 ↓
Validate

Pero la autonomía debe estar limitada por:

permissions;
policies;
consent;
risk;
budgets;
delegation.
52.62 — User Core Governance

Toda decisión importante debe ser gobernable.

Debe existir:

Decision
 ↓
Policy
 ↓
Risk
 ↓
Authorization
 ↓
Execution

Esto permite construir un User Core confiable.

52.63 — User Core Decision Trace

Una decisión puede registrar:

Decision ID
User
Intent
Context
Rules
Policies
AI Input
AI Recommendation
Final Decision
Capability
Result
Timestamp

Esto crea:

Decision Traceability

52.64 — User Core Digital Twin

En etapas avanzadas, User Core puede utilizar una representación dinámica del estado del usuario.

User Digital Representation
        │
        ├── Current State
        ├── Goals
        ├── Preferences
        ├── Context
        ├── Intent
        ├── History
        └── Delegations

Esto no reemplaza al usuario.

Representa el contexto necesario para operar.

52.65 — User Core Architecture

La arquitectura conceptual completa:

                         USER
                           │
                    USER FOUNDATION
                           │
                           ▼
                       USER CORE
                           │
        ┌──────────────────┼──────────────────┐
        │                  │                  │
        ▼                  ▼                  ▼
      STATE             CONTEXT             INTENT
        │                  │                  │
        └──────────────────┼──────────────────┘
                           ▼
                        DECISION
                           │
                ┌──────────┼──────────┐
                ▼          ▼          ▼
             COMMAND      QUERY      EVENT
                │          │          │
                └──────────┼──────────┘
                           ▼
                     ORCHESTRATION
                           │
          ┌────────────────┼────────────────┐
          ▼                ▼                ▼
      APPLICATION         AI              AGENT
          │                │                │
          └────────────────┼────────────────┘
                           ▼
                       EXPERIENCE
                           │
                           ▼
                        FEEDBACK
                           │
                           ▼
                      OBSERVATION
                           │
                           ▼
                      INTELLIGENCE
52.66 — User Core Responsibility Model

La responsabilidad queda claramente definida:

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
52.67 — User Core Maturity

La madurez puede evolucionar:

Level 1 — Reactive

Responde a acciones.

Level 2 — Structured

Administra estado e interacciones.

Level 3 — Contextual

Utiliza contexto.

Level 4 — Personalized

Adapta experiencias.

Level 5 — Intelligent

Utiliza AI para comprender y recomendar.

Level 6 — Predictive

Anticipa necesidades.

Level 7 — Agent-Enabled

Coordina Agents.

Level 8 — Autonomous

Ejecuta acciones gobernadas.

Level 9 — Adaptive

Aprende de resultados.

52.68 — Principios fundamentales
User Foundation defines; User Core coordinates.
Intent must be explicit.
Context must be resolved before action.
Commands represent intent to change.
Queries represent information requests.
Events represent facts.
State must be observable.
Decisions must be traceable.
Policies must govern decisions.
AI supports intelligence but does not bypass governance.
Agents operate through controlled delegation.
Personalization must respect privacy and consent.
User context must remain tenant-aware.
Interactions must be resilient.
Critical actions must support human approval.
User experience should become increasingly contextual and proactive.
Autonomy must be proportional to risk.
The user remains the ultimate authority over delegated actions.
52.69 — Definición final

User Core es la capa central de EVOXA responsable de coordinar el estado, contexto, intención, decisiones, comandos, consultas, eventos, preferencias, objetivos, interacciones, personalización y relaciones del usuario con Applications, AI y Agents, transformando la identidad y contexto definidos por User Foundation en comportamiento coordinado, observable, seguro, personalizado e inteligente.

52.70 — Visión final

La evolución del User Core puede resumirse:

Reactive
   ↓
Structured
   ↓
Contextual
   ↓
Personalized
   ↓
Intelligent
   ↓
Predictive
   ↓
Agent-Enabled
   ↓
Autonomous
   ↓
Adaptive

Y el ciclo conceptual:

                         USER
                           │
                        CONTEXT
                           │
                         INTENT
                           │
                        DECISION
                           │
                     ORCHESTRATION
                           │
                ┌──────────┼──────────┐
                ▼          ▼          ▼
             COMMAND     QUERY      EVENT
                │          │          │
                └──────────┼──────────┘
                           ▼
                    APPLICATION / AI
                           │
                           ▼
                         AGENT
                           │
                           ▼
                       EXPERIENCE
                           │
                           ▼
                        FEEDBACK
                           │
                           ▼
                      INTELLIGENCE
                           │
                           ▼
                       ADAPTATION

Con esto queda establecido 52 — User Core.

El siguiente capítulo será:

53 — User Runtime

donde pasaremos de coordinar lo que el usuario quiere y cómo EVOXA responde a definir cómo se ejecuta físicamente esa interacción a través de sesiones, dispositivos, canales, aplicaciones, tiempo real, web, mobile, voz, chat y otros puntos de contacto.
