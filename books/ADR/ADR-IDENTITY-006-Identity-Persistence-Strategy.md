ADR-IDENTITY-006 — Identity Persistence Strategy

Status: Proposed
Version: 1.0.0
Domain: Identity
Decision Type: Architecture / Data
Scope: Identity persistence, transactional data, sessions, tokens, MFA, audit, caching and events
Related: ESP-0001 — Identity
Depends on:

ADR-IDENTITY-001 — Identity Architecture & Technology Decisions
ADR-IDENTITY-002 — JWT & Token Lifecycle
ADR-IDENTITY-003 — Refresh Token Security & Session Management
ADR-IDENTITY-005 — Authorization & Policy Engine
1. Context

EVOXA Identity requiere una estrategia de persistencia capaz de soportar:

usuarios;
organizaciones;
memberships;
roles;
permisos;
sesiones;
refresh tokens;
MFA;
políticas de seguridad;
auditoría;
eventos;
aislamiento multi-tenant;
alta disponibilidad;
trazabilidad;
seguridad;
evolución futura.

El Blueprint establece una arquitectura de plataforma Domain-Driven, Modular, API-First, Event-Driven, Cloud-Ready y Observable, y contempla una plataforma de datos basada en diferentes mecanismos de persistencia según el tipo de información.

La arquitectura de referencia contempla específicamente:

PostgreSQL
Event Store
Redis
Object Storage
Vector Database
Knowledge Graph
Data Warehouse

pero no prescribe que todos los dominios deban utilizar todos estos mecanismos.

2. Problem Statement

Identity maneja información con características muy diferentes.

Por ejemplo:

User
Organization
Membership
Role
Permission

requieren consistencia transaccional.

Mientras que:

Session
Rate Limit
Temporary MFA Challenge
Security Cache

pueden requerir acceso rápido y expiración.

Por otra parte:

Audit
Domain Events
Integration Events

requieren propiedades diferentes de las entidades transaccionales.

Por lo tanto, utilizar una única tecnología de almacenamiento para absolutamente todo produciría una arquitectura menos adecuada.

3. Decision

Se adopta una estrategia de Polyglot Persistence controlada, con:

Primary Transactional Store

PostgreSQL

como fuente de verdad para los datos persistentes y transaccionales de Identity.

High-Speed / Temporary Store

Redis

como almacenamiento complementario para información temporal, efímera o de acceso frecuente.

Event Persistence

Los eventos críticos serán publicados mediante una arquitectura basada en Transactional Outbox, evitando que una transacción de negocio exitosa quede sin su correspondiente evento.

Audit Persistence

La auditoría tendrá un mecanismo de persistencia separado conceptualmente de los logs operacionales y deberá permitir retención, integridad y consulta histórica.

4. Persistence Architecture

La arquitectura propuesta es:

                    IDENTITY DOMAIN
                          │
              ┌───────────┴───────────┐
              │                       │
              ▼                       ▼
       PostgreSQL                  Redis
       Source of Truth             Temporary
              │                       │
              │                       ├── Rate Limits
              │                       ├── Temporary MFA
              │                       ├── Security Counters
              │                       ├── Cache
              │                       └── Session Acceleration
              │
              ├── Users
              ├── Organizations
              ├── Memberships
              ├── Roles
              ├── Permissions
              ├── Sessions
              ├── Refresh Tokens
              ├── MFA Metadata
              └── Security Policies
                          │
                          ▼
                 Transactional Outbox
                          │
                          ▼
                    Event Platform
                          │
             ┌────────────┼────────────┐
             ▼            ▼            ▼
        Analytics      Security    Notifications
5. PostgreSQL as Source of Truth

PostgreSQL será la fuente de verdad para las entidades transaccionales de Identity.

La arquitectura de referencia contempla PostgreSQL como parte de la plataforma de datos, y ESP-0001 establece PostgreSQL como persistencia transaccional principal de Identity.

Por tanto:

Redis nunca será considerado la fuente de verdad de Identity.

6. Identity Data Model

Las principales entidades persistentes serán:

users
organizations
memberships
roles
permissions
role_permissions
sessions
refresh_tokens
mfa_factors
mfa_challenges
recovery_codes
trusted_devices
security_policies
audit_events
outbox_events

No necesariamente todas deben implementarse en la primera iteración del sistema.

7. Users

La entidad User deberá persistirse en PostgreSQL.

Información conceptual:

User
├── id
├── email
├── password_hash
├── status
├── created_at
├── updated_at
├── last_login_at
└── audit metadata

Nunca se almacenará:

password

en texto plano.

El password deberá almacenarse mediante un algoritmo resistente apropiado; ESP-0001 y ADR-IDENTITY-001 contemplan Argon2id.

8. Organizations

Las organizaciones también serán persistidas en PostgreSQL.

Organization
├── id
├── name
├── status
├── created_at
├── updated_at
└── audit metadata

La organización constituye el límite principal de tenant definido para Identity.

9. Memberships

La relación entre usuarios y organizaciones será persistida mediante:

memberships

Conceptualmente:

User
   │
   ├── Membership → Organization A
   │
   └── Membership → Organization B

Esto permite que un usuario pueda pertenecer a una o potencialmente múltiples organizaciones, según la política que finalmente se adopte.

La persistencia debe permitir validar:

User
   ↓
Membership
   ↓
Organization

antes de autorizar operaciones.

10. Roles and Permissions

Los roles y permisos serán persistidos en PostgreSQL.

Modelo:

roles
permissions
role_permissions

Relación:

Role
  │
  └── RolePermission
          │
          └── Permission

Ejemplo:

Coach
 ├── clients.read
 ├── clients.update
 ├── training.read
 └── training.update

La estrategia de autorización RBAC + ABAC definida en ADR-IDENTITY-005 utiliza estos datos como parte del contexto de autorización.

11. Sessions

Las sesiones serán persistentes.

sessions

La sesión será la principal frontera de seguridad stateful definida por ADR-IDENTITY-003.

Conceptualmente:

Session
├── id
├── user_id
├── organization_id
├── status
├── created_at
├── last_activity_at
├── expires_at
├── revoked_at
├── revocation_reason
└── metadata

Esto permite:

logout;
logout-all;
revocación administrativa;
revocación por seguridad;
gestión multi-dispositivo;
detección de sesiones activas.
12. Refresh Tokens

Los refresh tokens serán persistidos en PostgreSQL.

Sin embargo:

Nunca se almacenará el refresh token en texto plano.

Se almacenará una representación segura, por ejemplo:

token_hash

junto con:

token_id
session_id
family_id
status
created_at
used_at
expires_at
revoked_at

Esto permite implementar la estrategia definida en ADR-IDENTITY-003:

Refresh Token
       ↓
Rotation
       ↓
Token Family
       ↓
Reuse Detection
       ↓
Family Revocation
13. Refresh Token Family

Las familias de refresh tokens serán persistentes.

Ejemplo:

Family F1

Token A
   ↓
Token B
   ↓
Token C
   ↓
Token D

Si se detecta reutilización de:

Token B

el sistema podrá identificar:

family_id = F1

y revocar la familia correspondiente.

14. Redis

Redis será utilizado como almacenamiento complementario.

Casos apropiados:

Rate Limiting
MFA Temporary State
Security Counters
Authorization Cache
Session Acceleration
Short-lived Data

Por ejemplo:

login_attempts:user_123
mfa_challenge:abc123
authorization:user_123:org_456

Los datos de Redis deben poder reconstruirse o expirar cuando sea posible.

15. Redis Is Not the Source of Truth

Esta regla es fundamental:

PostgreSQL
     ↓
SOURCE OF TRUTH

Redis:

Redis
  ↓
CACHE / TEMPORARY STATE

Si Redis falla:

Redis unavailable
       ↓
Identity must remain secure
       ↓
Fallback / controlled degradation
       ↓
Never fail-open

En determinadas operaciones críticas puede ser preferible rechazar temporalmente la operación antes que autorizarla incorrectamente.

16. Authorization Cache

En relación con ADR-IDENTITY-005, se podrá utilizar Redis para cachear decisiones o datos necesarios para autorización.

Por ejemplo:

user_id
organization_id
role
permissions
policy_version

Sin embargo, los cambios en:

Role
Permission
Membership
Security Policy
User Status
Organization Status

deben provocar invalidación del cache correspondiente.

17. MFA Persistence

Los metadatos de MFA serán persistidos.

Ejemplo:

mfa_factors
mfa_challenges
recovery_codes
trusted_devices

Los secretos MFA sensibles no deberán almacenarse en texto plano.

La arquitectura deberá utilizar un mecanismo seguro de protección de secretos.

El mecanismo exacto de gestión criptográfica queda como decisión posterior.

18. Security Policies

Las políticas de seguridad serán persistidas en PostgreSQL.

Conceptualmente:

security_policies

con:

id
name
scope
status
version
priority
rules
created_at
updated_at

Esto permite:

Policy
   ↓
Version
   ↓
Evaluation
   ↓
Authorization Decision
19. Audit Persistence

Los audit events deben mantenerse separados conceptualmente de los logs de aplicación.

Application Logs
        ≠
Audit Events

Los logs sirven principalmente para:

debugging
observability
operations

Los audit events sirven para:

security
compliance
accountability
forensics

El Blueprint establece la necesidad de auditabilidad y ESP-0001 contempla una estrategia de retención de largo plazo para eventos de auditoría.

20. Audit Immutability

Los registros de auditoría críticos deben ser append-oriented.

Idealmente:

INSERT
  ↓
Audit Event

y no:

UPDATE
DELETE

como operaciones normales.

Los mecanismos exactos de almacenamiento inmutable, retención y protección contra manipulación deberán definirse en una decisión específica de Audit Persistence.

21. Transactional Outbox

Se adopta Transactional Outbox para eventos que requieran consistencia entre:

Database Transaction
        +
Domain Event

Ejemplo:

BEGIN TRANSACTION

Create User
      +
Create Outbox Event

COMMIT

Posteriormente:

Outbox
   ↓
Event Publisher
   ↓
Event Platform

Esto evita:

DB COMMIT = SUCCESS
Event Publish = FAILURE

sin mecanismo de recuperación.

22. Outbox Table

Conceptualmente:

outbox_events

podría contener:

id
event_id
event_type
event_version
aggregate_type
aggregate_id
tenant_id
payload
created_at
published_at
status
retry_count
last_error

El esquema definitivo deberá quedar definido en la implementación de persistencia.

23. Event Delivery

El sistema asumirá:

At-Least-Once Delivery

por lo que los consumidores deben ser idempotentes.

Ejemplo:

UserCreated
     ↓
Event Platform
     ↓
Analytics

Si el evento se entrega dos veces:

UserCreated
UserCreated

Analytics no debe generar dos efectos incorrectos.

24. Database Transactions

Las operaciones críticas deberán utilizar transacciones.

Ejemplos:

User creation
Create User
Create Membership
Create Audit Event
Create Outbox Event
Refresh rotation
Validate token
Mark old token USED
Create new token
Update session
Create security event if necessary
Commit

La rotación debe ser atómica para evitar condiciones de carrera.

25. Concurrency Control

La persistencia debe proteger operaciones concurrentes.

Especialmente:

Refresh Token Rotation
Session Revocation
Membership Changes
Role Changes
Permission Changes
Security Policy Changes

El diseño deberá evitar que dos solicitudes concurrentes puedan utilizar exitosamente el mismo refresh token.

26. Indexing

PostgreSQL deberá utilizar índices adecuados para las operaciones críticas.

Ejemplos conceptuales:

users.email
memberships.user_id
memberships.organization_id
memberships.user_id + organization_id
sessions.user_id
sessions.status
refresh_tokens.session_id
refresh_tokens.family_id
refresh_tokens.status
permissions.resource + action
audit_events.tenant_id
audit_events.timestamp
outbox_events.status

Los índices exactos serán definidos durante la especificación física de la base de datos.

27. Tenant Isolation at Persistence Layer

La seguridad multi-tenant no debe depender únicamente de la aplicación.

Las consultas deben incorporar explícitamente el tenant cuando corresponda.

Ejemplo conceptual:

SELECT *
FROM clients
WHERE id = :resource_id
AND organization_id = :tenant_id;

Nunca:

SELECT *
FROM clients
WHERE id = :resource_id;

si el contexto requiere aislamiento de tenant.

La implementación puede complementarse posteriormente con mecanismos adicionales de PostgreSQL, pero la estrategia exacta queda abierta.

28. Soft Delete and Retention

Identity deberá distinguir:

Business Status

de:

Physical Deletion

Por ejemplo:

User
ACTIVE
SUSPENDED
DISABLED

no implica necesariamente eliminar físicamente el registro.

La eliminación física debe estar gobernada por:

requisitos legales;
retención;
privacidad;
auditoría;
dependencias;
políticas de seguridad.

La estrategia definitiva de retención y eliminación deberá establecerse en una decisión específica.

29. Database Migrations

El esquema de Identity deberá evolucionar mediante migraciones versionadas.

No se deberá depender de:

automatic schema synchronization

como mecanismo principal de producción.

El flujo será conceptualmente:

Migration
    ↓
Validation
    ↓
QA
    ↓
Staging
    ↓
Production

Las migraciones deberán considerar compatibilidad hacia atrás cuando sea necesario para despliegues sin downtime.

30. Backup and Disaster Recovery

PostgreSQL deberá formar parte de la estrategia de:

backup;
restore;
disaster recovery;
replication;
monitoring.

Los objetivos generales de arquitectura contemplan:

Availability ≥ 99.9%
RTO < 1 hour
RPO < 15 minutes

Estos valores pertenecen al objetivo arquitectónico de EVOXA y deben validarse según el entorno de despliegue real.

31. Data Encryption

Los datos sensibles deben protegerse:

At Rest
Transit

La arquitectura de seguridad contempla:

TLS 1.3
AES-256

y protección específica para secretos criptográficos.

Los mecanismos concretos de gestión de claves no quedan completamente definidos por este ADR.

32. Repository Pattern

El dominio no deberá depender directamente de PostgreSQL.

Se utilizarán abstracciones:

Domain
   │
   ▼
Repository Interface
   │
   ▼
PostgreSQL Implementation

Ejemplo conceptual:

UserRepository
OrganizationRepository
MembershipRepository
RoleRepository
PermissionRepository
SessionRepository
RefreshTokenRepository
SecurityPolicyRepository
AuditRepository

Esto permite mantener el dominio independiente de detalles de infraestructura.

33. Persistence Boundaries

Identity será responsable de sus propios datos.

Otros dominios no deberían acceder directamente a sus tablas.

Incorrecto:

Training
   ↓
Identity PostgreSQL tables

Correcto:

Training
   ↓
Identity API / Context

o:

Training
   ↓
Identity Events

Esto respeta el principio de ownership de dominio.

34. Data Access Rules

No se permitirá:

Domain A
   ↓
Direct SQL
   ↓
Domain B Tables

Los dominios colaborarán mediante:

API
Events
Shared Contracts

Esto mantiene el desacoplamiento requerido por la arquitectura modular de EVOXA.

35. Failure Strategy

La estrategia de fallos debe ser segura.

PostgreSQL unavailable
Request
  ↓
Database unavailable
  ↓
Controlled failure
  ↓
No authorization bypass
Redis unavailable

Dependiendo de la operación:

Cache unavailable
       ↓
Fallback to source of truth

o:

Security-critical temporary state unavailable
       ↓
DENY / controlled failure

Nunca:

Infrastructure failure
       ↓
ALLOW
36. Observability

La persistencia debe integrarse con:

Logs
Metrics
Traces
Audit
Events

Cada operación relevante debe poder correlacionarse mediante:

trace_id
correlation_id
request_id

y cuando corresponda:

user_id
tenant_id
session_id

Esto es consistente con la arquitectura de observabilidad definida para EVOXA.

37. Performance

La estrategia debe soportar los objetivos generales:

API P95 < 300 ms
Authentication P95 < 500 ms

Para ello:

PostgreSQL tendrá índices apropiados;
Redis podrá utilizarse para datos temporales y caches;
consultas deberán evitar N+1;
operaciones pesadas serán asíncronas;
eventos serán procesados de forma desacoplada;
operaciones críticas tendrán acceso eficiente a índices.
38. Alternatives Considered
A. PostgreSQL para todo
Rechazada

Simplifica la arquitectura, pero no es óptima para:

rate limiting;
temporal state;
high-frequency caching;
algunos escenarios de sesión;
counters.
B. Redis como fuente principal
Rechazada

No proporciona el modelo de persistencia transaccional y durabilidad requerido para las entidades principales de Identity.

C. Micro-databases independientes por entidad
Rechazada inicialmente

Introduciría una complejidad innecesaria.

Identity mantiene un límite de dominio claro y PostgreSQL proporciona una base transaccional adecuada.

D. PostgreSQL + Redis + Event Platform
Seleccionada

Proporciona:

PostgreSQL
    ↓
Transactional Truth

Redis
    ↓
Fast / Temporary State

Event Platform
    ↓
Asynchronous Integration

manteniendo una arquitectura modular y evolutiva.

39. Consequences
Positivas
Fuente de verdad clara.
Integridad transaccional.
Soporte multi-tenant.
Sesiones persistentes.
Refresh token rotation segura.
Cache de alto rendimiento.
Arquitectura preparada para eventos.
Mejor desacoplamiento entre dominios.
Posibilidad de escalar componentes independientemente.
Negativas
Mayor complejidad operacional.
Necesidad de administrar PostgreSQL y Redis.
Necesidad de manejar consistencia entre DB y eventos.
Necesidad de invalidación de cache.
Mayor complejidad en testing.
Necesidad de observabilidad sobre múltiples componentes.
40. Implementation Direction

La persistencia podría organizarse:

apps/api/app/domains/identity/

└── infrastructure/
    └── persistence/
        ├── models/
        ├── repositories/
        ├── migrations/
        ├── postgres/
        └── outbox/

Y:

apps/api/app/domains/identity/

└── infrastructure/
    └── cache/
        └── redis/

Esta estructura es una dirección de implementación derivada, no una estructura prescrita literalmente por el Blueprint.

41. Testing Requirements

Se deberán probar:

PostgreSQL
CRUD;
constraints;
foreign keys;
unique constraints;
transactions;
migrations;
rollback.
Sessions
creation;
expiration;
revocation;
concurrent access.
Refresh Tokens
rotation;
replay;
reuse detection;
family revocation;
concurrent refresh.
Redis
cache hit;
cache miss;
expiration;
invalidation;
Redis unavailable.
Multi-Tenant
same tenant;
cross tenant;
suspended tenant;
membership removal.
Outbox
transaction success;
transaction rollback;
publish retry;
duplicate event;
DLQ.
Disaster Recovery
database restore;
backup validation;
service recovery;
consistency verification.
42. Traceability
BP-0002 Reference Architecture
        ↓
ESP-0001 Identity
        ↓
ESP-0001 / 02 Identity Architecture
        ↓
ESP-0001 / 06 Sessions & Token Management
        ↓
ESP-0001 / 09 Audit & Compliance
        ↓
ESP-0001 / 11 Persistence & Data Model
        ↓
ESP-0001 / 12 Events & Integration
        ↓
ESP-0001 / 15 Deployment & Operations
        ↓
ADR-IDENTITY-006

La arquitectura de referencia establece explícitamente la utilización de una plataforma de datos que incluye PostgreSQL, Redis, Event Store, Object Storage y otros mecanismos especializados, dentro de una arquitectura modular y orientada a dominios.

43. Follow-up ADRs

Este ADR deja como decisiones posteriores:

ADR-IDENTITY-006.1

PostgreSQL Schema & Physical Data Model

Definir:

tablas definitivas;
columnas;
tipos;
índices;
constraints;
foreign keys.
ADR-IDENTITY-006.2

Audit Persistence & Retention

Definir:

almacenamiento;
inmutabilidad;
retención;
archivado;
eliminación.
ADR-IDENTITY-006.3

MFA Secret Storage & Key Management

Definir:

cifrado;
KMS;
secrets manager;
rotación;
acceso.
ADR-IDENTITY-006.4

Redis Architecture & Cache Policy

Definir:

TTL;
invalidación;
namespaces;
HA;
persistence;
fallback.
ADR-IDENTITY-006.5

Transactional Outbox Implementation

Definir:

esquema;
publisher;
retry;
DLQ;
ordering;
idempotencia.
44. Final Decision Summary

La estrategia de persistencia de Identity queda definida conceptualmente como:

                     IDENTITY
                        │
          ┌─────────────┼─────────────┐
          │             │             │
          ▼             ▼             ▼
     PostgreSQL       Redis       Event Platform
          │             │             │
          │             │             │
     Source of      Temporary       Integration
       Truth           State          & Events
          │
          ▼
   Transactional
      Outbox
          │
          ▼
    Domain Events
Decisión principal

EVOXA Identity utilizará PostgreSQL como fuente de verdad transaccional, Redis como almacenamiento complementario para estado temporal, caching y operaciones de alta frecuencia, y una arquitectura Event-Driven basada en Transactional Outbox para garantizar la consistencia entre cambios persistentes y eventos. Las auditorías tendrán una estrategia de persistencia y retención diferenciada, y los dominios no accederán directamente a las tablas de otros dominios.

Estado: PROPOSED.

Esta decisión permite avanzar posteriormente hacia la definición física de PostgreSQL, Redis y Outbox sin introducir todavía decisiones de infraestructura que el Blueprint no haya establecido como obligatorias.
