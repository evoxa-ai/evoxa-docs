ADR-IDENTITY-019 — Event Partitioning, Ordering & Routing

Estado: Proposed
Versión: 1.0
Fecha: 2026-09-02
Dominio: Identity / Event Platform
Prioridad: Critical
Tipo: Event Architecture / Scalability / Consistency / Routing / Performance

1. Título

ADR-IDENTITY-019 — Event Partitioning, Ordering & Routing

2. Contexto

EVOXA adopta una arquitectura Event-Driven y una Event Platform centralizada.

En ADR-IDENTITY-018 — Event Broker Strategy se estableció:

PostgreSQL
    ↓
Transactional Outbox
    ↓
Outbox Relay
    ↓
Event Broker
    ↓
Consumer Groups
    ↓
Consumers

También se estableció que:

la entrega es at-least-once;
los consumidores deben ser idempotentes;
no existe orden global de eventos;
los eventos deben ser inmutables y versionados;
el sistema debe soportar retries;
debe existir DLQ;
debe existir replay;
debe existir tenant isolation;
debe existir observabilidad mediante event_id, correlation_id, causation_id y trace_id.

La arquitectura de eventos de EVOXA establece además que el orden debe garantizarse únicamente donde sea necesario, por ejemplo dentro de un aggregate, usuario o workflow, evitando imponer una serialización global que limite la escalabilidad.

Por lo tanto, es necesario definir cómo se distribuirán los eventos entre:

topics/streams;
partitions;
producers;
consumers;
consumer groups.
3. Problema

Si todos los eventos fueran procesados en una única secuencia:

Event 1
   ↓
Event 2
   ↓
Event 3
   ↓
...

se produciría un cuello de botella importante.

Por otro lado, si todos los eventos fueran procesados completamente en paralelo:

Event A ─┐
Event B ─┤
Event C ─┤
Event D ─┘

podrían producirse inconsistencias cuando el orden sea significativo.

Por ejemplo:

UserCreated
UserActivated
UserSuspended

no debería necesariamente procesarse como:

UserSuspended
UserCreated
UserActivated

para el mismo usuario.

Necesitamos, por tanto, un modelo que consiga:

Orden donde sea necesario + paralelismo donde sea posible.

4. Decisión

EVOXA adoptará una estrategia de:

Partitioned Event Streaming with Deterministic Routing and Scoped Ordering

Es decir:

los eventos serán distribuidos mediante partitions;
cada evento tendrá una routing key cuando el orden sea relevante;
eventos con la misma routing key deberán dirigirse a la misma partition mientras esa estrategia sea válida;
el orden será garantizado solamente dentro del ámbito definido;
no existirá orden global;
diferentes routing keys podrán procesarse en paralelo;
los consumidores trabajarán mediante consumer groups;
la routing strategy deberá preservar tenant isolation.

Conceptualmente:

                         Event Broker
                              │
                    ┌─────────┼─────────┐
                    │         │         │
                Partition  Partition  Partition
                    0          1          2
                    │          │          │
                 User A     User B     User C
                 User D     User E     User F

De esta forma:

User A events
    ↓
Partition X
    ↓
Ordered processing

mientras:

User B events
    ↓
Partition Y
    ↓
Parallel processing
5. Principio fundamental

La unidad de ordenamiento no será necesariamente el tenant completo.

Esto es importante.

Si se utilizara:

tenant_id

como única routing key:

Tenant A
   ↓
Partition 1
   ↓
todos sus eventos

se podría terminar serializando todas las operaciones de una organización.

En cambio, cuando el negocio lo permita, se utilizará una clave más granular.

Ejemplo:

tenant_id + aggregate_id

o una clave equivalente derivada del dominio.

6. Routing Key

Cada evento podrá tener una routing key cuando sea necesario garantizar orden.

Conceptualmente:

Event
├── tenant_id
├── aggregate_type
├── aggregate_id
└── routing_key

Ejemplo:

UserCreated
tenant_id = T1
aggregate_type = User
aggregate_id = U123
routing_key = T1:User:U123

Los eventos posteriores del mismo aggregate utilizarían la misma clave:

UserCreated
UserActivated
UserSuspended

y serían dirigidos a la misma partition según el algoritmo de routing del broker.

7. Scope de Ordering

El orden será definido por contexto.

Los scopes posibles incluyen:

Aggregate
User U1
Event 1
Event 2
Event 3
Session
Session S1
Event 1
Event 2
Membership
Membership M1
Event 1
Event 2
Workflow

Cuando varios eventos pertenecen a un mismo workflow y deben conservar una secuencia.

El scope definitivo deberá definirse en el catálogo de eventos.

8. No Global Ordering

EVOXA no requerirá:

Event 1
Event 2
Event 3
Event 4

como orden global absoluto.

Esto permitiría:

Partition 1
A1 → A2 → A3

Partition 2
B1 → B2 → B3

Partition 3
C1 → C2 → C3

procesándose simultáneamente.

Esto permite escalar horizontalmente.

9. Ejemplo de Identity

Supongamos:

User A
User B
User C

Los eventos pueden distribuirse:

Partition 0
User A:
  UserCreated
  UserActivated
  UserSuspended

Partition 1
User B:
  UserCreated
  UserActivated

Partition 2
User C:
  UserCreated
  UserActivated

Se preserva el orden por usuario sin serializar toda la plataforma.

10. Tenant Isolation

La routing strategy deberá considerar el tenant.

Para eventos tenant-scoped, la clave deberá incluir suficiente información para evitar ambigüedades.

Conceptualmente:

tenant_id + aggregate_id

en lugar de:

aggregate_id

cuando el identificador no sea globalmente único.

Ejemplo:

Tenant A + User 123
Tenant B + User 123

deben poder distinguirse.

11. System Events

No todos los eventos tienen tenant_id.

Podemos tener:

SYSTEM
PLATFORM

events.

Estos deberán clasificarse explícitamente y tener una routing strategy apropiada.

No se deberá inventar un tenant ficticio simplemente para satisfacer el algoritmo de routing.

12. Event Categories

Los eventos podrán organizarse conceptualmente por dominio y categoría.

Ejemplo:

identity.user.*
identity.organization.*
identity.membership.*
identity.session.*
identity.authentication.*
identity.authorization.*
identity.mfa.*
identity.security.*
identity.policy.*

Estos nombres son conceptuales y deberán consolidarse en el catálogo de eventos.

13. Topic / Stream Strategy

La decisión de particionamiento estará relacionada con la topología de topics/streams.

Se evitará:

topic per individual user

porque produciría una explosión de recursos.

También se evitará necesariamente:

single topic for everything

si esto mezcla eventos con:

diferentes niveles de seguridad;
diferentes patrones de consumo;
diferentes necesidades de retención;
diferentes necesidades de escalabilidad.

La separación deberá basarse en características reales de los eventos.

14. Partition Count

El número de partitions no se fijará arbitrariamente en este ADR.

Deberá determinarse mediante:

throughput;
consumer parallelism;
ordering requirements;
storage;
broker limits;
expected growth;
benchmark.

Conceptualmente:

Required partitions
       ↓
Capacity
       +
Ordering
       +
Parallelism
       +
Growth
15. Partition Count and Ordering

La elección del número de partitions tiene consecuencias sobre routing y escalabilidad.

Si se utiliza una estrategia hash:

routing_key
    ↓
hash
    ↓
partition

los eventos con la misma key continuarán dirigiéndose a la misma partition mientras la estrategia de particionamiento permanezca compatible.

Por ello, cambios en el número de partitions deberán analizarse cuidadosamente.

16. Partition Expansion

Incrementar el número de partitions puede modificar la asignación de routing keys dependiendo de la tecnología y algoritmo utilizados.

Por ejemplo:

hash(key) % N

con:

N = 4

no necesariamente produce el mismo resultado que:

N = 8

Por lo tanto:

La expansión de partitions debe considerarse una operación arquitectónica que puede afectar ordering.

La estrategia exacta deberá definirse una vez seleccionado el broker.

17. Ordering Guarantees

La garantía será:

Orden por routing key dentro del alcance soportado por el broker y consumer model.

No se garantizará:

orden global;
orden entre tenants;
orden entre aggregates diferentes;
orden entre topics diferentes;

salvo que un caso de negocio lo requiera explícitamente.

18. Event Sequence

Cuando un caso requiera detectar gaps de secuencia, podrá contemplarse metadata como:

aggregate_version

o un mecanismo equivalente.

Ejemplo:

User U1

version 1 → Created
version 2 → Activated
version 3 → Suspended

El consumer puede detectar:

received version 3
expected version 2

y actuar según la estrategia correspondiente.

Esto no convierte el broker en fuente de verdad; únicamente permite detectar inconsistencias de procesamiento.

19. Event Version vs Aggregate Version

Debe distinguirse:

event_version

de:

aggregate_version
Event version

Define la versión del contrato/event schema.

UserCreated v1
UserCreated v2
Aggregate version

Representa la evolución del aggregate.

User aggregate
version 1
version 2
version 3

No deben confundirse.

20. Consumer Groups

Los consumidores se organizarán mediante grupos según su necesidad funcional.

Ejemplo:

identity.user.events
        │
        ├── analytics-group
        ├── notification-group
        ├── search-group
        └── ai-context-group

Cada consumer group podrá procesar los eventos independientemente.

Dentro de un mismo grupo, las partitions podrán distribuirse entre instancias.

21. Consumer Scaling

Ejemplo:

6 partitions
     ↓
3 consumer instances

puede permitir aproximadamente:

Consumer 1 → P0, P1
Consumer 2 → P2, P3
Consumer 3 → P4, P5

La distribución exacta depende del broker.

Esto permite escalar consumidores horizontalmente.

22. Consumer Count vs Partitions

El número de consumidores activos dentro de un consumer group no puede aprovechar indefinidamente más paralelismo que las partitions disponibles.

Conceptualmente:

Partitions = 6
Consumers  = 3

permite paralelismo.

Pero:

Partitions = 3
Consumers  = 10

no significa que existan 10 unidades de procesamiento paralelo dentro del mismo grupo.

Por ello, capacity planning y partition planning deben diseñarse conjuntamente.

23. Hot Partitions

Una mala routing key puede generar:

Partition 0
████████████████████
Partition 1
██
Partition 2
█
Partition 3
█

Esto ocurre cuando demasiados eventos comparten la misma key.

Ejemplo peligroso:

tenant_id

como única key para un tenant extremadamente activo.

Por ello, la routing strategy deberá evitar hot partitions cuando el dominio permita mayor granularidad.

24. Identity Hotspot Example

Una organización grande podría generar:

Tenant A
  100,000 events/sec

Si todos utilizan:

routing_key = TenantA

podríamos terminar con:

Tenant A
    ↓
Partition X
    ↓
Bottleneck

Una estrategia más granular podría utilizar:

Tenant A + Aggregate ID

permitiendo distribuir los eventos del tenant.

25. Routing Priority

La routing strategy seguirá aproximadamente:

1. Preserve required ordering
2. Preserve tenant context
3. Maximize safe parallelism
4. Avoid hot partitions
5. Maintain predictable routing

La consistencia necesaria tendrá prioridad sobre el paralelismo.

26. Cross-Aggregate Ordering

No se garantizará automáticamente el orden entre aggregates.

Ejemplo:

UserCreated
MembershipCreated

pueden pertenecer a aggregates distintos.

Si un consumer necesita una relación causal:

UserCreated
     ↓
MembershipCreated

deberá utilizar:

causation_id;
dependencia explícita;
workflow;
state validation;
retry;
buffering;

según el caso.

No se deberá depender de un orden global accidental.

27. Causation ID

causation_id permitirá expresar relaciones causales.

Ejemplo:

UserCreated
event_id = E1

MembershipCreated
causation_id = E1

Esto permite al consumidor entender:

MembershipCreated
      ↓
caused by
      ↓
UserCreated

sin requerir orden global.

28. Correlation ID

correlation_id permitirá agrupar eventos pertenecientes a una misma operación o workflow.

Ejemplo:

HTTP Request
     ↓
User Registration
     │
     ├── UserCreated
     ├── MembershipCreated
     └── SessionCreated

Todos pueden compartir:

correlation_id = C1
29. Trace ID

trace_id permitirá correlacionar:

API
 ↓
Database
 ↓
Outbox
 ↓
Broker
 ↓
Consumer

con la observabilidad distribuida.

Esto será especialmente útil para debugging de eventos fuera de orden o retrasados.

30. Retry and Ordering

Los retries pueden interactuar con ordering.

Ejemplo:

Event 1 → failed
Event 2 → available

No necesariamente se debe permitir que:

Event 2

sea procesado antes que:

Event 1

si el workflow exige orden.

La política final deberá considerar:

broker semantics;
consumer semantics;
retry strategy;
ordering requirements.
31. DLQ and Ordering

Enviar un evento a DLQ puede romper la secuencia lógica.

Ejemplo:

Event 1 → DLQ
Event 2 → processed
Event 3 → processed

Si Event 1 era obligatorio para comprender los siguientes eventos, el consumer deberá disponer de una estrategia apropiada.

Opciones conceptuales:

detener procesamiento de esa key;
marcar gap;
esperar;
procesar parcialmente;
enviar dependientes a retry;
replay.

La estrategia específica se definirá en el gobierno de eventos y replay.

32. Replay

El replay deberá respetar las reglas de ordering necesarias.

No se deberá hacer:

Replay random events

si existe una dependencia secuencial.

Preferido:

Aggregate
   ↓
Ordered events
   ↓
Replay

cuando el caso lo requiera.

33. Routing y Replay

Un replay debe conservar suficiente metadata original para reproducir correctamente:

tenant;
aggregate;
routing key;
event type;
event version;
timestamps relevantes;
correlation;
causation.

No se deberá recalcular arbitrariamente una routing key histórica si esto puede modificar el comportamiento esperado.

34. Security Routing

La routing strategy no debe exponer información sensible.

No se deberá utilizar como routing key:

email
phone
password
health information

ni datos PII innecesarios.

Preferido:

opaque identifier

como:

tenant_id
aggregate_id

cuando corresponda.

35. Tenant Routing

Para eventos tenant-scoped:

tenant_id

debe estar presente en el envelope según la arquitectura.

Pero no necesariamente debe ser la única routing key.

Por ejemplo:

routing_key =
tenant_id + ":" + aggregate_id

puede proporcionar:

tenant awareness;
aggregate ordering;
paralelismo.
36. System Routing

Para eventos de plataforma:

tenant_id = null
scope = SYSTEM

o el equivalente definido por el modelo.

La routing key deberá basarse en el recurso o workflow apropiado.

No deberá mezclarse arbitrariamente información de tenants.

37. Event Type Routing

No todos los consumidores necesitan todos los eventos.

Por ejemplo:

Security Consumer

puede necesitar:

identity.authentication.*
identity.authorization.*
identity.security.*
identity.mfa.*

mientras que:

Analytics Consumer

puede necesitar una selección distinta.

El routing a nivel de topic/stream debe reducir consumo innecesario.

38. Routing Layers

La estrategia puede tener varias capas:

Event Type
    ↓
Topic / Stream
    ↓
Partition
    ↓
Consumer Group
    ↓
Consumer Instance

Cada capa tiene una responsabilidad diferente.

Event Type

¿Qué ocurrió?

Topic/Stream

¿Dónde se transporta?

Partition

¿Dónde se procesa en paralelo/secuencialmente?

Consumer Group

¿Quién necesita procesarlo?

Consumer

¿Quién ejecuta el procesamiento?

39. Event Broker Independence

Este ADR no fija el algoritmo específico de partitioning.

Podrá utilizarse:

hash partitioning;
key-based partitioning;
broker-native partitioning;
estrategia equivalente.

La elección depende del broker seleccionado en ADR-018.

40. Performance

La estrategia debe permitir:

procesamiento paralelo;
escalamiento horizontal;
baja latencia;
evitar hot partitions.

Los objetivos generales de Event Architecture deben mantenerse, incluyendo los objetivos previamente definidos para:

publicación;
delivery;
procesamiento;
replay.

El cumplimiento deberá verificarse mediante benchmarks.

41. Capacity Planning

El diseño deberá considerar:

Event rate
     +
Event size
     +
Partitions
     +
Consumer rate
     +
Retention
     +
Growth

Una estimación conceptual:

Required throughput
        ↓
Partition capacity
        ↓
Number of partitions
        ↓
Consumer capacity

No se fijarán cifras arbitrarias en este ADR.

42. Observability

Se deberán observar:

events por partition;
throughput;
partition utilization;
consumer lag;
hot partitions;
rebalance frequency;
processing latency;
ordering violations;
retry count;
DLQ;
replay.

Métricas conceptuales:

event_partition_throughput
event_partition_utilization
event_partition_hotspot
event_consumer_lag
event_consumer_rebalance
event_ordering_violation
event_routing_failures
event_replay_lag
43. Failure Handling
Routing failure

El evento no debe perderse.

Partition unavailable

Debe existir estrategia de recuperación proporcionada por el broker.

Consumer failure

El grupo debe poder reasignar procesamiento.

Rebalance

Debe minimizar interrupciones.

Ordering violation

Debe detectarse cuando el dominio requiera secuencia.

Hot partition

Debe detectarse mediante observabilidad y capacidad.

44. Rebalancing

Cuando consumidores se agregan o eliminan:

Consumer Group
      ↓
Rebalance
      ↓
Partitions reassigned

el sistema debe mantener las garantías de ordering definidas.

La estrategia exacta depende del broker.

45. Consecuencias positivas
Escalabilidad

Permite procesar miles o millones de eventos en paralelo según capacidad.

Ordering controlado

Se mantiene orden únicamente donde es necesario.

Menor contention

No existe una secuencia global.

Tenant isolation

El contexto de tenant permanece asociado al evento.

Flexibilidad

Los diferentes consumer groups pueden escalar independientemente.

Resiliencia

Los eventos pueden continuar disponibles durante fallos de consumidores.

46. Consecuencias negativas

La estrategia introduce complejidad en:

routing keys;
partition count;
rebalancing;
hot partitions;
ordering;
retries;
replay;
capacity planning.

También existe el riesgo de diseñar una routing key demasiado amplia y generar un cuello de botella.

47. Alternativas consideradas
Alternativa A — Orden global

Rechazada.

Reduce severamente el paralelismo.

Alternativa B — Una partition única

Rechazada.

No escala adecuadamente.

Alternativa C — Partition por tenant

Rechazada como estrategia general.

Puede producir hot partitions en tenants grandes.

Alternativa D — Partition aleatoria

Rechazada para eventos que requieren ordering.

No garantiza que eventos del mismo aggregate permanezcan juntos.

Alternativa E — Partition por aggregate/routing key

Seleccionada.

Proporciona:

ordering localizado;
paralelismo;
escalabilidad;
routing determinístico.
48. Non-Negotiables
No existe orden global obligatorio.
Ordering debe definirse explícitamente por caso de uso.
Los eventos que requieren orden deben compartir routing key.
Tenant context debe preservarse.
Tenant ID no será automáticamente la única routing key.
No se utilizarán datos sensibles como routing keys.
Se debe evitar hot partition.
El número de partitions debe basarse en capacidad y benchmark.
Consumer groups deben soportar escalamiento.
Ordering no debe depender de casualidad temporal.
event_version y aggregate_version son conceptos diferentes.
correlation_id y causation_id deben preservarse.
Replay debe respetar ordering cuando corresponda.
DLQ debe considerar posibles dependencias de orden.
Retries no deben romper silenciosamente workflows ordenados.
Partitioning no sustituye tenant isolation.
Routing no sustituye authorization.
La estrategia debe ser observable.
Los cambios de partitioning deben analizar su impacto sobre ordering.
La tecnología concreta del broker queda pendiente de ADR-018.
49. Traceability
Blueprint

Relacionado con:

Event Architecture;
Event Platform;
scalability;
ordering;
routing;
reliability;
tenant isolation;
observability;
replay.

La arquitectura de eventos establece explícitamente que:

el ordering no es global;
el orden puede garantizarse por aggregate/user/workflow;
los consumidores son idempotentes;
existe at-least-once delivery;
existe retry;
existe DLQ;
existe replay.
ESP

ESP-0001 — Identity Domain

Especialmente:

Domain Events;
Integration Events;
Security Events;
Outbox.
Implementation Stories
IS-IDENTITY-013 — Domain Events & Outbox
IS-IDENTITY-015 — Identity Persistence
IS-IDENTITY-016 — Identity Testing & Quality
IS-IDENTITY-017 — Identity Runtime & Deployment
50. ADR relacionados
ADR-IDENTITY-007 — Event Architecture
ADR-IDENTITY-013 — Data Retention
ADR-IDENTITY-016 — Concurrency & Transaction Strategy
ADR-IDENTITY-018 — Event Broker Strategy
ADR-IDENTITY-020 — Event Schema Registry & Compatibility
ADR-IDENTITY-021 — Event Replay & Governance
ADR-IDENTITY-022 — Consumer Idempotency
ADR-IDENTITY-028 — Observability & Operational Security
ADR-IDENTITY-030 — Autoscaling & Capacity
51. Dependencias

La arquitectura queda:

ADR-007
Event Architecture
       ↓
ADR-013
Transactional Outbox
       ↓
ADR-018
Event Broker
       ↓
ADR-019
Partitioning / Ordering / Routing
       ├──────────────┐
       ▼              ▼
ADR-020          ADR-022
Schema           Consumer
Compatibility    Idempotency
       │
       ▼
ADR-021
Replay / Governance
52. Decisiones pendientes

Quedan pendientes:

Routing Key definitiva

Determinar por cada tipo de evento:

aggregate;
user;
membership;
session;
workflow;
tenant;
combinación.
Partition Count

Definir mediante benchmark.

Broker Algorithm

Determinar:

hash;
key-based;
broker-native.
Topic Strategy

Determinar:

topic por dominio;
topic por categoría;
streams;
namespaces.
Rebalancing

Definir comportamiento exacto.

Hot Partition Mitigation

Definir mecanismos específicos.

Cross-Partition Ordering

Determinar si algún workflow requiere coordinación adicional.

53. Criterios de aceptación

ADR-IDENTITY-019 podrá pasar de Proposed a Accepted cuando exista:

 catálogo de eventos;
 routing key definida por evento;
 ordering scope definido;
 topic/stream topology;
 partition strategy;
 partition count inicial;
 consumer group strategy;
 hot partition strategy;
 rebalancing strategy;
 retry/order interaction;
 DLQ/order interaction;
 replay/order strategy;
 tenant routing strategy;
 system-event routing;
 capacity model;
 benchmark;
 ordering tests;
 load tests;
 failure tests;
 observability;
 operational runbook.
54. Siguiente ADR

Con ADR-018 y ADR-019 ya tenemos definida la arquitectura de transporte y distribución.

El siguiente punto crítico es evitar que diferentes versiones de eventos rompan a los consumidores.

Por eso, el siguiente paso lógico es:

ADR-IDENTITY-020 — Event Schema Registry & Compatibility

La siguiente cadena será:

ADR-018
Event Broker
      ↓
ADR-019
Partitioning / Ordering / Routing
      ↓
ADR-020
Schema Registry / Compatibility
      ↓
ADR-021
Replay / Governance
      ↓
ADR-022
Consumer Idempotency

En ADR-020 definiremos cómo EVOXA controlará la evolución de eventos, por ejemplo:

UserCreated v1
      ↓
UserCreated v2
      ↓
Consumer compatibility
      ↓
Schema validation
      ↓
Backward / Forward compatibility
      ↓
Breaking changes
      ↓
New event version

Estado final: Proposed.
