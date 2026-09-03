ADR-IDENTITY-047 — Identity Authorization Policy Engine Technology & Implementation Strategy

Status: Proposed
Version: 1.0
Date: 2026-09-03
Domain: Identity / Authorization
Priority: Critical
Type: Architecture / Security / Technology Decision

1. Context

EVOXA define Authorization como una capacidad central del Identity Domain, combinando:

RBAC
ABAC
Context-Aware Policies
Organization / Tenant Isolation
Resource Ownership
Delegation
Security Policies

El Blueprint establece que las políticas deben evaluar, entre otros elementos:

Role
Organization
Tenant
Resource Ownership
Device Trust
Risk Level
Session State
Time Restrictions

Además, el pipeline de autorización contempla:

Request
   ↓
Authentication
   ↓
Role Resolution
   ↓
Policy Evaluation
   ↓
Permission Check
   ↓
Execution

El Blueprint también exige que Identity sea un Core Domain independiente, utilizado por todos los dominios de negocio mediante contratos estables.

ADR-IDENTITY-045 definió el modelo conceptual del Authorization Policy Engine, mientras que ADR-IDENTITY-046 definió la estructura y gobernanza de las políticas.

Este ADR determina cómo será implementado técnicamente ese Policy Engine.

2. Problem Statement

EVOXA necesita un mecanismo de autorización que:

Sea centralizado.
Sea determinista.
Sea seguro por defecto.
Soporte RBAC + ABAC + ownership + delegation.
Permita políticas configurables.
Permita versionado.
Permita simulación y testing.
Sea auditable.
No permita ejecución arbitraria de código.
Sea suficientemente rápido para cumplir los objetivos de autorización.
Pueda evolucionar sin acoplar los dominios de negocio al motor.
Pueda ser utilizado también por servicios y AI Agents.

El Blueprint establece como objetivo una autorización inferior a 50 ms, y una resolución de permisos inferior a 30 ms.

3. Decision

EVOXA adoptará un Central Authorization Policy Engine implementado inicialmente como un componente del Identity Domain, con una arquitectura preparada para evolucionar posteriormente hacia un Policy Decision Point (PDP) independiente si las necesidades de escala, aislamiento o distribución lo requieren.

La implementación se dividirá conceptualmente en:

                 ┌──────────────────────┐
                 │ Authorization Request│
                 └──────────┬───────────┘
                            ↓
                 ┌──────────────────────┐
                 │ Policy Enforcement   │
                 │ Point (PEP)          │
                 └──────────┬───────────┘
                            ↓
                 ┌──────────────────────┐
                 │ Authorization        │
                 │ Context Builder      │
                 └──────────┬───────────┘
                            ↓
                 ┌──────────────────────┐
                 │ Policy Decision Point│
                 │ (PDP)                │
                 └──────────┬───────────┘
                            ↓
                 ┌──────────────────────┐
                 │ Policy Repository    │
                 └──────────────────────┘

La separación conceptual entre PEP y PDP será obligatoria aunque inicialmente puedan ejecutarse dentro del mismo proceso.

4. Initial Technology Strategy

La primera implementación utilizará un Policy Engine propio dentro de Identity, utilizando:

Python
FastAPI
Pydantic / modelos tipados
PostgreSQL
Redis como cache auxiliar
Event / Outbox architecture
Testing automatizado

Esta elección se considera una decisión de implementación inicial, no una obligación permanente del Blueprint.

El Blueprint establece Python/FastAPI como baseline tecnológico de referencia, pero permite que las decisiones tecnológicas evolucionen. La arquitectura también exige mantener la separación Presentation → API → Application → Domain → Infrastructure.

5. Policy Engine Architecture

La arquitectura interna será:

identity/
└── authorization/
    ├── domain/
    │   ├── policy.py
    │   ├── policy_rule.py
    │   ├── policy_condition.py
    │   ├── authorization_context.py
    │   ├── authorization_decision.py
    │   └── policy_obligation.py
    │
    ├── application/
    │   ├── authorize.py
    │   ├── validate_policy.py
    │   ├── simulate_policy.py
    │   ├── activate_policy.py
    │   └── rollback_policy.py
    │
    ├── infrastructure/
    │   ├── policy_repository.py
    │   ├── policy_cache.py
    │   └── policy_compiler.py
    │
    └── api/
        ├── authorization.py
        └── policies.py

La estructura exacta del repositorio seguirá siendo validada durante las Technical Tasks; el punto importante es preservar las capas arquitectónicas.

6. Policy Representation

Las políticas se almacenarán como datos estructurados, no como código ejecutable.

Conceptualmente:

{
  "id": "policy.training.read",
  "version": 3,
  "status": "ACTIVE",
  "scope": "ORGANIZATION",
  "effect": "ALLOW",
  "target": {
    "resource": "training_session",
    "actions": ["read"]
  },
  "conditions": [
    {
      "attribute": "subject.role",
      "operator": "IN",
      "value": ["COACH"]
    }
  ]
}

Este formato es conceptual.

El formato definitivo podrá ser JSON, YAML u otra representación interna, pero deberá cumplir las reglas establecidas en ADR-IDENTITY-046.

7. No Arbitrary Code Execution

El Policy Engine NO ejecutará código arbitrario definido por administradores o usuarios.

No se permitirá:

eval()
exec()
SQL dinámico
Python dinámico
JavaScript dinámico
scripts arbitrarios

Las condiciones deberán utilizar:

operadores conocidos
atributos permitidos
tipos definidos
funciones permitidas
recursos conocidos
acciones conocidas

Ejemplo:

subject.role == "COACH"
AND
resource.organization_id == context.organization_id
AND
resource.owner_id == subject.user_id

Pero nunca:

execute(user_defined_python)

Esto es fundamental para evitar que el sistema de autorización se convierta en una superficie de ejecución remota.

8. Policy Evaluation Pipeline

El proceso será:

Authorization Request
        ↓
Authenticate Actor
        ↓
Resolve Organization
        ↓
Resolve Membership
        ↓
Resolve Roles
        ↓
Resolve Permissions
        ↓
Build Authorization Context
        ↓
Load Applicable Policies
        ↓
Evaluate Tenant Isolation
        ↓
Evaluate RBAC
        ↓
Evaluate Ownership
        ↓
Evaluate ABAC
        ↓
Evaluate Security Policies
        ↓
Evaluate Risk / MFA / Session
        ↓
Resolve Conflicts
        ↓
Authorization Decision
        ↓
Obligations
        ↓
Audit / Telemetry

Esto mantiene la lógica del Blueprint, donde RBAC, ABAC y Policy Engine forman parte del pipeline de autorización.

9. Authorization Context

El motor no consultará directamente cualquier tabla del sistema.

Recibirá un AuthorizationContext estructurado.

Conceptualmente:

AuthorizationContext

actor
 ├── id
 ├── type
 ├── status
 └── authentication

organization
 ├── id
 └── status

membership
 ├── id
 ├── status
 └── roles

session
 ├── id
 ├── status
 ├── device
 └── authentication_assurance

request
 ├── ip
 ├── method
 ├── endpoint
 ├── time
 └── correlation_id

resource
 ├── type
 ├── id
 ├── organization_id
 ├── owner_id
 └── classification

environment
 ├── risk
 ├── device_trust
 └── location

El contexto deberá aplicar data minimization.

No se enviarán al Policy Engine datos que no sean necesarios para tomar la decisión.

10. Attribute Registry

Los atributos utilizables por políticas deberán pertenecer a un Attribute Registry controlado.

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

resource.type
resource.id
resource.owner_id
resource.organization_id

request.ip
request.method

environment.risk_level
environment.device_trust
environment.timestamp

No se permitirá que una política invente atributos arbitrarios.

Esto permite:

validación
documentación
testing
seguridad
performance
compatibilidad futura.
11. Resource & Action Registry

Los recursos y acciones también deberán estar registrados.

Ejemplo:

Resource:
training_session

Actions:
training_session.read
training_session.create
training_session.update
training_session.delete
training_session.share

Las políticas solamente podrán utilizar recursos y acciones registrados.

Esto evita errores como:

training.readEverything

cuando dicha acción no existe realmente.

12. Policy Compiler / Validator

Antes de activar una política:

Draft
  ↓
Syntax Validation
  ↓
Schema Validation
  ↓
Attribute Validation
  ↓
Resource Validation
  ↓
Action Validation
  ↓
Type Validation
  ↓
Conflict Analysis
  ↓
Simulation
  ↓
Approval
  ↓
Activation

Una política inválida no puede llegar al estado ACTIVE.

13. Policy Repository

PostgreSQL será el almacenamiento durable de las políticas.

Conceptualmente:

security_policies
security_policy_versions
policy_rules
policy_conditions
policy_targets
policy_obligations
policy_audit

Se conservará historial de versiones.

Una actualización no deberá sobrescribir destructivamente una política activa anterior.

Ejemplo:

policy.training.read

v1 → SUPERSEDED
v2 → SUPERSEDED
v3 → ACTIVE
14. Policy Cache

Redis podrá utilizarse para acelerar la evaluación.

Arquitectura:

PostgreSQL
     │
     ▼
Policy Repository
     │
     ▼
Policy Cache
     │
     ▼
Policy Engine

Redis no será la fuente de verdad.

Si Redis falla:

Redis unavailable
       ↓
Fallback
       ↓
Durable policy source

Nunca:

Redis unavailable
       ↓
ALLOW

El principio será fail closed para decisiones críticas.

15. Cache Invalidation

Los cambios de:

roles
permissions
membership
organization
policy
delegation
MFA assurance
security state

deberán invalidar los datos de autorización afectados.

Conceptualmente:

Policy Updated
      ↓
Transaction
      ↓
Outbox Event
      ↓
Authorization Cache Invalidation
      ↓
New Policy Version
      ↓
PDP Uses New Version

Esto conecta directamente el Policy Engine con la arquitectura de eventos de EVOXA.

16. Policy Version Consistency

Cada decisión deberá poder identificar la versión de política utilizada.

Ejemplo:

policy_id:
training.coach.read

policy_version:
7

Esto es fundamental para:

auditoría
debugging
incident response
reproducción de decisiones
compliance
análisis AI.
17. Decision Object

El Policy Engine devolverá una estructura conceptual:

AuthorizationDecision

decision
 ├── ALLOW
 ├── DENY
 ├── CHALLENGE
 ├── REAUTHENTICATE
 ├── REQUIRE_MFA
 └── REVOKE

reason_code
policy_id
policy_version
obligations
evaluation_metadata

No se devolverá información sensible de las políticas internas al cliente final.

18. Explainability

El sistema deberá diferenciar entre:

Internal explanation

Información detallada para:

Security Admin
auditoría
debugging autorizado
incident response
External reason

Información mínima para el consumidor API.

Ejemplo:

{
  "decision": "DENY",
  "reason_code": "INSUFFICIENT_PERMISSION"
}

No:

{
  "policy": "...internal security policy...",
  "secret_condition": "...",
  "risk_algorithm": "..."
}

La explicación nunca debe convertirse en una herramienta para evadir controles.

19. Policy Decision Caching

Podrán cachearse decisiones cuando sea seguro.

Sin embargo, la cache deberá invalidarse ante cambios críticos.

Por ejemplo:

User suspended
      ↓
Invalidate user authorization cache
      ↓
Existing cached ALLOW removed

La autorización no podrá permanecer válida indefinidamente por una decisión cacheada.

20. Performance Strategy

El Policy Engine deberá diseñarse para cumplir:

Métrica	Objetivo
Authorization	< 50 ms
Permission Resolution	< 30 ms

Estos objetivos provienen del Blueprint de Security & Permissions.

Para conseguirlo:

políticas prevalidada
estructuras tipadas
cache
índices PostgreSQL
eliminación de consultas innecesarias
resolución eficiente de roles
evaluación determinista
evitar llamadas de red innecesarias
evaluación local cuando sea posible.
21. No Network Call per Authorization

El camino crítico de autorización no debería depender de múltiples servicios externos.

Evitar:

API
 ↓
Identity
 ↓
Policy Service
 ↓
Role Service
 ↓
Organization Service
 ↓
Risk Service
 ↓
Database

como flujo obligatorio para cada request.

Preferido:

API
 ↓
Identity Authorization
 ↓
Cached / Local Policy
 ↓
Decision

Los datos necesarios deberán estar disponibles de manera eficiente y segura.

22. Embedded PDP vs Independent PDP
Opción inicial
Identity Service
 ├── PEP
 └── PDP
Evolución futura
Applications
     │
     ▼
   PEP
     │
     ▼
Policy Decision Service
     │
     ├── Policy Repository
     └── Policy Cache

La primera opción se adopta inicialmente porque reduce:

complejidad
latencia
infraestructura
superficie operacional.

La segunda queda preparada para una evolución futura.

23. Domain Independence

Los dominios de negocio no implementarán sus propias políticas de Identity.

Incorrecto:

Training
 └── if user.role == "COACH"

Correcto:

Training
    ↓
Authorization Contract
    ↓
Identity Authorization
    ↓
Decision

Esto mantiene Identity como propietario exclusivo de autenticación y autorización.

Los dominios pueden seguir teniendo business rules, pero no deben duplicar las reglas de Identity.

24. Authorization vs Business Rules

Debe mantenerse una separación estricta.

Authorization
¿Puede este actor realizar esta operación?
Business Rule
¿La operación está permitida según las reglas del negocio?

Ejemplo:

Authorization:
Coach puede actualizar entrenamiento.

Business Rule:
El entrenamiento solamente puede modificarse
hasta 24 horas antes de su ejecución.

Ambas reglas son necesarias, pero pertenecen a capas diferentes.

25. AI Agent Authorization

Los AI Agents utilizarán exactamente el mismo sistema de autorización.

AI Agent
   ↓
Agent Identity
   ↓
Authorization Context
   ↓
Policy Engine
   ↓
Decision
   ↓
Tool / API

Un AI Agent no tendrá:

bypass de autorización
super-permission implícito
acceso global por defecto
acceso directo al Policy Repository
capacidad de modificar políticas críticas.

El Blueprint considera AI Service y System Service como identidades diferenciadas.

26. Policy Administration

Las operaciones:

Create Policy
Update Policy
Validate Policy
Simulate Policy
Approve Policy
Activate Policy
Deactivate Policy
Rollback Policy

serán operaciones altamente protegidas.

Deberán requerir:

autorización
tenant context
audit
versioning
validación
y, para políticas críticas, posiblemente MFA/step-up.
27. Emergency Policies

Se permitirá contemplar políticas de emergencia.

Ejemplo:

Emergency Deny

para bloquear inmediatamente una capacidad comprometida.

Pero:

Emergency Policy ≠ bypass de governance.

Debe existir:

autorización específica
motivo
duración
auditoría
identificación del actor
posterior revisión.
28. Security Events

Las decisiones de seguridad relevantes producirán telemetry/audit según corresponda.

Ejemplos:

authorization.denied
authorization.policy_changed
authorization.policy_activated
authorization.policy_rollback
authorization.cross_tenant_attempt
authorization.privilege_escalation

El Blueprint exige auditoría de acciones de seguridad y considera específicamente los intentos de acceso cross-tenant como telemetry de seguridad.

29. Observability

El Policy Engine deberá exponer métricas como:

authorization_requests_total
authorization_allowed_total
authorization_denied_total
authorization_latency
policy_evaluation_latency
policy_cache_hit_ratio
policy_cache_miss_ratio
policy_validation_failures
policy_activation_total
policy_rollback_total
authorization_errors_total

No deberán utilizarse labels de cardinalidad excesivamente alta.

Nunca:

user_id
resource_id
token
email

como labels métricos sin una justificación explícita.

30. Failure Strategy
Policy Repository unavailable
Critical authorization
        ↓
Fail Closed
Redis unavailable
Use durable source / safe fallback
Risk service unavailable

La decisión dependerá de la política.

Nunca se asumirá automáticamente:

Risk unavailable = ALLOW
Policy invalid
Policy = REJECTED
Unknown policy
DENY / INDETERMINATE

Nunca:

unknown = ALLOW
31. Policy Testing

Cada política deberá poder probarse independientemente.

Ejemplo:

Given:
  actor.role = COACH

And:
  actor.organization = ORG-A

And:
  resource.organization = ORG-A

And:
  resource.owner = actor

When:
  action = training_session.update

Then:
  decision = ALLOW

Caso negativo:

Given:
  actor.organization = ORG-A

And:
  resource.organization = ORG-B

When:
  action = training_session.update

Then:
  decision = DENY

Los tests de autorización deberán incluir especialmente cross-tenant isolation, ya que el Blueprint prohíbe el acceso entre tenants salvo autorización explícita.

32. Policy Simulation

Antes de activar una política:

Policy Draft
      ↓
Simulation
      ↓
Existing Requests / Synthetic Cases
      ↓
Expected Decisions
      ↓
Impact Analysis

Esto permitirá detectar:

permisos inesperadamente ampliados
permisos eliminados
shadowed policies
conflictos
cross-tenant access
cambios de comportamiento.
33. Policy Shadowing

El sistema deberá detectar situaciones donde una política nunca puede ejecutarse debido a otra regla.

Ejemplo:

DENY all training
      ↓
ALLOW coach training

Si el orden de evaluación hace que el ALLOW nunca sea alcanzable, deberá marcarse como posible shadowed policy.

No necesariamente será un error, pero deberá ser visible.

34. Alternatives Considered
A. Authorization únicamente con RBAC

Rechazada.

No permite representar adecuadamente:

ownership
tenant
device trust
risk
time restrictions
contextual policies.

El Blueprint requiere RBAC + ABAC + Context-Aware Policies.

B. Policy logic directamente en Controllers

Rechazada.

Generaría:

duplicación
inconsistencia
difícil auditoría
difícil testing
riesgo de bypass.
C. Policy logic dentro de cada Domain

Rechazada.

Contradice la propiedad central de Identity sobre autorización.

D. SQL-based Authorization

Rechazada como motor principal.

SQL puede utilizarse para recuperar datos, pero no deberá convertirse en lenguaje de políticas.

E. Arbitrary scripting

Rechazada.

Ejecutar código proporcionado por usuarios/admins dentro del Policy Engine sería una superficie de seguridad inaceptable.

F. External PDP desde el primer día

No adoptada inicialmente.

Puede ser apropiada posteriormente, pero agrega:

infraestructura
latencia
complejidad operacional
disponibilidad adicional.

La arquitectura quedará preparada para evolucionar hacia ella.

35. Consequences
Positive
Authorization centralizada.
RBAC + ABAC + ownership.
Tenant isolation.
Policies versionadas.
Auditabilidad.
Simulación.
Testing.
Explainability controlada.
AI-compatible.
Evolución hacia PDP independiente.
Menor duplicación entre dominios.
Negative
Identity se convierte en componente crítico.
Mayor complejidad que RBAC simple.
Requiere governance.
Requiere cache/invalidation.
Requiere testing exhaustivo.
Las políticas necesitan versionado y control de cambios.
Requiere fuerte disciplina arquitectónica.
36. Architectural Non-Negotiables
Authorization fail-closed.
No arbitrary code execution.
No cross-tenant access by default.
No secrets inside policies.
No health data unnecessarily inside authorization context.
No authorization duplicated across business domains.
No direct user permission grants unless explicitly governed.
No policy activation without validation.
No destructive policy version overwrite.
Every critical policy change is auditable.
AI cannot bypass the Policy Engine.
Redis is not the source of truth.
Unknown/indeterminate authorization cannot become ALLOW.
Policy evaluation must remain deterministic.
Tenant context must always be explicit and validated.
37. Traceability
Source	Relationship
BP-0002 Architecture Map	Lifecycle and architectural traceability
ESP-0001 Identity	Identity authorization foundation
Security & Permissions	RBAC + ABAC + Context Policies
ADR-IDENTITY-005	Authorization model
ADR-IDENTITY-043	Multi-organization / tenant strategy
ADR-IDENTITY-044	Roles, delegation and fine-grained access
ADR-IDENTITY-045	Policy evaluation architecture
ADR-IDENTITY-046	Policy language, schema and governance
ADR-IDENTITY-047	Technology and implementation strategy
IS-IDENTITY-006	Authorization & Permission Evaluation
IS-IDENTITY-007	Role & Permission Management
IS-IDENTITY-011	Security Policy Engine
IS-IDENTITY-012	Audit & Security Events

La Architecture Map establece que la implementación debe mantener la trazabilidad:

Blueprint
   ↓
Engineering Standards
   ↓
ESP
   ↓
IS
   ↓
Technical Tasks
   ↓
Source Code
   ↓
Testing

y explícitamente indica que ninguna implementación debe saltarse este lifecycle.

38. Dependencies

Este ADR depende principalmente de:

ADR-IDENTITY-005
ADR-IDENTITY-006
ADR-IDENTITY-007
ADR-IDENTITY-011
ADR-IDENTITY-022
ADR-IDENTITY-026
ADR-IDENTITY-028
ADR-IDENTITY-043
ADR-IDENTITY-044
ADR-IDENTITY-045
ADR-IDENTITY-046

Y será utilizado directamente por:

IS-006
IS-007
IS-011
IS-012
IS-014
IS-015
IS-016
IS-017
39. Pending Decisions

Este ADR define la arquitectura tecnológica, pero todavía deben cerrarse decisiones específicas:

DSL definitivo.
JSON vs YAML como formato de authoring.
Expression language.
Implementación exacta del evaluator.
Compilación de políticas.
Repository schema definitivo.
Cache keys.
Cache TTL.
Cache invalidation.
Policy conflict algorithm.
Precedence formal.
Policy inheritance.
Policy composition.
Obligation execution.
Simulation engine.
Policy testing framework.
PEP/PDP interfaces definitivas.
Future external PDP strategy.
Policy distribution.
Emergency policy activation.
Reason-code catalog.
Explainability model.
AI-assisted policy analysis.

Estas decisiones deberán resolverse mediante Technical Tasks y, cuando tengan impacto arquitectónico independiente, nuevos ADR.

40. Acceptance Criteria

Este ADR se considera correctamente implementado cuando:

 Existe un Policy Engine centralizado.
 PEP y PDP están conceptualmente separados.
 Las políticas utilizan representación estructurada.
 No existe ejecución arbitraria de código.
 Existe Policy Validation.
 Existe Policy Versioning.
 Existe Policy Repository.
 Existe cache segura.
 Existe invalidación de cache.
 Existe Authorization Context tipado.
 Existe Attribute Registry.
 Existe Resource/Action Registry.
 Existe Policy Simulation.
 Existe conflict/shadow detection.
 Existe auditabilidad.
 Existe observabilidad.
 Existe testing unitario e integración.
 Existe protección cross-tenant.
 Existe fail-closed.
 AI Agents pasan por la misma autorización.
 Los dominios de negocio no duplican Identity Authorization.
 El diseño puede evolucionar a un PDP independiente.
 Se validan los objetivos de rendimiento del Blueprint.
41. Architectural Outcome

Con este ADR queda definida la dirección tecnológica del Identity Authorization Policy Engine:

                    EVOXA REQUEST
                          │
                          ▼
                ┌───────────────────┐
                │ Authentication    │
                └─────────┬─────────┘
                          ▼
                ┌───────────────────┐
                │ PEP               │
                └─────────┬─────────┘
                          ▼
                ┌───────────────────┐
                │ Context Builder   │
                └─────────┬─────────┘
                          ▼
                ┌───────────────────┐
                │       PDP         │
                │                   │
                │ RBAC              │
                │ ABAC              │
                │ Ownership         │
                │ Delegation        │
                │ Tenant Isolation  │
                │ Security Policies │
                │ Risk / MFA        │
                └─────────┬─────────┘
                          ▼
                ┌───────────────────┐
                │ Decision          │
                │ ALLOW / DENY /    │
                │ CHALLENGE / MFA   │
                └─────────┬─────────┘
                          ▼
                ┌───────────────────┐
                │ Audit / Telemetry │
                └───────────────────┘

La decisión principal es, por tanto:

EVOXA implementará inicialmente el Authorization Policy Engine dentro del Identity Domain como un componente centralizado, con PEP/PDP desacoplados conceptualmente, políticas estructuradas y versionadas, evaluación determinista, cache segura, fail-closed y una arquitectura preparada para evolucionar hacia un PDP independiente.

Esto mantiene coherencia con el Blueprint, que define Identity como propietario central de autenticación y autorización y exige que la seguridad esté integrada transversalmente en la plataforma.

Próximo ADR recomendado

El siguiente paso lógico es:

ADR-IDENTITY-048 — Authorization Policy DSL & Expression Language Strategy

Ahí deberíamos decidir cómo se escriben realmente las políticas, incluyendo:

Policy
 ├── Target
 ├── Conditions
 ├── Operators
 ├── Attributes
 ├── Functions
 ├── Effects
 ├── Obligations
 └── Variables

y definir algo especialmente importante: qué sintaxis utilizará EVOXA para expresar condiciones como subject.role, resource.owner_id, organization.id, risk_level, mfa_level, etc., antes de pasar a las Technical Tasks y posteriormente al código.
