ADR-IDENTITY-038 — Identity Service-to-Service Authentication & Workload Identity Strategy

Status: Proposed
Version: 1.0
Date: 2026-09-03
Domain: Identity
Priority: Critical
Type: Security / Architecture / Identity

1. Context

EVOXA evolucionará hacia una plataforma modular, API-first, event-driven y preparada para operaciones distribuidas.

Además de usuarios humanos, existirán múltiples componentes que necesitarán comunicarse entre sí:

API
 ↓
Identity
 ↓
Training
 ↓
Nutrition
 ↓
Billing
 ↓
Analytics
 ↓
AI Services
 ↓
Event Platform

Estas comunicaciones no pueden depender de:

credenciales de usuarios humanos;
API Keys globales;
passwords compartidas;
secretos embebidos en código;
credenciales permanentes con privilegios excesivos.

ADR-IDENTITY-037 estableció que las identidades de máquina son entidades independientes:

User
    → Human Identity

ServiceAccount
    → Machine Identity

Este ADR profundiza específicamente en cómo un servicio demuestra su identidad ante otro servicio.

2. Problem Statement

Un escenario típico será:

Training Service
       ↓
Identity Service

o:

Nutrition Service
       ↓
Training Service

El servicio receptor debe poder determinar:

quién está llamando;
si la identidad es auténtica;
qué servicio representa;
qué tenant representa;
qué permisos tiene;
qué credencial utilizó;
si la credencial está vigente;
si existe delegation;
si la llamada cumple las políticas de seguridad.

Por lo tanto:

Service authentication must establish a verifiable workload identity before authorization is evaluated.

3. Decision

EVOXA adoptará una arquitectura de Workload Identity + Short-Lived Service Credentials como dirección preferente para comunicaciones internas.

El modelo será:

Workload
   ↓
Workload Identity
   ↓
Authentication Credential
   ↓
Target Service
   ↓
Identity Validation
   ↓
Authorization
   ↓
Policy Evaluation
   ↓
ALLOW / DENY

Las credenciales estáticas podrán existir para compatibilidad o integraciones donde sean necesarias, pero no serán la estrategia preferida para comunicaciones internas de alta seguridad.

4. Core Principle

La arquitectura seguirá:

Authenticate the workload, then authorize the operation.

No:

Credential exists
      ↓
Everything allowed

sino:

Credential
   ↓
Service Identity
   ↓
Tenant Context
   ↓
Scope
   ↓
Permission
   ↓
Policy
   ↓
Operation
5. Workload Identity

Un workload representa una instancia lógica de software que ejecuta una función de EVOXA.

Ejemplos:

identity-api
training-api
nutrition-api
billing-api
notification-worker
event-consumer
ai-orchestrator

Cada workload deberá poder asociarse con una identidad de máquina.

Conceptualmente:

Workload
   │
   └── ServiceAccount
6. Service Identity vs Workload Instance

Debe existir una distinción entre:

Service Identity

y:

Running Instance

Por ejemplo:

ServiceAccount:
    training-service

Instances:
    training-service-pod-001
    training-service-pod-002
    training-service-pod-003

Las instancias pueden cambiar constantemente.

La identidad lógica del servicio debe permanecer estable.

Esto es fundamental para entornos dinámicos y autoscaling.

7. Identity Hierarchy

La arquitectura conceptual será:

Platform
   │
   ├── Organization
   │
   └── Service Identity
          │
          ├── Workload
          ├── Credentials
          └── Scopes

Cuando el servicio opera sobre recursos de un tenant:

Service Identity
       +
Tenant Context
       +
Authorization Context

debe acompañar la solicitud.

8. Preferred Authentication Model

Para comunicaciones internas se favorecerá:

Short-Lived Credential

sobre:

Long-Lived Static Credential

Ejemplo:

Service A
   ↓
Identity Provider
   ↓
Short-Lived Credential
   ↓
Service B

Esto reduce la ventana de exposición ante compromiso.

9. Credential Options

La arquitectura podrá soportar diferentes mecanismos:

Option A — OAuth 2.1 Client Credentials
Service A
   ↓
Identity
   ↓
Access Token
   ↓
Service B
Option B — Signed JWT Assertion
Service A
   ↓
Signed Assertion
   ↓
Identity
   ↓
Access Token
Option C — mTLS
Service A
   ⇄ TLS Client Certificate
   ⇄
Service B
Option D — Workload Identity Provider
Workload
   ↓
Platform Identity
   ↓
Short-Lived Credential

La elección concreta de mecanismo queda pendiente de la arquitectura de plataforma.

10. No Single Mechanism Mandate

EVOXA no establecerá un único mecanismo universal para todos los escenarios.

La decisión dependerá de:

entorno;
trust boundary;
infraestructura;
protocolo;
latencia;
compliance;
operación;
soporte del proveedor.

Sin embargo:

Static shared secrets should not be the default service-to-service authentication mechanism.

11. OAuth2.1 / Client Credentials

Para APIs internas donde OAuth sea apropiado, el flujo conceptual será:

Service A
    │
    │ Client Authentication
    ▼
Identity
    │
    │ Short-Lived Access Token
    ▼
Service A
    │
    │ Authorization: Bearer
    ▼
Service B

El token deberá representar una identidad de servicio, no un usuario ficticio.

12. Short-Lived Access Tokens

Los tokens de servicio deberán tener una duración limitada.

Conceptualmente:

issued_at
     ↓
ACTIVE
     ↓
expires_at
     ↓
INVALID

Un servicio deberá obtener un nuevo token mediante un mecanismo autorizado.

El TTL exacto queda pendiente.

13. JWT Access Tokens

Cuando se utilicen JWT:

deberán ser firmados criptográficamente;
utilizarán algoritmos aprobados;
deberán soportar kid;
deberán validar iss;
deberán validar aud;
deberán validar exp;
deberán contener identidad suficiente;
no deberán incluir secretos.

Claims conceptuales:

sub
iss
aud
iat
exp
jti
service_account_id
tenant_id
scope

El catálogo definitivo queda sujeto a API/Authentication Specification.

14. Audience Restriction

Los tokens de servicio deberán restringirse al servicio destino cuando el mecanismo lo permita.

Ejemplo:

aud = training-api

no debería utilizarse automáticamente para:

billing-api

Esto reduce token replay y movimiento lateral.

15. Issuer Validation

El servicio receptor deberá validar el issuer esperado.

Conceptualmente:

Token
  ↓
Issuer Validation
  ↓
Signature Validation
  ↓
Audience Validation
  ↓
Expiration
  ↓
Identity

Un token válido criptográficamente pero emitido por una autoridad no confiable deberá rechazarse.

16. mTLS

mTLS podrá utilizarse como mecanismo complementario o principal para autenticación de workloads.

Conceptualmente:

Service A
   │
   │ Client Certificate
   ▼
Service B

El certificado permite establecer identidad del workload durante el handshake TLS.

mTLS puede ser especialmente apropiado para:

comunicaciones internas;
service mesh;
entornos de alta confianza operacional;
autenticación mutua.

La estrategia concreta de certificados queda pendiente.

17. mTLS Does Not Replace Authorization

Aunque mTLS autentique:

Service A

no implica:

Service A can do everything.

El flujo sigue siendo:

mTLS Authentication
       ↓
Service Identity
       ↓
Authorization
       ↓
Policy
18. Workload Identity Provider

En entornos cloud/Kubernetes, EVOXA podrá utilizar una plataforma de workload identity.

Conceptualmente:

Pod / Workload
      ↓
Platform Attestation / Identity
      ↓
Identity Provider
      ↓
Short-Lived Credential

Esto evita almacenar permanentemente secrets dentro de workloads.

La tecnología concreta queda pendiente.

19. Kubernetes Considerations

El Blueprint contempla una plataforma cloud-ready y Kubernetes como baseline tecnológico posible.

Por ello, la arquitectura debe ser compatible conceptualmente con:

Deployment
   ↓
Pod
   ↓
Workload Identity
   ↓
Service Credential

pero Kubernetes no debe convertirse en una dependencia del Domain Layer.

La integración deberá permanecer en Infrastructure/Platform.

20. Service Discovery

La autenticación y service discovery deben mantenerse conceptualmente separadas.

Por ejemplo:

Service Discovery
    ↓
Where is training-api?

mientras:

Identity
    ↓
Who is calling training-api?

Conocer la ubicación de un servicio no constituye autenticación.

21. Network Location Is Not Identity

EVOXA no deberá confiar únicamente en:

IP Address
Subnet
VPC
Namespace
Pod

para determinar autorización.

El principio será:

Network location is contextual evidence, not sufficient identity.

Esto sigue el modelo Zero Trust.

22. Service-to-Service Authorization

Una vez autenticado:

Service A

el receptor construirá el Authorization Context:

Actor Type
Service Account
Tenant
Scopes
Permissions
Target Resource
Operation
Policy
Risk

y evaluará:

RBAC
+
ABAC
+
Tenant Isolation
+
Ownership
+
Security Policy
23. Service Scopes

Cada servicio deberá recibir únicamente los scopes requeridos.

Ejemplo:

training-worker

podría tener:

training:read
training:write

pero no:

identity:admin
billing:delete
organization:delete

salvo autorización explícita.

24. Service-to-Service Trust Matrix

La plataforma deberá poder representar relaciones como:

Caller	Target	Capability
Training	Identity	identity validation
Nutrition	Training	training read
Billing	Identity	identity validation
Notification	Identity	notification context
AI Orchestrator	Training	scoped operations

La matriz definitiva deberá derivarse del modelo de permisos y no mantenerse como lógica duplicada arbitraria en cada servicio.

25. Tenant Context

Un servicio puede operar:

Tenant-bound
Service
  ↓
Organization A
Platform-level
Platform Service
  ↓
Multiple Organizations

La segunda categoría requiere privilegios significativamente mayores y controles adicionales.

Un servicio platform-level no debe poder acceder arbitrariamente a datos tenant sin una autorización explícita.

26. Cross-Tenant Operations

Las operaciones cross-tenant deben ser excepcionales.

Deben requerir:

Platform Authorization
+
Explicit Scope
+
Policy
+
Audit

Nunca:

service = platform
→ access everything
27. Delegated User Context

Un servicio puede necesitar ejecutar una operación en nombre de un usuario.

Debe distinguirse:

Authenticated Actor
    = Service A

de:

Delegated Subject
    = User B

Ejemplo:

Mobile User
      ↓
API Gateway
      ↓
Training Service

El Training Service puede actuar como servicio, pero la operación puede conservar:

user_id = B
service_id = Training

para autorización y auditoría.

28. Delegation Security

La delegación no debe permitir privilege escalation.

Debe cumplirse:

Delegated Permissions
    ≤
Delegator Permissions

salvo mecanismos explícitos de servicio confiable definidos por política.

La delegación deberá ser:

explícita;
limitada;
auditable;
contextual.
29. AI Service-to-Service

Los AI Services deberán autenticarse como workloads o Service Accounts.

Ejemplo:

AI Orchestrator
      ↓
Workload Identity
      ↓
Scoped Credential
      ↓
Training Tool/API

No deberán utilizar credenciales personales.

30. AI Agent Delegation

Cuando un AI Agent actúe en nombre de un usuario:

User
 ↓
AI Agent
 ↓
Tool
 ↓
Service

el contexto deberá conservar la separación entre:

Human Subject
AI Agent
Service

Esto permitirá attribution completa.

Conceptualmente:

subject_user_id
agent_id
service_account_id
credential_id
31. AI Cannot Self-Elevate

Un AI Agent no puede:

Request broader scope
      ↓
Approve itself
      ↓
Receive privileged credential

La emisión de nuevas credenciales deberá estar controlada por Identity y Security Policy.

32. Credential Rotation

La plataforma debe permitir rotación sin downtime.

Ejemplo:

Credential A
     │
     ├── ACTIVE
     │
Credential B
     │
     └── NEW

Durante una ventana controlada:

A + B

pueden coexistir.

Después:

A → REVOKED
B → ACTIVE

El período exacto de overlap queda pendiente.

33. Key Rotation

Para JWT/mTLS/workload credentials deberán existir procesos separados para:

Signing Keys
Certificate Authorities
Service Credentials
Credential Protection Keys

No deberá existir una única clave universal.

Esto sigue ADR-IDENTITY-008 y ADR-IDENTITY-027.

34. Revocation

La revocación debe poder afectar:

Single Credential
Service Account
Workload Identity
Tenant
Platform Identity

El receptor debe poder detectar rápidamente credenciales inválidas cuando el mecanismo lo requiera.

Para tokens de corta duración, la expiración reduce la ventana de exposición.

35. Revocation vs JWT Statelessness

Un JWT puede ser criptográficamente válido y aun así el Service Account haber sido:

SUSPENDED

o:

REVOKED

Por lo tanto:

Cryptographic validity is not equivalent to authorization validity.

El servicio receptor debe aplicar las reglas correspondientes para account/service state.

36. Replay Protection

Debe mitigarse el replay de credenciales.

Mecanismos posibles:

short TTL;
audience restriction;
jti;
mTLS binding;
sender-constrained tokens;
credential rotation;
revocation;
nonce cuando corresponda.

La combinación exacta queda pendiente.

37. Credential Theft Response

Ante sospecha de compromiso:

Credential Compromise
        ↓
Revoke
        ↓
Rotate
        ↓
Assess Service Account
        ↓
Analyze Usage
        ↓
Security Event
        ↓
Incident Response

Si existe evidencia de movimiento lateral:

Assess Related Credentials

podrá ser requerido.

38. Rate Limiting

Las comunicaciones internas también deberán tener límites.

No se debe asumir:

Internal Network
=
Trusted Unlimited Traffic

Deberán considerarse:

Service
Credential
Tenant
Endpoint
Operation
Risk

Esto protege ante:

bugs;
loops;
compromised workloads;
denial of service interno;
AI runaway behavior.
39. Circuit Breakers and Timeouts

Service-to-service authentication no debe producir dependencias infinitas.

Los clientes deberán utilizar:

timeout;
bounded retry;
exponential backoff;
circuit breaker cuando corresponda.

Los retries de operaciones no idempotentes deben estar protegidos mediante Idempotency-Key según ADR-026.

40. Authentication Failure

Si Identity o el mecanismo de autenticación no puede verificar la identidad:

Authentication unavailable
        ↓
DENY

No:

Authentication unavailable
        ↓
Trust caller

La seguridad debe fallar cerradamente.

41. Identity Service Dependency

Una preocupación importante es:

Training
  ↓
Identity

Si cada request requiere una llamada sincrónica a Identity, Identity podría convertirse en un bottleneck.

Por ello, se favorecerán:

validación local de JWT cuando sea apropiado;
JWKS caching;
short-lived credentials;
caching controlado;
policy caching con invalidation;
mecanismos de workload identity.

Esto debe respetar las garantías de revocación y autorización.

42. JWKS

Cuando se utilicen JWT firmados asimétricamente:

Identity
   ↓
JWKS
   ↓
Services

Los servicios podrán mantener una copia cacheada de claves públicas.

La cache debe:

respetar key rotation;
actualizarse ante kid desconocido;
tener TTL;
evitar dependencia remota en cada request.

Las claves privadas nunca deben distribuirse a los servicios consumidores.

43. Secret Manager

Los static credentials que sean inevitables deberán obtenerse desde:

Secret Manager

y no desde:

Git
Dockerfile
Source Code

La estrategia seguirá ADR-IDENTITY-027.

44. Environment Separation

Credentials de:

Development
QA
Staging
Production

deben estar separadas.

Una credential de Production no debe funcionar accidentalmente en Development o QA, cuando el mecanismo permita restringir audiencia/issuer/entorno.

45. Production Security

Production deberá aplicar controles superiores.

Ejemplo:

Development
→ Static credentials may be temporarily permitted

Production
→ Workload Identity / short-lived credentials preferred

Las excepciones deberán ser explícitas y auditables.

46. Observability

Cada request service-to-service deberá poder correlacionarse mediante:

correlation_id
trace_id
causation_id

y atribuirse a:

service_account_id
credential_id
tenant_id

cuando sea necesario.

No debe registrarse el token.

47. Security Events

Deberán contemplarse eventos como:

SERVICE_AUTH_SUCCESS
SERVICE_AUTH_FAILURE
SERVICE_CREDENTIAL_REJECTED
SERVICE_SCOPE_DENIED
SERVICE_AUDIENCE_MISMATCH
SERVICE_IDENTITY_SUSPENDED
SERVICE_CREDENTIAL_REVOKED
SERVICE_CREDENTIAL_COMPROMISED
SERVICE_IMPERSONATION_ATTEMPT
SERVICE_CROSS_TENANT_ATTEMPT
SERVICE_ANOMALOUS_TRAFFIC

El catálogo final será definido mediante Event Specification.

48. Audit

Una operación service-to-service deberá permitir responder:

Who?
Which service?
Which credential?
Which tenant?
Which target?
Which resource?
Which operation?
Was delegation used?
Which policy?
Which result?

Ejemplo conceptual:

actor_type = SERVICE
service_account_id = ...
credential_id = ...
tenant_id = ...
delegated_user_id = ...
target_service = ...
operation = ...
decision = ALLOW
49. Persistence

Las estructuras conceptuales pueden incluir:

service_accounts
service_credentials
service_scopes
service_account_scopes
workload_identities
workload_bindings
credential_usage

Las tablas definitivas corresponden a Persistence Specification / IS-IDENTITY-015 y a futuras IS específicas.

50. Concurrency

Deben controlarse especialmente:

Credential Rotation
Credential Revocation
Service Suspension
Token Issuance
Token Validation
Workload Registration

Ejemplo:

Credential A
    ↓
REVOKE

Concurrent request
    ↓
Credential A
    ↓
REJECT

No deberá existir una condición de carrera que permita continuar utilizando una credencial después de una revocación efectiva.

51. Disaster Recovery

El estado crítico debe sobrevivir correctamente a recuperación:

Service Accounts;
credential status;
revocations;
scopes;
tenant bindings;
workload identities;
signing keys;
certificate authorities;
audit/security events.

Una restauración no puede reactivar accidentalmente una identidad previamente revocada.

52. Testing Strategy

Deberán existir pruebas para:

Authentication
valid credential;
invalid credential;
expired token;
wrong audience;
wrong issuer;
invalid signature.
Authorization
allowed scope;
denied scope;
cross-tenant denial;
ownership denial;
suspended service.
Security
credential replay;
credential theft;
credential rotation;
revocation race;
impersonation;
privilege escalation.
Infrastructure
Identity unavailable;
JWKS unavailable;
Secret Manager unavailable;
certificate expiration;
provider failure.
AI
Agent privilege escalation;
Agent credential leakage;
delegated authorization;
runaway service calls.
53. Performance

La autenticación service-to-service deberá respetar los objetivos generales de EVOXA.

Particularmente:

API P95 < 300 ms
Authentication P95 < 500 ms

según los objetivos establecidos en el Blueprint.

Se deberá evitar una llamada remota innecesaria a Identity en cada request cuando una validación local segura sea posible.

54. Consequences
Positivas
Reduce dependencia de secretos estáticos.
Permite Zero Trust.
Mejora attribution.
Reduce impacto de credential theft.
Facilita autoscaling.
Compatible con Kubernetes/cloud.
Permite short-lived credentials.
Mejora separación de servicios.
Facilita AI Agent security.
Permite rotación sin downtime.
Reduce riesgo de movimiento lateral.
Negativas
Mayor complejidad de infraestructura.
Requiere Identity Provider o mecanismo equivalente.
Requiere gestión de certificados/keys.
Requiere observabilidad distribuida.
Mayor complejidad de debugging.
Requiere testing de failover y rotación.
55. Alternatives Considered
A. Shared API Key Between Services

Rejected.

No proporciona una identidad suficientemente granular ni lifecycle adecuado.

B. User Credentials for Service Calls

Rejected.

Confunde identidad humana con identidad de máquina.

C. Long-Lived JWT

Rejected as default.

Aumenta la ventana de exposición.

D. IP Allowlisting as Primary Authentication

Rejected.

Network location no es identidad.

E. mTLS Only

Not universally adopted.

Es una excelente capa de autenticación, pero no resuelve por sí sola autorización, scopes, tenant isolation o delegation.

F. OAuth Only

Not universally mandated.

Puede ser adecuado para APIs, pero no necesariamente para todos los límites de infraestructura.

G. Static Secrets Everywhere

Rejected as default.

Incompatible con la dirección de workload identity y least privilege.

56. Non-Negotiable Security Rules
Service-to-service authentication debe identificar inequívocamente al workload.
Service identity no debe depender de un usuario humano.
Static shared secrets no serán el mecanismo predeterminado.
Short-lived credentials serán preferidos.
Authentication no implica autorización.
Authorization debe aplicar RBAC/ABAC/tenant isolation/policy.
Network location no es suficiente como identidad.
Tokens deben tener audience cuando el mecanismo lo permita.
JWT debe validarse criptográficamente.
JWT signing keys privadas nunca se distribuyen a consumidores.
Credential revocation debe ser soportada.
Credential rotation debe ser soportada.
Credential secrets nunca deben aparecer en logs.
Production credentials deben mantenerse fuera del código.
Cross-tenant access requiere autorización explícita.
Impersonation requiere autorización.
Delegation debe ser limitada.
AI Agents no pueden elevar privilegios.
Authentication failure debe fail-closed.
Identity outage no debe producir implicit trust.
Service-to-service traffic debe tener rate limiting apropiado.
Retries deben respetar idempotency.
Audit debe permitir attribution.
DR no puede reactivar credenciales revocadas.
Service identity y workload instance deben mantenerse conceptualmente separadas.
57. Traceability
Requirement	Source / Relationship
Identity Architecture	Identity ESP / ARCHITECTURE-MAP
Authorization	ADR-IDENTITY-005
Cryptographic Keys	ADR-IDENTITY-008
Token Storage	ADR-IDENTITY-009
API Tokens / Service Accounts	ADR-IDENTITY-037
Idempotency	ADR-IDENTITY-026
Secrets / KMS	ADR-IDENTITY-027
Observability	ADR-IDENTITY-028
Disaster Recovery	ADR-IDENTITY-029/030
Rate Limiting	ADR-IDENTITY-032
AI Security	Blueprint / Identity Security
Service-to-Service Authentication	ADR-IDENTITY-038

El Blueprint establece como baseline tecnológico posible Python/FastAPI, PostgreSQL, Redis, Docker, Kubernetes y GitHub Actions, pero esas elecciones no deben interpretarse como decisiones irrevocables del Domain Layer.

58. Dependencies

Principalmente:

ADR-IDENTITY-005
ADR-IDENTITY-008
ADR-IDENTITY-009
ADR-IDENTITY-025
ADR-IDENTITY-026
ADR-IDENTITY-027
ADR-IDENTITY-028
ADR-IDENTITY-029
ADR-IDENTITY-030
ADR-IDENTITY-032
ADR-IDENTITY-037

Y deberá materializarse mediante:

IS-IDENTITY-038
Technical Tasks
API Contracts
Persistence Specification
Security Specification
Event Specification
Infrastructure Specification
Testing Specification
59. Pending Decisions

Todavía deben definirse formalmente:

Authentication Protocol
OAuth 2.1 Client Credentials;
JWT Assertion;
mTLS;
workload identity;
combinación de mecanismos.
Workload Identity
provider;
registration;
attestation;
binding;
lifecycle.
JWT
exact algorithm;
TTL;
claims;
JWKS caching;
key rotation.
mTLS
CA hierarchy;
certificate TTL;
automatic issuance;
rotation;
revocation;
service mesh.
Service Authorization
scope catalog;
permission mapping;
service trust matrix;
cross-tenant policies.
Delegation
user-to-service;
service-to-service;
AI Agent delegation;
token exchange;
impersonation.
Infrastructure
Kubernetes integration;
service mesh;
Secret Manager;
KMS;
identity provider;
multi-region behavior.

Estos puntos deberán cerrarse antes de la implementación definitiva.

60. Acceptance Criteria

El ADR se considerará correctamente implementado cuando:

 Cada workload tenga una identidad de máquina identificable.
 Service Identity esté separada de User Identity.
 Service Identity esté separada de workload instances.
 Exista un mecanismo seguro de service-to-service authentication.
 Short-lived credentials sean soportadas.
 Static credentials no sean el mecanismo predeterminado.
 JWT, cuando sea utilizado, sea validado criptográficamente.
 Audience e issuer sean validados.
 Credential rotation sea soportada.
 Credential revocation sea soportada.
 Authentication y Authorization permanezcan separadas.
 Scopes sean mínimos.
 Tenant isolation sea aplicado.
 Cross-tenant access sea explícito.
 Delegation sea limitada y auditable.
 Impersonation sea controlada.
 AI Agents tengan identidad propia.
 AI Agents no puedan autoelevar privilegios.
 Secrets no aparezcan en logs/events/traces.
 Production secrets estén fuera del código.
 Identity failures sean fail-closed.
 JWKS/key rotation sean soportados cuando corresponda.
 Rate limiting proteja service-to-service traffic.
 Audit permita atribuir las operaciones.
 DR preserve correctamente el estado de las credenciales.
 Existan pruebas de replay, revocation, rotation, failover, privilege escalation y cross-tenant isolation.
61. Next ADR

Con ADR-IDENTITY-038 queda definida la estrategia conceptual para la autenticación entre servicios y la identidad de workloads de EVOXA.

El siguiente paso natural es:

ADR-IDENTITY-039 — Identity OAuth2.1 & OpenID Connect Protocol Strategy

Este ADR permitirá definir formalmente la capa de protocolos sobre la arquitectura que acabamos de establecer:

Human Authentication
        │
        ├── OAuth 2.1
        ├── OpenID Connect
        │
        └── MFA / Step-Up
                 
Machine Authentication
        │
        ├── OAuth 2.1 M2M
        ├── Workload Identity
        └── mTLS / JWT

y establecer qué mecanismo se utilizará para usuarios, aplicaciones, Service Accounts, integraciones externas y AI Agents, antes de pasar a los ADRs posteriores y finalmente a las Implementation Stories → Technical Tasks → Source Code.
