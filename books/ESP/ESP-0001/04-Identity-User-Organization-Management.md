ESP-0001 / 04 — Identity User & Organization Management

Document ID: ESP-0001/04
Module: Identity
Version: 1.1.0
Status: Draft — Consolidated
Parent: ESP-0001 — Identity
Previous Specifications:

ESP-0001/01 — Identity Domain Model
ESP-0001/02 — Identity Architecture
ESP-0001/03 — Identity Authentication & Authorization

Architecture Parent: BP-0002 — Reference Architecture
Repository: evoxa-platform
Target Path: apps/api/app/domains/identity/

1. Purpose

Este documento define el modelo y comportamiento de:

User;
Organization;
Membership;
Invitation;
User Lifecycle;
Organization Lifecycle;
Membership Lifecycle;
Tenant Context;
User Administration;
Organization Administration;
Membership Administration;
Identity Audit;
Privacy;
Tenant Isolation.

Identity proporciona la identidad y el contexto organizacional/de seguridad.

Los dominios de negocio mantienen sus propios:

recursos;
entidades;
reglas de negocio;
ownership;
datos funcionales.

Esta separación ya está establecida en el modelo arquitectónico de Identity. El documento original también establece que Identity es propietario de Users, Organizations, Memberships y del lifecycle de identidad, pero no de Training, Nutrition, Recovery, AI u otros dominios de negocio.

2. Scope

Este ESP cubre:

User
Organization
Membership
Invitation
User Lifecycle
Organization Lifecycle
Membership Lifecycle
Tenant Context
Identity Administration
Organization Administration
Membership Administration
Audit
Privacy
Tenant Isolation

No desarrolla en profundidad:

Authentication
Authorization
Roles
Permissions
MFA
Sessions
Refresh Tokens
API Tokens
Training
Nutrition
Recovery
Billing
AI

Estas capacidades tienen sus propios ESP/ADR.

3. Core Identity Model

El modelo fundamental es:

                    ┌──────────────┐
                    │     User     │
                    └──────┬───────┘
                           │
                      Membership
                           │
                           ▼
                  ┌─────────────────┐
                  │  Organization   │
                  └─────────────────┘

La definición consolidada es:

User
    =
Identity

Organization
    =
Organizational / Tenant Boundary

Membership
    =
Relationship + Access Boundary

Por tanto:

User ≠ Organization
User ≠ Membership
Membership ≠ Role
Role ≠ Permission
4. User
4.1 Definition

User representa una identidad individual dentro de EVOXA.

El User es independiente de los dominios de negocio.

Ejemplo:

User
 │
 ├── Coach
 ├── Nutritionist
 ├── Physiotherapist
 └── Team Manager

Los perfiles/roles no forman parte de la identidad básica.

Son parte de Authorization.

5. Stable User Identity

Cada User debe tener un identificador estable:

user_id

Debe:

ser único;
permanecer estable durante la vida de la identidad;
utilizarse en referencias internas;
poder utilizarse en eventos;
utilizarse en auditoría;
no depender del email.

La identidad no debe depender de:

email
username
organization_id
role
6. User Attributes

El modelo conceptual incluye:

User
├── id
├── identity
├── status
├── created_at
├── updated_at
├── last_authenticated_at
├── version
└── metadata

Estos son atributos conceptuales, no todavía el schema físico definitivo.

El documento original expresamente advertía que el Blueprint no fijaba un esquema físico completo de User.

7. Email Identity

Email debe tratarse como concepto de dominio/Value Object:

Email
├── normalization
├── validation
└── equality

La implementación debe definir formalmente:

normalización;
comparación;
unicidad;
cambio;
verificación;
comportamiento después de un cambio.

Importante:

email ≠ user_id

El email puede cambiar; el user_id no.

8. User Lifecycle

Los estados consolidados son:

PENDING
ACTIVE
SUSPENDED
DISABLED

Modelo:

             ┌─────────┐
             │ PENDING │
             └────┬────┘
                  │
                  ▼
             ┌─────────┐
             │ ACTIVE  │
             └────┬────┘
                  │
          ┌───────┴────────┐
          ▼                ▼
     SUSPENDED          DISABLED
          │
          │
          ▼
       ACTIVE

Estos estados deben ser tratados como security state, no solamente como información administrativa.

9. User States
PENDING

Identidad creada pero no completamente activada.

ACTIVE

Identidad habilitada para autenticación y operaciones autorizadas.

SUSPENDED

Identidad temporalmente restringida por una condición de seguridad, administración, cumplimiento u otra política.

DISABLED

Identidad deshabilitada.

Una identidad DISABLED no debe poder utilizar Authentication normal.

10. User Lifecycle Rules
UMR-001 — Unique Identity

Cada User debe tener un identificador único.

UMR-002 — Security State Enforcement

El estado del User debe ser considerado durante Authentication y Authorization.

UMR-003 — Suspended User

Un User suspendido no debe obtener acceso normal mientras la suspensión esté vigente.

UMR-004 — Disabled User

Un User deshabilitado no debe autenticarse mediante mecanismos normales.

UMR-005 — Auditable State

Los cambios de estado deben ser auditables.

UMR-006 — No Physical Delete by Default

La eliminación física no debe ser el mecanismo normal para desactivar una identidad.

La retención definitiva queda determinada por las políticas de Data Governance/Privacy/Compliance.

11. User Creation

Flujo:

Create User
    │
    ▼
Validate Request
    │
    ▼
Normalize Identity
    │
    ▼
Check Uniqueness
    │
    ▼
Create User
    │
    ▼
Audit
    │
    ▼
Domain Event / Outbox

Cuando corresponda, la creación produce un evento de dominio.

El evento se publica mediante:

Domain State
      +
Transactional Outbox
      ↓
Event Platform
12. User Activation
PENDING
   │
   │ authorized activation
   ▼
ACTIVE

La activación puede estar vinculada a:

invitación;
verificación;
administración;
proceso de onboarding;
mecanismo autorizado de Identity.

Authentication/MFA determinan los controles de autenticación asociados.

13. User Suspension
ACTIVE
   │
   ▼
SUSPENDED

Puede originarse por:

administración;
seguridad;
comportamiento sospechoso;
cumplimiento;
policy;
detección de riesgo.

La razón debe registrarse para auditoría.

14. User Disable
ACTIVE
   │
   ▼
DISABLED

El estado DISABLED debe afectar al proceso de autenticación y al estado de seguridad de la identidad.

Según las reglas establecidas en ADR-033, una condición de suspensión/deshabilitación puede provocar la revocación de Sessions/Refresh Tokens cuando corresponda.

Esto ya no debe quedar como una simple decisión futura de arquitectura.

15. Organization

Una Organization representa una unidad organizacional y constituye la frontera de tenant cuando el modelo de despliegue/dominio la utiliza como tal.

Ejemplos:

Fitness Center
Nutrition Clinic
Professional Practice
Sports Team
EVOXA Enterprise

La organización es una frontera fundamental para:

acceso;
autorización;
datos;
memberships;
recursos;
políticas organizacionales.
16. Organization Attributes

Modelo conceptual:

Organization
├── id
├── name
├── status
├── created_at
├── updated_at
├── version
└── metadata

El schema físico definitivo pertenece a ESP-0001/11.

17. Organization Lifecycle

Estados conceptuales:

PENDING
ACTIVE
SUSPENDED

Modelo:

PENDING
   │
   ▼
ACTIVE
   │
   ▼
SUSPENDED
   │
   ▼
ACTIVE

Una Organization suspendida continúa existiendo.

Suspenderla no significa eliminar:

Users
Memberships
Historical Data
Audit Records
18. Organization Security State

Organization status forma parte del contexto de seguridad.

Conceptualmente:

Organization
      │
      ▼
Security State
      │
      ├── ACTIVE
      └── SUSPENDED

Una organización suspendida no debe proporcionar acceso organizacional normal aunque:

User = ACTIVE
Membership = ACTIVE

Es decir:

User ACTIVE
+
Membership ACTIVE
+
Organization SUSPENDED
=
No normal organizational access
19. Organization Creation

Flujo conceptual:

Create Organization
       │
       ▼
Validate
       │
       ▼
Create Organization
       │
       ▼
Establish Initial Membership
       │
       ▼
Configure Initial Access
       │
       ▼
Audit
       │
       ▼
OrganizationCreated

La creación inicial de Membership/Role debe utilizar los contratos definidos por Membership y Role/Permission.

20. Membership

La decisión arquitectónica queda consolidada:

User
  │
  │
Membership
  │
  ▼
Organization

Membership representa la relación entre:

User
+
Organization

y constituye un access boundary.

Esto es especialmente importante en el modelo multi-organización definido por ADR-043.

21. Multi-Organization Users

A diferencia del ESP original, ya no debemos dejar esta cuestión como pendiente.

El modelo consolidado es:

Global User
      │
      ├── Membership → Organization A
      │
      ├── Membership → Organization B
      │
      └── Membership → Organization C

Un User puede tener múltiples Memberships.

Cada Membership tiene su propio contexto organizacional.

Por tanto:

User
 ≠
Tenant Access

El acceso se obtiene mediante:

User
 ↓
Membership
 ↓
Organization
22. Membership as Access Boundary

La Membership es la frontera que conecta identidad y organización.

User
 │
 ▼
Membership
 │
 ├── Organization
 ├── Status
 ├── Roles
 └── Authorization Context

No debe asumirse que:

User exists

implica:

User can access Organization

Debe existir una Membership válida.

23. Membership Attributes

Modelo conceptual:

Membership
├── id
├── user_id
├── organization_id
├── status
├── joined_at
├── created_at
├── updated_at
└── version

Regla:

UNIQUE(user_id, organization_id)

No debe existir más de una Membership activa estructural para la misma relación User/Organization.

24. Membership Lifecycle

Estados:

INVITED
ACTIVE
SUSPENDED
REMOVED

Modelo:

INVITED
   │
   ▼
ACTIVE
   │
   ├── SUSPENDED
   │      │
   │      └── ACTIVE
   │
   └── REMOVED

La separación entre User y Membership es fundamental.

Por ejemplo:

User = ACTIVE
Membership = REMOVED

significa:

El usuario continúa existiendo, pero ya no pertenece a esa organización.

25. User vs Membership vs Organization

La semántica queda:

User
"¿Quién es?"

Membership
"¿En qué organización tiene relación?"

Organization
"¿Cuál es el contexto organizacional/tenant?"

Role
"¿Qué función tiene?"

Permission
"¿Qué puede hacer?"

Por tanto:

User
   ↓
Membership
   ↓
Organization
   ↓
Role
   ↓
Permission
26. Membership Roles

Una Membership puede tener múltiples roles:

Membership
 ├── Coach
 └── TeamManager

Los roles son scoped a la Membership cuando corresponda.

No existe jerarquía implícita entre roles.

El modelo completo de roles pertenece a ESP-0001/05.

27. Membership Suspension
ACTIVE
   │
   ▼
SUSPENDED

Una Membership suspendida no debe producir acceso autorizado dentro de la Organization.

La User puede continuar:

ACTIVE

y mantener otras Memberships activas.

Ejemplo:

User
 ├── Org A → ACTIVE
 └── Org B → SUSPENDED
28. Membership Removal
ACTIVE
   │
   ▼
REMOVED

Una Membership REMOVED no proporciona acceso.

La eliminación de una Membership debe provocar los cambios de security state/invalidation que correspondan a Sessions, Authorization Cache y Security Events.

Esto se deriva de los ADR-043, 050–059.

29. Tenant Context

El contexto organizacional debe resolverse antes de Authorization:

Request
   │
   ▼
Authenticated User
   │
   ▼
Membership
   │
   ▼
Organization
   │
   ▼
Tenant Context
   │
   ▼
Authorization

Conceptualmente:

TenantContext
├── organization_id
├── user_id
├── membership_id
├── roles
└── security_context
30. Active Organization Context

Cuando un User posee múltiples Memberships:

User
 ├── Org A
 ├── Org B
 └── Org C

la request debe establecer explícitamente el contexto organizacional.

Nunca se debe seleccionar arbitrariamente:

Org A

simplemente porque sea la primera organización encontrada.

Debe existir un contexto explícito y validado.

31. Tenant Context Validation

Antes de autorizar:

Requested Organization
          │
          ▼
Membership Exists?
          │
       ┌──┴──┐
       │     │
      YES    NO
       │     │
       ▼     ▼
 Validate   DENY
 Membership
       │
       ▼
Organization State
       │
       ▼
Authorization
32. Cross-Tenant Access

Regla fundamental:

Tenant A
   │
   │ request
   ▼
Tenant B

Resultado:

DENY

salvo que exista una capacidad cross-tenant explícita, autorizada, auditada y permitida por la Policy Engine.

Esto deriva directamente del modelo de Tenant Isolation consolidado en los ADR de Authorization.

33. Tenant Isolation

No se debe confiar en un:

organization_id

proporcionado arbitrariamente por el cliente.

El sistema debe validar:

Authenticated User
        +
Membership
        +
Requested Organization
        +
Resource Organization

antes de permitir acceso.

34. Organization Administration

Las capacidades conceptuales incluyen:

Create Organization
View Organization
Update Organization
Suspend Organization
Reactivate Organization

Manage Members
Invite Members
Suspend Members
Remove Members

La autorización concreta pertenece al Authorization Model.

35. User Administration

Identity debe soportar:

Create User
View User
Update User
Activate User
Suspend User
Disable User
Reactivate User
View Memberships

Pero el alcance depende del actor.

36. Administration Boundaries

Debemos distinguir tres niveles:

Self-Service
User
  ↓
Own Identity
Organization Administration
Organization Administrator
  ↓
Organization Members
Platform Administration
Platform Administrator
  ↓
Platform Identity

No debe existir elevación implícita:

Organization Admin
      ≠
Platform Admin
37. Sensitive Identity Changes

Se consideran sensibles:

Email Change
Password Change
MFA Change
Account Status Change
Membership Change
Role Change
Permission Change

Estas operaciones deben estar sujetas a controles superiores de:

Authentication;
Authorization;
MFA;
Policy;
Audit;
Security Events.
38. Invitation

Invitation permite incorporar una identidad a una Organization.

Modelo:

Organization
      │
      ▼
 Invitation
      │
      ▼
Membership
      │
      ▼
 User

Una Invitation no debe convertirse automáticamente en acceso sin completar el proceso requerido.

39. Invitation Model

Modelo conceptual:

Invitation
├── invitation_id
├── organization_id
├── invited_email
├── status
├── expires_at
├── created_at
└── accepted_at

El modelo físico definitivo pertenece a Persistence.

40. Invitation Lifecycle
PENDING
   │
   ├── ACCEPTED
   ├── EXPIRED
   └── REVOKED

Regla:

EXPIRED → cannot accept
REVOKED → cannot accept
41. Existing User Invitation

Si el User ya existe:

Existing User
      │
      ▼
Invitation
      │
      ▼
Accept
      │
      ▼
Membership

No debe crearse una nueva identidad.

42. New User Invitation

Si el User no existe:

Invitation
      │
      ▼
Identity Creation
      │
      ▼
User
      │
      ▼
Membership

La creación de User debe respetar Authentication, Verification y Security Policy.

43. Invitation Security

Una Invitation debe ser:

purpose-bound;
de duración limitada;
revocable;
de uso controlado;
no reutilizable después de aceptación;
auditable.

No debe almacenar ni exponer secretos innecesarios.

44. Organization Membership Rules
OMR-001

Membership debe referenciar un User válido.

OMR-002

Membership debe referenciar una Organization válida.

OMR-003

No debe existir duplicación User + Organization.

OMR-004

Membership REMOVED no proporciona acceso.

OMR-005

Membership SUSPENDED no proporciona acceso normal.

OMR-006

Membership debe respetar Organization state.

OMR-007

Membership changes deben ser auditables.

OMR-008

Membership revocation debe invalidar el contexto de autorización correspondiente.

45. Organization Suspension

Cuando:

Organization = SUSPENDED

sus Users no son eliminados.

Ejemplo:

Organization
   │
   ├── User A → identity remains
   ├── User B → identity remains
   └── User C → identity remains

pero el acceso organizacional debe quedar restringido según Security Policy.

46. Organization Deletion

La eliminación física de una Organization no debe ser el mecanismo normal de lifecycle.

Debe existir una política explícita de:

retention;
compliance;
historical references;
audit;
event preservation;
privacy/deletion requests.

Los detalles pertenecen a Data Governance/Privacy/Compliance.

47. Events

Eventos conceptuales:

UserCreated
UserUpdated
UserActivated
UserSuspended
UserDisabled
UserReactivated

OrganizationCreated
OrganizationUpdated
OrganizationSuspended
OrganizationReactivated

MembershipCreated
MembershipActivated
MembershipSuspended
MembershipRemoved

InvitationCreated
InvitationAccepted
InvitationExpired
InvitationRevoked

Estos nombres son event catalog candidates; el contrato definitivo se establecerá en ESP-0001/12.

El principio ya está decidido:

Domain State
     +
Outbox
     ↓
Event Platform
48. Event Principles

Los eventos:

representan hechos ocurridos;
son inmutables;
son versionados;
no son commands;
deben permitir consumidores desacoplados;
no deben contener secretos.

Ejemplo:

MembershipRemoved
{
    membership_id,
    user_id,
    organization_id,
    timestamp,
    actor,
    correlation_id
}

El payload definitivo deberá aplicar minimización de datos.

49. Audit

Las operaciones relevantes deben permitir responder:

WHO
WHAT
WHEN
ORGANIZATION
TARGET
RESULT
CORRELATION

Ejemplo:

Actor:
user-123

Action:
MEMBERSHIP_REMOVED

Target:
membership-456

Organization:
org-789

Result:
SUCCESS
50. Privacy

Identity aplica:

Privacy by Design

Debe minimizarse la exposición de:

email;
authentication metadata;
security metadata;
internal identifiers;
membership metadata;
personal information.

No deben aparecer secretos en:

Logs
Events
Traces
Audit
AI Context
51. Data Ownership

Identity es propietario de:

Users
Organizations
Memberships
Identity Lifecycle
Identity Security State

No es propietario de:

Training Programs
Training Sessions
Nutrition Plans
Assessments
Recovery Data
Human Digital Twin
AI Recommendations
Billing

Cada dominio debe mantener sus propias entidades y reglas.

52. Repository Boundary

Los repositorios conceptuales son:

UserRepository
OrganizationRepository
MembershipRepository
InvitationRepository

Ejemplo:

UserRepository
├── create
├── get_by_id
├── get_by_email
├── update
└── exists

Los contratos definitivos pertenecen a Persistence.

53. Module Boundary

No se permite:

Organization Module
       ↓
direct User table access

ni:

Training Module
       ↓
direct Membership table access

La interacción debe producirse mediante:

Application Contracts
Queries
Commands
Domain Events
Integration Events

según corresponda.

54. Transactions

Las operaciones que cambian múltiples agregados pueden requerir una transacción de aplicación.

Ejemplo:

Create Organization
       +
Create Initial Membership

Conceptualmente:

BEGIN

Create Organization
Create Membership
Create Outbox Event(s)

COMMIT

o:

ROLLBACK

si falla.

El diseño físico de transacciones pertenece a ESP-0001/11.

55. Concurrency

Las operaciones administrativas deben protegerse contra actualizaciones concurrentes.

Ejemplo:

Admin A → Suspend User
Admin B → Reactivate User

No debe producirse una sobrescritura silenciosa.

El modelo admite:

version

como mecanismo conceptual de optimistic concurrency.

La estrategia física se especificará posteriormente.

56. Security State Propagation

Los cambios de:

User
Membership
Organization

pueden modificar el contexto de Authorization.

Por eso deben integrarse con:

Security Epoch
Entity Version
Policy Version
Cache Invalidation
Session State
Audit
Security Events

Ejemplo:

Membership Revoked
       │
       ├── Security State
       ├── Security Epoch
       ├── Authorization Invalidation
       ├── Session handling
       ├── Audit
       └── Event

Esto conecta directamente ESP-0001/04 con los ADR-050 → 059.

57. Error Handling

Las APIs utilizarán el error contract estándar:

{
  "error": {
    "code": "IDENTITY_ERROR",
    "message": "Human readable message",
    "correlation_id": "..."
  }
}

Ejemplos conceptuales:

USER_NOT_FOUND
USER_ALREADY_EXISTS
USER_SUSPENDED
ORGANIZATION_NOT_FOUND
ORGANIZATION_SUSPENDED
MEMBERSHIP_NOT_FOUND
MEMBERSHIP_ALREADY_EXISTS
MEMBERSHIP_SUSPENDED
INVITATION_EXPIRED
INVITATION_REVOKED
TENANT_ACCESS_DENIED

Los códigos definitivos pertenecen a ESP-0001/10.

58. API Boundary

Las operaciones posteriormente podrán exponerse mediante:

/api/v1/users
/api/v1/users/{id}

/api/v1/organizations
/api/v1/organizations/{id}

/api/v1/organizations/{id}/members

/api/v1/organizations/{id}/invitations

Pero estos no son todavía contratos definitivos.

Los contratos oficiales pertenecen a ESP-0001/10.

59. Testing Requirements

Como mínimo deben probarse:

User
Create
Duplicate
Update
Activate
Suspend
Disable
Reactivate
Invalid Transition
Organization
Create
Duplicate
Update
Suspend
Reactivate
Invalid Transition
Membership
Create
Duplicate
Activate
Suspend
Reactivate
Remove
Access after removal
Multi-Organization
User → Org A
User → Org B
Correct Context
Wrong Context
Missing Context
Cross-Tenant Attempt
Invitation
Create
Accept
Expire
Revoke
Reuse
Duplicate
Existing User
New User
Security
Unauthorized
Forbidden
Cross-Tenant
Wrong Membership
Suspended User
Disabled User
Suspended Organization
Suspended Membership
60. Critical Security Test Cases

Debemos garantizar específicamente:

ACTIVE User
+
REMOVED Membership
=
DENY
ACTIVE User
+
ACTIVE Membership
+
SUSPENDED Organization
=
DENY
User in Organization A
+
Resource in Organization B
=
DENY
User without Membership
+
Organization Resource
=
DENY
Organization Admin
+
Platform Resource
=
DENY

salvo autorización explícita de plataforma.

61. Proposed Component Structure

La estructura derivada puede mantenerse:

identity/
│
├── domain/
│   ├── entities/
│   │   ├── user.py
│   │   ├── organization.py
│   │   ├── membership.py
│   │   └── invitation.py
│   │
│   ├── value_objects/
│   │   ├── user_id.py
│   │   ├── organization_id.py
│   │   └── email.py
│   │
│   └── events/
│
├── application/
│   ├── users/
│   ├── organizations/
│   ├── memberships/
│   └── invitations/
│
├── api/
│   ├── users/
│   ├── organizations/
│   └── invitations/
│
└── infrastructure/
    ├── persistence/
    ├── messaging/
    └── audit/

Esto sigue siendo Derived Design, no una imposición directa del Blueprint.

62. Architectural Boundary

La frontera fundamental queda:

                 IDENTITY
                    │
          ┌─────────┴─────────┐
          │                   │
        WHO                 WHERE
          │                   │
        User            Organization
          │                   │
          └─────────┬─────────┘
                    │
                Membership
                    │
                    ▼
            Security Context
                    │
                    ▼
              AUTHORIZATION
                    │
                    ▼
             BUSINESS DOMAIN

Identity determina:

quién es el actor y en qué contexto organizacional puede actuar.

El dominio de negocio determina:

qué significa esa operación y cuáles son sus reglas específicas.

63. ADR Consolidation

Aquí hay una corrección importante respecto al documento original.

La versión 1.0.0 tenía una sección de “ADR Candidates” que proponía decisiones como:

ADR-IDENTITY-002 Multi-Organization Users
ADR-IDENTITY-016 User States
ADR-IDENTITY-017 Organization States
ADR-IDENTITY-018 Membership States
...

Eso ya está obsoleto.

Esos números no corresponden al catálogo actual de ADRs.

Las decisiones relevantes ya fueron consolidadas principalmente mediante:

ADR-IDENTITY-033
Account Lifecycle / Lockout

ADR-IDENTITY-043
Multi-Organization Identity

ADR-IDENTITY-044
Membership Roles / Delegation / Fine-Grained Authorization

ADR-IDENTITY-045+
Authorization Policy Model

ADR-IDENTITY-050–059
Authorization Cache / Freshness / Criticality

ADR-IDENTITY-068
Identity Domain Integration & Module Boundaries

ADR-IDENTITY-069
Identity Module Dependency Matrix

Por tanto, no debemos crear ADR-070 únicamente para volver a decidir User/Organization/Membership.

64. Traceability
BP-0002
   │
   ▼
Identity Domain
   │
   ▼
ESP-0001
   │
   ├── 01 Domain Model
   ├── 02 Architecture
   ├── 03 Authentication & Authorization
   └── 04 User & Organization
          │
          ├── User
          ├── Organization
          ├── Membership
          ├── Invitation
          └── Tenant Context

Relaciones principales:

Capacidad	ADR / ESP
User lifecycle	ADR-033
Multi-Organization	ADR-043
Membership	ADR-043/044
Roles scoped to Membership	ADR-044
Tenant Isolation	ADR-005/043/045+
Authorization	ESP-0001/03
Session Security	ADR-003
MFA Security	ADR-023/024
Cache Invalidation	ADR-050–056
Criticality	ADR-058/059
Events	ADR-007/018–022
Architecture Boundaries	ADR-068/069
API	ESP-0001/10
Persistence	ESP-0001/11
Events Integration	ESP-0001/12
Testing	ESP-0001/13
65. Implementation Readiness

Con esta consolidación:

Área	Estado
User Model	🟢
User Lifecycle	🟢
Organization Model	🟢
Organization Lifecycle	🟢
Multi-Organization	🟢
Membership	🟢
Membership Lifecycle	🟢
Tenant Isolation	🟢
Tenant Context	🟢
User Administration	🟢
Organization Administration	🟢
Invitation	🟢
Audit	🟢
Privacy	🟢
Event Principles	🟢
Persistence Schema	⏳ ESP-11
API Contracts	⏳ ESP-10
Event Contracts	⏳ ESP-12
Test Implementation	⏳ ESP-13
66. Resultado

Con ESP-0001/04 v1.1.0, la relación Identity queda mucho más sólida:

                    IDENTITY
                       │
                       ▼
                     User
                       │
               ┌───────┼───────┐
               │       │       │
               ▼       ▼       ▼
             Org A   Org B   Org C
               ▲       ▲       ▲
               │       │       │
          Membership Membership Membership
               │       │       │
               └───────┼───────┘
                       │
                       ▼
                Authorization
                       │
             ┌─────────┼─────────┐
             ▼         ▼         ▼
            RBAC      ABAC   Ownership
                       │
                       ▼
                 Policy Engine
                       │
                       ▼
                  Criticality
                       │
                       ▼
                    Freshness
                       │
                       ▼
                    Decision
Mi evaluación

ESP-0001/04 original: 🟡

ESP-0001/04 v1.1.0: 🟢 arquitectónicamente consolidado.

La principal mejora respecto al documento que estaba en GitHub es que ya no dejamos como “pendiente” el modelo multi-organización: User global + Membership scoped a Organization queda alineado con ADR-043, y Membership pasa a ser explícitamente una frontera de acceso.

También corregimos la antigua lista de ADR candidates para que no genere conflictos con el catálogo real de ADR-001 → ADR-069.

El siguiente documento lógico es ESP-0001/05 — Identity Roles & Permissions, donde ahora sí podemos cerrar formalmente Role, Permission, RolePermission, roles globales vs organizacionales, asignación a Membership, revocación y cómo se integra RBAC con ABAC, Delegation y Policy Engine.
