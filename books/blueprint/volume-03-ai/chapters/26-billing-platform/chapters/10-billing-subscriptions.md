1. Billing Subscriptions

Billing Subscriptions define la relación contractual y recurrente mediante la cual un Customer adquiere uno o más productos o servicios bajo un determinado Plan y Pricing durante un período de tiempo.

La Subscription es el puente entre la configuración comercial y la ejecución financiera:

Customer
   │
   ▼
Product
   │
   ▼
Plan
   │
   ▼
Price
   │
   ▼
Subscription
   │
   ├── Usage
   ├── Charges
   ├── Renewals
   ├── Upgrades
   ├── Downgrades
   └── Cancellations
           │
           ▼
        Invoice

La Subscription representa qué contrató el cliente, bajo qué condiciones y durante qué período.

2. Propósito

Billing Subscriptions debe administrar:

creación de suscripciones;
activación;
períodos;
renovaciones;
upgrades;
downgrades;
pausas;
reactivaciones;
cancelaciones;
trials;
proration;
cambios de precio;
add-ons;
quantities;
usage;
commitments;
entitlements;
contratos;
lifecycle;
billing schedules.

Debe permitir modelos simples y Enterprise.

3. Subscription vs Customer

El Customer representa:

Quién compra.

La Subscription representa:

Qué compró y bajo qué condiciones.

Customer
   │
   ├── Subscription A
   ├── Subscription B
   └── Subscription C

Un Customer puede tener múltiples subscriptions simultáneas.

4. Subscription vs Billing Account

La Billing Account representa:

Dónde se registra financieramente la relación.

La Subscription representa:

Qué servicio está contratado.

Customer
   │
   ▼
Billing Account
   │
   ├── Subscription A
   ├── Subscription B
   └── Subscription C

Los cargos generados por las subscriptions se registran posteriormente contra la cuenta correspondiente.

5. Subscription vs Plan

El Plan es una oferta del catálogo:

Professional Plan

La Subscription es una instancia concreta:

Customer ACME
→ Professional Plan
→ Started 01/08/2026
→ $499/month

Por lo tanto:

Plan
   ↓
Template / Commercial Definition


Subscription
   ↓
Customer-specific Instance
6. Subscription Identity

Cada subscription debe tener una identidad propia:

Subscription
│
├── id
├── subscription_number
├── customer_id
├── tenant_id
├── billing_account_id
├── product_id
├── plan_id
├── price_id
└── status

El subscription_number puede utilizarse en documentos y atención al cliente.

7. Subscription Lifecycle

El lifecycle puede ser:

DRAFT
   │
   ▼
PENDING
   │
   ▼
TRIALING
   │
   ▼
ACTIVE
   │
   ├── PAUSED
   │     │
   │     ▼
   │   ACTIVE
   │
   ├── PAST_DUE
   │
   ├── CANCELED
   │
   └── EXPIRED

No todos los estados tienen que estar disponibles para todos los productos.

8. Subscription Status

Los estados principales:

Draft

Configuración aún no activada.

Pending

Esperando condiciones previas.

Trialing

Período de prueba.

Active

Suscripción activa.

Paused

Temporalmente suspendida.

Past Due

Existe obligación vencida.

Canceled

Cancelada.

Expired

Finalizada por término contractual.

9. Subscription Lifecycle vs Billing Status

Deben existir como conceptos independientes.

Ejemplo:

Subscription Status = ACTIVE
Billing Status     = PAST_DUE

El cliente puede seguir teniendo acceso mientras se ejecutan políticas de cobranza.

Otro caso:

Subscription Status = PAUSED
Billing Status     = CURRENT
10. Subscription Creation

Flujo:

Customer
   │
   ▼
Select Product
   │
   ▼
Select Plan
   │
   ▼
Select Price
   │
   ▼
Configure Quantity
   │
   ▼
Apply Discounts
   │
   ▼
Calculate Taxes
   │
   ▼
Create Subscription

Posteriormente:

Subscription
   ↓
Billing Schedule
   ↓
Charges
11. Subscription Configuration

Una subscription puede contener:

Subscription
│
├── Product
├── Plan
├── Price
├── Quantity
├── Billing Cycle
├── Billing Anchor
├── Start Date
├── End Date
├── Trial
├── Discount
├── Tax Configuration
├── Payment Configuration
└── Contract
12. Subscription Start Date

La fecha de inicio determina cuándo comienza la relación contractual.

Start Date
15/08/2026

Puede ser:

inmediata;
futura;
asociada a una fecha contractual;
dependiente de provisioning.
13. Subscription End Date

Puede existir:

Open-ended
No End Date
Fixed Term
Start
01/01/2026


End
31/12/2026
Contract-based

El término depende del contrato.

14. Subscription Billing Cycle

La suscripción puede utilizar:

Monthly
Quarterly
Semi-Annual
Annual
Custom

Ejemplo:

Annual Subscription
$5,000 / year
15. Billing Anchor

La subscription puede tener un anchor:

Billing Anchor = 15

Entonces:

15 Aug → 14 Sep
15 Sep → 14 Oct
15 Oct → 14 Nov

Esto debe ser consistente con el Billing Account.

16. Trial Subscriptions

Una subscription puede comenzar con trial:

Subscription
   │
   ▼
Trial
30 Days
   │
   ▼
Conversion
   │
   ▼
Paid Subscription

El trial debe definir:

duración;
precio;
límites;
usage included;
payment method;
conversion behavior.
17. Trial Lifecycle
TRIALING
   │
   ├── CONVERTED
   │
   ├── CANCELED
   │
   └── EXPIRED

Si el cliente no convierte:

Trial Expired
      ↓
Subscription Canceled

o puede pasar a un plan gratuito, según configuración.

18. Subscription Quantity

Las subscriptions pueden tener quantity.

Ejemplo:

Plan = Professional
Quantity = 25 Seats

El precio:

25 × $10
=
$250/month

La quantity puede ser:

seats;
units;
licenses;
devices;
locations;
packages.
19. Subscription Add-ons

Una subscription puede incorporar add-ons:

Subscription
│
├── Base Plan
│
├── AI Add-on
├── Storage Add-on
└── Premium Support

Cada add-on puede tener su propia:

cantidad;
precio;
ciclo;
vigencia.
20. Subscription Components

Para planes complejos:

Subscription
│
├── Component A
│     └── Fixed Price
│
├── Component B
│     └── Per Seat
│
├── Component C
│     └── Usage
│
└── Component D
      └── Add-on

Esto permite subscriptions híbridas.

21. Subscription Pricing Snapshot

Al crear la subscription debe conservarse:

Subscription
│
├── Product Version
├── Plan Version
├── Price Version
├── Pricing Rules
├── Discounts
└── Contract Terms

Esto es crítico.

Si mañana el precio del plan cambia de:

$499 → $599

la subscription histórica no debería cambiar automáticamente.

22. Subscription Overrides

Una subscription puede tener condiciones especiales:

Standard Price
$499


Subscription Override
$449

Debe existir trazabilidad:

Base Price
   ↓
Override
   ↓
Final Subscription Price

Los overrides deben estar gobernados.

23. Discounts

Una subscription puede recibir descuentos:

Base Price       $1,000
Discount           -20%
────────────────────────
Net Price          $800

Los descuentos pueden ser:

porcentuales;
monto fijo;
temporales;
contractuales;
promocionales.
24. Subscription Proration

Cuando cambia una subscription a mitad del período:

Current Plan
$500
   │
   ▼
Upgrade
   │
   ▼
New Plan
$900

Billing debe calcular:

Unused Old Plan
       +
Remaining New Plan
       =
Proration Adjustment

La fórmula exacta debe ser determinística y configurable.

25. Upgrade

Un cliente puede pasar:

Starter
   ↓
Professional
   ↓
Business
   ↓
Enterprise

El upgrade puede ser:

Immediate

Cambio inmediato.

Next Billing Cycle

Cambio al próximo ciclo.

Contractual

Cambio sujeto a aprobación.

26. Downgrade

También:

Enterprise
   ↓
Business

Puede aplicarse:

inmediatamente;
al final del período;
después de cumplir contrato;
con restricciones de entitlement.
27. Plan Change

Cada cambio debe generar un evento:

SubscriptionPlanChanged

Debe conservar:

Previous Plan
New Plan
Effective Date
Reason
Actor
Pricing Impact
28. Subscription Pause

Una subscription puede pausarse:

ACTIVE
  │
  ▼
PAUSED
  │
  ▼
RESUMED

La pausa puede:

detener billing;
mantener billing;
conservar entitlements;
suspender entitlements.

La política debe estar explícitamente definida.

29. Subscription Resume

Al reactivar:

Paused Subscription
       │
       ▼
Resume
       │
       ▼
Billing Calculation
       │
       ▼
Active

Puede existir proration o ajuste.

30. Subscription Cancellation

El cliente puede cancelar:

ACTIVE
  │
  ▼
CANCEL_REQUESTED
  │
  ▼
CANCELLED

La cancelación puede ser:

Immediate

Termina inmediatamente.

End of Period

Permanece activa hasta el final del ciclo.

Contractual

Respeta el término contractual.

31. Cancellation Reasons

Debe registrarse:

Cancellation Reason
│
├── Customer Request
├── Non-Payment
├── Product Retirement
├── Contract Expiration
├── Fraud
├── Business Decision
└── Other

Esto es fundamental para churn analytics.

32. Subscription Renewal

Una subscription recurrente puede renovarse:

Current Period
      │
      ▼
Renewal Date
      │
      ▼
Next Period

Puede existir:

automatic renewal;
manual renewal;
conditional renewal;
contract renewal.
33. Auto-Renewal

Configuración:

Auto Renew = TRUE

Workflow:

Renewal Window
      │
      ▼
Validate Subscription
      │
      ▼
Validate Payment
      │
      ▼
Renew
      │
      ▼
Generate Billing
34. Manual Renewal

Para contratos donde el cliente debe renovar:

Subscription
      │
      ▼
Expiring
      │
      ▼
Renewal Required
      │
      ▼
Customer Approval
      │
      ▼
Renewed

Esto resulta especialmente útil para Enterprise.

35. Subscription Commitments

Una subscription puede tener commitment:

Commitment
12 Months
Minimum Spend
$50,000

Puede incluir:

committed quantity;
committed spend;
committed term;
minimum usage.
36. Minimum Spend

Ejemplo:

Annual Commitment
$120,000


Actual Usage
$100,000


Commitment Adjustment
$20,000

La lógica debe estar asociada al contrato y no simplemente al precio.

37. Subscription Usage

La subscription puede consumir meters:

Subscription
   │
   ▼
Meter
   │
   ├── API Calls
   ├── Storage
   ├── AI Tokens
   └── Events

El usage alimenta el Rating Engine.

38. Subscription Rating

El flujo:

Usage Event
     │
     ▼
Subscription
     │
     ▼
Meter
     │
     ▼
Pricing Rules
     │
     ▼
Rated Usage
     │
     ▼
Charge

La subscription proporciona el contexto comercial para valorar el consumo.

39. Subscription Entitlements

La subscription puede generar derechos:

Subscription
   │
   ▼
Entitlements
   │
   ├── Users
   ├── Storage
   ├── API
   ├── AI
   └── Reports

El sistema de entitlement debe poder consultar:

Does Customer have access?

sin depender directamente de la factura.

40. Subscription Limits

Puede establecer:

Included
1,000,000 Events


Warning
800,000


Overage
1,000,000+

Las políticas pueden ser:

Allow
Warn
Charge
Block
41. Subscription Billing Schedule

La subscription puede generar un calendario:

Subscription
   │
   ▼
Billing Schedule
   │
   ├── 01/08
   ├── 01/09
   ├── 01/10
   └── 01/11

Cada período genera:

Recurring Charge
+
Usage Charge
+
Add-ons
-
Credits
=
Invoice
42. Subscription Invoice Generation

El flujo:

Billing Period Closed
        │
        ▼
Collect Subscription Charges
        │
        ▼
Collect Usage
        │
        ▼
Apply Discounts
        │
        ▼
Apply Credits
        │
        ▼
Calculate Taxes
        │
        ▼
Generate Invoice
43. Subscription Payment Failure

Si el pago falla:

Payment Failed
      │
      ▼
Retry
      │
      ├── Success → Active
      │
      └── Failure
            │
            ▼
          Dunning
            │
            ▼
        Past Due

La subscription puede tener una política propia.

44. Subscription Dunning

El dunning puede considerar:

Attempt 1
   ↓
Attempt 2
   ↓
Attempt 3
   ↓
Warning
   ↓
Restriction
   ↓
Suspension
   ↓
Cancellation

Las reglas pueden variar por Customer Segment.

45. Subscription Taxes

La subscription puede determinar contexto fiscal:

Subscription
   │
   ├── Product Tax Category
   ├── Customer Tax Profile
   ├── Billing Address
   └── Legal Entity
          │
          ▼
       Tax Engine

El resultado fiscal se registra en los cargos/factura correspondientes.

46. Subscription Multi-Tenant

Una subscription puede asociarse a:

Customer
   │
   ├── Tenant A
   │     └── Subscription
   │
   ├── Tenant B
   │     └── Subscription
   │
   └── Tenant C
         └── Subscription

También puede existir una subscription corporativa:

Corporate Subscription
        │
        ├── Tenant A
        ├── Tenant B
        └── Tenant C

Esto es importante para Enterprise.

47. Subscription Consolidation

Varias subscriptions pueden consolidarse:

Subscription A ─┐
Subscription B ─┼──→ Billing Account
Subscription C ─┘
                       │
                       ▼
                    Invoice

La factura puede mantener el detalle por subscription.

48. Subscription State Machine

Una representación completa:

                    ┌───────────┐
                    │   DRAFT   │
                    └─────┬─────┘
                          ▼
                    ┌───────────┐
                    │  PENDING  │
                    └─────┬─────┘
                          ▼
                  ┌──────────────┐
                  │   TRIALING   │
                  └──────┬───────┘
                         ▼
                  ┌──────────────┐
             ┌────│    ACTIVE    │────┐
             │    └──────┬───────┘    │
             │           │            │
             ▼           ▼            ▼
          PAUSED      PAST_DUE     CANCELLED
             │           │
             ▼           ▼
          RESUMED     RECOVERY
             │           │
             └──────┬────┘
                    ▼
                  ACTIVE
49. Subscription Events

El sistema debe publicar eventos como:

SubscriptionCreated
SubscriptionActivated
SubscriptionTrialStarted
SubscriptionTrialConverted
SubscriptionRenewed
SubscriptionPaused
SubscriptionResumed
SubscriptionUpgraded
SubscriptionDowngraded
SubscriptionQuantityChanged
SubscriptionPriceChanged
SubscriptionDiscountApplied
SubscriptionPastDue
SubscriptionCanceled
SubscriptionExpired

Estos eventos alimentan otros dominios.

50. Subscription Audit

Cada cambio debe registrar:

Actor
Timestamp
Subscription
Previous State
New State
Reason
Effective Date
Correlation ID

Ejemplo:

SubscriptionUpgrade


Previous:
Professional


New:
Enterprise


Effective:
01/09/2026


Actor:
Customer Admin
51. Subscription Data Model

Modelo conceptual:

Subscription
│
├── id
├── subscription_number
├── customer_id
├── tenant_id
├── billing_account_id
├── product_id
├── product_version_id
├── plan_id
├── plan_version_id
├── price_id
├── price_version_id
├── status
├── billing_interval
├── billing_anchor
├── quantity
├── start_date
├── end_date
├── trial_start
├── trial_end
├── renewal_date
├── auto_renew
├── cancellation_date
├── cancellation_reason
├── contract_id
├── created_at
└── updated_at
52. Subscription Items

Para subscriptions complejas:

Subscription
│
├── Subscription Item A
│     └── Base Plan
│
├── Subscription Item B
│     └── AI Add-on
│
├── Subscription Item C
│     └── Storage
│
└── Subscription Item D
      └── Usage

Modelo:

SubscriptionItem
│
├── id
├── subscription_id
├── product_id
├── plan_id
├── price_id
├── quantity
├── meter_id
├── status
├── effective_from
└── effective_to
53. Subscription Schedule

Los cambios futuros pueden programarse.

Ejemplo:

Current
Professional
$500


01/01/2027
      ↓
Enterprise
$900

Esto requiere:

Subscription Schedule

con:

effective date;
target plan;
target price;
quantity;
duration.
54. Subscription Schedule Architecture
Subscription
   │
   ▼
Schedule
   │
   ├── Phase 1
   │    └── Professional
   │
   ├── Phase 2
   │    └── Business
   │
   └── Phase 3
        └── Enterprise

Esto es útil para contratos Enterprise y pricing escalonado.

55. Subscription APIs
Core
POST   /billing/subscriptions
GET    /billing/subscriptions
GET    /billing/subscriptions/{id}
PATCH  /billing/subscriptions/{id}
Lifecycle
POST /billing/subscriptions/{id}/activate
POST /billing/subscriptions/{id}/pause
POST /billing/subscriptions/{id}/resume
POST /billing/subscriptions/{id}/cancel
POST /billing/subscriptions/{id}/renew
Changes
POST /billing/subscriptions/{id}/upgrade
POST /billing/subscriptions/{id}/downgrade
POST /billing/subscriptions/{id}/change-plan
POST /billing/subscriptions/{id}/change-quantity
56. Subscription Preview APIs

Antes de ejecutar cambios:

POST /billing/subscriptions/{id}/preview-upgrade
POST /billing/subscriptions/{id}/preview-downgrade
POST /billing/subscriptions/{id}/preview-cancellation

La respuesta puede mostrar:

Current Amount
Proration
Credits
New Amount
Tax
Effective Date

Esto evita sorpresas financieras.

57. Subscription Financial Traceability

La cadena completa:

Customer
   ↓
Subscription
   ↓
Plan
   ↓
Price
   ↓
Usage
   ↓
Rating
   ↓
Charge
   ↓
Invoice
   ↓
Payment
   ↓
Revenue

Debe poder reconstruirse en ambos sentidos.

Por ejemplo:

¿Qué subscriptions generaron esta factura?

Y:

¿Qué invoices fueron generadas por esta subscription?

58. Subscription Analytics

KPIs principales:

Growth
new subscriptions;
activated subscriptions;
upgrade rate.
Retention
renewal rate;
churn rate;
retention.
Revenue
MRR;
ARR;
subscription revenue;
expansion revenue.
Usage
average usage;
overage;
utilization.
Billing
failed payments;
past-due subscriptions;
collection rate.
59. Subscription MRR

El MRR puede calcularse desde las subscriptions activas:

MRR
=
Σ Active Subscription Recurring Revenue

Debe considerar:

discounts;
quantities;
recurring add-ons;
currency normalization.

No debería incluir automáticamente:

one-time charges;
refunds;
taxes.
60. Subscription Churn

El sistema debe distinguir:

Customer Churn

Cliente completo abandona.

Subscription Churn

Una subscription específica termina.

Product Churn

Cliente deja un producto.

Revenue Churn

Pérdida de revenue.

Estas métricas no son equivalentes.

61. Subscription Expansion

Puede producirse por:

Upgrade
+
Additional Seats
+
Add-ons
+
Usage
=
Expansion Revenue

Esto alimenta Net Revenue Retention.

62. Subscription AI

La IA puede analizar subscriptions:

Subscription History
      +
Usage
      +
Payments
      +
Customer Behavior
      ↓
AI

Predicciones:

churn probability;
renewal probability;
upgrade probability;
expansion opportunity;
payment risk.
63. Subscription AI Recommendations

Ejemplo:

Customer Usage
       │
       ▼
Current Plan: Professional
       │
       ▼
Usage = 92% of limits
       │
       ▼
AI Recommendation
       │
       ▼
Business Plan

La recomendación no debe cambiar automáticamente la subscription sin autorización.

64. Subscription Architecture

La arquitectura completa:

                         CUSTOMER
                            │
                            ▼
                       BILLING ACCOUNT
                            │
                            ▼
                       SUBSCRIPTION
                            │
          ┌─────────────────┼─────────────────┐
          ▼                 ▼                 ▼
       Product             Plan              Price
          │                 │                 │
          └─────────────────┼─────────────────┘
                            ▼
                      Subscription Items
                            │
             ┌──────────────┼──────────────┐
             ▼              ▼              ▼
          Entitlements     Usage          Add-ons
             │              │              │
             └──────────────┼──────────────┘
                            ▼
                         Rating
                            │
                            ▼
                          Charges
                            │
                            ▼
                         Invoices
                            │
                            ▼
                         Payments
                            │
                            ▼
                          Revenue
65. Enterprise Subscription Model

Para EVOXA:

                       ACME CORPORATION
                              │
                              ▼
                       Billing Account
                              │
                              ▼
                      Enterprise Contract
                              │
                              ▼
                       Subscription
                              │
        ┌─────────────────────┼─────────────────────┐
        ▼                     ▼                     ▼
     Platform              Mobility                AI
        │                     │                     │
     Base Fee              Usage Fee             Token Fee
        │                     │                     │
        └─────────────────────┼─────────────────────┘
                              ▼
                       Consolidated Invoice

Esto permite combinar:

recurring fees;
usage;
AI consumption;
add-ons;
contractual commitments.
66. Subscription Governance

Las operaciones críticas deben estar controladas.

Ejemplo:

Plan Change
    │
    ├── Standard
    │      → Automatic
    │
    └── Enterprise
           → Approval

Igualmente:

Discount > 30%
       ↓
Approval Required
67. Core Principles
1. Subscription as Contractual Instance

Representa una contratación concreta.

2. Plan Separation

Plan y Subscription no son la misma entidad.

3. Pricing Snapshot

El precio aplicado debe quedar congelado históricamente.

4. Lifecycle Management

Toda subscription debe tener estados y transiciones claras.

5. Proration

Los cambios deben producir ajustes determinísticos.

6. Usage Integration

Debe soportar consumo recurrente y variable.

7. Enterprise Support

Debe soportar contratos, commitments y estructuras consolidadas.

8. Auditability

Todo cambio debe ser trazable.

9. Financial Integrity

Una subscription no debe modificar retroactivamente cargos ya facturados.

10. Automation

Renewals, billing, dunning y lifecycle deben poder automatizarse.

68. Resultado de 26/10

Billing Subscriptions establece la entidad que materializa la relación comercial entre el Customer y el catálogo.

La cadena queda:

Customer
   ↓
Billing Account
   ↓
Subscription
   ↓
Product
   ↓
Plan
   ↓
Price
   ↓
Entitlements
   ↓
Usage
   ↓
Rating
   ↓
Charge
   ↓
Invoice
   ↓
Payment
   ↓
Revenue

Y el principio fundamental es:

Una Subscription representa una instancia contractual y monetizable de un producto o servicio adquirido por un Customer, manteniendo de forma explícita su plan, pricing, cantidades, vigencia, entitlements, consumo, cambios, renovaciones y condiciones comerciales a lo largo de todo su ciclo de vida.

Con esto queda preparada la siguiente pieza de la arquitectura:

26/11 — Billing Usage & Metering
