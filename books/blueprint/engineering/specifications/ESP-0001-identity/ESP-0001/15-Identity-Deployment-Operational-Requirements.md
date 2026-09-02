ESP-0001 / 15 — Identity Deployment & Operational Requirements

Documento: Engineering Specification — ESP-0001 Identity
Capítulo: 15 — Identity Deployment & Operational Requirements
Estado: Draft
Versión: 1.0.0
Dominio: Identity
Tipo: Engineering Specification

1. Propósito

Este capítulo define los requisitos necesarios para desplegar, operar, monitorizar, recuperar y evolucionar el dominio Identity de EVOXA.

Identity es una capacidad crítica de plataforma. Por ello, su operación debe garantizar:

Disponibilidad.
Seguridad.
Observabilidad.
Recuperación.
Escalabilidad.
Integridad de datos.
Auditabilidad.
Despliegues controlados.
Rollback.
Disaster Recovery.
Gestión segura de configuración y secretos.

El Blueprint establece que EVOXA debe operar mediante prácticas de SRE, DevOps, Platform Operations, Security Operations, AI Operations y FinOps, con actividades automatizadas, observables, repetibles, documentadas y medibles.

2. Alcance

Este documento cubre:

Identity
│
├── Runtime
├── Database
├── Cache
├── Event Platform
├── Secrets
├── Configuration
├── Health Checks
├── Observability
├── Scaling
├── Deployment
├── Rollback
├── Backup
├── Disaster Recovery
├── Security Operations
├── Incident Management
└── Operational Governance
3. Entorno de despliegue

Identity deberá soportar la estrategia de ambientes definida por EVOXA:

Local
   ↓
Development
   ↓
QA
   ↓
Staging
   ↓
Production
   ↓
Disaster Recovery

El Blueprint establece promoción de ambientes:

Dev → QA → Staging → Production.

Cada ambiente deberá estar aislado.

4. Principio de paridad de ambientes

Los ambientes deberán mantener una arquitectura suficientemente equivalente para detectar problemas antes de producción.

No obstante:

Local ≠ Production
Development ≠ Production

por lo que:

Credenciales.
Secrets.
URLs.
Databases.
Providers.
Configuración de seguridad.

deberán ser específicos de cada ambiente.

5. Runtime de Identity

Identity deberá ejecutarse como un servicio desacoplado:

                    API Gateway
                         │
                         ▼
                 Identity Service
                  /      |       \
                 /       |        \
                ▼        ▼         ▼
          PostgreSQL   Redis    Event Platform

El servicio deberá ser:

Stateless donde sea posible.
Horizontalmente escalable.
Observable.
Health-checkable.
Independiente de una instancia específica.

Esto sigue la arquitectura general de EVOXA basada en servicios stateless y escalamiento horizontal.

6. Containerization

El servicio deberá poder ejecutarse de forma reproducible mediante contenedores.

Conceptualmente:

Identity Source
      ↓
Build
      ↓
Container Image
      ↓
Security Scan
      ↓
Registry
      ↓
Deployment

La imagen deberá ser:

Versionada.
Inmutable.
Reproducible.
Escaneada.
Identificable mediante versión/commit.
7. Configuration Management

La configuración deberá estar separada del código.

Ejemplos:

APP_ENV
DATABASE_URL
REDIS_URL
JWT_ISSUER
JWT_AUDIENCE
JWT_SIGNING_CONFIGURATION
EVENT_PLATFORM_URL
MFA_CONFIGURATION
OIDC_CONFIGURATION
RATE_LIMIT_CONFIGURATION

Los valores deberán depender del ambiente.

8. Secrets Management

Los secretos no deberán almacenarse en el repositorio.

Nunca deberán incluirse directamente en:

Source Code
Dockerfile
Git
Logs
Audit Events
API Responses
Configuration committed to repository

Los secretos deberán gestionarse mediante el mecanismo de secrets management de la infraestructura EVOXA.

Ejemplos:

Database credentials.
JWT signing keys.
OAuth secrets.
OIDC secrets.
MFA secrets.
Email provider credentials.
SMS provider credentials.
9. Cryptographic Key Management

Identity utiliza componentes criptográficos críticos.

Deberá existir una estrategia para:

Generación.
Almacenamiento.
Rotación.
Versionado.
Revocación.
Backup seguro.
Recuperación.

Especialmente para:

JWT signing keys
Encryption keys
External provider secrets

La implementación definitiva deberá establecerse mediante ADR.

10. Database Deployment

Identity utilizará PostgreSQL como persistencia transaccional principal propuesta.

Las migraciones deberán ser:

Versionadas.
Repetibles.
Reversibles cuando sea viable.
Ejecutables automáticamente en CI/CD.
Auditables.

No se deberá depender de cambios manuales directamente sobre Production.

11. Database Migration Strategy

Flujo:

Migration
    ↓
Local
    ↓
CI
    ↓
QA
    ↓
Staging
    ↓
Production

Antes de ejecutar una migration en Production deberá verificarse:

Compatibilidad.
Integridad.
Performance.
Rollback strategy.
Impacto sobre versiones anteriores.
12. Backward-Compatible Migrations

Cuando exista un deployment gradual:

Version N
   ↓
Database migration
   ↓
Version N + 1

la migration deberá permitir, cuando sea necesario, coexistencia temporal entre ambas versiones.

Especialmente importante para:

User schema.
Memberships.
Roles.
Sessions.
Refresh tokens.
MFA.
Audit events.
13. Redis Operational Requirements

Redis podrá utilizarse para:

Rate limiting.
Temporary MFA state.
Security counters.
Session acceleration.
Caching.

Pero:

Redis no deberá convertirse accidentalmente en el único origen de información crítica de Identity.

La pérdida de Redis no deberá provocar un estado de autorización permisivo.

14. Event Platform

Identity deberá integrarse con Event Platform:

Identity
   ↓
Event Publisher
   ↓
Event Platform
   ↓
Consumers

Los eventos deberán cumplir:

Versioning.
Schema validation.
Correlation.
Causation.
Tenant context.
Retryability.
Idempotency.

El modelo de eventos de EVOXA contempla entrega at-least-once, retries, DLQ y replay.

15. Transactional Outbox

La publicación de eventos críticos deberá utilizar, cuando corresponda, un patrón equivalente a:

┌─────────────────────────────┐
│ PostgreSQL Transaction      │
│                             │
│  Identity Change            │
│        +                    │
│  Outbox Event               │
└──────────────┬──────────────┘
               │
               ▼
        Event Publisher
               │
               ▼
        Event Platform

Esto reduce el riesgo de inconsistencias entre cambios de datos y eventos.

La decisión definitiva sobre Outbox deberá formalizarse mediante ADR.

16. Health Checks

Identity deberá exponer health checks diferenciados.

Liveness

Pregunta:

¿El proceso está vivo?

GET /health/live
Readiness

Pregunta:

¿El servicio está preparado para recibir tráfico?

GET /health/ready
Startup

Cuando el runtime lo requiera:

GET /health/startup
17. Dependency Health

Readiness deberá considerar las dependencias críticas según la arquitectura final.

Por ejemplo:

Identity
   │
   ├── PostgreSQL
   ├── Redis
   └── Event Platform

Pero deberá evitarse que una dependencia no crítica impida innecesariamente que Identity reciba tráfico.

La clasificación de dependencias deberá quedar definida durante implementación.

18. Metrics

Identity deberá generar métricas operacionales.

Authentication
authentication_attempts
authentication_success
authentication_failure
Authorization
authorization_allow
authorization_deny
Sessions
sessions_created
sessions_revoked
sessions_expired
Tokens
token_refresh
token_refresh_failure
token_reuse_detected
MFA
mfa_challenges
mfa_success
mfa_failure
API
request_count
request_latency
request_errors
19. Logging

Los logs deberán ser:

Structured.
Machine-readable.
Correlacionables.
Sanitizados.
Centralizados.

Formato conceptual:

{
  "timestamp": "...",
  "level": "INFO",
  "service": "identity",
  "event": "authentication_success",
  "correlation_id": "...",
  "trace_id": "...",
  "tenant_id": "...",
  "user_id": "..."
}

Nunca deberán aparecer passwords, tokens, MFA secrets u otros secretos.

20. Distributed Tracing

Identity deberá soportar distributed tracing.

El trace deberá poder seguir:

Client
 ↓
API Gateway
 ↓
Identity
 ↓
PostgreSQL
 ↓
Redis
 ↓
Event Platform

Utilizando:

trace_id
span_id
correlation_id

Esto está alineado con los requisitos generales de observabilidad de EVOXA.

21. Audit Operations

Identity deberá mantener audit separado de logs operacionales.

Ejemplo:

User suspended
      ↓
Audit Event

Role changed
      ↓
Audit Event

MFA enabled
      ↓
Audit Event

Los eventos de auditoría deberán mantenerse según la política de retención aprobada.

La arquitectura de EVOXA establece como objetivo una retención de audit de hasta 7 años, dependiendo del contexto regulatorio y de despliegue.

22. Security Monitoring

Identity deberá generar señales para detectar:

Brute force.
Credential attacks.
Token replay.
Refresh-token reuse.
MFA abuse.
Privilege escalation.
Suspicious authentication.
Cross-tenant attempts.
Repeated authorization failures.

Estas señales podrán alimentar posteriormente Security Operations y Analytics.

23. Rate Limiting

Deberá existir rate limiting sobre operaciones sensibles.

Mínimo:

Login
Password reset
MFA
OTP
Token refresh
Recovery

Los límites deberán ser configurables por ambiente y política.

24. Scaling

Identity deberá permitir escalamiento horizontal:

                 Load Balancer
                      │
          ┌───────────┼───────────┐
          ▼           ▼           ▼
      Identity-1 Identity-2 Identity-3
          │           │           │
          └───────────┼───────────┘
                      ▼
                  PostgreSQL

Las instancias no deberán depender de memoria local para estado crítico compartido.

25. Autoscaling

La implementación deberá permitir autoscaling basado en métricas como:

CPU.
Memory.
Request rate.
Latency.
Concurrent requests.
Queue/event pressure.

Los valores concretos de thresholds deberán definirse por ambiente.

26. Performance Targets

Identity deberá respetar los objetivos arquitectónicos:

Operación	Target
Authentication	P95 < 500 ms
API general	P95 < 300 ms
Authorization	Compatible con API target
Token refresh	Compatible con API target

Los valores son objetivos arquitectónicos y deberán validarse mediante pruebas de carga reales.

27. Availability

El objetivo general de EVOXA es:

Availability ≥ 99.9%

para servicios de plataforma.

Identity deberá considerarse un servicio crítico para alcanzar este objetivo.

28. Deployment Strategy

Los deployments deberán seguir:

Build
 ↓
Validate
 ↓
Test
 ↓
Security Scan
 ↓
Artifact
 ↓
Deploy
 ↓
Verify
 ↓
Monitor

El Blueprint establece el patrón:

Artifact → Validation → Deployment → Verification.

29. Deployment Strategies

Identity deberá poder soportar estrategias como:

Rolling.
Blue/Green.
Canary.
Progressive Delivery.

La estrategia concreta dependerá del entorno y criticidad del release.

30. Zero-Downtime Deployment

Los deployments de Production deberán buscar:

Old Version
     │
     ├─────────────┐
     │             ▼
     │        New Version
     │             │
     └─────────────┘

sin interrupción innecesaria del servicio.

Esto requiere:

Backward-compatible APIs.
Compatible database migrations.
Session continuity.
Token compatibility.
31. Token Compatibility During Deployment

Un deployment no deberá invalidar accidentalmente las sesiones válidas.

Por ejemplo:

Version A
  ↓
Access Token
  ↓
Version B
  ↓
Token remains valid

salvo que exista una decisión explícita de seguridad que indique lo contrario.

32. Rollback

Todo deployment deberá tener rollback definido.

New Version
    ↓
Problem detected
    ↓
Rollback
    ↓
Previous Version
    ↓
Verification

El rollback deberá considerar:

Application version.
Database schema.
Configuration.
Secrets.
Events.
Token compatibility.
33. Database Rollback

No todas las migrations son reversibles.

Por ello, se deberá distinguir:

Application Rollback

de:

Database Recovery

Para cambios destructivos deberá existir estrategia de recuperación antes del deployment.

34. Backup

Deberán existir backups de:

PostgreSQL.
Configuración crítica.
Audit data.
Event data cuando corresponda.

Los backups deberán ser:

Automatizados.
Monitorizados.
Protegidos.
Probados mediante restauración.
35. Restore Testing

Un backup no será considerado confiable simplemente porque exista.

Deberá probarse:

Backup
 ↓
Restore
 ↓
Validation
 ↓
Application Recovery

La restauración deberá comprobar integridad de:

Users.
Organizations.
Memberships.
Roles.
Sessions según estrategia.
Audit.
Eventos.
36. Disaster Recovery

El Blueprint establece como objetivos:

RTO < 1 hora
RPO < 15 minutos.

Identity deberá participar en la estrategia global de DR.

37. Disaster Recovery Scenario

Ejemplo:

Production Failure
       ↓
Detect
       ↓
Classify
       ↓
Activate DR
       ↓
Restore Data
       ↓
Deploy Identity
       ↓
Verify
       ↓
Resume Traffic

Las operaciones críticas de autenticación deberán validarse antes de declarar el servicio recuperado.

38. Session Recovery

No se deberá asumir que todas las sesiones existentes pueden restaurarse automáticamente después de un desastre.

La estrategia deberá definir:

Session persistence.
Refresh-token persistence.
Token invalidation.
Reauthentication requirements.

Esta decisión requiere ADR.

39. Security Incident Response

Ante un incidente de Identity:

Detection
   ↓
Classification
   ↓
Containment
   ↓
Mitigation
   ↓
Recovery
   ↓
Postmortem
   ↓
Improvement

Este lifecycle está alineado con Operational Excellence del Blueprint.

40. Incident Examples

Identity deberá tener runbooks para incidentes como:

INC-IDENTITY-001
Authentication outage

INC-IDENTITY-002
Database unavailable

INC-IDENTITY-003
Redis unavailable

INC-IDENTITY-004
Refresh token compromise

INC-IDENTITY-005
JWT signing key compromise

INC-IDENTITY-006
MFA abuse

INC-IDENTITY-007
Cross-tenant access detected

INC-IDENTITY-008
Privilege escalation detected
41. Key Compromise

Si una signing key de Identity es comprometida, deberá existir capacidad operacional para:

Detect
 ↓
Revoke/rotate key
 ↓
Publish new key
 ↓
Invalidate affected tokens if necessary
 ↓
Reauthenticate users if required
 ↓
Audit incident

La estrategia exacta deberá formalizarse mediante ADR.

42. Service Ownership

Identity deberá tener definidos:

Business Owner
Technical Owner
Operational Owner
On-call Team
Escalation Contacts
Runbooks
SLOs
Error Budget

Esto sigue el modelo de ownership definido por Operational Excellence.

43. SLOs

Como mínimo se deberán monitorizar:

Availability
≥ 99.9%
Authentication latency
P95 < 500 ms
API latency
P95 < 300 ms
Error rate

Objetivo arquitectónico:

< 1%

Los SLO definitivos deberán ajustarse al entorno Productivo real.

44. Error Budget

Identity deberá adoptar un modelo de Error Budget.

Conceptualmente:

SLO
 │
 ├── Healthy
 │
 ├── Error Budget Remaining
 │
 └── Error Budget Exhausted

Cuando el error budget se consuma, deberán revisarse:

Releases.
Cambios de arquitectura.
Performance.
Incidentes.
Technical debt.
45. Monitoring Dashboard

Deberá existir un dashboard operacional de Identity mostrando como mínimo:

Availability
Latency
Error Rate
Authentication Success
Authentication Failure
Authorization Denials
Active Sessions
Token Refresh
Token Reuse
MFA Failures
Rate Limits
Database Health
Redis Health
Event Processing
46. Alerts

Deberán existir alertas para:

Authentication outage.
Elevated authentication failures.
Elevated latency.
High authorization denial rate.
Token reuse detection.
MFA abuse.
Database failure.
Redis failure cuando afecte operaciones.
Event publishing failures.
Error budget exhaustion.
47. Release Governance

Un release de Identity deberá tener:

Change
 ↓
Review
 ↓
Tests
 ↓
Security Validation
 ↓
Approval
 ↓
Deployment
 ↓
Verification

Para cambios críticos de seguridad deberá existir una aprobación explícita según la gobernanza definida por EVOXA.

48. Production Verification

Después del deployment:

Deployment
 ↓
Smoke Tests
 ↓
Health Checks
 ↓
Authentication Test
 ↓
Authorization Test
 ↓
Database Check
 ↓
Event Check
 ↓
Monitoring

No se deberá considerar exitoso un deployment simplemente porque el container esté running.

49. Operational Smoke Tests

Como mínimo:

GET /health/live
GET /health/ready
POST /api/v1/identity/auth/login
GET /api/v1/identity/auth/me
POST /api/v1/identity/auth/refresh
POST /api/v1/identity/auth/logout

Los tests reales dependerán del contrato API final.

50. CI/CD

Pipeline conceptual:

Commit
 ↓
Lint
 ↓
Type Check
 ↓
Unit Tests
 ↓
Coverage
 ↓
Integration Tests
 ↓
API Contract Tests
 ↓
Security Tests
 ↓
Build
 ↓
Container Scan
 ↓
Artifact
 ↓
Deploy QA
 ↓
E2E
 ↓
Deploy Staging
 ↓
Approval
 ↓
Production

Esto sigue la estrategia de Continuous Delivery y quality gates del Blueprint.

51. Artifact Management

Cada release deberá producir un artefacto identificable:

identity:<version>

Idealmente asociado a:

Git commit
Build ID
Release ID
Environment
Timestamp

Los artefactos deberán ser inmutables.

52. Versioning

Identity deberá seguir versionado semántico cuando corresponda:

MAJOR.MINOR.PATCH

Los cambios incompatibles de API deberán requerir una nueva versión mayor, siguiendo la estrategia API del Blueprint.

53. Operational Documentation

Identity deberá disponer de:

README
Architecture
Runbooks
Deployment Guide
Recovery Guide
Security Guide
Troubleshooting
API Documentation
Monitoring Guide
Incident Procedures

La documentación deberá evolucionar junto con el código.

54. Runbooks mínimos

Se deberán crear runbooks para:

Identity deployment.
Identity rollback.
Database migration.
Database recovery.
Redis failure.
Event Platform failure.
JWT key rotation.
Refresh token compromise.
MFA incident.
Authentication outage.
Authorization incident.
Disaster recovery.
55. Operational Security

Production deberá utilizar:

TLS.
Secrets management.
Restricted network access.
Least privilege.
MFA para operadores.
Audit logging.
Administrative access controls.
Security monitoring.

Esto sigue los principios de Zero Trust, least privilege y defense in depth establecidos para EVOXA.

56. Production Access

El acceso administrativo a Identity deberá:

Estar autenticado.
Estar autorizado.
Requerir MFA cuando corresponda.
Ser auditable.
Aplicar least privilege.
Evitar cuentas compartidas.

Las operaciones administrativas críticas deberán generar audit events.

57. Data Retention

Identity deberá aplicar políticas diferenciadas para:

Operational data
Audit data
Security events
Sessions
Refresh tokens
MFA challenges
Logs
Metrics
Traces

No todo dato deberá tener la misma retención.

La política definitiva deberá considerar requisitos legales y de privacidad.

58. Privacy Operations

Identity deberá soportar las políticas de privacidad aplicables a:

User data.
Authentication history.
Sessions.
Audit.
Security events.

Deberán respetarse los principios de:

Data minimization.
Purpose limitation.
Access control.
Retention.
Secure deletion cuando corresponda.
59. Operational Compliance

Identity deberá proporcionar evidencia suficiente para soportar los marcos aplicables al deployment:

GDPR.
HIPAA cuando corresponda.
ISO 27001.
SOC 2.
Legislación local aplicable.

El Blueprint deja claro que la aplicabilidad depende del contexto de despliegue.

60. FinOps

Identity deberá monitorizar costos operacionales asociados a:

Compute.
PostgreSQL.
Redis.
Event Platform.
Storage.
Observability.
External identity providers.
MFA providers.

Esto permitirá posteriormente aplicar las capacidades de FinOps de EVOXA.

61. Operational Quality Gates

Antes de Production:

Infrastructure
 Container validado.
 Configuration validada.
 Secrets configurados.
 Database disponible.
 Redis disponible si aplica.
 Event Platform disponible.
Security
 Security tests.
 Vulnerability scan.
 Key configuration.
 MFA administrativa.
 Tenant isolation.
Application
 Health checks.
 Metrics.
 Logs.
 Tracing.
 Audit.
 Alerts.
Recovery
 Backup.
 Restore validation.
 Rollback.
 DR procedure.
62. Definition of Operational Readiness

Identity estará Operationally Ready cuando:

Code
 ↓
Tests
 ↓
Security
 ↓
Container
 ↓
Deployment
 ↓
Health
 ↓
Observability
 ↓
Backup
 ↓
Recovery
 ↓
Runbooks
 ↓
SLO
 ↓
Operational Approval

estén validados.

63. Traceability

La trazabilidad queda:

Blueprint
   ↓
Reference Architecture
   ↓
ESP-0001
   ↓
Implementation Stories
   ↓
Technical Tasks
   ↓
Source Code
   ↓
Tests
   ↓
Deployment
   ↓
Monitoring
   ↓
Continuous Evolution

Esto es especialmente importante porque el Blueprint establece que ninguna implementación debería saltarse este lifecycle.

64. ADR Candidates

Este capítulo genera los siguientes ADR:

ADR	Tema
ADR-IDENTITY-013	Runtime & Container Strategy
ADR-IDENTITY-014	Environment Strategy
ADR-IDENTITY-015	Secrets Management
ADR-IDENTITY-016	Cryptographic Key Management
ADR-IDENTITY-017	Database Migration Strategy
ADR-IDENTITY-018	Redis Operational Strategy
ADR-IDENTITY-019	Transactional Outbox
ADR-IDENTITY-020	Health Check Strategy
ADR-IDENTITY-021	Deployment Strategy
ADR-IDENTITY-022	Rollback Strategy
ADR-IDENTITY-023	Backup & Restore
ADR-IDENTITY-024	Disaster Recovery
ADR-IDENTITY-025	Session Recovery
ADR-IDENTITY-026	SLO & Error Budget
ADR-IDENTITY-027	Production Key Rotation
ADR-IDENTITY-028	Identity Incident Response

Estos IDs son propuestos, no IDs oficiales todavía.

65. Estado de ESP-0001

Con este capítulo tenemos:

Capítulo	Estado
01 Domain Model	✅ Draft
02 Architecture	✅ Draft
03 Authentication & Authorization	✅ Draft
04 User & Organization Management	✅ Draft
05 Roles & Permissions	✅ Draft
06 Sessions & Token Management	✅ Draft
07 MFA	✅ Draft
08 Security Policies	✅ Draft
09 Audit & Compliance	✅ Draft
10 API Contracts	✅ Draft
11 Persistence & Data Model	✅ Draft
12 Events & Integration	✅ Draft
13 Testing & Quality	✅ Draft
14 Implementation Requirements	✅ Draft
15 Deployment & Operational Requirements	✅ Draft
Conclusión

Con ESP-0001 / 15 ya cerramos la especificación principal de Identity desde el punto de vista de:

modelo → arquitectura → seguridad → usuarios → organizaciones → permisos → sesiones → MFA → políticas → auditoría → API → persistencia → eventos → testing → implementación → deployment → operaciones.

El siguiente paso no debería ser empezar a escribir código todavía. Lo correcto, siguiendo la trazabilidad del Blueprint, es pasar a la capa de ADR (Architecture Decision Records) para convertir las decisiones abiertas de Identity en decisiones formales.

El orden que recomiendo es:

ESP-0001 / 01–15
        ↓
ADR-IDENTITY
        ↓
Implementation Stories
        ↓
Technical Tasks
        ↓
Source Code
        ↓
Tests
        ↓
Deployment

El siguiente documento recomendado sería:

ADR-IDENTITY-001 — Identity Architecture & Technology Decisions

Ahí podemos tomar todas las decisiones que quedaron abiertas en los 15 capítulos —por ejemplo PostgreSQL, Redis, JWT, Argon2id, refresh-token rotation, MFA, RBAC + ABAC, estructura del proyecto, ORM, API framework, migrations, outbox, event bus, secrets, deployment y testing— y decidirlas formalmente antes de generar las Implementation Stories.
