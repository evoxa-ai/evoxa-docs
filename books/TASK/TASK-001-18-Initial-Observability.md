TASK-001-18 — Initial Observability

Implementation Story: IS-001 — Identity Domain Foundation
ESP: ESP-0001/14 — Identity Implementation Requirements
Related ESP: ESP-0001/02, ESP-0001/03, ESP-0001/09, ESP-0001/10, ESP-0001/12, ESP-0001/13, ESP-0001/15
Related ADRs: ADR-IDENTITY-001, 007, 014, 027, 028, 031, 032, 054, 055
Priority: 🔴 Critical
Status: 🟢 Ready for Implementation
Task Type: Observability / Platform Infrastructure

1. Objetivo

Establecer la capa inicial de observabilidad de EVOXA Identity, proporcionando una base uniforme para conocer qué está ocurriendo dentro de la aplicación sin mezclar:

Application Logs
Audit Events
Security Events
Distributed Tracing
Metrics

La observabilidad debe permitir responder rápidamente:

Qué ocurrió, cuándo ocurrió, dónde ocurrió, quién lo ejecutó, sobre qué tenant, cuál fue el resultado y cómo seguir la operación a través del sistema.

Esta tarea establece la infraestructura inicial.

No implementa todavía toda la observabilidad avanzada de EVOXA.

2. Principio fundamental

La observabilidad no debe modificar el comportamiento de negocio.

Business Operation
       │
       ├──────────► Logs
       │
       ├──────────► Metrics
       │
       ├──────────► Traces
       │
       └──────────► Audit/Security Events

Pero:

Observability
       ≠
Authorization
       ≠
Business Logic

Un fallo de logging nunca debe convertir:

DENY

en:

ALLOW
3. Los cinco pilares

Para Identity se mantendrá la separación:

                 Observability
                      │
       ┌──────────────┼──────────────┐
       ▼              ▼              ▼
     Logs          Metrics         Traces
       │
       ├──────────────► Audit
       │
       └──────────────► Security Events

Aunque todos pueden compartir correlation/trace information, son conceptos diferentes.

4. Application Logs

Los logs sirven para conocer el comportamiento operacional de la aplicación.

Ejemplo:

{
  "level": "INFO",
  "message": "Application request completed",
  "request_id": "...",
  "correlation_id": "...",
  "trace_id": "...",
  "operation": "health_check",
  "duration_ms": 12
}

Los logs no deben convertirse en una segunda base de auditoría.

5. Audit Events

Los Audit Events representan evidencia de acciones relevantes.

Ejemplos futuros:

USER_CREATED
USER_UPDATED
MEMBERSHIP_CREATED
ROLE_ASSIGNED
PERMISSION_CHANGED
SESSION_REVOKED
MFA_FACTOR_REVOKED
POLICY_ACTIVATED

Estos eventos tendrán el modelo definido en ESP-0001/09 y las reglas de auditoría correspondientes.

6. Security Events

Los Security Events representan señales de seguridad.

Ejemplos:

AUTHENTICATION_FAILURE
MFA_FAILURE
REFRESH_TOKEN_REUSE
ACCOUNT_SUSPENDED
SESSION_REVOKED
SUSPICIOUS_ACTIVITY
TENANT_ACCESS_DENIED

Deberán distinguirse de un log operacional normal.

7. Distributed Tracing

El tracing permite seguir una operación:

HTTP Request
     │
     ▼
API
     │
     ▼
Application
     │
     ▼
Database
     │
     ▼
Outbox
     │
     ▼
Event Platform
     │
     ▼
Consumer

utilizando:

request_id
correlation_id
trace_id
causation_id

según corresponda.

8. Contexto transversal

TASK-001-14 estableció TraceContext.

Esta tarea debe reutilizar ese mecanismo.

No se deben crear sistemas paralelos.

Conceptualmente:

@dataclass(frozen=True)
class TraceContext:
    request_id: str
    correlation_id: str
    trace_id: str
    causation_id: str | None = None

El contexto será accesible de forma segura para la ejecución async.

9. ContextVar y concurrencia

Para contexto request-scoped se recomienda utilizar mecanismos equivalentes a contextvars.ContextVar.

Esto permite:

Request A ──► trace-A
Request B ──► trace-B
Request C ──► trace-C

sin que:

Request A

termine utilizando accidentalmente el contexto de:

Request B

No se utilizarán variables globales mutables para esto.

10. Actor y Tenant Context

Cuando corresponda, los logs operacionales podrán incluir:

actor_type
actor_id
tenant_id
organization_id

pero únicamente cuando sea apropiado y seguro.

El sistema debe mantener separados:

TraceContext
ActorContext
TenantContext
SecurityContext
11. Actor Types

El modelo definido para EVOXA contempla:

USER
SERVICE
SYSTEM
AI_AGENT

Los logs pueden identificar el tipo de actor.

Ejemplo:

{
  "actor_type": "AI_AGENT",
  "actor_id": "..."
}

Esto no concede ningún privilegio.

12. Structured Logging

Los logs deberán ser estructurados, preferentemente JSON.

En lugar de:

User login failed

se recomienda:

{
  "level": "WARNING",
  "event": "authentication_failed",
  "actor_type": "USER",
  "request_id": "...",
  "correlation_id": "...",
  "trace_id": "...",
  "operation": "auth.login"
}

Esto permite posteriormente consultar y analizar los logs de forma automática.

13. Campos base

El esquema inicial podrá incluir:

timestamp
level
service
environment
message
event
request_id
correlation_id
trace_id
causation_id
actor_type
actor_id
tenant_id
organization_id
operation
resource_type
resource_id
duration_ms
status
error_code

No todos los campos son obligatorios en todos los eventos.

14. Campos que NO deben aparecer

Está estrictamente prohibido registrar:

password
password_hash
access_token
refresh_token
JWT
private_key
API_secret
MFA_secret
recovery_code
OTP
session_cookie
authorization_header
KMS_secret
database_password
Redis_password

También debe evitarse registrar información sensible innecesaria.

15. Sanitization Layer

Antes de emitir logs deberá existir una capa de sanitización.

Conceptualmente:

Application Data
      │
      ▼
Log Sanitizer
      │
      ▼
Structured Logger
      │
      ▼
Log Sink

La sanitización debe funcionar como defense in depth.

No debemos depender únicamente de que los desarrolladores recuerden no registrar secretos.

16. Exception Logging

Las excepciones deberán integrarse con el modelo creado en TASK-001-15.

Ejemplo:

{
  "level": "ERROR",
  "event": "request_failed",
  "error_code": "INTERNAL_ERROR",
  "exception_type": "UnexpectedError",
  "request_id": "...",
  "correlation_id": "...",
  "trace_id": "..."
}

En producción no se debe enviar al cliente:

stack trace
SQL
hostname interno
credenciales
paths sensibles

El detalle técnico puede permanecer en el sistema interno de observabilidad cuando corresponda.

17. Correlation ID

Debe reutilizarse la infraestructura establecida en TASK-001-01/14.

Si el cliente proporciona:

X-Correlation-ID

se podrá preservar si cumple las reglas de validación.

Si no existe o no es válido:

generate correlation_id

La respuesta podrá incluir:

X-Correlation-ID: ...
18. Request ID

Cada request HTTP debe tener un identificador único generado por el sistema:

request_id

No debe depender del cliente.

19. Trace ID

El trace_id identifica la operación distribuida.

Ejemplo:

trace-123
   │
   ├── HTTP span
   ├── DB span
   ├── Outbox span
   └── Event consumer span

No debe confundirse con request_id.

20. Causation ID

causation_id representa el evento o acción que causó otra operación.

Ejemplo:

User Action
     │
     ▼
Domain Event
     │
     ▼
Integration Event
     │
     ▼
Consumer

El consumer puede conservar la relación causal.

21. Metrics

Se establecerá una base para métricas.

Inicialmente:

HTTP
http_requests_total
http_request_duration
http_requests_failed
http_requests_by_status
Infrastructure
database_connection_errors
database_pool_usage
redis_connection_errors
redis_latency
Application
application_errors_total
dependency_resolution_errors
22. Identity Metrics futuras

La arquitectura permitirá posteriormente métricas como:

authentication_attempts
authentication_failures
authorization_requests
authorization_denials
authorization_latency
mfa_challenges
mfa_failures
refresh_token_reuse
session_revocations
policy_evaluations
policy_errors
cache_invalidations

Pero no todas se implementan en TASK-001-18.

Esta tarea establece la infraestructura y métricas base.

23. Métricas de latencia

Se deberán preparar histogramas/mediciones para:

request duration
database duration
redis duration
dependency resolution duration

Esto permitirá posteriormente validar:

API P95 < 300 ms
Authentication P95 < 500 ms
Authorization P95 < 50 ms

Los objetivos completos pertenecen a sus respectivos ESP/ADR y no deben considerarse demostrados únicamente por crear la métrica.

24. Health vs Observability

Debe mantenerse la separación:

Health Check
    ≠
Metric
    ≠
Log
    ≠
Trace

Por ejemplo:

/health/live

determina si el proceso está vivo.

Mientras:

database_connection_errors

es una métrica operacional.

25. Logging Levels

Se establecerán inicialmente:

DEBUG
INFO
WARNING
ERROR
CRITICAL
DEBUG

Información de desarrollo controlada.

INFO

Operaciones normales relevantes.

WARNING

Condiciones anormales que no necesariamente representan fallo.

ERROR

Operaciones fallidas.

CRITICAL

Condiciones que requieren atención inmediata.

Los Security Events utilizarán además su clasificación propia de severidad definida en el modelo de Security Events.

26. Audit ≠ Log

Ejemplo:

Log:
"Repository query completed"

Audit:
"ROLE_ASSIGNED"

Security Event:
"PRIVILEGE_ESCALATION_DETECTED"

No debemos convertir cada línea de log en un registro de auditoría.

27. Security Event ≠ Log

Un:

ERROR: database unavailable

es operacional.

Un:

REFRESH_TOKEN_REUSE_DETECTED

es una señal de seguridad.

Puede generar:

Log
+
Security Event
+
Audit evidence
+
Metric
+
Trace

pero cada elemento mantiene su propósito.

28. Audit y Security Events

Cuando se implementen los módulos correspondientes:

Application
    │
    ├── Audit Service
    │
    └── Security Event Service

Estos mecanismos deberán seguir las reglas de ESP-0001/09.

TASK-001-18 solamente proporciona la infraestructura observability transversal que permite correlacionarlos.

29. Observability Architecture

La arquitectura inicial será:

                    HTTP Request
                         │
                         ▼
                 Request Context
                         │
          ┌──────────────┼──────────────┐
          ▼              ▼              ▼
        Logs          Metrics         Traces
          │              │              │
          └──────────────┼──────────────┘
                         ▼
                  Operational Sink

Mientras:

Business Operation
       │
       ├── Audit Event
       │
       └── Security Event

se mantienen como dominios de evidencia separados.

30. OpenTelemetry

La arquitectura debe quedar preparada para una futura integración con OpenTelemetry.

Sin embargo:

No debemos acoplar Domain a OpenTelemetry.

La dependencia tecnológica debe quedar en Infrastructure/Core.

Conceptualmente:

Application
     │
     ▼
Observability Port
     │
     ▼
OpenTelemetry Adapter

Si posteriormente se cambia la tecnología de tracing, Domain/Application no deberían necesitar cambios estructurales.

31. Proposed Structure
apps/api/app/core/
│
├── observability/
│   ├── __init__.py
│   ├── config.py
│   │
│   ├── context/
│   │   ├── __init__.py
│   │   ├── trace.py
│   │   └── accessor.py
│   │
│   ├── logging/
│   │   ├── __init__.py
│   │   ├── logger.py
│   │   ├── formatter.py
│   │   └── sanitizer.py
│   │
│   ├── metrics/
│   │   ├── __init__.py
│   │   └── metrics.py
│   │
│   └── tracing/
│       ├── __init__.py
│       └── tracer.py
│
└── middleware/
    └── ...

La estructura exacta puede ajustarse al árbol que resulte de las tareas anteriores.

32. Observability Ports

Cuando exista necesidad de desacoplar completamente la tecnología:

class Logger(Protocol):
    def info(self, event: str, **fields) -> None:
        ...

y:

class Metrics(Protocol):
    def increment(self, name: str, **labels) -> None:
        ...

La implementación concreta queda en Infrastructure.

Para logging básico puede utilizarse directamente el mecanismo estándar de Python desde Core, siempre que no contamine Domain.

33. Correlation Middleware

El middleware establecido anteriormente debe ampliarse para poblar:

request_id
correlation_id
trace_id

y dejar el contexto disponible durante todo el request.

Conceptualmente:

Incoming Request
      │
      ▼
Create/Validate Context
      │
      ▼
ContextVar
      │
      ├── API
      ├── Application
      ├── Infrastructure
      └── Logging
      │
      ▼
Cleanup
34. Context Cleanup

Al terminar un request, el contexto debe eliminarse/restaurarse correctamente.

Esto es obligatorio para evitar contaminación entre requests concurrentes.

Test:

Request A
trace-A

Request B
trace-B

Request A != Request B
35. Async Safety

Dado que EVOXA utilizará FastAPI/async:

Request A
Request B
Request C

podrán ejecutarse concurrentemente.

Los tests deberán comprobar que:

trace-A ≠ trace-B
tenant-A ≠ tenant-B
actor-A ≠ actor-B

y que ningún contexto se cruza.

36. Error Observability

TASK-001-15 definió:

EvoxaError
    │
    ├── DomainError
    ├── ApplicationError
    ├── SecurityError
    ├── InfrastructureError
    └── SystemError

TASK-001-18 debe permitir registrar:

error_type
error_code
operation
request_id
correlation_id
trace_id

sin exponer información sensible.

37. Security Observability

Las decisiones de seguridad futuras podrán registrar metadatos como:

policy_id
policy_version
decision
reason_code
criticality
freshness_class
cacheability
runtime_version

Esto está alineado con el modelo de observabilidad de autorización.

Pero nunca:

password
token
MFA secret
private key
38. Authorization Observability

Cuando se implemente Authorization Runtime, podremos observar:

Authorization Request
       │
       ├── tenant
       ├── actor
       ├── resource
       ├── action
       ├── policy version
       ├── criticality
       ├── freshness
       └── cache state
               │
               ▼
            Decision

Esto será fundamental para debugging y auditoría.

39. AI Observability

Los AI Agents tendrán:

actor_type = AI_AGENT

y podrán participar en tracing.

Pero:

Trace ID
   ≠
Authorization

El tracing de un AI Agent no le concede ningún privilegio adicional.

Además, nunca se registrarán:

AI secrets
API credentials
system credentials
user secrets
40. Privacy / Data Minimization

No todo dato disponible debe registrarse.

La regla será:

Log only what is necessary for operational, security or audit purposes.

Especialmente debemos evitar registrar innecesariamente:

email completo
phone
address
health information
nutrition information
fitness information

cuando no sea necesario.

Esto será especialmente importante cuando EVOXA incorpore los dominios de entrenamiento, nutrición y salud.

41. Observability Configuration

Configuración conceptual:

OBSERVABILITY_ENABLED
LOG_LEVEL
LOG_FORMAT
METRICS_ENABLED
TRACING_ENABLED
TRACE_SAMPLE_RATE
ENVIRONMENT
SERVICE_NAME

Los valores deben provenir del sistema de configuración establecido en TASK-001-10.

Nunca:

SECRET
PASSWORD
TOKEN

como configuración de logging.

42. Environment Awareness

Debe distinguirse:

development
qa
staging
production

En desarrollo podremos utilizar logs más detallados.

En producción:

minimize sensitive detail

y mantener políticas de seguridad más estrictas.

43. Sampling

El tracing puede utilizar sampling.

Inicialmente podemos dejar la configuración preparada:

trace_sample_rate

pero no fijar todavía una política definitiva para producción.

La tasa final dependerá de:

volumen;
coste;
criticidad;
infraestructura;
necesidades de diagnóstico.
44. Performance

La observabilidad no debe degradar significativamente la API.

Objetivo:

Observability overhead
≈ minimal

Los mecanismos deben evitar:

blocking I/O

innecesario durante requests.

Especialmente:

logging
metrics
tracing

no deberían convertirse en cuellos de botella.

45. Failure Behavior

Una regla crítica:

Observability failure must not become a security bypass.

Por ejemplo:

Metrics backend DOWN
        ↓
Application continues

siempre que sea seguro.

Pero:

Authorization Runtime failure
        ↓
DENY / FAIL CLOSED

según sus reglas.

La observabilidad simplemente registra el estado.

46. Tests
Unit Tests
OBS-UNIT-001

Logger produce estructura válida.

OBS-UNIT-002

Sanitizer elimina secretos.

OBS-UNIT-003

Correlation ID se conserva.

OBS-UNIT-004

Request ID es único.

OBS-UNIT-005

TraceContext es inmutable.

OBS-UNIT-006

Context cleanup funciona.

OBS-UNIT-007

Metrics no modifican lógica de negocio.

47. Security Tests
OBS-SEC-001

Password nunca aparece en logs.

OBS-SEC-002

Access token nunca aparece.

OBS-SEC-003

Refresh token nunca aparece.

OBS-SEC-004

MFA secret nunca aparece.

OBS-SEC-005

Recovery code nunca aparece.

OBS-SEC-006

Private keys nunca aparecen.

OBS-SEC-007

Authorization headers nunca aparecen.

48. Concurrency Tests
OBS-CONC-001

Dos requests concurrentes tienen diferentes request IDs.

OBS-CONC-002

Dos requests concurrentes no comparten TraceContext.

OBS-CONC-003

Tenant Context no cruza requests.

OBS-CONC-004

Actor Context no cruza requests.

OBS-CONC-005

Context cleanup ocurre correctamente.

49. Integration Tests
OBS-INT-001

HTTP request genera contexto.

OBS-INT-002

Logs contienen correlation ID.

OBS-INT-003

Trace ID se propaga.

OBS-INT-004

Error HTTP queda correlacionado.

OBS-INT-005

Database operation conserva trace context.

OBS-INT-006

Outbox conserva correlation/causation/trace metadata cuando corresponda.

50. Failure Tests

Debemos probar:

logging failure
metrics failure
tracing failure
Redis failure
database failure

y confirmar que observabilidad:

no crashes application unexpectedly
no changes authorization decision
no leaks secrets
51. Acceptance Criteria
AC-01

Given un HTTP request,
When entra a la aplicación,
Then obtiene request_id, correlation_id y trace_id.

AC-02

Given una operación interna,
When genera logs,
Then conserva el contexto de tracing correspondiente.

AC-03

Given requests concurrentes,
When se procesan,
Then sus contextos permanecen aislados.

AC-04

Given un error,
When se registra,
Then utiliza el error_code seguro definido en TASK-001-15.

AC-05

Given un password,
When accidentalmente se incluye en un objeto de log,
Then el sanitizer evita su exposición.

AC-06

Given un refresh token,
When accidentalmente se intenta registrar,
Then no aparece en el output.

AC-07

Given una operación de negocio,
When falla el sistema de métricas,
Then la operación no obtiene un ALLOW incorrecto.

AC-08

Given una operación de negocio,
When falla tracing,
Then la operación no obtiene privilegios adicionales.

AC-09

Given un AI Agent,
When ejecuta una operación,
Then puede conservar trazabilidad sin obtener privilegios adicionales.

AC-10

Given un tenant context,
When se registra operacionalmente,
Then no puede cruzarse con otro request.

AC-11

Given una excepción interna,
When se genera el log,
Then no se exponen stack traces al cliente.

AC-12

Given un request completado,
When termina,
Then su contexto es limpiado.

AC-13

Given Application code,
When utiliza observabilidad,
Then no importa directamente una implementación tecnológica específica cuando exista un port apropiado.

AC-14

Given Domain code,
When se ejecutan architecture tests,
Then no puede importar OpenTelemetry, logging infrastructure u otros adapters.

AC-15

Given las métricas base,
When se ejecuta la aplicación,
Then se pueden medir latencia y errores HTTP.

AC-16

Given observability disabled donde esté permitido,
When inicia la aplicación,
Then no se rompe el funcionamiento esencial.

52. Definition of Done
 Structured logging establecido.
 Sanitization implementada.
 Request ID integrado.
 Correlation ID integrado.
 Trace ID integrado.
 Causation context preparado.
 ContextVar/request context implementado.
 Context cleanup implementado.
 Actor/Tenant context integrable.
 Metrics base implementadas.
 Tracing abstraction preparada.
 OpenTelemetry boundary definida.
 Error observability integrada.
 Security logging rules implementadas.
 Secret leakage tests implementados.
 Concurrency tests implementados.
 Failure tests implementados.
 Architecture boundary tests actualizados.
 CI/CD ejecuta observability tests.
 Documentación actualizada.
 Traceability registrada.
53. Resultado arquitectónico

Con TASK-001-18 tendremos:

                         EVOXA Identity
                              │
                ┌─────────────┼─────────────┐
                ▼             ▼             ▼
              Logs         Metrics        Traces
                │             │             │
                └─────────────┼─────────────┘
                              │
                     Operational Data
                              │
             ┌────────────────┴────────────────┐
             ▼                                 ▼
        Audit Events                    Security Events

Y todos podrán relacionarse mediante:

request_id
correlation_id
trace_id
causation_id
actor
tenant

sin mezclar sus responsabilidades.

54. Traceability
Blueprint
    │
    ▼
Architecture Map
    │
    ▼
ESP-0001
    │
    ├── ESP-0001/02 Architecture
    ├── ESP-0001/03 Auth & Authorization
    ├── ESP-0001/09 Audit
    ├── ESP-0001/10 API
    ├── ESP-0001/12 Events
    ├── ESP-0001/13 Testing
    ├── ESP-0001/14 Implementation
    └── ESP-0001/15 Deployment
    │
    ▼
ADR-028 Observability & Operational Security
ADR-027 Configuration & Secrets
ADR-007 Transactional Outbox
    │
    ▼
TASK-001-14 Trace Context
TASK-001-15 Exception Model
    │
    ▼
TASK-001-18
    │
    ▼
Initial Observability
55. Estado de IS-001
TASK-001-01  API Runtime Foundation             ✅
TASK-001-02  Identity Domain Package             ✅
TASK-001-03  Layer Architecture                  ✅
TASK-001-04  Identity Shared Kernel              ✅
TASK-001-05  Domain Events Foundation             ✅
TASK-001-06  Repository Contracts                ✅
TASK-001-07  Unit of Work                        ✅
TASK-001-08  PostgreSQL Infrastructure            ✅
TASK-001-09  Redis Infrastructure                 ✅
TASK-001-10  Configuration & Secrets              ✅
TASK-001-11  Security Foundation                  ✅
TASK-001-12  Tenant Context Foundation            ✅
TASK-001-13  Actor Context                        ✅
TASK-001-14  Correlation & Trace Context          ✅
TASK-001-15  Exception Model                      ✅
TASK-001-16  Dependency Injection                 ✅
TASK-001-17  Architecture Boundary Tests          ✅
TASK-001-18  Initial Observability                🟢
Siguiente tarea

TASK-001-19 — Initial Migration Framework

Aquí estableceremos la base para que las futuras tablas de Identity —User, Organization, Membership, Role, Permission, Session, RefreshToken, MFA, Policy, Audit, Security Events, Outbox, etc.— se incorporen mediante migraciones versionadas, reproducibles y controladas, en lugar de depender de sequelize.sync, create_all o modificaciones automáticas del esquema.
