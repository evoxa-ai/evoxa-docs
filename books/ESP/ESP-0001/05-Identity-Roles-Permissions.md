ESP-0001 / 05 — Identity Roles & Permissions

Document ID: ESP-0001/05
Module: Identity
Version: 1.1.0
Status: Draft — Consolidated
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

Este documento define el modelo de Roles & Permissions de Identity.

Su objetivo es establecer cómo una identidad obtiene capacidades autorizables dentro de un determinado contexto organizacional y bajo las políticas de seguridad aplicables.

Comprende:

Role
Permission
RolePermission
Membership Roles
RBAC
ABAC
Tenant Authorization
Ownership
Delegation
Role Assignment
Role Revocation
Permission Lifecycle
Role Lifecycle
Authorization Context
Policy Integration
Criticality
Freshness
Cache Eligibility
Audit
Security Events

La autorización completa de EVOXA no se basa exclusivamente en RBAC.

El modelo consolidado es:

RBAC
+
ABAC
+
Tenant Isolation
+
Ownership
+
Delegation
+
Security Policy Engine
+
Criticality
+
Freshness
2. Fundamental Authorization Model

La relación básica es:

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

Pero esto no constituye por sí solo una autorización final.

El modelo completo es:

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
  ├── Tenant
  ├── Ownership
  ├── ABAC
  ├── Delegation
  └── Policy
        │
        ▼
   Criticality
        │
        ▼
    Freshness
        │
        ▼
Authorization Decision

Por tanto:

Role
    ≠
Permission
    ≠
Authorization Decision
3. Relationship with Membership

Los roles organizacionales están asociados al contexto de Membership.

El modelo consolidado es:

User
 │
 ├── Membership A
 │      ├── Organization A
 │      └── Roles
 │
 └── Membership B
        ├── Organization B
        └── Roles

Ejemplo:

User U123

Membership → Organization A
    ├── Coach
    └── TeamManager

Membership → Organization B
    └── Nutritionist

Esto significa que el mismo User puede tener capacidades diferentes dependiendo de la Organization activa.

4. Role

Un Role representa un conjunto lógico de responsabilidades y capacidades.

Ejemplos de roles profesionales:

Coach
Nutritionist
Physiotherapist
Team Manager

También pueden existir roles administrativos de plataforma.

Pero:

Role

no debe interpretarse automáticamente como:

global permission
5. Role Scope

El modelo consolidado soporta dos scopes principales:

SYSTEM
ORGANIZATION
SYSTEM

Role aplicable a nivel de plataforma.

Ejemplo conceptual:

PlatformAdmin
PlatformOperator
PlatformSupport
ORGANIZATION

Role limitado a una Organization mediante Membership.

Ejemplo:

Coach
Nutritionist
Physiotherapist
TeamManager
OrganizationAdmin

La diferencia fundamental es:

SYSTEM
   ↓
Platform Security Boundary

ORGANIZATION
   ↓
Membership / Tenant Boundary
6. Role Context

Un role organizacional debe evaluarse dentro de una Membership.

Correcto:

User
 ↓
Membership
 ↓
Organization A
 ↓
Role = Coach

Incorrecto:

User
 ↓
Role = Coach

sin contexto organizacional.

Por tanto:

Coach no significa que el usuario sea Coach globalmente.

Significa que posee ese role dentro del contexto autorizado correspondiente.

7. Multiple Roles per Membership

Esta es una decisión que el documento original todavía dejaba pendiente.

Ya no debemos mantenerla como pendiente.

El modelo consolidado permite:

Membership
   │
   ├── Role A
   ├── Role B
   └── Role C

Ejemplo:

User
 └── Organization A
       ├── Coach
       └── TeamManager

La asignación se representa conceptualmente mediante:

MembershipRole

o equivalente.

8. No Implicit Role Hierarchy

No existe herencia implícita.

Por defecto:

Admin
   ↓
Manager
   ↓
Coach

NO significa que Admin herede automáticamente todos los permisos de Manager y Coach.

Cada role debe obtener sus permisos explícitamente.

Esto evita privilegios implícitos difíciles de auditar.

9. Role Hierarchy

La jerarquía de roles no forma parte del modelo inicial.

Si en el futuro se introduce:

Role
   │
   ▼
Parent Role

deberá ser una capacidad explícita y gobernada.

No se permite que una implementación introduzca role inheritance implícitamente.

10. Permission

Una Permission representa una capacidad autorizable.

El modelo conceptual es:

Permission
├── resource
└── action

Ejemplo:

resource = training-programs
action   = update
11. Permission Naming

El estándar conceptual consolidado es:

<resource>.<action>

Ejemplos:

users.read
users.create
users.update
users.delete

organizations.read
organizations.update

memberships.read
memberships.create
memberships.remove

training-programs.read
training-programs.create
training-programs.update
training-programs.delete

La nomenclatura debe ser:

explícita;
estable;
allowlisted;
auditable;
compatible con Policy Engine.
12. Actions

El catálogo puede incluir acciones como:

read
create
update
delete
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

Pero un Resource no debe asumir automáticamente que todas las acciones existen.

Ejemplo:

training-programs.publish

puede existir aunque:

training-programs.import

no exista.

El catálogo oficial Resource/Action será una especificación gobernada.

13. Resource Registry

Identity/Authorization debe mantener un concepto de:

Resource Registry

y:

Action Registry

Estos registros permiten validar que:

training-programs.update

sea una combinación conocida.

No debe permitirse que un cliente invente arbitrariamente:

super-admin.everything

y la convierta en una Permission válida.

14. RolePermission

RolePermission representa:

Role
  │
  ▼
RolePermission
  │
  ▼
Permission

Ejemplo:

Coach
 ├── clients.read
 ├── clients.update
 ├── training-programs.read
 ├── training-programs.create
 └── training-programs.update

Debe existir una restricción conceptual:

UNIQUE(role_id, permission_id)

para impedir duplicados.

15. Permission Assignment

La asignación de Permissions ocurre normalmente mediante Role:

Role
  │
  ▼
RolePermission
  │
  ▼
Permission

Esto permite mantener RBAC administrable y auditable.

La asignación directa de Permission a User no forma parte del modelo base salvo que una futura capacidad explícita la introduzca.

16. RBAC

RBAC determina si el actor posee una Permission mediante sus roles.

User
 ↓
Membership
 ↓
Role(s)
 ↓
Permission(s)

Ejemplo:

Coach
   ↓
training-programs.update

Si el role no contiene:

training-programs.update

RBAC no concede la capacidad.

17. RBAC Is Not Final Authorization

Tener una Permission no implica automáticamente ALLOW.

Ejemplo:

Coach
 ↓
training-programs.update

puede producir:

RBAC = ALLOW

pero posteriormente:

Tenant = DENY

o:

Ownership = DENY

o:

Policy = DENY

Por lo tanto:

RBAC ALLOW
      +
ABAC DENY
      =
FINAL DENY
18. ABAC

ABAC agrega contexto.

Conceptualmente:

Actor
+
Organization
+
Membership
+
Role
+
Permission
+
Resource
+
Resource State
+
Ownership
+
Risk
+
Security State
+
Policy

El resultado alimenta Authorization Runtime.

19. Ownership

Ownership constituye una dimensión independiente.

Ejemplo:

Coach A
 └── Client 1

Coach B
 └── Client 2

Ambos pueden poseer:

clients.read

pero eso no implica:

Coach A → Client 2

La ownership rule puede producir:

DENY

aunque RBAC permita la acción.

20. Tenant Isolation

Una Permission no concede acceso fuera de su tenant.

Por ejemplo:

training-programs.update

no significa:

Update ANY training program

Significa:

El actor puede intentar actualizar recursos de ese tipo sujeto a tenant, ownership, ABAC y Policy.

21. Cross-Tenant Protection

Ejemplo:

User U123
Organization A
Role Coach
Permission training-programs.update

intenta:

Training Program
Organization B

Evaluación:

RBAC
   ↓
ALLOW

Tenant Guard
   ↓
DENY

FINAL
   ↓
DENY

Tenant isolation tiene prioridad de seguridad.

22. Platform vs Organization Roles

Debe distinguirse:

Platform Role

de:

Organization Role

Ejemplo:

PlatformAdmin

no equivale a:

OrganizationAdmin

Una Organization Admin puede administrar su tenant sin obtener automáticamente:

Platform Security
Platform Users
Other Organizations
Platform Policies
23. Organization Administration

Permissions típicas:

organization.read
organization.update

memberships.read
memberships.create
memberships.suspend
memberships.remove

roles.read
roles.assign
roles.revoke

Estas permissions están sujetas al Organization Context.

24. Platform Administration

Las capabilities de plataforma pueden utilizar Permissions específicas:

platform.users.manage
platform.organizations.manage
platform.security.manage

El scope debe ser:

SYSTEM

y no simplemente una Organization Membership.

25. Role Assignment

Flujo consolidado:

Role Assignment Request
          │
          ▼
Authentication
          │
          ▼
Tenant Context
          │
          ▼
Authorization
          │
          ▼
Validate Target Membership
          │
          ▼
Validate Role
          │
          ▼
Assign Role
          │
          ├── Audit
          ├── Security Event
          ├── Security Epoch / Version
          └── Cache Invalidation

El actor que asigna el Role también debe estar autorizado para hacerlo.

26. Privilege Boundary

Regla fundamental:

Un actor no puede concederse a sí mismo privilegios que no posee.

Por ejemplo:

Coach
   ↓
POST assign PlatformAdmin
   ↓
DENY

También:

Coach
   ↓
assign training.admin
   ↓
DENY

salvo que una autorización explícita permita esa operación.

27. Delegated Role Assignment

Cuando exista Delegation:

Delegator
   ↓
Delegation
   ↓
Delegate

la capacidad delegada nunca puede superar la capacidad efectiva del delegador.

Regla:

Delegated Capability
        ≤
Delegator Effective Capability

No se permite privilege amplification.

28. Role Revocation

Flujo:

Role Assignment
      │
      ▼
Revoke
      │
      ├── Update Authorization State
      ├── Security Epoch / Version
      ├── Cache Invalidation
      ├── Audit
      └── Security Event

La revocación debe propagarse según las garantías de freshness definidas por Authorization.

29. Permission Revocation

Ejemplo:

Antes:

Coach
 ├── training.read
 ├── training.create
 └── training.update

Después:

Coach
 ├── training.read
 └── training.create

Una nueva:

training.update

debe producir:

DENY

No debe depender únicamente de TTL.

30. Authorization Cache

El sistema puede utilizar:

L1 Cache
L2 Redis

para optimizar la evaluación.

Pero:

Cache ≠ Source of Truth

La fuente autoritativa sigue siendo PostgreSQL y el estado normativo de Identity/Authorization.

31. Cache Invalidation

Cambios como:

RoleAssigned
RoleRevoked
PermissionAssigned
PermissionRevoked
MembershipRevoked
SecurityStateChanged
PolicyChanged

pueden requerir invalidación.

Modelo:

State Change
     │
     ▼
PostgreSQL Transaction
     +
Transactional Outbox
     │
     ▼
Event Platform
     │
     ▼
Invalidation Consumer
     │
     ├── L1
     └── L2

Esto sigue ADR-053.

32. Security Epoch

La invalidación no debe depender exclusivamente de borrar físicamente una entrada.

Debe existir una estrategia basada en:

Security Epoch
+
Entity Version
+
Policy Version
+
Context Fingerprint
+
Generation

Una entrada antigua debe considerarse inválida aunque físicamente todavía exista.

33. Criticality

La autorización de Roles & Permissions debe integrarse con el modelo de Criticality.

La fórmula consolidada es:

Effective Criticality =
MAX(
    Base,
    Dynamic Risk,
    Context,
    Security State,
    Tenant,
    Delegation,
    MFA,
    Platform Minimum
)

La Criticality no concede autorización.

Criticality
    ≠
Authorization Decision

Determina el nivel de seguridad requerido para evaluar/usar la autorización.

34. Criticality Levels
C0 — Critical Security
C1 — High Security
C2 — Standard
C3 — Low Risk

Relación con Freshness:

C0 → F0
C1 → F1
C2 → F2-SHORT
C3 → F2-MEDIUM

Por tanto, una modificación de Role/Permission de alto impacto puede requerir una evaluación más fresca.

35. Freshness

Clases:

F0 — Authoritative
F1 — Security Validated
F2 — Controlled Eventual
F3 — Restricted Degraded

Una revocación de Permission o Role no debe quedar autorizada indefinidamente por una entrada cacheada antigua.

Las revocaciones tienen prioridad sobre TTL.

36. Authorization Runtime

Roles & Permissions se integran en el Authorization Runtime definido anteriormente:

Authorization Request
        ↓
Request Validation
        ↓
Authentication Validation
        ↓
Tenant Guard
        ↓
Context Construction
        ↓
Policy Selection
        ↓
Policy Evaluation
        ↓
Mandatory Security Guards
        ↓
RBAC
        ↓
Ownership
        ↓
ABAC
        ↓
Delegation
        ↓
Policy Composition
        ↓
Conflict Resolution
        ↓
Criticality
        ↓
Freshness
        ↓
Cache Eligibility
        ↓
Authorization Decision

Por tanto, este ESP no crea un segundo Authorization Engine.

37. Policy Engine Integration

La Policy Engine permanece centralizada.

Roles & Permissions proporciona información para la evaluación:

Roles
Permissions
Membership
Organization

pero la Policy Engine puede imponer condiciones adicionales.

Ejemplo:

Role = Coach
Permission = training.update

pero:

Policy:
resource.status == DRAFT

Si:

resource.status == PUBLISHED

la Policy puede producir:

DENY
38. Policy Effects

La Policy Engine puede producir:

ALLOW
DENY
CHALLENGE
REAUTHENTICATE
REQUIRE_MFA
REVOKE

Por tanto, Roles & Permissions no debe reducir la autorización a:

true / false
39. Explicit Deny

El modelo soporta explicit deny.

Ejemplo:

Role
   ↓
ALLOW

Policy
   ↓
DENY

Final
   ↓
DENY

La composición debe seguir las reglas del Authorization Runtime y Policy Engine.

40. Fail Closed

Si no puede determinarse de forma segura:

Role
Permission
Membership
Tenant
Policy
Security State

no se debe asumir:

ALLOW

El resultado debe ser seguro:

DENY

o el efecto de seguridad explícitamente requerido por el runtime.

41. Role Lifecycle

El lifecycle conceptual queda:

DRAFT
   ↓
ACTIVE
   ↓
DISABLED

Un Role que haya sido utilizado históricamente no debe eliminarse de forma que destruya la trazabilidad histórica.

La estrategia de persistence/retention se definirá en ESP-0001/11.

42. Permission Lifecycle

El lifecycle conceptual:

DRAFT
   ↓
ACTIVE
   ↓
DEPRECATED
   ↓
RETIRED

Esto permite evolucionar el catálogo sin destruir referencias históricas.

43. Permission Evolution

Las modificaciones de una Permission deben distinguir entre:

Metadata Change

y:

Security Semantic Change

Una modificación que cambie el significado de una Permission puede requerir una nueva versión o revisión gobernada.

No se debe modificar silenciosamente el significado de:

training.update

de manera que cambie las garantías de seguridad.

44. Role Changes and Security Propagation

Una modificación de Role puede afectar a múltiples usuarios:

Role
 ↓
Permissions
 ↓
Memberships
 ↓
Users

Por ello:

Role Change

puede provocar:

Authorization Cache Invalidation
Security Epoch Update
Audit
Security Event

según el alcance.

45. Authorization Context

El contexto conceptual:

AuthorizationContext
├── actor
├── user_id
├── organization_id
├── membership_id
├── roles
├── permissions
├── resource
├── action
├── ownership
├── delegation
├── session
├── MFA
├── risk
├── security_state
├── environment
└── timestamp

Debe permanecer inmutable durante la evaluación.

46. Authorization Decision

La salida debe contener conceptualmente:

AuthorizationDecision
├── decision
├── reason_code
├── criticality
├── freshness
├── cacheability
├── policy_id
├── policy_version
└── obligations

Decisiones:

ALLOW
DENY
CHALLENGE
REAUTHENTICATE
REQUIRE_MFA
REVOKE
47. Audit

Toda modificación de autorización debe poder responder:

WHO
WHAT
WHO WAS AFFECTED
ROLE/PERMISSION
ORGANIZATION
WHEN
RESULT
CORRELATION

Ejemplo:

RoleAssigned
{
    actor_id,
    user_id,
    membership_id,
    organization_id,
    role_id,
    timestamp,
    correlation_id
}

No deben incluirse secretos.

48. Security Events

Eventos conceptuales:

RoleCreated
RoleUpdated
RoleActivated
RoleDisabled

RoleAssigned
RoleRevoked

PermissionCreated
PermissionUpdated
PermissionDeprecated
PermissionRetired

PermissionAssigned
PermissionRevoked

Los eventos de autorización deben utilizar el envelope común de Event Architecture.

49. Authorization Denied Telemetry

Los rechazos importantes deben ser observables:

authorization.denied
authorization.allowed
authorization.latency
authorization.policy_failure
authorization.cache_hit
authorization.cache_miss
authorization.freshness_violation

Pero la telemetría no debe revelar información sensible innecesaria.

50. API Boundary

Las APIs conceptuales incluyen:

GET    /api/v1/roles
POST   /api/v1/roles
GET    /api/v1/roles/{id}
PATCH  /api/v1/roles/{id}

GET    /api/v1/permissions
GET    /api/v1/permissions/{id}

POST   /api/v1/roles/{id}/permissions
DELETE /api/v1/roles/{id}/permissions/{permissionId}

POST   /api/v1/memberships/{id}/roles
DELETE /api/v1/memberships/{id}/roles/{roleId}

Importante: estos son límites conceptuales, no contratos API definitivos.

Los contratos oficiales corresponden a:

ESP-0001/10 — Identity API Contracts
51. Persistence Model

Modelo conceptual:

users
organizations
memberships

roles
permissions
role_permissions

membership_roles

Relaciones:

Membership
     │
     ▼
MembershipRole
     │
     ▼
Role
     │
     ▼
RolePermission
     │
     ▼
Permission

El schema físico definitivo corresponde a ESP-0001/11.

52. Domain Boundary

Identity define el mecanismo de autorización.

Pero cada dominio conserva sus reglas específicas.

Ejemplo:

Identity
   ↓
training-programs.update

Training determina:

¿El programa pertenece al tenant?
¿El usuario es owner?
¿Está publicado?
¿Puede modificarse?
¿Existe una regla específica del negocio?

Por tanto:

Identity
    =
Security Authorization Framework

Business Domain
    =
Business Rules
53. Example — Coach
User:
U123

Organization:
ORG-A

Membership:
M456

Role:
Coach

Permissions:

clients.read
clients.update

training-programs.read
training-programs.create
training-programs.update

Request:

PATCH /api/v1/training-programs/456

Evaluación:

Authentication
      ↓
U123
      ↓
ORG-A
      ↓
Membership M456
      ↓
Coach
      ↓
training-programs.update
      ↓
Tenant
      ↓
Ownership
      ↓
Policy
      ↓
Criticality
      ↓
Freshness
      ↓
Decision
54. Example — Cross Tenant
User U123
Organization A
Role Coach
Permission training-programs.update

intenta modificar:

Training Program 456
Organization B

Resultado:

RBAC
  ↓
ALLOW

Tenant Guard
  ↓
DENY

FINAL
  ↓
DENY

Aunque la Permission exista.

55. Example — Permission Missing

Coach:

clients.read
training-programs.read

Request:

DELETE /api/v1/clients/123

Evaluación:

clients.delete
      ↓
Permission not present
      ↓
DENY
56. Example — Role Revoked

Antes:

Coach
 └── training-programs.update

Después:

Coach
 └── training-programs.read

Nueva solicitud:

training-programs.update

debe producir:

DENY

aunque exista una entrada antigua en cache.

57. Example — Privilege Escalation
Coach
   │
   ▼
Assign Role = PlatformAdmin

La evaluación:

Actor Capability
      <
Requested Capability

produce:

DENY

No se permite privilege amplification.

58. Security Requirements

Roles & Permissions deben cumplir:

Least Privilege
Zero Trust
Secure by Default
Tenant Isolation
Explicit Authorization
No Privilege Escalation
No Cross-Tenant Access
Auditability
Fail Closed
Policy Enforcement
59. AI Authorization

Los AI Agents utilizan exactamente el mismo modelo:

AI Agent
   ↓
Identity
   ↓
Membership / Service Identity
   ↓
Role / Permission
   ↓
Policy
   ↓
Tenant
   ↓
Ownership
   ↓
Criticality
   ↓
Freshness
   ↓
Decision

Un AI Agent no puede:

bypass authorization
grant itself permissions
reduce criticality
disable MFA
manipulate authorization cache
activate security policies
approve security regressions
60. Testing

La matriz mínima debe incluir:

RBAC
Role has permission
Role lacks permission
Multiple roles
Role revoked
Permission revoked
Tenant
Correct tenant
Wrong tenant
Cross-tenant
Missing membership
Suspended membership
Ownership
Correct owner
Wrong owner
Ownership changed
Policy
Policy allows
Policy denies
Policy requires MFA
Policy requires reauthentication
Policy unavailable
Policy indeterminate
Privilege
Self-role escalation
Unauthorized role assignment
Unauthorized permission assignment
Delegation amplification
Cross-tenant delegation
Cache
Fresh cache
Expired cache
Stale ALLOW
Stale DENY
Role revocation invalidation
Permission revocation invalidation
Out-of-order invalidation
Duplicate invalidation
61. Critical Security Invariants

Este ESP debe respetar, entre otros:

INV-ROLE-001
No Privilege Amplification

INV-ROLE-002
Tenant Isolation

INV-ROLE-003
No Implicit Role Hierarchy

INV-ROLE-004
No Permission Without Valid Scope

INV-ROLE-005
Revocation Priority

INV-ROLE-006
Unknown Is Not Allow

INV-ROLE-007
Error Is Not Allow

INV-ROLE-008
Cache Is Not Source of Truth

INV-ROLE-009
Criticality Cannot Grant Permission

INV-ROLE-010
AI Cannot Reduce Security

Estos deben integrarse con los invariants generales de ADR-062/063/067.

62. Architectural Components

La estructura propuesta:

identity/
│
├── domain/
│   ├── roles/
│   │   ├── entities/
│   │   │   └── role.py
│   │   ├── value_objects/
│   │   └── events/
│   │
│   ├── permissions/
│   │   ├── entities/
│   │   │   └── permission.py
│   │   ├── value_objects/
│   │   └── events/
│   │
│   └── authorization/
│
├── application/
│   ├── roles/
│   ├── permissions/
│   └── assignments/
│
├── api/
│   ├── roles/
│   └── permissions/
│
└── infrastructure/
    ├── persistence/
    ├── cache/
    └── messaging/

Esto es Derived Design y debe mantenerse sujeto a ESP-0001/02 y ADR-068/069.

63. Traceability
Capability	Referencia
Identity Domain	ESP-0001/01
Architecture	ESP-0001/02
Authentication/Authorization	ESP-0001/03
User/Organization/Membership	ESP-0001/04
RBAC	ADR-005/044/045
ABAC	ADR-005/045–049
Delegation	ADR-044
Policy Model	ADR-045–049
Policy DSL	ADR-048
Authorization Runtime	ADR-049
Authorization Cache	ADR-050–057
Criticality	ADR-058/059
Criticality Testing	ADR-062/063
Test Contracts	ADR-064–067
Identity Boundaries	ADR-068
Module Dependencies	ADR-069
API	ESP-0001/10
Persistence	ESP-0001/11
Events	ESP-0001/12
Testing	ESP-0001/13
64. Corrección importante respecto al ESP original

El documento actual todavía contiene una sección de ADR Candidates con referencias como:

ADR-IDENTITY-003
ADR-IDENTITY-004
ADR-IDENTITY-011
ADR-IDENTITY-026
ADR-IDENTITY-027
ADR-IDENTITY-028
ADR-IDENTITY-029
ADR-IDENTITY-030
ADR-IDENTITY-031
ADR-IDENTITY-032
ADR-IDENTITY-033

Estas referencias no deben conservarse en la versión consolidada, porque esos números actualmente corresponden a decisiones diferentes del catálogo real.

Por ejemplo, muchas decisiones que el ESP original marcaba como pendientes ya quedaron resueltas por:

ADR-044
Membership Roles / Delegation

ADR-045
Authorization Policy Model

ADR-046
Policy Language / Governance

ADR-047
Policy Engine Implementation

ADR-048
Policy DSL

ADR-049
Authorization Runtime

ADR-050–057
Authorization Cache / Freshness

ADR-058
Criticality Matrix

ADR-059
Dynamic Criticality

ADR-062–067
Criticality Test Governance

ADR-068
Identity Module Boundaries

ADR-069
Module Dependency Matrix
65. Implementation Readiness
Área	Estado
Role model	🟢
Permission model	🟢
Role scope	🟢
Multiple roles per Membership	🟢
Membership role assignment	🟢
RBAC	🟢
ABAC integration	🟢
Tenant isolation	🟢
Ownership	🟢
Delegation	🟢
Platform vs Organization roles	🟢
Role hierarchy	🟢 — explícitamente no implícita
Permission inheritance	🟢 — no implícita
Role lifecycle	🟢 conceptual
Permission lifecycle	🟢 conceptual
Policy integration	🟢
Criticality	🟢
Freshness	🟢
Cache integration	🟢
Audit	🟢
Security Events	🟢
API contracts	⏳ ESP-10
Persistence schema	⏳ ESP-11
Event contracts	⏳ ESP-12
Full test implementation	⏳ ESP-13
66. Resultado

La arquitectura de autorización de EVOXA queda ahora:

                         USER
                           │
                           ▼
                     MEMBERSHIP
                           │
                 ┌─────────┴─────────┐
                 │                   │
             ORGANIZATION          ROLES
                                     │
                                     ▼
                               PERMISSIONS
                                     │
                                     ▼
                                    RBAC
                                     │
                    ┌────────────────┼────────────────┐
                    ▼                ▼                ▼
                  ABAC           OWNERSHIP        DELEGATION
                    │                │                │
                    └────────────────┼────────────────┘
                                     ▼
                              POLICY ENGINE
                                     │
                                     ▼
                                CRITICALITY
                                     │
                                     ▼
                                  FRESHNESS
                                     │
                                     ▼
                              CACHE ELIGIBILITY
                                     │
                                     ▼
                         AUTHORIZATION DECISION
Evaluación

ESP-0001/05 original: 🟡 DRAFT

ESP-0001/05 v1.1.0 consolidado: 🟢 arquitectónicamente alineado.

La diferencia más importante es que ya no estamos diseñando RBAC como un sistema aislado. Roles y Permissions pasan a ser una parte del Authorization Runtime completo, respetando Membership, Tenant Isolation, ABAC, Ownership, Delegation, Policy Engine, Criticality, Freshness y Cache.

Además, no necesitamos crear nuevos ADR simplemente para resolver las decisiones antiguas de este ESP. El catálogo ADR-001 → ADR-069 ya contiene esas decisiones.

El siguiente paso lógico es:

ESP-0001/06 — Identity Sessions & Token Management

y ahí debemos hacer exactamente lo mismo: tomar el documento actual, compararlo con ADR-002, 003, 008, 009, 025, 027, 029–038, 054–059 y 067, eliminar las antiguas decisiones pendientes que ya fueron resueltas y dejar Sessions + Access JWT + Refresh Token + Rotation + Reuse Detection + Revocation completamente listos para implementación.
