1. Billing Taxes & Compliance

Billing Taxes & Compliance es la capa responsable de determinar, calcular, documentar y controlar las obligaciones tributarias asociadas a las operaciones de Billing.

Esta capa debe permanecer separada de:

Pricing;
Rating;
Invoicing;
Payments;
Revenue Recognition;
Accounting.

La cadena completa:

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
Discounts / Credits / Adjustments
   ↓
Tax Determination
   ↓
Tax Calculation
   ↓
Invoice
   ↓
Payment
   ↓
Settlement
   ↓
Accounting

La pregunta que responde esta capa es:

¿Qué impuestos corresponden a esta operación, quién debe pagarlos, dónde se generan y cómo deben documentarse y reportarse?

2. Propósito

El módulo debe permitir:

determinar jurisdicción fiscal;
identificar entidad legal;
identificar tax profile;
almacenar tax IDs;
clasificar productos fiscalmente;
determinar tasas;
calcular impuestos;
manejar exenciones;
manejar reverse charge;
manejar múltiples jurisdicciones;
manejar múltiples países;
soportar IVA/VAT/GST y otros impuestos;
generar información fiscal para invoices;
soportar documentos fiscales;
controlar compliance;
mantener evidencia;
generar reportes;
integrar con sistemas contables;
auditar cambios;
versionar reglas fiscales.
3. Tax vs Pricing

Es fundamental separar:

Pricing
= cuánto cuesta el producto

de:

Tax
= cuánto impuesto corresponde

Ejemplo:

Product Price
$1,000


Discount
-$100


Taxable Base
$900


Tax
19%


Tax Amount
$171


Invoice Total
$1,071
4. Tax vs Invoice

La Invoice presenta los impuestos, pero el Tax Engine determina cómo se calcularon.

Tax Engine
   ↓
Tax Result
   ↓
Invoice

La Invoice no debería contener toda la lógica fiscal.

5. Tax Architecture
                    TAX ENGINE
                        │
        ┌───────────────┼───────────────┐
        ▼               ▼               ▼
   Jurisdiction      Tax Rules       Tax Rates
        │               │               │
        └───────────────┼───────────────┘
                        ▼
                  Tax Determination
                        │
                        ▼
                   Tax Calculation
                        │
                        ▼
                    Tax Result
                        │
                        ▼
                     Invoice
6. Tax Determination

El primer paso no es calcular.

Primero hay que determinar:

Is this transaction taxable?

y:

Which tax rules apply?

Para eso se necesita:

seller location;
customer location;
product;
service type;
customer tax status;
transaction date;
legal entity;
jurisdiction.
7. Tax Jurisdiction

Una operación puede estar asociada a:

Country
   ↓
State / Region
   ↓
Province
   ↓
Municipality
   ↓
Special Tax Jurisdiction

No todos los países utilizan el mismo modelo.

Por eso EVOXA debe tener un modelo de jurisdicciones configurable.

8. Tax Nexus

La plataforma debe poder determinar dónde la empresa tiene obligación fiscal.

Conceptualmente:

Legal Entity
       │
       ▼
Tax Nexus
       │
 ┌─────┼─────┐
 ▼     ▼     ▼
Chile  USA   Mexico

Esto afecta:

obligación de cobrar impuestos;
reporting;
tax registration;
tax rates.
9. Legal Entity

En Enterprise:

EVOXA Group
     │
     ├── EVOXA Chile
     ├── EVOXA USA
     └── EVOXA Mexico

Cada entidad puede tener:

Tax IDs
Tax Registrations
Tax Jurisdictions
Tax Rules

La Invoice debe utilizar la entidad correcta.

10. Tax Profile

Cada Customer puede tener un perfil fiscal:

Tax Profile
│
├── Tax Country
├── Tax Region
├── Tax ID
├── Tax Type
├── Tax Exempt
├── Exemption Reason
└── Tax Treatment

Ejemplo:

Customer
ACME


Country
Chile


Tax ID
XXXXXXXX-X


Tax Treatment
STANDARD
11. Tax Identification Number

El sistema debe soportar diferentes identificadores:

Chile
RUT


USA
EIN


EU
VAT ID


Mexico
RFC

El modelo debe ser genérico:

TaxIdentifier
│
├── type
├── country
├── value
├── validation_status
└── verified_at
12. Tax ID Validation

Cuando sea posible:

Tax ID
   ↓
Validation Service
   ↓
Valid / Invalid

Debe almacenarse:

Validation Status
Validation Source
Validation Date
13. Product Tax Classification

Cada Product debe tener clasificación fiscal:

Product
   │
   ▼
Tax Category

Ejemplos conceptuales:

DIGITAL_SERVICE
SOFTWARE
DATA_SERVICE
CONSULTING
ADVERTISING
AI_SERVICE
STORAGE

La clasificación exacta depende de la jurisdicción.

14. Tax Category

El modelo:

TaxCategory
│
├── code
├── name
├── country
├── tax_treatment
└── metadata

Esto desacopla Products del sistema fiscal.

15. Tax Treatment

Un producto puede tener:

STANDARD
ZERO_RATED
EXEMPT
OUT_OF_SCOPE
REVERSE_CHARGE
SPECIAL_RATE

No deben confundirse:

0%
≠
EXEMPT
≠
OUT_OF_SCOPE
16. Tax Rate

Una tasa debe estar versionada:

Tax Rate
19%


Effective From
01/01/2026


Effective To
31/12/2026

El Rating/Tax Engine debe usar la tasa vigente para la fecha de la operación.

17. Effective Tax Date

Debe definirse qué fecha determina la tasa:

Transaction Date
Invoice Date
Supply Date
Payment Date

Esto depende de la legislación aplicable.

La regla debe ser explícita y configurable.

18. Tax Calculation

Una fórmula simple:

Taxable Base × Tax Rate
=
Tax Amount

Ejemplo:

Taxable Base
$1,000


Rate
19%


Tax
$190
19. Taxable Base

La base puede ser:

Gross Amount
-
Eligible Discounts
-
Eligible Credits
+
Taxable Adjustments
=
Taxable Base

Pero el tratamiento exacto depende de la naturaleza de cada operación.

20. Discount and Tax

Ejemplo:

Gross
$1,000


Discount
-$100


Taxable Base
$900


Tax 19%
$171

Por eso el Tax Engine debe recibir información suficiente sobre el origen del descuento.

21. Credit and Tax

Los créditos pueden tener tratamientos diferentes.

Ejemplo conceptual:

Original Taxable Base
$1,000


Credit
-$200


Adjusted Base
$800

Pero si el crédito ya fue facturado y tributado, puede requerir:

Credit Note
+
Tax Adjustment

La regla depende de la jurisdicción.

22. Tax Lines

La Invoice puede contener:

Tax Lines
│
├── Tax Type
├── Jurisdiction
├── Rate
├── Taxable Base
└── Tax Amount

Ejemplo:

IVA
Base: $1,000
Rate: 19%
Tax: $190
23. Multiple Taxes

Una operación puede tener múltiples impuestos:

Tax
│
├── VAT
├── Local Tax
└── Special Tax

La plataforma debe soportar composición de impuestos cuando corresponda.

24. Tax-on-Tax

Algunas jurisdicciones pueden tener impuestos calculados sobre bases que incluyen otros impuestos.

El motor debe permitir reglas:

Tax A
   ↓
Tax B

sin asumir que todos los impuestos son independientes.

25. Tax Exemptions

Un Customer puede tener exención:

Tax Exempt
YES

Debe conservarse:

Exemption Type
Certificate
Effective Date
Expiration Date
Jurisdiction
26. Tax Exemption Certificate

Conceptualmente:

Customer
   ↓
Exemption Certificate
   ↓
Validation
   ↓
Tax Treatment

Si está vencido:

Certificate
EXPIRED

la exención no debería aplicarse automáticamente.

27. Reverse Charge

En determinadas operaciones B2B internacionales:

Seller
   ↓
Customer
   ↓
Reverse Charge

El proveedor puede no cobrar el impuesto directamente, dependiendo de las reglas aplicables.

La Invoice debe mostrar el tratamiento correspondiente cuando sea requerido.

28. B2B vs B2C

El tratamiento fiscal puede depender de:

Customer Type
│
├── B2B
├── B2C
├── Government
└── Non-Profit

También puede depender de:

tax ID;
location;
product;
transaction type.
29. Customer Location

Para servicios digitales puede ser necesario determinar ubicación mediante múltiples señales:

Billing Address
+
Tax Address
+
Business Registration
+
Other Valid Evidence

La plataforma debe almacenar la fuente de la determinación.

30. Tax Evidence

En operaciones internacionales:

Tax Determination
      │
      ▼
Evidence
      │
 ┌────┼─────┐
 ▼    ▼     ▼
Address Tax ID Country

Esto permite auditoría.

31. Tax Address

Debe existir un concepto explícito:

Tax Address

que puede ser diferente de:

Shipping Address
Billing Address
Legal Address
32. Tax Rules

Una regla puede ser:

IF
Customer Country = CL
AND
Product Category = DIGITAL_SERVICE


THEN
Tax = IVA 19%

El sistema debe permitir reglas mucho más complejas sin codificarlas directamente dentro de Invoice.

33. Tax Rules Engine
Transaction
    │
    ▼
Customer Profile
    │
    ▼
Product Tax Category
    │
    ▼
Legal Entity
    │
    ▼
Jurisdiction
    │
    ▼
Tax Rule
    │
    ▼
Tax Rate
    │
    ▼
Tax Result
34. Tax Determination Result

Debe producir:

TaxDeterminationResult
│
├── taxable
├── jurisdiction
├── tax_category
├── tax_treatment
├── tax_rate
├── reason
├── evidence
└── rule_version
35. Tax Calculation Result
TaxCalculationResult
│
├── taxable_amount
├── tax_rate
├── tax_amount
├── currency
├── jurisdiction
├── tax_type
├── calculation_method
└── rule_version
36. Tax Engine Versioning

Las reglas deben ser versionadas:

Tax Rule v1
       ↓
Tax Rule v2
       ↓
Tax Rule v3

Una Invoice histórica debe conservar:

Tax Rule Version
Tax Rate Version
37. Tax Configuration

El sistema debe administrar:

Countries
Jurisdictions
Tax Categories
Tax Rates
Tax Rules
Tax Registrations
Tax Exemptions
Tax Treatments
38. Tax Registration

Una Legal Entity puede registrarse:

Legal Entity
   │
   ├── Chile
   │     └── Tax Registration
   │
   ├── USA
   │     └── Tax Registration
   │
   └── Mexico
         └── Tax Registration

Esto determina dónde puede/debe operar fiscalmente.

39. Tax Registration Model
TaxRegistration
│
├── legal_entity_id
├── country
├── jurisdiction
├── registration_type
├── registration_number
├── effective_from
├── effective_to
└── status
40. Fiscal Documents

Dependiendo de la jurisdicción:

Invoice
Credit Note
Debit Note
Receipt
Tax Invoice
Electronic Invoice

El Billing Platform debe permitir diferentes tipos documentales.

41. Electronic Invoicing

En países donde existe facturación electrónica:

Invoice
   ↓
Fiscal Validation
   ↓
Electronic Tax Document
   ↓
Tax Authority / Certified Provider
   ↓
Accepted

El sistema debe manejar:

SUBMITTED
ACCEPTED
REJECTED
CANCELED
42. Fiscal Document Number

Puede existir una numeración fiscal diferente de la Invoice interna:

Internal Invoice
INV-2026-000125


Fiscal Document
Folio 123456

Deben mantenerse ambos identificadores.

43. Fiscal Authority Reference

Una respuesta fiscal puede devolver:

Authority
Reference
Timestamp
Status

Estos datos deben almacenarse como evidencia.

44. Chile Considerations

Para operaciones en Chile, el modelo debe poder representar información fiscal propia del país, como:

RUT;
IVA;
documentos tributarios electrónicos;
folios;
información de emisor;
información del receptor;
estados de aceptación/rechazo;
referencias tributarias.

La implementación concreta debe validarse contra las exigencias vigentes del Servicio de Impuestos Internos (SII) y no debe quedar codificada de forma rígida en el Billing Core.

45. Multi-Country Architecture

EVOXA debe evitar:

if country == Chile:
   ...
elif country == USA:
   ...
elif country == Mexico:
   ...

como arquitectura principal.

Debe utilizar:

Tax Country Configuration
        ↓
Tax Rules
        ↓
Tax Provider / Tax Engine

Esto permite escalar.

46. Tax Provider Abstraction

Puede integrarse con proveedores externos:

Tax Engine
│
├── Internal Rules
├── Provider A
├── Provider B
└── Provider C

La arquitectura:

Billing
   ↓
Tax Provider Interface
   ↓
Provider Adapter
47. Tax Provider Response

El proveedor puede devolver:

Taxable
YES


Jurisdiction
CL


Rate
19%


Tax
$190

EVOXA debe almacenar la respuesta relevante como evidencia.

48. Tax Caching

Para operaciones de alto volumen puede existir caching:

Same Context
      ↓
Cached Tax Result

Pero la cache debe respetar:

tax rule version;
effective date;
jurisdiction;
product;
customer status.
49. Tax Calculation Precision

El cálculo fiscal requiere precisión:

Taxable Base
$123.4567


Tax Rate
19%


Tax
$23.456773

Luego se aplica la política de redondeo correspondiente.

Debe evitarse redondear prematuramente.

50. Tax Rounding

Debe existir una configuración:

ROUND_HALF_UP
ROUND_HALF_EVEN
LINE_LEVEL
INVOICE_LEVEL

La política puede variar según jurisdicción.

51. Tax on Invoice Lines

Puede calcularse:

Line A
$100 → Tax $19


Line B
$200 → Tax $38

Total:

Tax
$57

O puede existir una metodología de cálculo a nivel de documento.

La configuración debe ser explícita.

52. Tax Adjustments

Si cambia una invoice:

Original Tax
$190

y existe una credit note:

Tax Adjustment
-$38

debe existir trazabilidad hacia el documento original.

53. Tax on Refunds

Un refund puede implicar reversión fiscal:

Original
$1,000
Tax
$190


Refund
$500
Tax Reversal
$95

La lógica depende del tipo de transacción y jurisdicción.

54. Tax Exemption Lifecycle
Submitted
   ↓
Validated
   ↓
Approved
   ↓
Active
   ↓
Expires

Cada etapa debe quedar registrada.

55. Tax Compliance Calendar

El sistema puede gestionar:

Tax Filing
│
├── Period
├── Jurisdiction
├── Due Date
├── Status
├── Amount
└── Filing Reference

Esto puede alimentar workflows de Finanzas.

56. Tax Reporting

Reportes:

taxable sales;
tax collected;
tax payable;
tax exempt sales;
reverse charge;
tax by jurisdiction;
tax by product;
tax by customer;
tax adjustments.
57. Tax Liability

El sistema puede calcular:

Tax Collected
-
Tax Adjustments
-
Tax Credits
=
Tax Liability

El saldo luego se integra con Accounting.

58. Tax Reconciliation

Debe existir:

Invoice Tax
      │
      ▼
Tax Ledger
      │
      ▼
Tax Report
      │
      ▼
Accounting

La suma debe cuadrar.

59. Tax Ledger

Puede existir:

Tax Ledger
│
├── Tax Charged
├── Tax Adjusted
├── Tax Credited
├── Tax Refunded
├── Tax Payable
└── Tax Settled

Esto permite reconstrucción fiscal.

60. Tax Audit Trail

Cada cálculo debe poder responder:

¿Por qué se cobró este impuesto?

Ejemplo:

Invoice
$1,000


Taxable Base
$1,000


Jurisdiction
Chile


Tax Category
Digital Service


Tax Rule
CL-DIGITAL-001


Rate
19%


Tax
$190
61. Tax Compliance Evidence

Debe conservarse evidencia suficiente:

Customer Address
Tax ID
Tax Certificate
Tax Rule
Tax Rate
Jurisdiction
Provider Response
Fiscal Authority Response

La retención debe seguir las políticas legales aplicables.

62. Tax Risk

El sistema puede detectar:

Invalid Tax ID
Expired Exemption
Missing Tax Registration
Wrong Jurisdiction
Unexpected Tax Rate
Tax Calculation Variance

Esto genera:

Compliance Exception
63. Compliance Workflow
Tax Exception
      │
      ▼
Detected
      │
      ▼
Assigned
      │
      ▼
Reviewed
      │
      ▼
Corrected
      │
      ▼
Validated
64. Tax Approvals

Operaciones sensibles pueden requerir aprobación:

Tax Exemption
      ↓
Review
      ↓
Tax Team
      ↓
Approve
65. Tax Permissions

Permisos sugeridos:

tax.view
tax.configure
tax.rates.manage
tax.rules.manage
tax.exemptions.manage
tax.registrations.manage
tax.reports.view
tax.compliance.manage

Y separados por:

Organization
Tenant
Legal Entity
Country
66. Tax API
Tax Calculation
POST /billing/tax/calculate
Tax Preview
POST /billing/tax/preview
Tax Profiles
GET  /billing/tax-profiles
POST /billing/tax-profiles
PATCH /billing/tax-profiles/{id}
Tax Exemptions
GET  /billing/tax-exemptions
POST /billing/tax-exemptions
Tax Rates
GET /billing/tax-rates
67. Tax Data Model
TaxProfile
│
├── id
├── customer_id
├── tax_country
├── tax_region
├── tax_status
├── tax_id
├── exemption_status
├── exemption_reference
├── effective_from
├── effective_to
└── metadata
68. Tax Rate Model
TaxRate
│
├── id
├── country
├── jurisdiction
├── tax_type
├── rate
├── tax_category
├── effective_from
├── effective_to
├── status
└── metadata
69. Tax Rule Model
TaxRule
│
├── id
├── rule_code
├── country
├── jurisdiction
├── priority
├── conditions
├── tax_treatment
├── rate_reference
├── effective_from
├── effective_to
├── version
└── status
70. Tax Result Model
TaxResult
│
├── id
├── transaction_id
├── invoice_id
├── invoice_line_id
├── tax_type
├── jurisdiction
├── tax_category
├── taxable_amount
├── rate
├── tax_amount
├── currency
├── rule_version
├── calculation_timestamp
└── metadata
71. Tax Document Model
FiscalDocument
│
├── id
├── invoice_id
├── document_type
├── fiscal_number
├── authority_reference
├── status
├── submitted_at
├── accepted_at
├── rejected_at
├── cancellation_reference
└── metadata
72. Tax Architecture for EVOXA
                         EVOXA BILLING
                              │
                              ▼
                         Invoice Data
                              │
                              ▼
                         TAX ENGINE
                              │
             ┌────────────────┼────────────────┐
             ▼                ▼                ▼
         Customer          Product         Legal Entity
         Tax Profile       Tax Category    Registration
             │                │                │
             └────────────────┼────────────────┘
                              ▼
                         Jurisdiction
                              │
                              ▼
                           Tax Rules
                              │
                              ▼
                          Tax Rates
                              │
                              ▼
                       Tax Calculation
                              │
                              ▼
                         Tax Result
                              │
             ┌────────────────┼────────────────┐
             ▼                ▼                ▼
          Invoice         Fiscal Doc       Tax Ledger
73. Multi-Country EVOXA

La plataforma puede evolucionar:

EVOXA
│
├── Chile
│   ├── IVA
│   └── DTE
│
├── USA
│   ├── Sales Tax
│   └── State Rules
│
├── Mexico
│   ├── IVA
│   └── CFDI
│
└── EU
    ├── VAT
    └── OSS / Other Rules

La arquitectura central no debería cambiar.

Solo las configuraciones y adapters.

74. Tax Provider Architecture
                 EVOXA TAX SERVICE
                         │
              ┌──────────┴──────────┐
              ▼                     ▼
        Internal Rules          External Provider
              │                     │
              └──────────┬──────────┘
                         ▼
                   Tax Decision
                         │
                         ▼
                    Tax Result

Esto permite comenzar con reglas propias y posteriormente incorporar proveedores especializados.

75. Tax and Billing Close

El cierre debe seguir:

Usage Finalized
      ↓
Rating Finalized
      ↓
Charges Finalized
      ↓
Discounts / Credits
      ↓
Tax Determination
      ↓
Tax Calculation
      ↓
Invoice
      ↓
Fiscal Validation
      ↓
Issue

No debe emitirse una invoice fiscalmente válida si faltan datos requeridos.

76. Tax and Accounting

La cadena:

Invoice
   ↓
Tax Calculation
   ↓
Tax Liability
   ↓
Accounting

Ejemplo conceptual:

Accounts Receivable
$1,190


Revenue
$1,000


Tax Payable
$190

La estructura contable exacta se implementará en:

26/18 — Billing Accounting & Financial Integration.

77. Tax and Revenue

Es importante:

Tax
≠
Revenue

El IVA/VAT recaudado normalmente representa una obligación frente a la autoridad fiscal, no revenue de la empresa.

78. Tax and Payments

Payment:

$1,190

puede cubrir:

Revenue
$1,000


Tax
$190

Payments debe conservar el total recibido, mientras Accounting distribuye correctamente las partes.

79. Tax Analytics

KPIs:

Tax
tax collected;
tax payable;
effective tax rate;
tax by jurisdiction;
tax by product;
tax by customer.
Compliance
exemptions;
invalid tax IDs;
rejected documents;
filing status;
compliance exceptions.
80. Effective Tax Rate

Puede calcularse:

Total Tax
÷
Taxable Revenue
=
Effective Tax Rate

Esto ayuda a Finanzas a detectar anomalías.

81. Tax Anomaly Detection

Ejemplo:

Expected Tax
$190


Calculated
$1,900


Variance
+900%

El sistema puede generar:

Tax Anomaly

antes de emitir la invoice.

82. Tax Compliance Dashboard

EVOXA puede mostrar:

Tax Compliance
│
├── Active Registrations
├── Expiring Registrations
├── Tax Exemptions
├── Invalid Tax IDs
├── Pending Fiscal Documents
├── Rejected Documents
├── Tax Liability
└── Filing Deadlines
83. Tax Audit Dashboard

Puede permitir:

Invoice
   ↓
Tax Result
   ↓
Tax Rule
   ↓
Tax Rate
   ↓
Customer Tax Profile
   ↓
Evidence

Esto permite auditorías mucho más rápidas.

84. Tax Event Model

Eventos:

TaxCalculated
TaxAdjusted
TaxExemptionCreated
TaxExemptionExpired
TaxIdValidated
TaxDocumentSubmitted
TaxDocumentAccepted
TaxDocumentRejected
TaxDocumentCanceled
TaxRuleChanged
TaxRateChanged
85. Tax Audit Model
TaxAuditEvent
│
├── event_id
├── entity_type
├── entity_id
├── action
├── previous_value
├── new_value
├── actor
├── timestamp
├── jurisdiction
└── metadata
86. Tax Compliance Automation

Se pueden automatizar:

Tax ID Expiring
      ↓
Alert


Exemption Expiring
      ↓
Alert


Tax Rate Change
      ↓
Review


Fiscal Document Rejected
      ↓
Exception Workflow

Esto evita errores operativos.

87. Tax Configuration Governance

Los cambios de tasas y reglas deben pasar por:

Draft
  ↓
Review
  ↓
Approval
  ↓
Effective

No deberían modificarse directamente en producción sin control.

88. Tax Rate Change Example

Supongamos:

Old Rate
19%


New Rate
20%

Debe configurarse:

19%
Effective To
31/12/2026


20%
Effective From
01/01/2027

Nunca:

UPDATE rate = 20%

porque destruiría el histórico.

89. Tax Historical Integrity

Una invoice de 2026 debe seguir mostrando:

Tax Rate
19%

aunque en 2027 la tasa sea:

20%

Por eso:

Tax Rules
+
Tax Rates
+
Effective Dates
+
Versioning

son esenciales.

90. Core Principles
1. Tax Is Separate From Pricing

El precio y el impuesto tienen responsabilidades diferentes.

2. Jurisdiction First

Primero determinar dónde aplica el impuesto.

3. Tax Classification Matters

El tratamiento depende del tipo de producto/servicio.

4. Version Everything

Reglas y tasas deben estar versionadas.

5. Preserve Historical Tax Results

Una Invoice histórica no debe cambiar porque cambió una tasa.

6. Evidence Matters

Debe poder demostrarse por qué se aplicó determinado impuesto.

7. Compliance Is Continuous

No se trata solamente de calcular impuestos, sino también de cumplir obligaciones.

8. Multi-Country From the Beginning

La arquitectura debe poder crecer internacionalmente.

9. Separate Tax From Accounting

Tax determina obligaciones fiscales; Accounting registra los efectos financieros.

10. Auditability

Toda decisión fiscal debe poder reconstruirse.

91. Resultado de 26/16

Billing Taxes & Compliance establece la capa fiscal de EVOXA, permitiendo determinar qué impuestos corresponden, dónde aplican, a qué producto/cliente afectan, qué tasa debe utilizarse y cómo debe documentarse la operación.

La cadena queda:

Customer Tax Profile
        +
Product Tax Category
        +
Legal Entity
        +
Tax Registration
        +
Transaction
        ↓
Tax Jurisdiction
        ↓
Tax Rule
        ↓
Tax Rate
        ↓
Tax Determination
        ↓
Tax Calculation
        ↓
Tax Result
        ↓
Fiscal Document
        ↓
Tax Reporting
        ↓
Accounting

Y la arquitectura acumulada:

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

Billing Taxes & Compliance debe proporcionar una capa fiscal independiente, configurable, versionada y auditable que determine jurisdicción, tratamiento tributario, tasas, exenciones, documentos fiscales y obligaciones de compliance, permitiendo que EVOXA opere de forma consistente desde Chile hacia múltiples jurisdicciones sin contaminar el Billing Core con reglas fiscales específicas de cada país.

Arquitectura financiera acumulada
                    BILLING PLATFORM
                           │
                           ▼
                  COMMERCIAL LAYER
                           │
          ┌────────────────┼────────────────┐
          ▼                ▼                ▼
       Catalog          Pricing       Subscriptions
          │                │                │
          └────────────────┼────────────────┘
                           ▼
                    USAGE & METERING
                           │
                           ▼
                 RATING & CHARGES
                           │
              ┌────────────┼────────────┐
              ▼            ▼            ▼
          Discounts      Credits    Adjustments
              │            │            │
              └────────────┼────────────┘
                           ▼
                       TAX ENGINE
                           │
                           ▼
                        INVOICE
                           │
                           ▼
                        PAYMENT
                           │
                           ▼
                      SETTLEMENT
                           │
                           ▼
                       ACCOUNTING
                           │
                           ▼
                         REVENUE

El siguiente capítulo será:

26/17 — Billing Revenue Management

donde pasaremos de "qué se facturó y qué impuestos se cobraron" a "cómo EVOXA reconoce, administra, pronostica y analiza el Revenue generado por esas operaciones".
