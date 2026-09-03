ADR-IDENTITY-021 — Event Replay & Governance

Estado: Proposed
Versión: 1.0
Fecha: 2026-09-03
Dominio: Identity / Event Platform
Prioridad: Critical
Tipo: Event Architecture / Recovery / Governance / Reliability

1. Título

ADR-IDENTITY-021 — Event Replay & Governance

2. Contexto

EVOXA adopta una arquitectura Event-Driven Architecture (EDA) donde los eventos son activos arquitectónicos de primera clase: representan hechos de negocio, son inmutables, versionados, observables y trazables. La arquitectura establece además soporte para At-Least-Once delivery, Retry, Dead Letter Queue y Replay.

El Blueprint establece explícitamente que el replay puede utilizarse para:

recuperación;
analytics;
AI training/retraining;
debugging;
reconstrucción de datos;
nuevas proyecciones.

Y establece una regla fundamental:

Replay nunca debe modificar los eventos históricos ni producir resultados de negocio duplicados.

Además, los eventos deben conservar:

Event ID;
Event Type;
Version;
Timestamp;
Correlation ID;
Trace ID;
Producer;
Tenant ID;
User ID;
Schema Version.

El Event Governance también exige que cada evento tenga:

Owner;
Version;
Publisher;
Consumers;
Payload Schema;
Business Rules;
Retention Policy.

Por lo tanto, el replay no puede considerarse simplemente como "volver a enviar mensajes". Debe tratarse como una capacidad controlada de recuperación y reconstrucción de la plataforma.

3. Problema

En una plataforma como EVOXA existirán múltiples consumidores:

Identity
   │
   ├── Analytics
   ├── AI Platform
   ├── Notifications
   ├── Billing
   ├── Human Digital Twin
   ├── Mobile Platform
   ├── Audit Platform
   └── Reporting

El Blueprint contempla precisamente este modelo de consumidores desacoplados.

Un evento puede haber sido procesado correctamente por un consumidor y fallar en otro.

Por ejemplo:

UserCreated
     │
     ├── Analytics       ✓
     ├── Notifications  ✓
     ├── AI              ✗
     └── Reporting      ✓

El sistema necesita poder recuperar el procesamiento de AI sin repetir innecesariamente las operaciones exitosas de los demás consumidores.

También puede ocurrir:

Evento histórico
       ↓
Nuevo servicio
       ↓
Nueva proyección

o:

DLQ
 ↓
Investigación
 ↓
Corrección
 ↓
Replay

La arquitectura del Blueprint define precisamente el flujo:

DLQ
 ↓
Investigation
 ↓
Replay
 ↓
Resolution

Por ello se necesita una política formal que determine:

quién puede ejecutar replay;
qué eventos pueden reproducirse;
sobre qué consumidores;
con qué versión de schema;
en qué tenant;
desde qué punto;
hasta qué punto;
cómo evitar duplicados;
cómo preservar ordering;
cómo controlar efectos secundarios;
cómo auditar la operación;
cómo detener un replay peligroso.
4. Decisión

EVOXA adoptará un modelo de:

Controlled Event Replay with Explicit Governance

El replay será una operación controlada, auditable, autorizada, idempotente y reversible a nivel operacional cuando sea posible, pero nunca modificará los eventos históricos.

La arquitectura conceptual será:

Historical Events
       │
       ▼
Replay Request
       │
       ▼
Authorization
       │
       ▼
Replay Validation
       │
       ├── Schema
       ├── Tenant
       ├── Consumer
       ├── Ordering
       ├── Idempotency
       └── Security
       │
       ▼
Replay Job
       │
       ▼
Replay Stream
       │
       ▼
Consumer
       │
       ▼
Idempotent Processing
       │
       ▼
Observability + Audit
5. Replay no modifica eventos históricos

Esta será una regla absoluta.

Si existe:

WorkoutCompleted.v1
event_id = EVT-123

un replay no deberá modificar:

event_id
timestamp
payload
tenant_id
schema version
business meaning

El evento histórico permanece exactamente como fue registrado.

El replay genera una nueva operación de procesamiento, no un nuevo hecho histórico.

6. Replay no significa volver a producir el evento de negocio

Debe distinguirse:

Event
WorkoutCompleted
Replay Operation
Replay WorkoutCompleted

El segundo no significa que el usuario haya completado nuevamente el entrenamiento.

Significa:

"Procesar nuevamente el hecho histórico para recuperar o reconstruir una capacidad."

Esto evita confundir:

Historical Fact

con:

Processing Operation
7. Replay Use Cases

Se reconocen inicialmente los siguientes casos, derivados del Blueprint:

7.1 Recovery

Recuperar procesamiento fallido.

Consumer Failure
      ↓
DLQ
      ↓
Fix
      ↓
Replay
7.2 Analytics

Reconstruir métricas o proyecciones.

7.3 AI Training

Reprocesar información histórica para capacidades de AI.

El Blueprint identifica explícitamente AI training/retraining como uso válido del replay.

7.4 Debugging

Reproducir un flujo para investigar un comportamiento.

7.5 Data Reconstruction

Reconstruir una proyección o dataset.

7.6 New Projections

Crear una nueva proyección a partir del histórico sin modificar la fuente de eventos.

8. Tipos de Replay

Se establecerán conceptualmente diferentes modalidades.

8.1 Consumer Replay

Reprocesa eventos para un consumer específico.

Events
  ↓
Consumer A

Esto será preferido para recuperación selectiva.

8.2 DLQ Replay

Reprocesa únicamente eventos que terminaron en DLQ.

DLQ
 ↓
Validation
 ↓
Replay
8.3 Range Replay

Reprocesa un rango:

from_timestamp
to_timestamp
8.4 Aggregate Replay

Reprocesa eventos de un aggregate específico.

aggregate_id = X
8.5 Tenant Replay

Reprocesa eventos pertenecientes a un tenant específico.

tenant_id = T

Este tipo requiere autorización especialmente estricta.

8.6 Event-Type Replay

Reprocesa un tipo específico:

UserCreated
MembershipCreated
SessionCreated
8.7 Projection Replay

Reprocesa eventos para reconstruir una proyección determinada.

9. Replay por defecto debe ser Selectivo

No se deberá asumir:

Replay Everything

como operación normal.

La estrategia preferida será:

Replay
   ↓
Specific Consumer
   ↓
Specific Scope
   ↓
Specific Event Range

Esto reduce:

carga;
duplicados;
efectos secundarios;
riesgo operacional.
10. Replay Job

Toda ejecución significativa de replay deberá representarse conceptualmente como un Replay Job.

Ejemplo:

ReplayJob
---------
id
requested_by
tenant_scope
consumer_scope
event_type_scope
from
to
status
created_at
started_at
completed_at
processed_count
success_count
failure_count

Los campos exactos quedan sujetos a diseño de implementación.

11. Replay Job Lifecycle

El lifecycle propuesto será:

REQUESTED
    ↓
VALIDATING
    ↓
APPROVED
    ↓
RUNNING
    ↓
PAUSED ─────┐
    │       │
    ↓       │
COMPLETED   │
    │       │
    ↓       │
FAILED ←────┘

También deberá contemplarse:

CANCELLED

cuando el mecanismo lo permita.

12. Replay Authorization

Replay será una operación privilegiada.

No cualquier servicio o usuario podrá ejecutarlo.

El control deberá pasar por Identity:

Replay Request
      ↓
Authentication
      ↓
Tenant Context
      ↓
Authorization
      ↓
Security Policy
      ↓
Replay

Esto mantiene la regla arquitectónica de que las decisiones de autorización se centralizan en Identity y no deben duplicarse arbitrariamente en los dominios consumidores.

13. Replay Permissions

Se propone conceptualmente separar permisos como:

events.replay
events.replay.tenant
events.replay.consumer
events.replay.dlq
events.replay.admin

Estos nombres son propuesta de diseño, no nomenclatura normativa del Blueprint.

Deberán validarse antes de implementarlos.

14. Replay Tenant Isolation

Un operador autorizado para:

Tenant A

no deberá poder ejecutar:

Replay Tenant B

aunque tenga permisos generales de replay.

La evaluación deberá considerar:

actor
+
tenant
+
event
+
consumer
+
operation

Esto es especialmente importante porque Identity establece tenant isolation como una garantía fundamental.

15. Replay Security Policy

El Security Policy Engine podrá imponer controles adicionales.

Por ejemplo:

Replay sensitive event
       ↓
REAUTHENTICATE

o:

Replay security event
       ↓
REQUIRE_MFA

o:

Replay cross-tenant
       ↓
DENY

Las decisiones disponibles están alineadas con la estrategia de autorización definida previamente:

ALLOW
DENY
CHALLENGE
REAUTHENTICATE
REQUIRE_MFA
REVOKE
16. Replay de eventos sensibles

Los eventos de:

Authentication;
Authorization;
MFA;
Security;
Audit;

deberán recibir controles adicionales.

El Blueprint clasifica eventos por niveles:

Public
Internal
Confidential
Restricted

y establece que los campos sensibles deberán estar protegidos mediante encryption o tokenization.

Por lo tanto, replay no deberá convertirse en una vía para exponer información sensible.

17. Replay y AI

EVOXA tiene una arquitectura AI-Native.

Los eventos pueden alimentar:

AI Platform
Human Digital Twin
Analytics
Prediction
Recommendation

El replay hacia AI deberá mantener las mismas restricciones de:

tenant isolation;
authorization;
data minimization;
security classification;
governance.

Especialmente:

Replay no debe otorgar a AI acceso a información que el contexto autorizado original no permitía.

18. Replay y efectos secundarios

Este es uno de los puntos más importantes.

Un consumer puede producir efectos secundarios:

Event
 ↓
Send Email
 ↓
Charge
 ↓
Create Notification
 ↓
Update External System

Reprocesar indiscriminadamente podría producir:

Duplicate Email
Duplicate Payment
Duplicate Notification
Duplicate External Action

Por ello:

Los consumidores que producen efectos secundarios deberán implementar mecanismos explícitos de idempotencia o modos especiales de replay.

La arquitectura ya establece que los consumers deben tolerar duplicados y que los workflows críticos implementan deduplicación.

19. Replay Mode

Se propone distinguir conceptualmente:

NORMAL

Procesamiento normal.

mode = NORMAL
REPLAY

Procesamiento histórico controlado.

mode = REPLAY
RECOVERY

Procesamiento de recuperación.

mode = RECOVERY
REBUILD

Reconstrucción de proyecciones.

mode = REBUILD

La implementación final deberá determinar cuáles de estos modos serán necesarios.

20. Side-Effect Policy

Cada consumer deberá declarar conceptualmente:

side_effect_level

Ejemplo:

Nivel	Descripción
NONE	Solo lectura/proyección
INTERNAL	Modifica datos internos
EXTERNAL	Interactúa con sistemas externos
FINANCIAL	Produce efectos financieros
SECURITY	Produce acciones de seguridad

Los consumers con efectos:

EXTERNAL
FINANCIAL
SECURITY

requerirán políticas de replay más estrictas.

21. Replay Dry Run

Se deberá considerar un modo:

DRY_RUN

que permita evaluar:

cantidad de eventos;
consumers afectados;
schemas;
tenants;
posibles errores;

sin ejecutar efectos de negocio.

Conceptualmente:

Replay Request
      ↓
Dry Run
      ↓
Impact Report
      ↓
Approval
      ↓
Real Replay

Esta capacidad se considera recomendada para operaciones de alto riesgo, pero la implementación concreta queda pendiente.

22. Replay Approval

Los replay de alto impacto podrán requerir aprobación adicional.

Por ejemplo:

Small DLQ Replay
    ↓
Single Approval

versus:

Large Tenant Replay
    ↓
Security Review
    ↓
Architecture / Operations Approval

La matriz definitiva deberá ser definida posteriormente.

23. Replay Ordering

ADR-IDENTITY-019 establece que el ordering se garantiza dentro del scope definido, no globalmente.

Por lo tanto, replay deberá respetar el ordering requerido por:

aggregate;
user;
workflow;
u otro scope aprobado.

No deberá asumir:

Global Event Ordering

porque dicha garantía no existe en EVOXA.

24. Replay y Partitioning

Cuando los eventos estén particionados por routing key:

Aggregate A → Partition 1
Aggregate B → Partition 2

el replay deberá conservar el routing necesario para mantener el ordering requerido.

No deberá redistribuir arbitrariamente:

Aggregate A
v1
v2
v3

en diferentes secuencias que alteren el orden requerido.

25. Replay y Schema Version

ADR-IDENTITY-020 establece que los eventos son versionados y que los schemas históricos deben conservarse.

Por lo tanto:

Historical Event v1

no deberá reinterpretarse automáticamente como:

Current Event v3

sin una estrategia explícita.

El replay deberá conocer:

event_type
event_version
schema_version

y utilizar el mecanismo de compatibilidad o transformación definido.

26. Replay y Schema Adapter

Cuando sea necesario:

Event v1
   ↓
Schema Adapter
   ↓
Internal Model v3

Esto permitirá que consumidores modernos procesen eventos históricos.

La estrategia específica de transformación será refinada en conjunto con ADR-020 y la futura gobernanza de replay.

27. Replay y DLQ

El DLQ será una de las fuentes principales de replay.

Flujo:

Consumer
   ↓
Failure
   ↓
Retry
   ↓
Failure
   ↓
DLQ
   ↓
Investigation
   ↓
Correction
   ↓
Validation
   ↓
Replay

El Blueprint establece explícitamente este patrón.

No deberá hacerse replay automático de una DLQ sin conocer la causa del fallo.

28. DLQ Replay Safety

Antes de reprocesar un evento DLQ deberá evaluarse:

causa del error;
schema;
versión;
consumer;
tenant;
dependencia;
idempotencia;
side effects.

Ejemplo:

Bug en Consumer
      ↓
Fix
      ↓
Deploy
      ↓
Contract Test
      ↓
Replay DLQ
29. Replay Checkpoint

Los replay grandes deberán permitir checkpoints conceptuales.

Ejemplo:

100,000 events
       ↓
Checkpoint 20,000
       ↓
Checkpoint 40,000
       ↓
Checkpoint 60,000

Si el worker falla:

Restart
   ↓
Last safe checkpoint

Esto reduce el coste de recuperación.

La estrategia física de checkpointing queda pendiente.

30. Replay Pause / Resume

Los replay largos deberán poder:

PAUSE
RESUME
CANCEL

cuando la tecnología y el tipo de replay lo permitan.

Ejemplo:

Replay
  ↓
High consumer lag
  ↓
PAUSE
  ↓
System recovers
  ↓
RESUME
31. Backpressure

El replay no deberá saturar los consumers normales.

Debe existir control de:

throughput;
concurrency;
consumer lag;
CPU;
database load;
broker capacity.

Por ejemplo:

Replay
   ↓
Rate Limit
   ↓
Consumer

La capacidad objetivo definida por el Blueprint para replay es:

>10.000 eventos/minuto, aunque esta métrica deberá validarse en pruebas reales y según el tipo de evento.

32. Replay Priority

Se podrá definir conceptualmente:

NORMAL
HIGH
CRITICAL

pero un replay nunca deberá desplazar indefinidamente los workflows productivos críticos.

La prioridad final dependerá del Event Platform.

33. Replay Observability

Cada replay deberá ser completamente observable.

Deberá poder conocerse:

Replay ID
Requested By
Tenant
Consumer
Event Type
Start
End
Processed
Succeeded
Failed
Retried
Skipped
DLQ

Además deberán mantenerse:

correlation ID;
trace ID;
processing metrics.

El Blueprint establece que cada evento debe ser trazable mediante correlation ID, trace ID, distributed tracing, event logs y processing metrics.

34. Replay Audit

Cada operación de replay deberá generar auditoría.

Ejemplo:

ReplayRequested
ReplayApproved
ReplayStarted
ReplayPaused
ReplayResumed
ReplayCompleted
ReplayFailed
ReplayCancelled

La auditoría deberá registrar:

actor
tenant
scope
reason
result
correlation_id
trace_id

Nunca deberá registrar:

tokens;
passwords;
MFA secrets;
private keys;
recovery codes;
payload sensible innecesario.
35. Replay Reason

Toda operación manual deberá tener un motivo.

Ejemplo:

reason:
"Recover failed Identity consumers after deployment incident"

Esto mejora:

auditoría;
incident response;
compliance;
debugging.
36. Replay Idempotency

Replay dependerá directamente de Consumer Idempotency.

Conceptualmente:

event_id
     ↓
Deduplication
     ↓
Already processed?
   /       \
 YES       NO
 ↓          ↓
Skip      Process

Por lo tanto:

Replay no sustituye la idempotencia.

Replay y Consumer Idempotency son capacidades complementarias.

Esto se alinea con el Blueprint, que exige consumers capaces de tolerar duplicados.

37. Replay y At-Least-Once

La arquitectura mantiene:

At-Least-Once

Por lo tanto, el sistema no debe asumir que replay produce exactamente una ejecución física.

Puede existir:

Event
 ↓
Replay
 ↓
Retry
 ↓
Duplicate delivery

La garantía debe provenir de:

Idempotency
+
Deduplication
+
Correct Consumer Design
38. Replay Isolation

Los replay grandes deberán aislarse operacionalmente cuando sea necesario.

Ejemplo:

Production Consumers
        ↑
        │
   Controlled Replay
        │
        ↓
Replay Workers

Esto permite controlar:

recursos;
throughput;
prioridades;
seguridad.
39. Replay Governance Registry

Cada evento deberá tener metadata de replay.

El Event Blueprint Object (EBO) ya exige:

Event ID;
Name;
Category;
Version;
Producer;
Consumers;
Payload Schema;
Trigger;
Correlation ID;
Trace ID;
Retention Policy;
Security Classification;
Business Owner.

Se propone extender conceptualmente el gobierno con:

Replay Policy
Replay Supported
Replay Scope
Side Effect Policy
Replay Owner

La incorporación definitiva de estos campos deberá validarse contra el modelo EBO.

40. Replay Policy por Evento

Cada evento deberá declarar conceptualmente una política:

Replay:
  ALLOWED
  RESTRICTED
  PROHIBITED

Ejemplo:

Analytics Event
→ ALLOWED

Security Event
→ RESTRICTED

Financial External Action
→ RESTRICTED / SPECIAL MODE

La clasificación concreta deberá definirse por catálogo.

41. Replay Prohibited

Algunos eventos podrán tener:

Replay = PROHIBITED

cuando reproducirlos pueda causar consecuencias irreversibles o inseguras.

Esto no implica borrar el evento.

Simplemente significa:

Historical Event
       ≠
Replayable Event
42. Replay Governance para AI

Para eventos destinados a AI deberá existir una revisión adicional cuando el replay:

reconstruya datasets;
genere nuevos embeddings;
alimente entrenamiento;
modifique modelos;
produzca recomendaciones;
reconstruya Human Digital Twin.

El replay de datos históricos no debe considerarse automáticamente equivalente a autorización para reutilizar esos datos con cualquier propósito de AI.

43. Cross-Tenant Replay

Por defecto:

Cross-Tenant Replay = DENY

Una operación multi-tenant deberá ser una capacidad explícita de plataforma y estar sujeta a:

autorización;
seguridad;
audit;
justificación;
políticas específicas.
44. Global Replay

El:

Replay Everything

deberá considerarse una operación excepcional.

Requerirá como mínimo:

scope explícito;
autorización elevada;
observabilidad;
límites;
validación;
protección contra side effects;
audit.
45. Replay Disaster Recovery

Replay será una herramienta de recuperación, pero no sustituirá:

backups;
restore;
disaster recovery;
infraestructura redundante.

El Blueprint contempla replay como mecanismo de recuperación, pero también establece objetivos de resiliencia y recuperación.

Por lo tanto:

Backup/Restore
      +
Event Replay

son mecanismos complementarios.

46. Event Sourcing

El Blueprint indica que ciertos aggregates pueden soportar Event Sourcing, incluyendo ejemplos como:

Human Digital Twin;
Assessments;
Programs.

Los eventos permanecen inmutables.

Cuando un aggregate sea efectivamente event-sourced:

Events
   ↓
Replay
   ↓
Aggregate State

podrá utilizarse para reconstrucción.

Pero:

No todos los dominios deben interpretarse automáticamente como Event Sourced.

Replay debe funcionar tanto para event sourcing selectivo como para reconstrucción de proyecciones.

47. Replay Data Reconstruction

Para reconstruir una proyección:

Historical Events
       ↓
Replay
       ↓
Projection Consumer
       ↓
New Projection

La proyección podrá eliminarse/reconstruirse sin modificar los eventos fuente.

Este patrón permite:

corregir proyecciones;
crear nuevas vistas;
migrar modelos;
recuperar índices.
48. Replay y Retention

Replay depende directamente de la retención.

Si los eventos ya no están disponibles:

Retention expired
       ↓
Replay unavailable

Por ello, cada evento debe tener una Retention Policy, tal como exige el gobierno de eventos.

La política de retención específica no se definirá en este ADR.

49. Archived Events

Los eventos archivados podrán seguir siendo fuente de replay cuando la política lo permita.

Conceptualmente:

Active Event Store
       ↓
Archive
       ↓
Replay Source

Esto permite desacoplar:

Hot operational storage

de:

Historical replay storage
50. Replay desde Archive

El replay desde archive deberá considerar:

schema availability;
integrity;
tenant metadata;
event ordering;
event version;
security classification;
retention;
access authorization.

No se deberá asumir que un archivo histórico es automáticamente replayable.

51. Integrity

Antes del replay deberá verificarse, cuando corresponda:

Event Integrity
Schema Validity
Tenant Context

Un evento corrupto o no verificable no deberá introducirse nuevamente al pipeline.

52. Replay Tampering Protection

Los eventos históricos deberán permanecer protegidos contra modificaciones.

Si se detecta:

Historical Event Changed

deberá tratarse como incidente de integridad, no como una operación normal de replay.

53. Replay Concurrency

Múltiples replay jobs podrían ejecutarse simultáneamente:

Replay A → Analytics
Replay B → AI
Replay C → DLQ

La plataforma deberá evitar que estos trabajos interfieran indebidamente.

Se deberán controlar:

resource limits;
consumer concurrency;
duplicate jobs;
overlapping ranges;
ordering.
54. Duplicate Replay Jobs

Debe detectarse o controlarse un caso como:

Replay Job A
UserCreated
Tenant A
01:00-02:00

Replay Job B
UserCreated
Tenant A
01:00-02:00

El sistema podrá:

impedir duplicado;
permitirlo con idempotencia;
advertir;
requerir aprobación.

La estrategia definitiva queda pendiente.

55. Replay Cancellation

Cancelar un replay no significa eliminar los eventos.

Significa:

Replay Job
   ↓
CANCEL

Los eventos históricos permanecen intactos.

Los eventos ya procesados tampoco deben asumirse automáticamente reversibles.

56. Replay Failure

Si un replay falla:

Replay
 ↓
Failure
 ↓
Observability
 ↓
Investigation

no se deberán modificar los eventos fuente.

El job podrá quedar:

FAILED

y posteriormente continuar mediante una estrategia de retry/resume/replay específica.

57. Replay Error Classification

Los errores deberán distinguirse conceptualmente:

Schema Error
Invalid event/schema
Authorization Error
Replay not permitted
Tenant Error
Invalid tenant scope
Consumer Error
Consumer processing failed
Infrastructure Error
Broker / DB / worker unavailable
Business Error
Consumer business rule rejected event

Esto facilitará troubleshooting.

58. Replay Metrics

La plataforma deberá medir:

replay_jobs_total
replay_events_total
replay_success_total
replay_failure_total
replay_skipped_total
replay_duration
replay_throughput
replay_lag
replay_dlq_total

Además:

consumer_processing_time
retry_count

alineado con la observabilidad definida para eventos.

59. Replay Audit Trail

El historial deberá permitir responder:

Who?
What?
When?
Why?
Which tenant?
Which consumer?
Which events?
Which version?
What happened?

Esto será especialmente importante para:

seguridad;
compliance;
incident response;
AI governance.
60. Governance Workflow

El workflow general será:

Replay Proposal
      ↓
Scope Definition
      ↓
Impact Analysis
      ↓
Authorization
      ↓
Security Policy
      ↓
Schema Validation
      ↓
Idempotency Validation
      ↓
Dry Run / Approval
      ↓
Replay Execution
      ↓
Monitoring
      ↓
Validation
      ↓
Audit
61. Repository / Contract Organization

El Blueprint define una estructura de contratos:

contracts/
└── events/
    ├── EVT-TRN-002-workout-completed/
    │   ├── README.md
    │   ├── metadata.yml
    │   ├── event.schema.json
    │   ├── asyncapi.yaml
    │   ├── payload-example.json
    │   ├── producers.yml
    │   ├── consumers.yml
    │   ├── routing.yml
    │   ├── retry-policy.yml
    │   ├── retention.yml
    │   ├── telemetry.yml
    │   ├── contract-tests.yml
    │   └── changelog.md

La política de replay deberá integrarse en esta gobernanza de contratos, aunque el nombre exacto del archivo o estructura adicional queda pendiente.

62. Event Governance Board

La arquitectura establece que cada evento requiere:

Architecture approval;
Schema validation;
Consumer documentation;
Versioning strategy;
Security review;
Automated contract tests.

El replay deberá quedar bajo la misma gobernanza.

En particular, los eventos críticos deberán tener una política de replay definida antes de ser publicados.

63. Consecuencias positivas
Recuperación

Permite recuperar consumidores fallidos sin modificar datos históricos.

Resiliencia

Mejora la capacidad de recuperación de la plataforma.

Evolución

Permite crear nuevas proyecciones a partir del histórico.

Analytics

Permite reconstrucción de datos analíticos.

AI

Permite procesamiento histórico controlado para AI training/retraining.

Debugging

Permite reproducir comportamientos históricos.

Governance

Cada replay puede ser autorizado, auditado y observado.

Seguridad

Tenant isolation y authorization se mantienen durante el replay.

64. Consecuencias negativas

Esta estrategia introduce:

Replay Jobs;
governance;
permisos;
auditoría;
controles de idempotencia;
gestión de schemas históricos;
mecanismos de throttling;
observabilidad adicional;
complejidad operacional.

Sin estos controles, el replay podría convertirse en una fuente de:

duplicados;
efectos secundarios;
sobrecarga;
violaciones de tenant isolation.

Por ello la complejidad se considera justificada.

65. Alternativas consideradas
Alternativa A — No soportar Replay

Rechazada.

Contradice explícitamente la arquitectura del Blueprint.

Alternativa B — Replay manual desde el Broker

Rechazada como estrategia general.

No proporciona governance suficiente.

Alternativa C — Replay de todos los consumers

Rechazada como comportamiento por defecto.

Aumenta innecesariamente el riesgo.

Alternativa D — Modificar eventos históricos para facilitar replay

Rechazada.

Los eventos son hechos inmutables.

Alternativa E — Replay selectivo y gobernado

Seleccionada.

Permite:

recovery;
projection rebuild;
analytics;
AI;
debugging;

manteniendo control operativo.

66. Non-Negotiables
Los eventos históricos son inmutables.
Replay nunca modifica el evento fuente.
Replay no representa un nuevo hecho de negocio.
Replay debe ser autorizado.
Tenant isolation debe mantenerse.
Los consumers deben ser idempotentes.
Los eventos versionados deben conservar su schema histórico.
Breaking changes requieren nueva versión.
DLQ puede ser fuente de replay.
Replay debe ser observable.
Replay debe ser auditable.
Replay no debe producir duplicados de negocio.
Los efectos secundarios deben tener política explícita.
No existe global ordering.
Replay debe respetar el ordering scope aplicable.
Replay no debe saturar consumers productivos.
Replay debe considerar retention.
Eventos sensibles requieren controles adicionales.
AI replay debe respetar autorización y tenant isolation.
Cross-tenant replay no debe permitirse por defecto.
Replay no reemplaza backup/restore.
Los schemas históricos necesarios para replay deben preservarse.
No se debe ejecutar un replay masivo sin scope y autorización explícitos.
No se debe usar replay para ocultar o corregir un evento histórico.
67. Traceability
Blueprint — Event Architecture

El Blueprint establece:

eventos inmutables;
eventos versionados;
At-Least-Once;
Retry;
DLQ;
Replay;
consumidores idempotentes.

También establece los casos de uso:

Recovery;
Analytics;
AI Training;
Debugging;
Data Reconstruction.
Event Governance

Cada evento debe definir owner, version, publisher, consumers, payload schema, business rules y retention policy.

Event Contracts

La estructura de contratos contempla schemas, producers, consumers, routing, retry, retention, telemetry y contract tests.

68. Relación con Identity

Este ADR afecta directamente:

Authentication;
Authorization;
Sessions;
Refresh Tokens;
MFA;
Security Policies;
Audit;
Security Events;
Domain Events;
Outbox.

Especialmente:

Identity Event
      ↓
Outbox
      ↓
Broker
      ↓
Consumer
      ↓
Replay
69. ADR relacionados
ADR-IDENTITY-007 — Event Architecture
ADR-IDENTITY-013 — Data Retention
ADR-IDENTITY-014 — Encryption & Secret Protection
ADR-IDENTITY-015 — Event Partitioning
ADR-IDENTITY-016 — Concurrency & Transaction Integrity
ADR-IDENTITY-018 — Event Broker Strategy
ADR-IDENTITY-019 — Event Partitioning / Ordering / Routing
ADR-IDENTITY-020 — Event Schema Registry & Compatibility
ADR-IDENTITY-022 — Consumer Idempotency
ADR-IDENTITY-028 — Observability / Operational Security
ADR-IDENTITY-029 — Disaster Recovery / Backup
70. Dependencias

La relación queda:

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

Y transversalmente:

ADR-013
Retention
       │
       ├──────► ADR-021
       │
ADR-014
Security
       │
       ├──────► ADR-021
       │
ADR-016
Concurrency
       │
       └──────► ADR-021
71. Decisiones pendientes

Este ADR no fija todavía:

Replay Storage
broker retention;
event store;
archive;
object storage;
combinación.
Replay Engine
broker-native;
dedicated replay workers;
event processing service.
Checkpointing
offset;
event ID;
timestamp;
batch checkpoint.
Rate Limiting
events/sec;
events/min;
adaptive throttling.
Approval
cuándo requiere aprobación;
quién puede aprobar;
doble control para operaciones críticas.
Dry Run
obligatorio;
opcional;
solo para determinados scopes.
Side Effects
política exacta por consumer;
replay-safe mode;
suppression de acciones externas.
Cross-Tenant
si existirá alguna capacidad administrativa multi-tenant.
Retention
períodos;
archive tiers;
restore strategy.
Replay Transformation
adapters;
schema migration;
version-specific consumers.
72. Criterios de aceptación

ADR-IDENTITY-021 podrá pasar de Proposed → Accepted cuando se encuentre definido:

 Replay Job model.
 Replay lifecycle.
 Replay authorization.
 Tenant isolation.
 Replay permissions.
 Replay policy por evento.
 DLQ replay strategy.
 Consumer replay strategy.
 Schema compatibility.
 Historical schema retention.
 Ordering during replay.
 Checkpoint strategy.
 Pause/resume strategy.
 Cancellation strategy.
 Rate limiting.
 Backpressure.
 Side-effect policy.
 Idempotency integration.
 Dry-run strategy.
 Approval workflow.
 Audit events.
 Metrics.
 Tracing.
 Security controls.
 AI replay governance.
 Archive replay.
 Disaster recovery integration.
 Operational runbooks.
73. Resultado arquitectónico

Con este ADR queda establecida una regla muy importante para EVOXA:

             EVENTO HISTÓRICO
                    │
                    │ INMUTABLE
                    ▼
             ┌───────────────┐
             │ Replay Engine │
             └───────┬───────┘
                     │
          ┌──────────┼──────────┐
          ▼          ▼          ▼
       Recovery   Analytics     AI
          │          │          │
          └──────────┼──────────┘
                     ▼
              Idempotent Consumer
                     │
                     ▼
               Audit + Trace

El principio fundamental queda:

Los eventos representan hechos históricos inmutables; el replay representa una operación controlada para volver a procesar esos hechos, nunca para modificarlos ni convertirlos nuevamente en hechos de negocio.

Esto mantiene coherencia con el Blueprint, que considera el replay una capacidad fundamental para recovery, analytics, AI training, debugging y data reconstruction, manteniendo simultáneamente la inmutabilidad de los eventos.

Estado final: Proposed.

Siguiente paso lógico

ADR-IDENTITY-022 — Consumer Idempotency & Deduplication

Este será especialmente importante porque completa el mecanismo:

018 Event Broker
       ↓
019 Partitioning / Ordering / Routing
       ↓
020 Schema Registry / Compatibility
       ↓
021 Event Replay / Governance
       ↓
022 Consumer Idempotency / Deduplication

Y en el ADR-022 podremos definir exactamente cómo EVOXA garantiza que At-Least-Once + Retry + DLQ + Replay nunca se conviertan en duplicación de resultados de negocio.
