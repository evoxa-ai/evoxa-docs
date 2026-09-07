ADR-IDENTITY-065 — Authorization Criticality Test Schema Versioning, Compatibility & Migration Strategy

Estado: Proposed
Versión: 1.0
Fecha: 2026-09-07
Dominio: Identity / Authorization / Security / Policy Engine / Testing / Governance
Prioridad: Critical
Tipo: Architecture / Security / Testing / Schema Governance / Compatibility / Migration

1. Contexto

Los ADR anteriores establecieron progresivamente la arquitectura de validación de Authorization Criticality Policies:

ADR-059
Criticality Policy Model
        ↓
ADR-062
Test Catalog / Golden Scenarios / Security Invariants
        ↓
ADR-063
Test Execution / Scenario Generation / Regression
        ↓
ADR-064
Test Data Model / Machine-Readable Catalog / Evidence

ADR-064 definió que los artefactos de testing serán machine-readable y versionados, incluyendo:

Test Catalog;
Scenario;
Golden Scenario;
Security Invariant;
Test Case;
Generated Scenario;
Test Execution;
Scenario Execution;
Test Evidence;
Regression Result;
Coverage Result;
Mutation Result;
Baseline.

Sin embargo, estos artefactos evolucionarán.

Por ejemplo, en una versión futura podríamos necesitar:

Scenario v1
    ↓
Scenario v2
    ↓
Scenario v3

o:

Test Evidence Schema 1.0
    ↓
Test Evidence Schema 2.0

Sin una estrategia explícita, una modificación de schema podría:

romper CI/CD;
invalidar catálogos anteriores;
impedir reproducir ejecuciones históricas;
romper baselines;
hacer ilegible evidencia antigua;
alterar el significado de resultados;
generar falsos positivos o negativos;
dificultar auditorías;
permitir migraciones inseguras.

Por lo tanto, se necesita una política formal para versionar, compatibilizar, migrar y retirar schemas.

2. Problema

El sistema necesita distinguir claramente entre:

Artifact Version
Schema Version
Policy Version
Runtime Version
Catalog Version
Scenario Version
Invariant Version
Generator Version

Estas versiones tienen significados diferentes.

Por ejemplo:

Policy v12
Scenario v8
Schema v1.2
Runtime v4

no representa lo mismo que:

Policy v13
Scenario v8
Schema v1.2
Runtime v4

Asimismo, cambiar el schema no significa necesariamente cambiar el comportamiento esperado.

La arquitectura debe evitar que una migración técnica sea interpretada como una modificación funcional o de seguridad.

3. Decisión

Se establece una estrategia formal de:

Independent Artifact Versioning + Schema Semantic Versioning + Explicit Compatibility + Non-Destructive Migration

Cada tipo de artefacto mantendrá su propio versionamiento, mientras que el schema tendrá una versión independiente.

La estrategia seguirá estos principios:

los artefactos históricos deben permanecer reproducibles;
los schemas nuevos no deben alterar silenciosamente el significado de datos antiguos;
los cambios incompatibles requieren una nueva versión mayor;
las migraciones deben ser explícitas;
los artefactos históricos no se sobrescriben;
los resultados de seguridad no se reinterpretan silenciosamente;
un schema antiguo debe seguir siendo legible mientras exista evidencia que dependa de él;
los cambios de schema deben pasar por governance;
las migraciones deben ser deterministas;
una migración nunca debe reducir requisitos de seguridad.
4. Separación de Versiones

Se establece formalmente la separación:

Schema Version
Artifact Version
Policy Version
Runtime Version
Catalog Version
Generator Version
Invariant Version
Baseline Version
5. Schema Version

El schema_version representa la estructura y semántica del formato machine-readable.

Ejemplo:

1.0
1.1
1.2
2.0

No representa una nueva versión de un escenario.

6. Artifact Version

El artifact_version representa la evolución del objeto específico.

Ejemplo:

Scenario:
id = GSC-TENANT-001
version = 4

El schema podría seguir siendo:

schema_version = 1.2

Por tanto:

GSC-TENANT-001 v4
Schema v1.2

es perfectamente válido.

7. Policy Version

Una Criticality Policy tiene su propio lifecycle/version.

Ejemplo:

Policy CRIT-POL-001
v10
v11
v12

Un cambio de policy no implica necesariamente cambio de schema.

8. Runtime Version

El Runtime también será independiente.

Ejemplo:

Authorization Runtime v5

Esto es importante porque un cambio en el evaluator puede cambiar resultados aun cuando:

Policy Version = unchanged
Scenario Version = unchanged

Por eso las ejecuciones deben conservar todas las versiones relevantes.

9. Semantic Versioning

Para los schemas se utilizará conceptualmente:

MAJOR.MINOR.PATCH
PATCH

Correcciones compatibles que no alteran la semántica.

Ejemplos:

corrección de metadata;
aclaración documental;
constraint adicional que no invalida datos válidos existentes;
corrección de descripción.
MINOR

Extensión compatible.

Ejemplos:

nuevo campo opcional;
nueva metadata opcional;
nuevo tipo de evidence no obligatorio.
MAJOR

Cambio incompatible.

Ejemplos:

eliminar un campo obligatorio;
cambiar su tipo;
cambiar significado;
modificar semántica;
hacer obligatorio un campo previamente opcional;
cambiar enumeraciones de manera incompatible;
modificar la semántica de una condición.
10. Regla Fundamental

Una nueva versión de schema no debe cambiar silenciosamente el significado de un artefacto existente.

Si:

Scenario v5

tenía:

expected.criticality = C1

una migración de schema no puede convertirlo silenciosamente en:

expected.criticality = C2

Eso sería un cambio funcional y debe pasar por governance de policy/test.

11. Compatibility Classes

Cada cambio de schema se clasificará como:

BACKWARD_COMPATIBLE
FORWARD_COMPATIBLE
FULLY_COMPATIBLE
INCOMPATIBLE
12. Backward Compatibility

Un schema nuevo puede leer artefactos producidos por un schema anterior.

Ejemplo:

Schema 1.1
      ↑
      │
Schema 1.2 reader

Si esto es posible, se declara:

BACKWARD_COMPATIBLE
13. Forward Compatibility

Un schema anterior puede leer un artefacto producido por una versión nueva.

Esto será mucho más restrictivo.

Por defecto:

Old Reader ← New Artifact

no se asumirá compatible.

14. Full Compatibility

Sólo se declarará FULLY_COMPATIBLE cuando:

Old Reader ↔ New Artifact

y:

New Reader ↔ Old Artifact

sean compatibles sin pérdida semántica.

15. Incompatible

Un cambio será incompatible si puede producir:

pérdida de información;
cambio de significado;
cambio de expected result;
pérdida de invariants;
pérdida de seguridad;
pérdida de tenant context;
pérdida de freshness context;
interpretación diferente del resultado.
16. Schema Registry

Se establece conceptualmente un:

Authorization Test Schema Registry

responsable de almacenar:

schema;
versión;
tipo de artifact;
compatibility class;
status;
migration path;
metadata;
owner;
approval;
effective date.
17. Schema Lifecycle

Cada schema seguirá:

DRAFT
   ↓
VALIDATING
   ↓
APPROVED
   ↓
ACTIVE
   ↓
DEPRECATED
   ↓
RETIRED

Un schema no podrá pasar directamente de:

DRAFT → ACTIVE

sin validación y governance.

18. Schema Deprecation

Cuando un schema deje de ser recomendado:

ACTIVE
   ↓
DEPRECATED

Esto significa:

puede seguir leyéndose;
puede seguir utilizándose para reproducir evidencia;
no debe utilizarse para nuevos artefactos cuando exista una versión sucesora;
debe existir una ruta de migración si corresponde.
19. Schema Retirement

RETIRED significa que el schema ya no debe utilizarse para nuevas ejecuciones.

Pero retirement no implica necesariamente borrar los artefactos históricos.

La evidencia histórica debe permanecer interpretable durante el período requerido por governance/retention.

20. No Destructive Migration

Las migraciones serán no destructivas.

No se permitirá:

Old Artifact
   ↓
overwrite
   ↓
New Artifact

como único registro.

En su lugar:

Old Artifact
   ↓
Migration
   ↓
New Artifact

manteniendo referencia al original.

21. Migration Artifact

Toda migración relevante generará metadata:

Migration
 ├── migration_id
 ├── source_schema
 ├── target_schema
 ├── source_artifact
 ├── target_artifact
 ├── migration_version
 ├── executed_at
 ├── tool_version
 ├── result
 ├── warnings
 └── evidence
22. Migration Direction

Las migraciones deben ser explícitas:

Schema 1.0
   ↓
Schema 1.1

No se asumirá que:

1.0 → 2.0

sea automáticamente posible.

Para una migración mayor se requerirá una estrategia específica.

23. Deterministic Migration

Una migración debe ser determinista.

Para:

Artifact A
+
Migration M

debe producirse siempre:

Artifact B

bajo la misma versión de migrator.

24. Migration Idempotency

Ejecutar dos veces la misma migración sobre el mismo artefacto no debe producir dos estados diferentes.

Conceptualmente:

Migrate(A, M) = B

Migrate(B, M) = B

o debe rechazarse explícitamente como:

ALREADY_MIGRATED

Nunca debe producir corrupción.

25. Migration Validation

Después de migrar:

Source Artifact
      ↓
Migration
      ↓
Target Artifact
      ↓
Schema Validation
      ↓
Semantic Validation
      ↓
Security Validation

No basta con que el JSON resultante sea sintácticamente válido.

26. Semantic Equivalence

Para migraciones compatibles deberá verificarse que:

Meaning(Source)
=
Meaning(Target)

cuando el objetivo sea una migración semánticamente equivalente.

Especialmente:

Expected Criticality
Expected Freshness
Expected Cacheability
Security Behavior
Invariant References
Tenant Semantics
Risk Semantics
MFA Semantics
Session Semantics
Delegation Semantics

no pueden alterarse accidentalmente.

27. Security Equivalence

Una migración nunca puede producir:

C0 → C1

o:

F0 → F1

por una transformación puramente estructural.

Si ocurre, debe clasificarse como:

SECURITY_BEHAVIOR_CHANGE

y pasar por governance.

28. Migration Safety Levels

Se establecen conceptualmente:

SAFE
SAFE_WITH_REVIEW
SECURITY_SENSITIVE
BLOCKED
SAFE

Cambio estructural sin impacto semántico.

SAFE_WITH_REVIEW

Cambio compatible pero con impacto potencial.

SECURITY_SENSITIVE

Afecta elementos de seguridad.

BLOCKED

No puede realizarse automáticamente.

29. Schema Evolution Rules
Regla 1

Agregar campo opcional:

MINOR

cuando no cambia semántica.

Regla 2

Eliminar campo:

MAJOR
Regla 3

Cambiar tipo:

MAJOR

salvo mecanismo explícito compatible.

Regla 4

Cambiar significado:

MAJOR
Regla 5

Cambiar enum de seguridad:

SECURITY_SENSITIVE
30. Security-Sensitive Fields

Los siguientes campos tendrán controles reforzados:

criticality
freshness_class
cacheability
security_behavior
tenant
security_state
risk
mfa
session
delegation
expected_result
invariants
policy_reference

Cambios sobre estos campos no podrán clasificarse simplemente como cambios estructurales.

31. Invariant Compatibility

Los schemas deben proteger las referencias a:

SecurityInvariant

Una migración que pierda un invariant debe fallar.

Ejemplo:

Scenario v3
 ├── INV-001
 ├── INV-004
 └── INV-013

Después de migración:

Scenario v4
 ├── INV-001
 └── INV-004

La desaparición de INV-013 debe detectarse.

32. Golden Scenario Compatibility

Los Golden Scenarios tendrán protección adicional.

Una migración estructural no podrá:

cambiar expected criticality;
eliminar invariants;
cambiar tenant semantics;
cambiar security behavior.

sin governance explícito.

33. Evidence Compatibility

La evidencia histórica deberá conservar:

schema_version
artifact_version
policy_version
runtime_version
catalog_version
scenario_version
invariant_version
generator_version

Esto permitirá reproducir exactamente qué significaba un resultado histórico.

34. Historical Reproducibility

Un principio obligatorio será:

La evolución del schema no debe destruir la capacidad de interpretar o reproducir una ejecución histórica.

Ejemplo:

Execution 2026
   ↓
Schema 1.1
Policy 10
Runtime 4

Aunque en 2028 exista:

Schema 3.0
Policy 27
Runtime 12

la ejecución histórica deberá seguir siendo interpretable con su contexto original.

35. Migration vs Re-Execution

Una migración de schema no equivale a volver a ejecutar un test.

Son operaciones diferentes:

Migration
    = transforma representación

Re-execution
    = vuelve a evaluar comportamiento

Nunca deberán confundirse.

36. Policy Re-Evaluation

Si un cambio de schema modifica la semántica de un campo de seguridad, no se considerará suficiente migrar el JSON.

Será necesario:

Migration
   ↓
Validation
   ↓
Re-execution
   ↓
Regression Evaluation
37. Runtime Compatibility

Cada Test Execution debe declarar qué Runtime Version entiende el artefacto.

Un catálogo puede ser estructuralmente válido pero incompatible con un runtime.

Ejemplo:

Catalog Schema = 2.0
Runtime = 1.x

debe detectarse antes de ejecución.

38. Compatibility Matrix

Conceptualmente:

Artifact	Schema	Runtime	Policy	Status
Scenario v1	1.0	1.x	10	Compatible
Scenario v2	1.1	2.x	11	Compatible
Scenario v2	1.1	1.x	11	Requiere validación
Evidence v1	1.0	1.x	10	Histórico
Candidate	2.0	4.x	12	Nuevo

La matriz exacta se definirá posteriormente.

39. Migration Tooling

La implementación deberá proporcionar un componente lógico:

Schema Migration Engine

responsable de:

detectar versión;
validar source;
ejecutar migración;
validar target;
comprobar equivalencia;
registrar warnings;
producir evidencia;
generar migration result.

No se deberá permitir que cada servicio implemente migraciones independientes sin governance.

40. Schema Diff

Antes de aprobar un nuevo schema se deberá producir un:

Schema Diff

que clasifique:

ADDED
REMOVED
CHANGED
RENAMED
REQUIRED
OPTIONAL
SEMANTIC_CHANGE
SECURITY_SENSITIVE
41. Automated Compatibility Check

El Schema Registry deberá ejecutar automáticamente:

Schema Validation
+
Compatibility Check
+
Migration Check
+
Security Field Check

antes de permitir aprobación.

42. Contract Tests

Cada nueva versión deberá pasar contract tests que verifiquen:

lectura de versiones soportadas;
escritura de versión actual;
compatibility;
migration;
backward compatibility;
referencias;
required fields;
enums;
security fields.
43. Golden Migration Tests

Los Golden Scenarios también se utilizarán para probar migraciones.

Ejemplo:

GSC-TENANT-001 v1
      ↓
Migration
      ↓
GSC-TENANT-001 v2

Debe comprobarse que:

Tenant Isolation

permanece intacto.

44. Migration Regression

Toda migración deberá pasar por:

Source Validation
      ↓
Migration
      ↓
Target Validation
      ↓
Semantic Comparison
      ↓
Invariant Evaluation
      ↓
Regression Evaluation

Una migración que produzca:

SECURITY_REGRESSION

queda bloqueada.

45. Schema Rollback

El rollback de un schema no debe significar simplemente:

current → previous

si ya existen artefactos generados bajo la versión nueva.

Debe existir:

Compatibility Assessment

antes del rollback.

46. Policy Rollback vs Schema Rollback

También se diferencian:

Policy Rollback

de:

Schema Rollback

Un rollback de Policy puede modificar comportamiento de autorización.

Un rollback de Schema modifica representación.

No deben compartir automáticamente el mismo mecanismo.

47. Artifact Retention

Los artefactos históricos conservarán su schema original mientras sea necesario para:

auditoría;
reproducibilidad;
debugging;
compliance;
incident investigation;
disaster recovery.

No se obliga a migrar inmediatamente toda la historia al schema actual.

48. Lazy vs Eager Migration

Se establece como principio:

Preferir migración explícita y controlada frente a migración automática durante lectura.

No se recomienda que:

Read Old Artifact
       ↓
Automatically Rewrite
       ↓
New Schema

sea el comportamiento por defecto.

Esto podría ocultar cambios y dificultar auditoría.

49. Dual-Read / Dual-Write

Podrá utilizarse temporalmente:

Old Schema
+
New Schema

durante una transición.

Pero cualquier dual-write debe garantizar:

equivalencia;
integridad;
trazabilidad;
consistencia;
ausencia de divergencia.

La duración exacta de este mecanismo queda pendiente.

50. Schema Deprecation Window

Cuando un schema sea deprecado deberá existir una ventana durante la cual:

pueda leerse;
pueda reproducirse;
pueda validarse;
pueda migrarse.

La duración exacta queda pendiente de governance y retention.

51. Unknown Schema

Si un sistema recibe:

schema_version = UNKNOWN

no deberá intentar inferir su estructura.

Debe responder:

UNSUPPORTED_SCHEMA_VERSION

y no ejecutar el artefacto.

52. Invalid Schema

Si el artefacto declara:

schema_version = 1.2

pero no cumple el schema:

SCHEMA_VALIDATION_ERROR

No podrá continuar a evaluación.

53. Security-Sensitive Migration Failure

Si una migración no puede determinar con certeza la equivalencia de:

criticality
freshness
security behavior
tenant
invariants

el resultado será:

BLOCKED

y no:

BEST_EFFORT
54. Machine-Readable Schema Metadata

Cada schema deberá declarar conceptualmente:

{
  "schema_id": "...",
  "schema_version": "1.2.0",
  "artifact_type": "SCENARIO",
  "status": "ACTIVE",
  "compatibility": {
    "backward": true,
    "forward": false
  },
  "previous_version": "1.1.0",
  "migration_available": true
}

El formato exacto queda para la especificación JSON Schema posterior.

55. Governance

Todo cambio de schema deberá incluir:

Change Request
+
Schema Diff
+
Compatibility Analysis
+
Migration Plan
+
Security Impact Analysis
+
Test Results
+
Approval

Los cambios mayores requerirán revisión reforzada.

56. AI-Assisted Schema Evolution

La IA podrá:

proponer modificaciones;
detectar incompatibilidades;
generar migraciones candidatas;
generar contract tests;
identificar campos potencialmente afectados.

Pero no podrá:

aprobar un schema;
aprobar una migración crítica;
eliminar invariants;
modificar silenciosamente security semantics;
retirar un schema;
decidir equivalencia de seguridad sin validación determinista.
57. Observabilidad

Se deberán medir:

schema_validation_count
schema_validation_failure
migration_count
migration_failure
migration_duration
compatibility_failure
unsupported_schema_count
security_sensitive_migration_count
migration_regression_count
artifact_version_distribution
schema_version_distribution
deprecated_schema_usage

Esto permitirá saber cuándo una versión antigua sigue siendo utilizada.

58. Security Invariants

Se incorporan invariants específicos del versionamiento.

INV-SCHEMA-001 — No Semantic Downgrade

Una migración estructural no puede reducir requisitos de seguridad.

INV-SCHEMA-002 — Historical Reproducibility

Los artefactos históricos deben permanecer interpretables.

INV-SCHEMA-003 — Explicit Compatibility

No se asumirá compatibilidad no declarada.

INV-SCHEMA-004 — Deterministic Migration

La migración debe ser reproducible.

INV-SCHEMA-005 — No Silent Rewrite

No se modifican artefactos históricos silenciosamente.

INV-SCHEMA-006 — Invariant Preservation

Una migración no puede eliminar Security Invariants sin governance.

INV-SCHEMA-007 — Tenant Semantics Preservation

La semántica tenant debe conservarse.

INV-SCHEMA-008 — Criticality Preservation

Una migración compatible no puede modificar Criticality.

INV-SCHEMA-009 — Freshness Preservation

Una migración compatible no puede degradar Freshness.

INV-SCHEMA-010 — Security Evidence Integrity

La evidencia histórica debe conservar su contexto de versiones.

59. Non-Negotiables
Schema Version y Artifact Version son conceptos distintos.
Policy Version y Schema Version son independientes.
Runtime Version debe registrarse en ejecuciones.
Los schemas usan versionamiento semántico.
Cambios incompatibles requieren MAJOR.
La compatibilidad debe declararse explícitamente.
Los artefactos históricos no se sobrescriben silenciosamente.
Las migraciones son deterministas.
Las migraciones son idempotentes.
Las migraciones deben validarse después de ejecutarse.
Debe comprobarse equivalencia semántica cuando corresponda.
Los Security Invariants no pueden perderse silenciosamente.
Tenant semantics no puede degradarse.
Criticality no puede degradarse por migración.
Freshness no puede degradarse por migración.
Unknown schema debe bloquear.
Invalid schema debe bloquear.
Security-sensitive migration failure debe bloquear.
Migration ≠ Re-execution.
Policy rollback ≠ Schema rollback.
Evidence histórica debe permanecer reproducible.
Schema retirement no implica borrar automáticamente evidencia.
AI no puede aprobar migraciones críticas.
Schema changes requieren governance.
Compatibility debe probarse automáticamente.
Golden Scenarios deben utilizarse para validar migraciones críticas.
Los resultados de migración deben ser auditables.
No se permite best-effort para semántica de seguridad incierta.
60. Consecuencias positivas

Este ADR proporciona:

evolución controlada;
compatibilidad explícita;
reproducibilidad histórica;
migraciones seguras;
separación de responsabilidades;
menor riesgo de romper CI/CD;
protección de evidencia;
protección de Security Invariants;
trazabilidad;
soporte para múltiples versiones simultáneas;
capacidad de deprecación gradual.
61. Consecuencias negativas

Introduce:

Schema Registry;
Migration Engine;
más metadata;
mayor governance;
necesidad de mantener schemas antiguos;
coste de contract testing;
complejidad temporal durante migraciones;
necesidad de compatibilidad entre componentes.

Se considera aceptable debido a la criticidad del sistema de autorización.

62. Alternativas rechazadas
62.1 Un único schema mutable

Rechazada.

Rompería la reproducibilidad histórica.

62.2 Versionar solamente el catálogo

Rechazada.

No permite diferenciar evolución estructural de evolución del contenido.

62.3 Migración automática durante lectura

Rechazada como comportamiento por defecto.

Puede ocultar cambios y destruir trazabilidad.

62.4 Reescribir todos los artefactos históricos

Rechazada.

Es innecesario y aumenta el riesgo de pérdida de evidencia.

62.5 Ignorar incompatibilidades

Rechazada.

Un parser que acepta datos incompatibles puede producir resultados de seguridad incorrectos.

63. Dependencias

Este ADR depende directamente de:

ADR-046 — Policy Schema/Governance.
ADR-048 — Policy DSL.
ADR-059 — Criticality Policy Model.
ADR-062 — Test Catalog / Golden / Invariants.
ADR-063 — Test Execution Engine.
ADR-064 — Test Data Model / Evidence.

También depende de:

Schema Registry;
Policy Registry;
Resource Registry;
Action Registry;
Attribute Registry;
Test Execution Engine;
Migration Engine;
Governance;
CI/CD;
Audit;
Observability;
Artifact Storage.
64. Trazabilidad

La evolución queda:

ADR-059
Criticality Policy
       ↓
ADR-062
What must be tested
       ↓
ADR-063
How tests execute
       ↓
ADR-064
How test data is represented
       ↓
ADR-065
How test schemas evolve safely

Por lo tanto:

ADR-065 cierra el ciclo de gobernanza técnica de los artefactos de testing definidos en ADR-064.

65. Acceptance Criteria
AC-01

Debe existir versionamiento independiente para Schema y Artifact.

AC-02

Debe existir versionamiento independiente para Policy y Runtime.

AC-03

Debe utilizarse Semantic Versioning para schemas.

AC-04

Debe existir clasificación explícita de compatibilidad.

AC-05

Debe existir Schema Registry conceptual.

AC-06

Debe existir lifecycle de schema.

AC-07

Debe existir estrategia de deprecación.

AC-08

Debe existir estrategia de retirement.

AC-09

Debe existir Migration Engine conceptual.

AC-10

Las migraciones deben ser deterministas.

AC-11

Las migraciones deben ser idempotentes.

AC-12

Las migraciones deben validarse después de ejecutarse.

AC-13

Debe comprobarse equivalencia semántica.

AC-14

Debe protegerse Criticality.

AC-15

Debe protegerse Freshness.

AC-16

Debe protegerse Tenant Semantics.

AC-17

Debe protegerse Security Invariant References.

AC-18

Debe preservarse la reproducibilidad histórica.

AC-19

Security Regression durante migración debe bloquear.

AC-20

Toda migración crítica debe ser auditable y gobernada.

66. Resultado arquitectónico

Con ADR-065 queda definida la evolución controlada de los artefactos de testing:

                 ┌──────────────────────┐
                 │ Schema Registry       │
                 └──────────┬───────────┘
                            │
             ┌──────────────┴──────────────┐
             ▼                             ▼
       Schema v1.x                    Schema v2.x
             │                             │
             ▼                             ▼
       Old Artifacts                 New Artifacts
             │                             │
             └──────────────┬──────────────┘
                            ▼
                    Migration Engine
                            │
                            ▼
                    Validation Engine
                            │
                            ▼
                    Security Invariants
                            │
                            ▼
                    Regression Engine
                            │
                            ▼
                     Governance Gate

La regla arquitectónica central queda establecida:

Los schemas pueden evolucionar, pero la evolución estructural nunca puede modificar silenciosamente el significado de seguridad de una política, escenario, invariant o evidencia.

Y también:

La historia de EVOXA debe permanecer reproducible: una ejecución pasada debe poder interpretarse con las versiones exactas de schema, policy, runtime, catalog, scenario e invariants con las que fue ejecutada.

67. Siguiente ADR recomendado

El siguiente paso lógico es:

ADR-IDENTITY-066 — Authorization Criticality Test Schema Registry, Contract Validation & Governance Gate

Este ADR debería convertir lo definido en 064–065 en el mecanismo de gobierno de schemas, definiendo:

Schema Registry
      ↓
Schema Registration
      ↓
Schema Validation
      ↓
Compatibility Check
      ↓
Migration Validation
      ↓
Security Validation
      ↓
Contract Tests
      ↓
Approval
      ↓
Activation
      ↓
Deprecation
      ↓
Retirement

Con esto cerraríamos prácticamente toda la arquitectura de Criticality Policy Testing & Schema Governance antes de pasar a la siguiente gran pieza del sistema.
