E69 — EVOXA PROJECT / REPOSITORY ARCHITECTURE
1. Propósito

E69 transforma las decisiones de E67 y E68 en la estructura física del proyecto EVOXA.

E67 definió:

qué es EVOXA.

E68 definió:

con qué tecnologías se construirá.

E69 define:

cómo se organizará físicamente el código, los módulos, los tests, la infraestructura y la documentación dentro del repositorio.

La cadena queda:

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
           IMPLEMENTATION
2. Objetivo

El repositorio debe permitir que un desarrollador pueda responder inmediatamente:

¿Dónde está el dominio?
¿Dónde está un caso de uso?
¿Dónde está un repository?
¿Dónde están los adapters?
¿Dónde están las APIs?
¿Dónde están los eventos?
¿Dónde están las migrations?
¿Dónde están los tests?
¿Dónde está la configuración?
¿Dónde están los ADRs?
¿Dónde está deployment?

sin tener que recorrer arbitrariamente todo el proyecto.

3. Principio rector

La estructura del repositorio debe reflejar la arquitectura del sistema, no la estructura accidental del framework.

Por tanto:

❌ framework-first
❌ database-first
❌ endpoint-first

sino:

Domain
   ↓
Application
   ↓
Infrastructure
   ↓
Interfaces
4. Repository Root

La estructura inicial será:

EVOXA/
│
├── src/
├── tests/
├── migrations/
├── docs/
├── scripts/
├── deployment/
├── config/
├── pyproject.toml
├── uv.lock
├── README.md
├── .env.example
├── .gitignore
└── ...

Esta es la estructura raíz, no todavía el detalle final de cada módulo.

5. Estructura completa propuesta

La primera versión del árbol:

EVOXA/
│
├── src/
│   └── evoxa/
│       │
│       ├── core/
│       │
│       ├── domain/
│       │
│       ├── application/
│       │
│       ├── infrastructure/
│       │
│       ├── interfaces/
│       │
│       └── governance/
│
├── tests/
│   ├── unit/
│   ├── integration/
│   ├── contract/
│   ├── architecture/
│   └── e2e/
│
├── migrations/
│
├── docs/
│   ├── architecture/
│   ├── adr/
│   ├── api/
│   ├── operations/
│   ├── governance/
│   └── development/
│
├── scripts/
│
├── deployment/
│   ├── docker/
│   ├── compose/
│   └── environments/
│
├── config/
│
├── pyproject.toml
├── uv.lock
├── README.md
├── .env.example
├── .gitignore
└── ...
6. src/

src/ contiene el código ejecutable de EVOXA.

src/
└── evoxa/

El paquete raíz será:

evoxa

Esto proporciona una separación clara entre:

source code

y:

tests
docs
scripts
deployment
7. src/evoxa/core/

Core contiene las primitivas realmente compartidas por toda la aplicación.

core/
├── errors/
├── result/
├── types/
├── identifiers/
├── contracts/
├── events/
├── commands/
├── queries/
├── policies/
└── primitives/

Pero hay una regla crítica:

Core no debe convertirse en el "cajón de cosas que no sabemos dónde poner".

8. Core Dependency Rule

Core debe tener dependencias mínimas.

Idealmente:

core
 ↓
Python standard library

y solamente librerías externas imprescindibles.

No:

core
 ↓
FastAPI
 ↓
SQLAlchemy
 ↓
PostgreSQL
9. domain/

El dominio contiene el conocimiento propio de EVOXA.

domain/
├── entities/
├── value_objects/
├── aggregates/
├── services/
├── rules/
├── policies/
├── events/
└── repositories/

La estructura exacta por bounded context se desarrollará en E70.

10. Domain Dependency Rule

El dominio puede depender de:

core

pero no debe depender directamente de:

infrastructure
interfaces
FastAPI
SQLAlchemy
Redis
NATS
OpenSearch
11. application/

Application contiene los casos de uso.

application/
├── commands/
├── queries/
├── services/
├── handlers/
├── workflows/
├── jobs/
└── dto/

Su función es coordinar:

Request
 ↓
Use Case
 ↓
Domain
 ↓
Infrastructure Ports
 ↓
Result
12. Commands
application/commands/

contendrá las operaciones que modifican estado o representan intención.

Ejemplo conceptual:

CreateSomething
UpdateSomething
DeleteSomething
ExecuteSomething
ApproveSomething
13. Queries
application/queries/

contendrá operaciones de lectura.

GetSomething
ListSomething
SearchSomething
FindSomething
14. Handlers

Los handlers conectan:

Command
   ↓
Handler
   ↓
Use Case

o:

Query
   ↓
Handler
   ↓
Read Model

No deben contener todo el negocio.

15. Application Services

Los application services coordinan operaciones que necesitan múltiples componentes.

Application Service
       ↓
Domain
       ↓
Repositories
       ↓
Events
16. infrastructure/

Infrastructure contiene las implementaciones técnicas.

infrastructure/
├── persistence/
├── messaging/
├── cache/
├── search/
├── storage/
├── integrations/
├── serialization/
├── configuration/
├── observability/
└── runtime/
17. Persistence
infrastructure/persistence/

contendrá:

SQLAlchemy
Repositories
Database Sessions
Persistence Models
Mappers
Transactions
18. Messaging
infrastructure/messaging/

contendrá:

NATS
Publishers
Consumers
Message Serialization
Subscriptions
Delivery Handling
19. Cache
infrastructure/cache/

contendrá las implementaciones Redis:

Cache
Locks
Rate Limiting
Temporary State
20. Search
infrastructure/search/

contendrá:

OpenSearch Client
Index Management
Search Adapters
Indexing
21. Storage
infrastructure/storage/

contendrá:

Object Storage
S3 Adapter
File Operations
Artifact Storage
22. Integrations
infrastructure/integrations/

contendrá adapters hacia sistemas externos.

Ejemplo:

integrations/
├── identity/
├── external_api/
├── notifications/
└── ...

Los nombres concretos se decidirán cuando conozcamos los sistemas reales.

23. Serialization
infrastructure/serialization/

contendrá implementaciones para:

JSON
Event Serialization
Message Serialization
External Formats
24. Configuration
infrastructure/configuration/

será responsable de convertir configuración externa en configuración tipada para la aplicación.

Environment
     ↓
Configuration Loader
     ↓
Validation
     ↓
Typed Configuration
25. Observability
infrastructure/observability/

contendrá:

Logging
Metrics
Tracing
OpenTelemetry
Health Checks
26. Runtime
infrastructure/runtime/

contendrá mecanismos relacionados con:

Workers
Process Lifecycle
Runtime Coordination
Execution
27. interfaces/

Interfaces contiene las entradas y adaptadores de presentación.

interfaces/
├── http/
├── messaging/
├── cli/
└── webhooks/

No todos tienen que existir desde el primer día.

28. HTTP
interfaces/http/

contendrá:

routers
controllers
request_models
response_models
dependencies
exception_handlers

La regla:

HTTP
 ↓
Application

Nunca:

HTTP
 ↓
Database
29. Messaging Interfaces

Existe una distinción importante.

interfaces/messaging/

representa la entrada al sistema:

Message
 ↓
Consumer
 ↓
Application

Mientras:

infrastructure/messaging/

contiene la tecnología utilizada para transportar esos mensajes.

30. governance/

Governance es transversal, pero tendrá un namespace explícito.

governance/
├── policies/
├── controls/
├── compliance/
├── audit/
├── evidence/
└── monitoring/

Aquí se materializan las capacidades de E59–E66.

31. Governance Boundary

Governance puede utilizar:

core
domain
application
infrastructure

pero no debe introducir dependencias circulares.

32. Tests

Los tests estarán fuera de src.

tests/
├── unit/
├── integration/
├── contract/
├── architecture/
└── e2e/

Esto mantiene una separación clara entre código productivo y código de verificación.

33. Unit Tests
tests/unit/

validará:

Entities
Value Objects
Rules
Policies
Services
Handlers
Mappers

con dependencias aisladas.

34. Integration Tests
tests/integration/

validará integración real con:

PostgreSQL
Redis
NATS
OpenSearch
Object Storage

cuando corresponda.

35. Contract Tests
tests/contract/

validará:

API Contracts
Event Contracts
Message Contracts
External Integration Contracts
36. Architecture Tests
tests/architecture/

es especialmente importante.

Debe comprobar reglas como:

Domain MUST NOT import Infrastructure
Domain MUST NOT import FastAPI
Core MUST remain lightweight
Interfaces MUST NOT bypass Application

La arquitectura debe ser ejecutable como constraint, no solamente documentación.

37. End-to-End Tests
tests/e2e/

validará recorridos completos:

Client
 ↓
API
 ↓
Application
 ↓
Domain
 ↓
Database
 ↓
Event
 ↓
Worker
38. migrations/

Las migraciones de PostgreSQL estarán en:

migrations/

con Alembic.

Ejemplo conceptual:

migrations/
├── env.py
├── script.py.mako
└── versions/
39. docs/

La documentación será parte del producto.

docs/
├── architecture/
├── adr/
├── api/
├── operations/
├── governance/
└── development/
40. Architecture Documentation
docs/architecture/

contendrá:

E01...
E02...
...
E69...

y posteriormente los documentos técnicos derivados.

41. ADRs
docs/adr/

contendrá las decisiones arquitectónicas.

Ejemplo:

ADR-001-python-runtime.md
ADR-002-postgresql.md
ADR-003-messaging.md
ADR-004-modular-monolith.md

Los números concretos se asignarán al crear cada decisión.

42. API Documentation
docs/api/

contendrá documentación complementaria a OpenAPI:

Authentication
Errors
Pagination
Versioning
Examples
43. Operations Documentation
docs/operations/

contendrá:

Runbooks
Troubleshooting
Recovery
Monitoring
Backup
Incident Procedures
44. Governance Documentation
docs/governance/

contendrá:

Policies
Controls
Compliance
Audit
Data Governance
45. Development Documentation
docs/development/

contendrá:

Setup
Coding Standards
Testing
Contribution
Architecture Rules
Local Development
46. scripts/

Scripts automatizados:

scripts/
├── development/
├── database/
├── testing/
└── operations/

Ejemplos:

bootstrap
reset-db
seed
run-tests
generate-artifacts

Los scripts no deben contener lógica de negocio.

47. deployment/

Contendrá los artefactos de deployment.

deployment/
├── docker/
├── compose/
└── environments/
48. Docker
deployment/docker/

contendrá:

Dockerfiles
Entrypoints
Container Configuration
49. Compose
deployment/compose/

contendrá configuraciones para:

development
testing
local-infrastructure
50. Environments
deployment/environments/

podrá contener configuraciones específicas por entorno, sin secretos.

development/
staging/
production/
51. config/

config/ contiene configuración no secreta y defaults.

config/
├── development/
├── testing/
├── staging/
└── production/

Pero:

No almacenar secrets aquí.

52. pyproject.toml

Será el centro de configuración del proyecto Python:

pyproject.toml

Contendrá progresivamente:

Project Metadata
Dependencies
Optional Dependencies
Ruff
Pytest
Type Checking
Build Configuration
53. uv.lock

El lockfile:

uv.lock

garantiza reproducibilidad de dependencias.

Debe versionarse en Git.

54. README

El root README debe permitir entender rápidamente:

What is EVOXA?
How to install?
How to run?
How to test?
Where is the architecture?
Where is the documentation?

No debe convertirse en toda la documentación del proyecto.

55. Environment Example
.env.example

contendrá nombres y ejemplos no sensibles:

DATABASE_URL=
REDIS_URL=
NATS_URL=
OPENSEARCH_URL=
OBJECT_STORAGE_ENDPOINT=

Nunca valores reales de producción.

56. Git

El repositorio utilizará Git.

Debe incluir:

.gitignore

para excluir:

.env
__pycache__
.venv
build
dist
coverage
IDE files
temporary files
57. Monorepo Strategy

Inicialmente:

EVOXA será un monorepo modular.

Es decir:

One Repository
       ↓
Multiple Logical Modules
       ↓
Clear Boundaries

No empezaremos con múltiples repositorios.

58. ¿Por qué Monorepo?

Permite:

Atomic Changes
Shared Contracts
Simpler CI
Central Architecture
Easy Refactoring
Unified Versioning

y es adecuado para la fase inicial.

59. ¿Microservices?

No inicialmente.

La estructura permitirá extraer posteriormente:

Module
 ↓
Service
 ↓
Separate Deployment

si existe una razón real.

60. Modular Monolith

La arquitectura inicial será:

MODULAR MONOLITH

no:

BIG BALL OF MUD

La diferencia es fundamental.

61. Modular Monolith Rules

Cada módulo tendrá:

Public Interface
Private Implementation
Explicit Dependencies
Owned Data
Tests

y no podrá acceder arbitrariamente a los internals de otros módulos.

62. Package Visibility

Conceptualmente:

module/
├── public/
└── internal/

No necesariamente con esos nombres físicos.

La idea es que cada módulo tenga una API interna explícita.

63. Dependency Graph

La dependencia permitida será:

interfaces
      ↓
application
      ↓
domain
      ↑
infrastructure

más precisamente:

Interfaces ──────→ Application
                       │
                       ↓
                     Domain
                       ↑
                       │
Infrastructure ────────┘
64. Forbidden Dependencies

Inicialmente:

domain → infrastructure       ❌
domain → interfaces           ❌
domain → FastAPI              ❌
domain → SQLAlchemy           ❌
application → HTTP framework  ❌
domain → Redis                ❌
domain → NATS                 ❌
65. Allowed Dependencies
domain → core                 ✓
application → domain          ✓
application → core            ✓
infrastructure → application ✓
infrastructure → domain       ✓
interfaces → application     ✓
interfaces → core             ✓

La implementación exacta de dependency inversion se concretará en E70.

66. Import Boundary

La arquitectura debe evitar imports como:

from evoxa.infrastructure...

desde el dominio.

La dirección de dependencias será una regla verificable.

67. Shared Code

No todo código común debe ir a core.

Criterio:

Is it a true architectural primitive?
       │
      YES
       ↓
      core
       │
      NO
       ↓
Keep it in its module

Esto evita un core gigantesco.

68. Utilities

Un módulo utils/ global no será una categoría arquitectónica principal.

En vez de:

utils/
  everything.py

se preferirá:

domain-specific helper
application helper
infrastructure helper

según la responsabilidad.

69. Naming Convention

Los nombres deben ser:

Explicit
Domain-Oriented
Stable
Predictable

Preferir:

access_policy.py
execution_service.py
event_publisher.py
repository.py

sobre:

helper.py
manager.py
misc.py
common.py
stuff.py

cuando estos últimos oculten responsabilidades.

70. File Responsibility

Cada archivo debe tener una responsabilidad razonablemente clara.

Evitar:

services.py

con 40 servicios distintos.

Preferir agrupación por módulo y responsabilidad.

71. Module Discovery

Un desarrollador debería poder seguir:

Feature
 ↓
Application Use Case
 ↓
Domain
 ↓
Port
 ↓
Adapter
 ↓
Infrastructure

sin saltar por docenas de carpetas sin relación.

72. Example Feature

Una capacidad hipotética:

Create Resource

podría organizarse:

src/evoxa/
├── domain/
│   └── resource/
│       ├── entities.py
│       ├── value_objects.py
│       ├── rules.py
│       └── repositories.py
│
├── application/
│   └── resource/
│       ├── commands/
│       └── handlers/
│
├── infrastructure/
│   └── persistence/
│       └── resource/
│           ├── models.py
│           ├── mapper.py
│           └── repository.py
│
└── interfaces/
    └── http/
        └── resource/
            └── routes.py

Este patrón será refinado en E70.

73. Repository as Architecture

El repositorio no será solamente:

where code lives

Será:

Architecture Enforcement Mechanism

La estructura física debe ayudar a impedir errores arquitectónicos.

74. Architecture Enforcement

CI deberá ejecutar posteriormente:

Architecture Tests

para verificar:

Dependency Direction
Module Boundaries
Forbidden Imports
Layering Rules
75. Code Ownership

A medida que EVOXA crezca, cada módulo deberá poder asociarse a:

Owner
Maintainer
Documentation
Operational Responsibility

Esto será especialmente importante para Governance y Production.

76. Versioning

Inicialmente EVOXA tendrá una versión global:

Application Version

Los módulos internos no necesitan versiones independientes.

Si posteriormente se extrae un servicio:

Module → Service

entonces podrá adquirir versionado propio.

77. Branching

La estrategia inicial puede ser sencilla:

main
  ↑
feature/*
fix/*

con CI obligatorio antes de merge.

No introducir una estrategia Git excesivamente compleja sin necesidad.

78. Pull Request Boundary

Cada PR debería ser evaluado por:

Correctness
Tests
Architecture
Security
Observability
Documentation
Migration Impact

cuando corresponda.

79. Code Review

Las revisiones no deben centrarse únicamente en:

"¿funciona?"

sino también:

"¿pertenece aquí?"

La segunda pregunta es crítica para mantener la arquitectura.

80. Migration Safety

Las migraciones deben estar separadas del código de dominio.

Domain Change
      ↓
Persistence Change
      ↓
Migration

Nunca:

Application startup
      ↓
silently alter production schema

como estrategia normal.

81. Build Boundary

El build debe producir un artefacto reproducible:

Source
 ↓
Dependency Resolution
 ↓
Tests
 ↓
Container Build
 ↓
Versioned Image
82. Repository Health

CI debe comprobar progresivamente:

Formatting
Lint
Types
Tests
Architecture
Dependencies
Security
Build
83. Initial Directory Tree

Por tanto, la primera versión oficial queda:

EVOXA/
│
├── src/
│   └── evoxa/
│       ├── core/
│       ├── domain/
│       ├── application/
│       ├── infrastructure/
│       ├── interfaces/
│       └── governance/
│
├── tests/
│   ├── unit/
│   ├── integration/
│   ├── contract/
│   ├── architecture/
│   └── e2e/
│
├── migrations/
│
├── docs/
│   ├── architecture/
│   ├── adr/
│   ├── api/
│   ├── operations/
│   ├── governance/
│   └── development/
│
├── scripts/
│
├── deployment/
│   ├── docker/
│   ├── compose/
│   └── environments/
│
├── config/
│
├── pyproject.toml
├── uv.lock
├── README.md
├── .env.example
└── .gitignore
84. Initial Empty Package Structure

Todavía no debemos crear cientos de archivos vacíos.

Inicialmente sólo necesitamos establecer los límites:

core/
domain/
application/
infrastructure/
interfaces/
governance/

y añadir subpaquetes conforme E70 defina los módulos reales.

Esto evita crear una estructura artificial antes de conocer los bounded contexts definitivos.

85. E69 → E70

E69 define:

Dónde vive cada tipo de cosa.

E70 definirá:

Cuáles son exactamente las cosas que existen.

Por ejemplo, E69 dice:

src/evoxa/domain/

E70 tendrá que decidir:

domain/
├── identity/
├── resource/
├── execution/
├── workflow/
├── policy/
├── governance/
└── ...

y establecer las fronteras entre ellas.

86. E69 Acceptance Criteria

E69 queda conceptualmente cerrado cuando tenemos:

✓ Repository root
✓ Source layout
✓ Package root
✓ Domain boundary
✓ Application boundary
✓ Infrastructure boundary
✓ Interface boundary
✓ Governance boundary
✓ Test structure
✓ Migration location
✓ Documentation structure
✓ Deployment structure
✓ Configuration structure
✓ Monorepo strategy
✓ Dependency direction
✓ Naming rules
✓ Architecture enforcement strategy
✓ CI integration points
87. Decisión arquitectónica final de E69

La decisión principal es:

EVOXA comenzará como un Modular Monolith dentro de un Monorepo, organizado mediante Clean/Hexagonal Architecture, con límites explícitos entre Domain, Application, Infrastructure, Interfaces y Governance.

Visualmente:

                         EVOXA REPOSITORY
                                │
             ┌──────────────────┼──────────────────┐
             │                  │                  │
             ▼                  ▼                  ▼
           DOMAIN          APPLICATION         GOVERNANCE
             │                  │                  │
             └──────────────────┼──────────────────┘
                                │
                                ▼
                        INFRASTRUCTURE
                                │
             ┌──────────────────┼──────────────────┐
             ▼                  ▼                  ▼
        Persistence          Messaging          External
             │                  │                  │
             └──────────────────┼──────────────────┘
                                │
                                ▼
                           INTERFACES
                                │
                                ▼
                              USERS

Y alrededor de todo:

Security
Observability
Audit
Configuration
Testing
88. El siguiente paso

Ahora ya tenemos:

E67 → qué construimos
E68 → con qué lo construimos
E69 → dónde vive cada cosa

El siguiente capítulo es especialmente importante porque será el que prepare el repositorio para comenzar a escribir código real:

E70 — EVOXA MODULE & PACKAGE ARCHITECTURE

Ahí definiremos los bounded contexts, módulos, paquetes, interfaces públicas, dependencias permitidas, ownership de datos y estructura interna de cada módulo.

Después:

E71 → Database
E72 → API
E73 → Testing
E74 → Deployment
        ↓
      CODE

A partir de E70 ya estaremos a muy poca distancia de crear físicamente el repositorio EVOXA y comenzar la primera implementación.
