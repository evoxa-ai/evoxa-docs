1. Billing Accounting & Financial Integration

Billing Accounting & Financial Integration es la capa que conecta el Billing Platform con la estructura financiera y contable de EVOXA.

Hasta ahora tenemos:

26/13  Invoicing
26/14  Payments
26/15  Credits, Discounts & Adjustments
26/16  Taxes & Compliance
26/17  Revenue Management

Esta capa toma todos esos eventos y los transforma en información financiera estructurada:

Billing
   ↓
Subledgers
   ↓
Accounting Entries
   ↓
General Ledger
   ↓
Financial Reporting
   ↓
ERP / Accounting System

La regla fundamental es:

Billing administra la lógica comercial y financiera de las transacciones; Accounting registra sus efectos contables.

2. Propósito

Esta capa debe administrar:

Chart of Accounts;
Accounting Rules;
Journal Entries;
Journal Entry Lines;
Accounts Receivable;
Accounts Payable cuando corresponda;
Cash;
Revenue;
Deferred Revenue;
Tax Liabilities;
Credits;
Refunds;
Payment Fees;
Settlement;
Financial Subledgers;
General Ledger integration;
Accounting Periods;
Closing;
Reconciliation;
Multi-currency;
FX;
ERP integrations;
Accounting exports;
Financial audit;
Accounting adjustments.
3. Billing vs Accounting

Una distinción fundamental:

Billing

Responde:

¿Qué debe pagar el Customer?

Accounting

Responde:

¿Cuál es el efecto financiero y contable de esa operación?

Ejemplo:

Invoice
$1,190

Billing:

Revenue Base
$1,000


Tax
$190


Total
$1,190

Accounting puede registrar conceptualmente:

Accounts Receivable
+1,190


Revenue
+1,000


Tax Payable
+190
4. Accounting Architecture
                    BILLING PLATFORM
                           │
       ┌───────────────────┼───────────────────┐
       ▼                   ▼                   ▼
    Invoices             Payments           Revenue
       │                   │                   │
       ▼                   ▼                   ▼
                  FINANCIAL SUBLEDGERS
                           │
                           ▼
                    ACCOUNTING ENGINE
                           │
                           ▼
                    JOURNAL ENTRIES
                           │
                           ▼
                     GENERAL LEDGER
                           │
              ┌────────────┼────────────┐
              ▼            ▼            ▼
           Finance      Reporting      ERP
5. Financial Subledgers

EVOXA no debería enviar directamente cada evento a un ERP sin una capa intermedia.

Debe existir:

Billing
   ↓
Financial Subledgers
   ↓
Accounting Engine
   ↓
GL / ERP

Subledgers posibles:

Accounts Receivable
Revenue
Deferred Revenue
Tax
Payments
Credits
Refunds
Settlement
6. General Ledger

El General Ledger es el nivel contable central.

Ejemplo:

Accounts Receivable
Revenue
Tax Payable
Cash
Deferred Revenue
Payment Fees

Billing genera los hechos económicos; Accounting genera los asientos.

7. Chart of Accounts

EVOXA debe soportar un Chart of Accounts configurable.

Ejemplo:

1000 Assets
│
├── 1100 Cash
├── 1200 Accounts Receivable
└── 1300 Contract Assets


2000 Liabilities
│
├── 2100 Tax Payable
└── 2200 Deferred Revenue


4000 Revenue
│
├── 4100 Subscription Revenue
├── 4200 Usage Revenue
└── 4300 Services Revenue


5000 Expenses
│
└── 5100 Payment Processing Fees
8. Account Model
Account
│
├── id
├── code
├── name
├── type
├── category
├── parent_id
├── currency
├── legal_entity_id
├── status
└── metadata

Tipos:

ASSET
LIABILITY
EQUITY
REVENUE
EXPENSE
9. Accounting Rules

Billing debe poder determinar:

Transaction
   ↓
Accounting Rule
   ↓
Debit Account
Credit Account

Ejemplo:

Invoice
$1,000

Regla:

Debit
Accounts Receivable


Credit
Subscription Revenue
10. Accounting Rule Engine
                     TRANSACTION
                          │
                          ▼
                  ACCOUNTING RULE
                          │
             ┌────────────┴────────────┐
             ▼                         ▼
        Debit Account            Credit Account
             │                         │
             └────────────┬────────────┘
                          ▼
                    Journal Entry
11. Journal Entry

Un asiento debe ser balanceado:

Journal Entry
│
├── Debit
│
└── Credit

Regla:

Total Debits
=
Total Credits

Nunca debe existir un asiento contable desequilibrado.

12. Journal Entry Example — Invoice

Invoice:

$1,190

Accounting:

Debit
Accounts Receivable
$1,190


Credit
Revenue
$1,000


Credit
Tax Payable
$190

Resultado:

Debits
$1,190


Credits
$1,190
13. Journal Entry Model
JournalEntry
│
├── id
├── entry_number
├── legal_entity_id
├── accounting_period_id
├── transaction_date
├── posting_date
├── currency
├── source_type
├── source_id
├── status
├── description
└── metadata
14. Journal Entry Lines
JournalEntryLine
│
├── id
├── journal_entry_id
├── account_id
├── debit
├── credit
├── currency
├── exchange_rate
├── amount
├── customer_id
├── billing_account_id
├── cost_center_id
├── department_id
└── metadata
15. Accounting Event

Cada operación de Billing puede generar un evento:

InvoiceIssued
PaymentCaptured
PaymentSettled
CreditIssued
RefundIssued
TaxCalculated
RevenueRecognized
AdjustmentPosted

Luego:

Event
 ↓
Accounting Rule
 ↓
Journal Entry
16. Invoice Accounting

Cuando se emite:

Invoice
$1,000

conceptualmente:

DR Accounts Receivable
   $1,000


CR Revenue
   $1,000

Si existen impuestos:

DR Accounts Receivable
   $1,190


CR Revenue
   $1,000


CR Tax Payable
   $190
17. Payment Accounting

Cuando se recibe:

Payment
$1,190

puede producir:

DR Cash
   $1,190


CR Accounts Receivable
   $1,190

Así:

AR
$1,190
   ↓
Payment
   ↓
AR
$0
18. Payment Fees

Proveedor:

Payment
$1,190


Provider Fee
$30


Settlement
$1,160

Accounting puede representar conceptualmente:

DR Cash
   $1,160


DR Payment Processing Expense
   $30


CR Accounts Receivable
   $1,190

La regla exacta depende de la política contable.

19. Settlement Accounting

Payment:

$10,000

Settlement:

$9,700

Fees:

$300

Debe poder conciliar:

Gross
$10,000


Fees
-$300


Net
$9,700
20. Accounts Receivable

Accounts Receivable (AR) representa dinero que los Customers deben a EVOXA.

Ejemplo:

Customer
ACME


Invoice
$10,000


Payment
$6,000


AR
$4,000
21. AR Aging

Debe soportar:

Current
1–30 Days
31–60 Days
61–90 Days
90+ Days

Ejemplo:

Current       $500K
1–30          $200K
31–60         $100K
61–90          $50K
90+             $80K

Esto se conecta directamente con:

26/19 — Billing Dunning & Collections.

22. AR Ledger
AR Ledger
│
├── Invoice
├── Payment
├── Credit
├── Adjustment
├── Write-Off
└── Refund

El saldo:

Opening AR
+
Invoices
-
Payments
-
Credits
-
Write-Offs
=
Ending AR
23. Accounts Receivable Reconciliation

Debe cumplirse:

Billing Outstanding Balance
=
AR Subledger Balance

Y:

AR Subledger
=
General Ledger AR Account

Si no:

AR Reconciliation Exception
24. Revenue Accounting

Revenue Management genera:

Revenue Schedule

Accounting transforma eso en:

Journal Entry

Ejemplo:

Recognize $1,000

Conceptualmente:

DR Deferred Revenue
   $1,000


CR Revenue
   $1,000
25. Deferred Revenue Accounting

Cuando se recibe un pago anticipado:

DR Cash
   $12,000


CR Deferred Revenue
   $12,000

Luego cada mes:

DR Deferred Revenue
   $1,000


CR Revenue
   $1,000
26. Contract Asset Accounting

Si existe Revenue ganado antes de facturar:

DR Contract Asset
CR Revenue

Posteriormente:

DR Accounts Receivable
CR Contract Asset

La plataforma debe poder soportar este ciclo cuando la política contable aplicable lo requiera.

27. Tax Accounting

Si se genera:

Tax
$190

conceptualmente:

DR Accounts Receivable
$190


CR Tax Payable
$190

Esto permite separar:

Revenue
≠
Tax Liability
28. Credit Accounting

Un crédito puede requerir:

DR Revenue Adjustment
CR Accounts Receivable

o una cuenta específica, dependiendo de la naturaleza.

La regla debe ser configurable.

29. Credit Note Accounting

Ejemplo:

Invoice
$1,000


Credit Note
-$200

Puede producir:

DR Revenue Adjustment
$200


CR Accounts Receivable
$200

Si existe impuesto, también deben generarse los efectos fiscales correspondientes.

30. Debit Note Accounting
Debit Note
+$200

Conceptualmente:

DR Accounts Receivable
$200


CR Revenue
$200
31. Refund Accounting

Payment original:

$1,000

Refund:

$200

El asiento puede representar:

DR Refund / Liability / Revenue Adjustment
$200


CR Cash
$200

La cuenta exacta depende de la naturaleza de la devolución y la política contable.

32. Write-Off Accounting

Outstanding:

$500

Write-off:

DR Bad Debt Expense
$500


CR Accounts Receivable
$500

Esto debe estar conectado con las políticas de credit management.

33. Bad Debt

La plataforma debe distinguir:

Past Due
≠
Bad Debt
≠
Write-Off

Una invoice vencida no significa automáticamente que sea incobrable.

34. Allowance for Doubtful Accounts

En arquitecturas Enterprise puede existir:

Accounts Receivable
$1M


Expected Credit Loss
$50K

Esto puede alimentar:

Allowance

según la política contable.

35. Accounting Adjustments

Las correcciones contables deben ser:

Immutable
Auditable
Reversible
Approved

Nunca:

UPDATE journal_entry

para modificar el pasado.

Mejor:

Original Entry
   ↓
Reversal
   ↓
Correct Entry
36. Accounting Reversal

Ejemplo:

Original
Revenue +$10,000

Reversal:

Revenue -$10,000

Nuevo:

Revenue +$9,000

Resultado:

Revenue
$9,000
37. Accounting Periods

EVOXA debe administrar períodos:

2026
│
├── January
├── February
├── March
...
└── December

Estados:

OPEN
SOFT_CLOSED
CLOSED
LOCKED
38. Period Close

Proceso:

Billing Close
      ↓
Revenue Close
      ↓
Tax Reconciliation
      ↓
AR Reconciliation
      ↓
Payment Reconciliation
      ↓
Accounting Close
      ↓
Financial Reporting
39. Period Lock

Después de cerrar:

August 2026
LOCKED

No deben modificarse directamente:

invoices;
payments;
revenue entries;
journal entries.

Las correcciones deben producir nuevas transacciones.

40. Accounting Calendar

Debe existir:

AccountingCalendar
│
├── Fiscal Year
├── Fiscal Period
├── Start Date
├── End Date
├── Status
└── Closing Checklist
41. Fiscal Year

No todas las empresas utilizan:

January → December

Por lo tanto:

Fiscal Year
Start
End

debe ser configurable por Legal Entity.

42. Multi-Entity Accounting

EVOXA puede tener:

Organization
│
├── EVOXA Chile
├── EVOXA USA
├── EVOXA Mexico
└── EVOXA Europe

Cada entidad puede tener:

Chart of Accounts
Currency
Fiscal Calendar
Tax Rules
Accounting Policies
43. Entity-Level Accounting

Un Revenue debe asociarse a:

Organization
Tenant
Legal Entity
Billing Account
Customer
Contract

Ejemplo:

ACME
   ↓
Tenant A
   ↓
EVOXA Chile
   ↓
Revenue
$100,000
44. Multi-Currency Accounting

Puede existir:

Transaction
USD 10,000


Functional Currency
CLP


FX Rate
950


Functional Amount
CLP 9,500,000

El sistema debe conservar:

Transaction Amount
Functional Amount
Exchange Rate
Rate Date
Rate Source
45. FX Gain / Loss

Si posteriormente:

Original
USD 10,000 × 950
= CLP 9,500,000

y al settlement:

USD 10,000 × 970
= CLP 9,700,000

la diferencia:

CLP 200,000

puede representar una diferencia de FX según la política contable.

Debe distinguirse de Revenue.

46. Exchange Rate Service

EVOXA puede utilizar:

Internal FX
External FX Provider
ERP FX Rates
Central Bank Data

La fuente debe registrarse.

47. Dimensions

Accounting Entries deberían soportar dimensiones:

Legal Entity
Tenant
Customer
Product
Business Unit
Department
Cost Center
Profit Center
Region
Project
Contract

Ejemplo:

Revenue
$100,000


Business Unit
AI


Region
Chile


Customer
ACME
48. Cost Centers

Aunque Billing se concentra en Revenue, debe poder enviar:

Cost Center
CC-AI-001

para permitir análisis financiero.

49. Profit Centers

También:

Profit Center
AI Platform

Esto permite:

Revenue
-
Costs
=
Profitability
50. Accounting Segments

Una cuenta puede utilizar segmentos:

Account
4100 Revenue


Entity
CL01


Department
AI


Region
CL


Product
AI Platform

El sistema debe permitir mapear estos segmentos al ERP.

51. Accounting Mapping

Ejemplo:

Billing Product
AI Platform


↓ Mapping


GL Account
4200


Cost Center
AI-001


Tax Account
2100
52. Accounting Mapping Engine
Billing Event
     ↓
Mapping Rules
     ↓
GL Account
     ↓
Dimensions
     ↓
Journal Entry
53. ERP Integration

EVOXA debe poder integrarse con:

ERP / Accounting System
        │
        ▼
Integration Layer
        │
        ▼
EVOXA Accounting Adapter

No conviene acoplar Billing directamente a un ERP específico.

54. ERP Adapter
AccountingProvider
│
├── createJournalEntry()
├── postJournalEntry()
├── reverseJournalEntry()
├── getAccountBalance()
├── getPeriodStatus()
└── reconcile()
55. ERP Integrations

La arquitectura puede soportar:

EVOXA
│
├── ERP A
├── ERP B
├── ERP C
└── Internal GL

mediante adapters.

56. Integration Modes
Real-Time
Invoice
 ↓
Accounting Entry
 ↓
ERP
Batch
Daily
 ↓
Generate Accounting Batch
 ↓
Send ERP
Hybrid
Critical Events
→ Real-Time


Reports
→ Batch
57. Accounting Batch

Un batch puede contener:

AccountingBatch
│
├── Batch Number
├── Period
├── Entity
├── Entry Count
├── Debit Total
├── Credit Total
├── Status
└── ERP Reference
58. Batch Validation

Antes de enviar:

Debits = Credits
Accounts Valid
Period Open
Currency Valid
Dimensions Valid
No Duplicates

Si falla:

Batch
REJECTED
59. Integration Idempotency

Si se envía dos veces:

Invoice
INV-1000

el ERP no debe recibir dos asientos.

Debe existir:

External Idempotency Key

Ejemplo:

EVOXA-INV-1000-ACCOUNTING
60. Integration Status
PENDING
SENT
ACCEPTED
REJECTED
RETRYING
FAILED
RECONCILED
61. Integration Retry
ERP
Unavailable
   ↓
Retry
   ↓
Retry
   ↓
Success

Pero:

Accounting Entry

debe permanecer idempotente.

62. Integration Outbox

Para resiliencia:

Billing Transaction
       ↓
Accounting Event
       ↓
Outbox
       ↓
Integration Worker
       ↓
ERP

Esto evita perder eventos.

63. Accounting Event Outbox
AccountingOutbox
│
├── event_id
├── event_type
├── source_type
├── source_id
├── payload
├── status
├── attempts
├── last_attempt_at
└── processed_at
64. Financial Reconciliation

Debe reconciliar:

Billing
   ↓
Subledger
   ↓
General Ledger
   ↓
ERP

Ejemplo:

Billing AR
$1,000,000


AR Subledger
$1,000,000


GL AR
$1,000,000


ERP AR
$1,000,000

Resultado:

Variance = $0
65. Reconciliation Exception

Si:

Billing
$1,000,000


GL
$999,500

entonces:

Variance
$500

Debe crearse:

Reconciliation Exception
66. Reconciliation Workflow
Detected
   ↓
Assigned
   ↓
Investigating
   ↓
Root Cause
   ↓
Correction
   ↓
Reconciled
67. Financial Close Dashboard
August 2026 Close
│
├── Billing Reconciliation       ✓
├── AR Reconciliation            ✓
├── Revenue Reconciliation       ✓
├── Tax Reconciliation           ✓
├── Payment Reconciliation       ✓
├── ERP Integration              ✓
├── Exceptions                   2
└── Close Status                 OPEN
68. Accounting Audit Trail

Cada asiento debe conservar:

Source
Source ID
Rule
Created By
Created At
Posted By
Posted At
Period
ERP Reference
Reversal Reference

Ejemplo:

Invoice
INV-1000


→ Journal Entry
JE-2026-00500


→ ERP Entry
ERP-88921
69. Source Traceability

Debe poder navegar:

Customer
   ↓
Contract
   ↓
Subscription
   ↓
Charge
   ↓
Invoice
   ↓
Payment
   ↓
Revenue
   ↓
Journal Entry
   ↓
GL

Esta trazabilidad será crítica para auditorías.

70. Accounting Provenance

Cada asiento debe responder:

¿Qué originó este asiento?

Ejemplo:

Source Type
INVOICE


Source ID
INV-2026-00125


Billing Event
InvoiceIssued


Accounting Rule
AR_REVENUE_TAX_V1
71. Financial Documents

El sistema debe relacionar:

Invoice
Credit Note
Debit Note
Payment
Refund
Settlement
Journal Entry

con sus efectos contables.

72. Financial Transaction Model
FinancialTransaction
│
├── id
├── source_type
├── source_id
├── transaction_type
├── amount
├── currency
├── legal_entity_id
├── accounting_date
├── posting_date
├── period_id
└── status
73. Accounting Status

Una transacción puede estar:

UNPOSTED
POSTED
REVERSED
ADJUSTED
RECONCILED
74. Financial Ledger

Podemos separar:

Billing Ledger
Financial Subledger
General Ledger

Conceptualmente:

Billing Ledger
→ Business transaction


Subledger
→ Detailed financial balance


General Ledger
→ Accounting summary
75. Subledger vs General Ledger

Ejemplo:

Customer A
$500


Customer B
$300


Customer C
$200

AR Subledger:

$1,000

GL:

Accounts Receivable
$1,000

El Subledger contiene el detalle; el GL contiene el saldo contable agregado.

76. Customer Statement

El Customer puede recibir:

Opening Balance
+
Invoices
-
Payments
-
Credits
+
Adjustments
=
Closing Balance

Ejemplo:

Opening
$500


Invoice
+$1,000


Payment
-$800


Credit
-$100


Closing
$600
77. Financial Statement Integration

Accounting debe alimentar:

Balance Sheet
Income Statement
Cash Flow

Billing principalmente contribuye a:

Revenue
Accounts Receivable
Deferred Revenue
Tax Liabilities
Cash
Fees
78. Balance Sheet Impact

Billing puede afectar:

Assets
Cash
Accounts Receivable
Contract Assets
Liabilities
Deferred Revenue
Tax Payable
Customer Credits
79. Income Statement Impact

Puede afectar:

Revenue
Payment Processing Expense
Bad Debt Expense
Revenue Adjustments
80. Cash Flow

Payments pueden alimentar:

Operating Cash Flow

pero Revenue Recognition y Cash son conceptos diferentes.

81. Financial Reporting

Reportes:

Revenue;
AR;
Deferred Revenue;
Tax Liability;
Cash Collections;
Refunds;
Credits;
Payment Fees;
Bad Debt;
Adjustments;
FX.
82. Revenue Reconciliation Example
Revenue Management
$5,000,000


Accounting Revenue
$5,000,000


Variance
$0

Si:

Revenue Management
$5,000,000


Accounting
$4,950,000

debe investigarse:

Variance
$50,000
83. AR Reconciliation Example
Invoices
$10M


Payments
-$7M


Credits
-$500K


Write-offs
-$100K


Expected AR
$2.4M

El AR Ledger debe mostrar:

$2.4M
84. Deferred Revenue Reconciliation
Opening
$3M


New Billings
+$1M


Recognized
-$800K


Ending
$3.2M

Debe coincidir con:

Deferred Revenue Subledger
=
GL Deferred Revenue
85. Tax Reconciliation
Tax Engine
$500K


Tax Subledger
$500K


GL Tax Payable
$500K

Debe cerrar:

Variance = $0
86. Payment Reconciliation
Payments Captured
$1M


Settlement
$970K


Fees
$30K

Debe cerrar:

$1M
=
$970K + $30K
87. Accounting Dimensions

Para Enterprise:

Legal Entity
Tenant
Business Unit
Department
Cost Center
Profit Center
Product
Customer
Contract
Region
Currency

Esto permite reporting financiero multidimensional.

88. Multi-Tenant Accounting

La arquitectura debe soportar:

Organization
│
├── Tenant A
│   └── Revenue
│
├── Tenant B
│   └── Revenue
│
└── Tenant C
    └── Revenue

Pero también consolidar:

Organization Total
=
Tenant A
+
Tenant B
+
Tenant C
89. Consolidated Accounting

Para grupos empresariales:

EVOXA Group
│
├── Chile
├── USA
└── Mexico

puede existir:

Entity Financials
       ↓
Consolidation
       ↓
Eliminations
       ↓
Group Financial Statements

La consolidación completa puede pertenecer a una capa financiera superior, pero Billing debe proporcionar los datos correctamente clasificados.

90. Intercompany

Si una entidad factura a otra:

EVOXA Chile
      ↓
EVOXA USA

debe poder marcarse:

Intercompany = TRUE

para posterior eliminación/consolidación.

91. Financial Controls

Controles esenciales:

Debit = Credit
Period Open
Valid Account
Valid Entity
Valid Currency
Valid Tax
Valid Source
No Duplicate
Approval Required
92. Financial Approval

Operaciones como:

grandes write-offs;
revenue adjustments;
manual journal entries;
unusual credits;

pueden requerir:

Maker
   ↓
Reviewer
   ↓
Approver
   ↓
Poster
93. Manual Journal Entries

Debe existir soporte para ajustes manuales:

POST /accounting/journal-entries

pero con:

permisos;
reason;
approval;
period validation;
audit.
94. Manual Entry Restriction

El sistema debe distinguir:

System Generated
vs
Manual

Ejemplo:

JE-001
Source: Invoice
Type: AUTOMATED

versus:

JE-002
Source: Manual
Type: MANUAL
Reason: Correction
95. Financial Audit

Auditor debe poder consultar:

Invoice
   ↓
Accounting Entry
   ↓
GL Account
   ↓
ERP Entry

y regresar:

GL
   ↓
Subledger
   ↓
Billing
   ↓
Customer
96. Accounting Events

Eventos principales:

JournalEntryCreated
JournalEntryPosted
JournalEntryReversed
AccountingBatchCreated
AccountingBatchPosted
PeriodOpened
PeriodClosed
PeriodLocked
ReconciliationStarted
ReconciliationCompleted
ReconciliationExceptionCreated
97. Accounting APIs
Chart of Accounts
GET  /billing/accounting/accounts
POST /billing/accounting/accounts
PATCH /billing/accounting/accounts/{id}
Journal Entries
GET  /billing/accounting/journal-entries
POST /billing/accounting/journal-entries
GET  /billing/accounting/journal-entries/{id}
Reconciliation
POST /billing/accounting/reconciliation
GET  /billing/accounting/reconciliation
Periods
GET  /billing/accounting/periods
POST /billing/accounting/periods/{id}/close
POST /billing/accounting/periods/{id}/lock
98. ERP API
POST /billing/accounting/integrations/{provider}/sync
GET  /billing/accounting/integrations/{provider}/status
POST /billing/accounting/integrations/{provider}/retry
99. Accounting Data Model
AccountingPeriod
│
├── id
├── legal_entity_id
├── fiscal_year
├── period
├── start_date
├── end_date
├── status
└── closed_at
100. Accounting Integration Model
AccountingIntegration
│
├── id
├── provider
├── legal_entity_id
├── status
├── last_sync_at
├── configuration
└── metadata
101. Integration Mapping Model
AccountingMapping
│
├── id
├── source_type
├── source_code
├── gl_account
├── tax_account
├── cost_center
├── profit_center
├── legal_entity_id
└── effective_from
102. Financial Close Architecture
                PERIOD CLOSE
                     │
       ┌─────────────┼─────────────┐
       ▼             ▼             ▼
     Billing        Tax         Revenue
       │             │             │
       └─────────────┼─────────────┘
                     ▼
                    AR
                     │
                     ▼
                 Payments
                     │
                     ▼
              Reconciliation
                     │
                     ▼
               Journal Entries
                     │
                     ▼
                     GL / ERP
                     │
                     ▼
                  CLOSE
103. Accounting Integration Reliability

La arquitectura debe soportar:

retries;
idempotency;
dead-letter queue;
outbox;
monitoring;
error handling;
reconciliation;
replay.

Ejemplo:

Billing Event
   ↓
Outbox
   ↓
Queue
   ↓
Accounting Worker
   ↓
ERP
104. Dead Letter Queue

Si un evento falla repetidamente:

Invoice
   ↓
Accounting
   ↓
Failed
   ↓
Retry
   ↓
Retry
   ↓
DLQ

Luego:

Finance / Engineering
        ↓
Investigate
        ↓
Replay
105. Accounting Monitoring

Dashboard:

Accounting Integration
│
├── Pending Entries
├── Posted Entries
├── Failed Entries
├── ERP Rejections
├── Reconciliation Variances
├── Unbalanced Entries
└── Period Close Status
106. Financial Data Retention

Los datos contables y fiscales pueden tener requisitos específicos de conservación.

La plataforma debe soportar:

Retention Policy
Archive
Immutable Storage
Audit Access

sin asumir una única duración universal.

107. Accounting Security

Debe existir:

Role Based Access Control
Tenant Isolation
Legal Entity Isolation
Encryption
Audit Logging
Approval Workflow
Least Privilege
108. AI Financial Intelligence

La IA puede ayudar a detectar:

Accounting Anomalies
Reconciliation Variances
Revenue Recognition Issues
Duplicate Entries
Unexpected Credits
Unusual Write-Offs
Cash Collection Patterns

Pero:

La IA no debe modificar automáticamente los registros contables críticos sin controles y aprobación apropiados.

109. Financial Anomaly Detection

Ejemplo:

Typical Payment Fee
2.5%


Observed
8%


Variance
+5.5%

El sistema genera:

Financial Anomaly
110. Financial Forecast

Con Accounting + Billing:

Historical Revenue
+
Contracted Revenue
+
Collections
+
AR
+
Deferred Revenue

se puede construir:

Financial Forecast

Esto será útil para:

26/21 — Billing Analytics & Revenue Intelligence

y:

26/22 — Billing FinOps & Cost Optimization.

111. End-to-End Example

EVOXA genera:

Subscription
$1,000

Tax:

$190

Invoice:

$1,190

Accounting:

DR Accounts Receivable
$1,190


CR Revenue
$1,000


CR Tax Payable
$190

Customer paga:

$1,190

Payment Accounting:

DR Cash
$1,190


CR Accounts Receivable
$1,190

Resultado:

AR = $0
Cash = +$1,190
Revenue = $1,000
Tax Payable = $190
112. End-to-End Annual Subscription

Contrato:

$12,000
12 months

Customer paga anticipadamente.

Payment
Cash
+$12,000
Deferred Revenue
Deferred Revenue
+$12,000
Month 1
DR Deferred Revenue
$1,000


CR Revenue
$1,000
Month 2
DR Deferred Revenue
$1,000


CR Revenue
$1,000

Y así sucesivamente.

113. End-to-End Credit

Invoice:

$1,190

Luego:

Credit
$200

Billing:

Balance
$990

Accounting puede producir:

DR Revenue Adjustment
$200


DR Tax Adjustment
$38


CR Accounts Receivable
$238

La estructura exacta dependerá de la naturaleza del crédito y del tratamiento fiscal.

114. End-to-End Refund

Payment:

$1,190

Refund:

$238

Debe relacionarse:

Original Payment
      ↓
Refund
      ↓
Accounting
      ↓
Cash

y, cuando corresponda:

Revenue Adjustment
Tax Adjustment
115. End-to-End Write-Off

Invoice:

$1,000

Payment:

$0

Write-off:

$1,000

Accounting:

DR Bad Debt Expense
$1,000


CR Accounts Receivable
$1,000

AR:

$0

pero Cash:

$0

Esto demuestra nuevamente:

Write-Off
≠
Payment
116. Financial Data Flow
                       CUSTOMER
                           │
                           ▼
                       CONTRACT
                           │
                           ▼
                      SUBSCRIPTION
                           │
                           ▼
                         USAGE
                           │
                           ▼
                         CHARGE
                           │
                           ▼
                        INVOICE
                           │
             ┌─────────────┼──────────────┐
             ▼             ▼              ▼
          PAYMENT        TAX          REVENUE
             │             │              │
             └─────────────┼──────────────┘
                           ▼
                     SUBLEDGERS
                           │
                           ▼
                   ACCOUNTING ENGINE
                           │
                           ▼
                    JOURNAL ENTRIES
                           │
                           ▼
                     GENERAL LEDGER
                           │
                           ▼
                         ERP
117. Core Principles
1. Billing Is Not Accounting

Billing genera eventos financieros; Accounting registra sus efectos.

2. Double Entry

Todo asiento debe cumplir:

Debits = Credits
3. Immutable History

Los asientos históricos no deben modificarse destructivamente.

4. Subledger First

Los detalles deben permanecer disponibles antes de agregarse al GL.

5. Reconciliation

Billing, Subledgers, GL y ERP deben poder reconciliarse.

6. Multi-Entity

Cada operación debe pertenecer a la Legal Entity correcta.

7. Multi-Currency

Debe conservarse moneda, tasa y monto funcional.

8. Period Control

Los períodos cerrados deben estar protegidos.

9. Idempotency

Una operación no debe contabilizarse dos veces.

10. Full Traceability

Debe poder recorrerse:

Business Transaction
→ Billing
→ Accounting
→ GL
→ ERP
118. Resultado de 26/18

Billing Accounting & Financial Integration convierte los eventos de Billing en información contable estructurada y conecta EVOXA con su ecosistema financiero.

La arquitectura queda:

Billing Event
     ↓
Financial Subledger
     ↓
Accounting Rule
     ↓
Journal Entry
     ↓
General Ledger
     ↓
ERP
     ↓
Financial Reporting

Y las principales relaciones:

Invoice
   ↓
Accounts Receivable


Payment
   ↓
Cash


Revenue Recognition
   ↓
Revenue


Prepaid Revenue
   ↓
Deferred Revenue


Tax
   ↓
Tax Payable


Write-Off
   ↓
Bad Debt Expense


Payment Fee
   ↓
Processing Expense
Arquitectura acumulada 26/08 → 26/18
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

Ahora tenemos una arquitectura financiera mucho más completa:

                       EVOXA BILLING
                            │
          ┌─────────────────┼─────────────────┐
          ▼                 ▼                 ▼
     COMMERCIAL          BILLING            FISCAL
          │                 │                 │
      Catalog            Invoices            Tax
      Pricing            Payments            Compliance
      Plans              Credits
      Discounts          Adjustments
          │                 │
          └─────────────────┼─────────────────┘
                            ▼
                       REVENUE
                            │
                            ▼
                     SUBLEDGERS
                            │
                            ▼
                      ACCOUNTING
                            │
                            ▼
                         GL / ERP

Billing Accounting & Financial Integration debe proporcionar una capa contable robusta, multi-entidad, multi-moneda, auditable e idempotente que transforme los eventos de Billing en subledgers y asientos contables balanceados, permita reconciliar Billing con el General Ledger y conecte EVOXA con ERPs y sistemas financieros externos sin acoplar el Billing Core a un proveedor contable específico.

Siguiente capítulo
26/19 — Billing Dunning & Collections

Aquí entraremos en la parte de cobranza: cuentas vencidas, AR Aging, Dunning Policies, Payment Retries, Collection Workflows, promesas de pago, suspensión de servicios, escalamiento, recuperación de deuda, Collection Agencies, Risk Scoring y estrategias automáticas de recuperación.
