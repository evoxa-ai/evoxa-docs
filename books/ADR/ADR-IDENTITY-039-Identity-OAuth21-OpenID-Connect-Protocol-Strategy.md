ADR-IDENTITY-039 — Identity OAuth 2.1 & OpenID Connect Protocol Strategy

Status: Proposed
Version: 1.0
Date: 2026-09-03
Domain: Identity
Priority: Critical
Type: Security / Architecture / Protocol

1. Context

EVOXA necesita una estrategia de protocolos estándar para autenticación y autorización que permita soportar de forma coherente:

aplicaciones Web;
aplicaciones Mobile;
APIs;
Service Accounts;
Machine-to-Machine;
integraciones externas;
federación de identidad;
futuros AI Agents.

Los ADR anteriores han establecido la separación entre:

Human Identity
Service Identity
Credential
Session
Authorization

y han definido:

User
   ↓
Authentication
   ↓
Session
   ↓
Access Token

para usuarios, mientras que para servicios:

Service Account
   ↓
Machine Authentication
   ↓
Short-Lived Credential
   ↓
Target Service

Ahora es necesario definir qué papel tendrán OAuth 2.1 y OpenID Connect (OIDC) dentro de esta arquitectura.

2. Problem Statement

OAuth y OpenID Connect resuelven problemas relacionados pero diferentes.

OAuth
→ Authorization

OpenID Connect
→ Authentication / Identity Layer on OAuth

Confundir ambos conceptos podría provocar diseños incorrectos como:

OAuth Access Token
       ↓
Treat as User Identity

o:

ID Token
       ↓
Use as API Authorization Token

EVOXA necesita una separación explícita entre:

autenticación;
autorización;
identidad;
access tokens;
ID tokens;
refresh tokens;
sessions;
service credentials.
3. Decision

EVOXA adoptará conceptualmente:

OAuth 2.1 como protocolo principal para autorización delegada y emisión de access tokens.

y:

OpenID Connect como capa de identidad/autenticación para escenarios de usuarios humanos cuando corresponda.

La arquitectura conceptual será:

                    EVOXA Identity
                          │
             ┌────────────┴────────────┐
             │                         │
        OAuth 2.1                  OpenID Connect
             │                         │
      Authorization              Authentication
             │                         │
       Access Token                ID Token
             │                         │
             └────────────┬────────────┘
                          │
                     EVOXA APIs

OAuth no será utilizado como sustituto conceptual de OIDC para identidad de usuario.

4. Protocol Separation

La separación fundamental será:

Protocolo / Artefacto	Propósito
OAuth 2.1	Authorization
OpenID Connect	User Authentication / Identity
Access Token	Acceso a API/recurso
ID Token	Información de autenticación/identidad para el cliente
Refresh Token	Obtención de nuevos access tokens
Session	Estado de autenticación de Identity
Service Credential	Machine Identity

Por lo tanto:

ID Tokens are not API authorization tokens.

5. OAuth 2.1 Role in EVOXA

OAuth 2.1 será la base conceptual para:

autorización de aplicaciones;
acceso delegado;
access tokens;
scopes;
refresh token flows cuando correspondan;
Machine-to-Machine;
integración con clientes externos.

La implementación deberá seguir las prácticas de seguridad establecidas por el Blueprint y los ADR de Identity.

6. OpenID Connect Role

OIDC se utilizará para escenarios donde EVOXA necesita establecer autenticación de un usuario.

Conceptualmente:

User
 ↓
Authentication
 ↓
OIDC
 ↓
Identity Claims
 ↓
Client

El ID Token representará información sobre la autenticación y el sujeto autenticado para el cliente OIDC.

No deberá utilizarse como sustituto de:

session state;
access token;
authorization policy.
7. Authorization Code Flow

Para aplicaciones Web y Mobile que actúan como clientes públicos, el flujo preferido será conceptualmente:

User
  ↓
Client Application
  ↓
Authorization Request
  ↓
EVOXA Identity
  ↓
Authentication
  ↓
MFA / Policy
  ↓
Authorization
  ↓
Authorization Code
  ↓
Client
  ↓
Token Endpoint
  ↓
Access Token
+
ID Token when OIDC
+
Refresh Token when applicable

El uso de un authorization code evita entregar directamente tokens de acceso en el authorization endpoint.

8. PKCE

Los clientes públicos deberán utilizar PKCE.

Conceptualmente:

Client
  ↓
code_verifier
  ↓
code_challenge
  ↓
Authorization Request

Posteriormente:

Authorization Code
+
code_verifier
       ↓
Token Endpoint

El servidor deberá validar la relación entre:

code_challenge

y:

code_verifier

Esto será obligatorio para clientes públicos.

9. Public vs Confidential Clients

EVOXA deberá distinguir:

Public Client

No puede mantener un secret de cliente de forma confidencial.

Ejemplos:

Mobile App
Browser Application
Confidential Client

Puede proteger credenciales de cliente dentro de un entorno controlado.

Ejemplos:

Backend Application
Trusted Server
Server-side Integration

No se deberá asumir que una aplicación móvil o SPA puede proteger un client_secret.

10. Client Registration

Los clientes OAuth/OIDC deberán registrarse.

Conceptualmente:

OAuthClient
├── client_id
├── client_type
├── redirect_uris
├── allowed_grants
├── allowed_scopes
├── status
├── organization_id
└── metadata

Los secretos de clientes confidenciales deberán gestionarse como secrets.

11. Redirect URI Security

Las redirect_uri deberán estar estrictamente controladas.

No se permitirá como comportamiento normal:

https://example.com/*

o:

wildcard redirect

cuando permita redirecciones arbitrarias.

La URI utilizada en la autorización debe coincidir con una URI registrada según las reglas de OAuth/OIDC.

12. Authorization Code Lifetime

Los authorization codes deberán ser:

de corta duración;
de un solo uso;
asociados al cliente;
asociados al redirect URI;
asociados al contexto de autorización;
protegidos contra replay.

Conceptualmente:

ISSUED
  ↓
CONSUMED

o:

ISSUED
  ↓
EXPIRED

Nunca:

Consumed
  ↓
Reusable
13. Access Token Strategy

Los access tokens serán credenciales de acceso a recursos.

Podrán ser:

JWT;
opaque tokens.

La elección dependerá del caso de uso y de las necesidades de validación/revocación.

Para APIs internas y distribuidas, los JWT firmados asimétricamente pueden facilitar validación local, mientras que opaque tokens pueden proporcionar mayor control centralizado.

La decisión concreta por tipo de cliente queda pendiente.

14. JWT Access Tokens

Cuando EVOXA utilice JWT como access token:

deberá validar como mínimo:

signature
issuer
audience
expiration
not-before when used
subject

y los claims adicionales requeridos por el contexto.

Claims conceptuales:

sub
iss
aud
iat
exp
jti
scope
client_id
tenant_id

No deberán incluirse datos sensibles innecesarios.

15. Audience

Los access tokens deberán restringirse al recurso previsto cuando corresponda.

Ejemplo:

aud = training-api

no significa automáticamente:

aud = billing-api

El receptor deberá rechazar tokens cuya audiencia no corresponda.

16. Issuer

Los servicios deberán validar que el token provenga del issuer esperado.

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
Authorization

Un JWT correctamente firmado pero emitido por una autoridad no confiable debe rechazarse.

17. ID Token

El ID Token pertenece al protocolo OIDC y se utiliza para comunicar información sobre la autenticación del usuario al cliente OIDC.

Conceptualmente:

Identity
   ↓
Authentication
   ↓
ID Token
   ↓
OIDC Client

No deberá utilizarse:

ID Token
   ↓
API Authorization

La API debe recibir y validar un access token destinado a ella.

18. ID Token Validation

El cliente OIDC deberá validar apropiadamente:

issuer;
signature;
audience;
expiration;
nonce cuando corresponda;
claims requeridos.

La validación exacta dependerá del tipo de cliente y protocolo utilizado.

19. Nonce

En flujos OIDC, EVOXA deberá soportar mecanismos para proteger contra replay/mix-up de autenticación, incluyendo nonce cuando corresponda.

Conceptualmente:

Client
 ↓
nonce
 ↓
Authorization Request
 ↓
ID Token
 ↓
nonce validation

Un mismatch deberá producir rechazo.

20. State Parameter

Los clientes deberán utilizar state para correlacionar solicitudes de autorización y proteger el flujo del cliente.

Conceptualmente:

Client
 ↓
state = X
 ↓
Authorization
 ↓
Callback
 ↓
validate state = X

Un state inválido o inesperado debe producir rechazo.

21. PKCE + State + Nonce

Para clientes OIDC modernos, el patrón de seguridad será conceptualmente:

Authorization Request
├── state
├── code_challenge
└── nonce

seguido de:

Authorization Code
       ↓
Token Endpoint
       ↓
ID Token
       ↓
Validate nonce

Cada mecanismo cumple una función diferente y no debe considerarse intercambiable.

22. OAuth Scopes

Los scopes representan las capacidades solicitadas.

Ejemplo:

openid
profile
email
training:read
training:write

openid identifica que la solicitud es OIDC.

Los scopes de OAuth no deben reemplazar el modelo de permisos interno de EVOXA.

El flujo será:

OAuth Scope
      ↓
Authorization Context
      ↓
RBAC / ABAC
      ↓
Security Policy
      ↓
Decision
23. OpenID Connect Scopes

EVOXA podrá soportar scopes estándar de OIDC como:

openid
profile
email

cuando correspondan.

La exposición de claims deberá seguir data minimization.

No se deben devolver claims innecesarios solamente porque el cliente los solicita.

24. Claims and Privacy

Los tokens no deben convertirse en un depósito de información del usuario.

Especialmente no deberán contener:

datos de salud;
información clínica;
información nutricional;
datos deportivos detallados;
secretos;
información interna de otros tenants.

La información debe obtenerse mediante APIs autorizadas.

Esto mantiene:

Identity Token
      ≠
User Data Container
25. Refresh Tokens

Los refresh tokens de OAuth/OIDC deberán seguir las decisiones de:

ADR-IDENTITY-025 — Refresh Token Cryptography & Concurrency

Por lo tanto:

serán stateful;
deberán protegerse criptográficamente;
tendrán lifecycle;
utilizarán rotation;
tendrán reuse detection;
podrán revocarse;
estarán asociados a una sesión/contexto cuando corresponda.

No se deberá crear una segunda estrategia independiente de refresh tokens para OAuth/OIDC.

26. Refresh Token Rotation

Conceptualmente:

Refresh Token A
       ↓
Token Endpoint
       ↓
Access Token
+
Refresh Token B
       ↓
A = USED
B = ACTIVE

Si se detecta reuse:

USED Token A reused
       ↓
Security Event
       ↓
Revoke Family / Session

según las reglas establecidas en ADR-025.

27. OAuth Client Credentials

Para Machine-to-Machine:

Service
   ↓
OAuth Token Endpoint
   ↓
Client Credentials
   ↓
Short-Lived Access Token
   ↓
Target API

Esto será compatible con ADR-IDENTITY-038.

No deberá utilizarse una identidad humana ficticia para representar el servicio.

28. M2M Authorization

El flujo conceptual será:

Service Account
       ↓
Client Authentication
       ↓
Authorization Server
       ↓
Access Token
       ↓
Target Service
       ↓
Authorization Policy

Los permisos serán limitados por:

Service Account;
client;
scopes;
tenant;
resource;
policy.
29. OAuth vs Workload Identity

OAuth 2.1 no reemplaza necesariamente a Workload Identity.

Pueden coexistir:

Workload Identity
       ↓
Authenticates Workload
       ↓
OAuth Authorization Server
       ↓
Short-Lived Access Token

Esto permite separar:

Who is the workload?

de:

What may the workload access?
30. External Integrations

Las integraciones externas podrán utilizar OAuth/OIDC cuando corresponda.

Conceptualmente:

External Client
      ↓
EVOXA Authorization Server
      ↓
Authorization
      ↓
Access Token
      ↓
EVOXA API

Los clientes externos deberán recibir únicamente los scopes necesarios.

31. Third-Party Identity Providers

EVOXA podrá soportar Identity Federation con proveedores externos mediante OIDC u otros mecanismos compatibles.

Conceptualmente:

External IdP
      ↓
Federation
      ↓
EVOXA Identity
      ↓
Local Identity / Session
      ↓
EVOXA APIs

La federación no debe eliminar:

tenant isolation;
local authorization;
Security Policy;
MFA policy cuando corresponda;
auditability.
32. External IdP Trust

Un proveedor externo no debe convertirse automáticamente en autoridad sobre permisos internos.

Por ejemplo:

External IdP
→ authenticates user

pero:

EVOXA
→ determines authorization

La asignación de roles/permisos debe permanecer bajo las políticas de EVOXA.

33. Account Linking

Cuando se vinculen identidades externas con cuentas EVOXA:

External Identity
       ↓
Verified Mapping
       ↓
EVOXA User

el linking deberá ser una operación protegida.

No debe permitirse que un atacante vincule arbitrariamente un IdP externo a una cuenta existente.

34. Authentication Assurance

OIDC Authentication deberá integrarse con:

MFA;
step-up;
Security Policy;
risk assessment;
account lifecycle.

Por ejemplo:

OIDC Authentication
       ↓
Policy
       ↓
MFA required?
       ↓
YES
       ↓
MFA

OIDC no debe convertirse en bypass de políticas internas.

35. Step-Up Authentication

Para operaciones sensibles, EVOXA podrá requerir una autenticación de mayor assurance.

Ejemplo:

Existing Session
       ↓
Sensitive Operation
       ↓
Security Policy
       ↓
REAUTHENTICATE / REQUIRE_MFA

La evidencia de step-up deberá tener duración limitada.

Esto se relaciona directamente con ADR-IDENTITY-024.

36. Consent

Cuando una aplicación solicite autorización delegada, EVOXA podrá presentar consentimiento al usuario cuando corresponda.

El consentimiento deberá mostrar claramente:

Application
Requested Scopes
Purpose
Tenant / Organization Context

No deberá presentar información engañosa.

El consentimiento no reemplaza Security Policy.

Una política puede negar una operación incluso si el usuario intenta consentirla.

37. Scope Escalation

Un cliente no podrá aumentar sus scopes simplemente modificando una solicitud.

Ejemplo:

Client Registered:
training:read

No puede solicitar arbitrariamente:

billing:admin

y esperar autorización.

El Authorization Server debe verificar:

Client Allowed Scopes
+
User Authorization
+
Tenant Policy
+
Platform Policy
38. Redirect Security

Se deberán proteger:

redirect URI;
authorization code;
state;
nonce;
PKCE;
session binding.

No se deberán aceptar redirects arbitrarios.

Esto será especialmente importante para clientes Web.

39. Browser Security

Los tokens no deberán exponerse innecesariamente al navegador.

La estrategia deberá seguir ADR-IDENTITY-009:

Access Token
→ memory where applicable

Refresh Token
→ HttpOnly + Secure cookie

No se deberá utilizar localStorage como almacenamiento predeterminado para refresh tokens.

40. Mobile Security

Para Mobile:

Access Token
→ application memory

Refresh Token
→ OS secure storage

El SDK deberá abstraer el almacenamiento y lifecycle.

No deberán utilizarse mecanismos de almacenamiento público o inseguro.

41. Client Authentication

Para confidential clients, EVOXA podrá soportar mecanismos como:

client secret;
private key JWT;
mTLS;
otros mecanismos seguros compatibles.

La elección dependerá del contexto.

Los client secrets deberán seguir ADR-027.

42. Public Client Restrictions

Los public clients:

SPA
Mobile

no deben depender de secretos que puedan extraerse del cliente.

Por ello:

Public Client
+
Authorization Code
+
PKCE

será el patrón preferido.

43. Token Endpoint Security

El Token Endpoint deberá protegerse contra:

brute force;
credential stuffing;
token replay;
client abuse;
malformed requests;
excessive retries.

Deberá integrarse con ADR-032.

44. Authorization Endpoint Security

El Authorization Endpoint deberá estar protegido contra:

request tampering;
redirect attacks;
session fixation;
login CSRF;
authorization code injection;
consent manipulation.

Los parámetros deberán validarse estrictamente.

45. Token Revocation

EVOXA deberá soportar revocación de credenciales cuando corresponda.

Puede afectar:

Access Credential
Refresh Token
Client Credential
Service Credential
Session

La semántica exacta dependerá del tipo de token.

46. Logout

OIDC/OAuth logout debe distinguirse de:

Session Revocation

y:

Refresh Token Revocation

Una operación de logout debe integrarse con el lifecycle de Session definido previamente.

Dependiendo del contexto puede implicar:

User Logout
   ↓
Session Revocation
   ↓
Refresh Family Revocation

pero un access token ya emitido puede permanecer criptográficamente válido hasta su expiración si la arquitectura es stateless.

47. Logout and External IdP

Cuando exista federación:

EVOXA Session
     ≠
External IdP Session

Cerrar una sesión local no implica automáticamente cerrar la sesión del IdP externo, salvo que se implemente un flujo de logout federado.

Esta diferencia deberá documentarse claramente.

48. Token Introspection

EVOXA podrá soportar introspection para escenarios donde se requiera validación centralizada de tokens.

Sin embargo, no se deberá convertir obligatoriamente cada API request en:

API
 ↓
Identity Introspection
 ↓
API

si esto genera un bottleneck innecesario.

Se deberá combinar según el caso con:

JWT local validation;
short TTL;
caching;
revocation strategy.
49. JWKS

Para JWT firmados asimétricamente:

Identity Authorization Server
       ↓
JWKS
       ↓
API Services

Los servicios deberán poder validar firmas sin recibir claves privadas.

Las claves públicas deberán tener lifecycle y rotation según ADR-008/027.

50. Discovery

Cuando EVOXA exponga OIDC, podrá proporcionar metadata de discovery.

Conceptualmente:

/.well-known/openid-configuration

Esta información permitirá a clientes conocer:

issuer;
authorization endpoint;
token endpoint;
JWKS URI;
supported scopes;
supported response types;
supported authentication methods.

La lista definitiva dependerá de la implementación del proveedor OIDC.

51. API Versioning

Las APIs relacionadas con OAuth/OIDC deberán integrarse con:

/api/v1

cuando formen parte de los APIs versionados de EVOXA.

Los endpoints de protocolo estándar podrán requerir rutas específicas compatibles con el estándar, pero deberán quedar documentados dentro de la API Architecture.

52. Error Handling

Los errores de OAuth/OIDC deberán seguir los formatos definidos por el protocolo correspondiente cuando sea obligatorio.

Para APIs generales de EVOXA continuará aplicándose el error envelope:

{
  "error": {
    "code": "...",
    "message": "...",
    "correlation_id": "..."
  }
}

No se debe mezclar arbitrariamente el formato OAuth con el error envelope general cuando el protocolo requiera un formato específico.

53. Idempotency

Los endpoints de autorización/token deberán tratar los retries cuidadosamente.

La semántica de OAuth no debe modificarse artificialmente para introducir Idempotency-Key donde el protocolo no lo contempla.

Para operaciones administrativas asociadas a clientes/credentials sí se aplicará ADR-026 cuando corresponda.

54. Audit Events

Deben existir eventos como:

OAUTH_CLIENT_CREATED
OAUTH_CLIENT_UPDATED
OAUTH_CLIENT_DISABLED
OAUTH_AUTHORIZATION_REQUESTED
OAUTH_AUTHORIZATION_GRANTED
OAUTH_AUTHORIZATION_DENIED
OAUTH_TOKEN_ISSUED
OAUTH_TOKEN_REVOKED
OAUTH_REFRESH_ROTATED
OIDC_AUTHENTICATION_COMPLETED
OIDC_AUTHENTICATION_FAILED
OIDC_FEDERATION_USED

Los eventos nunca deben contener:

access token;
refresh token;
client secret;
authorization code;
ID token completo si contiene información innecesaria.
55. Security Events

Se deberán considerar:

OAUTH_INVALID_CLIENT
OAUTH_INVALID_REDIRECT
OAUTH_CODE_REUSE
OAUTH_TOKEN_REUSE
OAUTH_SCOPE_ESCALATION
OAUTH_SUSPICIOUS_CLIENT
OIDC_NONCE_FAILURE
OIDC_STATE_FAILURE
OIDC_FEDERATION_ANOMALY
TOKEN_AUDIENCE_MISMATCH
TOKEN_ISSUER_MISMATCH

El severity será definido por Security Policy.

56. AI Agents and OAuth

AI Agents podrán utilizar OAuth cuando necesiten acceso autorizado a APIs.

Sin embargo:

AI Agent
   ↓
OAuth
   ↓
Scoped Credential

no implica:

AI Agent
   ↓
User's Full Authority

El Agent deberá mantener:

identity;
scopes;
delegation;
tenant;
task context.
57. AI Agent User Delegation

Cuando el Agent actúe en nombre de un usuario:

User
 ↓
AI Agent
 ↓
OAuth / Delegation
 ↓
API

la autorización deberá comprobar:

Agent Permissions
+
Delegated User Permissions
+
Tenant Policy
+
Operation Policy

La delegación no puede ampliar arbitrariamente los privilegios del usuario.

58. Security Policy Integration

OAuth/OIDC será una capa de protocolo.

No será la autoridad final para autorización.

La decisión será:

OAuth/OIDC
      ↓
Authenticated Identity
      ↓
Authorization Context
      ↓
Security Policy Engine
      ↓
RBAC / ABAC / Tenant Isolation
      ↓
Decision

Esto evita que la configuración OAuth se convierta en una segunda plataforma de autorización desconectada.

59. Rate Limiting & Abuse

Se aplicarán controles a:

Authorization Endpoint
Token Endpoint
Revocation Endpoint
Introspection Endpoint
Client Registration
Federation Endpoints

Las dimensiones pueden incluir:

IP
Client
User
Service Account
Tenant
Endpoint
Risk

La estrategia exacta continuará bajo ADR-032.

60. Performance

Los objetivos generales continúan siendo:

API P95 < 300ms
Authentication P95 < 500ms

según los objetivos establecidos en el Blueprint.

El diseño deberá evitar:

introspection obligatoria en cada request;
consultas innecesarias a Identity;
dependencia síncrona excesiva de proveedores externos;
llamadas repetitivas a JWKS.
61. Availability and Fail-Closed

Si el Authorization Server no puede emitir o validar correctamente una credencial:

Cannot Authenticate
→ DENY

Si una API no puede validar una credencial requerida:

Cannot Validate
→ DENY

Nunca:

Identity unavailable
→ Trust token blindly

Las excepciones solo podrán existir para capacidades no críticas y explícitamente diseñadas como degraded functionality.

62. Consequences
Positivas
Utiliza estándares ampliamente reconocidos.
Separa claramente authentication y authorization.
Soporta Web, Mobile y APIs.
Permite Machine-to-Machine.
Facilita federación.
Permite integración con terceros.
Compatible con Service Accounts.
Compatible con Workload Identity.
Prepara OAuth para AI Agents.
Facilita ecosistema externo.
Reduce diseños propietarios innecesarios.
Negativas
OAuth/OIDC aumenta la complejidad de Identity.
Requiere una implementación rigurosa.
Los diferentes tipos de clientes tienen requisitos diferentes.
Token, session y credential lifecycle deben mantenerse coordinados.
Federation añade nuevas superficies de seguridad.
63. Alternatives Considered
A. Custom Authentication Protocol

Rejected.

Generaría complejidad y menor interoperabilidad.

B. OAuth Without OIDC

Rejected for user identity scenarios.

OAuth por sí solo no debe utilizarse como protocolo de autenticación de usuario.

C. OIDC for Everything

Rejected.

OIDC resuelve identidad/autenticación sobre OAuth, pero no reemplaza el modelo general de autorización de APIs y servicios.

D. Long-Lived Access Tokens

Rejected.

Incrementan el impacto de una credencial comprometida.

E. ID Token as API Credential

Rejected.

ID Tokens y Access Tokens tienen propósitos diferentes.

F. OAuth as Authorization Policy Engine

Rejected.

OAuth scopes no reemplazan RBAC/ABAC, tenant isolation ni Security Policy Engine.

64. Non-Negotiable Security Rules
OAuth 2.1 se utilizará para authorization.
OIDC se utilizará para user authentication/identity.
ID Tokens no son API access tokens.
Access Tokens deben estar destinados al recurso correspondiente.
Public clients deben utilizar PKCE.
Redirect URIs deben estar registradas y validadas estrictamente.
Authorization Codes deben ser one-time.
Authorization Codes deben expirar.
state debe utilizarse para proteger el flujo del cliente.
nonce debe validarse en OIDC cuando corresponda.
Refresh Tokens seguirán ADR-025.
Scopes no reemplazan Authorization Policy.
Tenant isolation sigue siendo obligatorio.
External IdP no determina automáticamente permisos internos.
OAuth authentication no puede bypass MFA o Security Policy.
Tokens no deben contener información sensible innecesaria.
Tokens y secrets nunca deben aparecer en logs.
Production client secrets deben utilizar Secret Manager/KMS.
AI Agents no pueden autoelevar sus scopes.
Service Accounts deben mantenerse separadas de Users.
Authentication failures deben fail-closed.
Token audience e issuer deben validarse.
JWT private keys nunca deben distribuirse a APIs consumidoras.
OAuth/OIDC debe integrarse con Audit y Security Events.
OAuth/OIDC no reemplaza el Security Policy Engine de EVOXA.
65. Traceability
Requirement	Source / Relationship
Identity architecture	Identity ESP / ARCHITECTURE-MAP
Authorization model	ADR-IDENTITY-005
JWT cryptography	ADR-IDENTITY-008
Token storage	ADR-IDENTITY-009
Testing	ADR-IDENTITY-010
Sessions	ADR-IDENTITY-008 / IS-008
Refresh Tokens	ADR-IDENTITY-025
Pagination / Idempotency	ADR-IDENTITY-026
Secrets / KMS	ADR-IDENTITY-027
Observability	ADR-IDENTITY-028
Disaster Recovery	ADR-IDENTITY-029/030
Rate Limiting	ADR-IDENTITY-032
Account Lifecycle	ADR-IDENTITY-033
Password Security	ADR-IDENTITY-034
Account Recovery	ADR-IDENTITY-035
Email Verification	ADR-IDENTITY-036
API Tokens / Service Accounts	ADR-IDENTITY-037
Service-to-Service	ADR-IDENTITY-038
OAuth 2.1 / OIDC	ADR-IDENTITY-039

El Blueprint exige mantener trazabilidad desde los Engineering Specifications hacia Implementation Stories, Technical Tasks y Source Code.

66. Dependencies

Este ADR depende principalmente de:

ADR-IDENTITY-005
ADR-IDENTITY-008
ADR-IDENTITY-009
ADR-IDENTITY-010
ADR-IDENTITY-023
ADR-IDENTITY-024
ADR-IDENTITY-025
ADR-IDENTITY-026
ADR-IDENTITY-027
ADR-IDENTITY-028
ADR-IDENTITY-032
ADR-IDENTITY-033
ADR-IDENTITY-035
ADR-IDENTITY-037
ADR-IDENTITY-038

La implementación deberá materializarse posteriormente mediante:

IS-IDENTITY-039
Technical Tasks
API Contracts
Security Specification
Persistence Specification
Event Specification
Testing Specification
67. Pending Decisions

Quedan deliberadamente abiertos:

OAuth
Authorization Server implementation;
exact OAuth 2.1 grant support;
access token format;
opaque vs JWT;
token TTL;
scope catalog;
consent model;
revocation model;
introspection.
OIDC
supported OIDC profiles;
claims catalog;
userinfo endpoint;
ID Token claims;
logout strategy;
federation support.
Clients
client registration model;
public/confidential client catalog;
client authentication methods;
dynamic client registration;
client secret lifecycle.
Federation
external IdP catalog;
trust model;
account linking;
provisioning/deprovisioning;
claim mapping.
M2M
Client Credentials;
private_key_jwt;
mTLS;
workload identity integration;
token exchange/delegation.
AI
AI Agent OAuth client model;
delegated authorization;
task-scoped credentials;
token lifetime;
tool authorization.

Estas decisiones deberán cerrarse mediante las siguientes ADRs/especificaciones antes de implementar el protocolo definitivamente.

68. Acceptance Criteria

El ADR se considerará correctamente implementado cuando:

 OAuth 2.1 sea utilizado como base de authorization.
 OIDC sea utilizado para user authentication/identity.
 OAuth y OIDC estén claramente separados conceptualmente.
 Authorization Code Flow sea soportado cuando corresponda.
 PKCE sea obligatorio para public clients.
 state sea validado.
 nonce sea validado cuando corresponda.
 Redirect URIs sean estrictamente validadas.
 Authorization Codes sean one-time y de corta duración.
 Access Tokens tengan audience apropiada.
 Issuer sea validado.
 JWT signatures sean verificadas.
 ID Tokens no sean aceptados como API authorization tokens.
 Refresh Tokens sigan ADR-025.
 OAuth scopes no bypassen RBAC/ABAC/Policy.
 Tenant Isolation sea obligatorio.
 M2M sea soportado mediante Service Accounts.
 Workload Identity pueda integrarse con OAuth.
 External IdP federation pueda integrarse sin delegar autorización interna.
 AI Agents tengan identidad y scopes propios.
 Client secrets sean gestionados mediante mecanismos seguros.
 Tokens no aparezcan en logs/traces/events.
 OAuth endpoints tengan abuse protection.
 Audit/Security Events permitan investigar autenticación y autorización.
 Failures sean fail-closed.
 Existan pruebas de protocol security, replay, redirect attacks, token substitution, scope escalation, federation y tenant isolation.
69. Next ADR

Con ADR-IDENTITY-039 queda definida la estrategia de protocolos para la capa de Identity:

                    EVOXA IDENTITY
                          │
          ┌───────────────┴────────────────┐
          │                                │
      HUMAN USERS                    MACHINE IDENTITIES
          │                                │
    OpenID Connect                    OAuth 2.1 / M2M
          │                                │
    Authentication                  Service Authentication
          │                                │
       OAuth 2.1                    Workload Identity
          │                                │
     Access Token                   Short-Lived Token
          │                                │
          └───────────────┬────────────────┘
                          ▼
                EVOXA Authorization
                          │
              RBAC + ABAC + Tenant
              + Ownership + Policy

El siguiente paso lógico es:

ADR-IDENTITY-040 — Identity Federation & External Identity Provider Strategy

para definir cómo EVOXA confiará en proveedores externos de identidad, cómo se realizará el account linking, cómo se mapearán claims externos a Users/Memberships/Roles, cómo se manejará el aprovisionamiento y desprovisionamiento, y cómo se evitará que un IdP externo pueda alterar directamente la autorización interna de EVOXA.
