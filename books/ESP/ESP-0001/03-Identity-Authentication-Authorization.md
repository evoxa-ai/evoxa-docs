ESP-0001 / 03 — Identity Authentication & Authorization

Document ID: ESP-0001/03
Module: Identity
Version: 1.1.0
Status: Draft — Consolidated
Parent: ESP-0001 — Identity
Previous Specifications:

ESP-0001/01 — Identity Domain Model
ESP-0001/02 — Identity Architecture

Architecture Parent: BP-0002 — Reference Architecture
Repository: evoxa-platform
Target Path: apps/api/app/domains/identity/

1. Purpose

Este documento define el comportamiento arquitectónico de las capacidades de:

Authentication;
Session Establishment;
Token Issuance;
Token Refresh;
Token Revocation;
MFA;
Authorization;
Tenant Authorization;
RBAC;
ABAC;
Ownership;
Delegation;
Security Policy;
Authorization Criticality;
Authorization Freshness;
Authorization Cache Eligibility;
Audit;
Security Events.

La distinción fundamental continúa siendo:

AUTHENTICATION
      │
      │ ¿Quién eres?
      ▼
   IDENTITY
      │
      ▼
AUTHORIZATION
      │
      │ ¿Qué puedes hacer?
      ▼
 SECURITY DECISION

Un usuario autenticado no implica que esté autorizado.

2. Fundamental Security Model

EVOXA debe separar claramente:

Authentication
    =
Who are you?

Authorization
    =
What are you allowed to do?

Criticality
    =
How strong must the security guarantees be?

Freshness
    =
How current must the authorization state be?

Decision
    =
What security action must occur?

Por tanto:

Authenticated ≠ Authorized

y:

Criticality ≠ Authorization Decision

Una Criticality C0 no significa ALLOW.

3. Authentication Architecture

El flujo principal será:

Client
   │
   ▼
Identity API
   │
   ▼
Authentication
   │
   ├── Validate Request
   ├── Identify Authentication Method
   ├── Validate Identity
   ├── Validate Credentials
   ├── Validate Account State
   ├── Evaluate Security Policy
   ├── Evaluate MFA
   ├── Establish Authentication Context
   ├── Create Session
   └── Issue Credentials
   │
   ▼
Authenticated Context
4. Authentication Methods

Identity debe utilizar una arquitectura extensible.

                    Authentication
                           │
          ┌────────────────┼────────────────┐
          ▼                ▼                ▼
       Local             OIDC              SAML
          │                │                │
          └────────────────┼────────────────┘
                           ▼
                    EVOXA Identity

Mecanismos contemplados:

Local credentials;
OAuth2.1;
OIDC;
SAML;
API Tokens;
Service Accounts;
futuras modalidades compatibles.

El proveedor concreto permanece abstraído del Domain Layer.

La especificación original contempla OAuth2.1/OIDC, JWT, API keys y service accounts como mecanismos de identidad.

5. Local Authentication

Para autenticación mediante credenciales:

Email
+
Password
      │
      ▼
Normalize / Validate
      │
      ▼
Find User
      │
      ▼
Validate Account State
      │
      ▼
Verify Password
      │
      ▼
Evaluate Security Policy
      │
      ▼
MFA Required?
      │
      ▼
Authentication Context

Las contraseñas deben almacenarse exclusivamente mediante hashing seguro.

La arquitectura establece:

Argon2id

como mecanismo de password hashing.

6. Authentication Enumeration Protection

Los errores de autenticación no deben revelar innecesariamente si:

Email exists

o:

Password is incorrect

El comportamiento externo debe evitar facilitar:

user enumeration;
credential stuffing;
account discovery.

Internamente sí debe generarse la telemetría de seguridad necesaria.

7. Account State

Antes de establecer una autenticación válida:

User
 │
 ▼
Account State
 │
 ├── PENDING
 ├── ACTIVE
 ├── SUSPENDED
 └── DISABLED

Las transiciones y restricciones deben seguir el modelo definido en ESP-0001/01 y ADR-033.

Como regla:

SUSPENDED → no normal authentication
DISABLED  → no authentication

Una condición de seguridad no debe ser ignorada para permitir el login.

8. Authentication Policy

Authentication debe consultar Security Policy antes de completar operaciones sensibles.

Conceptualmente:

Credentials Valid
       │
       ▼
Security Policy
       │
       ├── Continue
       ├── REQUIRE_MFA
       ├── REAUTHENTICATE
       ├── CHALLENGE
       └── DENY

La Policy Engine decide las condiciones de seguridad.

Authentication ejecuta el flujo correspondiente.

9. MFA Integration

MFA no debe ser tratado como un simple boolean.

El flujo es:

Authentication
      │
      ▼
Policy Evaluation
      │
      ▼
MFA Required?
      │
   ┌──┴──┐
   │     │
  NO    YES
   │     │
   │     ▼
   │  MFA Challenge
   │     │
   │     ▼
   │  Verification
   │     │
   └─────┤
         ▼
Authentication Context

MFAFactor y MFAChallenge son conceptos separados.

10. MFA Factor Model

Identity debe soportar una abstracción común:

MFAFactor

Los tipos contemplados arquitectónicamente incluyen:

TOTP;
WebAuthn;
Passkey;
SMS;
Email;
Push;
Hardware Key.

No todos deben necesariamente implementarse en la primera versión.

La selección concreta pertenece a la estrategia de MFA.

11. MFA Verification

Un challenge debe:

tener propósito;
tener TTL;
tener límite de intentos;
poder expirar;
ser consumible una sola vez;
ser resistente a replay;
ser verificado de forma concurrente segura.

Estados conceptuales:

PENDING
   │
   ├── VERIFIED
   ├── FAILED
   ├── EXPIRED
   └── REVOKED

Las decisiones detalladas están establecidas por ADR-023 y ADR-024.

12. Authentication Context

Una autenticación exitosa produce:

AuthenticationContext
│
├── actor
├── user_id
├── session_id
├── organization context
├── membership context
├── authentication method
├── authentication timestamp
├── MFA evidence
├── security state
└── authentication metadata

Este contexto será utilizado por Authorization.

El contexto no debe ser mutable arbitrariamente durante una evaluación.

13. Session Establishment

Después de completar Authentication:

Authentication
      │
      ▼
Create Session
      │
      ├── User
      ├── Organization
      ├── Membership
      ├── Authentication Method
      ├── MFA State
      ├── Created At
      ├── Expiration
      └── Security Metadata
      │
      ▼
Token Issuance

Las Sessions son stateful.

Una misma User puede disponer de:

User
 ├── Web Session
 ├── Mobile Session
 └── Other Session
14. Access Token

El Access Token es una credencial temporal.

Authorization: Bearer <access_token>

La arquitectura contempla JWT Bearer para APIs protegidas.

El token debe:

ser short-lived;
estar firmado criptográficamente;
utilizar firma asimétrica;
utilizar kid;
poder validarse mediante JWKS;
contener información mínima necesaria.
15. JWT Claims

El modelo conceptual incluye:

sub
iss
aud
iat
exp
jti
session_id

Podrá existir contexto adicional de seguridad cuando sea necesario.

No debe incluirse:

password;
refresh token;
secretos;
información sanitaria;
catálogo completo de permisos;
datos innecesarios del usuario.

El JWT no debe convertirse en una copia de la base de datos.

16. Access Token vs Session

Es importante distinguir:

Access Token
=
Credential

Session
=
Server-side authentication state

Por tanto:

JWT válido
≠
Session necesariamente activa
≠
Authorization necesariamente válida

Una Session revocada no debe generar un nuevo Refresh Token.

Para operaciones críticas, el runtime debe considerar el estado actual de seguridad.

17. Refresh Token

El Refresh Token permite obtener un nuevo Access Token sin repetir el proceso completo de autenticación.

Access Token
    │
    │ expires
    ▼
Refresh Token
    │
    ▼
Validate Session
    │
    ▼
Validate Refresh Token
    │
    ▼
Apply Security Policy
    │
    ▼
Rotate
    │
    ▼
New Access Token

El Refresh Token es stateful.

18. Refresh Token Security

La estrategia consolidada establece:

Opaque
+
Cryptographically Random
+
Stateful
+
Hashed/Protected
+
Rotated
+
Family-Based
+
Reuse Detection

La información completa de esta estrategia está definida en ADR-003 y ADR-025.

19. Refresh Token Family

Los tokens pertenecen a una familia:

Family A

Token 1
   │
   ▼
Token 2
   │
   ▼
Token 3
   │
   ▼
Token 4

Cada rotación debe invalidar el token anterior.

20. Refresh Token Reuse Detection

Si un token previamente consumido vuelve a utilizarse:

Used Token
    │
    ▼
Reuse Detected
    │
    ├── Security Event
    ├── Audit Event
    ├── Revoke Token Family
    └── Revoke Session

La respuesta debe ser segura incluso cuando el reuse pueda proceder de un escenario de concurrencia o retry.

Los detalles de grace/retry deben implementarse conforme a la estrategia definida para Refresh Tokens.

21. Logout

Logout debe revocar la Session correspondiente.

POST /api/v1/auth/logout
            │
            ▼
      Revoke Session
            │
       ┌────┴────┐
       ▼         ▼
     Audit    Security State

Las credenciales posteriores deben respetar el nuevo estado de Session.

22. Global Logout

Identity debe soportar conceptualmente:

Logout Current Session
Logout All Sessions
Revoke All Sessions

Ejemplo:

User
 ├── Session A → REVOKED
 ├── Session B → REVOKED
 └── Session C → REVOKED
23. Authorization Fundamental Model

Authorization responde:

¿Puede este actor realizar esta operación sobre este recurso, bajo este contexto de seguridad?

Ya no se modelará solamente como:

RBAC → ABAC → Ownership → ALLOW/DENY

La evaluación completa será:

Authorization Request
        │
        ▼
Request Validation
        │
        ▼
Authentication Validation
        │
        ▼
Tenant Guard
        │
        ▼
Context Construction
        │
        ▼
Policy Selection
        │
        ▼
Policy Evaluation
        │
        ▼
Mandatory Security Guards
        │
        ▼
RBAC
        │
        ▼
Ownership
        │
        ▼
ABAC
        │
        ▼
Delegation
        │
        ▼
Policy Composition
        │
        ▼
Conflict Resolution
        │
        ▼
Criticality
        │
        ▼
Freshness
        │
        ▼
Cache Eligibility
        │
        ▼
Authorization Decision

Esta es una de las modificaciones fundamentales respecto al ESP original.

24. Authorization Request

El request conceptual contiene:

AuthorizationRequest
│
├── actor
├── organization
├── membership
├── action
├── resource
├── request
├── session
├── authentication
├── MFA
├── delegation
├── ownership
├── risk
├── security state
├── environment
├── timestamp
└── correlation context

El tenant debe estar explícitamente presente.

25. Authentication Validation Before Authorization

Authorization no debe aceptar simplemente:

user_id = request.user_id

Debe derivar el actor del contexto autenticado.

Authenticated Identity
        │
        ▼
Authentication Context
        │
        ▼
Authorization Context

Un user_id enviado por el cliente nunca debe ser considerado autoridad suficiente.

26. Tenant Guard

Antes de autorizar:

Actor
 │
 ▼
Membership
 │
 ▼
Organization
 │
 ▼
Resource Tenant
 │
 ▼
Tenant Validation

Si existe:

Actor Tenant ≠ Resource Tenant

el resultado seguro es:

DENY

salvo que exista una capacidad cross-tenant explícita, autorizada y auditada.

27. Tenant Isolation

Tenant isolation es una regla de seguridad fundamental.

Tenant A
   │
   │ request
   ▼
Tenant B Resource
   │
   ▼
DENY

No se debe confiar en:

organization_id

enviado por el frontend.

28. RBAC

RBAC utiliza:

User
 ↓
Membership
 ↓
Role
 ↓
Permission

Una Membership puede tener múltiples roles:

Membership
 ├── Coach
 └── TeamManager

No existe jerarquía implícita de roles.

29. Permission Model

El modelo conceptual es:

resource.action

Ejemplos:

clients.read
clients.create
clients.update
clients.delete

training.read
training.create
training.update
training.delete

La autorización debe basarse en permisos, no en:

if role == "coach"

Los controllers no deben contener la lógica central de autorización.

30. ABAC

ABAC incorpora atributos dinámicos.

Actor
+
Organization
+
Membership
+
Resource
+
Resource State
+
Ownership
+
Environment
+
Risk
+
Security State

Ejemplo:

actor.role = Coach
AND
membership.organization = resource.organization
AND
resource.owner = actor
AND
resource.active = true
31. Ownership

Ownership constituye una restricción independiente.

Por ejemplo:

Permission:
training.update

+
Ownership:
resource.owner == actor

Tener el permiso no implica automáticamente tener ownership.

32. Delegation

Identity soporta delegación explícita.

Actor A
   │
   │ delegation
   ▼
Actor B

La delegación debe:

ser explícita;
ser scoped;
tener vigencia;
poder revocarse;
ser auditable;
respetar tenant;
respetar los privilegios del delegador.

Regla:

Delegated Capability
       ≤
Delegator Effective Capability

No puede existir privilege amplification.

33. Authorization Policy Engine

La autorización se centraliza mediante:

Authorization Policy Engine

Componentes conceptuales:

Policy Engine
│
├── Policy Matcher
├── Policy Evaluator
├── Attribute Registry
├── Resource Registry
├── Action Registry
├── Conflict Resolver
├── Effect Resolver
└── Obligation Resolver

La arquitectura se basa en un modelo declarativo, tipado y determinista.

No se permite:

SQL arbitrario;
código arbitrario;
llamadas de red durante evaluación;
acceso a secretos;
side effects.
34. Policy Evaluation

Una Policy puede producir:

MATCH
NO_MATCH
INDETERMINATE
ERROR

Regla de seguridad:

INDETERMINATE
    ↓
No ALLOW

ERROR
    ↓
No ALLOW

Unknown no puede convertirse silenciosamente en una autorización positiva.

35. Policy Effects

El runtime puede producir:

ALLOW
DENY
CHALLENGE
REAUTHENTICATE
REQUIRE_MFA
REVOKE

La decisión final debe incorporar todas las restricciones obligatorias.

36. Policy Precedence

La precedencia conceptual es:

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
Organization Policy
             ↓
Default

Los niveles inferiores no pueden debilitar los mínimos superiores.

37. Conflict Resolution

Cuando existen múltiples resultados:

DENY

debe prevalecer sobre:

ALLOW

cuando pertenecen al mismo nivel de precedencia.

Además:

Platform Minimum

no puede ser debilitado por una policy organizacional.

38. Authorization Criticality

Una vez evaluadas las condiciones relevantes, se determina:

Effective Criticality

mediante:

MAX(
    Base Criticality,
    Dynamic Risk,
    Context,
    Security State,
    Tenant,
    Delegation,
    MFA,
    Platform Minimum
)

Niveles:

C0 — Critical Security
C1 — High Security
C2 — Standard
C3 — Low Risk

La criticality es monotónica:

C3 → C2 → C1 → C0

Nunca:

C0 → C1

por una policy de menor autoridad.

39. Criticality vs Authorization

Debe mantenerse una separación estricta:

Criticality
     │
     ▼
Security Requirements
     │
     ▼
Freshness Requirements
     │
     ▼
Cache Eligibility

pero:

Criticality
   ≠
ALLOW

Una operación C0 puede terminar en:

DENY
40. Dynamic Risk

Risk puede elevar Criticality.

Ejemplo:

Normal Request
   ↓
C2

High Risk
   ↓
C1

Critical Risk
   ↓
C0

Risk Engine:

PROVIDES CONTEXT

pero no:

GRANTS AUTHORIZATION

Un riesgo desconocido nunca debe utilizarse para reducir seguridad.

41. Freshness

Después de determinar Criticality se establece el requisito de freshness.

F0 — Authoritative
F1 — Security Validated
F2 — Controlled Eventual
F3 — Restricted Degraded

Mapeo base:

C0 → F0
C1 → F1
C2 → F2-SHORT
C3 → F2-MEDIUM
42. Freshness Validation

Una decisión cacheada sólo puede utilizarse si cumple los requisitos correspondientes de:

Tenant
Actor
Membership
Security Epoch
Policy Version
Entity Version
Context Fingerprint
Generation
TTL
Invalidation State
Criticality
Freshness Class

TTL por sí solo nunca constituye garantía de seguridad.

43. Security Revocation Priority

Las revocaciones tienen prioridad sobre grants.

Por ejemplo:

Membership Revoked
      ↓
Security Epoch
      ↓
Existing Authorization Cache
      ↓
INVALID

aunque:

TTL > 0
44. Authorization Cache

El cache es una optimización:

PostgreSQL
    │
    ▼
Authoritative State
    │
    ▼
Redis L2
    │
    ▼
L1 Cache

Nunca:

Cache
=
Source of Truth
45. Cache Eligibility

La decisión de utilizar cache ocurre después de determinar:

Authorization context;
Criticality;
Freshness requirement.

Conceptualmente:

Authorization
     ↓
Criticality
     ↓
Freshness
     ↓
Cache Eligibility
     ↓
Cached / Fresh Evaluation

Una operación C0 no puede depender exclusivamente de un ALLOW cacheado.

46. Cache Invalidation

Los cambios de seguridad se propagan mediante:

State Change
      │
      ▼
PostgreSQL Transaction
      │
      ├── State
      └── Outbox Event
              │
              ▼
       Event Platform
              │
              ▼
      Invalidation Consumer
          │          │
          ▼          ▼
         L1         L2

Las categorías incluyen:

Policy;
Role;
Permission;
Membership;
Delegation;
Ownership;
Session;
MFA;
Security State.
47. Security Epoch

Security Epoch permite invalidar estados anteriores:

Epoch 10
   │
   │ security change
   ▼
Epoch 11

Una autorización cacheada con:

epoch = 10

no puede considerarse válida cuando:

current_epoch = 11
48. Authorization Decision

El resultado conceptual debe ser:

AuthorizationDecision
│
├── decision
├── reason_code
├── criticality
├── freshness
├── cacheability
├── policy_reference
├── policy_version
├── security_requirements
└── obligations

Decisiones:

ALLOW
DENY
CHALLENGE
REAUTHENTICATE
REQUIRE_MFA
REVOKE

Los detalles internos de reason_code no necesariamente deben exponerse al usuario final.

49. Authorization Runtime

El runtime central debe contener conceptualmente:

AuthorizationRuntime
│
├── RequestValidator
├── AuthenticationValidator
├── TenantGuard
├── ContextBuilder
├── PolicyMatcher
├── PolicyEvaluator
├── RBACEvaluator
├── OwnershipEvaluator
├── ABACEvaluator
├── DelegationEvaluator
├── ConflictResolver
├── CriticalityEngine
├── FreshnessEvaluator
├── CacheEligibilityEvaluator
├── EffectResolver
├── ObligationResolver
└── DecisionBuilder

Este runtime constituye el punto normativo de evaluación.

50. Business Domain Boundary

Identity no debe convertirse en propietario de todas las reglas de negocio.

Ejemplo:

Identity
   │
   ▼
training.update
   │
   ▼
Authorized
   │
   ▼
Training Domain
   │
   ▼
Specific Resource Rule

Identity puede determinar:

El actor posee el permiso y pertenece al tenant.

Training puede determinar:

Este programa específico está en un estado que permite modificación.

51. Authorization Middleware

El middleware debe ser delgado:

Request
   │
   ▼
Authentication Middleware
   │
   ▼
Authorization Middleware
   │
   ▼
Application

Pero:

Middleware

no debe contener todo el Policy Engine.

Debe delegar al runtime central.

52. API Authentication

La API base será:

/api/v1

Ejemplos:

POST /api/v1/auth/login
POST /api/v1/auth/logout
POST /api/v1/auth/refresh

Los contratos completos pertenecen a:

ESP-0001/10 — Identity API Contracts

y no deben duplicarse aquí.

53. API Keys

Las API Keys son credenciales para integraciones.

Conceptualmente:

API Key
├── Owner
├── Scope
├── Status
├── Expiration
└── Audit

Deben poder:

revocarse;
expirar;
auditarse;
limitarse por scope.

API Token no debe confundirse con User Session.

54. Service Accounts

Las identidades no humanas deben utilizar:

Service Account

Ejemplo:

analytics-service
       │
       ▼
analytics.events.read

No debe otorgarse automáticamente:

admin.*

Las service accounts deben seguir:

least privilege;
tenant isolation;
authorization;
audit;
lifecycle management.
55. OAuth2.1 / OIDC

Identity debe soportar conceptualmente:

Authorization Code
+
PKCE

para clientes apropiados.

OIDC proporciona autenticación federada.

OAuth proporciona autorización delegada.

Los scopes OAuth no sustituyen:

RBAC
+
ABAC
+
Tenant Isolation
+
Policy Engine

dentro de EVOXA.

56. Federation

El flujo federado será:

External IdP
     │
     ▼
Federation Adapter
     │
     ▼
External Identity
     │
     ▼
EVOXA User
     │
     ▼
Membership
     │
     ▼
EVOXA Authorization

No debe realizarse linking únicamente por email.

57. Security Events

Authentication y Authorization deben producir eventos de seguridad cuando corresponda.

Ejemplos:

AuthenticationFailed
MFAFailed
AuthorizationDenied
SuspiciousAuthentication
RefreshTokenReuseDetected
SessionRevoked
AccountSuspended

Los eventos deben seguir la arquitectura de Event Platform.

58. Audit

Operaciones sensibles deben ser auditables.

Ejemplos:

login;
authentication failure;
authorization denial;
role changes;
permission changes;
membership changes;
MFA changes;
session revocation;
refresh token reuse;
policy activation;
policy security change;
delegation changes;
account security state.

Los logs técnicos no sustituyen los audit events.

59. Rate Limiting & Abuse Protection

Authentication debe estar protegido contra:

brute force;
credential stuffing;
password spraying;
MFA brute force;
recovery abuse;
refresh abuse.

Las protecciones pueden utilizar dimensiones:

IP
User
Organization
Endpoint
Session
Device
Risk

Redis puede ayudar a implementar rate limiting, pero una caída de Redis no puede convertir Authentication en permisiva.

60. Fail-Closed

Regla fundamental:

Unknown
   ↓
No unsafe ALLOW
Authorization Error
   ↓
No unsafe ALLOW
Tenant Uncertainty
   ↓
DENY
Critical Freshness Unavailable
   ↓
Fresh Evaluation / Fail Closed

Nunca:

Infrastructure Failure
        ↓
ALLOW Everything
61. Security State Changes

Los siguientes eventos tienen prioridad de seguridad:

Account Suspension
Account Disable
Membership Revocation
Role Removal
Permission Revocation
Session Revocation
Refresh Family Revocation
MFA Revocation
Delegation Revocation
Policy Tightening

Deben provocar la invalidación/fresh validation correspondiente.

62. AI Authorization

AI Agents son actores de Identity.

AI Agent
   │
   ▼
Authentication
   │
   ▼
Authorization
   │
   ▼
Policy
   │
   ▼
Criticality
   │
   ▼
Freshness
   │
   ▼
Decision

No existe:

AI bypass

AI no puede:

saltarse autorización;
reducir Criticality;
desactivar MFA;
ampliar privilegios;
activar una security policy directamente;
aprobar una security regression;
manipular cache;
utilizar secretos sin autorización.
63. Test Runtime

El runtime utilizado para validar políticas debe estar separado conceptualmente del Production Runtime.

Production
Authorization Runtime
        │
        │ same normative contracts
        ▼
Test Runtime

El Test Runtime puede ejecutar:

Golden Scenarios;
Security Invariants;
Generated Scenarios;
Property-Based Tests;
Boundary Tests;
Mutation Tests;
Differential Tests;
Chaos;
Recovery.

Esto deriva de ADR-062 → ADR-067.

64. Reproducibility

Una evaluación debe poder reproducirse utilizando:

Policy Version
Runtime Version
Schema Version
Scenario Version
Invariant Version
Generator Version
Seed
Input Context
Evaluation Time

Esto permite saber exactamente por qué una decisión fue producida.

65. Security Invariants

La implementación debe preservar como mínimo:

Tenant Isolation
No Privilege Amplification
Unknown ≠ Safe Allow
Error ≠ Safe Allow
Security Monotonicity
Platform Minimum Immutability
Revocation Priority
Security Epoch Supersedes TTL
Critical ALLOW Requires Freshness
No Version Regression
MFA Evidence Freshness
AI Cannot Reduce Security
Cache Is Not Source of Truth
66. Performance Requirements

Objetivos:

Authentication P95 < 500 ms
Authorization P95 < 50 ms
General API P95 < 300 ms

La optimización nunca puede sacrificar:

tenant isolation;
MFA;
revocation;
freshness;
policy enforcement;
auditability;
fail-closed.
67. Observability

Authentication y Authorization deben producir:

Logs
Metrics
Traces
Audit Events
Security Events

Debe ser posible correlacionar una operación mediante:

correlation_id
trace_id
causation_id

Las decisiones de Policy deben poder observarse sin revelar secretos.

68. Security Data Protection

Nunca deben aparecer secretos en:

Logs
Events
Traces
Audit Payloads
API Responses
AI Context

Especialmente:

Passwords
Refresh Tokens
API Secrets
MFA Secrets
Recovery Codes
Private Keys
69. Non-Negotiable Rules
AUTH-001

Authentication y Authorization son capacidades distintas.

AUTH-002

JWT válido no implica autorización.

AUTH-003

Tenant context debe derivarse de identidad/membership.

AUTH-004

Cross-tenant access está denegado por defecto.

AUTH-005

Authorization utiliza el runtime central.

AUTH-006

Unknown/Error nunca puede producir unsafe ALLOW.

AUTH-007

Criticality no otorga permisos.

AUTH-008

C0 no puede depender de un ALLOW cacheado sin autoridad/freshness requerida.

AUTH-009

Security Epoch supera TTL.

AUTH-010

Revocations tienen prioridad sobre grants.

AUTH-011

Refresh Tokens son stateful y protegidos.

AUTH-012

Refresh Token reuse debe generar respuesta de seguridad.

AUTH-013

MFA no puede ser bypass mediante Recovery.

AUTH-014

Delegation no puede ampliar privilegios.

AUTH-015

AI no puede reducir seguridad.

AUTH-016

Redis no es fuente de verdad.

AUTH-017

No existen accesos directos cross-module a tablas.

AUTH-018

Los eventos son facts, no commands.

AUTH-019

Las operaciones críticas deben ser auditables.

AUTH-020

Los resultados de seguridad deben ser reproducibles.

70. Traceability
Área	ADR
Identity Architecture	001
Authentication	002
Sessions	003
Authorization	005
Persistence	006
Events	007
Cryptography	008
Token Storage	009
Testing	010
Runtime	011
MFA	023–024
Refresh Tokens	025
API	026
Secrets/KMS	027
Observability	028
DR	029–030
HA	031
Rate Limiting	032
Account Lifecycle	033
Password Security	034
Recovery	035
Email Verification	036
API Tokens	037
S2S	038
OAuth/OIDC	039
Federation	040
Provisioning	041
Attribute Ownership	042
Multi-Organization	043
Delegation	044
Policy Engine	045–049
Authorization Cache	050–057
Criticality	058–059
Criticality Testing	062–063
Test Data	064
Schema Evolution	065
Schema Registry	066
Runtime Contracts	067
71. Relationship with Other Specifications

Este documento establece el comportamiento arquitectónico integrado.

Las implementaciones detalladas deben estar en:

ESP-0001/05
Roles & Permissions

ESP-0001/06
Sessions & Token Management

ESP-0001/07
MFA

ESP-0001/08
Security Policies

ESP-0001/10
API Contracts

ESP-0001/11
Persistence

ESP-0001/12
Events

ESP-0001/13
Testing

ESP-0001/14
Implementation

No debemos duplicar esas especificaciones aquí.

72. Remaining Implementation Decisions

Después de los ADR-001 → 069, las cuestiones pendientes ya son principalmente de implementación:

catálogo definitivo de permisos;
catálogo de resources/actions;
esquema exacto de Authentication Context;
esquema machine-readable del Policy Engine;
DSL/AST definitivo;
parámetros exactos de Argon2id;
TTL exactos de Sessions/Tokens;
configuración definitiva de MFA providers;
API contracts;
PostgreSQL schema;
Event Catalog;
implementación concreta del cache;
infraestructura OIDC/SAML;
estructura definitiva de paquetes;
deployment topology.

No necesitamos crear otro ADR solamente para resolver cuestiones que ya están decididas por los ADR existentes.

73. Implementation Readiness

Con esta consolidación, el flujo completo queda:

┌───────────────────────────┐
│      AUTHENTICATION       │
│ Who are you?              │
└─────────────┬─────────────┘
              │
              ▼
┌───────────────────────────┐
│   AUTHENTICATION CONTEXT  │
└─────────────┬─────────────┘
              │
              ▼
┌───────────────────────────┐
│       TENANT GUARD        │
└─────────────┬─────────────┘
              │
              ▼
┌───────────────────────────┐
│     POLICY EVALUATION     │
└─────────────┬─────────────┘
              │
              ▼
┌───────────────────────────┐
│ RBAC + OWNERSHIP + ABAC   │
│ + DELEGATION              │
└─────────────┬─────────────┘
              │
              ▼
┌───────────────────────────┐
│       CRITICALITY         │
└─────────────┬─────────────┘
              │
              ▼
┌───────────────────────────┐
│        FRESHNESS          │
└─────────────┬─────────────┘
              │
              ▼
┌───────────────────────────┐
│    CACHE ELIGIBILITY      │
└─────────────┬─────────────┘
              │
              ▼
┌───────────────────────────┐
│  AUTHORIZATION DECISION   │
│ ALLOW / DENY / CHALLENGE  │
│ REAUTH / MFA / REVOKE     │
└───────────────────────────┘
Veredicto

ESP-0001/03 original: 🔴 requiere actualización profunda.

ESP-0001/03 v1.1.0: 🟢 alineado arquitectónicamente con los ADR-001 → 069, manteniéndose en Draft hasta completar la consolidación de los ESP posteriores.

Y aquí hay algo importante: este ESP ya deja preparada la base para empezar a convertir Identity en una implementación real, porque ahora Authentication y Authorization no son simplemente CRUD + JWT + roles; tenemos definido el modelo completo de seguridad que después tendremos que traducir a ESP → Implementation Stories → Technical Tasks → código.

El siguiente sería ESP-0001/04 — Identity User & Organization Management, donde debemos hacer lo mismo pero concentrándonos en User + Organization + Membership + Multi-Organization + lifecycle + tenant boundary.
