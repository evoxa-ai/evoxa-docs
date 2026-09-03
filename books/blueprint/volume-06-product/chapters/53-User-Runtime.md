53 — User Runtime
53.1 — Introducción

En 51 — User Foundation definimos quién es el usuario.

En 52 — User Core definimos cómo EVOXA comprende y coordina:

estado;
contexto;
intención;
decisiones;
comandos;
consultas;
eventos;
personalización;
AI;
Agents.

Ahora debemos definir una capa diferente:

¿Cómo se ejecuta realmente la interacción del usuario con EVOXA?

Esta es la responsabilidad de User Runtime.

User Runtime constituye la capa de ejecución de la experiencia del usuario. Gestiona las condiciones necesarias para que una interacción pueda ocurrir de forma segura, consistente, resiliente y observable independientemente del canal utilizado.

User
  ↓
User Foundation
  ↓
User Core
  ↓
USER RUNTIME
  ↓
Application / Services / Capabilities
53.2 — Definición

User Runtime es la capa responsable de ejecutar y administrar las interacciones activas entre el usuario y el ecosistema EVOXA, gestionando sesiones, dispositivos, canales, conexiones, contexto de ejecución, requests, eventos en tiempo real, recursos, seguridad, rendimiento, resiliencia, sincronización y continuidad de experiencia.

Su objetivo es convertir:

la intención coordinada por User Core

en:

una interacción ejecutable y observable.

53.3 — User Runtime vs Application Runtime

Esta separación es fundamental.

Application Runtime

Ejecuta:

aplicaciones;
procesos;
workers;
jobs;
workflows;
servicios;
workloads.
User Runtime

Ejecuta:

sesiones;
interacción;
canales;
conexiones;
requests del usuario;
presencia;
contexto de interacción;
sincronización de experiencia.
                    USER
                      │
                      ▼
                 USER RUNTIME
                      │
             Interaction Layer
                      │
                      ▼
              APPLICATION RUNTIME
                      │
                      ▼
                   SERVICES

Por lo tanto:

User Runtime ejecuta la interacción; Application Runtime ejecuta la aplicación.

53.4 — User Runtime como Interaction Execution Layer

El User Runtime puede entenderse como una capa entre el usuario y las aplicaciones.

Human
  ↓
Device
  ↓
Channel
  ↓
User Runtime
  ↓
User Core
  ↓
Application
  ↓
Capability
  ↓
Component

Esta capa permite que la experiencia sea independiente del dispositivo.

53.5 — Canales de interacción

EVOXA debe poder soportar diferentes canales:

Web
Mobile
Desktop
Tablet
Voice
Chat
API
Wearable
TV
IoT
External Systems

El usuario debe poder cambiar de canal sin perder necesariamente su contexto.

Ejemplo:

Mobile
  ↓
Start Interaction
  ↓
Web
  ↓
Continue
  ↓
Voice
  ↓
Complete
53.6 — Omnichannel Experience

User Runtime proporciona la infraestructura conceptual para una experiencia omnicanal.

El objetivo es:

un usuario, múltiples canales, contexto coherente.

                    USER
                      │
          ┌───────────┼───────────┐
          ▼           ▼           ▼
        WEB         MOBILE       VOICE
          │           │           │
          └───────────┼───────────┘
                      ▼
                 USER RUNTIME
                      │
                      ▼
                  USER CORE
53.7 — Session Runtime

La sesión es uno de los elementos principales de User Runtime.

Una sesión representa una interacción temporal activa.

Session
├── Session ID
├── User
├── Device
├── Channel
├── Application
├── Tenant
├── Authentication
├── Context
├── Permissions
├── Locale
├── Timezone
├── Start Time
├── Last Activity
├── Expiration
└── State
53.8 — Session Lifecycle

Una sesión puede evolucionar:

CREATED
   ↓
AUTHENTICATING
   ↓
AUTHENTICATED
   ↓
ACTIVE
   ↓
IDLE
   ↓
EXPIRED

Estados alternativos:

REVOKED
SUSPENDED
TERMINATED
53.9 — Session Continuity

Una sesión no necesariamente debe quedar limitada a un dispositivo.

EVOXA puede mantener continuidad contextual.

User
 │
 ├── Mobile Session
 │
 ├── Web Session
 │
 └── Voice Session

El sistema puede mantener una relación común:

User Context
      ↓
Interaction Context
      ↓
Multiple Sessions

Esto permite experiencias continuas.

53.10 — Device Runtime

User Runtime debe administrar el contexto del dispositivo.

Device
├── Device ID
├── Type
├── OS
├── Application Version
├── Capabilities
├── Network
├── Security State
├── Trust
└── Runtime State
53.11 — Device Capabilities

Un dispositivo puede informar:

Camera
Microphone
GPS
Bluetooth
Biometrics
Notifications
Storage
Sensors
Network

El User Runtime puede utilizar esta información para adaptar la interacción.

Por ejemplo:

Voice Request
      ↓
Microphone available?
      ↓
Yes
      ↓
Voice Interaction
53.12 — Device Trust

Un dispositivo puede tener un estado de confianza:

Unknown
 ↓
Registered
 ↓
Verified
 ↓
Trusted

También:

Compromised
Revoked
Blocked

Esto puede influir en políticas de acceso.

53.13 — Request Runtime

Cada request de usuario debe tener un contexto de ejecución.

Request
├── Request ID
├── User
├── Session
├── Device
├── Channel
├── Application
├── Tenant
├── Permissions
├── Policies
├── Locale
├── Trace ID
└── Deadline
53.14 — Request Lifecycle
Request Received
      ↓
Authenticate
      ↓
Resolve Context
      ↓
Authorize
      ↓
Validate
      ↓
Route
      ↓
Execute
      ↓
Response
      ↓
Observe

Si ocurre un error:

Request
 ↓
Failure
 ↓
Retry / Fallback
 ↓
Response
53.15 — Runtime Context

El User Runtime debe construir un contexto de ejecución.

User
+
Session
+
Device
+
Channel
+
Application
+
Tenant
+
Permissions
+
Policies
+
Intent
+
Request

se convierte en:

User Execution Context

Este contexto se entrega a User Core y a las capas que necesitan ejecutarlo.

53.16 — Context Propagation

El contexto debe propagarse de forma controlada.

User Runtime
      ↓
User Core
      ↓
Application Runtime
      ↓
Application Services
      ↓
Capability
      ↓
Component

No todos los elementos necesitan recibir todo el contexto.

Debe aplicarse:

Context Minimization

53.17 — Channel Abstraction

User Runtime debe abstraer las diferencias entre canales.

Por ejemplo:

Mobile Event
Web Event
Voice Event
Chat Event
API Request
       ↓
Channel Adapter
       ↓
Normalized Interaction

Así User Core recibe una representación común.

53.18 — Interaction Normalization

Ejemplo:

Voice:
"Quiero comenzar mi entrenamiento"

Chat:
"Comenzar entrenamiento"

Button:
START WORKOUT

API:
POST /workouts/start

Todos pueden convertirse en:

Intent:
StartWorkout

User Runtime normaliza el canal.

User Core interpreta y coordina.

53.19 — Real-Time Runtime

Muchas experiencias requieren tiempo real.

User Runtime debe poder soportar:

WebSockets;
Server-Sent Events;
streaming;
push events;
live updates;
presence;
real-time notifications.

Arquitectura:

Application Event
       ↓
User Runtime
       ↓
Active Sessions
       ↓
Connected Devices
       ↓
Real-Time Experience
53.20 — User Presence

Presence representa si el usuario está disponible o interactuando.

Estados posibles:

ONLINE
ACTIVE
IDLE
AWAY
OFFLINE

También pueden existir estados por canal:

User
├── Web → Active
├── Mobile → Idle
└── Voice → Offline
53.21 — Notification Runtime

Las notificaciones deben considerar contexto.

Event
 ↓
Notification Decision
 ↓
User Preferences
 ↓
Channel Availability
 ↓
Notification

El runtime puede determinar:

qué dispositivo;
qué canal;
cuándo;
si el usuario está activo;
si la notificación puede interrumpir.
53.22 — Push Runtime

Puede gestionar:

Push Token
Device
Application
User
Notification
Delivery
Status

El objetivo es separar:

generación de la notificación

de:

entrega de la notificación.

53.23 — Messaging Runtime

El usuario puede participar en conversaciones:

User
 ↕
Message
 ↕
Application / AI / Agent

El runtime debe manejar:

conexión;
delivery;
ordering;
retries;
acknowledgements;
presence;
streaming.
53.24 — Voice Runtime

Para interacciones de voz:

Voice Input
    ↓
Speech Processing
    ↓
Intent
    ↓
User Core
    ↓
Action
    ↓
Voice Response

User Runtime administra la ejecución del canal.

53.25 — Chat Runtime

El canal conversacional puede funcionar como:

Message
 ↓
Session
 ↓
Context
 ↓
User Core
 ↓
AI
 ↓
Agent
 ↓
Response

El runtime debe mantener la continuidad de la conversación.

53.26 — Streaming Runtime

Para AI y datos en tiempo real:

Request
 ↓
Streaming Response
 ↓
Partial Result
 ↓
Partial Result
 ↓
Final Result

Esto permite:

AI streaming;
audio streaming;
video;
progress updates;
long-running tasks.
53.27 — Long-Running Interaction

No todas las interacciones terminan inmediatamente.

Por ejemplo:

User Request
      ↓
Long Running Workflow
      ↓
Processing
      ↓
Background
      ↓
Completed

El User Runtime puede mantener el vínculo con la sesión.

User
 ↓
Request ID
 ↓
Job ID
 ↓
Progress
 ↓
Completion Event
53.28 — Interaction Continuity

Una tarea iniciada en un canal puede continuar posteriormente.

Mobile
 ↓
Create Task
 ↓
Background Processing
 ↓
Web
 ↓
Review Result
 ↓
Voice
 ↓
Confirm

El contexto debe mantenerse de forma coherente.

53.29 — Runtime State Synchronization

Cuando existen múltiples dispositivos:

Device A
   │
   ├── State
   │
   ▼
User Runtime
   │
   ▼
Shared User Context
   │
   ├── Device B
   └── Device C

El objetivo es evitar estados contradictorios.

53.30 — Offline Runtime

Los dispositivos móviles pueden perder conectividad.

User Runtime debe contemplar:

Online
 ↓
Offline
 ↓
Local Interaction
 ↓
Reconnect
 ↓
Sync
 ↓
Resolve Conflicts
53.31 — Offline Queue

Las operaciones que puedan ejecutarse posteriormente pueden mantenerse en una cola local.

User Action
 ↓
Local Queue
 ↓
Offline
 ↓
Reconnect
 ↓
Server

Debe existir protección contra:

duplicación;
conflictos;
operaciones expiradas.
53.32 — Conflict Resolution

Cuando dos dispositivos modifican el mismo recurso:

Device A
 ↓
Update A

Device B
 ↓
Update B

      ↓

Conflict Resolution

Las estrategias pueden incluir:

last-write-wins;
version checking;
merge;
user resolution;
domain-specific rules.
53.33 — Runtime Security

User Runtime es una frontera crítica de seguridad.

Debe controlar:

authentication;
session security;
device security;
authorization;
token validation;
request integrity;
rate limits;
suspicious activity.
53.34 — Session Security

Las sesiones deben protegerse contra:

session hijacking;
token theft;
replay;
fixation;
unauthorized reuse.

Puede utilizarse:

Session
 ↓
Device Binding
 ↓
Token
 ↓
Risk Evaluation
53.35 — Runtime Rate Limiting

El runtime puede aplicar límites por:

User
Session
Device
IP
Tenant
Application
Capability
Channel

Ejemplo:

100 requests/minute

o límites más estrictos para operaciones sensibles.

53.36 — Runtime Abuse Protection

Puede detectar:

request floods;
automated abuse;
suspicious sessions;
credential attacks;
unusual interaction patterns.

El User Runtime puede bloquear o degradar una interacción antes de que alcance la Application.

53.37 — Runtime Privacy

El contexto runtime puede contener información sensible.

Por eso debe minimizar:

Stored Context
Transmitted Context
Logged Context
Cached Context

La regla debe ser:

Use only the minimum context required for execution.

53.38 — Runtime Logging

No todo contexto debe aparecer en logs.

El runtime debe separar:

Operational Metadata
     vs
User Data

Los logs deben evitar exponer información innecesaria.

53.39 — Runtime Performance

User Runtime tiene impacto directo en la experiencia.

Debe observar:

latency;
throughput;
connection time;
request duration;
rendering-related signals;
event delivery;
synchronization latency;
AI response latency.
53.40 — Runtime SLO

Pueden definirse objetivos como:

Session Availability
Request Latency
Message Delivery
Notification Delivery
Realtime Availability
Synchronization Time

Esto permite medir la calidad de la experiencia.

53.41 — Runtime Resilience

El User Runtime debe soportar fallos.

Puede utilizar:

retry;
timeout;
circuit breaker;
fallback;
queue;
buffering;
reconnection;
graceful degradation.

Ejemplo:

WebSocket Failure
      ↓
Reconnect
      ↓
Restore Session
      ↓
Resume Context
53.42 — Reconnection

Las conexiones persistentes pueden romperse.

CONNECTED
    ↓
DISCONNECTED
    ↓
RECONNECTING
    ↓
CONNECTED

El runtime puede intentar recuperar:

sesión;
subscription;
context;
pending events.
53.43 — Graceful Degradation

Si un servicio no está disponible:

Full Experience
      ↓
Dependency Failure
      ↓
Reduced Experience

Por ejemplo:

AI Recommendations unavailable

Core Application
      ↓
Still operational
53.44 — User Runtime and Application Runtime

La interacción completa queda:

USER
 │
 ▼
USER RUNTIME
 │
 ├── Session
 ├── Device
 ├── Channel
 ├── Request
 ├── Realtime
 └── Context
 │
 ▼
USER CORE
 │
 ├── Intent
 ├── Decision
 ├── Command
 └── Event
 │
 ▼
APPLICATION RUNTIME
 │
 ▼
APPLICATION

Esta separación es esencial para EVOXA.

53.45 — User Runtime and AI

AI puede ser parte de la interacción:

User
 ↓
Channel
 ↓
User Runtime
 ↓
User Core
 ↓
AI
 ↓
Response
 ↓
User Runtime
 ↓
Channel
 ↓
User

El runtime gestiona la entrega.

AI gestiona inteligencia.

53.46 — AI Streaming

Cuando AI genera respuestas progresivamente:

User
 ↓
AI Request
 ↓
Token Stream
 ↓
User Runtime
 ↓
Channel
 ↓
User

El usuario puede recibir respuesta mientras el modelo todavía está procesando.

53.47 — User Runtime and Agents

Para Agents:

User
 ↓
User Runtime
 ↓
User Core
 ↓
Agent
 ↓
Capability
 ↓
Application

Cuando el Agent termina:

Agent Result
 ↓
User Core
 ↓
User Runtime
 ↓
User
53.48 — Agent Progress

Los Agents pueden ejecutar tareas largas.

User Runtime puede mostrar:

Agent Task
   ↓
Started
   ↓
Analyzing
   ↓
Executing
   ↓
Waiting
   ↓
Completed

Esto mejora la transparencia.

53.49 — Human Approval Runtime

Cuando una acción necesita aprobación:

Agent
 ↓
Approval Required
 ↓
User Runtime
 ↓
User
 ↓
Approve / Reject
 ↓
Agent

La interacción de aprobación debe ser segura y auditable.

53.50 — Runtime Experience State

El runtime puede mantener información temporal sobre la experiencia:

Current Screen
Current Flow
Current Interaction
Current Task
Current Channel
Current Device
Pending Action
Pending Approval

Este estado no necesariamente pertenece al Profile.

53.51 — Navigation Context

En aplicaciones complejas, el runtime puede conocer:

Application
 ↓
Module
 ↓
Domain
 ↓
Screen
 ↓
Component
 ↓
Interaction

Esto permite mantener continuidad de navegación.

53.52 — Runtime Feature Flags

La experiencia puede cambiar según:

User
Tenant
Application
Device
Channel
Region
Experiment

Ejemplo:

AI Assistant
   ↓
Enabled for 10% of users

El runtime puede aplicar esta decisión.

53.53 — Experimentation Runtime

EVOXA puede soportar experimentos controlados:

User
 ↓
Experiment Assignment
 ↓
Variant A / Variant B
 ↓
Experience
 ↓
Metrics

Esto permite evaluar experiencias sin modificar permanentemente la aplicación.

53.54 — User Runtime Observability

Debe existir observabilidad completa.

User Interaction
      ↓
Request
      ↓
Session
      ↓
Application
      ↓
Service
      ↓
Capability
      ↓
Component

Todo debe poder correlacionarse.

53.55 — Distributed Tracing

Puede utilizar:

User Interaction ID
        ↓
Session ID
        ↓
Request ID
        ↓
Trace ID
        ↓
Span
        ↓
Application
        ↓
Service

Esto permite reconstruir una experiencia completa.

53.56 — User Runtime Metrics

Algunas métricas:

Interaction
interactions/sec;
active users;
session duration.
Performance
latency;
throughput;
errors.
Connectivity
reconnect rate;
offline duration.
Realtime
delivery latency;
dropped events.
AI
response latency;
streaming duration.
Agent
task execution duration;
approval latency.
53.57 — Runtime Health

Puede existir:

Healthy
Degraded
Recovering
Unavailable
Maintenance

La salud debe poder evaluarse por canal.

Web → Healthy
Mobile → Healthy
Voice → Degraded
Realtime → Healthy
53.58 — Runtime Capacity

El User Runtime debe poder escalar.

Factores:

concurrent users;
active sessions;
requests;
WebSocket connections;
streaming;
notifications;
messages.

Puede utilizar:

Horizontal Scaling
Vertical Scaling
Connection Sharding
Load Balancing
Regional Distribution
53.59 — Edge Runtime

En arquitecturas avanzadas, parte de User Runtime puede ejecutarse cerca del usuario.

User
 ↓
Edge
 ↓
User Runtime
 ↓
Platform

Esto puede reducir:

latency;
bandwidth;
round trips.
53.60 — Regional User Runtime

Usuarios distribuidos globalmente pueden utilizar regiones.

Global User
      │
 ┌────┼────┐
 ▼    ▼    ▼
US   EU   LATAM
 │    │    │
Runtime Runtime Runtime

El sistema debe mantener coherencia de identidad y contexto según las políticas correspondientes.

53.61 — User Runtime Cost

Debe medirse:

active sessions;
connection resources;
bandwidth;
realtime;
messaging;
notification delivery;
AI streaming;
edge execution.

Esto permite optimizar la experiencia sin ignorar costos.

53.62 — User Runtime Governance

El runtime debe estar gobernado por:

security policies;
privacy policies;
session policies;
rate limits;
tenant policies;
channel policies;
AI policies;
Agent policies.
53.63 — User Runtime Lifecycle

Aunque 60 — User Lifecycle será la capa de evolución del usuario, el runtime tiene su propio ciclo operacional:

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

Una sesión también posee lifecycle:

Created
 ↓
Authenticated
 ↓
Active
 ↓
Idle
 ↓
Expired
53.64 — User Runtime Automation

El runtime puede automatizar:

reconnect;
session renewal;
failover;
scaling;
routing;
notification channel selection;
context synchronization.

Por ejemplo:

Connection Failure
      ↓
Detect
      ↓
Reconnect
      ↓
Restore Session
      ↓
Resume
53.65 — Intelligent User Runtime

AI puede ayudar a optimizar runtime.

Por ejemplo:

Traffic Pattern
      ↓
AI Prediction
      ↓
Capacity Forecast
      ↓
Pre-Scaling

También puede predecir:

congestion;
session spikes;
connection failures;
latency degradation.
53.66 — Adaptive Runtime

El runtime puede adaptar la experiencia.

Context
 ↓
Device
 ↓
Network
 ↓
User Preference
 ↓
Runtime Decision

Ejemplo:

Poor Network
 ↓
Reduce Payload
 ↓
Disable Heavy Features
 ↓
Prioritize Essential Interaction
53.67 — Autonomous Runtime

En un nivel superior:

Observe
 ↓
Predict
 ↓
Decide
 ↓
Scale
 ↓
Recover
 ↓
Validate

Siempre bajo políticas de EVOXA.

53.68 — User Runtime Architecture

La arquitectura completa:

                           USER
                             │
                             ▼
                       USER RUNTIME
                             │
          ┌──────────────────┼──────────────────┐
          │                  │                  │
          ▼                  ▼                  ▼
       SESSION            DEVICE             CHANNEL
          │                  │                  │
          └──────────────────┼──────────────────┘
                             ▼
                         REQUEST
                             │
                             ▼
                      EXECUTION CONTEXT
                             │
              ┌──────────────┼──────────────┐
              ▼              ▼              ▼
          REAL-TIME       PRESENCE      SYNCHRONIZATION
              │              │              │
              └──────────────┼──────────────┘
                             ▼
                        USER CORE
                             │
                             ▼
                      APPLICATION RUNTIME
                             │
                    ┌────────┼────────┐
                    ▼        ▼        ▼
                SERVICES     AI      AGENTS
                    │        │        │
                    └────────┼────────┘
                             ▼
                       EXPERIENCE
53.69 — User Runtime como Experience Execution Fabric

A partir de esta arquitectura podemos definir:

User Experience Runtime Fabric

Esta fabric conecta:

Users
Sessions
Devices
Channels
Applications
AI
Agents
Events
Notifications
Realtime

y proporciona:

conectividad;
continuidad;
ejecución;
sincronización;
seguridad;
observabilidad;
resiliencia.
53.70 — Principios fundamentales
1. Interaction First

El runtime está diseñado alrededor de la interacción.

2. Channel Agnostic

La lógica no debe depender de un único canal.

3. Session Aware

La sesión debe ser explícita.

4. Context Aware

Toda interacción debe tener contexto.

5. Secure by Default

Toda conexión debe estar protegida.

6. Privacy by Design

El contexto debe minimizarse.

7. Real-Time Ready

Debe soportar interacción en tiempo real.

8. Offline Capable

Los canales apropiados deben soportar desconexión.

9. Resilient

Debe recuperarse ante fallos.

10. Observable

Las interacciones deben ser trazables.

11. Scalable

Debe soportar crecimiento de usuarios y sesiones.

12. AI Ready

Debe soportar interacción con AI.

13. Agent Ready

Debe soportar interacción con Agents.

14. Context Continuity

El usuario debe poder continuar su experiencia.

15. Adaptive

El runtime debe poder adaptarse al contexto.

53.71 — User Runtime Maturity

La evolución puede definirse:

Level 1 — Basic Runtime

Requests y sesiones básicas.

Level 2 — Managed Runtime

Sessions, devices y channels.

Level 3 — Contextual Runtime

Context propagation.

Level 4 — Omnichannel Runtime

Múltiples canales coherentes.

Level 5 — Real-Time Runtime

Streaming, events y presence.

Level 6 — Resilient Runtime

Offline, reconnection y graceful degradation.

Level 7 — Intelligent Runtime

AI-assisted optimization.

Level 8 — Autonomous Runtime

Self-healing y adaptive routing.

Level 9 — Adaptive Runtime

El runtime evoluciona según contexto, patrones y objetivos gobernados.

53.72 — Relación con las diez capas de User & Experience

La arquitectura queda:

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
53.73 — Definición final

User Runtime es la capa de ejecución de EVOXA responsable de administrar sesiones, dispositivos, canales, requests, conexiones, presencia, tiempo real, streaming, sincronización, continuidad de contexto, seguridad, rendimiento, escalabilidad, resiliencia y entrega de interacciones entre el usuario y Applications, AI y Agents, proporcionando una experiencia consistente, observable, segura, omnicanal y adaptable.

53.74 — Visión final

La evolución del User Runtime puede resumirse:

Basic Runtime
      ↓
Managed Runtime
      ↓
Contextual Runtime
      ↓
Omnichannel Runtime
      ↓
Real-Time Runtime
      ↓
Resilient Runtime
      ↓
Intelligent Runtime
      ↓
Autonomous Runtime
      ↓
Adaptive Runtime

Y conceptualmente:

                         USER
                           │
                       DEVICE
                           │
                       CHANNEL
                           │
                        SESSION
                           │
                        REQUEST
                           │
                    EXECUTION CONTEXT
                           │
                 ┌─────────┼─────────┐
                 ▼         ▼         ▼
              REALTIME   PRESENCE   SYNC
                 │         │         │
                 └─────────┼─────────┘
                           ▼
                       USER CORE
                           │
                           ▼
                     APPLICATION
                           │
                    ┌──────┴──────┐
                    ▼             ▼
                   AI           AGENTS
                    │             │
                    └──────┬──────┘
                           ▼
                      EXPERIENCE
                           │
                           ▼
                      ADAPTATION

Con 53 — User Runtime, ya tenemos definidas las tres primeras capas del Volume 06 — Users & Experience:

51 — User Foundation
        ↓
52 — User Core
        ↓
53 — User Runtime

La siguiente capa será 54 — User Services, donde definiremos el conjunto de servicios reutilizables que proporcionarán identidad, perfiles, preferencias, sesiones, privacidad, consentimiento, personalización, comunicación, recomendaciones, AI y soporte a Agents sobre este User Runtime.
