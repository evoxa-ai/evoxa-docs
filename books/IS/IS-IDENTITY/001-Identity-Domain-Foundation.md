IS-IDENTITY-001 — Identity Domain Foundation

Document ID: IS-IDENTITY-001
Domain: Identity
ESP: ESP-0001 — Identity
Status: Draft
Priority: Critical
Version: 1.0.0
Repository: evoxa-docs
Target Repository: evoxa-platform

Esta Implementation Story pertenece a la cadena oficial ESP → IS → Technical Tasks → Source Code, que el Blueprint define como obligatoria para toda implementación EVOXA.

1. Propósito

Establecer la fundación técnica del dominio Identity sobre la cual se implementarán posteriormente:

Users
Organizations
Memberships
Roles
Permissions
Authentication
Sessions
Refresh Tokens
MFA
Security Policies
Audit
Domain Events
Outbox

Identity está definido como un dominio arquitectónico independiente y su ubicación prevista es:

apps/api/app/domains/identity/

2. Business Capability
Identity Domain Foundation

Proporcionar una estructura de dominio coherente, modular, segura y extensible que permita implementar las capacidades de identidad de EVOXA sin acoplar la lógica de negocio a HTTP, FastAPI, PostgreSQL, Redis u otras tecnologías de infraestructura.

Esto sigue el principio arquitectónico de EVOXA de organizar el sistema alrededor de business domains, manteniendo la tecnología como mecanismo de implementación y no como definición de la arquitectura.

3. Implementation Story
Story

As the EVOXA Platform,
I want a well-defined Identity Domain foundation,
so that identity, authentication, authorization and security capabilities can be implemented consistently, securely and independently.

4. Alcance

Esta Story cubre exclusivamente la fundación del dominio.

Incluye
Domain structure
Entities
Value Objects
Aggregates
Repository contracts
Domain services boundaries
Domain errors
Lifecycle states
Domain event contracts
Infrastructure boundaries
Testing foundation
Traceability
No incluye todavía
❌ Implementación completa de login
❌ Implementación JWT
❌ Implementación MFA
❌ Implementación PostgreSQL
❌ Implementación Redis
❌ API endpoints completos
❌ UI
❌ Kubernetes
❌ CI/CD

Esas capacidades serán desarrolladas por las Stories posteriores.

5. Architectural Context

La estructura seguirá las capas definidas por la arquitectura:

┌─────────────────────┐
│    Presentation     │
└──────────┬──────────┘
           ↓
┌─────────────────────┐
│        API          │
└──────────┬──────────┘
           ↓
┌─────────────────────┐
│    Application      │
└──────────┬──────────┘
           ↓
┌─────────────────────┐
│       Domain        │
└──────────┬──────────┘
           ↓
┌─────────────────────┐
│   Infrastructure    │
└─────────────────────┘

Esta separación está establecida por el Architecture Map.

6. Target Structure

La primera estructura será:

apps/api/app/domains/identity/
│
├── api/
│
├── application/
│
├── domain/
│   │
│   ├── entities/
│   ├── value_objects/
│   ├── aggregates/
│   ├── services/
│   ├── policies/
│   ├── events/
│   └── repositories/
│
└── infrastructure/

Esta estructura representa la aplicación de la arquitectura por capas al dominio Identity.

7. Domain Model Foundation
7.1 Core Entities

La fundación deberá contemplar:

User
Organization
Membership
Role
Permission
Session
7.2 Security Entities

Posteriormente se incorporarán:

RefreshToken
MFAFactor
MFAChallenge
RecoveryCode
TrustedDevice
SecurityPolicy
7.3 Platform Entities

Y finalmente:

AuditEvent
OutboxEvent

Estas entidades corresponden a las responsabilidades que ya quedaron definidas para ESP-0001 y sus ADR asociados.

8. Value Objects

Se definirán como mínimo:

UserId
OrganizationId
MembershipId
RoleId
PermissionId
SessionId
Email
Principio

El dominio debe trabajar con conceptos de negocio fuertemente tipados en lugar de depender directamente de strings, UUIDs o estructuras propias de la base de datos.

Por ejemplo:

UserId

representa una identidad de usuario.

No debe convertirse simplemente en:

database_id: str

dentro de toda la lógica de negocio.

9. Aggregates

Los aggregates iniciales serán:

User
Organization
Role
Session
User Aggregate

Responsable de invariantes relacionadas con la cuenta del usuario.

Organization Aggregate

Responsable del contexto organizacional/tenant.

Role Aggregate

Responsable de la definición y estado del rol.

Session Aggregate

Responsable del ciclo de vida de la sesión.

10. Tenant Boundary

Identity debe reconocer explícitamente el contexto de organización cuando corresponda.

User
   │
   └── Membership
           │
           └── Organization

La organización se utiliza como límite de tenant dentro de la arquitectura propuesta.

Esto es particularmente importante para impedir:

Tenant A
   ↓
datos de Tenant B

La autorización deberá considerar posteriormente:

User
Organization
Membership
Roles
Permissions
Resource
Operation
Security Context
11. Repository Contracts

El dominio definirá contratos para persistencia:

UserRepository
OrganizationRepository
MembershipRepository
RoleRepository
PermissionRepository
SessionRepository

Conceptualmente:

Domain
   │
   ▼
Repository Contract
   │
   ▼
Infrastructure
   │
   ▼
PostgreSQL

No:

Domain
   │
   ▼
SQLAlchemy / PostgreSQL

La persistencia primaria definida en nuestros ADR de Identity será PostgreSQL, pero el dominio no debe conocer los detalles de su implementación.

12. Domain Services

Se establecerán las fronteras de:

AuthenticationService
AuthorizationService
SessionService
SecurityPolicyService
AuditService

No significa que todos serán implementados dentro de esta Story.

Aquí solamente dejamos definidos sus límites y responsabilidades.

13. Domain Errors

La taxonomía inicial deberá permitir distinguir errores de negocio de errores técnicos.

IdentityError
├── UserNotFound
├── OrganizationNotFound
├── MembershipNotFound
├── InvalidCredentials
├── AccountSuspended
├── AccountDisabled
├── MembershipSuspended
├── Unauthorized
├── Forbidden
├── SessionNotFound
├── SessionRevoked
├── SessionExpired
└── SecurityPolicyViolation

Posteriormente estos errores serán traducidos por la capa API al contrato estándar de errores de EVOXA.

14. Lifecycle States
User
PENDING
   ↓
ACTIVE
   ↓
SUSPENDED
   ↓
ACTIVE

ACTIVE
   ↓
DISABLED
Organization
PENDING
   ↓
ACTIVE
   ↓
SUSPENDED
   ↓
ACTIVE
Membership
INVITED
   ↓
ACTIVE
   ↓
SUSPENDED
   ↓
ACTIVE

ACTIVE
   ↓
REMOVED
Session
CREATED
   ↓
ACTIVE
   ├──→ EXPIRED
   └──→ REVOKED

Estos estados serán utilizados por las Stories posteriores.

15. Domain Events

La fundación deberá establecer contratos para eventos Identity.

User
UserCreated
UserUpdated
UserActivated
UserSuspended
UserDisabled
Organization
OrganizationCreated
OrganizationUpdated
OrganizationSuspended
OrganizationReactivated
Membership
MembershipCreated
MembershipActivated
MembershipSuspended
MembershipRemoved
MembershipRoleChanged
Session
SessionCreated
SessionRevoked
SessionExpired

La implementación de publicación mediante Outbox corresponde posteriormente a IS-IDENTITY-013.

16. Acceptance Criteria
AC-001 — Domain Structure

Given Identity is an EVOXA domain

When the Identity module is created

Then it shall maintain separation between API, Application, Domain and Infrastructure layers.

AC-002 — Domain Independence

Given the Identity Domain

When domain logic is executed

Then it shall not directly depend on HTTP, FastAPI or database implementation details.

AC-003 — Entity Boundaries

Given the Identity domain model

When entities are defined

Then each entity shall have a clearly defined responsibility.

AC-004 — Aggregate Boundaries

Given Identity aggregates

When business operations modify aggregate state

Then aggregate invariants shall be enforced inside the appropriate aggregate boundary.

AC-005 — Repository Abstraction

Given Identity requires persistence

When repository contracts are defined

Then domain contracts shall remain independent of the persistence technology.

AC-006 — Tenant Context

Given an organization-scoped operation

When the operation is executed

Then the organization/tenant context shall be explicitly represented.

AC-007 — Lifecycle Validation

Given an Identity entity

When its state changes

Then only valid lifecycle transitions shall be permitted.

AC-008 — Security Boundary

Given Identity contains security-sensitive information

When domain objects or events are generated

Then secrets and credentials shall not be exposed.

AC-009 — Event Contract

Given a state-changing Identity operation

When the operation produces a domain event

Then the event shall contain the information required by the EVOXA event contract without exposing secrets.

AC-010 — Traceability

Given IS-IDENTITY-001

When implementation begins

Then every technical implementation shall trace back to ESP-0001 and its applicable ADRs.

17. Technical Tasks
TASK-IDENTITY-001-01
Create Identity directory structure
apps/api/app/domains/identity/
├── api/
├── application/
├── domain/
└── infrastructure/
TASK-IDENTITY-001-02
Create domain substructure
domain/
├── entities/
├── value_objects/
├── aggregates/
├── services/
├── policies/
├── events/
└── repositories/
TASK-IDENTITY-001-03
Define core entities

Crear los contratos iniciales para:

User
Organization
Membership
Role
Permission
Session
TASK-IDENTITY-001-04
Define security entities

Preparar los contratos para:

RefreshToken
MFAFactor
MFAChallenge
RecoveryCode
TrustedDevice
SecurityPolicy
TASK-IDENTITY-001-05
Define Value Objects

Implementar:

UserId
OrganizationId
MembershipId
RoleId
PermissionId
SessionId
Email
TASK-IDENTITY-001-06
Define aggregate boundaries

Documentar responsabilidades e invariantes de:

User
Organization
Role
Session
TASK-IDENTITY-001-07
Define repository contracts

Crear contratos para:

UserRepository
OrganizationRepository
MembershipRepository
RoleRepository
PermissionRepository
SessionRepository
TASK-IDENTITY-001-08
Define lifecycle state models

Implementar los estados y transiciones válidas de:

User
Organization
Membership
Session
TASK-IDENTITY-001-09
Define domain error taxonomy

Crear la jerarquía de errores de Identity.

TASK-IDENTITY-001-10
Define domain service boundaries

Establecer interfaces/responsabilidades para:

Authentication
Authorization
Session
Security Policy
Audit
TASK-IDENTITY-001-11
Define Identity domain event contracts

Establecer los tipos y estructura conceptual de los eventos Identity.

TASK-IDENTITY-001-12
Define infrastructure boundaries

Garantizar separación:

Domain
   ↓
Contract
   ↓
Infrastructure
TASK-IDENTITY-001-13
Define test foundation

Preparar:

tests/
└── identity/
    ├── unit/
    ├── component/
    ├── integration/
    ├── contract/
    ├── e2e/
    ├── security/
    ├── performance/
    └── resilience/

La estrategia de testing de EVOXA contempla múltiples niveles de validación y exige trazabilidad de las Stories hacia sus pruebas.

TASK-IDENTITY-001-14
Establish traceability

Cada task deberá mantener:

Technical Task
      ↓
IS-IDENTITY-001
      ↓
ESP-0001
      ↓
Applicable ADR
      ↓
Source Code
      ↓
Tests
18. Dependencies
Upstream
BP-0001
   ↓
BP-0002
   ↓
Engineering Standards
   ↓
ESP-0001
   ↓
ADR-IDENTITY-001...011
Downstream
IS-IDENTITY-001
       ↓
IS-IDENTITY-002
IS-IDENTITY-003
IS-IDENTITY-004
IS-IDENTITY-005
...

La arquitectura de dependencias de EVOXA establece precisamente ESP → IS → Technical Tasks → Source Code.

19. Traceability Matrix
Artifact	Reference
Architecture Domain	Identity
ESP	ESP-0001
Architecture	BP-0002
ADR	ADR-IDENTITY-001
ADR	ADR-IDENTITY-005
ADR	ADR-IDENTITY-006
ADR	ADR-IDENTITY-007
ADR	ADR-IDENTITY-010
ADR	ADR-IDENTITY-011
Implementation Story	IS-IDENTITY-001
Technical Tasks	TASK-IDENTITY-001-01 → 014
Source Code	evoxa-platform/apps/api/app/domains/identity/
Tests	evoxa-platform/apps/api/tests/identity/
20. Definition of Done

IS-IDENTITY-001 podrá pasar a Approved cuando:

☐ Domain structure approved
☐ Core entities defined
☐ Security entities defined
☐ Value Objects defined
☐ Aggregates defined
☐ Repository contracts defined
☐ Domain service boundaries defined
☐ Error taxonomy defined
☐ Lifecycle states defined
☐ Event contracts defined
☐ Infrastructure boundaries defined
☐ Test structure defined
☐ Traceability complete
☐ Architecture review complete
☐ QA review complete
☐ ADR dependencies verified

La Story entonces podrá pasar:

Draft
  ↓
Refined
  ↓
Approved
  ↓
Planned
  ↓
Implemented
  ↓
Tested
  ↓
Released

Ese ciclo es consistente con el modelo de lifecycle y gobernanza definido para las Implementation/User Stories.

21. Resultado esperado

Al terminar IS-IDENTITY-001, todavía no tendremos el login funcionando.

Tendremos algo mucho más importante:

             ESP-0001
                 │
                 ▼
        Identity Foundation
                 │
       ┌─────────┼─────────┐
       ▼         ▼         ▼
     Domain   Contracts   Events
       │         │         │
       └─────────┼─────────┘
                 ▼
        Technical Tasks
                 │
                 ▼
          Ready for Code

Y desde aquí podemos comenzar la siguiente Story:

IS-IDENTITY-002 — User & Account Management

donde ya definiremos exactamente qué debe hacer el sistema con Users, sus casos de uso, reglas, endpoints involucrados, eventos, acceptance criteria y posteriormente sus Technical Tasks.

Este enfoque mantiene la regla fundamental del Blueprint: el código no aparece hasta que existe una especificación implementable y trazable.
