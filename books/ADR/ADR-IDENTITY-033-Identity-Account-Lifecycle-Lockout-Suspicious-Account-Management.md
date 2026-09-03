ADR-IDENTITY-033 — Identity Account Lifecycle, Lockout & Suspicious Account Management

Estado: Proposed
Versión: 1.0
Fecha: 2026-09-03
Dominio: Identity
Prioridad: Critical
Tipo: Architecture Decision Record

1. Contexto

El dominio Identity necesita administrar el ciclo de vida completo de una cuenta y responder de forma segura cuando una cuenta presenta:

múltiples autenticaciones fallidas;
comportamiento sospechoso;
posibles credenciales comprometidas;
actividad anómala;
abuso;
incidentes de seguridad;
suspensión administrativa;
deshabilitación permanente o prolongada.

El Blueprint establece estados de ciclo de vida para User:

PENDING
ACTIVE
SUSPENDED
DISABLED

También define estados para otros recursos de Identity, incluyendo:

Membership;
Session;
Invitation;
Refresh Token;
MFA Factor.

ADR-IDENTITY-032 estableció mecanismos de:

Rate Limiting;
Brute-Force Protection;
Credential Stuffing Protection;
Password Spraying Protection;
Abuse Detection;
Progressive Protection.

Por lo tanto, este ADR define qué sucede con la cuenta cuando esos mecanismos detectan riesgo, evitando confundir:

Rate Limit → Lockout → Suspension → Disablement

como si fueran el mismo concepto.

2. Problema

Un sistema de Identity necesita responder preguntas como:

¿Cuándo una cuenta se considera sospechosa?
¿Cuándo se bloquea temporalmente?
¿Quién puede suspenderla?
¿Qué sucede con sus sesiones?
¿Qué sucede con sus Refresh Tokens?
¿Qué ocurre con MFA?
¿Puede iniciar sesión una cuenta suspendida?
¿Puede refrescar un token una cuenta suspendida?
¿Cómo se recupera una cuenta?
¿Qué ocurre ante una sospecha de compromiso?
¿Qué diferencia existe entre suspensión automática y administrativa?
¿Cómo evitar que un atacante use lockout para provocar un DoS contra una víctima?

Sin una estrategia explícita, diferentes partes del sistema podrían implementar comportamientos inconsistentes.

3. Decisión

EVOXA Identity utilizará un Account Lifecycle centralizado, gobernado por Identity y Security Policy Engine.

Los estados principales serán:

PENDING
   ↓
ACTIVE
   ↓
SUSPENDED
   ↓
ACTIVE
   ↓
DISABLED

No todas las transiciones estarán permitidas desde todos los estados.

Además, se diferenciarán cuatro conceptos:

1. Rate Limit

Restricción temporal sobre solicitudes.

2. Temporary Protection / Lockout

Restricción temporal de determinadas operaciones debido a actividad de riesgo.

3. Account Suspension

Cambio explícito del estado de la cuenta a SUSPENDED.

4. Account Disablement

Cambio del estado de la cuenta a DISABLED.

4. Principio fundamental

El estado de la cuenta debe ser una fuente única de decisión.

No deberá existir lógica dispersa como:

Login → own lockout logic

Refresh → different lockout logic

MFA → another account status

Password Reset → another status

En su lugar:

Request
   ↓
Identity Account State
   ↓
Security Policy
   ↓
Operation Decision
5. Account States
5.1 PENDING

La cuenta existe pero todavía no está completamente habilitada.

Puede corresponder a:

invitación;
verificación pendiente;
onboarding incompleto;
activación pendiente.

Una cuenta PENDING no deberá considerarse una cuenta completamente activa.

Las operaciones permitidas dependerán de la política.

6. ACTIVE

ACTIVE representa una cuenta habilitada para utilizar Identity normalmente, siempre que:

Authentication sea válida;
Authorization permita la operación;
Security Policy no imponga restricciones;
MFA sea satisfecha cuando corresponda;
Rate Limiting no bloquee la solicitud.
7. SUSPENDED

SUSPENDED representa una cuenta temporalmente restringida.

Puede producirse por:

sospecha de compromiso;
actividad abusiva;
investigación de seguridad;
decisión administrativa;
violación de políticas;
incident response.

La suspensión debe ser:

explícita;
auditable;
tenant-aware;
reversible mediante una transición autorizada.
8. DISABLED

DISABLED representa una cuenta que no puede utilizar Identity normalmente.

Puede corresponder a:

desactivación administrativa;
terminación de la cuenta;
incidente grave;
política organizacional;
cumplimiento;
decisión permanente o de larga duración.

DISABLED no debe interpretarse como un simple rate limit.

9. Diferencia entre Lockout y Suspension

Esta distinción es fundamental.

Lockout

Puede significar:

Too many failures
        ↓
Temporary restriction
        ↓
Automatic recovery
Suspension

Significa:

Security / Administrative Decision
        ↓
Account = SUSPENDED
        ↓
Explicit recovery required

Un lockout no debe convertirse automáticamente en suspensión permanente.

10. Temporary Lockout

Cuando se detecte actividad sospechosa, Identity podrá aplicar una restricción temporal.

Ejemplo:

ACTIVE
  ↓
Repeated failures
  ↓
Temporary Protection
  ↓
ACTIVE

La cuenta puede permanecer conceptualmente ACTIVE mientras determinadas operaciones están temporalmente restringidas.

Esto evita utilizar SUSPENDED para cada evento menor de brute force.

11. Progressive Protection

La estrategia de ADR-IDENTITY-032 se integrará con el lifecycle.

Conceptualmente:

Normal
   ↓
Rate Limit
   ↓
Progressive Delay
   ↓
Challenge
   ↓
Temporary Lockout
   ↓
Suspicious Account
   ↓
Possible Suspension

La transición a suspensión deberá estar gobernada por Security Policy.

12. Account Lockout Abuse

No se permitirá que un atacante pueda bloquear permanentemente una cuenta legítima simplemente provocando errores de autenticación.

Por ejemplo:

Attacker
   ↓
Repeated bad passwords
   ↓
Victim account
   ↓
Permanent lock

Este comportamiento debe evitarse.

Se favorecerán:

progressive delays;
risk-based challenges;
temporary restrictions;
IP/device controls;
MFA;
anomaly detection.
13. Suspicious Account

Una cuenta puede considerarse sospechosa sin estar todavía suspendida.

Conceptualmente:

ACTIVE
   +
Risk Signals
   ↓
SUSPICIOUS

SUSPICIOUS se tratará inicialmente como una condición/risk state, no necesariamente como un nuevo estado principal del lifecycle.

Esto evita ampliar innecesariamente el catálogo oficial:

ACTIVE
SUSPENDED
DISABLED

La señal de riesgo puede almacenarse y utilizarse por el Security Policy Engine.

14. Suspicious Signals

Pueden contribuir a determinar comportamiento sospechoso:

múltiples login failures;
credential stuffing;
password spraying;
MFA brute force;
refresh token reuse;
anomalous session behavior;
unusual device activity;
abnormal API token usage;
abnormal geographic/contextual behavior;
excessive password reset requests;
security policy violations.

Ninguna señal aislada deberá implicar automáticamente suspensión permanente.

15. Risk Evaluation

La evaluación conceptual será:

Identity Activity
      ↓
Signals
      ↓
Risk Evaluation
      ↓
Security Policy Engine
      ↓
Decision

Decisiones posibles:

ALLOW;
DENY;
CHALLENGE;
REAUTHENTICATE;
REQUIRE_MFA;
REVOKE.
16. Automatic Suspension

La suspensión automática podrá utilizarse para incidentes de alto riesgo.

Ejemplo:

Confirmed Token Reuse
        ↓
Security Event
        ↓
Policy
        ↓
Session / Token Revocation
        ↓
Possible Account Suspension

Sin embargo:

No toda detección de riesgo debe producir automáticamente SUSPENDED.

La severidad y acción deben estar gobernadas por política.

17. Administrative Suspension

Un administrador autorizado podrá suspender una cuenta cuando exista una razón legítima.

Debe requerirse:

Authorization;
tenant context;
Security Policy;
audit;
actor attribution;
reason.

Conceptualmente:

Admin
 ↓
Authorize
 ↓
Policy
 ↓
Suspend Account
 ↓
Audit
 ↓
Security Event
18. Suspension Reason

Toda suspensión debería registrar un motivo estructurado.

Ejemplos conceptuales:

SECURITY_RISK
SUSPECTED_COMPROMISE
ABUSE
POLICY_VIOLATION
ADMINISTRATIVE
COMPLIANCE
OTHER

Los códigos definitivos quedan pendientes.

No se debe almacenar información sensible innecesaria.

19. Suspension Metadata

Conceptualmente pueden existir:

suspended_at
suspended_by
suspension_reason
suspension_source
suspension_expires_at

Donde:

suspension_source podría diferenciar:

USER;
ADMIN;
SYSTEM;
SECURITY_POLICY;
AI_AGENT mediante acción autorizada.
20. Automatic Temporary Suspension

Una suspensión automática podría incluir una expiración:

SUSPENDED
   ↓
suspension_expires_at
   ↓
ACTIVE

Pero la activación automática deberá depender de la política.

Para incidentes de compromiso confirmado puede requerirse revisión explícita.

21. Account Recovery

La recuperación debe depender de la causa.

Temporary Lockout

Puede recuperarse automáticamente después del periodo establecido.

Suspicious Activity

Puede requerir:

reauthentication;
MFA;
identity verification.
Suspected Compromise

Puede requerir:

revoke sessions;
revoke token families;
credential reset;
MFA verification;
administrative review.
Disabled

Puede requerir una acción administrativa explícita.

22. Sessions al Suspender una Cuenta

Al suspender una cuenta, las sesiones activas deberán tratarse según la Security Policy.

Para una sospecha de compromiso de alto riesgo:

Account Suspended
      ↓
Revoke Sessions
      ↓
Revoke Refresh Token Families

Esto evita que una sesión previamente autenticada continúe proporcionando acceso.

23. Access JWT durante Suspension

Debe distinguirse entre:

JWT criptográficamente válido;
autorización actual de la cuenta.

Un access token puede seguir siendo criptográficamente válido hasta su expiración.

Pero para operaciones que requieran consulta de estado de cuenta, una cuenta suspendida debe recibir:

DENY / REAUTHENTICATE / appropriate security decision

según la política.

La arquitectura no debe asumir que:

Valid JWT
=
Active Account
24. Refresh Tokens durante Suspension

Una cuenta SUSPENDED no deberá poder utilizar Refresh Tokens normalmente.

Conceptualmente:

Refresh Request
      ↓
Account State = SUSPENDED
      ↓
DENY

Además, para una suspensión de seguridad se recomienda:

revocar token families;
revocar sesiones;
generar Security Event.
25. MFA durante Suspension

Suspender una cuenta no debe:

eliminar MFA;
convertir MFA en opcional;
destruir factores;
eliminar recovery configuration.

Los factores deben conservarse según las políticas de retención.

La recuperación puede requerir MFA o mecanismos de recovery apropiados.

26. MFA Factor Compromise

Si se sospecha que un factor MFA fue comprometido:

MFA Factor
   ↓
Suspicious / Compromised
   ↓
REVOKED
   ↓
Alternative Factor / Recovery

Esto se relaciona directamente con:

ADR-IDENTITY-023
ADR-IDENTITY-024

27. Credential Compromise

Cuando se detecte compromiso de credenciales:

impedir autenticación con credencial comprometida;
solicitar cambio de contraseña;
evaluar sesiones;
evaluar refresh token families;
evaluar MFA;
generar Security Event;
aplicar Security Policy.

No necesariamente implica DISABLED.

28. Password Reset después de Compromise

Cuando una cuenta sea considerada comprometida, Password Reset puede convertirse en requisito.

La secuencia conceptual:

Compromise Detection
       ↓
Restrict Credential
       ↓
Identity Verification
       ↓
Password Reset
       ↓
MFA / Step-Up
       ↓
Restore Access
29. Refresh Token Reuse

Refresh Token Reuse es una señal de seguridad particularmente importante.

Según ADR-IDENTITY-025:

Reuse Detected
      ↓
Security Event
      ↓
Revoke Family
      ↓
Potential Session Revocation

Este evento puede alimentar el Account Risk Engine.

Dependiendo de la política:

Reuse
 ↓
Suspicious

o:

High Confidence Compromise
 ↓
SUSPENDED
30. Account Suspension y Tenant Isolation

Una suspensión debe aplicarse dentro del contexto correcto.

Debe existir distinción entre:

User-level suspension

Afecta la cuenta global.

Organization-level restriction

Afecta la participación del usuario en una organización concreta.

Esto es importante porque:

User
 ├── Organization A
 └── Organization B

Una restricción de Membership en Organization A no necesariamente significa que el User deba ser suspendido globalmente.

31. Membership vs User Suspension

Se establece:

Suspender un Membership no implica automáticamente suspender el User.

Ejemplo:

User = ACTIVE

Membership A = SUSPENDED
Membership B = ACTIVE

Esto permite modelos multi-tenant.

La política puede determinar cuándo un incidente en una organización justifica una suspensión global.

32. Organization-Level Abuse

Una organización también puede generar actividad abusiva.

Ejemplos:

múltiples cuentas atacando sistemas;
automation abuse;
excessive API token activity;
security policy violations.

La respuesta puede aplicarse a:

actor;
membership;
organization;
user;
sessions;
token families.

Siempre respetando la jerarquía de políticas.

33. Account Reinstatement

Para pasar:

SUSPENDED → ACTIVE

deberá existir una operación autorizada.

Puede requerir:

identity verification;
MFA;
password reset;
admin approval;
security review.

El método exacto depende de la razón de suspensión.

34. Disabled Account Recovery

DISABLED tendrá controles más estrictos.

No debe existir un simple:

POST /unlock

que permita reactivar una cuenta sin autorización adecuada.

La reactivación deberá:

estar autorizada;
validarse mediante policy;
registrarse en audit;
generar security event cuando corresponda.
35. Account Lifecycle Transitions

El catálogo conceptual será:

PENDING → ACTIVE

PENDING → DISABLED

ACTIVE → SUSPENDED

ACTIVE → DISABLED

SUSPENDED → ACTIVE

SUSPENDED → DISABLED

DISABLED → ACTIVE

No todas las transiciones deberán estar disponibles para todos los actores.

36. Invalid Transitions

Ejemplos:

DISABLED → ACTIVE

sin autorización adecuada:

DENY

o:

PENDING → SUSPENDED

cuando la operación no tenga sentido según la política:

DENY

Las transiciones deben ser explícitas y auditables.

37. Account State Machine

Conceptualmente:

                    ┌──────────────┐
                    │    PENDING   │
                    └──────┬───────┘
                           │
                        Activate
                           ↓
                    ┌──────────────┐
                    │    ACTIVE    │
                    └───┬──────┬───┘
                        │      │
                  Suspend      │ Disable
                        ↓      ↓
                ┌──────────┐  ┌──────────┐
                │ SUSPENDED│  │ DISABLED │
                └────┬─────┘  └────┬─────┘
                     │              │
                  Reinstate       Reactivate
                     │              │
                     └──────┬───────┘
                            ↓
                         ACTIVE
38. Security Event Model

Cambios importantes deberán generar Security Events.

Ejemplos:

account_suspended;
account_reinstated;
account_disabled;
account_reactivated;
suspicious_account_detected;
brute_force_detected;
credential_compromise_detected;
account_lockout;
security_restriction_applied.

Estos eventos deberán ser:

auditables;
tenant-aware;
correlacionables;
sin secretos.
39. Audit Model

Audit deberá registrar:

actor;
target user;
tenant;
previous state;
new state;
reason;
source;
timestamp;
correlation ID;
policy ID/version cuando corresponda.

Nunca deberá registrar:

password;
password hash;
refresh token;
access token;
MFA secret;
recovery code.
40. Notifications

Para eventos importantes, Identity podrá generar notificaciones de seguridad.

Ejemplos:

cuenta suspendida;
nuevo bloqueo;
actividad sospechosa;
cambio de contraseña;
cambio MFA;
sesión revocada;
posible compromiso.

La notificación nunca debe revelar información que facilite al atacante validar una cuenta o mecanismo de seguridad.

41. Anti-Enumeration

Los endpoints públicos no deben revelar innecesariamente:

Account exists
Account does not exist
Account suspended
Account disabled

cuando dicha información pueda utilizarse para enumeración.

Las respuestas públicas deberán seguir políticas de privacy/security.

42. Abuse Response Matrix

Conceptualmente:

Riesgo	Respuesta
Few failed logins	Rate limit
Repeated failures	Progressive protection
Brute force	Temporary restriction
Credential stuffing	Throttle / challenge / deny
Password spraying	Distributed protection
MFA brute force	Challenge restriction
Suspicious session	Reauthentication
Refresh reuse	Revoke family/session
Confirmed compromise	Suspension possible
Administrative issue	Suspension
Permanent disablement	DISABLED

Los valores exactos serán definidos por Security Policy.

43. Fail Closed

Si Identity no puede determinar de forma confiable el estado de seguridad necesario para una operación sensible:

no deberá asumir que la cuenta está segura.

Debe producirse una decisión segura:

DENY;
REAUTHENTICATE;
REQUIRE_MFA;
temporal unavailability.
44. High Availability Considerations

El Account Lifecycle debe funcionar correctamente en arquitectura distribuida.

Dos API instances no deben poder ejecutar simultáneamente:

ACTIVE → SUSPENDED

y:

SUSPENDED → ACTIVE

sin control de concurrencia.

Las transiciones críticas deberán ser:

transaccionales;
concurrency-safe;
auditables.
45. Concurrency

Debe evitarse:

Request A → suspend
Request B → reinstate
Request C → disable

produciendo un estado impredecible.

Se deberán utilizar mecanismos como:

optimistic concurrency;
version fields;
conditional updates;
database transactions.

La estrategia exacta se definirá en función de la persistencia implementada.

46. Idempotency

Operaciones administrativas críticas como:

suspend;
disable;
reinstate;
reactivate;

deberán poder manejar retries de forma segura.

Cuando corresponda, deberán soportar:

Idempotency-Key

según ADR-IDENTITY-026.

Ejemplo:

Suspend Account
      ↓
Network Retry
      ↓
Same Idempotency-Key
      ↓
No duplicate side effect
47. AI Agents

Los AI Agents podrán detectar señales y recomendar acciones.

También podrán ejecutar acciones previamente autorizadas mediante herramientas controladas.

Pero:

no podrán suspender cuentas arbitrariamente;
no podrán reactivar cuentas sin autorización;
no podrán modificar lifecycle policies;
no podrán eliminar auditoría;
no podrán ignorar MFA;
no podrán saltarse Authorization.

Toda acción de AI Agent deberá registrar:

actor = AI_AGENT

cuando corresponda.

48. AI-Assisted Suspicious Account Detection

AI puede complementar el análisis de:

anomalías;
comportamiento;
patrones distribuidos;
credential stuffing;
account compromise.

Pero:

AI no será la única autoridad para determinar un cambio crítico de estado.

Las acciones deberán estar gobernadas por:

Signals
 ↓
Risk Model
 ↓
Security Policy
 ↓
Authorized Action
49. Observability

Se deberán medir:

active accounts;
suspended accounts;
disabled accounts;
lockouts;
suspicious detections;
reinstatements;
credential compromise;
session revocations;
refresh token family revocations.

También:

tiempo promedio de suspensión;
frecuencia de falsos positivos;
número de reinstatements;
cuentas afectadas por abuso.
50. Security Monitoring

Deberán existir alertas para:

spike de suspensions;
spike de lockouts;
múltiples cuentas comprometidas;
múltiples tenants afectados;
unusual administrative suspensions;
mass disablement;
repeated reinstatement;
AI Agent attempting unauthorized lifecycle action.

Un aumento masivo de suspensiones puede indicar:

un ataque contra el propio mecanismo de seguridad.

51. Mass Account Protection

Una operación que afecte grandes cantidades de cuentas deberá tener controles adicionales.

Ejemplos:

bulk suspension;
bulk disable;
bulk reinstatement.

Podrán requerirse:

autorización adicional;
MFA;
approval workflow;
rate limiting;
audit;
dry-run;
rollback strategy.
52. Emergency Security Response

Durante un incidente grave puede ser necesario ejecutar:

Revoke Sessions
+
Revoke Refresh Families
+
Require MFA
+
Suspend Accounts

Estas operaciones deben contar con mecanismos de emergencia controlados.

La emergencia no significa:

eliminar controles de autorización.

53. Recovery después de Security Incident

Después de un incidente, la recuperación de cuentas deberá considerar:

credentials;
sessions;
refresh tokens;
MFA;
trusted devices;
API tokens;
security policies.

No se debe limitar la recuperación simplemente a:

Enable Account

si existe evidencia de compromiso.

54. Consecuencias positivas

Esta decisión proporciona:

lifecycle consistente;
separación clara entre lockout y suspension;
protección contra account lockout abuse;
mejor respuesta ante compromisos;
control centralizado;
tenant-aware security;
integración con MFA;
integración con sessions;
integración con refresh tokens;
auditabilidad;
observabilidad;
soporte para AI-assisted security sin delegar el control crítico.
55. Consecuencias negativas

Introduce:

mayor complejidad de lifecycle;
más estados y transiciones;
necesidad de políticas;
necesidad de recovery workflows;
complejidad de concurrencia;
necesidad de auditing;
riesgo de falsos positivos;
necesidad de herramientas administrativas seguras.
56. Alternativas consideradas
A. Account Lockout permanente después de N intentos

Rechazada.

Facilita Account Lockout Abuse y puede convertirse en un mecanismo de DoS.

B. Un único estado ACTIVE/INACTIVE

Rechazada.

No permite diferenciar:

pendiente;
sospechoso;
suspendido;
deshabilitado.
C. Suspender automáticamente ante cualquier comportamiento sospechoso

Rechazada.

Generaría demasiados falsos positivos y problemas de disponibilidad.

D. Lifecycle centralizado + Progressive Protection + Security Policy

Adoptada.

Permite separar:

Rate Limiting;
Lockout;
Risk;
Suspension;
Disablement.
57. No negociables
User lifecycle debe soportar PENDING, ACTIVE, SUSPENDED, DISABLED.
Lockout no debe confundirse con Suspension.
Account Lockout permanente no será el mecanismo principal.
Suspicious Account será una condición de riesgo, no necesariamente un nuevo lifecycle state.
Security Policy Engine gobernará las decisiones de riesgo.
Authentication deberá respetar account state.
Refresh deberá respetar account state.
Sessions deberán gestionarse durante suspension.
Refresh Token Families deberán revocarse cuando la política lo determine.
MFA no podrá deshabilitarse como mecanismo automático de recuperación.
Tenant isolation deberá preservarse.
Membership suspension no implica automáticamente User suspension.
Cambios de lifecycle deberán auditarse.
Cambios críticos deberán generar Security Events.
Las transiciones deberán ser concurrency-safe.
Operaciones críticas deberán ser idempotentes cuando corresponda.
No se deben revelar estados sensibles mediante account enumeration.
AI Agents no pueden saltarse lifecycle authorization.
No se pueden registrar secretos.
Mass account operations deberán tener controles adicionales.
Failover no puede generar estados inconsistentes.
Recovery posterior a compromiso debe contemplar sessions, tokens, MFA y credentials.
Security controls deben fallar closed.
58. Traceability
Referencia	Relación
Blueprint — Identity Domain	User lifecycle
Blueprint — Authorization	RBAC/ABAC/Policy
ESP-0001	Identity domain model
ADR-IDENTITY-003	Sessions / Refresh lifecycle
ADR-IDENTITY-005	Security Policy Engine
ADR-IDENTITY-008	Security / Cryptography
ADR-IDENTITY-010	Testing
ADR-IDENTITY-022	Consumer Idempotency
ADR-IDENTITY-024	MFA Attempts / Recovery
ADR-IDENTITY-025	Refresh Token Security
ADR-IDENTITY-026	API Idempotency
ADR-IDENTITY-028	Observability
ADR-IDENTITY-031	HA & Failover
ADR-IDENTITY-032	Rate Limiting / Abuse Prevention
ADR-IDENTITY-033	Account Lifecycle / Lockout / Suspicious Accounts
59. Dependencias

Este ADR depende de:

User domain;
Authentication;
Authorization;
Security Policy Engine;
Sessions;
Refresh Tokens;
MFA;
Audit;
Security Events;
Rate Limiting;
Abuse Detection;
Observability;
PostgreSQL transactions;
Idempotency.
60. Decisiones pendientes

Quedan pendientes:

límites exactos para lockout;
duración de temporary lockout;
progressive delay values;
suspicious-account scoring;
risk thresholds;
criterios exactos de automatic suspension;
criterios de confirmed compromise;
suspensión global vs membership;
automatic reinstatement;
manual reinstatement workflow;
disabled-account recovery;
credential reset requirements;
session revocation policy;
refresh-family revocation policy;
MFA recovery requirements;
notification strategy;
bulk suspension workflow;
emergency security controls;
lifecycle database schema;
concurrency/version strategy;
exact Security Event catalog;
retention de lifecycle/security signals;
integración final con Risk Engine;
AI-assisted detection governance.
61. Acceptance Criteria

El ADR se considerará implementado cuando:

 exista lifecycle PENDING → ACTIVE → SUSPENDED → DISABLED;
 existan transiciones válidas e inválidas definidas;
 Lockout esté separado de Suspension;
 exista temporary lockout;
 exista protección contra account lockout abuse;
 exista suspicious-account detection;
 Security Policy gobierne decisiones de suspensión;
 exista administrative suspension;
 exista automatic suspension controlada;
 exista reinstatement;
 exista disabled-account recovery controlado;
 sessions sean tratadas correctamente durante suspension;
 refresh tokens sean tratados correctamente;
 MFA permanezca protegido;
 credential compromise tenga workflow;
 tenant isolation sea preservado;
 membership y user suspension estén diferenciados;
 lifecycle transitions sean concurrency-safe;
 operaciones críticas sean idempotentes;
 lifecycle events sean auditables;
 security events sean generados;
 account enumeration esté protegida;
 bulk operations estén protegidas;
 AI Agents estén sujetos a autorización;
 observability esté implementada;
 existan pruebas de lifecycle;
 existan pruebas de lockout;
 existan pruebas de compromise response;
 existan pruebas de concurrent transitions;
 existan pruebas de tenant isolation;
 existan pruebas de recovery.
62. Siguiente ADR

El siguiente ADR lógico es:

ADR-IDENTITY-034 — Identity Password Security & Credential Lifecycle

Ahí podemos definir formalmente todo el ciclo de las credenciales de usuario: creación de contraseña, Argon2id, políticas de complejidad, password history, expiración cuando corresponda, cambio de contraseña, reset, credenciales comprometidas, rehash, protección contra credential stuffing, sesiones después de un cambio de contraseña y cómo interactúa todo esto con MFA, Security Policy y Account Lifecycle.
