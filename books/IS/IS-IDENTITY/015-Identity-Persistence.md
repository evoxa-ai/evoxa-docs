IS-IDENTITY-015 — Identity Persistence

Estado: Draft
Prioridad: Critical
Dominio: Identity
Capability: Persistence / Data Model
ESP principal: ESP-0001
ADR principal: ADR-IDENTITY-006 — PostgreSQL as Transactional Source of Truth
ADRs relacionados: ADR-IDENTITY-003, 007, 008, 010, 011
Dependencias: IS-IDENTITY-001 → IS-IDENTITY-014
Arquitectura: Domain → Application → Infrastructure → PostgreSQL

1. Propósito

Definir la estrategia formal de persistencia del dominio Identity de EVOXA, estableciendo:

entidades persistentes;
relaciones;
claves primarias;
claves foráneas;
constraints;
índices;
unicidad;
estados;
integridad referencial;
transacciones;
concurrencia;
migraciones;
auditoría;
Outbox;
retención;
aislamiento por tenant;
estrategia PostgreSQL/Redis.

Esta IS constituye el puente entre el modelo de dominio definido en las IS anteriores y la futura implementación de Infrastructure.

Importante: esta especificación define el modelo de persistencia; no constituye todavía la implementación de modelos ORM, repositories o migraciones de código.

2. Business Goal

Proporcionar a Identity una capa de persistencia:

consistente + transaccional + segura + multi-tenant + observable + escalable + recuperable.

La persistencia debe garantizar especialmente:

integridad de identidad;
aislamiento entre organizaciones;
consistencia de Memberships;
consistencia de Roles/Permissions;
seguridad de Sessions y Refresh Tokens;
persistencia confiable de MFA;
integridad de Security Policies;
Audit append-only;
Transactional Outbox;
recuperación ante fallos.
3. Fundamento arquitectónico

Según el Blueprint y las decisiones de Identity:

PostgreSQL
    │
    └── Source of Truth

Mientras:

Redis
    │
    ├── cache
    ├── temporary state
    └── high-frequency operations

Redis no reemplaza PostgreSQL como fuente transaccional primaria.

Esta separación deriva de ADR-IDENTITY-006.

4. Alcance
4.1 In Scope

Esta IS define persistencia para:

User
Organization
Membership
Role
Permission
RolePermission
MembershipRole
Session
RefreshToken
MFAFactor
MFAChallenge
RecoveryCode
TrustedDevice
SecurityPolicy
AuditEvent
SecurityEvent
OutboxEvent
4.2 Out of Scope

No se define todavía:

SQL final;
ORM específico;
código de migrations;
implementación concreta de repositories;
proveedor cloud de PostgreSQL;
proveedor de Redis;
proveedor del Event Platform;
configuración física de Kubernetes;
particionamiento físico definitivo;
backup provider específico.
5. Modelo general

La relación principal será:

User
 │
 ├───────────────┐
 │               │
 ▼               ▼
Membership     Session
 │
 ▼
Organization
 │
 └── Membership
        │
        ▼
       Role
        │
        ▼
 RolePermission
        │
        ▼
    Permission

Y alrededor:

Session
 │
 └── RefreshToken

User
 │
 ├── MFAFactor
 ├── MFAChallenge
 ├── RecoveryCode
 └── TrustedDevice

Organization/User/Role
 │
 └── SecurityPolicy

All critical operations
 │
 ├── AuditEvent
 ├── SecurityEvent
 └── OutboxEvent
6. Persistence Model
6.1 Users

Tabla conceptual:

users

Campos:

Campo	Tipo conceptual	Requerido
id	UUID	Sí
email	Email/String	Sí
password_hash	String	Condicional
status	Enum	Sí
created_at	Timestamp	Sí
updated_at	Timestamp	Sí
activated_at	Timestamp	No
suspended_at	Timestamp	No
disabled_at	Timestamp	No
metadata	JSON	No

El nombre y tipo físico definitivo quedan para Data Model/Technical Tasks.

Reglas
id único.
email debe tener constraint de unicidad según la política de identidad.
password nunca se almacena en texto plano.
password_hash nunca se expone mediante API.
lifecycle debe ser consistente con IS-IDENTITY-002.
7. Email Uniqueness

Se debe definir explícitamente el scope de unicidad de email.

Existen dos posibilidades:

Global
UNIQUE(email)
Contextual/tenant
UNIQUE(organization_id, email)

Sin embargo, dado que el modelo establecido permite que un User pertenezca a múltiples organizaciones, la segunda opción no representa correctamente la identidad global del usuario.

Por lo tanto, la opción arquitectónicamente coherente con el modelo actual es:

User es global; Membership determina pertenencia al tenant.

Por ello se propone email globalmente único, sujeto a confirmación del contrato definitivo de Identity.

8. Email Normalization

La comparación de emails debe utilizar una política consistente.

Debe evitarse:

User@example.com
user@example.com

como dos identidades accidentalmente distintas.

La estrategia exacta de normalización/case folding debe quedar definida antes de la migration final.

9. Organizations

Tabla conceptual:

organizations

Campos:

Campo	Propósito
id	OrganizationId
name	Nombre
status	Lifecycle
created_at	Creación
updated_at	Actualización
metadata	Metadata

Lifecycle:

PENDING
   │
   ├── ACTIVE
   │
   └── SUSPENDED

ACTIVE ↔ SUSPENDED
10. Organization como Tenant Boundary

organization_id constituye el límite principal de tenant.

Las entidades que pertenecen directamente a una organización deben conservar referencia al tenant cuando corresponda.

Ejemplo:

Membership.organization_id
Session.organization_id
SecurityPolicy.organization_id
AuditEvent.tenant_id
OutboxEvent.tenant_id

No todas las entidades necesitan tener un organization_id directo si el tenant puede determinarse de manera segura por una relación.

La decisión debe evitar duplicación innecesaria, pero favorecer aislamiento explícito donde sea crítico.

11. Memberships

Tabla:

memberships

Campos conceptuales:

Campo	Propósito
id	MembershipId
user_id	Usuario
organization_id	Organización
status	Lifecycle
created_at	Creación
updated_at	Actualización
activated_at	Activación
removed_at	Eliminación
metadata	Metadata

Constraint fundamental:

UNIQUE(user_id, organization_id)

Debe impedir múltiples Memberships activas/duplicadas para el mismo usuario y organización.

12. Membership Lifecycle
INVITED
   │
   ├── ACTIVE
   │
   └── REMOVED

ACTIVE
   ├── SUSPENDED
   └── REMOVED

SUSPENDED
   ├── ACTIVE
   └── REMOVED

La persistencia debe permitir detectar estados inválidos mediante lógica de dominio y constraints donde sea apropiado.

13. Roles

Tabla:

roles

Campos:

Campo	Propósito
id	RoleId
name	Nombre
description	Descripción
scope	SYSTEM / ORGANIZATION
status	Estado
organization_id	Tenant cuando aplique
created_at	Creación
updated_at	Actualización
Regla

Los roles de organización deben estar aislados por tenant.

Los roles SYSTEM pertenecen a la plataforma.

14. Permission

Tabla:

permissions

Campos:

Campo	Propósito
id	PermissionId
resource	Recurso
action	Acción
description	Descripción
status	Estado
created_at	Creación
updated_at	Actualización

La identidad lógica será:

resource.action

Ejemplo:

users.read
users.create
users.update
users.delete
15. RolePermission

Tabla:

role_permissions

Campos:

Campo	Propósito
role_id	Role
permission_id	Permission
created_at	Timestamp

Constraint:

UNIQUE(role_id, permission_id)

No deben existir duplicados.

16. MembershipRole

Aunque algunas versiones iniciales del modelo podían representar un único role directamente en Membership, la arquitectura Identity establecida requiere soportar:

Membership
   │
   ├── Role A
   ├── Role B
   └── Role C

Por ello se propone:

membership_roles

Campos:

membership_id
role_id
created_at

Constraint:

UNIQUE(membership_id, role_id)

Esto permite múltiples roles sin crear una dependencia rígida entre Membership y un único Role.

17. Authorization Persistence

El modelo efectivo será:

User
  ↓
Membership
  ↓
MembershipRole
  ↓
Role
  ↓
RolePermission
  ↓
Permission

Pero Authorization no deberá depender exclusivamente de estas tablas.

También debe incorporar:

ABAC
+
Ownership
+
Tenant Isolation
+
Security Policies
+
Session Context
+
MFA
+
Risk

Por tanto, Persistence proporciona datos para Authorization, pero no implementa por sí sola la decisión de autorización.

18. Sessions

Tabla:

sessions

Campos conceptuales:

Campo	Propósito
id	SessionId
user_id	Usuario
organization_id	Organización
membership_id	Membership
status	Estado
created_at	Creación
activated_at	Activación
last_activity_at	Última actividad
expires_at	Expiración
revoked_at	Revocación
revoke_reason	Motivo
auth_method	Método
mfa_status	Estado MFA
device_context	Contexto dispositivo
client_context	Cliente
metadata	Metadata
19. Session Constraints

Debe existir integridad referencial hacia:

user
organization
membership

cuando correspondan.

Una Session no debe continuar siendo efectiva si:

User = DISABLED
Membership = REMOVED
Organization = SUSPENDED
Session = REVOKED

Esta regla se aplica principalmente en Application/Domain/Authorization, no solamente mediante FK.

20. Refresh Tokens

Tabla:

refresh_tokens

Campos conceptuales:

Campo	Propósito
id	RefreshTokenId
session_id	Session
family_id	Token Family
parent_token_id	Parent
token_hash	Hash
status	Estado
issued_at	Emisión
expires_at	Expiración
used_at	Uso
revoked_at	Revocación
revoke_reason	Motivo
created_at	Creación
metadata	Metadata
21. Refresh Token Security

La persistencia no debe almacenar el refresh token en texto plano.

Debe almacenar una representación segura suficiente para:

validación;
detección;
rotación;
revocación;
reuse detection.

La estrategia criptográfica exacta debe quedar alineada con ADR-IDENTITY-008 y una Technical Task específica.

22. Token Family

Los refresh tokens deben permitir reconstruir:

Family
 │
 ├── Token 1
 │      ↓
 ├── Token 2
 │      ↓
 ├── Token 3
 │      ↓
 └── Token 4

family_id será fundamental para:

rotation;
reuse detection;
family revocation.
23. MFA Factors

Tabla:

mfa_factors

Campos conceptuales:

Campo	Propósito
id	Factor ID
user_id	Usuario
factor_type	Tipo
status	Estado
created_at	Creación
activated_at	Activación
revoked_at	Revocación
metadata	Metadata cifrada/no sensible

Los secretos MFA requieren protección criptográfica y acceso extremadamente restringido.

24. MFA Challenges

Tabla:

mfa_challenges

Campos:

id
user_id
session_id
factor_id
purpose
status
created_at
expires_at
verified_at
attempt_count
metadata

Debe existir relación con User y, cuando corresponda, Session/Factor.

25. Recovery Codes

Tabla:

recovery_codes

No se deben almacenar códigos en texto plano.

Se debe persistir una representación segura que permita verificar su uso.

Campos conceptuales:

id
user_id
code_hash
status
used_at
created_at
metadata
26. Trusted Devices

Tabla:

trusted_devices

Campos conceptuales:

id
user_id
device_identifier
status
created_at
expires_at
revoked_at
metadata

El identificador no debe ser tratado como secreto de autenticación permanente.

27. Security Policies

Tabla:

security_policies

Campos conceptuales:

Campo	Propósito
id	Policy ID
name	Nombre
description	Descripción
scope	Scope
organization_id	Tenant
status	Estado
priority	Prioridad
version	Versión
rules	Reglas
effective_from	Inicio
effective_until	Fin
created_at	Creación
updated_at	Actualización
metadata	Metadata
28. Policy Versioning

Las políticas deben ser versionadas.

Ejemplo:

Policy
 ├── v1
 ├── v2
 └── v3

Las decisiones de autorización deben poder identificar:

policy_id
policy_version

Esto resulta importante para auditoría y forensic analysis.

29. Audit Events

Tabla conceptual:

audit_events

Campos:

id
event_type
event_version
timestamp
tenant_id
actor
resource
operation
result
reason
correlation_id
causation_id
trace_id
source
metadata
30. Audit Append-Only

Audit debe comportarse como:

INSERT
  ↓
Audit Event
  ↓
Immutable

No se debe permitir el flujo normal:

UPDATE audit_event
DELETE audit_event

La modificación de datos históricos debe quedar fuera del modelo normal.

31. Security Events

Se puede utilizar una estructura compatible con Audit, pero conceptualmente representa otra responsabilidad.

Ejemplos:

AuthorizationDenied
TenantAccessDenied
PrivilegeEscalationDetected
RefreshTokenReuseDetected
PolicyViolationDetected

Estos eventos pueden alimentar mecanismos de:

detección;
alertas;
respuesta;
análisis de seguridad.
32. Outbox Events

Tabla:

outbox_events

Campos:

id
event_id
event_type
event_version
occurred_at
tenant_id
aggregate_type
aggregate_id
actor
correlation_id
causation_id
trace_id
payload
metadata
status
attempt_count
next_attempt_at
published_at
created_at

Esta tabla es crítica para Transactional Outbox definido en IS-IDENTITY-013.

33. Transactional Boundary

Cuando una operación modifica estado y genera evento:

BEGIN

  UPDATE domain state

  INSERT INTO outbox_events

COMMIT

Debe ser una única unidad transaccional.

Esto es un requisito crítico.

34. Referential Integrity

Las Foreign Keys deberán utilizarse para proteger relaciones estructurales.

Ejemplo:

membership.user_id
      ↓
users.id
membership.organization_id
      ↓
organizations.id
role_permission.role_id
      ↓
roles.id

La estrategia ON DELETE debe definirse explícitamente y evitar cascadas peligrosas.

35. Delete Strategy

Para entidades críticas de Identity no se recomienda asumir eliminación física como comportamiento por defecto.

Por ejemplo:

User
Organization
Membership
Session
Role
Permission

deben preservar integridad histórica cuando sea necesario.

El lifecycle debe utilizar estados:

ACTIVE
SUSPENDED
DISABLED
REVOKED
REMOVED

según la entidad.

La política definitiva de hard-delete/retention queda pendiente.

36. Indexing Strategy

Se deberán definir índices sobre las rutas de acceso reales.

Users
email
status
created_at
Organizations
status
created_at
Membership
user_id
organization_id
status
(user_id, organization_id)
Roles
organization_id
scope
status
Permissions
resource
action
Sessions
user_id
organization_id
status
expires_at
Refresh Tokens
session_id
family_id
status
expires_at
Audit
tenant_id
timestamp
actor_id
event_type
resource_type
resource_id
correlation_id
trace_id
Outbox
status
next_attempt_at
created_at
event_id
tenant_id
aggregate_id

Los índices definitivos deberán validarse mediante query patterns y performance tests.

37. Unique Constraints

Como mínimo deben evaluarse:

users.email

memberships(user_id, organization_id)

role_permissions(role_id, permission_id)

membership_roles(membership_id, role_id)

permissions(resource, action)

También deben definirse unicidades para entidades auxiliares cuando el negocio lo requiera.

38. Concurrency

Identity contiene operaciones sensibles a concurrencia.

Especialmente:

Refresh Token Rotation

Dos solicitudes simultáneas no deben poder rotar exitosamente el mismo token.

Role Assignment

Dos requests concurrentes no deben crear duplicados.

Membership

Dos requests no deben crear Memberships duplicadas.

MFA

Dos operaciones concurrentes no deben producir estados inconsistentes.

Session Revocation

La revocación debe ser segura frente a requests concurrentes.

39. Transaction Isolation

La implementación deberá utilizar el nivel de aislamiento apropiado para cada operación.

No se debe elevar indiscriminadamente el isolation level.

Las operaciones críticas deberán diseñarse considerando:

race conditions
lost updates
duplicate writes
token reuse

Los valores concretos de PostgreSQL y locks quedan para Technical Tasks.

40. Optimistic Concurrency

Para entidades donde sea necesario controlar modificaciones concurrentes puede utilizarse un mecanismo de:

version

o equivalente.

No todas las entidades necesitan obligatoriamente version column.

La estrategia debe determinarse por caso de uso.

41. Redis

Redis podrá utilizarse para:

cache;
rate limiting;
challenges temporales;
datos efímeros;
locks distribuidos cuando esté justificado;
información de alta frecuencia.

Pero:

Redis no deberá convertirse en fuente única de verdad para Identity.

Datos críticos deben permanecer en PostgreSQL.

42. Cache Invalidation

Los cambios de:

Role
Permission
Membership
Organization
User
SecurityPolicy

pueden invalidar caches de autorización.

Ejemplo:

RolePermissionChanged
       │
       ▼
Invalidate Authorization Cache

La estrategia concreta se coordinará con IS-IDENTITY-006/011.

43. Migration Strategy

Las modificaciones de esquema deben realizarse mediante migrations versionadas.

Conceptualmente:

Migration 001
Migration 002
Migration 003
...

Nunca depender exclusivamente de:

auto-create tables

en producción.

La sincronización automática del ORM no debe ser el mecanismo definitivo de evolución del schema productivo.

44. Backward Compatibility

Las migrations deben considerar:

old application
      ↓
migration
      ↓
new application

cuando se realicen deployments rolling/blue-green.

Las modificaciones incompatibles deberán utilizar estrategias como:

expand
   ↓
migrate
   ↓
contract

cuando sea necesario.

45. Database Transactions

Las operaciones que modifican múltiples aggregates/tables deberán definir explícitamente su boundary transaccional.

Ejemplo:

Create Membership
BEGIN
  membership
  membership roles
  outbox event
COMMIT
Assign Permission
BEGIN
  role_permission
  outbox event
  audit/security event
COMMIT

La transacción no debe incluir operaciones externas lentas.

46. External Systems

Nunca se deberá mantener una transacción PostgreSQL abierta esperando:

HTTP external service
Email provider
AI service
Event broker

La comunicación externa debe resolverse mediante:

Outbox;
asynchronous processing;
retry.
47. Audit Consistency

Cuando una operación requiera audit obligatorio:

Business State
+
Audit/Outbox

debe tener una estrategia consistente de persistencia.

La implementación concreta deberá decidir si el Audit Event se escribe:

directamente dentro de la misma transacción;
como evento derivado mediante Outbox;
o mediante una combinación controlada.

No se debe permitir que una operación crítica tenga éxito silenciosamente perdiendo su audit obligatorio.

48. Data Protection

Los datos sensibles deben clasificarse.

Nunca en texto plano
password
refresh token
MFA secret
recovery code
private key
Alta sensibilidad
email
device information
security metadata
audit metadata
Metadata operacional
timestamps
correlation IDs
trace IDs

La clasificación definitiva debe alinearse con Security Standards y Privacy.

49. Encryption

La arquitectura debe contemplar:

At Rest

Protección de PostgreSQL y backups.

Field-level / application-level

Para secretos especialmente sensibles.

In Transit

TLS entre:

API ↔ PostgreSQL
API ↔ Redis
API ↔ Event Platform

La elección exacta de qué campos requieren application-level encryption queda pendiente de ADR-IDENTITY-017.

50. Backup & Recovery

PostgreSQL deberá formar parte de la estrategia global de Disaster Recovery.

Objetivos definidos en el Blueprint:

RTO < 1h
RPO < 15m

La implementación de backups y restore deberá validarse mediante pruebas, no solamente configuración declarativa.

51. Partitioning

No se deberá introducir particionamiento prematuramente.

Entidades que potencialmente podrían necesitarlo por crecimiento:

AuditEvent
SecurityEvent
OutboxEvent

La decisión debe basarse en:

volumen;
retention;
query patterns;
performance;
operational complexity.
52. Retention

Las políticas definitivas de retención quedan pendientes para:

Audit;
Security Events;
Outbox;
Sessions;
Refresh Tokens;
MFA Challenges.

No se deben inventar períodos exactos en esta IS.

53. Observabilidad de Persistence

Se deberán monitorizar:

connection pool;
query latency;
transaction latency;
locks;
deadlocks;
failed transactions;
slow queries;
database availability;
replication lag cuando exista;
storage;
index health;
outbox backlog.
54. Technical Tasks
Data Model

T01 — Definir schema Identity.
T02 — Definir users table.
T03 — Definir organizations table.
T04 — Definir memberships table.
T05 — Definir roles table.
T06 — Definir permissions table.
T07 — Definir role_permissions.
T08 — Definir membership_roles.
T09 — Definir sessions.
T10 — Definir refresh_tokens.
T11 — Definir MFA factors.
T12 — Definir MFA challenges.
T13 — Definir recovery codes.
T14 — Definir trusted devices.
T15 — Definir security policies.
T16 — Definir audit events.
T17 — Definir security events.
T18 — Definir outbox events.

Constraints

T19 — Definir primary keys.
T20 — Definir foreign keys.
T21 — Definir unique constraints.
T22 — Definir lifecycle constraints.
T23 — Definir nullability.
T24 — Definir referential actions.

Indexes

T25 — Definir user indexes.
T26 — Definir organization indexes.
T27 — Definir membership indexes.
T28 — Definir authorization indexes.
T29 — Definir session indexes.
T30 — Definir refresh token indexes.
T31 — Definir audit indexes.
T32 — Definir outbox indexes.

Transactions

T33 — Definir transaction boundaries.
T34 — Definir concurrency strategy.
T35 — Definir token rotation transaction.
T36 — Definir membership transaction.
T37 — Definir role/permission transactions.
T38 — Definir MFA transactions.
T39 — Integrar Outbox transaccional.

Security

T40 — Definir password hashing persistence.
T41 — Definir token hashing.
T42 — Definir MFA secret protection.
T43 — Definir recovery code protection.
T44 — Definir field classification.
T45 — Definir encryption requirements.

Migrations

T46 — Definir migration strategy.
T47 — Crear initial migration.
T48 — Definir rollback strategy.
T49 — Probar migrations desde database vacía.
T50 — Probar upgrade desde versión anterior.

Performance

T51 — Crear query benchmarks.
T52 — Validar índices.
T53 — Probar connection pool.
T54 — Probar concurrencia.
T55 — Probar Outbox throughput.

Recovery

T56 — Definir backup strategy.
T57 — Definir restore procedure.
T58 — Ejecutar restore test.
T59 — Validar RPO.
T60 — Validar RTO.

Testing

T61 — Persistence unit tests.
T62 — Repository tests.
T63 — Integration tests.
T64 — Constraint tests.
T65 — Transaction tests.
T66 — Concurrency tests.
T67 — Tenant isolation tests.
T68 — Security tests.
T69 — Migration tests.
T70 — Backup/restore tests.

55. Acceptance Criteria
AC-001 — Source of Truth

Given datos críticos de Identity
When se persisten
Then PostgreSQL debe actuar como fuente transaccional de verdad.

AC-002 — Tenant Isolation

Given datos pertenecientes al tenant A
When se consulta utilizando contexto del tenant B
Then la operación debe ser rechazada o devolver únicamente el conjunto autorizado.

AC-003 — Membership Uniqueness

Given un usuario perteneciente a una organización
When se intenta crear otra Membership equivalente
Then la base de datos debe impedir el duplicado.

AC-004 — Role Permission Uniqueness

Given un Role con una Permission
When se intenta asignar nuevamente
Then no debe generarse un registro duplicado.

AC-005 — Token Security

Given un Refresh Token
When se persiste
Then el valor original no debe almacenarse en texto plano.

AC-006 — Transactional Outbox

Given una operación que modifica estado y genera un evento
When la transacción confirma
Then ambos deben quedar persistidos.

AC-007 — Rollback

Given una operación que falla
When la transacción hace rollback
Then no debe quedar un cambio parcial ni Outbox Event huérfano.

AC-008 — Audit Integrity

Given un Audit Event persistido
When se consulta
Then el registro histórico no debe poder modificarse mediante las operaciones normales.

AC-009 — Referential Integrity

Given una entidad dependiente
When se intenta persistir una FK inexistente
Then PostgreSQL debe rechazar la operación.

AC-010 — Concurrency

Given dos operaciones concurrentes sobre el mismo Refresh Token
When ambas intentan rotarlo
Then la persistencia debe permitir que solamente una rotación sea válida.

AC-011 — Migration

Given una base de datos vacía
When se ejecutan las migrations
Then debe crearse el schema completo de Identity correctamente.

AC-012 — Recovery

Given una pérdida de base de datos simulada
When se ejecuta el procedimiento de restore
Then se deben cumplir los objetivos definidos de recuperación.

56. Definition of Done
 Modelo de datos Identity definido.
 Relaciones definidas.
 PKs definidas.
 FKs definidas.
 Unique constraints definidas.
 Índices definidos.
 Lifecycle persistence definido.
 Tenant isolation definido.
 Session persistence definido.
 Refresh token persistence definido.
 MFA persistence definido.
 Security Policy persistence definido.
 Audit persistence definido.
 Security Event persistence definido.
 Outbox persistence definido.
 Transaction boundaries definidos.
 Concurrency strategy definida.
 Migration strategy definida.
 Security classification definida.
 Backup strategy definida.
 Restore probado.
 Persistence tests implementados.
 Migration tests implementados.
 Concurrency tests implementados.
 Security tests implementados.
 Performance tests implementados.
 Traceability completa.
57. Dependencias
Upstream
ESP-0001

ADR-IDENTITY-001
ADR-IDENTITY-003
ADR-IDENTITY-006
ADR-IDENTITY-007
ADR-IDENTITY-008
ADR-IDENTITY-010
ADR-IDENTITY-011

IS-IDENTITY-001
IS-IDENTITY-002
IS-IDENTITY-003
IS-IDENTITY-004
IS-IDENTITY-005
IS-IDENTITY-006
IS-IDENTITY-007
IS-IDENTITY-008
IS-IDENTITY-009
IS-IDENTITY-010
IS-IDENTITY-011
IS-IDENTITY-012
IS-IDENTITY-013
IS-IDENTITY-014
58. Downstream

Esta IS habilita:

IS-IDENTITY-016
       │
       ▼
Repository Implementation
       │
       ▼
ORM / PostgreSQL
       │
       ▼
Application Services
       │
       ▼
API

También habilita posteriormente:

integración real con PostgreSQL;
migrations;
repositories;
integration tests;
deployment.
59. Traceability
Nivel	Referencia
Blueprint	Data Model / Persistence
ESP	ESP-0001
ADR principal	ADR-IDENTITY-006
Sessions	ADR-IDENTITY-003
Events	ADR-IDENTITY-007
Security	ADR-IDENTITY-008
Testing	ADR-IDENTITY-010
Runtime	ADR-IDENTITY-011
IS	IS-IDENTITY-015
Technical Tasks	T01–T70
Source Code	Pendiente
Tests	Pendiente
Deployment	Pendiente

La trazabilidad sigue el ciclo establecido por el Blueprint: ESP → IS → Technical Tasks → Source Code → Testing → Deployment.

60. ADRs pendientes

Esta IS deja identificadas varias decisiones que debemos cerrar antes de generar las migrations definitivas.

ADR-IDENTITY-018 — Identity Database Schema & Naming

Definir:

nombres definitivos;
schemas PostgreSQL;
UUID strategy;
naming conventions.
ADR-IDENTITY-019 — Identity Data Retention & Deletion

Definir:

soft delete;
hard delete;
retention;
legal hold;
anonymization.
ADR-IDENTITY-020 — Identity Database Encryption

Definir:

encryption at rest;
field-level encryption;
key management;
encrypted fields.
ADR-IDENTITY-021 — Identity Database Partitioning

Definir si:

Audit;
Security Events;
Outbox

requieren partitioning.

ADR-IDENTITY-022 — Identity Concurrency Strategy

Definir:

optimistic locking;
pessimistic locking;
PostgreSQL locks;
token rotation concurrency.
ADR-IDENTITY-023 — Identity Migration Strategy

Definir:

migration tooling;
expand/contract;
rollback;
zero-downtime migrations.
61. Punto de control antes de programar

Con esta IS tenemos ahora:

001 Foundation
002 Users
003 Organizations
004 Membership
005 Authentication
006 Authorization
007 Roles & Permissions
008 Sessions
009 Refresh Tokens
010 MFA
011 Security Policy
012 Audit & Security Events
013 Domain Events & Outbox
014 API Contracts
👉 015 Persistence

Esto significa que Identity ya tiene definidos los principales bloques funcionales, de seguridad, eventos, API y persistencia a nivel de Implementation Stories.

El siguiente paso lógico es:

IS-IDENTITY-016 — Identity Testing & Quality

Ahí debemos definir formalmente la estrategia de pruebas de Identity —unitarias, integración, API/contract, seguridad, multi-tenant, concurrencia, MFA, refresh-token reuse, eventos, Outbox, performance y quality gates— antes de pasar a la implementación real.
