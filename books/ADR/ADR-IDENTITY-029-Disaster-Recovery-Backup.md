ADR-IDENTITY-029 — Disaster Recovery & Backup

Estado: Proposed
Versión: 1.0
Fecha: 2026-09-03
Dominio: Identity
Prioridad: Critical
Tipo: Architecture Decision Record

1. Contexto

Identity es un dominio crítico de EVOXA y concentra información cuya pérdida, corrupción o indisponibilidad puede impedir el acceso de usuarios y organizaciones a la plataforma.

El Blueprint establece que EVOXA debe ser:

Cloud-Ready
Secure by Design
Observable
Resilient
Continuously Evolvable

La arquitectura de Identity depende principalmente de:

PostgreSQL como fuente transaccional de verdad.
Redis para estado temporal, cache y operaciones de alta frecuencia.
Event Platform para eventos de dominio e integración.
KMS / Secret Manager para secretos y material criptográfico.
Transactional Outbox para garantizar consistencia entre estado transaccional y publicación de eventos.

Los objetivos globales establecidos son:

Availability ≥ 99.9%
RTO < 1 hora
RPO < 15 minutos

Por lo tanto, Disaster Recovery no puede limitarse a realizar copias de seguridad. Debe cubrir recuperación de datos, servicios, configuración, secretos, claves criptográficas, eventos y capacidad operacional.

2. Problema

Identity debe poder recuperarse frente a escenarios como:

pérdida o corrupción de PostgreSQL;
eliminación accidental de información;
fallo de infraestructura;
indisponibilidad de una región;
corrupción de datos;
fallo de Redis;
pérdida o retraso del Event Platform;
pérdida de Outbox;
compromiso o pérdida de claves;
indisponibilidad de KMS / Secret Manager;
errores de despliegue;
ransomware o comportamiento destructivo;
error humano;
desastre completo de infraestructura.

Además, una recuperación incorrecta podría producir problemas especialmente graves:

pérdida de usuarios;
pérdida de organizaciones;
pérdida de memberships;
sesiones inconsistentes;
refresh tokens reutilizables;
duplicación de eventos;
pérdida de eventos de seguridad;
pérdida de auditoría;
inconsistencias entre PostgreSQL y Event Platform;
imposibilidad de validar tokens existentes;
violaciones de tenant isolation.
3. Decisión

EVOXA Identity utilizará una estrategia de Backup + Disaster Recovery + Recovery Validation, donde los backups son solamente uno de los mecanismos de recuperación.

La estrategia tendrá cinco principios fundamentales:

PostgreSQL es el principal activo transaccional que debe poder restaurarse.
Los backups deben estar protegidos contra corrupción, eliminación y compromiso.
La recuperación debe ser reproducible y verificable.
Los eventos y Outbox deben poder reconstruirse o reprocesarse de manera controlada.
KMS / Secret Manager deben formar parte explícita del plan de recuperación.
4. Activos que deben formar parte del Disaster Recovery
4.1 PostgreSQL

PostgreSQL es la fuente primaria de verdad para Identity.

Debe existir capacidad para recuperar:

Users
Organizations
Memberships
Roles
Permissions
RolePermissions
MembershipRoles
Sessions
RefreshTokens
MFAFactors
MFAChallenges
RecoveryCodes
TrustedDevices
SecurityPolicies
AuditEvents
SecurityEvents
OutboxEvents
Idempotency records
cualquier otra entidad persistente introducida posteriormente.

La recuperación de PostgreSQL debe preservar:

integridad referencial;
constraints;
índices;
relaciones;
timestamps;
estados;
identificadores;
datos de auditoría;
información de tenant.
5. Backup de PostgreSQL

La estrategia deberá soportar dos mecanismos conceptuales.

5.1 Backup completo

Se realizarán backups completos periódicos de PostgreSQL.

Su finalidad principal será:

recuperación ante corrupción;
recuperación ante eliminación accidental;
recuperación ante desastre;
creación de puntos de restauración.
5.2 Point-in-Time Recovery

Cuando la infraestructura lo permita, PostgreSQL deberá soportar recuperación a un punto específico en el tiempo mediante mecanismos equivalentes a:

WAL;
continuous archiving;
incremental/differential backups;
snapshots consistentes.

El objetivo es poder recuperar el sistema a un instante anterior al incidente.

Esto es especialmente importante para escenarios como:

"Una migración eliminó accidentalmente información a las 14:32."

El sistema debería poder recuperar el estado anterior al incidente sin depender exclusivamente del último backup completo.

6. Objetivo RPO

El Recovery Point Objective objetivo será:

RPO < 15 minutos

Esto significa que la arquitectura debe diseñarse para que, ante un desastre, la pérdida máxima esperada de datos esté por debajo de 15 minutos.

El valor exacto alcanzable dependerá de:

frecuencia de backups;
replicación;
WAL/log shipping;
infraestructura;
región;
Event Platform;
mecanismos de almacenamiento.

El cumplimiento real deberá ser validado mediante pruebas.

7. Objetivo RTO

El Recovery Time Objective será:

RTO < 1 hora

La estrategia deberá permitir recuperar el servicio de Identity dentro de este objetivo.

El RTO debe medirse desde:

detección/clasificación del incidente → restauración → validación → disponibilidad operacional.

No se considerará suficiente decir que existe un backup.

Debe demostrarse que el sistema puede:

detectar el incidente;
activar el procedimiento;
recuperar infraestructura;
restaurar datos;
recuperar dependencias;
validar integridad;
habilitar tráfico;
verificar autenticación y autorización.
8. Redis

Redis no será considerado fuente primaria de verdad.

La pérdida de Redis no debe provocar pérdida permanente de información crítica de Identity.

Ante una recuperación:

Redis podrá reconstruirse;
caches podrán invalidarse;
información temporal podrá regenerarse;
rate limits podrán reiniciarse bajo política;
datos derivados deberán reconstruirse desde PostgreSQL u otras fuentes autorizadas.

No se debe diseñar el Disaster Recovery dependiendo de que un snapshot de Redis sea la única forma de recuperar información crítica.

9. Event Platform

La Event Platform deberá formar parte de la estrategia de recuperación.

Debe existir capacidad para manejar:

eventos pendientes;
eventos publicados;
eventos no publicados;
eventos duplicados;
mensajes en retry;
Dead Letter Queues;
consumer offsets;
replay controlado.

El diseño debe permitir reconstruir procesamiento cuando sea necesario.

La arquitectura mantiene la garantía:

At-Least-Once + Idempotent Consumers

Por lo tanto, durante una recuperación no se debe asumir que cada evento será procesado exactamente una sola vez.

10. Transactional Outbox durante Disaster Recovery

OutboxEvent es especialmente importante durante la recuperación.

Si PostgreSQL se restaura a un punto anterior, debe determinarse qué eventos:

estaban confirmados en la transacción;
estaban pendientes de publicación;
ya habían sido publicados;
fueron procesados por consumidores.

La recuperación deberá utilizar:

event IDs;
idempotency;
estado del Outbox;
offsets;
deduplicación;
replay controlado.

No se debe crear un segundo efecto de negocio simplemente porque un evento sea reprocesado.

11. Auditoría y Security Events

Los siguientes datos requieren tratamiento especial:

AuditEvents
SecurityEvents
eventos relacionados con autenticación;
eventos de autorización;
detección de refresh-token reuse;
cambios de MFA;
cambios de roles/permisos;
acciones administrativas.

La pérdida de estos datos puede afectar:

seguridad;
investigaciones;
compliance;
trazabilidad;
análisis de incidentes.

Por lo tanto, deberán incluirse explícitamente dentro de la estrategia de backup y recuperación.

La recuperación no podrá simplemente eliminar o sobrescribir el histórico de auditoría como parte de una operación normal.

12. KMS y Secret Manager

El Disaster Recovery debe considerar explícitamente la recuperación de:

claves de firma JWT;
claves de cifrado;
secretos utilizados para refresh tokens;
secretos MFA;
credenciales de infraestructura;
configuraciones sensibles.

Sin embargo:

Las claves y secretos no deben copiarse como texto plano dentro de los backups de PostgreSQL.

La recuperación deberá depender de mecanismos propios de:

KMS;
HSM;
Secret Manager;
mecanismos equivalentes.

Esto implica que el plan de DR debe incluir también:

metadata;
versiones;
políticas;
permisos;
referencias;
procedimiento de recuperación;
procedimientos de rotación/emergencia.
13. JWT Signing Keys

La pérdida de las claves utilizadas para firmar JWT puede dejar inutilizable la validación de tokens.

Por ello:

las claves deben administrarse fuera del código;
deben existir mecanismos de recuperación;
deben conservarse versiones necesarias;
kid debe permitir identificar la clave correspondiente;
la rotación debe ser compatible con recuperación;
las claves retiradas no deben eliminarse inmediatamente si todavía pueden ser necesarias para validar tokens existentes.

La estrategia concreta de backup/replicación de las claves dependerá del KMS/HSM seleccionado.

14. Separación de Backups

Los backups deben estar separados del entorno operativo principal.

La arquitectura debe evitar que un atacante que comprometa la infraestructura de producción pueda automáticamente:

eliminar todos los backups;
modificar todos los backups;
cifrar todos los backups;
obtener acceso administrativo a los backups.

Se recomienda una estrategia de aislamiento mediante:

cuentas/proyectos separados cuando corresponda;
permisos mínimos;
almacenamiento protegido;
retención;
versiones;
mecanismos de eliminación protegida;
controles administrativos separados.

La implementación exacta queda pendiente del proveedor cloud seleccionado.

15. Backup Immutability

Los backups críticos deberán contar, cuando la infraestructura lo permita, con mecanismos equivalentes a:

WORM;
Object Lock;
versioning;
retention lock;
snapshots protegidos;
eliminación diferida.

El objetivo es proteger contra:

ransomware;
borrado accidental;
sabotaje;
compromiso de credenciales.
16. Cifrado

Los backups deberán estar cifrados:

En tránsito

Cuando sean transferidos entre:

producción;
almacenamiento de backup;
regiones;
sistemas de recuperación.
En reposo

Utilizando mecanismos de cifrado administrados por la plataforma y/o KMS.

Las claves utilizadas para proteger los backups deberán tener lifecycle y recuperación independientes de los datos que protegen.

17. Separación geográfica

La arquitectura deberá permitir almacenar copias fuera de la infraestructura primaria.

La estrategia podrá evolucionar hacia:

Primary Region → Secondary Region / Backup Region

La elección de regiones exactas queda pendiente de:

proveedor cloud;
requisitos regulatorios;
residencia de datos;
costos;
latencia;
arquitectura multi-region.

No se establece en este ADR una región concreta.

18. Recuperación por niveles

El Disaster Recovery se dividirá conceptualmente en niveles.

Nivel 1 — Recuperación de datos

Restaurar:

PostgreSQL;
backups;
WAL/PITR;
integridad.
Nivel 2 — Recuperación de infraestructura

Recuperar:

containers;
configuración;
networking;
secrets references;
service dependencies.
Nivel 3 — Recuperación de Identity

Validar:

login;
JWT;
refresh;
sessions;
authorization;
MFA;
policies.
Nivel 4 — Recuperación de eventos

Validar:

Outbox;
Event Platform;
consumers;
retries;
DLQ;
replay.
Nivel 5 — Recuperación operacional

Validar:

monitoring;
alerts;
audit;
security events;
dashboards;
runbooks.
19. Recovery Order

El orden conceptual de recuperación será:

Infrastructure
      ↓
KMS / Secret Manager
      ↓
PostgreSQL
      ↓
Identity API
      ↓
Redis
      ↓
Event Platform / Outbox
      ↓
Consumers
      ↓
Observability
      ↓
Traffic

Este orden es conceptual y podrá modificarse dependiendo de la infraestructura final.

20. Validaciones posteriores a Restore

Una restauración no será considerada exitosa únicamente porque PostgreSQL haya arrancado.

Debe ejecutarse una batería de validaciones.

Data integrity
foreign keys;
constraints;
índices;
registros críticos;
counts;
estados;
relaciones.
Identity
login;
logout;
refresh;
session validation;
authorization;
tenant isolation.
MFA
MFA challenge;
factor validation;
recovery;
step-up.
Security
roles;
permissions;
policies;
audit;
security events.
Events
Outbox;
publishing;
consumer processing;
idempotency;
DLQ.
21. Tenant Isolation durante Recovery

La recuperación debe preservar estrictamente:

Tenant isolation

No se permitirá que una restauración produzca:

memberships cruzados;
roles de otra organización;
permisos cruzados;
datos de una organización visibles para otra;
eventos asociados al tenant incorrecto.

La validación de tenant isolation será una prueba obligatoria del Disaster Recovery.

22. Sessions y Refresh Tokens después de Disaster Recovery

Sessions y Refresh Tokens requieren especial cuidado.

Una recuperación puede dejar el sistema en un estado donde existan tokens que:

ya habían sido utilizados;
estaban revocados;
estaban activos;
fueron comprometidos antes del desastre.

Por lo tanto, la recuperación debe preservar los estados necesarios para mantener:

revocation;
token rotation;
token family;
reuse detection.

No se permitirá que un restore convierta automáticamente un refresh token previamente revocado en válido.

23. Recovery y Replay

Después de recuperar PostgreSQL y Event Platform podría ser necesario ejecutar replay.

El replay deberá ser:

explícito;
autorizado;
auditable;
controlado;
idempotente;
limitado por scope;
reversible operacionalmente cuando sea posible.

El replay no debe modificar artificialmente el histórico de eventos ni crear duplicación de business outcomes.

24. Backup Retention

La política de retención deberá distinguir entre:

backups operacionales;
backups diarios;
backups semanales;
backups mensuales;
backups necesarios para compliance;
snapshots;
WAL/archive;
eventos.

Los valores exactos de retención deberán alinearse con:

ADR-IDENTITY-013 — Data Retention;
requisitos regulatorios;
política global de EVOXA;
costos;
RPO/RTO.

No se fija aquí una duración exacta porque el Blueprint no establece un valor único obligatorio para todos los tipos de backup.

25. Testing del Disaster Recovery

El Disaster Recovery debe probarse periódicamente.

Las pruebas deben incluir como mínimo:

Backup restoration
Backup
  ↓
Restore
  ↓
Integrity Check
  ↓
Application Validation
Point-in-Time Recovery

Simular:

Normal Operation
      ↓
Data Corruption
      ↓
Incident Detection
      ↓
Select Recovery Point
      ↓
PITR
      ↓
Validation
PostgreSQL failure

Simular pérdida total de la instancia primaria.

Redis failure

Verificar que Identity continúa o puede recuperarse sin pérdida permanente de datos.

Event Platform failure

Verificar:

Outbox accumulation;
retry;
recovery;
publishing;
idempotency.
KMS / Secret Manager failure

Validar comportamiento fail closed.

Regional disaster

Cuando la arquitectura lo soporte, realizar prueba de recuperación en región secundaria.

26. Recovery Drills

Las pruebas no deben limitarse a documentación.

Debe existir un Recovery Drill ejecutable.

El drill deberá medir:

RTO real;
RPO real;
tiempo de detección;
tiempo de restauración;
tiempo de validación;
cantidad de datos perdidos;
cantidad de eventos recuperados;
errores;
acciones manuales necesarias.

Los resultados deberán generar acciones correctivas.

27. Runbooks

Identity deberá disponer de runbooks para escenarios críticos:

PostgreSQL unavailable;
PostgreSQL corruption;
accidental data deletion;
ransomware;
Redis unavailable;
Event Platform unavailable;
Outbox backlog;
KMS unavailable;
Secret Manager unavailable;
JWT signing key compromise;
refresh-token compromise;
regional outage;
failed migration;
corrupted deployment;
complete Identity outage.

Los runbooks deben ser versionados junto con la documentación operacional correspondiente.

28. Disaster Recovery y Deployments

Una nueva versión no debe destruir la capacidad de recovery.

Antes de migrations críticas:

validar backup;
validar restore;
comprobar compatibilidad;
revisar rollback strategy;
verificar migraciones.

Para cambios destructivos:

Backup → Migration → Validation

No deberá ejecutarse una operación destructiva irreversible sin un recovery point apropiado.

29. Disaster Recovery y AI Agents

Los AI Agents no podrán:

acceder directamente a backups;
recuperar secretos;
administrar KMS;
modificar retention;
eliminar backups;
ejecutar restores destructivos sin autorización;
alterar políticas de recuperación;
saltarse controles de seguridad.

Un AI Agent podrá, bajo herramientas autorizadas:

consultar estado;
generar diagnósticos;
analizar métricas;
recomendar acciones;
iniciar procedimientos aprobados.

Las operaciones destructivas o de recuperación crítica deberán permanecer bajo controles explícitos.

30. Consecuencias positivas

Esta decisión proporciona:

recuperación ante pérdida de infraestructura;
protección frente a errores humanos;
protección frente a corrupción;
recuperación temporal mediante PITR;
alineamiento con RPO <15 min;
alineamiento con RTO <1h;
recuperación controlada de eventos;
protección de auditoría;
recuperación de Identity completa;
mayor resiliencia;
capacidad de realizar recovery drills.
31. Consecuencias negativas

Introduce:

mayor complejidad operacional;
costo adicional de almacenamiento;
costo de replicación;
complejidad KMS/Secret Manager;
necesidad de recovery drills;
necesidad de mantener runbooks;
necesidad de probar periódicamente los backups;
complejidad adicional en eventos y replay.

La resiliencia se considera prioritaria frente a minimizar únicamente el costo de infraestructura.

32. Alternativas consideradas
A. Solo backups periódicos

Rechazada.

No garantiza por sí sola:

RPO <15 minutos;
recuperación rápida;
recuperación de eventos;
recuperación de claves;
recuperación operacional.
B. Solo PostgreSQL replication

Rechazada como estrategia completa.

Replication mejora disponibilidad, pero no sustituye:

backups;
PITR;
protección contra corrupción;
protección contra errores humanos;
protección contra ransomware.
C. Active-Active Multi-Region desde el inicio

No adoptada como requisito inicial.

Puede proporcionar mayor resiliencia, pero introduce considerable complejidad y costos.

Se mantiene como posible evolución futura.

D. Backup + PITR + Recovery Validation

Adoptada.

Proporciona un equilibrio adecuado entre:

resiliencia;
complejidad;
costo;
capacidad operacional.
33. No negociables
RPO objetivo <15 minutos.
RTO objetivo <1 hora.
PostgreSQL debe poder restaurarse.
Los backups críticos deben estar protegidos.
Los backups deben cifrarse.
Los backups deben estar separados del entorno productivo.
Debe existir protección contra eliminación/corrupción.
Redis no es fuente primaria de verdad.
Outbox debe formar parte del recovery.
Event replay debe ser idempotente.
AuditEvents y SecurityEvents deben formar parte del recovery.
KMS / Secret Manager deben estar contemplados explícitamente.
Recovery debe preservar tenant isolation.
Recovery no puede reactivar refresh tokens revocados.
Deben existir restore tests.
Deben existir recovery drills.
Las operaciones críticas deben quedar auditadas.
AI Agents no pueden saltarse controles de Disaster Recovery.
Failures críticos deben comportarse fail-closed.
El Disaster Recovery debe evolucionar junto con la arquitectura.
34. Traceability
Referencia	Relación
Blueprint — Architecture Map	Disaster Recovery dentro del lifecycle operacional
Blueprint — Platform Overview	Cloud-ready / resilient platform
ESP-0001 Identity	Persistencia, seguridad y operación de Identity
ADR-IDENTITY-006	PostgreSQL como source of truth
ADR-IDENTITY-007	Outbox + Event Platform
ADR-IDENTITY-008	KMS, claves y secret management
ADR-IDENTITY-011	Runtime, deployment, health y RTO/RPO
ADR-IDENTITY-012	PostgreSQL schema
ADR-IDENTITY-013	Data retention
ADR-IDENTITY-014	Encryption & secret protection
ADR-IDENTITY-017	Migrations
ADR-IDENTITY-018	Shared Event Broker
ADR-IDENTITY-019	Event partitioning/routing
ADR-IDENTITY-020	Schema compatibility
ADR-IDENTITY-021	Controlled replay
ADR-IDENTITY-022	Consumer idempotency
ADR-IDENTITY-028	Observability & Operational Security
ADR-IDENTITY-029	Disaster Recovery & Backup
35. Dependencias

Este ADR depende especialmente de:

PostgreSQL architecture;
backup infrastructure;
Event Platform;
Transactional Outbox;
KMS;
Secret Manager;
deployment platform;
observability platform;
retention policies.
36. Decisiones pendientes

Quedan deliberadamente abiertas para ADRs posteriores:

proveedor de backup;
proveedor cloud;
estrategia exacta de PostgreSQL PITR;
frecuencia exacta de full/incremental backups;
WAL retention;
backup retention;
número de regiones;
estrategia multi-region;
backup storage;
immutable backup mechanism;
KMS disaster recovery;
Secret Manager disaster recovery;
procedimiento exacto de key recovery;
automatización de restore;
recovery orchestration;
recovery validation framework;
frecuencia de recovery drills;
criterios exactos de RPO/RTO por componente;
estrategia de Event Platform recovery;
procedimiento de Outbox replay;
estrategia de database failover;
criterios para activar región secundaria.

Estas decisiones deberán quedar documentadas antes de considerar cerrado el diseño operacional de Identity.

37. Acceptance Criteria

El ADR se considerará correctamente implementado cuando:

 exista una estrategia formal de backup;
 PostgreSQL pueda restaurarse;
 exista PITR o mecanismo equivalente cuando la infraestructura lo permita;
 RPO <15 min haya sido validado;
 RTO <1h haya sido validado;
 backups estén cifrados;
 backups estén aislados de producción;
 exista protección contra eliminación accidental;
 Redis pueda reconstruirse;
 Outbox pueda recuperarse;
 Event replay sea idempotente;
 AuditEvents sean recuperables;
 SecurityEvents sean recuperables;
 KMS/Secret Manager estén contemplados en DR;
 tenant isolation sea validado después del restore;
 refresh-token revocation sea preservado;
 existan restore tests;
 existan recovery drills;
 existan runbooks;
 los procedimientos estén auditados;
 los escenarios críticos hayan sido probados.
38. Siguiente ADR

El siguiente paso lógico de la serie es:

ADR-IDENTITY-030 — Disaster Recovery Testing & Recovery Validation

Este ADR debería profundizar específicamente en cómo demostrar técnicamente que el RPO <15 minutos y RTO <1 hora realmente se cumplen, incluyendo restore drills, PITR tests, validación de PostgreSQL, Outbox/Event Platform, tenant isolation, sesiones, refresh tokens, MFA y criterios de aprobación del recovery.
