ESP-0001/01 — Identity Domain Model
Versión consolidada propuesta

Document ID: ESP-0001/01
Module: Identity
Version: 1.1.0
Status: Draft — Consolidated with ADR-001–069
Parent: ESP-0001 — Identity
Architecture Parent: BP-0002 — Reference Architecture
Domain: Identity
Target Repository: evoxa-platform
Target Path: apps/api/app/domains/identity/

1. Purpose

Este documento define el modelo de dominio de Identity de EVOXA.

Identity constituye una capacidad transversal responsable de establecer:

quién es un usuario;
cómo se autentica;
qué identidad posee;
a qué organizaciones pertenece;
qué memberships posee;
qué roles y permisos tiene dentro de cada contexto organizacional;
qué contexto de tenant/organización está utilizando;
qué sesiones y credenciales están activas;
qué políticas de seguridad son aplicables;
cómo se evalúa la autorización;
cómo se controla MFA;
cómo se registran eventos de auditoría y seguridad;
cómo se integran los demás dominios con Identity.

Identity no incorpora lógica propia de los dominios de negocio.

2. Architectural Context

Identity forma parte de la arquitectura por capas:

Presentation
      ↓
API
      ↓
Application
      ↓
Domain
      ↓
Infrastructure

El dominio Identity constituye un Bounded Domain de EVOXA.

Su implementación inicial seguirá un enfoque de Modular Monolith, manteniendo límites que permitan una futura extracción de módulos o servicios cuando exista evidencia arquitectónica que lo justifique.

La estructura concreta de implementación es diseño derivado y no debe interpretarse como una imposición literal del Blueprint.

La separación de módulos y sus reglas de interacción están definidas por:

ADR-068 — Identity Domain Integration & Module Boundaries
ADR-069 — Identity Module Dependency Matrix, Contracts & Interaction Rules
3. Identity Responsibilities

Identity es responsable de las siguientes capacidades:

Identity
│
├── Account
├── Organization
├── Membership
├── Authentication
├── Authorization
├── Role & Permission Management
├── Security Policy
├── Session Management
├── Token Management
├── Multi-Factor Authentication
├── Account Recovery
├── Federation
├── Provisioning
├── Audit
├── Security Events
└── Integration

Estas capacidades se implementan mediante módulos internos claramente delimitados.

4. Non-Responsibilities

Identity no debe implementar lógica propia de:

Training
Nutrition
Recovery
Human Digital Twin
Recommendations
Analytics
Billing
Community
Marketplace
Business-specific AI

Los demás dominios consumen Identity mediante contratos.

Ejemplo:

Training
    │
    │ "¿Puede este actor modificar este recurso?"
    ▼
Identity Authorization
    │
    ▼
Authorization Decision

Identity determina si una operación está autorizada.

Identity no determina si la operación tiene sentido funcional dentro del dominio Training.

5. Domain Model

El modelo conceptual consolidado es:

                       ┌───────────────┐
                       │     User      │
                       └───────┬───────┘
                               │
                    ┌──────────┴──────────┐
                    │                     │
              Credentials             Membership
                                          │
                              ┌───────────┴───────────┐
                              │                       │
                        Organization              Roles
                                                      │
                                                 Permissions

Una identidad global puede pertenecer a múltiples organizaciones:

User
 │
 ├── Membership → Organization A
 │                   └── Roles
 │
 ├── Membership → Organization B
 │                   └── Roles
 │
 └── Membership → Organization C
                     └── Roles

Esta decisión queda establecida por ADR-043 — Multi-Organization Identity.

Por tanto:

Membership constituye el límite de acceso organizacional.

6. User
6.1 Purpose

User representa la identidad principal de una persona dentro de EVOXA.

Un User no representa por sí mismo un rol profesional.

Una misma identidad puede participar en diferentes organizaciones y contextos.

Ejemplo:

User
 ├── Membership → Organization A
 │                  └── Coach
 │
 └── Membership → Organization B
                    └── Team Manager
7. User Lifecycle

Los estados fundamentales del User son:

PENDING
   │
   ▼
ACTIVE
   │
   ├──────────────► SUSPENDED
   │                    │
   │                    ▼
   │                ACTIVE
   │
   └──────────────► DISABLED

El modelo distingue claramente:

estado de cuenta;
bloqueo temporal;
suspensión;
deshabilitación;
estado de membership.

Esto evita utilizar un único estado para representar situaciones de seguridad diferentes.

La semántica de lifecycle y lockout se encuentra respaldada por ADR-033 — Account Lifecycle & Lockout.

8. Organization
8.1 Purpose

Organization representa el contexto organizacional que constituye la frontera principal de tenant.

Ejemplos conceptuales:

Professional Practice
Fitness Center
Nutrition Clinic
Sports Organization
Enterprise

La Organization constituye una frontera fundamental para:

autorización;
aislamiento de datos;
memberships;
roles;
políticas;
delegaciones;
auditoría.
9. Membership

Membership representa:

User ↔ Organization

Una membership determina la participación de un User dentro de una Organization.

User
 │
 ├── Membership A
 │       └── Organization A
 │
 └── Membership B
         └── Organization B

La autorización organizacional se evalúa utilizando la Membership como parte del contexto.

10. Membership Roles

Una Membership puede tener múltiples roles.

Membership
 │
 ├── Role A
 ├── Role B
 └── Role C

No existe una jerarquía implícita de roles.

La capacidad efectiva resulta de la combinación de:

Membership
+
Roles
+
Permissions
+
Ownership
+
ABAC
+
Delegation
+
Security Policies
+
Context

Esta decisión está definida principalmente por ADR-044.

11. Role

Un Role representa una agrupación lógica de permisos.

Ejemplo:

Coach
 ├── clients.read
 ├── clients.update
 ├── training.read
 └── training.create

Role no constituye por sí mismo una autorización.

Es solamente uno de los inputs utilizados por el Authorization Runtime.

12. Permission

Una Permission representa una capacidad autorizable.

El modelo conceptual utiliza:

<resource>.<action>

Ejemplos:

clients.read
clients.create
clients.update
clients.delete

training.read
training.create
training.update

nutrition.read
nutrition.create

La Permission no debe confundirse con:

Role;
Policy;
Criticality;
Authorization Decision.
13. RolePermission

Representa la relación:

Role
 │
 └── RolePermission
          │
          └── Permission

Debe garantizarse la unicidad de:

(role_id, permission_id)

para evitar asignaciones duplicadas.

14. Delegation

Identity incorpora delegación explícita.

Actor A
   │
   │ delegation
   ▼
Actor B

La delegación debe ser:

explícita;
limitada;
revocable;
temporal cuando corresponda;
tenant-aware;
auditable.

La delegación nunca puede proporcionar al delegado privilegios superiores a los que posee el delegador.

Delegated Capability
        ≤
Delegator Effective Capability

La profundidad de delegación inicial es limitada y no debe existir escalamiento implícito.

Basado en ADR-044.

15. Session

Una Session representa un contexto autenticado activo.

User
 │
 ├── Session
 ├── Session
 └── Session

Una misma identidad puede tener sesiones simultáneas en:

Mobile
Web
Coach Portal
Admin Portal

Una Session tiene relación con:

User;
Membership/contexto organizacional;
authentication method;
MFA state;
lifecycle;
revocation;
expiration;
device/client context.

La Session es stateful.

Esto queda establecido por ADR-003 — Session State & Refresh Token Lifecycle.

16. Refresh Token Family

Los Refresh Tokens forman parte de una familia asociada a una Session.

Family
 │
 ├── Token 1
 │      │
 │      ▼
 ├── Token 2
 │      │
 │      ▼
 ├── Token 3
 │
 └── Token N

La rotación es atómica.

El reuse de un Refresh Token utilizado previamente constituye una condición de seguridad.

Ante reuse confirmado:

Refresh Token Reuse
        ↓
Security Event
        ↓
Audit
        ↓
Revoke Family
        ↓
Revoke Session

Basado en ADR-002, ADR-003 y ADR-025.

17. MFA

MFA constituye un conjunto independiente de conceptos:

MFAFactor
MFAChallenge
RecoveryCode
TrustedDevice

Los factores pueden incluir conceptualmente:

TOTP
WebAuthn
Passkey
SMS
Email
Push
Hardware Key

No todos necesariamente deben estar disponibles en la primera implementación.

El Policy Engine determina cuándo MFA es obligatorio.

MFA determina cómo se verifica el factor.

Policy Engine
      │
      │ REQUIRE_MFA
      ▼
MFA
      │
      ▼
Verification

Basado en ADR-023 y ADR-024.

18. Recovery

Account Recovery no constituye un bypass de seguridad.

La recuperación debe mantener un nivel de assurance compatible con la operación.

Especialmente:

MFA Recovery
Password Recovery
Account Recovery
Administrative Recovery

deben mantener controles específicos.

AI no puede:

desactivar MFA;
emitir recovery credentials;
aprobar recuperación;
modificar security state.

Basado en ADR-035.

19. Authentication

Authentication responde:

¿Quién es el actor?

Conceptualmente:

Credential
     ↓
User
     ↓
Account State
     ↓
Security Policy
     ↓
MFA if required
     ↓
Session
     ↓
Access Token
     ↓
Refresh Token

Authentication no equivale a Authorization.

20. Authorization Model

El modelo de autorización consolidado es:

Authorization Request
        ↓
Request Validation
        ↓
Actor Authentication
        ↓
Tenant Guard
        ↓
Authorization Context
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

Este modelo reemplaza la representación simplificada anterior:

Authentication
↓
Role
↓
Permissions
↓
Policy
↓
ALLOW / DENY

La especificación actual debe considerar Authorization como un runtime centralizado de evaluación, definido principalmente por ADR-045–059.

21. Authorization Decision

Las decisiones posibles son:

ALLOW
DENY
CHALLENGE
REAUTHENTICATE
REQUIRE_MFA
REVOKE

Un fallo interno de evaluación no puede transformarse en ALLOW.

Conceptualmente:

UNKNOWN
   ↓
Fail Closed

ERROR
   ↓
Fail Closed
22. Criticality

La Criticality no constituye una autorización.

Determina el nivel de seguridad/freshness requerido para evaluar una operación.

Base Criticality
        +
Dynamic Risk
        +
Context
        +
Security State
        +
Tenant
        +
Delegation
        +
MFA
        +
Platform Minimum
        ↓
Effective Criticality

Se utiliza:

C0 — Critical Security
C1 — High Security
C2 — Standard
C3 — Low Risk

La combinación es monotónica:

C3 → C2 → C1 → C0

Nunca se puede reducir artificialmente la criticidad.

Basado en ADR-058 y ADR-059.

23. Freshness

La autorización utiliza clases de freshness:

F0 — Authoritative
F1 — Security Validated
F2 — Controlled Eventual
F3 — Restricted Degraded

Relación base:

Criticality	Freshness
C0	F0
C1	F1
C2	F2-SHORT
C3	F2-MEDIUM

Las revocaciones de seguridad tienen prioridad sobre TTL.

Security Revocation
        >
Security Epoch
        >
Policy Version
        >
Entity Version
        >
Context
        >
TTL

Basado en ADR-056–059.

24. Tenant Isolation

El contexto tenant debe ser explícito.

Request
   ↓
Authentication
   ↓
User
   ↓
Membership
   ↓
Organization
   ↓
Tenant Context
   ↓
Authorization

Ejemplo:

User A
Tenant A
   │
   ▼
Resource Tenant B
   │
   ▼
DENY

La incertidumbre respecto al tenant no puede interpretarse como autorización.

Tenant Unknown
      ↓
DENY

El acceso cross-tenant sólo puede existir mediante una capacidad explícita, autorizada y auditable.

Basado en ADR-043–045.

25. Ownership

Ownership constituye un atributo independiente de Role y Permission.

Ejemplo:

Coach
+
clients.update
+
owns(client)

puede producir autorización.

Pero:

Coach
+
clients.update

no implica automáticamente ownership.

Ownership debe evaluarse dentro del Authorization Runtime.

26. Security Policy

Las políticas de seguridad se evalúan mediante el Policy Engine.

Las políticas poseen:

ID
Name
Version
Status
Scope
Priority
Effect
Target
Conditions
Obligations
Metadata
Effective Period

Lifecycle:

DRAFT
   ↓
VALIDATING
   ↓
APPROVED
   ↓
ACTIVE
   ↓
SUPERSEDED
   ↓
RETIRED

Los scopes conceptuales son:

PLATFORM
ORGANIZATION
RESOURCE

Basado en ADR-046–048.

27. Policy Effects

El Policy Engine puede producir:

ALLOW
DENY
CHALLENGE
REAUTHENTICATE
REQUIRE_MFA
REVOKE

La precedencia de seguridad debe impedir que una política de menor autoridad debilite una política de plataforma.

Conceptualmente:

Regulatory / Mandatory Security
        ↓
Platform Security Minimum
        ↓
Security State
        ↓
Tenant Isolation
        ↓
Resource Security
        ↓
Dynamic Risk
        ↓
Context
        ↓
Organization
        ↓
Default

Basado en ADR-046 y ADR-059.

28. Authorization Policy DSL

Las políticas utilizan un modelo declarativo, estructurado y tipado.

No se permite:

arbitrary code;
arbitrary SQL;
network calls;
secrets;
side effects.

Las condiciones se expresan mediante una estructura tipada:

AND
OR
NOT

con referencias conceptuales como:

subject.*
resource.*
organization.*
membership.*
session.*
request.*
environment.*
delegation.*

La implementación exacta del DSL, AST y compilador queda gobernada por ADR-048 y su implementación posterior.

29. AI and Identity Authorization

Los AI Agents forman parte del modelo de actores de Identity.

Un AI Agent:

tiene identidad;
tiene contexto;
tiene permisos;
está sujeto a tenant isolation;
está sujeto a políticas;
está sujeto a criticality;
está sujeto a freshness;
está sujeto a auditabilidad.

AI no puede:

bypass authorization
reduce criticality
disable MFA
change platform minimums
activate security policies
approve security regressions

AI puede asistir en:

análisis;
generación de escenarios;
detección de anomalías;
recomendaciones.

Pero las decisiones de seguridad permanecen bajo los mecanismos normativos de Identity.

30. Domain Events

Identity genera eventos cuando ocurren hechos significativos.

Los eventos son facts, no commands.

Ejemplos conceptuales:

UserCreated
UserUpdated
UserActivated
UserSuspended
UserDisabled

OrganizationCreated

MembershipCreated
MembershipSuspended
MembershipRevoked

RoleCreated
RoleUpdated

PermissionAssigned
PermissionRevoked

SessionCreated
SessionRevoked

UserAuthenticated
UserLoggedOut

MFAFactorEnrolled
MFAFactorRevoked

SecurityPolicyActivated
SecurityPolicyRetired

Los nombres definitivos deben quedar en el Event Contract Catalog.

31. Event Envelope

Los eventos siguen el envelope común de EVOXA:

{
  "event_id": "...",
  "event_type": "...",
  "version": 1,
  "timestamp": "...",
  "tenant_id": "...",
  "actor": "...",
  "resource": "...",
  "payload": {},
  "metadata": {
    "correlation_id": "...",
    "causation_id": "...",
    "trace_id": "..."
  }
}

Los eventos deben ser:

versionados;
inmutables;
tenant-aware;
correlacionables;
idempotentemente consumibles;
compatibles con at-least-once delivery;
compatibles con retry/DLQ/replay.
32. Transactional Outbox

Los cambios de dominio que requieren publicación de eventos deben utilizar Transactional Outbox.

┌──────────────────────────────┐
│ PostgreSQL Transaction       │
│                              │
│ Identity State Change        │
│            +                 │
│ Outbox Event                 │
└──────────────┬───────────────┘
               ↓
       Event Publisher
               ↓
        Event Platform

La escritura del estado y del Outbox debe formar parte de la misma transacción cuando corresponda.

Esto queda establecido por ADR-007 y ADR-013.

33. Repository Interfaces

El dominio no depende directamente de PostgreSQL.

Conceptualmente:

Domain
   ↓
Repository Interface
   ↓
Infrastructure
   ↓
PostgreSQL

Ejemplos:

UserRepository
OrganizationRepository
MembershipRepository
RoleRepository
PermissionRepository
SessionRepository

Los módulos no deben acceder directamente a repositories o tablas internas de otros módulos.

Esta regla está reforzada por ADR-068 y ADR-069.

34. Aggregate Model

El modelo consolidado propone como agregados principales:

User
Organization
Membership
Role
Session
RefreshTokenFamily
MFAFactor
MFAChallenge
SecurityPolicy
Delegation

Sin embargo, la elección final de aggregate boundaries debe respetar:

invariantes;
consistencia transaccional;
ownership;
concurrencia;
publicación de eventos;
límites de módulo.

No se debe asumir que cada entidad constituye automáticamente un aggregate.

35. Domain Invariants

Se establecen como invariantes fundamentales:

INV-001 — User Identity

Un User debe poseer un identificador único.

INV-002 — Permission Identity

Una Permission debe identificarse de forma única.

INV-003 — Role Permission Uniqueness

No puede existir duplicación:

Role + Permission
INV-004 — Session Revocation

Una Session revocada no puede volver a considerarse válida.

INV-005 — Valid Identity Context

Una operación autenticada debe disponer de un contexto de identidad válido.

INV-006 — Tenant Isolation

Un actor no puede acceder implícitamente a recursos de otro tenant.

INV-007 — Delegation Boundary

Una delegación no puede ampliar los privilegios efectivos del delegador.

INV-008 — Fail Closed

Unknown/Error no puede convertirse en autorización permisiva.

INV-009 — Platform Minimum

Una política organizacional no puede reducir un mínimo de seguridad de plataforma.

INV-010 — Criticality Monotonicity

Una evaluación posterior no puede reducir una criticality requerida por una condición de mayor autoridad.

INV-011 — No Privilege Amplification

Ninguna combinación de roles, delegation, policy o AI context puede generar privilegios superiores a los permitidos por las restricciones aplicables.

INV-012 — Cache Is Not Source of Truth

El cache no constituye autoridad sobre el estado de seguridad.

Estas últimas invariantes deben mantenerse coordinadas con los catálogos de invariantes definidos en ADR-062–067.

36. Security Requirements

Identity debe cumplir:

Zero Trust
Least Privilege
Defense in Depth
Secure by Default
Privacy by Design
Continuous Verification
Auditability
Tenant Isolation
Fail Closed

Los secretos no deben aparecer en:

Logs
Events
Audit
API Responses
AI Context
Source Code

La gestión criptográfica se rige por ADR-008 y ADR-027.

37. Audit

Las operaciones críticas deben ser auditables.

Ejemplos:

Authentication
Failed Authentication
Logout
Password Change
MFA Change
Role Assignment
Permission Assignment
Membership Change
Account Activation
Account Suspension
Session Revocation
Refresh Token Reuse
Policy Activation
Delegation Change

La auditoría debe permitir establecer:

WHO
WHAT
WHEN
WHERE
WHY / CONTEXT
RESULT

Audit es diferente de:

Operational Logs
Security Events
Domain Events
38. Observability

Identity debe producir:

Logs
Metrics
Traces
Audit Events
Security Events
Domain Events

Las operaciones deben poder correlacionarse mediante:

trace_id
span_id
correlation_id
causation_id

Los AI Agents y servicios también deben ser trazables.

39. Testing Requirements

El modelo de dominio debe ser validado mediante:

Unit Tests
Component Tests
Integration Tests
API Tests
Security Tests
Authorization Tests
Tenant Isolation Tests
Session Tests
MFA Tests
Event Tests
Performance Tests
Chaos Tests
Recovery Tests

Además, para Authorization Criticality:

Golden Scenarios
Security Invariants
Generated Scenarios
Property-Based Tests
Boundary Tests
Mutation Tests
Differential Tests
Regression Tests

La ejecución debe producir evidencia reproducible.

Debe conservarse la combinación:

Policy Version
Runtime Version
Schema Version
Catalog Version
Scenario Version
Invariant Version
Generator Version

Basado en ADR-062–067.

40. Traceability

La trazabilidad queda:

Blueprint
   ↓
Reference Architecture
   ↓
Engineering Standards
   ↓
Engineering Specifications
   ↓
ESP-0001 Identity
   ↓
ESP-0001/01 Domain Model
   ↓
Implementation Stories
   ↓
Technical Tasks
   ↓
Source Code
   ↓
Tests
   ↓
Deployment
   ↓
Monitoring

Cada implementación debe poder rastrearse hasta su requisito y decisión arquitectónica correspondiente.

41. ADR Traceability

Este documento queda principalmente relacionado con:

Tema	ADR
Identity Architecture	ADR-001
Access/Refresh Tokens	ADR-002
Sessions	ADR-003
Authorization	ADR-005
Persistence	ADR-006
Events/Outbox	ADR-007
Cryptography	ADR-008
Token Storage	ADR-009
Testing	ADR-010
Runtime	ADR-011
MFA	ADR-023–024
Refresh Token Security	ADR-025
API Idempotency/Pagination	ADR-026
Secrets/KMS	ADR-027
Observability	ADR-028
DR	ADR-029–030
HA	ADR-031
Rate Limiting	ADR-032
Account Lifecycle	ADR-033
Password Security	ADR-034
Recovery	ADR-035
Email Verification	ADR-036
API Tokens	ADR-037
S2S	ADR-038
OAuth/OIDC	ADR-039
Federation	ADR-040
Provisioning	ADR-041
Attribute Ownership	ADR-042
Multi-Organization	ADR-043
Roles/Delegation	ADR-044
Policy Model	ADR-045–049
Authorization Cache	ADR-050–057
Criticality	ADR-058–059
Criticality Testing	ADR-062–063
Test Data/Evidence	ADR-064
Schema Evolution	ADR-065
Schema Registry	ADR-066
Runtime Contracts	ADR-067
Module Boundaries	ADR-068
Dependency Matrix	ADR-069
42. Implementation Readiness

Después de esta consolidación, ESP-0001/01 deja de tener las antiguas preguntas:

¿Membership será formal?
¿Multi-organization?
¿Roles globales?
¿JWT?
¿Refresh Token?
¿MFA?
¿Redis?
¿Outbox?
¿RBAC + ABAC?

como decisiones abiertas.

Esas decisiones ya están representadas por los ADR correspondientes.

Por tanto, el estado pasa a ser:

Domain Purpose              ✅
Responsibilities             ✅
Non-Responsibilities         ✅
User Model                   ✅
Organization Model           ✅
Membership Model             ✅
Multi-Organization           ✅
Role Model                   ✅
Permission Model             ✅
Delegation                   ✅
Session Model                ✅
Refresh Token Model          ✅
MFA                          ✅
Authorization Model          ✅
Policy Engine                ✅
Criticality                  ✅
Freshness                    ✅
Tenant Isolation             ✅
Domain Events                🟡 Contracts pending
Persistence                  🟡 ESP-011
API Contracts                🟡 ESP-010
Event Contracts              🟡 ESP-012
Testing                      🟡 ESP-013
Implementation               🟡 ESP-014
Deployment                   🟡 ESP-015
43. Decisions Still Open

Después de ADR-001 → 069, las decisiones realmente abiertas ya no son de arquitectura fundamental del dominio.

Quedan principalmente decisiones de especificación/implementación:

1. Aggregate boundaries definitivos
2. Machine-readable domain schemas
3. Exact Permission catalog
4. Exact Event catalog
5. Exact API contract
6. Exact PostgreSQL schema
7. Exact Policy DSL syntax
8. Exact policy AST
9. Exact cache implementation
10. Exact provider configuration
11. Exact MFA factor rollout
12. Exact operational thresholds

Estas decisiones deben resolverse en los ESP correspondientes o Technical Tasks, no mediante ADR adicionales salvo que aparezca una decisión arquitectónica nueva.

44. Cambio fundamental respecto al documento original

El cambio más importante es este:

Antes
Identity Domain Model
       │
       ├── User
       ├── Organization
       ├── Membership
       ├── Role
       ├── Permission
       └── Session

Authorization
       ↓
RBAC + ABAC
Ahora
                       IDENTITY
                           │
             ┌─────────────┴──────────────┐
             │                            │
          ACCOUNT                    ORGANIZATION
             │                            │
             │                       MEMBERSHIP
             │                            │
             │                     ┌──────┴──────┐
             │                     │             │
             │                   ROLES       DELEGATION
             │                     │
             │                PERMISSIONS
             │                     │
             └──────────────┬──────┘
                            │
                    AUTHORIZATION
                            │
                     POLICY ENGINE
                            │
                 ┌──────────┴──────────┐
                 │                     │
             CRITICALITY            CONTEXT
                 │                     │
             FRESHNESS             OWNERSHIP
                 │                     │
                 └──────────┬──────────┘
                            │
                    AUTHORIZATION
                       RUNTIME
                            │
              ┌─────────────┼─────────────┐
              ▼             ▼             ▼
           ALLOW          DENY       REQUIRE_MFA

Esto refleja mucho mejor lo que hemos definido en los ADR-045 → 059.

45. Veredicto de la auditoría de ESP-0001/01
Antes

🟡 Draft — demasiado conceptual y con decisiones pendientes antiguas.

Después de esta consolidación

🟢 Arquitectónicamente alineado, pero todavía:

🟡 Draft — pendiente de validación cruzada con ESP-0001/02–15.

No lo marcaría todavía como Approved, porque algunas decisiones de este Domain Model dependen de los documentos siguientes, especialmente:

API;
Persistence;
Events;
Testing;
Implementation.

Esto también respeta el principio de trazabilidad del Blueprint.

Una decisión importante antes de seguir

Yo no subiría todavía esta versión 1.1.0 a GitHub como definitiva.

La usaría como nuestra versión consolidada de trabajo y continuaría con:

ESP-0001/02 — Identity Architecture

Allí podremos hacer exactamente el mismo proceso:

ESP-0001/02 actual
       +
ADR-001 → 069
       ↓
Auditoría
       ↓
ESP-0001/02 v1.1.0
       ↓
alineado con ESP-0001/01

Y una vez terminemos los 15:

01 → 15
   ↓
Consistency Review
   ↓
ESP-0001 v1.1.0
   ↓
Approved
   ↓
IS-001 → IS-017
   ↓
Technical Tasks
   ↓
PROGRAMACIÓN

ESP-0001/01 queda, por tanto, conceptualmente actualizado y sin necesidad de crear un ADR-070 para cubrir lo que ya está decidido.
