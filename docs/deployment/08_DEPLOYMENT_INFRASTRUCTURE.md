# Deployment & Infrastructure - U-USMS

## Cloud Infrastructure

### Multi-Cloud Strategy

**Primary:** AWS (East Africa - Cape Town region)  
**Secondary:** Google Cloud Platform (Disaster recovery)  
**Local:** Uganda Data Center (Government compliance)

---

## Kubernetes Deployment

### Cluster Architecture

```yaml
# k8s-cluster-config.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: cluster-config
data:
  cluster_name: u-usms-production
  region: af-south-1
  node_groups:
    - name: general-purpose
      instance_type: t3.xlarge
      min_size: 3
      max_size: 20
      desired_capacity: 5
    
    - name: ai-workloads
      instance_type: g4dn.xlarge  # GPU instances
      min_size: 1
      max_size: 5
      desired_capacity: 2
    
    - name: database
      instance_type: r5.2xlarge  # Memory optimized
      min_size: 2
      max_size: 4
      desired_capacity: 2
```

### Application Deployment

```yaml
# deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: usms-api
  namespace: production
spec:
  replicas: 5
  selector:
    matchLabels:
      app: usms-api
  template:
    metadata:
      labels:
        app: usms-api
        version: v1.0.0
    spec:
      containers:
      - name: api
        image: usms/api:1.0.0
        ports:
        - containerPort: 3000
        env:
        - name: NODE_ENV
          value: "production"
        - name: DB_HOST
          valueFrom:
            secretKeyRef:
              name: db-credentials
              key: host
        - name: DB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: db-credentials
              key: password
        resources:
          requests:
            memory: "512Mi"
            cpu: "500m"
          limits:
            memory: "1Gi"
            cpu: "1000m"
        livenessProbe:
          httpGet:
            path: /health
            port: 3000
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /ready
            port: 3000
          initialDelaySeconds: 10
          periodSeconds: 5
---
apiVersion: v1
kind: Service
metadata:
  name: usms-api-service
spec:
  type: LoadBalancer
  selector:
    app: usms-api
  ports:
  - protocol: TCP
    port: 80
    targetPort: 3000
```

### Auto-Scaling

```yaml
# hpa.yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: usms-api-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: usms-api
  minReplicas: 3
  maxReplicas: 50
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 80
  behavior:
    scaleUp:
      stabilizationWindowSeconds: 60
      policies:
      - type: Percent
        value: 50
        periodSeconds: 60
    scaleDown:
      stabilizationWindowSeconds: 300
      policies:
      - type: Percent
        value: 10
        periodSeconds: 60
```

---

## Database Infrastructure

### PostgreSQL High Availability

```yaml
# postgresql-ha.yaml
apiVersion: postgresql.cnpg.io/v1
kind: Cluster
metadata:
  name: usms-postgres
spec:
  instances: 3
  primaryUpdateStrategy: unsupervised
  
  postgresql:
    parameters:
      max_connections: "500"
      shared_buffers: "4GB"
      effective_cache_size: "12GB"
      maintenance_work_mem: "1GB"
      checkpoint_completion_target: "0.9"
      wal_buffers: "16MB"
      default_statistics_target: "100"
      random_page_cost: "1.1"
      effective_io_concurrency: "200"
      work_mem: "10MB"
      min_wal_size: "2GB"
      max_wal_size: "8GB"
  
  bootstrap:
    initdb:
      database: usms
      owner: usms_admin
      secret:
        name: postgres-credentials
  
  storage:
    size: 500Gi
    storageClass: gp3-encrypted
  
  backup:
    barmanObjectStore:
      destinationPath: s3://usms-backups/postgres
      s3Credentials:
        accessKeyId:
          name: s3-credentials
          key: access_key
        secretAccessKey:
          name: s3-credentials
          key: secret_key
      wal:
        compression: gzip
        maxParallel: 8
    retentionPolicy: "30d"
```

### Redis Cache

```yaml
# redis.yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: redis
spec:
  serviceName: redis
  replicas: 3
  selector:
    matchLabels:
      app: redis
  template:
    metadata:
      labels:
        app: redis
    spec:
      containers:
      - name: redis
        image: redis:7-alpine
        command:
        - redis-server
        - --maxmemory
        - "2gb"
        - --maxmemory-policy
        - allkeys-lru
        - --appendonly
        - "yes"
        ports:
        - containerPort: 6379
        volumeMounts:
        - name: data
          mountPath: /data
  volumeClaimTemplates:
  - metadata:
      name: data
    spec:
      accessModes: ["ReadWriteOnce"]
      storageClassName: gp3
      resources:
        requests:
          storage: 50Gi
```

---

## Load Balancing

### Application Load Balancer

```yaml
# ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: usms-ingress
  annotations:
    kubernetes.io/ingress.class: alb
    alb.ingress.kubernetes.io/scheme: internet-facing
    alb.ingress.kubernetes.io/target-type: ip
    alb.ingress.kubernetes.io/certificate-arn: arn:aws:acm:...
    alb.ingress.kubernetes.io/ssl-policy: ELBSecurityPolicy-TLS-1-2-2017-01
    alb.ingress.kubernetes.io/listen-ports: '[{"HTTP": 80}, {"HTTPS": 443}]'
    alb.ingress.kubernetes.io/actions.ssl-redirect: >
      {"Type": "redirect", "RedirectConfig": {"Protocol": "HTTPS", "Port": "443", "StatusCode": "HTTP_301"}}
spec:
  rules:
  - host: api.u-usms.ug
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: usms-api-service
            port:
              number: 80
  - host: app.u-usms.ug
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: usms-frontend-service
            port:
              number: 80
```

---

## CI/CD Pipeline

### GitHub Actions Workflow

```yaml
# .github/workflows/deploy.yml
name: Deploy to Production

on:
  push:
    branches:
      - main
  workflow_dispatch:

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '18'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Run tests
        run: npm test
      
      - name: Run linter
        run: npm run lint
      
      - name: Security scan
        run: npm audit
  
  build:
    needs: test
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v2
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: af-south-1
      
      - name: Login to Amazon ECR
        id: login-ecr
        uses: aws-actions/amazon-ecr-login@v1
      
      - name: Build and push Docker image
        env:
          ECR_REGISTRY: ${{ steps.login-ecr.outputs.registry }}
          IMAGE_TAG: ${{ github.sha }}
        run: |
          docker build -t $ECR_REGISTRY/usms-api:$IMAGE_TAG .
          docker push $ECR_REGISTRY/usms-api:$IMAGE_TAG
          docker tag $ECR_REGISTRY/usms-api:$IMAGE_TAG $ECR_REGISTRY/usms-api:latest
          docker push $ECR_REGISTRY/usms-api:latest
  
  deploy:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Configure kubectl
        uses: azure/k8s-set-context@v3
        with:
          method: kubeconfig
          kubeconfig: ${{ secrets.KUBE_CONFIG }}
      
      - name: Deploy to Kubernetes
        run: |
          kubectl set image deployment/usms-api \
            api=${{ steps.login-ecr.outputs.registry }}/usms-api:${{ github.sha }} \
            -n production
          
          kubectl rollout status deployment/usms-api -n production
      
      - name: Notify Slack
        uses: 8398a7/action-slack@v3
        with:
          status: ${{ job.status }}
          text: 'Deployment to production completed'
          webhook_url: ${{ secrets.SLACK_WEBHOOK }}
```

---

## Monitoring & Observability

### Prometheus + Grafana

```yaml
# prometheus-values.yaml
prometheus:
  prometheusSpec:
    retention: 30d
    storageSpec:
      volumeClaimTemplate:
        spec:
          accessModes: ["ReadWriteOnce"]
          resources:
            requests:
              storage: 100Gi

grafana:
  adminPassword: $GRAFANA_ADMIN_PASSWORD
  datasources:
    - name: Prometheus
      type: prometheus
      url: http://prometheus-server
      isDefault: true
  
  dashboards:
    - name: U-USMS Overview
      folder: General
      type: file
      options:
        path: /var/lib/grafana/dashboards/overview.json
```

### Logging (ELK Stack)

```yaml
# elasticsearch.yaml
apiVersion: elasticsearch.k8s.elastic.co/v1
kind: Elasticsearch
metadata:
  name: usms-logs
spec:
  version: 8.10.0
  nodeSets:
  - name: default
    count: 3
    config:
      node.store.allow_mmap: false
    volumeClaimTemplates:
    - metadata:
        name: elasticsearch-data
      spec:
        accessModes:
        - ReadWriteOnce
        resources:
          requests:
            storage: 200Gi
        storageClassName: gp3
```

---

## Cost Optimization

### Resource Right-Sizing

**Monthly Infrastructure Cost Estimate:**

| Component | Configuration | Monthly Cost (USD) |
|-----------|--------------|-------------------|
| EKS Cluster | 3 node groups | $500 |
| EC2 Instances | 10 × t3.xlarge | $1,200 |
| RDS PostgreSQL | db.r5.2xlarge (Multi-AZ) | $900 |
| ElastiCache Redis | cache.r5.large × 3 | $400 |
| S3 Storage | 1TB | $23 |
| CloudFront CDN | 2TB transfer | $170 |
| Load Balancer | ALB | $25 |
| Monitoring | Prometheus, Grafana | $100 |
| **Total** | | **$3,318/month** |

**At Scale (10,000 schools):**
- Infrastructure: $10,000/month
- Bandwidth: $2,000/month
- Storage: $1,000/month
- **Total:** $13,000/month

**Revenue vs. Cost:**
- Revenue (10,000 schools, 2M students): ~$583,333/month
- Infrastructure: $13,000/month
- **Profit Margin:** 97.8%

---

## Disaster Recovery

### Backup Locations

```
Primary:    AWS S3 (af-south-1)
Secondary:  GCP Cloud Storage (europe-west1)
Tertiary:   Local Uganda Data Center
```

### Recovery Procedures

```bash
# 1. Database Recovery
pg_restore -h new-db-host -U postgres -d usms backup.dump

# 2. Application Deployment
kubectl apply -f k8s/
kubectl rollout status deployment/usms-api

# 3. DNS Failover
aws route53 change-resource-record-sets \
  --hosted-zone-id Z123456 \
  --change-batch file://failover.json

# 4. Verification
curl https://api.u-usms.ug/health
```

---

## Scaling Roadmap

### Phase 1: 1,000 Schools
- Single region deployment
- 5 app servers
- 1 database (with replicas)
- Monthly cost: ~$3,500

### Phase 2: 5,000 Schools
- Multi-region deployment
- 20 app servers
- Database sharding (5 shards)
- Monthly cost: ~$8,000

### Phase 3: 10,000+ Schools
- Global CDN
- 50+ app servers
- Horizontally scaled databases
- Advanced caching
- Monthly cost: ~$15,000

---

## Deployment Checklist

### Pre-Production
- [ ] Infrastructure provisioned
- [ ] Kubernetes cluster configured
- [ ] Database migrated
- [ ] SSL certificates installed
- [ ] DNS configured
- [ ] Monitoring enabled
- [ ] Backups configured
- [ ] Security audit passed
- [ ] Load testing completed
- [ ] Disaster recovery tested

### Go-Live
- [ ] Final data migration
- [ ] DNS cutover
- [ ] Smoke tests passed
- [ ] Support team notified
- [ ] Rollback plan ready

### Post-Production
- [ ] Monitor metrics (24 hours)
- [ ] Verify backups
- [ ] Review logs
- [ ] User feedback collection
- [ ] Performance optimization

---

**Next:** See `docs/business/09_BUSINESS_MODEL.md`
