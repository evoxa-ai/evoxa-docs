IS-IDENTITY-007 — Role & Permission Management

Implementation Story ID: IS-IDENTITY-007
Título: Role & Permission Management
Dominio: Identity
ESP: ESP-0001 — Identity
Prioridad: High / Critical para autorización
Estado: Draft
Versión: 1.0.0
Repositorio documental: evoxa-docs
Repositorio de implementación: evoxa-platform

1. Propósito

Esta Implementation Story define la capacidad de crear, administrar, asignar, modificar y revocar Roles y Permissions dentro de EVOXA.

Esta capacidad proporciona la base estructural sobre la cual trabaja el sistema de Authorization & Permission Evaluation definido en:

IS-IDENTITY-006 — Authorization & Permission Evaluation

La separación fundamental será:

Role
  ↓
agrupa
  ↓
Permissions
  ↓
se asigna a
  ↓
Membership
  ↓
User + Organization

Por tanto:

User
  ↓
Membership
  ↓
Role
  ↓
Permission
  ↓
Authorization
  ↓
Decision

La gestión de Roles y Permissions debe permanecer separada de Authentication y de la evaluación de autorización.

2. Business Goal

EVOXA necesita un modelo flexible de control de acceso que permita definir qué operaciones puede realizar cada tipo de usuario dentro de cada organización.

Debe ser posible:

Crear Roles.
Crear Permissions.
Actualizar Roles.
Actualizar Permissions.
Eliminar/desactivar Roles.
Eliminar/desactivar Permissions.
Asignar Roles a Memberships.
Revocar Roles.
Asociar Permissions a Roles.
Revocar Permissions de Roles.
Definir alcance de Roles.
Consultar Roles y Permissions.
Evitar privilegios implícitos.
Mantener tenant isolation.
Auditar cambios de seguridad.
Invalidar caches de autorización cuando cambie el modelo de permisos.
3. Principio Fundamental

EVOXA debe distinguir claramente:

Role
    = conjunto lógico de permissions

Permission
    = capacidad específica

Membership
    = relación del usuario con una organización

Authorization
    = evaluación de si esa capacidad puede utilizarse

Ejemplo:

Role: Coach

Permissions:
    training_programs.read
    training_programs.create
    training_programs.update

El usuario no obtiene esos permisos simplemente por existir.

Debe existir:

User
 ↓
Membership
 ↓
Role = Coach
 ↓
Permissions
4. Personas

Roles del sistema pueden representar personas como:

Coach.
Nutritionist.
Physiotherapist.
Team Manager.
Organization Administrator.
Platform Administrator.

La arquitectura de Identity ya contempla roles como Coach, Nutritionist, Physiotherapist, Team Manager y Admin.

Importante: los roles funcionales exactos y su catálogo definitivo deben ser tratados como configuración/datos de plataforma, no como lógica hard-coded dentro de los dominios.

5. Feature Pack

Identity → Role & Permission Management

Incluye:

Role Management.
Permission Management.
Role-Permission Association.
Membership-Role Assignment.
Role Scope.
Permission Naming.
Role Lifecycle.
Permission Lifecycle.
Assignment/Revoke.
Authorization Cache Invalidation.
Audit.
Security Events.
Tenant isolation.
Administrative controls.
6. Alcance
Incluido

Esta IS cubre:

Role.
Permission.
RolePermission.
Role scope.
Role lifecycle.
Permission lifecycle.
Role creation.
Permission creation.
Role update.
Permission update.
Role deletion/deactivation.
Permission deletion/deactivation.
Role assignment a Membership.
Role revocation.
Permission assignment a Role.
Permission revocation.
Validación.
Audit.
Domain events.
Authorization cache invalidation.
Tests.
Fuera de alcance

No implementa:

Authentication.
JWT.
Passwords.
Sessions.
Refresh Tokens.
MFA.
Security Policy Engine completo.
User Management.
Organization Management.
Membership Management.
UI.
Mobile.
Business-domain specific authorization.
7. Modelo Conceptual

El modelo principal:

                    User
                      │
                      ▼
                 Membership
                      │
                      ▼
                     Role
                      │
                      ▼
              RolePermission
                      │
                      ▼
                 Permission

Ejemplo:

Sebastian
   │
   ▼
Membership
   │
   ├── Organization: EVOXA Gym
   │
   └── Role: Coach
             │
             ├── training_programs.read
             ├── training_programs.create
             └── training_programs.update
8. Permission

Un Permission representa una capacidad específica del sistema.

La convención propuesta:

<resource>.<action>

Ejemplos:

users.read
users.create
users.update
users.delete

training_programs.read
training_programs.create
training_programs.update
training_programs.delete

nutrition_plans.read
nutrition_plans.create
nutrition_plans.update
nutrition_plans.publish

La nomenclatura final deberá mantenerse centralizada y gobernada para evitar duplicaciones semánticas.

9. Permission Entity

Modelo conceptual:

Permission
├── id
├── resource
├── action
├── description
├── status
├── created_at
└── updated_at

El identificador lógico del permiso debe derivarse de:

resource + action

sin depender de texto presentado al usuario.

10. PermissionId

Value Object:

PermissionId

Debe ser:

único;
estable;
independiente del nombre visible;
no reutilizable después de eliminar un permiso cuando exista trazabilidad histórica.
11. Permission Lifecycle

El Blueprint/ADRs disponibles no establecen un lifecycle detallado para Permission con valores concretos.

Por tanto, no debemos inventar estados normativos.

Para implementación se propone como diseño derivado:

ACTIVE
INACTIVE

pero queda sujeto a revisión arquitectónica.

Mientras no sea aprobado, debe tratarse como:

Derived Design Proposal — no Blueprint normative requirement.

12. Role

Un Role representa una agrupación lógica de Permissions.

Modelo:

Role
├── id
├── name
├── description
├── scope
├── status
├── created_at
└── updated_at
13. RoleId

Value Object:

RoleId

Características:

único;
estable;
independiente del nombre;
no dependiente del usuario;
no dependiente de la organización.
14. Role Scope

La arquitectura/ADR existente propone dos scopes:

SYSTEM
ORGANIZATION
SYSTEM

Rol gestionado a nivel de plataforma EVOXA.

Ejemplo conceptual:

Platform Administrator
ORGANIZATION

Rol aplicable dentro de una organización específica.

Ejemplo:

Coach
Nutritionist
Team Manager

Esta distinción es importante:

SYSTEM Role
     ≠
Organization Role
15. No Implicit Role Hierarchy

No debe asumirse automáticamente:

Admin > Manager > Coach

La arquitectura establecida indica que inicialmente no habrá una jerarquía implícita de roles.

Si un rol necesita permisos superiores:

Role A

debe tener explícitamente esos permissions.

No:

Role A inherits Role B

a menos que una futura decisión arquitectónica lo habilite.

16. RolePermission

La relación entre Role y Permission debe ser explícita:

Role
  │
  │
  ▼
RolePermission
  │
  │
  ▼
Permission

Ejemplo:

Coach
 │
 ├── RolePermission → training_programs.read
 ├── RolePermission → training_programs.create
 └── RolePermission → training_programs.update
17. Membership ↔ Role

La asignación de roles ocurre dentro del contexto de Membership:

User
 │
 ▼
Membership
 │
 ├── Organization
 │
 └── Role

Esto permite:

User A
 │
 ├── Organization A → Coach
 │
 └── Organization B → Team Manager

Por tanto, un rol organizacional no debe ser un atributo global del User.

18. User Stories
US-IDENTITY-007-01 — Crear Permission

Como administrador autorizado
Quiero crear un Permission
Para definir una capacidad del sistema.

Acceptance Criteria
Given que el permission es válido
When se crea
Then debe generarse un PermissionId único
And debe quedar disponible para Role Management.
19. US-IDENTITY-007-02 — Crear Role

Como administrador autorizado
Quiero crear un Role
Para agrupar permissions.

Acceptance Criteria
Given que el Role es válido
When se crea
Then debe generarse un RoleId único
And debe quedar disponible para asignación.
20. US-IDENTITY-007-03 — Asignar Permission a Role
Given que existe un Role
And existe un Permission
When se crea RolePermission
Then el Role debe obtener ese Permission.
21. US-IDENTITY-007-04 — Revocar Permission
Given que un Role posee un Permission
When un administrador autorizado lo revoca
Then el Permission deja de pertenecer al Role.

Debe producir la invalidación correspondiente de authorization cache.

22. US-IDENTITY-007-05 — Asignar Role a Membership
Given que existe Membership
And existe Role válido
When se asigna el Role
Then Membership obtiene ese Role
And la autorización puede utilizarlo.
23. US-IDENTITY-007-06 — Revocar Role
Given que Membership posee un Role
When el Role es revocado
Then Membership deja de tener ese Role
And las decisiones de autorización afectadas deben invalidarse.
24. US-IDENTITY-007-07 — Actualizar Role
Given que el Role existe
When un administrador modifica información permitida
Then el Role debe actualizarse
And debe producirse el evento correspondiente.
25. US-IDENTITY-007-08 — Administrar Scope
Given que se crea un Role
When se define su scope
Then el sistema debe validar SYSTEM u ORGANIZATION.

No se debe permitir:

scope = "anything"
26. US-IDENTITY-007-09 — Evitar Privilege Escalation
Given que un actor administra roles
When intenta asignarse directa o indirectamente permisos superiores
Then la operación debe ser evaluada por Authorization + Policy
And debe rechazarse si no está permitida.

Este punto es crítico.

Un usuario no debe poder modificar el sistema de autorización para obtener más privilegios sin una autorización previa equivalente.

27. Role Assignment Security

Una operación como:

POST /memberships/{id}/roles

no significa automáticamente:

ALLOW

Debe pasar por:

Actor
 ↓
Authentication
 ↓
Authorization
 ↓
Security Policy
 ↓
Role Management
28. Domain Rules
BR-IDENTITY-007-01

Cada Role debe tener un RoleId único.

BR-IDENTITY-007-02

Cada Permission debe tener un PermissionId único.

BR-IDENTITY-007-03

Un Role puede tener múltiples Permissions.

BR-IDENTITY-007-04

Un Permission puede pertenecer a múltiples Roles.

BR-IDENTITY-007-05

RolePermission representa explícitamente la relación Role ↔ Permission.

BR-IDENTITY-007-06

Los Roles ORGANIZATION deben operar dentro del contexto de una Organization.

BR-IDENTITY-007-07

Los Roles SYSTEM no deben quedar accidentalmente limitados a un tenant.

BR-IDENTITY-007-08

No existe jerarquía implícita de roles.

BR-IDENTITY-007-09

La asignación de Roles debe realizarse sobre Membership y no directamente sobre User.

BR-IDENTITY-007-10

No se debe permitir la duplicación lógica de RolePermission.

BR-IDENTITY-007-11

No se debe permitir la duplicación lógica de asignación de Role a Membership.

BR-IDENTITY-007-12

Los cambios de Roles/Permissions deben invalidar las decisiones de autorización afectadas.

BR-IDENTITY-007-13

Los cambios de seguridad deben ser auditables.

BR-IDENTITY-007-14

Role Management debe estar protegido por Authorization.

BR-IDENTITY-007-15

Un actor no puede ampliar sus propios privilegios sin autorización explícita.

BR-IDENTITY-007-16

Las operaciones deben respetar Tenant Isolation.

29. Role Scope Rules
SYSTEM
Role.scope = SYSTEM

Puede representar capacidades de plataforma.

ORGANIZATION
Role.scope = ORGANIZATION

Debe estar asociado al contexto de una organización.

Ejemplo:

Membership
 ├── Organization = EVOXA Gym
 └── Role = Coach
30. Role & Permission Evaluation

Esta IS proporciona datos a:

IS-IDENTITY-006

La relación será:

IS-IDENTITY-007
Role & Permission Management
          │
          ▼
      Role
          │
          ▼
     Permissions
          │
          ▼
IS-IDENTITY-006
Authorization Evaluation

IS-007 administra qué existe.

IS-006 decide si se puede utilizar.

31. Application Layer

Casos de uso:

CreateRole
GetRole
UpdateRole
DeactivateRole

CreatePermission
GetPermission
UpdatePermission
DeactivatePermission

AssignPermissionToRole
RevokePermissionFromRole

AssignRoleToMembership
RevokeRoleFromMembership
32. Repository Contracts

Se requieren:

RoleRepository
PermissionRepository
RolePermissionRepository

y para asignación:

MembershipRoleRepository

Conceptualmente:

domain/repositories/
├── role_repository
├── permission_repository
├── role_permission_repository
└── membership_role_repository

Las implementaciones concretas pertenecen a Infrastructure.

33. RoleRepository

Operaciones conceptuales:

create()
get_by_id()
get_by_name()
update()
exists()
save()
34. PermissionRepository

Operaciones:

create()
get_by_id()
get_by_resource_action()
update()
exists()
save()
35. RolePermissionRepository

Operaciones:

assign()
revoke()
exists()
list_permissions()
36. MembershipRoleRepository

Operaciones:

assign()
revoke()
exists()
list_roles()

Esta separación evita introducir directamente relaciones SQL en el Domain Layer.

37. Domain Events

Eventos principales:

RoleCreated
RoleUpdated
RoleDeleted
RoleAssigned
RoleRemoved

PermissionCreated
PermissionUpdated
PermissionAssigned
PermissionRevoked

Para Membership:

MembershipRoleChanged
38. RoleCreated

Payload conceptual:

{
  "role_id": "...",
  "name": "...",
  "scope": "ORGANIZATION"
}

No debe incluir información sensible.

39. PermissionCreated

Payload conceptual:

{
  "permission_id": "...",
  "resource": "training_programs",
  "action": "read"
}
40. PermissionAssigned

Evento:

PermissionAssigned

representa:

Role
 ↓
Permission
41. PermissionRevoked

Representa:

Role
 X
Permission

Este evento debe poder activar invalidación de authorization cache.

42. RoleAssigned

Representa:

Membership
 ↓
Role

Este cambio modifica directamente el contexto de autorización del usuario.

Por ello debe considerarse un cambio de seguridad de alta importancia.

43. RoleRemoved

Representa:

Membership
 X
Role

Debe invalidar las decisiones de autorización afectadas.

44. Audit

Deben auditarse:

RoleCreated
RoleUpdated
RoleDeleted
RoleAssigned
RoleRemoved

PermissionCreated
PermissionUpdated
PermissionAssigned
PermissionRevoked

Además:

Privilege escalation attempts
Unauthorized role changes
Unauthorized permission changes
Cross-tenant role operations

Nunca almacenar:

password
tokens
MFA secrets
private keys
recovery codes
45. Cache Invalidation

Un cambio de Role o Permission puede cambiar inmediatamente una autorización.

Ejemplo:

Coach
 ↓
training_programs.update

Si se revoca:

Coach
 X
training_programs.update

una decisión previamente cacheada como:

ALLOW

no debe continuar indefinidamente.

Debe existir:

Role/Permission Change
        ↓
Security Event
        ↓
Cache Invalidation
        ↓
New Authorization Evaluation
46. Tenant Isolation

Roles ORGANIZATION deben respetar:

Role
 ↓
Organization Context

Un Role creado para:

Organization A

no debe asignarse automáticamente a:

Organization B

sin una operación válida y autorizada.

47. SYSTEM Roles

Los Roles SYSTEM deben manejarse con especial protección.

Ejemplo conceptual:

SYSTEM
 └── Platform Administrator

Un administrador organizacional no debe poder asignarse:

Platform Administrator

simplemente porque tiene:

roles.update

La asignación de privilegios SYSTEM debe estar protegida por políticas de seguridad superiores.

48. Permission Naming Governance

Debe evitarse crear permisos semánticamente duplicados:

training.read
training.view
training_programs.read

si realmente representan la misma capacidad.

Debe existir una gobernanza central del catálogo.

El catálogo de Permissions debe ser tratado como un contrato de seguridad.

49. API Requirements

Namespace:

/api/v1/identity

Endpoints conceptuales:

Roles
POST   /roles
GET    /roles
GET    /roles/{role_id}
PATCH  /roles/{role_id}
DELETE /roles/{role_id}
Permissions
POST   /permissions
GET    /permissions
GET    /permissions/{permission_id}
PATCH  /permissions/{permission_id}
DELETE /permissions/{permission_id}
Role Permissions
POST   /roles/{role_id}/permissions
DELETE /roles/{role_id}/permissions/{permission_id}
GET    /roles/{role_id}/permissions
Membership Roles
POST   /memberships/{membership_id}/roles
DELETE /memberships/{membership_id}/roles/{role_id}
GET    /memberships/{membership_id}/roles

La definición definitiva pertenece a:

IS-IDENTITY-014 — Identity API Contracts.

50. Error Model

Errores conceptuales:

RoleNotFound
RoleAlreadyExists
PermissionNotFound
PermissionAlreadyExists
RolePermissionAlreadyExists
RolePermissionNotFound
MembershipRoleAlreadyExists
MembershipRoleNotFound
InvalidRoleScope
InvalidRoleState
PermissionInUse
RoleInUse
Forbidden
Unauthorized
TenantAccessDenied
PrivilegeEscalationDenied
51. Persistence Model

Tablas conceptuales:

roles
permissions
role_permissions
membership_roles

Relaciones:

roles
  │
  ▼
role_permissions
  │
  ▼
permissions

y:

memberships
  │
  ▼
membership_roles
  │
  ▼
roles

La definición física final corresponde a:

IS-IDENTITY-015 — Identity Persistence.

52. Database Integrity

Debe existir protección para:

role_permissions
    UNIQUE(role_id, permission_id)

y:

membership_roles
    UNIQUE(membership_id, role_id)

Las constraints exactas de PostgreSQL deben definirse durante Persistence.

53. Concurrency

Debe protegerse:

Request A → Assign Role
Request B → Assign Role

para evitar:

MembershipRole 1
MembershipRole 2

equivalentes.

Lo mismo para:

RolePermission
54. Idempotency

Operaciones como:

AssignPermissionToRole
AssignRoleToMembership
RevokePermissionFromRole
RevokeRoleFromMembership

deben ser deterministas.

Una segunda solicitud equivalente no debe crear una segunda relación.

55. Security Requirements

Esta IS debe cumplir:

Least privilege.
Fail closed.
Tenant isolation.
No implicit role hierarchy.
No privilege self-escalation.
Protection de SYSTEM roles.
Auditability.
Cache invalidation.
No secrets.
Authorization previa a operaciones administrativas.
Protección contra cross-tenant role assignment.
Protección contra permission escalation.
56. AI Authorization

Los Roles y Permissions también serán relevantes para los componentes de IA.

Ejemplo:

Coach
   ↓
nutrition_plans.read

no significa automáticamente que un AI Agent pueda leer todos los planes nutricionales.

La IA debe recibir un contexto autorizado limitado:

Authorized Context
        ↓
AI Context

y nunca:

AI Context > Authorized Context

Esto mantiene la regla establecida en IS-IDENTITY-006.

57. Acceptance Criteria Consolidated
ID	Criterio
AC-001	Se pueden crear Roles
AC-002	Se pueden crear Permissions
AC-003	Se evita duplicación de Roles
AC-004	Se evita duplicación de Permissions
AC-005	Se pueden asociar Permissions a Roles
AC-006	Se pueden revocar Permissions
AC-007	Se pueden asignar Roles a Memberships
AC-008	Se pueden revocar Roles
AC-009	Se soportan scopes SYSTEM y ORGANIZATION
AC-010	No existe jerarquía implícita de Roles
AC-011	Organization Roles respetan tenant isolation
AC-012	SYSTEM Roles están protegidos
AC-013	Se evita privilege self-escalation
AC-014	Los cambios invalidan authorization cache
AC-015	Las operaciones relevantes son auditables
AC-016	Se generan Domain Events
AC-017	Se mantiene separación Role/Permission/Authorization
AC-018	Las relaciones son idempotentes
AC-019	Se protege concurrencia
AC-020	No se exponen secretos
AC-021	AI no puede superar el contexto autorizado
58. Technical Tasks
TASK-IDENTITY-007-01 — Definir Role Entity

Crear entidad Role.

TASK-IDENTITY-007-02 — Implementar RoleId

Crear Value Object RoleId.

TASK-IDENTITY-007-03 — Definir Permission Entity

Crear entidad Permission.

TASK-IDENTITY-007-04 — Implementar PermissionId

Crear Value Object PermissionId.

TASK-IDENTITY-007-05 — Definir Role Scope

Implementar:

SYSTEM
ORGANIZATION
TASK-IDENTITY-007-06 — Definir Role Lifecycle

Implementar lifecycle aprobado para Role.

TASK-IDENTITY-007-07 — Definir Permission Lifecycle

Implementar lifecycle aprobado para Permission.

TASK-IDENTITY-007-08 — Definir RolePermission Entity

Crear representación de:

Role ↔ Permission
TASK-IDENTITY-007-09 — Definir MembershipRole Entity

Crear representación de:

Membership ↔ Role
TASK-IDENTITY-007-10 — Implementar RoleRepository

Crear contrato abstracto.

TASK-IDENTITY-007-11 — Implementar PermissionRepository

Crear contrato abstracto.

TASK-IDENTITY-007-12 — Implementar RolePermissionRepository

Crear contrato abstracto.

TASK-IDENTITY-007-13 — Implementar MembershipRoleRepository

Crear contrato abstracto.

TASK-IDENTITY-007-14 — Implementar CreateRole

Crear caso de uso.

TASK-IDENTITY-007-15 — Implementar UpdateRole

Crear actualización controlada.

TASK-IDENTITY-007-16 — Implementar Delete/Deactivate Role

Implementar lifecycle de Role sin destruir información histórica de seguridad innecesariamente.

TASK-IDENTITY-007-17 — Implementar CreatePermission

Crear caso de uso.

TASK-IDENTITY-007-18 — Implementar UpdatePermission

Actualizar atributos permitidos.

TASK-IDENTITY-007-19 — Implementar Delete/Deactivate Permission

Gestionar lifecycle sin romper referencias históricas.

TASK-IDENTITY-007-20 — Implementar AssignPermissionToRole

Crear:

Role → Permission
TASK-IDENTITY-007-21 — Implementar RevokePermissionFromRole

Eliminar/revocar relación.

TASK-IDENTITY-007-22 — Implementar AssignRoleToMembership

Crear:

Membership → Role
TASK-IDENTITY-007-23 — Implementar RevokeRoleFromMembership

Eliminar/revocar relación.

TASK-IDENTITY-007-24 — Implementar Privilege Escalation Protection

Evitar que un actor pueda ampliar sus propios privilegios sin autorización.

TASK-IDENTITY-007-25 — Implementar SYSTEM Role Protection

Proteger operaciones sobre roles de scope SYSTEM.

TASK-IDENTITY-007-26 — Implementar Tenant Isolation

Validar el contexto de Organization para roles organizacionales.

TASK-IDENTITY-007-27 — Implementar Domain Events

Crear:

RoleCreated
RoleUpdated
RoleDeleted
RoleAssigned
RoleRemoved
PermissionCreated
PermissionUpdated
PermissionAssigned
PermissionRevoked
TASK-IDENTITY-007-28 — Integrar AuditService

Auditar cambios de seguridad.

TASK-IDENTITY-007-29 — Implementar Cache Invalidation

Invalidar autorizaciones afectadas por cambios de Role/Permission.

TASK-IDENTITY-007-30 — Unit Tests

Cubrir:

Role.
Permission.
RolePermission.
MembershipRole.
Scope.
Lifecycle.
TASK-IDENTITY-007-31 — Application Tests

Cubrir todos los casos de uso.

TASK-IDENTITY-007-32 — Tenant Isolation Tests

Probar operaciones cross-tenant.

TASK-IDENTITY-007-33 — Privilege Escalation Tests

Probar:

self-role assignment;
self-permission escalation;
SYSTEM role escalation;
organization boundary violations.
TASK-IDENTITY-007-34 — Cache Security Tests

Validar invalidación correcta después de:

role assignment;
role removal;
permission assignment;
permission revocation.
TASK-IDENTITY-007-35 — Concurrency Tests

Probar asignaciones concurrentes.

TASK-IDENTITY-007-36 — Idempotency Tests

Probar operaciones repetidas.

TASK-IDENTITY-007-37 — Event Tests

Verificar todos los eventos de seguridad.

TASK-IDENTITY-007-38 — Security Regression Tests

Prevenir:

authorization bypass;
privilege escalation;
tenant escape;
stale permissions;
role manipulation.
TASK-IDENTITY-007-39 — API Contract Tests

Validar contratos de Roles y Permissions.

TASK-IDENTITY-007-40 — Traceability

Actualizar:

ESP-0001
    ↓
IS-IDENTITY-007
    ↓
TASK-IDENTITY-007-01 ... 040
    ↓
Source Code
    ↓
Tests
59. Definition of Done
 Role Entity implementada.
 Permission Entity implementada.
 RoleId implementado.
 PermissionId implementado.
 Role Scope definido.
 Role lifecycle definido.
 Permission lifecycle definido.
 RolePermission definido.
 MembershipRole definido.
 Repository contracts implementados.
 CRUD de Roles implementado.
 CRUD de Permissions implementado.
 Permission assignment implementado.
 Permission revocation implementada.
 Role assignment implementado.
 Role revocation implementada.
 SYSTEM role protection implementada.
 Privilege escalation protection implementada.
 Tenant isolation implementado.
 Domain Events implementados.
 Audit integrado.
 Cache invalidation implementada.
 Unit tests implementados.
 Application tests implementados.
 Tenant isolation tests implementados.
 Privilege escalation tests implementados.
 Cache security tests implementados.
 Concurrency tests implementados.
 Idempotency tests implementados.
 Event tests implementados.
 Security regression tests implementados.
 API contract tests implementados.
 Coverage ≥85% general.
 Coverage ≥95% en áreas críticas.
 Traceability actualizada.
 Architecture Review aprobado.
 QA Review aprobado.
 Security Review aprobado.
60. Traceability Matrix
Elemento	Referencia
Blueprint	Architecture Map
Domain	Identity
ESP	ESP-0001
Predecessor	IS-IDENTITY-001
Predecessor	IS-IDENTITY-002
Predecessor	IS-IDENTITY-003
Predecessor	IS-IDENTITY-004
Predecessor	IS-IDENTITY-005
Predecessor	IS-IDENTITY-006
Current IS	IS-IDENTITY-007
ADR	ADR-IDENTITY-001
ADR	ADR-IDENTITY-005
ADR	ADR-IDENTITY-006
ADR	ADR-IDENTITY-007
ADR	ADR-IDENTITY-010
ADR	ADR-IDENTITY-011
Source	apps/api/app/domains/identity/
Tests	apps/api/tests/identity/

La cadena ESP → IS → Technical Tasks → Source Code es la trazabilidad requerida por la documentación de ingeniería de EVOXA.

61. Dependencies
Depends on
IS-IDENTITY-001 — Identity Domain Foundation
IS-IDENTITY-002 — User & Account Management
IS-IDENTITY-003 — Organization & Tenant Management
IS-IDENTITY-004 — Membership Management
IS-IDENTITY-005 — Authentication
IS-IDENTITY-006 — Authorization & Permission Evaluation

ADR-IDENTITY-001
ADR-IDENTITY-005
ADR-IDENTITY-006
ADR-IDENTITY-007
ADR-IDENTITY-008
ADR-IDENTITY-010
ADR-IDENTITY-011
Required by
IS-IDENTITY-008 — Session Management
IS-IDENTITY-009 — Refresh Token Lifecycle
IS-IDENTITY-010 — Multi-Factor Authentication
IS-IDENTITY-011 — Security Policy Engine
IS-IDENTITY-012 — Audit & Security Events
IS-IDENTITY-014 — Identity API Contracts
62. Architectural Model

Con esta IS queda completo el núcleo RBAC inicial:

                         USER
                           │
                           ▼
                      MEMBERSHIP
                           │
                  ┌────────┴────────┐
                  │                 │
             ORGANIZATION          ROLE
                                    │
                                    ▼
                             ROLE_PERMISSION
                                    │
                                    ▼
                               PERMISSION
                                    │
                                    ▼
                              AUTHORIZATION
                                    │
                    ┌───────────────┼──────────────┐
                    │               │              │
                   RBAC            ABAC         POLICY
                    │               │              │
                    └───────────────┼──────────────┘
                                    ▼
                                 DECISION

La responsabilidad de cada pieza queda:

User
→ identidad

Organization
→ tenant

Membership
→ pertenencia

Role
→ agrupación de capacidades

Permission
→ capacidad concreta

RolePermission
→ relación Role ↔ Permission

Authorization
→ evaluación

Policy Engine
→ condiciones y restricciones
63. Secuencia de Identity actualizada
IS-IDENTITY-001
Identity Domain Foundation
        ↓
IS-IDENTITY-002
User & Account Management
        ↓
IS-IDENTITY-003
Organization & Tenant Management
        ↓
IS-IDENTITY-004
Membership Management
        ↓
IS-IDENTITY-005
Authentication
        ↓
IS-IDENTITY-006
Authorization & Permission Evaluation
        ↓
IS-IDENTITY-007
Role & Permission Management
        ↓
IS-IDENTITY-008
Session Management
        ↓
IS-IDENTITY-009
Refresh Token Lifecycle
        ↓
IS-IDENTITY-010
Multi-Factor Authentication
        ↓
IS-IDENTITY-011
Security Policy Engine
        ↓
IS-IDENTITY-012
Audit & Security Events
        ↓
IS-IDENTITY-013
Domain Events & Outbox
        ↓
IS-IDENTITY-014
Identity API Contracts
        ↓
IS-IDENTITY-015
Identity Persistence
        ↓
IS-IDENTITY-016
Identity Testing & Quality
        ↓
IS-IDENTITY-017
Identity Runtime & Deployment
64. Estado
IS-IDENTITY-007 — Role & Permission Management

Estado: Draft

Con esta historia ya tenemos definido el núcleo de RBAC de EVOXA y su conexión con Membership y Authorization.

El siguiente paso lógico es:

IS-IDENTITY-008 — Session Management

Ahí debemos separar claramente:

Authentication
      ↓
Session
      ↓
Access Token
      ↓
Refresh Token

y definir el lifecycle de las sesiones, múltiples dispositivos, expiración, revocación, logout, límites de sesiones, estados y relación entre sesión y usuario/Membership.
