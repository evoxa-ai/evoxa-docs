ADR-IDENTITY-052 — Authorization Cache Eligibility, TTL & Security-Sensitive Decision Caching

Status: Proposed
Version: 1.0
Date: 2026-09-03
Domain: Identity / Authorization
Priority: Critical
Type: Architecture / Security / Performance

1. Context

El ADR-IDENTITY-050 — Authorization Policy Cache, Invalidation & Consistency Strategy estableció que EVOXA utilizará caching para reducir la latencia del Authorization Runtime, pero que:

La caché mejora performance; nunca constituye una fuente de verdad de seguridad.

El ADR-IDENTITY-051 — Authorization Cache Key Schema, Versioning & Context Fingerprinting definió la identidad lógica necesaria para una entrada cacheada:

Tenant
Actor
Membership
Resource
Action
Policy Version
Security Epochs
Context Fingerprint

Sin embargo, todavía falta resolver una cuestión fundamental:

¿Qué decisiones de autorización pueden almacenarse en cache y durante cuánto tiempo?

No todas las decisiones tienen el mismo riesgo.

Por ejemplo:

ALLOW

para consultar información de bajo riesgo puede tolerar una estrategia de caching diferente de:

ALLOW

para modificar:

permisos;
MFA;
tenants;
roles;
credenciales;
configuraciones de seguridad.

Además, decisiones como:

REQUIRE_MFA
REAUTHENTICATE
CHALLENGE
REVOKE

pueden depender de condiciones dinámicas que hacen que su cacheabilidad sea muy limitada.

Este ADR define la política de eligibilidad, TTL y restricciones para decision caching.

2. Problem

Un sistema de caching mal diseñado puede convertir una optimización de performance en una vulnerabilidad.

Ejemplo:

10:00
User → MFA verified
       ↓
ALLOW cached

10:05
MFA/session/security state changes

10:06
User → same resource/action

Cache → ALLOW

Si el runtime no detecta el cambio, una decisión anterior puede continuar autorizando una operación que ya debería requerir:

MFA
REAUTHENTICATE
DENY

Por ello, EVOXA debe diferenciar:

decisiones cacheables;
decisiones cacheables bajo condiciones;
decisiones que no deben cachearse.
3. Decision

EVOXA adoptará una Decision Cache Eligibility Policy con tres categorías:

CACHEABLE
CONDITIONALLY_CACHEABLE
NON_CACHEABLE

La decisión dependerá de:

tipo de efecto;
sensibilidad de la operación;
estabilidad del contexto;
dependencia de MFA;
dependencia de risk;
dependencia de ownership;
dependencia de delegation;
dependencia temporal;
security state;
posibilidad de revocación;
TTL permitido;
versiones/epochs disponibles.
4. Core Principle

La regla fundamental será:

Si una decisión depende de un atributo que no puede representarse y validarse de forma segura dentro de la cache identity/version model, la decisión no debe cachearse.

Por ejemplo:

risk_score

si cambia frecuentemente y no existe un mecanismo confiable para detectar el cambio:

→ NON_CACHEABLE
5. Decision Cache Categories
5.1 CACHEABLE

Decisiones suficientemente estables y con contexto correctamente representado.

Ejemplo conceptual:

ALLOW

para una operación de bajo riesgo cuyo contexto de autorización sea estable.

5.2 CONDITIONALLY_CACHEABLE

Puede cachearse sólo si se cumplen condiciones específicas.

Ejemplos:

ALLOW + MFA context
ALLOW + ownership
ALLOW + delegation
DENY based on policy

Requieren:

context fingerprint;
security versions;
TTL apropiado;
invalidation;
condiciones adicionales.
5.3 NON_CACHEABLE

Nunca deben reutilizarse como decision cache normal.

Ejemplos:

operaciones altamente sensibles;
decisiones dependientes de información extremadamente dinámica;
decisiones cuyo contexto no pueda representarse de manera segura;
determinadas decisiones MFA/step-up;
operaciones críticas de Identity.
6. Default Rule

La política será:

Unknown eligibility
        ↓
NON_CACHEABLE

Nunca:

Unknown eligibility
        ↓
CACHEABLE

Esto mantiene el principio fail-closed.

7. Decision Effects

El Authorization Runtime definido en ADR-049 soporta:

ALLOW
DENY
CHALLENGE
REAUTHENTICATE
REQUIRE_MFA
REVOKE

Cada efecto tendrá una política de caching diferente.

8. ALLOW

ALLOW será el principal candidato a decision caching.

Pero:

ALLOW no significa automáticamente CACHEABLE.

La decisión deberá pasar por la matriz de elegibilidad.

9. ALLOW — Cacheable

Un ALLOW podrá cachearse cuando:

tenant está correctamente identificado;
actor está identificado;
membership es estable;
resource/action están definidos;
policy version está disponible;
security epochs están disponibles;
context fingerprint representa todas las variables relevantes;
no existe condición altamente dinámica;
operación no está clasificada como security-critical;
TTL es compatible con el riesgo.

Ejemplo:

User
+
Tenant
+
Membership
+
workout.view
+
stable policy
+
stable context

→ potencialmente cacheable.

10. ALLOW — Conditionally Cacheable

Un ALLOW será condicional cuando dependa de:

ownership;
delegation;
MFA;
device trust;
risk;
session security;
temporal conditions.

El cacheo sólo será válido si la dimensión correspondiente está incluida en:

Security Version

o:

Context Fingerprint

y existe un TTL apropiado.

11. ALLOW — Non-Cacheable

No deberá cachearse cuando:

el contexto relevante no puede representarse de forma confiable;
la operación requiere evaluación fresh;
el cambio de autorización puede tener consecuencias críticas;
existe una condición dinámica no versionable;
la política explícitamente prohíbe caching.

Ejemplo:

change MFA configuration

puede requerir:

fresh authorization evaluation

en lugar de reutilizar un ALLOW anterior.

12. DENY

Las decisiones DENY podrán cachearse en determinados escenarios.

Esto puede reducir evaluaciones repetitivas para solicitudes claramente no autorizadas.

Ejemplo:

User A
→ workout.delete
→ DENY

Sin embargo, el usuario puede recibir posteriormente un rol:

Coach Admin

Por ello el DENY debe invalidarse mediante:

role_epoch
permission_epoch
membership_epoch
policy_epoch

según corresponda.

13. DENY TTL

El TTL de DENY podrá ser diferente del de ALLOW.

La decisión exacta deberá considerar:

frecuencia esperada de cambios;
sensibilidad;
coste de evaluación;
riesgo de stale denial.

Una denegación stale generalmente produce:

DENY

en vez de:

ALLOW

por lo que su impacto de seguridad es diferente.

Aun así, no se permitirá caching indefinido.

14. CHALLENGE

CHALLENGE representa que deben cumplirse pasos adicionales.

Ejemplo:

CHALLENGE

puede requerir:

verification;
device challenge;
additional proof.

Las decisiones CHALLENGE no deberán tratarse como autorización permanente.

Su cacheabilidad será:

CONDITIONALLY_CACHEABLE

o:

NON_CACHEABLE

según la naturaleza del challenge.

15. REQUIRE_MFA

REQUIRE_MFA tiene una característica especial:

REQUIRE_MFA

no es equivalente a:

ALLOW

Es una instrucción de seguridad.

Puede ser cacheada sólo cuando:

la policy que exige MFA permanece vigente;
el contexto sigue siendo equivalente;
la operación sigue siendo la misma;
no existe evidencia de MFA válida que cambie el resultado;
TTL es extremadamente controlado.

En muchos escenarios será preferible:

NON_CACHEABLE

para operaciones sensibles.

16. REAUTHENTICATE

REAUTHENTICATE es altamente dependiente del estado de autenticación.

Por ello:

REAUTHENTICATE

será generalmente:

NON_CACHEABLE

o, como máximo, estrictamente condicionado.

No se debe utilizar una decisión antigua para omitir una nueva autenticación requerida por una policy.

17. REVOKE

REVOKE representa una acción de seguridad.

No se debe cachear como si fuera una autorización ordinaria.

La decisión:

REVOKE

debe generar/aplicar el efecto correspondiente en el estado de seguridad.

Por defecto:

REVOKE → NON_CACHEABLE

y cualquier resultado de revocación deberá mantenerse en las fuentes de verdad correspondientes.

18. Preliminary Eligibility Matrix
Decision	Default Eligibility	Security Rule
ALLOW	Conditionally Cacheable	Depende del contexto
DENY	Conditionally Cacheable	Version/TTL obligatorio
CHALLENGE	Conditionally Cacheable	Muy dependiente del challenge
REQUIRE_MFA	Non-Cacheable / Conditional	Nunca bypass de MFA
REAUTHENTICATE	Non-Cacheable	Fresh security evaluation preferred
REVOKE	Non-Cacheable	Efecto de seguridad, no cache normal

Esta matriz es normativa a nivel arquitectónico; los TTL exactos y excepciones deberán definirse posteriormente.

19. Security-Sensitive Operations

Las operaciones de Identity se dividirán conceptualmente por sensibilidad.

Nivel 1 — Low Risk

Ejemplos:

view
list
read metadata

Mayor posibilidad de caching.

Nivel 2 — Moderate Risk

Ejemplos:

create
edit
update

Caching condicionado.

Nivel 3 — High Risk

Ejemplos:

change permissions
assign roles
change membership
delegation

Caching muy restringido.

Nivel 4 — Critical Security

Ejemplos:

change MFA
reset credentials
security policy changes
tenant security administration
credential management

Preferencia:

fresh authorization evaluation
20. Critical Security Rule

Para operaciones clasificadas como Critical Security:

La decisión de autorización no deberá depender de una decision cache normal que pueda ocultar cambios recientes de seguridad.

Debe realizarse una evaluación fresh o una validación de seguridad equivalente definida por el runtime.

21. MFA-Sensitive Caching

Si una policy contiene:

require MFA

la decisión depende del nivel de assurance de autenticación.

Por lo tanto:

MFA state
+
session security version
+
step-up evidence

deben formar parte del contexto.

Una entrada:

ALLOW + MFA

no puede reutilizarse para:

no MFA
22. Step-Up Authentication

El step-up es temporal.

Ejemplo:

10:00
User authenticates

Luego:

10:05
Step-up MFA

Una decisión que dependa de esa evidencia no puede permanecer válida indefinidamente.

Por ello:

step_up_evidence

debe tener:

expiración;
contexto;
scope;
purpose;
session binding.

Y su decisión cacheada debe respetar esos límites.

23. Risk-Sensitive Caching

Si una autorización depende de:

risk_level

la elegibilidad dependerá de la estabilidad de ese valor.

Ejemplo:

risk = LOW
ALLOW cached

Posteriormente:

risk = HIGH

La decisión anterior no debe reutilizarse.

Por tanto:

dynamic risk
→ short TTL or no cache

La estrategia exacta depende de la capacidad del sistema para versionar el risk state.

24. Device Trust

Cuando:

device_trust

afecta una policy:

trusted device

no debe equivaler a:

trusted forever

El contexto debe incluir:

device identity;
trust state;
relevant security version;
expiration cuando corresponda.
25. Ownership-Sensitive Caching

Si:

resource.owner == actor

es parte de la policy, el resultado podrá cachearse sólo si el ownership state es detectable.

Por ejemplo:

ownership_epoch = 8

Una transferencia:

owner A → owner B

produce:

ownership_epoch = 9

y las decisiones antiguas quedan stale.

26. Delegation-Sensitive Caching

Delegation introduce:

delegator;
delegate;
scope;
resource;
action;
validity period;
revocation.

Por ello, una decisión basada en delegation deberá incluir:

delegation identity
+
delegation version/epoch
+
expiration

Una delegation revocada nunca podrá continuar proporcionando privilegios por cache.

27. Time-Based Policies

Ejemplo:

ALLOW only 09:00–18:00

Un decision cache no puede ignorar el tiempo.

La implementación podrá:

evitar decision caching;
utilizar TTL hasta el próximo boundary;
incluir un time bucket seguro.

La opción exacta queda pendiente.

Regla:

Nunca reutilizar una autorización fuera de la ventana temporal de la policy.

28. Policy-Defined Cacheability

Las políticas podrán incluir metadata conceptual:

cacheability

por ejemplo:

CACHE_ALLOWED
CACHE_RESTRICTED
CACHE_FORBIDDEN

Esto permite que determinadas políticas críticas impidan explícitamente decision caching.

La capacidad de restringir caching no podrá utilizarse para debilitar los controles de plataforma.

29. Platform Minimums

Una organización podrá definir:

CACHE_ALLOWED

pero no podrá cambiar una policy de plataforma:

CACHE_FORBIDDEN

a:

CACHE_ALLOWED

si esto viola un mínimo de seguridad.

Esto sigue el modelo de precedencia establecido en ADR-046.

30. TTL Hierarchy

El TTL efectivo deberá estar limitado por varias restricciones.

Conceptualmente:

Effective TTL =
MIN(
    cache_policy_TTL,
    security_class_TTL,
    context_TTL,
    session_TTL,
    MFA_TTL,
    delegation_TTL,
    policy_expiration,
    resource_expiration
)

No se utilizará un TTL mayor que el periodo durante el cual todos los supuestos de seguridad continúan siendo válidos.

31. Maximum TTL

Cada categoría tendrá un máximo definido por la plataforma.

Ejemplo conceptual:

LOW_RISK
    short/medium

MODERATE_RISK
    short

HIGH_RISK
    very short

CRITICAL_SECURITY
    no decision cache

Los valores concretos quedan pendientes de benchmark y security validation.

32. TTL Does Not Replace Invalidation

Un error común sería:

TTL = 30 seconds

y asumir:

después de 30 segundos todo está seguro.

No.

Debe utilizarse:

TTL
+
Versioning
+
Event Invalidation

El TTL es un mecanismo de seguridad secundaria y de freshness, no el mecanismo principal de revocación.

33. Security Revocation

Cuando exista una revocación crítica:

Membership revoked
Role revoked
Permission revoked
Delegation revoked
Session revoked
Security state changed

no se debe esperar simplemente a que:

TTL expires

Debe producirse:

event
+
epoch/version change
+
cache invalidation

según corresponda.

34. Cache Eligibility Evaluation

Antes de guardar una decisión:

Decision
   ↓
Eligibility Evaluator

deberá determinar:

CanCache?
Why?
TTL?
Dependencies?
RequiredVersions?

Conceptualmente:

CacheEligibility
{
    eligible
    category
    ttl
    dependencies
    security_class
    reason
}
35. Cacheability Must Be Deterministic

La misma solicitud y contexto deben producir la misma decisión de elegibilidad.

No debe depender de:

random behavior

ni de:

AI-generated decision

sin policy determinística.

36. AI and Cache Eligibility

Los AI agents están sujetos a las mismas reglas.

Un agente AI no puede:

request ALLOW
→ cache it
→ use it after policy/security context changes

La cache eligibility será determinada por el mismo Authorization Runtime.

AI no puede solicitar:

CACHE_FORCED

ni:

TTL_OVERRIDE

para ampliar privilegios.

37. Negative Caching

El sistema podrá utilizar:

DENY caching

para reducir abuso y carga.

Sin embargo, debe protegerse contra:

stale deny

porque un usuario al que se le concedió posteriormente un permiso podría continuar recibiendo:

DENY

hasta la expiración/invalidation.

Por ello también requiere:

permission_epoch
role_epoch
membership_epoch
policy_epoch

cuando sean relevantes.

38. Cache Poisoning

El resultado cacheado sólo podrá generarse después de una evaluación válida.

Nunca:

client → Redis → ALLOW

ni:

AI Agent → cache write

La escritura de cache debe pertenecer al runtime autorizado.

39. Cache Write Rules

Una entrada sólo puede almacenarse cuando:

decision evaluation succeeded
AND
cache eligibility == true
AND
required versions available
AND
context fingerprint valid
AND
TTL valid

Si cualquiera falla:

do not cache
40. Cache Read Rules

Para utilizar una entrada:

entry exists
AND
not expired
AND
tenant matches
AND
versions match
AND
context matches
AND
runtime compatible
AND
decision still eligible

Si no:

cache miss / stale
→ fresh evaluation
41. Cache Invalidation Priority

Los mecanismos tendrán esta prioridad conceptual:

1. Security revocation
2. Version/Epoch mismatch
3. Explicit event invalidation
4. Policy expiration
5. TTL expiration

Un evento de seguridad no debe quedar esperando al TTL.

42. Cache Stampede

Cuando una entrada expira:

1000 requests
   ↓
cache miss

no deberán generar necesariamente:

1000 policy evaluations

Se podrán utilizar:

single-flight;
request coalescing;
locking;
background refresh.

La implementación exacta queda pendiente.

43. Grace Period

No se establece un grace period general para stale authorization.

Por defecto:

stale
→ invalid

No:

stale
→ temporarily accepted

Cualquier excepción deberá estar explícitamente definida por una política de seguridad.

44. Fail-Closed

Si el sistema no puede determinar:

eligibility
TTL
version
context
tenant

de manera confiable:

do not use cache

y:

fresh authorization

o una decisión segura según el runtime.

Nunca:

unknown → ALLOW
45. Performance Target

El decision cache debe contribuir al objetivo de:

Authorization <50ms

y el permission resolution al objetivo:

<30ms

establecidos para el dominio de seguridad.

Sin embargo:

Performance nunca tendrá precedencia sobre seguridad.

46. Observability

Se deberán medir:

cache_eligible_total
cache_ineligible_total
cache_hit_total
cache_miss_total
cache_stale_total
cache_expired_total
cache_bypass_total
cache_invalidation_total
cache_security_bypass_total

Además:

cache_hit_by_decision
cache_hit_by_security_class
cache_bypass_by_reason

Esto permitirá detectar si el caching está:

siendo demasiado agresivo;
siendo demasiado conservador;
provocando stale decisions;
degradando performance.
47. Security Events

Una anomalía de cache que pueda afectar seguridad deberá ser observable como security event cuando corresponda.

Ejemplos:

AuthorizationCacheVersionMismatch
AuthorizationCacheCorruption
AuthorizationCacheTenantMismatch
AuthorizationCacheSecurityBypass

No se deben registrar secretos.

48. Testing Requirements

Se deberán probar especialmente:

Test 1 — Role Revocation
ALLOW cached
↓
role removed
↓
epoch changes
↓
ALLOW rejected
Test 2 — MFA
ALLOW with MFA
↓
MFA evidence expires
↓
cached ALLOW rejected
Test 3 — Ownership
Owner A
↓
ALLOW
↓
ownership changes to B
↓
old ALLOW rejected
Test 4 — Delegation
Delegation active
↓
ALLOW
↓
delegation revoked
↓
old ALLOW rejected
Test 5 — Risk
risk LOW
↓
ALLOW
↓
risk HIGH
↓
old decision not reused
Test 6 — Tenant Isolation
Tenant A ALLOW
↓
Tenant B request
↓
cache entry rejected
Test 7 — Runtime Version
runtime v3
↓
cached ALLOW
↓
runtime v4 incompatible
↓
cache rejected
Test 8 — Critical Operation
critical operation
↓
decision cache
↓
must bypass/reject according to policy
49. Consequences
Positives
Security

Reduce el riesgo de:

stale authorization;
privilege escalation;
MFA bypass;
delegation abuse;
cross-tenant leakage.
Performance

Permite caching agresivo únicamente donde sea seguro.

Flexibility

No obliga a tratar todas las decisiones igual.

Governance

Permite que políticas críticas prohíban caching explícitamente.

50. Negative Consequences

Aumenta la complejidad del sistema:

decision eligibility;
TTL calculation;
security classification;
context analysis;
cache invalidation;
testing;
observability.

También puede reducir el cache hit rate en operaciones críticas.

Esto es aceptable porque:

Security-critical authorization tiene prioridad sobre cache efficiency.

51. Alternatives Rejected
A. Cache Everything

Rechazado.

Demasiado riesgo de stale authorization.

B. Never Cache Decisions

Rechazado como estrategia global.

Limitaría innecesariamente performance, aunque podrá aplicarse a categorías críticas.

C. Same TTL for All Decisions

Rechazado.

No todas las decisiones tienen el mismo riesgo.

D. TTL Only

Rechazado.

No proporciona suficiente protección frente a revocaciones.

E. Cache Only ALLOW

Rechazado como regla general.

DENY también puede ser cacheable bajo condiciones.

F. Cache Only DENY

Rechazado.

No proporciona el beneficio de performance esperado para autorizaciones frecuentes.

52. Non-Negotiable Rules
Unknown eligibility = no cache.
Critical security operations prefer fresh authorization.
TTL nunca sustituye versioning/invalidation.
Stale ALLOW nunca es aceptable.
MFA no puede ser bypass mediante cache.
Revoked delegation no puede seguir autorizando.
Ownership changes deben invalidar decisiones afectadas.
Risk-sensitive decisions requieren contexto válido.
Cross-tenant reuse está prohibido.
AI agents siguen exactamente las mismas reglas.
REVOKE no es una decisión cacheable normal.
REAUTHENTICATE debe preferir evaluación fresh.
REQUIRE_MFA nunca puede convertirse en ALLOW por cache.
Policy puede prohibir caching.
Organization no puede debilitar platform security minimums.
Cache failure debe fail closed.
No secrets en cache.
Decision cache no sustituye Authorization Runtime.
Cache writes sólo después de evaluación válida.
Security siempre tiene prioridad sobre performance.
53. Traceability
Blueprint

Relacionado con:

Identity Domain;
Security & Permissions;
RBAC;
ABAC;
Tenant Isolation;
Resource Ownership;
Device Trust;
Risk;
Session State;
Security Policy Engine;
Event-Driven Architecture;
Observability;
Performance.

El Blueprint establece una autorización que combina RBAC, ABAC y políticas contextuales, considerando organización/tenant, ownership, device trust, risk y session state.

También define el objetivo de Authorization <50 ms y Permission Resolution <30 ms, haciendo relevante el caching como mecanismo de performance, pero subordinado a los controles de seguridad.

Implementation Stories

Principalmente:

IS-006 — Authorization & Permission Evaluation
IS-007 — Role & Permission Management
IS-008 — Session Management
IS-009 — Refresh Token Lifecycle
IS-010 — Multi-Factor Authentication
IS-011 — Security Policy Engine
IS-012 — Audit & Security Events
IS-013 — Domain Events & Outbox
IS-015 — Identity Persistence
IS-016 — Identity Testing & Quality
IS-017 — Identity Runtime & Deployment
54. Related ADRs
ADR-IDENTITY-005 — Authorization Model
ADR-IDENTITY-007 — Transactional Outbox
ADR-IDENTITY-010 — Testing & Quality
ADR-IDENTITY-019 — Partitioned Event Streaming
ADR-IDENTITY-020 — Event Schema Registry
ADR-IDENTITY-021 — Controlled Replay
ADR-IDENTITY-022 — Consumer Idempotency
ADR-IDENTITY-028 — Observability & Operational Security
ADR-IDENTITY-032 — Rate Limiting & Abuse
ADR-IDENTITY-033 — Account Lifecycle & Lockout
ADR-IDENTITY-043 — Multi-Organization Identity
ADR-IDENTITY-044 — Membership Roles & Delegation
ADR-IDENTITY-045 — Authorization Policy Evaluation
ADR-IDENTITY-046 — Policy Language & Governance
ADR-IDENTITY-047 — Policy Engine Technology
ADR-IDENTITY-048 — Authorization Policy DSL
ADR-IDENTITY-049 — Authorization Evaluation Runtime
ADR-IDENTITY-050 — Cache, Invalidation & Consistency
ADR-IDENTITY-051 — Cache Key Schema, Versioning & Context Fingerprinting
55. Pending Decisions

Todavía deberán definirse en especificaciones posteriores:

TTL exacto por security class.
TTL máximo para ALLOW.
TTL máximo para DENY.
TTL específico de MFA evidence.
TTL de step-up.
TTL de device trust.
TTL de delegation.
TTL de risk context.
Time-policy cache strategy.
Decision eligibility matrix completa por recurso/acción.
Security classification catalog.
Policy metadata schema para cacheability.
Cache bypass mechanism.
Exact cache eligibility evaluator.
Single-flight implementation.
Cache stampede protection.
Redis eviction policy.
Cross-region consistency.
Cache warming.
Cache serialization.
Security-sensitive operation catalog.
Exact audit/security event contracts.
56. Acceptance Criteria
AC-01

Una decisión desconocida como cacheable deberá tratarse como no cacheable.

AC-02

Una operación crítica de Identity no podrá reutilizar una decisión cacheada si la política exige fresh authorization.

AC-03

Una ALLOW stale deberá rechazarse y reevaluarse.

AC-04

Una revocación de permiso deberá invalidar las decisiones afectadas.

AC-05

Una revocación de membership deberá invalidar las decisiones afectadas.

AC-06

Una revocación de delegation deberá invalidar sus decisiones.

AC-07

Una expiración de MFA/step-up no podrá producir una ALLOW cacheada inválida.

AC-08

Un cambio relevante de risk deberá impedir reutilización incorrecta.

AC-09

Un cambio de ownership deberá invalidar las decisiones afectadas.

AC-10

REQUIRE_MFA nunca podrá convertirse en ALLOW por cache.

AC-11

REAUTHENTICATE no podrá utilizarse para evitar una autenticación requerida.

AC-12

REVOKE no deberá tratarse como una decisión cacheable normal.

AC-13

Una policy con CACHE_FORBIDDEN deberá impedir decision caching.

AC-14

Una organización no podrá debilitar una restricción de caching impuesta por platform security.

AC-15

AI agents deberán respetar exactamente la misma eligibility policy.

AC-16

El TTL efectivo nunca podrá superar el límite de seguridad de cualquiera de sus dependencias.

AC-17

Cache failure no podrá producir ALLOW.

AC-18

No podrán almacenarse secretos en las entradas de decision cache.

AC-19

La estrategia deberá ser compatible con los objetivos de performance de Authorization.

AC-20

Las pruebas deberán demostrar ausencia de stale privilege después de cambios críticos de seguridad.

57. Architectural Result

Con ADR-050, ADR-051 y ADR-052 queda establecida una arquitectura coherente:

                         PostgreSQL
                      Source of Truth
                            │
                            ▼
                  Security Versions
                  + Policy Versions
                            │
                            ▼
              ┌─────────────────────────┐
              │ Authorization Context   │
              │                         │
              │ Tenant                  │
              │ Actor                   │
              │ Membership              │
              │ Resource                │
              │ Action                  │
              │ MFA                     │
              │ Session                 │
              │ Ownership               │
              │ Delegation              │
              │ Risk                    │
              │ Environment             │
              └────────────┬────────────┘
                           │
                           ▼
                  Context Fingerprint
                           │
                           ▼
                 Cache Eligibility
                           │
             ┌─────────────┼──────────────┐
             │             │              │
          CACHEABLE   CONDITIONAL     NON-CACHEABLE
             │             │              │
             └─────────────┼──────────────┘
                           ▼
                    TTL Calculation
                           │
                           ▼
                 Authorization Cache
                           │
                    Validity Check
                           │
             ┌─────────────┴────────────┐
             │                          │
           VALID                     STALE
             │                          │
             ▼                          ▼
       Cached Decision            Fresh Evaluation
             │                          │
             └────────────┬─────────────┘
                          ▼
                Authorization Runtime
                          │
                          ▼
       ALLOW / DENY / CHALLENGE / MFA /
       REAUTHENTICATE / REVOKE
Principio arquitectónico final

EVOXA sólo reutilizará una decisión de autorización cuando pueda demostrar que la identidad del cache entry, sus versiones de seguridad, su contexto y su vigencia siguen siendo compatibles con la solicitud actual. Cuando esa demostración no sea posible, se ejecutará una evaluación fresh.

Siguiente ADR recomendado

El siguiente paso lógico es:

ADR-IDENTITY-053 — Authorization Cache Invalidation Event Model & Propagation Protocol

Ahí deberíamos definir exactamente qué eventos invalidan qué caches, incluyendo:

PolicyUpdated
PolicyActivated
PolicyRetired
RolePermissionChanged
MembershipRoleChanged
MembershipRevoked
DelegationRevoked
OwnershipChanged
SessionRevoked
MFAStateChanged
SecurityStateChanged

y establecer para cada uno:

evento → scope → epoch/version → L1 invalidation → Redis invalidation → comportamiento durante propagación → idempotencia → ordering → recovery.

Ese ADR dejaría cerrada la arquitectura de invalidación antes de pasar a los Technical Tasks de implementación.
