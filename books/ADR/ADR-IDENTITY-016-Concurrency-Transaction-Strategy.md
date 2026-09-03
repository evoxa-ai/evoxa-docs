ADR-IDENTITY-016 — Concurrency & Transaction Strategy

Estado: Proposed
Versión: 1.0
Fecha: 2026-09-02
Dominio: Identity
Prioridad: Critical
Tipo: Data Architecture / Transaction Management / Security / Reliability

1. Título

ADR-IDENTITY-016 — Concurrency & Transaction Strategy

2. Contexto

El dominio Identity contiene operaciones altamente sensibles a problemas de concurrencia.

Entre ellas:

creación y actualización de usuarios;
creación de memberships;
asignación de roles;
asignación de permisos;
creación y revocación de sesiones;
rotación de Refresh Tokens;
detección de reuse de tokens;
MFA challenges;
cambios de Security Policies;
generación de Audit Events;
escritura del Transactional Outbox.

El Blueprint establece una arquitectura:

Domain-Driven;
API-First;
Event-Driven;
PostgreSQL como fuente transaccional;
Redis como infraestructura auxiliar;
Transactional Outbox;
eventos at-least-once;
consumidores idempotentes;
retries;
DLQ;
replay;
seguridad fail-closed.

Por lo tanto, las operaciones críticas no pueden depender únicamente de que las solicitudes lleguen de forma secuencial.

Dos o más solicitudes pueden intentar modificar simultáneamente el mismo recurso.

3. Problema

Sin una estrategia explícita de concurrencia podrían producirse situaciones como:

Request A ─────┐
               ├── Refresh Token
Request B ─────┘

donde ambas solicitudes intentan utilizar el mismo Refresh Token.

O:

Request A ── Assign Role
Request B ── Assign Role

simultáneamente.

O:

Transaction A ── Domain State
Transaction B ── Domain State

produciendo pérdida de actualizaciones, estados inconsistentes o eventos duplicados.

Los riesgos principales son:

lost updates;
double processing;
duplicate assignments;
token replay;
race conditions;
inconsistent audit;
inconsistent outbox;
deadlocks;
dirty/inconsistent reads;
doble creación de recursos;
escalamiento accidental de privilegios.

Por ello, Identity necesita una estrategia explícita para transacciones y concurrencia.

4. Decisión

EVOXA adoptará una estrategia de transacciones ACID en PostgreSQL combinada con controles explícitos de concurrencia, idempotencia y Transactional Outbox.

La estrategia seguirá estos principios:

Domain Operation
      ↓
Application Service
      ↓
Transaction Boundary
      ↓
PostgreSQL
      ├── Domain State
      └── Outbox Event
      ↓
COMMIT
      ↓
Event Platform

Las operaciones críticas deberán definir explícitamente:

límite transaccional;
nivel de aislamiento requerido;
estrategia de locking;
idempotencia;
comportamiento ante conflicto;
retry policy;
manejo de deadlocks;
publicación de eventos.
5. Principio fundamental

Toda operación que cambie estado crítico de Identity debe tener una frontera transaccional explícita.

No se permitirá implementar operaciones críticas como una secuencia arbitraria de:

SELECT
UPDATE
INSERT
UPDATE

sin determinar cómo se comportará el sistema cuando otra transacción ejecute las mismas operaciones simultáneamente.

6. ACID

PostgreSQL será responsable de proporcionar las garantías transaccionales para el estado persistente.

La estrategia utilizará:

Atomicity

Una operación debe completarse completamente o no producir cambios parciales.

Consistency

Las constraints y reglas del dominio deben mantenerse.

Isolation

Las operaciones concurrentes no deben producir estados inválidos.

Durability

Una vez confirmado el commit, el estado debe persistir de acuerdo con las garantías de PostgreSQL y la infraestructura.

7. Transaction Boundary

La frontera transaccional estará principalmente en la capa Application.

Conceptualmente:

Presentation
     ↓
API
     ↓
Application Service
     │
     ├── BEGIN
     │
     ├── Domain Operation
     │
     ├── Repository Operations
     │
     ├── Audit / Outbox
     │
     └── COMMIT
     ↓
Infrastructure

El dominio no deberá depender directamente de mecanismos específicos de PostgreSQL.

8. Domain State + Outbox

Una de las decisiones más importantes es que el cambio de estado y su correspondiente evento Outbox deberán confirmarse dentro de la misma transacción cuando corresponda.

Ejemplo:

BEGIN
   │
   ├── UPDATE membership
   │
   ├── INSERT audit/event
   │
   ├── INSERT outbox_event
   │
COMMIT

No se deberá realizar:

UPDATE database
COMMIT

publish event

como dos operaciones independientes para las cuales una pueda tener éxito y la otra fallar.

Esto evita el problema:

Database = SUCCESS
Event    = FAILURE

sin mecanismo de recuperación.

Esta decisión deriva directamente de:

ADR-IDENTITY-007 — Event Architecture

e

IS-IDENTITY-013 — Domain Events & Outbox.

9. Isolation Level

PostgreSQL será utilizado con un nivel de aislamiento apropiado para cada operación.

No se establece en este ADR que todas las operaciones deban utilizar el nivel máximo de aislamiento.

La elección dependerá de:

tipo de operación;
riesgo;
volumen;
duración;
necesidad de consistencia;
posibilidad de retry.

La estrategia deberá evitar elevar innecesariamente el aislamiento cuando una solución más simple sea suficiente.

10. Read Operations

Las consultas puramente de lectura no deberán abrir transacciones largas innecesariamente.

El sistema deberá evitar:

BEGIN
   SELECT ...
   espera larga
COMMIT

cuando no exista una necesidad de consistencia transaccional.

Las lecturas de autorización, por ejemplo, deberán diseñarse para minimizar:

locks;
latencia;
contention.
11. Optimistic Concurrency

Para determinadas entidades de actualización frecuente se podrá utilizar optimistic concurrency control.

Conceptualmente:

Entity
version = 10

Request A:

version 10
    ↓
update
    ↓
version 11

Request B:

version 10
    ↓
update
    ↓
CONFLICT

En ese caso, Request B no deberá sobrescribir silenciosamente el cambio realizado por A.

El mecanismo exacto —campo version, timestamp o estrategia equivalente— deberá definirse por entidad cuando corresponda.

12. Pessimistic Locking

Para operaciones donde exista un riesgo alto de doble procesamiento se podrá utilizar locking pesimista.

Ejemplo conceptual:

SELECT ...
FOR UPDATE

El objetivo es serializar operaciones sobre el recurso específico.

Esto será especialmente relevante para operaciones como:

Refresh Token rotation;
token family;
estados sensibles;
operaciones críticas de seguridad.

No se deberá utilizar locking pesimista indiscriminadamente.

13. Refresh Token Rotation

La rotación de Refresh Tokens constituye uno de los casos más importantes de concurrencia.

Ejemplo:

Token T1
   │
   ├── Request A
   └── Request B

Ambas solicitudes podrían llegar prácticamente al mismo tiempo.

La estrategia deberá garantizar que no puedan producirse dos rotaciones válidas independientes del mismo token.

Conceptualmente:

BEGIN
   │
   ├── Lock / validate T1
   │
   ├── Verify status
   │
   ├── Mark T1 USED
   │
   ├── Create T2
   │
   └── COMMIT

La segunda solicitud deberá detectar que el estado de T1 ya cambió.

El comportamiento exacto ante:

retry legítimo;
duplicate request;
replay;
reuse malicioso;

será definido en:

ADR-IDENTITY-025 — Refresh Token Cryptography & Concurrency.

14. Token Reuse

Si un Refresh Token previamente utilizado vuelve a presentarse, el sistema deberá distinguir el escenario según la política definida.

Conceptualmente:

T1 ACTIVE
   ↓
T1 USED
   ↓
T2 ACTIVE

Si vuelve a aparecer:

T1 USED
   ↓
Reuse Detection

la respuesta de seguridad podrá implicar:

Revoke Token Family
       +
Revoke Session
       +
Security Event
       +
Audit Event

La decisión detallada pertenece al ADR-025.

15. Unique Constraints

Las constraints de base de datos serán una línea adicional de defensa contra concurrencia.

Ejemplos:

memberships
UNIQUE(user_id, organization_id)
role_permissions
UNIQUE(role_id, permission_id)
membership_roles
UNIQUE(membership_id, role_id)

Esto garantiza que incluso si dos solicitudes intentan realizar la misma operación simultáneamente, la base de datos mantenga la integridad.

16. Idempotency

Las operaciones retryable deberán diseñarse para tolerar repetición.

Esto es especialmente importante porque la arquitectura de EVOXA contempla:

retries;
at-least-once events;
distributed systems;
network failures.

Ejemplo:

POST operation
       ↓
Request A
       ↓
Database COMMIT
       ↓
Network timeout
       ↓
Client retries
       ↓
Request B

El sistema no debe asumir que:

timeout = operation failed

Por ello, las operaciones apropiadas deberán soportar Idempotency-Key u otro mecanismo equivalente.

La definición completa de la API de idempotencia queda vinculada a:

IS-IDENTITY-014 — Identity API Contracts

y al ADR correspondiente de API.

17. Idempotency Store

Cuando se utilice Idempotency-Key, deberá existir una estrategia para almacenar y consultar el resultado de una operación idempotente.

Conceptualmente:

Idempotency-Key
       ↓
Existing?
   ┌───┴────┐
   │        │
 YES       NO
   │        │
return    execute
result      │
            ↓
          store

La estrategia concreta de:

almacenamiento;
TTL;
scope;
tenant;
fingerprint;
resultado;
concurrencia;

queda pendiente del ADR/API correspondiente.

18. Race Conditions en Roles y Permisos

Las operaciones de autorización requieren especial atención.

Ejemplo:

Request A:
Assign admin role

Request B:
Revoke admin role

El sistema debe garantizar que el resultado final sea determinista y auditable.

Las operaciones deberán considerar:

estado actual;
versión;
locks cuando sean necesarios;
constraints;
eventos;
audit.

No se deberá asumir que el orden de llegada HTTP representa necesariamente el orden definitivo de negocio.

19. Race Conditions en Membership

Operaciones simultáneas:

Activate Membership
Suspend Membership
Remove Membership

deberán respetar las transiciones de lifecycle definidas por el dominio.

Por ejemplo, no se debe permitir que una condición de carrera produzca:

REMOVED → ACTIVE

si esa transición no está permitida.

La regla debe residir en el dominio y la persistencia debe impedir estados inconsistentes.

20. User Lifecycle

El mismo principio se aplica a:

PENDING
ACTIVE
SUSPENDED
DISABLED

Si dos operadores intentan cambiar simultáneamente el estado de un usuario:

A → SUSPEND
B → ACTIVATE

la plataforma deberá:

serializar cuando corresponda;
detectar conflicto;
respetar las reglas de lifecycle;
generar auditoría;
evitar pérdida silenciosa de cambios.
21. Organization Lifecycle

Una organización también puede recibir operaciones concurrentes:

Suspend Organization
Reactivate Organization
Update Organization

Las operaciones deberán garantizar que:

no exista estado imposible;
los cambios sean auditables;
el estado final sea consistente;
los consumidores reciban los eventos correspondientes.
22. Session Revocation

Dos solicitudes podrían intentar:

Revoke Session
Revoke All Sessions

simultáneamente.

La operación deberá ser segura aunque ambas lleguen al mismo tiempo.

Ejemplo:

Session ACTIVE
      │
      ├── Request A → REVOKE
      │
      └── Request B → REVOKE

La segunda operación debe ser idempotente o manejar explícitamente que la sesión ya está revocada.

23. MFA Challenges

Los MFA challenges también requieren control de concurrencia.

Ejemplo:

Challenge C1
     │
     ├── Verify Request A
     └── Verify Request B

No debería ser posible utilizar exitosamente el mismo challenge más veces de las permitidas.

El estado deberá evolucionar de forma controlada:

PENDING
   ↓
VERIFIED

y una segunda verificación deberá reconocer que el challenge ya fue consumido.

24. Recovery Codes

Los Recovery Codes presentan un patrón similar.

Dos solicitudes concurrentes no deberán poder utilizar exitosamente el mismo código de recuperación.

Conceptualmente:

Code ACTIVE
    │
    ├── Request A → USE
    └── Request B → USE

solo una deberá conseguir el consumo válido.

25. Deadlocks

El sistema deberá contemplar la posibilidad de deadlocks.

Ejemplo conceptual:

Transaction A
    locks User
       ↓
    waits Role

Transaction B
    locks Role
       ↓
    waits User

Esto puede producir:

DEADLOCK

La estrategia deberá:

mantener transacciones cortas;
establecer orden consistente de adquisición de locks;
evitar locks innecesarios;
detectar errores transitorios;
permitir retry seguro cuando corresponda.
26. Retry Strategy

No todos los errores transaccionales deben reintentarse.

Se distinguirá conceptualmente:

Transient
   ↓
Potential Retry

Business Conflict
   ↓
Do Not Blind Retry

Security Violation
   ↓
Do Not Retry

Validation Error
   ↓
Do Not Retry

Un retry solamente será seguro cuando la operación sea:

idempotente;
transaccionalmente segura;
compatible con el estado actual.
27. Transaction Duration

Las transacciones deberán mantenerse lo más cortas posible.

No deberán incluir operaciones externas lentas dentro de una transacción PostgreSQL, por ejemplo:

BEGIN
   ↓
Database
   ↓
HTTP external service
   ↓
Wait 5 seconds
   ↓
Database
   ↓
COMMIT

Esto incrementaría:

locks;
contention;
conexiones ocupadas;
deadlocks;
latencia.

Las comunicaciones externas deberán ejecutarse fuera de la transacción cuando la arquitectura lo permita.

28. Event Publishing

No se deberá mantener una transacción abierta esperando al Event Platform.

Incorrecto:

BEGIN
   ↓
DB update
   ↓
Publish event
   ↓
Wait broker
   ↓
COMMIT

Preferido:

BEGIN
   ↓
DB update
   ↓
Outbox insert
   ↓
COMMIT
   ↓
Outbox Publisher
   ↓
Event Platform

Esto mantiene desacopladas las garantías transaccionales de la disponibilidad del broker.

29. Audit Events

Cuando una operación requiera auditoría obligatoria, el evento de auditoría deberá formar parte de la estrategia transaccional apropiada.

Por ejemplo:

BEGIN
   │
   ├── Change Role
   ├── Audit Event
   └── Outbox Event
   │
COMMIT

De esta forma se evita:

Business Change = SUCCESS
Audit = LOST

cuando el audit sea obligatorio.

La implementación concreta se alinea con:

IS-IDENTITY-012

e

IS-IDENTITY-013.

30. Redis y Concurrency

Redis no será considerado la fuente transaccional principal de Identity.

Por lo tanto, Redis:

puede utilizarse para cache;
rate limiting;
información temporal;
coordinación cuando esté explícitamente diseñada;

pero una caída de Redis no deberá provocar una degradación de seguridad que permita:

bypass de autorización;
bypass de MFA;
bypass de tenant isolation;
aceptación indebida de tokens.

La autoridad final para el estado crítico permanece en PostgreSQL.

31. Distributed Locks

Los distributed locks no serán la primera solución para todos los problemas de concurrencia.

Se priorizará:

constraints;
transacciones;
atomic updates;
optimistic concurrency;
PostgreSQL locking;
idempotencia.

Un distributed lock solamente se utilizará cuando exista una necesidad real que no pueda resolverse adecuadamente mediante los mecanismos anteriores.

32. Atomic State Transitions

Cuando sea posible, las transiciones críticas deberán expresarse como operaciones atómicas.

Ejemplo conceptual:

UPDATE refresh_tokens
SET status = USED
WHERE id = X
AND status = ACTIVE

El resultado deberá indicar si realmente se realizó la transición.

Conceptualmente:

affected_rows = 1
    → success

affected_rows = 0
    → already changed / conflict

Este patrón reduce race conditions y evita depender únicamente de:

SELECT status
UPDATE status

como dos operaciones separadas.

33. Concurrency y Tenant Isolation

Toda operación concurrente deberá mantener las garantías de tenant isolation.

Nunca debe ocurrir:

Tenant A transaction
       ↓
Resource Tenant B
       ↓
UPDATE

El tenant context deberá formar parte de la validación de la operación.

Para recursos tenant-scoped, las operaciones críticas deberán verificar:

tenant_id;
membership;
authorization;
resource ownership cuando corresponda.
34. Transaction Context

Conceptualmente, la aplicación manejará:

TransactionContext
├── transaction_id
├── correlation_id
├── actor
├── tenant_id
└── request_context

Esto facilitará:

auditoría;
trazabilidad;
debugging;
observabilidad;
correlación con eventos.

Los nombres y estructura definitivos quedan sujetos a los estándares técnicos.

35. Failure Matrix
Situación	Estrategia
DB commit falla	Rollback
Deadlock	Retry controlado
Serialization conflict	Retry si seguro
Unique constraint conflict	Tratar como conflicto/idempotencia
Validation error	No retry
Authorization failure	No retry
Security violation	No retry
Broker unavailable	Outbox permanece
Network timeout después de commit	Idempotency/reconciliation
Duplicate request	Idempotency
Duplicate event	Consumer idempotency
Redis unavailable	Fail-safe / fallback
Concurrent refresh	Atomic rotation
36. Performance

La estrategia debe contribuir a los objetivos del Blueprint:

API P95 <300 ms;
Authentication P95 <500 ms.

Para conseguirlo se deberá evitar:

transacciones largas;
locks excesivos;
serialización global;
consultas innecesarias;
distributed locks indiscriminados.

La concurrencia debe controlarse sobre el recurso mínimo necesario, no sobre toda la plataforma.

37. Testing

Las pruebas deberán incluir explícitamente escenarios concurrentes.

Ejemplos:

Refresh
100 concurrent requests
       ↓
same refresh token

Resultado esperado:

exactly one valid rotation

según la política final de retry/reuse.

Membership
Concurrent activate/suspend/remove
Roles
Concurrent assign/revoke
MFA
Concurrent challenge verification
Recovery
Concurrent recovery-code usage
Outbox
Concurrent publisher workers

Resultado:

No duplicate state transition
No lost event
38. Observability

Se deberán medir:

transaction latency;
lock wait time;
deadlocks;
serialization failures;
retry count;
failed transactions;
connection pool exhaustion;
outbox transaction failures;
duplicate requests;
idempotency conflicts.

Métricas conceptuales:

identity_db_transaction_duration
identity_db_lock_wait
identity_db_deadlocks
identity_db_transaction_retries
identity_idempotency_conflicts
identity_refresh_rotation_conflicts
identity_outbox_commit_failures
39. Consecuencias positivas
Integridad

Reduce estados inconsistentes.

Seguridad

Protege operaciones críticas como:

refresh;
MFA;
roles;
permisos;
sesiones.
Resiliencia

Permite retries controlados.

Event consistency

Mantiene Domain State + Outbox de forma atómica.

Escalabilidad

Permite múltiples instancias de API trabajando simultáneamente.

Auditability

Las operaciones críticas mantienen trazabilidad.

40. Consecuencias negativas

La estrategia introduce complejidad en:

locking;
isolation;
retries;
idempotency;
testing;
transaction boundaries;
deadlock handling.

También existe el riesgo de:

exceso de locks;
retries excesivos;
contention;
starvation;
transacciones demasiado largas.

Por ello, las decisiones deberán basarse en métricas y pruebas de carga.

41. Alternativas consideradas
Alternativa A — Sin control explícito

Rechazada.

No es adecuada para Identity.

Alternativa B — Lock global

Rechazada.

Limitaría severamente la escalabilidad.

Alternativa C — Redis como lock principal

Rechazada como estrategia general.

Redis no debe convertirse en la autoridad transaccional del dominio Identity.

Alternativa D — SERIALIZABLE para todo

Rechazada.

Puede introducir contention y retries innecesarios.

Alternativa E — Pessimistic locking para todo

Rechazada.

Generaría exceso de contention.

Alternativa F — Optimistic concurrency + atomic DB operations + selective locking

Seleccionada.

Permite equilibrar:

consistencia;
seguridad;
rendimiento;
escalabilidad.
42. Non-Negotiables
PostgreSQL es la autoridad transaccional.
Las operaciones críticas tienen transaction boundaries explícitas.
Domain State + Outbox deben ser atómicos cuando corresponda.
No se publican eventos esperando dentro de una transacción DB.
No se utilizarán locks globales como solución general.
No se utilizará Redis como autoridad transaccional.
Las constraints de PostgreSQL son parte del modelo de integridad.
Refresh Token rotation debe ser concurrency-safe.
MFA challenges deben ser concurrency-safe.
Recovery Codes deben ser concurrency-safe.
Cambios de roles y permisos deben ser concurrency-safe.
Membership lifecycle debe ser concurrency-safe.
Retries deben ser controlados.
Deadlocks deben detectarse y manejarse.
No se deben hacer retries ciegos.
Las transacciones deben ser cortas.
Las llamadas externas no deben permanecer innecesariamente dentro de transacciones.
Tenant isolation debe mantenerse bajo concurrencia.
Los escenarios críticos deben tener pruebas concurrentes.
La estrategia debe observar lock contention y transaction failures.
43. Traceability
Blueprint

Relacionado con:

Domain-Driven Architecture;
PostgreSQL transactional source;
Event-Driven Architecture;
Transactional Outbox;
Reliability;
Performance;
Security;
Auditability.
ESP

ESP-0001 — Identity Domain

Particularmente:

User;
Organization;
Membership;
Role;
Session;
Refresh Token;
MFA;
Authorization;
Audit;
Outbox.
Implementation Stories

Principalmente:

IS-IDENTITY-004 — Membership Management
IS-IDENTITY-005 — Authentication
IS-IDENTITY-006 — Authorization
IS-IDENTITY-007 — Role & Permission Management
IS-IDENTITY-008 — Session Management
IS-IDENTITY-009 — Refresh Token Lifecycle
IS-IDENTITY-010 — MFA
IS-IDENTITY-012 — Audit & Security Events
IS-IDENTITY-013 — Domain Events & Outbox
IS-IDENTITY-015 — Identity Persistence
IS-IDENTITY-016 — Identity Testing & Quality
44. ADR relacionados
ADR-IDENTITY-003 — Stateful Sessions & Refresh Tokens
ADR-IDENTITY-006 — Persistence Architecture
ADR-IDENTITY-007 — Event Architecture
ADR-IDENTITY-010 — Identity Testing & Quality
ADR-IDENTITY-012 — PostgreSQL Schema
ADR-IDENTITY-013 — Data Retention
ADR-IDENTITY-014 — Encryption & Secret Protection
ADR-IDENTITY-015 — Database Partitioning
ADR-IDENTITY-017 — Database Migration Strategy
ADR-IDENTITY-022 — Event Replay & Governance
ADR-IDENTITY-025 — Refresh Token Cryptography & Concurrency
ADR-IDENTITY-026 — API Pagination / Filtering / Idempotency
45. Dependencias

Las principales dependencias son:

ADR-012
PostgreSQL Schema
      ↓
ADR-015
Partitioning
      ↓
ADR-016
Concurrency & Transactions
      ↓
ADR-017
Migration Strategy

Y transversalmente:

ADR-007
Event Architecture
      ↓
Transactional Outbox
      ↓
ADR-016
Concurrency
46. Decisiones pendientes

Este ADR establece la estrategia general, pero deben concretarse posteriormente:

PostgreSQL
isolation level por operación;
pool sizing;
timeout;
lock timeout;
deadlock retry policy.
Optimistic concurrency
entidades que utilizarán version;
representación;
comportamiento del conflicto.
Pessimistic locking
operaciones exactas;
locks requeridos;
orden de adquisición.
Idempotency
storage;
TTL;
scope;
fingerprint;
response persistence.
Refresh Token
atomic rotation;
family locking;
retry legítimo vs reuse;
exact transaction strategy.
Distributed systems
cuándo utilizar distributed locks;
cuándo utilizar Redis;
límites de uso.
47. Criterios de aceptación

ADR-IDENTITY-016 podrá pasar de Proposed a Accepted cuando exista:

 transaction boundary definido por caso crítico;
 isolation level definido por operación;
 optimistic concurrency definida donde corresponda;
 pessimistic locking definido para operaciones críticas;
 estrategia definitiva de Refresh Token rotation;
 idempotency strategy;
 deadlock strategy;
 retry strategy;
 timeout strategy;
 lock timeout strategy;
 connection pool strategy;
 Domain + Outbox atomicity implementada;
 pruebas de concurrencia;
 pruebas de race conditions;
 pruebas de deadlock/retry;
 pruebas de tenant isolation concurrente;
 métricas y alertas;
 runbooks operacionales.
48. Siguiente ADR

La secuencia de persistencia queda ahora:

ADR-IDENTITY-012
PostgreSQL Schema
        ↓
ADR-IDENTITY-013
Data Retention
        ↓
ADR-IDENTITY-014
Encryption & Secret Protection
        ↓
ADR-IDENTITY-015
Database Partitioning
        ↓
ADR-IDENTITY-016
Concurrency & Transaction Strategy
        ↓
ADR-IDENTITY-017
Database Migration Strategy

El siguiente paso lógico es:

ADR-IDENTITY-017 — Database Migration Strategy

Ahí definiremos formalmente cómo EVOXA evolucionará el esquema PostgreSQL sin romper:

producción;
compatibilidad entre versiones;
zero downtime;
rollback;
particiones;
constraints;
índices;
datos existentes;
despliegues rolling/blue-green/canary.
