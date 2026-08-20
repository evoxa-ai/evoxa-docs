1. Billing Dunning & Collections

Billing Dunning & Collections es la capa responsable de gestionar el dinero pendiente de cobro después de que EVOXA ha generado una obligación financiera.

Su objetivo es transformar:

Invoice
   ↓
Outstanding Balance
   ↓
Payment Due
   ↓
Payment Attempt
   ↓
Past Due
   ↓
Dunning
   ↓
Collection
   ↓
Recovery / Resolution

en un proceso controlado, automatizado, auditable y configurable.

La diferencia fundamental es:

Dunning intenta recuperar el pago mediante comunicaciones y acciones automáticas; Collections administra la recuperación de deuda cuando el atraso requiere una gestión más avanzada.

2. Posición dentro de Billing

Hasta ahora:

26/13  Billing Invoicing
26/14  Billing Payments
26/15  Billing Credits, Discounts & Adjustments
26/16  Billing Taxes & Compliance
26/17  Billing Revenue Management
26/18  Billing Accounting & Financial Integration

Ahora:

26/19  Billing Dunning & Collections

La relación:

Invoice
   ↓
Accounts Receivable
   ↓
Payment Due
   ↓
Payment Attempt
   ↓
Payment Failure / Past Due
   ↓
Dunning
   ↓
Collections
   ↓
Recovery
3. Propósito

La plataforma debe administrar:

cuentas por cobrar;
invoices vencidas;
payment failures;
retries;
dunning campaigns;
dunning stages;
collection workflows;
collection cases;
promises to pay;
payment plans;
customer communication;
account suspension;
service restrictions;
write-offs;
bad debt;
external collection agencies;
recovery;
collection analytics;
collection forecasting;
risk scoring;
compliance;
audit trail.
4. AR vs Dunning vs Collections

Son conceptos diferentes.

Accounts Receivable
¿Cuánto debe el Customer?
Dunning
¿Cómo recordamos y recuperamos el pago?
Collections
¿Cómo gestionamos una deuda que requiere intervención?

Ejemplo:

Invoice
$10,000


Due Date
01/08


02/08
Past Due
1 day


Dunning
Stage 1


30 days
Collection Case


90 days
Potential External Collection
5. Outstanding Balance

El sistema debe calcular:

Opening Balance
+
Invoices
+
Debit Adjustments
-
Payments
-
Credits
-
Refunds
-
Write-Offs
=
Outstanding Balance

Ejemplo:

Opening
$0


Invoice
+$10,000


Payment
-$6,000


Credit
-$500
────────────
Outstanding
$3,500
6. AR Aging

La deuda debe clasificarse por antigüedad:

Current
1–30
31–60
61–90
91–120
120+

Ejemplo:

Current       $500K
1–30          $250K
31–60         $150K
61–90          $80K
91–120         $40K
120+           $60K

Esto permite priorizar Collections.

7. Aging Buckets

EVOXA debe permitir configurar:

AgingBucket
│
├── code
├── name
├── min_days
├── max_days
├── severity
├── collection_stage
└── status

No todos los negocios necesitan los mismos períodos.

8. Due Date

Cada invoice debe tener:

Issue Date
Due Date
Payment Terms

Ejemplo:

Invoice Date
01/08/2026


Payment Terms
Net 30


Due Date
31/08/2026

Dunning no debería considerar la invoice vencida antes de la fecha correspondiente, salvo reglas específicas.

9. Payment Terms

Ejemplos:

DUE_ON_RECEIPT
NET_7
NET_15
NET_30
NET_45
NET_60
NET_90

También:

CUSTOM
MILESTONE
INSTALLMENT
10. Dunning

Dunning es el conjunto de acciones destinadas a obtener el pago:

Payment Due
   ↓
Reminder
   ↓
Reminder
   ↓
Warning
   ↓
Escalation
   ↓
Suspension

La estrategia debe ser configurable.

11. Dunning Policy
DunningPolicy
│
├── name
├── customer_segment
├── payment_method
├── currency
├── grace_period
├── stages
├── escalation_rules
├── suspension_rules
└── status
12. Dunning Stages

Ejemplo:

Stage 0
Payment Due


Stage 1
+1 day


Stage 2
+7 days


Stage 3
+15 days


Stage 4
+30 days


Stage 5
+60 days

Cada etapa puede tener acciones diferentes.

13. Dunning Stage Example
+1 day
→ Email reminder


+7 days
→ Email + SMS


+15 days
→ Account warning


+30 days
→ Service restriction


+60 days
→ Collection case

Las acciones deben respetar contrato, legislación y políticas del negocio.

14. Grace Period

Puede existir:

Due Date
31/08


Grace Period
5 days


Dunning Starts
05/09

Esto evita generar comunicaciones inmediatamente después del vencimiento.

15. Dunning Rules

Ejemplo conceptual:

IF
invoice.days_past_due >= 7


AND
outstanding_balance > $100


THEN
execute Dunning Stage 2
16. Dunning Rule Engine
Invoice
   ↓
Days Past Due
   ↓
Outstanding Balance
   ↓
Customer Segment
   ↓
Risk
   ↓
Dunning Policy
   ↓
Dunning Stage
   ↓
Actions
17. Customer Segmentation

No todos los Customers deben recibir el mismo tratamiento.

Segmentos:

SMB
MID_MARKET
ENTERPRISE
STRATEGIC
GOVERNMENT

También:

LOW_RISK
MEDIUM_RISK
HIGH_RISK
18. Enterprise Customer Rules

Un Customer Enterprise puede tener:

Payment Terms
Net 60


Grace Period
10 days


Manual Collection
Required

mientras que un Customer self-service puede tener:

Net 0
Automatic Retry
Automatic Suspension
19. Collection Priority

La plataforma puede calcular:

Collection Priority

basándose en:

Amount
Days Past Due
Customer Value
Risk
Payment History
Contract Status
Strategic Importance
20. Collection Priority Example
Customer A
Debt $500
120 days


Priority
MEDIUM

vs:

Customer B
Debt $100,000
60 days


Priority
CRITICAL
21. Collection Score

Conceptualmente:

Collection Score
=
Debt Amount
+
Aging
+
Risk
+
Customer Value
+
Probability of Recovery

No tiene que ser una suma literal; puede ser un modelo configurable.

22. Payment Failure

Uno de los principales triggers:

Payment Attempt
       ↓
FAILED

Causas:

INSUFFICIENT_FUNDS
CARD_EXPIRED
CARD_DECLINED
BANK_DECLINED
NETWORK_ERROR
INVALID_PAYMENT_METHOD
FRAUD_REVIEW
23. Soft vs Hard Decline

El sistema puede clasificar:

Soft Failure

Puede recuperarse:

Insufficient Funds
Temporary Bank Error
Network Error
Hard Failure

Probablemente requiere acción:

Expired Card
Invalid Account
Closed Account
Payment Method Invalid

Esto afecta la estrategia de retry.

24. Payment Retry
Payment Failed
   ↓
Retry #1
   ↓
Retry #2
   ↓
Retry #3
   ↓
Dunning

La política debe evitar reintentos excesivos o ineficientes.

25. Retry Policy
RetryPolicy
│
├── max_attempts
├── retry_interval
├── backoff_strategy
├── eligible_failure_codes
├── payment_method_rules
└── status
26. Exponential Backoff

Ejemplo:

Attempt 1
Day 0


Attempt 2
Day 2


Attempt 3
Day 5


Attempt 4
Day 10

La estrategia debe ser configurable.

27. Smart Retry

EVOXA puede evolucionar hacia:

Payment Failure
       ↓
Failure Analysis
       ↓
Optimal Retry Time
       ↓
Payment Retry

Ejemplo:

Temporary Bank Failure
→ Retry later

mientras:

Expired Card
→ Request payment method update
28. Payment Method Recovery

Cuando falla una tarjeta:

Payment Failed
   ↓
Customer Notification
   ↓
Update Payment Method
   ↓
Retry Payment

Esto debe integrarse con:

26/14 — Billing Payments

29. Customer Payment Portal

El Customer puede acceder a:

Outstanding Balance
Invoices
Payment Methods
Payment History
Due Dates
Payment Plan

y ejecutar:

Pay Now
Update Payment Method
Download Invoice
Request Payment Plan
30. Dunning Communication

Canales:

Email
SMS
Push
In-App
WhatsApp
Phone
Postal

La disponibilidad depende del mercado y configuración.

31. Communication Templates
DunningTemplate
│
├── channel
├── language
├── subject
├── body
├── variables
├── stage
└── status

Variables:

{{customer_name}}
{{invoice_number}}
{{amount_due}}
{{due_date}}
{{payment_link}}
32. Communication Localization

EVOXA debe soportar:

Spanish
English
Portuguese
French

y posteriormente otros idiomas.

Cada Customer puede tener:

Preferred Language
Preferred Channel
Timezone
33. Communication Frequency

Debe existir un control:

Max Emails
Max SMS
Max Notifications

para evitar:

Over-communication
34. Communication Consent

El sistema debe respetar:

Marketing Consent
Transactional Communication
Collections Communication

No deben tratarse todos como marketing.

Las comunicaciones relacionadas con obligaciones de pago pueden tener reglas diferentes, sujetas a la legislación aplicable.

35. Dunning Event

Eventos:

DunningStarted
DunningStageEntered
DunningReminderSent
DunningEscalated
PaymentRecovered
DunningPaused
DunningClosed
36. Dunning Case

Debe existir una entidad:

DunningCase
│
├── id
├── customer_id
├── billing_account_id
├── outstanding_amount
├── oldest_due_date
├── aging_days
├── current_stage
├── priority
├── status
└── assigned_to
37. Dunning Case Lifecycle
OPEN
 ↓
ACTIVE
 ↓
ESCALATED
 ↓
COLLECTION
 ↓
RESOLVED

También:

PAUSED
DISPUTED
WRITTEN_OFF
CLOSED
38. Dunning Case Aggregation

Un Customer puede tener:

Invoice A
$1,000


Invoice B
$2,000


Invoice C
$500

En lugar de crear tres campañas independientes:

Customer Debt
$3,500

puede crearse una gestión consolidada.

39. Invoice-Level Dunning

Pero también puede ser necesario:

Invoice A
Overdue


Invoice B
Current

Por lo que EVOXA debe soportar:

Invoice-level
+
Account-level
+
Customer-level
40. Collection Case

Cuando Dunning ya no es suficiente:

Dunning
   ↓
Collection Case

Modelo:

CollectionCase
│
├── id
├── customer_id
├── account_id
├── amount
├── aging
├── priority
├── assigned_agent
├── status
├── resolution
└── external_agency
41. Collection Workflow
Debt Identified
      ↓
Case Created
      ↓
Assigned
      ↓
Contact Customer
      ↓
Promise / Payment Plan
      ↓
Follow-Up
      ↓
Recovered / Escalated
42. Collection Agent

Un usuario interno puede ser asignado:

Collection Agent

con:

Portfolio
Customers
Cases
Tasks
Promises
43. Collection Queue
Collections Queue
│
├── Critical
├── High
├── Medium
└── Low

Ordenada por:

Amount
Aging
Risk
Priority
44. Collection Task
CollectionTask
│
├── id
├── case_id
├── type
├── assigned_to
├── due_date
├── status
├── outcome
└── notes

Ejemplos:

Call Customer
Send Reminder
Verify Payment
Escalate
Request Approval
45. Collection Notes

Los agentes deben poder registrar:

Contact Date
Channel
Person Contacted
Outcome
Notes
Next Action

Debe existir auditabilidad.

46. Promise to Pay

El Customer puede comprometerse:

Promise to Pay
$10,000


Date
15/09/2026

Modelo:

PromiseToPay
│
├── amount
├── promised_date
├── customer_id
├── case_id
├── status
└── fulfilled_at
47. Promise Lifecycle
CREATED
   ↓
CONFIRMED
   ↓
DUE
   ↓
FULFILLED

o:

BROKEN

si no se cumple.

48. Broken Promise

Si:

Promise
$10,000

pero:

Payment
$5,000

entonces:

Promise
BROKEN

Esto puede aumentar el Collection Risk.

49. Payment Plans

Para deudas importantes:

Debt
$12,000

puede establecerse:

Payment Plan
│
├── $3,000
├── $3,000
├── $3,000
└── $3,000
50. Payment Plan Model
PaymentPlan
│
├── id
├── customer_id
├── case_id
├── total_amount
├── installments
├── frequency
├── start_date
├── end_date
├── status
└── approved_by
51. Payment Plan Controls

Debe controlar:

Maximum Duration
Minimum Installment
Interest
Fees
Down Payment
Eligibility
Approval

Las reglas deben depender de las políticas comerciales y legales aplicables.

52. Collections Settlement

En algunos casos puede negociarse:

Debt
$10,000


Settlement
$8,000

La diferencia:

$2,000

requiere tratamiento financiero y autorización.

Debe registrarse:

Original Debt
Settlement Amount
Forgiven Amount
Approval
Reason
53. Debt Forgiveness

No debe confundirse:

Credit
Refund
Write-Off
Settlement
Forgiveness

Cada uno debe tener un motivo y tratamiento específico.

54. Write-Off

Cuando una deuda se considera incobrable:

Outstanding
$10,000


Write-Off
$10,000

Esto debe integrarse con:

26/18 — Accounting & Financial Integration

para generar el tratamiento contable correspondiente.

55. Bad Debt Lifecycle
Past Due
   ↓
Collection
   ↓
High Risk
   ↓
Uncollectible
   ↓
Write-Off

Pero:

Write-Off
≠
Debt Erasure

Debe conservarse el historial.

56. External Collection Agency

EVOXA puede integrarse con:

External Collection Agency

flujo:

Collection Case
   ↓
Approved for External Collection
   ↓
Agency
   ↓
Recovery
   ↓
Settlement
57. Agency Assignment
CollectionAssignment
│
├── case_id
├── agency_id
├── assigned_at
├── assigned_amount
├── commission_rate
├── status
└── external_reference
58. Agency Commission

Ejemplo:

Recovered
$10,000


Agency Fee
20%


EVOXA Net
$8,000

Debe integrarse con Accounting.

59. External Recovery

El proveedor puede reportar:

Recovered
$5,000


Outstanding
$5,000

EVOXA debe reconciliar:

Agency Report
vs
EVOXA Collection Ledger
60. Collection Recovery

KPIs:

Gross Collections
Net Collections
Recovery Rate
Collection Cost
Days to Recovery
61. Recovery Rate

Conceptualmente:

Recovered Amount
÷
Debt Referred
=
Recovery Rate

Ejemplo:

Recovered
$800K


Referred
$1M


Recovery Rate
80%
62. DSO

Days Sales Outstanding mide cuánto tarda en cobrarse una venta.

Conceptualmente:

Accounts Receivable
÷
Revenue
×
Days

Es una métrica financiera importante.

63. DSO Dashboard
Current DSO
42 days


Previous Month
39 days


Variance
+3 days

Un aumento sostenido puede indicar problemas de cobranza.

64. Collection Effectiveness

Métricas:

DSO;
Collection Effectiveness Index;
past-due rate;
recovery rate;
payment success rate;
promise-to-pay fulfillment;
average collection time;
bad debt rate.
65. Past Due Rate
Past Due AR
÷
Total AR

Ejemplo:

Past Due
$300K


Total AR
$2M


Past Due Rate
15%
66. Collection Effectiveness Index

Puede utilizar una métrica como:

CEI

para evaluar cuánto de lo potencialmente cobrable se recuperó durante un período.

La fórmula exacta debe definirse de acuerdo con la metodología financiera utilizada por EVOXA.

67. Dunning Effectiveness

El sistema debe analizar:

Stage 1
Recovery 40%


Stage 2
Recovery 25%


Stage 3
Recovery 15%


Stage 4
Recovery 10%

Esto permite saber qué acciones son realmente efectivas.

68. Dunning A/B Testing

EVOXA puede evolucionar a:

Template A
→ Recovery 35%


Template B
→ Recovery 42%

Pero cualquier experimentación debe respetar las restricciones legales y operativas aplicables a comunicaciones de cobranza.

69. Smart Dunning

La evolución:

Generic Dunning
       ↓
Segmented Dunning
       ↓
Risk-Based Dunning
       ↓
AI-Assisted Dunning

La IA puede recomendar:

Best Channel
Best Time
Best Message
Best Retry Strategy

pero las reglas de compliance deben prevalecer.

70. Collection Risk

El riesgo puede considerar:

Payment History
Days Past Due
Debt Amount
Failed Payments
Customer Tenure
Contract Value
Promise History
Credit Profile

Resultado:

LOW
MEDIUM
HIGH
CRITICAL
71. Collection Risk Model
Customer
   │
   ├── Payment History
   ├── AR Aging
   ├── Failed Payments
   ├── Debt
   └── Promises
           │
           ▼
      Risk Engine
           │
           ▼
     Collection Risk
72. Account Restriction

Cuando la deuda supera ciertos criterios:

Past Due
   ↓
Grace Period
   ↓
Warning
   ↓
Restriction

Restricciones posibles:

LIMIT_USAGE
DISABLE_NEW_PURCHASES
LIMIT_FEATURES
SUSPEND_SERVICE

No necesariamente todas las funciones deben bloquearse.

73. Suspension Policy
SuspensionPolicy
│
├── trigger
├── grace_period
├── warning_days
├── restricted_features
├── suspension_action
├── reactivation_condition
└── status
74. Reactivation

Cuando se paga:

Payment Received
   ↓
Balance Validated
   ↓
Restriction Removed
   ↓
Service Reactivated

Debe poder automatizarse.

75. Partial Payment

Si:

Outstanding
$10,000


Payment
$6,000

queda:

Outstanding
$4,000

Dunning debe continuar sobre el saldo restante.

76. Multiple Invoices

Customer:

Invoice A
$5,000


Invoice B
$3,000


Invoice C
$2,000

Payment:

$6,000

Debe existir una política de aplicación:

Oldest First
Specific Invoice
Proportional
Customer Directed
77. Payment Allocation
Payment
$6,000

puede aplicarse:

Invoice A
$5,000 → Paid


Invoice B
$1,000 → Partial

Esto debe quedar registrado.

78. Unapplied Cash

Si llega:

Payment
$5,000

pero no puede identificarse la invoice:

Unapplied Cash
$5,000

Debe existir un workflow:

Identify
   ↓
Match
   ↓
Apply
79. Payment Matching

El sistema puede usar:

Invoice Number
Customer
Amount
Reference
Bank Account
Payment Metadata

para realizar matching automático.

80. Auto-Reconciliation
Payment
   ↓
Matching Engine
   ↓
Invoice
   ↓
Apply
   ↓
AR Update

Cuando la confianza sea insuficiente:

Manual Review
81. Collection Disputes

No toda deuda vencida es una deuda legítima.

Puede existir:

Invoice Dispute

Entonces:

Past Due
   ↓
Dispute
   ↓
Dunning Pause

según las reglas configuradas.

82. Dispute vs Collection
Unpaid
+
Disputed

no necesariamente debe recibir el mismo tratamiento que:

Unpaid
+
Undisputed

Esto se conecta con:

26/20 — Billing Refunds, Adjustments & Disputes.

83. Dunning Pause

Puede pausarse por:

Active Dispute
Payment Plan
Manual Review
Customer Escalation
Legal Hold

Debe existir:

Pause Reason
Started At
Expected Resume
Approved By
84. Legal Hold

Para determinados casos:

Legal Hold

puede bloquear:

Automatic Dunning
Automatic Suspension
External Collection

hasta que se libere el caso.

85. Compliance

Collections debe respetar:

legislación local;
contratos;
privacidad;
consentimiento cuando corresponda;
horarios de comunicación;
límites de contacto;
retención de información;
derechos del consumidor;
reglas específicas de cobranza.

La arquitectura debe ser configurable por jurisdicción.

86. Collection Policy by Country
Chile
   ↓
Collection Policy CL


USA
   ↓
Collection Policy US


Mexico
   ↓
Collection Policy MX

No debe existir una única política global rígida.

87. Collection Communication Audit

Cada contacto:

Date
Channel
Template
Recipient
Result
Agent

debe quedar registrado.

Ejemplo:

2026-08-19
Email
Dunning Stage 2
Sent
88. Collection Audit Trail

Debe conservar:

Dunning Started
Stage Changed
Message Sent
Payment Attempted
Payment Received
Promise Created
Promise Broken
Case Escalated
Case Assigned
Write-Off Approved
Case Closed
89. Collection Permissions

Permisos sugeridos:

collections.view
collections.manage
collections.assign
collections.contact
collections.promise.create
collections.payment_plan.manage
collections.escalate
collections.writeoff.request
collections.writeoff.approve
collections.agency.manage
collections.report
90. Separation of Duties

Una buena arquitectura:

Collection Agent
→ Manage Case


Supervisor
→ Approve Settlement


Finance
→ Approve Write-Off


Accounting
→ Post Financial Entry

Evita que una sola persona controle todo el proceso.

91. Collection API
Aging
GET /billing/collections/aging
Dunning
GET  /billing/collections/dunning
POST /billing/collections/dunning/{id}/execute
Cases
GET  /billing/collections/cases
POST /billing/collections/cases
GET  /billing/collections/cases/{id}
PATCH /billing/collections/cases/{id}
92. Promise API
POST /billing/collections/promises
GET  /billing/collections/promises
POST /billing/collections/promises/{id}/fulfill
POST /billing/collections/promises/{id}/break
93. Payment Plan API
POST /billing/collections/payment-plans
GET  /billing/collections/payment-plans
POST /billing/collections/payment-plans/{id}/approve
POST /billing/collections/payment-plans/{id}/cancel
94. Collection Agency API
POST /billing/collections/agencies
GET  /billing/collections/agencies
POST /billing/collections/cases/{id}/assign-agency
POST /billing/collections/cases/{id}/recall
95. Dunning Data Model
DunningPolicy
│
├── id
├── name
├── customer_segment
├── grace_period
├── status
└── metadata
96. Dunning Stage Model
DunningStage
│
├── id
├── policy_id
├── sequence
├── days_past_due
├── actions
├── escalation_level
├── status
└── metadata
97. Collection Case Model
CollectionCase
│
├── id
├── customer_id
├── billing_account_id
├── tenant_id
├── legal_entity_id
├── outstanding_amount
├── oldest_invoice_date
├── days_past_due
├── priority
├── risk_score
├── status
├── assigned_to
├── external_agency_id
└── metadata
98. Collection Event Model
CollectionEvent
│
├── id
├── case_id
├── event_type
├── actor
├── timestamp
├── amount
├── channel
├── outcome
└── metadata
99. Payment Promise Model
PaymentPromise
│
├── id
├── case_id
├── customer_id
├── promised_amount
├── promised_date
├── status
├── fulfilled_amount
├── fulfilled_at
└── metadata
100. Payment Plan Model
PaymentPlan
│
├── id
├── case_id
├── total_amount
├── installment_amount
├── frequency
├── number_of_installments
├── start_date
├── end_date
├── status
└── metadata
101. Collection Agency Model
CollectionAgency
│
├── id
├── name
├── country
├── contact
├── commission_rate
├── status
└── metadata
102. Dunning Communication Model
DunningCommunication
│
├── id
├── case_id
├── stage_id
├── channel
├── template_id
├── recipient
├── sent_at
├── delivery_status
├── opened_at
└── metadata
103. Collection Architecture
                       ACCOUNTS RECEIVABLE
                               │
                               ▼
                        Aging Engine
                               │
                               ▼
                       Risk Evaluation
                               │
                               ▼
                        Dunning Engine
                               │
                 ┌─────────────┼─────────────┐
                 ▼             ▼             ▼
              Email           SMS         Portal
                 │             │             │
                 └─────────────┼─────────────┘
                               ▼
                        Payment Recovery
                               │
                         ┌─────┴─────┐
                         ▼           ▼
                    Recovered     Unresolved
                                      │
                                      ▼
                                  Collection
                                      │
                        ┌─────────────┼────────────┐
                        ▼             ▼            ▼
                     Agent       Payment Plan   Agency
                        │             │            │
                        └─────────────┼────────────┘
                                      ▼
                                   Recovery
                                      │
                                      ▼
                                  Accounting
104. Dunning Automation

Ejemplo completo:

Invoice Due
   ↓
Payment Attempt
   ↓
Failed
   ↓
Retry
   ↓
Failed
   ↓
Dunning Stage 1
   ↓
Reminder
   ↓
Retry
   ↓
Failed
   ↓
Dunning Stage 2
   ↓
Warning
   ↓
Customer Pays
   ↓
Dunning Closed
105. Enterprise Collection Automation

Para una cuenta Enterprise:

Invoice
$250,000


Due
Net 60


Day 61
Reminder


Day 70
Account Manager Notification


Day 75
Finance Escalation


Day 90
Collection Case


Day 100
Executive Review

No necesariamente debe suspenderse el servicio automáticamente.

106. Self-Service Collection Automation

Para SaaS:

Payment Failed
   ↓
Retry
   ↓
Update Card
   ↓
Retry
   ↓
Grace Period
   ↓
Restricted Mode
   ↓
Suspension
107. Dunning vs Customer Experience

Una buena arquitectura no debe buscar simplemente:

Maximum Collection Pressure

sino:

Maximum Recovery
+
Minimum Customer Friction

Por eso EVOXA debe permitir segmentar las estrategias.

108. Collection Analytics

Dashboard:

Collections
│
├── Total AR
├── Past Due AR
├── DSO
├── Collection Rate
├── Recovery Rate
├── Failed Payments
├── Dunning Success
├── Promise Fulfillment
├── Payment Plans
├── Write-Offs
└── Bad Debt
109. Dunning Funnel
Past Due
100%


Contacted
95%


Payment Attempt
80%


Recovered
65%


Escalated
20%


Written Off
5%

Esto permite medir la eficacia del proceso.

110. Collections by Aging
1–30
$500K
Recovery 80%


31–60
$300K
Recovery 65%


61–90
$200K
Recovery 45%


90+
$150K
Recovery 20%

Esto permite identificar dónde se pierde dinero.

111. Collections by Customer Segment
SMB
Recovery 82%


Mid-Market
Recovery 88%


Enterprise
Recovery 91%


Strategic
Recovery 94%
112. Collections Forecast

Puede estimar:

Expected Collections
Next 30 Days
$2.5M


Next 60 Days
$3.2M


Next 90 Days
$4.0M

basado en:

AR Aging
Payment History
Promises
Payment Plans
Risk
113. Cash Collection Forecast

Esto alimenta:

Cash Flow Forecast

con:

Expected Collections
+
Expected New Payments
-
Refunds
-
Fees
114. AI Collections

La IA puede recomendar:

Customer
$50K overdue

y sugerir:

High recovery probability
Contact Account Manager
Offer payment plan
Retry payment tomorrow

Pero:

Las reglas contractuales, legales y de compliance deben tener prioridad sobre las recomendaciones del modelo.

115. Collection Intelligence

El sistema puede detectar:

Customer A
Usually pays on Day 5


Customer B
Usually pays after reminder


Customer C
Frequent payment failures


Customer D
Increasing debt

Esto permite personalizar estrategias.

116. Revenue Leakage vs Collection Leakage

Importante distinguir:

Revenue Leakage
Usage
→ Billing

se pierde Revenue.

Collection Leakage
Invoice
→ Payment

se genera Revenue/Billing, pero no se cobra.

Ambos deben analizarse por separado.

117. Collection and Accounting

Cuando se cobra:

Payment
   ↓
AR Reduction
   ↓
Cash Increase

Cuando se hace Write-Off:

Write-Off
   ↓
AR Reduction
   ↓
Bad Debt Expense

Cuando se negocia un Settlement:

Settlement
   ↓
AR Adjustment
   ↓
Financial Adjustment
118. Collection and Revenue

Collections no debería modificar directamente Revenue.

Por ejemplo:

Invoice
$10,000


Unpaid

no significa:

Revenue = $0

Revenue Management y Accounting determinan el tratamiento correspondiente.

119. Collection and Tax

Una deuda vencida:

Invoice
$10,000
Tax
$1,900

no significa automáticamente que:

Tax = Reversed

El tratamiento fiscal de créditos, incobrables o ajustes depende de la jurisdicción.

Por eso Tax y Collections deben estar desacoplados pero integrados.

120. Collection and Disputes
Invoice
   ↓
Dispute
   ↓
Collection Hold

El sistema debe poder detener temporalmente determinadas acciones mientras el dispute está activo.

Esto conecta con:

26/20 — Billing Refunds, Adjustments & Disputes.

121. Collection and Subscription

Una subscription puede tener:

ACTIVE
PAST_DUE
RESTRICTED
SUSPENDED
CANCELED

Dunning puede producir transiciones:

ACTIVE
   ↓
PAST_DUE
   ↓
RESTRICTED
   ↓
SUSPENDED

pero las reglas deben ser configurables.

122. Account State Machine
ACTIVE
  │
  ▼
PAST_DUE
  │
  ├───────────────┐
  ▼               ▼
RESTRICTED      RECOVERED
  │
  ▼
SUSPENDED
  │
  ├───────────────┐
  ▼               ▼
CANCELED       REACTIVATED
123. Collection Recovery

Cuando se paga:

Payment Received
      ↓
Payment Applied
      ↓
Outstanding Balance Updated
      ↓
Dunning Evaluated
      ↓
Case Resolved
      ↓
Service Restored
124. Collection Close

Un caso puede cerrarse por:

PAID
SETTLED
WRITTEN_OFF
DISPUTED
CANCELED
TRANSFERRED

Debe existir una razón obligatoria.

125. Collection Close Controls

Antes de cerrar:

Outstanding Balance
Payment Status
Promises
Disputes
Adjustments
Agency Status

deben estar reconciliados.

126. Collection Security

Debe existir:

Tenant Isolation
Legal Entity Isolation
RBAC
PII Protection
Encryption
Audit Logs
Approval Controls

Especialmente porque Collections maneja información financiera y personal.

127. Financial Privacy

El sistema debe minimizar la exposición de:

Payment Details
Bank Information
Personal Information
Collection Notes
Financial Risk

a usuarios que no necesitan verla.

128. Collection Audit

Auditoría completa:

Invoice
   ↓
Past Due
   ↓
Dunning
   ↓
Communication
   ↓
Payment Attempt
   ↓
Collection Case
   ↓
Promise
   ↓
Payment
   ↓
Resolution
129. Core Principles
1. Recover Before Escalate

Intentar recuperar antes de pasar a Collection.

2. Segment Customers

Enterprise y SMB no necesariamente requieren la misma estrategia.

3. Automate Carefully

Automatizar lo repetitivo, no las decisiones sensibles sin control.

4. Respect Disputes

Una deuda disputada no debe tratarse igual que una deuda indiscutida.

5. Preserve History

Nunca eliminar el historial de cobranza.

6. Integrate With Payments

Payment Recovery depende directamente de Payments.

7. Integrate With Accounting

Write-offs, settlements y recoveries deben contabilizarse correctamente.

8. Multi-Tenant

Collections debe respetar la separación de Tenants y Legal Entities.

9. Compliance First

Las estrategias de cobranza deben adaptarse a cada jurisdicción.

10. Measure Recovery

No basta con enviar emails; EVOXA debe medir cuánto dinero recupera cada estrategia.

130. Resultado de 26/19

Billing Dunning & Collections crea la capa de recuperación de cuentas por cobrar de EVOXA.

La cadena completa queda:

Invoice
   ↓
Accounts Receivable
   ↓
Due Date
   ↓
Payment Attempt
   ↓
Payment Failure
   ↓
Retry
   ↓
Past Due
   ↓
Dunning
   ↓
Collection Case
   ↓
Promise / Payment Plan
   ↓
Recovery
   ↓
Accounting

Y cuando la recuperación no funciona:

Collection
   ↓
Escalation
   ↓
External Agency
   ↓
Settlement / Recovery
   ↓
Write-Off
   ↓
Accounting
Arquitectura acumulada 26/08 → 26/19
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

La arquitectura financiera ya tiene prácticamente cerrado todo el circuito:

                     BILLING
                        │
                        ▼
                    INVOICE
                        │
                        ▼
                       AR
                        │
            ┌───────────┴───────────┐
            ▼                       ▼
        PAYMENT                  PAST DUE
            │                       │
            ▼                       ▼
          CASH                   DUNNING
                                    │
                                    ▼
                               COLLECTION
                                    │
                         ┌──────────┼──────────┐
                         ▼          ▼          ▼
                     Recovered   Promise    Agency
                         │          │          │
                         └──────────┼──────────┘
                                    ▼
                               RESOLUTION
                                    │
                                    ▼
                                ACCOUNTING

Y las capas anteriores siguen funcionando en paralelo:

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
Charges
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

26/19 establece el sistema de recuperación financiera de EVOXA: desde el primer Payment Failure hasta Dunning, Collection Cases, Payment Plans, Promises to Pay, escalamiento, agencias externas, Recovery, Write-Off y reconciliación contable, manteniendo separación entre AR, Revenue, Collections y Accounting.

Siguiente capítulo
26/20 — Billing Refunds, Adjustments & Disputes

Aquí entraremos en una de las capas más delicadas del Billing: refunds completos y parciales, credit notes, debit notes, billing adjustments, disputes, chargebacks, reversals, cancellations, approvals, workflows, impacto en Revenue, Tax, AR, Payments y Accounting, y cómo mantener toda la trazabilidad sin alterar el historial financiero.
