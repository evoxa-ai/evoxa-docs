ESP-0001 / 08 — Identity Security Policies

Estado: Draft
Dominio: Identity
Código: ESP-0001
Sección: 08 — Security Policies
Parent Specification: ESP-0001 — Identity Engineering Specification

1. Propósito

Este documento define las políticas de seguridad que gobiernan el comportamiento del dominio Identity de EVOXA.

Las políticas de seguridad determinan qué condiciones deben cumplirse para permitir, restringir, bloquear, desafiar o revocar operaciones relacionadas con identidad y acceso.

Identity Security Policies debe actuar como una capa transversal sobre:

Authentication
Authorization
MFA
Sessions
Tokens
Users
Organizations
Memberships
Roles & Permissions
Recovery
Audit
Security Events

La arquitectura de EVOXA establece Zero Trust, Least Privilege, Defense in Depth, Secure by Default, Privacy by Design y Auditability como principios fundamentales de seguridad.

2. Objetivos

Las políticas deben permitir que EVOXA:

Deniegue acceso por defecto.
Controle autenticación y autorización.
Controle sesiones y tokens.
Determine cuándo exigir MFA.
Proteja operaciones sensibles.
Detecte comportamientos anómalos.
Aplique rate limiting.
Controle acceso entre tenants.
Proteja cuentas suspendidas o deshabilitadas.
Controle recuperación de cuentas.
Mantenga trazabilidad completa.
Permita políticas diferentes según organización, usuario, rol o contexto.
Permita evolucionar las políticas sin modificar la lógica de cada dominio.
3. Principios normativos
3.1 Deny by Default

Cuando no existe una política explícita que permita una operación:

DENY

Debe ser el comportamiento predeterminado.

3.2 Fail Closed

Ante:

error de autorización;
política no disponible;
contexto incompleto;
servicio de políticas no disponible;
información de tenant desconocida;

el sistema debe denegar, no permitir.

3.3 Least Privilege

Cada identidad debe tener solamente los permisos necesarios para ejecutar sus responsabilidades.

User
 ↓
Membership
 ↓
Role
 ↓
Permissions
 ↓
Policies
 ↓
Resource
3.4 Zero Trust

EVOXA no debe asumir confianza únicamente porque:

el usuario ya inició sesión;
pertenece a una organización;
está usando un dispositivo conocido;
una petición proviene de una red determinada.

Cada operación debe evaluarse según su contexto.

4. Modelo conceptual

La evaluación de una política puede representarse como:

                    Request
                       │
                       ▼
               Authentication
                       │
                       ▼
              Security Context
                       │
          ┌────────────┼────────────┐
          ▼            ▼            ▼
        User         Tenant       Session
          │            │            │
          └────────────┼────────────┘
                       ▼
                  MFA Status
                       │
                       ▼
                 RBAC / ABAC
                       │
                       ▼
               Security Policies
                       │
                       ▼
             ┌─────────────────┐
             │ ALLOW / DENY /   │
             │ CHALLENGE /      │
             │ REVOKE           │
             └─────────────────┘
5. Security Policy

Una política puede modelarse conceptualmente como:

SecurityPolicy
--------------
id
name
description
scope
status
priority
rules
created_at
updated_at
version
Scope

Puede contemplarse:

PLATFORM
ORGANIZATION
USER
ROLE
RESOURCE
OPERATION

La combinación exacta deberá cerrarse mediante ADR.

6. Policy Evaluation Context

La evaluación debe recibir un contexto explícito.

SecurityPolicyContext
---------------------
user
organization
membership
roles
permissions
session
authentication
mfa
device
ip
request
resource
operation
risk

No todos los campos necesitan estar disponibles para todas las políticas.

El motor debe utilizar únicamente el contexto necesario.

7. Policy Decision

Una política puede producir:

ALLOW
DENY
CHALLENGE
REAUTHENTICATE
REQUIRE_MFA
REVOKE

Conceptualmente:

Policy Evaluation
       │
       ▼
 ┌───────────────┐
 │ Decision      │
 └───────┬───────┘
         │
   ┌─────┼──────────────┐
   ▼     ▼              ▼
 ALLOW  DENY        CHALLENGE

Esto permite que Security Policies no sea únicamente un mecanismo binario de autorización.

8. Tipos de políticas
8.1 Authentication Policies

Controlan condiciones para iniciar una sesión.

Ejemplos:

Password requirements
Account status
MFA requirement
Authentication attempts
Session creation
Risk-based authentication
9. Password Policies

Identity debe permitir definir políticas para contraseñas.

Conceptualmente:

PasswordPolicy
--------------
minimum_length
maximum_length
complexity
history
expiration
reuse_prevention
breach_detection

La arquitectura establece Argon2id como algoritmo recomendado para almacenamiento de contraseñas.

La política de complejidad exacta no debe fijarse todavía sin un ADR.

10. Password History

El sistema puede impedir reutilizar contraseñas recientes.

Ejemplo:

Password 1
Password 2
Password 3
Password 4

Una nueva contraseña no debería poder coincidir con una contraseña prohibida por la política.

Las contraseñas anteriores nunca deben almacenarse en texto plano.

11. Password Reset

Password reset debe considerarse una operación de seguridad.

El flujo conceptual:

Request Reset
      │
      ▼
Identity Verification
      │
      ▼
Reset Challenge
      │
      ▼
New Password
      │
      ▼
Revoke Existing Sessions
      │
      ▼
Audit

La política debe poder determinar qué ocurre con las sesiones existentes después de un cambio o recuperación de contraseña.

12. Account Lockout

Identity debe proteger contra ataques de fuerza bruta.

Conceptualmente:

Failed Attempts
       │
       ▼
Threshold
       │
       ▼
Temporary Lock
       │
       ▼
Recovery / Unlock

El mecanismo puede considerar:

usuario;
IP;
dispositivo;
organización;
challenge.

No se debe utilizar únicamente IP como mecanismo de protección.

13. Rate Limiting

Las políticas deben poder limitar frecuencia de operaciones.

Ejemplos:

Login
MFA verification
Password reset
MFA enrollment
Token refresh
API requests
Recovery

El rate limiting debe ser configurable.

14. MFA Policies

Las políticas de MFA definidas en ESP-0001/07 deben poder expresarse desde Security Policies.

Ejemplo:

IF role == ADMIN
THEN REQUIRE_MFA

Otro:

IF operation == CHANGE_SECURITY_SETTINGS
THEN REQUIRE_MFA

Otro:

IF authentication_risk == HIGH
THEN REQUIRE_MFA
15. Step-Up Authentication

Security Policies debe poder exigir una nueva autenticación.

Normal Session
      │
      ▼
Sensitive Operation
      │
      ▼
Policy Evaluation
      │
      ▼
REQUIRE_MFA
      │
      ▼
MFA Challenge
      │
      ▼
Operation

Esto conecta directamente:

Security Policies
        ↓
MFA
        ↓
Sessions
16. Session Security Policies

Las políticas deben poder controlar:

duración máxima;
expiración;
inactividad;
sesiones concurrentes;
revocación;
sesiones de alto riesgo;
logout global;
reautenticación.

Los valores concretos de expiración y concurrencia siguen siendo decisiones que deben resolverse mediante ADR.

17. Token Security Policies

Las políticas deben gobernar:

Access Token
Refresh Token
API Token
Service Account Token

Ejemplos:

Token Lifetime
Refresh Lifetime
Rotation
Revocation
Reuse Detection
Issuer
Audience
Scopes

El diseño de sesiones y tokens ya establecido contempla rotación y detección de reutilización de refresh tokens como controles recomendados.

18. Token Reuse Detection

Un refresh token que ya fue utilizado no debería poder reutilizarse.

Conceptualmente:

Refresh Token A
      │
      ▼
Used
      │
      ▼
Refresh Token B

Si aparece nuevamente:

Refresh Token A
      │
      ▼
REUSE DETECTED
      │
      ▼
Security Event
      │
      ▼
Revoke Token Family

La política exacta de revocación deberá definirse mediante ADR.

19. Account Lifecycle Policies

Security Policies debe reaccionar al estado del usuario.

PENDING
ACTIVE
SUSPENDED
DISABLED

Por ejemplo:

Estado	Login
PENDING	Restringido
ACTIVE	Permitido
SUSPENDED	Denegado
DISABLED	Denegado

Cuando una cuenta pasa a:

SUSPENDED

la política puede exigir:

Revoke Sessions
Revoke Tokens
Invalidate MFA Challenges

La decisión exacta debe quedar documentada.

20. Organization Security Policies

Las organizaciones pueden necesitar políticas propias.

Ejemplos:

Organization A
 ├── MFA required
 ├── Strong password
 ├── Session 8h
 └── Max 3 devices

Organization B
 ├── MFA optional
 ├── Session 12h
 └── Max 5 devices

Esto permite que EVOXA soporte diferentes niveles de seguridad sin duplicar la implementación.

21. Platform Security Policies

Algunas políticas deben estar por encima de cualquier organización.

Ejemplo:

Platform Admin
      │
      ▼
MFA REQUIRED

Una organización no debería poder desactivar una política de seguridad global obligatoria.

22. Policy Precedence

Cuando varias políticas aplican simultáneamente debe existir una precedencia definida.

Propuesta:

Regulatory
    ↓
Platform Security
    ↓
Organization Security
    ↓
Role Security
    ↓
User Security
    ↓
Contextual Policy

Y ante conflicto:

DENY > ALLOW

Esto es consistente con el principio de seguridad de EVOXA donde las restricciones regulatorias y de seguridad prevalecen sobre reglas de negocio o preferencias.

23. Tenant Isolation

Security Policies debe impedir accesos entre tenants.

Organization A
      │
      ├── User A
      └── Resources A

Organization B
      │
      ├── User B
      └── Resources B

Una petición:

User A → Resource B

debe producir:

DENY

salvo que exista una política explícita de acceso cross-tenant.

24. Cross-Tenant Access

El acceso cross-tenant debe ser excepcional.

Si se permite, debe existir:

Explicit Permission
+
Explicit Policy
+
Audit
+
Business Authorization

No debe derivarse simplemente de tener un rol administrativo.

25. Device Policies

Security Policies puede considerar el dispositivo.

Ejemplos:

Known Device
New Device
Trusted Device
Revoked Device

Un nuevo dispositivo podría provocar:

REQUIRE_MFA

mientras que un dispositivo confiable podría no requerir MFA según la política.

26. IP / Network Policies

La plataforma puede soportar políticas basadas en:

IP;
región;
red corporativa;
VPN;
contexto de acceso.

Ejemplo:

IF admin_access
AND unknown_network
THEN REQUIRE_MFA

Estas capacidades deben tratarse como controles adicionales, nunca como sustituto de autenticación fuerte.

27. Risk-Based Policies

Security Policies puede consumir señales de riesgo.

Ejemplo:

Risk
 ├── New device
 ├── Unusual location
 ├── Multiple failed logins
 ├── Token anomaly
 └── Suspicious behavior

Resultado:

LOW
MEDIUM
HIGH

Y una política podría responder:

LOW    → ALLOW
MEDIUM → REQUIRE_MFA
HIGH   → DENY / REAUTHENTICATE

El Risk Engine completo pertenece a una evolución posterior y no debe asumirse como requisito del MVP de Identity.

28. Administrative Security Policies

Las operaciones administrativas deben tener controles reforzados.

Ejemplos:

Change Role
Grant Permission
Disable User
Reset MFA
Change Organization Security
Delete Security Configuration

Podrían requerir:

Admin Permission
+
MFA
+
Audit
29. Sensitive Operations

Security Policies debe mantener una clasificación de operaciones sensibles.

Conceptualmente:

LOW
NORMAL
SENSITIVE
CRITICAL

Ejemplo:

View Profile         → NORMAL
Change Password      → SENSITIVE
Change MFA           → CRITICAL
Grant Admin Role     → CRITICAL

Esto permitirá reutilizar la misma lógica de seguridad en diferentes módulos.

30. Authorization Boundary

Security Policies no debe apropiarse de las reglas de negocio de todos los dominios.

La separación será:

Identity
 └── Who is the user?
 └── What tenant?
 └── What roles?
 └── What permissions?
 └── What security conditions?

Business Domain
 └── Is this action valid for this resource?
 └── Does the business rule allow it?
 └── Does ownership allow it?

Esto mantiene la responsabilidad de dominio definida en la arquitectura.

31. Policy Engine

La arquitectura puede utilizar un componente conceptual:

Security Policy Engine

Responsabilidades:

Load Policy
Evaluate Policy
Resolve Context
Apply Precedence
Return Decision
Audit Decision

No se debe acoplar todavía a un producto tecnológico específico.

32. Policy Versioning

Las políticas deben ser versionables.

Ejemplo:

Policy: password-security
Version: 1
Status: ACTIVE

Posteriormente:

Version: 2
Status: DRAFT

Luego:

Version 1 → RETIRED
Version 2 → ACTIVE

Esto permite evolución sin perder trazabilidad histórica.

33. Policy Lifecycle
DRAFT
  ↓
ACTIVE
  ↓
SUSPENDED
  ↓
RETIRED

Una política retirada no debe aplicarse a nuevas decisiones.

Las decisiones históricas deben conservar referencia a la versión aplicada cuando sea necesario para auditoría.

34. Security Events

Security Policies debe producir eventos relevantes.

Propuestos:

SecurityPolicyEvaluated
SecurityPolicyDenied
SecurityPolicyTriggered
SecurityPolicyChanged
SecurityPolicyActivated
SecurityPolicyDeactivated
AuthenticationBlocked
AccountLocked
AccountUnlocked
SuspiciousAuthenticationDetected
TokenReuseDetected
SecurityChallengeRequired

No todos estos eventos necesitan ser eventos de dominio públicos; algunos pueden ser eventos internos de seguridad.

35. Audit

Cada decisión crítica debería poder responder:

Who?
What?
When?
Where?
Why?
Which policy?
Which version?
What decision?

Ejemplo:

Actor:
user-123

Operation:
change_mfa

Policy:
mfa-security

Policy Version:
3

Decision:
REQUIRE_MFA

Timestamp:
...

Correlation ID:
...

Esto proporciona trazabilidad y facilita investigación de incidentes.

36. API conceptual

Las APIs administrativas de políticas podrían evolucionar hacia:

GET    /api/v1/identity/security-policies
POST   /api/v1/identity/security-policies
GET    /api/v1/identity/security-policies/{policy_id}
PATCH  /api/v1/identity/security-policies/{policy_id}
DELETE /api/v1/identity/security-policies/{policy_id}

Evaluación:

POST /api/v1/identity/security-policies/evaluate

Sin embargo, no recomiendo exponer directamente un endpoint genérico de evaluación de políticas al cliente final.

La evaluación debería producirse normalmente dentro del flujo de autenticación/autorización.

37. Configuración

Las políticas deberían poder configurarse sin modificar código.

Conceptualmente:

Security Configuration
        │
        ├── Authentication
        ├── Password
        ├── MFA
        ├── Sessions
        ├── Tokens
        ├── Recovery
        ├── Rate Limits
        └── Administration

Esto es especialmente importante para organizaciones Enterprise.

38. Security Configuration Hierarchy

Propuesta:

Platform Defaults
       ↓
Organization Policy
       ↓
Role Policy
       ↓
User Policy
       ↓
Request Context

Pero:

Security Minimums

definidos por la plataforma nunca deben poder ser debilitados por una organización.

39. Observabilidad

Security Policies debe generar métricas como:

security_policy_evaluations_total
security_policy_denials_total
security_policy_challenges_total
security_policy_failures_total
account_lockouts_total
token_reuse_detected_total
mfa_policy_triggers_total

También debe permitir análisis por:

organization
policy
operation
decision
risk

sin exponer secretos.

40. Performance

Security Policy evaluation debe ser suficientemente rápida para no convertirse en cuello de botella.

Debe respetar los objetivos de rendimiento generales de Identity y APIs:

Authentication P95 < 500 ms.
APIs generales P95 < 300 ms.

Las evaluaciones frecuentes podrían utilizar caching, pero cualquier cache de políticas debe tener una estrategia explícita de invalidación.

41. Seguridad del Policy Engine

El propio motor de políticas debe estar protegido.

Debe:

autenticar callers;
autorizar modificaciones;
validar configuración;
registrar cambios;
evitar ejecución arbitraria;
evitar bypass;
aplicar fail-closed;
versionar políticas.

Una política corrupta o inválida no debe transformarse automáticamente en:

ALLOW
42. Testing

Security Policies requiere pruebas específicas.

Unit Tests
Precedence.
Allow.
Deny.
Challenge.
Policy lifecycle.
Integration Tests
Authentication + policy.
MFA + policy.
Session + policy.
Organization + policy.
Role + policy.
Security Tests
Cross-tenant access.
Policy bypass.
Privilege escalation.
Token reuse.
Rate limit bypass.
Policy tampering.
E2E
Login
 ↓
MFA
 ↓
Session
 ↓
Sensitive Operation
 ↓
Policy
 ↓
Step-Up
 ↓
Operation

El Blueprint exige pruebas de seguridad, autorización, API, integración y E2E como parte de los quality gates.

43. ADRs necesarios

Este capítulo introduce varias decisiones arquitectónicas que todavía no deberíamos fijar como implementación definitiva.

ADR-IDENTITY-013

Security Policy Engine

¿Implementación interna o utilización de un policy engine especializado?

ADR-IDENTITY-014

Policy Scope

Determinar:

Platform
Organization
Role
User
Resource
Operation
ADR-IDENTITY-015

Policy Precedence

Definir formalmente:

Platform
Organization
Role
User
Context

y comportamiento ante conflictos.

ADR-IDENTITY-016

Policy Decision Model

Definir si se soportará:

ALLOW
DENY
CHALLENGE
REAUTHENTICATE
REQUIRE_MFA
REVOKE
ADR-IDENTITY-017

Security Configuration Override

Determinar qué políticas puede modificar una organización y cuáles son inmutables.

ADR-IDENTITY-018

Risk-Based Authentication

Determinar si el MVP tendrá evaluación de riesgo o será una capacidad futura.

ADR-IDENTITY-019

Policy Caching

Definir:

TTL;
invalidación;
consistencia;
fallback;
comportamiento durante indisponibilidad.
44. Criterios de aceptación

ESP-0001/08 estará listo para implementación cuando:

 Exista modelo definitivo de Security Policy.
 Exista modelo definitivo de Policy Context.
 Exista modelo definitivo de Policy Decision.
 Esté definido el scope.
 Esté definida la precedencia.
 Esté definido deny-by-default.
 Esté definido fail-closed.
 Estén definidas las políticas de autenticación.
 Estén definidas las políticas de contraseña.
 Estén definidas las políticas MFA.
 Estén definidas las políticas de sesión.
 Estén definidas las políticas de tokens.
 Estén definidas las políticas de recuperación.
 Estén definidas las políticas administrativas.
 Esté definida la estrategia de tenant isolation.
 Esté definida la estrategia de rate limiting.
 Esté definido el modelo de auditoría.
 Estén definidos los eventos.
 Estén definidos los ADR.
 Existan API Contracts.
 Existan pruebas unitarias.
 Existan pruebas de integración.
 Existan pruebas de seguridad.
 Existan pruebas E2E.
45. Trazabilidad
BP-0002
Reference Architecture
        │
        ▼
Security Architecture
        │
        ├── Zero Trust
        ├── Least Privilege
        ├── RBAC + ABAC
        ├── Tenant Isolation
        ├── Auditability
        └── AI Security
                │
                ▼
       ESP-0001 Identity
                │
        ┌───────┴────────┐
        ▼                ▼
ESP-0001/03        ESP-0001/07
Authentication          MFA
        │                │
        └───────┬────────┘
                ▼
        ESP-0001/08
      Security Policies
                │
        ┌───────┼────────┐
        ▼       ▼        ▼
     Sessions  Tokens   Authorization
        │       │        │
        └───────┼────────┘
                ▼
          API Contracts
                │
                ▼
       Implementation Stories
                │
                ▼
              Code
                │
                ▼
             Testing

La separación es coherente con el mapa oficial de arquitectura de EVOXA, que coloca Identity como dominio y mantiene la trazabilidad desde ESP hasta implementación y testing.

46. Estado actual de ESP-0001
Sección	Estado
01 — Identity Domain Model	Draft
02 — Identity Architecture	Draft
03 — Authentication & Authorization	Draft
04 — User & Organization Management	Draft
05 — Roles & Permissions	Draft
06 — Sessions & Token Management	Draft
07 — Multi-Factor Authentication	Draft
08 — Security Policies	Draft
Siguiente sección recomendada

ESP-0001 / 09 — Identity Audit & Compliance

Ahí deberíamos definir formalmente:

Audit Logs
Security Audit
Authentication Audit
Authorization Audit
MFA Audit
Administrative Audit
Compliance Records
Retention
Immutability
Audit Events
Audit Query
Traceability
Privacy
Data Access
Regulatory Evidence

Esto cerraría la parte de seguridad y trazabilidad de Identity antes de pasar a los contratos API y a la implementación.
