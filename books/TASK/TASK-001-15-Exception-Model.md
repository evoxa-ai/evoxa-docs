TASK-001-15 — Exception Model

IS: IS-001 — Identity Domain Foundation
ESP: ESP-0001/14 — Identity Implementation Requirements
Dependencias: TASK-001-03, TASK-001-04, TASK-001-07, TASK-001-10, TASK-001-11, TASK-001-12, TASK-001-13, TASK-001-14
Prioridad: 🔴 Critical
Estado: Ready for Implementation

1. Objetivo

Establecer un modelo único, estructurado y seguro de excepciones para EVOXA Identity.

La finalidad es que todos los errores tengan una semántica clara desde:

Domain
   ↓
Application
   ↓
Infrastructure
   ↓
API
   ↓
HTTP Response

y que ningún componente exponga accidentalmente:

passwords
tokens
SQL
stack traces
secrets
MFA secrets
internal infrastructure
2. Problema que resolvemos

Sin un modelo centralizado podemos terminar con errores diferentes:

ValueError
Exception
SQLAlchemyError
HTTPException
KeyError
JWTError

mezclados dentro de Domain, Application e Infrastructure.

Esto provoca:

respuestas HTTP inconsistentes;
errores difíciles de observar;
filtración de información;
dificultad para hacer testing;
dependencia del Domain respecto de FastAPI;
dificultad para correlacionar errores;
comportamiento inconsistente de seguridad.

La arquitectura objetivo será:

Domain Error
      ↓
Application Error
      ↓
Infrastructure Error
      ↓
Exception Mapper
      ↓
API Error Envelope
3. Principio fundamental

Una excepción interna no es automáticamente una respuesta HTTP.

Por ejemplo:

DomainRuleViolation

no debe conocer:

HTTP 400

El Domain sólo conoce su propia semántica.

La API decide posteriormente cómo representar ese error externamente.

4. Jerarquía conceptual

Proponemos:

EvoxaError
│
├── DomainError
│   ├── DomainRuleViolation
│   ├── InvalidEntityState
│   └── DomainInvariantViolation
│
├── ApplicationError
│   ├── ValidationError
│   ├── ResourceNotFound
│   ├── ConflictError
│   └── OperationNotAllowed
│
├── SecurityError
│   ├── AuthenticationError
│   ├── AuthorizationError
│   ├── TenantAccessError
│   └── SecurityPolicyError
│
├── InfrastructureError
│   ├── DatabaseError
│   ├── CacheError
│   ├── EventPublicationError
│   └── ExternalServiceError
│
└── SystemError
    ├── ConfigurationError
    └── UnexpectedError

Esta jerarquía es conceptual; las clases exactas deben mantenerse simples.

5. EvoxaError

La raíz común:

class EvoxaError(Exception):
    ...

No debería contener automáticamente:

request object
database session
HTTP response
password
token
secret

Debe ser un error de aplicación portable.

6. Error Code

Cada error expuesto a través de API debe tener un código estable.

Ejemplos:

AUTHENTICATION_FAILED
AUTHORIZATION_DENIED
TENANT_ACCESS_DENIED
RESOURCE_NOT_FOUND
VALIDATION_ERROR
CONCURRENCY_CONFLICT
INTERNAL_ERROR

El código es más importante para clientes que el texto humano.

7. Error Message

El mensaje debe ser:

safe
stable
human-readable
non-sensitive

Ejemplo:

{
  "error": {
    "code": "AUTHENTICATION_FAILED",
    "message": "Authentication failed.",
    "correlation_id": "..."
  }
}

No:

{
  "error": {
    "message": "User sebastian@example.com exists but password hash verification failed."
  }
}
8. Error Envelope

EVOXA ya establece un formato estándar:

{
  "error": {
    "code": "...",
    "message": "...",
    "correlation_id": "..."
  }
}

TASK-001-15 convierte ese contrato en una regla técnica central.

9. Correlation ID

El error debe utilizar el contexto creado en:

TASK-001-14 — Correlation & Trace Context

Por ejemplo:

{
  "error": {
    "code": "INTERNAL_ERROR",
    "message": "An unexpected error occurred.",
    "correlation_id": "c8b..."
  }
}

Esto permite localizar el incidente internamente sin revelar detalles técnicos.

10. Request ID vs Correlation ID

La respuesta pública utilizará principalmente:

correlation_id

según el contrato API.

Internamente podremos disponer de:

request_id
trace_id

para observabilidad.

No debemos añadir todos los identificadores a la respuesta API por defecto.

11. Domain Errors

El Domain puede generar:

DomainError

cuando se viola una regla propia del dominio.

Ejemplo conceptual:

raise DomainRuleViolation(
    code="INVALID_ACCOUNT_STATE",
    message="The account state transition is invalid."
)

El Domain no sabe que esto posteriormente será:

HTTP 409
12. Domain Invariants

Una violación de una invariante debe diferenciarse conceptualmente:

DomainRuleViolation

vs.

DomainInvariantViolation

La segunda puede indicar un problema más grave:

Aggregate state inconsistent

No debería convertirse automáticamente en un error detallado para el cliente.

13. Application Errors

Application representa errores relacionados con la ejecución del caso de uso.

Ejemplos:

ValidationError
ResourceNotFound
ConflictError
OperationNotAllowed

Ejemplo:

User update
   ↓
version mismatch
   ↓
CONCURRENCY_CONFLICT
14. Security Errors

Security errors necesitan especial cuidado.

Ejemplos:

AuthenticationError
AuthorizationError
TenantAccessError
SecurityPolicyError

No todos deben revelar exactamente por qué falló una decisión.

15. Authentication Error

Ejemplo:

AUTHENTICATION_FAILED

Debe evitar enumeración.

En general no debemos distinguir públicamente:

user doesn't exist

de:

password incorrect

cuando hacerlo facilite account enumeration.

16. Authorization Error

Ejemplo:

AUTHORIZATION_DENIED

Puede representar:

RBAC denied
ABAC denied
ownership denied
policy denied
delegation denied

pero no necesariamente debemos exponer el razonamiento interno completo.

17. Tenant Error

Un acceso a tenant no autorizado puede producir:

TENANT_ACCESS_DENIED

Pero no debemos confirmar innecesariamente:

Tenant X exists
Tenant X belongs to User Y

La información detallada queda en logs/audit autorizados.

18. Security Policy Error

Un fallo interno del Policy Engine no debe transformarse en:

ALLOW

Por ejemplo:

Policy evaluation
       ↓
ERROR
       ↓
FAIL CLOSED

La respuesta externa puede ser:

AUTHORIZATION_UNAVAILABLE

o un código equivalente que definamos en el contrato final.

19. Infrastructure Errors

Los errores de infraestructura deben ser traducidos.

Por ejemplo:

SQLAlchemyError

no debe escapar hasta el cliente.

Debe convertirse internamente en algo como:

DatabaseError

y después:

INTERNAL_ERROR

o un error operacional específico cuando sea seguro exponerlo.

20. Database Error

Incorrecto:

{
  "error": {
    "message": "duplicate key value violates unique constraint users_email_key"
  }
}

Correcto:

{
  "error": {
    "code": "CONFLICT",
    "message": "The requested resource conflicts with existing data.",
    "correlation_id": "..."
  }
}

cuando corresponda.

21. SQL Leakage

Nunca deben aparecer en respuestas:

SQL query
table name
column name
database hostname
connection string
stack trace
driver error

salvo que exista una necesidad operacional interna, nunca pública.

22. Redis Errors

Un fallo Redis:

Redis unavailable

no debe convertirse en:

ALLOW

Especialmente en operaciones relacionadas con seguridad.

Debe seguir las reglas de degradación establecidas por las ADR de cache.

23. Event Errors

Si publicar un evento falla:

EventPublisherError

no debemos ocultar el problema ejecutando una publicación parcial o inventando éxito.

La interacción con Outbox garantiza:

Business state
+
Outbox state

sean transaccionales.

24. Configuration Errors

Errores de configuración:

ConfigurationError

deben producir comportamiento fail-fast en startup cuando afectan componentes obligatorios.

Ejemplo:

JWT signing configuration missing

no debe permitir levantar Identity en un estado inseguro.

25. Unexpected Error

Cualquier error no controlado:

Exception

debe terminar en:

INTERNAL_ERROR

externamente.

Pero internamente debe:

log
trace
correlate
alert

según severidad.

26. Nunca except Exception: pass

Está prohibido:

try:
    ...
except Exception:
    pass

porque puede convertir:

security failure
database failure
authorization failure

en comportamiento aparentemente exitoso.

27. Fail Closed

Para operaciones de seguridad:

Unknown
Error
Timeout
Unavailable
Corrupt context

no deben generar:

ALLOW

Regla:

Security uncertainty
       ↓
DENY / FAIL CLOSED

según el contexto.

28. Exception Mapping

Debemos centralizar la transformación:

Exception
    ↓
ExceptionMapper
    ↓
API Error

Propuesta:

app/core/errors/
├── exceptions.py
├── codes.py
├── mapper.py
└── handlers.py
29. Exception Handler

FastAPI debe tener handlers globales para:

EvoxaError
Request validation
HTTP errors
Unexpected exceptions

Pero:

Los endpoints no deberían implementar su propio patrón de try/except para construir respuestas estándar.

30. API Handler

Conceptualmente:

@app.exception_handler(EvoxaError)
async def evoxa_error_handler(request, exc):
    ...

El handler:

obtiene correlation context;
determina código seguro;
determina status HTTP;
construye envelope;
registra información operacional apropiada;
devuelve respuesta.
31. Error Classification

Cada error debería poder clasificarse como:

DOMAIN
APPLICATION
SECURITY
INFRASTRUCTURE
SYSTEM

Esto ayuda a:

logging
metrics
alerting
testing
observability
32. Error Severity

Podemos establecer:

INFO
LOW
MEDIUM
HIGH
CRITICAL

pero no todos los errores deben ser automáticamente ERROR o CRITICAL.

Por ejemplo:

ResourceNotFound

puede ser una operación normal.

Mientras:

RefreshTokenReuseDetected

es un Security Event de alta importancia.

33. Exception ≠ Security Event

Una excepción:

AuthorizationDenied

no necesariamente es un Security Event.

Pero determinados eventos de seguridad sí pueden generar:

SecurityEvent
AuditEvent
Alert

Ejemplo:

Refresh token reuse
       ↓
SecurityEvent
       +
AuditEvent
       +
Exception/response
34. Audit Integration

Cuando sea necesario, el error puede estar asociado a:

actor
tenant
request_id
correlation_id
trace_id
operation
resource
result

pero la excepción no debe almacenar toda esta información como estado permanente.

El Audit layer consume el contexto apropiado.

35. Authorization Error Detail

El Authorization Runtime puede producir internamente:

decision
reason_code
policy_id
policy_version
criticality
freshness
cacheability

Pero la API pública no debe exponer automáticamente:

policy internals
ABAC attributes
security rules
risk score
hidden resources
36. Error Details

Podemos permitir un campo opcional:

{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "The request is invalid.",
    "correlation_id": "...",
    "details": {
      "field": "email",
      "reason": "invalid_format"
    }
  }
}

Pero:

details debe pasar por una política de sanitización.

37. No Internal Details

Nunca:

{
  "details": {
    "exception": "sqlalchemy.exc.IntegrityError",
    "stack": "...",
    "database": "identity-prod"
  }
}
38. HTTP Mapping

El mapping conceptual:

Error	HTTP
ValidationError	400
AuthenticationError	401
AuthorizationError	403
TenantAccessError	403
ResourceNotFound	404
ConflictError	409
ConcurrencyConflict	409
RateLimitError	429
Infrastructure unavailable	503
UnexpectedError	500

Estos mappings son una base técnica, pero los códigos HTTP definitivos deben quedar congelados en el contrato API de ESP-0001/10.

39. 401 vs 403

La distinción debe mantenerse:

401
→ no existe autenticación válida

403
→ existe identidad pero no autorización

Ejemplo:

No token
   ↓
401

vs:

Authenticated
   ↓
Policy DENY
   ↓
403
40. Tenant y 403

Cuando:

Actor authenticated
+
tenant access invalid

puede representarse como:

403

sin revelar si:

tenant inexistente

o:

membership inexistente

cuando esa distinción pueda permitir enumeración.

41. Validation Errors

FastAPI/Pydantic puede producir errores detallados de validación.

Debemos normalizarlos al envelope EVOXA:

{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "The request is invalid.",
    "correlation_id": "...",
    "details": [...]
  }
}

Los detalles deben contener solamente información segura sobre los campos enviados.

42. Error Response Headers

La respuesta puede incluir:

X-Correlation-ID: ...

manteniendo consistencia con TASK-001-14.

El body también incluye:

correlation_id

según el contrato API.

43. Logging

Cuando se produce una excepción:

Exception
 ↓
Classification
 ↓
Structured Log

debe incluir, cuando corresponda:

timestamp
error_code
error_type
correlation_id
request_id
trace_id
actor_type
actor_id
tenant_id
endpoint
operation

sin secretos.

44. Stack Trace

Los stack traces:

pueden existir internamente para debugging.

Pero:

❌ API response
❌ audit payload
❌ client-visible error

No deben exponerse.

45. Exception Chaining

Cuando una infraestructura produce:

DatabaseDriverError

podemos preservar internamente:

raise DatabaseError(...) from exc

para debugging.

Pero el mapper debe evitar que la excepción original termine expuesta al cliente.

46. Retryable Errors

Algunos errores pueden ser:

retryable

y otros no.

Por ejemplo:

Transient DB connection failure

puede ser retryable internamente.

Mientras:

AUTHENTICATION_FAILED

no debe reintentarse automáticamente de forma ilimitada.

Esto deberá integrarse con las políticas de resiliencia correspondientes.

47. Idempotency Errors

TASK-001-15 también debe soportar errores como:

IDEMPOTENCY_CONFLICT

cuando:

same Idempotency-Key
+
different request fingerprint

según ADR-026.

48. Concurrency Conflict

El repository/UoW puede detectar:

version mismatch

y producir:

CONCURRENCY_CONFLICT

Esto es preferible a:

500 INTERNAL_ERROR

porque el cliente puede reaccionar correctamente.

49. Error Codes

Propuesta inicial:

VALIDATION_ERROR
AUTHENTICATION_FAILED
AUTHENTICATION_REQUIRED
AUTHORIZATION_DENIED
TENANT_ACCESS_DENIED
RESOURCE_NOT_FOUND
CONFLICT
CONCURRENCY_CONFLICT
IDEMPOTENCY_CONFLICT
RATE_LIMITED
SERVICE_UNAVAILABLE
INTERNAL_ERROR

No debemos crear cientos de códigos prematuramente.

Los códigos deben representar contratos estables.

50. Error Code Governance

Los códigos deben:

ser únicos
ser estables
no depender de librerías
no depender de SQL
no cambiar por refactoring interno

Incorrecto:

SQLALCHEMY_INTEGRITY_ERROR

Correcto:

CONFLICT
51. Domain Error Codes

Los códigos internos de Domain pueden ser más específicos:

INVALID_USER_STATE_TRANSITION
INVALID_MEMBERSHIP_STATE
INVALID_ROLE_ASSIGNMENT

pero no todos necesitan convertirse en códigos públicos diferentes.

Puede existir:

internal error code

y:

public API code

separados.

52. Security Information Disclosure

Los errores no deben revelar:

existence of sensitive resources
policy configuration
roles
permissions
MFA configuration
session identifiers
token state
risk score
tenant membership

cuando esa información pueda ayudar a un atacante.

53. Timing

El modelo de errores debe evitar diferencias innecesarias de comportamiento que faciliten:

account enumeration
tenant enumeration
resource enumeration

La protección de timing exacta corresponde a Authentication/Security tasks, pero Exception Model debe respetar ese principio.

54. Exception Boundary

La regla arquitectónica:

Domain
    ↓
DomainError

Application
    ↓
ApplicationError

Infrastructure
    ↓
InfrastructureError

API
    ↓
API Error Envelope

No:

Domain
 ↓
HTTPException
55. FastAPI Isolation

FastAPI pertenece a:

Presentation/API

por lo tanto:

from fastapi import HTTPException

no debe aparecer en:

domains/identity/*/domain/
domains/identity/*/application/
56. Proposed Structure
apps/api/app/core/errors/
│
├── base.py
├── codes.py
├── categories.py
├── exceptions.py
├── mapper.py
├── handlers.py
└── sanitization.py

Y los errores específicos de Domain pueden vivir en:

domains/identity/shared/domain/exceptions.py

o dentro del módulo que sea dueño de la regla.

57. Error Context

No recomiendo que EvoxaError contenga directamente:

ActorContext
TenantContext
TraceContext
Request

porque eso acoplaría errores con context management.

En cambio:

Exception
+
Request Context

se combinan en el handler.

58. Error Flow

El flujo final:

                         REQUEST
                            │
                            ▼
                    Application Operation
                            │
             ┌──────────────┼──────────────┐
             │              │              │
             ▼              ▼              ▼
          DOMAIN      INFRASTRUCTURE    SECURITY
             │              │              │
             └──────────────┼──────────────┘
                            ▼
                       EvoxaError
                            │
                            ▼
                    Exception Mapper
                            │
                  ┌─────────┴─────────┐
                  ▼                   ▼
             Safe Response        Structured Log
                  │                   │
                  ▼                   ▼
             Error Envelope      Trace/Observability
59. Testing
Domain
Given:
  invalid domain operation

When:
  domain executes

Then:
  DomainRuleViolation is raised
Application
Given:
  resource doesn't exist

When:
  use case executes

Then:
  ResourceNotFound is raised
Authorization
Given:
  authenticated actor

When:
  policy denies operation

Then:
  AuthorizationError is produced
60. Fail Closed Test
Given:
  authorization engine returns ERROR

When:
  protected operation executes

Then:
  operation does not produce ALLOW
61. Exception Sanitization Test
Given:
  database throws internal exception

When:
  API responds

Then:
  response does not contain:
    SQL
    stack trace
    hostname
    credentials
62. Correlation Test
Given:
  request has correlation_id C-123

When:
  exception occurs

Then:
  response contains C-123
63. Context Test
Given:
  actor A
  tenant A
  trace A

When:
  exception occurs

Then:
  internal structured log associates:
    actor A
    tenant A
    trace A
64. Cross-Request Test
Given:
  Request A → correlation A
  Request B → correlation B

When:
  both execute concurrently

Then:
  errors never cross-correlate
65. Unexpected Error Test
Given:
  unknown Python exception

When:
  API receives it

Then:
  HTTP 500
  code = INTERNAL_ERROR
  safe message
  correlation_id present

Y además:

internal stack trace
→ observability

pero no:

client
66. Acceptance Criteria
ID	Criterio
AC-01	Existe EvoxaError
AC-02	Domain/Application/Infrastructure/Security errors están diferenciados
AC-03	Existe catálogo de error codes
AC-04	Existe mapper centralizado
AC-05	Existe handler global
AC-06	Se utiliza el error envelope EVOXA
AC-07	correlation_id aparece cuando corresponde
AC-08	Domain no depende de FastAPI
AC-09	Infrastructure errors son sanitizados
AC-10	SQL nunca se expone
AC-11	Stack traces nunca se exponen
AC-12	Secrets nunca se exponen
AC-13	Authentication errors protegen enumeration
AC-14	Authorization errors no exponen policy internals
AC-15	Tenant errors no permiten enumeration innecesaria
AC-16	Security failures fail closed
AC-17	Concurrency conflicts tienen tratamiento específico
AC-18	Idempotency conflicts tienen tratamiento específico
AC-19	Unexpected exceptions generan INTERNAL_ERROR
AC-20	Existen tests de aislamiento y sanitización
67. Definition of Done
[ ] EvoxaError
[ ] DomainError
[ ] ApplicationError
[ ] SecurityError
[ ] InfrastructureError
[ ] SystemError
[ ] ErrorCode catalog
[ ] Error categories
[ ] Exception mapper
[ ] Global exception handlers
[ ] Error sanitization
[ ] API envelope integration
[ ] Correlation integration
[ ] Structured logging
[ ] 401/403 mapping
[ ] 404 mapping
[ ] 409 mapping
[ ] 429 mapping
[ ] 500 mapping
[ ] 503 mapping
[ ] Security fail-closed tests
[ ] Secret leakage tests
[ ] SQL leakage tests
[ ] Stack trace leakage tests
[ ] Enumeration tests
[ ] Concurrent context tests
[ ] Architecture tests
[ ] Documentation
68. Trazabilidad
IS-001
 │
 └── TASK-001-15
       │
       ├── ESP-0001/03
       │     Authentication & Authorization
       │
       ├── ESP-0001/09
       │     Audit & Compliance
       │
       ├── ESP-0001/10
       │     Identity API Contracts
       │
       ├── ESP-0001/12
       │     Events & Integration
       │
       ├── ESP-0001/13
       │     Testing & Quality
       │
       ├── ESP-0001/14
       │     Implementation Requirements
       │
       ├── ADR-005
       │     Authorization Model
       │
       ├── ADR-028
       │     Observability & Operational Security
       │
       ├── ADR-032
       │     Rate Limiting & Abuse
       │
       ├── ADR-033
       │     Account Lifecycle
       │
       ├── ADR-043
       │     Multi-Organization Identity
       │
       ├── ADR-045–049
       │     Authorization Runtime
       │
       ├── ADR-054–059
       │     Authorization Failure/Freshness
       │
       └── ADR-067/069
             Identity Boundaries & Contracts
69. Regla de oro de TASK-001-15

Toda excepción debe seguir esta cadena:

                        INTERNAL ERROR
                              │
                              ▼
                    CLASSIFY + SANITIZE
                              │
                  ┌───────────┴───────────┐
                  │                       │
                  ▼                       ▼
           CLIENT RESPONSE          OBSERVABILITY
                  │                       │
                  ▼                       ▼
           SAFE ENVELOPE             FULL CONTEXT
                  │                       │
                  ▼                       ▼
        code/message/correlation     logs/traces/audit

El cliente recibe lo mínimo necesario.

El sistema interno conserva lo necesario para diagnosticar y auditar.

Estado de IS-001
TASK-001-01  API Runtime Foundation       ✅
TASK-001-02  Identity Domain Package      ✅
TASK-001-03  Layer Architecture           ✅
TASK-001-04  Identity Shared Kernel       ✅
TASK-001-05  Domain Events Foundation     ✅
TASK-001-06  Repository Contracts         ✅
TASK-001-07  Unit of Work                 ✅
TASK-001-08  PostgreSQL Infrastructure    ✅
TASK-001-09  Redis Infrastructure         ✅
TASK-001-10  Configuration & Secrets      ✅
TASK-001-11  Security Foundation          ✅
TASK-001-12  Tenant Context Foundation    ✅
TASK-001-13  Actor Context                ✅
TASK-001-14  Correlation & Trace Context  ✅
TASK-001-15  Exception Model              ✅
Siguiente

TASK-001-16 — Dependency Injection

Aquí vamos a establecer cómo se conectan de forma controlada API → Application → Domain → Repository → Unit of Work → PostgreSQL/Redis/Event infrastructure, evitando instancias globales, dependencias circulares y acceso directo de los módulos a infraestructura.
