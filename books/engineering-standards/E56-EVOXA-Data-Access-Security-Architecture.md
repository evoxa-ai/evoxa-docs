E56 — EVOXA Data Access Security Architecture
Engineering Specification

Pertenece a: Engineering Specification
Capítulo: E56 — Data Access Security Architecture
Anterior: E55 — Data Access Governance Architecture
Siguiente: E57 — Data Protection Architecture

1. Propósito

E56 define los mecanismos técnicos mediante los cuales EVOXA protege los accesos a datos contra acceso no autorizado, interceptación, manipulación, abuso, escalación de privilegios, exposición accidental y compromiso de credenciales o workloads.

La diferencia fundamental con E55 es:

E55 — Data Access Governance
    ↓
¿Está permitido este acceso?

E56 — Data Access Security
    ↓
¿Cómo protegemos técnicamente ese acceso?

Por tanto:

Identity
   │
   ▼
Authentication
   │
   ▼
Authorization
   │
   ▼
Governance
   │
   ▼
Security Enforcement
   │
   ▼
Data Access
2. Objetivo Arquitectónico

EVOXA no debe considerar que una autorización válida implica automáticamente un acceso seguro.

Authorized
    ≠
Secure

El acceso debe atravesar controles de seguridad:

Access Request
      │
      ▼
Identity Security
      │
      ▼
Authorization Security
      │
      ▼
Transport Security
      │
      ▼
Access Enforcement
      │
      ▼
Data Security
      │
      ▼
Audit
3. Principios Fundamentales
3.1 Zero Trust

Cada acceso debe verificarse explícitamente.

Never Trust
Always Verify
3.2 Least Privilege

El principal recibe únicamente los permisos necesarios.

3.3 Defense in Depth

No debe existir un único control cuya caída exponga todos los datos.

Identity
  +
Authorization
  +
Network
  +
Encryption
  +
Policy
  +
Audit
3.4 Secure by Default

Los nuevos recursos deben comenzar en un estado seguro:

new resource
    ↓
no public access
    ↓
no implicit trust
    ↓
explicit configuration
3.5 Fail Secure

Ante una condición de seguridad no resuelta:

Unknown
   ↓
Deny
3.6 Strong Workload Identity

Los servicios deben identificarse mediante identidades propias.

No:

shared credentials

Preferido:

Service
   ↓
Workload Identity
   ↓
Access
3.7 Credential Minimization

Las credenciales no deben distribuirse innecesariamente.

3.8 Encryption Everywhere Appropriate

Los datos deben protegerse:

in transit
at rest
in backups
in replicas
in exports

cuando su clasificación lo requiera.

4. Scope

E56 cubre:

Access Authentication Security
Credential Security
Workload Identity
Authorization Enforcement
Network Security
Transport Security
Encryption
Key Management
Secrets
Database Security
API Data Security
Cache Security
Search Security
Replica Security
Tenant Isolation
Field Protection
Data Masking
Token Security
Session Security
Access Monitoring
Security Telemetry
Threat Detection
Security Incident Response
5. Non-Goals

E56 no sustituye:

E04 Authentication Architecture
E05 Authorization Architecture
E06 Policy Architecture
E20 Runtime Policy Architecture
E43 Data Integrity Architecture
E55 Data Access Governance Architecture

E56 proporciona los controles de seguridad técnicos que protegen esos mecanismos.

6. Security Architecture
                         ┌───────────────────┐
                         │ Identity Provider │
                         └─────────┬─────────┘
                                   │
                                   ▼
                         ┌───────────────────┐
                         │ Authentication    │
                         └─────────┬─────────┘
                                   │
                                   ▼
Request ───────────────────────────┤
                                   ▼
                         ┌───────────────────┐
                         │ Authorization     │
                         └─────────┬─────────┘
                                   │
                                   ▼
                         ┌───────────────────┐
                         │ Security Policy   │
                         └─────────┬─────────┘
                                   │
                                   ▼
                         ┌───────────────────┐
                         │ Access Enforcement│
                         └─────────┬─────────┘
                                   │
                    ┌──────────────┼──────────────┐
                    ▼              ▼              ▼
                 Database        Cache          Search
                    │              │              │
                    └──────────────┼──────────────┘
                                   ▼
                              Protected Data
7. Security Boundary

Todo acceso a datos debe cruzar una frontera de seguridad explícita.

Untrusted Context
       │
       ▼
Security Boundary
       │
       ▼
Trusted Data Access

No se debe asumir confianza simplemente porque:

same network
same cluster
same process
same tenant
same application
8. Trust Zones

EVOXA debe distinguir conceptualmente:

Internet / External
        │
        ▼
Edge
        │
        ▼
Application
        │
        ▼
Data Access
        │
        ▼
Data Stores

Cada transición debe aplicar controles apropiados.

9. Data Access Security Flow
Request
  │
  ▼
Authenticate
  │
  ▼
Validate Credential
  │
  ▼
Resolve Identity
  │
  ▼
Authorize
  │
  ▼
Evaluate Governance
  │
  ▼
Validate Context
  │
  ▼
Secure Channel
  │
  ▼
Access Data
  │
  ▼
Audit
10. Authentication Security

La autenticación debe resistir:

credential theft
token theft
replay
session hijacking
brute force
credential stuffing
11. Authentication Strength

Los recursos más sensibles pueden requerir autenticación más fuerte:

LOW
  → normal authentication

HIGH
  → stronger authentication

CRITICAL
  → phishing-resistant / step-up authentication
12. Step-Up Authentication

Para operaciones sensibles:

Authenticated
     │
     ▼
Sensitive Access
     │
     ▼
Step-Up
     │
     ▼
Access

Ejemplos:

bulk export
restricted data
administrative access
break-glass
key management
13. Token Security

Los tokens deben:

expire
be scoped
be audience-bound
be issuer-validated
be integrity-protected

No deben representar permisos más amplios que los necesarios.

14. Token Scope

Ejemplo:

token
 ├── tenant = T1
 ├── resource = Customer
 ├── operation = READ
 └── expiration = short

No:

global admin token

para una operación de lectura normal.

15. Token Audience

Un token emitido para:

service-a

no debe ser aceptado automáticamente por:

service-b
16. Token Replay Protection

Cuando el riesgo lo requiera:

token
 +
short lifetime
 +
binding
 +
nonce / replay controls

debe limitar reutilización maliciosa.

17. Session Security

Las sesiones deben controlar:

expiration
idle timeout
absolute timeout
revocation
rotation
concurrent sessions
18. Workload Identity

Cada workload debe tener una identidad diferenciada:

CustomerService
OrderService
BillingService
AnalyticsJob
SearchIndexer

Cada identidad debe poder recibir permisos diferentes.

19. No Shared Service Accounts

Evitar:

Service A ─┐
Service B ─┼──> shared-account
Service C ─┘

Preferido:

Service A → identity-A
Service B → identity-B
Service C → identity-C
20. Credential Isolation

Las credenciales deben estar aisladas por:

tenant
environment
service
resource

cuando sea necesario.

21. Secret Management

Los secretos no deben almacenarse en:

source code
Git
container images
logs
configuration files

Deben utilizar un sistema dedicado de secrets management.

22. Secret Lifecycle
Generate
   ↓
Store
   ↓
Distribute securely
   ↓
Use
   ↓
Rotate
   ↓
Revoke
   ↓
Destroy
23. Secret Rotation

Los secretos deben poder rotarse sin despliegues destructivos.

Secret v1
   │
   ▼
Secret v2
   │
   ▼
Grace Period
   │
   ▼
v1 revoked
24. Credential Compromise

Ante compromiso:

Detect
  ↓
Revoke
  ↓
Rotate
  ↓
Invalidate sessions
  ↓
Investigate
25. Authorization Enforcement

La autorización debe verificarse cerca del recurso protegido.

API
 │
 ▼
Authorization
 │
 ▼
Data Access Boundary
 │
 ▼
Database

No debe depender únicamente de controles del frontend.

26. Defense Against Authorization Bypass

Debe evitarse:

API protected
     │
     X
     ▼
direct database path

o:

normal endpoint protected
     │
     X
     ▼
export endpoint unprotected
27. Object-Level Authorization

Cada recurso debe verificarse:

Can principal access this object?

No basta con:

Can principal access this endpoint?
28. Field-Level Security

Los campos sensibles pueden requerir controles adicionales:

Customer
├── id
├── name
├── email
├── phone
└── financialData

El servicio puede devolver:

id
name
email

sin exponer:

financialData
29. Data Masking

Para determinados consumidores:

4111111111111111

puede convertirse en:

************1111
30. Dynamic Masking

El masking puede depender del contexto:

Support Agent
    → masked

Finance Service
    → authorized full value

Unauthorized
    → denied
31. Encryption in Transit

Toda comunicación sensible debe utilizar canales autenticados y cifrados.

Service A
   │
   │ encrypted + authenticated
   ▼
Service B
32. TLS

Como principio:

TLS
+
certificate validation
+
secure protocol configuration
33. Mutual TLS

Para comunicaciones service-to-service de alta sensibilidad:

Service A
  ⇄ mTLS ⇄
Service B

Ambos extremos se autentican.

34. Internal Network Security

La red interna no debe considerarse inherentemente confiable.

Internal
   ≠
Trusted
35. Network Segmentation

Los componentes sensibles pueden aislarse:

Public Zone
    │
    ▼
Application Zone
    │
    ▼
Data Access Zone
    │
    ▼
Data Zone
36. Database Network Access

Las bases de datos no deben estar expuestas directamente a redes públicas.

Internet
   X
   │
Database

Preferido:

Application
    │
    ▼
Private Network
    │
    ▼
Database
37. Database Authentication

Cada aplicación debe utilizar identidad/credencial propia cuando el motor lo permita.

Evitar:

all applications
      │
      ▼
root/admin
38. Database Least Privilege

Un servicio de lectura debe tener:

SELECT

cuando sea suficiente.

No:

SELECT + INSERT + UPDATE + DELETE + ADMIN
39. Database Privilege Separation

Separar:

application identity
migration identity
administrative identity
backup identity
analytics identity
40. Database Encryption at Rest

Los stores sensibles deben utilizar cifrado en reposo.

41. Storage Encryption

Debe proteger:

primary database
replicas
snapshots
backups
object storage
search indexes

según clasificación.

42. Encryption Key Separation

Las claves deben mantenerse separadas de los datos protegidos.

Data
 │
 X
 │
Key Management System
43. Key Management

Las claves deben tener:

creation
rotation
versioning
access control
audit
revocation
destruction
44. Key Hierarchy

Modelo conceptual:

Root / Master Key
       │
       ▼
Data Encryption Key
       │
       ▼
Encrypted Data
45. Envelope Encryption

Preferido para grandes volúmenes:

Data
 │
 ▼
DEK
 │
 ▼
Encrypted Data
 │
 ▼
KEK
46. Key Access Governance

No todo servicio que puede leer datos debe poder administrar las claves.

Separar:

Data Reader
     ≠
Key Administrator
47. Key Rotation

La rotación debe preservar capacidad de descifrado histórico durante el período necesario.

Key v1
  │
  ▼
Key v2
  │
  ▼
New Data → v2
Old Data → v1 until re-encryption / retirement
48. Data Re-Encryption

Para datos altamente sensibles:

Encrypted v1
     │
     ▼
Re-encrypt
     │
     ▼
Encrypted v2
49. Backup Security

Los backups heredan las necesidades de seguridad del dataset.

Production Data
      │
      ▼
Backup
      │
      ▼
Same or stronger protection
50. Replica Security

Una réplica no debe tener controles menores que el origen.

Primary
  │
  ▼
Replica
  │
  ▼
Same security boundary
51. Read Replica Access

Si una réplica se utiliza para analytics:

Analytics Identity
      │
      ▼
Read Replica

debe existir un control específico.

52. Cache Security

Las caches pueden contener datos sensibles y deben proteger:

authentication
authorization
encryption
tenant isolation
expiration
invalidation
53. Cache Key Isolation

Nunca:

customer:{id}

si el mismo ID puede existir en múltiples tenants.

Preferido:

tenant:{tenantId}:customer:{id}
54. Cache Authorization

Una cache no debe permitir recuperar un dato que el usuario no podría obtener de la fuente.

55. Cache Poisoning Protection

Debe validarse:

cache source
key ownership
serialization
authorization context
TTL

para impedir inyección de datos maliciosos.

56. Search Security

Los índices de búsqueda deben respetar autorización.

Query
 │
 ▼
Authorization Filter
 │
 ▼
Search
 │
 ▼
Authorized Results
57. Search Index Exposure

No debe indexarse automáticamente:

restricted fields
credentials
secrets
tokens
unnecessary PII
58. Search Result Filtering

Aunque el índice contenga un documento:

Document exists
      ≠
User can retrieve document
59. Data Federation Security

Para fuentes externas:

EVOXA
 │
 ▼
Federation Gateway
 │
 ▼
External Data

El gateway debe controlar:

identity
credentials
authorization
transport
audit
60. Data Virtualization Security

La capa virtual no debe convertirse en bypass:

Virtual Layer
     │
     ▼
Underlying Sources

Cada acceso debe conservar el contexto de seguridad.

61. Cross-Tenant Isolation

Cada request debe llevar contexto de tenant cuando corresponda.

Request
  │
  ▼
Tenant Context
  │
  ▼
Authorization
  │
  ▼
Data Query
62. Tenant Context Integrity

El tenant no debe depender exclusivamente de un parámetro manipulable por el cliente.

No confiar únicamente en:

?tenantId=otherTenant

Debe derivarse/verificarse mediante identidad y contexto autorizado.

63. Tenant Context Propagation

Debe propagarse de forma segura a:

API
Service
Repository
Cache
Search
Events
Jobs
Workflows
64. Cross-Tenant Query Protection

Una query debe aplicar filtros de tenant de forma sistemática cuando corresponda.

WHERE tenant_id = currentTenant

Pero el filtro no debe depender únicamente de que un desarrollador recuerde agregarlo manualmente.

65. Defense in Depth for Tenants

Idealmente:

Application
   +
Authorization
   +
Repository
   +
Database Row-Level Controls

cuando sea apropiado.

66. Row-Level Security

Para recursos críticos, el motor de datos puede reforzar:

tenant A → rows A
tenant B → rows B

Esto añade una barrera contra errores de aplicación.

67. Tenant Escape Detection

Debe detectarse:

requestedTenant != authorizedTenant

y producir:

DENY
+
security event

cuando corresponda.

68. API Security

Los endpoints de datos deben proteger:

authentication
authorization
input validation
rate limiting
payload size
pagination
export controls
69. API Rate Limiting

Debe limitarse abuso:

Normal Read
    ↓
rate policy

Bulk Read
    ↓
stricter policy
70. Data Enumeration Protection

Debe evitarse que APIs permitan descubrir datos mediante:

sequential IDs
unbounded search
high-volume probing
71. Pagination Security

Los mecanismos de paginación no deben permitir:

cross-tenant traversal
authorization bypass
unbounded extraction
72. Cursor Security

Los cursors deben estar:

opaque
tamper-resistant
scope-bound
time-limited

cuando sea necesario.

73. Bulk Access Protection

Operaciones masivas requieren:

authorization
rate limits
purpose
audit
possibly approval
74. Export Protection

Exportaciones sensibles pueden requerir:

step-up authentication
approval
watermarking
encryption
destination restrictions
expiration
75. Data Exfiltration Controls

EVOXA debe detectar o limitar:

mass reads
mass exports
unexpected destinations
high-frequency queries
unusual data volume
76. Destination Security

Cuando los datos abandonan EVOXA:

Data
  │
  ▼
Destination Validation
  │
  ▼
Approved Destination
77. Secure Serialization

La serialización debe evitar:

secret leakage
overexposure
unsafe deserialization

E23 define la serialización; E56 define sus controles de seguridad.

78. Unsafe Deserialization

Debe evitarse aceptar objetos arbitrarios que puedan ejecutar comportamiento no deseado.

Preferido:

strict schema
typed payload
allowlisted types
79. Input Security

Antes de acceder a datos:

validate
normalize
authorize
query safely
80. Injection Protection

Debe proteger contra:

SQL injection
NoSQL injection
search injection
command injection
template injection
81. Parameterized Queries

Las queries deben utilizar parámetros seguros.

No:

string concatenation

Preferido:

parameterized query
82. Repository Security

Los repositorios deben impedir que consumidores salten:

governance
authorization
tenant filtering

mediante APIs de bajo nivel.

83. Direct Query Restrictions

No exponer una API genérica tipo:

executeArbitraryQuery()

a consumidores normales.

84. Administrative Access

El acceso administrativo debe estar separado:

Application Access
     ≠
Administrative Access
85. Privileged Access Management

Los privilegios administrativos deberían ser:

temporary
justified
approved
audited
revocable
86. Production Data Access

El acceso humano directo a producción debe ser excepcional.

Engineer
   │
   ▼
Approved Access Request
   │
   ▼
JIT Privilege
   │
   ▼
Production
87. Break-Glass Security

El mecanismo de emergencia debe:

authenticate strongly
require reason
limit scope
limit duration
produce enhanced audit
trigger review
88. Session Recording

Para operaciones administrativas de alto riesgo puede requerirse:

session recording
command logging
enhanced audit

según política.

89. Security Monitoring

Debe observar:

successful access
failed access
privilege changes
credential changes
exports
bulk queries
cross-tenant attempts
administrative actions
90. Security Telemetry

Cada evento relevante debería contener:

timestamp
principal
resource
operation
tenant
source
decision
risk
correlationId

sin incluir secretos innecesarios.

91. No Secret Logging

Nunca registrar:

password
private key
access token
refresh token
encryption key
secret value
92. Sensitive Data Logging

Los datos sensibles deben evitarse en logs.

En lugar de:

customer.email = actual-email

preferir:

customerId = C123
field = email
action = READ
93. Security Correlation

Los eventos deben poder correlacionarse:

Request
   │
   ├── Authentication
   ├── Authorization
   ├── Governance
   ├── Data Access
   └── Audit

mediante un correlationId apropiado.

94. Threat Detection

Debe detectar patrones como:

credential abuse
privilege escalation
data enumeration
mass extraction
tenant traversal
unusual administrative access
95. Anomaly Detection

Ejemplo:

Normal:
100 reads/hour

Observed:
50,000 reads/hour

Debe generar:

risk signal

y potencialmente:

rate limit
temporary block
step-up
incident
96. Security Controls by Data Classification

Ejemplo:

Clasificación	Controles
PUBLIC	baseline
INTERNAL	authenticated access
CONFIDENTIAL	strong authorization + audit
RESTRICTED	encryption + enhanced audit
HIGHLY_RESTRICTED	strong auth + approval + enhanced controls

La matriz final debe ser configurable.

97. Security Policy Evaluation
Principal
  +
Resource
  +
Classification
  +
Operation
  +
Context
      │
      ▼
Security Policy
      │
      ▼
Security Decision
98. Security Decision

Puede producir:

ALLOW
DENY
ALLOW_WITH_CONTROLS
STEP_UP
REQUIRE_APPROVAL
RATE_LIMIT
QUARANTINE
99. Security Enforcement Points

Puede existir más de uno:

Edge PEP
API PEP
Service PEP
Repository PEP
Database PEP
Export PEP
100. Layered Enforcement
Client
  │
  ▼
API Security
  │
  ▼
Service Security
  │
  ▼
Repository Security
  │
  ▼
Database Security

La redundancia es deliberada para recursos críticos.

101. Security Boundary Around E54
                 E55 Governance
                       │
                       ▼
                ┌─────────────┐
                │ E56 Security│
                └──────┬──────┘
                       │
                       ▼
                ┌─────────────┐
                │ E54 Access  │
                │ Abstraction │
                └──────┬──────┘
                       │
                       ▼
                    Data
102. Security vs Governance
E55 Governance	E56 Security
Defines allowed access	Protects the access path
Ownership	Credential security
Approval	Authentication security
Purpose	Encryption
Policy lifecycle	Network controls
Access review	Threat detection
Accountability	Attack resistance
Compliance evidence	Security telemetry
103. Security vs Authorization
Authorization
→ "Can this principal perform this operation?"

Security
→ "How do we ensure the operation cannot be
   abused, intercepted, forged, escalated or bypassed?"
104. Security vs Data Protection

E56 protege el acceso.

La protección criptográfica y de datos puede extenderse a:

E57 — Data Protection Architecture

E56 debe integrar esos controles sin duplicarlos.

105. Access Security Lifecycle
Provision
   ↓
Authenticate
   ↓
Authorize
   ↓
Govern
   ↓
Secure Access
   ↓
Monitor
   ↓
Detect
   ↓
Respond
   ↓
Revoke
106. Provisioning Security

Al crear un nuevo acceso:

Identity
   │
   ▼
Minimum Scope
   │
   ▼
Secure Credential
   │
   ▼
Audit
107. Revocation Security

Cuando el acceso deja de ser válido:

Revoke Grant
   │
   ▼
Invalidate Token
   │
   ▼
Invalidate Session
   │
   ▼
Invalidate Cache
   │
   ▼
Audit
108. Security Cache Invalidation

La revocación no debe quedar bloqueada por caches autorizadas anteriormente.

Grant revoked
     ↓
security cache invalidation
109. Security Configuration

Los controles deben centralizarse donde sea posible:

Security Configuration
       │
       ├── TLS
       ├── token lifetime
       ├── session lifetime
       ├── rate limits
       ├── encryption
       └── security policies
110. Secure Defaults

Por defecto:

encryption = enabled
public access = disabled
admin access = disabled
cross-tenant = denied
export = restricted
logging = enabled
audit = enabled

salvo excepciones explícitas.

111. Security Configuration Drift

Debe detectarse:

Declared:
encryption required

Runtime:
encryption disabled

y producir una alerta o bloqueo según criticidad.

112. Security Health Checks

Deben comprobar:

TLS configuration
certificate validity
key availability
secret rotation
identity connectivity
policy availability
audit availability
encryption state
network exposure
113. Certificate Management

Los certificados deben:

have owner
expire predictably
rotate automatically where possible
be monitored
be revoked when compromised
114. Certificate Expiration

Debe existir alerta previa a:

expiration

y mecanismos automáticos de renovación cuando sea posible.

115. Service Identity Rotation

La identidad criptográfica de un workload debe poder rotarse sin downtime.

Identity v1
   │
   ▼
Identity v2
   │
   ▼
v1 revoked
116. Security Testing

Debe cubrir:

authentication bypass
authorization bypass
tenant escape
token forgery
token replay
credential leakage
injection
data exfiltration
encryption failures
cache leakage
search leakage
117. Negative Security Testing

Ejemplos:

invalid token → DENY
expired token → DENY
wrong audience → DENY
wrong tenant → DENY
insufficient scope → DENY
revoked grant → DENY
expired grant → DENY
untrusted network → DENY
118. Security Regression Testing

Cada cambio en:

API
repository
policy
database
cache
search
identity

debe evaluar impacto de seguridad.

119. Security Chaos Testing

Escenarios:

identity provider unavailable
key service unavailable
certificate expired
policy service unavailable
audit unavailable
network partition
credential rotation during request
120. Incident Response

Ante una sospecha de compromiso:

Detect
  ↓
Contain
  ↓
Revoke
  ↓
Rotate
  ↓
Investigate
  ↓
Recover
  ↓
Review
121. Automatic Security Containment

Puede incluir:

disable principal
revoke token
block source
reduce rate
disable export
isolate workload

según riesgo.

122. Compromised Workload

Si un servicio se compromete:

Compromised Service
       │
       ▼
Identity Revocation
       │
       ▼
Credential Rotation
       │
       ▼
Access Review
       │
       ▼
Incident Investigation
123. Security Evidence

Debe conservarse evidencia de:

authentication
authorization
policy decision
credential issuance
credential rotation
data access
security alerts
administrative actions
revocation
124. Security Audit Integrity

Los eventos de seguridad deben protegerse contra:

modification
deletion
unauthorized access
125. Security Metrics

Métricas mínimas:

auth_failures
auth_success
authorization_denials
token_failures
credential_rotations
secret_age
certificate_expiry
encryption_coverage
security_policy_errors
cross_tenant_attempts
bulk_access_events
security_incidents
126. Security SLOs

Deben definirse objetivos para:

authentication availability
authorization availability
policy evaluation latency
credential rotation
revocation propagation
security event delivery
incident detection
127. Revocation Propagation

Una revocación crítica debe propagarse rápidamente:

Revocation
   │
   ├── API
   ├── Services
   ├── Cache
   ├── Sessions
   └── Tokens
128. Security Availability

La seguridad no debe ser una dependencia frágil que provoque:

security service failure
       ↓
application-wide outage

Debe existir una estrategia de alta disponibilidad y fail-secure apropiada.

129. Security Decision Caching

Las decisiones pueden cachearse únicamente con:

bounded TTL
policy version
identity context
scope
revocation awareness
130. No Stale Authorization

Una decisión obsoleta no debe permitir acceso sensible indefinidamente.

131. Security for Jobs

Cada job debe usar:

job identity
minimal permissions
short-lived credentials
audit
132. Security for Workflows

Cada workflow step debe utilizar el menor privilegio necesario.

Workflow
  │
  ├── Step A → read
  ├── Step B → transform
  └── Step C → write

No:

Workflow → admin access

para todos los steps.

133. Security for Agents

Los agentes deben acceder a datos mediante herramientas explícitas:

Agent
  │
  ▼
Approved Tool
  │
  ▼
Security Enforcement
  │
  ▼
Data

Nunca mediante credenciales de infraestructura expuestas al modelo.

134. Agent Data Boundary

Debe existir una frontera clara:

Model
   X
Raw Infrastructure Credentials

Model
   ↓
Governed Data Tool
135. Security for Analytics

Analytics debe utilizar:

dedicated identities
controlled datasets
read-only access
limited export
audit

cuando sea apropiado.

136. Security for Reporting

Los reportes pueden contener datos derivados sensibles.

Por tanto:

Report
  ≠
Automatically safe

Debe heredar controles de clasificación y acceso.

137. Security for Events

Los eventos pueden transportar información sensible.

Debe controlarse:

producer
consumer
payload
tenant
classification
transport
retention
138. Event Replay Security

El replay de eventos no debe permitir acceso no autorizado a datos históricos.

Replay Request
      │
      ▼
Authorization
      │
      ▼
Historical Data
139. Security for Replication

Replication credentials deben ser independientes:

Application Identity
      ≠
Replication Identity
140. Security for Migration

Migration jobs pueden requerir privilegios elevados.

Deben ser:

temporary
isolated
audited
approved
141. Security for Federation

External credentials deben almacenarse y rotarse mediante secret management.

142. Security for Virtualization

El contexto de seguridad debe viajar hasta las fuentes subyacentes cuando sea necesario.

Caller
  ↓
Virtual Layer
  ↓
Security Context
  ↓
Source
143. Security for Access Abstraction

E54 no debe exponer una API que elimine los controles de E56.

Ejemplo inseguro:

repository.rawQuery(...)

accesible desde cualquier servicio.

144. Security Reference Architecture
                    ┌──────────────────────┐
                    │ Identity / PKI / KMS │
                    └───────────┬──────────┘
                                │
                                ▼
                         ┌─────────────┐
                         │ AuthN/AuthZ │
                         └──────┬──────┘
                                │
                                ▼
                         ┌─────────────┐
                         │ E55         │
                         │ Governance  │
                         └──────┬──────┘
                                │
                                ▼
                         ┌─────────────┐
                         │ E56         │
                         │ Security    │
                         └──────┬──────┘
                                │
                ┌───────────────┼───────────────┐
                ▼               ▼               ▼
             Network          API            Workload
                │               │               │
                └───────────────┼───────────────┘
                                ▼
                         ┌─────────────┐
                         │ E54 Access  │
                         │ Abstraction │
                         └──────┬──────┘
                                │
                ┌───────────────┼───────────────┐
                ▼               ▼               ▼
             Database         Cache           Search
                │               │               │
                └───────────────┼───────────────┘
                                ▼
                           Protected Data
145. Core Security Invariants
Invariant 1 — No Implicit Trust

Ningún componente se considera confiable únicamente por su ubicación en la red.

Invariant 2 — Strong Identity

Todo acceso significativo debe poder atribuirse a una identidad verificable.

Invariant 3 — Least Privilege

Cada identidad recibe únicamente los privilegios necesarios.

Invariant 4 — Secure Transport

Los datos sensibles no deben viajar por canales no protegidos.

Invariant 5 — Encryption at Rest

Los datos que requieren protección deben estar cifrados en almacenamiento.

Invariant 6 — Key Separation

Las claves criptográficas no deben estar almacenadas junto a los datos que protegen de forma insegura.

Invariant 7 — Tenant Isolation

Un principal no puede acceder a datos de otro tenant sin autorización explícita.

Invariant 8 — No Bypass

Cache, search, replica, federation, virtualization y read models no pueden convertirse en bypass de seguridad.

Invariant 9 — Revocation

Un acceso revocado debe dejar de ser efectivo dentro del límite de propagación definido.

Invariant 10 — No Secret Leakage

Secrets, tokens y claves privadas nunca deben aparecer en logs o respuestas no autorizadas.

Invariant 11 — Defense in Depth

Los recursos críticos deben estar protegidos por múltiples controles independientes.

Invariant 12 — Auditability

Los eventos de seguridad relevantes deben poder reconstruirse posteriormente.

Invariant 13 — Secure Failure

Un fallo de un componente de seguridad no debe producir privilegios adicionales.

Invariant 14 — Credential Rotation

Las credenciales sensibles deben poder rotarse y revocarse.

Invariant 15 — Data Minimization

Los componentes reciben únicamente los datos necesarios para realizar su función.

146. Completion Criteria

E56 se considera completo cuando EVOXA dispone de:

✓ Zero-trust access model
✓ Strong workload identities
✓ Secure authentication integration
✓ Token validation
✓ Token scoping
✓ Token audience validation
✓ Session security
✓ Credential isolation
✓ Secret management
✓ Secret rotation
✓ Credential revocation
✓ Step-up authentication
✓ Authorization enforcement
✓ Object-level authorization
✓ Field-level security
✓ Data masking
✓ TLS
✓ mTLS where required
✓ Network segmentation
✓ Private database access
✓ Database least privilege
✓ Database identity separation
✓ Encryption at rest
✓ Key management
✓ Key rotation
✓ Envelope encryption
✓ Backup encryption
✓ Replica protection
✓ Cache security
✓ Search security
✓ Federation security
✓ Virtualization security
✓ Tenant isolation
✓ Tenant context integrity
✓ API security
✓ Rate limiting
✓ Enumeration protection
✓ Bulk access protection
✓ Export security
✓ Data exfiltration controls
✓ Injection protection
✓ Secure serialization
✓ Privileged access controls
✓ JIT access
✓ Break-glass security
✓ Security monitoring
✓ Security telemetry
✓ Threat detection
✓ Anomaly detection
✓ Security incident response
✓ Automatic containment
✓ Revocation propagation
✓ Security configuration management
✓ Certificate management
✓ Security health checks
✓ Negative security testing
✓ Regression testing
✓ Chaos testing
✓ Security evidence
✓ Security metrics
✓ Security SLOs
147. Principio Rector de E56

Data Access Security garantiza que todo acceso autorizado a datos en EVOXA sea técnicamente protegido contra suplantación, interceptación, manipulación, escalación, abuso y bypass, aplicando identidad fuerte, mínimo privilegio, aislamiento, cifrado, controles de red, protección de credenciales, enforcement distribuido y detección continua.

La relación entre los capítulos queda:

E53 — DATA VIRTUALIZATION
        │
        ▼
E54 — DATA ACCESS ABSTRACTION
        │
        ▼
E55 — DATA ACCESS GOVERNANCE
        │
        ├── ¿Quién?
        ├── ¿Qué?
        ├── ¿Para qué?
        ├── ¿Bajo qué política?
        └── ¿Durante cuánto tiempo?
        │
        ▼
E56 — DATA ACCESS SECURITY
        │
        ├── ¿Cómo autenticamos?
        ├── ¿Cómo protegemos las credenciales?
        ├── ¿Cómo aislamos tenants?
        ├── ¿Cómo protegemos el canal?
        ├── ¿Cómo ciframos?
        ├── ¿Cómo evitamos bypass?
        ├── ¿Cómo detectamos abuso?
        └── ¿Cómo respondemos a compromiso?
        │
        ▼
Protected Data Access

E55 determina la legitimidad del acceso. E56 asegura técnicamente ese acceso.
