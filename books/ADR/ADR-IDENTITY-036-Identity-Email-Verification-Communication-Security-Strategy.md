ADR-IDENTITY-036 — Identity Email Verification & Communication Security Strategy

Status: Proposed
Version: 1.0
Date: 2026-09-03
Domain: Identity
Priority: Critical
Type: Security / Architecture / Identity

1. Context

EVOXA Identity necesita utilizar canales de comunicación para operaciones relacionadas con la identidad, especialmente:

verificación de email;
recuperación de cuenta;
cambio de email;
notificaciones de seguridad;
confirmación de operaciones sensibles;
alertas de autenticación;
comunicación relacionada con MFA y recuperación.

El email puede funcionar como evidencia de control de una dirección, pero no debe considerarse automáticamente equivalente a una identidad completamente verificada.

La estrategia debe integrarse con:

User Account Lifecycle;
Authentication;
Password Security;
Account Recovery;
MFA;
Sessions;
Refresh Tokens;
Security Policy Engine;
Rate Limiting;
Audit & Security Events;
Event / Outbox Architecture;
KMS / Secret Management;
Tenant Isolation.

Este ADR continúa especialmente:

ADR-IDENTITY-023 — MFA Factor Strategy;
ADR-IDENTITY-024 — MFA Challenge TTL, Attempts & Recovery;
ADR-IDENTITY-027 — Runtime Configuration, Secrets & KMS;
ADR-IDENTITY-028 — Observability & Operational Security;
ADR-IDENTITY-032 — Rate Limiting, Abuse Prevention & Brute-Force Protection;
ADR-IDENTITY-034 — Password Security & Credential Lifecycle;
ADR-IDENTITY-035 — Account Recovery & Credential Recovery Strategy.
2. Problem Statement

El email es uno de los principales canales para:

Registration
    ↓
Email Verification
    ↓
Account Activation

y:

Password Recovery
    ↓
Recovery Email

pero también representa un riesgo.

Un atacante podría intentar:

enumerar cuentas;
solicitar miles de emails;
reutilizar verification tokens;
reutilizar recovery links;
interceptar enlaces;
cambiar el email de una cuenta;
utilizar un email no verificado como factor de recuperación;
abusar de proveedores de email;
provocar spam;
aprovechar una cuenta de email comprometida.

Por lo tanto:

Email ownership verification must be treated as a controlled security capability, not merely as a notification feature.

3. Decision

EVOXA adoptará una arquitectura de comunicación de Identity basada en:

Email Ownership Verification
Purpose-Bound Verification Tokens
Short-Lived One-Time Tokens
Anti-Enumeration
Rate Limiting & Abuse Prevention
Security-Aware Email Change
Security Notifications
Policy-Controlled Recovery
Secure Provider Integration
Auditability
Tenant Isolation
No Secrets in Logs or Events

El sistema distinguirá claramente:

Communication
      ≠
Authentication
      ≠
Identity Verification
      ≠
Account Recovery

aunque puedan utilizar el mismo canal físico.

4. Email Verification

El sistema deberá permitir demostrar que el usuario tiene control sobre una dirección de email.

Flujo conceptual:

User Registration
       ↓
Email Address
       ↓
Verification Request
       ↓
Verification Token
       ↓
Email Delivery
       ↓
User Opens Link
       ↓
Token Validation
       ↓
Email Ownership Verified

El resultado será una propiedad del email, no necesariamente una validación completa de identidad civil.

5. Email Verification State

La dirección de email deberá mantener un estado explícito.

Conceptualmente:

UNVERIFIED
     ↓
VERIFICATION_PENDING
     ↓
VERIFIED

También podrán existir estados derivados:

REVOKED
CHANGED

cuando corresponda.

Una dirección previamente verificada no debe permanecer automáticamente como verificada después de un cambio hacia una nueva dirección.

6. Verification Token

Los tokens de verificación deberán seguir principios similares a los definidos para Recovery Tokens.

Conceptualmente:

EmailVerificationToken
├── id
├── user_id
├── email
├── purpose
├── token_hash
├── status
├── created_at
├── expires_at
├── consumed_at
└── revoked_at

El token deberá ser:

criptográficamente seguro;
impredecible;
de un solo uso;
de duración limitada;
asociado a un propósito;
asociado al usuario;
invalidable;
protegido contra replay.
7. Purpose Binding

Un token debe estar ligado a su propósito.

Ejemplo:

EMAIL_VERIFICATION

no puede utilizarse como:

PASSWORD_RESET

ni:

EMAIL_CHANGE_CONFIRMATION

ni:

MFA_RECOVERY

La validación deberá comprobar explícitamente:

Token
+
Purpose
+
User
+
Email
+
Expiration
+
Status
8. One-Time Consumption

Una vez utilizado correctamente:

ACTIVE
   ↓
CONSUMED

el token no podrá volver a utilizarse.

Ejemplo:

Request A → Verify → SUCCESS
Request B → Same Token → REJECT

La operación deberá ser concurrency-safe.

9. Token Expiration

Los tokens de verificación deben tener una duración limitada.

Conceptualmente:

created_at <= now < expires_at

Si:

now >= expires_at

el token deberá considerarse inválido.

El usuario podrá solicitar un nuevo token de acuerdo con las políticas de rate limiting.

El TTL exacto queda pendiente de definición.

10. Resend Verification

El sistema deberá permitir solicitar nuevamente un email de verificación.

Sin embargo:

Resend

no debe generar una vía de abuso.

Debe estar protegido por límites basados, según corresponda, en:

IP
User
Email
Device
Tenant
Endpoint
Time Window

El sistema deberá evitar generar grandes cantidades de mensajes para la misma dirección.

11. Email Enumeration Protection

Los endpoints de Identity relacionados con email deberán evitar revelar información sobre cuentas.

Por ejemplo:

POST /auth/email/verification/request

no debería responder de forma diferente según:

existing user

o:

non-existing user

cuando eso permita enumerar usuarios.

La respuesta pública debe ser genérica.

Los detalles internos podrán registrarse mediante Audit/Security Events bajo los controles apropiados.

12. Email Change

Cambiar el email principal de una cuenta es una operación sensible.

El flujo recomendado será:

Authenticated User
        ↓
Request Email Change
        ↓
Security Policy
        ↓
Current Authentication / Step-Up
        ↓
New Email Verification
        ↓
Confirm Ownership
        ↓
Apply Email Change
        ↓
Security Notification

La nueva dirección no debe convertirse en VERIFIED simplemente porque fue introducida en el perfil.

Debe demostrarse control sobre ella.

13. Existing Email vs New Email

Cuando sea posible y de acuerdo con la política, un cambio sensible de email podrá requerir evidencia tanto del mecanismo actual como del nuevo.

Conceptualmente:

Current Account Control
          +
New Email Ownership
          ↓
Email Change

Esto reduce el riesgo de:

Attacker gains temporary account access
        ↓
Changes email
        ↓
Uses new email for recovery
        ↓
Permanent Account Takeover

Los requisitos exactos dependerán del Security Policy Engine.

14. Email Change Notification

Un cambio de email deberá generar una notificación de seguridad al canal anterior cuando resulte posible y esté permitido por la política.

Conceptualmente:

Old Email
   ↓
"Your account email was changed"

Esto permite detectar rápidamente un takeover.

La notificación no deberá contener:

passwords;
tokens;
MFA secrets;
recovery codes;
refresh tokens.
15. Recovery Integration

El email verificado puede utilizarse como mecanismo de recuperación únicamente cuando la política lo permita.

Por ejemplo:

Verified Email
      ↓
Password Recovery
      ↓
Recovery Token

Pero:

Unverified Email
      ↓
Password Recovery

no debe recibir automáticamente el mismo nivel de confianza.

Esto conecta directamente con ADR-IDENTITY-035.

16. Email as Evidence

El sistema debe diferenciar:

Email Address Exists

de:

User Controls Email

y de:

User Identity Is Verified

Por lo tanto:

VERIFIED_EMAIL

significa conceptualmente:

EVOXA obtuvo evidencia de que el usuario pudo controlar ese canal de email durante el proceso de verificación.

No significa necesariamente:

identidad legal verificada;
persona real validada;
organización validada;
ausencia de compromiso del buzón.
17. Security Policy Integration

El Security Policy Engine determinará cuándo una dirección de email verificada es suficiente como evidencia.

El contexto puede incluir:

User
Organization
Membership
Account State
Email State
Authentication Method
MFA State
Device
Session
Risk
Operation
Recovery Purpose
Policy

La decisión podrá ser:

ALLOW
DENY
CHALLENGE
REAUTHENTICATE
REQUIRE_MFA
18. Sensitive Operations

No se deberá utilizar simplemente:

Verified Email

como autorización universal para operaciones críticas.

Ejemplos de operaciones que pueden requerir evidencia adicional:

disable MFA;
change security factors;
account ownership changes;
organization ownership changes;
administrative recovery;
high-risk credential recovery;
privileged role changes.

La política deberá determinar el nivel de assurance necesario.

19. Email Security Notifications

Identity deberá poder emitir notificaciones para eventos relevantes.

Ejemplos:

LOGIN_FROM_NEW_CONTEXT
PASSWORD_CHANGED
PASSWORD_RESET
EMAIL_CHANGED
MFA_ENABLED
MFA_DISABLED
MFA_FACTOR_ADDED
MFA_FACTOR_REMOVED
SESSION_REVOKED
ACCOUNT_SUSPENDED
RECOVERY_COMPLETED
SECURITY_EVENT

No todas las notificaciones serán necesariamente obligatorias en la primera implementación; el catálogo definitivo deberá establecerse posteriormente.

20. Notification vs Security Event

Una notificación enviada al usuario no reemplaza un Security Event.

Por ejemplo:

Password Changed
       │
       ├── Audit Event
       ├── Security Event
       └── User Notification

Cada uno tiene un propósito diferente:

Audit Event

Registro histórico y trazabilidad.

Security Event

Detección, investigación y respuesta.

User Notification

Comunicación con el usuario.

21. Communication Provider

Los proveedores externos de email deberán estar aislados mediante adapters de Infrastructure.

Conceptualmente:

Identity
   ↓
Communication Application Service
   ↓
Email Provider Interface
   ↓
Infrastructure Adapter
   ↓
External Provider

El Domain no debe depender directamente de:

SendGrid;
SES;
Mailgun;
SMTP;
otro proveedor específico.

La elección concreta queda pendiente.

22. Provider Failure

Un fallo del proveedor de email nunca debe convertirse automáticamente en:

ALLOW

Ejemplo:

Email Verification
       ↓
Provider Failure
       ↓
NOT VERIFIED

y no:

Provider Failure
       ↓
Assume Verified

La arquitectura debe fallar de forma segura.

23. Email Delivery Reliability

La arquitectura deberá contemplar:

retries controlados;
timeouts;
provider errors;
rate limits del proveedor;
backoff;
observabilidad;
dead-letter handling cuando corresponda.

No se debe bloquear innecesariamente una transacción crítica de Identity esperando una operación externa.

Preferentemente:

Identity Transaction
      ↓
Outbox Event
      ↓
Notification Consumer
      ↓
Email Provider

Esto mantiene separadas la transacción de Identity y la entrega externa.

24. Transactional Outbox

Las notificaciones relevantes deberán utilizar el patrón Outbox cuando formen parte de un cambio transaccional.

Ejemplo:

Transaction
├── Email state updated
└── Email verification event created
             ↓
          COMMIT
             ↓
      Event Platform
             ↓
     Notification Worker
             ↓
       Email Provider

Esto evita el problema:

Database COMMIT
     +
Email send failed
     +
No retry information
25. No Sensitive Data in Events

Los eventos de comunicación no deben contener secretos.

Ejemplo permitido:

{
  "event_type": "identity.email.verification.requested",
  "user_id": "...",
  "tenant_id": "...",
  "email_reference": "...",
  "purpose": "EMAIL_VERIFICATION"
}

No debe contener:

password
token
mfa_secret
recovery_code
refresh_token

Cuando un worker necesite un secreto temporal para generar una comunicación, debe obtenerlo mediante mecanismos seguros y controlados.

26. Email Template Security

Los templates de Identity deberán tratarse como componentes de seguridad.

Deberán evitar:

inclusión accidental de secretos;
URLs no confiables;
contenido generado dinámicamente sin sanitización;
información excesiva sobre la cuenta;
exposición de tenant data;
logs con contenido sensible.

Los templates deberán estar versionados y sujetos a cambios controlados.

27. Verification URLs

Los enlaces de verificación deben:

utilizar HTTPS;
contener únicamente tokens diseñados específicamente para ese propósito;
no incluir passwords;
no incluir refresh tokens;
no incluir MFA secrets;
evitar información sensible en query parameters.

Cuando sea posible, el frontend puede recibir el token y enviarlo inmediatamente al backend mediante un canal seguro.

28. Token Leakage

Los sistemas de observabilidad deben evitar registrar:

/full-verification-url?token=SECRET

en:

logs;
traces;
analytics;
monitoring;
reverse proxies;
error reports.

Debe existir redaction específica para URLs y parámetros sensibles.

29. Browser Referrer Protection

La arquitectura web deberá evitar que un verification/recovery token pueda filtrarse mediante mecanismos de navegación o recursos de terceros.

La implementación deberá considerar:

políticas de referrer;
URLs de corta duración;
consumo inmediato;
no reutilización;
limpieza del token del estado del cliente después de procesarlo.

Los detalles exactos se definirán en la API/Web Security Specification.

30. Account Recovery and Email Ownership

Cuando un usuario utiliza email como mecanismo de recovery:

Verified Email
      ↓
Recovery Request
      ↓
Recovery Token
      ↓
Identity Recovery

la verificación del email no elimina las políticas adicionales de:

MFA;
risk;
account status;
session state;
tenant policy.

Por tanto:

Verified Email
≠
Unconditional Recovery Authorization
31. Compromised Email

EVOXA debe considerar que:

Un email verificado puede posteriormente quedar comprometido.

Por lo tanto, VERIFIED no significa que el canal sea permanentemente confiable.

Ante evidencia de compromiso, la política podrá:

Revoke Email Trust
        ↓
Require Additional Verification
        ↓
Revoke Sessions
        ↓
Revoke Refresh Tokens
        ↓
Require MFA / Recovery

La decisión dependerá del riesgo.

32. Email Provider Credentials

Las credenciales del proveedor deberán gestionarse según ADR-IDENTITY-027.

Nunca:

hardcodeadas;
almacenadas en Git;
incluidas en Docker images;
escritas en logs;
incluidas en eventos;
entregadas a AI Agents.

Deben utilizar:

Secret Manager
+
KMS / protected cryptographic infrastructure

según la arquitectura de plataforma.

33. Tenant Isolation

Las comunicaciones de Identity deberán respetar el tenant context.

Un mensaje de una organización no puede:

utilizar templates de otra organización sin autorización;
exponer información de otro tenant;
enviarse a un destinatario de otro tenant por un error de asociación;
utilizar configuraciones de seguridad incorrectas.

Los templates/configuraciones tenant-specific deberán estar aislados.

34. Custom Tenant Email Configuration

La plataforma podrá soportar posteriormente:

Organization
   ↓
Custom Email Branding
   ↓
Custom Sender
   ↓
Tenant Notification Policy

pero esta capacidad deberá permanecer separada de la lógica de seguridad de Identity.

Una organización no podrá configurar políticas que debiliten los mínimos de seguridad de plataforma.

35. Rate Limiting

Los endpoints de email deben estar sujetos a ADR-IDENTITY-032.

Especialmente:

/request-verification
/resend-verification
/change-email
/request-recovery

Se deberán controlar:

frecuencia;
destinatario;
usuario;
IP;
dispositivo;
tenant;
operación.

También deberán existir límites para evitar abuso contra proveedores externos.

36. Anti-Spam and Abuse

El sistema debe prevenir que EVOXA pueda utilizarse para:

Email Spam
Email Bombing
Verification Flooding
Recovery Flooding
Provider Abuse

Las medidas pueden incluir:

rate limiting;
cooldown;
progressive delays;
risk scoring;
CAPTCHA/challenge cuando corresponda;
provider throttling;
temporary blocking.

Los valores exactos quedan pendientes.

37. Auditability

Deberán existir eventos de auditoría como:

EMAIL_VERIFICATION_REQUESTED
EMAIL_VERIFICATION_SENT
EMAIL_VERIFICATION_COMPLETED
EMAIL_VERIFICATION_FAILED
EMAIL_VERIFICATION_EXPIRED
EMAIL_CHANGE_REQUESTED
EMAIL_CHANGE_VERIFIED
EMAIL_CHANGE_COMPLETED
EMAIL_CHANGE_REJECTED
SECURITY_NOTIFICATION_SENT
SECURITY_NOTIFICATION_FAILED

Los eventos exactos serán definidos en la Event Specification.

38. Security Events

Los siguientes casos podrán generar Security Events:

Repeated Verification Abuse
Verification Token Reuse
Verification Token Guessing
Mass Email Change
Suspicious Recovery Through Email
Unusual Email Change
Provider Abuse
Repeated Failed Verification
High-Risk Email Change

El severity será determinado por Security Policy.

39. Observability

Se deberán medir como mínimo:

verification_requests
verification_success
verification_failure
verification_expiration
verification_resend
email_change_requests
email_change_success
email_change_failure
notification_delivery_success
notification_delivery_failure
provider_latency
provider_errors
rate_limit_hits

Las métricas deberán evitar labels de alta cardinalidad.

Nunca debe utilizarse el email completo como una dimensión de métrica.

40. Privacy

Identity deberá aplicar data minimization.

No se debe copiar innecesariamente información del contenido de emails hacia:

logs;
analytics;
eventos;
AI context;
traces.

El sistema debe almacenar únicamente la información necesaria para:

identidad;
seguridad;
auditoría;
operación.
41. AI Integration

AI podrá ayudar a:

detectar abuso de emails;
detectar patrones de takeover;
identificar anomalías;
priorizar incidentes;
recomendar acciones.

AI no podrá:

marcar un email como verificado arbitrariamente;
aprobar un cambio crítico de email ignorando policy;
recuperar tokens;
revelar verification secrets;
desactivar rate limits;
enviar secretos;
convertir DENY en ALLOW.
42. Consequences
Positivas
Reduce account takeover.
Evita reutilización de verification tokens.
Protege frente a email enumeration.
Separa email ownership de identidad completa.
Integra email con Account Recovery de manera segura.
Permite cambiar de proveedor de email.
Facilita auditoría.
Permite notifications de seguridad.
Mantiene tenant isolation.
Reduce riesgo de abuso del proveedor.
Mantiene AI fuera de las decisiones críticas de Identity.
Negativas
Mayor complejidad.
Requiere infraestructura de comunicación.
Introduce dependencia operacional de proveedores externos.
Requiere políticas específicas de email.
Email delivery puede ser eventual.
Recovery de alto riesgo puede requerir mecanismos adicionales.
43. Alternatives Considered
A. Mark Email Verified on Registration

Rejected.

No demuestra control sobre la dirección.

B. Use Email as Permanent Authentication Factor

Rejected.

El control de un email puede cambiar o verse comprometido.

C. Email Verification Without Expiration

Rejected.

Aumenta el riesgo de replay y abuso.

D. Reusable Verification Tokens

Rejected.

Los tokens deben ser one-time.

E. Provider Directly Coupled to Domain

Rejected.

Generaría acoplamiento innecesario y dificultaría evolución.

F. Email Change Without Re-Verification

Rejected.

Permitiría registrar una dirección no demostrada como controlada.

44. Non-Negotiable Security Rules
Email verification debe demostrar control del canal.
Email verification no equivale automáticamente a identity proofing.
Verification tokens deben ser one-time.
Verification tokens deben expirar.
Tokens deben estar ligados a propósito.
Tokens no deben almacenarse en plaintext cuando pueda evitarse.
Tokens no deben aparecer en logs.
Email enumeration debe minimizarse.
Verification endpoints deben tener rate limiting.
Email change es una operación sensible.
Un nuevo email debe verificarse.
Verified email no equivale a autorización universal.
Provider failure nunca debe producir un ALLOW.
Provider credentials deben utilizar Secret Manager/KMS.
Notifications no sustituyen Audit/Security Events.
Cambios críticos deben producir trazabilidad.
Tenant isolation debe aplicarse a comunicaciones.
AI no puede aprobar o saltarse controles de Identity.
Email recovery debe respetar Account Recovery Policy.
Verification debe ser concurrency-safe.
Eventos de email no deben contener secretos.
Security notifications deben minimizar información sensible.
45. Traceability
Requirement	Source / Relationship
Identity lifecycle	Identity ESP / ARCHITECTURE-MAP
Authentication	IS-IDENTITY-005
MFA	IS-IDENTITY-010
Sessions	IS-IDENTITY-008
Refresh Tokens	IS-IDENTITY-009 / ADR-025
API Idempotency	ADR-IDENTITY-026
Secrets/KMS	ADR-IDENTITY-027
Observability	ADR-IDENTITY-028
Rate limiting	ADR-IDENTITY-032
Account Lifecycle	ADR-IDENTITY-033
Password Security	ADR-IDENTITY-034
Account Recovery	ADR-IDENTITY-035
Email Verification	ADR-IDENTITY-036

El Blueprint establece que las decisiones deben mantenerse trazables desde ESP → IS → Technical Tasks → Source Code.

46. Dependencies

Este ADR depende principalmente de:

ADR-IDENTITY-023
ADR-IDENTITY-024
ADR-IDENTITY-026
ADR-IDENTITY-027
ADR-IDENTITY-028
ADR-IDENTITY-032
ADR-IDENTITY-033
ADR-IDENTITY-034
ADR-IDENTITY-035

Y deberá materializarse posteriormente mediante:

IS-IDENTITY-036
Technical Tasks
API Contracts
Persistence Specification
Event Specification
Testing Specification
47. Pending Decisions

Quedan deliberadamente abiertos:

Verification
exact token TTL;
token entropy;
encoding;
hashing/HMAC;
resend cooldown;
maximum resend attempts;
verification attempt limits.
Email Change
whether old email confirmation is always required;
exact step-up requirements;
session revocation;
refresh-token revocation;
notification behavior.
Communication
provider;
fallback provider;
retry strategy;
provider failover;
delivery tracking;
bounce/complaint handling.
Security
exact anti-enumeration strategy;
risk thresholds;
CAPTCHA/challenge policy;
compromised-email response;
notification retention.
Tenant
custom templates;
custom sender domains;
tenant branding;
tenant-specific communication policies.

Estos valores no deben fijarse todavía sin una especificación posterior.

48. Acceptance Criteria

El ADR se considerará correctamente implementado cuando:

 Email ownership pueda verificarse explícitamente.
 Verification tokens sean one-time.
 Verification tokens tengan TTL.
 Tokens estén ligados a propósito.
 Replay sea rechazado.
 Verification sea concurrency-safe.
 Resend esté protegido por rate limiting.
 Email enumeration esté mitigado.
 Email change requiera verificación de la nueva dirección.
 Email change pueda requerir step-up según policy.
 Security notifications puedan emitirse para operaciones críticas.
 Notifications no sustituyan Audit/Security Events.
 Provider failures no produzcan bypass.
 Provider credentials estén protegidas.
 Comunicaciones utilicen Outbox cuando corresponda.
 Eventos no contengan secretos.
 Tenant isolation se aplique a comunicaciones.
 AI no pueda alterar el estado de verificación arbitrariamente.
 Existan pruebas de replay, enumeration, abuse, concurrency y tenant isolation.
 Existan métricas y observabilidad sin exponer PII innecesaria.
49. Next ADR

Con ADR-IDENTITY-036 queda definida la estrategia conceptual para que el email pueda funcionar como canal verificado y mecanismo controlado de comunicación dentro de Identity, sin convertirlo en una autoridad de autenticación absoluta.

El siguiente ADR natural es:

ADR-IDENTITY-037 — Identity API Token & Service Credential Security Strategy

para definir formalmente:

API Tokens;
Service Accounts;
Machine-to-Machine Authentication;
token lifecycle;
token scopes;
token rotation;
revocation;
credential hashing;
service identity;
AI Agent credentials;
workload identity;
secret exposure controls;
audit y tenant isolation.

Esto será especialmente importante antes de cerrar completamente la arquitectura de Identity para usuarios, servicios y AI Agents.
