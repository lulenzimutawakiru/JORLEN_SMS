# System Architecture - U-USMS

## Architecture Overview

U-USMS follows a modern **cloud-native microservices architecture** designed for scalability, resilience, and multi-tenancy.

---

## High-Level Architecture Diagram

```
┌─────────────────────────────────────────────────────────────────────┐
│                         CLIENT LAYER                                 │
├─────────────────────────────────────────────────────────────────────┤
│  Web App    │  Mobile App  │  USSD Gateway │  SMS Gateway │  APIs   │
│  (React)    │  (Flutter)   │               │              │          │
└──────────────┬──────────────┴───────────────┴──────────────┴─────────┘
               │
               ▼
┌─────────────────────────────────────────────────────────────────────┐
│                      CDN & LOAD BALANCER                             │
│              (CloudFlare CDN + AWS ALB/NLB)                         │
└──────────────┬──────────────────────────────────────────────────────┘
               │
               ▼
┌─────────────────────────────────────────────────────────────────────┐
│                      API GATEWAY LAYER                               │
│         (Kong/AWS API Gateway - Rate Limiting, Auth)                │
└──────────────┬──────────────────────────────────────────────────────┘
               │
               ▼
┌─────────────────────────────────────────────────────────────────────┐
│                    MICROSERVICES LAYER (K8s)                         │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐              │
│  │   Academic   │  │  Financial   │  │   Payroll    │              │
│  │   Service    │  │   Service    │  │   Service    │              │
│  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘              │
│         │                  │                  │                      │
│  ┌──────┴───────┐  ┌──────┴───────┐  ┌──────┴───────┐              │
│  │   Student    │  │   Payment    │  │     HR       │              │
│  │   Service    │  │   Service    │  │   Service    │              │
│  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘              │
│         │                  │                  │                      │
│  ┌──────┴───────┐  ┌──────┴───────┐  ┌──────┴───────┐              │
│  │ Attendance   │  │SchoolPay Int │  │  AI Engine   │              │
│  │   Service    │  │   Service    │  │   Service    │              │
│  └──────────────┘  └──────────────┘  └──────────────┘              │
│                                                                      │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐              │
│  │     SMS      │  │    NIRA      │  │   Reporting  │              │
│  │   Service    │  │   Service    │  │   Service    │              │
│  └──────────────┘  └──────────────┘  └──────────────┘              │
│                                                                      │
└──────────────┬───────────────────────────────────────────────────────┘
               │
               ▼
┌─────────────────────────────────────────────────────────────────────┐
│                        DATA LAYER                                    │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ┌──────────────────┐  ┌──────────────┐  ┌──────────────┐          │
│  │  Primary DB      │  │  Read Replicas│ │   Redis      │          │
│  │  PostgreSQL      │  │  PostgreSQL   │ │   Cache      │          │
│  │  (Multi-tenant)  │  └───────────────┘ └──────────────┘          │
│  └──────────────────┘                                               │
│                                                                      │
│  ┌──────────────────┐  ┌──────────────┐  ┌──────────────┐          │
│  │  Object Storage  │  │  Search DB   │  │  Time Series │          │
│  │  (S3/MinIO)      │  │ (ElasticSrch)│  │  (InfluxDB)  │          │
│  └──────────────────┘  └──────────────┘  └──────────────┘          │
│                                                                      │
└──────────────────────────────────────────────────────────────────────┘
               │
               ▼
┌─────────────────────────────────────────────────────────────────────┐
│                    EXTERNAL INTEGRATIONS                             │
├─────────────────────────────────────────────────────────────────────┤
│  SchoolPay API  │  URA Portal  │  NSSF API  │  NIRA API │  Banks   │
└─────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────┐
│                    OBSERVABILITY & MONITORING                        │
├─────────────────────────────────────────────────────────────────────┤
│  Prometheus  │  Grafana  │  ELK Stack  │  Jaeger  │  Sentry        │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Multi-Tenancy Strategy

### Option A: Shared Database with Tenant Isolation (Recommended for SaaS)

**Architecture:**
```
┌─────────────────────────────────────────────┐
│         PostgreSQL Database                  │
├─────────────────────────────────────────────┤
│  All Tables have 'tenant_id' column         │
│  Row-Level Security (RLS) enforced          │
│  Indexed on tenant_id for performance       │
│  Shared schema, partitioned data            │
└─────────────────────────────────────────────┘
```

**Advantages:**
- Cost-effective (single DB maintenance)
- Easy to scale horizontally
- Simpler backup and recovery
- Better resource utilization
- Centralized updates and patches

**Risks & Mitigation:**
- **Risk:** Data leakage between tenants
  - **Mitigation:** PostgreSQL Row-Level Security (RLS), application-level checks, audit logging
- **Risk:** Noisy neighbor problem
  - **Mitigation:** Connection pooling, query timeout limits, resource quotas
- **Risk:** Single point of failure
  - **Mitigation:** High availability setup, automatic failover, read replicas

**Security Measures:**
```sql
-- Example: Row-Level Security Policy
CREATE POLICY tenant_isolation ON students
  USING (tenant_id = current_setting('app.current_tenant')::uuid);

-- Application sets tenant context per request
SET app.current_tenant = '550e8400-e29b-41d4-a716-446655440000';
```

---

### Option B: Per-Tenant Database (Government/Enterprise)

**Architecture:**
```
┌───────────────┐  ┌───────────────┐  ┌───────────────┐
│  Tenant A DB  │  │  Tenant B DB  │  │  Tenant C DB  │
│  (Gov School) │  │  (University) │  │  (Network)    │
└───────────────┘  └───────────────┘  └───────────────┘
        │                   │                   │
        └───────────────────┴───────────────────┘
                            │
                   ┌────────┴────────┐
                   │  Master Catalog │
                   │  (Tenant Registry)
                   └─────────────────┘
```

**Advantages:**
- Complete data isolation
- Custom performance tuning per tenant
- Independent scaling
- Easier compliance for government entities
- Custom backup schedules

**Disadvantages:**
- Higher infrastructure costs
- Complex multi-tenant queries
- More maintenance overhead
- Schema migration complexity

**Use Cases:**
- Government schools (data sovereignty)
- Large universities (>50,000 students)
- School networks with compliance requirements

---

## Technology Stack

### Backend
- **Language:** Node.js (TypeScript) / Python (FastAPI)
- **Framework:** NestJS / Express.js / Django
- **API:** REST + GraphQL
- **Authentication:** JWT + OAuth2
- **Real-time:** WebSocket (Socket.io)

### Frontend
- **Web:** React.js + TypeScript + Next.js
- **Mobile:** Flutter (iOS + Android)
- **State Management:** Redux / Zustand
- **UI Framework:** Material-UI / Tailwind CSS

### Database
- **Primary:** PostgreSQL 15+ (ACID compliance, JSON support)
- **Cache:** Redis (session, query cache)
- **Search:** Elasticsearch (full-text search)
- **Time-Series:** InfluxDB (metrics, attendance trends)
- **Queue:** RabbitMQ / AWS SQS

### Infrastructure
- **Container:** Docker
- **Orchestration:** Kubernetes (EKS/GKE/AKS)
- **CI/CD:** GitHub Actions / GitLab CI
- **IaC:** Terraform / Pulumi
- **Service Mesh:** Istio (optional for advanced scenarios)

### Cloud Providers (Multi-Cloud Strategy)
- **Primary:** AWS (proven reliability)
- **Backup:** Google Cloud Platform (disaster recovery)
- **Local:** Uganda-based data center (government compliance)

### Monitoring & Observability
- **Metrics:** Prometheus + Grafana
- **Logging:** ELK Stack (Elasticsearch, Logstash, Kibana)
- **Tracing:** Jaeger / OpenTelemetry
- **APM:** New Relic / Datadog
- **Error Tracking:** Sentry

### Security
- **WAF:** AWS WAF / CloudFlare
- **Secrets:** HashiCorp Vault / AWS Secrets Manager
- **Scanning:** SonarQube, Snyk, Trivy
- **Penetration Testing:** Annual third-party audits

---

## Microservices Design

### Core Services

#### 1. Academic Service
**Responsibilities:**
- Student enrollment
- Curriculum management
- Assessment and grading
- Timetable generation
- Result processing

**APIs:**
```
POST   /api/v1/students
GET    /api/v1/students/{id}
PUT    /api/v1/students/{id}
POST   /api/v1/assessments
GET    /api/v1/results/{studentId}
POST   /api/v1/grades/calculate
```

#### 2. Financial Service
**Responsibilities:**
- Fee structure management
- Invoicing
- Payment processing
- Ledger management
- Financial reporting

**APIs:**
```
POST   /api/v1/fees/structures
POST   /api/v1/invoices
GET    /api/v1/payments/status/{invoiceId}
POST   /api/v1/reconciliation
GET    /api/v1/reports/financial
```

#### 3. Payroll Service
**Responsibilities:**
- Salary calculation
- PAYE computation
- NSSF deductions
- Payslip generation
- Bank file export

**APIs:**
```
POST   /api/v1/payroll/run
GET    /api/v1/payroll/payslips/{staffId}
POST   /api/v1/payroll/tax/compute
GET    /api/v1/payroll/reports/ura
GET    /api/v1/payroll/reports/nssf
```

#### 4. SchoolPay Integration Service
**Responsibilities:**
- Payment gateway integration
- Webhook handling
- Payment verification
- Receipt generation

**APIs:**
```
POST   /api/v1/schoolpay/initiate
POST   /webhooks/schoolpay/callback
GET    /api/v1/schoolpay/verify/{txnId}
POST   /api/v1/schoolpay/refund
```

#### 5. AI Engine Service
**Responsibilities:**
- Performance prediction
- Risk detection
- Recommendations
- Chatbot processing

**APIs:**
```
POST   /api/v1/ai/predict/performance
POST   /api/v1/ai/detect/dropout-risk
POST   /api/v1/ai/chatbot/query
GET    /api/v1/ai/insights/{studentId}
```

#### 6. Communication Service
**Responsibilities:**
- SMS sending
- Email delivery
- USSD handling
- Notification management

**APIs:**
```
POST   /api/v1/sms/send
POST   /api/v1/email/send
POST   /api/v1/ussd/session
GET    /api/v1/notifications/{userId}
```

---

## Scalability Design

### Horizontal Scaling Strategy

```
┌─────────────────────────────────────────────────────────────┐
│              Application Load Balancer                       │
│                  (AWS ALB + Auto-scaling)                    │
└────────────────────┬────────────────────────────────────────┘
                     │
        ┌────────────┼────────────┐
        │            │            │
┌───────▼──────┐ ┌──▼────────┐ ┌─▼───────────┐
│  Service Pod │ │Service Pod│ │ Service Pod │
│  (Replica 1) │ │(Replica 2)│ │ (Replica 3) │
└──────────────┘ └───────────┘ └─────────────┘
     (Min: 3 replicas, Max: 50 replicas per service)
```

**Auto-Scaling Policies:**
- **CPU-based:** Scale up when CPU > 70%
- **Memory-based:** Scale up when Memory > 80%
- **Request-based:** Scale up when RPS > 1000/pod
- **Custom Metrics:** Queue depth, response time

### Database Scaling

```
┌──────────────────────────────────────────────────────┐
│                Primary PostgreSQL                     │
│              (Writes + Critical Reads)                │
└─────────────────────┬────────────────────────────────┘
                      │ (Async Replication)
          ┌───────────┼───────────┐
          │           │           │
┌─────────▼──┐ ┌──────▼────┐ ┌───▼──────┐
│ Read Replica│ │Read Replica│ │  Read    │
│  (Region 1) │ │ (Region 2) │ │ Replica  │
└─────────────┘ └───────────┘ └──────────┘
```

**Database Partitioning:**
- **Horizontal:** Partition by tenant_id
- **Vertical:** Separate frequently accessed tables
- **Sharding:** For >10,000 tenants, shard by tenant region

### Caching Strategy

**Multi-Level Caching:**
```
┌─────────────────────────────────────────────────────┐
│  Level 1: Browser Cache (Static assets - 7 days)    │
└─────────────────────────────────────────────────────┘
┌─────────────────────────────────────────────────────┐
│  Level 2: CDN Cache (CloudFlare - Global Edge)      │
└─────────────────────────────────────────────────────┘
┌─────────────────────────────────────────────────────┐
│  Level 3: Redis Cache (Query results - 5 min)       │
└─────────────────────────────────────────────────────┘
┌─────────────────────────────────────────────────────┐
│  Level 4: Application Cache (Hot data - 1 min)      │
└─────────────────────────────────────────────────────┘
┌─────────────────────────────────────────────────────┐
│  Level 5: Database Query Cache (PostgreSQL)         │
└─────────────────────────────────────────────────────┘
```

**Cache Invalidation:**
- TTL-based for read-heavy data
- Event-driven for write operations
- Manual purge for critical updates

---

## Disaster Recovery & High Availability

### RTO/RPO Targets

| Service Tier | RTO (Recovery Time Objective) | RPO (Recovery Point Objective) |
|-------------|-------------------------------|--------------------------------|
| Critical (Payment, Payroll) | 15 minutes | 5 minutes |
| High (Academic, Student) | 1 hour | 15 minutes |
| Standard (Reports, Analytics) | 4 hours | 1 hour |

### Backup Strategy

**Database Backups:**
- **Continuous WAL archiving** (Point-in-time recovery)
- **Full backup:** Daily at 02:00 EAT
- **Incremental backup:** Every 6 hours
- **Retention:** 30 days online, 1 year archival (Glacier)

**Application State:**
- Redis snapshots every hour
- Configuration backups in S3 (versioned)

### Multi-Region Deployment

```
┌─────────────────────────────────────────────────────┐
│           PRIMARY REGION (East Africa)               │
│              (Active - 100% traffic)                 │
└────────────────────┬────────────────────────────────┘
                     │ (Async Replication)
┌────────────────────▼────────────────────────────────┐
│        SECONDARY REGION (South Africa)               │
│              (Standby - 0% traffic)                  │
│         (Automatic failover in 5 minutes)            │
└─────────────────────────────────────────────────────┘
```

**Failover Triggers:**
- Primary region unavailable for >2 minutes
- Database connection failure
- Manual initiation by ops team

---

## Performance Targets

| Metric | Target | Measurement |
|--------|--------|-------------|
| Page Load Time | <2 seconds | 95th percentile |
| API Response Time | <200ms | Median |
| Database Query Time | <50ms | 90th percentile |
| Concurrent Users | 10,000+ | Load testing verified |
| Transactions/Second | 5,000+ | Peak capacity |
| Uptime | 99.9% | Monthly SLA |
| Data Sync Latency | <5 seconds | Average |

---

## Security Architecture

### Zero-Trust Model

**Principles:**
1. **Never trust, always verify**
2. **Least privilege access**
3. **Assume breach mentality**
4. **Verify explicitly**

**Implementation:**
```
┌─────────────────────────────────────────────────────┐
│  User Authentication (MFA Required)                  │
└────────────────────┬────────────────────────────────┘
                     │
┌────────────────────▼────────────────────────────────┐
│  Service-to-Service Auth (mTLS + JWT)               │
└────────────────────┬────────────────────────────────┘
                     │
┌────────────────────▼────────────────────────────────┐
│  Fine-Grained Authorization (RBAC + ABAC)           │
└────────────────────┬────────────────────────────────┘
                     │
┌────────────────────▼────────────────────────────────┐
│  Data Encryption (At Rest + In Transit)             │
└────────────────────┬────────────────────────────────┘
                     │
┌────────────────────▼────────────────────────────────┐
│  Audit Logging (All actions logged)                 │
└─────────────────────────────────────────────────────┘
```

### Network Security

```
┌─────────────────────────────────────────────────────┐
│  Internet │ WAF (CloudFlare) │ DDoS Protection      │
└─────────────────────┬───────────────────────────────┘
                      │
┌─────────────────────▼───────────────────────────────┐
│  Public Subnet │ Load Balancer │ API Gateway        │
└─────────────────────┬───────────────────────────────┘
                      │
┌─────────────────────▼───────────────────────────────┐
│  Private Subnet │ Application Services (K8s)        │
└─────────────────────┬───────────────────────────────┘
                      │
┌─────────────────────▼───────────────────────────────┐
│  Database Subnet │ PostgreSQL │ Redis (Isolated)    │
└─────────────────────────────────────────────────────┘
```

**Security Groups:**
- Inbound traffic only on required ports
- Outbound traffic whitelisted
- Inter-service communication via private network

---

## Offline Capability for Rural Schools

### Architecture

```
┌─────────────────────────────────────────────────────┐
│           School Premises (No Internet)              │
├─────────────────────────────────────────────────────┤
│                                                      │
│  ┌──────────────────────────────────────┐           │
│  │   Local Edge Server (Raspberry Pi)   │           │
│  │   - Lightweight DB (SQLite/PouchDB)  │           │
│  │   - Core features cached             │           │
│  │   - Offline-first PWA                │           │
│  │   - Sync queue                       │           │
│  └──────────────────┬───────────────────┘           │
│                     │                                │
│  ┌──────────────────▼───────────────────┐           │
│  │  Local Network (WiFi)                │           │
│  │  - Teachers' devices                 │           │
│  │  - Admin devices                     │           │
│  └──────────────────────────────────────┘           │
│                                                      │
└─────────────────────┬───────────────────────────────┘
                      │ (When internet available)
                      │ (Auto-sync every hour)
┌─────────────────────▼───────────────────────────────┐
│              Cloud Infrastructure                    │
│           (Conflict resolution engine)               │
└─────────────────────────────────────────────────────┘
```

**Offline Features:**
- Student attendance marking
- Grade entry (cached locally)
- Fee payments (queued for sync)
- Basic reporting (local data)

**Sync Strategy:**
- **When online:** Auto-sync every 10 minutes
- **Conflict resolution:** Last-write-wins with audit trail
- **Data compression:** Minimize bandwidth usage
- **Incremental sync:** Only changed records

---

## Conclusion

This architecture is designed to be:
- **Scalable:** From 10 schools to 10,000+ schools
- **Resilient:** Multi-region, high availability, disaster recovery
- **Secure:** Zero-trust, encryption, compliance-ready
- **Flexible:** Multi-tenant options for different deployment models
- **Future-proof:** Microservices allow independent scaling and updates

Next: See detailed database design in `02_DATABASE_DESIGN.md`
