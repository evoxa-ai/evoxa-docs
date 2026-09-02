IS-IDENTITY-004 — Membership Management

Implementation Story ID: IS-IDENTITY-004
Título: Membership Management
Dominio: Identity
ESP: ESP-0001 — Identity
Prioridad: Critical
Estado: Draft
Versión: 1.0.0
Repositorio documental: evoxa-docs
Repositorio de implementación: evoxa-platform

1. Propósito

Esta Implementation Story define la capacidad de gestionar la relación entre usuarios y organizaciones dentro de EVOXA mediante la entidad Membership.

El Membership representa la pertenencia de un User a una Organization y constituye una pieza fundamental del modelo de multi-tenancy, autorización y aislamiento de datos.

La relación debe mantenerse separada de las entidades User y Organization:

User
 │
 │
 ▼
Membership
 │
 │
 ▼
Organization

Esto permite que un mismo usuario pueda potencialmente pertenecer a múltiples organizaciones.

La arquitectura de EVOXA establece Identity como responsable de Users, Organizations, Memberships, Roles, Permissions, Sessions y Security, y establece el tenant isolation como una responsabilidad central de seguridad.

2. Business Goal

EVOXA necesita una forma segura y auditable de determinar:

A qué organizaciones pertenece un usuario.
Qué estado tiene esa pertenencia.
Qué rol o roles tiene dentro de una organización.
Quién creó la relación.
Quién puede administrarla.
Cuándo fue creada, activada, suspendida o eliminada.
Qué organización constituye el contexto de una operación.

El objetivo es que los dominios posteriores puedan confiar en Membership como parte del Security Context sin tener que implementar nuevamente la lógica de pertenencia.

3. Persona
Personas principales
Platform Administrator
Organization Administrator
Organization Owner
Team Manager
Coach
Nutritionist
Physiotherapist
Usuario final

Los permisos concretos serán determinados por Authorization & Policy Engine.

4. Journey

El ciclo general será:

Usuario
   ↓
Invitación / creación de Membership
   ↓
INVITED
   ↓
Aceptación
   ↓
ACTIVE
   ↓
Uso dentro de Organization
   ↓
SUSPENDED
   ↓
ACTIVE
   ↓
REMOVED

La pertenencia no debe confundirse con el lifecycle del usuario.

Un usuario puede estar:

User = ACTIVE
Membership A = ACTIVE
Membership B = SUSPENDED
5. Feature Pack

Identity → Membership Management

Incluye:

Membership creation
Membership identity
User ↔ Organization relationship
Membership lifecycle
Membership activation
Membership suspension
Membership removal
Membership role association
Tenant context
Membership validation
Audit
Domain events
6. Alcance
Incluido

Esta historia cubre:

Entidad Membership.
MembershipId.
Relación User ↔ Organization.
Estados de Membership.
Creación.
Consulta.
Actualización.
Activación.
Suspensión.
Eliminación lógica.
Validación de pertenencia.
Asociación con roles.
Repository contract.
Application services.
Domain events.
Audit integration.
Tenant isolation.
Tests.
Fuera de alcance

No se implementa aquí:

User Management.
Organization Management.
Authentication.
JWT.
Refresh Tokens.
Sessions.
MFA.
Permission Management.
Role Definition.
Billing.
Subscription.
UI.
Mobile.
Kubernetes.

Las capacidades anteriores corresponden a otras IS.

7. Membership Entity

Entidad conceptual:

Membership
├── id
├── user_id
├── organization_id
├── status
├── role references
├── created_at
├── updated_at
└── lifecycle metadata

El Membership es una entidad propia y no simplemente un atributo de User o Organization.

8. MembershipId

Debe existir:

MembershipId

como Value Object.

Características:

Único.
Estable.
No depende del email.
No depende del nombre de organización.
No cambia durante la vida del Membership.
9. Relación User ↔ Organization

La relación fundamental es:

User
  │
  │ 1
  │
  │ N
  ▼
Membership
  │
  │ N
  │
  │ 1
  ▼
Organization

Esto permite:

Un usuario → múltiples organizaciones
User A
 ├── Membership → Organization A
 ├── Membership → Organization B
 └── Membership → Organization C
Una organización → múltiples usuarios
Organization A
 ├── Membership → User A
 ├── Membership → User B
 ├── Membership → User C
 └── Membership → User D
10. Regla Fundamental de Tenant

El acceso de un usuario a una organización debe derivarse de un Membership válido.

Por tanto:

User + Organization

por sí solos no constituyen autorización.

Debe existir:

User
 ↓
Membership ACTIVE
 ↓
Organization
11. Membership Lifecycle

Los estados definidos son:

INVITED
ACTIVE
SUSPENDED
REMOVED
INVITED

La pertenencia fue creada/iniciada pero todavía no ha sido activada.

ACTIVE

El usuario pertenece activamente a la organización.

SUSPENDED

La pertenencia está temporalmente restringida.

REMOVED

La pertenencia fue terminada.

12. Lifecycle Transitions

Modelo conceptual:

INVITED ─────→ ACTIVE
    │
    └─────────→ REMOVED

ACTIVE ──────→ SUSPENDED
   │              │
   │              ▼
   │            ACTIVE
   │
   └──────────→ REMOVED

SUSPENDED ────→ REMOVED

Las transiciones deben validarse en el dominio.

No se debe permitir modificar status arbitrariamente.

13. User Stories
US-IDENTITY-004-01 — Crear Membership

Como administrador autorizado
Quiero asociar un usuario a una organización
Para permitirle pertenecer a ella.

Acceptance Criteria

AC-001

Given que existe un User válido
And existe una Organization válida
When se crea un Membership
Then se genera un MembershipId único
And la relación queda registrada.

AC-002

Given que el usuario ya posee un Membership activo con la organización
When se intenta crear otro Membership equivalente
Then la operación debe ser rechazada.
14. US-IDENTITY-004-02 — Consultar Membership

Como actor autorizado
Quiero consultar la pertenencia de un usuario
Para conocer su relación con una organización.

Acceptance Criteria

AC-003

Given que el Membership existe
When un actor autorizado lo consulta
Then se devuelve la información permitida.

AC-004

Given que el Membership no existe
When se consulta
Then debe devolverse MembershipNotFound.
15. US-IDENTITY-004-03 — Activar Membership

Como administrador autorizado
Quiero activar una pertenencia
Para permitir al usuario operar dentro de la organización.

Acceptance Criteria

AC-005

Given que Membership está INVITED
When se completa un flujo válido de activación
Then Membership pasa a ACTIVE.

Debe generarse:

MembershipActivated
16. US-IDENTITY-004-04 — Suspender Membership

Como administrador autorizado
Quiero suspender una pertenencia
Para impedir temporalmente que el usuario opere dentro de una organización.

Acceptance Criteria

AC-006

Given que Membership está ACTIVE
When un actor autorizado lo suspende
Then pasa a SUSPENDED
And se genera MembershipSuspended.

La suspensión del Membership no implica necesariamente suspender al usuario globalmente.

Ejemplo:

User = ACTIVE

Organization A
Membership = SUSPENDED

Organization B
Membership = ACTIVE

El usuario podría continuar operando en Organization B si las políticas lo permiten.

17. US-IDENTITY-004-05 — Remover Membership

Como administrador autorizado
Quiero remover la pertenencia
Para terminar el acceso del usuario a una organización.

Acceptance Criteria

AC-007

Given que Membership está ACTIVE o SUSPENDED
When se remueve
Then pasa a REMOVED
And se genera MembershipRemoved.

REMOVED representa terminación lógica de la relación.

No implica necesariamente eliminación física del registro.

18. US-IDENTITY-004-06 — Reactivar Membership

Como administrador autorizado
Quiero reactivar un Membership suspendido
Para permitir nuevamente el acceso a la organización.

Acceptance Criteria

AC-008

Given que Membership está SUSPENDED
When un actor autorizado solicita reactivarlo
Then pasa a ACTIVE
And se genera MembershipActivated.

La reactivación de un Membership REMOVED no debe asumirse automáticamente válida.

19. Membership + Roles

El Membership es el contexto natural para asignar roles organizacionales.

Conceptualmente:

User
 │
 ▼
Membership
 │
 ├── Organization
 │
 └── Role

Ejemplo:

Sebastian
   │
   └── Membership
          ├── Organization: EVOXA Gym
          └── Role: Coach

Mientras que el mismo usuario podría tener:

Sebastian
   │
   └── Membership
          ├── Organization: EVOXA Clinic
          └── Role: Team Manager

Esto permite que los permisos puedan variar según la organización.

La gestión completa de Roles y Permissions se implementará posteriormente en:

IS-IDENTITY-007 — Role & Permission Management.

20. Membership como Security Context

Una solicitud autenticada podrá construir conceptualmente:

SecurityContext
├── user_id
├── organization_id
├── membership_id
├── roles
├── permissions
├── session
├── authentication_method
├── MFA state
├── device
└── request metadata

El Membership proporciona la relación que conecta:

User
    ↓
Organization

dentro del contexto de autorización.

21. Tenant Isolation

Una de las responsabilidades críticas de esta IS es impedir acceso cross-tenant.

Ejemplo:

User A
Membership → Organization A

Debe poder acceder a recursos de:

Organization A

pero no automáticamente a:

Organization B

aunque conozca el organization_id.

La validación debe ser:

Request
   ↓
Authenticated User
   ↓
Membership
   ↓
Organization
   ↓
Authorization
   ↓
Resource
22. Membership Uniqueness

Debe existir una restricción lógica para evitar duplicación de la relación:

User A + Organization A

no debería generar:

Membership 1
Membership 2
Membership 3

equivalentes y activos.

La estrategia física de constraint/index será definida en Persistence.

23. Organization Suspension

Si:

Organization = SUSPENDED

un Membership:

Membership = ACTIVE

no debe interpretarse como autorización efectiva para operar.

El acceso final debe considerar ambos estados:

User Status
       +
Membership Status
       +
Organization Status
       +
Authorization Policy

Ejemplo:

User ACTIVE
Membership ACTIVE
Organization SUSPENDED
             ↓
           DENY
24. User Suspension

De igual manera:

User = SUSPENDED
Membership = ACTIVE
Organization = ACTIVE

no debe permitir automáticamente acceso.

El estado global del usuario tiene precedencia sobre el Membership.

25. Domain Rules
BR-IDENTITY-004-01

Un Membership debe referenciar un User existente.

BR-IDENTITY-004-02

Un Membership debe referenciar una Organization existente.

BR-IDENTITY-004-03

Un Membership debe tener un identificador único.

BR-IDENTITY-004-04

No deben existir Memberships activos duplicados para el mismo User + Organization.

BR-IDENTITY-004-05

Membership debe representar la relación User ↔ Organization.

BR-IDENTITY-004-06

Las transiciones de lifecycle deben ser validadas por el dominio.

BR-IDENTITY-004-07

SUSPENDED restringe la pertenencia sin necesariamente suspender al User globalmente.

BR-IDENTITY-004-08

REMOVED representa una relación terminada.

BR-IDENTITY-004-09

Un Membership no puede proporcionar acceso si el User está suspendido/deshabilitado.

BR-IDENTITY-004-10

Un Membership no puede proporcionar acceso efectivo si la Organization está suspendida.

BR-IDENTITY-004-11

El acceso debe respetar tenant isolation.

BR-IDENTITY-004-12

Los cambios relevantes deben generar domain events.

BR-IDENTITY-004-13

Las operaciones administrativas deben ser auditables.

26. Application Layer

Casos de uso:

CreateMembership
GetMembership
ActivateMembership
SuspendMembership
RemoveMembership
ReactivateMembership

Flujo:

API
 ↓
Application Service
 ↓
Authorization / Policy
 ↓
Membership Domain
 ↓
Repository
 ↓
Infrastructure
27. Repository Contract

Se debe definir:

MembershipRepository

Operaciones conceptuales:

create()
get_by_id()
get_by_user_and_organization()
list_by_user()
list_by_organization()
exists()
update()
save()

La implementación física queda fuera del Domain Layer.

28. Domain Events

Esta IS introduce:

MembershipCreated
MembershipActivated
MembershipSuspended
MembershipRemoved
MembershipRoleChanged

MembershipRoleChanged queda preparado para integrarse con Role Management.

El envelope debe seguir:

event_id
event_type
event_version
timestamp
tenant_id
actor
resource
payload
metadata
correlation_id
causation_id
trace_id
29. MembershipCreated

Payload conceptual:

{
    membership_id,
    user_id,
    organization_id,
    status,
    created_at
}

No incluir:

passwords;
tokens;
MFA secrets;
recovery codes;
private keys.
30. MembershipActivated

Se genera cuando:

INVITED → ACTIVE

o mediante otra transición de reactivación explícitamente válida.

31. MembershipSuspended

Se genera cuando:

ACTIVE → SUSPENDED

Puede ser consumido por:

Authorization.
Session Management.
Notifications.
Audit.
Analytics.
32. MembershipRemoved

Se genera cuando:

ACTIVE → REMOVED

o:

SUSPENDED → REMOVED

Los consumidores pueden utilizarlo para invalidar contextos de acceso existentes.

33. MembershipRoleChanged

Este evento permitirá comunicar cambios de rol dentro de una organización.

Ejemplo:

Coach
  ↓
Team Manager

El evento deberá incluir únicamente la información necesaria para comunicar el cambio.

34. Audit

Operaciones auditables:

MembershipCreated
MembershipActivated
MembershipSuspended
MembershipRemoved
MembershipRoleChanged

Información conceptual:

actor
user
organization
membership
action
timestamp
result
correlation_id
metadata

Nunca:

password
password_hash
tokens
MFA secrets
recovery codes
private keys
35. Authorization Boundary

Membership Management no debe decidir directamente:

"Este usuario puede administrar Membership."

Debe consultar:

Authorization
       +
Security Policy Engine

Por ejemplo:

Actor
 ↓
Authentication
 ↓
Security Context
 ↓
Authorization
 ↓
Policy
 ↓
Membership Management

Esto permite posteriormente implementar reglas como:

Organization Administrator
    → gestionar memberships de su organización

Platform Administrator
    → gestionar memberships según políticas SYSTEM

Regular Coach
    → no gestionar memberships

Las reglas concretas quedan fuera de esta IS.

36. API Requirements

Namespace:

/api/v1/identity

Endpoints conceptuales:

POST   /memberships
GET    /memberships/{membership_id}
POST   /memberships/{membership_id}/activate
POST   /memberships/{membership_id}/suspend
POST   /memberships/{membership_id}/reactivate
POST   /memberships/{membership_id}/remove

También serán necesarios endpoints de consulta contextual, por ejemplo:

GET /users/{user_id}/memberships
GET /organizations/{organization_id}/memberships

La forma definitiva de estos contratos debe quedar en:

IS-IDENTITY-014 — Identity API Contracts.

37. Error Model

Errores conceptuales:

MembershipNotFound
MembershipAlreadyExists
InvalidMembershipState
InvalidMembershipTransition
UserNotFound
OrganizationNotFound
UserSuspended
UserDisabled
OrganizationSuspended
TenantAccessDenied
Unauthorized
Forbidden
38. Persistence Requirements

PostgreSQL será la fuente transaccional según ADR-IDENTITY-006.

Tabla conceptual:

memberships

Campos conceptuales:

id
user_id
organization_id
status
created_at
updated_at

La asociación con roles podrá evolucionar mediante las estructuras definidas en Role & Permission Management.

La estructura física definitiva será responsabilidad de:

IS-IDENTITY-015 — Identity Persistence.

39. Database Integrity

La persistencia debe proteger:

Foreign Keys
membership.user_id
       ↓
users.id
membership.organization_id
       ↓
organizations.id
Uniqueness

Debe existir una protección contra Memberships activos equivalentes:

(user_id, organization_id)

La estrategia exacta —constraint, índice parcial u otra— se determinará durante el diseño físico de PostgreSQL.

40. Concurrency

Debe protegerse el siguiente escenario:

Request A ──→ Create Membership
Request B ──→ Create Membership
                  ↓
              same User
              same Org

El resultado debe ser:

Membership único

y no dos Memberships equivalentes.

También debe protegerse el lifecycle:

ACTIVE → SUSPENDED
ACTIVE → REMOVED

cuando ambas operaciones ocurren simultáneamente.

41. Idempotency

Operaciones como:

ActivateMembership
SuspendMembership
RemoveMembership

deben tener comportamiento determinista.

Ejemplo:

Suspend Membership
Suspend Membership
Suspend Membership

no debe producir múltiples cambios semánticos.

42. Security Requirements

Debe cumplirse:

Fail closed.
Least privilege.
Tenant isolation.
No cross-tenant access.
No privilege escalation.
No exposición innecesaria de memberships.
Validación de estado de User.
Validación de estado de Organization.
Auditabilidad.
No secretos en eventos.
No secretos en logs.
No secretos en respuestas API.
43. Acceptance Criteria Consolidated
ID	Criterio
AC-001	Se puede crear Membership entre User y Organization válidos
AC-002	Se evita duplicación de Membership
AC-003	Se puede consultar Membership
AC-004	Se puede activar Membership
AC-005	Se puede suspender Membership
AC-006	Se puede reactivar Membership
AC-007	Se puede remover Membership
AC-008	Se validan todas las transiciones
AC-009	User suspendido no puede obtener acceso efectivo
AC-010	User disabled no puede obtener acceso efectivo
AC-011	Organization suspendida bloquea acceso efectivo
AC-012	Se mantiene tenant isolation
AC-013	Membership puede asociarse a contexto de roles
AC-014	Se generan domain events
AC-015	Las operaciones son auditables
AC-016	No se exponen secretos
AC-017	Se protegen operaciones concurrentes
AC-018	Operaciones críticas son idempotentes
AC-019	Se mantiene separación Domain/Application/Infrastructure
44. Technical Tasks
TASK-IDENTITY-004-01 — Definir Membership Entity

Crear la entidad Membership.

TASK-IDENTITY-004-02 — Implementar MembershipId

Crear:

MembershipId
TASK-IDENTITY-004-03 — Definir Membership Status

Implementar:

INVITED
ACTIVE
SUSPENDED
REMOVED
TASK-IDENTITY-004-04 — Implementar Lifecycle Rules

Definir y validar las transiciones.

TASK-IDENTITY-004-05 — Definir User ↔ Organization Relationship

Establecer formalmente la relación mediante Membership.

TASK-IDENTITY-004-06 — Definir MembershipRepository

Crear:

MembershipRepository

con sus operaciones abstractas.

TASK-IDENTITY-004-07 — Implementar CreateMembership

Debe:

validar User;
validar Organization;
verificar duplicidad;
crear Membership;
establecer estado inicial;
persistir;
generar MembershipCreated;
registrar auditoría.
TASK-IDENTITY-004-08 — Implementar GetMembership

Debe respetar:

autorización;
tenant context;
minimización de información.
TASK-IDENTITY-004-09 — Implementar ActivateMembership

Implementar:

INVITED → ACTIVE

y reactivación válida de SUSPENDED.

TASK-IDENTITY-004-10 — Implementar SuspendMembership

Implementar:

ACTIVE → SUSPENDED
TASK-IDENTITY-004-11 — Implementar RemoveMembership

Implementar:

ACTIVE → REMOVED
SUSPENDED → REMOVED
TASK-IDENTITY-004-12 — Implementar Tenant Context

Integrar:

user_id
organization_id
membership_id

en el Security Context.

TASK-IDENTITY-004-13 — Implementar Membership Validation

Validar conjuntamente:

User
+
Membership
+
Organization
TASK-IDENTITY-004-14 — Implementar Domain Events

Crear:

MembershipCreated
MembershipActivated
MembershipSuspended
MembershipRemoved
MembershipRoleChanged
TASK-IDENTITY-004-15 — Integrar AuditService

Registrar las operaciones administrativas relevantes.

TASK-IDENTITY-004-16 — Definir Persistence Mapping

Preparar:

Membership Entity
       ↓
Persistence Model
       ↓
memberships
TASK-IDENTITY-004-17 — Implementar Uniqueness Protection

Proteger:

user_id + organization_id

contra duplicación equivalente.

TASK-IDENTITY-004-18 — Implementar Concurrency Protection

Proteger creación y lifecycle frente a operaciones simultáneas.

TASK-IDENTITY-004-19 — Implementar Idempotency

Probar operaciones repetidas de lifecycle.

TASK-IDENTITY-004-20 — Unit Tests

Cubrir:

Membership;
MembershipId;
lifecycle;
transitions;
validation;
uniqueness;
domain errors.
TASK-IDENTITY-004-21 — Application Tests

Cubrir:

CreateMembership
GetMembership
ActivateMembership
SuspendMembership
ReactivateMembership
RemoveMembership
TASK-IDENTITY-004-22 — Tenant Isolation Tests

Probar:

User A + Organization A → ALLOW
User A + Organization B → DENY

cuando no existe Membership válido.

TASK-IDENTITY-004-23 — Security Tests

Cubrir:

Unauthorized.
Forbidden.
Cross-tenant access.
Privilege escalation.
Suspended User.
Disabled User.
Suspended Organization.
TASK-IDENTITY-004-24 — Event Tests

Verificar generación correcta de todos los eventos.

TASK-IDENTITY-004-25 — Traceability

Actualizar:

ESP-0001
    ↓
IS-IDENTITY-004
    ↓
TASK-IDENTITY-004-01 ... 025
    ↓
Source Code
    ↓
Tests
45. Definition of Done
 Membership Entity implementada.
 MembershipId implementado.
 Lifecycle implementado.
 Transiciones validadas.
 User ↔ Organization definido mediante Membership.
 Repository contract implementado.
 CreateMembership implementado.
 GetMembership implementado.
 ActivateMembership implementado.
 SuspendMembership implementado.
 ReactivateMembership implementado.
 RemoveMembership implementado.
 Tenant Context integrado.
 Membership validation implementada.
 Domain Events implementados.
 Audit integrado.
 Uniqueness protection implementada.
 Concurrency protection implementada.
 Idempotency validada.
 Unit tests implementados.
 Application tests implementados.
 Tenant isolation tests implementados.
 Security tests implementados.
 Event tests implementados.
 Coverage ≥85% general.
 Coverage ≥95% en áreas críticas.
 Traceability actualizada.
 Architecture Review aprobado.
 QA Review aprobado.
 Security Review aprobado cuando corresponda.
46. Traceability Matrix
Elemento	Referencia
Blueprint	Architecture Map
Domain	Identity
ESP	ESP-0001
Predecessor	IS-IDENTITY-001
Predecessor	IS-IDENTITY-002
Predecessor	IS-IDENTITY-003
Current IS	IS-IDENTITY-004
ADR	ADR-IDENTITY-001
ADR	ADR-IDENTITY-005
ADR	ADR-IDENTITY-006
ADR	ADR-IDENTITY-007
ADR	ADR-IDENTITY-010
Source	apps/api/app/domains/identity/
Tests	apps/api/tests/identity/

La documentación de arquitectura establece que Identity corresponde al dominio apps/api/app/domains/identity/ y a ESP-0001, y que las Implementation Stories deben mantener trazabilidad hasta las tareas técnicas y posteriormente el código fuente.

47. Dependencies
Depends on
IS-IDENTITY-001 — Identity Domain Foundation
IS-IDENTITY-002 — User & Account Management
IS-IDENTITY-003 — Organization & Tenant Management

ADR-IDENTITY-001
ADR-IDENTITY-005
ADR-IDENTITY-006
ADR-IDENTITY-007
ADR-IDENTITY-010
Required by
IS-IDENTITY-005 — Authentication
IS-IDENTITY-006 — Authorization & Permission Evaluation
IS-IDENTITY-007 — Role & Permission Management
IS-IDENTITY-008 — Session Management
IS-IDENTITY-009 — Refresh Token Lifecycle
IS-IDENTITY-010 — Multi-Factor Authentication
IS-IDENTITY-011 — Security Policy Engine
48. Architectural Model

Con las tres IS anteriores, el modelo empieza a tomar esta forma:

                         EVOXA Identity
                              │
                ┌─────────────┴─────────────┐
                │                           │
               User                    Organization
                │                           │
                └───────────┬───────────────┘
                            │
                       Membership
                            │
                   ┌────────┴────────┐
                   │                 │
                 Role              Tenant
                   │                 │
                   └────────┬────────┘
                            │
                     Authorization
                            │
                     Business Domains

La separación es deliberada:

User
  = ¿Quién eres?

Organization
  = ¿A qué tenant pertenece el contexto?

Membership
  = ¿Cuál es tu relación con ese tenant?

Role
  = ¿Qué función tienes allí?

Permission
  = ¿Qué puedes hacer?

Policy
  = ¿En qué contexto puedes hacerlo?

Esto prepara correctamente el terreno para Authentication → Authorization → Roles & Permissions sin mezclar responsabilidades entre entidades.

49. Estado de la IS

IS-IDENTITY-004 — Membership Management

Estado: Draft

La historia queda lista para revisión arquitectónica y para continuar con:

IS-IDENTITY-005 — Authentication

Ahí comenzaremos a conectar formalmente:

User
   ↓
Organization
   ↓
Membership
   ↓
Authentication
   ↓
Session
   ↓
Access Token
   ↓
Refresh Token

sin saltarnos las capas de arquitectura ni comenzar todavía con código.
