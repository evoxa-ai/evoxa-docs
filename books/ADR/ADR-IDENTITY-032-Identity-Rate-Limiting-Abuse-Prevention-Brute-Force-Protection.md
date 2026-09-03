ADR-IDENTITY-032 — Identity Rate Limiting, Abuse Prevention & Brute-Force Protection

Estado: Proposed
Versión: 1.0
Fecha: 2026-09-03
Dominio: Identity
Prioridad: Critical
Tipo: Architecture Decision Record

1. Contexto

Identity expone operaciones altamente sensibles que pueden ser objetivo de abuso automatizado, ataques de fuerza bruta, credential stuffing, enumeración de cuentas, abuso de MFA, abuso de recuperación de contraseña y ataques distribuidos.

Entre las operaciones de mayor riesgo se encuentran:

Login
Refresh Token
MFA Challenge
MFA Enrollment
Password Reset
Password Reset Request
Account Recovery
API Token operations
Session operations
Role/Permission administration
Security Policy administration
Sensitive operations protegidas por step-up authentication.

ADR-IDENTITY-005 estableció que Authorization utiliza:

RBAC + ABAC + Tenant Isolation + Resource Ownership + Security Policy Engine

y ADR-IDENTITY-011 / 031 establecieron que Identity debe ser altamente disponible.

Por lo tanto, Rate Limiting no debe diseñarse como una simple restricción por IP.

Debe considerar simultáneamente:

usuario;
organización;
endpoint;
IP;
sesión;
dispositivo;
factor MFA;
operación;
riesgo;
tenant;
contexto de seguridad.
2. Problema

Un atacante puede intentar:

Credential Stuffing
        ↓
Login
        ↓
Thousands of attempts

o:

MFA Brute Force
        ↓
Challenge
        ↓
Repeated codes

o:

Password Reset Abuse
        ↓
Repeated requests
        ↓
Email/SMS flooding

También puede distribuir los ataques:

IP 1 ─┐
IP 2 ─┤
IP 3 ─┤──→ Same Account
IP 4 ─┤
IP 5 ─┘

Por lo tanto:

IP-based rate limiting por sí solo no es suficiente.

Además, una protección demasiado agresiva puede producir:

bloqueo injustificado de usuarios legítimos;
account lockout abuse;
problemas para organizaciones grandes;
pérdida de disponibilidad;
aumento de falsos positivos.
3. Decisión

EVOXA Identity adoptará una estrategia de:

Multi-Dimensional Rate Limiting + Abuse Detection + Progressive Protection + Security Policy Enforcement

La protección será distribuida en varias capas.

Request
   ↓
Edge / Network Protection
   ↓
IP / Client Rate Limit
   ↓
Identity Rate Limit
   ↓
Security Policy Engine
   ↓
Authentication / Authorization
   ↓
Abuse Detection
   ↓
Audit / Security Event

No existirá un único mecanismo de protección.

4. Principios fundamentales

La estrategia seguirá estos principios:

Defense in Depth
Fail Closed para controles de seguridad críticos
Tenant-aware
User-aware
Operation-aware
Risk-aware
Distributed
Observable
Privacy-aware
Resistant to distributed attacks
No secrets in telemetry
No bypass mediante AI Agents
5. Rate Limiting vs Abuse Prevention

Se diferenciarán dos conceptos.

Rate Limiting

Controla:

¿Cuántas solicitudes puede realizar un actor durante un periodo determinado?

Abuse Prevention

Evalúa:

¿El comportamiento observado parece abusivo aunque todavía no haya superado un límite simple?

Ejemplo:

100 requests / minute

puede estar dentro de un límite.

Pero:

100 failed logins
against 100 different accounts
from same device

puede ser claramente un patrón de ataque.

Por ello se necesitan ambos mecanismos.

6. Dimensiones de Rate Limiting

El sistema podrá aplicar límites por:

IP

Protege contra:

ataques desde una fuente concreta;
flooding;
scanning.
User

Protege una cuenta individual.

Organization / Tenant

Evita que una organización genere tráfico abusivo.

Endpoint

Protege operaciones específicas.

Operation

Permite límites diferentes para:

login;
refresh;
MFA;
password reset.
Session

Controla abuso asociado a una sesión.

Device

Puede ayudar a detectar automatización.

Risk Context

Permite aplicar controles más estrictos a solicitudes de mayor riesgo.

7. No confiar únicamente en IP

Una IP no representa necesariamente un usuario.

Puede existir:

1 IP
 ↓
1000 legitimate users

o:

1000 IPs
 ↓
1 attacker

Por eso las decisiones deben combinar múltiples señales.

No se establecerá un algoritmo exacto de scoring en este ADR.

8. Sensitive Endpoint Classes

Identity clasificará endpoints por riesgo.

Clase Critical
Login
MFA verification
Password reset confirmation
Account recovery
Token operations sensibles
Security policy changes
MFA factor changes.
Clase High
Refresh
Password reset request
Session creation
API token creation
sensitive profile/security operations.
Clase Medium
Profile updates
Session listing
standard administrative operations.
Clase Low
operaciones de lectura de bajo riesgo, cuando corresponda.

Los límites exactos quedan pendientes.

9. Login Protection

Login será una de las operaciones más protegidas.

Se deberán considerar simultáneamente:

IP
+
Account
+
Device
+
Tenant
+
Failure History
+
Risk Context

El objetivo es detectar:

brute force;
credential stuffing;
password spraying;
automated login;
distributed attacks.
10. Brute-Force Protection

El sistema deberá detectar múltiples intentos fallidos.

Conceptualmente:

Failed Login
      ↓
Failure Counter
      ↓
Threshold
      ↓
Progressive Protection

La protección podrá evolucionar:

Normal
 ↓
Warning / Increased Delay
 ↓
Challenge
 ↓
Temporary Block
 ↓
Security Event

La respuesta exacta dependerá de la política de seguridad.

11. Evitar Account Lockout Abuse

No se adoptará un modelo ingenuo donde cualquiera pueda bloquear permanentemente una cuenta simplemente enviando contraseñas incorrectas.

Ejemplo que debe evitarse:

Attacker
   ↓
100 failed attempts
   ↓
Victim account permanently locked

La protección debe equilibrar:

seguridad;
disponibilidad;
experiencia del usuario.

Podrán utilizarse mecanismos como:

progressive delays;
risk-based challenge;
temporary restrictions;
IP/device reputation;
MFA step-up;
security alerts.
12. Credential Stuffing

Credential stuffing utiliza credenciales obtenidas de otras fuentes.

Patrón:

IP A → user1
IP B → user2
IP C → user3
...

La protección deberá observar patrones agregados, no solamente fallos por usuario.

Se deberán poder detectar señales como:

muchos usuarios atacados;
passwords inválidas repetitivas;
alta tasa de failures;
distribución anormal de IPs;
dispositivos repetidos;
patrones temporales.
13. Password Spraying

Password spraying consiste en probar una misma contraseña o conjunto pequeño contra muchas cuentas.

La estrategia deberá detectar:

Password X
   ↓
User A
User B
User C
User D
...

La protección debe considerar:

account distribution;
failure rate;
IP;
device;
tenant;
temporal patterns.
14. MFA Rate Limiting

MFA tendrá límites específicos.

Se controlará:

challenges;
verification attempts;
resend operations;
factor enrollment;
factor changes;
recovery.

Los intentos incorrectos deberán integrarse con:

ADR-IDENTITY-024 — MFA Challenge TTL, Attempts & Recovery

Cuando se alcance el máximo de intentos del challenge:

Challenge
   ↓
FAILED
   ↓
New Challenge Required

No se debe permitir continuar indefinidamente con el mismo challenge.

15. MFA Brute Force

El sistema deberá protegerse contra:

OTP brute force;
repeated verification;
challenge enumeration;
recovery code guessing;
factor abuse.

Los controles podrán aplicarse a:

challenge;
factor;
user;
device;
IP;
tenant.
16. MFA Provider Abuse

Cuando MFA utilice proveedores externos como:

SMS;
email;
push;

deberán existir límites para evitar:

SMS bombing;
email flooding;
provider cost abuse;
challenge spam.

Ejemplo:

Attacker
 ↓
Request MFA
 ↓
Request MFA
 ↓
Request MFA
 ↓
Provider Cost

El Rate Limiting debe aplicarse antes de generar repetidamente operaciones costosas.

17. Password Reset Protection

Password Reset Request debe estar especialmente protegido.

El sistema deberá limitar:

requests por IP;
requests por account identifier;
requests por device;
requests por tenant;
frecuencia de envío.

La respuesta pública debe evitar account enumeration.

Ejemplo conceptual:

Request password reset
        ↓
Generic Response

No debe revelar innecesariamente:

"Este email no existe."

18. Account Recovery

Account Recovery será tratado como una operación de alto riesgo.

Debe aplicar:

strict rate limiting;
identity verification;
MFA/recovery policies;
audit;
security events;
progressive protection.

No deberá existir:

bypass automático de MFA por superar un límite o mediante una operación administrativa no autorizada.

19. Refresh Token Protection

Refresh Token requests también deberán estar protegidos.

Se aplicarán controles contra:

automated refresh abuse;
token replay;
token spraying;
distributed attacks.

Sin embargo, no debe implementarse un límite tan agresivo que interfiera con el comportamiento normal de los clientes.

La detección de:

Refresh Token Reuse

seguirá siendo responsabilidad específica del lifecycle definido en ADR-IDENTITY-009 / 025.

Rate Limiting es complementario, no sustituto de reuse detection.

20. API Token Protection

Las operaciones relacionadas con API Tokens deberán tener protección elevada:

creation;
rotation;
revoke;
listing;
sensitive configuration.

Especialmente:

Create API Token

deberá requerir:

authorization;
tenant context;
policy;
posiblemente step-up/MFA;
rate limiting;
audit.
21. Administrative Abuse Prevention

Las operaciones administrativas deberán estar protegidas.

Ejemplos:

crear usuarios;
cambiar roles;
cambiar permisos;
modificar policies;
revocar sessions;
revocar tokens;
cambiar MFA;
modificar tenant security configuration.

Rate limiting no reemplaza Authorization.

La secuencia correcta continúa siendo:

Authenticate
   ↓
Authorize
   ↓
Policy
   ↓
Rate Limit
   ↓
Execute

La implementación exacta del orden puede variar según endpoint, pero nunca deberá permitir que Rate Limiting sea utilizado como sustituto de Authorization.

22. Distributed Rate Limiting

Dado que Identity será stateless y tendrá múltiples instancias, el rate limiting no puede depender exclusivamente de memoria local.

Ejemplo:

Request
   ↓
Load Balancer
   ├── API-01
   ├── API-02
   └── API-03

Si cada instancia mantiene un contador independiente:

API-01 → 10
API-02 → 10
API-03 → 10

un atacante puede superar accidentalmente el límite global.

Por ello, los límites compartidos deberán utilizar un mecanismo distribuido.

Redis es un candidato natural, pero:

Redis no será source of truth para datos transaccionales de Identity.

23. Redis Failure

El diseño debe definir comportamiento seguro ante Redis unavailable.

No se permitirá que:

Redis unavailable
       ↓
Rate Limiting disabled
       ↓
Unlimited login attempts

El sistema deberá tener una estrategia de fallback segura.

La estrategia exacta queda pendiente y podrá incluir:

edge protection;
local emergency limits;
temporary stricter controls;
degraded mode.
24. Rate Limit Algorithms

La implementación podrá utilizar algoritmos apropiados como:

Token Bucket;
Leaky Bucket;
Fixed Window;
Sliding Window;
Sliding Window Counter.

No se establece un algoritmo único obligatorio en este ADR.

La elección dependerá de:

endpoint;
precisión requerida;
costo;
distribución;
volumen;
infraestructura.
25. Burst Handling

El sistema deberá diferenciar entre:

Normal burst

Por ejemplo:

Application
→ multiple concurrent requests

y:

Abuse burst
Automated attacker
→ hundreds of security-sensitive requests

Los límites deberán permitir comportamiento legítimo sin abrir una ventana excesiva para ataques.

26. Progressive Rate Limiting

Para operaciones sensibles se favorecerá una protección progresiva.

Conceptualmente:

Normal
 ↓
Rate Limit Warning
 ↓
Delay
 ↓
Challenge
 ↓
Temporary Block
 ↓
Security Event

La progresión dependerá de:

número de fallos;
historial;
riesgo;
endpoint;
tenant policy;
contexto.
27. Risk-Based Protection

El Rate Limiting podrá integrarse con el Security Policy Engine.

Ejemplo:

Normal request
→ standard limit

Suspicious request
→ stricter limit

High-risk request
→ challenge / MFA

Confirmed abuse
→ deny / temporary block

La decisión de seguridad deberá quedar gobernada por:

Security Policy Engine

y no por lógica arbitraria dispersa en cada endpoint.

28. Security Policy Engine Integration

El Policy Engine podrá recibir contexto como:

user;
tenant;
endpoint;
operation;
IP;
device;
authentication method;
MFA state;
recent failures;
risk signals.

Y devolver decisiones como:

ALLOW;
DENY;
CHALLENGE;
REAUTHENTICATE;
REQUIRE_MFA;
REVOKE.

Rate Limiting podrá aportar señales, pero no deberá reemplazar el Policy Engine.

29. Abuse Detection

Además de límites numéricos se deberán detectar patrones.

Ejemplos:

Pattern A
1 account
100 failed logins
Pattern B
1000 accounts
1 failed login each
Pattern C
1 device
50 accounts
Pattern D
1 IP range
high MFA requests
Pattern E
successful login
immediately followed by
abnormal token operations

Estos patrones pueden alimentar Security Policy y Security Events.

30. Reputation Signals

La arquitectura podrá incorporar señales de reputación como:

IP history;
device history;
account history;
tenant context;
previous abuse;
geographic anomalies;
authentication anomalies.

No se establece en este ADR un proveedor externo específico.

31. Privacy

La estrategia deberá respetar:

minimización de datos;
retención limitada;
protección de PII;
tenant isolation.

No se deben almacenar más datos de comportamiento de los necesarios para seguridad.

Las señales utilizadas para abuse detection deben tener:

propósito definido;
acceso controlado;
retención apropiada.
32. Observability

Rate Limiting debe integrarse con la observabilidad definida en ADR-IDENTITY-028.

Métricas conceptuales:

requests allowed;
requests throttled;
requests denied;
login failures;
MFA failures;
password reset requests;
refresh attempts;
suspicious patterns;
temporary blocks;
abuse detections;
challenge escalations.
33. Security Events

Eventos de seguridad podrán generarse para:

brute-force detected;
credential stuffing detected;
password spraying detected;
MFA brute force;
account abuse;
excessive reset requests;
distributed attack;
suspicious token behavior.

Los eventos deben contener contexto suficiente para investigación, pero nunca:

password;
password hash;
token;
MFA secret;
recovery code;
API secret.
34. Audit

Las acciones administrativas relacionadas con controles de abuso deberán auditarse.

Ejemplos:

cambiar límites;
modificar policies;
desbloquear manualmente;
cambiar security thresholds;
modificar tenant abuse configuration.
35. Tenant-Specific Policies

Las organizaciones podrán eventualmente tener políticas diferentes según sus requisitos.

Por ejemplo:

Organization A
→ strict authentication policy

Organization B
→ standard policy

Pero ninguna configuración de tenant podrá reducir controles globales de seguridad por debajo del mínimo establecido por la plataforma.

Precedencia conceptual:

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

y:

DENY > ALLOW

36. Global Security Minimums

El sistema deberá mantener límites mínimos globales para operaciones críticas.

Un tenant no podrá configurar:

Login attempts = unlimited

si esto contradice una política de seguridad de plataforma.

37. Response Codes

Cuando corresponda, la API podrá devolver:

HTTP 429 Too Many Requests

La respuesta deberá utilizar el modelo estándar de errores de EVOXA.

Conceptualmente:

{
  "error": {
    "code": "RATE_LIMIT_EXCEEDED",
    "message": "Too many requests",
    "correlation_id": "..."
  }
}

No deberá revelar información sensible sobre:

cuentas;
mecanismos internos;
thresholds internos;
scoring de riesgo.
38. Retry-After

Cuando sea apropiado, la API podrá indicar:

Retry-After

para permitir que clientes legítimos sepan cuándo reintentar.

Los valores exactos dependerán de la estrategia de endpoint.

39. Anti-Enumeration

Rate limiting debe contribuir a reducir:

account enumeration;
organization enumeration;
permission enumeration;
security state enumeration.

Sin embargo, no será el único mecanismo.

Las respuestas de endpoints sensibles deberán diseñarse para minimizar diferencias observables innecesarias.

40. Availability Considerations

La protección contra abuso no debe convertirse en un nuevo Single Point of Failure.

Debe soportar:

múltiples API instances;
failover;
Redis failover;
deployment;
scaling.

Durante incidentes de alta carga:

La protección debe reducir presión sobre Identity, no aumentar el problema.

41. DDoS Boundary

DDoS volumétrico debe ser tratado principalmente en capas externas:

Internet
 ↓
CDN / Edge / DDoS Protection
 ↓
Load Balancer
 ↓
Identity

Identity Rate Limiting no sustituirá:

network DDoS protection;
WAF;
CDN;
cloud edge protection.

El proveedor concreto queda pendiente.

42. Internal Service Abuse

El abuso no necesariamente proviene de Internet.

También puede provenir de:

service accounts;
internal services;
AI Agents;
compromised applications.

Por ello, límites podrán aplicarse también a:

service identity;
client identity;
API token;
application;
tenant.
43. AI Agent Protection

Los AI Agents estarán sujetos a los mismos controles.

No podrán:

saltarse rate limits;
generar unlimited authentication attempts;
desactivar abuse detection;
cambiar security thresholds sin autorización;
acceder a mecanismos internos de bypass.

Las acciones de AI Agents deberán identificarse mediante:

actor = AI_AGENT

cuando corresponda.

44. False Positive Management

Un sistema de protección demasiado agresivo puede bloquear usuarios legítimos.

Se deberán monitorizar:

false positives;
blocked legitimate users;
challenge rates;
recovery rates;
support incidents.

Las políticas podrán ajustarse utilizando evidencia.

45. Security Response

Cuando se detecte abuso confirmado, el sistema podrá:

throttle;
deny;
challenge;
require MFA;
require reauthentication;
revoke session;
revoke token family;
generar security event;
alertar.

La acción concreta dependerá del nivel de riesgo y de la Security Policy.

46. Integration with Session Security

Cuando exista evidencia de compromiso de una sesión:

Abuse Detection
      ↓
Risk Assessment
      ↓
Security Policy
      ↓
Session Revocation

Esto se relaciona con:

ADR-IDENTITY-008 / 025 / 031

y el lifecycle de sessions y refresh tokens.

47. Testing

El sistema deberá probar:

Functional
limit reached;
reset;
windows;
different actors.
Security
brute force;
credential stuffing;
password spraying;
MFA abuse;
reset abuse;
token abuse.
Distributed
multiple API instances;
multiple IPs;
distributed attacks.
Resilience
Redis unavailable;
API failover;
PostgreSQL failover;
deployment.
Performance
high request volume;
concurrent authentication;
latency impact.
48. Abuse Test Scenarios

Ejemplos:

Scenario 1
1 IP
→ 10,000 login attempts
Scenario 2
1 account
→ distributed brute force
Scenario 3
1 password
→ thousands of accounts
Scenario 4
1 device
→ multiple MFA challenges
Scenario 5
multiple API instances
→ global rate limit
Scenario 6
Redis failure
→ security controls remain safe
49. Performance Requirements

El Rate Limiting no debe comprometer significativamente los objetivos de rendimiento de Identity.

Debe mantenerse compatible con:

API P95 <300 ms;
Authentication P95 <500 ms.

La implementación debe evitar:

múltiples round trips innecesarios;
locks globales;
hot keys;
alta cardinalidad no controlada;
dependencia excesiva de servicios externos.

Los valores exactos deberán validarse mediante pruebas de performance.

50. Configuration Management

Los límites deberán ser configurables mediante configuración/policies controladas.

No deben estar dispersos como constantes hardcoded por todo el código.

La configuración deberá soportar:

versioning;
audit;
validation;
rollback;
tenant scope cuando corresponda.

Cambios críticos deberán requerir autorización adecuada.

51. Security Configuration Protection

Modificar los límites de seguridad será una operación sensible.

Por lo tanto, deberá aplicarse:

RBAC;
ABAC;
tenant isolation;
Security Policy;
audit;
posiblemente MFA/step-up.

Un administrador no debe poder reducir controles críticos simplemente porque tenga acceso administrativo genérico.

52. Configuration Rollback

Una configuración incorrecta puede producir:

bloqueo masivo;
exceso de tráfico;
bypass de protección.

Por ello debe existir capacidad de:

detectar configuración incorrecta;
revertir;
auditar;
restaurar configuración conocida.
53. Consecuencias positivas

Esta decisión proporciona:

protección contra brute force;
protección contra credential stuffing;
protección contra password spraying;
protección MFA;
protección de Password Reset;
protección de Refresh;
defensa distribuida;
tenant-aware security;
integración con Policy Engine;
mejor observabilidad;
reducción de abuso automatizado;
mayor resiliencia.
54. Consecuencias negativas

Introduce:

complejidad distribuida;
dependencia de Redis u otro mecanismo compartido;
mayor consumo de infraestructura;
riesgo de falsos positivos;
necesidad de tuning;
complejidad de abuse detection;
necesidad de monitoreo;
necesidad de pruebas de carga;
necesidad de mantener políticas de seguridad.
55. Alternativas consideradas
A. Rate limiting únicamente por IP

Rechazada.

No protege adecuadamente contra ataques distribuidos ni diferencia usuarios legítimos detrás de una misma IP.

B. Account Lockout permanente

Rechazada.

Facilita Account Lockout Abuse y puede afectar disponibilidad de usuarios legítimos.

C. Rate limiting únicamente en API Gateway

Rechazada como estrategia completa.

Es útil para protección perimetral, pero no conoce suficientemente:

user;
session;
MFA;
tenant;
authorization;
risk context.
D. Rate Limiting + Abuse Detection + Policy Engine

Adoptada.

Proporciona defensa por capas y permite adaptar la protección al contexto.

56. No negociables
Login debe tener protección contra brute force.
Debe existir protección contra credential stuffing.
Debe existir protección contra password spraying.
MFA debe tener rate limiting.
Password Reset debe tener rate limiting.
Account Recovery debe tener protección reforzada.
Refresh debe estar protegido.
API Token operations sensibles deben estar protegidas.
La protección no dependerá únicamente de IP.
Rate limiting debe funcionar en múltiples API instances.
Redis no será source of truth transaccional.
Redis failure no podrá producir un bypass de seguridad.
Authentication nunca debe fail-open.
Authorization nunca debe fail-open.
Tenant isolation debe preservarse.
Account lockout permanente no será el mecanismo principal.
Security Policy Engine gobernará decisiones contextuales.
Los controles deben ser observables.
Los cambios de configuración deben auditarse.
No deben registrarse secretos.
DDoS volumétrico debe protegerse también en edge/network layers.
AI Agents están sujetos a los mismos controles.
Los límites deben probarse bajo carga.
Los falsos positivos deben monitorizarse.
Rate Limiting no sustituye Authentication, Authorization, MFA ni Token Reuse Detection.
57. Traceability
Referencia	Relación
Blueprint — Security Architecture	Defense in Depth / Secure by Design
ESP-0001 Identity	Seguridad de Identity
ADR-IDENTITY-005	Authorization / Policy Engine
ADR-IDENTITY-008	Cryptographic Security
ADR-IDENTITY-009	Token Storage
ADR-IDENTITY-022	Consumer Idempotency
ADR-IDENTITY-024	MFA Challenge Attempts
ADR-IDENTITY-025	Refresh Token Security
ADR-IDENTITY-026	API Idempotency
ADR-IDENTITY-028	Observability
ADR-IDENTITY-031	High Availability & Failover
ADR-IDENTITY-032	Rate Limiting & Abuse Prevention
58. Dependencias

Este ADR depende de:

API Gateway / Edge;
Redis o mecanismo distribuido equivalente;
Security Policy Engine;
Authentication;
MFA;
Sessions;
Refresh Tokens;
Audit;
Security Events;
Observability;
Load Balancing;
HA / Failover.
59. Decisiones pendientes

Quedan pendientes:

algoritmo de rate limiting por endpoint;
límites exactos por endpoint;
ventanas de tiempo;
burst limits;
límites por IP;
límites por usuario;
límites por tenant;
límites por device;
límites para service accounts;
límites para AI Agents;
Redis architecture;
Redis fallback;
abuse scoring;
risk scoring;
IP reputation;
device reputation;
progressive delay strategy;
temporary blocking duration;
challenge escalation;
integration exacta con WAF;
integración con DDoS protection;
false-positive thresholds;
alert thresholds;
configuration management;
emergency override procedure;
abuse response automation;
retention de abuse signals;
privacy controls.
60. Acceptance Criteria

El ADR se considerará implementado cuando:

 Login tenga rate limiting.
 exista brute-force protection.
 exista credential-stuffing detection.
 exista password-spraying detection.
 MFA tenga rate limiting.
 MFA tenga protección contra brute force.
 Password Reset esté protegido.
 Account Recovery esté protegido.
 Refresh esté protegido.
 API Token operations sensibles estén protegidas.
 exista rate limiting multidimensional.
 funcione entre múltiples API instances.
 Redis/mecanismo distribuido esté disponible.
 exista estrategia segura ante Redis failure.
 exista integración con Security Policy Engine.
 exista abuse detection.
 existan security events.
 existan métricas.
 exista protección contra account lockout abuse.
 exista anti-enumeration.
 exista edge/DDoS protection complementaria.
 tenant isolation sea preservado.
 AI Agents estén sujetos a los mismos controles.
 existan pruebas distribuidas.
 existan pruebas de carga.
 existan pruebas de failover.
 se midan false positives.
 los cambios de configuración estén auditados.
 no existan secretos en logs/events.
 los controles críticos fallen closed.
61. Siguiente ADR

El siguiente paso lógico de la arquitectura Identity sería:

ADR-IDENTITY-033 — Identity Account Lifecycle, Lockout & Suspicious Account Management

Este ADR debería definir con precisión qué ocurre con una cuenta cuando se detectan múltiples fallos, comportamiento sospechoso, compromiso potencial, suspensión administrativa o señales de riesgo, diferenciando correctamente:

PENDING → ACTIVE → SUSPENDED → DISABLED, los bloqueos temporales, desbloqueos, sesiones, refresh-token families, MFA, notificaciones de seguridad y acciones automáticas del Security Policy Engine.
