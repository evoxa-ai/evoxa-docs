E74 — EVOXA BUILD & CI/CD ARCHITECTURE
1. Propósito

E74 define cómo EVOXA pasa de código fuente a software verificable, empaquetado, versionado, liberado y desplegable.

E73 definió cómo comprobamos que el software funciona.

E74 define cómo automatizamos el camino:

SOURCE CODE
    ↓
BUILD
    ↓
TEST
    ↓
SECURITY
    ↓
QUALITY GATES
    ↓
ARTIFACT
    ↓
RELEASE
    ↓
DEPLOY
    ↓
VERIFY

La arquitectura queda:

E67 Master Application Blueprint
        ↓
E68 Technical Stack
        ↓
E69 Repository Architecture
        ↓
E70 Module Architecture
        ↓
E71 Database Architecture
        ↓
E72 API Architecture
        ↓
E73 Testing Architecture
        ↓
E74 Build & CI/CD Architecture
        ↓
E75 Deployment Architecture
2. Principio Fundamental

Ningún artefacto debe llegar a producción sin atravesar un proceso automatizado y reproducible de construcción, validación y promoción.

Por tanto:

Developer
   ↓
Git
   ↓
CI
   ↓
Validated Artifact
   ↓
Release
   ↓
CD
   ↓
Environment
3. CI/CD Definition
Continuous Integration

Cada cambio debe poder integrarse frecuentemente y validarse automáticamente.

Commit
  ↓
Build
  ↓
Tests
  ↓
Quality Gates
Continuous Delivery

El software debe quedar permanentemente en estado desplegable.

Validated Artifact
      ↓
Release Candidate
      ↓
Deployable
Continuous Deployment

Si EVOXA adopta deployment automático posteriormente:

Validated Artifact
      ↓
Production Deployment

La arquitectura soportará ambos modelos.

4. Git como Source of Truth

El repositorio será la fuente de verdad para:

Source Code
Configuration Templates
Infrastructure Definitions
Database Migrations
API Contracts
Tests
Build Definitions
CI/CD Definitions
Documentation

No debe existir una versión "especial" de producción mantenida manualmente fuera del repositorio.

5. Repository Flow
Developer
    ↓
Feature Branch
    ↓
Pull Request
    ↓
CI
    ↓
Code Review
    ↓
Merge
    ↓
Main
6. Main Branch

main debe representar:

El estado integrado y potencialmente desplegable de EVOXA.

No debe utilizarse como rama de desarrollo experimental permanente.

7. Branch Strategy

La estrategia inicial:

main
 │
 ├── feature/*
 ├── fix/*
 ├── refactor/*
 └── chore/*

Preferencia por ramas de vida corta.

8. Pull Request

Todo cambio relevante debe pasar por:

Pull Request
     ↓
Automated CI
     ↓
Review
     ↓
Approval
     ↓
Merge
9. CI Pipeline

Pipeline conceptual:

                 COMMIT
                    │
                    ▼
              SOURCE CHECK
                    │
                    ▼
                  BUILD
                    │
                    ▼
               UNIT TESTS
                    │
                    ▼
           INTEGRATION TESTS
                    │
                    ▼
             API CONTRACTS
                    │
                    ▼
              SECURITY
                    │
                    ▼
          ARCHITECTURE CHECKS
                    │
                    ▼
             QUALITY GATE
                    │
                    ▼
               ARTIFACT
10. Fast Feedback

No todos los checks deben esperar al final.

Primero:

Compile
Lint
Unit Tests
Type Checks

Después:

Integration
Contract
Security

Esto reduce el tiempo de feedback del desarrollador.

11. Pipeline Stages

La arquitectura recomienda separar:

1. Checkout
2. Dependency Resolution
3. Static Analysis
4. Build
5. Unit Testing
6. Integration Testing
7. Contract Testing
8. Security Testing
9. Packaging
10. Artifact Publication
12. Deterministic Builds

Un build debe ser reproducible.

Idealmente:

Same Source
+
Same Dependencies
+
Same Build Definition
=
Same Artifact

Esto reduce:

"It works on my machine."
13. Dependency Locking

Las dependencias deben controlarse mediante mecanismos de lock/version pinning apropiados al stack.

Evitar builds que cambien silenciosamente por:

Latest dependency
Latest image
Floating version
Uncontrolled transitive update
14. Build Environment

El build debe ejecutarse en un entorno controlado:

CI Runner
   ↓
Pinned Runtime
   ↓
Pinned Toolchain
   ↓
Pinned Dependencies
15. Build Artifact

El resultado del build será un artefacto inmutable.

Por ejemplo:

EVOXA Application
      ↓
Container Image

o el formato correspondiente al runtime definido en E68.

16. Artifact Principle

Se construye una vez y se promueve el mismo artefacto entre entornos.

No:

Build Dev
Build Staging
Build Production

Preferir:

Build Once
    ↓
Artifact
 ├── Dev
 ├── Staging
 └── Production
17. Artifact Immutability

Una vez publicado:

Artifact v1.8.3

no debe modificarse.

Si existe un cambio:

v1.8.3
   ↓
v1.8.4
18. Artifact Registry

Los artefactos deberán almacenarse en un registry controlado:

CI
 ↓
Artifact
 ↓
Artifact Registry

El registry será la fuente de artefactos desplegables.

19. Artifact Metadata

Cada artefacto debe poder asociarse con:

Version
Commit SHA
Build ID
Build Timestamp
Branch
Repository
Dependency Information
Security Scan
20. Software Bill of Materials

Los builds deberán poder producir un:

SBOM

para conocer:

Direct Dependencies
Transitive Dependencies
Versions
Components

Esto será importante para seguridad y supply-chain management.

21. Build Provenance

Debe poder responderse:

¿De dónde salió este artefacto?

Por ejemplo:

Production Artifact
       ↓
Version
       ↓
Commit
       ↓
Pull Request
       ↓
Tests
       ↓
Build
22. Versioning

EVOXA debe utilizar una estrategia de versionado consistente.

Preferencia:

Semantic Versioning

cuando el producto/librería lo requiera:

MAJOR.MINOR.PATCH
23. Version Semantics

Conceptualmente:

MAJOR
Breaking change

MINOR
Backward-compatible feature

PATCH
Backward-compatible fix

Las APIs tienen además el versioning definido en E72.

24. Commit Traceability

Cada deployment debe poder rastrearse hasta:

Deployment
 ↓
Artifact
 ↓
Commit
 ↓
PR
 ↓
Author
25. CI Security

El pipeline debe proteger:

Source
Secrets
Dependencies
Artifacts
Build Environment
Deployment Credentials
26. Secret Management

Nunca almacenar secretos directamente en:

Git
Source Code
Dockerfile
CI YAML
Logs

Los secretos deben provenir de un mecanismo de secret management.

27. CI Credentials

Principio:

Least privilege.

Un job de CI solo recibe los permisos necesarios para ejecutar su tarea.

28. Pull Request Security

Un PR no confiable no debe obtener automáticamente acceso privilegiado a secretos de producción.

Esto es especialmente importante cuando existen:

Forks
External Contributions
Untrusted Code
29. Static Analysis

CI ejecutará:

Linting
Type Checking
Code Quality
Architecture Checks

antes de considerar el código listo.

30. Dependency Security

El pipeline verificará:

Known CVEs
Outdated dependencies
Malicious packages
License constraints

según las políticas definidas.

31. Container Security

Si EVOXA utiliza containers:

Source
 ↓
Image Build
 ↓
Image Scan
 ↓
Registry

El scan deberá realizarse antes de permitir promoción.

32. Infrastructure Security

La infraestructura definida como código también debe verificarse:

IaC
 ↓
Static Analysis
 ↓
Security Validation
 ↓
Policy Checks
33. Quality Gates

El pipeline deberá detener la promoción si existe una condición crítica.

Ejemplo:

Build
  │
  ├── Tests ─────────────── FAIL → STOP
  │
  ├── Security ──────────── FAIL → STOP
  │
  ├── Contract ──────────── FAIL → STOP
  │
  └── Architecture ──────── FAIL → STOP
34. Quality Gate Categories
Functional
Security
Architecture
Performance
Compliance
Artifact Integrity
35. Test Gates

Como mínimo:

Unit Tests
Integration Tests
API/Contract Tests
Critical E2E

según el tipo de cambio.

36. Change-Aware Testing

No todos los cambios necesitan exactamente la misma estrategia.

Ejemplo:

Documentation-only
    ↓
Minimal CI

mientras:

Database Change
    ↓
Migration Tests
Integration Tests

y:

Authentication Change
    ↓
Security Suite
API Suite
E2E
37. Database Changes in CI

Cuando un PR contiene migraciones:

Build
 ↓
Create Test DB
 ↓
Apply Migrations
 ↓
Run Tests
 ↓
Validate Schema

Una migración inválida bloquea el pipeline.

38. API Changes in CI

Si cambia OpenAPI:

OpenAPI
 ↓
Lint
 ↓
Schema Validation
 ↓
Breaking Change Detection
 ↓
Contract Tests
39. Event Changes

Si cambia un evento:

Event Schema
 ↓
Compatibility Check
 ↓
Consumer Tests
 ↓
Integration Tests
40. Build Cache

Para acelerar CI:

Dependency Cache
Build Cache
Test Cache
Container Layers

pueden utilizarse.

Pero nunca debe sacrificarse la reproducibilidad.

41. Cache Invalidation

Los caches deben invalidarse cuando cambien:

Dependency Lock
Build Configuration
Compiler Version
Relevant Source
Base Image
42. Parallel CI

Los jobs independientes pueden ejecutarse en paralelo:

              Build
                │
       ┌────────┼────────┐
       ▼        ▼        ▼
     Unit    Security   Lint
       │        │        │
       └────────┼────────┘
                ▼
           Integration
43. Pipeline Optimization

Optimizar:

Fast feedback
Parallelism
Caching
Incremental builds
Selective testing
Artifact reuse

No optimizar eliminando controles críticos.

44. Release Candidate

Una vez superados los quality gates:

Validated Artifact
        ↓
Release Candidate

El RC debe ser exactamente el artefacto que potencialmente llegará a producción.

45. Release

Una release representa:

Version
+
Artifact
+
Changelog
+
Metadata
+
Evidence
46. Release Evidence

Cada release deberá conservar evidencia:

Build successful
Tests successful
Security checks
Artifact digest
Commit SHA
Approval
Deployment history
47. Environment Promotion

El flujo será:

Artifact
   ↓
Development
   ↓
Staging
   ↓
Production

El artefacto no se recompila en cada etapa.

48. Development Environment

Objetivo:

Fast feedback
Feature validation
Integration development

Puede tener mayor flexibilidad.

49. Staging Environment

Objetivo:

Production-like validation
E2E
Smoke
Integration
Performance
Release verification
50. Production Environment

Objetivo:

Controlled deployment
Minimal mutation
Strong access controls
Full observability
Rollback capability
51. Continuous Delivery Flow
Developer
    ↓
Git
    ↓
Pull Request
    ↓
CI
    ↓
Merge
    ↓
Build
    ↓
Artifact
    ↓
Registry
    ↓
Development
    ↓
Staging
    ↓
Approval
    ↓
Production
52. Continuous Deployment Option

Posteriormente:

main
 ↓
CI
 ↓
Quality Gates
 ↓
Artifact
 ↓
Staging
 ↓
Automated Verification
 ↓
Production

La arquitectura no obliga desde el inicio a deployment automático a producción.

53. Deployment Approval

Producción puede requerir:

Automated Gates
+
Human Approval

dependiendo del nivel de riesgo.

54. Progressive Delivery

EVOXA podrá evolucionar hacia:

Canary
Blue/Green
Rolling
Feature Flags

La elección concreta pertenece a E75/E76.

55. Rollback Principle

Debe existir una ruta clara:

Current
  ↓
New Release
  ↓
Failure
  ↓
Previous Artifact
56. Rollback vs Rollforward

No todo problema debe resolverse con rollback.

Rollback
→ volver al artefacto anterior

Rollforward
→ corregir y desplegar una nueva versión

La decisión dependerá del tipo de fallo.

57. Database Rollback

Especial cuidado.

Un deployment puede incluir:

Application Change
+
Database Migration

No siempre es seguro simplemente revertir ambos.

Por eso las migraciones deberán diseñarse preferentemente de manera compatible con deployments progresivos.

58. Expand / Contract

Para cambios complejos:

Expand
 ↓
Deploy Compatible Code
 ↓
Migrate Data
 ↓
Switch Usage
 ↓
Contract

Esto reduce riesgo durante releases.

59. CI/CD Observability

El pipeline también debe ser observable:

Build Duration
Queue Time
Failure Rate
Deployment Frequency
Lead Time
Rollback Rate
60. DORA Metrics

EVOXA puede utilizar métricas tipo:

Deployment Frequency
Lead Time for Changes
Change Failure Rate
Time to Restore

para medir la eficiencia real del proceso de entrega.

61. Pipeline Failure Classification

Cuando un pipeline falla, debe poder distinguir:

Code Failure
Test Failure
Infrastructure Failure
Dependency Failure
Security Failure
CI Failure
External Service Failure

Esto evita perder tiempo investigando la causa equivocada.

62. Retry Policy for CI

Reintentar automáticamente solo cuando la causa sea probablemente transitoria:

Network
Registry
Temporary Infrastructure

No ocultar:

Failing Tests
Compilation Errors
Security Findings
Contract Breaks

mediante retries.

63. Artifact Promotion

El registry puede representar estados:

Built
 ↓
Validated
 ↓
Release Candidate
 ↓
Production Approved
64. Artifact Signing

Para supply-chain security, los artefactos podrán firmarse:

Artifact
 ↓
Signature
 ↓
Registry
 ↓
Deployment Verification

Esto permite comprobar integridad y procedencia.

65. Deployment Policy

El sistema de deployment deberá verificar:

Artifact Exists
Artifact Trusted
Artifact Approved
Artifact Compatible
Environment Allowed

antes de ejecutar un release.

66. CI/CD Separation

Separar responsabilidades:

CI
 ↓
Build + Validate

CD
 ↓
Promote + Deploy

Esto simplifica seguridad y gobernanza.

67. Infrastructure as Code

La infraestructura utilizada por CI/CD deberá declararse como código cuando sea posible:

Repositories
CI configuration
Environments
Infrastructure
Policies
Deployment configuration
68. Environment Configuration

No construir diferentes binaries por entorno.

Preferir:

Same Artifact
+
Environment Configuration

Ejemplo:

Artifact X
   ├── DEV config
   ├── STAGING config
   └── PROD config
69. Configuration Separation

Los secretos y valores específicos del entorno no deben formar parte del artefacto.

Esto se conecta directamente con:

E18 Configuration
E20 Runtime Policy
E62 Governance Control Plane
70. Feature Flags

E19 se integra con CI/CD:

Deploy Code
   ↓
Feature OFF
   ↓
Validate
   ↓
Feature ON

Esto permite desacoplar:

Deployment

de:

Feature Release
71. Build vs Release

Importante:

Build
=
Crear artefacto

Release
=
Decidir qué versión se entrega

Deployment
=
Instalarla en un entorno

Son conceptos distintos.

72. CI/CD Architecture
                         GIT
                          │
                          ▼
                    PULL REQUEST
                          │
                          ▼
                         CI
                          │
        ┌─────────────────┼─────────────────┐
        ▼                 ▼                 ▼
      Build            Testing           Security
        │                 │                 │
        └─────────────────┼─────────────────┘
                          ▼
                   QUALITY GATES
                          │
                          ▼
                     ARTIFACT
                          │
                          ▼
                   ARTIFACT REGISTRY
                          │
             ┌────────────┼────────────┐
             ▼            ▼            ▼
            DEV        STAGING       RELEASE
             │            │            │
             └────────────┼────────────┘
                          ▼
                      PRODUCTION
                          │
                          ▼
                      VERIFY
                          │
                          ▼
                     OBSERVE
73. Developer Experience

El sistema debe permitir al desarrollador ejecutar localmente versiones equivalentes de:

Build
Lint
Unit Tests
Integration Tests
API Tests
Security Checks

cuando sea razonablemente posible.

74. Local → CI Consistency

Idealmente:

Local Command
     ≈
CI Command

Evitar:

Works locally
but CI uses completely different hidden logic.
75. Build Reproducibility

Debe poder reconstruirse un release histórico:

Commit SHA
+
Lockfiles
+
Build Definition
+
Toolchain
=
Historical Artifact
76. Disaster Recovery for CI/CD

El sistema de entrega también necesita resiliencia.

Debe existir capacidad de recuperar:

CI Configuration
Artifact Registry
Deployment Definitions
Release Metadata
Secrets References
77. Pipeline Availability

Si CI falla temporalmente:

Application Production

no debe quedar automáticamente afectada.

CI/CD es critical infrastructure, pero está separado del runtime de EVOXA.

78. Production Independence

Un fallo en:

CI

no debe apagar:

Production

Y un fallo en producción no debe corromper:

Source
Artifacts
Release Metadata
79. Auditability

Cada deployment deberá responder:

Who?
What?
When?
Which artifact?
Which commit?
Which environment?
Which approval?
What result?
80. Deployment Ledger

Se recomienda mantener historial:

Release
Artifact
Environment
Timestamp
Actor
Result
Rollback
81. Quality Dashboard

A nivel operativo se podrá visualizar:

Build Health
Test Health
Security Health
Release Health
Deployment Health
82. CI/CD Governance

Los pipelines críticos no deberán ser modificados sin control.

Cambios a:

Production deployment
Security gates
Artifact publishing
Credentials
Approval rules

requieren especial protección.

83. Branch Protection

main debe protegerse contra:

Direct unreviewed push
Failed CI
Missing approvals
Unsigned/invalid changes

según la política adoptada.

84. Release Governance

Una release debe cumplir:

Tests passed
Security passed
Contracts compatible
Artifact verified
Required approvals
Release metadata complete
85. CI/CD Anti-Patterns

EVOXA evitará:

Manual production builds
Manual copying of artifacts
"Latest" production tags
Secrets in repositories
Skipping tests to deploy
Different builds per environment
Unreviewed production deployments
Silent migration changes
86. The "Latest" Problem

No desplegar:

evoxa:latest

como referencia de producción.

Preferir:

evoxa:1.8.3

y/o digest inmutable:

sha256:...
87. Deployment Traceability

El runtime debe poder informar:

Application Version
Commit SHA
Artifact Digest
Build ID

Esto conecta directamente producción con CI/CD.

88. Release Verification

Después del deployment:

Deploy
 ↓
Health
 ↓
Smoke
 ↓
Metrics
 ↓
Error rate
 ↓
Critical workflows

Si falla:

Rollback / Halt / Investigate
89. CI/CD Security Boundary
Developer
   ↓
Git
   ↓
CI
   ↓
Artifact Registry
   ↓
Deployment System
   ↓
Production

Cada transición debe tener:

Authentication
Authorization
Audit
Integrity
90. E74 Architectural Decisions
AD-074-01
Git is the source of truth for application and delivery definitions.

AD-074-02
Main represents integrated, potentially deployable software.

AD-074-03
Pull requests require automated validation.

AD-074-04
Builds must be deterministic and reproducible.

AD-074-05
Dependencies must be controlled and locked.

AD-074-06
Artifacts are immutable.

AD-074-07
The same artifact is promoted across environments.

AD-074-08
CI validates code before artifact publication.

AD-074-09
Security scanning is part of CI/CD.

AD-074-10
Critical quality-gate failures block promotion.

AD-074-11
API and event contract changes are automatically validated.

AD-074-12
Database migrations are validated in CI.

AD-074-13
Build provenance must be traceable to source commit.

AD-074-14
Artifacts should carry version and build metadata.

AD-074-15
Secrets are never stored directly in source repositories.

AD-074-16
Production deployments require controlled authorization.

AD-074-17
Deployment history must be auditable.

AD-074-18
Production deployments must have a verification and recovery path.

AD-074-19
CI/CD failures must not directly compromise the running production system.

AD-074-20
Feature deployment and feature activation may be separated through feature flags.
91. E74 Complete Architecture

La cadena completa queda:

                         SOURCE
                           │
                           ▼
                      PULL REQUEST
                           │
                           ▼
                         REVIEW
                           │
                           ▼
                          CI
                           │
        ┌──────────────────┼──────────────────┐
        ▼                  ▼                  ▼
      BUILD             TESTING            SECURITY
        │                  │                  │
        └──────────────────┼──────────────────┘
                           ▼
                    QUALITY GATES
                           │
                           ▼
                      ARTIFACT
                           │
                           ▼
                    ARTIFACT REGISTRY
                           │
                  ┌────────┼────────┐
                  ▼        ▼        ▼
                 DEV    STAGING   RELEASE
                  │        │        │
                  └────────┼────────┘
                           ▼
                      PRODUCTION
                           │
                           ▼
                       VERIFY
                           │
                           ▼
                      OBSERVE
                           │
                           ▼
                    FEEDBACK LOOP
                           │
                           └──────→ DEVELOPMENT
92. Relación con los capítulos anteriores

Con E74 ya tenemos:

E71  DATABASE
       ↓
E72  API
       ↓
E73  TESTING
       ↓
E74  BUILD & CI/CD

Es decir:

Persist
   ↓
Expose
   ↓
Verify
   ↓
Build
   ↓
Release

Y ahora podemos dar el siguiente paso crítico: cómo ese artefacto validado llega físicamente a los entornos de EVOXA y cómo se ejecuta allí.

E75 — EVOXA DEPLOYMENT ARCHITECTURE

Ahí definiremos:

Deployment Model
Environment Architecture
Container/Runtime Deployment
Infrastructure
Networking
Ingress
Service Discovery
Rolling Deployment
Blue/Green
Canary
Configuration Injection
Secrets
Database Migration Deployment
Health Checks
Readiness/Liveness
Autoscaling
Rollback
Zero-Downtime Deployment
Release Strategies
Production Promotion
Deployment Security

Con E74 terminamos la arquitectura de construcción y entrega; con E75 empezamos la arquitectura de ejecución y despliegue real de EVOXA.
