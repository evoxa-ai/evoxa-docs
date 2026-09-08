TASK-001-10 — Configuration & Secrets

IS: IS-001 — Identity Domain Foundation
ESP: ESP-0001/14 — Identity Implementation Requirements
Dependencias: TASK-001-08 — PostgreSQL Infrastructure, TASK-001-09 — Redis Infrastructure
Prioridad: 🔴 Critical
Estado: Ready for Implementation

1. Objetivo

Establecer el sistema centralizado de Configuration & Secrets de EVOXA Identity.

La finalidad es que toda la aplicación obtenga su configuración desde una única capa controlada, diferenciando claramente:

Configuration
Secrets
Runtime State

La regla principal será:

La configuración puede ser externa y versionable; los secretos deben ser gestionados como material sensible y nunca formar parte del código fuente, imágenes, logs, eventos o contratos públicos.

2. Problema que resolvemos

No queremos tener:

os.getenv("DATABASE_HOST")
os.getenv("REDIS_HOST")
os.getenv("JWT_SECRET")
os.getenv("...")

dispersos por todo el código.

Tampoco:

DATABASE_PASSWORD = "..."
JWT_SECRET = "..."

dentro del proyecto.

La arquitectura será:

                 Environment
                     │
                     ▼
              Configuration
                     │
             ┌───────┴───────┐
             ▼               ▼
        Non-Sensitive      Secrets
        Configuration      Provider
             │               │
             └───────┬───────┘
                     ▼
                  Settings
                     │
                     ▼
                Application
3. Alcance

Esta tarea establece:

configuración central;
variables de entorno;
clasificación de configuración;
secretos;
validación;
startup configuration checks;
integración con PostgreSQL;
integración con Redis;
configuración de seguridad;
configuración de observabilidad;
configuración de Event Platform;
configuración de JWT/KMS;
configuración por ambiente;
reglas de logging;
protección contra exposición accidental;
testing de configuración.

No implementa todavía:

❌ KMS real
❌ Secret Manager real
❌ JWT signing
❌ OAuth
❌ MFA
❌ Authorization

Aquí establecemos la infraestructura de configuración que esas funcionalidades utilizarán.

4. Clasificación

Utilizaremos tres categorías principales.

4.1 Public Configuration

Información que no representa un secreto.

Ejemplos:

APP_NAME
APP_VERSION
APP_ENVIRONMENT
API_PREFIX
DATABASE_HOST
DATABASE_PORT
REDIS_HOST
REDIS_PORT
LOG_LEVEL

Aunque algunos datos sean internos, no son credenciales.

4.2 Sensitive Configuration

Información operacional que puede requerir protección adicional.

Ejemplos:

security configuration
tenant security settings
rate-limit configuration
internal service configuration

No necesariamente son secretos criptográficos, pero tampoco deben exponerse indiscriminadamente.

4.3 Secrets

Material que permite autenticación, acceso o descifrado.

Ejemplos:

DATABASE_PASSWORD
REDIS_PASSWORD
JWT private key
KMS credentials
OAuth client secret
SMTP credentials
API credentials
encryption keys
webhook signing secrets

Regla:

SECRET
  ✗ Git
  ✗ source code
  ✗ Docker image
  ✗ logs
  ✗ events
  ✗ API response
  ✗ AI context
5. Fuente de configuración

La prioridad conceptual será:

Environment / Secret Provider
          │
          ▼
       Settings
          │
          ▼
       Application

La aplicación no debería consultar directamente al Secret Manager desde cualquier módulo.

En su lugar:

Secret Provider
       │
       ▼
Configuration Layer
       │
       ▼
Settings
       │
       ▼
Dependencies
6. Settings

La configuración central seguirá ubicada en:

apps/api/app/core/config.py

Pero la estructura debe crecer de forma organizada.

Propuesta:

from pydantic_settings import BaseSettings, SettingsConfigDict


class Settings(BaseSettings):

    # Application
    app_name: str = "EVOXA Identity"
    app_version: str = "0.1.0"
    environment: str = "development"
    debug: bool = False
    api_prefix: str = "/api/v1"

    # Database
    database_host: str
    database_port: int = 5432
    database_name: str
    database_user: str
    database_password: str

    # Redis
    redis_host: str
    redis_port: int = 6379
    redis_database: int = 0
    redis_username: str | None = None
    redis_password: str | None = None
    redis_ssl: bool = True

    model_config = SettingsConfigDict(
        env_file=".env",
        env_file_encoding="utf-8",
        extra="ignore",
    )

Esta estructura es inicial y se ampliará.

7. No duplicar configuración

TASK-001-08 ya definió configuración PostgreSQL.

TASK-001-09 ya definió configuración Redis.

Ahora TASK-001-10 debe convertirse en el punto central de configuración.

Por lo tanto, no debemos terminar con:

database/config.py
redis/config.py
security/config.py

cada uno leyendo variables de entorno por separado.

Debe existir:

                    Settings
                       │
        ┌──────────────┼──────────────┐
        ▼              ▼              ▼
    PostgreSQL       Redis         Security
8. Configuración agrupada

Cuando el proyecto crezca, podemos estructurar:

Settings
├── ApplicationSettings
├── DatabaseSettings
├── RedisSettings
├── SecuritySettings
├── JWTSettings
├── KMSSettings
├── EventSettings
├── ObservabilitySettings
└── RuntimeSettings

La implementación puede utilizar nested settings de Pydantic.

Esto evita que Settings se convierta en una clase monolítica inmanejable.

9. Application Settings

Configuración inicial:

APP_NAME
APP_VERSION
APP_ENVIRONMENT
APP_DEBUG
API_PREFIX

Ejemplo:

APP_NAME=EVOXA Identity
APP_VERSION=0.1.0
APP_ENVIRONMENT=development
APP_DEBUG=true
API_PREFIX=/api/v1
10. Environment

Los ambientes conceptuales son:

development
qa
staging
production

La aplicación debe conocer explícitamente su ambiente.

Esto permite evitar errores como:

production + debug=true
11. Startup Validation

La aplicación debe validar la configuración al iniciar.

Flujo:

START
  │
  ▼
Load configuration
  │
  ▼
Validate
  │
  ├── valid → continue
  │
  └── invalid → FAIL

No debemos permitir:

missing JWT signing configuration
       ↓
application starts anyway

cuando esa configuración sea obligatoria para el entorno.

12. Fail Closed

Si falta una configuración crítica:

Application startup
       │
       ▼
Missing critical secret
       │
       ▼
STARTUP FAILURE

No:

use default secret

ni:

disable security
13. Development Defaults

Podemos permitir defaults únicamente para configuración no sensible.

Por ejemplo:

api_prefix: str = "/api/v1"
database_port: int = 5432

No:

database_password: str = "password"

ni:

jwt_secret: str = "dev-secret"

La seguridad no debe depender de defaults inseguros.

14. .env

En desarrollo:

.env

puede contener:

DATABASE_HOST=localhost
DATABASE_PORT=5432
DATABASE_NAME=evoxa
DATABASE_USER=evoxa
DATABASE_PASSWORD=...
REDIS_HOST=localhost
REDIS_PORT=6379

Pero:

.env

debe estar en:

.gitignore
15. .env.example

Sí debemos versionar:

.env.example

pero sin valores reales.

Por ejemplo:

DATABASE_HOST=
DATABASE_PORT=5432
DATABASE_NAME=
DATABASE_USER=
DATABASE_PASSWORD=

REDIS_HOST=
REDIS_PORT=6379
REDIS_PASSWORD=

Esto documenta qué necesita el proyecto sin filtrar secretos.

16. Secret Provider

En producción la arquitectura será:

Application
      │
      ▼
Secret Provider
      │
      ▼
Secret Manager / KMS

El proveedor concreto queda fuera de esta tarea.

Podrá ser:

AWS Secrets Manager
AWS KMS
Azure Key Vault
GCP Secret Manager
Kubernetes Secrets + external KMS

según la infraestructura final.

17. KMS

Identity necesita diferenciar:

JWT Signing Key
Encryption Key
MFA secrets
Webhook signing key
Other cryptographic material

No debemos utilizar una única clave universal:

MASTER_SECRET

para todo.

Esto respeta la separación de propósitos establecida anteriormente.

18. JWT Configuration

La configuración futura deberá soportar:

JWT_ISSUER
JWT_AUDIENCE
JWT_KEY_ID
JWT_PRIVATE_KEY
JWT_PUBLIC_KEY
JWT_ACCESS_TOKEN_TTL

Pero la arquitectura debe preferir:

private signing key
        │
        ▼
KMS / Secret Manager

y:

public keys
        │
        ▼
JWKS

No se deben almacenar claves privadas directamente en el repositorio.

19. JWT TTL

Los valores de:

access token TTL
refresh token TTL
session lifetime

no deben definirse arbitrariamente en esta tarea.

Serán establecidos por las decisiones de autenticación/session/token correspondientes.

Aquí solamente proporcionamos la configuración necesaria.

20. Database Configuration

Consolidamos:

DATABASE_HOST
DATABASE_PORT
DATABASE_NAME
DATABASE_USER
DATABASE_PASSWORD
DATABASE_POOL_SIZE
DATABASE_MAX_OVERFLOW
DATABASE_POOL_TIMEOUT
DATABASE_POOL_RECYCLE
DATABASE_ECHO

y añadimos, si corresponde:

DATABASE_SSL_MODE
DATABASE_SSL_CA

Los certificados también deben tratarse como material protegido cuando corresponda.

21. Redis Configuration

Consolidamos:

REDIS_HOST
REDIS_PORT
REDIS_DATABASE
REDIS_USERNAME
REDIS_PASSWORD
REDIS_SSL
REDIS_MAX_CONNECTIONS
REDIS_SOCKET_TIMEOUT
REDIS_SOCKET_CONNECT_TIMEOUT

La aplicación debe evitar que la contraseña llegue a cualquier logging accidental.

22. Event Platform Configuration

Prepararemos:

EVENT_PLATFORM_ENDPOINT
EVENT_PLATFORM_NAMESPACE
EVENT_PLATFORM_TIMEOUT
EVENT_PLATFORM_RETRY_LIMIT
EVENT_PLATFORM_TLS
EVENT_PLATFORM_CREDENTIAL

La credencial será secret.

El endpoint puede ser configuración no secreta dependiendo de la infraestructura.

23. Observability Configuration

Prepararemos:

LOG_LEVEL
LOG_FORMAT
TRACE_ENABLED
METRICS_ENABLED
TRACING_ENDPOINT
METRICS_ENDPOINT

Si existe una credencial para observability:

OBSERVABILITY_API_KEY

deberá tratarse como secret.

24. Security Configuration

Podemos tener:

PASSWORD_HASHING_ALGORITHM
PASSWORD_HASHING_PARAMS
MFA_CHALLENGE_TTL
RATE_LIMIT_ENABLED
SECURITY_HEADERS_ENABLED

Pero los parámetros criptográficos finales de Password/MFA deben respetar los ADR correspondientes.

Esta configuración no debe permitir que un administrador de organización debilite las políticas de plataforma.

25. Configuration ≠ Policy

Esto es muy importante.

Configuration:

DATABASE_POOL_SIZE=20

Policy:

MFA required for sensitive operation

no son lo mismo.

La configuración controla:

mientras que Policy controla:

No debemos permitir que una variable de entorno accidentalmente modifique una política de seguridad sin governance.

26. Runtime Configuration

Algunos valores pueden cambiar durante la operación.

Pero debemos diferenciar:

Startup Configuration

de:

Dynamic Configuration

No todo debe ser dinámico.

Por ejemplo:

JWT signing configuration

requiere lifecycle controlado.

27. Secret Rotation

Los secretos deben poder rotarse.

Conceptualmente:

Secret V1
   │
   ▼
Rotation
   │
   ▼
Secret V2

La aplicación debe estar diseñada para que una rotación no requiera modificar código fuente.

28. Key Rotation

Para JWT:

kid = key-001

posteriormente:

kid = key-002

durante una transición.

El sistema debe poder conocer las claves públicas necesarias para validar tokens todavía válidos durante el periodo de rotación.

La implementación exacta corresponde a las tareas de Authentication/Token.

29. Secrets en memoria

Aunque un secret necesariamente puede existir temporalmente en memoria del proceso para algunas operaciones, debemos minimizar:

lifetime
scope
copies
logging
exposure

No debemos propagar un secret como parte de objetos globales innecesariamente.

30. Secrets en Exceptions

Nunca:

raise Exception(
    f"Database connection failed: {database_url}"
)

porque podría incluir:

password

El error debe ser:

Database connection failed

con:

correlation_id

para investigar el problema de forma segura.

31. Secrets en Logs

Regla absoluta:

NO SECRET VALUES

ni siquiera parcialmente:

password=***

es preferible a:

password=secret123

Pero incluso el patrón de logging debe evitar accidentalmente serializar objetos que contengan secrets.

32. Secrets en Events

Nunca:

{
  "event": "UserCreated",
  "password": "...",
  "token": "..."
}

Los eventos solamente contienen la información necesaria para comunicar el hecho de negocio.

33. Secrets y AI

La configuración de secrets debe impedir:

AI Agent
    ↓
DATABASE_PASSWORD

o:

AI Agent
    ↓
JWT private key

La IA puede trabajar con:

configuration metadata

cuando sea necesario, pero no debe recibir secretos.

34. Secret Redaction

Podemos preparar una utilidad:

core/security/redaction.py

para sanitizar estructuras antes de logging.

Ejemplo conceptual:

SENSITIVE_FIELDS = {
    "password",
    "secret",
    "token",
    "private_key",
    "api_key",
    "authorization",
    "cookie",
}

Y:

def redact(data: dict) -> dict:
    ...

Esto no reemplaza la regla de no loggear secrets, pero agrega una defensa adicional.

35. Configuration Dump

No debemos tener:

GET /debug/config

que devuelva:

{
  "database_password": "...",
  "jwt_private_key": "..."
}

En caso de necesitar diagnóstico, solamente se deben exponer metadatos no sensibles:

environment
version
database configured = true
redis configured = true

Nunca valores secretos.

36. Startup Diagnostic

Podemos generar un diagnóstico interno:

Configuration validation:
  application: OK
  database: OK
  redis: OK
  event platform: configured
  security: OK

Sin:

password values
key values
tokens
credentials
37. Configuration Validation

Debemos validar:

Tipos
PORT → integer
TTL → duration/integer
BOOLEAN → bool
Rangos

Por ejemplo:

PORT > 0
POOL_SIZE > 0
TIMEOUT > 0
Combinaciones

Ejemplo:

production + debug=true

debe ser rechazado o explícitamente bloqueado según la política de runtime.

38. Environment-specific Validation

Development:

local .env

Production:

Secret Provider
TLS
secure configuration

La validación debe ser más estricta en producción.

39. Configuration Contract

Podemos establecer:

class ConfigurationError(Exception):
    pass

y:

def validate_settings(settings):
    ...

Esto permitirá fallar de manera controlada antes de iniciar servidores.

40. Startup Order

La aplicación deberá seguir:

1. Load configuration
        ↓
2. Validate configuration
        ↓
3. Initialize PostgreSQL
        ↓
4. Initialize Redis
        ↓
5. Initialize Event infrastructure
        ↓
6. Start API

Si falla una dependencia crítica:

STARTUP FAILURE

o se entra en el modo degradado explícitamente permitido por la arquitectura.

41. No Silent Fallback

Nunca:

try:
    load_secret()
except:
    use_default_secret()

Eso sería un grave problema de seguridad.

Correcto:

Secret missing
    ↓
ConfigurationError
    ↓
Startup fails
42. Testing
Test de configuración válida
valid environment
      ↓
Settings loads
Test de secret faltante
missing DATABASE_PASSWORD
      ↓
ConfigurationError
Test de producción insegura
production
+
debug=true

debe ser detectado.

Test de .env

Verificar:

.env

no está trackeado por Git.

43. Test de Redaction

Entrada:

{
  "username": "admin",
  "password": "secret",
  "token": "abc"
}

Resultado esperado:

{
  "username": "admin",
  "password": "[REDACTED]",
  "token": "[REDACTED]"
}
44. Test de Secret Leakage

Se debe comprobar que:

logs
events
errors
responses

no contienen valores de secrets utilizados durante tests.

Este será posteriormente un quality/security gate.

45. Configuration Structure

La estructura final inicial:

apps/api/app/core/
│
├── config.py
│
├── configuration/
│   ├── validation.py
│   └── classification.py
│
├── security/
│   └── redaction.py
│
└── infrastructure/
    ├── database/
    └── redis/

Más adelante:

core/configuration/
├── application.py
├── database.py
├── redis.py
├── security.py
├── jwt.py
├── kms.py
├── events.py
└── observability.py
46. Git Protection

Debemos incluir en .gitignore:

.env
.env.*
!.env.example
*.pem
*.key
*.p12
*.pfx
secrets/

La lista exacta deberá revisarse para no excluir accidentalmente archivos legítimos del proyecto.

47. Docker Protection

No debemos hacer:

ENV DATABASE_PASSWORD=...

porque el secret podría terminar en metadata/image layers dependiendo del build.

Preferimos:

runtime secret injection
48. Kubernetes

En producción:

Pod
 │
 ├── Config
 │
 └── Secret
       │
       ▼
   Application

La solución exacta de Secret Manager/KMS será definida por deployment.

49. Configuration Ownership

La configuración se dividirá por responsabilidad:

Application
   └── Runtime

Database
   └── PostgreSQL

Redis
   └── Redis

Security
   └── Security infrastructure

JWT
   └── Authentication/Token

KMS
   └── Cryptography

Events
   └── Event infrastructure

Observability
   └── Logging/Metrics/Tracing

Pero el loading/validation permanece centralizado.

50. Criterios de aceptación
ID	Criterio
AC-01	Existe una fuente central de configuración
AC-02	PostgreSQL utiliza Settings
AC-03	Redis utiliza Settings
AC-04	Secrets están separados conceptualmente de configuration
AC-05	No existen secrets hardcodeados
AC-06	.env no se versiona
AC-07	Existe .env.example sin secretos
AC-08	Configuración crítica se valida durante startup
AC-09	Missing critical secret produce startup failure
AC-10	No existen default secrets
AC-11	Secrets no aparecen en logs
AC-12	Secrets no aparecen en events
AC-13	Secrets no aparecen en API responses
AC-14	Secrets no llegan al contexto de AI
AC-15	Existe redaction defensiva
AC-16	Configuration soporta ambientes
AC-17	Existe soporte para Secret Provider
AC-18	Existe soporte para KMS integration
AC-19	Existe configuración para JWT/KMS/Event Platform
AC-20	Existen tests de configuration security
51. Definition of Done
[ ] Central Settings
[ ] Application configuration
[ ] PostgreSQL configuration
[ ] Redis configuration
[ ] Security configuration
[ ] JWT configuration foundation
[ ] KMS configuration foundation
[ ] Event Platform configuration foundation
[ ] Observability configuration
[ ] Environment support
[ ] Configuration validation
[ ] Secret classification
[ ] Secret provider boundary
[ ] Redaction
[ ] .env protection
[ ] .env.example
[ ] Docker secret rules
[ ] Startup validation
[ ] No default secrets
[ ] Unit tests
[ ] Security tests
[ ] Architecture tests
52. Trazabilidad
IS-001
 │
 └── TASK-001-10
       │
       ├── ESP-0001/14
       │     Implementation Requirements
       │
       ├── ESP-0001/15
       │     Deployment & Operational Requirements
       │
       ├── ADR-008
       │     Cryptographic Key Management
       │
       ├── ADR-014
       │     Encryption & Secret Protection
       │
       ├── ADR-027
       │     Runtime Configuration, Secrets & KMS
       │
       ├── ADR-028
       │     Observability & Operational Security
       │
       ├── ADR-034
       │     Password Security
       │
       ├── ADR-037
       │     API Token & Service Credential Security
       │
       └── ADR-038
             S2S Authentication & Workload Identity
53. Regla de oro de TASK-001-10

A partir de esta tarea debemos considerar esta frontera como no negociable:

                    CONFIGURATION
                         │
              ┌──────────┴──────────┐
              ▼                     ▼
        Non-Sensitive             Secrets
              │                     │
              │                     ▼
              │              Secret Manager/KMS
              │                     │
              └──────────┬──────────┘
                         ▼
                      Settings
                         │
                         ▼
                    Application

Y jamás:

Secret
 │
 ├── Git             ❌
 ├── Docker Image    ❌
 ├── Log             ❌
 ├── Event           ❌
 ├── API Response    ❌
 ├── Audit           ❌
 └── AI Context      ❌
Estado de IS-001

Con esto ya tenemos:

TASK-001-01  API Runtime Foundation       ✅
TASK-001-02  Identity Domain Package      ✅
TASK-001-03  Layer Architecture           ✅
TASK-001-04  Identity Shared Kernel       ✅
TASK-001-05  Domain Events Foundation     ✅
TASK-001-06  Repository Contracts         ✅
TASK-001-07  Unit of Work                 ✅
TASK-001-08  PostgreSQL Infrastructure    ✅
TASK-001-09  Redis Infrastructure         ✅
TASK-001-10  Configuration & Secrets      ✅

La base técnica empieza a quedar bastante sólida:

             EVOXA Identity
                   │
          ┌────────┴────────┐
          ▼                 ▼
    Configuration       Security
          │                 │
          ▼                 ▼
    Infrastructure     Secret/KMS
       │       │
       ▼       ▼
 PostgreSQL   Redis
       │
       ▼
 Unit of Work
       │
       ▼
 Repositories
       │
       ▼
 Domain
       │
       ▼
 Domain Events
       │
       ▼
 Transactional Outbox

Siguiente: TASK-001-11 — Security Foundation. Ahí empezaremos a implementar la base de seguridad transversal: hashing/cryptography boundaries, secure random generation, constant-time comparisons, security context, protección de credenciales, clasificación de secretos y primitives que luego utilizarán Authentication, Sessions, Tokens, MFA y Authorization, sin implementar todavía el login completo.
