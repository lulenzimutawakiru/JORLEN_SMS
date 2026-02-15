# Database Design - U-USMS

## Overview

The U-USMS database is designed for:
- **Multi-tenancy:** Support for 10,000+ schools
- **Scalability:** 5-8M students
- **Performance:** Optimized queries with proper indexing
- **Compliance:** Audit trails, data sovereignty
- **Flexibility:** Support for various school types

---

## Database Technology

**Primary Database:** PostgreSQL 15+

**Reasons:**
- ACID compliance for financial transactions
- Advanced indexing (B-tree, GiST, GIN)
- JSON/JSONB support for flexible schemas
- Row-Level Security (RLS) for multi-tenancy
- Excellent performance at scale
- Strong community and tooling

---

## Entity Relationship Diagram (ERD)

### Core Entities

```
┌─────────────────┐
│    TENANTS      │
├─────────────────┤
│ PK tenant_id    │──┐
│    name         │  │
│    type         │  │  (One tenant = One school/institution)
│    subscription │  │
│    created_at   │  │
└─────────────────┘  │
                     │
        ┌────────────┴──────────────┬──────────────┬──────────────┐
        │                           │              │              │
        │                           │              │              │
┌───────▼──────────┐    ┌───────────▼────┐   ┌────▼──────────┐  │
│    SCHOOLS       │    │   SCHOOL_      │   │   ACADEMIC    │  │
├──────────────────┤    │   CONFIG       │   │   YEARS       │  │
│ PK school_id     │    ├────────────────┤   ├───────────────┤  │
│ FK tenant_id     │    │ PK config_id   │   │ PK year_id    │  │
│    name          │    │ FK school_id   │   │ FK school_id  │  │
│    type          │    │    key         │   │    name       │  │
│    level         │    │    value       │   │    start_date │  │
│    location      │    └────────────────┘   │    end_date   │  │
│    curriculum    │                         │    is_current │  │
└──────────────────┘                         └───────────────┘  │
        │                                             │          │
        ├─────────────────────────────────────────────┤          │
        │                                             │          │
┌───────▼──────────┐                       ┌──────────▼────────┐│
│    STUDENTS      │                       │     TERMS         ││
├──────────────────┤                       ├───────────────────┤│
│ PK student_id    │                       │ PK term_id        ││
│ FK school_id     │                       │ FK year_id        ││
│ FK tenant_id     │                       │    name           ││
│    first_name    │                       │    start_date     ││
│    last_name     │                       │    end_date       ││
│    dob           │                       └───────────────────┘│
│    gender        │                                            │
│    nin           │ (NIRA ID if available)                     │
│    enrollment_no │                                            │
│    status        │                                            │
│    class_id      │──────────┐                                 │
│    created_at    │          │                                 │
└──────────────────┘          │                                 │
        │                     │                                 │
        │                     │                                 │
┌───────▼──────────┐  ┌───────▼──────────┐                     │
│   GUARDIANS      │  │     CLASSES      │                     │
├──────────────────┤  ├──────────────────┤                     │
│ PK guardian_id   │  │ PK class_id      │                     │
│ FK tenant_id     │  │ FK school_id     │                     │
│    first_name    │  │ FK tenant_id     │                     │
│    last_name     │  │    name          │                     │
│    phone         │  │    level         │                     │
│    email         │  │    stream        │                     │
│    relationship  │  │    capacity      │                     │
│    nin           │  │    year_id       │                     │
└──────────────────┘  └──────────────────┘                     │
        │                      │                                │
        │                      │                                │
┌───────▼──────────────────────▼────────┐                      │
│    STUDENT_GUARDIANS                  │                      │
├───────────────────────────────────────┤                      │
│ PK id                                 │                      │
│ FK student_id                         │                      │
│ FK guardian_id                        │                      │
│ FK tenant_id                          │                      │
│    is_primary                         │                      │
│    can_pickup                         │                      │
└───────────────────────────────────────┘                      │
                                                                │
┌───────────────────────────────────────────────────────────────┘
│
│    ┌──────────────────┐
│    │      STAFF       │
│    ├──────────────────┤
│    │ PK staff_id      │
│    │ FK school_id     │
│    │ FK tenant_id     │
│    │    first_name    │
│    │    last_name     │
│    │    nin           │
│    │    email         │
│    │    phone         │
│    │    role          │ (Teacher, Admin, etc.)
│    │    hire_date     │
│    │    status        │
│    └──────────────────┘
│            │
│            │
│    ┌───────▼─────────────┐
│    │  STAFF_SUBJECTS     │
│    ├─────────────────────┤
│    │ PK id               │
│    │ FK staff_id         │
│    │ FK subject_id       │
│    │ FK class_id         │
│    │ FK tenant_id        │
│    └─────────────────────┘
│            │
│            │
│    ┌───────▼──────────┐
│    │    SUBJECTS      │
│    ├──────────────────┤
│    │ PK subject_id    │
│    │ FK tenant_id     │
│    │    code          │
│    │    name          │
│    │    level         │
│    │    is_compulsory │
│    │    curriculum    │
│    └──────────────────┘
│
│
└─────┐
      │
┌─────▼────────────────────────────────────────────────────────┐
│                  FINANCIAL ENTITIES                           │
├───────────────────────────────────────────────────────────────┤
│                                                               │
│  ┌────────────────────┐      ┌─────────────────────┐         │
│  │   FEE_STRUCTURES   │      │   FEE_CATEGORIES    │         │
│  ├────────────────────┤      ├─────────────────────┤         │
│  │ PK structure_id    │──┐   │ PK category_id      │         │
│  │ FK school_id       │  │   │ FK tenant_id        │         │
│  │ FK tenant_id       │  │   │    name             │         │
│  │    year_id         │  │   │    description      │         │
│  │    class_id        │  │   │    is_recurring     │         │
│  │    category_id     │──┘   └─────────────────────┘         │
│  │    amount          │                                      │
│  │    currency        │                                      │
│  └────────────────────┘                                      │
│           │                                                   │
│           │                                                   │
│  ┌────────▼───────────┐                                      │
│  │    INVOICES        │                                      │
│  ├────────────────────┤                                      │
│  │ PK invoice_id      │                                      │
│  │ FK student_id      │                                      │
│  │ FK tenant_id       │                                      │
│  │    invoice_number  │                                      │
│  │    total_amount    │                                      │
│  │    paid_amount     │                                      │
│  │    balance         │                                      │
│  │    due_date        │                                      │
│  │    status          │                                      │
│  │    created_at      │                                      │
│  └────────────────────┘                                      │
│           │                                                   │
│           │                                                   │
│  ┌────────▼───────────┐                                      │
│  │  INVOICE_ITEMS     │                                      │
│  ├────────────────────┤                                      │
│  │ PK item_id         │                                      │
│  │ FK invoice_id      │                                      │
│  │ FK fee_structure_id│                                      │
│  │ FK tenant_id       │                                      │
│  │    description     │                                      │
│  │    amount          │                                      │
│  │    quantity        │                                      │
│  └────────────────────┘                                      │
│                                                               │
│  ┌────────────────────┐                                      │
│  │     PAYMENTS       │                                      │
│  ├────────────────────┤                                      │
│  │ PK payment_id      │                                      │
│  │ FK invoice_id      │                                      │
│  │ FK student_id      │                                      │
│  │ FK tenant_id       │                                      │
│  │    reference_no    │                                      │
│  │    amount          │                                      │
│  │    payment_method  │ (Cash, Mobile Money, Bank, SchoolPay)│
│  │    provider        │ (MTN, Airtel, Bank name)            │
│  │    transaction_id  │                                      │
│  │    status          │                                      │
│  │    schoolpay_txn_id│                                      │
│  │    paid_at         │                                      │
│  │    verified_at     │                                      │
│  │    created_at      │                                      │
│  └────────────────────┘                                      │
│           │                                                   │
│           │                                                   │
│  ┌────────▼───────────┐                                      │
│  │  PAYMENT_RECEIPTS  │                                      │
│  ├────────────────────┤                                      │
│  │ PK receipt_id      │                                      │
│  │ FK payment_id      │                                      │
│  │ FK tenant_id       │                                      │
│  │    receipt_number  │                                      │
│  │    issued_by       │                                      │
│  │    issued_at       │                                      │
│  │    pdf_url         │                                      │
│  └────────────────────┘                                      │
│                                                               │
│  ┌────────────────────┐                                      │
│  │  LEDGER_ENTRIES    │                                      │
│  ├────────────────────┤                                      │
│  │ PK entry_id        │                                      │
│  │ FK tenant_id       │                                      │
│  │    transaction_id  │                                      │
│  │    account         │                                      │
│  │    debit           │                                      │
│  │    credit          │                                      │
│  │    description     │                                      │
│  │    created_at      │                                      │
│  └────────────────────┘                                      │
│                                                               │
└───────────────────────────────────────────────────────────────┘

┌───────────────────────────────────────────────────────────────┐
│                   PAYROLL ENTITIES                            │
├───────────────────────────────────────────────────────────────┤
│                                                               │
│  ┌────────────────────┐                                      │
│  │   SALARY_SCALES    │                                      │
│  ├────────────────────┤                                      │
│  │ PK scale_id        │                                      │
│  │ FK tenant_id       │                                      │
│  │    name            │ (e.g., "U4", "U5", "Teacher I")     │
│  │    level           │                                      │
│  │    basic_salary    │                                      │
│  │    currency        │                                      │
│  │    effective_date  │                                      │
│  └────────────────────┘                                      │
│           │                                                   │
│           │                                                   │
│  ┌────────▼───────────┐                                      │
│  │  STAFF_SALARIES    │                                      │
│  ├────────────────────┤                                      │
│  │ PK salary_id       │                                      │
│  │ FK staff_id        │                                      │
│  │ FK scale_id        │                                      │
│  │ FK tenant_id       │                                      │
│  │    basic_salary    │                                      │
│  │    housing_allow   │                                      │
│  │    transport_allow │                                      │
│  │    hardship_allow  │                                      │
│  │    other_allow     │                                      │
│  │    effective_date  │                                      │
│  └────────────────────┘                                      │
│                                                               │
│  ┌────────────────────┐                                      │
│  │  PAYROLL_RUNS      │                                      │
│  ├────────────────────┤                                      │
│  │ PK run_id          │                                      │
│  │ FK school_id       │                                      │
│  │ FK tenant_id       │                                      │
│  │    period_month    │                                      │
│  │    period_year     │                                      │
│  │    status          │ (Draft, Approved, Paid)             │
│  │    total_gross     │                                      │
│  │    total_net       │                                      │
│  │    total_paye      │                                      │
│  │    total_nssf_emp  │                                      │
│  │    total_nssf_empr │                                      │
│  │    approved_by     │                                      │
│  │    approved_at     │                                      │
│  │    created_at      │                                      │
│  └────────────────────┘                                      │
│           │                                                   │
│           │                                                   │
│  ┌────────▼───────────┐                                      │
│  │   PAYSLIPS         │                                      │
│  ├────────────────────┤                                      │
│  │ PK payslip_id      │                                      │
│  │ FK run_id          │                                      │
│  │ FK staff_id        │                                      │
│  │ FK tenant_id       │                                      │
│  │    basic_salary    │                                      │
│  │    allowances      │ (JSON: {housing, transport, etc})   │
│  │    gross_salary    │                                      │
│  │    nssf_employee   │                                      │
│  │    nssf_employer   │                                      │
│  │    taxable_income  │                                      │
│  │    paye            │                                      │
│  │    lst             │ (Local Service Tax)                 │
│  │    other_deductions│                                      │
│  │    net_salary      │                                      │
│  │    payment_method  │                                      │
│  │    bank_account    │                                      │
│  │    status          │                                      │
│  │    generated_at    │                                      │
│  └────────────────────┘                                      │
│                                                               │
│  ┌────────────────────┐                                      │
│  │  LEAVE_REQUESTS    │                                      │
│  ├────────────────────┤                                      │
│  │ PK request_id      │                                      │
│  │ FK staff_id        │                                      │
│  │ FK tenant_id       │                                      │
│  │    leave_type      │ (Annual, Sick, Maternity, etc)      │
│  │    start_date      │                                      │
│  │    end_date        │                                      │
│  │    days_requested  │                                      │
│  │    status          │ (Pending, Approved, Rejected)       │
│  │    approved_by     │                                      │
│  │    approved_at     │                                      │
│  │    created_at      │                                      │
│  └────────────────────┘                                      │
│                                                               │
│  ┌────────────────────┐                                      │
│  │  LEAVE_BALANCES    │                                      │
│  ├────────────────────┤                                      │
│  │ PK balance_id      │                                      │
│  │ FK staff_id        │                                      │
│  │ FK tenant_id       │                                      │
│  │    leave_type      │                                      │
│  │    year            │                                      │
│  │    entitled_days   │                                      │
│  │    used_days       │                                      │
│  │    remaining_days  │                                      │
│  └────────────────────┘                                      │
│                                                               │
└───────────────────────────────────────────────────────────────┘

┌───────────────────────────────────────────────────────────────┐
│                   ACADEMIC ENTITIES                           │
├───────────────────────────────────────────────────────────────┤
│                                                               │
│  ┌────────────────────┐                                      │
│  │   ASSESSMENTS      │                                      │
│  ├────────────────────┤                                      │
│  │ PK assessment_id   │                                      │
│  │ FK subject_id      │                                      │
│  │ FK class_id        │                                      │
│  │ FK term_id         │                                      │
│  │ FK tenant_id       │                                      │
│  │    name            │                                      │
│  │    type            │ (Exam, Test, Practical, Coursework) │
│  │    total_marks     │                                      │
│  │    weight          │ (Percentage contribution)           │
│  │    date            │                                      │
│  └────────────────────┘                                      │
│           │                                                   │
│           │                                                   │
│  ┌────────▼───────────┐                                      │
│  │  STUDENT_SCORES    │                                      │
│  ├────────────────────┤                                      │
│  │ PK score_id        │                                      │
│  │ FK assessment_id   │                                      │
│  │ FK student_id      │                                      │
│  │ FK tenant_id       │                                      │
│  │    marks_obtained  │                                      │
│  │    grade           │                                      │
│  │    remarks         │                                      │
│  │    entered_by      │                                      │
│  │    entered_at      │                                      │
│  └────────────────────┘                                      │
│                                                               │
│  ┌────────────────────┐                                      │
│  │   RESULTS          │                                      │
│  ├────────────────────┤                                      │
│  │ PK result_id       │                                      │
│  │ FK student_id      │                                      │
│  │ FK term_id         │                                      │
│  │ FK tenant_id       │                                      │
│  │    total_marks     │                                      │
│  │    average         │                                      │
│  │    grade           │                                      │
│  │    division        │ (For UCE/UACE/PLE)                  │
│  │    rank_in_class   │                                      │
│  │    rank_in_stream  │                                      │
│  │    remarks         │                                      │
│  │    approved_by     │                                      │
│  │    approved_at     │                                      │
│  └────────────────────┘                                      │
│                                                               │
│  ┌────────────────────┐                                      │
│  │   ATTENDANCE       │                                      │
│  ├────────────────────┤                                      │
│  │ PK attendance_id   │                                      │
│  │ FK student_id      │                                      │
│  │ FK class_id        │                                      │
│  │ FK tenant_id       │                                      │
│  │    date            │                                      │
│  │    status          │ (Present, Absent, Late, Excused)    │
│  │    remarks         │                                      │
│  │    marked_by       │                                      │
│  │    marked_at       │                                      │
│  └────────────────────┘                                      │
│                                                               │
│  ┌────────────────────┐                                      │
│  │   TIMETABLE        │                                      │
│  ├────────────────────┤                                      │
│  │ PK timetable_id    │                                      │
│  │ FK class_id        │                                      │
│  │ FK subject_id      │                                      │
│  │ FK staff_id        │                                      │
│  │ FK tenant_id       │                                      │
│  │    day_of_week     │                                      │
│  │    period_number   │                                      │
│  │    start_time      │                                      │
│  │    end_time        │                                      │
│  │    room            │                                      │
│  └────────────────────┘                                      │
│                                                               │
└───────────────────────────────────────────────────────────────┘

┌───────────────────────────────────────────────────────────────┐
│                   AI & ANALYTICS ENTITIES                     │
├───────────────────────────────────────────────────────────────┤
│                                                               │
│  ┌────────────────────┐                                      │
│  │  AI_PREDICTIONS    │                                      │
│  ├────────────────────┤                                      │
│  │ PK prediction_id   │                                      │
│  │ FK student_id      │                                      │
│  │ FK tenant_id       │                                      │
│  │    prediction_type │ (Performance, Dropout, Risk)        │
│  │    confidence      │                                      │
│  │    predicted_value │                                      │
│  │    factors         │ (JSON array of contributing factors)│
│  │    generated_at    │                                      │
│  │    model_version   │                                      │
│  └────────────────────┘                                      │
│                                                               │
│  ┌────────────────────┐                                      │
│  │  RISK_SCORES       │                                      │
│  ├────────────────────┤                                      │
│  │ PK score_id        │                                      │
│  │ FK student_id      │                                      │
│  │ FK tenant_id       │                                      │
│  │    risk_type       │ (Dropout, Performance, Behavioral)  │
│  │    score           │ (0-100, higher = higher risk)       │
│  │    severity        │ (Low, Medium, High, Critical)       │
│  │    last_updated    │                                      │
│  │    interventions   │ (JSON array of recommended actions) │
│  └────────────────────┘                                      │
│                                                               │
│  ┌────────────────────┐                                      │
│  │   INTERVENTIONS    │                                      │
│  ├────────────────────┤                                      │
│  │ PK intervention_id │                                      │
│  │ FK student_id      │                                      │
│  │ FK risk_score_id   │                                      │
│  │ FK tenant_id       │                                      │
│  │    type            │                                      │
│  │    description     │                                      │
│  │    assigned_to     │                                      │
│  │    status          │                                      │
│  │    outcome         │                                      │
│  │    started_at      │                                      │
│  │    completed_at    │                                      │
│  └────────────────────┘                                      │
│                                                               │
└───────────────────────────────────────────────────────────────┘

┌───────────────────────────────────────────────────────────────┐
│                   SECURITY & AUDIT ENTITIES                   │
├───────────────────────────────────────────────────────────────┤
│                                                               │
│  ┌────────────────────┐                                      │
│  │      USERS         │                                      │
│  ├────────────────────┤                                      │
│  │ PK user_id         │                                      │
│  │ FK tenant_id       │                                      │
│  │    username        │                                      │
│  │    email           │                                      │
│  │    password_hash   │                                      │
│  │    role            │                                      │
│  │    mfa_enabled     │                                      │
│  │    mfa_secret      │                                      │
│  │    last_login      │                                      │
│  │    is_active       │                                      │
│  │    created_at      │                                      │
│  └────────────────────┘                                      │
│           │                                                   │
│           │                                                   │
│  ┌────────▼───────────┐                                      │
│  │  USER_PERMISSIONS  │                                      │
│  ├────────────────────┤                                      │
│  │ PK permission_id   │                                      │
│  │ FK user_id         │                                      │
│  │ FK tenant_id       │                                      │
│  │    resource        │                                      │
│  │    action          │ (Create, Read, Update, Delete)      │
│  │    granted_at      │                                      │
│  └────────────────────┘                                      │
│                                                               │
│  ┌────────────────────┐                                      │
│  │   AUDIT_LOGS       │                                      │
│  ├────────────────────┤                                      │
│  │ PK log_id          │                                      │
│  │ FK user_id         │                                      │
│  │ FK tenant_id       │                                      │
│  │    action          │                                      │
│  │    resource_type   │                                      │
│  │    resource_id     │                                      │
│  │    old_values      │ (JSON)                              │
│  │    new_values      │ (JSON)                              │
│  │    ip_address      │                                      │
│  │    user_agent      │                                      │
│  │    timestamp       │                                      │
│  └────────────────────┘                                      │
│                                                               │
│  ┌────────────────────┐                                      │
│  │  SESSION_LOGS      │                                      │
│  ├────────────────────┤                                      │
│  │ PK session_id      │                                      │
│  │ FK user_id         │                                      │
│  │ FK tenant_id       │                                      │
│  │    token_hash      │                                      │
│  │    ip_address      │                                      │
│  │    device_info     │                                      │
│  │    started_at      │                                      │
│  │    expires_at      │                                      │
│  │    last_activity   │                                      │
│  └────────────────────┘                                      │
│                                                               │
└───────────────────────────────────────────────────────────────┘

┌───────────────────────────────────────────────────────────────┐
│               COMMUNICATION ENTITIES                          │
├───────────────────────────────────────────────────────────────┤
│                                                               │
│  ┌────────────────────┐                                      │
│  │  SMS_LOGS          │                                      │
│  ├────────────────────┤                                      │
│  │ PK sms_id          │                                      │
│  │ FK tenant_id       │                                      │
│  │    recipient       │                                      │
│  │    message         │                                      │
│  │    type            │ (Fee, Result, Alert, etc)           │
│  │    status          │ (Pending, Sent, Failed, Delivered)  │
│  │    provider        │                                      │
│  │    cost            │                                      │
│  │    sent_at         │                                      │
│  │    delivered_at    │                                      │
│  └────────────────────┘                                      │
│                                                               │
│  ┌────────────────────┐                                      │
│  │  NOTIFICATIONS     │                                      │
│  ├────────────────────┤                                      │
│  │ PK notification_id │                                      │
│  │ FK user_id         │                                      │
│  │ FK tenant_id       │                                      │
│  │    title           │                                      │
│  │    message         │                                      │
│  │    type            │                                      │
│  │    priority        │                                      │
│  │    is_read         │                                      │
│  │    created_at      │                                      │
│  │    read_at         │                                      │
│  └────────────────────┘                                      │
│                                                               │
└───────────────────────────────────────────────────────────────┘
```

---

## Key Design Decisions

### 1. Multi-Tenancy Implementation

**Every table includes `tenant_id` for data isolation:**
```sql
CREATE TABLE students (
    student_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL REFERENCES tenants(tenant_id) ON DELETE CASCADE,
    -- other fields...
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Row-Level Security Policy
ALTER TABLE students ENABLE ROW LEVEL SECURITY;

CREATE POLICY tenant_isolation ON students
    USING (tenant_id = current_setting('app.current_tenant')::uuid);
```

### 2. Partitioning Strategy

**Partition large tables by tenant_id for performance:**
```sql
-- Partition audit_logs by tenant_id (hash partitioning)
CREATE TABLE audit_logs (
    log_id BIGSERIAL,
    tenant_id UUID NOT NULL,
    -- other fields...
    timestamp TIMESTAMP NOT NULL
) PARTITION BY HASH (tenant_id);

-- Create partitions
CREATE TABLE audit_logs_p0 PARTITION OF audit_logs FOR VALUES WITH (MODULUS 10, REMAINDER 0);
CREATE TABLE audit_logs_p1 PARTITION OF audit_logs FOR VALUES WITH (MODULUS 10, REMAINDER 1);
-- ... up to p9
```

**Partition time-series data by date:**
```sql
-- Partition attendance by month
CREATE TABLE attendance (
    attendance_id BIGSERIAL,
    -- other fields...
    date DATE NOT NULL
) PARTITION BY RANGE (date);

-- Create partitions per month
CREATE TABLE attendance_2026_01 PARTITION OF attendance
    FOR VALUES FROM ('2026-01-01') TO ('2026-02-01');
CREATE TABLE attendance_2026_02 PARTITION OF attendance
    FOR VALUES FROM ('2026-02-01') TO ('2026-03-01');
-- Automated partition creation via cron
```

### 3. Indexing Strategy

**Critical indexes for performance:**
```sql
-- Tenant-based queries (most common)
CREATE INDEX idx_students_tenant ON students(tenant_id);
CREATE INDEX idx_payments_tenant ON payments(tenant_id);

-- Composite indexes for common queries
CREATE INDEX idx_students_tenant_class ON students(tenant_id, class_id);
CREATE INDEX idx_payments_tenant_student ON payments(tenant_id, student_id, created_at DESC);
CREATE INDEX idx_invoices_tenant_status ON invoices(tenant_id, status, due_date);

-- Full-text search
CREATE INDEX idx_students_name_search ON students USING GIN(
    to_tsvector('english', first_name || ' ' || last_name)
);

-- JSONB indexes for flexible schemas
CREATE INDEX idx_payslips_allowances ON payslips USING GIN(allowances);

-- Partial indexes for active records
CREATE INDEX idx_students_active ON students(tenant_id, class_id) WHERE status = 'active';
```

### 4. Data Types

- **UUIDs:** Primary keys for distributed system compatibility
- **JSONB:** Flexible schemas (allowances, configurations, metadata)
- **BIGSERIAL:** Auto-incrementing for high-volume tables
- **TIMESTAMP WITH TIME ZONE:** Consistent time tracking
- **NUMERIC(15,2):** Currency values (avoid floating-point issues)

### 5. Soft Deletes

**Never physically delete critical records:**
```sql
ALTER TABLE students ADD COLUMN deleted_at TIMESTAMP;

-- Query only active records
CREATE VIEW active_students AS
SELECT * FROM students WHERE deleted_at IS NULL;
```

### 6. Audit Trail

**Track all changes for compliance:**
```sql
-- Trigger function to log changes
CREATE OR REPLACE FUNCTION audit_trigger_func()
RETURNS TRIGGER AS $$
BEGIN
    INSERT INTO audit_logs (
        user_id,
        tenant_id,
        action,
        resource_type,
        resource_id,
        old_values,
        new_values,
        timestamp
    ) VALUES (
        current_setting('app.current_user')::uuid,
        current_setting('app.current_tenant')::uuid,
        TG_OP,
        TG_TABLE_NAME,
        COALESCE(NEW.id, OLD.id),
        CASE WHEN TG_OP IN ('UPDATE', 'DELETE') THEN row_to_json(OLD) END,
        CASE WHEN TG_OP IN ('INSERT', 'UPDATE') THEN row_to_json(NEW) END,
        CURRENT_TIMESTAMP
    );
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

-- Apply to critical tables
CREATE TRIGGER audit_students AFTER INSERT OR UPDATE OR DELETE ON students
    FOR EACH ROW EXECUTE FUNCTION audit_trigger_func();
```

---

## Sample SQL Schema

```sql
-- =====================================================
-- U-USMS Database Schema
-- PostgreSQL 15+
-- =====================================================

-- Enable required extensions
CREATE EXTENSION IF NOT EXISTS "uuid-ossp";
CREATE EXTENSION IF NOT EXISTS "pg_trgm";  -- Fuzzy text search
CREATE EXTENSION IF NOT EXISTS "btree_gin"; -- Multi-column GIN indexes

-- =====================================================
-- TENANTS
-- =====================================================

CREATE TABLE tenants (
    tenant_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name VARCHAR(255) NOT NULL,
    type VARCHAR(50) NOT NULL, -- 'private', 'government', 'network'
    subscription_tier VARCHAR(50) DEFAULT 'basic',
    subscription_status VARCHAR(20) DEFAULT 'active',
    max_students INT,
    max_staff INT,
    features JSONB DEFAULT '{}',
    settings JSONB DEFAULT '{}',
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    deleted_at TIMESTAMP WITH TIME ZONE
);

CREATE INDEX idx_tenants_status ON tenants(subscription_status) WHERE deleted_at IS NULL;

-- =====================================================
-- SCHOOLS
-- =====================================================

CREATE TABLE schools (
    school_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL REFERENCES tenants(tenant_id) ON DELETE CASCADE,
    name VARCHAR(255) NOT NULL,
    code VARCHAR(50) UNIQUE,
    type VARCHAR(50) NOT NULL, -- 'nursery', 'primary', 'secondary', 'vocational', 'tertiary'
    level VARCHAR(50), -- 'basic', 'ordinary', 'advanced'
    curriculum VARCHAR(50)[] DEFAULT ARRAY['UNEB'], -- Array: ['UNEB', 'Cambridge', 'IB']
    motto TEXT,
    logo_url TEXT,
    address TEXT,
    district VARCHAR(100),
    region VARCHAR(100),
    phone VARCHAR(20),
    email VARCHAR(255),
    website VARCHAR(255),
    registration_number VARCHAR(100),
    ownership VARCHAR(50), -- 'government', 'private', 'religious', 'ngo'
    status VARCHAR(20) DEFAULT 'active',
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    deleted_at TIMESTAMP WITH TIME ZONE
);

CREATE INDEX idx_schools_tenant ON schools(tenant_id);
CREATE INDEX idx_schools_type ON schools(type);
CREATE INDEX idx_schools_code ON schools(code);

-- =====================================================
-- ACADEMIC YEARS
-- =====================================================

CREATE TABLE academic_years (
    year_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    school_id UUID NOT NULL REFERENCES schools(school_id) ON DELETE CASCADE,
    tenant_id UUID NOT NULL REFERENCES tenants(tenant_id) ON DELETE CASCADE,
    name VARCHAR(100) NOT NULL, -- e.g., "2026"
    start_date DATE NOT NULL,
    end_date DATE NOT NULL,
    is_current BOOLEAN DEFAULT FALSE,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    UNIQUE(school_id, name)
);

CREATE INDEX idx_academic_years_school ON academic_years(school_id);
CREATE INDEX idx_academic_years_current ON academic_years(school_id, is_current) WHERE is_current = TRUE;

-- =====================================================
-- TERMS
-- =====================================================

CREATE TABLE terms (
    term_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    year_id UUID NOT NULL REFERENCES academic_years(year_id) ON DELETE CASCADE,
    tenant_id UUID NOT NULL REFERENCES tenants(tenant_id) ON DELETE CASCADE,
    name VARCHAR(50) NOT NULL, -- 'Term 1', 'Term 2', 'Term 3', 'Semester 1', etc.
    start_date DATE NOT NULL,
    end_date DATE NOT NULL,
    is_current BOOLEAN DEFAULT FALSE,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_terms_year ON terms(year_id);
CREATE INDEX idx_terms_current ON terms(year_id, is_current) WHERE is_current = TRUE;

-- =====================================================
-- CLASSES
-- =====================================================

CREATE TABLE classes (
    class_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    school_id UUID NOT NULL REFERENCES schools(school_id) ON DELETE CASCADE,
    tenant_id UUID NOT NULL REFERENCES tenants(tenant_id) ON DELETE CASCADE,
    year_id UUID NOT NULL REFERENCES academic_years(year_id) ON DELETE CASCADE,
    name VARCHAR(100) NOT NULL, -- 'Primary 1', 'Senior 1', 'Form 1A', etc.
    level VARCHAR(50), -- 'P1', 'P2', ..., 'S1', 'S2', ..., 'Year 1', etc.
    stream VARCHAR(10), -- 'A', 'B', 'C', etc.
    capacity INT DEFAULT 50,
    current_enrollment INT DEFAULT 0,
    class_teacher_id UUID, -- References staff
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    deleted_at TIMESTAMP WITH TIME ZONE,
    UNIQUE(school_id, year_id, name)
);

CREATE INDEX idx_classes_school ON classes(school_id);
CREATE INDEX idx_classes_tenant ON classes(tenant_id);
CREATE INDEX idx_classes_year ON classes(year_id);

-- =====================================================
-- STUDENTS
-- =====================================================

CREATE TABLE students (
    student_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    school_id UUID NOT NULL REFERENCES schools(school_id) ON DELETE CASCADE,
    tenant_id UUID NOT NULL REFERENCES tenants(tenant_id) ON DELETE CASCADE,
    enrollment_number VARCHAR(50) UNIQUE NOT NULL,
    class_id UUID REFERENCES classes(class_id),
    first_name VARCHAR(100) NOT NULL,
    last_name VARCHAR(100) NOT NULL,
    middle_name VARCHAR(100),
    date_of_birth DATE NOT NULL,
    gender VARCHAR(10) NOT NULL, -- 'Male', 'Female', 'Other'
    nin VARCHAR(20), -- National ID Number (NIRA)
    photo_url TEXT,
    blood_group VARCHAR(5),
    medical_conditions TEXT,
    allergies TEXT,
    status VARCHAR(20) DEFAULT 'active', -- 'active', 'suspended', 'graduated', 'transferred', 'dropped'
    admission_date DATE NOT NULL,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    deleted_at TIMESTAMP WITH TIME ZONE
);

CREATE INDEX idx_students_tenant ON students(tenant_id);
CREATE INDEX idx_students_school ON students(school_id);
CREATE INDEX idx_students_class ON students(class_id);
CREATE INDEX idx_students_enrollment ON students(enrollment_number);
CREATE INDEX idx_students_nin ON students(nin);
CREATE INDEX idx_students_status ON students(status) WHERE status = 'active';
CREATE INDEX idx_students_name_search ON students USING GIN(
    to_tsvector('english', first_name || ' ' || last_name)
);

-- Enable Row-Level Security
ALTER TABLE students ENABLE ROW LEVEL SECURITY;
CREATE POLICY tenant_isolation ON students
    USING (tenant_id = current_setting('app.current_tenant')::uuid);

-- =====================================================
-- GUARDIANS
-- =====================================================

CREATE TABLE guardians (
    guardian_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL REFERENCES tenants(tenant_id) ON DELETE CASCADE,
    first_name VARCHAR(100) NOT NULL,
    last_name VARCHAR(100) NOT NULL,
    nin VARCHAR(20),
    phone VARCHAR(20) NOT NULL,
    email VARCHAR(255),
    occupation VARCHAR(100),
    employer VARCHAR(255),
    address TEXT,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    deleted_at TIMESTAMP WITH TIME ZONE
);

CREATE INDEX idx_guardians_tenant ON guardians(tenant_id);
CREATE INDEX idx_guardians_phone ON guardians(phone);
CREATE INDEX idx_guardians_nin ON guardians(nin);

-- =====================================================
-- STUDENT-GUARDIAN RELATIONSHIP
-- =====================================================

CREATE TABLE student_guardians (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    student_id UUID NOT NULL REFERENCES students(student_id) ON DELETE CASCADE,
    guardian_id UUID NOT NULL REFERENCES guardians(guardian_id) ON DELETE CASCADE,
    tenant_id UUID NOT NULL REFERENCES tenants(tenant_id) ON DELETE CASCADE,
    relationship VARCHAR(50) NOT NULL, -- 'Father', 'Mother', 'Guardian', 'Sibling', etc.
    is_primary BOOLEAN DEFAULT FALSE,
    can_pickup BOOLEAN DEFAULT TRUE,
    emergency_contact BOOLEAN DEFAULT FALSE,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    UNIQUE(student_id, guardian_id)
);

CREATE INDEX idx_student_guardians_student ON student_guardians(student_id);
CREATE INDEX idx_student_guardians_guardian ON student_guardians(guardian_id);

-- =====================================================
-- STAFF
-- =====================================================

CREATE TABLE staff (
    staff_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    school_id UUID NOT NULL REFERENCES schools(school_id) ON DELETE CASCADE,
    tenant_id UUID NOT NULL REFERENCES tenants(tenant_id) ON DELETE CASCADE,
    employee_number VARCHAR(50) UNIQUE NOT NULL,
    first_name VARCHAR(100) NOT NULL,
    last_name VARCHAR(100) NOT NULL,
    middle_name VARCHAR(100),
    nin VARCHAR(20) UNIQUE,
    date_of_birth DATE NOT NULL,
    gender VARCHAR(10) NOT NULL,
    photo_url TEXT,
    email VARCHAR(255) UNIQUE,
    phone VARCHAR(20) NOT NULL,
    address TEXT,
    role VARCHAR(50) NOT NULL, -- 'Teacher', 'Admin', 'Principal', 'HOD', 'Accountant', etc.
    department VARCHAR(100),
    hire_date DATE NOT NULL,
    contract_type VARCHAR(50), -- 'Permanent', 'Contract', 'Part-time'
    qualification VARCHAR(255),
    bank_name VARCHAR(100),
    bank_account_number VARCHAR(50),
    bank_account_name VARCHAR(255),
    bank_branch VARCHAR(100),
    nssf_number VARCHAR(50),
    tin_number VARCHAR(50), -- URA TIN
    status VARCHAR(20) DEFAULT 'active', -- 'active', 'on_leave', 'suspended', 'terminated'
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    deleted_at TIMESTAMP WITH TIME ZONE
);

CREATE INDEX idx_staff_tenant ON staff(tenant_id);
CREATE INDEX idx_staff_school ON staff(school_id);
CREATE INDEX idx_staff_employee_number ON staff(employee_number);
CREATE INDEX idx_staff_nin ON staff(nin);
CREATE INDEX idx_staff_role ON staff(role);

ALTER TABLE staff ENABLE ROW LEVEL SECURITY;
CREATE POLICY tenant_isolation ON staff
    USING (tenant_id = current_setting('app.current_tenant')::uuid);

-- Continue in next section...
```

---

## Database Performance Optimization

### Query Optimization Tips

1. **Always filter by tenant_id first:**
```sql
-- GOOD: Uses index efficiently
SELECT * FROM students
WHERE tenant_id = '...' AND class_id = '...'

-- BAD: Slow without tenant_id
SELECT * FROM students WHERE class_id = '...'
```

2. **Use EXPLAIN ANALYZE for slow queries:**
```sql
EXPLAIN ANALYZE
SELECT s.*, g.phone
FROM students s
JOIN student_guardians sg ON s.student_id = sg.student_id
JOIN guardians g ON sg.guardian_id = g.guardian_id
WHERE s.tenant_id = '...' AND s.status = 'active';
```

3. **Batch operations for bulk inserts:**
```sql
-- Use COPY or multi-row INSERT
INSERT INTO students (tenant_id, school_id, ...)
VALUES
    ('...', '...', ...),
    ('...', '...', ...),
    -- up to 1000 rows per batch
ON CONFLICT (enrollment_number) DO NOTHING;
```

### Connection Pooling

```javascript
// Node.js with pg-pool
const { Pool } = require('pg');

const pool = new Pool({
  host: process.env.DB_HOST,
  port: 5432,
  database: process.env.DB_NAME,
  user: process.env.DB_USER,
  password: process.env.DB_PASSWORD,
  max: 100, // Max connections
  idleTimeoutMillis: 30000,
  connectionTimeoutMillis: 2000,
});
```

---

## Data Migration Strategy

### Initial Data Load
1. **Government schools:** Bulk import from Ministry of Education
2. **Private schools:** CSV/Excel upload interface
3. **Data validation:** Pre-migration checks
4. **Rollback plan:** Database snapshots before migration

### Schema Versioning
```sql
CREATE TABLE schema_migrations (
    version VARCHAR(50) PRIMARY KEY,
    description TEXT,
    applied_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

---

## Backup & Recovery

### Backup Schedule
- **Full backup:** Daily at 02:00 EAT
- **Incremental:** Every 6 hours
- **Retention:** 30 days online, 1 year S3 Glacier

### Recovery Procedures
```bash
# Point-in-time recovery
pg_restore -d usms_db -T tenants -T schools --clean backup_file.dump

# Specific tenant recovery
pg_restore -d usms_db -t students -t guardians \
  --data-only --where="tenant_id='<UUID>'" backup_file.dump
```

---

## Next Steps

See:
- API Design: `docs/api/03_API_SPECIFICATION.md`
- Payroll System: `docs/compliance/04_PAYROLL_SYSTEM.md`
- SchoolPay Integration: `docs/api/05_SCHOOLPAY_INTEGRATION.md`
