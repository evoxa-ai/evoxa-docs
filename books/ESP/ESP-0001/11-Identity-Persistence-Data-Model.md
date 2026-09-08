ESP-0001/11 — Identity Persistence & Data Model

Document ID: ESP-0001/11
Module: Identity
Version: 1.1.0
Status: Draft — Consolidated
Parent Specification: ESP-0001 — Identity
Target Repository: evoxa-platform
Target Path: apps/api/app/domains/identity/

1. Purpose

Este documento define la estrategia de persistencia y el modelo de datos del Identity Domain de EVOXA.

Debe soportar:

Users
Organizations
Memberships
Roles
Permissions
Delegations
Sessions
Refresh Token Families
MFA Factors
MFA Challenges
Recovery Codes
Trusted Devices
Security Policies
Audit Events
Security Events
Outbox Events
Security State / Epochs
Versioning
Tenant Isolation
Concurrency
Recovery
Retention
Observability

El modelo deberá preservar las propiedades de seguridad establecidas por los ADR de Identity.

2. Persistence Principles

Identity Persistence se basa en:

PostgreSQL como fuente transaccional principal.
Domain ownership.
Tenant isolation.
Referential integrity.
Least privilege.
Data minimization.
Security by design.
Immutable historical records cuando corresponda.
Versioning.
Optimistic concurrency.
Transactional consistency.
Transactional Outbox.
Explicit lifecycle.
No plaintext secrets.
Controlled retention.
Disaster recovery.
Auditability.
Evolvability.
3. Storage Architecture

La arquitectura consolidada es:

                         Identity
                            │
              ┌─────────────┼─────────────┐
              │             │             │
              ▼             ▼             ▼
         PostgreSQL       Redis      Event Platform
              │             │             │
              │             │             └── Events
              │             │
              │             └── Cache /
              │                 Temporary State
              │
              └── Authoritative Identity State
PostgreSQL

Es la fuente de verdad para el estado transaccional de Identity.

Incluye:

User
Organization
Membership
Role
Permission
RolePermission
MembershipRole
Delegation
Session
RefreshToken
MFAFactor
MFAChallenge cuando requiera persistencia durable
RecoveryCode
TrustedDevice
SecurityPolicy
AuditEvent
SecurityEvent
OutboxEvent
security/version state
4. Redis

Redis será utilizado como mecanismo de:

cache;
rate limiting;
temporal authentication state;
MFA challenge state cuando sea apropiado;
short-lived security state;
authorization cache.

Redis no es la fuente de verdad de Identity.

Una pérdida completa de Redis deberá ser recuperable desde PostgreSQL y los mecanismos de reconstrucción correspondientes.

5. Event Platform

Event Platform transportará:

Domain Events;
Security Events;
integración de Audit;
cambios relevantes de Identity;
invalidation propagation;
eventos hacia otros dominios.

Los eventos representan hechos, no instrucciones para modificar Redis.

6. Event Store

Identity no adoptará Event Sourcing completo por defecto.

El modelo será:

PostgreSQL
    +
Domain Events
    +
Audit Events
    +
Transactional Outbox

Event Sourcing podrá utilizarse selectivamente si posteriormente existe una justificación arquitectónica explícita.

Esto evita convertir innecesariamente todas las entidades de Identity en event-sourced aggregates.

7. User

User representa la identidad global de una persona.

User
----
id
email
email_normalized
status
created_at
updated_at
version

Estados:

PENDING
ACTIVE
SUSPENDED
DISABLED

Un User no debe contener directamente autorización específica de una organización.

La pertenencia organizacional se representa mediante Membership.

8. User Identity Boundary

Modelo:

User
 │
 ├── Membership → Organization A
 │
 ├── Membership → Organization B
 │
 └── Membership → Organization C

Esto permite que el mismo usuario tenga diferentes capacidades en diferentes organizaciones.

9. User Email

Se recomienda persistir:

email
email_normalized

La unicidad deberá aplicarse sobre la representación normalizada cuando corresponda.

La semántica definitiva de:

case sensitivity;
alias;
email verification;
email change;
historical email;

deberá respetar los contratos de Identity y las políticas de recuperación/verificación.

No se debe utilizar el email como sustituto del user_id.

10. Organization

Organization representa la frontera organizacional/tenant.

Organization
------------
id
name
status
created_at
updated_at
version

Estados:

PENDING
ACTIVE
SUSPENDED

Una organización constituye un boundary de seguridad.

11. Membership

Membership conecta:

User
  ↕
Organization

Modelo:

Membership
----------
id
user_id
organization_id
status
created_at
updated_at
version

Estados:

INVITED
ACTIVE
SUSPENDED
REMOVED

Membership es el access boundary para la pertenencia de un usuario a una organización.

12. Membership Uniqueness

La regla conceptual es:

User + Organization = Membership

No deben existir dos memberships activas equivalentes para el mismo usuario y organización.

La persistencia debe permitir conservar información histórica cuando una membership es removida.

Por ello, la estrategia final de unicidad deberá distinguir:

current active membership

de:

historical membership

sin destruir evidencia histórica.

13. Membership Roles

La relación será:

Membership
     │
     ├── Role A
     ├── Role B
     └── Role C

mediante:

MembershipRole
--------------
membership_id
role_id
created_at
version

Una membership puede tener múltiples roles.

No existe una relación directa:

User → Role

para autorización organizacional.

14. Role
Role
----
id
name
description
scope
organization_id
status
created_at
updated_at
version

Scopes:

SYSTEM
ORGANIZATION

Un Role SYSTEM puede representar capacidades globales.

Un Role ORGANIZATION pertenece a una organización.

15. Permission
Permission
----------
id
resource
action
description
created_at
updated_at
version

Modelo lógico:

resource.action

Ejemplo:

users.read
users.create
users.update
users.delete

training.read
training.create
training.update

resource y action deben corresponder a registros/catalogs controlados.

No se permiten permisos arbitrarios generados dinámicamente por el cliente.

16. RolePermission
Role
 │
 ▼
RolePermission
 │
 ▼
Permission

Modelo:

RolePermission
--------------
role_id
permission_id
created_at
version

Constraint conceptual:

UNIQUE(role_id, permission_id)

La asignación debe respetar tenant scope y authorization.

17. Delegation

Delegation debe persistirse explícitamente.

Modelo conceptual:

Delegation
----------
id
delegator
delegate
organization
scope
status
valid_from
valid_until
created_at
updated_at
version

Una delegación debe ser:

explícita;
limitada;
revocable;
temporal cuando corresponda;
auditable;
tenant-aware.

Regla fundamental:

Delegated Capability
        ≤
Delegator Effective Capability

Nunca debe existir privilege amplification mediante delegation.

18. Session

Session representa una autenticación activa.

Session
-------
id
user_id
organization_id
status
authentication_method
mfa_verified
assurance_level
created_at
last_activity_at
expires_at
revoked_at
metadata
version

Estados:

CREATED
ACTIVE
EXPIRED
REVOKED

Una persona puede tener múltiples sesiones.

19. Session Security State

Session deberá permitir determinar:

usuario;
organización;
membership/contexto;
authentication method;
MFA evidence;
assurance;
expiration;
revocation;
device/client context.

No se almacenarán secrets innecesarios.

20. Refresh Token

Los refresh tokens serán stateful.

RefreshToken
------------
id
session_id
token_family_id
parent_token_id
token_hash
status
issued_at
expires_at
used_at
revoked_at
replaced_by
created_at
version

Estados:

ACTIVE
USED
REVOKED
EXPIRED

El valor original del token no se almacena.

21. Refresh Token Family

La persistencia debe permitir:

Token A
   ↓
Token B
   ↓
Token C
   ↓
Token D

mediante:

token_family_id
parent_token_id
replaced_by

Esto permite detectar reutilización.

Una reutilización confirmada debe permitir revocar:

Token Family
      +
Session

según las reglas establecidas por los ADR de Refresh Token.

22. Atomic Token Rotation

La rotación debe ser concurrency-safe.

Conceptualmente:

BEGIN TRANSACTION

Validate Token
      ↓
Verify ACTIVE
      ↓
Mark USED
      ↓
Create Replacement
      ↓
Update Family
      ↓
Create Security Event / Outbox

COMMIT

Dos requests concurrentes no pueden obtener dos rotaciones válidas independientes del mismo token.

23. MFAFactor
MFAFactor
---------
id
user_id
organization_id
type
status
name
secret_reference
created_at
verified_at
last_used_at
revoked_at
version

Tipos conceptuales:

TOTP
WEBAUTHN
PASSKEY
SMS
EMAIL
PUSH
HARDWARE_KEY

secret_reference es una referencia protegida, no necesariamente el secreto en sí.

24. MFAChallenge
MFAChallenge
------------
id
user_id
session_id
factor_id
purpose
status
attempt_count
created_at
expires_at
verified_at
version

Estados:

CREATED
PENDING
VERIFIED
FAILED
EXPIRED
CANCELLED

Reglas:

created_at <= now < expires_at

Un challenge verificado no puede reutilizarse.

25. MFA Persistence Strategy

No todo estado MFA necesariamente debe tener el mismo storage.

Durable security state
        ↓
PostgreSQL

Short-lived challenge/rate state
        ↓
Redis

Critical verification state
        ↓
Authoritative persistence

Redis no deberá convertirse en la única fuente de verdad de una operación crítica.

26. RecoveryCode
RecoveryCode
------------
id
user_id
code_hash
status
created_at
used_at
revoked_at
version

Estados:

ACTIVE
USED
REVOKED

Los códigos originales no deberán almacenarse después de su generación de manera recuperable.

Una vez utilizados:

ACTIVE → USED

La regeneración deberá invalidar el conjunto anterior según la política establecida.

27. TrustedDevice
TrustedDevice
-------------
id
user_id
device_reference
status
created_at
last_used_at
expires_at
revoked_at
metadata
version

Estados:

ACTIVE
EXPIRED
REVOKED

No debe utilizarse un identificador invasivo del dispositivo si no es necesario.

Trusted Device no equivale a:

permanent authentication;
MFA factor;
active session.
28. SecurityPolicy
SecurityPolicy
--------------
id
name
description
scope
priority
status
version
organization_id
rules
effective_from
effective_until
created_at
updated_at

Lifecycle:

DRAFT
VALIDATING
APPROVED
ACTIVE
SUPERSEDED
RETIRED

La estructura de rules deberá corresponder al modelo declarativo establecido por los ADR del Policy Engine.

No se permite ejecutar:

arbitrary code
arbitrary SQL
network calls

desde una policy.

29. Policy Versioning

La versión de policy debe formar parte del estado de autorización y de la evidencia.

Un cambio de policy de seguridad puede producir:

Policy Version
      ↓
Security Epoch / Version
      ↓
Cache Invalidation
      ↓
Fresh Authorization

La activación de una policy no deberá reutilizar ciegamente resultados de autorización calculados bajo una versión anterior.

30. AuditEvent
AuditEvent
----------
id
event_type
event_version
timestamp
actor_type
actor_id
tenant_id
organization_id
membership_id
target_type
target_id
action
result
session_reference
correlation_id
causation_id
trace_id
metadata
version

El AuditEvent debe ser append-oriented.

No se deben realizar updates normales sobre registros históricos.

31. SecurityEvent

Security Events deben poder representar eventos como:

FailedLogin
RefreshTokenReuseDetected
AccountSuspended
MFARevoked
CrossTenantAccessAttempt
PrivilegeEscalationAttempt
SessionRevoked
PolicySecurityChange

Pueden integrarse con Event Platform y sistemas de monitoring/SIEM.

32. Audit vs Domain Events vs Security Events

Son conceptos distintos:

Domain Event
    ↓
Business fact

Security Event
    ↓
Security-relevant fact

Audit Event
    ↓
Accountability / compliance evidence

Application Log
    ↓
Technical diagnostic information

No deben fusionarse automáticamente en una única tabla conceptual.

33. OutboxEvent

Identity deberá persistir eventos transaccionales mediante Transactional Outbox.

Modelo conceptual:

OutboxEvent
-----------
id
event_type
event_version
aggregate_type
aggregate_id
tenant_id
occurred_at
payload
metadata
status
attempt_count
published_at
created_at

La transacción será:

Business State
      +
Outbox Event
      ↓
Same Transaction

Posteriormente:

Outbox
   ↓
Publisher
   ↓
Event Platform

Esta es una de las decisiones ya establecidas por los ADR posteriores y, por tanto, el texto original que la dejaba pendiente debe actualizarse.

34. Outbox Guarantees

El sistema deberá soportar:

At-Least-Once delivery;
retries;
DLQ;
idempotent consumers;
replay control;
scoped ordering;
observability.

Los eventos no deben contener:

passwords;
tokens;
MFA secrets;
recovery codes;
private keys;
API secrets.
35. Security Epoch

Identity deberá mantener mecanismos de version/epoch para invalidar estados derivados de autorización.

Conceptualmente:

security_epoch

puede existir a nivel:

Platform
Organization
Membership
User
Session
Security State

según el scope necesario.

Regla:

Cache Epoch < Authoritative Epoch
            ↓
          STALE

Una revocación de seguridad tiene prioridad sobre TTL.

36. Entity Versioning

Las entidades administrativas relevantes tendrán:

version

Ejemplo:

Role version = 4

Actualización:

UPDATE roles
SET version = 5
WHERE id = ?
  AND version = 4;

Si no se actualiza ninguna fila:

CONCURRENCY_CONFLICT

Esto evita que dos administradores sobrescriban silenciosamente cambios.

37. Referential Integrity

Foreign keys deberán proteger las relaciones críticas.

Ejemplo:

Membership.user_id
    → User.id

Membership.organization_id
    → Organization.id

Role.organization_id
    → Organization.id

RolePermission.role_id
    → Role.id

RolePermission.permission_id
    → Permission.id

MembershipRole.membership_id
    → Membership.id

MembershipRole.role_id
    → Role.id

Session.user_id
    → User.id

RefreshToken.session_id
    → Session.id

MFAFactor.user_id
    → User.id
38. Cascade Strategy

Identity no utilizará ON DELETE CASCADE indiscriminadamente.

Especialmente para:

Audit;
Security Events;
Membership history;
Security evidence;
Outbox;
historical records.

Una eliminación de una entidad no debe destruir automáticamente evidencia que deba conservarse.

Cada FK tendrá una política de delete explícita.

39. Soft Delete

No todas las entidades deben utilizar soft delete.

Debe diferenciarse:

Lifecycle State

de:

Physical Deletion

Ejemplo:

User
 ↓
DISABLED

no significa:

DELETE FROM users;

El borrado físico estará gobernado por:

retention;
privacy;
compliance;
legal requirements;
referential integrity.
40. Tenant Isolation

El aislamiento tenant es una propiedad fundamental del modelo.

Los datos organization-scoped deberán incluir o resolver inequívocamente:

organization_id

No deberá dependerse únicamente de un filtro del frontend.

La aplicación deberá garantizar:

Authenticated Context
        ↓
Tenant Context
        ↓
Database Query
41. Tenant-Scoped Persistence

Una consulta conceptualmente correcta:

SELECT *
FROM memberships
WHERE organization_id = :authorized_organization;

El authorized_organization deberá provenir del contexto autorizado.

No se deberá confiar ciegamente en:

organization_id

enviado por el cliente.

42. Database Security

La base de datos deberá aplicar:

least privilege;
encrypted connections;
credential management externo;
restricted database users;
migration controls;
audit;
backup encryption;
controlled administrative access.

Las credenciales de PostgreSQL nunca deberán almacenarse en Git.

43. Index Strategy

Índices iniciales previstos:

User
email_normalized
status
created_at
Organization
status
created_at
Membership
user_id
organization_id
status
(user_id, organization_id)
Role
organization_id
scope
status
Permission
resource
action
RolePermission
role_id
permission_id
Session
user_id
status
expires_at
organization_id
RefreshToken
session_id
token_family_id
token_hash
status
expires_at
MFA
user_id
status
type
Audit
tenant_id
actor_id
event_type
timestamp
target_id
correlation_id

Los índices finales deberán validarse mediante profiling y workload real.

44. Unique Constraints

Restricciones conceptuales:

User.email_normalized

Permission(resource, action)

RolePermission(role_id, permission_id)

Membership(user_id, organization_id)

MembershipRole(membership_id, role_id)

La estrategia definitiva de membership histórica debe permitir preservar registros removidos sin crear duplicados activos.

45. Transactions

Las operaciones que modifican múltiples estados relacionados deben ejecutarse de manera transaccional.

Ejemplo:

Assign Role
    │
    ├── Validate Actor
    ├── Validate Tenant
    ├── Validate Membership
    ├── Validate Role
    ├── Create MembershipRole
    ├── Update Security Version
    ├── Create Audit Event
    └── Create Outbox Event

Las operaciones críticas deberán mantener consistencia transaccional.

46. Authorization Cache Persistence

Authorization cache no pertenece al modelo transaccional principal.

PostgreSQL
    ↓
Authoritative Authorization State

Redis
    ↓
Derived Authorization Cache

El cache deberá considerar:

tenant;
actor;
membership;
resource;
action;
policy version;
entity version;
security epoch;
context fingerprint;
cache generation;
freshness;
TTL;
invalidation state.
47. Cache Invalidation

Un cambio de seguridad seguirá conceptualmente:

Database Change
      ↓
Security Version / Epoch
      ↓
Transactional Outbox
      ↓
Event Platform
      ↓
Invalidation Consumer
      ↓
L1 / L2 Invalidation

El consumer no ejecuta autorización.

El Authorization Runtime sigue siendo la autoridad para la decisión.

48. Criticality Persistence

La base de datos no debe confundir:

Authorization Decision

con:

Criticality

Criticality es una propiedad derivada del contexto y policy.

Conceptualmente:

Effective Criticality =
MAX(
  Base,
  Dynamic Risk,
  Context,
  Security State,
  Tenant,
  Delegation,
  MFA,
  Platform Minimum
)

Esto puede requerir persistencia de versiones/evidencia, pero no significa almacenar necesariamente cada resultado como estado permanente.

49. Historical Security Evidence

Para operaciones sensibles debe conservarse evidencia suficiente para reconstruir:

Who
What
When
Where
Why
Tenant
Policy Version
Runtime Version
Criticality
Freshness
Result
Correlation

La evidencia debe respetar minimización de datos.

50. Data Retention

La retención deberá diferenciar:

Operational State
Security State
Audit
Security Events
Test Evidence
Outbox
Historical Data

No se debe aplicar una única política de retention a todo Identity.

Las políticas concretas se alinearán con ADR-013 y requisitos regulatorios aplicables.

51. Encryption

Datos sensibles deberán protegerse mediante mecanismos apropiados.

Particularmente:

MFA secrets;
token hashes/protected representations;
recovery code hashes;
sensitive authentication material.

Las claves criptográficas deberán gestionarse mediante KMS/Secret Manager.

La base de datos no deberá convertirse en un almacén de secretos plaintext.

52. Backup & Recovery

PostgreSQL deberá soportar:

backups;
PITR;
encrypted backups;
restore testing;
integrity validation.

Objetivos de arquitectura:

RTO < 1 hour
RPO < 15 minutes

La recuperación deberá validar:

users;
organizations;
memberships;
roles;
permissions;
sessions;
refresh token state;
MFA;
policies;
audit;
security events;
outbox;
tenant isolation.
53. Migration Strategy

Las modificaciones del modelo se realizarán mediante migraciones versionadas.

No se deberá depender de:

automatic schema sync

como mecanismo de evolución de producción.

Las migraciones deberán:

ser versionadas;
ser reproducibles;
ser revisables;
soportar rollback cuando sea seguro;
preservar datos;
ser compatibles con zero-downtime cuando corresponda.
54. Data Migration Safety

Una migración que afecte:

tenant isolation;
authorization;
MFA;
sessions;
refresh tokens;
security policies;

deberá tener pruebas de seguridad específicas.

No se permitirá una migración que provoque:

Privilege Escalation
Cross-Tenant Access
Security State Downgrade
55. Persistence and Domain Boundaries

Cada Identity module es dueño de su estado.

Account
Organization
Membership
Authentication
Authorization
Role & Permission
Policy
Session
Token
MFA
Recovery
Federation
Provisioning
Audit
Security Events
Integration

Otros módulos no deben acceder directamente a las tablas de Identity.

Deben utilizar:

Public APIs
+
Domain/Integration Events
56. Repository Boundaries

La estructura deberá mantener:

API
Application
Domain
Infrastructure

Los repositories pertenecen al módulo que posee el aggregate.

No se permitirá:

Module A
   ↓
Module B Repository

de manera directa.

La interacción deberá realizarse mediante contratos.

57. Aggregate Persistence

Aggregates principales:

User
Organization
Membership
Role
Session
RefreshTokenFamily
MFAFactor
MFAChallenge
SecurityPolicy
Delegation

No significa que cada aggregate deba convertirse en una tabla única.

Un aggregate puede utilizar varias tablas dentro de su boundary transaccional.

58. Persistence Model Overview

El modelo consolidado queda:

                         User
                          │
             ┌────────────┼────────────┐
             │            │            │
             ▼            ▼            ▼
        Membership     Session      MFAFactor
             │            │            │
             │            ▼            ├── Challenge
             │       RefreshToken      └── RecoveryCode
             │
             ▼
       Organization
             │
             ▼
           Roles
             │
             ▼
       RolePermission
             │
             ▼
        Permissions

User
 │
 └── TrustedDevice

Membership
 │
 └── Delegation

Organization
 │
 └── SecurityPolicy

Identity State
 │
 ├── AuditEvent
 ├── SecurityEvent
 └── OutboxEvent
59. Security Invariants

La persistencia deberá garantizar como mínimo:

PERSIST-001 — No cross-tenant data access.

PERSIST-002 — No plaintext refresh tokens.

PERSIST-003 — No plaintext MFA secrets.

PERSIST-004 — No recovery codes in plaintext.

PERSIST-005 — Referential integrity for critical relationships.

PERSIST-006 — No duplicate active membership.

PERSIST-007 — No duplicate role-permission assignment.

PERSIST-008 — Refresh token rotation is atomic.

PERSIST-009 — Refresh token reuse can revoke its family.

PERSIST-010 — Security state cannot silently regress.

PERSIST-011 — Audit history cannot be casually modified.

PERSIST-012 — Outbox state is transactionally consistent with domain state.

PERSIST-013 — Cache is never the source of truth.

PERSIST-014 — Entity versions prevent unsafe concurrent updates.

PERSIST-015 — Security epochs cannot regress.

PERSIST-016 — Historical security evidence cannot be destroyed by normal cascades.

PERSIST-017 — AI cannot modify persistence security boundaries outside authorized contracts.

60. Testing Requirements

El modelo deberá tener pruebas:

Unit
entity invariants;
value objects;
state transitions.
Integration
PostgreSQL;
foreign keys;
transactions;
migrations;
concurrency.
Security
tenant isolation;
privilege escalation;
token reuse;
MFA state;
revoked session;
security epoch.
Contract
repository contracts;
persistence adapters;
event schemas.
Performance
authorization lookup;
membership lookup;
session lookup;
token lookup;
audit queries.
Recovery
backup;
PITR;
restore;
cache loss;
outbox replay.

El Blueprint exige automatización para capacidades críticas y contempla unit, integration, contract, security y performance testing.

61. Observability

Persistence debe exponer métricas sobre:

DB latency
DB errors
connection pool
transaction duration
deadlocks
lock contention
migration status
outbox backlog
event publication latency
cache hit/miss
cache invalidation lag
security epoch propagation
audit writes

Nunca deben incluirse secretos en telemetry.

62. Implementation Readiness
Capability	Estado
PostgreSQL authoritative source	🟢
User	🟢
Organization	🟢
Membership	🟢
Membership Roles	🟢
Role	🟢
Permission	🟢
RolePermission	🟢
Delegation	🟢
Session	🟢
Refresh Token	🟢
Refresh Token Family	🟢
MFA Factor	🟢
MFA Challenge	🟢
Recovery Codes	🟢
Trusted Devices	🟢
Security Policies	🟢
Audit	🟢
Security Events	🟢
Transactional Outbox	🟢
Security Epoch	🟢
Optimistic Concurrency	🟢
Tenant Isolation	🟢
Cache model	🟢
Encryption strategy	🟢
Backup/Recovery	🟢
Migration strategy	🟢
Exact PostgreSQL DDL	🟡
Exact indexes after profiling	🟡
Exact retention periods	🟡
Exact policy schema	🟡
Exact event schemas	🟡
63. Remaining Implementation Decisions

Aquí sí dejamos solamente decisiones que todavía pertenecen a la fase de implementación:

PostgreSQL exact version.
UUID implementation.
exact table names;
exact column types;
exact enum representation;
JSONB usage;
exact indexes;
partitioning where justified;
exact retention configuration;
migration tooling;
connection pool configuration;
KMS integration;
exact security epoch storage strategy;
exact Redis key structures;
exact Outbox schema;
exact Event Platform adapter.

Estas decisiones ya no requieren crear otro ADR simplemente porque faltan detalles de implementación.

64. Traceability
Specification / ADR	Relación
ESP-0001/01	Domain Model
ESP-0001/03	Authentication & Authorization
ESP-0001/05	Roles & Permissions
ESP-0001/06	Sessions & Tokens
ESP-0001/07	MFA
ESP-0001/08	Security Policies
ESP-0001/09	Audit & Compliance
ESP-0001/10	API Contracts
ESP-0001/12	Events
ESP-0001/13	Testing
ESP-0001/14	Implementation
ESP-0001/15	Deployment
ADR-006	PostgreSQL Persistence
ADR-007	Transactional Outbox
ADR-012	PostgreSQL Schema
ADR-013	Retention
ADR-014	Encryption
ADR-015	Partitioning
ADR-016	Concurrency
ADR-017	Migrations
ADR-022	Consumer Idempotency
ADR-025	Refresh Token Cryptography
ADR-043	Multi-Organization Identity
ADR-044	Membership Roles/Delegation
ADR-050–059	Authorization Cache/Criticality
ADR-062–067	Testing/Evidence Contracts
65. Final Assessment
Documento original

ESP-0001/11 original: 🟡

La base conceptual es buena y cubre prácticamente todas las entidades principales, pero contiene varias decisiones que en el momento de su creación estaban abiertas.

Por ejemplo, el documento original todavía indicaba que la separación entre PostgreSQL, Redis y Event Store debía cerrarse mediante un ADR y que Transactional Outbox era una propuesta que debía resolverse posteriormente.

Versión consolidada 1.1.0

ESP-0001/11 v1.1.0: 🟢 Arquitecturalmente alineado

Ahora el modelo queda coherente con los ADR que desarrollamos posteriormente:

PostgreSQL
   ↓
Source of Truth
   ↓
Transactions
   ↓
Outbox
   ↓
Event Platform
   ↓
Cache / Other Domains

y:

Identity Persistence
       ↓
Tenant Isolation
       ↓
Security State
       ↓
Versioning
       ↓
Authorization
       ↓
Audit
       ↓
Recovery

No recomiendo crear un ADR-070 solo para esta etapa. Ya tenemos las decisiones arquitectónicas necesarias; ahora corresponde llevar estas definiciones hacia ESP-0001/12 — Identity Events & Integration y posteriormente /13 Testing, /14 Implementation y /15 Deployment.

Además, esto nos deja muy cerca de pasar de documentación arquitectónica a la fase realmente importante: IS → Technical Tasks → PostgreSQL migrations → código del backend.
