ADR-IDENTITY-064 — Authorization Criticality Policy Test Data Model, Machine-Readable Catalog & Evidence Schema

Estado: Proposed
Versión: 1.0
Fecha: 2026-09-07
Dominio: Identity / Authorization / Security / Policy Engine / Testing / Governance
Prioridad: Critical
Tipo: Architecture / Security / Testing / Data Model / Governance

1. Contexto

ADR-062 estableció el Authorization Criticality Policy Test Catalog, los Golden Scenarios y los Security Invariants.

ADR-063 estableció el Authorization Criticality Policy Test Execution Engine, responsable de:

ejecutar escenarios;
generar escenarios;
ejecutar Golden Scenarios;
evaluar invariants;
ejecutar property-based tests;
ejecutar boundary tests;
ejecutar mutation tests;
realizar differential testing;
comparar baseline y candidate;
clasificar regresiones;
producir evidencia;
proteger el Governance Gate.

Sin embargo, para implementar este mecanismo necesitamos definir exactamente cómo se representan los datos que el Engine consume y produce.

Actualmente existen conceptos como:

Scenario
Golden Scenario
Security Invariant
Test Case
Generated Scenario
Expected Result
Test Execution
Test Evidence
Regression Result
Baseline
Coverage Result
Mutation Result

pero todavía no existe un modelo machine-readable común.

Este ADR define ese contrato.

2. Problema

Si cada componente representa los tests de una manera diferente, aparecerán problemas de:

incompatibilidad;
duplicación;
pérdida de trazabilidad;
resultados no reproducibles;
dificultad para comparar versiones;
imposibilidad de automatizar Governance Gates;
dificultad para almacenar evidencia;
dificultad para ejecutar escenarios desde CI/CD;
riesgo de modificar silenciosamente resultados esperados.

Por lo tanto, necesitamos un modelo de datos común que permita:

Policy
   ↓
Test Catalog
   ↓
Scenario
   ↓
Execution
   ↓
Result
   ↓
Evidence
   ↓
Regression
   ↓
Governance
3. Decisión

Se establece un Authorization Criticality Test Data Model versionado y machine-readable.

El modelo tendrá cuatro grandes capas:

┌───────────────────────────────────────┐
│ 1. Definition                         │
│ Scenario / Golden / Invariant / Case  │
├───────────────────────────────────────┤
│ 2. Execution                           │
│ Test Run / Scenario Run / Evaluation  │
├───────────────────────────────────────┤
│ 3. Evidence                            │
│ Inputs / Outputs / Assertions / Hashes│
├───────────────────────────────────────┤
│ 4. Governance                          │
│ Regression / Coverage / Approval      │
└───────────────────────────────────────┘

El formato exacto de serialización podrá ser JSON como formato primario, con YAML como representación humana opcional.

La especificación definitiva de JSON Schema queda como decisión pendiente de implementación, pero la estructura conceptual queda fijada por este ADR.

4. Principios del modelo

El modelo deberá ser:

versionado;
determinista;
reproducible;
validable mediante schema;
extensible;
backward-compatible cuando sea posible;
auditable;
tenant-aware;
security-aware;
independiente de un framework de testing concreto.

No deberá depender de:

Python;
FastAPI;
pytest;
una base de datos específica;
Redis;
Kubernetes;
un proveedor cloud.

Estas son decisiones de implementación posteriores.

5. Identidad y versionamiento

Todo artefacto de testing deberá tener una identidad estable.

Conceptualmente:

artifact_id
artifact_type
artifact_version
schema_version

Ejemplo:

GSC-TENANT-001
version: 3
schema_version: 1.0

La identidad del escenario no debe cambiar simplemente porque cambien sus resultados esperados.

6. Artifact Types

Se establecen inicialmente:

POLICY_TEST_CATALOG
SCENARIO
GOLDEN_SCENARIO
SECURITY_INVARIANT
TEST_CASE
GENERATED_SCENARIO
TEST_EXECUTION
SCENARIO_EXECUTION
TEST_EVIDENCE
REGRESSION_RESULT
COVERAGE_RESULT
MUTATION_RESULT
BASELINE
7. Test Catalog

El Test Catalog será el contenedor lógico de escenarios e invariants.

Conceptualmente:

TestCatalog
 ├── catalog_id
 ├── version
 ├── schema_version
 ├── status
 ├── description
 ├── scenarios[]
 ├── golden_scenarios[]
 ├── invariants[]
 ├── profiles[]
 ├── metadata
 └── governance
8. Catalog Status

Se establece:

DRAFT
VALIDATING
APPROVED
ACTIVE
SUPERSEDED
RETIRED

El catálogo activo será el utilizado por defecto en validaciones correspondientes a su ámbito.

9. Scenario Model

Un Scenario representa una entrada reproducible para el Test Engine.

Conceptualmente:

Scenario
 ├── id
 ├── version
 ├── category
 ├── priority
 ├── description
 ├── tags
 ├── input_context
 ├── expected
 ├── invariants[]
 ├── preconditions
 ├── postconditions
 ├── golden
 ├── enabled
 └── metadata
10. Scenario Categories

Se utilizarán inicialmente las categorías definidas en ADR-062:

CORE
RESOURCE
ACTION
RISK
SECURITY_STATE
MFA
SESSION
TENANT
DELEGATION
POLICY_PRECEDENCE
FRESHNESS
CACHE
BOUNDARY
FAILURE
UNKNOWN
REGRESSION
PROPERTY
MUTATION
PERFORMANCE
CHAOS
RECOVERY

La lista podrá ampliarse mediante governance.

11. Scenario Priority
P0
P1
P2
P3
P4

Donde:

P0: invariantes y seguridad crítica.
P1: operaciones C0/C1 y controles de seguridad importantes.
P2: risk, precedence, freshness y cache.
P3: cobertura funcional normal.
P4: cobertura extendida.
12. Input Context

El input debe representar el contexto utilizado para evaluar Criticality.

Conceptualmente:

input_context:
  actor:
  organization:
  membership:
  request:
  resource:
  action:
  ownership:
  session:
  mfa:
  delegation:
  risk:
  security_state:
  freshness:
  environment:
  time:

No todos los escenarios necesitan todos los campos.

13. Actor Context

Ejemplo conceptual:

actor:
  type: USER
  id: user-123
  status: ACTIVE

También podrá representar:

SERVICE
AI_AGENT
SYSTEM

cuando corresponda.

14. Tenant Context

Debe distinguir:

organization
membership
tenant_state

Ejemplo:

organization:
  id: org-A

membership:
  id: membership-123
  status: ACTIVE

Un escenario cross-tenant deberá representarse explícitamente.

15. Resource Context

Conceptualmente:

resource:
  type: USER
  id: user-456
  organization_id: org-A
  sensitivity: R1

El modelo no deberá asumir un catálogo de recursos fijo dentro del ADR.

El Resource Registry será la autoridad.

16. Action Context

Conceptualmente:

action:
  name: user.update
  sensitivity: C1

La acción deberá resolverse contra el Action Registry.

No deberá permitirse que un escenario introduzca arbitrariamente una acción inexistente y la considere válida.

17. Risk Context

El modelo soportará:

risk:
  level: HIGH
  score:
  confidence:
  source:
  observed_at:

Los valores opcionales deberán estar controlados por el Risk Context contract.

18. Security State

Conceptualmente:

security_state:
  account: ACTIVE
  membership: ACTIVE
  session: ACTIVE
  mfa: VERIFIED
  delegation: ACTIVE

Los estados deberán ser evaluados por el Runtime y no simplemente tratados como strings libres.

19. MFA Context

Ejemplo:

mfa:
  required: true
  status: VERIFIED
  factor_type: TOTP
  verified_at:
  expires_at:

Los secretos de MFA nunca serán parte del escenario.

20. Session Context

Ejemplo:

session:
  status: ACTIVE
  authentication_method: PASSWORD
  security_epoch:
  created_at:
  expires_at:

Tokens reales no deberán incluirse.

21. Delegation Context

Ejemplo:

delegation:
  status: ACTIVE
  delegator_id:
  delegate_id:
  organization_id:
  allowed_actions:
  allowed_resources:
  expires_at:
  depth:

El modelo deberá permitir comprobar:

scope;
expiración;
tenant;
action;
resource;
depth;
privilege boundary.
22. Ownership Context

Conceptualmente:

ownership:
  owner_type: USER
  owner_id:
  relationship:

El escenario podrá representar:

OWNER
NON_OWNER
UNKNOWN

según el caso.

23. Freshness Context

Debe poder representar:

freshness:
  class: F1
  cache_state:
  ttl_valid:
  security_epoch:
  policy_version:
  entity_version:
  generation:
  invalidation_state:

Esto permite probar directamente ADR-056/057/058.

24. Expected Result

Cada escenario podrá definir:

expected:
  criticality:
  freshness_class:
  cacheability:
  security_behavior:
  decision_constraints:

Ejemplo conceptual:

expected:
  criticality: C0
  freshness_class: F0
  cacheability: false

Importante:

Criticality no equivale a Authorization Decision.

Por lo tanto, el modelo no debe asumir que:

C0 = DENY

ni:

C3 = ALLOW
25. Expected Security Behavior

Podrá especificar restricciones como:

security_behavior:
  cross_tenant: DENY
  stale_allow: FORBIDDEN
  unknown_security_state: FAIL_CLOSED
  privilege_amplification: FORBIDDEN

Estas restricciones serán evaluadas junto con los invariants.

26. Security Invariant Model

Un invariant representa una propiedad que debe mantenerse independientemente del escenario concreto.

Conceptualmente:

SecurityInvariant
 ├── id
 ├── version
 ├── name
 ├── category
 ├── description
 ├── expression
 ├── severity
 ├── scope
 ├── enabled
 └── metadata
27. Invariant Expression

La expresión debe utilizar la semántica del modelo de políticas.

Ejemplo conceptual:

EffectiveCriticality >= PlatformMinimum

o:

Tenant(actor) == Tenant(resource)

cuando la política de aislamiento así lo requiera.

No se permitirá código arbitrario como expresión.

28. Invariant Severity

Se establecen:

CRITICAL
HIGH
MEDIUM
LOW

Todo invariant relacionado con:

tenant isolation;
platform minimum;
privilege amplification;
unsafe ALLOW;
security state;

deberá ser como mínimo crítico o high según governance.

29. Golden Scenario

Un Golden Scenario es un Scenario con garantía de estabilidad contractual.

Conceptualmente:

golden: true

Un Golden Scenario sólo podrá cambiar mediante governance explícito.

Modificar su expected result silenciosamente estará prohibido.

30. Generated Scenario

Los escenarios generados deberán registrar su origen.

Conceptualmente:

generation:
  generator_version:
  strategy:
  seed:
  source_catalog_version:
  dimensions:

Ejemplo:

strategy: PAIRWISE
seed: 481923
generator_version: 2.1

Esto garantiza reproducibilidad.

31. Property Test

Un Property Test podrá utilizar:

property:
  id:
  expression:
  generator:
  constraints:
  iterations:
  seed:

La propiedad será evaluada sobre múltiples escenarios generados.

32. Mutation Test

El modelo deberá permitir describir una mutación.

Conceptualmente:

Mutation:
  id:
  target:
  operator:
  original:
  mutated:
  expected_detection:

Ejemplo:

operator: DOWNGRADE_SECURITY
original: C0
mutated: C1
expected_detection: true
33. Test Execution Model

Cada ejecución será un artefacto independiente.

TestExecution
 ├── execution_id
 ├── execution_version
 ├── catalog_version
 ├── policy_version
 ├── baseline
 ├── runtime_version
 ├── generator_version
 ├── scenario_selection
 ├── environment
 ├── seed
 ├── started_at
 ├── completed_at
 ├── result
 ├── severity
 └── evidence_reference
34. Scenario Execution

Cada Scenario ejecutado producirá:

ScenarioExecution
 ├── execution_id
 ├── scenario_id
 ├── scenario_version
 ├── input_hash
 ├── expected
 ├── actual
 ├── invariant_results
 ├── assertions
 ├── duration
 ├── result
 └── evidence_reference
35. Result States

Para cada test:

PASS
FAIL
BLOCKED
SKIPPED
ERROR
INFRASTRUCTURE_FAILURE

ERROR no equivale a PASS.

INFRASTRUCTURE_FAILURE no deberá transformarse automáticamente en PASS.

36. Evidence Model

La evidencia deberá demostrar qué ocurrió.

Conceptualmente:

TestEvidence
 ├── evidence_id
 ├── execution_id
 ├── scenario_id
 ├── policy_version
 ├── runtime_version
 ├── input_hash
 ├── expected_hash
 ├── actual_hash
 ├── assertions
 ├── invariant_results
 ├── timestamps
 ├── environment
 ├── trace_reference
 └── integrity
37. Evidence Minimization

La evidencia no deberá contener secretos.

Prohibido:

password
access_token
refresh_token
MFA_secret
recovery_code
private_key
KMS_secret

Cuando un dato sensible sea necesario para reproducibilidad, se utilizará:

identificador;
hash;
referencia;
valor anonimizado/sintético.
38. Evidence Integrity

La evidencia crítica deberá poder verificarse contra una representación íntegra.

Conceptualmente:

evidence_hash

y opcionalmente:

signature

La decisión definitiva sobre firma criptográfica queda pendiente.

39. Regression Result

El Regression Result será un artefacto explícito.

RegressionResult
 ├── regression_id
 ├── baseline
 ├── candidate
 ├── scenario_id
 ├── baseline_result
 ├── candidate_result
 ├── classification
 ├── severity
 ├── expected_change
 ├── rationale
 ├── invariant_impact
 └── evidence_reference
40. Regression Classifications

Se mantienen las categorías de ADR-063:

NO_CHANGE
EXPECTED_CHANGE
HARDENING
BEHAVIOR_CHANGE
SECURITY_REGRESSION
INVARIANT_VIOLATION
TEST_INFRASTRUCTURE_FAILURE
41. Baseline Model

El baseline deberá ser explícito.

Baseline
 ├── policy_id
 ├── policy_version
 ├── runtime_version
 ├── catalog_version
 ├── invariant_version
 ├── generator_version
 ├── created_at
 └── status

Esto evita comparar resultados incompatibles.

42. Coverage Result

La cobertura será multidimensional.

CoverageResult
 ├── execution_id
 ├── policy_version
 ├── scenario_coverage
 ├── resource_coverage
 ├── action_coverage
 ├── risk_coverage
 ├── mfa_coverage
 ├── session_coverage
 ├── tenant_coverage
 ├── delegation_coverage
 ├── freshness_coverage
 ├── invariant_coverage
 ├── precedence_coverage
 └── mutation_coverage

La métrica exacta de cada dimensión queda pendiente.

43. Machine-Readable Example

A continuación se establece una representación conceptual compatible con JSON.

{
  "schema_version": "1.0",
  "artifact_type": "GOLDEN_SCENARIO",
  "id": "GSC-TENANT-001",
  "version": 1,
  "category": "TENANT",
  "priority": "P0",
  "description": "Cross-tenant access must not reduce security requirements",

  "input_context": {
    "actor": {
      "type": "USER",
      "id": "user-a",
      "status": "ACTIVE"
    },

    "organization": {
      "id": "org-a"
    },

    "resource": {
      "type": "USER",
      "id": "user-b",
      "organization_id": "org-b"
    },

    "action": {
      "name": "user.read"
    },

    "risk": {
      "level": "LOW"
    },

    "security_state": {
      "account": "ACTIVE",
      "membership": "ACTIVE"
    }
  },

  "expected": {
    "criticality": "C0",
    "freshness_class": "F0",
    "cacheability": false,
    "security_behavior": {
      "cross_tenant": "DENY"
    }
  },

  "invariants": [
    "INV-CRITICALITY-003",
    "INV-CRITICALITY-004"
  ],

  "golden": true,
  "enabled": true
}

Este ejemplo es conceptual y no constituye todavía el JSON Schema definitivo.

44. Schema Versioning

Se diferenciarán:

artifact version

de:

schema version

Por ejemplo:

schema_version = 1.0
scenario_version = 8

El schema puede mantenerse estable mientras el escenario evoluciona.

45. Compatibility

Las modificaciones al schema deberán clasificarse como:

PATCH
MINOR
MAJOR
PATCH

Corrección compatible.

MINOR

Extensión compatible.

MAJOR

Cambio incompatible.

Un cambio MAJOR requerirá migración explícita.

46. Catalog Validation

Antes de utilizar un catálogo, deberá validarse:

Schema
Identity
References
Registry References
Expected Values
Invariant References
Policy References
Version Compatibility

Un catálogo inválido no podrá ejecutarse como catálogo activo.

47. Referential Integrity

Los siguientes elementos deberán resolverse correctamente:

scenario → invariant
scenario → resource
scenario → action
scenario → policy
scenario → freshness profile
scenario → criticality profile

Una referencia inexistente debe producir:

CATALOG_VALIDATION_ERROR

y no un resultado de test exitoso.

48. Immutability

Una vez que una ejecución haya finalizado:

TestExecution
ScenarioExecution
TestEvidence
RegressionResult

no deberán modificarse silenciosamente.

Las correcciones deberán generar una nueva versión o nuevo artefacto con trazabilidad.

49. Expected Result Governance

Los resultados esperados son parte del control de seguridad.

Por lo tanto:

Expected Result Change

requiere:

Change Request
+
Reason
+
Impact Analysis
+
Validation
+
Approval

No se permitirá:

test fails
→ change expected result
→ test passes

sin governance.

50. Evidence Retention

Los resultados relacionados con cambios de políticas críticas deberán conservarse conforme a las políticas de audit/retention establecidas para Identity.

La duración exacta deberá definirse en coordinación con:

ADR-013 — Data Retention;
ADR-028 — Observability;
ADR-030 — DR Testing;
governance de Identity.
51. CI/CD Artifact Flow

El modelo permitirá:

Git
 │
 ▼
Policy Change
 │
 ▼
Catalog Validation
 │
 ▼
Test Execution
 │
 ▼
Evidence
 │
 ▼
Regression Result
 │
 ▼
Coverage
 │
 ▼
Governance Gate
 │
 ├── BLOCK
 │
 └── APPROVE

Los artefactos deberán quedar vinculados al commit/release cuando corresponda.

52. Security Gate Data

El Governance Gate podrá consumir:

regression_result
invariant_results
coverage_result
mutation_result
test_execution_result

y evaluar:

PASS
REVIEW
BLOCK
53. Traceability Model

La trazabilidad deberá ser navegable:

Policy
  │
  ▼
Policy Version
  │
  ▼
Test Catalog
  │
  ├── Scenario
  ├── Golden Scenario
  └── Invariant
          │
          ▼
      Test Execution
          │
          ▼
      Scenario Execution
          │
          ▼
       Evidence
          │
          ▼
      Regression Result
          │
          ▼
     Governance Decision
          │
          ▼
       Deployment
54. Artifact Hashing

Para permitir reproducibilidad e integridad, podrá utilizarse hashing de:

Policy;
Scenario;
Catalog;
Input Context;
Expected Result;
Actual Result;
Evidence.

Conceptualmente:

policy_hash
scenario_hash
input_hash
expected_hash
actual_hash
evidence_hash

El algoritmo criptográfico exacto queda pendiente.

55. Synthetic Test Data

Los escenarios deben preferir datos sintéticos.

Por ejemplo:

user-a
org-a
resource-a
membership-a

en lugar de datos reales.

Esto reduce:

exposición de PII;
riesgo de seguridad;
dependencia de producción;
problemas de privacidad.
56. Tenant-Aware Test Data

Aunque los datos sean sintéticos, los escenarios deben conservar la semántica multi-tenant:

org-a
org-b
membership-a
membership-b

Esto es necesario para probar:

aislamiento;
cross-tenant;
delegation;
ownership;
platform administration.
57. Test Data Factories

La implementación podrá utilizar factories para crear:

User
Organization
Membership
Role
Permission
Session
MFA
Delegation
Resource
RiskContext
SecurityState

Pero estas factories deberán producir datos deterministas cuando se utilice un seed.

58. Privacy Boundary

Los artefactos de testing no deberán convertirse en un mecanismo alternativo para copiar datos de producción.

No se permitirá que un test:

Production User Data
        ↓
Test Evidence

sin controles explícitos.

59. Failure Evidence

Cuando una prueba falle, la evidencia deberá permitir responder:

qué policy se evaluó;
qué versión;
qué escenario;
qué inputs;
qué esperaba;
qué ocurrió;
qué invariant falló;
qué runtime se utilizó;
qué seed se utilizó;
si hubo infraestructura involucrada.
60. Deterministic Replay

Un escenario deberá poder reconstruirse utilizando:

Policy Version
Scenario Version
Catalog Version
Runtime Version
Generator Version
Seed
Input Hash

Cuando exista una ejecución generada, el Engine deberá poder regenerar el mismo caso.

61. AI-Generated Scenarios

Cuando AI proponga un escenario:

origin:
  type: AI_ASSISTED
  model:
  generated_at:
  proposal_reference:

El escenario no será considerado automáticamente Golden ni Approved.

Debe pasar por:

Validation
Testing
Governance
62. AI Governance

La IA podrá proponer:

scenarios;
invariants;
mutations;
coverage improvements.

Pero no podrá modificar silenciosamente:

expected result;
Platform Minimum;
invariant severity;
regression classification;
approval state.
63. Observability

Las ejecuciones deberán producir métricas como:

test_execution_count
test_pass_count
test_fail_count
invariant_violation_count
security_regression_count
scenario_generation_count
mutation_survival_count
coverage_percentage
execution_duration
flaky_test_count

Las métricas no sustituyen la evidencia de auditoría.

64. Non-Negotiables
Todos los artefactos críticos deben estar versionados.
Scenario IDs deben ser estables.
Expected Results deben estar gobernados.
Golden Scenarios requieren governance.
Security Invariants son obligatorios.
Unknown no puede interpretarse como safe allow.
Error no puede interpretarse como pass.
Tenant isolation debe estar representado.
Platform Minimum debe estar representado.
Criticality no es Authorization Decision.
Freshness debe poder representarse.
Cache state debe poder representarse.
Risk debe poder representarse.
MFA debe poder representarse.
Session debe poder representarse.
Delegation debe poder representarse.
Inputs deben ser reproducibles.
Generated scenarios deben registrar seed.
Evidence no puede contener secretos.
Execution results no deben modificarse silenciosamente.
Baselines deben ser explícitos.
Differential results deben ser trazables.
Regression classification debe ser machine-readable.
Security Regression debe bloquear.
Invariant Violation debe bloquear.
Mutation testing debe ser auditable.
AI-generated artifacts requieren governance.
AI no puede aprobar seguridad.
Production testing debe ser no destructivo.
El schema debe evolucionar mediante governance.
65. Consecuencias positivas

Este ADR proporciona:

contrato de datos común;
automatización;
reproducibilidad;
trazabilidad;
auditabilidad;
integración CI/CD;
comparación de políticas;
evidencia estructurada;
soporte para generación automática;
soporte para Golden Scenarios;
soporte para Security Invariants;
soporte para property/mutation testing;
separación entre definición y ejecución;
protección contra modificación silenciosa de tests.

Además, permite que el Test Execution Engine definido en ADR-063 opere sobre un modelo estable.

66. Consecuencias negativas

Introduce:

más artefactos versionados;
mayor governance;
necesidad de schema management;
almacenamiento de evidencia;
complejidad de compatibilidad;
necesidad de mantener referencias entre catálogos;
necesidad de mecanismos de migración.

Se considera aceptable debido al carácter crítico del sistema de autorización.

67. Alternativas rechazadas
67.1 Tests directamente en código

Rechazada.

Dificulta:

versionamiento independiente;
governance;
generación;
auditoría;
portability.
67.2 Solo JSON sin schema

Rechazada.

JSON por sí mismo no garantiza:

estructura;
tipos;
referencias;
compatibilidad.
67.3 Solo YAML

Rechazada como formato normativo.

YAML puede utilizarse como representación humana, pero JSON/schema será preferible para integración machine-readable.

67.4 Guardar solamente el resultado PASS/FAIL

Rechazada.

No permite reconstruir por qué se produjo el resultado.

67.5 Usar datos de producción como fixtures

Rechazada por seguridad y privacidad.

68. Dependencias

Este ADR depende de:

ADR-045 — Authorization Policy Model.
ADR-046 — Policy Schema/Governance.
ADR-048 — Policy DSL.
ADR-049 — Evaluation Runtime.
ADR-056 — Consistency/Freshness.
ADR-057 — Freshness Classes.
ADR-058 — Criticality Matrix.
ADR-059 — Criticality Policy Model.
ADR-062 — Test Catalog/Golden Scenarios/Invariants.
ADR-063 — Test Execution Engine.

También depende de:

Resource Registry;
Action Registry;
Attribute Registry;
Policy Registry;
Audit;
Observability;
CI/CD;
Governance.
69. Trazabilidad
Blueprint
Identity
 └── Authorization
      ├── Policy Engine
      ├── RBAC
      ├── ABAC
      ├── Tenant Isolation
      ├── Delegation
      ├── Security
      └── Testing / Governance
Implementation Stories
IS-IDENTITY-006
IS-IDENTITY-007
IS-IDENTITY-010
IS-IDENTITY-011
IS-IDENTITY-012
IS-IDENTITY-013
IS-IDENTITY-015
IS-IDENTITY-016
IS-IDENTITY-017
ADR chain
059
 ↓
062
 ↓
063
 ↓
064

ADR-062 define qué se prueba.

ADR-063 define cómo se ejecuta.

ADR-064 define cómo se representan los datos, resultados y evidencias.

70. Decisiones pendientes

Antes de implementación deberán especificarse:

JSON Schema definitivo;
YAML representation;
URI/versioning strategy;
exact enum catalogs;
exact Input Context Schema;
Resource Registry reference format;
Action Registry reference format;
Attribute Registry reference format;
Risk Context Schema;
MFA Context Schema;
Session Context Schema;
Delegation Context Schema;
Freshness Context Schema;
Security State Schema;
Expected Result Schema;
Invariant Expression Schema;
Property Test Schema;
Mutation Schema;
Test Execution Schema;
Evidence Schema;
Regression Schema;
Coverage Schema;
hashing algorithm;
signature mechanism;
evidence storage;
evidence retention;
schema migration mechanism;
compatibility rules;
artifact signing;
CI/CD artifact format;
test-result export format;
AI-generated artifact metadata;
production shadow evidence format;
multi-region evidence;
performance result schema.
71. Acceptance Criteria
AC-01

Debe existir un modelo machine-readable común para Criticality Policy Testing.

AC-02

Debe existir un Test Catalog versionado.

AC-03

Debe existir un Scenario Schema.

AC-04

Debe existir un Golden Scenario Schema.

AC-05

Debe existir un Security Invariant Schema.

AC-06

Debe existir soporte para Generated Scenarios.

AC-07

Los escenarios generados deben registrar seed y generator version.

AC-08

Debe existir un Test Execution Schema.

AC-09

Debe existir un Scenario Execution Schema.

AC-10

Debe existir un Evidence Schema.

AC-11

Debe existir un Regression Result Schema.

AC-12

Debe existir un Baseline Schema.

AC-13

Debe existir un Coverage Result Schema.

AC-14

Debe existir un Mutation Result Schema.

AC-15

Debe ser posible representar Tenant Context.

AC-16

Debe ser posible representar MFA, Session, Risk, Delegation y Freshness.

AC-17

No deben almacenarse secretos en los artefactos.

AC-18

Los resultados deben ser reproducibles.

AC-19

Las regresiones de seguridad deben ser machine-readable y bloqueables.

AC-20

La trazabilidad Policy → Scenario → Execution → Evidence → Regression → Governance debe mantenerse.

72. Resultado arquitectónico

Con este ADR queda establecida una separación clara:

ADR-059
¿Qué es Criticality Policy?
        ↓
ADR-062
¿Qué debemos probar?
        ↓
ADR-063
¿Cómo ejecutamos las pruebas?
        ↓
ADR-064
¿Cómo representamos los datos y la evidencia?

La arquitectura queda preparada para pasar posteriormente de conceptos a contratos machine-readable reales, sin saltar todavía a implementación.

73. Siguiente ADR recomendado

El siguiente paso lógico es:

ADR-IDENTITY-065 — Authorization Criticality Test Schema Versioning, Compatibility & Migration Strategy

Este ADR debería cerrar una pieza importante que dejamos pendiente en el 064: cómo evolucionarán los schemas y catálogos sin romper ejecuciones históricas, CI/CD, baselines, evidencia ni políticas existentes.

La cadena quedaría:

062 — Test Catalog / Golden / Invariants
          ↓
063 — Test Execution / Generation / Regression
          ↓
064 — Test Data Model / Machine-Readable / Evidence
          ↓
065 — Schema Versioning / Compatibility / Migration

Después de 065 ya estaríamos en una posición mucho más sólida para empezar a definir los contratos concretos (JSON Schema), estructuras de archivos y posteriormente Technical Tasks, antes de tocar código.
