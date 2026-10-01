E03 — EVOXA API Architecture
Architecture & Engineering Specification

Depende de:

A01 — EVOXA Master Architecture
A02 — EVOXA System Architecture
A03 — EVOXA Domain Architecture
A04 — EVOXA Data Architecture
A05 — Security Architecture
A06 — EVOXA API Architecture
A13 — EVOXA Multi-Tenant Architecture
E01 — EVOXA Backend Architecture
E02 — EVOXA Database Architecture

Siguiente: E04 — EVOXA Authentication Architecture

1. Propósito

E03 define la arquitectura de implementación de las APIs de EVOXA.

Su objetivo es convertir la arquitectura API definida en A06 — EVOXA API Architecture en reglas concretas para:

REST APIs
endpoints
recursos
operaciones
DTOs
request/response contracts
versionado
autenticación
autorización
errores
validaciones
paginación
filtros
ordenamiento
búsqueda
idempotencia
webhooks
eventos
OpenAPI
observabilidad
rate limiting
evolución de contratos

La API constituye el contrato público de interacción con EVOXA.

2. Principio fundamental

La arquitectura seguirá:

CLIENT
   ↓
API
   ↓
APPLICATION
   ↓
DOMAIN
   ↓
REPOSITORY
   ↓
DATABASE

Nunca:

CLIENT
   ↓
DATABASE

ni:

CONTROLLER
   ↓
DIRECT DATABASE LOGIC
3. API como Contract Boundary

La API debe actuar como frontera entre EVOXA y sus consumidores:

                    EVOXA API
                       │
       ┌───────────────┼────────────────┐
       │               │                │
    Web App         Mobile App       External
       │               │             Integrations
       └───────────────┼────────────────┘
                       │
                  Application
                       │
                     Domain

La API no debe exponer directamente:

modelos ORM
tablas
estructuras internas
secretos
detalles de infraestructura
excepciones internas
4. API Style

El estilo principal será:

RESTful HTTP API
+
JSON
+
OpenAPI

La API debe ser:

Predictable
Consistent
Versioned
Discoverable
Secure
Observable
Idempotent where required
5. API Base URL

La estructura lógica será:

/api/v1

Ejemplo:

/api/v1/auth
/api/v1/users
/api/v1/tenants
/api/v1/projects
/api/v1/roadmaps
/api/v1/tasks
6. API Versioning

El versionado inicial será mediante URL:

/api/v1

Evolución:

/api/v1
/api/v2

No se debe crear una nueva versión por cada cambio menor.

7. Versioning Rules
Compatible

Puede permanecer en v1:

ADD optional field
ADD endpoint
ADD filter
ADD optional query parameter
Breaking

Debe considerar nueva versión:

REMOVE field
RENAME field
CHANGE field semantics
CHANGE response structure
CHANGE authentication behavior
CHANGE required parameters
8. Resource-Oriented API

Los endpoints deben representar recursos.

Correcto:

GET /api/v1/projects
GET /api/v1/projects/{projectId}
POST /api/v1/projects
PATCH /api/v1/projects/{projectId}
DELETE /api/v1/projects/{projectId}

Evitar:

GET /api/v1/getProjects
POST /api/v1/createProject
POST /api/v1/deleteProject
9. HTTP Methods

Reglas:

GET
→ Read

POST
→ Create / Action

PUT
→ Full replacement

PATCH
→ Partial update

DELETE
→ Delete
10. POST

Crear recursos:

POST /api/v1/projects

Request:

{
  "name": "EVOXA Platform",
  "description": "Main platform project"
}

Response:

201 Created
11. GET Collection
GET /api/v1/projects

Respuesta:

{
  "data": [],
  "pagination": {
    "nextCursor": null,
    "hasMore": false
  }
}
12. GET Resource
GET /api/v1/projects/{projectId}

Respuesta:

{
  "data": {
    "id": "project-id",
    "name": "EVOXA Platform",
    "status": "ACTIVE"
  }
}
13. PATCH

Para modificaciones parciales:

PATCH /api/v1/projects/{projectId}

Request:

{
  "name": "EVOXA Enterprise Platform"
}
14. DELETE
DELETE /api/v1/projects/{projectId}

Cuando el recurso utilice soft delete:

DELETE
↓
deleted_at

No necesariamente implica eliminación física.

15. Actions

Algunas operaciones representan una acción de dominio.

Ejemplo:

POST /api/v1/projects/{projectId}/archive

o:

POST /api/v1/agents/{agentId}/execute

Esto es aceptable cuando la operación no representa simplemente CRUD.

16. API Resource Hierarchy

La estructura principal:

Tenant
 ├── Applications
 │      └── Projects
 │             └── Roadmaps
 │                    ├── Objectives
 │                    │      └── Initiatives
 │                    │             └── Tasks
 │                    └── Milestones
 │
 ├── Users
 ├── Roles
 └── Integrations
17. Tenant Context

Toda request debe resolver:

User
 ↓
Authentication
 ↓
Tenant Context
 ↓
Authorization
 ↓
Resource

Ejemplo:

GET /api/v1/projects

no debe devolver proyectos de todos los tenants.

Debe operar sobre:

currentTenant
18. Tenant Selection

El tenant puede determinarse mediante:

JWT
+
membership
+
request context

El mecanismo concreto se definirá en:

E04 — Authentication Architecture

y:

E07 — Multi-Tenant Engineering.

19. Authorization

La autorización debe ocurrir antes de acceder al recurso.

Request
 ↓
Authentication
 ↓
Tenant Resolution
 ↓
Authorization
 ↓
Use Case

Ejemplo:

roadmap.read
roadmap.update
roadmap.delete
20. DTO Architecture

Nunca exponer directamente una entidad de dominio.

HTTP Request
      ↓
Request DTO
      ↓
Use Case
      ↓
Domain
      ↓
Response DTO
      ↓
HTTP Response
21. Request DTO

Ejemplo:

CreateProjectRequest {
  name: string;
  description?: string;
}

El DTO representa el contrato externo.

22. Response DTO
ProjectResponse {
  id: string;
  name: string;
  description?: string;
  status: string;
  createdAt: string;
  updatedAt: string;
}

No debe devolver:

password_hash
internal_flags
database_metadata
23. DTO Mapping
Request DTO
    ↓
Command
    ↓
Use Case
    ↓
Domain Entity
    ↓
Response Mapper
    ↓
Response DTO
24. Validation

La validación ocurre en múltiples niveles:

HTTP Schema
 ↓
Application Validation
 ↓
Domain Validation
 ↓
Database Constraints

Ejemplo:

name required
name max length
slug format
date format
enum validity
25. Validation Errors

Respuesta estándar:

{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Request validation failed",
    "details": [
      {
        "field": "name",
        "code": "REQUIRED"
      }
    ]
  }
}
26. Error Contract

Todos los errores deben seguir una estructura consistente.

{
  "error": {
    "code": "RESOURCE_NOT_FOUND",
    "message": "Project not found",
    "requestId": "req_123"
  }
}
27. Error Codes

Los códigos deben ser estables.

Ejemplos:

VALIDATION_ERROR
UNAUTHENTICATED
FORBIDDEN
RESOURCE_NOT_FOUND
RESOURCE_CONFLICT
DUPLICATE_RESOURCE
RATE_LIMITED
INTERNAL_ERROR
SERVICE_UNAVAILABLE

El cliente no debe depender del texto de message.

28. HTTP Status Codes

Reglas principales:

200 OK
201 Created
202 Accepted
204 No Content

400 Bad Request
401 Unauthorized
403 Forbidden
404 Not Found
409 Conflict
422 Unprocessable Entity
429 Too Many Requests

500 Internal Server Error
502 Bad Gateway
503 Service Unavailable
29. 401 vs 403
401

El usuario no está autenticado.

No valid credentials
403

El usuario está autenticado pero no tiene autorización.

Authenticated
+
Insufficient permissions
30. 404

No revelar información innecesaria sobre recursos protegidos.

Dependiendo del contexto, una operación sobre un recurso de otro tenant puede responder:

404 Not Found

en lugar de revelar que el recurso existe.

31. Conflict

Utilizar 409 para conflictos de estado o unicidad.

Ejemplo:

POST /projects

si ya existe:

tenant + slug

Respuesta:

409 Conflict
32. Pagination

Todas las colecciones potencialmente grandes deben paginarse.

Preferencia:

cursor pagination

Ejemplo:

GET /api/v1/projects?limit=25&cursor=abc
33. Pagination Response
{
  "data": [],
  "pagination": {
    "limit": 25,
    "nextCursor": "xyz",
    "hasMore": true
  }
}
34. Pagination Limits

Debe existir:

default limit
maximum limit

Ejemplo conceptual:

default = 25
maximum = 100

Los valores definitivos se establecerán en la implementación.

35. Filtering

Ejemplo:

GET /api/v1/projects?status=ACTIVE

Filtros deben estar definidos por recurso.

No aceptar filtros arbitrarios directamente contra columnas SQL.

36. Sorting

Ejemplo:

GET /api/v1/projects?sort=-createdAt

La API debe utilizar una whitelist:

createdAt
name
updatedAt
status

No:

sort=password_hash
37. Search

Ejemplo:

GET /api/v1/projects?search=evoxa

La búsqueda debe utilizar mecanismos controlados.

No generar SQL dinámico sin validación.

38. Field Selection

Puede soportarse posteriormente:

GET /projects?fields=id,name,status

pero solamente cuando exista una necesidad real.

No debe complicar prematuramente la API.

39. Expand / Include

Para relaciones:

GET /projects/{id}?include=roadmap

Debe existir una whitelist de relaciones.

No permitir:

include=*

sin control.

40. Nested Resources

Cuando la relación sea fuerte:

GET /roadmaps/{roadmapId}/objectives

Puede ser útil.

Pero evitar URLs excesivamente profundas:

/tenants/{id}/projects/{id}/roadmaps/{id}/objectives/{id}/initiatives/{id}/tasks

Preferir endpoints simples cuando sea posible.

41. API Idempotency

Operaciones sensibles pueden utilizar:

Idempotency-Key

Especialmente:

payments
billing
external integrations
agent execution
critical commands

Ejemplo:

Idempotency-Key: 8f7c...
42. Idempotency Flow
REQUEST
 ↓
Idempotency Key
 ↓
Already processed?
 ├── YES → Return previous result
 └── NO
       ↓
    Execute
       ↓
    Store result
43. Request ID

Cada request debe tener:

X-Request-ID

o un identificador generado por el gateway.

Ejemplo:

req_01H...

Debe propagarse a:

Logs
Events
Database Audit
Services
Agent Executions
44. Correlation ID

Para operaciones distribuidas:

X-Correlation-ID

Ejemplo:

HTTP Request
 ↓
Use Case
 ↓
Event
 ↓
Worker
 ↓
External Service

Todos deben poder relacionarse.

45. API Observability

Registrar:

request_id
correlation_id
tenant_id
user_id
method
path
status
duration

Nunca registrar secretos.

46. Sensitive Logging

No registrar:

password
access_token
refresh_token
api_key
client_secret
payment credentials

Los payloads deben filtrarse cuando contengan información sensible.

47. Rate Limiting

La API debe implementar límites por:

IP
User
Tenant
Endpoint
API Key

según el contexto.

48. Rate Limit Response

Cuando se supera el límite:

429 Too Many Requests

La respuesta puede incluir:

Retry-After
49. Authentication Headers

La API utilizará:

Authorization: Bearer <access_token>

para endpoints protegidos.

La arquitectura exacta de tokens se definirá en E04.

50. Public vs Protected API
PUBLIC
 ├── health
 ├── authentication
 ├── registration
 └── public metadata

PROTECTED
 ├── users
 ├── tenants
 ├── projects
 ├── roadmaps
 ├── AI
 └── agents

Cada endpoint debe declarar explícitamente su requisito de autenticación.

51. Health Endpoints

Debe existir:

GET /health

y:

GET /health/ready
GET /health/live

Conceptualmente:

liveness
→ process alive

readiness
→ process capable of serving traffic
52. API Documentation

OpenAPI será el contrato técnico.

Debe documentar:

Endpoints
Parameters
Request Bodies
Responses
Authentication
Errors
Schemas
Examples
53. OpenAPI

La API debe generar:

openapi.json

y documentación interactiva:

Swagger UI

además de documentación alternativa compatible con:

Redoc
54. Schema Organization

Los schemas deben organizarse por dominio:

schemas/
├── auth
├── users
├── tenants
├── projects
├── roadmaps
├── engineering
├── operations
├── ai
└── agents
55. API Module Structure

Una estructura inicial:

src/
└── modules/
    ├── auth/
    │   ├── controller/
    │   ├── dto/
    │   ├── application/
    │   └── domain/
    │
    ├── projects/
    ├── roadmaps/
    ├── engineering/
    ├── operations/
    ├── ai/
    └── agents/
56. Controller Responsibility

Controller:

HTTP
 ↓
Parse
 ↓
Validate
 ↓
Authorize
 ↓
Call Use Case
 ↓
Map Response

Controller no debe contener:

Business Logic
SQL
Complex Domain Rules
57. Application Layer

Ejemplo:

CreateProjectUseCase
UpdateProjectUseCase
DeleteProjectUseCase
GetProjectUseCase
ListProjectsUseCase
58. Domain Layer

Contiene:

Entities
Value Objects
Domain Services
Business Rules
Domain Events

No contiene:

HTTP
Express
Fastify
Sequelize
Redis
59. API → Application
POST /projects
      ↓
CreateProjectRequest
      ↓
CreateProjectCommand
      ↓
CreateProjectUseCase
      ↓
Project
60. API → Repository

No debe existir:

Controller
 ↓
ProjectModel.findAll()

Debe existir:

Controller
 ↓
UseCase
 ↓
Repository
 ↓
Sequelize
61. Response Envelope

La estrategia será consistente:

{
  "data": {}
}

Para listas:

{
  "data": [],
  "pagination": {}
}

Para errores:

{
  "error": {}
}
62. Metadata

Puede utilizarse:

{
  "data": {},
  "meta": {}
}

para información adicional.

No incluir metadata innecesaria.

63. Dates

Las APIs deben utilizar formato ISO 8601.

Ejemplo:

2026-09-24T15:30:00Z

No:

24/09/2026 15:30
64. Null vs Missing

Debe definirse claramente:

{
  "description": null
}

significa distinto de:

{}

Especialmente en PATCH.

65. PATCH Semantics

Un campo ausente:

no modificar

Un campo:

"description": null

puede significar:

clear value

según el contrato.

66. API Contract Stability

Una vez publicada una API:

Consumer
   ↓
Contract

Cambios deben evaluarse por compatibilidad.

La estabilidad del contrato es una responsabilidad arquitectónica.

67. Deprecation

Cuando un endpoint sea reemplazado:

ACTIVE
 ↓
DEPRECATED
 ↓
SUNSET
 ↓
REMOVED

La API debe documentar:

Deprecation Date
Replacement
Removal Date
68. API Gateway

La arquitectura puede incorporar:

Client
 ↓
API Gateway
 ↓
EVOXA Backend

El gateway puede encargarse de:

TLS
Rate limiting
Routing
Authentication edge
Request size
WAF
Observability

pero no debe absorber lógica de dominio.

69. CORS

La configuración CORS debe ser explícita.

No:

Access-Control-Allow-Origin: *

para APIs autenticadas de producción.

Debe utilizar una whitelist de orígenes autorizados.

70. Request Size

La API debe establecer límites para:

JSON payload
multipart upload
file upload
headers

Esto reduce riesgos de abuso.

71. File Upload API

Los archivos no deben viajar siempre a través del backend.

Arquitectura preferida:

Client
 ↓
Request Upload URL
 ↓
EVOXA
 ↓
Signed URL
 ↓
Object Storage

Posteriormente:

Object Storage
 ↓
File Metadata
 ↓
EVOXA Database
72. Webhooks

EVOXA puede exponer:

POST /webhooks/{provider}

para integraciones externas.

Los webhooks deben tener:

Signature
Timestamp
Event ID
Replay Protection
73. Outbound Webhooks

EVOXA también puede enviar eventos:

Tenant
 ↓
Webhook Subscription
 ↓
Event
 ↓
Webhook Delivery
 ↓
External System
74. Webhook Delivery

Debe registrar:

webhook_deliveries
├── id
├── subscription_id
├── event_id
├── attempt
├── status
├── response_code
├── delivered_at
└── next_retry_at
75. Webhook Retry

Estrategia:

Attempt 1
 ↓
Failure
 ↓
Retry
 ↓
Exponential Backoff
 ↓
Max Attempts
 ↓
Dead Letter
76. API Events

Una mutación puede generar:

ProjectCreated
ProjectUpdated
ProjectArchived
RoadmapCreated
TaskCompleted

La API no debe depender de que el consumidor consulte constantemente.

77. Async Operations

Operaciones largas pueden responder:

202 Accepted

Ejemplo:

POST /api/v1/agents/{id}/execute

Response:

{
  "data": {
    "executionId": "exec_123",
    "status": "QUEUED"
  }
}
78. Async Status
GET /api/v1/agent-executions/{executionId}

Estados:

QUEUED
RUNNING
WAITING
COMPLETED
FAILED
CANCELLED
79. API Cancellation

Operaciones largas pueden soportar:

POST /agent-executions/{id}/cancel

La cancelación debe ser una operación de dominio, no simplemente eliminar un registro.

80. API for AI

Ejemplo:

POST /api/v1/ai/generate
POST /api/v1/ai/chat
GET  /api/v1/ai/requests/{id}

Debe registrar:

model
provider
usage
latency
cost
tenant
user
81. API for Agents
GET  /api/v1/agents
POST /api/v1/agents
GET  /api/v1/agents/{id}
PATCH /api/v1/agents/{id}
POST /api/v1/agents/{id}/execute
POST /api/v1/agents/{id}/pause
POST /api/v1/agents/{id}/resume
82. Agent Authorization

Un Agent nunca debe heredar automáticamente todos los permisos del usuario.

Debe existir:

User Permissions
      ↓
Agent Permissions
      ↓
Allowed Capabilities
      ↓
Execution
83. API for Roadmaps
GET    /api/v1/roadmaps
POST   /api/v1/roadmaps
GET    /api/v1/roadmaps/{id}
PATCH  /api/v1/roadmaps/{id}
DELETE /api/v1/roadmaps/{id}

Subresources:

GET  /roadmaps/{id}/objectives
POST /roadmaps/{id}/objectives
84. API for Objectives
GET    /api/v1/objectives/{id}
POST   /api/v1/objectives
PATCH  /api/v1/objectives/{id}
DELETE /api/v1/objectives/{id}
85. API for Initiatives
GET    /api/v1/initiatives
POST   /api/v1/initiatives
GET    /api/v1/initiatives/{id}
PATCH  /api/v1/initiatives/{id}
DELETE /api/v1/initiatives/{id}
86. API for Tasks
GET    /api/v1/tasks
POST   /api/v1/tasks
GET    /api/v1/tasks/{id}
PATCH  /api/v1/tasks/{id}
DELETE /api/v1/tasks/{id}

Actions:

POST /tasks/{id}/complete
POST /tasks/{id}/reopen
87. Bulk Operations

Cuando sea necesario:

POST /api/v1/tasks/bulk-update

Debe existir límite de tamaño.

Las operaciones bulk deben mantener:

Authorization
Validation
Audit
Tenant Isolation
88. Import APIs

Para grandes cantidades:

POST /api/v1/imports
GET  /api/v1/imports/{id}

Flujo:

Upload
 ↓
Validate
 ↓
Process
 ↓
Import
 ↓
Report
89. Export APIs
POST /api/v1/exports
GET  /api/v1/exports/{id}

Las exportaciones grandes deben ser asíncronas.

90. API Security Layers
TLS
 ↓
Gateway / WAF
 ↓
Rate Limiting
 ↓
Authentication
 ↓
Tenant Resolution
 ↓
Authorization
 ↓
Validation
 ↓
Domain
 ↓
Database
91. API Threat Model

La API debe proteger contra:

Broken Authentication
Broken Authorization
IDOR
Injection
Mass Assignment
Rate Abuse
Replay
Credential Theft
Data Leakage
Improper Error Handling
92. IDOR Protection

Nunca confiar solamente en:

/project/123

Debe validarse:

resource
+
tenant
+
authorization
93. Mass Assignment

No aceptar objetos completos directamente:

{
  "name": "...",
  "status": "ACTIVE",
  "role": "SUPER_ADMIN"
}

si role no pertenece al contrato de actualización.

DTOs deben controlar exactamente qué campos son modificables.

94. API Security Headers

La infraestructura debe considerar:

Strict-Transport-Security
Content-Security-Policy
X-Content-Type-Options
X-Frame-Options
Referrer-Policy

según el componente y contexto.

95. API Performance

Objetivos:

Low latency
Predictable latency
Controlled payloads
Efficient queries
Caching where appropriate
Async processing for expensive operations

No convertir la API en una capa que simplemente expone consultas SQL.

96. N+1 Protection

Las APIs que cargan relaciones deben evitar:

1 query
+
N queries

Debe utilizarse:

JOIN
Batching
Preloading
Read Models

cuando corresponda.

97. API Metrics

Métricas mínimas:

requests_total
request_duration
errors_total
5xx_total
4xx_total
rate_limited_total
active_requests

Por:

endpoint
method
status
tenant

cuando sea seguro y útil.

98. API Tracing

Tracing:

Client
 ↓
Gateway
 ↓
API
 ↓
Use Case
 ↓
Database
 ↓
Event
 ↓
Worker

Todos deben compartir:

trace_id

cuando exista infraestructura distribuida.

99. API Testing

Cada endpoint debe cubrir:

Happy Path
Validation
Authentication
Authorization
Tenant Isolation
Not Found
Conflict
Concurrency
Rate Limit
Failure
100. Contract Testing

Debe existir validación de:

Request Schema
Response Schema
Status Codes
Error Contract
Headers
Authentication

Esto permite detectar breaking changes.

101. Integration Testing

Flujo:

HTTP
 ↓
Controller
 ↓
Application
 ↓
Domain
 ↓
Repository
 ↓
Database

Debe existir una suite de integración que pruebe el comportamiento real.

102. API Documentation Lifecycle
Design
 ↓
OpenAPI
 ↓
Implementation
 ↓
Validation
 ↓
Testing
 ↓
Release

La documentación no debe escribirse después de implementar como tarea secundaria.

103. API Governance

Cada nuevo endpoint debe responder:

Why does it exist?
Who consumes it?
Which domain owns it?
What permissions are required?
What data does it expose?
Is it synchronous?
Is it idempotent?
How is it audited?
How is it versioned?
104. API Design Review

Antes de publicar una API:

✓ Resource naming
✓ HTTP semantics
✓ DTO design
✓ Validation
✓ Authorization
✓ Tenant isolation
✓ Error contract
✓ Pagination
✓ Idempotency
✓ Audit
✓ Observability
✓ OpenAPI
✓ Tests
105. API Evolution

La evolución seguirá:

Proposal
 ↓
API Design
 ↓
OpenAPI Contract
 ↓
Implementation
 ↓
Testing
 ↓
Documentation
 ↓
Release
 ↓
Monitoring
 ↓
Deprecation
106. API Architecture Final
                         EVOXA API
                            │
              ┌─────────────┼─────────────┐
              │             │             │
           REST API      Webhooks       Async API
              │             │             │
              └─────────────┼─────────────┘
                            │
                      API Gateway
                            │
                    Authentication
                            │
                      Tenant Context
                            │
                      Authorization
                            │
                       Validation
                            │
                       Controllers
                            │
                      Application
                            │
                         Domain
                            │
                       Repository
                            │
                        Database
107. API Contract Flow
Client
  ↓
HTTP Request
  ↓
API Router
  ↓
Authentication
  ↓
Tenant Context
  ↓
Authorization
  ↓
DTO Validation
  ↓
Use Case
  ↓
Domain
  ↓
Repository
  ↓
Database
  ↓
Domain Event
  ↓
Response Mapper
  ↓
HTTP Response
108. Definition of Done

E03 se considera definido cuando:

✓ REST strategy defined
✓ URL versioning defined
✓ Resource naming defined
✓ HTTP semantics defined
✓ DTO strategy defined
✓ Validation defined
✓ Error contract defined
✓ Status codes defined
✓ Pagination defined
✓ Filtering defined
✓ Sorting defined
✓ Search defined
✓ Idempotency defined
✓ Request IDs defined
✓ Correlation IDs defined
✓ Authentication boundary defined
✓ Authorization boundary defined
✓ Tenant isolation defined
✓ Rate limiting defined
✓ Webhooks defined
✓ Async operations defined
✓ OpenAPI defined
✓ API testing defined
✓ Contract testing defined
✓ Observability defined
✓ Deprecation defined
✓ API governance defined
109. Principios oficiales E03
01 — API Is a Contract
02 — APIs Are Resource-Oriented
03 — Contracts Must Be Explicit
04 — Version Before Breaking
05 — Validate at the Boundary
06 — Never Expose Persistence Models
07 — Authorization Before Data Access
08 — Tenant Isolation Is Mandatory
09 — Errors Must Be Consistent
10 — Clients Must Not Depend on Error Text
11 — Large Collections Must Be Paginated
12 — Critical Mutations Must Support Idempotency
13 — Long Operations Should Be Asynchronous
14 — Every Request Must Be Observable
15 — Secrets Must Never Be Logged
16 — Webhooks Must Be Verifiable
17 — APIs Must Be Documented With OpenAPI
18 — APIs Must Be Contract Tested
19 — Deprecated APIs Must Have a Migration Path
20 — API Evolution Must Be Governed
110. Relación E02 → E03

La cadena técnica queda:

E01 — Backend Architecture
        │
        ▼
E02 — Database Architecture
        │
        ▼
E03 — API Architecture
        │
        ├── HTTP
        ├── REST
        ├── DTO
        ├── Validation
        ├── Auth
        ├── Authorization
        ├── Tenant Context
        ├── Events
        ├── Webhooks
        └── OpenAPI

Y desde aquí la siguiente capa natural es:

E04 — EVOXA Authentication Architecture

donde definiremos en detalle JWT/access tokens, refresh tokens, sesiones, MFA/2FA, password security, login, logout, token rotation, revocation, OAuth/OIDC, device sessions, recovery, email verification y el ciclo completo de identidad/autenticación de EVOXA.
