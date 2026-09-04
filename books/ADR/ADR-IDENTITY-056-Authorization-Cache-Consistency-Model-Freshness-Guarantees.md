ADR-IDENTITY-056 — Authorization Cache Consistency Model & Freshness Guarantees

Status: Proposed
Version: 1.0
Date: 2026-09-04
Domain: Identity / Authorization
Priority: Critical
Type: Architecture / Security / Distributed Systems / Consistency

1. Context

EVOXA utiliza un Authorization Evaluation Runtime centralizado para resolver decisiones de autorización mediante:

RBAC
ABAC
Tenant Isolation
Resource Ownership
Delegation
Security Policy Engine

Los ADR anteriores establecieron una arquitectura de cache para reducir latencia sin convertir el cache en una autoridad de seguridad:

ADR-IDENTITY-049 — Authorization Policy Evaluation Runtime & Decision Algorithm
ADR-IDENTITY-050 — Authorization Policy Cache, Invalidation & Consistency Strategy
ADR-IDENTITY-051 — Authorization Cache Key Schema, Versioning & Context Fingerprinting
ADR-IDENTITY-052 — Authorization Cache Eligibility, TTL & Security-Sensitive Decision Caching
ADR-IDENTITY-053 — Authorization Cache Invalidation Event Model & Propagation Protocol
ADR-IDENTITY-054 — Authorization Cache Failure, Degradation & Recovery Strategy
ADR-IDENTITY-055 — Authorization Cache Recovery, Reconciliation & Warm-Up Protocol

La arquitectura resultante puede contener:

                    PostgreSQL
                 Source of Truth
                       │
                       ▼
              Transactional Outbox
                       │
                       ▼
                 Event Platform
                       │
                       ▼
              Invalidation Layer
                 │           │
                 ▼           ▼
               L1          L2 Redis
                 │           │
                 └─────┬─────┘
                       ▼
             Authorization Runtime
                       │
                       ▼
                    Decision

El problema pendiente es definir formalmente qué significa que una decisión cacheada sea:

fresca;
suficientemente fresca;
potencialmente obsoleta;
stale;
no confiable;
inválida;
o incapaz de producir un ALLOW.

El Blueprint establece principios de:

seguridad por diseño;
tenant isolation;
autorización centralizada;
fail closed;
consistencia controlada;
eventos at-least-once;
consumidores idempotentes;
replay;
observabilidad.

Por lo tanto, no es suficiente establecer un TTL genérico.

2. Problem Statement

El sistema debe responder de manera determinista preguntas como:

¿Cuánto tiempo puede utilizarse una decisión cacheada?

¿Qué ocurre inmediatamente después de revocar un permiso?

¿Qué ocurre si el evento de invalidación todavía no llegó?

¿Qué ocurre si el TTL todavía no expiró pero el security epoch cambió?

¿Qué operaciones requieren siempre una evaluación fresca?

¿Cuándo una decisión DENY también debe considerarse stale?

¿Puede una operación de bajo riesgo utilizar una autorización ligeramente antigua?

¿Cuál es el máximo tiempo aceptable durante el cual un cambio de autorización puede no reflejarse en cache?

Estas preguntas deben resolverse mediante un modelo explícito de consistencia y freshness.

3. Decision

EVOXA adoptará un modelo de:

Scoped Authorization Consistency with Security-First Freshness Guarantees

Esto significa que no todas las decisiones requieren exactamente el mismo nivel de consistencia, pero las operaciones de seguridad crítica tendrán garantías de freshness superiores a las operaciones ordinarias.

La consistencia será determinada por una combinación de:

Decision
=
Cache Eligibility
+
TTL
+
Security Epoch
+
Entity Version
+
Policy Version
+
Context Fingerprint
+
Operation Criticality
+
Invalidation State
+
Authorization State
4. Core Consistency Model

Se establecen cuatro estados conceptuales:

FRESH
  ↓
ACCEPTABLE
  ↓
STALE
  ↓
INVALID

Pero estos estados no representan únicamente tiempo.

Una entrada puede ser:

TTL válido

y simultáneamente:

Security Epoch inválido

En ese caso:

INVALID
5. Fresh

Una entrada será considerada FRESH cuando cumpla todos los requisitos aplicables:

TTL vigente;
security epoch vigente;
entity version vigente;
policy version compatible;
tenant correcto;
actor correcto;
membership correcto;
context fingerprint compatible;
recurso y acción correctos;
cache generation válida;
no exista invalidación pendiente conocida que la afecte;
operación elegible para cache.

Conceptualmente:

Fresh =
TTL Valid
AND
Version Valid
AND
Epoch Valid
AND
Context Valid
AND
Scope Valid
AND
Policy Valid
AND
Generation Valid
6. Acceptable

Una entrada puede encontrarse dentro de una ventana operacional donde su información todavía podría ser utilizable para determinadas operaciones de bajo riesgo.

Sin embargo:

ACCEPTABLE no significa equivalente a FRESH.

El uso de esta categoría deberá estar explícitamente permitido por la política de cache.

No podrá utilizarse para operaciones clasificadas como security-critical cuando no pueda garantizarse la frescura requerida.

7. Stale

Una entrada es STALE cuando:

su TTL ha expirado;
su versión es inferior a la autoritativa;
su epoch es inferior;
su policy version ya no corresponde;
su contexto ya no es válido;
existe una invalidación aplicable;
o se ha superado el freshness bound permitido.

Una entrada stale:

MUST NOT

producir un ALLOW en una operación que requiera freshness superior.

8. Invalid

Una entrada será INVALID cuando exista evidencia suficiente de que no representa el estado actual o no puede confiarse en ella.

Ejemplos:

Tenant mismatch
Security epoch regression
Corrupt cache entry
Policy version mismatch
Revoked membership
Revoked session

En estos casos:

Discard
+
Invalidate
+
Fresh Evaluation

cuando corresponda.

9. Strong Consistency for Critical Security State

Las siguientes categorías requerirán un modelo de consistencia fuerte o equivalente mediante validación autoritativa:

account disable;
account suspension;
membership revocation;
role removal;
permission removal;
session revocation;
refresh token family revocation;
MFA factor revocation;
security policy tightening;
delegation revocation;
cross-tenant authorization;
administración de Identity;
cambios de privilegio;
operaciones clasificadas como críticas.

La intención es:

Security-critical state
        ↓
Fresh authoritative validation
10. Eventual Consistency for Non-Critical Cache Data

Las decisiones que no involucren un riesgo de seguridad elevado podrán utilizar un modelo de consistencia eventual controlada cuando:

la entrada sea elegible;
TTL sea válido;
versions sean válidas;
epochs sean válidos;
tenant context sea válido;
no exista invalidación crítica pendiente;
la política permita el uso cacheado.

Esto permite aprovechar:

L1
+
L2 Redis

sin exigir una consulta a PostgreSQL en cada request.

11. Freshness Is Not Only TTL

Este ADR establece explícitamente:

TTL es una condición necesaria en determinados escenarios, pero no es suficiente para determinar freshness.

Ejemplo:

Cache created:
10:00

TTL:
10 minutes

Permission revoked:
10:02

A las 10:05:

TTL = valid

pero:

Authorization = stale

si el security epoch/version ha cambiado.

Por tanto:

TTL Valid
+
Security Version Invalid
=
INVALID
12. Maximum Authorization Staleness

Se introduce el concepto:

Maximum Authorization Staleness — MAS

Representa el máximo período durante el cual una decisión cacheada podría potencialmente no reflejar el estado autoritativo actual para una determinada categoría de operación.

Conceptualmente:

MAS =
maximum allowed uncertainty window

El valor exacto deberá definirse por categoría de autorización y no como un único valor global.

13. Freshness Classes

Se establecen inicialmente cuatro clases conceptuales.

Clase	Consistencia	Uso
F0	Autoritativa/Fresh	Operaciones críticas
F1	Strongly validated	Security-sensitive
F2	Controlled eventual	Operaciones normales
F3	Restricted	Degradación/uncertainty
F0

Requiere estado autoritativo o equivalente validado.

F1

Puede utilizar mecanismos optimizados siempre que las garantías de security epoch/version se mantengan.

F2

Puede utilizar cache bajo reglas normales.

F3

El uso del cache queda altamente restringido y puede requerir:

fresh evaluation;
MFA;
reauthentication;
DENY.
14. Operation Criticality

La freshness no será determinada únicamente por el recurso.

Debe considerar:

Resource
+
Action
+
Actor
+
Tenant
+
Context
+
Risk
+
Operation Criticality

Por ejemplo:

users.read

puede requerir menos consistencia que:

users.disable

aunque ambos pertenezcan al mismo recurso.

15. Read vs Write

Las operaciones de escritura tendrán normalmente requisitos de freshness superiores a lecturas.

Especialmente:

Grant Permission
Revoke Permission
Change Role
Disable User
Revoke Session
Change MFA
Change Policy

no deberán depender de información cacheada potencialmente stale para determinar si la operación está autorizada.

16. ALLOW Freshness

El ALLOW es el resultado que presenta mayor riesgo cuando existe información stale.

Por lo tanto:

Potentially stale ALLOW
        ↓
Not trusted for critical operation

El sistema nunca debe asumir:

No invalidation observed
=
No authorization change
17. DENY Freshness

Los DENY también requieren freshness.

Ejemplo:

Cached DENY

y posteriormente:

Role Granted

Si el DENY permanece indefinidamente en cache, el usuario continuará sin acceso aunque ahora esté autorizado.

Por lo tanto:

DENY

también está sujeto a:

TTL;
versions;
epochs;
invalidation;
context.
18. Security Revocation Priority

Las revocaciones deberán tener garantías de propagación superiores a las concesiones de privilegios.

Conceptualmente:

REVOKE
   >
GRANT

en cuanto a prioridad de consistencia.

La razón es minimizar la ventana durante la cual una autorización revocada podría continuar siendo utilizable.

Esto no significa que los grants puedan permanecer indefinidamente stale.

19. Authorization Freshness Hierarchy

La jerarquía será:

Security Revocation
        ↓
Security Epoch
        ↓
Policy Version
        ↓
Membership/Role/Permission Version
        ↓
Context Validity
        ↓
TTL

La invalidez en una capa superior domina a las inferiores.

Ejemplo:

TTL valid
Policy version stale

resultado:

INVALID
20. Freshness Validation Pipeline

Antes de aceptar una decisión cacheada:

Cache Lookup
     ↓
Tenant Validation
     ↓
Actor Validation
     ↓
Context Fingerprint
     ↓
Cache Generation
     ↓
Security Epoch
     ↓
Entity Version
     ↓
Policy Version
     ↓
TTL
     ↓
Eligibility
     ↓
Authorization Runtime
     ↓
Decision

No necesariamente todos los pasos requerirán una consulta externa en cada request; el objetivo es que el Runtime pueda demostrar que la entrada cumple las garantías necesarias.

21. Invalidation Interaction

ADR-053 estableció que los eventos proporcionan propagación y que versions/epochs proporcionan seguridad contra stale state.

Este ADR establece:

Invalidation received
      ↓
Entry immediately untrusted

Cuando una invalidación aplicable es conocida, el cache no deberá continuar utilizándose simplemente porque el TTL no haya expirado.

22. Invalidation Lag

Si existe:

Event creation
      ↓
      │ lag
      ↓
Cache invalidation

existe potencialmente una ventana de inconsistencia.

Por ello el sistema deberá medir:

Propagation Lag

y compararlo con el freshness requirement de la operación.

23. Freshness Budget

Cada clase de operación podrá tener un:

Freshness Budget

Conceptualmente:

Freshness Budget
=
maximum tolerated authorization uncertainty

Ejemplo conceptual:

Critical operation
Budget = near-zero

Normal operation
Budget = controlled finite window

Los valores exactos no se fijan en este ADR.

24. Zero-Staleness Security Operations

Algunas operaciones podrán requerir:

Zero-Staleness Guarantee

Esto significa que antes de producir un ALLOW deberá existir una validación suficientemente cercana al estado autoritativo.

Ejemplos potenciales:

cambio de permisos;
administración de roles;
desactivación de cuentas;
administración de tenant;
recovery administrativo;
operaciones cross-tenant.

El catálogo definitivo queda pendiente.

25. Context Freshness

Una decisión puede tener una policy correcta y aun así ser inválida porque el contexto cambió.

Ejemplo:

IP
Device
Risk
MFA
Session
Ownership
Location

pueden cambiar sin modificar la policy.

Por ello:

Policy Fresh
≠
Decision Fresh

El contexto debe formar parte de la evaluación de freshness cuando sea relevante.

26. MFA Freshness

Una evidencia de MFA debe tener su propia validez temporal.

Por ejemplo:

MFA verified

no implica:

MFA always valid

El sistema debe considerar:

authentication method;
MFA state;
step-up evidence;
evidence age;
session;
security policy.

Una decisión cacheada que requiere MFA no puede utilizar evidencia MFA expirada.

27. Session Freshness

Una decisión cacheada debe poder quedar invalidada cuando:

session revoked;
session expired;
account suspended;
security incident;
authentication assurance changes.

Por tanto:

Valid Authorization Cache
+
Revoked Session
=
INVALID
28. Membership Freshness

La membership es una frontera fundamental de tenant.

Si:

Membership revoked

entonces cualquier decisión dependiente de ella debe considerarse inválida.

No debe existir:

Old Membership
      ↓
Cached ALLOW

después de la revocación efectiva.

29. Role and Permission Freshness

Los cambios de:

role assignment;
role removal;
permission grant;
permission revoke;

deben producir actualización de versiones/epochs e invalidación según el alcance correspondiente.

Una decisión cacheada no debe permanecer válida sólo porque su TTL todavía no expiró.

30. Policy Freshness

Una policy deberá asociarse a:

Policy ID
Policy Version
Status
Effective Period

Una decisión basada en:

Policy Version 5

no debe utilizarse si:

Current Active Version = 6
31. Ownership Freshness

Si la autorización depende de ownership:

resource.owner_id

un cambio de propietario puede invalidar decisiones previamente cacheadas.

Por tanto:

Ownership Change
      ↓
Authorization Invalidation
32. Delegation Freshness

Las delegaciones requieren:

validity period;
status;
scope;
delegator;
delegatee;
permission boundary.

Una delegation expirada o revocada debe invalidar decisiones que dependan de ella.

33. Cross-Tenant Freshness

Las operaciones cross-tenant tendrán requisitos de freshness máximos.

Si no puede validarse:

Source Tenant
Target Tenant
Membership
Delegation
Policy

el resultado será:

DENY

o una respuesta segura equivalente según la política.

34. Cache Freshness by Layer
L1

Debe respetar:

local generation;
version;
epoch;
TTL;
invalidation state.
L2

Debe respetar:

Redis namespace;
generation;
version;
epoch;
TTL.
PostgreSQL

Es la fuente autoritativa para el estado transaccional.

35. Freshness and Recovery

Durante los estados definidos por ADR-055:

DEGRADED
RECONCILING
INVALIDATING
REBUILD
WARM-UP

las garantías de cache no deben considerarse normales.

La operación crítica debe utilizar:

Fresh Evaluation

hasta que el sistema alcance:

VALIDATED

y posteriormente:

RECOVERED
36. Freshness During Event Consumer Outage

Si el consumer está detenido:

Event Stream
   ↓
Backlog

el sistema debe asumir que determinadas entradas pueden ser stale.

La ausencia de procesamiento de invalidaciones no debe interpretarse como ausencia de cambios.

37. Freshness During Redis Outage

Redis unavailable:

Cache unavailable

no implica automáticamente que Identity esté unavailable.

Puede realizarse:

Fresh Evaluation

cuando la infraestructura autoritativa esté disponible.

38. Freshness During PostgreSQL Outage

Si PostgreSQL no está disponible:

Cannot obtain authoritative state

se podrá utilizar cache únicamente cuando la entrada satisfaga completamente las garantías requeridas para esa operación.

Para operaciones F0/F1 críticas:

No trusted fresh state
        ↓
DENY
39. Freshness and Read Availability

El sistema puede preferir:

Read availability

para operaciones de bajo riesgo cuando el estado cacheado todavía cumple las garantías permitidas.

Pero para operaciones críticas:

Security correctness
>
Read availability
40. Freshness and Write Availability

Para operaciones que modifican autorización:

Security correctness
>
Write availability

No se debe aceptar una escritura de alto impacto basándose exclusivamente en información de autorización que no pueda demostrarse fresca.

41. Stale-While-Revalidate

El patrón:

Serve stale
+
Refresh asynchronously

no será permitido para ALLOW en operaciones security-critical.

Puede utilizarse solamente cuando:

la operación sea elegible;
la política lo permita;
el stale window esté dentro del límite;
no exista revocación/security epoch conflict;
no exista incertidumbre crítica.
42. Fail-Closed Boundary

El sistema deberá identificar claramente el punto donde deja de ser aceptable la consistencia eventual:

Freshness uncertainty
        ↓
Allowed?
   ┌────┴────┐
  YES        NO
   │          │
Continue     DENY

Nunca:

UNKNOWN → ALLOW
43. Consistency Guarantees

Se establecen las siguientes garantías conceptuales:

Guarantee G1

Una entrada con security epoch inferior no puede producir un ALLOW crítico.

Guarantee G2

Una entrada con policy version incompatible no puede producir un ALLOW.

Guarantee G3

Una entrada perteneciente a un tenant incorrecto nunca puede producir ALLOW.

Guarantee G4

Una entrada invalidada explícitamente no puede producir ALLOW.

Guarantee G5

Una sesión revocada no puede reactivarse mediante cache.

Guarantee G6

Una membership revocada no puede mantener autorización mediante cache.

Guarantee G7

Una delegation expirada no puede producir autorización.

Guarantee G8

Una policy retirada no puede utilizarse como policy activa.

Guarantee G9

La incertidumbre sobre freshness nunca puede producir privilege escalation.

Guarantee G10

La recuperación no puede degradar las garantías de seguridad establecidas.

44. Freshness Decision Algorithm

Conceptualmente:

function evaluate_cache(entry, request):

    if entry is missing:
        return CACHE_MISS

    if tenant_invalid(entry, request):
        return INVALID

    if context_invalid(entry, request):
        return INVALID

    if generation_invalid(entry):
        return INVALID

    if security_epoch_invalid(entry):
        return INVALID

    if entity_version_invalid(entry):
        return INVALID

    if policy_version_invalid(entry):
        return INVALID

    if ttl_expired(entry):
        return STALE

    if not eligible_for_operation(entry, request):
        return BYPASS

    if freshness_requirement_not_met(entry, request):
        return FRESH_EVALUATION

    return VALID

La implementación final deberá integrarse con ADR-049 y ADR-052.

45. Decision Matrix
Estado	Operación normal	Operación sensible	Operación crítica
FRESH	ALLOW/DENY según Runtime	permitido si elegible	permitido sólo si cumple F0/F1
ACCEPTABLE	permitido según política	normalmente fresh	fresh
STALE	fresh recomendado	fresh obligatorio	fresh obligatorio
INVALID	fresh	fresh	DENY/fresh
UNKNOWN	restringido	fresh	DENY
Epoch mismatch	invalidar	fresh	DENY/fresh
Tenant mismatch	DENY	DENY	DENY
Policy mismatch	fresh	fresh	DENY/fresh
Security state unknown	restringido	DENY/fresh	DENY
46. Observability

Se deberán medir:

Freshness
cache freshness age;
stale decisions;
invalid decisions;
freshness violations;
maximum observed staleness;
freshness budget violations.
Consistency
epoch mismatch;
policy version mismatch;
entity version mismatch;
context mismatch;
tenant mismatch.
Authorization
cached ALLOW;
cached DENY;
fresh ALLOW;
fresh DENY;
cache bypass;
uncertainty DENY.
Infrastructure
invalidation lag;
consumer lag;
Redis latency;
PostgreSQL latency;
cache recovery state.
47. Security Monitoring

Debe generarse una señal cuando:

stale ALLOW sea detectado;
security epoch mismatch ocurra;
cache entry pertenezca a tenant incorrecto;
policy version antigua intente utilizarse;
revoked session aparezca como autorizada;
revoked membership aparezca como autorizada;
freshness guarantees sean violadas;
existan repetidos cache mismatches.

Los incidentes potencialmente relacionados con seguridad deberán poder correlacionarse con:

audit events;
security events;
trace IDs;
correlation IDs.
48. Testing Requirements

Se deberán crear pruebas para:

TTL
before expiration;
exact expiration;
after expiration.
Versions
same version;
newer version;
older version;
regression.
Epoch
same epoch;
newer epoch;
older epoch.
Policy
active;
superseded;
retired;
future effective date.
Membership
active;
revoked;
suspended;
changed tenant.
Session
active;
expired;
revoked.
MFA
valid;
expired;
revoked;
step-up required.
Delegation
active;
expired;
revoked;
scope mismatch.
Failure
Redis unavailable;
consumer unavailable;
event lag;
PostgreSQL unavailable.
49. Chaos Testing

La arquitectura deberá probar:

Redis failure
Event Platform failure
Consumer pause
Network partition
PostgreSQL failover
Delayed invalidation
Duplicate events
Out-of-order events
Cache corruption
Cache loss
Regional partition

La propiedad fundamental que deberá comprobarse es:

No unauthorized ALLOW
50. Performance Impact

El modelo de freshness debe respetar los objetivos de rendimiento definidos para Identity.

Se debe evitar consultar PostgreSQL para cada request de bajo riesgo.

La arquitectura debe permitir:

Normal Request
   ↓
L1/L2
   ↓
Fast Authorization

mientras mantiene:

Critical Request
   ↓
Fresh Validation

cuando sea necesario.

El objetivo de performance del Authorization Runtime establecido previamente continúa vigente.

51. AI Authorization Freshness

Los agentes AI estarán sujetos a las mismas garantías.

Una IA no podrá decir:

Cache stale
→
Assume permission

ni:

PostgreSQL unavailable
→
Continue privileged operation

Los agentes deberán recibir:

same authorization decision
+
same freshness guarantees

que cualquier otro actor.

52. Alternatives Considered
Alternative A — Strong Consistency Everywhere

Rejected.

Obligaría a consultar estado autoritativo constantemente y reduciría innecesariamente:

performance;
scalability;
availability.
Alternative B — Eventual Consistency Everywhere

Rejected.

No proporciona garantías suficientes para:

revocations;
security changes;
tenant isolation;
privileged operations.
Alternative C — TTL Only

Rejected.

TTL no detecta inmediatamente:

revocations;
policy changes;
membership changes;
security epoch changes.
Alternative D — ALLOW Cache Without Validation

Rejected.

Podría producir privilege escalation.

Alternative E — DENY on Every Cache Uncertainty

Rejected as universal behavior.

Aunque seguro, produciría una degradación excesiva para operaciones no críticas.

Se adopta un modelo escalonado.

53. Non-Negotiable Rules
TTL no define por sí solo freshness.
Security epoch tiene prioridad sobre TTL.
Policy version tiene prioridad sobre TTL.
Tenant mismatch siempre produce DENY.
Revoked membership invalida autorizaciones dependientes.
Revoked session invalida autorizaciones dependientes.
Revoked delegation invalida autorizaciones dependientes.
Expired MFA evidence no puede satisfacer un requisito MFA.
Stale ALLOW no puede utilizarse en operaciones críticas.
Stale DENY debe poder invalidarse.
UNKNOWN no puede convertirse en ALLOW.
Cache no es source of truth.
PostgreSQL continúa siendo la fuente autoritativa.
Event lag debe ser observable.
Recovery debe respetar las mismas garantías de freshness.
AI agents no pueden reducir los requisitos de freshness.
Cross-tenant authorization requiere garantías superiores.
Security revocations tienen prioridad de propagación.
No debe existir privilege amplification durante fallback.
Las garantías de freshness deben ser verificables mediante pruebas y observabilidad.
54. Traceability
Blueprint

Este ADR implementa y refuerza:

Identity
Authorization
RBAC
ABAC
Tenant Isolation
Resource Ownership
Delegation
Security Policy Engine
Event-Driven Architecture
At-Least-Once Delivery
Idempotent Consumers
Replay
Observability
High Availability
Disaster Recovery
Secure by Design
Implementation Stories

Principalmente:

IS-IDENTITY-006 — Authorization & Permission Evaluation
IS-IDENTITY-007 — Role & Permission Management
IS-IDENTITY-008 — Session Management
IS-IDENTITY-010 — Multi-Factor Authentication
IS-IDENTITY-011 — Security Policy Engine
IS-IDENTITY-012 — Audit & Security Events
IS-IDENTITY-013 — Domain Events & Outbox
IS-IDENTITY-015 — Identity Persistence
IS-IDENTITY-016 — Identity Testing & Quality
IS-IDENTITY-017 — Identity Runtime & Deployment
55. Related ADRs

Especialmente:

ADR-IDENTITY-043 — Multi-Organization Identity
ADR-IDENTITY-045 — Authorization Policy Model & Evaluation
ADR-IDENTITY-046 — Policy Language & Governance
ADR-IDENTITY-047 — Policy Engine Implementation
ADR-IDENTITY-048 — Authorization Policy DSL
ADR-IDENTITY-049 — Authorization Evaluation Runtime
ADR-IDENTITY-050 — Authorization Cache Strategy
ADR-IDENTITY-051 — Cache Key, Versioning & Context Fingerprinting
ADR-IDENTITY-052 — Cache Eligibility, TTL & Security-Sensitive Decision Caching
ADR-IDENTITY-053 — Cache Invalidation & Propagation
ADR-IDENTITY-054 — Cache Failure & Degradation
ADR-IDENTITY-055 — Cache Recovery & Warm-Up
56. Pending Decisions

Este ADR establece el modelo, pero deja deliberadamente abiertos los parámetros concretos.

56.1 Exact Maximum Authorization Staleness

Definir valores por:

operación;
recurso;
action;
riesgo;
tenant;
contexto.
56.2 Exact Freshness Classes

Definir formalmente:

F0
F1
F2
F3

y sus garantías técnicas.

56.3 Critical Operation Catalog

Definir la lista oficial de operaciones que requieren:

ZERO / NEAR-ZERO STALENESS
56.4 Security Revocation SLA

Definir el máximo tiempo permitido entre:

Revocation
   ↓
No longer usable authorization
56.5 Grant Propagation SLA

Definir el tiempo máximo esperado entre:

Grant
   ↓
Authorization reflects grant
56.6 Event Propagation SLA

Definir:

target latency;
maximum latency;
alert threshold;
fail-safe threshold.
56.7 Exact TTL

Definir TTL por categoría de cache.

56.8 Exact Context Freshness

Determinar qué atributos deben ser:

evaluated every request;
fingerprinted;
versioned;
cached.
56.9 Cross-Region Freshness

Definir garantías durante:

network partition;
regional failover;
replication lag.
57. Acceptance Criteria
AC-01 — TTL

Given cache TTL has expired
When authorization is requested
Then the entry must not be treated as fresh.

AC-02 — Epoch

Given cache security epoch is older than authoritative epoch
When authorization is requested
Then the cached result must be invalidated.

AC-03 — Policy Version

Given cached decision references an outdated policy version
When authorization is requested
Then fresh evaluation must occur when required.

AC-04 — Tenant Isolation

Given cache context belongs to another tenant
When authorization is evaluated
Then the result must be DENY.

AC-05 — Membership Revocation

Given membership is revoked
When cached authorization is evaluated
Then the old authorization must not remain valid for critical operations.

AC-06 — Session Revocation

Given session is revoked
When cached authorization is evaluated
Then the revoked session cannot be reactivated through cache.

AC-07 — Delegation Expiration

Given delegation has expired
When cached authorization is evaluated
Then the delegation cannot produce ALLOW.

AC-08 — MFA Freshness

Given MFA evidence has expired
When policy requires MFA
Then the cached authorization cannot satisfy the MFA requirement.

AC-09 — Stale DENY

Given user receives a new permission
When cached DENY becomes stale
Then fresh evaluation must be possible.

AC-10 — Event Lag

Given invalidation events are delayed
When a security-critical operation occurs
Then the system must not rely on potentially stale authorization.

AC-11 — PostgreSQL Failure

Given PostgreSQL is unavailable
When a critical authorization requires fresh state
Then the system must fail closed.

AC-12 — Redis Failure

Given Redis is unavailable
When authorization is requested
Then the system must use a safe alternative or fail closed.

AC-13 — Recovery

Given cache recovery is underway
When authorization is requested
Then critical operations must respect recovery freshness restrictions.

AC-14 — Version Regression

Given an older version arrives
When the cache processes it
Then it must not replace newer authorization state.

AC-15 — Context Change

Given a context attribute relevant to authorization changes
When authorization is requested
Then the previous decision must not be reused if its context fingerprint is no longer valid.

AC-16 — Cross-Tenant

Given cross-tenant context cannot be validated
When authorization is requested
Then the result must be DENY.

AC-17 — AI Agent

Given an AI agent requests a privileged operation
When cache freshness requirements are not satisfied
Then the AI agent must receive the same secure degradation behavior as any other actor.

AC-18 — Freshness Observability

Given an authorization cache becomes stale
When it is detected
Then the event must be observable through appropriate telemetry.

AC-19 — Recovery Integrity

Given cache is rebuilt after failure
When freshness is validated
Then no stale security authorization may be promoted as trusted.

AC-20 — No Unauthorized ALLOW

Given authorization freshness cannot be established
When the operation requires a stronger guarantee
Then the system must not return an unauthorized ALLOW.

58. Architectural Result

Con este ADR, EVOXA establece que consistencia de autorización y freshness son propiedades de seguridad, no simplemente características de rendimiento.

El modelo completo queda:

                    AUTHORIZATION REQUEST
                             │
                             ▼
                 ┌───────────────────────┐
                 │ Authorization Runtime │
                 └───────────┬───────────┘
                             │
                             ▼
                    Cache Eligibility
                             │
                             ▼
                    Freshness Validation
                             │
          ┌──────────────────┼──────────────────┐
          │                  │                  │
          ▼                  ▼                  ▼
       TTL               Version             Epoch
          │                  │                  │
          └──────────────────┼──────────────────┘
                             │
                             ▼
                       Policy Version
                             │
                             ▼
                       Context Validity
                             │
                             ▼
                    Operation Criticality
                             │
              ┌──────────────┴──────────────┐
              │                             │
              ▼                             ▼
        Fresh Enough                    Not Fresh
              │                             │
              ▼                             ▼
       Authorization                 Fresh Evaluation
         Decision                          │
              │                            ▼
              │                     PostgreSQL /
              │                     authoritative state
              │                            │
              └──────────────┬─────────────┘
                             ▼
                          DECISION
                             │
             ┌───────────────┼────────────────┐
             ▼               ▼                ▼
           ALLOW            DENY          MFA/REAUTH

La regla central queda formalizada:

Una autorización cacheada sólo es válida cuando su freshness es suficiente para el nivel de riesgo de la operación y cuando todas las garantías de tenant, security epoch, versiones, policy, contexto y elegibilidad se mantienen satisfechas.

Y la propiedad de seguridad fundamental:

La consistencia eventual puede utilizarse como optimización controlada; nunca puede utilizarse como justificación para permitir una autorización cuya validez de seguridad sea incierta.

59. Next ADR

El siguiente paso natural es:

ADR-IDENTITY-057 — Authorization Cache Freshness Classes, TTL Profiles & Security SLAs

Este ADR debería tomar los conceptos definidos aquí y convertirlos en parámetros normativos concretos, definiendo:

F0/F1/F2/F3;
TTL por tipo de decisión;
Maximum Authorization Staleness;
Security Revocation SLA;
Grant Propagation SLA;
Event Propagation SLA;
operaciones Zero-Staleness;
operaciones cacheables;
operaciones nunca cacheables;
comportamiento por riesgo;
límites de degraded mode;
objetivos medibles para producción.

Ese ADR cerraría la transición desde el modelo conceptual de consistencia hacia las garantías cuantificables que posteriormente podrán convertirse en Engineering Specifications, Technical Tasks y finalmente código.
