ADR-IDENTITY-050 — Authorization Policy Cache, Invalidation & Consistency Strategy

Status: Proposed
Version: 1.0
Date: 2026-09-03
Domain: Identity / Authorization
Priority: Critical
Type: Architecture / Security / Performance

1. Context

El motor de autorización de EVOXA debe evaluar solicitudes de acceso de forma:

determinista;
segura;
tenant-aware;
fail-closed;
auditable;
suficientemente rápida para cumplir los objetivos de rendimiento definidos para Identity.

El ADR-IDENTITY-049 — Authorization Policy Evaluation Runtime & Decision Algorithm establece que el runtime debe seleccionar y evaluar políticas activas, resolver RBAC, ownership, ABAC y delegaciones, aplicar precedencia y producir una decisión final.

También establece objetivos de rendimiento de aproximadamente:

Authorization <50 ms
Permission Resolution <30 ms

Por lo tanto, consultar PostgreSQL y reconstruir todo el contexto de políticas para cada autorización no debe ser el comportamiento normal del runtime.

Sin embargo, una caché de autorización introduce un problema de seguridad:

Una decisión de autorización almacenada en caché puede quedar obsoleta después de que cambie una política, rol, permiso, membership, ownership, estado de sesión, suspensión de usuario u otra condición relevante.

La caché, por lo tanto, no puede convertirse en una fuente de verdad de seguridad.

Este ADR define cómo EVOXA utilizará caching para authorization policies y cómo garantizará la invalidación y consistencia necesarias.

2. Problem

El sistema debe resolver simultáneamente cuatro objetivos:

Performance
evitar evaluar repetidamente políticas idénticas;
reducir consultas a PostgreSQL;
soportar alta concurrencia.
Seguridad
impedir que una autorización revocada continúe siendo válida indefinidamente;
evitar cross-tenant cache pollution;
evitar privilege escalation por entradas obsoletas.
Consistencia
propagar cambios de políticas y permisos;
invalidar caches entre múltiples instancias;
manejar correctamente carreras entre evaluación e invalidación.
Disponibilidad
mantener el sistema operativo ante pérdida parcial de Redis;
no transformar una falla de cache en un bypass de seguridad.
3. Decision

EVOXA adoptará una estrategia de multi-level authorization caching, donde:

                 PostgreSQL
             Source of Truth
                    │
                    ▼
          Policy / Permission Data
                    │
                    ▼
          ┌───────────────────┐
          │ Authorization     │
          │ Cache Layer       │
          └───────────────────┘
             │             │
        Local Cache       Redis
             │             │
             └──────┬──────┘
                    ▼
        Authorization Runtime
                    │
                    ▼
             Final Decision

La regla fundamental será:

La caché mejora performance, pero nunca redefine la autoridad de seguridad.

PostgreSQL y el estado transaccional del dominio permanecen como fuente de verdad.

4. Cache Layers
4.1 L1 — Local Process Cache

Cada instancia del API/Authorization Runtime podrá mantener una caché local en memoria.

Propósito:

reducir latencia;
evitar round-trips a Redis;
reducir carga sobre infraestructura compartida.

Ejemplos:

CompiledPolicySet
ResourceRegistry
ActionRegistry
AttributeRegistry
PermissionResolution
PolicyMetadata

La caché local debe tener:

TTL;
límite de tamaño;
eviction;
version/epoch;
invalidación distribuida.

No debe almacenar indefinidamente decisiones de autorización críticas.

5. L2 — Distributed Cache

Redis será el candidato inicial para la caché distribuida.

Su función será compartir información cacheable entre múltiples instancias:

API Instance A
      │
API Instance B ─── Redis
      │
API Instance C

Redis podrá almacenar:

políticas compiladas;
conjuntos de políticas;
permission resolution;
metadata de políticas;
versiones/epochs;
información temporal necesaria para invalidación.

Redis no será la fuente primaria de verdad.

Si Redis desaparece:

Redis unavailable
       ↓
cache miss
       ↓
PostgreSQL
       ↓
rebuild cache

No:

Redis unavailable
       ↓
ALLOW
6. What Should Be Cached

La estrategia distingue entre datos de políticas y decisiones finales de autorización.

6.1 Policy Data

La siguiente información puede ser cacheada:

políticas activas;
versiones de políticas;
policy metadata;
targets;
conditions;
obligations;
compiled policy;
AST;
resource/action mappings;
permission sets;
role-to-permission resolution.

Estos elementos son buenos candidatos porque son relativamente estables.

7. Compiled Policy Cache

El runtime no debería tener que interpretar desde cero una política estructurada en cada request.

Pipeline:

Policy Definition
      ↓
Validation
      ↓
Parsing
      ↓
AST
      ↓
Compilation
      ↓
Compiled Policy
      ↓
Cache
      ↓
Authorization Runtime

La caché podrá utilizar una clave conceptual como:

policy:{tenant}:{policy_id}:{version}

o, para conjuntos:

policy-set:{tenant}:{scope}:{version}

La clave exacta queda como decisión de implementación.

8. Authorization Decision Cache

El cacheo de decisiones finales será mucho más restrictivo.

Una decisión:

ALLOW

no debe almacenarse únicamente con:

user_id + resource + action

porque la autorización depende de contexto.

El resultado puede depender de:

usuario;
organización;
membership;
roles;
permisos;
recurso;
ownership;
sesión;
MFA;
device trust;
risk;
IP;
tiempo;
delegación;
estado de cuenta;
policy version.

Por lo tanto:

No se permitirá un decision cache genérico que ignore atributos relevantes del contexto.

9. Decision Cache Key

Cuando se utilice decision caching, la clave debe representar todas las variables de seguridad que influyen en la decisión.

Conceptualmente:

authorization-decision:
    tenant
    actor
    membership
    resource
    action
    policy_version
    role_version
    permission_version
    ownership_version
    session_security_version
    delegation_version
    relevant_context_hash

No necesariamente todos estos valores tienen que aparecer literalmente en la clave.

Puede utilizarse:

context_hash

calculado de forma determinista.

Ejemplo:

authz:
  tenant:123
  actor:456
  resource:workout:789
  action:edit
  context_version:abc123
10. Tenant Isolation

La caché debe ser estrictamente tenant-aware.

Nunca se permitirá:

authz:user:123:resource:456

si la clave no permite distinguir correctamente el tenant.

Debe existir separación conceptual:

tenant-A → cache namespace A
tenant-B → cache namespace B

Un cache hit nunca podrá producir autorización en un tenant distinto del contexto solicitado.

Esto es un non-negotiable security control.

11. Version-Based Invalidation

La estrategia principal será versioning/epochs, complementada por invalidación basada en eventos.

En lugar de depender exclusivamente de:

DELETE cache:key

se utilizarán versiones de seguridad.

Conceptualmente:

Tenant Authorization Epoch
Role Epoch
Permission Epoch
Policy Epoch
Membership Epoch
Session Security Epoch
Delegation Epoch

Ejemplo:

policy_version = 42

Después de modificar una política:

policy_version = 43

Una entrada creada con:

version = 42

queda obsoleta.

12. Why Versioning Is Preferred

La invalidación mediante eliminación directa puede fallar en escenarios distribuidos.

Ejemplo:

Instance A → cache hit
Instance B → policy update
Instance B → DELETE cache
Instance C → stale local cache

El versioning permite detectar:

cached_version != current_version

y tratar la entrada como inválida.

Por ello:

Version/epoch validation será el mecanismo de seguridad; explicit invalidation será un mecanismo de optimización.

13. Event-Driven Cache Invalidation

Los cambios relevantes emitirán eventos mediante el sistema de eventos de EVOXA.

Ejemplos:

PolicyCreated
PolicyUpdated
PolicyActivated
PolicySuperseded
PolicyRetired

RolePermissionChanged
MembershipRoleChanged
MembershipSuspended
MembershipRevoked

DelegationCreated
DelegationRevoked

SecurityStateChanged
SessionRevoked

El flujo será:

Domain Transaction
      │
      ├── PostgreSQL change
      │
      └── Outbox event
              │
              ▼
        Event Platform
              │
       ┌──────┼──────┐
       ▼      ▼      ▼
    API A   API B   API C
       │      │      │
       └── invalidate ──┘

Esto mantiene consistencia entre instancias.

14. Transactional Outbox

La actualización de seguridad y el evento de invalidación deben mantener la garantía establecida por ADR-IDENTITY-007 e IS-013.

Ejemplo:

BEGIN TRANSACTION

UPDATE authorization_policy
SET version = 43

INSERT INTO outbox_events (...)

COMMIT

Nunca:

UPDATE policy

publish event

sin protección transaccional.

Si el proceso falla después del update pero antes del publish, el outbox permite recuperar el evento.

15. At-Least-Once Delivery

La invalidación recibirá eventos At-Least-Once.

Por lo tanto, un consumidor puede recibir:

PolicyUpdated v43
PolicyUpdated v43
PolicyUpdated v43

Esto debe ser seguro.

Los consumidores de invalidación serán idempotentes.

16. Stale Event Protection

Los eventos pueden llegar fuera de orden.

Ejemplo:

PolicyUpdated v43
PolicyUpdated v42

El consumidor nunca debe retroceder la versión:

current = 43
incoming = 42

ignore

Regla:

if incoming_version <= current_version:
    ignore

Conceptualmente.

Esto protege frente a eventos retrasados o duplicados.

17. Invalidation Scope

No todas las modificaciones requieren invalidar todo.

Se utilizará invalidación granular cuando sea posible.

Ejemplo:

Policy changed
      ↓
invalidate policy cache
      ↓
invalidate affected authorization contexts

En cambio:

Global security policy changed
      ↓
invalidate broader policy namespace

El objetivo será minimizar:

cache stampede;
latencia;
carga sobre PostgreSQL;
reconstrucción innecesaria.
18. Security-Critical Changes

Determinados cambios deben provocar invalidación inmediata o incremento de epoch.

Entre ellos:

suspensión de usuario;
deshabilitación de cuenta;
revocación de membership;
cambio crítico de permisos;
revocación de rol;
modificación de policy de seguridad;
revocación de delegation;
revocación de sesión;
cambios relacionados con MFA;
cambios de security state.

La caché no debe prolongar artificialmente estos estados.

19. TTL Strategy

El TTL será complementario al versioning.

Conceptualmente:

TTL + Version Validation + Event Invalidation

No:

TTL only

Un TTL largo podría mantener información incorrecta demasiado tiempo.

Un TTL demasiado corto eliminaría gran parte del beneficio del caching.

Los valores exactos quedan como pending implementation decision y deberán validarse mediante performance/security testing.

20. Fail-Closed Behavior

Si el runtime no puede determinar con suficiente confianza que una entrada cacheada es válida:

UNKNOWN

no debe transformarse en:

ALLOW

Debe producir:

DENY

o un resultado seguro apropiado según la semántica del runtime.

Esto sigue directamente la regla de ADR-IDENTITY-049:

internal errors/indeterminate states cannot produce ALLOW.

21. Redis Failure

Si Redis no está disponible:

Redis failure
     ↓
L2 unavailable
     ↓
L1 cache if safely valid
     ↓
otherwise PostgreSQL

El sistema debe seguir pudiendo tomar decisiones de autorización utilizando la fuente primaria cuando sea necesario.

Sin embargo:

Redis failure ≠ authorization bypass
22. Local Cache Failure

Una caché local puede descartarse completamente:

L1 corrupted/expired
       ↓
discard
       ↓
L2 / PostgreSQL

No debe existir dependencia funcional crítica de la persistencia de L1.

23. Cache Stampede Protection

Cuando una entrada expira o es invalidada, múltiples requests pueden intentar reconstruirla simultáneamente.

Ejemplo:

1000 requests
     ↓
cache miss
     ↓
1000 PostgreSQL queries

Esto debe evitarse mediante mecanismos como:

single-flight;
request coalescing;
distributed lock;
jittered expiration;
background refresh.

La técnica exacta queda pendiente.

24. Cache Poisoning Protection

Las entradas de autorización deben generarse exclusivamente desde fuentes confiables.

No se permitirá:

que el cliente escriba directamente en Redis;
que un usuario controle cache keys;
que un agente AI escriba decisiones directamente;
que información no validada se convierta en policy cache.

Todo cache entry deberá derivarse del runtime autorizado.

25. Cache and AI Agents

Los agentes AI utilizan el mismo Authorization Runtime.

Por lo tanto:

AI Agent
   ↓
Authorization Request
   ↓
Policy Cache
   ↓
Authorization Runtime

El AI no podrá:

modificar cache entries;
incrementar privilegios;
invalidar controles para obtener ALLOW;
manipular policy versions;
saltarse tenant isolation.

La caché será transparente al modelo de autorización.

26. Policy Updates

Una actualización de política seguirá conceptualmente:

Draft
 ↓
Validate
 ↓
Approve
 ↓
Activate
 ↓
Version++
 ↓
Persist
 ↓
Outbox
 ↓
Event Platform
 ↓
Cache Invalidation
 ↓
New policy active

Una policy nueva no deberá depender de que todas las cachés hayan expirado naturalmente.

27. Race Condition: Evaluation vs Policy Update

Escenario:

T1:
Authorization starts
Policy v42 loaded

T2:
Policy v43 activated

T3:
Authorization finishes using v42

El runtime debe definir una frontera clara de consistencia.

Para operaciones normales se permitirá una ventana extremadamente corta determinada por la arquitectura distribuida, pero para cambios críticos de seguridad deberán existir mecanismos adicionales de invalidación/versioning.

La implementación deberá impedir que una política obsoleta sea reutilizada indefinidamente.

28. Stronger Consistency for Critical Revocation

No todas las decisiones tienen el mismo riesgo.

Se establecerán categorías conceptuales:

Normal authorization

Puede utilizar:

L1 → L2 → PostgreSQL

con versioning y TTL.

Security-critical authorization

Puede requerir:

fresh security state
+
current epoch
+
policy validation

antes de ALLOW.

Ejemplos:

administración de Identity;
cambios de MFA;
modificación de permisos;
administración de tenants;
operaciones altamente privilegiadas;
acciones sensibles definidas por Security Policy Engine.
29. Cache Invalidation Observability

Los siguientes eventos deberán ser observables:

cache hit;
cache miss;
cache stale;
cache invalidation;
invalidation event received;
duplicate invalidation;
stale event ignored;
rebuild;
Redis failure;
PostgreSQL fallback;
cache stampede;
policy version mismatch.

Las métricas no deberán contener:

tokens;
passwords;
MFA secrets;
API credentials;
información sensible innecesaria.
30. Metrics

Se definirán métricas conceptuales:

authorization_cache_hit_total
authorization_cache_miss_total
authorization_cache_stale_total
authorization_cache_invalidation_total
authorization_cache_rebuild_total
authorization_cache_error_total
authorization_cache_latency
authorization_policy_version_mismatch_total
authorization_postgres_fallback_total

También:

authorization_decision_cache_hit_total
authorization_decision_cache_miss_total

si el decision cache se habilita.

31. Auditability

La invalidación de cache no sustituye Audit Events.

Los cambios relevantes deberán generar los eventos/auditoría correspondientes.

Por ejemplo:

PolicyUpdated
RolePermissionChanged
MembershipRevoked
DelegationRevoked

El cache layer no será considerado historial de seguridad.

32. Cache Retention

La caché tendrá naturaleza temporal.

Nunca deberá utilizarse como:

historial;
audit store;
source of truth;
recovery store;
business record.

Los datos pueden desaparecer en cualquier momento.

33. Data Security

La caché debe seguir las mismas reglas de seguridad del dominio Identity.

No almacenar:

passwords;
refresh tokens;
API token plaintext;
MFA secrets;
recovery codes;
private cryptographic keys;
secretos de proveedores.

Las entradas de caché deben contener únicamente la información necesaria para acelerar la evaluación.

34. Consistency Model

La estrategia adoptada será:

Policy Source of Truth

Strong consistency

PostgreSQL.

Cache

Eventually consistent for propagation, pero con mecanismos de:

versioning;
epoch validation;
TTL;
event invalidation;
fail-closed.
Critical Security Revocation

Stronger consistency requirements, pudiendo requerir consulta/validación del estado actual.

35. Decision Flow

El flujo definitivo será conceptualmente:

Authorization Request
        │
        ▼
Request Validation
        │
        ▼
Tenant Validation
        │
        ▼
Context Construction
        │
        ▼
Cache Lookup
        │
   ┌────┴─────┐
   │          │
 Valid      Miss/Stale
   │          │
   │          ▼
   │      Load Current
   │      Policy State
   │          │
   │          ▼
   │      Compile/Resolve
   │          │
   │          ▼
   │      Update Cache
   │          │
   └────┬─────┘
        ▼
Authorization Runtime
        │
        ▼
Security Guards
        │
        ▼
RBAC
        │
        ▼
Ownership
        │
        ▼
ABAC
        │
        ▼
Delegation
        │
        ▼
Policy Composition
        │
        ▼
Conflict Resolution
        │
        ▼
Final Decision
        │
        ▼
Audit / Telemetry

La caché no sustituye al Authorization Runtime.

36. Consequences
Positivas
Performance

Reduce:

queries;
policy parsing;
policy compilation;
permission resolution repetitivo.
Scalability

Permite múltiples instancias:

API 1
API 2
API 3
API N
   ↓
Redis
Security

Versioning e invalidation reducen el riesgo de stale authorization.

Resilience

Redis puede fallar sin convertirse en source of truth.

Evolución

La arquitectura permite evolucionar desde:

local cache

hacia:

distributed cache

sin cambiar el modelo de autorización.

37. Negative Consequences

Introduce complejidad en:

invalidation;
versioning;
distributed events;
concurrency;
stale data handling;
cache stampede;
observability.

También existe riesgo de:

una implementación incorrecta de cache puede introducir una vulnerabilidad de autorización.

Por esto el cache layer debe considerarse security-sensitive infrastructure.

38. Alternatives Rejected
A. No Cache

Rechazado porque dificulta cumplir:

Authorization <50ms
Permission Resolution <30ms

a escala.

B. TTL-Only Cache

Rechazado.

Una política revocada podría continuar produciendo resultados cacheados hasta la expiración.

C. Redis as Source of Truth

Rechazado.

Redis es infraestructura temporal/cache y no sustituye PostgreSQL como fuente transaccional.

D. Cache ALLOW Forever

Rechazado explícitamente.

Una autorización no es una propiedad permanente.

E. Global Cache Without Tenant Namespace

Rechazado por riesgo de cross-tenant authorization leakage.

F. First-Instance Local Cache Only

Rechazado como arquitectura definitiva porque las invalidaciones no se propagarían adecuadamente entre múltiples instancias.

39. Non-Negotiable Rules
Cache nunca es source of truth.
Nunca utilizar cache para bypass de autorización.
Tenant isolation obligatoria en cache keys/context.
Stale data no puede producir ALLOW indefinidamente.
Versioning/epochs deben participar en validación.
Cambios críticos deben provocar invalidación inmediata o incremento de versión.
Invalidation events deben ser idempotentes.
Eventos fuera de orden no pueden retroceder versiones.
Redis failure debe fallar de forma segura.
Authorization uncertainty/error debe fail closed.
AI agents utilizan exactamente el mismo modelo.
No secretos en cache.
No decisión cacheada sin considerar el contexto de seguridad relevante.
Auditabilidad de cambios de políticas y permisos se mantiene fuera del cache.
Cache no puede producir privilege amplification.
40. Traceability
Blueprint
Identity Domain
Security & Permissions
RBAC
ABAC
Tenant Isolation
Security Policy Engine
Performance
Event-Driven Architecture
Observability
Architecture
ARCHITECTURE-MAP
        ↓
Identity
        ↓
Authorization
        ↓
Policy Engine
        ↓
Authorization Runtime
        ↓
Cache / Invalidation
Implementation Stories

Principalmente:

IS-006 — Authorization & Permission Evaluation
IS-007 — Role & Permission Management
IS-011 — Security Policy Engine
IS-013 — Domain Events & Outbox
IS-015 — Identity Persistence
IS-016 — Identity Testing & Quality
IS-017 — Identity Runtime & Deployment
ADR dependencies
ADR-IDENTITY-005 — Authorization Model
ADR-IDENTITY-006 — PostgreSQL Persistence
ADR-IDENTITY-007 — Transactional Outbox
ADR-IDENTITY-019 — Partitioned Event Streaming
ADR-IDENTITY-020 — Schema Registry
ADR-IDENTITY-021 — Controlled Replay
ADR-IDENTITY-022 — Consumer Idempotency
ADR-IDENTITY-026 — API Idempotency
ADR-IDENTITY-028 — Observability
ADR-IDENTITY-031 — HA & Failover
ADR-IDENTITY-032 — Rate Limiting
ADR-IDENTITY-043 — Multi-Organization Identity
ADR-IDENTITY-044 — Membership Roles/Delegation
ADR-IDENTITY-045 — Policy Evaluation Runtime
ADR-IDENTITY-046 — Policy Language/Governance
ADR-IDENTITY-047 — Policy Engine Technology
ADR-IDENTITY-048 — Policy DSL
ADR-IDENTITY-049 — Evaluation Runtime
41. Acceptance Criteria

El diseño será considerado conforme cuando:

AC-01

Una cache hit no pueda autorizar una solicitud de otro tenant.

AC-02

Una política actualizada invalide o haga inválida su versión cacheada.

AC-03

Una política retirada no pueda permanecer indefinidamente activa por TTL.

AC-04

Eventos duplicados de invalidación sean seguros.

AC-05

Eventos fuera de orden no puedan retroceder la versión de seguridad.

AC-06

Redis unavailable no produzca ALLOW por defecto.

AC-07

Cache corruption no produzca privilege escalation.

AC-08

Authorization runtime continúe utilizando el algoritmo de ADR-049.

AC-09

Las invalidaciones críticas sean observables.

AC-10

No existan secretos en cache.

AC-11

Los AI agents no puedan manipular directamente la cache para alterar decisiones.

AC-12

Las pruebas demuestren que una revocación crítica no permanece efectiva más allá de la ventana de consistencia aprobada.

AC-13

La estrategia pueda soportar múltiples instancias concurrentes del Authorization Runtime.

AC-14

El comportamiento ante cache miss sea determinista y fail-closed.

42. Pending Decisions

Quedan deliberadamente abiertos para ADRs o especificaciones posteriores:

TTL exacto de L1.
TTL exacto de L2/Redis.
Maximum cache size.
Eviction algorithm.
Exact Redis key schema.
Exact version/epoch model.
Decision-cache eligibility.
Context hashing algorithm.
Single-flight vs distributed lock.
Cache warming strategy.
Background refresh.
Critical-security cache bypass rules.
Exact invalidation event contracts.
Event partitioning strategy específica para authorization.
Redis HA configuration.
Cache encryption requirements.
Serialization format.
Cache metrics/SLOs.
Maximum acceptable stale window.
Testing strategy para race conditions.

Estas decisiones no deben inventarse durante la implementación; deberán quedar definidas antes de implementar las partes correspondientes.

43. Recommended Next ADR

El siguiente ADR recomendado es:

ADR-IDENTITY-051 — Authorization Cache Key Schema, Versioning & Context Fingerprinting

Este ADR debería definir específicamente:

tenant
actor
membership
role
permission
resource
action
policy
ownership
delegation
session
security state
context

y cómo cada dimensión participa en:

cache key;
version;
epoch;
context fingerprint;
invalidation scope.

Esto es importante porque ADR-050 define la estrategia de caching, pero todavía no debemos fijar prematuramente el formato exacto de las claves ni el algoritmo de fingerprinting.
