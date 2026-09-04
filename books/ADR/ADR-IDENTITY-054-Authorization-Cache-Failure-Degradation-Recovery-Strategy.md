ADR-IDENTITY-054 — Authorization Cache Failure, Degradation & Recovery Strategy

Status: Proposed
Version: 1.0
Date: 2026-09-04
Domain: Identity / Authorization
Priority: Critical
Type: Architecture / Security / Distributed Systems / Resilience

1. Context

EVOXA utiliza un modelo de autorización compuesto por:

RBAC
ABAC
Tenant Isolation
Resource Ownership
Delegation
Security Policy Engine

La decisión de autorización es responsabilidad del Authorization Evaluation Runtime, definido en:

ADR-IDENTITY-045 — Authorization Policy Model & Evaluation
ADR-IDENTITY-046 — Policy Language, Schema & Governance
ADR-IDENTITY-047 — Policy Engine Technology & Implementation
ADR-IDENTITY-048 — Authorization Policy DSL & Expression Language Strategy
ADR-IDENTITY-049 — Authorization Policy Evaluation Runtime & Decision Algorithm

Para cumplir los objetivos de rendimiento, las decisiones y/o resultados derivados de autorización pueden utilizar caché según las reglas establecidas en:

ADR-IDENTITY-050 — Authorization Policy Cache, Invalidation & Consistency Strategy
ADR-IDENTITY-051 — Authorization Cache Key Schema, Versioning & Context Fingerprinting
ADR-IDENTITY-052 — Authorization Cache Eligibility, TTL & Security-Sensitive Decision Caching
ADR-IDENTITY-053 — Authorization Cache Invalidation Event Model & Propagation Protocol

La arquitectura contempla potencialmente:

                 PostgreSQL
              Source of Truth
                    │
                    │
              Domain Change
                    │
                    ▼
            Transactional Outbox
                    │
                    ▼
              Event Platform
                    │
                    ▼
        Cache Invalidation Consumer
             │              │
             ▼              ▼
          L1 Cache        L2 Redis
             │              │
             └──────┬───────┘
                    ▼
       Authorization Runtime
                    │
                    ▼
             ALLOW / DENY /
       CHALLENGE / REAUTHENTICATE /
        REQUIRE_MFA / REVOKE

Esta arquitectura introduce múltiples puntos potenciales de falla.

Por ejemplo:

Redis puede estar temporalmente indisponible.
El cache L1 puede contener información corrupta u obsoleta.
El Event Platform puede estar caído.
El consumidor de invalidaciones puede estar detenido.
Puede existir retraso significativo en la propagación de invalidaciones.
PostgreSQL puede presentar alta latencia o indisponibilidad.
El version store puede no estar disponible.
Puede producirse una cache stampede.
Puede existir una partición de red.
Puede producirse una falla regional.
Puede existir una DLQ acumulada.
Un proceso de recuperación puede intentar reconstruir cachés con información incompleta.

En Identity, una falla de infraestructura no puede transformarse en un bypass de autorización.

Por lo tanto, se necesita una estrategia explícita para determinar qué hacer cuando el sistema de caché, invalidación, persistencia o autorización se encuentra degradado.

2. Problem Statement

El sistema debe responder de manera determinista ante fallas de:

L1 Authorization Cache
L2 Redis Authorization Cache
Event Platform
Invalidation Consumer
PostgreSQL
Version/Epoch Store
Network
Multi-region connectivity
Cache corruption
Cache stampede
DLQ accumulation
Propagation lag
Recovery/reconciliation

La principal pregunta arquitectónica es:

¿Cuándo EVOXA puede utilizar información cacheada, cuándo debe consultar una fuente autoritativa y cuándo debe negar o bloquear una operación?

La respuesta no puede depender únicamente de disponibilidad.

En particular:

Availability ≠ Authorization Safety

Un sistema que permanece disponible pero permite un ALLOW basado en información de autorización potencialmente obsoleta puede constituir una vulnerabilidad de seguridad.

3. Decision

EVOXA adoptará una estrategia de Fail-Closed Authorization Degradation with Controlled Recovery.

La regla fundamental será:

Cuando exista incertidumbre sobre la validez de una autorización, el sistema no podrá convertir dicha incertidumbre en un ALLOW.

La degradación estará basada en niveles de confianza y criticidad de la operación.

4. Principio Fundamental

La jerarquía de confianza será:

                  PostgreSQL
              Authoritative State
                     │
                     ▼
             Authorization Runtime
                     │
          ┌──────────┴──────────┐
          │                     │
       Valid Cache          Fresh Evaluation
          │                     │
          └──────────┬──────────┘
                     ▼
                 Decision

El cache es:

optimización
acelerador
mecanismo de reducción de latencia

El cache no es:

fuente primaria de autorización
autoridad de seguridad
sustituto permanente de PostgreSQL
mecanismo para resolver conflictos de autorización
5. Authorization Runtime Remains the Authority

Aunque el sistema utilice L1/L2 cache, la autoridad lógica para tomar la decisión continúa siendo:

Authorization Evaluation Runtime.

El cache únicamente puede proporcionar datos o resultados que el Runtime considere válidos bajo las reglas de:

versión
epoch
TTL
contexto
tenant
actor
resource
action
security state
criticidad

Por lo tanto:

Cache
  ↓
Validation
  ↓
Authorization Runtime
  ↓
Decision

y nunca:

Cache
  ↓
Direct ALLOW

sin validación de elegibilidad.

6. Failure Classification

Las fallas se clasificarán en:

F1 — Performance Degradation

El sistema funciona, pero:

Redis está lento.
PostgreSQL presenta latencia elevada.
Event propagation está retrasada.

Puede permitirse degradación controlada.

F2 — Cache Unavailability

L1 o L2 no están disponibles.

Debe intentarse una ruta alternativa segura.

F3 — Cache Staleness

Existe información cacheada pero no puede demostrarse que sea suficientemente actual.

No podrá utilizarse para producir un ALLOW cuando la operación sea sensible.

F4 — Authorization State Uncertainty

No puede determinarse con suficiente confianza:

membership
role
permission
policy
delegation
ownership
session
MFA
security state

En este escenario se deberá:

DENY

o requerir una decisión adicional segura:

REQUIRE_MFA
REAUTHENTICATE
CHALLENGE

según corresponda.

F5 — Critical Infrastructure Failure

No existe acceso confiable a las fuentes necesarias para realizar una decisión segura.

Las operaciones críticas deberán:

FAIL CLOSED
7. Authorization Degradation Levels

Se establecen cuatro niveles conceptuales.

Level 0 — Normal

Todos los componentes operan normalmente.

L1 → L2 → PostgreSQL

según elegibilidad y estrategia de cache.

El Runtime puede utilizar decisiones cacheadas válidas.

Level 1 — Partial Degradation

Existe falla de un componente no crítico.

Ejemplos:

L1 indisponible.
Redis temporalmente indisponible.
Event consumer con retraso pequeño.

El sistema puede continuar utilizando rutas alternativas.

Ejemplo:

L1 unavailable
       ↓
L2
       ↓
Authorization Runtime

o:

L2 unavailable
       ↓
PostgreSQL
       ↓
Authorization Runtime

si la operación y la infraestructura lo permiten.

8. Level 2 — Security-Sensitive Degradation

Existe incertidumbre sobre la frescura del estado.

Ejemplos:

Event Platform con lag.
invalidation consumer detenido.
version store temporalmente inaccesible.
cache potencialmente stale.

Para operaciones sensibles:

Do not trust cached ALLOW

Se deberá obtener estado fresco.

Ejemplos de operaciones que pueden clasificarse como sensibles:

cambio de contraseña
modificación de MFA
recuperación de cuenta
administración de roles
modificación de permisos
cambio de membership
acceso administrativo
acciones cross-tenant
modificación de políticas
revocación de seguridad
operaciones de alto impacto

La lista definitiva deberá ser establecida por política.

9. Level 3 — Critical Authorization Failure

Cuando no sea posible establecer una decisión de autorización confiable:

Authorization uncertainty
          ↓
Fail Closed
          ↓
DENY

No se permitirá:

Cache unavailable
      ↓
Assume ALLOW

ni:

Authorization service unavailable
      ↓
Bypass authorization
10. Redis Failure Strategy

Redis será tratado como infraestructura de aceleración y no como fuente autoritativa.

Si Redis está indisponible:

L1 Cache
   ↓
Authorization Runtime
   ↓
PostgreSQL

cuando corresponda.

Si tampoco existe una ruta segura hacia PostgreSQL:

DENY / REQUIRE_MFA / REAUTHENTICATE

dependiendo de la operación.

Nunca:

Redis DOWN → ALLOW everything
11. L1 Cache Failure

La falla de L1 no deberá afectar la corrección de autorización.

El Runtime podrá:

consultar L2;
consultar PostgreSQL;
reconstruir el estado;
continuar sin cache.

Ejemplo:

Request
   │
   ▼
L1
   │
   X
   │
   ▼
L2
   │
   ▼
Runtime

Si L2 también falla:

Request
   │
   ▼
PostgreSQL
   │
   ▼
Runtime
12. L1 Cache Corruption

Si el Runtime detecta:

estructura inválida
versión imposible
tenant inconsistente
fingerprint incorrecto
policy version inválida
epoch regresivo
metadata incompatible

el registro deberá considerarse no confiable.

Acción:

Discard cache entry
        ↓
Invalidate local entry
        ↓
Fresh evaluation

Nunca se intentará "reparar" una entrada potencialmente insegura utilizando otra entrada cacheada no validada.

13. L2 Redis Cache Corruption

Una entrada corrupta en Redis deberá:

marcarse inválida;
eliminarse o invalidarse;
registrarse operacionalmente;
reconstruirse desde la fuente autoritativa cuando corresponda.

No deberá utilizarse la entrada para producir una autorización positiva.

14. Event Platform Failure

Si el Event Platform está temporalmente indisponible:

PostgreSQL
    │
    ▼
Transactional Outbox
    │
    X
Event Platform

El cambio transaccional no debe perderse.

El Outbox permanecerá pendiente hasta que pueda publicarse.

Durante este período puede existir:

Authoritative state = NEW
Cache state = OLD

Por ello, una autorización cacheada que pueda haber quedado obsoleta no debe utilizarse para operaciones sensibles.

15. Invalidation Consumer Failure

Si el consumidor está detenido:

Event Platform
      │
      ▼
Pending Events
      │
      X
Consumer

Los eventos deberán permanecer disponibles para procesamiento posterior.

El sistema no debe asumir:

No invalidation processed = no authorization change

Por el contrario:

Consumer lag
     ↓
Potential stale cache
     ↓
Reduced cache trust
16. Event Propagation Lag

La propagación de invalidaciones será observable mediante métricas.

Conceptualmente:

Propagation Lag =
Consumer Processing Time - Event Creation Time

Cuando el lag supere un umbral definido:

NORMAL
   ↓
DEGRADED
   ↓
SECURITY-SENSITIVE CACHE BYPASS
   ↓
FAIL CLOSED

Los valores exactos de estos umbrales quedan pendientes de definición.

17. PostgreSQL Failure

PostgreSQL es la fuente transaccional primaria.

Si PostgreSQL está:

temporalmente lento
en failover
indisponible
parcialmente inaccesible

el sistema podrá utilizar cache únicamente cuando la entrada cumpla completamente las reglas de elegibilidad establecidas por ADR-052.

Para operaciones críticas:

No authoritative state
        ↓
No reliable ALLOW
        ↓
FAIL CLOSED

No se deberá convertir la disponibilidad del cache en autoridad permanente.

18. Version/Epoch Store Failure

Los mecanismos de versionado/epoch son parte fundamental de la seguridad del cache.

Si no puede verificarse correctamente la versión:

Cannot validate freshness
        ↓
Cache trust reduced

Para operaciones sensibles:

Fresh evaluation required

Si tampoco es posible realizar una evaluación fresca:

DENY
19. Cache Stampede Protection

Una caída o expiración masiva de cache puede provocar:

10,000 requests
       ↓
10,000 PostgreSQL queries

lo que puede producir una segunda falla.

Se adoptarán mecanismos de protección como:

request coalescing
single-flight
bounded concurrency
jittered TTL
controlled warm-up
rate limiting
backpressure
circuit breakers
prioritized authorization requests

La implementación exacta queda pendiente.

20. Circuit Breaker Strategy

Los componentes externos podrán utilizar circuit breakers.

Estados conceptuales:

CLOSED
   ↓
OPEN
   ↓
HALF-OPEN
   ↓
CLOSED

El circuit breaker no podrá alterar las reglas de seguridad.

Por ejemplo:

Redis circuit OPEN
       ↓
Fallback
       ↓
PostgreSQL / fresh evaluation

pero nunca:

Redis circuit OPEN
       ↓
ALLOW

por defecto.

21. PostgreSQL Backpressure

Si PostgreSQL presenta alta carga debido a una falla de cache, el Runtime deberá evitar una cascada.

Se podrán aplicar:

bounded concurrency
request prioritization
rate limiting
queueing
circuit breakers
rejection of non-critical authorization requests
cache rebuild throttling

La protección de PostgreSQL no deberá provocar autorización permisiva.

22. Multi-Region Failure

En una arquitectura multi-región:

Region A
 ├─ API
 ├─ L1
 ├─ L2
 └─ Authorization Runtime

Region B
 ├─ API
 ├─ L1
 ├─ L2
 └─ Authorization Runtime

una región no deberá asumir automáticamente que el estado de otra región es válido.

Particularmente:

security epochs
session revocation
membership revocation
MFA changes
permission changes
policy changes

deben respetar los mecanismos de consistencia definidos por Identity.

Ante una partición que impida comprobar un cambio de seguridad crítico:

Cross-region uncertainty
        ↓
Critical authorization
        ↓
FAIL CLOSED
23. Session and Security Revocation

Las siguientes situaciones recibirán tratamiento de alta prioridad:

session revocation
refresh token reuse
account suspension
account disable
MFA factor removal
password compromise
membership removal
role removal
permission removal
security policy tightening

Una autorización cacheada previamente positiva no deberá sobrevivir indefinidamente a estos cambios.

La invalidación y/o epoch debe permitir detectar:

Cached Version < Current Security Version

y considerar la entrada stale.

24. Critical Security Events

Los eventos de seguridad deberán tener prioridad sobre optimizaciones de cache.

Ejemplo:

Refresh Token Reuse
       ↓
Security Event
       ↓
Session/Family Revocation
       ↓
Security Epoch
       ↓
Cache Invalidation

Si existe retraso en la propagación:

Critical operation
       ↓
Fresh security state

cuando sea necesario.

25. Stale ALLOW Protection

Este ADR establece una regla explícita:

Un ALLOW cacheado no es válido únicamente porque no haya expirado su TTL.

La validez dependerá de:

TTL
security epoch
entity version
policy version
tenant
actor
membership
resource
action
context fingerprint
elegibilidad de cache
estado de seguridad

Por lo tanto:

TTL valid
+
Version stale
=
INVALID
26. Stale DENY

Un DENY cacheado tampoco debe asumirse indefinidamente válido.

Ejemplo:

User had no permission
       ↓
Role granted
       ↓
Cached DENY

El sistema debe poder invalidarlo y permitir una nueva evaluación.

Por lo tanto:

Security correctness
≠
ALLOW-only invalidation

Tanto ALLOW como DENY deben respetar la estrategia de invalidación.

27. Decision Matrix
Situación	Operación normal	Operación sensible
L1 disponible y válida	Cache permitido	Cache según elegibilidad
L1 caída	L2/fresh evaluation	Fresh si necesario
Redis caída	L1/PostgreSQL	Fresh
Event Platform caída	Cache solo si confianza válida	Fresh
Consumer detenido	Cache restringida	Fresh
Propagation lag bajo	Normal	Según política
Propagation lag alto	Degradación	Fresh
Version store caída	Cache limitada	Fresh
PostgreSQL caída	Cache elegible	DENY si no puede validarse
Cache corrupta	Discard + rebuild	Discard + fresh
Security state incierto	Restringir	DENY / challenge
Authorization Runtime no disponible	DENY	DENY
Cross-tenant state incierto	DENY	DENY
28. Failure Decision Algorithm

Conceptualmente:

function authorize(request):

    validate_request()

    validate_authentication()

    validate_tenant_context()

    classify_operation()

    cache_state = inspect_cache()

    if cache_state == VALID:
        return evaluate_cached_state()

    if cache_state == STALE:
        invalidate_cache()

    if cache_state == UNKNOWN:
        treat_as_UNTRUSTED()

    if operation_is_security_sensitive:
        return fresh_authorization_evaluation()

    if authoritative_state_available:
        return fresh_authorization_evaluation()

    if safe_fallback_available:
        return safe_fallback()

    return DENY

La implementación final deberá respetar completamente ADR-049.

29. Safe Fallback

Un fallback sólo será válido si:

mantiene tenant isolation;
no permite privilege escalation;
no ignora security state;
no utiliza información cuya frescura no pueda justificarse;
respeta RBAC/ABAC/ownership/delegation;
mantiene las políticas obligatorias;
no transforma UNKNOWN en ALLOW.
30. MFA as Degradation Mechanism

En algunos escenarios la política podrá determinar:

ALLOW

solamente después de:

REQUIRE_MFA

o:

REAUTHENTICATE

Sin embargo, MFA no debe utilizarse como mecanismo genérico para convertir un estado de autorización desconocido en ALLOW.

Ejemplo:

Unknown tenant membership
       ↓
MFA
       ↓
NO

La autenticación reforzada no sustituye la autorización.

31. DLQ Strategy

Los eventos de invalidación que no puedan procesarse después de los reintentos normales podrán llegar a una DLQ.

Pero:

DLQ ≠ safe to use stale cache

La existencia de un evento pendiente de invalidación debe poder reflejarse en el estado operacional de confianza.

Se deberá implementar posteriormente un proceso de:

DLQ
 ↓
Investigation
 ↓
Replay / Reconciliation
 ↓
Verification
 ↓
Resolution
32. Reconciliation

La recuperación no dependerá exclusivamente de los eventos.

Se establecerá un mecanismo de reconciliación contra PostgreSQL.

Conceptualmente:

PostgreSQL
     │
     ▼
Authoritative Versions
     │
     ▼
Compare
     │
 ┌───┴────┐
 │        │
Match   Mismatch
 │        │
Done   Invalidate
          │
          ▼
        Rebuild

Esto protege contra:

eventos perdidos
eventos duplicados
consumer failures
Redis data loss
cache corruption
deployment errors
33. Recovery Sequence

La recuperación deberá seguir una secuencia controlada.

1. Detect failure
        ↓
2. Activate degraded mode
        ↓
3. Protect critical authorization
        ↓
4. Restore infrastructure
        ↓
5. Validate connectivity
        ↓
6. Reconcile versions/epochs
        ↓
7. Process pending events
        ↓
8. Invalidate stale cache
        ↓
9. Warm cache gradually
        ↓
10. Validate authorization
        ↓
11. Exit degraded mode

No se deberá pasar directamente de:

FAILURE → NORMAL

sin reconciliación.

34. Cache Warm-Up

Después de una recuperación:

DO NOT
rebuild entire cache instantly

Debe evitarse una segunda sobrecarga.

Se recomienda:

warm-up progresivo;
prioritización de entradas frecuentes;
bounded concurrency;
jitter;
observación de DB load;
suspensión automática si PostgreSQL se sobrecarga.

Los detalles operativos quedan pendientes.

35. Recovery Validation

Antes de declarar recuperación completa se deberán validar:

authorization decisions
tenant isolation
policy versions
security epochs
session revocations
membership revocations
role changes
permission changes
MFA state
delegation state
ownership state
cache consistency
event backlog
DLQ
propagation latency
36. Observability

Se deberán registrar métricas como:

Cache
L1 hit rate
L2 hit rate
miss rate
stale rate
corruption rate
eviction rate
rebuild rate
Infrastructure
Redis latency
Redis availability
PostgreSQL latency
PostgreSQL errors
Event Platform availability
consumer lag
Authorization
fresh evaluations
cached evaluations
fallback evaluations
DENY due to uncertainty
REQUIRE_MFA due to policy
REAUTHENTICATE decisions
authorization failures
Recovery
reconciliation duration
recovery duration
cache warm-up duration
DLQ size
replay rate
propagation lag
37. Security Telemetry

Los siguientes eventos deberán poder generar señales de seguridad:

unexpected cache corruption
impossible version
epoch regression
tenant mismatch
repeated invalid cache signatures/metadata
abnormal invalidation lag
repeated stale ALLOW attempts
unauthorized cache manipulation
unusual authorization fallback rate

El sistema debe diferenciar:

Operational failure

de:

Potential security incident
38. AI and Autonomous Agents

Los agentes de IA de EVOXA deberán utilizar exactamente las mismas reglas de autorización.

Un agente AI:

no puede ignorar cache invalidation;
no puede forzar ALLOW;
no puede modificar security epochs;
no puede eliminar invalidaciones;
no puede manipular Redis para obtener permisos;
no puede desactivar circuit breakers de seguridad;
no puede convertir UNKNOWN en ALLOW.

La IA podrá:

detectar anomalías;
sugerir recuperación;
analizar propagation lag;
proponer reconciliaciones;
recomendar acciones operacionales.

Pero las acciones críticas deberán estar sujetas a las políticas de seguridad correspondientes.

39. Disaster Recovery Interaction

Este ADR debe integrarse con:

ADR-IDENTITY-029 — Disaster Recovery & Backup
ADR-IDENTITY-030 — DR Testing & Recovery Validation
ADR-IDENTITY-031 — Identity HA & Failover

Después de un disaster recovery:

PostgreSQL restore
        ↓
Validate authoritative state
        ↓
Restore Event capability
        ↓
Reconcile security versions
        ↓
Invalidate/rebuild caches
        ↓
Validate Identity

Nunca deberá restaurarse un cache antiguo y considerarlo automáticamente confiable.

40. Alternatives Considered
Alternative A — Cache Failure = Allow

Rejected.

Representa un bypass de autorización.

Alternative B — Cache Failure = Deny Everything

Rejected as universal strategy.

Aunque seguro, puede generar indisponibilidad innecesaria.

Se adopta degradación controlada para operaciones que puedan evaluarse de manera segura.

Alternative C — TTL as the Only Safety Mechanism

Rejected.

Un cambio de seguridad puede producirse inmediatamente después de que una entrada haya sido cacheada.

TTL no garantiza invalidación inmediata.

Alternative D — Redis as Authorization Source

Rejected.

Redis es una capa de rendimiento y no la fuente transaccional autoritativa.

Alternative E — Event Platform as Authorization Source

Rejected.

Los eventos representan hechos del dominio; no reemplazan el estado autoritativo.

Alternative F — Global Cache Flush on Any Failure

Rejected.

Puede provocar:

cache stampede;
sobrecarga de PostgreSQL;
degradación general;
pérdida innecesaria de rendimiento.

Se prefiere invalidación selectiva y epochs.

41. Non-Negotiable Security Rules

Las siguientes reglas son obligatorias:

Authorization must fail closed when authorization state is uncertain.
Cache nunca será la autoridad final.
Redis nunca será la fuente primaria.
TTL no sustituye invalidación.
UNKNOWN nunca puede convertirse automáticamente en ALLOW.
Cross-tenant uncertainty debe resultar en DENY.
Revocaciones de seguridad deben tener prioridad.
Cache corruption implica invalidación.
Event lag no debe permitir stale ALLOW indefinido.
DLQ no significa que el cache siga siendo confiable.
Recovery requiere reconciliación.
Cache warm-up debe ser controlado.
Los agentes AI no pueden alterar decisiones de seguridad.
No se almacenarán secretos en cache de autorización.
Los mecanismos de degradación deben ser auditables.
No se permitirá privilege amplification durante fallback.
La recuperación no debe reactivar autorizaciones revocadas.
No se debe asumir consistencia entre regiones durante una partición.
Las decisiones críticas deben poder requerir estado fresco.
Authorization Runtime mantiene la autoridad lógica sobre la decisión.
42. Traceability
Blueprint

Este ADR implementa y operacionaliza conceptos de:

Identity
Authorization
Tenant Isolation
RBAC
ABAC
Security Policy Engine
Session Security
MFA
Event-Driven Architecture
Transactional Outbox
At-Least-Once Delivery
Idempotent Consumers
Replay
Observability
High Availability
Disaster Recovery
Secure-by-Design
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
Related ADRs
ADR-IDENTITY-007 — Transactional Outbox & Event Delivery
ADR-IDENTITY-018 — Central Shared Event Broker
ADR-IDENTITY-019 — Partitioned Event Streaming
ADR-IDENTITY-020 — Event Schema Registry
ADR-IDENTITY-021 — Controlled Replay
ADR-IDENTITY-022 — Consumer Idempotency
ADR-IDENTITY-028 — Observability & Operational Security
ADR-IDENTITY-029 — Disaster Recovery & Backup
ADR-IDENTITY-030 — DR Testing
ADR-IDENTITY-031 — HA & Failover
ADR-IDENTITY-043 — Multi-Organization Identity
ADR-IDENTITY-045 — Authorization Policy Model
ADR-IDENTITY-046 — Policy Language & Governance
ADR-IDENTITY-047 — Policy Engine Implementation
ADR-IDENTITY-048 — Authorization DSL
ADR-IDENTITY-049 — Authorization Evaluation Runtime
ADR-IDENTITY-050 — Authorization Cache Strategy
ADR-IDENTITY-051 — Cache Key & Versioning
ADR-IDENTITY-052 — Cache Eligibility & TTL
ADR-IDENTITY-053 — Cache Invalidation & Propagation
43. Dependencies

Este ADR depende conceptualmente de:

PostgreSQL
    +
Transactional Outbox
    +
Event Platform
    +
Authorization Runtime
    +
Security Policy Engine
    +
L1 Cache
    +
L2 Cache
    +
Version/Epoch Mechanism
    +
Observability

La implementación tecnológica exacta de cada componente queda sujeta a las decisiones correspondientes de implementación.

44. Pending Decisions

Los siguientes puntos quedan deliberadamente abiertos y deberán resolverse en ADRs posteriores o durante las Engineering Specifications:

44.1 Exact degraded-mode thresholds
propagation lag máximo;
Redis latency threshold;
PostgreSQL latency threshold;
event consumer lag threshold.
44.2 Exact stale window

Definir:

Maximum Safe Authorization Staleness

por categoría de operación.

44.3 Critical operation catalog

Definir formalmente qué operaciones requieren siempre evaluación fresca.

44.4 Circuit breaker parameters
failure threshold;
timeout;
recovery interval;
half-open policy.
44.5 Cache stampede controls

Definir mecanismo concreto de:

single-flight;
locking;
request coalescing;
backpressure.
44.6 Multi-region behavior

Definir:

consistency model;
regional failover;
security epoch propagation;
partition behavior.
44.7 Recovery automation

Definir qué acciones serán:

automáticas;
semiautomáticas;
manuales;
sujetas a approval.
44.8 Reconciliation frequency

Definir reconciliación:

periódica;
event-triggered;
failure-triggered;
manual.
45. Acceptance Criteria
AC-01 — Redis Failure

Given Redis is unavailable
When an authorization request arrives
Then the system must use a safe fallback or fail closed.

AC-02 — L1 Failure

Given L1 cache is unavailable
When authorization is requested
Then L2 or authoritative evaluation may be used.

AC-03 — Cache Corruption

Given a cache entry is detected as invalid
When authorization is evaluated
Then the entry must not produce an ALLOW.

AC-04 — Event Platform Failure

Given Event Platform is unavailable
When an authorization-affecting transaction commits
Then the transactional outbox must preserve the event.

AC-05 — Consumer Failure

Given invalidation consumer is stopped
When authorization state changes
Then pending events must remain recoverable.

AC-06 — Stale ALLOW

Given cache contains an outdated authorization result
When version/epoch validation detects staleness
Then the cached result must not be used.

AC-07 — Stale DENY

Given authorization changes from DENY to ALLOW
When cache invalidation occurs
Then the stale DENY must be invalidated.

AC-08 — Security Revocation

Given a session or membership is revoked
When authorization is evaluated
Then stale authorization must not bypass the revocation.

AC-09 — Tenant Isolation

Given authorization state cannot be confidently associated with the current tenant
When authorization is requested
Then the result must be DENY.

AC-10 — Version Store Failure

Given authorization cache version validation is unavailable
When a sensitive operation is requested
Then a fresh authoritative evaluation is required.

AC-11 — PostgreSQL Failure

Given PostgreSQL is unavailable
When a critical operation requires fresh authorization state
Then the operation must fail closed.

AC-12 — Cache Stampede

Given a large number of cache entries expire simultaneously
When requests arrive
Then the system must prevent uncontrolled database amplification.

AC-13 — DLQ

Given an invalidation event enters the DLQ
When authorization is requested
Then the system must not assume that stale cached state is safe.

AC-14 — Recovery

Given infrastructure has recovered
When degraded mode is exited
Then authoritative state and cache versions must be reconciled first.

AC-15 — Cache Warm-Up

Given cache is being rebuilt
When recovery is occurring
Then warm-up must be bounded and controlled.

AC-16 — Multi-Region Partition

Given regional connectivity is partitioned
When security state cannot be validated
Then critical authorization must fail closed.

AC-17 — AI Agent

Given an AI agent requests authorization
When cache infrastructure is degraded
Then the AI agent must follow exactly the same authorization degradation rules.

AC-18 — Observability

Given authorization enters degraded mode
When the failure occurs
Then metrics and operational/security telemetry must record the degradation.

AC-19 — Recovery Integrity

Given a previously revoked authorization exists
When caches are rebuilt
Then the revoked state must not be reactivated.

AC-20 — No Authorization Bypass

Given any cache or infrastructure failure
When the system cannot establish a trustworthy authorization decision
Then it must not return an unauthorized ALLOW.

46. Architectural Result

Con este ADR, EVOXA establece una separación clara entre:

Availability
     │
     ├── Cache
     ├── Redis
     └── Event Platform
     
Security Authority
     │
     ├── PostgreSQL
     └── Authorization Runtime

La arquitectura resultante es:

                    ┌─────────────────────┐
                    │ Authorization       │
                    │ Request             │
                    └──────────┬──────────┘
                               │
                               ▼
                    ┌─────────────────────┐
                    │ Authorization       │
                    │ Runtime             │
                    └──────────┬──────────┘
                               │
                 ┌─────────────┼─────────────┐
                 │             │             │
                 ▼             ▼             ▼
              L1 Cache      L2 Redis    Fresh State
                 │             │             │
                 └─────────────┼─────────────┘
                               │
                               ▼
                    ┌─────────────────────┐
                    │ Validation          │
                    │ Version / Epoch     │
                    │ Tenant / Context    │
                    └──────────┬──────────┘
                               │
                               ▼
                    ┌─────────────────────┐
                    │ Security Policy     │
                    │ Evaluation           │
                    └──────────┬──────────┘
                               │
                               ▼
               ┌───────────────┼────────────────┐
               ▼               ▼                ▼
             ALLOW            DENY        CHALLENGE/
                                           MFA/REAUTH

La regla arquitectónica central queda establecida como:

EVOXA puede degradar rendimiento, pero nunca degradar la seguridad de autorización para mantener disponibilidad.

47. Next ADR

El siguiente ADR recomendado, siguiendo la secuencia natural de esta arquitectura, es:

ADR-IDENTITY-055 — Authorization Cache Recovery, Reconciliation & Warm-Up Protocol

Este ADR debería definir con precisión el procedimiento posterior a una falla:

detect → isolate → reconcile → invalidate → rebuild → warm-up → validate → restore normal operation, incluyendo algoritmos de reconciliación, prioridades, límites de carga, recuperación de L1/L2, validación de epochs/versiones, DLQ replay y criterios objetivos para salir del modo degradado.
