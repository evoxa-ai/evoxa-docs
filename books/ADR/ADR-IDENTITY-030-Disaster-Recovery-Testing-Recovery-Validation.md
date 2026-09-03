ADR-IDENTITY-030 — Disaster Recovery Testing & Recovery Validation

Estado: Proposed
Versión: 1.0
Fecha: 2026-09-03
Dominio: Identity
Prioridad: Critical
Tipo: Architecture Decision Record

1. Contexto

ADR-IDENTITY-029 establece la estrategia de Disaster Recovery & Backup para Identity, incluyendo:

PostgreSQL como fuente de verdad;
backups;
Point-in-Time Recovery;
RPO objetivo <15 minutos;
RTO objetivo <1 hora;
recuperación de Redis;
recuperación de Outbox;
recuperación de Event Platform;
protección y recuperación de KMS / Secret Manager;
recuperación de AuditEvents y SecurityEvents;
preservación de tenant isolation;
recuperación correcta de Sessions y Refresh Tokens.

Sin embargo:

La existencia de backups y procedimientos documentados no demuestra que el sistema sea recuperable.

La capacidad real de Disaster Recovery debe ser demostrada mediante pruebas reproducibles, medibles y auditables.

Este ADR define cómo EVOXA deberá comprobar que la estrategia de recuperación funciona realmente.

2. Problema

Un Disaster Recovery puede fallar incluso cuando aparentemente existen todos los componentes necesarios.

Por ejemplo:

un backup puede estar corrupto;
una restauración puede romper foreign keys;
el backup puede ser demasiado antiguo;
el PITR puede no alcanzar el RPO requerido;
las claves KMS pueden no estar disponibles;
los secrets pueden no poder recuperarse;
las sesiones pueden quedar inconsistentes;
los refresh tokens revocados podrían volver a aparecer como activos;
el Outbox podría duplicar eventos;
los eventos podrían procesarse nuevamente;
podría perderse tenant isolation;
el servicio podría restaurarse pero permanecer incapaz de autenticar usuarios;
el restore podría superar el RTO.

Por esta razón, Disaster Recovery debe ser tratado como una capacidad verificable, no solamente como infraestructura.

3. Decisión

EVOXA Identity adoptará un modelo formal de:

Disaster Recovery Testing & Recovery Validation

Cada capacidad crítica de recuperación deberá:

tener un escenario de prueba;
tener condiciones iniciales conocidas;
ejecutar un procedimiento de recuperación;
medir RPO;
medir RTO;
validar integridad de datos;
validar funcionalidad;
validar seguridad;
validar eventos;
registrar resultados;
generar evidencia;
producir acciones correctivas cuando falle.
4. Principio de Recovery Validation

La regla fundamental será:

Backup válido ≠ Recovery validado.

Y:

Servicio iniciado ≠ Recovery exitoso.

Un recovery solo será considerado exitoso cuando haya superado las validaciones técnicas y funcionales definidas.

5. Objetivos de las pruebas

Las pruebas deberán demostrar que Identity puede recuperar:

Datos
usuarios;
organizaciones;
memberships;
roles;
permisos;
sesiones;
refresh tokens;
MFA;
políticas;
auditoría;
security events;
outbox.
Funcionalidad
authentication;
authorization;
MFA;
session management;
refresh;
logout;
tenant isolation.
Integraciones
Event Platform;
Redis;
KMS;
Secret Manager;
observability.
Objetivos operacionales
RPO <15 minutos
RTO <1 hora
6. Tipos de Recovery Tests

Se establecen los siguientes niveles.

6.1 Backup Restore Test

Validar que un backup pueda restaurarse.

Flujo:

Backup
   ↓
Restore
   ↓
Integrity Validation
   ↓
Application Validation

Debe comprobarse:

backup legible;
restore exitoso;
schema correcto;
constraints correctas;
datos disponibles;
aplicación funcional.
7. Point-in-Time Recovery Test

Debe probarse la capacidad de volver a un punto temporal anterior a un incidente.

Escenario:

T0 ─────── T1 ─────── T2 ─────── T3
           │
           └── Data corruption

La recuperación debe poder regresar a un punto válido anterior a T2.

La prueba debe medir:

recovery point seleccionado;
recovery point alcanzado;
diferencia temporal;
datos recuperados;
datos perdidos;
tiempo total de recuperación.
8. RPO Validation

El RPO no debe ser asumido.

Debe medirse.

Ejemplo conceptual
Último dato válido:
14:00

Incidente:
14:12

Recovery point:
14:10

RPO:

2 minutos

Resultado:

PASS

Si:

Último dato recuperable:
13:45

Incidente:
14:12

RPO:

27 minutos

Resultado:

FAIL

porque supera el objetivo de 15 minutos.

9. RTO Validation

El RTO debe medirse desde el inicio formal del recovery hasta la restauración operacional.

Ejemplo:

Recovery Start
     ↓
Infrastructure Ready
     ↓
Database Restored
     ↓
Identity API Ready
     ↓
Dependencies Ready
     ↓
Functional Validation
     ↓
Traffic Restored

El tiempo total debe ser:

< 1 hora

La medición deberá registrar timestamps para cada etapa.

10. Data Integrity Validation

Después de cualquier restore deberán realizarse pruebas de integridad.

Referential Integrity

Validar:

foreign keys;
relaciones;
referencias;
constraints.
Entity Integrity

Validar:

IDs;
estados;
timestamps;
campos obligatorios;
uniqueness.
Domain Integrity

Validar reglas específicas de Identity.

Por ejemplo:

Membership → Organization
Membership → User
Session → User
Session → Organization
RefreshToken → Session
Role → Organization
RolePermission → Role
11. Identity Functional Validation

Después del restore deben ejecutarse pruebas funcionales.

Authentication
login exitoso;
login inválido;
account status;
credential validation.
Authorization
permission ALLOW;
permission DENY;
tenant isolation;
resource ownership;
policy enforcement.
Sessions
crear sesión;
consultar sesión;
revocar sesión;
revoke-all.
Refresh
refresh válido;
refresh expirado;
refresh revocado;
refresh token reuse;
rotation.
12. MFA Recovery Validation

El recovery debe comprobar que MFA conserva su estado.

Validar:

MFA factors;
factor status;
enrollment;
challenge;
recovery codes;
trusted devices;
step-up.

Especialmente importante:

Un factor MFA previamente revocado no puede aparecer como activo después del restore.

Y:

Un recovery no puede deshabilitar MFA como mecanismo de recuperación.

13. Refresh Token Validation

Los Refresh Tokens requieren pruebas específicas.

Se debe comprobar:

Active token

Permanece válido si correspondía al recovery point.

Used token

Continúa marcado como USED.

Revoked token

Continúa REVOKED.

Expired token

Continúa EXPIRED cuando corresponda.

Reuse detection

Un token previamente consumido no puede volver a utilizarse simplemente porque se restauró una copia anterior de la base de datos.

Esta prueba es crítica porque afecta directamente la seguridad de Identity.

14. Session Validation

Después de recovery deberán validarse:

sessions activas;
sessions expiradas;
sessions revocadas;
timestamps;
authentication method;
MFA state;
tenant association.

Debe comprobarse que una sesión revocada no vuelva a aparecer como activa.

15. Tenant Isolation Validation

Esta será una prueba Critical Security Test.

Después del restore deberá verificarse que:

Tenant A
   ↓
Users A
Memberships A
Roles A
Permissions A
Sessions A

no puedan acceder a:

Tenant B

La prueba debe cubrir:

API;
database queries;
authorization;
audit;
security events;
events;
cache.

Cualquier cross-tenant access será:

FAIL crítico

y bloqueará la aprobación del recovery.

16. Outbox Recovery Validation

Después de restaurar PostgreSQL debe validarse:

Outbox records;
event IDs;
event status;
timestamps;
tenant;
actor;
resource;
payload;
metadata.

Debe determinarse qué eventos:

fueron publicados;
quedaron pendientes;
requieren retry;
deben entrar en replay.
17. Event Replay Validation

Los eventos recuperados podrán ser reprocesados cuando sea necesario.

La prueba debe verificar:

Event
 ↓
Consumer
 ↓
Processing
 ↓
Duplicate Event
 ↓
Idempotency
 ↓
No duplicate business outcome

La recuperación no debe producir:

doble creación;
doble actualización;
doble envío;
doble efecto financiero;
doble acción de negocio.
18. Redis Recovery Test

Debe probarse la pérdida completa de Redis.

El objetivo es demostrar que:

Identity puede recuperar caches;
los datos críticos no desaparecen;
las sesiones críticas no dependen exclusivamente de Redis;
authorization cache puede reconstruirse;
rate limiting puede reiniciarse bajo política.

La pérdida de Redis no debe provocar pérdida permanente de información transaccional.

19. KMS / Secret Manager Recovery Test

Debe comprobarse que Identity pueda recuperar o volver a acceder a:

JWT signing keys;
refresh token protection secrets;
MFA protection secrets;
database credentials;
service credentials;
otros secretos críticos.

La prueba debe verificar:

Identity Startup
      ↓
Secret/Key Access
      ↓
Crypto Initialization
      ↓
JWT Validation
      ↓
Authentication

Si una dependencia criptográfica crítica no está disponible:

Identity debe fallar de manera segura.

Nunca deberá hacer fallback hacia:

claves hardcoded;
secretos temporales no autorizados;
cifrado débil;
bypass de autenticación.
20. Observability Recovery Validation

Después del recovery debe comprobarse que vuelvan a funcionar:

logs;
metrics;
traces;
audit;
security events;
correlation IDs;
alerts.

Una recuperación sin observabilidad operacional completa no se considerará totalmente validada.

21. Disaster Scenarios

Se establece un catálogo inicial de escenarios.

Scenario	Criticality
PostgreSQL failure	Critical
PostgreSQL corruption	Critical
Accidental deletion	Critical
Failed migration	Critical
Redis failure	High
Event Platform failure	High
Outbox backlog	High
KMS unavailable	Critical
Secret Manager unavailable	Critical
JWT key compromise	Critical
Refresh token compromise	Critical
Regional outage	Critical
Full Identity outage	Critical
Deployment corruption	High
Ransomware/destructive incident	Critical
22. Chaos / Failure Injection

Cuando la plataforma esté suficientemente madura, se podrán realizar pruebas controladas de failure injection.

Ejemplos:

detener PostgreSQL;
bloquear Redis;
bloquear Event Platform;
simular KMS unavailable;
simular secret unavailable;
introducir latencia;
provocar consumer failure;
generar Outbox backlog.

Estas pruebas deben ejecutarse únicamente en ambientes controlados.

Nunca deben comprometer producción sin un procedimiento explícitamente aprobado.

23. Recovery Test Environment

Las pruebas deberán ejecutarse preferentemente en:

QA;
Staging;
Recovery Environment;
DR Environment.

Cuando sea necesario validar características reales de infraestructura, podrán realizarse pruebas controladas adicionales.

La prueba debe utilizar datos:

sintéticos;
anonimizados;
o específicamente autorizados.

No se deben exponer datos productivos innecesariamente.

24. Recovery Test Dataset

Cada recovery test debería tener un dataset conocido.

Por ejemplo:

Users:             1,000
Organizations:       100
Memberships:       2,500
Roles:               200
Permissions:         500
Sessions:          1,500
Refresh Tokens:    3,000
MFA Factors:         800
Audit Events:     50,000
Security Events:   5,000
Outbox Events:    10,000

Los valores anteriores son ejemplos de prueba y no representan cantidades normativas del Blueprint.

El dataset deberá contener casos especiales:

usuarios suspendidos;
memberships removed;
roles revoked;
sessions revoked;
refresh tokens used;
refresh tokens revoked;
MFA factors revoked;
múltiples tenants;
eventos pendientes.
25. Recovery Validation Checklist

Cada ejecución deberá validar al menos:

Infrastructure
 containers;
 networking;
 configuration;
 secrets;
 KMS.
Database
 restore;
 schema;
 constraints;
 indexes;
 integrity;
 data consistency.
Identity
 authentication;
 authorization;
 tenant isolation;
 sessions;
 refresh;
 MFA;
 policies.
Events
 Outbox;
 Event Platform;
 retry;
 DLQ;
 replay;
 idempotency.
Observability
 logs;
 metrics;
 traces;
 audit;
 security events;
 alerts.
26. Recovery Test Result

Cada prueba deberá producir un resultado estructurado.

Conceptualmente:

Recovery Test
├── Test ID
├── Scenario
├── Environment
├── Start Time
├── Incident Time
├── Recovery Point
├── RPO
├── RTO
├── Data Integrity
├── Security Validation
├── Tenant Isolation
├── Identity Validation
├── Event Validation
├── Observability Validation
├── Result
└── Corrective Actions

Resultado:

PASS
FAIL
PARTIAL
27. Critical Failure Conditions

La prueba será automáticamente FAIL si ocurre cualquiera de los siguientes:

cross-tenant access;
autenticación bypass;
autorización fail-open;
MFA bypass;
refresh token revocado vuelve a ser válido;
pérdida no controlada de audit/security events;
corrupción de datos críticos;
pérdida de integridad referencial;
exposición de secretos;
recuperación fuera del RTO objetivo;
recuperación fuera del RPO objetivo;
duplicación no controlada de business outcomes;
incapacidad para validar JWT por pérdida no planificada de claves.
28. Recovery Evidence

Cada prueba deberá generar evidencia suficiente para demostrar el resultado.

La evidencia puede incluir:

timestamps;
logs;
metrics;
traces;
database validation results;
test reports;
screenshots cuando corresponda;
recovery commands;
deployment records;
event processing results.

Nunca deberá incluir:

passwords;
refresh tokens;
access tokens;
MFA secrets;
private keys;
recovery codes;
otros secretos.
29. Recovery Regression Testing

Cada cambio significativo en Identity deberá evaluar si afecta Disaster Recovery.

Especialmente:

schema migrations;
nuevas entidades;
cambios en sessions;
refresh tokens;
MFA;
roles;
permissions;
policies;
Outbox;
events;
KMS;
Secret Manager;
deployment architecture.

Una nueva feature que agregue un activo persistente deberá actualizar:

backup strategy;
restore validation;
test dataset;
recovery checklist;
runbook cuando corresponda.
30. Frequency

La frecuencia exacta queda pendiente de la estrategia operacional final.

Como principio:

backups → ejecución automatizada;
backup restore verification → periódica;
PITR validation → periódica;
recovery drill → periódica;
regional recovery → periódica cuando exista multi-region;
critical changes → validation específica;
security incident → recovery validation posterior cuando corresponda.

Los valores exactos deberán definirse en la política operacional de EVOXA.

31. Recovery Drill

Se establece un proceso formal:

Plan
 ↓
Prepare
 ↓
Inject Failure
 ↓
Detect
 ↓
Recover
 ↓
Validate
 ↓
Measure
 ↓
Document
 ↓
Correct
 ↓
Retest

El objetivo no es solamente demostrar que funciona.

También debe descubrir:

pasos manuales innecesarios;
dependencias ocultas;
tiempos excesivos;
falta de permisos;
problemas de documentación;
fallos de automatización.
32. Post-Recovery Review

Cada fallo o recovery significativo deberá producir un análisis posterior.

Debe responder:

¿Qué ocurrió?
¿Qué recovery path se utilizó?
¿Cuál fue el RPO real?
¿Cuál fue el RTO real?
¿Qué datos se perdieron?
¿Qué datos se recuperaron?
¿Hubo inconsistencias?
¿Hubo problemas de seguridad?
¿Hubo problemas de tenant isolation?
¿Qué debe cambiar?
¿Debe actualizarse algún ADR?
¿Debe agregarse una nueva prueba?
33. Automatización

La mayor cantidad posible del proceso deberá automatizarse.

Especialmente:

restore;
integrity checks;
schema validation;
smoke tests;
authentication tests;
authorization tests;
tenant isolation tests;
refresh tests;
MFA tests;
event tests;
idempotency tests;
observability validation.

El objetivo es reducir dependencia de intervención manual y aumentar reproducibilidad.

34. AI Agents

Los AI Agents podrán utilizar resultados de recovery tests para:

detectar patrones;
analizar fallos;
comparar RPO/RTO históricos;
identificar regresiones;
generar recomendaciones;
priorizar acciones correctivas.

Pero no podrán:

modificar evidencias;
ocultar fallos;
marcar automáticamente un recovery como PASS sin reglas;
desactivar controles;
alterar RPO/RTO;
ignorar tenant isolation;
aprobar recovery de manera autónoma cuando exista un Critical Failure.

La decisión final debe permanecer gobernada por reglas de plataforma y procesos humanos autorizados.

35. Consecuencias positivas

Esta decisión proporciona:

evidencia real de recoverability;
medición objetiva de RPO/RTO;
detección temprana de fallos;
validación de seguridad;
validación de tenant isolation;
validación de MFA;
validación de refresh token lifecycle;
validación de Outbox/Event Platform;
reducción del riesgo operacional;
mayor confianza en deployments;
mejora continua del Disaster Recovery.
36. Consecuencias negativas

Introduce:

costo de infraestructura para ambientes de prueba;
tiempo operacional;
complejidad de automatización;
necesidad de mantener datasets de prueba;
necesidad de ejecutar recovery drills;
esfuerzo de documentación;
necesidad de mantener runbooks actualizados.

Estas consecuencias se consideran aceptables debido a la criticidad de Identity.

37. Alternativas consideradas
A. Confiar solamente en backups

Rechazada.

No demuestra recoverability.

B. Ejecutar restore manual ocasionalmente

Rechazada como estrategia principal.

Puede detectar algunos problemas, pero no garantiza:

reproducibilidad;
cobertura;
medición sistemática;
validación funcional;
validación de seguridad.
C. Automated Recovery Testing

Adoptada.

Permite:

repetibilidad;
métricas;
regresión;
integración con CI/CD;
evidencia;
detección temprana.
38. No negociables
Backup restore debe probarse.
PITR debe probarse cuando esté implementado.
RPO debe medirse.
RTO debe medirse.
Recovery no se considera exitoso solo porque la base de datos arranque.
Debe validarse Authentication.
Debe validarse Authorization.
Debe validarse MFA.
Debe validarse Sessions.
Debe validarse Refresh Token lifecycle.
Debe validarse Tenant Isolation.
Debe validarse Outbox.
Debe validarse Event replay/idempotency.
Debe validarse KMS/Secret Manager.
Debe validarse Audit/Security Events.
Debe validarse Observability.
Los secretos nunca deben aparecer en la evidencia.
Un cross-tenant access es Critical Failure.
Un MFA bypass es Critical Failure.
Un refresh token revocado que vuelva a ser válido es Critical Failure.
Los recovery drills deben producir evidencia.
Los fallos deben generar acciones correctivas.
Los cambios críticos deben generar regression testing.
AI Agents no pueden falsificar ni aprobar recovery saltándose las reglas.
39. Traceability
Referencia	Relación
Blueprint — Architecture Map	Lifecycle operacional
ESP-0001 Identity	Recovery de Identity
ADR-IDENTITY-006	PostgreSQL Source of Truth
ADR-IDENTITY-007	Outbox / Events
ADR-IDENTITY-008	KMS / Cryptography
ADR-IDENTITY-010	Testing & Quality
ADR-IDENTITY-011	Runtime / RTO / RPO
ADR-IDENTITY-013	Data Retention
ADR-IDENTITY-014	Encryption
ADR-IDENTITY-017	Migrations
ADR-IDENTITY-021	Controlled Replay
ADR-IDENTITY-022	Consumer Idempotency
ADR-IDENTITY-028	Observability
ADR-IDENTITY-029	Disaster Recovery & Backup
ADR-IDENTITY-030	DR Testing & Recovery Validation
40. Dependencias

Este ADR depende de:

ADR-IDENTITY-029;
PostgreSQL backup architecture;
PITR;
Event Platform;
Outbox;
KMS;
Secret Manager;
Observability;
CI/CD;
Test Automation;
Security Testing.
41. Decisiones pendientes

Quedan para fases posteriores:

frecuencia exacta de restore tests;
frecuencia de PITR tests;
frecuencia de full recovery drills;
frecuencia de regional recovery;
tooling de recovery automation;
framework de validation;
formato definitivo de recovery reports;
almacenamiento de evidencia;
integración con CI/CD;
thresholds exactos de RPO por componente;
thresholds exactos de RTO por componente;
estrategia de synthetic datasets;
automatización de tenant isolation testing;
automatización de Event replay testing;
disaster recovery scorecard;
ownership operacional de cada recovery procedure.
42. Acceptance Criteria

El ADR se considerará implementado cuando:

 exista un procedimiento automatizable de restore;
 exista Backup Restore Test;
 exista PITR Test cuando corresponda;
 RPO sea medido;
 RTO sea medido;
 exista Data Integrity Validation;
 exista Authentication Validation;
 exista Authorization Validation;
 exista Tenant Isolation Validation;
 exista Session Validation;
 exista Refresh Token Validation;
 exista MFA Validation;
 exista Outbox Validation;
 exista Event Replay Validation;
 exista Idempotency Validation;
 exista KMS/Secret Manager Validation;
 exista Observability Validation;
 existan Critical Failure Conditions;
 existan Recovery Reports;
 existan Recovery Drills;
 existan Post-Recovery Reviews;
 los fallos produzcan acciones correctivas;
 exista Regression Testing para cambios críticos;
 RPO <15 minutos haya sido demostrado;
 RTO <1 hora haya sido demostrado.
43. Siguiente ADR

La secuencia natural ahora es:

ADR-IDENTITY-031 — Identity High Availability & Failover Strategy

Ahí podemos definir cómo Identity permanece disponible durante fallos, diferenciando claramente High Availability, Failover y Disaster Recovery, incluyendo PostgreSQL failover, Redis, Event Platform, API instances, health checks, load balancing, graceful degradation y comportamiento de Authentication/Authorization durante una falla.
