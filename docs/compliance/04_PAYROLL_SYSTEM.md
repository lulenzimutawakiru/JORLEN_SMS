# Uganda-Compliant Payroll System - U-USMS

## Overview

The U-USMS Payroll System is fully compliant with Ugandan tax and labor regulations.

---

## Regulatory Compliance

### 1. Uganda Revenue Authority (URA) - PAYE
### 2. National Social Security Fund (NSSF)
### 3. Local Service Tax (LST)
### 4. Employment Act of Uganda
### 5. Public Service Commission (PSC) salary scales (for government schools)

---

## URA PAYE Tax Calculation

### PAYE Tax Bands (2026)

| Taxable Income (UGX/Month) | Tax Rate |
|---------------------------|----------|
| 0 - 235,000 | 0% (Tax-free) |
| 235,001 - 335,000 | 10% |
| 335,001 - 410,000 | 20% |
| 410,001 - 10,000,000 | 30% |
| Above 10,000,000 | 40% |

### PAYE Calculation Algorithm

```javascript
function calculatePAYE(taxableIncome) {
  let tax = 0;
  
  if (taxableIncome <= 235000) {
    tax = 0;
  } else if (taxableIncome <= 335000) {
    tax = (taxableIncome - 235000) * 0.10;
  } else if (taxableIncome <= 410000) {
    tax = (100000 * 0.10) + ((taxableIncome - 335000) * 0.20);
  } else if (taxableIncome <= 10000000) {
    tax = (100000 * 0.10) + (75000 * 0.20) + ((taxableIncome - 410000) * 0.30);
  } else {
    tax = (100000 * 0.10) + (75000 * 0.20) + (9590000 * 0.30) + ((taxableIncome - 10000000) * 0.40);
  }
  
  return Math.round(tax);
}
```

### Tax Relief

**Standard Relief:** UGX 235,000 per month (automatically applied)

**Additional Reliefs:**
- Spouse relief (if applicable)
- Child education relief
- Medical insurance relief

---

## NSSF Contributions

### Contribution Rates

- **Employee Contribution:** 5% of gross salary
- **Employer Contribution:** 10% of gross salary
- **Maximum Ceiling:** UGX 3,000,000 per month (contributions capped)

### NSSF Calculation

```javascript
function calculateNSSF(grossSalary) {
  const NSSF_MAX_CEILING = 3000000;
  const contributableSalary = Math.min(grossSalary, NSSF_MAX_CEILING);
  
  return {
    employee: Math.round(contributableSalary * 0.05),
    employer: Math.round(contributableSalary * 0.10)
  };
}
```

---

## Local Service Tax (LST)

**Amount:** Variable by district (typically UGX 2,000 - 30,000 per year)

**Application:** Deducted monthly, remitted annually to district

---

## Gross-to-Net Calculation Engine

### Complete Payroll Calculation

```javascript
class PayrollCalculator {
  constructor(staff) {
    this.staff = staff;
  }
  
  calculatePayslip(month, year) {
    // Step 1: Calculate Gross Salary
    const basicSalary = this.staff.basic_salary;
    const housingAllowance = this.staff.housing_allowance || 0;
    const transportAllowance = this.staff.transport_allowance || 0;
    const hardshipAllowance = this.staff.hardship_allowance || 0;
    const otherAllowances = this.staff.other_allowances || 0;
    
    const grossSalary = basicSalary + housingAllowance + transportAllowance 
                      + hardshipAllowance + otherAllowances;
    
    // Step 2: Calculate NSSF (Employee)
    const nssfEmployee = this.calculateNSSFEmployee(grossSalary);
    
    // Step 3: Calculate Taxable Income
    const taxableIncome = grossSalary - nssfEmployee;
    
    // Step 4: Calculate PAYE
    const paye = this.calculatePAYE(taxableIncome);
    
    // Step 5: Calculate LST (monthly portion)
    const lst = this.staff.lst_annual ? Math.round(this.staff.lst_annual / 12) : 0;
    
    // Step 6: Other Deductions
    const otherDeductions = this.staff.other_deductions || 0;
    
    // Step 7: Calculate Net Salary
    const totalDeductions = nssfEmployee + paye + lst + otherDeductions;
    const netSalary = grossSalary - totalDeductions;
    
    // Step 8: Calculate Employer Contributions
    const nssfEmployer = this.calculateNSSFEmployer(grossSalary);
    
    return {
      staff_id: this.staff.staff_id,
      employee_number: this.staff.employee_number,
      name: `${this.staff.first_name} ${this.staff.last_name}`,
      period: `${month}/${year}`,
      
      earnings: {
        basic_salary: basicSalary,
        housing_allowance: housingAllowance,
        transport_allowance: transportAllowance,
        hardship_allowance: hardshipAllowance,
        other_allowances: otherAllowances,
        gross_salary: grossSalary
      },
      
      deductions: {
        nssf_employee: nssfEmployee,
        paye: paye,
        lst: lst,
        other_deductions: otherDeductions,
        total_deductions: totalDeductions
      },
      
      net_salary: netSalary,
      
      employer_contributions: {
        nssf_employer: nssfEmployer,
        total_employer_cost: grossSalary + nssfEmployer
      }
    };
  }
  
  calculateNSSFEmployee(grossSalary) {
    const NSSF_MAX_CEILING = 3000000;
    const contributableSalary = Math.min(grossSalary, NSSF_MAX_CEILING);
    return Math.round(contributableSalary * 0.05);
  }
  
  calculateNSSFEmployer(grossSalary) {
    const NSSF_MAX_CEILING = 3000000;
    const contributableSalary = Math.min(grossSalary, NSSF_MAX_CEILING);
    return Math.round(contributableSalary * 0.10);
  }
  
  calculatePAYE(taxableIncome) {
    let tax = 0;
    
    if (taxableIncome <= 235000) {
      tax = 0;
    } else if (taxableIncome <= 335000) {
      tax = (taxableIncome - 235000) * 0.10;
    } else if (taxableIncome <= 410000) {
      tax = (100000 * 0.10) + ((taxableIncome - 335000) * 0.20);
    } else if (taxableIncome <= 10000000) {
      tax = (100000 * 0.10) + (75000 * 0.20) + ((taxableIncome - 410000) * 0.30);
    } else {
      tax = (100000 * 0.10) + (75000 * 0.20) + (9590000 * 0.30) + ((taxableIncome - 10000000) * 0.40);
    }
    
    return Math.round(tax);
  }
}
```

---

## Sample Payslip Calculation

### Example 1: Standard Teacher

**Input:**
- Basic Salary: UGX 2,000,000
- Housing Allowance: UGX 300,000
- Transport Allowance: UGX 150,000
- LST Annual: UGX 24,000

**Calculation:**

```
STEP 1: Gross Salary
Basic:     2,000,000
Housing:     300,000
Transport:   150,000
-----------------------
Gross:     2,450,000

STEP 2: NSSF Employee (5%)
2,450,000 × 0.05 = 122,500

STEP 3: Taxable Income
2,450,000 - 122,500 = 2,327,500

STEP 4: PAYE
Tax on first 235,000:           0
Tax on 100,000 (235,001-335,000): 10,000  (100,000 × 10%)
Tax on 75,000 (335,001-410,000):  15,000  (75,000 × 20%)
Tax on 1,917,500 (410,001-...):   575,250 (1,917,500 × 30%)
-----------------------------------------------------
Total PAYE:                      600,250

STEP 5: LST (Monthly)
24,000 / 12 = 2,000

STEP 6: Total Deductions
NSSF Employee: 122,500
PAYE:          600,250
LST:             2,000
----------------------
Total:         724,750

STEP 7: Net Salary
2,450,000 - 724,750 = 1,725,250

STEP 8: Employer Contributions
NSSF Employer (10%): 2,450,000 × 0.10 = 245,000
Total Employer Cost: 2,450,000 + 245,000 = 2,695,000
```

**Final Payslip:**

```
===============================================
         UNIVERSAL SCHOOL MANAGEMENT
              PAYSLIP - FEB 2026
===============================================

Employee: Jane Smith
Employee No: EMP2026001
Position: Senior Teacher
Department: Mathematics

-----------------------------------------------
EARNINGS:
-----------------------------------------------
Basic Salary            UGX 2,000,000.00
Housing Allowance       UGX   300,000.00
Transport Allowance     UGX   150,000.00
-----------------------------------------------
GROSS SALARY            UGX 2,450,000.00

-----------------------------------------------
DEDUCTIONS:
-----------------------------------------------
NSSF (5%)               UGX   122,500.00
PAYE                    UGX   600,250.00
LST                     UGX     2,000.00
-----------------------------------------------
TOTAL DEDUCTIONS        UGX   724,750.00

-----------------------------------------------
NET SALARY              UGX 1,725,250.00
===============================================

EMPLOYER CONTRIBUTIONS:
NSSF (10%)              UGX   245,000.00
Total Employer Cost     UGX 2,695,000.00

-----------------------------------------------
PAYMENT DETAILS:
Bank: Stanbic Bank Uganda
Account: 1234567890
Account Name: Jane Smith

Generated: 15-Feb-2026
Paid: 28-Feb-2026
===============================================
```

### Example 2: Senior Administrator

**Input:**
- Basic Salary: UGX 5,000,000
- Housing Allowance: UGX 800,000
- Transport Allowance: UGX 300,000
- Hardship Allowance: UGX 200,000

**Calculation:**

```
Gross: 5,000,000 + 800,000 + 300,000 + 200,000 = 6,300,000

NSSF Employee: 3,000,000 × 0.05 = 150,000 (capped at 3M ceiling)

Taxable Income: 6,300,000 - 150,000 = 6,150,000

PAYE:
0-235,000:           0
235,001-335,000:     10,000  (100k × 10%)
335,001-410,000:     15,000  (75k × 20%)
410,001-6,150,000:   1,722,000 (5,740,000 × 30%)
----------------------------------------------
Total PAYE:          1,747,000

Net Salary: 6,300,000 - 150,000 - 1,747,000 = 4,403,000

Employer NSSF: 3,000,000 × 0.10 = 300,000 (capped)
Total Cost: 6,300,000 + 300,000 = 6,600,000
```

---

## Allowances Structure

### Common Allowances in Uganda Education Sector

| Allowance Type | Typical Amount | Taxable? |
|----------------|----------------|----------|
| Housing | 15-30% of basic | Yes |
| Transport | 5-10% of basic | Yes |
| Hardship (rural areas) | 20-30% of basic | Yes |
| Responsibility (HODs, Principals) | 10-25% of basic | Yes |
| Medical | Fixed amount | Yes |
| Lunch | UGX 5,000-10,000/day | Yes |
| Acting Allowance | Variable | Yes |

### Non-Taxable Allowances (Exemptions)
- Funeral expenses reimbursement
- Medical treatment costs (verified)
- Meal allowance during travel
- Uniform/workwear allowance

---

## Leave Management

### Leave Types and Accrual

| Leave Type | Entitlement | Accrual | Carry Forward |
|-----------|-------------|---------|---------------|
| Annual Leave | 21 working days | 1.75 days/month | Max 42 days |
| Sick Leave | Unlimited (with medical cert) | N/A | N/A |
| Maternity Leave | 60 working days | N/A | N/A |
| Paternity Leave | 4 working days | N/A | N/A |
| Compassionate | 3-5 days | N/A | N/A |
| Study Leave | Negotiable | N/A | N/A |

### Leave Accrual Calculation

```javascript
function calculateLeaveAccrual(hireDate, currentDate) {
  const monthsWorked = moment(currentDate).diff(moment(hireDate), 'months');
  const annualLeaveAccrued = monthsWorked * 1.75;
  
  return {
    months_worked: monthsWorked,
    annual_leave_accrued: Math.floor(annualLeaveAccrued),
    max_carry_forward: 42
  };
}
```

---

## Overtime Calculation

### Overtime Rates (Uganda Labour Standards)

- **Normal days:** 1.5× hourly rate
- **Weekends:** 2.0× hourly rate
- **Public holidays:** 2.5× hourly rate

### Overtime Calculation

```javascript
function calculateOvertime(basicSalary, overtimeHours, overtimeType) {
  const monthlyHours = 160; // 8 hours/day × 20 days
  const hourlyRate = basicSalary / monthlyHours;
  
  const rates = {
    normal: 1.5,
    weekend: 2.0,
    holiday: 2.5
  };
  
  const overtimePay = hourlyRate * overtimeHours * (rates[overtimeType] || 1.5);
  
  return Math.round(overtimePay);
}

// Example:
// Basic: 2,000,000
// Overtime: 10 hours (weekend)
// Hourly: 2,000,000 / 160 = 12,500
// Overtime Pay: 12,500 × 10 × 2.0 = 250,000
```

---

## Public Service Salary Scales

### Government Teachers (Uganda Education Service Commission)

| Scale | Position | Basic Salary (UGX) |
|-------|----------|-------------------|
| U8 | Graduate Teacher | 600,000 - 800,000 |
| U7 | Diploma Teacher | 500,000 - 650,000 |
| U6 | Certificate Teacher | 400,000 - 550,000 |
| U5 | Head of Department | 800,000 - 1,000,000 |
| U4 | Deputy Head Teacher | 1,000,000 - 1,300,000 |
| U3 | Head Teacher (Primary) | 1,200,000 - 1,500,000 |
| U2 | Head Teacher (Secondary) | 1,500,000 - 2,000,000 |
| U1E | Principal (Large School) | 2,000,000 - 2,500,000 |

**Note:** Scales subject to periodic review by Ministry of Public Service

---

## URA Reporting

### Monthly PAYE Report (CSV Format)

```csv
TIN,Employee_Name,Gross_Pay,NSSF,Chargeable_Income,PAYE,Net_Pay
1000123456,Jane Smith,2450000,122500,2327500,600250,1725250
1000789012,John Doe,3200000,150000,3050000,772000,2278000
...
```

### PAYE Return Submission (e-Tax Portal)

**Steps:**
1. Login to URA e-Tax portal
2. Navigate to: Returns > PAYE > File Return
3. Upload CSV file
4. Review summary
5. Submit return
6. Pay tax via bank or mobile money
7. Download acknowledgment receipt

**Due Date:** 15th of following month

---

## NSSF Reporting

### Monthly Contribution Schedule (Excel Format)

| NSSF No | Employee Name | Gross Pay | Employee (5%) | Employer (10%) | Total |
|---------|---------------|-----------|---------------|----------------|-------|
| SF12345678 | Jane Smith | 2,450,000 | 122,500 | 245,000 | 367,500 |
| SF87654321 | John Doe | 3,200,000 | 150,000 | 300,000 | 450,000 |

**Submission:**
- Online portal: https://nssfug.org
- Payment: Bank deposit or mobile money
- Due: 15th of following month

---

## Bank Bulk Upload Format

### Standard Bank Format (CSV)

```csv
Account_Number,Account_Name,Amount,Reference
1234567890,Jane Smith,1725250,FEB2026-SALARY
9876543210,John Doe,2278000,FEB2026-SALARY
...
```

### Bank-Specific Formats

**Stanbic Bank:**
```csv
Beneficiary_Account,Beneficiary_Name,Amount,Narration
1234567890,JANE SMITH,1725250,SALARY FEB 2026
```

**Centenary Bank:**
```csv
Account,Name,Amount,Description
1234567890,JANE SMITH,1725250,PAYROLL FEB2026
```

---

## Payroll Approval Workflow

```
┌─────────────────────┐
│  Payroll Officer    │
│  Prepares Payroll   │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│  Finance Manager    │
│  Reviews & Verifies │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│  Head Teacher/      │
│  Principal Approves │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│  Accountant         │
│  Initiates Payment  │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│  Bank Processes     │
│  Bulk Transfer      │
└─────────────────────┘
```

---

## Payroll Audit Trail

**All payroll actions logged:**

```sql
CREATE TABLE payroll_audit_logs (
    log_id BIGSERIAL PRIMARY KEY,
    payroll_run_id UUID NOT NULL,
    action VARCHAR(50) NOT NULL,
    performed_by UUID NOT NULL,
    old_values JSONB,
    new_values JSONB,
    ip_address INET,
    timestamp TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

**Audit Events:**
- `payroll_created`
- `payroll_edited`
- `payroll_approved`
- `payroll_rejected`
- `payroll_payment_initiated`
- `payroll_payment_completed`

---

## Payroll Dashboard Metrics

### Key Performance Indicators

1. **Total Payroll Cost:** UGX 45,000,000/month
2. **Average Salary:** UGX 1,500,000
3. **Total PAYE:** UGX 9,000,000/month
4. **Total NSSF (Employee + Employer):** UGX 6,750,000/month
5. **On-time Payment Rate:** 98%
6. **Error Rate:** <0.5%

---

## Integration with SchoolPay

**Staff Salary Advances via SchoolPay:**

```javascript
// Request salary advance
POST /payroll/advances/request
{
  "staff_id": "...",
  "amount": 500000,
  "repayment_months": 3,
  "reason": "Emergency medical expense"
}

// Approve and process via SchoolPay
POST /schoolpay/disburse
{
  "recipient_phone": "+256700123456",
  "amount": 500000,
  "reference": "SALARY-ADVANCE-001"
}

// Auto-deduct from next 3 payrolls
// Month 1: Deduct 166,667
// Month 2: Deduct 166,667
// Month 3: Deduct 166,666
```

---

## Compliance Checklist

- [ ] PAYE calculated according to URA tax bands
- [ ] NSSF contributions at correct rates (5% employee, 10% employer)
- [ ] LST deducted as per district rates
- [ ] Monthly PAYE return filed by 15th
- [ ] Monthly NSSF contributions paid by 15th
- [ ] Payslips generated and distributed
- [ ] Bank payments initiated on time
- [ ] Audit trail maintained
- [ ] Staff records up to date (TIN, NSSF numbers)
- [ ] Compliance with Employment Act provisions

---

## Future Enhancements

1. **Direct Integration with URA e-Tax API**
2. **Real-time NSSF validation**
3. **AI-powered payroll anomaly detection**
4. **Predictive budgeting**
5. **Mobile payslip delivery**
6. **Biometric attendance integration for overtime**

---

## Support & Training

- **Payroll Training Manual:** Available in system
- **URA Helpline:** +256 417 123 000
- **NSSF Helpline:** +256 800 101 101
- **U-USMS Support:** support@u-usms.ug
