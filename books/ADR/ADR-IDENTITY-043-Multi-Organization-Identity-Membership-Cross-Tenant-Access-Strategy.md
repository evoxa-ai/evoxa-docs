ADR-IDENTITY-043 — Multi-Organization Identity, Membership & Cross-Tenant Access Strategy

Status: Proposed
Version: 1.0
Date: 2026-09-03
Domain: Identity
Priority: Critical
Type: Architecture / Security / Multi-Tenancy

1. Context

EVOXA define Organization como la frontera principal de aislamiento de tenant.

El modelo de Identity contempla una separación explícita entre:

User
Organization
Membership
Role
Permission
Session

Esto permite que una misma identidad pueda participar en múltiples Organizations sin duplicar necesariamente el User.

Ejemplo:

                         ┌───────────────┐
                         │     User      │
                         │   Sebastian   │
                         └───────┬───────┘
                                 │
                 ┌───────────────┼───────────────┐
                 │               │               │
                 ▼               ▼               ▼
          Organization A   Organization B   Organization C
                 │               │               │
                 ▼               ▼               ▼
             Membership       Membership       Membership
                 │               │               │
                 ▼               ▼               ▼
               Roles           Roles            Roles

La identidad es global dentro de EVOXA, mientras que el acceso y los permisos se evalúan dentro de un contexto organizacional.

Esto es especialmente importante porque EVOXA debe evitar:

acceso accidental entre tenants;
reutilización incorrecta de permisos;
sesiones ambiguas;
roles heredados incorrectamente;
consultas sin tenant context;
Federation cross-tenant;
operaciones administrativas sobre Organizations incorrectas.
2. Problem Statement

Se necesita definir cómo funcionará una identidad cuando:

Pertenece a una Organization.
Pertenece a múltiples Organizations.
Tiene roles diferentes en cada Organization.
Tiene memberships activas y suspendidas simultáneamente.
Cambia de Organization durante una sesión.
Recibe una invitación a otra Organization.
Es administradora en una Organization y usuario normal en otra.
Una Organization suspende su Membership.
El usuario queda globalmente suspendido.
Un AI Agent opera dentro de una Organization.
Una API recibe un request sin tenant explícito.
Un token intenta utilizarse contra otra Organization.

La arquitectura debe garantizar que:

Membership es la frontera de autorización organizacional; User por sí solo no concede acceso a una Organization.

3. Decision

EVOXA adoptará un modelo de:

Global User + Organization-Scoped Membership + Organization-Scoped Authorization

Esto significa:

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
Authorization

No:

User
  ↓
Global Roles
  ↓
All Organizations

salvo para capacidades de plataforma explícitamente definidas como SYSTEM scope.

4. User as Global Identity

User representa la identidad dentro de EVOXA.

El User no pertenece necesariamente a una única Organization.

Puede existir:

User ACTIVE

con:

Membership A ACTIVE
Membership B ACTIVE
Membership C REMOVED

Por tanto:

El lifecycle de User y el lifecycle de Membership son independientes, aunque estén relacionados.

5. Organization as Tenant Boundary

Organization será la frontera principal de aislamiento.

Conceptualmente:

Organization
├── Members
├── Roles
├── Policies
├── Resources
├── Audit Context
└── Security Context

Una operación sobre recursos organizacionales deberá estar asociada a un organization_id o a un contexto de tenant equivalente.

6. Membership as Access Boundary

Membership representa la relación:

User ↔ Organization

y será el elemento fundamental para determinar si el usuario puede operar dentro de una Organization.

Estados conceptuales:

INVITED
ACTIVE
SUSPENDED
REMOVED

Regla:

Membership != ACTIVE
        ↓
No organizational access

Una Membership SUSPENDED no implica necesariamente que el User esté suspendido globalmente.

7. Multiple Memberships

Un usuario puede tener múltiples memberships:

User 123
│
├── Org A → ACTIVE
├── Org B → ACTIVE
├── Org C → SUSPENDED
└── Org D → REMOVED

Esto permite escenarios como:

Coach que trabaja con varios clubes.
Nutritionist que trabaja con varios equipos.
Profesional que pertenece a varias organizaciones.
Administrador de una organización que colabora con otra.
Enterprise user con múltiples unidades organizacionales.
8. Organization-Scoped Roles

Los roles de Organization deberán estar asociados al contexto organizacional.

Ejemplo:

User
├── Org A
│    └── Role: Admin
│
└── Org B
     └── Role: Coach

El usuario no debe recibir:

Admin global

simplemente porque posee Admin en Organization A.

Por lo tanto:

Role(Org A) ≠ Role(Org B)
9. System Roles

EVOXA podrá tener roles de scope:

SYSTEM
ORGANIZATION

Los roles SYSTEM representan capacidades de plataforma y deberán ser extremadamente restringidos.

Ejemplo conceptual:

SYSTEM
 └── Platform Administrator

ORGANIZATION
 ├── Organization Administrator
 ├── Coach
 ├── Nutritionist
 └── Physiotherapist

Un rol SYSTEM no debe convertirse automáticamente en acceso irrestricto a todos los recursos de negocio.

Las capacidades cross-tenant deberán requerir una autorización explícita de plataforma.

10. Authorization Context

Toda decisión de autorización deberá considerar un contexto similar a:

AuthorizationContext
├── user
├── organization
├── membership
├── roles
├── permissions
├── session
├── authentication_method
├── MFA
├── device
├── request
├── resource
├── operation
└── risk

La Organization activa deberá formar parte explícita del contexto cuando la operación sea tenant-scoped.

11. Tenant Context

Toda request que opere sobre recursos de Organization deberá tener un tenant context inequívoco.

Conceptualmente:

Request
   ↓
Authenticate
   ↓
Resolve User
   ↓
Resolve Organization Context
   ↓
Resolve Membership
   ↓
Authorization
   ↓
Resource Access

No se permitirá ejecutar una operación tenant-scoped únicamente con:

user_id

cuando la operación requiera Organization.

12. Tenant Context Sources

El contexto de Organization puede provenir conceptualmente de:

URL/path.
Header controlado.
Access token/session context.
Subdomain.
Explicit organization selector.
Resource ownership.

Pero la fuente concreta no se considera autoridad por sí misma.

Ejemplo:

X-Organization-ID: org-123

no significa:

User automatically belongs to org-123

Debe existir:

User
 ↓
Membership(org-123)
 ↓
Policy
13. URL-Based Tenant Context

Una API puede utilizar:

/api/v1/organizations/{organization_id}/...

como mecanismo explícito.

Esto es recomendable para operaciones donde el tenant forma parte natural del recurso.

La existencia del organization_id en la URL no sustituye Authorization.

Debe verificarse:

organization_id
+
user
+
membership
+
roles
+
permissions
+
policy
14. Active Organization

En aplicaciones Web/Mobile donde el usuario tiene varias Organizations, podrá existir un concepto de:

Active Organization Context

Ejemplo:

User
 ├── Org A
 ├── Org B
 └── Org C

Current Context:
Org B

Cambiar de Organization no cambia de identidad.

Cambia el contexto de autorización.

Same User
      ↓
Different Membership
      ↓
Different Roles
      ↓
Different Permissions
15. Organization Switching

El cambio de Organization deberá ser una operación explícita.

Flujo conceptual:

User authenticated
       ↓
List ACTIVE memberships
       ↓
Select Organization
       ↓
Validate Membership
       ↓
Evaluate Policy
       ↓
Establish Organization Context
       ↓
Continue

No se permitirá cambiar simplemente manipulando un identificador enviado por el cliente.

16. Session Context

Una Session podrá estar asociada a:

User.
Organization context, cuando corresponda.
Membership.
Authentication method.
MFA state.
Device/client context.

Sin embargo, se deberá distinguir:

Session Identity
Who is the user?

de:

Authorization Context
Which organization is being accessed?

Esto evita duplicar innecesariamente sesiones cuando el mismo usuario puede cambiar entre Organizations.

La decisión final sobre si la Organization activa se persiste directamente en Session o se resuelve dinámicamente queda pendiente.

17. Token Context

Los access tokens podrán contener claims relacionados con contexto cuando sea necesario.

Sin embargo:

No se deberá confiar únicamente en un organization_id incluido en el JWT para autorizar acceso.

El token puede expresar contexto, pero la autorización debe validar que el contexto siga siendo válido.

Especialmente después de:

Membership suspension.
Membership removal.
Organization suspension.
User suspension.
Role changes.
Security policy changes.
18. JWT Validity vs Authorization Validity

Un JWT válido criptográficamente no garantiza acceso actual.

Ejemplo:

JWT
├── Signature VALID
├── exp VALID
└── issuer VALID

pero:

Membership = SUSPENDED

Resultado:

DENY

Por tanto:

Valid JWT
      ≠
Valid Authorization
19. Cross-Tenant Access

El acceso entre Organizations estará DENY by default.

Ejemplo:

User
 └── Org A → ACTIVE

Request:
 └── Org B

Resultado:

DENY

aunque:

el User exista;
el JWT sea válido;
el User tenga roles;
el email coincida;
el User sea administrador en Org A.
20. Cross-Tenant Operations

Algunas operaciones de plataforma pueden legítimamente necesitar operar sobre múltiples Organizations.

Estas operaciones deben utilizar un contexto especial:

PLATFORM

y no una Membership normal.

Ejemplo conceptual:

Platform Admin
       ↓
System Authorization
       ↓
Cross-Tenant Operation
       ↓
Explicit Scope
       ↓
Audit

No se permitirá que una Membership organizacional normal se convierta implícitamente en cross-tenant access.

21. Cross-Tenant Explicit Scope

Cuando una operación cross-tenant sea legítima deberá declarar explícitamente:

source_scope
target_scope
operation
reason
actor
policy

Esto permitirá diferenciar:

Organization-scoped operation

de:

Platform-scoped operation
22. Resource Ownership

Tenant isolation deberá complementarse con resource ownership.

Ejemplo:

Org A
 ├── Team 1
 ├── Athlete 1
 └── Training Plan 1

Un usuario de Org A puede tener Membership activa pero aun así no tener acceso a todos los recursos de Org A.

La autorización deberá considerar:

Tenant
+
Membership
+
Role
+
Permission
+
Ownership
+
Policy

Esto mantiene la estrategia RBAC + ABAC + Tenant Isolation + Ownership.

23. Organization Suspension

Si:

Organization = SUSPENDED

las Memberships pueden permanecer registradas, pero el acceso operativo deberá estar bloqueado según policy.

Ejemplo:

User ACTIVE
Membership ACTIVE
Organization SUSPENDED

        ↓

Access = DENY

Cuando la Organization vuelva a ACTIVE, las Memberships no deberían perder necesariamente su estado anterior.

24. Membership Suspension

Si:

Membership A = SUSPENDED

pero:

Membership B = ACTIVE

el usuario podrá continuar operando en Organization B.

Ejemplo:

User ACTIVE
│
├── Org A → SUSPENDED → DENY
└── Org B → ACTIVE    → ALLOW

Esto demuestra la importancia de no utilizar User como único estado de autorización.

25. User Suspension

Si:

User = SUSPENDED

el bloqueo tendrá alcance global sobre las operaciones del usuario, sujeto a las políticas de plataforma.

Conceptualmente:

User SUSPENDED
       ↓
Org A → DENY
Org B → DENY
Org C → DENY

La suspensión global deberá integrarse con:

Sessions.
Refresh Tokens.
MFA.
Authorization.
Security Events.
Audit.
26. Organization Deletion

La eliminación de una Organization no debe provocar necesariamente la eliminación física de Users.

Ejemplo:

Organization A
      ↓
DELETED / RETIRED
      ↓
Memberships no longer active

El User puede continuar existiendo:

User
 ├── Org A → REMOVED
 └── Org B → ACTIVE

Esto protege la integridad del modelo global de Identity.

27. Membership Invitation

Las invitaciones estarán siempre asociadas a una Organization.

Invitation
├── organization_id
├── target
├── role proposal
├── expiration
├── status
└── audit metadata

Aceptar una invitación no debe permitir seleccionar arbitrariamente:

Admin

si la invitación o policy no lo permiten.

28. Membership Role Assignment

Los roles deben estar asociados a Membership/context.

Ejemplo:

Membership
├── user_id
├── organization_id
└── roles

No:

User.roles

como mecanismo universal para roles organizacionales.

29. Role Changes

Un cambio de rol deberá afectar solamente el contexto correspondiente.

Ejemplo:

Org A
Coach → Admin

no debe modificar:

Org B
Coach

La propagación cross-tenant requerirá una operación explícita.

30. Permission Evaluation

Las permissions deberán derivarse considerando el contexto:

User
+
Organization
+
Membership
+
Roles
+
Permission
+
Resource
+
Policy

Por lo tanto:

has_permission(user, "edit_training_plan")

puede ser insuficiente.

Conceptualmente deberá evaluarse algo equivalente a:

authorize(
    user,
    organization,
    membership,
    operation,
    resource,
    context
)
31. Multi-Tenant Data Access

Las consultas tenant-scoped deberán incorporar el contexto de Organization.

Conceptualmente:

SELECT ...
FROM resource
WHERE organization_id = :authorized_organization_id

Nunca deberá depender exclusivamente de:

WHERE user_id = ...

cuando el recurso pertenezca a una Organization.

La protección debe existir en múltiples capas:

API
 ↓
Application
 ↓
Domain
 ↓
Persistence
32. Defense in Depth

Tenant isolation no debe depender de una única validación.

Se deberán considerar controles en:

API.
Authorization.
Application Services.
Domain Rules.
Repository/Data Access.
Database constraints cuando sean aplicables.
Audit.
Automated Security Tests.

Esto reduce el riesgo de un bypass accidental.

33. Cache Isolation

Si Redis u otra caché almacena datos tenant-scoped, las claves deberán estar aisladas.

Conceptualmente:

tenant:{organization_id}:resource:{id}

y nunca depender solamente de:

resource:{id}

cuando exista posibilidad de colisión o acceso cruzado.

Los datos cacheados no deben convertirse en una vía alternativa de autorización.

34. Authorization Cache

Los resultados cacheados de Authorization deberán incluir contexto suficiente.

Ejemplo:

authorization:
user
+
organization
+
membership
+
policy_version
+
resource
+
operation

Un permiso cacheado para:

User 1 / Org A

nunca debe reutilizarse para:

User 1 / Org B
35. Organization Context in AI Agents

Los AI Agents deberán operar con un contexto organizacional explícito.

Ejemplo:

AI Agent
 ├── Agent Identity
 ├── Organization
 ├── Task
 ├── Permissions
 └── Policy

Un AI Agent autorizado para:

Org A

no podrá asumir acceso a:

Org B

por pertenecer al mismo usuario, servicio o plataforma.

36. AI Cross-Tenant Operations

Las operaciones cross-tenant de AI deberán requerir:

identidad de Agent;
scope explícito;
autorización;
policy;
audit;
trazabilidad.

El modelo será:

AI Agent
   ↓
Task Scope
   ↓
Authorization
   ↓
Policy
   ↓
Operation
   ↓
Audit

No:

AI Agent
   ↓
Platform Master Access
37. Federation Integration

La Federation definida en ADR-IDENTITY-040 deberá respetar este modelo.

Un external identity autenticado podrá resultar en:

ExternalIdentity
      ↓
User
      ↓
Organization Mapping
      ↓
Membership
      ↓
Role Mapping

Nunca:

ExternalIdentity
      ↓
Global Access
38. Provisioning Integration

ADR-IDENTITY-041 deberá interpretarse de la siguiente manera:

Provision User
       ↓
Determine Organization
       ↓
Create/Update Membership
       ↓
Apply Role Mapping
       ↓
Apply Policy

El provisioning de un User no significa necesariamente provisioning global en todas las Organizations.

39. Synchronization Integration

ADR-IDENTITY-042 establece que los atributos y lifecycle tienen ownership específico.

Para Membership:

External Source
      ↓
Mapping Policy
      ↓
Organization
      ↓
Membership

El mapping deberá estar limitado al tenant autorizado.

40. Audit Context

Todo evento de autorización organizacional deberá poder identificar:

user_id
organization_id
membership_id
role/context
resource
operation
decision

En una operación cross-tenant deberá además registrarse el scope de plataforma correspondiente.

41. Security Event Context

Security Events deberán distinguir:

User scope
Organization scope
Platform scope

Ejemplo:

MFA failure
→ User + Organization

Cross-tenant access attempt
→ User + Source Org + Target Org

Platform administration
→ User + Platform scope
42. Error Handling

La API deberá evitar revelar información sensible sobre otras Organizations.

Por ejemplo, ante:

GET /organizations/org-B/resource/123

si el usuario pertenece solamente a Org A, la respuesta no debería revelar innecesariamente:

existencia del recurso;
existencia del usuario;
estructura interna;
permisos de Org B.

La estrategia exacta entre:

403 FORBIDDEN

y:

404 NOT FOUND

deberá depender del recurso y de la política anti-enumeration.

43. Organization Discovery

Cuando un usuario tenga varias Organizations, la aplicación podrá consultar:

GET /api/v1/me/organizations

conceptualmente.

La respuesta deberá incluir solamente Organizations para las cuales exista una relación autorizada.

Ejemplo:

{
  "organizations": [
    {
      "id": "org-a",
      "name": "Organization A",
      "membership_status": "ACTIVE"
    },
    {
      "id": "org-b",
      "name": "Organization B",
      "membership_status": "ACTIVE"
    }
  ]
}

Los campos definitivos se definirán en API Contracts.

44. Default Organization

Un usuario puede tener una Organization preferida para UX.

Pero:

Default Organization ≠ Authorization

Por ejemplo:

default_organization = Org A

no elimina la necesidad de validar Membership.

45. Organization Switching and MFA

Cambiar de Organization normalmente no debería requerir una nueva autenticación completa si la sesión ya tiene suficiente assurance.

Sin embargo, Security Policy podrá exigir:

Organization Switch
      ↓
Risk Evaluation
      ↓
MFA / Step-Up

para organizaciones o recursos de alta sensibilidad.

46. Sensitive Organizations

Una Organization podrá tener policies adicionales como:

MFA obligatorio.
Step-up.
Device restrictions.
IP/network restrictions.
Session limits.
Shorter session lifetime.

Estas políticas deben aplicarse en el contexto de esa Organization y no necesariamente modificar la política de otras Organizations.

47. Cross-Organization Collaboration

EVOXA podrá soportar escenarios donde usuarios de diferentes Organizations colaboren sobre recursos compartidos.

Esto no debe resolverse eliminando tenant isolation.

Se deberá utilizar un mecanismo explícito de:

Resource Sharing

o:

Delegated Access

con:

source organization;
target organization;
resource;
permissions;
expiration;
policy;
audit.
48. Explicit Delegation

Un usuario de Org A puede recibir acceso específico a un recurso de Org B mediante una delegación explícita.

Conceptualmente:

Org A User
      ↓
Delegation
      ↓
Specific Resource in Org B
      ↓
Limited Permission

Esto no convierte al usuario en miembro general de Org B.

49. Cross-Tenant Sharing ≠ Membership

Se establece una distinción fundamental:

Membership

representa pertenencia organizacional.

Mientras:

Delegated Resource Access

representa acceso específico.

No deberán confundirse.

50. Concurrency & State Changes

Ejemplo:

User changes Org context
          +
Membership gets suspended

simultáneamente.

La autorización debe evaluar el estado efectivo más reciente disponible según las garantías de consistencia definidas.

Una Membership que ya no está activa no debe continuar concediendo acceso simplemente porque el cliente conserve un contexto anterior.

51. Persistence Model — Conceptual

El modelo principal será:

User
Organization
Membership
Role
Permission
MembershipRole
RolePermission
Session

Relaciones:

User 1 ───── N Membership N ───── 1 Organization

Membership N ───── N Role

Role N ───── N Permission

Esto permite:

User
  ↓
Membership
  ↓
Organization
  ↓
Roles
  ↓
Permissions
52. Database Integrity

La persistencia deberá proteger invariantes como:

Membership pertenece a una Organization válida.
Membership pertenece a un User válido.
Membership no debe duplicarse para el mismo User/Organization cuando el modelo no lo permita.
Role assignment debe estar correctamente contextualizado.
Resource ownership debe permanecer tenant-consistent.

Las constraints exactas se definirán en los ADR de Persistence/Schema.

53. Event Model

Cambios relevantes pueden generar eventos:

identity.membership.created.v1
identity.membership.activated.v1
identity.membership.suspended.v1
identity.membership.removed.v1

identity.organization.suspended.v1
identity.organization.reactivated.v1

identity.organization.context.changed.v1
identity.cross_tenant.access.denied.v1

El evento organization.context.changed deberá evaluarse cuidadosamente, ya que un cambio de contexto de UI no necesariamente constituye un evento de dominio.

54. Idempotency

Las operaciones mutantes relacionadas con Membership deberán utilizar las reglas de idempotencia de:

ADR-IDENTITY-026

Ejemplos:

Accept invitation.
Create membership.
Suspend membership.
Remove membership.
Assign role.
Remove role.

Una misma operación repetida no deberá generar efectos duplicados.

55. Disaster Recovery

Después de una recuperación deberán conservarse:

User state.
Organization state.
Membership state.
Role assignments.
Session revocations.
Refresh token revocations.
Security policies.
Audit events.

Una pérdida de Membership state podría producir un riesgo crítico de cross-tenant access.

Por lo tanto:

La integridad de Membership es parte del security boundary del sistema.

56. Observability

Las métricas deberán permitir detectar:

authorization denials;
cross-tenant attempts;
membership suspension;
organization suspension;
abnormal organization switching;
role changes;
policy denials;
tenant isolation violations.

No se deberán utilizar identificadores de alta cardinalidad innecesarios en métricas.

Los detalles completos deberán permanecer en logs/audit/security events.

57. Testing Requirements

Deberán existir pruebas para:

Single Organization
User → Org A → ALLOW
Multiple Organizations
User → Org A → ALLOW
User → Org B → ALLOW
Wrong Organization
User → Org A
Request → Org B
Result → DENY
Different Roles
Org A → Admin
Org B → Coach

y verificar que:

Admin privileges from A
≠
B privileges
Suspended Membership
Org A → SUSPENDED
Org B → ACTIVE

Resultado:

A → DENY
B → ALLOW
Suspended User
User → SUSPENDED

Resultado:

All organizational access → DENY
Cross-Tenant Resource ID

Probar que conocer el ID de un recurso de otra Organization no concede acceso.

Cache Isolation

Verificar que permisos de Org A no se reutilicen en Org B.

AI Isolation

Verificar que un Agent scoped a Org A no pueda operar sobre Org B.

58. Security Non-Negotiables

No se permitirá:

User → acceso global a todas las Organizations.
Role de Org A heredado automáticamente a Org B.
Permission cache compartido entre tenants.
JWT válido utilizado como bypass de Membership.
organization_id enviado por cliente sin validación.
Resource ID utilizado como sustituto de authorization.
Cross-tenant access por coincidencia de email.
External Federation otorgando acceso global.
SCIM creando acceso en tenants no autorizados.
AI Agent con acceso cross-tenant implícito.
Membership suspendida permitiendo refresh.
Organization suspendida permitiendo acceso operativo normal.
Cross-tenant sharing implementado como Membership implícita.
Direct database access bypassing tenant authorization.
59. Consequences
Positivas
Aislamiento multi-tenant fuerte.
Un User puede pertenecer a múltiples Organizations.
Roles independientes por Organization.
Authorization contextual.
Compatible con Federation y SCIM.
Facilita Enterprise SaaS.
Reduce riesgo de privilege leakage.
Permite colaboración cross-organization de forma controlada.
Compatible con AI Agents scoped por tenant.
Permite políticas de seguridad específicas por Organization.
Negativas
Mayor complejidad de Authorization.
Mayor complejidad de Session/Context.
Mayor cantidad de validaciones.
Necesidad de tenant-aware caching.
Testing de aislamiento más exigente.
Cross-tenant collaboration requiere un modelo específico.
60. Alternatives Considered
A. Un User pertenece a una sola Organization

Rejected.

Limitaría casos de uso legítimos y contradice el modelo de Membership.

B. Roles globales por User

Rejected.

Provocaría privilege leakage entre Organizations.

C. Organization definida únicamente por JWT

Rejected.

El JWT no refleja necesariamente el estado actual de Membership.

D. Cross-Tenant Access basado en Admin

Rejected.

Un administrador organizacional no debe adquirir automáticamente autoridad de plataforma.

E. Duplicar User por Organization

Rejected as default.

Generaría problemas de identidad, account linking, MFA y recuperación.

61. Architectural Invariants

Se establecen como invariantes:

I1:
User identity is independent from Organization membership.

I2:
Membership is the organizational access boundary.

I3:
Organization is the primary tenant isolation boundary.

I4:
Organization-scoped roles do not cross Organizations.

I5:
Permissions are evaluated within authorization context.

I6:
Valid JWT does not imply valid tenant access.

I7:
Cross-tenant access is DENY by default.

I8:
Platform-level access requires explicit authorization.

I9:
Resource ownership cannot cross tenants implicitly.

I10:
AI Agents inherit explicit tenant scope and cannot broaden it.

I11:
Suspended/removed Membership cannot grant new access.

I12:
Tenant isolation must be enforced in defense-in-depth layers.
62. Traceability
Blueprint

Este ADR deriva principalmente de:

Identity.
Organization / Tenant Management.
Membership Management.
Authentication.
Authorization.
RBAC.
ABAC.
Tenant Isolation.
Resource Ownership.
Security Policy Engine.
Audit.
AI Agent Security.
Related ADRs
ADR-IDENTITY-001 — Identity Architecture
ADR-IDENTITY-005 — Authorization & Permission Evaluation
ADR-IDENTITY-006 — Identity Persistence
ADR-IDENTITY-007 — Domain Events & Outbox
ADR-IDENTITY-009 — Token Storage
ADR-IDENTITY-012 — PostgreSQL Schema
ADR-IDENTITY-021 — Controlled Replay
ADR-IDENTITY-022 — Consumer Idempotency
ADR-IDENTITY-025 — Refresh Token Cryptography & Concurrency
ADR-IDENTITY-026 — Pagination, Filtering & Idempotency
ADR-IDENTITY-028 — Observability & Operational Security
ADR-IDENTITY-029 — Disaster Recovery & Backup
ADR-IDENTITY-031 — High Availability & Failover
ADR-IDENTITY-033 — Account Lifecycle
ADR-IDENTITY-037 — API Tokens & Service Credentials
ADR-IDENTITY-038 — Service-to-Service Authentication
ADR-IDENTITY-039 — OAuth2.1 & OIDC
ADR-IDENTITY-040 — Federation
ADR-IDENTITY-041 — Provisioning & Deprovisioning
ADR-IDENTITY-042 — Source of Truth & Conflict Resolution
63. Pending Decisions

Antes de implementación deberán definirse:

Formato definitivo de Organization Context.
Si Session almacena organization_id.
Si Access Token contiene organization context.
Mecanismo definitivo de Organization Switching.
Endpoint definitivo para listar Organizations.
Default Organization behavior.
System Roles definitivos.
Cross-tenant platform scopes.
Resource Sharing model.
Delegated Access model.
Membership uniqueness constraints.
Role scope model definitivo.
Organization-specific MFA policies.
Session behavior durante Organization switching.
Cache key structure.
Authorization cache invalidation.
Exact 403 vs 404 strategy.
Organization suspension behavior.
Membership suspension and session revocation rules.
Global User suspension semantics.
Cross-tenant administrative workflows.
AI Agent organization-scoping model.
Database-level tenant enforcement strategy.
Row-Level Security evaluation.
API contract definitivo.
Event catalog definitivo.
64. Acceptance Criteria

El ADR podrá considerarse aceptado cuando:

 Un User pueda tener múltiples Memberships.
 Cada Membership esté asociada a una única Organization.
 Organization sea la frontera primaria de tenant.
 Roles organizacionales sean independientes por Membership.
 Permissions se evalúen dentro del contexto de Organization.
 Cross-tenant access sea DENY by default.
 JWT válido no permita saltarse Membership.
 Organization context sea validado.
 Resource ownership sea tenant-aware.
 Cache esté aislada por tenant.
 User suspension tenga alcance global según policy.
 Membership suspension afecte solamente el contexto correspondiente.
 Organization suspension bloquee acceso operativo según policy.
 Federation respete tenant isolation.
 Provisioning respete Organization scope.
 AI Agents tengan tenant scope explícito.
 Operaciones cross-tenant requieran autorización de plataforma.
 Exista soporte para colaboración cross-organization sin romper tenant isolation.
 Existan pruebas automatizadas de cross-tenant isolation.
 Recovery preserve Membership y revocation state.
 Audit y Security Events incluyan contexto organizacional.
65. Architectural Outcome

Con los ADR 040 → 043, el modelo de Identity queda evolucionando hacia una arquitectura multi-tenant bastante clara:

                         ┌──────────────────┐
                         │       USER       │
                         │ Global Identity  │
                         └────────┬─────────┘
                                  │
                     ┌────────────┼────────────┐
                     │            │            │
                     ▼            ▼            ▼
                 Membership    Membership   Membership
                     │            │            │
                     ▼            ▼            ▼
                  Org A        Org B        Org C
                     │            │            │
                  Admin         Coach       Suspended
                     │            │
                     ▼            ▼
                  Roles        Roles
                     │            │
                     ▼            ▼
                Permissions   Permissions
                     │            │
                     └──────┬─────┘
                            ▼
                  Authorization Engine
                            │
             ┌──────────────┼──────────────┐
             ▼              ▼              ▼
          Tenant         Ownership       Policy
          Isolation                     Engine

Y para Federation/Provisioning:

OIDC / SAML
     │
     ▼
ExternalIdentity
     │
     ▼
User
     │
     ▼
Provisioning / Mapping
     │
     ▼
Membership
     │
     ▼
Organization
     │
     ▼
Roles
     │
     ▼
Permissions
     │
     ▼
Authorization

La regla arquitectónica central queda:

Un User representa quién es la persona; una Membership determina en qué Organization puede participar; los Roles y Permissions determinan qué puede hacer dentro de ese contexto; y ninguna de esas capacidades debe cruzar tenants implícitamente.

Con esto, el siguiente paso natural sería ADR-IDENTITY-044 — Organization Membership Roles, Delegation & Fine-Grained Access Strategy, para bajar un nivel más y definir cómo funcionarán los roles por Membership, delegaciones, ownership y permisos granulares dentro de cada Organization.
