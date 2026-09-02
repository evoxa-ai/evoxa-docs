ADR-IDENTITY-011 — Identity Deployment & Runtime Architecture

Status: Proposed
Version: 1.0.0
Domain: Identity
Decision Type: Architecture / Deployment / Operations
Scope: Runtime, deployment, environments, containers, configuration, secrets, health checks, scaling, observability, resilience and recovery
Related: ESP-0001 — Identity
Depends on:

ADR-IDENTITY-001 — Identity Architecture & Technology Decisions
ADR-IDENTITY-002 — JWT & Token Lifecycle
ADR-IDENTITY-003 — Refresh Token Security & Session Management
ADR-IDENTITY-006 — Identity Persistence Strategy
ADR-IDENTITY-007 — Identity Event & Outbox Strategy
ADR-IDENTITY-008 — Identity Cryptographic Key Management
ADR-IDENTITY-009 — Identity Client Token Storage
ADR-IDENTITY-010 — Identity Testing Strategy
1. Context

Identity es un componente crítico de EVOXA y constituye una dependencia transversal para prácticamente todos los dominios de la plataforma.

La arquitectura de EVOXA define una plataforma:

modular;
orientada a dominios;
API-First;
Event-Driven;
Cloud-Ready;
observable;
segura por diseño;
continuamente evolutiva.

Por ello, Identity no puede desplegarse como un componente aislado sin considerar:

API
Database
Redis
Event Platform
Key Management
Secrets
Observability
Security
CI/CD
Disaster Recovery

Además, la arquitectura de despliegue de EVOXA contempla promoción entre ambientes:

Development
    ↓
QA
    ↓
Staging
    ↓
Production

con validación, despliegue, verificación y capacidad de rollback.

2. Problem Statement

Identity debe funcionar de manera:

Secure
Highly Available
Scalable
Observable
Recoverable
Deployable

sin introducir dependencias de estado local que impidan escalar horizontalmente.

Por ejemplo, no sería apropiado depender de:

Identity Instance A
    ↓
Local Session State

porque:

Request 1 → Instance A
Request 2 → Instance B

podría perder el contexto necesario.

La estrategia de sesiones y refresh tokens ya definida utiliza persistencia stateful, por lo que el runtime puede mantenerse esencialmente stateless.

3. Decision

Se adopta una arquitectura de runtime:

Containerized, Stateless Application Runtime + Externalized State + Horizontally Scalable Identity Instances.

Conceptualmente:

                    Client
                      │
                      ▼
                API Gateway
                      │
              ┌───────┴───────┐
              ▼               ▼
        Identity Instance  Identity Instance
              │               │
              └───────┬───────┘
                      │
          ┌───────────┼────────────┐
          ▼           ▼            ▼
     PostgreSQL     Redis     Event Platform
                      │
                      ▼
                Key / Secrets
                Management

Las instancias de Identity no deberán depender de almacenamiento local como fuente de estado de negocio.

4. Runtime Model

El runtime seguirá:

Client
  ↓
API Gateway / Load Balancer
  ↓
Identity Service
  ├── Instance A
  ├── Instance B
  └── Instance N
       │
       ├── PostgreSQL
       ├── Redis
       ├── Event Platform
       └── KMS / Secret Manager

La cantidad de instancias podrá variar según:

carga;
disponibilidad;
capacidad;
ambiente;
políticas de autoscaling.
5. Stateless Application

Identity deberá mantener el runtime de aplicación lo más stateless posible.

No se utilizará memoria local como fuente de verdad para:

User State
Session State
Refresh Token State
Membership State
Role State
Permission State
Security Policy State

Estos datos estarán en los sistemas persistentes definidos en ADR-IDENTITY-006.

6. Stateful Security Where Required

Aunque el runtime sea stateless, Identity mantiene estado de seguridad persistente:

PostgreSQL
   ↓
Sessions
Refresh Tokens
MFA
Policies

Por tanto:

Stateless Runtime
        +
Stateful Security Persistence

no son conceptos contradictorios.

7. Containerization

Identity será empaquetado como un artefacto desplegable reproducible, preferentemente mediante contenedor.

Conceptualmente:

Source Code
    ↓
Build
    ↓
Container Image
    ↓
Security Scan
    ↓
Artifact Registry
    ↓
Deployment

La imagen debe ser:

versionada;
reproducible;
inmutable;
escaneada;
libre de secretos;
promocionable entre ambientes.
8. Immutable Artifact

Una misma imagen validada deberá promoverse entre ambientes siempre que el proceso de release lo permita:

Build
 ↓
QA
 ↓
Staging
 ↓
Production

No se debe recompilar arbitrariamente una aplicación diferente para Production después de haber validado otra en QA.

Esto reduce el riesgo de:

QA Artifact ≠ Production Artifact
9. Environments

Identity deberá soportar:

LOCAL
DEVELOPMENT
QA
STAGING
PRODUCTION
DR

La arquitectura general de EVOXA contempla estos ambientes como parte de su estrategia de despliegue y operaciones.

10. Environment Isolation

Cada ambiente debe tener:

Database
Redis
Secrets
Keys
Configuration
Event Topics

aislados según corresponda.

Especialmente:

Production

no debe compartir:

JWT Signing Keys
Database Credentials
MFA Encryption Keys

con Development o QA.

Esta decisión es coherente con ADR-IDENTITY-008.

11. Configuration

La configuración debe estar externalizada.

Ejemplo conceptual:

Application
   ↓
Environment Configuration
   ↓
Runtime

No se deben hardcodear:

database credentials
JWT keys
KMS credentials
API secrets
SMTP passwords

en el código.

12. Secrets

Los secretos deberán obtenerse mediante mecanismos seguros de gestión de secretos.

Identity Runtime
      ↓
Secret Manager
      ↓
Secrets

La estrategia concreta de KMS/Secret Manager queda definida como decisión posterior de infraestructura.

13. Configuration Categories

Se deberá distinguir:

Non-sensitive configuration
API_PORT
LOG_LEVEL
ENVIRONMENT
FEATURE_FLAGS
Sensitive configuration
DATABASE_PASSWORD
OIDC_CLIENT_SECRET
WEBHOOK_SECRET
KMS_CREDENTIAL

Las segundas deben utilizar secret management.

14. Database Connectivity

Identity se conectará a PostgreSQL mediante un mecanismo de conexión administrado.

Debe contemplarse:

connection pooling;
connection limits;
timeout;
retry controlado;
health checks;
graceful shutdown.

La configuración exacta dependerá del entorno.

15. Redis Runtime

Redis será utilizado como dependencia complementaria.

Casos:

Rate Limiting
MFA Temporary State
Authorization Cache
Security Counters
Short-lived State

Identity no debe asumir que Redis contiene la única copia de información crítica cuando PostgreSQL es la fuente de verdad.

16. Redis Failure

Si Redis no está disponible:

Identity
   ↓
Redis unavailable

el comportamiento dependerá de la operación.

Para operaciones no críticas:

Fallback

cuando sea seguro.

Para controles de seguridad críticos:

Fail Closed

si no puede garantizarse una decisión segura.

Nunca:

Redis Failure
   ↓
Authorization Bypass
17. Event Platform

Identity publicará eventos mediante la arquitectura definida en ADR-IDENTITY-007:

PostgreSQL
     ↓
Transactional Outbox
     ↓
Publisher
     ↓
Event Platform

El runtime de Identity no deberá depender de la disponibilidad inmediata del Event Platform para completar una transacción de negocio crítica.

18. Outbox Worker

El procesamiento de outbox puede ejecutarse:

Identity Runtime

o mediante:

Dedicated Worker

La arquitectura debe permitir escalar el publisher independientemente cuando el volumen lo requiera.

Conceptualmente:

Identity API
     │
     ▼
PostgreSQL
     │
     ▼
Outbox
     │
     ▼
Publisher Workers
     │
     ▼
Event Platform
19. Cryptographic Runtime

Identity deberá integrarse con:

Key Management
Secret Management

según ADR-IDENTITY-008.

El runtime debe poder:

firmar JWT;
validar JWT cuando corresponda;
acceder a secretos autorizados;
utilizar claves de forma segura;
soportar rotación.

Las claves privadas no deberán formar parte de la imagen del contenedor.

20. JWT Key Rotation

Durante una rotación:

Identity
 ├── Old Key → Verify
 └── New Key → Sign + Verify

El runtime deberá soportar temporalmente múltiples claves públicas.

Esto permite:

Zero-Downtime Rotation

cuando la infraestructura de claves lo permita.

21. Health Checks

Identity deberá exponer endpoints de salud.

Se definen conceptualmente:

/health/live
/health/ready
/health/startup
Liveness

Responde:

¿El proceso está vivo?

No debería depender de todas las dependencias externas.

Readiness

Responde:

¿La instancia está preparada para recibir tráfico?

Puede considerar dependencias críticas.

Startup

Permite determinar:

¿La aplicación terminó correctamente su inicialización?

22. Liveness

Ejemplo:

GET /health/live

Respuesta conceptual:

{
  "status": "ok"
}

La liveness no debe generar falsos reinicios simplemente porque PostgreSQL o Redis estén temporalmente indisponibles.

23. Readiness

Ejemplo:

GET /health/ready

Puede verificar:

Database connectivity
Critical configuration
Required runtime dependencies

Si la instancia no está preparada:

ready = false

y el balanceador/orquestador debe dejar de enviarle tráfico.

24. Startup

Para inicializaciones más lentas:

GET /health/startup

permite diferenciar:

Starting

de:

Running but unhealthy
25. Graceful Shutdown

Antes de terminar una instancia:

Receive shutdown signal
       ↓
Stop accepting new traffic
       ↓
Finish active requests
       ↓
Close DB connections
       ↓
Close Redis connections
       ↓
Stop workers
       ↓
Exit

Esto reduce errores durante:

deployments;
autoscaling;
restarts;
node replacement.
26. Horizontal Scaling

Identity deberá poder escalar horizontalmente:

1 instance
   ↓
2 instances
   ↓
N instances

sin depender de estado local.

Ejemplo:

             Load Balancer
              /    |    \
             /     |     \
            ▼      ▼      ▼
         ID-01   ID-02   ID-03
            \      |      /
             \     |     /
              ▼    ▼    ▼
              PostgreSQL
27. Autoscaling

El autoscaling podrá basarse en:

CPU;
memoria;
request rate;
latency;
concurrency;
custom metrics.

No se establece aquí un threshold concreto porque depende del capacity planning real.

28. Availability

El objetivo arquitectónico para EVOXA es:

Availability ≥ 99.9%

Identity deberá diseñarse para no convertirse en un single point of failure.

Esto implica:

Multiple Instances
+
Redundant Dependencies
+
Health Checks
+
Automated Recovery

cuando el entorno de infraestructura lo soporte.

29. Deployment Strategy

Se utilizarán estrategias compatibles con:

Rolling
Blue-Green
Canary
Progressive Delivery

según el ambiente y el nivel de riesgo.

Para cambios críticos de Identity puede ser preferible:

Canary

o:

Progressive Rollout

antes de exponer la nueva versión a todo el tráfico.

30. Database Migration Strategy

Las migraciones deben ser compatibles con deployments graduales.

Ejemplo:

Version N
   ↓
Add nullable column
   ↓
Deploy N+1
   ↓
Backfill
   ↓
Enable new behavior
   ↓
Remove old field later

Se debe evitar:

Deploy
 ↓
Breaking DB migration
 ↓
Old instances fail

Esto es especialmente importante durante rolling deployments.

31. Zero-Downtime Deployment

Identity deberá diseñarse para soportar despliegues sin interrupciones cuando el entorno lo permita.

Conceptualmente:

Old Version
    │
    ├── Serving
    │
New Version
    │
    ├── Starting
    ├── Ready
    └── Receiving Traffic
             │
             ▼
Old Version
    ↓
Draining
    ↓
Shutdown
32. Rollback

Si una versión presenta problemas:

Deployment
    ↓
Monitoring
    ↓
Failure
    ↓
Rollback

El rollback debe ser posible sin producir una corrupción de datos.

Por ello:

Database migrations deben diseñarse teniendo en cuenta la posibilidad de rollback de aplicación.

33. Security at Runtime

Identity deberá ejecutarse con:

mínimo privilegio;
usuario no privilegiado cuando sea posible;
filesystem restringido;
capabilities mínimas;
network access limitado;
secretos externalizados;
imágenes escaneadas.

La arquitectura Zero Trust y Least Privilege se mantiene como principio transversal.

34. Network Architecture

Conceptualmente:

Internet
   ↓
WAF / API Gateway
   ↓
Identity
   ↓
Private Network
   ├── PostgreSQL
   ├── Redis
   ├── Event Platform
   └── KMS / Secret Manager

PostgreSQL y Redis no deberían exponerse directamente a Internet.

35. TLS

La comunicación entre componentes sensibles deberá utilizar canales protegidos.

Especialmente:

Client → API
Identity → Database
Identity → Redis
Identity → Event Platform
Identity → KMS

según las capacidades del entorno.

La arquitectura de seguridad contempla TLS 1.3 como referencia.

36. Rate Limiting

Identity deberá incorporar rate limiting especialmente para:

/login
/auth/refresh
/password-reset
/mfa/*

y otras operaciones sensibles.

Esto protege frente a:

brute force;
credential stuffing;
token abuse;
MFA abuse;
denial of service.
37. Runtime Observability

Identity deberá producir:

Logs
Metrics
Traces
Audit Events
Security Events

La arquitectura de observabilidad de EVOXA requiere trazabilidad distribuida y métricas técnicas, de negocio y de seguridad.

38. Structured Logging

Los logs deberán utilizar formato estructurado.

Conceptualmente:

{
  "timestamp": "...",
  "level": "INFO",
  "service": "identity",
  "event": "authentication.success",
  "user_id": "...",
  "tenant_id": "...",
  "correlation_id": "...",
  "trace_id": "..."
}

Nunca:

{
  "password": "...",
  "access_token": "...",
  "refresh_token": "..."
}
39. Metrics

Identity deberá observar métricas como:

authentication_requests
authentication_failures
authorization_denials
token_refresh_requests
token_refresh_failures
token_reuse_detected
active_sessions
mfa_challenges
mfa_failures
rate_limit_hits
database_latency
redis_latency
outbox_backlog
event_publish_failures
40. Distributed Tracing

Las operaciones críticas deben mantener:

trace_id
span_id
correlation_id

a través de:

API
Identity
Database
Redis
Event Platform
External IdP
KMS

cuando sea técnicamente posible.

41. Security Monitoring

Identity deberá generar alertas ante comportamientos como:

TokenReuseDetected
Multiple MFA failures
Credential attack pattern
Abnormal authentication volume
Cross-tenant access attempts
Privilege escalation attempts
Repeated authorization failures

La estrategia concreta de detección pertenece al dominio Security/Observability.

42. Disaster Recovery

Identity debe participar en el plan de DR de EVOXA.

Dependencias:

Identity
 ├── PostgreSQL
 ├── Redis
 ├── Event Platform
 └── Key / Secret Management

La recuperación debe considerar todas ellas.

43. RTO / RPO

Los objetivos arquitectónicos generales son:

RTO < 1 hour
RPO < 15 minutes

Estos valores deberán validarse según el entorno y deployment real.

44. Backup

PostgreSQL deberá tener:

backups;
restore testing;
monitoring;
retention;
disaster recovery.

Redis deberá evaluarse según el tipo de información que contenga.

Como principio:

No se debe depender de Redis como único almacenamiento de información que requiera recuperación durable.

45. Deployment Pipeline

El pipeline conceptual será:

Developer Commit
       ↓
Lint
       ↓
Unit Tests
       ↓
Component Tests
       ↓
Integration Tests
       ↓
Security Tests
       ↓
Build
       ↓
Container Scan
       ↓
Artifact
       ↓
Deploy Dev
       ↓
QA
       ↓
Staging
       ↓
Production
       ↓
Verification
       ↓
Monitoring

Esto integra ADR-IDENTITY-010 con la estrategia general de release de EVOXA.

46. Production Verification

Después del deployment:

Deploy
  ↓
Health Checks
  ↓
Smoke Tests
  ↓
Authentication Test
  ↓
Authorization Test
  ↓
Token Test
  ↓
Monitoring

Si se detecta degradación:

Rollback

cuando corresponda.

47. Configuration Changes

Los cambios de configuración sensibles deben ser controlados y auditables.

Ejemplos:

JWT configuration
MFA policy
Rate limits
Session policy
Security policies
External IdP configuration

No deberían modificarse directamente en producción sin trazabilidad.

48. Feature Flags

Cuando una funcionalidad de Identity sea riesgosa, podrá utilizar:

Feature Flag

Ejemplo:

New MFA Flow
      ↓
5% users
      ↓
25%
      ↓
50%
      ↓
100%

Esto es especialmente útil para cambios de alto impacto.

49. Runtime Dependencies

Identity tendrá dependencias clasificadas:

Critical
PostgreSQL
Key Management
Required Configuration
Important
Redis
Event Platform

según la operación.

El comportamiento ante la caída de cada dependencia deberá ser definido y probado.

50. Dependency Failure Matrix
Dependency	Failure	Expected Behavior
PostgreSQL	Unavailable	Controlled failure
Redis	Unavailable	Fallback o fail-closed según operación
Event Platform	Unavailable	Outbox + retry
KMS	Unavailable	Fail closed para operaciones criptográficas dependientes
External IdP	Unavailable	Provider-specific controlled failure
Secret Manager	Unavailable	Startup/runtime failure según secreto requerido
51. Security Failure Principle

Regla fundamental:

Dependency Failure
       ↓
Cannot Verify Security
       ↓
DENY / Fail Closed

Nunca:

Cannot Verify Security
       ↓
ALLOW

Esto mantiene la decisión establecida en ADR-IDENTITY-005.

52. Operational Ownership

Identity deberá tener ownership explícito.

Conceptualmente:

Business Owner
Technical Owner
Operational Owner
On-call Team
Runbooks
SLOs
Error Budget
Escalation

Esto es consistente con el modelo de Operational Excellence del Blueprint, que exige ownership, runbooks, SLOs, error budgets y procedimientos de incidentes.

53. Runbooks

Deben existir procedimientos documentados para:

Identity outage
Database outage
Redis outage
JWT key compromise
Refresh token attack
MFA attack
Event backlog
Deployment rollback
Emergency key rotation
Database restore
54. Operational Alerts

Alertas prioritarias:

Authentication failure spike
Authorization denial spike
TokenReuseDetected
MFA failure spike
Database unavailable
Redis unavailable
Outbox backlog
Event publish failures
High latency
High error rate
Instance unhealthy
55. Performance Targets

Identity debe respetar:

Authentication P95 < 500 ms
API P95 < 300 ms

cuando las condiciones de infraestructura y carga estén dentro de los objetivos definidos.

El despliegue debe observar estos indicadores continuamente.

56. Capacity Planning

Antes de Production deberá estimarse:

Concurrent users
Requests/sec
Login rate
Refresh rate
Authorization rate
Database connections
Redis operations
Event throughput

Los límites concretos no se fijan en esta ADR.

57. Testing Deployment

Los deployments deberán validar:

Startup
Application starts
Configuration valid
Secrets available
Database reachable
Runtime
Health checks
Authentication
Authorization
Token refresh
MFA
Events
Recovery
Restart
Failover
Rollback
Dependency failure
58. Alternatives Considered
A. Stateful Application Instances

Cada instancia mantendría sesiones en memoria.

Rechazada

Complica:

horizontal scaling;
failover;
deployments;
session consistency.
B. Single Identity Instance
Rechazada

Crearía un single point of failure y limitaría escalabilidad.

C. Serverless-Only Identity
No seleccionada como decisión principal

Puede ser viable en determinados entornos, pero introduce decisiones adicionales sobre:

cold starts;
connection management;
state;
event processing;
runtime limits.

No existe suficiente soporte en el Blueprint para imponerla.

D. Containerized Stateless Runtime
Seleccionada

Es consistente con:

modularidad;
escalamiento horizontal;
sesiones stateful externalizadas;
cloud readiness;
deployment progresivo.
59. Consequences
Positivas
Horizontal scaling.
Alta disponibilidad.
Deployments sin downtime.
Mejor aislamiento de estado.
Recovery controlado.
Integración clara con PostgreSQL/Redis/Event Platform.
Compatible con JWT.
Compatible con multi-instance runtime.
Facilita observabilidad.
Facilita CI/CD.
Negativas
Mayor complejidad operacional.
Requiere infraestructura externa.
Requiere configuración de health checks.
Requiere monitoring.
Requiere gestión de migrations.
Requiere estrategia de deployment y rollback.
60. Implementation Direction

Una estructura conceptual podría ser:

apps/api/

├── app/
│   └── domains/
│       └── identity/
│
├── health/
│   ├── liveness
│   ├── readiness
│   └── startup
│
├── workers/
│   └── outbox/
│
└── infrastructure/
    ├── config/
    ├── secrets/
    ├── observability/
    └── runtime/

Infraestructura:

infrastructure/

├── docker/
├── kubernetes/
├── terraform/
├── environments/
│   ├── dev/
│   ├── qa/
│   ├── staging/
│   └── production/
└── monitoring/

Esta estructura es una dirección de implementación derivada y no una estructura prescrita literalmente por el Blueprint.

61. Final Decision Summary

La arquitectura de runtime queda:

                         INTERNET
                            │
                            ▼
                    WAF / API Gateway
                            │
                            ▼
                     Load Balancer
                            │
             ┌──────────────┼──────────────┐
             ▼              ▼              ▼
        Identity-01    Identity-02    Identity-N
             │              │              │
             └──────────────┼──────────────┘
                            │
          ┌─────────────────┼─────────────────┐
          ▼                 ▼                 ▼
     PostgreSQL           Redis         Event Platform
          │                                   │
          ▼                                   ▼
      Source of                       Transactional
        Truth                             Events
          │
          └──────────────┐
                         ▼
                 KMS / Secret Manager
Decisión principal

EVOXA Identity utilizará un runtime containerizado y horizontalmente escalable, manteniendo la aplicación esencialmente stateless y externalizando el estado crítico hacia PostgreSQL, Redis y los mecanismos de eventos definidos por la arquitectura. Identity utilizará health checks, graceful shutdown, observabilidad, despliegues progresivos, migraciones compatibles, rollback y gestión externa de secretos y claves. El sistema deberá operar bajo principios de Zero Trust, Least Privilege y Fail-Closed, evitando que una falla de infraestructura pueda producir un bypass de seguridad.

Estado: PROPOSED.

62. Cadena completa de ADRs de Identity

Con ADR-011 queda prácticamente cerrada la primera serie arquitectónica de Identity:

ADR-IDENTITY-001
Architecture & Technology
        ↓
ADR-IDENTITY-002
JWT & Token Lifecycle
        ↓
ADR-IDENTITY-003
Refresh Token Security & Sessions
        ↓
ADR-IDENTITY-005
Authorization & Policy Engine
        ↓
ADR-IDENTITY-006
Persistence Strategy
        ↓
ADR-IDENTITY-007
Event & Outbox Strategy
        ↓
ADR-IDENTITY-008
Cryptographic Key Management
        ↓
ADR-IDENTITY-009
Client Token Storage
        ↓
ADR-IDENTITY-010
Testing Strategy
        ↓
ADR-IDENTITY-011
Deployment & Runtime Architecture

Esta secuencia encaja con el ciclo de trazabilidad del Blueprint:

Architecture → Engineering Specifications → ADRs → Implementation → Testing → Deployment → Monitoring → Continuous Evolution.

El siguiente paso recomendado ya no sería crear ADRs arbitrariamente, sino pasar a los ADRs que hayan quedado explícitamente abiertos o, si la serie de Identity está completa, comenzar a convertir ESP-0001 + ADR-001…011 en los Implementation Stories (IS) y Technical Tasks que preceden al código.
