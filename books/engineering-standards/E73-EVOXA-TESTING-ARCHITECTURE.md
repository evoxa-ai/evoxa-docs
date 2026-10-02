E73 — EVOXA TESTING ARCHITECTURE
1. Propósito

E73 define la arquitectura de pruebas de EVOXA.

Su objetivo no es simplemente decidir qué tests escribir, sino establecer un sistema que permita demostrar que EVOXA:

funciona correctamente;
respeta sus reglas de negocio;
mantiene sus contratos;
protege sus datos;
soporta carga;
tolera fallos;
puede evolucionar sin introducir regresiones;
y puede desplegarse con un nivel de confianza medible.

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
2. Principio Fundamental

Cada requisito arquitectónico importante debe tener una estrategia verificable.

Por tanto:

Requirement
    ↓
Design
    ↓
Implementation
    ↓
Test
    ↓
Evidence

Un componente no se considera terminado simplemente porque "funciona en desarrollo".

Debe existir evidencia de que cumple el contrato esperado.

3. Testing como Architecture

Testing no será una actividad posterior al desarrollo.

Forma parte de la arquitectura:

                EVOXA
                  │
       ┌──────────┼──────────┐
       ▼          ▼          ▼
   Production   Testing   Observability
                  │
        ┌─────────┼─────────┐
        ▼         ▼         ▼
      Unit    Integration   E2E
4. Quality Pyramid

EVOXA utilizará una estrategia basada en niveles:

                 E2E
                /   \
           Contract  UI
              /       \
       Integration   Security
            /           \
         Unit ─────────── Domain

La mayoría de las pruebas deben ser rápidas y deterministas.

5. Test Levels

La arquitectura contempla:

Unit Tests
Domain Tests
Application Tests
Integration Tests
Database Tests
API Tests
Contract Tests
Event Tests
Security Tests
Performance Tests
Load Tests
Resilience Tests
End-to-End Tests

Cada nivel tiene una responsabilidad diferente.

6. Unit Testing

Los Unit Tests verifican unidades aisladas.

Ejemplos:

Entity
Value Object
Domain Service
Mapper
Validator
Policy
Algorithm
Utility

Deben ser:

Fast
Deterministic
Isolated
Repeatable
7. Domain Testing

El dominio tendrá especial cobertura.

Por ejemplo:

Workflow
Policy
Decision
Execution
Resource
Rule

Las reglas críticas deben poder probarse sin:

HTTP
Database
Redis
Network
External APIs
8. Domain Test Example

Conceptualmente:

Given:
    Workflow is inactive

When:
    execute workflow

Then:
    execution is rejected

Esto valida comportamiento, no implementación.

9. Application Testing

Los Application Tests validarán casos de uso:

CreateResource
PublishWorkflow
ExecuteWorkflow
CancelExecution
EvaluatePolicy
CreateDecision

El foco será:

Input
    ↓
Use Case
    ↓
Expected Outcome
10. Application Boundary

El test puede sustituir dependencias:

Application Handler
       │
 ┌─────┼─────┐
 ▼     ▼     ▼
Fake  Mock  Stub
Repo  Event  Service

Esto permite probar orchestration sin infraestructura real.

11. Integration Testing

Integration Tests verifican que componentes reales funcionan juntos.

Ejemplos:

Application ↔ PostgreSQL
Application ↔ Redis
Application ↔ OpenSearch
Application ↔ Message Broker
API ↔ Application
Worker ↔ Queue
12. Database Integration Tests

Las pruebas de persistencia deben utilizar una base real o equivalente funcional.

No confiar únicamente en mocks de repositorios.

Debe verificarse:

Queries
Constraints
Indexes
Transactions
Mappings
Migrations
Concurrency
13. Migration Testing

Cada migración deberá probar:

Clean Database
     ↓
Migration N
     ↓
Expected Schema

Y cuando sea relevante:

Existing Production-like Data
     ↓
Migration
     ↓
Valid Data
14. API Testing

Las APIs se probarán desde el punto de vista del consumidor.

Ejemplo:

HTTP Request
      ↓
Authentication
      ↓
Validation
      ↓
Application
      ↓
HTTP Response

Se verificará:

Status code
Headers
Response schema
Validation
Authorization
Error behavior
Pagination
Idempotency
15. API Contract Testing

El contrato definido en E72 debe convertirse en una fuente verificable.

OpenAPI
   ↓
Contract Tests
   ↓
Implementation

La implementación no podrá desviarse silenciosamente del contrato.

16. Breaking Contract Detection

En CI:

API Contract
     ↓
Compare with previous version
     ↓
Breaking change?
     │
   ┌─┴─┐
  No  Yes
   │    │
   ▼    ▼
 Pass  Fail
17. Event Testing

Los eventos también son contratos.

Cada evento debe probar:

Event Type
Event Version
Schema
Required Fields
Semantics
Serialization
Consumer Compatibility
18. Event Contract

Ejemplo:

workflow.execution.completed

El test debe garantizar que una modificación interna no rompa consumidores existentes.

19. Serialization Testing

Relacionado directamente con E23.

Se probará:

Domain Object
      ↓
Serializer
      ↓
Payload
      ↓
Deserializer
      ↓
Equivalent Object

Especialmente importante para:

API DTOs
Events
Messages
Cache Objects
Persistence Models
20. Mapping Tests

E25 define mappings.

E73 verificará:

Domain
 ↓
DTO
 ↓
JSON

y:

Persistence
 ↓
Domain

evitando pérdida accidental de información.

21. Projection Testing

Para las proyecciones/read models:

Event
 ↓
Projection Handler
 ↓
Read Model

se comprobará que el estado resultante sea correcto.

22. Query Testing

Las queries deberán verificarse por:

Correctness
Filtering
Sorting
Pagination
Authorization
Performance

No basta con comprobar que devuelven algún resultado.

23. Security Testing

La seguridad será una dimensión transversal.

Se probará:

Authentication
Authorization
Tenant Isolation
Input Validation
Privilege Escalation
Injection
Data Exposure
Session Handling
Rate Limiting
Secrets Handling
24. Authentication Tests

Casos mínimos:

Valid token
Expired token
Invalid token
Wrong audience
Wrong issuer
Missing token
Malformed token
25. Authorization Tests

Debe comprobarse:

Allowed action
Denied action
Wrong role
Wrong tenant
Wrong resource owner
Insufficient permission
26. Tenant Isolation Testing

En sistemas multi-tenant:

Tenant A
   ↓
Request
   ↓
Must NOT access
   ↓
Tenant B data

Este tipo de test será crítico.

27. Security Regression Tests

Cuando se descubra una vulnerabilidad:

Security Bug
     ↓
Fix
     ↓
Regression Test
     ↓
Permanent Protection

Una vulnerabilidad corregida debe quedar representada por una prueba automatizada siempre que sea viable.

28. Validation Testing

Relacionado con E22.

Se probarán:

Required fields
Invalid formats
Boundary values
Invalid combinations
Malformed payloads
Unexpected values
29. Property-Based Testing

Para componentes matemáticos, reglas y transformaciones puede utilizarse:

Property-Based Testing

Ejemplo:

Para cualquier entrada válida, una transformación determinada conserva una propiedad específica.

Es especialmente útil en:

Rules
Mappings
Transformations
Calculations
Parsers
30. Boundary Testing

Las pruebas deberán concentrarse también en límites:

0
1
MAX
MAX + 1
Empty
Null
Missing
Very large
Very small

Los bugs suelen aparecer en los bordes, no en el caso nominal.

31. Negative Testing

No probar únicamente:

Happy Path

También:

Invalid input
Unauthorized request
Conflict
Timeout
Unavailable dependency
Duplicate request
Concurrent modification
Malformed event
32. Idempotency Testing

Relacionado con E72.

Debe comprobarse:

Request A
   ↓
Success

Request A again
   ↓
No duplicate side effect

Especialmente para:

Execution
Payments/integrations if later introduced
Commands
Webhooks
Event consumers
33. Concurrency Testing

Debe probarse comportamiento bajo operaciones simultáneas:

Request A ─┐
           ├── Resource
Request B ─┘

Casos:

Concurrent update
Duplicate creation
Race condition
Lock contention
Optimistic locking
34. Transaction Testing

Se comprobará:

Operation starts
      ↓
Failure
      ↓
Rollback

y:

Operation succeeds
      ↓
Commit

No debe quedar estado parcialmente aplicado cuando la operación requiere atomicidad.

35. Outbox Testing

Para el patrón Outbox:

Business Transaction
        │
        ├── State Change
        │
        └── Outbox Event

El test debe garantizar que ambas partes mantengan la consistencia esperada.

36. Worker Testing

Los workers deben probar:

Job received
Job processed
Job failed
Retry
Dead-letter
Duplicate delivery
Timeout
Cancellation
37. Retry Testing

Debe verificarse:

Attempt 1 → Failure
Attempt 2 → Failure
Attempt 3 → Success

y:

Max retries exceeded
      ↓
Dead Letter / Failed State
38. Failure Injection

Los tests de resiliencia pueden introducir fallos artificialmente:

Database unavailable
Redis unavailable
Message broker unavailable
Network timeout
External API failure

para comprobar que el sistema responde como fue diseñado.

39. Resilience Testing

Relacionado con E38–E42.

Debe comprobar:

Retry
Timeout
Circuit breaker
Failover
Recovery
Degraded mode

cuando estos mecanismos existan.

40. Performance Testing

Performance no significa únicamente "benchmark".

Debe medirse:

Latency
Throughput
Resource consumption
Concurrency
Database behavior
Cache behavior
41. Performance Metrics

Métricas principales:

p50
p95
p99
Throughput
Error rate
CPU
Memory
DB latency

No utilizar únicamente el promedio.

42. Load Testing

El Load Test simula carga esperada:

100 users
1,000 users
10,000 requests

según los objetivos reales de EVOXA.

Debe determinar:

Capacity
Bottlenecks
Saturation Point
43. Stress Testing

El Stress Test supera deliberadamente la capacidad prevista:

Normal Load
      ↓
Increasing Load
      ↓
Saturation
      ↓
Failure
      ↓
Recovery

Objetivo:

Conocer cómo falla EVOXA, no únicamente cuánto soporta.

44. Soak Testing

Para problemas acumulativos:

Long Duration
     ↓
Memory Leak
Connection Leak
Resource Exhaustion
Queue Growth

Es especialmente útil antes de producción.

45. End-to-End Testing

Los E2E validan flujos completos:

Client
 ↓
API
 ↓
Application
 ↓
Database
 ↓
Event
 ↓
Worker
 ↓
Result

Deben ser menos numerosos que los tests unitarios.

46. E2E Strategy

E2E se reservará para:

Critical User Journeys
Critical Business Flows
Cross-module workflows
Production-like integrations

No utilizar E2E para probar cada regla individual.

47. Test Data Architecture

Los tests necesitan datos controlados.

Tipos:

Fixtures
Factories
Builders
Seed Data
Synthetic Data
Production-like Data

Nunca depender de datos manuales inconsistentes.

48. Test Data Isolation

Cada test debe evitar depender accidentalmente de otro:

Test A
  ↓
Data A

Test B
  ↓
Data B

Preferiblemente:

Independent
Repeatable
Disposable
49. Database Test Isolation

Opciones:

Transaction rollback
Dedicated schema
Database reset
Containers
Ephemeral database

La estrategia concreta dependerá del stack de E68.

50. Test Doubles

Tipos:

Dummy
Stub
Fake
Mock
Spy

Usar cada uno de forma consciente.

51. Mocking Principle

No mockear indiscriminadamente.

Mala práctica:

Test
 ↓
Mock everything
 ↓
Everything passes

pero el sistema real puede fallar.

Preferir:

Unit → mocks/fakes where useful
Integration → real infrastructure
E2E → real system boundaries
52. Test Containers

Para dependencias reales puede utilizarse infraestructura efímera:

PostgreSQL Container
Redis Container
Message Broker Container
OpenSearch Container

Esto aumenta la fidelidad respecto a mocks.

53. Environment Strategy

EVOXA tendrá distintos niveles:

Local
 ↓
CI
 ↓
Development
 ↓
Staging
 ↓
Production

Cada entorno tiene objetivos diferentes.

54. Local Testing

Debe ser posible ejecutar una parte significativa de la suite localmente:

Unit
Domain
Application
Integration
API

sin depender obligatoriamente de servicios remotos.

55. CI Testing

Cada Pull Request deberá ejecutar una suite automática.

Conceptualmente:

Pull Request
     ↓
Build
     ↓
Lint
     ↓
Unit Tests
     ↓
Integration Tests
     ↓
Contract Tests
     ↓
Security Checks
     ↓
Quality Gate
56. Test Selection

No todos los tests necesitan ejecutarse siempre.

Clasificación:

Fast
Medium
Slow
Nightly
Release
57. Test Tags

Ejemplo:

unit
integration
api
contract
security
e2e
performance
slow

Esto permite seleccionar suites apropiadas.

58. Test Parallelization

Los tests independientes deberán poder ejecutarse en paralelo:

Test 1 ─┐
Test 2 ─┤
Test 3 ─┼──> CI
Test 4 ─┤
Test 5 ─┘

para reducir tiempo de feedback.

59. Flaky Tests

Un flaky test:

Pass
Fail
Pass
Fail

sin cambios en el código.

Esto será considerado un defecto de calidad.

No debe solucionarse simplemente con:

retry 10 times
60. Flaky Test Policy

Proceso:

Detect
 ↓
Quarantine if necessary
 ↓
Investigate
 ↓
Fix
 ↓
Return to main suite

Los flaky tests no deben normalizarse.

61. Code Coverage

Coverage será una métrica, no el objetivo final.

100% coverage

no garantiza:

100% correctness

Debe priorizarse:

Critical business logic
Security
Financial/data integrity
Infrastructure boundaries
62. Coverage Strategy

Medir:

Line coverage
Branch coverage
Critical path coverage
Mutation score

cuando sea apropiado.

63. Mutation Testing

Para componentes críticos:

Original Code
     ↓
Artificial Mutation
     ↓
Tests
     ↓
Should Fail

Si los tests siguen pasando, probablemente sean insuficientes.

64. Architecture Tests

E73 también debe validar arquitectura.

Ejemplos:

Domain must not depend on HTTP
Domain must not depend on PostgreSQL
API must not access repositories directly
Modules must respect boundaries
65. Dependency Rule Testing

Ejemplo:

Domain
  ✕ → Infrastructure

Application
  ✕ → HTTP framework internals

API
  ✓ → Application

Infrastructure
  ✓ → Domain/Application contracts

Esto evita erosión arquitectónica.

66. Repository Architecture Tests

Relacionado con E69/E70.

CI puede detectar:

Forbidden imports
Circular dependencies
Invalid module access
Layer violations
67. Database Architecture Tests

También pueden verificarse:

Required indexes
Foreign keys
Constraints
Migration consistency
Naming rules
68. API Architecture Tests

Automáticamente:

All endpoints documented
All errors follow schema
No undocumented public route
Version exists
Authentication applied where required
69. Security Automation

Pipeline:

Source
 ↓
SAST
 ↓
Dependency Scan
 ↓
Secret Scan
 ↓
Container Scan
 ↓
DAST where appropriate
70. Dependency Testing

Las dependencias deberán verificarse:

Known vulnerabilities
License constraints
Version policy
Transitive dependencies
71. Secret Detection

CI debe detectar accidentalmente committed:

API keys
Passwords
Private keys
Tokens
Cloud credentials
72. Test Observability

Cuando un test falle debe existir suficiente evidencia:

Test name
Environment
Correlation ID
Logs
Stack trace
Request/response where safe
Database state where appropriate
73. Failure Artifacts

CI puede conservar:

Logs
Screenshots
Traces
HTTP payloads
Test reports
Coverage
Performance reports

cuando corresponda.

74. Test Reporting

Cada pipeline deberá producir:

Passed
Failed
Skipped
Flaky
Duration
Coverage
Security findings
75. Quality Gates

Un build no podrá avanzar si falla una condición crítica.

Ejemplo:

Build
 ↓
Tests
 ↓
Security
 ↓
Contract
 ↓
Architecture
 ↓
Quality Gate
76. Critical Failures

Bloquean release:

Critical security vulnerability
Broken API contract
Critical test failure
Migration failure
Architecture violation
Data integrity failure
77. Non-Critical Findings

Pueden registrarse sin bloquear cuando estén dentro de una política controlada:

Low-risk warning
Non-critical technical debt
Known flaky test under quarantine

pero siempre con ownership.

78. Test Environment Parity

Staging deberá aproximarse a producción en:

Runtime
Database
Cache
Messaging
Configuration
Network topology
Security

cuando sea viable.

79. Production Verification

Después de deployment:

Deploy
 ↓
Health checks
 ↓
Smoke tests
 ↓
Critical API checks
 ↓
Observability

Esto permite detectar problemas que no aparecieron en CI.

80. Smoke Tests

Una suite pequeña verificará:

Application starts
Authentication works
Database available
Critical endpoint works
Critical workflow works

Debe ejecutarse rápidamente.

81. Canary Verification

Si posteriormente se utilizan canary deployments:

Small Traffic
      ↓
Smoke
      ↓
Metrics
      ↓
Increase Traffic

La estrategia de testing debe integrarse con E38 y las arquitecturas futuras de deployment.

82. Regression Testing

Cada bug importante debe seguir:

Bug
 ↓
Fix
 ↓
Regression Test
 ↓
Permanent Protection
83. Test Ownership

Cada área tendrá responsables:

Domain → Domain Team
API → API/Application Team
Infrastructure → Platform Team
Security → Security Ownership

El objetivo es evitar que los tests sean "de nadie".

84. Definition of Done

Una feature no estará terminada hasta que:

Implementation
✓
Unit Tests
✓
Integration Tests where needed
✓
API/Contract Tests where needed
✓
Security validation
✓
Documentation
✓
Observability
✓
No critical regressions
85. Test Matrix
Área	Unit	Integration	Contract	E2E	Security	Performance
Domain	✓	—	—	—	✓	—
Application	✓	✓	—	✓	✓	—
API	✓	✓	✓	✓	✓	✓
Database	—	✓	—	✓	✓	✓
Events	✓	✓	✓	✓	✓	✓
Workers	✓	✓	✓	✓	✓	✓
External integrations	✓	✓	✓	✓	✓	✓
86. Test Pyramid for EVOXA

La distribución conceptual:

                 ┌───────────┐
                 │    E2E    │
                 └─────┬─────┘
                       │
              ┌────────┴────────┐
              │ Contract/API    │
              │ Integration     │
              └────────┬────────┘
                       │
            ┌──────────┴──────────┐
            │ Application/Domain  │
            │ Unit Tests          │
            └─────────────────────┘

La base debe ser amplia y rápida.

87. Testing Lifecycle
Developer
   ↓
Local Tests
   ↓
Pull Request
   ↓
Fast CI
   ↓
Integration CI
   ↓
Security
   ↓
Contract
   ↓
Staging
   ↓
E2E
   ↓
Performance / Release
   ↓
Production Smoke
88. Test Automation Principle

Todo lo que sea:

Repetitive
Deterministic
Important
Machine-verifiable

debería automatizarse.

89. What Should Not Become Automated E2E

No utilizar E2E para:

Every validation rule
Every domain branch
Every mapper
Every edge case

Eso genera suites lentas y frágiles.

90. Testing Architecture Boundary

La arquitectura final:

                 EVOXA CODE
                     │
          ┌──────────┼──────────┐
          ▼          ▼          ▼
        Unit      Integration   E2E
          │          │          │
          └──────────┼──────────┘
                     ▼
              Quality Gates
                     │
        ┌────────────┼────────────┐
        ▼            ▼            ▼
     Security     Contracts    Architecture
        │            │            │
        └────────────┼────────────┘
                     ▼
               CI / Release
91. E73 Architectural Decisions
AD-073-01
Testing is part of EVOXA architecture, not a post-development activity.

AD-073-02
Unit tests provide the primary fast feedback layer.

AD-073-03
Critical domain rules must be testable without infrastructure.

AD-073-04
Integration tests validate real infrastructure boundaries.

AD-073-05
Public APIs are protected by contract tests.

AD-073-06
Events are treated as versioned testable contracts.

AD-073-07
Security testing is cross-cutting.

AD-073-08
Tenant isolation must be explicitly tested where multi-tenancy applies.

AD-073-09
Idempotency and concurrency require dedicated tests for critical operations.

AD-073-10
Long-running and asynchronous processing requires worker/retry/failure testing.

AD-073-11
Performance testing is separate from functional correctness testing.

AD-073-12
Architecture rules should be automatically verifiable.

AD-073-13
Flaky tests are defects and must not become normal behavior.

AD-073-14
Coverage is a signal, not the sole quality metric.

AD-073-15
Critical failures block quality gates.

AD-073-16
Production deployments require post-deployment verification.

AD-073-17
Important production bugs should become permanent regression tests.

AD-073-18
Test data must be controlled, isolated and reproducible.
92. E73 dentro de EVOXA

Con E73 queda cerrado un bloque fundamental:

E67  MASTER APPLICATION
      ↓
E68  TECHNICAL STACK
      ↓
E69  REPOSITORY
      ↓
E70  MODULES
      ↓
E71  DATABASE
      ↓
E72  API
      ↓
E73  TESTING

La consecuencia importante es que ya no solo sabemos cómo construir EVOXA; sabemos cómo verificar que lo que construimos cumple la arquitectura.

El siguiente bloque natural es:

E74 — EVOXA BUILD & CI/CD ARCHITECTURE

donde pasaremos de:

Código → Tests

a:

Código
  ↓
Build
  ↓
Tests
  ↓
Security
  ↓
Quality Gates
  ↓
Artifact
  ↓
Release
  ↓
Deployment

y ahí empieza a cerrarse la cadena que llevará el diseño de EVOXA hacia una aplicación realmente construible, versionable y desplegable.
