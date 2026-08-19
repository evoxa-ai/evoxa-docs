1. Billing Invoicing

Billing Invoicing es la capa responsable de transformar los Charges calculados por Billing en un documento financiero formal, consolidando cargos, descuentos, créditos, impuestos y ajustes para establecer la obligación de pago del Customer.

La relación con los capítulos anteriores es:

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

La distinción fundamental:

Charge
= cuánto corresponde cobrar por un concepto


Invoice
= documento que consolida lo que el cliente debe pagar
2. Propósito

El módulo de Invoicing debe permitir:

crear invoices;
consolidar charges;
generar invoice lines;
aplicar créditos;
aplicar ajustes;
calcular impuestos;
gestionar descuentos;
definir fechas;
gestionar estados;
emitir documentos;
enviar invoices;
generar PDF;
administrar notas de crédito;
soportar invoices recurrentes;
soportar invoices one-time;
consolidar múltiples subscriptions;
manejar múltiples tenants;
soportar múltiples monedas;
mantener trazabilidad;
permitir auditoría.
3. Invoice como Documento Financiero

Una Invoice representa formalmente:

La obligación financiera de un Customer frente a una Billing Account.

Ejemplo:

Customer
ACME Corporation


Billing Account
BA-001


Invoice
INV-2026-000123


Amount Due
$1,250 USD


Due Date
30/09/2026
4. Invoice vs Charge

Un Charge puede existir independientemente:

Charge A → $500
Charge B → $200
Charge C → $300

La Invoice los consolida:

Invoice
│
├── Charge A
├── Charge B
└── Charge C
      │
      ▼
Total = $1,000

Por lo tanto:

Charge
   ↓
Invoice Line

pero una Invoice puede contener múltiples líneas provenientes de múltiples Charges.

5. Invoice vs Payment

También deben mantenerse separados:

Invoice
= cuánto debe pagar


Payment
= cuánto pagó

Ejemplo:

Invoice
$1,000


Payment
$600


Remaining Balance
$400

Esto será desarrollado posteriormente en Billing Payments.

6. Invoice Lifecycle

El lifecycle puede ser:

DRAFT
   │
   ▼
FINALIZING
   │
   ▼
FINALIZED
   │
   ▼
ISSUED
   │
   ├───────────────┐
   ▼               ▼
PAID            PAST_DUE
                   │
                   ▼
                COLLECTION
                   │
                   ▼
                WRITTEN_OFF

También pueden existir:

VOIDED
CANCELED
REFUNDED
PARTIALLY_PAID
7. Invoice Status
Draft

Puede modificarse.

Finalizing

Se está consolidando y validando.

Finalized

El contenido financiero quedó congelado.

Issued

Fue emitida al cliente.

Partially Paid

Existe pago parcial.

Paid

Saldo completamente pagado.

Past Due

Venció la fecha de pago.

Void

Se anuló formalmente.

8. Invoice Number

Cada invoice debe tener un número único.

Ejemplo:

INV-2026-000001
INV-2026-000002
INV-2026-000003

La numeración debe estar gobernada.

En determinados países puede existir además numeración fiscal específica.

9. Invoice Identity

Modelo conceptual:

Invoice
│
├── id
├── invoice_number
├── customer_id
├── billing_account_id
├── organization_id
├── tenant_id
├── currency
├── status
├── issue_date
├── due_date
├── period_start
├── period_end
└── total
10. Billing Period

Una Invoice normalmente corresponde a un período:

01/08/2026
      ↓
31/08/2026

Por ejemplo:

Invoice
August 2026


Billing Period
01/08 → 31/08

Pero una invoice también puede incluir:

cargos atrasados;
ajustes;
one-time charges;
créditos.
11. Invoice Types

Puede existir:

RECURRING
ONE_TIME
USAGE
CREDIT
DEBIT
ADJUSTMENT
FINAL
PRO_FORMA

Ejemplo:

Recurring Invoice
→ Subscription


Usage Invoice
→ Consumption


Credit Note
→ Negative financial document
12. Invoice Lines

La Invoice está compuesta por líneas.

Invoice
│
├── Line 1
├── Line 2
├── Line 3
└── Line 4

Ejemplo:

Professional Plan       $500
50 Seats                $250
AI Usage                $200
Storage                  $50
────────────────────────────
Subtotal              $1,000
13. Invoice Line Structure

Cada línea puede contener:

InvoiceLine
│
├── id
├── invoice_id
├── product_id
├── plan_id
├── subscription_id
├── charge_id
├── description
├── quantity
├── unit_price
├── subtotal
├── discount
├── tax
├── total
└── metadata

Esto mantiene trazabilidad hasta el origen.

14. Invoice Line Traceability

Una línea debe poder responder:

¿Por qué apareció esta línea?

Ejemplo:

Invoice Line
   ↓
Charge
   ↓
Rating Result
   ↓
Pricing Rule
   ↓
Meter
   ↓
Usage

Esto es fundamental para disputas.

15. Invoice Consolidation

Una Billing Account puede tener varias subscriptions:

Billing Account
│
├── Subscription A
├── Subscription B
├── Subscription C
└── Subscription D

Puede generar:

One Consolidated Invoice

Ejemplo:

Platform        $500
Mobility        $300
AI              $250
Storage          $50
────────────────────
Total          $1,100
16. Multi-Subscription Invoicing

Una Invoice puede contener:

Subscription A
→ $500


Subscription B
→ $300


Subscription C
→ $200

Total:

$1,000

Pero cada línea conserva su subscription_id.

17. Multi-Tenant Invoicing

Para clientes Enterprise:

Corporate Billing Account
        │
        ▼
      Invoice
        │
 ┌──────┼──────┐
 ▼      ▼      ▼
Tenant A Tenant B Tenant C

La invoice puede mostrar:

Tenant A       $5,000
Tenant B       $3,000
Tenant C       $2,000
────────────────────
Total         $10,000
18. Invoice Subtotals

La estructura financiera:

Gross Charges
      ↓
Discounts
      ↓
Credits
      ↓
Net Subtotal
      ↓
Taxes
      ↓
Total

Ejemplo:

Gross Charges       $1,500
Discount              -$150
Credits                -$50
──────────────────────────
Net Subtotal         $1,300
Tax                    $247
──────────────────────────
Invoice Total        $1,547
19. Discounts on Invoice

Los descuentos pueden aparecer:

A nivel de línea
Plan
$500
Discount 10%
-$50
A nivel de Invoice
Subtotal
$2,000


Enterprise Discount
-$200

Debe existir una política clara sobre el orden de aplicación.

20. Credits

Los créditos pueden reducir el saldo:

Invoice
$1,000


Credit
-$100


Amount Due
$900

El crédito debe tener origen:

Credit
   ↓
Reason
   ↓
Approval
   ↓
Invoice
21. Taxes

La Invoice puede incluir impuestos:

Subtotal
$1,000


Tax
$190


Total
$1,190

Los impuestos pueden depender de:

customer;
location;
product;
tax category;
legal entity;
jurisdiction.

La lógica fiscal debe mantenerse desacoplada del cálculo de pricing.

22. Tax Lines

Puede existir detalle:

Invoice
│
├── Net Amount
├── VAT
├── Other Taxes
└── Total

Ejemplo:

Subtotal              $1,000
IVA 19%                 $190
────────────────────────────
Total                 $1,190

Para Chile, la configuración fiscal concreta deberá adaptarse a los requisitos tributarios aplicables.

23. Currency

Cada Invoice debe tener una moneda:

Currency = USD

No debería mezclarse directamente:

USD
+
CLP

en una misma Invoice sin una política explícita de conversión y presentación.

24. Exchange Rates

Si se requiere conversión:

Original Price
USD 1,000


Exchange Rate
950 CLP/USD


Converted
CLP 950,000

Debe conservarse:

exchange rate;
source;
effective date;
conversion method.
25. Invoice Dates

Una Invoice puede tener varias fechas:

Issue Date
Due Date
Billing Period Start
Billing Period End
Finalization Date
Payment Date

Ejemplo:

Period
01/08 → 31/08


Issue Date
01/09


Due Date
30/09
26. Payment Terms

Las condiciones pueden ser:

DUE_ON_RECEIPT
NET_7
NET_15
NET_30
NET_45
NET_60
NET_90
CUSTOM

Ejemplo:

Issue Date
01/09


Terms
Net 30


Due Date
01/10
27. Invoice Delivery

La Invoice puede enviarse mediante:

Email
Portal
API
EDI
ERP
Peppol / e-invoicing network

Para EVOXA:

Billing
   │
   ├── Customer Portal
   ├── Email
   └── API
28. Invoice PDF

La plataforma puede generar una representación documental:

Invoice
   │
   ▼
PDF

El PDF puede contener:

logo;
issuer;
customer;
invoice number;
dates;
line items;
taxes;
totals;
payment instructions;
legal information.

El PDF no debe ser la fuente primaria de datos.

29. Invoice Data vs PDF

La fuente de verdad debe ser:

Invoice Database

El PDF es:

Document Representation

Esto permite:

Invoice Data
   ├── API
   ├── Portal
   ├── PDF
   ├── Email
   └── ERP
30. Invoice Finalization

Una Invoice en DRAFT puede modificarse.

Cuando se finaliza:

DRAFT
  ↓
FINALIZED

debe congelarse:

líneas;
cantidades;
precios;
descuentos;
impuestos;
totales.

Después de finalizada, las correcciones deben utilizar mecanismos financieros apropiados.

31. Invoice Immutability

Incorrecto:

Invoice
$1,000


UPDATE
$900

Correcto:

Original Invoice
$1,000


Credit Note
-$100


Net Balance
$900

Esto preserva auditoría.

32. Invoice Corrections

Las correcciones pueden realizarse mediante:

Credit Note
Debit Note
Adjustment
Void
Replacement Invoice

Dependiendo del caso.

33. Credit Note

Una Credit Note reduce una obligación.

Original Invoice
$1,000


Credit Note
-$200


Adjusted Amount
$800

Debe referenciar la invoice original.

34. Debit Note

Una Debit Note incrementa una obligación:

Original Invoice
$1,000


Additional Charge
$100


Debit Note
+$100


Adjusted Amount
$1,100
35. Void Invoice

Una Invoice puede anularse:

ISSUED
  ↓
VOIDED

Pero el sistema debe conservar:

invoice number;
original data;
reason;
actor;
timestamp.
36. Replacement Invoice

En algunos casos:

Invoice Original
       │
       ▼
Void
       │
       ▼
Replacement Invoice

La relación debe mantenerse.

37. Invoice Payment Status

El estado financiero debe poder ser:

UNPAID
PARTIALLY_PAID
PAID
OVERPAID
PAST_DUE
WRITTEN_OFF

Ejemplo:

Invoice
$1,000


Payment
$700


Status
PARTIALLY_PAID


Balance
$300
38. Invoice Balance

La fórmula:

Amount Due
-
Payments
-
Applied Credits
+
Additional Adjustments
=
Outstanding Balance

Ejemplo:

Invoice             $1,000
Payment              -$600
Credit                -$100
──────────────────────────
Balance               $300
39. Invoice Aging

Las invoices pueden clasificarse:

Current
1–30 Days
31–60 Days
61–90 Days
90+ Days

Esto alimenta Collections.

40. Invoice Dunning

Cuando una invoice queda vencida:

Invoice
   │
   ▼
Past Due
   │
   ▼
Dunning
   │
   ├── Reminder
   ├── Retry Payment
   ├── Warning
   ├── Restriction
   └── Collection

Esto conecta con Billing Dunning & Collections.

41. Invoice Consolidation Rules

El sistema debe determinar cuándo agrupar charges:

Group By
│
├── Billing Account
├── Customer
├── Currency
├── Legal Entity
├── Billing Period
└── Payment Terms

Por ejemplo, dos subscriptions con monedas diferentes normalmente requieren invoices separadas.

42. Invoice Grouping

Ejemplo:

Customer
│
├── USD Subscriptions
│     └── Invoice USD
│
└── CLP Subscriptions
      └── Invoice CLP

Esto evita mezclar obligaciones incompatibles.

43. Invoice Generation Pipeline
Billing Period
      │
      ▼
Collect Charges
      │
      ▼
Validate Charges
      │
      ▼
Group Charges
      │
      ▼
Create Invoice
      │
      ▼
Create Invoice Lines
      │
      ▼
Apply Credits
      │
      ▼
Calculate Taxes
      │
      ▼
Calculate Totals
      │
      ▼
Finalize
      │
      ▼
Issue
      │
      ▼
Deliver
44. Invoice Validation

Antes de finalizar:

Invoice Validation
│
├── Customer valid?
├── Billing Account valid?
├── Currency consistent?
├── Charges valid?
├── Taxes valid?
├── Totals correct?
├── Invoice number valid?
├── Dates valid?
└── Payment terms valid?

Si falla:

Invoice = ERROR

y no debe emitirse.

45. Invoice Reconciliation

Debe existir reconciliación:

Charges
   ↓
Invoice Lines
   ↓
Invoice Total

Ejemplo:

Charges
$10,000


Invoice Lines
$10,000


Variance
$0

Si:

Charges = $10,000
Invoice = $9,950

debe generarse una excepción.

46. Invoice Idempotency

La generación de invoices debe ser idempotente.

Si un job se ejecuta dos veces:

Generate August Invoice

no debería crear:

INV-001
INV-002

para el mismo período y contexto.

Debe detectar:

Existing Invoice

y evitar duplicación.

47. Invoice Close

El período de facturación puede tener:

OPEN
  ↓
CHARGES_READY
  ↓
INVOICE_GENERATION
  ↓
FINALIZATION
  ↓
ISSUED
  ↓
CLOSED

Una vez cerrado, cualquier corrección debe seguir el flujo de ajustes.

48. Invoice Numbering

Puede utilizarse:

INV-2026-000001
INV-2026-000002

O estructuras por organización:

EVOXA-CL-2026-000001
EVOXA-US-2026-000001

La numeración debe evitar:

duplicados;
reutilización;
modificaciones.
49. Invoice Legal Entity

En Enterprise puede existir:

Organization
   │
   ├── Legal Entity Chile
   ├── Legal Entity USA
   └── Legal Entity Mexico

La Invoice debe identificar correctamente la entidad emisora.

Esto afecta:

impuestos;
moneda;
numeración;
payment instructions;
accounting.
50. Invoice Customer Information

Debe conservarse un snapshot de la información relevante:

Customer Name
Billing Address
Tax ID
Email
Payment Terms
Tax Profile

No debería depender exclusivamente del Customer actual después de emitir la Invoice.

51. Historical Snapshot

Ejemplo:

Customer
Name = ACME


Invoice 2026
Name = ACME Corporation


Customer changes later
Name = ACME Global Holdings

La Invoice histórica debe continuar mostrando:

ACME Corporation

porque esa era la información vigente al momento de emisión.

52. Invoice Metadata

Puede almacenar:

External ERP ID
Purchase Order
Contract Number
Cost Center
Department
Project
Customer Reference
Tax Reference

Esto es muy importante para Enterprise.

53. Purchase Orders

Una Invoice puede asociarse a:

PO Number

Ejemplo:

PO-2026-00125

La invoice puede requerir validación:

Invoice
   ↓
Purchase Order
   ↓
Contract
54. Invoice Delivery Status

Debe registrarse:

GENERATED
QUEUED
SENT
DELIVERED
BOUNCED
FAILED
VIEWED

Esto permite saber si el cliente recibió la factura.

55. Invoice Events

Eventos principales:

InvoiceCreated
InvoiceFinalized
InvoiceIssued
InvoiceSent
InvoiceViewed
InvoicePaymentReceived
InvoicePartiallyPaid
InvoicePaid
InvoicePastDue
InvoiceVoided
InvoiceCredited
InvoiceAdjusted

Estos eventos alimentan:

notifications;
collections;
analytics;
accounting.
56. Invoice Audit

Debe conservar:

Actor
Timestamp
Previous State
New State
Reason
Invoice
Correlation ID

Ejemplo:

InvoiceVoided


Reason:
Duplicate invoice


Actor:
Billing Administrator
57. Invoice API
Core
POST   /billing/invoices
GET    /billing/invoices
GET    /billing/invoices/{id}
Lifecycle
POST /billing/invoices/{id}/finalize
POST /billing/invoices/{id}/issue
POST /billing/invoices/{id}/void
Documents
GET /billing/invoices/{id}/pdf
Delivery
POST /billing/invoices/{id}/send
58. Invoice Preview

Antes de finalizar:

POST /billing/invoices/preview

Puede mostrar:

Charges
Discounts
Credits
Taxes
Total
Due Date

Esto es importante para evitar errores antes de emitir.

59. Invoice Data Model

Modelo conceptual:

Invoice
│
├── id
├── invoice_number
├── customer_id
├── billing_account_id
├── organization_id
├── tenant_id
├── legal_entity_id
├── currency
├── status
├── invoice_type
├── issue_date
├── due_date
├── period_start
├── period_end
├── subtotal
├── discount_total
├── credit_total
├── tax_total
├── total
├── amount_paid
├── amount_due
├── purchase_order
├── contract_id
├── payment_terms
├── billing_address_snapshot
├── customer_snapshot
├── metadata
├── created_at
└── updated_at
60. Invoice Line Data Model
InvoiceLine
│
├── id
├── invoice_id
├── charge_id
├── subscription_id
├── product_id
├── plan_id
├── description
├── quantity
├── unit
├── unit_price
├── gross_amount
├── discount_amount
├── net_amount
├── tax_amount
├── total_amount
├── period_start
├── period_end
└── metadata
61. Invoice Relationships
Customer
   │
   ▼
Billing Account
   │
   ├── Subscription A
   │      └── Charges
   │
   ├── Subscription B
   │      └── Charges
   │
   └── Subscription C
          └── Charges
               │
               ▼
            Invoice
               │
        ┌──────┼──────┐
        ▼      ▼      ▼
      Lines  Taxes  Credits
               │
               ▼
             Payment
62. Invoice Accounting Integration

Una Invoice debe poder alimentar:

Invoice
   │
   ▼
Accounting
   │
   ├── Accounts Receivable
   ├── Revenue
   ├── Tax Payable
   └── Deferred Revenue

La contabilización exacta pertenece a la capa de Accounting.

63. Invoice Revenue Separation

Importante:

Invoice
≠
Revenue

Una invoice puede ser:

$12,000 Annual Subscription

pero el revenue podría reconocerse:

$1,000/month

según las reglas contables.

Esto será desarrollado posteriormente en Billing Revenue Management.

64. Invoice and Cash

También:

Invoice
$10,000


Payment
$0


Cash
$0

Después:

Invoice
$10,000


Payment
$10,000


Cash
$10,000

Por eso:

Billing
≠
Payment
≠
Revenue Recognition
65. Invoice Analytics

KPIs principales:

Volume
invoices generated;
invoices issued;
invoices paid.
Revenue
invoiced revenue;
average invoice value;
recurring invoice value.
Collections
outstanding balance;
overdue amount;
DSO.
Quality
invoice errors;
failed delivery;
credit notes;
adjustments.
66. Invoice Aging Analytics
Outstanding
│
├── Current
├── 1–30
├── 31–60
├── 61–90
└── 90+

Esto permite construir:

Accounts Receivable Aging

y alimentar Collections.

67. Invoice Disputes

Cuando el cliente cuestiona una factura:

Customer
   │
   ▼
Invoice
   │
   ▼
Invoice Line
   │
   ▼
Charge
   │
   ▼
Rating
   │
   ▼
Usage

Esto permite investigar la causa real.

68. Invoice Dispute Flow
Dispute Opened
      │
      ▼
Investigate
      │
      ├── Valid
      │     ↓
      │   Adjustment
      │
      └── Invalid
            ↓
          Reject

El resultado debe quedar auditado.

69. Enterprise Invoice Model

Para EVOXA Enterprise:

                       CORPORATE CUSTOMER
                              │
                              ▼
                       BILLING ACCOUNT
                              │
                              ▼
                       LEGAL ENTITY
                              │
                              ▼
                         INVOICING
                              │
             ┌────────────────┼────────────────┐
             ▼                ▼                ▼
          Tenant A         Tenant B         Tenant C
             │                │                │
             ▼                ▼                ▼
          Charges           Charges           Charges
             │                │                │
             └────────────────┼────────────────┘
                              ▼
                     CONSOLIDATED INVOICE
                              │
             ┌────────────────┼────────────────┐
             ▼                ▼                ▼
         Platform           Mobility            AI
          $5,000             $3,000           $2,000
                              │
                              ▼
                          $10,000
70. EVOXA Invoice Example
EVOXA
Enterprise Invoice


Customer:
ACME Corporation


Billing Period:
01/08/2026 – 31/08/2026


────────────────────────────────────
Professional Platform        $500
Mobility Usage               $300
AI Input Tokens              $120
AI Output Tokens             $180
Storage                       $50
Premium Support               $50
────────────────────────────────────
Subtotal                   $1,200


Enterprise Discount          -$120
────────────────────────────────────
Net Subtotal               $1,080


Tax                          $205.20
────────────────────────────────────
TOTAL                      $1,285.20

Cada línea mantiene referencia a:

Product
Plan
Subscription
Charge
Usage
Pricing
71. Invoice Architecture
                         BILLING
                            │
             ┌──────────────┼──────────────┐
             ▼              ▼              ▼
       Subscriptions      Charges        Credits
             │              │              │
             └──────────────┼──────────────┘
                            ▼
                    INVOICE ENGINE
                            │
                  ┌─────────┼─────────┐
                  ▼         ▼         ▼
               Grouping   Tax      Discounts
                  │         │         │
                  └─────────┼─────────┘
                            ▼
                         Invoice
                            │
             ┌──────────────┼──────────────┐
             ▼              ▼              ▼
           PDF            Email           API
                            │
                            ▼
                         Customer
72. Invoice Closing Architecture
Billing Period
      │
      ▼
Usage Finalized
      │
      ▼
Rating Complete
      │
      ▼
Charges Posted
      │
      ▼
Invoice Generation
      │
      ▼
Invoice Validation
      │
      ▼
Invoice Finalization
      │
      ▼
Invoice Issuance
      │
      ▼
Payment

Esto establece una secuencia controlada y evita facturar antes de completar los cálculos.

73. Core Principles
1. Invoice ≠ Charge

La Invoice consolida cargos; no reemplaza el Charge.

2. Invoice ≠ Payment

La factura representa una obligación; el Payment representa dinero recibido.

3. Invoice ≠ Revenue

La emisión de una factura no determina necesariamente el reconocimiento contable del revenue.

4. Immutable After Finalization

Una Invoice finalizada debe ser esencialmente inmutable.

5. Traceability

Cada línea debe poder rastrearse hasta su Charge y Usage.

6. Idempotency

El mismo período no debe generar invoices duplicadas.

7. Multi-Currency

Debe soportar monedas y políticas de conversión.

8. Multi-Tenant

Debe permitir consolidación Enterprise.

9. Tax Separation

Pricing y Tax deben mantenerse conceptualmente separados.

10. Auditability

Toda modificación debe generar un evento auditable.

74. Resultado de 26/13

Billing Invoicing transforma los resultados financieros calculados por Rating en un documento formal de obligación de pago, consolidando:

Charges
+
Discounts
+
Credits
+
Adjustments
+
Taxes
=
Invoice

La arquitectura acumulada queda:

26/08  Products & Catalog
          ↓
26/09  Pricing & Plans
          ↓
26/10  Subscriptions
          ↓
26/11  Usage & Metering
          ↓
26/12  Rating & Charge Calculation
          ↓
26/13  Invoicing
          ↓
26/14  Payments
          ↓
26/15  Credits, Discounts & Adjustments
          ↓
26/16  Taxes & Compliance
          ↓
26/17  Revenue Management
          ↓
...

Y el principio fundamental de este capítulo es:

Billing Invoicing debe convertir de manera controlada, trazable e inmutable los cargos calculados en obligaciones financieras formales, consolidando múltiples subscriptions y cargos en invoices coherentes, auditables y fiscalmente configurables, manteniendo separadas las responsabilidades de billing, payment, taxation y revenue recognition.

26/14 — Billing Payments

será la siguiente capa, donde la obligación creada por la Invoice se convierte en un pago recibido, aplicado, conciliado y posteriormente contabilizado.
