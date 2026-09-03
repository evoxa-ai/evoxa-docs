ADR-IDENTITY-037 — Identity API Token & Service Credential Security Strategy

Status: Proposed
Version: 1.0
Date: 2026-09-03
Domain: Identity
Priority: Critical
Type: Security / Architecture / Identity

1. Context

EVOXA Identity no solo debe autenticar usuarios humanos. La plataforma también deberá soportar identidades no humanas para integraciones, servicios internos, automatizaciones y, eventualmente, AI Agents.

Esto introduce una separación fundamental:

Human Identity
      ≠
Service Identity
      ≠
API Credential
      ≠
AI Agent Identity

Una credencial utilizada por una aplicación o servicio no debe tratarse como una contraseña de usuario ni como un access token de una sesión humana.

La estrategia debe integrarse con:

Authentication;
Authorization;
RBAC / ABAC;
Tenant Isolation;
Sessions;
API Tokens;
Service Accounts;
Security Policy Engine;
KMS / Secret Management;
Audit & Security Events;
Rate Limiting;
Event Architecture;
AI Agent Security;
Observability.

Este ADR continúa especialmente:

ADR-IDENTITY-005 — Authorization Model;
ADR-IDENTITY-006 — Persistence;
ADR-IDENTITY-008 — Cryptographic Key Management;
ADR-IDENTITY-009 — Token Storage;
ADR-IDENTITY-025 — Refresh Token Cryptography & Concurrency;
ADR-IDENTITY-027 — Runtime Configuration, Secrets & KMS;
ADR-IDENTITY-028 — Observability & Operational Security;
ADR-IDENTITY-032 — Rate Limiting & Abuse Prevention;
ADR-IDENTITY-036 — Email Verification & Communication Security.
2. Problem Statement

EVOXA necesitará autenticar escenarios como:

Mobile/Web Application
        ↓
EVOXA API

pero también:

Internal Service
        ↓
EVOXA API
External Integration
        ↓
EVOXA API

y eventualmente:

AI Agent
        ↓
Authorized EVOXA Tool/API

Una arquitectura que utilice una única credencial para todos estos escenarios produciría:

exceso de privilegios;
dificultad para revocar accesos;
dificultad para atribuir acciones;
secretos compartidos;
riesgo de movimiento lateral;
problemas de rotación;
dificultad para aplicar tenant isolation;
dificultad para auditar acciones de máquinas.

Por lo tanto:

Machine identities must be first-class Identity entities with independently managed credentials, authorization and lifecycle.

3. Decision

EVOXA adoptará una estrategia de Service Identity + Scoped Credentials + Explicit Authorization.

Conceptualmente:

                     IDENTITY
                        │
          ┌─────────────┴─────────────┐
          │                           │
    Human Identity              Machine Identity
          │                           │
       User                      Service Account
          │                           │
      Sessions                  Credentials
          │                           │
      JWT Access              API Token / M2M

Las credenciales de servicios deberán:

identificar inequívocamente al consumidor;
tener scopes/permisos limitados;
pertenecer a un tenant cuando corresponda;
tener lifecycle;
poder rotarse;
poder revocarse;
ser auditables;
evitar almacenamiento plaintext;
respetar Security Policy;
soportar attribution;
evitar privilegios implícitos.
4. Machine Identity Model

Se establecerá conceptualmente una entidad:

ServiceAccount

que representa una identidad no humana.

Ejemplo:

ServiceAccount
├── id
├── organization_id
├── name
├── description
├── status
├── type
├── created_at
├── updated_at
├── expires_at
└── metadata

Estados conceptuales:

PENDING
ACTIVE
SUSPENDED
DISABLED
EXPIRED
REVOKED

El lifecycle deberá ser explícito y concurrency-safe.

5. Service Account vs User

Una Service Account no deberá representarse como un usuario humano ficticio.

Debe existir una distinción explícita:

User
   → Human Actor

ServiceAccount
   → Machine Actor

Esto permite que Audit/Security Events distingan:

actor_type = USER

de:

actor_type = SERVICE

y eventualmente:

actor_type = AI_AGENT
6. Credential Model

Una Service Account podrá tener una o más credenciales.

Conceptualmente:

ServiceAccount
       │
       ├── Credential A
       ├── Credential B
       └── Credential C

Esto permite:

rotación gradual;
múltiples aplicaciones;
transición entre credenciales;
revocación individual;
separación por entorno;
recuperación operacional.

Una credencial no deberá representar por sí misma toda la identidad.

7. API Token Strategy

Los API Tokens serán opaque credentials por defecto.

No se adoptará como estrategia principal un token JWT de larga duración para integraciones externas.

Conceptualmente:

API Token
    ↓
Random Secret
    ↓
Stored Protected Representation
    ↓
Credential Validation
    ↓
Service Identity
    ↓
Authorization

Esto mantiene separación entre:

Credential

y:

Authorization
8. API Token Structure

Un token puede utilizar una estructura conceptual:

<public_identifier>.<secret>

El identificador público permite localizar la credencial sin almacenar necesariamente el secreto.

Conceptualmente:

Credential
├── id
├── service_account_id
├── token_identifier
├── token_hash
├── status
├── created_at
├── expires_at
├── last_used_at
├── revoked_at
└── metadata

El secreto nunca debe almacenarse en plaintext.

9. Token Entropy

Los secretos deberán generarse mediante un CSPRNG.

No deberán utilizarse:

IDs secuenciales;
timestamps;
UUIDs como único secreto;
información del tenant;
nombres del servicio;
valores predecibles.

La entropía exacta y encoding quedan pendientes de especificación criptográfica detallada.

10. Credential Storage

Los API Token secrets no deben almacenarse en texto plano.

Se deberá utilizar una representación protegida, por ejemplo:

token_hash

o un mecanismo criptográfico equivalente.

La decisión exacta entre:

hash;
HMAC;
pepper;
KMS-backed derivation;

deberá alinearse con ADR-IDENTITY-027 y una futura especificación criptográfica.

11. Token Presentation

Un API Token no deberá aparecer:

en URLs;
query parameters;
logs;
traces;
eventos;
mensajes de error;
analytics;
screenshots automatizados;
AI context.

El mecanismo preferido será:

Authorization: Bearer <token>

o un esquema explícitamente definido por la API.

12. Token Lifecycle

Cada credential tendrá un lifecycle explícito:

CREATED
   ↓
ACTIVE
   ↓
ROTATING
   ↓
REVOKED

También:

ACTIVE
   ↓
EXPIRED

Una credencial:

REVOKED

no podrá volver a ACTIVE.

13. Credential Rotation

La rotación debe soportar overlap controlado.

Ejemplo:

Credential A → ACTIVE
Credential B → CREATED

        ↓

Credential A → ROTATING
Credential B → ACTIVE

        ↓

Credential A → REVOKED

Esto permite:

deployment gradual;
actualización de secrets;
zero downtime;
rollback controlado.

El período exacto de overlap queda pendiente.

14. Credential Expiration

Las credenciales podrán tener expiración obligatoria dependiendo de:

tipo;
tenant;
entorno;
riesgo;
policy;
integración.

Las credenciales críticas o privilegiadas deberían evitar lifetime indefinido salvo excepción explícita y controlada.

El TTL exacto queda pendiente.

15. Credential Revocation

La revocación debe ser inmediata desde el punto de vista de la autoridad de Identity.

Casos:

Credential Compromise
Service Disabled
Service Decommissioned
Tenant Suspended
Security Incident
Manual Revocation
Expiration
Policy Violation

La revocación debe quedar auditada.

16. Service Account Lifecycle

El estado del Service Account debe afectar a todas sus credenciales.

Por ejemplo:

ServiceAccount = SUSPENDED
       ↓
Credentials
       ↓
Authentication rejected

No debe ser posible mantener una credencial funcional perteneciente a un Service Account deshabilitado.

17. Organization / Tenant Binding

Cuando un Service Account pertenezca a una organización:

ServiceAccount
       ↓
Organization
       ↓
Tenant Context

toda autenticación deberá establecer correctamente el tenant.

Una credencial de:

Organization A

no puede autenticarse como:

Organization B

ni ejecutar operaciones sobre recursos de B salvo una autorización explícita de plataforma.

18. Authorization

Autenticar una Service Account no significa otorgarle acceso universal.

El flujo será:

Credential
    ↓
Authenticate Service Identity
    ↓
Build Authorization Context
    ↓
RBAC / ABAC
    ↓
Tenant Isolation
    ↓
Ownership
    ↓
Security Policy
    ↓
ALLOW / DENY / CHALLENGE

La autorización seguirá centralizada en Identity.

No deberá duplicarse lógica de autorización arbitrariamente en cada servicio.

19. Scopes

Las credenciales podrán tener scopes explícitos.

Ejemplo conceptual:

users:read
organizations:read
training:read
training:write
nutrition:read

Los scopes deberán ser:

allowlisted;
versionados;
auditables;
mínimos;
revocables.

No deberá existir:

scope = *

como mecanismo normal de integración.

20. Scope vs Permission

Debe distinguirse:

Scope

de:

Permission

El scope puede expresar:

Qué capacidad solicita una credencial.

La permission expresa:

Qué operación está autorizada en el modelo de autorización de EVOXA.

Por tanto:

Credential Scope
       ↓
Authorization Context
       ↓
Permission / Policy Evaluation

Un scope nunca debe saltarse RBAC/ABAC/tenant isolation.

21. Resource-Level Authorization

Cuando sea necesario, el acceso deberá restringirse también por:

tenant;
resource;
ownership;
organization;
environment;
data classification;
policy.

Ejemplo:

training:read

no implica:

all_training_data

Puede significar solamente:

authorized_training_resources
22. Service-to-Service Authentication

Para comunicación interna se priorizará una estrategia de identidad de workload/service.

Conceptualmente:

Service A
   ↓
Service Identity
   ↓
Authentication
   ↓
Service B

Las credenciales estáticas compartidas deben evitarse cuando exista una alternativa segura de workload identity.

La solución concreta queda pendiente de decisión de plataforma.

23. M2M Authentication

Para Machine-to-Machine:

Client Service
      ↓
Identity
      ↓
M2M Authentication
      ↓
Short-Lived Access Credential
      ↓
Target Service

Cuando sea apropiado, EVOXA podrá utilizar credenciales de corta duración en lugar de secretos estáticos de larga duración.

Los detalles del protocolo concreto quedan pendientes.

24. Static API Tokens vs Short-Lived Tokens
Static API Tokens

Ventajas:

simples;
fáciles de integrar;
compatibles con sistemas legacy.

Desventajas:

mayor impacto si son robados;
requieren rotación;
pueden permanecer válidos durante mucho tiempo.
Short-Lived Credentials

Ventajas:

menor ventana de exposición;
mejor seguridad;
menor impacto ante compromiso.

Desventajas:

mayor complejidad;
requieren mecanismos de emisión/renovación.

EVOXA utilizará ambos cuando sea necesario, favoreciendo short-lived credentials para escenarios donde la infraestructura lo permita.

25. AI Agent Identity

Los AI Agents deberán disponer de identidad explícita.

No se debe utilizar:

AI Agent
   ↓
Shared Admin API Token

En su lugar:

AI Agent
   ↓
Agent Identity
   ↓
Authorized Tool
   ↓
Scoped Credential
   ↓
Identity Authorization
   ↓
Operation

El Agent deberá ser atribuible.

26. AI Agent Least Privilege

Un AI Agent debe recibir únicamente las capacidades necesarias para ejecutar su tarea.

Ejemplo:

Nutrition Agent

puede tener:

nutrition:read
nutrition:write

pero no:

identity:admin
billing:admin
organization:delete

salvo que una política explícita lo permita y exista un caso legítimo.

27. AI Agent Tool Boundary

El Agent no debería recibir directamente:

database credentials;
KMS credentials;
Secret Manager credentials;
private keys;
master API tokens;
refresh tokens;
passwords.

La arquitectura preferida es:

AI Agent
   ↓
Tool Interface
   ↓
Authorization
   ↓
Backend Operation

El backend controla los privilegios reales.

28. Credential Delegation

Una Service Account no debe poder delegar arbitrariamente todos sus privilegios.

Cuando exista delegation:

Actor A
   ↓
Delegates limited capability
   ↓
Actor B

la delegación deberá estar:

explícitamente autorizada;
limitada por scope;
limitada por tenant;
limitada temporalmente;
auditada.
29. Credential Impersonation

La impersonation de un Service Account deberá considerarse una operación privilegiada.

No debe ser posible simplemente presentando una credencial válida.

Debe existir:

Requester Authorization
+
Target Service Authorization
+
Policy Evaluation
+
Audit
30. Credential Ownership

Toda credencial deberá tener un propietario claro:

Credential
   ↓
ServiceAccount
   ↓
Organization / Platform

No se deberán crear credenciales "huérfanas".

Cuando un Service Account sea eliminado/decommissioned:

ServiceAccount
       ↓
Credentials revoked
31. Credential Naming

El nombre de una credencial debe ser descriptivo, pero no debe incluir secretos.

Ejemplo válido:

production-sync-worker

No válido:

production-sync-worker-sk_live_xxxxx
32. Last Used Tracking

Identity podrá registrar:

last_used_at

y metadata operacional limitada.

Puede utilizarse para:

detectar credenciales abandonadas;
identificar integraciones inactivas;
apoyar rotaciones;
detectar anomalías.

No deberá almacenarse información excesiva.

33. Credential Usage Metadata

Puede registrarse de manera controlada:

credential_id
service_account_id
tenant_id
timestamp
operation
resource
client/service identity
correlation_id

No debe almacenarse:

full token

ni secretos derivados innecesarios.

34. Rate Limiting

Las credenciales de servicio estarán sujetas a rate limiting.

Dimensiones potenciales:

ServiceAccount
Credential
Tenant
Endpoint
Operation
IP
Resource
Risk

Una credencial comprometida no debe poder ejecutar ilimitadas operaciones.

Esto complementa ADR-IDENTITY-032.

35. Suspicious Service Credential

El sistema podrá detectar:

uso desde ubicación inesperada;
incremento anormal de volumen;
uso fuera de horario esperado;
operaciones no habituales;
cambios de IP;
credential sharing;
scope violations;
repeated failures.

El resultado puede ser:

ALLOW
CHALLENGE
RATE_LIMIT
SUSPEND
REVOKE

según policy.

36. Emergency Revocation

Debe existir capacidad para revocar rápidamente:

Single Credential
Entire Service Account
All Credentials of Tenant

La última opción será altamente privilegiada y deberá requerir autorización explícita.

Los eventos deben quedar auditados.

37. Credential Compromise

Ante sospecha o confirmación:

Credential Compromise
        ↓
Revoke Credential
        ↓
Assess Service Account
        ↓
Assess Other Credentials
        ↓
Security Event
        ↓
Rotate Credential
        ↓
Investigate Usage

La política podrá determinar si también deben revocarse otras credenciales o suspenderse el Service Account.

38. API Token and Sessions

API Tokens de Service Accounts no deberán utilizar el mismo modelo conceptual que las sesiones humanas.

Human:

User
 ↓
Session
 ↓
Refresh Token
 ↓
Access Token

Machine:

Service Account
 ↓
Credential
 ↓
Machine Authentication
 ↓
Access Credential

Esto evita mezclar lifecycles.

39. JWT for Service Identity

JWT puede utilizarse para representar una identidad de servicio cuando sea apropiado, especialmente para credenciales de corta duración.

Pero:

Long-lived machine credentials should not become effectively permanent JWTs.

Cuando se utilicen JWT:

deberán firmarse mediante mecanismos definidos en ADR-008/027;
deberán utilizar kid;
deberán ser de corta duración;
deberán tener audience;
deberán tener issuer;
deberán identificar claramente el actor;
no deberán contener secretos;
no deberán contener permisos innecesariamente extensos.
40. Service Credential Claims

Para tokens de acceso de corta duración, los claims conceptuales pueden incluir:

sub
iss
aud
iat
exp
jti
service_account_id
tenant_id
scope

Los claims definitivos dependerán de la API Contract y Authentication Specification.

No se debe introducir información sensible innecesaria.

41. Credential Storage in Applications

Las aplicaciones consumidoras deben almacenar sus credenciales según su entorno.

Preferencia:

Workload Identity
       ↓
Short-Lived Credential

antes que:

Static Secret
       ↓
Environment Variable

Cuando un static secret sea inevitable:

Secret Manager;
encrypted storage;
restricted access;
rotation;
audit.

Nunca:

Git;
source code;
Docker image;
frontend bundle.
42. Frontend Restriction

Los API Tokens de Service Accounts no deben exponerse a aplicaciones frontend públicas.

No deberán colocarse en:

Angular bundle
Flutter application
Browser JavaScript
Mobile public configuration

porque cualquier secreto embebido en un cliente distribuido debe considerarse potencialmente recuperable.

Las aplicaciones cliente deberán utilizar mecanismos apropiados de autenticación de usuario o flujos públicos diseñados para clientes no confiables.

43. Tenant Administration

Los administradores de una organización podrán administrar Service Accounts de su propio tenant cuando tengan las permissions necesarias.

No podrán:

acceder a Service Accounts de otro tenant;
obtener plaintext credentials existentes;
consultar secretos;
modificar credentials sin autorización;
saltarse platform security policies.
44. Platform-Level Service Accounts

Algunas identidades pueden pertenecer a la plataforma y no a una organización concreta.

Ejemplo:

Platform Event Processor
Platform Notification Worker
Platform AI Infrastructure

Estas identidades deberán estar claramente clasificadas:

scope = PLATFORM

y sujetas a controles superiores.

No deben utilizarse para ejecutar arbitrariamente acciones como usuario de un tenant.

45. Impersonation of Human Users

Los Service Accounts no deben convertirse automáticamente en usuarios humanos.

Si una operación debe ejecutarse "en nombre de" un usuario:

Service Identity
      +
Delegated User Context

deberá mantenerse explícita la distinción entre:

authenticated_actor

y:

delegated_subject

Esto es especialmente importante para Audit y AI Agents.

46. Audit Attribution

Una acción realizada por un servicio debe poder responder:

Who authenticated?
What service?
Which credential?
Which tenant?
What operation?
Which resource?
Was there delegation?
Which policy?
Which AI Agent, if applicable?

Ejemplo conceptual:

actor_type = SERVICE
service_account_id = ...
credential_id = ...
tenant_id = ...
delegated_user_id = ...
operation = ...

Los secretos nunca forman parte del evento.

47. Security Events

Deberán contemplarse eventos como:

SERVICE_ACCOUNT_CREATED
SERVICE_ACCOUNT_SUSPENDED
SERVICE_ACCOUNT_DISABLED
SERVICE_CREDENTIAL_CREATED
SERVICE_CREDENTIAL_ROTATED
SERVICE_CREDENTIAL_REVOKED
SERVICE_CREDENTIAL_EXPIRED
SERVICE_CREDENTIAL_REUSE_DETECTED
SERVICE_CREDENTIAL_ANOMALY
SERVICE_IMPERSONATION_ATTEMPT
SERVICE_SCOPE_VIOLATION
SERVICE_CREDENTIAL_COMPROMISED

El catálogo final será definido por Event Specification.

48. Outbox Integration

Los cambios críticos deberán utilizar:

Database State
      +
Outbox Event

en la misma transacción cuando corresponda.

Ejemplo:

Revoke Credential
      ↓
DB Transaction
├── credential.status = REVOKED
└── credential.revoked event
      ↓
COMMIT

Esto garantiza consistencia entre estado e integración eventual.

49. Persistence

La persistencia conceptual puede incluir:

service_accounts
service_credentials
service_account_scopes
service_account_permissions
credential_usage_records

además de las estructuras de Identity existentes.

Las tablas y constraints definitivas corresponden a:

IS-IDENTITY-015

y futuras persistence specifications.

50. Concurrency

Debe garantizarse que operaciones como:

Rotate Credential
Revoke Credential
Authenticate Credential
Disable Service Account

sean concurrency-safe.

Ejemplo:

Request A → Revoke Credential → SUCCESS

Request B → Authenticate Same Credential
           → REJECT

No debe existir una ventana en la cual una credencial revocada sea aceptada debido a una condición de carrera.

51. Idempotency

Operaciones administrativas como:

create credential;
rotate credential;
revoke credential;
disable service account;

deberán utilizar Idempotency-Key cuando corresponda.

Un retry no debe generar:

2 credentials

cuando el cliente pretendía crear una sola.

52. Secrets and KMS

La estrategia deberá seguir ADR-IDENTITY-027.

Separación:

Configuration
Secrets
Cryptographic Keys

Las credenciales de Service Accounts son secretos de alta sensibilidad.

Las claves utilizadas para protegerlas deben tener lifecycle independiente.

53. Key Rotation

La rotación de:

JWT Signing Keys
Credential Protection Keys
HMAC/Pepper Keys

debe ser independiente cuando corresponda.

No se debe utilizar una única clave maestra para todas las funciones criptográficas.

54. Observability

Deberán existir métricas para:

service_auth_success
service_auth_failure
credential_revocations
credential_rotations
expired_credentials
scope_denials
service_rate_limits
service_anomalies
impersonation_attempts

Nunca:

api_token_value

como metric/log field.

55. Disaster Recovery

La recuperación debe preservar:

Service Account state;
credential status;
revocations;
expiration;
scopes;
tenant relationships;
audit events.

Un restore no puede convertir:

REVOKED

en:

ACTIVE

accidentalmente.

Esto debe validarse mediante las pruebas de DR definidas en ADR-029/030.

56. AI Agent Credential Recovery

Si un AI Agent pierde una credencial:

Agent Credential Recovery

no debe permitir que el Agent genere por sí mismo una credencial privilegiada.

Debe existir:

Agent
 ↓
Authorized Recovery Mechanism
 ↓
Policy
 ↓
New Scoped Credential

La nueva credencial deberá respetar como mínimo el mismo principio de least privilege.

57. AI Credential Delegation

Un AI Agent podrá recibir credenciales temporales y específicas para una tarea:

Task
  ↓
Scoped Authorization
  ↓
Short-Lived Credential
  ↓
Tool Execution
  ↓
Credential Expiration

Este patrón será preferido frente a entregar un API Token permanente al Agent.

58. Consequences
Positivas
Identidades humanas y de máquinas quedan separadas.
Reduce riesgo de secretos compartidos.
Facilita rotación.
Permite revocación individual.
Mejora attribution.
Soporta multi-tenant.
Permite scopes y least privilege.
Facilita integración M2M.
Prepara EVOXA para AI Agents seguros.
Reduce impacto de credenciales comprometidas.
Compatible con workload identity.
Negativas
Mayor complejidad de Identity.
Requiere lifecycle adicional.
Requiere infraestructura de Secret Manager/KMS.
Necesita mecanismos de rotación.
M2M introduce nuevos escenarios de seguridad.
Requiere mayor cobertura de testing y observabilidad.
59. Alternatives Considered
A. Compartir un API Token Global

Rejected.

Un compromiso afectaría potencialmente toda la plataforma.

B. Utilizar Usuarios Humanos para Integraciones

Rejected.

Mezcla identidades humanas y máquinas y dificulta attribution.

C. JWT Permanente para Integraciones

Rejected.

Una credencial de larga duración firmada se convierte en un secreto operacional de alto impacto.

D. Credenciales Sin Scopes

Rejected.

Viola least privilege.

E. Guardar API Tokens en Plaintext

Rejected.

No cumple los requisitos de protección de secretos.

F. Entregar Credenciales Maestras a AI Agents

Rejected.

Crearía una superficie de compromiso excesivamente elevada.

60. Non-Negotiable Security Rules
Service Accounts son identidades independientes de Users.
Cada credential debe tener propietario.
API Token secrets nunca deben almacenarse plaintext.
API Tokens deben poder revocarse.
API Tokens deben poder expirar.
Credentials deben soportar rotación.
Scopes deben ser allowlisted.
Least privilege es obligatorio.
Scope no reemplaza Authorization Policy.
Tenant isolation es obligatorio.
Service Account suspendido implica rechazo de sus credentials.
Credentials nunca deben aparecer en logs/events/traces.
Credentials no deben estar en frontend bundles.
Workload Identity debe preferirse cuando esté disponible.
M2M debe tener identidad atribuible.
Impersonation requiere autorización explícita.
Delegation debe ser limitada y auditable.
AI Agents no reciben master credentials.
AI Agents no pueden elevar sus propios privilegios.
Revocation debe ser concurrency-safe.
Credential state debe sobrevivir correctamente a DR.
Credential operations críticas deben ser auditadas.
Secrets y cryptographic keys deben seguir ADR-027.
Authentication y Authorization deben permanecer separadas.
Failures de Identity deben fallar cerradamente.
61. Traceability
Requirement	Source / Relationship
Identity architecture	Identity ESP / ARCHITECTURE-MAP
Authorization	ADR-IDENTITY-005
Persistence	ADR-IDENTITY-006
Cryptographic keys	ADR-IDENTITY-008
Token storage	ADR-IDENTITY-009
Refresh Tokens	ADR-IDENTITY-025
Idempotency	ADR-IDENTITY-026
Secrets / KMS	ADR-IDENTITY-027
Observability	ADR-IDENTITY-028
Disaster Recovery	ADR-IDENTITY-029/030
Rate Limiting	ADR-IDENTITY-032
Account Lifecycle	ADR-IDENTITY-033
Password Security	ADR-IDENTITY-034
Email Security	ADR-IDENTITY-036
API Tokens / Service Credentials	ADR-IDENTITY-037

La trazabilidad debe conservar la cadena:

ESP
 ↓
ADR
 ↓
IS
 ↓
Technical Tasks
 ↓
Source Code
 ↓
Tests

como exige el lifecycle del Blueprint.

62. Dependencies

Principalmente:

ADR-IDENTITY-005
ADR-IDENTITY-006
ADR-IDENTITY-008
ADR-IDENTITY-009
ADR-IDENTITY-025
ADR-IDENTITY-026
ADR-IDENTITY-027
ADR-IDENTITY-028
ADR-IDENTITY-029
ADR-IDENTITY-030
ADR-IDENTITY-032
ADR-IDENTITY-033
ADR-IDENTITY-036

La implementación deberá materializarse posteriormente mediante:

IS-IDENTITY-037
Technical Tasks
API Contracts
Persistence Specification
Event Specification
Security Specification
Testing Specification
63. Pending Decisions

Todavía quedan abiertos:

Service Accounts
catálogo definitivo de tipos;
lifecycle exacto;
platform vs organization scope;
ownership model.
API Tokens
entropy exacta;
encoding;
prefix;
hashing/HMAC;
pepper;
TTL;
expiration policy;
maximum credentials per Service Account.
Scopes
catálogo inicial;
naming;
versioning;
relationship con Permissions.
M2M
OAuth2.1 client credentials;
workload identity;
mTLS;
JWT assertion;
combinación de mecanismos.
Rotation
rotation interval;
overlap/grace window;
emergency rotation;
automatic rotation.
AI Agents
Agent identity model;
task-scoped credentials;
delegation;
credential lifetime;
tool authorization boundary.
Infrastructure
Secret Manager;
KMS;
service mesh;
workload identity provider;
provider-specific implementation.

Estos valores deben definirse antes de convertir el ADR en implementación.

64. Acceptance Criteria

El ADR se considerará correctamente implementado cuando:

 Service Accounts sean identidades independientes.
 Service Credentials tengan lifecycle explícito.
 API Tokens sean criptográficamente seguros.
 Tokens no se almacenen en plaintext.
 Tokens puedan expirar.
 Tokens puedan revocarse.
 Token rotation sea soportada.
 Scopes estén allowlisted.
 Least privilege sea aplicado.
 Authorization siga RBAC/ABAC/Policy.
 Tenant isolation sea obligatorio.
 Service Account suspendido invalide sus credentials.
 Frontend no pueda contener Service Credentials.
 M2M authentication sea atribuible.
 Impersonation requiera autorización explícita.
 Delegation esté limitada y auditada.
 AI Agents utilicen identidades propias.
 AI Agents no puedan obtener master credentials.
 Credential revocation sea concurrency-safe.
 Credential operations sean auditables.
 No existan secretos en logs/events/traces.
 Outbox sea utilizado para cambios/eventos correspondientes.
 DR preserve correctamente credential state.
 Existan pruebas de compromise, rotation, revocation, concurrency, abuse y tenant isolation.
65. Next ADR

Con ADR-IDENTITY-037 queda definida la estrategia conceptual para las identidades no humanas, API Tokens, Service Accounts, credenciales M2M y AI Agent Credentials.

El siguiente paso natural es:

ADR-IDENTITY-038 — Identity Service-to-Service Authentication & Workload Identity Strategy

Ahí podemos profundizar específicamente en la frontera entre Service Accounts, OAuth2.1/M2M, workload identity, mTLS, short-lived credentials, service mesh y autenticación interna entre microservicios, sin mezclarla nuevamente con la autenticación de usuarios humanos.
