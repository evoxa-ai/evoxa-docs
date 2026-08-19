1. Billing Credits, Discounts & Adjustments

Billing Credits, Discounts & Adjustments es la capa responsable de administrar todas las modificaciones comerciales y financieras que pueden afectar el importe que un Customer debe pagar, sin alterar destructivamente los Charges, Invoices o Payments históricos.

Esta capa permite manejar:

descuentos;
créditos;
promociones;
compensaciones;
ajustes;
notas de crédito;
notas de débito;
correcciones;
write-offs;
goodwill credits;
refunds relacionados con billing;
excepciones comerciales;
acuerdos especiales Enterprise.

La cadena queda:

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
Discount / Credit / Adjustment
   ↓
Invoice
   ↓
Payment
   ↓
Settlement
2. Propósito

El módulo debe permitir modificar el resultado económico de Billing de forma controlada, trazable y auditable.

Debe soportar:

descuentos;
promociones;
créditos;
ajustes positivos;
ajustes negativos;
notas de crédito;
notas de débito;
compensaciones;
write-offs;
correcciones de billing;
créditos por SLA;
créditos por errores;
créditos promocionales;
créditos por sobrecobro;
ajustes manuales;
ajustes automáticos;
aprobaciones;
expiración;
límites;
auditoría.
3. Diferencia entre Discount, Credit y Adjustment

Es fundamental no mezclarlos.

Discount

Reduce el precio antes o durante el cálculo del Charge.

Price
$1,000


Discount
10%


Net
$900
Credit

Representa un valor monetario disponible o aplicado a favor del Customer.

Credit
$100

Puede utilizarse contra futuras invoices.

Adjustment

Modifica financieramente un importe existente.

Original
$1,000


Adjustment
-$100


Adjusted
$900
4. Modelo conceptual
                BILLING ADJUSTMENT LAYER
                         │
        ┌────────────────┼────────────────┐
        ▼                ▼                ▼
     Discount          Credit         Adjustment
        │                │                │
        ▼                ▼                ▼
      Pricing         Balance         Invoice/Charge
        │                │                │
        └────────────────┼────────────────┘
                         ▼
                       Invoice
5. Discount Lifecycle

Un descuento puede tener:

DRAFT
   ↓
ACTIVE
   ↓
EXPIRED
   ↓
ARCHIVED

También:

PENDING_APPROVAL
   ↓
APPROVED
   ↓
ACTIVE
6. Credit Lifecycle

Un crédito puede tener:

CREATED
   ↓
AVAILABLE
   ↓
PARTIALLY_APPLIED
   ↓
FULLY_APPLIED

o:

AVAILABLE
   ↓
EXPIRED
7. Adjustment Lifecycle
DRAFT
   ↓
PENDING_APPROVAL
   ↓
APPROVED
   ↓
POSTED
   ↓
APPLIED

Una vez posteado, no debería editarse destructivamente.

8. Discounts

Los descuentos pueden aplicarse a diferentes niveles:

Customer
   │
Billing Account
   │
Subscription
   │
Product
   │
Plan
   │
Charge
   │
Invoice

Ejemplo:

Enterprise Customer
      ↓
20% Discount
      ↓
All EVOXA Platform Services
9. Discount Types

Puede soportarse:

PERCENTAGE
FIXED_AMOUNT
UNIT_PRICE
TIER_DISCOUNT
VOLUME_DISCOUNT
BUNDLE_DISCOUNT
PROMOTIONAL
CONTRACTUAL
LOYALTY
PARTNER
ENTERPRISE
10. Percentage Discount
Original
$1,000


Discount
20%


Discount Amount
$200


Net
$800

Modelo:

Discount Amount
=
Base Amount × Discount %
11. Fixed Discount
Original
$1,000


Discount
$150


Net
$850
12. Unit Price Discount

En lugar de reducir el total:

Original Rate
$0.010


Discounted Rate
$0.008

Para:

100,000 units

resultado:

$800
13. Tier Discount

Puede variar según consumo:

0–10K
0%


10K–100K
10%


100K+
20%

Esto conecta directamente con:

26/09 — Billing Pricing & Plans

y:

26/12 — Billing Rating & Charge Calculation.

14. Volume Discount

Ejemplo:

Quantity
500,000


Volume Tier
500K


Discount
25%

El descuento puede aplicarse sobre todo el volumen según la regla configurada.

15. Graduated Discount

Cada tramo puede tener una reducción diferente:

First 10K
0%


Next 90K
10%


Next 900K
20%

El motor debe distinguir claramente:

Volume
vs
Graduated
16. Bundle Discount

Ejemplo:

Platform
+
Mobility
+
AI

Compra conjunta:

Bundle Discount
15%

El descuento se aplica por la combinación de productos.

17. Promotional Discount

Ejemplo:

NEWCUSTOMER20

Puede otorgar:

20% off
First 3 months

Debe incluir:

Start Date
End Date
Usage Limit
Customer Eligibility
Product Eligibility
18. Contractual Discount

Para Enterprise:

Standard Price
$10,000


Contract Discount
30%


Net
$7,000

Debe estar vinculado al contrato:

Contract
   ↓
Pricing Terms
   ↓
Discount
19. Customer-Specific Discount

Un Customer puede tener condiciones especiales:

Customer
ACME


Discount
25%

Esto debe tener prioridad y alcance claramente definidos.

20. Discount Scope

Un descuento puede aplicarse a:

ONE_PRODUCT
PRODUCT_CATEGORY
PLAN
SUBSCRIPTION
CUSTOMER
BILLING_ACCOUNT
TENANT
INVOICE
GLOBAL

Ejemplo:

20% only on AI Services

no debería afectar:

Mobility
Storage
Support
21. Discount Eligibility

El motor debe comprobar:

Customer eligible?
Product eligible?
Plan eligible?
Region eligible?
Contract valid?
Date valid?
Usage valid?

Solo si todas las reglas necesarias se cumplen:

Discount
→ Applied
22. Discount Priority

Puede haber varios descuentos:

Enterprise
20%


Promotion
10%


Partner
5%

La plataforma debe tener una regla de prioridad.

Ejemplo:

Contract
   ↓
Enterprise
   ↓
Promotion
23. Discount Stacking

Debe definirse si pueden acumularse.

Stacking permitido
$1,000
↓ 20%
$800
↓ 10%
$720
Stacking prohibido
20% Enterprise
+
10% Promotion


→ Only Enterprise Discount

La política debe ser explícita.

24. Maximum Discount

Puede existir:

Maximum Discount
30%

Aunque existan varias promociones:

20%
+
20%
=
40%

el motor limitará:

30%

si esa es la regla contractual.

25. Discount Expiration

Un descuento puede tener:

Start
01/08/2026


End
31/10/2026

Después:

Discount
EXPIRED

La siguiente invoice vuelve al precio normal.

26. Discount Duration

Puede definirse:

FIRST_INVOICE
FIRST_3_MONTHS
FIRST_YEAR
LIFETIME
CUSTOM_PERIOD

Ejemplo:

30% off
first 3 months
27. Credit

Un Credit representa valor financiero a favor del Customer.

Ejemplo:

Credit Balance
$500

Puede utilizarse para:

Invoice A
$300


Remaining Credit
$200
28. Credit Sources

Los créditos pueden provenir de:

OVERPAYMENT
SERVICE_CREDIT
PROMOTION
REFUND_TO_CREDIT
SLA_COMPENSATION
BILLING_ERROR
CUSTOMER_RETENTION
MANUAL_ADJUSTMENT
CONTRACTUAL_CREDIT
29. Credit Balance

El Customer puede tener:

Available Credit
$1,000

Después:

Invoice
$700


Credit Applied
$700


Remaining Credit
$300
30. Credit Application

La aplicación debe registrarse:

Credit
$1,000
   │
   ├── Invoice A → $500
   ├── Invoice B → $300
   └── Remaining → $200

Esto requiere:

CreditApplication
31. Credit Allocation

Modelo:

Credit
   │
   ├── Customer
   ├── Billing Account
   ├── Invoice
   ├── Amount
   └── Applied Date

Esto permite saber exactamente dónde se utilizó.

32. Credit Expiration

Un crédito puede tener:

Created
01/08


Expires
31/12

Si no se utiliza:

EXPIRED

La política debe indicar qué ocurre con el saldo restante.

33. Credit Restrictions

Un crédito puede restringirse:

Only AI Products
Only Mobility
Only Customer X
Only Tenant Y
Only Subscription Z

Ejemplo:

Credit
$500


Scope
AI Platform

No debería utilizarse para:

Storage
Support
Mobility
34. SLA Credits

Para Enterprise:

Service Availability
99.2%


Contract SLA
99.9%

El contrato puede establecer:

SLA Credit
10%

Resultado:

Invoice
$10,000


SLA Credit
-$1,000


Adjusted
$9,000
35. Billing Error Credit

Si se detecta un error:

Original Invoice
$1,500


Correct Amount
$1,300

Debe generarse:

Credit
-$200

No modificar la invoice histórica.

36. Goodwill Credit

Customer Success puede otorgar:

Goodwill Credit
$100

Debe requerir:

Reason
Approver
Expiration
Scope

Esto evita abuso.

37. Adjustment

Un Adjustment es una modificación financiera controlada.

Puede ser:

POSITIVE
NEGATIVE

Ejemplo:

Charge
$1,000


Adjustment
-$100


Net
$900
38. Adjustment Types
BILLING_CORRECTION
MANUAL_ADJUSTMENT
CONTRACT_ADJUSTMENT
ROUNDING
TAX_ADJUSTMENT
PRICE_CORRECTION
USAGE_CORRECTION
FX_ADJUSTMENT
WRITE_OFF
39. Positive Adjustment

Aumenta la obligación:

Invoice
$1,000


Adjustment
+$100


Adjusted Total
$1,100

Puede utilizarse cuando:

faltó un charge;
se corrigió un error;
se detectó usage tardío.
40. Negative Adjustment

Reduce la obligación:

Invoice
$1,000


Adjustment
-$100


Adjusted Total
$900
41. Credit Note

Una Credit Note formaliza una reducción.

Original Invoice
$1,000


Credit Note
-$200


Outstanding
$800

Debe conservar referencia a:

Original Invoice
42. Debit Note

Una Debit Note incrementa el valor:

Original Invoice
$1,000


Debit Note
+$200


Outstanding
$1,200
43. Write-Off

Cuando una cantidad no se cobrará:

Outstanding
$500

puede existir:

Write-Off
-$500

Resultado:

Balance
$0

Pero:

Write-Off
≠
Payment

El dinero no fue recibido.

44. Write-Off Governance

Los write-offs deben tener:

Amount
Reason
Approver
Policy
Date
Customer
Invoice
Accounting Reference

Puede existir una matriz:

<$100
Manager


$100–$1,000
Finance Manager


>$1,000
Finance Director
45. Adjustment Approval

Para evitar fraude:

Adjustment
      │
      ▼
Approval Required?
      │
 ┌────┴────┐
 ▼         ▼
No        Yes
 │         │
 ▼         ▼
Post     Approval
            │
            ▼
           Post
46. Segregation of Duties

Idealmente:

Customer Success
→ Requests Credit


Billing
→ Calculates


Finance
→ Approves


Accounting
→ Posts

Ningún usuario debería poder realizar todas las etapas críticas sin control.

47. Adjustment Limits

Puede existir:

Manual Adjustment Limit
$1,000

Sobre ese valor:

Approval Required

Esto protege contra errores y abuso.

48. Adjustment Reason Codes

Debe existir un catálogo:

Reason Code
│
├── BILLING_ERROR
├── CUSTOMER_COMPENSATION
├── SLA_CREDIT
├── CONTRACT_CORRECTION
├── USAGE_CORRECTION
├── PRICING_ERROR
├── GOODWILL
├── REFUND_ADJUSTMENT
└── OTHER

Esto facilita reporting.

49. Adjustment Audit

Cada ajuste debe registrar:

Actor
Timestamp
Original Amount
Adjustment Amount
Reason
Approval
Invoice
Customer
Subscription

Ejemplo:

Adjustment
-$500


Reason
SLA_COMPENSATION


Requested By
Customer Success


Approved By
Finance


Posted
18/08/2026
50. Adjustment Immutability

Incorrecto:

Adjustment
-$500


UPDATE
-$200

Correcto:

Original Adjustment
-$500


Reversal
+$500


New Adjustment
-$200

Resultado neto:

-$200

Esto mantiene un historial completo.

51. Reversal

Un ajuste puede revertirse:

Original Adjustment
-$500


Reversal
+$500

Resultado:

Net Adjustment
$0

El original permanece intacto.

52. Credit vs Refund

Una distinción muy importante:

Credit
Money remains in Billing

Ejemplo:

Credit Balance
$500
Refund
Money returns to Customer

Ejemplo:

Payment
$1,000


Refund
$500

Por lo tanto:

Credit
≠
Refund
53. Credit vs Discount
Discount

Reduce un precio.

$1,000
-10%
$900
Credit

Reduce una obligación existente o futura.

Invoice
$1,000


Credit
-$100


Due
$900
54. Discount vs Adjustment
Discount

Se conoce antes o durante Rating.

Price
↓
Discount
↓
Charge
Adjustment

Corrige después:

Charge
↓
Invoice
↓
Adjustment

Esto es una distinción arquitectónica esencial.

55. Adjustment Timing

Los ajustes pueden producirse:

Before Invoice
Pricing
↓
Discount
↓
Charge
After Invoice
Invoice
↓
Credit / Debit
After Payment
Payment
↓
Refund / Credit

Cada escenario tiene un tratamiento diferente.

56. Credits and Payments

Supongamos:

Invoice
$1,000


Payment
$1,000

Luego:

Credit
$200

La plataforma puede:

devolver dinero;
crear customer credit;
aplicar credit a próxima invoice;

según la política comercial.

57. Post-Payment Credit

Ejemplo:

Invoice
$1,000


Payment
$1,000


SLA Credit
$200

El Customer ya pagó.

Entonces:

Customer Credit Balance
$200

o:

Refund
$200

según las reglas.

58. Credit Ledger

Debe existir un ledger:

Credit Ledger
│
├── Credit Created
├── Credit Applied
├── Credit Reversed
├── Credit Expired
├── Credit Transferred
└── Credit Refunded

Esto permite reconstruir el balance.

59. Credit Balance Calculation
Credits Created
+
Credits Returned
-
Credits Applied
-
Credits Expired
-
Credits Refunded
=
Available Credit

Ejemplo:

Created       $1,000
Applied        -$400
Refunded       -$100
Expired        -$50
─────────────────────
Available      $450
60. Credit Data Model
Credit
│
├── id
├── credit_number
├── customer_id
├── billing_account_id
├── tenant_id
├── source_type
├── source_reference
├── original_amount
├── remaining_amount
├── currency
├── reason_code
├── scope
├── effective_from
├── expires_at
├── status
├── created_by
├── approved_by
└── metadata
61. Credit Application Model
CreditApplication
│
├── id
├── credit_id
├── invoice_id
├── amount
├── currency
├── applied_at
├── applied_by
└── metadata
62. Discount Data Model
Discount
│
├── id
├── discount_code
├── name
├── type
├── value
├── currency
├── scope
├── eligibility_rules
├── stacking_policy
├── maximum_discount
├── start_date
├── end_date
├── usage_limit
├── status
└── metadata
63. Adjustment Data Model
Adjustment
│
├── id
├── adjustment_number
├── customer_id
├── billing_account_id
├── tenant_id
├── invoice_id
├── charge_id
├── type
├── direction
├── amount
├── currency
├── reason_code
├── description
├── status
├── requested_by
├── approved_by
├── posted_at
└── metadata
64. Discount Application

También conviene registrar:

DiscountApplication
│
├── discount_id
├── customer_id
├── subscription_id
├── invoice_id
├── invoice_line_id
├── original_amount
├── discount_amount
├── applied_at
└── metadata

Esto permite explicar exactamente cómo se aplicó el descuento.

65. Adjustment Architecture
                 ADJUSTMENT ENGINE
                        │
       ┌────────────────┼────────────────┐
       ▼                ▼                ▼
    Discount          Credit         Adjustment
       │                │                │
       ▼                ▼                ▼
    Pricing          Balance          Invoice
       │                │                │
       └────────────────┼────────────────┘
                        ▼
                   Financial Result
66. Enterprise Commercial Overrides

Un Customer Enterprise puede tener:

Standard Price
$10,000


Contract Price
$8,000


Volume Discount
-$500


SLA Credit
-$200

Resultado:

Final
$7,300

El sistema debe saber exactamente qué regla produjo cada modificación.

67. Pricing Override

Puede existir:

Standard
$100/user


Enterprise Contract
$75/user

Esto no necesariamente es un Discount tradicional.

Puede modelarse como:

Contractual Price Override

y debe diferenciarse de:

Discount
68. Commercial Rule Hierarchy

Una posible jerarquía:

Global Pricing
      ↓
Product Pricing
      ↓
Plan Pricing
      ↓
Customer Segment
      ↓
Enterprise Contract
      ↓
Promotion
      ↓
Discount
      ↓
Credits
      ↓
Adjustments

Las reglas exactas deben ser configurables.

69. Adjustment Precedence

Debe definirse qué sucede primero:

Gross Charge
      ↓
Contract Pricing
      ↓
Discount
      ↓
Credit
      ↓
Adjustment
      ↓
Tax

No todos los tipos deben seguir exactamente este orden.

Por eso el Billing Rules Engine debe controlar el orden de aplicación.

70. Credit and Tax

Debe definirse si un crédito afecta la base imponible.

Ejemplo:

Invoice
$1,000


Credit
-$100


Tax Base
¿$900 o $1,000?

La respuesta depende de:

naturaleza del crédito;
jurisdicción;
legislación;
tipo de documento.

Por eso Credits & Adjustments debe integrarse con 26/16 — Billing Taxes & Compliance.

71. Adjustment and Accounting

Un Adjustment puede producir:

Debit
Credit
Revenue Adjustment
Accounts Receivable Adjustment

La integración debe conservar:

Billing Adjustment
       ↓
Accounting Entry

sin que Billing tenga que convertirse en el sistema contable completo.

72. Adjustment and Revenue

Una corrección posterior puede afectar revenue:

Original Revenue
$10,000


Credit
-$1,000


Adjusted Revenue
$9,000

Pero la contabilización dependerá de las reglas de Revenue Recognition.

73. Credits and Customer Experience

El Customer Portal debe mostrar:

Available Credit
$350

Y:

Credit History
│
├── SLA Credit      +$200
├── Billing Credit +$100
├── Applied         -$150
└── Remaining       $150

Esto mejora transparencia.

74. Customer Portal

El cliente debería poder ver:

Billing
│
├── Invoices
├── Payments
├── Credits
├── Discounts
├── Adjustments
└── Balance

Pero los detalles internos de aprobación deben estar restringidos.

75. Admin Portal

Billing Administrators pueden acceder:

Customer
   │
   ├── Discounts
   ├── Credits
   ├── Adjustments
   ├── Approvals
   └── Audit

Con permisos específicos:

credits.view
credits.create
credits.approve
credits.apply
credits.reverse
76. Approval Workflow
Request
   │
   ▼
Validate
   │
   ▼
Risk / Limit Check
   │
   ▼
Approval
   │
   ▼
Post
   │
   ▼
Apply

Puede utilizar:

Maker
Checker

para operaciones sensibles.

77. Adjustment Automation

Algunos ajustes pueden automatizarse:

SLA Breach
   ↓
Calculate Credit
   ↓
Create Credit
   ↓
Apply Next Invoice

Otros deben requerir aprobación:

Manual $50,000 Credit
   ↓
Finance Approval
78. Credit Automation

Ejemplo:

Service Outage
      ↓
SLA Policy
      ↓
Customer Eligible
      ↓
Credit Calculation
      ↓
Credit Issued
      ↓
Next Invoice

Esto reduce trabajo manual.

79. Discount Automation

Ejemplo:

New Customer
      ↓
Eligibility
      ↓
Promotion
      ↓
20% Discount
      ↓
First 3 Invoices

El sistema debe detener automáticamente el descuento cuando termina su duración.

80. Adjustment Controls

Debe evitarse:

Admin
→ arbitrary amount
→ arbitrary customer
→ no approval

La plataforma debe controlar:

permissions;
limits;
reason codes;
approval;
audit;
segregation of duties.
81. Adjustment Fraud Prevention

Se pueden detectar:

Same User
→ Many Credits
→ Same Customer


Large Credits
→ Unusual Frequency


Repeated Goodwill Adjustments

Esto puede alimentar Risk/AI.

82. Analytics

KPIs:

Discounts
discount amount;
discount rate;
average discount;
discount by product;
discount by customer;
promotion performance.
Credits
issued credits;
applied credits;
expired credits;
outstanding credit balance.
Adjustments
adjustment volume;
adjustment value;
reason distribution;
approval rate;
manual vs automated.
83. Discount Leakage

Una métrica importante:

Expected Revenue
-
Actual Revenue
=
Discount Leakage

Ejemplo:

Standard
$1,000,000


Discounts
$150,000


Net
$850,000

La plataforma puede analizar si los descuentos están generando el valor comercial esperado.

84. Credit Exposure

La empresa puede tener:

Outstanding Customer Credits
$500,000

Esto representa potencial reducción futura de cash collection.

Debe monitorizarse.

85. Adjustment Exposure

También:

Pending Adjustments
$100,000

Esto permite Finanzas conocer obligaciones potenciales antes del cierre.

86. Billing Close

Antes de cerrar:

Usage
   ↓
Rating
   ↓
Charges
   ↓
Discounts
   ↓
Credits
   ↓
Adjustments
   ↓
Taxes
   ↓
Invoice

Debe existir un control para garantizar que no queden ajustes pendientes.

87. Reconciliation

La ecuación conceptual:

Charges
-
Discounts
-
Credits
+
Adjustments
+
Taxes
=
Invoice

Y:

Invoice
-
Payments
-
Credits Applied
=
Outstanding Balance

Esto permite reconciliar toda la cadena.

88. Full Billing Adjustment Flow
                       USAGE
                         │
                         ▼
                       RATING
                         │
                         ▼
                       CHARGE
                         │
                 ┌───────┼────────┐
                 ▼       ▼        ▼
             Discount  Credit  Adjustment
                 │       │        │
                 └───────┼────────┘
                         ▼
                      INVOICE
                         │
                         ▼
                      PAYMENT
                         │
                         ▼
                    SETTLEMENT
89. EVOXA Example

Supongamos:

Platform Subscription       $1,000
AI Usage                       $500
Mobility Usage                 $300
Storage                        $100
────────────────────────────────
Gross                         $1,900

Contrato Enterprise:

Discount 15%
-$285

SLA:

Credit
-$100

Corrección:

Adjustment
+$50

Resultado:

$1,900
-$285
-$100
+$50
──────
$1,565

Luego Tax:

Tax
+$297.35

Invoice:

TOTAL
$1,862.35

Cada elemento conserva su propia trazabilidad.

90. EVOXA Enterprise Model
                         CUSTOMER
                            │
                            ▼
                     BILLING ACCOUNT
                            │
                     ┌──────┴──────┐
                     ▼             ▼
                Contract        Subscriptions
                     │             │
                     ▼             ▼
                 Discounts       Charges
                     │             │
                     └──────┬──────┘
                            ▼
                         Credits
                            │
                            ▼
                       Adjustments
                            │
                            ▼
                         Invoice
91. API
Discounts
POST   /billing/discounts
GET    /billing/discounts
GET    /billing/discounts/{id}
PATCH  /billing/discounts/{id}
Customer Discounts
POST /billing/customers/{id}/discounts
Credits
POST /billing/credits
GET  /billing/credits
GET  /billing/credits/{id}
Apply Credit
POST /billing/credits/{id}/apply
Adjustments
POST /billing/adjustments
GET  /billing/adjustments
GET  /billing/adjustments/{id}
Approvals
POST /billing/adjustments/{id}/approve
POST /billing/adjustments/{id}/reject
Reversal
POST /billing/adjustments/{id}/reverse
92. Permissions

Se recomienda separar permisos:

billing.discounts.view
billing.discounts.create
billing.discounts.edit
billing.discounts.delete
billing.credits.view
billing.credits.create
billing.credits.approve
billing.credits.apply
billing.credits.reverse
billing.adjustments.view
billing.adjustments.create
billing.adjustments.approve
billing.adjustments.post
billing.adjustments.reverse
93. Audit Events

Eventos importantes:

DiscountCreated
DiscountApplied
DiscountRemoved
DiscountExpired


CreditCreated
CreditApproved
CreditApplied
CreditReversed
CreditExpired


AdjustmentCreated
AdjustmentApproved
AdjustmentRejected
AdjustmentPosted
AdjustmentReversed


CreditNoteIssued
DebitNoteIssued
WriteOffCreated
WriteOffApproved
94. Versioning

Las reglas deben tener versiones:

Discount v1
Discount v2


Credit Policy v1
Credit Policy v2


Adjustment Policy v1
Adjustment Policy v2

Los documentos históricos deben conservar la versión utilizada.

95. Multi-Tenant

Los descuentos, créditos y ajustes deben respetar:

Organization
   ↓
Tenant
   ↓
Billing Account
   ↓
Customer

Un administrador de Tenant A no debería poder modificar créditos de Tenant B.

96. Cross-Tenant Credits

En Enterprise podría existir:

Corporate Credit
$10,000

y permitir:

Tenant A → $4,000
Tenant B → $3,000
Tenant C → $3,000

Esto requiere políticas explícitas de asignación.

97. Cross-Subscription Credits

Un crédito puede ser:

Subscription Specific

o:

Billing Account Wide

Ejemplo:

Billing Account Credit
$1,000

aplicable a:

Platform
Mobility
AI

según las restricciones.

98. Credit Transfer

Puede permitirse:

Tenant A
Credit $500
       ↓
Corporate Pool
       ↓
Tenant B
Credit $500

Pero esto debe ser altamente controlado y auditable.

99. Core Principles
1. Never Mutate Financial History

No modificar destructivamente invoices o charges históricos.

2. Discounts Before Billing

Los descuentos deben entrar al cálculo de pricing/rating cuando corresponda.

3. Credits Are Balances

Los créditos representan valor disponible o aplicado.

4. Adjustments Are Explicit

Toda corrección debe ser una operación financiera identificable.

5. Everything Is Traceable

Cada modificación debe tener origen, razón y actor.

6. Approval Matters

Las operaciones de alto impacto requieren aprobación.

7. Expiration Must Be Explicit

Promociones y créditos pueden expirar.

8. Segregation of Duties

Solicitar, aprobar y contabilizar deben poder separarse.

9. Reversibility

Las operaciones deben poder revertirse mediante nuevas transacciones.

10. Multi-Tenant Isolation

Los créditos y ajustes nunca deben cruzar tenants accidentalmente.

100. Resultado de 26/15

Billing Credits, Discounts & Adjustments proporciona la capa de flexibilidad comercial y financiera que permite a EVOXA manejar excepciones y condiciones especiales sin romper la integridad histórica de Billing.

La cadena queda:

Usage
   ↓
Rating
   ↓
Charge
   ↓
Discount
   ↓
Credit
   ↓
Adjustment
   ↓
Tax
   ↓
Invoice
   ↓
Payment
   ↓
Settlement

Y la diferencia fundamental:

Discount
→ modifica el precio/cálculo


Credit
→ genera valor a favor del Customer


Adjustment
→ corrige un resultado financiero existente


Refund
→ devuelve dinero recibido

La arquitectura acumulada:

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
          ↓
26/18  Billing Accounting & Financial Integration
          ↓
26/19  Billing Dunning & Collections
          ↓
26/20  Billing Refunds, Adjustments & Disputes
          ↓
26/21  Billing Analytics & Revenue Intelligence
          ↓
26/22  Billing FinOps & Cost Optimization
          ↓
26/23  Billing Platform Evolution

Billing Credits, Discounts & Adjustments debe proporcionar un mecanismo controlado, versionado, reversible y auditable para modificar las obligaciones financieras de los clientes, separando claramente descuentos, créditos, ajustes, notas de crédito, notas de débito, write-offs y refunds, y garantizando que ninguna corrección destruya el historial financiero original.

Arquitectura completa de la capa hasta 26/15
                 ┌──────────────────────────────┐
                 │       BILLING PLATFORM       │
                 └──────────────┬───────────────┘
                                │
             ┌──────────────────┼──────────────────┐
             ▼                  ▼                  ▼
         CATALOG             PRICING          SUBSCRIPTIONS
             │                  │                  │
             └──────────────────┼──────────────────┘
                                ▼
                         USAGE & METERING
                                │
                                ▼
                       RATING & CHARGES
                                │
                    ┌───────────┼───────────┐
                    ▼           ▼           ▼
                Discounts     Credits    Adjustments
                    │           │           │
                    └───────────┼───────────┘
                                ▼
                            INVOICING
                                │
                                ▼
                            PAYMENTS
                                │
                                ▼
                           SETTLEMENT
                                │
                                ▼
                           ACCOUNTING

El siguiente capítulo es 26/16 — Billing Taxes & Compliance, donde debemos separar la lógica fiscal del pricing y billing core, incluyendo tax determination, tax jurisdictions, VAT/IVA, tax exemptions, tax IDs, electronic invoicing, fiscal documents, compliance, tax reporting y reglas multi-país.
