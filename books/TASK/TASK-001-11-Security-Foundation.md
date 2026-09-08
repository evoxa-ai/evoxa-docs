TASK-001-11 — Security Foundation

IS: IS-001 — Identity Domain Foundation
ESP: ESP-0001/14 — Identity Implementation Requirements
Dependencias: TASK-001-10 — Configuration & Secrets
Prioridad: 🔴 Critical
Estado: Ready for Implementation

1. Objetivo

Establecer las primitivas de seguridad transversales que utilizará todo EVOXA Identity.

Esta tarea no implementa todavía:

❌ Login
❌ JWT completo
❌ Refresh Tokens
❌ MFA
❌ Authorization Runtime
❌ Password Reset
❌ OAuth/OIDC

Su objetivo es crear la base segura sobre la que esas funcionalidades serán construidas.

La regla arquitectónica es:

Las funcionalidades de Identity no deben implementar sus propias primitivas criptográficas o mecanismos de seguridad cuando exista una capacidad centralizada.

2. Alcance

Esta tarea cubre:

Secure Random
Password Hashing Boundary
Cryptographic Key Boundary
Constant-Time Comparison
Secret Redaction
Security Context
Security Actor
Security Metadata
Credential Protection
Security Exceptions
Security Utilities
Security Dependency Injection
Security Testing

Además establece las reglas para:

tokens
passwords
MFA secrets
API credentials
cryptographic keys
security-sensitive identifiers
3. Arquitectura

La seguridad transversal estará ubicada fuera de los módulos de negocio:

apps/api/app/
│
├── core/
│   └── security/
│       ├── hashing/
│       ├── random/
│       ├── crypto/
│       ├── comparison/
│       ├── redaction/
│       ├── context/
│       ├── exceptions.py
│       └── dependencies.py
│
└── domains/
    └── identity/

Los módulos Identity consumirán estas capacidades mediante contratos.

4. Principio Zero Trust

La infraestructura de seguridad debe asumir:

Input = untrusted
Request = untrusted
Token = untrusted
Client = untrusted
External Identity = untrusted
AI Agent = untrusted

La confianza se obtiene mediante validación explícita.

5. Secure Random Generation

Identity necesita generar valores impredecibles para:

verification tokens
password reset tokens
refresh tokens
MFA challenges
recovery codes
API tokens
authorization codes
nonce
state
security identifiers

Nunca debemos utilizar:

random.random()

para secretos de seguridad.

Debe utilizarse un generador criptográficamente seguro.

En Python:

import secrets


def generate_token_bytes(length: int = 32) -> bytes:
    return secrets.token_bytes(length)


def generate_token_hex(length: int = 32) -> str:
    return secrets.token_hex(length)

La función debe vivir detrás de una abstracción de seguridad.

6. Randomness Contract

Podemos definir:

class SecureRandom:
    def bytes(self, length: int) -> bytes:
        ...

    def token(self, length: int = 32) -> str:
        ...

Esto permite:

Production
    ↓
CSPRNG

Tests
    ↓
controlled implementation

sin que los módulos conozcan detalles de implementación.

7. Password Hashing

La contraseña nunca debe almacenarse directamente.

Password
   │
   ▼
Argon2id
   │
   ▼
Password Hash
   │
   ▼
PostgreSQL

El Blueprint/ADRs establecen Argon2id como algoritmo.

No debemos almacenar:

plaintext password

ni utilizar:

MD5
SHA1
SHA256(password)

como password hashing.

8. Password Hashing Boundary

Crear:

core/security/hashing/
├── interface.py
└── argon2.py

Conceptualmente:

class PasswordHasher:

    def hash(self, password: str) -> str:
        ...

    def verify(
        self,
        password: str,
        password_hash: str,
    ) -> bool:
        ...

La implementación concreta utiliza Argon2id.

9. Password Hash Parameters

Los parámetros exactos:

memory_cost
time_cost
parallelism
hash length
salt length

no deben inventarse en esta tarea.

Deben ser establecidos y versionados conforme a:

ADR-034 — Password Security

y posteriormente validados mediante performance/security testing.

10. Adaptive Rehashing

La infraestructura debe permitir detectar:

stored hash parameters
        ↓
current recommended parameters

Si el hash está obsoleto:

successful verification
        ↓
rehash with current parameters
        ↓
persist new hash

Esto permite evolucionar los parámetros sin obligar a todos los usuarios a cambiar su contraseña inmediatamente.

11. Password Verification

Nunca debemos hacer:

if password == stored_hash:

La verificación debe delegarse al password hasher.

Password
   │
   ▼
Argon2id Verify
   │
   ├── valid
   └── invalid
12. Password Validation

Esta tarea puede preparar un boundary:

validate_password_input(...)

pero las reglas definitivas de password policy pertenecen a Identity/Policy.

Por ejemplo:

minimum length
breached-password checks
reuse rules
history
complexity

no deben codificarse arbitrariamente aquí.

13. Constant-Time Comparison

Para comparar valores sensibles utilizaremos comparación resistente a timing attacks cuando corresponda.

Ejemplo:

import hmac


def secure_compare(
    expected: bytes,
    provided: bytes,
) -> bool:
    return hmac.compare_digest(
        expected,
        provided,
    )

Aplicable a:

security tokens
HMAC values
verification codes
derived secret values

según el caso.

14. Token Comparison

No debemos asumir que:

provided == expected

sea apropiado para cualquier secreto.

Los mecanismos de comparación deben quedar centralizados cuando el material sea security-sensitive.

15. Hash vs Encryption

Debemos diferenciar claramente:

Hash

Para valores que necesitamos verificar pero no recuperar:

password
refresh token representation
API token representation

según sus respectivos ADR.

Encryption

Para información que necesitamos recuperar:

MFA secret
encrypted credential material
other recoverable sensitive data

cuando corresponda.

16. Key Management Boundary

Esta tarea no implementa un KMS.

Establece un contrato:

Application
     │
     ▼
Key Management Port
     │
     ▼
KMS / Secret Manager

Nunca:

Application
     ↓
hardcoded private key
17. Cryptographic Key Separation

No debemos utilizar una única clave para todos los propósitos.

Separaremos conceptualmente:

JWT signing
JWT encryption, if used
MFA encryption
Webhook signing
API credential protection
Other encryption

La clave debe tener un propósito explícito.

18. Key IDs

Las claves deben poder identificarse mediante un kid o equivalente:

key-001
key-002
key-003

Esto será especialmente importante para JWT:

JWT
 └── kid
       ↓
JWKS / KMS

La rotación será tratada en las tareas de Authentication/Token.

19. Secure Secret Handling

Un secret debe tener el menor alcance posible.

Evitar:

global_secret = ...

Preferir:

Secret Provider
      ↓
specific operation
      ↓
short-lived usage
20. Security Context

Debemos crear un contexto de seguridad común:

SecurityContext

que posteriormente podrá representar:

actor
authentication state
session
tenant
organization
membership
MFA assurance
authentication method
risk context
correlation

Pero no debe convertirse en un contenedor de todos los datos del usuario.

21. Actor Model

El actor debe soportar:

USER
SERVICE
SYSTEM
AI_AGENT

Esto está alineado con Audit/Security Events.

Ejemplo:

class ActorType(str, Enum):
    USER = "USER"
    SERVICE = "SERVICE"
    SYSTEM = "SYSTEM"
    AI_AGENT = "AI_AGENT"
22. Actor Identity

Conceptualmente:

@dataclass(frozen=True)
class Actor:
    type: ActorType
    id: UUID | None

Un SYSTEM actor puede no tener un user ID.

Un AI_AGENT debe tener una identidad de servicio propia.

23. Authentication Context

SecurityContext podrá posteriormente contener:

authenticated
session_id
authentication_method
mfa_verified
assurance_level

pero los valores definitivos y su lifecycle serán establecidos por Authentication/MFA.

24. Tenant Context

También debe existir una separación entre:

Actor
Tenant Context

No debemos inferir automáticamente:

User → Organization

porque un User puede tener múltiples Memberships.

El contexto activo debe representar una membership válida.

25. Security Context Inmutability

Una vez construido para una evaluación crítica:

SecurityContext

debe tratarse como immutable.

No queremos:

Authorization starts
       ↓
context says tenant A
       ↓
context modified
       ↓
tenant B

Esto sería una vulnerabilidad crítica.

26. Request Security Metadata

Podremos asociar:

correlation_id
trace_id
request_id
client_ip
user_agent

pero debemos distinguir:

observability metadata

de:

authorization attributes

No todo dato HTTP debe convertirse automáticamente en una condición de autorización.

27. Security Exceptions

Crear:

core/security/exceptions.py

con categorías como:

class SecurityError(Exception):
    pass


class InvalidCredential(SecurityError):
    pass


class InvalidSecret(SecurityError):
    pass


class SecurityConfigurationError(SecurityError):
    pass


class CryptographicError(SecurityError):
    pass

Los errores públicos serán transformados por la API al error envelope estándar.

28. No Leakage Through Exceptions

Nunca:

raise CryptographicError(private_key)

ni:

raise InvalidCredential(
    f"Invalid password: {password}"
)

Los detalles sensibles nunca forman parte de exceptions públicas.

29. Secret Redaction

Continuaremos lo establecido en TASK-001-10.

Centralizaremos:

core/security/redaction.py

para impedir que datos sensibles aparezcan en:

logs
traces
exceptions
audit metadata
events
diagnostics
30. Sensitive Field Registry

Inicialmente:

SENSITIVE_FIELDS = {
    "password",
    "password_hash",
    "token",
    "access_token",
    "refresh_token",
    "secret",
    "mfa_secret",
    "recovery_code",
    "private_key",
    "client_secret",
    "api_key",
    "authorization",
    "cookie",
}

Esta lista debe crecer conforme aparezcan nuevos tipos de credenciales.

31. Secure Logging Boundary

Los módulos no deberían decidir individualmente cómo sanitizar información.

Preferimos:

Application
     ↓
Structured Logger
     ↓
Redaction
     ↓
Log sink

Así reducimos el riesgo de:

Developer accidentally logs credential
32. Security Dependency Injection

Las primitivas se proporcionarán mediante DI.

Ejemplo conceptual:

def get_password_hasher() -> PasswordHasher:
    ...

y:

def get_secure_random() -> SecureRandom:
    ...

Los Use Cases dependerán de interfaces, no de librerías concretas.

33. Domain Boundary

Domain:

NO
 ↓
argon2 library
redis
jwt library
kms sdk
fastapi

El Domain debe expresar reglas.

Infrastructure implementa mecanismos.

Domain
   │
   ▼
Security Contract
   ▲
   │
Infrastructure
34. Authentication Boundary

Más adelante:

Authentication
      │
      ├── PasswordHasher
      ├── SecureRandom
      ├── SecurityContext
      └── KeyManagement

pero Authentication será responsable del flujo de autenticación.

Security Foundation solamente proporciona primitivas.

35. Token Boundary

Token utilizará:

SecureRandom
KeyManagement
SecureComparison

para implementar posteriormente:

access tokens
refresh tokens
API tokens
authorization codes

cada uno según su ADR.

36. MFA Boundary

MFA utilizará:

SecureRandom
KeyManagement
SecureComparison
SecretProtection

para:

challenges
recovery codes
TOTP secrets
trusted-device material

sin que Security Foundation conozca la lógica de MFA.

37. Authorization Boundary

Authorization utilizará:

SecurityContext
Actor
TenantContext
SecureComparison

pero:

Security Foundation no decide ALLOW/DENY.

La decisión pertenece al Authorization Runtime.

38. AI Security Boundary

Los AI Agents utilizarán:

ActorType.AI_AGENT

y deberán pasar por las mismas primitivas de seguridad.

La IA no puede:

disable security
obtain private keys
retrieve passwords
bypass MFA
bypass authorization
modify security policy directly
39. Secure Random Tests

Debemos verificar:

length
uniqueness
allowed character representation
cryptographic API usage

No debemos testear que el valor sea "aleatorio" mediante una expectativa estadística frágil.

40. Password Tests

Mínimo:

hash generated
hash differs from plaintext
verify valid password
reject invalid password
different passwords produce different hashes
malformed hash handled safely
rehash detection
41. Constant-Time Tests

Debemos probar:

equal values → true
different values → false
empty values handled
different lengths handled

La propiedad de timing exacta requiere herramientas especializadas; no debe inferirse únicamente desde un unit test.

42. Secret Leakage Tests

Debemos crear tests que verifiquen que secretos no aparezcan en:

exceptions
logs
audit events
domain events
API responses
diagnostics
43. Configuration Security Tests

Integración con TASK-001-10:

missing critical secret
invalid cryptographic configuration
production insecure configuration
invalid key configuration

deben provocar comportamiento seguro.

44. Architecture Tests

Debemos verificar:

Domain → no security library
Domain → no FastAPI
Domain → no Redis
Domain → no KMS SDK

Mientras que:

Infrastructure → allowed
Application → security contracts
API → application
45. Acceptance Criteria
ID	Criterio
AC-01	Existe Security Foundation central
AC-02	Existe Secure Random abstraction
AC-03	Secure Random utiliza CSPRNG
AC-04	Existe PasswordHasher abstraction
AC-05	Password hashing utiliza Argon2id
AC-06	Los parámetros criptográficos no están hardcodeados arbitrariamente
AC-07	Existe constant-time comparison
AC-08	Existe Key Management boundary
AC-09	Se soporta separación de claves por propósito
AC-10	Existe SecurityContext
AC-11	SecurityContext es inmutable para operaciones críticas
AC-12	Existe Actor model
AC-13	Se soporta USER/SERVICE/SYSTEM/AI_AGENT
AC-14	Existe secret redaction
AC-15	Secrets no aparecen en exceptions
AC-16	Secrets no aparecen en logs/events
AC-17	Domain no depende de crypto frameworks
AC-18	Security dependencies utilizan DI
AC-19	Existen unit/security tests
AC-20	Existen architecture boundary tests
46. Definition of Done
[ ] Security package
[ ] SecureRandom
[ ] PasswordHasher
[ ] Argon2id adapter
[ ] Constant-time comparison
[ ] Key Management interface
[ ] Secret redaction
[ ] Security exceptions
[ ] Actor model
[ ] SecurityContext
[ ] Tenant context integration
[ ] Security metadata
[ ] Dependency injection
[ ] Unit tests
[ ] Security tests
[ ] Leakage tests
[ ] Architecture tests
[ ] Configuration integration
[ ] Documentation
47. Estructura propuesta

Al terminar esta tarea:

apps/api/app/
│
├── core/
│   ├── config/
│   │   ├── settings.py
│   │   └── validation.py
│   │
│   └── security/
│       ├── __init__.py
│       │
│       ├── hashing/
│       │   ├── interface.py
│       │   └── argon2.py
│       │
│       ├── random/
│       │   └── secure_random.py
│       │
│       ├── crypto/
│       │   └── key_management.py
│       │
│       ├── comparison/
│       │   └── secure_compare.py
│       │
│       ├── context/
│       │   ├── actor.py
│       │   └── security_context.py
│       │
│       ├── redaction/
│       │   └── redact.py
│       │
│       ├── exceptions.py
│       └── dependencies.py
│
└── domains/
    └── identity/
48. Trazabilidad
IS-001
 │
 └── TASK-001-11
       │
       ├── ESP-0001/03
       │     Authentication & Authorization
       │
       ├── ESP-0001/07
       │     MFA
       │
       ├── ESP-0001/14
       │     Implementation Requirements
       │
       ├── ADR-008
       │     Cryptographic Key Management
       │
       ├── ADR-014
       │     Encryption & Secret Protection
       │
       ├── ADR-023
       │     MFA Factor Strategy
       │
       ├── ADR-027
       │     Configuration, Secrets & KMS
       │
       ├── ADR-034
       │     Password Security
       │
       ├── ADR-035
       │     Credential Recovery
       │
       ├── ADR-037
       │     API Token Security
       │
       ├── ADR-038
       │     S2S Authentication
       │
       └── ADR-049
             Authorization Runtime
49. No debemos adelantar implementación funcional

Esta distinción es importante.

Después de TASK-001-11 tendremos:

Security primitives
        ✅

pero todavía no:

Authentication
        ❌

Authorization
        ❌

MFA
        ❌

Token lifecycle
        ❌

Esto mantiene el lifecycle que estamos siguiendo:

Blueprint
   ↓
ESP
   ↓
ADR
   ↓
IS
   ↓
Technical Tasks
   ↓
Security Foundation
   ↓
Authentication / Authorization / MFA
50. Estado de IS-001
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
TASK-001-11  Security Foundation          ✅
Próximo paso

TASK-001-12 — Tenant Context Foundation

Aquí vamos a construir la base para que EVOXA pueda manejar correctamente User → Organization → Membership → Active Tenant Context, incluyendo aislamiento tenant, resolución del contexto, validación de membership, prevención de cross-tenant access y las reglas que después utilizarán Authorization, RBAC, ABAC, Ownership, Delegation y los AI Agents.
