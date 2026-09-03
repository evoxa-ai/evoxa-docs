ADR-IDENTITY-022 — Consumer Idempotency & Deduplication

Estado: Proposed
Versión: 1.0
Fecha: 2026-09-03
Dominio: Identity / Event Platform
Prioridad: Critical
Tipo: Event Architecture / Reliability / Data Integrity / Distributed Systems

1. Título

ADR-IDENTITY-022 — Consumer Idempotency & Deduplication

2. Contexto

EVOXA utiliza una arquitectura Event-Driven en la que los eventos son hechos de negocio inmutables, versionados y desacoplados de sus consumidores.

El Blueprint establece explícitamente una semántica de entrega:

At-Least-Once
+
Retry
+
Dead Letter Queue
+
Replay

y establece que los consumidores deben tolerar entregas duplicadas.

La arquitectura también indica que los workflows críticos deben implementar deduplicación.

Esto significa que EVOXA no puede asumir exactamente-once delivery a nivel de infraestructura.

El sistema debe garantizar, en cambio:

Exactly-once business effect, cuando el workflow lo requiera, mediante idempotencia y deduplicación en los consumidores.

3. Problema

En un sistema distribuido pueden ocurrir situaciones como:

Event
  ↓
Consumer
  ↓
Process
  ↓
DB Commit
  ↓
Network Failure
  ↓
Consumer doesn't receive ACK
  ↓
Broker retries

Resultado:

Same Event
     ↓
Delivered twice

También puede ocurrir mediante replay:

Historical Event
       ↓
Replay
       ↓
Consumer
       ↓
Event already processed

Sin idempotencia:

UserCreated
    ↓
Create Record
    ↓
Create Record AGAIN

o:

PaymentCompleted
       ↓
Apply Payment
       ↓
Apply Payment AGAIN

o:

NotificationRequested
       ↓
Send Notification
       ↓
Send Notification AGAIN

Por ello:

La entrega At-Least-Once exige consumidores diseñados para procesamiento repetible y seguro.

4. Decisión

EVOXA adoptará un modelo de:

Consumer-Side Idempotency with Durable Deduplication

La responsabilidad principal de evitar efectos duplicados estará en el consumidor.

La arquitectura conceptual será:

Event Broker
     │
     ▼
Consumer
     │
     ▼
Extract Event ID
     │
     ▼
Idempotency Check
     │
 ┌───┴────────┐
 │            │
Already       New
Processed     Event
 │            │
 ▼            ▼
Skip       Process
              │
              ▼
        Persist Outcome

La infraestructura puede proporcionar mecanismos de entrega y ordering, pero:

el consumer es responsable de garantizar que procesar un evento más de una vez no produzca resultados de negocio incorrectos.

5. Principio Fundamental

Debe distinguirse:

Delivery
¿Cuántas veces llegó el evento?

de:

Business Effect
¿Cuántas veces produjo efecto?

EVOXA acepta:

Delivery = 1..N

pero para operaciones idempotentes críticas busca:

Business Effect = 1
6. Event ID como Identidad de Procesamiento

Cada evento posee:

event_id

El event_id será el identificador principal para detectar procesamiento duplicado.

Ejemplo:

event_id = EVT-123

Si el consumer recibe:

EVT-123
EVT-123
EVT-123

deberá reconocer que corresponden al mismo evento.

7. Event ID vs Aggregate ID

No deben confundirse:

event_id

con:

aggregate_id

Un aggregate puede producir múltiples eventos:

UserCreated
UserUpdated
UserSuspended
UserActivated

todos sobre:

user_id = USER-001

pero cada evento tendrá:

event_id diferente

Por lo tanto:

Deduplicación identifica eventos; ordering y estado utilizan aggregate/context.

8. Event ID vs Correlation ID

También se deberá diferenciar:

event_id
correlation_id
causation_id
trace_id
event_id

Identifica el evento.

correlation_id

Agrupa una operación o workflow.

causation_id

Representa la relación causal.

trace_id

Permite observabilidad distribuida.

La deduplicación deberá utilizar principalmente event_id, no correlation_id.

9. Idempotency Record

Se propone conceptualmente un registro de procesamiento:

ConsumerProcessingRecord

con información como:

event_id
consumer_id
event_type
event_version
status
processed_at
result_reference
attempt_count
tenant_id
metadata

El schema físico será definido posteriormente.

10. Uniqueness Constraint

El mecanismo de deduplicación deberá tener una garantía fuerte de unicidad.

Conceptualmente:

UNIQUE (
    consumer_id,
    event_id
)

Esto permite que:

Consumer A + EVT-123

sea diferente de:

Consumer B + EVT-123

porque cada consumer mantiene su propio estado de procesamiento.

11. Por qué consumer_id + event_id

El mismo evento puede ser procesado legítimamente por múltiples consumidores:

EVT-123
  ├── Analytics
  ├── AI
  ├── Notification
  └── Reporting

Por lo tanto:

event_id

globalmente único no significa:

processed_by_every_consumer

La identidad de deduplicación deberá considerar el consumidor.

12. Processing Lifecycle

El registro de procesamiento podrá tener conceptualmente:

RECEIVED
   ↓
PROCESSING
   ↓
PROCESSED

con estados adicionales como:

FAILED
SKIPPED
DEAD_LETTER

La taxonomía final queda pendiente.

13. Atomicidad

La operación crítica será:

Check duplicate
+
Business processing
+
Mark processed

y deberá diseñarse para evitar una ventana como:

Check
 ↓
Process
 ↓
Crash
 ↓
No processed marker
 ↓
Retry
 ↓
Duplicate effect

Por ello, cuando el almacenamiento y la operación lo permitan, se deberá buscar una frontera transaccional.

14. Transactional Processing

Para operaciones internas sobre PostgreSQL:

BEGIN
   │
   ├── Check / reserve event
   ├── Business state change
   └── Record processed event
   │
COMMIT

Si ocurre:

ROLLBACK

el evento podrá ser procesado nuevamente.

Esto es preferible a marcarlo como procesado antes de ejecutar el efecto.

15. Reservation Strategy

Para evitar procesamiento concurrente:

Consumer Instance A
       │
       └── EVT-123

Consumer Instance B
       │
       └── EVT-123

deberá existir un mecanismo de:

unique constraint;
atomic insert;
row locking;
optimistic concurrency;

según el caso.

La estrategia exacta será definida a nivel técnico.

16. Concurrent Delivery

Supongamos:

T1 → Consumer A receives EVT-123
T2 → Consumer B receives EVT-123

No debe producirse:

Business Effect A
+
Business Effect B

si ambos representan el mismo procesamiento del mismo consumer.

La infraestructura de deduplicación deberá permitir que solamente uno gane la operación.

17. Unique Constraint como última defensa

Aunque la aplicación implemente:

if already_processed:
    skip

esto no es suficiente ante concurrencia.

Por eso:

La unicidad deberá estar respaldada por almacenamiento transaccional cuando el consumer lo requiera.

Ejemplo:

Application Check
        +
Database UNIQUE Constraint

La segunda capa evita carreras.

18. Processing Outcome

No basta con saber:

processed = true

En determinados workflows será útil conocer el resultado:

result_reference

Ejemplo:

event_id = EVT-123
result_reference = notification-456

Esto evita crear nuevamente un resultado cuando llega un duplicate.

19. Duplicate Handling

Cuando llega un evento ya procesado:

EVT-123

el consumer deberá:

Lookup
   ↓
Already processed
   ↓
Skip business effect
   ↓
Record duplicate metric
   ↓
ACK / complete delivery

El duplicate no debería enviarse nuevamente a DLQ simplemente por ser duplicado.

20. Duplicate ≠ Failure

Una entrega duplicada es normalmente:

Expected condition

dentro de At-Least-Once.

Por lo tanto:

Duplicate

no debe interpretarse automáticamente como:

Processing Failure

Debe distinguirse claramente en observabilidad.

21. Replay

Replay hace que la idempotencia sea todavía más importante.

Ejemplo:

Historical Event
      ↓
Original Processing
      ↓
Processed
      ↓
Months Later
      ↓
Replay
      ↓
Consumer
      ↓
Duplicate Detection
      ↓
Skip

Así, el replay no produce un segundo efecto.

El Blueprint establece explícitamente que replay no debe generar resultados de negocio duplicados.

22. DLQ

Cuando un evento llega a DLQ y posteriormente se corrige el problema:

DLQ
 ↓
Fix
 ↓
Replay

el consumer debe poder determinar si:

event_id

ya fue procesado correctamente.

Esto es particularmente importante cuando:

el error ocurrió después del business commit;
falló el ACK;
hubo timeout;
hubo un error de comunicación.
23. Retry

El mismo principio aplica a retries:

Attempt 1
   ↓
Business Commit
   ↓
ACK Failure
   ↓
Attempt 2

El segundo intento deberá ser seguro.

Por lo tanto:

Retry Safety
=
Idempotency
+
Durable Processing State
24. Consumer Restart

Si un consumer se reinicia:

Consumer
   ↓
Crash
   ↓
Restart

debe conservarse la información de procesamiento.

Por ello el estado de deduplicación no deberá depender exclusivamente de memoria local.

25. No usar memoria como única deduplicación

Este patrón no será suficiente:

processed_events = set()

porque se pierde ante:

restart;
scaling;
failover;
múltiples replicas.

La deduplicación crítica deberá utilizar almacenamiento durable apropiado.

26. Redis

Redis podrá utilizarse como:

cache;
acelerador;
deduplication optimization;

pero no deberá convertirse automáticamente en la única autoridad para garantías críticas de procesamiento.

Esto es consistente con la arquitectura de Identity donde Redis es auxiliar y PostgreSQL es source of truth.

La utilización de Redis para deduplicación deberá evaluarse según el workflow.

27. PostgreSQL

Para consumidores con efectos críticos sobre PostgreSQL, una estrategia preferida será:

PostgreSQL
    │
    ├── Business State
    └── Processing Record

en una frontera transaccional.

Esto permite:

Business State
+
Idempotency State

commit juntos.

28. External Side Effects

El caso más complejo es:

Event
 ↓
DB Transaction
 ↓
External API

No siempre es posible hacer:

DB
+
External API

en una misma transacción.

Ejemplo:

PaymentCompleted
       ↓
Send External Notification

Si ocurre:

External API success
       ↓
Consumer crash
       ↓
No local processed marker

el retry puede repetir la llamada externa.

Por ello:

Los efectos externos requieren mecanismos adicionales de idempotencia.

29. Idempotency Key hacia sistemas externos

Cuando el sistema externo lo soporte, EVOXA deberá utilizar:

Idempotency-Key

derivada de una identidad estable de la operación.

Conceptualmente:

event_id
    ↓
External Idempotency Key

Esto evita:

same event
    ↓
same external operation
    ↓
duplicate effect

La implementación concreta dependerá del proveedor externo.

30. Outbox / Inbox Pattern

La arquitectura podrá utilizar conceptualmente:

Transactional Outbox

para productores y:

Transactional Inbox

para consumers.

Flujo:

Producer
   ↓
Outbox
   ↓
Broker
   ↓
Consumer
   ↓
Inbox
   ↓
Business State

La Inbox representa el estado de procesamiento del consumer.

31. Inbox Semantics

El patrón Inbox deberá permitir:

event received
      ↓
register processing
      ↓
execute business effect
      ↓
mark processed

dentro de una estrategia segura contra concurrencia y fallos.

32. Exactly-Once

EVOXA no establecerá exactamente-once delivery como garantía global del Event Platform.

La arquitectura mantiene:

At-Least-Once Delivery

y consigue efectos únicos mediante:

Consumer Idempotency
+
Deduplication
+
Transactional Processing
+
External Idempotency

cuando el workflow lo requiera.

33. Exactly-Once Business Effect

Por lo tanto, el concepto aceptado será:

Exactly-once business effect where required, not exactly-once message delivery.

Esta distinción es fundamental.

34. Idempotency Scope

La idempotencia deberá definirse por consumer y operación.

Ejemplo:

Analytics Consumer

puede necesitar:

consumer_id + event_id

Mientras un consumer que produzca una operación externa podría requerir:

consumer_id
+
event_id
+
operation_type

La clave exacta deberá definirse según el caso.

35. Event Version

La deduplicación normalmente utilizará:

event_id

como identidad primaria.

No deberá tratar:

UserCreated v1

y:

UserCreated v2

como eventos diferentes si representan el mismo event_id.

La versión describe el contrato; el event_id identifica la instancia concreta del hecho.

36. Consumer Version

Un cambio de implementación del consumer tampoco deberá necesariamente invalidar el historial de deduplicación.

Por ejemplo:

Consumer v1

procesa:

EVT-123

y posteriormente:

Consumer v2

recibe el mismo evento.

La estrategia de migration deberá determinar si:

comparte deduplication state;
utiliza namespace versionado;
reconstruye estado;
requiere replay controlado.
37. Consumer Group

En un broker con consumer groups deberá distinguirse:

Consumer Group

de:

Consumer Instance

La deduplicación no debe depender de la instancia concreta.

Ejemplo:

Group: analytics
 ├── instance A
 ├── instance B
 └── instance C

Si EVT-123 fue procesado por A, B no debe volver a producir el efecto si recibe el mismo evento posteriormente.

38. Tenant Isolation

La deduplicación deberá respetar tenant context.

Conceptualmente:

tenant_id
+
consumer_id
+
event_id

podrá formar parte del modelo lógico dependiendo del diseño.

No deberá permitirse que el estado de deduplicación de un tenant interfiera accidentalmente con otro.

39. Security Events

Los consumidores de eventos de seguridad requieren especial cuidado.

Ejemplo:

UserAuthenticationFailed

Si se procesa dos veces incorrectamente podría:

increment attempts

dos veces.

Por ello, los eventos de seguridad deberán tener mecanismos de idempotencia particularmente estrictos.

40. MFA

Eventos relacionados con MFA podrían incluir:

MFAChallengeCreated
MFAChallengeVerified
MFAFactorActivated

Un duplicate no deberá:

verify challenge twice
activate factor twice
consume recovery code twice

La idempotencia debe combinarse con las reglas transaccionales del dominio.

41. Authorization

Un evento:

MembershipRoleChanged

podría ser recibido nuevamente.

El consumer deberá evitar:

duplicate permission grant

o efectos equivalentes.

Los cambios de autorización deben además mantener coherencia con:

cache invalidation;
audit;
policy evaluation.
42. Audit Events

Los audit events tienen una particularidad.

Un evento de auditoría representa un hecho histórico.

Por ello:

Duplicate delivery

no deberá crear dos registros de auditoría equivalentes.

La identidad del evento deberá permitir detectar duplicados.

43. AI Consumers

Los consumidores AI también deberán ser idempotentes.

Ejemplo:

RecommendationGenerated

No debería producir dos recomendaciones idénticas simplemente porque el evento fue entregado dos veces.

Para operaciones de AI se deberán considerar además:

model version;
prompt version;
inference identifier;
correlation;
tenant;
authorization context.

La estrategia específica de AI Idempotency podrá requerir ADR posterior si la complejidad aumenta.

44. Idempotency Retention

El estado de deduplicación también necesita una política de retención.

Si se elimina demasiado pronto:

event processed
   ↓
dedup record deleted
   ↓
historical replay
   ↓
event processed again

puede producirse un duplicado.

Por lo tanto:

La retención del estado de idempotencia debe ser compatible con la ventana máxima de retry, replay y retención del evento.

45. Retention Relationship

Conceptualmente:

Deduplication Retention
          ≥
Relevant Replay / Retry Window

No se fija un período exacto en este ADR.

Deberá definirse en conjunto con:

ADR-013;
ADR-020;
ADR-021.
46. Cleanup

Los registros antiguos de deduplicación podrán eliminarse o archivarse cuando ya no sean necesarios.

La eliminación deberá ser:

controlada;
observable;
compatible con replay;
tenant-aware.
47. Failure Matrix
Situación	Comportamiento
Event nuevo	Procesar
Event duplicate	Skip
Consumer crash antes commit	Retry
Consumer crash después commit	Dedup
ACK perdido	Retry + Dedup
DLQ después de éxito	Dedup
Replay de evento procesado	Skip
Replay de evento no procesado	Process
Concurrent delivery	Una ejecución efectiva
Schema inválido	No procesar / DLQ según política
Unauthorized event context	Reject
Tenant mismatch	Reject
48. Observability

Los consumers deberán medir:

events_received
events_processed
events_duplicate
events_failed
events_retried
events_dlq
processing_latency
deduplication_latency

Además:

event_id
consumer_id
tenant_id
correlation_id
trace_id

deberán estar disponibles para diagnóstico según las políticas de seguridad.

49. Duplicate Rate

Una métrica importante será:

duplicate_rate

Un aumento inesperado puede indicar:

broker issues;
consumer instability;
ACK failures;
timeout problems;
replay accidental;
producer duplication.

Por lo tanto, duplicate rate no debe simplemente ignorarse.

50. Alerts

Podrán definirse alertas para:

High duplicate rate
High retry rate
High DLQ rate
Processing failures
Dedup store failures
Replay collisions

La política definitiva se definirá con Observability ADR.

51. Performance

La deduplicación debe tener bajo overhead.

El camino normal será:

Receive
 ↓
Dedup Check
 ↓
Process

Por lo tanto, deberán medirse:

dedup latency;
DB contention;
storage growth;
lock contention.

No se deberá sacrificar integridad para conseguir micro-optimizaciones.

52. Cache Optimization

Podrá utilizarse:

Redis

como optimización para eventos recientemente procesados.

Pero:

Cache miss

deberá poder recurrir a la fuente durable cuando el workflow requiera garantía fuerte.

La arquitectura no debe aceptar:

Redis unavailable
     ↓
Idempotency disabled

para operaciones críticas.

53. Consumer Contract

Cada consumer deberá declarar conceptualmente:

Idempotency Supported: Yes/No
Deduplication Key: ...
Side Effects: ...
Replay Safe: Yes/No
Retry Safe: Yes/No

Los consumers críticos no podrán declarar simplemente:

unknown

sin una revisión arquitectónica.

54. Event Governance Integration

El gobierno de eventos ya requiere:

producer;
consumers;
schema;
version;
retention;
business rules;
owner.

Se propone incorporar conceptualmente:

Idempotency Policy
Deduplication Strategy
Replay Safety
Side Effect Policy

al contrato del evento o metadata asociada.

55. Contract Testing

Los contract tests deberán verificar que un consumer:

procesa un evento nuevo;
procesa nuevamente el mismo evento;
no duplica el resultado;
soporta retry;
soporta replay;
maneja concurrencia;
respeta tenant isolation.

Ejemplo:

Given EVT-123
When processed twice
Then business effect exists once
56. E2E Test

Un escenario crítico será:

Publish EVT-123
       ↓
Consumer processes
       ↓
ACK fails
       ↓
Broker retries
       ↓
Consumer receives EVT-123
       ↓
Dedup detects
       ↓
No duplicate business effect

Otro:

EVT-123
 ↓
Processed
 ↓
Months later
 ↓
Replay
 ↓
Consumer
 ↓
No duplicate effect
57. Chaos Testing

La estrategia deberá probar:

consumer crash;
network interruption;
broker redelivery;
DB failure;
Redis failure;
worker restart;
concurrent delivery;
replay durante processing.

Esto permitirá verificar que la garantía no dependa de condiciones ideales.

58. Consecuencias positivas
Reliability

Tolera At-Least-Once delivery.

Replay Safety

Permite replay sin duplicar resultados.

Recovery

Facilita recuperación después de fallos.

Horizontal Scaling

Permite múltiples instancias del mismo consumer.

Data Integrity

Protege contra efectos duplicados.

Operational Resilience

Tolera ACK failures y restarts.

Security

Protege workflows críticos de Identity.

59. Consecuencias negativas

Introduce:

storage adicional;
procesamiento de deduplicación;
índices;
locks/concurrency;
lifecycle de registros;
complejidad para efectos externos;
necesidad de contract tests.

También requiere que cada consumer comprenda su propia semántica de idempotencia.

60. Alternativas consideradas
Alternativa A — Exactly-Once Delivery

Rechazada como garantía global.

La arquitectura EVOXA se basa en At-Least-Once.

Alternativa B — Deduplicación solamente en Broker

Rechazada.

El broker no conoce necesariamente el efecto de negocio.

Alternativa C — Deduplicación solamente en Redis

Rechazada como garantía universal.

No debe ser la única autoridad para operaciones críticas.

Alternativa D — Consumer sin deduplicación

Rechazada.

Contradice los requisitos del Event Architecture.

Alternativa E — Consumer-side durable idempotency

Seleccionada.

Es la opción compatible con:

At-Least-Once;
Retry;
DLQ;
Replay;
horizontal scaling.
61. Non-Negotiables
EVOXA utilizará At-Least-Once delivery.
Los consumers deben tolerar duplicados.
Los workflows críticos deben implementar deduplicación.
event_id será la identidad principal del evento.
event_id no se debe confundir con aggregate_id.
event_id no se debe confundir con correlation_id.
La deduplicación debe considerar el consumer.
La deduplicación crítica debe tener almacenamiento durable.
La unicidad debe estar respaldada por mecanismos transaccionales cuando corresponda.
Los duplicate events no deben producir efectos de negocio duplicados.
Replay debe ser seguro respecto a idempotencia.
Retry debe ser seguro respecto a idempotencia.
Consumer restart no debe destruir el estado de deduplicación.
Redis no será la única autoridad para garantías críticas.
Los efectos externos necesitan mecanismos adicionales de idempotencia.
Los efectos financieros y de seguridad requieren controles reforzados.
Tenant isolation debe mantenerse.
La retención de deduplication state debe ser compatible con replay/retry.
Duplicate no equivale automáticamente a failure.
La observabilidad debe diferenciar duplicates de failures.
Los consumers deben probarse contra concurrent delivery.
Los consumers deben probarse contra replay.
No se asumirá exactly-once delivery global.
Se buscará exactly-once business effect cuando el workflow lo requiera.
62. Traceability
Event Architecture

El Blueprint establece:

At-Least-Once
Retry
DLQ
Replay
Idempotent Consumers

También establece que los workflows críticos implementan deduplicación.

Replay

Replay debe permitir recovery, analytics, AI training, debugging y data reconstruction sin producir resultados duplicados.

Event Governance

Cada evento tiene owner, publisher, consumers, version, schema y retention policy.

Event Contracts

Los contratos incluyen producers, consumers, schemas, routing, retry, retention, telemetry y contract tests.

63. ADR relacionados
ADR-IDENTITY-007 — Event Architecture
ADR-IDENTITY-013 — Data Retention
ADR-IDENTITY-015 — Event Partitioning
ADR-IDENTITY-016 — Concurrency & Transaction Integrity
ADR-IDENTITY-018 — Event Broker Strategy
ADR-IDENTITY-019 — Event Partitioning / Ordering / Routing
ADR-IDENTITY-020 — Event Schema Registry & Compatibility
ADR-IDENTITY-021 — Event Replay & Governance
ADR-IDENTITY-028 — Observability / Operational Security
ADR-IDENTITY-029 — Disaster Recovery / Backup
64. Dependencias

La cadena queda ahora:

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
Consumer Idempotency / Deduplication

Y el flujo completo:

                 ┌──────────────────┐
                 │ Domain Operation │
                 └────────┬─────────┘
                          ↓
                 Transactional Outbox
                          ↓
                    Event Broker
                          ↓
                    Consumer
                          ↓
                Idempotency Check
                    /          \
              Duplicate        New
                 ↓               ↓
                Skip          Process
                                  ↓
                           Business State
                                  ↓
                          Mark Processed
65. Decisiones pendientes

Quedan deliberadamente abiertas:

Storage
PostgreSQL Inbox;
dedicated storage;
broker-native state;
combinación.
Deduplication Key
consumer_id + event_id;
tenant_id + consumer_id + event_id;
otra clave para determinados workflows.
Retention
ventana de retry;
ventana de replay;
archive;
cleanup.
Concurrency
unique constraint;
optimistic concurrency;
pessimistic locking;
atomic insert.
External Effects
idempotency keys;
outbox;
workflow orchestration;
provider-specific guarantees.
Consumer Migration
shared dedup state;
versioned namespace;
migration strategy.
Redis
cache;
optimization;
temporary deduplication;
failover behavior.
66. Criterios de aceptación

ADR-IDENTITY-022 podrá pasar de Proposed → Accepted cuando esté definido:

 Consumer Idempotency Model.
 Durable Deduplication Store.
 Deduplication Key.
 Unique Constraint Strategy.
 Inbox Pattern.
 Processing Lifecycle.
 Transaction Boundary.
 Concurrent Delivery Strategy.
 Retry Strategy.
 DLQ Integration.
 Replay Integration.
 External Side Effect Strategy.
 External Idempotency Keys.
 Tenant Isolation.
 Retention Policy.
 Cleanup Strategy.
 Consumer Version Migration.
 Consumer Group Semantics.
 Redis Optimization Strategy.
 Metrics.
 Alerts.
 Contract Tests.
 Integration Tests.
 Concurrency Tests.
 Replay Tests.
 Chaos Tests.
 Operational Runbooks.
67. Resultado arquitectónico

Con este ADR se completa una pieza fundamental del modelo Event-Driven de EVOXA:

┌─────────────────────────────────────────────────────┐
│                 EVOXA EVENT PLATFORM                │
├─────────────────────────────────────────────────────┤
│                                                     │
│  018  Event Broker                                  │
│        ↓                                            │
│  019  Partitioning / Ordering / Routing             │
│        ↓                                            │
│  020  Schema Registry / Compatibility               │
│        ↓                                            │
│  021  Replay / Governance                           │
│        ↓                                            │
│  022  Consumer Idempotency / Deduplication          │
│                                                     │
└─────────────────────────────────────────────────────┘

La garantía conceptual queda:

At-Least-Once Delivery
          +
Retry
          +
DLQ
          +
Replay
          ↓
Consumer Idempotency
          +
Deduplication
          +
Transactional Processing
          ↓
Exactly-Once Business Effect
when required

Y esta es la distinción arquitectónica clave:

EVOXA no intentará hacer que la infraestructura entregue cada mensaje exactamente una vez. EVOXA hará que los consumidores procesen de forma segura una o múltiples entregas del mismo evento, evitando efectos de negocio duplicados.

Estado final: Proposed.

El siguiente ADR lógico es ADR-IDENTITY-023 — MFA Factor Strategy, ya que después de cerrar el bloque de Event Platform 018–022, retomamos las decisiones pendientes específicas de MFA que dejamos abiertas en IS-IDENTITY-010.
