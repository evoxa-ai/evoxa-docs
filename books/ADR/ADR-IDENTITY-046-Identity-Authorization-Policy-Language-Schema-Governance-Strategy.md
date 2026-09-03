ADR-IDENTITY-046 — Identity Authorization Policy Language, Schema & Governance Strategy

Status: Proposed
Version: 1.0
Date: 2026-09-03
Domain: Identity / Authorization
Priority: Critical
Type: Architecture / Security / Policy

1. Context

ADR-IDENTITY-045 estableció que EVOXA tendrá un modelo centralizado de autorización basado en:

RBAC
+
ABAC
+
Tenant Isolation
+
Resource Ownership
+
Delegation
+
Security Policy Engine

y que las decisiones podrán ser:

ALLOW
DENY
CHALLENGE
REAUTHENTICATE
REQUIRE_MFA
REVOKE

Sin embargo, para que el Authorization Engine pueda evaluar estas condiciones de forma consistente necesitamos definir cómo se representan las políticas.

No es suficiente tener código como:

if user.role == "admin":
    allow()

porque EVOXA deberá manejar políticas mucho más complejas:

User
+
Organization
+
Membership
+
Role
+
Permission
+
Resource
+
Ownership
+
MFA
+
Risk
+
Device
+
Context

Además, las políticas deben poder evolucionar sin tener que modificar constantemente el código de cada dominio.

2. Problem Statement

Se necesita definir un mecanismo que permita expresar:

permisos;
restricciones;
condiciones;
excepciones;
MFA;
step-up;
riesgo;
ownership;
tenant isolation;
delegation;
límites temporales;
restricciones de recursos;
políticas de Organization;
políticas de Platform.

El mecanismo debe ser:

determinista;
versionable;
auditable;
testeable;
seguro;
extensible;
compatible con AI;
independiente del frontend;
independiente del proveedor externo de Identity.
3. Decision

EVOXA adoptará un modelo de:

Structured Authorization Policy Model

Las policies deberán representarse mediante una estructura formal y validable.

La arquitectura conceptual será:

Policy Definition
        ↓
Policy Validation
        ↓
Policy Version
        ↓
Policy Repository
        ↓
Policy Evaluation
        ↓
Authorization Decision

No se permitirá que cada servicio invente su propio formato de autorización.

4. Policy Structure

Una policy tendrá conceptualmente:

Policy
├── id
├── name
├── version
├── status
├── scope
├── priority
├── effect
├── target
├── conditions
├── obligations
├── metadata
├── effective_from
└── effective_until
5. Policy Identity

Cada policy deberá tener un identificador estable.

Ejemplo:

policy_id:
identity.training-plan.update

La versión será independiente:

policy_id:
identity.training-plan.update

version:
3

Esto permite saber exactamente qué versión produjo una decisión.

6. Policy Lifecycle

Las policies utilizarán un lifecycle conceptual:

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

Una policy DRAFT nunca deberá aplicarse a tráfico productivo.

7. Policy Scope

Las policies podrán tener:

PLATFORM
ORGANIZATION
RESOURCE
PLATFORM

Aplica a toda la plataforma.

ORGANIZATION

Aplica a una Organization concreta.

RESOURCE

Aplica a un tipo/recurso específico cuando corresponda.

8. Policy Effect

Las policies podrán expresar efectos como:

ALLOW
DENY
REQUIRE_MFA
REAUTHENTICATE
CHALLENGE
REVOKE

Sin embargo, una policy individual no necesariamente debe poder producir todos los efectos.

El catálogo permitido deberá estar controlado por el Policy Engine.

9. Policy Target

Una policy debe indicar a qué operaciones aplica.

Conceptualmente:

target
├── resource
├── action
├── actor_type
├── organization_scope
└── context

Ejemplo:

resource:
training_plans

action:
update
10. Conditions

Las condiciones utilizarán atributos del contexto.

Ejemplo:

conditions:
    membership.status == ACTIVE
    organization.status == ACTIVE
    role contains Coach
    resource.organization_id == organization.id

Las condiciones deberán ser expresables de manera estructurada y validable.

11. Policy Example

Ejemplo conceptual:

Policy:
training-plan-update

Target:
training_plans.update

Conditions:
    membership.status == ACTIVE
    organization.status == ACTIVE
    permission == training_plans.update
    resource.organization_id == membership.organization_id

Effect:
ALLOW
12. Ownership Policy

Una policy puede incorporar ownership:

Conditions:

permission == training_plans.update
AND
resource.organization_id == membership.organization_id
AND
resource.owner_id == actor.user_id

Esto permite diferenciar:

Can update training plans

de:

Can update any training plan
13. ABAC Attributes

El policy model podrá evaluar atributos de:

Actor
user.id
user.status
membership.id
membership.status
role
department
risk
Organization
organization.id
organization.status
organization.policy
Resource
resource.id
resource.type
resource.organization_id
resource.owner_id
resource.team_id
resource.status
Environment
time
network
device
authentication_method
mfa
risk
14. Attribute Allowlist

No se permitirá que una policy pueda consultar arbitrariamente cualquier campo interno del sistema.

El Policy Engine deberá utilizar un catálogo de atributos permitidos.

Ejemplo:

actor.user_id
actor.organization_id
actor.roles
resource.organization_id
resource.owner_id
request.ip
authentication.mfa
risk.level

Esto limita:

exposición;
complejidad;
accidental data leakage;
dependencia excesiva del modelo interno.
15. Sensitive Attributes

Algunos atributos requerirán controles especiales.

Ejemplos:

health-related data;
financial information;
credentials;
security metadata;
recovery information.

El Policy Engine deberá aplicar minimización de datos.

No se deberá proporcionar al motor más información de la necesaria para tomar la decisión.

16. Policy Composition

Una decisión puede depender de múltiples policies.

Ejemplo:

Platform Security
        +
Organization Security
        +
Role Policy
        +
Resource Policy

El motor deberá combinar estas policies según precedence.

17. Policy Precedence

Se conserva:

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

Una policy inferior no puede debilitar una restricción superior.

18. Explicit Deny

Las policies podrán expresar restricciones explícitas.

Ejemplo:

ALLOW:
Coach → training_plans.read

DENY:
Coach → training_plans.read
IF
resource.classification == RESTRICTED

Resultado:

DENY

cuando ambas sean aplicables.

19. Exceptions

Las excepciones deberán ser explícitas.

No se permitirá una excepción implícita como:

if admin:
    bypass_everything()

Una excepción deberá indicar:

condition
scope
effect
authority
expiration
audit requirement
20. Platform Minimum Security

Una Organization podrá definir restricciones adicionales:

Organization Policy

pero no podrá desactivar controles mínimos de Platform Security.

Ejemplo:

Platform:
Sensitive Export → REQUIRE_MFA

Organization:

Sensitive Export → ALLOW without MFA

Resultado:

REQUIRE_MFA
21. Organization Policy

Las Organizations podrán configurar políticas dentro de límites establecidos por Platform.

Ejemplos:

MFA obligatorio;
horarios;
network restrictions;
device restrictions;
session policies;
resource access;
export restrictions.
22. Policy Variables

Las policies podrán utilizar referencias dinámicas.

Ejemplo:

resource.organization_id
==
actor.organization_id

o:

risk.level <= organization.maximum_risk

Las variables deberán estar tipadas y validadas.

23. Type Safety

Las expresiones deberán respetar tipos.

Ejemplo:

organization.id == resource.organization_id

válido.

Mientras:

organization.id > risk.level

deberá ser inválido.

La validación deberá ocurrir antes de activar una policy.

24. No Arbitrary Code Execution

Las policies no deberán permitir ejecutar código arbitrario.

No se permitirá que una policy contenga:

Python
JavaScript
SQL
Shell

ejecutable directamente.

El modelo deberá utilizar un lenguaje/representación declarativa.

25. SQL Isolation

Una policy no podrá construir SQL arbitrario.

Por ejemplo:

policy → SELECT * FROM ...

no será permitido.

El Policy Engine deberá trabajar con atributos/contexto ya autorizado.

Esto evita convertir el sistema de policies en un mecanismo de acceso directo a la base de datos.

26. Policy Validation

Antes de activar una policy se deberá validar:

sintaxis;
schema;
tipos;
atributos;
resources;
actions;
effects;
precedence;
ciclos;
referencias inexistentes;
condiciones imposibles.
27. Policy Conflict Detection

El sistema debería poder detectar conflictos potenciales.

Ejemplo:

Policy A:
ALLOW

Policy B:
DENY

Same target
Same scope
Same conditions

El sistema deberá determinar si:

el conflicto es intencional;
existe precedence;
requiere revisión.

No se deberá asumir automáticamente que todo conflicto es un error.

28. Policy Shadowing

Una policy puede ser técnicamente válida pero nunca ejecutarse porque otra policy superior siempre la bloquea.

Esto se conoce conceptualmente como:

Policy Shadowing

El sistema debería detectar estas situaciones durante validación/simulación.

29. Policy Simulation

Se deberá soportar conceptualmente:

simulate(policy, authorization_context)

Resultado:

Current:
DENY

Proposed:
ALLOW

La simulación nunca debe modificar:

User;
Membership;
Roles;
Permissions;
Sessions;
Resources.
30. Policy Testing

Cada policy crítica deberá tener casos de prueba.

Ejemplo:

Given:
Coach / Org A / Owner

When:
training_plans.update

Then:
ALLOW

Y:

Given:
Coach / Org A / Non-owner

When:
training_plans.update

Then:
DENY
31. Policy-as-Code vs Policy-as-Data

EVOXA podrá utilizar conceptualmente una combinación:

Policy-as-Data

Para policies configurables por Organization.

Policy-as-Code

Para reglas de seguridad fundamentales de Platform.

Esto permite:

Platform Security
→ Controlled deployment

Organization Policies
→ Controlled configuration

La tecnología concreta queda pendiente.

32. Platform Security Policies

Las políticas críticas de plataforma deberán estar protegidas contra modificaciones arbitrarias.

Ejemplos:

tenant isolation;
MFA minimums;
credential security;
cross-tenant restrictions;
service identity restrictions;
AI Agent restrictions.
33. Organization Configurable Policies

Las Organizations podrán configurar determinadas políticas permitidas.

Pero siempre:

Organization Policy
≤
Platform Security Boundary
34. User Policies

Las policies específicas de User serán una capacidad avanzada y deberán tener menor precedencia.

Ejemplo:

User preference:
"Do not allow exports"

puede restringir el acceso.

Pero no:

User policy:
"Allow me to access another tenant"
35. Delegation Policies

Delegations deberán estar sujetas a policies.

Ejemplo:

ALLOW delegation
IF
delegator.permission == training_plans.update
AND
scope.team == Team 10
AND
expiration <= 7 days

Esto evita delegaciones excesivamente amplias.

36. AI Agent Policies

Las policies deberán poder diferenciar:

actor.type == AI_AGENT

Esto permitirá imponer restricciones adicionales.

Ejemplo:

AI_AGENT
+
export_sensitive_data

→

REQUIRE_MFA

o:

DENY

según policy.

37. Service Account Policies

También se podrán establecer condiciones específicas para:

SERVICE_ACCOUNT

Ejemplo:

service account
+
organization scope
+
specific API
+
specific permission

Esto evita que un Service Account obtenga capacidades humanas innecesarias.

38. Policy Obligations

Una policy podrá devolver obligaciones.

Ejemplo:

ALLOW

Obligation:
audit_sensitive_access

Otro:

ALLOW

Obligation:
mask_sensitive_fields

Otro:

REQUIRE_MFA

Obligation:
reauthorize

Las obligaciones deben ser interpretadas por componentes autorizados.

39. Policy Metadata

Las policies podrán incluir:

owner
description
business_reason
security_classification
created_by
approved_by
created_at
updated_at

La metadata no debe formar parte automáticamente de la lógica de autorización.

40. Policy Governance

Los cambios de policies críticas deberán pasar por:

Draft
 ↓
Validation
 ↓
Simulation
 ↓
Review
 ↓
Approval
 ↓
Activation
 ↓
Monitoring
41. Four-Eyes Principle

Para políticas de alto impacto, EVOXA podrá requerir:

Author
+
Approver

Es decir:

La persona que crea/modifica una policy crítica no necesariamente debe ser quien la aprueba.

El uso obligatorio y alcance de esta regla quedan pendientes.

42. Policy Rollback

Las policies deberán poder revertirse cuando sea posible.

Ejemplo:

v12 ACTIVE
   ↓
v13 ACTIVE
   ↓
incident
   ↓
rollback to v12

El rollback deberá quedar auditado.

43. Effective Dates

Una policy podrá tener:

effective_from
effective_until

Esto permite:

cambios programados;
políticas temporales;
mantenimiento;
excepciones temporales.
44. Temporary Security Policy

Una Organization podría requerir temporalmente:

REQUIRE_MFA

durante un periodo de riesgo elevado.

Ejemplo:

2026-09-01 → 2026-09-15

Después de la expiración, la policy deja de aplicarse automáticamente.

45. Policy Dependency

Una policy puede depender conceptualmente de:

Role
Permission
Organization Policy
MFA
Risk
Resource Attribute

Las dependencias deben ser detectables para evitar activar una policy cuyo requisito no exista.

46. Circular Dependencies

Se evitarán dependencias circulares.

Ejemplo problemático:

Policy A
requires Policy B

Policy B
requires Policy A

El Policy Engine deberá detectar ciclos.

47. Policy Evaluation Determinism

Para un mismo:

Authorization Context
+
Policy Set Version

la evaluación deberá producir la misma decisión.

Conceptualmente:

same context
+
same policy version
=
same decision

salvo atributos externos explícitamente definidos como dinámicos.

48. Dynamic Context

Algunos atributos pueden cambiar rápidamente:

risk;
session state;
MFA evidence;
device state.

Estos deben estar identificados como dinámicos.

El motor no deberá cachear indefinidamente una decisión que dependa de atributos de alta volatilidad.

49. Policy Version Pinning

Las decisiones críticas podrán registrar:

policy_set_version

para permitir reconstrucción posterior.

Esto es especialmente importante para:

audit;
compliance;
incident response;
debugging.
50. Authorization Explainability

El motor deberá poder producir:

decision
reason
policy_id
policy_version

y opcionalmente:

matched_rules
obligations

Los detalles internos estarán restringidos a operadores autorizados.

51. Privacy

El Policy Engine deberá utilizar el mínimo contexto necesario.

Por ejemplo, una policy que solamente requiere:

membership.status

no necesita recibir:

full user profile

Esto reduce exposición de datos.

52. Secrets

Nunca deberán formar parte del contexto de policy:

password;
password hash;
refresh token;
access token;
TOTP secret;
recovery code;
private key;
KMS credentials.
53. Policy Distribution

En una arquitectura distribuida:

Policy Repository
        ↓
Policy Distribution
        ↓
Policy Engine Instances

Las instancias deberán saber qué versión de policy tienen cargada.

No se debe asumir que todas las instancias actualizan exactamente al mismo milisegundo.

54. Policy Consistency

Para políticas de seguridad críticas deberá existir una estrategia para evitar que una instancia antigua continúe autorizando operaciones después de un cambio crítico.

Se deberá definir posteriormente:

propagation guarantees;
version checks;
cache invalidation;
emergency policy activation.
55. Performance

El Policy Engine debe cumplir los objetivos generales de Identity:

API P95 < 300ms
Authentication P95 < 500ms

La evaluación de policy no deberá convertirse en el principal cuello de botella.

Se podrán utilizar:

precompilation;
caching;
indexing;
local evaluation;
immutable policy versions.
56. High Availability

El Policy Engine deberá soportar múltiples instancias cuando corresponda.

PDP-1
PDP-2
PDP-3

Una caída de una instancia no deberá modificar las garantías de seguridad.

57. Fail Closed

Si una policy crítica no puede ser evaluada correctamente:

PDP failure
      ↓
Critical operation
      ↓
DENY

Nunca:

PDP failure
      ↓
ALLOW
58. AI Policy Assistance

AI podrá analizar:

policies;
conflictos;
redundancias;
shadowing;
posibles privilege escalations;
escenarios de prueba.

AI no podrá activar directamente una policy crítica sin pasar por el lifecycle de governance.

59. Audit

Los cambios de policy deberán registrar:

policy_id
old_version
new_version
actor
approver
scope
reason
timestamp
correlation_id

No se deberán registrar secretos.

60. Security Events

Podrán generarse Security Events para:

policy tampering;
unauthorized policy change;
failed policy activation;
unexpected policy version;
cross-tenant policy;
privilege escalation policy;
invalid policy;
emergency rollback.
61. Persistence Model — Conceptual

Se consideran entidades:

SecurityPolicy
SecurityPolicyVersion
PolicyRule
PolicyTarget
PolicyCondition
PolicyObligation
PolicyAudit

No necesariamente todas serán tablas independientes.

El schema definitivo se establecerá en los ADR de persistence.

62. API Model — Conceptual

Podrían existir endpoints como:

GET    /api/v1/authorization/policies
GET    /api/v1/authorization/policies/{id}
POST   /api/v1/authorization/policies
PUT    /api/v1/authorization/policies/{id}
POST   /api/v1/authorization/policies/{id}/validate
POST   /api/v1/authorization/policies/{id}/simulate
POST   /api/v1/authorization/policies/{id}/activate
POST   /api/v1/authorization/policies/{id}/rollback

Estos son contratos conceptuales, no API definitivas.

63. Testing Requirements

Se deberán probar:

Policy Syntax
schema válido;
schema inválido;
tipos incorrectos.
Policy Logic
ALLOW;
DENY;
conflicting rules;
precedence.
Tenant
cross-tenant deny;
organization scope.
Roles
role grant;
role removal.
Ownership
owner;
non-owner.
Delegation
valid;
expired;
revoked;
privilege amplification.
MFA
required;
satisfied;
expired evidence.
Risk
low;
medium;
high;
critical.
Versioning
old policy;
new policy;
rollback.
Failure
PDP unavailable;
repository unavailable;
invalid policy;
cache stale.
AI
AI policy suggestion;
AI cannot bypass approval;
AI Agent scope isolation.
64. Consequences
Positivas
Modelo de policies formal.
Versionado.
Governance.
Simulación.
Testing.
Explainability.
Compatibilidad con RBAC/ABAC.
Integración con MFA.
Integración con AI.
Multi-tenant seguro.
Menor lógica de autorización duplicada.
Negativas
Mayor complejidad.
Necesidad de Policy Repository.
Necesidad de lifecycle.
Mayor esfuerzo de testing.
Necesidad de governance.
Requiere una definición formal del lenguaje.
65. Alternatives Considered
A. Hard-coded authorization

Rejected.

No escala con la complejidad futura de EVOXA.

B. RBAC configuration only

Rejected.

No permite condiciones contextuales suficientes.

C. Arbitrary scripting

Rejected.

Introduce riesgo de ejecución de código y dificulta governance.

D. Un proveedor externo obligatorio de Policy Engine

Deferred.

La arquitectura debe permanecer independiente del proveedor.

E. Un DSL propietario desde el inicio

Deferred.

Primero debe definirse el modelo semántico; la tecnología concreta puede determinarse posteriormente.

66. Architectural Invariants
I1:
Policies are versioned.

I2:
Policies are validated before activation.

I3:
Production cannot execute DRAFT policies.

I4:
Platform security policies cannot be weakened by Organization policies.

I5:
DENY takes precedence over conflicting ALLOW.

I6:
Policies cannot execute arbitrary code.

I7:
Policies cannot directly execute arbitrary SQL.

I8:
Policy evaluation is deterministic for the same context and policy version.

I9:
Critical policy failures fail closed.

I10:
Policy changes are auditable.

I11:
Policy evaluation uses minimum necessary context.

I12:
Secrets never enter policy context.

I13:
AI cannot bypass policy governance.

I14:
Policy scope cannot implicitly cross tenants.

I15:
Authorization decisions can be reconstructed from policy version and context metadata.
67. Traceability
Previous ADRs
ADR-IDENTITY-005 — Authorization & Permission Evaluation
ADR-IDENTITY-021 — Controlled Event Replay
ADR-IDENTITY-026 — API Pagination, Filtering & Idempotency
ADR-IDENTITY-027 — Runtime Configuration, Secrets & KMS
ADR-IDENTITY-028 — Observability & Operational Security
ADR-IDENTITY-033 — Account Lifecycle
ADR-IDENTITY-040 — Federation
ADR-IDENTITY-041 — Provisioning
ADR-IDENTITY-042 — Source of Truth
ADR-IDENTITY-043 — Multi-Organization
ADR-IDENTITY-044 — Membership Roles & Delegation
ADR-IDENTITY-045 — Authorization Policy Evaluation
68. Pending Decisions

Antes de implementación deberán definirse:

Policy DSL.
JSON/YAML representation.
Policy schema definitivo.
Policy expression language.
Policy engine technology.
Policy repository.
Policy compiler.
Policy evaluator.
Rule precedence formal.
Conflict resolution algorithm.
Attribute catalog.
Attribute types.
Policy inheritance.
Policy composition.
Policy simulation API.
Policy testing framework.
Policy approval workflow.
Four-eyes requirements.
Policy propagation.
Cache strategy.
Cache invalidation.
Policy distribution.
Emergency policy activation.
Rollback strategy.
Reason-code catalog.
Obligation model.
Explainability levels.
AI policy analysis.
AI policy approval boundaries.
Platform vs Organization policy catalog.
Cross-tenant policy constraints.
Performance benchmarks.
69. Acceptance Criteria

El ADR podrá considerarse aceptado cuando:

 Exista un modelo formal de Policy.
 Exista policy lifecycle.
 Exista policy versioning.
 Exista schema validation.
 Exista policy scope.
 Exista policy precedence.
 Exista explicit DENY.
 Exista soporte para ALLOW/DENY/MFA/REAUTH/CHALLENGE/REVOKE.
 Exista soporte para RBAC.
 Exista soporte para ABAC.
 Exista soporte para Ownership.
 Exista soporte para Delegation.
 Exista tenant isolation.
 Exista attribute allowlist.
 No exista arbitrary code execution.
 No exista arbitrary SQL execution.
 Exista policy simulation.
 Exista policy testing.
 Exista policy governance.
 Exista audit trail.
 Exista rollback.
 Exista fail-closed.
 AI no pueda activar policies críticas sin governance.
 Se preserve performance objetivo.
 Se puedan reconstruir decisiones históricas.
70. Architectural Outcome

Con ADR-045 + ADR-046, la arquitectura ya tiene tanto el motor de decisión como el modelo que alimenta ese motor:

                    ┌──────────────────────┐
                    │   POLICY AUTHORING   │
                    │ Admin / Governance   │
                    └──────────┬───────────┘
                               │
                               ▼
                    ┌──────────────────────┐
                    │ POLICY VALIDATION     │
                    └──────────┬───────────┘
                               │
                               ▼
                    ┌──────────────────────┐
                    │ POLICY VERSION       │
                    │ + SCOPE + RULES      │
                    └──────────┬───────────┘
                               │
                               ▼
                    ┌──────────────────────┐
                    │ POLICY REPOSITORY    │
                    └──────────┬───────────┘
                               │
                               ▼
┌──────────────┐      ┌──────────────────────┐
│ Authorization│─────▶│ POLICY DECISION      │
│ Request      │      │ POINT (PDP)          │
└──────────────┘      └──────────┬───────────┘
                                 │
                    ┌────────────┼────────────┐
                    ▼            ▼            ▼
                  RBAC         ABAC       Delegation
                    │            │            │
                    └────────────┼────────────┘
                                 ▼
                         Tenant / Ownership
                                 │
                                 ▼
                         Security Policies
                                 │
                                 ▼
                         Final Decision
                                 │
                  ┌──────────────┼──────────────┐
                  ▼              ▼              ▼
                ALLOW          DENY        REQUIRE_MFA

La regla principal queda:

Las políticas de autorización de EVOXA serán declarativas, versionadas, validadas, gobernadas y evaluadas mediante un motor centralizado; ningún dominio podrá crear atajos que permitan saltarse Tenant Isolation, Membership, RBAC, ABAC, Ownership, Delegation o las Platform Security Policies.

Siguiente paso

El siguiente ADR natural es:

ADR-IDENTITY-047 — Identity Authorization Policy Engine Technology & Implementation Strategy

Ahí ya podemos pasar de la arquitectura conceptual a decidir cómo implementaremos técnicamente el Policy Engine, incluyendo si conviene un DSL, JSON-based policies, motor embebido, servicio PDP separado, integración con FastAPI/Python, caching y cómo encajará en la estructura de apps/api/app/domains/identity/.
