ADR-IDENTITY-014 — Encryption & Secret Protection

Estado: Proposed
Versión: 1.0
Fecha: 2026-09-02
Dominio: Identity
Prioridad: Critical
Tipo: Security / Architecture Decision Record

1. Título

ADR-IDENTITY-014 — Encryption & Secret Protection

2. Contexto

El dominio Identity es uno de los componentes con mayor sensibilidad de seguridad de EVOXA. Maneja información y credenciales cuyo compromiso puede permitir acceso no autorizado a usuarios, organizaciones, sesiones y recursos protegidos.

El Blueprint establece principios de:

Zero Trust.
Least Privilege.
Defense in Depth.
Secure by Design.
Privacy by Design.
Protección de secretos.
Gestión segura de credenciales.
Tokens de acceso y refresh.
MFA.
Auditoría y trazabilidad.
Gestión centralizada de claves.
No exposición de secretos en logs ni eventos.

La arquitectura de Identity contempla además:

PostgreSQL como fuente transaccional.
Redis como infraestructura auxiliar.
Event Platform.
Transactional Outbox.
JWT con firma asimétrica.
Refresh Tokens persistidos y rotados.
MFA.
Recovery Codes.
Trusted Devices.
Audit Events.
Security Events.

Por lo tanto, no es suficiente con definir que determinados datos "deben estar cifrados". Es necesario establecer una estrategia coherente para:

datos en tránsito;
datos almacenados;
secretos de autenticación;
claves criptográficas;
tokens;
secretos MFA;
recovery codes;
credenciales de infraestructura;
claves de firma JWT;
logs, eventos y auditoría;
rotación y revocación;
acceso de servicios y operadores.
3. Problema

Sin una política criptográfica centralizada, diferentes componentes podrían implementar mecanismos incompatibles o inseguros.

Ejemplos:

almacenar passwords de forma reversible;
almacenar tokens en texto plano;
guardar claves JWT dentro del código;
incluir secretos en variables de entorno permanentes;
registrar tokens en logs;
almacenar TOTP secrets sin protección;
enviar información sensible mediante eventos;
reutilizar una misma clave criptográfica para múltiples propósitos;
no disponer de mecanismos de rotación;
perder capacidad de revocar una clave comprometida;
permitir acceso directo de múltiples servicios a claves maestras;
cifrar datos sin una estrategia de gestión de claves.

Esto produciría un modelo de seguridad fragmentado y dificultaría la auditoría, rotación y respuesta ante incidentes.

4. Decisión

EVOXA adoptará una estrategia de Encryption & Secret Protection centralizada, basada en separación de responsabilidades, gestión externa de secretos y claves, criptografía adecuada para cada tipo de dato y rotación controlada.

La estrategia se divide en:

Encryption in Transit
Encryption at Rest
Password Protection
Token Protection
MFA Secret Protection
Cryptographic Key Management
Secret Management
JWT Signing Keys
Event and Log Protection
Key Rotation & Revocation
Access Control
Incident Response
5. Principio fundamental

La arquitectura distinguirá explícitamente entre:

Datos que deben cifrarse

Información que debe poder recuperarse posteriormente y cuyo contenido requiere confidencialidad.

Ejemplos conceptuales:

determinados datos sensibles;
MFA secrets;
información sensible de dispositivos;
credenciales de integración cuando corresponda.
Datos que deben ser hasheados

Información que no necesita recuperarse en forma original.

Ejemplo principal:

Password
    ↓
Argon2id
    ↓
Password Hash

El sistema nunca debe necesitar descifrar un password.

Datos que deben protegerse mediante hashing/HMAC u otra representación no reversible

Principalmente tokens y valores que solamente necesitan ser comparados o invalidados.

Esto se aplica especialmente a:

Refresh Tokens;
recovery codes;
identificadores secretos derivados;
otros secretos cuya recuperación no sea necesaria.

La elección exacta del mecanismo criptográfico para Refresh Tokens se detallará en:

ADR-IDENTITY-025 — Refresh Token Cryptography & Concurrency.

6. Encryption in Transit

Toda comunicación que transporte información de Identity deberá utilizar canales protegidos.

La arquitectura deberá utilizar:

Client
   │
   │ TLS
   ▼
API Gateway / API
   │
   │ TLS cuando corresponda
   ▼
Identity
   │
   ├── PostgreSQL
   ├── Redis
   ├── Event Platform
   └── Secret / Key Management

No se permitirá transmitir credenciales, tokens o información sensible mediante canales no protegidos.

Aplicación

Debe contemplarse protección para:

Web → API
Mobile → API
API → PostgreSQL
API → Redis cuando corresponda
API → Event Platform cuando corresponda
API → Secret Manager
API → KMS
servicios internos → servicios internos

Los detalles de certificados, mTLS y políticas concretas de red serán definidos en los ADR correspondientes de infraestructura y runtime.

7. Encryption at Rest

Los sistemas de almacenamiento deberán disponer de mecanismos de protección de datos almacenados.

Esto incluye, según corresponda:

PostgreSQL;
backups;
snapshots;
almacenamiento de objetos;
Redis persistence si está habilitada;
Event Platform;
archivos generados;
artefactos que contengan información sensible.

La arquitectura deberá distinguir entre:

Encryption at Rest
        +
Application-Level Encryption
        +
Hashing
        +
Access Control

No se considerará que cifrar el disco sustituye la protección de secretos a nivel de aplicación.

8. Password Protection

Los passwords serán protegidos mediante Argon2id, de acuerdo con la definición establecida para Identity.

El flujo será:

Password ingresado
       ↓
Argon2id
       ↓
Password Hash
       ↓
PostgreSQL

Nunca se almacenará:

password

en texto plano.

Tampoco se deberá almacenar:

password

mediante cifrado reversible como sustituto del hashing.

El password hash:

no debe aparecer en logs;
no debe aparecer en eventos;
no debe aparecer en respuestas API;
no debe formar parte del JWT;
no debe ser enviado a servicios que no lo necesiten.
9. JWT Signing Keys

Los Access Tokens JWT serán firmados utilizando criptografía asimétrica.

Conceptualmente:

                Private Key
                    │
                    ▼
Identity ───────► Sign JWT
                    │
                    ▼
                 Access Token
                    │
                    ▼
              ┌─────────────┐
              │ Other APIs   │
              └─────────────┘
                    │
                    ▼
              Public Key
                 Verify

La clave privada:

nunca estará en el código fuente;
nunca estará en Git;
nunca estará en imágenes Docker;
nunca estará en logs;
nunca será enviada a otros servicios innecesariamente.

Las claves públicas podrán distribuirse mediante un mecanismo como JWKS, utilizando identificadores kid.

10. Separación de claves

No se utilizará una única clave maestra para todos los propósitos.

Deberán existir dominios criptográficos separados conceptualmente para:

JWT Signing
      │
      ├── Access Token Signing

MFA
      │
      └── MFA Secret Protection

Data Encryption
      │
      └── Application Sensitive Data

Infrastructure
      │
      └── Service / Integration Secrets

Webhooks / Integrations
      │
      └── Signing / Verification

Una vulneración de una clave no deberá implicar automáticamente el compromiso de todos los demás dominios criptográficos.

11. Key Management System

Las claves criptográficas críticas deberán gestionarse mediante un mecanismo especializado de gestión de claves.

Conceptualmente:

Identity
   │
   ▼
Secret / Key Management Layer
   │
   ├── KMS
   ├── Secret Manager
   └── Key Rotation

El Blueprint establece la necesidad de gestión centralizada de claves y secretos.

La tecnología concreta —por ejemplo, un KMS/Secret Manager específico del proveedor cloud— queda pendiente de definición en el ADR de infraestructura correspondiente.

12. Secret Management

Los secretos no deberán residir permanentemente en:

código fuente;
repositorios Git;
archivos versionados;
Dockerfiles;
imágenes Docker;
manifests públicos;
logs;
eventos;
respuestas API.

Ejemplos de secretos:

DATABASE_PASSWORD
REDIS_PASSWORD
JWT_PRIVATE_KEY
MFA_ENCRYPTION_KEY
SERVICE_CREDENTIAL
WEBHOOK_SECRET
API_SECRET

deberán ser gestionados externamente.

13. Variables de entorno

Las variables de entorno podrán utilizarse como mecanismo de inyección runtime, pero no se considerarán por sí mismas un sistema de gestión de secretos.

Por ejemplo:

SECRET_MANAGER
      ↓
Runtime
      ↓
Environment / Secret Mount
      ↓
Application

La arquitectura deberá evitar que los secretos se incorporen al artefacto de despliegue.

14. Refresh Tokens

Los Refresh Tokens serán tratados como secretos de alta sensibilidad.

La arquitectura establecida previamente define:

refresh tokens stateful;
rotación;
token families;
reuse detection;
revocación;
asociación a sesiones.

El sistema no deberá almacenar innecesariamente el Refresh Token original en texto plano.

Su protección criptográfica específica, incluyendo:

algoritmo;
hashing;
HMAC;
representación persistida;
comparación;
rotación;
concurrencia;
reuse detection;

será definida formalmente en:

ADR-IDENTITY-025 — Refresh Token Cryptography & Concurrency.

15. MFA Secrets

Los secretos asociados a factores MFA deberán recibir protección especial.

Por ejemplo, si se utiliza TOTP:

TOTP Secret
     ↓
Protected Storage
     ↓
MFA Verification

El secreto no deberá:

aparecer en logs;
aparecer en eventos;
aparecer en respuestas posteriores al enrollment;
almacenarse innecesariamente en texto plano.

El sistema deberá separar:

MFA Secret

de:

MFA Metadata

La estrategia exacta de cifrado, rotación, recuperación y lifecycle de factores se complementará con:

ADR-IDENTITY-023 — MFA Factor Strategy

y

ADR-IDENTITY-024 — MFA TTL, Attempts & Recovery.

16. Recovery Codes

Los Recovery Codes son secretos de uso limitado y alta sensibilidad.

El sistema deberá evitar almacenar una representación recuperable innecesariamente.

Conceptualmente:

Recovery Code
      ↓
Secure Representation
      ↓
Database

Una vez utilizado:

USED
 ↓
No reusable

Los logs y auditorías podrán registrar que un recovery code fue utilizado, pero nunca el código mismo.

17. Trusted Devices

Los mecanismos asociados a Trusted Devices deberán evitar almacenar secretos innecesarios.

La información del dispositivo podrá incluir metadata necesaria para:

identificación;
estado;
expiración;
revocación;
auditoría;
evaluación de riesgo.

Pero no deberá utilizarse metadata del dispositivo como sustituto de un mecanismo criptográfico seguro.

18. Logs

Los logs deberán aplicar una política estricta de secret redaction.

Nunca deberán registrarse:

password
password_hash
access_token
refresh_token
JWT
TOTP secret
OTP
recovery code
private key
encryption key
client secret
database password

Ejemplo prohibido:

User login:
email=user@example.com
password=...
access_token=eyJ...

Ejemplo aceptable conceptualmente:

Authentication succeeded
user_id=...
session_id=...
correlation_id=...

La observabilidad debe proporcionar trazabilidad sin convertir los logs en una fuente secundaria de secretos.

19. Events

La misma regla se aplicará a:

Domain Events;
Integration Events;
Security Events;
Audit Events;
Outbox Events.

Los eventos deberán transportar únicamente la información necesaria.

Especialmente prohibido:

password
password_hash
refresh_token
access_token
TOTP secret
recovery code
private key

Esto es consistente con la arquitectura de eventos y el principio de minimizar información sensible.

20. Audit

La auditoría podrá registrar:

who
what
when
where
result
resource
correlation_id
trace_id

pero no deberá registrar el secreto que produjo la operación.

Por ejemplo:

RefreshTokenReuseDetected
user_id
session_id
token_family_id
timestamp
correlation_id

es aceptable conceptualmente.

Registrar:

refresh_token=abc123...

no lo es.

21. Key Rotation

Todas las claves críticas deberán contemplar un ciclo de vida.

Conceptualmente:

ACTIVE
   ↓
ROTATING
   ↓
NEW KEY ACTIVE
   ↓
OLD KEY RETIRED
   ↓
OLD KEY DESTROYED

La rotación deberá considerar:

identificación de versiones;
periodo de transición;
compatibilidad;
validación;
rollback;
revocación;
auditoría.

Para JWT se utilizará conceptualmente:

kid

para identificar la clave utilizada.

Durante una transición puede existir:

Key A → verify
Key B → sign

hasta completar la migración.

22. Key Revocation

Una clave comprometida deberá poder retirarse.

El procedimiento deberá permitir:

Detect compromise
       ↓
Identify key
       ↓
Revoke / retire
       ↓
Issue replacement
       ↓
Update consumers
       ↓
Audit incident

La estrategia exacta dependerá del tipo de clave.

23. Access Control sobre claves

El acceso a claves deberá seguir:

Least Privilege.

Un servicio que solamente necesita validar JWT no debería tener acceso a la clave privada utilizada para firmarlos.

Por ejemplo:

Identity
   │
   └── private signing capability

API Consumer
   │
   └── public verification key

De esta manera se reduce el blast radius ante compromiso de servicios.

24. Separation of Duties

Cuando sea posible, se deberá separar:

desarrollo;
despliegue;
administración de secretos;
administración de claves;
operación;
auditoría.

Ningún componente debería disponer de privilegios criptográficos superiores a los necesarios para realizar su función.

25. Backup y recuperación

Las claves y secretos críticos deberán formar parte de una estrategia segura de continuidad operacional.

No se deberá asumir que:

Database Backup

equivale automáticamente a:

Cryptographic Recovery

La estrategia deberá garantizar que la recuperación de infraestructura pueda recuperar los datos protegidos sin crear una nueva vulnerabilidad.

Esto deberá alinearse con:

RTO < 1 hora;
RPO < 15 minutos.
26. Threat Model

Esta arquitectura busca reducir principalmente:

Amenaza	Protección
Password leak	Argon2id
Token database leak	Secure representation
JWT private-key leak	KMS + external secret management
Log leakage	Redaction
Event leakage	Payload minimization
DB compromise	Encryption at rest + protected secrets
Network interception	TLS
Key compromise	Rotation + revocation
Service compromise	Least privilege
Cross-service secret exposure	Key separation
Insider access	Access control + audit
Backup compromise	Encrypted backups
Credential reuse	Token rotation / MFA
27. Consecuencias positivas
Seguridad

Se obtiene una estrategia criptográfica consistente para todo Identity.

Menor blast radius

El compromiso de una clave no implica automáticamente el compromiso de todas.

Rotación

Las claves podrán evolucionar sin rediseñar todo el sistema.

Auditoría

Las operaciones sensibles mantienen trazabilidad sin registrar secretos.

Compliance

La arquitectura queda preparada para requisitos futuros de seguridad y privacidad.

Cloud readiness

La estrategia es compatible con KMS y Secret Managers administrados.

Zero Trust

Cada componente recibe únicamente las capacidades criptográficas necesarias.

28. Consecuencias negativas

La estrategia introduce mayor complejidad operacional:

KMS/Secret Manager;
rotación;
permisos;
recuperación;
bootstrap;
configuración;
observabilidad;
testing criptográfico.

También requiere definir cuidadosamente:

políticas de acceso;
ciclos de vida;
recuperación;
rotación;
compatibilidad entre versiones.

La seguridad criptográfica pasa a ser una capacidad operacional permanente, no una configuración puntual.

29. Alternativas consideradas
Alternativa A — Guardar secretos en .env

Rechazada.

Puede utilizarse únicamente como mecanismo de desarrollo controlado, nunca como estrategia definitiva de producción.

Alternativa B — Guardar claves en PostgreSQL

Rechazada para claves maestras.

Crear un círculo donde la base de datos contiene tanto los datos protegidos como las claves para protegerlos reduce significativamente la separación de seguridad.

Alternativa C — Una única clave para todo

Rechazada.

Incrementaría el blast radius de un compromiso.

Alternativa D — Cifrar absolutamente todo

Rechazada.

No todo dato necesita cifrado reversible.

Debe distinguirse entre:

Hash
Encryption
HMAC
Token protection
Key management

según el propósito.

Alternativa E — Confiar solamente en Encryption at Rest

Rechazada.

No protege adecuadamente contra:

acceso autorizado indebido;
logs;
eventos;
respuestas API;
servicios comprometidos;
exposición accidental de secretos.
30. Non-Negotiables

Las siguientes reglas son obligatorias:

Nunca almacenar passwords en texto plano.
Passwords mediante Argon2id.
Nunca almacenar secretos en código fuente.
Nunca almacenar claves privadas en Git.
Nunca registrar tokens o secretos en logs.
Nunca incluir secretos en eventos.
JWT mediante firma asimétrica.
Claves críticas gestionadas externamente.
Separación de claves por propósito.
Least Privilege sobre acceso criptográfico.
Rotación de claves soportada.
Revocación de claves soportada.
Refresh Tokens protegidos y no almacenados innecesariamente en texto plano.
MFA secrets protegidos.
Recovery Codes protegidos y de un solo uso.
Encryption at Rest para almacenamiento aplicable.
TLS para comunicaciones protegidas.
Secret redaction obligatorio.
No utilizar una clave maestra única para toda la plataforma.
Las claves no deben estar dentro de las imágenes de despliegue.
31. Traceability
Blueprint

Security Architecture

Zero Trust
Defense in Depth
Secure by Design
Privacy by Design
Secret Management
Cryptographic Key Management
ESP

ESP-0001 — Identity Domain

Relacionada con:

Authentication
Session Management
Refresh Tokens
MFA
Security Policy
Audit
Identity Infrastructure
ADR relacionados
ADR-IDENTITY-001 — Identity Domain Architecture
ADR-IDENTITY-002 — JWT Access Token Strategy
ADR-IDENTITY-003 — Stateful Sessions & Refresh Tokens
ADR-IDENTITY-005 — Authorization Model
ADR-IDENTITY-006 — Persistence Architecture
ADR-IDENTITY-007 — Event Architecture
ADR-IDENTITY-008 — Cryptographic Key Management
ADR-IDENTITY-009 — Token Storage
ADR-IDENTITY-010 — Identity Testing & Quality
ADR-IDENTITY-012 — PostgreSQL Schema
ADR-IDENTITY-013 — Data Retention
Implementation Stories

Principalmente:

IS-IDENTITY-005 — Authentication
IS-IDENTITY-008 — Session Management
IS-IDENTITY-009 — Refresh Token Lifecycle
IS-IDENTITY-010 — MFA
IS-IDENTITY-011 — Security Policy Engine
IS-IDENTITY-012 — Audit & Security Events
IS-IDENTITY-013 — Domain Events & Outbox
IS-IDENTITY-015 — Identity Persistence
IS-IDENTITY-017 — Runtime & Deployment
32. Dependencias

Este ADR depende o interactúa con:

ADR-IDENTITY-008
Cryptographic Key Management.

ADR-IDENTITY-025
Refresh Token Cryptography & Concurrency.

ADR-IDENTITY-023
MFA Factor Strategy.

ADR-IDENTITY-024
MFA TTL, Attempts & Recovery.

ADR-IDENTITY-027
Runtime, Secrets & KMS.

ADR-IDENTITY-028
Observability & Operational Security.

33. Decisiones pendientes

Este ADR establece la arquitectura, pero deja deliberadamente abiertas decisiones que requieren ADRs específicos:

Criptografía
algoritmos concretos para encryption at rest;
algoritmos de application-level encryption;
HMAC cuando corresponda;
tamaños de claves;
parámetros criptográficos.
KMS
proveedor;
arquitectura de key hierarchy;
master keys;
data encryption keys;
permisos;
rotation schedules.
JWT
algoritmo concreto de firma;
JWKS;
kid;
periodo de coexistencia durante rotación.
MFA
cifrado concreto de TOTP secrets;
protección de WebAuthn/passkey metadata;
recovery-code representation.
Refresh Tokens
estrategia exacta de hashing/HMAC;
comparación;
protección ante ataques offline;
manejo de familias.
Runtime
Secret Manager;
secret injection;
Kubernetes Secrets;
workload identity;
acceso de pods/servicios a KMS.

Estas decisiones no deben inventarse dentro de este ADR porque corresponden a decisiones especializadas posteriores.

34. Criterios de aceptación

ADR-IDENTITY-014 podrá considerarse Accepted cuando exista definición formal de:

 estrategia de Encryption at Rest;
 estrategia de Application-Level Encryption;
 estrategia de hashing;
 gestión de JWT signing keys;
 KMS;
 Secret Manager;
 key hierarchy;
 key rotation;
 key revocation;
 MFA secret protection;
 Refresh Token cryptography;
 recovery-code protection;
 secret redaction;
 event secret protection;
 backup encryption;
 least-privilege cryptographic access;
 disaster recovery de claves;
 auditoría de operaciones criptográficas;
 pruebas de seguridad criptográfica.
35. Próximos ADR relacionados

La secuencia lógica queda:

ADR-IDENTITY-014
Encryption & Secret Protection
        │
        ├── ADR-IDENTITY-015
        │   Database Partitioning
        │
        ├── ADR-IDENTITY-016
        │   Concurrency & Transaction Strategy
        │
        ├── ADR-IDENTITY-017
        │   Database Migration Strategy
        │
        ├── ADR-IDENTITY-023
        │   MFA Factor Strategy
        │
        ├── ADR-IDENTITY-024
        │   MFA TTL / Attempts / Recovery
        │
        ├── ADR-IDENTITY-025
        │   Refresh Token Cryptography
        │
        ├── ADR-IDENTITY-027
        │   Runtime / Secrets / KMS
        │
        └── ADR-IDENTITY-028
            Observability / Operational Security
36. Decisión final

ADR-IDENTITY-014 queda en estado Proposed.

EVOXA adoptará Encryption & Secret Protection como una capacidad transversal de seguridad, separando explícitamente hashing, encryption, token protection y key management.

La arquitectura establece como principios fundamentales:

Los secretos no son datos normales.

Las claves no deben almacenarse junto con aquello que protegen cuando eso pueda evitarse.

Cada secreto debe tener el mínimo acceso necesario.

La criptografía debe poder rotarse, revocarse y auditarse.

La observabilidad nunca debe convertirse en un canal de fuga de secretos.

Esto deja preparada la base para continuar con ADR-IDENTITY-015 — Database Partitioning, manteniendo la numeración y la línea arquitectónica que ya venimos consolidando.
