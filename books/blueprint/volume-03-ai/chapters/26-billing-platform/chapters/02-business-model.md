1. Billing Business Model

El Billing Business Model define cómo una plataforma Enterprise transforma sus productos, servicios, funcionalidades y consumo en obligaciones financieras, ingresos y relaciones comerciales sostenibles.

Mientras 26/01 — Billing Overview establece qué es Billing y cuál es su función dentro de la plataforma, esta sección define cómo funciona económicamente Billing.

Billing debe ser capaz de soportar múltiples modelos de monetización simultáneamente, permitiendo que cada producto, organización, tenant o cliente utilice una estrategia comercial diferente.

2. Billing como modelo económico

El modelo puede representarse de la siguiente manera:

                         VALUE
                           │
                           ▼
                      PRODUCT / SERVICE
                           │
                           ▼
                    COMMERCIAL MODEL
                           │
             ┌─────────────┼─────────────┐
             │             │             │
          Pricing      Subscription     Usage
             │             │             │
             └─────────────┼─────────────┘
                           ▼
                      BILLING ENGINE
                           │
                           ▼
                         CHARGE
                           │
                           ▼
                        INVOICE
                           │
                           ▼
                        PAYMENT
                           │
                           ▼
                        REVENUE
                           │
                           ▼
                    BUSINESS VALUE

El objetivo es conectar:

Valor entregado → Monetización → Cobro → Ingreso → Rentabilidad.

3. Principios del Billing Business Model

El modelo económico debe basarse en varios principios.

3.1 Monetization Flexibility

La plataforma no debe depender de un único modelo de cobro.

Debe poder soportar:

pago único;
suscripción;
pago recurrente;
consumo;
usage-based billing;
tiered pricing;
volume pricing;
seat-based pricing;
feature-based pricing;
hybrid pricing;
marketplace fees;
commissions;
revenue sharing.
3.2 Commercial Independence

El modelo comercial debe poder evolucionar sin modificar profundamente el motor financiero.

Por ejemplo:

Product
   │
   ├── Monthly Subscription
   │
   ├── Annual Subscription
   │
   ├── Usage Based
   │
   └── Hybrid

El producto no debe estar acoplado rígidamente a una única estrategia de monetización.

3.3 Tenant-Level Monetization

En una plataforma multi-tenant, cada tenant puede representar una unidad comercial diferente.

Enterprise
│
├── Tenant A
│     └── Subscription Model
│
├── Tenant B
│     └── Usage Model
│
├── Tenant C
│     └── Hybrid Model
│
└── Tenant D
      └── Enterprise Contract

Esto permite que la plataforma soporte diferentes modelos comerciales simultáneamente.

4. Principales modelos de monetización
4.1 Flat Rate

El cliente paga una cantidad fija.

Plan Professional
$99 / month

Características:

simple;
predecible;
fácil de administrar;
independiente del consumo.

Es adecuado para productos con costos relativamente estables.

5. Per-Seat Pricing

El precio depende del número de usuarios.

Price = Users × PricePerUser

Ejemplo:

50 users
×
$10/user
=
$500/month

Es común en:

SaaS;
collaboration;
CRM;
ERP;
enterprise platforms.
6. Usage-Based Pricing

El cliente paga según consumo.

Usage
   ↓
Meter
   ↓
Rate
   ↓
Charge

Ejemplos:

API calls;
storage;
bandwidth;
AI tokens;
transactions;
messages;
reports;
mobility events;
data processing.

La fórmula conceptual es:

Charge = Billable Usage × Unit Price
7. Tiered Pricing

El precio cambia según rangos de consumo.

0 – 1,000       → $0.10
1,001 – 10,000  → $0.08
10,001 – 100k   → $0.05
100k+           → $0.03

Puede utilizarse para incentivar mayor consumo.

8. Volume Pricing

A diferencia del tiered pricing, el precio final puede determinarse según el nivel total alcanzado.

Usage = 15,000


Applicable Tier
= 10,001 – 20,000


Price
= $0.05 × 15,000

La plataforma debe distinguir claramente entre:

tiered pricing;
volume pricing;
graduated pricing.
9. Hybrid Billing

Uno de los modelos más importantes para Enterprise es el Hybrid Billing Model.

Combina diferentes fuentes de monetización.

Subscription
      +
Seats
      +
Usage
      +
Add-ons
      +
Overage
      =
Total Billing

Ejemplo:

Base Plan             $500
10 Users              $200
API Usage             $120
Storage Overage        $50
AI Consumption        $180
──────────────────────────
Total                $1,050

Este modelo debe ser una capacidad nativa de Billing.

10. Contract-Based Billing

Los clientes Enterprise pueden utilizar contratos personalizados.

Enterprise Contract
        │
        ├── Contract Value
        ├── Start Date
        ├── End Date
        ├── Minimum Commitment
        ├── Pricing Rules
        ├── Discounts
        ├── Usage Limits
        └── Payment Terms

Esto permite negociar:

precios especiales;
descuentos;
mínimos de consumo;
compromisos anuales;
SLAs;
créditos;
condiciones de pago.
11. Commitment-Based Billing

El cliente se compromete a consumir un determinado volumen.

Annual Commitment
      │
      ▼
$120,000
      │
      ├── Used $80,000
      │
      └── Remaining $40,000

El sistema debe poder administrar:

committed amount;
consumed amount;
remaining commitment;
expiration;
overage;
under-utilization.
12. Credit-Based Billing

Los clientes pueden utilizar créditos prepagados.

Credit Purchase
      │
      ▼
1,000 Credits
      │
      ├── API → -100
      ├── AI  → -250
      ├── Data → -150
      │
      ▼
Remaining = 500

Esto resulta especialmente útil para:

IA;
APIs;
data platforms;
procesamiento;
marketplaces;
servicios digitales.
13. Prepaid Billing

El cliente paga antes de utilizar el servicio.

Payment
   ↓
Wallet / Balance
   ↓
Usage
   ↓
Balance Reduction

Ejemplo:

Prepaid Balance = $1,000


Usage = $150


Remaining = $850

Debe existir protección contra:

saldo negativo;
doble consumo;
concurrencia;
eventos duplicados.
14. Postpaid Billing

El cliente consume primero y paga posteriormente.

Usage
  ↓
Rating
  ↓
Charges
  ↓
Invoice
  ↓
Payment

Es uno de los modelos tradicionales de SaaS y Enterprise.

15. Billing Frequency

Billing debe soportar múltiples frecuencias.

Monthly
Monthly
Quarterly
Quarterly
Semi-Annual
Every 6 months
Annual
Annual
Custom
Custom Billing Period

También debe permitir:

billing anchor;
proration;
partial periods;
anniversary billing.
16. Revenue Model

Billing no debe confundirse con Revenue.

Billing
   ↓
Invoice
   ↓
Payment
   ↓
Revenue Recognition

Un monto facturado no necesariamente representa ingreso reconocido inmediatamente.

Por ejemplo:

Annual Contract
$12,000


Invoice
$12,000


Recognized Revenue
$1,000 / month

Por lo tanto, Billing debe proporcionar información para Revenue Management.

17. Pricing Architecture

El modelo económico puede estructurarse:

Product
   │
   ▼
Plan
   │
   ▼
Price
   │
   ▼
Pricing Model
   │
   ├── Flat
   ├── Seat
   ├── Usage
   ├── Tiered
   ├── Volume
   ├── Hybrid
   └── Contract

Esto permite separar:

qué se vende de cómo se cobra.

18. Commercial Entities

El Business Model debe definir las principales entidades económicas.

Product
   │
   ▼
Offer
   │
   ▼
Price
   │
   ▼
Subscription
   │
   ▼
Billing Account
   │
   ▼
Invoice
   │
   ▼
Payment

Estas entidades deben mantenerse desacopladas para permitir evolución comercial.

19. Billing Account Economics

El Billing Account representa la posición financiera del cliente.

Conceptualmente:

Opening Balance
      +
Charges
      +
Taxes
      -
Discounts
      -
Credits
      -
Payments
      +
Adjustments
      =
Closing Balance

Esto permite calcular:

saldo;
deuda;
créditos;
pagos pendientes;
exposición financiera.
20. Discounts

Los descuentos son parte del modelo comercial.

Pueden ser:

percentage;
fixed amount;
promotional;
volume;
contract;
loyalty;
seasonal;
introductory;
customer-specific.

Ejemplo:

Base Price       $1,000
Discount           20%
────────────────────
Net Price          $800

Billing debe registrar tanto el precio original como el descuento aplicado.

21. Taxes

Los impuestos deben formar parte del modelo económico pero permanecer desacoplados del pricing.

Net Amount
    ↓
Tax Engine
    ↓
Tax Amount
    ↓
Gross Amount

Conceptualmente:

Gross Amount =
Net Amount
+
Taxes
-
Discounts
-
Credits

El cálculo exacto dependerá de la jurisdicción y configuración fiscal.

22. Payment Economics

Billing debe distinguir:

Amount Due
      │
      ▼
Payment Attempt
      │
      ▼
Payment Authorized
      │
      ▼
Payment Captured
      │
      ▼
Settlement

Un payment intentado no necesariamente representa dinero recibido.

Por eso el sistema debe distinguir:

authorized;
captured;
settled;
failed;
refunded;
disputed.
23. Marketplace & Revenue Sharing

Una plataforma Enterprise puede actuar como marketplace.

Customer
    │
    ▼
Platform
    │
    ├── Platform Fee
    │
    └── Provider Revenue

Ejemplo:

Transaction       $1,000


Platform Fee        $150
Provider Revenue    $850

Billing debe soportar:

commissions;
fees;
revenue sharing;
partner payouts;
marketplace settlements.
24. Unit Economics

El modelo debe permitir analizar la rentabilidad real.

Revenue
   -
Infrastructure Cost
   -
Payment Cost
   -
Support Cost
   -
AI Cost
   -
Operational Cost
   =
Contribution Margin

Esto conecta Billing con FinOps.

Por ejemplo:

Customer Revenue       $1,000
AI Cost                  $120
Cloud Cost                $80
Payment Fee               $30
────────────────────────────
Contribution Margin      $770
25. Billing Business Model + AI

La IA puede transformar el modelo comercial.

AI Pricing

Los servicios de IA pueden monetizarse mediante:

tokens;
requests;
compute time;
model usage;
storage;
generated content;
agent executions.

Por ejemplo:

AI Request
   ↓
Input Tokens
   +
Output Tokens
   +
Model Cost
   ↓
Rating
   ↓
Charge
26. Dynamic Monetization

En una plataforma avanzada, los modelos de pricing pueden evolucionar dinámicamente.

Customer Behavior
        ↓
Usage Analytics
        ↓
AI / ML
        ↓
Pricing Intelligence
        ↓
Commercial Recommendation

La plataforma puede identificar:

clientes subutilizados;
clientes sobreutilizados;
oportunidades de upselling;
planes inadecuados;
exceso de descuentos;
oportunidades de expansión.

La IA puede recomendar, pero las reglas financieras críticas deben permanecer gobernadas.

27. Billing Business Model en EVOXA

Para EVOXA, el modelo puede evolucionar hacia una estructura:

                         EVOXA
                           │
                    Monetization Layer
                           │
                ┌──────────┼──────────┐
                │          │          │
             SaaS        Usage       AI
                │          │          │
                └──────────┼──────────┘
                           │
                     Billing Engine
                           │
          ┌────────────────┼────────────────┐
          │                │                │
      Subscriptions     Charges          Credits
          │                │                │
          └────────────────┼────────────────┘
                           │
                        Invoice
                           │
                        Payment
                           │
                     Revenue Engine
                           │
                  ┌────────┴────────┐
                  │                 │
              Accounting          FinOps
                  │                 │
                  └────────┬────────┘
                           │
                    AI Revenue Intelligence

Esto permite que EVOXA soporte tanto modelos simples como Enterprise.

28. Business Model Matrix
Modelo	Pricing Driver	Billing Type	Ejemplo
Flat Rate	Plan	Recurring	$99/mes
Per Seat	Usuarios	Recurring	$10/user
Usage	Consumo	Metered	$0.01/event
Tiered	Rango	Metered	Escalas por volumen
Volume	Volumen total	Metered	Precio según nivel
Hybrid	Múltiples	Mixed	Plan + Usage
Prepaid	Créditos	Prepaid	Wallet
Postpaid	Consumo	Invoice	Pago posterior
Commitment	Contrato	Contract	$120K/año
Marketplace	GMV	Commission	15% fee
AI Usage	Tokens/requests	Metered	AI consumption

29. Economic Control Model

El Billing Business Model debe mantener separación entre:

Commercial Decision
        │
        ▼
Pricing Configuration
        │
        ▼
Billing Calculation
        │
        ▼
Financial Transaction
        │
        ▼
Accounting

Esto evita que una modificación comercial altere directamente los registros financieros históricos.

30. Core Business Model

El modelo completo puede resumirse como:

                     CUSTOMER VALUE
                           │
                           ▼
                        PRODUCT
                           │
                           ▼
                         OFFER
                           │
                           ▼
                         PRICE
                           │
                           ▼
                    COMMERCIAL MODEL
                           │
          ┌────────────────┼────────────────┐
          │                │                │
     Subscription        Usage          Contract
          │                │                │
          └────────────────┼────────────────┘
                           ▼
                         BILLING
                           │
                  ┌────────┴────────┐
                  │                 │
                Charge           Credit
                  │                 │
                  └────────┬────────┘
                           ▼
                        INVOICE
                           │
                           ▼
                        PAYMENT
                           │
                           ▼
                        REVENUE
                           │
                           ▼
                      PROFITABILITY
31. Resultado de 26/02

Billing Business Model establece que Billing debe ser una plataforma capaz de soportar múltiples estrategias de monetización simultáneamente, manteniendo separación entre producto, pricing, consumo, facturación, pago, revenue y rentabilidad.

El principio central es:

Billing no solamente calcula cuánto debe pagar un cliente; convierte una estrategia comercial en una realidad financiera medible, cobrable, auditable y escalable.

Sobre esta base, 26/03 — Billing Architecture deberá definir la arquitectura técnica y lógica que permite implementar todos estos modelos sin convertir Billing en un sistema rígido o altamente acoplado.
