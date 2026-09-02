IS-IDENTITY-006 — Authorization & Permission Evaluation

Implementation Story ID: IS-IDENTITY-006
Título: Authorization & Permission Evaluation
Dominio: Identity
ESP: ESP-0001 — Identity
Prioridad: Critical
Estado: Draft
Versión: 1.0.0
Repositorio documental: evoxa-docs
Repositorio de implementación: evoxa-platform

1. Propósito

Esta Implementation Story define la capacidad de determinar si un actor autenticado está autorizado para realizar una operación concreta dentro de EVOXA.

La responsabilidad principal de Authorization es responder:

¿Este actor puede realizar esta operación sobre este recurso, dentro de este contexto y bajo estas políticas?

Authentication responde:

¿Quién eres?

Authorization responde:

¿Qué puedes hacer?

Y el Policy Engine determina:

¿Puedes hacerlo bajo estas condiciones?

La autorización de EVOXA se basa en la combinación de:

RBAC
+
ABAC
+
Tenant Isolation
+
Resource Ownership
+
Security Policy Engine

Esta decisión ya quedó establecida en ADR-IDENTITY-005 — Authorization & Policy Engine.

2. Business Goal

EVOXA necesita un mecanismo centralizado y consistente para evitar que usuarios accedan o modifiquen recursos para los cuales no tienen autorización.

Debe poder determinar:

Quién realiza la operación.
En qué organización.
Qué Membership posee.
Qué Roles tiene.
Qué Permissions tiene.
Qué recurso intenta utilizar.
Qué acción intenta realizar.
Si es propietario del recurso.
Qué políticas aplican.
Si MFA es requerido.
Si debe realizarse una reautenticación.
Si existen restricciones contextuales.
Si el acceso debe ser permitido o rechazado.
3. Principio Fundamental

La autorización nunca debe depender exclusivamente de:

user_id

ni exclusivamente de:

role

ni exclusivamente de:

permission

La decisión debe considerar el contexto completo:

                  Authorization Request
                           │
                           ▼
                    Security Context
                           │
          ┌────────────────┼────────────────┐
          │                │                │
         User         Membership       Organization
          │                │                │
          └────────────────┼────────────────┘
                           │
                        Roles
                           │
                     Permissions
                           │
                    Resource Context
                           │
                     Policy Engine
                           │
                           ▼
                       Decision
4. Persona

Actores principales:

Platform Administrator.
Organization Administrator.
Organization Owner.
Team Manager.
Coach.
Nutritionist.
Physiotherapist.
Usuario final.
Service Account.

La autorización debe funcionar independientemente del tipo de cliente:

Web
Mobile
API Consumer
Service
AI Agent
5. Feature Pack

Identity → Authorization & Permission Evaluation

Incluye:

Authorization Context.
Permission Evaluation.
RBAC Evaluation.
ABAC Evaluation.
Tenant Isolation.
Resource Ownership.
Policy Evaluation.
Authorization Decision.
Security Context.
Authorization caching.
Authorization audit.
Security events.
AI authorization boundary.
Fail-closed behavior.
6. Alcance
Incluido

Esta IS cubre:

AuthorizationService.
Authorization Context.
Authorization Request.
Authorization Decision.
Permission evaluation.
RBAC.
ABAC.
Tenant isolation.
Ownership evaluation.
Security Policy integration.
Decision precedence.
Fail-closed behavior.
Authorization cache boundary.
Audit integration.
Authorization events.
Security tests.
Cross-tenant tests.
Fuera de alcance

No implementa:

Authentication.
Passwords.
JWT issuance.
Refresh token lifecycle.
Session lifecycle.
MFA factor management.
Role creation.
Permission creation.
Organization creation.
Membership creation.
Business-domain authorization rules específicas.

Las entidades Role y Permission serán administradas mediante:

IS-IDENTITY-007 — Role & Permission Management.

7. Authorization Model

El modelo principal:

Authentication
      ↓
Security Context
      ↓
Authorization Request
      ↓
RBAC
      ↓
ABAC
      ↓
Tenant Isolation
      ↓
Resource Ownership
      ↓
Security Policy
      ↓
Authorization Decision
8. Authorization Request

Se define conceptualmente:

AuthorizationRequest
├── actor
├── organization
├── membership
├── roles
├── permissions
├── session
├── authentication_method
├── MFA state
├── device
├── request
├── resource
├── operation
└── risk/context
9. Actor Context

El actor debe incluir como mínimo:

user_id
session_id

y, cuando corresponda:

organization_id
membership_id
roles
permissions
10. Resource Context

La autorización debe conocer el recurso sobre el cual se solicita la operación.

Ejemplo:

resource:
    type = training_program
    id = TP-123
    organization_id = ORG-001
    owner_id = USER-123

Esto permite aplicar:

tenant isolation;
ownership;
permissions;
policies.
11. Operation

Las operaciones deben ser expresables mediante un modelo consistente:

resource.action

Ejemplos:

users.read
users.update

training_programs.read
training_programs.create
training_programs.update

nutrition_plans.read
nutrition_plans.publish

assessments.read
assessments.approve

El naming final de permisos deberá ser definido en Role & Permission Management.

12. RBAC

RBAC significa:

Role-Based Access Control

La decisión se basa en:

User
 ↓
Membership
 ↓
Role
 ↓
Permission
 ↓
Operation

Ejemplo:

Coach
  ↓
training_programs.read
training_programs.update

Si el actor posee el permiso requerido, RBAC puede permitir la operación.

Pero RBAC por sí solo no es suficiente.

13. ABAC

ABAC significa:

Attribute-Based Access Control

La decisión puede depender de atributos del:

usuario;
Membership;
organización;
recurso;
operación;
dispositivo;
sesión;
contexto;
riesgo;
política.

Ejemplo conceptual:

User.role = Coach
AND
Membership.organization = Resource.organization
AND
Resource.owner = User

Entonces:

ALLOW
14. Tenant Isolation

Tenant isolation es una condición fundamental.

Ejemplo:

Actor
  │
  └── Membership → Organization A

Resource
  │
  └── Organization B

Resultado:

DENY

aunque el actor posea el permiso:

training_programs.read

Esto es importante:

Tener Permission no elimina Tenant Isolation.

15. Resource Ownership

La autorización debe poder evaluar ownership.

Ejemplo:

User A
   │
   └── owns → Training Plan 123

El actor podría tener:

training_plans.update

y además:

resource.owner_id == actor.user_id

Resultado:

ALLOW

Pero si:

resource.owner_id != actor.user_id

la política podría producir:

DENY
16. Security Policy Engine

Authorization debe integrarse con:

SecurityPolicyService

La evaluación puede producir:

ALLOW
DENY
CHALLENGE
REAUTHENTICATE
REQUIRE_MFA
REVOKE

Estos resultados corresponden al modelo definido previamente en ADR-IDENTITY-005.

17. Authorization Decision

Se define:

AuthorizationDecision
├── decision
├── reason
├── policy_id?
├── evaluated_at
└── metadata

La decisión principal será una de:

ALLOW
DENY
CHALLENGE
REAUTHENTICATE
REQUIRE_MFA
REVOKE
18. Decision Precedence

La precedencia definida conceptualmente es:

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

Además:

DENY > ALLOW

Es decir, una regla explícita de denegación debe prevalecer sobre una autorización equivalente.

19. Fail Closed

Authorization debe operar bajo:

Fail Closed

Si el sistema no puede determinar de forma segura si una operación está permitida:

Unknown
   ↓
DENY

No:

Unknown
   ↓
ALLOW

Esto es crítico para evitar authorization bypass.

20. User Stories
US-IDENTITY-006-01 — Evaluar Permission

Como sistema EVOXA
Quiero evaluar una permission
Para determinar si una operación está permitida.

Acceptance Criteria

AC-001

Given que el actor posee el permission requerido
And el contexto de tenant es válido
And no existe una política que lo bloquee
When se evalúa la autorización
Then el resultado puede ser ALLOW.
21. US-IDENTITY-006-02 — Denegar acceso sin Permission
Given que el actor no posee el permission requerido
When solicita una operación protegida
Then la autorización debe producir DENY.
22. US-IDENTITY-006-03 — Tenant Isolation
Given que el actor pertenece a Organization A
And el recurso pertenece a Organization B
When se evalúa autorización
Then debe producir DENY.
23. US-IDENTITY-006-04 — Ownership
Given que una política requiere ownership
And el actor no es propietario del recurso
When se evalúa la autorización
Then debe producir DENY.
24. US-IDENTITY-006-05 — MFA Requirement
Given que una política requiere MFA
And el actor no ha completado MFA
When se solicita una operación protegida
Then debe producir REQUIRE_MFA.
25. US-IDENTITY-006-06 — Reauthentication
Given que una política requiere reautenticación
When el actor solicita la operación
Then debe producir REAUTHENTICATE.
26. US-IDENTITY-006-07 — Security Challenge
Given que una política requiere challenge adicional
When se evalúa la operación
Then debe producir CHALLENGE.
27. Domain Rules
BR-IDENTITY-006-01

Toda operación protegida debe pasar por una evaluación de autorización.

BR-IDENTITY-006-02

Authentication no implica autorización.

BR-IDENTITY-006-03

Poseer un Permission no elimina las restricciones de Tenant Isolation.

BR-IDENTITY-006-04

Un Membership válido es necesario para acceso organizacional cuando corresponda.

BR-IDENTITY-006-05

Una Organization suspendida no debe permitir operaciones organizacionales normales.

BR-IDENTITY-006-06

Un User suspendido o disabled no debe obtener autorización efectiva.

BR-IDENTITY-006-07

Las políticas explícitas de DENY prevalecen sobre ALLOW.

BR-IDENTITY-006-08

Authorization debe fallar cerradamente ante incertidumbre o dependencia crítica no disponible.

BR-IDENTITY-006-09

La autorización de recursos específicos debe poder considerar ownership.

BR-IDENTITY-006-10

La autorización debe soportar RBAC y ABAC.

BR-IDENTITY-006-11

Las decisiones sensibles deben ser auditables.

BR-IDENTITY-006-12

El contexto de autorización debe ser correlacionable mediante correlation_id y trace_id.

28. Application Layer

Casos de uso principales:

EvaluateAuthorization
CheckPermission
CheckResourceAccess
EvaluatePolicy

Flujo:

Request
 ↓
Authentication Context
 ↓
Authorization Context
 ↓
RBAC
 ↓
ABAC
 ↓
Tenant Isolation
 ↓
Ownership
 ↓
Policy Engine
 ↓
Decision
29. Authorization Service

Contrato conceptual:

AuthorizationService

Operaciones:

authorize()
check_permission()
check_resource_access()
evaluate_policy()

El servicio debe devolver una decisión estructurada, no simplemente un boolean.

30. Permission Evaluation

Proceso:

1. Identify actor
2. Validate authentication
3. Resolve organization context
4. Resolve membership
5. Resolve roles
6. Resolve permissions
7. Identify resource
8. Identify operation
9. Evaluate tenant boundary
10. Evaluate ownership
11. Evaluate ABAC
12. Evaluate security policies
13. Resolve precedence
14. Produce decision
15. Audit if required
31. Authorization Context

Se define conceptualmente:

AuthorizationContext
├── user_id
├── organization_id
├── membership_id
├── roles
├── permissions
├── session_id
├── authentication_method
├── mfa_state
├── device_context
├── request_context
├── resource_context
├── operation
└── risk_context
32. Authorization Cache

La arquitectura permite caching de autorización cuando sea seguro.

Ejemplo:

Authorization Request
       ↓
Cache
       │
   ┌───┴───┐
   │       │
  HIT    MISS
   │       │
   ▼       ▼
Decision  Evaluate

Pero el cache no debe convertirse en una fuente de verdad permanente.

Debe existir una estrategia de invalidación cuando cambien:

Roles.
Permissions.
Membership.
Organization status.
User status.
Security policies.

La arquitectura de Redis definida para Identity permite su utilización para cache temporal/alta frecuencia, pero PostgreSQL sigue siendo la fuente transaccional. Esto deberá concretarse en Persistence/Infrastructure.

33. Authorization Cache Security

Nunca se debe permitir que una entrada cacheada sobreviva de manera insegura a cambios críticos.

Ejemplo:

User has permission
       ↓
Cache ALLOW
       ↓
Permission revoked
       ↓
Cache stale
       ↓
Potential security vulnerability

Por ello, los cambios de seguridad deben poder invalidar o hacer expirar las decisiones cacheadas.

34. Organization Context

Una autorización organizacional debe identificar explícitamente:

organization_id

Nunca debe depender únicamente de un valor proporcionado por el cliente.

Ejemplo inseguro:

POST /resource
organization_id = ORG-B

El sistema debe verificar que:

Authenticated User
       ↓
Membership
       ↓
ORG-B

antes de permitir el acceso.

35. AI Authorization Boundary

Los agentes y servicios de IA de EVOXA deben utilizar exactamente el mismo modelo de autorización.

Un AI Agent no debe obtener acceso superior al del contexto que lo autorizó.

Principio:

User Authorization
       ↓
AI Context
       ↓
AI Agent

El AI Agent no puede hacer:

ALLOW(user)
      ↓
AI Agent
      ↓
ALLOW(more data)

El acceso de IA debe ser un subset del contexto autorizado.

Esto es particularmente importante para Human Digital Twin, Nutrition, Training y otros dominios sensibles.

36. Service-to-Service Authorization

La autorización también debe soportar:

Service A
   ↓
Service B

utilizando mecanismos como:

service accounts;
OAuth2;
API keys;
service identity.

El servicio debe tener únicamente los permisos necesarios.

37. API Requirements

Namespace:

/api/v1/identity

Authorization normalmente funcionará como componente interno/middleware/policy layer, aunque puede existir un endpoint administrativo o de evaluación cuando sea necesario.

Conceptualmente:

POST /authorization/check

Request:

{
  "resource": {
    "type": "training_program",
    "id": "TP-123"
  },
  "action": "read"
}

Response conceptual:

{
  "decision": "ALLOW"
}

Esto no constituye todavía el contrato definitivo.

La especificación final pertenece a:

IS-IDENTITY-014 — Identity API Contracts.

38. Errors

Errores conceptuales:

Unauthorized
Forbidden
TenantAccessDenied
PermissionDenied
PolicyDenied
MFARequired
ReauthenticationRequired
AuthorizationChallengeRequired
AuthorizationContextInvalid

Importante:

Una operación legítimamente denegada no debe tratarse automáticamente como un error interno.

Por ejemplo:

403 Forbidden

puede representar una decisión normal del sistema.

39. Authorization Audit

Deben auditarse especialmente:

Denegaciones administrativas.
Cross-tenant attempts.
Policy violations.
Privilege escalation attempts.
Sensitive resource access.
MFA-required decisions cuando sean relevantes.
Reauthentication decisions.
Security challenges.
Authorization failures repetidos.

No necesariamente todas las evaluaciones exitosas necesitan un registro de auditoría persistente.

La política de auditoría deberá distinguir entre:

Operational Authorization

y:

Security-Sensitive Authorization

para evitar volumen innecesario.

40. Security Events

Eventos potenciales:

AuthorizationDenied
TenantAccessDenied
PermissionDenied
PolicyViolation
PrivilegeEscalationDetected
AuthorizationChallengeRequired

Los nombres definitivos deberán integrarse con el catálogo global de Security Events.

41. Observability

Métricas conceptuales:

authorization_requests_total
authorization_allowed_total
authorization_denied_total
authorization_latency
tenant_access_denied_total
permission_denied_total
policy_denied_total
mfa_required_total
reauthentication_required_total
authorization_cache_hit_total
authorization_cache_miss_total

Se debe poder correlacionar una autorización con la request original:

correlation_id
trace_id
42. Performance

Authorization es un componente de alta frecuencia.

Por tanto debe ser eficiente.

Objetivos:

API P95 < 300ms

La evaluación interna de autorización debe mantenerse suficientemente rápida para no convertirse en el cuello de botella de los dominios.

Se deben considerar:

caching;
índices adecuados;
context preloading;
Redis;
policy evaluation eficiente;
bounded timeouts.
43. Failure Modes
Authorization Service unavailable

Debe producir:

DENY / controlled failure

y nunca:

ALLOW

por defecto.

Redis unavailable

Si Redis únicamente contiene cache:

Fallback → authoritative evaluation

si es seguro hacerlo.

Si la operación depende de un estado temporal crítico:

Fail Closed
PostgreSQL unavailable

Si no puede determinarse el estado autoritativo:

DENY / controlled failure
44. Acceptance Criteria Consolidated
ID	Criterio
AC-001	Authorization evalúa una operación protegida
AC-002	Permission válido puede producir ALLOW
AC-003	Falta de Permission produce DENY
AC-004	Tenant mismatch produce DENY
AC-005	Membership inválido bloquea acceso
AC-006	Organization suspendida bloquea acceso organizacional
AC-007	User suspendido/disabled bloquea autorización efectiva
AC-008	Ownership puede formar parte de la decisión
AC-009	RBAC está soportado
AC-010	ABAC está soportado
AC-011	Security Policy Engine está integrado
AC-012	DENY prevalece sobre ALLOW
AC-013	El sistema falla cerrado
AC-014	MFARequired puede producir REQUIRE_MFA
AC-015	Reauthentication puede producir REAUTHENTICATE
AC-016	Challenge puede producir CHALLENGE
AC-017	Las decisiones sensibles son auditables
AC-018	Las decisiones pueden generar security events
AC-019	Authorization soporta caching seguro
AC-020	Los cambios de seguridad invalidan decisiones cacheadas cuando corresponda
AC-021	AI no puede ampliar el contexto autorizado
AC-022	Se soporta service-to-service authorization
AC-023	No existe fail-open
AC-024	Se mantiene tenant isolation
AC-025	Las decisiones son observables y trazables
45. Technical Tasks
TASK-IDENTITY-006-01 — Definir AuthorizationService

Crear el contrato central:

AuthorizationService
TASK-IDENTITY-006-02 — Definir AuthorizationRequest

Implementar el modelo de solicitud de autorización.

TASK-IDENTITY-006-03 — Definir AuthorizationDecision

Implementar:

ALLOW
DENY
CHALLENGE
REAUTHENTICATE
REQUIRE_MFA
REVOKE
TASK-IDENTITY-006-04 — Definir AuthorizationContext

Crear el contexto completo de autorización.

TASK-IDENTITY-006-05 — Implementar RBAC Evaluation

Resolver:

User
 ↓
Membership
 ↓
Role
 ↓
Permission
TASK-IDENTITY-006-06 — Implementar Permission Matching

Evaluar:

resource.action

contra los permisos disponibles.

TASK-IDENTITY-006-07 — Implementar ABAC Evaluation

Evaluar atributos de:

User;
Organization;
Membership;
Resource;
Request;
Device;
Session;
Context.
TASK-IDENTITY-006-08 — Implementar Tenant Isolation

Validar:

Actor Organization
        ==
Resource Organization

cuando la operación sea tenant-scoped.

TASK-IDENTITY-006-09 — Implementar Ownership Evaluation

Permitir que las políticas evalúen:

resource.owner_id

contra:

actor.user_id
TASK-IDENTITY-006-10 — Integrar SecurityPolicyService

Conectar:

Authorization
      ↓
Security Policy Engine
TASK-IDENTITY-006-11 — Implementar Decision Precedence

Aplicar:

Regulatory
Platform
Organization
Role
User
Contextual

y:

DENY > ALLOW
TASK-IDENTITY-006-12 — Implementar Fail-Closed

Garantizar que una condición desconocida/no verificable nunca genere ALLOW automáticamente.

TASK-IDENTITY-006-13 — Implementar Authorization Cache Boundary

Crear la abstracción para cache de decisiones.

TASK-IDENTITY-006-14 — Implementar Cache Invalidation

Invalidar decisiones ante cambios relevantes de:

User.
Membership.
Role.
Permission.
Organization.
Security Policy.
TASK-IDENTITY-006-15 — Implementar AI Authorization Boundary

Garantizar que AI Agents no puedan superar el contexto autorizado.

TASK-IDENTITY-006-16 — Implementar Service-to-Service Authorization

Preparar autorización de identidades de servicio.

TASK-IDENTITY-006-17 — Integrar AuditService

Registrar decisiones sensibles y violaciones.

TASK-IDENTITY-006-18 — Implementar Security Events

Crear contratos para:

AuthorizationDenied
TenantAccessDenied
PermissionDenied
PolicyViolation
PrivilegeEscalationDetected
TASK-IDENTITY-006-19 — Crear Unit Tests

Probar:

RBAC.
ABAC.
Ownership.
Tenant isolation.
Decision precedence.
Fail closed.
TASK-IDENTITY-006-20 — Crear Application Tests

Cubrir:

EvaluateAuthorization
CheckPermission
CheckResourceAccess
EvaluatePolicy
TASK-IDENTITY-006-21 — Crear Cross-Tenant Tests

Casos:

Organization A → Organization A = ALLOW
Organization A → Organization B = DENY
TASK-IDENTITY-006-22 — Crear Privilege Escalation Tests

Probar:

role manipulation;
permission escalation;
membership manipulation;
cross-tenant role usage.
TASK-IDENTITY-006-23 — Crear Policy Tests

Probar:

ALLOW
DENY
CHALLENGE
REAUTHENTICATE
REQUIRE_MFA
REVOKE
TASK-IDENTITY-006-24 — Crear Cache Security Tests

Validar que:

stale ALLOW no sobreviva indebidamente;
revocación invalide cache;
tenant context no se mezcle;
cache keys estén correctamente aisladas.
TASK-IDENTITY-006-25 — Crear AI Authorization Tests

Verificar:

Authorized Context
       ⊇
AI Context

y nunca:

AI Context
       ⊃
Authorized Context
TASK-IDENTITY-006-26 — Crear Failure Mode Tests

Probar:

PostgreSQL unavailable;
Redis unavailable;
Policy Engine unavailable;
Authorization dependency unavailable.
TASK-IDENTITY-006-27 — Performance Tests

Validar que Authorization no degrade los objetivos de API:

P95 < 300ms
TASK-IDENTITY-006-28 — Observability Tests

Validar:

metrics;
logs;
traces;
correlation ID;
security events.
TASK-IDENTITY-006-29 — Security Regression Tests

Crear suite permanente para prevenir:

authorization bypass;
tenant escape;
privilege escalation;
fail-open;
cache poisoning;
stale authorization.
TASK-IDENTITY-006-30 — Traceability

Actualizar:

ESP-0001
    ↓
IS-IDENTITY-006
    ↓
TASK-IDENTITY-006-01 ... 030
    ↓
Source Code
    ↓
Tests
46. Definition of Done
 AuthorizationService definido.
 AuthorizationRequest definido.
 AuthorizationDecision definido.
 AuthorizationContext definido.
 RBAC implementado.
 Permission matching implementado.
 ABAC implementado.
 Tenant isolation implementado.
 Ownership evaluation implementado.
 Security Policy Engine integrado.
 Decision precedence implementada.
 Fail-closed implementado.
 Authorization cache definido.
 Cache invalidation implementada.
 AI authorization boundary implementado.
 Service-to-service authorization preparado.
 Audit integrado.
 Security events implementados.
 Unit tests implementados.
 Application tests implementados.
 Cross-tenant tests implementados.
 Privilege escalation tests implementados.
 Policy tests implementados.
 Cache security tests implementados.
 AI authorization tests implementados.
 Failure mode tests implementados.
 Performance tests implementados.
 Observability tests implementados.
 Security regression tests implementados.
 Coverage ≥85% general.
 Coverage ≥95% en áreas críticas de Identity.
 Traceability actualizada.
 Architecture Review aprobado.
 QA Review aprobado.
 Security Review aprobado.
47. Traceability Matrix
Elemento	Referencia
Blueprint	Architecture Map
Domain	Identity
ESP	ESP-0001
Predecessor	IS-IDENTITY-001
Predecessor	IS-IDENTITY-002
Predecessor	IS-IDENTITY-003
Predecessor	IS-IDENTITY-004
Predecessor	IS-IDENTITY-005
Current IS	IS-IDENTITY-006
ADR	ADR-IDENTITY-001
ADR	ADR-IDENTITY-005
ADR	ADR-IDENTITY-006
ADR	ADR-IDENTITY-007
ADR	ADR-IDENTITY-008
ADR	ADR-IDENTITY-010
ADR	ADR-IDENTITY-011
Source	apps/api/app/domains/identity/
Tests	apps/api/tests/identity/

La cadena de trazabilidad de EVOXA exige que las Implementation Stories conecten los requisitos del ESP con las tareas técnicas y posteriormente con código y pruebas.

48. Dependencies
Depends on
IS-IDENTITY-001 — Identity Domain Foundation
IS-IDENTITY-002 — User & Account Management
IS-IDENTITY-003 — Organization & Tenant Management
IS-IDENTITY-004 — Membership Management
IS-IDENTITY-005 — Authentication

ADR-IDENTITY-001 — Identity Architecture
ADR-IDENTITY-005 — Authorization & Policy Engine
ADR-IDENTITY-006 — Persistence Strategy
ADR-IDENTITY-007 — Event & Outbox Strategy
ADR-IDENTITY-008 — Cryptographic Key Management
ADR-IDENTITY-010 — Testing Strategy
ADR-IDENTITY-011 — Deployment & Runtime
Required by
IS-IDENTITY-007 — Role & Permission Management
IS-IDENTITY-008 — Session Management
IS-IDENTITY-009 — Refresh Token Lifecycle
IS-IDENTITY-010 — Multi-Factor Authentication
IS-IDENTITY-011 — Security Policy Engine
IS-IDENTITY-012 — Audit & Security Events
IS-IDENTITY-014 — Identity API Contracts
49. Architectural Model

Con IS-IDENTITY-006 ya queda definido el núcleo conceptual de seguridad:

                         USER
                           │
                           ▼
                      MEMBERSHIP
                           │
                           ▼
                     ORGANIZATION
                           │
                           ▼
                    AUTHENTICATION
                           │
                           ▼
                    SECURITY CONTEXT
                           │
                           ▼
                    AUTHORIZATION
                           │
          ┌────────────────┼────────────────┐
          │                │                │
         RBAC             ABAC          OWNERSHIP
          │                │                │
          └────────────────┼────────────────┘
                           │
                    TENANT ISOLATION
                           │
                           ▼
                 SECURITY POLICY ENGINE
                           │
                           ▼
                       DECISION
                           │
        ┌──────────┬───────┼────────┬────────────┐
        ▼          ▼       ▼        ▼            ▼
      ALLOW      DENY  CHALLENGE  MFA       REAUTHENTICATE

La idea fundamental de EVOXA queda:

Authentication
    ↓
¿Quién eres?

Membership
    ↓
¿En qué organización tienes relación?

RBAC
    ↓
¿Qué permissions tienes por tu rol?

ABAC
    ↓
¿Qué atributos/contexto aplican?

Ownership
    ↓
¿Este recurso te pertenece?

Tenant Isolation
    ↓
¿El recurso pertenece a tu tenant?

Policy Engine
    ↓
¿Existe alguna condición adicional?

Authorization Decision
    ↓
¿ALLOW / DENY / CHALLENGE / MFA / REAUTH?
50. Relación con las próximas IS

La secuencia de Identity ahora queda:

IS-IDENTITY-001
Identity Domain Foundation
        ↓
IS-IDENTITY-002
User & Account Management
        ↓
IS-IDENTITY-003
Organization & Tenant Management
        ↓
IS-IDENTITY-004
Membership Management
        ↓
IS-IDENTITY-005
Authentication
        ↓
IS-IDENTITY-006
Authorization & Permission Evaluation
        ↓
IS-IDENTITY-007
Role & Permission Management
        ↓
IS-IDENTITY-008
Session Management
        ↓
IS-IDENTITY-009
Refresh Token Lifecycle
        ↓
IS-IDENTITY-010
Multi-Factor Authentication
        ↓
IS-IDENTITY-011
Security Policy Engine
        ↓
IS-IDENTITY-012
Audit & Security Events

Esto mantiene la lógica arquitectónica que venimos construyendo: primero se define quién es el usuario, luego dónde pertenece, después cómo se autentica, y ahora cómo se decide lo que puede hacer. La gestión concreta de roles y permisos queda inmediatamente después porque Authorization necesita consumir esos conceptos, mientras que su creación y administración pertenecen a Role & Permission Management.

Estado

IS-IDENTITY-006 — Authorization & Permission Evaluation → Draft

La siguiente es IS-IDENTITY-007 — Role & Permission Management, donde definiremos formalmente Role, Permission, RolePermission, scopes SYSTEM/ORGANIZATION, asignación de roles a Membership y todo el lifecycle de roles y permisos.
