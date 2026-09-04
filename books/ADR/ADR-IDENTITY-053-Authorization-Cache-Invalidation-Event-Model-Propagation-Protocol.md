ADR-IDENTITY-053 — Authorization Cache Invalidation Event Model & Propagation Protocol

Status: Proposed
Version: 1.0
Date: 2026-09-04
Domain: Identity / Authorization
Priority: Critical
Type: Architecture / Security / Distributed Systems / Eventing

1. Context

Los ADR anteriores establecieron progresivamente la arquitectura de caching del Authorization Runtime:

ADR-IDENTITY-049 — define el algoritmo de evaluación de autorización.
ADR-IDENTITY-050 — define caching, invalidation y consistency.
ADR-IDENTITY-051 — define cache keys, versions, epochs y context fingerprinting.
ADR-IDENTITY-052 — define qué decisiones pueden ser cacheadas y bajo qué condiciones.

El modelo resultante es:

PostgreSQL
   ↓
Security / Policy State
   ↓
Version / Epoch
   ↓
Authorization Cache
   ↓
Authorization Runtime

Pero en una arquitectura distribuida existe un problema adicional:

API Instance A
API Instance B
API Instance C
API Instance D

Cada instancia puede tener:

L1 local cache;
acceso a Redis/L2;
diferentes solicitudes concurrentes;
diferentes tiempos de recepción de eventos.

Cuando cambia una política, rol, permiso, membership, delegation, ownership o estado de seguridad, todas las instancias afectadas deben dejar de utilizar decisiones obsoletas.

El Blueprint establece además un modelo event-driven con:

eventos inmutables/versionados;
entrega At-Least-Once;
consumidores idempotentes;
retries;
DLQ;
replay;
ordering scoped;
transactional outbox.

Por tanto, la invalidación de authorization cache debe integrarse en ese modelo y no convertirse en un mecanismo paralelo.

2. Problem

Consideremos:

Instance A → L1 cache → ALLOW
Instance B → L1 cache → ALLOW
Instance C → L1 cache → ALLOW

Se revoca un permiso:

Permission revoked
      ↓
PostgreSQL updated
      ↓
Cache must become stale

Si sólo se elimina la entrada de Redis:

Redis → invalidated

pero Instance A mantiene:

L1 → ALLOW

podría producirse una autorización incorrecta.

Por tanto necesitamos:

eventos de invalidación;
scopes de invalidación;
propagation protocol;
version/epoch propagation;
idempotencia;
ordering;
manejo de eventos duplicados;
manejo de eventos atrasados;
recovery;
comportamiento seguro durante pérdida de infraestructura.
3. Decision

EVOXA utilizará un modelo:

Transactional Change → Outbox → Event Platform → Invalidation Consumer → Version/Epoch Update → L1/L2 Invalidation

Conceptualmente:

                 PostgreSQL
                     │
              Transaction
                     │
          ┌──────────┴──────────┐
          │                     │
     Domain Change          Outbox Event
                                │
                                ▼
                         Event Platform
                                │
                 ┌──────────────┼──────────────┐
                 ▼              ▼              ▼
              Instance A     Instance B     Instance C
                 │              │              │
                 ▼              ▼              ▼
             L1 Invalidate  L1 Invalidate  L1 Invalidate
                 │              │              │
                 └──────────────┼──────────────┘
                                ▼
                              Redis

La fuente de verdad sigue siendo PostgreSQL.

El evento comunica que el estado cambió; no reemplaza el estado persistente.

4. Fundamental Principle

La invalidación tendrá dos mecanismos complementarios:

Explicit Invalidation
+
Version/Epoch Invalidation

La eliminación física de cache será una optimización.

La seguridad dependerá de:

current_version / current_epoch

Por lo tanto:

Un evento perdido no debe permitir que una autorización stale permanezca válida indefinidamente.

5. Event Categories

Los eventos de invalidación se clasificarán por el objeto cuyo cambio afecta autorización.

Policy
PolicyCreated
PolicyUpdated
PolicyActivated
PolicySuperseded
PolicyRetired
Roles / Permissions
RoleCreated
RoleUpdated
RolePermissionChanged
PermissionCreated
PermissionUpdated
PermissionRevoked
Membership
MembershipCreated
MembershipRoleChanged
MembershipSuspended
MembershipRevoked
MembershipReactivated
Delegation
DelegationCreated
DelegationUpdated
DelegationRevoked
DelegationExpired
Ownership
OwnershipChanged
ResourceSecurityStateChanged
Session / Security
SessionRevoked
SessionSecurityStateChanged
MFAStateChanged
SecurityStateChanged

La lista definitiva de eventos de dominio deberá mantenerse alineada con los contratos de eventos de EVOXA.

6. Event Is a Fact

Un evento debe representar:

algo que ocurrió

Ejemplo:

RolePermissionChanged

No:

InvalidateUser123Cache

La invalidación es una consecuencia del evento.

Esto mantiene separación entre:

Domain Event

y:

Cache Implementation
7. Why Not Emit Cache Events Directly

No se recomienda:

Application
   ↓
DELETE Redis

como único mecanismo.

Porque:

no es durable;
no garantiza propagación;
no tiene replay adecuado;
puede perderse ante failures;
no representa un hecho de dominio.

En cambio:

Domain Change
   ↓
Transactional Outbox
   ↓
Event

permite reconstruir el estado de invalidación.

8. Transactional Outbox

Un cambio de seguridad y su evento deberán persistirse en la misma transacción.

Ejemplo:

BEGIN

UPDATE role_permissions
SET ...

INSERT INTO outbox_events
VALUES (
   event_id,
   event_type,
   version,
   tenant_id,
   ...
)

COMMIT

Si la transacción falla:

No domain change
No invalidation event

Si commit funciona:

Domain change exists
Event exists

Esto evita inconsistencias entre estado y evento.

9. Event Envelope

Los eventos deberán utilizar el envelope estándar de EVOXA.

Conceptualmente:

{
  "event_id": "...",
  "event_type": "RolePermissionChanged",
  "event_version": 1,
  "timestamp": "...",
  "tenant_id": "...",
  "actor": {
    "type": "USER",
    "id": "..."
  },
  "resource": {
    "type": "ROLE",
    "id": "..."
  },
  "payload": {},
  "metadata": {
    "correlation_id": "...",
    "causation_id": "...",
    "trace_id": "..."
  }
}

No deben incluirse:

passwords;
tokens;
MFA secrets;
API credentials;
private keys;
datos innecesarios.
10. Invalidation Metadata

Para permitir invalidación eficiente, el evento deberá transportar metadata suficiente para determinar:

what changed
where
which tenant
which security scope
which version/epoch

Conceptualmente:

invalidation_scope
entity_type
entity_id
tenant_id
security_epoch
entity_version

El formato exacto será definido por el contrato de evento.

11. Invalidation Scope

Cada evento deberá tener un scope conceptual:

PLATFORM
ORGANIZATION
MEMBERSHIP
ROLE
PERMISSION
RESOURCE
SESSION
DELEGATION
SECURITY

Esto evita invalidaciones innecesariamente globales.

12. Policy Invalidation

Cuando una policy cambia:

PolicyUpdated

el sistema deberá:

increment policy version/epoch
+
publish event

Los consumidores:

invalidate affected policy cache
invalidate dependent decision cache

Las decisiones antiguas se consideran stale.

13. Policy Activation

Un draft no necesariamente afecta autorización.

Por eso:

PolicyDraftCreated

no debe necesariamente invalidar authorization decisions.

Pero:

PolicyActivated

sí.

La invalidación deberá estar asociada al momento en que la policy se vuelve efectiva.

14. Policy Retirement

Cuando una policy deja de estar activa:

PolicyRetired

debe invalidar las entradas que dependan de ella.

No se debe esperar únicamente al TTL.

15. Role Permission Change

Cuando se modifica la relación:

Role → Permission

deberá actualizarse el version/epoch correspondiente.

Ejemplo:

role_epoch
permission_epoch

según el scope afectado.

Esto invalidará:

permission resolution cache

y las decision caches dependientes.

16. Membership Role Change

Si:

Membership A
   ↓
Role Coach

cambia a:

Role Admin

debe incrementarse el estado/version correspondiente de membership.

Las decisiones antiguas deberán quedar stale.

17. Membership Revocation

La revocación de membership es un cambio de seguridad crítico.

Flujo:

MembershipRevoked
        ↓
membership security version++
        ↓
event published
        ↓
L1 invalidation
        ↓
L2 invalidation/version update
        ↓
future authorization
        ↓
DENY

No debe depender solamente de TTL.

18. Delegation Revocation

Cuando una delegación es revocada:

DelegationRevoked

deberá invalidarse su ámbito.

Ejemplo:

delegation_epoch++

Cualquier cached ALLOW basado en esa delegation será stale.

19. Ownership Change

Cuando cambia el ownership:

OwnershipChanged

debe actualizarse:

ownership_version

y las decisiones que dependan de ownership deberán dejar de ser válidas.

20. Session Revocation

Cuando una sesión es revocada:

SessionRevoked

el cambio debe propagarse a la infraestructura de authorization state.

Las decisiones que dependan de esa sesión deberán considerarse stale.

Este mecanismo debe mantenerse alineado con:

ADR-003;
ADR-008;
ADR-009;
ADR-031;
ADR-033.
21. MFA State Change

Cambios relevantes en MFA:

MFAStateChanged

pueden afectar:

authentication assurance
step-up evidence
trusted device
security state

Por ello deben invalidar las decisiones dependientes del contexto MFA.

Nunca se debe utilizar una invalidación de MFA para conceder privilegios.

22. L1 Local Cache Propagation

Cada instancia debe recibir eventos de invalidación.

Ejemplo:

Event Platform
       │
       ├── Instance A
       ├── Instance B
       ├── Instance C
       └── Instance D

Cada instancia actualiza:

local version/epoch

y elimina o invalida entradas correspondientes.

23. L2 Redis Invalidation

Redis puede recibir:

delete
invalidate
version update
namespace update

dependiendo de la implementación.

Pero el mecanismo físico exacto no se considera parte de este ADR.

Lo normativo es:

Una entrada cuyo security version ya no coincide debe ser tratada como inválida aunque todavía exista físicamente en Redis.

24. Version-Based Safety

Ejemplo:

Cached:
role_epoch = 10

Luego:

RolePermissionChanged
role_epoch = 11

Una instancia que todavía tiene físicamente:

cache entry role_epoch=10

debe considerarla stale.

Por tanto:

event invalidation

y:

version validation

se complementan.

25. Event Ordering

Los eventos relacionados con el mismo security scope deben tener ordering suficiente para mantener coherencia.

Ejemplo:

RolePermissionChanged v11
RolePermissionChanged v12
RolePermissionChanged v13

No se debe permitir:

v13
v11
v12

que haga retroceder el estado.

Por ello, ordering será scoped, no global.

26. Partitioning Key

La partición del event stream deberá utilizar una clave que preserve el ordering requerido.

Candidatos conceptuales:

tenant_id

o:

tenant_id + entity_id

dependiendo del tipo de evento.

La selección exacta se definirá en el contrato/event architecture correspondiente.

27. Out-of-Order Events

Si llega:

version = 10

cuando el consumidor conoce:

version = 12

deberá evitarse retroceder:

12 → 10

La regla será:

incoming_version <= current_version
    → do not regress

Puede registrarse como:

stale_event_ignored

para observabilidad.

28. Duplicate Events

Debido a At-Least-Once delivery:

PolicyUpdated v12
PolicyUpdated v12

puede recibirse más de una vez.

El consumidor debe ser idempotente.

El segundo evento:

no-op

desde el punto de vista del estado.

29. Idempotency

Cada evento tendrá:

event_id

y/o una combinación equivalente de:

entity
version
scope

para detectar duplicados.

Esto debe integrarse con el modelo de consumer idempotency establecido en ADR-022.

30. Propagation Protocol

El protocolo conceptual será:

1. Domain state changes
2. Security version increments
3. Outbox event committed
4. Event published
5. Consumer receives event
6. Consumer validates event
7. Consumer checks version
8. Consumer updates local security version
9. Consumer invalidates L1
10. Consumer invalidates/reconciles L2
11. Consumer acknowledges event
31. Acknowledgement

Un evento sólo debe considerarse procesado cuando:

security version update
+
required invalidation action

hayan alcanzado el estado requerido por el consumidor.

Si ocurre:

process
↓
failure

antes del acknowledgement:

retry
32. Retry

Los consumidores deberán implementar retries para fallos transitorios:

Event
 ↓
Consumer
 ↓
temporary failure
 ↓
retry

No se debe descartar silenciosamente un evento de seguridad.

33. Dead Letter Queue

Si un evento no puede procesarse después de los retries establecidos:

Event
 ↓
Retry
 ↓
Retry
 ↓
Retry
 ↓
DLQ

Pero:

Un evento de invalidación de seguridad enviado a DLQ no significa que la autorización pueda continuar usando automáticamente la entrada stale.

Debe existir una estrategia segura de recuperación.

34. Fail-Safe During Propagation Failure

Este es uno de los puntos más importantes.

Supongamos:

PostgreSQL
permission revoked

pero:

Event Platform unavailable

Una instancia podría conservar:

L1 ALLOW

Por eso la arquitectura no debe depender exclusivamente del evento.

Las decisiones deben incorporar:

version / epoch validation

y las operaciones críticas podrán requerir consulta fresh.

35. Maximum Stale Window

La arquitectura deberá definir posteriormente un:

maximum acceptable stale authorization window

Este valor será una decisión de seguridad/SLO.

No se fija aquí porque debe considerar:

event latency;
Redis latency;
PostgreSQL latency;
risk;
security classification;
cross-region propagation.
36. Critical Security Bypass

Para operaciones críticas:

MFA changes
Role administration
Permission administration
Tenant security administration
Credential/security changes

podrá utilizarse:

fresh state validation

incluso si existe una cache entry.

Esto proporciona una protección adicional durante ventanas de propagación.

37. Cache Invalidation Does Not Execute Authorization

El consumer de invalidación no debe decidir:

ALLOW
DENY

Su responsabilidad es:

maintain cache freshness

La autorización continúa perteneciendo al:

Authorization Runtime

definido en ADR-049.

38. Invalidation Consumer Architecture

Conceptualmente:

                 Event Platform
                       │
                       ▼
            Authorization Invalidation
                    Consumer
                       │
          ┌────────────┼─────────────┐
          ▼            ▼             ▼
     Version Store    L1          Redis/L2
          │
          └────────────┬────────────┘
                       ▼
              Authorization Runtime

El consumer no debe convertirse en un segundo Authorization Engine.

39. Event-to-Invalidation Matrix
Event	Primary Version	L1	L2	Decision Cache
PolicyActivated	Policy	Invalidate	Invalidate/reconcile	Affected
PolicyUpdated	Policy	Invalidate	Invalidate/reconcile	Affected
PolicyRetired	Policy	Invalidate	Invalidate/reconcile	Affected
RolePermissionChanged	Role/Permission	Invalidate	Invalidate/reconcile	Affected
MembershipRoleChanged	Membership	Invalidate	Invalidate/reconcile	Affected
MembershipRevoked	Membership/Security	Immediate	Immediate	Affected
DelegationRevoked	Delegation	Immediate	Immediate	Affected
OwnershipChanged	Ownership	Affected	Affected	Affected
SessionRevoked	Session/Security	Immediate	Immediate	Affected
MFAStateChanged	Security/MFA	Affected	Affected	Affected
SecurityStateChanged	Security	Immediate	Immediate	Affected

“Affected” significa que el scope concreto debe determinarse mediante las dependencias de la cache identity.

40. Global Security Events

Algunos eventos pueden afectar todo el sistema:

PlatformSecurityPolicyChanged
GlobalSecurityEpochChanged

Estos pueden provocar:

platform_security_epoch++

y una invalidación amplia.

Pero deben utilizarse excepcionalmente para evitar:

global cache invalidation storm
41. Invalidation Storm Protection

Un cambio masivo podría generar:

millions of invalidations

La arquitectura debe permitir invalidación lógica mediante:

epoch increment

en vez de borrar físicamente cada key.

Esto reduce:

Redis load;
network traffic;
CPU;
latency.
42. Batch Invalidation

Cuando múltiples eventos afectan el mismo scope:

RolePermissionChanged
RolePermissionChanged
RolePermissionChanged

pueden coalescerse cuando sea seguro.

Ejemplo:

role_epoch:
10 → 11 → 12 → 13

Una instancia puede finalmente observar:

role_epoch = 13

sin necesitar ejecutar tres ciclos físicos de eliminación.

43. Cache Rebuild

Después de invalidación:

cache miss

el runtime reconstruirá la entrada utilizando:

current PostgreSQL state
+
current versions

No debe reconstruirse desde:

old cache
44. Cache Warming

Opcionalmente podrá existir:

event
 ↓
invalidate
 ↓
precompute frequently-used policy data

Pero el warming no es obligatorio para correctness.

Si falla:

warming failure

el sistema debe seguir funcionando mediante cache miss + evaluation.

45. Recovery After Consumer Failure

Si un consumidor estuvo offline:

Consumer offline
     ↓
events accumulate

al regresar deberá:

recuperar eventos;
procesarlos idempotentemente;
actualizar versiones;
invalidar caches;
validar estado actual.

No debe asumir que:

no recibió eventos = no hubo cambios
46. Recovery After Event Loss

Si existe evidencia de pérdida o inconsistencia de eventos:

reconciliation

debe comparar:

event-derived state

contra:

authoritative PostgreSQL state

y reconstruir:

versions
cache validity

La reconciliación será una operación de recuperación, no un mecanismo normal.

47. Replay

El sistema deberá soportar replay de eventos de invalidación.

Ejemplo:

Replay Policy Events
        ↓
Reconstruct security versions
        ↓
Rebuild/invalidate caches

El replay debe ser:

explícito;
controlado;
auditable;
idempotente.

No debe producir duplicación de business outcomes.

48. Event Schema Evolution

Los eventos de invalidación estarán versionados.

Ejemplo:

RolePermissionChanged.v1
RolePermissionChanged.v2

Cambios breaking requieren nueva versión.

Los consumidores deben mantener compatibilidad según las reglas del Schema Registry.

49. Security of Event Consumers

Los consumers deben utilizar:

identidad de servicio;
least privilege;
tenant-aware processing;
autenticación service-to-service;
autorización centralizada.

No se deben conceder permisos administrativos simplemente porque un proceso tenga capacidad de invalidar cache.

50. AI and Event Invalidation

AI agents no pueden:

emit security invalidation event

para modificar indirectamente authorization state, salvo que exista una operación explícitamente autorizada.

Un agente AI no podrá utilizar:

cache invalidation

como mecanismo para:

obtener ALLOW;
ocultar una policy;
eliminar audit trail;
evadir controles.
51. Observability

Se deberán medir:

authorization_cache_invalidation_events_total
authorization_cache_invalidation_latency
authorization_cache_invalidation_failures
authorization_cache_invalidation_retries
authorization_cache_invalidation_dlq_total
authorization_cache_stale_events_total
authorization_cache_duplicate_events_total
authorization_cache_out_of_order_events_total
authorization_cache_reconciliation_total
authorization_cache_propagation_lag

Especialmente importante:

propagation_lag

que representa cuánto tiempo pasa entre:

security change

y:

all affected caches updated/invalidated
52. Security SLO

Deberá definirse posteriormente un SLO específico para:

security change → authorization cache convergence

Ejemplo conceptual:

P95 propagation < X
P99 propagation < Y

Los valores no se fijan en este ADR.

Para operaciones críticas puede existir un requisito más estricto mediante fresh validation.

53. Audit

Los siguientes eventos deben ser auditables:

policy changes;
permission changes;
role changes;
membership revocation;
delegation revocation;
security-state changes.

La auditoría del cambio de seguridad es independiente de:

cache invalidation logs

El cache consumer no sustituye AuditService.

54. Security Failure Scenarios
Scenario A — Redis Down
Redis DOWN
↓
L2 unavailable
↓
fresh evaluation / safe L1

Nunca:

Redis DOWN → ALLOW
Scenario B — Event Platform Down
Event Platform DOWN
↓
version mismatch / fresh state validation

Critical operations deberán usar fresh validation cuando sea necesario.

Scenario C — Consumer Down
Consumer DOWN
↓
events retained
↓
consumer recovers
↓
replay
Scenario D — Duplicate Event
same event twice
↓
idempotent processing
Scenario E — Out-of-Order Event
v13
v11
↓
v11 ignored
Scenario F — Stale L1
L1 old
↓
version mismatch
↓
reject cache
↓
fresh evaluation
55. Consequences
Positives
Stronger Security

Reduce stale authorization risk.

Distributed Consistency

Permite sincronizar múltiples API instances.

Scalability

Epoch invalidation evita eliminar millones de entries individualmente.

Resilience

Events + versions + reconciliation ofrecen múltiples mecanismos de recuperación.

Observability

Propagation lag y invalidation failures pueden medirse.

56. Negative Consequences

Introduce complejidad en:

event contracts;
consumers;
retries;
DLQ;
ordering;
versioning;
reconciliation;
cache invalidation;
operational monitoring.

También aumenta el coste de mantener consistencia distribuida.

Esto es aceptable debido a la criticidad del Authorization Domain.

57. Alternatives Rejected
A. Direct Redis Deletion

Rechazado.

No ofrece durabilidad ni propagation robusta.

B. Local Cache TTL Only

Rechazado.

Una revocación podría permanecer válida demasiado tiempo.

C. Global Cache Flush

Rechazado.

Es costoso y no escalable.

D. Event-Only Invalidation

Rechazado.

Un evento perdido no debe convertirse en una autorización stale indefinida.

E. Version-Only Without Events

Rechazado como mecanismo completo.

Puede garantizar safety, pero perjudica convergence/performance.

F. Authorization Consumer as PDP

Rechazado.

El consumer sólo mantiene cache freshness.

El Authorization Runtime sigue siendo el PDP lógico.

58. Non-Negotiable Rules
Security changes deben generar eventos mediante transactional outbox.
Events representan hechos, no instrucciones de cache.
Invalidation events son At-Least-Once.
Consumers deben ser idempotentes.
Out-of-order events no pueden retroceder versiones.
Event loss no puede permitir stale ALLOW indefinido.
Tenant isolation debe mantenerse durante propagation.
Critical security changes deben tener invalidación inmediata o fresh validation.
L1 debe recibir/derivar invalidation.
L2 debe mantenerse coherente mediante invalidation/versioning.
Version/epoch es el mecanismo de seguridad; deletion es optimización.
Redis no es source of truth.
Event consumer no decide ALLOW/DENY.
No secrets en eventos.
AI agents no pueden manipular cache para modificar autorización.
Replay debe ser idempotente y auditable.
DLQ no puede convertirse en autorización stale silenciosa.
Propagation failures deben ser observables.
Cache invalidation no reemplaza AuditService.
Fail-closed permanece obligatorio.
59. Traceability
Blueprint

Este ADR deriva de:

Identity Domain;
Authorization;
RBAC;
ABAC;
Tenant Isolation;
Security Policy Engine;
Event-Driven Architecture;
Transactional Outbox;
At-Least-Once Delivery;
Idempotent Consumers;
Retry/DLQ;
Replay;
Observability.

El modelo de seguridad contempla RBAC/ABAC, tenant isolation, ownership y políticas contextuales, haciendo que los cambios en cualquiera de estos elementos puedan afectar decisiones cacheadas.

La arquitectura de eventos del Blueprint exige eventos inmutables/versionados, entrega At-Least-Once, consumidores idempotentes y mecanismos de retry/DLQ/replay.

Implementation Stories

Principalmente:

IS-006 — Authorization & Permission Evaluation
IS-007 — Role & Permission Management
IS-008 — Session Management
IS-010 — Multi-Factor Authentication
IS-011 — Security Policy Engine
IS-012 — Audit & Security Events
IS-013 — Domain Events & Outbox
IS-015 — Identity Persistence
IS-016 — Identity Testing & Quality
IS-017 — Identity Runtime & Deployment
60. Related ADRs
ADR-IDENTITY-007 — Transactional Outbox & Event Delivery
ADR-IDENTITY-018 — Central Shared Event Broker
ADR-IDENTITY-019 — Partitioned Event Streaming/Routing
ADR-IDENTITY-020 — Schema Registry & Compatibility
ADR-IDENTITY-021 — Controlled Replay
ADR-IDENTITY-022 — Consumer Idempotency/Deduplication
ADR-IDENTITY-028 — Observability & Operational Security
ADR-IDENTITY-029 — Disaster Recovery & Backup
ADR-IDENTITY-031 — Identity HA & Failover
ADR-IDENTITY-043 — Multi-Organization Identity
ADR-IDENTITY-044 — Membership Roles/Delegation
ADR-IDENTITY-045 — Authorization Policy Evaluation
ADR-IDENTITY-046 — Policy Language & Governance
ADR-IDENTITY-047 — Policy Engine Technology
ADR-IDENTITY-048 — Policy DSL
ADR-IDENTITY-049 — Authorization Evaluation Runtime
ADR-IDENTITY-050 — Cache, Invalidation & Consistency
ADR-IDENTITY-051 — Cache Key, Versioning & Context Fingerprinting
ADR-IDENTITY-052 — Cache Eligibility, TTL & Security-Sensitive Decisions
61. Pending Decisions

Quedan para especificaciones posteriores:

Exact event names.
Exact event payload schemas.
Event version numbering.
Exact partition keys.
Exact propagation topology.
Redis invalidation mechanism.
L1 invalidation mechanism.
Version store implementation.
Maximum propagation latency.
Critical security propagation SLA.
Retry policy.
Backoff strategy.
DLQ retention.
Reconciliation frequency.
Event replay procedure.
Batch/coalescing rules.
Cache warming after invalidation.
Cross-region propagation.
Multi-region conflict handling.
Exact security-event classifications.
Event consumer deployment model.
Consumer concurrency.
Ordering guarantees per scope.
Failure escalation procedure.
62. Acceptance Criteria
AC-01 — Transactional Event

Un cambio de seguridad debe persistir el estado y el outbox event en una única transacción.

AC-02 — L1 Propagation

Una invalidación relevante debe poder propagarse a las instancias que mantienen L1 cache.

AC-03 — L2 Propagation

La estrategia debe permitir invalidar o hacer stale la información correspondiente en L2.

AC-04 — Version Safety

Una entrada con una versión anterior no podrá producir ALLOW cuando exista una versión posterior aplicable.

AC-05 — Duplicate Safety

Procesar dos veces el mismo evento debe producir el mismo estado que procesarlo una vez.

AC-06 — Ordering Safety

Un evento antiguo no podrá revertir una versión más nueva.

AC-07 — Membership Revocation

Una membership revocada no podrá continuar autorizando mediante cache.

AC-08 — Role Permission Revocation

Una revocación de permisos debe invalidar las decisiones afectadas.

AC-09 — Delegation Revocation

Una delegation revocada no podrá seguir proporcionando privilegios.

AC-10 — Ownership Change

Un cambio de ownership debe invalidar las decisiones dependientes.

AC-11 — Session Revocation

Una sesión revocada no podrá mantener decisiones cacheadas incompatibles con su nuevo estado.

AC-12 — MFA

Una modificación relevante del estado MFA no podrá permitir reutilización incorrecta de una autorización anterior.

AC-13 — Event Platform Failure

La caída del Event Platform no podrá producir un authorization bypass.

AC-14 — Redis Failure

La caída de Redis no podrá producir un authorization bypass.

AC-15 — Consumer Failure

Un consumer que estuvo offline deberá poder recuperar y reconciliar su estado.

AC-16 — DLQ

Los eventos que fallen repetidamente deberán poder llegar a DLQ sin perder trazabilidad.

AC-17 — Replay

Los eventos de invalidación deberán poder reprocesarse de forma idempotente.

AC-18 — Tenant Isolation

Un evento de Tenant A no podrá invalidar o modificar accidentalmente el estado de Tenant B.

AC-19 — Critical Operations

Las operaciones críticas deberán poder utilizar fresh security state cuando la propagation consistency no sea suficiente.

AC-20 — Observability

Debe poder medirse el tiempo entre:

security change

y:

cache convergence
63. Architectural Result

Con los ADR-050, 051, 052 y 053 queda cerrada conceptualmente la arquitectura completa de Authorization Cache:

                         ┌──────────────────┐
                         │   PostgreSQL     │
                         │ Source of Truth  │
                         └────────┬─────────┘
                                  │
                         Transactional Change
                                  │
                    ┌─────────────┴─────────────┐
                    │                           │
              Domain State                 Outbox Event
                                                │
                                                ▼
                                      ┌──────────────────┐
                                      │  Event Platform  │
                                      └────────┬─────────┘
                                               │
                              ┌────────────────┼────────────────┐
                              ▼                ▼                ▼
                         Instance A       Instance B       Instance C
                              │                │                │
                              ▼                ▼                ▼
                         L1 Cache          L1 Cache          L1 Cache
                              │                │                │
                              └────────────────┼────────────────┘
                                               ▼
                                           Redis L2
                                               │
                                               ▼
                                    Version / Epoch State
                                               │
                                               ▼
                                   Authorization Runtime
                                               │
                         ┌─────────────────────┼────────────────────┐
                         ▼                     ▼                    ▼
                       RBAC                Ownership              ABAC
                         │                     │                    │
                         └─────────────────────┼────────────────────┘
                                               ▼
                                          Delegation
                                               │
                                               ▼
                                      Policy Evaluation
                                               │
                                               ▼
                                  Final Authorization Decision

Y el principio de seguridad queda consolidado:

Los eventos proporcionan propagación; las versiones proporcionan seguridad contra stale state; la caché proporciona performance; PostgreSQL mantiene la autoridad del estado; y el Authorization Runtime mantiene la autoridad sobre la decisión.

Siguiente ADR recomendado

El siguiente paso lógico sería:

ADR-IDENTITY-054 — Authorization Cache Failure, Degradation & Recovery Strategy

Este ADR debería cerrar qué ocurre cuando algo falla, definiendo formalmente el comportamiento ante:

Redis DOWN
L1 corruption
Event Platform DOWN
Consumer DOWN
PostgreSQL latency/failure
Event delivery lag
DLQ
Version Store unavailable
Cache stampede
Network partition
Multi-region partition

y, especialmente, cuándo EVOXA debe:

usar cache → bypass cache → consultar PostgreSQL → DENY → REQUIRE_MFA → bloquear operación crítica.

Ese ADR nos dejaría prácticamente cerrada la arquitectura de resiliencia del Authorization Cache antes de entrar en los Technical Tasks.
