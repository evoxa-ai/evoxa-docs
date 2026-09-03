ADR-IDENTITY-049 — Authorization Policy Evaluation Runtime & Decision Algorithm

Status: Proposed
Version: 1.0
Date: 2026-09-03
Domain: Identity / Authorization
Priority: Critical
Type: Architecture / Security / Runtime

1. Context

ADR-IDENTITY-045 definió el Authorization Policy Engine como el componente central encargado de producir decisiones de autorización.

ADR-IDENTITY-046 definió el modelo estructurado de políticas y su gobernanza.

ADR-IDENTITY-047 definió la estrategia tecnológica del Policy Engine:

PEP/PDP conceptualmente separados.
Implementación inicial dentro de Identity.
PostgreSQL como fuente durable.
Redis como cache.
Evaluación determinista.
Fail-closed.
Posibilidad futura de evolucionar hacia un PDP independiente.

ADR-IDENTITY-048 definió un DSL declarativo, estructurado y tipado, sin ejecución arbitraria de código.

Por lo tanto, este ADR define cómo el runtime del PDP procesará una solicitud de autorización y cómo llegará a la decisión final.

El Blueprint establece que el pipeline de autorización comprende autenticación, resolución de roles, evaluación de políticas y comprobación de permisos antes de la ejecución.

Además, las políticas deben considerar contexto como role, organization, tenant, ownership, device trust, risk, session state y restricciones temporales.

2. Problem Statement

No es suficiente con definir un lenguaje de políticas.

EVOXA necesita establecer una semántica precisa para responder:

¿Cómo se transforma un Authorization Request en una Authorization Decision?

El runtime debe determinar:

qué políticas son aplicables;
qué contexto puede utilizarse;
cómo se evalúan las condiciones;
cómo interactúan RBAC y ABAC;
cómo se aplica tenant isolation;
cómo se verifica ownership;
cómo se procesa delegation;
cómo se resuelven conflictos;
cuándo se requiere MFA;
cuándo se requiere reautenticación;
qué ocurre ante errores;
cómo se producen obligations;
cómo se audita la decisión.
3. Decision

EVOXA adoptará un Deterministic Authorization Evaluation Runtime.

El algoritmo general será:

Authorization Request
        ↓
Request Validation
        ↓
Actor / Authentication Validation
        ↓
Tenant Context Validation
        ↓
Authorization Context Construction
        ↓
Policy Selection
        ↓
Policy Evaluation
        ↓
Mandatory Security Guards
        ↓
RBAC Evaluation
        ↓
Ownership Evaluation
        ↓
ABAC Evaluation
        ↓
Delegation Evaluation
        ↓
Policy Composition
        ↓
Conflict Resolution
        ↓
Effect Resolution
        ↓
Obligation Resolution
        ↓
Authorization Decision
        ↓
Audit / Security Telemetry

La evaluación deberá ser:

determinista;
segura;
reproducible;
fail-closed;
side-effect free;
tenant-aware;
version-aware.
4. Runtime Components

El runtime estará compuesto conceptualmente por:

AuthorizationRuntime
│
├── RequestValidator
├── ContextBuilder
├── TenantGuard
├── PolicyMatcher
├── PolicyEvaluator
├── RBACEvaluator
├── OwnershipEvaluator
├── ABACEvaluator
├── DelegationEvaluator
├── ConflictResolver
├── EffectResolver
├── ObligationResolver
├── DecisionBuilder
└── AuditEmitter

No necesariamente cada componente será una clase independiente en la primera implementación.

La separación es principalmente responsabilidad arquitectónica.

5. Authorization Request

El runtime recibirá una solicitud conceptual:

AuthorizationRequest

actor
organization
action
resource
request_context
session_context
delegation_context

Ejemplo:

{
  "actor": {
    "id": "user-123",
    "type": "USER"
  },
  "organization_id": "org-001",
  "action": "training_session.update",
  "resource": {
    "type": "training_session",
    "id": "training-456"
  }
}

Este objeto representa una intención de autorización, no una ejecución.

6. Step 1 — Request Validation

El runtime validará:

actor;
action;
resource;
organization;
request context;
session;
delegation, si existe.

Se rechazarán solicitudes:

incompletas;
malformadas;
con acciones desconocidas;
con recursos desconocidos;
con tenant ambiguo.

Resultado:

INVALID REQUEST
        ↓
DENY
7. Step 2 — Authentication Validation

El Policy Engine no sustituye Authentication.

Primero debe existir evidencia válida de autenticación.

Se comprobarán, según corresponda:

Identity
Session
Authentication Method
Authentication Assurance
MFA State
Account State
Membership State

Una identidad:

SUSPENDED
DISABLED

no deberá recibir un ALLOW normal simplemente porque exista un JWT válido.

8. Step 3 — Tenant Context Validation

Tenant isolation será un mandatory security guard.

El runtime deberá determinar:

Actor Organization
        =
Active Organization Context
        =
Resource Organization

cuando la operación sea organization-scoped.

Si existe:

ORG-A ≠ ORG-B

resultado:

DENY

por defecto.

El Blueprint establece que el acceso cross-tenant está prohibido salvo autorización explícita.

9. Cross-Tenant Exception

Las operaciones legítimas cross-tenant deberán ser explícitas.

Ejemplo:

actor.type = SYSTEM_SERVICE
operation = platform_support
scope = PLATFORM

No será suficiente:

role = ADMIN

para obtener automáticamente acceso global.

La excepción deberá estar respaldada por:

scope;
permission;
policy;
authorization context;
audit.
10. Step 4 — Authorization Context Construction

El runtime construirá el contexto:

AuthorizationContext

utilizando únicamente los atributos necesarios.

Ejemplo:

subject
organization
membership
roles
permissions
session
mfa
resource
request
environment
delegation
risk

El contexto será inmutable durante la evaluación.

11. Immutable Evaluation Context

Una vez iniciado el proceso:

AuthorizationContext

no deberá modificarse arbitrariamente durante la evaluación.

Esto permite:

determinismo;
reproducibilidad;
debugging;
auditoría;
testing.
12. Step 5 — Policy Selection

El runtime no evaluará todas las políticas existentes.

Primero realizará Policy Matching.

Criterios:

Policy Status
Policy Scope
Resource Type
Action
Actor Type
Organization
Effective Period

Ejemplo:

100,000 policies stored
        ↓
Policy Index
        ↓
Relevant policies
        ↓
15 policies
        ↓
Evaluation
13. Active Policy Only

Solamente se podrán evaluar políticas:

ACTIVE

y dentro de su periodo de efectividad.

No deberán utilizarse:

DRAFT
VALIDATING
APPROVED
SUPERSEDED
RETIRED

como políticas activas de autorización.

14. Policy Version Resolution

Cada evaluación deberá resolver una versión concreta.

Policy ID
    ↓
Active Version
    ↓
Compiled Representation
    ↓
Evaluation

La decisión deberá poder identificar:

policy_id
policy_version
dsl_version
schema_version
15. Step 6 — Policy Evaluation

Cada policy seleccionada será evaluada contra el contexto.

Conceptualmente:

evaluate(policy, context)
        ↓
target matches?
        ↓
conditions evaluate?
        ↓
effect

Resultados internos posibles:

MATCH
NO_MATCH
INDETERMINATE
ERROR
16. Target Evaluation

Primero se determina si el target coincide.

Ejemplo:

Policy:
resource = training_session
action = update

Request:
resource = training_session
action = update

→ MATCH

Si:

resource = nutrition_plan

entonces:

→ NO_MATCH
17. Condition Evaluation

Si el target coincide, se evalúan las condiciones.

Ejemplo:

subject.role == COACH
AND
resource.organization_id == organization.id
AND
resource.owner_id == subject.id

Cada expresión produce:

TRUE
FALSE
INDETERMINATE
18. Three-Valued Evaluation

El runtime utilizará conceptualmente tres estados:

TRUE
FALSE
UNKNOWN

Esto es importante cuando falta un atributo.

Ejemplo:

resource.owner_id

no disponible.

El runtime no deberá asumir:

UNKNOWN = TRUE

Para condiciones críticas, UNKNOWN deberá producir una decisión segura.

19. Fail-Closed Semantics

La regla fundamental será:

UNKNOWN
   ↓
NOT ALLOW

y:

ERROR
   ↓
NOT ALLOW

El Blueprint establece secure defaults y protección contra Broken Access Control.

20. Step 7 — Mandatory Security Guards

Antes de permitir un ALLOW, el runtime deberá comprobar los controles obligatorios.

Conceptualmente:

Account Active
      ↓
Membership Active
      ↓
Tenant Valid
      ↓
Session Valid
      ↓
Policy Valid
      ↓
Authorization

Un guard obligatorio puede producir:

DENY
REQUIRE_MFA
REAUTHENTICATE
REVOKE
21. RBAC Evaluation

RBAC determina las capacidades derivadas de los roles.

Ejemplo:

COACH
  ↓
training_session.update

Pero RBAC por sí solo no garantiza ALLOW.

Después deberán evaluarse:

tenant;
ownership;
ABAC;
security policies;
delegation;
session/risk.
22. Permission Resolution

El runtime podrá resolver:

Role
 ↓
Role Permissions
 ↓
Permission

Ejemplo:

COACH
 ↓
training_session.update

Si la permission no existe:

DENY
23. Ownership Evaluation

Ownership se evaluará cuando sea aplicable.

Ejemplo:

resource.owner_id == subject.id

Resultado:

TRUE  → continue
FALSE → DENY

Un ownership match no debe superar una prohibición superior.

24. ABAC Evaluation

ABAC evaluará atributos del:

Subject
Resource
Environment
Request
Session

Ejemplo:

subject.role == COACH
AND
environment.risk_level <= MEDIUM
AND
resource.classification != HIGHLY_SENSITIVE
25. Delegation Evaluation

Cuando exista delegation:

Delegation
 ↓
Valid?
 ↓
Active?
 ↓
Not expired?
 ↓
Correct scope?
 ↓
Delegator authorized?
 ↓
No privilege amplification?

Si cualquiera de los controles críticos falla:

DENY
26. Delegation Privilege Boundary

El runtime deberá garantizar:

Delegated Capability
        ≤
Delegator Effective Capability

Esto evita:

Coach
 ↓
Delegates
 ↓
SUPER_ADMIN

aunque la policy de delegation esté mal configurada.

27. Policy Composition

Después de evaluar las políticas aplicables, el runtime tendrá un conjunto:

PolicyResult[]

Policy A → ALLOW
Policy B → ALLOW
Policy C → DENY
Policy D → NO_MATCH

Estos resultados deberán combinarse mediante el algoritmo de precedence.

28. Conflict Resolution

Regla principal:

DENY
   >
ALLOW

cuando ambos sean aplicables y tengan el mismo nivel de autoridad.

Ejemplo:

Policy A → ALLOW
Policy B → DENY

Final → DENY

Esto proporciona una postura segura por defecto.

29. Policy Precedence

La precedencia arquitectónica será:

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

Una política inferior no podrá debilitar una restricción superior.

Ejemplo:

Platform:
DENY sensitive operation

Organization:
ALLOW sensitive operation

Resultado:

DENY
30. Security Minimums

Las políticas organizacionales no podrán reducir los mínimos de seguridad establecidos por la plataforma.

Ejemplo:

Platform minimum:
MFA required for sensitive operation

Una organización no podrá configurar:

MFA disabled

para esa operación.

31. Effect Resolution

Una vez compuestos los resultados:

Policy Results
      ↓
Precedence
      ↓
Conflict Resolution
      ↓
Effect Resolution

El runtime producirá uno de:

ALLOW
DENY
CHALLENGE
REAUTHENTICATE
REQUIRE_MFA
REVOKE
32. Step-Up Decisions

Si una operación requiere mayor assurance:

Current Assurance
       ↓
Required Assurance
       ↓
Insufficient
       ↓
REQUIRE_MFA

El runtime no ejecutará MFA directamente.

Devolverá:

REQUIRE_MFA

al Application Layer.

33. Reauthentication

Para operaciones extremadamente sensibles:

session.age > allowed_age

puede producir:

REAUTHENTICATE

Esto obliga a renovar la evidencia de autenticación antes de continuar.

34. Challenge

CHALLENGE será utilizado para situaciones donde la policy requiera una prueba adicional.

Ejemplo:

Risk = elevated
       ↓
CHALLENGE

La naturaleza concreta del challenge será responsabilidad del servicio correspondiente.

35. REVOKE

REVOKE tendrá un tratamiento especial.

Podrá utilizarse para indicar que una condición de seguridad exige revocar:

session;
refresh token family;
factor;
credential;
access capability.

El Policy Engine no deberá ejecutar directamente todas esas operaciones.

Emitirá una decisión/obligation controlada.

36. Obligations

Después de determinar el effect:

Decision
   ↓
Obligations

Ejemplos:

AUDIT
REQUIRE_STEP_UP
REQUIRE_MFA
MASK_RESPONSE
REVOKE_SESSION

Las obligations deberán tener un catálogo controlado.

37. Obligation Enforcement

Una obligation no podrá ser simplemente ignorada.

Ejemplo:

Decision:
ALLOW

Obligation:
MASK_SENSITIVE_FIELDS

La capa responsable deberá cumplirla antes de entregar el resultado al consumidor.

Si una obligation crítica no puede cumplirse:

DENY / INDETERMINATE

según la política.

38. Final Authorization Decision

La decisión final será estructurada:

AuthorizationDecision

decision
reason_code
policy_ids
policy_versions
obligations
authorization_context_reference
evaluation_metadata

No contendrá:

tokens;
passwords;
secrets;
MFA secrets;
recovery codes;
información sensible innecesaria.
39. Reason Codes

El runtime utilizará códigos normalizados.

Ejemplos:

AUTHENTICATION_REQUIRED
ACCOUNT_SUSPENDED
MEMBERSHIP_INACTIVE
TENANT_MISMATCH
INSUFFICIENT_PERMISSION
OWNERSHIP_REQUIRED
POLICY_DENIED
MFA_REQUIRED
REAUTHENTICATION_REQUIRED
DELEGATION_INVALID
POLICY_EVALUATION_ERROR

Los reason codes serán estables y aptos para APIs/auditoría.

40. Internal vs External Decision

El runtime podrá generar una decisión interna detallada:

DENY
policy_123
version_7
rule_4
condition_3
tenant_mismatch

Pero la API pública podrá devolver:

403
INSUFFICIENT_PERMISSION

La explicación detallada permanecerá protegida.

41. Deterministic Decision Algorithm

El algoritmo conceptual será:

authorize(request):

    validate(request)

    validate_authentication(request)

    validate_tenant(request)

    context = build_context(request)

    policies = select_applicable_policies(context)

    results = []

    for policy in policies:

        result = evaluate(policy, context)

        results.append(result)

    security_result = evaluate_mandatory_security_guards(context)

    combined = combine(results, security_result)

    decision = resolve_effect(combined)

    obligations = resolve_obligations(combined)

    return build_decision(
        decision,
        obligations,
        policy_versions,
        reason_codes
    )

Este pseudocódigo es arquitectural, no código de implementación.

42. Short-Circuit Evaluation

El runtime podrá utilizar short-circuit cuando sea seguro.

Ejemplo:

Mandatory Tenant DENY
       ↓
No need to evaluate lower policies
       ↓
DENY

Pero el short-circuit no deberá impedir:

auditoría necesaria;
security telemetry;
detección de incidentes;
obligaciones obligatorias.
43. Policy Ordering

Las políticas deberán evaluarse siguiendo un orden que permita aplicar primero las restricciones más importantes.

Conceptualmente:

Security Guards
      ↓
Regulatory
      ↓
Platform Security
      ↓
Organization
      ↓
Role
      ↓
Context

Esto también puede reducir el coste de evaluación.

44. Policy Cache Integration

El runtime utilizará políticas compiladas/cacheadas cuando estén disponibles:

Request
 ↓
Policy Cache
 ↓
Compiled Policies
 ↓
Evaluation

Si no están disponibles:

Cache Miss
 ↓
Policy Repository
 ↓
Compile
 ↓
Cache
 ↓
Evaluation

El parsing/compilation no debería ejecutarse repetidamente en cada request.

45. Cache Safety

La cache nunca podrá:

cruzar tenants;
utilizar políticas retiradas;
ignorar revocaciones;
mantener decisiones críticas indefinidamente.

Las claves deberán incorporar contexto suficiente para impedir contaminación cross-tenant.

46. Authorization Decision Cache

Se podrá utilizar cache de decisiones para operaciones seguras y de corta duración.

Pero deberán invalidarse ante cambios relevantes:

Role changed
Membership changed
User suspended
Session revoked
Policy changed
Delegation revoked
MFA state changed
Risk changed
47. Concurrency

El runtime deberá soportar múltiples evaluaciones simultáneas.

No deberá mantener estado mutable global.

Preferido:

Request A → Context A → Evaluation A
Request B → Context B → Evaluation B
Request C → Context C → Evaluation C

No:

Global mutable authorization state
48. Runtime Failure

Si ocurre:

Parser failure
Policy corruption
Repository failure
Unexpected evaluator exception
Context resolution failure

el resultado no podrá convertirse en:

ALLOW

Debe producir:

DENY

o:

INDETERMINATE

que el PEP tratará de manera fail-closed.

49. Availability vs Security

El sistema priorizará seguridad sobre disponibilidad para decisiones críticas.

Security-critical authorization
        ↓
Security > Availability

Esto significa que una caída parcial no debe provocar:

Policy unavailable
      ↓
ALLOW everything
50. Business Rule Boundary

El Policy Runtime no evaluará reglas de negocio que no pertenezcan a Identity.

Ejemplo:

"Coach puede editar entrenamiento"

→ Identity Authorization.

Pero:

"Training cannot be edited 24h before session"

→ Training Domain.

Esto mantiene la autonomía de los módulos. El Blueprint establece que los módulos colaboran mediante APIs y eventos mientras mantienen autonomía.

51. Audit Integration

Cada decisión relevante podrá generar audit/security telemetry con:

actor
tenant
organization
action
resource
decision
reason_code
policy_id
policy_version
timestamp
correlation_id

Nunca:

password
token
MFA secret
private key
recovery code

El Blueprint exige que las acciones de seguridad sean auditables y que los registros sean inmutables.

52. Explainability

El runtime deberá conservar suficiente metadata para responder:

¿Por qué se tomó esta decisión?

Ejemplo interno:

Decision: DENY

Reason:
TENANT_MISMATCH

Policy:
training-session-update

Policy Version:
4

Failed condition:
resource.organization_id == organization.id

La explicación externa deberá estar limitada por seguridad.

53. Replay & Determinism

La arquitectura permitirá reconstruir una decisión utilizando:

Policy Version
+
Authorization Context
+
Runtime Version

Esto será importante para:

debugging;
auditoría;
incident response;
testing;
compliance.

Los contextos históricos deberán tratarse cuidadosamente y no almacenar datos sensibles innecesarios.

54. Runtime Versioning

Las decisiones críticas podrán identificar:

runtime_version
policy_version
dsl_version
schema_version

Esto permitirá determinar si un cambio del evaluator modificó el comportamiento.

55. Performance

El runtime deberá contribuir al objetivo del Blueprint:

Métrica	Objetivo
Authorization	<50 ms
Permission Resolution	<30 ms

Para ello:

políticas compiladas;
cache;
evaluación local;
índices;
context pre-resolved;
short-circuit seguro;
ausencia de network calls durante evaluación.
56. Security Invariants

El runtime deberá garantizar siempre:

No Authentication
      → DENY

Invalid Tenant
      → DENY

Unknown Permission
      → DENY

Unknown Policy
      → DENY

Policy Error
      → DENY

Indeterminate
      → DENY

Explicit Security DENY
      → DENY

Expired Delegation
      → DENY

Y nunca:

ERROR → ALLOW
UNKNOWN → ALLOW
TENANT MISMATCH → ALLOW
57. Testing Strategy

El algoritmo deberá probarse mediante:

Unit Tests
operator evaluation;
conditions;
precedence;
effects;
obligations.
Integration Tests
PostgreSQL;
Redis;
Policy Repository;
Identity context.
Security Tests
privilege escalation;
tenant escape;
policy bypass;
malformed policy;
missing context;
stale cache.
Concurrency Tests
simultaneous policy updates;
simultaneous authorization requests;
cache invalidation races.
Performance Tests
P50;
P95;
P99;
high policy count;
high concurrency.
Regression Tests

Todas las políticas críticas deberán tener casos de regresión.

58. Critical Security Test Cases

Obligatoriamente:

User from Org A
→ Access resource Org B
→ DENY
Suspended user
→ Update resource
→ DENY
Expired delegation
→ Action
→ DENY
Sensitive operation
+
Insufficient MFA
→ REQUIRE_MFA
Policy evaluator error
→ DENY
Unknown attribute
→ Policy invalid / DENY
59. Alternatives Considered
A. First-Match-Wins

Rejected.

Puede ocultar conflictos y generar comportamientos inseguros.

B. ALLOW-Wins

Rejected.

Es incompatible con una postura de seguridad fail-closed.

C. DENY-Wins without Precedence

Insufficient.

Aunque es seguro, no representa adecuadamente:

regulatory policies;
platform policies;
organization policies;
role policies;
contextual policies.

Se utilizará DENY-wins dentro del mismo nivel, combinado con precedence jerárquica.

D. Business Domains Evaluate Their Own Policies

Rejected.

Rompe la centralización de Identity Authorization.

E. External PDP for Every Request

Not adopted initially.

Podrá introducirse posteriormente si las necesidades de escala justifican separar el PDP.

60. Consequences
Positive
Algoritmo de autorización definido.
Comportamiento determinista.
Seguridad fail-closed.
Tenant isolation reforzado.
RBAC/ABAC/ownership/delegation integrados.
MFA y risk integrables.
Auditoría reproducible.
Cache segura.
Evolución hacia PDP independiente.
Menor riesgo de authorization bypass.
Negative
Mayor complejidad del runtime.
Necesidad de mantener semántica estable.
Cache invalidation compleja.
Testing extenso.
Requiere fuerte governance.
Cambios del evaluator pueden tener impacto global.
61. Architectural Non-Negotiables
Authorization runtime es determinista.
Fail-closed es obligatorio.
Unknown no puede producir ALLOW.
Errores no pueden producir ALLOW.
Tenant isolation se evalúa como guard obligatorio.
Cross-tenant access está prohibido por defecto.
DENY prevalece sobre ALLOW en el mismo nivel.
Policies superiores no pueden ser debilitadas por policies inferiores.
No existe privilege amplification mediante delegation.
MFA/step-up son decisiones, no bypasses.
Policy evaluation no tiene side effects.
No se ejecuta código arbitrario.
No se realizan network calls arbitrarios durante evaluación.
Business rules permanecen en sus dominios.
AI Agents utilizan el mismo runtime de autorización.
Las decisiones críticas son auditables.
Policy version debe ser identificable.
Runtime version debe poder identificarse.
Redis nunca es fuente de verdad.
La disponibilidad nunca debe justificar un authorization bypass.
62. Traceability
Artifact	Relationship
BP-0002	Architecture lifecycle
ESP-0001	Identity
Security & Permissions	Authorization architecture
ADR-IDENTITY-005	RBAC + ABAC + Tenant + Ownership + Policy
ADR-IDENTITY-043	Multi-Organization / Tenant
ADR-IDENTITY-044	Roles / Delegation
ADR-IDENTITY-045	Authorization Decision Architecture
ADR-IDENTITY-046	Policy Language / Governance
ADR-IDENTITY-047	Policy Engine Technology
ADR-IDENTITY-048	Policy DSL / Expression Language
ADR-IDENTITY-049	Evaluation Runtime / Decision Algorithm
IS-IDENTITY-006	Authorization & Permission Evaluation
IS-IDENTITY-011	Security Policy Engine
IS-IDENTITY-012	Audit & Security Events
IS-IDENTITY-016	Testing & Quality
63. Dependencies

Este ADR depende directamente de:

ADR-IDENTITY-005
ADR-IDENTITY-043
ADR-IDENTITY-044
ADR-IDENTITY-045
ADR-IDENTITY-046
ADR-IDENTITY-047
ADR-IDENTITY-048

Y alimentará directamente:

Authorization Runtime
Policy Evaluator
Policy Compiler
Policy Repository
Authorization Cache
Authorization API
Audit
Security Events
Testing Framework.
64. Pending Decisions

Quedan para próximos ADRs o Technical Tasks:

Algoritmo formal de precedence.
Formalización de UNKNOWN.
Semántica exacta de INDETERMINATE.
Política de short-circuit.
Policy matching indexes.
Compiled policy representation.
Cache key design.
Decision cache TTL.
Invalidation propagation.
Exact obligation semantics.
Exact reason-code catalog.
Policy conflict analyzer.
Simulation runtime.
Runtime versioning mechanism.
Distributed PDP strategy.
Performance benchmarking.
Formal policy conformance test suite.
65. Acceptance Criteria

El ADR se considerará satisfecho cuando:

 Existe AuthorizationRuntime.
 Existe AuthorizationContext.
 Existe PolicyMatcher.
 Existe PolicyEvaluator.
 Existe RBAC evaluation.
 Existe ABAC evaluation.
 Existe Ownership evaluation.
 Existe Delegation evaluation.
 Existe Tenant Guard.
 Existe Policy Precedence.
 Existe Conflict Resolution.
 Existe Effect Resolution.
 Existe Obligation Resolution.
 Existe Reason Code catalog.
 Existe fail-closed.
 Existe deterministic evaluation.
 Existe policy version tracking.
 Existe runtime version tracking.
 Existe audit integration.
 Existe cache integration.
 Existe cross-tenant security testing.
 Existe concurrency testing.
 Existe performance testing.
 Se cumplen los objetivos de autorización definidos por Blueprint.
66. Architectural Outcome

Con ADR-IDENTITY-045 → 049, la arquitectura de autorización ya tiene una cadena coherente:

                 AUTHORIZATION REQUEST
                          │
                          ▼
                 ┌──────────────────┐
                 │       PEP        │
                 └────────┬─────────┘
                          ▼
                Request Validation
                          │
                          ▼
                 Authentication
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
                 ┌──────────────────┐
                 │       PDP        │
                 │                  │
                 │ RBAC             │
                 │ ABAC             │
                 │ Ownership        │
                 │ Delegation       │
                 │ Security Policy  │
                 │ MFA / Risk       │
                 └────────┬─────────┘
                          ▼
                  Policy Composition
                          │
                          ▼
                  Conflict Resolution
                          │
                          ▼
                   Effect Resolution
                          │
                          ▼
                  Obligation Resolution
                          │
                          ▼
              ┌────────────────────────┐
              │ Authorization Decision │
              └───────────┬────────────┘
                          │
                ┌─────────┼─────────┐
                ▼         ▼         ▼
              ALLOW      DENY     STEP-UP
                                    │
                                    ▼
                             MFA / REAUTH
                          │
                          ▼
                   Audit / Telemetry
Decisión principal

EVOXA utilizará un runtime de autorización determinista y fail-closed, basado en evaluación estructurada de políticas, con tenant isolation como guard obligatorio, integración de RBAC + ABAC + ownership + delegation, precedence jerárquica, DENY-wins dentro del mismo nivel, soporte para MFA/reauthentication/challenge, obligaciones controladas y trazabilidad de policy/runtime versions.

Con esto, ADR-IDENTITY-047, 048 y 049 forman prácticamente el núcleo técnico del Policy Engine: el primero define dónde y cómo vive, el segundo define cómo se expresan las políticas y este tercero define cómo se ejecutan.

Siguiente ADR recomendado

ADR-IDENTITY-050 — Authorization Policy Cache, Invalidation & Consistency Strategy

Este sería el siguiente paso natural porque ya definimos que el runtime necesita políticas compiladas/cacheadas. Ahí deberíamos cerrar Redis, claves de cache, TTL, invalidación por eventos, consistencia entre instancias, race conditions y qué ocurre durante una actualización de policy, antes de entrar en las Technical Tasks de implementación.
