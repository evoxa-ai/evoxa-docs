ADR-IDENTITY-066 — Authorization Criticality Test Schema Registry, Contract Validation & Governance Gate

Status: Proposed
Version: 1.0
Date: 2026-09-07
Domain: Identity / Authorization / Security / Policy Engine / Testing / Governance
Priority: Critical
Type: Architecture / Security / Testing / Schema Governance / Contract Validation

1. Contexto

EVOXA ha definido una arquitectura de pruebas de seguridad y autorización que requiere que los artefactos de testing sean:

deterministas;
versionados;
reproducibles;
compatibles entre versiones;
auditables;
seguros;
capaces de bloquear cambios que produzcan regresiones de seguridad.

Los ADR anteriores establecieron progresivamente:

el modelo de criticidad de autorización;
las clases de freshness;
la elegibilidad de cache;
las políticas de criticidad;
los Golden Scenarios;
los Security Invariants;
el Test Catalog;
el Test Execution Engine;
el modelo machine-readable de escenarios, ejecuciones y evidencias;
la estrategia de versionado y migración de schemas.

En particular:

ADR-062 define el catálogo de pruebas, Golden Scenarios e invariantes.

ADR-063 define el motor de ejecución, generación de escenarios y evaluación de regresiones.

ADR-064 define el modelo de datos machine-readable y el esquema de evidencias.

ADR-065 define versionado, compatibilidad y migración de schemas.

Este ADR completa ese ciclo definiendo cómo se registran, validan y gobiernan los schemas utilizados por todo ese ecosistema.

2. Problema

Si los schemas de pruebas evolucionan sin un mecanismo central de gobierno, pueden aparecer problemas graves:

Schema inválido
      ↓
Artifacto mal formado
      ↓
Test incorrecto
      ↓
Resultado incorrecto
      ↓
Regresión de seguridad no detectada

También puede ocurrir:

Schema v1
   ↓
Schema v2
   ↓
Cambio semántico no detectado
   ↓
Golden Scenario cambia de significado
   ↓
Baseline histórica deja de ser comparable

Por lo tanto, EVOXA necesita un mecanismo central que permita:

registrar schemas;
identificarlos unívocamente;
versionarlos;
validar artefactos contra ellos;
determinar compatibilidad;
controlar migraciones;
impedir schemas incompatibles en producción;
mantener trazabilidad histórica;
establecer governance gates;
impedir que un cambio de schema reduzca garantías de seguridad.
3. Decisión

Se establece un Authorization Criticality Test Schema Registry como componente central de gobierno de schemas.

El Registry será la autoridad para:

identidad del schema;
versión;
estado;
compatibilidad;
validación;
deprecación;
retirement;
migraciones;
dependencias;
seguridad;
governance.

El Registry no ejecutará las pruebas.

La separación será:

Schema Registry
       │
       │ define / validates
       ▼
Test Catalog
       │
       ▼
Test Execution Engine
       │
       ▼
Evidence
       │
       ▼
Regression / Governance
4. Responsabilidades

El Registry tendrá las siguientes responsabilidades.

4.1 Schema Registration

Todo schema utilizado por el sistema de testing deberá estar registrado.

Ejemplos:

ScenarioSchema
GoldenScenarioSchema
SecurityInvariantSchema
TestCaseSchema
GeneratedScenarioSchema
TestExecutionSchema
ScenarioExecutionSchema
EvidenceSchema
RegressionResultSchema
CoverageResultSchema
MutationResultSchema
BaselineSchema
MigrationSchema
5. Identidad del Schema

Cada schema tendrá una identidad estable.

Conceptualmente:

schema_id
schema_name
schema_version
schema_version_type
schema_uri
schema_hash
status
domain
artifact_type
compatibility
created_at
approved_at
deprecated_at
retired_at

Ejemplo conceptual:

schema_id:
authorization-criticality/scenario

schema_version:
1.2.0

artifact_type:
SCENARIO

status:
ACTIVE

El schema_id no deberá cambiar simplemente porque cambie la versión.

6. Versionado

Se utilizará Semantic Versioning según lo establecido en ADR-065:

MAJOR.MINOR.PATCH
PATCH

Correcciones compatibles.

Ejemplo:

1.2.0 → 1.2.1

No deberá modificar el significado de seguridad.

MINOR

Extensiones compatibles.

Ejemplo:

1.2.0 → 1.3.0

Puede incorporar:

campos opcionales;
metadatos;
nuevas capacidades compatibles.
MAJOR

Cambios incompatibles.

Ejemplo:

1.3.0 → 2.0.0

Puede incluir:

eliminación de campos;
cambio de tipos;
cambios estructurales;
cambios semánticos;
modificación de invariantes;
cambios incompatibles de enumeraciones.
7. El Schema Registry será version-aware

El Registry deberá poder responder:

¿Qué schema validaba este escenario?

y obtener:

schema_id
schema_version
schema_hash
artifact_version

De esta manera una ejecución histórica no dependerá de que actualmente exista otra versión del schema.

8. Schema Hash

Cada schema aprobado deberá disponer de una representación canónica y un hash.

Conceptualmente:

schema_hash =
HASH(canonical_schema_definition)

Esto permite detectar:

modificación accidental;
modificación no registrada;
corrupción;
drift;
alteración posterior a aprobación.

Un cambio del contenido del schema deberá producir un cambio detectable del hash.

9. Estados del Schema

El lifecycle será:

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
DRAFT

Schema en diseño.

No puede utilizarse para validaciones oficiales.

VALIDATING

Schema sometido a:

validación estructural;
compatibilidad;
seguridad;
tests.
APPROVED

Ha pasado los governance gates requeridos.

ACTIVE

Puede utilizarse oficialmente.

DEPRECATED

No debe utilizarse para nuevos artefactos, pero puede ser necesario para interpretar históricos.

RETIRED

No puede utilizarse para nuevos artefactos.

Debe mantenerse disponible para reproducibilidad histórica cuando corresponda.

10. Regla fundamental de histórico

RETIRAR un schema no significa eliminarlo.

Por ejemplo:

ScenarioSchema v1.0
        ↓
ACTIVE
        ↓
DEPRECATED
        ↓
RETIRED

Los escenarios históricos que utilizaron v1.0 deben seguir siendo interpretables.

Esto protege:

auditoría;
reproducibilidad;
análisis histórico;
regresiones;
investigaciones de seguridad.
11. Schema Contract

Todo schema registrado deberá definir explícitamente:

Identity
Structure
Types
Required fields
Optional fields
Enums
Constraints
References
Compatibility
Security semantics
Version
Lifecycle
Dependencies

No se aceptarán schemas ambiguos.

12. Contract Validation

Antes de que un schema pueda pasar a ACTIVE, deberá superar un proceso de validación.

Schema
  ↓
Structural Validation
  ↓
Type Validation
  ↓
Reference Validation
  ↓
Compatibility Validation
  ↓
Security Validation
  ↓
Golden Tests
  ↓
Regression Tests
  ↓
Governance Gate
  ↓
APPROVED
  ↓
ACTIVE
13. Structural Validation

Debe verificarse:

sintaxis;
estructura;
tipos;
campos obligatorios;
campos opcionales;
referencias;
enums;
restricciones;
formatos;
identificadores.

Un schema inválido debe bloquearse.

14. Semantic Validation

La validación no puede limitarse a comprobar que el JSON/YAML sea válido.

También deberá comprobar que su significado sea correcto.

Por ejemplo, no sería suficiente:

criticality = "C1"

si la definición del schema permite accidentalmente interpretar C1 como C0.

Debe validarse la semántica de:

Criticality;
Freshness;
Cacheability;
Security Behavior;
Tenant;
Risk;
MFA;
Session;
Delegation;
Security State.
15. Security Contract Validation

Los schemas que contengan campos relacionados con seguridad serán considerados security-sensitive.

Entre ellos:

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

Un cambio sobre cualquiera de estos campos deberá pasar por un governance gate reforzado.

16. No Semantic Downgrade

Se establece como regla fundamental:

Una evolución de schema no puede reducir silenciosamente una garantía de seguridad.

Por ejemplo:

C0 → C1

no podrá considerarse una simple modificación compatible.

Igualmente:

F0 → F1

no podrá ser tratado automáticamente como un cambio menor.

Debe clasificarse como:

SECURITY_BEHAVIOR_CHANGE

y requerir aprobación explícita.

Esto mantiene las invariantes definidas anteriormente.

17. Compatibility Engine

El Registry deberá proporcionar una evaluación de compatibilidad.

Resultado conceptual:

BACKWARD_COMPATIBLE
FORWARD_COMPATIBLE
FULLY_COMPATIBLE
INCOMPATIBLE
SECURITY_SENSITIVE

La clasificación deberá ser determinista.

18. Schema Diff

Cada cambio de schema deberá generar un diff.

Categorías:

ADDED
REMOVED
CHANGED
RENAMED
REQUIRED
OPTIONAL
SEMANTIC_CHANGE
SECURITY_SENSITIVE

Ejemplo:

ScenarioSchema 1.2.0
        ↓
ScenarioSchema 1.3.0

Added:
risk.confidence

Changed:
freshness_class

Classification:
SECURITY_SENSITIVE

Aunque el campo agregado sea opcional, el impacto sobre seguridad debe evaluarse.

19. Contract Testing

Cada schema deberá disponer de tests de contrato.

Como mínimo:

Valid artifact
Invalid artifact
Missing required field
Invalid type
Invalid enum
Boundary values
Unknown fields
Version mismatch
Reference mismatch
Security-sensitive mutation
Compatibility test
20. Golden Schema Tests

Los schemas críticos deberán tener casos Golden.

Ejemplo conceptual:

GSC-SCHEMA-001
Scenario schema preserves C0

GSC-SCHEMA-002
Scenario schema preserves F0

GSC-SCHEMA-003
Tenant context remains mandatory

GSC-SCHEMA-004
Security invariant references remain valid

GSC-SCHEMA-005
Historical artifact remains readable

Los resultados esperados de estos Golden Tests estarán gobernados igual que los Golden Scenarios definidos en ADR-062.

21. Schema Dependency Graph

Los schemas pueden depender unos de otros.

Ejemplo:

ScenarioSchema
      │
      ├── ContextSchema
      │
      ├── ExpectedResultSchema
      │
      └── InvariantReferenceSchema

Por lo tanto, el Registry deberá mantener un modelo conceptual de dependencias.

Antes de activar una nueva versión:

Changed Schema
      ↓
Dependency Analysis
      ↓
Affected Artifacts
      ↓
Compatibility Analysis
      ↓
Regression Tests
22. Impact Analysis

Un cambio de schema deberá identificar potencialmente:

Affected Scenarios
Affected Golden Scenarios
Affected Invariants
Affected Test Cases
Affected Baselines
Affected Executions
Affected Evidence
Affected Migration Tools
Affected CI/CD Contracts

Esto evita modificar un schema sin conocer su impacto.

23. Governance Gate

Ningún schema crítico podrá pasar directamente de:

DRAFT

a:

ACTIVE

Debe pasar por:

DRAFT
 ↓
VALIDATING
 ↓
Compatibility Analysis
 ↓
Security Analysis
 ↓
Contract Tests
 ↓
Golden Tests
 ↓
Regression Tests
 ↓
Approval
 ↓
APPROVED
 ↓
ACTIVE
24. Governance Gate obligatorio

Para cambios security-sensitive será obligatorio contar con:

Change Request
Schema Diff
Compatibility Analysis
Security Impact Analysis
Affected Artifact Analysis
Contract Test Results
Golden Test Results
Regression Results
Migration Plan
Approval

Sin estos elementos:

ACTIVE = BLOCKED
25. AI Governance

La IA podrá ayudar a:

proponer cambios;
generar schemas;
detectar inconsistencias;
generar tests;
generar diffs;
analizar impacto;
sugerir migraciones.

Pero:

La IA no podrá aprobar ni activar un schema de seguridad.

Especialmente no podrá:

alterar platform minimums
alterar invariants
aprobar security regression
activar schema security-sensitive

La decisión final será humana y gobernada.

26. Schema Validation API

Conceptualmente el Registry podrá proporcionar APIs como:

POST /schemas
GET  /schemas
GET  /schemas/{schema_id}
GET  /schemas/{schema_id}/versions
POST /schemas/{schema_id}/validate
POST /schemas/{schema_id}/compatibility
POST /schemas/{schema_id}/diff
POST /schemas/{schema_id}/approve
POST /schemas/{schema_id}/deprecate
POST /schemas/{schema_id}/retire

Estos endpoints son conceptuales y deberán convertirse posteriormente en contratos API formales.

27. Validación de Artifactos

El Registry deberá poder responder:

Artifact
   ↓
Schema ID
   ↓
Schema Version
   ↓
Validation
   ↓
VALID / INVALID

Ejemplo:

Scenario:
GSC-TENANT-001

Schema:
authorization-criticality/scenario

Version:
1.2.0

Result:
VALID
28. Unknown Schema Behavior

Si un sistema recibe:

schema_version = desconocida

no podrá asumir compatibilidad.

La respuesta será:

SCHEMA_UNKNOWN

y el artefacto no podrá utilizarse como evidencia oficial hasta resolver el schema.

Esto evita interpretar datos de seguridad utilizando una estructura desconocida.

29. Invalid Schema Behavior

Si el schema:

no existe;
está corrupto;
tiene hash incorrecto;
no está aprobado;
es incompatible;
contiene una referencia inválida;

la validación debe:

FAIL CLOSED

No se deberá convertir automáticamente un error de schema en un resultado válido.

30. Schema Registry vs Test Execution Engine

Se mantiene una separación explícita:

Schema Registry

Responde:

"¿Qué significa este artefacto y cómo se valida?"

Test Execution Engine

Responde:

"¿Qué resultado produce este escenario al ejecutarse?"

Por lo tanto:

Registry ≠ Execution Engine

Esta separación evita acoplamiento excesivo.

31. Schema Registry vs Policy Engine

También se mantiene:

Policy Engine
      ≠
Schema Registry
      ≠
Test Execution Engine

El Policy Engine determina:

Authorization / Criticality

El Schema Registry determina:

validity / structure / compatibility

El Test Engine determina:

test execution / regression
32. CI/CD Integration

El Registry deberá integrarse con CI/CD.

Conceptualmente:

Pull Request
     ↓
Schema Diff
     ↓
Compatibility Check
     ↓
Contract Tests
     ↓
Security Tests
     ↓
Golden Tests
     ↓
Regression
     ↓
Governance Gate
     ↓
PASS / BLOCK

Un cambio incompatible no aprobado deberá bloquear el pipeline.

33. Deployment Gate

Un deployment que incluya cambios de schema críticos deberá verificar:

Schema registered
Schema approved
Schema compatible
Security tests passed
Golden tests passed
Regression passed
Migration available
Rollback strategy available

Si cualquiera falla:

DEPLOYMENT BLOCKED
34. Rollback

Un rollback de schema no deberá consistir simplemente en cambiar:

1.3.0 → 1.2.0

sin análisis.

Deberá comprobar:

compatibilidad del runtime;
artefactos existentes;
migraciones;
baselines;
evidencias;
referencias;
policy version;
scenario catalog version.
35. Historical Reproducibility

Una ejecución histórica deberá poder reconstruirse utilizando:

Policy Version
Runtime Version
Schema Version
Catalog Version
Scenario Version
Invariant Version
Generator Version

Esto mantiene el principio:

Same inputs + same versions → reproducible interpretation

36. Schema Migration

La migración continuará siguiendo ADR-065.

Nunca:

Artifact v1
   ↓
overwrite
   ↓
Artifact v2

En su lugar:

Artifact v1
   │
   ├── permanece
   │
   ▼
Migration
   │
   ▼
Artifact v2

La migración deberá ser:

determinista;
idempotente;
auditable;
reversible cuando corresponda;
validada.
37. Security Migration Gate

Las migraciones de schemas security-sensitive deberán clasificarse:

SAFE
SAFE_WITH_REVIEW
SECURITY_SENSITIVE
BLOCKED

Un cambio que reduzca garantías de seguridad será:

BLOCKED

hasta que exista una decisión explícita de arquitectura/gobierno.

38. Schema Immutability

Una versión de schema que esté:

APPROVED
ACTIVE
DEPRECATED
RETIRED

será considerada inmutable.

No se podrá modificar silenciosamente su definición.

Si cambia:

schema_hash

deberá tratarse como una nueva versión o cambio formal.

39. Observabilidad

El Registry deberá producir métricas sobre:

schema_validation_total
schema_validation_failed
schema_compatibility_failed
schema_security_sensitive_changes
schema_migration_total
schema_migration_failed
schema_unknown
schema_invalid
schema_deprecated_usage
schema_retired_usage
governance_gate_blocked

También deberá existir trazabilidad mediante:

correlation_id
trace_id
artifact_id
schema_id
schema_version

No deberán registrarse secretos.

40. Auditoría

Los siguientes eventos deberán ser auditables:

SchemaCreated
SchemaValidated
SchemaApproved
SchemaActivated
SchemaDeprecated
SchemaRetired
SchemaCompatibilityEvaluated
SchemaMigrationCreated
SchemaMigrationExecuted
SchemaGovernanceBlocked
SchemaGovernanceApproved

El audit trail deberá conservar:

actor
timestamp
schema
version
change
reason
result
approval
41. Tenant Isolation

Los schemas de plataforma y los schemas específicos de organización deberán distinguirse.

Sin embargo:

Una organización no podrá modificar un schema que reduzca las garantías mínimas de seguridad de plataforma.

Especialmente no podrá debilitar:

C0 requirements
F0 requirements
tenant isolation
security invariants
mandatory fields
security states
MFA semantics
delegation boundaries
42. Version Compatibility Matrix

El Registry deberá mantener conceptualmente una matriz:

Schema	Compatible con	Estado
1.0.0	1.0.x	Compatible
1.1.0	1.0.x	Compatible según reglas
2.0.0	1.x	Incompatible
Security-sensitive change	—	Governance required

La matriz exacta será generada por el Compatibility Engine.

43. Security Invariants

Se establecen como invariantes mínimas:

INV-SCHEMA-011 — Schema Integrity

Un schema ACTIVE debe coincidir con su identidad y hash registrado.

INV-SCHEMA-012 — No Unknown Schema Allow

Un schema desconocido no puede producir evidencia oficial válida.

INV-SCHEMA-013 — No Silent Semantic Change

Una modificación semántica debe generar una nueva versión o governance change explícito.

INV-SCHEMA-014 — Security Field Protection

Los cambios sobre campos security-sensitive requieren validación reforzada.

INV-SCHEMA-015 — Historical Reproducibility

Los schemas históricos necesarios para interpretar evidencia deben conservarse.

INV-SCHEMA-016 — Compatibility Explicitness

Ninguna compatibilidad puede asumirse implícitamente.

INV-SCHEMA-017 — No Security Downgrade

Una evolución de schema no puede reducir garantías de seguridad sin aprobación explícita.

INV-SCHEMA-018 — Immutable Active Schema

Un schema ACTIVE no puede modificarse silenciosamente.

INV-SCHEMA-019 — Tenant Security Preservation

La evolución de schema debe conservar la semántica de aislamiento de tenant.

INV-SCHEMA-020 — Governance Gate Integrity

Un schema security-sensitive no puede activarse sin completar los controles requeridos.

44. Consecuencias positivas

Esta decisión proporciona:

control central de schemas;
reproducibilidad;
compatibilidad explícita;
detección de breaking changes;
protección de Golden Scenarios;
protección de invariantes;
integración CI/CD;
trazabilidad;
auditoría;
migraciones controladas;
seguridad histórica;
governance automatizable.

Además, permite que el sistema de testing evolucione sin perder la capacidad de comparar resultados históricos.

45. Costos / consecuencias negativas

La arquitectura agrega:

Registry;
lifecycle de schemas;
compatibility engine;
contract tests;
governance gates;
migraciones;
almacenamiento histórico;
mayor complejidad de CI/CD.

También aumenta el trabajo inicial antes de implementar.

Sin embargo, este coste se considera justificado porque estamos hablando de componentes que controlan autorización, criticidad y seguridad.

46. Alternativas consideradas
A. No utilizar Registry

Rechazada.

Produciría schemas distribuidos y difícilmente gobernables.

B. Guardar schemas solamente en Git

Rechazada como solución completa.

Git puede ser el mecanismo de versionado de código, pero no sustituye:

validación;
compatibility analysis;
governance;
runtime lookup;
historical registry.

Puede formar parte de la implementación.

C. Validación únicamente en runtime

Rechazada.

Los errores deben detectarse antes del deployment siempre que sea posible.

D. Permitir backward compatibility automática

Rechazada.

La compatibilidad debe demostrarse, especialmente para campos de seguridad.

E. Permitir a la IA aprobar schemas

Rechazada.

La IA puede asistir, pero no puede ejercer la autoridad de governance sobre seguridad.

47. No negociables
Existe un Schema Registry.
Todo schema crítico debe estar registrado.
Cada schema tiene identidad estable.
Cada versión es explícita.
Se utiliza Semantic Versioning.
Los schemas ACTIVE son inmutables.
Cada schema tiene hash verificable.
La compatibilidad debe evaluarse explícitamente.
Los cambios security-sensitive requieren governance reforzado.
No existe compatibilidad implícita.
Unknown schema no puede producir evidencia oficial.
Schema inválido falla closed.
Se preserva histórico.
Se preservan Golden Scenarios.
Se preservan Security Invariants.
No puede existir silent semantic change.
No puede existir silent security downgrade.
Las migraciones son no destructivas.
Las migraciones son deterministas.
Las migraciones son idempotentes.
El Registry no ejecuta políticas.
El Registry no ejecuta tests.
El Test Engine no gobierna schemas.
CI/CD debe validar cambios críticos.
Los cambios security-sensitive requieren aprobación.
AI no puede aprobar ni activar cambios críticos.
Tenant isolation debe preservarse.
Historical reproducibility es obligatoria.
No se almacenan secretos en schemas/evidencias/logs.
Governance debe ser auditable.
48. Traceability
Blueprint
Identity
Authorization
Security Policy Engine
Testing
Eventing
Observability
AI Governance
Implementation Stories
IS-006 — Authorization
IS-011 — Security Policy Engine
IS-012 — Audit & Security Events
IS-013 — Domain Events & Outbox
IS-016 — Testing & Quality
IS-017 — Runtime & Deployment
ADRs relacionados
ADR-045 — Authorization Policy Model
ADR-046 — Policy Language & Governance
ADR-048 — Policy DSL
ADR-049 — Evaluation Runtime

ADR-053 — Cache Invalidation
ADR-054 — Cache Failure & Recovery
ADR-055 — Cache Recovery
ADR-056 — Consistency
ADR-057 — Freshness Classes
ADR-058 — Criticality Matrix
ADR-059 — Criticality Policy Model

ADR-062 — Test Catalog & Security Invariants
ADR-063 — Test Execution Engine
ADR-064 — Test Data Model & Evidence
ADR-065 — Schema Versioning & Migration
49. Dependencias

Este ADR depende de:

Authorization Policy Engine
Criticality Policy Model
Security Invariants
Golden Scenarios
Test Execution Engine
Test Data Model
Evidence Model
Schema Migration Strategy
CI/CD
Audit
Observability
Governance
50. Decisiones pendientes

Este ADR define la arquitectura, pero todavía quedan decisiones de implementación:

50.1 Formato primario

Pendiente definir:

JSON Schema

como formato normativo definitivo o combinación con otros formatos.

50.2 Storage

Pendiente decidir:

PostgreSQL
Git
Object Storage
Schema Registry Service

o combinación.

50.3 Hash

Pendiente determinar:

algoritmo;
canonicalización;
firma;
key management si existe firma criptográfica.
50.4 API definitiva

Pendiente convertir las APIs conceptuales en API Contracts.

50.5 Compatibility Engine

Pendiente definir algoritmo exacto.

50.6 Governance Workflow

Pendiente definir:

roles;
approvals;
Change Request;
separación de funciones.
50.7 Retention

Pendiente determinar cuánto tiempo deben conservarse:

schemas;
migrations;
validation results;
governance evidence.
50.8 CI/CD

Pendiente definir los gates exactos para:

PR
CI
Release
Deployment
Production
51. Acceptance Criteria
AC-01

Todo schema crítico puede registrarse con identidad y versión únicas.

AC-02

Un schema no registrado no puede utilizarse oficialmente.

AC-03

La versión del schema puede recuperarse históricamente.

AC-04

El Registry detecta cambios incompatibles.

AC-05

Los cambios security-sensitive son detectados.

AC-06

Los schemas ACTIVE son inmutables.

AC-07

Cada versión tiene hash verificable.

AC-08

Un artifacto puede validarse contra una versión específica.

AC-09

Un artifacto con schema desconocido falla.

AC-10

Un artifacto inválido falla.

AC-11

Los Golden Schema Tests son ejecutables.

AC-12

Los Security Invariants permanecen referenciables.

AC-13

Las dependencias de schemas pueden analizarse.

AC-14

Los cambios generan Schema Diff.

AC-15

Los cambios generan Compatibility Analysis.

AC-16

Las migraciones son no destructivas.

AC-17

Las migraciones son deterministas e idempotentes.

AC-18

CI/CD puede bloquear schemas no aprobados.

AC-19

Los cambios security-sensitive requieren governance.

AC-20

La IA no puede aprobar ni activar schemas críticos.

52. Resultado arquitectónico

Con este ADR completamos una pieza importante del sistema de testing de Authorization Criticality:

ADR-062
Test Catalog
     ↓
ADR-063
Test Execution Engine
     ↓
ADR-064
Machine-Readable Data & Evidence
     ↓
ADR-065
Schema Versioning & Migration
     ↓
ADR-066
Schema Registry + Contract Validation
     ↓
Governance Gate

El resultado es:

Policy
  ↓
Criticality
  ↓
Golden Scenarios
  ↓
Security Invariants
  ↓
Machine-Readable Tests
  ↓
Schema Registry
  ↓
Contract Validation
  ↓
Test Execution
  ↓
Regression
  ↓
Governance
  ↓
Deployment

ADR-IDENTITY-066 queda definido como Proposed.

Siguiente ADR recomendado

ADR-IDENTITY-067 — Authorization Criticality Policy & Test Runtime Contracts

Este será especialmente importante porque nos permitirá cerrar el contrato exacto entre el Policy Engine, el Authorization Runtime, el Criticality Engine y el Test Execution Engine. Después de eso podremos acercarnos mucho más a las ESP y a las Technical Tasks, que es el punto donde finalmente empezaremos a preparar el código real.
