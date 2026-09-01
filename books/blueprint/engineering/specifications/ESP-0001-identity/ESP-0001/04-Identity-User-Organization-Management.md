ESP-0001 / 04 — Identity User & Organization Management

Document ID: ESP-0001/04
Module: Identity
Version: 1.0.0
Status: Draft
Parent: ESP-0001 — Identity
Previous Specifications:

ESP-0001/01 — Identity Domain Model
ESP-0001/02 — Identity Architecture
ESP-0001/03 — Identity Authentication & Authorization

Architecture Parent: BP-0002 — Reference Architecture
Repository: evoxa-platform
Target Path: apps/api/app/domains/identity/

1. Purpose

Este documento define la arquitectura funcional y técnica para la gestión de usuarios y organizaciones dentro del módulo Identity de EVOXA.

Comprende:

creación y gestión de usuarios;
activación y desactivación;
suspensión;
gestión del ciclo de vida;
organizaciones;
membresías;
invitaciones;
pertenencia organizacional;
contexto de tenant;
administración de identidades;
relación entre usuarios y organizaciones;
auditoría de cambios.

Identity proporciona la identidad y el contexto de seguridad, mientras que los dominios de negocio mantienen sus propias reglas y recursos.

La arquitectura de EVOXA establece Identity como un dominio central y contempla explícitamente Tenant Management como capability de plataforma.

2. Scope

Este documento cubre:

User
Organization
Membership
Invitation
User Lifecycle
Organization Lifecycle
Membership Lifecycle
Tenant Context
User Administration
Organization Administration
Identity Audit

No cubre en profundidad:

Authentication
Authorization
Roles
Permissions
MFA
API Tokens
Training
Nutrition
Recovery
AI
Billing

Authentication y Authorization ya fueron tratados en:

ESP-0001/03 — Identity Authentication & Authorization

Roles y Permissions serán desarrollados posteriormente.

3. Architectural Model

La relación fundamental será:

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

Un usuario representa una identidad.

Una organización representa un contexto organizacional/tenant.

Una membership representa la relación entre ambos.

4. User
4.1 Definition

User representa una identidad individual dentro de EVOXA.

El User debe ser independiente del dominio de negocio en el que participe.

Por ejemplo:

User
 │
 ├── Coach
 ├── Nutritionist
 ├── Physiotherapist
 └── Team Manager

El rol no forma parte de la identidad básica del usuario; es una dimensión de autorización.

La API Architecture contempla precisamente estos perfiles profesionales dentro del modelo de acceso: Coach, Nutritionist, Physiotherapist, Team Manager y Admin.

5. User Identity

La identidad debe disponer de un identificador estable:

user_id

Este identificador:

no debe cambiar durante la vida de la cuenta;
debe utilizarse como referencia interna;
debe poder aparecer en eventos;
debe utilizarse para auditoría;
no debe depender del email.

Conceptualmente:

User
├── id
├── identity
├── status
└── lifecycle metadata
6. User Attributes

Propuesta de atributos:

Campo	Descripción
id	Identificador único
email	Identificador de contacto/autenticación
status	Estado de la cuenta
created_at	Fecha de creación
updated_at	Última modificación
last_authenticated_at	Última autenticación
version	Versión de entidad
metadata	Metadatos controlados

Nota: El Blueprint no define un esquema físico completo de User. Estos campos son una propuesta de diseño derivada y deberán consolidarse antes de implementación.

7. Email Identity

El email deberá tratarse como un Value Object o concepto de dominio equivalente.

Email
├── normalization
├── validation
└── equality

El sistema deberá definir explícitamente:

normalización;
comparación;
unicidad;
cambio de email;
verificación;
comportamiento cuando cambia.

Estas reglas todavía deben quedar formalizadas en una especificación posterior.

8. User Lifecycle

Propuesta inicial:

                 ┌─────────┐
                 │ PENDING │
                 └────┬────┘
                      │ activate
                      ▼
                 ┌─────────┐
                 │ ACTIVE  │
                 └────┬────┘
                      │
             ┌────────┴────────┐
             │                 │
          suspend            deactivate
             │                 │
             ▼                 ▼
        ┌──────────┐      ┌──────────┐
        │SUSPENDED │      │ DISABLED │
        └────┬─────┘      └──────────┘
             │
          reactivate
             │
             ▼
         ┌────────┐
         │ ACTIVE │
         └────────┘

Los nombres exactos de estados deben validarse antes de convertirlos en contrato de API.

9. User States
PENDING

Cuenta creada pero todavía no completamente activada.

ACTIVE

Cuenta disponible para autenticación y operaciones autorizadas.

SUSPENDED

Cuenta temporalmente bloqueada.

DISABLED

Cuenta desactivada.

La transición de estados debe ser explícita y auditable.

10. User Lifecycle Rules
UMR-001

Un usuario debe tener un id único.

UMR-002

Un usuario no puede autenticarse si su estado no permite autenticación.

UMR-003

La suspensión debe impedir acceso según las políticas de Identity.

UMR-004

Los cambios de estado deben ser auditables.

UMR-005

La eliminación física de identidad no debe ser el mecanismo normal para desactivar una cuenta.

Nota: UMR-005 es una recomendación de diseño. La política definitiva de retención/eliminación deberá alinearse con Privacy y Compliance.

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
Create Audit Record
    │
    ▼
Publish UserCreated

La creación debe producir un evento de dominio cuando represente una acción empresarial significativa, siguiendo Event Architecture.

12. User Activation
PENDING
   │
   │ activation
   ▼
ACTIVE

La activación puede ocurrir mediante:

invitación aceptada;
verificación de identidad;
administración;
otro mecanismo autorizado.

El mecanismo exacto deberá definirse en las especificaciones de Authentication.

13. User Suspension
ACTIVE
   │
   ▼
SUSPENDED

Una suspensión puede producirse por:

acción administrativa;
seguridad;
política organizacional;
cumplimiento;
comportamiento sospechoso.

La razón de suspensión debe ser auditable, pero no necesariamente expuesta al usuario final.

14. User Deactivation
ACTIVE
   │
   ▼
DISABLED

La desactivación debe provocar, según la política definitiva:

Disable Authentication
+
Revoke Sessions
+
Audit

La revocación automática de todas las sesiones es una decisión derivada recomendada que debe consolidarse en ADR.

15. Organization
15.1 Definition

Una Organization representa una unidad organizacional y, cuando corresponda, un tenant boundary.

Ejemplos:

EVOXA Enterprise
Fitness Center
Professional Practice
Nutrition Clinic
Sports Team

La arquitectura de plataforma contempla Tenant Management como capability transversal.

16. Organization Attributes

Propuesta:

Campo	Descripción
id	Identificador único
name	Nombre
status	Estado
created_at	Creación
updated_at	Modificación
version	Control de versión
metadata	Metadatos controlados

El modelo físico definitivo queda pendiente.

17. Organization Lifecycle

Propuesta:

┌──────────┐
│ PENDING  │
└────┬─────┘
     │ activate
     ▼
┌──────────┐
│  ACTIVE  │
└────┬─────┘
     │ suspend
     ▼
┌───────────┐
│ SUSPENDED │
└─────┬─────┘
      │ reactivate
      ▼
   ACTIVE

Una organización desactivada/suspendida puede afectar a todas sus memberships, pero esta relación deberá definirse explícitamente en las reglas de negocio.

18. Organization Creation

Flujo:

Create Organization
       │
       ▼
Validate
       │
       ▼
Create Organization
       │
       ▼
Create Initial Membership
       │
       ▼
Assign Initial Role
       │
       ▼
Audit
       │
       ▼
OrganizationCreated

La asignación automática de una membership inicial y rol inicial es una propuesta de diseño y debe validarse con el modelo de Roles.

19. Membership

Membership representa:

User
  ↕
Organization

No debe confundirse con:

Role

ni con:

Permission

La separación será:

User
 │
 └── Membership
       │
       ├── Organization
       │
       └── Authorization Context
20. Why Membership Exists

La membership permite representar:

User A
 ├── Organization X
 │      └── Role A
 │
 └── Organization Y
        └── Role B

Esto proporciona un modelo flexible para organizaciones y tenants.

Importante: la capacidad multi-organización del usuario es una decisión de diseño derivada; el Blueprint establece tenant management y aislamiento, pero no fija en los documentos revisados una cardinalidad definitiva.

Por ello:

ADR-IDENTITY-002 — Multi-Organization Users

deberá decidir esta cuestión antes de implementar.

21. Membership Attributes

Propuesta:

Campo	Descripción
id	Identificador
user_id	Usuario
organization_id	Organización
status	Estado
joined_at	Fecha de incorporación
created_at	Creación
updated_at	Modificación
version	Versión

La relación:

(user_id, organization_id)

debería ser única.

22. Membership Lifecycle

Propuesta:

INVITED
   │
   │ accept
   ▼
ACTIVE
   │
   ├── suspend
   │
   ▼
SUSPENDED
   │
   │ reactivate
   ▼
ACTIVE
   │
   │ remove
   ▼
REMOVED

Esto permite distinguir:

User disabled

de:

User removed from organization

Son conceptos diferentes.

23. User vs Membership

Esta distinción será fundamental para EVOXA.

User
"¿Quién es?"
Membership
"¿A qué organización pertenece?"
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
24. Invitation

Para incorporar usuarios a organizaciones se propone una entidad/concepto Invitation.

Organization
      │
      ▼
 Invitation
      │
      ▼
 User

Una invitación puede contener:

invitation_id
organization_id
invited_email
status
expires_at
created_at
accepted_at

El modelo definitivo deberá decidir si la invitación es:

una entidad persistente;
un agregado independiente;
o un proceso temporal asociado a Membership.
25. Invitation Lifecycle

Propuesta:

PENDING
   │
   ├── accept → ACCEPTED
   │
   ├── expire → EXPIRED
   │
   └── revoke → REVOKED

Una invitación expirada o revocada no debe permitir crear una membership.

26. Invitation Flow
Organization Admin
       │
       ▼
Invite User
       │
       ▼
Create Invitation
       │
       ▼
Send Notification
       │
       ▼
User Receives Invitation
       │
       ▼
Accept
       │
       ▼
Create / Resolve User
       │
       ▼
Create Membership
       │
       ▼
Activate Membership

El envío puede integrarse con la capability de Email/Notifications de la plataforma.

27. Existing User Invitation

Si el usuario ya existe:

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

No se debe crear una segunda identidad.

28. New User Invitation

Si no existe:

Invitation
      │
      ▼
Registration / Identity Creation
      │
      ▼
User
      │
      ▼
Membership

La creación de la identidad debe respetar los controles de Authentication.

29. Organization Membership Rules
OMR-001

Una membership debe referenciar un User válido.

OMR-002

Una membership debe referenciar una Organization válida.

OMR-003

Una combinación User + Organization no debe duplicarse.

OMR-004

Una membership REMOVED no representa acceso activo.

OMR-005

Una membership SUSPENDED no debe proporcionar acceso autorizado mientras permanezca suspendida.

OMR-006

Los cambios de membership deben ser auditables.

30. Organization Isolation

La organización representa una frontera lógica:

Organization A
 ├── Users
 ├── Resources
 └── Data

Organization B
 ├── Users
 ├── Resources
 └── Data

El sistema debe impedir acceso cross-tenant no autorizado.

La arquitectura de seguridad establece explícitamente:

tenant isolation;
organization membership;
ownership;
cross-tenant restrictions.
31. Tenant Context Resolution

Durante una request:

Request
  │
  ▼
Authenticated User
  │
  ▼
Membership Resolution
  │
  ▼
Organization Context
  │
  ▼
Authorization

El contexto puede representarse conceptualmente como:

TenantContext
├── organization_id
├── user_id
├── membership_id
├── roles
└── security claims
32. Multiple Organization Context

Si se aprueba el modelo multi-organización:

User
 │
 ├── Organization A
 │
 ├── Organization B
 │
 └── Organization C

la request deberá indicar o resolver explícitamente el contexto organizacional.

Nunca se debe asumir arbitrariamente una organización cuando el usuario posee varias.

Esto requiere una decisión arquitectónica formal.

33. Organization Administration

Las operaciones organizacionales deberán incluir:

Create Organization
View Organization
Update Organization
Suspend Organization
Reactivate Organization
Manage Members
Invite Members
Remove Members

La autorización para estas operaciones dependerá del modelo de roles y permisos.

34. User Administration

Identity deberá permitir operaciones como:

Create User
View User
Update User
Activate User
Suspend User
Disable User
Restore User
View Memberships

No todas estarán disponibles para cualquier usuario.

Ejemplo:

Regular User
   → update own profile

Organization Administrator
   → manage organization members

Platform Administrator
   → platform-level administration

Los permisos exactos pertenecen al modelo RBAC.

35. Self-Service vs Administration

Debemos distinguir:

Self-Service

El usuario administra su propia identidad.

User
 └── Own Profile
Organization Administration

Un administrador gestiona miembros de su organización.

Org Admin
 └── Organization Members
Platform Administration

Un administrador de EVOXA gestiona capacidades de plataforma.

Platform Admin
 └── Platform Identity

Esta separación evita privilegios excesivos.

36. Ownership Boundary

Un usuario no obtiene acceso administrativo global solamente por ser miembro de una organización.

Ejemplo:

User
 └── Organization A

no implica:

Organization B

ni:

Platform Administration

La autorización siempre debe comprobar el contexto correspondiente.

37. User Update

Flujo:

Update User
    │
    ▼
Authenticate
    │
    ▼
Authorize
    │
    ▼
Validate Changes
    │
    ▼
Persist
    │
    ├── Audit
    └── UserUpdated

Los cambios sensibles deben generar eventos/auditoría específicos.

38. Sensitive Identity Changes

Se consideran operaciones sensibles:

Email Change
Password Change
MFA Change
Account Status Change
Membership Change
Role Change
Permission Change

Estas operaciones deben tener controles de autorización superiores a una actualización ordinaria.

39. Organization Update

Flujo:

Update Organization
      │
      ▼
Authentication
      │
      ▼
Organization Authorization
      │
      ▼
Validate
      │
      ▼
Persist
      │
      ├── Audit
      └── OrganizationUpdated
40. Removing Membership

Eliminar una membership:

Membership ACTIVE
       │
       ▼
Authorization
       │
       ▼
Remove
       │
       ├── Revoke relevant sessions/context
       ├── Audit
       └── MembershipRemoved

La revocación de sesiones asociadas al contexto organizacional es una decisión derivada y debe formalizarse.

41. Organization Suspension

Cuando una organización es suspendida:

Organization
      │
      ▼
SUSPENDED
      │
      ├── Members remain identities
      │
      └── Organization access denied

Esto es importante:

Suspender una organización no significa necesariamente eliminar sus usuarios.

El User y la Membership son conceptos independientes.

42. Organization Deletion

No se recomienda tratar una organización como un simple registro que puede borrarse físicamente.

Por razones de:

auditoría;
trazabilidad;
cumplimiento;
referencias históricas;
eventos;

deberá existir una política explícita de lifecycle/retention.

La política definitiva pertenece a Data Governance / Security / Compliance.

43. Events

Eventos propuestos:

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

Estos nombres son Derived Design.

El principio de eventos sí está establecido en Event Architecture: los eventos representan hechos ocurridos y deben ser inmutables.

44. UserCreated Event

Conceptualmente:

UserCreated
{
    user_id,
    timestamp,
    actor,
    tenant_id,
    correlation_id
}

El payload definitivo debe minimizar información sensible.

45. MembershipCreated Event
MembershipCreated
{
    membership_id,
    user_id,
    organization_id,
    timestamp,
    actor,
    correlation_id
}

Esto permite que otros componentes reaccionen sin acceder directamente a tablas internas.

46. Event Consumers

Ejemplo:

Identity
   │
   ▼
MembershipCreated
   │
   ▼
Event Platform
   │
   ├── Audit
   ├── Analytics
   ├── Notifications
   └── Other Services

Esto respeta el desacoplamiento requerido por la arquitectura Event-Driven.

47. Audit Requirements

Cada cambio relevante deberá poder responder:

WHO
WHAT
WHEN
ORGANIZATION
TARGET
RESULT
CORRELATION

Ejemplo:

WHO:
user-123

WHAT:
MEMBERSHIP_REMOVED

TARGET:
membership-456

ORGANIZATION:
org-789

WHEN:
timestamp

RESULT:
SUCCESS

La arquitectura establece auditabilidad para operaciones críticas.

48. Privacy

Identity debe aplicar Privacy by Design.

No se debe exponer innecesariamente:

Email
Authentication metadata
Security metadata
MFA data
Internal identifiers

Además, los eventos y logs deben minimizar datos personales.

La Reference Architecture establece Privacy by Design como principio arquitectónico.

49. Data Ownership

Identity es propietario de:

Users
Organizations
Memberships
Identity Lifecycle
Identity Credentials
Sessions
Identity Security

No es propietario de:

Training Programs
Nutrition Plans
Assessments
Recovery Data
Human Digital Twin
AI Recommendations

Cada dominio debe mantener su propio modelo y datos.

50. Repository Model

Interfaces propuestas:

UserRepository
OrganizationRepository
MembershipRepository
InvitationRepository

Ejemplo conceptual:

UserRepository
├── create()
├── get_by_id()
├── get_by_email()
├── update()
└── exists()

Los métodos concretos deberán definirse en Persistence Specification.

51. Transaction Boundaries

Operaciones como:

Create Organization
+
Create Initial Membership

pueden requerir una única transacción de aplicación.

Ejemplo:

BEGIN
  Create Organization
  Create Membership
  Commit

Si falla:

ROLLBACK

El diseño transaccional definitivo deberá documentarse en la especificación de persistence.

52. Concurrency

Identity debe contemplar modificaciones concurrentes.

Ejemplo:

Admin A → Suspend User
Admin B → Activate User

No se debe permitir que una actualización posterior sobrescriba silenciosamente una decisión anterior.

El campo conceptual:

version

puede utilizarse para optimistic concurrency control.

Esto es una propuesta de diseño.

53. API Boundary

Las operaciones descritas aquí posteriormente serán expuestas mediante API Contracts.

Ejemplos:

POST   /api/v1/users
GET    /api/v1/users/{id}
PATCH  /api/v1/users/{id}

POST   /api/v1/organizations
GET    /api/v1/organizations/{id}
PATCH  /api/v1/organizations/{id}

GET    /api/v1/organizations/{id}/members
POST   /api/v1/organizations/{id}/members

POST   /api/v1/organizations/{id}/invitations

Estos endpoints son propuesta inicial y no deben considerarse todavía contratos oficiales.

API Architecture exige que los contratos sean definidos antes de la implementación.

54. Error Handling

Las operaciones deberán utilizar el estándar:

{
  "error": {
    "code": "IDENTITY_ERROR",
    "message": "Human readable message",
    "correlation_id": "..."
  }
}

Este formato está establecido por API Architecture.

Ejemplos de códigos:

USER_NOT_FOUND
USER_ALREADY_EXISTS
USER_SUSPENDED
ORGANIZATION_NOT_FOUND
MEMBERSHIP_NOT_FOUND
MEMBERSHIP_ALREADY_EXISTS
INVITATION_EXPIRED
INVITATION_REVOKED
TENANT_ACCESS_DENIED

Los códigos definitivos pertenecerán a API Contracts.

55. Security Rules

Las operaciones administrativas deberán aplicar:

Authentication
       ↓
Tenant Context
       ↓
RBAC
       ↓
ABAC
       ↓
Ownership / Organization Policy
       ↓
Operation

Nunca:

Request
  ↓
Database

sin autorización.

Esto sigue Zero Trust y Least Privilege.

56. Observability

User y Organization operations deberán generar:

Logs
Metrics
Traces
Audit
Events

con:

trace_id
span_id
correlation_id

La arquitectura de observabilidad exige trazabilidad distribuida y correlación entre servicios.

57. Testing

Debe probarse:

User
Create
Update
Activate
Suspend
Disable
Reactivate
Duplicate
Invalid state transition
Organization
Create
Update
Suspend
Reactivate
Duplicate
Membership
Create
Duplicate
Suspend
Remove
Reactivate
Cross-tenant protection
Invitation
Create
Accept
Expire
Revoke
Reuse
Duplicate
Security
Unauthorized
Forbidden
Cross-tenant
Wrong organization
Wrong ownership

La estrategia global de testing de EVOXA exige pruebas unitarias, integración, API, seguridad y aislamiento.

58. Proposed Component Structure

La implementación podría organizarse así:

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
│   ├── policies/
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

Esto sigue siendo Derived Design.

59. Important Architectural Boundary

Una de las reglas más importantes que debemos conservar:

Identity
   │
   ├── WHO
   ├── WHERE
   └── AUTHORIZATION CONTEXT
   │
   ▼
Business Domains
   │
   ├── WHAT
   ├── HOW
   └── BUSINESS RULES

Por ejemplo, Identity puede decir:

"User 123 pertenece a Organization A."

Pero no debe decidir:

"User 123 puede modificar el entrenamiento porque el entrenamiento tiene estas características."

Esa última decisión pertenece al dominio Training.

60. Traceability
BP-0002
   │
   ▼
Identity Module
   │
   ▼
ESP-0001
   │
   ├── 01 Domain Model
   │
   ├── 02 Architecture
   │
   ├── 03 Authentication & Authorization
   │
   └── 04 User & Organization Management
           │
           ├── User
           ├── Organization
           ├── Membership
           └── Invitation

Esto mantiene la trazabilidad establecida en ARCHITECTURE-MAP.md.

61. ADR Candidates

Este documento introduce varias decisiones que debemos NO congelar todavía:

ADR	Decisión
ADR-IDENTITY-002	¿Usuario multi-organización?
ADR-IDENTITY-016	Estados definitivos de User
ADR-IDENTITY-017	Estados definitivos de Organization
ADR-IDENTITY-018	Estados de Membership
ADR-IDENTITY-019	Modelo de Invitations
ADR-IDENTITY-020	Organización = Tenant boundary
ADR-IDENTITY-021	Política de eliminación/retención
ADR-IDENTITY-022	Revocación de sesiones al suspender usuario
ADR-IDENTITY-023	Revocación de sesiones al eliminar membership
ADR-IDENTITY-024	Optimistic concurrency
ADR-IDENTITY-025	Unicidad de email
62. Implementation Readiness

Con esto ya tenemos:

ESP-0001
│
├── 01 — Domain Model                    ✅
├── 02 — Architecture                   ✅
├── 03 — Authentication & Authorization  ✅
└── 04 — User & Organization Management  🟡
Estado de ESP-0001/04
Área	Estado
User model	🟡
User lifecycle	🟡
Organization model	🟡
Organization lifecycle	🟡
Membership	🟡
Invitations	🟡
Tenant isolation	✅
Administration	🟡
Events	🟡
Audit	✅
Privacy	✅
API boundary	🟡
Persistence	⏳
ADRs	⏳
63. Resultado

La arquitectura de Identity empieza a quedar estructurada de esta forma:

                         ESP-0001
                           IDENTITY
                              │
          ┌───────────────────┼───────────────────┐
          │                   │                   │
          ▼                   ▼                   ▼
     DOMAIN MODEL        ARCHITECTURE       AUTHENTICATION
          │                   │                   │
          │                   │                   ▼
          │                   │              AUTHORIZATION
          │                   │                   │
          └───────────────────┼───────────────────┘
                              │
                              ▼
                     USER & ORGANIZATION
                              │
               ┌──────────────┼──────────────┐
               ▼              ▼              ▼
             User       Organization    Membership
                                             │
                                             ▼
                                        Invitation

No pasaría todavía al código. El siguiente documento lógico sería:

ESP-0001 / 05 — Identity Roles & Permissions

Ahí vamos a definir formalmente:

Role
Permission
RolePermission
RBAC
ABAC
System Roles
Organization Roles
Permission Inheritance
Permission Evaluation
Role Lifecycle
Permission Lifecycle
Role Assignment
Role Revocation

y podremos resolver varias de las decisiones que ahora aparecen como ADR pendientes, especialmente si los roles pertenecen a la organización, si pueden existir roles globales y cómo se combina RBAC con ABAC.
