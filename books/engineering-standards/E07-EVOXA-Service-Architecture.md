E07 — EVOXA Service Architecture
Engineering Specification

Pertenece a: Engineering Specification
Volumen: volume-09-engineering
Capítulo: E07 — EVOXA Service Architecture
Anterior: E06 — Policy Architecture
Siguiente: E08 — EVOXA Domain Services Architecture

1. Propósito

E07 define la arquitectura de Services de EVOXA.

Mientras los capítulos anteriores establecen:

E01 — Backend Architecture: estructura general del backend.
E02 — Database Architecture: persistencia y datos.
E03 — API Architecture: exposición de capacidades.
E04 — Authentication Architecture: identidad y autenticación.
E05 — Authorization Architecture: permisos y autorización.
E06 — Policy Architecture: reglas y decisiones.

E07 define cómo se organizan, comunican, ejecutan y evolucionan los servicios de EVOXA.

La pregunta central es:

¿Cómo se estructuran los servicios de software que implementan las capacidades de EVOXA?

2. Objetivos

La Service Architecture debe proporcionar:

Service Boundaries
Service Responsibilities
Service Interfaces
Service Dependencies
Service Communication
Service Lifecycle
Service Discovery
Service Configuration
Service Resilience
Service Security
Service Observability
Service Scaling
Service Deployment
Service Governance
3. Principio Fundamental

Un Service representa una unidad coherente de comportamiento del sistema, no simplemente una colección arbitraria de endpoints.

API
 │
 ▼
Service
 │
 ├── Business Logic
 ├── Domain Logic
 ├── Policies
 ├── Data Access
 └── Integrations
4. Service Architecture Model
                         EVOXA
                           │
              ┌────────────┼────────────┐
              │            │            │
          API Layer    Event Layer   Jobs Layer
              │            │            │
              └────────────┼────────────┘
                           ▼
                       Services
                           │
            ┌──────────────┼──────────────┐
            │              │              │
         Domain         Platform       Supporting
        Services        Services        Services
            │              │              │
            └──────────────┼──────────────┘
                           ▼
                     Infrastructure
5. Service Definition

Cada service debe tener:

serviceId
name
description
domain
responsibilities
interfaces
dependencies
owner
version
status

Ejemplo:

serviceId:
training-service

domain:
Training

responsibility:
Manage training plans, workouts and exercises.
6. Service Boundary

Cada service debe tener una responsabilidad claramente delimitada.

Incorrecto:

UserTrainingBillingService

Correcto:

Identity Service
Training Service
Billing Service

Las relaciones entre dominios deben establecerse mediante contratos.

7. Service Ownership

Cada service debe tener un propietario técnico y funcional:

Service
 │
 ├── Technical Owner
 ├── Domain Owner
 └── Security Owner

Esto permite determinar quién responde por cambios, incidentes y evolución.

8. Service Types

EVOXA debe distinguir diferentes tipos de servicios.

Platform Services
Domain Services
Application Services
Integration Services
Infrastructure Services
AI Services
Supporting Services
9. Platform Services

Servicios utilizados transversalmente.

Ejemplos:

Identity Service
Authentication Service
Authorization Service
Policy Service
Tenant Service
Configuration Service
Notification Service
Audit Service
10. Domain Services

Implementan capacidades específicas del negocio.

Ejemplos:

Training Service
Nutrition Service
Progress Service
Coaching Service
Exercise Service
Client Service
11. Application Services

Coordinan casos de uso.

Application
    ↓
Application Service
    ↓
Domain Services
    ↓
Infrastructure

Su responsabilidad principal es orquestar, no convertirse en repositorios gigantes de lógica.

12. Integration Services

Gestionan integraciones externas:

Payment Integration
Email Integration
Storage Integration
AI Provider Integration
Messaging Integration
Identity Provider Integration

La integración externa no debe contaminar directamente los servicios de dominio.

13. Infrastructure Services

Ejemplos:

Storage Service
Queue Service
Cache Service
Search Service
Scheduler Service
14. AI Services

EVOXA tendrá servicios especializados para:

AI Inference
AI Orchestration
Agent Execution
AI Safety
AI Memory
AI Tool Execution
AI Evaluation

Estos deberán respetar E06 Policy Architecture.

15. Service Layering

Arquitectura recomendada:

┌─────────────────────────────┐
│        API / Interface      │
├─────────────────────────────┤
│      Application Layer      │
├─────────────────────────────┤
│        Domain Layer         │
├─────────────────────────────┤
│      Service Interfaces     │
├─────────────────────────────┤
│     Infrastructure Layer    │
└─────────────────────────────┘
16. API Layer

La API debe:

Validate Request
Authenticate
Authorize
Invoke Application Service
Serialize Response

No debería contener lógica empresarial compleja.

17. Application Service

Ejemplo:

CreateWorkoutService

Puede coordinar:

Authorization
Policy Evaluation
Workout Domain
Exercise Service
Event Publishing
Audit
18. Domain Service

El Domain Service implementa reglas propias del dominio.

Ejemplo:

WorkoutPlanningService

Puede determinar:

Training Volume
Exercise Ordering
Workout Constraints
Progression Rules
19. Infrastructure Service

Se encarga de detalles técnicos:

Database
Redis
Object Storage
Queues
External APIs

Los dominios no deberían depender directamente de estos detalles.

20. Dependency Direction

Regla:

Interface
    ↓
Application
    ↓
Domain
    ↓
Abstractions
    ↓
Infrastructure

Infrastructure implementa interfaces requeridas por capas superiores.

21. Dependency Inversion

Ejemplo:

Training Service
      │
      ▼
WorkoutRepository
      ▲
      │
PostgresWorkoutRepository

El dominio no necesita conocer PostgreSQL.

22. Service Interfaces

Cada service debe definir contratos claros.

Ejemplo:

WorkoutService

createWorkout()
getWorkout()
updateWorkout()
deleteWorkout()

Pero el contrato real debe estar definido por casos de uso y API/event contracts, no por una simple lista de métodos.

23. Service Contracts

Cada service debe especificar:

Inputs
Outputs
Errors
Authorization
Policies
Events
Dependencies
Consistency
Idempotency
24. Synchronous Communication

Para operaciones que necesitan respuesta inmediata:

Client
  ↓
API
  ↓
Service A
  ↓
Service B
  ↓
Response

Ejemplos:

Get User
Get Workout
Validate Permission
Calculate Current Progress
25. Asynchronous Communication

Para procesos que no requieren respuesta inmediata:

Service A
    ↓
Event
    ↓
Message Broker
    ↓
Service B

Ejemplos:

UserRegistered
WorkoutCompleted
PaymentConfirmed
SubscriptionChanged
26. Synchronous vs Asynchronous

Regla general:

Immediate Result
    → Synchronous

Background / Decoupled
    → Asynchronous

No convertir automáticamente todo en eventos.

27. Service-to-Service Communication

La comunicación interna debe utilizar contratos explícitos:

REST
gRPC
Events
Commands
Message Queues

La tecnología concreta debe depender del caso de uso.

28. Internal APIs

Las APIs internas deben diferenciarse de APIs públicas.

Public API
Internal API
Administrative API
Service API

Cada una debe tener controles de acceso apropiados.

29. Service Discovery

En ambientes distribuidos, los servicios deben poder localizarse sin hardcodear direcciones.

Service Registry
       │
 ┌─────┼─────┐
 ▼     ▼     ▼
Auth  User  Training

En una arquitectura inicial monolítica modular esto puede simplificarse.

30. Modular Monolith First

EVOXA no debe fragmentarse artificialmente en microservicios desde el inicio.

Arquitectura inicial recomendada:

EVOXA Backend
 │
 ├── Identity Module
 ├── Training Module
 ├── Nutrition Module
 ├── Billing Module
 ├── AI Module
 └── Security Module

con boundaries preparados para futura extracción.

31. Evolution Toward Services

La evolución puede ser:

Modular Monolith
       ↓
Well-defined Modules
       ↓
Service Boundaries
       ↓
Selective Extraction
       ↓
Distributed Services

No:

Monolith
 ↓
50 Microservices immediately
32. Service Extraction Criteria

Un módulo puede convertirse en servicio cuando exista:

High Scale
Independent Deployment
Independent Ownership
Strong Boundary
Different Availability Requirements
Different Security Boundary
Different Scaling Pattern
33. Service Independence

Un service debería poder evolucionar independientemente dentro de los límites arquitectónicos.

Debe evitarse:

Service A
   ↓
Service B
   ↓
Service C
   ↓
Service D

para una operación simple.

34. Distributed Monolith

Anti-pattern:

Service A
 └── synchronous call B

Service B
 └── synchronous call C

Service C
 └── synchronous call D

Si todos dependen entre sí, se obtiene un monolito distribuido.

35. Service Dependency Graph

EVOXA debe mantener un mapa de dependencias:

Identity
   ↓
Authorization
   ↓
Training
   ↓
Progress

y:

Billing
   ↓
Entitlements
   ↓
AI
36. Dependency Rules

Las dependencias deben ser:

Explicit
Documented
Observable
Versioned
Justified
37. Circular Dependencies

No deben existir:

Service A → Service B
Service B → Service A

La lógica común debe extraerse a una abstracción adecuada o rediseñarse el boundary.

38. Shared Libraries

Se permiten librerías compartidas para:

Logging
Tracing
Validation
Security primitives
Common contracts

Pero no deben convertirse en un mecanismo para compartir toda la lógica de negocio.

39. Shared Database

Los servicios deben evitar compartir tablas directamente cuando evolucionen hacia arquitectura distribuida.

Preferencia:

Service A
  ↓
Owns Data A

Service B
  ↓
Owns Data B

La integración debe producirse mediante APIs o eventos.

40. Data Ownership

Cada service debe saber qué datos posee:

Training Service
 → Workouts
 → Training Plans

Nutrition Service
 → Nutrition Plans
 → Meals

Billing Service
 → Subscriptions
 → Invoices
41. Transaction Boundaries

Cada service debe definir sus límites transaccionales.

Una transacción local debe preferirse sobre una transacción distribuida.

42. Distributed Transactions

Evitar:

Service A
 +
Service B
 +
Service C

dentro de una única transacción ACID distribuida salvo necesidad excepcional.

Preferir:

Local Transaction
      ↓
Event
      ↓
Compensation
43. Saga Pattern

Para procesos largos:

Order
 ↓
Payment
 ↓
Provisioning
 ↓
Notification

puede utilizarse:

Saga

con compensaciones.

44. Idempotency

Los servicios que reciben comandos/eventos deben soportar idempotencia cuando sea necesario.

Ejemplo:

PaymentConfirmed

procesado dos veces:

First → Apply
Second → Ignore
45. Service Commands

Los comandos representan intención:

CreateWorkout
CompleteWorkout
CancelSubscription
GenerateNutritionPlan
46. Service Events

Los eventos representan hechos ocurridos:

WorkoutCreated
WorkoutCompleted
SubscriptionCancelled
NutritionPlanGenerated

No deben confundirse:

Command = Do something
Event = Something happened
47. Event Ownership

El service que posee el dominio publica sus eventos.

Ejemplo:

Training Service
       │
       ▼
WorkoutCompleted

Otros servicios pueden reaccionar.

48. Event Contracts

Cada evento debe definir:

eventId
eventType
version
occurredAt
producer
tenantId
payload
correlationId
49. Service Versioning

Los services deben versionar contratos cuando existan breaking changes.

v1
v2
v3

Preferiblemente mediante compatibilidad hacia atrás cuando sea posible.

50. API Compatibility

Cambios deben clasificarse:

Non-breaking
Breaking
Deprecated
51. Service Configuration

Configuración separada del código:

Environment
Database
Queues
External Services
Feature Flags
Policy References

Nunca incluir secrets directamente en código.

52. Service Secrets

Deben utilizar:

Secret Manager
Environment Injection
Secure Configuration

Nunca:

Git repository
Source code
Plain configuration committed to repository
53. Service Health

Cada servicio debe proporcionar health checks:

/health
/health/live
/health/ready

Conceptualmente:

Liveness
→ Process is alive

Readiness
→ Service can accept traffic
54. Service Dependencies Health

Readiness puede considerar dependencias críticas:

Database
Queue
Required external services

pero no debe convertir una dependencia no crítica en un punto único de fallo.

55. Service Resilience

Los servicios deben incorporar:

Timeouts
Retries
Circuit Breakers
Bulkheads
Rate Limits
Fallbacks

cuando corresponda.

56. Timeout Policy

Toda comunicación entre servicios debe tener timeout explícito.

Nunca:

Infinite wait
57. Retry Policy

Retries solamente para errores recuperables.

Transient error
    ↓
Retry

No:

Invalid request
    ↓
Retry forever
58. Exponential Backoff

Los retries deben utilizar backoff:

1s
2s
4s
8s

con límites apropiados.

59. Circuit Breaker

Si un servicio dependiente falla repetidamente:

Closed
 ↓
Open
 ↓
Half Open
 ↓
Closed

Esto evita cascadas de fallos.

60. Bulkhead Isolation

Los recursos deben aislarse:

AI Requests
Billing Requests
Training Requests

para evitar que una carga excesiva de un dominio afecte a todo EVOXA.

61. Service Rate Limiting

Los servicios sensibles deben poder limitar:

Requests
Commands
AI operations
Exports
Heavy calculations
62. Service Observability

Cada service debe producir:

Logs
Metrics
Traces
Events
Health
63. Correlation

Una solicitud debe poder seguirse:

Request
 ↓
API
 ↓
Service A
 ↓
Service B
 ↓
Event
 ↓
Service C

mediante:

traceId
correlationId
requestId
64. Service Logging

Los logs deben estructurarse:

{
  "timestamp": "...",
  "service": "training-service",
  "level": "INFO",
  "traceId": "...",
  "message": "Workout completed"
}
65. Service Metrics

Métricas mínimas:

request_count
request_latency
error_count
dependency_latency
dependency_errors
queue_depth
active_requests
66. Service Tracing

Las llamadas internas deben generar spans:

API
 └── Training Service
      ├── Policy Evaluation
      ├── Database
      └── Event Publish
67. Service Security

Cada service debe asumir:

Zero Trust

No confiar automáticamente en otro servicio solo porque está dentro de la red interna.

68. Service Authentication

Service-to-service communication puede utilizar:

Service Identity
JWT
mTLS
OAuth2
Workload Identity

según el entorno.

69. Service Authorization

Además de autenticar un service:

Who is calling?

se debe determinar:

What can this service do?
70. Service Policy

E06 debe aplicarse también entre servicios.

Ejemplo:

AI Service
 →
Training Service

debe estar sujeto a una policy explícita.

71. Tenant Context

Toda operación tenant-aware debe transportar contexto de tenant:

tenantId

de manera confiable.

Nunca confiar exclusivamente en:

tenantId supplied by client
72. Tenant Isolation

El service debe validar:

Caller Tenant
=
Resource Tenant

cuando corresponda.

73. Service Context

Un contexto interno puede contener:

requestId
traceId
tenantId
subjectId
serviceIdentity
authenticationContext
authorizationContext
74. Service Execution Context

El contexto debe propagarse de forma segura:

API
 ↓
Service A
 ↓
Service B

pero no debe permitir que un servicio arbitrario falsifique:

subjectId
tenantId
roles
75. Service API Gateway

Las APIs públicas pueden pasar por:

Client
 ↓
API Gateway
 ↓
Backend Services

El Gateway puede encargarse de:

Routing
Rate Limiting
Authentication
TLS
Observability

pero la autorización crítica debe mantenerse en los servicios correspondientes.

76. Service Mesh

Puede incorporarse posteriormente para:

mTLS
Service Discovery
Traffic Management
Retries
Observability

No es requisito inicial.

77. Background Services

EVOXA tendrá procesos asíncronos:

Worker
Scheduler
Job Processor
Event Consumer
AI Worker
Notification Worker

Deben considerarse parte de la Service Architecture.

78. Worker Model
Queue
 ↓
Worker
 ↓
Service Logic
 ↓
Result
79. Job Idempotency

Jobs deben poder reintentarse sin duplicar efectos.

jobId
idempotencyKey
80. Scheduled Services

Ejemplos:

Daily Progress Calculation
Subscription Renewal
Reminder Processing
Data Aggregation
AI Evaluation

Deben tener:

Schedule
Owner
Retry Policy
Failure Handling
Audit
81. Service State

Los servicios deberían minimizar estado local mutable.

Preferencia:

Stateless Service
       +
External State

Esto facilita escalamiento horizontal.

82. Stateful Services

Cuando el estado sea necesario:

Cache
Session
Workflow
AI Memory

debe existir una estrategia explícita.

83. Service Scaling

Escalamiento horizontal:

Service Instance 1
Service Instance 2
Service Instance 3

La mayoría de servicios HTTP deberían ser stateless.

84. Independent Scaling

Uno de los beneficios de boundaries bien definidos:

AI Service
→ scale independently

Training Service
→ scale independently
85. Resource Isolation

Cada service debe tener límites de:

CPU
Memory
Connections
Workers
Queue Consumption
86. Service Deployment

Pipeline:

Source
 ↓
Build
 ↓
Test
 ↓
Security Scan
 ↓
Package
 ↓
Deploy
 ↓
Health Check
 ↓
Observe
87. Deployment Strategies

EVOXA puede soportar:

Rolling
Blue/Green
Canary
Feature-based rollout

según criticidad.

88. Service Rollback

Toda versión desplegada debe poder revertirse.

v2
 ↓
Incident
 ↓
Rollback
 ↓
v1
89. Service Lifecycle
Design
 ↓
Implement
 ↓
Test
 ↓
Deploy
 ↓
Operate
 ↓
Monitor
 ↓
Evolve
 ↓
Deprecate
 ↓
Retire
90. Service Deprecation

Antes de retirar un service:

Dependency Analysis
Migration
Communication
Monitoring
Shutdown
91. Service Registry

EVOXA debe mantener un catálogo de servicios:

Service Registry

con:

serviceId
domain
owner
version
status
dependencies
interfaces
deployment
92. Service Catalog Example
training-service
nutrition-service
progress-service
billing-service
identity-service
policy-service
notification-service
ai-service
agent-service
93. Service Documentation

Cada service debe tener:

README
Architecture
Responsibilities
API
Events
Dependencies
Configuration
Security
Operations
Runbook
94. Service Repository Structure

Conceptualmente:

services/
├── identity/
├── authorization/
├── policy/
├── training/
├── nutrition/
├── progress/
├── billing/
├── ai/
└── notifications/
95. Service Internal Structure

Ejemplo:

training/
├── api/
├── application/
├── domain/
├── infrastructure/
├── policies/
├── events/
├── tests/
└── README.md
96. Service Testing

Cada service debe incluir:

Unit Tests
Integration Tests
Contract Tests
Security Tests
Performance Tests
97. Contract Testing

Los consumidores y productores deben validar contratos.

Producer
   ↓
Contract
   ↓
Consumer

Esto reduce breaking changes.

98. Service Integration Testing

Debe validarse:

API
Service
Database
Events
Dependencies
Policies
99. Service Security Testing

Debe probar:

Unauthorized Access
Tenant Escape
Privilege Escalation
Invalid Tokens
Policy Bypass
Service Identity Spoofing
100. Definition of Done

E07 queda definido cuando EVOXA dispone de:

✓ Service Model
✓ Service Boundaries
✓ Service Types
✓ Service Ownership
✓ Application Services
✓ Domain Services
✓ Infrastructure Services
✓ Integration Services
✓ AI Services
✓ Service Contracts
✓ Service Interfaces
✓ Sync Communication
✓ Async Communication
✓ Commands
✓ Events
✓ Event Contracts
✓ Service Dependencies
✓ Dependency Rules
✓ Data Ownership
✓ Transaction Boundaries
✓ Idempotency
✓ Service Discovery
✓ Service Registry
✓ Service Configuration
✓ Service Security
✓ Service Authentication
✓ Service Authorization
✓ Tenant Isolation
✓ Service Health
✓ Resilience
✓ Timeouts
✓ Retries
✓ Circuit Breakers
✓ Observability
✓ Distributed Tracing
✓ Metrics
✓ Structured Logging
✓ Background Workers
✓ Scheduling
✓ Scaling
✓ Deployment
✓ Rollback
✓ Service Lifecycle
✓ Service Governance
✓ Service Catalog
✓ Contract Testing
✓ Security Testing
101. Relación con E06

La secuencia de Engineering Specification queda:

E04 Authentication
       │
       ▼
E05 Authorization
       │
       ▼
E06 Policy
       │
       ▼
E07 Service Architecture
       │
       ▼
E08 Domain Services Architecture

Y el flujo de ejecución:

Request
   │
   ▼
API
   │
   ▼
Authentication
   │
   ▼
Authorization
   │
   ▼
Policy
   │
   ▼
Application Service
   │
   ▼
Domain Service
   │
   ▼
Infrastructure
   │
   ▼
Database / External Systems

E07 establece así la columna vertebral de ejecución de EVOXA: define dónde vive la lógica, cómo se separan las responsabilidades, cómo se comunican los componentes y cómo los servicios pueden evolucionar desde el monolito modular inicial hacia servicios distribuidos sin perder límites de dominio, seguridad, observabilidad ni gobernanza.
