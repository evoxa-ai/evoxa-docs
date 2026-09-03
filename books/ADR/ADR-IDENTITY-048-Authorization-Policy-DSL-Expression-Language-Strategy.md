ADR-IDENTITY-048 — Authorization Policy DSL & Expression Language Strategy

Status: Proposed
Version: 1.0
Date: 2026-09-03
Domain: Identity / Authorization
Priority: Critical
Type: Architecture / Security / Policy Language

1. Context

ADR-IDENTITY-045 estableció el Authorization Policy Engine como componente central para evaluar:

RBAC
ABAC
Resource Ownership
Delegation
Tenant Isolation
Security Policies
Contexto de sesión
MFA / Authentication Assurance
Risk / Device Trust

ADR-IDENTITY-046 estableció que las políticas deben utilizar una representación estructurada, con:

Policy
Target
Conditions
Effects
Obligations
Metadata
Versioning
Lifecycle
Validation
Simulation

ADR-IDENTITY-047 estableció que el Policy Engine será inicialmente un componente del Identity Domain, con PEP/PDP conceptualmente separados, evaluación determinista, políticas estructuradas, validación, versionado y posibilidad futura de evolucionar hacia un PDP independiente.

Por lo tanto, ahora es necesario definir cómo se expresarán las condiciones de autorización.

El Blueprint establece explícitamente que autorización combina RBAC, ABAC y políticas contextuales.

También define que el Policy Engine puede evaluar elementos como organización, tenant, ownership, device trust, risk, session y restricciones temporales.

2. Problem Statement

EVOXA necesita un lenguaje de políticas que permita expresar reglas como:

subject.role == COACH
resource.organization_id == subject.organization_id
resource.owner_id == subject.id
environment.risk_level <= MEDIUM
session.mfa_level >= REQUIRED

sin permitir:

ejecución arbitraria de código
SQL dinámico
acceso directo a infraestructura
acceso a secretos
funciones no controladas
modificación de estado
llamadas externas impredecibles.

El lenguaje también debe ser:

legible
validable
versionable
determinista
testeable
auditable
extensible
seguro
suficientemente eficiente.
3. Decision

EVOXA adoptará un Structured Policy DSL, basado en una representación declarativa y tipada.

La política no será código ejecutable.

La arquitectura será:

Policy Definition
       │
       ▼
Structured DSL
       │
       ▼
Parser / Schema Validator
       │
       ▼
AST / Intermediate Representation
       │
       ▼
Policy Validator
       │
       ▼
Policy Compiler
       │
       ▼
Policy Evaluator
       │
       ▼
Authorization Decision

La implementación concreta del parser/evaluator podrá evolucionar, pero la semántica del lenguaje deberá permanecer estable.

4. Principle: Policy as Data

Las políticas serán tratadas principalmente como Policy-as-Data.

Ejemplo conceptual:

id: training-session-read
version: 1
effect: ALLOW

target:
  resource: training_session
  actions:
    - read

conditions:
  - attribute: subject.role
    operator: IN
    value:
      - COACH

El formato anterior es ilustrativo y no constituye todavía el formato definitivo de almacenamiento.

La política representa una declaración:

"En estas condiciones, esta acción puede producir este resultado."

No representa un programa.

5. Declarative Model

El lenguaje será declarativo.

Una política deberá describir:

WHO
  ↓
CAN DO WHAT
  ↓
TO WHICH RESOURCE
  ↓
UNDER WHICH CONDITIONS
  ↓
WITH WHICH EFFECT

Ejemplo:

COACH
  ↓
update
  ↓
training_session
  ↓
same organization
  AND
owner
  ↓
ALLOW
6. Policy Structure

La estructura conceptual será:

Policy
├── id
├── name
├── version
├── status
├── scope
├── priority
├── effect
├── target
├── conditions
├── obligations
├── metadata
└── effective_period

Esto mantiene la estructura definida en ADR-IDENTITY-046.

7. Target Model

El target determina cuándo una política es relevante.

Ejemplo:

target:
  resource: training_session
  actions:
    - read
    - update

También podrá incorporar:

actor_type
organization
tenant
resource_type
operation

El target deberá ser suficientemente específico para evitar evaluar innecesariamente políticas irrelevantes.

8. Condition Model

Las condiciones serán expresiones estructuradas.

Conceptualmente:

condition:
  attribute: resource.organization_id
  operator: EQUALS
  value: subject.organization_id

Se podrán combinar mediante:

AND
OR
NOT

Ejemplo:

conditions:
  all:
    - attribute: subject.role
      operator: IN
      value: [COACH]

    - attribute: resource.organization_id
      operator: EQUALS
      value: subject.organization_id
9. Expression Language

Se adopta un Expression Language restringido, no un lenguaje de programación general.

Las expresiones estarán limitadas a:

Literales
string
integer
decimal
boolean
datetime
duration
null
Colecciones
list
set
map
Referencias
subject.*
resource.*
organization.*
membership.*
session.*
request.*
environment.*
delegation.*
Operadores
EQUALS
NOT_EQUALS
IN
NOT_IN
GREATER_THAN
GREATER_THAN_OR_EQUAL
LESS_THAN
LESS_THAN_OR_EQUAL
EXISTS
NOT_EXISTS
MATCHES

La lista definitiva deberá validarse durante la implementación.

10. Logical Operators

Se soportarán operadores lógicos controlados:

AND
OR
NOT

Ejemplo:

subject.role == COACH
AND
resource.organization_id == organization.id

Otro:

subject.role == ADMIN
OR
subject.role == SECURITY_ADMIN

Y:

NOT resource.sensitive

Las expresiones deberán formar un árbol determinista.

11. Typed Expressions

Las expresiones serán tipadas.

Ejemplo:

subject.id

→ UUID / Identifier

subject.role

→ Role

environment.risk_level

→ RiskLevel

request.timestamp

→ DateTime

Esto permite detectar errores antes de activar la política.

Ejemplo inválido:

subject.id > 100

si subject.id no admite comparación numérica.

La política deberá ser rechazada durante validación.

12. Attribute References

Los atributos deberán utilizar un namespace explícito.

Ejemplo:

subject.id
subject.type
subject.status
subject.roles

organization.id
organization.status

membership.id
membership.status

session.id
session.status
session.mfa_level

resource.id
resource.type
resource.owner_id
resource.organization_id

request.ip
request.method

environment.risk_level
environment.device_trust
environment.timestamp

El uso de atributos deberá estar restringido al Attribute Registry definido en ADR-IDENTITY-047.

13. No Arbitrary Attribute Access

Una política no podrá solicitar:

database.users.password_hash

ni:

system.environment.SECRET_KEY

ni:

internal.service.credentials

El Policy Engine solamente podrá recibir atributos explícitamente autorizados.

Esto sigue el principio de minimización y protección de información sensible establecido por la arquitectura de seguridad de EVOXA.

14. Function Model

Las funciones estarán restringidas a un catálogo aprobado.

Ejemplos potenciales:

contains()
starts_with()
ends_with()
lower()
upper()
is_empty()

Funciones relacionadas con autorización podrán incluir:

has_role()
has_permission()
is_owner()
is_member_of()

Pero deberán ser funciones puras y deterministas.

No deberán:

modificar datos
enviar requests
escribir archivos
ejecutar comandos
acceder a secretos
realizar SQL arbitrario.
15. External Calls

El DSL no permitirá llamadas HTTP arbitrarias.

Incorrecto:

http.get("https://...")

Tampoco:

call("risk-service")

directamente desde una policy.

Si información externa es necesaria, deberá formar parte previamente del AuthorizationContext.

Ejemplo:

environment.risk_level

El Policy Engine recibe el dato y evalúa.

No decide cómo obtenerlo mediante llamadas arbitrarias.

16. Time Expressions

El lenguaje deberá soportar condiciones temporales.

Ejemplo:

environment.timestamp >= policy.effective_from

o:

environment.time BETWEEN
09:00 AND 18:00

También podrá soportar:

session.age < duration("30m")

La sintaxis definitiva de duración y fechas queda pendiente.

17. Risk Expressions

El Policy Engine podrá utilizar un nivel de riesgo previamente calculado.

Ejemplo:

environment.risk_level <= MEDIUM

No será responsabilidad del DSL calcular modelos complejos de riesgo.

El principio será:

Risk Engine
     ↓
Risk Context
     ↓
Policy Engine

y no:

Policy
 ↓
ML Model
 ↓
External API
 ↓
Risk Decision

Esto mantiene el Policy Engine determinista y predecible.

18. MFA Expressions

El DSL podrá evaluar assurance de autenticación.

Ejemplo:

session.mfa_level >= HIGH

o:

session.mfa_verified == true

Una política podrá producir:

REQUIRE_MFA

en lugar de simplemente DENY.

Ejemplo:

IF
  resource.classification == HIGHLY_SENSITIVE
AND
  session.mfa_level < HIGH

THEN
  REQUIRE_MFA

La ejecución concreta del challenge MFA seguirá siendo responsabilidad del Identity MFA subsystem.

19. Ownership Expressions

Ownership será una capacidad de primera clase.

Ejemplo:

resource.owner_id == subject.id

También:

resource.team_id IN subject.team_ids

o:

resource.organization_id == organization.id

Esto permitirá combinar:

RBAC
+
Tenant
+
Ownership

en una misma decisión.

20. Tenant Isolation

Tenant isolation será una condición arquitectónica especial.

El lenguaje podrá expresar:

resource.organization_id == context.organization_id

pero la protección de tenant no dependerá exclusivamente de que un desarrollador recuerde escribir esa condición.

La arquitectura deberá aplicar un guard obligatorio:

Tenant Isolation Guard
        ↓
Policy Evaluation

Por lo tanto:

cross-tenant
      ↓
DENY

por defecto.

Esto es coherente con el Blueprint, que establece aislamiento mediante tenant identifiers, access boundaries y configuración independiente, y prohíbe el acceso cross-tenant salvo autorización explícita.

21. Role & Permission Expressions

RBAC podrá expresarse mediante:

has_role("COACH")

o conceptualmente:

subject.roles CONTAINS COACH

Las permissions podrán evaluarse mediante:

has_permission("training_session.update")

Sin embargo:

Permission no sustituye Policy Evaluation.

La arquitectura continúa siendo:

Authentication
      ↓
Role Resolution
      ↓
Policy Evaluation
      ↓
Permission Check

tal como establece el Blueprint.

22. Delegation Expressions

El lenguaje deberá poder representar delegación.

Ejemplo conceptual:

delegation.active == true
AND
delegation.target_resource == resource.type
AND
delegation.expires_at > environment.timestamp

Además:

delegated_permission <= delegator_effective_permission

No podrá existir privilege amplification mediante delegation.

23. Policy Effects

Los efectos soportados conceptualmente serán:

ALLOW
DENY
CHALLENGE
REAUTHENTICATE
REQUIRE_MFA
REVOKE

Una política individual podrá estar restringida a determinados efectos según su tipo y scope.

Por ejemplo, una política normal de acceso podría producir:

ALLOW
DENY
REQUIRE_MFA

Mientras una política de seguridad crítica podría producir:

REVOKE

bajo controles adicionales.

24. Obligations

El DSL podrá producir obligaciones.

Ejemplo:

obligations:
  - type: AUDIT
  - type: REQUIRE_STEP_UP

Una obligación no debe confundirse con un efecto.

Effect

Determina el resultado de autorización.

Obligation

Indica una acción requerida después/durante la decisión.

Ejemplo:

Decision:
ALLOW

Obligation:
AUDIT_HIGH_SENSITIVITY_ACCESS
25. Policy Composition

Las políticas deberán poder combinarse.

Conceptualmente:

Applicable Policies
        ↓
Policy Evaluation
        ↓
Conflict Resolution
        ↓
Final Decision

La composición deberá respetar la precedencia establecida anteriormente:

Regulatory
   ↓
Platform Security
   ↓
Organization Security
   ↓
Role
   ↓
User
   ↓
Context

Y:

DENY prevalece sobre ALLOW cuando existe conflicto aplicable, salvo que una regla superior explícitamente determine otro comportamiento seguro.

26. Policy Priority

Cada policy podrá tener:

scope
priority
version
effective_period

Ejemplo:

Platform Security
priority: 1000

Organization Security
priority: 700

Role Policy
priority: 500

Context Policy
priority: 100

Los valores son ilustrativos; la numeración definitiva queda pendiente.

27. Policy Inheritance

No se permitirá inicialmente una herencia implícita compleja.

Por ejemplo, no se asumirá:

ADMIN
  ↓
inherits
  ↓
COACH

El Blueprint no define una jerarquía universal de roles; por tanto, la semántica deberá permanecer explícita.

Si se introduce inheritance en el futuro:

deberá estar versionada
deberá ser explícita
deberá ser validada
deberá ser auditable
no deberá permitir privilege escalation accidental.
28. Policy Validation

El parser/validator deberá detectar:

Syntax errors
invalid expression
Type errors
string compared with integer
Unknown attribute
subject.secret_level
Unknown resource
unknown_resource
Unknown action
training.magic_action
Invalid operator
UUID GREATER_THAN integer
Invalid effect
SUPER_ALLOW
Invalid function
execute_shell()
29. Static Analysis

Antes de activar una policy se ejecutará análisis estático.

Debe detectar, cuando sea posible:

políticas imposibles
condiciones redundantes
condiciones contradictorias
políticas shadowed
recursos inexistentes
acciones inexistentes
atributos inexistentes
tipos incompatibles
ciclos
efectos incompatibles.
30. Policy AST

Las expresiones deberán convertirse conceptualmente en un Abstract Syntax Tree (AST).

Ejemplo:

subject.role == COACH
AND
resource.organization_id == organization.id

se representa como:

AND
├── EQUALS
│   ├── subject.role
│   └── COACH
│
└── EQUALS
    ├── resource.organization_id
    └── organization.id

Esto facilita:

validación
compilación
evaluación
caching
testing
explainability.
31. Compiled Policy

Las políticas activas podrán compilarse a una representación interna optimizada.

Stored Policy
      ↓
Validated AST
      ↓
Compiled Policy
      ↓
Policy Cache
      ↓
Evaluation

Esto evita repetir parsing en cada request.

32. Deterministic Evaluation

Para el mismo:

Policy Version
+
Authorization Context

el resultado deberá ser determinista.

Ejemplo:

Policy v7
+
Context X
=
DENY

No deberá producir:

ALLOW

en una evaluación y:

DENY

en otra sin que haya cambiado el contexto o la política.

33. Side-Effect Free Evaluation

La evaluación del DSL deberá ser:

Pure
Deterministic
Side-effect free

El Policy Engine no modificará directamente:

users
roles
permissions
organizations
sessions
MFA factors
business resources.

Su responsabilidad principal será producir una decisión.

Las acciones posteriores serán ejecutadas por servicios autorizados.

34. AI-Assisted Policy Authoring

EVOXA podrá utilizar AI para ayudar a crear políticas.

Ejemplo:

Human:
"Los coaches pueden modificar entrenamientos
de su propia organización."

AI
 ↓
Draft Policy
 ↓
Validation
 ↓
Simulation
 ↓
Human Approval
 ↓
Activation

La AI no podrá activar directamente una política crítica.

La arquitectura de seguridad del Blueprint exige protección de AI, incluyendo control de acceso y aislamiento de contexto.

35. AI Policy Interpretation

AI también podrá ayudar a explicar:

¿Por qué este usuario no puede modificar
este entrenamiento?

Pero la explicación se generará a partir de:

Decision
Policy Version
Reason Code

y no tendrá autoridad para modificar la decisión.

36. Security Boundaries

El DSL no tendrá acceso directo a:

filesystem
network
database
environment variables
secrets
KMS
process execution
shell

La policy solamente podrá acceder al contexto autorizado.

37. Version Compatibility

Una policy deberá identificar:

policy_version
dsl_version
schema_version

Ejemplo:

policy_version: 4
dsl_version: 1
schema_version: 2

Esto permitirá evolucionar el lenguaje sin invalidar automáticamente todas las políticas existentes.

38. DSL Evolution

Las nuevas versiones deberán preservar compatibilidad cuando sea posible.

DSL v1
   ↓
DSL v1.x
   ↓
DSL v2

Una breaking change deberá producir:

nueva versión de DSL
migración
validación
pruebas
compatibilidad definida
rollback posible.
39. Policy Migration

No se deberán modificar automáticamente políticas críticas sin validación.

Flujo:

Old Policy
    ↓
Migration
    ↓
New Policy
    ↓
Validation
    ↓
Simulation
    ↓
Approval
    ↓
Activation
40. Performance

El DSL deberá estar diseñado para no comprometer el objetivo de autorización:

Authorization <50 ms

y:

Permission Resolution <30 ms.

Por ello:

parsing fuera del request path
compilation fuera del request path
cache de políticas compiladas
AST optimizado
context pre-resolved
funciones limitadas
evaluación sin network calls.
41. Testing Strategy

El lenguaje tendrá tests independientes.

Parser
valid syntax
invalid syntax
Type system
valid type
invalid type
Operators
EQUALS
IN
NOT_IN
...
Logical expressions
AND
OR
NOT
Security
unknown attribute
secret access
SQL injection
code execution
cross-tenant
Semantic
ALLOW
DENY
MFA
REAUTH
Regression

Cada cambio de DSL deberá ejecutar el conjunto completo de políticas críticas.

42. Example Complete Policy

Ejemplo conceptual:

id: training-session-update
version: 1
status: ACTIVE
scope: ORGANIZATION

target:
  resource: training_session
  actions:
    - update

conditions:
  all:

    - attribute: subject.roles
      operator: CONTAINS
      value: COACH

    - attribute: subject.status
      operator: EQUALS
      value: ACTIVE

    - attribute: membership.status
      operator: EQUALS
      value: ACTIVE

    - attribute: resource.organization_id
      operator: EQUALS
      value: organization.id

    - attribute: resource.owner_id
      operator: EQUALS
      value: subject.id

effect: ALLOW

Resultado:

Coach
+
Active User
+
Active Membership
+
Same Organization
+
Resource Owner
=
ALLOW
43. Example MFA Policy
id: sensitive-training-update
version: 1

target:
  resource: sensitive_training_data
  actions:
    - update

conditions:
  all:
    - attribute: subject.roles
      operator: CONTAINS
      value: COACH

    - attribute: session.mfa_level
      operator: LESS_THAN
      value: HIGH

effect: REQUIRE_MFA

El Policy Engine determina:

REQUIRE_MFA

pero MFA Service realiza el challenge.

44. Example Cross-Tenant Attempt
subject.organization_id = ORG-A

resource.organization_id = ORG-B

Resultado:

DENY

independientemente de que:

subject.role = ADMIN

salvo que exista una autorización explícita de plataforma para una operación cross-tenant.

Esto es importante porque ADMIN no significa automáticamente GLOBAL ACCESS.

45. Alternatives Considered
A. Python-like DSL

Rejected.

Permitiría demasiadas capacidades y aumentaría el riesgo de:

code execution
side effects
imports
network calls
filesystem access.
B. SQL as Policy Language

Rejected.

SQL no representa adecuadamente la semántica de autorización y podría generar una superficie de ataque innecesaria.

C. Free-form JSON Logic

Partially rejected.

Puede ser útil como representación interna, pero por sí solo no define:

governance
types
security semantics
registries
policy lifecycle
authorization semantics.

Podrá inspirar la representación interna, pero EVOXA necesitará una semántica propia bien definida.

D. Hard-coded Policies

Rejected as primary strategy.

Sería demasiado rígido para:

organizations
enterprise customers
delegated administration
contextual security
evolving requirements.
E. Arbitrary Expression Engine

Rejected.

No se permitirá introducir un motor de expresiones sin sandboxing y límites explícitos.

46. Consequences
Positive
Lenguaje seguro.
Policies legibles.
Validación previa.
Tipado.
Versionado.
Simulación.
Compilación.
Performance.
Explainability.
Evolución futura.
AI-assisted authoring.
Menor riesgo de authorization bypass.
Negative
Necesidad de desarrollar y mantener DSL.
Mayor inversión inicial.
Necesidad de definir operadores y tipos.
Necesidad de mantener compatibilidad.
Mayor esfuerzo de testing.
Requiere governance estricta.
47. Architectural Non-Negotiables
El DSL es declarativo.
No ejecuta código arbitrario.
No ejecuta SQL arbitrario.
No realiza network calls arbitrarios.
No accede directamente a secretos.
No modifica estado durante evaluación.
Las expresiones son tipadas.
Los atributos pertenecen al Attribute Registry.
Recursos y acciones pertenecen a registros autorizados.
Las políticas deben validarse antes de activarse.
Las políticas activas deben estar versionadas.
La evaluación debe ser determinista.
Cross-tenant access debe estar denegado por defecto.
AI no puede activar políticas críticas sin governance.
INDETERMINATE debe fallar cerrado.
Las políticas no pueden otorgar privilegios fuera del modelo de autorización.
El DSL no debe convertirse en un lenguaje de programación general.
48. Traceability
Artifact	Relationship
BP-0002	Architecture lifecycle
ESP-0001	Identity
Security & Permissions	RBAC / ABAC / Context
ADR-IDENTITY-005	Authorization model
ADR-IDENTITY-043	Multi-Organization / Tenant
ADR-IDENTITY-044	Roles / Delegation / Fine-Grained Access
ADR-IDENTITY-045	Policy Evaluation
ADR-IDENTITY-046	Policy Schema & Governance
ADR-IDENTITY-047	Policy Engine Technology
ADR-IDENTITY-048	Policy DSL & Expression Language
IS-IDENTITY-006	Authorization
IS-IDENTITY-011	Security Policy Engine
IS-IDENTITY-016	Testing
49. Dependencies

Este ADR depende de:

ADR-IDENTITY-005
ADR-IDENTITY-043
ADR-IDENTITY-044
ADR-IDENTITY-045
ADR-IDENTITY-046
ADR-IDENTITY-047

Y tendrá impacto directo en:

Policy Repository
Policy Compiler
Policy Validator
Authorization Context
Authorization Evaluator
Policy Simulation
Policy API
Audit
Cache
Testing.
50. Pending Decisions

Todavía deben definirse en ADRs/Technical Tasks posteriores:

Sintaxis exacta del DSL.
JSON vs YAML para authoring.
AST definitivo.
Type system completo.
Operator catalog definitivo.
Function catalog.
Date/time semantics.
Duration semantics.
Collection semantics.
Null semantics.
Error semantics.
INDETERMINATE semantics.
Policy composition formal.
Conflict resolution algorithm.
Policy inheritance.
Policy references.
Policy imports.
Schema versioning.
DSL compiler.
Runtime evaluator.
Performance optimization.
Policy simulation language.
AI policy generation contract.
51. Acceptance Criteria

El ADR será considerado satisfecho cuando:

 Existe una definición formal del DSL.
 El DSL es declarativo.
 Existe un modelo tipado.
 Existe un Attribute Registry.
 Existe Resource Registry.
 Existe Action Registry.
 Existe catálogo de operadores.
 Existe catálogo de funciones.
 Existe AST o representación equivalente.
 Existe validación estática.
 Existe compilación.
 Existe evaluación determinista.
 No existe ejecución arbitraria.
 No existe SQL arbitrario.
 No existen llamadas de red desde policies.
 Existe control de versiones.
 Existe simulación.
 Existe testing.
 Existe protección cross-tenant.
 Existe fail-closed.
 Existe integración con RBAC/ABAC/ownership/delegation.
 Existe integración con MFA/risk/session context.
 Existe integración con audit/observability.
 AI puede asistir pero no bypass.
 El lenguaje cumple los objetivos de performance.
52. Architectural Outcome

Con ADR-IDENTITY-047 y ADR-IDENTITY-048 ya tenemos definida una parte muy importante de la arquitectura de autorización:

                    AUTHORIZATION
                         REQUEST
                            │
                            ▼
                    ┌───────────────┐
                    │      PEP      │
                    └───────┬───────┘
                            │
                            ▼
                 Authorization Context
                            │
                            ▼
                    ┌───────────────┐
                    │      PDP      │
                    └───────┬───────┘
                            │
                            ▼
                    Policy Repository
                            │
                            ▼
                      Policy DSL
                            │
                            ▼
                    Schema Validation
                            │
                            ▼
                         AST
                            │
                            ▼
                       Compiler
                            │
                            ▼
                       Evaluator
                            │
                            ▼
                 ┌───────────────────┐
                 │ Authorization     │
                 │ Decision          │
                 └─────────┬─────────┘
                           │
             ┌─────────────┼─────────────┐
             ▼             ▼             ▼
           ALLOW          DENY        REQUIRE_MFA

La decisión arquitectónica central es:

EVOXA utilizará un DSL declarativo, estructurado, tipado y restringido para representar políticas de autorización. Las políticas serán datos versionados que podrán validarse, simularse y compilarse antes de su activación. El lenguaje no tendrá capacidad de ejecutar código arbitrario, SQL, llamadas de red ni operaciones con side effects.

Esto nos deja en una posición adecuada para pasar a la siguiente decisión arquitectónica: cómo se implementará formalmente el runtime que interpreta ese DSL.

Próximo ADR recomendado

ADR-IDENTITY-049 — Authorization Policy Evaluation Runtime & Decision Algorithm

Ahí definiremos exactamente cómo el PDP procesa una policy, incluyendo:

Policy Loading
      ↓
Policy Matching
      ↓
Condition Evaluation
      ↓
RBAC
      ↓
ABAC
      ↓
Ownership
      ↓
Delegation
      ↓
Tenant Isolation
      ↓
Policy Precedence
      ↓
Conflict Resolution
      ↓
Obligations
      ↓
Final Decision

Ese ADR será especialmente importante porque será el puente entre el DSL que acabamos de definir y el código real del Policy Engine.
