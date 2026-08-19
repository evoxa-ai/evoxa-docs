1. Billing Revenue Management

Billing Revenue Management es la capa responsable de administrar el ciclo económico del Revenue generado por EVOXA, desde la identificación del ingreso asociado a una operación comercial hasta su reconocimiento, diferimiento, asignación, análisis y cierre.

Es importante separar:

Billing
= cuánto se cobra al Customer

de:

Revenue Management
= cuándo y cómo ese importe se convierte en Revenue reconocido

Por ejemplo:

Subscription anual
$12,000


Payment recibido
$12,000


Revenue reconocido
$1,000 / mes

Por lo tanto:

Payment ≠ Revenue

y también:

Invoice ≠ Revenue

2. Propósito

Revenue Management debe administrar:

Revenue Recognition;
Deferred Revenue;
Contract Revenue;
Performance Obligations;
Revenue Allocation;
Revenue Schedules;
Revenue Adjustments;
Revenue Forecasting;
Revenue Backlog;
Revenue Waterfall;
Revenue Reconciliation;
Revenue Analytics;
Revenue Close;
Revenue Policies;
Revenue Contracts;
Revenue Events;
Revenue Accounting Integration.
3. Revenue Lifecycle

La cadena:

Contract
   ↓
Subscription
   ↓
Performance Obligation
   ↓
Charge
   ↓
Invoice
   ↓
Payment
   ↓
Revenue Allocation
   ↓
Revenue Schedule
   ↓
Revenue Recognition
   ↓
Accounting

Pero Payment puede ocurrir antes:

Invoice
   ↓
Payment
   ↓
Deferred Revenue
   ↓
Revenue Recognition
4. Billing vs Revenue

Ejemplo:

Annual Subscription
$12,000

Invoice:

$12,000

Payment:

$12,000

Revenue:

Jan   $1,000
Feb   $1,000
Mar   $1,000
...
Dec   $1,000

La plataforma debe mantener las tres realidades separadas.

5. Revenue Recognition

Revenue Recognition determina:

¿En qué período debe reconocerse el ingreso?

Ejemplo:

Contract
12 months


Value
$12,000


Monthly Revenue
$1,000

El Revenue Schedule:

2026-01 → $1,000
2026-02 → $1,000
2026-03 → $1,000
...
2026-12 → $1,000
6. Revenue Recognition Principle

La arquitectura debe basarse en reglas contables aplicables a la empresa y jurisdicción.

La plataforma debe permitir configurar políticas compatibles con marcos como:

IFRS;
US GAAP;
políticas contables internas.

EVOXA no debe codificar una única metodología universal.

7. Revenue Recognition Model
Contract
   │
   ▼
Identify Obligations
   │
   ▼
Determine Transaction Price
   │
   ▼
Allocate Price
   │
   ▼
Determine Recognition Pattern
   │
   ▼
Recognize Revenue
8. Performance Obligations

Un contrato puede contener múltiples obligaciones:

Enterprise Contract
$120,000
│
├── Platform
├── AI Services
├── Support
├── Implementation
└── Training

Cada componente puede tener un patrón de reconocimiento diferente.

9. Performance Obligation Model
PerformanceObligation
│
├── id
├── contract_id
├── product_id
├── description
├── standalone_price
├── allocated_price
├── recognition_method
├── start_date
├── end_date
└── status
10. Standalone Selling Price

Supongamos:

Platform
$100,000


AI
$30,000


Support
$20,000


Total SSP
$150,000

Pero el contrato se vende por:

$120,000

El Transaction Price debe asignarse según las reglas correspondientes.

11. Revenue Allocation

Ejemplo:

Contract Value
$120,000

Asignación proporcional:

Platform
$80,000


AI
$24,000


Support
$16,000

Cada Performance Obligation obtiene su propio Revenue Schedule.

12. Revenue Allocation Engine
Transaction Price
       │
       ▼
Performance Obligations
       │
       ▼
Standalone Selling Prices
       │
       ▼
Allocation Method
       │
       ▼
Allocated Revenue
13. Recognition Methods

EVOXA debería soportar diferentes métodos:

STRAIGHT_LINE
MILESTONE
USAGE_BASED
DELIVERABLE
POINT_IN_TIME
OVER_TIME
MANUAL
CUSTOM
14. Straight-Line

Ejemplo:

Contract
$12,000


Duration
12 months

Revenue:

$1,000/month

Es apropiado para ciertos servicios prestados uniformemente durante el tiempo.

15. Point-in-Time

Algunos productos o servicios pueden reconocerse en un momento concreto:

Delivery
   ↓
Acceptance
   ↓
Revenue

Ejemplo:

Implementation Complete
$10,000

Revenue:

$10,000

en el momento definido por la política correspondiente.

16. Milestone Recognition

Ejemplo:

Contract
$100,000


Milestones
│
├── Design Complete → 20%
├── Implementation → 50%
└── Go-Live → 30%

Revenue:

$20,000
$50,000
$30,000
17. Usage-Based Revenue

Puede existir:

Usage
   ↓
Rating
   ↓
Charge
   ↓
Revenue

Ejemplo:

API Calls
1,000,000


Revenue
$5,000

El reconocimiento puede seguir la prestación del servicio.

18. Deferred Revenue

Cuando el Customer paga antes de recibir el servicio:

Payment
$12,000

pero el servicio dura:

12 months

entonces:

Deferred Revenue
$12,000

y posteriormente:

$1,000/month

se transfiere a Revenue.

19. Deferred Revenue Lifecycle
Payment
   ↓
Deferred Revenue
   ↓
Recognition Schedule
   ↓
Revenue

Ejemplo:

Opening Deferred
$12,000


Recognized
-$1,000


Ending Deferred
$11,000
20. Contract Liability

En determinados modelos contables:

Deferred Revenue
=
Contract Liability

conceptualmente.

El sistema debe poder representar la terminología contable que utilice la empresa.

21. Accrued Revenue

También puede existir Revenue ganado antes de facturarse:

Service Delivered
   ↓
Revenue Earned
   ↓
Invoice Later

Esto puede generar:

Accrued Revenue

o conceptos equivalentes según la política contable.

22. Contract Asset

Conceptualmente:

Revenue Earned
>
Amount Billable

Puede generarse un:

Contract Asset

Esto es diferente de:

Accounts Receivable

La plataforma debe permitir representar ambos conceptos.

23. Revenue Recognition Schedule
RevenueSchedule
│
├── id
├── contract_id
├── performance_obligation_id
├── start_date
├── end_date
├── total_amount
├── recognized_amount
├── remaining_amount
├── recognition_method
└── status
24. Revenue Schedule Example
Contract
$24,000
24 months

Schedule:

Month 1
$1,000


Month 2
$1,000


...


Month 24
$1,000

Total:

$24,000
25. Revenue Events

Cada reconocimiento debe generar un evento:

RevenueScheduled
RevenueRecognized
RevenueDeferred
RevenueAdjusted
RevenueReversed
RevenueReallocated
26. Revenue Ledger

Debe existir un Revenue Ledger:

Revenue Ledger
│
├── Recognized Revenue
├── Deferred Revenue
├── Accrued Revenue
├── Revenue Reversal
├── Revenue Adjustment
└── Revenue Reallocation

Esto permite reconstruir el historial.

27. Revenue Ledger Example
Contract
$12,000


Deferred
+$12,000


Jan Recognition
-$1,000 Deferred
+$1,000 Revenue


Feb Recognition
-$1,000 Deferred
+$1,000 Revenue

Ending:

Deferred
$10,000


Recognized Revenue
$2,000
28. Revenue Waterfall

Una herramienta fundamental:

Opening Deferred Revenue
        +
New Billings
        -
Recognized Revenue
        +
Adjustments
        -
Reversals
        =
Ending Deferred Revenue

Ejemplo:

Opening
$500,000


New Billings
+$200,000


Recognized
-$150,000


Adjustments
+$10,000
────────────────
Ending
$560,000
29. Revenue Waterfall by Customer
Customer
ACME


Opening Deferred
$100,000


New Contracts
+$50,000


Recognized
-$30,000


Ending
$120,000
30. Revenue Waterfall by Product
Platform
Opening       $500K
New           $200K
Recognized    -$150K
Ending        $550K

Esto permite analizar Revenue por producto.

31. Revenue Waterfall by Tenant
Tenant A
$500K


Tenant B
$300K


Tenant C
$200K

Muy importante para la arquitectura Multi-Tenant.

32. Revenue Recognition Calendar

El sistema debe saber:

Current Period
2026-08

qué Revenue corresponde reconocer:

Platform
$100K


AI
$40K


Support
$20K

Total:

$160K
33. Revenue Period Close

Al cerrar un período:

Revenue Close
   ↓
Validate Schedules
   ↓
Calculate Recognition
   ↓
Post Revenue
   ↓
Reconcile
   ↓
Lock Period
34. Revenue Close Controls

Antes del cierre:

All Contracts Valid
All Schedules Valid
No Missing Allocations
No Unresolved Adjustments
No Duplicate Entries
Tax Reconciled
Billing Reconciled
35. Revenue Period Lock

Una vez cerrado:

August 2026
LOCKED

Los cambios posteriores deben realizarse mediante:

Adjustment
Reversal
Catch-up

y no modificando silenciosamente el histórico.

36. Catch-Up Revenue

Si cambia una estimación:

Original Recognition
$1,000/month

y después:

Remaining Revenue
$11,000
Remaining Months
5

la nueva política podría requerir:

$2,200/month

El sistema debe generar el ajuste necesario.

37. Revenue Reallocation

Puede ocurrir que una modificación contractual cambie la asignación:

Platform
Before: $80K
After:  $70K


AI
Before: $20K
After:  $30K

El sistema debe recalcular los schedules afectados y generar ajustes de forma auditable.

38. Contract Modification

Una subscription puede cambiar:

Original
$10,000/month

a:

New
$12,000/month

El Revenue Engine debe determinar:

qué parte pertenece al contrato original;
qué parte corresponde a la modificación;
qué schedules deben ajustarse;
qué revenue ya fue reconocido.
39. Contract Cancellation

Ejemplo:

Annual Contract
$12,000


Recognized
$5,000


Deferred
$7,000

Si se cancela:

Remaining Deferred
$7,000

no necesariamente se reconoce inmediatamente.

La acción depende de las obligaciones restantes y de la política contable.

40. Refund and Revenue

Un refund puede requerir:

Revenue Adjustment
+
Deferred Revenue Adjustment
+
Accounting Adjustment

Por eso:

Refund
≠
Revenue Reversal

aunque puedan estar relacionados.

41. Credit and Revenue

Un credit puede afectar Revenue dependiendo de su naturaleza.

Ejemplo:

Original Revenue
$10,000


Commercial Credit
-$1,000


Adjusted Revenue
$9,000

La plataforma debe conservar la razón del ajuste.

42. Discount and Revenue

Si:

Price
$10,000


Discount
20%

entonces:

Transaction Price
$8,000

El Revenue Recognition debe basarse en el precio de transacción correspondiente, no en el precio de lista.

43. Revenue Allocation After Discount

Ejemplo:

Platform SSP
$100K


AI SSP
$50K


Total SSP
$150K


Contract Price
$120K

El descuento total es:

$30K

La asignación debe seguir la metodología definida.

44. Revenue by Product

Dashboard:

Revenue
│
├── Platform       $2.5M
├── AI             $1.2M
├── Mobility       $900K
├── Storage        $500K
└── Support        $300K
45. Revenue by Customer
ACME
$1.2M


Globex
$800K


Customer C
$500K
46. Revenue by Tenant
Tenant A
$3.0M


Tenant B
$1.5M


Tenant C
$900K
47. Revenue by Contract
Contract
CTR-000123


Total Contract Value
$1.2M


Recognized
$800K


Deferred
$400K
48. ARR

Annual Recurring Revenue puede calcularse desde las obligaciones recurrentes.

Ejemplo:

Monthly Recurring Revenue
$100,000


ARR
$1,200,000

Pero:

ARR
≠
Revenue Recognized
49. MRR
MRR
=
Recurring Monthly Revenue

Puede provenir de:

subscriptions;
recurring plans;
recurring usage commitments.

Debe excluir componentes no recurrentes según la política definida.

50. Bookings

Bookings representa compromisos comerciales:

New Contract
$1,000,000

Pero:

Bookings
≠
Billings
≠
Revenue
≠
Cash

Esta distinción será fundamental para Revenue Intelligence.

51. Bookings vs Billings vs Revenue vs Cash
Contract Signed
      ↓
   Bookings
      ↓
Invoice
      ↓
   Billings
      ↓
Payment
      ↓
    Cash
      ↓
Recognition
      ↓
   Revenue

Estos conceptos deben mantenerse separados.

52. Revenue Backlog

Si existen contratos futuros:

Contract Value
$1M


Recognized
$400K


Remaining
$600K

puede existir:

Revenue Backlog
$600K
53. Remaining Performance Obligations

El sistema puede mostrar:

Total Contract
$1,000,000


Recognized
$400,000


Remaining Performance Obligation
$600,000

Esto es muy importante para Enterprise SaaS.

54. Revenue Forecasting

El sistema puede proyectar:

Current Revenue
$500K/month


Contracted Future Revenue
$2.5M


Expected New Business
$500K

y producir:

Forecast
$3.0M

Debe diferenciar:

Committed
Expected
Probabilistic
55. Revenue Forecast Categories
Committed Revenue
= Contracted


Expected Revenue
= High confidence


Forecast Revenue
= Model-based


Potential Revenue
= Pipeline
56. Revenue Forecast Model
Contracts
+
Subscriptions
+
Usage Forecast
+
Renewal Probability
-
Churn
-
Contraction
=
Revenue Forecast
57. Revenue Recognition Forecast

También puede proyectar:

August
$500K


September
$520K


October
$550K

basado en schedules existentes.

58. Revenue Forecast vs Actual
Month       Forecast    Actual
August      $500K       $490K
September   $520K       $530K
October     $550K       —

La plataforma debe analizar:

Variance
59. Revenue Variance
Actual
$530K


Forecast
$520K


Variance
+$10K

Puede clasificarse:

New Business
Expansion
Churn
Timing
Usage
Pricing
Adjustments
60. Revenue Analytics

KPIs principales:

Revenue;
MRR;
ARR;
bookings;
billings;
deferred revenue;
recognized revenue;
revenue backlog;
revenue growth;
revenue retention;
churn;
expansion;
contraction.
61. Revenue Recognition Analytics
Recognized Revenue
$5M


Deferred Revenue
$2M


Accrued Revenue
$500K

Permite entender la posición económica del negocio.

62. Revenue by Recognition Method
Straight Line
$3M


Usage Based
$1M


Milestone
$500K


Point in Time
$500K
63. Revenue Concentration

El sistema puede detectar:

Top Customer
35% Revenue

o:

Top 10 Customers
70% Revenue

Esto ayuda al análisis Enterprise.

64. Revenue Quality

No todo Revenue tiene el mismo valor.

Puede clasificarse:

Recurring
Non-Recurring
Usage
Professional Services
One-Time
Contractual

Esto alimenta métricas de calidad de Revenue.

65. Recurring Revenue

Ejemplo:

Subscription
$100K/month

Recurring Revenue:

$100K
66. Non-Recurring Revenue

Ejemplo:

Implementation
$50K

Puede reconocerse como Revenue pero no necesariamente cuenta como recurring revenue.

67. Revenue Recognition Policy

EVOXA debe permitir configurar políticas:

RevenuePolicy
│
├── accounting_standard
├── recognition_method
├── allocation_method
├── contract_modification_policy
├── refund_policy
├── credit_policy
└── version
68. Revenue Policy Versioning
Policy v1
Effective 2026


Policy v2
Effective 2027

Los contratos históricos conservan la política aplicable.

69. Revenue Contract

Modelo:

RevenueContract
│
├── id
├── contract_id
├── customer_id
├── billing_account_id
├── tenant_id
├── contract_value
├── currency
├── start_date
├── end_date
├── recognition_policy
└── status
70. Revenue Performance Obligation
PerformanceObligation
│
├── id
├── revenue_contract_id
├── product_id
├── quantity
├── standalone_selling_price
├── allocated_transaction_price
├── recognition_method
├── start_date
├── end_date
└── status
71. Revenue Schedule Model
RevenueSchedule
│
├── id
├── performance_obligation_id
├── period
├── scheduled_amount
├── recognized_amount
├── remaining_amount
├── status
└── metadata
72. Revenue Entry
RevenueEntry
│
├── id
├── revenue_schedule_id
├── period
├── amount
├── currency
├── entry_type
├── source_reference
├── status
└── created_at
73. Revenue Reversal

Nunca:

UPDATE revenue = 0

Correcto:

Original Revenue
+$10,000


Reversal
-$10,000


New Revenue
+$8,000

Net:

$8,000
74. Revenue Adjustment

Si se detecta:

Expected
$10,000


Correct
$9,000

se genera:

Revenue Adjustment
-$1,000

con:

Reason
Approval
Source
Period
75. Revenue Reconciliation

Debe reconciliar:

Billing
     ↓
Invoices
     ↓
Revenue Schedules
     ↓
Revenue Ledger
     ↓
Accounting

La plataforma debe identificar diferencias automáticamente.

76. Revenue Reconciliation Formula

Conceptualmente:

Opening Deferred
+
New Billings
-
Recognized Revenue
+
Adjustments
=
Ending Deferred

Y:

Revenue Ledger
=
Accounting Revenue

después de las diferencias y ajustes permitidos.

77. Revenue Close Exceptions

Ejemplos:

Missing Schedule
Unallocated Revenue
Negative Schedule
Duplicate Recognition
Unbalanced Contract
Invalid Recognition Date
Missing Performance Obligation
Accounting Mismatch
78. Revenue Exception Workflow
Exception
   ↓
Detected
   ↓
Assigned
   ↓
Investigated
   ↓
Corrected
   ↓
Approved
   ↓
Reconciled
79. Revenue Audit

Cada reconocimiento debe poder responder:

¿Por qué se reconocieron $1,000 este mes?

Ejemplo:

Contract
CTR-1001


Performance Obligation
Platform Subscription


Recognition Method
Straight Line


Contract Value
$12,000


Period
August 2026


Recognition
$1,000
80. Revenue Governance

Roles:

Billing
→ Creates invoices


Revenue Operations
→ Manages schedules


Finance
→ Approves policies


Accounting
→ Posts entries


Auditor
→ Reviews
81. Revenue Permissions
revenue.view
revenue.create
revenue.schedule.manage
revenue.recognize
revenue.adjust
revenue.reverse
revenue.close
revenue.forecast
revenue.report
82. Revenue APIs
Contracts
GET  /billing/revenue/contracts
POST /billing/revenue/contracts
Performance Obligations
GET  /billing/revenue/performance-obligations
POST /billing/revenue/performance-obligations
Schedules
GET /billing/revenue/schedules
POST /billing/revenue/schedules
Recognition
POST /billing/revenue/recognize
Forecast
GET /billing/revenue/forecast
83. Revenue Close API
POST /billing/revenue/periods/{period}/close

Y:

GET /billing/revenue/periods/{period}/status
84. Revenue Event Architecture
InvoiceIssued
      ↓
RevenueScheduleCreated
      ↓
RevenueRecognitionScheduled
      ↓
RevenueRecognized
      ↓
RevenuePosted

Para cambios:

ContractModified
      ↓
RevenueReallocation
      ↓
RevenueAdjustment
85. Revenue Event Sourcing

Para operaciones críticas:

Event
   ↓
Revenue State

En lugar de depender únicamente de un valor mutable.

Esto permite:

auditoría;
reconstrucción;
reversión;
debugging;
compliance.
86. Revenue and Multi-Tenant

Cada Revenue Entry debe poder relacionarse con:

Organization
Tenant
Legal Entity
Billing Account
Customer
Contract
Subscription
Product

Ejemplo:

Organization
   ↓
Tenant
   ↓
Billing Account
   ↓
Customer
   ↓
Contract
   ↓
Subscription
   ↓
Revenue
87. Revenue and Legal Entity

Muy importante para Enterprise:

Customer
ACME


Contract
$1M


Legal Entity
EVOXA Chile SpA

Revenue debe asociarse a la entidad legal correcta.

88. Intercompany Revenue

Si existen varias entidades:

EVOXA Chile
       ↓
EVOXA USA

pueden existir operaciones intercompany.

Estas deben integrarse con:

26/18 — Billing Accounting & Financial Integration

y no tratarse como Revenue externo normal.

89. Revenue Currency

Debe conservarse:

Contract Currency
USD


Billing Currency
USD


Revenue Currency
USD


Accounting Currency
CLP

Si existe conversión:

Exchange Rate

debe quedar registrada.

90. Revenue FX

Ejemplo:

Revenue
USD 10,000


FX Rate
950


Accounting
CLP 9,500,000

Las diferencias posteriores de FX deben distinguirse del Revenue original.

91. Revenue Forecasting with AI

La IA puede analizar:

Contracts
Subscriptions
Usage
Renewals
Churn
Expansion
Historical Revenue

para generar:

Revenue Forecast

Ejemplo:

Committed
$10M


Expected
$2M


Probabilistic
$1M

La IA debe complementar, no reemplazar, las reglas financieras oficiales.

92. Revenue Intelligence

Puede detectar:

Revenue Growth
Revenue Leakage
Unexpected Churn
Expansion
Contraction
Customer Concentration
Deferred Revenue Growth
Forecast Variance

Esto conecta con:

26/21 — Billing Analytics & Revenue Intelligence.

93. Revenue Leakage

Ejemplo:

Usage
1,000,000 units


Expected Revenue
$10,000


Actual Billing
$8,000

Potential Leakage:

$2,000

El sistema puede rastrear:

Usage
→ Rating
→ Charge
→ Invoice
→ Revenue

para encontrar dónde se produjo la diferencia.

94. Revenue Quality Monitoring

El sistema puede analizar:

Recurring Revenue
One-Time Revenue
Usage Revenue
Discounted Revenue
Deferred Revenue
Contracted Revenue

y producir un Revenue Quality Score.

95. Revenue Operations Dashboard
Revenue Management
│
├── Recognized Revenue
├── Deferred Revenue
├── Accrued Revenue
├── Bookings
├── Billings
├── MRR
├── ARR
├── Backlog
├── Forecast
├── Variance
└── Revenue Exceptions
96. Revenue Example — EVOXA

Supongamos:

Enterprise Contract
12 months


Contract Value
$120,000

Invoice:

$120,000

Payment:

$120,000

Revenue:

$10,000/month

Al tercer mes:

Recognized Revenue
$30,000


Deferred Revenue
$90,000
97. Revenue Example with Usage

Contrato:

Base Subscription
$10,000/month

Usage:

AI
$5,000

Total:

$15,000

Si corresponde reconocerlo durante el período de prestación:

Revenue
$15,000
98. Revenue Example with Discount
Subscription
$10,000


Enterprise Discount
-$2,000


Transaction Price
$8,000

Revenue Schedule:

12 months


$666.67/month

considerando las reglas de precisión y redondeo correspondientes.

99. Revenue Example with Credit
Contract
$12,000


SLA Credit
-$1,200


Adjusted Transaction Price
$10,800

Si corresponde distribuirlo:

$900/month

durante 12 meses.

100. Revenue Example with Cancellation
Contract
$120,000


Recognized
$40,000


Remaining
$80,000

Customer cancels.

La plataforma debe evaluar:

Contract Terms
+
Performance Obligations
+
Accounting Policy

antes de determinar qué sucede con los $80,000 restantes.

101. Core Principles
1. Revenue ≠ Billing

Una Invoice no determina automáticamente Revenue.

2. Revenue ≠ Cash

Payment recibido no significa necesariamente Revenue reconocido.

3. Deferred Revenue Matters

Los pagos anticipados deben mantenerse hasta que corresponda reconocerlos.

4. Performance Obligations

Los contratos complejos deben dividirse correctamente.

5. Allocation

El precio debe asignarse según las reglas definidas.

6. Versioning

Las políticas de Revenue deben estar versionadas.

7. Period Close

Los períodos cerrados deben protegerse.

8. Immutable Ledger

No modificar Revenue histórico destructivamente.

9. Reconciliation

Billing, Revenue y Accounting deben poder reconciliarse.

10. Multi-Entity

Revenue debe asociarse correctamente a la Legal Entity.

102. Resultado de 26/17

Billing Revenue Management transforma la información comercial y financiera de Billing en una estructura formal de Revenue reconocido, diferido, acumulado y proyectado.

La cadena:

Contract
   ↓
Performance Obligations
   ↓
Transaction Price
   ↓
Revenue Allocation
   ↓
Revenue Schedule
   ↓
Revenue Recognition
   ↓
Revenue Ledger
   ↓
Accounting

Y paralelamente:

Bookings
   ↓
Billings
   ↓
Payments
   ↓
Cash

permitiendo distinguir claramente:

Bookings
≠
Billings
≠
Payments
≠
Cash
≠
Revenue
Arquitectura acumulada 26/08 → 26/17
26/08  Billing Products & Catalog
          ↓
26/09  Billing Pricing & Plans
          ↓
26/10  Billing Subscriptions
          ↓
26/11  Billing Usage & Metering
          ↓
26/12  Billing Rating & Charge Calculation
          ↓
26/13  Billing Invoicing
          ↓
26/14  Billing Payments
          ↓
26/15  Billing Credits, Discounts & Adjustments
          ↓
26/16  Billing Taxes & Compliance
          ↓
26/17  Billing Revenue Management

Y ahora EVOXA tiene conceptualmente separadas las cuatro grandes dimensiones:

             BILLING
                │
     ┌──────────┼───────────┐
     ▼          ▼           ▼
 Commercial   Financial    Fiscal
     │          │           │
 Pricing      Payments      Tax
 Discounts    Revenue       Compliance
 Credits      Settlement

La siguiente capa:

26/18 — Billing Accounting & Financial Integration

será especialmente importante porque conectará Invoices, Payments, Taxes, Revenue, Credits, Refunds y Adjustments con el sistema contable, incluyendo Chart of Accounts, Journal Entries, Accounts Receivable, Accounts Payable cuando corresponda, General Ledger, subledgers, reconciliation, accounting periods, financial posting y ERP integrations.
