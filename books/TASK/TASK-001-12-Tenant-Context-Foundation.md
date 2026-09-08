TASK-001-12 — Tenant Context Foundation

IS: IS-001 — Identity Domain Foundation
ESP: ESP-0001/14 — Identity Implementation Requirements
Dependencias: TASK-001-03, TASK-001-04, TASK-001-06, TASK-001-07, TASK-001-10, TASK-001-11
Prioridad: 🔴 Critical
Estado: Ready for Implementation

1. Objetivo

Establecer la infraestructura transversal que representa y valida el Tenant Context de EVOXA.

En EVOXA, el tenant no debe inferirse simplemente desde el usuario:

User ≠ Tenant

La relación correcta es:

User
 │
 ├── Membership ──→ Organization A
 │
 ├── Membership ──→ Organization B
 │
 └── Membership ──→ Organization C

Por lo tanto:

Membership es la frontera de acceso entre un User y una Organization.

El TenantContext representa la organización activa dentro de una operación y debe validarse antes de permitir acceso a recursos tenant-scoped.

2. Qué problema resolvemos

Queremos impedir situaciones como:

Usuario pertenece a:
    Organization A
    Organization B

Request:
    tenant_id = Organization C

Resultado:
    ❌ DENY

También:

User A
  ↓
Organization A
  ↓
Resource A

no debe poder acceder automáticamente a:

Organization B
  ↓
Resource B

aunque conozca el ID del recurso.

3. Modelo conceptual

La estructura fundamental será:

┌──────────────┐
│     User     │
└──────┬───────┘
       │
       │
┌──────▼───────┐
│  Membership  │
└──────┬───────┘
       │
       │
┌──────▼──────────┐
│  Organization   │
│    (Tenant)     │
└─────────────────┘

Y durante una request:

Request
   │
   ▼
Authenticated Actor
   │
   ▼
Tenant Context
   │
   ▼
Membership Validation
   │
   ▼
Application Use Case
   │
   ▼
Authorization
4. Tenant ≠ Organization técnicamente

En la arquitectura de EVOXA:

Organization

es la entidad que representa la frontera tenant propuesta.

Conceptualmente:

tenant_id = organization_id

pero debemos mantener el concepto de Tenant Context separado porque posteriormente podrían existir otras formas de tenancy.

Por eso no queremos llenar todo el código con:

organization_id

cuando realmente estamos hablando de:

tenant_id

en el contexto de seguridad.

5. Principios

TASK-001-12 establece:

TENANT-001  Tenant isolation by default
TENANT-002  Membership is access boundary
TENANT-003  Explicit tenant context
TENANT-004  Tenant context must be validated
TENANT-005  No implicit tenant switching
TENANT-006  Cross-tenant access denied by default
TENANT-007  Tenant context is immutable
TENANT-008  Tenant uncertainty fails closed
TENANT-009  AI follows identical tenant rules
TENANT-010  Platform exceptions must be explicit and audited
6. Tenant Context

Proponemos:

@dataclass(frozen=True)
class TenantContext:
    tenant_id: UUID
    membership_id: UUID

Pero conceptualmente podemos necesitar posteriormente:

tenant_id
membership_id
user_id
organization_id
membership_status
role context
security_epoch

No debemos añadir todos estos campos todavía.

El contexto debe mantenerse mínimo.

7. Actor Context + Tenant Context

TASK-001-11 creó el concepto:

Actor

Ahora tendremos:

SecurityContext
├── Actor
└── TenantContext

Conceptualmente:

SecurityContext
       │
       ├── Actor
       │     ├── type
       │     └── id
       │
       └── TenantContext
             ├── tenant_id
             └── membership_id
8. Contexto inmutable

Una vez construido:

TenantContext

no debe modificarse durante la operación.

Incorrecto:

tenant = A

execute_operation()

tenant = B

execute_other_operation()

Correcto:

Request
   ↓
Tenant A
   ↓
Immutable Context
   ↓
Entire Operation
9. Resolución del Tenant

La fuente del tenant activo puede venir inicialmente de:

authenticated request
+
explicit tenant context

Por ejemplo:

X-Tenant-ID: <organization-id>

o posteriormente mediante:

JWT claim
subdomain
route
session context
API contract

Pero:

La presencia de un tenant_id en la request nunca constituye autorización.

10. Header de Tenant

Si utilizamos inicialmente:

X-Tenant-ID

debe considerarse únicamente:

tenant selection hint

No:

authorization proof

Flujo:

X-Tenant-ID
     │
     ▼
Parse UUID
     │
     ▼
Authenticated Actor
     │
     ▼
Membership lookup
     │
     ▼
Validate membership
     │
     ├── valid → TenantContext
     │
     └── invalid → DENY
11. No confiar en el cliente

Nunca:

tenant_id = request.headers["X-Tenant-ID"]

y después:

return repository.list(tenant_id)

sin validar membership.

Eso produciría:

IDOR / Cross-Tenant Access

El tenant proporcionado por el cliente siempre es untrusted input.

12. Membership Validation

Para construir un TenantContext válido:

Actor
  +
Tenant ID
  ↓
Membership
  ↓
Membership valid?

Debe verificarse:

membership exists
membership belongs to actor
membership belongs to tenant
membership active
membership not revoked/suspended

Las reglas definitivas del lifecycle de Membership serán desarrolladas en las tareas correspondientes.

13. Membership como Boundary

La regla central:

User
  │
  ▼
Membership
  │
  ▼
Organization

No:

User
  │
  └────────→ Organization

Esto es importante porque un usuario puede pertenecer a múltiples organizaciones.

14. Active Tenant

Un usuario puede tener:

Membership A → Organization A
Membership B → Organization B
Membership C → Organization C

Pero una request determinada debe tener un contexto activo:

Current Request
      ↓
Active Membership
      ↓
Active Tenant

No existe una autorización implícita sobre todas las organizaciones.

15. Tenant Switching

Cambiar de organización debe ser una operación explícita:

Organization A
     ↓
switch tenant
     ↓
validate Membership B
     ↓
Organization B

No:

request A
   ↓
request magically becomes B

El cambio de contexto debe generar posteriormente los eventos/auditoría correspondientes cuando sea una operación relevante.

16. Tenant Context Lifetime

El contexto debe existir únicamente durante la operación:

HTTP Request
      │
      ▼
SecurityContext
      │
      ▼
Use Case
      │
      ▼
Response
      │
      ▼
Context discarded

No debemos almacenar el tenant actual en:

global variables
singleton mutable state
process-global state

Esto es especialmente importante con FastAPI async.

17. Async Safety

Nunca:

current_tenant = organization_a

como variable global.

Con requests concurrentes:

Request A → Tenant A
Request B → Tenant B
Request C → Tenant C

cada una debe tener su propio contexto.

18. ContextVar

Para transportar contexto dentro de una request podemos utilizar una abstracción basada en contextvars.

Por ejemplo:

core/security/context/
├── actor.py
├── tenant.py
├── security_context.py
└── accessor.py

Conceptualmente:

tenant_context_var: ContextVar[TenantContext | None]

Pero el acceso debe estar encapsulado.

No queremos que cualquier módulo manipule directamente el ContextVar.

19. Context Accessor

Podemos definir:

class TenantContextAccessor:

    def get(self) -> TenantContext | None:
        ...

    def require(self) -> TenantContext:
        ...

require() debe fallar de forma segura si no existe contexto.

20. Tenant Required

Para una operación tenant-scoped:

TenantContext = required

Si falta:

DENY

No:

assume first organization

ni:

assume user default organization
21. Platform Scope

No todo en Identity necesita tenant.

Existen operaciones de plataforma:

Platform
  ↓
Tenantless

Por ejemplo, ciertos procesos internos.

Pero una operación platform-scoped debe declararse explícitamente.

No debemos usar:

tenant_id = null

como mecanismo genérico para saltarse tenant isolation.

22. Scope

Podemos introducir conceptualmente:

class ResourceScope(str, Enum):
    PLATFORM = "PLATFORM"
    ORGANIZATION = "ORGANIZATION"

Esto posteriormente se alineará con los scopes de Policy:

PLATFORM
ORGANIZATION
RESOURCE
23. Tenant Guard

El Tenant Guard será posteriormente parte del Authorization Runtime.

Pero esta tarea establece su foundation:

Request
   │
   ▼
Authentication
   │
   ▼
Tenant Guard
   │
   ├── valid → continue
   │
   └── invalid → DENY

Esto corresponde a la regla de que tenant validation ocurre antes de una autorización positiva.

24. Tenant Guard ≠ Authorization

Importante:

Tenant Guard

responde:

¿El contexto tenant de esta request es válido para este actor?

Mientras:

Authorization Runtime

responde:

¿Este actor puede realizar esta operación sobre este recurso bajo las políticas aplicables?

Son conceptos relacionados pero diferentes.

25. Tenant Guard ≠ Repository Filter

No debemos pensar que:

WHERE organization_id = :tenant_id

es suficiente.

Eso es una defensa de persistencia.

La seguridad requiere:

Tenant Context
      +
Tenant Guard
      +
Authorization
      +
Repository scoping
26. Repository Integration

Los repositories tenant-scoped deberán recibir explícitamente el contexto necesario.

Por ejemplo, conceptualmente:

await membership_repository.list_for_tenant(
    tenant_id=tenant_context.tenant_id
)

Pero el repository no debe ser responsable de decidir si el actor tiene autorización.

Esa responsabilidad pertenece al Authorization boundary.

27. Defense in Depth

Tendremos:

Request
   ↓
Authentication
   ↓
Tenant Context
   ↓
Tenant Guard
   ↓
Authorization
   ↓
Application
   ↓
Repository Tenant Scope
   ↓
Database

Si una capa falla, otra reduce el riesgo.

28. Database Boundary

Más adelante podremos reforzar PostgreSQL con mecanismos como:

tenant-aware queries
Row Level Security
database roles
transaction tenant context

Pero TASK-001-12 no debe introducir todavía una implementación específica de PostgreSQL RLS.

Eso corresponde a la especificación de Persistence/Data Model.

29. Cross-Tenant Access

Default:

Tenant A → Tenant B
       ❌ DENY

Incluso si:

actor authenticated

Incluso si:

actor is admin

La excepción debe ser explícita:

Platform-scoped authority
+
explicit policy
+
audit
30. Platform Admin

Un administrador de plataforma puede tener capacidades cross-tenant, pero no debe ocurrir simplemente porque tenga:

role = ADMIN

La decisión deberá pasar por:

RBAC
+
ABAC
+
Tenant Policy
+
Ownership
+
Security Policy Engine

y las reglas de cross-tenant definidas explícitamente.

31. AI Agent

Un AI Agent puede tener:

ActorType.AI_AGENT

pero eso no elimina tenant isolation.

Ejemplo:

AI Agent
   ↓
Tenant A
   ↓
Resource A

no puede acceder automáticamente a:

Tenant B
32. AI Cross-Tenant

Si una operación legítima necesita información de múltiples organizaciones:

AI Agent
    ↓
Platform-authorized operation
    ↓
Explicit policy
    ↓
Tenant scope
    ↓
Audit

No:

AI Agent
    ↓
ignore tenant
33. Tenant Context y Cache

Esto será especialmente importante para Authorization Cache.

Una cache key nunca debe ser:

user_id + resource + action

si falta tenant context.

Conceptualmente deberá incluir:

tenant
+
actor
+
membership
+
resource
+
action
+
relevant context

Esto será desarrollado por los ADR de Authorization Cache.

34. Tenant Context y Events

Los eventos tenant-scoped deberán transportar:

tenant_id

en su envelope cuando corresponda.

Por ejemplo:

{
  "event_id": "...",
  "event_type": "...",
  "tenant_id": "...",
  "actor": "...",
  "resource": "..."
}

Un evento de plataforma puede ser tenantless si realmente corresponde a Platform scope.

35. Tenant Context y Audit

Los audit events deben identificar:

tenant
organization
membership
actor

cuando estén disponibles y sean pertinentes.

Esto permite posteriormente demostrar:

Who?
What?
When?
Which tenant?
Which membership?
Which resource?
What decision?
36. Tenant Context y Correlation

El contexto deberá convivir con:

correlation_id
trace_id
causation_id

Conceptualmente:

SecurityContext
├── Actor
├── TenantContext
└── Request Metadata

No debemos mezclar todos los identificadores en una sola estructura sin separación semántica.

37. Tenant Context Validation Service

Podemos definir un application-level port:

class TenantContextValidator(ABC):

    @abstractmethod
    async def validate(
        self,
        actor_id: UUID,
        tenant_id: UUID,
    ) -> TenantContext:
        ...

La implementación consultará Membership.

38. Dependencia con Membership

Actualmente todavía no implementamos completamente:

Membership

Por ello TASK-001-12 puede crear el contrato, pero no debe inventar la persistencia definitiva.

Arquitectura:

Tenant Context
      │
      ▼
Membership Query Contract
      │
      ▼
Membership Module

No:

Tenant Context
      ↓
direct SQL membership table
39. No Cross-Module Database Access

Tenant Context no podrá hacer:

SELECT * FROM memberships

directamente desde:

core/security/

La consulta deberá pasar por el contrato público del módulo Membership.

Esto respeta la separación definida por la arquitectura modular.

40. Tenant Resolution Contract

Podemos separar:

TenantResolver

de:

TenantValidator
Resolver

Determina qué tenant solicita la request.

Request
 ↓
TenantResolver
 ↓
tenant_id
Validator

Determina si ese tenant es válido para el actor.

actor + tenant_id
 ↓
TenantValidator
 ↓
TenantContext

Esto evita mezclar selección con autorización.

41. Flujo completo

El flujo objetivo será:

HTTP Request
      │
      ▼
Correlation Context
      │
      ▼
Authentication
      │
      ▼
Actor
      │
      ▼
Tenant Resolver
      │
      ▼
Tenant ID
      │
      ▼
Membership Validator
      │
      ├─────────────┐
      │             │
      ▼             ▼
   VALID          INVALID
      │             │
      ▼             ▼
TenantContext     DENY
      │
      ▼
SecurityContext
      │
      ▼
Application
      │
      ▼
Authorization
42. Tenant Status

La organización puede tener estados como:

PENDING
ACTIVE
SUSPENDED

según el modelo de Organization.

Una membership válida no significa automáticamente que el tenant esté operativo.

Por ejemplo:

Membership = ACTIVE
Organization = SUSPENDED

no debe generar automáticamente un acceso ALLOW.

La validación de estado se integrará posteriormente con Authorization Policy.

43. Membership Status

De forma similar:

Membership = REVOKED

debe impedir la construcción de un contexto tenant válido.

No debemos confiar únicamente en:

membership exists
44. Security Epoch

Posteriormente el Tenant Context podrá interactuar con:

security_epoch

para invalidación de Authorization Cache.

Pero no debemos persistir ni definir aquí la semántica completa.

Eso ya está establecido en las ADR de cache/freshness.

45. Context Fingerprint

Authorization Cache utilizará posteriormente información del contexto para construir fingerprints.

Tenant Context debe ser estable y determinista:

tenant_id
membership_id

y no depender de datos arbitrarios del request.

46. Error Handling

Errores internos como:

membership lookup failed
database unavailable
tenant service unavailable

no deben convertirse en:

ALLOW

Regla:

Tenant validation unknown
        ↓
FAIL CLOSED
47. Diferencia entre DENY y ERROR

Internamente:

Membership does not exist
    → DENY

mientras:

Membership service unavailable
    → security failure / fail closed

No necesariamente debemos revelar al cliente si:

membership doesn't exist

o:

tenant doesn't exist

para evitar enumeration.

48. Anti-Enumeration

Una respuesta externa no debería permitir fácilmente descubrir:

Tenant A exists
Tenant B exists
Tenant C does not exist

a través de diferencias innecesarias.

Las respuestas de acceso deben mantener el error model seguro.

49. API Integration

Inicialmente podemos preparar:

core/security/context/

y middleware/dependencies:

core/security/dependencies.py

Más adelante los endpoints utilizarán:

tenant = require_tenant_context()

pero no deben resolver manualmente el tenant en cada endpoint.

50. Proposed Structure
apps/api/app/core/security/
│
├── context/
│   ├── actor.py
│   ├── tenant.py
│   ├── security_context.py
│   ├── accessor.py
│   └── exceptions.py
│
├── tenant/
│   ├── resolver.py
│   ├── validator.py
│   └── dependencies.py
│
└── ...

Y el contrato que consume Membership:

domains/identity/membership/
└── application/
    └── ports.py
51. Ejemplo conceptual

Request:

GET /api/v1/users
Authorization: Bearer ...
X-Tenant-ID: 8b...

Flujo:

JWT
 ↓
Actor = User-123
 ↓
Tenant ID = Organization-ABC
 ↓
Membership Query
 ↓
User-123 ∈ Organization-ABC ?
 ↓
YES
 ↓
TenantContext
 ↓
Authorization
 ↓
ALLOW/DENY
52. Caso cross-tenant
JWT
 ↓
Actor = User-123
 ↓
Tenant ID = Organization-XYZ
 ↓
Membership Query
 ↓
User-123 ∉ Organization-XYZ
 ↓
DENY

Aunque el usuario conozca:

Organization-XYZ ID

no obtiene acceso.

53. Caso sin tenant
GET /api/v1/users
Authorization: Bearer ...

sin contexto cuando el endpoint requiere tenant:

TenantContext missing
        ↓
DENY

No:

search all organizations
54. Caso Platform
Platform operation
      ↓
explicit PLATFORM scope
      ↓
platform authorization
      ↓
ALLOW/DENY

No se fuerza artificialmente:

tenant_id = fake UUID
55. Testing
Tenant Context válido
Given:
  actor has ACTIVE membership in tenant A

When:
  tenant A is requested

Then:
  TenantContext is created
Tenant no autorizado
Given:
  actor has no membership in tenant B

When:
  tenant B is requested

Then:
  access is denied
Membership revoked
Given:
  membership is revoked

When:
  actor requests tenant

Then:
  access is denied
Organization suspended
Given:
  organization is suspended

When:
  tenant context is used

Then:
  security policy prevents unauthorized access
56. Concurrency Test

Debemos probar:

Request A → Tenant A
Request B → Tenant B

simultáneamente.

El resultado esperado:

Request A → Context A
Request B → Context B

Nunca:

Request A → Context B
57. Context Leakage Test

Test importante:

Request A
  tenant A
  ↓
complete

Request B
  no tenant
  ↓
must NOT inherit tenant A

Esto evita errores con ContextVar y middleware.

58. AI Tenant Isolation Test
AI Agent
  ↓
Tenant A

debe ser incapaz de acceder a:

Tenant B

sin una autorización explícita de plataforma.

59. Cache Isolation Test

Posteriormente debemos comprobar:

Tenant A authorization result

nunca pueda ser utilizado para:

Tenant B

por una cache key incorrecta.

60. Acceptance Criteria
ID	Criterio
AC-01	Existe TenantContext
AC-02	TenantContext es inmutable
AC-03	TenantContext identifica tenant
AC-04	TenantContext identifica membership
AC-05	Actor y TenantContext están separados
AC-06	Existe TenantResolver
AC-07	Existe TenantValidator
AC-08	Membership es la frontera de acceso
AC-09	Tenant no autorizado produce DENY
AC-10	Tenant ausente en operación tenant-scoped produce DENY
AC-11	Cross-tenant access está bloqueado por defecto
AC-12	No existe estado tenant mutable global
AC-13	Contexto funciona correctamente con requests concurrentes
AC-14	Tenant validation falla closed
AC-15	No existe acceso directo a tablas Membership
AC-16	Tenant Context se integra con SecurityContext
AC-17	Tenant Context es compatible con Authorization
AC-18	Tenant Context es compatible con Event/Audit metadata
AC-19	AI Agents respetan tenant isolation
AC-20	Existen tests de aislamiento y context leakage
61. Definition of Done
[ ] TenantContext
[ ] TenantResolver
[ ] TenantValidator contract
[ ] Membership query contract
[ ] SecurityContext integration
[ ] Request context handling
[ ] ContextVar/accessor abstraction
[ ] Tenant-required dependency
[ ] Cross-tenant protection
[ ] Platform scope foundation
[ ] Fail-closed behavior
[ ] Anti-enumeration behavior
[ ] Concurrent request tests
[ ] Context leakage tests
[ ] Tenant isolation tests
[ ] AI tenant isolation tests
[ ] Architecture tests
[ ] Documentation
62. Trazabilidad
IS-001
 │
 └── TASK-001-12
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
       ├── ESP-0001/10
       │     Identity API Contracts
       │
       ├── ESP-0001/11
       │     Identity Persistence
       │
       ├── ESP-0001/14
       │     Implementation Requirements
       │
       ├── ADR-005
       │     Authorization Model
       │
       ├── ADR-043
       │     Multi-Organization Identity
       │
       ├── ADR-044
       │     Membership Roles & Delegation
       │
       ├── ADR-045
       │     Authorization Policy Model
       │
       ├── ADR-049
       │     Authorization Evaluation Runtime
       │
       ├── ADR-050–058
       │     Authorization Cache & Freshness
       │
       └── ADR-067/069
             Identity Module Boundaries
             Dependency & Interaction Rules
63. Regla fundamental

Después de TASK-001-12, debemos poder expresar la frontera de seguridad de EVOXA así:

                         REQUEST
                            │
                            ▼
                       AUTHENTICATION
                            │
                            ▼
                           ACTOR
                            │
                            ▼
                     TENANT RESOLUTION
                            │
                            ▼
                  MEMBERSHIP VALIDATION
                            │
                     ┌──────┴──────┐
                     │             │
                   VALID         INVALID
                     │             │
                     ▼             ▼
               TENANT CONTEXT     DENY
                     │
                     ▼
                AUTHORIZATION
                     │
          ┌──────────┼──────────┐
          ▼          ▼          ▼
         RBAC       ABAC     OWNERSHIP
          │          │          │
          └──────────┼──────────┘
                     ▼
                 POLICY ENGINE
                     │
                     ▼
                   DECISION

La idea clave es que el tenant se valida antes de que una operación pueda obtener una autorización positiva.

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
Siguiente tarea

TASK-001-13 — Actor Context

Aquí separaremos formalmente Actor, Authentication Context y Security Context, estableciendo cómo una request representa a un USER, SERVICE, SYSTEM o AI_AGENT, cómo se transporta esa identidad por las capas Application/Domain y cómo se evita que un actor pueda ser falsificado o alterado durante una operación.
