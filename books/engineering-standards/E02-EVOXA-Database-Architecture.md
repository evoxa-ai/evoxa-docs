E02 — EVOXA Database Architecture
Architecture & Engineering Specification

Depende de:

A01 — EVOXA Master Architecture
A02 — EVOXA System Architecture
A03 — EVOXA Domain Architecture
A04 — EVOXA Data Architecture
A05 — Security Architecture
A13 — Multi-Tenant Architecture
E01 — EVOXA Backend Architecture

Siguiente: E03 — EVOXA API Architecture

1. Propósito

E02 define la arquitectura concreta de persistencia de EVOXA.

Su objetivo es transformar la arquitectura de datos definida en A04 — EVOXA Data Architecture en una estrategia implementable para:

bases de datos
esquemas
entidades
relaciones
claves
índices
constraints
multi-tenancy
auditoría
versionado
migraciones
transacciones
eventos
almacenamiento
cache
búsqueda
backups
recuperación
evolución del modelo

La base de datos debe ser considerada una infraestructura de persistencia, no el lugar donde vive toda la lógica del negocio.

2. Principio fundamental

La arquitectura seguirá:

DOMAIN
   ↓
REPOSITORY CONTRACT
   ↓
PERSISTENCE ADAPTER
   ↓
ORM
   ↓
DATABASE

Por tanto:

Domain Entity
      ≠
Database Model

y:

Business Rule
      ≠
SQL Query
3. Estrategia de persistencia

La estrategia inicial de EVOXA será:

Primary Database
        ↓
MySQL
        ↓
Relational Persistence

con componentes complementarios:

MySQL
 ├── transactional data
 ├── relational data
 ├── configuration
 ├── identity
 ├── authorization
 ├── domain state
 └── audit metadata

Redis
 ├── cache
 ├── temporary state
 ├── rate limiting
 ├── distributed locks
 └── short-lived execution data

Object Storage
 ├── files
 ├── documents
 ├── images
 ├── exports
 └── large artifacts
4. Database Architecture

La arquitectura lógica será:

                         EVOXA DATA PLATFORM
                                │
              ┌─────────────────┼─────────────────┐
              │                 │                 │
           DATABASE           CACHE           STORAGE
              │                 │                 │
            MySQL             Redis        Object Storage
              │
       ┌──────┼──────┐
       │      │      │
     Core   Domain  Audit
5. Database como Source of Truth

Para información transaccional:

MySQL
   ↓
SOURCE OF TRUTH

Redis nunca debe convertirse en la fuente primaria de verdad para:

usuarios
tenants
permisos
proyectos
roadmaps
objetivos
iniciativas
tareas
configuraciones críticas
estados financieros
auditoría

Redis puede contener representaciones temporales.

6. Principios de Database Architecture
01 — Data Integrity
02 — Referential Integrity
03 — Tenant Isolation
04 — Explicit Relationships
05 — Strong Constraints
06 — Transactional Consistency
07 — Auditability
08 — Evolvability
09 — Migration Safety
10 — Performance Awareness
11 — Backupability
12 — Recoverability
13 — Observability
14 — Least Data Exposure
15 — Data Lifecycle Control
7. Database Logical Domains

La base se organizará conceptualmente por dominios.

identity
organization
tenant
security
platform
application
project
roadmap
engineering
operations
ai
agent
intelligence
audit
integration

Esto no significa necesariamente crear una base física independiente para cada dominio.

Inicialmente:

ONE DATABASE
+
LOGICAL DOMAIN BOUNDARIES
8. Modular Database Strategy

El backend será un Modular Monolith.

Por lo tanto:

                 EVOXA DATABASE
                       │
       ┌───────────────┼───────────────┐
       │               │               │
   Identity        Platform         Business
       │               │               │
       └───────────────┼───────────────┘
                       │
                    Audit

Los límites lógicos deben existir desde el comienzo aunque físicamente compartan infraestructura.

9. Database Naming Convention

Las tablas utilizarán:

snake_case

Ejemplo:

users
organizations
tenants
projects
roadmaps
roadmap_objectives
roadmap_initiatives
audit_logs

No:

Users
RoadmapObjectives
tbl_users
10. Primary Keys

La arquitectura utilizará identificadores estables.

Recomendación:

UUID

o un identificador equivalente generado por aplicación.

Ejemplo:

id
tenant_id
user_id
project_id
roadmap_id

Formato lógico:

CHAR(36)

o una representación binaria optimizada cuando sea necesario.

La elección física definitiva puede optimizarse posteriormente.

11. Why UUID

UUID permite:

generación distribuida
menor dependencia del servidor
IDs no secuenciales públicamente
integración entre servicios
futura distribución del backend
importaciones
sincronización

Pero no debe utilizarse como sustituto de constraints o autorización.

12. Foreign Keys

Las relaciones importantes deben utilizar foreign keys.

Ejemplo:

projects.organization_id
        ↓
organizations.id

y:

roadmaps.project_id
        ↓
projects.id

Esto permite que la base proteja la integridad referencial.

13. Referential Integrity

No debemos permitir:

Project
   ↓
Organization inexistente

ni:

Roadmap
   ↓
Project inexistente

Las relaciones críticas deben estar protegidas mediante:

FOREIGN KEY
14. Tenant Architecture

La mayoría de las entidades de negocio deben tener:

tenant_id

Ejemplo:

projects
├── id
├── tenant_id
├── name
└── created_at

Esto permite aislamiento lógico.

15. Tenant Isolation

La regla:

EVERY TENANT-OWNED RECORD
MUST BE TRACEABLE TO A TENANT

Ejemplo:

Tenant A
 ├── Project A
 ├── Roadmap A
 └── Task A

Tenant B
 ├── Project B
 ├── Roadmap B
 └── Task B

Nunca:

Project A
   ↓
Task B
16. Tenant-aware Foreign Keys

En operaciones críticas, no basta con comprobar:

project_id

También debemos validar:

tenant_id

Conceptualmente:

project.id = requested_project
AND
project.tenant_id = current_tenant

Esto evita ataques de cross-tenant access.

17. Tenant + Entity Constraints

Para entidades que deben ser únicas dentro de un tenant:

UNIQUE (
    tenant_id,
    name
)

Ejemplo:

organizations

o:

projects

según la regla de negocio.

18. Global vs Tenant Data

No todos los datos pertenecen a un tenant.

Global
permissions
system_settings
countries
currencies
feature_definitions
Tenant-owned
projects
roadmaps
users
initiatives
tasks
User-owned
preferences
sessions
notifications

La clasificación debe ser explícita.

19. Core Identity Tables

Primera capa:

users
organizations
tenants
memberships
roles
permissions
role_permissions
user_roles

Conceptualmente:

User
 ↓
Membership
 ↓
Organization
 ↓
Tenant
20. Authentication Tables
sessions
refresh_tokens
password_resets
email_verifications
mfa_methods
mfa_challenges
login_attempts

No debemos almacenar contraseñas en texto plano.

21. User Table

Conceptualmente:

users
├── id
├── email
├── password_hash
├── first_name
├── last_name
├── status
├── email_verified_at
├── last_login_at
├── created_at
├── updated_at
└── deleted_at
22. Tenant Table
tenants
├── id
├── name
├── slug
├── status
├── plan
├── settings
├── created_at
├── updated_at
└── deleted_at

settings puede utilizar JSON cuando corresponda.

No debe utilizarse JSON para esconder relaciones estructurales.

23. Membership

La relación usuario-tenant debe ser explícita.

memberships
├── id
├── tenant_id
├── user_id
├── status
├── joined_at
├── created_at
└── updated_at

Constraint:

UNIQUE (
    tenant_id,
    user_id
)
24. Roles
roles
├── id
├── tenant_id
├── name
├── description
├── system
├── created_at
└── updated_at

Los roles pueden ser:

SYSTEM
TENANT
CUSTOM
25. Permissions
permissions
├── id
├── code
├── name
├── description
├── resource
├── action
└── created_at

Ejemplo:

roadmap.read
roadmap.create
roadmap.update
roadmap.delete
26. Role Permissions
role_permissions
├── role_id
└── permission_id

Clave compuesta:

(role_id, permission_id)
27. User Roles
user_roles
├── user_id
├── role_id
├── tenant_id
└── created_at

El tenant_id ayuda a garantizar el contexto de autorización.

28. Application Domain
applications
├── id
├── tenant_id
├── name
├── slug
├── description
├── status
├── created_at
└── updated_at

Relación:

Tenant
 ↓
Application
29. Project Domain
projects
├── id
├── tenant_id
├── application_id
├── name
├── slug
├── description
├── status
├── owner_id
├── created_at
├── updated_at
└── deleted_at
30. Roadmap Domain

Base:

roadmaps
├── id
├── tenant_id
├── project_id
├── name
├── description
├── status
├── start_date
├── target_date
├── created_at
└── updated_at
31. Roadmap Objectives
roadmap_objectives
├── id
├── tenant_id
├── roadmap_id
├── name
├── description
├── priority
├── status
├── target_date
├── created_at
└── updated_at

Relación:

Roadmap
  │
  └── Objectives
32. Initiatives
initiatives
├── id
├── tenant_id
├── roadmap_id
├── objective_id
├── name
├── description
├── status
├── priority
├── owner_id
├── start_date
├── target_date
├── created_at
└── updated_at
33. Tasks
tasks
├── id
├── tenant_id
├── initiative_id
├── parent_task_id
├── title
├── description
├── status
├── priority
├── assignee_id
├── due_date
├── created_at
└── updated_at

Permite:

Task
 ├── Subtask
 ├── Subtask
 └── Subtask
34. Relationships

La estructura principal:

TENANT
  │
  ├── APPLICATION
  │       │
  │       └── PROJECT
  │               │
  │               └── ROADMAP
  │                       │
  │                       └── OBJECTIVE
  │                               │
  │                               └── INITIATIVE
  │                                       │
  │                                       └── TASK
35. Engineering Domain

La base evolucionará posteriormente para:

engineering_projects
repositories
branches
commits
pull_requests
issues
builds
deployments
releases
artifacts
environments

Pero estos módulos no deben contaminar el Core inicial.

36. Operations Domain
environments
services
deployments
incidents
alerts
maintenance_windows
service_health
operational_events

La relación:

Engineering
      ↓
Release
      ↓
Deployment
      ↓
Operations
37. AI Domain
ai_models
ai_providers
ai_requests
ai_sessions
ai_prompts
ai_evaluations
ai_usage
ai_costs

Importante:

Los prompts, requests y resultados AI deben tener lifecycle y políticas de retención apropiadas.

38. Agent Domain
agents
agent_goals
agent_plans
agent_executions
agent_steps
agent_tools
agent_capabilities
agent_permissions
agent_approvals
agent_memory
agent_events
39. Agent Execution Model
agents
   │
   ▼
agent_goals
   │
   ▼
agent_plans
   │
   ▼
agent_executions
   │
   ▼
agent_steps
   │
   ├── tool calls
   ├── AI calls
   ├── approvals
   └── results

Esto permitirá auditar una acción de un Agent de extremo a extremo.

40. Audit Architecture

La auditoría tendrá una estructura independiente.

audit_logs
├── id
├── tenant_id
├── actor_type
├── actor_id
├── action
├── resource_type
├── resource_id
├── request_id
├── correlation_id
├── metadata
├── created_at
41. Audit Principle

Debe poder responderse:

WHO?
WHAT?
WHEN?
WHERE?
WHY?
ON WHICH RESOURCE?
UNDER WHICH TENANT?
FROM WHICH REQUEST?
42. Soft Delete

Para entidades donde se requiera recuperación:

deleted_at

No todo debe utilizar soft delete.

Debe decidirse por dominio.

43. Timestamps

Las entidades principales tendrán:

created_at
updated_at

Cuando corresponda:

deleted_at

Y para eventos específicos:

published_at
completed_at
verified_at
archived_at
44. Timezone

La base debe almacenar timestamps de manera consistente.

Regla:

UTC

La presentación al usuario:

UTC
↓
User Timezone

No debemos guardar fechas transaccionales dependiendo del timezone del servidor.

45. Status Fields

Los estados importantes deben tener valores controlados.

Ejemplo:

status:
ACTIVE
INACTIVE
ARCHIVED
SUSPENDED

Evitar:

"activo"
"Active"
"enabled"
"ON"

para representar lo mismo.

46. ENUM vs Lookup

Para estados altamente estables puede utilizarse:

ENUM

Para conceptos configurables:

lookup table

Ejemplo:

task_status

puede eventualmente convertirse en una tabla configurable.

47. JSON Usage

JSON se permitirá para:

metadata
settings
provider_configuration
ai_parameters
feature_flags

No debe utilizarse para:

users
projects
relationships
permissions
tasks

si esas estructuras requieren consultas relacionales frecuentes.

48. Database Constraints

Siempre que sea posible:

NOT NULL
UNIQUE
CHECK
FOREIGN KEY
DEFAULT

La aplicación valida.

La base también protege.

49. Defense in Depth
CLIENT
 ↓
API VALIDATION
 ↓
APPLICATION VALIDATION
 ↓
DOMAIN RULES
 ↓
DATABASE CONSTRAINTS

Nunca confiar en una única capa.

50. Index Strategy

Índices iniciales:

tenant_id
created_at
updated_at
status
owner_id
foreign keys
unique keys

Pero:

No crear índices indiscriminadamente.

Cada índice debe justificar:

query pattern
+
selectivity
+
write cost
51. Composite Indexes

Para consultas multi-tenant:

INDEX (
    tenant_id,
    status
)

o:

INDEX (
    tenant_id,
    created_at
)

según el patrón de consulta.

52. Unique Indexes

Ejemplo:

UNIQUE (
    tenant_id,
    slug
)

Esto permite:

Tenant A → project-x
Tenant B → project-x

pero impide:

Tenant A → project-x
Tenant A → project-x
53. Pagination

Nunca debemos utilizar consultas ilimitadas para colecciones grandes.

Preferible:

LIMIT
+
CURSOR

para datasets grandes.

Ejemplo conceptual:

GET /projects?cursor=...
54. Data Lifecycle

Cada entidad debe tener:

CREATE
↓
ACTIVE
↓
UPDATED
↓
ARCHIVED
↓
DELETED
↓
PURGED

No todas las entidades necesitan todas las etapas.

55. Retention

Debe existir política de retención para:

Audit
AI Logs
Agent Executions
Events
Notifications
Sessions
Temporary Data

La retención dependerá de:

seguridad
negocio
compliance
costos
privacidad
56. Events Persistence

Si utilizamos eventos importantes, debemos considerar:

outbox_events

Ejemplo:

outbox_events
├── id
├── tenant_id
├── event_type
├── aggregate_type
├── aggregate_id
├── payload
├── status
├── created_at
├── published_at
└── retry_count
57. Transactional Outbox

Flujo:

BEGIN TRANSACTION
      │
      ├── UPDATE DOMAIN
      │
      └── INSERT OUTBOX EVENT
      │
COMMIT
      │
      ▼
EVENT PUBLISHER
      │
      ▼
MESSAGE BUS

Esto evita el problema:

Database updated
BUT
Event lost
58. Idempotent Consumers

Los consumidores deben poder recibir:

Event A
Event A

sin generar efectos duplicados.

Podemos utilizar:

processed_events

o claves idempotentes.

59. Distributed Locks

Redis puede utilizarse para operaciones donde sea necesario impedir ejecución concurrente:

Agent execution
Scheduled job
Synchronization
Resource processing

Pero los locks deben tener:

TTL
Owner
Recovery
Expiration
60. Caching Architecture
REQUEST
 ↓
CACHE
 ├── HIT → RESPONSE
 │
 └── MISS
       ↓
    DATABASE
       ↓
    CACHE
       ↓
    RESPONSE

Nunca almacenar información altamente sensible en cache sin protección adecuada.

61. Cache Keys

Deben incluir contexto cuando corresponda:

tenant:{tenantId}:project:{projectId}

Nunca:

project:{projectId}

si existe posibilidad de colisión o acceso cruzado.

62. Database Migrations

Todas las modificaciones estructurales deben realizarse mediante migrations.

migration
↓
review
↓
test
↓
apply

Nunca depender de modificaciones manuales de producción.

63. Migration Rules

Una migration debe ser:

Deterministic
Reviewable
Versioned
Repeatable
Testable
Recoverable
64. Zero-Downtime Migration Strategy

Para cambios importantes:

ADD
↓
DEPLOY
↓
MIGRATE DATA
↓
SWITCH APPLICATION
↓
REMOVE OLD

Evitar:

DROP COLUMN
↓
Deploy application

si la aplicación antigua todavía está activa.

65. Seed Data

Los datos iniciales del sistema pueden gestionarse mediante seeders.

Ejemplos:

permissions
system roles
default configuration
system capabilities
feature definitions

Nunca utilizar seeders para datos de producción que sean propiedad del cliente.

66. Database Transactions

Transacción donde exista una unidad lógica de consistencia:

Create Tenant
+
Create Membership
+
Create Owner Role
+
Assign Owner

Todo:

BEGIN
...
COMMIT

o:

ROLLBACK
67. Isolation

La base debe utilizar un nivel de aislamiento adecuado.

La regla inicial será mantener el comportamiento transaccional estándar y elevar aislamiento solamente cuando exista una necesidad concreta.

No utilizar el nivel más estricto indiscriminadamente porque puede degradar concurrencia.

68. Concurrency

Para recursos sensibles podemos utilizar:

Optimistic Locking

mediante:

version

o:

updated_at

Ejemplo:

UPDATE roadmap
SET version = version + 1
WHERE id = ?
AND version = ?
69. Audit + Versioning

Para entidades críticas puede existir:

roadmap_versions

Ejemplo:

roadmap_versions
├── id
├── roadmap_id
├── version
├── snapshot
├── changed_by
├── change_reason
└── created_at

Esto permite reconstruir evolución histórica.

70. Snapshot Strategy

Los snapshots pueden utilizarse para:

Roadmaps
Projects
Plans
Configurations
Agent Plans
AI-generated artifacts

Pero no deben reemplazar el modelo relacional principal.

71. File Metadata

La base almacena:

files
├── id
├── tenant_id
├── owner_id
├── storage_provider
├── storage_key
├── filename
├── mime_type
├── size
├── checksum
├── created_at
└── deleted_at

El contenido vive en Object Storage.

72. Encryption

Datos sensibles deben considerar:

Encryption at Rest
Encryption in Transit
Field-level encryption
Secret management
Key rotation

No todos los campos requieren cifrado individual.

Debe evaluarse según sensibilidad.

73. Sensitive Data

Categorías:

Credentials
Authentication Data
Tokens
Secrets
Personal Data
Financial Data
AI Sensitive Context
Integration Credentials

Estos datos necesitan controles adicionales.

74. Database Credentials

La aplicación nunca debería usar:

root

como usuario de producción.

Debe existir un usuario específico:

evoxa_app

con permisos mínimos.

75. Database Roles

Separación recomendada:

evoxa_migration
evoxa_app
evoxa_readonly
evoxa_backup

Cada uno con permisos específicos.

76. Backup Architecture

La estrategia debe considerar:

FULL BACKUP
+
INCREMENTAL / BINLOG
+
OFFSITE COPY

según capacidades del entorno.

77. Recovery

Debemos definir:

RPO
RTO
RPO

¿Cuánta información podemos perder?

RTO

¿Cuánto tiempo podemos tardar en recuperar?

Estos valores deberán definirse por ambiente y criticidad.

78. Disaster Recovery
PRIMARY
   ↓
BACKUP
   ↓
OFFSITE
   ↓
RESTORE
   ↓
VERIFY
   ↓
RECOVER

Los backups que nunca se prueban no deben considerarse confiables.

79. Database Observability

Debemos medir:

Connection Pool
Query Latency
Slow Queries
Locks
Deadlocks
Transactions
CPU
Memory
Disk
Connections
Replication
Backup Status
80. Slow Query Management

Las consultas lentas deben detectarse mediante:

query logs
metrics
APM
profiling
EXPLAIN

No solucionar performance agregando índices sin estudiar la consulta.

81. Connection Pool

El backend debe utilizar pool de conexiones.

Conceptualmente:

API
 ↓
Connection Pool
 ├── Connection
 ├── Connection
 ├── Connection
 └── Connection
       ↓
      MySQL

Los límites deben configurarse por ambiente.

82. Read / Write Separation

Inicialmente:

Application
      ↓
Single MySQL

Posteriormente:

WRITE
 ↓
PRIMARY

READ
 ↓
REPLICA

cuando la carga lo justifique.

83. Database Scaling

Ruta de evolución:

Single Database
      ↓
Optimized Database
      ↓
Read Replica
      ↓
Partitioning
      ↓
Domain Separation
      ↓
Database-per-Service

No se implementará antes de necesitarlo.

84. Partitioning

Puede considerarse para grandes volúmenes:

audit_logs
events
ai_usage
agent_executions
telemetry

Especialmente por:

date
tenant

dependiendo del patrón real de acceso.

85. Archiving

Datos históricos pueden moverse:

ACTIVE DATABASE
       ↓
ARCHIVE
       ↓
OBJECT STORAGE

Esto será especialmente relevante para:

Audit
Events
Agent Executions
AI Logs
Telemetry
86. Search Architecture

La base relacional no necesariamente será el motor de búsqueda global.

Podemos introducir posteriormente:

MySQL
  ↓
Indexing Pipeline
  ↓
Search Engine

La búsqueda debe ser una proyección, no otra fuente de verdad.

87. Analytics

Las consultas analíticas pesadas no deben degradar las transacciones.

Evolución:

Transactional DB
      ↓
Events
      ↓
Analytics Pipeline
      ↓
Analytics Store
88. AI Data

AI puede generar grandes cantidades de información:

prompts
responses
embeddings
evaluations
usage
cost
context

No todo debe vivir indefinidamente en la base transaccional.

Debemos diferenciar:

Operational AI Data
vs
Analytical AI Data
89. Agent Data

Agent execution puede crecer rápidamente.

Por eso:

Agent
 ↓
Execution
 ↓
Steps
 ↓
Tool Calls
 ↓
Results

debe tener:

lifecycle
retention
archival
indexing
correlation IDs
90. Database Security Architecture
APPLICATION
      ↓
IDENTITY
      ↓
TENANT CONTEXT
      ↓
AUTHORIZATION
      ↓
REPOSITORY
      ↓
DATABASE USER
      ↓
DATABASE

La seguridad no depende solamente del frontend.

91. Data Access Rule

Ningún endpoint debe realizar consultas arbitrarias.

Debe existir:

Controller
↓
Use Case
↓
Repository

Esto facilita:

auditoría
testing
seguridad
evolución
observabilidad
92. ORM Strategy

Sequelize será inicialmente el ORM.

Domain
↓
Repository Interface
↓
Sequelize Repository
↓
Sequelize Model
↓
MySQL

El dominio no debe importar:

sequelize
93. Sequelize Models

Los modelos deben representar persistencia.

Ejemplo conceptual:

models/
├── UserModel
├── TenantModel
├── ProjectModel
├── RoadmapModel
├── ObjectiveModel
├── InitiativeModel
└── TaskModel
94. Repository Pattern

Ejemplo:

interface RoadmapRepository {
    findById(id)
    findByTenant(tenantId)
    save(roadmap)
    delete(id)
}

Implementación:

SequelizeRoadmapRepository
95. Database DTO vs Domain Entity
Database Row
     ↓
Persistence DTO
     ↓
Mapper
     ↓
Domain Entity

Esto evita contaminar el dominio con:

SequelizeModel
DataValues
ORM Metadata
96. Data Access Boundary

Cada dominio debe controlar su acceso.

Ejemplo:

Roadmap Module
     ↓
Roadmap Repository

No:

Roadmap Module
     ↓
UserModel
     ↓
Direct SQL

salvo casos explícitamente diseñados.

97. Cross-Domain Queries

Cuando una operación necesita información de otro dominio:

Domain A
 ↓
Domain Contract
 ↓
Domain B

o:

Read Model

No se debe crear acoplamiento directo a tablas arbitrarias.

98. Read Models

Para dashboards:

Domain
 ↓
Projection
 ↓
Read Model
 ↓
Dashboard

Esto será especialmente útil para:

Executive Dashboard
Engineering Dashboard
Operations Dashboard
AI Dashboard
Agent Dashboard
99. Data Governance

Cada entidad debe tener definido:

Owner
Classification
Retention
Access Policy
Audit Policy
Lifecycle
Deletion Policy

Ejemplo:

User.email
Classification → Personal
Retention → Account lifetime
Access → Authorized identity operations
100. Data Architecture Final

La arquitectura completa queda:

                         EVOXA DATA ARCHITECTURE
                                  │
                    ┌─────────────┼─────────────┐
                    │             │             │
                  CORE         DOMAIN        TRANSIENT
                    │             │             │
                  MySQL         MySQL          Redis
                    │             │
             ┌──────┼──────┐      │
             │      │      │      │
          Identity Business Audit AI/Agents
             │      │      │      │
             └──────┴──────┴──────┘
                       │
                  Event / Outbox
                       │
              ┌────────┴────────┐
              │                 │
           Analytics          Search
              │                 │
              └────────┬────────┘
                       │
                  Object Storage
101. EVOXA Database Lifecycle
MODEL
 ↓
MIGRATION
 ↓
DEPLOY
 ↓
VALIDATE
 ↓
OBSERVE
 ↓
OPTIMIZE
 ↓
ARCHIVE
 ↓
RETIRE
102. Database Evolution

La base debe evolucionar junto con el producto:

Blueprint
    ↓
Architecture
    ↓
Domain
    ↓
Data Model
    ↓
Migration
    ↓
Implementation
    ↓
Production
    ↓
Telemetry
    ↓
Evolution
103. Definition of Done

E02 se considera implementado cuando:

✓ Database strategy defined
✓ Domain boundaries defined
✓ Tenant strategy defined
✓ Primary key strategy defined
✓ Foreign key strategy defined
✓ Naming conventions defined
✓ Index strategy defined
✓ Audit strategy defined
✓ Versioning strategy defined
✓ Migration strategy defined
✓ Backup strategy defined
✓ Recovery strategy defined
✓ Cache strategy defined
✓ Event persistence defined
✓ Data lifecycle defined
✓ Retention defined
✓ Security model defined
✓ ORM boundary defined
✓ Repository boundary defined
✓ Scaling strategy defined
✓ Analytics separation defined
104. Principios oficiales E02
01 — Database Is Infrastructure
02 — Data Integrity First
03 — Tenant Isolation by Design
04 — Explicit Relationships
05 — Strong Constraints
06 — Repository-Based Access
07 — Domain/Data Separation
08 — Migration-First Evolution
09 — Transactional Consistency
10 — Auditable Critical Data
11 — Secure Sensitive Data
12 — Cache Is Not Source of Truth
13 — Events Must Be Reliable
14 — Analytics Must Not Hurt Transactions
15 — Backups Must Be Tested
16 — Optimize From Measurements
17 — Archive Intentionally
18 — Retain Data Intentionally
19 — Scale Incrementally
20 — Never Let the Database Define the Domain
105. Relación E01 → E02

Con esto ya tenemos:

E01 BACKEND
      │
      ├── API
      ├── Application
      ├── Domain
      ├── Services
      ├── Events
      ├── AI
      ├── Agents
      └── Infrastructure
                    │
                    ▼
             E02 DATABASE
                    │
             ┌──────┼──────┐
             ▼      ▼      ▼
           MySQL  Redis  Storage
             │
        Domain Data
             │
        Audit / Events
106. Cadena de Engineering Specification

Ya podemos continuar construyendo la siguiente capa:

E01 — Backend Architecture
        ↓
E02 — Database Architecture
        ↓
E03 — API Architecture
        ↓
E04 — Authentication Architecture
        ↓
E05 — Authorization Architecture
        ↓
E06 — Identity Architecture
        ↓
E07 — Multi-Tenant Engineering
        ↓
E08 — Event Engineering
        ↓
E09 — AI Engineering
        ↓
E10 — Agent Engineering
        ↓
...

La idea es que E01 y E02 ya constituyan una base técnica coherente: E01 define cómo se estructura y ejecuta el backend; E02 define cómo ese backend persiste, protege, consulta y evoluciona sus datos.

Siguiente documento recomendado: E03 — EVOXA API Architecture, donde bajaremos A06 a la implementación concreta de REST API, endpoints, recursos, DTOs, request/response contracts, versionado, errores, paginación, filtros, autenticación, autorización, webhooks y OpenAPI.
