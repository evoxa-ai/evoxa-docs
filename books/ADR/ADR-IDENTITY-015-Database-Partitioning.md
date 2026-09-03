ADR-IDENTITY-015 — Database Partitioning

Estado: Proposed
Versión: 1.0
Fecha: 2026-09-02
Dominio: Identity
Prioridad: High / Critical
Tipo: Data Architecture / Scalability / Performance / Security

1. Título

ADR-IDENTITY-015 — Database Partitioning

2. Contexto

El dominio Identity utiliza PostgreSQL como fuente transaccional de verdad y contiene diferentes categorías de información con comportamientos muy distintos.

Entre las estructuras definidas para Identity se encuentran:

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

No todas estas tablas presentan el mismo volumen, patrón de acceso o necesidad de retención.

Por ejemplo:

users
organizations
roles
permissions
        │
        └── crecimiento relativamente controlado

sessions
refresh_tokens
mfa_challenges
        │
        └── alto churn

audit_events
security_events
outbox_events
        │
        └── crecimiento continuo / histórico

El Blueprint establece requisitos de:

escalabilidad;
alta disponibilidad;
observabilidad;
auditoría;
retención;
replay de eventos;
aislamiento por tenant;
rendimiento;
evolución continua.

Por ello, debe definirse cómo evolucionará la persistencia cuando determinadas tablas alcancen volúmenes elevados.

3. Problema

Si todas las tablas crecieran indefinidamente sin una estrategia de particionamiento, podrían aparecer problemas relacionados con:

tamaño de índices;
mantenimiento de tablas;
consultas históricas;
eliminación/archivado;
vacuum;
rendimiento de búsquedas;
operaciones de retención;
backups;
recuperación;
crecimiento de audit_events;
crecimiento de security_events;
crecimiento de outbox_events;
crecimiento de sesiones y tokens históricos.

Por otro lado, particionar indiscriminadamente todas las tablas también introduce complejidad innecesaria.

Por lo tanto, EVOXA necesita una política que determine:

qué tablas deben permanecer simples y cuáles son candidatas a particionamiento.

4. Decisión

EVOXA adoptará una estrategia de partitioning selectivo y basado en comportamiento, no un particionamiento indiscriminado.

La decisión inicial será:

                    Identity PostgreSQL
                           │
             ┌─────────────┴─────────────┐
             │                           │
       Tablas core                 Tablas históricas
             │                           │
             ▼                           ▼
       Sin particionar             Candidatas a
       inicialmente                particionamiento

El particionamiento será utilizado cuando exista una justificación demostrable basada en:

volumen;
crecimiento;
patrón de consultas;
retención;
archivado;
mantenimiento;
rendimiento;
aislamiento operacional.
5. Principio fundamental

Partitioning no es una estrategia de tenant isolation.

Aunque el tenant_id pueda formar parte de claves e índices, no se deberá asumir:

partition = tenant

como mecanismo general de seguridad.

La seguridad deberá continuar dependiendo de:

autorización;
Tenant Context;
constraints;
políticas;
application enforcement;
controles de acceso.

El particionamiento es una estrategia de persistencia y operación, no el mecanismo principal de autorización.

6. Clasificación de tablas

Las tablas de Identity se clasificarán inicialmente en tres categorías.

Categoría A — Core transactional
users
organizations
memberships
roles
permissions
role_permissions
membership_roles

Estas tablas contienen el estado actual del dominio.

Categoría B — Estado operativo de alto churn
sessions
refresh_tokens
mfa_challenges
trusted_devices
outbox_events

Estas tablas pueden presentar crecimiento y eliminación/expiración frecuente.

Categoría C — Histórico / auditoría
audit_events
security_events

Son las principales candidatas a particionamiento debido a:

crecimiento continuo;
consultas por tiempo;
retención;
archivado;
investigación histórica.
7. Estrategia para Core Transactional

Inicialmente, las tablas:

users
organizations
memberships
roles
permissions
role_permissions
membership_roles

permanecerán sin particionar.

La razón es que:

su crecimiento es relativamente más controlable;
contienen estado actual;
tienen relaciones fuertes;
requieren joins frecuentes;
el particionamiento añadiría complejidad;
no existe evidencia suficiente para justificarlo inicialmente.

Esto podrá cambiar mediante una decisión arquitectónica posterior basada en métricas reales.

8. Audit Events

audit_events será la principal candidata a particionamiento.

El patrón preferido será conceptualmente:

audit_events
       │
       ├── partition_YYYY_MM
       ├── partition_YYYY_MM
       ├── partition_YYYY_MM
       └── ...

La dimensión temporal es apropiada porque las consultas de auditoría normalmente utilizan:

rango de fechas;
tenant;
actor;
tipo de evento;
recurso;
correlación.
9. Security Events

security_events seguirá una estrategia similar.

Conceptualmente:

security_events
       │
       ├── period_1
       ├── period_2
       ├── period_3
       └── ...

El particionamiento temporal permitirá:

consultar periodos específicos;
gestionar retención;
archivar periodos antiguos;
reducir operaciones sobre toda la tabla;
simplificar mantenimiento.
10. Outbox Events

outbox_events requiere un tratamiento diferente.

Su objetivo principal es servir como mecanismo de transición:

Domain Transaction
       ↓
Outbox
       ↓
Publisher
       ↓
Event Platform

Una vez publicados los eventos y superado el periodo operacional requerido, podrán entrar en una política de:

PUBLISHED
   ↓
RETENTION
   ↓
ARCHIVE
   ↓
DELETE

Por ello, outbox_events es también candidata a particionamiento, especialmente cuando el volumen sea elevado.

Sin embargo, la estrategia deberá evitar que el particionamiento interfiera con:

polling;
claiming;
locking;
publicación;
reintentos;
DLQ.
11. Sessions

sessions puede crecer significativamente en una plataforma multiusuario.

Sin embargo, no se establece inicialmente que deba particionarse.

La primera estrategia será:

índices adecuados;
expiración;
lifecycle;
limpieza controlada;
consultas optimizadas.

El particionamiento será considerado cuando métricas reales demuestren que el volumen justifica su introducción.

12. Refresh Tokens

refresh_tokens presenta un patrón de alto churn debido a:

rotación;
expiración;
revocación;
token families;
reuse detection.

Sin embargo, no se establecerá automáticamente particionamiento.

Su estrategia estará condicionada por:

volumen;
duración de retención;
consultas por family_id;
consultas por session_id;
comportamiento de rotación;
rendimiento de concurrencia.

La decisión criptográfica y de concurrencia pertenece a:

ADR-IDENTITY-025 — Refresh Token Cryptography & Concurrency.

13. MFA Challenges

mfa_challenges puede presentar un volumen elevado, pero sus registros tienen normalmente una vida operacional relativamente corta.

Por ello, la estrategia inicial será:

Create
  ↓
Verify / Fail / Expire
  ↓
Retention period
  ↓
Cleanup

El particionamiento solo se introducirá si el volumen operacional lo requiere.

14. Método de particionamiento

Para las tablas históricas, la estrategia preferida será particionamiento temporal.

Conceptualmente:

PARTITION BY RANGE (timestamp)

La unidad exacta:

diaria;
semanal;
mensual;

no se fija en este ADR.

Deberá determinarse mediante:

volumen esperado;
consultas;
tamaño de partición;
frecuencia de retención;
capacidad operacional.

No se debe crear una partición por cada tenant como estrategia predeterminada.

15. Tenant-aware Partitioning

Aunque no se adopte:

tenant → partition

las tablas tenant-aware deberán mantener:

tenant_id

cuando corresponda al modelo.

Por ejemplo:

audit_events
├── tenant_id
├── timestamp
├── actor_id
├── event_type
└── resource_id

Los índices deberán considerar los patrones reales de consulta.

Ejemplo conceptual:

(tenant_id, timestamp)

podría resultar útil para consultas de auditoría por organización y periodo.

La estructura exacta de índices queda pendiente de validación mediante benchmarks.

16. Partition Pruning

Las consultas deberán diseñarse de forma que PostgreSQL pueda aprovechar el particionamiento cuando corresponda.

Por ejemplo:

WHERE timestamp >= X
AND timestamp < Y

debería permitir limitar las particiones consultadas.

No se deberá asumir que particionar una tabla automáticamente mejora cualquier consulta.

17. Retention y Partitioning

El particionamiento deberá integrarse con:

ADR-IDENTITY-013 — Data Retention.

La relación será:

Partitioning
     ↓
Retention
     ↓
Archive
     ↓
Dispose

Una ventaja importante es que, cuando una política permita eliminar un periodo completo, una operación sobre una partición puede ser operacionalmente más eficiente que eliminar millones de filas individualmente.

Sin embargo:

Una partición no define por sí misma la política de retención.

La retención continúa siendo una decisión independiente.

18. Archiving

Las particiones históricas podrán utilizarse como unidad operacional para archivado.

Conceptualmente:

ACTIVE
  ↓
AGED
  ↓
ARCHIVED
  ↓
DISPOSED

El mecanismo exacto de archivado —storage, formato, compresión, cifrado, recuperación— queda fuera de este ADR y deberá definirse en la arquitectura de almacenamiento/retención.

19. Indexes

Cada tabla particionada deberá tener índices diseñados de acuerdo con sus patrones de acceso.

Especial atención a:

Audit
tenant_id
timestamp
actor_id
event_type
resource_type
resource_id
correlation_id
trace_id
Security Events
tenant_id
timestamp
severity
event_type
actor_id
correlation_id
Outbox
status
next_attempt_at
occurred_at
aggregate_id
event_type

Estas columnas son referencias conceptuales derivadas de los modelos definidos previamente; la combinación final deberá validarse con benchmarks y cargas reales.

20. Constraints

El particionamiento no debe eliminar las garantías de integridad del dominio.

Se deben mantener:

primary keys;
foreign keys cuando sean compatibles con la estrategia;
unique constraints;
check constraints;
lifecycle constraints;
tenant consistency;
integridad referencial.

Especial cuidado deberá tenerse con restricciones únicas globales sobre tablas particionadas, dado que determinadas estrategias de PostgreSQL condicionan cómo pueden implementarse.

Esta es una razón adicional para no particionar indiscriminadamente las tablas core.

21. UUID / IDs y Partitioning

El uso de IDs distribuidos aleatoriamente puede tener efectos sobre:

índices;
locality;
page utilization;
inserciones;
rendimiento.

Por ello, la decisión de representación concreta de los IDs deberá coordinarse con:

ADR-IDENTITY-012 — PostgreSQL Schema

y las decisiones futuras de performance.

Este ADR no fija si el sistema utilizará:

UUID;
UUIDv7;
otro identificador.
22. Backup y Restore

El particionamiento deberá ser compatible con:

backups;
restore;
disaster recovery;
RTO < 1h;
RPO < 15m.

La existencia de múltiples particiones no debe crear una estrategia de recuperación excesivamente compleja.

Deberá probarse:

Backup
   ↓
Restore
   ↓
Partition metadata
   ↓
Indexes
   ↓
Constraints
   ↓
Application availability
23. Migrations

Las modificaciones de particionamiento deberán realizarse mediante migraciones versionadas.

Nunca se deberá depender de:

sequelize.sync()

o mecanismo equivalente de auto-modificación en producción.

Las migraciones deberán poder:

crear particiones;
modificar particiones;
crear índices;
preparar nuevas particiones;
archivar particiones;
revertir cambios cuando sea seguro.

La estrategia completa de migrations se definirá en:

ADR-IDENTITY-017 — Database Migration Strategy.

24. Automatic Partition Management

Para tablas con particionamiento temporal, la plataforma deberá poder anticipar la creación de nuevas particiones.

Conceptualmente:

Current partition
       │
       ├── Active
       └── Future partition
              ↓
          Created before
          required period

No se deberá esperar a que una inserción falle porque no existe la partición correspondiente.

El mecanismo puede ser:

migration;
scheduled job;
database procedure;
operational controller;

pero la elección concreta queda pendiente.

25. Failure Handling

El sistema deberá contemplar escenarios como:

Partición futura inexistente

Debe detectarse antes de afectar operaciones.

Partición corrupta

Debe existir estrategia de recuperación.

Índice faltante

Debe detectarse mediante health/validation.

Error durante creación

Debe existir alerta y mecanismo de reparación.

Error durante archivado

No debe provocar pérdida silenciosa de información.

26. Observability

El sistema deberá observar:

tamaño de particiones;
crecimiento;
número de filas;
índices;
consultas lentas;
partition pruning;
vacuum;
bloat;
errores de creación;
retrasos de archivado;
errores de retención.

Métricas conceptuales:

identity_db_partition_size
identity_db_partition_growth
identity_db_partition_creation_failures
identity_db_retention_lag
identity_db_archive_failures

Los nombres finales quedan sujetos a los estándares de observabilidad.

27. Security

El particionamiento no deberá convertirse en mecanismo de bypass de seguridad.

Una consulta:

tenant A

nunca deberá poder acceder a:

tenant B

simplemente porque ambas organizaciones comparten una partición.

La autorización seguirá aplicándose antes de entregar los datos.

28. Performance

El objetivo del particionamiento será apoyar los objetivos de rendimiento definidos por el Blueprint, especialmente:

API P95 <300 ms;
Authentication P95 <500 ms.

No obstante:

Partitioning no garantiza estos objetivos.

El rendimiento deberá comprobarse mediante:

benchmarks;
integration tests;
load tests;
stress tests;
consultas representativas;
crecimiento simulado.
29. Evolución esperada

La estrategia seguirá:

Measure
   ↓
Identify bottleneck
   ↓
Benchmark
   ↓
Partition if justified
   ↓
Validate
   ↓
Observe
   ↓
Evolve

No se implementará particionamiento únicamente porque una tabla pueda crecer.

30. Consecuencias positivas
Escalabilidad

Las tablas históricas podrán crecer durante largos periodos sin concentrar todo el mantenimiento en una única estructura.

Retención

Facilita operaciones por periodos.

Archivado

Las particiones proporcionan unidades naturales de archivado.

Performance

Puede reducir el conjunto de datos considerado en consultas temporales.

Mantenimiento

Permite aislar operaciones sobre datos históricos.

Evolución

Permite cambiar la estrategia de almacenamiento de forma incremental.

31. Consecuencias negativas

Introduce complejidad en:

migrations;
indexes;
constraints;
backups;
monitoring;
retention;
archive;
queries;
operaciones DBA.

También puede provocar problemas si:

se crean demasiadas particiones;
se selecciona una mala clave de particionamiento;
no se realiza partition pruning;
se utilizan índices incorrectos;
se confunde particionamiento con tenant isolation.

Por ello, la estrategia debe mantenerse selectiva.

32. Alternativas consideradas
Alternativa A — No utilizar particionamiento

Rechazada como estrategia definitiva.

Puede funcionar inicialmente, pero no proporciona una estrategia adecuada para tablas históricas de crecimiento continuo.

Alternativa B — Particionar todas las tablas

Rechazada.

Añadiría complejidad innecesaria a las entidades core.

Alternativa C — Una partición por tenant

Rechazada como estrategia general.

Puede producir:

demasiadas particiones;
operaciones complejas;
problemas de crecimiento;
fuerte dependencia del número de organizaciones.

Además, no sustituye tenant isolation.

Alternativa D — Particionamiento únicamente por ID

No seleccionado inicialmente.

No se adapta tan bien a los patrones de:

auditoría;
retención;
archivado;
consultas temporales.
Alternativa E — Particionamiento temporal

Seleccionado como estrategia preferente para datos históricos.

Se alinea con:

audit;
security events;
retention;
archival.
33. Non-Negotiables
PostgreSQL continúa siendo la fuente de verdad.
Partitioning no sustituye autorización.
Partitioning no sustituye tenant isolation.
No particionar todas las tablas automáticamente.
Audit Events son candidatos prioritarios.
Security Events son candidatos prioritarios.
Outbox Events son candidatos según volumen.
Core Identity permanece inicialmente sin particionar.
La estrategia preferida para históricos es temporal.
Retention y partitioning permanecen conceptualmente separados.
No deben existir particiones futuras inexistentes para periodos activos.
Las migraciones deben ser versionadas.
Backup/restore debe incluir particiones.
El rendimiento debe validarse mediante mediciones.
Las operaciones de retención deben ser observables.
La creación/eliminación de particiones debe ser segura y auditable.
No debe producirse pérdida silenciosa de datos durante archival/disposal.
Las claves e índices deben diseñarse según access patterns.
No se deben inventar tamaños o periodos de partición sin benchmarks.
La estrategia debe poder evolucionar con el crecimiento real de EVOXA.
34. Traceability
Blueprint

Relacionado con:

Data Architecture
Scalability
Performance
Auditability
Retention
Event Architecture
Security
Continuous Evolution
ESP

ESP-0001 — Identity Domain

Particularmente:

persistence;
audit;
sessions;
security events;
event/outbox architecture.
Implementation Stories

Principalmente:

IS-IDENTITY-012 — Audit & Security Events
IS-IDENTITY-013 — Domain Events & Outbox
IS-IDENTITY-015 — Identity Persistence
IS-IDENTITY-016 — Identity Testing & Quality
IS-IDENTITY-017 — Identity Runtime & Deployment
ADR relacionados
ADR-IDENTITY-012 — PostgreSQL Schema
ADR-IDENTITY-013 — Data Retention
ADR-IDENTITY-014 — Encryption & Secret Protection
ADR-IDENTITY-016 — Concurrency & Transaction Strategy
ADR-IDENTITY-017 — Database Migration Strategy
ADR-IDENTITY-029 — Disaster Recovery & Backup
35. Dependencias

Este ADR depende principalmente de:

ADR-IDENTITY-012
PostgreSQL Schema.

ADR-IDENTITY-013
Data Retention.

ADR-IDENTITY-016
Concurrency & Transaction Strategy.

ADR-IDENTITY-017
Database Migration Strategy.

Y se relaciona con:

ADR-IDENTITY-028
Observability & Operational Security.

ADR-IDENTITY-029
Disaster Recovery & Backup.

36. Decisiones pendientes

Quedan abiertas las siguientes decisiones:

Partition key
timestamp exacto;
otra dimensión cuando exista una necesidad específica.
Granularidad
diaria;
semanal;
mensual.
Tablas adicionales

Determinar si eventualmente deberán particionarse:

sessions;
refresh_tokens;
MFA challenges.
Index strategy

Validar mediante benchmarks.

Automated management

Determinar mecanismo de creación de futuras particiones.

Archive strategy

Definir almacenamiento y recuperación.

PostgreSQL version

Determinar versión objetivo y capacidades específicas soportadas.

Tenant + time

Determinar si determinadas tablas requieren índices compuestos:

tenant_id + timestamp

o estrategias alternativas.

37. Criterios de aceptación

ADR-IDENTITY-015 podrá pasar de Proposed a Accepted cuando exista:

 clasificación definitiva de tablas particionables;
 definición de partition keys;
 granularidad definida;
 estrategia de creación automática;
 estrategia de archivado;
 integración con retention;
 índices definidos;
 estrategia de constraints;
 estrategia de migrations;
 pruebas de partition pruning;
 pruebas de rendimiento;
 pruebas de crecimiento;
 pruebas de backup/restore;
 monitoreo;
 alertas;
 runbook operacional;
 estrategia de recuperación ante fallo de partición.
38. Próximo ADR

La siguiente decisión lógica de la cadena es:

ADR-IDENTITY-016 — Concurrency & Transaction Strategy

Este ADR deberá definir cómo EVOXA manejará:

transacciones;
concurrencia;
locking;
optimistic/pessimistic concurrency;
refresh-token rotation;
outbox atomicity;
race conditions;
idempotencia;
aislamiento transaccional;
deadlocks;
retry seguro;
operaciones concurrentes de Identity.

La secuencia queda así:

ADR-012  PostgreSQL Schema
   ↓
ADR-013  Data Retention
   ↓
ADR-014  Encryption & Secret Protection
   ↓
ADR-015  Database Partitioning
   ↓
ADR-016  Concurrency & Transaction Strategy
   ↓
ADR-017  Database Migration Strategy

Estado final de este ADR: Proposed.
