# API Specification - U-USMS

## Overview

U-USMS provides a comprehensive RESTful API for all system operations.

---

## Base URL

```
Production:  https://api.u-usms.ug/v1
Staging:     https://api-staging.u-usms.ug/v1
Development: https://api-dev.u-usms.ug/v1
```

---

## Authentication

### JWT-Based Authentication

**Login Endpoint:**
```
POST /auth/login
```

**Request:**
```json
{
  "email": "admin@school.ug",
  "password": "SecurePassword123!",
  "tenant_id": "550e8400-e29b-41d4-a716-446655440000"
}
```

**Response:**
```json
{
  "success": true,
  "data": {
    "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "refresh_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "expires_in": 3600,
    "token_type": "Bearer",
    "user": {
      "user_id": "...",
      "email": "admin@school.ug",
      "role": "admin",
      "tenant_id": "...",
      "school_id": "..."
    }
  }
}
```

**Using the Token:**
```
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
X-Tenant-ID: 550e8400-e29b-41d4-a716-446655440000
```

### Multi-Factor Authentication (MFA)

**Enable MFA:**
```
POST /auth/mfa/enable
Authorization: Bearer <token>
```

**Response:**
```json
{
  "success": true,
  "data": {
    "secret": "JBSWY3DPEHPK3PXP",
    "qr_code_url": "data:image/png;base64,iVBORw0KGgoAAAA...",
    "backup_codes": [
      "12345678",
      "87654321",
      "..."
    ]
  }
}
```

**Verify MFA:**
```
POST /auth/mfa/verify
```

**Request:**
```json
{
  "code": "123456"
}
```

---

## Rate Limiting

**Limits per IP/User:**
- **Anonymous:** 100 requests/hour
- **Authenticated:** 1,000 requests/hour
- **Premium:** 10,000 requests/hour

**Rate Limit Headers:**
```
X-RateLimit-Limit: 1000
X-RateLimit-Remaining: 999
X-RateLimit-Reset: 1640995200
```

**Rate Limit Exceeded Response (429):**
```json
{
  "success": false,
  "error": {
    "code": "RATE_LIMIT_EXCEEDED",
    "message": "Too many requests. Please try again later.",
    "retry_after": 3600
  }
}
```

---

## Versioning

**API versioning via URL path:**
```
https://api.u-usms.ug/v1/students
https://api.u-usms.ug/v2/students  (Future version)
```

**Version compatibility:**
- **v1:** Supported until Dec 2027
- **Deprecation notice:** 12 months before sunset
- **Breaking changes:** Major version bump only

---

## Standard Response Format

### Success Response
```json
{
  "success": true,
  "data": { ... },
  "meta": {
    "timestamp": "2026-02-15T05:30:00Z",
    "request_id": "req_abc123"
  }
}
```

### Error Response
```json
{
  "success": false,
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Invalid input data",
    "details": [
      {
        "field": "email",
        "message": "Invalid email format"
      }
    ]
  },
  "meta": {
    "timestamp": "2026-02-15T05:30:00Z",
    "request_id": "req_abc123"
  }
}
```

### Pagination Response
```json
{
  "success": true,
  "data": [ ... ],
  "pagination": {
    "page": 1,
    "per_page": 20,
    "total": 150,
    "total_pages": 8,
    "has_next": true,
    "has_prev": false
  },
  "meta": {
    "timestamp": "2026-02-15T05:30:00Z",
    "request_id": "req_abc123"
  }
}
```

---

## API Endpoints

### 1. Students API

#### List Students
```
GET /students?page=1&per_page=20&class_id=<uuid>&status=active
```

**Query Parameters:**
- `page` (optional): Page number (default: 1)
- `per_page` (optional): Items per page (default: 20, max: 100)
- `class_id` (optional): Filter by class
- `status` (optional): Filter by status (active, suspended, etc.)
- `search` (optional): Search by name or enrollment number

**Response:**
```json
{
  "success": true,
  "data": [
    {
      "student_id": "550e8400-e29b-41d4-a716-446655440000",
      "enrollment_number": "STU2026001",
      "first_name": "John",
      "last_name": "Doe",
      "class": {
        "class_id": "...",
        "name": "Primary 5A"
      },
      "status": "active",
      "admission_date": "2024-01-15"
    }
  ],
  "pagination": { ... }
}
```

#### Get Student Details
```
GET /students/{student_id}
```

**Response:**
```json
{
  "success": true,
  "data": {
    "student_id": "550e8400-e29b-41d4-a716-446655440000",
    "enrollment_number": "STU2026001",
    "first_name": "John",
    "last_name": "Doe",
    "middle_name": "Peter",
    "date_of_birth": "2015-06-10",
    "gender": "Male",
    "nin": "CM12345678ABCD",
    "photo_url": "https://cdn.u-usms.ug/photos/...",
    "class": {
      "class_id": "...",
      "name": "Primary 5A",
      "level": "P5",
      "class_teacher": {
        "staff_id": "...",
        "name": "Mrs. Jane Smith"
      }
    },
    "guardians": [
      {
        "guardian_id": "...",
        "name": "Mr. John Doe Sr.",
        "relationship": "Father",
        "phone": "+256700123456",
        "is_primary": true
      }
    ],
    "medical_conditions": "None",
    "allergies": "Peanuts",
    "status": "active",
    "admission_date": "2024-01-15"
  }
}
```

#### Create Student
```
POST /students
```

**Request:**
```json
{
  "enrollment_number": "STU2026001",
  "class_id": "550e8400-e29b-41d4-a716-446655440000",
  "first_name": "John",
  "last_name": "Doe",
  "middle_name": "Peter",
  "date_of_birth": "2015-06-10",
  "gender": "Male",
  "nin": "CM12345678ABCD",
  "admission_date": "2024-01-15",
  "guardians": [
    {
      "first_name": "John",
      "last_name": "Doe Sr.",
      "relationship": "Father",
      "phone": "+256700123456",
      "email": "johndoe@example.com",
      "is_primary": true
    }
  ]
}
```

**Response: (201 Created)**
```json
{
  "success": true,
  "data": {
    "student_id": "550e8400-e29b-41d4-a716-446655440000",
    "enrollment_number": "STU2026001",
    ...
  }
}
```

#### Update Student
```
PUT /students/{student_id}
PATCH /students/{student_id}  (Partial update)
```

#### Delete Student (Soft Delete)
```
DELETE /students/{student_id}
```

---

### 2. Staff API

#### List Staff
```
GET /staff?role=teacher&status=active
```

#### Get Staff Details
```
GET /staff/{staff_id}
```

#### Create Staff
```
POST /staff
```

**Request:**
```json
{
  "employee_number": "EMP2026001",
  "first_name": "Jane",
  "last_name": "Smith",
  "nin": "CF12345678WXYZ",
  "date_of_birth": "1985-03-20",
  "gender": "Female",
  "email": "jane.smith@school.ug",
  "phone": "+256700987654",
  "role": "Teacher",
  "department": "Mathematics",
  "hire_date": "2020-01-10",
  "contract_type": "Permanent",
  "qualification": "Bachelor of Education",
  "bank_name": "Stanbic Bank",
  "bank_account_number": "1234567890",
  "bank_account_name": "Jane Smith",
  "nssf_number": "SF12345678",
  "tin_number": "1000123456"
}
```

---

### 3. Classes API

#### List Classes
```
GET /classes?year_id=<uuid>&level=P5
```

#### Create Class
```
POST /classes
```

**Request:**
```json
{
  "year_id": "550e8400-e29b-41d4-a716-446655440000",
  "name": "Primary 5A",
  "level": "P5",
  "stream": "A",
  "capacity": 50,
  "class_teacher_id": "..."
}
```

---

### 4. Attendance API

#### Mark Attendance
```
POST /attendance
```

**Request:**
```json
{
  "class_id": "550e8400-e29b-41d4-a716-446655440000",
  "date": "2026-02-15",
  "attendance": [
    {
      "student_id": "...",
      "status": "present"
    },
    {
      "student_id": "...",
      "status": "absent",
      "remarks": "Sick"
    }
  ]
}
```

**Response:**
```json
{
  "success": true,
  "data": {
    "attendance_id": "...",
    "class_id": "...",
    "date": "2026-02-15",
    "total_students": 45,
    "present": 43,
    "absent": 2,
    "marked_by": "..."
  }
}
```

#### Get Attendance Report
```
GET /attendance/reports?class_id=<uuid>&start_date=2026-02-01&end_date=2026-02-15
```

---

### 5. Assessments & Grades API

#### Create Assessment
```
POST /assessments
```

**Request:**
```json
{
  "subject_id": "550e8400-e29b-41d4-a716-446655440000",
  "class_id": "...",
  "term_id": "...",
  "name": "Mid-term Exam",
  "type": "exam",
  "total_marks": 100,
  "weight": 40,
  "date": "2026-02-20"
}
```

#### Enter Scores
```
POST /assessments/{assessment_id}/scores
```

**Request:**
```json
{
  "scores": [
    {
      "student_id": "...",
      "marks_obtained": 85,
      "remarks": "Excellent"
    },
    {
      "student_id": "...",
      "marks_obtained": 72
    }
  ]
}
```

#### Calculate Results
```
POST /results/calculate
```

**Request:**
```json
{
  "term_id": "550e8400-e29b-41d4-a716-446655440000",
  "class_id": "..."
}
```

**Response:**
```json
{
  "success": true,
  "data": {
    "results_generated": 45,
    "status": "completed"
  }
}
```

#### Get Student Results
```
GET /students/{student_id}/results?term_id=<uuid>
```

**Response:**
```json
{
  "success": true,
  "data": {
    "student": {
      "student_id": "...",
      "name": "John Doe",
      "class": "Primary 5A"
    },
    "term": {
      "term_id": "...",
      "name": "Term 1 2026"
    },
    "subjects": [
      {
        "subject_name": "Mathematics",
        "assessments": [
          {
            "name": "CAT 1",
            "marks": 20,
            "out_of": 25
          },
          {
            "name": "Mid-term",
            "marks": 38,
            "out_of": 40
          },
          {
            "name": "Final Exam",
            "marks": 58,
            "out_of": 60
          }
        ],
        "total_marks": 116,
        "percentage": 77.33,
        "grade": "A"
      }
    ],
    "overall": {
      "total_marks": 580,
      "average": 82.86,
      "grade": "A",
      "division": "I",
      "rank_in_class": 3,
      "out_of": 45
    },
    "remarks": "Excellent performance. Keep it up!"
  }
}
```

---

### 6. Financial API

#### Create Fee Structure
```
POST /fees/structures
```

**Request:**
```json
{
  "year_id": "550e8400-e29b-41d4-a716-446655440000",
  "class_id": "...",
  "category_id": "...",
  "amount": 500000,
  "currency": "UGX"
}
```

#### Generate Invoice
```
POST /invoices
```

**Request:**
```json
{
  "student_id": "550e8400-e29b-41d4-a716-446655440000",
  "term_id": "...",
  "items": [
    {
      "fee_structure_id": "...",
      "description": "Tuition Fee - Term 1",
      "amount": 500000,
      "quantity": 1
    },
    {
      "fee_structure_id": "...",
      "description": "Books & Stationery",
      "amount": 100000,
      "quantity": 1
    }
  ],
  "due_date": "2026-03-15"
}
```

**Response:**
```json
{
  "success": true,
  "data": {
    "invoice_id": "...",
    "invoice_number": "INV-2026-001234",
    "student": {
      "student_id": "...",
      "name": "John Doe",
      "enrollment_number": "STU2026001"
    },
    "total_amount": 600000,
    "paid_amount": 0,
    "balance": 600000,
    "status": "pending",
    "due_date": "2026-03-15",
    "items": [ ... ]
  }
}
```

#### Record Payment
```
POST /payments
```

**Request:**
```json
{
  "invoice_id": "550e8400-e29b-41d4-a716-446655440000",
  "amount": 300000,
  "payment_method": "mobile_money",
  "provider": "MTN",
  "reference_number": "MP260215.1234.A12345",
  "paid_at": "2026-02-15T10:30:00Z"
}
```

**Response:**
```json
{
  "success": true,
  "data": {
    "payment_id": "...",
    "reference_no": "PAY-2026-001234",
    "amount": 300000,
    "status": "verified",
    "receipt": {
      "receipt_id": "...",
      "receipt_number": "RCT-2026-001234",
      "pdf_url": "https://cdn.u-usms.ug/receipts/..."
    },
    "invoice": {
      "invoice_id": "...",
      "total_amount": 600000,
      "paid_amount": 300000,
      "balance": 300000
    }
  }
}
```

#### Payment Summary
```
GET /students/{student_id}/payments/summary
```

**Response:**
```json
{
  "success": true,
  "data": {
    "student_id": "...",
    "total_invoiced": 1800000,
    "total_paid": 1200000,
    "balance": 600000,
    "invoices": [
      {
        "invoice_number": "INV-2026-001234",
        "amount": 600000,
        "paid": 300000,
        "balance": 300000,
        "status": "partially_paid",
        "due_date": "2026-03-15"
      }
    ]
  }
}
```

---

### 7. Payroll API

#### Create Salary Structure
```
POST /payroll/salary-structures
```

**Request:**
```json
{
  "staff_id": "550e8400-e29b-41d4-a716-446655440000",
  "scale_id": "...",
  "basic_salary": 2000000,
  "housing_allowance": 300000,
  "transport_allowance": 150000,
  "hardship_allowance": 0,
  "other_allowances": 50000,
  "effective_date": "2026-01-01"
}
```

#### Run Payroll
```
POST /payroll/runs
```

**Request:**
```json
{
  "period_month": 2,
  "period_year": 2026,
  "staff_ids": ["...", "..."]  // Optional: specific staff, or all if omitted
}
```

**Response:**
```json
{
  "success": true,
  "data": {
    "run_id": "...",
    "period": "February 2026",
    "status": "draft",
    "staff_count": 45,
    "total_gross": 90000000,
    "total_net": 67500000,
    "total_paye": 18000000,
    "total_nssf_employee": 4500000,
    "total_nssf_employer": 9000000
  }
}
```

#### Get Payslip
```
GET /payroll/payslips/{payslip_id}
```

**Response:**
```json
{
  "success": true,
  "data": {
    "payslip_id": "...",
    "staff": {
      "staff_id": "...",
      "employee_number": "EMP2026001",
      "name": "Jane Smith",
      "nin": "CF12345678WXYZ"
    },
    "period": "February 2026",
    "earnings": {
      "basic_salary": 2000000,
      "housing_allowance": 300000,
      "transport_allowance": 150000,
      "other_allowances": 50000,
      "gross_salary": 2500000
    },
    "deductions": {
      "nssf_employee": 125000,
      "paye": 400000,
      "lst": 0,
      "other_deductions": 0,
      "total_deductions": 525000
    },
    "net_salary": 1975000,
    "employer_contributions": {
      "nssf_employer": 250000
    },
    "payment_details": {
      "bank_name": "Stanbic Bank",
      "account_number": "1234567890",
      "payment_method": "bank_transfer",
      "status": "pending"
    }
  }
}
```

#### Approve Payroll
```
POST /payroll/runs/{run_id}/approve
```

#### Export Payroll Report
```
GET /payroll/runs/{run_id}/export?format=pdf
GET /payroll/runs/{run_id}/export?format=csv
GET /payroll/runs/{run_id}/export?format=ura   (URA submission format)
GET /payroll/runs/{run_id}/export?format=nssf  (NSSF submission format)
GET /payroll/runs/{run_id}/export?format=bank  (Bank bulk upload)
```

---

### 8. AI Engine API

#### Predict Performance
```
POST /ai/predict/performance
```

**Request:**
```json
{
  "student_id": "550e8400-e29b-41d4-a716-446655440000",
  "subject_id": "...",
  "prediction_horizon": "next_term"
}
```

**Response:**
```json
{
  "success": true,
  "data": {
    "student_id": "...",
    "subject": "Mathematics",
    "current_performance": {
      "average": 75,
      "grade": "B"
    },
    "prediction": {
      "predicted_average": 82,
      "predicted_grade": "A",
      "confidence": 0.85
    },
    "factors": [
      {
        "factor": "Attendance",
        "impact": "positive",
        "weight": 0.3
      },
      {
        "factor": "Previous performance trend",
        "impact": "positive",
        "weight": 0.5
      }
    ],
    "recommendations": [
      "Continue current study habits",
      "Focus on algebra topics"
    ]
  }
}
```

#### Detect Dropout Risk
```
POST /ai/detect/dropout-risk
```

**Request:**
```json
{
  "student_id": "550e8400-e29b-41d4-a716-446655440000"
}
```

**Response:**
```json
{
  "success": true,
  "data": {
    "student_id": "...",
    "risk_score": 35,
    "severity": "medium",
    "factors": [
      {
        "factor": "Declining academic performance",
        "weight": 0.4
      },
      {
        "factor": "Irregular attendance (75%)",
        "weight": 0.3
      },
      {
        "factor": "Fee payment delays",
        "weight": 0.2
      }
    ],
    "interventions": [
      {
        "type": "Academic support",
        "description": "Assign peer tutor",
        "priority": "high"
      },
      {
        "type": "Counseling",
        "description": "Schedule meeting with guidance counselor",
        "priority": "medium"
      }
    ]
  }
}
```

#### AI Chatbot Query
```
POST /ai/chatbot/query
```

**Request:**
```json
{
  "user_id": "...",
  "message": "What is my child's fee balance?",
  "context": {
    "student_id": "..."
  }
}
```

**Response:**
```json
{
  "success": true,
  "data": {
    "response": "Your child John Doe (P5A) has a current balance of UGX 300,000. The last payment of UGX 300,000 was received on 15th February 2026. The next installment of UGX 300,000 is due on 15th March 2026.",
    "intent": "fee_balance_inquiry",
    "confidence": 0.95,
    "suggested_actions": [
      {
        "label": "Pay Now",
        "action": "initiate_payment",
        "amount": 300000
      },
      {
        "label": "View Invoice",
        "action": "view_invoice",
        "invoice_id": "..."
      }
    ]
  }
}
```

---

### 9. SMS API

#### Send SMS
```
POST /sms/send
```

**Request:**
```json
{
  "recipients": ["+256700123456", "+256700987654"],
  "message": "Dear parent, your child John Doe scored 85% in Mathematics. Keep encouraging!",
  "type": "result",
  "schedule_at": "2026-02-15T14:00:00Z"  // Optional
}
```

**Response:**
```json
{
  "success": true,
  "data": {
    "sms_batch_id": "...",
    "recipients_count": 2,
    "estimated_cost": 200,
    "status": "queued",
    "scheduled_at": "2026-02-15T14:00:00Z"
  }
}
```

#### Get SMS Status
```
GET /sms/{sms_id}/status
```

---

### 10. Reports API

#### Academic Report
```
GET /reports/academic?type=class_performance&class_id=<uuid>&term_id=<uuid>
```

#### Financial Report
```
GET /reports/financial?type=revenue&start_date=2026-01-01&end_date=2026-02-15
```

#### Payroll Report
```
GET /reports/payroll?run_id=<uuid>&format=pdf
```

---

## Webhooks

### Registering Webhooks
```
POST /webhooks
```

**Request:**
```json
{
  "url": "https://yourserver.com/webhook/payments",
  "events": ["payment.created", "payment.verified"],
  "secret": "your_webhook_secret"
}
```

### Webhook Events

**Payment Events:**
- `payment.created`
- `payment.verified`
- `payment.failed`

**Student Events:**
- `student.enrolled`
- `student.transferred`
- `student.graduated`

**Result Events:**
- `results.published`

**Payroll Events:**
- `payroll.approved`
- `payroll.paid`

### Webhook Payload
```json
{
  "event": "payment.verified",
  "timestamp": "2026-02-15T10:30:00Z",
  "data": {
    "payment_id": "...",
    "amount": 300000,
    "student_id": "...",
    "invoice_id": "..."
  },
  "signature": "sha256=..."
}
```

**Verifying Webhook:**
```javascript
const crypto = require('crypto');

function verifyWebhook(payload, signature, secret) {
  const hash = crypto
    .createHmac('sha256', secret)
    .update(JSON.stringify(payload))
    .digest('hex');
  
  return signature === `sha256=${hash}`;
}
```

---

## Error Codes

| Code | HTTP Status | Description |
|------|-------------|-------------|
| `SUCCESS` | 200 | Request succeeded |
| `CREATED` | 201 | Resource created |
| `NO_CONTENT` | 204 | Request succeeded, no content |
| `BAD_REQUEST` | 400 | Invalid request format |
| `UNAUTHORIZED` | 401 | Authentication required |
| `FORBIDDEN` | 403 | Insufficient permissions |
| `NOT_FOUND` | 404 | Resource not found |
| `CONFLICT` | 409 | Resource conflict (duplicate) |
| `VALIDATION_ERROR` | 422 | Input validation failed |
| `RATE_LIMIT_EXCEEDED` | 429 | Too many requests |
| `INTERNAL_SERVER_ERROR` | 500 | Server error |
| `SERVICE_UNAVAILABLE` | 503 | Service temporarily unavailable |

---

## SDK Examples

### JavaScript/Node.js
```javascript
const USMS = require('usms-sdk');

const client = new USMS({
  apiKey: 'your_api_key',
  tenantId: 'your_tenant_id',
  environment: 'production'
});

// Create student
const student = await client.students.create({
  enrollment_number: 'STU2026001',
  first_name: 'John',
  last_name: 'Doe',
  // ...
});

// List students
const students = await client.students.list({
  class_id: '...',
  status: 'active',
  page: 1,
  per_page: 20
});

// Record payment
const payment = await client.payments.create({
  invoice_id: '...',
  amount: 300000,
  payment_method: 'mobile_money',
  provider: 'MTN'
});
```

### Python
```python
from usms import Client

client = Client(
    api_key='your_api_key',
    tenant_id='your_tenant_id',
    environment='production'
)

# Create student
student = client.students.create(
    enrollment_number='STU2026001',
    first_name='John',
    last_name='Doe'
)

# List students
students = client.students.list(
    class_id='...',
    status='active',
    page=1,
    per_page=20
)

# Record payment
payment = client.payments.create(
    invoice_id='...',
    amount=300000,
    payment_method='mobile_money',
    provider='MTN'
)
```

---

## Testing

### Sandbox Environment
```
Base URL: https://api-sandbox.u-usms.ug/v1
```

### Test Credentials
```
Email: test@school.ug
Password: TestPassword123!
Tenant ID: test-tenant-550e8400
```

### Test Payment Methods
- **Mobile Money:** Use `+256700000001` with any amount
- **Bank Transfer:** Reference `TEST-BANK-12345`

---

## Postman Collection

Download: [U-USMS API Postman Collection](https://api.u-usms.ug/postman/collection.json)

---

## Support

- **Documentation:** https://docs.u-usms.ug
- **Support Email:** support@u-usms.ug
- **Developer Forum:** https://community.u-usms.ug
- **Status Page:** https://status.u-usms.ug
