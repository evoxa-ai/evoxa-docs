E17 — EVOXA Caching Architecture
Engineering Specification

Pertenece a: Engineering Specification
Volumen: volume-09-engineering
Capítulo: E17 — Caching Architecture
Anterior: E16 — Scheduling Architecture
Siguiente: E18 — EVOXA Configuration Architecture

1. Propósito

E17 define la arquitectura de caching de EVOXA.

Su responsabilidad es proporcionar mecanismos para reducir:

latencia,
carga sobre bases de datos,
llamadas repetitivas a servicios,
coste computacional,
presión sobre sistemas externos.

La regla fundamental es:

El cache acelera el acceso a información; no debe convertirse accidentalmente en la fuente primaria de verdad.

2. Objetivos

EVOXA debe soportar conceptualmente:

In-Memory Cache
Distributed Cache
Application Cache
Domain Cache
Repository Cache
API Response Cache
Computed Result Cache
Session Cache
Metadata Cache
Configuration Cache
Reference Data Cache
Negative Cache
Short-Lived Cache
Long-Lived Cache
Multi-Level Cache
3. Cache vs Source of Truth

La arquitectura debe separar:

Source of Truth
       │
       ▼
Persistent Storage
       │
       ▼
Cache

No:

Cache
  ↓
Database

como dependencia circular de autoridad.

4. Principio de Cache

Un cache debe ser:

Fast
Optional
Replaceable
Evictable
Observable
Bounded

Una aplicación correctamente diseñada debe poder recuperarse cuando el cache esté vacío.

5. Cache Failure

Si el cache falla:

Application
    │
    ▼
Cache ──X
    │
    ▼
Source of Truth

cuando la política lo permita.

El cache no debe convertirse automáticamente en un single point of failure.

6. Arquitectura General
                         Application
                              │
                              ▼
                        Cache Interface
                              │
                ┌─────────────┴─────────────┐
                ▼                           ▼
          Local Cache                 Distributed Cache
                │                           │
                └─────────────┬─────────────┘
                              ▼
                       Source of Truth
                              │
              ┌───────────────┼───────────────┐
              ▼               ▼               ▼
           Database        Service         External API
7. Cache Abstraction

Los consumidores no deberían depender directamente de una tecnología específica.

Conceptualmente:

Cache
 ├── get
 ├── set
 ├── delete
 ├── exists
 └── invalidate

La implementación puede cambiar sin modificar la lógica empresarial.

8. Cache Entry

Una entrada debe tener conceptualmente:

key
value
created_at
expires_at
version
metadata

Opcionalmente:

tenant_id
region
source
tags
9. Cache Key

Las claves deben ser:

Deterministic
Unique
Stable
Namespaced
Bounded

Ejemplo:

tenant:{tenant_id}:user:{user_id}
10. Key Namespacing

Debe evitarse:

user:123

cuando múltiples tenants puedan tener usuarios con el mismo identificador.

Preferible:

tenant:abc:user:123
11. Tenant Isolation

Los caches multi-tenant deben garantizar que:

Tenant A
   ✕
Tenant B

no puedan acceder accidentalmente a la misma entrada.

La identidad del tenant debe formar parte de la clave o estar garantizada por otro mecanismo equivalente.

12. Cache Scope

Un cache puede pertenecer a:

Process
Instance
Node
Cluster
Region
Tenant
Global

El scope debe estar explícitamente definido.

13. Local Cache

Un local cache vive dentro del proceso:

Application
   │
   ▼
Memory

Ventajas:

Very Low Latency
No Network Hop
Simple

Desventajas:

Not Shared
Lost on Restart
Potential Memory Pressure
Potential Stale Data
14. Distributed Cache

Un distributed cache permite compartir datos:

App A ─┐
App B ─┼──> Distributed Cache
App C ─┘

Es útil cuando múltiples instancias necesitan acceder al mismo conjunto de datos cacheados.

15. Multi-Level Cache

EVOXA puede utilizar:

L1 → Local Cache
L2 → Distributed Cache
L3 → Source of Truth

Flujo:

Request
   ↓
L1
   │ miss
   ▼
L2
   │ miss
   ▼
Database / Service
16. Cache Hit

Cuando la entrada existe:

Request
   ↓
Cache
   ↓
HIT
   ↓
Response

No se consulta el origen.

17. Cache Miss

Cuando no existe:

Request
   ↓
Cache
   ↓
MISS
   ↓
Source
   ↓
Cache
   ↓
Response
18. Cache Hit Ratio

Debe medirse:

cache_hits
cache_misses

y:

hit_ratio =
hits / (hits + misses)

Un cache con hit ratio muy bajo puede estar mal configurado o no aportar valor.

19. TTL

Las entradas pueden tener:

TTL

Ejemplo:

TTL = 5 minutes

Después:

Entry
  ↓
Expired
20. TTL Selection

El TTL debe depender de:

Data Volatility
Business Criticality
Read Frequency
Write Frequency
Staleness Tolerance
Cost of Source Lookup

No debe existir necesariamente un TTL universal.

21. Expiration

Puede utilizarse:

Absolute Expiration
Sliding Expiration
Absolute

Expira en un momento concreto.

Sliding

La expiración se extiende con cada acceso.

22. Stale Data

Toda estrategia de caching debe reconocer:

Cached Value
      ≠
Current Source Value

durante cierto período.

La tolerancia a datos obsoletos debe ser explícita.

23. Cache Consistency

No todos los datos requieren consistencia fuerte.

Puede clasificarse:

Strong
Near-Real-Time
Eventual
Best-Effort
24. Cache-Aside

El patrón principal recomendado:

Application
    │
    ▼
  Cache
    │
   miss
    ▼
 Source
    │
    ▼
  Cache
    │
    ▼
Application

La aplicación controla la lectura y escritura del cache.

25. Read-Through

En un modelo read-through:

Application
     │
     ▼
 Cache Layer
     │
   miss
     ▼
 Source

El propio cache obtiene el valor.

26. Write-Through

La escritura pasa por el cache:

Application
     │
     ▼
 Cache
     │
     ▼
 Source

Puede facilitar determinadas garantías de consistencia, aunque aumenta la complejidad.

27. Write-Behind

La escritura puede ser diferida:

Application
     │
     ▼
 Cache
     │
     ▼
Async Persistence

Debe utilizarse únicamente cuando la pérdida temporal de datos o recuperación posterior sean aceptables.

28. Cache Invalidation

La invalidación es uno de los problemas centrales.

Debe soportarse conceptualmente:

Delete
Expire
Version Change
Event-Based Invalidation
Tag Invalidation
Namespace Invalidation
29. Explicit Invalidation

Cuando cambia el origen:

Update Entity
      ↓
Invalidate Cache

Ejemplo:

User Updated
    ↓
Invalidate user:{id}
30. Event-Based Invalidation

E13 puede colaborar:

Domain Event
    ↓
Cache Invalidation

Ejemplo:

UserUpdated
    ↓
Invalidate User Cache
31. Eventual Cache Invalidation

Si la invalidación ocurre de forma asíncrona:

Update
  ↓
Event
  ↓
Consumer
  ↓
Invalidate

puede existir una ventana temporal de stale data.

Debe aceptarse conscientemente.

32. Versioned Cache

Una estrategia alternativa:

user:v2:{id}

permite cambiar el namespace sin eliminar físicamente todas las entradas antiguas inmediatamente.

33. Namespace Versioning

Ejemplo:

product:v1:{id}
product:v2:{id}

Cuando se despliega una nueva representación:

v2 becomes active
34. Tag-Based Invalidation

Una entrada puede tener:

tags:
  user:123
  organization:456

y permitir invalidación por tag.

35. Bulk Invalidation

Puede ser necesario invalidar:

Tenant Cache
Organization Cache
Configuration Cache

Debe evitarse que estas operaciones provoquen una carga masiva inesperada sobre el origen.

36. Cache Stampede

Problema:

10,000 requests
       ↓
Same key expired
       ↓
10,000 source requests

Esto puede sobrecargar el sistema.

37. Request Coalescing

Una estrategia:

Request A ─┐
Request B ─┤
Request C ─┼──> One Source Request
Request D ─┘

El resultado se comparte entre los solicitantes.

38. Single Flight

Por cada key:

Key X
  ↓
One in-flight load

Los demás consumidores esperan el mismo resultado.

39. Cache Warmup

Algunas entradas pueden precargarse:

Application Startup
      ↓
Warmup
      ↓
Cache

Debe utilizarse sólo cuando el coste de warmup sea razonable.

40. Lazy Population

Alternativamente:

Cache Empty
    ↓
First Request
    ↓
Load
    ↓
Populate

Es el patrón natural de cache-aside.

41. Negative Caching

Puede cachearse también la ausencia:

Key
 ↓
NOT_FOUND

Esto evita repetir consultas costosas para recursos inexistentes.

Debe tener TTL corto para evitar ocultar nuevos datos.

42. Null Values

Debe distinguirse:

Cache Miss

de:

Cached Not Found

Son estados diferentes.

43. Serialization

Los valores cacheados deben tener una representación definida:

JSON
Binary
MessagePack
Protocol Buffers

según las necesidades del sistema.

44. Cache Schema

El formato de los valores debe poder versionarse.

Ejemplo:

cache_schema_version = 3

Esto permite evolucionar estructuras sin asumir compatibilidad infinita.

45. Serialization Compatibility

Durante deployments graduales puede existir:

Application v1
Application v2

simultáneamente.

El cache debe soportar la estrategia de compatibilidad correspondiente.

46. Compression

Los valores grandes pueden comprimirse.

Pero la compresión debe evaluarse contra:

CPU Cost
Memory Savings
Latency
Payload Size

No debe activarse indiscriminadamente.

47. Memory Management

Los caches en memoria deben tener límites.

Debe existir:

Maximum Size
Maximum Entry Size
Eviction Policy
48. Eviction Policies

Posibles políticas:

LRU
LFU
FIFO
TTL-Based
Size-Based
Adaptive

La elección depende del patrón de acceso.

49. LRU

Least Recently Used:

Oldest unused entries
        ↓
Evicted first

Adecuado para muchos patrones de acceso generales.

50. LFU

Least Frequently Used:

Least accessed entries
        ↓
Evicted first

Puede ser útil cuando existen valores con alta reutilización.

51. Cache Capacity

Debe evitarse:

Unbounded Cache

porque puede provocar:

Memory Exhaustion
OOM
Process Restart
52. Entry Size Limit

Una única entrada excesivamente grande puede ser peligrosa.

Debe existir un límite razonable:

max_entry_size
53. Cache Priority

No todas las entradas tienen la misma importancia.

Puede definirse:

HIGH
NORMAL
LOW

si la implementación soporta eviction priorizada.

54. Sensitive Data

No debe cachearse información sensible sin evaluar:

Encryption
Access Control
TTL
Isolation
Data Residency
Memory Exposure
55. Authentication Data

Tokens, sesiones o credenciales requieren tratamiento especial.

Un cache compartido no debe permitir:

Tenant A
   ↓
Credential
   ↓
Tenant B
56. Cache Encryption

Los valores especialmente sensibles pueden requerir cifrado en reposo.

Pero cifrar todos los caches puede aumentar:

CPU
Latency
Complexity

La decisión debe basarse en clasificación de datos.

57. Cache and Authorization

Nunca debe asumirse:

Cached Object
    =
Authorized Object

La autorización sigue siendo responsabilidad de la capa correspondiente.

58. Authorization-Aware Keys

Cuando el resultado depende de identidad o permisos:

user:{user_id}:resource:{id}

o una representación equivalente debe reflejar ese scope.

59. Security Boundary

El cache no debe convertirse en un bypass de autorización.

Request
   ↓
Authorization
   ↓
Cache Access

cuando el recurso cacheado sea sensible.

60. Repository Integration

E10 puede utilizar caching:

Application Service
       ↓
Repository
       ↓
Cache
       ↓
Database

pero el Repository debe mantener una semántica clara respecto a qué datos cachea.

61. Domain Service Integration

E08 puede consumir datos cacheados, pero el cache no debe introducir decisiones de dominio.

Domain Logic
     │
     ▼
Data Access
     │
     ▼
Cache
62. Application Service Integration

E09 puede utilizar caching para:

Computed Results
Read Models
Reference Data
Expensive Queries
63. API Integration

E03 puede aplicar response caching cuando sea seguro.

HTTP Request
     ↓
API Cache
     ↓
Response

Debe respetar:

Authorization
Tenant
User Scope
Freshness
Privacy
64. Event Integration

E13 puede producir:

EntityChanged
ConfigurationChanged
PermissionChanged

que desencadenen invalidaciones.

65. Messaging Integration

E12 puede transportar eventos de invalidación cuando el sistema requiera propagación distribuida.

66. Configuration Cache

E18 será responsable de configuración, pero E17 puede proporcionar caching de configuraciones.

Separación:

E18
Configuration Authority
       │
       ▼
E17
Configuration Cache
67. Cache Dependency Graph
Application
    │
    ▼
Cache
    │
    ▼
Repository / Service
    │
    ▼
Source of Truth

La dependencia debe fluir en esa dirección.

68. Cache Failure Modes

Debe contemplarse:

Cache Unavailable
Cache Timeout
Cache Corruption
Serialization Failure
Memory Pressure
Network Partition
Stale Data
Eviction Storm
Stampede
69. Cache Timeout

Una operación de cache no debería bloquear indefinidamente una request.

Debe existir:

Timeout
Fallback
Circuit Breaker

cuando corresponda.

70. Fail-Open vs Fail-Closed

Dependiendo del tipo de dato:

Datos no críticos

Puede hacerse:

Cache failure
   ↓
Source lookup
Datos de seguridad

Puede ser preferible:

Cache failure
   ↓
Fail closed

La política debe definirse por caso.

71. Circuit Breaker

Si el distributed cache falla repetidamente:

Application
   ↓
Cache
   ↓
Failures
   ↓
Circuit Open

puede evitarse que cada request espere un timeout.

72. Cache Availability

La disponibilidad del cache debe medirse separadamente de la disponibilidad de la aplicación.

73. Cache Metrics

Métricas mínimas:

cache_requests_total
cache_hits_total
cache_misses_total
cache_errors_total
cache_evictions_total
cache_expirations_total
cache_set_total
cache_delete_total
cache_latency
cache_hit_ratio
74. Memory Metrics

Para caches locales:

cache_memory_used
cache_memory_limit
cache_entry_count
cache_average_entry_size
75. Distributed Cache Metrics

También:

network_latency
connection_errors
timeouts
reconnects
cluster_health
replication_lag

cuando aplique.

76. Cache Observability

Debe poder responderse:

Why was this value returned?
Was it cached?
Which cache layer?
When was it created?
When does it expire?
What source produced it?

sin exponer información sensible.

77. Cache Trace

Un trace puede mostrar:

Request
 ↓
L1 MISS
 ↓
L2 HIT
 ↓
Response

o:

Request
 ↓
L1 MISS
 ↓
L2 MISS
 ↓
Database
 ↓
L2 SET
 ↓
L1 SET
 ↓
Response
78. Cache Key Privacy

Las claves no deben incluir innecesariamente:

Passwords
Tokens
PII
Secrets

porque pueden aparecer en logs o métricas.

79. Cache Logging

Nunca debe registrarse el valor completo si contiene información sensible.

Preferible:

key_hash
entry_size
hit/miss
latency
80. Cache Invalidation on Deployment

Un deployment puede cambiar:

Schema
Serialization
Business Semantics

Puede ser necesario:

Versioned Namespace
Selective Flush
Warmup

en lugar de un flush global.

81. Cache Invalidation on Data Migration

Después de migraciones:

Migration
   ↓
Invalidate affected cache

Debe formar parte del plan de deployment cuando corresponda.

82. Cache Stampede Protection

Además de single-flight pueden utilizarse:

Jittered TTL
Probabilistic Early Refresh
Locking
Background Refresh
83. Jittered TTL

En lugar de:

TTL = exactly 60m

puede utilizarse una variación controlada:

TTL = 55–65m

para evitar expiración sincronizada.

84. Early Refresh

Una entrada próxima a expirar puede renovarse antes:

Entry
   ↓
Near Expiration
   ↓
Background Refresh

reduciendo misses visibles.

85. Stale-While-Revalidate

Puede permitirse:

Request
   ↓
Stale Value
   ↓
Return immediately
   ↓
Background Refresh

cuando la aplicación tolere stale data.

86. Cache Priority Classes

EVOXA puede clasificar caches:

Critical
Important
Best-Effort

Esto ayuda a definir qué ocurre durante presión de memoria o fallos.

87. Cache Ownership

Cada cache debe tener:

Owner
Purpose
Source of Truth
TTL
Invalidation Strategy
Security Classification
Capacity
88. Cache Contract

Cada cache debería documentar:

Key Format
Value Schema
TTL
Freshness Guarantee
Invalidation
Failure Behavior
Scope
Owner
89. Example Contract
Cache: User Profile

Source:
User Repository

Key:
tenant:{tenant_id}:user:{user_id}

TTL:
5 minutes

Invalidation:
UserUpdated event

Consistency:
Eventual

Failure:
Fallback to repository

Owner:
Identity Domain
90. Cache Lifecycle
CREATE
  ↓
POPULATE
  ↓
ACTIVE
  ↓
REFRESH
  ↓
EXPIRE / INVALIDATE
  ↓
EVICT
91. Cache Refresh

Puede ser:

On Demand
Scheduled
Event Driven
Background
92. Scheduled Refresh

E16 puede colaborar:

E16 Schedule
      ↓
Cache Refresh Job
      ↓
E15
      ↓
Cache

E16 decide cuándo; E15 ejecuta el refresh.

93. Event-Driven Refresh
EntityChanged
      ↓
Refresh Cache

Puede ser preferible a esperar el TTL cuando la frescura sea importante.

94. Cache and Scheduling

La relación correcta:

E16
 ↓
Refresh Job
 ↓
E15
 ↓
E17

y no:

E17
 ↓
Own Scheduler

Cada módulo conserva su responsabilidad.

95. Cache and Workflow

Un Workflow puede leer información cacheada, pero:

Workflow State

debe permanecer durable en su propio mecanismo de persistencia.

El cache no sustituye al Workflow State Store.

96. Cache and Events

Los eventos pueden invalidar caches:

EntityChanged
      ↓
Cache Invalidation

pero el evento no garantiza por sí solo que el cache esté actualizado inmediatamente.

97. Cache and Jobs

Un Job puede:

Read Cache
Refresh Cache
Invalidate Cache
Warm Cache

según el contrato del Job.

98. Distributed Consistency

En múltiples nodos:

Node A
  ↓
Cache

Node B
  ↓
Cache

puede existir inconsistencia temporal.

Debe definirse si el sistema requiere:

Local Consistency
Cluster Consistency
Eventual Consistency
99. Regional Cache

En multi-region:

Region A
  └── Cache A

Region B
  └── Cache B

pueden existir datos distintos temporalmente.

La arquitectura debe definir si existe:

Global Cache
Regional Cache
Replication
Independent Caches
100. Data Residency

Los datos cacheados pueden estar sujetos a requisitos de residencia.

Debe conocerse:

Where is cached data stored?
Who can access it?
How long?
101. Cache Purging

Debe existir capacidad de eliminar datos por:

Key
Tenant
Entity
Tag
Namespace
Region

especialmente para:

Privacy
Security
Compliance
Incident Response
102. Right to Delete

Si un dato debe eliminarse:

Source
 ↓
Cache
 ↓
Search Index
 ↓
Derived Stores

deben considerarse todos los lugares donde pueda existir.

103. Cache and Compliance

No debe asumirse que:

TTL

equivale a eliminación garantizada.

Para requisitos estrictos puede requerirse invalidación explícita.

104. Cache Disaster Recovery

El cache normalmente puede reconstruirse desde el Source of Truth.

Por tanto:

Cache Loss
   ↓
Rebuild

debe ser una operación soportada.

105. Cache Persistence

Persistir el cache puede ser útil para acelerar recuperación, pero:

No convierte al cache en Source of Truth.

La autoridad continúa perteneciendo al almacenamiento correspondiente.

106. Cache Warm Restart

Después de un restart:

Application Start
      ↓
Cache Empty
      ↓
Traffic
      ↓
Lazy Population

o:

Application Start
      ↓
Warmup
      ↓
Ready

según el caso.

107. Read Model Cache

Puede cachearse un read model:

Query
 ↓
Read Model Cache
 ↓
Response

especialmente para consultas frecuentes.

108. Computation Cache

También pueden cachearse resultados costosos:

Input
  ↓
Expensive Computation
  ↓
Result

La key debe incluir todos los inputs relevantes.

109. Computation Cache Key

Si:

f(a,b,c)

entonces la key debe representar:

f:a:b:c

o una forma equivalente.

Omitir un input puede producir resultados incorrectos.

110. Reference Data Cache

Datos relativamente estáticos pueden beneficiarse especialmente:

Countries
Currencies
Feature Metadata
Static Rules
Reference Tables
111. Cache of External APIs

Las llamadas externas pueden cachearse:

Application
   ↓
Cache
   ↓ miss
External API

pero deben considerarse:

Provider Terms
Freshness
Rate Limits
Privacy
Errors
112. Error Caching

Puede ser útil cachear ciertos errores temporalmente:

External Service
   ↓
Unavailable
   ↓
Short Negative Cache

para evitar hammering.

Debe tener TTL corto y una estrategia de recuperación.

113. Cache Penetration

Una gran cantidad de requests para claves inexistentes puede saturar el origen.

Mitigaciones:

Negative Cache
Bloom Filter
Rate Limiting
Validation

cuando sea apropiado.

114. Cache Breakdown

Si una entrada muy popular expira:

Hot Key
 ↓
Expire
 ↓
Huge Load

debe utilizarse:

Single Flight
Lock
Early Refresh
Stale-While-Revalidate

según el caso.

115. Hot Keys

Debe detectarse:

Top Accessed Keys

porque una única clave puede concentrar una proporción significativa del tráfico.

116. Hot Key Mitigation

Opciones:

Replication
Local L1 Cache
Refresh
Request Coalescing
117. Cache Fairness

El cache no debe permitir que unas pocas claves o tenants consuman toda la capacidad.

Puede ser necesario aplicar:

Per-Tenant Quotas
Per-Key Limits
Admission Policies
118. Cache Admission

No todos los datos deben entrar automáticamente al cache.

Puede aplicarse una política:

Frequently Read?
Expensive Source?
Small Enough?
Safe to Cache?
119. Cache Bypass

Debe existir una forma de evitar el cache cuando sea necesario:

Force Refresh
Administrative Bypass
Consistency-Critical Read
Debug Mode

con controles apropiados.

120. Canonical Architecture
                              EVOXA
                                │
                                ▼
                         Application Layer
                                │
                                ▼
                         Cache Abstraction
                                │
                   ┌────────────┴────────────┐
                   ▼                         ▼
              L1 Local                  L2 Distributed
                   │                         │
                   └────────────┬────────────┘
                                │
                              MISS
                                │
                                ▼
                         Source of Truth
                                │
             ┌──────────────────┼──────────────────┐
             ▼                  ▼                  ▼
          Database          Domain Service      External API
             │                  │                  │
             └──────────────────┼──────────────────┘
                                │
                                ▼
                           Cache Populate
121. Integración con la Arquitectura EVOXA
E03 — API
        │
        ▼
E09 — Application Services
        │
        ▼
E10 — Repository ─────────┐
        │                 │
        ▼                 ▼
     Database          E17 Cache
                          ▲
                          │
E13 — Events ─────────────┘
        │
        ▼
 Cache Invalidation

E16 — Scheduling
        │
        ▼
E15 — Jobs
        │
        ▼
Cache Refresh / Warmup
122. Architectural Rules
Rule 1

Cache nunca sustituye al Source of Truth.

Rule 2

Todo cache debe tener un owner.

Rule 3

Todo cache debe definir TTL o una política equivalente de expiración/invalidation.

Rule 4

Las cache keys deben incluir el scope necesario, especialmente tenant.

Rule 5

Cache miss debe ser un comportamiento esperado.

Rule 6

Cache failure debe tener una estrategia explícita.

Rule 7

Datos sensibles requieren clasificación antes de cachearse.

Rule 8

La autorización no debe depender accidentalmente de un cache.

Rule 9

Los caches distribuidos deben soportar concurrencia y duplicación.

Rule 10

Debe existir protección contra cache stampede.

Rule 11

Las estructuras cacheadas deben ser versionables.

Rule 12

Las invalidaciones deben ser observables.

Rule 13

Un cache debe poder reconstruirse desde su Source of Truth.

Rule 14

E17 proporciona caching; no debe convertirse en propietario de scheduling, workflows o jobs.

Rule 15

Las políticas de freshness deben estar definidas por caso de uso.

123. Definition of Done

E17 queda definido cuando EVOXA dispone de:

✓ Cache Abstraction
✓ Local Cache
✓ Distributed Cache
✓ Multi-Level Cache
✓ Cache Entry Model
✓ Cache Key Strategy
✓ Namespacing
✓ Tenant Isolation
✓ Cache Scopes
✓ Cache Hit
✓ Cache Miss
✓ Hit Ratio
✓ TTL
✓ Absolute Expiration
✓ Sliding Expiration
✓ Stale Data Policy
✓ Cache Consistency
✓ Cache-Aside
✓ Read-Through
✓ Write-Through
✓ Write-Behind
✓ Explicit Invalidation
✓ Event-Based Invalidation
✓ Versioned Cache
✓ Namespace Versioning
✓ Tag Invalidation
✓ Bulk Invalidation
✓ Cache Stampede Protection
✓ Request Coalescing
✓ Single Flight
✓ Cache Warmup
✓ Lazy Population
✓ Negative Caching
✓ Null Handling
✓ Serialization
✓ Schema Versioning
✓ Compression Strategy
✓ Memory Limits
✓ Entry Size Limits
✓ Eviction Policies
✓ Cache Priority
✓ Sensitive Data Handling
✓ Authorization-Aware Caching
✓ Repository Integration
✓ API Integration
✓ Event Integration
✓ Messaging Integration
✓ Configuration Cache
✓ Cache Failure Handling
✓ Timeout Handling
✓ Fail-Open / Fail-Closed Policy
✓ Circuit Breaker Compatibility
✓ Cache Metrics
✓ Memory Metrics
✓ Distributed Cache Metrics
✓ Cache Tracing
✓ Privacy-Safe Logging
✓ Deployment Invalidation
✓ Migration Invalidation
✓ Jittered TTL
✓ Early Refresh
✓ Stale-While-Revalidate
✓ Cache Ownership
✓ Cache Contracts
✓ Cache Lifecycle
✓ Scheduled Refresh
✓ Event-Driven Refresh
✓ Distributed Consistency
✓ Regional Cache Strategy
✓ Data Residency
✓ Cache Purging
✓ Compliance Handling
✓ Disaster Recovery
✓ Cache Rebuild
✓ Read Model Cache
✓ Computation Cache
✓ Reference Data Cache
✓ External API Cache
✓ Error Caching
✓ Cache Penetration Protection
✓ Cache Breakdown Protection
✓ Hot Key Detection
✓ Hot Key Mitigation
✓ Cache Fairness
✓ Cache Admission
✓ Cache Bypass
✓ Observability
✓ Security
✓ E15 Integration
✓ E16 Integration
124. Engineering Specification Progress
E01 — EVOXA Backend Architecture
E02 — EVOXA Database Architecture
E03 — EVOXA API Architecture
E04 — EVOXA Authentication Architecture
E05 — Authorization Architecture
E06 — Policy Architecture
E07 — EVOXA Service Architecture
E08 — Domain Services Architecture
E09 — Application Services Architecture
E10 — Repository Architecture
E11 — Integration Architecture
E12 — Messaging Architecture
E13 — Event Processing Architecture
E14 — Workflow & Orchestration Architecture
E15 — Job & Task Processing Architecture
E16 — Scheduling Architecture
E17 — Caching Architecture

La cadena queda ahora:

                    E14 Workflow
                         │
                    E16 Scheduling
                         │
                         ▼
                    E15 Jobs
                         │
                         ▼
                     Workers
                         │
                         ▼
                Application Services
                         │
                         ▼
                    E17 Cache
                         │
              ┌──────────┴──────────┐
              ▼                     ▼
         E10 Repository        External Services
              │
              ▼
           Database

Y la regla central de esta pieza es:

E17 acelera el acceso a información, pero nunca sustituye la autoridad de los datos.

Siguiente capítulo: E18 — EVOXA Configuration Architecture.
