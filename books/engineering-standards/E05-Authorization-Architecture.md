E05 — EVOXA Authorization Architecture
Engineering Specification

Pertenece a: Engineering Specification
Volumen: Engineering
Precede: E04 — Authentication Architecture
Siguiente: E06 — EVOXA Policy Architecture

1. Propósito

E05 define la arquitectura de Authorization de EVOXA.

Si E04 responde:

¿Quién eres?

E05 responde:

¿Qué puedes hacer, sobre qué recursos y bajo qué condiciones?

La autorización debe controlar el acceso a:

Resources
Actions
APIs
Modules
Domains
Tenants
Data
Operations
Administrative Functions
AI/Agents
Billing
Integrations
2. Principio Fundamental

La arquitectura debe separar claramente:

Authentication
      │
      ▼
Identity
      │
      ▼
Tenant Context
      │
      ▼
Authorization
      │
      ▼
Resource Access

Nunca debe asumirse:

Authenticated = Authorized

Un usuario autenticado solamente demuestra su identidad.

3. Objetivos

Authorization debe proporcionar:

RBAC
Permissions
Roles
Scopes
Policies
Tenant Authorization
Resource Authorization
Action Authorization
Contextual Authorization
Delegation
Service Authorization
Agent Authorization
Administrative Authorization
Policy Enforcement
Auditability
4. Authorization Architecture
                         AUTHORIZATION
                              │
          ┌───────────────────┼───────────────────┐
          │                   │                   │
         RBAC              Policies             Scopes
          │                   │                   │
          └───────────────────┼───────────────────┘
                              │
                       Authorization
                          Decision
                              │
                 ┌────────────┴────────────┐
                 │                         │
              ALLOW                      DENY
                 │                         │
                 ▼                         ▼
             Resource                   Audit
               Access                   Event
5. Authorization Model

La decisión de autorización debe considerar:

Subject
Action
Resource
Context
Policy

Formalmente:

AuthorizationDecision =
    Evaluate(
        Subject,
        Action,
        Resource,
        Context,
        Policies
    )
6. Subject

El Subject representa quién intenta ejecutar una acción.

Puede ser:

User
Service
Agent
Application
API Client
System Identity

Ejemplo:

Subject
 ├── userId
 ├── tenantId
 ├── roles
 ├── permissions
 ├── scopes
 └── authenticationContext
7. Resource

Un recurso puede ser:

User
Workout
Exercise
Nutrition Plan
Food
Meal
Trainer
Client
Tenant
Subscription
Invoice
Report
Dashboard
AI Agent
Integration
API Resource

Cada recurso debe poseer un identificador estable.

8. Action

Las acciones deben ser explícitas.

Ejemplos:

create
read
update
delete
list
execute
approve
publish
assign
export
manage
configure

No depender solamente de HTTP verbs.

Por ejemplo:

POST /reports/{id}/publish

representa:

action = report.publish

y no simplemente:

action = create
9. Permission

Una Permission representa una capacidad autorizable.

Formato recomendado:

resource:action

Ejemplos:

users:read
users:create
users:update
users:delete

workouts:read
workouts:create
workouts:update

nutrition:read
nutrition:create
nutrition:update

reports:export
reports:publish
10. Permission Naming

Las permissions deben seguir una convención estable:

<domain>.<resource>.<action>

o:

<resource>:<action>

Para EVOXA se recomienda el formato jerárquico:

training.workouts.read
training.workouts.create
nutrition.meals.read
nutrition.meals.create

Esto permite organizar permissions por dominio.

11. Permission Hierarchy
training
 ├── workouts
 │    ├── read
 │    ├── create
 │    ├── update
 │    └── delete
 │
 └── exercises
      ├── read
      ├── create
      └── update
12. Role

Un Role agrupa permissions.

Role
 ├── name
 ├── description
 ├── permissions
 └── scope

Ejemplo:

Trainer
 ├── training.workouts.read
 ├── training.workouts.create
 ├── training.workouts.update
 └── nutrition.plans.read
13. RBAC

EVOXA utilizará Role-Based Access Control como mecanismo fundamental.

User
 ↓
Role
 ↓
Permissions
 ↓
Resources

Ejemplo:

Trainer
    ↓
training.workouts.*
    ↓
Workout Resources
14. Role Types

Los roles pueden clasificarse:

Platform Roles
Tenant Roles
Domain Roles
System Roles
Service Roles
Agent Roles
15. Platform Roles

Aplican al nivel global de EVOXA.

Ejemplos:

PlatformAdmin
PlatformSupport
PlatformSecurityAdmin
PlatformBillingAdmin

Estos roles deben estar estrictamente controlados.

16. Tenant Roles

Aplican dentro de un tenant.

Ejemplos:

TenantOwner
TenantAdmin
TenantManager
Trainer
Coach
Member
Viewer
17. Domain Roles

Pueden limitarse a determinados dominios.

Ejemplo:

NutritionManager
TrainingManager
BillingManager
AnalyticsManager
18. Role Scope

Un rol debe tener un ámbito.

GLOBAL
TENANT
DOMAIN
RESOURCE

Ejemplo:

PlatformAdmin
    scope = GLOBAL

Trainer
    scope = TENANT
19. Role Assignment

Una asignación debe representar:

User
+
Role
+
Scope

Ejemplo:

User A
 ↓
Trainer
 ↓
Tenant X

El mismo usuario podría tener:

Trainer
Tenant X

TenantAdmin
Tenant Y
20. Tenant Authorization

Toda operación tenant-aware debe validar:

User
 ↓
Tenant Membership
 ↓
Tenant Role
 ↓
Permission
 ↓
Resource

No basta con incluir tenantId en el request.

21. Tenant Isolation

Una regla fundamental:

Un usuario nunca puede acceder a recursos de otro tenant simplemente modificando un identificador.

Ejemplo prohibido:

GET /clients/client-123

cuando:

client-123 → Tenant B
user → Tenant A

Debe producir:

403 Forbidden

o una respuesta equivalente según la política de no divulgación de existencia del recurso.

22. Resource Ownership

Algunos recursos pertenecen directamente a un usuario.

Ejemplo:

User
 └── PersonalWorkout

La autorización puede utilizar:

ownerId == subject.userId

Ejemplo conceptual:

member.workouts.read
AND
workout.ownerId == currentUser.id
23. Resource-Level Authorization

RBAC no siempre es suficiente.

Ejemplo:

Trainer

puede tener:

clients.read

pero solamente para clientes asignados a ese entrenador.

Por lo tanto:

Permission
+
Resource Relationship

debe formar parte de la decisión.

24. Relationship-Based Authorization

EVOXA puede utilizar relaciones:

Trainer
   ↓ coaches
Client

o:

User
   ↓ owns
Workout

o:

Tenant
   ↓ owns
Subscription
25. Authorization Decision

Ejemplo:

Subject:
Trainer A

Action:
clients.read

Resource:
Client B

Context:
Tenant X

Evaluación:

Trainer A
   │
   ├── has permission? YES
   ├── same tenant? YES
   ├── assigned to client? YES
   │
   └── ALLOW
26. Contextual Authorization

La autorización puede depender de contexto:

Tenant
Time
Location
Device
Authentication Level
Risk
Network
Resource State
Business Rules

Ejemplo:

billing.refund

requiere:

BillingAdmin
+
MFA
+
Refund policy
27. ABAC

EVOXA debe permitir evolución hacia Attribute-Based Access Control.

Subject Attributes
+
Resource Attributes
+
Environment Attributes
+
Policy

Ejemplo:

subject.role = trainer
resource.type = workout
resource.tenant = subject.tenant
resource.status = draft
28. RBAC + ABAC

La arquitectura recomendada:

RBAC
  +
Resource Rules
  +
Context
  +
Policy

No intentar resolver todo exclusivamente mediante roles.

29. Scopes

Scopes representan permisos delegados a un token o cliente.

Ejemplo:

training:read
training:write
nutrition:read
nutrition:write

Un Access Token podría contener:

{
  "scope": [
    "training:read",
    "nutrition:read"
  ]
}
30. Scope vs Permission

La distinción:

Permission
→ capability granted to identity

Scope
→ capability delegated to credential/token

Ejemplo:

User
 └── Permission: nutrition.write

OAuth Token
 └── Scope: nutrition:read

El token no debería poder exceder los permisos autorizados al subject.

31. Effective Permissions

Las permissions efectivas resultan de:

User
+
Roles
+
Tenant Membership
+
Scopes
+
Policies
+
Resource Rules

Conceptualmente:

EffectiveAccess =
    RolePermissions
    ∩
    TokenScopes
    ∩
    PolicyConstraints
    ∩
    ResourceConstraints
32. Authorization Decision Engine

EVOXA debe tener un componente central de decisión:

Authorization Engine

Responsabilidades:

Evaluate Permission
Evaluate Role
Evaluate Scope
Evaluate Policy
Evaluate Resource Relationship
Return Decision
33. PDP / PEP Architecture

Se recomienda separar:

PDP

Policy Decision Point

Decide:

ALLOW
DENY
PEP

Policy Enforcement Point

Aplica la decisión.

Arquitectura:

Request
  ↓
PEP
  ↓
PDP
  ↓
Decision
  ↓
PEP
  ↓
Resource
34. Policy Enforcement

El enforcement puede existir en:

API Gateway
Application Service
Domain Service
Repository
Resource Layer

Pero la autorización de negocio crítica debe ejecutarse lo más cerca posible del recurso protegido.

35. API Gateway Authorization

Gateway puede realizar controles generales:

Token Valid
Scope
Rate Limit
Basic Permission

Pero no debe ser el único mecanismo de autorización.

36. Application Authorization

Los servicios deben validar permisos específicos.

Ejemplo:

WorkoutService
    ↓
authorize(
    subject,
    "training.workouts.update",
    workout
)
37. Domain Authorization

Las reglas críticas pueden pertenecer al dominio.

Ejemplo:

A trainer can modify a workout
only if the workout belongs
to one of the trainer's assigned clients.

Esto no debe depender exclusivamente de middleware HTTP.

38. Repository Security

El acceso a datos debe reforzarse con:

Tenant Filters
Ownership Filters
Row-Level Security where appropriate

Nunca confiar únicamente en filtros provenientes del cliente.

39. PostgreSQL Row-Level Security

Para determinados recursos críticos puede utilizarse:

PostgreSQL RLS

Arquitectura:

Application Authorization
        +
Database Row-Level Security

Esto proporciona defensa en profundidad.

40. Authorization Middleware

Pipeline:

Request
 ↓
Authentication
 ↓
Tenant Context
 ↓
Authorization Middleware
 ↓
Controller
 ↓
Application Service
 ↓
Domain Authorization
 ↓
Repository
41. Authorization Decorators

El backend puede soportar una abstracción como:

@RequirePermission("training.workouts.update")

o:

@RequireRole("Trainer")

Pero decorators deben ser una comodidad de enforcement, no la única fuente de reglas.

42. Permission Checks

Los checks deben ser explícitos:

authorization.can(
  subject,
  "training.workouts.update",
  workout
)

Resultado:

ALLOW
DENY
43. Deny by Default

Principio obligatorio:

No explicit permission
        ↓
      DENY

Nunca:

No rule
 ↓
ALLOW
44. Least Privilege

Cada identity debe recibir solamente las permissions necesarias.

Minimal Role
+
Minimal Scope
+
Minimal Resource Access
45. Separation of Duties

Algunas operaciones requieren separar responsabilidades.

Ejemplo:

Billing Operator

puede:

invoice.read
invoice.create

pero no necesariamente:

invoice.approve
46. Administrative Authorization

Las funciones administrativas deben tener una capa específica:

Platform Administration
        │
        ▼
Administrative Permissions
        │
        ▼
Admin Resources

No utilizar:

isAdmin = true

como mecanismo completo.

47. Super Administrator

Si EVOXA implementa un rol equivalente a:

SuperAdmin

debe ser:

Rare
Restricted
Audited
MFA-protected
Highly privileged
48. Break-Glass Access

Para emergencias:

Break Glass

debe permitir acceso temporal altamente controlado.

Características:

Explicit activation
Strong authentication
Reason required
Time limit
Audit
Automatic expiration
49. Delegated Authorization

EVOXA puede permitir delegación:

User A
 ↓ delegates
User B
 ↓
Specific Resource
 ↓
Specific Action

Ejemplo:

Trainer
 ↓
Assistant
 ↓
nutrition.plan.read
50. Delegation Constraints

Una delegación debe limitar:

Who
What
Resource
Tenant
Duration
Scope

Nunca:

Delegate everything

por defecto.

51. Temporary Permissions

Una permission puede tener expiración:

Permission
 ├── startAt
 └── expiresAt

Ejemplo:

TemporarySupportAccess
expiresAt = ...
52. Authorization for Agents

Los agentes EVOXA deben tener permisos explícitos.

Agent
 ↓
Agent Role
 ↓
Agent Permissions
 ↓
Tool Access

Un agente no debe heredar automáticamente todos los permisos del usuario que lo invocó.

53. User-to-Agent Delegation

Cuando un usuario solicita una acción mediante un agente:

User
 ↓
Agent
 ↓
Delegated Context
 ↓
Authorization
 ↓
Tool

El agente debe operar dentro de los límites delegados.

54. Agent Permission Boundary

Ejemplo:

User:
training.workouts.write

Agent:
training.workouts.read

Resultado:

Agent cannot write.

La delegación debe ser:

min(user_permissions, agent_permissions)

más las políticas aplicables.

55. Service Authorization

Los servicios deben tener identidades propias.

Service A
 ↓
Service Identity
 ↓
Service Role
 ↓
Permission
 ↓
Service B
56. Service-to-Service Permissions

Ejemplo:

nutrition-service

puede:

nutrition.foods.read

pero no:

billing.invoices.delete
57. Authorization for Integrations

Una integración externa debe tener:

Integration Identity
+
Scopes
+
Tenant Binding
+
Permissions
58. API Client Authorization

Los clientes externos pueden recibir:

Client
 ↓
Scopes
 ↓
Tenant
 ↓
API Resources

Los scopes deben ser explícitos.

59. Authorization Policy

Una Policy representa una regla declarativa.

Ejemplo conceptual:

ALLOW
IF
subject.role == Trainer
AND
resource.tenantId == subject.tenantId
AND
resource.trainerId == subject.userId
60. Policy Engine

La arquitectura debe permitir eventualmente integrar un motor especializado de políticas.

Conceptualmente:

Application
    ↓
Authorization Service
    ↓
Policy Engine
    ↓
Decision

La implementación concreta puede evolucionar según escala y complejidad.

61. Policy Versioning

Las políticas deben poder versionarse:

Policy v1
Policy v2
Policy v3

Esto permite:

Audit
Rollback
Testing
Controlled Deployment
62. Policy Evaluation Context

Ejemplo:

{
  "subject": {
    "id": "user_123",
    "tenantId": "tenant_1",
    "roles": ["trainer"]
  },
  "action": "training.workouts.update",
  "resource": {
    "id": "workout_1",
    "tenantId": "tenant_1",
    "trainerId": "user_123"
  }
}
63. Authorization Response

Internamente:

{
  "decision": "ALLOW",
  "policy": "training.workouts.update",
  "reason": "trainer_owns_resource"
}

Los detalles internos de las políticas no necesariamente deben exponerse al cliente.

64. Deny Reasons

Internamente pueden existir:

NOT_AUTHENTICATED
MISSING_PERMISSION
INVALID_SCOPE
TENANT_MISMATCH
RESOURCE_NOT_FOUND
RESOURCE_NOT_OWNED
POLICY_DENIED
MFA_REQUIRED
SESSION_INVALID
65. HTTP Responses

Generalmente:

401 Unauthorized

cuando falta una autenticación válida.

403 Forbidden

cuando existe identidad pero la autorización falla.

La estrategia exacta puede ajustarse para evitar información que permita enumeración de recursos.

66. Authorization Audit

Registrar:

subject
tenant
action
resource
decision
policy
timestamp
requestId

No registrar secretos.

67. Authorization Events

Ejemplos:

AuthorizationGranted
AuthorizationDenied
RoleAssigned
RoleRemoved
PermissionGranted
PermissionRevoked
PolicyChanged
DelegationCreated
DelegationRevoked
68. Privileged Actions

Las acciones administrativas deben generar eventos de mayor criticidad.

Ejemplo:

PlatformAdmin
 ↓
Tenant deletion
 ↓
Security Audit
69. Permission Management

Debe existir una fuente central de verdad:

Permission Registry

Ejemplo:

training.workouts.read
training.workouts.create
training.workouts.update
training.workouts.delete
70. Permission Registry

Cada permission puede definir:

id
name
domain
resource
action
description
systemDefined
sensitive
71. System Permissions

Algunas permissions deben ser propias del sistema:

SYSTEM_DEFINED = true

No deberían eliminarse arbitrariamente desde el tenant.

72. Custom Roles

Los tenants pueden crear roles personalizados:

Custom Role
 ↓
Tenant
 ↓
Selected Permissions

Ejemplo:

Nutrition Assistant
 ├── nutrition.meals.read
 ├── nutrition.meals.create
 └── nutrition.meals.update
73. Custom Permissions

Las permissions de sistema deberían mantenerse controladas.

EVOXA debe preferir:

System Permission Registry
+
Custom Roles

en lugar de permitir permissions arbitrarias creadas por usuarios.

74. Role Hierarchy

Puede existir:

TenantOwner
   ↓
TenantAdmin
   ↓
Manager
   ↓
Trainer
   ↓
Member

Pero la herencia debe ser explícita.

No asumir:

Role A > Role B

sin una definición formal.

75. Role Inheritance

Ejemplo:

TenantAdmin
 inherits
 Manager

Entonces:

TenantAdmin
=
Manager permissions
+
Additional permissions
76. Permission Conflicts

Cuando existan reglas contradictorias:

ALLOW
+
DENY

EVOXA debe definir una política determinista.

Recomendación:

Explicit Deny
     ↓
takes precedence

cuando una policy explícita de deny exista.

77. Authorization Cache

Las decisiones pueden cachearse cuidadosamente:

subject
+
action
+
resource
+
policyVersion

Pero cambios como:

Role revoked
Permission revoked
Session revoked

deben invalidar rápidamente la información correspondiente.

78. Authorization Consistency

Los cambios críticos de autorización deben tener propagación controlada.

Ejemplo:

Role Removed
 ↓
Permission Cache Invalidation
 ↓
Authorization Nodes
 ↓
New Decision
79. Distributed Authorization

En una arquitectura distribuida:

Service A
Service B
Service C

todos deben aplicar reglas coherentes.

No permitir:

Service A → DENY
Service B → ALLOW

para la misma operación y contexto sin una razón arquitectónica explícita.

80. Authorization Service

Puede existir como servicio dedicado:

Authorization Service

Responsabilidades:

Role Management
Permission Management
Policy Evaluation
Authorization Decisions
Delegation
Audit

Pero la lógica de dominio específica puede permanecer en los servicios correspondientes.

81. Centralized vs Distributed Authorization

Modelo recomendado:

Central Policy
       +
Distributed Enforcement

Es decir:

Authorization Rules
       ↓
Central Governance
       ↓
Local Enforcement

Esto evita un único cuello de botella para cada request.

82. Authorization API

Conceptualmente:

POST /api/v1/authorization/check

Request:

{
  "action": "training.workouts.update",
  "resource": {
    "type": "workout",
    "id": "workout_123"
  }
}

Response:

{
  "decision": "ALLOW"
}

Este endpoint debe protegerse cuidadosamente y no necesariamente exponerse públicamente.

83. Permission APIs

Administración:

GET    /api/v1/permissions
GET    /api/v1/roles
POST   /api/v1/roles
PATCH  /api/v1/roles/{id}
DELETE /api/v1/roles/{id}

POST   /api/v1/roles/{id}/permissions
DELETE /api/v1/roles/{id}/permissions/{permissionId}
84. Role Assignment APIs
POST /api/v1/users/{userId}/roles
DELETE /api/v1/users/{userId}/roles/{roleId}

La asignación debe incluir el contexto:

tenant
scope
expiration

cuando corresponda.

85. Authorization Data Model

Modelo conceptual:

users
   │
   └── user_roles
          │
          ▼
        roles
          │
          ▼
   role_permissions
          │
          ▼
     permissions

Complementado por:

policies
delegations
resource_relationships
86. Authorization Model
User
 │
 ├── Membership
 │      │
 │      └── Tenant
 │
 ├── Roles
 │      │
 │      └── Permissions
 │
 └── Delegations
        │
        └── Scopes
87. Tenant Membership

La membership puede contener:

userId
tenantId
status
role
createdAt
expiresAt

Esto permite controlar acceso tenant-specific.

88. Membership Lifecycle
INVITED
   ↓
ACTIVE
   ↓
SUSPENDED
   ↓
REMOVED

Una membership no activa no debe conceder acceso.

89. Invitations

La invitación debe seguir:

Tenant Admin
 ↓
Invitation
 ↓
Invitee
 ↓
Accept
 ↓
Membership
 ↓
Role Assignment

La invitación no debe convertirse automáticamente en una autorización permanente hasta ser aceptada.

90. Temporary Membership

Puede existir:

Support User
 ↓
Tenant Access
 ↓
expiresAt

El acceso debe revocarse automáticamente.

91. Resource Sharing

EVOXA puede permitir compartir recursos:

Owner
 ↓
Share
 ↓
User
 ↓
Permission

Ejemplo:

Workout
 ↓
Shared with Coach
 ↓
read
92. Resource Sharing Model
ResourceShare
 ├── resourceId
 ├── subjectId
 ├── permissions
 ├── startsAt
 └── expiresAt
93. Authorization and AI

Los modelos AI no deben saltarse autorización.

AI Request
 ↓
User Context
 ↓
Authorization
 ↓
Allowed Tools
 ↓
Tool Execution
94. AI Tool Authorization

Cada tool debe declarar:

tool
required permission
resource type
risk level

Ejemplo:

Tool:
create_workout

Permission:
training.workouts.create
95. High-Risk AI Actions

Algunas acciones pueden requerir:

MFA
User Confirmation
Additional Authorization
Human Approval

Ejemplo:

Delete Client
Issue Refund
Change Subscription
Delete Tenant
96. Authorization and Billing

Billing debe tener permissions específicas:

billing.subscription.read
billing.subscription.update
billing.invoice.read
billing.invoice.create
billing.refund.create
billing.payment.manage

No utilizar:

billing.admin

como única granularidad.

97. Authorization and Security

Security permissions:

security.audit.read
security.sessions.manage
security.mfa.manage
security.policies.manage

deben ser separadas de permissions generales.

98. Authorization and Data Privacy

Recursos con información sensible deben utilizar controles específicos:

Sensitive Data
 ↓
Permission
+
Tenant
+
Purpose
+
Resource Relationship
99. Authorization Testing

Debe probarse:

Allowed Access
Denied Access
Cross-Tenant Access
Ownership
Role Changes
Permission Changes
Scope Restrictions
Expired Roles
Delegations
Resource Sharing
Agent Permissions
Service Permissions
100. Authorization Security Testing

Casos adicionales:

IDOR
Privilege Escalation
Horizontal Privilege Escalation
Vertical Privilege Escalation
Tenant Escape
Role Manipulation
Permission Bypass
Token Scope Escalation
Policy Bypass
Mass Assignment
Authorization Cache Staleness
101. Authorization Observability

Métricas:

authorization_allow_total
authorization_deny_total
authorization_policy_error_total
permission_check_total
role_assignment_total
role_revocation_total
delegation_created_total
delegation_revoked_total
102. Authorization Alerts

Alertas potenciales:

Mass Permission Changes
Unexpected Privilege Escalation
Cross-Tenant Access Attempts
Repeated Authorization Denials
SuperAdmin Usage
Break-Glass Activation
Suspicious Agent Actions
103. Authorization Performance

El diseño debe evitar una consulta completa de permisos en cada request.

Estrategia:

Token
 ↓
Cached Role/Permission Metadata
 ↓
Authorization Check
 ↓
Resource Constraint

Las decisiones críticas deben mantener consistencia adecuada.

104. Authorization Resilience

Si el Policy Engine no está disponible:

Critical Authorization
        ↓
FAIL CLOSED

Nunca:

Authorization unavailable
        ↓
ALLOW

especialmente para operaciones sensibles.

105. Authorization Failure

Errores internos nunca deben exponerse.

No devolver:

Policy SQL
Role Configuration
Internal IDs
Policy Source

al cliente.

106. Authorization Audit Trail

Debe ser posible reconstruir:

Who
Did What
To Which Resource
In Which Tenant
When
From Which Session
Under Which Policy
With Which Result
107. Authorization Lifecycle
Identity
   ↓
Membership
   ↓
Role Assignment
   ↓
Permission
   ↓
Policy Evaluation
   ↓
Resource Access
   ↓
Audit

Revocación:

Role Removed
   ↓
Permission Removed
   ↓
Cache Invalidated
   ↓
Access Denied
108. Authorization Architecture Principles
01 — Authentication and Authorization Must Be Separate
02 — Deny by Default
03 — Least Privilege
04 — Explicit Permissions
05 — Tenant Isolation
06 — Resource-Level Authorization
07 — Ownership Must Be Enforced Server-Side
08 — Roles Must Not Be the Only Authorization Mechanism
09 — Policies Must Be Explicit
10 — Scopes Must Limit Delegated Access
11 — Agents Must Have Independent Permissions
12 — Services Must Have Independent Identities
13 — Administrative Access Must Be Audited
14 — Sensitive Operations Require Strong Authorization
15 — Authorization Must Fail Closed
16 — Permission Changes Must Propagate Quickly
17 — Authorization Decisions Must Be Observable
18 — Privileged Operations Must Be Auditable
19 — Authorization Must Be Testable Independently
20 — Authorization Must Never Trust Client-Supplied Permissions
109. Definition of Done

E05 queda definido cuando EVOXA dispone de:

✓ RBAC
✓ Roles
✓ Permissions
✓ Permission Registry
✓ Tenant Roles
✓ Platform Roles
✓ Domain Roles
✓ Role Assignment
✓ Role Hierarchy
✓ Scopes
✓ Resource Authorization
✓ Ownership
✓ Relationship Authorization
✓ ABAC Foundation
✓ Policy Engine Architecture
✓ PDP / PEP
✓ Tenant Isolation
✓ Delegation
✓ Temporary Permissions
✓ Resource Sharing
✓ Service Authorization
✓ Agent Authorization
✓ AI Tool Authorization
✓ Administrative Authorization
✓ Break-Glass Access
✓ Authorization Audit
✓ Authorization Events
✓ Authorization Metrics
✓ Authorization Alerts
✓ Authorization Caching
✓ Fail-Closed Security
✓ Authorization Testing
110. E04 → E05 → E06

La arquitectura de Engineering continúa:

E01
Backend Architecture
       │
       ▼
E02
Database Architecture
       │
       ▼
E03
API Architecture
       │
       ▼
E04
Authentication Architecture
       │
       ▼
E05
Authorization Architecture
       │
       ▼
E06
Policy Architecture

La separación fundamental queda:

                 EVOXA SECURITY
                       │
          ┌────────────┴────────────┐
          │                         │
 AUTHENTICATION                AUTHORIZATION
          │                         │
       E04                         E05
          │                         │
     "Who are you?"          "What can you do?"
                                    │
                                    ▼
                              POLICY ENGINE
                                    │
                                   E06
                                    │
                              "Under what
                               conditions?"

E05 establece la capa de autorización de EVOXA: quién puede ejecutar qué acción, sobre qué recurso, dentro de qué tenant y bajo qué permisos, roles, scopes y relaciones. E06 deberá profundizar la arquitectura formal de Policies y el motor de decisión que utilizará EVOXA para expresar y evaluar esas reglas.
