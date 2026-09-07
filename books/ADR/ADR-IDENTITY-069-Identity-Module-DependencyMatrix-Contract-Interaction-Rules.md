ADR-IDENTITY-069 — Identity Module Dependency Matrix, Contracts & Interaction Rules

Status: Proposed
Version: 1.0
Date: 2026-09-07
Domain: Identity / Architecture / Domain-Driven Design / Security / Integration
Priority: Critical
Type: Architecture / Module Dependencies / Contracts / Interaction Rules

1. Contexto

El ADR-IDENTITY-068 — Identity Domain Integration & Module Boundaries estableció Identity como un Bounded Domain modular, definiendo sus principales módulos, responsabilidades, ownership de datos y reglas generales de interacción.

Sin embargo, todavía falta una definición formal de:

qué módulos pueden comunicarse directamente;
qué módulos solamente pueden comunicarse mediante eventos;
qué contratos deben utilizar;
qué datos pueden consultar;
qué datos no pueden modificar;
qué dependencias están prohibidas;
quién publica cada evento;
quién consume cada evento;
cómo se evita la creación de ciclos;
cómo se preserva Tenant Isolation;
cómo se evita que módulos secundarios implementen autorización por su cuenta.

Este ADR convierte los límites conceptuales definidos en el ADR-068 en una matriz de dependencias y reglas de interacción verificables.

2. Problema

Sin una matriz explícita, una implementación podría terminar evolucionando hacia:

Account
   ↕
Authentication
   ↕
Authorization
   ↕
Session
   ↕
MFA
   ↕
Policy
   ↕
Membership

con acceso indiscriminado entre módulos.

Esto produciría:

dependencia circular;
acceso directo a datos ajenos;
duplicación de lógica;
acoplamiento entre aggregates;
bypass de Authorization;
problemas de tenant isolation;
dificultad para separar módulos posteriormente;
dificultad para realizar pruebas arquitectónicas.

Por tanto:

La arquitectura necesita reglas explícitas que puedan comprobarse automáticamente durante CI/CD.

3. Decisión

Se establece una Identity Module Dependency Matrix como contrato arquitectónico obligatorio.

Cada módulo deberá declarar:

Owned Aggregates
Owned Data
Public Contracts
Allowed Synchronous Dependencies
Allowed Event Dependencies
Published Events
Consumed Events
Forbidden Dependencies
Security Responsibilities

Ningún módulo podrá utilizar componentes internos de otro módulo sin pasar por un contrato autorizado.

4. Identity Module Catalog

Los módulos establecidos por ADR-068 son:

ID-01  Account
ID-02  Organization
ID-03  Membership
ID-04  Authentication
ID-05  Authorization
ID-06  Role & Permission
ID-07  Policy
ID-08  Session
ID-09  Token
ID-10  MFA
ID-11  Recovery
ID-12  Federation
ID-13  Provisioning
ID-14  Audit
ID-15  Security Events
ID-16  Integration
5. Dependency Classification

Cada dependencia se clasificará como:

Código	Tipo
SYNC	contrato síncrono permitido
QUERY	consulta mediante contrato
EVENT	comunicación asíncrona
PUBLISH	publicación de eventos
FORBIDDEN	dependencia prohibida
INTERNAL	únicamente dentro del mismo módulo

No se permite interpretar una dependencia EVENT como permiso para acceder directamente al aggregate del módulo emisor.

6. Matriz Principal de Dependencias

La matriz inicial será:

From ↓ / To →	Account	Org	Membership	AuthN	AuthZ	Role/Perm	Policy	Session	Token	MFA	Recovery	Federation	Provisioning	Audit	Sec Events
Account	—	Q	Q	E	Q	F	Q	E	E	E	E	E	E	P	P
Organization	Q	—	S	F	Q	Q	Q	E	E	E	F	E	E	P	P
Membership	Q	Q	—	Q	S	S	Q	E	E	E	E	E	E	P	P
Authentication	Q	Q	Q	—	S	Q	S	S	S	S	E	S	E	P	P
Authorization	Q	Q	Q	Q	—	S	S	Q	Q	Q	Q	F	F	P	P
Role/Permission	F	Q	S	F	S	—	Q	E	F	F	F	F	E	P	P
Policy	Q	Q	Q	Q	S	Q	—	Q	Q	Q	Q	F	F	P	P
Session	Q	Q	Q	Q	Q	F	Q	—	S	Q	E	F	E	P	P
Token	Q	Q	Q	Q	Q	F	Q	S	—	Q	E	F	E	P	P
MFA	Q	Q	Q	S	Q	F	S	Q	Q	—	S	F	E	P	P
Recovery	S	Q	Q	S	Q	F	S	E	E	S	—	F	E	P	P
Federation	S	Q	Q	S	F	F	F	E	E	F	E	—	E	P	P
Provisioning	S	S	S	E	F	E	E	E	E	E	E	E	—	P	P
Audit	F	F	F	F	F	F	F	F	F	F	F	F	F	—	E
Security Events	F	F	F	F	F	F	F	F	F	F	F	F	F	E	—
Leyenda
Q = QUERY
S = SYNC
E = EVENT
P = PUBLISH
F = FORBIDDEN
— = INTERNAL

Esta matriz es normativa a nivel arquitectónico.

Los contratos concretos podrán refinar una relación, pero no podrán convertir una dependencia FORBIDDEN en una dependencia permitida sin modificación formal del ADR/ESP correspondiente.

7. Regla Fundamental

La matriz no significa:

A puede importar cualquier clase de B

Significa:

A puede utilizar
un contrato explícitamente publicado por B

Por ejemplo:

Authorization
       │
       ▼
MembershipContract

y no:

Authorization
       │
       ▼
MembershipRepository
8. Account Contract

Account publicará contratos conceptuales como:

AccountQuery
AccountStatusQuery
CredentialStatusQuery
IdentityLookup

Ejemplo:

GetAccountStatus(user_id)

Respuesta conceptual:

AccountStatus
- user_id
- status
- security_state
- verification_state

El consumidor no recibe información interna innecesaria.

9. Organization Contract

Organization expondrá:

OrganizationQuery
OrganizationSecurityContext
OrganizationStatus

Ejemplo:

GetOrganizationSecurityContext(organization_id)

No expone directamente:

OrganizationRepository
OrganizationModel
DatabaseSession
10. Membership Contract

Membership expondrá:

MembershipQuery
MembershipAuthorizationContext
MembershipStatus
MembershipRoles

Ejemplo:

GetMembershipContext(
    user_id,
    organization_id
)

Este contrato es especialmente importante para Authorization.

11. Authentication Contract

Authentication expondrá:

Authenticate
AuthenticationContext
AuthenticationAssurance

Por ejemplo:

Authenticate(credentials)

El resultado no implica:

ALLOW

El resultado significa únicamente:

AUTHENTICATED
12. Authorization Contract

Authorization expondrá el contrato central:

Authorize(request)

Conceptualmente:

AuthorizationRequest
        ↓
AuthorizationRuntime
        ↓
AuthorizationDecision

Los consumidores no deberán implementar lógica paralela.

13. Role & Permission Contract

Role & Permission expondrá información necesaria para Authorization:

GetEffectivePermissions(membership_id)
GetMembershipRoles(membership_id)

No deberá exponer directamente repositorios.

Además:

El resultado de Role & Permission nunca equivale automáticamente a ALLOW.

Authorization debe combinar:

RBAC
+
ABAC
+
Ownership
+
Delegation
+
Tenant
+
Policy
+
Security State
+
Criticality
+
Freshness
14. Policy Contract

Policy expondrá:

GetApplicablePolicies(context)
EvaluatePolicy(context)
GetPolicyVersion()
GetPolicySecurityContext()

El Policy Module no podrá asumir que una policy individual constituye por sí misma una autorización final.

La composición continúa siendo responsabilidad del Authorization Runtime.

15. Session Contract

Session expondrá:

CreateSession
GetSessionSecurityContext
RevokeSession
RevokeAllSessions
GetSessionStatus

Authorization podrá consultar:

session_id
status
authentication_method
MFA evidence
security state

pero no modificará directamente tablas de Session.

16. Token Contract

Token expondrá:

IssueAccessToken
RotateRefreshToken
RevokeRefreshToken
RevokeTokenFamily
ValidateRefreshContext

El almacenamiento de refresh tokens permanece interno al Token Module.

Ningún módulo puede leer directamente:

refresh_token
token_hash
token_family

salvo mediante contratos autorizados.

17. MFA Contract

MFA expondrá:

EvaluateMFARequirement
CreateChallenge
VerifyChallenge
GetMFAStatus
GetStepUpEvidence
RevokeFactor

El contrato debe diferenciar:

MFA_REQUIRED
MFA_SATISFIED
MFA_FAILED
MFA_EXPIRED
MFA_UNAVAILABLE

Nunca:

MFA → ALLOW

La decisión final continúa perteneciendo a Authorization.

18. Recovery Contract

Recovery expondrá casos de uso específicos:

StartPasswordRecovery
CompletePasswordRecovery
StartAccountRecovery
CompleteAccountRecovery
RecoverMFA

Recovery no podrá ejecutar:

DisableAuthorization
BypassTenantIsolation
GrantAdministrativeRole
DisableSecurityPolicy
19. Federation Contract

Federation será responsable exclusivamente de:

ExternalIdentity
OIDC
SAML
External Authentication
Identity Linking

Una identidad federada validada se transforma en contexto local.

Después:

Local Identity
      ↓
Local Membership
      ↓
Local Authorization
20. Provisioning Contract

Provisioning podrá producir eventos como:

UserProvisioned
MembershipProvisioned
MembershipSuspended
MembershipDeprovisioned

Pero no podrá manipular directamente:

Authorization Runtime
Policy Engine
Cache
21. Audit Contract

Audit tendrá una interfaz de escritura:

RecordAuditEvent()

y contratos controlados de lectura:

QueryAuditEvents()

Las consultas de auditoría deberán estar sujetas a Authorization.

Por tanto:

Audit
   ↑
Authorization

conceptualmente no implica que Audit pueda decidir quién puede acceder.

22. Security Event Contract

Security Events podrán ser generados por:

Authentication
Authorization
Session
Token
MFA
Recovery
Membership
Policy
Provisioning

Ejemplos:

AccountSuspended
RefreshTokenReuseDetected
SessionRevoked
MFAFactorRevoked
MembershipRevoked
PolicySecurityChange
23. Event Ownership

Cada evento debe tener un único owner.

Ejemplo:

Evento	Owner
UserCreated	Account
UserSuspended	Account
OrganizationCreated	Organization
MembershipCreated	Membership
MembershipRevoked	Membership
UserAuthenticated	Authentication
SessionCreated	Session
SessionRevoked	Session
RefreshTokenRotated	Token
RefreshTokenReuseDetected	Token
MFAFactorActivated	MFA
MFAFactorRevoked	MFA
PolicyActivated	Policy
RoleAssigned	Role/Permission
PermissionRevoked	Role/Permission
AccountRecoveryCompleted	Recovery

Los nombres definitivos y versiones deberán cerrarse posteriormente en los Event Contracts.

24. Regla de Ownership

El módulo que posee el aggregate posee:

State
Invariants
Transitions
Persistence
Domain Events

Por ejemplo:

Membership
    owns
    Membership lifecycle

Authorization puede consultar Membership.

No puede modificar directamente Membership.

25. Interaction Rule — Queries

Las consultas entre módulos son permitidas únicamente cuando:

existe contrato explícito;
el dato es necesario;
se respeta tenant context;
no se exponen datos innecesarios;
no se devuelve implementación interna.
26. Interaction Rule — Commands

Los commands cross-module deberán utilizar:

Application Contract

Ejemplo:

Authentication
    ↓
CreateSession
    ↓
Session

No:

Authentication
    ↓
SessionRepository.save()
27. Interaction Rule — Events

Los eventos se utilizarán cuando:

el consumidor no necesite respuesta inmediata;
exista desacoplamiento beneficioso;
se trate de una reacción a un cambio de estado;
se requiera propagación de seguridad;
se necesite integración con otros dominios.
28. Eventual Consistency

La propagación por eventos puede introducir consistencia eventual.

Pero esto no puede utilizarse para justificar:

stale security ALLOW

En operaciones críticas:

Freshness
Security Epoch
Entity Version
Policy Version

deberán evaluarse conforme a ADR-056–059.

29. Authorization Dependency Rule

Authorization puede depender de:

Account
Organization
Membership
Authentication
Role & Permission
Policy
Session
Token
MFA
Recovery

pero solamente mediante contratos.

Authorization no depende de:

Federation internals
Provisioning internals
Audit internals
Infrastructure repositories
External provider SDKs
30. Authorization como Security Boundary

Todos los dominios externos utilizarán:

Authorization Contract

para operaciones protegidas.

Ejemplo:

Training API
     ↓
Authorization
     ↓
Decision

No:

Training API
     ↓
Role lookup
     ↓
ALLOW
31. Cross-Tenant Rule

Toda interacción deberá transportar suficiente contexto para preservar:

tenant_id / organization_id

cuando corresponda.

Si el contexto organizacional es ambiguo:

UNKNOWN

no podrá convertirse en:

ALLOW
32. AI Agent Rule

Los AI Agents se consideran actores normales desde la perspectiva de Identity.

AI Agent
   ↓
Agent Identity
   ↓
Membership / Service Context
   ↓
Authorization

No existirán contratos especiales que permitan:

AI → bypass Authorization

ni:

AI → bypass MFA
AI → bypass tenant isolation
AI → elevate privilege
33. Prohibición de acceso a infraestructura

Los módulos de Domain/Application no podrán depender directamente de:

PostgreSQL
Redis
Kafka/Event Broker
KMS
HTTP clients
External SDKs

Estas dependencias deberán entrar mediante Ports/Adapters.

34. Prohibición de acceso a tablas ajenas

Esto es explícitamente inválido:

Authorization
   ↓
SELECT * FROM sessions

También:

Training
   ↓
SELECT * FROM identity.memberships

La forma correcta es:

Training
   ↓
Authorization Contract

o:

Identity Contract

cuando exista una necesidad legítima.

35. Shared Kernel Rule

El Shared Kernel puede contener:

TenantId
UserId
OrganizationId
MembershipId
SessionId
CorrelationId

pero no puede contener servicios completos.

Prohibido:

SharedKernel.AuthorizationService
SharedKernel.PolicyEngine
SharedKernel.AuthenticationService
36. Dependency Direction

La dirección general será:

API
 ↓
Application
 ↓
Domain
 ↓
Ports
 ↓
Infrastructure

Nunca:

Domain
 ↓
Infrastructure

ni:

Domain
 ↓
API
37. Module Internal Structure

Cada módulo deberá seguir aproximadamente:

<module>/
├── domain/
│   ├── entities/
│   ├── value_objects/
│   ├── aggregates/
│   ├── services/
│   └── events/
│
├── application/
│   ├── commands/
│   ├── queries/
│   └── services/
│
├── contracts/
│   ├── commands/
│   ├── queries/
│   └── events/
│
├── infrastructure/
│   ├── persistence/
│   ├── adapters/
│   └── providers/
│
└── api/
    ├── routes/
    ├── schemas/
    └── handlers/

La estructura concreta de carpetas deberá cerrarse en el ESP de implementación.

38. Contract Versioning

Los contratos públicos deberán versionarse.

Ejemplo conceptual:

AuthorizationRequest v1
AuthorizationDecision v1

Un cambio incompatible requiere:

v2

No se deben cambiar silenciosamente los contratos existentes.

Esto aplica también a:

Domain Events
Integration Events
API Contracts
Test Contracts
39. Contract Compatibility

Los cambios deberán clasificarse:

BACKWARD_COMPATIBLE
FORWARD_COMPATIBLE
FULLY_COMPATIBLE
INCOMPATIBLE

Los contratos security-critical requieren validación adicional.

40. Interaction Failure Rules

Si una dependencia crítica falla:

Authorization
MFA
Session
Membership
Policy
Security State

no se puede convertir automáticamente:

ERROR → ALLOW

El resultado debe seguir las reglas de fail-closed de los ADR anteriores.

41. Dependency Failure Matrix
Dependencia	Failure esperado
Account	DENY/SAFE FAILURE
Membership	DENY
Policy	DENY
MFA	REQUIRE_MFA / DENY
Session	DENY/REAUTH
Token	DENY
Role/Permission	DENY
Organization	DENY
Redis	fallback controlado
Event Platform	Outbox/retry
Audit	política operacional definida, sin bypass
External Provider	no bypass
42. Ciclos prohibidos

No se permitirán ciclos como:

Authentication
   ↓
Authorization
   ↓
Authentication

o:

Policy
   ↓
Authorization
   ↓
Policy

si implican dependencia estructural directa.

Los ciclos conceptuales deberán resolverse mediante:

Contracts
Events
Ports
Application orchestration
43. Example — Login

Flujo permitido:

API
 ↓
Authentication Application
 ↓
Account Query
 ↓
Membership Query
 ↓
Policy
 ↓
MFA
 ↓
Session
 ↓
Token
 ↓
Audit/Event

Pero el flujo no significa que Authentication sea dueño de esos módulos.

Authentication únicamente orquesta el caso de uso.

44. Example — Authorization
External Domain
       ↓
Authorization Contract
       ↓
Authorization Runtime
       ├── Account
       ├── Organization
       ├── Membership
       ├── Role/Permission
       ├── Policy
       ├── Session
       ├── MFA
       ├── Delegation
       └── Security Context
       ↓
Criticality
       ↓
Freshness
       ↓
Decision
45. Example — Membership Revocation
Membership
    │
    ├── update state
    │
    └── Outbox
          │
          ▼
MembershipRevoked
          │
          ├── Session
          ├── Authorization Cache
          ├── Security Monitoring
          └── Audit

El evento representa:

"Membership fue revocado."

No:

"Borra esta clave Redis."

Esto mantiene la separación definida en ADR-053.

46. Example — Policy Activation
Policy
  ↓
PolicyActivated
  ↓
Outbox
  ↓
Event Platform
  ↓
Authorization Cache Invalidation
  ↓
Fresh Authorization Evaluation

El Policy Module no manipula directamente el cache interno de Authorization.

47. Contract Registry

Deberá existir un registro de contratos Identity.

Conceptualmente:

identity/contracts/

con:

authentication
authorization
account
organization
membership
session
token
mfa
policy
recovery
federation
provisioning
audit
events

La ubicación física definitiva será definida posteriormente.

48. Architectural Dependency Validation

La matriz deberá convertirse posteriormente en reglas automatizadas.

Ejemplo conceptual:

RULE:
authorization → session
allowed = QUERY

RULE:
authorization → session.repository
allowed = FALSE

RULE:
training → identity.authorization.contract
allowed = TRUE

RULE:
training → identity.database
allowed = FALSE
49. CI/CD Enforcement

Una build podrá fallar si detecta:

Forbidden Import
Forbidden Repository Access
Cross-Module Table Access
Circular Dependency
Unversioned Contract
Unauthorized Event Consumer
Missing Tenant Context
Authorization Bypass

Esto transforma la arquitectura en una regla ejecutable.

50. Consecuencias positivas

Esta decisión proporciona:

límites claros;
menor acoplamiento;
mejor testabilidad;
evolución independiente;
mejor seguridad;
mejor tenant isolation;
mejor capacidad de observación;
mejor integración con AI;
posibilidad futura de extraer servicios;
validación automática de arquitectura.

Además permite que el equipo conozca exactamente:

"¿Cómo puedo comunicarme con este módulo?"

sin tener que inspeccionar su implementación interna.

51. Consecuencias negativas

Se introduce mayor disciplina:

más contratos;
más versionado;
más documentación;
más tests arquitectónicos;
necesidad de mantener la matriz;
mayor esfuerzo inicial.

Se acepta este coste porque Identity es un componente crítico de seguridad de toda la plataforma.

52. Alternativas rechazadas
A. Acceso directo entre módulos

Rechazado.

B. Compartir todas las entidades

Rechazado.

C. Compartir repositorios

Rechazado.

D. Microservicio por módulo

No adoptado inicialmente.

E. Comunicación exclusivamente por eventos

Rechazado porque algunas operaciones requieren respuesta inmediata.

F. Comunicación exclusivamente síncrona

Rechazado porque aumentaría acoplamiento y reduciría resiliencia.

53. No negociables
Cada módulo tiene ownership explícito.
Ningún módulo accede directamente a tablas de otro.
Ningún módulo accede directamente a repositories de otro.
Los contratos son la frontera de comunicación.
Authorization tiene enforcement central.
No existen authorization bypasses.
Tenant context es obligatorio cuando corresponda.
Cross-tenant uncertainty falla cerrado.
Domain no depende de Infrastructure.
Domain no depende de API.
No existen ciclos de dependencia.
Events no son instrucciones de infraestructura.
Cada evento tiene owner.
Los eventos críticos utilizan Outbox.
Los contratos están versionados.
Los cambios incompatibles requieren nueva versión.
AI utiliza los mismos contratos.
AI no obtiene privilegios implícitos.
MFA no concede autorización.
Authentication no concede autorización.
Role/Permission no concede autorización por sí solo.
Criticality no concede autorización.
Cache no es source of truth.
Redis no es source of truth.
Infrastructure permanece detrás de ports/adapters.
La matriz de dependencias debe ser verificable automáticamente.
Las violaciones arquitectónicas deben poder bloquear CI/CD.
Las excepciones requieren decisión arquitectónica explícita.
54. Traceability
Blueprint
Identity Domain
Domain-Driven Architecture
Authorization
Tenant Isolation
Security Policy Engine
Event Architecture
API Architecture
Testing
Observability
AI Governance
Implementation Stories
IS-001 Identity Domain Foundation
IS-002 User & Account Management
IS-003 Organization & Tenant Management
IS-004 Membership Management
IS-005 Authentication
IS-006 Authorization
IS-007 Role & Permission Management
IS-008 Session Management
IS-009 Refresh Token Lifecycle
IS-010 MFA
IS-011 Security Policy Engine
IS-012 Audit & Security Events
IS-013 Domain Events & Outbox
IS-014 Identity API Contracts
IS-015 Identity Persistence
IS-016 Identity Testing
IS-017 Identity Runtime
ADRs relacionados
ADR-001
ADR-005
ADR-006
ADR-007
ADR-014
ADR-018
ADR-019
ADR-020
ADR-021
ADR-022
ADR-043
ADR-044
ADR-045
ADR-046
ADR-047
ADR-048
ADR-049
ADR-053
ADR-054
ADR-055
ADR-056
ADR-057
ADR-058
ADR-059
ADR-062
ADR-063
ADR-064
ADR-065
ADR-066
ADR-067
ADR-068
55. Dependencias

Este ADR depende de:

Identity Domain Foundation
Authorization Runtime
Security Policy Engine
Membership
Organization
Account
Session
Token
MFA
Recovery
Event Platform
Transactional Outbox
API Contracts
Schema Registry
Observability
Testing Architecture
56. Decisiones pendientes

Este ADR deja deliberadamente algunos detalles para los siguientes artefactos de ingeniería:

56.1 Contratos exactos

Definir schemas formales para:

AuthorizationRequest
AuthorizationDecision
SecurityContext
MembershipContext
SessionContext
MFAContext
PolicyContext
56.2 Event Contracts

Cerrar:

event name
event version
payload
partition key
tenant key
metadata
compatibility
56.3 Dependency Manifest

Crear formato machine-readable para la matriz.

Ejemplo conceptual:

module: authorization

dependencies:
  - module: membership
    type: query
    contract: MembershipAuthorizationContext

forbidden:
  - membership.repository
  - membership.database
56.4 Architecture Enforcement Tooling

Definir cómo CI/CD validará:

imports
dependencies
contracts
events
cycles
database boundaries
56.5 ESP

Convertir esta decisión en una especificación técnica ejecutable.

57. Acceptance Criteria
AC-01

Todos los módulos Identity están catalogados.

AC-02

Cada módulo tiene ownership definido.

AC-03

Existe una matriz de dependencias.

AC-04

Las dependencias están clasificadas.

AC-05

Las dependencias prohibidas están explícitas.

AC-06

Los contratos son la frontera entre módulos.

AC-07

No existe acceso directo cross-module a tablas.

AC-08

No existe acceso cross-module a repositories.

AC-09

Authorization es el enforcement point central.

AC-10

Authentication no concede autorización.

AC-11

MFA no concede autorización.

AC-12

Role/Permission no concede autorización por sí solo.

AC-13

Criticality no concede autorización.

AC-14

Tenant isolation está incorporado.

AC-15

Los eventos tienen ownership.

AC-16

Los cambios críticos utilizan Outbox.

AC-17

Los contratos están versionados.

AC-18

Las dependencias circulares están prohibidas.

AC-19

La arquitectura puede validarse automáticamente.

AC-20

AI utiliza los mismos contratos de Identity.

AC-21

Las violaciones críticas pueden bloquear CI/CD.

AC-22

Las excepciones requieren aprobación arquitectónica.

58. Resultado arquitectónico

Con ADR-068 + ADR-069, Identity deja de ser solamente una colección conceptual de funcionalidades y pasa a tener una estructura modular formal:

                         IDENTITY
                            │
          ┌─────────────────┼──────────────────┐
          │                 │                  │
          ▼                 ▼                  ▼
     IDENTITY CORE     SECURITY CORE      INTEGRATION
          │                 │                  │
    ┌─────┼─────┐      ┌────┼────┐        ┌────┼────┐
    │     │     │      │    │    │        │         │
 Account Org Membership AuthZ Policy     Federation Provisioning
                       │
                  ┌────┼──────────────┐
                  │    │      │       │
                RBAC  MFA   Session  Token
                       │
                    Recovery
                       │
                 Audit / Events

Y las fronteras quedan:

┌───────────────────────────────────────────┐
│                 IDENTITY                  │
│                                           │
│  Module A ──Contract──► Module B          │
│       │                                   │
│       └──── Event ─────► Module C         │
│                                           │
│  ✗ Direct table access                   │
│  ✗ Repository sharing                    │
│  ✗ Authorization bypass                  │
│  ✗ Cross-tenant access                   │
│  ✗ Circular dependencies                 │
└───────────────────────────────────────────┘
59. Punto de transición hacia implementación

Este ADR es particularmente importante porque, junto con el ADR-068, cierra una de las últimas piezas de arquitectura estructural:

Identity
   │
   ├── What exists?        → ADRs anteriores
   ├── What does it do?    → IS
   ├── Where are limits?  → ADR-068
   ├── How interact?       → ADR-069
   └── How implement?      → ESP / Technical Tasks

Por tanto, no recomiendo seguir creando ADR indefinidamente.

El siguiente paso natural ya no es otro ADR puramente conceptual, sino empezar a convertir lo decidido en especificaciones ejecutables.

El siguiente artefacto que recomendaría preparar es:

ESP-IDENTITY-001 — Identity Domain Engineering Specification

y dentro de él consolidar:

1. Module Structure
2. Dependency Matrix
3. Domain Models
4. Aggregates
5. Value Objects
6. Application Services
7. Contracts
8. API Endpoints
9. PostgreSQL Schema
10. Event Contracts
11. Authorization Integration
12. MFA Integration
13. Session/Token Integration
14. Cache/Freshness
15. Audit/Security Events
16. Error Model
17. Testing Strategy
18. Observability
19. Security Controls
20. Deployment Requirements

Después de ese ESP podremos pasar de forma ordenada a:

ESP → API Contract → Data Model → Event Contract → IS refinement → Technical Tasks → Source Code.

Ese es el camino que nos permite empezar a programar EVOXA sin perder toda la arquitectura que hemos construido.
