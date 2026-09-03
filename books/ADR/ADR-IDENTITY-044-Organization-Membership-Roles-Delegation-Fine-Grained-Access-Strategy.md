ADR-IDENTITY-044 — Organization Membership Roles, Delegation & Fine-Grained Access Strategy

Status: Proposed
Version: 1.0
Date: 2026-09-03
Domain: Identity / Authorization
Priority: Critical
Type: Architecture / Security / Access Control

1. Context

El ADR-IDENTITY-043 estableció que EVOXA utiliza:

Global User + Organization-Scoped Membership + Organization-Scoped Authorization

Por lo tanto, la relación del usuario con una Organization constituye la frontera principal de acceso organizacional.

Sin embargo, Membership por sí sola no determina qué puede hacer un usuario.

Dentro de una misma Organization pueden existir usuarios con diferentes responsabilidades:

Organization
│
├── Administrator
├── Coach
├── Nutritionist
├── Physiotherapist
└── Team Manager

Además, determinados escenarios requieren capacidades más precisas que un simple Role.

Por ejemplo:

Coach
 ├── Puede ver atletas
 ├── Puede crear entrenamientos
 ├── Puede modificar sus propios planes
 └── No puede administrar facturación

Mientras otro usuario podría tener:

Team Manager
 ├── Puede ver equipos
 ├── Puede gestionar miembros
 ├── Puede consultar estadísticas
 └── No puede modificar planes clínicos

También existirán escenarios donde un usuario necesite acceso temporal o específico a un recurso sin convertirse en miembro general de otra Organization.

Por ello se requiere combinar:

Membership;
Roles;
Permissions;
Resource Ownership;
Delegation;
ABAC;
Security Policy Engine;
Tenant Isolation.
2. Problem Statement

Se necesita definir una estrategia que permita:

Asignar roles a Memberships.
Derivar permissions desde roles.
Permitir múltiples roles por Membership.
Soportar permisos granulares.
Aplicar ownership.
Soportar delegaciones explícitas.
Soportar acceso temporal.
Evitar privilege escalation.
Evitar cross-tenant leakage.
Mantener autorización centralizada.
Permitir políticas contextuales.
Mantener trazabilidad y auditoría.
Soportar AI Agents con scopes limitados.

La solución no debe convertir RBAC en un sistema excesivamente rígido, pero tampoco permitir que ABAC o delegaciones creen permisos arbitrarios fuera del modelo de seguridad de EVOXA.

3. Decision

EVOXA adoptará un modelo híbrido:

RBAC + ABAC + Resource Ownership + Explicit Delegation + Security Policy Engine + Tenant Isolation

La autorización seguirá conceptualmente:

User
  ↓
Membership
  ↓
Organization
  ↓
Roles
  ↓
Permissions
  ↓
Resource Ownership / Attributes
  ↓
Security Policy
  ↓
Authorization Decision

La decisión final podrá ser:

ALLOW
DENY
CHALLENGE
REAUTHENTICATE
REQUIRE_MFA
REVOKE
4. Membership as Authorization Anchor

La Membership será el ancla para roles organizacionales.

Conceptualmente:

Membership
├── User
├── Organization
├── Status
└── Roles

Un Role organizacional no deberá asignarse directamente al User de forma global.

Correcto:

User
 ↓
Membership Org A
 ↓
Role Coach

No:

User
 ↓
Global Role Coach
5. Multiple Roles per Membership

Una Membership podrá tener múltiples roles.

Ejemplo:

User
└── Org A
     └── Membership
          ├── Coach
          └── Team Manager

Las permissions efectivas serán la combinación de los roles autorizados, sujetas a las políticas aplicables.

Conceptualmente:

Effective Permissions
=
Union(Role Permissions)
+
ABAC Constraints
+
Ownership
+
Security Policy

La unión de roles no debe permitir superar restricciones de seguridad de mayor precedencia.

6. Role Scope

Los roles tendrán un scope explícito.

Conceptualmente:

SYSTEM
ORGANIZATION
RESOURCE

El modelo mínimo requerido será:

SYSTEM

Capacidades de plataforma.

ORGANIZATION

Capacidades dentro de una Organization.

RESOURCE

Cuando un futuro modelo requiera roles asociados a un recurso o subconjunto de recursos.

No se adoptará RESOURCE como requisito inicial para todos los módulos; será una capacidad extensible.

7. Organization Role

Un Organization Role aplica dentro de una Organization.

Ejemplo:

Org A
 └── Membership
      └── Coach

El mismo usuario puede tener:

Org B
 └── Membership
      └── Nutritionist

Por tanto:

Role(User, Org A)
≠
Role(User, Org B)
8. Role Hierarchy

No se asumirá una jerarquía implícita como:

Admin > Manager > Coach

La existencia de un nombre como Administrator no debe automáticamente otorgar todas las capabilities de otros roles.

Si EVOXA necesita jerarquías de roles, estas deberán ser explícitas y gobernadas.

Por defecto:

Roles are composable capabilities, not implicit organizational rank.

9. Permission Model

Las permissions utilizarán una nomenclatura conceptual:

<resource>.<action>

Ejemplos:

athletes.read
athletes.create
athletes.update
athletes.delete

training_plans.read
training_plans.create
training_plans.update
training_plans.delete

teams.read
teams.manage

memberships.read
memberships.manage

La catalogación definitiva de resources/actions será responsabilidad de las Engineering Specifications correspondientes.

10. Permission ≠ Role

Se mantiene la separación:

Role
 ↓
Permission

Ejemplo:

Coach
 ├── athletes.read
 ├── training_plans.read
 ├── training_plans.create
 └── training_plans.update

El Role es una agrupación administrable.

La Permission es la capability concreta evaluada por Authorization.

11. Permission ≠ Scope

Se mantienen separados:

OAuth/API Scope

Define lo que un cliente/token puede solicitar.

EVOXA Permission

Define lo que el actor puede realizar.

Por ejemplo:

OAuth Scope:
training

EVOXA Permissions:
training_plans.read
training_plans.create
training_plans.update

Tener un scope OAuth no implica automáticamente todas las permissions internas.

12. Fine-Grained Authorization

RBAC será complementado con condiciones contextuales.

Ejemplo:

Coach
+
training_plans.update
+
resource.owner_id == current_user

Resultado:

ALLOW

Pero:

Coach
+
training_plans.update
+
resource.owner_id != current_user

puede resultar:

DENY

según la policy.

13. Resource Ownership

Ownership será una dimensión independiente del Role.

Un usuario puede tener:

training_plans.update

pero no necesariamente:

training_plans.update

sobre cualquier training plan de la Organization.

El sistema podrá evaluar:

resource.owner_id
resource.organization_id
resource.team_id
resource.created_by

según el dominio.

14. Ownership Models

Se consideran varios modelos:

User-Owned
Resource.owner = User
Team-Owned
Resource.owner = Team
Organization-Owned
Resource.owner = Organization
Shared
Resource
 └── Authorized Principals

El dominio específico determinará qué modelo aplica.

15. Ownership Does Not Override Deny

Ownership por sí solo no concede acceso.

Ejemplo:

User owns Resource
+
User suspended

Resultado:

DENY

De igual forma:

User owns Resource
+
Organization suspended

no implica acceso.

La autorización siempre debe respetar las restricciones de mayor precedencia.

16. Delegation

EVOXA soportará Explicit Delegation como mecanismo independiente de Membership.

Una delegación permite:

Principal A
   ↓
Delegates
   ↓
Principal B
   ↓
Specific Capability
   ↓
Specific Resource / Scope

Ejemplo:

Coach A
   ↓
delegates
   ↓
Coach B
   ↓
training_plans.update
   ↓
Team X

Esto no convierte automáticamente a B en administrador ni en propietario del recurso.

17. Delegation Principles

Toda delegación deberá ser:

explícita;
limitada;
autorizada;
auditable;
revocable;
opcionalmente temporal;
tenant-aware;
policy-aware.

La delegación no puede ampliar arbitrariamente los privilegios del delegante.

18. No Privilege Amplification

Se establece:

A principal cannot delegate authority it does not possess.

Ejemplo:

Coach

no puede delegar:

organization.memberships.manage

si no posee esa permission.

Además:

Delegated Permission
≤
Delegator Effective Permission

sujeto a policy.

19. Delegation Scope

Una delegación podrá limitarse por:

Organization
Team
Resource
Action
Time
Context

Ejemplo:

Organization = Org A
Team = Team 10
Action = training_plans.read
Expires = ...

La delegación no debe convertirse en permiso global.

20. Temporary Delegation

Las delegaciones podrán tener:

starts_at
expires_at

Esto permitirá casos como:

reemplazo temporal;
vacaciones;
cobertura de equipo;
soporte;
colaboración puntual.

Una delegación expirada debe resultar en:

DENY

sin requerir necesariamente una operación manual de cleanup.

21. Delegation Revocation

El delegante o una autoridad superior podrá revocar una delegación cuando la policy lo permita.

Estados conceptuales:

PENDING
ACTIVE
EXPIRED
REVOKED

La revocación debe ser inmediatamente considerada por Authorization según las garantías de consistencia del sistema.

22. Delegation Chains

No se permitirá por defecto:

A delegates B
B delegates C
C delegates D

porque puede producir privilege amplification difícil de controlar.

Por defecto:

Delegation depth = 1

Si posteriormente se requiere delegación encadenada, deberá existir un ADR específico.

23. Delegation Across Organizations

Cross-tenant delegation estará:

DENY by default

Una delegación desde:

Org A

hacia:

Org B

requerirá:

explicit cross-tenant authorization;
source organization;
target organization;
policy;
limited scope;
audit.

No será suficiente que ambos usuarios sean miembros de EVOXA.

24. Resource Sharing vs Delegation

Se distinguen:

Resource Sharing

El propietario comparte un recurso.

Delegation

Un principal otorga temporalmente una capacidad.

Ejemplo:

Sharing:
"User B can view Resource X"

Delegation:
"User B may perform action Y on Resource X"

El modelo final puede combinar ambos conceptos, pero no deben confundirse.

25. ABAC

ABAC permitirá evaluar atributos del:

Subject
user
role
membership
organization
department
risk
device
Resource
organization
owner
team
classification
status
Action
read
create
update
delete
approve
export
Environment
time
location
network
authentication_assurance
risk
26. Example ABAC Rule

Conceptualmente:

ALLOW
IF

membership.status == ACTIVE
AND
role includes Coach
AND
permission == training_plans.update
AND
resource.organization_id == membership.organization_id
AND
resource.team_id IN membership.allowed_teams
AND
risk <= MEDIUM

La regla exacta no se implementará dentro del ADR; pertenece al Security Policy Engine.

27. Security Policy Engine

El Security Policy Engine seguirá siendo la autoridad central para condiciones complejas.

La arquitectura será:

Authorization Request
        ↓
RBAC
        ↓
Tenant Isolation
        ↓
Ownership
        ↓
ABAC
        ↓
Security Policy Engine
        ↓
Final Decision

La implementación puede optimizar el orden internamente siempre que preserve la semántica de seguridad.

28. Policy Precedence

Se conserva la precedencia conceptual:

Regulatory
    ↓
Platform Security
    ↓
Organization Security
    ↓
Role
    ↓
User
    ↓
Context

Y:

DENY tiene precedencia sobre ALLOW cuando las políticas aplicables entren en conflicto.

Una delegación no puede superar una restricción de nivel superior.

29. MFA and Fine-Grained Access

Una permission sensible podrá requerir:

REQUIRE_MFA

Ejemplos potenciales:

exportar información;
modificar roles;
administrar Memberships;
acceder a información sensible;
modificar políticas;
acciones administrativas.

El MFA requirement será determinado por Security Policy.

30. Step-Up Authorization

Una operación puede comenzar con:

ALLOW? → CHALLENGE

y requerir step-up.

Flujo:

Authorization Request
        ↓
Policy
        ↓
REQUIRE_MFA
        ↓
MFA Challenge
        ↓
Verified
        ↓
Re-evaluate Authorization
        ↓
ALLOW

La evidencia de step-up será temporal y purpose-bound.

31. Role Assignment Security

Asignar un Role también es una operación protegida.

Ejemplo:

Membership Admin
        ↓
Assign Coach

debe pasar por:

Authentication.
Authorization.
Tenant validation.
Security Policy.
Audit.

La capacidad de asignar roles no debe concederse simplemente por ser miembro.

32. Role Management Delegation

Una Organization podrá delegar administración de roles a determinados usuarios.

Pero el alcance deberá ser limitado.

Ejemplo:

Role Manager

puede administrar:

Coach
Nutritionist

pero no:

Organization Administrator

si la policy no lo permite.

Esto evita privilege escalation por role assignment.

33. Permission Administration

La administración de permissions será todavía más sensible.

Los usuarios no deberán crear arbitrariamente nuevas permissions para concederse acceso.

El catálogo de permissions será gobernado por EVOXA.

Las Organizations podrán configurar:

qué roles utilizar;
qué mappings están disponibles;
qué policies aplican;

pero no romper las restricciones mínimas de plataforma.

34. Custom Roles

El modelo podrá soportar Custom Roles en el futuro.

Ejemplo:

Custom Role:
"Assistant Coach"

Permissions:
- athletes.read
- training_plans.read
- training_plans.create

Los Custom Roles deberán utilizar únicamente permissions existentes y permitidas.

No podrán crear capacidades fuera del permission catalog.

35. Role Templates

EVOXA podrá ofrecer Role Templates:

Coach
Nutritionist
Physiotherapist
Team Manager
Administrator

Una Organization podrá activar o adaptar templates según las reglas de plataforma.

La evolución de un Role Template deberá ser versionada para evitar cambios inesperados en permisos existentes.

36. Role Versioning

Los roles podrán tener una versión lógica.

Ejemplo:

Coach v1
Coach v2

Una actualización de Role no deberá cambiar silenciosamente las permissions históricas sin una estrategia de migración.

Esto es especialmente importante para:

compliance;
auditoría;
seguridad;
debugging.
37. Permission Revocation

Cuando una permission sea removida de un Role:

Role
 ↓
Permission removed

la autorización efectiva deberá reflejar el cambio según los mecanismos de cache invalidation.

No se debe depender de que expire el JWT para actualizar permisos críticos.

38. Authorization Cache Invalidation

Los siguientes cambios podrán invalidar authorization caches:

Role assignment.
Role removal.
Permission changes.
Membership suspension.
Membership removal.
Organization suspension.
User suspension.
Policy changes.
Delegation creation/revocation.

La estrategia exacta de caching continúa alineada con los ADR previos.

39. Fail-Closed

Cuando el sistema no pueda determinar de manera confiable:

Membership;
Role;
Permission;
Policy;
Delegation;
Tenant context;

una operación sensible deberá:

DENY

Nunca:

Unknown
 ↓
ALLOW
40. API Authorization Contract

Una API podrá expresar conceptualmente:

authorize(
    actor,
    organization,
    membership,
    action,
    resource,
    context
)

La aplicación no deberá duplicar reglas de autorización de forma independiente en cada controller.

Debe existir una capacidad centralizada de evaluación.

41. Example — Training Plan

Supongamos:

Organization: Org A
Team: Team 10
Training Plan: TP-100
Owner: Coach A

Coach A:

training_plans.update

Resultado:

ALLOW

Coach B:

training_plans.update

pero no pertenece al Team 10.

Resultado potencial:

DENY

aunque tenga la permission global dentro de Org A.

Esto demuestra:

RBAC + Ownership/ABAC
42. Example — Delegated Access
Coach A
   │
   │ delegates
   ▼
Coach B
   │
   └── training_plans.update
       Team 10
       expires_at = ...

Coach B obtiene únicamente el alcance delegado.

No obtiene:

Admin
Role Management
Billing
Other Teams
Other Organizations
43. Example — Cross-Tenant Attempt
User
 └── Org A / Coach

Request:
 └── Org B / Training Plan

Resultado:

DENY

No importa que:

conozca el ID;
tenga el mismo email;
tenga el mismo Role name;
haya sido autenticado por el mismo IdP.
44. Service Accounts

El modelo también deberá aplicarse a Service Accounts.

Una Service Account podrá tener:

Service Identity
 ↓
Organization Scope
 ↓
Scopes
 ↓
Permissions
 ↓
Policy

No deberá utilizarse un User humano ficticio para representar servicios.

Esto mantiene consistencia con ADR-IDENTITY-037 y 038.

45. AI Agents

Los AI Agents serán tratados como principals explícitos.

Un Agent podrá recibir:

Agent Identity
Organization
Task Scope
Permissions
Policy

La autorización será evaluada igual que para otros principals, con controles adicionales cuando corresponda.

Un Agent no puede inferir:

User has access to Org A + Org B
        ↓
Agent has access to Org A + Org B

El Agent necesita su propio authorization context.

46. AI Delegation

Una tarea puede ejecutarse mediante delegación:

User
 ↓
Delegates Task
 ↓
AI Agent
 ↓
Scoped Permission
 ↓
Resource

La delegación deberá especificar:

actor;
Agent;
Organization;
task;
capabilities;
resources;
expiration;
policy.

El Agent no podrá conservar capacidades indefinidamente si la delegación expiró.

47. Auditability

Todas las decisiones sensibles deberán ser auditables.

Debe poder reconstruirse:

WHO
WHAT
WHERE
WHEN
WHY
WITH WHICH ROLE
WITH WHICH PERMISSION
ON WHICH RESOURCE
UNDER WHICH POLICY
RESULT

Especialmente:

role assignments;
role removals;
permission changes;
delegations;
cross-tenant attempts;
administrative operations.
48. Security Events

Podrán generarse Security Events para:

privilege escalation attempt;
unauthorized role assignment;
delegation abuse;
cross-tenant attempt;
policy denial;
suspicious permission usage;
expired delegation usage;
invalid tenant context;
authorization cache inconsistency.
49. Event Model

Eventos conceptuales:

identity.membership.role.assigned.v1
identity.membership.role.removed.v1

identity.delegation.created.v1
identity.delegation.revoked.v1
identity.delegation.expired.v1

identity.authorization.policy.updated.v1
identity.authorization.permission.changed.v1

No todos los authorization evaluations deberán generar eventos de dominio; los eventos de alta frecuencia deberán diferenciarse de Audit/Security telemetry.

50. Persistence Model — Conceptual

Se mantienen las relaciones:

Membership
     │
     ├── MembershipRole
     │       │
     │       ▼
     │      Role
     │       │
     │       ▼
     │   Permission
     │
     └── Delegation

Podría existir conceptualmente:

Delegation
├── id
├── delegator
├── delegate
├── organization
├── scope
├── status
├── starts_at
├── expires_at
├── created_at
├── revoked_at
└── metadata

El schema definitivo queda pendiente.

51. Database Integrity

Deberán existir invariantes como:

MembershipRole pertenece a una Membership válida.
Role pertenece al scope correcto.
Organization Role no puede asignarse fuera de su Organization.
Delegation debe tener un delegator autorizado.
Delegate debe ser identificable.
Delegation no debe superar los permisos del delegator.
Expired delegation no debe conceder acceso.
Cross-tenant delegation requiere scope explícito.
52. Concurrency

Deben protegerse especialmente:

Role assignment
Role removal
Delegation creation
Delegation revocation
Permission updates
Membership suspension

Ejemplo:

Admin removes Role
        +
User executes operation

La decisión deberá respetar las garantías de consistencia y cache invalidation definidas por Identity.

53. Performance

Authorization es una operación de alta frecuencia.

Por ello:

roles y permissions podrán cachearse;
policies podrán cachearse;
mappings podrán versionarse;
tenant context debe resolverse eficientemente;
JWT validation podrá ser local;
llamadas externas no deberán ser necesarias para cada autorización normal.

Objetivo de referencia:

Authorization / API
P95 < 300ms

y:

Authentication
P95 < 500ms

Las optimizaciones no podrán sacrificar fail-closed ni tenant isolation.

54. Disaster Recovery

El recovery debe preservar:

MembershipRole assignments.
Role definitions.
Permission definitions.
Delegations.
Revocations.
Policy versions.
Organization boundaries.

Una pérdida de roles/delegaciones podría producir tanto:

Privilege Escalation

como:

Unauthorized Access

por lo que forman parte del security-critical state.

55. Testing Requirements

Se deberán implementar pruebas para:

RBAC
Role grants permission.
Role removal removes permission.
Multi-role
Multiple roles combine correctly.
Deny policies remain effective.
Ownership
Owner access.
Non-owner denial.
Team ownership.
Delegation
Valid delegation.
Expired delegation.
Revoked delegation.
Delegator without permission.
Delegation scope restriction.
Delegation depth.
Cross-Tenant
Same User / different Organizations.
Same Role / different Organizations.
Resource ID guessing.
Delegation across tenants.
Cache isolation.
Policy
MFA required.
Step-up.
Risk-based deny.
Organization-specific restrictions.
Concurrency
Role assignment/removal races.
Delegation/revocation races.
Membership suspension during request.
AI
Agent scoped to Org A cannot access Org B.
Expired AI delegation fails.
Agent cannot escalate permissions.
56. Security Non-Negotiables

No se permitirá:

Roles globales implícitos para Organization access.
Permission sin tenant context cuando el recurso sea tenant-scoped.
Delegación de permisos no poseídos.
Delegación sin expiration cuando la policy exija temporalidad.
Delegation chains por defecto.
Cross-tenant delegation implícita.
External groups → direct permissions.
Ownership utilizado para saltarse Security Policy.
JWT utilizado para saltarse Membership.
Authorization cache compartida entre tenants.
AI Agent con privilegios superiores a su scope.
Role assignment sin autorización.
Permission catalog creado arbitrariamente por usuarios.
Fail-open ante incertidumbre de autorización.
57. Consequences
Positivas
Modelo de autorización flexible.
RBAC simple para casos comunes.
ABAC para escenarios avanzados.
Ownership para granularidad.
Delegation para colaboración temporal.
Compatible con multi-tenant.
Compatible con AI Agents.
Permite Custom Roles futuros.
Evita privilege escalation.
Facilita auditoría.
Centraliza la autorización.
Negativas
Mayor complejidad que RBAC puro.
Mayor cantidad de estados y políticas.
Cache invalidation más compleja.
Delegation requiere controles adicionales.
Testing de autorización será considerablemente más exigente.
58. Alternatives Considered
A. RBAC puro

Rejected.

No cubre ownership, contexto, riesgo ni delegaciones.

B. ABAC puro

Rejected.

Introduciría complejidad innecesaria para casos sencillos y dificultaría la administración de roles.

C. Permission directa por User

Rejected.

Dificulta governance y aumenta el riesgo de privilege drift.

D. Delegation ilimitada

Rejected.

Puede producir privilege amplification.

E. Global roles

Rejected.

Contradice el modelo multi-organization.

59. Architectural Invariants

Se establecen:

I1:
Membership is the anchor of organization-scoped roles.

I2:
Roles do not automatically cross Organizations.

I3:
Permissions are evaluated within authorization context.

I4:
RBAC alone is not sufficient for fine-grained access.

I5:
Ownership cannot bypass higher-level security policies.

I6:
Delegation cannot exceed delegator authority.

I7:
Delegation is explicit, scoped and revocable.

I8:
Cross-tenant delegation is denied by default.

I9:
JWT validity does not imply current authorization.

I10:
AI Agents require their own explicit authorization context.

I11:
Authorization failures must fail closed.

I12:
Authorization state changes must be auditable.

I13:
Security-critical authorization state must survive disaster recovery.

I14:
Permission and role caches must be tenant/context isolated.
60. Traceability
Blueprint

Este ADR se deriva principalmente de:

Identity.
Authorization.
RBAC.
ABAC.
Organization/Tenant Management.
Membership Management.
Resource Ownership.
Security Policy Engine.
Tenant Isolation.
Audit.
AI Agent Security.
Previous ADRs
ADR-IDENTITY-005 — Authorization & Permission Evaluation
ADR-IDENTITY-011 — Runtime & Deployment
ADR-IDENTITY-022 — Consumer Idempotency
ADR-IDENTITY-025 — Refresh Token Lifecycle
ADR-IDENTITY-026 — API Pagination, Filtering & Idempotency
ADR-IDENTITY-028 — Observability & Operational Security
ADR-IDENTITY-033 — Account Lifecycle
ADR-IDENTITY-037 — API Tokens & Service Credentials
ADR-IDENTITY-038 — Service-to-Service Authentication
ADR-IDENTITY-040 — Federation
ADR-IDENTITY-041 — Provisioning & Deprovisioning
ADR-IDENTITY-042 — Source of Truth & Conflict Resolution
ADR-IDENTITY-043 — Multi-Organization Identity & Cross-Tenant Access
61. Pending Decisions

Antes de implementación deberán definirse:

Catálogo definitivo de Roles.
Catálogo definitivo de Permissions.
Role scope definitivo.
Custom Roles.
Role Templates.
Role versioning.
MembershipRole schema.
Delegation schema.
Delegation lifecycle.
Delegation maximum duration.
Delegation depth.
Cross-tenant delegation rules.
Resource ownership model por dominio.
ABAC attribute catalog.
Security Policy language/model.
Permission evaluation algorithm.
Authorization cache strategy.
Cache invalidation events.
System Roles.
Platform Administrator scope.
Organization Administrator scope.
MFA requirements by permission.
Step-up requirements.
Resource sharing model.
Delegated access API.
Role management API.
Permission management API.
Audit event schema.
Authorization decision telemetry.
AI Agent authorization model.
Service Account fine-grained permissions.
Database-level tenant enforcement.
PostgreSQL Row-Level Security evaluation.
Performance benchmarks.
62. Acceptance Criteria

El ADR podrá considerarse aceptado cuando:

 Roles organizacionales estén asociados a Membership.
 Una Membership pueda tener múltiples Roles.
 Roles y Permissions estén separados.
 Permission catalog sea explícito.
 RBAC sea complementado por ABAC.
 Resource Ownership sea soportado.
 Delegation sea explícita y revocable.
 Delegation no pueda ampliar privilegios del delegante.
 Delegation tenga scope.
 Delegation pueda expirar.
 Delegation cross-tenant sea DENY by default.
 Role hierarchy no sea implícita.
 Permission changes invaliden autorización efectiva correctamente.
 JWT no pueda saltarse Membership.
 Security Policy pueda requerir MFA/step-up.
 Authorization falle closed.
 Cache sea tenant/context-aware.
 AI Agents tengan authorization context propio.
 Role/permission/delegation changes sean auditables.
 Security-critical authorization state sea recuperable.
 Existan pruebas automatizadas de privilege escalation y cross-tenant access.
63. Architectural Outcome

Con ADR-043 + ADR-044, el modelo de autorización de EVOXA queda definido conceptualmente como:

                         ┌───────────────┐
                         │     USER      │
                         └───────┬───────┘
                                 │
                                 ▼
                         ┌───────────────┐
                         │  MEMBERSHIP   │
                         │ Organization  │
                         └───────┬───────┘
                                 │
                    ┌────────────┼────────────┐
                    ▼            ▼            ▼
                  Role 1       Role 2      Delegation
                    │            │            │
                    └────────────┼────────────┘
                                 ▼
                         ┌───────────────┐
                         │  PERMISSIONS  │
                         └───────┬───────┘
                                 │
                    ┌────────────┼────────────┐
                    ▼            ▼            ▼
                 Tenant       Ownership      ABAC
                 Isolation
                    │            │            │
                    └────────────┼────────────┘
                                 ▼
                       Security Policy Engine
                                 │
                                 ▼
                     ┌─────────────────────┐
                     │ Authorization       │
                     │ Decision            │
                     └─────────────────────┘
                                 │
                    ┌────────────┼────────────┐
                    ▼            ▼            ▼
                  ALLOW        DENY       STEP-UP

La regla central queda:

Membership determina el contexto organizacional; Roles agrupan capabilities; Permissions expresan acciones concretas; Ownership y ABAC limitan el alcance; Delegation permite acceso explícito y temporal; y el Security Policy Engine determina la decisión final sin permitir privilege escalation ni cross-tenant access implícito.

Siguiente ADR recomendado

El siguiente paso natural sería:

ADR-IDENTITY-045 — Identity Authorization Policy Model, Evaluation & Decision Architecture

Ahí podemos definir cómo funciona internamente el motor que toma toda esta información y produce ALLOW, DENY, CHALLENGE, REAUTHENTICATE, REQUIRE_MFA o REVOKE, incluyendo evaluación, precedencia, policy versioning, caching, explainability, conflictos y performance.
