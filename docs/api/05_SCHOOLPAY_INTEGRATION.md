# SchoolPay Integration - U-USMS

## Overview

SchoolPay is Uganda's leading education payment gateway, enabling seamless fee collection via:
- Mobile Money (MTN, Airtel)
- Bank transfers
- Card payments
- Agent banking

---

## Integration Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                     U-USMS System                            │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  ┌──────────────┐      ┌──────────────┐                    │
│  │   Invoice    │      │   Payment    │                    │
│  │   Generator  │      │   Service    │                    │
│  └──────┬───────┘      └──────┬───────┘                    │
│         │                     │                             │
└─────────┼─────────────────────┼─────────────────────────────┘
          │                     │
          │  Payment Request    │  Payment Verification
          │  (API Call)         │  (Webhook)
          ▼                     ▼
┌─────────────────────────────────────────────────────────────┐
│              SchoolPay API Gateway                           │
├─────────────────────────────────────────────────────────────┤
│  OAuth2 Authentication │ Payment Processing │ Webhooks      │
└─────────┬───────────────────────┬─────────────────────────┬─┘
          │                       │                         │
          ▼                       ▼                         ▼
┌─────────────────┐   ┌──────────────────┐   ┌─────────────────┐
│  Mobile Money   │   │  Bank Transfers  │   │  Card Payments  │
│  (MTN, Airtel)  │   │  (All Banks)     │   │  (Visa, MC)     │
└─────────────────┘   └──────────────────┘   └─────────────────┘
```

---

## Authentication - OAuth2

### Step 1: Obtain Access Token

**Endpoint:**
```
POST https://api.schoolpay.ug/oauth/token
```

**Request:**
```json
{
  "grant_type": "client_credentials",
  "client_id": "your_client_id",
  "client_secret": "your_client_secret",
  "scope": "payments:create payments:read"
}
```

**Response:**
```json
{
  "access_token": "eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9...",
  "token_type": "Bearer",
  "expires_in": 3600,
  "scope": "payments:create payments:read"
}
```

### Step 2: Use Token in Requests

```
Authorization: Bearer eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9...
```

**Token Refresh:**
```
POST /oauth/token/refresh
{
  "refresh_token": "..."
}
```

---

## Payment Initiation Flow

### 1. Generate Invoice in U-USMS

```javascript
// U-USMS creates invoice
POST /api/v1/invoices
{
  "student_id": "550e8400-e29b-41d4-a716-446655440000",
  "term_id": "...",
  "items": [
    {
      "description": "Tuition Fee - Term 1",
      "amount": 500000
    },
    {
      "description": "Books & Stationery",
      "amount": 100000
    }
  ],
  "due_date": "2026-03-15"
}

// Response:
{
  "invoice_id": "inv_abc123",
  "invoice_number": "INV-2026-001234",
  "total_amount": 600000,
  "balance": 600000
}
```

### 2. Initiate Payment via SchoolPay

**Endpoint:**
```
POST https://api.schoolpay.ug/v1/payments/initiate
```

**Request:**
```json
{
  "merchant_reference": "INV-2026-001234",
  "amount": 600000,
  "currency": "UGX",
  "customer": {
    "name": "John Doe Sr.",
    "email": "johndoe@example.com",
    "phone": "+256700123456"
  },
  "payment_methods": ["mobile_money", "bank", "card"],
  "metadata": {
    "student_id": "550e8400-e29b-41d4-a716-446655440000",
    "student_name": "John Doe Jr.",
    "class": "Primary 5A",
    "invoice_id": "inv_abc123"
  },
  "callback_url": "https://api.u-usms.ug/webhooks/schoolpay/callback",
  "return_url": "https://app.u-usms.ug/payments/success",
  "cancel_url": "https://app.u-usms.ug/payments/cancel"
}
```

**Response:**
```json
{
  "success": true,
  "data": {
    "payment_id": "pay_xyz789",
    "merchant_reference": "INV-2026-001234",
    "status": "pending",
    "payment_url": "https://pay.schoolpay.ug/pay/xyz789",
    "expires_at": "2026-02-15T12:00:00Z",
    "qr_code": "data:image/png;base64,iVBORw0KGgoAAAA..."
  }
}
```

### 3. Customer Makes Payment

**Options:**

#### Option A: Mobile Money (USSD)
```
Customer dials: *165#
Selects: SchoolPay
Enters: Payment Code (XYZ789)
Confirms: Amount (UGX 600,000)
Enters: PIN
```

#### Option B: Mobile Money (App)
```
Customer opens: MTN MoMo App
Selects: Pay Bill
Business: SchoolPay
Reference: XYZ789
Amount: 600,000
Confirms with PIN
```

#### Option C: Bank Transfer
```
Customer logs into bank app
Selects: Pay to SchoolPay
Account: 1234567890 (Auto-populated)
Reference: XYZ789
Amount: 600,000
```

#### Option D: Payment Link
```
Customer clicks payment_url
Redirected to SchoolPay checkout
Selects payment method
Completes payment
```

---

## Webhook Integration

### Webhook Endpoint Setup

**U-USMS Webhook Endpoint:**
```
POST /webhooks/schoolpay/callback
```

**Webhook Events:**
- `payment.pending` - Payment initiated
- `payment.processing` - Payment being processed
- `payment.successful` - Payment successful
- `payment.failed` - Payment failed
- `payment.reversed` - Payment reversed/refunded

### Webhook Payload

**payment.successful Example:**
```json
{
  "event": "payment.successful",
  "timestamp": "2026-02-15T10:30:00Z",
  "data": {
    "payment_id": "pay_xyz789",
    "merchant_reference": "INV-2026-001234",
    "amount": 600000,
    "currency": "UGX",
    "payment_method": "mobile_money",
    "provider": "MTN",
    "provider_reference": "MP260215.1234.A12345",
    "customer": {
      "name": "John Doe Sr.",
      "phone": "+256700123456"
    },
    "metadata": {
      "student_id": "550e8400-e29b-41d4-a716-446655440000",
      "invoice_id": "inv_abc123"
    },
    "paid_at": "2026-02-15T10:30:00Z"
  },
  "signature": "sha256:a1b2c3d4e5f6..."
}
```

### Webhook Verification

**Verify signature to ensure authenticity:**

```javascript
const crypto = require('crypto');

function verifySchoolPayWebhook(payload, signature, secret) {
  const expectedSignature = crypto
    .createHmac('sha256', secret)
    .update(JSON.stringify(payload))
    .digest('hex');
  
  return signature === `sha256:${expectedSignature}`;
}

// Express.js webhook handler
app.post('/webhooks/schoolpay/callback', async (req, res) => {
  const payload = req.body;
  const signature = req.headers['x-schoolpay-signature'];
  const secret = process.env.SCHOOLPAY_WEBHOOK_SECRET;
  
  // Verify signature
  if (!verifySchoolPayWebhook(payload, signature, secret)) {
    return res.status(401).json({ error: 'Invalid signature' });
  }
  
  // Process webhook
  try {
    await processWebhook(payload);
    res.status(200).json({ received: true });
  } catch (error) {
    res.status(500).json({ error: 'Processing failed' });
  }
});
```

### Processing Webhook

```javascript
async function processWebhook(payload) {
  const { event, data } = payload;
  
  switch (event) {
    case 'payment.successful':
      await handleSuccessfulPayment(data);
      break;
    
    case 'payment.failed':
      await handleFailedPayment(data);
      break;
    
    case 'payment.reversed':
      await handleReversedPayment(data);
      break;
    
    default:
      console.log(`Unhandled event: ${event}`);
  }
}

async function handleSuccessfulPayment(data) {
  const {
    payment_id,
    merchant_reference,
    amount,
    provider_reference,
    metadata
  } = data;
  
  // 1. Create payment record
  const payment = await Payment.create({
    payment_id: uuidv4(),
    invoice_id: metadata.invoice_id,
    student_id: metadata.student_id,
    amount: amount,
    payment_method: 'mobile_money',
    provider: data.provider,
    transaction_id: provider_reference,
    schoolpay_txn_id: payment_id,
    status: 'verified',
    paid_at: data.paid_at,
    verified_at: new Date()
  });
  
  // 2. Update invoice
  await Invoice.update(
    { invoice_id: metadata.invoice_id },
    {
      paid_amount: sequelize.literal(`paid_amount + ${amount}`),
      balance: sequelize.literal(`balance - ${amount}`),
      status: sequelize.literal(
        `CASE WHEN balance - ${amount} <= 0 THEN 'paid' ELSE 'partially_paid' END`
      )
    }
  );
  
  // 3. Post to ledger
  await LedgerEntry.create({
    transaction_id: payment.payment_id,
    account: 'BANK_ACCOUNT',
    debit: amount,
    credit: 0,
    description: `Fee payment from ${data.customer.name}`
  });
  
  await LedgerEntry.create({
    transaction_id: payment.payment_id,
    account: 'FEE_REVENUE',
    debit: 0,
    credit: amount,
    description: `Fee payment for ${metadata.student_name}`
  });
  
  // 4. Generate receipt
  const receipt = await generateReceipt(payment);
  
  // 5. Send SMS confirmation
  await sendSMS({
    recipient: data.customer.phone,
    message: `Payment of UGX ${amount.toLocaleString()} received for ${metadata.student_name}. Receipt No: ${receipt.receipt_number}. Thank you!`,
    type: 'payment_confirmation'
  });
  
  // 6. Send email with receipt
  await sendEmail({
    to: data.customer.email,
    subject: 'Payment Receipt - Universal School',
    template: 'payment_receipt',
    data: { payment, receipt }
  });
}
```

---

## Payment Verification API

### Verify Payment Status

**Endpoint:**
```
GET https://api.schoolpay.ug/v1/payments/{payment_id}
```

**Request:**
```
GET /v1/payments/pay_xyz789
Authorization: Bearer <token>
```

**Response:**
```json
{
  "success": true,
  "data": {
    "payment_id": "pay_xyz789",
    "merchant_reference": "INV-2026-001234",
    "amount": 600000,
    "currency": "UGX",
    "status": "successful",
    "payment_method": "mobile_money",
    "provider": "MTN",
    "provider_reference": "MP260215.1234.A12345",
    "customer": {
      "name": "John Doe Sr.",
      "phone": "+256700123456"
    },
    "created_at": "2026-02-15T10:25:00Z",
    "paid_at": "2026-02-15T10:30:00Z"
  }
}
```

---

## Installment Payments

### Scenario: Parent Pays in 3 Installments

**Total Fee:** UGX 600,000  
**Installments:** 3 × UGX 200,000

#### Installment 1
```javascript
POST /schoolpay/initiate
{
  "merchant_reference": "INV-2026-001234-INST1",
  "amount": 200000,
  "metadata": {
    "invoice_id": "inv_abc123",
    "installment_number": 1,
    "total_installments": 3
  }
}
```

#### Installment 2 (2 weeks later)
```javascript
POST /schoolpay/initiate
{
  "merchant_reference": "INV-2026-001234-INST2",
  "amount": 200000,
  "metadata": {
    "invoice_id": "inv_abc123",
    "installment_number": 2,
    "total_installments": 3
  }
}
```

#### Installment 3 (4 weeks later)
```javascript
POST /schoolpay/initiate
{
  "merchant_reference": "INV-2026-001234-INST3",
  "amount": 200000,
  "metadata": {
    "invoice_id": "inv_abc123",
    "installment_number": 3,
    "total_installments": 3
  }
}
```

**Auto-update invoice after each payment:**
```
After Installment 1: Balance = 400,000
After Installment 2: Balance = 200,000
After Installment 3: Balance = 0 (Status: Paid)
```

---

## Failed Payment Retry Logic

### Automatic Retry Mechanism

```javascript
async function handleFailedPayment(data) {
  const { payment_id, merchant_reference, metadata } = data;
  
  // 1. Log failure
  await PaymentFailure.create({
    payment_id: payment_id,
    invoice_id: metadata.invoice_id,
    reason: data.failure_reason,
    failed_at: new Date()
  });
  
  // 2. Check retry count
  const retryCount = await PaymentRetry.count({
    where: { invoice_id: metadata.invoice_id }
  });
  
  if (retryCount < 3) {
    // 3. Schedule retry (24 hours later)
    await schedulePaymentRetry({
      invoice_id: metadata.invoice_id,
      retry_at: moment().add(24, 'hours').toDate(),
      retry_number: retryCount + 1
    });
    
    // 4. Notify customer
    await sendSMS({
      recipient: data.customer.phone,
      message: `Your payment of UGX ${data.amount.toLocaleString()} failed. Please try again or contact support.`,
      type: 'payment_failed'
    });
  } else {
    // 5. Escalate to support after 3 failed attempts
    await createSupportTicket({
      type: 'payment_failure',
      invoice_id: metadata.invoice_id,
      priority: 'high'
    });
  }
}
```

---

## Payment Reconciliation

### Daily Reconciliation Process

```javascript
async function dailyReconciliation(date) {
  // 1. Fetch SchoolPay settlement report
  const settlementReport = await fetchSchoolPaySettlement(date);
  
  // 2. Fetch U-USMS payment records
  const usmsPayments = await Payment.findAll({
    where: {
      paid_at: {
        [Op.between]: [
          moment(date).startOf('day').toDate(),
          moment(date).endOf('day').toDate()
        ]
      },
      status: 'verified'
    }
  });
  
  // 3. Compare records
  const discrepancies = [];
  
  settlementReport.payments.forEach(spPayment => {
    const usmsPayment = usmsPayments.find(
      p => p.schoolpay_txn_id === spPayment.payment_id
    );
    
    if (!usmsPayment) {
      discrepancies.push({
        type: 'missing_in_usms',
        payment_id: spPayment.payment_id,
        amount: spPayment.amount
      });
    } else if (usmsPayment.amount !== spPayment.amount) {
      discrepancies.push({
        type: 'amount_mismatch',
        payment_id: spPayment.payment_id,
        usms_amount: usmsPayment.amount,
        schoolpay_amount: spPayment.amount
      });
    }
  });
  
  // 4. Generate reconciliation report
  await ReconciliationReport.create({
    date: date,
    total_schoolpay: settlementReport.total_amount,
    total_usms: usmsPayments.reduce((sum, p) => sum + p.amount, 0),
    discrepancies: discrepancies,
    status: discrepancies.length > 0 ? 'has_discrepancies' : 'matched'
  });
  
  // 5. Alert if discrepancies found
  if (discrepancies.length > 0) {
    await alertFinanceTeam({
      subject: `Payment Reconciliation Alert - ${date}`,
      discrepancies: discrepancies
    });
  }
}
```

---

## Bulk Fee Invoicing

### Generate Bulk Invoices for Term

```javascript
async function generateTermInvoices(termId, classId) {
  // 1. Get students in class
  const students = await Student.findAll({
    where: { class_id: classId, status: 'active' }
  });
  
  // 2. Get fee structure for class
  const feeStructure = await FeeStructure.findAll({
    where: { class_id: classId, year_id: term.year_id }
  });
  
  // 3. Generate invoices
  const invoices = [];
  
  for (const student of students) {
    const invoice = await Invoice.create({
      student_id: student.student_id,
      term_id: termId,
      invoice_number: generateInvoiceNumber(),
      total_amount: calculateTotalFees(feeStructure),
      paid_amount: 0,
      balance: calculateTotalFees(feeStructure),
      due_date: moment().add(30, 'days').toDate(),
      status: 'pending'
    });
    
    // 4. Add invoice items
    for (const fee of feeStructure) {
      await InvoiceItem.create({
        invoice_id: invoice.invoice_id,
        fee_structure_id: fee.structure_id,
        description: fee.description,
        amount: fee.amount,
        quantity: 1
      });
    }
    
    invoices.push(invoice);
  }
  
  // 5. Send SMS reminders to parents
  for (const invoice of invoices) {
    const guardians = await getStudentGuardians(invoice.student_id);
    
    for (const guardian of guardians) {
      await sendSMS({
        recipient: guardian.phone,
        message: `Dear parent, ${guardian.student_name}'s Term 1 fee is UGX ${invoice.total_amount.toLocaleString()}. Invoice: ${invoice.invoice_number}. Pay via SchoolPay.`,
        type: 'fee_invoice'
      });
    }
  }
  
  return invoices;
}
```

---

## Payment Analytics Dashboard

### Key Metrics

```javascript
async function getPaymentAnalytics(schoolId, startDate, endDate) {
  const analytics = {
    // Total collected
    total_collected: await Payment.sum('amount', {
      where: {
        school_id: schoolId,
        paid_at: { [Op.between]: [startDate, endDate] },
        status: 'verified'
      }
    }),
    
    // Payment methods breakdown
    by_method: await Payment.findAll({
      attributes: [
        'payment_method',
        [sequelize.fn('COUNT', sequelize.col('payment_id')), 'count'],
        [sequelize.fn('SUM', sequelize.col('amount')), 'total']
      ],
      where: {
        school_id: schoolId,
        paid_at: { [Op.between]: [startDate, endDate] }
      },
      group: ['payment_method']
    }),
    
    // Daily collections
    daily: await Payment.findAll({
      attributes: [
        [sequelize.fn('DATE', sequelize.col('paid_at')), 'date'],
        [sequelize.fn('SUM', sequelize.col('amount')), 'total']
      ],
      where: {
        school_id: schoolId,
        paid_at: { [Op.between]: [startDate, endDate] }
      },
      group: [sequelize.fn('DATE', sequelize.col('paid_at'))]
    }),
    
    // Outstanding balance
    outstanding: await Invoice.sum('balance', {
      where: {
        school_id: schoolId,
        status: { [Op.in]: ['pending', 'partially_paid'] }
      }
    }),
    
    // Collection rate
    collection_rate: calculateCollectionRate(schoolId, startDate, endDate)
  };
  
  return analytics;
}
```

---

## Fraud Detection

### Transaction Monitoring

```javascript
async function detectSuspiciousPayment(payment) {
  const flags = [];
  
  // 1. Check for duplicate payments
  const duplicates = await Payment.count({
    where: {
      student_id: payment.student_id,
      amount: payment.amount,
      paid_at: {
        [Op.between]: [
          moment(payment.paid_at).subtract(5, 'minutes').toDate(),
          moment(payment.paid_at).add(5, 'minutes').toDate()
        ]
      }
    }
  });
  
  if (duplicates > 1) {
    flags.push('DUPLICATE_PAYMENT');
  }
  
  // 2. Check for unusually large amount
  const avgPayment = await Payment.findOne({
    attributes: [[sequelize.fn('AVG', sequelize.col('amount')), 'avg']],
    where: { school_id: payment.school_id }
  });
  
  if (payment.amount > avgPayment.avg * 5) {
    flags.push('UNUSUALLY_LARGE_AMOUNT');
  }
  
  // 3. Check for rapid successive payments
  const recentPayments = await Payment.count({
    where: {
      student_id: payment.student_id,
      paid_at: {
        [Op.gte]: moment().subtract(1, 'hour').toDate()
      }
    }
  });
  
  if (recentPayments > 3) {
    flags.push('RAPID_PAYMENTS');
  }
  
  // 4. Alert if suspicious
  if (flags.length > 0) {
    await createAlert({
      type: 'SUSPICIOUS_PAYMENT',
      payment_id: payment.payment_id,
      flags: flags,
      severity: 'high'
    });
  }
}
```

---

## Testing

### Sandbox Credentials

```
API URL: https://sandbox.schoolpay.ug/v1
Client ID: test_client_abc123
Client Secret: test_secret_xyz789
Webhook Secret: test_webhook_secret
```

### Test Phone Numbers

```
Success:  +256700000001
Pending:  +256700000002
Failed:   +256700000003
```

### Test Amounts

```
Any amount ending in 00: Success
Any amount ending in 99: Failed
Example: 10000 (success), 10099 (failed)
```

---

## Production Checklist

- [ ] Obtain production API credentials from SchoolPay
- [ ] Configure OAuth2 client ID and secret
- [ ] Set up webhook endpoint with SSL certificate
- [ ] Configure webhook secret for signature verification
- [ ] Test payment flow end-to-end
- [ ] Implement reconciliation process
- [ ] Set up monitoring and alerts
- [ ] Train finance team on reconciliation
- [ ] Prepare customer support documentation
- [ ] Go live!

---

## Support & Resources

- **SchoolPay Documentation:** https://docs.schoolpay.ug
- **SchoolPay Support:** support@schoolpay.ug / +256 800 100 200
- **API Status:** https://status.schoolpay.ug
- **Developer Portal:** https://developers.schoolpay.ug
