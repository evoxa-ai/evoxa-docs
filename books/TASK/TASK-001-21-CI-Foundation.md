TASK-001-21 — CI Foundation

Identity Implementation Story: IS-001 — Identity Domain Foundation
ESP: ESP-0001/13 — Identity Testing & Quality Requirements
ESP relacionado: ESP-0001/14 — Identity Implementation Requirements
Dependencias: TASK-001-20 — Test Foundation
Prioridad: 🔴 Critical
Estado: 🟢 Ready for Implementation

1. Objetivo

Establecer la base de Continuous Integration (CI) de EVOXA Identity, de manera que cada cambio de código sea validado automáticamente antes de considerarse integrable.

La CI debe convertirse en una primera barrera de calidad:

Developer
    ↓
Git Commit
    ↓
Pull Request
    ↓
CI
    ├── Code Quality
    ├── Type Checks
    ├── Architecture Tests
    ├── Unit Tests
    ├── Coverage
    └── Security Checks
          ↓
       Quality Gate
          ↓
    Merge / Reject

El objetivo de este Task no es construir todavía todo el pipeline de CI/CD de EVOXA.

Aquí establecemos la Continuous Integration Foundation que posteriormente será ampliada con:

Build
Docker
Integration Tests
Security Scanning
Deployment
Migration Validation
Release
Staging
Production
2. Principios

La CI debe respetar los principios definidos para Identity:

CI-001 — Fail Closed

Si un quality gate crítico falla:

❌ Pipeline FAILED

No:

⚠️ Warning → merge igualmente

Especialmente para:

tests;
architecture tests;
security tests;
type checking;
coverage crítico.
CI-002 — Reproducibility

El mismo commit debe producir resultados reproducibles siempre que sea técnicamente posible.

Commit SHA
    ↓
same source
    ↓
same dependencies
    ↓
same tests
CI-003 — No Production Dependencies

La CI nunca debe utilizar:

❌ Production Database
❌ Production Redis
❌ Production secrets
❌ Production API keys
❌ Production credentials
CI-004 — Security First

Una optimización de velocidad nunca puede eliminar un control de seguridad obligatorio.

CI-005 — Traceability

Cada ejecución debe poder relacionarse con:

Repository
Commit
Branch
Pull Request
Pipeline
Test Result
Coverage
Artifact
3. Plataforma

El Blueprint/ESP establece GitHub Actions como baseline de CI/CD.

Por tanto, la implementación propuesta será:

GitHub
   ↓
GitHub Actions

La configuración inicial estará en:

.github/workflows/
4. Estructura propuesta
.github/
└── workflows/
    ├── ci.yml
    └── architecture.yml

Inicialmente podríamos mantenerlo incluso más simple:

.github/
└── workflows/
    └── ci.yml

y separar posteriormente cuando exista una necesidad real.

5. Pipeline inicial

La primera versión deberá seguir:

Checkout
   ↓
Python Setup
   ↓
Dependency Installation
   ↓
Lint
   ↓
Type Check
   ↓
Architecture Tests
   ↓
Unit Tests
   ↓
Coverage
   ↓
Quality Gate

Conceptualmente:

                    ┌─────────────┐
                    │   Commit    │
                    └──────┬──────┘
                           ↓
                    ┌─────────────┐
                    │   Checkout   │
                    └──────┬──────┘
                           ↓
                ┌─────────────────────┐
                │ Install Dependencies│
                └──────────┬──────────┘
                           ↓
              ┌─────────────────────────┐
              │   Quality Validation    │
              ├─────────────────────────┤
              │ Lint                    │
              │ Type Check              │
              │ Architecture            │
              │ Unit Tests              │
              │ Coverage                │
              └────────────┬────────────┘
                           ↓
                    ┌─────────────┐
                    │ Quality Gate│
                    └──────┬──────┘
                           ↓
                       PASS / FAIL
6. Trigger de CI

La CI debe ejecutarse principalmente en:

pull_request
push

Por ejemplo:

on:
  pull_request:
  push:
    branches:
      - main
      - develop

La configuración exacta de branches debe alinearse con la estrategia Git del proyecto cuando quede formalizada.

7. Pull Request Protection

La intención es que una Pull Request no pueda integrarse si los checks obligatorios fallan.

Conceptualmente:

Pull Request
     ↓
CI
     ↓
FAILED
     ↓
❌ Merge blocked

y:

Pull Request
     ↓
CI
     ↓
PASSED
     ↓
✅ Eligible for merge

La protección de branches será una configuración de repositorio, no únicamente código dentro de ci.yml.

8. Python Version

Debe existir una versión de Python definida de manera centralizada.

No debemos tener:

Local → Python 3.X
CI    → Python 3.Y
Production → Python 3.Z

sin una decisión explícita.

La versión deberá derivarse del baseline real del proyecto.

Si todavía no está congelada, se debe tratar como configuración pendiente y no inventarla dentro del Task.

9. Dependency Installation

La CI debe instalar dependencias desde los archivos oficiales del proyecto.

Por ejemplo, según la estrategia que adoptemos:

requirements.txt

o:

pyproject.toml

o el mecanismo de lock que posteriormente se adopte.

El objetivo es evitar que CI y desarrollo utilicen conjuntos de dependencias diferentes.

10. Dependency Reproducibility

Cuando se congele el mecanismo de dependencias, debemos garantizar:

same dependency definition
+
same lock state

para conseguir builds reproducibles.

No debemos depender de:

pip install latest

como estrategia permanente.

11. Lint

El pipeline deberá incluir una validación estática del código.

Conceptualmente:

Source Code
    ↓
Linter
    ↓
PASS / FAIL

La herramienta concreta puede establecerse durante la implementación.

El principio es:

errores de calidad de código detectables automáticamente no deben llegar a integración.

12. Type Checking

Como el proyecto utiliza Python con contratos tipados, debemos preparar:

Type Check

para detectar:

tipos incompatibles;
interfaces incorrectas;
retorno incorrecto;
Optional mal manejado;
dependencias incorrectas;
contratos inconsistentes.

La herramienta concreta puede ser definida posteriormente.

13. Architecture Tests

Este check es especialmente importante para EVOXA.

Debe ejecutar los tests creados en:

TASK-001-20 — Test Foundation.

Ejemplo:

pytest tests/architecture

Debe validar:

Domain
  ❌ → Infrastructure

Domain
  ❌ → FastAPI

Domain
  ❌ → Redis

Domain
  ❌ → SQLAlchemy

y las demás reglas arquitectónicas.

14. Unit Tests

La CI debe ejecutar automáticamente:

pytest tests/unit

Estos tests deben ser rápidos y no depender de infraestructura externa.

Objetivo:

fast feedback
15. Coverage

El pipeline deberá generar coverage:

pytest --cov=app

y producir un reporte.

Formatos posibles:

terminal
XML
HTML

El XML puede utilizarse posteriormente para integraciones con herramientas de quality reporting.

16. Coverage Gate

El ESP-0001/13 establece:

Overall Coverage ≥ 85%

y:

Critical Security Logic ≥ 95%

Pero debemos hacer una distinción importante.

Durante la fundación inicial:

TASK-001-20
+
TASK-001-21

no existen todavía todos los módulos críticos.

Por lo tanto, no debemos inventar una cobertura funcional inexistente.

La CI debe quedar preparada para aplicar los thresholds definitivos conforme los módulos sean implementados.

17. Security Test Gate

Los tests marcados como:

@pytest.mark.security

deberán poder ejecutarse separadamente.

Posteriormente:

pytest -m security

será un quality gate obligatorio para cambios de seguridad.

18. Test Markers

La CI debe reconocer los markers establecidos en TASK-001-20:

unit
integration
api
architecture
security
slow

Esto permitirá crear diferentes pipelines posteriormente:

FAST
STANDARD
FULL
19. FAST Pipeline

Para feedback rápido de Pull Requests:

Lint
Type Check
Architecture
Unit Tests
Basic Coverage

Objetivo:

developer feedback rápido
20. STANDARD Pipeline

Posteriormente:

Lint
Type Check
Architecture
Unit
Integration
API
Security
Coverage
21. FULL Pipeline

Para releases o cambios críticos:

Unit
Integration
API
Security
Contract
Architecture
Mutation
Property
Performance
Chaos
Recovery

TASK-001-21 solamente deja preparado el concepto.

No debemos implementar todavía todo el FULL pipeline.

22. Dependency Caching

La CI puede utilizar cache de dependencias para mejorar velocidad.

Pero:

Cache

no debe alterar el resultado de la ejecución.

Si existe corrupción:

cache invalidation
↓
fresh installation
23. Cache ≠ Source of Truth

Importante mantener coherencia con ADR-050/054/055:

La cache de GitHub Actions:

❌ no es source of truth

Es únicamente optimización del pipeline.

24. Secrets

GitHub Actions podrá utilizar secretos únicamente cuando sean necesarios.

Nunca:

echo $SECRET

ni:

logs
artifacts
test output

deben contener secretos.

Los secrets no deben escribirse en:

source code
configuration files
test fixtures
25. CI Environment

La CI debe utilizar configuración específica:

ENVIRONMENT=test

y recursos aislados.

Ejemplo conceptual:

CI
 ├── TEST_DATABASE
 └── TEST_REDIS

Nunca:

CI
 ↓
PRODUCTION_DATABASE
26. Integration Tests

Inicialmente pueden quedar separados del pipeline FAST.

Cuando se incorporen:

PostgreSQL
Redis

el workflow podrá utilizar servicios efímeros o containers.

Por ejemplo conceptualmente:

GitHub Runner
      │
      ├── PostgreSQL test
      │
      └── Redis test

No se debe depender de infraestructura permanente si no es necesario.

27. Migration Validation

Como TASK-001-19 estableció Alembic como baseline, posteriormente CI deberá validar:

migration syntax
migration ordering
fresh database
upgrade head

Pero la ejecución completa de migrations deberá integrarse cuando exista el esquema real de Identity.

28. Architecture Regression

Una característica importante de EVOXA será que la arquitectura también forma parte del código verificable.

Por ejemplo:

PR
 ↓
Architecture Test
 ↓
new forbidden dependency
 ↓
FAIL

Esto evita que la arquitectura se degrade silenciosamente con el tiempo.

29. Quality Gate

El Quality Gate inicial será:

                CI
                 │
       ┌─────────┼─────────┐
       ↓         ↓         ↓
     Lint      Types      Tests
       │         │         │
       └─────────┼─────────┘
                 ↓
          Architecture
                 ↓
             Coverage
                 ↓
          QUALITY GATE
           /          \
        PASS          FAIL
         ↓              ↓
      Merge          Block
30. Condiciones de FAIL

La CI debe fallar cuando:

❌ Unit test fails
❌ Architecture test fails
❌ Type check fails
❌ Required lint check fails
❌ Security test fails
❌ Critical quality gate fails

Y posteriormente:

❌ Tenant isolation failure
❌ Authorization regression
❌ MFA bypass
❌ Token security regression
❌ Unsafe ALLOW
❌ Critical freshness violation
❌ Security invariant violation

deben bloquear la integración.

31. No ignorar errores

No debemos utilizar:

pytest || true

para ocultar fallos.

Tampoco:

continue-on-error: true

para controles críticos.

Si una herramienta no es obligatoria todavía, simplemente no debe estar dentro del gate obligatorio.

32. Artifacts

La CI debería conservar resultados útiles:

test results
coverage.xml
coverage HTML
logs de CI

pero sin:

passwords
tokens
MFA secrets
private keys
credentials
33. Test Evidence

Esto prepara el terreno para ADR-063/064.

Posteriormente cada ejecución podrá asociarse a:

execution_id
commit_sha
policy_version
runtime_version
scenario_catalog_version
invariant_version
generator_version
result

En TASK-001-21 todavía no implementamos el Test Execution Engine completo.

Solo dejamos CI preparada para integrarlo.

34. Pull Request Checks

La PR debería mostrar algo conceptualmente parecido a:

EVOXA CI
──────────────────────
✓ Lint
✓ Type Check
✓ Architecture
✓ Unit Tests
✓ Coverage
──────────────────────
✓ Quality Gate

Si falla:

✗ Architecture

la PR queda bloqueada.

35. Branch Strategy

No debemos imponer todavía una estrategia Git que no esté formalmente definida.

Sin embargo, CI debe soportar al menos:

feature branch
      ↓
Pull Request
      ↓
develop/main

La estrategia exacta de branching queda como decisión de ingeniería del repositorio.

36. Commit Validation

Además de Pull Requests, la CI debe poder ejecutarse sobre commits directos en las ramas protegidas.

Esto evita que:

PR checks

se conviertan en la única protección.

37. Concurrency

GitHub Actions debe evitar ejecuciones innecesarias cuando múltiples commits son enviados rápidamente.

Conceptualmente:

PR #10 commit A
PR #10 commit B
PR #10 commit C

No necesariamente necesitamos mantener activas las tres ejecuciones.

La estrategia de cancelación debe configurarse sin cancelar pipelines que produzcan evidencia necesaria para releases.

38. Permissions

El workflow debe utilizar el principio:

Least Privilege

No debe tener:

permissions: write-all

si no es necesario.

Debe solicitar únicamente los permisos requeridos.

39. CI debe ser independiente del código de negocio

El pipeline:

.github/workflows/

no debe contener lógica de negocio de Identity.

Debe ejecutar:

tools
tests
scripts
commands

del proyecto.

40. Scripts reutilizables

Cuando sea conveniente, comandos importantes pueden estar centralizados:

scripts/
├── test.sh
├── test-unit.sh
├── test-integration.sh
└── quality.sh

Esto permite que:

Developer

y:

CI

ejecuten esencialmente los mismos comandos.

Por ejemplo:

./scripts/test.sh

en lugar de que la CI tenga una lógica completamente distinta.

41. Local = CI

Uno de los objetivos:

Developer
   ↓
./scripts/quality.sh
   ↓
PASS
   ↓
Push
   ↓
CI
   ↓
PASS

Esto reduce el clásico problema:

“En mi máquina funciona.”

42. Test Failure Diagnostics

Cuando un test falle, la CI debe conservar suficiente información para diagnosticarlo.

Debe mostrar:

test name
failure
stack trace seguro
duration
job
commit

pero nunca:

secrets
credentials
tokens
43. Timeout

Cada job debe tener límites razonables.

No queremos:

CI job
 ↓
hang
 ↓
6 hours

Los timeouts exactos se definirán según el pipeline real.

44. Observability de CI

La CI debe permitir conocer:

pipeline duration
test duration
failure rate
flaky tests
coverage trend

Esto será importante posteriormente para medir la calidad del proceso de ingeniería.

45. Flaky Tests

Un test que falla intermitentemente es un problema.

No debemos solucionar:

flaky test

simplemente con:

retry: 5

y considerar que está solucionado.

La estrategia correcta será:

detect
 ↓
classify
 ↓
investigate
 ↓
fix

Los retries podrán existir para determinados tests de infraestructura, pero nunca deben ocultar regresiones.

46. Security of CI

La CI forma parte de la superficie de ataque.

Debemos proteger:

GitHub Actions
Secrets
Artifacts
Dependencies
Workflow files
Runner

Especialmente:

Pull Request
    ↓
untrusted code
    ↓
CI
    ↓
secrets

No debemos exponer secretos sensibles a workflows que ejecutan código no confiable.

47. Dependency Security

Posteriormente CI deberá incorporar:

dependency vulnerability scanning

pero no es necesario bloquear TASK-001-21 esperando implementar todo el programa de seguridad de supply chain.

Debe quedar preparado como extensión.

48. Supply Chain

A futuro debemos considerar:

Source
 ↓
Dependencies
 ↓
Build
 ↓
Artifact
 ↓
Deployment

con controles de integridad.

Esto se incorporará progresivamente al pipeline de deployment.

49. Scope
Incluye
✓ GitHub Actions
✓ CI workflow
✓ Unit tests
✓ Architecture tests
✓ Type checking
✓ Lint
✓ Coverage
✓ Quality gate
✓ Test configuration
✓ CI test environment
✓ Basic artifacts
✓ PR validation
No incluye
❌ Production deployment
❌ Kubernetes deployment
❌ Docker release pipeline completo
❌ CD
❌ Production migrations
❌ Canary deployment
❌ Blue/Green deployment
❌ Production secrets
❌ Full security supply chain
❌ Full performance pipeline
❌ Full chaos pipeline
50. Acceptance Criteria
AC-01

Existe workflow de GitHub Actions para CI.

AC-02

CI se ejecuta en Pull Requests.

AC-03

CI se ejecuta en las ramas principales definidas.

AC-04

Las dependencias se instalan de manera reproducible.

AC-05

Lint está integrado.

AC-06

Type checking está integrado.

AC-07

Architecture tests están integrados.

AC-08

Unit tests están integrados.

AC-09

Coverage se genera automáticamente.

AC-10

Los fallos críticos hacen fallar el pipeline.

AC-11

Los tests no utilizan infraestructura de producción.

AC-12

Los secretos no aparecen en logs ni artifacts.

AC-13

Los workflows utilizan permisos mínimos.

AC-14

Los resultados de testing pueden ser inspeccionados.

AC-15

La CI puede ejecutarse sobre un commit reproducible.

AC-16

Existe separación conceptual FAST/STANDARD/FULL.

AC-17

La arquitectura puede bloquear una regresión.

AC-18

La CI puede evolucionar hacia integration/security/performance testing.

AC-19

Los comandos principales pueden ejecutarse localmente.

AC-20

Existe documentación mínima para ejecutar los checks localmente.

51. Definition of Done
[ ] .github/workflows creado
[ ] CI workflow funcionando
[ ] Pull Request trigger
[ ] Branch trigger
[ ] Python environment definido
[ ] Dependencies instaladas
[ ] Lint
[ ] Type checking
[ ] Architecture tests
[ ] Unit tests
[ ] Coverage
[ ] Quality gate
[ ] Test artifacts
[ ] Secret protection
[ ] Minimal permissions
[ ] CI documentation
[ ] Local/CI command alignment
[ ] Pipeline ejecutado exitosamente

Y especialmente:

[ ] un test fallido bloquea CI
[ ] una regresión arquitectónica bloquea CI
[ ] no se utilizan secretos de producción
52. Traceability
Blueprint
   ↓
Engineering Standards
   ↓
ESP-0001/13
   ↓
ESP-0001/14
   ↓
IS-001
   ↓
TASK-001-20
   ↓
TASK-001-21
   ↓
GitHub Actions CI

ADRs principales:

ADR-IDENTITY-010  Testing & Quality
ADR-IDENTITY-017  Deployment & Operations
ADR-IDENTITY-028  Observability
ADR-IDENTITY-030  DR Testing
ADR-IDENTITY-062  Golden Scenarios & Invariants
ADR-IDENTITY-063  Test Execution Engine
ADR-IDENTITY-064  Test Evidence
ADR-IDENTITY-065  Test Schema Versioning
ADR-IDENTITY-066  Test Schema Registry
53. Estado de IS-001
Task	Descripción	Estado
001-01	API Runtime Foundation	✅
001-02	Identity Domain Package	✅
001-03	Layer Architecture	✅
001-04	Shared Kernel	✅
001-05	Domain Events Foundation	✅
001-06	Repository Contracts	✅
001-07	Unit of Work	✅
001-08	PostgreSQL Infrastructure	✅
001-09	Redis Infrastructure	✅
001-10	Configuration & Secrets	✅
001-11	Security Foundation	✅
001-12	Tenant Context	✅
001-13	Actor Context	✅
001-14	Correlation & Trace Context	✅
001-15	Exception Model	✅
001-16	Dependency Injection	✅
001-17	Architecture Boundary Tests	🟢
001-18	Initial Observability	✅
001-19	Migration Framework	✅
001-20	Test Foundation	✅
001-21	CI Foundation	🟢 Ready
Próximo paso

Después de TASK-001-21, seguimos con:

TASK-001-22 — Identity Public Contracts

Aquí comenzaremos a dejar formalizados los contratos públicos de Identity que utilizarán posteriormente:

API
Application
Events
Other Domains
AI Agents
Mobile
Web
External Integrations

Este Task será especialmente importante porque conectará la arquitectura interna que acabamos de construir con las interfaces públicas estables de EVOXA Identity, antes de empezar con la implementación funcional de las Implementation Stories.
