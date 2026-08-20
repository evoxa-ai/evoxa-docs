1. Billing Refunds, Adjustments & Disputes

Billing Refunds, Adjustments & Disputes es la capa responsable de gestionar todas las situaciones en las que una transacción de Billing debe ser corregida, revertida, compensada, disputada o devuelta después de haber sido creada.

Esta capa es especialmente importante porque debe modificar el resultado financiero sin destruir el historial original.

La regla fundamental es:

Nunca se debe borrar o modificar destructivamente una transacción financiera histórica; se debe generar una nueva transacción que explique la corrección.

2. Posición dentro de Billing

Hasta ahora:

26/13  Billing Invoicing
26/14  Billing Payments
26/15  Billing Credits, Discounts & Adjustments
26/16  Billing Taxes & Compliance
26/17  Billing Revenue Management
26/18  Billing Accounting & Financial Integration
26/19  Billing Dunning & Collections
26/20  Billing Refunds, Adjustments & Disputes

La relación:

Invoice
   │
   ├── Refund
   ├── Credit
   ├── Adjustment
   ├── Dispute
   ├── Chargeback
   ├── Reversal
   └── Correction
3. Objetivo

La plataforma debe administrar:

Refunds;
Partial Refunds;
Full Refunds;
Credits;
Credit Notes;
Debit Notes;
Billing Adjustments;
Revenue Adjustments;
Invoice Corrections;
Payment Reversals;
Chargebacks;
Disputes;
Dispute Evidence;
Dispute Resolution;
Refund Approval;
Adjustment Approval;
Refund Policies;
Cancellation Adjustments;
Tax Adjustments;
Accounting Impact;
Audit Trail.
4. Conceptos que deben mantenerse separados

Es fundamental no mezclar:

Refund
Credit
Adjustment
Dispute
Chargeback
Reversal
Write-Off
Cancellation

Cada uno tiene una finalidad distinta.

5. Refund

Un Refund devuelve dinero al Customer.

Ejemplo:

Payment
$1,000


Refund
$300

Resultado:

Customer recibió de vuelta
$300
6. Full Refund
Original Payment
$1,000


Refund
$1,000

Resultado:

Refunded
100%

Debe impedirse accidentalmente:

Refund
$1,500

si solo existen $1,000 disponibles para devolver, salvo que exista un flujo explícito para otro tipo de saldo.

7. Partial Refund
Payment
$1,000


Partial Refund
$250


Remaining Refundable
$750

El sistema debe mantener:

Original Amount
Refunded Amount
Remaining Refundable Amount
8. Refund Lifecycle
Refund Requested
       ↓
Validation
       ↓
Approval
       ↓
Refund Processing
       ↓
Payment Provider
       ↓
Refunded
       ↓
Reconciled

También:

REJECTED
FAILED
CANCELED
9. Refund Status
REQUESTED
PENDING_APPROVAL
APPROVED
PROCESSING
SUBMITTED
SUCCEEDED
FAILED
REJECTED
CANCELED
RECONCILED
10. Refund Request
RefundRequest
│
├── id
├── customer_id
├── payment_id
├── invoice_id
├── amount
├── currency
├── reason
├── requested_by
├── requested_at
├── status
└── metadata
11. Refund Reasons

Ejemplos:

CUSTOMER_REQUEST
DUPLICATE_PAYMENT
OVERCHARGE
SERVICE_CANCELLATION
SERVICE_FAILURE
SLA_CREDIT
BILLING_ERROR
FRAUD
CHARGEBACK
OTHER

Los motivos deben ser configurables.

12. Refund Policy
RefundPolicy
│
├── eligibility_rules
├── maximum_amount
├── approval_threshold
├── allowed_reasons
├── refund_window
├── payment_method_rules
└── status

Ejemplo:

Refund < $100
→ Auto approval


Refund $100–$1,000
→ Manager approval


Refund > $1,000
→ Finance approval
13. Refund Approval

Debe soportar:

Requester
   ↓
Reviewer
   ↓
Approver
   ↓
Payment Processor

Esto evita que cualquier usuario pueda devolver grandes cantidades sin control.

14. Separation of Duties

Ejemplo:

Customer Support
→ Request Refund


Finance
→ Approve Refund


Billing
→ Execute Refund


Accounting
→ Reconcile
15. Refundable Amount

El sistema debe calcular:

Refundable Amount
=
Paid Amount
-
Previously Refunded
-
Restricted Amount

Ejemplo:

Paid
$1,000


Previous Refund
$200


Refundable
$800
16. Refund to Original Payment Method

La opción preferida puede ser:

Payment
   ↓
Original Payment Method
   ↓
Refund

Por ejemplo:

Card
→ Card Refund


Bank Transfer
→ Bank Refund

La disponibilidad depende del payment provider.

17. Refund Alternative

En algunos casos puede utilizarse:

Refund
Credit

Ejemplo:

Customer paid
$1,000


Instead of refund:
Credit
$1,000

Pero:

Credit
≠
Refund

El Customer no recibe necesariamente Cash.

18. Credit Balance

Un Customer puede tener:

Credit Balance
$500

y aplicarlo:

Invoice
$1,000


Credit
-$500


Amount Due
$500
19. Credit Lifecycle
Credit Created
      ↓
Available
      ↓
Applied
      ↓
Consumed

También:

EXPIRED
CANCELED
REVERSED

si las políticas correspondientes lo permiten.

20. Credit Note

Una Credit Note corrige o reduce una factura.

Ejemplo:

Invoice
$10,000


Credit Note
-$2,000


Adjusted Invoice Balance
$8,000

Esto debe estar relacionado con la invoice original.

21. Debit Note

Una Debit Note incrementa una obligación.

Invoice
$10,000


Debit Note
+$500


Adjusted Balance
$10,500
22. Adjustment

Un Adjustment es una corrección controlada de Billing.

Ejemplo:

Expected Charge
$1,000


Actual
$900


Adjustment
-$100

Debe existir:

Reason
Source
Approval
Actor
Timestamp
23. Adjustment Types
PRICE_CORRECTION
QUANTITY_CORRECTION
TAX_CORRECTION
USAGE_CORRECTION
DUPLICATE_CHARGE
MISSING_CHARGE
CONTRACT_CORRECTION
BILLING_ERROR
MANUAL_ADJUSTMENT
24. Adjustment Scope

Un ajuste puede aplicarse a:

Charge
Invoice Line
Invoice
Subscription
Billing Account
Customer
Contract
Revenue Schedule

Esto permite granularidad.

25. Invoice Adjustment

Ejemplo:

Invoice
$5,000


Adjustment
-$500


New Balance
$4,500

No se modifica:

Invoice.total = 4500

destructivamente.

Debe existir:

Original
$5,000


Adjustment
-$500


Effective
$4,500
26. Immutable Billing History

La arquitectura correcta:

Original Invoice
      +
Adjustment
      +
Credit Note
      +
Payment
      +
Refund

en lugar de:

UPDATE invoice
SET total = ...

Esto es fundamental para auditoría.

27. Adjustment Ledger
Adjustment Ledger
│
├── Original Transaction
├── Adjustment
├── Reason
├── Approval
└── Result

Ejemplo:

INV-1000
$10,000


ADJ-001
-$1,000


Effective
$9,000
28. Dispute

Un Dispute ocurre cuando el Customer cuestiona una obligación o transacción.

Ejemplos:

"I did not authorize this payment."
"The amount is incorrect."
"I never received the service."
"I was charged twice."
29. Dispute vs Refund

No son iguales.

Refund

EVOXA decide devolver dinero.

Dispute

El Customer cuestiona una transacción.

Dispute
   ↓
Investigation
   ↓
Accept / Reject / Partial Resolution
30. Dispute Lifecycle
DISPUTE_OPENED
       ↓
UNDER_REVIEW
       ↓
EVIDENCE_COLLECTION
       ↓
DECISION
       ↓
RESOLVED

Posibles resultados:

CUSTOMER_WON
MERCHANT_WON
PARTIAL_WIN
WITHDRAWN
31. Dispute Model
Dispute
│
├── id
├── customer_id
├── billing_account_id
├── invoice_id
├── payment_id
├── amount
├── currency
├── reason_code
├── description
├── status
├── priority
├── assigned_to
└── created_at
32. Dispute Reasons
UNAUTHORIZED_PAYMENT
DUPLICATE_CHARGE
INCORRECT_AMOUNT
SERVICE_NOT_RECEIVED
SERVICE_NOT_AS_DESCRIBED
CANCELED_SERVICE
REFUND_NOT_RECEIVED
OTHER
33. Dispute Priority
LOW
MEDIUM
HIGH
CRITICAL

Puede depender de:

Amount
Customer Value
Deadline
Risk
Payment Provider
Legal Requirement
34. Dispute Evidence

Para un dispute de payment:

Evidence
│
├── Invoice
├── Payment Confirmation
├── Contract
├── Terms
├── Delivery Record
├── Usage Logs
├── Customer Communication
└── Refund History
35. Evidence Model
DisputeEvidence
│
├── id
├── dispute_id
├── type
├── file_reference
├── description
├── submitted_at
├── submitted_by
└── status
36. Evidence Package

El sistema puede construir:

Dispute
   ↓
Evidence Collection
   ↓
Evidence Package
   ↓
Payment Provider

Debe conservar:

Submission Date
Provider Reference
Status
Response
37. Chargeback

Un Chargeback normalmente proviene del payment ecosystem/banco y no de una simple solicitud interna del Customer.

Flujo:

Payment
   ↓
Bank / Card Network
   ↓
Chargeback
   ↓
EVOXA
   ↓
Evidence
   ↓
Representment
   ↓
Decision
38. Chargeback vs Dispute

En la plataforma:

Customer Dispute
=
Customer-level complaint


Chargeback
=
Payment-system financial dispute

Pueden estar relacionados:

Customer Dispute
       ↓
Chargeback

pero no son necesariamente la misma entidad.

39. Chargeback Model
Chargeback
│
├── id
├── payment_id
├── dispute_id
├── provider
├── provider_case_id
├── amount
├── fee
├── reason_code
├── status
├── received_at
├── response_deadline
└── resolved_at
40. Chargeback Lifecycle
RECEIVED
   ↓
UNDER_REVIEW
   ↓
EVIDENCE_REQUIRED
   ↓
SUBMITTED
   ↓
PENDING_DECISION
   ↓
WON / LOST
41. Chargeback Financial Impact

Payment:

$1,000

Chargeback:

-$1,000

Fee:

-$25

Total impact:

-$1,025

Debe reflejarse correctamente en Payments y Accounting.

42. Chargeback Reserve

Dependiendo del payment provider, puede existir:

Chargeback Reserve

EVOXA puede registrar la información necesaria para conciliación.

43. Dispute Hold

Una transacción disputada puede generar:

Dispute
   ↓
Collection Hold

para evitar cobrar agresivamente un importe que está siendo investigado.

44. Dunning Integration

La integración:

Past Due
   ↓
Dunning

pero:

Dispute Open
   ↓
Dunning Pause

según la política.

45. Dispute Resolution

Posibles resoluciones:

FULL_REFUND
PARTIAL_REFUND
CREDIT
NO_ADJUSTMENT
CHARGEBACK_ACCEPTED
CHARGEBACK_WON
INVOICE_CORRECTED
PAYMENT_REVERSED
46. Full Resolution Example
Invoice
$1,000


Customer disputes
$1,000


Investigation
Customer correct


Resolution
Full Refund
$1,000
47. Partial Resolution
Invoice
$1,000


Disputed
$400


Resolution
$250 Refund


Remaining
$750
48. Dispute Rejection
Dispute
$1,000


Evidence
Valid


Decision
Merchant Won

Entonces:

Refund
$0

y la deuda/payment sigue según corresponda.

49. Dispute Withdrawal

El Customer puede retirar:

Dispute
   ↓
Withdrawn

El sistema conserva el historial.

50. Refund and Revenue

Un refund no significa necesariamente:

Revenue = 0

El impacto depende de:

período;
naturaleza del refund;
Revenue Recognition;
contrato;
accounting policy.

Puede requerir:

Revenue Adjustment
Revenue Reversal
Contract Liability Adjustment
51. Refund and Accounting

Ejemplo:

Payment:

$1,190

Refund:

$238

Accounting debe registrar el efecto correspondiente sobre:

Cash
AR
Revenue
Tax

según el origen del refund.

52. Refund and Tax

Si:

Net
$1,000


Tax
$190


Total
$1,190

y se devuelve todo:

Refund
$1,190

el sistema debe determinar también el tratamiento de:

Tax
$190

No se debe asumir que todos los refunds fiscales funcionan igual en todas las jurisdicciones.

53. Partial Tax Refund

Ejemplo:

Original
$1,190


Partial Refund
$238

Puede corresponder:

Net
$200


Tax
$38

El motor fiscal debe calcular el efecto aplicable.

54. Adjustment and Revenue

Ejemplo:

Revenue
$10,000


Billing Correction
-$1,000

Revenue Management debe recibir:

Revenue Adjustment
-$1,000

y Accounting:

Journal Entry

correspondiente.

55. Adjustment and AR
Original AR
$10,000


Credit
-$1,000


Adjusted AR
$9,000

El AR Ledger debe mostrar el origen.

56. Adjustment and Subscription

Una subscription puede modificarse:

$1,000/month

a:

$800/month

El sistema debe decidir:

Current Invoice
Future Invoice
Revenue Schedule
Contract Modification

según el caso.

57. Cancellation Adjustment

Customer cancela:

Annual Subscription
$12,000

Después de:

4 months recognized
$4,000

Queda:

$8,000

El sistema debe determinar qué sucede con:

Deferred Revenue
Refund
Credit
Revenue

según contrato y política.

58. Proration

Cuando cambia una subscription:

Old Plan
$1,000/month


New Plan
$1,500/month

a mitad del período:

Old Plan Credit
+
New Plan Charge

puede generar:

Proration Adjustment
59. Proration Engine
Subscription Change
       ↓
Effective Date
       ↓
Unused Period
       ↓
Credit
       +
New Period
       ↓
Charge
       ↓
Net Adjustment
60. Upgrade Example
Old Plan
$100


New Plan
$200


15 days remaining

El sistema calcula:

Unused Old Plan
→ Credit


Remaining New Plan
→ Charge

Resultado:

Net Proration
61. Downgrade Example
Old Plan
$200


New Plan
$100

Puede generar:

Credit

pero el tratamiento puede depender de:

billing cycle;
contract;
refund policy;
plan rules.
62. Duplicate Charge

Si el Customer fue cobrado dos veces:

Payment A
$1,000


Payment B
$1,000

el sistema puede detectar:

Duplicate Payment

y generar:

Refund
$1,000

o:

Credit
$1,000

según la resolución.

63. Billing Error

Ejemplo:

Expected
$500


Billed
$700

Corrección:

Adjustment
-$200

Debe quedar:

Original
$700


Adjustment
-$200


Effective
$500
64. Missing Charge

Caso inverso:

Expected
$700


Billed
$500

Puede generarse:

Debit Adjustment
+$200

o una nueva factura/debit note, dependiendo de la política.

65. Adjustment Approval

Los ajustes deben poder tener límites:

< $100
→ Automatic


$100–$1,000
→ Manager


> $1,000
→ Finance

Esto protege contra errores y fraude interno.

66. Refund Fraud Controls

El sistema debe detectar:

Multiple Refunds
Refund > Payment
Refund Shortly After Payment
Unusual Refund Rate
Refund to New Payment Method
Repeated Refund Requests
67. Refund Risk Score
Customer
   │
   ├── Refund History
   ├── Payment History
   ├── Disputes
   ├── Chargebacks
   └── Amount
          │
          ▼
     Refund Risk

Resultado:

LOW
MEDIUM
HIGH
68. Refund Velocity

Puede monitorizar:

Refunds
Last 30 Days
$100K

comparado con:

Payments
Last 30 Days
$500K

Refund Rate:

20%

Un valor inusual puede generar una alerta.

69. Dispute Analytics

KPIs:

Dispute Rate
Chargeback Rate
Win Rate
Loss Rate
Average Resolution Time
Refund Rate
Recovery Rate
70. Chargeback Rate

Conceptualmente:

Chargebacks
÷
Transactions

o una metodología basada en volumen/valor según el proveedor.

EVOXA debe permitir configurar la métrica utilizada.

71. Chargeback Win Rate
Chargebacks Won
÷
Chargebacks Contested

Ejemplo:

Won
80


Contested
100


Win Rate
80%
72. Dispute Resolution Time
Opened
01/08


Resolved
10/08


Resolution Time
9 days

Debe medirse por:

Average
Median
P90
P95
73. Refund Analytics
Refund Volume
Refund Rate
Average Refund
Refund by Reason
Refund by Product
Refund by Customer
Refund by Agent
74. Refund by Reason
Billing Error       35%
Cancellation        25%
Customer Request    20%
Duplicate            10%
SLA                  10%

Esto puede revelar problemas operativos.

75. Root Cause Analysis

EVOXA debe relacionar:

Refund
   ↓
Reason
   ↓
Product
   ↓
Billing Rule
   ↓
Root Cause

Ejemplo:

High Refund Rate
       ↓
Product X
       ↓
Usage Calculation
       ↓
Rating Error

Esto conecta directamente con:

26/12 — Billing Rating & Charge Calculation.

76. Dispute Root Cause
Disputes
│
├── Duplicate Charge
├── Incorrect Usage
├── Wrong Price
├── Service Issue
├── Unauthorized Payment
└── Tax Issue

Esto permite mejorar todo el Billing Platform.

77. Refund Workflow
Customer Request
       ↓
Validate
       ↓
Check Refundable Amount
       ↓
Check Policy
       ↓
Approval
       ↓
Execute
       ↓
Payment Provider
       ↓
Accounting
       ↓
Reconciliation
       ↓
Customer Notification
78. Adjustment Workflow
Adjustment Requested
       ↓
Validate Source
       ↓
Calculate Impact
       ↓
Approval
       ↓
Create Adjustment
       ↓
Update AR
       ↓
Revenue / Tax Impact
       ↓
Accounting
       ↓
Audit
79. Dispute Workflow
Dispute Created
       ↓
Validate
       ↓
Assign
       ↓
Investigate
       ↓
Collect Evidence
       ↓
Decision
       ↓
Refund / Credit / Reject
       ↓
Accounting
       ↓
Close
80. Chargeback Workflow
Chargeback Received
       ↓
Match Payment
       ↓
Create Case
       ↓
Evaluate
       ↓
Collect Evidence
       ↓
Submit
       ↓
Provider Decision
       ↓
Won / Lost
       ↓
Accounting
       ↓
Reconcile
81. Refund Ledger

Debe existir trazabilidad:

Payment
$1,000


Refund
-$200


Refund
-$100


Remaining Refundable
$700

Nunca simplemente:

payment.refunded = true

porque puede haber múltiples refunds parciales.

82. Dispute Ledger
Original Payment
$1,000


Chargeback
-$1,000


Representment
+$1,000


Final Decision

Esto permite reconstruir la historia completa.

83. Adjustment Ledger
Original
$10,000


Adjustment A
-$500


Adjustment B
+$200


Effective
$9,700
84. Financial Event Chain
Original Transaction
       │
       ├── Refund
       ├── Adjustment
       ├── Credit
       ├── Dispute
       ├── Chargeback
       └── Reversal

Todos deben estar relacionados mediante:

source_id
source_type
parent_transaction_id
85. Parent-Child Transactions

Ejemplo:

INV-1000
   │
   ├── CR-100
   │
   ├── REF-100
   │
   └── ADJ-100

Esto permite navegación completa.

86. Transaction Relationship Model
BillingTransaction
│
├── id
├── type
├── parent_transaction_id
├── source_type
├── source_id
├── amount
├── currency
├── effective_date
├── status
└── metadata
87. Refund Model
Refund
│
├── id
├── payment_id
├── invoice_id
├── customer_id
├── amount
├── currency
├── reason
├── status
├── provider
├── provider_reference
├── requested_by
├── approved_by
├── processed_at
└── metadata
88. Adjustment Model
BillingAdjustment
│
├── id
├── source_type
├── source_id
├── adjustment_type
├── amount
├── currency
├── reason
├── requested_by
├── approved_by
├── status
├── effective_date
└── metadata
89. Dispute Model
BillingDispute
│
├── id
├── customer_id
├── invoice_id
├── payment_id
├── amount
├── currency
├── type
├── reason_code
├── status
├── priority
├── assigned_to
├── opened_at
├── resolved_at
└── metadata
90. Dispute Evidence Model
DisputeEvidence
│
├── id
├── dispute_id
├── evidence_type
├── document_reference
├── submitted_by
├── submitted_at
├── status
└── metadata
91. Chargeback Model
Chargeback
│
├── id
├── payment_id
├── dispute_id
├── provider
├── provider_case_id
├── reason_code
├── amount
├── fee
├── status
├── received_at
├── deadline
└── resolved_at
92. Approval Model

Puede existir una entidad genérica:

ApprovalRequest
│
├── id
├── entity_type
├── entity_id
├── requested_by
├── approver
├── threshold
├── status
├── reason
└── decided_at

Esto permite reutilizar aprobación para:

Refund
Adjustment
Write-Off
Settlement
93. API — Refunds
POST /billing/refunds
GET  /billing/refunds
GET  /billing/refunds/{id}
POST /billing/refunds/{id}/approve
POST /billing/refunds/{id}/process
POST /billing/refunds/{id}/cancel
94. API — Adjustments
POST /billing/adjustments
GET  /billing/adjustments
GET  /billing/adjustments/{id}
POST /billing/adjustments/{id}/approve
POST /billing/adjustments/{id}/apply
POST /billing/adjustments/{id}/reverse
95. API — Disputes
POST /billing/disputes
GET  /billing/disputes
GET  /billing/disputes/{id}
POST /billing/disputes/{id}/assign
POST /billing/disputes/{id}/evidence
POST /billing/disputes/{id}/resolve
POST /billing/disputes/{id}/close
96. API — Chargebacks
POST /billing/chargebacks
GET  /billing/chargebacks
GET  /billing/chargebacks/{id}
POST /billing/chargebacks/{id}/submit
POST /billing/chargebacks/{id}/respond
POST /billing/chargebacks/{id}/resolve
97. API — Credit Notes
POST /billing/credit-notes
GET  /billing/credit-notes
GET  /billing/credit-notes/{id}
POST /billing/credit-notes/{id}/apply
POST /billing/credit-notes/{id}/reverse
98. API — Debit Notes
POST /billing/debit-notes
GET  /billing/debit-notes
GET  /billing/debit-notes/{id}
POST /billing/debit-notes/{id}/apply
99. Event Architecture

Eventos principales:

RefundRequested
RefundApproved
RefundProcessed
RefundSucceeded
RefundFailed


AdjustmentRequested
AdjustmentApproved
AdjustmentApplied
AdjustmentReversed


DisputeOpened
DisputeAssigned
EvidenceSubmitted
DisputeResolved
DisputeClosed


ChargebackReceived
ChargebackSubmitted
ChargebackWon
ChargebackLost
100. Integration Architecture
                         BILLING
                            │
                ┌───────────┼───────────┐
                ▼           ▼           ▼
             Refund     Adjustment   Dispute
                │           │           │
                └───────────┼───────────┘
                            ▼
                       Event Bus
                            │
           ┌────────────────┼────────────────┐
           ▼                ▼                ▼
        Payments         Revenue         Accounting
           │                │                │
           ▼                ▼                ▼
        Provider       Recognition           GL
101. Idempotency

Refunds y chargebacks son especialmente sensibles.

Si llega dos veces:

RefundRequested
REF-1000

EVOXA no debe generar dos refunds.

Debe utilizar:

Idempotency Key
Provider Reference
Source Transaction
102. Provider Reconciliation

Ejemplo:

EVOXA
Refunds
$100,000


Payment Provider
Refunds
$99,500

Debe aparecer:

Refund Reconciliation Variance
$500
103. Refund Reconciliation
Refund Created
   ↓
Refund Submitted
   ↓
Provider
   ↓
Provider Confirmation
   ↓
EVOXA Reconciliation

Solo después:

RECONCILED
104. Adjustment Reconciliation
Billing Adjustment
-$1,000


AR Adjustment
-$1,000


Revenue Adjustment
-$1,000


Accounting
-$1,000

Debe existir trazabilidad entre todas las capas.

105. Dispute Reconciliation
Payment Provider
Chargeback
$1,000


EVOXA
Chargeback
$1,000


Accounting
Chargeback
$1,000

Si existen diferencias:

Reconciliation Exception
106. Customer Portal

El Customer puede ver:

Billing
│
├── Invoices
├── Payments
├── Refunds
├── Credits
├── Disputes
└── Adjustments

Debe mostrar solo información que el Customer tenga derecho a consultar.

107. Refund Portal

Opcionalmente:

Invoice
   ↓
Request Refund

pero:

Request
≠
Approval

El sistema debe aplicar la política correspondiente.

108. Dispute Portal

Customer puede:

Open Dispute
Upload Evidence
Describe Issue
Track Status
Respond to Questions

Esto reduce carga operacional.

109. Dispute SLA

Cada dispute puede tener:

Response Deadline
Evidence Deadline
Resolution Target

Ejemplo:

Opened
01/08


Evidence Deadline
05/08


Resolution Target
15/08
110. SLA Monitoring

Dashboard:

Open Disputes
120


Due Soon
15


Overdue
4

Esto permite evitar pérdida de derechos de representación frente a proveedores.

111. Refund SLA

También:

Refund Requested
→ Approval SLA
→ Processing SLA
→ Provider SLA

Ejemplo:

Requested
01/08


Approved
02/08


Submitted
02/08


Provider Completed
04/08
112. Customer Experience

El Customer debe poder entender:

What happened?
Why?
How much?
When?
What happens next?

Ejemplo:

Refund
$250


Status
Processing


Expected Completion
03/09
113. Financial Controls

Para cada operación:

Source Valid
Amount Valid
Currency Valid
Approval Valid
Period Valid
Tax Impact Evaluated
Revenue Impact Evaluated
Accounting Impact Generated
114. Security Controls

Especialmente:

Refund Fraud
Unauthorized Adjustment
Unauthorized Credit
Dispute Manipulation
Duplicate Refund

Controles:

RBAC
Approval
MFA for sensitive operations
Audit
Rate Limits
Idempotency
115. Audit Trail

Ejemplo:

Refund
REF-1001


Requested by
User A


Approved by
User B


Processed by
System


Provider
Stripe/Bank/etc.


Provider Reference
XYZ123


Accounting Entry
JE-5001

Todo debe estar relacionado.

116. Financial Impact Matrix

Cada operación debe determinar su impacto.

Operación	AR	Cash	Revenue	Tax	Accounting
Refund	Puede afectar	↓	Puede afectar	Puede afectar	Sí
Credit	↓	—	Puede afectar	Puede afectar	Sí
Debit	↑	—	Puede afectar	Puede afectar	Sí
Adjustment	↑/↓	—	Puede afectar	Puede afectar	Sí
Chargeback	↓	↓	Puede afectar	Puede afectar	Sí
Dispute	Depende	Depende	Depende	Depende	Puede requerir
Write-Off	↓	—	—/depende	—/depende	Sí

La columna Revenue debe depender de la naturaleza de la operación y de la política contable.

117. Refund vs Credit vs Adjustment

Una regla práctica:

REFUND
→ Devolver Cash


CREDIT
→ Dar saldo al Customer


ADJUSTMENT
→ Corregir una transacción


CREDIT NOTE
→ Documento formal que reduce una obligación


DISPUTE
→ Cuestionar una transacción


CHARGEBACK
→ Disputa financiera iniciada a través del payment ecosystem


REVERSAL
→ Revertir una transacción previamente registrada
118. Full Transaction Example

Original:

Subscription
$1,000


Tax
$190


Invoice
$1,190


Payment
$1,190

Customer requests partial refund:

Refund
$238

Billing:

Net Refund
$200


Tax Refund
$38

Después:

Original Paid
$1,190


Refunded
$238


Net Cash Retained
$952

Accounting y Revenue reciben los eventos correspondientes.

119. Dispute Example
Payment
$1,000

Customer afirma:

Unauthorized

EVOXA:

Dispute
Opened

Luego:

Evidence
Contract
Payment Record
Usage Logs
Communication

Resultado:

Merchant Won

Entonces el sistema:

Dispute
→ Resolved


Payment
→ Remains valid


Collection
→ Continues if applicable
120. Chargeback Example
Payment
$1,000


Chargeback
$1,000


Provider Fee
$25

EVOXA:

Chargeback Case
   ↓
Evidence
   ↓
Representment

Resultado:

WON

El provider devuelve el importe según sus reglas.

O:

LOST

El impacto financiero permanece.

121. End-to-End Correction
Invoice
$10,000
   ↓
Billing Error Detected
   ↓
Adjustment
-$1,000
   ↓
Credit Note
-$1,000
   ↓
AR
$9,000
   ↓
Revenue Adjustment
-$1,000
   ↓
Accounting Entry
   ↓
Audit Trail
122. Root Cause Architecture

Toda corrección debería intentar responder:

Why?

Por ejemplo:

Refund
   ↓
Reason
BILLING_ERROR
   ↓
Root Cause
Rating Rule
   ↓
Rule Version
v3.2

Esto permite corregir el problema original y no solo sus consecuencias.

123. Adjustment Analytics

EVOXA puede medir:

Adjustment Volume
Adjustment Rate
Adjustment by Reason
Adjustment by Product
Adjustment by Customer
Adjustment by Billing Rule
124. Refund Analytics
Refund Amount
Refund Rate
Refund Count
Refund Reason
Refund Processing Time
Refund Failure Rate
Refund by Product
Refund by Customer
125. Dispute Analytics
Dispute Count
Dispute Amount
Dispute Rate
Win Rate
Loss Rate
Average Resolution Time
Dispute by Reason
Dispute by Product
126. Chargeback Analytics
Chargeback Count
Chargeback Amount
Chargeback Rate
Win Rate
Provider
Reason Code
Fees
Recovery
127. AI-Assisted Refund Detection

La IA puede detectar:

Customer
10 purchases


Refund requests
8

y generar:

Refund Risk
HIGH

La decisión final debe permanecer sujeta a políticas y controles humanos cuando corresponda.

128. AI-Assisted Dispute Classification

La IA puede clasificar:

Customer Message
        ↓
Classification
        ↓
DUPLICATE_CHARGE

o:

SERVICE_NOT_RECEIVED

Esto acelera el routing.

129. AI Evidence Assistant

La IA puede ayudar a recopilar:

Invoice
Payment
Contract
Usage
Delivery
Communication

y preparar un resumen para revisión.

Pero:

La evidencia enviada a un proveedor debe pasar por las políticas de autorización y compliance de EVOXA.

130. Core Principles
1. Never Destroy History

Nunca modificar destructivamente una transacción histórica.

2. Refund ≠ Credit

Cash refund y customer credit son diferentes.

3. Dispute ≠ Chargeback

Un dispute interno y un chargeback de payment network no son necesariamente iguales.

4. Every Adjustment Has a Reason

Todo ajuste necesita causa.

5. Approval Matters

Los montos sensibles deben requerir autorización.

6. Accounting Must Follow

Cada operación relevante debe producir su impacto contable correspondiente.

7. Tax Must Be Recalculated

Cuando corresponda, refunds y adjustments deben revisar el impacto fiscal.

8. Everything Is Traceable

Original → Correction → Financial Impact.

9. Idempotency Is Mandatory

Nunca procesar dos veces un refund o chargeback por error.

10. Compliance First

Las reglas de refunds, disputes y chargebacks deben poder variar por jurisdicción.

131. Resultado de 26/20

Billing Refunds, Adjustments & Disputes establece la capa de corrección y resolución financiera de EVOXA.

La arquitectura queda:

                     BILLING TRANSACTION
                              │
            ┌─────────────────┼──────────────────┐
            ▼                 ▼                  ▼
         REFUND           ADJUSTMENT          DISPUTE
            │                 │                  │
            ▼                 ▼                  ▼
         PAYMENT          CREDIT/DEBIT       EVIDENCE
         PROVIDER          NOTE               REVIEW
            │                 │                  │
            ▼                 ▼                  ▼
       RECONCILIATION      AR/Revenue       RESOLUTION
            │                 │                  │
            └─────────────────┼──────────────────┘
                              ▼
                          ACCOUNTING
                              │
                              ▼
                             GL

Y todo mantiene una relación con la transacción original:

Original Transaction
       │
       ├── Refund
       ├── Credit
       ├── Adjustment
       ├── Dispute
       ├── Chargeback
       └── Reversal
Arquitectura acumulada 26/08 → 26/20
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

Con esto, EVOXA ya tiene cubierto prácticamente todo el Billing Transaction Lifecycle:

Product
   ↓
Pricing
   ↓
Subscription
   ↓
Usage
   ↓
Rating
   ↓
Charge
   ↓
Invoice
   ↓
Tax
   ↓
Payment
   ↓
Revenue
   ↓
Accounting
   ↓
Collections
   ↓
Refunds / Adjustments / Disputes

Y lo más importante: cada operación conserva trazabilidad, versionamiento, auditoría, relaciones padre-hijo, impacto contable y capacidad de reconciliación.

Siguiente capítulo
26/21 — Billing Analytics & Revenue Intelligence

Aquí pasamos de procesar Billing a entender el negocio: dashboards financieros, Revenue Analytics, MRR, ARR, LTV, churn, expansion, contraction, cohortes, AR analytics, collection analytics, billing leakage, revenue leakage, forecasting, variance analysis, profitability, customer economics y una capa de Revenue Intelligence + AI que permita detectar oportunidades, anomalías y riesgos.
