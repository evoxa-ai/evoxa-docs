TASK-001-14 — Correlation & Trace Context

IS: IS-001 — Identity Domain Foundation
ESP: ESP-0001/14 — Identity Implementation Requirements
Dependencias: TASK-001-05, TASK-001-07, TASK-001-11, TASK-001-12, TASK-001-13
Prioridad: 🔴 Critical
Estado: Ready for Implementation

1. Objetivo

Establecer el contexto transversal de trazabilidad de una operación dentro de EVOXA.

Esta tarea define cómo una operación puede seguirse desde:

HTTP Request
   ↓
API
   ↓
Application
   ↓
Domain
   ↓
Database
   ↓
Outbox
   ↓
Event Platform
   ↓
Consumer

sin confundir trazabilidad con:

autenticación,
autorización,
tenant context,
identidad del actor.

La finalidad es poder responder:

¿De dónde vino esta operación, qué componentes participaron y qué eventos/procesos derivaron de ella?

2. Problema que resolvemos

Sin un contexto común podemos terminar con:

API log:
request = 123

Application log:
request = abc

Database log:
transaction = xyz

Event:
event = 456

y no saber si todos pertenecen a la misma operación.

Queremos:

Request
   │
   ├── correlation_id
   ├── trace_id
   └── request_id
          │
          ▼
Application
          │
          ▼
Domain Event
          │
          ├── correlation_id
          ├── causation_id
          └── trace_id
          │
          ▼
Outbox
          │
          ▼
Event Platform
3. Conceptos que debemos separar

EVOXA tendrá al menos:

Request ID
Correlation ID
Trace ID
Causation ID

No son sinónimos.

4. Request ID

Identifica una request concreta.

Ejemplo:

Request A
request_id = R-001

Otra request:

Request B
request_id = R-002

Aunque ambas formen parte de la misma operación lógica:

correlation_id = C-001
5. Correlation ID

Representa una operación lógica o flujo relacionado.

Ejemplo:

POST /auth/login
       │
       correlation_id = C-001
       │
       ├── authentication
       ├── session creation
       ├── security event
       └── audit event

Todos pueden compartir:

correlation_id = C-001
6. Trace ID

El trace_id identifica una traza distribuida.

Conceptualmente:

Trace
 │
 ├── API span
 ├── Application span
 ├── DB span
 ├── Event publish span
 └── Consumer span

Todos forman parte de una misma ejecución/traza distribuida.

La implementación concreta de tracing queda fuera de esta tarea; aquí establecemos el contexto que EVOXA debe transportar.

7. Causation ID

causation_id responde:

¿Qué evento u operación causó este evento?

Ejemplo:

UserCreated
     │
     ▼
MembershipCreated

El segundo evento puede tener:

causation_id = UserCreated.event_id

Mientras ambos pueden compartir:

correlation_id = C-001
8. Diferencia fundamental
Request ID
    = una request específica

Correlation ID
    = una operación/flujo lógico

Trace ID
    = una traza distribuida

Causation ID
    = relación causal inmediata
9. Ejemplo completo

Supongamos:

POST /api/v1/users

Request:

request_id     = R-100
correlation_id = C-100
trace_id       = T-100

Se crea:

UserCreated

Evento:

event_id       = E-100
correlation_id = C-100
causation_id   = R-100
trace_id       = T-100

Posteriormente:

MembershipCreated

puede tener:

event_id       = E-101
correlation_id = C-100
causation_id   = E-100
trace_id       = T-100

Así podemos reconstruir:

Request
  ↓
UserCreated
  ↓
MembershipCreated
10. Context Model

Proponemos:

@dataclass(frozen=True)
class TraceContext:
    request_id: str
    correlation_id: str
    trace_id: str
    causation_id: str | None = None

Los formatos definitivos de los identificadores deben quedar centralizados y no repartidos por la aplicación.

11. Security Context vs Trace Context

No debemos mezclar:

SecurityContext

con:

TraceContext

La estructura conceptual será:

RequestContext
│
├── TraceContext
│   ├── request_id
│   ├── correlation_id
│   ├── trace_id
│   └── causation_id
│
└── SecurityContext
    ├── ActorContext
    └── TenantContext

Esto mantiene responsabilidades claras.

12. El Trace Context no autoriza

Nunca:

if correlation_id == trusted:
    allow()

Ni:

trace_id

puede utilizarse para:

identificar permisos
seleccionar tenant
elevar privilegios
bypassear MFA

El contexto de trazabilidad es exclusivamente operacional.

13. Entrada desde HTTP

Para HTTP podemos aceptar:

X-Correlation-ID

como identificador de correlación.

Pero:

El cliente no controla los privilegios por proporcionar un X-Correlation-ID.

El sistema debe validar formato y aplicar sus propias reglas.

14. Generación

Si el cliente no proporciona un correlation ID válido:

Request
   ↓
No valid correlation ID
   ↓
Generate new ID

No debemos rechazar automáticamente todas las requests por ausencia del header.

15. Request ID

El request_id debe ser generado por EVOXA para cada request.

Request A → R-A
Request B → R-B
Request C → R-C

Nunca reutilizar el mismo request_id entre requests independientes.

16. Correlation ID

Podemos mantener el mismo:

correlation_id

durante un flujo lógico.

Ejemplo:

API Request
      │
      ▼
Application
      │
      ▼
Domain Event
      │
      ▼
Outbox
17. Event Propagation

Cuando un Domain Event se transforma en Integration Event:

Domain Event
      │
      ▼
Event Envelope

debe conservar el contexto relevante:

event_id
correlation_id
causation_id
trace_id
actor
tenant

sin incluir secretos.

18. Causation Chain

Ejemplo:

Request
  │
  ▼
UserCreated
  │
  ▼
MembershipCreated
  │
  ▼
AuditEvent

Podemos representar:

UserCreated.causation_id
    ← request/operation source

MembershipCreated.causation_id
    ← UserCreated.event_id

AuditEvent.causation_id
    ← MembershipCreated.event_id

Esto permite reconstruir causalidad.

19. Correlation ≠ Causation

Importante:

correlation_id

puede permanecer igual:

C-001

mientras:

causation_id

cambia:

E-001
E-002
E-003

Ejemplo:

C-001
 │
 ├── E-001
 │
 ├── E-002 caused by E-001
 │
 └── E-003 caused by E-002
20. Trace Propagation

El trace context debe poder propagarse entre:

API
Application
Database
Events
Workers
External services

cuando técnicamente corresponda.

Pero no todos los sistemas deben compartir necesariamente el mismo trace_id si la herramienta de tracing establece un nuevo trace boundary.

La semántica definitiva del distributed tracing debe alinearse posteriormente con la plataforma de observabilidad.

21. Async Processing

Consideremos:

HTTP Request
    ↓
Outbox
    ↓
Event Platform
    ↓
Worker

El worker no debe perder la relación con la operación original.

Por ello el evento debe transportar:

correlation_id
causation_id

y la información de tracing necesaria.

22. Correlation Across Services

Ejemplo:

Identity API
     │
     │ correlation C-100
     ▼
Event Platform
     │
     ▼
Notification Service
     │
     ▼
Analytics

Los servicios pueden relacionar sus operaciones mediante:

correlation_id

sin compartir:

session secrets
access tokens
refresh tokens
23. Actor + Tenant + Trace

Una operación completa puede tener:

TraceContext
├── request_id
├── correlation_id
├── trace_id
└── causation_id

SecurityContext
├── ActorContext
│   ├── actor
│   └── authentication
│
└── TenantContext
    ├── tenant_id
    └── membership_id

Esto permite distinguir:

Who?
Where?
Which operation?
What caused it?
24. Logging

Los logs estructurados deberán poder incluir:

{
  "request_id": "...",
  "correlation_id": "...",
  "trace_id": "...",
  "actor_type": "USER",
  "actor_id": "...",
  "tenant_id": "..."
}

cuando dichos campos sean aplicables y estén autorizados para logging.

25. Minimización

No debemos registrar automáticamente todo el SecurityContext.

Especialmente:

❌ access token
❌ refresh token
❌ password
❌ password hash
❌ MFA secret
❌ recovery code
❌ private key
❌ API secret
❌ cookies

El contexto de observabilidad debe ser mínimo y seguro.

26. Audit

Los Audit Events podrán relacionarse con:

request_id
correlation_id
trace_id
causation_id

Esto permite pasar de:

Audit Event

a:

operational trace

sin convertir el audit log en un sistema de tracing.

27. Security Events

Lo mismo aplica a Security Events.

Ejemplo:

Refresh Token Reuse Detected

puede contener:

security_event_id
correlation_id
trace_id
causation_id
actor
tenant

pero nunca:

refresh_token
28. Database

Las operaciones de PostgreSQL pueden asociarse al:

trace_id

mediante observabilidad.

Sin embargo:

No debemos almacenar automáticamente todos los identificadores de tracing como columnas en todas las tablas.

El modelo de persistencia deberá decidir cuáles son datos de negocio, auditoría o infraestructura.

29. Outbox

El Outbox debe preservar el contexto necesario para publicar el evento.

Conceptualmente:

OutboxEvent
├── event_id
├── event_type
├── event_version
├── tenant_id
├── actor
├── correlation_id
├── causation_id
├── trace information
└── payload

La estructura definitiva queda gobernada por ESP-0001/12 y los ADR de Events.

30. Transaction Boundary

En:

BEGIN
    persist business state
    persist outbox event
COMMIT

el contexto debe permanecer disponible para ambos.

Por ejemplo:

Application
   │
   ├── Business Change
   │
   └── Outbox Event
          │
          ├── correlation
          ├── causation
          └── trace
31. Retries

Si una operación se reintenta:

Request
  ↓
Retry

no debemos asumir que:

request_id

y:

idempotency key

son lo mismo.

Esto es especialmente importante porque:

Traceability

e:

Idempotency

son mecanismos diferentes.

32. Idempotency

Una request puede utilizar:

Idempotency-Key: ABC-123

y además tener:

request_id = R-001
correlation_id = C-001
trace_id = T-001

Estos valores tienen diferentes responsabilidades:

Idempotency-Key
    → evita efectos duplicados

Request ID
    → identifica request

Correlation ID
    → relaciona operación

Trace ID
    → tracing

Causation ID
    → causalidad
33. Middleware

Se recomienda un middleware dedicado:

app/core/observability/
└── context/
    ├── trace.py
    ├── correlation.py
    ├── request.py
    └── accessor.py

Podemos reutilizar el mecanismo establecido en TASK-001-01 para X-Correlation-ID, pero ahora debemos evolucionarlo hacia un contexto transversal formal.

34. Context Accessor

Propuesta:

class TraceContextAccessor:

    def get(self) -> TraceContext | None:
        ...

    def require(self) -> TraceContext:
        ...

Esto evita que cada componente manipule directamente ContextVar.

35. ContextVar

Podemos utilizar:

ContextVar[TraceContext | None]

para mantener el contexto por request/task.

La regla:

NO global mutable trace state

es obligatoria.

36. Async Safety

Debe soportar:

Request A → Trace A
Request B → Trace B
Request C → Trace C

simultáneamente.

Nunca:

Request A → Trace B

por contaminación de contexto.

37. Cleanup

Al finalizar la request:

TraceContext
    ↓
reset

Debe utilizarse el mecanismo de token/reset de ContextVar para garantizar que el contexto anterior no contamine requests o tareas posteriores.

38. Background Jobs

Para un background job iniciado desde una request:

HTTP Request
    │
    ▼
Background Job

debemos decidir explícitamente qué contexto se propaga.

Por defecto:

correlation_id
causation_id

pueden ser propagados mediante el mensaje/job.

Pero el contexto no debe copiarse indiscriminadamente.

Especialmente:

❌ authentication credentials
❌ access tokens
❌ refresh tokens
39. Event Consumers

Un consumer recibe:

EventEnvelope

y crea su propio contexto operacional.

Conceptualmente:

Event
 │
 ├── correlation_id
 ├── causation_id
 └── trace metadata
       │
       ▼
Consumer Context

El consumer no debe asumir que el actor del evento es el actor de todas sus propias operaciones.

40. Actor vs Causation

Ejemplo:

USER A
  ↓
UserCreated
  ↓
SYSTEM job
  ↓
WelcomeNotificationSent

El evento puede conservar:

original actor = USER A

pero el nuevo procesamiento puede ejecutarse por:

actor = SYSTEM

Por eso:

actor context

y:

causation context

no deben confundirse.

41. AI Agent

Para AI:

AI_AGENT

puede generar:

correlation_id
trace_id
causation_id

igual que cualquier otro actor.

Ejemplo:

AI_AGENT
   ↓
Recommendation request
   ↓
Domain operation
   ↓
Event

La trazabilidad no proporciona ningún privilegio adicional.

42. AI Observability

Las operaciones de AI deberían poder identificarse como:

actor_type = AI_AGENT

cuando corresponda.

Esto permite analizar:

AI operation
↓
authorization
↓
tool call
↓
domain action
↓
event

manteniendo las mismas reglas de seguridad.

43. External Calls

Cuando EVOXA llame a otro servicio:

EVOXA
  ↓
External Service

debe propagarse únicamente el contexto de tracing permitido por el protocolo y la política.

Nunca debemos enviar automáticamente:

Authorization credentials
tenant secrets
session tokens

simplemente porque existe un trace context.

44. Error Handling

Cuando ocurre:

HTTP 500

la respuesta puede incluir:

{
  "error": {
    "code": "INTERNAL_ERROR",
    "message": "An unexpected error occurred.",
    "correlation_id": "..."
  }
}

Esto ya está alineado con el error envelope de EVOXA.

El correlation_id ayuda al soporte a localizar la operación sin revelar información interna.

45. Error Correlation

El cliente debe poder entregar:

correlation_id

a soporte.

Internamente podemos buscar:

correlation_id
   ↓
logs
   ↓
traces
   ↓
audit
   ↓
security events
   ↓
outbox/events

cuando corresponda.

46. No Sensitive IDs

Aunque los identificadores de trazabilidad no sean secretos, no debemos utilizarlos como contenedores de:

user information
tenant information
permissions
security decisions

Deben ser identificadores opacos.

47. Proposed Structure
apps/api/app/core/observability/
│
├── context/
│   ├── trace.py
│   ├── request.py
│   ├── correlation.py
│   └── accessor.py
│
├── logging/
│   └── context.py
│
└── ...

Y posteriormente:

identity/shared/
└── ...

puede consumir los contratos sin depender de una herramienta concreta de observabilidad.

48. Reutilización de TASK-001-01

TASK-001-01 ya estableció:

X-Correlation-ID

Por tanto, no debemos crear un segundo mecanismo paralelo.

La implementación correcta de TASK-001-14 es:

TASK-001-01
    ↓
basic correlation middleware
    ↓
TASK-001-14
    ↓
formal TraceContext

y posteriormente refactorizar el middleware inicial para consumir el nuevo contexto.

49. Architecture Rule

La dependencia debe quedar:

Presentation/API
       ↓
Observability Context
       ↓
Application
       ↓
Domain Events

No:

Domain
  ↓
FastAPI Request

ni:

Domain
  ↓
OpenTelemetry SDK

El Domain debe permanecer independiente de la tecnología de tracing.

50. Domain Events

Los Domain Events pueden contener o asociarse a metadatos de contexto.

Pero debemos mantener:

Domain Event

separado de:

Transport Envelope

Por tanto:

Domain
  ↓
DomainEvent
  ↓
Mapper
  ↓
EventEnvelope
  ↓
Outbox

El transport envelope es quien incorpora los metadatos necesarios para integración.

51. Testing
Request ID
Given:
  request enters API

When:
  context is created

Then:
  request_id is unique
Correlation ID
Given:
  valid X-Correlation-ID

When:
  request enters API

Then:
  correlation_id is preserved
Missing Correlation
Given:
  no X-Correlation-ID

When:
  request enters API

Then:
  EVOXA generates one
52. Context Isolation
Given:
  Request A → C-A
  Request B → C-B

When:
  both execute concurrently

Then:
  A never sees C-B
  B never sees C-A
53. Context Cleanup
Given:
  Request A has trace context

When:
  Request A completes

Then:
  context is cleared

And:
  Request B does not inherit A
54. Causation
Given:
  Event A causes Event B

Then:
  B.causation_id = A.event_id
55. Correlation Chain
Given:
  multiple events belong to one workflow

Then:
  they may share correlation_id

pero:

causation_id

debe representar la relación causal inmediata.

56. Security Test

Debe verificarse:

TraceContext

no puede:

change actor
change tenant
change role
change permission
bypass MFA
produce ALLOW
57. AI Test
Given:
  AI_AGENT executes an operation

Then:
  trace context identifies the operation

And:
  tracing does not alter authorization
58. Acceptance Criteria
ID	Criterio
AC-01	Existe TraceContext
AC-02	Existe Request ID
AC-03	Existe Correlation ID
AC-04	Existe Trace ID
AC-05	Existe Causation ID
AC-06	Los cuatro conceptos están separados
AC-07	Correlation ID puede propagarse entre componentes
AC-08	Request ID es único por request
AC-09	Trace context es inmutable
AC-10	No existe estado global mutable
AC-11	Context funciona con requests concurrentes
AC-12	Context se limpia al finalizar
AC-13	Events pueden conservar correlation/causation
AC-14	Outbox conserva contexto requerido
AC-15	Logs estructurados pueden correlacionarse
AC-16	Audit puede correlacionarse
AC-17	Security Events pueden correlacionarse
AC-18	Trace context no participa en autorización
AC-19	No contiene secretos
AC-20	AI Agent utiliza el mismo mecanismo
59. Definition of Done
[ ] TraceContext
[ ] Request ID
[ ] Correlation ID
[ ] Trace ID
[ ] Causation ID
[ ] ContextVar/accessor
[ ] HTTP middleware integration
[ ] TASK-001-01 correlation mechanism refactored
[ ] Application propagation
[ ] Domain/Event mapping support
[ ] Outbox metadata support
[ ] Structured logging integration
[ ] Error response correlation
[ ] Async isolation
[ ] Context cleanup
[ ] Background propagation rules
[ ] Event consumer propagation rules
[ ] Security tests
[ ] AI tracing tests
[ ] Architecture tests
[ ] Documentation
60. Trazabilidad
IS-001
 │
 └── TASK-001-14
       │
       ├── ESP-0001/09
       │     Audit & Compliance
       │
       ├── ESP-0001/12
       │     Events & Integration
       │
       ├── ESP-0001/14
       │     Implementation Requirements
       │
       ├── ADR-007
       │     Transactional Outbox
       │
       ├── ADR-018
       │     Central Event Broker
       │
       ├── ADR-019
       │     Event Streaming & Ordering
       │
       ├── ADR-022
       │     Consumer Idempotency
       │
       ├── ADR-028
       │     Observability & Operational Security
       │
       ├── ADR-053
       │     Cache Invalidation Event Model
       │
       ├── ADR-067
       │     Identity Module Boundaries
       │
       └── ADR-069
             Module Interaction Rules
61. Modelo final de Contextos

Con las tareas 12, 13 y 14, estamos construyendo una base transversal bastante importante:

                         REQUEST
                            │
              ┌─────────────┴─────────────┐
              │                           │
              ▼                           ▼
       TRACE CONTEXT               SECURITY CONTEXT
              │                           │
       ┌──────┼──────┐              ┌─────┴──────┐
       │      │      │              │            │
       ▼      ▼      ▼              ▼            ▼
    request correlation trace     ACTOR        TENANT
      id        id      id          │            │
                       │            │            │
                       ▼            ▼            ▼
                   causation     Auth        Membership
                      id

Y posteriormente:

TraceContext
      +
SecurityContext
      ↓
Application Operation
      ↓
Authorization Runtime
      ↓
Domain
      ↓
Outbox
      ↓
Event Platform
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
Siguiente tarea

TASK-001-15 — Exception Model

Esta será importante porque vamos a establecer un modelo único de errores de EVOXA: errores de dominio, aplicación, infraestructura, autenticación, autorización, tenant, concurrencia y errores internos, incluyendo cómo se transforman en el error envelope /api/v1 sin filtrar información sensible.
