E72 — EVOXA API ARCHITECTURE
1. Propósito

E72 define la arquitectura completa de APIs de EVOXA: cómo los clientes, módulos, servicios y sistemas externos interactúan con EVOXA de forma segura, consistente, versionable y observable.

E72 conecta directamente:

E67 Master Application Blueprint
        ↓
E68 Technical Stack
        ↓
E69 Repository Architecture
        ↓
E70 Module Architecture
        ↓
E71 Database Architecture
        ↓
E72 API Architecture
        ↓
E73 Testing Architecture

La API será una frontera arquitectónica, no simplemente una colección de endpoints.

2. Principio Fundamental

La regla principal:

La API expone capacidades de aplicación, no la estructura interna de la base de datos.

Por tanto:

API
 ↓
Application Layer
 ↓
Domain
 ↓
Persistence
 ↓
PostgreSQL

y nunca:

HTTP
 ↓
SQL Table
3. API Ecosystem

EVOXA tendrá varias categorías de interfaces:

                         EVOXA
                           │
        ┌──────────────────┼──────────────────┐
        │                  │                  │
        ▼                  ▼                  ▼
   Public APIs        Internal APIs       Async APIs
        │                  │                  │
        ▼                  ▼                  ▼
    REST/HTTP          Service Calls       Events
        │                  │                  │
        └──────────────────┼──────────────────┘
                           │
                           ▼
                      API Contracts
4. API Types

La arquitectura reconoce:

4.1 External API

Para:

Web Applications
Mobile Apps
Customers
Partners
Integrations
External Systems
4.2 Internal API

Para:

Internal Modules
Platform Services
Workers
Infrastructure Components
4.3 Async API

Para:

Events
Commands
Notifications
Background Processing
Integration
5. API Style

La interfaz síncrona principal será:

REST + HTTP/HTTPS

La comunicación asíncrona:

Events + Messages

No se utilizará una única tecnología para todos los problemas.

6. API Gateway

La entrada externa seguirá:

Client
  ↓
DNS / Edge
  ↓
API Gateway
  ↓
Authentication
  ↓
Authorization
  ↓
Rate Limiting
  ↓
EVOXA API

El Gateway será responsable de concerns transversales.

7. Gateway Responsibilities

El Gateway puede manejar:

TLS termination
Authentication forwarding
Rate limiting
Request size limits
Routing
Correlation IDs
Basic threat protection
API version routing
Observability

Pero no debe contener lógica de negocio.

8. API Application Boundary

Una request llegará a:

Controller
   ↓
Application Command / Query
   ↓
Handler
   ↓
Domain
   ↓
Repository
9. Request Lifecycle
HTTP Request
     ↓
Gateway
     ↓
Authentication
     ↓
Authorization
     ↓
Controller
     ↓
Request Validation
     ↓
Application Handler
     ↓
Domain
     ↓
Transaction
     ↓
Persistence
     ↓
Response Mapping
     ↓
HTTP Response
10. API Modules

Los endpoints seguirán los módulos definidos en E70.

Por ejemplo:

/api/v1/resources
/api/v1/workflows
/api/v1/executions
/api/v1/policies
/api/v1/decisions
/api/v1/actions
/api/v1/data
/api/v1/governance

La estructura final dependerá del modelo funcional, pero el principio será modular.

11. Resource-Oriented API

Cuando sea apropiado:

GET    /api/v1/resources
GET    /api/v1/resources/{id}
POST   /api/v1/resources
PATCH  /api/v1/resources/{id}
DELETE /api/v1/resources/{id}

Pero no todo debe forzarse artificialmente a CRUD.

12. Commands

Para operaciones que representan intención:

POST /api/v1/workflows/{id}/publish
POST /api/v1/workflows/{id}/execute
POST /api/v1/executions/{id}/cancel
POST /api/v1/actions/{id}/retry

Esto expresa mejor el dominio que:

PATCH /executions/{id}
status = "cancelled"
13. Queries

Las consultas estarán separadas conceptualmente:

GET /api/v1/resources
GET /api/v1/resources/{id}
GET /api/v1/executions/{id}
GET /api/v1/workflows/{id}/runs
14. Commands vs Queries

La distinción:

QUERY
↓
Read
↓
No business mutation

y:

COMMAND
↓
Intent
↓
State change

Esto permite evolucionar hacia CQRS sin imponerlo prematuramente.

15. API Versioning

La versión principal será explícita:

/api/v1/...

Ejemplo:

/api/v1/resources
/api/v2/resources

No se utilizará versioning basado exclusivamente en headers para la API principal.

16. Versioning Principle

Una nueva versión solo será necesaria cuando exista un breaking change.

No crear:

v2
v3
v4

por cambios internos que no rompen el contrato.

17. Backward Compatibility

La regla:

Los clientes existentes deben continuar funcionando durante el periodo de soporte de una versión.

Cambios compatibles:

Adding optional field
Adding endpoint
Adding enum value when clients tolerate it

Cambios potencialmente incompatibles:

Removing field
Renaming field
Changing semantics
Changing requiredness
Changing response shape
18. API Contract

Cada API deberá tener contrato formal:

OpenAPI

El contrato define:

Endpoints
Parameters
Request schemas
Response schemas
Errors
Authentication
Examples
19. Contract-First

La arquitectura favorecerá:

API Contract
      ↓
Implementation
      ↓
Tests

en lugar de:

Implementation
      ↓
Generate accidental API
20. Request DTO

No exponer directamente entidades de dominio.

Incorrecto:

HTTP JSON
 ↓
Domain Entity

Preferir:

HTTP JSON
 ↓
Request DTO
 ↓
Command
 ↓
Domain
21. Response DTO

Igualmente:

Domain Entity
 ↓
Response Mapper
 ↓
Response DTO
 ↓
JSON

Esto protege el modelo interno.

22. Resource Representation

Ejemplo conceptual:

{
  "id": "01...",
  "type": "workflow",
  "name": "Example Workflow",
  "status": "active",
  "version": 3,
  "createdAt": "...",
  "updatedAt": "..."
}

La API no necesita reflejar exactamente las columnas SQL.

23. Naming Convention

JSON:

camelCase

Ejemplo:

createdAt
updatedAt
resourceId
workflowVersion

URLs:

kebab-case

cuando corresponda:

/workflow-runs
24. HTTP Methods

Uso estándar:

Method	Semántica
GET	Read
POST	Create / Command
PUT	Full replacement
PATCH	Partial update
DELETE	Delete
25. POST Semantics

POST se utilizará para:

Create
Commands
Actions
Non-idempotent operations

Ejemplo:

POST /api/v1/executions

o:

POST /api/v1/executions/{id}/cancel
26. Idempotency

Las operaciones críticas de creación/ejecución soportarán:

Idempotency-Key

Ejemplo:

POST /api/v1/executions
Idempotency-Key: abc-123

Esto evita duplicar operaciones ante retries.

27. Idempotency Storage

La infraestructura puede almacenar:

idempotency_key
request_hash
response
status
created_at
expires_at

en una estructura persistente apropiada.

28. Retry Safety

Una API debe distinguir:

Safe
Idempotent
Non-idempotent

Ejemplo:

GET       → safe
PUT       → normally idempotent
DELETE    → normally idempotent
POST      → not necessarily idempotent
29. Pagination

Collections grandes nunca devolverán todo el dataset.

Preferencia:

Cursor-based pagination

Ejemplo:

GET /api/v1/executions?limit=50&cursor=...
30. Why Cursor Pagination

Es especialmente apropiada para:

Execution History
Audit Events
Logs
Large Resource Collections

porque evita problemas de offset a gran escala.

31. Offset Pagination

Puede permitirse para:

Small datasets
Administrative UI
Simple reports

cuando sea suficiente.

32. Pagination Response

Ejemplo conceptual:

{
  "items": [],
  "nextCursor": "...",
  "hasMore": true
}
33. Filtering

Los endpoints de collection podrán soportar filtros explícitos:

GET /api/v1/executions?status=failed

No permitir arbitrariamente:

?sql=...

ni expresiones SQL expuestas al cliente.

34. Sorting

Ejemplo:

GET /api/v1/executions?sort=-createdAt

Solo se permitirán campos incluidos en una whitelist.

35. Field Selection

Puede soportarse:

?fields=id,name,status

cuando exista una necesidad real.

No es obligatorio para la primera versión.

36. Search

Search complejo se separará del filtering simple.

GET /api/v1/resources?status=active

vs:

GET /api/v1/search?q=...

Cuando se necesite full-text search, OpenSearch podrá intervenir.

37. API Search Flow
Client
  ↓
Search API
  ↓
Search Service
  ↓
OpenSearch
  ↓
IDs / Documents
  ↓
Optional authoritative lookup
38. Error Architecture

Todos los errores seguirán una estructura consistente.

Ejemplo:

{
  "error": {
    "code": "RESOURCE_NOT_FOUND",
    "message": "Resource not found.",
    "details": [],
    "correlationId": "..."
  }
}
39. Error Codes

Los códigos deben ser estables:

RESOURCE_NOT_FOUND
VALIDATION_FAILED
UNAUTHORIZED
FORBIDDEN
CONFLICT
RATE_LIMITED
INTERNAL_ERROR

No depender exclusivamente del texto del mensaje.

40. HTTP Status Mapping
Status	Meaning
200	Success
201	Created
202	Accepted
204	No Content
400	Bad Request
401	Unauthenticated
403	Forbidden
404	Not Found
409	Conflict
422	Validation/Semantic Error
429	Rate Limited
500	Internal Error
503	Service Unavailable
41. Validation

La validación tendrá capas:

Transport Validation
        ↓
Application Validation
        ↓
Domain Validation
        ↓
Database Constraints
42. Transport Validation

Ejemplo:

required field
string length
format
enum
number range
JSON structure
43. Domain Validation

Ejemplo:

Workflow cannot publish without valid definition.
Execution cannot cancel after terminal state.
Policy cannot activate without required rules.

Estas reglas no pertenecen exclusivamente al controller.

44. Authentication

La API utilizará un mecanismo estándar de identidad:

OAuth 2.0 / OpenID Connect

cuando exista identidad de usuario/aplicación.

45. Access Tokens

El API Gateway / authorization layer validará:

issuer
signature
expiration
audience
scopes
claims
46. Authorization

Authentication:

Who are you?

Authorization:

What are you allowed to do?

Ambas deben mantenerse separadas.

47. Authorization Model

EVOXA podrá combinar:

RBAC
ABAC
Policy-Based Authorization
Resource-Level Authorization

según el módulo.

48. Authorization Flow
Request
  ↓
Identity
  ↓
Role / Claims
  ↓
Policy
  ↓
Resource Context
  ↓
Allow / Deny
49. Tenant Authorization

En escenarios multi-tenant:

Token
 ↓
tenant_id
 ↓
Authorization
 ↓
Resource Ownership

Nunca confiar únicamente en un tenant_id enviado por el cliente.

50. API Rate Limiting

Se aplicará por:

IP
Identity
Client
Tenant
Endpoint

según el caso.

51. Rate Limit Response

Cuando corresponda:

HTTP 429

con información suficiente para que el cliente pueda reintentar correctamente.

52. Request Correlation

Cada request tendrá:

X-Correlation-ID

o equivalente.

Si el cliente no lo proporciona, EVOXA podrá generarlo.

53. Correlation Flow
HTTP Request
   ↓
Correlation ID
   ↓
Application
   ↓
Database
   ↓
Event
   ↓
Worker
   ↓
Logs

Esto permite reconstruir una operación distribuida.

54. Causation ID

Para operaciones derivadas:

causationId

indica qué evento/request originó la acción.

Esto complementa:

correlationId
55. API Observability

Registrar:

Request Count
Latency
Status Codes
Error Rate
Rate Limits
Authentication Failures
Authorization Failures
56. Sensitive Data

Nunca registrar automáticamente:

Passwords
Tokens
Secrets
API Keys
Sensitive Payloads

en logs.

57. Request Logging

Debe registrarse metadata suficiente:

method
path
status
duration
correlationId
client
principal

pero con redacción de datos sensibles.

58. API Security

Protecciones:

TLS
Authentication
Authorization
Rate limiting
Input validation
Payload size limits
Schema validation
Security headers
Audit
59. Request Size

Cada endpoint deberá tener límites razonables:

JSON payload
File upload
Query length
Header size

para evitar abuso.

60. File Upload API

Para archivos grandes:

Client
 ↓
API
 ↓
Pre-signed URL
 ↓
Object Storage

en lugar de enviar siempre el archivo a través de la aplicación.

61. Async Operations

Operaciones largas no bloquearán indefinidamente HTTP.

Ejemplo:

POST /api/v1/executions
        ↓
202 Accepted
        ↓
executionId

El cliente podrá consultar:

GET /api/v1/executions/{id}
62. Async Command Pattern
POST Command
      ↓
202 Accepted
      ↓
Command ID
      ↓
Queue
      ↓
Worker
      ↓
Execution
63. Webhooks

EVOXA podrá exponer eventos mediante webhooks para integraciones externas.

Ejemplo:

workflow.execution.completed
workflow.execution.failed
resource.updated
64. Webhook Security

Los webhooks deberán soportar:

HTTPS
Signature
Timestamp
Replay protection
Retry
Idempotency
65. Webhook Delivery
Event
 ↓
Webhook Dispatcher
 ↓
External Endpoint
 ↓
2xx

Si falla:

Retry
 ↓
Backoff
 ↓
Dead Letter
66. Event API

Los eventos internos tendrán contratos versionados:

eventType
eventVersion
eventId
aggregateId
timestamp
payload
67. Event Envelope

Ejemplo conceptual:

{
  "eventId": "...",
  "eventType": "workflow.execution.completed",
  "eventVersion": 1,
  "aggregateId": "...",
  "occurredAt": "...",
  "correlationId": "...",
  "causationId": "...",
  "payload": {}
}
68. API vs Event

No confundir:

API Request

con:

Domain Event

Una request expresa:

"Haz esto."

Un evento expresa:

"Esto ocurrió."

69. Commands vs Events
Command
"Execute Workflow"

        ↓

Execution

        ↓

Event
"Workflow Execution Started"
70. Internal Module API

Incluso dentro del mismo proceso:

Module A
 ↓
Application Contract
 ↓
Module B

en vez de acceder directamente a:

Module B Repository

Esto mantiene las fronteras.

71. API Contract Between Modules

Puede ser:

Application Interface

en un monolito modular.

Si EVOXA se distribuye posteriormente:

Application Interface
       ↓
HTTP / RPC / Event

sin cambiar necesariamente la semántica del caso de uso.

72. Modular Monolith Compatibility

La arquitectura API está diseñada para que EVOXA pueda comenzar como:

Modular Monolith

y posteriormente extraer módulos:

Module
 ↓
Service

sin rehacer completamente los contratos.

73. Service Extraction

Ejemplo:

EVOXA
├── Workflow Module
├── Execution Module
└── Resource Module

más adelante:

Workflow Service
Execution Service
Resource Service

Los contratos deben sobrevivir a esa transición.

74. API Gateway vs BFF

Para interfaces complejas puede introducirse:

BFF
Backend For Frontend

pero no será obligatorio para todos los clientes.

75. Public API vs Internal API

Separación:

/api/v1/...

para public-facing contracts.

Y:

/internal/...

o mecanismos equivalentes para interfaces internas, cuando sean necesarias.

76. API Documentation

La documentación deberá incluir:

OpenAPI
Examples
Authentication
Errors
Pagination
Rate Limits
Idempotency
Lifecycle
Deprecation
77. API Testing

Cada API tendrá:

Unit Tests
Contract Tests
Integration Tests
Security Tests
End-to-End Tests

E73 detallará la estrategia.

78. Contract Testing

Especialmente importante entre:

EVOXA
 ↕
External Consumers

y:

Producer
 ↕
Consumer

en eventos.

79. API Deprecation

Una API obsoleta debe pasar por:

Active
 ↓
Deprecated
 ↓
Migration Period
 ↓
Sunset
 ↓
Removed

Nunca eliminar silenciosamente una API pública.

80. Deprecation Headers

Cuando sea necesario:

Deprecation
Sunset

podrán utilizarse para comunicar lifecycle.

81. API Compatibility

La compatibilidad debe validarse automáticamente:

OpenAPI
 ↓
Breaking Change Detector
 ↓
CI

Un breaking change no autorizado debe bloquear el build.

82. API Governance

Cada API deberá tener:

Owner
Version
Purpose
Consumers
Security Classification
SLA
Rate Limits
Lifecycle
Documentation
83. API Naming Rules

Consistencia obligatoria:

resources
workflows
workflow-runs
executions
policies
decisions
actions
datasets

Evitar nombres arbitrarios como:

getAllThings
doWorkflow
processDataNow
84. Avoid CRUD Explosion

No crear automáticamente cinco endpoints por cada tabla.

La API representa:

Business Capabilities

no:

Database Tables
85. API Aggregation

Cuando una UI necesite información de varios módulos:

Client
 ↓
API Aggregator / BFF
 ├── Resource
 ├── Workflow
 └── Execution

en vez de forzar al cliente a conocer la arquitectura interna.

86. API Transaction Boundary

Una request puede ejecutar:

Application Use Case

que puede involucrar varias operaciones internas.

Pero la transacción debe permanecer bajo control del Application Layer.

87. API Timeout Strategy

Cada endpoint tendrá timeout apropiado.

Por ejemplo:

Synchronous API
    ↓
Short bounded execution

Long operation
    ↓
Async job

Nunca utilizar HTTP como mecanismo de espera indefinida.

88. API Retry Strategy

El cliente puede reintentar solamente cuando sea seguro.

Especialmente importante:

POST
Commands
External side effects

deben utilizar idempotency cuando exista riesgo de duplicación.

89. External Integration Layer

Integraciones externas deberán pasar por adapters:

EVOXA Application
       ↓
Integration Port
       ↓
External Adapter
       ↓
External API

No contaminar el dominio con SDKs externos.

90. External API Failure

Los fallos externos deben mapearse:

Timeout
Rate Limit
Unavailable
Invalid Response
Authentication Failure

a errores internos controlados.

91. API Resilience

Las integraciones deberán poder utilizar:

Timeouts
Retries
Exponential Backoff
Circuit Breaker
Bulkheads
Fallbacks

cuando corresponda.

92. API Security Boundary

La API debe considerarse zero-trust:

Every Request
      ↓
Authenticate
      ↓
Authorize
      ↓
Validate
      ↓
Execute

Nunca asumir confianza simplemente porque la request proviene de una red interna.

93. API Audit

Las operaciones sensibles deberán producir audit events:

User
 ↓
API
 ↓
Sensitive Action
 ↓
Audit Event

Ejemplos:

Policy changed
Workflow published
Execution cancelled
Access granted
Data exported
94. API Data Privacy

Las respuestas deben aplicar:

Data Minimization
Field Filtering
Authorization
Redaction
Purpose Limitation

No devolver información simplemente porque existe en la DB.

95. API Response Envelope

No envolver obligatoriamente todas las respuestas.

Preferir respuestas naturales:

{
  "id": "...",
  "name": "..."
}

y envelopes solo cuando aporten semántica:

{
  "items": [],
  "nextCursor": "..."
}
96. API Health Endpoints

Separar:

/health/live
/health/ready
Liveness

¿El proceso está vivo?

Readiness

¿Puede recibir tráfico?

97. Metrics Endpoint

La observabilidad interna puede exponer:

/metrics

pero no como API pública.

Debe estar protegida dentro de la infraestructura.

98. API Deployment Model

Inicialmente:

Internet
   ↓
Load Balancer
   ↓
API Gateway
   ↓
EVOXA Application

y:

EVOXA Application
   ↓
PostgreSQL
99. Future Distributed Model

La arquitectura permite evolucionar a:

                    API Gateway
                         │
        ┌────────────────┼────────────────┐
        ▼                ▼                ▼
 Workflow Service   Execution Service   Data Service
        │                │                │
        └────────────────┼────────────────┘
                         ▼
                    Event Platform

sin convertir esta distribución en un requisito prematuro.

100. API Performance

Objetivos iniciales deben definirse por endpoint, pero conceptualmente:

Simple reads
   ↓
Low latency

Complex queries
   ↓
Controlled latency

Long-running operations
   ↓
Async

No utilizar un único SLA para toda la API.

101. API Capacity

Se deberán medir:

Requests/sec
Concurrent requests
Latency p50
Latency p95
Latency p99
Error rate
CPU
Memory
DB connection usage
102. API Rate Policy

Los límites deberán poder diferenciar:

Anonymous
Authenticated User
Application
Tenant
Admin
Internal Service
103. API Quotas

Rate limit:

requests / time

Quota:

total resource consumption

EVOXA podrá soportar ambos.

104. API Governance Matrix

Cada endpoint deberá poder responder:

Who owns it?
Who can call it?
What does it change?
What data does it expose?
Is it synchronous?
Is it idempotent?
What is its SLA?
What is its lifecycle?
105. API Architecture Summary

La arquitectura completa:

                         CLIENTS
                            │
                 ┌──────────┼──────────┐
                 │          │          │
               Web       Mobile     External
                 │          │          │
                 └──────────┼──────────┘
                            ▼
                       API GATEWAY
                            │
             ┌──────────────┼──────────────┐
             │              │              │
       Authentication   Rate Limit    Observability
             │
             ▼
                      EVOXA API
                            │
        ┌───────────────────┼───────────────────┐
        │                   │                   │
     Queries             Commands           Async API
        │                   │                   │
        └───────────────────┼───────────────────┘
                            ▼
                    APPLICATION LAYER
                            │
                         DOMAIN
                            │
                ┌───────────┼───────────┐
                ▼           ▼           ▼
           PostgreSQL     Outbox      External
                │           │          Adapters
                │           ▼
                │        Events
                │
                ├── Redis
                ├── OpenSearch
                └── Object Storage
106. E72 Architectural Decisions
AD-072-01
REST/HTTP is the primary synchronous external API style.

AD-072-02
Async events/messages are used for asynchronous integration.

AD-072-03
APIs expose application capabilities, not database tables.

AD-072-04
API contracts are versioned and documented through OpenAPI.

AD-072-05
Public API versioning uses explicit major versions.

AD-072-06
Commands and queries are conceptually separated.

AD-072-07
Long-running operations use asynchronous execution.

AD-072-08
Critical POST operations support idempotency.

AD-072-09
Cursor pagination is preferred for large collections.

AD-072-10
Authentication and authorization are separate concerns.

AD-072-11
API authorization is policy-driven.

AD-072-12
All requests support correlation tracing.

AD-072-13
Sensitive operations produce audit events.

AD-072-14
Domain entities are never exposed directly as API contracts.

AD-072-15
API contracts must remain independent from persistence schemas.

AD-072-16
Internal module boundaries use application contracts.

AD-072-17
External integrations use adapters.

AD-072-18
Breaking API changes require explicit version/lifecycle management.
107. E71 → E72 → E73

La arquitectura ya queda encadenada:

E71 DATABASE
       │
       ▼
Persistence
       │
       ▼
Repository
       │
       ▼
Application
       │
       ▼
E72 API
       │
       ▼
Contract
       │
       ▼
Client

Y ahora el siguiente capítulo lógico es:

E73 — EVOXA TESTING ARCHITECTURE

Ahí definiremos cómo demostramos que todo lo diseñado hasta E72 realmente funciona:

Unit Testing
Domain Testing
Application Testing
Integration Testing
Database Testing
API Testing
Contract Testing
Event Testing
Security Testing
Performance Testing
Load Testing
Resilience Testing
End-to-End Testing
Test Data Architecture
Test Environments
CI Quality Gates
Coverage Strategy
Mutation Testing

Con E73 terminamos de cerrar el núcleo técnico de desarrollo antes de entrar en las arquitecturas de build, deployment, infraestructura y operación.
