ESP-0001 / 11 — Identity Persistence & Data Model

Estado: Draft
Dominio: Identity
Código: ESP-0001
Sección: 11 — Persistence & Data Model
Parent Specification: ESP-0001 — Identity Engineering Specification

1. Propósito

Este documento define el modelo de persistencia y datos del dominio Identity de EVOXA.

Su objetivo es transformar los modelos conceptuales definidos en ESP-0001/01–10 en una estructura persistente capaz de soportar:

usuarios;
organizaciones;
memberships;
roles;
permisos;
sesiones;
tokens;
MFA;
dispositivos confiables;
políticas de seguridad;
auditoría;
aislamiento multi-tenant;
versionamiento;
trazabilidad;
evolución futura.

La arquitectura de EVOXA establece una estrategia de Domain-Driven Design y persistencia poliglota, con PostgreSQL como almacenamiento transaccional principal, además de Event Store, Redis, Object Storage, Vector DB, Knowledge Graph y Data Warehouse según las necesidades del dominio.

2. Principios de persistencia

El modelo Identity debe respetar:

Domain ownership.
Tenant isolation.
Referential integrity.
Least privilege.
Data minimization.
Security by design.
Auditability.
Versioning.
Explicit lifecycle.
Immutable historical records where required.
No sensitive secrets in plaintext.
Transactional consistency.
Evolvability.
3. Estrategia de almacenamiento

Para Identity proponemos inicialmente:

                    Identity
                       │
          ┌────────────┼────────────┐
          ▼            ▼            ▼
      PostgreSQL      Redis      Event Platform
          │            │            │
          │            │            └── Domain/Security Events
          │            │
          │            └── Temporary Security State
          │
          └── Transactional Identity Data
PostgreSQL

Fuente principal para:

User;
Organization;
Membership;
Role;
Permission;
RolePermission;
Session;
RefreshToken;
MFAFactor;
RecoveryCode;
TrustedDevice;
SecurityPolicy.
Redis

Adecuado para información temporal como:

MFA challenges;
rate limiting;
OTP state;
temporal authentication state;
caches de corta duración.
Event Platform

Para:

Identity events;
Security events;
audit integration;
propagación a otros dominios.

La separación exacta entre PostgreSQL, Redis y Event Store deberá cerrarse mediante ADR.

4. Modelo general

La estructura conceptual:

User
 │
 ├───────────────┐
 │               │
 ▼               ▼
Membership     MFAFactor
 │               │
 │               ├── MFAChallenge
 │               └── RecoveryCode
 │
 ▼
Organization
 │
 ▼
Role
 │
 ▼
Permission

Y paralelamente:

User
 │
 ├── Session
 │      └── RefreshToken
 │
 ├── TrustedDevice
 │
 └── SecurityPolicy

Auditoría:

User / Admin / System
          │
          ▼
      AuditEvent
5. User
5.1 Propósito

Representa la identidad principal de una persona dentro de EVOXA.

User
----
id
email
status
created_at
updated_at

El User no debe contener directamente información específica de una organización.

La relación con organizaciones debe producirse mediante Membership.

6. User Identity

Campos conceptuales:

Campo	Tipo conceptual	Requerido
id	UUID	Sí
email	String	Sí
status	Enum	Sí
created_at	Timestamp	Sí
updated_at	Timestamp	Sí
version	Integer	Sí

Estados:

PENDING
ACTIVE
SUSPENDED
DISABLED

Estos estados fueron definidos anteriormente para el lifecycle de User.

7. Email

El email debe tener restricciones de unicidad según la política de identidad.

Posible índice:

UNIQUE(email_normalized)

Se recomienda almacenar una representación normalizada para búsquedas y unicidad.

La estrategia exacta de:

case sensitivity;
aliases;
cambio de email;
verificación;

debe definirse mediante ADR.

8. Organization

Representa un tenant organizacional.

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

La arquitectura de EVOXA utiliza organizaciones/tenants como límite fundamental de aislamiento.

9. Membership

Membership representa la relación entre:

User
  │
  ▼
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
10. Membership Uniqueness

Un usuario no debería tener múltiples memberships activas duplicadas dentro de la misma organización.

Restricción propuesta:

UNIQUE(user_id, organization_id)

Sin embargo, si el modelo histórico necesita conservar memberships removidas y permitir una nueva posteriormente, la estrategia de unicidad deberá definirse cuidadosamente.

11. Membership Roles

Una Membership puede tener uno o más roles.

Modelo recomendado:

Membership
    │
    ├── Role A
    ├── Role B
    └── Role C

Esto es preferible a:

User → Role

porque el mismo usuario puede tener diferentes responsabilidades en diferentes organizaciones.

Ejemplo:

User X
 │
 ├── Organization A
 │      └── Coach
 │
 └── Organization B
        └── Team Manager

La posibilidad de múltiples roles fue identificada anteriormente como una decisión que requiere ADR.

12. Role
Role
----
id
name
description
scope
status
organization_id
created_at
updated_at
version

Scope conceptual:

SYSTEM
ORGANIZATION

Un rol SYSTEM podría existir para capacidades globales.

Un rol ORGANIZATION pertenece al contexto de una organización.

13. Permission
Permission
----------
id
resource
action
description
created_at
updated_at

Ejemplo:

resource = clients
action   = read

Representación lógica:

clients.read

Otros ejemplos:

clients.create
clients.update
clients.delete

training.read
training.create
training.update

nutrition.read
nutrition.publish
14. RolePermission

Tabla intermedia:

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

Restricción:

UNIQUE(role_id, permission_id)
15. Session

Las sesiones representan autenticaciones activas.

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

El modelo se relaciona directamente con ESP-0001/06.

16. Session → User

Una persona puede tener múltiples sesiones.

User
 │
 ├── Session A
 ├── Session B
 └── Session C

Esto permite:

teléfono;
navegador;
tablet;
otro dispositivo.
17. RefreshToken

Los refresh tokens deben persistirse de forma segura.

RefreshToken
------------
id
session_id
token_hash
status
issued_at
expires_at
used_at
revoked_at
replaced_by
created_at

Estados conceptuales:

ACTIVE
USED
REVOKED
EXPIRED

Nunca debe almacenarse el refresh token en texto plano.

Debe almacenarse una representación protegida, por ejemplo un hash apropiado.

18. Refresh Token Family

Para soportar rotación:

RefreshToken A
      │
      ▼
RefreshToken B
      │
      ▼
RefreshToken C

Puede existir una relación:

replaced_by

o un identificador de familia:

token_family_id

Esto permite detectar:

RefreshToken A
       │
       ├── normal use
       │
       └── reuse attempt
              │
              ▼
       Security Incident

La estructura definitiva requiere ADR.

19. MFAFactor
MFAFactor
---------
id
user_id
type
status
name
secret_reference
created_at
verified_at
last_used_at
revoked_at

Tipos:

TOTP
WEBAUTHN
EMAIL_OTP
SMS_OTP

El campo secret_reference es conceptual.

No debe almacenar secretos directamente en una columna normal sin una estrategia de protección definida.

20. MFAChallenge

Dependiendo de la estrategia final, puede vivir en Redis, PostgreSQL o ambos.

Conceptualmente:

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

Estados:

CREATED
PENDING
VERIFIED
FAILED
EXPIRED
CANCELLED

Para challenges de corta duración, Redis puede ser especialmente adecuado.

21. RecoveryCode
RecoveryCode
------------
id
user_id
code_hash
status
created_at
used_at
revoked_at

Restricción:

code_hash

debe ser suficiente para verificar el código sin almacenar el código original.

Un código:

ACTIVE

pasa a:

USED

una vez utilizado.

22. TrustedDevice
TrustedDevice
-------------
id
user_id
device_identifier
status
created_at
last_used_at
expires_at
revoked_at
metadata

Estados:

ACTIVE
EXPIRED
REVOKED

No se deben almacenar identificadores de dispositivo excesivamente invasivos si no son necesarios.

23. SecurityPolicy
SecurityPolicy
--------------
id
name
description
scope
status
priority
version
rules
organization_id
created_at
updated_at

Estados:

DRAFT
ACTIVE
SUSPENDED
RETIRED

La estructura de rules todavía debe resolverse.

Puede ser:

JSON estructurado;
modelo relacional;
policy language;
motor externo.

No debemos decidirlo aquí sin ADR.

24. AuditEvent
AuditEvent
----------
id
event_type
event_version
timestamp
actor_type
actor_id
tenant_id
target_type
target_id
action
result
correlation_id
causation_id
trace_id
metadata

Debe diseñarse como un registro append-oriented.

No debe permitir modificaciones normales después de su creación.

25. Audit Payload

Debe existir una política estricta sobre qué información puede almacenarse.

Permitido conceptualmente:

Actor
Action
Target
Result
Timestamp
Policy
Correlation

No permitido:

Password
Password Hash
Access Token
Refresh Token
TOTP Secret
Private Key
Recovery Code
26. Common Metadata

Las entidades relevantes pueden compartir:

created_at
updated_at
version

Y algunas:

deleted_at

cuando corresponda.

Sin embargo, no recomiendo aplicar deleted_at indiscriminadamente a todas las tablas.

27. Soft Delete

Identity debe diferenciar entre:

Business Lifecycle

y:

Physical Deletion

Por ejemplo:

User
 ↓
DISABLED

no significa necesariamente:

DELETE FROM users

Esto permite conservar:

auditoría;
relaciones;
trazabilidad;
evidencia de seguridad.

La eliminación definitiva debe estar gobernada por políticas de retención.

28. Foreign Keys

Relaciones principales:

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

Session.user_id
        → User.id

Session.organization_id
        → Organization.id

RefreshToken.session_id
        → Session.id

MFAFactor.user_id
        → User.id

MFAChallenge.user_id
        → User.id

MFAChallenge.session_id
        → Session.id

MFAChallenge.factor_id
        → MFAFactor.id

RecoveryCode.user_id
        → User.id

TrustedDevice.user_id
        → User.id
29. Referential Integrity

Las relaciones críticas deben utilizar foreign keys.

Ejemplo:

RolePermission
      │
      ├── role_id → Role
      └── permission_id → Permission

No debe ser posible crear una asignación hacia un role o permission inexistente.

30. Cascade Policies

No debemos utilizar:

ON DELETE CASCADE

indiscriminadamente en Identity.

Por ejemplo:

Organization
    ↓
Membership
    ↓
Audit

Eliminar físicamente la organización no debería destruir automáticamente evidencia histórica.

Las estrategias de cascade deben definirse entidad por entidad.

31. Indexes

Identity tendrá alta frecuencia de consultas sobre:

User
email_normalized
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
Session
user_id
status
expires_at
RefreshToken
session_id
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

Los índices definitivos deben basarse posteriormente en profiling real.

32. Tenant Isolation

Este es uno de los puntos más importantes del modelo.

Para recursos organization-scoped:

organization_id

debe estar disponible directa o indirectamente.

Ejemplo:

Organization A
      │
      ├── Membership
      ├── Roles
      └── Sessions

Nunca debemos confiar únicamente en filtros aplicados por el frontend.

La aplicación debe garantizar aislamiento en backend.

La arquitectura de seguridad exige aislamiento entre tenants y restricciones de acceso cross-tenant.

33. Tenant-Scoped Queries

Una consulta conceptual:

SELECT *
FROM memberships
WHERE organization_id = :currentTenant

debe derivarse del contexto autenticado.

No:

organization_id

tomado ciegamente desde el request.

34. Unique Constraints

Restricciones iniciales:

User.email_normalized

Membership(user_id, organization_id)

RolePermission(role_id, permission_id)

Permission(resource, action)

Posiblemente:

Organization.slug

si posteriormente se incorpora slug.

35. Optimistic Concurrency

Entidades administrativas importantes pueden utilizar:

version

Ejemplo:

Role version = 4

Un update puede requerir:

WHERE id = ?
AND version = 4

y producir:

version = 5

Esto evita que dos administradores sobrescriban cambios simultáneamente.

36. Transactions

Operaciones que modifican múltiples entidades relacionadas deben utilizar transacciones.

Ejemplo:

Assign Role
    │
    ├── Membership validation
    ├── Role validation
    ├── Assignment
    └── Audit/Event

Debe existir una estrategia clara sobre qué parte ocurre dentro de la transacción y qué parte mediante publicación de eventos.

37. Transactional Outbox

Para garantizar consistencia entre:

Database
+
Event Platform

se recomienda evaluar Transactional Outbox.

Conceptualmente:

Transaction
    │
    ├── Business Data
    │
    └── Outbox Event
             │
             ▼
       Event Publisher
             │
             ▼
       Event Platform

Esto es una propuesta arquitectónica derivada, no un requisito explícito del Blueprint.

Debe resolverse mediante ADR.

38. Event Store

No todos los datos de Identity necesitan event sourcing.

El Blueprint indica que determinados aggregates pueden soportar event sourcing, incluyendo algunos agregados como Digital Twin, Assessments y Programs.

Para Identity:

No recomiendo asumir Event Sourcing completo para todas las entidades.

El modelo inicial puede ser:

PostgreSQL
+
Domain Events
+
Audit

y utilizar event sourcing solamente donde una decisión posterior lo justifique.

39. Data Versioning

Las entidades críticas pueden incorporar:

version

Mientras que los eventos tienen:

event_version

Son conceptos diferentes:

Entity Version

representa el estado de una entidad.

Event Version

representa la versión del contrato del evento.

40. Data Encryption

Datos sensibles deben estar protegidos.

La arquitectura establece:

AES-256

para protección de datos en reposo y:

TLS 1.3

para datos en tránsito.

Los secretos especialmente sensibles pueden requerir una estrategia adicional:

KMS
Secrets Manager
Envelope Encryption

La solución concreta requiere ADR.

41. Password Storage

La contraseña no debe almacenarse.

Solo:

password_hash

utilizando el algoritmo aprobado para EVOXA.

La arquitectura de seguridad establece:

Argon2id

como algoritmo para passwords.

42. MFA Secret Storage

Los secretos TOTP requieren protección adicional.

Modelo conceptual:

Application
     │
     ▼
Secrets Protection Layer
     │
     ▼
Encrypted Secret

No recomiendo colocar un secreto MFA directamente en:

mfa_factors.secret

en texto plano.

La solución exacta debe decidirse mediante ADR.

43. Sensitive Data Classification

Podemos clasificar los datos:

Public
Organization Name
Internal
User ID
Role ID
Sensitive
Email
IP
Device Information
Highly Sensitive
Password Hash
TOTP Secret
Recovery Code
Authentication Tokens
Private Keys

Los niveles exactos deben ser definidos por la política global de clasificación de datos de EVOXA.

44. Data Retention

Cada tipo de información debería tener una política de retención.

Ejemplo conceptual:

User
 → Business retention

Session
 → Short retention

RefreshToken
 → Security lifecycle

MFAChallenge
 → Very short retention

AuditEvent
 → Long-term retention

La arquitectura establece una referencia de retención de auditoría de siete años, pero la aplicación concreta debe considerar las obligaciones legales y el despliegue.

45. Backup & Recovery

Identity es un dominio crítico.

Debe incluirse en:

Backup
Restore
Disaster Recovery
Replication
Monitoring

Los objetivos generales de EVOXA incluyen:

Availability ≥ 99.9%
RTO < 1 hour
RPO < 15 minutes

46. Data Migration

Los cambios del esquema deben utilizar migraciones versionadas.

No debe dependerse únicamente de:

sequelize.sync()

o mecanismos equivalentes automáticos en producción.

La evolución de la base de datos debe ser explícita:

Migration 001
Migration 002
Migration 003
...

Esto será especialmente importante cuando comencemos la implementación real de EVOXA.

47. API ↔ Persistence

La API no debe acceder directamente a las tablas.

Arquitectura:

API
 │
 ▼
Application
 │
 ▼
Domain
 │
 ▼
Repository
 │
 ▼
Persistence

Por ejemplo:

UserController
      ↓
CreateUserUseCase
      ↓
UserAggregate
      ↓
UserRepository
      ↓
PostgreSQL

Esto mantiene las capas definidas por la arquitectura de EVOXA.

48. Repository Boundaries

Identity puede definir repositorios como:

UserRepository
OrganizationRepository
MembershipRepository
RoleRepository
PermissionRepository
SessionRepository
RefreshTokenRepository
MFAFactorRepository
MFAChallengeRepository
RecoveryCodeRepository
TrustedDeviceRepository
SecurityPolicyRepository
AuditRepository

El dominio no debería conocer directamente SQL, ORM o PostgreSQL.

49. Data Ownership

Identity será propietario de sus datos:

Identity
 ├── Users
 ├── Organizations
 ├── Memberships
 ├── Roles
 ├── Permissions
 ├── Sessions
 ├── Tokens
 ├── MFA
 ├── Security Policies
 └── Identity Audit

Otros dominios deben acceder mediante:

API
+
Events

y no directamente mediante las tablas de Identity.

Esto sigue el principio de que cada dominio posee su lógica y datos.

50. Modelo relacional conceptual
┌─────────────────┐
│      users      │
├─────────────────┤
│ id PK           │
│ email           │
│ status          │
│ version         │
│ created_at      │
│ updated_at      │
└────────┬────────┘
         │
         ├──────────────────────┐
         │                      │
         ▼                      ▼
┌─────────────────┐     ┌─────────────────┐
│  memberships    │     │   mfa_factors   │
├─────────────────┤     ├─────────────────┤
│ id PK           │     │ id PK           │
│ user_id FK      │     │ user_id FK      │
│ org_id FK       │     │ type            │
│ status          │     │ status          │
└────────┬────────┘     └─────────────────┘
         │
         ▼
┌─────────────────┐
│ organizations   │
├─────────────────┤
│ id PK           │
│ name            │
│ status          │
└─────────────────┘

Roles:

memberships
     │
     ▼
membership_roles
     │
     ▼
roles
     │
     ▼
role_permissions
     │
     ▼
permissions
51. Modelo completo conceptual
                           ┌───────────────┐
                           │     User      │
                           └───────┬───────┘
                                   │
          ┌────────────────────────┼────────────────────┐
          │                        │                    │
          ▼                        ▼                    ▼
    Membership                 Sessions             MFA Factors
          │                        │                    │
          ▼                        ▼             ┌──────┴──────┐
   Organization               RefreshTokens       │             │
          │                                     Challenges   Recovery
          ▼
        Roles
          │
          ▼
     Permissions

User / Admin / System
          │
          ▼
      Audit Events

Organization / Platform
          │
          ▼
    Security Policies
52. Criterios de aceptación

ESP-0001/11 estará listo para implementación cuando:

 User model definido.
 Organization model definido.
 Membership model definido.
 Role model definido.
 Permission model definido.
 RolePermission definido.
 Session definido.
 RefreshToken definido.
 MFAFactor definido.
 MFAChallenge definido.
 RecoveryCode definido.
 TrustedDevice definido.
 SecurityPolicy definido.
 AuditEvent definido.
 Foreign keys definidas.
 Unique constraints definidas.
 Indexes definidos.
 Tenant isolation definida.
 Cascade policies definidas.
 Lifecycle states definidos.
 Versioning definido.
 Encryption strategy definida.
 Retention definida.
 Backup/recovery definido.
 Transaction boundaries definidas.
 Event persistence definida.
 Repository boundaries definidas.
 Migration strategy definida.
 ADRs aprobados.
53. ADRs necesarios

Este capítulo genera varias decisiones importantes.

ADR-IDENTITY-036

Identity Database

Definir PostgreSQL como datastore principal y su configuración.

ADR-IDENTITY-037

Identity Persistence Boundaries

Definir qué vive en:

PostgreSQL
Redis
Event Store
Object Storage
ADR-IDENTITY-038

Membership Role Model

Resolver definitivamente:

User → Role

vs.

Membership → Role

y si existen múltiples roles.

ADR-IDENTITY-039

Refresh Token Persistence

Definir:

hash;
rotation;
family;
reuse detection;
revocation.
ADR-IDENTITY-040

MFA Secret Storage

Definir:

encryption;
KMS;
secret manager;
key rotation.
ADR-IDENTITY-041

Audit Persistence

Definir:

append-only;
immutable storage;
PostgreSQL;
Event Store;
archival.
ADR-IDENTITY-042

Identity Soft Delete & Retention

Definir lifecycle vs eliminación física.

ADR-IDENTITY-043

Transactional Outbox

Determinar si Identity utilizará Outbox para garantizar publicación confiable de eventos.

ADR-IDENTITY-044

Identity Concurrency

Definir optimistic/pessimistic locking y manejo de conflictos.

54. Trazabilidad
BP-0002 Reference Architecture
          │
          ▼
22 — Data Model
          │
          ▼
ESP-0001 Identity
          │
 ┌────────┼─────────────┐
 ▼        ▼             ▼
Domain   Security      APIs
Model    Policies      Contracts
 │        │             │
 └────────┼─────────────┘
          ▼
ESP-0001/11
Persistence & Data Model
          │
          ├── PostgreSQL
          ├── Redis
          ├── Event Platform
          └── Audit Storage
                  │
                  ▼
          Implementation Stories
                  │
                  ▼
             Technical Tasks
                  │
                  ▼
              Migrations
                  │
                  ▼
             Source Code
                  │
                  ▼
                Tests

La arquitectura de datos de EVOXA define explícitamente el enfoque DDD/polyglot persistence y la separación de dominios de datos.

55. Estado de ESP-0001
#	Sección	Estado
01	Identity Domain Model	Draft
02	Identity Architecture	Draft
03	Authentication & Authorization	Draft
04	User & Organization Management	Draft
05	Roles & Permissions	Draft
06	Sessions & Token Management	Draft
07	Multi-Factor Authentication	Draft
08	Security Policies	Draft
09	Audit & Compliance	Draft
10	Identity API Contracts	Draft
11	Persistence & Data Model	Draft
Siguiente sección recomendada

ESP-0001 / 12 — Identity Events & Integration

Ahí conectaremos definitivamente el modelo de Identity con el Event Platform y definiremos los eventos que saldrán de:

User
Organization
Membership
Role
Permission
Authentication
Session
Token
MFA
Security Policy
Audit

incluyendo event types, payloads, versioning, ordering, idempotency, retries, DLQ, correlation/causation IDs y reglas de publicación/consumo. Eso nos dejará Identity mucho más cerca de estar preparado para pasar posteriormente a Implementation Stories y código.
