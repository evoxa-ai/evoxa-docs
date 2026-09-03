51 — User Foundation
51.1 — Introducción

User Foundation define la base estructural, conceptual, identitaria y de gobierno del usuario dentro de EVOXA.

En las capas anteriores establecimos que:

Platform Foundation define la base de la plataforma.
Application Foundation define la base de las aplicaciones.
User Foundation debe definir la base de la relación entre EVOXA y las personas, organizaciones y entidades que interactúan con el ecosistema.

El User Foundation no debe entenderse simplemente como una tabla users.

Un usuario puede tener:

identidad;
perfil;
preferencias;
contexto;
relaciones;
roles;
permisos;
organizaciones;
tenants;
dispositivos;
sesiones;
consentimiento;
privacidad;
actividad;
objetivos;
historial;
interacciones;
relación con AI;
relación con Agents.

Por lo tanto:

User Foundation define quién es el usuario, dentro de qué contexto existe, qué relación mantiene con EVOXA, qué puede hacer, qué información puede utilizarse sobre él y bajo qué condiciones puede interactuar con aplicaciones, AI, Agents y el resto de la plataforma.

51.2 — Definición

User Foundation es la capa fundamental que define la identidad, representación, contexto, pertenencia, relaciones, seguridad, privacidad, preferencias, ownership y metadatos de cada usuario dentro del ecosistema EVOXA.

Su responsabilidad es establecer las reglas estructurales sobre las cuales se construyen:

User Core
User Runtime
User Services
User Modules
User Domains
User Components
User Capabilities
User Contracts
User Lifecycle

User Foundation no ejecuta procesos complejos.

Primero establece:

qué es un usuario y cuáles son las condiciones bajo las cuales puede existir e interactuar dentro de EVOXA.

51.3 — El User como entidad de primera clase

EVOXA debe tratar al User como una entidad de primera clase.

No solamente:

User
├── id
├── email
└── password

Sino como:

User
├── Identity
├── Profile
├── Context
├── Organization
├── Tenant
├── Roles
├── Permissions
├── Preferences
├── Privacy
├── Consent
├── Relationships
├── Devices
├── Sessions
├── Activity
├── Goals
├── History
├── AI Relationship
├── Agent Relationship
├── Security
└── Lifecycle

Esto permite que EVOXA pueda comprender al usuario de forma estructurada.

51.4 — User Identity

La identidad representa quién es el usuario dentro de EVOXA.

Puede incluir:

User Identity
├── User ID
├── Identity Type
├── External IDs
├── Username
├── Email
├── Phone
├── Verification Status
├── Identity Provider
└── Identity Metadata

El User ID debe ser estable durante toda la existencia del usuario.

Otros atributos pueden cambiar.

Por ejemplo:

User ID
    ↓
stable

Email
    ↓
changeable

Phone
    ↓
changeable

Display Name
    ↓
changeable

Esto permite separar:

identidad estable de atributos modificables.

51.5 — Identity Types

EVOXA puede soportar diferentes tipos de identidad.

Human User

Persona física.

Organization User

Usuario asociado a una organización.

Service Identity

Identidad utilizada por servicios.

AI Identity

Identidad asociada a una entidad de inteligencia artificial.

Agent Identity

Identidad asociada a un Agent.

External Identity

Identidad proveniente de un sistema externo.

La arquitectura puede representarse:

Identity
   │
   ├── Human
   ├── Organization
   ├── Service
   ├── AI
   ├── Agent
   └── External

Esto será importante para los siguientes capítulos.

51.6 — Human User vs AI vs Agent

Aunque pueden compartir ciertos conceptos de identidad, no deben confundirse.

Human
  ↓
User

AI
  ↓
AI Identity

Agent
  ↓
Agent Identity

Un Agent puede actuar para un User, pero no debe convertirse en el User.

Debe mantenerse una relación explícita:

User
  │
  └── delegates
          ↓
        Agent

Esto permite saber:

quién solicitó una acción y quién la ejecutó.

51.7 — User Profile

El Profile representa los atributos descriptivos del usuario.

Puede incluir:

Profile
├── Name
├── Display Name
├── Avatar
├── Language
├── Locale
├── Timezone
├── Country
├── Contact Information
├── Demographic Attributes
└── Custom Attributes

El Profile debe mantenerse separado de la Identity.

Por ejemplo:

Identity
   ↓
Who is the user?

Profile
   ↓
What attributes describe the user?
51.8 — User Context

El usuario nunca interactúa en un vacío.

Cada interacción ocurre dentro de un contexto.

El contexto puede incluir:

User Context
├── User
├── Organization
├── Tenant
├── Application
├── Session
├── Device
├── Location
├── Language
├── Timezone
├── Permissions
├── Policies
├── Preferences
├── Current Intent
└── Security Context

Por ejemplo:

User
 ↓
Application
 ↓
Tenant
 ↓
Session
 ↓
Device
 ↓
Context
 ↓
Action

El mismo usuario puede tener diferentes contextos simultáneamente.

51.9 — Organization

Un User puede pertenecer a una o varias organizaciones.

Organization
     │
     ├── User A
     ├── User B
     ├── User C
     └── User D

La relación puede tener atributos:

role;
status;
permissions;
ownership;
department;
position;
scope;
start date;
end date.

Esto evita asumir que un usuario pertenece únicamente a una organización.

51.10 — Tenant

En una arquitectura multi-tenant, el usuario también debe tener un contexto de tenant.

Platform
   │
   ├── Tenant A
   │     ├── Users
   │     └── Applications
   │
   ├── Tenant B
   │     ├── Users
   │     └── Applications
   │
   └── Tenant C
         ├── Users
         └── Applications

Debe garantizarse:

aislamiento correcto de identidad, datos, permisos y contexto entre tenants.

51.11 — User Membership

La pertenencia del usuario a una organización o tenant debe modelarse explícitamente.

User
  │
  ▼
Membership
  │
  ├── Organization
  ├── Tenant
  ├── Role
  ├── Permissions
  ├── Status
  └── Scope

Esto permite que un mismo usuario pueda tener diferentes roles dependiendo del contexto.

Por ejemplo:

User
├── Organization A → Admin
├── Organization B → Viewer
└── Organization C → Manager
51.12 — Roles

Los roles representan conjuntos de responsabilidades.

Ejemplo:

Role
├── Administrator
├── Manager
├── User
├── Analyst
├── Developer
├── Operator
└── Viewer

Pero User Foundation no debe depender de nombres específicos de roles.

Debe definir el modelo estructural.

User
   ↓
Membership
   ↓
Role
   ↓
Permissions
51.13 — Permissions

Las permissions determinan qué puede hacer un usuario.

Ejemplo:

user.profile.read
user.profile.update
user.data.export
application.access
application.configure
agent.execute
agent.approve

La permission debe estar separada del rol.

Role
   ↓
Permissions

Así un mismo permiso puede utilizarse en distintos roles.

51.14 — User Scope

Las permissions deben tener scope.

Por ejemplo:

Global
Organization
Tenant
Application
Domain
Module
Resource

Ejemplo:

User
 ↓
Organization A
 ↓
Training Application
 ↓
Training Domain
 ↓
Workout Module
 ↓
Workout Resource

Esto permite autorización granular.

51.15 — User Preferences

Las preferencias representan cómo el usuario desea interactuar con EVOXA.

Pueden incluir:

Preferences
├── Language
├── Theme
├── Notifications
├── Communication
├── Privacy
├── Accessibility
├── Display
├── AI Behavior
└── Experience

Las preferencias deben ser explícitas.

No deben confundirse con políticas.

Preference
    ↓
User preference

Policy
    ↓
System rule
51.16 — User Privacy

Privacy debe formar parte de Foundation.

El usuario debe poder tener control sobre:

datos almacenados;
datos compartidos;
datos utilizados por AI;
datos utilizados para personalización;
datos compartidos con Agents;
integraciones externas;
comunicaciones.

La arquitectura puede representar:

User
 ↓
Privacy Profile
 ↓
Data Policies
 ↓
Consent
 ↓
Usage
51.17 — User Consent

El consentimiento representa una autorización explícita para determinados usos.

Ejemplo:

Consent
├── Purpose
├── Scope
├── Data
├── Recipient
├── Duration
├── Status
├── Version
└── Timestamp

Esto es especialmente importante para AI.

Por ejemplo:

User
 ↓
Consent
 ↓
AI Personalization

El consentimiento debe poder revocarse cuando corresponda.

51.18 — Data Ownership

EVOXA debe distinguir entre:

datos sobre el usuario

y:

datos propiedad del usuario.

El modelo debe permitir determinar:

Who owns the data?
Who can access it?
Who can process it?
Who can modify it?
Who can export it?
Who can delete it?

Esto será fundamental en los futuros sistemas de Data, AI y Governance.

51.19 — User Data Boundaries

El User Foundation debe establecer límites sobre el acceso a datos.

User
   │
   ▼
Data Access Policy
   │
   ├── Allowed
   ├── Restricted
   ├── Sensitive
   └── Forbidden

Los servicios superiores no deberían asumir que todo dato del usuario está disponible.

51.20 — User Security

Security debe acompañar la identidad.

Incluye:

authentication state;
identity verification;
authorization context;
security policies;
credentials;
MFA;
devices;
sessions;
risk;
security events.

La identidad debe poder representar:

Trusted
Verified
Restricted
Suspended
Compromised

sin modificar necesariamente la identidad original.

51.21 — Authentication vs Identity

Una distinción fundamental:

Identity

¿Quién eres?

Authentication

¿Puedes demostrar que eres esa identidad?

Authorization

¿Qué puedes hacer?

Identity
   ↓
Authentication
   ↓
Authorization
   ↓
Action

User Foundation define la identidad.

Las capas superiores implementarán los mecanismos de ejecución correspondientes.

51.22 — User Devices

El usuario puede interactuar desde múltiples dispositivos.

User
├── Mobile
├── Tablet
├── Desktop
├── Browser
├── Wearable
├── TV
└── Other Device

Cada dispositivo puede tener:

identity;
status;
trust;
capabilities;
operating system;
application versions;
security state.
51.23 — User Sessions

Una sesión representa una interacción temporal.

User
 ↓
Session
 ↓
Application
 ↓
Runtime

Una sesión puede incluir:

Session
├── Session ID
├── User
├── Device
├── Application
├── Tenant
├── Authentication
├── Permissions
├── Context
├── Start
├── Last Activity
└── Expiration

El User Foundation define la relación conceptual; User Runtime se encargará de la ejecución operacional.

51.24 — User Relationships

Los usuarios pueden relacionarse entre sí.

User A
   │
   ├── follows
   ├── collaborates
   ├── manages
   ├── coaches
   ├── belongs_to
   └── communicates_with

Las relaciones deben tener:

type;
source;
target;
status;
permissions;
scope;
timestamps.

Esto permitirá posteriormente construir capacidades sociales y colaborativas.

51.25 — User Relationship with Applications

El usuario mantiene relaciones con Applications.

User
  │
  ├── owns
  ├── administers
  ├── uses
  ├── configures
  └── subscribes

No todos los usuarios tienen el mismo tipo de relación.

51.26 — User Relationship with AI

EVOXA introduce una relación adicional:

User
   ↓
AI

La relación puede incluir:

personalization;
preferences;
context;
consent;
interaction history;
AI permissions;
AI memory;
AI behavior preferences.

Esto permite evolucionar desde:

“el usuario utiliza una IA”

hacia:

“la IA comprende el contexto autorizado del usuario y adapta su comportamiento dentro de límites definidos.”

51.27 — User AI Identity

Puede existir un perfil específico de relación con AI:

User AI Profile
├── AI Preferences
├── Personalization
├── Interaction Style
├── Context Permissions
├── Memory Permissions
├── Data Permissions
└── AI Consent

No significa que AI tenga acceso irrestricto a toda la información.

El acceso debe depender de:

Permission
+
Policy
+
Consent
+
Context
51.28 — User Relationship with Agents

Los Agents introducen una relación diferente.

User
   │
   ├── uses Agent
   ├── delegates to Agent
   ├── approves Agent
   └── revokes Agent

Un usuario puede delegar una tarea:

User
 ↓
Intent
 ↓
Agent
 ↓
Capability
 ↓
Execution

Pero la delegación debe estar limitada.

51.29 — Delegation Model

El User Foundation debe permitir representar:

Delegation
├── User
├── Agent
├── Purpose
├── Capabilities
├── Permissions
├── Scope
├── Duration
├── Conditions
├── Approval Rules
└── Revocation

Esto crea la base para una arquitectura donde:

Agents actúan en nombre del usuario, pero nunca se convierten en el usuario.

51.30 — User Goals

El usuario puede tener objetivos.

User
 ↓
Goals
 ├── Personal
 ├── Business
 ├── Operational
 ├── Learning
 └── Experience

Los objetivos pueden posteriormente alimentar:

AI;
recommendations;
workflows;
Agents;
personalization.
51.31 — User Intent

El Intent representa lo que el usuario quiere conseguir.

Por ejemplo:

Intent
├── Goal
├── Context
├── Constraints
├── Preferences
├── Permissions
└── Expected Outcome

La Foundation no ejecuta el intent.

Pero debe permitir representarlo.

Posteriormente User Core podrá convertirlo en comportamiento.

51.32 — User Metadata

El usuario puede tener metadata adicional.

Ejemplo:

Metadata
├── Tags
├── Attributes
├── Segments
├── Source
├── External IDs
├── Classification
└── Custom Data

La metadata debe estar gobernada y no utilizarse automáticamente como autorización.

51.33 — User Classification

EVOXA puede clasificar usuarios según diferentes dimensiones:

User
├── Type
├── Role
├── Segment
├── Organization
├── Tenant
├── Experience Level
└── Risk Level

Pero la clasificación no debe convertirse automáticamente en una decisión sobre el usuario sin reglas explícitas.

51.34 — User Experience Identity

La identidad del usuario también puede incluir características relacionadas con experiencia:

Experience Identity
├── Preferences
├── Accessibility
├── Interaction Style
├── Favorite Channels
├── Language
├── Personalization
└── Experience History

Esto será especialmente importante para el futuro Experience Layer.

51.35 — Accessibility

User Foundation debe contemplar accesibilidad como propiedad estructural.

Puede incluir:

visual preferences;
font preferences;
contrast;
motion preferences;
language;
input preferences;
assistive technologies.

La experiencia debe adaptarse al usuario sin modificar su identidad.

51.36 — User Context Graph

Los diferentes elementos pueden conectarse en un grafo.

                         USER
                          │
             ┌────────────┼────────────┐
             ▼            ▼            ▼
        ORGANIZATION    TENANT      PROFILE
             │            │            │
             └────────────┼────────────┘
                          ▼
                       CONTEXT
                          │
             ┌────────────┼────────────┐
             ▼            ▼            ▼
         SESSION       DEVICE       PREFERENCES
             │
             ▼
        APPLICATION
             │
        ┌────┴────┐
        ▼         ▼
       AI       AGENT

Este grafo permitirá contextualizar las interacciones.

51.37 — User Foundation Registry

EVOXA debe mantener un registro centralizado de entidades de usuario.

User Registry

Puede contener:

User ID
Identity
Profile
Organizations
Tenants
Roles
Permissions
Preferences
Privacy
Consent
Devices
Relationships
AI Profile
Agent Delegations
Metadata
Security State
Lifecycle State

No significa necesariamente que toda esta información deba almacenarse físicamente en una sola base de datos.

Significa que debe existir una representación coherente del usuario.

51.38 — User Digital Twin

En etapas avanzadas, EVOXA puede construir un modelo digital del usuario.

No debe interpretarse como una copia completa de la persona.

Es una representación estructurada y autorizada.

User Digital Representation
├── Identity
├── Context
├── Preferences
├── Goals
├── Relationships
├── Interactions
├── Permissions
├── Consent
├── AI Relationship
└── Agent Relationship

Esto permite personalización sin confundir:

modelo digital del usuario

con:

usuario real.

51.39 — Privacy-Aware User Model

El modelo digital debe ser:

Privacy-Aware by Design

Cada atributo puede tener:

Visibility
Access Policy
Purpose
Retention
Sensitivity
Consent Requirement

Por ejemplo:

User Data
    │
    ├── Public
    ├── Private
    ├── Restricted
    └── Sensitive
51.40 — User Foundation and AI

AI puede consumir información del usuario, pero debe hacerlo mediante contexto gobernado.

User
 ↓
User Foundation
 ↓
Context
 ↓
Permissions
 ↓
Policies
 ↓
AI

No:

AI
 ↓
All User Data

La Foundation proporciona el límite estructural.

51.41 — User Foundation and Agents

Los Agents también deben consumir User Context mediante interfaces controladas.

User
 ↓
Foundation
 ↓
Context
 ↓
Delegation
 ↓
Agent

Esto permite que un Agent conozca únicamente aquello que necesita para cumplir su propósito.

51.42 — User Identity Graph

EVOXA puede representar diferentes identidades asociadas a un mismo usuario.

Human User
    │
    ├── EVOXA Identity
    ├── External Identity
    ├── Organization Identity
    ├── Application Identity
    └── Device Identity

Esto será fundamental para federación de identidad e integración con sistemas externos.

51.43 — External Identity

Un usuario puede autenticarse mediante diferentes proveedores.

Conceptualmente:

External Identity
       ↓
Identity Mapping
       ↓
EVOXA User

La identidad externa no reemplaza necesariamente la identidad interna.

Se mantiene una relación:

External ID
    ↕
EVOXA User ID
51.44 — User Federation

EVOXA puede soportar usuarios federados.

Identity Provider
       ↓
Federation
       ↓
EVOXA Identity
       ↓
User Context

Esto permite integrar:

organizaciones;
aplicaciones;
proveedores de identidad;
ecosistemas externos.
51.45 — User Security Context

Toda interacción debería poder reconstruir:

Who?
Where?
When?
From What Device?
In Which Application?
For Which Tenant?
With Which Permissions?
Under Which Policies?

Esto forma parte del:

User Security Context
51.46 — User Trust

En sistemas avanzados puede existir un concepto de confianza.

User Trust
├── Identity Trust
├── Device Trust
├── Session Trust
├── Behavioral Risk
└── Context Trust

Esto no debe sustituir las políticas de seguridad.

Debe complementar el modelo.

51.47 — User Risk Context

El sistema puede representar:

Risk
├── Low
├── Medium
├── High
└── Critical

El riesgo puede depender de:

sesión;
dispositivo;
acción;
contexto;
comportamiento;
recurso;
sensibilidad.

La evaluación de riesgo podrá utilizarse posteriormente en User Core y Security Services.

51.48 — User Preferences vs Policies vs Permissions

Estas tres entidades deben mantenerse separadas.

Preference

Lo que el usuario prefiere.

Permission

Lo que el usuario puede hacer.

Policy

Lo que el sistema permite bajo determinadas condiciones.

Ejemplo:

Preference:
User prefers email.

Permission:
User can export own data.

Policy:
Export requires authentication.

Esta separación evita confusiones arquitectónicas.

51.49 — User Ownership

Debe existir claridad sobre quién controla cada elemento.

User
 ↓
Owns Data
 ↓
Grants Permission
 ↓
Delegates Action

Ownership, Permission y Delegation son conceptos diferentes.

51.50 — User Foundation Governance

La Foundation debe estar gobernada.

Debe existir:

ownership;
identity management;
privacy;
security;
data governance;
consent;
audit;
access policies;
lifecycle rules.
51.51 — User Foundation Observability

Aunque Foundation es principalmente estructural, debe existir observabilidad sobre cambios importantes.

Ejemplo:

Identity Created
Profile Updated
Role Changed
Permission Granted
Consent Granted
Consent Revoked
Device Registered
Delegation Created
Delegation Revoked

Estos eventos pueden alimentar Audit y Governance.

51.52 — User Foundation Events

Algunos eventos conceptuales:

UserCreated
UserVerified
UserUpdated
UserRoleChanged
UserPermissionGranted
UserPermissionRevoked
UserConsentGranted
UserConsentRevoked
UserDeviceRegistered
UserDeviceRevoked
UserOrganizationJoined
UserOrganizationLeft
UserDelegationCreated
UserDelegationRevoked

Estos eventos serán utilizados por las siguientes capas.

51.53 — Machine-Readable User Foundation

La representación debe poder ser entendida por máquinas.

Ejemplo conceptual:

user:
  id:
  identity:
  profile:
  organizations:
  tenants:
  roles:
  permissions:
  preferences:
  privacy:
  consent:
  devices:
  relationships:
  goals:
  ai:
  agents:
  security:
  lifecycle:

Esto permite que:

Applications;
AI;
Agents;
Platform;
Services

comprendan una representación común del usuario.

51.54 — User Foundation API Boundary

Foundation debe definir la frontera conceptual, no necesariamente todas las operaciones.

Ejemplos:

User Identity
User Profile
User Context
User Membership
User Permissions
User Preferences
User Privacy
User Consent
User Relationships
User Delegations

Las capas superiores podrán implementar servicios y capacidades sobre estas entidades.

51.55 — User Foundation Security Principles
Identity First

Toda interacción debe tener una identidad o contexto anónimo explícito.

Least Privilege

El usuario recibe solamente los permisos necesarios.

Explicit Consent

El consentimiento debe ser explícito cuando corresponda.

Contextual Access

El acceso depende del contexto.

Tenant Isolation

Los tenants deben mantenerse aislados.

Auditability

Los cambios importantes deben ser trazables.

Privacy by Design

La privacidad debe formar parte de la arquitectura desde el inicio.

AI Governance

AI no debe obtener acceso irrestricto al contexto del usuario.

Agent Delegation

Los Agents deben actuar mediante delegaciones explícitas.

51.56 — User Foundation Architecture

La arquitectura conceptual queda:

                         USER
                           │
                    USER FOUNDATION
                           │
       ┌───────────────────┼───────────────────┐
       │                   │                   │
       ▼                   ▼                   ▼
    IDENTITY            PROFILE             CONTEXT
       │                   │                   │
       └───────────────────┼───────────────────┘
                           ▼
                      MEMBERSHIP
                           │
                ┌──────────┼──────────┐
                ▼          ▼          ▼
             TENANT     ORGANIZATION  ROLES
                │          │          │
                └──────────┼──────────┘
                           ▼
                      PERMISSIONS
                           │
          ┌────────────────┼────────────────┐
          ▼                ▼                ▼
     PREFERENCES        PRIVACY          CONSENT
          │                │                │
          └────────────────┼────────────────┘
                           ▼
                      RELATIONSHIPS
                           │
             ┌─────────────┼─────────────┐
             ▼             ▼             ▼
          DEVICES       APPLICATIONS      AI
                                           │
                                           ▼
                                         AGENTS
51.57 — Relación con las capas siguientes

La responsabilidad de cada capa queda claramente delimitada:

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
51.58 — User Foundation Maturity

La evolución puede definirse como:

Level 1 — Basic Identity

Usuario identificado.

Level 2 — Structured User

Identity + Profile + Context.

Level 3 — Governed User

Roles, permissions, privacy y governance.

Level 4 — Contextual User

Tenant, organization, application y session context.

Level 5 — Personalized User

Preferences, goals y experience context.

Level 6 — AI-Aware User

Relación estructurada con AI.

Level 7 — Agent-Ready User

Delegation, agent permissions y controlled autonomy.

Level 8 — Intelligent User Model

Contextual intelligence y personalization avanzada.

Level 9 — Autonomous Interaction

Agents pueden actuar bajo delegaciones y políticas.

Level 10 — Adaptive User Experience

La experiencia evoluciona dinámicamente según contexto, preferencias y objetivos autorizados.

51.59 — Principios fundamentales

El User Foundation de EVOXA debe seguir estos principios:

User is a First-Class Entity
Identity Must Be Stable
Profile Must Be Separate from Identity
Context Must Be Explicit
Membership Must Be Governed
Permissions Must Be Granular
Preferences Must Be Explicit
Privacy Must Be Built-In
Consent Must Be Traceable
Ownership Must Be Explicit
AI Access Must Be Governed
Agent Delegation Must Be Explicit
Tenant Boundaries Must Be Enforced
User Relationships Must Be Modelled
User Data Must Be Context-Aware
Identity Must Be Observable
User Representation Must Be Machine-Readable
User Experience Must Remain User-Controlled
51.60 — Definición final

User Foundation es la capa fundamental de EVOXA que define la identidad, perfil, contexto, pertenencia, relaciones, roles, permisos, preferencias, privacidad, consentimiento, ownership, seguridad, dispositivos, relación con Applications, AI y Agents, estableciendo la representación estructural y las reglas fundamentales mediante las cuales un usuario existe e interactúa dentro del ecosistema.

51.61 — Visión final

La evolución de User Foundation puede resumirse:

Identity
   ↓
Structured User
   ↓
Contextual User
   ↓
Governed User
   ↓
Personalized User
   ↓
AI-Aware User
   ↓
Agent-Ready User
   ↓
Intelligent User
   ↓
Autonomous Interaction
   ↓
Adaptive Experience

Y conceptualmente:

                         USER
                           │
                     IDENTITY
                           │
                       PROFILE
                           │
                       CONTEXT
                           │
                    ORGANIZATION
                           │
                        TENANT
                           │
                    PERMISSIONS
                           │
                    PREFERENCES
                           │
                       PRIVACY
                           │
                       CONSENT
                           │
                    RELATIONSHIPS
                           │
                    APPLICATIONS
                           │
                          AI
                           │
                        AGENTS
                           │
                  INTELLIGENT USER
                           │
                ADAPTIVE EXPERIENCE

Con 51 — User Foundation, establecemos la base conceptual del User & Experience Layer.

El siguiente nivel natural será:

52 — User Core

donde la identidad y el contexto definidos aquí comenzarán a transformarse en estado, intención, decisiones, interacción, personalización, coordinación y comportamiento del usuario dentro de EVOXA.
