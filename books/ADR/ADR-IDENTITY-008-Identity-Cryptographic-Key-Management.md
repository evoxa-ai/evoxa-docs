ADR-IDENTITY-008 — Identity Cryptographic Key Management

Status: Proposed
Version: 1.0.0
Domain: Identity
Decision Type: Security / Architecture
Scope: JWT signing keys, key storage, key rotation, key distribution, encryption keys, MFA secrets, secret protection and cryptographic lifecycle
Related: ESP-0001 — Identity
Depends on:

ADR-IDENTITY-001 — Identity Architecture & Technology Decisions
ADR-IDENTITY-002 — JWT & Token Lifecycle
ADR-IDENTITY-003 — Refresh Token Security & Session Management
ADR-IDENTITY-006 — Identity Persistence Strategy
ADR-IDENTITY-007 — Identity Event & Outbox Strategy
1. Context

EVOXA Identity depende de material criptográfico para proteger diferentes capacidades de seguridad:

JWT access tokens;
autenticación;
refresh-token protection;
MFA;
recuperación de cuentas;
sesiones;
integraciones de identidad;
protección de secretos;
firma de eventos o webhooks cuando corresponda.

La arquitectura de seguridad de EVOXA establece protección criptográfica para datos en reposo y tránsito y contempla algoritmos como AES-256, TLS 1.3, Argon2id y mecanismos de firma asimétrica como Ed25519 / RSA-4096. Sin embargo, la arquitectura de referencia no fija una tecnología concreta de Key Management ni obliga a uno de esos algoritmos como decisión definitiva.

Por ello, esta ADR debe definir cómo se administrará el ciclo de vida de las claves, manteniendo abierta la decisión concreta del algoritmo cuando todavía no exista suficiente información para cerrarla.

2. Problem Statement

Las claves criptográficas son activos de seguridad críticos.

Un diseño inseguro podría almacenar:

JWT_PRIVATE_KEY
DATABASE_ENCRYPTION_KEY
MFA_SECRET

directamente en:

source code
.env
Git repository
Docker image
application logs
database

Esto no es aceptable.

Además, Identity necesita resolver:

generación;
almacenamiento;
distribución;
rotación;
versionado;
revocación;
backup;
recuperación;
acceso;
auditoría;
separación por ambiente;
respuesta ante compromiso.
3. Decision

Se adopta una estrategia de:

Centralized Cryptographic Key Management + External Secret Management + Key Separation + Rotation + Least Privilege + Auditability.

La aplicación Identity no será propietaria directa del almacenamiento físico de las claves maestras.

La arquitectura será conceptualmente:

                    Identity
                       │
                       ▼
              Cryptographic Service
                       │
             ┌─────────┴─────────┐
             ▼                   ▼
        Key Management       Secret Management
             │                   │
             ▼                   ▼
       Signing Keys        Application Secrets
       Encryption Keys     Credentials
       Key Metadata        API Secrets

La tecnología concreta de KMS/HSM/Secret Manager se decidirá posteriormente.

4. Key Management Principles

La gestión criptográfica seguirá estos principios:

Least Privilege
Key Separation
Secure by Default
No Hardcoded Secrets
No Secrets in Source Control
No Secrets in Logs
Controlled Rotation
Auditable Access
Environment Isolation
Revocation Capability
Minimal Key Exposure
Cryptographic Agility
5. Key Categories

Identity deberá distinguir diferentes tipos de material criptográfico.

Cryptographic Material
│
├── JWT Signing Keys
├── Encryption Keys
├── Secret Protection Keys
├── MFA Secret Protection
├── Integration Signing Keys
└── Webhook Signing Keys

No todas estas categorías deben compartir una misma clave.

6. JWT Signing Keys

Los access tokens JWT definidos en ADR-IDENTITY-002 utilizarán:

Asymmetric Signing

conceptualmente:

Private Key
     │
     ▼
JWT Issuer
     │
     ▼
Signed Access Token
     │
     ▼
Public Key
     │
     ▼
API / Gateway Validation

La clave privada debe permanecer bajo control exclusivo del componente autorizado para emitir tokens.

Los servicios consumidores no necesitan recibir la clave privada.

7. Private Key Protection

La clave privada de JWT:

JWT_PRIVATE_KEY

no debe:

estar en Git;
estar en el código;
incluirse en Docker images;
aparecer en logs;
enviarse mediante eventos;
almacenarse en texto plano en tablas de negocio.

El runtime debe obtener acceso a ella mediante el mecanismo seguro de gestión de secretos/claves.

8. Public Key Distribution

Las claves públicas podrán distribuirse a los componentes que necesitan validar JWT.

Conceptualmente:

Identity
   │
   ├── Private Key
   │
   └── Public Key
          │
          ├── API Gateway
          ├── Services
          └── Internal Consumers

Para JWT, se contempla utilizar un mecanismo compatible con JWKS para distribución y rotación de claves públicas.

9. Key Identifier

Cada clave de firma debe tener un identificador:

kid

Ejemplo:

kid = identity-key-2026-01

El JWT incluirá el kid correspondiente en su header.

Conceptualmente:

{
  "alg": "...",
  "kid": "identity-key-2026-01",
  "typ": "JWT"
}

Esto permite identificar qué clave pública debe utilizarse para validar el token.

10. Key Rotation

La rotación de claves será una capacidad obligatoria.

Conceptualmente:

Key A
  │
  ├── ACTIVE
  │
  ▼
Key B
  │
  ├── ACTIVE
  │
  ▼
Key A → RETIRED

Durante una transición puede existir:

Old Key → verification only
New Key → signing + verification

Esto permite evitar invalidar inmediatamente todos los JWT existentes.

11. Zero-Downtime Key Rotation

La rotación debe permitir:

Before
    Key A → sign

Transition
    Key A → verify
    Key B → sign + verify

After
    Key B → sign + verify
    Key A → retired

La clave anterior no debe eliminarse inmediatamente después de generar la nueva.

Debe permanecer disponible para validación durante una ventana controlada.

12. Key Lifecycle

Se define conceptualmente:

GENERATED
    ↓
STAGED
    ↓
ACTIVE
    ↓
ROTATION_PENDING
    ↓
VERIFY_ONLY
    ↓
RETIRED
    ↓
DESTROYED

Una clave comprometida puede seguir un camino excepcional:

ACTIVE
   ↓
COMPROMISED
   ↓
REVOKED
   ↓
DESTROYED
13. Compromised Key

Si una clave privada de signing se considera comprometida:

Compromise Detected
        ↓
Revoke Key
        ↓
Generate Replacement
        ↓
Activate Replacement
        ↓
Invalidate / Revoke affected sessions
        ↓
Security Investigation

La estrategia exacta para invalidar access tokens dependerá de su TTL y del mecanismo de revocación adoptado.

ADR-IDENTITY-002 establece que los access tokens son stateless y que la revocación inmediata de access tokens queda como una decisión abierta.

14. Cryptographic Algorithm

La arquitectura contempla como candidatos:

Ed25519
RSA-4096

pero este ADR no fija todavía uno como algoritmo definitivo.

La selección deberá considerar:

soporte de librerías;
interoperabilidad;
infraestructura;
JWKS;
rendimiento;
soporte de clientes;
requisitos de compliance;
capacidades del KMS.

Por tanto:

La selección definitiva entre Ed25519 y RSA-4096 queda como Open Decision.

15. Cryptographic Agility

La plataforma deberá evitar un diseño que haga imposible cambiar de algoritmo posteriormente.

Conceptualmente:

JWT Service
      │
      ▼
Signing Provider Interface
      │
      ├── Algorithm A
      └── Algorithm B

La lógica de negocio no deberá depender directamente de una implementación criptográfica específica.

16. Encryption Keys

Las claves utilizadas para cifrar información sensible deben mantenerse separadas de las claves de firma JWT.

Ejemplo:

JWT Signing Key
       ≠
Database Encryption Key
       ≠
MFA Protection Key
       ≠
Webhook Signing Key

El compromiso de una categoría no debe comprometer automáticamente las demás.

17. Envelope Encryption

Cuando corresponda, se recomienda utilizar un modelo de:

Master / Key Encryption Key
              ↓
       Data Encryption Key
              ↓
          Encrypted Data

Conceptualmente:

KMS
 │
 └── KEK
      │
      ▼
    DEK
      │
      ▼
Sensitive Data

La aplicación puede utilizar una DEK para cifrar datos y delegar la protección de la clave maestra al KMS.

La implementación concreta queda abierta.

18. MFA Secrets

Los secretos utilizados por MFA requieren protección especial.

Nunca deberán almacenarse:

TOTP_SECRET
RECOVERY_CODE

en logs ni exponerse mediante APIs normales.

Conceptualmente:

MFA Secret
    ↓
Encryption
    ↓
Protected Storage

El secreto sólo debe ser accesible al componente autorizado para realizar la operación criptográfica correspondiente.

19. Recovery Codes

Los recovery codes deben tratarse como secretos altamente sensibles.

No deberán almacenarse en texto plano después de su generación si la arquitectura de recuperación permite validarlos mediante hashes.

Conceptualmente:

Recovery Code
      ↓
Secure Hash
      ↓
Database

Durante la generación:

Plain Code
   ↓
Shown Once
   ↓
Never Logged

El usuario debe poder guardarlo, pero EVOXA no debería registrar el valor en texto plano.

20. Refresh Tokens

De acuerdo con ADR-IDENTITY-003, los refresh tokens no serán almacenados en texto plano.

Se utilizará:

Refresh Token
      ↓
Secure Representation / Hash
      ↓
PostgreSQL

Esto reduce el impacto de una exposición de la base de datos.

La protección de refresh tokens es distinta de la protección de las claves criptográficas maestras.

21. Passwords

Las contraseñas no deben ser cifradas para posteriormente recuperarlas.

Deben utilizar hashing seguro:

Password
   ↓
Argon2id
   ↓
Password Hash

Por tanto:

Password
   ≠
Encryption Key

La arquitectura de EVOXA contempla Argon2id para protección de passwords.

22. Secret Management

Los secretos de aplicación deberán mantenerse fuera del código fuente.

Ejemplos:

Database credentials
OIDC client secrets
SMTP credentials
External API secrets
KMS credentials
Webhook secrets

Flujo:

Application
    ↓
Secret Manager
    ↓
Secret

No:

Application
    ↓
.env committed to Git
23. Environment Isolation

Cada ambiente deberá utilizar material criptográfico independiente.

LOCAL
   ↓
Development Keys

DEV
   ↓
Development Keys

QA
   ↓
QA Keys

STAGING
   ↓
Staging Keys

PRODUCTION
   ↓
Production Keys

Una clave de Production nunca deberá reutilizarse en Development.

Especialmente:

Production JWT signing key

no debe existir en:

local developer machine
Git repository
QA
test fixtures
24. Access Control

El acceso a claves debe estar limitado por:

Identity
Role
Service
Environment
Operation

Ejemplo:

Token Issuer
    → SIGN JWT

API Gateway
    → VERIFY JWT

Developer
    → NO ACCESS

La regla general será:

Los componentes deben tener únicamente el acceso criptográfico mínimo que necesitan.

25. Human Access

El acceso humano directo a claves de producción debe ser excepcional.

Preferentemente:

Engineer
   ↓
Authorized Operational Procedure
   ↓
KMS / Secret Manager

y no:

Engineer
   ↓
Copy Private Key
   ↓
Laptop

Los accesos administrativos deben ser auditables.

26. Key Access Auditing

El acceso a material criptográfico debe generar trazabilidad cuando el sistema utilizado lo permita.

Debe poder identificarse:

who
what key
operation
when
environment
result

Nunca se deberá registrar:

private key value
secret value
plaintext token
27. Key Backup

Las claves críticas deberán formar parte de una estrategia de recuperación.

Pero:

Backup no significa copiar claves a ubicaciones arbitrarias.

Los backups deben conservar las mismas garantías de:

cifrado;
acceso restringido;
auditoría;
separación de ambientes;
control de acceso.

La estrategia específica de backup dependerá del KMS elegido.

28. Disaster Recovery

Identity debe poder recuperarse sin perder la capacidad de validar tokens durante una recuperación controlada.

Debe considerarse:

Database Recovery
       +
Key Management Recovery
       +
Secret Management Recovery

Una restauración de PostgreSQL sin disponibilidad de las claves necesarias puede dejar Identity inutilizable.

Por ello, las dependencias criptográficas forman parte del Disaster Recovery Plan.

29. Key Rotation and Database Data

Cuando una clave de cifrado sea rotada:

Old Key
   ↓
Decrypt
   ↓
Data
   ↓
Encrypt
   ↓
New Key

La rotación debe poder realizarse progresivamente cuando el volumen de datos lo requiera.

No se debe asumir que toda la base de datos puede cifrarse nuevamente en una única operación bloqueante.

30. Webhook Signing

Cuando Identity utilice webhooks firmados:

Identity
   ↓
Sign Payload
   ↓
Webhook
   ↓
External Consumer
   ↓
Verify Signature

La clave de firma del webhook debe ser independiente de:

JWT Signing Key

y deberá soportar rotación.

31. Event Signing

Si posteriormente se requiere firma criptográfica de determinados eventos de integración:

Event
  ↓
Signature
  ↓
Event Platform

la clave utilizada deberá ser independiente de las claves JWT.

No todos los eventos requieren necesariamente firma criptográfica; la necesidad debe establecerse por categoría de evento y amenaza.

32. Secret Exposure Prevention

El sistema deberá prevenir exposición accidental en:

Logs
Metrics
Traces
Audit
Events
Exceptions
Error Messages
API Responses

Por ejemplo, un error nunca debería devolver:

{
  "error": "Invalid JWT",
  "token": "eyJ..."
}

Debe utilizar información segura:

{
  "error": {
    "code": "INVALID_TOKEN",
    "message": "Authentication failed",
    "correlation_id": "corr_123"
  }
}
33. Application Architecture

La aplicación no deberá implementar directamente detalles criptográficos dispersos.

Se utilizarán abstracciones conceptuales:

CryptographicService
KeyProvider
SigningService
EncryptionService
SecretProvider

Ejemplo:

Application
     │
     ▼
SigningService
     │
     ▼
KeyProvider
     │
     ▼
KMS

Esto mantiene la arquitectura preparada para cambiar de proveedor.

34. Rotation Automation

La rotación deberá ser automatizable.

Conceptualmente:

Rotation Scheduler
       ↓
Generate Key
       ↓
Register Key
       ↓
Publish Public Key
       ↓
Activate
       ↓
Old Key → Verify Only
       ↓
Retire

La rotación manual debería reservarse para situaciones excepcionales.

35. Emergency Rotation

Debe existir un procedimiento para:

Emergency Key Rotation

cuando exista:

sospecha de compromiso;
pérdida de confidencialidad;
incidente de seguridad;
exposición accidental;
vulnerabilidad criptográfica;
compromiso de infraestructura.

El procedimiento deberá incluir:

Detect
 ↓
Contain
 ↓
Revoke
 ↓
Rotate
 ↓
Deploy
 ↓
Verify
 ↓
Investigate
36. JWT Validation During Rotation

Durante una rotación:

JWT(kid=A)
JWT(kid=B)

podrán coexistir temporalmente.

El validador debe poder obtener:

Public Key A
Public Key B

hasta que la ventana de transición finalice.

Esto evita invalidaciones masivas innecesarias.

37. Token Security Relationship

La estrategia completa queda:

                    Identity Security
                           │
          ┌────────────────┼────────────────┐
          ▼                ▼                ▼
       JWT Keys       Refresh Tokens       Sessions
          │                │                │
     Asymmetric         Hashed          Stateful
       Signing         Storage         Persistence
          │
          ▼
     Key Management

Esto mantiene separadas:

signing;
token persistence;
session state.
38. Performance

La gestión criptográfica no debe comprometer los objetivos generales de Identity:

Authentication P95 < 500 ms
API P95 < 300 ms

Para ello se podrá utilizar:

caching de claves públicas;
JWKS caching;
conexiones persistentes al KMS;
operaciones criptográficas eficientes;
rotación fuera de los paths críticos cuando sea posible.

La clave privada no debe recuperarse del KMS en cada request si la arquitectura del proveedor permite un mecanismo seguro de caching controlado.

39. Security Requirements

Son requisitos no negociables:

No hardcoded cryptographic keys.
No production keys in source control.
No private keys in logs.
No secrets in events.
No secrets in API responses.
Separation of key purposes.
Least-privilege key access.
Environment-specific keys.
Key rotation capability.
Emergency revocation capability.
Public/private key separation.
Auditable administrative access.
Secure backup/recovery.
Cryptographic agility.
Fail closed when cryptographic validation cannot be trusted.
40. Alternatives Considered
A. Keys in Environment Variables
Rechazada como estrategia principal

Aunque puede utilizarse para desarrollo local controlado, no es suficiente como mecanismo de gestión de claves de producción.

B. Keys Stored in PostgreSQL
Rechazada

La base de datos no debe convertirse en el almacén principal de las claves maestras que protegen al propio sistema.

C. Application-Owned Key Files
Rechazada

Introduce riesgos de:

exposición;
copia accidental;
backup inseguro;
distribución;
rotación manual.
D. External KMS / HSM / Secret Manager
Seleccionada

Permite:

centralización;
control de acceso;
auditoría;
rotación;
separación;
recuperación;
integración con infraestructura.

La tecnología concreta permanece abierta.

41. Consequences
Positivas
Reduce exposición de claves.
Facilita rotación.
Permite separación por propósito.
Mejora auditoría.
Facilita Disaster Recovery.
Permite cambiar de proveedor.
Reduce dependencia de archivos secretos.
Mejora la seguridad de JWT.
Prepara EVOXA para entornos Enterprise.
Negativas
Mayor complejidad de infraestructura.
Dependencia de un KMS/Secret Manager.
Costos adicionales.
Necesidad de diseñar correctamente IAM.
Necesidad de pruebas de recuperación.
Rotación requiere coordinación entre servicios.
42. Implementation Direction

La estructura conceptual podría ser:

apps/api/app/domains/identity/

└── infrastructure/
    └── security/
        ├── cryptography/
        │   ├── signing.py
        │   ├── encryption.py
        │   └── hashing.py
        │
        ├── keys/
        │   ├── provider.py
        │   ├── rotation.py
        │   └── jwks.py
        │
        └── secrets/
            └── provider.py

Y:

Application
     │
     ├── TokenService
     │
     ├── MFAService
     │
     └── WebhookService
              │
              ▼
      Cryptographic Services
              │
              ▼
       External KMS / Secret Manager

Esta estructura es una dirección de implementación derivada, no una estructura prescrita literalmente por el Blueprint.

43. Testing Requirements

Se deberán probar:

Key Generation
generación;
uniqueness;
metadata;
registration.
JWT
signing;
validation;
wrong key;
invalid signature;
expired key;
unknown kid.
Rotation
old key verification;
new key signing;
transition period;
retired key;
emergency rotation.
Secrets
unauthorized access;
secret redaction;
missing secret;
secret provider failure.
MFA
protected secret;
verification;
recovery;
secret exposure prevention.
Disaster Recovery
KMS unavailable;
key restoration;
service recovery;
JWT validation after recovery.
Security
key leakage;
privilege escalation;
unauthorized key access;
environment isolation.
44. Traceability
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
ESP-0001 / 09 Audit & Compliance
        ↓
ESP-0001 / 11 Persistence
        ↓
ESP-0001 / 12 Events & Integration
        ↓
ESP-0001 / 15 Deployment & Operations
        ↓
ADR-IDENTITY-008

La arquitectura de seguridad de EVOXA contempla explícitamente protección de datos en reposo y tránsito, además de Argon2id para passwords y mecanismos de firma asimétrica como Ed25519/RSA-4096 como opciones consideradas.

45. Follow-up ADRs

Se recomienda mantener las siguientes decisiones separadas:

ADR-IDENTITY-008.1

JWT Signing Algorithm

Definir finalmente:

Ed25519

o:

RSA-4096
ADR-IDENTITY-008.2

Key Management Provider

Definir:

KMS
HSM
Secret Manager

y proveedor concreto.

ADR-IDENTITY-008.3

JWT Key Rotation Policy

Definir:

rotation interval;
overlap period;
retirement period;
emergency rotation.
ADR-IDENTITY-008.4

MFA Secret Protection

Definir:

encryption mechanism;
KMS integration;
secret access;
rotation.
ADR-IDENTITY-008.5

Production Secret Management

Definir:

secret manager;
access policies;
injection;
rotation;
audit.
46. Final Decision Summary

La estrategia queda:

                     IDENTITY
                        │
                        ▼
               Cryptographic Layer
                        │
        ┌───────────────┼────────────────┐
        ▼               ▼                ▼
    JWT Signing      Encryption       MFA Secrets
        │               │                │
        └───────────────┼────────────────┘
                        ▼
               Key Management Layer
                        │
              ┌─────────┴─────────┐
              ▼                   ▼
             KMS              Secret Manager
              │                   │
              ▼                   ▼
       Key Lifecycle        Secret Lifecycle
              │
       ┌──────┼────────┐
       ▼      ▼        ▼
   Rotation Revocation Audit
Decisión principal

EVOXA Identity utilizará gestión centralizada y externa de claves y secretos, con separación criptográfica por propósito, acceso basado en mínimo privilegio, aislamiento por ambiente, rotación controlada, capacidad de revocación de emergencia y auditoría. Las claves privadas de firma JWT nunca serán almacenadas en código, repositorios, imágenes de aplicación ni bases de datos de negocio. La arquitectura será criptográficamente agnóstica para permitir seleccionar posteriormente el algoritmo y proveedor de KMS más apropiados.

Estado: PROPOSED.

Decisiones que quedan abiertas
1. Ed25519 vs RSA-4096
2. KMS/HSM/Secret Manager concreto
3. Frecuencia de rotación
4. Ventana de overlap de claves
5. Política de emergencia
6. Protección específica de secretos MFA
7. Estrategia definitiva de secret injection

Esto deja ADR-IDENTITY-008 listo como marco de seguridad sin cerrar prematuramente decisiones de infraestructura que todavía deben definirse en ADRs específicos.
