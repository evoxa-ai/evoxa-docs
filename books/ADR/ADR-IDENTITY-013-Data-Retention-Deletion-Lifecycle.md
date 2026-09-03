ADR-IDENTITY-013 — Data Retention, Deletion & Lifecycle

Estado: Proposed
Versión: 1.0
Fecha: 2026-09-02
Dominio: Identity
Prioridad: Critical
Tipo: Architecture Decision Record

1. Contexto

El dominio Identity mantiene información crítica para la seguridad y operación de EVOXA:

Users
Organizations
Memberships
Roles
Permissions
Sessions
Refresh Tokens
MFA Factors
MFA Challenges
Recovery Codes
Trusted Devices
Security Policies
Audit Events
Security Events
Outbox Events

Identity no solamente necesita almacenar información; debe controlar correctamente cuánto tiempo permanece disponible, cuándo deja de estar activa, cuándo puede anonimizarse y cuándo puede eliminarse.

El Blueprint establece que Identity es el dominio central de confianza de EVOXA y que la seguridad, disponibilidad, confiabilidad y auditabilidad son atributos críticos.

Además, los eventos de auditoría son definidos como inmutables, por lo que su ciclo de vida no puede tratarse igual que el de una entidad operacional normal.

2. Problema

Si no existe una estrategia explícita de lifecycle y retención, pueden aparecer dos problemas opuestos.

Retención excesiva
Datos innecesarios
       ↓
Mayor superficie de ataque
       ↓
Mayor exposición de PII
       ↓
Mayor costo
       ↓
Mayor complejidad de cumplimiento
Eliminación prematura
Datos eliminados
       ↓
Pérdida de trazabilidad
       ↓
Pérdida de evidencia de seguridad
       ↓
Problemas de auditoría
       ↓
Problemas de investigación

Por lo tanto, Identity necesita diferenciar entre:

datos operacionales;
datos de seguridad;
datos de auditoría;
datos temporales;
secretos;
eventos históricos.
3. Decisión

Identity adoptará una estrategia de lifecycle explícito + retención por categoría de dato.

La regla general será:

Los datos no se eliminan simplemente porque dejan de estar activos. Su estado funcional y su retención son conceptos independientes.

La arquitectura será:

ACTIVE
   │
   ▼
INACTIVE / SUSPENDED / REVOKED / EXPIRED
   │
   ▼
RETENTION PERIOD
   │
   ├── RETAIN
   ├── ARCHIVE
   ├── ANONYMIZE
   └── DELETE

La acción final dependerá de la naturaleza del dato y de las obligaciones aplicables.

4. Principio fundamental: Lifecycle ≠ Retention

Se establece una separación explícita:

Lifecycle
=
¿Está activo?

Retention
=
¿Debemos conservarlo?

Por ejemplo:

User
DISABLED
   +
Retention still active

significa que el usuario ya no puede utilizar la cuenta, pero sus datos todavía pueden necesitar conservarse.

Otro ejemplo:

Session
REVOKED
   +
Retention period active

La sesión ya no permite acceso, pero puede mantenerse para auditoría o análisis de seguridad.

5. Clasificación de datos

Identity utilizará inicialmente estas categorías:

Categoría	Ejemplos	Tratamiento
Operacional	User, Organization, Membership	Lifecycle + retención
Seguridad	Sessions, Security Events	Retención controlada
Auditoría	Audit Events	Inmutable + retención
Temporal	MFA Challenges, estados temporales	Retención corta
Credenciales	Refresh Tokens, MFA secrets	Protección + lifecycle
Configuración	Roles, Permissions, Policies	Lifecycle + versionado
Integración	Outbox Events	Retención operacional + publicación

Esta clasificación será utilizada posteriormente para definir políticas concretas.

6. User Lifecycle

El User seguirá un lifecycle explícito:

PENDING
   │
   ▼
ACTIVE
   │
   ├──► SUSPENDED
   │       │
   │       └──► ACTIVE
   │
   └──► DISABLED

Una cuenta DISABLED no deberá autenticarse.

Sin embargo:

DISABLED ≠ DELETE

El registro puede permanecer disponible durante el período de retención correspondiente.

7. Organization Lifecycle
PENDING
   │
   ▼
ACTIVE
   │
   ▼
SUSPENDED
   │
   └──► ACTIVE

Una organización suspendida no proporciona acceso efectivo a sus recursos.

Su información puede seguir siendo necesaria para:

auditoría;
facturación futura;
trazabilidad;
investigación;
cumplimiento.

Por ello:

SUSPENDED ≠ DELETE
8. Membership Lifecycle
INVITED
   │
   ├──► ACTIVE
   └──► REMOVED

ACTIVE
   │
   ├──► SUSPENDED
   └──► REMOVED

SUSPENDED
   │
   └──► ACTIVE / REMOVED

Una Membership REMOVED deja de conceder acceso.

Pero su existencia histórica puede ser necesaria para responder:

¿Quién tuvo acceso a esta organización y durante qué período?

Por ello, no se eliminará automáticamente al cambiar a REMOVED.

9. Session Lifecycle

Las sesiones son información de seguridad de duración limitada.

CREATED
   │
   ▼
ACTIVE
   │
   ├──► EXPIRED
   └──► REVOKED

Una sesión EXPIRED o REVOKED no puede volver a ser utilizada.

Posteriormente podrá pasar a:

Retention
   ↓
Archive / Delete

dependiendo de las necesidades de auditoría y seguridad.

10. Refresh Token Lifecycle

Los Refresh Tokens tienen un tratamiento especial.

ACTIVE
   │
   ├──► USED
   ├──► EXPIRED
   └──► REVOKED

Un token USED no vuelve a ser válido.

Un token EXPIRED no puede renovarse.

Un token REVOKED no puede utilizarse.

La información necesaria para investigar:

rotation;
reuse;
revocation;
token family;

podrá necesitar mantenerse durante un período de seguridad.

El tratamiento criptográfico detallado se mantiene en:

ADR-IDENTITY-025 — Refresh Token Cryptography & Concurrency.

11. MFA Lifecycle

Los factores MFA:

PENDING
   │
   ▼
ACTIVE
   │
   ▼
REVOKED

Los challenges son temporales:

PENDING
   │
   ├──► VERIFIED
   ├──► FAILED
   ├──► EXPIRED
   └──► REVOKED

Por su naturaleza, los challenges podrán tener una retención mucho más corta que los Audit Events.

El Blueprint no establece un TTL concreto, por lo que este ADR no inventa uno.

12. Recovery Codes

Los recovery codes son credenciales sensibles.

Una vez utilizados:

AVAILABLE
   ↓
USED

No deben conservarse en forma recuperable una vez que dejan de ser necesarios.

Además:

Recovery Code
      ≠
Audit Record

La auditoría puede registrar que un recovery mechanism fue utilizado, pero no debe almacenar el código.

13. Trusted Devices

Los Trusted Devices deberán tener lifecycle propio:

ACTIVE
   │
   ├──► EXPIRED
   └──► REVOKED

Cuando un dispositivo deja de ser confiable:

REVOKED

no significa necesariamente eliminación inmediata del registro.

La retención dependerá de las necesidades de seguridad.

14. Roles y Permissions

Roles y permissions son configuración de autorización.

No deberán desaparecer simplemente porque dejan de estar activos.

Ejemplo:

Role
ACTIVE
   ↓
INACTIVE

Mantener el historial puede ser importante para responder:

¿Qué permiso tenía un usuario cuando realizó determinada operación?

Por ello, los cambios de roles y permisos deberán quedar auditados.

15. Security Policies

Las políticas deberán ser versionadas.

Ejemplo:

Policy v1
   ↓
Policy v2
   ↓
Policy v3

Una política antigua puede dejar de estar activa:

ACTIVE
   ↓
RETIRED

pero su versión histórica puede necesitar conservarse para comprender decisiones de autorización realizadas en el pasado.

Esto es especialmente importante porque las decisiones de autorización deben poder asociarse con la política/version que las produjo.

16. Audit Events

Los audit_events tienen un tratamiento especial.

Regla

Audit Events son append-only e inmutables.

Created
   ↓
Stored
   ↓
Retained
   ↓
Archived
   ↓
Disposed according to policy

No se permitirá:

UPDATE audit_event

como operación normal.

Tampoco:

DELETE audit_event

como parte de una operación de negocio.

El Blueprint define explícitamente la inmutabilidad de los audit events.

17. Security Events

Los security_events deberán conservarse para permitir:

detección;
investigación;
respuesta ante incidentes;
análisis;
correlación;
seguridad operacional.

Su lifecycle será:

Generated
   ↓
Stored
   ↓
Processed
   ↓
Retained
   ↓
Archived / Disposed

El período exacto queda pendiente.

18. Outbox Events

Los outbox_events tienen una naturaleza diferente.

Su función principal es garantizar:

Database Transaction
       +
Event Publication

Una vez que el evento ha sido publicado correctamente y ya no es necesario para recuperación operacional inmediata, podrá pasar al proceso de retención/archivo.

PENDING
   ↓
PROCESSING
   ↓
PUBLISHED
   ↓
RETENTION
   ↓
ARCHIVE / DELETE

El Blueprint exige que el event lifecycle contemple publicación, consumo y archivado, y que el replay no modifique eventos históricos.

19. Secrets

Los secretos tienen una regla distinta.

No se aplicará:

Secret
   ↓
Long-term retention

cuando no exista una necesidad explícita.

Ejemplos:

password;
MFA secret;
recovery code;
refresh token utilizable;
private key.

Deben mantenerse solamente mientras sean necesarios y siempre mediante representación segura.

Nunca deberán aparecer en:

logs
audit_events
security_events
outbox_events
error messages
20. Anonymization

Cuando una obligación de negocio o legal requiera mantener información histórica pero ya no sea necesario conservar PII identificable, podrá utilizarse:

PII
 ↓
Anonymization
 ↓
Historical Record

Ejemplo conceptual:

User:
  original_email → anonymized representation

Pero la anonimización no se aplicará automáticamente.

Debe definirse por política porque podría romper:

integridad referencial;
investigación;
trazabilidad;
correlación histórica.
21. Deletion

La eliminación física será una operación controlada.

Nunca deberá ocurrir simplemente porque:

status = DISABLED

o:

status = REMOVED

La eliminación deberá pasar por:

Retention Policy
       ↓
Eligibility Check
       ↓
Legal / Security Constraints
       ↓
Deletion or Anonymization
22. Cascading Deletes

No se utilizarán CASCADE DELETE indiscriminadamente en Identity.

Por ejemplo:

DELETE User
   ↓
DELETE Sessions
DELETE Audit
DELETE Security Events
DELETE Memberships

sería peligroso.

La eliminación deberá considerar las dependencias y obligaciones de retención.

En consecuencia:

Las relaciones de Identity no deberán diseñarse suponiendo que eliminar una entidad raíz implica eliminar automáticamente todo su historial.

23. Legal Hold / Security Hold

La estrategia deberá permitir que determinados registros queden temporalmente fuera del proceso normal de eliminación.

Conceptualmente:

Retention Eligible
      │
      ▼
   Legal Hold?
    /      \
  YES       NO
   │         │
 RETAIN    DELETE /
           ARCHIVE

El detalle jurídico de Legal Hold no está definido por el Blueprint y queda como decisión posterior de Compliance/Data Governance.

24. Retention Policy Engine

La política de retención no deberá estar codificada directamente en cada repository.

La arquitectura futura podrá utilizar:

Retention Policy
       ↓
Retention Evaluation
       ↓
Lifecycle Action

Por ejemplo:

Session
   ↓
Expired
   ↓
Retention Policy
   ↓
Archive/Delete

Esto permite cambiar períodos de retención sin modificar toda la lógica de persistencia.

25. Procesamiento Asíncrono

Las operaciones de retención que puedan ser costosas no deberán bloquear requests normales.

Arquitectura:

Retention Scheduler
        ↓
Retention Evaluation
        ↓
Batch
        ↓
Archive / Anonymize / Delete

Deberán existir mecanismos para:

retries;
observabilidad;
errores parciales;
reanudación;
métricas.
26. Tenant Isolation

La retención deberá respetar el tenant.

Una operación de limpieza de:

Organization A

no deberá afectar datos pertenecientes a:

Organization B

salvo que exista una operación administrativa explícita y autorizada a nivel plataforma.

27. Auditabilidad de la eliminación

Las operaciones de:

deletion;
anonymization;
archival;
retention policy changes;

deberán quedar auditadas cuando sean relevantes para seguridad o compliance.

La auditoría no deberá registrar:

password
token
MFA secret
recovery code
private key

sino información sobre la acción:

what
when
who
why
resource
result
correlation_id
28. Data Lifecycle Matrix

La siguiente matriz establece la dirección arquitectónica:

Entidad	Lifecycle	Retención	Acción final
User	Sí	Política	Archive/Delete/Anonymize
Organization	Sí	Política	Archive/Delete
Membership	Sí	Política	Retain/Archive
Role	Sí	Política	Retain/Archive
Permission	Sí	Política	Retain/Archive
Session	Sí	Seguridad	Archive/Delete
Refresh Token	Sí	Seguridad	Delete/Protected retention
MFA Factor	Sí	Seguridad	Revoke/Delete
MFA Challenge	Sí	Corta	Delete
Recovery Code	Sí	Corta/segura	Delete
Trusted Device	Sí	Seguridad	Delete/Archive
Security Policy	Sí + Version	Política	Retain/Archive
Audit Event	Inmutable	Política	Archive/Dispose
Security Event	Inmutable	Política	Archive/Dispose
Outbox Event	Sí	Operacional	Archive/Delete

Los períodos concretos todavía no se fijan.

29. Lo que NO decide este ADR

Es importante no mezclar responsabilidades.

Este ADR no determina todavía:

30/90/180 días;
años exactos de auditoría;
período exacto de sessions;
TTL exacto de MFA;
TTL exacto de refresh tokens;
proveedor de archival;
proveedor de almacenamiento;
método concreto de anonymization;
requisitos legales específicos por jurisdicción;
particionamiento de tablas;
cifrado específico de cada columna.

Esas decisiones pertenecen a otros ADRs y/o a políticas de Compliance.

30. Alternativas consideradas
A. Eliminar físicamente al desactivar

Descartada.

Confundiría lifecycle con retention y destruiría información potencialmente necesaria.

B. Conservar absolutamente todo indefinidamente

Descartada.

Aumentaría innecesariamente:

riesgo;
superficie de ataque;
costos;
complejidad.
C. Definir un único período de retención

Descartada.

Una sesión, un recovery code y un audit event no tienen la misma naturaleza.

D. Retención implementada independientemente en cada módulo

Descartada.

Generaría políticas inconsistentes.

31. Consecuencias
Positivas
Lifecycle explícito.
Menor riesgo de eliminación accidental.
Mayor trazabilidad.
Mejor control de PII.
Separación entre datos operacionales y de seguridad.
Preparación para compliance.
Posibilidad de archivar información histórica.
Mejor evolución futura.
Negativas
Mayor complejidad de infraestructura.
Requiere jobs/procesos de lifecycle.
Requiere políticas configurables.
Requiere pruebas de eliminación/anonymization.
Requiere coordinación con Compliance y Data Governance.
32. Reglas no negociables
1. Lifecycle ≠ Retention.

2. DISABLED ≠ DELETE.

3. REMOVED ≠ DELETE.

4. REVOKED ≠ DELETE inmediato.

5. Audit Events son append-only.

6. Security Events no son logs técnicos.

7. Secrets nunca se almacenan en plaintext.

8. Secrets nunca aparecen en logs/events/audit.

9. No cascading deletes indiscriminados.

10. Retention es por categoría de dato.

11. Deletion requiere evaluación de política.

12. Tenant isolation aplica también a retention.

13. Operaciones sensibles de deletion/anonymization deben ser auditables.

14. Los períodos concretos no se inventan hasta contar con la política correspondiente.
33. Trazabilidad
Blueprint
   │
   ├── Identity Domain
   ├── Security Architecture
   ├── Data Model
   ├── Event Architecture
   └── Compliance / Governance
          │
          ▼
       ESP-0001
          │
          ▼
ADR-IDENTITY-012
   PostgreSQL Schema
          │
          ▼
ADR-IDENTITY-013
   Retention & Lifecycle
          │
          ▼
IS-IDENTITY-015
Identity Persistence

La arquitectura de eventos también contempla explícitamente políticas de retención como parte de la definición de cada evento.

34. Dependencias
Depende de
ESP-0001
ADR-IDENTITY-003 — Organization & Tenant
ADR-IDENTITY-006 — Authorization
ADR-IDENTITY-008 — Session Management
ADR-IDENTITY-009 — Refresh Token Lifecycle
ADR-IDENTITY-010 — MFA
ADR-IDENTITY-012 — PostgreSQL Schema
Será utilizado por
IS-IDENTITY-002
IS-IDENTITY-003
IS-IDENTITY-004
IS-IDENTITY-008
IS-IDENTITY-009
IS-IDENTITY-010
IS-IDENTITY-012
IS-IDENTITY-013
IS-IDENTITY-015
IS-IDENTITY-017
35. ADRs relacionados
ADR-IDENTITY-012
PostgreSQL Schema

ADR-IDENTITY-014
Encryption & Secret Protection

ADR-IDENTITY-015
PostgreSQL Partitioning & Scalability

ADR-IDENTITY-017
Database Migration Strategy

ADR-IDENTITY-025
Refresh Token Cryptography & Concurrency

ADR-IDENTITY-028
Observability & Operational Security

ADR-IDENTITY-029
Disaster Recovery & Backup
36. Criterios para Accepted

El ADR podrá pasar de:

Proposed → Accepted

cuando estén definidos y revisados:

clasificación definitiva de datos;
políticas de retención;
requisitos legales/compliance aplicables;
estrategia de archival;
estrategia de anonymization;
eliminación segura;
tratamiento de Legal Hold;
ownership de las políticas;
automatización de lifecycle;
auditoría de operaciones de retención.
37. Estado final

ADR-IDENTITY-013 — Proposed

La decisión central queda establecida:

Identity utilizará lifecycle explícito y retención diferenciada por categoría de dato. La desactivación, suspensión, revocación o expiración de una entidad no implica automáticamente su eliminación física. Audit Events y Security Events tendrán un tratamiento especial orientado a preservar trazabilidad e integridad, mientras que secretos y datos temporales tendrán políticas de retención mucho más restrictivas.

Con esto queda correctamente preparado el terreno para el siguiente ADR:

ADR-IDENTITY-014 — Encryption & Secret Protection

Ese será especialmente importante porque aquí ya hemos definido qué datos pueden permanecer y durante cuánto tiempo, y el siguiente debe definir cómo se protegen criptográficamente mientras permanecen almacenados.
