ADR-IDENTITY-027 — Runtime Configuration, Secrets & KMS

Estado: Proposed
Versión: 1.0
Fecha: 2026-09-03
Dominio: Identity / Platform / Security
Prioridad: Critical
Tipo: Architecture Decision Record

1. Título

ADR-IDENTITY-027 — Runtime Configuration, Secrets & KMS

Define la estrategia arquitectónica para gestionar en runtime:

configuración;
secretos;
credenciales;
claves criptográficas;
certificados;
configuración sensible;
JWT signing keys;
MFA secrets;
Refresh Token protection secrets;
integración con KMS/Secret Manager;
rotación;
acceso de servicios;
separación de ambientes;
auditoría;
recuperación;
seguridad operacional.

Este ADR complementa especialmente:

ADR-IDENTITY-008 — Cryptographic Key Management
ADR-IDENTITY-014 — Encryption & Secret Protection
ADR-IDENTITY-017 — Database Migration Strategy
ADR-IDENTITY-025 — Refresh Token Cryptography & Concurrency
ADR-IDENTITY-026 — API Pagination, Filtering & Idempotency
IS-IDENTITY-017 — Identity Runtime & Deployment
2. Contexto

El Blueprint de EVOXA establece una arquitectura:

Cloud-Ready;
Secure by Design;
Zero Trust;
Least Privilege;
Defense in Depth;
Observable;
preparada para despliegues reproducibles y escalables.

La configuración del runtime no debe depender del código fuente ni de valores secretos incluidos en las imágenes de despliegue.

La arquitectura debe diferenciar claramente:

Configuration
     ≠
Secret
     ≠
Cryptographic Key

Además, Identity utiliza información extremadamente sensible:

JWT Signing Keys
Refresh Token Protection
MFA Secrets
Database Credentials
External Provider Credentials
Session Infrastructure Credentials
KMS Credentials

Por lo tanto, se necesita una estrategia centralizada.

3. Problema

Una implementación incorrecta podría producir:

Secret
   ↓
Source Code
   ↓
Git
   ↓
Container Image
   ↓
Logs

o:

Production Secret
   ↓
Development Environment

Esto representa un riesgo crítico.

También existe el problema de rotación:

Old Key
   ↓
New Key

sin interrumpir necesariamente el servicio.

Por ello, Identity necesita separar:

configuración pública/no sensible;
secretos;
claves criptográficas;
mecanismo de distribución;
autorización de acceso;
rotación;
auditoría.
4. Decisión

EVOXA adoptará una arquitectura de Configuration Externalization + Secret Manager + KMS, donde:

la configuración se externaliza del código;
los secretos se almacenan en un Secret Manager;
las claves criptográficas se gestionan mediante KMS/HSM o mecanismo equivalente;
los containers no contienen secretos permanentes;
los servicios reciben únicamente los secretos necesarios;
el acceso se controla mediante identidad de workload y least privilege;
la rotación se realiza sin depender de cambios de código;
cada ambiente mantiene secretos separados.

Conceptualmente:

                    ┌────────────────────┐
                    │ Configuration      │
                    │ Management         │
                    └─────────┬──────────┘
                              │
                              ▼
                       Runtime Config
                              │
                              │
┌───────────────┐      ┌──────▼──────┐
│ Secret        │─────►│ Identity API │
│ Manager       │      └─────────────┘
└───────┬───────┘
        │
        ▼
┌───────────────┐
│ KMS / HSM      │
│ Key Management │
└───────────────┘
5. Clasificación

Toda configuración utilizada por Identity deberá clasificarse como mínimo en:

PUBLIC_CONFIGURATION
INTERNAL_CONFIGURATION
SENSITIVE_CONFIGURATION
SECRET
CRYPTOGRAPHIC_KEY

La clasificación determina:

almacenamiento;
acceso;
logging;
exposición;
rotación;
auditoría.
6. Configuration

Configuración no sensible puede incluir:

APP_ENV
LOG_LEVEL
SERVICE_NAME
API_VERSION
FEATURE_FLAGS
TIMEOUTS
DEFAULT_LIMITS

Estos valores no son necesariamente secretos.

Pueden distribuirse mediante mecanismos de configuración del runtime.

Sin embargo, la configuración no debe permitir introducir valores que comprometan controles de seguridad.

7. Sensitive Configuration

Algunos valores pueden no ser secretos estrictos, pero requieren protección.

Ejemplos:

internal endpoints
security thresholds
provider configuration
operational identifiers

El tratamiento concreto dependerá de su clasificación.

8. Secrets

Secretos incluyen conceptualmente:

DATABASE_PASSWORD
REDIS_CREDENTIAL
EXTERNAL_PROVIDER_CREDENTIAL
API_SECRET
WEBHOOK_SECRET
MFA_SECRET
REFRESH_TOKEN_PEPPER

Estos no deben almacenarse:

en Git;
en código;
en Dockerfiles;
en imágenes;
en documentación pública;
en logs.
9. Cryptographic Keys

Las claves criptográficas tienen una categoría superior.

Ejemplos:

JWT Signing Key
Encryption Key
Webhook Signing Key
MFA Protection Key
Refresh Token Protection Key

No deben tratarse simplemente como variables de entorno ordinarias cuando exista una infraestructura de KMS adecuada.

La gestión deberá centralizarse.

10. KMS

EVOXA utilizará conceptualmente un:

Key Management Service (KMS)

para administrar claves criptográficas.

El KMS deberá proporcionar, según las capacidades de la plataforma elegida:

generación;
almacenamiento protegido;
versionado;
rotación;
control de acceso;
auditoría;
revocación/deshabilitación;
lifecycle.

La tecnología concreta de KMS queda pendiente.

11. HSM

Cuando los requisitos de seguridad o cumplimiento lo justifiquen, KMS podrá utilizar capacidades respaldadas por HSM.

No se establece que todas las claves deban necesariamente residir en HSM desde el primer release.

La selección deberá basarse en:

riesgo;
compliance;
costo;
disponibilidad;
performance;
región.
12. Separación de claves

No se utilizará una única clave para todos los propósitos.

Conceptualmente:

JWT Signing Key
        ≠
Data Encryption Key
        ≠
Refresh Token Protection
        ≠
MFA Secret Protection
        ≠
Webhook Signing Key

Esto limita el impacto de un compromiso.

13. JWT Signing Keys

Los JWT de EVOXA utilizarán firma asimétrica conforme a las decisiones anteriores.

Conceptualmente:

Private Key
    │
    ▼
JWT Signing
    │
    ▼
Access Token

Public Key
    │
    ▼
Verification / JWKS

La clave privada:

no debe estar en Git;
no debe estar en una imagen;
no debe aparecer en logs;
debe estar protegida mediante KMS/Secret Manager o mecanismo equivalente.
14. Key ID — kid

Los JWT deberán poder identificar la versión de la clave mediante:

kid

Esto permite:

Old Key
kid = K1

New Key
kid = K2

durante una transición.

La validación deberá conocer las claves públicas activas correspondientes.

15. Rotación de JWT Keys

La rotación deberá poder realizarse sin requerir necesariamente downtime.

Conceptualmente:

K1 ACTIVE
      │
      ▼
K2 CREATED
      │
      ▼
K2 ACTIVE
      │
      ▼
K1 DEPRECATED
      │
      ▼
K1 RETIRED

Durante la transición puede ser necesario aceptar temporalmente tokens firmados con la clave anterior.

El período exacto queda pendiente.

16. Revocación de claves

Una clave podrá ser deshabilitada/revocada ante:

compromiso;
exposición;
incidente;
error de configuración;
política de seguridad.

La revocación debe integrarse con:

Identity;
JWKS;
Security Events;
Incident Response.
17. Refresh Token Secrets

La protección criptográfica definida en:

ADR-IDENTITY-025

deberá utilizar secretos gestionados externamente.

Por ejemplo:

Refresh Token
      +
Protection Secret
      ↓
Protected Representation

El secreto no pertenece al código de la aplicación.

18. MFA Secrets

Los secretos utilizados por factores MFA deben seguir:

ADR-IDENTITY-014

y esta arquitectura de runtime.

Ejemplo:

TOTP Secret
    ↓
Protected Storage

La aplicación debe obtener acceso solamente cuando sea necesario.

19. Database Credentials

Las credenciales de PostgreSQL no deben estar hardcodeadas.

Conceptualmente:

Identity API
    │
    ▼
Secret Manager
    │
    ▼
DB Credential
    │
    ▼
PostgreSQL

La aplicación no debe asumir que la credencial permanecerá estática durante toda su vida.

20. Redis Credentials

El mismo principio aplica a Redis.

Redis es infraestructura auxiliar y no debe requerir secretos embebidos en la imagen.

21. External Providers

Para:

Email;
SMS;
Push;
observabilidad;
otros proveedores;

las credenciales deberán gestionarse externamente.

Ejemplo:

Provider API Key
      ↓
Secret Manager
      ↓
Runtime

Nunca:

provider_api_key = "..."

dentro del código.

22. Workload Identity

La arquitectura deberá preferir identidad de workload sobre credenciales estáticas cuando la plataforma lo permita.

Conceptualmente:

Container / Pod
      │
      ▼
Workload Identity
      │
      ▼
Secret Manager / KMS

Esto evita distribuir credenciales permanentes innecesariamente.

La implementación concreta dependerá del runtime cloud elegido.

23. Least Privilege

Un servicio deberá acceder solamente a:

los secretos que necesita;
las claves que necesita;
las operaciones criptográficas que necesita.

Ejemplo:

Identity API
    ├── read JWT public configuration
    ├── sign JWT
    ├── access required secrets
    └── NOT administer all KMS keys

No deberá tener privilegios administrativos globales sobre KMS.

24. Separación de responsabilidades

La arquitectura favorecerá separación entre:

Application Runtime
Security Administration
Key Administration
Infrastructure Administration

El servicio que utiliza una clave no debe necesariamente poder:

eliminarla;
cambiar políticas;
administrar todas sus versiones;
acceder a todas las claves.
25. Runtime Injection

Los secretos deberán inyectarse al runtime mediante mecanismos seguros.

Posibilidades conceptuales:

Secret Manager
     ↓
Runtime Secret Injection
     ↓
Application

o mediante integración nativa del orchestrator.

La aplicación no debe requerir secretos almacenados en la imagen.

26. Environment Variables

Las variables de entorno podrán utilizarse para configuración y, cuando sea necesario, secretos temporales.

Sin embargo:

Las variables de entorno no deben considerarse por sí mismas un Secret Manager.

Especialmente en ambientes donde pueden quedar expuestas mediante:

debugging;
dumps;
procesos;
tooling;
observabilidad.

La preferencia será integración directa o segura con Secret Manager.

27. Docker Images

Las imágenes deben ser:

Immutable
Reproducible
Secret-free

Nunca:

Docker image
 └── .env.production

ni:

Docker image
 └── private_key.pem
28. Git

El repositorio no debe contener:

passwords;
tokens;
private keys;
API keys;
KMS credentials;
MFA secrets.

Debe existir secret scanning en CI/CD.

Esto complementa:

IS-IDENTITY-016 — Identity Testing & Quality

29. .env

Los archivos .env podrán utilizarse para desarrollo local cuando sea necesario.

Pero:

.env.production

no debe ser la estrategia de secret management de producción.

Los secretos reales de producción deberán residir en el mecanismo oficial de secrets.

30. Ambientes

Cada ambiente debe tener aislamiento de secretos:

Development
     ≠
QA
     ≠
Staging
     ≠
Production

Un secreto de Production no debe utilizarse desde Development.

31. Environment Isolation

También deben separarse:

KMS keys;
Secret Manager namespaces;
credentials;
databases;
Redis;
event infrastructure;

según el nivel de aislamiento requerido.

32. Configuration Versioning

La configuración importante deberá poder versionarse o auditarse.

Esto permite conocer:

What
When
Who
Why

cambió.

Los secretos no necesitan necesariamente exponerse en texto para lograr auditoría.

Puede auditarse:

secret_version
changed_at
actor
operation

sin registrar el valor.

33. Secret Versioning

Los secretos deberán soportar versiones cuando la plataforma lo permita.

Conceptualmente:

Secret
 ├── Version 1
 ├── Version 2
 └── Version 3

Esto facilita:

rotación;
rollback controlado;
transición;
troubleshooting.
34. Secret Rotation

La rotación debe diseñarse como lifecycle:

CURRENT
   ↓
NEW VERSION CREATED
   ↓
APPLICATION VALIDATES
   ↓
NEW VERSION ACTIVE
   ↓
OLD VERSION RETIRED

No se debe eliminar inmediatamente la versión anterior si eso pudiera interrumpir conexiones activas.

35. Zero Downtime Rotation

Cuando sea técnicamente posible:

Old Credential
+
New Credential

podrán coexistir durante una ventana controlada.

Después:

Old
 ↓
Retired

Esto es especialmente relevante para:

DB credentials;
JWT signing keys;
external providers.
36. Emergency Rotation

Debe existir un procedimiento para rotación inmediata ante compromiso.

Ejemplo:

Security Incident
       ↓
Disable compromised secret/key
       ↓
Generate replacement
       ↓
Deploy/activate replacement
       ↓
Revoke old
       ↓
Audit

La infraestructura debe permitir realizarlo sin modificar el código fuente.

37. Secret Revocation

Un secreto comprometido debe poder:

deshabilitarse;
reemplazarse;
auditarse;
invalidarse cuando sea posible.

El sistema debe soportar incident response.

38. Access Logging

El acceso a secretos y claves debe poder auditarse.

Debe registrarse información como:

actor/workload
resource
operation
timestamp
environment
result

sin registrar el secreto.

39. No Secret Logging

Está prohibido registrar:

password
access_token
refresh_token
mfa_secret
private_key
api_key
database_password

También debe existir redacción automática en:

logs;
traces;
exceptions;
metrics labels;
audit metadata.
40. Error Handling

Un error como:

Database authentication failed

no debe convertirse en:

password=SuperSecret123

Los errores deben ser sanitizados.

41. Configuration Validation

La aplicación debe validar la configuración al iniciar.

Ejemplo:

Application startup
       ↓
Load configuration
       ↓
Validate required values
       ↓
Validate security requirements
       ↓
Start service

Si falta una configuración crítica:

Startup Failure

es preferible a iniciar con valores inseguros.

42. No Unsafe Defaults

No deberán existir defaults como:

JWT_SECRET=secret
DB_PASSWORD=password
ENCRYPTION_KEY=test

en producción.

Los valores obligatorios deben requerirse explícitamente.

43. Fail Closed

Si una clave o secreto crítico no puede obtenerse:

Secret unavailable
      ↓
Fail Closed

No:

Secret unavailable
      ↓
Use development fallback

Esto es especialmente importante para:

JWT signing;
Refresh Token protection;
MFA;
database credentials.
44. Startup vs Runtime Secret Access

No todos los secretos necesitan cargarse al iniciar.

La estrategia preferida será:

cargar startup-critical secrets cuando sea necesario;
obtener dinámicamente secretos rotables cuando sea apropiado;
evitar mantener innecesariamente secretos en memoria.

La implementación deberá balancear:

Security
+
Performance
+
Availability
45. Cache de Secretos

Puede existir cache temporal de secretos para reducir latencia.

Pero:

debe existir TTL;
debe poder invalidarse;
no debe escribirse en logs;
debe protegerse en memoria;
no debe convertirse en almacenamiento permanente.
46. KMS Availability

KMS es infraestructura crítica, pero Identity no debe quedar innecesariamente dependiente de una llamada remota para cada operación.

Cuando sea seguro, las claves públicas y material no secreto podrán cachearse.

Las operaciones que requieren protección criptográfica deben diseñarse teniendo en cuenta disponibilidad y latencia de KMS.

47. JWT Verification

La verificación de JWT puede utilizar claves públicas cacheadas:

JWKS
 ↓
Cache
 ↓
JWT Verification

La aplicación deberá actualizar el conjunto cuando aparezca un kid desconocido o cuando corresponda según la política de refresh.

48. JWT Signing

Para firma de JWT, la arquitectura podrá utilizar:

Application
   ↓
KMS Sign Operation
   ↓
Signature

o material de clave privada protegido en runtime cuando esté justificado.

La decisión concreta dependerá de:

KMS seleccionado;
latencia;
throughput;
costos;
capacidades criptográficas.
49. Key Usage vs Key Administration

Debe diferenciarse:

Use Key

de:

Manage Key

Un servicio puede estar autorizado a:

SIGN

sin estar autorizado a:

DELETE
ROTATE
CHANGE_POLICY

Esto implementa least privilege.

50. KMS Key Lifecycle

Conceptualmente:

CREATED
   ↓
ACTIVE
   ↓
ROTATING
   ↓
DEPRECATED
   ↓
RETIRED

Los estados exactos dependerán de la tecnología seleccionada.

51. Backup y Recovery

Los mecanismos de backup deben considerar:

configuración;
secretos;
claves;
metadata;
versiones;
recovery procedures.

No se debe asumir que restaurar PostgreSQL es suficiente para restaurar Identity.

Por ejemplo:

Database restored
+
KMS keys unavailable
=
Identity not fully restored

Por ello, KMS/Secret Manager forman parte del disaster recovery.

Esto se conectará con:

ADR-IDENTITY-029 — Disaster Recovery & Backup

52. Multi-Region

Si EVOXA evoluciona hacia múltiples regiones, deberá definirse:

ubicación de secretos;
replicación;
disponibilidad de KMS;
key residency;
failover;
recuperación.

No se fija aquí una estrategia multi-region definitiva.

53. Tenant Secrets

Los secretos específicos de tenant no deberán mezclarse indiscriminadamente con secretos globales.

Si un futuro dominio requiere:

Tenant-specific integration credential

deberá existir una estrategia separada de:

Platform Secrets

y:

Tenant Secrets

La arquitectura de aislamiento será definida por el dominio correspondiente.

54. Tenant Isolation

Nunca debe ocurrir:

Tenant A
   ↓
secret lookup
   ↓
Tenant B secret

La autorización y el contexto tenant deberán controlar cualquier acceso a secretos específicos del tenant.

55. AI Agents

Los AI Agents no deberán recibir:

private keys;
database passwords;
JWT signing keys;
Refresh Token protection secrets;
MFA secrets;
KMS administrative credentials.

La IA solo podrá recibir información explícitamente autorizada y necesaria para una operación.

56. AI Tool Calls

Si un agente AI ejecuta una operación que requiere un secreto, la arquitectura preferida será:

AI Agent
   ↓
Authorized Tool
   ↓
Backend
   ↓
Secret Manager / KMS

No:

Secret Manager
   ↓
AI context

Esto evita convertir el contexto de IA en un almacén de secretos.

57. Observabilidad

Las métricas deberán identificar problemas como:

secret_fetch_failure
kms_operation_failure
key_rotation_failure
secret_rotation_failure
configuration_validation_failure

Sin incluir valores secretos.

58. Alertas

Deben evaluarse alertas para:

accesos inesperados a secretos;
uso de claves deprecated;
fallos de KMS;
múltiples fallos de autenticación contra Secret Manager;
intentos de acceso no autorizado;
rotaciones fallidas;
secreto próximo a expiración cuando aplique.
59. Runtime Health

La disponibilidad de KMS/Secret Manager debe reflejarse correctamente en readiness.

Por ejemplo:

Application process alive
       ↓
Liveness = OK

pero:

Required Secret unavailable
       ↓
Readiness = NOT READY

cuando ese secreto sea crítico para operar.

60. Deployment

El pipeline deberá evitar transportar secretos directamente entre stages.

Conceptualmente:

GitHub Actions
      ↓
Build
      ↓
Artifact
      ↓
Deploy
      ↓
Runtime Identity
      ↓
Secret Manager / KMS

El artifact es el mismo.

Los secretos son inyectados por ambiente.

61. CI/CD

CI/CD deberá incluir:

secret scanning;
dependency scanning;
configuration validation;
IaC scanning cuando corresponda;
permisos mínimos;
detección de credenciales accidentales.

Los secretos reales de Production no deberán estar disponibles en builds normales.

62. Development

Development podrá utilizar:

secretos locales;
Secret Manager de Development;
mocks;
emuladores;

pero nunca deberá requerir credenciales de Production.

63. Testing

Los tests deberán utilizar:

Test Secrets
Mock KMS
Ephemeral Credentials

cuando sea apropiado.

Nunca deben utilizarse:

Production secrets
Production keys
Production credentials

en pruebas automatizadas.

64. Security Testing

Se deberá verificar:

secretos ausentes del repositorio;
secretos ausentes de imágenes;
logs sanitizados;
permisos mínimos;
acceso denegado a secretos no autorizados;
rotación;
revocación;
KMS failure;
Secret Manager failure;
startup failure;
recovery.
65. Consecuencias positivas
Seguridad
secretos fuera del código;
least privilege;
rotación;
separación de claves;
menor blast radius.
Operación
rotación sin cambiar código;
gestión centralizada;
auditoría;
disaster recovery mejor definido.
Arquitectura
runtime desacoplado;
containers reproducibles;
ambientes aislados;
soporte Cloud/Kubernetes.
Evolución

Permite incorporar nuevos proveedores de Secret Manager/KMS sin modificar el dominio Identity.

66. Consecuencias negativas

La estrategia introduce:

dependencia de infraestructura de seguridad;
complejidad de configuración;
costos de KMS/Secret Manager;
operaciones de rotación;
necesidad de permisos precisos;
mayor complejidad de recuperación.

Además, una mala configuración del KMS puede convertirse en una causa de indisponibilidad.

67. Alternativas consideradas
Alternativa A — Secrets en código

Rechazada.

Representa un riesgo crítico.

Alternativa B — Secrets en Git

Rechazada.

Incluso repositorios privados no deben utilizarse como Secret Manager.

Alternativa C — Secrets dentro de Docker Images

Rechazada.

Las imágenes deben ser reutilizables entre ambientes.

Alternativa D — .env como estrategia de producción

Rechazada como estrategia principal.

Puede servir para desarrollo local, pero no proporciona por sí sola el modelo requerido de:

rotation;
audit;
access control;
KMS;
lifecycle.
Alternativa E — Una sola clave para todo

Rechazada.

Aumenta el blast radius ante compromiso.

Alternativa F — KMS utilizado directamente para absolutamente todas las operaciones

No adoptada como regla universal.

Puede generar:

latencia;
costos;
dependencia excesiva;
impacto de disponibilidad.

La utilización debe evaluarse según el tipo de operación.

68. No negociables
Production secrets nunca deben estar en Git.
Production secrets nunca deben estar en código.
Production secrets nunca deben estar en Docker images.
Production secrets nunca deben aparecer en logs.
Production secrets nunca deben aparecer en eventos.
Production secrets nunca deben entrar en contexto AI.
Los ambientes deben estar aislados.
Debe utilizarse Secret Manager para secretos de producción.
Debe utilizarse KMS o mecanismo equivalente para claves criptográficas.
Las claves deben estar separadas por propósito.
JWT signing keys deben gestionarse de forma segura.
Refresh Token protection secrets deben estar externalizados.
MFA secrets deben estar protegidos.
Database credentials deben estar externalizadas.
Debe existir least privilege.
Workload identity debe preferirse cuando esté disponible.
Debe existir rotación.
Debe existir revocación.
Debe existir auditoría de acceso.
No se deben registrar valores secretos.
Debe existir fail closed ante secretos críticos no disponibles.
No deben existir defaults inseguros.
CI/CD debe incluir secret scanning.
Disaster Recovery debe incluir KMS/Secret Manager.
Los AI Agents no deben acceder directamente a secretos.
La administración de claves debe estar separada del uso de claves.
La arquitectura debe soportar rotación sin downtime cuando sea técnicamente posible.
69. Decisiones pendientes

Quedan abiertas:

KMS
proveedor;
KMS vs HSM;
regiones;
multi-region;
jerarquía de claves.
Secret Manager
proveedor;
namespaces;
naming;
integración Kubernetes;
estrategia de injection.
JWT
algoritmo definitivo;
KMS signing vs protected private key;
rotation interval;
JWKS caching.
Refresh Tokens
hashing vs HMAC;
pepper;
rotación criptográfica;
protección de secret versions.
MFA
algoritmo/protección de TOTP secrets;
key hierarchy;
secret access model.
Runtime
secret injection;
reload dinámico;
startup-only secrets;
runtime fetch;
cache TTL.
Disaster Recovery
backup de metadata;
key recovery;
Secret Manager recovery;
multi-region.

Estas decisiones se deberán cerrar antes de la implementación definitiva de la infraestructura de producción.

70. Criterios de aceptación arquitectónica

Este ADR podrá considerarse satisfecho cuando:

 Configuration esté separada de Secrets.
 Secrets estén separados de Cryptographic Keys.
 Exista Secret Manager.
 Exista KMS/HSM o equivalente.
 Exista separación de claves.
 JWT signing keys estén externalizadas.
 Refresh Token secrets estén externalizados.
 MFA secrets estén protegidos.
 DB credentials estén externalizadas.
 Existan ambientes aislados.
 Exista workload identity cuando corresponda.
 Exista least privilege.
 Exista secret rotation.
 Exista key rotation.
 Exista revocation.
 Exista audit trail.
 No existan secretos en logs.
 No existan secretos en eventos.
 No existan secretos en imágenes.
 No existan secretos en Git.
 Exista secret scanning.
 Exista fail closed.
 Exista soporte de recuperación.
 Exista estrategia de AI isolation.
 Exista estrategia de disaster recovery.
 Existan pruebas de seguridad y recuperación.
71. Trazabilidad
Blueprint
Security by Design
Zero Trust
Least Privilege
Defense in Depth
Cloud Ready
Observable
Continuously Evolvable
Engineering Specification
ESP-0001 — Identity
Implementation Stories
IS-IDENTITY-005
Authentication

IS-IDENTITY-009
Refresh Token Lifecycle

IS-IDENTITY-010
Multi-Factor Authentication

IS-IDENTITY-011
Security Policy Engine

IS-IDENTITY-012
Audit & Security Events

IS-IDENTITY-013
Domain Events & Outbox

IS-IDENTITY-015
Identity Persistence

IS-IDENTITY-016
Identity Testing & Quality

IS-IDENTITY-017
Identity Runtime & Deployment
ADRs relacionados
ADR-IDENTITY-002
JWT Strategy

ADR-IDENTITY-003
Stateful Sessions & Refresh Tokens

ADR-IDENTITY-008
Cryptographic Key Management

ADR-IDENTITY-009
Token & Client Storage

ADR-IDENTITY-012
PostgreSQL Schema

ADR-IDENTITY-014
Encryption & Secret Protection

ADR-IDENTITY-016
Concurrency & Transaction Boundaries

ADR-IDENTITY-017
Database Migration Strategy

ADR-IDENTITY-025
Refresh Token Cryptography & Concurrency

ADR-IDENTITY-029
Disaster Recovery & Backup
72. Siguiente ADR

La secuencia queda:

ADR-IDENTITY-023
MFA Factor Strategy
        │
        ▼
ADR-IDENTITY-024
MFA Challenge TTL, Attempts & Recovery
        │
        ▼
ADR-IDENTITY-025
Refresh Token Cryptography & Concurrency
        │
        ▼
ADR-IDENTITY-026
API Pagination, Filtering & Idempotency
        │
        ▼
ADR-IDENTITY-027
Runtime Configuration, Secrets & KMS
        │
        ▼
ADR-IDENTITY-028
Observability & Operational Security
        │
        ▼
ADR-IDENTITY-029
Disaster Recovery & Backup
        │
        ▼
ADR-IDENTITY-030
Autoscaling & Capacity

Con ADR-IDENTITY-027 queda establecido el modelo de configuración, secretos y material criptográfico en runtime, evitando que estas responsabilidades terminen mezcladas con el código de Identity.

El siguiente paso lógico es ADR-IDENTITY-028 — Observability & Operational Security, donde definiremos cómo EVOXA observará autenticación, autorización, sesiones, MFA, tokens, eventos de seguridad, métricas, logs, trazas, alertas y respuesta operacional sin exponer información sensible.
