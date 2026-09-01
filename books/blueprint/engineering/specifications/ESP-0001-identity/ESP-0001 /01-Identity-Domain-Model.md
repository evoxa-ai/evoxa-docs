ESP-0001 / 01 — Identity Domain Model

Document ID: ESP-0001/01
Module: Identity
Version: 1.0.0
Status: Draft
Parent: ESP-0001 — Identity
Architecture Parent: BP-0002 — Reference Architecture
Domain: Identity
Target Repository: evoxa-platform
Target Path: apps/api/app/domains/identity/

1. Purpose

Este documento define el modelo de dominio de Identity de EVOXA.

Identity constituye la capacidad transversal responsable de establecer:

quién es un usuario;
cómo se autentica;
qué identidad posee;
a qué organización pertenece;
qué roles y permisos posee;
qué contexto de tenant/organización está utilizando;
qué sesiones y credenciales están activas;
y cómo se registra y controla el acceso.

El dominio Identity debe proporcionar estas capacidades sin incorporar lógica perteneciente a otros dominios de negocio.

El Blueprint establece que Identity es uno de los dominios centrales de EVOXA y que cada dominio debe mantener límites claros y responsabilidad propia.

2. Architectural Context

El modelo se sitúa dentro de las capas arquitectónicas:

Presentation
     ↓
API
     ↓
Application
     ↓
Domain
     ↓
Infrastructure

La implementación de Identity deberá ubicarse en:

apps/api/app/domains/identity/

Esta ubicación está definida en el mapa arquitectónico para el dominio Identity y asociada a ESP-0001.

3. Identity Responsibilities

Identity es responsable de:

Identity
├── Authentication
├── Authorization
├── User Identity
├── Organization Identity
├── Tenant Context
├── Role Management
├── Permission Management
├── Session Management
├── API Tokens
├── MFA
├── Security Policies
└── Identity Audit

Las capacidades de Authentication, Authorization, Role Management, Permission Management, Tenant Management, Session Management, API Tokens y MFA aparecen expresamente dentro de las capacidades de plataforma de EVOXA.

4. Non-Responsibilities

Identity NO debe implementar lógica propia de:

Training
Nutrition
Recovery
Human Digital Twin
Recommendations
AI
Analytics
Billing
Community
Marketplace

Identity solamente proporciona identidad, autenticación, autorización y contexto de seguridad para que esos dominios puedan operar.

Por ejemplo:

Training
   ↓
"¿Este usuario puede modificar este programa?"
   ↓
Identity Authorization
   ↓
ALLOW / DENY

Identity no decide qué constituye un buen programa de entrenamiento.

5. Domain Model
5.1 Core Concepts

El modelo conceptual inicial de Identity contiene:

User
 │
 ├── Identity Credentials
 │
 ├── Sessions
 │
 └── Memberships
          │
          ├── Organization
          │
          └── Roles
                   │
                   └── Permissions

Conceptualmente:

                   ┌──────────────┐
                   │     User     │
                   └──────┬───────┘
                          │
                     Membership
                          │
             ┌────────────┴────────────┐
             │                         │
      Organization                  Role
                                       │
                                  Permissions
Importante

La utilización de Membership como entidad explícita es una decisión de diseño derivada, no una afirmación literal del Blueprint.

Se propone porque EVOXA es multi-tenant y una identidad puede necesitar distintos contextos organizacionales.

Esta decisión debe convertirse posteriormente en un ADR si la adoptamos como arquitectura definitiva.

6. User
Purpose

Representa la identidad principal de una persona que utiliza EVOXA.

Un User no representa necesariamente un rol profesional concreto.

Por ejemplo:

User
 ├── Identity
 ├── Credentials
 ├── Status
 └── Memberships

El usuario puede posteriormente participar como:

Coach
Nutritionist
Physiotherapist
Team Manager
Administrator

La arquitectura de API identifica esos roles profesionales dentro del modelo de autorización.

Proposed Attributes
Attribute	Type	Purpose
id	UUID	Identificador único
email	Email	Identidad de acceso
status	Enum	Estado de la cuenta
created_at	DateTime	Creación
updated_at	DateTime	Última modificación
version	Integer	Control de versión
last_authenticated_at	DateTime	Última autenticación
metadata	JSON	Metadatos controlados

Nota: Estos atributos son una propuesta de implementación. El Blueprint establece responsabilidades y principios, pero no fija todavía este esquema físico exacto.

7. User Lifecycle

Propuesta inicial:

                ┌──────────┐
                │  PENDING │
                └────┬─────┘
                     │ activate
                     ▼
                ┌──────────┐
                │  ACTIVE  │
                └────┬─────┘
                     │ deactivate
                     ▼
               ┌───────────┐
               │ SUSPENDED │
               └─────┬─────┘
                     │ reactivate
                     ▼
                ┌──────────┐
                │  ACTIVE  │
                └──────────┘

Estados y transiciones concretas deberán validarse con las reglas de Identity antes de implementarse.

8. Organization
Purpose

Representa un contexto organizacional dentro de EVOXA.

Ejemplos conceptuales:

Professional Practice
Fitness Center
Nutrition Clinic
Enterprise
Sports Organization

La organización es un límite fundamental para el aislamiento de datos.

9. Organization Attributes

Propuesta:

Attribute	Type
id	UUID
name	String
status	Enum
created_at	DateTime
updated_at	DateTime
version	Integer

Nuevamente, este esquema es Derived Design y debe considerarse candidato para ADR.

10. Membership
Purpose

Membership representa la relación:

User ↔ Organization

Ejemplo:

Sebastian
   │
   ├── Organization A
   │       └── Coach
   │
   └── Organization B
           └── Team Manager

Esto permite que la autorización sea evaluada dentro de un contexto organizacional.

11. Membership Attributes

Propuesta:

Attribute	Type
id	UUID
user_id	UUID
organization_id	UUID
status	Enum
joined_at	DateTime
created_at	DateTime
updated_at	DateTime
12. Role

Un Role representa una agrupación lógica de permisos.

Ejemplo:

Coach
   ├── clients.read
   ├── clients.update
   ├── training.read
   └── training.create

La arquitectura de seguridad establece RBAC como uno de los mecanismos principales de autorización. También contempla ABAC para políticas más contextuales.

13. Permission

Una Permission representa una capacidad autorizable.

Propuesta:

resource + action

Ejemplos:

clients.read
clients.create
clients.update
clients.delete

training.read
training.create
training.update

nutrition.read
nutrition.create

Esto permite:

Role
  ↓
RolePermission
  ↓
Permission
14. RolePermission

Entidad de relación:

Role
 │
 └──── RolePermission ──── Permission

Propuesta:

Attribute	Type
role_id	UUID
permission_id	UUID

Debe existir una restricción de unicidad:

(role_id, permission_id)

para evitar asignaciones duplicadas.

15. Session

Identity debe proporcionar Session Management, definido como capability de plataforma.

Una sesión representa un contexto autenticado activo.

Conceptualmente:

User
 │
 ├── Session
 ├── Session
 └── Session

Una misma identidad puede tener sesiones simultáneas en:

Mobile
Web
Coach Portal
Admin Portal
16. Session Attributes

Propuesta:

Attribute	Type
id	UUID
user_id	UUID
organization_id	UUID / nullable
created_at	DateTime
expires_at	DateTime
revoked_at	DateTime / nullable
last_activity_at	DateTime
ip_address	String / protected
user_agent	String / protected

La estructura exacta debe validarse posteriormente con Security y Privacy specifications.

17. Value Objects

Identity debería utilizar Value Objects para conceptos que requieren validación propia.

Propuesta:

UserId
OrganizationId
MembershipId
RoleId
PermissionId
SessionId
Email

Ejemplo:

Email
 ├── normalized value
 ├── validation
 └── equality

En lugar de distribuir reglas de email por todo el sistema:

if email...

el dominio debería centralizarlas en:

Email
18. Aggregates

Propuesta inicial:

User Aggregate
Organization Aggregate
Role Aggregate
Session Aggregate

Pero esto todavía no debe considerarse definitivo.

El Blueprint establece el uso de DDD y agregados como parte del modelo de dominio, pero la definición exacta de los agregados de Identity debe quedar establecida en la ESP.

19. Domain Invariants

Las invariantes que se derivan del modelo son:

INV-001

Un User debe tener un identificador único.

INV-002

Un Permission debe ser identificable de forma única.

INV-003

Una combinación:

Role + Permission

no puede estar duplicada.

INV-004

Una sesión revocada no puede volver a utilizarse como sesión válida.

INV-005

Una solicitud autenticada debe disponer de un contexto de identidad válido.

INV-006

El acceso entre tenants/organizaciones debe estar restringido.

La arquitectura de seguridad establece explícitamente aislamiento de tenants, pertenencia organizacional, ownership y restricciones cross-tenant.

20. Authorization Model

EVOXA utilizará conceptualmente:

Authentication
      ↓
Who are you?
      ↓
Identity
      ↓
Organization Context
      ↓
Role
      ↓
Permissions
      ↓
Policy / Ownership / Context
      ↓
ALLOW / DENY

Esto permite combinar:

RBAC
Coach
   → training.create

con:

ABAC
Coach
   AND
belongs_to_organization
   AND
owns_client
   AND
client_is_active

La arquitectura establece RBAC + ABAC como mecanismos complementarios.

21. Tenant Isolation

Todo acceso a información perteneciente a una organización debe establecer un contexto de tenant.

Conceptualmente:

Request
   ↓
Authentication
   ↓
User
   ↓
Tenant Context
   ↓
Authorization
   ↓
Resource Access

No debemos permitir:

User A / Tenant A
        ↓
   Resource Tenant B
        ↓
       DENY

Esto es una condición fundamental de seguridad de EVOXA.

22. Domain Events

Identity debe generar eventos cuando ocurren hechos relevantes del dominio.

Propuesta inicial:

UserCreated
UserUpdated
UserActivated
UserDeactivated

OrganizationCreated

MembershipCreated
MembershipRemoved

RoleCreated
RoleUpdated

PermissionAssigned
PermissionRevoked

UserAuthenticated
UserLoggedOut

SessionCreated
SessionRevoked

Estos nombres son Derived Design.

El principio de eventos sí está establecido por la arquitectura: las acciones empresariales significativas generan eventos de dominio inmutables y los consumidores deben ser idempotentes.

23. Event Structure

Los eventos deberán seguir el envelope definido por Event Architecture:

{
  "event_id": "...",
  "event_type": "UserCreated",
  "version": 1,
  "timestamp": "...",
  "tenant_id": "...",
  "actor": "...",
  "resource": "...",
  "payload": {},
  "metadata": {
    "correlation_id": "...",
    "causation_id": "..."
  }
}

La arquitectura de eventos establece precisamente estos conceptos de envelope, versionado, tenant, actor, correlation y causation.

24. Repository Interfaces

El dominio no debería depender directamente de PostgreSQL.

Propuesta:

UserRepository
OrganizationRepository
MembershipRepository
RoleRepository
PermissionRepository
SessionRepository

El dominio conoce interfaces:

UserRepository
      ↑
      │
PostgreSQLUserRepository

De esta forma:

Domain
  ↓
Repository Interface
  ↓
Infrastructure
  ↓
PostgreSQL

Esto respeta la separación de capas establecida por la arquitectura.

25. Proposed Domain Structure

La primera estructura concreta sería:

apps/api/app/domains/identity/

├── api/
│   ├── routes/
│   ├── schemas/
│   └── controllers/
│
├── application/
│   ├── commands/
│   ├── queries/
│   ├── services/
│   └── handlers/
│
├── domain/
│   ├── entities/
│   │   ├── user.py
│   │   ├── organization.py
│   │   ├── membership.py
│   │   ├── role.py
│   │   ├── permission.py
│   │   └── session.py
│   │
│   ├── value_objects/
│   ├── aggregates/
│   ├── services/
│   ├── policies/
│   ├── events/
│   └── repositories/
│
└── infrastructure/
    ├── persistence/
    ├── security/
    ├── authentication/
    └── messaging/

Nota: La estructura anterior es una propuesta de implementación derivada. El Blueprint solamente fija la separación arquitectónica y la ubicación del dominio, no esta estructura exacta de archivos.

26. Security Requirements

Identity deberá cumplir como mínimo con los principios:

Zero Trust
Least Privilege
Defense in Depth
Secure by Default
Privacy by Design
Continuous Verification
Auditability

Estos principios están establecidos por la arquitectura de seguridad de EVOXA.

La arquitectura también especifica:

Passwords
→ Argon2id

Transport
→ TLS 1.3

Encryption at rest
→ AES-256

Token / signature mechanisms
→ Ed25519 / RSA-4096

Estas decisiones pertenecen a Security Architecture y deberán ser respetadas por Identity.

27. Audit

Las operaciones críticas de Identity deben ser auditables.

Ejemplos:

Login
Logout
Failed authentication
Password change
MFA change
Role assignment
Permission assignment
Membership change
Account activation
Account suspension
Session revocation

El sistema debe permitir determinar:

WHO
WHAT
WHEN
WHERE
WHY / CONTEXT
RESULT

La arquitectura exige auditabilidad de operaciones críticas y contempla retención prolongada de audit logs.

28. Observability

Identity deberá integrarse con la plataforma de observabilidad.

Cada operación relevante deberá poder correlacionarse mediante:

trace_id
span_id
correlation_id

Especialmente en:

Client
 ↓
API
 ↓
Identity
 ↓
Database
 ↓
Event Platform

La arquitectura establece tracing distribuido a través de APIs, AI, Event Bus, bases de datos e integraciones externas.

29. Testing Requirements

Identity deberá tener como mínimo:

Unit Tests
Integration Tests
API Tests
Security Tests
Authorization Tests
Tenant Isolation Tests
Session Tests
Event Tests

La arquitectura establece una estrategia de pruebas multicapa y exige validación de autenticación, autorización, validación, errores e idempotencia en APIs.

30. Traceability
BP-0001
   ↓
BP-0002
   ↓
EES
   ↓
ESP-0001 Identity
   ↓
ESP-0001/01 Identity Domain Model
   ↓
Implementation Stories
   ↓
Technical Tasks
   ↓
Source Code
   ↓
Tests

Esto mantiene el principio de trazabilidad definido en ARCHITECTURE-MAP.md.

31. ADR Candidates

Antes de cerrar esta especificación, identifico estas decisiones que no deberíamos asumir silenciosamente:

ADR-IDENTITY-001

¿Membership será una entidad/agregado formal del dominio?

ADR-IDENTITY-002

¿Un usuario puede pertenecer a múltiples organizaciones?

ADR-IDENTITY-003

¿Los roles son globales, por organización o ambos?

ADR-IDENTITY-004

¿Las permissions serán exclusivamente:

resource.action

o tendrán una estructura más rica?

ADR-IDENTITY-005

¿Identity utilizará JWT como mecanismo principal de access token?

ADR-IDENTITY-006

¿Dónde se almacenarán los refresh tokens?

ADR-IDENTITY-007

¿Cuál será el proveedor principal de Identity?

EVOXA Identity
Entra ID
Google
Okta
Auth0
SAML/OIDC

La arquitectura contempla EVOXA Identity y proveedores externos mediante OIDC/SAML, pero todavía debemos determinar la estrategia definitiva de implementación.

32. Implementation Readiness

Estado actual: 🟡 Draft

Tenemos definido el modelo conceptual suficiente para continuar con:

ESP-0001 / 02
Identity Architecture

pero todavía no recomiendo escribir código.

Primero deberíamos cerrar las decisiones arquitectónicas anteriores y después convertirlas en una especificación definitiva.

Estado de ESP-0001/01
Área	Estado
Domain purpose	✅
Responsibilities	✅
Non-responsibilities	✅
User	🟡 Propuesta
Organization	🟡 Propuesta
Membership	🟡 ADR necesario
Role	🟡
Permission	🟡
Session	🟡
Aggregates	🟡
Invariants	🟡
Tenant isolation	✅ Principio
RBAC	✅
ABAC	✅
Events	🟡 Nombres propuestos
Persistence	🟡
Security	✅ Principios
Audit	✅
Observability	✅
Testing	✅
Implementation	⛔ No comenzar todavía
Lo importante

No subiría todavía este documento a GitHub como v1.0.0 definitivo. Lo subiría como:

ESP-0001/01 — Identity Domain Model
Status: Draft

y cuando revisemos los ADR y cerremos las decisiones, lo marcamos como Approved.

El siguiente paso lógico es ESP-0001 / 02 — Identity Architecture, donde vamos a transformar este modelo conceptual en la arquitectura técnica del dominio, incluyendo capas, componentes, flujos de autenticación/autorización, JWT/OIDC, sesiones, PostgreSQL, Redis, Event Platform y relación con API Gateway.
