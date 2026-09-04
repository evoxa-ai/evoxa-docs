ADR-IDENTITY-055 — Authorization Cache Recovery, Reconciliation & Warm-Up Protocol

Status: Proposed
Version: 1.0
Date: 2026-09-04
Domain: Identity / Authorization
Priority: Critical
Type: Architecture / Security / Distributed Systems / Recovery

1. Context

ADR-IDENTITY-054 estableció cómo debe comportarse EVOXA cuando existen fallas o degradación en:

L1 Authorization Cache
L2 Redis
Event Platform
Invalidation Consumer
PostgreSQL
Version/Epoch Store
red
regiones
mecanismos de cache
mecanismos de invalidación

Ese ADR establece que una falla de infraestructura no puede convertirse en un bypass de autorización.

Sin embargo, después de una falla aparece un segundo problema:

¿Cómo vuelve el sistema desde un estado degradado a un estado normal sin reintroducir información de autorización obsoleta o inconsistente?

La recuperación de cache no puede limitarse a:

Redis vuelve
      ↓
llenar cache
      ↓
NORMAL

porque durante la falla pueden haber ocurrido:

cambios de permisos;
cambios de roles;
revocaciones;
cambios de membership;
cambios de policy;
cambios de MFA;
cambios de ownership;
cambios de delegation;
eventos pendientes;
eventos duplicados;
eventos fuera de orden;
eventos en DLQ;
pérdida de L1;
pérdida de Redis;
corrupción de cache;
cambios de security epoch;
failover de PostgreSQL;
recuperación regional.

Por ello se define un protocolo formal de:

Detection
   ↓
Isolation
   ↓
Recovery
   ↓
Reconciliation
   ↓
Invalidation
   ↓
Rebuild
   ↓
Warm-Up
   ↓
Validation
   ↓
Normal Operation
2. Problem Statement

EVOXA necesita garantizar que la recuperación del Authorization Cache:

no restaure autorizaciones revocadas;
no restaure permisos eliminados;
no pierda nuevos permisos concedidos;
no mezcle tenants;
no reutilice versiones antiguas;
no produzca privilege escalation;
no provoque una nueva sobrecarga de PostgreSQL;
no considere recuperado un sistema que todavía tiene eventos pendientes;
pueda recuperarse después de pérdida total de cache;
sea repetible e idempotente;
sea observable;
sea auditable;
pueda validarse automáticamente;
pueda soportar replay y reconciliación.
3. Decision

EVOXA adoptará un protocolo de recuperación basado en:

1. PostgreSQL como autoridad

El estado autoritativo proviene de PostgreSQL.

2. Eventos como mecanismo de propagación

Los eventos permiten conocer cambios ocurridos durante la operación normal y durante la degradación.

3. Versiones/Epochs como mecanismo de seguridad

Las versiones y security epochs permiten detectar información obsoleta.

4. Reconciliation como requisito previo

No se permitirá reconstruir y declarar confiable el cache únicamente a partir de los eventos.

5. Warm-up progresivo

El cache será reconstruido gradualmente y bajo control de carga.

6. Validation before promotion

Una instancia de cache recuperada deberá validarse antes de considerarse confiable.

7. Fail-closed durante incertidumbre

Mientras no pueda establecerse la consistencia necesaria, las operaciones críticas deberán permanecer en modo seguro.

4. Recovery State Machine

El sistema tendrá estados conceptuales:

                    ┌───────────────┐
                    │    NORMAL     │
                    └───────┬───────┘
                            │
                         Failure
                            │
                            ▼
                    ┌───────────────┐
                    │   DEGRADED    │
                    └───────┬───────┘
                            │
                       Infrastructure
                        Recovery
                            │
                            ▼
                    ┌───────────────┐
                    │ RECONCILING   │
                    └───────┬───────┘
                            │
                      Consistency
                         verified
                            │
                            ▼
                    ┌───────────────┐
                    │ INVALIDATING  │
                    └───────┬───────┘
                            │
                            ▼
                    ┌───────────────┐
                    │   REBUILD     │
                    └───────┬───────┘
                            │
                            ▼
                    ┌───────────────┐
                    │   WARM-UP     │
                    └───────┬───────┘
                            │
                       Validation
                            │
                            ▼
                    ┌───────────────┐
                    │   RECOVERED   │
                    └───────┬───────┘
                            │
                            ▼
                    ┌───────────────┐
                    │    NORMAL     │
                    └───────────────┘

Cualquier fallo durante:

reconciliation;
invalidation;
rebuild;
warm-up;
validation;

deberá devolver el sistema a un estado degradado seguro.

5. Recovery Principle

La recuperación debe asumir que:

Todo estado cacheado existente después de una falla puede ser incorrecto hasta demostrar lo contrario.

Por lo tanto:

Recovered infrastructure
        ≠
Recovered authorization state

La infraestructura puede estar disponible antes de que el estado de autorización sea confiable.

6. Recovery Phases

El protocolo estará compuesto por diez fases.

Phase 1 — Failure Detection

El sistema detecta:

Redis failure;
L1 failure;
consumer failure;
event lag;
PostgreSQL failover;
version inconsistency;
cache corruption;
regional failure.

Debe registrarse:

timestamp;
componente;
tenant scope cuando aplique;
severity;
correlation ID;
recovery state;
incident identifier.
7. Phase 2 — Safe Isolation

Antes de reconstruir cache se deben aislar las entradas potencialmente inseguras.

Ejemplo:

Potentially stale cache
          ↓
Mark untrusted
          ↓
No sensitive ALLOW

La estrategia podrá ser:

invalidación selectiva;
epoch bump;
namespace isolation;
cache generation change;
complete cache invalidation.

La selección dependerá del alcance de la falla.

8. Cache Generation

Se recomienda introducir el concepto de Cache Generation.

Ejemplo conceptual:

Generation 41
     ↓
Failure
     ↓
Generation 42

Las entradas pertenecientes a generaciones anteriores pueden considerarse obsoletas cuando el incidente afecta la integridad general del cache.

Esto permite evitar depender de la eliminación física inmediata de todas las entradas.

9. Phase 3 — Infrastructure Recovery

Primero se recuperan los componentes:

PostgreSQL
Event Platform
Redis
Authorization Runtime
Invalidation Consumer

El orden exacto dependerá del tipo de incidente.

La disponibilidad de Redis no significa todavía que pueda recibir tráfico completo.

10. Phase 4 — Authoritative State Validation

Antes de reconstruir cache se debe comprobar que PostgreSQL representa un estado coherente.

Debe validarse:

users;
memberships;
roles;
permissions;
role assignments;
policies;
delegations;
ownership;
sessions;
MFA/security state;
security epochs;
entity versions.

La validación deberá utilizar las restricciones y consistencia transaccional de PostgreSQL.

11. Phase 5 — Event Reconciliation

Los eventos pendientes deberán compararse con el estado autoritativo.

Conceptualmente:

PostgreSQL State
       +
Event Stream
       +
Outbox
       +
DLQ
       ↓
Reconciliation

La reconciliación debe identificar:

eventos pendientes;
eventos duplicados;
eventos fuera de orden;
eventos ya reflejados;
eventos faltantes;
eventos no procesados;
eventos en DLQ.
12. Event Replay

Cuando sea necesario se podrá ejecutar replay controlado.

El replay deberá ser:

explícito;
auditable;
idempotente;
tenant-aware;
version-aware;
limitado al alcance necesario.

No se debe ejecutar un replay indiscriminado de todo el Event Platform si solamente existe una falla localizada.

13. Event vs State Reconciliation

Un evento puede decir:

RolePermissionChanged

pero PostgreSQL puede contener actualmente una versión posterior.

Por ello el objetivo de la reconciliación no es simplemente:

replay everything

sino:

determine authoritative current state

Los eventos son evidencia de cambios y mecanismo de propagación; PostgreSQL continúa siendo la autoridad transaccional.

14. Phase 6 — Version and Epoch Reconciliation

Se deberá comparar:

Cache Version
        vs
Authoritative Version

y:

Cache Security Epoch
        vs
Authoritative Security Epoch

Regla:

Cache Version < Authoritative Version
        ↓
STALE

y:

Cache Epoch < Authoritative Epoch
        ↓
INVALID

Nunca se deberá aceptar una regresión:

Current Epoch = 25
Incoming Epoch = 24

El valor 24 no puede reemplazar al 25.

15. Phase 7 — Cache Invalidation

Después de la reconciliación se ejecutará invalidación.

Dependiendo del incidente:

Selective invalidation
Tenant A
   ↓
Invalidate affected entries
Scope invalidation
Membership X
   ↓
Invalidate related decisions
Security epoch invalidation
Epoch changed
   ↓
Old cache entries become invalid
Full generation invalidation
Cache integrity uncertain
       ↓
New generation
       ↓
Old generation untrusted
16. Full Cache Loss

Si Redis perdió completamente los datos:

Redis = EMPTY

esto no constituye pérdida de autorización.

El cache podrá reconstruirse desde:

PostgreSQL
+
current versions
+
security epochs

El sistema debe poder funcionar temporalmente con:

Cache MISS
   ↓
Fresh Evaluation
17. Cache Rebuild

El rebuild debe producir entradas únicamente desde información validada.

Pipeline:

Authoritative State
       ↓
Authorization Context
       ↓
Authorization Runtime
       ↓
Validated Decision
       ↓
Cache Entry

No se deberá copiar una entrada desde:

Old Cache → New Cache

si la entrada antigua no fue previamente validada.

18. Rebuild Ordering

El rebuild debe priorizar:

security-critical entries;
high-frequency authorization paths;
administrative paths;
high-volume tenants;
frequently accessed resources.

Sin embargo, una entrada no debe considerarse confiable simplemente por ser prioritaria.

19. Warm-Up

El warm-up es distinto del rebuild.

Rebuild

Reconstruye explícitamente información necesaria.

Warm-up

Permite que el tráfico real reconstruya gradualmente el cache.

Rebuild
   ↓
Minimal trusted cache
   ↓
Warm-up
   ↓
Normal traffic
20. Warm-Up Strategy

El warm-up deberá ser:

progresivo;
bounded;
observable;
reversible;
tenant-aware;
protegido contra stampede.

Se podrán utilizar:

TTL jitter;
request coalescing;
single-flight;
bounded workers;
concurrency limits;
priority queues;
adaptive throttling.
21. Warm-Up Load Protection

El sistema deberá proteger PostgreSQL.

Ejemplo:

100,000 cache misses
        ↓
NOT
100,000 DB queries

Debe aplicarse:

Cache Miss
    ↓
Single Flight
    ↓
One authoritative evaluation
    ↓
Populate cache
    ↓
Concurrent requests reuse result

Los detalles concretos de implementación quedan pendientes.

22. Cache Warming Failure

Si el warm-up provoca:

DB overload;
Redis overload;
authorization latency;
error spikes;

el proceso deberá reducir automáticamente su intensidad.

Puede ocurrir:

WARM-UP
   ↓
OVERLOAD
   ↓
THROTTLE
   ↓
RECOVER

No se debe sacrificar la estabilidad de PostgreSQL para acelerar la recuperación del cache.

23. Validation Before Promotion

Una cache recuperada no debe pasar directamente a producción.

Debe superar validaciones.

Functional
ALLOW correcto;
DENY correcto;
MFA correcto;
reauthentication correcto.
Security
tenant isolation;
revoked membership;
revoked role;
revoked permission;
revoked session;
MFA changes;
delegation changes.
Consistency
versions;
epochs;
policy versions;
cache generation.
Operational
latency;
error rate;
Redis health;
consumer lag.
24. Shadow Validation

Cuando sea posible, durante la recuperación se podrá ejecutar:

Cached Decision
       vs
Fresh Decision

sin exponer necesariamente el resultado fresco al usuario.

Ejemplo:

Cache → ALLOW
Fresh → DENY
       ↓
MISMATCH
       ↓
Invalidate
       ↓
Security investigation

Esta estrategia permite detectar inconsistencias antes de declarar recuperación completa.

25. Mismatch Handling

Si:

Cached Decision != Fresh Decision

la entrada cacheada deberá considerarse no confiable.

Acción:

Invalidate
   ↓
Re-evaluate
   ↓
Store only validated result

Los mismatches significativos deberán generar telemetry y, cuando corresponda, security events.

26. Recovery Confidence

El sistema deberá mantener un indicador conceptual:

Recovery Confidence

Estados:

UNKNOWN
DEGRADED
RECONCILED
VALIDATED
RECOVERED

No se deberá marcar:

RECOVERED

hasta que se hayan cumplido los criterios establecidos.

27. Tenant-Aware Recovery

La recuperación debe respetar tenant isolation.

Un problema en:

Tenant A

no debe provocar automáticamente:

Tenant B

salvo que el componente afectado sea global.

Por ejemplo:

Organization scope

puede permitir recuperación selectiva.

Mientras que:

Platform security epoch

puede requerir invalidación global.

28. Security Epoch Recovery

Los security epochs son especialmente importantes durante recuperación.

Si:

Platform Epoch = 100
Tenant Epoch = 42
Membership Epoch = 8

las entradas deberán respetar la combinación correspondiente.

Una entrada antigua no podrá reaparecer simplemente porque Redis haya sido reconstruido.

29. Policy Recovery

Las policies deberán reconstruirse únicamente desde estados:

ACTIVE

y dentro de su:

effective period

No deberán reconstruirse policies:

DRAFT;
RETIRED;
SUPERSEDED;

como políticas activas.

30. Membership Recovery

La recuperación debe validar que:

Membership
    +
Status
    +
Organization
    +
Roles

continúen siendo válidos.

Un membership revocado durante la falla debe continuar revocado después de la recuperación.

31. Delegation Recovery

Las delegaciones deberán validarse:

status;
scope;
expiration;
delegator;
delegatee;
tenant;
permission boundary.

No se deberá restaurar una delegation expirada.

32. Session Recovery

Las sesiones deberán respetar:

revoked;
expired;
suspended;
security state.

La reconstrucción de cache no debe reactivar una sesión revocada.

33. MFA Recovery

Cambios de MFA deberán recibir tratamiento prioritario.

Ejemplo:

Factor removed
       ↓
Security state updated
       ↓
Cache invalidation
       ↓
Rebuild

Una entrada antigua que indique:

MFA satisfied

no puede utilizarse si el estado actual exige nuevamente MFA.

34. Recovery of Revocations

Las revocaciones tendrán prioridad durante reconciliación.

Ejemplos:

account disabled;
session revoked;
refresh family revoked;
membership revoked;
role removed;
permission removed;
MFA factor revoked;
delegation revoked.

La recuperación debe garantizar:

Revocation survives cache loss
35. DLQ Recovery

El proceso será:

DLQ
 ↓
Classify
 ↓
Determine authoritative state
 ↓
Replay if necessary
 ↓
Validate
 ↓
Remove from recovery queue

No se deberá simplemente reinyectar toda la DLQ sin análisis.

36. Idempotent Recovery

Cada operación de recovery deberá poder ejecutarse nuevamente sin producir resultados incorrectos.

Por ejemplo:

Invalidate X
Invalidate X
Invalidate X

debe ser seguro.

Igualmente:

Rebuild X
Rebuild X

debe terminar en el mismo estado autoritativo.

37. Recovery Concurrency

Dos procesos de recuperación no deberán producir:

version regression;
conflicting rebuilds;
duplicate security actions;
cache poisoning.

Se deberá controlar la concurrencia mediante mecanismos apropiados.

38. Recovery Locking

Podrá existir un mecanismo conceptual de:

Recovery Coordinator

para controlar:

tenant;
cache namespace;
generation;
rebuild partition.

No se requiere necesariamente un lock global.

Se prefiere:

Scoped Recovery

para evitar bloquear innecesariamente toda la plataforma.

39. Progressive Recovery

La recuperación podrá realizarse por etapas:

Platform
   ↓
Organization
   ↓
Membership
   ↓
Resource

o según el alcance real de la falla.

Esto permite reducir riesgo.

40. Recovery Rollback

Si durante la recuperación se detecta una inconsistencia:

Recovery
   ↓
Mismatch
   ↓
Rollback to Degraded

El rollback significa volver a:

safe authorization behavior

y no necesariamente restaurar el cache anterior.

41. Normal Operation Exit Criteria

El sistema sólo podrá regresar a NORMAL cuando:

PostgreSQL esté saludable;
Authorization Runtime esté saludable;
version/epoch state sea consistente;
Event Platform esté operacional;
invalidation consumer esté operacional;
backlog esté dentro de límites aceptables;
DLQ esté controlada;
cache integrity esté validada;
security-critical entries estén consistentes;
no existan mismatches críticos;
tenant isolation esté validada;
métricas se encuentren dentro de límites operacionales.
42. Recovery SLO

Se deberá medir:

Recovery Time

como:

Failure Detected
        ↓
Normal Operation Restored

Pero se deberán medir también:

Time to Safe State
Time to Reconciliation
Time to Cache Recovery
Time to Full Warm-Up

Estos valores exactos quedan pendientes de definición.

43. Observability

Durante recovery deberán existir métricas:

Recovery
recovery state;
recovery duration;
reconciliation duration;
rebuild duration;
warm-up duration;
recovery failures;
rollback count.
Consistency
stale entries;
version mismatches;
epoch mismatches;
policy mismatches;
tenant mismatches;
cache generation mismatches.
Events
event backlog;
consumer lag;
replay rate;
DLQ count;
duplicate events;
out-of-order events.
Performance
DB load;
Redis load;
authorization latency;
cache hit ratio;
cache miss ratio;
fresh evaluation rate.
44. Security Auditing

Las siguientes acciones deberán ser auditables:

entering degraded mode;
entering recovery;
cache generation change;
full invalidation;
selective invalidation;
replay;
reconciliation;
recovery override;
manual intervention;
transition to normal;
recovery rollback.

No deberán registrarse:

secrets;
tokens;
passwords;
MFA secrets;
raw credentials.
45. Manual Recovery

En situaciones excepcionales puede ser necesaria intervención humana.

Los operadores deberán:

autenticarse mediante identidad de servicio/administrativa apropiada;
tener permisos mínimos;
operar dentro del tenant scope correspondiente;
generar audit trail;
no poder forzar ALLOW;
no poder eliminar security epochs para evitar controles.

Una acción manual de recuperación nunca debe constituir un bypass de Identity.

46. AI-Assisted Recovery

La IA podrá asistir en:

análisis de incidentes;
identificación de mismatches;
priorización de tenants;
detección de anomalías;
recomendaciones de replay;
análisis de propagation lag;
predicción de DB overload;
recomendación de warm-up rate.

Pero:

AI recommendation
       ≠
Authorization authority

La IA no podrá:

marcar recovery como seguro por sí sola;
disminuir security epochs;
eliminar revocaciones;
activar policies;
otorgar permisos;
manipular cache para producir ALLOW.
47. Disaster Recovery Integration

Este protocolo debe integrarse con:

ADR-IDENTITY-029 — Disaster Recovery & Backup

y:

ADR-IDENTITY-030 — DR Testing & Recovery Validation

y:

ADR-IDENTITY-031 — Identity HA & Failover

En un disaster recovery completo:

Restore PostgreSQL
       ↓
Validate authoritative state
       ↓
Restore Event Capability
       ↓
Validate Outbox
       ↓
Reconcile Events
       ↓
Rebuild Security Versions
       ↓
Invalidate Cache
       ↓
Rebuild Cache
       ↓
Warm-Up
       ↓
Validate
       ↓
NORMAL
48. Recovery Testing

La estrategia deberá probar al menos:

Scenario A

Redis complete loss.

Scenario B

L1 complete loss.

Scenario C

Redis + L1 simultaneous failure.

Scenario D

Event consumer stopped for extended period.

Scenario E

Event Platform unavailable.

Scenario F

PostgreSQL failover during cache recovery.

Scenario G

DLQ accumulation.

Scenario H

Out-of-order invalidation events.

Scenario I

Duplicate invalidation events.

Scenario J

Security epoch mismatch.

Scenario K

Cache corruption.

Scenario L

Multi-region partition.

Scenario M

Cache stampede.

Scenario N

Recovery interrupted halfway.

Scenario O

Recovery executed twice concurrently.

49. Recovery Invariants

Después de cualquier recuperación deben cumplirse:

No unauthorized ALLOW
No cross-tenant access
No privilege escalation
No revoked authorization restored
No expired delegation restored
No revoked session restored
No retired policy activated
No security epoch regression
No plaintext secret exposure
No duplicate business outcome caused by replay
50. Alternatives Considered
Alternative A — Restore Redis Backup Directly

Rejected.

Un backup puede representar un estado anterior al incidente.

Restaurarlo sin reconciliación podría reintroducir permisos o sesiones revocadas.

Alternative B — Replay All Events

Rejected as the sole recovery mechanism.

Los eventos no sustituyen el estado autoritativo actual.

Alternative C — Rebuild Cache Entirely From PostgreSQL

Partially accepted but insufficient alone.

PostgreSQL es la autoridad, pero el proceso de recuperación también necesita reconciliar:

eventos;
invalidaciones;
versiones;
epochs;
DLQ.
Alternative D — Flush Redis and Return to Normal Immediately

Rejected.

Un cache vacío no significa que todo el sistema haya recuperado consistencia operacional.

Alternative E — Warm Cache Only Through Live Traffic

Rejected as sole mechanism.

Puede generar:

latencia elevada;
DB overload;
recuperación impredecible.

Se utilizará warm-up controlado combinado con rebuild.

51. Non-Negotiable Rules
PostgreSQL sigue siendo la fuente autoritativa.
Recovery no implica automáticamente normal operation.
Reconciliation es obligatoria cuando exista incertidumbre.
Security epochs no pueden retroceder.
Cache generation antigua puede ser invalidada globalmente.
Revocaciones deben sobrevivir a pérdida de cache.
Replay debe ser idempotente.
DLQ debe ser recuperable y auditable.
Cache rebuild debe utilizar estado validado.
Warm-up debe estar limitado.
PostgreSQL debe estar protegido contra stampede.
Cache corruption nunca puede generar ALLOW.
Cross-tenant inconsistency debe fallar closed.
Recovery no puede reactivar estados revocados.
AI no puede controlar la seguridad del recovery.
Manual recovery no puede constituir authorization bypass.
La salida de degraded mode debe tener criterios verificables.
Recovery debe ser observable y auditable.
La recuperación debe poder interrumpirse y reanudarse de forma segura.
La consistencia de seguridad tiene prioridad sobre la disponibilidad del cache.
52. Traceability
Blueprint

Este ADR implementa:

Identity
Authorization
RBAC
ABAC
Tenant Isolation
Resource Ownership
Delegation
Security Policy Engine
Event-Driven Architecture
Transactional Outbox
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
53. Related ADRs

Directamente relacionados:

ADR-IDENTITY-007 — Transactional Outbox & Event Delivery
ADR-IDENTITY-018 — Central Shared Event Broker
ADR-IDENTITY-019 — Partitioned Event Streaming
ADR-IDENTITY-020 — Event Schema Registry
ADR-IDENTITY-021 — Controlled Replay
ADR-IDENTITY-022 — Consumer Idempotency
ADR-IDENTITY-028 — Observability & Operational Security
ADR-IDENTITY-029 — Disaster Recovery & Backup
ADR-IDENTITY-030 — DR Testing & Recovery Validation
ADR-IDENTITY-031 — Identity HA & Failover
ADR-IDENTITY-043 — Multi-Organization Identity
ADR-IDENTITY-045 — Authorization Policy Model & Evaluation
ADR-IDENTITY-046 — Policy Language & Governance
ADR-IDENTITY-047 — Policy Engine Implementation
ADR-IDENTITY-048 — Authorization Policy DSL
ADR-IDENTITY-049 — Authorization Evaluation Runtime
ADR-IDENTITY-050 — Authorization Cache Strategy
ADR-IDENTITY-051 — Cache Key & Versioning
ADR-IDENTITY-052 — Cache Eligibility & TTL
ADR-IDENTITY-053 — Cache Invalidation & Propagation
ADR-IDENTITY-054 — Cache Failure, Degradation & Recovery Strategy
54. Pending Decisions

Los siguientes aspectos quedan abiertos para posteriores ADRs/Engineering Specifications:

54.1 Exact Recovery SLOs

Definir:

maximum time to safe state;
maximum reconciliation time;
maximum cache recovery time;
maximum warm-up time.
54.2 Exact Warm-Up Algorithm

Definir:

batch size;
concurrency;
single-flight implementation;
queueing;
adaptive throttling.
54.3 Cache Generation Implementation

Definir:

generation storage;
generation propagation;
namespace structure;
cleanup strategy.
54.4 Recovery Coordinator

Definir:

distributed coordination;
locking;
leadership;
failure handling.
54.5 Reconciliation Algorithm

Definir formalmente:

State vs Event

y:

Version vs Epoch

comparisons.

54.6 Recovery Confidence Model

Definir métricas y thresholds exactos para:

UNKNOWN
DEGRADED
RECONCILED
VALIDATED
RECOVERED
54.7 Cross-Region Recovery

Definir:

regional ownership;
global security epochs;
partition recovery;
cache replication.
55. Acceptance Criteria
AC-01 — Complete Cache Loss

Given Redis loses all authorization cache data
When infrastructure recovers
Then EVOXA must reconstruct authorization state from authoritative sources.

AC-02 — No Revocation Resurrection

Given a session, membership, permission or delegation was revoked
When cache is rebuilt
Then the revoked state must remain revoked.

AC-03 — Epoch Consistency

Given cache contains an older security epoch
When reconciliation occurs
Then the old cache state must be invalidated.

AC-04 — Version Regression

Given an incoming state has an older version
When recovery processes it
Then it must not replace the newer version.

AC-05 — Event Replay

Given events remain unprocessed
When recovery executes replay
Then replay must be idempotent.

AC-06 — DLQ

Given invalidation events exist in DLQ
When recovery executes
Then they must be analyzed and reconciled before normal operation.

AC-07 — Cache Corruption

Given cache integrity cannot be trusted
When recovery begins
Then the affected generation must be invalidated.

AC-08 — Tenant Isolation

Given recovery is scoped to Tenant A
When recovery executes
Then Tenant B must not receive Tenant A authorization state.

AC-09 — Rebuild Source

Given cache must be rebuilt
When entries are created
Then they must derive from validated authoritative state.

AC-10 — Warm-Up Protection

Given thousands of cache misses occur
When warm-up is active
Then PostgreSQL must not receive uncontrolled duplicate evaluations.

AC-11 — Recovery Interruption

Given recovery is interrupted
When it resumes
Then the process must safely continue without corrupting authorization state.

AC-12 — Concurrent Recovery

Given two recovery processes execute concurrently
When they modify the same scope
Then they must not produce version regression or inconsistent cache state.

AC-13 — Security Validation

Given cache has been rebuilt
When security validation executes
Then revoked sessions, memberships, roles and permissions must remain revoked.

AC-14 — Shadow Validation

Given cached and fresh evaluations are compared
When they differ
Then the cache entry must be treated as untrusted.

AC-15 — Normal Transition

Given recovery is complete
When all recovery exit criteria are satisfied
Then the system may transition to NORMAL.

AC-16 — Recovery Failure

Given reconciliation fails
When recovery cannot establish consistency
Then the system must remain in safe degraded mode.

AC-17 — AI Recovery

Given an AI agent assists with recovery
When it proposes an action
Then it cannot override authorization security invariants.

AC-18 — Audit

Given recovery performs a security-relevant action
When that action occurs
Then it must be observable and auditable.

AC-19 — Disaster Recovery

Given PostgreSQL and cache are restored after disaster recovery
When Identity resumes
Then cache must be reconciled before normal authorization operation.

AC-20 — No Authorization Bypass

Given any incomplete recovery state
When an authorization request is received
Then uncertainty must never be converted into an unauthorized ALLOW.

56. Architectural Result

Con este ADR, la recuperación de autorización de EVOXA deja de ser una simple operación de infraestructura y pasa a ser un protocolo de seguridad controlado:

                 FAILURE
                    │
                    ▼
              SAFE DEGRADE
                    │
                    ▼
          AUTHORITATIVE VALIDATION
                    │
                    ▼
             EVENT RECONCILIATION
                    │
                    ▼
            VERSION / EPOCH CHECK
                    │
                    ▼
               INVALIDATION
                    │
                    ▼
                REBUILD
                    │
                    ▼
              CONTROLLED WARM-UP
                    │
                    ▼
              SECURITY VALIDATION
                    │
                    ▼
               RECOVERY CONFIDENCE
                    │
                    ▼
                  NORMAL

La regla fundamental queda establecida:

La recuperación de infraestructura no implica recuperación de confianza. La confianza debe reconstruirse, reconciliarse y validarse antes de restaurar la operación normal de autorización.

Y, en conjunto con ADR-054:

ADR-054
Failure / Degradation
        │
        ▼
Safe Authorization
        │
        ▼
ADR-055
Recovery / Reconciliation
        │
        ▼
Validated Authorization State
        │
        ▼
Normal Operation
57. Next ADR

El siguiente ADR recomendado en la secuencia es:

ADR-IDENTITY-056 — Authorization Cache Consistency Model & Freshness Guarantees

Este debería cerrar una pieza que ahora se vuelve especialmente importante: definir formalmente qué significa “fresco”, “stale”, “eventually consistent” y “strongly consistent” para una decisión de autorización, incluyendo garantías por tipo de operación, límites máximos de staleness, comportamiento ante cambios de seguridad y qué operaciones requieren siempre estado autoritativo fresco.
