TASK-001-19 — Initial Migration Framework

Implementation Story: IS-001 — Identity Domain Foundation
ESP: ESP-0001/14 — Identity Implementation Requirements
Related ESP: ESP-0001/01, ESP-0001/02, ESP-0001/06, ESP-0001/09, ESP-0001/10, ESP-0001/11, ESP-0001/12, ESP-0001/13, ESP-0001/15
Related ADRs: ADR-IDENTITY-006, 012, 013, 015, 016, 017, 029, 030, 055, 065
Priority: 🔴 Critical
Status: 🟢 Ready for Implementation
Task Type: Persistence / Database Governance

1. Objetivo

Establecer el framework oficial de migraciones de base de datos para EVOXA Identity.

A partir de esta tarea, la estructura PostgreSQL de EVOXA deberá evolucionar mediante:

Migration 001
      ↓
Migration 002
      ↓
Migration 003
      ↓
...

y no mediante modificaciones automáticas del ORM.

La regla fundamental será:

El código define el modelo; las migraciones controlan la evolución del esquema persistente.

2. Decisión arquitectónica

PostgreSQL es la fuente de verdad transaccional de Identity.

Por lo tanto:

Application
     │
     ▼
Domain / Repository Contracts
     │
     ▼
Infrastructure
     │
     ▼
PostgreSQL

La evolución de PostgreSQL será:

Source Code Change
       │
       ▼
Migration
       │
       ▼
Migration Runner
       │
       ▼
PostgreSQL Schema

No:

Application Startup
       │
       ▼
create_all()
       │
       ▼
automatic schema mutation
3. Regla crítica: no create_all

Queda establecido que EVOXA no utilizará:

Base.metadata.create_all(...)

como mecanismo de evolución de producción.

Tampoco:

ORM auto-sync
sequelize.sync()

ni mecanismos equivalentes.

Esto es especialmente importante porque el proyecto deberá evolucionar durante años y necesitamos:

historial;
reproducibilidad;
rollback controlado;
auditoría;
revisión;
CI/CD;
recuperación ante desastre;
compatibilidad entre versiones.
4. Tecnología inicial

La tecnología base del Blueprint contempla:

Python
FastAPI
PostgreSQL
Redis
Docker
Kubernetes
GitHub Actions

Para la implementación inicial del migration framework se propone:

Alembic
+
SQLAlchemy
+
PostgreSQL

Esta es una decisión de implementación, no una modificación del Blueprint.

La arquitectura sigue siendo independiente del ORM/migration tool.

5. Responsabilidades del Migration Framework

Debe permitir:

crear migraciones;
versionarlas;
ejecutarlas;
verificar estado;
detectar divergencias;
ejecutar migraciones en CI/CD;
mantener historial;
soportar migraciones transaccionales cuando PostgreSQL lo permita;
controlar cambios destructivos;
validar compatibilidad;
facilitar recuperación.
6. Estructura propuesta
apps/api/
│
├── alembic.ini
│
├── migrations/
│   ├── env.py
│   ├── script.py.mako
│   │
│   └── versions/
│       ├── 0001_initial_identity_foundation.py
│       ├── ...
│       └── ...
│
└── app/
    ├── core/
    │   └── infrastructure/
    │       └── database/
    │           ├── engine.py
    │           ├── session.py
    │           └── health.py
    │
    └── domains/
        └── identity/

La ubicación exacta podrá ajustarse al repositorio definitivo, pero debe existir un único mecanismo oficial de migración.

7. Migration Metadata

Alembic mantendrá una tabla de control conceptual:

alembic_version

que permite conocer qué revisión está aplicada.

Conceptualmente:

Database
   │
   └── alembic_version
             │
             └── current_revision

Esto no sustituye el historial Git.

Ambos deben mantenerse:

Git
 +
Migration History
 +
Database Revision
8. Migration ID

Cada migración tendrá un identificador único.

Ejemplo:

0001
0002
0003

y un nombre descriptivo:

0001_initial_identity_foundation
0002_add_membership
0003_add_role_permission

El ID debe ser inmutable.

Una migración aplicada no debe cambiarse silenciosamente.

9. Migration Inmutability

Una vez que una migración haya sido aplicada en un entorno compartido:

DEV
QA
STAGING
PRODUCTION

no se debe modificar su contenido.

Incorrecto:

Migration 0005
   ↓
aplicada
   ↓
editar 0005

Correcto:

Migration 0005
   ↓
aplicada
   ↓
Migration 0006

Esto mantiene reproducibilidad.

10. Schema Versioning

El esquema debe evolucionar mediante una cadena:

V1
 ↓
V2
 ↓
V3
 ↓
V4

Nunca debemos asumir que:

current code
=
current database

El deployment debe validar explícitamente la compatibilidad.

11. Initial Migration

TASK-001-19 no debe crear todavía todas las tablas de Identity.

En este momento estamos creando el framework.

La primera migración funcional de tablas llegará cuando las respectivas Implementation Stories/Tasks definan las entidades.

Por ejemplo, posteriormente:

User
Organization
Membership
Role
Permission
RolePermission
Session
RefreshToken
MFAFactor
SecurityPolicy
AuditEvent
SecurityEvent
OutboxEvent

se incorporarán mediante sus migraciones correspondientes.

12. No adelantar el Data Model

No debemos aprovechar TASK-001-19 para inventar:

user table
role table
permission table

sin que corresponda a la Task/IS que define esa entidad.

Esto mantiene la trazabilidad:

ESP
 ↓
ADR
 ↓
IS
 ↓
Technical Task
 ↓
Migration
13. Migration Ownership

Cada migración deberá poder rastrearse hasta el cambio que la originó.

Ejemplo:

IS-002
   │
   ▼
TASK-002-03
   │
   ▼
Migration 0002

La migración podrá incluir metadata/documentación:

Story:
IS-002

Task:
TASK-002-03

Reason:
Create User persistence model

Esto facilita auditoría técnica.

14. Naming Convention

Propuesta:

<revision>_<description>.py

Ejemplo:

0001_initial_identity_foundation.py
0002_create_user.py
0003_create_organization.py
0004_create_membership.py

Cuando la evolución sea compleja:

0008_add_refresh_token_family.py
0009_add_security_epoch.py

Los nombres deben describir qué cambia, no quién lo hizo.

15. Forward Migration

Toda migración debe definir claramente:

def upgrade():
    ...

El upgrade representa:

Current Schema
      ↓
New Schema

Debe ser:

determinista;
reproducible;
revisable;
segura.
16. Downgrade

Se podrá definir:

def downgrade():
    ...

pero debemos distinguir:

reversible

de:

safe rollback

No toda operación destructiva puede revertirse sin pérdida de datos.

Por ejemplo:

DROP COLUMN

puede provocar pérdida permanente.

Por eso:

La existencia de downgrade() no significa que una migración sea operacionalmente reversible.

17. Expand / Contract

Para cambios de esquema importantes utilizaremos preferentemente:

EXPAND
   ↓
MIGRATE
   ↓
CONTRACT

Ejemplo:

Expand

Agregar nueva columna nullable:

old_column
new_column
Migrate

Transferir datos:

old_column → new_column
Contract

Eliminar la columna antigua únicamente cuando ya no sea necesaria.

Esto evita despliegues donde código viejo y nuevo sean incompatibles.

18. Zero-Downtime Consideration

Las migraciones deben diseñarse considerando:

Old Application
       +
New Application
       +
Current Database

durante un deployment progresivo.

No debemos asumir:

migration
↓
all instances instantly updated

porque EVOXA utilizará despliegues rolling/blue-green/canary.

19. Dangerous Migrations

Se consideran potencialmente peligrosas:

DROP TABLE
DROP COLUMN
ALTER TYPE
large UPDATE
large DELETE
NOT NULL sobre tabla con datos
unique constraint sobre datos existentes

Estas migraciones requieren revisión adicional.

20. Destructive Migration Policy

Una migración destructiva deberá:

identificarse;
documentarse;
validar impacto;
tener estrategia de backup;
tener estrategia de recuperación;
pasar tests;
recibir aprobación correspondiente.

No se debe ejecutar automáticamente un DROP simplemente porque el modelo ORM eliminó un atributo.

21. Data Migration

Debemos distinguir:

Schema Migration

de:

Data Migration

Ejemplo:

Schema:
add new column

vs.

Data:
populate new column from existing data

Una migración de datos debe considerar:

volumen;
duración;
locking;
rollback;
idempotencia;
impacto en producción.
22. Large Data Migrations

No debemos ejecutar indiscriminadamente:

UPDATE millions_of_rows;

dentro de una única operación si puede generar:

locks largos;
saturación;
downtime;
problemas de WAL;
impacto en replicas.

Para grandes volúmenes podrá utilizarse posteriormente un proceso controlado de backfill.

23. Transactional Migrations

Cuando PostgreSQL permita ejecutar la migración de forma segura dentro de una transacción:

BEGIN
   migration
COMMIT

se preferirá ese comportamiento.

Si una migración requiere operaciones que no pueden ejecutarse transaccionalmente, debe quedar explícitamente documentado.

24. Migration Locking

Debe evitarse que múltiples instancias intenten ejecutar simultáneamente:

Migration Runner A
Migration Runner B
Migration Runner C

La ejecución deberá estar controlada.

Conceptualmente:

              Migration Runner
                     │
                     ▼
              Migration Lock
                     │
             ┌───────┴───────┐
             ▼               ▼
           APPLY            WAIT

La implementación exacta de locking queda dentro del framework seleccionado.

25. Migration Runner

El framework deberá soportar comandos equivalentes a:

alembic upgrade head

y:

alembic current

además de:

alembic history

y generación de nuevas migraciones cuando corresponda.

26. Autogenerate

Se puede utilizar:

alembic revision --autogenerate

como herramienta auxiliar.

Pero:

Nunca se debe confiar ciegamente en una migración autogenerada.

Debe revisarse manualmente.

Especialmente para:

índices;
constraints;
rename vs drop/add;
datos;
tipos;
seguridad;
tenant isolation.
27. Migration Review

Toda migración debe pasar por revisión.

Flujo:

Developer
    │
    ▼
Generate Migration
    │
    ▼
Manual Review
    │
    ▼
Migration Tests
    │
    ▼
Architecture / CI
    │
    ▼
PR Approval
28. Schema Drift Detection

Debemos detectar divergencias entre:

Expected Schema

y:

Actual Database Schema

Ejemplos:

manual column added
manual index deleted
constraint changed
migration missing

El pipeline debe poder detectar drift.

29. Production Rule

No se deben realizar modificaciones manuales permanentes del esquema de producción.

No:

ALTER TABLE ...

directamente en producción como solución permanente.

Debe convertirse en:

SQL change
   ↓
Migration
   ↓
Review
   ↓
CI
   ↓
Deployment

Las intervenciones manuales de emergencia deben quedar como excepciones documentadas y reconciliadas posteriormente mediante migración.

30. Migration Testing

Cada migración debe probarse al menos en:

empty database
current schema
upgrade path
downgrade where supported

Y, cuando corresponda:

existing representative data
31. Migration Test Flow
Clean DB
   │
   ▼
Apply migrations
   │
   ▼
Expected schema
   │
   ▼
Application tests

También:

Previous Version
      │
      ▼
Upgrade
      │
      ▼
Current Version
32. Fresh Database Test

Debe existir una prueba periódica:

DROP test DB
     ↓
create empty DB
     ↓
alembic upgrade head
     ↓
start application
     ↓
run integration tests

Esto comprueba que el historial de migraciones realmente reconstruye la base desde cero.

33. Upgrade Path Test

También necesitamos:

Version N
   ↓
Migration N+1
   ↓
Migration N+2
   ↓
...
   ↓
HEAD

No solo probar:

empty → HEAD

porque los problemas reales aparecen durante actualizaciones.

34. Schema Integrity

Las migraciones deben validar:

PK;
FK;
unique constraints;
indexes;
nullability;
data types;
tenant boundaries;
version columns;
audit/outbox requirements cuando correspondan.
35. Tenant Isolation

Cuando se incorporen tablas tenant-scoped:

organization_id

y sus constraints/indexes deberán formar parte de las migraciones correspondientes.

La migración no puede introducir accidentalmente una tabla tenant-scoped sin mecanismo para identificar el tenant.

36. Security State

Los mecanismos futuros relacionados con:

security_epoch
entity_version
policy_version

deberán evolucionar mediante migraciones versionadas.

No se deben introducir manualmente en producción.

37. Outbox

Cuando se implemente OutboxEvent, su estructura deberá llegar mediante migration.

Conceptualmente:

Business Table
       +
Outbox Table

y las operaciones deberán respetar:

same transaction

según las reglas de Transactional Outbox.

38. Audit Data

Las estructuras de:

AuditEvent
SecurityEvent

también deben tener migrations propias.

Debemos evitar crear tablas de auditoría improvisadas desde el código.

39. Indexes

Los índices deben declararse explícitamente.

Ejemplos futuros conceptuales:

users(email)
memberships(organization_id, user_id)
sessions(user_id, status)
refresh_tokens(family_id)
outbox_events(status, occurred_at)

Pero no se deben crear todavía en TASK-001-19.

Los índices definitivos deben salir de sus respectivas especificaciones de persistencia.

40. PostgreSQL Extensions

Si alguna funcionalidad requiere extensiones PostgreSQL:

extension

su instalación deberá ser:

versionada;
documentada;
validada en todos los entornos.

No se debe depender de que un desarrollador haya instalado manualmente una extensión en su máquina.

41. Environment Consistency

Las migraciones deben funcionar consistentemente en:

Development
QA
Staging
Production

aunque las configuraciones de infraestructura sean diferentes.

No debemos tener:

migration works in DEV
migration fails in PROD

por depender de modificaciones manuales.

42. CI/CD

El pipeline deberá incorporar:

Lint
   ↓
Unit Tests
   ↓
Architecture Tests
   ↓
Migration Validation
   ↓
Integration Tests
   ↓
Security Tests

Las migraciones forman parte del código y deben tratarse como tal.

43. Deployment Strategy

La ejecución en deployment debe seguir aproximadamente:

Build
  ↓
Validate migration
  ↓
Backup / recovery readiness
  ↓
Apply migration
  ↓
Deploy application
  ↓
Health checks
  ↓
Validation

El orden exacto dependerá de si la migración es:

backward compatible

o:

breaking
44. Migration Compatibility

Las migraciones deberán clasificarse:

COMPATIBLE
EXPAND
DATA
CONTRACT
DESTRUCTIVE

Esto ayudará a decidir cómo desplegar.

45. Migration Failure

Si una migración falla:

Migration
   │
   ▼
FAIL

no debemos continuar ciegamente con el deployment.

Debe quedar:

FAILED

y detenerse el pipeline según la política del entorno.

46. Recovery

La recuperación debe considerar:

Database Backup
+
Migration State
+
Application Version

Nunca debemos asumir que simplemente ejecutar:

downgrade

es suficiente para recuperar producción.

Para cambios destructivos, la recuperación puede requerir:

backup / PITR

de acuerdo con ADR-029/030.

47. Migration Observability

Las migraciones deberán producir información operacional:

migration_started
migration_completed
migration_failed
migration_duration
migration_revision

Nunca:

database_password
connection_string
secret
48. Auditability

Los cambios de esquema son importantes desde el punto de vista de gobernanza.

Debemos poder responder:

¿Qué cambió?
¿Cuándo?
¿Por qué?
¿Quién lo aprobó?
¿Qué versión lo introdujo?
¿En qué entorno se aplicó?

Por eso tendremos:

Git history
+
PR
+
Migration file
+
CI evidence
+
Deployment evidence
49. Migration Manifest

A futuro podemos incorporar metadata como:

migration_id: "0004"
story: "IS-002"
task: "TASK-002-03"
type: "EXPAND"
destructive: false
requires_review: false

No es obligatorio implementar todo este manifiesto en TASK-001-19.

Lo importante es dejar la arquitectura preparada.

50. Alembic Environment

El env.py debe utilizar la configuración central de EVOXA.

Conceptualmente:

from app.core.config import get_settings

y obtener:

DATABASE_HOST
DATABASE_PORT
DATABASE_NAME
DATABASE_USER
DATABASE_PASSWORD

desde el mecanismo de configuración/secrets.

No deben quedar credenciales escritas en:

alembic.ini

ni dentro del repositorio.

51. Async PostgreSQL

Como el runtime de EVOXA será async, la aplicación utilizará SQLAlchemy async.

El migration runner puede requerir una configuración específica para Alembic.

La separación debe ser:

Application Runtime
      │
      └── Async DB

Migration Runtime
      │
      └── Migration Engine

No debemos forzar que el migration runner sea idéntico al runtime HTTP.

52. No Migration Logic in Domain

El Domain nunca debe contener:

ALTER TABLE
CREATE TABLE
DROP TABLE

ni importar:

Alembic
SQLAlchemy migration APIs

La migración pertenece a Infrastructure/Deployment.

53. No Business Logic in Migration

Las migrations tampoco deben convertirse en una segunda implementación del Domain.

Una migration debe transformar:

Schema
+
Data

de forma controlada.

No debe implementar:

authorization
authentication
MFA
business workflows
54. Migration Boundary

La frontera será:

                  Application
                       │
                       ▼
                  Repositories
                       │
                       ▼
                 PostgreSQL
                       ▲
                       │
               Migration Framework
                       │
                       ▼
                 Schema Evolution

Migration Framework y Runtime comparten la misma base, pero tienen responsabilidades distintas.

55. Tests de Architecture Boundary

TASK-001-17 deberá proteger también:

Domain
   ❌
Alembic

Application
   ❌
Alembic

API
   ❌
Alembic runtime calls

Las migraciones solo pueden vivir en la zona definida.

56. Tests específicos
MIG-UNIT-001

Migration configuration carga correctamente.

MIG-UNIT-002

Database URL proviene de configuración segura.

MIG-UNIT-003

Migration IDs son únicos.

MIG-UNIT-004

Migration files tienen naming válido.

MIG-INT-001

Fresh database alcanza HEAD.

MIG-INT-002

Upgrade path completo funciona.

MIG-INT-003

Migration failure no marca incorrectamente la revisión como aplicada.

MIG-INT-004

Migration runner detecta current revision.

MIG-SEC-001

No existen credentials hardcoded.

MIG-SEC-002

No se imprimen passwords.

MIG-SEC-003

No se registran connection strings completas.

MIG-SEC-004

Destructive migrations son detectables.

57. Migration Static Checks

El CI podrá comprobar:

migration filename valid
revision unique
down_revision valid
no duplicate revision
no missing parent
no obvious destructive operation without metadata/review

Esto evita romper la cadena de migraciones.

58. Acceptance Criteria
AC-01

Given una base PostgreSQL vacía,
When se ejecuta el migration runner,
Then puede construirse el esquema hasta HEAD.

AC-02

Given una base con una versión anterior,
When se ejecutan las nuevas migrations,
Then se alcanza la versión actual correctamente.

AC-03

Given una migration aplicada en un entorno compartido,
When se modifica su archivo,
Then CI debe detectar la inconsistencia o la política de revisión correspondiente debe bloquearla.

AC-04

Given el ORM cambia un modelo,
When inicia la aplicación,
Then el esquema no se modifica automáticamente.

AC-05

Given una migration contiene un cambio destructivo,
When entra al pipeline,
Then debe ser identificable y someterse al control correspondiente.

AC-06

Given una migration falla,
When termina su ejecución,
Then la aplicación no debe asumir que el esquema fue actualizado.

AC-07

Given una migration necesita credenciales,
When se ejecuta,
Then las obtiene mediante configuración segura.

AC-08

Given una migration,
When se ejecuta,
Then no registra secretos.

AC-09

Given dos migration runners concurrentes,
When intentan modificar el mismo esquema,
Then la ejecución debe estar serializada/controlada.

AC-10

Given un deployment,
When la migration validation falla,
Then el pipeline debe detenerse.

AC-11

Given un cambio backward-compatible,
When se despliega,
Then debe poder coexistir con la versión anterior durante el rollout cuando sea necesario.

AC-12

Given una migration de datos,
When procesa información existente,
Then debe ser reproducible/idempotente según su naturaleza.

AC-13

Given una tabla futura tenant-scoped,
When se crea mediante migration,
Then su modelo de tenant isolation debe estar explícitamente definido.

AC-14

Given una tabla futura de Outbox,
When se crea,
Then su evolución debe estar versionada mediante migration.

AC-15

Given un nuevo desarrollador,
When crea una migration siguiendo el procedimiento,
Then puede ejecutar y validar la migration sin modificar manualmente PostgreSQL.

59. Definition of Done
 Alembic configurado.
 alembic.ini preparado sin secretos.
 env.py integrado con configuración EVOXA.
 Directory migrations/versions creado.
 Migration runner funcionando.
 Migration metadata funcionando.
 Current revision detectable.
 Upgrade path funcionando.
 Fresh database test funcionando.
 Migration naming convention definida.
 Migration immutability definida.
 Destructive migration policy definida.
 Expand/Contract strategy definida.
 Migration locking controlado.
 CI migration validation preparado.
 Schema drift detection preparado.
 Migration observability preparada.
 Secret leakage tests implementados.
 Architecture boundaries actualizados.
 Documentation actualizada.
 Traceability registrada.
60. Traceability
Blueprint
    │
    ▼
Architecture Map
    │
    ▼
ESP-0001
    │
    ├── ESP-0001/01 Domain Model
    ├── ESP-0001/02 Architecture
    ├── ESP-0001/06 Sessions & Tokens
    ├── ESP-0001/09 Audit
    ├── ESP-0001/11 Persistence
    ├── ESP-0001/12 Events
    ├── ESP-0001/13 Testing
    ├── ESP-0001/14 Implementation
    └── ESP-0001/15 Deployment
    │
    ▼
ADR-006   PostgreSQL Source of Truth
ADR-012   PostgreSQL Schema
ADR-013   Data Retention
ADR-015   Partitioning
ADR-016   Concurrency
ADR-017   Migrations
ADR-029   Disaster Recovery
ADR-030   DR Testing
    │
    ▼
IS-001
    │
    ▼
TASK-001-19
    │
    ▼
Migration Framework
61. Resultado arquitectónico

Con esta tarea EVOXA pasa de:

Código
  │
  ▼
ORM
  │
  ▼
Database

a:

                    ┌─────────────────┐
                    │   Source Code   │
                    └────────┬────────┘
                             │
                             ▼
                       Domain Model
                             │
                             ▼
                       Migration
                             │
                             ▼
                    Migration Validation
                             │
                             ▼
                       PostgreSQL

Y, sobre todo:

❌ create_all()
❌ auto-sync
❌ cambios manuales permanentes
❌ schema drift silencioso

                VS

✅ versioned migrations
✅ reviewed migrations
✅ reproducible schema
✅ CI validation
✅ controlled deployment
✅ auditable evolution
62. Estado de IS-001
TASK-001-01  API Runtime Foundation             ✅
TASK-001-02  Identity Domain Package             ✅
TASK-001-03  Layer Architecture                  ✅
TASK-001-04  Identity Shared Kernel              ✅
TASK-001-05  Domain Events Foundation             ✅
TASK-001-06  Repository Contracts                ✅
TASK-001-07  Unit of Work                        ✅
TASK-001-08  PostgreSQL Infrastructure            ✅
TASK-001-09  Redis Infrastructure                 ✅
TASK-001-10  Configuration & Secrets              ✅
TASK-001-11  Security Foundation                  ✅
TASK-001-12  Tenant Context Foundation            ✅
TASK-001-13  Actor Context                        ✅
TASK-001-14  Correlation & Trace Context          ✅
TASK-001-15  Exception Model                      ✅
TASK-001-16  Dependency Injection                 ✅
TASK-001-17  Architecture Boundary Tests          ✅
TASK-001-18  Initial Observability                ✅
TASK-001-19  Initial Migration Framework          🟢
Siguiente tarea

TASK-001-20 — Test Foundation

Esta será la última gran pieza de infraestructura de calidad de IS-001 antes de entrar en la parte final de CI/Governance. Aquí estableceremos la estructura oficial de testing de EVOXA, fixtures, test configuration, unit/integration/API test foundation, aislamiento de base de datos, test doubles, coverage y las bases para los Quality Gates definidos en ESP-0001/13.
