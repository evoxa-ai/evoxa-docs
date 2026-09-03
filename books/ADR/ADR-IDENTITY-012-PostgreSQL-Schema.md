ADR-IDENTITY-012 — PostgreSQL Schema, Naming & Persistence Strategy

Estado: Proposed
Versión: 1.0
Fecha: 2026-09-02
Dominio: Identity
Prioridad: Critical
Tipo: Architecture Decision Record

1. Contexto

El dominio Identity constituye la base de confianza de EVOXA y es consumido por los demás bounded contexts. El Blueprint establece que Identity debe mantenerse como un dominio independiente y ser responsable de usuarios, organizaciones, tenants, roles, permisos, sesiones y políticas de autenticación/autorización.

Para soportar estas capacidades se requiere una persistencia transaccional consistente para:

Users
Organizations
Memberships
Roles
Permissions
Role Permissions
Membership Roles
Sessions
Refresh Tokens
MFA Factors
MFA Challenges
Recovery Codes
Trusted Devices
Security Policies
Audit Events
Security Events
Outbox Events

La persistencia debe además soportar:

integridad referencial;
aislamiento por tenant;
concurrencia;
auditoría;
evolución mediante migraciones;
publicación segura de eventos;
recuperación ante fallos;
escalabilidad futura.

El Blueprint establece PostgreSQL como referencia para persistencia transaccional y utiliza Redis como mecanismo auxiliar de caché/estado temporal, no como fuente de verdad.

2. Problema

Sin una convención arquitectónica explícita para PostgreSQL podrían aparecer inconsistencias entre los distintos componentes de Identity:

users
Users
user
identity_users
tbl_users

o diferencias como:

createdAt
created_at
creation_date

También existiría riesgo de:

duplicación de información;
relaciones incorrectas entre entidades;
ausencia de constraints;
inconsistencias de tenant;
eliminación accidental de información crítica;
dificultad para realizar migraciones;
dificultad para auditar cambios;
acoplamiento entre Domain Model y Database Model.

Por lo tanto, necesitamos definir una estrategia común de persistencia.

3. Decisión
3.1 PostgreSQL como fuente de verdad

PostgreSQL será la fuente de verdad transaccional de Identity.

La arquitectura será:

                    IDENTITY DOMAIN
                         │
                         ▼
                 Application Layer
                         │
                         ▼
                   Repository Port
                         │
                         ▼
              PostgreSQL Infrastructure
                         │
                         ▼
                     PostgreSQL

El dominio no deberá depender directamente de:

SQL;
ORM;
PostgreSQL;
drivers;
connection pools.

La infraestructura implementará los contratos definidos por el dominio/aplicación.

4. Modelo lógico

El modelo inicial de Identity estará compuesto conceptualmente por:

users
organizations
memberships

roles
permissions
role_permissions
membership_roles

sessions
refresh_tokens

mfa_factors
mfa_challenges
recovery_codes
trusted_devices

security_policies

audit_events
security_events

outbox_events

Relación principal:

                         ┌───────────────┐
                         │     User      │
                         └───────┬───────┘
                                 │
                                 │
                         ┌───────▼────────┐
                         │   Membership   │
                         └───────┬────────┘
                                 │
                   ┌─────────────┴─────────────┐
                   │                           │
             Organization                  Role
                   │                           │
                   │                     RolePermission
                   │                           │
                   │                      Permission
                   │
                   ▼
                Tenant

Y la sesión:

User
 │
 └── Session
       │
       ├── Refresh Tokens
       │
       └── Authentication Context
5. Convención de nombres

Se adopta:

Tablas

snake_case, plural:

users
organizations
memberships
roles
permissions
role_permissions
membership_roles
sessions
refresh_tokens
mfa_factors
mfa_challenges
recovery_codes
trusted_devices
security_policies
audit_events
security_events
outbox_events
Columnas

También:

snake_case

Ejemplo:

created_at
updated_at
user_id
organization_id
membership_id
role_id
permission_id
session_id
Claves primarias

Convención:

id

Ejemplo:

users.id
organizations.id
sessions.id
Foreign Keys

Formato:

<entity>_id

Ejemplos:

user_id
organization_id
membership_id
role_id
permission_id
session_id
6. Identificadores

Las entidades de Identity utilizarán identificadores técnicamente únicos.

Conceptualmente:

UserId
OrganizationId
MembershipId
RoleId
PermissionId
SessionId
RefreshTokenId
MFAFactorId
MFAChallengeId

El tipo concreto de UUID y su representación física quedan sujetos a la especificación de persistencia y a la implementación tecnológica final.

Este ADR no fija todavía un formato binario concreto ni una versión específica de UUID.

7. Timestamps

Las entidades persistentes que requieran temporalidad utilizarán:

created_at
updated_at

cuando corresponda.

Para entidades con lifecycle temporal también podrán existir:

activated_at
suspended_at
revoked_at
expired_at
deleted_at

pero no todas las tablas deberán tener automáticamente todas estas columnas.

Cada campo deberá responder a una necesidad funcional o de auditoría.

8. Lifecycle sobre eliminación física

Identity utilizará estados explícitos para representar lifecycle.

Ejemplo:

User

PENDING
   ↓
ACTIVE
   ↓
SUSPENDED
   ↓
DISABLED

En lugar de interpretar:

row deleted = user disabled

la aplicación utilizará estados explícitos.

Esto es especialmente importante para:

users;
organizations;
memberships;
sessions;
roles;
permissions;
security policies.

La eliminación física, cuando sea necesaria, será tratada posteriormente mediante ADR-IDENTITY-013 — Data Retention, Deletion & Lifecycle.

9. Integridad referencial

Las relaciones entre entidades deberán estar protegidas mediante constraints de base de datos.

Ejemplo:

memberships.user_id
        ↓
users.id
memberships.organization_id
        ↓
organizations.id
role_permissions.role_id
        ↓
roles.id
role_permissions.permission_id
        ↓
permissions.id

La aplicación no será la única responsable de garantizar integridad.

10. Unicidad

Se definirán constraints de unicidad para relaciones que conceptualmente no pueden duplicarse.

Ejemplo:

memberships
UNIQUE (
    user_id,
    organization_id
)

Esto garantiza que:

User A
   +
Organization B

no tenga accidentalmente dos memberships equivalentes.

De igual forma:

role_permissions
UNIQUE (
    role_id,
    permission_id
)

y:

membership_roles
UNIQUE (
    membership_id,
    role_id
)

La definición definitiva de cada constraint será formalizada en IS-IDENTITY-015 — Identity Persistence.

11. Tenant Isolation

La organización constituye el límite de tenant.

Conceptualmente:

Organization
      │
      ├── Memberships
      ├── Organization Roles
      ├── Sessions
      ├── Policies
      └── Security/Audit Context

Una consulta de aplicación nunca deberá asumir que:

resource_id

es suficiente para autorizar acceso.

Debe existir un contexto de tenant apropiado.

Ejemplo:

Request
  ↓
TenantContext
  ↓
Authorization
  ↓
Repository Query
  ↓
PostgreSQL

La base de datos deberá colaborar con la aplicación mediante:

foreign keys;
índices;
constraints;
consultas tenant-aware.

La estrategia avanzada de aislamiento físico/lógico queda fuera de este ADR y deberá definirse posteriormente si fuera necesaria.

12. Users

La tabla conceptual:

users

representará la identidad global del usuario.

Contendrá información de identidad y lifecycle.

Conceptualmente:

users
├── id
├── email
├── status
├── created_at
├── updated_at
└── lifecycle_metadata
Regla

El usuario no tendrá necesariamente organization_id.

La relación será:

User
  │
  └── Membership
          │
          └── Organization

Esto permite que un usuario pertenezca a múltiples organizaciones.

13. Organizations
organizations

representará el tenant.

Conceptualmente:

organizations
├── id
├── name
├── status
├── created_at
└── updated_at

El estado de la organización participa en la evaluación de acceso.

Una organización suspendida no deberá proporcionar acceso efectivo aunque:

User = ACTIVE
Membership = ACTIVE
Role = valid
Permission = valid
14. Memberships
memberships

representará:

User ↔ Organization

No será simplemente una tabla auxiliar.

Representará una relación con lifecycle propio:

INVITED
   ↓
ACTIVE
   ↓
SUSPENDED
   ↓
REMOVED

La pertenencia a una organización será parte del contexto de autorización.

15. Roles y Permissions

La estructura será:

Membership
     │
     ▼
MembershipRole
     │
     ▼
Role
     │
     ▼
RolePermission
     │
     ▼
Permission

Permission utilizará conceptualmente:

<resource>.<action>

Ejemplos:

users.read
users.update
users.delete

workouts.read
workouts.create

El modelo detallado pertenece a IS-IDENTITY-007 e IS-IDENTITY-015.

16. Sessions
sessions

representará sesiones autenticadas persistentes.

La sesión estará relacionada con:

user
organization
membership

cuando corresponda.

La sesión será independiente del JWT.

Session
   │
   ├── Access Token
   │
   └── Refresh Token Family

El JWT podrá ser stateless mientras que la sesión seguirá siendo stateful.

Esto mantiene la decisión establecida en los ADRs anteriores.

17. Refresh Tokens
refresh_tokens

almacenará información necesaria para:

rotation;
token families;
lineage;
expiration;
revocation;
reuse detection.

Nunca se almacenará el refresh token utilizable como secreto en texto plano.

El diseño criptográfico detallado corresponde a:

ADR-IDENTITY-025 — Refresh Token Cryptography & Concurrency.

18. MFA

La persistencia contempla:

mfa_factors
mfa_challenges
recovery_codes
trusted_devices

La separación permite distinguir:

Factor
   ≠
Challenge
   ≠
Recovery Code
   ≠
Trusted Device

Los secretos MFA deberán tener protección criptográfica apropiada.

No deberán aparecer en:

logs
audit_events
outbox_events
application errors
19. Security Policies
security_policies

almacenará las políticas necesarias para el Security Policy Engine.

Conceptualmente:

id
name
scope
status
priority
version
rules
effective_from
effective_until
created_at
updated_at

La estructura definitiva de rules no queda definida por este ADR.

Será definida en:

ADR-IDENTITY-011 — Security Policy Engine

y posteriormente en la especificación de persistencia.

20. Audit Events
audit_events

será una estructura especializada para auditoría.

Características:

append-only;
inmutable;
tenant-aware;
searchable;
trazable;
sin secretos.

La arquitectura del Blueprint establece que los eventos de auditoría son inmutables.

No se utilizará audit_events como sustituto de logs técnicos.

21. Security Events
security_events

representará hechos relevantes para seguridad.

Ejemplos:

AuthenticationFailed
AuthorizationDenied
TenantAccessDenied
RefreshTokenReuseDetected
PrivilegeEscalationDetected
PolicyViolationDetected

Podrán alimentar:

Security Monitoring
SIEM
Alerting
Incident Response
Analytics
22. Outbox Events
outbox_events

será parte fundamental de la arquitectura transaccional.

Regla:

Domain State Change
       +
Outbox Event
       │
       ▼
   SAME TRANSACTION

De esta manera se evita:

Database committed
        +
Event lost

La arquitectura de eventos del Blueprint exige publicación segura, at-least-once delivery, retry, DLQ, replay e idempotencia.

23. Índices

Los índices se diseñarán según patrones reales de acceso.

Como mínimo se evaluarán:

users
    email

memberships
    user_id
    organization_id
    user_id + organization_id

sessions
    user_id
    organization_id
    status
    expires_at

refresh_tokens
    session_id
    family_id
    status

audit_events
    tenant_id
    timestamp
    actor_id
    event_type
    correlation_id
    trace_id

security_events
    tenant_id
    timestamp
    severity
    event_type

outbox_events
    status
    next_attempt_at
    occurred_at

Importante: esta lista es arquitectónica. Los índices definitivos deberán validarse mediante patrones de consulta y pruebas de rendimiento.

24. Transacciones

Las operaciones que modifiquen múltiples recursos relacionados deberán utilizar transacciones cuando la consistencia lo requiera.

Ejemplo:

Create Membership
       │
       ├── Membership
       ├── Role Assignment
       ├── Audit Event
       └── Outbox Event

Las operaciones que deban ser atómicas no podrán quedar parcialmente persistidas.

25. Redis

Redis no será la fuente de verdad de Identity.

Su utilización podrá incluir:

Redis
├── Cache
├── Rate Limiting
├── Temporary MFA State
├── High-frequency state
└── Authorization cache

Pero:

PostgreSQL = Source of Truth
Redis      = Auxiliary

Una caída de Redis no deberá provocar un bypass de seguridad.

La aplicación deberá poder degradarse de forma segura.

26. ORM

El ORM, si se utiliza, pertenecerá exclusivamente a Infrastructure.

La arquitectura seguirá:

Domain
  ↓
Repository Interface
  ↓
Infrastructure
  ↓
ORM
  ↓
PostgreSQL

No:

Domain
  ↓
ORM Model

Esto protege al dominio frente a decisiones específicas de persistencia.

27. Migraciones

La base de datos será evolucionada mediante migraciones versionadas.

No se permitirá en producción:

startup()
   ↓
auto-create tables

ni depender de mecanismos equivalentes a:

sync schema automatically

La evolución seguirá:

Migration
    ↓
Validation
    ↓
Deployment
    ↓
Verification

La estrategia completa será definida en:

ADR-IDENTITY-017 — Database Migration Strategy.

El Blueprint establece explícitamente migraciones versionadas y prácticas controladas de evolución del esquema.

28. Seguridad de la base de datos

El acceso a PostgreSQL deberá cumplir:

mínimo privilegio;
credenciales externas;
conexiones seguras;
rotación de secretos;
separación de ambientes;
no credenciales en código;
no credenciales en logs;
auditoría de accesos administrativos.

La estrategia criptográfica específica se desarrolla en:

ADR-IDENTITY-014 — Encryption & Secret Protection.

29. Failure Model

El diseño deberá contemplar:

PostgreSQL unavailable
API
 ↓
DB Failure
 ↓
Fail safely

No deberá:

crear identidades parcialmente;
aceptar autorizaciones sin validación;
utilizar datos inconsistentes.
Redis unavailable
Redis DOWN
   ↓
Fallback / Degraded Mode
   ↓
Security preserved
Event Platform unavailable
DB Transaction
      ↓
Outbox
      ↓
Event Platform DOWN
      ↓
Event remains pending

Esto es precisamente el objetivo del patrón Outbox.

30. Alternativas consideradas
A. PostgreSQL + Redis como fuentes equivalentes

Descartada.

Generaría dos fuentes de verdad.

B. MongoDB como persistencia primaria

No adoptada.

No está respaldada por la arquitectura actual de Identity.

C. Base de datos independiente por entidad

No adoptada inicialmente.

Aumentaría la complejidad transaccional y operativa sin una necesidad establecida.

D. ORM como modelo de dominio

Descartada.

Generaría acoplamiento entre Domain e Infrastructure.

E. Auto-sync del esquema

Descartada para producción.

Las modificaciones de esquema deberán estar controladas mediante migraciones.

31. Consecuencias
Positivas
Fuente de verdad única.
Integridad referencial.
Tenant isolation.
Migraciones controladas.
Mejor auditabilidad.
Soporte para concurrencia.
Compatibilidad con Outbox.
Independencia del dominio respecto del ORM.
Evolución controlada.
Negativas
Mayor complejidad inicial.
Requiere disciplina en migrations.
Requiere diseño cuidadoso de índices.
Requiere estrategia de backup/restore.
Algunas decisiones quedan pendientes de ADRs especializados.
32. Reglas no negociables

Quedan establecidas como reglas arquitectónicas:

1. PostgreSQL = Source of Truth.

2. Redis no reemplaza PostgreSQL.

3. Domain no depende directamente del ORM.

4. No secrets en la base de datos en plaintext.

5. No secrets en logs/events/audit.

6. Tenant isolation debe verificarse server-side.

7. Relaciones críticas utilizan constraints.

8. Operaciones críticas utilizan transacciones.

9. Domain State + Outbox deben ser atómicos.

10. Producción utiliza migrations versionadas.

11. Audit Events son append-only.

12. Security Events no se utilizan como logs técnicos.

13. No auto-sync de schema en producción.

14. Los índices deben responder a patrones de acceso reales.

15. Las decisiones pendientes no deben inventarse en implementación.
33. Trazabilidad
Blueprint
   │
   ├── Identity Domain
   │
   ├── Security Architecture
   │
   ├── Data Model
   │
   └── Event Architecture
          │
          ▼
       ESP-0001
          │
          ▼
 ADR-IDENTITY-001
 ADR-IDENTITY-003
 ADR-IDENTITY-005
 ADR-IDENTITY-006
 ADR-IDENTITY-007
 ADR-IDENTITY-008
 ADR-IDENTITY-009
 ADR-IDENTITY-012
          │
          ▼
 IS-IDENTITY-015
 Identity Persistence

La arquitectura de eventos del Blueprint además establece que cada evento debe disponer de metadata, tenant, versión, correlación y trazabilidad.

34. Dependencias

Este ADR depende de:

ESP-0001 — Identity Platform
ADR-IDENTITY-003 — Organization & Tenant
ADR-IDENTITY-005 — Authentication
ADR-IDENTITY-006 — Authorization
ADR-IDENTITY-007 — Roles & Permissions
ADR-IDENTITY-008 — Sessions
ADR-IDENTITY-009 — Refresh Tokens
ADR-IDENTITY-011 — Security Policy Engine

Y será utilizado por:

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
IS-IDENTITY-015
IS-IDENTITY-016
IS-IDENTITY-017
35. ADRs relacionados

Este ADR deja explícitamente separados los siguientes temas:

ADR-IDENTITY-013
Retention / Deletion

ADR-IDENTITY-014
Encryption / Secret Protection

ADR-IDENTITY-015
Partitioning / Scalability

ADR-IDENTITY-016
Concurrency / Consistency

ADR-IDENTITY-017
Migration Strategy

ADR-IDENTITY-025
Refresh Token Cryptography
36. Decisiones pendientes

Este ADR no decide todavía:

período exacto de retención;
política de eliminación/anonymization;
particionamiento definitivo;
estrategia exacta de cifrado por campo;
proveedor KMS;
formato físico definitivo de UUID;
ORM específico;
pool sizing;
aislamiento físico de tenants;
estrategia de archivado;
política exacta de backup;
PostgreSQL topology.

Esas decisiones deberán quedar en sus respectivos ADRs.

37. Estado

ADR-IDENTITY-012 — Proposed

Criterio para pasar a Accepted

Debe existir consenso sobre:

modelo lógico;
naming conventions;
tenant model;
integrity constraints;
transaction boundaries;
relación PostgreSQL/Redis;
separación Domain/Infrastructure;
migration strategy.

Una vez aceptado, este ADR será la referencia arquitectónica para IS-IDENTITY-015 — Identity Persistence.

Resultado

Con ADR-IDENTITY-012 ya dejamos establecida la columna vertebral de persistencia de Identity, pero sin adelantarnos a decisiones que pertenecen a otros ADRs.

El siguiente en orden lógico es:

ADR-IDENTITY-013 — Data Retention, Deletion & Lifecycle

porque antes de definir completamente las tablas de Audit Events, Security Events, Sessions, Refresh Tokens y Outbox Events, necesitamos saber qué información se conserva, durante cuánto tiempo, qué puede anonimizarse y qué nunca debe eliminarse.
