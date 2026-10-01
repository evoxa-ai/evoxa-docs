E11 — EVOXA Integration Architecture
Engineering Specification

Pertenece a: Engineering Specification
Volumen: volume-09-engineering
Capítulo: E11 — Integration Architecture
Anterior: E10 — Repository Architecture
Siguiente: E12 — Messaging Architecture

1. Propósito

E11 define la arquitectura de integraciones de EVOXA con sistemas externos.

Mientras E10 establece cómo EVOXA persiste y recupera su propio estado, E11 establece cómo EVOXA se comunica con:

External APIs
Payment Providers
Identity Providers
AI Providers
Email Providers
SMS Providers
Push Providers
Cloud Services
Webhooks
Partner Platforms
Mobile Applications
Enterprise Systems

La pregunta central es:

¿Cómo puede EVOXA integrarse con sistemas externos sin contaminar el dominio, mantener seguridad, resiliencia, observabilidad y permitir cambiar proveedores sin reconstruir la plataforma?

2. Objetivos

La arquitectura de integración debe proporcionar:

Provider Abstraction
API Integration
Webhook Integration
Event Integration
Authentication
Authorization
Retries
Timeouts
Circuit Breaking
Idempotency
Rate Limiting
Contract Validation
Error Translation
Observability
Security
Versioning
Provider Substitution
3. Principio Fundamental

Las integraciones externas deben estar aisladas mediante adapters.

EVOXA
  │
  ▼
Application Service
  │
  ▼
Integration Interface
  │
  ▼
Adapter
  │
  ▼
External Provider

El dominio nunca debería depender directamente de:

Stripe
OpenAI
Google
AWS
Twilio
SendGrid

o cualquier proveedor específico.

4. Integration Boundary

La frontera arquitectónica:

┌──────────────────────────────────┐
│ EVOXA Core                       │
│                                  │
│ Domain                           │
│ Application                      │
│ Policies                         │
└────────────────┬─────────────────┘
                 │
═════════════════╪══════════════════
     Integration Boundary
═════════════════╪══════════════════
                 │
┌────────────────▼─────────────────┐
│ Integration Layer                │
│                                  │
│ Interfaces                       │
│ Adapters                         │
│ Clients                          │
│ Mappers                          │
│ Retry Policies                   │
└────────────────┬─────────────────┘
                 │
                 ▼
        External Systems
5. Integration Categories

EVOXA debe distinguir diferentes tipos de integración:

Synchronous API
Asynchronous Messaging
Webhooks
File Exchange
Event Streaming
OAuth / Identity
Payment Integration
AI Integration
Communication Integration
Storage Integration
Analytics Integration
6. Synchronous Integration

Una integración síncrona espera una respuesta.

EVOXA
  │
  ▼
External API
  │
  ▼
Response

Ejemplos:

Payment Authorization
AI Generation
Address Validation
Identity Verification
7. Asynchronous Integration

Una operación puede ejecutarse mediante eventos:

EVOXA
  │
  ▼
Message
  │
  ▼
Broker
  │
  ▼
External Consumer

Ventajas:

Decoupling
Resilience
Scalability
Retry
8. Webhook Integration

EVOXA también puede recibir eventos externos:

External Provider
       │
       ▼
Webhook
       │
       ▼
EVOXA Webhook Endpoint
       │
       ▼
Validation
       │
       ▼
Application Service

Ejemplos:

PaymentCompleted
SubscriptionChanged
IdentityUpdated
ExternalJobCompleted
9. Integration Adapter

Un Adapter traduce el contrato externo al modelo interno.

External API
     │
     ▼
Provider Adapter
     │
     ▼
EVOXA Integration Contract

Por ejemplo:

StripePaymentAdapter

implementa:

PaymentProvider
10. Provider Abstraction

EVOXA debe depender de contratos internos:

PaymentProvider
AIProvider
EmailProvider
SmsProvider
StorageProvider
IdentityProvider

y no directamente del proveedor.

11. Provider Substitution

La arquitectura debe permitir:

PaymentProvider
       │
 ┌─────┼──────┐
 ▼     ▼      ▼
Provider A
Provider B
Provider C

sin cambiar el dominio.

12. Integration Interface

Ejemplo conceptual:

interface PaymentProvider:

    authorize(payment)
    capture(payment)
    refund(payment)

La implementación puede ser:

StripePaymentAdapter

o:

AnotherPaymentAdapter
13. Application Integration Flow

El patrón estándar:

Command
  ↓
Application Service
  ↓
Policy
  ↓
Domain
  ↓
Integration Interface
  ↓
Adapter
  ↓
External Provider
14. Domain Isolation

Incorrecto:

Workout
   ↓
OpenAI API

Correcto:

Application Service
   ↓
AI Provider Interface
   ↓
AI Adapter
   ↓
AI Provider
15. Integration Mapping

Los modelos externos nunca deberían entrar directamente al dominio.

External DTO
     ↓
Adapter Mapper
     ↓
EVOXA Integration Model
     ↓
Application
16. External DTO

Un proveedor puede responder:

provider_user_id
provider_status
provider_metadata

EVOXA debe traducirlo a:

ExternalCustomer
ExternalStatus
ExternalReference

antes de entrar al resto del sistema.

17. API Client

Los clientes HTTP deben estar encapsulados.

PaymentAdapter
      │
      ▼
PaymentHttpClient
      │
      ▼
HTTP

El Application Service no debería ejecutar HTTP directamente.

18. HTTP Client Responsibilities

Un HTTP client puede gestionar:

Base URL
Headers
Authentication
Timeout
Serialization
Deserialization
Connection Pool
Retry
Tracing
19. Timeout Strategy

Toda llamada externa debe tener timeout explícito.

EVOXA
  │
  ▼
External API
  │
  ├── Response
  └── Timeout

Nunca depender indefinidamente de una respuesta externa.

20. Connection Management

Los clientes deben reutilizar conexiones cuando sea apropiado.

Connection Pool
     │
     ├── Request 1
     ├── Request 2
     └── Request 3

Esto mejora rendimiento y reduce overhead.

21. Retry Strategy

Los retries deben utilizarse solamente para errores transitorios.

Ejemplos:

Timeout
Temporary Network Failure
HTTP 502
HTTP 503
HTTP 504

No necesariamente:

HTTP 400
HTTP 401
HTTP 403
Business Validation Error
22. Exponential Backoff

Los retries deben utilizar backoff:

Attempt 1
   ↓
100 ms

Attempt 2
   ↓
200 ms

Attempt 3
   ↓
400 ms

con jitter cuando corresponda.

23. Retry Limits

Nunca realizar retries infinitos.

Ejemplo:

max_attempts = 3

El número exacto dependerá del tipo de integración.

24. Idempotency

Las operaciones externas críticas deben utilizar idempotencia.

Ejemplos:

CreatePayment
CapturePayment
CreateSubscription
SendWebhook
ProvisionAccount
25. Idempotency Key

EVOXA puede generar:

idempotency_key

basado en:

tenant
operation
business_reference

cuando el proveedor lo soporte.

26. Duplicate Request Protection

Ejemplo:

Request A
   ↓
Payment Provider
   ↓
Timeout

Request A retry
   ↓
Payment Provider

Sin idempotencia podría producir:

Payment #1
Payment #2

Con idempotencia:

Payment #1
Payment #1
27. Circuit Breaker

Cuando un proveedor presenta fallas persistentes:

Closed
  ↓
Failures
  ↓
Open
  ↓
Recovery
  ↓
Half Open
  ↓
Closed

Esto evita saturar un proveedor caído.

28. Circuit Breaker Scope

El Circuit Breaker debe ser específico para:

Provider
Operation
Integration

cuando sea necesario.

Ejemplo:

AIProvider
PaymentProvider
EmailProvider

pueden tener comportamientos independientes.

29. Rate Limiting

Las integraciones deben respetar límites externos:

Requests / second
Requests / minute
Daily quotas
Token limits
30. Rate Limit Handling

Cuando el proveedor devuelve:

429 Too Many Requests

EVOXA puede:

Read Retry-After
Backoff
Queue Request
Retry

según el caso.

31. Integration Security

Toda integración debe considerar:

Authentication
Authorization
Secret Management
TLS
Certificate Validation
Signature Validation
Credential Rotation
Least Privilege
32. Secrets

Las credenciales nunca deben estar hardcoded.

Incorrecto:

API_KEY = "secret..."

Preferible:

Secret Manager
Environment Configuration
Secure Credential Store
33. Credential Rotation

Las credenciales externas deben poder rotarse sin modificar el código.

Old Credential
     ↓
Rotation
     ↓
New Credential
34. OAuth 2.0

Para integraciones que utilicen OAuth:

Authorization
     ↓
Access Token
     ↓
API Request

debe gestionarse:

Token Expiration
Refresh
Revocation
Scopes
35. Webhook Security

Los webhooks deben validar:

Signature
Timestamp
Nonce
Source
Event Type
Schema

antes de procesarse.

36. Webhook Flow
External Provider
      │
      ▼
Webhook Endpoint
      │
      ▼
Authenticate Signature
      │
      ▼
Validate Schema
      │
      ▼
Check Idempotency
      │
      ▼
Persist Event
      │
      ▼
Process
37. Webhook Idempotency

Un webhook puede recibirse varias veces:

Event A
Event A
Event A

EVOXA debe producir un único efecto empresarial.

External Event ID
       ↓
Deduplication
       ↓
Process once
38. Webhook Acknowledgement

Los endpoints webhook deben responder rápidamente cuando sea posible.

Webhook
  ↓
Validate
  ↓
Persist
  ↓
HTTP 2xx
  ↓
Async Processing

Esto evita que el proveedor reintente innecesariamente.

39. Webhook Event Store

Puede mantenerse un registro:

IntegrationEvent

con:

event_id
provider
event_type
received_at
payload_hash
status
processed_at
40. Integration Event States
RECEIVED
VALIDATED
PROCESSING
PROCESSED
FAILED
RETRYING
DEAD_LETTER
41. Dead Letter Handling

Eventos que no pueden procesarse después de los retries pueden pasar a:

Dead Letter Queue

para:

Investigation
Replay
Manual Recovery
42. External Event Ordering

No asumir siempre que eventos llegan ordenados.

Ejemplo:

SubscriptionUpdated
SubscriptionCreated

podrían llegar fuera de orden.

EVOXA debe utilizar:

Event Version
Timestamp
Sequence
Provider State

cuando el proveedor lo permita.

43. Integration Contract

Cada integración debe tener un contrato explícito:

Provider
Operation
Request
Response
Authentication
Errors
Timeout
Retry
Idempotency
Rate Limits
Version
44. Contract Versioning

Las APIs externas pueden cambiar.

EVOXA debe soportar:

Provider API v1
Provider API v2

mediante adapters/versiones cuando sea necesario.

45. Contract Testing

Las integraciones deben validar:

Request Schema
Response Schema
Required Fields
Error Codes
Authentication
Webhook Schema
46. Consumer Contract

Cuando EVOXA expone integraciones para terceros:

Third Party
     ↓
EVOXA API

EVOXA debe mantener contratos versionados.

47. Integration Catalog

EVOXA debe mantener un catálogo:

Integration
Provider
Capability
Direction
Protocol
Authentication
Owner
Version
Status
SLA

Ejemplo:

Payment
AI
Email
Identity
Storage
Analytics
48. Integration Ownership

Cada integración debe tener un responsable:

Integration Owner
Technical Owner
Business Owner
Security Owner
49. Integration Health

Cada integración debe poder informar:

Available
Degraded
Unavailable
Misconfigured
Rate Limited
Authentication Failed
50. Health Checks

No todos los proveedores deben consultarse constantemente.

Puede utilizarse:

Connectivity Check
Authentication Check
Synthetic Request
Provider Status

según el coste y riesgo.

51. Integration Observability

Cada llamada debe permitir rastrear:

trace_id
correlation_id
provider
operation
duration
status
retry_count
error_type
52. Sensitive Data

Nunca registrar indiscriminadamente:

Passwords
Access Tokens
API Keys
Payment Credentials
Personal Sensitive Data

Los logs deben aplicar masking/redaction.

53. Integration Metrics

Métricas recomendadas:

integration_requests_total
integration_failures_total
integration_latency
integration_retries_total
integration_timeouts_total
integration_rate_limits_total
integration_circuit_open_total
54. Provider SLA

Las integraciones críticas deben considerar:

Availability
Latency
Error Rate
Rate Limits
Support
Recovery
55. Integration Dependency Map

EVOXA debe conocer qué capacidades dependen de cada proveedor.

Payment Provider
 ├── Subscription
 ├── Billing
 └── Refunds

AI Provider
 ├── Training Plans
 ├── Nutrition Analysis
 └── Coaching

Esto facilita evaluar impacto ante fallos.

56. Failure Isolation

Una integración caída no debería derribar toda la plataforma.

Ejemplo:

AI Provider DOWN
       │
       ├── AI features → degraded
       │
       └── Training CRUD → available
57. Graceful Degradation

Cuando sea posible:

Primary Provider
      ↓
Failure
      ↓
Fallback

o:

External Service unavailable
      ↓
Queue Request
      ↓
Process Later
58. Fallback Providers

Algunas capacidades pueden utilizar:

Primary Provider
Secondary Provider

Ejemplo:

EmailProvider
   ├── Provider A
   └── Provider B

El fallback debe ser explícito y gobernado.

59. Integration Queue

Las operaciones no urgentes pueden utilizar:

Application Service
      ↓
Queue
      ↓
Integration Worker
      ↓
External API

Ejemplos:

Email
Reports
AI Batch
Analytics
Notifications
60. Synchronous vs Asynchronous

Regla orientativa:

Synchronous
User requires immediate result
Asynchronous
Long running
Retryable
High volume
Non-critical immediate response
61. Integration Workflow

Un workflow puede ser:

CreateSubscription
      ↓
Payment Authorization
      ↓
Provider Confirmation
      ↓
Entitlement Provisioning
      ↓
Subscription Activation

Si existe una dependencia de múltiples sistemas, puede requerir una Saga.

62. Saga Integration
Step 1
Payment

Step 2
Provision

Step 3
Activate

Si Step 3 falla:

Compensation
63. Compensation

Ejemplo:

Payment successful
Provisioning failed

EVOXA puede:

Retry Provisioning

o:

Refund Payment

según las reglas del negocio.

64. Integration State Machine

Procesos externos pueden tener estados:

PENDING
PROCESSING
SUCCEEDED
FAILED
CANCELLED
UNKNOWN

El estado debe persistirse cuando la operación no es instantánea.

65. Unknown State

Especial atención:

EVOXA
  ↓
Provider
  ↓
Timeout

No significa necesariamente:

FAILED

Podría significar:

UNKNOWN

Debe existir un mecanismo de reconciliación.

66. Reconciliation

EVOXA debe soportar procesos de reconciliación para integraciones críticas.

EVOXA State
     │
     ▼
Provider State
     │
     ▼
Compare
     │
 ┌───┴────┐
 ▼        ▼
Match   Mismatch
          │
          ▼
       Repair

Especialmente importante para:

Payments
Subscriptions
Entitlements
External Provisioning
67. Integration Jobs

Los procesos de reconciliación pueden ejecutarse:

Hourly
Daily
On Demand
Event Triggered
68. Integration Data Ownership

Debe definirse:

System of Record
System of Reference
Cached Data
Derived Data

Ejemplo:

Payment status
→ Payment Provider may be authoritative

EVOXA subscription state
→ EVOXA may be authoritative for product entitlement

La autoridad depende del dominio.

69. External Identity

Si EVOXA utiliza identidad externa:

External Identity Provider
       ↓
Identity Adapter
       ↓
EVOXA Identity

Debe evitarse que el proveedor externo se convierta automáticamente en el modelo de identidad interno.

70. AI Provider Integration

La arquitectura AI debe utilizar:

AIProvider

con adapters para distintos proveedores.

AI Application Service
       ↓
AIProvider
       ↓
AI Adapter
       ↓
Model Provider

Esto permite cambiar modelos sin modificar el dominio.

71. AI Integration Safety

La respuesta de un proveedor AI debe tratarse como:

Untrusted External Output

antes de utilizarla para modificar estado crítico.

Debe pasar por:

Schema Validation
Policy Validation
Domain Validation
72. Payment Integration

Payment integration debe separar:

Payment Domain

de:

Payment Provider

Ejemplo:

PaymentApplicationService
       ↓
PaymentProvider
       ↓
Provider Adapter
73. Communication Integrations

Para:

Email
SMS
Push
WhatsApp

se recomienda:

NotificationProvider

y adapters específicos.

74. Storage Integration

Para archivos:

ObjectStorage
       ↓
StorageProvider
       ↓
S3 Adapter

El dominio trabaja con:

FileReference

no con APIs específicas de S3.

75. Integration Security Boundary
Application
     │
     ▼
Integration Interface
     │
     ▼
Security Adapter
     │
     ▼
External Provider

Las credenciales permanecen exclusivamente en la capa de infraestructura.

76. Integration Configuration

La configuración debe separarse del código:

Provider URL
Timeout
Retry Policy
Credentials Reference
Rate Limit
Feature Flags
77. Feature Flags

Las integraciones pueden activarse mediante:

Feature Flag

Ejemplo:

AI_PROVIDER_V2_ENABLED

Esto permite despliegues graduales.

78. Integration Rollout

Una nueva integración puede desplegarse:

Internal
 ↓
5%
 ↓
25%
 ↓
50%
 ↓
100%

con monitoreo en cada etapa.

79. Provider Migration

Migrar proveedores debe permitir:

Provider A
    │
    ▼
Dual Read / Dual Write
    │
    ▼
Provider B

cuando sea necesario.

80. Integration Testing Environments

Las integraciones deben soportar:

Mock
Sandbox
Test Environment
Production

Nunca probar operaciones destructivas directamente en producción cuando exista sandbox.

81. Contract Mocks

Los mocks deben representar el contrato real:

Request
Response
Errors
Timeouts
Rate Limits

No solamente respuestas felices.

82. Failure Testing

Cada integración crítica debe probar:

Timeout
5xx
4xx
Malformed Response
Invalid Signature
Duplicate Event
Rate Limit
Provider Down
Credential Expired
83. Chaos Testing

Para integraciones críticas:

Provider Failure
Network Failure
Latency Injection
Message Duplication
Out-of-order Events

pueden utilizarse pruebas controladas.

84. Integration Anti-Patterns

EVOXA debe evitar:

Direct HTTP from Domain
Provider SDK in Domain
Hardcoded Credentials
Infinite Retries
No Timeout
No Idempotency
No Webhook Verification
Provider-specific Models Everywhere
Hidden External Side Effects
No Reconciliation
85. Provider SDK Isolation

Los SDK externos deben estar encapsulados:

Provider Adapter
      ↓
Provider SDK

No:

Domain
      ↓
Provider SDK
86. Integration Documentation

Cada integración debe documentar:

Purpose
Provider
Capabilities
Direction
Protocol
Authentication
Endpoints
Schemas
Retries
Timeout
Idempotency
Rate Limits
Errors
Events
Webhooks
Security
Observability
SLA
Recovery
Owner
87. Integration Lifecycle
Identify Need
      ↓
Define Contract
      ↓
Select Provider
      ↓
Define Adapter
      ↓
Implement
      ↓
Security Review
      ↓
Contract Tests
      ↓
Sandbox Testing
      ↓
Production
      ↓
Monitor
      ↓
Reconcile
      ↓
Upgrade
      ↓
Deprecate
88. Integration Governance

Toda integración debe cumplir:

Security Standards
Architecture Standards
Data Protection
Observability
Contract Versioning
Operational Ownership
Failure Handling
Documentation
89. Integration Registry

EVOXA debería mantener conceptualmente un:

Integration Registry

con:

integration_id
provider
capability
version
status
owner
environment
authentication_type
criticality
sla
90. Criticality Levels

Las integraciones pueden clasificarse:

CRITICAL
HIGH
MEDIUM
LOW

Ejemplo:

Payment
→ CRITICAL

Email
→ MEDIUM

Analytics
→ LOW

La clasificación determinará:

Retry
Monitoring
Fallback
SLA
Recovery
91. End-to-End Integration Example
Generate AI Training Plan
Client
  │
  ▼
Application Service
  │
  ├── Authorization
  ├── Entitlement
  ├── Policy
  │
  ▼
AIProvider Interface
  │
  ▼
AI Adapter
  │
  ▼
AI Provider
  │
  ▼
Response
  │
  ▼
Schema Validation
  │
  ▼
Domain Validation
  │
  ▼
TrainingPlan
  │
  ▼
Repository
  │
  ▼
Outbox

La IA nunca modifica directamente la base de datos.

92. End-to-End Payment Example
CreateSubscription
       │
       ▼
Authorization
       │
       ▼
Subscription Domain
       │
       ▼
PaymentProvider
       │
       ▼
Payment Adapter
       │
       ▼
External Payment Provider
       │
       ▼
Payment Result
       │
       ▼
Persist
       │
       ▼
Outbox
       │
       ▼
SubscriptionActivated
93. End-to-End Webhook Example
Payment Provider
       │
       ▼
POST /webhooks/payment
       │
       ▼
Signature Validation
       │
       ▼
Schema Validation
       │
       ▼
Idempotency Check
       │
       ▼
Persist Integration Event
       │
       ▼
ACK
       │
       ▼
Async Processor
       │
       ▼
Application Service
       │
       ▼
Domain
94. Integration Architecture Model

La arquitectura consolidada:

                       EVOXA
                         │
                         ▼
                Application Services
                         │
                         ▼
                Integration Contracts
                         │
              ┌──────────┼──────────┐
              ▼          ▼          ▼
           API         Events     Webhooks
              │          │          │
              ▼          ▼          ▼
           Adapter     Broker     Receiver
              │
       ┌──────┼──────────┐
       ▼      ▼          ▼
    Payment    AI      Communication
       │       │          │
       ▼       ▼          ▼
   External Providers / Systems
95. Relationship E09–E11

La secuencia de Engineering Specification queda:

E09 — Application Services
        │
        ▼
     Orchestrates
        │
        ▼
E10 — Repository Architecture
        │
        ▼
     Internal State
        │
        ▼
E11 — Integration Architecture
        │
        ▼
     External Systems

Por lo tanto:

E09
→ What EVOXA executes

E10
→ How EVOXA persists its own state

E11
→ How EVOXA communicates with the outside world
96. Definition of Done

E11 queda definido cuando EVOXA dispone de:

✓ Integration Boundaries
✓ Integration Categories
✓ Synchronous Integrations
✓ Asynchronous Integrations
✓ Webhooks
✓ Adapters
✓ Provider Abstraction
✓ API Clients
✓ DTO Mapping
✓ Timeout Strategy
✓ Retry Strategy
✓ Exponential Backoff
✓ Circuit Breakers
✓ Rate Limiting
✓ Idempotency
✓ OAuth
✓ Credential Management
✓ Secret Rotation
✓ Webhook Security
✓ Event Deduplication
✓ Dead Letter Handling
✓ Contract Versioning
✓ Contract Testing
✓ Integration Catalog
✓ Integration Ownership
✓ Health Monitoring
✓ Observability
✓ Sensitive Data Protection
✓ Failure Isolation
✓ Graceful Degradation
✓ Fallback Strategy
✓ Saga Coordination
✓ Reconciliation
✓ AI Integration
✓ Payment Integration
✓ Communication Integration
✓ Storage Integration
✓ Feature Flags
✓ Provider Migration
✓ Sandbox Strategy
✓ Failure Testing
✓ Chaos Testing
✓ Governance
✓ Integration Registry
✓ Integration Lifecycle
✓ Documentation
97. Engineering Architecture Progress

Con E11 completado, avanzamos:

E01 — Backend Architecture
E02 — Database Architecture
E03 — API Architecture
E04 — Authentication Architecture
E05 — Authorization Architecture
E06 — Policy Architecture
E07 — Service Architecture
E08 — Domain Services Architecture
E09 — Application Services Architecture
E10 — Repository Architecture
E11 — Integration Architecture

La arquitectura comienza a formar una cadena completa:

                 EVOXA ENGINEERING
                        │
        ┌───────────────┴───────────────┐
        ▼                               ▼
   API / Security                  Application
        │                               │
        ▼                               ▼
 Authentication                  Application Services
        │                               │
        ▼                               ▼
 Authorization                   Domain Services
        │                               │
        ▼                               ▼
 Policies                         Domain Model
                                        │
                              ┌─────────┴─────────┐
                              ▼                   ▼
                        Repositories       Integrations
                              │                   │
                              ▼                   ▼
                         PostgreSQL          External World

E11 establece oficialmente la frontera de integración de EVOXA: el núcleo de la plataforma permanece independiente de proveedores externos, mientras que adapters, contracts, clients, webhooks, resiliencia, seguridad y observabilidad encapsulan toda interacción con el ecosistema externo.

Siguiente capítulo: E12 — EVOXA Messaging Architecture.
