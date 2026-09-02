IS-IDENTITY-017 — Identity Runtime & Deployment

Estado: Draft
Prioridad: High / Critical
Dominio: Identity
Capability: Runtime, Deployment & Operations
ESP principal: ESP-0001
ADR principal: ADR-IDENTITY-011 — Identity Runtime & Deployment
ADRs relacionados: ADR-IDENTITY-006, 007, 008, 010
Dependencias: IS-IDENTITY-001 → IS-IDENTITY-016
Arquitectura: Presentation → API → Application → Domain → Infrastructure → Runtime

1. Propósito

Definir cómo el dominio Identity será ejecutado, desplegado, configurado, monitorizado y recuperado en los distintos ambientes de EVOXA.

Esta IS establece los requisitos para:

runtime;
containers;
configuración;
secrets;
PostgreSQL;
Redis;
Event Platform;
health checks;
readiness/liveness;
observabilidad;
escalamiento;
disponibilidad;
deployment;
rollback;
disaster recovery;
seguridad operacional;
CI/CD;
ambientes Dev → QA → Staging → Production.

La implementación deberá permitir que Identity funcione como un servicio stateless a nivel de proceso, manteniendo su estado crítico en servicios externos apropiados.

2. Business Goal

Garantizar que Identity pueda ejecutarse en producción de manera:

segura + altamente disponible + escalable + observable + recuperable + reproducible.

El runtime no debe convertirse en una dependencia frágil de una instancia específica de aplicación.

La arquitectura debe soportar:

                Identity API
                     │
          ┌──────────┼──────────┐
          ▼          ▼          ▼
       Instance   Instance   Instance
          │          │          │
          └──────────┼──────────┘
                     │
        ┌────────────┼────────────┐
        ▼            ▼            ▼
   PostgreSQL      Redis      Event Platform
3. Fundamento arquitectónico

El Blueprint establece una plataforma:

cloud-ready;
containerized;
stateless;
observable;
secure by design;
preparada para escalamiento.

El runtime debe mantener separadas:

Application Process
        │
        ├── Configuration
        ├── Secrets
        ├── Database
        ├── Cache
        └── Event Platform

La aplicación no debe depender de almacenamiento local como fuente de verdad.

4. Principio Stateless

Las instancias de Identity API deberán ser intercambiables.

Ejemplo:

Request 1 → Instance A
Request 2 → Instance B
Request 3 → Instance C

El resultado debe ser consistente.

No se debe almacenar información crítica exclusivamente en:

/tmp
local filesystem
process memory
local session
5. Estado externo

Los datos persistentes deben residir fuera del proceso:

Identity API
    │
    ├── PostgreSQL → persistent state
    ├── Redis → temporary/cache
    └── Event Platform → asynchronous events

Esto permite:

restart;
rescheduling;
scaling;
rolling deployment;
failover.
6. Containerización

Identity deberá poder ejecutarse como container reproducible.

El container deberá incluir solamente:

aplicación;
dependencias;
configuración necesaria no secreta.

No debe incluir:

credentials;
private keys;
production secrets;
tokens;
passwords.
7. Immutable Runtime

La imagen desplegada deberá ser tratada como artefacto inmutable.

Conceptualmente:

Source Code
    ↓
Build
    ↓
Container Image
    ↓
Security Scan
    ↓
Registry
    ↓
Deployment

No se deben modificar archivos arbitrariamente dentro del container en producción para corregir configuración.

8. Environment Separation

Identity deberá operar al menos en:

Development
     ↓
QA
     ↓
Staging
     ↓
Production

Cada ambiente deberá tener:

configuración independiente;
secrets independientes;
databases independientes;
Redis independiente;
Event Platform/configuración aislada según arquitectura.
9. Development

Development debe permitir:

desarrollo rápido;
debugging controlado;
ejecución local;
test automation;
infraestructura reproducible.

No debe utilizar:

Production credentials
Production database
Production tokens
10. QA

QA deberá validar:

functional testing;
integration testing;
API contracts;
security;
persistence;
events;
migrations;
deployment.

Debe ser un entorno suficientemente cercano a producción para detectar problemas de integración.

11. Staging

Staging debe aproximarse a Production en:

runtime;
configuración;
networking;
observabilidad;
deployment;
dependencies.

Debe utilizarse para:

release candidate;
E2E;
performance;
security;
migration validation;
rollback validation.
12. Production

Production debe priorizar:

Availability
Security
Observability
Performance
Recovery

Los cambios deben llegar mediante pipeline controlado.

13. Configuration Management

La configuración no debe estar hardcodeada.

Conceptualmente:

Application
   │
   ├── Environment Variables
   ├── Configuration Service
   └── Secret Manager

Debe diferenciarse claramente:

Configuration

Ejemplo:

LOG_LEVEL
API_PORT
ENVIRONMENT
FEATURE_FLAG
Secret

Ejemplo:

DATABASE_PASSWORD
JWT_PRIVATE_KEY
MFA_ENCRYPTION_KEY
14. Secret Management

Los secrets deben provenir de un mecanismo externo de gestión de secretos.

Nunca deben almacenarse directamente en:

Git
Dockerfile
source code
logs
OpenAPI examples
configuration files committed to repository
15. Cryptographic Keys

Identity requiere gestión centralizada de claves criptográficas.

Debe contemplarse:

JWT signing keys
MFA encryption keys
Webhook/integration keys
Other cryptographic material

Las claves deben tener:

rotación;
versionado;
identificación mediante kid cuando corresponda;
acceso restringido.
16. JWT Key Rotation

El runtime debe permitir:

Key A → active
Key B → next

durante una transición.

Los tokens previamente emitidos deben poder validarse según la política de rotación.

El servicio deberá exponer las claves públicas mediante el mecanismo establecido por la arquitectura de autenticación, como JWKS cuando corresponda.

17. Database Runtime

Identity utilizará PostgreSQL como datastore transaccional.

La aplicación debe utilizar:

connection pooling
timeouts
retry policy
health checks

Los parámetros concretos dependerán del entorno.

18. Database Availability

La disponibilidad de PostgreSQL es crítica.

Si PostgreSQL está indisponible, Identity debe:

detectar la condición;
no generar respuestas falsas de éxito;
evitar loops agresivos de reconexión;
exponer estado mediante health/observability;
recuperarse cuando la dependencia vuelva.
19. Database Migrations

Las migrations deben ejecutarse mediante un mecanismo controlado.

Deployment:

Build
 ↓
Migration Validation
 ↓
Migration
 ↓
Application Deployment

No se debe depender de auto-sync del ORM en Production.

20. Zero-Downtime Migrations

Cuando existan deployments rolling/blue-green/canary, las migrations deben ser compatibles con las versiones coexistentes.

Se recomienda el patrón:

Expand
  ↓
Deploy
  ↓
Migrate Data
  ↓
Switch
  ↓
Contract

La aplicación no debe requerir que todas las instancias sean reemplazadas simultáneamente.

21. Redis Runtime

Redis podrá utilizarse para:

cache;
rate limiting;
temporary challenges;
authorization cache;
distributed coordination cuando corresponda.

No debe convertirse en dependencia exclusiva para reconstruir el estado persistente de Identity.

22. Redis Failure

Si Redis falla:

Redis DOWN

Identity debe aplicar la política de degradación correspondiente.

Nunca deberá:

Redis failure
    ↓
Authentication bypass

ni:

Redis failure
    ↓
Authorization ALLOW by default

El principio continúa siendo:

Fail closed para controles de seguridad.

23. Event Platform Runtime

Identity debe conectarse al Event Platform mediante configuración externa.

El runtime debe soportar:

connection failure;
timeout;
retry;
reconnect;
backpressure;
publisher restart.
24. Outbox Worker

El Outbox Publisher puede ejecutarse:

Como worker separado
Identity API
      │
      ▼
 PostgreSQL
      │
      ▼
 Outbox Worker
      │
      ▼
 Event Platform

o como componente administrado dentro del runtime.

La decisión final depende de la arquitectura de plataforma.

El requisito fundamental es que la publicación continúe funcionando aunque una instancia específica de API sea reemplazada.

25. Health Checks

Identity debe exponer health checks.

Conceptualmente:

GET /health
GET /health/live
GET /health/ready
26. Liveness

Liveness debe responder:

¿El proceso está vivo?

No debería depender de que PostgreSQL o Event Platform estén disponibles para determinar simplemente si el proceso existe.

Ejemplo:

Process alive
→ LIVE
27. Readiness

Readiness debe responder:

¿Esta instancia está preparada para recibir tráfico?

Puede evaluar dependencias críticas:

Application
+
Required dependencies

Una instancia que no pueda operar correctamente puede retirarse del tráfico.

28. Dependency Health

Debe diferenciarse:

Application health
Database health
Redis health
Event Platform health

Esto evita ocultar el origen de una falla.

29. Startup

El startup debe:

cargar configuración;
validar configuración;
inicializar logging;
inicializar telemetry;
inicializar database connection;
inicializar required dependencies;
iniciar API;
iniciar workers cuando corresponda;
exponer readiness.

Si falta un secret obligatorio:

STARTUP FAILURE

No debe arrancar parcialmente en un estado inseguro.

30. Graceful Shutdown

Ante shutdown:

SIGTERM
   ↓
Stop accepting new requests
   ↓
Finish active requests
   ↓
Finish/stop workers safely
   ↓
Close connections
   ↓
Exit

Debe evitarse corrupción o pérdida de operaciones.

31. Request Draining

Durante rolling deployment, la instancia debe dejar de recibir tráfico antes de finalizar.

Esto permite:

Instance A
   ↓
Drain
   ↓
Finish requests
   ↓
Shutdown

mientras:

Instance B
Instance C

continúan atendiendo tráfico.

32. Scaling

Identity API debe poder escalar horizontalmente:

1 instance
     ↓
3 instances
     ↓
10 instances

según demanda.

El estado crítico no debe depender de una instancia.

33. Autoscaling

El autoscaling podrá basarse en:

CPU;
memory;
request rate;
latency;
queue/outbox backlog.

La métrica exacta deberá definirse en Platform Architecture.

34. Session Scaling

El modelo stateful de Session no significa que la API deba mantener sesiones en memoria.

Las Sessions viven en PostgreSQL.

Por lo tanto:

Instance A
     │
     ▼
PostgreSQL
     ▲
     │
Instance B

puede consultar el mismo estado.

35. Deployment Strategy

El despliegue debe soportar estrategias como:

rolling;
blue/green;
canary.

La estrategia exacta queda pendiente de la arquitectura de plataforma.

36. Rolling Deployment

Ejemplo:

v1 v1 v1
 ↓
v2 v1 v1
 ↓
v2 v2 v1
 ↓
v2 v2 v2

Debe mantenerse compatibilidad entre versiones durante la transición.

37. Blue/Green

Conceptualmente:

        Load Balancer
             │
       ┌─────┴─────┐
       ▼           ▼
    Blue v1      Green v2

Después de validación:

Traffic → Green

Permite rollback rápido.

38. Canary

Para cambios de alto riesgo:

99% → v1
 1% → v2

y progresivamente:

10%
25%
50%
100%

La promoción debe depender de métricas y Quality Gates.

39. Rollback

Debe existir rollback de:

application version;
container image;
configuration;
compatible database changes.

No debe asumirse que todas las migrations pueden simplemente revertirse.

Por eso las migrations deberán diseñarse para soportar deployment recovery.

40. Deployment Failure

Si un nuevo release produce:

error rate alto;
authentication failures;
authorization failures;
latency degradation;
crash loop;

el deployment debe detenerse o revertirse según la estrategia.

41. Observability

Identity deberá integrarse con:

Logs
Metrics
Traces
Audit
Security Events

Estos sistemas tienen propósitos distintos y no deben mezclarse.

42. Structured Logging

Los logs deberán ser estructurados.

Deben permitir correlacionar:

timestamp
service
environment
instance
level
correlation_id
trace_id
request_id
user_id cuando sea apropiado
tenant_id cuando sea apropiado
event_id cuando corresponda

No deben contener secretos.

43. Metrics

Identity deberá exponer métricas para:

API
request count;
error rate;
latency;
status codes.
Authentication
login attempts;
successful logins;
failed logins;
MFA challenges;
blocked attempts.
Authorization
decisions;
DENY;
challenges;
policy violations.
Sessions
active sessions;
created;
revoked;
expired.
Refresh
refresh operations;
rotation;
failures;
reuse detection.
Outbox
pending;
published;
failed;
DLQ;
backlog.
44. Distributed Tracing

Los traces deberán propagarse a:

API
 ↓
Application
 ↓
Database
 ↓
Redis
 ↓
Outbox
 ↓
Event Platform

cuando las tecnologías permitan instrumentación.

45. Security Monitoring

Security Events deberán poder alimentar sistemas de:

detection;
alerting;
incident response.

Ejemplos:

RefreshTokenReuseDetected
PrivilegeEscalationDetected
TenantAccessDenied
PolicyViolationDetected
46. Alerting

Deben existir alertas para situaciones críticas.

Ejemplos:

Authentication failure spike
Authorization denial spike
Refresh token reuse spike
Database unavailable
Outbox backlog growing
DLQ growth
High latency
High error rate

Los thresholds exactos deben definirse operacionalmente.

47. Availability

El objetivo de plataforma establecido es:

Availability ≥ 99.9%

Identity debe diseñarse para contribuir a ese objetivo.

Esto implica:

múltiples instancias;
externalized state;
health checks;
graceful shutdown;
recovery;
observability.
48. Disaster Recovery

Identity debe participar en la estrategia global de Disaster Recovery.

Objetivos:

RTO < 1 hour
RPO < 15 minutes

La recuperación debe incluir:

Application
Database
Configuration
Secrets
Event processing

según las dependencias.

49. Backup

Debe existir backup de:

PostgreSQL
identity data;
sessions;
refresh token state;
MFA state;
policies;
audit;
required outbox data.
Configuration

Los mecanismos externos deben permitir recuperar configuración necesaria.

50. Restore Testing

No basta con tener backups.

Debe probarse periódicamente:

Backup
 ↓
Restore
 ↓
Validation
 ↓
Identity startup
 ↓
Functional verification
51. Security Runtime

El container/runtime deberá aplicar:

least privilege;
non-root execution cuando sea posible;
minimal image;
restricted filesystem;
network restrictions;
secret isolation;
resource limits.
52. Network Security

La comunicación entre componentes deberá estar restringida.

Conceptualmente:

Internet
   │
   ▼
API Gateway
   │
   ▼
Identity
   │
   ├── PostgreSQL
   ├── Redis
   └── Event Platform

No todas las dependencias deben quedar expuestas públicamente.

53. Resource Limits

Identity debe definir:

CPU requests;
CPU limits;
memory requests;
memory limits;
connection limits.

Los valores exactos serán definidos según benchmarks.

54. Rate Limiting Runtime

Los mecanismos de rate limiting deben poder escalar horizontalmente.

Cuando el estado de rate limit sea distribuido, no deberá depender de memoria local de una única instancia.

55. Deployment Security

Cada artefacto debe poder trazarse:

Git Commit
   ↓
Build
   ↓
Image
   ↓
Security Scan
   ↓
Deployment

Debe ser posible determinar qué versión de código está ejecutándose.

56. Supply Chain Security

El pipeline debe validar:

dependencies;
container image;
vulnerabilities;
provenance cuando corresponda;
secrets;
artifact integrity.
57. CI/CD

Pipeline conceptual:

Commit
 ↓
Lint
 ↓
Unit Tests
 ↓
Security Scan
 ↓
Build
 ↓
Integration Tests
 ↓
Contract Tests
 ↓
Container Build
 ↓
Image Scan
 ↓
Deploy QA
 ↓
E2E
 ↓
Deploy Staging
 ↓
Quality Gate
 ↓
Production
58. Production Deployment Gate

Production deberá bloquearse si:

Critical tests FAIL
Security gate FAIL
Migration validation FAIL
Contract compatibility FAIL
Tenant isolation FAIL
Critical vulnerability detected
59. Configuration Rollout

Los cambios de configuración sensibles deben ser auditables.

Ejemplos:

JWT configuration;
MFA policies;
rate limits;
authorization policies;
feature flags.

Un cambio de configuration crítico puede requerir:

authorization
+
audit
+
rollback capability
60. Feature Flags

Cuando se utilicen feature flags:

Feature OFF
     ↓
Deploy
     ↓
Validate
     ↓
Feature ON

Los flags de seguridad no deben permitir accidentalmente:

Authorization OFF
MFA OFF
Tenant Isolation OFF

sin una política explícita y controlada.

61. Operational Runbooks

Deberán existir procedimientos para:

Authentication outage
Database outage
Redis outage
Event Platform outage
Refresh Token compromise
MFA incident
Cross-tenant security incident
Deployment rollback
Database restore
Outbox backlog
DLQ recovery
62. Incident Response

Un incidente crítico de Identity debe poder responder:

What happened?
When?
Which tenant?
Which users?
Which sessions?
Which tokens?
Which instance?
Which deployment?
Which policy?
Which events?

La observabilidad y Audit/Security Events deben permitir reconstruir el incidente.

63. Security Incident — Token Reuse

Ejemplo:

Refresh Token Reuse
        ↓
Security Event
        ↓
Audit
        ↓
Family Revocation
        ↓
Session Revocation
        ↓
Alert

Los procedimientos operacionales deberán permitir investigar el incidente.

64. Security Incident — Tenant Isolation

Ante evidencia de:

Tenant A → Tenant B access

deberá activarse el procedimiento de incident response.

Esto es un incidente de máxima prioridad porque viola uno de los controles fundamentales de Identity.

65. Technical Tasks
Runtime

T01 — Definir runtime architecture.
T02 — Definir container strategy.
T03 — Crear runtime configuration model.
T04 — Definir stateless requirements.
T05 — Definir resource limits.

Configuration

T06 — Definir configuration management.
T07 — Separar config/secrets.
T08 — Implementar configuration validation.
T09 — Definir environment profiles.

Secrets

T10 — Integrar secret manager.
T11 — Definir secret rotation.
T12 — Integrar cryptographic key management.
T13 — Implementar JWT key rotation.
T14 — Definir secret access policies.

PostgreSQL

T15 — Configurar connection pooling.
T16 — Configurar timeouts.
T17 — Implementar database health.
T18 — Integrar migrations.
T19 — Validar zero-downtime migration.

Redis

T20 — Configurar Redis connection.
T21 — Configurar cache.
T22 — Configurar failure handling.
T23 — Validar fail-closed security behavior.

Events

T24 — Configurar Event Platform.
T25 — Configurar Outbox Publisher.
T26 — Implementar publisher recovery.
T27 — Configurar retries.
T28 — Configurar DLQ handling.

Health

T29 — Implement liveness.
T30 — Implement readiness.
T31 — Implement dependency health.
T32 — Implement graceful shutdown.
T33 — Implement request draining.

Scaling

T34 — Configure horizontal scaling.
T35 — Define autoscaling metrics.
T36 — Validate multi-instance runtime.
T37 — Validate concurrent workloads.

Deployment

T38 — Define deployment strategy.
T39 — Implement rolling deployment.
T40 — Validate blue/green/canary strategy.
T41 — Implement rollback.
T42 — Validate deployment recovery.

Observability

T43 — Structured logging.
T44 — Metrics.
T45 — Distributed tracing.
T46 — Security monitoring.
T47 — Alerting.

Security

T48 — Harden container.
T49 — Network restrictions.
T50 — Runtime least privilege.
T51 — Image scanning.
T52 — Supply chain validation.

CI/CD

T53 — Build pipeline.
T54 — Test gates.
T55 — Security gates.
T56 — Image build.
T57 — Image scanning.
T58 — QA deployment.
T59 — Staging deployment.
T60 — Production gate.

Recovery

T61 — Database backup.
T62 — Configuration recovery.
T63 — Restore procedure.
T64 — Restore testing.
T65 — RPO validation.
T66 — RTO validation.

Operations

T67 — Authentication outage runbook.
T68 — Database outage runbook.
T69 — Redis outage runbook.
T70 — Event outage runbook.
T71 — Token compromise runbook.
T72 — Tenant isolation incident runbook.
T73 — Rollback runbook.
T74 — Outbox/DLQ recovery runbook.

66. Acceptance Criteria
AC-001 — Stateless

Given múltiples instancias Identity
When un usuario realiza requests consecutivas
Then las requests deben poder ser atendidas por distintas instancias sin perder contexto crítico.

AC-002 — Configuration

Given un ambiente determinado
When Identity inicia
Then debe cargar configuración específica del ambiente sin utilizar valores hardcodeados sensibles.

AC-003 — Secrets

Given un secret requerido
When Identity inicia
Then debe obtenerlo desde el mecanismo de secret management configurado.

AC-004 — Missing Secret

Given un secret crítico ausente
When Identity inicia
Then debe fallar de manera segura y no iniciar un runtime parcialmente inseguro.

AC-005 — Liveness

Given un proceso Identity funcionando
When se consulta liveness
Then debe indicar correctamente si el proceso está vivo.

AC-006 — Readiness

Given una instancia incapaz de procesar correctamente requests
When se evalúa readiness
Then la instancia debe poder retirarse del tráfico.

AC-007 — Graceful Shutdown

Given una instancia recibe SIGTERM
When inicia shutdown
Then debe dejar de aceptar nuevas operaciones y finalizar correctamente las operaciones activas según la política.

AC-008 — Horizontal Scaling

Given tres instancias Identity
When reciben tráfico simultáneamente
Then deben operar contra el estado compartido sin inconsistencias.

AC-009 — Database Failure

Given PostgreSQL está indisponible
When Identity recibe una operación dependiente de DB
Then no debe responder falsamente como exitosa.

AC-010 — Redis Failure

Given Redis está indisponible
When se ejecuta una operación de seguridad
Then no debe producirse fail-open.

AC-011 — Event Platform Failure

Given Event Platform está indisponible
When se genera un Domain Event
Then el evento debe permanecer recuperable mediante Outbox.

AC-012 — Deployment

Given una nueva versión
When se realiza un rolling deployment
Then Identity debe permanecer disponible dentro de los objetivos establecidos.

AC-013 — Rollback

Given una nueva versión defectuosa
When se activa rollback
Then debe poder restaurarse una versión operativa anterior siguiendo el procedimiento definido.

AC-014 — Observability

Given una request Identity
When ocurre un error
Then debe poder correlacionarse mediante logs/metrics/traces según corresponda.

AC-015 — Security

Given un container Identity
When se ejecuta en Production
Then no debe contener secrets embebidos ni ejecutarse con privilegios innecesarios.

AC-016 — Recovery

Given una pérdida simulada del datastore
When se ejecuta restore
Then Identity debe poder recuperarse dentro de los objetivos RTO/RPO establecidos.

67. Definition of Done
 Runtime architecture definida.
 Container strategy definida.
 Stateless requirements implementados.
 Environment separation implementada.
 Configuration management implementado.
 Secret management implementado.
 Key management integrado.
 JWT key rotation soportada.
 PostgreSQL runtime configurado.
 Database migrations integradas.
 Redis configurado.
 Event Platform integrada.
 Outbox Publisher operacional.
 Liveness implementado.
 Readiness implementado.
 Dependency health implementado.
 Graceful shutdown implementado.
 Horizontal scaling validado.
 Deployment strategy implementada.
 Rollback probado.
 Structured logging implementado.
 Metrics implementadas.
 Distributed tracing implementado.
 Security monitoring implementado.
 Alerts configuradas.
 Container hardened.
 Supply chain scanning implementado.
 CI/CD pipeline configurado.
 Quality gates configurados.
 Backup configurado.
 Restore probado.
 RTO validado.
 RPO validado.
 Operational runbooks creados.
 Security incident procedures creados.
 Documentación actualizada.
 Traceability completa.
68. Dependencias
Upstream
ESP-0001

ADR-IDENTITY-006
ADR-IDENTITY-007
ADR-IDENTITY-008
ADR-IDENTITY-010
ADR-IDENTITY-011

IS-IDENTITY-001
IS-IDENTITY-002
IS-IDENTITY-003
IS-IDENTITY-004
IS-IDENTITY-005
IS-IDENTITY-006
IS-IDENTITY-007
IS-IDENTITY-008
IS-IDENTITY-009
IS-IDENTITY-010
IS-IDENTITY-011
IS-IDENTITY-012
IS-IDENTITY-013
IS-IDENTITY-014
IS-IDENTITY-015
IS-IDENTITY-016
Downstream
Identity Implementation
        ↓
Platform Integration
        ↓
CI/CD
        ↓
Deployment
        ↓
Production Operations
69. Traceability
Nivel	Referencia
Blueprint	Runtime & Deployment
ESP	ESP-0001
ADR principal	ADR-IDENTITY-011
Persistence	ADR-IDENTITY-006
Events	ADR-IDENTITY-007
Cryptography	ADR-IDENTITY-008
Testing	ADR-IDENTITY-010
IS	IS-IDENTITY-017
Technical Tasks	T01–T74
Source Code	Pendiente
Tests	IS-IDENTITY-016
Deployment	Esta IS

La trazabilidad continúa siguiendo:

ESP → IS → Technical Tasks → Source Code → Testing → Deployment.

70. ADRs pendientes

Esta IS deja identificadas decisiones que deben cerrarse antes de implementar el runtime productivo.

ADR-IDENTITY-024 — Container & Runtime Platform

Definir:

container base;
runtime;
orchestration;
resource policies.
ADR-IDENTITY-025 — Deployment Strategy

Definir:

rolling;
blue/green;
canary;
promotion;
rollback.
ADR-IDENTITY-026 — Secrets & Key Management Platform

Definir:

Secret Manager;
KMS;
key rotation;
access policies.
ADR-IDENTITY-027 — Identity Observability Stack

Definir:

logs;
metrics;
tracing;
alerting;
dashboards.
ADR-IDENTITY-028 — Identity Disaster Recovery

Definir:

backups;
replication;
restore;
RTO/RPO implementation;
regional recovery.
ADR-IDENTITY-029 — Identity Autoscaling

Definir:

metrics;
min/max replicas;
scaling thresholds;
cooldown;
workload limits.
71. Estado de Identity

Con IS-IDENTITY-017 cerramos el primer bloque completo de Implementation Stories de Identity:

┌──────────────────────────────────────────────┐
│              IDENTITY DOMAIN                 │
├──────────────────────────────────────────────┤
│ 001 Foundation                               │
│ 002 User & Account Management                │
│ 003 Organization & Tenant Management         │
│ 004 Membership Management                    │
│ 005 Authentication                           │
│ 006 Authorization & Permission Evaluation    │
│ 007 Role & Permission Management             │
│ 008 Session Management                       │
│ 009 Refresh Token Lifecycle                  │
│ 010 Multi-Factor Authentication              │
│ 011 Security Policy Engine                   │
│ 012 Audit & Security Events                  │
│ 013 Domain Events & Outbox                   │
│ 014 Identity API Contracts                   │
│ 015 Identity Persistence                     │
│ 016 Identity Testing & Quality               │
│ 017 Identity Runtime & Deployment            │
└──────────────────────────────────────────────┘

Esto nos deja Identity definido desde dominio hasta operación, sin saltar directamente al código.

72. Punto de control antes de Source Code

A partir de aquí, el flujo correcto del Blueprint es no comenzar a programar directamente todos los endpoints.

Primero debemos consolidar las decisiones pendientes que afectan la implementación:

Implementation Stories
        ↓
Pending ADRs
        ↓
Technical Specifications
        ↓
Technical Tasks
        ↓
Source Code
        ↓
Testing
        ↓
Deployment

El Blueprint establece precisamente que ninguna implementación debe saltarse este lifecycle.

Por tanto, IS-IDENTITY-017 completa el bloque inicial de Identity, pero todavía quedan por formalizar los ADRs técnicos pendientes antes de convertir estas IS en código productivo.

Próximo paso recomendado

El siguiente trabajo debería ser consolidar los ADRs pendientes de Identity (ADR-IDENTITY-012 en adelante) y cerrar las decisiones de:

Event Platform → Database Schema → Concurrency → API details → MFA → Secrets/KMS → Runtime → Deployment → Observability → Disaster Recovery.

Después de eso podremos pasar a la siguiente etapa:

Identity Technical Tasks → estructura real del proyecto → implementación del Domain Layer → Application Layer → Infrastructure → API → tests, manteniendo la trazabilidad completa del Blueprint.
