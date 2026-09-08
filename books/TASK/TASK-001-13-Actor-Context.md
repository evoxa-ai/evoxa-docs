TASK-001-13 — Actor Context

IS: IS-001 — Identity Domain Foundation
ESP: ESP-0001/14 — Identity Implementation Requirements
Dependencias: TASK-001-11 — Security Foundation, TASK-001-12 — Tenant Context Foundation
Prioridad: 🔴 Critical
Estado: Ready for Implementation

1. Objetivo

Establecer el Actor Context de EVOXA Identity.

Esta tarea define cómo el sistema representa de manera segura quién está ejecutando una operación, independientemente de si se trata de:

USER
SERVICE
SYSTEM
AI_AGENT

La distinción fundamental será:

Authentication ≠ Actor ≠ Authorization

Authentication determina cómo se autenticó una identidad.

Actor representa quién ejecuta la operación.

Authorization determina qué puede hacer ese actor.

2. Problema que resolvemos

Queremos evitar que las capas internas trabajen con información ambigua como:

user_id = request.user

o:

actor = request.headers["X-Actor-ID"]

La identidad del actor debe provenir de un contexto de seguridad validado.

Flujo:

Request
   │
   ▼
Authentication
   │
   ▼
Authenticated Identity
   │
   ▼
Actor Context
   │
   ▼
Tenant Context
   │
   ▼
Authorization
   │
   ▼
Application Operation
3. Actor Model

EVOXA utilizará cuatro tipos de actor:

USER
SERVICE
SYSTEM
AI_AGENT

Esto está alineado con los modelos de Audit/Security Events y con la arquitectura Identity.

4. USER

Representa una persona autenticada.

Actor
├── type = USER
└── id = user_id

Ejemplo conceptual:

USER
  ID: 7c...

El actor no debe contener automáticamente:

password
email
roles completos
permissions completos
MFA secrets
tokens
5. SERVICE

Representa una identidad técnica utilizada por otro servicio.

SERVICE
   │
   └── service_id

Ejemplo:

Billing Service
     ↓
Identity
     ↓
SERVICE actor

Los servicios no deben impersonar usuarios automáticamente.

6. SYSTEM

Representa operaciones internas controladas por la plataforma.

Por ejemplo:

SYSTEM
  ↓
scheduled security operation

Debe existir una identidad operacional clara para auditoría.

7. AI_AGENT

Representa un agente de IA con identidad propia.

AI_AGENT
   │
   └── agent/service identity

Regla fundamental:

Un AI Agent es un actor, no una excepción de seguridad.

Por tanto:

AI_AGENT
   ↓
Authentication
   ↓
Tenant Context
   ↓
Authorization
   ↓
Policy

exactamente como cualquier otro actor.

8. Actor ≠ User

No debemos modelar:

Actor
  = User

porque:

SERVICE
SYSTEM
AI_AGENT

también pueden ejecutar operaciones.

La abstracción será:

Actor
├── type
└── id
9. Actor Identity

Propuesta inicial:

@dataclass(frozen=True)
class Actor:
    type: ActorType
    id: UUID | None

Para USER:

id = user_id

Para SERVICE:

id = service_account_id

Para AI_AGENT:

id = agent identity

Para SYSTEM:

id = system identity cuando corresponda

No debemos imponer artificialmente un UUID cuando conceptualmente no exista.

10. Actor Type
class ActorType(str, Enum):
    USER = "USER"
    SERVICE = "SERVICE"
    SYSTEM = "SYSTEM"
    AI_AGENT = "AI_AGENT"

Este enum debe ser compartido por:

Security
Audit
Security Events
Authorization
Observability

evitando definiciones divergentes.

11. Actor Context

El contexto será:

ActorContext

y deberá representar:

actor
authentication context
tenant context

sin duplicar responsabilidades.

Conceptualmente:

ActorContext
├── Actor
└── AuthenticationContext

y:

SecurityContext
├── ActorContext
└── TenantContext
12. Separación de conceptos

Es importante mantener:

Actor

como identidad de ejecución.

AuthenticationContext

como evidencia/estado de autenticación.

TenantContext

como contexto organizacional.

AuthorizationDecision

como resultado de autorización.

Por tanto:

Actor
    ≠
Authentication
    ≠
Tenant
    ≠
Authorization
13. Authentication Context

Podemos preparar:

@dataclass(frozen=True)
class AuthenticationContext:
    authenticated: bool
    authentication_method: str | None
    session_id: UUID | None

Pero algunos campos, como:

MFA assurance
authentication strength
step-up evidence

serán ampliados posteriormente por Authentication/MFA.

No debemos inventar todavía su modelo definitivo.

14. Actor Context completo

Conceptualmente:

SecurityContext
│
├── ActorContext
│   ├── Actor
│   └── AuthenticationContext
│
└── TenantContext
    ├── tenant_id
    └── membership_id

Esto proporciona:

Who?
How authenticated?
In which tenant?
Through which membership?
15. Construcción del Actor

El actor no debe venir directamente del cliente.

Incorrecto:

X-Actor-ID: 123

y:

actor = Actor(USER, 123)

Correcto:

Credential / Token
       │
       ▼
Authentication
       │
       ▼
Validated Identity
       │
       ▼
ActorContext
16. Actor Spoofing

El sistema debe impedir:

Authenticated User A
       │
       └── claims to be User B

Por ejemplo:

X-User-ID: B

nunca debe sobreescribir la identidad obtenida de autenticación.

17. Source of Truth

Para requests autenticadas:

Authentication result
        ↓
Actor

es la fuente confiable del actor.

No:

request body
query parameter
HTTP header
URL path
18. Actor Context Immutability

Una vez creado:

ActorContext

debe ser inmutable.

Request
   ↓
Actor A
   ↓
Application

No puede convertirse durante la operación en:

Actor B
19. ContextVar

Al igual que Tenant Context, podemos transportar el contexto mediante una abstracción basada en contextvars.

Propuesta:

core/security/context/
├── actor.py
├── authentication.py
├── tenant.py
├── security_context.py
└── accessor.py

Pero los módulos no deben manipular directamente la variable global de contexto.

20. Actor Context Accessor

Propuesta:

class ActorContextAccessor:

    def get(self) -> ActorContext | None:
        ...

    def require(self) -> ActorContext:
        ...

require() debe fallar cuando una operación requiere actor autenticado y no existe contexto válido.

21. Anonymous Requests

No todas las requests tienen actor.

Por ejemplo:

POST /api/v1/auth/login

antes de autenticarse.

Por tanto:

ActorContext = None

puede ser válido en determinados endpoints públicos.

Pero:

authenticated endpoint
+
ActorContext missing

debe producir:

UNAUTHENTICATED

y nunca:

SYSTEM

automáticamente.

22. No Default Actor

Nunca debemos hacer:

actor = SYSTEM

cuando no existe actor.

Eso podría convertir una request anónima en una operación privilegiada.

Regla:

No actor
    ≠
SYSTEM actor
23. SYSTEM debe ser explícito

Las operaciones internas deben crear explícitamente:

ActorType.SYSTEM

por ejemplo:

Scheduled Security Job
       ↓
SYSTEM actor
       ↓
Identity operation

No se obtiene por ausencia de autenticación.

24. SERVICE Authentication

Un servicio debe autenticarse mediante mecanismos de Service-to-Service definidos posteriormente:

OAuth2.1 Client Credentials
Signed JWT
mTLS
Workload Identity

según ADR-038.

Después:

Authenticated Service
        ↓
SERVICE Actor
25. AI Agent Authentication

Un AI Agent también debe tener identidad.

Conceptualmente:

AI Agent
   ↓
Workload / Service Identity
   ↓
Authentication
   ↓
AI_AGENT Actor

No:

AI Agent
   ↓
SYSTEM

por defecto.

26. AI Delegated User Context

En el futuro un AI Agent podría actuar en nombre de un usuario:

User
   ↓
AI Agent
   ↓
Operation

Esto no debe destruir la distinción entre:

actor
delegator
delegated authority

Debe utilizarse el modelo explícito de delegación.

27. Impersonation

La suplantación administrativa será tratada como operación especial.

No debemos permitir:

Admin
 ↓
pretend to be User

simplemente cambiando:

Actor.id

Si se implementa impersonation:

Original Actor
      ↓
Impersonation Authority
      ↓
Target User
      ↓
Explicit Security Context

deberá ser:

scoped
time-bound
audited
policy-controlled
28. Delegation vs Impersonation

Son conceptos diferentes.

Delegation
User A
   ↓ delegates capability
User B / Agent

B actúa como B.

Impersonation
Admin
   ↓
acts as User A

El contexto de identidad cambia de manera explícita.

No debemos mezclarlos.

29. Actor + Tenant

Actor Context y Tenant Context trabajan juntos:

Actor
  +
Membership
  +
Tenant

Ejemplo:

Actor:
  USER-123

Tenant:
  ORGANIZATION-A

Membership:
  MEMBERSHIP-456

Esto representa:

User 123 está actuando dentro de Organization A mediante Membership 456.

30. Actor + Roles

No debemos colocar todos los roles dentro de ActorContext como fuente de verdad.

Incorrecto:

Actor
 └── roles = ["ADMIN"]

y utilizar eso directamente para autorizar.

Los roles efectivos deben resolverse mediante:

Membership
+
Role
+
Permission
+
Policy
31. Actor + Permissions

Tampoco:

Actor
 └── permissions = [...]

como autorización definitiva.

La autorización utiliza el Authorization Runtime.

Esto evita que un contexto antiguo contenga permisos obsoletos.

32. Actor + Security State

El ActorContext podrá posteriormente incorporar información derivada de:

account status
session state
MFA
risk
security epoch

pero estas fuentes siguen siendo autoritativas fuera del objeto Actor.

No debemos convertir ActorContext en una copia completa de Identity.

33. Security Context Creation

El objetivo final:

Authentication Result
        │
        ▼
ActorContext
        │
        ▼
Tenant Resolution
        │
        ▼
TenantContext
        │
        ▼
SecurityContext

La construcción debe ser determinista.

34. Application Layer

Los Use Cases pueden recibir explícitamente:

async def execute(
    command: Command,
    security_context: SecurityContext,
):
    ...

Esto tiene una ventaja:

security dependency
        ↓
visible in use case

en lugar de esconder toda la seguridad en variables globales.

35. Domain Layer

El Domain no debería depender de FastAPI:

❌ Request
❌ Header
❌ JWT
❌ HTTP

Si una regla de dominio necesita conocer al actor, debe recibir una abstracción apropiada.

Por ejemplo:

ActorId

o un concepto de dominio específico.

36. Application vs Domain Actor

Hay que evitar pasar todo el:

SecurityContext

al Domain indiscriminadamente.

El Application Layer determina qué información necesita el caso de uso.

Por ejemplo:

Application
  SecurityContext
       ↓
  actor_id
       ↓
Domain command

Esto mantiene el Domain desacoplado de seguridad de infraestructura.

37. Actor Authorization

El Authorization Runtime recibirá:

Actor
+
Authentication Context
+
Tenant Context
+
Request Context
+
Resource
+
Operation

y producirá:

ALLOW
DENY
CHALLENGE
REAUTHENTICATE
REQUIRE_MFA
REVOKE

según ADR-049.

38. Actor Context y Authorization Cache

El actor será parte fundamental del contexto de autorización.

Una decisión no puede reutilizarse incorrectamente:

Actor A

para:

Actor B

La cache deberá estar ligada al actor y al contexto relevante.

Esto será desarrollado en las tareas de Authorization Cache.

39. Actor Context y Audit

Cada operación relevante deberá poder identificar:

actor_type
actor_id

Por ejemplo:

{
  "actor_type": "AI_AGENT",
  "actor_id": "..."
}

Esto permite diferenciar:

USER
SERVICE
SYSTEM
AI_AGENT

en auditoría.

40. Actor Context y Events

Los eventos utilizarán:

actor_type
actor_id

cuando corresponda.

Ejemplo:

{
  "event_type": "SecurityPolicyChanged",
  "actor_type": "USER",
  "actor_id": "..."
}

Los secretos nunca deben formar parte del actor context ni del event envelope.

41. Correlation

El Actor Context debe coexistir con:

correlation_id
trace_id
causation_id

pero no deben mezclarse.

Conceptualmente:

Request Context
├── Correlation
├── Trace
│
└── Security Context
    ├── Actor
    ├── Authentication
    └── Tenant
42. Concurrency

Debe funcionar correctamente:

Request A → USER-1
Request B → USER-2
Request C → SERVICE-1
Request D → AI_AGENT-1

simultáneamente.

Nunca:

Request A → USER-2

por contaminación de contexto.

43. Context Leakage Test

Escenario:

Request A
Actor = USER-A
      ↓
complete

Request B
Actor = USER-B
      ↓
must NOT inherit USER-A

Y:

Request C
Actor = None
      ↓
must NOT inherit previous actor
44. Cross-Tenant + Actor Test

Debe probarse:

Actor A
Tenant A

y simultáneamente:

Actor B
Tenant B

para garantizar que:

Actor A → Tenant B

no pueda aparecer accidentalmente por contaminación de contexto.

45. Actor Spoofing Tests

Debe probarse que:

JWT → User A
Header → User B

produce:

Actor = User A

y el header no puede reemplazar la identidad autenticada.

46. Anonymous Test
Request
Authorization = none

Resultado:

ActorContext = None

hasta que un endpoint explícitamente público lo permita.

Para un endpoint protegido:

DENY / UNAUTHENTICATED
47. AI Agent Test
ActorType = AI_AGENT

debe producir:

normal authorization flow

No:

automatic ALLOW
48. Service Test
ActorType = SERVICE

debe utilizar:

S2S authentication

y luego:

RBAC + ABAC + Policy + Tenant Isolation

según corresponda.

49. System Test
ActorType = SYSTEM

solamente puede ser creado por una ruta interna autorizada.

Una request HTTP anónima no puede declararse:

actor_type=SYSTEM
50. Proposed Files
apps/api/app/core/security/
│
├── context/
│   ├── actor.py
│   ├── authentication.py
│   ├── security_context.py
│   ├── accessor.py
│   └── exceptions.py
│
└── actor/
    ├── types.py
    └── factory.py

Podemos simplificar la estructura para evitar duplicación:

core/security/context/
├── actor.py
├── authentication.py
├── tenant.py
├── security_context.py
└── accessor.py

Recomiendo esta segunda opción por ahora.

51. Modelo propuesto
from dataclasses import dataclass
from enum import Enum
from uuid import UUID


class ActorType(str, Enum):
    USER = "USER"
    SERVICE = "SERVICE"
    SYSTEM = "SYSTEM"
    AI_AGENT = "AI_AGENT"


@dataclass(frozen=True)
class Actor:
    type: ActorType
    id: UUID | None

Authentication:

@dataclass(frozen=True)
class AuthenticationContext:
    authenticated: bool
    authentication_method: str | None = None
    session_id: UUID | None = None

Actor Context:

@dataclass(frozen=True)
class ActorContext:
    actor: Actor
    authentication: AuthenticationContext
52. Integración con SecurityContext

La estructura conceptual queda:

@dataclass(frozen=True)
class SecurityContext:
    actor_context: ActorContext
    tenant_context: TenantContext | None

Para requests públicas:

tenant_context = None

y posiblemente tampoco exista ActorContext.

No debemos forzar objetos ficticios para representar ausencia de autenticación.

53. Factory

La creación debe centralizarse:

class ActorContextFactory:

    def from_authentication(
        self,
        authentication_result,
    ) -> ActorContext:
        ...

Así evitamos que cada módulo construya actores de manera diferente.

54. Security Boundary

La factory debe aceptar únicamente resultados de autenticación confiables.

No:

factory.create(
    actor_id=request.headers["X-Actor-ID"]
)

Sí:

Validated Authentication Result
        ↓
ActorContextFactory
        ↓
ActorContext
55. Error Model

Errores internos:

ActorContextMissing
InvalidActorContext
InvalidAuthenticationContext
ActorContextAlreadySet

deben ser errores internos controlados.

Nunca debemos revelar detalles sensibles al cliente.

56. Context Lifecycle
HTTP request begins
       │
       ▼
Authentication
       │
       ▼
ActorContext created
       │
       ▼
TenantContext created
       │
       ▼
SecurityContext created
       │
       ▼
Application
       │
       ▼
HTTP response
       │
       ▼
Context cleared

La limpieza es obligatoria.

57. Context Cleanup

Después de cada request:

ActorContext → cleared
TenantContext → cleared
SecurityContext → cleared

Esto es crítico para evitar:

request-to-request leakage
58. Acceptance Criteria
ID	Criterio
AC-01	Existe Actor abstraction
AC-02	Se soporta USER
AC-03	Se soporta SERVICE
AC-04	Se soporta SYSTEM
AC-05	Se soporta AI_AGENT
AC-06	Actor es distinto de User
AC-07	Actor no puede ser proporcionado directamente por el cliente
AC-08	Actor deriva de autenticación validada
AC-09	Existe ActorContext
AC-10	Existe AuthenticationContext
AC-11	ActorContext es inmutable
AC-12	ActorContext se integra con SecurityContext
AC-13	Anonymous requests no reciben SYSTEM automáticamente
AC-14	Existe aislamiento entre requests concurrentes
AC-15	Existe context cleanup
AC-16	Existe protección contra actor spoofing
AC-17	SERVICE utiliza S2S authentication
AC-18	AI_AGENT sigue autorización normal
AC-19	SYSTEM actor requiere creación explícita
AC-20	Existen tests de actor/tenant/context isolation
59. Definition of Done
[ ] ActorType
[ ] Actor
[ ] AuthenticationContext
[ ] ActorContext
[ ] SecurityContext integration
[ ] ActorContextFactory
[ ] Context accessor
[ ] Context lifecycle
[ ] Context cleanup
[ ] USER support
[ ] SERVICE support
[ ] SYSTEM support
[ ] AI_AGENT support
[ ] Spoofing protection
[ ] Anonymous handling
[ ] Concurrency tests
[ ] Context leakage tests
[ ] AI isolation tests
[ ] Architecture tests
[ ] Documentation
60. Trazabilidad
IS-001
 │
 └── TASK-001-13
       │
       ├── ESP-0001/01
       │     Identity Domain Model
       │
       ├── ESP-0001/03
       │     Authentication & Authorization
       │
       ├── ESP-0001/04
       │     User & Organization Management
       │
       ├── ESP-0001/06
       │     Sessions & Token Management
       │
       ├── ESP-0001/07
       │     MFA
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
       ├── ADR-038
       │     S2S Authentication
       │
       ├── ADR-043
       │     Multi-Organization Identity
       │
       ├── ADR-044
       │     Membership / Delegation
       │
       ├── ADR-049
       │     Authorization Runtime
       │
       ├── ADR-067
       │     Identity Module Boundaries
       │
       └── ADR-069
             Module Dependencies & Contracts
61. Regla fundamental

Con TASK-001-12 y TASK-001-13, la frontera de contexto queda:

                         REQUEST
                            │
                            ▼
                     AUTHENTICATION
                            │
                            ▼
                          ACTOR
                            │
                  ┌─────────┴─────────┐
                  │                   │
                  ▼                   ▼
          ActorContext          Tenant Resolution
                  │                   │
                  │                   ▼
                  │             Membership
                  │                   │
                  │                   ▼
                  │             TenantContext
                  │                   │
                  └─────────┬─────────┘
                            ▼
                     SecurityContext
                            │
                            ▼
                      AUTHORIZATION
                            │
                            ▼
                         DECISION

Y las cuatro identidades posibles quedan:

USER
SERVICE
SYSTEM
AI_AGENT

ninguna obtiene privilegios automáticamente por su tipo.

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
Siguiente

TASK-001-14 — Correlation & Trace Context

Ahí vamos a separar formalmente Correlation ID, Request ID, Trace ID y Causation ID, establecer cómo viajan por API → Application → Domain Events → Outbox → Event Platform, y cómo se conectan con Audit, Security Events, Observability y AI Agent operations sin convertirlos en datos de autorización ni exponer información sensible.
