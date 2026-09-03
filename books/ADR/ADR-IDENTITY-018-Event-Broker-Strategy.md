ADR-IDENTITY-018 — Event Broker Strategy

Estado: Proposed
Versión: 1.0
Fecha: 2026-09-02
Dominio: Identity / Event Platform
Prioridad: Critical
Tipo: Event Architecture / Integration / Reliability / Scalability / Security

1. Título

ADR-IDENTITY-018 — Event Broker Strategy

2. Contexto

EVOXA adopta una arquitectura Event-Driven en la que los cambios relevantes del dominio pueden producir eventos que serán consumidos por otros componentes de la plataforma.

En Identity ya se ha establecido:

Domain
   ↓
Application
   ↓
PostgreSQL Transaction
   ├── Domain State
   └── Outbox Event

La publicación hacia otros componentes no debe realizarse directamente desde la transacción de negocio.

La arquitectura definida para eventos establece:

eventos inmutables;
eventos versionados;
entrega at-least-once;
consumidores idempotentes;
retries;
Dead Letter Queue;
replay;
observabilidad;
seguridad;
tenant isolation;
correlación mediante IDs;
Transactional Outbox.

El Blueprint también contempla una Event Platform como componente de infraestructura.

Por lo tanto, falta definir la estrategia arquitectónica del componente que recibirá los eventos desde el Outbox y los distribuirá a los consumidores.

3. Problema

Necesitamos un mecanismo que permita pasar de:

PostgreSQL
    ↓
Outbox

a:

Event Platform
    ↓
Multiple Consumers

de manera:

confiable;
escalable;
observable;
segura;
desacoplada;
compatible con retries;
compatible con DLQ;
compatible con replay;
compatible con evolución de schemas.

Sin una decisión formal sobre el broker podrían aparecer implementaciones incompatibles entre dominios.

Por ejemplo:

Identity → Broker A
Billing  → Broker B
Fitness  → Broker C
AI       → Broker D

lo que fragmentaría la arquitectura de eventos de EVOXA.

4. Decisión

EVOXA adoptará una Event Broker Platform centralizada y compartida, utilizada como infraestructura común para la comunicación asíncrona entre dominios.

La arquitectura será:

                    ┌───────────────────┐
                    │    PostgreSQL     │
                    └─────────┬─────────┘
                              │
                         Transaction
                              │
                              ▼
                    ┌───────────────────┐
                    │ Transactional     │
                    │ Outbox            │
                    └─────────┬─────────┘
                              │
                         Outbox Relay
                              │
                              ▼
                    ┌───────────────────┐
                    │   Event Broker    │
                    │  Event Platform   │
                    └─────────┬─────────┘
                              │
             ┌────────────────┼────────────────┐
             │                │                │
             ▼                ▼                ▼
        Consumer A       Consumer B       Consumer C

El broker será responsable principalmente de:

transporte;
distribución;
buffering;
delivery;
consumer coordination;
retry infrastructure;
DLQ;
ordering dentro de los límites definidos;
observabilidad de transporte.

El broker no será la fuente de verdad del dominio.

5. Separación de responsabilidades

La arquitectura distinguirá claramente:

PostgreSQL

Fuente de verdad transaccional.

Domain State
Transactional Outbox

Garantiza:

Domain State + Event

dentro de la misma transacción.

Event Broker

Distribuye:

Integration Events
Consumers

Procesan eventos y actualizan sus propios modelos o ejecutan acciones.

Event Store / Histórico

Cuando sea requerido, podrá existir almacenamiento histórico adicional, pero no se deberá asumir que el broker constituye por sí mismo el almacenamiento histórico permanente.

6. Broker no es Database

El Event Broker no sustituirá PostgreSQL.

No deberá utilizarse como:

Primary Database

para:

usuarios;
organizaciones;
memberships;
roles;
permisos;
sesiones;
MFA;
políticas de seguridad.

El broker proporciona transporte y distribución, no autoridad de negocio.

7. Modelo de publicación

El modelo principal será:

Business Transaction
       │
       ▼
PostgreSQL
       │
       ├── Domain State
       │
       └── Outbox Event
              │
           COMMIT
              │
              ▼
        Outbox Relay
              │
              ▼
         Event Broker

Esto evita el patrón problemático:

DB commit
    ↓
Publish event

donde una caída del broker podría producir pérdida de eventos.

8. Outbox Relay

El Outbox Relay será responsable de recoger eventos pendientes y publicarlos al broker.

Conceptualmente:

outbox_events
      │
      ▼
  Relay Worker
      │
      ├── claim
      ├── publish
      ├── retry
      └── mark published

El relay deberá ser capaz de ejecutarse con múltiples instancias cuando sea necesario.

La estrategia de concurrencia deberá alinearse con:

ADR-IDENTITY-016 — Concurrency & Transaction Strategy.

9. At-Least-Once Delivery

La estrategia de entrega será:

At-Least-Once

Esto significa que un evento puede llegar más de una vez.

Ejemplo:

Event E1
   ↓
Broker
   ↓
Consumer
   ↓
Processing SUCCESS
   ↓
ACK lost
   ↓
Event E1 delivered again

Por ello:

Los consumidores deben ser idempotentes.

La estrategia específica de idempotencia será desarrollada en:

ADR-IDENTITY-022 — Consumer Idempotency.

10. Exactly-Once

No se establecerá Exactly-Once End-to-End como requisito global de EVOXA.

Aunque determinados brokers puedan proporcionar garantías adicionales en determinados escenarios, la arquitectura de EVOXA tratará la entrega distribuida como:

At-Least-Once
+
Idempotent Consumers

Esto proporciona una estrategia más robusta para sistemas distribuidos.

11. Topics / Streams

Los eventos se organizarán mediante una estructura lógica de canales del broker.

Conceptualmente:

identity.user
identity.organization
identity.membership
identity.session
identity.authentication
identity.authorization
identity.mfa
identity.security
identity.policy

Estos nombres son conceptuales, no definitivos.

La taxonomía final será definida conjuntamente con:

ADR-IDENTITY-019 — Event Partitioning, Ordering & Routing.

12. Event Types

El broker transportará eventos con tipos explícitos.

Ejemplos:

UserCreated
UserActivated
UserSuspended

OrganizationCreated
OrganizationSuspended

MembershipCreated
MembershipActivated
MembershipRemoved

SessionCreated
SessionRevoked

RefreshTokenRotated
RefreshTokenReuseDetected

MFAFactorActivated
MFAChallengeVerified

AuthorizationDenied
PolicyViolationDetected

Los nombres y versiones definitivos pertenecen al catálogo de eventos.

13. Event Envelope

Los eventos deberán utilizar el envelope definido por la arquitectura de EVOXA.

Conceptualmente:

{
  event_id,
  event_type,
  event_version,
  timestamp,
  tenant_id,
  actor,
  resource,
  payload,
  metadata
}

Y metadata relacionada con:

correlation ID;
causation ID;
trace ID;
producer;
schema version.

El envelope exacto deberá mantenerse compatible con la arquitectura general de eventos.

14. Tenant Isolation

Los eventos tenant-scoped deberán transportar información suficiente para mantener el aislamiento.

Conceptualmente:

Event
 ├── tenant_id
 ├── actor
 ├── resource
 └── payload

Un consumer deberá poder determinar:

¿A qué tenant pertenece este evento?

Los eventos que no pertenezcan a un tenant específico deberán clasificarse explícitamente como:

SYSTEM / PLATFORM

cuando corresponda.

15. Seguridad del Broker

El Event Broker deberá proteger:

autenticación de productores;
autenticación de consumidores;
autorización;
cifrado en tránsito;
cifrado en almacenamiento cuando corresponda;
aislamiento de tenants;
acceso mínimo;
gestión de credenciales;
auditoría.

Un consumer no deberá tener automáticamente acceso a todos los eventos de EVOXA.

16. Least Privilege

El acceso deberá ser granular.

Conceptualmente:

Identity Publisher
      ↓
publish identity.*

Billing Consumer
      ↓
consume billing.*

AI Consumer
      ↓
consume authorized event types

No se deberá conceder:

consume *

sin justificación.

17. Event Classification

Los eventos deberán clasificarse según su sensibilidad.

Por ejemplo:

PUBLIC
INTERNAL
CONFIDENTIAL
SECURITY_SENSITIVE

La clasificación concreta será definida en el gobierno de eventos.

Los eventos de Identity pueden contener información especialmente sensible.

Por ello:

El hecho de que un dato pueda transportarse mediante eventos no significa que deba incluirse en el evento.

18. No Secrets in Events

Queda prohibido incluir en eventos:

password
password_hash
access_token
refresh_token
TOTP secret
OTP
recovery_code
private_key
encryption_key
database_password
service_secret

Los eventos deben utilizar referencias cuando sea suficiente.

19. Retry

Los errores temporales deberán permitir retry.

Conceptualmente:

Publish
   ↓
Failure
   ↓
Retry 1
   ↓
Retry 2
   ↓
Retry 3
   ↓
DLQ

La estrategia deberá utilizar backoff.

La arquitectura de eventos previamente definida contempla:

Exponential Backoff

como estrategia preferida.

Los parámetros exactos quedan pendientes.

20. Dead Letter Queue

Los eventos que no puedan procesarse después de los retries definidos deberán poder llegar a una DLQ.

Conceptualmente:

Event
 ↓
Consumer
 ↓
Failure
 ↓
Retry
 ↓
Retry
 ↓
DLQ

La DLQ debe conservar información suficiente para:

diagnóstico;
identificación del evento;
identificación del consumer;
causa del error;
replay controlado.
21. Replay

La plataforma deberá permitir replay cuando sea necesario.

Ejemplo:

Historical Events
       ↓
Replay
       ↓
Consumer
       ↓
Rebuild Projection

El replay no deberá modificar el evento histórico.

Los eventos originales permanecen inmutables.

La estrategia detallada se definirá en:

ADR-IDENTITY-021 — Event Replay & Governance.

22. Ordering

No se garantizará orden global.

La garantía será limitada a los límites definidos posteriormente.

Por ejemplo:

User U1
  Event 1
  Event 2
  Event 3

podrá requerir orden:

1 → 2 → 3

mientras que:

User U1
User U2

no necesita necesariamente:

U1 Event 1
U2 Event 1
U1 Event 2
...

en un orden global.

La estrategia exacta se define en:

ADR-IDENTITY-019.

23. Consumer Groups

La plataforma deberá soportar conceptualmente grupos de consumidores.

Ejemplo:

identity.user.events
          │
          ├── Analytics Consumer Group
          ├── Notification Consumer Group
          ├── Search Consumer Group
          └── AI Context Consumer Group

Cada grupo puede procesar el evento independientemente.

24. Producer Scaling

El sistema deberá permitir múltiples Outbox Relay workers.

Conceptualmente:

             PostgreSQL
                 │
          ┌──────┴──────┐
          │   Outbox    │
          └──────┬──────┘
                 │
       ┌─────────┼─────────┐
       ▼         ▼         ▼
    Relay 1   Relay 2   Relay 3
       │         │         │
       └─────────┼─────────┘
                 ▼
             Broker

La coordinación deberá evitar publicar repetidamente un mismo evento de manera innecesaria, aunque la arquitectura seguirá asumiendo que duplicados pueden producirse.

25. Broker Failure

Si el broker está temporalmente indisponible:

PostgreSQL
    ↓
Outbox
    ↓
Broker DOWN

el evento deberá permanecer en Outbox.

Cuando el broker vuelva:

Outbox
   ↓
Relay
   ↓
Broker

continuará la publicación.

Esto constituye una de las principales razones para utilizar Transactional Outbox.

26. Consumer Failure

Si un consumer falla:

Broker
   ↓
Consumer DOWN

el broker debe permitir que el evento permanezca disponible según su política de delivery/retention.

Cuando el consumer se recupere:

Consumer UP
   ↓
Resume

La estrategia exacta dependerá del mecanismo de broker seleccionado.

27. Poison Messages

Un evento puede provocar fallos repetidos.

Ejemplo:

Event E
 ↓
Consumer
 ↓
Failure
 ↓
Retry
 ↓
Failure
 ↓
Retry
 ↓
Failure

No se deberá permitir un retry infinito.

La estrategia será:

Retry limit
   ↓
DLQ
   ↓
Investigation
   ↓
Fix
   ↓
Replay
28. Event Retention

La retención del broker deberá distinguirse de:

retención del Outbox;
retención de Audit;
retención histórica;
retención de eventos archivados.

No se deberá asumir que el broker es el repositorio histórico permanente.

La política exacta será definida en:

ADR-IDENTITY-021 — Event Replay & Governance.

29. Performance

La arquitectura general de eventos establece objetivos conceptuales para:

publicación;
delivery;
procesamiento;
replay.

El broker deberá ser capaz de soportar los objetivos de plataforma sin convertirse en un cuello de botella.

La selección concreta deberá validarse mediante:

benchmarks;
load tests;
stress tests;
failure tests.
30. Backpressure

La arquitectura deberá contemplar diferencias entre:

Producer Rate
      >
Consumer Rate

Esto puede producir backlog.

El sistema deberá observar:

queue depth;
consumer lag;
processing latency;
retry volume.

Cuando sea necesario, se deberá permitir escalar consumidores.

31. Consumer Lag

El retraso de consumidores será una métrica crítica.

Conceptualmente:

Published
   │
   ▼
Broker
   │
   │ ← Consumer Lag
   ▼
Processed

Un incremento sostenido puede indicar:

consumer failure;
insufficient capacity;
broker saturation;
poison events;
downstream failure.
32. Observability

La Event Platform deberá integrarse con observabilidad distribuida.

Se deberán poder correlacionar:

HTTP Request
      ↓
Domain Operation
      ↓
Database Transaction
      ↓
Outbox Event
      ↓
Broker
      ↓
Consumer

utilizando:

correlation ID;
causation ID;
trace ID;
event ID.
33. Métricas

Métricas conceptuales:

event_publish_rate
event_publish_latency
event_delivery_latency
event_consumer_lag
event_retry_count
event_dlq_count
event_processing_latency
event_processing_failures
event_replay_rate
outbox_backlog

Los nombres finales deberán seguir los estándares de observabilidad de EVOXA.

34. Auditabilidad

Las operaciones administrativas sensibles del Event Platform deberán poder auditarse.

Por ejemplo:

creación de topic;
modificación de permisos;
cambio de retention;
creación de consumer;
modificación de routing;
replay administrativo;
acceso a DLQ.
35. Disaster Recovery

El Event Broker deberá formar parte de la estrategia de continuidad operacional.

Sin embargo:

La recuperación del broker no sustituye la recuperación de PostgreSQL ni del Outbox.

La arquitectura debe permitir reconstruir el flujo desde la fuente transaccional cuando sea necesario.

Esto se coordinará con:

ADR-IDENTITY-029 — Disaster Recovery & Backup.

36. Multi-Region

El Blueprint contempla una arquitectura preparada para escenarios de alta disponibilidad y disaster recovery.

La estrategia multi-region del broker queda pendiente de una decisión de infraestructura.

No se asumirá automáticamente:

Active/Active

ni:

Active/Passive

sin análisis de:

consistencia;
ordering;
latency;
replication;
failover;
costo;
replay.
37. Tecnología del Broker

Este ADR no fija todavía un producto específico.

La selección deberá evaluarse posteriormente en función de:

throughput;
latency;
durability;
ordering;
partitioning;
replay;
DLQ;
retention;
consumer groups;
security;
operational complexity;
cloud compatibility;
cost;
ecosystem integration.

Esto evita convertir una decisión arquitectónica en una decisión prematura de proveedor.

38. Candidatos tecnológicos

Podrán evaluarse soluciones de categorías como:

Kafka-compatible platforms;
managed event streaming platforms;
cloud-native messaging systems;
otros brokers compatibles con los requisitos.

La elección final deberá realizarse mediante un análisis técnico y, cuando corresponda, benchmark.

39. Consecuencias positivas
Desacoplamiento

Los dominios no necesitan conocerse directamente.

Identity
   ↓
Event Broker
   ↓
Multiple Domains
Escalabilidad

Los productores y consumidores pueden escalar independientemente.

Resiliencia

El Outbox protege contra la indisponibilidad temporal del broker.

Extensibilidad

Nuevos consumidores pueden incorporarse sin modificar Identity.

Replay

La arquitectura permite reprocesar eventos cuando sea necesario.

Observabilidad

La cadena completa puede correlacionarse mediante IDs.

Evolución

La plataforma permite incorporar nuevos dominios de EVOXA.

40. Consecuencias negativas

Introduce complejidad operacional:

broker management;
topics;
partitions;
consumers;
consumer groups;
lag;
retries;
DLQ;
replay;
schema evolution;
observability.

Además, introduce una característica fundamental de sistemas distribuidos:

Los consumidores deben asumir que los eventos pueden llegar duplicados.

41. Alternativas consideradas
Alternativa A — Comunicación HTTP síncrona entre dominios

No seleccionada como mecanismo principal de eventos.

Genera mayor acoplamiento temporal.

Identity
   ↓ HTTP
Billing
   ↓
Notification

Una dependencia caída puede afectar al productor.

Alternativa B — PostgreSQL como Event Queue

Rechazada como arquitectura general.

El Outbox es adecuado como buffer transaccional, pero no debe convertirse en el Event Platform completo de EVOXA.

Alternativa C — Redis Pub/Sub como Event Platform principal

Rechazada como estrategia principal.

No proporciona por sí mismo todas las capacidades necesarias para:

durable event streaming;
replay;
governance;
historical retention;
consumer coordination.

Redis continúa siendo una infraestructura auxiliar.

Alternativa D — Broker independiente por dominio

Rechazada inicialmente.

Aumentaría:

complejidad;
costo operacional;
fragmentación;
dificultad de observabilidad.
Alternativa E — Event Broker centralizado

Seleccionada.

Proporciona una plataforma común y gobernada para EVOXA.

42. Non-Negotiables
Event Broker no es fuente de verdad del dominio.
PostgreSQL permanece como source of truth.
Transactional Outbox es el mecanismo de publicación confiable.
Delivery será tratado como at-least-once.
Los consumidores deben ser idempotentes.
No se requiere exactly-once end-to-end.
No existe orden global.
Tenant isolation es obligatorio.
No se permiten secretos en eventos.
El acceso al broker debe aplicar least privilege.
Deben existir retries.
Debe existir DLQ.
Debe existir estrategia de replay.
Debe existir observabilidad.
Debe existir correlación mediante event/correlation/causation/trace IDs.
Broker failure no debe provocar pérdida de eventos confirmados en Outbox.
Los consumers deben soportar fallos y recuperación.
Poison messages no deben generar retries infinitos.
El broker debe poder escalar horizontalmente cuando la tecnología elegida lo permita.
La tecnología concreta del broker no se fija mediante este ADR.
La retención del broker no equivale a retención histórica permanente.
Los permisos del broker deben ser mínimos y explícitos.
43. Traceability
Blueprint

Relacionado con:

Event-Driven Architecture;
Event Platform;
Integration;
Scalability;
Reliability;
Observability;
Security;
Tenant Isolation;
Replay;
DLQ.
ESP

ESP-0001 — Identity Domain

Especialmente:

Domain Events;
Integration Events;
Security Events;
Audit Events;
Outbox.
Implementation Stories

Principalmente:

IS-IDENTITY-012 — Audit & Security Events
IS-IDENTITY-013 — Domain Events & Outbox
IS-IDENTITY-015 — Identity Persistence
IS-IDENTITY-016 — Identity Testing & Quality
IS-IDENTITY-017 — Identity Runtime & Deployment
44. ADR relacionados
ADR-IDENTITY-007 — Event Architecture
ADR-IDENTITY-013 — Data Retention
ADR-IDENTITY-015 — Database Partitioning
ADR-IDENTITY-016 — Concurrency & Transaction Strategy
ADR-IDENTITY-019 — Event Partitioning / Ordering / Routing
ADR-IDENTITY-020 — Event Schema Registry / Compatibility
ADR-IDENTITY-021 — Event Replay / Governance
ADR-IDENTITY-022 — Consumer Idempotency
ADR-IDENTITY-028 — Observability / Operational Security
ADR-IDENTITY-029 — Disaster Recovery / Backup
ADR-IDENTITY-030 — Autoscaling / Capacity
45. Dependencias

La arquitectura queda:

ADR-IDENTITY-007
Event Architecture
       │
       ▼
ADR-IDENTITY-013
Transactional Outbox
       │
       ▼
ADR-IDENTITY-016
Concurrency
       │
       ▼
ADR-IDENTITY-018
Event Broker
       │
       ├───────────────┐
       ▼               ▼
ADR-019             ADR-020
Ordering            Schema
Routing             Compatibility
       │               │
       └───────┬───────┘
               ▼
           ADR-021
        Replay/Governance
               │
               ▼
           ADR-022
      Consumer Idempotency
46. Decisiones pendientes

Este ADR deja explícitamente abiertas:

Broker
producto;
proveedor;
managed vs self-managed.
Topology
topics;
streams;
namespaces;
tenant strategy.
Partitioning
partition key;
número de partitions;
rebalancing.
Ordering
garantía por aggregate;
user;
tenant;
workflow.
Retention
duración por categoría;
broker retention;
archival.
DLQ
estructura;
retention;
replay process.
Security
authentication;
authorization;
ACLs;
encryption;
credential rotation.
Multi-region
active/active;
active/passive;
replication;
failover.
Capacity
throughput;
partitions;
consumers;
storage;
autoscaling.
47. Criterios de aceptación

ADR-IDENTITY-018 podrá pasar de Proposed a Accepted cuando exista:

 broker tecnológico seleccionado;
 evaluación comparativa documentada;
 arquitectura de topics/streams;
 estrategia de productores;
 estrategia de consumidores;
 consumer groups;
 delivery semantics;
 retry policy;
 DLQ;
 retention;
 replay;
 security model;
 tenant isolation;
 encryption;
 credential management;
 partitioning;
 ordering;
 observability;
 capacity model;
 load testing;
 failure testing;
 disaster recovery;
 multi-region strategy;
 operational runbooks.
48. Siguiente ADR

El siguiente paso lógico es:

ADR-IDENTITY-019 — Event Partitioning, Ordering & Routing

Ahí vamos a bajar un nivel más en la arquitectura del broker y definir:

Event
  ↓
Routing
  ↓
Topic / Stream
  ↓
Partition
  ↓
Consumer Group
  ↓
Consumer

y, especialmente, qué significa "orden" en EVOXA.

Esto es importante porque ya tenemos definido que EVOXA no tendrá orden global, pero sí debemos establecer cómo garantizaremos el orden cuando sea necesario, por ejemplo:

User U1

UserCreated
     ↓
UserActivated
     ↓
UserSuspended

sin impedir que miles de usuarios puedan procesarse en paralelo.

Estado final: Proposed.
