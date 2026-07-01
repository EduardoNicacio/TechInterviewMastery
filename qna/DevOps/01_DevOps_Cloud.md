# DevOps & Cloud – Interview Questions & Answers

**Question**: How do Docker image layers affect build caching and what best practices follow?

**Answer**: Each instruction in a Dockerfile creates a read-only layer that Docker caches. If a layer hasn't changed, Docker reuses it from cache, dramatically speeding up rebuilds. Best practices include ordering infrequent changes (e.g., `apt-get install`) before frequent ones (e.g., source code), combining `RUN` commands to reduce layers, and leveraging `--cache-from` in CI.

```dockerfile
FROM node:18 AS base
WORKDIR /app
# Infrequent changes first – cached unless package.json changes
COPY package.json package-lock.json ./
RUN npm ci --omit=dev
# Frequent changes last – invalidates cache often
COPY . .
RUN npm run build
```

---

**Question**: What is the purpose of multi-stage builds in Docker?

**Answer**: Multi-stage builds use multiple `FROM` statements to separate build-time dependencies from the runtime image. The final stage copies only the compiled artifacts, resulting in significantly smaller images without compilers, SDKs, or dev tooling.

```dockerfile
# Build stage
FROM golang:1.21 AS builder
WORKDIR /src
COPY . .
RUN CGO_ENABLED=0 go build -o /app .

# Runtime stage
FROM alpine:3.19
COPY --from=builder /app /app
CMD ["/app"]
```

---

**Question**: What are Dockerfile best practices for production images?

**Answer**: Use a specific base image tag (not `latest`), prefer distroless or Alpine for smaller footprint, run as a non-root user (`USER` directive), use `.dockerignore` to exclude unnecessary files, and leverage `HEALTHCHECK` for container monitoring. Avoid installing unnecessary packages and always pin versions.

```dockerfile
FROM node:18-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

FROM node:18-alpine AS production
WORKDIR /app
RUN addgroup -S appgroup && adduser -S appuser -G appgroup
COPY --from=builder /app/dist ./dist
USER appuser
EXPOSE 3000
HEALTHCHECK --interval=30s --timeout=3s CMD wget -qO- http://localhost:3000/health
CMD ["node", "dist/server.js"]
```

---

**Question**: How do Docker Compose and Kubernetes differ in purpose?

**Answer**: Docker Compose is designed for single-host, development, and testing scenarios, defining multi-container apps with a simple YAML file. Kubernetes targets production-grade, multi-host orchestration with features like auto-scaling, self-healing, rolling updates, and service discovery. Compose is simpler but lacks Kubernetes' robustness for large-scale deployments.

---

**Question**: What is a Pod in Kubernetes and how does it relate to containers?

**Answer**: A Pod is the smallest deployable unit in Kubernetes, encapsulating one or more tightly coupled containers that share the same network namespace, storage volumes, and lifecycle. Containers within a Pod communicate via `localhost` and are scheduled together on the same node, ideal for sidecar patterns.

---

**Question**: How does a Deployment differ from a StatefulSet?

**Answer**: Deployments manage stateless applications where Pods are interchangeable and can be scaled or rolled out arbitrarily. StatefulSets provide stable network identities (`pod-name-0`, `pod-name-1`), ordered startup/shutdown, and persistent storage per Pod, making them suitable for stateful workloads like databases or message queues.

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: postgres
spec:
  serviceName: postgres
  replicas: 3
  selector:
    matchLabels:
      app: postgres
  template:
    metadata:
      labels:
        app: postgres
    spec:
      containers:
      - name: postgres
        image: postgres:16
        volumeMounts:
        - name: data
          mountPath: /var/lib/postgresql/data
  volumeClaimTemplates:
  - metadata:
      name: data
    spec:
      accessModes: ["ReadWriteOnce"]
      resources:
        requests:
          storage: 10Gi
```

---

**Question**: What are the three main Kubernetes Service types and their use cases?

**Answer**: `ClusterIP` exposes the service on an internal IP reachable only within the cluster, suitable for internal microservice communication. `NodePort` opens a static port on every node (30000-32767) for external traffic, useful for development. `LoadBalancer` provisions a cloud load balancer with a public IP, the standard for production internet-facing services.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: web
spec:
  type: LoadBalancer
  selector:
    app: web
  ports:
  - port: 80
    targetPort: 8080
```

---

**Question**: What role does an Ingress controller play in Kubernetes?

**Answer**: An Ingress controller is a reverse proxy (e.g., NGINX, Traefik, AWS ALB Ingress Controller) that routes external HTTP/HTTPS traffic to Services based on rules, hostnames, or paths. It terminates TLS, supports virtual hosting, and enables advanced routing without provisioning individual LoadBalancer Services per app.

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: app-ingress
  annotations:
    cert-manager.io/cluster-issuer: letsencrypt-prod
spec:
  ingressClassName: nginx
  tls:
  - hosts:
    - app.example.com
    secretName: app-tls
  rules:
  - host: app.example.com
    http:
      paths:
      - path: /api
        pathType: Prefix
        backend:
          service:
            name: api-service
            port:
              number: 80
```

---

**Question**: How do ConfigMaps and Secrets differ in Kubernetes?

**Answer**: Both store configuration data, but Secrets encode values in base64 (with optional encryption at rest) for sensitive data like passwords and API keys, while ConfigMaps store plaintext configuration like environment variables or config files. Secrets can also be mounted with stricter filesystem permissions. Neither provides end-to-end encryption in transit; use external secret stores (e.g., HashiCorp Vault) for production.

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  APP_ENV: production
  LOG_LEVEL: info
---
apiVersion: v1
kind: Secret
metadata:
  name: db-secret
type: Opaque
data:
  DB_PASSWORD: cGFzc3dvcmQxMjM=  # base64-encoded
```

---

**Question**: What are Helm charts and how do they simplify Kubernetes deployments?

**Answer**: Helm is a package manager for Kubernetes that bundles related YAML manifests into a reusable chart with templating support. Charts parameterize configuration via `values.yaml`, enabling environment-specific overrides, versioned releases, and easy rollback – reducing repetitive YAML management.

```yaml
# values.yaml
replicaCount: 3
image:
  repository: myapp
  tag: v1.2.3
service:
  port: 80
  type: ClusterIP
```

```bash
helm upgrade --install myapp ./charts/myapp \
  --set image.tag=v1.2.3 \
  --set replicaCount=5
```

---

**Question**: How do you enforce Pod security in Kubernetes?

**Answer**: Use Pod Security Standards (Privileged, Baseline, Restricted) via admission controllers (Pod Security Admission) or third-party tools like OPA/Gatekeeper or Kyverno. At the Pod level, set `securityContext` to drop capabilities, run as non-root, use read-only root filesystems, and seccomp profiles.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: secure-pod
spec:
  securityContext:
    runAsNonRoot: true
    runAsUser: 1000
    fsGroup: 2000
  containers:
  - name: app
    image: myapp:1.0
    securityContext:
      allowPrivilegeEscalation: false
      readOnlyRootFilesystem: true
      capabilities:
        drop: ["ALL"]
      seccompProfile:
        type: RuntimeDefault
```

---

**Question**: What is the difference between resource requests and limits in Kubernetes?

**Answer**: Requests guarantee the minimum resources a container is scheduled with; the scheduler uses them to place Pods on nodes with sufficient capacity. Limits cap the maximum resources a container can consume. Exceeding the CPU limit throttles the container; exceeding the memory limit kills (OOMKill) the container. Always set both for predictability.

```yaml
resources:
  requests:
    memory: "256Mi"
    cpu: "250m"
  limits:
    memory: "512Mi"
    cpu: "500m"
```

---

**Question**: How does the Horizontal Pod Autoscaler (HPA) work?

**Answer**: HPA automatically scales the number of Pod replicas based on observed CPU, memory, or custom metrics (via Prometheus Adapter). It periodically checks the current metric value against the target and adjusts replicas using the formula `desiredReplicas = ceil(currentMetricValue / targetMetricValue * currentReplicas)`.

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: app-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: myapp
  minReplicas: 2
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
```

---

**Question**: What are init containers and sidecar containers, and when would you use each?

**Answer**: Init containers run to completion before the main app containers start, useful for setup tasks like database migrations, permission changes, or waiting for dependencies. Sidecar containers run alongside the main container for its entire lifetime, handling cross-cutting concerns like logging (Fluentd), service mesh proxies (Envoy), or secret rotation.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: app-with-init
spec:
  initContainers:
  - name: db-migrate
    image: myapp-migrations:1.0
    command: ["rake", "db:migrate"]
  containers:
  - name: main-app
    image: myapp:1.0
  - name: sidecar-logger
    image: fluentd:v1.16
    volumeMounts:
    - name: logs
      mountPath: /var/log/app
  volumes:
  - name: logs
    emptyDir: {}
```

---

**Question**: What are the key stages in a CI/CD pipeline and their purpose?

**Answer**: The **build stage** compiles code and produces artifacts (binaries, images). The **test stage** runs unit, integration, and security scans to catch regressions early. The **deploy stage** promotes artifacts through environments (dev → staging → production) with gated approvals or automated rollouts.

```yaml
# GitHub Actions example
name: CI/CD Pipeline
on: [push]
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v4
    - run: npm ci && npm run build

  test:
    needs: build
    runs-on: ubuntu-latest
    steps:
    - run: npm run test && npm run lint

  deploy:
    needs: test
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    steps:
    - run: ./deploy.sh
```

---

**Question**: How do you manage and version build artifacts in a CI/CD pipeline?

**Answer**: Artifacts are stored in a binary repository manager like Nexus or Artifactory, versioned by build number or Git commit SHA. Immutable versioning ensures traceability – each artifact is uniquely tagged and never overwritten, enabling deterministic rollbacks to any previous version.

```bash
# Publish artifact with version
VERSION=$(git rev-parse --short HEAD)
docker build -t myapp:$VERSION .
docker tag myapp:$VERSION myrepo:5000/myapp:$VERSION
docker push myrepo:5000/myapp:$VERSION
```

---

**Question**: What is a canary deployment and how do you implement it?

**Answer**: Canary deployment gradually shifts a small percentage of traffic (e.g., 5%) to the new version while monitoring error rates and latency. If metrics remain healthy, traffic increments until 100% is reached. In Kubernetes, this is achieved via Service mesh traffic splitting or multiple Deployments with a shared selector.

```yaml
# Using Flagger with Istio for automated canary
apiVersion: flagger.app/v1beta1
kind: Canary
metadata:
  name: myapp
spec:
  targetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: myapp
  service:
    port: 80
  analysis:
    interval: 1m
    threshold: 10
    maxWeight: 50
    stepWeight: 10
    metrics:
    - name: request-success-rate
      thresholdRange:
        min: 99
    webhooks:
    - name: load-test
      url: http://loadtester.flagger
      timeout: 5s
```

---

**Question**: How do blue-green deployments minimize downtime during releases?

**Answer**: Two identical environments (blue = current, green = new) run simultaneously. Once green passes health checks, the router or load balancer switches traffic from blue to green instantly. If issues arise, reverting means switching back to blue – a simple router change rather than a full redeploy.

```yaml
# Switching traffic via Kubernetes Service label selector
apiVersion: v1
kind: Service
metadata:
  name: myapp
spec:
  selector:
    app: myapp
    version: green  # Toggle between "blue" and "green"
  ports:
  - port: 80
```

---

**Question**: What rollback strategies exist for failed deployments?

**Answer**: **Revert**: undo the deployment to the previous known-good version (e.g., `kubectl rollout undo`). **Roll forward**: deploy a new version that fixes the issue without reverting other changes. Revert is faster but may lose fixes; roll forward preserves all changes but requires immediate development effort. Kubernetes supports both via Deployment revision history.

```bash
# Revert to previous revision
kubectl rollout undo deployment/myapp

# Roll forward to a fixed version
kubectl set image deployment/myapp myapp=myapp:v1.2.4-hotfix
```

---

**Question**: What is GitOps and how do ArgoCD or Flux implement it?

**Answer**: GitOps uses a Git repository as the single source of truth for declarative infrastructure and application configuration. ArgoCD and Flux continuously sync the cluster state with the Git repo, automatically applying any changes committed to the desired branch. This provides audit trails, easy rollbacks (via `git revert`), and developer-friendly workflows.

```yaml
# ArgoCD Application manifest
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: myapp
spec:
  destination:
    namespace: production
    server: https://kubernetes.default.svc
  source:
    repoURL: https://github.com/team/manifests.git
    targetRevision: HEAD
    path: environments/production
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
```

---

**Question**: How does Terraform manage infrastructure state and why is it important?

**Answer**: Terraform tracks all managed resources in a state file (`.tfstate`) that maps configuration to real-world infrastructure. The state enables Terraform to detect drift, compute diffs, and execute precise changes. State must be stored remotely (e.g., S3, Azure Storage) with locking (e.g., DynamoDB) to prevent concurrent modification.

```hcl
terraform {
  backend "s3" {
    bucket         = "mycompany-terraform-state"
    key            = "prod/network.tfstate"
    region         = "us-east-1"
    dynamodb_table = "terraform-state-lock"
    encrypt        = true
  }
}
```

---

**Question**: What are Terraform modules and workspaces used for?

**Answer**: Modules encapsulate reusable infrastructure components (e.g., VPC module, EKS module) with input/output variables, promoting DRY configurations across projects. Workspaces allow managing multiple environments (dev, staging, prod) from the same configuration using separate state files, ideal for environment isolation.

```hcl
# Calling a module
module "vpc" {
  source   = "terraform-aws-modules/vpc/aws"
  version  = "5.0.0"
  name     = "myapp-vpc"
  cidr     = "10.0.0.0/16"
  azs      = ["us-east-1a", "us-east-1b"]
  private_subnets = ["10.0.1.0/24", "10.0.2.0/24"]
  public_subnets  = ["10.0.101.0/24", "10.0.102.0/24"]
}
```

```bash
terraform workspace new staging
terraform workspace select prod
```

---

**Question**: How does Pulumi compare to Terraform for infrastructure as code?

**Answer**: Pulumi uses general-purpose programming languages (TypeScript, Python, Go, C#) instead of HCL, enabling loops, conditionals, and shared libraries natively. Terraform has richer provider ecosystem and mature state management. Pulumi suits teams that prefer real code; Terraform is better for Ops-heavy teams with strict policy-as-code needs.

```typescript
// Pulumi example (TypeScript)
import * as aws from "@pulumi/aws";

const bucket = new aws.s3.BucketV2("my-bucket");

new aws.s3.BucketVersioningV2("my-bucket-versioning", {
    bucket: bucket.id,
    versioningConfiguration: { status: "Enabled" },
});

new aws.s3.BucketAcl("my-bucket-acl", {
    bucket: bucket.id,
    acl: "private",
});

export const bucketName = bucket.id;
```

---

**Question**: What is the difference between Ansible's push and pull modes?

**Answer**: In push mode, the control node connects to managed nodes via SSH and executes playbooks – the default and most common approach. In pull mode (`ansible-pull`), each node periodically fetches a playbook from a Git repo and runs it locally. Pull mode scales better for thousands of nodes but adds complexity around scheduling and reporting.

---

**Question**: What distinguishes immutable infrastructure from mutable infrastructure?

**Answer**: In mutable infrastructure, servers are updated in-place via configuration management (e.g., SSH + Ansible), leading to configuration drift and snowflake servers. Immutable infrastructure replaces entire servers on every change – new AMIs or container images are built and deployed, ensuring identical, reproducible environments. Immutable is safer for production but slower for experimental changes.

---

**Question**: How do you manage secrets in infrastructure as code (IaC)?

**Answer**: Never hardcode secrets in configuration files. Use tools like HashiCorp Vault, AWS Secrets Manager, or Azure Key Vault to store and rotate secrets dynamically. In Terraform, reference secrets via data sources (e.g., `aws_secretsmanager_secret_version`) or use `terraform` with `sensitive = true`. For GitOps, use SealedSecrets or External Secrets Operator.

```hcl
# Terraform referencing AWS Secrets Manager
data "aws_secretsmanager_secret_version" "db_password" {
  secret_id = "prod/db/password"
}

resource "aws_db_instance" "main" {
  engine         = "postgres"
  username       = "admin"
  password       = data.aws_secretsmanager_secret_version.db_password.secret_string
  skip_final_snapshot = true
}
```

---

**Question**: How do VPC, subnets, security groups, and NACLs interact in AWS networking?

**Answer**: A VPC is an isolated virtual network. Subnets divide the VPC into public (with Internet Gateway) and private (without direct internet) segments. Security Groups are stateful firewalls at the instance/ENI level that allow only specified inbound/outbound traffic. NACLs are stateless firewalls at the subnet level, supporting allow/deny rules. NACLs are evaluated first (subnet level, stateless), then Security Groups (instance level, stateful) — both must permit traffic for it to flow.

---

**Question**: When would you choose EC2 vs Lambda vs ECS vs EKS for compute?

**Answer**: Use **EC2** for full control, long-running workloads, or specialized hardware. **Lambda** for event-driven, short-lived functions with unpredictable traffic – it auto-scales but has cold starts and 15-minute timeout limits. **ECS** (Fargate) for containers without managing control plane. **EKS** for Kubernetes-native workflows, multi-cloud portability, or complex orchestration needs.

---

**Question**: How do S3 storage classes differ, and when should you transition objects between them?

**Answer**: **S3 Standard** for frequently accessed data. **S3 Intelligent-Tiering** auto-shifts between tiers based on access patterns. **S3 Standard-IA** and **One Zone-IA** for infrequent access at lower cost. **Glacier** and **Glacier Deep Archive** for archival with retrieval times from minutes to hours. Use S3 Lifecycle Policies to automate transitions (e.g., Standard → IA after 30 days → Glacier after 90 days).

```json
{
  "Rules": [
    {
      "Id": "archive-rule",
      "Status": "Enabled",
      "Filter": { "Prefix": "logs/" },
      "Transitions": [
        { "Days": 30, "StorageClass": "STANDARD_IA" },
        { "Days": 90, "StorageClass": "GLACIER" }
      ],
      "Expiration": { "Days": 365 }
    }
  ]
}
```

---

**Question**: How does RDS differ from Aurora, and when would you pick one over the other?

**Answer**: RDS is a managed relational database service supporting multiple engines (MySQL, PostgreSQL, SQL Server, Oracle) with automated backups and patching. Aurora is a MySQL/PostgreSQL-compatible database with up to 5x throughput, 6-way replication across AZs, and automatic storage scaling up to 128 TB. Choose RDS for engine flexibility or cost sensitivity; choose Aurora for higher performance, availability, and auto-scaling storage.

---

**Question**: What considerations drive a multi-region architecture on AWS?

**Answer**: Multi-region architecture provides disaster recovery (active-passive or active-active), lower latency for global users, and regulatory compliance for data residency. Key considerations include data replication lag, cross-region traffic costs, Route 53 routing policies (latency-based, geolocation), and eventual consistency challenges. Use services like Aurora Global Database, DynamoDB Global Tables, or S3 Cross-Region Replication.

---

**Question**: How do Auto Scaling Groups and launch templates work together?

**Answer**: A launch template defines instance configuration (AMI, instance type, security groups, user data). An Auto Scaling Group (ASG) uses the launch template to automatically launch or terminate instances based on scaling policies (e.g., average CPU > 70%) and maintains a desired/min/max count across AZs. The launch template can have multiple versions, enabling rolling updates.

```hcl
resource "aws_launch_template" "web" {
  name          = "web-template"
  image_id      = "ami-0abcdef1234567890"
  instance_type = "t3.medium"
  user_data     = base64encode(file("userdata.sh"))

  network_interfaces {
    associate_public_ip_address = false
    security_groups             = [aws_security_group.web.id]
  }
}

resource "aws_autoscaling_group" "web" {
  name               = "web-asg"
  vpc_zone_identifier = aws_subnet.private[*].id
  min_size           = 2
  max_size           = 10
  desired_capacity   = 3

  launch_template {
    id      = aws_launch_template.web.id
    version = "$Latest"
  }
}
```

---

**Question**: What are Spot Instances and how do you handle interruptions?

**Answer**: Spot Instances offer up to 90% discount by using spare EC2 capacity, but can be reclaimed with a 2-minute warning. Handle interruptions by using Spot Instance interruption notices (via CloudWatch Events or Instance Metadata), designing stateless workloads, diversifying instance types/pools, and using Spot Fleet or EC2 Auto Scaling with mixed instances. Ideal for batch processing, CI/CD runners, and fault-tolerant workloads.

---

**Question**: What are key cloud cost optimization strategies beyond simply reducing usage?

**Answer**: Use **Reserved Instances** or **Savings Plans** for predictable, steady-state workloads (up to 72% discount). **Right-size** instances by analyzing CloudWatch metrics and switching to appropriate instance families. Use **Spot Instances** for fault-tolerant workloads. Implement **auto-scaling** to match capacity to demand. Use **S3 Lifecycle Policies** to archive or delete stale data. Tag resources for chargeback visibility.

```bash
# Compute Optimizer recommendation example
aws compute-optimizer get-ec2-instance-recommendations \
  --instance-arn arn:aws:ec2:us-east-1:123456789012:instance/i-abc123
```

---

**Question**: What are the key tradeoffs of serverless architecture (Lambda)?

**Answer**: Serverless eliminates infrastructure management and scales automatically to zero, reducing costs for variable workloads. Tradeoffs include **cold starts** (latency when a new Lambda sandbox spins up – 100ms–1s for Java/C#, faster for Python/Node), **15-minute timeout**, limited execution environments (up to 10 vCPUs depending on memory configuration), **statelessness** (no local filesystem persistence), and **vendor lock-in** for advanced integrations.

```python
# Mitigate cold starts with provisioned concurrency
# (AWS Console or CloudFormation)
# ProvisionedConcurrencyConfig:
#   ProvisionedConcurrentExecutions: 10
```

---

**Question**: How do you collect and query metrics with Prometheus?

**Answer**: Prometheus scrapes metrics from instrumented targets (applications, infrastructure) via HTTP endpoints exposing a standard text format. Metrics are stored in a time-series database and queried using PromQL – a functional query language with aggregation, filtering, and rate calculations. Alerting rules evaluated against PromQL expressions trigger notifications via Alertmanager.

```promql
# PromQL: 99th percentile request latency over 5 minutes
histogram_quantile(0.99,
  sum(rate(http_request_duration_seconds_bucket[5m])) by (le, service)
)

# Alert: high error rate
rate(http_requests_total{status=~"5.*"}[5m]) / rate(http_requests_total[5m]) > 0.05
```

---

**Question**: How do you build effective Grafana dashboards for SRE monitoring?

**Answer**: Organize dashboards by service or domain with a logical top-to-bottom flow: red/green summary row → request rates and error rates → latency histograms (p50, p95, p99) → resource utilization (CPU, memory, disk) → business metrics. Use templating variables for environment or service selection. Avoid chart junk – each panel should answer a specific operational question.

---

**Question**: How does log aggregation work with the ELK stack (Elasticsearch, Logstash, Kibana) or Grafana Loki?

**Answer**: Filebeat or Fluentd ships logs from containers/instances to Logstash (for transformation) or directly to Elasticsearch. Elasticsearch indexes and stores the logs for full-text search. Kibana provides visualization and dashboards. Loki is a Prometheus-inspired log system that indexes only metadata labels, not full text, making it cheaper and faster for Kubernetes-native log aggregation when combined with Promtail.

```yaml
# Promtail config for Loki
scrape_configs:
- job_name: kubernetes-pods
  kubernetes_sd_configs:
  - role: pod
  pipeline_stages:
  - cri: {}
  - regex:
      expression: "^(?P<level>\\w+)\\s+(?P<message>.*)"
  relabel_configs:
  - source_labels: [__meta_kubernetes_pod_label_app]
    target_label: app
```

---

**Question**: How does Alertmanager route alerts in Prometheus, and what are common routing strategies?

**Answer**: Alertmanager receives alerts from Prometheus and routes them based on labels using a routing tree. Routes can group alerts by severity, service, or team to reduce notification noise. Common strategies include: critical alerts routed to PagerDuty/Opsgenie with immediate escalation, warning alerts to Slack during business hours only, and informational alerts to email digests.

```yaml
route:
  receiver: 'default'
  routes:
  - match:
      severity: critical
    receiver: pagerduty-critical
    group_wait: 10s
    group_interval: 1m
    repeat_interval: 30m
  - match:
      severity: warning
    receiver: slack-warnings
    group_wait: 1m
    group_interval: 5m
    repeat_interval: 4h
receivers:
- name: pagerduty-critical
  pagerduty_configs:
  - routing_key: "<key>"
    severity: critical
```

---

**Question**: How do you triage and respond to incidents based on severity levels?

**Answer**: Establish severity definitions (SEV1: service down, SEV2: degraded, SEV3: minor). SEV1 triggers immediate page to on-call with 5-minute response SLA; SEV2 pages within 15 minutes; SEV3 is addressed during business hours. On-call acknowledges, diagnoses, mitigates (even if mitigation is not a full fix), then documents the timeline and root cause. Follow a "swarming" model for SEV1 – multiple responders converge immediately rather than hierarchical escalation.

---

**Question**: What should a runbook include for effective incident response?

**Answer**: A runbook contains: (1) symptoms and how to detect them, (2) severity classification, (3) step-by-step diagnosis commands or dashboards to check, (4) mitigation steps (restart, rollback, scale up), (5) escalation contacts, (6) post-mitigation validation. Runbooks must be version-controlled, tested during game days, and updated after every incident.

```bash
# Example runbook: High error rate in web service
# 1. Check error rate dashboard
# 2. Check recent deployments
kubectl rollout history deployment/web
# 3. Check Pod status
kubectl get pods -l app=web
# 4. Check logs
kubectl logs -l app=web --tail=100 | grep ERROR
# 5. If bad deploy: rollback
kubectl rollout undo deployment/web
# 6. Verify recovery
```

---

**Question**: What is an error budget and how does burn rate affect SLO compliance?

**Answer**: An error budget = 100% - SLO target (e.g., 99.9% SLO gives 0.1% error budget = 8.64 hours of downtime per year). Burn rate measures how fast the error budget is consumed. A burn rate of 1.0 consumes the budget evenly over the window; 2.0 exhausts it in half the time. Multi-window, multi-burn-rate alerts fire when budget is depleting faster than expected over both short and long windows, reducing false positives.

```promql
# Multi-window, multi-burn-rate alert: 5m and 30m windows
- alert: HighErrorRate
  expr: |
    (
      sum(rate(http_requests_total{status=~"5.*"}[5m]))
      / sum(rate(http_requests_total[5m]))
    ) > 0.001  # 0.1% error rate → SLO 99.9% breached
    and
    (
      sum(rate(http_requests_total{status=~"5.*"}[30m]))
      / sum(rate(http_requests_total[30m]))
    ) > 0.0005  # slower burn rate check
  for: 2m
  labels:
    severity: critical
```

---

**Question**: How do you identify and reduce toil in SRE practice?

**Answer**: Toil is manual, repetitive, automatable, and non-valuable work (e.g., manually restarting Pods, creating tickets for capacity). Identify it by tracking time spent on operational tasks and categorizing by frequency. Set automation targets (e.g., "automate 80% of toil by Q3") and invest in tooling – auto-remediation via webhooks, ChatOps bots, or Kubernetes operators that handle common failure modes automatically.

---

**Question**: How do you approach capacity planning for a growing system?

**Answer**: Combine trend-based forecasting (using historical metrics like CPU, memory, requests/sec) with load testing to model future growth. Use tools like AWS Compute Optimizer or Prometheus + Prophet for prediction. Maintain a buffer of 20-30% headroom for traffic spikes. Implement auto-scaling policies that scale proactively based on leading indicators (e.g., request queue depth) rather than reactive CPU thresholds.

---

**Question**: What makes a blameless postmortem effective in incident response?

**Answer**: A blameless postmortem focuses on systemic failures rather than individual mistakes – the goal is to improve the system, not assign fault. It documents the timeline, impact, root cause, contributing factors, and actionable action items (with owners and deadlines). Key principles: assume good faith, share openly, and treat every incident as a learning opportunity. Conduct postmortems within 48 hours while details are fresh.

---

**Question**: How do SLI, SLO, and SLA relate to each other in practice?

**Answer**: SLI (Service Level Indicator) is a raw metric like request latency or error rate. SLO (Service Level Objective) is a target threshold for the SLI (e.g., 99.9% of requests complete in <200ms). SLA (Service Level Agreement) is a contractual commitment derived from the SLO, often with financial penalties. SLOs should be tighter than SLAs to provide an internal safety buffer.

---

**Question**: How does DNS resolution work, and what are A, CNAME, and ALIAS records used for?

**Answer**: DNS resolves human-readable domain names to IP addresses. An **A record** maps a domain to an IPv4 address directly. A **CNAME** maps an alias domain to another canonical domain (e.g., `www.example.com` → `example.com`) – it cannot coexist with other record types at the same name. An **ALIAS** record (Route 53-specific) behaves like CNAME but can be used at the zone apex and automatically resolves to the target's A/AAAA records.

---

**Question**: What are the key differences between HTTP/1.1, HTTP/2, and HTTP/3 (QUIC)?

**Answer**: HTTP/1.1 supports persistent connections (keep-alive); browsers historically limited concurrent connections to ~6 per domain. HTTP/2 multiplexes multiple streams over a single TCP connection, reducing latency via header compression (HPACK) and server push. HTTP/3 uses QUIC (UDP-based) instead of TCP, eliminating head-of-line blocking even if a packet is lost, reducing connection establishment to 0-1 RTT, and handling network changes gracefully.

---

**Question**: How does an L4 load balancer differ from an L7 load balancer?

**Answer**: An L4 load balancer (e.g., AWS NLB) operates at the transport layer, forwarding TCP/UDP traffic based on IP and port without inspecting the payload – it's fast, preserves client IP, and handles millions of requests per second. An L7 load balancer (e.g., AWS ALB) operates at the application layer, inspecting HTTP/HTTPS headers, paths, and cookies for content-based routing, SSL termination, and sticky sessions.

| Feature | L4 (NLB) | L7 (ALB) |
|---------|----------|----------|
| Layer | Transport (TCP/UDP) | Application (HTTP/HTTPS) |
| Routing | IP:Port | Host/Path/Headers |
| SSL Termination | No | Yes |
| Performance | Very high (millions RPS) | Moderate (~100k RPS) |

---

**Question**: What is a service mesh (e.g., Istio, Linkerd) and what problems does it solve?

**Answer**: A service mesh injects a sidecar proxy alongside each service to handle inter-service communication transparently. It provides mTLS encryption between all pods, traffic shifting for canary deployments, circuit breaking, observability (metrics, traces, logs), and fine-grained access policies – all without modifying application code. Istio uses Envoy proxies; Linkerd uses its own ultralight proxy written in Rust.

```yaml
# Istio VirtualService for traffic shifting
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: myapp
spec:
  hosts:
  - myapp
  http:
  - route:
    - destination:
        host: myapp
        subset: v1
      weight: 90
    - destination:
        host: myapp
        subset: v2
      weight: 10
```

---

**Question**: How does mTLS work in a service mesh and why is it important?

**Answer**: mTLS (mutual TLS) requires both client and server to present certificates for authentication. In a service mesh, the sidecar proxy automatically issues and rotates short-lived certificates via the control plane (e.g., Istio's Citadel or SPIRE). This ensures all inter-service traffic is encrypted and authenticated, preventing man-in-the-middle attacks and enforcing zero-trust networking – critical for compliance (PCI-DSS, HIPAA).

---

**Question**: What are the tradeoffs between serverless containers (Fargate) and serverless functions (Lambda)?

**Answer**: Fargate abstracts EC2 management while still running long-lived containers with no timeout limits, supporting any runtime and persistent storage (EFS). Lambda has sub-second billing granularity and auto-scales to zero but imposes timeouts (15 min), execution environment limits, and cold start latency. Choose Fargate for stateful or long-running workloads; choose Lambda for event-driven, bursty tasks.
