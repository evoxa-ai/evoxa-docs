ADR-IDENTITY-068 — Identity Domain Integration & Module Boundaries

Status: Proposed
Version: 1.0
Date: 2026-09-07
Domain: Identity / Architecture / Domain-Driven Design / Authorization / Security / Integration
Priority: Critical
Type: Architecture / Domain Boundaries / Module Integration / Security

1. Contexto

Los ADR anteriores han definido progresivamente las capacidades de Identity y sus mecanismos de seguridad:

ADR-001: Identity como Domain/Platform Service independiente.
ADR-005: Authorization = RBAC + ABAC + Tenant Isolation + Ownership + Security Policy Engine.
ADR-006: PostgreSQL como fuente transaccional.
ADR-007: Domain Events + Transactional Outbox.
ADR-008: gestión centralizada de claves.
ADR-009: estrategia de almacenamiento de tokens.
ADR-023/024: MFA.
ADR-025: Refresh Tokens.
ADR-033–044: lifecycle, recuperación, federation, provisioning y multi-organization.
ADR-045–049: Authorization Policy y Runtime.
ADR-050–058: Cache, Freshness y Criticality.
ADR-059: Criticality Policy.
ADR-062–067: Testing, Schema Registry y Runtime Contracts.

También existe una decisión arquitectónica fundamental:

Presentation
      ↓
API
      ↓
Application
      ↓
Domain
      ↓
Infrastructure

y el Blueprint establece que Identity debe vivir como un dominio independiente dentro de la plataforma.

A medida que Identity ha crecido, existe un nuevo riesgo:

construir un único módulo Identity demasiado grande, donde Authentication, Authorization, MFA, Sessions, Organizations, Memberships, Policies, Audit y otros componentes terminen acoplados entre sí.

Por ello necesitamos definir formalmente los límites internos de Identity.

2. Problema

Sin límites explícitos podrían aparecer dependencias como:

Authentication
   ↔ Authorization
   ↔ MFA
   ↔ Session
   ↔ Organization
   ↔ Audit
   ↔ Policy
   ↔ Token

Esto produciría:

acoplamiento fuerte;
ciclos de dependencias;
dificultad para testear;
dificultad para evolucionar módulos;
riesgo de duplicar reglas de seguridad;
dificultad para separar componentes posteriormente;
riesgo de bypass de Authorization;
dificultad para integrar AI Agents;
dificultad para mantener tenant isolation.

Necesitamos establecer una estructura modular que permita:

Identity
├── Account
├── Organization
├── Membership
├── Authentication
├── Authorization
├── Session
├── Token
├── MFA
├── Security Policy
├── Recovery
├── Federation
├── Provisioning
├── Audit
└── Security Events

pero sin convertirlos en módulos completamente independientes que dupliquen seguridad.

3. Decisión

Identity se implementará como un Bounded Domain modular, compuesto por submódulos con responsabilidades explícitas.

La estructura conceptual será:

                         IDENTITY
                            │
        ┌───────────────────┼───────────────────┐
        │                   │                   │
     Identity Core     Security Core       Integration
        │                   │                   │
        ├─ Account          ├─ Authorization   ├─ Federation
        ├─ Organization     ├─ Policy          ├─ Provisioning
        ├─ Membership       ├─ MFA             └─ External IdP
        │                   ├─ Session
        │                   ├─ Token
        │                   └─ Recovery
        │
        └────────────── Audit / Events ──────────────

Los módulos comparten el Identity Domain, pero cada uno tendrá:

responsabilidad propia;
modelos propios;
invariantes propias;
servicios propios;
contratos explícitos;
eventos explícitos;
dependencias permitidas;
dependencias prohibidas.
4. Principio de Bounded Context

Identity será considerado un Bounded Context de plataforma.

Los dominios externos no podrán acceder directamente a sus tablas internas.

Por ejemplo:

Training Domain
      │
      ▼
Identity Contract
      │
      ▼
Identity

y no:

Training
   ───────► identity.users

Esto significa:

Los otros dominios conocen contratos de Identity, no su implementación interna.

5. Módulos de Identity

Se establecen inicialmente los siguientes módulos:

Módulo	Responsabilidad
Account	identidad y lifecycle de usuario
Organization	organizaciones/tenants
Membership	relación User ↔ Organization
Authentication	autenticación
Authorization	evaluación de acceso
Role & Permission	RBAC
Policy	Security Policy Engine
Session	sesiones
Token	refresh/API/service credentials
MFA	factores y challenges
Recovery	recuperación de cuenta
Federation	identidad federada
Provisioning	lifecycle externo
Audit	auditoría
Security Events	eventos de seguridad
Integration	contratos/adapters externos
6. Account Module

El módulo Account representa la identidad principal del usuario.

Responsabilidades:

creación de User;
actualización de atributos permitidos;
lifecycle;
estado de cuenta;
credenciales;
email verification;
password lifecycle;
account recovery coordination.

Estados principales:

PENDING
ACTIVE
SUSPENDED
DISABLED

El Account Module no decide por sí mismo si un usuario puede ejecutar una operación de negocio.

7. Organization Module

Representa la frontera organizacional/tenant.

Responsabilidades:

crear Organization;
actualizar configuración permitida;
lifecycle de Organization;
estado de tenant;
configuración de seguridad organizacional;
límites organizacionales.

Conceptualmente:

Organization
     │
     ├── Members
     ├── Roles
     ├── Policies
     └── Security Configuration
8. Membership Module

Membership es el límite entre User y Organization.

User
 │
 ├── Membership A → Organization A
 │
 ├── Membership B → Organization B
 │
 └── Membership C → Organization C

Responsabilidades:

crear membership;
activar;
suspender;
revocar;
eliminar;
asignar roles;
validar pertenencia organizacional;
delegation context cuando corresponda.

Membership no pertenece exclusivamente a User ni a Organization.

Es un aggregate boundary propio.

9. Authentication Module

Authentication responde:

¿Quién es el actor?

Responsabilidades:

login;
credential verification;
authentication methods;
authentication assurance;
authentication context;
integración con MFA;
creación/inicio de Session.

Authentication no debe implementar directamente las reglas completas de Authorization.

10. Authorization Module

Authorization responde:

¿Puede este actor ejecutar esta operación en este recurso y contexto?

Incluye:

RBAC
ABAC
Ownership
Delegation
Tenant Isolation
Policy Engine
Criticality
Freshness
Authorization Runtime

Este módulo es el único punto autorizado para resolver decisiones de autorización.

11. Prohibición de Authorization Bypass

Ningún dominio externo podrá implementar:

if user.role == "admin":
    allow()

como mecanismo paralelo de autorización.

Debe utilizar:

Identity Authorization Contract

Esto es especialmente importante para AI Agents.

12. Role & Permission Module

Este módulo administra el modelo RBAC:

Role
Permission
RolePermission
MembershipRole

Permisos utilizan la convención:

<resource>.<action>

Ejemplos conceptuales:

users.read
users.update
training.create
training.read
nutrition_plan.update

El Role & Permission Module proporciona permisos, pero no reemplaza al Authorization Runtime.

13. Policy Module

El Policy Module contiene:

Security Policies;
Policy lifecycle;
Policy validation;
Policy versions;
Policy evaluation;
Criticality Policies;
Policy governance.

Debe seguir los contratos definidos en:

ADR-046
ADR-048
ADR-047
ADR-059
ADR-065
ADR-066
ADR-067
14. Session Module

Session representa la sesión autenticada.

Responsabilidades:

creación;
consulta;
lifecycle;
revocación;
revoke-all;
authentication context;
device/client metadata;
session security state.

Session no genera permisos.

Authorization consulta su estado cuando corresponda.

15. Token Module

Token contiene mecanismos relacionados con:

Refresh Tokens
API Tokens
Service Credentials

El módulo deberá distinguir:

User Authentication Tokens
Service Credentials
API Tokens
Workload Identity

No deben mezclarse conceptualmente.

16. MFA Module

MFA es responsable de:

MFAFactor
MFAChallenge
RecoveryCode
TrustedDevice
Step-Up Evidence

MFA responde:

¿Se satisfizo el mecanismo adicional de autenticación requerido?

No decide por sí mismo qué operación está permitida.

17. Recovery Module

Recovery coordina:

password recovery;
account recovery;
MFA recovery;
recovery tokens;
recovery codes;
administrative recovery.

Recovery no puede convertirse en un bypass de:

MFA
Authorization
Tenant Isolation
Security Policy
18. Federation Module

Federation integra proveedores externos:

OIDC
SAML
External Identity Providers

Su responsabilidad termina en:

External Identity
       ↓
Validated Identity
       ↓
Local Identity

La autorización continúa siendo responsabilidad de EVOXA Identity.

19. Provisioning Module

Provisioning gestiona:

JIT
Invitation
Pre-provisioning
SCIM
Deprovisioning
Lifecycle synchronization

Debe respetar:

Idempotency
At-Least-Once
Tenant Isolation
Attribute Ownership
Stale Event Handling
20. Audit Module

Audit mantiene registros de:

cambios de seguridad;
autenticación;
autorización relevante;
administración;
lifecycle;
políticas;
MFA;
sessions;
recovery;
delegation.

Audit no debe convertirse en una dependencia sincrónica obligatoria para cada operación si ello introduce una dependencia crítica innecesaria.

La persistencia de auditoría seguirá las decisiones establecidas previamente.

21. Security Events Module

Los Security Events son diferentes de Audit Logs.

Audit
=
"What happened?"

Security Event
=
"What security-relevant condition occurred?"

Ejemplo:

Refresh Token Reuse
       ↓
Security Event
       ↓
Revoke Family
       ↓
Audit
22. Integration Module

Las integraciones externas deben utilizar adapters.

Ejemplos:

Email Provider
SMS Provider
OIDC Provider
SAML Provider
KMS
Secret Manager
Event Platform
Risk Provider

El Domain no debe depender directamente de SDKs externos.

23. Dependencias entre módulos

Se establece una regla de dependencia dirigida.

Conceptualmente:

Account
   ↓
Membership
   ↓
Organization

Authentication
   ↓
Session
   ↓
Token

Authentication
   ↓
MFA

Authorization
   ↓
Role & Permission
Authorization
   ↓
Policy
Authorization
   ↓
Session / MFA / Membership / Account

Pero las dependencias deberán ser mediante contratos, no acceso directo a implementación interna.

24. Regla contra ciclos

No se permiten ciclos de dependencias estructurales.

Por ejemplo:

Authentication
   ↓
Authorization
   ↓
Authentication

debe evitarse.

Cuando exista una dependencia conceptual bidireccional, se utilizará:

domain event;
application contract;
port/interface;
query contract.
25. Domain Layer

El Domain Layer contendrá:

Entities
Value Objects
Aggregates
Domain Services
Domain Events
Invariants
Policies

No debe contener:

HTTP;
SQL;
Redis;
SDKs;
framework-specific code;
secretos;
detalles de infraestructura.
26. Application Layer

Application coordina casos de uso.

Ejemplo:

LoginUser
   ↓
Validate Credentials
   ↓
Check Account
   ↓
Evaluate Security Policy
   ↓
Require MFA?
   ↓
Create Session
   ↓
Issue Tokens
   ↓
Emit Events

El Application Layer orquesta.

No debe duplicar las reglas del Domain.

27. API Layer

API traduce:

HTTP
   ↓
Application Command / Query

y:

Application Result
   ↓
HTTP Response

No debe contener:

Authorization business logic
Password hashing rules
Policy evaluation
Tenant rules
28. Infrastructure Layer

Infrastructure implementará ports/adapters para:

PostgreSQL
Redis
Event Platform
KMS
Secret Manager
Email
SMS
OIDC
SAML
External Services

Infrastructure no define las reglas de negocio.

29. Cross-Cutting Security Services

Algunos mecanismos serán transversales:

Correlation
Audit
Security Events
Cryptography
Key Management
Rate Limiting
Observability
Idempotency

Pero deberán evitar convertirse en un "God Module".

Cada uno tendrá contratos explícitos.

30. Comunicación entre módulos

Se establecen tres mecanismos:

30.1 Synchronous Application Contract

Para operaciones que requieren respuesta inmediata.

Ejemplo:

Authentication
      ↓
Session.Create()
30.2 Domain Events

Para hechos que otros módulos necesitan conocer.

Ejemplo:

MembershipRevoked
      ↓
Session Security Handler
      ↓
Revoke affected sessions
30.3 Integration Events

Para comunicar Identity con otros dominios/plataforma.

Ejemplo:

UserCreated
MembershipActivated
UserSuspended
31. Regla Events vs Direct Calls

No todo debe convertirse en evento.

Usar llamada síncrona cuando:

el resultado sea necesario inmediatamente

Usar evento cuando:

el consumidor pueda reaccionar posteriormente

Ejemplo:

Login
→ Session creation

puede ser síncrono.

Mientras:

UserSuspended
→ Notification
→ Analytics
→ Security Monitoring

puede utilizar eventos.

32. Aggregate Boundaries

Los aggregates iniciales serán conceptualmente:

User
Organization
Membership
Session
RefreshTokenFamily
MFAFactor
MFAChallenge
SecurityPolicy
Role
Permission
Delegation
AuditEvent
SecurityEvent
OutboxEvent

Cada aggregate deberá proteger sus propias invariantes.

33. Regla de acceso a aggregates

Un módulo no podrá modificar directamente el aggregate interno de otro módulo.

Incorrecto:

Authorization
   ↓
UPDATE sessions

Correcto:

Authorization
   ↓
Session Contract

o:

SessionRevocationRequested

según el caso.

34. Shared Kernel

Se permitirá un Shared Kernel mínimo.

Contendrá únicamente conceptos verdaderamente compartidos.

Por ejemplo:

UserId
OrganizationId
MembershipId
SessionId
CorrelationId
TenantId
DomainEvent metadata

No deberá contener:

AuthenticationService
AuthorizationService
PolicyEngine
MFAService

porque eso crearía acoplamiento.

35. Identity Public Contract

Los dominios externos interactuarán con Identity únicamente mediante contratos públicos.

Conceptualmente:

Identity Public Contracts
├── Authentication
├── Authorization
├── Identity Lookup
├── Membership
├── Organization
├── Security Context
└── Events

Los contratos serán independientes de:

PostgreSQL schema
Python classes
Redis keys
internal tables
internal modules
36. Domain Integration Example

Un Training Domain necesita comprobar:

Can user X edit training session Y?

No debe hacer:

SELECT role FROM users...

Debe hacer:

AuthorizationRequest
       ↓
Identity Authorization Runtime
       ↓
Decision

con:

actor
tenant
resource
action
ownership
context
37. AI Integration

Un AI Agent también utilizará:

AuthorizationRequest

No existirá:

AI bypass

El flujo será:

AI Agent
    ↓
Agent Identity
    ↓
Authorization Runtime
    ↓
Policy
    ↓
Criticality
    ↓
Freshness
    ↓
Decision
38. Tenant Isolation Boundary

Tenant isolation se tratará como una responsabilidad transversal, pero el Authorization Runtime será el enforcement point.

Los módulos deben preservar:

tenant_id
organization_id
membership_id

cuando corresponda.

Nunca debe existir una consulta cross-tenant accidental.

39. Data Ownership

Cada módulo será dueño de sus datos.

Conceptualmente:

Account
  → User identity data

Organization
  → Organization data

Membership
  → Membership data

Session
  → Session data

Token
  → Token data

MFA
  → MFA data

Policy
  → Policy data

Audit
  → Audit data

Otros módulos consumen contratos, no tablas.

40. Transaction Boundary

Una transacción debe permanecer, por defecto, dentro de un aggregate/module boundary.

Cuando una operación involucre múltiples módulos:

Application Orchestrator
       ↓
Transaction / Outbox
       ↓
Events
       ↓
Other Modules

No se recomienda crear transacciones distribuidas entre módulos como mecanismo normal.

41. Outbox Integration

Los eventos derivados de cambios de estado deberán seguir:

Domain Change
      ↓
DB Transaction
      +
Outbox
      ↓
Commit
      ↓
Event Platform

Esto aplica especialmente a:

MembershipRevoked
RoleChanged
PolicyActivated
SessionRevoked
MFAFactorRevoked
AccountSuspended
42. Security-Critical Events

Los siguientes cambios requieren especial atención:

Account Suspension
Membership Revocation
Role Removal
Permission Removal
Session Revocation
Refresh Family Revocation
MFA Revocation
Delegation Revocation
Policy Tightening
Tenant Security Change

Deben integrarse con:

Authorization Cache
Security Epoch
Audit
Security Events

según los ADR anteriores.

43. Module Dependency Rule

Cada módulo deberá declarar:

Allowed Dependencies
Forbidden Dependencies
Public Contracts
Events Published
Events Consumed
Owned Aggregates
Owned Tables
Security Responsibilities

Esto deberá quedar formalizado en los ESP.

44. Architecture Enforcement

La arquitectura deberá poder validarse automáticamente.

Ejemplo:

authorization/
   ├── domain/
   ├── application/
   ├── infrastructure/
   └── api/

No deberá permitirse:

domain → infrastructure
domain → api

ni:

external-domain → identity.infrastructure
45. Testing de límites

Los límites de módulos deberán tener tests específicos.

Architectural Tests

Validar:

No forbidden dependency
No cyclic dependency
No infrastructure dependency from domain
No direct database access between modules
No authorization bypass
No cross-tenant access
46. Consecuencias positivas

Esta decisión permite:

reducir acoplamiento;
evolucionar módulos independientemente;
facilitar testing;
facilitar futuras extracciones de servicios;
mantener límites de seguridad;
evitar duplicación;
facilitar integración con AI;
facilitar observabilidad;
simplificar migraciones futuras.

Además permite que Identity evolucione desde:

Modular Monolith

hacia:

Selective Services

si posteriormente existe una razón real para hacerlo.

47. Modular Monolith como estrategia inicial

No se establece que cada módulo de Identity deba convertirse inmediatamente en un microservicio.

La decisión inicial es:

Modularidad lógica fuerte antes de distribución física.

Conceptualmente:

                Identity
                   │
        ┌──────────┼──────────┐
        ▼          ▼          ▼
     Account   Authorization  MFA
        │          │          │
        └──────────┼──────────┘
                   ▼
              PostgreSQL

Esto reduce complejidad operacional mientras mantiene límites arquitectónicos.

48. Evolución futura

Si un módulo requiere separación física, deberá existir evidencia:

Performance
Scalability
Security Isolation
Team Ownership
Deployment Independence
Failure Isolation
Compliance
Operational Need

No se separará un módulo simplemente porque "es un microservicio".

49. Alternativas rechazadas
A. Identity como un único módulo monolítico

Rechazado.

Produce demasiado acoplamiento.

B. Microservicio por cada capacidad

Rechazado inicialmente.

Introduce complejidad distribuida prematura.

C. Cada dominio implementa su propia autorización

Rechazado.

Generaría inconsistencias y posibles bypasses.

D. Compartir directamente todas las tablas

Rechazado.

Rompe bounded contexts.

E. Utilizar eventos para absolutamente todo

Rechazado.

Algunas operaciones requieren respuesta sincrónica.

50. No negociables
Identity es un bounded domain.
Identity tiene módulos internos explícitos.
Cada módulo tiene responsabilidades definidas.
Cada módulo tiene ownership de sus datos.
Los dominios externos no acceden directamente a tablas Identity.
Authorization tiene un único enforcement point.
No existen authorization bypasses.
Tenant isolation es obligatorio.
Authentication no reemplaza Authorization.
Criticality no reemplaza Authorization.
MFA no reemplaza Authorization.
Role & Permission no reemplaza Authorization Runtime.
Domain no depende de Infrastructure.
Domain no depende de API.
No existen ciclos de dependencia.
Cross-module access utiliza contracts.
Eventos se utilizan para integración asincrónica.
Outbox protege eventos derivados de cambios transaccionales.
Shared Kernel debe permanecer mínimo.
AI utiliza los mismos límites de seguridad.
Test Runtime utiliza los mismos contratos.
Security-critical state changes deben integrarse con cache/freshness.
Cada módulo debe declarar ownership y dependencias.
Modular Monolith es la estrategia inicial.
La distribución física futura requiere justificación arquitectónica.
51. Traceability
Blueprint
Identity
Domain Architecture
Authorization
Security
Multi-Tenancy
Event Architecture
API Architecture
AI-Native Architecture
Implementation Stories
IS-001 — Identity Domain Foundation
IS-002 — User & Account Management
IS-003 — Organization & Tenant Management
IS-004 — Membership Management
IS-005 — Authentication
IS-006 — Authorization
IS-007 — Role & Permission Management
IS-008 — Session Management
IS-009 — Refresh Token Lifecycle
IS-010 — MFA
IS-011 — Security Policy Engine
IS-012 — Audit & Security Events
IS-013 — Domain Events & Outbox
IS-014 — Identity API Contracts
IS-015 — Identity Persistence
IS-016 — Identity Testing
IS-017 — Identity Runtime
Related ADRs
ADR-001
ADR-005
ADR-006
ADR-007
ADR-023–044
ADR-045–059
ADR-062–067
52. Dependencias

Este ADR depende principalmente de:

Identity Domain Foundation
Authorization Runtime
Security Policy Engine
Membership
Organization/Tenant
Authentication
Session
MFA
Token
Audit
Event Platform
Transactional Outbox
API Contracts
Schema Registry
Testing Framework
Observability
53. Decisiones pendientes

Para convertir esta decisión en implementación todavía debemos concretar:

53.1 Module Dependency Matrix

Definir formalmente:

Module A → Module B
ALLOW / FORBIDDEN / EVENT ONLY
53.2 Aggregate Ownership

Cerrar cada aggregate y su owner definitivo.

53.3 Public Contracts

Definir interfaces exactas para:

Authentication
Authorization
Membership
Organization
Session
MFA
Policy
53.4 Event Ownership

Definir qué módulo publica y consume cada evento.

53.5 Persistence Boundaries

Definir tablas, schemas PostgreSQL y ownership exacto.

53.6 Package/Folder Structure

Definir la estructura definitiva del código.

53.7 Cross-Module Transactions

Identificar cuáles requieren coordinación síncrona y cuáles eventos.

54. Acceptance Criteria
AC-01

Identity está definido como bounded domain.

AC-02

Los módulos internos están definidos.

AC-03

Cada módulo tiene responsabilidad explícita.

AC-04

Cada aggregate tiene owner.

AC-05

Cada módulo tiene data ownership.

AC-06

No existe acceso directo cross-module a tablas.

AC-07

Authorization tiene un enforcement point único.

AC-08

Tenant isolation está integrado en los límites.

AC-09

No existen ciclos de dependencia.

AC-10

Domain no depende de Infrastructure.

AC-11

Domain no depende de API.

AC-12

Los contratos públicos de Identity están separados de implementación.

AC-13

Los eventos tienen ownership.

AC-14

Los cambios críticos utilizan Outbox.

AC-15

Shared Kernel permanece mínimo.

AC-16

AI utiliza los mismos contratos.

AC-17

Test Runtime utiliza los mismos contratos.

AC-18

Los límites arquitectónicos pueden validarse automáticamente.

AC-19

La estrategia inicial es Modular Monolith.

AC-20

La futura extracción de servicios requiere justificación arquitectónica.

55. Resultado arquitectónico

Con este ADR la estructura de Identity queda mucho más definida:

                         EVOXA
                           │
                           ▼
                    ┌─────────────┐
                    │   IDENTITY  │
                    │  BOUNDED    │
                    │   DOMAIN    │
                    └──────┬──────┘
                           │
       ┌───────────────────┼───────────────────┐
       │                   │                   │
       ▼                   ▼                   ▼
 Identity Core       Security Core       Integration
       │                   │                   │
 Account             Authorization        Federation
 Organization        RBAC                Provisioning
 Membership           Policy
                      Session
                      Token
                      MFA
                      Recovery
       │                   │                   │
       └───────────────────┼───────────────────┘
                           ▼
                    Audit / Events
                           │
                           ▼
                  PostgreSQL / Redis
                  Event Platform / KMS

Y hacia afuera:

Training
Nutrition
Workout
Payments
AI Agents
Mobile
Web
External Systems
       │
       ▼
Identity Public Contracts
       │
       ▼
Authorization Runtime

La consecuencia más importante es que ya no estamos diseñando solamente funcionalidades de Identity; estamos definiendo la frontera arquitectónica que protegerá todos los dominios posteriores de EVOXA.

Siguiente paso recomendado

El siguiente ADR debería ser:

ADR-IDENTITY-069 — Identity Module Dependency Matrix, Contracts & Interaction Rules

Este ADR debería cerrar exactamente lo que dejamos pendiente en el 068:

Módulo
   ↓
Puede llamar a
   ↓
Puede recibir eventos de
   ↓
Puede publicar eventos
   ↓
Puede consultar
   ↓
No puede acceder
   ↓
Contrato utilizado

Después de 069, yo recomendaría hacer un pequeño bloque de cierre de arquitectura —probablemente ADR-070/071— y luego dejar de crear ADR indefinidamente y pasar a:

ADR → ESP → API Contracts → Persistence Model → Event Contracts → Test Specifications → Technical Tasks → Implementación.

Ese es el punto importante: no necesitamos tener cientos de ADR antes de programar. Ya estamos acercándonos al momento de congelar las decisiones arquitectónicas y empezar a convertirlas en especificaciones ejecutables.
