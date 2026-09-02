Cómo queda relacionado con ESP-0001

No vamos a meter el ADR dentro de ESP-0001.

La relación debe ser:

Blueprint
    │
    ▼
ESP-0001 Identity
    │
    ├── 01 Domain Model
    ├── 02 Architecture
    ├── 03 Authentication & Authorization
    ├── 04 User & Organization
    ├── 05 Roles & Permissions
    ├── 06 Sessions & Tokens
    ├── 07 MFA
    ├── 08 Security Policies
    ├── 09 Audit & Compliance
    ├── 10 API Contracts
    ├── 11 Persistence
    ├── 12 Events
    ├── 13 Testing
    ├── 14 Implementation
    └── 15 Deployment
             │
             ▼
       ADR-IDENTITY-001
             │
             ▼
       Decisiones formales
             │
             ▼
    Implementation Stories
             │
             ▼
       Technical Tasks
             │
             ▼
          Código

Esto es importante: el ADR no reemplaza al ESP. El ESP describe los requisitos y diseño de ingeniería; el ADR registra las decisiones arquitectónicas que tomamos para poder implementarlo.

📝 ¿Qué vamos a poner en ADR-IDENTITY-001?

Este ADR será particularmente importante porque consolidará las decisiones tecnológicas que necesitamos antes de programar.

Por ejemplo:

ADR-IDENTITY-001

Identity Architecture & Technology Decisions

Decidiríamos formalmente:

1. Backend framework
2. Programming language
3. API architecture
4. Database
5. ORM / data access
6. Authentication
7. JWT strategy
8. Refresh token strategy
9. Password hashing
10. Redis usage
11. MFA architecture
12. RBAC + ABAC
13. Security Policy Engine
14. Audit architecture
15. Event architecture
16. Transactional Outbox
17. API documentation
18. Testing framework
19. Containerization
20. Configuration
21. Secrets management
22. Observability
23. Deployment strategy

Pero hay algo importante:

No quiero inventar estas decisiones basándome solamente en conocimientos generales.

Tenemos que cruzarlas con los 15 capítulos de ESP-0001 y con lo que realmente establece el Blueprint. Cuando el Blueprint deje una decisión abierta, la marcaremos como:

Decision Required

y luego elegiremos una alternativa y justificaremos por qué.

🏷️ Estado inicial

Yo lo dejaría:

Status: Proposed

y no Accepted.

Después de revisar todas las decisiones:

Proposed
   ↓
Under Review
   ↓
Accepted

Una vez Accepted, esa decisión pasa a ser la referencia que utilizaremos durante la programación.

Y después de este ADR

No saltaría directamente al código.

Haría:

ESP-0001 / 01–15
          ↓
ADR-IDENTITY-001
          ↓
ADR-IDENTITY-002
ADR-IDENTITY-003
...
          ↓
Identity Implementation Stories
          ↓
Technical Tasks
          ↓
PROGRAMACIÓN

Así mantenemos la misma filosofía de trazabilidad que definimos en el Blueprint: no improvisar decisiones importantes durante la programación.

Si quieres, el siguiente paso es que hagamos ahora mismo el documento completo ADR-IDENTITY-001 — Identity Architecture & Technology Decisions, utilizando lo que ya definimos en los 15 capítulos y separando claramente lo que exige el Blueprint, lo que ya decidimos y lo que todavía tenemos que decidir.
