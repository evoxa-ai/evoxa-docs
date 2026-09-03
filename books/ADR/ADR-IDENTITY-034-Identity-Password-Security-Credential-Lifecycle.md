ADR-IDENTITY-034 — Identity Password Security & Credential Lifecycle

Estado: Proposed
Versión: 1.0
Fecha: 2026-09-03
Dominio: Identity
Prioridad: Critical
Tipo: Architecture Decision Record

1. Contexto

Las credenciales constituyen uno de los componentes más sensibles de Identity.

La estrategia de EVOXA ya establece:

autenticación centralizada;
Argon2id para protección de contraseñas;
MFA;
Sessions;
Refresh Tokens;
Security Policy Engine;
Rate Limiting;
Abuse Prevention;
Account Lifecycle;
Audit y Security Events.

ADR-IDENTITY-032 estableció protección contra:

brute force;
credential stuffing;
password spraying;
abuso de Password Reset.

ADR-IDENTITY-033 estableció el lifecycle:

PENDING
ACTIVE
SUSPENDED
DISABLED

Este ADR define cómo deben gestionarse las contraseñas y credenciales durante todo su ciclo de vida, desde su creación hasta cambio, recuperación, compromiso y eventual invalidación.

2. Problema

Una implementación incorrecta del manejo de contraseñas puede provocar:

almacenamiento de credenciales en texto plano;
hashes débiles;
exposición de contraseñas en logs;
password reset inseguro;
reutilización de contraseñas comprometidas;
sesiones que permanecen activas después de un cambio crítico;
credenciales que continúan funcionando después de un compromiso;
ausencia de protección contra credential stuffing;
recuperación de cuentas que permite bypass de MFA.

Además, no basta con definir cómo almacenar el password.

Es necesario definir:

Creation
   ↓
Storage
   ↓
Authentication
   ↓
Change
   ↓
Reset
   ↓
Compromise
   ↓
Invalidation
   ↓
Recovery
3. Decisión

EVOXA utilizará una estrategia centralizada de:

Secure Password Lifecycle Management

basada en:

Argon2id;
password hashing adaptativo;
nunca almacenar passwords en texto plano;
nunca registrar passwords;
password reset mediante mecanismo seguro;
protección contra brute force;
protección contra credential stuffing;
integración con MFA;
integración con Account Lifecycle;
invalidación apropiada de sesiones/tokens después de cambios críticos;
auditoría de operaciones sensibles;
Security Events;
rehash progresivo cuando sea necesario.
4. Password como Credential

La contraseña será considerada una credencial de autenticación.

El dominio no debe tratar:

password

como un dato ordinario de usuario.

La contraseña deberá permanecer:

fuera de logs, events, audit payloads, traces, analytics y AI context.

5. Password Storage

Las contraseñas deberán almacenarse únicamente como:

password hash

Nunca como:

plaintext;
reversible encryption;
Base64;
encoding;
cifrado reversible como sustituto de hashing.
6. Argon2id

La estrategia adoptada será:

Argon2id

como algoritmo principal de password hashing.

Argon2id proporciona una función de derivación de contraseña diseñada para incrementar el costo computacional y de memoria de ataques offline.

Los parámetros exactos quedan pendientes.

7. Password Hash Parameters

La configuración de Argon2id deberá considerar:

memory cost;
time cost;
parallelism;
salt;
output length;
version.

Los parámetros exactos deberán definirse según:

capacidad de infraestructura;
rendimiento objetivo;
pruebas de seguridad;
evolución tecnológica.

No se fijan valores arbitrarios en este ADR.

8. Unique Salt

Cada contraseña deberá utilizar un salt criptográficamente seguro y único.

El salt:

no necesita ser secreto;
debe almacenarse asociado al hash;
debe generarse automáticamente;
no debe reutilizarse entre usuarios.

El objetivo es evitar que dos usuarios con la misma contraseña produzcan necesariamente el mismo hash.

9. Password Hash Verification

Durante login:

Password Input
      ↓
Argon2id Verification
      ↓
Hash Match?
   ┌──┴──┐
  YES    NO
   ↓      ↓
Continue  Failure

La aplicación nunca debe comparar passwords mediante comparación manual del texto.

10. Password Hash Rehashing

Si los parámetros de hashing evolucionan, Identity podrá detectar:

hash válido pero parámetros obsoletos.

Conceptualmente:

Login
 ↓
Verify old hash
 ↓
Authentication succeeds
 ↓
Rehash with current parameters
 ↓
Persist new hash

Este mecanismo permite mejorar la seguridad sin obligar a todos los usuarios a cambiar inmediatamente su contraseña.

11. Password Migration

Si EVOXA incorpora usuarios provenientes de otro sistema, las contraseñas no deberán migrarse como plaintext.

Las estrategias posibles incluyen:

password reset obligatorio;
migración mediante mecanismo criptográfico compatible;
progressive rehash;
migración controlada.

La estrategia exacta para legacy migration queda pendiente.

12. Password Creation

Una contraseña podrá establecerse durante:

registration;
invitation acceptance;
password reset;
administrative provisioning, cuando corresponda;
credential recovery.

Cada operación deberá pasar por las políticas de seguridad correspondientes.

13. Password Policy

La Password Policy podrá considerar:

minimum length;
maximum length;
allowed characters;
breached password detection;
password history;
contextual restrictions.

La política exacta deberá ser configurable y gobernada por Security Policy.

14. Password Length

La estrategia deberá priorizar contraseñas suficientemente largas sobre reglas excesivamente complejas.

No se deberá asumir que:

más reglas arbitrarias de caracteres = necesariamente mayor seguridad.

La política exacta de longitud queda pendiente de definición y validación.

15. Password Complexity

Podrán establecerse requisitos como:

letras;
números;
caracteres especiales.

Sin embargo, estas reglas no deben convertirse en una fuente innecesaria de:

passwords predecibles;
reutilización;
dificultad de recuperación.

La política final deberá considerar seguridad y usabilidad.

16. Breached Password Protection

Identity podrá verificar si una contraseña elegida aparece en conjuntos conocidos de credenciales comprometidas.

La verificación deberá diseñarse para evitar:

enviar plaintext password a servicios externos;
registrar passwords;
filtrar credenciales.

La integración concreta queda pendiente.

17. Password History

Identity podrá mantener un historial protegido de hashes anteriores para evitar reutilización inmediata.

Conceptualmente:

Current Password
      ↓
Password History
      ↓
Compare
      ↓
Reuse?

Si la política lo prohíbe:

DENY

La cantidad exacta de contraseñas históricas queda pendiente.

18. Password Change

Un usuario autenticado podrá cambiar su contraseña mediante una operación protegida.

Flujo:

Authenticated User
       ↓
Current Password Verification
       ↓
Security Policy
       ↓
New Password Validation
       ↓
Hash
       ↓
Persist
       ↓
Security Event / Audit
19. Current Password Verification

Para cambios iniciados voluntariamente por el usuario, la política podrá exigir verificar la contraseña actual.

Esto reduce el riesgo de que una sesión parcialmente comprometida permita cambiar inmediatamente la credencial.

En operaciones de recuperación, la verificación puede ser diferente.

20. Password Change + MFA

Los cambios de contraseña pueden ser operaciones sensibles.

Security Policy podrá exigir:

Password Change
      ↓
Step-Up
      ↓
MFA
      ↓
Change Password

No se deberá asumir que una sesión existente siempre es suficiente para modificar una credencial crítica.

21. Password Reset

Password Reset será diferente de Password Change.

Password Change

El usuario normalmente conoce la contraseña actual.

Password Reset

El usuario no puede utilizar la contraseña actual y debe demostrar control mediante un mecanismo de recuperación.

22. Password Reset Flow

Flujo conceptual:

Reset Request
      ↓
Rate Limit
      ↓
Generic Response
      ↓
Recovery Challenge
      ↓
Identity Verification
      ↓
New Password
      ↓
MFA / Policy
      ↓
Password Updated

La implementación exacta depende del mecanismo de recovery definido por Identity.

23. Password Reset Token

Si se utilizan tokens de recuperación, deberán ser:

criptográficamente seguros;
de corta duración;
de uso limitado;
no reutilizables;
protegidos;
revocables cuando corresponda.

No deberán aparecer en:

logs;
analytics;
audit payloads;
URLs cuando exista una alternativa segura.
24. Reset Token Storage

El token de reset no debería almacenarse en plaintext cuando no sea necesario.

Se favorecerá almacenar una representación protegida que permita validar el token sin conservar el secreto recuperable.

La estrategia criptográfica exacta queda pendiente.

25. Password Reset Enumeration

El endpoint de solicitud de reset debe evitar revelar:

Account exists

vs.

Account does not exist

Las respuestas públicas deberán ser suficientemente uniformes.

Esto se relaciona con:

ADR-IDENTITY-032 — Rate Limiting & Abuse Prevention.

26. Password Reset Abuse

Password Reset deberá estar protegido contra:

flooding;
email bombing;
enumeration;
automated abuse;
token guessing;
repeated requests.

Debe utilizar:

rate limiting;
challenge;
abuse detection;
audit;
security events.
27. Password Reset + Sessions

Cuando se cambie la contraseña como resultado de un reset, Security Policy deberá evaluar las sesiones existentes.

Para un recovery asociado a sospecha de compromiso, la estrategia recomendada es:

Password Reset
      ↓
Revoke Existing Sessions
      ↓
Revoke Refresh Token Families
      ↓
Reauthenticate
      ↓
MFA

La política exacta puede variar según el riesgo.

28. Password Change + Sessions

Un cambio voluntario de contraseña también deberá evaluar sesiones activas.

Podrá existir una política diferenciada entre:

mantener sesión actual;
revocar otras sesiones;
revocar todas;
requerir reauthentication.

La decisión deberá ser centralizada por Security Policy.

29. Credential Compromise

Si existe evidencia de que una contraseña fue comprometida:

Compromise Detected
      ↓
Credential Invalidated
      ↓
Password Reset Required

Además, podrá requerirse:

session revocation;
refresh token family revocation;
MFA;
trusted device review;
security review.
30. Credential Compromise Detection

Las señales pueden provenir de:

credential stuffing;
unusual login behavior;
breach intelligence;
user report;
administrator;
security systems;
AI-assisted anomaly detection.

AI puede aportar señales, pero no debe poder declarar por sí sola una acción crítica sin la política correspondiente.

31. Password Compromise vs Account Compromise

Debe diferenciarse:

Password Compromised

La credencial puede haber sido expuesta.

Account Compromised

Existe evidencia de acceso o control no autorizado sobre la cuenta.

Una password comprometida puede provocar:

Credential Reset

mientras un account compromise puede requerir:

Credential Reset
+
Session Revocation
+
Refresh Token Revocation
+
MFA
+
Account Suspension

según el riesgo.

32. Password Expiration

No se establecerá una expiración periódica obligatoria como regla universal en este ADR.

La expiración forzada puede:

generar passwords predecibles;
aumentar reutilización;
generar frustración;
provocar comportamientos inseguros.

Cuando exista evidencia de compromiso o una política específica que lo requiera:

Password Reset / Change deberá poder ser exigido inmediatamente.

33. Forced Password Change

Una cuenta puede marcarse como:

PASSWORD_CHANGE_REQUIRED

como condición de seguridad.

Esto puede ocurrir después de:

provisioning;
reset;
credential compromise;
migración;
política de organización.

El indicador exacto podrá formar parte del modelo de credential state.

34. Temporary Credentials

Si se crean credenciales temporales:

deberán tener duración limitada;
no deben convertirse automáticamente en credenciales permanentes;
deberán requerir cambio;
deberán estar auditadas.

No deben utilizarse passwords administrativas compartidas.

35. Administrative Password Reset

Los administradores podrán iniciar procesos de recuperación cuando la política lo permita.

Pero:

Un administrador no debería poder ver la contraseña del usuario.

La operación debe generar:

audit event;
security event cuando corresponda;
actor attribution;
tenant context.
36. Support Personnel

Personal de soporte no deberá tener acceso directo a:

password hashes;
passwords;
MFA secrets;
recovery codes;
refresh tokens.

Las operaciones de soporte deben ejecutarse mediante workflows autorizados.

37. Passwords y Audit

Nunca se deben registrar:

password
password_hash
new_password
old_password
reset_token

Ni siquiera parcialmente.

Los logs pueden registrar:

password_change_success
password_change_failed
password_reset_requested
password_reset_completed

sin incluir el secreto.

38. Passwords y Events

Los Domain Events y Security Events no deberán contener passwords.

Ejemplo válido:

{
  "event_type": "identity.password_changed",
  "user_id": "...",
  "tenant_id": "...",
  "actor": "...",
  "timestamp": "..."
}

Ejemplo prohibido:

{
  "password": "..."
}
39. Passwords y AI

Las contraseñas no deben entrar en:

prompts;
AI context;
embeddings;
model training;
traces;
AI logs.

AI Agents tampoco deberán poder:

leer passwords;
leer password hashes innecesariamente;
generar passwords en nombre del usuario como mecanismo de autenticación;
recuperar secretos;
saltarse password policy.
40. Password Policy y Tenant

Las organizaciones podrán tener políticas específicas cuando la arquitectura lo permita.

Por ejemplo:

Organization A
→ stronger password policy

Pero ninguna política de tenant podrá reducir controles mínimos de seguridad de la plataforma.

41. Password Policy Precedence

La precedencia conceptual será:

Regulatory
    ↓
Platform Security
    ↓
Organization Security
    ↓
Role
    ↓
User
    ↓
Context

Y:

DENY > ALLOW

42. Authentication Failure

Un password incorrecto deberá producir una respuesta genérica.

No debe revelarse:

si el usuario existe;
si el password era parcialmente correcto;
qué política falló;
qué información interna fue utilizada.

Las fallas se integrarán con:

Rate Limiting;
Abuse Detection;
Security Events.
43. Timing Considerations

La autenticación debe diseñarse evitando diferencias de tiempo innecesarias que faciliten enumeración.

Las operaciones de verificación deberán utilizar mecanismos apropiados del algoritmo criptográfico.

No se deberán crear shortcuts como:

User doesn't exist
→ return immediately

si eso crea una señal observable innecesaria.

44. Password Reset Completion

Al completar exitosamente un reset:

persistir nuevo hash;
invalidar el mecanismo de reset;
actualizar credential metadata;
evaluar sesiones;
evaluar refresh tokens;
generar audit;
generar security event cuando corresponda;
aplicar MFA/policy;
notificar al usuario cuando corresponda.
45. Transactional Consistency

El cambio de password y los cambios de seguridad relacionados deberán manejarse de forma consistente.

Cuando corresponda, deberán utilizar una transacción para garantizar que:

Password Updated
+
Credential State Updated
+
Security State Updated

no queden accidentalmente en estados incompatibles.

Eventos posteriores deberán utilizar Outbox cuando correspondan a eventos de dominio.

46. Password History Consistency

La actualización del password y del password history deberá ser consistente.

No debería ocurrir:

Password = New
History = Old

si la operación requiere que ambos estados se actualicen conjuntamente.

47. Concurrent Password Changes

Deben manejarse escenarios como:

Request A → Password Change
Request B → Password Change

simultáneamente.

La estrategia debe impedir que una operación sobrescriba silenciosamente a otra sin control.

Podrán utilizarse:

optimistic concurrency;
row locking;
version fields;
conditional updates.

La implementación exacta queda pendiente.

48. Password Reset Race Conditions

También debe protegerse:

Reset Token
   ↓
Request A
Request B

Solo una operación válida deberá consumir el mecanismo de reset.

Esto se relaciona con:

token lifecycle;
idempotency;
concurrency control.
49. Password Reset Token Expiration

Los mecanismos de reset deberán tener expiración.

Conceptualmente:

CREATED
   ↓
PENDING
   ↓
USED / EXPIRED / REVOKED

Una vez utilizado:

no podrá reutilizarse.

Los valores exactos de TTL quedan pendientes.

50. Credential Metadata

Podrán mantenerse metadatos no sensibles como:

password_changed_at;
password_version;
hash_algorithm;
hash_parameters_version;
password_change_required;
compromised_at.

Nunca deberán contener el password.

51. Security Events

Eventos conceptuales:

password_changed;
password_reset_requested;
password_reset_completed;
password_reset_failed;
password_compromise_detected;
credential_invalidated;
password_policy_violation.

Los nombres definitivos deberán alinearse con el Event Schema Registry.

52. Notifications

Podrán generarse notificaciones para:

password changed;
password reset;
suspicious password activity;
credential compromise.

Las notificaciones no deben incluir:

password;
reset token;
MFA secret;
recovery code.
53. Recovery Security

Recovery nunca debe convertirse en el camino más débil del sistema.

Principio:

La recuperación de una cuenta debe ser al menos tan segura como la autenticación que reemplaza.

No deberá existir un mecanismo como:

"I forgot my password"
        ↓
Email only
        ↓
Disable MFA

sin una justificación explícita de Security Policy.

54. Password + MFA Relationship

Password y MFA son capas distintas.

Password
   +
MFA
   ↓
Authentication

Comprometer una password no debe automáticamente:

deshabilitar MFA;
eliminar factores;
eliminar recovery protections.
55. Account Lifecycle Integration

Password operations deben respetar el estado de la cuenta.

PENDING

Puede establecer contraseña durante activation cuando corresponda.

ACTIVE

Operaciones normales.

SUSPENDED

Password change/reset puede estar restringido o requerir recovery workflow.

DISABLED

No debe poder utilizarse normalmente.

Las políticas exactas deberán ser centralizadas.

56. Lockout Integration

Password authentication deberá integrarse con:

ADR-IDENTITY-032

y:

ADR-IDENTITY-033

Por ejemplo:

Failed Password
      ↓
Rate Limit
      ↓
Risk Evaluation
      ↓
Temporary Lockout
      ↓
Possible Suspension

No se debe implementar un lockout paralelo e independiente dentro del password service.

57. High Availability

Password verification debe funcionar en múltiples API instances.

El password hash no deberá depender de memoria local.

La pérdida de una instancia no debe perder:

credentials;
credential state;
password history.

PostgreSQL seguirá siendo la fuente de verdad.

58. Performance

Argon2id es intencionalmente costoso.

Por ello deberán realizarse pruebas para mantener los objetivos:

API P95 <300 ms;
Authentication P95 <500 ms.

La configuración deberá equilibrar:

Security
   ↕
CPU
   ↕
Memory
   ↕
Latency

No se deberá reducir la seguridad criptográfica simplemente para alcanzar una métrica de rendimiento sin una decisión explícita.

59. Resource Exhaustion Protection

Argon2id puede ser utilizado abusivamente para consumir CPU/memoria.

Por ello Authentication deberá combinar:

Rate Limiting;
connection limits;
concurrency controls;
abuse detection;
infrastructure scaling.

Esto protege contra:

Password Hashing DoS

60. Backup & Recovery

Los password hashes forman parte de los datos críticos de Identity y deben recuperarse mediante:

PostgreSQL backup;
PITR;
Disaster Recovery.

Pero nunca deben incluir:

plaintext passwords;
reset tokens activos innecesariamente;
secretos temporales.

Recovery deberá preservar la integridad de credential state.

61. Security During Restore

Después de restore deberá verificarse:

hashes presentes;
algoritmo correcto;
parámetros;
password history;
password change state;
compromised state;
reset token state.

Una restauración no deberá reactivar una credencial que estaba invalidada en el recovery point elegido.

62. Testing

Se deberán implementar pruebas para:

Hashing
Argon2id;
salt;
verification;
rehash.
Password Policy
valid password;
invalid password;
breached password;
history;
policy changes.
Authentication
correct password;
incorrect password;
suspended account;
disabled account.
Change
current password;
new password;
MFA;
sessions.
Reset
request;
enumeration;
expiration;
reuse;
concurrency.
Security
brute force;
credential stuffing;
timing attacks;
token guessing;
recovery abuse.
63. Consecuencias positivas

Esta decisión proporciona:

almacenamiento seguro de passwords;
Argon2id;
lifecycle completo;
protección contra credential abuse;
password reset controlado;
integración con MFA;
integración con sessions;
integración con refresh tokens;
protección contra compromised credentials;
auditabilidad;
observabilidad;
recuperación segura.
64. Consecuencias negativas

Introduce:

mayor costo computacional de hashing;
complejidad de password lifecycle;
necesidad de rehash;
password history;
recovery workflows;
mayor complejidad de testing;
necesidad de tuning de Argon2id;
necesidad de coordinación con MFA y sessions.
65. Alternativas consideradas
A. SHA-256 directo

Rechazada.

No está diseñado para password hashing resistente a ataques offline.

B. bcrypt

No adoptada como mecanismo principal.

Puede ser válido en sistemas existentes, pero EVOXA ha establecido Argon2id como dirección principal.

C. Encryption reversible

Rechazada.

La contraseña no necesita ser recuperada.

D. Passwords + MFA sin password lifecycle formal

Rechazada.

MFA no elimina la necesidad de proteger correctamente las credenciales.

E. Argon2id + Centralized Credential Lifecycle

Adoptada.

Es consistente con la estrategia de seguridad de Identity.

66. No negociables
Passwords nunca se almacenan en plaintext.
Passwords nunca se almacenan mediante cifrado reversible como sustituto del hashing.
Argon2id será el algoritmo principal.
Cada password utilizará salt único.
Passwords nunca aparecerán en logs.
Passwords nunca aparecerán en events.
Passwords nunca aparecerán en traces.
Passwords nunca deberán entrar en AI context.
Password reset deberá ser de corta duración y no reutilizable.
Password reset no podrá utilizarse para bypass de MFA.
Password reset deberá tener rate limiting.
Password reset deberá proteger contra enumeration.
Credential compromise deberá poder invalidar la credencial.
Password changes deberán auditarse.
Security-sensitive changes deberán generar Security Events.
Sessions y Refresh Tokens deberán evaluarse después de cambios críticos.
Account state deberá respetarse.
Password history, si se implementa, deberá almacenar únicamente representaciones protegidas.
Hash parameters deberán poder evolucionar.
Rehash deberá ser posible.
Password verification debe funcionar en múltiples API instances.
Argon2id deberá protegerse contra resource exhaustion.
Recovery debe ser al menos tan seguro como el mecanismo que reemplaza.
AI Agents no podrán acceder a passwords ni saltarse credential policies.
Restore debe preservar credential state.
Failover no debe modificar el estado de las credenciales.
67. Traceability
Referencia	Relación
Blueprint — Security Architecture	Secure by Design
ESP-0001 Identity	Credential Management
ADR-IDENTITY-005	Security Policy
ADR-IDENTITY-008	Cryptography / KMS
ADR-IDENTITY-010	Testing
ADR-IDENTITY-022	Idempotency
ADR-IDENTITY-023	MFA Factor Strategy
ADR-IDENTITY-024	MFA Challenge / Recovery
ADR-IDENTITY-025	Refresh Token Security
ADR-IDENTITY-026	API Idempotency
ADR-IDENTITY-028	Observability
ADR-IDENTITY-029	Disaster Recovery
ADR-IDENTITY-031	High Availability
ADR-IDENTITY-032	Rate Limiting / Abuse
ADR-IDENTITY-033	Account Lifecycle
ADR-IDENTITY-034	Password Security & Credential Lifecycle
68. Dependencias

Este ADR depende de:

Authentication;
User lifecycle;
Security Policy Engine;
MFA;
Sessions;
Refresh Tokens;
Rate Limiting;
Abuse Detection;
PostgreSQL;
KMS / Secret Manager;
Audit;
Security Events;
Outbox;
Observability.
69. Decisiones pendientes

Quedan pendientes:

parámetros exactos de Argon2id;
minimum password length;
maximum password length;
complexity policy;
breached password provider/strategy;
password history count;
password reset token format;
password reset token hashing;
password reset TTL;
reset attempt limits;
password change session policy;
password reset session policy;
credential compromise workflow;
forced password change model;
temporary credential strategy;
administrative reset workflow;
legacy password migration;
hash versioning;
rehash trigger;
Argon2id performance thresholds;
resource exhaustion controls;
credential metadata schema;
password security event catalog;
notification strategy;
tenant-specific password policies;
exact recovery/MFA interaction.
70. Acceptance Criteria

El ADR se considerará implementado cuando:

 passwords estén almacenados únicamente como hashes;
 Argon2id esté implementado;
 salts sean únicos;
 password verification sea segura;
 exista password policy;
 exista password change;
 exista password reset;
 reset tokens sean seguros;
 reset tokens expiren;
 reset tokens sean de uso único;
 exista protección contra reset abuse;
 exista anti-enumeration;
 exista password history cuando la policy lo requiera;
 exista breached-password protection cuando corresponda;
 exista hash rehashing;
 exista credential compromise workflow;
 exista forced password change;
 exista integración con MFA;
 exista integración con Sessions;
 exista integración con Refresh Tokens;
 exista integración con Account Lifecycle;
 exista Rate Limiting;
 exista Abuse Detection;
 exista Audit;
 existan Security Events;
 passwords nunca aparezcan en observability;
 passwords nunca aparezcan en AI context;
 password operations sean concurrency-safe;
 exista protección contra hashing resource exhaustion;
 el sistema funcione en múltiples API instances;
 password state sobreviva a failover;
 password state sobreviva a Disaster Recovery;
 existan pruebas de seguridad;
 existan pruebas de performance;
 existan pruebas de recovery.
71. Siguiente ADR

El siguiente paso lógico sería:

ADR-IDENTITY-035 — Identity Account Recovery & Credential Recovery Strategy

Este ADR debería separar formalmente Password Reset, Account Recovery, MFA Recovery, Recovery Codes, Trusted Devices, Identity Verification y Administrative Recovery, definiendo qué nivel de evidencia se requiere para recuperar una cuenta comprometida y evitando que Recovery se convierta en el punto más débil de Identity.
