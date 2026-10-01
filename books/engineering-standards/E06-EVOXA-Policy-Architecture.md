E06 — EVOXA Policy Architecture
Engineering Specification

Pertenece a: Engineering Specification
Volumen: volume-09-engineering
Capítulo: E06
Anterior: E05 — Authorization Architecture
Siguiente: E07 — EVOXA Service Architecture

1. Propósito

E06 define la arquitectura de Policies de EVOXA.

Mientras E05 establece cómo se autoriza una operación, E06 define cómo se expresan, almacenan, versionan, evalúan y gobiernan las reglas que producen esa decisión.

La separación conceptual es:

Authentication
      │
      ▼
Identity
      │
      ▼
Authorization
      │
      ▼
Policy
      │
      ▼
Decision
      │
      ▼
Enforcement

La pregunta principal de E06 es:

¿Bajo qué reglas y condiciones EVOXA debe permitir, denegar o requerir una acción adicional?

2. Objetivos

Policy Architecture debe proporcionar:

Policy Definition
Policy Storage
Policy Evaluation
Policy Composition
Policy Versioning
Policy Lifecycle
Policy Testing
Policy Deployment
Policy Governance
Policy Audit
Policy Simulation
Policy Delegation
Policy Context
Policy Enforcement
3. Principio Fundamental

Una Policy no debe estar implícita dentro del código de una aplicación cuando represente una regla de seguridad, acceso o gobierno que necesite evolucionar independientemente.

La arquitectura debe permitir:

Policy
   │
   ├── Definition
   ├── Version
   ├── Scope
   ├── Conditions
   ├── Effect
   ├── Priority
   └── Lifecycle
4. Policy vs Permission

Una Permission responde:

¿Puede realizar esta acción?

Una Policy responde:

¿Puede realizar esta acción
bajo estas condiciones?

Ejemplo:

Permission:
training.workouts.update

Policy:

ALLOW
IF
subject.role = trainer
AND
resource.trainerId = subject.id
AND
resource.status != archived

Por lo tanto:

Permission
    +
Policy
    +
Context
    =
Authorization Decision
5. Policy Architecture
                       POLICY ARCHITECTURE
                              │
          ┌───────────────────┼───────────────────┐
          │                   │                   │
      Definition          Evaluation          Governance
          │                   │                   │
          ▼                   ▼                   ▼
      Policies             PDP Engine        Lifecycle
          │                   │                   │
          └───────────────────┼───────────────────┘
                              │
                              ▼
                     Authorization Decision
6. Policy Components

Cada Policy debe poder representar:

Policy
 ├── Identity
 ├── Name
 ├── Description
 ├── Effect
 ├── Actions
 ├── Resources
 ├── Conditions
 ├── Scope
 ├── Priority
 ├── Version
 ├── Status
 └── Metadata
7. Policy Identity

Cada policy debe tener un identificador estable:

policyId

Ejemplo:

training.workout.update.ownership

Debe ser independiente de su versión.

policyId:
training.workout.update.ownership

version:
3
8. Policy Name

El nombre debe ser descriptivo.

Ejemplo:

Trainer Workout Ownership Policy

Debe existir además un identificador técnico estable.

9. Policy Description

Toda policy debe documentar:

Purpose
Scope
Expected Behavior
Security Impact
Dependencies

Ejemplo:

Permits trainers to update workouts
only when the workout belongs to a
client assigned to the trainer.
10. Policy Effect

Las policies deben soportar al menos:

ALLOW
DENY

Opcionalmente:

REQUIRE_APPROVAL
REQUIRE_MFA
REQUIRE_CONFIRMATION

Sin embargo, estos últimos pueden modelarse como obligaciones asociadas a una decisión.

11. Policy Target

Una policy debe definir a qué operaciones aplica:

Subject
Action
Resource
Scope

Ejemplo:

subject = trainer
action = training.workouts.update
resource = workout
scope = tenant
12. Policy Conditions

Las condiciones representan restricciones.

Ejemplo:

subject.tenantId == resource.tenantId

Otra:

resource.ownerId == subject.id

Otra:

resource.status == draft
13. Policy Context

Las condiciones pueden utilizar información contextual:

Subject
Resource
Tenant
Request
Session
Device
Location
Time
Risk
Authentication Level
14. Policy Input

El motor de políticas debe recibir un contexto estructurado.

Ejemplo:

{
  "subject": {
    "id": "user_123",
    "roles": ["trainer"],
    "tenantId": "tenant_001"
  },
  "action": "training.workouts.update",
  "resource": {
    "type": "workout",
    "id": "workout_001",
    "tenantId": "tenant_001",
    "ownerId": "user_123",
    "status": "draft"
  },
  "context": {
    "authenticationLevel": "strong"
  }
}
15. Policy Decision

El Policy Engine debe producir una decisión estructurada:

ALLOW
DENY

acompañada internamente por:

policyId
policyVersion
reason
obligations
metadata
16. Policy Decision Model
Policy Request
      │
      ▼
Policy Resolution
      │
      ▼
Policy Evaluation
      │
      ▼
Policy Combination
      │
      ▼
Final Decision
17. Policy Decision Point

El PDP — Policy Decision Point es responsable de evaluar policies.

Application
     │
     ▼
Authorization Request
     │
     ▼
PDP
     │
     ├── Policy Store
     ├── Policy Context
     └── Policy Engine
     │
     ▼
Decision
18. Policy Enforcement Point

El PEP — Policy Enforcement Point aplica la decisión.

Request
   ↓
PEP
   ↓
PDP
   ↓
ALLOW / DENY
   ↓
Resource

La separación debe mantenerse:

PDP = Decide
PEP = Enforce
19. Policy Administration Point

Debe existir conceptualmente un:

PAP — Policy Administration Point

Responsable de:

Create Policy
Update Policy
Publish Policy
Disable Policy
Version Policy
Review Policy

Arquitectura:

                PAP
                 │
                 ▼
            Policy Store
                 │
                 ▼
                PDP
                 │
                 ▼
                PEP
20. Policy Information Point

El PIP — Policy Information Point proporciona información necesaria para evaluar policies.

Fuentes posibles:

Identity Service
Tenant Service
User Service
Resource Service
Risk Service
Device Service
Billing Service
Training Service
Nutrition Service

Ejemplo:

PDP
 │
 └── PIP
      ├── user role
      ├── tenant
      ├── resource owner
      └── subscription status
21. Policy Architecture Full Model
                     POLICY SYSTEM
                          │
       ┌──────────────────┼──────────────────┐
       │                  │                  │
      PAP                PDP                PIP
       │                  │                  │
       ▼                  │                  ▼
 Policy Store             │             Context Sources
                          │
                          ▼
                       Decision
                          │
                          ▼
                         PEP
                          │
                          ▼
                       Resource
22. Policy Store

El Policy Store contiene:

Policies
Versions
Metadata
Scopes
Status
Dependencies

Debe proporcionar:

Versioning
Consistency
Auditability
Rollback
23. Policy Repository

Conceptualmente:

policies
policy_versions
policy_bindings
policy_conditions
policy_audit
24. Policy Definition

Una policy puede representarse conceptualmente como:

{
  "id": "training.workout.update.ownership",
  "version": 1,
  "effect": "ALLOW",
  "actions": [
    "training.workouts.update"
  ],
  "resources": [
    "workout"
  ],
  "conditions": [
    {
      "attribute": "resource.trainerId",
      "operator": "equals",
      "value": "subject.id"
    }
  ]
}

El formato final puede evolucionar.

25. Policy Language

EVOXA debe evitar que cada servicio invente su propio lenguaje.

Debe existir una abstracción común:

EVOXA Policy Model

La implementación concreta puede posteriormente utilizar:

DSL
JSON
Expression Language
Policy Engine

pero la semántica debe mantenerse estable.

26. Policy DSL

Una posible representación conceptual:

POLICY training.workouts.update.ownership

WHEN
    action == "training.workouts.update"

AND
    subject.role == "trainer"

AND
    resource.trainerId == subject.id

THEN
    ALLOW

Esto facilita lectura humana.

27. Policy Conditions

Los operadores básicos deben incluir:

equals
not_equals
in
not_in
contains
starts_with
ends_with
greater_than
less_than
exists
not_exists
28. Logical Operators

Debe soportarse:

AND
OR
NOT

Ejemplo:

role == trainer
AND
tenant == resource.tenant
AND
(
    owner == subject
    OR
    assigned == subject
)
29. Policy Composition

Las policies deben poder combinarse.

Policy A
+
Policy B
+
Policy C

para producir una decisión final.

30. Policy Combination

Se recomienda una semántica determinista:

Explicit DENY
      ↓
highest precedence

Después:

ALLOW

Si ninguna policy permite:

DEFAULT DENY

Modelo:

DENY
  >
ALLOW
  >
DEFAULT DENY
31. Default Deny

Principio obligatorio:

No applicable policy
        ↓
      DENY

Nunca:

No applicable policy
        ↓
      ALLOW
32. Explicit Deny

Una policy explícita de deny debe poder bloquear una permission existente.

Ejemplo:

Role:
trainer

Permission:
clients.read

Policy:
DENY if client.status == restricted

Resultado:

DENY
33. Policy Priority

Cuando sea necesario, las policies pueden tener prioridad:

priority = 100
priority = 200
priority = 300

Debe existir una regla clara de evaluación.

No debe depender del orden accidental de almacenamiento.

34. Policy Scope

Una policy puede tener alcance:

PLATFORM
TENANT
DOMAIN
RESOURCE
USER
ROLE
SERVICE
AGENT

Ejemplo:

Tenant Policy

solamente afecta a:

Tenant X
35. Platform Policies

Ejemplos:

PlatformAdminAccess
GlobalSecurityPolicy
GlobalDataRetentionPolicy
GlobalAIUsagePolicy

Aplican globalmente.

36. Tenant Policies

Permiten personalización controlada:

Tenant X
 └── PasswordPolicy
 └── AccessPolicy
 └── AIUsagePolicy
 └── DataExportPolicy
37. Domain Policies

Ejemplo:

Training
Nutrition
Billing
Security
Analytics

Cada dominio puede tener políticas específicas.

38. Resource Policies

Ejemplo:

Workout
Client
Invoice
NutritionPlan
AIConversation

Una policy puede limitar acceso a un recurso específico.

39. Policy Inheritance

Las policies pueden heredarse:

Platform
   ↓
Tenant
   ↓
Domain
   ↓
Resource

Pero la herencia debe ser explícita.

40. Policy Override

Un tenant puede personalizar ciertas policies si la arquitectura lo permite.

Debe existir una separación:

Platform Policy
      │
      ▼
Tenant Override

Las policies que afecten seguridad crítica no deberían ser libremente anulables.

41. Immutable Security Policies

Algunas policies deben ser inmutables a nivel tenant:

CrossTenantIsolation
SystemAdminProtection
AuditIntegrity
CredentialSecurity
42. Policy Versioning

Toda modificación significativa debe crear una nueva versión.

Policy v1
   ↓
Policy v2
   ↓
Policy v3

No sobrescribir silenciosamente una policy activa.

43. Policy States

Lifecycle mínimo:

DRAFT
   ↓
TESTING
   ↓
APPROVED
   ↓
PUBLISHED
   ↓
ACTIVE
   ↓
DEPRECATED
   ↓
RETIRED
44. Policy Lifecycle
Create
  ↓
Validate
  ↓
Test
  ↓
Review
  ↓
Approve
  ↓
Publish
  ↓
Activate
  ↓
Monitor
  ↓
Deprecate
  ↓
Retire
45. Policy Approval

Policies sensibles deben requerir aprobación.

Ejemplo:

Security Policy
     ↓
Author
     ↓
Reviewer
     ↓
Security Approval
     ↓
Publish

Esto soporta Separation of Duties.

46. Policy Testing

Antes de publicar una policy:

Policy
 ↓
Test Cases
 ↓
Simulation
 ↓
Expected Decision
 ↓
Actual Decision
47. Policy Test Case

Ejemplo:

{
  "name": "Trainer can update owned workout",
  "input": {
    "subjectRole": "trainer",
    "subjectId": "user_1",
    "resourceOwnerId": "user_1"
  },
  "expected": "ALLOW"
}
48. Negative Policy Testing

También deben existir casos negativos:

Trainer
+
Workout owned by another trainer

Resultado:

DENY
49. Policy Simulation

EVOXA debe poder responder:

¿Qué habría ocurrido si esta policy estuviera activa?

Sin modificar el comportamiento real.

Simulation
   ↓
Policy Version
   ↓
Historical Requests
   ↓
Predicted Decisions
50. Policy Dry Run

Una policy nueva puede ejecutarse en:

DRY_RUN

produciendo:

Current Decision
+
New Policy Decision

Esto permite detectar impactos antes de activarla.

51. Policy Deployment

Las policies deben desplegarse controladamente:

Development
   ↓
Testing
   ↓
Staging
   ↓
Production
52. Policy Rollout

Se puede utilizar:

100%
50%
10%
Tenant subset
Role subset

para políticas de alto impacto.

53. Policy Rollback

Toda policy publicada debe poder revertirse.

v3 ACTIVE
   ↓
Problem
   ↓
Rollback
   ↓
v2 ACTIVE
54. Policy Dependencies

Una policy puede depender de:

Role
Permission
Resource Attribute
External Service
Tenant Configuration
Risk Score
Authentication Level

Las dependencias deben declararse cuando sea posible.

55. Policy Evaluation Pipeline
Request
   ↓
Normalize Context
   ↓
Resolve Policies
   ↓
Load Attributes
   ↓
Evaluate Conditions
   ↓
Combine Results
   ↓
Apply Obligations
   ↓
Return Decision
56. Context Normalization

El PDP no debería recibir estructuras diferentes por cada servicio.

Debe existir un modelo común:

Subject
Action
Resource
Context
Environment
57. Subject Context
subject.id
subject.type
subject.tenantId
subject.roles
subject.permissions
subject.authenticationLevel
58. Resource Context
resource.id
resource.type
resource.tenantId
resource.ownerId
resource.status
resource.attributes
59. Environment Context
time
ip
device
location
risk
network
requestId
60. Policy Attributes

Las policies pueden utilizar atributos:

Subject Attributes
Resource Attributes
Tenant Attributes
Environment Attributes

Esto constituye la base para ABAC.

61. Policy Attribute Sources
Identity Service
Tenant Service
Resource Service
Security Service
Risk Engine
Configuration Service

El PIP será responsable de obtener estos atributos.

62. Attribute Trust

No todos los atributos tienen el mismo nivel de confianza.

Ejemplo:

Client supplied:
userRole = admin

no debe considerarse confiable.

Debe utilizarse:

Server-side identity context
63. Policy Security Boundary

Las policies deben evaluarse únicamente con información confiable.

Client Input
      │
      ▼
Validation
      │
      ▼
Trusted Context
      │
      ▼
Policy Engine
64. Policy Obligations

Una decisión puede incluir obligaciones.

Ejemplo:

ALLOW
+
REQUIRE_MFA

o:

ALLOW
+
AUDIT_HIGH_RISK
65. Policy Advice

El motor puede devolver información adicional:

reason
requiredAction
requiredAuthentication
auditLevel

El PEP decide cómo aplicarla según la arquitectura.

66. Step-Up Authorization

Ejemplo:

User authenticated
       ↓
Request sensitive action
       ↓
Policy:
MFA required
       ↓
Step-up authentication
       ↓
Re-evaluate policy
       ↓
ALLOW
67. Risk-Based Policies

Las policies pueden incorporar riesgo:

riskScore < 30
    → ALLOW

30–70
    → STEP-UP

> 70
    → DENY

Esto permite integrar Security Architecture con Policy Architecture.

68. Time-Based Policies

Ejemplo:

ALLOW
IF
supportAccess == true
AND
currentTime < expiration
69. Location-Based Policies

Puede existir:

ALLOW
IF
location.country IN approvedCountries

siempre que exista una necesidad legítima y una fuente confiable de ubicación.

70. Device Policies

Ejemplo:

ALLOW
IF
device.trusted == true

para operaciones de alto riesgo.

71. Subscription Policies

Billing puede proporcionar atributos:

subscription.plan
subscription.status
subscription.features

Ejemplo:

ALLOW AI advanced
IF
subscription.features.aiAdvanced == true
72. Feature Policies

Las políticas también pueden controlar features:

Feature
 ↓
Entitlement
 ↓
Policy
 ↓
ALLOW / DENY

Esto conecta Policy Architecture con Billing y Product Architecture.

73. Policy and Entitlements

No confundir:

Entitlement
→ What the customer has purchased

Policy
→ Under what conditions it can be used

Ejemplo:

Plan:
AI Pro

Entitlement:
advanced_ai

Policy:
maximum 100 AI operations/day
74. Rate Policies

Las policies pueden establecer límites:

AI operations
Exports
API requests
Reports

Aunque los límites de infraestructura deben continuar siendo responsabilidad del rate limiting system.

75. Data Access Policies

Ejemplo:

ALLOW nutrition.plan.read
IF
subject.id == resource.ownerId
OR
subject.id IN resource.authorizedTrainers
76. Data Export Policies

Las exportaciones pueden requerir:

Permission
+
Tenant Policy
+
MFA
+
Audit
77. Privacy Policies

Ejemplo:

Personal Data Export

puede requerir:

subject == dataOwner

o:

privacy.admin

según el caso.

78. AI Safety Policies

Las políticas también deben controlar agentes y AI:

AI Tool
 ↓
Risk Classification
 ↓
Policy Evaluation
 ↓
ALLOW
DENY
REQUIRE_CONFIRMATION
79. Agent Policies

Ejemplo:

ALLOW agent.create_workout
IF
user has training.workouts.create
AND
agent has training.workouts.create
AND
workout belongs to user's tenant
80. Policy Delegation

Una policy puede permitir delegación limitada:

User
 ↓
Delegates
 ↓
Assistant
 ↓
Policy

La delegación nunca debe ampliar los permisos originales.

81. Policy Boundary

Principio:

Delegated Access
≤
Original Access

y:

Agent Access
≤
Granted Agent Permissions
82. Policy Audit

Cada cambio significativo debe registrar:

policyId
version
actor
action
timestamp
reason
previousState
newState
83. Policy Evaluation Audit

Para operaciones de alto riesgo:

subject
action
resource
policy
version
decision
timestamp
requestId

debe ser trazable.

84. Privacy of Policy Logs

Los logs no deben contener:

Passwords
Tokens
Secrets
Sensitive Payloads
85. Policy Metrics

Métricas:

policy_evaluation_total
policy_allow_total
policy_deny_total
policy_error_total
policy_evaluation_latency
policy_version_active
policy_changes_total
policy_rollbacks_total
86. Policy Performance

La evaluación debe ser rápida.

Optimización:

Policy Cache
Attribute Cache
Compiled Policies
Decision Cache

siempre respetando invalidación y consistencia.

87. Policy Cache Invalidation

Cambios en:

Policy
Role
Permission
Tenant Configuration
Resource Ownership

pueden invalidar decisiones previamente almacenadas.

88. Distributed Policy Cache

En arquitectura distribuida:

Policy Store
      ↓
Policy Distribution
      ↓
Service Nodes
      ↓
Local Policy Cache

Debe existir control de versión:

policyVersion
89. Policy Consistency

Un nodo que utiliza una versión antigua debe poder identificarse.

Ejemplo:

Node A → policy v4
Node B → policy v3

No debería permitirse indefinidamente una divergencia en policies críticas.

90. Policy Availability

El PDP debe diseñarse para alta disponibilidad:

PDP Instance 1
PDP Instance 2
PDP Instance 3

con acceso al Policy Store.

91. Policy Failure

Ante una falla:

Policy Engine unavailable

la decisión debe ser:

DENY

para operaciones sensibles.

No:

ALLOW because PDP is unavailable
92. Policy Resilience

Debe existir:

Timeout
Circuit Breaker
Retry
Cache
Fallback

pero el fallback debe mantener la postura de seguridad.

93. Policy Governance

Las policies deben estar sujetas a:

Ownership
Review
Approval
Versioning
Testing
Audit
Retirement
94. Policy Ownership

Cada policy crítica debe tener:

owner
domain
securityClassification

Ejemplo:

Owner:
Security Domain

Domain:
Authorization

Classification:
Critical
95. Policy Classification

Recomendado:

LOW
MEDIUM
HIGH
CRITICAL

Esto determina el nivel de revisión requerido.

96. Critical Policies

Ejemplos:

Cross-Tenant Isolation
Platform Admin
Billing Refund
Tenant Deletion
Security Configuration
AI High-Risk Operations

deben tener controles adicionales.

97. Policy Change Management

Flujo:

Developer
   ↓
Create Change
   ↓
Automated Tests
   ↓
Security Review
   ↓
Approval
   ↓
Deploy
   ↓
Monitor
98. Policy-as-Code

Las policies críticas deberían poder gestionarse como código:

policies/
 ├── security/
 ├── authorization/
 ├── training/
 ├── nutrition/
 ├── billing/
 └── ai/

Ventajas:

Git
Code Review
Versioning
CI/CD
Testing
Rollback
99. Policy Repository

Conceptualmente:

evoxa-policy/
├── policies/
├── schemas/
├── tests/
├── fixtures/
├── documentation/
└── versions/
100. Policy CI/CD

Pipeline:

Commit
 ↓
Lint
 ↓
Schema Validation
 ↓
Unit Tests
 ↓
Policy Tests
 ↓
Security Tests
 ↓
Simulation
 ↓
Approval
 ↓
Publish
101. Policy Contract

Una policy debe cumplir un contrato:

Policy Contract
 ├── ID
 ├── Version
 ├── Schema
 ├── Inputs
 ├── Outputs
 ├── Conditions
 ├── Effects
 └── Compatibility
102. Policy Schema

El esquema debe validar:

Required fields
Data types
Operators
Effects
Scopes
References

Una policy inválida no debe poder publicarse.

103. Policy Compatibility

Una nueva versión debe evaluarse por:

Backward Compatibility
Behavior Changes
Security Impact
Affected Domains
104. Policy Breaking Change

Ejemplo:

v1:
Trainer can export reports.

v2:
Trainer cannot export reports.

Debe considerarse un cambio de comportamiento relevante.

Debe existir:

Migration
Communication
Testing
Audit
105. Policy Simulation Against Production

Antes de activar una policy:

Historical Requests
       ↓
New Policy
       ↓
Simulation
       ↓
Decision Diff

Ejemplo:

Current: ALLOW
New: DENY

Affected Requests: 12,481

Esto permite evaluar impacto.

106. Policy Decision Explainability

Para operadores internos:

Decision:
DENY

Policy:
training.workout.update.ownership

Reason:
resource.trainerId != subject.id

La explicación debe estar disponible para debugging y auditoría sin exponer detalles sensibles al usuario final.

107. Policy Debugging

Debe existir capacidad para investigar:

Request ID
Subject
Action
Resource
Policy Version
Decision
Conditions
108. Policy Security Boundary

El Policy Engine debe considerarse componente crítico.

Debe protegerse contra:

Policy Injection
Expression Injection
Unauthorized Policy Changes
Policy Tampering
Privilege Escalation
Cache Poisoning
109. Policy Input Validation

Todo input debe validarse:

Schema
Type
Allowed Operators
Allowed Attributes
Allowed Resources

Nunca ejecutar expresiones arbitrarias provenientes del usuario.

110. Policy Execution Isolation

Si el motor utiliza expresiones dinámicas:

Policy
 ↓
Sandboxed Evaluation

Nunca ejecutar código arbitrario dentro del proceso principal.

111. Policy Secrets

Las policies no deben contener directamente:

Passwords
API Keys
Private Keys
Tokens
Database Credentials

Los secretos deben permanecer en Secret Management.

112. Policy References

Una policy puede referenciar:

attribute
configuration
entitlement
role
permission
resource

pero esas referencias deben estar tipadas y validadas.

113. Policy Registry

EVOXA debe mantener un catálogo:

Policy Registry

con:

ID
Domain
Owner
Version
Status
Classification
Description
114. Policy Discovery

Los administradores autorizados deben poder consultar:

Policies
Versions
Status
Owners
Scope

sin necesariamente poder modificarlas.

115. Policy Administration Permissions

Ejemplos:

policies.read
policies.create
policies.update
policies.publish
policies.rollback
policies.retire

Estas permissions deben estar separadas.

116. Policy Approval Permissions

Ejemplo:

policies.review
policies.approve

El creador de una policy crítica no debería aprobar su propia modificación cuando se requiera Separation of Duties.

117. Policy Retirement

Una policy retirada debe conservar historial:

Policy
 └── Versions
      ├── v1
      ├── v2
      └── v3 RETIRED

No eliminar físicamente el historial de políticas críticas.

118. Policy Retention

La retención debe alinearse con:

Security
Compliance
Audit
Privacy
Tenant Requirements
119. Policy Architecture Integration

E06 conecta múltiples componentes:

E04 Authentication
       │
       ▼
E05 Authorization
       │
       ▼
E06 Policy
       │
 ┌─────┼─────┐
 ▼     ▼     ▼
AI   Billing Security
 │     │     │
 └─────┼─────┘
       ▼
   Final Decision
120. Policy and Authentication

Authentication proporciona atributos como:

authenticationLevel
mfaVerified
sessionAge
identity

Policy puede utilizarlos.

Ejemplo:

IF
action == billing.refund
AND
mfaVerified == true
121. Policy and Authorization

E05 proporciona:

Roles
Permissions
Scopes
Resource Access

E06 agrega:

Conditions
Context
Rules
Priorities
Obligations
122. Policy and Security

Security puede proporcionar:

Risk
Threat Level
Device Trust
Session Risk

Policy puede consumir esos atributos.

123. Policy and Billing

Billing puede proporcionar:

Plan
Subscription
Entitlements
Account Status

Policy puede determinar:

Feature Access
Usage Restrictions
Premium Capabilities
124. Policy and AI

AI puede consultar policies para:

Tool Access
Data Access
Agent Actions
High-Risk Operations
Human Approval
125. Policy and Multi-Tenancy

Toda policy tenant-aware debe respetar:

tenantId

y nunca permitir:

cross-tenant access

por una configuración accidental.

126. Policy and Data Architecture

Policies pueden requerir datos de:

Users
Roles
Tenants
Resources
Relationships
Entitlements
Security Context

El acceso a estos datos debe seguir E02 Database Architecture.

127. Policy and API Architecture

Cada endpoint protegido debe mapear:

Endpoint
 ↓
Action
 ↓
Permission
 ↓
Policy
 ↓
Decision

Ejemplo:

PATCH /workouts/{id}
        ↓
training.workouts.update
        ↓
Permission
        ↓
Ownership Policy
        ↓
Decision
128. Policy and Event Architecture

Los cambios de policy deben emitir eventos:

PolicyCreated
PolicyUpdated
PolicyPublished
PolicyActivated
PolicyDeprecated
PolicyRetired
PolicyRolledBack
129. Policy and Observability

Cada evaluación crítica debe poder correlacionarse mediante:

requestId
traceId
policyId
policyVersion
decision
130. Policy and Operations

Operations debe poder monitorear:

Policy Engine Health
Evaluation Latency
Decision Errors
Policy Distribution
Cache State
Version Drift
131. Policy Runtime

En runtime:

Application
    ↓
PEP
    ↓
PDP
    ↓
Policy Cache
    ↓
Policy Evaluation
    ↓
Decision
132. Policy Control Plane

El control plane administra:

Policies
Versions
Publishing
Approvals
Governance
133. Policy Data Plane

El data plane ejecuta:

Policy Evaluation
Decision
Enforcement

Separar ambos permite:

Control Plane
      │
      ▼
Policy Distribution
      │
      ▼
Data Plane
134. Policy Distribution

Cuando una policy cambia:

Policy Store
   ↓
Policy Published
   ↓
Event
   ↓
Policy Distribution
   ↓
PDP Nodes
   ↓
Cache Refresh
135. Policy Change Event

Ejemplo conceptual:

{
  "event": "PolicyPublished",
  "policyId": "training.workouts.update.ownership",
  "version": 4,
  "timestamp": "..."
}
136. Policy Runtime Contract

El PDP debe ofrecer una interfaz estable:

evaluate(
    subject,
    action,
    resource,
    context
)

Resultado:

Decision
137. Policy Engine Abstraction

El resto de EVOXA no debe depender directamente del proveedor concreto del motor.

Arquitectura:

EVOXA Authorization
       ↓
Policy Engine Interface
       ↓
Implementation

Esto permite cambiar la tecnología posteriormente.

138. Policy Engine Adapter

Ejemplo:

PolicyEngine
     │
     ├── LocalEngine
     ├── RemoteEngine
     └── ExternalPolicyEngine
139. Policy Testing Layers

Debe existir testing en:

Unit
Policy
Integration
Security
Regression
Simulation
Load
140. Policy Regression Testing

Cada nueva versión debe ejecutar las policies existentes:

Existing Test Suite
       ↓
New Policy Version
       ↓
Regression
       ↓
PASS / FAIL
141. Policy Load Testing

Debe medirse:

Latency
Throughput
Concurrent Evaluations
Cache Hit Rate
PDP Availability
142. Policy Security Testing

Casos mínimos:

Unauthorized Policy Modification
Policy Injection
Tenant Policy Escape
Policy Bypass
Default Allow
Explicit Deny Failure
Stale Policy Cache
Privilege Escalation
143. Policy Threat Model

Amenazas:

Policy Tampering
Policy Injection
Policy Misconfiguration
Policy Drift
Policy Bypass
Unauthorized Publication
Unauthorized Approval
Stale Policy
Compromised PDP
Compromised PIP
144. Policy Defense in Depth

Protecciones:

Authentication
Authorization
Policy Validation
Code Review
Policy Testing
Approval
Audit
Runtime Monitoring
Fail Closed
145. Policy Misconfiguration

Un error de policy puede ser tan crítico como un error de código.

Por ello:

Policy Change
=
Production Code Change

en cuanto a governance y revisión de seguridad.

146. Policy Documentation

Cada policy crítica debe documentar:

Purpose
Owner
Scope
Inputs
Outputs
Conditions
Effects
Dependencies
Risks
Test Cases
Rollback
147. Policy Ownership Model
Platform Policy
    → Platform Owner

Security Policy
    → Security Owner

Billing Policy
    → Billing Owner

Training Policy
    → Training Owner

AI Policy
    → AI Governance Owner
148. Policy Lifecycle Governance
Draft
 ↓
Owner Review
 ↓
Security Review
 ↓
Testing
 ↓
Approval
 ↓
Publish
 ↓
Monitor
 ↓
Review
 ↓
Retire
149. Policy Review Cycle

Las policies críticas deberían revisarse periódicamente.

Revisión puede evaluar:

Usage
Effectiveness
Security
Incidents
Business Changes
Regulatory Changes
150. Policy Architecture Principles
01 — Policies Must Be Explicit
02 — Default Deny
03 — Explicit Deny Has Precedence
04 — Policies Must Be Versioned
05 — Policies Must Be Testable
06 — Policies Must Be Auditable
07 — Policies Must Be Governed
08 — Policies Must Be Independently Deployable
09 — Policies Must Use Trusted Context
10 — Policies Must Respect Tenant Isolation
11 — Policies Must Support Resource-Level Decisions
12 — Policies Must Support Contextual Authorization
13 — Policy Evaluation Must Be Deterministic
14 — Critical Policies Must Fail Closed
15 — Policy Changes Must Be Reviewable
16 — Policy History Must Be Preserved
17 — Policy Engine Must Be Replaceable
18 — Agents Must Respect Policy Boundaries
19 — Policy Administration Must Be Protected
20 — Security Policies Must Be Treated as Critical Infrastructure
151. Definition of Done

E06 queda definido cuando EVOXA dispone de:

✓ Policy Model
✓ Policy Definition
✓ Policy Store
✓ Policy Registry
✓ Policy Language
✓ Policy Conditions
✓ Policy Context
✓ Policy Evaluation
✓ PDP
✓ PEP
✓ PAP
✓ PIP
✓ Policy Composition
✓ Explicit Deny
✓ Default Deny
✓ Policy Priority
✓ Policy Scope
✓ Policy Inheritance
✓ Policy Versioning
✓ Policy Lifecycle
✓ Policy Approval
✓ Policy Testing
✓ Policy Simulation
✓ Dry Run
✓ Policy Rollout
✓ Policy Rollback
✓ Policy-as-Code
✓ Policy CI/CD
✓ Policy Audit
✓ Policy Metrics
✓ Policy Observability
✓ Policy Caching
✓ Policy Distribution
✓ Policy Governance
✓ AI Policy Control
✓ Tenant Policy Control
✓ Billing Policy Integration
✓ Security Policy Integration
✓ Policy Threat Model
✓ Policy Failure Handling
✓ Policy Security Testing
152. E04 → E05 → E06

La arquitectura de seguridad de Engineering queda ahora mucho más clara:

E04 — Authentication
        │
        │ Who are you?
        ▼
     Identity
        │
        ▼
E05 — Authorization
        │
        │ What can you do?
        ▼
 Permissions / Roles / Scopes
        │
        ▼
E06 — Policy
        │
        │ Under what conditions?
        ▼
 Policies / Context / Rules
        │
        ▼
   PDP Decision
        │
        ▼
   PEP Enforcement
        │
        ▼
      Resource

Y conceptualmente:

                EVOXA ACCESS CONTROL
                       │
        ┌──────────────┼──────────────┐
        │              │              │
 Authentication   Authorization     Policy
     E04              E05             E06
        │              │              │
      WHO?            WHAT?          WHEN/
                                      HOW?
        │              │              │
        └──────────────┼──────────────┘
                       ▼
               ACCESS DECISION
                       │
                       ▼
                  ENFORCEMENT

E06 establece así el Policy Control Plane de EVOXA y proporciona la base para que las reglas de seguridad, negocio, tenant, AI, billing y acceso puedan evolucionar de manera gobernada, versionada, testeable y auditable sin acoplar toda la lógica directamente a los servicios de aplicación.
