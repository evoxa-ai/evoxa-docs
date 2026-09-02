ADR-IDENTITY-009 — Identity Client Token Storage

Status: Proposed
Version: 1.0.0
Domain: Identity
Decision Type: Security / Architecture
Scope: Browser, Web, Mobile, API Clients, Access Tokens, Refresh Tokens, Session Security
Related: ESP-0001 — Identity
Depends on:

ADR-IDENTITY-001 — Identity Architecture & Technology Decisions
ADR-IDENTITY-002 — JWT & Token Lifecycle
ADR-IDENTITY-003 — Refresh Token Security & Session Management
ADR-IDENTITY-008 — Identity Cryptographic Key Management
1. Context

EVOXA Identity utilizará:

Access Token
Refresh Token
Session

como parte del modelo de autenticación definido previamente.

La forma en que estos tokens se almacenan en los distintos clientes es una decisión crítica porque una exposición del token puede permitir:

suplantación de identidad;
acceso no autorizado;
reutilización de sesiones;
robo de sesión;
acceso entre aplicaciones;
compromiso de cuentas.

El Blueprint establece seguridad por diseño, Zero Trust, least privilege, privacidad y defensa en profundidad, pero la arquitectura de referencia no prescribe un mecanismo único de almacenamiento de tokens para todos los tipos de cliente.

Por ello, esta ADR define una estrategia por plataforma.

2. Problem Statement

EVOXA tendrá potencialmente múltiples clientes:

                 EVOXA Identity
                       │
          ┌────────────┼────────────┐
          ▼            ▼            ▼
        Web          Mobile       External
       Browser         App          API

Cada uno tiene diferentes capacidades de almacenamiento y amenazas.

Por ejemplo:

Browser
   → XSS
   → CSRF
   → Browser extensions

Mientras:

Mobile
   → Device compromise
   → Backup exposure
   → Local storage

Por lo tanto:

No se utilizará una única estrategia de almacenamiento para todas las plataformas.

3. Decision

Se adopta una estrategia diferenciada:

Cliente	Access Token	Refresh Token
Web Browser	Memoria / mecanismo de sesión seguro	HttpOnly + Secure Cookie
Mobile	Memoria cuando sea posible	Secure OS Storage
Server-to-Server	Secure runtime storage	Según credencial/service-account policy
SDKs externos	Nunca persistentemente en texto plano	Según plataforma

La estrategia definitiva deberá mantener como principio:

Los refresh tokens requieren mayor protección que los access tokens.

4. Token Hierarchy

La arquitectura será:

Authentication
      │
      ▼
Session
      │
      ├───────────────┐
      ▼               ▼
Access Token     Refresh Token
      │               │
 Short-lived       Long-lived
      │               │
Memory preferred   Strong protection
      │               │
      ▼               ▼
API Requests       Token Rotation

Esto es consistente con ADR-IDENTITY-002 y ADR-IDENTITY-003.

5. Browser Strategy

Para aplicaciones Web, se recomienda:

Access Token
     ↓
In Memory

y:

Refresh Token
     ↓
HttpOnly
Secure
SameSite Cookie

Conceptualmente:

Browser
│
├── Memory
│    └── Access Token
│
└── Cookie
     └── Refresh Token
6. Why HttpOnly

El refresh token no debería estar disponible para JavaScript de la aplicación.

Por ello:

HttpOnly = true

impide que código JavaScript normal pueda leer directamente la cookie.

Esto reduce el impacto de determinados escenarios de XSS sobre el refresh token.

No elimina el riesgo de XSS completamente, porque una aplicación comprometida podría realizar solicitudes desde el contexto del usuario.

7. Secure Cookie

El refresh token deberá utilizar:

Secure = true

en entornos donde corresponda.

Esto garantiza que la cookie sea enviada únicamente mediante HTTPS.

La arquitectura de EVOXA requiere comunicación segura y contempla TLS 1.3.

8. SameSite

La cookie de refresh deberá utilizar una política SameSite apropiada al modelo de despliegue.

Las opciones conceptuales son:

Strict
Lax
None

La selección definitiva dependerá de:

arquitectura frontend/backend;
dominios;
subdominios;
SSO;
integraciones;
necesidad de cross-site requests.

Por tanto:

Este ADR no fija universalmente SameSite=Strict.

9. CSRF Protection

Cuando los tokens se transporten mediante cookies, las solicitudes de operaciones protegidas deben considerar protección contra CSRF.

Conceptualmente:

Cookie Authentication
        +
CSRF Protection

La implementación exacta podrá utilizar:

SameSite;
CSRF token;
Origin validation;
combinación de mecanismos.

La estrategia final dependerá de la arquitectura Web.

10. Access Token in Browser Memory

Se recomienda mantener el access token en memoria:

JavaScript Runtime
      ↓
Access Token

en lugar de utilizar persistent storage como:

localStorage
sessionStorage

como mecanismo principal.

Ventaja:

Page Reload
    ↓
Token disappears
    ↓
Refresh Session
    ↓
New Access Token

Esto reduce la persistencia local del access token.

11. localStorage

No se recomienda almacenar refresh tokens en:

localStorage

ni como estrategia principal almacenar access tokens de larga duración allí.

Motivo arquitectónico:

JavaScript
     ↓
localStorage
     ↓
Token

El token queda directamente accesible para código JavaScript ejecutándose en el origen.

Por tanto:

localStorage no será el mecanismo estándar de almacenamiento de refresh tokens para EVOXA Web.

12. Mobile Strategy

En aplicaciones móviles, el almacenamiento persistente de tokens debe utilizar almacenamiento seguro proporcionado por el sistema operativo.

Conceptualmente:

EVOXA Mobile
      │
      ▼
Secure Storage
      │
 ┌────┴────┐
 ▼         ▼
iOS       Android
Secure    Keystore /
Enclave   secure storage

La aplicación no debe almacenar refresh tokens directamente en:

SQLite plaintext
SharedPreferences plaintext
UserDefaults plaintext
plain files
13. Mobile Access Token

El access token debería mantenerse en memoria cuando sea posible.

App Runtime
     ↓
Access Token

Cuando la aplicación se reinicia:

App Restart
     ↓
Access Token unavailable
     ↓
Refresh Token
     ↓
New Access Token

Esto mantiene el access token corto y reduce su persistencia.

14. Mobile Refresh Token

El refresh token podrá persistirse en secure storage.

Refresh Token
      ↓
OS Secure Storage
      ↓
Encrypted / Protected

Nunca:

Refresh Token
      ↓
Plain SQLite
15. Mobile Device Binding

El diseño podrá incorporar posteriormente mecanismos de device binding.

Conceptualmente:

Refresh Token
      +
Device Identity
      ↓
Session

Esto puede ayudar a detectar uso anómalo.

Sin embargo:

Device binding obligatorio no queda decidido por este ADR.

Se deberá evaluar como parte de una futura política de seguridad.

16. Server-to-Server Clients

Los clientes backend-to-backend no deberán utilizar el mismo patrón que un navegador.

Ejemplo:

EVOXA Service A
       │
       ▼
Identity

Podrán utilizar:

Service Account
OAuth2 Client Credentials
API Key

según el contrato correspondiente.

Las credenciales deben almacenarse en un Secret Manager/KMS.

17. SDKs

Los SDKs de EVOXA deberán abstraer la gestión de tokens.

Un SDK no debería obligar al desarrollador a implementar manualmente:

refresh
rotation
logout
retry
token expiration

cuando el SDK pueda manejarlo de forma segura.

Conceptualmente:

Application
     ↓
EVOXA SDK
     ↓
Token Manager
     ↓
Secure Storage
18. Token Manager

Los clientes podrán implementar un:

TokenManager

responsable de:

obtener access token;
detectar expiración;
refrescar;
almacenar refresh token de forma segura;
eliminar credenciales;
manejar logout;
manejar refresh failures.

Ejemplo:

TokenManager
├── getAccessToken()
├── refresh()
├── logout()
└── clear()
19. Refresh Token Rotation

El almacenamiento del refresh token debe funcionar con la estrategia definida en ADR-IDENTITY-003:

Refresh Token A
      ↓
Refresh
      ↓
Refresh Token B
      ↓
Refresh
      ↓
Refresh Token C

El cliente debe reemplazar inmediatamente:

A → B

y no continuar utilizando:

A
20. Concurrent Refresh

Los clientes deben evitar múltiples refresh simultáneos.

Problema:

Request A → expired
Request B → expired
Request C → expired

A ──┐
B ──┼──> Refresh Token A
C ──┘

Con rotation estricta, esto puede provocar:

TokenReuseDetected

Por tanto, el cliente deberá utilizar un mecanismo de sincronización:

Request A
   │
   ▼
Refresh Lock
   │
   ▼
Refresh
   │
   ▼
Token B
   │
   ├── Request B reuses B
   └── Request C reuses B
21. Access Token Expiration

Cuando expire el access token:

API → 401
   ↓
TokenManager
   ↓
Refresh
   ↓
New Access Token
   ↓
Retry original request

El retry debe estar limitado para evitar:

Infinite Refresh Loop
22. Refresh Failure

Si el refresh falla:

Refresh
   ↓
401 / invalid refresh
   ↓
Clear local credentials
   ↓
Session considered invalid
   ↓
Login required

No se debe intentar indefinidamente.

23. Token Storage Isolation

Los tokens deben estar aislados por:

Environment
Application
Tenant Context
User Session

Por ejemplo:

EVOXA Dev
   ≠
EVOXA Production

y:

App A
   ≠
App B

cuando el modelo de seguridad lo requiera.

24. Browser Multi-Tab

Las aplicaciones Web pueden tener múltiples tabs:

Tab A
Tab B
Tab C

El sistema debe considerar sincronización de:

login
logout
session expiration
token refresh

El mecanismo concreto puede utilizar capacidades del navegador, pero:

Una pestaña no debe provocar accidentalmente múltiples refreshes concurrentes del mismo refresh token.

25. Logout
Web

Logout debe:

Client
  ↓
POST /auth/logout
  ↓
Server revokes session
  ↓
Refresh cookie cleared
  ↓
Access token removed from memory
Mobile
Client
  ↓
POST /auth/logout
  ↓
Server revokes session
  ↓
Secure Storage cleared
  ↓
Memory cleared
26. Logout All

Cuando el usuario selecciona:

Logout All

el servidor revocará:

All Sessions
All Refresh Token Families

según ADR-IDENTITY-003.

Los clientes deberán eliminar sus credenciales locales.

27. Security Incident

Ante:

TokenReuseDetected
Security Incident
User Suspension

el cliente debe asumir que su sesión puede haber sido invalidada.

Ejemplo:

API
 ↓
401
 ↓
Refresh
 ↓
REUSE_DETECTED
 ↓
Clear Credentials
 ↓
Require Login
28. Token Exposure

Si un refresh token es expuesto:

Attacker
   ↓
Refresh Token
   ↓
Identity

la estrategia de rotation + reuse detection definida en ADR-IDENTITY-003 debe permitir detectar su reutilización.

Por eso:

Secure Storage
+
Rotation
+
Reuse Detection
+
Session Revocation

funcionan como defensa en profundidad.

29. Browser XSS

El almacenamiento seguro de tokens no sustituye la seguridad de la aplicación.

EVOXA deberá aplicar:

Content Security Policy;
output encoding;
input validation;
dependency security;
secure headers;
protección contra XSS;
revisión de third-party scripts.

La implementación exacta queda fuera del alcance de esta ADR.

30. Browser Extensions

Las extensiones del navegador pueden representar un riesgo adicional.

La arquitectura debe asumir que:

Browser
   ≠
Trusted Execution Environment

Por ello, minimizar la persistencia y exposición de tokens sigue siendo importante.

31. API Security

Cada request autenticado deberá utilizar:

Authorization: Bearer <access_token>

cuando el mecanismo seleccionado sea Bearer JWT.

El access token debe ser enviado únicamente a endpoints confiables.

Nunca:

URL query parameter

como:

/api/users?token=...

porque puede terminar en:

logs;
browser history;
analytics;
proxies.
32. Referrer Leakage

Los tokens no deben formar parte de URLs.

Esto reduce riesgos derivados de:

Browser History
Referrer
Proxy Logs
Analytics
Monitoring
33. Logging

Los clientes y servidores deben evitar registrar:

Access Token
Refresh Token
Cookie Value
Authorization Header

Los logs podrán registrar eventos seguros como:

TOKEN_REFRESH_SUCCESS
TOKEN_REFRESH_FAILED
SESSION_EXPIRED
LOGOUT

sin revelar secretos.

34. Error Handling

Los errores de autenticación no deben revelar información sensible.

Evitar:

"Refresh token abc123 belongs to session XYZ"

Preferir:

{
  "error": {
    "code": "AUTHENTICATION_FAILED",
    "message": "Authentication failed",
    "correlation_id": "corr_123"
  }
}
35. Client Storage Matrix

La estrategia completa:

Tipo	Access Token	Refresh Token	Persistencia
Browser	Memory	HttpOnly Secure Cookie	Refresh solamente
Mobile	Memory	OS Secure Storage	Refresh solamente
Backend	Runtime memory	Secret Manager / credential store	Según cliente
SDK	Memory	Platform secure storage	Según plataforma

Esta matriz es la dirección arquitectónica propuesta y deberá concretarse en los SDKs y aplicaciones correspondientes.

36. Security Requirements

Son requisitos no negociables:

Refresh tokens no deben almacenarse en localStorage.
Refresh tokens Web deben utilizar cookies protegidas cuando el modelo Web lo permita.
Cookies de autenticación deben utilizar HTTPS.
Mobile debe utilizar almacenamiento seguro del sistema operativo.
Access tokens deben ser short-lived.
Refresh tokens deben rotarse.
El cliente debe reemplazar el refresh token después de una rotación exitosa.
Debe existir protección contra refresh concurrente.
Tokens nunca deben aparecer en logs.
Tokens nunca deben enviarse mediante URLs.
Logout debe limpiar credenciales locales.
Logout server-side debe revocar la sesión.
Las aplicaciones deben tratar los 401 de manera controlada.
Debe evitarse el refresh loop.
Los ambientes deben mantener credenciales separadas.
Los SDKs deben abstraer la gestión segura de tokens.
37. Alternatives Considered
A. localStorage para todos los tokens
Rechazada

Aumenta la exposición de tokens al JavaScript del navegador.

B. Access + Refresh Tokens en Cookies
No adoptada como estrategia universal

Puede funcionar para aplicaciones Web, pero no es apropiada como mecanismo universal para Mobile y otros tipos de clientes.

C. Access + Refresh Tokens en memoria
Rechazada como estrategia completa

El usuario perdería la sesión al reiniciar aplicaciones móviles y navegadores sin un mecanismo persistente de renovación.

D. Estrategia específica por plataforma
Seleccionada

Permite adaptar el almacenamiento al modelo de seguridad de:

Web
Mobile
Backend
SDK

manteniendo los mismos principios de Identity.

38. Consequences
Positivas
Reduce exposición de refresh tokens.
Adapta seguridad al tipo de cliente.
Mejora protección frente a XSS.
Aprovecha secure storage en Mobile.
Mantiene access tokens de corta duración.
Compatible con token rotation.
Compatible con multi-device sessions.
Facilita SDKs seguros.
Negativas
Mayor complejidad.
Web y Mobile requieren implementaciones diferentes.
Cookie authentication introduce requisitos de CSRF.
Mobile requiere integración con capacidades nativas.
Multi-tab y concurrent refresh requieren coordinación.
Debe existir una estrategia clara de logout y recuperación de sesión.
39. Implementation Direction

Conceptualmente:

apps/
├── web/
│   └── auth/
│       ├── token-manager
│       ├── session-manager
│       └── csrf
│
├── mobile/
│   └── auth/
│       ├── token-manager
│       ├── secure-storage
│       └── session-manager
│
└── sdk/
    └── auth/
        └── token-manager

Backend:

apps/api/app/domains/identity/

└── application/
    └── authentication/
        ├── token_service
        ├── session_service
        └── refresh_service

La estructura es una dirección de implementación derivada, no una estructura prescrita literalmente por el Blueprint.

40. Testing Requirements
Web
HttpOnly cookie;
Secure cookie;
SameSite;
CSRF;
XSS token exposure;
logout;
session expiration;
multi-tab;
concurrent refresh.
Mobile
secure storage;
app restart;
token refresh;
logout;
device compromise scenarios;
concurrent refresh.
API
expired access token;
invalid refresh token;
rotated refresh token;
reused refresh token;
revoked session;
suspended user;
logout-all.
Security
token leakage;
URL token exposure;
logs;
error responses;
cross-environment token use.
41. Traceability
BP-0002 Reference Architecture
        ↓
ESP-0001 Identity
        ↓
ESP-0001 / 03 Authentication & Authorization
        ↓
ESP-0001 / 06 Sessions & Token Management
        ↓
ESP-0001 / 07 MFA
        ↓
ESP-0001 / 08 Security Policies
        ↓
ESP-0001 / 10 API Contracts
        ↓
ESP-0001 / 15 Deployment & Operations
        ↓
ADR-IDENTITY-002
        ↓
ADR-IDENTITY-003
        ↓
ADR-IDENTITY-008
        ↓
ADR-IDENTITY-009

La arquitectura de Identity ya establece sesiones stateful, access tokens JWT de corta duración y refresh tokens stateful con rotación y detección de reutilización; esta ADR determina cómo dichos tokens se manejan desde los clientes.

42. Follow-up ADRs

Se recomienda separar posteriormente:

ADR-IDENTITY-009.1

Web Authentication Cookie Policy

Definir:

Domain;
Path;
SameSite;
CSRF;
cookie lifetime;
subdomain strategy.
ADR-IDENTITY-009.2

Mobile Secure Storage

Definir:

Android Keystore;
iOS Keychain/Secure Enclave;
encryption;
biometric unlock;
device binding.
ADR-IDENTITY-009.3

SDK Token Manager

Definir:

refresh synchronization;
retry;
interceptor;
logout;
storage abstraction.
ADR-IDENTITY-009.4

Browser Session Architecture

Definir:

BFF vs direct API;
cookie architecture;
multi-tab synchronization;
session bootstrap.
43. Final Decision Summary

La estrategia queda:

                         EVOXA IDENTITY
                               │
                       Authentication
                               │
                            Session
                               │
                 ┌─────────────┴─────────────┐
                 │                           │
                 ▼                           ▼
           Access Token                Refresh Token
            Short-lived                  Stateful
                 │                       Rotating
                 │                           │
        ┌────────┴────────┐          ┌───────┴────────┐
        ▼                 ▼          ▼                ▼
      Web              Mobile      Web              Mobile
        │                 │          │                │
     Memory            Memory     HttpOnly          Secure
                                  Cookie            Storage
Decisión principal

EVOXA utilizará una estrategia de almacenamiento de tokens específica por plataforma. En Web, los access tokens se mantendrán preferentemente en memoria y los refresh tokens se protegerán mediante cookies HttpOnly/Secure, con una política SameSite y protección CSRF acorde al modelo de despliegue. En Mobile, los refresh tokens se almacenarán mediante mecanismos seguros proporcionados por el sistema operativo y los access tokens permanecerán preferentemente en memoria. Los clientes backend y SDKs utilizarán mecanismos seguros apropiados a su entorno. Todos los clientes deberán integrarse con la estrategia de refresh-token rotation, reuse detection y session revocation definida por Identity.

Estado: PROPOSED.

Punto importante para EVOXA

Este ADR no cambia la arquitectura definida en ADR-002/003. La mantiene coherente desde el lado cliente:

ADR-002
JWT + Token Lifecycle
        ↓
ADR-003
Refresh Rotation + Sessions
        ↓
ADR-008
Cryptographic Key Management
        ↓
ADR-009
Client Token Storage

Con esto, la cadena de seguridad de tokens de Identity queda bastante completa antes de pasar a las decisiones específicas de implementación.
