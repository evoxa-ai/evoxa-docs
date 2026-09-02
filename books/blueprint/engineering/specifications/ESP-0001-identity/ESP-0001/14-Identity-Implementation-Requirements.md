ESP-0001 / 14 — Identity Implementation Requirements

Documento: Engineering Specification — ESP-0001 Identity
Capítulo: 14 — Identity Implementation Requirements
Estado: Draft
Versión: 1.0.0
Dominio: Identity
Tipo: Engineering Specification

1. Propósito

Este capítulo transforma las especificaciones anteriores de ESP-0001 — Identity en requisitos concretos para su implementación.

El objetivo es establecer qué debe construirse, dónde debe vivir cada componente, cuáles son sus responsabilidades y qué condiciones debe cumplir, sin entrar todavía en código fuente.

La implementación deberá respetar el lifecycle definido por EVOXA:

Vision → Blueprint → Reference Architecture → Engineering Standards → ESP → Implementation Stories → Technical Tasks → Source Code → Testing → Deployment → Monitoring → Continuous Evolution.

Por lo tanto, este documento no reemplaza las Implementation Stories ni los Technical Tasks.

2. Alcance

La implementación de Identity deberá cubrir:

Identity
│
├── Authentication
├── Authorization
├── Users
├── Organizations
├── Memberships
├── Roles
├── Permissions
├── Sessions
├── Access Tokens
├── Refresh Tokens
├── MFA
├── Security Policies
├── Audit
└── Events & Integrations

Estas capacidades corresponden a Identity como Shared Platform Domain.

3. Principios de implementación

La implementación deberá respetar:

IMP-001 — Domain Ownership

Identity será responsable exclusivamente de sus propias reglas y datos.

IMP-002 — API First

Toda capacidad expuesta externamente deberá tener un contrato API definido antes de su implementación.

IMP-003 — Domain Driven

La lógica de negocio deberá residir en el dominio y no en controllers o infraestructura.

IMP-004 — Secure by Default

Las operaciones deberán comenzar en estado:

DENY

y solamente permitir acceso cuando exista autorización válida.

IMP-005 — Fail Closed

Ante una condición de seguridad desconocida o fallo de autorización:

DENY
IMP-006 — Tenant Isolation

Toda operación que dependa de una organización deberá ejecutarse dentro del contexto de tenant correspondiente.

IMP-007 — Auditability

Las operaciones críticas deberán producir evidencia auditable.

IMP-008 — Observability

Las operaciones críticas deberán poder rastrearse mediante correlation/trace identifiers.

4. Estructura de implementación

La arquitectura de EVOXA define las capas:

Presentation
      ↓
API
      ↓
Application
      ↓
Domain
      ↓
Infrastructure

Y el mapa arquitectónico establece Identity en:

apps/api/app/domains/identity/

La estructura propuesta será:

apps/
└── api/
    └── app/
        └── domains/
            └── identity/
                ├── api/
                ├── application/
                ├── domain/
                └── infrastructure/
5. API Layer

La capa api será responsable exclusivamente de la exposición de Identity.

Propuesta:

identity/api/
├── controllers/
├── routes/
├── schemas/
├── serializers/
└── middleware/
Responsabilidades
HTTP.
Routing.
Request validation.
Response serialization.
Authentication middleware.
Authorization middleware.
API error mapping.
Correlation ID.
OpenAPI integration.

Los controllers no deberán contener reglas de negocio complejas.

6. Application Layer

Propuesta:

identity/application/
├── commands/
├── queries/
├── services/
├── handlers/
├── dto/
└── ports/

Esta capa coordinará los casos de uso.

Ejemplos:

AuthenticateUser
RefreshSession
LogoutUser
CreateUser
UpdateUser
SuspendUser
CreateOrganization
AddMembership
AssignRole
RevokeRole
CreateMFAFactor
VerifyMFAChallenge
EvaluateSecurityPolicy
RevokeSession
7. Domain Layer

Propuesta:

identity/domain/
├── entities/
├── aggregates/
├── value-objects/
├── services/
├── policies/
├── events/
├── repositories/
└── errors/

Aquí deberá residir la lógica esencial de Identity.

Ejemplos:

User
Organization
Membership
Role
Permission
Session
RefreshToken
MFAFactor
SecurityPolicy
AuditEvent
8. Infrastructure Layer

Propuesta:

identity/infrastructure/
├── persistence/
├── repositories/
├── database/
├── cache/
├── authentication/
├── mfa/
├── events/
├── audit/
└── integrations/

Responsabilidades:

PostgreSQL.
Redis.
Event Platform.
External Identity Providers.
MFA infrastructure.
Audit persistence.
Cryptographic services.
External integrations.
9. User Implementation

La implementación deberá proporcionar:

User
├── create
├── read
├── update
├── activate
├── suspend
├── disable
└── lifecycle management

El ciclo propuesto es:

PENDING
   ↓
ACTIVE
   ↓
SUSPENDED
   ↓
DISABLED

Las transiciones deberán estar controladas por reglas de dominio.

10. Organization Implementation

Debe implementarse:

Organization
├── create
├── update
├── suspend
└── reactivate

La organización constituye la frontera de tenant propuesta para EVOXA, pendiente de formalización mediante ADR.

11. Membership Implementation

Debe existir una separación clara entre:

User
Organization
Membership
Role
Permission

Una membership representará la relación:

User
   │
   └── Membership
          │
          └── Organization

Lifecycle:

INVITED
   ↓
ACTIVE
   ↓
SUSPENDED
   ↓
REMOVED
12. Role Implementation

Roles deberán poder:

Crear.
Modificar.
Activar/desactivar según política.
Asociar permisos.
Remover permisos.

La implementación deberá soportar inicialmente los roles contemplados por la arquitectura:

Coach
Nutritionist
Physiotherapist
Team Manager
Admin

No se deberá asumir que esta lista es definitiva para todas las futuras implementaciones.

13. Permission Implementation

Las permissions deberán representar capacidades concretas.

Convención propuesta:

<resource>.<action>

Ejemplos:

clients.read
clients.create
clients.update
clients.delete

training.read
training.create
training.update

El modelo deberá permitir:

Role
  ↓
RolePermission
  ↓
Permission
14. Authorization Engine

Deberá existir un componente central de autorización.

Conceptualmente:

AuthorizationService

Entrada:

User
Organization
Membership
Roles
Permissions
Resource
Action
Context

Salida:

ALLOW
DENY

o las decisiones ampliadas definidas en Security Policies:

ALLOW
DENY
CHALLENGE
REAUTHENTICATE
REQUIRE_MFA
REVOKE
15. Authentication Implementation

El flujo deberá ser:

Credentials
     ↓
User lookup
     ↓
Credential validation
     ↓
Account validation
     ↓
Membership/Tenant validation
     ↓
MFA
     ↓
Session
     ↓
Access Token
     ↓
Refresh Token

Authentication deberá estar separada de Authorization.

16. Password Service

Debe existir un componente dedicado para:

PasswordService

Responsabilidades:

Hash.
Verify.
Password policy validation.
Password change.
Password reset.

Las contraseñas deberán utilizar Argon2id.

La implementación nunca deberá almacenar passwords en texto plano.

17. Token Service

Debe existir una abstracción:

TokenService

Responsable de:

Access token generation.
JWT validation.
Refresh token generation.
Refresh rotation.
Token revocation.
Token reuse detection.

No deberá incluir datos sensibles innecesarios dentro del JWT.

18. Session Service

Debe existir:

SessionService

Responsabilidades:

Create session.
Get session.
Revoke session.
Expire session.
Revoke all sessions.
List user sessions.

Lifecycle:

CREATED
   ↓
ACTIVE
   ↓
EXPIRED
   ↓
REVOKED
19. Refresh Token Security

Los refresh tokens deberán:

Ser almacenados de forma segura.
Utilizar hash cuando sean persistidos.
Soportar rotation.
Detectar reuse.
Permitir revocación.
Estar asociados a una sesión.
Permitir identificar una familia de tokens cuando corresponda.

No deberán almacenarse como secretos recuperables en texto plano.

20. MFA Implementation

Propuesta de estructura:

identity/
└── domain/
    └── mfa/
        ├── factors/
        ├── challenges/
        ├── recovery/
        └── trusted-devices/

Deberá permitir la incorporación de:

TOTP.
WebAuthn/Passkeys.
Email OTP.
SMS OTP.
Recovery codes.

La arquitectura no obliga todavía a habilitar todos simultáneamente.

21. Security Policy Engine

Debe existir una abstracción equivalente a:

SecurityPolicyEngine

Entrada:

SecurityContext

incluyendo:

user
organization
membership
roles
permissions
session
authentication
mfa
device
ip
request
resource
operation
risk

Salida:

SecurityDecision
22. Audit Service

Debe existir:

AuditService

Responsable de registrar operaciones críticas.

Ejemplos:

UserCreated
UserAuthenticated
UserSuspended
RoleChanged
PermissionChanged
SessionRevoked
MFAEnabled
MFAFailed
SecurityPolicyViolation

Audit deberá mantenerse separado de los logs técnicos normales.

23. Event Publisher

Identity deberá disponer de:

EventPublisher

Responsable de publicar eventos de dominio/integración.

Ejemplo:

UserCreated
   ↓
EventPublisher
   ↓
Event Platform

La arquitectura de eventos de EVOXA establece eventos inmutables, versionados y con metadata de correlación.

24. Transactional Outbox

La implementación deberá considerar el patrón:

Database Transaction
       │
       ├── Domain Data
       │
       └── Outbox Event
                    ↓
              Event Publisher
                    ↓
              Event Platform

Esto evita el problema:

DB commit = SUCCESS
Event publish = FAILURE

La utilización definitiva del patrón deberá quedar respaldada por ADR.

25. Repository Interfaces

El dominio no deberá depender directamente de Sequelize, PostgreSQL, Redis u otra tecnología.

Ejemplo conceptual:

UserRepository
OrganizationRepository
MembershipRepository
RoleRepository
PermissionRepository
SessionRepository
RefreshTokenRepository
MFAFactorRepository
SecurityPolicyRepository
AuditRepository

Las implementaciones concretas deberán pertenecer a Infrastructure.

26. Database Implementation

PostgreSQL será la persistencia transaccional principal propuesta para Identity.

Tablas principales:

users
organizations
memberships
roles
permissions
role_permissions
sessions
refresh_tokens
mfa_factors
mfa_challenges
recovery_codes
trusted_devices
security_policies
audit_events
outbox_events

Esta estructura deriva de la especificación de persistencia de Identity.

27. Redis Implementation

Redis podrá utilizarse para información temporal o de alta frecuencia como:

Rate limiting.
Temporary MFA challenges.
Session acceleration/cache.
Security counters.
Short-lived security state.

Redis no deberá convertirse automáticamente en source of truth para información transaccional de Identity.

28. API Implementation

La API deberá comenzar en:

/api/v1/identity

Y mantener la estructura definida en ESP-0001/10.

Ejemplos:

POST   /auth/login
POST   /auth/refresh
POST   /auth/logout
POST   /auth/logout-all
GET    /auth/me
GET    /users
POST   /users
GET    /users/{id}
PATCH  /users/{id}
GET    /organizations
POST   /organizations
GET    /organizations/{id}
GET    /organizations/{id}/memberships
POST   /organizations/{id}/memberships
GET    /roles
GET    /permissions
GET    /sessions
POST   /sessions/{id}/revoke
POST   /sessions/revoke-all
GET    /mfa/factors
POST   /mfa/factors
POST   /mfa/challenges/{id}/verify
29. Request Validation

Todas las entradas deberán validarse antes de llegar al dominio.

Validar:

Tipo.
Formato.
Required fields.
Length.
Enum.
IDs.
Pagination.
Filtering.
Security constraints.

La validación HTTP no reemplaza la validación de reglas de dominio.

30. Error Handling

Identity deberá implementar errores de dominio y mapearlos a HTTP.

Ejemplo:

UserNotFound
       ↓
404

Unauthorized
       ↓
401

Forbidden
       ↓
403

Conflict
       ↓
409

ValidationError
       ↓
422

El formato externo deberá respetar:

{
  "error": {
    "code": "...",
    "message": "...",
    "correlation_id": "..."
  }
}
31. Middleware

Identity deberá proporcionar middleware reutilizable para:

Authentication
Authorization
TenantContext
CorrelationId
RateLimit
SecurityPolicy
AuditContext

La autenticación deberá ejecutarse antes de autorización.

32. Tenant Context

Cada request autenticada que opere dentro de una organización deberá establecer un:

TenantContext

Conceptualmente:

Request
 ↓
Authentication
 ↓
Tenant Resolution
 ↓
Authorization
 ↓
Application

No deberá permitirse que un usuario controle arbitrariamente el tenant mediante un parámetro sin validación.

33. Security Context

La implementación deberá centralizar el contexto de seguridad:

SecurityContext
├── user
├── session
├── organization
├── membership
├── roles
├── permissions
├── authentication_method
├── mfa_state
├── device
├── ip
└── request

Esto permitirá que Authorization y Security Policy trabajen sobre un modelo consistente.

34. External Identity Providers

La integración deberá utilizar adapters/interfaces.

Conceptualmente:

IdentityProvider
      │
      ├── OIDC
      ├── OAuth
      ├── SAML
      ├── Google
      ├── Entra ID
      ├── Okta
      └── Auth0

No se deberá acoplar el dominio a un proveedor específico.

35. Configuration

Los secretos y configuraciones sensibles deberán gestionarse mediante configuración segura.

Ejemplos:

JWT issuer
JWT audience
JWT signing configuration
Database credentials
Redis credentials
OAuth client secrets
MFA configuration
Email provider
SMS provider

Nunca deberán estar hardcodeados en el código fuente.

36. Observability

Cada operación importante deberá generar contexto suficiente para observabilidad:

trace_id
correlation_id
request_id
session_id
user_id
tenant_id

Los valores sensibles deberán ser filtrados.

37. Testing Implementation

Cada módulo implementado deberá incluir pruebas.

Propuesta:

identity/
├── api/
├── application/
├── domain/
├── infrastructure/
└── tests/
    ├── unit/
    ├── component/
    ├── integration/
    ├── api/
    ├── security/
    └── e2e/

Los requisitos detallados de testing están definidos en:

ESP-0001 / 13 — Identity Testing & Quality Requirements.

38. Definition of Done técnica

Una Implementation Story de Identity no podrá considerarse terminada solamente porque el endpoint funciona.

Deberá tener:

Code
 ↓
Domain Tests
 ↓
Integration Tests
 ↓
API Tests
 ↓
Security Tests
 ↓
Audit Tests
 ↓
Event Tests
 ↓
E2E
 ↓
Observability
 ↓
Documentation
39. Quality Gates

Antes de merge:

Build.
Type checking.
Lint.
Unit tests.
Coverage.
Security tests.

Antes de staging:

Integration.
API contract.
E2E.
Tenant isolation.
Performance.

Antes de production:

Todos los quality gates.
Security review.
Migration validation.
Rollback strategy.
Observability.
Audit validation.
40. Implementation Dependency Graph

Identity deberá implementarse respetando dependencias.

                    Identity Foundation
                           │
             ┌─────────────┴─────────────┐
             ↓                           ↓
           User                    Organization
             │                           │
             └──────────┬────────────────┘
                        ↓
                   Membership
                        │
              ┌─────────┴─────────┐
              ↓                   ↓
            Roles            Permissions
              │                   │
              └─────────┬─────────┘
                        ↓
                  Authorization
                        │
             ┌──────────┴──────────┐
             ↓                     ↓
       Authentication           Sessions
             │                     │
             └──────────┬──────────┘
                        ↓
                       MFA
                        │
                        ↓
                Security Policies
                        │
              ┌─────────┴─────────┐
              ↓                   ↓
            Audit               Events
41. Orden recomendado de implementación

No recomiendo comenzar programando todo Identity simultáneamente.

El orden técnico recomendado es:

Fase 1 — Foundation
Project structure
Configuration
Database
Migrations
Base errors
Base interfaces
Observability
Fase 2 — Core Identity
User
Organization
Membership
Fase 3 — Authorization
Role
Permission
RolePermission
Authorization
Tenant Context
Fase 4 — Authentication
Password
Login
Access Token
Refresh Token
Session
Logout
Fase 5 — MFA
Factors
Challenges
Recovery Codes
Trusted Devices
Fase 6 — Security
Security Policies
Rate Limiting
Security Events
Fase 7 — Audit & Events
Audit
Domain Events
Integration Events
Outbox
Fase 8 — External Identity
OIDC
OAuth
SAML
External Providers
Fase 9 — Hardening
Security Testing
Performance
Load
Chaos
E2E
42. Implementation Stories

Este documento deberá convertirse posteriormente en Implementation Stories.

Ejemplo:

IS-IDENTITY-001
Identity Project Foundation

IS-IDENTITY-002
User Management

IS-IDENTITY-003
Organization Management

IS-IDENTITY-004
Membership Management

IS-IDENTITY-005
Roles

IS-IDENTITY-006
Permissions

IS-IDENTITY-007
Authorization

IS-IDENTITY-008
Authentication

IS-IDENTITY-009
Sessions

IS-IDENTITY-010
Refresh Tokens

IS-IDENTITY-011
MFA

IS-IDENTITY-012
Security Policies

IS-IDENTITY-013
Audit

IS-IDENTITY-014
Identity Events

IS-IDENTITY-015
Identity Integration

IS-IDENTITY-016
Identity Security Hardening

Estos IDs son una propuesta de organización y todavía no deben considerarse IDs oficiales del repositorio.

43. Technical Tasks

Cada IS deberá dividirse posteriormente en Technical Tasks.

Por ejemplo:

IS-IDENTITY-008 Authentication
│
├── TASK-001 Password service
├── TASK-002 Credential validation
├── TASK-003 Account status validation
├── TASK-004 Authentication context
├── TASK-005 Login use case
├── TASK-006 Session creation
├── TASK-007 Access token generation
├── TASK-008 Refresh token generation
├── TASK-009 Audit
├── TASK-010 Events
├── TASK-011 API endpoint
├── TASK-012 Unit tests
├── TASK-013 Integration tests
└── TASK-014 Security tests
44. Traceability

La implementación deberá conservar:

Blueprint
   ↓
ESP-0001
   ↓
Implementation Story
   ↓
Technical Task
   ↓
Source Code
   ↓
Test
   ↓
Deployment

Esta trazabilidad es parte fundamental del modelo de arquitectura de EVOXA.

45. Requisitos abiertos antes del código

Hay decisiones que no debemos inventar antes de implementarlas.

Deben quedar resueltas mediante ADR:

Seguridad
Access token TTL.
Refresh token TTL.
Refresh rotation.
Reuse detection behavior.
JWT signing algorithm.
Token storage.
Browser storage.
Mobile storage.
Identity
Multi-organization users.
Multiple roles per membership.
Role hierarchy.
System vs organization roles.
Platform Admin model.
MFA
Factores obligatorios.
Recovery strategy.
Trusted devices.
Step-up authentication.
Assurance levels.
Persistence
Redis responsibilities.
Refresh token persistence.
MFA secret storage.
Audit storage.
Outbox implementation.
Authorization
RBAC/ABAC implementation.
Policy engine.
Permission caching.
Policy precedence.
Integrations
External IdP priority.
OIDC/SAML implementation order.
Webhook strategy.
46. Regla importante para la programación

A partir de este documento, el equipo de implementación no deberá interpretar una propuesta como una decisión arquitectónica definitiva.

La clasificación será:

Blueprint Requirement
        ↓
Normative

Derived Design
        ↓
Needs ADR / Approval

Implementation Detail
        ↓
Technical Task

Esto evita que una decisión tomada provisionalmente durante la programación termine convirtiéndose accidentalmente en arquitectura oficial.

47. Estado de ESP-0001

Con este capítulo tenemos:

Capítulo	Estado
01 Domain Model	✅ Draft
02 Architecture	✅ Draft
03 Authentication & Authorization	✅ Draft
04 User & Organization Management	✅ Draft
05 Roles & Permissions	✅ Draft
06 Sessions & Token Management	✅ Draft
07 MFA	✅ Draft
08 Security Policies	✅ Draft
09 Audit & Compliance	✅ Draft
10 API Contracts	✅ Draft
11 Persistence & Data Model	✅ Draft
12 Events & Integration	✅ Draft
13 Testing & Quality	✅ Draft
14 Implementation Requirements	✅ Draft
Próximo capítulo recomendado

ESP-0001 / 15 — Identity Deployment & Operational Requirements

Ahí cerraríamos cómo Identity debe funcionar en Dev → QA → Staging → Production, incluyendo configuración, secrets, migrations, health checks, observability, backups, disaster recovery, scaling, deployment, rollback y SLOs. Esto encaja directamente con los requisitos de Release/Deployment y Operational Excellence del Blueprint.

Después de completar ese capítulo, ya tendría sentido pasar a ADR de Identity → Implementation Stories → Technical Tasks → código, siguiendo la trazabilidad oficial en vez de saltar directamente desde el Blueprint al código.
