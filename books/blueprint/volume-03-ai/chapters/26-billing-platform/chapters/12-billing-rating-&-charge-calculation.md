1. Billing Rating & Charge Calculation

Billing Rating & Charge Calculation es la capa responsable de transformar el uso medido y las condiciones comerciales en cargos financieros concretos.

Esta capa responde a la pregunta:

¿Cuánto debe cobrarse por lo que el cliente consumió o contrató?

La relación con los capítulos anteriores queda:

Product
   ↓
Plan
   ↓
Price
   ↓
Subscription
   ↓
Usage
   ↓
Metering
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

La distinción fundamental es:

Usage
= cuánto consumió


Rating
= qué tarifa corresponde


Charge Calculation
= cuánto dinero genera ese consumo
2. Propósito

El Rating Engine debe poder:

identificar el precio aplicable;
evaluar pricing rules;
determinar cantidades facturables;
aplicar tiers;
calcular overages;
aplicar mínimos;
aplicar máximos;
aplicar descuentos;
calcular proration;
aplicar compromisos;
determinar currency;
generar cargos;
generar ajustes;
mantener trazabilidad;
soportar múltiples modelos de pricing;
procesar grandes volúmenes;
recalcular de manera controlada.
3. Rating vs Charge

Aunque están estrechamente relacionados, deben mantenerse separados.

Rating

Determina:

Usage
+
Pricing Rules
=
Rated Usage
Charge Calculation

Determina:

Rated Usage
+
Commercial Adjustments
=
Charge

Por ejemplo:

Usage
1,500,000 events


Rate
$0.0005/event


Rated Usage
$750


Discount
-$75


Charge
$675
4. Rating Architecture
Usage
  │
  ▼
Meter
  │
  ▼
Subscription
  │
  ▼
Plan
  │
  ▼
Price
  │
  ▼
Pricing Rules
  │
  ▼
Rating Engine
  │
  ▼
Rated Usage
  │
  ▼
Charge Engine
  │
  ▼
Charge

El Rating Engine debe ser independiente del Invoice Engine.

5. Inputs del Rating Engine

El motor puede recibir:

Rating Request
│
├── Customer
├── Tenant
├── Billing Account
├── Subscription
├── Product
├── Plan
├── Price
├── Meter
├── Usage
├── Currency
├── Region
├── Contract
├── Discounts
└── Effective Date

Con estos elementos determina la tarifa correcta.

6. Rating Context

El cálculo debe ejecutarse dentro de un contexto:

Rating Context
│
├── customer_id
├── tenant_id
├── subscription_id
├── product_id
├── plan_id
├── price_id
├── meter_id
├── quantity
├── unit
├── timestamp
├── currency
├── region
└── contract

Esto permite que el mismo consumo pueda tener precios diferentes según el cliente.

7. Pricing Resolution

El primer paso:

Usage
   │
   ▼
Identify Subscription
   │
   ▼
Identify Product
   │
   ▼
Identify Plan
   │
   ▼
Identify Price

Después:

Price
   +
Pricing Rules
   ↓
Applicable Rate
8. Effective Price

El Rating Engine debe determinar qué precio estaba vigente cuando ocurrió el consumo.

Ejemplo:

Price v1
$0.005
Jan → Jun


Price v2
$0.004
Jul → Dec

Evento:

15 Aug

Resultado:

Applicable Price
= v2
= $0.004

Nunca debe utilizarse accidentalmente el precio actual si el evento corresponde a un período anterior.

9. Basic Rating

El cálculo más simple:

Quantity × Unit Price
=
Charge

Ejemplo:

10,000 requests
×
$0.01
=
$100
10. Flat Rating

Para un plan fijo:

Plan Price
$499/month

El resultado:

Recurring Charge
= $499

No depende directamente de usage.

11. Per-Seat Rating
Seats = 50
Price = $10

Resultado:

50 × $10
=
$500

Puede existir:

Minimum Seats = 10

Si el cliente tiene 5:

Billable Seats = 10
12. Per-Unit Rating
Quantity = 250
Unit Price = $2

Resultado:

Charge = $500
13. Usage Rating
Usage
250,000 events


Rate
$0.0005


Charge
$125

La cantidad debe provenir del Metering Layer.

14. Tiered Rating

Ejemplo:

0 – 10K
$0.010


10K – 100K
$0.008


100K – 1M
$0.005

El Rating Engine determina primero:

Applicable Tier

y luego:

Quantity × Tier Rate
15. Volume Rating

En volume pricing:

100,000 events

entra al tier:

100K tier
$0.008

Entonces:

100,000 × $0.008
=
$800

Todo el volumen utiliza la tarifa correspondiente al tier alcanzado.

16. Graduated Rating

Cada tramo utiliza su propia tarifa:

First 10K
× $0.010
= $100


Next 90K
× $0.008
= $720

Resultado:

Total = $820

El motor debe diferenciar claramente:

Tiered
Volume
Graduated
17. Hybrid Rating

Un cargo puede combinar:

Base Subscription
        +
Seats
        +
Usage
        +
Add-ons
        +
Overage

Ejemplo:

Base Plan        $500
50 Seats         $250
API Usage        $100
AI Usage         $200
Storage           $50
────────────────────
Subtotal       $1,100
18. Included Usage

El Rating Engine debe determinar primero qué parte está incluida.

Actual Usage
1,250,000


Included
1,000,000

Entonces:

Billable Usage
=
1,250,000 - 1,000,000


=
250,000
19. Overage Rating
Overage
250,000 events


Rate
$0.0005


Charge
$125

El sistema debe mantener separados:

Included Usage
Billable Usage
Overage Quantity
Overage Charge
20. Minimum Charge

Puede existir un cargo mínimo:

Calculated Charge
$300


Minimum Charge
$500

Resultado:

Final Charge
$500

Debe quedar registrado que el mínimo fue aplicado.

21. Maximum Charge

También puede existir un máximo:

Calculated Charge
$12,000


Maximum Charge
$10,000

Resultado:

Final Charge
$10,000

El sistema debe registrar el cap.

22. Commitment Rating

Para contratos Enterprise:

Annual Commitment
$120,000

Si el consumo produce:

$100,000

el motor puede determinar:

Commitment Shortfall
$20,000

según los términos contractuales.

23. Discount Rating

El descuento debe aplicarse según una jerarquía definida.

Ejemplo:

Gross Charge
$1,000


Discount
20%


Discount Amount
$200


Net Charge
$800

Nunca debería simplemente sobrescribirse el precio original.

24. Multiple Discounts

Puede existir:

Base Price
$1,000


Contract Discount
10%


Promotion
5%

El motor debe tener una regla explícita:

Sequential
o
Stacked

Por ejemplo, secuencial:

$1,000
→ 10% = $900
→ 5%  = $855
25. Discount Limits

Puede existir una política:

Maximum Discount
30%

Si se solicita:

45%

el Rating Engine debe:

Reject

o requerir una aprobación especial.

26. Proration Rating

Cuando cambia una subscription durante un período:

Old Plan
$500


New Plan
$900

El motor calcula:

Unused Old Plan
+
Remaining New Plan
=
Proration

Ejemplo conceptual:

Old Credit       -$250
New Charge        $450
────────────────────
Net Adjustment    $200
27. Proration Precision

La plataforma debe definir:

daily;
hourly;
minute-based;
exact time;
calendar-day.

Ejemplo:

Monthly Price = $900


30-day period
Daily rate = $30

La política debe ser consistente.

28. Tax Calculation Boundary

El Rating Engine normalmente determina:

Gross Charge
Discount
Net Charge

El Tax Engine puede posteriormente calcular:

Tax

Por ejemplo:

Net Charge
$800


Tax
$152


Invoice Total
$952

La separación evita mezclar pricing con tax logic.

29. Currency Conversion

Cuando una subscription utiliza una moneda diferente:

Price
USD


Billing Account
CLP

Debe existir una política explícita:

Exchange Rate Source
+
Exchange Rate Date
+
Conversion Rule

El Rating Engine no debe convertir arbitrariamente.

30. Rounding

El cálculo monetario requiere reglas de redondeo.

Ejemplo:

Calculated
$10.4567

Puede convertirse a:

$10.46

La política debe definir:

decimal precision;
rounding mode;
currency precision;
cuándo se redondea.
31. Rounding Strategy

Debe existir una política consistente:

ROUND_HALF_UP
ROUND_HALF_EVEN
FLOOR
CEILING

El sistema debe evitar errores acumulativos por redondear prematuramente.

32. Charge Components

Un Charge puede estar compuesto por múltiples componentes:

Charge
│
├── Base Fee
├── Seat Fee
├── Usage Fee
├── Overage
├── Add-on
├── Discount
├── Credit
└── Adjustment

Esto permite transparencia.

33. Charge Line

Cada componente puede generar una línea:

Charge Line
│
├── description
├── quantity
├── unit_price
├── amount
├── currency
├── product
├── plan
├── price
├── meter
└── usage_period

Ejemplo:

AI Tokens
10,000,000
× $0.0002
= $2,000
34. Charge Calculation

La fórmula conceptual:

Gross Amount
=
Base
+
Usage
+
Add-ons
+
Overage
+
Commitments

Luego:

Net Amount
=
Gross Amount
-
Discounts
-
Credits
+
Adjustments

Finalmente:

Final Charge
=
Net Amount

Los impuestos se calculan según la arquitectura fiscal.

35. Charge Types

Los cargos pueden clasificarse:

RECURRING
USAGE
OVERAGE
ONE_TIME
SETUP
ACTIVATION
PRORATION
COMMITMENT
PENALTY
ADJUSTMENT

Esto facilita reporting.

36. Charge Status
CALCULATED
PENDING
APPROVED
POSTED
INVOICED
VOIDED
ADJUSTED
REFUNDED

Un charge calculado no necesariamente está todavía contabilizado.

37. Charge Posting

Flujo:

Calculated
   │
   ▼
Validated
   │
   ▼
Posted
   │
   ▼
Invoice

Una vez posteado, debe existir protección contra modificaciones destructivas.

38. Charge Immutability

Una vez facturado:

Charge
$500

no debería editarse directamente a:

$450

Debe generarse:

Adjustment
-$50

Esto conserva el historial.

39. Charge Adjustments

Los ajustes pueden ser:

Positive Adjustment
Negative Adjustment
Credit
Debit
Correction

Ejemplo:

Original Charge
$1,000


Correction
-$100


Net
$900
40. Rating Traceability

Cada charge debe poder responder:

¿Por qué existe este cargo?

Por ejemplo:

Charge
  │
  ├── Usage
  │     └── 250,000 events
  │
  ├── Meter
  │     └── API_CALL
  │
  ├── Price
  │     └── $0.0005
  │
  ├── Discount
  │     └── 10%
  │
  └── Rating Version
        └── v3

Esta trazabilidad es fundamental para auditoría.

41. Rating Versioning

El Rating Engine debe ser versionable.

Rating Engine v1
      │
      ▼
Rating Engine v2
      │
      ▼
Rating Engine v3

Un cálculo histórico debe conservar:

rating_version
pricing_version
meter_version
42. Rating Recalculation

Puede ser necesario recalcular:

Usage
   │
   ▼
Re-rating
   │
   ▼
Corrected Charge

Esto puede ocurrir cuando:

llega usage tardío;
se corrige un meter;
existe un error de pricing;
se detecta una anomalía;
se procesa una corrección autorizada.
43. Re-rating Controls

El re-rating nunca debe duplicar cargos.

Debe existir:

Original Rating
      │
      ▼
Re-rating Job
      │
      ▼
Compare
      │
      ▼
Adjustment

No:

Original Charge
+
New Charge

sin reversar o ajustar el anterior.

44. Rating Batch Processing

El Rating Engine puede procesar:

1M usage records
      │
      ▼
Batch
      │
      ▼
Rating
      │
      ▼
Aggregated Charges

Debe soportar:

retries;
checkpoints;
idempotency;
partial failures;
resumability.
45. Real-Time Rating

Para determinados casos:

Usage Event
      │
      ▼
Real-Time Rating
      │
      ▼
Current Charge

Esto puede utilizarse para:

prepaid;
quota enforcement;
real-time consumption;
usage alerts.
46. Batch vs Real-Time Rating
Real-Time
→ Immediate estimation / control


Batch
→ Final financial rating

Pueden coexistir:

Real-Time Rating
      +
Final Batch Rating

El segundo debe ser la referencia financiera final cuando así lo establezca el modelo.

47. Rating APIs
Calculate
POST /billing/rating/calculate
Preview
POST /billing/rating/preview
Batch
POST /billing/rating/batches
Re-rate
POST /billing/rating/rerate
Charge
GET  /billing/charges
GET  /billing/charges/{id}
48. Rating Request

Ejemplo conceptual:

{
  "subscription_id": "SUB-100",
  "meter_id": "API_CALL",
  "quantity": 250000,
  "usage_period": {
    "start": "2026-08-01",
    "end": "2026-08-31"
  }
}

El motor devuelve:

{
  "quantity": 250000,
  "included_quantity": 100000,
  "billable_quantity": 150000,
  "unit_price": 0.0005,
  "gross_amount": 75,
  "discount": 0,
  "net_amount": 75
}
49. Rating Result

Debe incluir:

RatingResult
│
├── id
├── subscription_id
├── meter_id
├── pricing_version
├── rating_version
├── quantity
├── included_quantity
├── billable_quantity
├── unit_price
├── gross_amount
├── discount_amount
├── credit_amount
├── adjustment_amount
├── net_amount
├── currency
├── calculated_at
└── metadata
50. Charge Data Model
Charge
│
├── id
├── charge_number
├── customer_id
├── tenant_id
├── billing_account_id
├── subscription_id
├── product_id
├── plan_id
├── price_id
├── meter_id
├── charge_type
├── quantity
├── unit_price
├── gross_amount
├── discount_amount
├── credit_amount
├── net_amount
├── currency
├── period_start
├── period_end
├── status
├── rating_version
└── created_at
51. Charge Relationships
Subscription
     │
     ├── Charge A
     ├── Charge B
     ├── Charge C
     └── Charge D
             │
             ▼
           Invoice

Una invoice puede contener cargos de múltiples subscriptions.

52. Charge Consolidation

Ejemplo Enterprise:

Tenant A
   └── $5,000


Tenant B
   └── $3,000


Tenant C
   └── $2,000


──────────────
Invoice
$10,000

Cada charge mantiene su origen.

53. Rating and Tax

La arquitectura recomendada:

Usage
   ↓
Metering
   ↓
Rating
   ↓
Charge
   ↓
Tax Engine
   ↓
Invoice

Así:

Rating determina precio;
Tax determina impuesto;
Invoice consolida.
54. Rating and Revenue

La cadena:

Usage
   ↓
Rating
   ↓
Charge
   ↓
Invoice
   ↓
Revenue Recognition

Esto permite distinguir:

Usage
Revenue
Cash

que son conceptos financieros diferentes.

55. Rating and FinOps

Para EVOXA:

AI Usage
   │
   ├── Customer Rating
   │       ↓
   │    Revenue
   │
   └── Provider Cost
           ↓
        FinOps

Esto permite calcular:

Customer Charge
-
Infrastructure Cost
=
Gross Margin
56. Rating Anomaly Detection

Puede detectarse:

Expected Charge
$500


Calculated
$5,000


Variance
+900%

Resultado:

Rating Anomaly

La transacción puede:

continuar;
quedar pendiente;
requerir aprobación;
ser rechazada.
57. Rating Explainability

Una característica Enterprise importante es poder mostrar:

¿Por qué se cobraron $1,250?


Base Plan             $500
Seats                 $200
API Usage              $75
AI Usage              $400
Storage                 $75
──────────────────────────
Gross                $1,250
Discount                 $0
──────────────────────────
Total                $1,250

Y para cada línea:

Source
Meter
Quantity
Rate
Pricing Rule
Version

Esto reduce disputas.

58. Charge Disputes

Cuando un cliente cuestiona un cargo:

Customer Dispute
      │
      ▼
Charge
      │
      ▼
Usage
      │
      ▼
Meter
      │
      ▼
Pricing
      │
      ▼
Rating Calculation

El sistema debe poder reconstruir todo el cálculo.

59. Rating Security

Las reglas de pricing deben estar protegidas.

Pricing Manager
    → Configure


Finance
    → Approve


Billing Engine
    → Execute


Auditor
    → Read

El motor de producción no debería permitir cambios manuales arbitrarios durante el cálculo.

60. Rating Observability

Debe medirse:

rating latency;
events processed;
failed calculations;
rejected usage;
rerating jobs;
charge variance;
calculation errors.

Ejemplo:

1,000,000 Events
       │
       ├── 999,500 Rated
       ├── 400 Rejected
       └── 100 Failed

Esto permite detectar problemas antes de cerrar billing.

61. Rating Reconciliation

Antes de facturar:

Usage
  ↓
Expected Charge
  ↓
Rated Charge
  ↓
Reconciliation

Ejemplo:

Expected
$100,000


Rated
$99,950


Variance
$50

La diferencia puede generar una excepción.

62. Rating Close

El período puede tener:

OPEN
  │
  ▼
RATING
  │
  ▼
RATING_COMPLETE
  │
  ▼
BILLING_READY
  │
  ▼
CLOSED

No debería generarse la invoice final hasta completar las validaciones requeridas.

63. Enterprise Rating Architecture
                         USAGE
                           │
                           ▼
                         METER
                           │
                           ▼
                     SUBSCRIPTION
                           │
                           ▼
                      PRICE BOOK
                           │
                           ▼
                    PRICING RULES
                           │
                           ▼
                     RATING ENGINE
                           │
          ┌────────────────┼────────────────┐
          ▼                ▼                ▼
        Flat             Usage            Hybrid
          │                │                │
          └────────────────┼────────────────┘
                           ▼
                    Discount Engine
                           │
                           ▼
                   Commitment Engine
                           │
                           ▼
                    Charge Engine
                           │
                           ▼
                        CHARGE
                           │
                           ▼
                     TAX ENGINE
                           │
                           ▼
                       INVOICE
64. EVOXA Rating Model

Para EVOXA podemos imaginar:

                     EVOXA SUBSCRIPTION
                            │
              ┌─────────────┼─────────────┐
              ▼             ▼             ▼
           Platform       Mobility        AI
              │             │             │
           $500/mo        Usage          Tokens
              │             │             │
              └─────────────┼─────────────┘
                            ▼
                       RATING ENGINE
                            │
          ┌─────────────────┼─────────────────┐
          ▼                 ▼                 ▼
       Base Fee          Usage Fee        Overage
          │                 │                 │
          └─────────────────┼─────────────────┘
                            ▼
                          CHARGE

Ejemplo:

Platform                  $500
Mobility Usage             $250
AI Input Tokens             $80
AI Output Tokens           $120
Storage                     $50
──────────────────────────────
Gross                    $1,000


Discount                   -$100
──────────────────────────────
Net Charge                 $900
65. Rating Data Flow
                 BILLING PLATFORM
                        │
          ┌─────────────┼─────────────┐
          ▼             ▼             ▼
       Catalog      Subscription      Usage
          │             │             │
          └─────────────┼─────────────┘
                        ▼
                  Rating Context
                        │
                        ▼
                  Pricing Resolver
                        │
                        ▼
                  Rating Engine
                        │
                        ▼
                  Charge Engine
                        │
                        ▼
                     Charges
                        │
                        ▼
                    Invoicing
66. Core Principles
1. Deterministic

El mismo contexto debe producir el mismo resultado.

2. Explainable

Cada cargo debe poder explicarse.

3. Versioned

Pricing, Meter y Rating deben conservar versiones.

4. Idempotent

El mismo consumo no debe producir cargos duplicados.

5. Immutable Financial Results

Los cargos facturados no deben modificarse destructivamente.

6. Re-Ratable

Debe poder recalcularse de manera controlada.

7. Multi-Model

Debe soportar:

flat;
seat;
usage;
tiered;
volume;
graduated;
hybrid;
commitment.
8. Enterprise Ready

Debe soportar:

custom contracts;
price books;
commitments;
overrides;
consolidated billing.
9. Auditable

Debe existir trazabilidad:

Charge
→ Rating
→ Pricing
→ Meter
→ Usage
10. Scalable

Debe procesar desde pequeñas operaciones hasta grandes volúmenes de eventos.

67. Resultado de 26/12

Billing Rating & Charge Calculation constituye el motor económico de Billing.

Su función es transformar:

Usage
+
Subscription
+
Plan
+
Price
+
Pricing Rules
+
Discounts
+
Commitments

en:

Rated Usage
       ↓
Charge

La arquitectura completa hasta ahora queda:

26/08 Catalog
      ↓
26/09 Pricing & Plans
      ↓
26/10 Subscriptions
      ↓
26/11 Usage & Metering
      ↓
26/12 Rating & Charge Calculation
      ↓
26/13 Invoicing
      ↓
26/14 Payments
      ↓
26/15 Credits & Adjustments
      ↓
...

Y el principio central es:

Billing Rating & Charge Calculation debe transformar de manera determinística, versionada, auditable e idempotente el consumo y las condiciones comerciales en cargos financieros, manteniendo una separación estricta entre medición, pricing, cálculo, impuestos, facturación y reconocimiento de ingresos.

Cadena conceptual de Billing hasta este punto
┌─────────────────────────────────────────────────────────────┐
│                    BILLING COMMERCIAL LAYER                 │
│                                                             │
│ Product → Plan → Price → Subscription                      │
│                                                             │
├─────────────────────────────────────────────────────────────┤
│                    BILLING USAGE LAYER                      │
│                                                             │
│ Usage Event → Meter → Aggregation → Billable Usage          │
│                                                             │
├─────────────────────────────────────────────────────────────┤
│                    BILLING RATING LAYER                     │
│                                                             │
│ Pricing Rules → Rating → Discounts → Commitments             │
│                         ↓                                   │
│                       Charge                                │
│                                                             │
├─────────────────────────────────────────────────────────────┤
│                    BILLING FINANCIAL LAYER                  │
│                                                             │
│ Charge → Invoice → Payment → Revenue                        │
│                                                             │
└─────────────────────────────────────────────────────────────┘

26/12 deja establecido el motor de cálculo financiero sobre el cual se apoyará el siguiente capítulo:

26/13 — Billing Invoicing
