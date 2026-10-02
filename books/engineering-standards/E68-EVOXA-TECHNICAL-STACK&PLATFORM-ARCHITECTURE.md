E68 — EVOXA TECHNICAL STACK & PLATFORM ARCHITECTURE
1. Propósito

E68 transforma el E67 — Master Application Blueprint en una arquitectura tecnológica concreta.

La pregunta que responde es:

¿Con qué tecnologías, plataformas, runtimes, frameworks, sistemas de almacenamiento y herramientas construiremos EVOXA, y por qué?

La secuencia queda:

E67 — MASTER APPLICATION BLUEPRINT
                ↓
E68 — TECHNICAL STACK & PLATFORM
                ↓
E69 — PROJECT / REPOSITORY ARCHITECTURE
                ↓
E70 — MODULE & PACKAGE ARCHITECTURE
                ↓
E71 — DATABASE ARCHITECTURE
                ↓
E72 — API ARCHITECTURE
                ↓
E73 — TESTING ARCHITECTURE
                ↓
E74 — DEPLOYMENT ARCHITECTURE
                ↓
             CODE
2. Objetivos

E68 debe fijar:

Programming Language
Runtime
Application Framework
Database
Cache
Messaging
Search
Object Storage
Scheduler
Job Infrastructure
Observability
Security
Containerization
CI/CD
Infrastructure
Development Tooling
Testing Stack

Pero con una regla importante:

La tecnología debe servir a la arquitectura; la arquitectura no debe deformarse para justificar una tecnología.

3. Principios tecnológicos de EVOXA

La plataforma debe priorizar:

Correctness
Maintainability
Testability
Observability
Security
Auditability
Scalability
Operational Simplicity
Portability

Y evitar:

Technology for technology's sake
Premature Microservices
Premature Distributed Systems
Vendor Lock-in
Unnecessary Complexity
Duplicated Infrastructure
4. Stack tecnológico propuesto

Para EVOXA propongo inicialmente:

Capa	Tecnología
Lenguaje	Python 3.13+
API	FastAPI
Validation	Pydantic 2
ORM / Persistence	SQLAlchemy 2
Migrations	Alembic
Database	PostgreSQL
Cache	Redis
Messaging	NATS JetStream
Background Jobs	Dramatiq / NATS-based workers
Search	OpenSearch
Object Storage	S3-compatible storage
Containers	Docker
Local orchestration	Docker Compose
Observability	OpenTelemetry
Metrics	Prometheus-compatible
Dashboards	Grafana
Logs	Structured JSON logging
Testing	pytest
Static typing	mypy / pyright
Linting	Ruff
CI	GitHub Actions
Package management	uv
Documentation	Markdown + OpenAPI

Este es el stack de referencia arquitectónico, no una obligación irreversible para cada componente futuro.

5. Lenguaje: Python

La plataforma principal será:

Python

por su adecuación para:

Domain Logic
Application Services
Data Processing
Automation
Rules
Analytics
AI / Intelligence
Integration
API Development

Además permite que EVOXA mantenga un lenguaje común entre:

Application
Data
Intelligence
Automation
Governance
6. Python Version

Objetivo:

Python 3.13+

La versión concreta de producción deberá fijarse mediante lockfile y CI.

No debemos depender de:

python >= 3.x

sin límite superior controlado.

7. Runtime

El runtime principal será:

CPython

con procesos separados para:

API
Workers
Schedulers
Consumers
Administrative Processes

según las necesidades de ejecución.

8. API Framework
FastAPI

La API principal utilizará:

FastAPI

sobre:

ASGI

Arquitectura:

HTTP
 ↓
FastAPI
 ↓
API Layer
 ↓
Application Layer
 ↓
Domain
9. Validation

Se utilizará:

Pydantic 2

para:

Request Models
Response Models
Configuration
Serialization
Boundary Validation

Pero:

Pydantic no sustituye la validación del dominio.

Debe existir una separación:

Input Validation
        ↓
Application Validation
        ↓
Domain Invariants
10. Persistence

La persistencia principal utilizará:

PostgreSQL

como base de datos relacional primaria.

Motivos arquitectónicos:

ACID
Transactions
Constraints
Indexes
JSON Support
Mature Ecosystem
Reliability
Operational Maturity
11. ORM

El acceso se implementará mediante:

SQLAlchemy 2

con separación entre:

Domain Models
Persistence Models

cuando la complejidad lo justifique.

No se permitirá que los modelos ORM se conviertan automáticamente en el modelo de dominio.

12. Migrations

Las modificaciones del schema serán gestionadas mediante:

Alembic

Flujo:

Model Change
   ↓
Migration
   ↓
Review
   ↓
Test
   ↓
Apply

Nunca se dependerá de modificaciones manuales de producción como mecanismo normal.

13. PostgreSQL como Source of Truth

Para el estado transaccional:

PostgreSQL
      ↓
Authoritative State

Mientras:

Redis
OpenSearch
Read Models
Caches
Replicas

no serán considerados automáticamente fuentes autoritativas.

14. Cache

La plataforma utilizará:

Redis

para:

Caching
Distributed Locks
Rate Limiting
Short-lived State
Coordination

cuando el caso de uso lo justifique.

Regla:

Redis ≠ Primary Database
15. Messaging

Para mensajería interna:

NATS JetStream

será el mecanismo de referencia.

Arquitectura:

Producer
   ↓
NATS
   ↓
Consumer

con capacidades de:

Durability
Replay
Consumer Groups
Acknowledgement
At-least-once Delivery
16. Event Architecture

Los eventos de dominio podrán publicarse mediante:

Domain Event
    ↓
Outbox
    ↓
Event Publisher
    ↓
NATS JetStream

Esto evita depender de una publicación directa e insegura desde la transacción de dominio.

17. Event Delivery

La arquitectura debe asumir inicialmente:

At-Least-Once Delivery

Por tanto los consumidores deben ser:

Idempotent

y soportar:

Duplicate Detection
Retry
Dead Letter / Failure Handling

cuando corresponda.

18. Background Processing

Los trabajos asíncronos se ejecutarán mediante workers.

Arquitectura:

API / Workflow
      ↓
Message / Job
      ↓
Queue
      ↓
Worker
      ↓
Execution

La tecnología concreta del job runner deberá mantenerse desacoplada de la lógica de negocio.

19. Scheduling

El scheduler será una capacidad independiente:

Scheduler
   ↓
Trigger
   ↓
Job / Command
   ↓
Worker

Esto mantiene la separación definida en E16.

20. Search

Cuando EVOXA necesite capacidades avanzadas de búsqueda:

OpenSearch

será el motor de referencia.

Arquitectura:

Authoritative Data
       ↓
Projection
       ↓
OpenSearch
       ↓
Search

OpenSearch no sustituye PostgreSQL como fuente de verdad.

21. Object Storage

Los objetos grandes o blobs deberán mantenerse fuera de PostgreSQL cuando corresponda.

Arquitectura:

Application
     ↓
Object Storage
     ↓
S3-Compatible API

Casos:

Documents
Exports
Large Files
Artifacts
Archives
Data Packages
22. Object Storage Provider

E68 debe mantener una abstracción:

ObjectStorage

para evitar que el dominio dependa de:

AWS S3
MinIO
Azure Blob
GCS

La implementación concreta puede variar por entorno.

23. Containers

El packaging de EVOXA será:

Docker

Cada unidad ejecutable tendrá una imagen reproducible.

Ejemplo:

evoxa-api
evoxa-worker
evoxa-scheduler
evoxa-consumer
24. Local Development

Para desarrollo local:

Docker Compose

permitirá ejecutar:

PostgreSQL
Redis
NATS
OpenSearch
Object Storage
Observability
EVOXA

como stack reproducible.

25. Platform Architecture

El entorno local:

Developer
    │
    ▼
Docker Compose
    │
 ┌──┼───────────────┐
 ▼  ▼   ▼    ▼      ▼
DB Redis NATS Search Storage
    │
    ▼
 EVOXA
26. Production Platform

La plataforma debe poder evolucionar desde:

Single Host

hasta:

Container Platform

sin modificar el dominio.

La arquitectura de aplicación debe permanecer independiente de la infraestructura de deployment.

27. Kubernetes

Kubernetes no debería ser un requisito inicial.

La decisión arquitectónica será:

Containers first
Kubernetes when justified

Se introducirá cuando existan necesidades reales de:

Horizontal Scaling
Multi-Node Scheduling
High Availability
Operational Isolation
Deployment Automation
28. Cloud Abstraction

EVOXA debe evitar depender directamente de APIs cloud dentro del dominio.

Incorrecto:

Domain
 ↓
AWS SDK

Correcto:

Domain
 ↓
Port
 ↓
Infrastructure Adapter
 ↓
Cloud SDK
29. Dependency Injection

FastAPI puede proporcionar dependency injection en interfaces, pero la arquitectura interna debe utilizar sus propias abstracciones.

Interface
 ↓
Application Dependency
 ↓
Port
 ↓
Adapter

No debemos convertir el framework en el centro de la arquitectura.

30. Configuration

La configuración se manejará mediante:

Environment Variables
Configuration Files
Secrets Manager
Runtime Configuration

con una capa central:

Configuration
     ↓
Validation
     ↓
Typed Settings
31. Secrets

Los secretos nunca deben almacenarse en:

Source Code
Git
Dockerfile
Logs
Configuration Repository

Deben utilizar mecanismos de secrets apropiados para cada entorno.

32. Feature Flags

Los feature flags se mantendrán separados de configuration.

Configuration
→ How system is configured

Feature Flag
→ Which behavior is active
33. Logging

El logging será:

Structured
JSON
Machine-readable
Correlated

Ejemplo conceptual:

{
    "timestamp": "...",
    "level": "INFO",
    "service": "evoxa-api",
    "correlation_id": "...",
    "event": "..."
}
34. OpenTelemetry

La observabilidad distribuida utilizará:

OpenTelemetry

para:

Traces
Metrics
Logs Correlation
Context Propagation
35. Trace Context

Una request debe poder seguirse:

HTTP Request
     ↓
Application Service
     ↓
Database
     ↓
Event
     ↓
Consumer
     ↓
Worker
     ↓
External Service

mediante correlation / trace context.

36. Metrics

Las métricas deben cubrir:

Latency
Throughput
Errors
Queue Depth
Job Duration
Database Health
Cache Hit Rate
Resource Usage
Governance Signals
37. Health Checks

Cada servicio ejecutable debe poder exponer:

Liveness
Readiness
Dependency Health

Ejemplo conceptual:

/health/live
/health/ready
38. Security Stack

La plataforma debe incorporar:

TLS
Authentication
Authorization
Secrets Management
Encryption
Audit Logging
Rate Limiting
Input Validation

La tecnología concreta del identity provider podrá encapsularse detrás de una abstracción.

39. Identity

EVOXA debe separar:

Authentication

de:

Authorization

y soportar conceptualmente:

Human
Service
Machine
Automation

como identidades distintas.

40. Authorization

La autorización deberá soportar progresivamente:

RBAC
ABAC
Policy-Based Access
Resource-Level Authorization

sin introducir todas las capacidades desde el primer día.

41. Cryptography

Las operaciones criptográficas no deben implementarse manualmente.

Se utilizarán librerías maduras y mecanismos estándar.

Regla:

Never implement cryptography primitives yourself.

42. Testing Stack

La base será:

pytest

complementada con:

pytest-asyncio
Property-Based Testing
Contract Testing
Integration Testing

cuando corresponda.

43. Static Analysis

El proyecto utilizará:

Ruff

para linting y formatting.

Y:

mypy

o:

pyright

para análisis estático de tipos.

La elección definitiva entre mypy y pyright puede cerrarse durante E69.

44. Package Management

La gestión del proyecto Python utilizará:

uv

para:

Dependencies
Virtual Environments
Locking
Reproducibility
45. Dependency Locking

Las versiones de producción deben estar bloqueadas.

pyproject.toml
      +
lockfile
      ↓
Reproducible Environment

No se debe depender de versiones flotantes en producción.

46. CI/CD

El pipeline inicial:

Commit
 ↓
Lint
 ↓
Type Check
 ↓
Unit Tests
 ↓
Integration Tests
 ↓
Architecture Tests
 ↓
Security Checks
 ↓
Build
 ↓
Artifact

Deployment será tratado con mayor detalle en E74.

47. GitHub Actions

Para CI inicial:

GitHub Actions

permitirá ejecutar automáticamente:

Tests
Lint
Type Checking
Build
Security Scanning
48. Artifact Architecture

Los artefactos deben ser versionables:

Application Image
Migration Bundle
Package
Schema
Documentation
49. Security Scanning

CI debe incluir progresivamente:

Dependency Scanning
Secret Detection
Container Scanning
Static Analysis
50. Supply Chain Security

EVOXA debe controlar:

Dependencies
Versions
Lockfiles
Container Base Images
Build Provenance
Artifacts

Esto es especialmente importante para una plataforma de governance.

51. Database Connectivity

La aplicación no debe abrir conexiones arbitrariamente.

Debe existir:

Connection Pool
Timeouts
Transaction Management
Health Checks
52. Async Architecture

FastAPI y Python async se utilizarán cuando aporten valor:

I/O Bound
Networking
Messaging
External APIs

No se convertirá todo el código en async por defecto.

53. CPU-Bound Processing

Para procesamiento intensivo:

Worker
Process
Job

en lugar de bloquear el proceso HTTP.

54. Concurrency

EVOXA debe considerar:

Optimistic Concurrency
Idempotency
Locks
Transactions
Retries

según el caso.

55. Resilience Stack

Las aplicaciones deberán incorporar cuando corresponda:

Timeout
Retry
Backoff
Circuit Breaker
Bulkhead
Rate Limit
Fallback

No todas las operaciones necesitan todos estos mecanismos.

56. Retry Policy

Nunca se debe hacer:

while failure:
    retry()

sin límites.

Debe existir:

Max Attempts
Backoff
Jitter
Retryable Errors
Non-Retryable Errors
57. Messaging Reliability

El sistema debe soportar:

Duplicate Delivery
Consumer Restart
Transient Failure
Message Retry
Dead Letter Handling
Replay

cuando sea necesario.

58. Database Reliability

PostgreSQL debe configurarse con:

Backups
Monitoring
Connection Limits
Timeouts
Migration Control
Recovery Strategy
59. Platform Environment Model

Mínimo:

Development
Testing
Staging
Production

Cada uno debe tener configuración independiente.

60. Environment Promotion

Idealmente:

Development
      ↓
CI
      ↓
Testing
      ↓
Staging
      ↓
Production

El artefacto debe promoverse, no recompilarse arbitrariamente en cada ambiente.

61. Development Experience

Un nuevo desarrollador debería poder:

Clone
 ↓
Install
 ↓
Configure
 ↓
docker compose up
 ↓
Run Tests
 ↓
Start EVOXA

sin realizar una configuración manual extensa.

62. Developer Tooling

Se proporcionarán comandos estandarizados:

make test
make lint
make format
make typecheck
make migrate
make dev
make build

o equivalentes.

La interfaz exacta se definirá en E69.

63. Documentation Stack

La documentación técnica vivirá junto al código:

docs/
architecture/
adr/
api/
operations/
governance/

La documentación crítica deberá versionarse.

64. API Documentation

FastAPI permitirá generar:

OpenAPI

pero el contrato deberá mantenerse intencionalmente.

No toda la estructura interna del código debe exponerse automáticamente.

65. Schema Versioning

Deben versionarse:

API Schemas
Event Schemas
Message Schemas
Database Schemas
Configuration Schemas
66. Event Schema Evolution

Los eventos deben poder evolucionar sin romper consumidores.

Estrategias posibles:

Backward Compatibility
Versioned Events
Schema Evolution
Consumer Tolerance
67. Serialization Format

Para APIs:

JSON

como formato principal.

Para messaging:

JSON

inicialmente, con posibilidad de evolucionar hacia formatos binarios cuando existan razones concretas.

68. Data Formats

La plataforma debe diferenciar:

API Representation
Event Representation
Persistence Representation
Analytics Representation
Export Representation

No asumir que todos deben utilizar el mismo schema.

69. Platform Architecture — Full
                         EVOXA PLATFORM
                               │
             ┌─────────────────┼─────────────────┐
             │                 │                 │
             ▼                 ▼                 ▼
          CLIENTS          OPERATIONS        GOVERNANCE
             │                 │                 │
             ▼                 ▼                 ▼
          FASTAPI          OBSERVABILITY       AUDIT
             │                 │                 │
             └─────────────────┼─────────────────┘
                               ▼
                     APPLICATION RUNTIME
                               │
          ┌────────────────────┼────────────────────┐
          ▼                    ▼                    ▼
       DOMAIN              WORKERS             SCHEDULER
          │                    │                    │
          └────────────────────┼────────────────────┘
                               ▼
                        INFRASTRUCTURE
                               │
       ┌────────────┬──────────┼──────────┬────────────┐
       ▼            ▼          ▼          ▼            ▼
 PostgreSQL      Redis      NATS      OpenSearch   Object Store
       │            │          │          │            │
       └────────────┴──────────┼──────────┴────────────┘
                               ▼
                       EXTERNAL SYSTEMS
70. Architectural Technology Boundaries

La regla fundamental será:

Domain
   ↓
Interfaces / Ports
   ↓
Adapters
   ↓
Technology

Nunca:

Domain
   ↓
Technology Vendor
71. Technology Decision Matrix

Cada tecnología importante deberá justificarse por:

Criterio	Pregunta
Correctness	¿Mantiene las garantías necesarias?
Performance	¿Cumple la carga esperada?
Reliability	¿Tiene comportamiento estable?
Security	¿Puede protegerse adecuadamente?
Operability	¿Podemos operarla?
Maintainability	¿Podemos mantenerla durante años?
Ecosystem	¿Tiene soporte suficiente?
Portability	¿Podemos cambiar de proveedor?
Cost	¿Es sostenible?
Complexity	¿Cuánto añade?
72. Technology Governance

Una tecnología nueva no debería entrar simplemente porque:

"es popular".

Debe pasar:

Proposal
 ↓
Technical Evaluation
 ↓
ADR
 ↓
Approval
 ↓
Adoption
73. Approved Technology Categories

EVOXA tendrá una lista explícita:

Approved
Conditional
Experimental
Deprecated
Rejected

Esto evita que cada módulo adopte herramientas arbitrarias.

74. Experimental Technologies

Las tecnologías experimentales deben permanecer aisladas:

Experimental
    ↓
Adapter
    ↓
Stable Interface

Así podemos eliminarlas sin contaminar el dominio.

75. Vendor Lock-In

La estrategia será:

Business Logic
       ↓
EVOXA Port
       ↓
Provider Adapter

Especialmente para:

Object Storage
Messaging
Identity
Cloud Services
Search
76. Operational Simplicity

Una regla crítica:

Cada infraestructura añadida crea una responsabilidad operacional.

Por tanto:

PostgreSQL
Redis
NATS
OpenSearch
Object Storage

son suficientes para una primera plataforma potente.

No debemos añadir:

Kafka
RabbitMQ
Elasticsearch
MongoDB
Cassandra
Kubernetes
etc.

sin una necesidad arquitectónica demostrable.

77. Initial Platform Profile

La primera plataforma EVOXA debería ser:

Python
FastAPI
PostgreSQL
Redis
NATS JetStream
OpenSearch
S3-Compatible Storage
Docker
OpenTelemetry
Prometheus
Grafana
pytest
Ruff
mypy/pyright
uv
GitHub Actions
78. MVP Platform

El MVP puede reducirse aún más:

Python
FastAPI
PostgreSQL
Redis
NATS
Docker
pytest
Ruff
uv
OpenTelemetry

OpenSearch, object storage avanzado y algunos componentes pueden incorporarse cuando un caso real los necesite.

79. Evolution Path
PHASE 1
Modular Application
      ↓
PHASE 2
Workers + Messaging
      ↓
PHASE 3
Read Models + Search
      ↓
PHASE 4
Horizontal Scaling
      ↓
PHASE 5
Service Extraction
      ↓
PHASE 6
Distributed Platform

No saltamos directamente a Phase 6.

80. Platform Invariants
Invariant 1
Technology
must not leak into Domain.
Invariant 2
Source of Truth
must be explicit.
Invariant 3
Every asynchronous operation
must be idempotent where required.
Invariant 4
Every critical dependency
must have failure behavior defined.
Invariant 5
Every production dependency
must be observable.
Invariant 6
Secrets must never enter source control.
Invariant 7
Infrastructure complexity
must be justified by actual requirements.
Invariant 8
Technology replacement
must be possible at architectural boundaries.
81. E68 Acceptance Criteria

E68 está completo cuando:

✓ Language selected
✓ Runtime selected
✓ API framework selected
✓ Validation stack selected
✓ Persistence stack selected
✓ Cache selected
✓ Messaging strategy selected
✓ Job strategy selected
✓ Search strategy selected
✓ Object storage strategy selected
✓ Container strategy selected
✓ Observability strategy selected
✓ Security strategy defined
✓ Testing stack selected
✓ CI strategy defined
✓ Dependency management defined
✓ Configuration strategy defined
✓ Secrets strategy defined
✓ Environment model defined
✓ Technology boundaries defined
✓ Vendor lock-in strategy defined
✓ Evolution strategy defined
82. Decisión arquitectónica principal

El resultado de E68 puede resumirse así:

                    EVOXA
                      │
              MODULAR APPLICATION
                      │
        ┌─────────────┼─────────────┐
        ▼             ▼             ▼
      DOMAIN      APPLICATION   GOVERNANCE
        │             │             │
        └─────────────┼─────────────┘
                      ▼
                PYTHON RUNTIME
                      │
          ┌───────────┼───────────┐
          ▼           ▼           ▼
      PostgreSQL    Redis       NATS
          │                       │
          ▼                       ▼
     Source of Truth          Messaging
                      │
          ┌───────────┴───────────┐
          ▼                       ▼
      OpenSearch             Object Storage
          │                       │
          └───────────┬───────────┘
                      ▼
                 OBSERVABILITY
                      │
             OpenTelemetry
                      │
              ┌───────┴───────┐
              ▼               ▼
          Metrics           Traces
83. Resultado

Con E68 ya tenemos una respuesta concreta a:

¿Con qué vamos a construir EVOXA?

La decisión de arquitectura tecnológica queda:

Python + FastAPI + PostgreSQL + Redis + NATS JetStream + OpenSearch + S3-compatible storage + Docker + OpenTelemetry + pytest + Ruff + uv + CI/CD.

Pero todavía no empezaría a crear las clases y módulos definitivos.

El siguiente documento debe convertir este stack en una estructura física de proyecto:

E69 — EVOXA PROJECT / REPOSITORY ARCHITECTURE

Ahí definiremos exactamente:

EVOXA/
├── src/
│   ├── core/
│   ├── domain/
│   ├── application/
│   ├── infrastructure/
│   ├── interfaces/
│   └── governance/
├── tests/
├── docs/
├── scripts/
├── migrations/
├── deployment/
└── ...

y, sobre todo, qué paquete depende de cuál, qué archivos existirán inicialmente y cómo quedará preparado el repositorio para empezar a escribir el primer código real de EVOXA.
