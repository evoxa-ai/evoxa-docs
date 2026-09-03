ADR-IDENTITY-017 — Database Migration Strategy

Estado: Proposed
Versión: 1.0
Fecha: 2026-09-02
Dominio: Identity
Prioridad: Critical
Tipo: Data Architecture / Deployment / Reliability / Database Governance

1. Título

ADR-IDENTITY-017 — Database Migration Strategy

2. Contexto

El dominio Identity utiliza PostgreSQL como fuente transaccional de verdad y contiene estructuras críticas como:

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

La arquitectura de EVOXA establece además:

despliegues Dev → QA → Staging → Production;
contenedores;
servicios stateless;
zero downtime;
rolling deployment;
blue/green;
canary/progressive deployment;
PostgreSQL como fuente de verdad;
Transactional Outbox;
alta disponibilidad;
observabilidad;
RTO < 1 hora;
RPO < 15 minutos.

La evolución del esquema de base de datos debe ser compatible con este modelo.

Una modificación aparentemente sencilla puede afectar simultáneamente:

Database
   ↓
API
   ↓
Application
   ↓
Domain
   ↓
Events
   ↓
Consumers
   ↓
Reports / Analytics

Por ello, el esquema de PostgreSQL debe evolucionar de forma versionada, reproducible, auditable y compatible con despliegues progresivos.

3. Problema

Modificar directamente el esquema de producción puede producir:

downtime;
pérdida de datos;
incompatibilidad entre versiones de API;
errores durante rolling deployment;
bloqueo de tablas;
migraciones irreversibles;
incompatibilidad entre productores y consumidores;
corrupción lógica de datos;
fallos en eventos;
problemas de rollback.

Un escenario particularmente peligroso sería:

API v1
   ↓
Old Schema

Deploy API v2
   ↓
New Schema only

Si durante un rolling deployment todavía existen instancias de API v1, podríamos tener:

API v1 ─────┐
            ├── PostgreSQL
API v2 ─────┘

con dos versiones de aplicación esperando estructuras diferentes.

Por lo tanto:

Las migraciones deben diseñarse para coexistencia temporal entre versiones de aplicación.

4. Decisión

EVOXA adoptará una estrategia de:

Versioned, Reproducible, Expand-and-Contract Database Migrations

Las migraciones serán:

versionadas;
ejecutables de forma determinista;
auditables;
promovibles entre ambientes;
compatibles con rolling/blue-green/canary;
diseñadas para zero downtime cuando sea técnicamente posible;
independientes del ciclo de arranque normal de la aplicación.
5. Principio fundamental

La base de datos no será modificada automáticamente al iniciar la aplicación.

Queda expresamente prohibido utilizar en producción un mecanismo equivalente a:

application startup
       ↓
automatic schema synchronization
       ↓
database modification

El esquema deberá evolucionar mediante:

Migration Artifact
       ↓
QA
       ↓
Staging
       ↓
Approval
       ↓
Production

Esto es especialmente importante para evitar que una nueva instancia de API modifique inesperadamente el esquema de producción.

6. Migration Source of Truth

El conjunto de migraciones versionadas será la fuente reproducible de evolución del esquema.

Conceptualmente:

Repository
   │
   └── migrations/
          │
          ├── 001_initial_identity
          ├── 002_memberships
          ├── 003_sessions
          ├── 004_refresh_tokens
          └── ...

La estructura exacta de carpetas y herramienta se definirá durante la implementación.

Lo importante es que:

Una base de datos nueva y una base de datos existente puedan alcanzar el mismo estado aplicando la misma secuencia de migraciones.

7. Migration Version

Cada migración deberá tener un identificador único y ordenable.

Conceptualmente:

Migration ID
Migration Name
Created At
Checksum
Applied At
Execution Status

Esto permitirá determinar:

qué migraciones fueron aplicadas;
cuáles faltan;
cuándo fueron ejecutadas;
si fueron modificadas;
qué versión de esquema existe.
8. Migration Ledger

La base de datos deberá disponer de un registro de migraciones aplicadas.

Conceptualmente:

schema_migrations
-------------------------
version
name
checksum
applied_at
execution_time
status

La estructura exacta será definida durante la implementación.

Este registro permitirá detectar:

Environment A → Migration 017
Environment B → Migration 015
Production     → Migration 016

y evitar aplicar accidentalmente una migración dos veces.

9. Immutable Migrations

Una migración que ya fue ejecutada en un ambiente compartido o productivo deberá considerarse inmutable.

No se deberá modificar silenciosamente:

migration_017

después de haber sido aplicada.

Si existe un error:

migration_017
      ↓
correction
      ↓
migration_018

en lugar de reescribir la historia.

Esto permite mantener:

trazabilidad;
reproducibilidad;
auditoría;
consistencia entre ambientes.
10. Expand-and-Contract

Para cambios incompatibles se utilizará preferentemente el patrón:

Expand → Migrate → Switch → Contract

Ejemplo:

Estado inicial
users
└── old_field
Expand
users
├── old_field
└── new_field

La aplicación puede continuar funcionando.

Migrate
old_field
    ↓
new_field

Se realiza backfill progresivamente.

Switch

La aplicación comienza a utilizar:

new_field
Contract

Cuando ninguna versión antigua lo necesite:

old_field
    ↓
DROP
11. Compatibilidad hacia atrás

Durante despliegues progresivos, las migraciones deberán mantener compatibilidad hacia atrás cuando sea necesario.

Por ejemplo:

DB Schema N
     ↑
API N
API N+1

deberán poder coexistir temporalmente cuando el mecanismo de deployment lo requiera.

Esto es fundamental para:

rolling deployments;
blue/green;
canary;
rollback.
12. Ejemplo de evolución de columna

Supongamos:

users.name

debe dividirse en:

users.first_name
users.last_name

No se hará inmediatamente:

DROP name
ADD first_name
ADD last_name

porque una versión anterior podría seguir utilizando name.

Se hará:

1. ADD first_name
2. ADD last_name
3. Deploy compatible
4. Backfill
5. Deploy que usa nuevos campos
6. Validación
7. Retirar name
13. Additive Changes

Las modificaciones aditivas son preferidas inicialmente.

Ejemplos:

agregar tabla;
agregar columna nullable;
agregar índice compatible;
agregar nuevo estado;
agregar nueva estructura.

Posteriormente podrá realizarse la eliminación o transformación.

Esto minimiza el riesgo de incompatibilidad.

14. Destructive Changes

Las operaciones destructivas requieren mayor control.

Ejemplos:

DROP COLUMN
DROP TABLE
DROP INDEX
ALTER TYPE incompatible

deberán pasar por una estrategia de:

Detect dependencies
      ↓
Expand
      ↓
Migrate
      ↓
Deploy
      ↓
Validate
      ↓
Contract

No deberán ejecutarse como parte de una migración improvisada durante un deploy normal.

15. Data Backfill

Los backfills grandes deberán tratarse como operaciones independientes cuando su duración pueda ser significativa.

Evitar:

BEGIN
  UPDATE millions_of_rows
  ...
COMMIT

si esto produce:

locks prolongados;
saturación;
crecimiento de WAL;
impacto sobre producción.

Preferido:

Batch 1
Batch 2
Batch 3
...
Batch N

con:

límites;
métricas;
pausabilidad;
retry;
control de errores.
16. Large Table Migration

Las tablas potencialmente grandes como:

audit_events;
security_events;
outbox_events;
eventualmente refresh_tokens;

requieren especial cuidado.

Una migración sobre estas tablas deberá evaluar:

tamaño;
número de filas;
índices;
locks;
tiempo estimado;
WAL;
impacto en replicas;
impacto en consultas;
impacto en backup.
17. Index Creation

La creación de índices grandes deberá diseñarse para minimizar interrupciones.

Cuando PostgreSQL y el caso concreto lo permitan, se evaluarán mecanismos equivalentes a:

CREATE INDEX CONCURRENTLY

en lugar de bloquear innecesariamente operaciones.

La utilización exacta deberá validarse según:

versión PostgreSQL;
tipo de índice;
contexto transaccional;
tamaño;
impacto operacional.
18. Constraints

La introducción de constraints también deberá considerar producción.

Ejemplo:

ADD NOT NULL

sobre una tabla grande puede requerir una estrategia gradual.

Preferido conceptualmente:

Add nullable
      ↓
Backfill
      ↓
Validate
      ↓
Enforce constraint

cuando sea necesario.

19. Foreign Keys

Las nuevas relaciones deberán introducirse cuidadosamente.

Ejemplo:

refresh_tokens.session_id
       ↓
sessions.id

Antes de imponer una constraint estricta deberán verificarse:

datos existentes;
registros huérfanos;
cardinalidad;
impacto;
locking.

No se deberá asumir que el esquema histórico ya cumple automáticamente una nueva constraint.

20. Enum y Lifecycle States

Las entidades de Identity utilizan estados como:

User:
PENDING
ACTIVE
SUSPENDED
DISABLED

y:

Membership:
INVITED
ACTIVE
SUSPENDED
REMOVED

La adición de nuevos estados deberá ser compatible con versiones anteriores de la aplicación cuando sea necesario.

No se deberá introducir un nuevo estado en DB si las versiones activas de la aplicación no pueden manejarlo correctamente.

21. Transactional Migrations

Las migraciones deberán utilizar transacciones cuando sea seguro y soportado por la operación.

Pero no todas las operaciones de PostgreSQL son compatibles con una única transacción de migración.

Por ello:

No se impondrá "todo dentro de una transacción" como regla absoluta.

La estrategia deberá elegir entre:

transactional migration;
non-transactional migration;
phased migration;

según la operación.

22. Schema Locking

Antes de ejecutar una migración en producción se deberá evaluar:

locks requeridos;
duración;
concurrencia;
impacto en API;
impacto en replicas;
posibilidad de timeout.

Las migraciones no deben bloquear indefinidamente el tráfico de Identity.

23. Migration Timeout

Las operaciones de migración deberán tener controles de tiempo adecuados.

Si una migración queda bloqueada:

Migration
   ↓
Waiting for lock
   ↓
Timeout

deberá fallar de forma controlada y generar observabilidad.

No se debe mantener una operación de schema change indefinidamente.

24. Migration Ordering

Las migraciones deberán ejecutarse en orden.

Conceptualmente:

015
 ↓
016
 ↓
017
 ↓
018

No deberá permitirse que:

018

se aplique sobre una base que todavía requiere:

016
017

salvo que la herramienta y la estrategia hayan definido explícitamente dependencias equivalentes.

25. Environment Promotion

El mismo artefacto de migración deberá promoverse:

DEV
 ↓
QA
 ↓
STAGING
 ↓
PRODUCTION

No se deberá reconstruir manualmente una migración para producción.

Esto reduce diferencias entre ambientes.

26. CI/CD Integration

El pipeline deberá incluir validaciones de migraciones.

Conceptualmente:

Commit
 ↓
Lint
 ↓
Unit Tests
 ↓
Migration Validation
 ↓
Integration Tests
 ↓
Security Tests
 ↓
Build
 ↓
Deploy QA
 ↓
Migration Test
 ↓
Staging
 ↓
Production Approval

Las migraciones críticas deberán ser verificadas antes del deployment.

27. Migration Testing

Cada migración deberá probarse sobre:

Base vacía
Empty DB
   ↓
All migrations
   ↓
Expected schema
Base existente
Previous version
   ↓
Migration
   ↓
Expected schema
Datos reales representativos
Representative dataset
   ↓
Migration
   ↓
Integrity validation
Rollback / recovery

Cuando sea técnicamente viable.

28. Schema Drift

EVOXA deberá detectar diferencias entre:

Expected Schema
       vs
Actual Schema

El schema drift puede producir:

errores de runtime;
migraciones inesperadas;
incompatibilidad;
despliegues fallidos.

Por ello, producción no deberá ser modificada manualmente salvo procedimientos excepcionales y controlados.

29. Emergency Changes

Puede existir una situación donde sea necesario realizar un cambio manual urgente.

Esto deberá considerarse una excepción.

El procedimiento será conceptualmente:

Incident
   ↓
Emergency DB Change
   ↓
Approval
   ↓
Controlled Execution
   ↓
Audit
   ↓
Migration Codification
   ↓
Repository

El cambio manual deberá posteriormente quedar representado en una migración versionada para evitar divergencia.

30. Rollback Strategy

No todas las migraciones deberán asumir que existe un rollback SQL sencillo.

Especialmente:

DROP
Data transformation
Data deletion

pueden ser irreversibles.

Por ello se distinguirá:

Rollback de aplicación
API v2
 ↓
API v1
Rollback de schema
Migration N+1
 ↓
Migration N
Recovery
Backup / Restore

Estas estrategias no son equivalentes.

31. Expand-and-Contract y Rollback

El patrón expand/contract permitirá separar:

Compatible phase
       ↓
Migration
       ↓
Application switch
       ↓
Validation
       ↓
Destructive cleanup

Esto hace posible realizar rollback de aplicación durante la fase compatible.

Después de ejecutar el Contract, el rollback puede dejar de ser posible.

Por ello:

Las operaciones destructivas deben ejecutarse únicamente después de confirmar que el sistema ya no depende de la estructura anterior.

32. Zero Downtime

El objetivo será que las migraciones compatibles puedan ejecutarse sin downtime.

Conceptualmente:

Users
 │
 ▼
Load Balancer
 │
 ├── API v1
 ├── API v1
 └── API v2
        │
        ▼
    Compatible DB

Posteriormente:

API v2
 │
 ▼
New Schema

y finalmente:

Contract
33. Deployment Strategy

La estrategia deberá ser compatible con:

rolling;
blue/green;
canary;
progressive delivery.

La secuencia preferida será:

1. Expand DB
2. Deploy compatible application
3. Backfill / migrate data
4. Validate
5. Switch traffic
6. Observe
7. Contract DB
34. Partitioning

Las decisiones de:

ADR-IDENTITY-015 — Database Partitioning

deberán reflejarse en migraciones versionadas.

Por ejemplo:

Create partition
Create index
Attach partition
Validate

La creación de nuevas particiones no deberá depender de modificaciones manuales no reproducibles.

35. Encryption

Los cambios relacionados con:

encrypted columns;
key references;
protected data;
encryption metadata;

deberán coordinarse con:

ADR-IDENTITY-014 — Encryption & Secret Protection.

Nunca se deberá ejecutar una migración que elimine el único acceso a información cifrada sin verificar previamente la estrategia de recuperación.

36. Audit

Las migraciones de producción deberán ser auditables.

Deberá quedar registro de:

migración;
versión;
timestamp;
ambiente;
actor/proceso;
resultado;
duración;
error cuando corresponda.

Los logs de migración no deberán revelar:

passwords;
tokens;
claves;
secretos.
37. Security

Las migraciones deberán ejecutarse con una identidad que posea únicamente los permisos necesarios.

No se deberá utilizar una credencial administrativa ilimitada desde la aplicación.

Conceptualmente:

Deployment System
      ↓
Migration Identity
      ↓
PostgreSQL

separada de:

Application Runtime Identity
      ↓
Normal CRUD

Esto refuerza Least Privilege.

38. Production Permissions

La aplicación en runtime no debería necesitar permisos equivalentes a:

DROP DATABASE
ALTER ANY TABLE
CREATE ANY SCHEMA

El proceso de migration deberá estar separado del proceso normal de ejecución de la API.

39. Backup Before Critical Migration

Antes de migraciones potencialmente destructivas o de alto riesgo deberá existir una estrategia de recovery adecuada.

No se deberá asumir que:

migration succeeds

implica:

data recoverable

La política exacta de backup y recovery se coordinará con:

ADR-IDENTITY-029 — Disaster Recovery & Backup.

40. Migration Observability

Se deberán medir:

migration duration;
migration success/failure;
lock waits;
schema drift;
backfill progress;
rows migrated;
errors;
replication lag;
database health.

Métricas conceptuales:

identity_migration_duration
identity_migration_success
identity_migration_failure
identity_migration_lock_wait
identity_migration_backfill_progress
identity_database_schema_version
identity_database_schema_drift
41. Migration Governance

Una migración crítica deberá incluir:

propósito;
impacto;
dependencias;
preconditions;
migration steps;
validation;
rollback/recovery strategy;
expected duration;
lock analysis;
compatibility analysis.

Especialmente para producción.

42. Data Validation

Después de una migración se deberán validar invariantes.

Ejemplos:

users
   ↓
no duplicate identities
memberships
   ↓
user_id + organization_id unique
role_permissions
   ↓
no invalid references
refresh_tokens
   ↓
valid session references
audit_events
   ↓
append-only integrity

La validación deberá ser proporcional al riesgo del cambio.

43. Migration and Events

Los cambios de esquema no deben confundirse con Domain Events.

Una migración:

ALTER TABLE

no implica automáticamente:

Domain Event

Sin embargo, cuando una migración transforme datos de negocio, deberá evaluarse si existe impacto en:

Domain State;
Audit;
Integration Events;
projections;
consumers.

No se deben emitir eventos falsos simplemente para representar operaciones técnicas de schema.

44. Migration and Outbox

Las migraciones sobre:

outbox_events

deberán considerar:

publishers activos;
workers;
consumers;
schema versions;
retries;
DLQ;
replay.

Una modificación incompatible podría romper consumidores o publishers que todavía ejecuten una versión anterior.

45. Migration and API Contracts

Una migración que cambie estructuras utilizadas por API deberá respetar:

IS-IDENTITY-014 — Identity API Contracts.

El cambio deberá considerar:

Database schema
      ↕
Application model
      ↕
API contract

No deberá asumirse que cambiar una columna internamente es transparente para API.

46. Consecuencias positivas
Seguridad

Reduce cambios manuales y no auditados.

Reliability

Reduce riesgo de corrupción o downtime.

Reproducibilidad

Permite reconstruir el esquema desde cero.

Zero Downtime

Facilita rolling/blue-green/canary.

Rollback

Mantiene la aplicación compatible durante fases de evolución.

Auditoría

Cada cambio de schema queda registrado.

Escalabilidad

Permite evolucionar las estructuras de alto volumen.

DevOps

Integra la evolución de PostgreSQL dentro de CI/CD.

47. Consecuencias negativas

La estrategia añade:

disciplina;
tooling;
testing;
governance;
coordinación entre DB y aplicación;
complejidad para cambios destructivos.

El patrón expand/contract también puede requerir varias versiones de aplicación para completar un único cambio.

Sin embargo, esta complejidad se considera aceptable debido al carácter crítico de Identity.

48. Alternativas consideradas
Alternativa A — Auto-sync al iniciar API

Rechazada.

Puede producir cambios inesperados y conflictos durante despliegues.

Alternativa B — SQL manual en producción

Rechazada como estrategia normal.

No proporciona reproducibilidad suficiente.

Alternativa C — Una migración gigante por release

Rechazada.

Incrementa riesgo y dificulta diagnóstico.

Alternativa D — Siempre hacer rollback automático

Rechazada.

Algunas transformaciones de datos no son reversibles.

Alternativa E — Expand-and-Contract

Seleccionada.

Permite:

compatibilidad;
zero downtime;
rolling deployments;
rollback de aplicación;
evolución progresiva.
49. Non-Negotiables
Las migraciones deben estar versionadas.
Las migraciones ejecutadas son inmutables.
No habrá auto-sync de schema en producción.
El schema debe ser reproducible.
Production debe recibir el mismo artefacto validado.
Los cambios incompatibles deben utilizar Expand-and-Contract cuando corresponda.
Las operaciones destructivas requieren validación explícita.
Las migraciones deben integrarse con CI/CD.
Deben probarse sobre bases nuevas y existentes.
Debe existir detección de schema drift.
Los backfills grandes deben ser controlados.
Las migraciones deben considerar locks.
Deben existir estrategias de timeout.
No se deben realizar transacciones enormes innecesariamente.
Los cambios de producción deben ser auditables.
Los secretos no deben aparecer en migraciones ni logs.
Runtime y migration privileges deben estar separados.
Backup/recovery debe considerarse para cambios de alto riesgo.
Partitioning debe evolucionar mediante migraciones.
Las migraciones no deben romper rolling/blue-green/canary.
No se debe asumir que todo cambio es reversible.
La historia de migraciones no debe reescribirse.
50. Traceability
Blueprint

Relacionado con:

Reference Architecture;
Engineering Standards;
Deployment;
Zero Downtime;
PostgreSQL;
Continuous Evolution;
Reliability;
Security;
Disaster Recovery.
ESP

ESP-0001 — Identity Domain

Principalmente:

persistence;
security;
audit;
sessions;
refresh tokens;
events.
Implementation Stories

Principalmente:

IS-IDENTITY-012 — Audit & Security Events
IS-IDENTITY-013 — Domain Events & Outbox
IS-IDENTITY-014 — Identity API Contracts
IS-IDENTITY-015 — Identity Persistence
IS-IDENTITY-016 — Identity Testing & Quality
IS-IDENTITY-017 — Identity Runtime & Deployment
51. ADR relacionados
ADR-IDENTITY-012 — PostgreSQL Schema
ADR-IDENTITY-013 — Data Retention
ADR-IDENTITY-014 — Encryption & Secret Protection
ADR-IDENTITY-015 — Database Partitioning
ADR-IDENTITY-016 — Concurrency & Transaction Strategy
ADR-IDENTITY-026 — API Pagination / Filtering / Idempotency
ADR-IDENTITY-027 — Runtime / Secrets / KMS
ADR-IDENTITY-028 — Observability / Operational Security
ADR-IDENTITY-029 — Disaster Recovery / Backup
52. Dependencias

La estrategia depende especialmente de:

ADR-012
PostgreSQL Schema
      ↓
ADR-015
Partitioning
      ↓
ADR-016
Concurrency
      ↓
ADR-017
Migration Strategy

Y se integra con:

ADR-011
Runtime & Deployment

ADR-013
Retention

ADR-014
Encryption

ADR-007
Event Architecture
53. Decisiones pendientes

Quedan pendientes:

Migration Tool

Definir herramienta concreta compatible con el stack de implementación.

Naming

Definir convención final de:

migration IDs;
nombres;
archivos.
Schema Version

Definir mecanismo definitivo para identificar la versión del schema.

Deployment Ownership

Definir si las migraciones serán ejecutadas por:

CI/CD;
Kubernetes Job;
deployment controller;
herramienta externa.
Lock Policy

Definir:

lock timeout;
statement timeout;
migration timeout.
Backfill Framework

Definir mecanismo para migraciones masivas.

Drift Detection

Definir herramienta y frecuencia.

Production Approval

Definir qué migraciones requieren aprobación manual.

54. Criterios de aceptación

ADR-IDENTITY-017 podrá pasar de Proposed a Accepted cuando exista:

 herramienta de migrations seleccionada;
 convention de versionado;
 migration ledger;
 política de inmutabilidad;
 estrategia Expand-and-Contract;
 política de rollback/recovery;
 estrategia de backfill;
 estrategia de index migrations;
 estrategia de constraints;
 lock/timeout policy;
 schema drift detection;
 integración CI/CD;
 promoción Dev → QA → Staging → Production;
 pruebas sobre DB nueva;
 pruebas sobre DB existente;
 pruebas de migración con datos representativos;
 pruebas de rollback/recovery;
 observabilidad;
 auditoría;
 permisos separados entre runtime y migrations;
 runbook de migraciones;
 estrategia de emergency migration;
 estrategia de backup para cambios críticos.
55. Cierre de la cadena de persistencia

Con este ADR queda definida la primera cadena completa de decisiones de persistencia de Identity:

ADR-IDENTITY-012
PostgreSQL Schema
        │
        ▼
ADR-IDENTITY-013
Data Retention
        │
        ▼
ADR-IDENTITY-014
Encryption & Secret Protection
        │
        ▼
ADR-IDENTITY-015
Database Partitioning
        │
        ▼
ADR-IDENTITY-016
Concurrency & Transaction Strategy
        │
        ▼
ADR-IDENTITY-017
Database Migration Strategy

La arquitectura resultante establece una regla importante para EVOXA:

El esquema de datos es parte del producto y de la arquitectura; no es un detalle accidental de implementación.

Su evolución deberá ser versionada, reproducible, compatible, observable, auditable y segura, especialmente porque Identity constituye una de las superficies más críticas de toda la plataforma.

Estado final: Proposed.
