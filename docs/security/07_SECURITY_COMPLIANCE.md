# Security & Compliance - U-USMS

## Overview

U-USMS implements enterprise-grade security aligned with international standards and Uganda's Data Protection and Privacy Act (2019).

---

## Security Framework

### Defense in Depth Strategy

```
┌─────────────────────────────────────────────────────────┐
│  Layer 1: Perimeter Security (WAF, DDoS Protection)     │
├─────────────────────────────────────────────────────────┤
│  Layer 2: Network Security (Firewall, VPN, TLS)        │
├─────────────────────────────────────────────────────────┤
│  Layer 3: Application Security (Auth, Input Validation)│
├─────────────────────────────────────────────────────────┤
│  Layer 4: Data Security (Encryption, Masking)          │
├─────────────────────────────────────────────────────────┤
│  Layer 5: Monitoring & Response (SIEM, Alerts)         │
└─────────────────────────────────────────────────────────┘
```

---

## Authentication & Authorization

### Multi-Factor Authentication (MFA)

**Required for:**
- All administrative users
- Financial transactions above UGX 1,000,000
- Payroll processing
- System configuration changes

**Supported Methods:**
- TOTP (Google Authenticator, Authy)
- SMS OTP
- Email OTP
- Biometric (mobile app)

### Role-Based Access Control (RBAC)

**Roles:**

| Role | Permissions | Scope |
|------|-------------|-------|
| Super Admin | Full system access | All tenants |
| School Admin | School management | Single school |
| Principal | Academic + Staff oversight | Single school |
| Teacher | Class & subject management | Assigned classes |
| Bursar/Accountant | Financial operations | Single school |
| Parent | View child's records only | Own children |
| Student | View own records | Self only |

**Permission Model:**
```json
{
  "role": "teacher",
  "permissions": [
    {
      "resource": "students",
      "actions": ["read"],
      "scope": "assigned_classes"
    },
    {
      "resource": "attendance",
      "actions": ["create", "read", "update"],
      "scope": "assigned_classes"
    },
    {
      "resource": "grades",
      "actions": ["create", "read", "update"],
      "scope": "assigned_subjects"
    }
  ]
}
```

---

## Data Encryption

### Encryption at Rest

**Database:**
- AES-256 encryption for all databases
- Transparent Data Encryption (TDE) enabled
- Encrypted backups

**File Storage:**
- S3/MinIO server-side encryption (SSE-S3)
- Customer Managed Keys (CMK) for sensitive documents

**Sensitive Fields:**
Additional column-level encryption for:
- National ID Numbers (NIN)
- Bank account numbers
- Phone numbers
- Email addresses
- Medical records

```sql
-- Example: Encrypted column
CREATE TABLE students (
    student_id UUID PRIMARY KEY,
    nin_encrypted BYTEA,  -- AES-256 encrypted
    nin_hash VARCHAR(64)  -- SHA-256 hash for lookups
);

-- Encryption function
CREATE FUNCTION encrypt_nin(nin TEXT, key TEXT) RETURNS BYTEA AS $$
    SELECT pgp_sym_encrypt(nin, key, 'cipher-algo=aes256');
$$ LANGUAGE SQL;
```

### Encryption in Transit

**TLS 1.3:**
- All API communication over HTTPS
- Certificate: Let's Encrypt / DigiCert
- Perfect Forward Secrecy (PFS)
- HSTS enabled

**API Security Headers:**
```
Strict-Transport-Security: max-age=31536000; includeSubDomains
Content-Security-Policy: default-src 'self'
X-Frame-Options: DENY
X-Content-Type-Options: nosniff
X-XSS-Protection: 1; mode=block
```

---

## Audit Logging

### Comprehensive Audit Trail

**All actions logged:**
- User authentication (login/logout)
- Data modifications (create/update/delete)
- Permission changes
- Financial transactions
- Report generation
- System configuration changes

**Log Structure:**
```json
{
  "timestamp": "2026-02-15T10:30:00Z",
  "log_id": "log_abc123",
  "user_id": "user_xyz789",
  "tenant_id": "tenant_550e8400",
  "action": "UPDATE",
  "resource_type": "student",
  "resource_id": "student_123",
  "changes": {
    "old_values": {"class_id": "class_A"},
    "new_values": {"class_id": "class_B"}
  },
  "ip_address": "41.202.xxx.xxx",
  "user_agent": "Mozilla/5.0...",
  "result": "success"
}
```

**Log Retention:**
- Hot storage: 90 days (Elasticsearch)
- Warm storage: 1 year (S3)
- Cold storage: 7 years (Glacier - compliance requirement)

---

## Compliance Alignment

### Uganda Data Protection and Privacy Act (2019)

**Key Requirements:**

1. **Lawful Basis for Processing**
   - Consent obtained from parents for minors
   - Clear privacy policy
   - Purpose limitation

2. **Data Subject Rights**
   - Right to access personal data
   - Right to rectification
   - Right to erasure ("right to be forgotten")
   - Right to data portability

3. **Data Protection Officer (DPO)**
   - Designated DPO contact
   - Regular compliance audits
   - Incident reporting to PDPO

4. **Cross-Border Data Transfer**
   - Data residency within Uganda (for government schools)
   - Standard contractual clauses for international transfers

**Implementation:**
```javascript
// Privacy consent management
async function obtainConsent(guardian_id, student_id) {
  const consent = await Consent.create({
    guardian_id: guardian_id,
    student_id: student_id,
    purposes: [
      'academic_management',
      'fee_collection',
      'communication',
      'ai_analytics'  // Optional
    ],
    granted_at: new Date(),
    expires_at: moment().add(2, 'years').toDate()
  });
  
  // Log consent
  await AuditLog.create({
    action: 'CONSENT_GRANTED',
    user_id: guardian_id,
    details: consent
  });
}
```

### Ministry of Education Policies

- Student data protection
- Teacher certification verification
- Curriculum compliance
- UNEB examination security

### Public Finance Management Act

**For Government Schools:**
- Financial transaction logging
- Budget accountability
- Procurement transparency
- Quarterly reporting

---

## Security Testing

### Regular Security Assessments

**Automated:**
- Daily: Dependency vulnerability scanning (Snyk, Dependabot)
- Weekly: SAST (SonarQube)
- Monthly: DAST (OWASP ZAP)

**Manual:**
- Quarterly: Penetration testing
- Annually: Third-party security audit
- As needed: Bug bounty program

**Vulnerability Management:**
```
Critical: Patch within 24 hours
High:     Patch within 7 days
Medium:   Patch within 30 days
Low:      Patch in next release
```

---

## Incident Response Plan

### Incident Classification

| Severity | Examples | Response Time |
|----------|----------|---------------|
| P1 (Critical) | Data breach, system down | <15 minutes |
| P2 (High) | Unauthorized access attempt | <1 hour |
| P3 (Medium) | Service degradation | <4 hours |
| P4 (Low) | Minor bug, feature request | <24 hours |

### Response Workflow

```
1. DETECT → Security monitoring alerts
2. ASSESS → Incident severity classification
3. CONTAIN → Isolate affected systems
4. INVESTIGATE → Root cause analysis
5. REMEDIATE → Fix vulnerability
6. RECOVER → Restore normal operations
7. REPORT → Notify stakeholders
8. REVIEW → Post-incident analysis
```

**Incident Communication:**
- Internal: Slack/Teams notification
- External: Email to affected users
- Regulatory: Report to PDPO within 72 hours (if data breach)

---

## Data Backup & Recovery

### Backup Strategy

**3-2-1 Rule:**
- 3 copies of data
- 2 different storage media
- 1 off-site backup

**Backup Schedule:**
```
Database:
  - Continuous WAL archiving (Point-in-time recovery)
  - Full backup: Daily at 02:00 EAT
  - Incremental: Every 6 hours
  
Files:
  - Daily snapshot to S3
  - Weekly full backup
  
Retention:
  - Daily: 30 days
  - Weekly: 12 weeks
  - Monthly: 12 months
  - Yearly: 7 years
```

### Disaster Recovery

**RTO/RPO Targets:**

| System Component | RTO | RPO |
|-----------------|-----|-----|
| Core Application | 1 hour | 5 minutes |
| Database | 15 minutes | 1 minute |
| File Storage | 2 hours | 15 minutes |
| Payment System | 30 minutes | 0 (synchronous) |

**DR Testing:**
- Quarterly DR drills
- Annual full failover test
- Documented runbooks

---

## Network Security

### Firewall Rules

```
# Inbound
ALLOW: 443 (HTTPS) from 0.0.0.0/0
ALLOW: 22 (SSH) from VPN IP only
DENY: All other inbound

# Outbound
ALLOW: 443 (HTTPS) to trusted APIs
ALLOW: 25, 587 (SMTP) to email servers
DENY: All other outbound
```

### DDoS Protection

- CloudFlare WAF
- Rate limiting (1000 req/hour/IP)
- IP reputation filtering
- Challenge page for suspicious traffic

### VPN Access

**For remote administration:**
- OpenVPN or WireGuard
- 2FA required
- Activity logging
- Access review quarterly

---

## Compliance Checklist

### Pre-Launch Checklist

- [ ] Security audit completed
- [ ] Penetration test passed
- [ ] Privacy policy published
- [ ] DPO appointed
- [ ] Consent management implemented
- [ ] Encryption enabled (rest + transit)
- [ ] RBAC configured
- [ ] Audit logging active
- [ ] Backup/DR tested
- [ ] Incident response plan documented
- [ ] Staff security training completed
- [ ] Compliance documentation prepared

### Ongoing Compliance

- [ ] Monthly vulnerability scanning
- [ ] Quarterly access reviews
- [ ] Annual security audit
- [ ] Annual DR test
- [ ] Continuous monitoring
- [ ] Regular staff training
- [ ] Policy updates as needed

---

## Security Training

**For All Users:**
- Password best practices
- Phishing awareness
- Social engineering recognition
- Incident reporting procedures

**For Administrators:**
- Secure configuration
- Vulnerability management
- Incident response
- Compliance requirements

---

## Contact & Reporting

**Security Issues:**
- Email: security@u-usms.ug
- PGP Key: Available on website
- Bug Bounty: HackerOne program

**Data Protection Officer:**
- Name: [To be appointed]
- Email: dpo@u-usms.ug
- Phone: +256 XXX XXX XXX

**Regulatory Authority:**
- Uganda Personal Data Protection Office (PDPO)
- Website: https://pdpo.go.ug
- Email: info@pdpo.go.ug
