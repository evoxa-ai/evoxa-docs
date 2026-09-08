Technical Tasks — IS-001
TASK-001-01 — API Runtime Foundation

Objetivo: preparar el runtime FastAPI del backend.

Debe incluir
aplicación FastAPI;
configuración;
environment handling;
estructura inicial de API;
/api/v1;
manejo global de excepciones;
correlation ID;
health endpoints;
configuración de logging estructurado.
Resultado
apps/api/
├── app/
│   ├── main.py
│   ├── core/
│   └── api/
│       └── v1/
└── tests/
TASK-001-02 — Identity Domain Package

Crear el bounded context:

app/domains/identity/

Debe existir como módulo independiente.

Regla

Ningún otro dominio podrá importar directamente elementos internos de Identity.

Los consumidores deberán utilizar contratos públicos.

TASK-001-03 — Layer Architecture

Crear explícitamente las capas:

API
Application
Domain
Infrastructure

Ejemplo:

identity/
├── account/
│   ├── api/
│   ├── application/
│   ├── domain/
│   └── infrastructure/
Regla
API
 ↓
Application
 ↓
Domain

Infrastructure implementa las interfaces necesarias.

TASK-001-04 — Identity Shared Kernel

Crear solamente los elementos realmente compartidos.

Por ejemplo:

identity/shared/
├── domain/
│   ├── identifiers/
│   ├── value_objects/
│   └── errors/
│
├── application/
│   └── context/
│
└── infrastructure/

Conceptos iniciales:

TenantContext
ActorContext
CorrelationContext
EntityId
DomainError

No debemos convertir shared en un cajón de sastre.

TASK-001-05 — Domain Events Foundation

Preparar la abstracción de eventos de dominio.

Conceptualmente:

Domain Event
├── event_id
├── event_type
├── event_version
├── occurred_at
├── tenant_id
├── actor
├── resource
└── payload

Todavía no vamos a congelar nombres de topics ni contratos concretos.

Eso deberá respetar la arquitectura de eventos y sus decisiones posteriores.

TASK-001-06 — Repository Contracts

Definir interfaces de repository en Domain.

Ejemplo:

UserRepository
OrganizationRepository
MembershipRepository
RoleRepository
PermissionRepository
SessionRepository

Pero en IS-001 solo necesitamos establecer el patrón.

No implementaremos todavía todo el CRUD.

TASK-001-07 — Unit of Work / Transaction Boundary

Crear la abstracción necesaria para controlar transacciones.

Conceptualmente:

Application Use Case
       ↓
Unit Of Work
       ↓
Repositories
       ↓
PostgreSQL

Esto será especialmente importante posteriormente para:

State Change
+
Outbox Event

dentro de una misma transacción.

TASK-001-08 — PostgreSQL Infrastructure

Preparar la conexión PostgreSQL.

Debe contemplar:

configuración externa;
connection lifecycle;
pooling;
transaction handling;
migrations;
health checking.

No debemos introducir todavía todo el modelo de Identity.

TASK-001-09 — Redis Infrastructure

Preparar Redis como infraestructura secundaria.

Uso futuro:

Redis
├── Cache
├── MFA temporary state
├── Rate limiting
├── authorization cache
└── temporary security state

Pero:

Redis nunca será la fuente de verdad de Identity.

TASK-001-10 — Configuration & Secrets

Crear configuración centralizada.

Conceptualmente:

config/
├── application
├── database
├── redis
├── security
├── events
└── observability

Secrets deben venir de configuración segura/Secret Manager.

Nunca:

password = "..."
jwt_secret = "..."

en código.

TASK-001-11 — Security Foundation

Crear las abstracciones base de seguridad.

Debe preparar:

PasswordHasher
TokenSigner
TokenVerifier
SecretProvider
SecurityContext

Todavía no debemos decidir parámetros concretos que quedaron pendientes en ADRs.

Por ejemplo, no debemos inventar ahora los parámetros exactos de Argon2id ni TTL definitivos de tokens.

TASK-001-12 — Tenant Context Foundation

Crear el contexto de tenant.

Conceptualmente:

TenantContext
├── tenant_id
├── organization_id
├── membership_id
└── actor

Debe ser validable.

La arquitectura define al Tenant como frontera de seguridad y ownership.

TASK-001-13 — Actor Context

Preparar:

ActorContext

Debe poder representar:

USER
SERVICE
SYSTEM
AI_AGENT

Esto será utilizado posteriormente por:

authorization;
audit;
security events;
observability.
TASK-001-14 — Correlation & Trace Context

Establecer:

correlation_id
trace_id
causation_id

para que una operación pueda seguirse:

HTTP Request
 ↓
Use Case
 ↓
Domain
 ↓
DB
 ↓
Outbox
 ↓
Event
 ↓
Consumer
TASK-001-15 — Exception Model

Crear errores de dominio y aplicación separados.

Ejemplo:

IdentityError
├── DomainError
├── ApplicationError
├── AuthenticationError
├── AuthorizationError
├── TenantIsolationError
├── ValidationError
└── InfrastructureError

La API posteriormente traducirá estos errores al contrato HTTP estándar.

TASK-001-16 — Dependency Injection

Preparar dependency injection para:

Database
Redis
Repositories
UnitOfWork
Security services
Event publisher
Configuration

Pero evitando acoplar Domain a FastAPI.

TASK-001-17 — Architecture Boundary Tests

Aquí comienza algo muy importante.

Crear pruebas que aseguren que la arquitectura no pueda degradarse accidentalmente.

Ejemplos:

Domain cannot import FastAPI
Domain cannot import Redis
Domain cannot import PostgreSQL adapter
Account cannot access Membership repository implementation
External domain cannot access Identity internal persistence
TASK-001-18 — Initial Observability

Preparar:

structured logging
metrics
tracing
health
readiness

No registrar:

passwords
tokens
MFA secrets
private keys
credentials
TASK-001-19 — Initial Migration Framework

Crear estructura de migrations.

apps/api/
└── migrations/

Las migrations serán versionadas.

No debemos utilizar create_all() como estrategia de producción.

TASK-001-20 — Test Foundation

Preparar:

tests/
└── domains/
    └── identity/
        ├── unit/
        ├── integration/
        ├── architecture/
        └── security/

Y configuración inicial de:

pytest;
async testing;
coverage;
fixtures;
database test environment.
TASK-001-21 — CI Foundation

El primer pipeline debe validar como mínimo:

Lint
 ↓
Type Check
 ↓
Unit Tests
 ↓
Architecture Tests
 ↓
Security Tests

Posteriormente agregaremos:

Integration
API
Mutation
Golden
Regression
Performance
Chaos

según IS correspondiente.

TASK-001-22 — Identity Public Contracts

Definir la primera frontera pública:

IdentityService
AuthorizationService
AuthenticationService

No necesariamente como microservicios.

En el Modular Monolith serán interfaces internas estables.

Esto es importante porque la arquitectura establece que las capacidades de plataforma deben exponer interfaces públicas estables y que los dominios consumidores interactúan mediante ellas.

TASK-001-23 — Architecture Documentation

Documentar:

Identity module map
Layer boundaries
Dependency rules
Public contracts
Repository boundaries
Testing boundaries

Y relacionarlo con:

ESP-0001
ADR-001–069
IS-001
TASK-001-XX
TASK-001-24 — IS-001 Validation

Antes de cerrar IS-001:

Architecture
        ✓
Runtime
        ✓
Domain structure
        ✓
Infrastructure
        ✓
Security foundation
        ✓
Testing foundation
        ✓
Observability
        ✓
CI
        ✓
Traceability
        ✓
Orden exacto de ejecución

Yo lo haría así:

PHASE 1 — Foundation

TASK-001-01  API Runtime
      ↓
TASK-001-02  Identity Package
      ↓
TASK-001-03  Layers
      ↓
TASK-001-04  Shared Kernel


PHASE 2 — Infrastructure

TASK-001-08  PostgreSQL
      ↓
TASK-001-09  Redis
      ↓
TASK-001-10  Configuration
      ↓
TASK-001-19  Migrations


PHASE 3 — Domain Infrastructure

TASK-001-06  Repositories
      ↓
TASK-001-07  Unit of Work
      ↓
TASK-001-05  Domain Events


PHASE 4 — Security

TASK-001-11  Security Foundation
      ↓
TASK-001-12  Tenant Context
      ↓
TASK-001-13  Actor Context
      ↓
TASK-001-14  Trace Context
      ↓
TASK-001-15  Exceptions


PHASE 5 — Application Infrastructure

TASK-001-16  DI
      ↓
TASK-001-22  Public Contracts


PHASE 6 — Quality

TASK-001-17  Architecture Tests
      ↓
TASK-001-18  Observability
      ↓
TASK-001-20  Test Foundation
      ↓
TASK-001-21  CI


PHASE 7 — Governance

TASK-001-23  Documentation
      ↓
TASK-001-24  Validation
Y aquí empieza realmente la programación

Una vez aprobado este desglose, el primer código que deberíamos escribir es TASK-001-01 — API Runtime Foundation, no User CRUD, no Login y no JWT todavía.

Eso es importante porque estamos siguiendo el modelo arquitectónico de EVOXA: Identity primero establece su fundación, y después vamos construyendo Account, Organization, Membership, Authentication, Authorization, etc. Identity es una capacidad central que será consumida por todos los demás dominios.

Siguiente paso

Si seguimos este orden, en el próximo paso podemos comenzar directamente con:

TASK-001-01 — API Runtime Foundation

y te voy a entregar la estructura real de archivos + código inicial de FastAPI + configuración + main.py + /api/v1 + health/readiness + manejo de errores, preparado para que lo llevemos al repositorio evoxa-platform.

No vamos a saltarnos la arquitectura ni empezar a crear código que después tengamos que rehacer.
