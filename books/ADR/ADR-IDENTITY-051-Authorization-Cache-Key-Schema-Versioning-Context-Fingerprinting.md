ADR-IDENTITY-051 — Authorization Cache Key Schema, Versioning & Context Fingerprinting

Status: Proposed
Version: 1.0
Date: 2026-09-03
Domain: Identity / Authorization
Priority: Critical
Type: Architecture / Security / Performance

1. Context

El ADR-IDENTITY-050 — Authorization Policy Cache, Invalidation & Consistency Strategy estableció que EVOXA utilizará caching para reducir la latencia del Authorization Runtime, pero que:

La caché nunca será la fuente de verdad de seguridad.

También estableció:

L1 local cache;
L2 distributed cache;
PostgreSQL como source of truth;
versioning/epochs;
event-driven invalidation;
tenant isolation;
TTL;
fail-closed;
protección frente a stale data;
invalidación de cambios críticos.

El ADR-IDENTITY-049 — Authorization Policy Evaluation Runtime & Decision Algorithm establece que una autorización depende de un contexto compuesto por múltiples dimensiones:

Actor
Organization
Membership
Authentication
Session
MFA
Request
Resource
Delegation
Environment
Risk
Policy

Por lo tanto, no es seguro utilizar una cache key simplificada como:

user_id + resource_id + action

La misma persona puede tener:

diferentes memberships;
diferentes roles;
diferentes organizaciones;
diferentes sesiones;
diferentes niveles MFA;
diferentes delegaciones;
diferentes contextos de riesgo;
diferentes permisos sobre distintos recursos.

Este ADR define cómo construir:

Authorization Cache Keys
Security Versions
Epochs
Context Fingerprints
Invalidation scopes

sin permitir que el mecanismo de caching introduzca privilege escalation o cross-tenant access.

2. Problem

El sistema necesita identificar cuándo dos solicitudes de autorización son suficientemente equivalentes para reutilizar información cacheada.

Por ejemplo:

Actor A
Tenant A
Resource X
Action edit
Policy v10

puede tener:

ALLOW

pero:

Actor A
Tenant B
Resource X
Action edit
Policy v10

podría producir:

DENY

También:

Actor A
Tenant A
Resource X
Action edit
MFA verified

puede producir:

ALLOW

mientras:

Actor A
Tenant A
Resource X
Action edit
MFA not verified

produce:

REQUIRE_MFA

Por ello, la cache key debe representar las dimensiones de seguridad relevantes.

3. Decision

EVOXA adoptará una Authorization Cache Identity Model basada en cuatro conceptos:

1. Tenant Boundary
2. Authorization Subject Identity
3. Security Version / Epoch
4. Context Fingerprint

Conceptualmente:

Authorization Cache Key
        │
        ├── Tenant
        ├── Actor
        ├── Membership
        ├── Resource
        ├── Action
        ├── Security Versions
        └── Context Fingerprint

La key final podrá ser derivada de estos componentes, pero la representación física exacta queda como decisión de implementación.

4. Canonical Authorization Cache Identity

La identidad lógica de una decisión será:

AuthorizationCacheIdentity

con los siguientes componentes conceptuales:

tenant_id
actor_id
membership_id
resource_type
resource_id
action
policy_version
security_versions
context_fingerprint

No todos los componentes necesariamente deben aparecer literalmente en la key física.

5. Tenant ID

tenant_id será obligatorio cuando la autorización sea tenant-scoped.

Ejemplo:

tenant:org_123

La ausencia del tenant no podrá interpretarse como:

global tenant

automáticamente.

Para operaciones platform-scoped deberá existir un scope explícito:

PLATFORM

o equivalente definido por el modelo de autorización.

6. Actor Identity

La key debe distinguir al actor que realiza la operación.

Conceptualmente:

actor_type
actor_id

Ejemplos de actor:

USER
SERVICE
AI_AGENT
SYSTEM

Esto es especialmente importante porque EVOXA permite identidades distintas para:

usuarios;
servicios;
agentes AI;
procesos internos.

No se debe asumir:

actor_id == user_id

para todas las autorizaciones.

7. Membership Identity

Cuando la autorización dependa de una organización, la membership será parte de la identidad lógica.

membership_id

Esto evita depender exclusivamente de:

user_id + tenant_id

Una misma persona podría tener estados o roles diferentes según membership.

Ejemplo:

User 100
   │
   ├── Membership A → Tenant A
   │
   └── Membership B → Tenant B

La autorización debe mantenerse separada.

8. Resource Identity

La identidad del recurso tendrá dos dimensiones:

resource_type
resource_id

Ejemplo:

resource_type = workout
resource_id   = 987

No se utilizará únicamente:

987

porque distintos tipos de recursos pueden compartir IDs.

9. Action

La acción será parte obligatoria de la identidad.

El modelo conceptual utiliza:

<resource>.<action>

por ejemplo:

workout.view
workout.edit
workout.delete

La representación interna puede almacenar:

resource_type = workout
action = edit

pero el runtime deberá tratar ambos componentes como una unidad de autorización.

10. Policy Version

Cada decisión deberá estar asociada a la versión de políticas que participó en la evaluación.

Conceptualmente:

policy_version = 42

Una entrada generada con:

policy_version = 42

no deberá considerarse equivalente automáticamente a una evaluación con:

policy_version = 43
11. Security Epochs

Además de la policy version, EVOXA utilizará security epochs para representar cambios que invalidan múltiples entradas relacionadas.

Conceptualmente:

tenant_authz_epoch
policy_epoch
role_epoch
permission_epoch
membership_epoch
delegation_epoch
security_epoch

Ejemplo:

Tenant 123

policy_epoch      = 8
role_epoch        = 17
permission_epoch  = 21
membership_epoch  = 5

Una entrada cacheada puede incorporar estos valores.

12. Why Epochs

Supongamos:

Role Coach

tiene:

100.000 authorization cache entries

Si cambia un permiso del rol, no es necesario localizar y eliminar físicamente las 100.000 entradas.

Se puede incrementar:

role_epoch: 17 → 18

Las entradas antiguas:

role_epoch = 17

quedan obsoletas.

Esto proporciona invalidación lógica eficiente.

13. Version Hierarchy

Las versiones tendrán diferentes scopes.

Conceptualmente:

PLATFORM
   │
   └── platform_security_epoch

ORGANIZATION
   │
   ├── tenant_authz_epoch
   ├── policy_epoch
   ├── role_epoch
   ├── permission_epoch
   └── membership_epoch

RESOURCE
   │
   ├── ownership_epoch
   └── resource_security_epoch

SESSION
   │
   └── session_security_epoch

No todas las solicitudes necesitan todas las versiones.

14. Version Scope

La regla será:

Una versión sólo debe invalidar el ámbito que realmente controla.

Por ejemplo:

Tenant A role change

no debe invalidar necesariamente:

Tenant B

Por ello, las versiones deben ser scope-aware.

15. Ownership Version

El ownership es parte del modelo de autorización definido anteriormente.

Si:

resource.owner_id

cambia, una autorización cacheada basada en ownership puede dejar de ser válida.

Se utilizará conceptualmente:

ownership_version

o una versión equivalente del estado relevante.

Ejemplo:

Resource 500
owner = User A
ownership_version = 3

Después:

owner = User B
ownership_version = 4

Las evaluaciones antiguas quedan obsoletas.

16. Delegation Version

Las delegaciones son explícitas, temporales y revocables.

Por ello, si una autorización depende de:

delegation_id

deberá incorporar el estado/version relevante de delegación.

Ejemplo:

delegation_epoch = 12

Después de revocación:

delegation_epoch = 13

Una entrada anterior no podrá seguir proporcionando el mismo privilegio.

17. Session Security Version

El estado de sesión puede influir en la autorización.

Ejemplos:

sesión revocada;
step-up authentication;
MFA evidence;
security state change.

Por ello podrá utilizarse:

session_security_version

Ejemplo:

session_security_version = 4

Si ocurre una modificación de seguridad:

4 → 5

las decisiones dependientes de esa sesión quedan obsoletas.

18. MFA Context

MFA no debe tratarse como un atributo permanente del usuario.

Debe distinguirse:

MFA configured

de:

MFA satisfied for this authentication context

Por ello, cuando una política dependa de MFA, el contexto debe representar el estado de autenticación relevante.

Conceptualmente:

mfa_assurance
mfa_verified_at
mfa_method
step_up_evidence_version

No se almacenarán secretos MFA.

19. Risk Context

Risk puede participar en ABAC/policies.

Ejemplos conceptuales:

risk_level
device_trust
network_context
anomaly_state

No necesariamente todos deben formar parte literalmente de la cache key.

Cuando una de estas variables pueda modificar la decisión, deberá participar en el:

context_fingerprint

o provocar bypass del decision cache.

20. Context Fingerprint

Para evitar claves excesivamente grandes, se utilizará un:

context_fingerprint

cuando sea apropiado.

Conceptualmente:

Context
   ↓
Canonicalization
   ↓
Stable Serialization
   ↓
Cryptographic Hash
   ↓
Context Fingerprint

Ejemplo:

context_fingerprint =
    H(canonical_security_context)

El algoritmo criptográfico exacto queda pendiente de implementación.

21. Canonical Context

El fingerprint sólo será determinista si el contexto se normaliza.

Por ejemplo:

{
  "device_trust": true,
  "mfa": "verified",
  "risk": "low"
}

y:

{
  "risk": "low",
  "mfa": "verified",
  "device_trust": true
}

deben producir el mismo fingerprint si semánticamente representan el mismo contexto.

Por ello se requiere canonicalización.

22. Security-Relevant Attributes Only

El fingerprint no debe incluir indiscriminadamente todo el request.

No se deben incluir datos que:

no afectan la autorización;
aumenten cardinalidad innecesariamente;
sean sensibles sin necesidad;
hagan que el cache sea prácticamente inútil.

Debe contener exclusivamente atributos relevantes para la decisión.

23. Example Context

Un contexto conceptual podría ser:

actor:
  type: USER
  id: user_100

tenant:
  id: org_10

membership:
  id: membership_50

resource:
  type: workout
  id: workout_900

request:
  action: edit

authentication:
  assurance: MFA

environment:
  risk: LOW

delegation:
  id: delegation_20

Después de canonicalización:

context_fingerprint = H(canonical_context)
24. Cache Key Structure

La estructura lógica recomendada será:

authz:
  version
  scope
  tenant
  actor
  membership
  resource
  action
  security_version
  context_fingerprint

Una representación conceptual podría ser:

authz:v1:
tenant:{tenant_id}:
actor:{actor_type}:{actor_id}:
membership:{membership_id}:
resource:{resource_type}:{resource_id}:
action:{action}:
versions:{version_fingerprint}:
context:{context_fingerprint}

Esto es una representación conceptual, no una obligación de formato literal.

25. Version Fingerprint

Para evitar una key excesivamente larga, las versiones pueden canonicalizarse:

policy_epoch
role_epoch
permission_epoch
membership_epoch
delegation_epoch
ownership_epoch
session_security_epoch

y convertirse en:

version_fingerprint

Ejemplo conceptual:

version_fingerprint =
H(
  policy_epoch,
  role_epoch,
  permission_epoch,
  membership_epoch,
  delegation_epoch,
  ownership_epoch,
  session_security_epoch
)

Sólo deberán incluirse las dimensiones relevantes.

26. Do Not Hash Away Tenant Isolation

Aunque la key utilice hashes/fingerprints, el tenant debe seguir formando parte explícita de la identidad lógica.

No se debe depender únicamente de:

hash(user + resource + context)

sin tenant separation.

La arquitectura debe poder demostrar:

Tenant A
≠
Tenant B

independientemente del fingerprint.

27. Cache Entry

La entrada cacheada deberá contener metadata suficiente para validar su vigencia.

Conceptualmente:

AuthorizationCacheEntry

podrá incluir:

decision
reason_code
policy_version
security_versions
created_at
expires_at
runtime_version

Opcionalmente:

obligations

cuando sean seguras y necesarias.

28. Decision Values

La cache debe soportar las decisiones del Authorization Runtime:

ALLOW
DENY
CHALLENGE
REAUTHENTICATE
REQUIRE_MFA
REVOKE

No debe asumir que solamente existen:

true / false
29. Cache ALLOW vs DENY

Las decisiones ALLOW y DENY no necesariamente tendrán el mismo TTL.

Por ejemplo:

ALLOW → short-lived
DENY  → short-lived / policy dependent

La razón es que una condición de seguridad puede cambiar entre evaluaciones.

Los TTL exactos quedan fuera de este ADR.

30. Obligations

Si la decisión contiene:

ALLOW

con obligaciones:

obligation = require_audit

la cache no debe eliminar esa obligación.

Una cache hit debe reproducir la semántica completa de la decisión.

Por ello:

Decision
=
Effect
+
Reason
+
Obligations
+
Relevant Metadata

cuando corresponda.

31. Policy Runtime Version

El algoritmo de evaluación puede evolucionar.

Por ello una entrada puede incorporar:

runtime_version

Ejemplo:

runtime_version = 3

Si cambia la semántica del Authorization Runtime de forma incompatible:

3 → 4

las entradas antiguas pueden invalidarse globalmente.

Esto evita que una nueva versión del runtime reutilice resultados producidos por una lógica anterior incompatible.

32. Policy DSL Version

La política también tiene una versión de lenguaje/schema.

Si cambia el significado de una expresión o del DSL, puede ser necesario invalidar las políticas compiladas.

Conceptualmente:

dsl_version
compiler_version
runtime_version

Esto es particularmente relevante para los ADR-048 y ADR-049.

33. Compiled Policy Cache Key

La caché de políticas compiladas será conceptualmente diferente de la decision cache.

compiled-policy:
    policy_id
    policy_version
    dsl_version
    compiler_version

Ejemplo:

policy:{tenant}:{policy_id}:{policy_version}:{compiler_version}

La policy cache no debe confundirse con:

authorization decision cache
34. Permission Resolution Cache

También puede existir una caché específica para:

Membership
      ↓
Roles
      ↓
Permissions

Su identidad conceptual:

tenant
membership
role_version
permission_version

Ejemplo:

permissions:
tenant_10:
membership_50:
role_epoch_8:
permission_epoch_12

Esto evita recalcular permisos para cada request.

35. Ownership Cache

Ownership puede cachearse separadamente cuando sea apropiado.

Ejemplo:

ownership:
tenant
resource_type
resource_id
ownership_version

Sin embargo, una ownership cache no debe sustituir las reglas de autorización.

36. Cache Dependency Graph

Las dependencias conceptuales serán:

                 Policy
                   │
                   ▼
              Policy Version
                   │
                   ▼
Role ───────► Permission
 │               │
 ▼               ▼
Role Epoch   Permission Epoch
        \       /
         \     /
          Membership
              │
              ▼
       Membership Epoch
              │
              ▼
       Authorization Context
              │
      ┌───────┼────────┐
      ▼       ▼        ▼
 Ownership Delegation Session
      │       │        │
      └───────┼────────┘
              ▼
       Context Fingerprint
              │
              ▼
       Authorization Cache
37. Invalidation Rules
Policy Update

Increment:

policy_epoch

and/o actualizar la policy version.

Role Permission Change

Increment:

role_epoch
permission_epoch

según el scope afectado.

Membership Role Change

Increment:

membership_epoch
Membership Revocation

Increment:

membership_epoch
security_epoch

y puede requerir invalidación inmediata de sesiones/refresh según las reglas correspondientes.

Delegation Revocation

Increment:

delegation_epoch
Ownership Change

Increment:

ownership_epoch
Security-Critical Account Change

Increment:

security_epoch

y aplicar las acciones definidas por los ADR correspondientes.

38. Global Security Epoch

EVOXA podrá mantener un epoch de seguridad de mayor alcance:

platform_security_epoch

Debe utilizarse únicamente para cambios verdaderamente globales.

No debe incrementarse ante cualquier modificación porque produciría invalidaciones masivas innecesarias.

39. Hierarchical Epoch Validation

El runtime podrá validar:

platform epoch
tenant epoch
membership epoch
resource epoch
session epoch

Conceptualmente:

Cached Version
      │
      ▼
Current Version
      │
      ├── equal → potentially valid
      │
      └── different → stale

Una sola versión obsoleta puede invalidar la entrada.

40. Context Fingerprint Stability

El fingerprint debe ser:

determinista;
reproducible;
independiente del orden de campos;
sensible a cambios semánticos;
seguro;
no reversible;
no basado en datos secretos innecesarios.

No debe utilizarse como mecanismo de autenticación.

Es únicamente un identificador de contexto para caching.

41. Context Fingerprint Is Not Authorization

Una advertencia fundamental:

fingerprint == X

no significa:

authorized

El fingerprint solamente significa:

este contexto corresponde a esta identidad cacheable

La decisión ALLOW/DENY/... sigue siendo producto del Authorization Runtime.

42. Cache Lookup Algorithm

El lookup conceptual será:

1. Validate request
2. Validate tenant
3. Build authorization context
4. Determine relevant security versions
5. Canonicalize relevant context
6. Generate context fingerprint
7. Build cache identity
8. Check L1
9. Check L2
10. Validate TTL
11. Validate versions
12. Validate runtime version
13. If valid → use cached decision
14. Otherwise → evaluate policy
15. Store new result
43. Cache Hit Must Be Revalidated

Un cache hit no significa automáticamente:

ALLOW

El runtime debe verificar:

TTL valid
+
versions valid
+
tenant valid
+
runtime compatible
+
context compatible

Sólo entonces podrá reutilizarse la entrada.

44. Cache Miss

Ante:

cache miss

el runtime ejecutará el algoritmo completo de ADR-049.

Cache Miss
   ↓
Current Policy State
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
Decision
45. Stale Entry

Una entrada con:

cached_version != current_version

se considera:

STALE

No debe utilizarse para producir la decisión final.

Puede eliminarse físicamente o simplemente quedar invalidada lógicamente.

46. Race Conditions

Debe contemplarse:

Request A → reads version 42

Request B → updates policy to 43

Request A → tries to write cache

La entrada generada por Request A no debe convertirse en una entrada válida para versión 43.

Por ello, el cache write debe incluir la versión con la que fue evaluado.

47. Compare-and-Set

Cuando sea necesario, la escritura puede utilizar mecanismos equivalentes a:

expected_version
current_version

para evitar que resultados antiguos sobrescriban resultados nuevos.

La tecnología exacta queda pendiente.

48. Cache Namespace

La arquitectura utilizará namespaces lógicos para separar:

policy
compiled_policy
permissions
ownership
decision
security_state

Ejemplo:

authz/policy/...
authz/compiled/...
authz/permissions/...
authz/ownership/...
authz/decision/...

El formato exacto de Redis key queda pendiente.

49. Avoid High Cardinality

No se deben incluir automáticamente atributos de alta variabilidad como:

request_id
trace_id
timestamp exacto
random nonce

porque destruirían el beneficio de caching.

Sólo deben formar parte del fingerprint si tienen significado directo para la autorización.

50. Time-Based Conditions

Las políticas pueden utilizar condiciones temporales:

current_time
business_hours
effective_from
effective_until

Cuando el tiempo pueda cambiar la decisión, existen dos opciones:

no utilizar decision cache;
incorporar una representación temporal controlada.

La opción exacta queda pendiente.

No se permitirá un cache hit que ignore una condición temporal relevante.

51. Risk-Based Conditions

Si una policy contiene:

risk_level == HIGH

el risk context debe participar en la identidad de caching o impedir el cacheo de la decisión.

Nunca se reutilizará:

ALLOW when risk=LOW

para:

risk=HIGH
52. Device Trust

La confianza del dispositivo puede afectar authorization.

Por ello:

trusted_device = true

y:

trusted_device = false

deben producir contextos diferentes cuando esa condición sea relevante.

53. Session State

Si una política depende de:

session active
session revoked
MFA state
authentication assurance

el session/security epoch deberá reflejar esos cambios.

Una sesión revocada no puede seguir utilizando una autorización cacheada que presuponga una sesión válida.

54. Cross-Tenant Protection

Antes de consultar la cache:

requested_tenant

debe compararse con:

authorized tenant context

Una mismatch debe producir:

DENY

o una decisión segura equivalente.

Nunca:

cache lookup → discover tenant

El tenant debe venir del contexto autenticado/autorizado.

55. Platform Scope

Las operaciones platform-scoped deberán utilizar un namespace distinto.

Conceptualmente:

authz:platform:...

Esto evita mezclar:

platform authorization

con:

tenant authorization
56. AI Scope

Los AI agents deberán tener:

actor_type = AI_AGENT

y su contexto deberá conservar:

identidad del agente;
tenant;
delegación;
task context cuando sea security-relevant;
scopes/permisos;
policy context.

Nunca se reutilizará automáticamente una autorización de:

USER

para:

AI_AGENT
57. Security and Privacy

El cache key no debe exponer datos sensibles innecesariamente.

Preferentemente:

opaque IDs

y fingerprints.

No:

email
name
phone
health data
password
token
MFA secret
58. Consequences
Positivas
Seguridad

La estructura reduce:

cross-tenant leakage;
stale privilege;
privilege amplification;
context confusion.
Performance

Permite reutilizar:

policies compiladas;
permission resolution;
decisiones cuando sea seguro.
Scalability

Epochs permiten invalidación lógica sin buscar millones de entradas.

Evolución

Runtime y DSL versions permiten invalidar caches cuando cambia la semántica.

59. Negative Consequences

La estrategia aumenta la complejidad en:

construcción de contexto;
versioning;
invalidación;
fingerprinting;
cardinalidad;
debugging;
observabilidad;
testing de concurrencia.

Una implementación incorrecta podría introducir vulnerabilidades de autorización.

60. Alternatives Rejected
A. user_id + resource_id + action

Rechazado.

No representa tenant, membership, roles, session, MFA, delegation ni contexto.

B. Full Request Hash

Rechazado.

Produciría cardinalidad excesiva e incluiría información irrelevante.

C. Tenant Only

Rechazado.

No distingue actores ni recursos.

D. TTL Without Versions

Rechazado.

No proporciona invalidación suficientemente segura.

E. Global Epoch Only

Rechazado.

Una modificación local provocaría invalidación global innecesaria.

F. Physical Deletion Only

Rechazado.

No es suficientemente robusto en escenarios distribuidos.

61. Non-Negotiable Rules
Tenant siempre debe estar representado en la identidad lógica.
Actor identity debe estar representada.
Membership debe distinguirse cuando sea relevante.
Resource type + resource ID deben distinguir el recurso.
Action es obligatoria.
Policy version debe poder identificarse.
Security epochs deben utilizarse para cambios relevantes.
Context fingerprint debe ser determinista.
Fingerprint sólo incluye atributos security-relevant.
No secretos en cache keys ni fingerprints.
Cross-tenant cache reuse está prohibido.
Stale versions no producen ALLOW.
Runtime version incompatible invalida cache entries.
AI agents no reutilizan automáticamente decisiones de usuarios.
Cache key no sustituye Authorization Runtime.
Cache miss ejecuta la evaluación completa.
Los resultados antiguos no pueden sobrescribir resultados nuevos.
La invalidación debe respetar el scope correspondiente.
Security-critical changes deben tener invalidación más estricta.
No se permite privilege amplification mediante cache.
62. Traceability
Blueprint

Relacionado con:

Identity Domain
Security & Permissions
RBAC
ABAC
Tenant Isolation
Resource Ownership
Delegation
Security Policy Engine
Event-Driven Architecture
Performance
Observability

El modelo de autorización combina RBAC, ABAC y políticas contextuales, incluyendo organización/tenant, ownership, device trust, risk y estado de sesión.

El pipeline de autorización definido en el Blueprint sigue conceptualmente:

Request
→ Authentication
→ Role Resolution
→ Policy Evaluation
→ Permission Check
→ Execution

y contempla protección contra cross-tenant access.

Los objetivos de rendimiento del dominio incluyen Authorization <50 ms y Permission Resolution <30 ms, justificando la necesidad de una estrategia de caching controlada.

Lifecycle

La trazabilidad continúa:

Blueprint
   ↓
Reference Architecture
   ↓
ESP
   ↓
ADR
   ↓
Implementation Story
   ↓
Technical Tasks
   ↓
Source Code
   ↓
Testing

Este enfoque coincide con el lifecycle arquitectónico establecido para EVOXA.

63. Related ADRs

Directamente relacionados:

ADR-IDENTITY-005 — Authorization Model
ADR-IDENTITY-006 — PostgreSQL Persistence
ADR-IDENTITY-007 — Transactional Outbox
ADR-IDENTITY-019 — Partitioned Event Streaming
ADR-IDENTITY-020 — Event Schema Registry
ADR-IDENTITY-021 — Controlled Replay
ADR-IDENTITY-022 — Consumer Idempotency
ADR-IDENTITY-028 — Observability & Operational Security
ADR-IDENTITY-031 — Identity HA & Failover
ADR-IDENTITY-043 — Multi-Organization Identity
ADR-IDENTITY-044 — Membership Roles/Delegation/Fine-Grained
ADR-IDENTITY-045 — Authorization Policy Model/Evaluation
ADR-IDENTITY-046 — Policy Language/Schema/Governance
ADR-IDENTITY-047 — Policy Engine Technology & Implementation
ADR-IDENTITY-048 — Authorization Policy DSL
ADR-IDENTITY-049 — Authorization Policy Evaluation Runtime
ADR-IDENTITY-050 — Authorization Policy Cache, Invalidation & Consistency
64. Pending Decisions

Este ADR define el modelo conceptual, pero todavía deben especificarse:

Algoritmo exacto del context fingerprint.
Algoritmo exacto de canonicalización.
Hash algorithm.
Exact Redis key format.
Exact version/epoch schema.
Qué dimensiones son obligatorias por tipo de autorización.
TTL de cada categoría.
Decision-cache eligibility matrix.
Time-based policy caching.
Risk-based policy caching.
MFA/step-up cache rules.
Ownership cache strategy.
Delegation cache strategy.
Cache invalidation event schema.
Single-flight/distributed locking.
Cache serialization.
Cache compression.
Maximum cardinality.
Cache eviction strategy.
Cross-region cache consistency.
Global vs tenant-local epoch implementation.
Runtime/DSL/compiler version migration.
Cache warm-up strategy.
Security-critical cache bypass matrix.

Estas decisiones deberán resolverse antes de implementar el mecanismo definitivo.

65. Acceptance Criteria
AC-01 — Tenant Isolation

Una entrada creada para Tenant A nunca podrá ser reutilizada para Tenant B.

AC-02 — Actor Isolation

Una entrada de User A no podrá reutilizarse para User B.

AC-03 — Membership Isolation

Memberships distintas deberán producir identidades distintas cuando afecten autorización.

AC-04 — Resource Isolation

Recursos diferentes no podrán compartir accidentalmente una decisión.

AC-05 — Action Isolation

resource.view y resource.edit deberán tratarse como autorizaciones distintas.

AC-06 — Policy Version

Una entrada de policy version antigua deberá detectarse como stale cuando exista una versión posterior aplicable.

AC-07 — Epoch Invalidation

Un cambio de epoch deberá invalidar lógicamente las entradas afectadas.

AC-08 — Out-of-Order Events

Un evento antiguo no podrá retroceder el epoch/version actual.

AC-09 — Context Fingerprint

Contextos security-equivalent producirán fingerprints equivalentes; contextos security-different no podrán colisionar de forma que produzcan una autorización incorrecta.

AC-10 — MFA

Un contexto que requiere MFA no podrá reutilizar una autorización generada sin MFA suficiente.

AC-11 — Risk

Un cambio relevante de risk context deberá impedir la reutilización incorrecta de una decisión.

AC-12 — Ownership

Un cambio de ownership deberá invalidar las decisiones dependientes del ownership anterior.

AC-13 — Delegation

Una delegación revocada no podrá continuar generando privilegios mediante una entrada cacheada.

AC-14 — Session

Una modificación de security state de sesión deberá invalidar las decisiones dependientes de ese estado.

AC-15 — Runtime Version

Una entrada generada por una versión incompatible del runtime no podrá reutilizarse.

AC-16 — AI Isolation

Una decisión de usuario no podrá reutilizarse automáticamente para un AI Agent.

AC-17 — Fail Closed

Cache corruption, missing version, inconsistent context o fingerprint failure no podrán producir ALLOW.

AC-18 — Concurrency

Una evaluación antigua no podrá sobrescribir una entrada correspondiente a una versión más nueva.

AC-19 — No Secrets

Ninguna cache key, fingerprint o entry podrá contener secretos.

AC-20 — Performance

La estrategia deberá contribuir a cumplir los objetivos establecidos para Authorization y Permission Resolution sin comprometer los controles de seguridad.

66. Architectural Result

Con este ADR queda definido el siguiente modelo:

                  ┌─────────────────────┐
                  │ PostgreSQL          │
                  │ Source of Truth     │
                  └──────────┬──────────┘
                             │
                    Policy / Security
                       Versions
                             │
                             ▼
              ┌──────────────────────────┐
              │ Authorization Context    │
              └────────────┬─────────────┘
                           │
              ┌────────────┴────────────┐
              │                         │
        Security Epochs          Context Fingerprint
              │                         │
              └────────────┬────────────┘
                           ▼
                 Authorization Cache
                           │
             ┌─────────────┴─────────────┐
             ▼                           ▼
          L1 Local                    L2 Redis
             │                           │
             └─────────────┬─────────────┘
                           ▼
                Authorization Runtime
                           │
                           ▼
              ALLOW / DENY / CHALLENGE
              REAUTHENTICATE / MFA / REVOKE

La regla arquitectónica central queda establecida:

La cache identity debe representar el contexto de seguridad suficiente para demostrar que una decisión cacheada sigue siendo aplicable; la cache nunca puede convertirse en una autoridad independiente de autorización.

Siguiente ADR recomendado

El siguiente paso lógico es:

ADR-IDENTITY-052 — Authorization Cache Eligibility, TTL & Security-Sensitive Decision Caching

Ahí podemos definir qué decisiones concretas se pueden cachear y cuáles nunca, diferenciando:

ALLOW
DENY
REQUIRE_MFA
REAUTHENTICATE
CHALLENGE
REVOKE

y además establecer una matriz de caching para:

MFA;
risk;
ownership;
delegation;
time-based policies;
operaciones críticas;
AI agents;
administración de Identity.

Eso cerraría la parte más importante antes de pasar a los Technical Tasks del Authorization Policy Engine.
