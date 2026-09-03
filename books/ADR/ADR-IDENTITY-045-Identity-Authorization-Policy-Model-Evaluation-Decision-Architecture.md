ADR-IDENTITY-045 — Identity Authorization Policy Model, Evaluation & Decision Architecture

Status: Proposed
Version: 1.0
Date: 2026-09-03
Domain: Identity / Authorization
Priority: Critical
Type: Architecture / Security / Policy Engine

1. Context

Los ADR anteriores establecieron progresivamente el modelo de autorización de EVOXA:

ADR-IDENTITY-005 — RBAC + ABAC + Tenant Isolation + Resource Ownership + Security Policy Engine.
ADR-IDENTITY-011 — Runtime & Deployment.
ADR-IDENTITY-026 — API Pagination, Filtering & Idempotency.
ADR-IDENTITY-028 — Observability & Operational Security.
ADR-IDENTITY-040 — Federation.
ADR-IDENTITY-041 — Provisioning & Deprovisioning.
ADR-IDENTITY-042 — Source of Truth & Conflict Resolution.
ADR-IDENTITY-043 — Multi-Organization & Cross-Tenant Access.
ADR-IDENTITY-044 — Membership Roles, Delegation & Fine-Grained Access.

Con estos ADR se ha definido qué información participa en una autorización.

Falta definir ahora cómo se procesa esa información para producir una decisión consistente.

El sistema debe poder responder preguntas como:

¿Puede este usuario realizar esta acción?
¿En qué Organization?
¿Sobre qué recurso?
¿Con qué Role?
¿Tiene ownership?
¿Existe una Delegation?
¿La política exige MFA?
¿Existe riesgo?
¿La cuenta está activa?
¿La Membership está activa?
¿La Organization está activa?
¿Existe alguna regla que niegue explícitamente la operación?

Por lo tanto, EVOXA necesita un modelo centralizado de evaluación.

2. Problem Statement

Sin un modelo formal, diferentes servicios podrían implementar reglas de autorización de manera distinta:

Service A → RBAC
Service B → RBAC + Ownership
Service C → Role check
Service D → custom if statements
Service E → external provider claim

Esto produciría:

inconsistencias;
bypasses;
privilege escalation;
cross-tenant access;
dificultad para auditar;
dificultad para explicar decisiones;
comportamiento diferente entre Web, Mobile, API y AI Agents.

La autorización debe convertirse en una capacidad central, determinista, auditable y reutilizable.

3. Decision

EVOXA adoptará un:

Centralized Identity Authorization Policy Engine

La evaluación conceptual será:

Authorization Request
        ↓
Context Resolution
        ↓
Identity State
        ↓
Tenant Isolation
        ↓
Membership
        ↓
RBAC
        ↓
Delegation
        ↓
Ownership
        ↓
ABAC
        ↓
Security Policies
        ↓
Risk / Assurance
        ↓
Final Decision

La implementación podrá optimizar el orden internamente, pero deberá preservar la semántica de seguridad definida por este ADR.

4. Authorization as a Decision

La autorización será modelada como una función:

Decision = Authorize(Context, PolicySet)

Donde:

Context
+
PolicySet
=
Decision

La decisión será explícita y no simplemente un booleano.

5. Decision Catalog

Se utilizarán las siguientes decisiones conceptuales:

ALLOW
DENY
CHALLENGE
REAUTHENTICATE
REQUIRE_MFA
REVOKE
ALLOW

La operación puede ejecutarse.

DENY

La operación no está autorizada.

CHALLENGE

Debe completarse un desafío adicional.

REAUTHENTICATE

La sesión requiere autenticación adicional.

REQUIRE_MFA

Debe completarse MFA/step-up.

REVOKE

Debe revocarse una capacidad/sesión/token según el contexto.

6. Decision Is Not Permission

Una Permission no equivale automáticamente a ALLOW.

Ejemplo:

Permission:
training_plans.update

puede terminar en:

DENY

si:

Membership = SUSPENDED

o:

Organization = SUSPENDED

o:

Ownership = false

o:

Policy = DENY
7. Authorization Context

El contexto de autorización será conceptualmente:

AuthorizationContext
│
├── Actor
│   ├── user_id / service_id / agent_id
│   ├── account_state
│   └── identity_assurance
│
├── Organization
│   ├── organization_id
│   └── organization_state
│
├── Membership
│   ├── membership_id
│   ├── status
│   └── roles
│
├── Authentication
│   ├── method
│   ├── session
│   └── MFA / step-up
│
├── Request
│   ├── operation
│   ├── endpoint
│   └── client
│
├── Resource
│   ├── resource_type
│   ├── resource_id
│   ├── organization_id
│   └── ownership
│
├── Delegation
│
├── Environment
│   ├── time
│   ├── network
│   ├── device
│   └── risk
│
└── Policy Context
8. Actor Types

El Authorization Engine deberá soportar diferentes tipos de principals:

USER
SERVICE_ACCOUNT
AI_AGENT
SYSTEM

No se debe asumir que todos los principals tienen Membership.

Por ejemplo:

AI Agent

puede utilizar una identidad de servicio y un Organization Scope explícito.

9. Resource Context

Los recursos tenant-scoped deberán proporcionar:

resource_type
resource_id
organization_id
owner_id
team_id
classification
state

No todos los recursos requieren todos los atributos.

La información necesaria dependerá del dominio.

10. Tenant Isolation as a Mandatory Check

Para recursos organizacionales:

Actor Organization
        =
Resource Organization

debe cumplirse, salvo que exista una autorización explícita de plataforma/delegación.

Si no se puede determinar el tenant:

DENY

No se permitirá inferirlo de forma insegura.

11. Membership Validation

Para una operación Organization-scoped:

Membership exists
AND
Membership.status == ACTIVE

debe cumplirse.

Membership:

INVITED
SUSPENDED
REMOVED

no concede acceso operativo normal.

12. User Lifecycle Validation

También deberá evaluarse el estado del User.

Conceptualmente:

User ACTIVE
+
Membership ACTIVE
+
Organization ACTIVE

son condiciones fundamentales para el acceso organizacional normal.

Un User SUSPENDED o DISABLED no deberá recibir ALLOW por una Membership activa.

13. Organization Lifecycle Validation

De igual forma:

Organization ACTIVE

es una condición de acceso operativo normal.

Si la Organization está:

SUSPENDED

la decisión será normalmente:

DENY

según las reglas específicas de la Organization y Platform.

14. RBAC Evaluation

El motor deberá determinar:

Roles
    ↓
Permissions

Ejemplo:

Coach
 ↓
training_plans.read

Si la Permission requerida no está presente:

DENY

salvo que exista una forma explícita y autorizada de acceso mediante Delegation u otra política.

15. Multiple Roles

Si una Membership posee:

Coach
Team Manager

las permissions efectivas serán calculadas a partir de los roles autorizados.

Conceptualmente:

Effective Permissions
=
Role A permissions
+
Role B permissions

pero:

La combinación de roles nunca puede superar una política de seguridad de mayor precedencia.

16. Delegation Evaluation

Si existe una delegación:

Delegator
 ↓
Delegate
 ↓
Permission
 ↓
Resource

el motor deberá validar:

Delegation existe.
Está ACTIVE.
No expiró.
No fue revocada.
Pertenece al tenant correcto.
El delegante tenía la capacidad delegada.
El alcance coincide con la operación.
La policy permite la delegación.
17. Ownership Evaluation

Cuando la policy lo requiera:

resource.owner_id == actor.id

o alguna otra relación válida deberá cumplirse.

Ownership podrá ser:

User;
Team;
Organization;
Shared;
Domain-specific.
18. ABAC Evaluation

ABAC permitirá utilizar atributos del contexto.

Ejemplo:

actor.department == resource.department

o:

actor.team_id == resource.team_id

o:

risk <= MEDIUM

La combinación concreta será definida mediante Security Policies.

19. Security Policy Evaluation

El Security Policy Engine será el componente encargado de reglas contextuales complejas.

Ejemplo:

IF
    permission == "memberships.manage"
AND
    actor.role == "OrganizationAdmin"
AND
    target_role != "PlatformAdmin"
THEN
    ALLOW

Otro ejemplo:

IF
    operation == "sensitive_export"
THEN
    REQUIRE_MFA
20. Policy Precedence

Se conserva la precedencia definida anteriormente:

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

Las políticas superiores no pueden ser debilitadas por políticas inferiores.

21. DENY Precedence

Cuando existan decisiones conflictivas:

ALLOW
+
DENY

la decisión final será:

DENY

salvo que una política superior explícita defina una excepción válida.

No se permitirá que una regla genérica de ALLOW neutralice una prohibición crítica.

22. Decision Pipeline

El flujo conceptual será:

                Authorization Request
                         │
                         ▼
                 Context Resolution
                         │
                         ▼
                 Identity Validation
                         │
                         ▼
                  Tenant Validation
                         │
                         ▼
                 Membership Validation
                         │
                         ▼
                    RBAC Check
                         │
                         ▼
                  Delegation Check
                         │
                         ▼
                 Ownership / ABAC
                         │
                         ▼
                 Security Policies
                         │
                         ▼
               MFA / Assurance / Risk
                         │
                         ▼
                  Final Decision

El pipeline puede implementar short-circuit cuando exista una condición determinante.

23. Short-Circuit Rules

Ejemplo:

User = DISABLED

puede terminar inmediatamente en:

DENY

No es necesario evaluar permisos de recurso si el actor está globalmente deshabilitado.

Sin embargo, la implementación debe mantener las garantías de auditabilidad y observability.

24. Unknown / Indeterminate State

El motor debe diferenciar:

ALLOW
DENY
INDETERMINATE

internamente, aunque la API exponga únicamente decisiones permitidas por el contrato.

Por ejemplo:

Policy unavailable

no debe transformarse en:

ALLOW

La resolución normal deberá ser:

FAIL CLOSED
25. External Dependency Failure

Una autorización crítica no debe depender innecesariamente de proveedores externos.

Si una dependencia externa resulta imprescindible y falla:

External dependency failure
        ↓
Unable to establish authorization
        ↓
DENY / safe challenge

Nunca:

External dependency failure
        ↓
ALLOW
26. Authentication Assurance

Authorization podrá depender del nivel de assurance de autenticación.

Ejemplo:

Password only

puede permitir:

training_plans.read

mientras:

Sensitive operation

requiere:

MFA / Step-Up
27. Risk Evaluation

El contexto podrá incorporar un risk score o risk level.

Ejemplo conceptual:

LOW
MEDIUM
HIGH
CRITICAL

Las policies podrán definir:

LOW       → ALLOW
MEDIUM    → ALLOW / MFA
HIGH      → CHALLENGE
CRITICAL  → DENY / REVOKE

Los valores exactos no se fijan en este ADR.

28. Policy Versioning

Toda policy relevante deberá poder identificarse mediante:

policy_id
policy_version

Esto permite responder:

¿Qué versión de la política produjo esta decisión?

Ejemplo:

Policy:
organization-security

Version:
12

Decision:
DENY
29. Policy Lifecycle

Las policies podrán seguir un lifecycle conceptual:

DRAFT
 ↓
VALIDATING
 ↓
ACTIVE
 ↓
SUPERSEDED
 ↓
RETIRED

No se deberá ejecutar una policy DRAFT en producción.

30. Policy Scope

Las policies podrán tener scope:

PLATFORM
ORGANIZATION
RESOURCE

Una Organization no podrá definir policies que contradigan los controles mínimos de Platform Security.

31. Policy Composition

Las policies pueden coexistir.

Ejemplo:

Platform Policy
+
Organization Policy
+
Resource Policy

El motor debe combinarlas respetando:

scope;
precedence;
deny rules;
conditions;
version.

No se utilizará una política de Organization para reemplazar completamente una política de Platform.

32. Explainability

El motor deberá poder producir una explicación estructurada para:

debugging;
audit;
security investigation;
support autorizado.

Ejemplo conceptual:

{
  "decision": "DENY",
  "reason_code": "MEMBERSHIP_SUSPENDED",
  "policy_id": "platform-access",
  "policy_version": 4
}

La explicación pública para el usuario podrá ser mucho más limitada.

33. Public vs Internal Explanation

No se deben exponer al usuario detalles internos que faciliten:

enumeration;
policy discovery;
privilege probing;
security bypass.

Por lo tanto:

Internal
MEMBERSHIP_SUSPENDED
policy_42
rule_17
risk_score=83
External
You are not authorized to perform this operation.

según la API policy.

34. Authorization Decision Object

Internamente se podrá utilizar una estructura conceptual:

AuthorizationDecision
├── decision
├── reason_code
├── policy_id
├── policy_version
├── organization_id
├── membership_id
├── resource
├── obligations
├── challenge
└── metadata

No todos los campos deben exponerse externamente.

35. Obligations

Una decisión ALLOW puede tener obligaciones.

Ejemplo:

ALLOW
+
obligation:
log_sensitive_access

o:

ALLOW
+
obligation:
mask_sensitive_fields

Esto permite separar:

de:

Las obligaciones definitivas se definirán posteriormente.

36. Step-Up as Obligation / Decision

Una operación puede producir:

REQUIRE_MFA

Después de completar MFA:

Re-evaluate authorization

No se debe asumir que haber completado MFA automáticamente concede la Permission.

MFA prueba assurance; no crea autorización.

37. Reauthorization

Si una operación requiere autenticación reciente:

REAUTHENTICATE

el cliente deberá realizar el flujo correspondiente.

Posteriormente:

Authorization Re-evaluation
38. Revocation Decision

REVOKE se utilizará para escenarios donde la evaluación determine que una capacidad existente debe retirarse.

Ejemplo conceptual:

Refresh Token Reuse
        ↓
Security Policy
        ↓
REVOKE
        ↓
Session / Token Family

Esta decisión no representa un permiso de negocio, sino una acción de seguridad.

39. Authorization Cache

Las decisiones o componentes de la decisión podrán cachearse cuando sea seguro.

Las claves deberán incluir suficiente contexto:

actor
organization
membership
resource
operation
policy_version

No se permitirá reutilizar:

Org A decision

para:

Org B
40. Cache Invalidation

Cambios que pueden invalidar decisiones:

Membership status
Role assignment
Role removal
Permission change
Policy version
Organization state
User state
Delegation
MFA assurance
Risk state

La invalidación deberá ser consistente con la criticidad de la autorización.

41. Authorization Consistency

Para operaciones críticas, el sistema debe priorizar seguridad sobre rendimiento.

Ejemplo:

Role removed
        ↓
Critical operation

No debe concederse acceso únicamente porque una cache antigua todavía contiene el Role.

Las estrategias exactas de consistency/cache quedan pendientes.

42. API Integration

La API deberá delegar autorización al componente central.

Conceptualmente:

Controller
   ↓
Application Service
   ↓
Authorization Service
   ↓
Policy Engine
   ↓
Decision
   ↓
Domain Operation

No:

Controller
   ↓
if user.role == "admin"

como mecanismo aislado.

43. Domain vs Authorization

Authorization no debe convertirse en lógica de negocio.

Por ejemplo:

"Coach may update training plan"

es Authorization.

Mientras:

"Training plan cannot be finalized after competition begins"

es Business Domain Rule.

Ambas pueden participar en el resultado final, pero deben mantenerse conceptualmente separadas.

44. Authorization vs Business Validation

Un request puede pasar Authorization:

ALLOW

y aun así fallar una regla de negocio:

BUSINESS_RULE_VIOLATION

Por lo tanto:

Authorized
≠
Business Operation Valid
45. Policy Enforcement Point

Las aplicaciones deberán utilizar puntos de enforcement.

Conceptualmente:

Request
 ↓
Policy Enforcement Point
 ↓
Authorization Decision
 ↓
Allow / Deny / Challenge

El PEP puede estar en:

API;
Application Service;
gateway;
resource service;

pero la decisión debe provenir del modelo de autorización centralizado.

46. Policy Decision Point

El Security Policy Engine actuará conceptualmente como:

Policy Decision Point (PDP)

mientras los servicios que aplican la decisión funcionan como:

Policy Enforcement Points (PEP)

Modelo:

        ┌─────────────┐
Request │     PEP     │
───────▶│ Enforcement │
        └──────┬──────┘
               │
               ▼
        ┌─────────────┐
        │     PDP     │
        │ Policy      │
        │ Engine      │
        └──────┬──────┘
               │
               ▼
           Decision
               │
               ▼
             PEP
47. Policy Administration Point

La administración de policies deberá estar separada conceptualmente.

Policy Administration
        ↓
Policy Repository
        ↓
Policy Decision Point
        ↓
Policy Enforcement Point

No todos estos componentes necesitan ser servicios independientes en la primera implementación.

La separación es arquitectónica.

48. Policy Repository

Las policies deberán tener:

ID;
version;
scope;
status;
conditions;
priority/precedence;
effective dates;
audit metadata.

La persistencia exacta será definida en los ADR de persistence.

49. Policy Change Governance

Cambios de policies críticas deberán ser:

autorizados;
auditados;
versionados;
validados;
testeados;
reversibles cuando sea posible.

No se debe desplegar una policy crítica directamente sin validación.

50. Policy Simulation

El sistema debería soportar, especialmente para administración:

Policy Simulation

Ejemplo:

"What happens if this policy is activated?"

Resultado:

Current:
ALLOW

Proposed:
REQUIRE_MFA

La simulación no debe modificar el estado real.

51. Policy Testing

Las policies deberán tener casos de prueba explícitos.

Ejemplo:

Given:
Coach / Org A

When:
update own training plan

Then:
ALLOW

y:

Given:
Coach / Org A

When:
update Team B resource

Then:
DENY
52. AI-Assisted Policy Analysis

AI podrá:

explicar una policy;
detectar conflictos;
sugerir simplificaciones;
encontrar reglas redundantes;
analizar decisiones históricas;
simular escenarios.

AI no podrá publicar automáticamente una policy crítica sin pasar por governance y autorización.

53. AI Authorization

AI Agents deberán pasar por el mismo modelo conceptual:

Agent Identity
+
Organization
+
Task Scope
+
Permissions
+
Delegation
+
Policy

No existirán:

AI = trusted by default

ni:

AI = platform administrator

por defecto.

54. Observability

Cada decisión crítica podrá producir telemetry con:

decision
reason_code
policy_id
policy_version
organization_id
actor_type
operation
resource_type
correlation_id
trace_id

Los datos sensibles deberán minimizarse.

No se deben registrar:

passwords;
tokens;
MFA secrets;
recovery codes;
private keys.
55. Decision Logging

No todas las decisiones de autorización necesitan convertirse en Audit Events persistentes.

Se distinguirá:

Technical Telemetry

Alta frecuencia.

Security Events

Eventos relevantes de seguridad.

Audit Events

Acciones que deben quedar como evidencia.

Esto evita generar una carga excesiva sobre PostgreSQL.

56. Performance

La autorización es una operación crítica de alta frecuencia.

Se mantendrán los objetivos de referencia del Blueprint:

API P95 < 300 ms
Authentication P95 < 500 ms

El Authorization Engine deberá diseñarse para no introducir una latencia desproporcionada.

Preferencias:

evaluación local cuando sea posible;
caching seguro;
policy precompilation cuando corresponda;
consultas optimizadas;
evitar llamadas externas innecesarias;
invalidación eficiente.
57. Availability

La indisponibilidad del Authorization Engine no debe producir un bypass.

Para operaciones críticas:

PDP unavailable
      ↓
DENY / FAIL CLOSED

La arquitectura podrá utilizar redundancia y caching para reducir este riesgo.

58. High Availability

El PDP deberá poder ejecutarse en múltiples instancias cuando sea un componente separado.

        Load Balancer
             │
       ┌─────┼─────┐
       ▼     ▼     ▼
     PDP-1 PDP-2 PDP-3

Debe ser preferentemente stateless respecto de decisiones temporales, utilizando fuentes externas para estado persistente.

59. Security of Policy Engine

El Policy Engine es un componente security-critical.

Por ello:

acceso administrativo restringido;
policies protegidas;
cambios auditados;
versionado;
validación;
secrets externos;
observabilidad;
fail-closed.

Comprometer el Policy Engine podría equivaler a comprometer Authorization.

60. Disaster Recovery

Debe recuperarse:

policy definitions;
policy versions;
active policy state;
role/permission mappings;
delegation state;
authorization-related configuration.

Después de recovery se deberá verificar:

ALLOW scenarios
DENY scenarios
MFA scenarios
Tenant isolation
Delegation
Role changes
Policy versions
61. Security Testing

Se deberán probar:

Authorization bypass.
Privilege escalation.
Cross-tenant access.
Role confusion.
Permission confusion.
Delegation abuse.
Policy conflict.
Policy version mismatch.
Cache poisoning.
Cache cross-tenant leakage.
Fail-open.
Stale authorization.
Concurrent role changes.
AI Agent escalation.
62. Formal Security Properties

El sistema deberá preservar propiedades como:

Tenant Isolation
Actor not authorized for Org B
        ⇒
Actor cannot access Org B resources
Least Privilege
Effective Permission
≤
Authorized Capability
Delegation Safety
Delegated Capability
≤
Delegator Capability
Fail Closed
Authorization uncertainty
        ⇒
DENY
No Implicit Cross-Tenant Access
Membership(Org A)
        ≠
Membership(Org B)
63. Consequences
Positivas
Autorización centralizada.
Comportamiento consistente.
RBAC + ABAC.
Ownership.
Delegation.
Tenant isolation.
MFA/step-up integrado.
Policy versioning.
Explainability.
AI compatibility.
Auditoría.
Mejor capacidad de testing.
Menor duplicación de reglas.
Negativas
Mayor complejidad arquitectónica.
Authorization se convierte en componente crítico.
Requiere governance de policies.
Cache invalidation compleja.
Necesidad de pruebas exhaustivas.
Cambios de policy requieren lifecycle y control.
64. Alternatives Considered
A. Autorización dentro de cada servicio

Rejected.

Produce reglas inconsistentes y aumenta el riesgo de bypass.

B. RBAC solamente

Rejected.

No cubre ownership, risk, delegation ni context.

C. Policy Engine completamente externo desde el inicio

Deferred.

Puede ser útil posteriormente, pero EVOXA no debe imponer desde ahora un proveedor o producto específico.

D. Autorización basada solamente en JWT

Rejected.

No refleja necesariamente el estado actual de User, Membership, Role, Policy o Resource.

E. Fail-Open para mejorar disponibilidad

Rejected.

La seguridad tiene prioridad sobre disponibilidad en decisiones críticas.

65. Architectural Invariants
I1:
Authorization is centralized conceptually.

I2:
Membership is required for normal organization-scoped access.

I3:
Tenant isolation is mandatory.

I4:
RBAC is combined with ABAC where required.

I5:
Ownership may constrain access.

I6:
Delegation cannot amplify authority.

I7:
Cross-tenant access is denied by default.

I8:
DENY has precedence over conflicting ALLOW.

I9:
Security Policy has higher precedence than ordinary role grants.

I10:
Unknown critical authorization state fails closed.

I11:
JWT validity does not equal authorization validity.

I12:
MFA increases authentication assurance but does not create permissions.

I13:
AI Agents require explicit authorization context.

I14:
Critical authorization decisions must be auditable.

I15:
Authorization state must be recoverable.

I16:
Authorization caches must be tenant/context isolated.
66. Traceability
Blueprint

Este ADR se relaciona directamente con:

Identity.
Authentication.
Authorization.
RBAC.
ABAC.
Tenant Isolation.
Resource Ownership.
Membership.
Security Policy Engine.
MFA.
Sessions.
Federation.
AI Agents.
Audit.
Event-Driven Architecture.
Previous ADRs
ADR-IDENTITY-005 — Authorization & Permission Evaluation
ADR-IDENTITY-007 — Domain Events & Outbox
ADR-IDENTITY-008 — Cryptographic Key Management
ADR-IDENTITY-011 — Runtime & Deployment
ADR-IDENTITY-021 — Controlled Replay
ADR-IDENTITY-022 — Consumer Idempotency
ADR-IDENTITY-025 — Refresh Token Cryptography & Concurrency
ADR-IDENTITY-026 — API Pagination, Filtering & Idempotency
ADR-IDENTITY-028 — Observability & Operational Security
ADR-IDENTITY-029 — Disaster Recovery & Backup
ADR-IDENTITY-031 — High Availability & Failover
ADR-IDENTITY-032 — Rate Limiting & Abuse Prevention
ADR-IDENTITY-033 — Account Lifecycle
ADR-IDENTITY-037 — API Tokens & Service Credentials
ADR-IDENTITY-038 — Service-to-Service Authentication
ADR-IDENTITY-039 — OAuth2.1 & OIDC
ADR-IDENTITY-040 — Federation
ADR-IDENTITY-041 — Provisioning
ADR-IDENTITY-042 — Source of Truth
ADR-IDENTITY-043 — Multi-Organization
ADR-IDENTITY-044 — Membership Roles & Delegation
67. Pending Decisions

Este ADR define la arquitectura, pero todavía debemos especificar:

Lenguaje/formato de policies.
Policy DSL vs configuración estructurada.
Policy evaluation engine.
Policy repository.
Policy compilation.
Policy versioning schema.
Policy precedence formal.
Conflict resolution formal.
Exact authorization context schema.
Decision object schema.
Reason-code catalog.
Obligations model.
PEP implementation.
PDP deployment model.
PAP administration model.
Cache architecture.
Cache TTL.
Cache invalidation.
Consistency guarantees.
Policy simulation.
Policy testing framework.
Explainability levels.
Audit decision granularity.
Risk scoring integration.
MFA/step-up integration.
AI policy analysis.
AI Agent decision constraints.
Cross-tenant platform authorization.
Resource sharing integration.
PostgreSQL enforcement / Row-Level Security.
Performance benchmarks.
Failure-mode behavior per operation criticality.
68. Acceptance Criteria

El ADR podrá considerarse aceptado cuando:

 Exista un Authorization Engine conceptualmente centralizado.
 Exista un Authorization Context formal.
 User, Organization y Membership sean evaluados.
 Tenant Isolation sea obligatorio.
 RBAC sea soportado.
 ABAC sea soportado.
 Ownership sea soportado.
 Delegation sea soportada.
 Security Policy Engine sea integrado.
 Existan decisiones ALLOW/DENY/CHALLENGE/REAUTHENTICATE/REQUIRE_MFA/REVOKE.
 Exista precedencia formal de policies.
 DENY conflictivo tenga prioridad.
 El sistema falle closed.
 JWT no pueda sustituir autorización actual.
 Exista policy versioning.
 Exista decision explainability controlada.
 Existan PEP/PDP conceptuales.
 Exista autorización específica para AI Agents.
 Cross-tenant access sea DENY by default.
 Authorization cache esté aislada por tenant.
 Exista audit/security telemetry.
 Existan pruebas de privilege escalation y tenant isolation.
 Authorization state pueda recuperarse después de DR.
 Se cumplan los objetivos de performance definidos.
69. Architectural Outcome

Con ADR-IDENTITY-043, 044 y 045, ya tenemos prácticamente definida la cadena conceptual completa de autorización de EVOXA:

                         ┌──────────────────┐
                         │      ACTOR       │
                         │ User / Service / │
                         │ AI Agent / System│
                         └────────┬─────────┘
                                  │
                                  ▼
                         ┌──────────────────┐
                         │    SESSION /     │
                         │ AUTH ASSURANCE   │
                         └────────┬─────────┘
                                  │
                                  ▼
                         ┌──────────────────┐
                         │  ORGANIZATION    │
                         │  TENANT CONTEXT  │
                         └────────┬─────────┘
                                  │
                                  ▼
                         ┌──────────────────┐
                         │    MEMBERSHIP    │
                         └────────┬─────────┘
                                  │
                    ┌─────────────┼─────────────┐
                    ▼             ▼             ▼
                  ROLES      DELEGATION     OWNERSHIP
                    │             │             │
                    └─────────────┼─────────────┘
                                  ▼
                         ┌──────────────────┐
                         │   PERMISSIONS    │
                         └────────┬─────────┘
                                  │
                                  ▼
                         ┌──────────────────┐
                         │      ABAC        │
                         │ Context / Risk   │
                         └────────┬─────────┘
                                  │
                                  ▼
                     ┌────────────────────────┐
                     │ SECURITY POLICY ENGINE │
                     │ PDP / Policy Evaluation│
                     └────────────┬───────────┘
                                  │
                     ┌────────────┼────────────┐
                     ▼            ▼            ▼
                   ALLOW         DENY       CHALLENGE
                                              │
                                              ▼
                                           MFA /
                                         Re-auth
                                              │
                                              ▼
                                       RE-EVALUATE

La regla arquitectónica fundamental queda:

Authorization no es una comprobación de Role; es una decisión contextual que combina identidad, estado de cuenta, Organization, Membership, Roles, Permissions, Ownership, Delegation, ABAC, autenticación, riesgo y Security Policy, produciendo una decisión explícita y auditable.

Y esto es importante para EVOXA porque significa que antes de comenzar a programar los endpoints de autorización, ya tenemos definida la arquitectura conceptual del motor. Lo que queda ahora es convertir esta arquitectura en especificaciones técnicas concretas: modelo de datos, contratos API, policy schema, algoritmo de evaluación, cache, errores y pruebas.
