ADR-IDENTITY-005 — Authorization & Policy Engine

Status: Proposed
Version: 1.0.0
Domain: Identity
Decision Type: Architecture / Security
Scope: Authentication, Authorization, RBAC, ABAC, Security Policies, Tenant Isolation
Related: ESP-0001 — Identity
Depends on: ADR-IDENTITY-001, ADR-IDENTITY-002, ADR-IDENTITY-003

1. Context

EVOXA requiere una arquitectura de autorización capaz de controlar el acceso a recursos y operaciones de forma:

segura;
multi-tenant;
auditable;
explicable;
extensible;
consistente entre APIs y servicios;
compatible con RBAC y ABAC;
preparada para futuras políticas basadas en riesgo e IA.

La arquitectura de referencia establece Zero Trust, Least Privilege, Defense in Depth, Secure by Default, Privacy by Design y Auditability como principios fundamentales.

Además, Identity debe proporcionar el contexto de seguridad, mientras que cada dominio de negocio mantiene las reglas específicas de acceso a sus propios recursos.

La autorización, por tanto, no debe quedar reducida a:

if user.role == "admin":
    allow

Debe considerar contexto adicional:

User
Organization
Membership
Role
Permission
Session
Authentication
MFA
Device
Request
Resource
Operation
Risk
2. Problem Statement

EVOXA necesita responder de manera consistente preguntas como:

¿Puede este usuario realizar esta operación sobre este recurso dentro de esta organización y bajo las condiciones de seguridad actuales?

Ejemplos:

¿Puede un Coach visualizar un cliente?
¿Puede modificar un programa de entrenamiento?
¿Puede acceder a información de otra organización?
¿Puede un administrador realizar una operación crítica sin MFA?
¿Puede un usuario suspendido continuar utilizando una sesión existente?
¿Puede un usuario acceder a un recurso que pertenece a otro usuario?
¿Puede una operación de alto riesgo exigir reautenticación?
¿Puede una organización imponer políticas adicionales a sus miembros?

Estas decisiones deben ser determinísticas, auditables y fail-closed.

3. Decision

Se adopta una arquitectura de autorización basada en:

RBAC + ABAC + Tenant Isolation + Resource Ownership + Security Policy Engine

La autorización se dividirá en varias capas.

                    Request
                       │
                       ▼
              Authentication
                       │
                       ▼
               Tenant Context
                       │
                       ▼
                   RBAC
                       │
                       ▼
                   ABAC
                       │
                       ▼
             Resource Ownership
                       │
                       ▼
            Security Policy Engine
                       │
                       ▼
             Authorization Decision
                       │
              ┌────────┴────────┐
              ▼                 ▼
            ALLOW              DENY

La decisión debe ser deny by default y cualquier condición no verificable debe producir una denegación.

4. Authorization Model
4.1 RBAC

RBAC será la primera capa de autorización.

Las organizaciones pueden asignar roles a sus miembros.

Ejemplo:

Organization
 ├── Coach
 ├── Nutritionist
 ├── Physiotherapist
 ├── Team Manager
 └── Admin

Las acciones se expresan mediante permisos:

clients.read
clients.create
clients.update
clients.delete

training.read
training.create
training.update
training.delete

nutrition.read
nutrition.create
nutrition.update
nutrition.publish

El permiso representa:

resource + action
5. Role Scope

Los roles podrán tener diferentes scopes:

SYSTEM
ORGANIZATION
SYSTEM

Aplica a nivel plataforma.

Ejemplo:

Platform Administrator
ORGANIZATION

Aplica únicamente dentro de una organización.

Ejemplo:

Organization Admin
Coach
Nutritionist
Physiotherapist

Un rol de organización no debe otorgar automáticamente acceso a otra organización.

6. ABAC

RBAC no será suficiente para representar todas las reglas de EVOXA.

Se utilizará ABAC para evaluar atributos del contexto.

Ejemplo:

User:
    role = Coach
    organization_id = ORG-001

Resource:
    client.organization_id = ORG-001
    client.owner_id = USER-123

Request:
    action = update

La decisión puede depender de:

role
organization
membership
resource ownership
resource state
authentication method
MFA state
device
IP
risk
operation sensitivity
security policy
7. Tenant Isolation

La organización será considerada el límite principal de tenant.

Una solicitud debe contener o derivar un:

tenant_id

El contexto de autorización debe verificar:

request.tenant_id
        ==
resource.tenant_id

cuando corresponda.

Nunca se debe confiar exclusivamente en un tenant_id enviado por el cliente.

Debe derivarse del contexto autenticado y validarse contra la pertenencia del usuario.

8. Resource Ownership

La pertenencia al tenant no implica automáticamente autorización sobre todos los recursos.

Ejemplo:

Organization
 ├── Coach A
 │    ├── Client 1
 │    └── Client 2
 │
 └── Coach B
      ├── Client 3
      └── Client 4

Un Coach podría tener:

clients.read

pero el dominio puede determinar que solamente puede acceder a:

Client 1
Client 2

y no necesariamente:

Client 3
Client 4

Por esta razón:

Identity determina el contexto de autorización; el dominio de negocio determina las reglas específicas del recurso.

Esto mantiene una separación adecuada de responsabilidades.

9. Authorization Context

Se define conceptualmente:

AuthorizationContext

con información como:

user_id
organization_id
membership_id

roles
permissions

session_id

authentication_method
mfa_authenticated

device_context

request_context

resource_context

operation

risk_context

Ejemplo:

{
  "user_id": "usr_123",
  "organization_id": "org_001",
  "membership_id": "mem_456",
  "roles": ["coach"],
  "permissions": [
    "clients.read",
    "training.read",
    "training.update"
  ],
  "session_id": "ses_789",
  "mfa_authenticated": true,
  "operation": "training.update"
}

Este ejemplo representa un diseño derivado, no un contrato JSON normativo del Blueprint.

10. Policy Engine

Se implementará conceptualmente un:

SecurityPolicyEngine

responsable de evaluar políticas de seguridad adicionales.

Las decisiones posibles serán:

ALLOW
DENY
CHALLENGE
REAUTHENTICATE
REQUIRE_MFA
REVOKE

Ejemplo:

Usuario autenticado
        │
        ▼
¿Tiene permiso?
        │
       Sí
        ▼
¿Pertenece al tenant?
        │
       Sí
        ▼
¿Puede acceder al recurso?
        │
       Sí
        ▼
¿La operación requiere MFA?
        │
      Sí
        ▼
¿MFA válido?
      /   \
    Sí     No
    │       │
  ALLOW   REQUIRE_MFA
11. Policy Precedence

La evaluación seguirá una jerarquía conceptual:

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

Cuando existan conflictos:

DENY prevalece sobre ALLOW.

Por ejemplo:

Role:
    training.update = ALLOW

Security Policy:
    MFA required = TRUE

MFA:
    NOT VERIFIED

Resultado:

REQUIRE_MFA

Otro ejemplo:

Role:
    clients.read = ALLOW

Tenant:
    suspended = TRUE

Resultado:

DENY
12. Policy Lifecycle

Las políticas tendrán un ciclo de vida:

DRAFT
   ↓
ACTIVE
   ↓
INACTIVE
   ↓
ARCHIVED

Cada modificación debe permitir identificar:

policy_id
version
created_by
updated_by
created_at
updated_at
scope
status

Las políticas críticas deben ser auditables.

13. Policy Scope

Las políticas podrán aplicarse conceptualmente a:

Platform
SYSTEM
Organization
ORGANIZATION
Role
ROLE
User
USER
Context
DEVICE
NETWORK
RISK
OPERATION
RESOURCE

No todas estas capacidades necesitan implementarse simultáneamente en la primera versión.

14. Security Policies

El Policy Engine debe poder soportar, como mínimo, políticas relacionadas con:

Authentication
password policy
failed login policy
account lockout
authentication requirements
MFA
MFA required
step-up authentication
high-risk MFA
administrative MFA
Sessions
session lifetime
maximum concurrent sessions
idle timeout
revocation
Authorization
role permissions
tenant restrictions
ownership
sensitive operations
Security
IP restrictions
device restrictions
risk-based authentication
suspicious activity
Administrative operations
require MFA
require reauthentication
require elevated privileges
15. Authorization Decision

Se define conceptualmente:

AuthorizationDecision

con:

decision
reason
policy_id
policy_version
correlation_id

Ejemplo:

{
  "decision": "DENY",
  "reason": "RESOURCE_OUTSIDE_TENANT",
  "policy_id": "tenant-isolation",
  "policy_version": 1,
  "correlation_id": "corr_123"
}

La información de diagnóstico destinada al usuario debe ser limitada para evitar revelar detalles de seguridad.

16. Fail-Closed

El sistema debe utilizar:

Fail Closed

Si el sistema no puede determinar correctamente si una operación está autorizada:

UNKNOWN
   ↓
DENY

Nunca:

UNKNOWN
   ↓
ALLOW

Esto es especialmente importante para:

fallas del Policy Engine;
pérdida de contexto de tenant;
permisos inexistentes;
políticas corruptas;
errores de integración;
información incompleta;
fallas de caché.
17. Policy Caching

Se podrá utilizar caching para mejorar rendimiento.

Ejemplo:

Authorization Request
        │
        ▼
Authorization Cache
      /     \
   HIT       MISS
    │         │
    ▼         ▼
 Decision   Policy Engine

Pero el cache debe tener mecanismos de invalidación cuando cambien:

roles;
permisos;
memberships;
políticas;
suspensión de usuarios;
suspensión de organizaciones.

Nunca se debe permitir que un cache obsoleto produzca una autorización insegura.

18. Domain Authorization

Los dominios de negocio conservarán sus propias reglas.

Ejemplo:

Identity
    ↓
"El usuario tiene training.update"
    ↓
Training Domain
    ↓
"El programa pertenece al cliente que puede administrar"
    ↓
ALLOW / DENY

Esto evita convertir Identity en un "God Service" que conozca todos los detalles de:

Training;
Nutrition;
Recovery;
Assessments;
Billing;
Analytics;
AI.
19. API Integration

La autorización se integrará en el pipeline:

HTTP Request
     ↓
API Gateway
     ↓
Authentication
     ↓
Security Context
     ↓
Authorization
     ↓
Application Layer
     ↓
Domain
     ↓
Infrastructure

El Gateway puede validar el token, pero:

la validación del JWT no equivale a autorización.

El servicio debe seguir verificando permisos y contexto.

20. AI Authorization

La autorización también debe aplicarse a operaciones de IA.

Ejemplo:

Coach
   ↓
AI Recommendation
   ↓
Authorization
   ↓
Client Context
   ↓
AI Service

La IA nunca debe utilizar un contexto que el usuario no podría consultar directamente.

Por tanto:

AI Context Access
    ⊆
User Authorized Access

Esto es especialmente importante para evitar exposición de información entre tenants.

21. Audit

Las decisiones de autorización relevantes deben poder auditarse.

Ejemplos:

AuthorizationAllowed
AuthorizationDenied
AuthorizationPolicyTriggered
MFARequired
ReauthenticationRequired
TenantAccessDenied
ResourceOwnershipDenied

No se deben registrar secretos ni tokens.

Debe existir correlación con:

correlation_id
trace_id
session_id
user_id
tenant_id

cuando sea apropiado.

22. Security Events

El sistema podrá generar eventos como:

AuthorizationDenied
AuthorizationPolicyViolation
MFARequired
ReauthenticationRequired
TenantAccessViolation
SecurityPolicyViolation

Estos eventos pueden ser consumidos por:

Security;
Analytics;
Administration;
Notifications;
Observability;
AI Security.

La arquitectura de eventos de EVOXA establece eventos inmutables, consumidores idempotentes y trazabilidad mediante correlation/causation IDs.

23. Performance

El Authorization Engine debe respetar los objetivos generales de plataforma.

Objetivos relevantes:

API P95 < 300 ms
Authentication P95 < 500 ms

La autorización no debe convertirse en un cuello de botella significativo.

Se favorecerán:

decisiones determinísticas;
consultas eficientes;
índices adecuados;
caching controlado;
contexto de seguridad reutilizable;
evaluación local cuando sea seguro.
24. Security Requirements

Son requisitos no negociables:

Deny by default.
Fail closed.
Least privilege.
Tenant isolation.
No cross-tenant access.
No implicit role escalation.
No secretos en logs.
No tokens en audit events.
Critical authorization decisions auditable.
MFA obligatorio cuando una política lo determine.
Resource ownership validado por el dominio.
Cache invalidation segura.
Todas las decisiones críticas deben ser trazables.
Un fallo del Policy Engine no debe convertirse en ALLOW.
25. Alternatives Considered
Alternative A — RBAC solamente
User → Role → Permission
Rechazada

No cubre suficientemente:

ownership;
tenant isolation;
contexto;
riesgo;
MFA;
operaciones sensibles;
políticas organizacionales.
Alternative B — ABAC solamente
Attributes → Policy → Decision
No seleccionada

Aunque es potente, introduce mayor complejidad y no ofrece una estructura simple para administrar roles organizacionales.

Alternative C — RBAC + ABAC
Seleccionada

Permite:

RBAC
+
ABAC
+
Tenant Isolation
+
Ownership
+
Security Policies

y mantiene una evolución gradual.

Alternative D — Authorization completamente centralizada

Todos los dominios delegarían todas las decisiones a Identity.

Rechazada

Generaría un fuerte acoplamiento y convertiría Identity en conocedor de reglas internas de todos los dominios.

26. Consequences
Positivas
Mayor seguridad.
Tenant isolation explícito.
Menor riesgo de escalamiento de privilegios.
Políticas centralizadas.
Reglas de negocio permanecen en sus dominios.
Auditoría consistente.
Preparación para risk-based authorization.
Preparación para AI Security.
Evolución hacia políticas más sofisticadas.
Negativas
Mayor complejidad inicial.
Necesidad de definir correctamente roles y permisos.
Necesidad de invalidación de caches.
Mayor cantidad de pruebas.
Mayor esfuerzo de observabilidad.
Requiere disciplina para evitar duplicar reglas entre Identity y dominios.
27. Implementation Direction

La estructura prevista para Identity será conceptualmente:

apps/api/app/domains/identity/

├── api/
├── application/
├── domain/
│   ├── authorization/
│   ├── policies/
│   ├── roles/
│   ├── permissions/
│   └── security/
└── infrastructure/
    ├── persistence/
    ├── cache/
    └── policy/

Una posible separación de responsabilidades:

AuthorizationService
        │
        ├── RBAC
        ├── ABAC
        ├── Tenant Policy
        ├── Ownership Context
        └── SecurityPolicyEngine

Esto es dirección de implementación, no una estructura normativa establecida literalmente por el Blueprint.

28. Testing Requirements

Se deberán probar como mínimo:

RBAC
role allowed;
role denied;
missing permission;
multiple roles.
ABAC
attribute match;
attribute mismatch;
ownership;
resource state.
Tenant
same tenant;
different tenant;
missing tenant;
suspended tenant.
Policies
ALLOW;
DENY;
REQUIRE_MFA;
REAUTHENTICATE;
CHALLENGE;
REVOKE.
Security
fail closed;
privilege escalation;
policy bypass;
cache invalidation;
stale authorization;
concurrent authorization changes.
AI
unauthorized AI context;
cross-tenant AI access;
restricted resources;
policy enforcement on AI operations.
29. Traceability
BP-0002 Reference Architecture
        ↓
ESP-0001 Identity
        ↓
ESP-0001/03 Authentication & Authorization
        ↓
ESP-0001/05 Roles & Permissions
        ↓
ESP-0001/08 Security Policies
        ↓
ESP-0001/09 Audit & Compliance
        ↓
ESP-0001/10 API Contracts
        ↓
ESP-0001/12 Events & Integration
        ↓
ESP-0001/13 Testing
        ↓
ADR-IDENTITY-005

La arquitectura de referencia establece explícitamente Identity como dominio responsable de autenticación, autorización, roles, permisos y tenant management, mientras que los dominios mantienen sus responsabilidades propias.

30. Open Decisions

Este ADR deja deliberadamente abiertas algunas decisiones que deberán convertirse en ADRs posteriores:

ADR-IDENTITY-005.1

Permission Model & Naming

Definir formalmente:

resource.action

y catálogo de permisos.

ADR-IDENTITY-005.2

Role Assignment Model

Determinar si una Membership puede tener:

1 role

o:

N roles
ADR-IDENTITY-005.3

Policy Engine Technology

Determinar si se implementará:

custom policy engine

o una solución especializada.

ADR-IDENTITY-005.4

Authorization Cache

Definir:

tecnología;
TTL;
invalidación;
consistency model.
ADR-IDENTITY-005.5

ABAC Policy Language

Definir cómo se expresarán las reglas ABAC.

ADR-IDENTITY-005.6

Risk-Based Authorization

Definir integración futura con:

risk scoring;
device trust;
anomaly detection;
AI security.
31. Final Decision Summary

EVOXA adoptará:

                    AUTHORIZATION
                          │
            ┌─────────────┴─────────────┐
            │                           │
           RBAC                        ABAC
            │                           │
      Roles/Permissions          Context/Attributes
            │                           │
            └─────────────┬─────────────┘
                          │
                  Tenant Isolation
                          │
                  Resource Ownership
                          │
                 Security Policies
                          │
                          ▼
                Policy Decision Engine
                          │
             ┌────────────┼────────────┐
             ▼            ▼            ▼
           ALLOW        DENY       CHALLENGE
                                      │
                               MFA / Reauth

Decisión principal:

EVOXA utilizará RBAC + ABAC + Tenant Isolation + Resource Ownership + Security Policy Engine, bajo un modelo Deny-by-Default y Fail-Closed. Identity proporcionará el contexto de seguridad y autorización, mientras que cada dominio de negocio conservará las reglas específicas de acceso a sus propios recursos.

Estado: PROPOSED, hasta que estas decisiones sean revisadas y aprobadas dentro del proceso formal de ADR del proyecto. Esto es consistente con el ciclo de trazabilidad del Blueprint, donde las decisiones arquitectónicas deben quedar registradas antes de pasar a implementación.
