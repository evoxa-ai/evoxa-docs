IS-IDENTITY-011 — Security Policy Engine

Estado: Draft
Prioridad: High / Critical para seguridad
Dominio: Identity
Capability: Security Policy Engine
Dependencias: IS-IDENTITY-001 → IS-IDENTITY-010
Siguiente historia: IS-IDENTITY-012 — Audit & Security Events

1. Propósito

Definir el Security Policy Engine de EVOXA como el componente responsable de evaluar políticas de seguridad y determinar qué nivel de confianza o control adicional debe aplicarse a una operación.

El Policy Engine debe poder tomar decisiones como:

ALLOW
DENY
CHALLENGE
REAUTHENTICATE
REQUIRE_MFA
REVOKE

El motor debe evaluar el contexto completo disponible y aplicar las políticas correspondientes, manteniendo separación entre:

Authentication.
Authorization.
Session Management.
MFA.
Tenant Isolation.
Resource Ownership.
Security Policy.

El Blueprint establece el Security Policy Engine como una pieza fundamental del modelo de autorización y seguridad de Identity.

2. Business Goal

Como plataforma EVOXA, necesitamos que las decisiones de seguridad no estén dispersas dentro de cada endpoint o módulo.

Queremos centralizar reglas que permitan responder preguntas como:

¿Puede este usuario realizar esta operación bajo estas condiciones?

Pero también:

¿Debe autenticarse nuevamente?

¿Debe completar MFA?

¿Debe ser desafiado?

¿Debe bloquearse la operación?

¿Debe revocarse la sesión?

El objetivo es pasar de:

Endpoint
 └── if user.role == ...

a:

Operation
    ↓
Security Policy Engine
    ↓
Policy Evaluation
    ↓
Security Decision
3. Concepto Fundamental

El Policy Engine no es solamente RBAC.

Debe poder evaluar:

RBAC
 +
ABAC
 +
Tenant Isolation
 +
Resource Ownership
 +
Authentication Context
 +
MFA
 +
Session
 +
Device
 +
Request Context
 +
Risk

Por lo tanto:

Security Policy Engine
        =
Context + Policies + Rules + Decision
4. Personas
Primary
Coach
Nutritionist
Physiotherapist
Team Manager
Admin
Security
Security Administrator
Organization Administrator
System
Identity Service
API Gateway
Application Services
AI Services
5. User Journey
5.1 Operación normal
User Request
     ↓
Authentication Context
     ↓
Authorization Request
     ↓
Security Policy Engine
     ↓
Evaluate Policies
     ↓
Decision
     ↓
ALLOW / DENY / ...
5.2 Operación que requiere MFA
Request
  ↓
Policy Evaluation
  ↓
REQUIRE_MFA
  ↓
MFA Challenge
  ↓
MFA Verified
  ↓
Policy Evaluation
  ↓
ALLOW
5.3 Reautenticación
Request
  ↓
Policy
  ↓
REAUTHENTICATE
  ↓
Authentication
  ↓
Policy Re-evaluation
  ↓
ALLOW
5.4 Riesgo elevado
Request
  ↓
Risk Context
  ↓
Policy
  ↓
CHALLENGE

El challenge puede conducir a:

MFA.
Reauthentication.
Otro mecanismo de verificación aprobado.
6. Feature Pack

Esta IS cubre:

Policy Core
Policy Definition.
Policy Evaluation.
Policy Context.
Policy Decision.
Policy Priority.
Policy Versioning.
Security Controls
RBAC.
ABAC.
Tenant Isolation.
Ownership.
MFA.
Reauthentication.
Risk.
Device context.
Governance
Policy lifecycle.
Policy changes.
Policy audit.
Policy versioning.
Policy traceability.
Runtime
Policy caching.
Evaluation performance.
Fail-closed.
Observability.
7. Scope
In Scope
Security Policy Engine.
Policy Context.
Policy Evaluation.
Policy Decision.
Policy precedence.
Rule evaluation.
RBAC integration.
ABAC integration.
Tenant isolation.
Ownership.
MFA integration.
Session integration.
Authentication integration.
Risk context.
Policy lifecycle.
Policy versioning.
Audit.
Security events.
Observability.
Out of Scope

No se implementa aquí:

User authentication.
Passwords.
JWT.
Refresh Token rotation.
MFA factor implementation.
Role CRUD.
Permission CRUD.
Full Audit implementation.
General workflow engine.
8. Security Decision Model

El resultado de evaluación debe ser explícito:

AuthorizationDecision
├── decision
├── reason
├── policy_id
├── policy_version
├── evaluated_at
└── metadata
Decision values
ALLOW
DENY
CHALLENGE
REAUTHENTICATE
REQUIRE_MFA
REVOKE
9. Meaning of Decisions
ALLOW

La operación puede continuar.

DENY

La operación debe ser rechazada.

CHALLENGE

Se requiere una verificación adicional antes de continuar.

REAUTHENTICATE

El usuario debe volver a autenticarse.

REQUIRE_MFA

Debe completarse una autenticación multifactor.

REVOKE

Debe invalidarse el contexto de seguridad afectado.

Por ejemplo:

Session
Refresh Token Family
Trusted Device

dependiendo de la política.

10. Authorization Request

El Policy Engine recibe un contexto conceptual:

AuthorizationRequest
├── actor
├── organization
├── membership
├── roles
├── permissions
├── session
├── authentication_method
├── mfa_context
├── device_context
├── request_context
├── resource
├── operation
└── risk_context

Este modelo deriva directamente del diseño de autorización definido previamente.

11. Actor Context

Debe poder conocer:

user_id
user_status
roles
permissions
membership
authentication_method
mfa_status

Nunca se deben incluir secretos.

12. Organization Context

Debe incluir:

organization_id
organization_status
membership_id
membership_status

Esto permite comprobar:

User
  ↓
Membership
  ↓
Organization

antes de permitir una operación.

13. Tenant Isolation

Tenant isolation tiene precedencia sobre permisos.

Ejemplo:

User belongs to Organization A

Request resource:
Organization B

Resultado:

DENY

Aunque el usuario posea un permiso global que normalmente permitiría la operación, el contexto de tenant debe impedir acceso no autorizado.

14. RBAC Integration

RBAC aporta:

Role
 ↓
RolePermission
 ↓
Permission

Ejemplo:

coach
  ↓
patients.read

Pero:

Permission exists

no significa automáticamente:

ALLOW

El Policy Engine debe evaluar el resto del contexto.

15. ABAC Integration

ABAC permite evaluar atributos.

Ejemplos conceptuales:

user.department
organization.type
resource.owner_id
resource.status
request.ip
device.trust
session.age
mfa.status
risk.level

La política puede combinar varios atributos.

16. Ownership

El Policy Engine debe poder evaluar ownership.

Ejemplo:

User A
  ↓
owns
  ↓
Resource X

Puede producir:

ALLOW

mientras:

User A
  ↓
Resource Y
  ↓
owned by User B

puede producir:

DENY

aunque exista un permiso genérico.

17. Policy Precedence

El modelo establecido previamente propone:

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
Contextual

Las políticas de mayor precedencia no deben ser anuladas por políticas de menor precedencia.

18. DENY Precedence

Cuando existe conflicto:

ALLOW
 +
DENY

debe prevalecer:

DENY

salvo que una regla superior explícitamente establezca otro comportamiento.

19. Fail Closed

Este es un requisito crítico.

Si el Policy Engine no puede determinar de forma segura la decisión:

Policy evaluation unavailable
          ↓
DENY

Nunca:

Policy unavailable
      ↓
ALLOW

No debe existir un bypass por:

Error de base de datos.
Timeout.
Cache failure.
Servicio externo caído.
Policy malformed.
Context incompleto.
20. Policy Entity

Entidad conceptual:

SecurityPolicy
├── id
├── name
├── description
├── scope
├── status
├── priority
├── version
├── rules
├── effective_from
├── effective_until
├── created_at
├── updated_at
└── metadata

La estructura exacta de rules debe definirse mediante el diseño técnico final.

21. Policy Scope

Conceptualmente:

SYSTEM
PLATFORM
ORGANIZATION
ROLE
USER
CONTEXTUAL

La jerarquía debe mantenerse alineada con la precedencia definida anteriormente.

22. Policy Lifecycle

Se propone:

DRAFT
  ↓
ACTIVE
  ↓
SUSPENDED
  ↓
RETIRED

El catálogo definitivo y las transiciones deberán validarse antes de implementación.

23. Policy Versioning

Las políticas deben ser versionadas.

Ejemplo:

Policy P-001

v1
 ↓
v2
 ↓
v3

Una decisión de seguridad debe poder identificar:

policy_id
policy_version

para permitir reproducibilidad y auditoría.

24. Policy Changes

Un cambio de política puede modificar el comportamiento de seguridad de todo un tenant.

Por ello:

Policy Change
     ↓
Audit
     ↓
Event
     ↓
Cache Invalidation
     ↓
New Evaluation

No debe existir actualización silenciosa.

25. Policy Evaluation

Flujo conceptual:

AuthorizationRequest
        ↓
Normalize Context
        ↓
Validate Context
        ↓
Load Applicable Policies
        ↓
Order by Precedence
        ↓
Evaluate Rules
        ↓
Resolve Conflicts
        ↓
Generate Decision
        ↓
Audit / Metrics
26. Policy Evaluation Service

Debe existir un servicio de dominio/aplicación equivalente a:

SecurityPolicyService

Responsabilidades:

Resolver políticas aplicables.
Evaluar reglas.
Resolver conflictos.
Determinar decisión.
Proporcionar reason.
Proporcionar policy ID/version.
Mantener fail-closed.
27. Separation of Concerns

Debe mantenerse:

Authentication
    ↓
Who are you?

Authorization
    ↓
What can you do?

Security Policy
    ↓
Under what conditions can you do it?

Esta separación es fundamental.

28. MFA Integration

El Policy Engine puede producir:

REQUIRE_MFA

Ejemplo:

User
 ↓
Authenticated
 ↓
Attempt sensitive operation
 ↓
Policy
 ↓
REQUIRE_MFA

Después:

MFA VERIFIED
 ↓
Policy Re-evaluation
 ↓
ALLOW
29. Reauthentication

Puede producir:

REAUTHENTICATE

Ejemplo:

Session age > allowed threshold
        ↓
Sensitive operation
        ↓
REAUTHENTICATE

El Policy Engine no ejecuta necesariamente la autenticación.

Determina que debe realizarse.

30. Risk Context

El contexto puede incluir un nivel de riesgo:

LOW
MEDIUM
HIGH
CRITICAL

Este catálogo es conceptual y queda sujeto a la definición del sistema de Risk Management.

Ejemplo:

HIGH RISK
   +
Sensitive Operation
   ↓
REQUIRE_MFA

o:

CRITICAL RISK
   ↓
DENY
31. Device Context

El Policy Engine puede considerar:

device_id
device_type
trusted
first_seen
last_seen
security_state

No debe confiar únicamente en datos proporcionados por el cliente.

32. Session Context

Puede considerar:

session_id
session_status
session_age
authentication_method
mfa_status
mfa_verified_at

Esto permite reglas como:

IF session_age > threshold
THEN REAUTHENTICATE

Los thresholds deben ser configurables.

33. Request Context

Puede incluir:

request_id
operation
resource
method
source
network_context
timestamp

El contexto debe minimizar información innecesaria.

34. AI Authorization Context

El Blueprint establece una regla especialmente importante:

El contexto proporcionado a servicios de IA debe ser un subconjunto del contexto autorizado para el usuario.

Por tanto:

User Authorized Context
          ↓
Policy Engine
          ↓
Authorized Context
          ↓
AI Service

Nunca:

AI Service
   ↓
consulta datos directamente
   ↓
bypass Authorization

El Policy Engine se convierte así en una barrera importante para la arquitectura AI-Native de EVOXA.

35. Policy Cache

Puede utilizarse cache para mejorar rendimiento.

Pero:

Policy Cache

no debe convertirse en fuente de verdad permanente.

Debe existir invalidación cuando cambien:

Policy.
Role.
Permission.
Membership.
Organization.
User.
Security context relevante.
36. Cache Failure

Si el cache no está disponible:

Cache failure
    ↓
Fallback to authoritative source

Si tampoco puede obtenerse una evaluación segura:

DENY
37. API Conceptual

El contrato final se definirá en IS-IDENTITY-014.

Evaluate Authorization
POST /api/v1/identity/authorization/check

Request conceptual:

actor
organization
resource
operation
context

Response:

decision
reason
policy_id
policy_version
metadata
38. Administrative Policy APIs

Podrán existir posteriormente:

GET    /security-policies
POST   /security-policies
GET    /security-policies/{id}
PATCH  /security-policies/{id}
POST   /security-policies/{id}/activate
POST   /security-policies/{id}/suspend
POST   /security-policies/{id}/retire

Los contratos no son definitivos hasta IS-014.

39. Domain Events

Cambios de políticas deben producir eventos como:

SecurityPolicyCreated
SecurityPolicyUpdated
SecurityPolicyActivated
SecurityPolicySuspended
SecurityPolicyRetired

Decisiones relevantes pueden producir:

AuthorizationDecisionMade
AuthorizationDenied
AuthorizationChallengeRequired
PolicyViolationDetected
40. Security Events

Eventos especialmente importantes:

AuthorizationDenied
TenantAccessDenied
PermissionDenied
PolicyViolationDetected
PrivilegeEscalationDetected
AuthorizationChallengeRequired

Estos eventos ya forman parte del modelo de seguridad definido anteriormente.

41. Audit

El Policy Engine debe proporcionar trazabilidad suficiente para responder:

Who?
What?
When?
Where?
Which organization?
Which resource?
Which operation?
Which policy?
Which version?
What decision?
Why?

Ejemplo conceptual:

User: U123
Organization: O456
Resource: Patient
Operation: patient.read
Policy: P-100
Version: 7
Decision: DENY
Reason: ownership_mismatch

No deben registrarse secretos ni datos innecesarios.

42. Security Rules
BR-011-001 — Fail Closed

No existe ALLOW cuando no puede determinarse una decisión segura.

BR-011-002 — Tenant Isolation

Tenant mismatch siempre debe impedir el acceso no autorizado.

BR-011-003 — DENY Precedence

En conflicto, DENY prevalece según la jerarquía establecida.

BR-011-004 — Policy Precedence

Reglas de mayor prioridad no pueden ser anuladas por reglas inferiores.

BR-011-005 — MFA

REQUIRE_MFA debe impedir la continuación hasta completar MFA.

BR-011-006 — Reauthentication

REAUTHENTICATE debe impedir la operación hasta completar reautenticación.

BR-011-007 — Ownership

Ownership debe evaluarse cuando la operación lo requiera.

BR-011-008 — Context Integrity

El Policy Engine no debe confiar ciegamente en atributos enviados por el cliente.

BR-011-009 — Versioning

Las decisiones deben ser trazables a una versión de política.

BR-011-010 — AI Context

AI solamente puede recibir contexto autorizado.

BR-011-011 — No Secret Data

Las políticas no deben contener secretos.

BR-011-012 — Auditability

Cambios de políticas y decisiones sensibles deben ser auditables.

43. Acceptance Criteria
AC-011-01 — Allow

Given un contexto autorizado
When se evalúa una operación
Then el Policy Engine debe devolver ALLOW.

AC-011-02 — Deny

Given un contexto no autorizado
When se evalúa una operación
Then debe devolver DENY.

AC-011-03 — Tenant Isolation

Given un usuario de Organization A
When intenta acceder a un recurso perteneciente a Organization B sin autorización válida
Then debe devolver DENY.

AC-011-04 — MFA

Given una política que exige MFA
When un usuario sin MFA verificado intenta ejecutar la operación
Then debe devolver REQUIRE_MFA.

AC-011-05 — Reauthentication

Given una política que exige reautenticación
When el contexto actual no satisface el requisito
Then debe devolver REAUTHENTICATE.

AC-011-06 — Challenge

Given una operación con riesgo elevado
When una política requiere verificación adicional
Then debe devolver CHALLENGE.

AC-011-07 — Policy Conflict

Given una política ALLOW y una política DENY aplicables
When se evalúa la operación
Then debe prevalecer la política DENY conforme a la precedencia.

AC-011-08 — Policy Failure

Given que el motor no puede obtener una evaluación confiable
When se procesa una operación protegida
Then debe producirse DENY.

AC-011-09 — Policy Version

Given una decisión de autorización
When se registra el resultado
Then debe poder identificarse la policy y version utilizadas.

AC-011-10 — AI Context

Given un AI Service
When solicita contexto de un usuario
Then solamente debe recibir información dentro del contexto autorizado.

44. Technical Tasks
Domain

TSK-011-01 — Crear SecurityPolicy.

TSK-011-02 — Crear PolicyId.

TSK-011-03 — Crear PolicyVersion.

TSK-011-04 — Crear PolicyContext.

TSK-011-05 — Crear PolicyDecision.

TSK-011-06 — Definir Decision enum.

TSK-011-07 — Definir policy lifecycle.

Policy Engine

TSK-011-08 — Crear SecurityPolicyService.

TSK-011-09 — Implementar policy resolution.

TSK-011-10 — Implementar policy precedence.

TSK-011-11 — Implementar rule evaluation.

TSK-011-12 — Implementar conflict resolution.

TSK-011-13 — Implementar fail-closed behavior.

Authorization Integration

TSK-011-14 — Integrar RBAC.

TSK-011-15 — Integrar ABAC.

TSK-011-16 — Integrar tenant isolation.

TSK-011-17 — Integrar ownership.

TSK-011-18 — Integrar Session context.

TSK-011-19 — Integrar MFA context.

TSK-011-20 — Integrar Authentication context.

TSK-011-21 — Integrar Risk context.

TSK-011-22 — Integrar Device context.

Policy Management

TSK-011-23 — Crear Policy Repository.

TSK-011-24 — Implementar create policy.

TSK-011-25 — Implementar update policy.

TSK-011-26 — Implementar activate policy.

TSK-011-27 — Implementar suspend policy.

TSK-011-28 — Implementar retire policy.

TSK-011-29 — Implementar policy versioning.

Infrastructure

TSK-011-30 — Crear persistence model.

TSK-011-31 — Crear índices.

TSK-011-32 — Implementar policy cache.

TSK-011-33 — Implementar cache invalidation.

TSK-011-34 — Implementar fallback behavior.

Security

TSK-011-35 — Implementar tenant enforcement.

TSK-011-36 — Implementar ownership enforcement.

TSK-011-37 — Implementar MFA requirement.

TSK-011-38 — Implementar reauthentication requirement.

TSK-011-39 — Implementar challenge decision.

TSK-011-40 — Implementar revoke decision.

TSK-011-41 — Implementar privilege escalation detection.

Events / Audit

TSK-011-42 — Implementar Policy Events.

TSK-011-43 — Implementar Security Events.

TSK-011-44 — Integrar AuditService.

Observability

TSK-011-45 — Metrics.

TSK-011-46 — Secure Logging.

TSK-011-47 — Distributed Tracing.

Testing

TSK-011-48 — Unit tests.

TSK-011-49 — Integration tests.

TSK-011-50 — Authorization tests.

TSK-011-51 — Tenant isolation tests.

TSK-011-52 — Ownership tests.

TSK-011-53 — MFA policy tests.

TSK-011-54 — Reauthentication tests.

TSK-011-55 — Policy conflict tests.

TSK-011-56 — Fail-closed tests.

TSK-011-57 — Cache failure tests.

TSK-011-58 — Performance tests.

TSK-011-59 — Security tests.

TSK-011-60 — AI context isolation tests.

45. Testing Strategy
Unit

Probar exhaustivamente:

Policy evaluation.
Precedence.
Conflict resolution.
Decisions.
Lifecycle.
Versioning.
Integration

Validar integración con:

User
Organization
Membership
Role
Permission
Session
MFA
Security

Casos críticos:

Cross-Tenant
Privilege Escalation
Ownership Bypass
MFA Bypass
Reauthentication Bypass
Policy Bypass
Fail Open
AI Context Leakage
46. Performance

El Policy Engine se encontrará potencialmente en el camino crítico de una gran cantidad de requests.

Por ello debe diseñarse para:

Baja latencia.
Evaluaciones determinísticas.
Cache eficiente.
Evitar llamadas innecesarias.
Evitar consultas N+1.

El objetivo general del Blueprint establece una meta de P95 de API inferior a 300 ms, por lo que el Policy Engine debe diseñarse como un componente compatible con ese presupuesto de latencia.

47. Observability
Metrics
identity.policy.evaluations
identity.policy.allow
identity.policy.deny
identity.policy.challenge
identity.policy.require_mfa
identity.policy.reauthenticate
identity.policy.revoke
identity.policy.errors
identity.policy.cache_hit
identity.policy.cache_miss
Security Metrics
tenant_denials
permission_denials
policy_violations
privilege_escalation_attempts
mfa_requirements
reauthentication_requirements
48. Failure Modes
Policy Store Unavailable
Policy Store unavailable
        ↓
No trusted evaluation
        ↓
DENY
Cache Stale

Debe existir invalidación/version checking para evitar decisiones basadas en políticas obsoletas.

Invalid Policy

Una política inválida no debe entrar en estado ACTIVE.

Policy Conflict

Debe resolverse mediante precedencia determinística.

Nunca depender de orden accidental de ejecución.

Missing Context

Si falta un atributo crítico:

Missing security context
       ↓
Cannot safely evaluate
       ↓
DENY / CHALLENGE

según la naturaleza del contexto requerido.

49. Definition of Done
 SecurityPolicy Entity implementada.
 PolicyContext implementado.
 PolicyDecision implementado.
 Decision model implementado.
 Policy precedence implementada.
 DENY precedence implementada.
 Fail-closed implementado.
 Policy lifecycle implementado.
 Policy versioning implementado.
 RBAC integrado.
 ABAC integrado.
 Tenant isolation integrado.
 Ownership integrado.
 Session integrado.
 MFA integrado.
 Authentication integrado.
 Risk context integrado.
 Device context integrado.
 Policy Repository implementado.
 Persistence implementada.
 Cache implementado.
 Cache invalidation implementado.
 Events implementados.
 Audit implementado.
 Metrics implementadas.
 Logging seguro implementado.
 Tracing implementado.
 Unit tests implementados.
 Integration tests implementados.
 Security tests implementados.
 Cross-tenant tests implementados.
 Fail-closed tests implementados.
 AI authorization context tests implementados.
 Performance tests implementados.
 Documentación actualizada.
50. Traceability
Elemento	Referencia
Blueprint	Identity / Authorization / Security Policy Engine
ESP	ESP-0001 — Identity
ADR	ADR-IDENTITY-005
ADR	ADR-IDENTITY-006
ADR	ADR-IDENTITY-007
ADR	ADR-IDENTITY-008
ADR	ADR-IDENTITY-010
Previous IS	IS-IDENTITY-006 — Authorization
Previous IS	IS-IDENTITY-007 — Roles & Permissions
Previous IS	IS-IDENTITY-008 — Session
Previous IS	IS-IDENTITY-009 — Refresh Token
Previous IS	IS-IDENTITY-010 — MFA
Next IS	IS-IDENTITY-012 — Audit & Security Events
API	IS-IDENTITY-014
Persistence	IS-IDENTITY-015
Testing	IS-IDENTITY-016
Runtime	IS-IDENTITY-017

La trazabilidad sigue el modelo ESP → IS → Technical Tasks → Source Code definido para el ciclo de implementación.

51. Architecture Model
                         ┌──────────────────────┐
                         │        Request       │
                         └──────────┬───────────┘
                                    ↓
                         ┌──────────────────────┐
                         │ Authentication       │
                         └──────────┬───────────┘
                                    ↓
                         ┌──────────────────────┐
                         │ Authorization        │
                         │ Request              │
                         └──────────┬───────────┘
                                    ↓
                  ┌──────────────────────────────────┐
                  │      Security Policy Engine      │
                  │                                  │
                  │  RBAC                            │
                  │  ABAC                            │
                  │  Tenant Isolation                │
                  │  Ownership                       │
                  │  MFA                             │
                  │  Session                         │
                  │  Device                          │
                  │  Risk                            │
                  └──────────────────┬───────────────┘
                                     ↓
                            ┌─────────────────┐
                            │ Security        │
                            │ Decision        │
                            └───────┬─────────┘
                                    ↓
             ┌──────────────┬───────┼────────┬──────────────┐
             ↓              ↓       ↓        ↓              ↓
          ALLOW           DENY   CHALLENGE  MFA       REAUTHENTICATE
52. Security Decision Pipeline

El modelo completo de Identity queda ahora:

                  AUTHENTICATION
                        │
                        ↓
                     SESSION
                        │
                        ↓
              AUTHORIZATION REQUEST
                        │
                        ↓
             ┌──────────────────────┐
             │ Security Policy      │
             │ Engine               │
             └──────────┬───────────┘
                        │
          ┌─────────────┼──────────────┐
          ↓             ↓              ↓
        RBAC           ABAC        TENANT
          │             │              │
          └─────────────┼──────────────┘
                        ↓
                  OWNERSHIP
                        ↓
                     MFA
                        ↓
                     RISK
                        ↓
                   DECISION
                        │
          ┌─────────────┼─────────────┐
          ↓             ↓             ↓
       ALLOW          DENY         CHALLENGE
                        │
                ┌───────┴────────┐
                ↓                ↓
          REQUIRE_MFA      REAUTHENTICATE
53. AI Security Boundary

Una de las consecuencias arquitectónicas más importantes para EVOXA es que el Policy Engine debe convertirse en una frontera de seguridad también para la IA:

                    User
                     ↓
               Authorization
                     ↓
              Policy Engine
                     ↓
           Authorized Context
                     ↓
                  AI
                     ↓
            AI Response/Action

La IA no debe convertirse en un mecanismo alternativo para saltarse Identity.

Esto será especialmente importante cuando EVOXA implemente:

AI Coaches.
AI Nutrition.
AI Planning.
AI Agents.
Autonomous Operations.
54. Architectural Decisions Pending

Antes de pasar esta IS a Approved, deben resolverse mediante ADR o especificaciones técnicas:

Lenguaje formal de políticas.
Estructura exacta de Policy Rules.
Policy evaluation engine.
Policy storage model.
Policy versioning mechanism.
Policy inheritance.
Policy conflict resolution detallada.
Policy simulation/dry-run.
Policy testing framework.
Policy rollback.
Cache strategy.
Cache invalidation.
Risk Engine integration.
Device Trust integration.
MFA policy integration.
Regulatory policy integration.
Organization policy customization.
AI authorization context enforcement.
Policy administration permissions.
Policy change approval workflow.
55. Dependency Flow

Con IS-011 completada conceptualmente:

001 Identity Foundation
        ↓
002 User
        ↓
003 Organization
        ↓
004 Membership
        ↓
005 Authentication
        ↓
006 Authorization
        ↓
007 Roles & Permissions
        ↓
008 Session
        ↓
009 Refresh Token
        ↓
010 MFA
        ↓
011 Security Policy Engine
        ↓
012 Audit & Security Events
        ↓
013 Domain Events & Outbox
        ↓
014 Identity API Contracts
        ↓
015 Identity Persistence
        ↓
016 Identity Testing & Quality
        ↓
017 Identity Runtime & Deployment
Estado

IS-IDENTITY-011 — Security Policy Engine queda definida como Draft — High/Critical.

Con esta historia ya tenemos definida la pieza que conecta prácticamente todo el modelo de seguridad de Identity:

User → Organization → Membership → Authentication → Session → MFA → RBAC/ABAC → Tenant Isolation → Ownership → Security Policy → Decision.

El siguiente bloque, IS-IDENTITY-012 — Audit & Security Events, deberá encargarse de convertir todas estas operaciones y decisiones en una traza de seguridad auditable, inmutable y correlacionable, sin almacenar secretos ni información sensible innecesaria.
