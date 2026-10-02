E76 — EVOXA RUNTIME INFRASTRUCTURE ARCHITECTURE
1. Propósito

E76 define la infraestructura técnica sobre la que EVOXA se ejecuta de forma continua.

E74 definió cómo construimos el software.

E75 definió cómo lo desplegamos.

E76 define dónde y bajo qué infraestructura vive y opera ese software.

E74 — BUILD / CI/CD
        ↓
E75 — DEPLOYMENT
        ↓
E76 — RUNTIME INFRASTRUCTURE
        ↓
EVOXA RUNNING

La pregunta central de E76 es:

¿Qué infraestructura necesita EVOXA para ejecutar sus workloads de forma segura, escalable, observable y resiliente?

2. Principio Fundamental

La infraestructura de runtime debe ser una plataforma estable y predecible para la aplicación, sin absorber responsabilidades que pertenecen a EVOXA.

Por tanto:

Application
    ↓
Runtime Platform
    ↓
Infrastructure
    ↓
Physical / Cloud Resources

Cada capa tiene responsabilidades distintas.

3. Runtime Architecture

La arquitectura conceptual:

                         EVOXA
                           │
              ┌────────────┼────────────┐
              ▼            ▼            ▼
             API         Workers      Schedulers
              │            │            │
              └────────────┼────────────┘
                           ▼
                    Runtime Platform
                           │
             ┌─────────────┼─────────────┐
             ▼             ▼             ▼
          Compute       Network        Storage
             │             │             │
             └─────────────┼─────────────┘
                           ▼
                    Infrastructure
4. Runtime Layers

EVOXA tendrá conceptualmente estas capas:

L1 — Application
L2 — Workload
L3 — Runtime
L4 — Compute
L5 — Network
L6 — Storage
L7 — Infrastructure
L8 — Physical / Cloud Platform
5. Application Layer

Aquí viven los componentes definidos anteriormente:

API
Application Services
Workers
Schedulers
Projection Processors
Search Processors
Reporting Processes

La aplicación no debería conocer detalles innecesarios de infraestructura.

6. Workload Layer

Un workload representa una unidad de ejecución:

EVOXA API
EVOXA Worker
EVOXA Scheduler
EVOXA Processor

Cada workload debe declarar sus necesidades:

CPU
Memory
Network
Storage
Identity
Configuration
7. Runtime Layer

El runtime proporciona:

Process Execution
Isolation
Networking
Configuration
Secrets
Health
Resource Limits
Lifecycle Management

La tecnología concreta se decidirá según E68, pero la arquitectura debe ser independiente del proveedor.

8. Containerization

La arquitectura favorece workloads empaquetables como unidades reproducibles.

Conceptualmente:

EVOXA Application
      ↓
Container Image
      ↓
Runtime

Esto permite:

Consistency
Isolation
Portability
Reproducibility
9. Container Image

Una imagen debe contener:

Application
Runtime Dependencies
Required Libraries
Startup Definition

No debe contener:

Secrets
Environment-specific Credentials
Mutable Production State
10. Image Immutability

La imagen desplegada debe ser inmutable.

evoxa:1.8.3
      ↓
Immutable Image

Si cambia:

1.8.3
 ↓
1.8.4

No modificar silenciosamente la imagen anterior.

11. Runtime Orchestration

Si EVOXA utiliza múltiples workloads, se requiere una capa de orchestration para gestionar:

Scheduling
Placement
Scaling
Restart
Networking
Health
Deployment

Conceptualmente:

                  ORCHESTRATOR
                       │
        ┌──────────────┼──────────────┐
        ▼              ▼              ▼
      API            Worker       Scheduler
        │              │              │
        └──────────────┼──────────────┘
                       ▼
                   Compute
12. Orchestrator Responsibilities

El orchestrator no debe contener lógica de negocio.

Su función es:

Run
Schedule
Restart
Scale
Replace
Route
Monitor
13. Compute Architecture

La infraestructura de compute proporciona:

CPU
Memory
Process Execution
Network Interfaces

Puede estar basada en:

Virtual Machines
Containers
Managed Container Platform
Kubernetes
Serverless Containers

La elección concreta permanece alineada con E68.

14. Compute Abstraction

EVOXA debe evitar acoplarse innecesariamente a:

Specific Host
Specific VM
Specific IP
Specific Machine

Preferir:

Workload
   ↓
Runtime Scheduler
   ↓
Available Compute
15. Horizontal Scaling

Los workloads stateless deberán poder escalar horizontalmente:

             EVOXA API
                │
       ┌────────┼────────┐
       ▼        ▼        ▼
     API-1    API-2    API-3

Esto permite absorber aumentos de tráfico.

16. Vertical Scaling

También puede existir:

Small Instance
      ↓
Larger Instance

pero el scaling horizontal debe ser la estrategia principal cuando el workload lo permita.

17. Autoscaling

El runtime podrá ajustar automáticamente el número de instancias:

Low Load
   ↓
2 instances

High Load
   ↓
8 instances
18. Autoscaling Signals

No limitarse a CPU.

Puede utilizarse:

CPU
Memory
Requests/sec
Latency
Queue Depth
Concurrent Requests
Custom Metrics
19. Minimum / Maximum Capacity

Cada workload debe poder definir conceptualmente:

Minimum Instances
Maximum Instances
Scaling Target
Scale-up Policy
Scale-down Policy

Esto evita:

Underprovisioning

y:

Uncontrolled Cost
20. Resource Requests

Cada workload debe declarar una capacidad mínima esperada:

CPU Request
Memory Request

El scheduler utiliza esta información para colocar workloads correctamente.

21. Resource Limits

Cada workload debe tener límites:

CPU Limit
Memory Limit

Esto evita que un componente consuma todos los recursos disponibles.

22. Resource Isolation

Un workload defectuoso:

Worker A

no debería poder destruir:

API
Database
Scheduler

por consumo descontrolado de recursos.

23. Runtime Quality of Service

Los workloads críticos pueden recibir políticas diferentes:

Critical API
High Priority

Background Worker
Normal Priority

Batch Processing
Lower Priority
24. Availability Zones

Cuando la infraestructura lo permita, las instancias críticas deberán distribuirse entre zonas:

             EVOXA
               │
       ┌───────┼───────┐
       ▼       ▼       ▼
     Zone A  Zone B  Zone C

Evita que una única zona destruya toda la capacidad de servicio.

25. Failure Domain

La infraestructura debe reconocer diferentes dominios de fallo:

Process
Container
Host
Zone
Region
Provider

La resiliencia debe diseñarse teniendo en cuenta estos niveles.

26. Runtime Availability

La disponibilidad de EVOXA depende de:

Application
+
Compute
+
Network
+
Storage
+
Dependencies

No basta con tener múltiples instancias si todas dependen del mismo punto único de fallo.

27. Networking Architecture

La red de runtime se divide conceptualmente:

External Network
       ↓
Ingress
       ↓
Application Network
       ↓
Internal Services
       ↓
Data Network
28. Network Segmentation

Separar:

Public Access
Application Traffic
Administrative Traffic
Data Traffic

cuando la infraestructura lo permita.

29. Public Exposure

No todos los workloads deben ser públicos.

Preferencia:

Internet
   ↓
Ingress
   ↓
API

pero:

Internet
   ✕
Database
30. Private Services

Los componentes internos deberían permanecer en redes privadas:

API
 ↓
Internal Worker
 ↓
Database
31. East-West Traffic

El tráfico interno:

Service A
   ↓
Service B

debe estar sujeto a:

Authentication
Authorization
Network Policy
Observability

según el nivel de segmentación.

32. North-South Traffic

El tráfico externo:

Client
  ↓
Ingress
  ↓
EVOXA

requiere controles adicionales:

TLS
WAF / Edge Security
Rate Limiting
Authentication

cuando corresponda.

33. DNS

La infraestructura deberá proporcionar resolución estable de:

Public Domains
Internal Services
Environment Endpoints

Evitar depender de IPs estáticas cuando no sea necesario.

34. Load Balancing

Para múltiples instancias:

             Load Balancer
                   │
       ┌───────────┼───────────┐
       ▼           ▼           ▼
      API1        API2        API3

El balanceador debe evitar enviar tráfico a instancias no saludables.

35. Internal Service Routing

Internamente:

API
 ↓
Service Discovery
 ↓
Worker / Service

La ubicación física de la instancia debe ser transparente para el consumidor.

36. Network Policies

La red debe poder expresar:

API → Worker       ALLOW
API → Database     ALLOW
Internet → DB      DENY
Worker → DB        según necesidad

Principio:

Default deny cuando el contexto de seguridad lo permita.

37. Storage Architecture

El almacenamiento se divide conceptualmente:

Ephemeral Storage
Persistent Storage
Object Storage
Database Storage
Cache Storage

Cada uno tiene una función diferente.

38. Ephemeral Storage

Utilizado para:

Temporary Files
Scratch Data
Runtime Cache
Temporary Processing

Puede desaparecer cuando se reinicia un workload.

39. Persistent Storage

Utilizado cuando los datos deben sobrevivir al ciclo de vida del workload.

Workload
   ↓
Persistent Volume
40. Database Storage

La base de datos permanece fuera del lifecycle de los procesos de aplicación.

API Instance
   ↓
Database

Eliminar una API instance no elimina la base de datos.

41. Object Storage

Para:

Documents
Exports
Reports
Large Files
Archives
Backups

cuando corresponda.

42. Cache Storage

E17 define la arquitectura de caching.

Runtime debe proporcionar la infraestructura necesaria para:

Cache Nodes
Network
Persistence Policy
Availability
Scaling

pero el significado del cache pertenece a la aplicación.

43. Message Infrastructure

Para workloads asíncronos:

Producer
   ↓
Message Broker
   ↓
Consumer

El broker debe considerarse infraestructura crítica cuando EVOXA dependa de él.

44. Queue Isolation

Las colas pueden separarse por función:

Commands
Events
Background Jobs
Priority Jobs
Retry
Dead Letter

según la arquitectura de mensajería.

45. Worker Runtime

Un worker debe poder:

Consume
Process
Ack
Retry
Fail
Shutdown Gracefully
46. Worker Scaling

Los workers pueden escalar según:

Queue Depth
Processing Latency
CPU
Memory

Una métrica especialmente útil suele ser:

Queue Depth
47. Scheduler Runtime

Los schedulers requieren control de concurrencia:

Scheduler
   ↓
Job

Debe evitarse:

Duplicate Job Execution

cuando el job no sea idempotente.

48. Distributed Coordination

Cuando varios nodos comparten responsabilidad:

Node A
Node B
Node C

puede necesitarse:

Leader Election
Distributed Lock
Lease
Consensus

según el caso.

49. Runtime Identity

Cada workload debe tener una identidad propia:

EVOXA-API
EVOXA-WORKER
EVOXA-SCHEDULER

y permisos específicos.

50. Identity-Based Access

Preferir:

Workload Identity
       ↓
Authorization
       ↓
Resource

en lugar de compartir credenciales estáticas.

51. Secrets Delivery

El runtime puede obtener secretos:

Secret Manager
      ↓
Runtime
      ↓
Application

Los secretos no deberían almacenarse en imágenes.

52. Secret Rotation

La infraestructura debe permitir rotación:

Secret v1
   ↓
Secret v2
   ↓
Runtime Reload / Restart

sin tener que reconstruir el artefacto.

53. Certificate Management

La infraestructura debe facilitar:

TLS Certificates
Internal Certificates
Rotation
Expiration Monitoring

cuando sean necesarios.

54. Time Synchronization

Todos los nodos deben disponer de tiempo sincronizado.

Esto es importante para:

Logs
Tokens
Auditing
Events
Distributed Systems
55. Runtime Clock

EVOXA no debe depender de:

Local machine clock

para decisiones distribuidas críticas sin considerar sincronización y tolerancia temporal.

56. Runtime Logging

Cada workload debe emitir logs estructurados.

Application
   ↓
Structured Logs
   ↓
Log Aggregation
57. Runtime Metrics

Cada workload debe producir métricas relevantes:

CPU
Memory
Requests
Latency
Errors
Throughput
Queue Depth
58. Runtime Tracing

Para sistemas distribuidos:

Request
  ↓
API
  ↓
Service
  ↓
Database

debe poder correlacionarse mediante trace/context identifiers.

59. Runtime Observability

La plataforma debe poder responder:

Is EVOXA healthy?
Where is the bottleneck?
Which workload is failing?
Which version is running?
Which dependency is degraded?
60. Runtime Health Model

Cada workload debería proporcionar:

Startup Health
Readiness
Liveness
Metrics
Logs
61. Restart Policy

Si un proceso falla:

Process Crash
     ↓
Runtime Detects
     ↓
Restart

pero reinicios repetidos deben generar una señal crítica:

Crash Loop
62. Crash Loop Protection

No reiniciar indefinidamente sin observabilidad.

Debe detectarse:

Restart
Restart
Restart
Restart

y elevarse:

ALERT
63. Graceful Termination

El runtime debe proporcionar tiempo para:

Stop accepting traffic
Finish requests
Finish messages
Close connections
Flush telemetry
64. Resource Pressure

Si el nodo entra en presión:

Memory Pressure
CPU Pressure
Disk Pressure

el runtime debe aplicar políticas controladas.

65. Runtime Eviction

Si la plataforma necesita liberar recursos:

Low Priority Workload
       ↓
Eviction

Los componentes críticos deberán tener mayor protección cuando la plataforma lo soporte.

66. Capacity Architecture

EVOXA necesita capacidad suficiente para:

Normal Load
Peak Load
Failure Load
Deployment Load
Recovery Load
67. N+1 Capacity

Para componentes críticos:

N required
+
1 additional capacity

permite tolerar la pérdida de una instancia.

68. Failure Capacity

La capacidad no debe dimensionarse únicamente para:

Normal Traffic

sino también para:

Normal Traffic
+
One Failure

cuando el SLA lo requiera.

69. Deployment Capacity

Durante un rolling deployment pueden coexistir:

Old Version
+
New Version

Por tanto, la infraestructura debe tener capacidad suficiente para ambos temporalmente.

70. Cost Control

La infraestructura debe evitar:

Overprovisioning
Unbounded Scaling
Unused Resources
Duplicate Infrastructure

pero sin sacrificar los objetivos de disponibilidad.

71. Environment Sizing

Conceptualmente:

DEV
Small

STAGING
Production-like

PRODUCTION
Capacity-based

Staging no tiene que ser necesariamente idéntico en tamaño, pero sí suficientemente representativo para las pruebas importantes.

72. Infrastructure as Code

Toda infraestructura crítica debe declararse como código cuando sea posible:

Network
Compute
Runtime
Storage
IAM
Policies
Monitoring
73. Immutable Infrastructure

Preferencia:

Replace

sobre:

Manually mutate server

Conceptualmente:

Old Infrastructure
      ↓
New Definition
      ↓
New Infrastructure
74. Infrastructure Drift

Debe detectarse:

Declared State
      ≠
Actual State

Esto se conoce como:

Configuration Drift
75. Drift Management

Cuando existe drift:

Detect
 ↓
Assess
 ↓
Reconcile

No corregir automáticamente cambios críticos sin una política clara.

76. Runtime Isolation

Separar workloads mediante:

Namespaces / Services
Network Policies
Resource Limits
Identity

según la plataforma.

77. Tenant Isolation

Si EVOXA soporta múltiples tenants, la infraestructura debe reforzar:

Tenant Identity
Data Isolation
Access Control
Resource Policies

La seguridad de tenant sigue perteneciendo principalmente a la arquitectura de aplicación y datos.

78. Runtime Security

La infraestructura debe aplicar:

Least Privilege
Network Segmentation
Image Security
Runtime Isolation
Secret Protection
Audit
79. Host Security

Cuando exista infraestructura de host administrada por EVOXA:

OS Patching
Kernel Security
Minimal Packages
Access Control
Monitoring

deben formar parte del ciclo operacional.

80. Runtime Patch Strategy

Separar:

Application Update

de:

Infrastructure Patch

pero garantizar compatibilidad entre ambos.

81. Base Image Updates

Las imágenes base deben actualizarse regularmente:

Base Image v1
   ↓
Security Update
   ↓
Base Image v2
   ↓
Rebuild

No modificar una imagen productiva directamente.

82. Supply Chain

La infraestructura debe confiar únicamente en artefactos verificables:

Source
 ↓
Build
 ↓
Artifact
 ↓
Signature / Provenance
 ↓
Runtime

Esto conecta directamente con E74.

83. Runtime Admission

Antes de ejecutar un artefacto:

Artifact
   ↓
Policy Check
   ↓
Trusted?
   ↓
Run
84. Runtime Policy Enforcement

E20 puede imponer:

Allowed Images
Allowed Resources
Allowed Network
Allowed Capabilities
Allowed Deployments
85. Platform Boundary

El runtime platform debe controlar:

How code runs

pero no:

Why the business operation happens

Ejemplo:

Runtime:
"Este container tiene 2 CPU."

Application:
"Este pedido debe aprobarse."
86. Infrastructure Failure

Si un nodo falla:

Node Failure
   ↓
Runtime detects
   ↓
Workload rescheduled
   ↓
Health restored

si existe capacidad suficiente.

87. Zone Failure

Si una zona falla:

Zone A ✕
Zone B ✓
Zone C ✓

los workloads críticos deben continuar cuando la arquitectura de disponibilidad lo permita.

88. Region Failure

La tolerancia regional no será obligatoria para todos los componentes desde el primer deployment.

Debe evaluarse según:

Business Criticality
RTO
RPO
Cost
Data Architecture
89. Runtime Backup Boundary

El runtime no sustituye el backup de datos.

Runtime
 ≠
Backup

Los datos críticos deben gestionarse según E42.

90. Runtime Recovery

Ante una pérdida de infraestructura:

Infrastructure Failure
       ↓
Provision / Recover
       ↓
Deploy Artifact
       ↓
Restore Data if required
       ↓
Verify
91. Disaster Recovery Integration

E76 se conecta con:

E38 Resilience
E39 Fault Tolerance
E40 Recovery
E41 Disaster Recovery
E42 Backup & Restore
92. Runtime Topology

Arquitectura conceptual completa:

                         INTERNET
                            │
                            ▼
                         DNS / EDGE
                            │
                            ▼
                      LOAD BALANCER
                            │
                            ▼
                         INGRESS
                            │
              ┌─────────────┼─────────────┐
              ▼             ▼             ▼
             API          API            API
              │             │             │
              └─────────────┼─────────────┘
                            │
                 ┌──────────┴──────────┐
                 ▼                     ▼
              WORKERS              SCHEDULERS
                 │                     │
                 └──────────┬──────────┘
                            ▼
                  INTERNAL SERVICES
                            │
             ┌──────────────┼──────────────┐
             ▼              ▼              ▼
         DATABASE        CACHE         MESSAGE BUS
             │              │              │
             └──────────────┼──────────────┘
                            ▼
                     OBSERVABILITY
93. Runtime Infrastructure Control Plane

El control plane gestiona:

Scheduling
Deployments
Scaling
Health
Configuration
Policies
Identity

Mientras el data plane ejecuta:

API Requests
Jobs
Events
Queries
Business Operations
94. Control Plane vs Data Plane
              CONTROL PLANE
                    │
        ┌───────────┼───────────┐
        ▼           ▼           ▼
    Deploy       Scale       Policy
        │           │           │
        └───────────┼───────────┘
                    ▼
               DATA PLANE
                    │
        ┌───────────┼───────────┐
        ▼           ▼           ▼
       API        Workers     Services
95. Runtime Capacity Control

La plataforma debe poder responder:

How much capacity exists?
How much is being used?
How much is reserved?
How much is available?
How much can be added?

Esto conecta con E37 — Runtime Capacity Architecture.

96. Runtime SLO Alignment

La infraestructura debe dimensionarse a partir de:

Availability SLO
Latency SLO
Throughput
RTO
RPO

No únicamente por número de usuarios.

97. Infrastructure SLOs

Ejemplos conceptuales:

Compute Availability
Network Availability
Runtime Availability
Deployment Success Rate
Recovery Time
98. Runtime Alerting

Alertas para:

Instance Failure
Crash Loop
High Error Rate
High Latency
Capacity Exhaustion
Disk Exhaustion
Memory Pressure
Network Failure
Certificate Expiry
99. Infrastructure Observability Loop
Runtime
  ↓
Metrics / Logs / Traces
  ↓
Monitoring
  ↓
Alert
  ↓
Operator / Automation
  ↓
Remediation
100. Operational Principle

La infraestructura debe fallar de manera observable.

Un fallo silencioso es peor que un fallo explícito.

101. Runtime Lifecycle

Cada workload debe seguir:

CREATE
  ↓
START
  ↓
READY
  ↓
RUNNING
  ↓
DRAINING
  ↓
STOPPED
102. Runtime Lifecycle Management

El orchestrator debe poder:

Create
Start
Stop
Restart
Scale
Replace
Drain
103. Deployment Lifecycle
Artifact
   ↓
Scheduled
   ↓
Starting
   ↓
Ready
   ↓
Receiving Traffic
   ↓
Stable
104. Failure Lifecycle
Running
   ↓
Degraded
   ↓
Unhealthy
   ↓
Restart / Replace
   ↓
Recovering
   ↓
Healthy
105. E76 Architectural Decisions
AD-076-01
EVOXA runtime infrastructure is separated from application business logic.

AD-076-02
Application workloads should be packaged as reproducible deployable units.

AD-076-03
Runtime workloads should be independently schedulable where operationally justified.

AD-076-04
Stateless workloads should support horizontal scaling.

AD-076-05
Critical workloads should be distributed across independent failure domains when required.

AD-076-06
Resource requests and limits are required for managed workloads.

AD-076-07
Runtime health must expose startup, readiness and liveness semantics where applicable.

AD-076-08
Unhealthy workloads must be removed from traffic before termination.

AD-076-09
Graceful shutdown is required for workloads handling requests or messages.

AD-076-10
Public network exposure must be minimized.

AD-076-11
Internal services should use service discovery rather than hardcoded infrastructure addresses.

AD-076-12
Network access follows least privilege.

AD-076-13
Persistent data must survive application workload replacement.

AD-076-14
Secrets are injected at runtime and never embedded into application images.

AD-076-15
Runtime identities are preferred over shared static credentials.

AD-076-16
Infrastructure is managed through Infrastructure as Code where practical.

AD-076-17
Infrastructure drift must be detectable.

AD-076-18
Runtime artifacts must be traceable to immutable build artifacts.

AD-076-19
Critical runtime infrastructure must be observable.

AD-076-20
Runtime failures must produce actionable operational signals.

AD-076-21
Capacity planning must include normal, peak, deployment and failure scenarios.

AD-076-22
Runtime architecture must support controlled scaling.

AD-076-23
Infrastructure failure must not automatically imply application data loss.

AD-076-24
Runtime recovery must integrate with backup, restore and disaster-recovery architecture.

AD-076-25
Control-plane responsibilities must remain distinct from application data-plane responsibilities.
106. E76 — Arquitectura Consolidada

La visión completa de runtime queda:

                         EVOXA
                           │
                           ▼
                    DEPLOYED ARTIFACT
                           │
                           ▼
                    RUNTIME PLATFORM
                           │
          ┌────────────────┼────────────────┐
          ▼                ▼                ▼
       COMPUTE          NETWORK          STORAGE
          │                │                │
          ▼                ▼                ▼
      Containers       Ingress          Database
      Workers          Routing          Cache
      API              DNS              Object Storage
      Schedulers       Policies         Volumes
          │                │                │
          └────────────────┼────────────────┘
                           ▼
                    ORCHESTRATION
                           │
             ┌─────────────┼─────────────┐
             ▼             ▼             ▼
          Scaling        Health        Lifecycle
             │             │             │
             └─────────────┼─────────────┘
                           ▼
                     OBSERVABILITY
                           │
                           ▼
                  RESILIENCE / RECOVERY
107. Posición de E76 en EVOXA

Ya tenemos una secuencia muy importante:

E67  MASTER APPLICATION BLUEPRINT
 ↓
E68  TECHNICAL STACK & PLATFORM
 ↓
E69  PROJECT / REPOSITORY
 ↓
E70  MODULE & PACKAGE
 ↓
E71  DATABASE
 ↓
E72  API
 ↓
E73  TESTING
 ↓
E74  BUILD & CI/CD
 ↓
E75  DEPLOYMENT
 ↓
E76  RUNTIME INFRASTRUCTURE

Y ahora estamos entrando en la parte donde la aplicación deja de ser solamente código y pasa a ser un sistema operativo real.

El siguiente capítulo natural es:

E77 — EVOXA RUNTIME ENVIRONMENT ARCHITECTURE

E77 puede definir la siguiente capa con mucha más precisión:

Runtime Processes
Application Lifecycle
Process Model
Environment Variables
Runtime Configuration
Secrets Injection
Service Discovery
Health / Readiness / Liveness
Runtime Dependencies
Connection Management
Thread / Worker Model
Concurrency
Timeouts
Retries
Graceful Shutdown
Runtime Isolation
Execution Context
Resource Limits
Runtime Policies

La diferencia será:

E76 = infraestructura que sostiene EVOXA.

E77 = entorno de ejecución dentro del cual EVOXA vive y se comporta.
