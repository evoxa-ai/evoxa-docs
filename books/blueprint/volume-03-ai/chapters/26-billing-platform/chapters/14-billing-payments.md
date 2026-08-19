1. Billing Payments

Billing Payments es la capa responsable de administrar el dinero recibido como consecuencia de las obligaciones financieras generadas por las Invoices.

Si:

Invoice
= cuánto debe pagar el Customer

entonces:

Payment
= cuánto dinero fue recibido

La relación completa:

Customer
   ↓
Billing Account
   ↓
Subscription
   ↓
Charges
   ↓
Invoice
   ↓
Payment
   ↓
Payment Allocation
   ↓
Settlement
   ↓
Accounting / Revenue

El objetivo de Payments no es solamente "recibir dinero". Debe controlar todo el ciclo:

métodos de pago;
intentos;
autorizaciones;
capturas;
pagos;
fallos;
reintentos;
pagos parciales;
pagos excedentes;
aplicación a invoices;
conciliación;
refunds;
chargebacks;
settlement;
múltiples monedas;
múltiples proveedores;
trazabilidad;
auditoría.
2. Propósito

Billing Payments debe permitir:

registrar payments;
iniciar pagos;
autorizar pagos;
capturar pagos;
almacenar referencias externas;
asociar pagos a invoices;
aplicar pagos parcialmente;
aplicar pagos a múltiples invoices;
manejar pagos automáticos;
manejar pagos manuales;
manejar ACH/bank transfer;
tarjetas;
wallets;
otros métodos;
detectar pagos duplicados;
procesar fallos;
realizar retries;
conciliar settlements;
administrar refunds;
administrar chargebacks;
mantener payment ledger;
generar eventos;
integrarse con Accounting.
3. Invoice vs Payment

La separación conceptual:

Invoice
$1,000
   │
   │ obligación
   ▼
Customer
   │
   │ pago
   ▼
Payment
$1,000

Pero también:

Invoice
$1,000


Payment
$600

Resultado:

Amount Due
$1,000


Amount Paid
$600


Balance
$400
4. Payment Lifecycle

Un payment puede pasar por:

CREATED
   │
   ▼
PENDING
   │
   ▼
AUTHORIZED
   │
   ▼
CAPTURED
   │
   ▼
SETTLED

También:

PENDING
   │
   ├── FAILED
   ├── CANCELED
   └── EXPIRED

Y posteriormente:

SETTLED
   │
   ├── REFUNDED
   ├── PARTIALLY_REFUNDED
   └── CHARGEBACK
5. Payment Status

Estados principales:

Pending

El pago está siendo procesado.

Authorized

El proveedor autorizó el importe.

Captured

El importe fue capturado.

Settled

El dinero fue liquidado al merchant.

Failed

El intento falló.

Canceled

Fue cancelado.

Refunded

Fue devuelto.

Partially Refunded

Se devolvió una parte.

Charged Back

El pago fue revertido mediante chargeback.

6. Payment Attempt

Una Invoice puede tener múltiples intentos:

Invoice
$1,000
   │
   ├── Attempt 1 → Failed
   ├── Attempt 2 → Failed
   └── Attempt 3 → Success

Por eso:

Payment
≠
Payment Attempt

El Payment representa la operación financiera; los Attempts representan los intentos realizados para completarla.

7. Payment Intent

Para pagos modernos conviene separar:

Payment Intent
      │
      ├── Attempt 1
      ├── Attempt 2
      └── Attempt 3

El Payment Intent representa la intención de pagar una obligación.

Ejemplo:

Invoice
$1,000


Payment Intent
$1,000


Attempt 1
Failed


Attempt 2
Success
8. Payment Methods

EVOXA debe poder soportar:

Payment Methods
│
├── Credit Card
├── Debit Card
├── Bank Transfer
├── ACH
├── Wire Transfer
├── Digital Wallet
├── Direct Debit
├── Check
└── Manual Payment

La disponibilidad dependerá del país, proveedor y entidad legal.

9. Payment Method vs Payment

Un Payment Method es:

Cómo puede pagar el Customer.

Un Payment es:

Una operación concreta de dinero.

Ejemplo:

Payment Method
Visa ending 4242

Luego:

Payment
$1,000
10. Stored Payment Methods

Un Customer puede guardar varios métodos:

Customer
│
├── Visa
├── Mastercard
└── Bank Account

Debe existir uno principal:

Default Payment Method

pero la plataforma debe permitir fallback.

11. Payment Method Tokenization

Billing no debería almacenar directamente datos sensibles de tarjetas.

La arquitectura recomendada:

Customer
   │
   ▼
Payment Provider
   │
   ▼
Token
   │
   ▼
Billing Platform

Billing conserva una referencia/token, no el número completo de tarjeta.

12. Payment Provider

EVOXA debe abstraer al proveedor:

Payment Provider
│
├── Provider A
├── Provider B
└── Provider C

La arquitectura debería evitar:

Billing
   ↓
Hardcoded Provider

y utilizar:

Billing Payments
       ↓
Payment Provider Abstraction
       ↓
Provider Adapter
13. Payment Provider Adapter

Cada proveedor implementa una interfaz común:

PaymentProvider
│
├── createPayment()
├── authorize()
├── capture()
├── cancel()
├── refund()
├── getPayment()
└── handleWebhook()

Esto permite cambiar o agregar proveedores sin modificar Billing Core.

14. Payment Authorization

En determinados métodos:

Payment
   ↓
Authorization
   ↓
Approved

La autorización significa:

El proveedor aprobó la posibilidad de cobrar el importe.

Todavía puede faltar:

Capture
15. Payment Capture

Flujo:

Authorize
   ↓
Capture
   ↓
Settlement

Ejemplo:

Authorized
$1,000


Captured
$1,000

No todos los métodos de pago utilizan necesariamente este flujo de la misma forma.

16. Automatic Payments

Para subscriptions:

Billing Period
   ↓
Invoice
   ↓
Payment Intent
   ↓
Default Payment Method
   ↓
Payment

Esto permite:

Auto-Pay = TRUE
17. Manual Payments

Un Customer también puede pagar manualmente:

Invoice
   ↓
Customer Portal
   ↓
Pay Now
   ↓
Payment Provider

O mediante:

Bank Transfer

que posteriormente debe conciliarse.

18. Bank Transfer Payments

Los pagos bancarios son diferentes porque pueden existir sin referencia directa inmediata.

Bank
  ↓
Incoming Payment
  ↓
Reconciliation
  ↓
Customer
  ↓
Invoice

El sistema debe intentar identificar:

customer;
billing account;
invoice;
amount;
bank reference.
19. Payment Reconciliation

Una de las funciones críticas:

Bank Settlement
       │
       ▼
Payment Record
       │
       ▼
Invoice Allocation

Debe responder:

¿A qué invoice corresponde este dinero?

20. Payment Allocation

Un payment puede aplicarse a una invoice:

Payment
$1,000
   │
   ▼
Invoice
$1,000

Pero también:

Payment
$2,000
   │
   ├── Invoice A → $1,000
   └── Invoice B → $1,000

Por eso debe existir una entidad:

PaymentAllocation
21. Partial Payment Allocation
Payment
$600


Invoice
$1,000

Allocation:

Applied
$600


Remaining Invoice Balance
$400
22. Overpayment

Puede ocurrir:

Invoice
$1,000


Payment
$1,200

Resultado:

Invoice Balance
$0


Customer Credit
$200

El excedente no debería perderse.

23. Customer Credit Balance

El Customer puede tener saldo:

Customer Credit
$200

que posteriormente puede aplicarse:

Next Invoice
$1,000


Credit
-$200


Amount Due
$800

Esto conecta Payments con Credits & Adjustments.

24. Unapplied Payments

Un pago puede llegar sin poder determinar inmediatamente su destino:

Payment
$5,000


Invoice
Unknown

Debe quedar:

UNAPPLIED

hasta que se reconcilie.

Nunca debería aplicarse arbitrariamente.

25. Payment Allocation Rules

Cuando existe más de una invoice:

Payment
$2,000

puede aplicarse según:

Oldest Invoice First
Due Date
Explicit Customer Allocation
Invoice Reference
Business Rule

La regla debe ser configurable.

26. Payment Idempotency

Si el mismo request llega dos veces:

Payment Request
IDEMPOTENCY_KEY = ABC123

el sistema debe evitar:

Payment A
$1,000


Payment B
$1,000

y producir solamente:

Payment A
$1,000
27. External Payment Reference

Cada payment debe guardar referencias externas:

Payment
│
├── internal_id
├── payment_number
├── provider
├── provider_payment_id
├── provider_transaction_id
└── provider_reference

Esto permite reconciliación.

28. Payment Number

Debe existir un identificador interno:

PAY-2026-000001
PAY-2026-000002
PAY-2026-000003

Este número es independiente del identificador del proveedor.

29. Payment Currency

Un Payment debe tener:

Amount
1,000


Currency
USD

La moneda debe ser consistente con:

invoice;
payment method;
provider;
settlement.

Si existe conversión:

Original
USD 1,000


Settlement
CLP 950,000

debe conservarse la información de conversión.

30. Exchange Rate

Cuando se convierte moneda:

Amount
USD 1,000


Exchange Rate
950


Settlement
CLP 950,000

Debe registrarse:

rate;
source;
timestamp;
currency pair;
conversion direction.
31. Payment Fees

El proveedor puede cobrar:

Customer Payment
$1,000


Provider Fee
$30


Net Settlement
$970

La plataforma debe distinguir:

Customer Amount
$1,000


Payment Fee
$30


Net Settlement
$970

Esto es importante para FinOps y Accounting.

32. Gross vs Net Settlement

La diferencia:

Gross Payment
$10,000


Provider Fees
-$300


Net Settlement
$9,700

La Invoice continúa siendo:

$10,000

El fee no debe reducir arbitrariamente el amount paid de la invoice.

33. Payment Settlement

El Payment puede estar:

CAPTURED
   ↓
SETTLEMENT_PENDING
   ↓
SETTLED

La fecha de settlement puede ser distinta de la fecha de pago.

Ejemplo:

Payment Date
18 Aug


Settlement Date
20 Aug
34. Settlement Batch

Los proveedores pueden entregar batches:

Settlement Batch
│
├── Payment A
├── Payment B
├── Payment C
└── Payment D

El sistema debe conciliar:

Σ Payments
-
Σ Fees
=
Settlement Amount
35. Settlement Reconciliation

Ejemplo:

Expected Settlement
$9,700


Provider Settlement
$9,700


Variance
$0

Si existe diferencia:

Variance
$25

debe generarse una excepción.

36. Payment Failure

Los fallos pueden clasificarse:

Payment Failed
│
├── Insufficient Funds
├── Card Declined
├── Expired Method
├── Invalid Account
├── Provider Error
├── Authentication Required
├── Fraud Suspected
└── Unknown

La clasificación alimenta Dunning.

37. Retry Strategy

Un fallo no necesariamente significa cancelar inmediatamente.

Attempt 1
   ↓
Failed
   ↓
Retry
   ↓
Attempt 2
   ↓
Retry
   ↓
Attempt 3
   ↓
Success

La política debe ser configurable.

38. Smart Retry

El sistema puede seleccionar cuándo reintentar:

Payment Failure
      │
      ▼
Failure Classification
      │
      ▼
Retry Policy
      │
      ▼
Optimal Retry

Puede considerar:

tipo de fallo;
historial;
payment method;
customer;
provider;
timing.
39. Payment Recovery

La cadena:

Invoice
   ↓
Payment Failed
   ↓
Retry
   ↓
Payment Success
   ↓
Invoice Paid

Esto evita que una falla temporal genere churn innecesario.

40. Payment Authentication

Algunos métodos requieren autenticación adicional:

Payment
   ↓
Authentication Required
   ↓
Customer Authentication
   ↓
Approved
   ↓
Capture

El Payment State Machine debe soportar este estado.

41. Payment Webhooks

Los proveedores suelen enviar eventos:

payment.authorized
payment.captured
payment.failed
payment.refunded
payment.disputed
payment.settled

La plataforma debe:

Webhook
   ↓
Authenticate
   ↓
Validate
   ↓
Deduplicate
   ↓
Process
42. Webhook Idempotency

El mismo webhook puede llegar varias veces:

payment.captured
payment.captured
payment.captured

Debe procesarse una sola vez.

Provider Event ID
       ↓
Already Processed?
       ↓
Yes → Ignore
43. Payment Security

La arquitectura debe aplicar:

tokenization;
encryption;
access control;
audit logging;
secret management;
provider isolation;
least privilege.

Billing no debería almacenar datos de pago sensibles innecesarios.

44. Payment Compliance

Dependiendo del mercado pueden existir requisitos asociados a:

PCI DSS;
protección de datos;
autenticación;
regulación financiera;
facturación electrónica;
AML/KYC en determinados escenarios.

La arquitectura debe separar:

Payment Core
      +
Compliance Layer
45. Refunds

Un Payment puede devolverse:

Payment
$1,000
   ↓
Refund
$200

Resultado:

Refunded
$200


Remaining Captured
$800

El Refund debe ser una operación independiente.

46. Partial Refund

Puede ocurrir:

Payment
$2,000


Refund
$500


Remaining
$1,500

Debe existir trazabilidad:

Payment
   └── Refund
47. Full Refund
Payment
$1,000


Refund
$1,000


Payment Status
REFUNDED
48. Chargebacks

Un chargeback es diferente de un refund:

Refund
= iniciado por Merchant


Chargeback
= iniciado por Customer / Issuer

El flujo:

Payment
   ↓
Chargeback
   ↓
Dispute
   ↓
Evidence
   ↓
Decision

Esto será parte importante de la capa de Disputes.

49. Payment Ledger

Debe existir un ledger financiero de payments:

Payment Ledger
│
├── Payment
├── Allocation
├── Refund
├── Chargeback
├── Fee
├── Settlement
└── Adjustment

Esto permite reconstruir el estado financiero.

50. Payment Ledger Example
Payment
+$1,000


Provider Fee
-$30


Refund
-$100


Chargeback
-$200


Net
$670

Cada movimiento debe tener:

timestamp;
source;
reference;
currency;
actor/system;
status.
51. Payment Allocation Ledger

Ejemplo:

Payment
$2,000
   │
   ├── Invoice A
   │     $1,200
   │
   ├── Invoice B
   │     $500
   │
   └── Customer Credit
         $300

Esto es especialmente importante para Enterprise.

52. Payment and Subscription

Una subscription puede configurar:

Auto Pay
Enabled


Payment Method
Card ****4242

Cuando se genera la Invoice:

Invoice
   ↓
Auto Payment
   ↓
Payment Intent
   ↓
Payment
53. Payment and Dunning

Si falla:

Invoice
   ↓
Payment Failed
   ↓
Dunning
   ↓
Retry

La Subscription puede permanecer:

ACTIVE

mientras:

Invoice
PAST_DUE

según la política comercial.

54. Payment and Customer Credit

Si existe saldo positivo:

Customer Credit
$500

puede utilizarse automáticamente:

Invoice
$1,000


Credit
-$500


Payment Required
$500

Esto reduce transacciones externas.

55. Payment Priority

Cuando existen múltiples obligaciones:

Customer
│
├── Invoice A → $500
├── Invoice B → $800
└── Invoice C → $300

y recibe:

Payment
$1,000

el sistema necesita una regla de aplicación.

Por ejemplo:

Oldest Due First

Resultado:

Invoice A → $500
Invoice B → $500
Invoice C → $0
56. Payment APIs
Payment Methods
GET  /billing/payment-methods
POST /billing/payment-methods
DELETE /billing/payment-methods/{id}
Payments
POST /billing/payments
GET  /billing/payments
GET  /billing/payments/{id}
Payment Attempts
POST /billing/payments/{id}/retry
Allocation
POST /billing/payments/{id}/allocate
Refund
POST /billing/payments/{id}/refund
57. Payment Preview

Antes de ejecutar:

POST /billing/payments/preview

Puede devolver:

Invoice Total
$1,000


Credits
-$100


Payment Required
$900


Payment Method
Visa ****4242
58. Payment Data Model

Modelo conceptual:

Payment
│
├── id
├── payment_number
├── customer_id
├── billing_account_id
├── tenant_id
├── invoice_id
├── payment_method_id
├── provider_id
├── provider_payment_id
├── provider_transaction_id
├── amount
├── currency
├── status
├── payment_type
├── captured_at
├── settled_at
├── failure_code
├── failure_message
├── idempotency_key
├── metadata
├── created_at
└── updated_at
59. Payment Attempt Model
PaymentAttempt
│
├── id
├── payment_id
├── attempt_number
├── provider
├── provider_reference
├── amount
├── currency
├── status
├── failure_code
├── failure_message
├── initiated_at
├── completed_at
└── metadata
60. Payment Allocation Model
PaymentAllocation
│
├── id
├── payment_id
├── invoice_id
├── amount
├── currency
├── allocation_date
├── allocation_type
├── status
└── metadata
61. Payment Method Model
PaymentMethod
│
├── id
├── customer_id
├── type
├── provider
├── provider_token
├── last4
├── brand
├── expiration_month
├── expiration_year
├── is_default
├── status
└── metadata

Los campos exactos dependerán del método y proveedor.

62. Refund Model
Refund
│
├── id
├── refund_number
├── payment_id
├── invoice_id
├── amount
├── currency
├── reason
├── provider_refund_id
├── status
├── requested_by
├── approved_by
└── created_at
63. Settlement Model
Settlement
│
├── id
├── provider_id
├── settlement_number
├── settlement_date
├── currency
├── gross_amount
├── fees
├── adjustments
├── net_amount
├── status
└── provider_reference
64. Payment Reconciliation Architecture
                PAYMENT PROVIDER
                       │
                       ▼
                  Transactions
                       │
                       ▼
                  Settlements
                       │
                       ▼
                Reconciliation
                       │
          ┌────────────┼────────────┐
          ▼            ▼            ▼
       Payments     Fees       Variances
          │
          ▼
      Allocations
          │
          ▼
       Invoices
65. Payment State Machine
                  ┌──────────┐
                  │ CREATED  │
                  └────┬─────┘
                       ▼
                  ┌──────────┐
                  │ PENDING  │
                  └────┬─────┘
                       │
              ┌────────┼────────┐
              ▼        ▼        ▼
         AUTHORIZED  FAILED   CANCELED
              │
              ▼
           CAPTURED
              │
              ▼
           SETTLED
              │
        ┌─────┴──────┐
        ▼            ▼
     REFUNDED    CHARGEBACK
66. Payment Security Architecture
Customer
   │
   ▼
Payment UI
   │
   ▼
Payment Provider
   │
   ▼
Token
   │
   ▼
EVOXA Billing

EVOXA debe minimizar el alcance de datos sensibles y utilizar tokens/referencias proporcionados por los payment providers.

67. Payment Observability

Deben medirse:

payment success rate;
payment failure rate;
authorization rate;
capture rate;
settlement time;
retry success rate;
refund rate;
chargeback rate;
payment processing latency;
provider errors.

Ejemplo:

10,000 Attempts
│
├── 9,200 Success
├── 500 Retry
├── 200 Failed
└── 100 Pending
68. Payment Analytics
Collections
collected amount;
outstanding amount;
recovery rate.
Payment Performance
success rate;
failure rate;
retry recovery.
Provider
authorization rate;
fees;
settlement latency.
Customer
payment behavior;
preferred method;
failed payment frequency.
69. Payment Risk

La plataforma puede identificar:

Customer
   │
   ▼
Payment History
   │
   ▼
Risk Engine

Puede detectar:

múltiples fallos;
unusual payment behavior;
high refund frequency;
suspicious transactions;
repeated chargebacks.

Las decisiones de bloqueo deben estar gobernadas por políticas y controles apropiados.

70. AI Payment Intelligence

La IA puede ayudar a:

Payment History
      │
      ▼
AI
      │
 ┌────┼─────────────┐
 ▼    ▼             ▼
Retry  Risk       Forecast

Por ejemplo:

Este método de pago tiene alta probabilidad de fallar en el próximo intento.

O:

La mejor ventana para un retry probablemente sea posterior a la fecha habitual de ingreso de fondos del cliente.

La IA debe recomendar; las políticas financieras determinan la acción final.

71. Enterprise Payments

Para EVOXA Enterprise:

Corporate Customer
       │
       ▼
Billing Account
       │
       ├── Invoice A
       ├── Invoice B
       └── Invoice C
               │
               ▼
          Payment
          $50,000
               │
       ┌───────┼────────┐
       ▼       ▼        ▼
    Invoice A B        C
    $20K      $15K     $15K

Esto permite pagos consolidados.

72. Payment Approval

Para grandes montos:

Payment
$100,000
    │
    ▼
Approval Required
    │
    ▼
Finance
    │
    ▼
Approved
    │
    ▼
Execute

Las políticas pueden depender de:

monto;
customer;
currency;
country;
payment type;
risk.
73. Payment Governance

Debe existir separación de funciones:

Billing
→ Creates Payment


Finance
→ Approves exceptional operations


Treasury
→ Reconciles Settlement


Accounting
→ Posts Financial Entries


Auditor
→ Reviews
74. Payment Audit

Eventos:

PaymentCreated
PaymentAuthorized
PaymentCaptured
PaymentFailed
PaymentRetried
PaymentSettled
PaymentAllocated
PaymentUnallocated
PaymentRefunded
PaymentDisputed
PaymentReconciled

Cada evento debe contener:

Actor
Timestamp
Payment
Previous State
New State
Provider
Reference
Reason
Correlation ID
75. Payment and Accounting

La cadena:

Payment
   ↓
Settlement
   ↓
Bank
   ↓
Reconciliation
   ↓
Accounting

La contabilización exacta dependerá de la arquitectura financiera.

76. Payment and Revenue

Importante:

Payment
≠
Revenue

Ejemplo:

Annual Subscription
$12,000


Payment
$12,000 today

El revenue puede reconocerse:

$1,000/month

según las reglas contables.

77. Payment and Cash

También:

Invoice
$12,000


Payment
$12,000


Cash
$12,000

La plataforma debe distinguir claramente:

Billings
Receivables
Cash
Revenue
78. Payment Processing Architecture
                      CUSTOMER
                         │
                         ▼
                    PAYMENT UI
                         │
                         ▼
                  PAYMENT INTENT
                         │
                         ▼
                  PAYMENT ROUTER
                         │
            ┌────────────┼────────────┐
            ▼            ▼            ▼
        Provider A   Provider B   Provider C
            │            │            │
            └────────────┼────────────┘
                         ▼
                      Payment
                         │
                         ▼
                    Allocation
                         │
                         ▼
                      Invoice
                         │
                         ▼
                     Settlement
                         │
                         ▼
                    Accounting
79. Payment Router

En una arquitectura Enterprise puede existir un Payment Router:

Payment Request
      │
      ▼
Payment Router
      │
 ┌────┼──────────┐
 ▼    ▼          ▼
A     B          C

La selección puede considerar:

país;
moneda;
método;
costo;
disponibilidad;
success rate;
customer;
legal entity.
80. Payment Failover

Si un proveedor está caído:

Provider A
   │
   ▼
Unavailable
   │
   ▼
Provider B
   │
   ▼
Payment

Esto mejora resiliencia.

No todos los payments pueden trasladarse automáticamente entre proveedores; la estrategia debe respetar las características del método de pago y la operación original.

81. Payment Reconciliation End-to-End
Invoice
$10,000
   │
   ▼
Payment
$10,000
   │
   ▼
Provider
$10,000
   │
   ▼
Fee
-$300
   │
   ▼
Settlement
$9,700
   │
   ▼
Bank
$9,700

El sistema debe poder demostrar cada paso.

82. Payment Exception Management

Las excepciones pueden incluir:

Payment without Invoice
Invoice without Payment
Duplicate Payment
Settlement Variance
Unknown Bank Transfer
Provider Mismatch
Currency Mismatch
Fee Mismatch
Chargeback
Refund Mismatch

Cada excepción debe tener workflow.

83. Payment Exception Workflow
Exception
   │
   ▼
Detected
   │
   ▼
Assigned
   │
   ▼
Investigating
   │
   ▼
Resolved
   │
   ▼
Reconciled

Esto es importante para operaciones financieras Enterprise.

84. Payment Data Flow
Invoice
   │
   ▼
Payment Intent
   │
   ▼
Payment Attempt
   │
   ▼
Provider
   │
   ▼
Payment
   │
   ▼
Allocation
   │
   ▼
Invoice Balance
   │
   ▼
Settlement
   │
   ▼
Accounting
85. EVOXA Payment Example
ACME Corporation


Invoice
INV-2026-00125


Total
$10,000 USD


Payment Method
Corporate Card


Payment Intent
$10,000


Attempt #1
FAILED


Attempt #2
SUCCESS


Payment
$10,000


Provider Fee
$300


Settlement
$9,700


Invoice Balance
$0


Invoice Status
PAID

La trazabilidad completa:

Invoice
   ↓
Payment Intent
   ↓
Attempt
   ↓
Provider Transaction
   ↓
Payment
   ↓
Allocation
   ↓
Settlement
86. Core Principles
1. Payment ≠ Invoice

La Invoice crea la obligación; el Payment registra dinero recibido.

2. Payment ≠ Settlement

El Payment puede existir antes de que el dinero sea liquidado.

3. Payment ≠ Revenue

Recibir dinero no determina automáticamente revenue recognition.

4. Idempotency

Un mismo pago nunca debe registrarse dos veces.

5. Tokenization

Los datos sensibles deben mantenerse fuera del Billing Core siempre que sea posible.

6. Provider Abstraction

Billing no debe depender directamente de un único proveedor.

7. Allocation Traceability

Cada pago debe poder rastrearse hasta las invoices a las que fue aplicado.

8. Immutable Financial History

Refunds y corrections deben utilizar operaciones financieras explícitas.

9. Reconciliation

Payments y settlements deben poder conciliarse.

10. Enterprise Governance

Los pagos críticos deben estar sujetos a controles, aprobación y auditoría.

87. Resultado de 26/14

Billing Payments convierte las obligaciones generadas por Invoicing en operaciones financieras reales, administrando desde la intención de pago hasta la captura, aplicación, liquidación y conciliación.

La cadena queda:

Subscription
     ↓
Charges
     ↓
Invoice
     ↓
Payment Intent
     ↓
Payment Attempt
     ↓
Payment
     ↓
Payment Allocation
     ↓
Settlement
     ↓
Accounting

Y cuando existen problemas:

Payment Failed
     ↓
Retry
     ↓
Dunning
     ↓
Recovery


Payment
     ↓
Refund


Payment
     ↓
Chargeback
     ↓
Dispute
Arquitectura acumulada de Billing
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
...

Y el principio fundamental es:

Billing Payments debe administrar de extremo a extremo la conversión de obligaciones facturadas en dinero recibido, manteniendo una separación clara entre Payment Intent, Payment Attempt, Payment, Allocation, Refund, Chargeback y Settlement, con idempotencia, seguridad, conciliación, trazabilidad y capacidad Enterprise multi-tenant y multi-provider.

26/15 — Billing Credits, Discounts & Adjustments

será la siguiente capa, encargada de administrar créditos comerciales, descuentos, notas de crédito, débitos, ajustes, compensaciones y correcciones financieras, sin alterar destructivamente el historial de invoices o payments.
