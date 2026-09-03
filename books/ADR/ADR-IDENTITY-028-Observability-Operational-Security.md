ADR-IDENTITY-028 — Observability & Operational Security

Estado: Proposed
Versión: 1.0
Fecha: 2026-09-03
Dominio: Identity / Platform / Security
Prioridad: Critical
Tipo: Architecture Decision Record

1. Título

ADR-IDENTITY-028 — Observability & Operational Security

Define la estrategia arquitectónica de observabilidad y seguridad operacional para el dominio Identity de EVOXA.

Incluye:

logging;
métricas;
distributed tracing;
audit trail;
security events;
correlation/causation/trace IDs;
monitoreo de autenticación;
monitoreo de autorización;
MFA;
sesiones;
Refresh Tokens;
políticas de seguridad;
eventos;
detección de anomalías;
alertas;
incident response;
protección de información sensible;
AI operational security.

Este ADR se apoya en la arquitectura de observabilidad y seguridad establecida por el Blueprint y en los ADR anteriores de Identity.

2. Contexto

EVOXA está definido como un sistema:

Observable
Secure by Design
Zero Trust
Event-Driven
Cloud-Ready
AI-Native

La observabilidad no debe limitarse a revisar logs cuando existe un problema.

Identity requiere poder responder preguntas como:

¿Quién intentó autenticarse?
¿La autenticación tuvo éxito?
¿Por qué fue rechazada?
¿Qué autorización se evaluó?
¿Qué política intervino?
¿Qué tenant estaba involucrado?
¿Qué sesión estaba activa?
¿Se detectó reuse de Refresh Token?
¿Hubo intentos MFA fallidos?
¿Qué servicio produjo el evento?
¿Qué ocurrió antes y después?

Pero esta trazabilidad no puede conseguirse almacenando indiscriminadamente información sensible.

3. Problema

Identity maneja información de altísima sensibilidad:

Passwords
Access Tokens
Refresh Tokens
MFA Secrets
Recovery Codes
Private Keys
Session Information
Security Policies
User/Organization Context

Un sistema de observabilidad mal diseñado podría convertirse en una segunda superficie de ataque.

Por ejemplo:

Request
   ↓
Application
   ↓
Log
   ↓
Refresh Token exposed

o:

MFA verification
   ↓
Trace
   ↓
TOTP secret exposed

Por tanto, EVOXA necesita observabilidad sin exposición de secretos.

4. Decisión

EVOXA adoptará una estrategia de observabilidad basada en cinco pilares:

┌──────────────────────────────────┐
│        IDENTITY OBSERVABILITY    │
├──────────────────────────────────┤
│  Logs                            │
│  Metrics                         │
│  Traces                          │
│  Audit Events                    │
│  Security Events                 │
└──────────────────────────────────┘

Estos mecanismos serán complementarios y no intercambiables.

La estrategia deberá proporcionar:

visibilidad operacional;
trazabilidad distribuida;
detección de problemas;
auditoría;
investigación de seguridad;
capacidad de respuesta;
métricas de SLO;
seguridad de AI/Agents.
5. Separación de tipos de información

EVOXA distinguirá claramente:

Technical Logs
      ≠
Metrics
      ≠
Distributed Traces
      ≠
Audit Events
      ≠
Security Events

Cada uno tiene un propósito diferente.

6. Technical Logs

Los logs técnicos describen el comportamiento del sistema.

Ejemplos:

HTTP request completed
Database connection failed
Redis unavailable
Outbox worker retry
External provider timeout

Su objetivo es facilitar:

debugging;
troubleshooting;
operación;
diagnóstico.

No deben utilizarse como sustituto de Audit Events.

7. Metrics

Las métricas proporcionan valores agregados.

Ejemplo:

authentication_failure_total
authorization_denied_total
mfa_failure_total
refresh_reuse_detected_total

No deben almacenar datos personales innecesarios.

8. Distributed Tracing

Los traces permiten seguir una operación a través de múltiples servicios.

Conceptualmente:

Client
  │
  ▼
API
  │
  ▼
Identity
  │
  ├── PostgreSQL
  ├── Redis
  ├── Policy Engine
  └── Event Platform

Todo el recorrido debe poder correlacionarse cuando corresponda.

9. Audit Events

Los Audit Events representan hechos relevantes desde el punto de vista de gobernanza y seguridad.

Ejemplo:

UserSuspended
RoleAssigned
PermissionChanged
SessionRevoked
MFAFactorRevoked
PolicyUpdated

Su propósito es responder:

¿Qué cambio ocurrió, quién lo realizó, sobre qué recurso y bajo qué contexto?

10. Security Events

Los Security Events representan hechos relevantes para seguridad.

Ejemplos:

AuthenticationFailed
AuthorizationDenied
TenantAccessDenied
RefreshTokenReuseDetected
MFABruteForceDetected
PrivilegeEscalationDetected
PolicyViolationDetected

Pueden alimentar:

detección;
alertas;
respuesta;
SIEM;
análisis de riesgo.
11. Correlation ID

Toda request relevante deberá disponer de:

correlation_id

cuando corresponda.

Ejemplo:

Client
  │
  │ correlation_id = C123
  ▼
API
  │
  ▼
Identity
  │
  ├── PostgreSQL
  └── Event

Esto permite reconstruir una operación.

12. Trace ID

Cuando se utilice distributed tracing:

trace_id

permitirá relacionar spans pertenecientes a la misma operación distribuida.

Los traces no deben incluir secretos.

13. Causation ID

Cuando exista una relación causal entre operaciones/eventos:

causation_id

permitirá identificar qué evento/acción provocó otro evento.

Ejemplo:

UserSuspended
      │
      ▼
SessionsRevoked

El segundo evento puede referenciar causalmente al primero.

14. Identidad del actor

Los eventos deberán poder identificar conceptualmente:

USER
SERVICE
SYSTEM
AI_AGENT

cuando corresponda.

Para un AI Agent deberá poder determinarse:

agent identity
+
authorization context
+
tenant
+
operation

sin almacenar información innecesaria.

15. No registrar secretos

Es un principio absoluto.

Nunca deben aparecer en:

logs
metrics
traces
audit events
security events
error messages

elementos como:

password
password_hash
access_token
refresh_token
TOTP secret
OTP
recovery code
private key
KMS credentials
API secret
16. Redaction

La plataforma deberá disponer de mecanismos de redacción.

Conceptualmente:

Request
   ↓
Sensitive Field Detection
   ↓
Redaction
   ↓
Log / Trace

Ejemplo:

Authorization: Bearer [REDACTED]

Nunca:

Authorization: Bearer eyJ....
17. Structured Logging

Los logs deberán ser estructurados.

Conceptualmente:

{
  "timestamp": "...",
  "level": "INFO",
  "service": "identity",
  "event": "authentication_completed",
  "correlation_id": "...",
  "trace_id": "...",
  "tenant_id": "...",
  "actor_id": "...",
  "result": "SUCCESS"
}

La estructura exacta deberá definirse en Engineering Standards.

18. Logging Levels

Se utilizarán niveles de logging apropiados:

DEBUG
INFO
WARN
ERROR

No se deberá utilizar DEBUG como mecanismo para registrar secretos.

En Production, el nivel por defecto deberá minimizar información innecesaria.

19. Error Logging

Los errores deberán registrar suficiente contexto para diagnóstico.

Pero no:

password
token
MFA secret
private key

El error público de API debe ser distinto del detalle interno del log.

Client
   ↓
Safe Error

Internal Log
   ↓
Diagnostic Context
20. Authentication Observability

Identity deberá producir métricas sobre:

authentication_attempts
authentication_success
authentication_failure
authentication_blocked
authentication_mfa_required
authentication_latency

Esto permitirá detectar:

degradación;
ataques;
errores de configuración;
problemas de proveedores.
21. Authentication Security Monitoring

Debe observarse comportamiento anómalo como:

Many failures
      ↓
Same user

Many users
      ↓
Same source/context

Repeated authentication
      ↓
Suspicious pattern

Las reglas exactas de detección serán definidas por Security Operations/Policy.

22. Authorization Observability

Se deberán monitorizar:

authorization_allowed
authorization_denied
tenant_access_denied
permission_denied
policy_violation
privilege_escalation_detected

Esto permitirá detectar tanto problemas legítimos de permisos como comportamiento sospechoso.

23. Tenant Isolation Monitoring

Los intentos de acceso cross-tenant deben tener visibilidad especial.

Ejemplo:

Tenant A
   │
   ▼
Request resource Tenant B
   │
   ▼
TenantAccessDenied

Este tipo de evento deberá poder generar alertas según riesgo.

24. Session Observability

Se deberán monitorizar:

sessions_created
sessions_activated
sessions_expired
sessions_revoked
sessions_revoked_all

También:

sesiones concurrentes;
sesiones por usuario;
revocaciones masivas;
patrones anómalos.
25. Refresh Token Monitoring

Eventos especialmente importantes:

refresh_success
refresh_failure
refresh_expired
refresh_revoked
refresh_rotation
refresh_reuse_detected

RefreshTokenReuseDetected deberá recibir tratamiento prioritario.

26. MFA Monitoring

Métricas conceptuales:

mfa_challenge_created
mfa_challenge_verified
mfa_challenge_failed
mfa_challenge_expired
mfa_factor_enrolled
mfa_factor_revoked
mfa_recovery_requested
mfa_recovery_success
mfa_recovery_denied

También deben monitorizarse:

mfa_bruteforce
mfa_replay
mfa_provider_failure
27. Security Policy Observability

Security Policy Engine deberá proporcionar visibilidad sobre:

ALLOW
DENY
CHALLENGE
REAUTHENTICATE
REQUIRE_MFA
REVOKE

Las decisiones importantes deben poder relacionarse con:

policy_id
policy_version
reason
tenant
actor
resource
operation

sin exponer información sensible.

28. Policy Debugging

En ambientes autorizados, deberá ser posible investigar por qué una política produjo una decisión.

Ejemplo:

Authorization Request
        ↓
Policy A
        ↓
ALLOW
        ↓
Policy B
        ↓
DENY
        ↓
Final Decision
        ↓
DENY

La información detallada de evaluación deberá estar restringida a operadores autorizados.

29. Audit Access

Los propios Audit Events son información sensible.

El acceso debe estar protegido mediante:

Authentication
+
Authorization
+
Tenant Isolation
+
Security Policy

No se debe permitir:

audit.read

como permiso universal sin considerar el tenant y sensibilidad.

30. Security Event Access

Security Events pueden contener información operacional sensible.

Por tanto, el acceso debe ser aún más restringido.

Podrán existir permisos conceptuales:

security_events.read
security_events.read_sensitive
security_events.export
security_events.admin

Los nombres finales deberán definirse en el catálogo de permisos.

31. Audit de acceso al Audit

Una operación como:

GET /audit-events

también puede ser auditada.

Esto es importante porque permite conocer:

Who accessed audit data?
When?
Which tenant?
What scope?
32. Exportación

Las capacidades de exportación de:

Audit Events;
Security Events;

deberán estar explícitamente autorizadas.

Las exportaciones deben:

registrar actor;
registrar tenant;
registrar scope;
registrar timestamp;
aplicar filtros de seguridad;
evitar exposición de secretos.
33. Data Minimization

La observabilidad seguirá el principio:

registrar lo necesario para operar y proteger el sistema, no todo lo disponible.

No se deben copiar objetos completos simplemente porque estén disponibles.

Ejemplo:

Preferible:

resource_type = User
resource_id = 123

en lugar de almacenar todo el objeto User.

34. PII

La información personal deberá minimizarse.

Cuando sea suficiente:

user_id

en lugar de:

full_name
email
phone

Especialmente en métricas.

35. Health Data

Si otros dominios de EVOXA manejan datos de salud, Identity no deberá replicarlos innecesariamente en observabilidad.

Identity debe utilizar referencias y contexto mínimo.

Esto mantiene separación entre:

Identity

y:

Health / Training / Nutrition / Medical domains
36. Logs y Tenant Isolation

Los logs deberán contener contexto tenant cuando sea apropiado para investigación.

Pero esto no significa que todos los operadores deban poder consultar todos los tenants.

La plataforma de observabilidad también deberá aplicar controles de acceso.

37. Operational Security

La seguridad operacional incluye:

monitoring;
alerting;
incident response;
key compromise response;
token compromise response;
tenant isolation incident response;
service outage response.

No se limita al application code.

38. Security Alerts

Se podrán definir alertas para:

High authentication failures
High authorization denials
Cross-tenant attempts
Refresh token reuse
MFA brute force
Privilege escalation
Policy violations
Unusual session creation
Mass session revocation
KMS failures
Secret access anomalies
39. Severity

Security Events deberán disponer de una clasificación de severidad.

Conceptualmente:

INFO
LOW
MEDIUM
HIGH
CRITICAL

La severidad exacta de cada evento será definida por el catálogo de Security Events.

40. Incident Correlation

Un incidente puede producir múltiples señales:

AuthenticationFailure
       ↓
MFAFailure
       ↓
AuthorizationDenied
       ↓
RefreshTokenReuseDetected

Los correlation/trace/causation IDs deben permitir relacionarlas.

Esto es especialmente importante para incident response.

41. Security Incident Flow

Conceptualmente:

Security Event
      ↓
Detection
      ↓
Classification
      ↓
Alert
      ↓
Investigation
      ↓
Response
      ↓
Containment
      ↓
Recovery
      ↓
Post-Incident Review
42. Automated Response

Algunos eventos podrán generar respuestas automáticas.

Por ejemplo:

RefreshTokenReuseDetected
        ↓
Revoke Token Family
        ↓
Revoke Session
        ↓
Security Event
        ↓
Alert

La automatización deberá estar gobernada por Security Policy.

No todos los eventos deben producir acciones automáticas.

43. AI Security Monitoring

Las operaciones realizadas por AI Agents deberán ser observables.

Ejemplo:

AI Agent
   ↓
Tool Call
   ↓
Authorization
   ↓
Operation
   ↓
Audit Event

Debe poder determinarse:

qué agente actuó;
qué usuario/actor lo autorizó;
qué tenant;
qué recurso;
qué operación;
qué policy;
resultado.
44. AI Agent Anomaly Detection

La arquitectura podrá posteriormente detectar:

excesivas operaciones;
intentos fuera de scope;
patrones inusuales;
escalamiento;
cross-tenant attempts;
operaciones repetidas.

Pero el mecanismo de detección AI no podrá convertirse en una vía para saltarse Identity.

45. Observability de Event Platform

Identity deberá monitorizar:

Outbox backlog
Publish latency
Publish failures
Retry count
DLQ count
Consumer lag
Replay jobs
Event processing failures

Esto se alinea con:

ADR-IDENTITY-018;
ADR-IDENTITY-019;
ADR-IDENTITY-020;
ADR-IDENTITY-021;
ADR-IDENTITY-022.
46. Outbox Monitoring

Especialmente:

outbox_pending
outbox_processing
outbox_failed
outbox_dlq
outbox_publish_latency

Un crecimiento sostenido del backlog debe generar alerta.

47. Database Observability

Identity deberá monitorizar:

connection pool;
latency;
errors;
locks;
deadlocks;
slow queries;
transaction duration;
migration status.

No deberán registrarse credenciales.

48. Redis Observability

Debe observarse:

disponibilidad;
latencia;
connection errors;
memory;
eviction;
command latency.

Redis continúa siendo auxiliar y no debe convertirse en una dependencia silenciosa para garantías críticas.

49. KMS / Secret Manager Observability

Se deberán observar:

secret access failures
KMS operation failures
key rotation failures
key access denied
secret rotation failures

Esto complementa:

ADR-IDENTITY-027 — Runtime Configuration, Secrets & KMS

50. Availability

Identity debe contribuir al objetivo global:

Availability ≥99.9%

La observabilidad debe permitir detectar degradaciones antes de que se conviertan en una indisponibilidad completa.

51. Performance Monitoring

Deberán monitorizarse:

API latency
Authentication latency
Authorization latency
MFA latency
Refresh latency
Database latency
Redis latency
Event publication latency

Objetivos previamente establecidos:

API P95 < 300 ms
Authentication P95 < 500 ms
AI P95 < 2 s

Estos objetivos deben validarse en ambientes reales de carga.

52. SLO / SLA / Error Budget

La plataforma deberá poder calcular:

Availability
Latency
Error Rate

y utilizar estos valores para gestionar:

SLO
Error Budget
Operational Priorities

Los valores definitivos de SLO por endpoint quedan pendientes.

53. Alert Fatigue

No toda métrica debe generar una alerta.

Las alertas deberán priorizar:

Actionable
Relevant
Security-sensitive
Availability-impacting

El objetivo es evitar:

Too many alerts
      ↓
Operators ignore alerts
54. Alert Routing

Las alertas podrán dirigirse según categoría:

Application
Infrastructure
Security
Database
Event Platform
Identity
Incident Response

La herramienta concreta queda pendiente.

55. Retención

Logs, metrics, traces, audit y security events no necesariamente tendrán la misma retención.

Se seguirá:

ADR-IDENTITY-013 — Data Retention & Lifecycle

Conceptualmente:

Logs
Metrics
Traces
Audit
Security Events

tendrán políticas diferentes según:

utilidad;
costo;
compliance;
investigación;
sensibilidad.
56. Inmutabilidad de Audit

Los Audit Events deben mantenerse:

Append Only
Immutable

No deben modificarse como parte de operaciones normales.

Esto se alinea con:

IS-IDENTITY-012

57. Integridad de Audit

La arquitectura podrá evolucionar hacia mecanismos de:

tamper evidence;
hashes;
firmas;
almacenamiento inmutable;
external archival.

No se fija aquí una tecnología específica.

58. Access Control del sistema de observabilidad

No basta con proteger Identity.

También debe protegerse:

Logging Platform
Metrics Platform
Tracing Platform
Audit Store
Security Event Store

Un administrador de observabilidad no debe automáticamente obtener permisos de negocio sobre todos los tenants.

59. Production Debugging

Los mecanismos de debugging en Production deberán evitar:

dumping de secretos;
dumping de sesiones;
dumping de tokens;
acceso indiscriminado a datos de usuario.

El debugging debe realizarse con herramientas seguras y permisos elevados explícitos.

60. Runbooks

Identity deberá disponer de runbooks para incidentes críticos.

Como mínimo:

Authentication outage
Authorization outage
Database outage
Redis outage
Event Broker outage
Refresh Token compromise
JWT key compromise
MFA provider outage
Tenant isolation incident
Secret Manager outage
KMS outage
Outbox backlog
DLQ growth
61. Disaster Response

Observability debe integrarse con:

ADR-IDENTITY-029 — Disaster Recovery & Backup

La recuperación debe permitir comprobar:

Service recovered
+
Database recovered
+
Secrets available
+
Keys available
+
Events recovered
+
Observability restored
62. Security Testing

Deberán probarse escenarios como:

secret leakage;
token leakage;
PII leakage;
cross-tenant logs;
unauthorized audit access;
unauthorized security-event access;
trace leakage;
malicious correlation IDs;
log injection;
alert bypass.
63. Log Injection

Los valores controlados por el cliente no deben poder manipular la estructura de los logs.

Ejemplo:

username = "...\nFAKE SECURITY EVENT..."

debe ser sanitizado correctamente.

64. Correlation ID Security

Los clientes podrán proporcionar correlation IDs cuando la arquitectura lo permita, pero el sistema debe:

validar formato;
limitar tamaño;
evitar control characters;
impedir log injection.

El servidor podrá reemplazar IDs inválidos.

65. High Cardinality

No se deben utilizar valores de alta cardinalidad indiscriminadamente como labels de métricas.

Por ejemplo, evitar:

metric{user_id="123456789"}

para millones de usuarios.

Esto puede destruir la eficiencia del sistema de métricas.

66. Privacy

La observabilidad deberá aplicar:

Privacy by Design
Data Minimization
Least Privilege
Purpose Limitation

La información debe recopilarse porque existe una razón operacional o de seguridad.

67. Consecuencias positivas
Operación
troubleshooting más rápido;
visibilidad end-to-end;
detección temprana;
mejores SLO.
Seguridad
detección de ataques;
trazabilidad;
incident response;
visibilidad de privilegios;
protección de tenants.
Arquitectura
logs/traces/events coherentes;
correlación distribuida;
mejor debugging;
integración con Event Platform.
AI
operaciones AI auditables;
detección de anomalías;
control de agentes.
68. Consecuencias negativas

La estrategia introduce:

mayor volumen de datos;
costos de almacenamiento;
complejidad operacional;
necesidad de redaction;
gestión de retención;
alert tuning;
control de acceso sobre observabilidad.

Una observabilidad excesiva también puede convertirse en riesgo de privacidad.

69. Alternativas consideradas
Alternativa A — Solo Logs

Rechazada.

No proporciona suficiente:

métricas;
trazabilidad;
auditoría;
security monitoring.
Alternativa B — Audit como único mecanismo

Rechazada.

Audit no reemplaza logs técnicos ni métricas.

Alternativa C — Registrar todo

Rechazada.

Viola:

seguridad;
privacidad;
minimización de datos;
costos razonables.
Alternativa D — SIEM como única fuente

Rechazada.

SIEM puede formar parte del ecosistema, pero no reemplaza observabilidad operacional.

70. No negociables
Logs, metrics, traces, audit y security events son conceptos separados.
Los secretos nunca deben aparecer en observabilidad.
Debe existir redaction.
Debe existir correlation ID.
Debe existir trace ID cuando corresponda.
Debe existir causation ID para relaciones causales.
Authentication debe ser observable.
Authorization debe ser observable.
Tenant isolation debe ser observable.
MFA debe ser observable.
Refresh Token reuse debe ser observable.
Security Policy decisions deben ser trazables.
Audit debe ser append-only.
Security Events deben tener clasificación de severidad.
El acceso a Audit/Security Events debe estar autorizado.
Las exportaciones deben quedar auditadas.
AI Agent actions deben ser trazables.
No debe almacenarse PII innecesaria.
No debe utilizarse información de alta cardinalidad indiscriminadamente en métricas.
Los sistemas de observabilidad también deben estar protegidos.
Deben existir alertas accionables.
Deben existir runbooks.
KMS/Secret Manager deben ser observables.
Event Platform/Outbox debe ser observable.
Deben existir pruebas de leakage.
Debe existir tenant-aware observability.
La observabilidad no puede convertirse en bypass de Identity.
71. Decisiones pendientes

Quedan pendientes:

Logging
formato final;
collector;
storage;
niveles Production;
sampling.
Metrics
stack;
nombres definitivos;
cardinalidad;
dashboards;
SLO específicos.
Tracing
tecnología;
sampling;
propagación;
retention.
Audit
almacenamiento;
tamper evidence;
archival;
retention.
Security Events
catálogo definitivo;
severidad;
routing;
SIEM integration.
Alerting
thresholds;
escalation;
on-call;
incident management.
Runtime
OpenTelemetry u otra tecnología;
collector architecture;
multi-region.

Estas decisiones se resolverán en los Engineering Standards y en los Technical Tasks de Identity Runtime/Observability.

72. Criterios de aceptación arquitectónica

Este ADR podrá considerarse satisfecho cuando:

 Existan logs estructurados.
 Existan métricas.
 Existan distributed traces.
 Existan Audit Events.
 Existan Security Events.
 Exista correlation ID.
 Exista trace ID.
 Exista causation ID cuando corresponda.
 Exista redaction automática.
 No existan secretos en logs.
 No existan secretos en traces.
 No existan secretos en eventos.
 Authentication sea observable.
 Authorization sea observable.
 MFA sea observable.
 Refresh Token reuse sea observable.
 Security Policy sea trazable.
 Tenant isolation sea observable.
 AI actions sean auditables.
 Existan métricas de PostgreSQL.
 Existan métricas de Redis.
 Existan métricas de Event Platform.
 Existan métricas de KMS/Secret Manager.
 Existan alertas.
 Existan runbooks.
 Exista control de acceso a observabilidad.
 Exista política de retención.
 Existan pruebas de secret/PII leakage.
 Existan dashboards operacionales.
 Exista estrategia de incident response.
73. Trazabilidad
Blueprint
Observable
Secure by Design
Zero Trust
Privacy by Design
Event-Driven
AI-Native
Cloud-Ready
Engineering Specification
ESP-0001 — Identity
Implementation Stories
IS-IDENTITY-005
Authentication

IS-IDENTITY-006
Authorization & Permission Evaluation

IS-IDENTITY-008
Session Management

IS-IDENTITY-009
Refresh Token Lifecycle

IS-IDENTITY-010
Multi-Factor Authentication

IS-IDENTITY-011
Security Policy Engine

IS-IDENTITY-012
Audit & Security Events

IS-IDENTITY-013
Domain Events & Outbox

IS-IDENTITY-016
Identity Testing & Quality

IS-IDENTITY-017
Identity Runtime & Deployment
ADRs relacionados
ADR-IDENTITY-007
Events & Outbox

ADR-IDENTITY-008
Cryptographic Key Management

ADR-IDENTITY-013
Data Retention & Lifecycle

ADR-IDENTITY-014
Encryption & Secret Protection

ADR-IDENTITY-018
Event Broker

ADR-IDENTITY-019
Event Partitioning & Ordering

ADR-IDENTITY-020
Event Schema Compatibility

ADR-IDENTITY-021
Event Replay

ADR-IDENTITY-022
Consumer Idempotency

ADR-IDENTITY-025
Refresh Token Cryptography & Concurrency

ADR-IDENTITY-027
Runtime Configuration, Secrets & KMS
74. Relación con los siguientes ADR

La secuencia continúa:

ADR-IDENTITY-025
Refresh Token Cryptography & Concurrency
        │
        ▼
ADR-IDENTITY-026
API Pagination, Filtering & Idempotency
        │
        ▼
ADR-IDENTITY-027
Runtime Configuration, Secrets & KMS
        │
        ▼
ADR-IDENTITY-028
Observability & Operational Security
        │
        ▼
ADR-IDENTITY-029
Disaster Recovery & Backup
        │
        ▼
ADR-IDENTITY-030
Autoscaling & Capacity

Con ADR-IDENTITY-028 queda definido el modelo de observabilidad + seguridad operacional de Identity, manteniendo una separación fundamental entre logs técnicos, métricas, trazas, auditoría y eventos de seguridad, y estableciendo como principio absoluto que la observabilidad nunca debe convertirse en una nueva fuente de exposición de secretos.

El siguiente ADR lógico es ADR-IDENTITY-029 — Disaster Recovery & Backup, donde cerraremos la estrategia de recuperación de Identity, PostgreSQL, Redis, Outbox/Event Platform, secretos, claves criptográficas y objetivos RTO < 1h / RPO < 15m.
