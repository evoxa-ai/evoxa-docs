E71 — EVOXA DATABASE ARCHITECTURE
1. Propósito

E71 define la arquitectura de persistencia de EVOXA: qué datos existen, quién los posee, cómo se almacenan, cómo se relacionan, cómo se transaccionan, cómo evolucionan y cómo se protegen.

La cadena queda ahora:

E67 — Master Application Blueprint
        ↓
E68 — Technical Stack & Platform
        ↓
E69 — Project / Repository
        ↓
E70 — Module & Package
        ↓
E71 — Database Architecture
        ↓
E72 — API Architecture
        ↓
E73 — Testing Architecture
        ↓
E74 — Deployment Architecture
        ↓
IMPLEMENTATION

E71 es el puente entre la arquitectura lógica de EVOXA y su estado persistente real.

2. Decisión arquitectónica principal

EVOXA utilizará una arquitectura de datos relacional, modular y orientada a ownership, con:

Primary transactional database
        ↓
PostgreSQL

y capacidades especializadas alrededor:

PostgreSQL
   │
   ├── Transactional State
   ├── Domain Data
   ├── Governance Data
   ├── Audit Data
   ├── Outbox
   └── Read / Projection Support

Mientras que:

Redis      → Cache
OpenSearch → Search
Object Storage → Files / Artifacts / Archives

no sustituirán a PostgreSQL como fuente primaria de verdad transaccional.

3. Database Architecture Principle

La regla fundamental:

Cada módulo es propietario de sus datos.

Por tanto:

Resource Module
      ↓
Resource Data

Workflow Module
      ↓
Workflow Data

Execution Module
      ↓
Execution Data

y no:

Everything
    ↓
One giant shared schema
4. Database Topology

La topología lógica inicial:

                         EVOXA
                           │
                           ▼
                    PostgreSQL Cluster
                           │
          ┌────────────────┼────────────────┐
          │                │                │
          ▼                ▼                ▼
       Domain          Governance        Platform
        Data              Data             Data
          │                │
          └────────────────┼────────────────┘
                           ▼
                       Outbox
                           │
                           ▼
                     Event Platform
5. Primary Database

PostgreSQL será el sistema principal para:

Entities
Aggregates
Relationships
Configuration State
Workflow State
Execution State
Policies
Governance State
Audit Metadata
Transactional Events
6. Database Responsibilities

PostgreSQL debe resolver:

✓ ACID transactions
✓ Referential integrity
✓ Constraints
✓ Concurrency control
✓ Versioning
✓ Durable persistence
✓ Querying
✓ Transactional event publication

No debe utilizarse como sustituto de:

✗ Message broker
✗ Cache
✗ Search engine
✗ Object storage
7. Database Boundaries

La arquitectura tendrá tres niveles:

Database
   ↓
Schema
   ↓
Table

Pero estos niveles representan diferentes responsabilidades.

8. Schema Strategy

Propuesta inicial:

PostgreSQL
│
├── core
├── resource
├── execution
├── workflow
├── policy
├── decision
├── action
├── data
├── governance
├── audit
└── platform

Esto permite que los límites de E70 tengan una representación persistente.

9. Why Schema per Module

Por ejemplo:

resource.resources
resource.resource_versions

y:

workflow.workflows
workflow.workflow_runs

en lugar de:

public.resources
public.workflows
public.everything

Ventajas:

Ownership
Isolation
Discoverability
Security
Migration Boundaries
Architecture Enforcement
10. Schema Ownership

Cada schema tiene un propietario:

Schema	Owner
core	Core Platform
resource	Resource Module
execution	Execution Module
workflow	Workflow Module
policy	Policy Module
decision	Decision Module
action	Action Module
data	Data Module
governance	Governance
audit	Audit
platform	Platform
11. Cross-Schema Access

Regla estricta:

Module A
   ✕
Direct SQL
   ↓
Module B tables

Preferir:

Module A
   ↓
Application Contract
   ↓
Module B

o:

Module A
   ↓
Read Model
12. Database as Source of Truth

La fuente de verdad será:

PostgreSQL

para el estado transaccional.

Por ejemplo:

Resource

vive en:

resource.resources

Mientras:

Redis
OpenSearch
Read Models

son representaciones derivadas.

13. Source of Truth Hierarchy
                    PostgreSQL
                        │
          ┌─────────────┼─────────────┐
          ▼             ▼             ▼
        Redis       OpenSearch    Projections
       (cache)       (search)      (read)

Nunca:

Redis → authoritative state
14. Naming Convention

Tablas:

snake_case
plural

Ejemplos:

resources
resource_versions
workflows
workflow_steps
executions
policies
decisions
actions
datasets
15. Primary Keys

La arquitectura utilizará IDs explícitos.

Recomendación:

UUID / UUIDv7

con preferencia por identificadores ordenables temporalmente cuando el stack lo permita.

Conceptualmente:

resource_id UUID
16. Why Not Integer IDs

Evitar depender exclusivamente de:

SERIAL
BIGSERIAL

porque IDs distribuidos presentan ventajas importantes para EVOXA:

Distributed creation
External references
Event correlation
Service extraction
Reduced predictability
17. Standard Metadata

Las entidades persistentes relevantes deberán soportar:

id
created_at
updated_at

y cuando corresponda:

created_by
updated_by
version
status
18. Versioning

El versionado será explícito donde exista concurrencia o evolución de estado.

Ejemplo:

version BIGINT

Esto permite:

Optimistic Concurrency Control
19. Optimistic Concurrency

Modelo:

Read entity
   ↓
version = 7
   ↓
Modify
   ↓
UPDATE ... WHERE id = X AND version = 7
   ↓
version = 8

Si no se modifica ninguna fila:

Concurrency Conflict
20. Entity vs Aggregate

No toda tabla es un Aggregate.

La jerarquía conceptual:

Aggregate
   ├── Entity
   ├── Entity
   └── Value Objects

La transacción debe normalmente respetar la frontera del Aggregate.

21. Aggregate Ownership

Ejemplo conceptual:

Resource
└── ResourceConfiguration

puede pertenecer al mismo Aggregate.

Mientras:

Resource
Workflow
Execution

son Aggregates distintos.

22. Aggregate IDs

Los aggregates deberán tener un identificador estable:

resource_id
workflow_id
execution_id

Las entidades internas pueden utilizar sus propios IDs cuando sea necesario.

23. Foreign Keys

Las FK serán utilizadas dentro de los límites apropiados.

Ejemplo:

resource.resource_versions
      ↓
resource.resources

sí.

Pero:

workflow.workflow_steps
      ↓
resource.resources

debe analizarse cuidadosamente antes de establecer una FK cross-schema.

24. Cross-Module Foreign Keys

Regla general:

Evitar foreign keys que conviertan un bounded context en dependiente físicamente de otro.

En su lugar:

resource_id

puede existir como referencia lógica.

La validación se realiza mediante el módulo propietario cuando sea necesario.

25. Why This Matters

Si Workflow contiene:

FOREIGN KEY resource_id
REFERENCES resource.resources

entonces:

Workflow

queda físicamente acoplado a:

Resource

Esto dificulta:

Service Extraction
Independent Migration
Independent Deployment
Data Partitioning
26. Transaction Boundaries

La regla principal:

One use case
     ↓
One explicit transaction boundary

Cuando sea posible:

Transaction
    ↓
Single Aggregate / Module
27. Cross-Module Transactions

No serán el mecanismo normal de integración.

Preferir:

Module A
  ↓
Commit
  ↓
Event
  ↓
Module B
28. Outbox Pattern

EVOXA utilizará un patrón Outbox para eventos que deben publicarse de forma fiable.

BEGIN TRANSACTION
      │
      ├── Update Domain State
      │
      └── Insert Outbox Event
      │
COMMIT
      │
      ▼
Outbox Publisher
      │
      ▼
Message Broker

Esto evita:

DB updated
BUT
event lost
29. Outbox Schema

Ejemplo conceptual:

platform.outbox_events

Campos:

id
aggregate_type
aggregate_id
event_type
event_version
payload
headers
created_at
published_at
attempts
status
30. Outbox Guarantees

El Outbox garantiza principalmente:

Transactional persistence
+
Reliable publication attempt

No garantiza por sí solo exactamente-once delivery.

Por ello los consumidores deben ser:

Idempotent
31. Inbox Pattern

Para consumidores críticos:

platform.inbox_messages

podrá registrar:

message_id
consumer
received_at
processed_at
status

Esto permite deduplicación.

32. Event Flow
Domain
  ↓
Application
  ↓
Transaction
 ┌─────────────────┐
 │ Domain State    │
 │ Outbox Event    │
 └─────────────────┘
          ↓
       Commit
          ↓
    Outbox Worker
          ↓
       Broker
          ↓
       Consumer
          ↓
       Inbox
          ↓
     Application
33. Resource Database

El módulo Resource tendrá inicialmente:

resource.resources
resource.resource_versions
resource.resource_metadata
34. Resource Table

Conceptualmente:

resources
---------
id
resource_type
name
status
owner_id
version
created_at
updated_at

No se deben añadir campos simplemente "por si acaso".

35. Resource Versioning

Si un Resource requiere histórico:

resources
    │
    └── resource_versions

Modelo:

Resource
  ↓
Version 1
Version 2
Version 3
36. Execution Database
execution.executions
execution.execution_attempts
execution.execution_results
37. Execution

Conceptualmente:

executions
-----------
id
action_id
status
started_at
completed_at
version
created_at
updated_at
38. Execution Attempts

Para retries:

execution_attempts
------------------
id
execution_id
attempt_number
status
started_at
completed_at
error_code
error_message

Esto permite distinguir:

Execution

de:

Attempt
39. Workflow Database
workflow.workflows
workflow.workflow_versions
workflow.workflow_runs
workflow.workflow_steps
workflow.workflow_step_runs
40. Workflow Definition vs Run

Debe existir una separación fundamental:

Workflow Definition
        │
        ▼
Workflow Run

La definición es declarativa.

El Run representa una ejecución concreta.

41. Workflow Versioning

Una definición publicada no debe cambiar arbitrariamente.

Modelo:

Workflow
   ├── Version 1
   ├── Version 2
   └── Version 3

Una ejecución debe apuntar a una versión concreta.

42. Policy Database
policy.policies
policy.policy_versions
policy.policy_rules
policy.policy_evaluations
43. Policy Definition
policies
--------
id
name
type
status
current_version
created_at
updated_at
44. Policy Version
policy_versions
---------------
id
policy_id
version
definition
status
created_at
created_by

La definición puede almacenarse estructuradamente, posiblemente mediante JSONB, siempre que exista un modelo contractual claro.

45. Decision Database
decision.decisions
decision.decision_inputs
decision.decision_outcomes

Una Decision debe poder reconstruirse:

Inputs
+
Policy Version
+
Context
+
Result
46. Decision Explainability

Para decisiones relevantes:

decision.decision_inputs
decision.decision_outcomes
decision.evidence

permitirán explicar:

Why?
Based on what?
Using which policy?
At what time?
47. Action Database
action.actions
action.action_parameters
action.action_status_history

La Action representa la intención ejecutable.

48. Data Database

El módulo Data puede contener:

data.datasets
data.data_products
data.data_sources
data.data_classifications
data.data_lineage
49. Dataset

Conceptualmente:

datasets
--------
id
name
description
source_id
classification
owner_id
lifecycle_state
created_at
updated_at
50. Data Product
data_products
-------------
id
name
owner_id
status
description
created_at
updated_at

Un Data Product no necesariamente es idéntico a un Dataset.

51. Data Lineage

La arquitectura debe permitir:

Source
 ↓
Transformation
 ↓
Dataset
 ↓
Data Product
 ↓
Consumer

Por ello:

data.data_lineage

será un elemento importante.

52. Governance Database
governance.policies
governance.controls
governance.control_evaluations
governance.compliance_findings
governance.evidence

Pero debe evitarse duplicar innecesariamente el Policy Domain.

53. Governance vs Policy

La distinción:

policy

representa reglas que pueden gobernar comportamiento.

governance

representa:

Controls
Compliance
Evidence
Assurance
Audit

Un Governance Policy puede referenciar una Policy existente en vez de duplicarla.

54. Audit Database

Audit tendrá una frontera especialmente protegida:

audit.audit_events
audit.access_events
audit.security_events
55. Audit Event

Modelo conceptual:

audit_events
------------
id
event_type
actor_id
action
resource_type
resource_id
correlation_id
causation_id
timestamp
result
metadata
56. Audit Immutability

Los registros de auditoría importantes deben ser:

Append Only

Evitar:

UPDATE audit_events
DELETE audit_events

excepto por procesos explícitos de lifecycle/retention controlados.

57. Audit Integrity

Para evidencia de alto valor se puede añadir:

hash
previous_hash
signature

creando una cadena verificable:

Event A
  ↓ hash
Event B
  ↓ hash
Event C

La implementación exacta se determinará en los capítulos de seguridad/auditoría detallada.

58. Soft Delete

EVOXA no utilizará soft delete universalmente.

No hacer:

deleted_at

en todas las tablas por defecto.

La estrategia depende del tipo de dato.

59. Data Deletion Strategies

Habrá cuatro estrategias:

1. Hard Delete
2. Soft Delete
3. Tombstone
4. Archive

La elección dependerá de:

Legal Requirements
Audit
Retention
Business Semantics
Dependencies
60. Temporal Data

Cuando necesitemos histórico:

valid_from
valid_to

o tablas de versión.

No debemos utilizar timestamps indiscriminadamente para simular versioning.

61. JSONB Strategy

JSONB se utilizará cuando el dato sea:

Semi-structured
Schema-variable
Configuration-like
External payload
Policy definition
Metadata

No como sustituto de un modelo relacional bien definido.

62. JSONB Anti-Pattern

Evitar:

entity
------
id
data JSONB

para guardar todo el sistema.

Esto destruye:

Constraints
Indexes
Referential Integrity
Discoverability
Type Safety
63. Relational First

Regla:

Si un dato tiene estructura estable y relaciones importantes, será modelado relacionalmente.

JSONB será complementario.

64. Indexing Strategy

Los índices se diseñarán según consultas reales.

Tipos:

B-tree
GIN
GiST
Partial Index
Composite Index

No crear índices por intuición.

65. Common Indexes

Candidatos habituales:

status
created_at
updated_at
owner_id
foreign references
natural lookup keys

Pero cada uno debe justificarse por query patterns.

66. Composite Indexes

Ejemplo conceptual:

(status, created_at)

cuando las consultas realmente utilicen ambas columnas.

El orden importa.

67. Unique Constraints

Las reglas de negocio que exijan unicidad deberán expresarse también en la base de datos.

Ejemplo:

UNIQUE(resource_type, name, owner_id)

cuando corresponda.

68. Database Constraints

La base de datos debe proteger invariantes importantes mediante:

PRIMARY KEY
FOREIGN KEY
UNIQUE
CHECK
NOT NULL

No depender exclusivamente de validación de aplicación.

69. Defense in Depth

La integridad tendrá varias capas:

API Validation
       ↓
Application Validation
       ↓
Domain Rules
       ↓
Database Constraints
70. Migration Architecture

Toda modificación del schema será versionada.

Migration 001
Migration 002
Migration 003
...

Nunca modificar producción manualmente como práctica normal.

71. Migration Ownership

Cada módulo podrá poseer sus migrations.

Conceptualmente:

modules/resource/
└── migrations/

modules/workflow/
└── migrations/

o una estructura central equivalente.

Lo importante es mantener:

Module Ownership
72. Migration Rules

Una migration debe ser:

Versioned
Reviewable
Repeatable
Traceable
Testable
73. Expand / Contract

Cambios incompatibles deberán seguir:

Expand
   ↓
Deploy
   ↓
Migrate
   ↓
Switch
   ↓
Contract

Nunca:

Drop column
+
Deploy application simultaneously

sin estrategia de compatibilidad.

74. Zero-Downtime Schema Evolution

Ejemplo:

Old Column
New Column
     ↓
Dual Write
     ↓
Backfill
     ↓
Switch Reads
     ↓
Remove Old Column
75. Seed Data

Debe diferenciarse:

Schema Migration

de:

Reference / Seed Data

No mezclar datos operacionales con migrations estructurales sin razón.

76. Environment Strategy

Inicialmente:

Development
Test
Staging
Production

cada uno con su propia base de datos.

Nunca compartir una DB de desarrollo con producción.

77. Database Configuration

Credentials:

Environment / Secret Management

No:

git repository

No incluir:

password
connection strings
tokens
private keys

en código fuente.

78. Connection Management

Cada runtime deberá utilizar pools controlados.

Application
   ↓
Connection Pool
   ↓
PostgreSQL

No abrir una conexión nueva por cada operación sin control.

79. Read Replicas

La arquitectura debe permitir:

Primary
   │
   ├── Read Replica 1
   └── Read Replica 2

para workloads de lectura intensiva.

Pero inicialmente no es obligatorio desplegarlas.

80. Read/Write Separation

Cuando sea necesario:

Writes → Primary
Reads  → Replica

pero solamente cuando la consistencia requerida lo permita.

81. CQRS Compatibility

EVOXA podrá evolucionar hacia:

Command Model
      ↓
Events
      ↓
Read Model

sin obligar a aplicar CQRS completo desde el primer día.

82. Projection Architecture

Las projections serán derivadas:

Domain State
     ↓
Event
     ↓
Projection
     ↓
Read Model

Nunca serán la autoridad original del dato.

83. Search Database

OpenSearch podrá mantener:

Search Documents
Aggregations
Full Text
Faceted Search

pero:

OpenSearch ≠ Source of Truth
84. Cache Database

Redis almacenará:

Cached Objects
Locks
Rate Limit State
Temporary Coordination State

pero:

Redis ≠ Durable Domain Database
85. Object Storage

Para:

Large Files
Artifacts
Exports
Archives
Evidence Attachments

utilizaremos Object Storage.

La DB almacena:

metadata
reference
checksum
content_type
size

no necesariamente el contenido binario.

86. Large Binary Data

Evitar almacenar grandes blobs directamente en PostgreSQL salvo necesidad explícita.

Preferir:

PostgreSQL
   ↓ metadata
Object Storage
   ↓
Binary Content
87. Checksums

Los artefactos importantes deberán poder registrar:

sha256
size
content_type
created_at
storage_reference

Esto permite verificar integridad.

88. Data Classification

Cada dato relevante podrá tener clasificación:

PUBLIC
INTERNAL
CONFIDENTIAL
RESTRICTED

La taxonomía final se definirá en Data Governance/Security.

89. Database Security

La base de datos deberá aplicar:

Least Privilege
Role Separation
Encryption
Network Isolation
Credential Rotation
Audit
90. Database Roles

Separación conceptual:

application_role
migration_role
readonly_role
reporting_role
administration_role

La aplicación no debe ejecutar migrations como parte de su runtime normal.

91. Encryption

Dos niveles:

Encryption in Transit
TLS

Encryption at Rest
Storage / Database Encryption

Para campos especialmente sensibles puede existir:

Application-Level Encryption
92. Row-Level Security

PostgreSQL RLS puede utilizarse cuando exista necesidad de aislamiento por:

Tenant
Organization
Security Domain

Pero no se activará indiscriminadamente.

93. Multi-Tenancy

La estrategia deberá quedar preparada para:

tenant_id

cuando EVOXA opere multi-tenant.

Modelo potencial:

tenant
   ↓
resource
workflow
data
governance
94. Tenant Isolation

Opciones:

Shared DB / Shared Schema
Shared DB / Schema per Tenant
Database per Tenant

La arquitectura inicial debe favorecer:

Shared DB
+
Logical Tenant Isolation

si el modelo de negocio lo permite.

95. Tenant ID

En un modelo multi-tenant:

tenant_id

debe estar presente en las entidades apropiadas y formar parte de constraints/indexes cuando sea necesario.

96. Data Lifecycle

E71 debe respetar los capítulos:

E45 Data Lifecycle
E46 Data Retention
E47 Data Disposal
E48 Data Archival

Por ello cada clase de dato deberá tener:

Lifecycle
Retention
Archive
Disposal

definidos posteriormente.

97. Retention Metadata

Cuando corresponda:

retention_policy_id
retention_until
archive_status
disposal_status

Pero no convertir estos campos en obligatorios para todas las tablas.

98. Backup Architecture

PostgreSQL deberá soportar:

Full Backups
Incremental / WAL
Point-in-Time Recovery

La arquitectura detallada se desarrollará en E42, pero E71 debe dejarla compatible.

99. Recovery Point Objective

El diseño deberá permitir establecer:

RPO
RTO

por clase de datos.

No todos los módulos necesariamente tendrán el mismo RPO/RTO.

100. Database Observability

Debemos monitorizar:

Connections
Query Latency
Locks
Deadlocks
Replication Lag
Storage
Cache Hit Ratio
Slow Queries
Transaction Rate
Errors
101. Slow Query Management

Las queries críticas deben poder detectarse mediante:

pg_stat_statements

o mecanismo equivalente.

102. Database Health

Health checks:

Connectivity
Readiness
Transaction Capability
Replication Health
Storage Capacity
103. Database Capacity

La capacidad deberá evaluarse por:

CPU
RAM
IOPS
Storage
Connections
Transaction Rate
Query Latency

No únicamente por tamaño de DB.

104. Partitioning

PostgreSQL partitioning podrá utilizarse para tablas de gran crecimiento:

audit_events
execution_history
event_logs

especialmente por:

time
tenant

cuando el volumen lo justifique.

No particionar prematuramente.

105. Archival Tables

Para grandes históricos:

active table
      ↓
archive table / object storage

según la política de lifecycle.

106. Database Disaster Recovery

La arquitectura debe soportar:

Primary
   ↓
Replication / WAL
   ↓
Backup
   ↓
Recovery

y deberá ser compatible con:

E40 Recovery
E41 Disaster Recovery
E42 Backup & Restore
107. Data Integrity

La integridad tendrá cuatro capas:

1. Application
2. Domain
3. Database
4. Operational / Recovery
108. Consistency Model

EVOXA distinguirá:

Strong Consistency
Eventual Consistency
Read-Your-Writes

según el caso de uso.

No se asumirá que todo el sistema debe ser fuertemente consistente.

109. Strong Consistency Examples

Normalmente:

Aggregate State
Financial / Critical State
Authorization State
Security-Critical State

pueden requerir consistencia fuerte.

110. Eventual Consistency Examples

Puede utilizarse para:

Search Index
Analytics
Reporting
Dashboards
Non-critical Projections
Caches
111. Reporting Database

EVOXA podrá evolucionar hacia:

Operational DB
      ↓
ETL / CDC
      ↓
Analytical Store

pero no debemos construir un Data Warehouse completo antes de tener workloads reales.

112. Analytics Boundary

La arquitectura deja preparado:

PostgreSQL
   ↓
CDC / Events
   ↓
Analytics Platform

para E31 — Analytics Architecture.

113. Database Governance

Toda tabla debe tener:

Owner
Purpose
Classification
Lifecycle
Retention
Access Policy
Migration Owner

Idealmente esto evolucionará hacia metadata automatizada.

114. Database Documentation

Cada módulo debe documentar:

Entities
Tables
Relationships
Constraints
Indexes
Queries
Transactions
Events
Retention
115. Entity-to-Table Mapping

No todas las entidades necesitan una tabla 1:1.

Puede existir:

Entity
 ↓
Multiple Tables

o:

Value Objects
 ↓
Embedded Columns

según el modelo.

116. ORM Boundary

Si EVOXA utiliza ORM:

Domain Entity
      ≠
ORM Model

cuando las diferencias arquitectónicas lo requieran.

Un ORM no debe dictar el modelo de dominio.

117. Persistence Mapping

Idealmente:

Domain
   ↓
Mapper
   ↓
Persistence Model
   ↓
ORM / SQL

Esto protege el dominio.

118. Repository Boundary
Domain
   ↓
Repository Interface
   ↑
Persistence Adapter
   ↓
PostgreSQL
119. Database Access Flow

El flujo estándar será:

HTTP / Message
       ↓
Application Handler
       ↓
Domain
       ↓
Repository Port
       ↓
Persistence Adapter
       ↓
PostgreSQL
120. Forbidden Flow

No:

HTTP
 ↓
SQL

ni:

Domain
 ↓
PostgreSQL Driver

ni:

Module A
 ↓
Module B Tables
121. Final Logical Database

La arquitectura objetivo queda:

                         PostgreSQL
                              │
 ┌────────────────────────────┼─────────────────────────────┐
 │                            │                             │
 ▼                            ▼                             ▼
DOMAIN DATA              GOVERNANCE DATA              PLATFORM DATA
 │                            │                             │
 ├─ resource                 ├─ governance                ├─ outbox
 ├─ execution                ├─ audit                     ├─ inbox
 ├─ workflow                 └─ evidence                  └─ runtime metadata
 ├─ policy
 ├─ decision
 ├─ action
 └─ data
122. Complete Data Ecosystem

Pero PostgreSQL no estará solo:

                         EVOXA DATA PLATFORM
                                │
       ┌────────────────────────┼─────────────────────────┐
       │                        │                         │
       ▼                        ▼                         ▼
 PostgreSQL                  Redis                   OpenSearch
 Source of Truth              Cache                    Search
       │
       ├──────────────┐
       │              │
       ▼              ▼
   Outbox          Read Models
       │
       ▼
 Message Platform
       │
       ▼
 Analytics / Intelligence
       │
       ▼
 Object Storage
123. E71 Architectural Decisions

Las decisiones principales de E71 son:

AD-071-01
PostgreSQL is EVOXA's primary transactional database.

AD-071-02
Data ownership follows module ownership.

AD-071-03
Schemas represent major module boundaries.

AD-071-04
Cross-module direct table access is prohibited.

AD-071-05
Cross-module foreign keys are minimized.

AD-071-06
UUID/UUIDv7-style identifiers are preferred.

AD-071-07
Optimistic concurrency is supported.

AD-071-08
Transactional Outbox is part of the architecture.

AD-071-09
Audit data is append-oriented.

AD-071-10
Redis/OpenSearch/Object Storage are specialized stores,
not transactional sources of truth.

AD-071-11
JSONB is complementary, not a replacement for relational modeling.

AD-071-12
Schema evolution uses versioned migrations and expand/contract.

AD-071-13
CQRS/read models are supported without requiring full CQRS.

AD-071-14
Database integrity is enforced through constraints as well as code.
124. E71 → E72

Con E71 ya tenemos:

MODULE
   ↓
DATA OWNERSHIP
   ↓
SCHEMA
   ↓
TABLE
   ↓
ENTITY
   ↓
TRANSACTION
   ↓
EVENT

Ahora podemos pasar a:

E72 — EVOXA API ARCHITECTURE

Ahí construiremos:

Module
   ↓
Use Case
   ↓
API Contract
   ↓
Endpoint
   ↓
Request
   ↓
Validation
   ↓
Application Handler
   ↓
Response

y definiremos:

REST / HTTP
API Versioning
Resource Naming
Commands
Queries
Pagination
Filtering
Sorting
Errors
Idempotency
Authentication
Authorization
Rate Limiting
Webhooks
Events
OpenAPI
API Governance

E71, por tanto, deja lista la capa de persistencia para que en E72 podamos empezar a definir las interfaces externas reales de EVOXA.
