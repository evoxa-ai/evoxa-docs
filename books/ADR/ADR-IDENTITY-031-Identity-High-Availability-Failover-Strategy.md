ADR-IDENTITY-031 — Identity High Availability & Failover Strategy

Estado: Proposed
Versión: 1.0
Fecha: 2026-09-03
Dominio: Identity
Prioridad: Critical
Tipo: Architecture Decision Record

1. Contexto

Identity es un dominio crítico de EVOXA. Una interrupción de Identity puede impedir que los usuarios accedan a la plataforma y puede afectar transversalmente a otros dominios.

ADR-IDENTITY-029 definió Disaster Recovery & Backup, mientras que ADR-IDENTITY-030 estableció cómo validar que dicha recuperación funciona.

Este ADR aborda una preocupación diferente:

High Availability y Failover buscan mantener el servicio disponible durante fallos, mientras Disaster Recovery busca recuperar el servicio después de un desastre.

La arquitectura debe evitar que la caída de una única instancia, nodo, proceso o dependencia provoque una interrupción completa de Identity.

Los objetivos globales ya definidos incluyen:

Availability ≥99.9%
API P95 <300 ms
Authentication P95 <500 ms
RTO <1 hora
RPO <15 minutos

Estos objetivos deben interpretarse conjuntamente con los mecanismos de HA y Failover.

2. Problema

Identity puede experimentar fallos de:

instancia de API;
container;
nodo;
disponibilidad zonal;
PostgreSQL;
Redis;
Event Platform;
KMS;
Secret Manager;
networking;
load balancer;
deployment;
dependencia externa.

Si Identity depende de un único componente crítico, ese componente se convierte en un Single Point of Failure (SPOF).

Además, una estrategia de failover mal diseñada puede generar:

pérdida de sesiones;
inconsistencia de datos;
doble procesamiento;
pérdida de eventos;
autorización incorrecta;
cross-tenant access;
refresh token reuse;
fail-open;
pérdida de observabilidad.
3. Decisión

EVOXA Identity utilizará una arquitectura Highly Available, Stateless at API Layer y Stateful at Data Layer, con failover controlado.

La estrategia tendrá los siguientes principios:

Las instancias de Identity API serán stateless.
Las sesiones persistentes residirán fuera de las instancias.
PostgreSQL será la fuente transaccional de verdad.
Redis no será fuente primaria de datos críticos.
Las instancias API podrán reemplazarse sin pérdida permanente de estado.
Los servicios deberán ejecutarse con redundancia.
Health checks deberán detectar instancias no saludables.
El tráfico deberá retirarse de instancias fallidas.
PostgreSQL deberá disponer de una estrategia de failover.
Event Platform deberá tolerar fallos de consumidores/productores.
Failover no debe violar tenant isolation.
Authentication y Authorization deben fallar de forma segura.
Failover debe ser observable y auditable.
Disaster Recovery continuará siendo el mecanismo para desastres que superen HA.
4. High Availability vs Disaster Recovery

La arquitectura diferenciará explícitamente:

High Availability

Objetivo:

Mantener el servicio disponible durante fallos parciales.

Ejemplos:

API Instance A ❌
       ↓
Load Balancer
       ↓
API Instance B ✅
Failover

Proceso mediante el cual el tráfico o servicio cambia desde un componente fallido hacia otro componente disponible.

Disaster Recovery

Proceso utilizado cuando la infraestructura primaria no puede continuar operando normalmente.

Ejemplo:

Primary Region ❌
       ↓
DR Recovery
       ↓
Secondary Environment

Por lo tanto:

Failover no sustituye Disaster Recovery.

5. API Layer

Identity API deberá ejecutarse con múltiples instancias.

Conceptualmente:

                Load Balancer
                     │
          ┌──────────┼──────────┐
          ↓          ↓          ↓
       API-01     API-02     API-03
          │          │          │
          └──────────┼──────────┘
                     ↓
               PostgreSQL

Las instancias deberán ser:

stateless;
reemplazables;
horizontalmente escalables;
independientes entre sí.

No se almacenará estado crítico exclusivamente en memoria local.

6. Stateless API

Una instancia API no debe depender de:

memoria local;
filesystem local;
session state local;
archivos temporales persistentes;
claves criptográficas hardcoded.

Si una instancia desaparece:

API-01 ❌

otra instancia debe poder atender la solicitud:

API-02 ✅

sin pérdida permanente del estado de Identity.

7. Load Balancing

El tráfico deberá distribuirse entre instancias saludables.

El Load Balancer deberá:

detectar instancias no saludables;
retirar tráfico de instancias fallidas;
incorporar nuevas instancias después de validación;
distribuir tráfico;
soportar rolling deployments.

No deberá enviarse tráfico a una instancia que no haya superado sus readiness checks.

8. Health Checks

Se utilizarán al menos dos conceptos:

Liveness

Responde a:

¿El proceso sigue vivo?

Readiness

Responde a:

¿Esta instancia está preparada para recibir tráfico?

Esto permite distinguir:

Process alive
      ≠
Ready for production traffic

Una instancia puede estar viva pero no preparada porque:

no puede acceder a PostgreSQL;
no puede cargar una dependencia criptográfica crítica;
tiene configuración inválida;
está iniciando;
está ejecutando una transición controlada.
9. Failover de API

Cuando una instancia falle:

API-01 ❌
   ↓
Health Check
   ↓
Removed from Load Balancer
   ↓
API-02 / API-03

La recuperación deberá ser automática cuando la plataforma lo permita.

La instancia defectuosa deberá:

ser reiniciada;
reemplazada;
o escalada fuera del pool.
10. Graceful Shutdown

Las instancias deberán soportar graceful shutdown.

Antes de terminar:

dejar de aceptar nuevo tráfico;
permitir finalizar solicitudes en curso cuando corresponda;
cerrar conexiones;
finalizar operaciones seguras;
liberar recursos;
terminar.

Esto reduce:

requests truncados;
transacciones incompletas;
errores durante deployment;
conexiones abandonadas.
11. PostgreSQL High Availability

PostgreSQL es el componente de estado más crítico de Identity.

La arquitectura deberá contemplar una estrategia de alta disponibilidad que pueda incluir:

primary;
standby/replica;
automated failover;
health detection;
promotion;
connection redirection.

La implementación exacta dependerá del proveedor de infraestructura seleccionado.

12. PostgreSQL Failover

Conceptualmente:

             PostgreSQL
                 │
          ┌──────┴──────┐
          ↓             ↓
       Primary        Standby
          │
          X
          ↓
       Failover
          ↓
       Standby
       becomes
       Primary

El objetivo es minimizar:

downtime;
pérdida de datos;
intervención manual.

La promoción debe realizarse solamente bajo mecanismos controlados para evitar split-brain.

13. Split-Brain Protection

No se permitirá que dos nodos PostgreSQL sean considerados simultáneamente como primary activo.

La estrategia deberá contar con mecanismos de:

leader election;
fencing;
quorum;
provider-managed failover;
mecanismo equivalente.

La implementación exacta queda pendiente de la plataforma seleccionada.

14. PostgreSQL RPO

El failover de PostgreSQL debe considerar el RPO.

Debe distinguirse entre:

Synchronous replication

Mayor protección de datos, potencialmente mayor latencia.

Asynchronous replication

Menor impacto de latencia, pero potencial pérdida de las últimas transacciones.

La estrategia final deberá seleccionar el equilibrio adecuado para cumplir los objetivos de EVOXA.

15. PostgreSQL RTO

El failover de PostgreSQL debe diseñarse para reducir significativamente el tiempo de recuperación frente a un fallo de primary.

El RTO final debe validarse mediante los mecanismos definidos en:

ADR-IDENTITY-030 — Disaster Recovery Testing & Recovery Validation.

16. Database Connection Handling

Las API instances deberán manejar correctamente cambios de primary.

Ante failover:

conexiones existentes pueden fallar;
connection pools pueden contener conexiones inválidas;
nuevas conexiones deberán dirigirse al primary vigente;
operaciones fallidas deben distinguirse de operaciones desconocidas.

No se debe reintentar ciegamente una transacción que pueda haber sido confirmada antes de producirse el fallo.

17. Transaction Retry

Las operaciones transaccionales podrán reintentarse únicamente cuando sea seguro.

Debe distinguirse:

Request failed
      ≠
Transaction was not committed

Una operación puede haber sido:

COMMITTED

y posteriormente producir un error de conexión.

Por eso:

operaciones idempotentes pueden reintentarse;
operaciones no idempotentes deben utilizar Idempotency-Key cuando corresponda;
no se debe duplicar un efecto de negocio.

Esto se relaciona directamente con ADR-IDENTITY-026.

18. Redis High Availability

Redis se utilizará como:

cache;
estado temporal;
soporte de operaciones de alta frecuencia.

La estrategia podrá utilizar:

replicas;
managed Redis;
failover;
cluster;
mecanismo equivalente.

Pero:

Redis no deberá convertirse en requisito único para recuperar datos transaccionales críticos.

19. Redis Failure

Si Redis deja de estar disponible:

Redis ❌
   ↓
Detect
   ↓
Degrade / Recover
   ↓
PostgreSQL remains source of truth

Dependiendo del componente afectado, Identity podrá:

reconstruir cache;
invalidar cache;
limitar temporalmente ciertas operaciones;
utilizar una estrategia alternativa segura.

Nunca deberá utilizar Redis failure como motivo para deshabilitar controles de seguridad.

20. Authorization Cache

Si Authorization utiliza cache:

el cache será derivado;
tendrá TTL;
tendrá invalidación;
podrá reconstruirse;
deberá respetar tenant context.

Ante pérdida del cache:

el sistema debe poder volver a consultar la fuente autorizada.

Nunca deberá interpretarse:

cache unavailable

como:

ALLOW
21. Event Platform High Availability

La Event Platform deberá soportar fallos de:

productores;
consumers;
nodos;
conexiones;
temporal availability.

Identity deberá utilizar:

retry;
backoff;
DLQ;
idempotency;
Outbox.

Un fallo temporal de Event Platform no debe provocar automáticamente rollback de una transacción de negocio ya confirmada en PostgreSQL.

22. Outbox durante Failover

La relación:

Business Transaction
       +
Outbox Event

debe permanecer atómica.

Si PostgreSQL confirma la transacción:

Business State = COMMITTED
Outbox = COMMITTED

la publicación del evento puede producirse posteriormente.

Esto permite:

PostgreSQL
   ↓
Outbox
   ↓
Retry
   ↓
Event Platform

sin perder el evento por una caída temporal.

23. Authentication durante Failover

Authentication debe comportarse de forma segura durante una falla parcial.

API instance failure

Debe ser transparente siempre que exista otra instancia saludable.

PostgreSQL failure

Authentication podrá quedar temporalmente indisponible si no existe un primary recuperable.

No se permitirá:

fail-open authentication

Es decir:

Database unavailable
       ↓
ALLOW LOGIN

Esto está prohibido.

24. Authorization durante Failover

Authorization seguirá el principio:

Fail Closed

Si el sistema no puede determinar correctamente:

user;
organization;
membership;
role;
permission;
policy;

no deberá emitir:

ALLOW

por defecto.

Podrá devolver:

DENY;
temporal unavailable;
REAUTHENTICATE;
REQUIRE_MFA;

según el contexto y las políticas definidas.

25. JWT durante Failover

Los access tokens JWT son stateless.

Por lo tanto, una instancia API saludable puede validar un JWT sin consultar necesariamente el estado de sesión para cada request, según la arquitectura de validación establecida.

Sin embargo, esto no debe interpretarse como bypass de:

authorization;
tenant isolation;
policy;
session requirements;
MFA requirements.

La capacidad de validar criptográficamente un JWT no significa automáticamente:

autorización concedida.

26. JWT Signing Keys durante Failover

Las instancias deberán poder acceder a las claves necesarias mediante:

KMS;
protected key material;
JWKS;
mecanismos equivalentes.

Las claves no deberán depender de una única instancia.

Una caída de una API instance no debe provocar pérdida de acceso al material criptográfico.

27. KMS Failure

KMS es una dependencia de seguridad crítica.

Si una operación criptográfica crítica requiere KMS y este no está disponible:

no debe utilizarse una clave alternativa no autorizada;
no debe deshabilitarse la protección;
no debe realizarse fallback inseguro.

El sistema deberá:

fail closed;
generar observability signals;
generar security/operational events cuando corresponda;
recuperar cuando KMS vuelva a estar disponible.
28. Secret Manager Failure

El comportamiento será equivalente.

No se permitirá:

Secret Manager unavailable
       ↓
Use hardcoded secret

Las aplicaciones deberán disponer de mecanismos seguros de:

carga;
cache temporal;
renovación;
invalidación.

La estrategia exacta se definirá en función del runtime final.

29. Deployment Failover

Los deployments no deberán requerir downtime completo.

Se utilizarán estrategias como:

rolling deployment;
blue/green;
canary.

Conceptualmente:

Version 1
API-01 API-02 API-03
       ↓
Version 2
       ↓
Health Validation
       ↓
Traffic Shift
       ↓
Version 1 Removed

Si la nueva versión falla:

Rollback
30. Zero-Downtime Migrations

Las migrations deberán ser compatibles con HA.

Para cambios de schema se deberá favorecer:

Expand → Migrate → Contract

En lugar de:

cambiar/eliminar inmediatamente una estructura utilizada por la versión anterior.

Esto permite que dos versiones de la aplicación puedan coexistir temporalmente durante un deployment.

31. Availability Zones

Cuando el proveedor lo permita, los componentes críticos deberán distribuirse entre múltiples Availability Zones.

Esto reduce el impacto de:

fallo de hardware;
fallo de networking;
fallo de una zona;
mantenimiento zonal.

La distribución exacta queda pendiente del proveedor cloud.

32. Multi-Region

Multi-region no se establece como requisito obligatorio de la primera implementación.

Se considera una evolución posible cuando:

los requisitos de disponibilidad lo justifiquen;
el negocio lo requiera;
el costo sea aceptable;
la arquitectura esté preparada.

Una estrategia multi-region deberá resolver adicionalmente:

database replication;
conflict handling;
global routing;
key management;
event ordering;
tenant placement;
data residency.
33. Graceful Degradation

Identity podrá degradar funcionalidad no crítica cuando sea seguro.

Ejemplo:

Analytics unavailable
        ↓
Identity continues

Pero no deberá degradar controles fundamentales como:

authentication integrity;
authorization;
tenant isolation;
MFA;
security policies.

Regla:

Availability nunca debe conseguirse sacrificando un control de seguridad crítico.

34. Dependency Classification

Las dependencias deberán clasificarse como:

Critical

La ausencia impide ejecutar de manera segura una operación crítica.

Ejemplos:

PostgreSQL;
JWT key material;
critical security configuration.
Important

Su ausencia degrada funcionalidad pero permite continuar parcialmente.

Optional

Su ausencia no afecta la operación fundamental.

Esta clasificación permitirá definir:

readiness;
failover;
retries;
circuit breakers;
graceful degradation.
35. Circuit Breakers

Las integraciones externas que puedan fallar deberán utilizar mecanismos equivalentes a:

timeout;
retry;
exponential backoff;
circuit breaker;
bulkhead.

Especialmente para:

MFA providers;
external identity providers;
notification providers;
external security services.

Nunca se debe permitir que una dependencia externa lenta bloquee indefinidamente los recursos de Identity.

36. Retry Policy

Los retries deberán:

ser limitados;
utilizar backoff;
incluir jitter cuando corresponda;
distinguir errores transitorios de permanentes;
respetar idempotency.

No se permitirá:

Failure
 ↓
Retry
 ↓
Retry
 ↓
Retry
 ↓
Infinite Retry

Esto puede producir una cascada de fallos.

37. Failover y Rate Limiting

Rate limiting debe sobrevivir razonablemente a cambios de instancia.

Si depende de Redis:

deberá soportar múltiples API instances;
deberá tolerar failover;
deberá evitar que una instancia aislada permita abuso.

Ante pérdida de Redis, la estrategia debe seguir siendo segura y no permitir un bypass accidental del rate limiting crítico.

38. Observability del Failover

Todos los failovers importantes deberán ser observables.

Deben registrarse métricas como:

instance failures;
health check failures;
failover count;
failover duration;
database promotion;
Redis failover;
event consumer failover;
retry count;
connection failures;
authentication failures;
authorization failures.

Deben conservarse:

correlation ID;
trace ID;
causation ID cuando corresponda.

Nunca se deberán registrar secretos o tokens.

39. Alerts

Deberán existir alertas para:

pérdida de instancia;
reducción de capacidad;
database failover;
replication lag;
database health degradation;
Redis failover;
Event Platform failures;
Outbox backlog;
KMS failures;
Secret Manager failures;
authentication degradation;
authorization errors;
elevated latency;
repeated failovers.

Las alertas deben ser accionables y evitar alert fatigue.

40. Failover Testing

Todo mecanismo de failover deberá probarse.

Como mínimo:

API
API-01 ❌
→ Traffic → API-02
PostgreSQL
Primary ❌
→ Standby
→ Promotion
→ Application Reconnect
Redis
Redis Primary ❌
→ Failover
→ Application Recovery
Event Platform
Broker/Consumer ❌
→ Retry
→ Recovery
→ Idempotent Processing
41. Failover Validation

Después de cada prueba se debe verificar:

availability;
RTO;
RPO cuando corresponda;
data integrity;
tenant isolation;
authentication;
authorization;
sessions;
refresh;
MFA;
events;
observability.

El proceso se relaciona con:

ADR-IDENTITY-030 — Disaster Recovery Testing & Recovery Validation

42. Security During Failover

Un failover nunca debe:

cambiar permisos;
cambiar tenant;
eliminar MFA;
reactivar sessions;
reactivar refresh tokens;
modificar roles;
deshabilitar policies;
exponer secretos.

Toda transición crítica debe mantener las mismas reglas de seguridad del entorno normal.

43. AI Agents

Los AI Agents podrán:

detectar anomalías;
analizar fallos;
recomendar failover;
predecir degradaciones;
iniciar acciones previamente autorizadas mediante herramientas controladas.

Pero no podrán:

deshabilitar authentication;
hacer fail-open;
deshabilitar tenant isolation;
eliminar audit trails;
cambiar KMS policies;
acceder a secretos;
promover bases de datos de manera arbitraria;
alterar mecanismos de seguridad para mantener disponibilidad.

La autonomía de los AI Agents deberá estar subordinada a:

Identity + Security Policy Engine + Operational Governance.

44. Consecuencias positivas

Esta decisión proporciona:

mayor disponibilidad;
menor impacto de fallos individuales;
recuperación automática de instancias;
menor downtime;
escalabilidad horizontal;
mejor tolerancia a fallos;
menor cantidad de SPOFs;
deployments con menor riesgo;
failover controlado de PostgreSQL;
recuperación de Event Platform;
mejor resiliencia operacional.
45. Consecuencias negativas

Introduce:

mayor complejidad;
costos adicionales;
infraestructura redundante;
complejidad de PostgreSQL HA;
complejidad de connection management;
necesidad de pruebas de failover;
mayor complejidad de observability;
necesidad de automatización;
potenciales problemas de consistencia durante fallos.
46. Alternativas consideradas
A. Single API + Single PostgreSQL

Rechazada.

Introduce múltiples Single Points of Failure.

B. Múltiples API pero PostgreSQL único

Rechazada como estrategia final.

Mejora la disponibilidad de la API, pero PostgreSQL continúa siendo SPOF.

C. Full Active-Active Multi-Region

No adoptada inicialmente.

Proporciona una disponibilidad potencialmente superior, pero introduce complejidad significativa.

Se mantiene como evolución futura.

D. HA Multi-Instance + Stateful Data Layer + Controlled Failover

Adoptada.

Representa el equilibrio adecuado entre:

disponibilidad;
consistencia;
seguridad;
complejidad;
costo.
47. No negociables
Identity API debe ser stateless.
No debe existir estado crítico únicamente en memoria de una instancia.
Deben existir múltiples API instances en producción.
Deben existir health checks.
Las instancias no saludables deben retirarse del tráfico.
PostgreSQL debe tener estrategia de HA/failover.
Split-brain debe prevenirse.
Redis no será source of truth.
Outbox debe mantenerse transaccional.
Event processing debe ser idempotente.
Authentication nunca debe fail-open.
Authorization nunca debe fail-open.
Tenant isolation debe mantenerse durante failover.
MFA no puede deshabilitarse por disponibilidad.
Refresh token revocation debe preservarse.
JWT signing keys deben estar disponibles de manera redundante y segura.
KMS/Secret Manager no pueden reemplazarse por secretos hardcoded.
Deployments deben minimizar downtime.
Database migrations deben ser compatibles con HA.
Failover debe ser observable.
Failover debe ser probado.
RTO debe medirse.
AI Agents no pueden saltarse controles de seguridad para mantener disponibilidad.
High Availability no reemplaza Disaster Recovery.
48. Traceability
Referencia	Relación
Blueprint — Architecture Map	Lifecycle operacional
Blueprint — Platform Overview	Cloud-ready / resilient
ESP-0001 Identity	Arquitectura de Identity
ADR-IDENTITY-006	PostgreSQL Source of Truth
ADR-IDENTITY-007	Outbox / Event Platform
ADR-IDENTITY-008	KMS / Cryptography
ADR-IDENTITY-011	Runtime / Deployment / RTO
ADR-IDENTITY-017	Database Migrations
ADR-IDENTITY-019	Event Partitioning
ADR-IDENTITY-021	Controlled Replay
ADR-IDENTITY-022	Consumer Idempotency
ADR-IDENTITY-026	API Idempotency
ADR-IDENTITY-028	Observability
ADR-IDENTITY-029	Disaster Recovery
ADR-IDENTITY-030	Recovery Validation
ADR-IDENTITY-031	High Availability & Failover
49. Dependencias

Este ADR depende de:

PostgreSQL HA;
Redis HA;
Event Platform;
Load Balancer;
Container orchestration;
KMS;
Secret Manager;
Observability;
Deployment platform;
Database migrations;
Idempotency;
Recovery Testing.
50. Decisiones pendientes

Quedan para posteriores decisiones técnicas:

proveedor cloud;
container orchestration;
Load Balancer;
número mínimo de API replicas;
Availability Zones;
PostgreSQL HA technology;
synchronous vs asynchronous replication;
failover manager;
PostgreSQL connection routing;
Redis HA technology;
Event Platform HA;
health check thresholds;
readiness dependency policy;
retry/backoff values;
circuit breaker thresholds;
graceful degradation matrix;
multi-region strategy;
global traffic routing;
automated failover criteria;
manual approval requirements;
failover observability;
failover drill frequency.
51. Acceptance Criteria

El ADR se considerará implementado cuando:

 Identity API tenga múltiples instancias;
 API sea stateless;
 exista Load Balancer;
 existan liveness/readiness checks;
 instancias unhealthy sean retiradas del tráfico;
 exista graceful shutdown;
 PostgreSQL tenga estrategia HA;
 PostgreSQL failover esté automatizado o controlado;
 exista protección contra split-brain;
 connection pools soporten failover;
 Redis tenga estrategia HA apropiada;
 Redis pueda perderse sin pérdida permanente de datos críticos;
 Event Platform tenga estrategia HA;
 Outbox sobreviva a fallos temporales;
 consumer idempotency funcione después de failover;
 Authentication permanezca fail-closed;
 Authorization permanezca fail-closed;
 tenant isolation sea validado;
 MFA permanezca protegido;
 refresh token revocation sea preservado;
 KMS sea redundante/disponible según arquitectura;
 Secret Manager tenga estrategia de disponibilidad;
 deployments puedan realizarse sin downtime completo;
 migrations sean compatibles con HA;
 existan métricas de failover;
 existan alertas;
 se ejecuten failover drills;
 RTO sea medido;
 los escenarios críticos estén documentados.
52. Siguiente ADR

La secuencia natural después de HA & Failover es:

ADR-IDENTITY-032 — Identity Rate Limiting, Abuse Prevention & Brute-Force Protection

Este ADR debería cerrar una pieza especialmente importante antes de pasar a implementación: cómo Identity protege Login, Refresh Token, MFA, Password Reset, API Tokens y otros endpoints sensibles contra brute-force, credential stuffing, scraping, abuse y ataques distribuidos, integrándolo con Redis, Security Policy Engine, observability y los mecanismos de failover definidos aquí.
