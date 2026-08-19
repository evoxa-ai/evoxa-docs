1. Billing Pricing & Plans

Billing Pricing & Plans define cómo los productos del catálogo se convierten en ofertas comerciales concretas, estableciendo cuánto paga el cliente, por qué unidad se cobra, con qué frecuencia, bajo qué reglas y qué capacidades obtiene.

Esta capa conecta directamente:

Product
   │
   ▼
Plan
   │
   ▼
Pricing
   │
   ▼
Subscription
   │
   ▼
Billing

Mientras 26/08 — Billing Products & Catalog define qué puede venderse, Pricing & Plans define cómo se comercializa y monetiza.

2. Objetivo

Pricing & Plans debe permitir administrar:

planes comerciales;
precios;
monedas;
ciclos de facturación;
modelos de pricing;
tiers;
volumen;
seats;
usage;
mínimos;
máximos;
overages;
descuentos;
promociones;
precios Enterprise;
precios regionales;
precios personalizados;
versiones;
vigencia;
reglas de pricing.

El objetivo es que el sistema pueda representar desde:

$49 / month

hasta:

Enterprise Contract
+
Annual Commitment
+
Usage-Based Pricing
+
Volume Discounts
+
Custom Terms
3. Plan vs Price

Una separación fundamental:

Plan define qué recibe el cliente; Price define cuánto paga por ello.

Ejemplo:

Professional Plan
│
├── 25 Users
├── 1M Events
├── Analytics
├── Reports
└── AI Insights

Y:

Price
│
├── $499 USD
├── Monthly
└── Flat Rate

Por lo tanto:

Plan ≠ Price

Un mismo plan puede tener múltiples precios.

4. Pricing Architecture

La estructura:

Product
   │
   ▼
Plan
   │
   ├── Features
   ├── Entitlements
   ├── Limits
   │
   ▼
Pricing Configuration
   │
   ├── Base Price
   ├── Usage Price
   ├── Tier Rules
   ├── Discount Rules
   └── Billing Cycle
   │
   ▼
Price
   │
   ▼
Subscription

Esto permite separar la definición comercial de la ejecución financiera.

5. Plan Architecture

Un plan puede contener:

Plan
│
├── Identity
├── Product
├── Features
├── Entitlements
├── Limits
├── Pricing
├── Billing Cycle
├── Eligibility
├── Availability
└── Version

Ejemplo:

EVOXA Professional
│
├── 25 Users
├── 1M Mobility Events
├── Analytics
├── OOH
├── Reports
└── AI Insights
6. Plan Types

La plataforma puede soportar:

Free
$0
Trial
30 Days
Starter

Para clientes pequeños.

Professional

Para clientes intermedios.

Business

Para organizaciones con mayores necesidades.

Enterprise

Para clientes con contratos personalizados.

Custom

Para acuerdos comerciales específicos.

7. Plan Hierarchy

Los planes pueden organizarse:

Product
│
├── Starter
│
├── Professional
│
├── Business
│
└── Enterprise

Esto facilita:

comparación;
upgrade;
downgrade;
packaging;
pricing strategy.
8. Plan Versioning

Los planes deben ser versionados.

Professional v1
      │
      ▼
Professional v2
      │
      ▼
Professional v3

Los clientes existentes pueden permanecer en una versión anterior.

Existing Customer
       ↓
Professional v1


New Customer
       ↓
Professional v3

Esto evita modificaciones retroactivas.

9. Plan Lifecycle
DRAFT
  │
  ▼
REVIEW
  │
  ▼
APPROVED
  │
  ▼
PUBLISHED
  │
  ▼
ACTIVE
  │
  ▼
DEPRECATED
  │
  ▼
RETIRED

Un plan retirado debe seguir siendo válido para clientes históricos cuando corresponda.

10. Price Object

El Price representa una configuración monetaria.

Price
│
├── Amount
├── Currency
├── Pricing Model
├── Billing Interval
├── Unit
├── Meter
├── Minimum
├── Maximum
├── Effective Date
└── Version

Ejemplo:

Amount = 499
Currency = USD
Interval = MONTH
Model = FLAT
11. Pricing Models

Debe soportar múltiples modelos.

Flat
Per Seat
Per Unit
Usage
Tiered
Volume
Graduated
Package
Hybrid
Commitment
Custom Contract
12. Flat Pricing

Precio fijo:

Professional Plan
$499 / month

Fórmula:

Charge = Fixed Price

Es el modelo más simple.

13. Per-Seat Pricing

El precio depende de usuarios.

Price = Users × Unit Price

Ejemplo:

50 Users
×
$10
=
$500/month

Puede existir:

Minimum Seats = 5
Maximum Seats = 500
14. Per-Unit Pricing

Se cobra por unidades específicas.

100 Reports
×
$2
=
$200

Unidades posibles:

reports;
projects;
locations;
devices;
campaigns;
transactions.
15. Usage-Based Pricing

El precio depende del consumo:

Usage
  ↓
Meter
  ↓
Quantity
  ↓
Rate
  ↓
Charge

Ejemplo:

1,000,000 events
×
$0.0005
=
$500
16. Tiered Pricing

El precio cambia según rangos.

0 – 10K       → $0.010
10K – 100K    → $0.008
100K – 1M     → $0.005
1M+           → $0.003

Debe especificarse claramente cómo se calcula cada tramo.

17. Volume Pricing

El precio unitario depende del volumen total alcanzado.

10K units
→ $0.010/unit


100K units
→ $0.008/unit

Si el cliente consume 100K:

100,000 × $0.008
=
$800

Esto es diferente de tiered pricing.

18. Graduated Pricing

En graduated pricing, cada tramo utiliza su propia tarifa.

Ejemplo:

First 10K
× $0.010
= $100


Next 90K
× $0.008
= $720


Total
= $820

El Pricing Engine debe distinguir correctamente:

Tiered
vs
Volume
vs
Graduated
19. Hybrid Pricing

Puede combinar múltiples componentes:

Base Subscription
       +
Seats
       +
Usage
       +
Add-ons
       +
Overage
       =
Total Charge

Ejemplo:

Base Plan          $500
20 Users           $200
Usage              $150
AI Consumption     $100
Storage Overage     $50
────────────────────────
Total            $1,000

Este modelo será especialmente importante para EVOXA.

20. Minimum Commitment

Un plan puede exigir un mínimo.

Minimum Commitment
= $5,000 / month

Si el consumo real es:

$3,800

puede cobrarse:

$5,000

dependiendo de las condiciones contractuales.

21. Maximum Commitment / Spending Caps

También pueden existir límites:

Monthly Spending Cap
= $10,000

Cuando el consumo se acerca al límite:

80% → Warning
90% → Alert
100% → Policy Action

La acción puede ser:

bloquear;
solicitar aprobación;
permitir overage;
cambiar pricing;
generar alerta.
22. Billing Interval

El precio puede tener diferentes frecuencias:

Monthly
Quarterly
Semi-Annual
Annual
Custom

Ejemplo:

Monthly = $500


Annual = $5,000

La tarifa anual puede ser comercialmente distinta de:

$500 × 12 = $6,000
23. Billing Anchor

El plan puede utilizar una fecha de referencia:

Subscription Start
15/08


Billing Anchor
15th of month

Los ciclos:

15 Aug → 14 Sep
15 Sep → 14 Oct
15 Oct → 14 Nov
24. Pricing Currency

Un precio debe definir explícitamente:

Currency

Ejemplo:

USD
EUR
CLP
MXN
BRL
COP

El sistema no debe asumir que todas las monedas utilizan la misma tarifa convertida automáticamente.

25. Regional Pricing

Puede existir pricing por región:

Professional


Chile → CLP 450,000
Mexico → MXN 9,000
USA → USD 499
Europe → EUR 470

Cada precio debe tener:

currency;
country/region;
effective date;
pricing version.
26. Customer-Specific Pricing

Enterprise puede requerir pricing personalizado:

Standard Price
$10,000


Customer Contract Price
$7,500

Debe existir una referencia explícita:

Customer
   ↓
Contract
   ↓
Custom Price

Nunca debería simplemente modificarse el precio global.

27. Contract Pricing

El precio puede formar parte de un contrato:

Contract
│
├── Base Fee
├── Usage Rate
├── Minimum Commitment
├── Discount
├── Payment Terms
└── Term

El contrato puede sobrescribir determinadas reglas comerciales.

28. Pricing Precedence

Debe existir una jerarquía clara:

Global Pricing
      ↓
Organization Pricing
      ↓
Tenant Pricing
      ↓
Customer Pricing
      ↓
Contract Pricing
      ↓
Subscription Override

No todas las capas deben poder sobrescribir cualquier propiedad.

Debe existir una política de gobernanza.

29. Pricing Rules

Las reglas pueden incluir:

Pricing Rule
│
├── Eligibility
├── Quantity
├── Unit
├── Rate
├── Tier
├── Discount
├── Minimum
├── Maximum
├── Currency
└── Effective Date

Estas reglas alimentan el Rating Engine.

30. Effective Dating

Cada precio debe tener vigencia:

Price v1
01/01/2026 → 30/06/2026


Price v2
01/07/2026 → 31/12/2026


Price v3
01/01/2027 → ...

Esto permite reconstruir:

¿Qué precio debía aplicarse el 15 de agosto de 2026?

La respuesta debe ser determinística.

31. Price Changes

Los cambios de precio no deben modificar registros históricos.

Incorrecto:

Price = $500
       ↓
UPDATE
       ↓
Price = $600

Correcto:

Price v1 = $500
Price v2 = $600

Las nuevas subscriptions pueden utilizar v2 mientras las antiguas continúan con v1 según contrato.

32. Price Books

Enterprise puede utilizar Price Books.

Price Book
│
├── Standard
├── Enterprise
├── Partner
├── Government
└── Regional

Ejemplo:

Standard Price Book
→ $500


Enterprise Price Book
→ $420


Partner Price Book
→ $350

Esto permite diferentes estrategias comerciales sin duplicar productos.

33. Pricing Segmentation

El precio puede depender de:

customer segment;
industry;
region;
company size;
contract type;
channel;
partner status;
volume.

Ejemplo:

Customer Segment
        │
        ▼
Pricing Segment
        │
        ▼
Price Book
        │
        ▼
Applicable Price
34. Discounts

Los descuentos pueden aplicarse a planes y precios.

Tipos:

Percentage
Fixed Amount
Volume
Promotional
Contractual
Loyalty
Introductory
Partner

Ejemplo:

Base Price = $1,000
Discount = 20%


Net Price = $800

El descuento debe quedar registrado como una entidad financiera trazable.

35. Promotional Pricing

Las promociones pueden definir:

Promotion
│
├── Code
├── Start Date
├── End Date
├── Eligibility
├── Discount
├── Usage Limit
└── Customer Limit

Ejemplo:

PROMO2026
20% off
First 3 months
36. Trial Pricing

Un plan puede comenzar gratuitamente:

Trial
30 Days
$0
   │
   ▼
Paid Subscription
$499/month

El pricing debe definir:

trial duration;
included usage;
payment requirements;
conversion rules.
37. Introductory Pricing

Puede existir:

Months 1–3
$199


Month 4+
$499

El Pricing Engine debe mantener ambos precios como reglas explícitas.

38. Upgrade / Downgrade Pricing

Cuando el cliente cambia de plan:

Starter
$100
   │
   ▼
Professional
$300

Puede requerirse:

immediate change;
next-cycle change;
proration;
credit;
additional charge.
39. Proration

Ejemplo:

Starter
$100/month


Used 15 days

El cliente cambia a:

Professional
$300/month

Billing debe calcular:

Unused Starter Credit
+
Remaining Professional Charge
=
Proration Adjustment

El resultado debe poder auditarse.

40. Plan Entitlements

Cada plan puede determinar capacidades:

Enterprise Plan
│
├── Users = Unlimited
├── Storage = 10 TB
├── API = 100M calls
├── AI = 50M tokens
├── Reports = Unlimited
└── Support = Premium

El pricing define el costo.

El entitlement define el derecho.

Pricing
   ≠
Entitlement

Aunque ambos estén relacionados.

41. Plan Limits

Los límites pueden ser:

hard limit;
soft limit;
warning threshold;
billable overage.

Ejemplo:

Included = 1M events


80% → Warning
100% → Limit
>100% → Overage
42. Pricing + Usage

La relación:

Plan
 │
 ├── Included Usage
 │
 └── Overage Price
        │
        ▼
      Meter
        │
        ▼
      Usage
        │
        ▼
      Rating

Esto permite construir modelos híbridos.

43. Pricing + AI

Para productos de IA:

AI Plan
│
├── Included Tokens
├── Included Requests
├── Model Access
└── Overage Pricing

Ejemplo:

Included
10M tokens


Overage
$2 / 1M tokens

El catálogo puede definir diferentes precios por modelo:

Model A → $X
Model B → $Y
Model C → $Z
44. Dynamic Pricing

La plataforma puede soportar pricing dinámico cuando el modelo comercial lo requiera.

Demand
  +
Usage
  +
Capacity
  +
Customer Segment
       │
       ▼
Pricing Engine
       │
       ▼
Applicable Price

Sin embargo, para billing financiero debe existir una regla:

El precio aplicado a una transacción debe quedar congelado y ser auditable.

45. Pricing Approval Workflow

Cambios importantes:

Pricing Draft
     │
     ▼
Business Review
     │
     ▼
Finance Review
     │
     ▼
Compliance Review
     │
     ▼
Approval
     │
     ▼
Publish

Los cambios no deben llegar directamente a producción.

46. Pricing Audit

Debe registrarse:

Price Created
Price Updated
Price Approved
Price Published
Price Deprecated
Price Retired

Cada evento debe incluir:

actor;
timestamp;
previous value;
new value;
reason;
approval;
effective date.
47. Pricing APIs

APIs conceptuales:

POST   /billing/pricing
GET    /billing/pricing
GET    /billing/pricing/{id}
PATCH  /billing/pricing/{id}

Price Books:

POST /billing/price-books
GET  /billing/price-books
GET  /billing/price-books/{id}

Pricing Rules:

POST /billing/pricing-rules
GET  /billing/pricing-rules

Price calculation:

POST /billing/pricing/calculate

Este último puede utilizarse para obtener una cotización antes de crear una subscription.

48. Quote / Pricing Preview

Antes de contratar:

Customer
   │
   ▼
Product
   │
   ▼
Plan
   │
   ▼
Pricing Engine
   │
   ▼
Quote

Ejemplo:

Base Plan          $500
Seats              $200
Usage Estimate     $150
Discount            -$50
Tax                 $120
────────────────────────
Estimated Total    $920

La cotización no debe considerarse automáticamente una transacción financiera.

49. Pricing Calculation Engine

La arquitectura:

Pricing Request
      │
      ▼
Eligibility
      │
      ▼
Select Price Book
      │
      ▼
Select Price
      │
      ▼
Evaluate Rules
      │
      ▼
Calculate Quantity
      │
      ▼
Apply Tiers
      │
      ▼
Apply Discounts
      │
      ▼
Calculate Net Amount
      │
      ▼
Tax Calculation
      │
      ▼
Final Amount

Debe ser determinístico.

50. Pricing Engine Inputs

El motor puede recibir:

Customer
Tenant
Organization
Product
Plan
Subscription
Usage
Quantity
Currency
Region
Contract
Date

Y producir:

Applicable Price
Pricing Version
Quantity
Discount
Net Amount
Tax Context
Final Amount
51. Pricing Engine Output

El resultado debe conservar información suficiente para reconstrucción:

PricingResult
│
├── price_id
├── pricing_version
├── pricing_model
├── quantity
├── unit_price
├── discount
├── subtotal
├── tax
├── total
├── currency
├── effective_date
└── calculation_metadata

Esto permite auditar posteriormente cómo se obtuvo el valor.

52. Pricing Data Model

Modelo conceptual:

Pricing
│
├── id
├── product_id
├── plan_id
├── price_book_id
├── pricing_model
├── currency
├── amount
├── unit
├── interval
├── meter_id
├── minimum_quantity
├── maximum_quantity
├── effective_from
├── effective_to
├── version
├── status
└── metadata
53. Plan Data Model
Plan
│
├── id
├── product_id
├── plan_code
├── name
├── description
├── version
├── status
├── billing_interval
├── trial_config
├── entitlement_profile_id
├── eligibility_rule_id
├── effective_from
├── effective_to
└── metadata
54. Pricing Relationships
Product
   │
   ├── Plan A
   │     ├── Price A1
   │     ├── Price A2
   │     └── Price A3
   │
   ├── Plan B
   │     ├── Price B1
   │     └── Price B2
   │
   └── Plan C
         └── Enterprise Pricing

Esto permite flexibilidad sin duplicar productos.

55. Subscription Pricing Snapshot

Al crear una subscription:

Subscription
│
├── Product Version
├── Plan Version
├── Price Version
├── Pricing Rules
├── Discounts
└── Contract Terms

Esto protege la integridad histórica.

56. Pricing Analytics

El sistema debe medir:

Revenue
revenue by plan;
revenue by price;
MRR;
ARR.
Conversion
trial → paid;
plan conversion;
upgrade rate.
Pricing
average selling price;
discount rate;
price realization.
Customer
ARPU;
LTV;
expansion;
churn.
57. Price Realization

Una métrica importante:

List Price
     ↓
Discount
     ↓
Contract Price
     ↓
Actual Price

Puede medirse:

Price Realization =
Actual Revenue / List Revenue

Esto permite detectar descuentos excesivos.

58. Pricing Intelligence

La IA puede analizar:

Historical Pricing
       +
Usage
       +
Customer Behavior
       +
Revenue
       +
Churn
       ↓
AI Pricing Intelligence

Puede identificar:

precios demasiado bajos;
precios demasiado altos;
oportunidades de packaging;
elasticidad;
descuentos excesivos;
productos con baja monetización.
59. Pricing Anomaly Detection

Puede detectar:

Standard Price = $500


Applied Price = $50


Variance = -90%

El sistema puede generar:

Pricing Anomaly
│
├── Customer
├── Product
├── Expected Price
├── Applied Price
├── Variance
└── Risk
60. Enterprise Pricing

Para clientes Enterprise:

Enterprise Customer
       │
       ▼
Contract
       │
       ├── Custom Plan
       ├── Custom Price
       ├── Minimum Commitment
       ├── Usage Rates
       ├── Discount
       └── Payment Terms

Este pricing puede convivir con los planes estándar.

61. Pricing Governance Model
Catalog
   │
   ▼
Product Manager
   │
   ▼
Pricing Proposal
   │
   ▼
Commercial Review
   │
   ▼
Finance Review
   │
   ▼
Approval
   │
   ▼
Published Price

Debe existir separación de funciones para cambios financieros sensibles.

62. Pricing Security

Las capacidades deben estar protegidas:

Product Manager
   → Create Plan


Pricing Manager
   → Create Price


Finance
   → Approve Price


Administrator
   → Publish

No todos los roles deben tener todas las capacidades.

63. Enterprise Pricing Architecture

La arquitectura completa:

                       PRODUCT
                          │
                          ▼
                         PLAN
                          │
        ┌─────────────────┼─────────────────┐
        ▼                 ▼                 ▼
    Features         Entitlements         Limits
                          │
                          ▼
                    PRICE BOOK
                          │
              ┌───────────┼───────────┐
              ▼           ▼           ▼
           Standard    Enterprise   Partner
              │           │           │
              └───────────┼───────────┘
                          ▼
                   PRICING ENGINE
                          │
             ┌────────────┼────────────┐
             ▼            ▼            ▼
          Flat         Usage        Hybrid
             │            │            │
             └────────────┼────────────┘
                          ▼
                     Subscription
                          │
                          ▼
                        Rating
                          │
                          ▼
                        Charge
                          │
                          ▼
                       Invoice
64. EVOXA Pricing Model

Para EVOXA, un modelo potencial puede ser:

                         EVOXA
                           │
                      Product Catalog
                           │
        ┌──────────────────┼──────────────────┐
        ▼                  ▼                  ▼
     Platform             Data                AI
        │                  │                  │
        ▼                  ▼                  ▼
     Plans              Plans              Plans
        │                  │                  │
        └──────────────────┼──────────────────┘
                           ▼
                      Pricing Engine
                           │
        ┌──────────────────┼──────────────────┐
        ▼                  ▼                  ▼
    Subscription         Usage            Hybrid
        │                  │                  │
        └──────────────────┼──────────────────┘
                           ▼
                       Billing

Ejemplo conceptual:

EVOXA Professional
────────────────────────
Base                 $500
Users                $10/user
Mobility Usage       $X/event
AI Usage             $Y/token
Storage              $Z/GB

Esto permite que EVOXA pueda monetizar tanto la plataforma base como el consumo real.

65. Core Principles
1. Plan ≠ Price

El plan define el paquete; el precio define la monetización.

2. Version Everything

Planes, precios y reglas deben versionarse.

3. Effective Dating

Los precios deben tener vigencia explícita.

4. Deterministic Calculation

El mismo contexto debe producir el mismo resultado.

5. Historical Integrity

Los precios históricos nunca deben alterarse.

6. Pricing Flexibility

Debe soportar múltiples modelos.

7. Enterprise Customization

Debe soportar contratos y precios personalizados.

8. Governance

Los cambios de precios deben estar controlados.

9. Auditability

Debe poder explicarse cómo se calculó cada cargo.

10. AI Readiness

Pricing debe alimentar Revenue Intelligence y Pricing Intelligence.

66. Resultado de 26/09

Billing Pricing & Plans establece la capa que transforma productos del catálogo en ofertas comerciales monetizables.

La estructura fundamental queda:

Product
   ↓
Plan
   ↓
Features / Entitlements / Limits
   ↓
Price Book
   ↓
Price
   ↓
Pricing Rules
   ↓
Pricing Engine
   ↓
Subscription
   ↓
Usage / Rating
   ↓
Charge
   ↓
Invoice
   ↓
Revenue

Y el principio central es:

Pricing & Plans constituye el motor comercial de Billing, separando qué recibe el cliente de cuánto paga, permitiendo múltiples modelos de monetización, precios personalizados, contratos Enterprise, pricing regional, descuentos, consumo y reglas versionadas, sin comprometer la integridad financiera histórica.

Con esto queda preparada la siguiente capa:

26/10 — Billing Subscriptions
