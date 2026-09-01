ESP-0001 / 05 — Identity Roles & Permissions

Document ID: ESP-0001/05
Module: Identity
Version: 1.0.0
Status: Draft
Parent: ESP-0001 — Identity
Previous Specifications:

ESP-0001/01 — Identity Domain Model
ESP-0001/02 — Identity Architecture
ESP-0001/03 — Identity Authentication & Authorization
ESP-0001/04 — Identity User & Organization Management

Architecture Parent: BP-0002 — Reference Architecture
Repository: evoxa-platform
Target Path: apps/api/app/domains/identity/

1. Purpose

Este documento define el modelo de Roles & Permissions de EVOXA.

Su objetivo es establecer cómo la plataforma determina:

qué puede hacer una identidad autenticada, dentro de qué organización y bajo qué condiciones.

El modelo debe soportar:

Roles;
Permissions;
Role-Permission assignments;
RBAC;
ABAC;
autorización por organización;
autorización por tenant;
ownership;
políticas contextuales;
asignación y revocación de roles;
evaluación de permisos;
roles de plataforma;
roles organizacionales;
auditoría;
evolución futura del modelo.

La arquitectura de EVOXA establece RBAC + ABAC, least privilege, tenant isolation, organization membership y ownership como elementos fundamentales de autorización.

2. Architectural Principle

El modelo debe separar cuatro conceptos:

User
 │
 ▼
Membership
 │
 ▼
Role
 │
 ▼
Permission

Pero la autorización completa no termina allí:

User
 │
 ▼
Membership
 │
 ▼
Role
 │
 ▼
Permission
 │
 ▼
ABAC / Policy
 │
 ▼
Ownership / Context
 │
 ▼
ALLOW / DENY

Por tanto:

Role ≠ Permission ≠ Authorization Decision.

3. Role
3.1 Definition

Un Role representa una función o conjunto lógico de responsabilidades dentro de EVOXA.

Ejemplos contemplados por la arquitectura:

Coach
Nutritionist
Physiotherapist
Team Manager
Admin

La API Architecture identifica estos perfiles dentro del modelo de autorización.

4. Permission

Una Permission representa una capacidad concreta que puede ser autorizada.

Modelo propuesto:

resource.action

Ejemplos:

clients.read
clients.create
clients.update
clients.delete

training.read
training.create
training.update
training.delete

nutrition.read
nutrition.create
nutrition.update

Este formato es una decisión de diseño derivada; el Blueprint establece autorización por permisos/RBAC, pero no fija literalmente este naming convention.

5. RolePermission

RolePermission representa la relación:

Role
 │
 └── RolePermission
          │
          ▼
      Permission

Ejemplo:

Coach
 ├── clients.read
 ├── clients.update
 ├── training.read
 └── training.create

Debe existir una restricción de unicidad:

(role_id, permission_id)

para evitar asignaciones duplicadas.

6. Role Scope

Una decisión importante es que no todos los roles necesariamente tienen el mismo alcance.

Proponemos distinguir:

SYSTEM
ORGANIZATION
System Role

Aplica a toda la plataforma.

Ejemplo conceptual:

Platform Administrator
Organization Role

Aplica dentro de una organización.

Ejemplo:

Coach
Nutritionist
Physiotherapist
Team Manager

Esta separación es Derived Design y debe convertirse en ADR antes de implementación.

7. Role Context

Un rol organizacional debe evaluarse dentro de una membership:

User
 │
 ▼
Membership
 │
 ├── Organization A
 │
 └── Role: Coach

Esto evita interpretar:

Coach

como un permiso global.

La pregunta correcta es:

¿Este usuario tiene el rol Coach en esta organización?

8. Role Assignment

La asignación conceptual será:

User
 │
 ▼
Membership
 │
 ▼
Role

No:

User
 └── global_role

para roles organizacionales.

Esto mantiene el modelo compatible con multi-tenancy.

9. Permission Assignment
Role
 │
 ▼
RolePermission
 │
 ▼
Permission

Ejemplo:

Role: Coach

Permissions:
    clients.read
    clients.update
    training.read
    training.create
10. RBAC Model

RBAC proporciona la primera capa:

User
  ↓
Membership
  ↓
Role
  ↓
Permission

Ejemplo:

Coach
   │
   └── training.create

Si el usuario no posee el permiso:

training.create

la operación debe ser denegada.

11. RBAC Is Not Enough

Un permiso válido no significa necesariamente que una operación concreta pueda ejecutarse.

Ejemplo:

Coach
   ↓
training.update = YES

pero:

Training Program 456
belongs to Coach B

Por tanto:

RBAC → ALLOW
ABAC / Ownership → DENY

Resultado final:

DENY

Este modelo es consistente con la arquitectura de seguridad que combina RBAC, ABAC y ownership.

12. ABAC Model

ABAC agrega contexto:

User
+
Role
+
Permission
+
Organization
+
Resource
+
Resource Owner
+
Resource State
+
Policy

Resultado:

Authorization Decision
13. Authorization Formula

Conceptualmente:

ALLOW =
    Authenticated
    AND AccountActive
    AND ValidMembership
    AND CorrectTenant
    AND HasPermission
    AND PolicyAllows
    AND OwnershipAllows

Si cualquiera de los requisitos obligatorios falla:

DENY
14. Permission Evaluation

La evaluación será:

Request
   │
   ▼
Authentication
   │
   ▼
Tenant Context
   │
   ▼
Membership
   │
   ▼
Roles
   │
   ▼
Permissions
   │
   ▼
ABAC Policies
   │
   ▼
Ownership
   │
   ▼
Decision
15. Permission Naming

Propuesta inicial:

<resource>.<action>

Ejemplo:

users.read
users.create
users.update
users.delete

organizations.read
organizations.create
organizations.update

memberships.read
memberships.create
memberships.delete

clients.read
clients.create
clients.update
clients.delete

El estándar definitivo deberá quedar registrado en un ADR.

16. Actions

Las acciones iniciales podrían incluir:

read
create
update
delete

y eventualmente:

list
view
approve
publish
assign
revoke
export
import
execute
manage

No todos los recursos deben implementar todas las acciones.

17. Resource Model

Una permission pertenece conceptualmente a un recurso:

Permission
├── resource
└── action

Ejemplo:

resource = training-programs
action   = update

Resultado:

training-programs.update
18. Permission Granularity

El sistema debe evitar permisos excesivamente amplios.

❌ Evitar:

admin.all

cuando sea posible.

Preferir:

users.read
users.update
roles.assign
memberships.manage

Esto sigue el principio de Least Privilege.

19. System Roles

Se propone permitir roles globales de plataforma.

Ejemplo conceptual:

PlatformAdmin
PlatformSupport
PlatformOperator

Pero estos nombres no están fijados por el Blueprint.

El Blueprint sí contempla Admin dentro de los roles de API y Identity como capability transversal.

La taxonomía definitiva requiere ADR.

20. Organization Roles

Los roles profesionales definidos por la arquitectura pueden funcionar como roles organizacionales:

Coach
Nutritionist
Physiotherapist
Team Manager

Ejemplo:

Organization A
 ├── Coach
 ├── Nutritionist
 └── Team Manager
21. Administrator

El término Admin requiere especial cuidado.

No deberíamos asumir automáticamente:

Admin = acceso absoluto

Podemos tener:

Platform Admin
Organization Admin

con diferentes scopes.

Esta distinción deberá resolverse mediante ADR.

22. Role Hierarchy

No se recomienda introducir herencia de roles automáticamente.

Ejemplo potencial:

Admin
  ↓
Manager
  ↓
Coach

puede generar privilegios implícitos difíciles de auditar.

Por defecto:

Role A
≠
Role B

y los permisos deben asignarse explícitamente.

Si posteriormente necesitamos role inheritance:

Role
 ↓
Parent Role

deberá diseñarse como capacidad independiente.

23. Permission Inheritance

Por defecto:

Role A
   └── Permission X

no implica:

Role B
   └── Permission X

Cada asignación debe ser explícita.

Esto facilita:

auditoría;
least privilege;
trazabilidad;
testing;
evolución.
24. Deny by Default

La regla fundamental:

No explicit authorization
          ↓
        DENY

No se debe utilizar:

Unknown
 ↓
ALLOW

La arquitectura de seguridad de EVOXA sigue Zero Trust y Secure by Default.

25. Explicit Deny

Aunque el modelo inicial puede funcionar únicamente con:

ALLOW

o:

DENY

se debe mantener abierta la posibilidad futura de políticas explícitas de deny.

Ejemplo:

Role → ALLOW
Policy → DENY

Resultado:

DENY

La prioridad exacta de políticas debe documentarse posteriormente.

26. Tenant Scope

Una permission debe evaluarse dentro de un contexto.

training.update

no significa:

Update ANY training program

significa:

May attempt to update training programs
subject to tenant and business policies.
27. Resource Ownership

Ejemplo:

Coach A
 └── Client 1

Coach B
 └── Client 2

Ambos podrían tener:

clients.read

pero eso no significa necesariamente:

Coach A → Client 2

La política de ownership debe decidirlo.

28. Organization-Level Permissions

Algunas permissions estarán orientadas a administración:

organization.read
organization.update

memberships.read
memberships.create
memberships.remove

roles.read
roles.assign
roles.revoke

Estas permissions deben estar limitadas al tenant correspondiente.

29. Platform-Level Permissions

Algunas capacidades pueden ser globales:

platform.users.manage
platform.organizations.manage
platform.security.manage

Solo identidades con scope de plataforma deberían poder ejecutarlas.

30. Role Assignment

Flujo:

Admin
 │
 ▼
Request Role Assignment
 │
 ▼
Authentication
 │
 ▼
Authorization
 │
 ▼
Validate Organization
 │
 ▼
Validate Role
 │
 ▼
Assign Role
 │
 ├── Audit
 └── RoleAssigned

La persona que asigna un rol también debe estar autorizada para hacerlo.

31. Role Revocation
Role Assignment
      │
      ▼
Revoke
      │
      ├── Update Authorization State
      ├── Audit
      └── RoleRevoked

El efecto debe ser inmediato o estar sujeto a una estrategia de cache claramente definida.

32. Permission Changes

Si se modifica un rol:

Coach
 │
 ├── training.read
 ├── training.create
 └── training.update

y se revoca:

training.update

la nueva autorización debe reflejar el cambio de forma segura.

No debemos mantener permisos obsoletos indefinidamente en cache.

33. Authorization Cache

Puede utilizarse caching para mejorar rendimiento:

User
 ↓
Role
 ↓
Permissions
 ↓
Cache

pero cualquier cambio crítico debe invalidar el cache correspondiente.

Esto es una decisión de implementación derivada.

34. Authorization Context

Conceptualmente:

AuthorizationContext
├── user_id
├── organization_id
├── membership_id
├── roles
├── permissions
├── resource
├── action
└── policy_context

La decisión:

AuthorizationDecision
├── allowed
├── policy
├── reason
└── metadata
35. Policy Engine

Propuesta:

AuthorizationService
        │
        ├── RBAC Evaluator
        │
        ├── ABAC Evaluator
        │
        ├── Tenant Policy
        │
        ├── Ownership Policy
        │
        └── Resource Policy

La arquitectura no obliga a una tecnología concreta para implementar este componente.

Por tanto, no debemos fijar todavía un motor específico de políticas.

36. Domain Ownership

Una consideración importante para EVOXA:

Identity debe proporcionar el framework de autorización, pero los dominios deben mantener las reglas específicas de sus recursos.

Ejemplo:

Identity
   │
   └── training.update

Training:

Is this program editable?
Is user the assigned coach?
Is program published?
Is modification allowed?

Esto mantiene DDD y bounded contexts.

37. Role Lifecycle

Propuesta:

DRAFT
  │
  ▼
ACTIVE
  │
  ▼
DISABLED

No se debería borrar inmediatamente un rol que haya sido utilizado históricamente.

El lifecycle definitivo debe definirse mediante ADR.

38. Permission Lifecycle

Propuesta:

DRAFT
  │
  ▼
ACTIVE
  │
  ▼
DEPRECATED
  │
  ▼
RETIRED

Esto permite evolucionar la plataforma sin romper inmediatamente referencias históricas.

39. Permission Versioning

Una permission puede necesitar evolución.

Ejemplo:

training.update

podría cambiar sus reglas internas sin cambiar necesariamente su nombre.

Para cambios incompatibles:

training.update.v2

podría considerarse.

Pero la estrategia exacta debe alinearse con API/Event versioning.

40. Role Assignment Audit

Toda modificación de autorización debe ser auditable:

WHO assigned
WHO received
WHAT role
IN WHICH organization
WHEN
RESULT

Ejemplo:

RoleAssigned
{
    actor_id,
    user_id,
    organization_id,
    role_id,
    timestamp,
    correlation_id
}
41. Security Events

Eventos relacionados:

RoleAssigned
RoleRevoked

PermissionAssigned
PermissionRevoked

AuthorizationGranted
AuthorizationDenied

Los eventos deben representar hechos ocurridos y seguir el envelope común de Event Architecture.

42. AuthorizationDenied

Los rechazos importantes deben producir telemetría de seguridad.

Ejemplo:

User
 │
 ▼
Permission Evaluation
 │
 ▼
DENIED
 │
 ├── Security Log
 ├── Audit
 └── Metrics

El sistema no debe exponer detalles internos innecesarios al atacante.

43. API Boundary

Las operaciones futuras podrían incluir:

GET    /api/v1/roles
POST   /api/v1/roles
PATCH  /api/v1/roles/{id}

GET    /api/v1/permissions

POST   /api/v1/roles/{id}/permissions
DELETE /api/v1/roles/{id}/permissions/{permissionId}

POST   /api/v1/users/{id}/roles
DELETE /api/v1/users/{id}/roles/{roleId}

Estos NO son todavía contratos oficiales.

Los contratos deberán definirse en:

ESP-0001 / 13 — Identity API Contracts

siguiendo API-First.

44. Example: Coach

Supongamos:

User: U123
Organization: ORG-A
Role: Coach

Permissions:

clients.read
clients.update
training.read
training.create
training.update

Request:

PATCH /api/v1/training-programs/456

Evaluación:

Authentication
      ↓
User U123
      ↓
Organization ORG-A
      ↓
Role Coach
      ↓
training.update
      ↓
Program tenant = ORG-A
      ↓
Ownership Policy
      ↓
ALLOW
45. Example: Cross-Tenant Attack
User U123
Organization A

intenta:

Training Program
Organization B

Aunque tenga:

training.update

el resultado debe ser:

RBAC:
ALLOW

Tenant Policy:
DENY

FINAL:
DENY

La protección cross-tenant es obligatoria desde la arquitectura de seguridad.

46. Example: Insufficient Permission
Role:
Coach

Permissions:

clients.read
training.read

Request:

DELETE /api/v1/clients/123

Evaluación:

clients.delete
     ↓
NOT FOUND
     ↓
DENY
47. Example: Role Revocation

Antes:

Coach
 └── training.update

Después:

Coach
 └── training.read

Nueva request:

training.update

resultado:

DENY

La invalidación de cache debe garantizar que la autorización actualizada sea aplicada correctamente.

48. Security Requirements

Roles y Permissions deben cumplir:

Least Privilege;
Zero Trust;
Secure by Default;
Tenant Isolation;
Auditability;
Explicit Authorization;
No privilege escalation;
No cross-tenant access.

Estos principios están definidos en la arquitectura de seguridad.

49. Privilege Escalation Protection

Un usuario no debe poder:

Asignarse un rol superior
Asignarse permisos
Modificar su propio rol
Modificar políticas de seguridad

salvo que disponga explícitamente de autorización para hacerlo.

Ejemplo:

Coach
   ↓
POST /roles/admin
   ↓
DENY
50. Separation of Duties

Para operaciones extremadamente sensibles puede ser necesario separar responsabilidades.

Ejemplo conceptual:

User A
 → creates role

User B
 → approves role

Esta capacidad todavía no está definida por el Blueprint y debe considerarse una evolución futura.

51. API Security

Las APIs de Roles & Permissions deben utilizar:

HTTPS
Authentication
Authorization
Input Validation
Rate Limiting
Audit
Correlation IDs

La API Architecture establece estos principios de seguridad para las APIs.

52. Persistence Model

Modelo conceptual:

users
organizations
memberships

roles
permissions
role_permissions

La asignación del rol a la membership puede representarse posteriormente mediante:

membership_roles

si se aprueba que una membership pueda tener múltiples roles.

53. Multiple Roles

Una decisión importante:

User
 ↓
Membership
 ↓
Role A
Role B

¿Debe una membership tener:

1 role

o:

N roles

El Blueprint no fija esta cardinalidad de manera suficientemente explícita.

Por tanto:

ADR-IDENTITY-026 — Multiple Roles per Membership

debe resolverse antes de implementar el esquema definitivo.

54. Proposed Flexible Model

Como diseño preliminar recomiendo:

Membership
    │
    └── MembershipRole
             │
             └── Role

Esto permitiría:

User
 └── Organization A
       ├── Coach
       └── Team Manager

Pero esta es una propuesta, no una decisión definitiva.

55. Role Scope Model

La propuesta completa sería:

Role
├── id
├── name
├── description
├── scope
│    ├── SYSTEM
│    └── ORGANIZATION
├── status
└── metadata

Esto permite diferenciar:

PlatformAdmin
    scope = SYSTEM

de:

Coach
    scope = ORGANIZATION
56. Permission Model

Propuesta:

Permission
├── id
├── resource
├── action
├── description
├── status
└── metadata

Ejemplo:

{
    "resource": "training-programs",
    "action": "update"
}
57. RolePermission Model
RolePermission
├── role_id
└── permission_id

Restricción:

UNIQUE(role_id, permission_id)
58. Authorization Evaluation Pseudoflow

Sin entrar todavía en código:

authorize(
    user,
    organization,
    resource,
    action
)

debe conceptualmente realizar:

1. Verify authentication
2. Verify account status
3. Resolve membership
4. Resolve roles
5. Resolve permissions
6. Check requested permission
7. Evaluate tenant policy
8. Evaluate ABAC
9. Evaluate ownership
10. Return decision
59. Fail-Closed

Si cualquiera de los elementos críticos no puede determinarse:

Permission unknown
Membership unknown
Tenant unknown
Policy unavailable

resultado:

DENY

Esto es especialmente importante para una plataforma de salud/wellness que manejará información sensible.

60. Observability

Authorization deberá producir métricas como:

authorization.allowed
authorization.denied
authorization.latency
authorization.policy_failures

y logs estructurados con:

user_id
tenant_id
resource
action
decision
correlation_id

sin incluir información sensible innecesaria.

La arquitectura de observabilidad exige logs estructurados, métricas, tracing y correlación.

61. Testing Matrix
RBAC
Role has permission
Role lacks permission
Multiple roles
Role revoked
Permission revoked
ABAC
Correct tenant
Wrong tenant
Correct owner
Wrong owner
Resource state
Policy condition
Security
Privilege escalation
Cross-tenant access
Unauthorized role assignment
Unauthorized permission assignment
Lifecycle
Role created
Role disabled
Permission deprecated
Role revoked
62. Acceptance Criteria

ESP-0001/05 estará preparado para implementación cuando:

Roles
 Role model definido.
 Role scope definido.
 Role lifecycle definido.
 Role assignment definido.
 Role revocation definido.
Permissions
 Permission model definido.
 Resource/action convention definida.
 Permission lifecycle definido.
 Permission assignment definido.
Authorization
 RBAC definido.
 ABAC definido.
 Tenant isolation definido.
 Ownership definido.
 Fail-closed definido.
Security
 Least privilege.
 Privilege escalation protection.
 Audit.
 Security events.
 Authorization telemetry.
63. ADR Candidates

Este documento deja identificadas las siguientes decisiones:

ADR	Decisión
ADR-IDENTITY-003	Roles globales vs organizacionales
ADR-IDENTITY-004	Permission naming
ADR-IDENTITY-011	Implementación RBAC + ABAC
ADR-IDENTITY-026	Múltiples roles por Membership
ADR-IDENTITY-027	Role hierarchy
ADR-IDENTITY-028	Permission inheritance
ADR-IDENTITY-029	Role lifecycle
ADR-IDENTITY-030	Permission lifecycle
ADR-IDENTITY-031	Authorization caching
ADR-IDENTITY-032	Policy engine
ADR-IDENTITY-033	Platform Admin vs Organization Admin
64. Implementation Readiness

Tenemos ahora:

ESP-0001 — Identity
│
├── 01 — Identity Domain Model
├── 02 — Identity Architecture
├── 03 — Authentication & Authorization
├── 04 — User & Organization Management
└── 05 — Roles & Permissions

La arquitectura empieza a tomar esta forma:

                         IDENTITY
                            │
            ┌───────────────┼───────────────┐
            ▼               ▼               ▼
           USER       ORGANIZATION       SESSION
            │               │
            └───────┬───────┘
                    ▼
               MEMBERSHIP
                    │
                    ▼
                  ROLE
                    │
                    ▼
              PERMISSION
                    │
                    ▼
                   RBAC
                    │
                    ▼
                   ABAC
                    │
          ┌─────────┴─────────┐
          ▼                   ▼
      TENANT POLICY       OWNERSHIP
          │                   │
          └─────────┬─────────┘
                    ▼
             AUTHORIZATION
                    │
              ┌─────┴─────┐
              ▼           ▼
            ALLOW        DENY
Estado

ESP-0001/05 — 🟡 DRAFT

La parte conceptual ya está bastante avanzada, pero todavía no debemos congelar el modelo físico.

El siguiente paso natural sería:

ESP-0001 / 06 — Identity Sessions & Token Management

Ahí definiremos en profundidad:

Session
Access Token
Refresh Token
JWT
Token Claims
Token Expiration
Refresh Rotation
Token Revocation
Session Revocation
Concurrent Sessions
Device Sessions
Logout
Global Logout
Token Theft
Refresh Token Reuse
Session Security

y después podremos avanzar hacia MFA, API Tokens, Audit, Persistence y finalmente API Contracts.
