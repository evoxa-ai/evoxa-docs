E18 — EVOXA Configuration Architecture
Engineering Specification

Pertenece a: Engineering Specification
Volumen: volume-09-engineering
Capítulo: E18 — Configuration Architecture
Anterior: E17 — Caching Architecture
Siguiente: E19 — EVOXA Feature Flag Architecture

1. Propósito

E18 define la arquitectura de configuración de EVOXA.

Su responsabilidad es establecer cómo EVOXA:

define configuración;
carga configuración;
valida configuración;
resuelve configuración;
distribuye configuración;
versiona configuración;
actualiza configuración;
aplica configuración por entorno;
aplica configuración por tenant;
protege secretos;
audita cambios;
recupera configuraciones anteriores.

La regla fundamental es:

La configuración controla el comportamiento del sistema; los secretos protegen el sistema; ninguno debe confundirse con código ni con estado de negocio.

2. Objetivos

La arquitectura debe soportar:

Environment Configuration
Application Configuration
Service Configuration
Module Configuration
Domain Configuration
Tenant Configuration
Runtime Configuration
Feature Configuration
Integration Configuration
Infrastructure Configuration
Security Configuration
Secret References
Dynamic Configuration
Static Configuration
Versioned Configuration
3. Configuration vs Code

Debe existir una separación clara:

Code
  │
  ├── Defines behavior
  │
  ▼
Configuration
  │
  └── Selects / tunes behavior

La configuración no debería convertirse en un lenguaje de programación oculto.

4. Configuration vs Business Data

Configuración:

"maximum_retry_count": 3

Estado de negocio:

"order_status": "paid"

Son conceptos diferentes.

La configuración no debe utilizarse como sustituto de una base de datos de dominio.

5. Configuration vs Secrets

Debe distinguirse:

Configuration
    │
    ├── Non-sensitive values
    │
    └── Secret References
             │
             ▼
        Secret Store

Nunca:

Configuration File
      │
      └── Plaintext Password
6. Configuration Hierarchy

La configuración puede resolverse mediante capas:

Global
  ↓
Environment
  ↓
Region
  ↓
Service
  ↓
Module
  ↓
Tenant
  ↓
Runtime

La precedencia debe ser explícita.

7. Configuration Resolution

Conceptualmente:

Default
   ↓
Environment Override
   ↓
Service Override
   ↓
Tenant Override
   ↓
Runtime Override
   ↓
Effective Configuration

El sistema debe poder determinar por qué un valor final fue seleccionado.

8. Effective Configuration

La aplicación consume:

Effective Configuration

no necesariamente el archivo o fuente original.

Ejemplo:

default:
  timeout = 30

production:
  timeout = 20

tenant-A:
  timeout = 10

Resultado:

tenant-A → timeout = 10
9. Configuration Sources

Las fuentes pueden incluir:

Static Files
Environment Variables
Configuration Service
Database
Secret Manager
Deployment Manifest
Infrastructure Parameters
Tenant Configuration Store
Runtime Control Plane

Cada fuente debe tener un propósito definido.

10. Configuration Authority

Cada configuración debe tener una fuente autoritativa.

Ejemplo:

Database Connection
        ↓
Deployment Configuration

API Secret
        ↓
Secret Manager

Tenant Preference
        ↓
Tenant Configuration Store

Debe evitarse tener múltiples fuentes igualmente autoritativas.

11. Static Configuration

Configuración estática puede cargarse durante startup:

Application Start
      ↓
Load Configuration
      ↓
Validate
      ↓
Build Runtime

Cambiarla puede requerir restart.

12. Dynamic Configuration

Configuración dinámica puede cambiar durante runtime:

Configuration Store
       ↓
Change
       ↓
Propagation
       ↓
Application

No toda configuración debe ser dinámica.

13. Dynamic Configuration Criteria

Una configuración debería ser dinámica sólo si:

Change Frequency
+
Operational Value
+
Safety
+
Consistency Requirements

justifican la complejidad.

14. Configuration Schema

Toda configuración estructurada debe tener un schema.

Conceptualmente:

Configuration
 ├── key
 ├── type
 ├── required
 ├── default
 ├── constraints
 └── description
15. Strong Typing

Debe preferirse:

timeout: integer
enabled: boolean
mode: enum

sobre:

timeout: "thirty"
enabled: "yes"
mode: "whatever"

La configuración debe validarse antes de entrar en el runtime.

16. Configuration Validation

Durante carga:

Load
 ↓
Parse
 ↓
Validate Schema
 ↓
Validate Constraints
 ↓
Resolve References
 ↓
Effective Configuration

Si una configuración crítica es inválida:

Application
    ↓
FAIL STARTUP

cuando continuar pueda producir comportamiento inseguro.

17. Validation Classes

Puede distinguirse:

Syntax Validation
Type Validation
Schema Validation
Semantic Validation
Security Validation
Dependency Validation
18. Syntax Validation

Ejemplo:

JSON/YAML/TOML

debe ser sintácticamente válido.

19. Type Validation

Ejemplo:

timeout = 30

es válido si espera un integer.

timeout = "fast"

debe rechazarse.

20. Semantic Validation

Una configuración puede ser sintácticamente válida pero semánticamente inválida.

Ejemplo:

min_connections = 100
max_connections = 10

Debe rechazarse.

21. Cross-Configuration Validation

También deben validarse dependencias:

feature_A = enabled
feature_A_endpoint = missing

La configuración efectiva debe ser coherente.

22. Configuration Defaults

Los defaults deben estar definidos explícitamente.

Ejemplo:

timeout:
  default: 30

Un default no debería quedar implícito en múltiples lugares del código.

23. Default Strategy

Los defaults deben ser:

Safe
Documented
Deterministic
Versioned
Tested
24. Environment Configuration

EVOXA puede tener:

Development
Test
Staging
Production

pero los entornos deben compartir el mismo modelo de configuración.

25. Environment Overrides

Debe evitarse duplicar completamente la configuración:

production.yaml
staging.yaml
development.yaml

si eso provoca divergencia.

Preferible:

Base Configuration
       +
Environment Overrides
26. Environment Isolation

Una aplicación de producción no debe accidentalmente cargar:

Development Configuration

Debe existir una identificación explícita del environment.

27. Region Configuration

En arquitecturas multi-region:

Global
  ↓
Region
  ↓
Service

pueden existir overrides regionales.

Ejemplo:

region = eu-west
28. Service Configuration

Cada servicio puede tener:

Service Defaults
Service Limits
Service Endpoints
Service Timeouts
Service Policies

pero la configuración debe permanecer dentro del boundary correspondiente.

29. Module Configuration

Los módulos pueden tener configuración específica:

Module
 ├── enabled
 ├── limits
 ├── policies
 └── dependencies

No deberían modificar arbitrariamente configuración de otros módulos.

30. Domain Configuration

Una parte de la configuración puede pertenecer a un dominio:

Identity
Billing
Orders
Organizations

La lógica de dominio debe consumir configuración mediante contratos claros.

31. Tenant Configuration

En un sistema multi-tenant:

Global Configuration
        ↓
Tenant Configuration
        ↓
Effective Tenant Configuration

Los tenants pueden tener configuraciones específicas cuando el producto lo permita.

32. Tenant Isolation

Una configuración de tenant:

Tenant A

no debe ser visible ni aplicable a:

Tenant B

La resolución debe incluir tenant context cuando corresponda.

33. Configuration Context

El contexto puede contener:

Environment
Region
Service
Module
Tenant
User
Runtime

No todos los parámetros deben depender de todos los contextos.

34. Context Explosion

Debe evitarse:

config(environment, region, service, module, tenant, user, ...)

para cada parámetro.

Cuantos más niveles de override existan, mayor es la complejidad de razonamiento.

35. Configuration Precedence

Debe existir una regla única.

Por ejemplo:

Default
<
Environment
<
Region
<
Service
<
Tenant
<
Runtime

El último valor aplicable gana.

La precedencia debe estar documentada y probada.

36. Configuration Immutability

Una vez cargada una configuración estática:

Config Snapshot

puede considerarse inmutable durante la vida del proceso.

Esto reduce comportamientos no deterministas.

37. Configuration Snapshot

Una instancia puede utilizar:

Configuration Snapshot v42

mientras otra instancia todavía utiliza:

Configuration Snapshot v41

durante una actualización gradual.

38. Configuration Versioning

Cada configuración dinámica debería poder tener:

version
created_at
created_by
source
status
39. Configuration Revision

Un cambio puede representarse:

v41
 ↓
v42

permitiendo:

Compare
Audit
Rollback
40. Configuration Rollback

Debe poder revertirse:

v42
 ↓
Rollback
 ↓
v41

especialmente para cambios operacionales.

41. Configuration Audit

Los cambios deben registrar:

Who
What
When
Why
Previous Value
New Value
Source

Los secretos no deben registrarse en plaintext.

42. Change Approval

Configuraciones sensibles pueden requerir:

Proposal
 ↓
Validation
 ↓
Approval
 ↓
Activation

según governance.

43. Configuration Deployment

Una estrategia segura:

Create
  ↓
Validate
  ↓
Stage
  ↓
Approve
  ↓
Deploy
  ↓
Observe
44. Progressive Configuration Rollout

Una configuración dinámica puede aplicarse gradualmente:

0%
 ↓
5%
 ↓
25%
 ↓
50%
 ↓
100%

Esto es especialmente útil para cambios de comportamiento.

45. Configuration Rollout vs Feature Flags

Debe mantenerse la distinción:

Configuration
    → parameterizes behavior

Feature Flag
    → controls feature exposure

E19 definirá específicamente feature flags.

46. Configuration Propagation

Cuando cambia una configuración:

Configuration Store
       ↓
Propagation
       ↓
Instance A
Instance B
Instance C

La arquitectura debe definir el mecanismo.

Puede ser:

Polling
Push
Event
Subscription
Restart
47. Configuration Polling

Una instancia puede consultar periódicamente:

Every N seconds
      ↓
Check Version
      ↓
Reload if Changed

Es sencillo pero añade tráfico y latencia de propagación.

48. Configuration Push

El sistema puede enviar:

ConfigurationChanged
        ↓
Application
        ↓
Reload

Permite menor latencia.

49. Configuration Event

E13 puede transportar:

ConfigurationChanged

pero el evento debe identificar la versión, no depender de que el consumidor reconstruya el estado desde información incompleta.

50. Configuration Consistency

La configuración distribuida puede tener:

Strong Consistency
Eventual Consistency

La elección depende del parámetro.

51. Safety-Critical Configuration

Algunas configuraciones requieren propagación controlada:

Security Policy
Authorization Policy
Rate Limit
Encryption Requirement

Un cambio parcial puede producir comportamiento inconsistente.

52. Configuration Activation

Una configuración puede tener estados:

DRAFT
VALIDATED
APPROVED
ACTIVE
SUPERSEDED
ROLLED_BACK
53. Configuration Lifecycle
CREATE
  ↓
VALIDATE
  ↓
APPROVE
  ↓
ACTIVATE
  ↓
OBSERVE
  ↓
SUPERSEDE
  ↓
ARCHIVE
54. Configuration Repository

Puede existir un repositorio de configuración:

Configuration Repository
       │
       ├── Schemas
       ├── Versions
       ├── Environments
       ├── Overrides
       └── Metadata
55. Configuration Store

El store puede estar respaldado por:

Database
Configuration Service
Object Store
Versioned Files

La tecnología es secundaria respecto al contrato.

56. Configuration API

Cuando exista un configuration service, su API puede proporcionar:

GET configuration
GET version
PUT configuration
VALIDATE configuration
ACTIVATE configuration
ROLLBACK configuration

El acceso debe estar protegido.

57. Configuration Access

Sólo actores autorizados pueden modificar configuración:

Operator
Admin
Deployment System
Automation

según la política.

58. Read Authorization

Leer configuración también puede requerir autorización.

Especialmente:

Security Configuration
Integration Configuration
Tenant Configuration
Secret References
59. Secrets

Los secretos deben vivir en un sistema especializado:

Secret Manager
     ↓
Secret Reference
     ↓
Configuration Resolver

No deben almacenarse directamente en repositorios de código.

60. Secret Reference

Ejemplo conceptual:

database.password:
  secret_ref: prod/database/password

La configuración contiene la referencia, no el secreto.

61. Secret Rotation

El sistema debe poder soportar:

Secret v1
   ↓
Secret v2
   ↓
Application Reload

sin exponer valores sensibles.

62. Configuration Encryption

Valores sensibles pueden requerir cifrado, pero la preferencia arquitectónica es:

No almacenar un secreto en configuración si puede almacenarse como referencia a un secret manager.

63. Environment Variables

Las environment variables pueden utilizarse para configuración de bootstrap:

ENVIRONMENT
SERVICE_NAME
CONFIG_ENDPOINT

pero no deben convertirse automáticamente en el único sistema de configuración de toda la plataforma.

64. Bootstrap Configuration

Algunas configuraciones son necesarias para encontrar el propio sistema de configuración:

Process
  ↓
Bootstrap Config
  ↓
Configuration Service
  ↓
Effective Config

Debe existir un conjunto mínimo de bootstrap configuration.

65. Bootstrap Dependency

Debe evitarse:

Configuration Service
   ↓
needs itself
   ↓
Configuration Service

El bootstrap debe ser suficientemente pequeño y autosuficiente.

66. Configuration Loading

Flujo recomendado:

Process Start
    ↓
Bootstrap Config
    ↓
Load Sources
    ↓
Merge
    ↓
Resolve References
    ↓
Validate
    ↓
Build Effective Config
    ↓
Freeze Snapshot
    ↓
Start Application
67. Fail Fast

Si falta una configuración crítica:

Missing Required Config
        ↓
Startup Failure

es preferible a:

Application starts
        ↓
Runtime failure later
68. Optional Configuration

La configuración opcional debe tener un default seguro.

Ejemplo:

telemetry.enabled
default = true
69. Configuration Errors

Los errores deben ser accionables:

Invalid configuration:
database.pool.max < database.pool.min

y no simplemente:

Configuration invalid
70. Configuration Diagnostics

El sistema debe poder mostrar:

Key
Effective Value
Source
Version
Last Updated

pero ocultando secretos.

71. Safe Configuration Inspection

Una herramienta de diagnóstico podría mostrar:

database.timeout = 30
source = production
version = 42

pero:

database.password = ********
72. Configuration Drift

Puede existir:

Desired Configuration
        ≠
Effective Configuration

Debe detectarse.

73. Configuration Drift Detection

Ejemplo:

Desired
  ↓
timeout = 30

Runtime
  ↓
timeout = 60

      ↓

DRIFT

Esto puede generar alertas.

74. Desired vs Observed

La arquitectura puede utilizar:

Desired State
       ↓
Configuration Controller
       ↓
Observed State

para sistemas dinámicos.

75. Configuration Reconciliation

Un controller puede asegurar:

Observed
   ↓
compare
   ↓
Desired
   ↓
Reconcile

Este patrón es especialmente útil en entornos distribuidos.

76. Configuration Health

Debe poder determinarse:

Configuration Loaded
Configuration Valid
Configuration Current
Configuration Consistent
Configuration Authorized
77. Configuration Metrics

Métricas mínimas:

configuration_load_total
configuration_load_errors
configuration_validation_errors
configuration_reload_total
configuration_reload_errors
configuration_version
configuration_propagation_latency
configuration_drift
78. Configuration Events

Eventos conceptuales:

ConfigurationCreated
ConfigurationValidated
ConfigurationActivated
ConfigurationChanged
ConfigurationRolledBack
ConfigurationExpired
79. Configuration Event Payload

Debe incluir:

configuration_id
version
scope
timestamp
actor
change_type

Nunca debe incluir secretos en plaintext.

80. Configuration Cache

E17 puede cachear configuración:

Configuration Store
       ↓
E17 Cache
       ↓
Application

Pero:

E17 acelera el acceso; E18 mantiene la semántica de configuración.

81. Configuration Refresh

Cuando cambia la configuración:

Change
 ↓
Invalidate / Refresh E17
 ↓
Resolve
 ↓
Validate
 ↓
Activate
82. Configuration and Scheduling

E16 puede programar cambios futuros:

Current Config
      ↓
Scheduled Activation
      ↓
Future Config

Ejemplo conceptual:

At 00:00
activate configuration v43
83. Configuration and Jobs

E15 puede ejecutar:

Configuration Validation Job
Configuration Reconciliation Job
Configuration Cleanup Job
84. Configuration and Workflows

E14 puede orquestar cambios complejos:

Draft
 ↓
Validate
 ↓
Approve
 ↓
Deploy
 ↓
Observe
 ↓
Rollback if needed
85. Configuration and Policies

E06 puede definir quién puede:

Read Configuration
Write Configuration
Approve Configuration
Activate Configuration
Rollback Configuration
86. Configuration and Authorization

E05 controla acceso:

Actor
  ↓
Authorization
  ↓
Configuration Operation
87. Configuration and Multi-Tenancy

La arquitectura debe soportar:

Global Config
Tenant Config
Tenant Override

sin mezclar:

Tenant A Config
      ✕
Tenant B Runtime
88. Tenant Configuration Lifecycle
Create Tenant
      ↓
Apply Defaults
      ↓
Tenant Configuration
      ↓
Overrides
      ↓
Effective Configuration
89. Tenant Defaults

Los defaults deben poder definirse globalmente:

Global Default
     ↓
Tenant A

y sobrescribirse sólo cuando sea necesario.

90. Configuration Templates

Puede utilizarse:

Template
   ↓
Tenant Configuration

para evitar duplicación.

91. Configuration Inheritance

La herencia debe ser limitada y explícita.

Demasiadas capas:

Global
 ↓
Region
 ↓
Environment
 ↓
Service
 ↓
Module
 ↓
Tenant
 ↓
User

pueden hacer extremadamente difícil determinar el valor final.

92. Configuration Transparency

Para cada valor debería ser posible responder:

What is the value?
Why is it this value?
Where did it come from?
Which version set it?
Who changed it?
93. Configuration Security

Debe proteger:

Secrets
Credentials
Private Endpoints
Encryption Keys
Security Policies
Tenant Configuration

según clasificación.

94. Configuration Integrity

La configuración crítica debe protegerse contra:

Unauthorized Modification
Tampering
Corruption
Rollback to Unsafe Version
95. Configuration Signing

Para configuraciones de alta sensibilidad puede utilizarse:

Configuration
      ↓
Sign
      ↓
Verify
      ↓
Activate
96. Configuration Backup

Las configuraciones versionadas deben poder recuperarse.

Version History
      ↓
Backup
      ↓
Restore
97. Disaster Recovery

Ante pérdida del configuration store:

Backup
  ↓
Restore
  ↓
Validate
  ↓
Reconcile
  ↓
Activate
98. Configuration Availability

Una aplicación debe poder definir qué ocurre si el configuration store deja de estar disponible:

Continue with Last Known Good
Fail Startup
Fail Closed
Degrade

según el tipo de configuración.

99. Last Known Good

Para configuración dinámica:

v41 Valid
   ↓
v42 Invalid

la instancia puede conservar:

Last Known Good = v41

en lugar de activar una configuración inválida.

100. Canonical Configuration Architecture
                         Configuration Control Plane
                                   │
                    ┌──────────────┼──────────────┐
                    ▼              ▼              ▼
                Schemas         Versions       Policies
                    │              │              │
                    └──────────────┼──────────────┘
                                   ▼
                          Configuration Store
                                   │
                                   ▼
                           Configuration Resolver
                                   │
             ┌─────────────────────┼─────────────────────┐
             ▼                     ▼                     ▼
        Environment              Region               Tenant
             │                     │                     │
             └─────────────────────┼─────────────────────┘
                                   ▼
                         Effective Configuration
                                   │
                    ┌──────────────┼──────────────┐
                    ▼              ▼              ▼
                 Service         Module         Runtime
                    │              │              │
                    └──────────────┼──────────────┘
                                   ▼
                              EVOXA Runtime
101. Integration Architecture
E06 — Policy
      │
      ▼
E05 — Authorization
      │
      ▼
E18 — Configuration
      │
      ├──────────► E17 — Cache
      │
      ├──────────► E13 — Events
      │
      ├──────────► E15 — Jobs
      │
      ├──────────► E16 — Scheduling
      │
      ├──────────► E14 — Workflow
      │
      └──────────► E19 — Feature Flags
102. Architectural Rules
Rule 1

Configuration must be separated from business state.

Rule 2

Secrets must be separated from ordinary configuration.

Rule 3

Every configuration value must have a defined owner.

Rule 4

Configuration precedence must be deterministic.

Rule 5

Invalid critical configuration must fail fast.

Rule 6

Dynamic configuration must be versioned.

Rule 7

Configuration changes must be auditable.

Rule 8

Rollback must be possible for operationally critical configuration.

Rule 9

Tenant configuration must respect tenant isolation.

Rule 10

Configuration inspection must never expose secrets.

Rule 11

Configuration propagation must be observable.

Rule 12

Runtime configuration must be distinguishable from desired configuration.

Rule 13

Last Known Good should be supported where dynamic configuration can fail.

Rule 14

E17 may cache configuration but does not own its semantics.

Rule 15

Configuration must not become an implicit programming language.

103. Definition of Done

E18 queda definido cuando EVOXA dispone de:

✓ Configuration Sources
✓ Configuration Authority
✓ Configuration Hierarchy
✓ Configuration Resolution
✓ Effective Configuration
✓ Configuration Context
✓ Environment Configuration
✓ Region Configuration
✓ Service Configuration
✓ Module Configuration
✓ Domain Configuration
✓ Tenant Configuration
✓ Bootstrap Configuration
✓ Static Configuration
✓ Dynamic Configuration
✓ Configuration Schema
✓ Strong Typing
✓ Schema Validation
✓ Semantic Validation
✓ Cross-Configuration Validation
✓ Configuration Defaults
✓ Configuration Precedence
✓ Configuration Snapshot
✓ Configuration Versioning
✓ Configuration Revision
✓ Configuration Rollback
✓ Configuration Audit
✓ Configuration Approval
✓ Configuration Deployment
✓ Progressive Rollout
✓ Configuration Propagation
✓ Polling
✓ Push
✓ Event-Based Updates
✓ Configuration Consistency
✓ Configuration Activation
✓ Configuration Lifecycle
✓ Configuration Repository
✓ Configuration Store
✓ Configuration API
✓ Configuration Authorization
✓ Secret References
✓ Secret Rotation
✓ Bootstrap Dependency
✓ Configuration Loading
✓ Fail Fast
✓ Configuration Diagnostics
✓ Safe Configuration Inspection
✓ Configuration Drift Detection
✓ Desired vs Observed State
✓ Configuration Reconciliation
✓ Configuration Health
✓ Configuration Metrics
✓ Configuration Events
✓ Configuration Cache
✓ Configuration Refresh
✓ Scheduled Activation
✓ Configuration Jobs
✓ Configuration Workflows
✓ Policy Integration
✓ Authorization Integration
✓ Multi-Tenant Configuration
✓ Configuration Templates
✓ Configuration Inheritance
✓ Configuration Transparency
✓ Configuration Security
✓ Configuration Integrity
✓ Configuration Signing
✓ Configuration Backup
✓ Disaster Recovery
✓ Configuration Availability
✓ Last Known Good
✓ E17 Integration
✓ E13 Integration
✓ E15 Integration
✓ E16 Integration
✓ E14 Integration
✓ E06 Integration
✓ E05 Integration
104. Position in Engineering Specification

La secuencia queda:

E14 — Workflow & Orchestration
        ↓
E15 — Job & Task Processing
        ↓
E16 — Scheduling
        ↓
E17 — Caching
        ↓
E18 — Configuration
        ↓
E19 — Feature Flags

La distinción importante es:

E17
Caching
   → "¿Cómo aceleramos el acceso?"

E18
Configuration
   → "¿Cómo determinamos el comportamiento configurable?"

E19
Feature Flags
   → "¿Cómo controlamos la exposición de capacidades?"

Siguiente capítulo: E19 — EVOXA Feature Flag Architecture.
