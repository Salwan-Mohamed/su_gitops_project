# GitOps Platform - Complete Project Map

**Visual guide to everything we've built and what's coming next**

---

## ���️ The Big Picture
```
GitHub Repository (Source of Truth)
        ↓
    ArgoCD (GitOps Engine)
        ↓
    ┌─────────────────────────────────────┐
    │   Kubernetes Cluster (7 nodes)      │
    │   ├── Infrastructure (Foundation)   │
    │   ├── Monitoring (Platform)         │
    │   └── Applications (Coming Phase 3) │
    └─────────────────────────────────────┘
```

---

## ��� Phase 1: Foundation (COMPLETE)

### What We Built

#### 1. **ArgoCD Installation**
- **What**: GitOps deployment tool
- **Where**: `argocd` namespace
- **Purpose**: Automatically deploy changes from Git to Kubernetes
- **Access**: http://10.1.5.184
- **Key Feature**: Watches Git repo, applies changes automatically

#### 2. **Root Application (App-of-Apps)**
- **File**: `bootstrap/root-app.yaml`
- **What**: Master application that manages all other applications
- **Purpose**: Single entry point for entire platform
- **How it works**: You apply this once, it creates everything else
- **Pattern**: Called "App-of-Apps" - one app creates many apps

#### 3. **AppProjects (RBAC)**
- **Files**: `infrastructure/base/projects/*.yaml`
- **What**: Security boundaries for different teams/purposes
- **Created**:
  - `infrastructure` - For cluster-level stuff
  - `platform-services` - For monitoring, logging, etc.
  - `applications` - For team applications
- **Purpose**: Control who can deploy what and where

#### 4. **Namespaces**
- **Files**: `infrastructure/base/namespaces/*.yaml`
- **What**: Logical separations in Kubernetes
- **Created**:
  - `argocd` - ArgoCD components
  - `platform-monitoring` - Monitoring stack
- **Purpose**: Organize and isolate resources

#### 5. **Git Repository Structure**
```
su_gitops_project/
├── bootstrap/           # Entry point
├── infrastructure/      # Cluster foundations
├── platform-services/   # Platform tools
├── applications/        # Team apps (Phase 3)
└── docs/               # Documentation
```

### How Phase 1 Works

1. You commit YAML files to GitHub
2. ArgoCD detects the change
3. ArgoCD applies changes to Kubernetes
4. Resources are created/updated automatically
5. All changes tracked in Git history

---

## ��� Phase 2: Monitoring Platform (COMPLETE)

### What We Built

#### 1. **Prometheus** (Metrics Database)
- **What**: Collects and stores metrics from everything
- **How**: Scrapes metrics from 73 targets every 30 seconds
- **Storage**: 20Gi NFS volume (15 days retention)
- **Collects**:
  - CPU, memory, disk from all nodes
  - Pod status and health
  - Network traffic
  - Application metrics
- **Why**: Foundation for knowing what's happening in cluster

#### 2. **Grafana** (Visualization Dashboard)
- **What**: Beautiful graphs and dashboards
- **Where**: http://10.1.5.186
- **Purpose**: Visualize all metrics collected by Prometheus
- **Features**:
  - Interactive graphs
  - Real-time updates
  - Query builder
  - Dashboard templates
- **Current State**: Working, showing 73 healthy targets

#### 3. **AlertManager** (Alert Router)
- **What**: Manages and routes alerts
- **Storage**: 5Gi NFS volume
- **Purpose**: 
  - Receive alerts from Prometheus
  - Route to Slack/Email/PagerDuty
  - Deduplicate and group alerts
- **Status**: Running, ready for Phase 3 configuration

#### 4. **Node Exporter** (Host Metrics)
- **What**: Runs on every node collecting host-level metrics
- **Deployment**: DaemonSet (1 pod per node)
- **Pods**: 7 total (3 masters + 4 workers)
- **Collects**:
  - CPU usage per core
  - Memory usage
  - Disk I/O
  - Network stats
  - System load

#### 5. **Kube State Metrics** (K8s Object Metrics)
- **What**: Exposes Kubernetes object states as metrics
- **Monitors**:
  - Pod status (running, pending, failed)
  - Deployment replicas
  - Service endpoints
  - ConfigMaps, Secrets
- **Why**: Know when deployments fail, pods crash, etc.

#### 6. **Prometheus Operator** (Automation)
- **What**: Makes Prometheus easier to manage
- **Manages**:
  - ServiceMonitors (what to scrape)
  - PodMonitors (pod metrics)
  - PrometheusRules (alert rules)
- **Why**: Declarative monitoring configuration

### How Phase 2 Works
```
Application → Exposes Metrics (:9090/metrics)
                    ↓
ServiceMonitor (tells Prometheus to scrape it)
                    ↓
Prometheus (collects and stores metrics)
                    ↓
Grafana (queries Prometheus and shows graphs)
                    ↓
AlertManager (sends alerts if problems)
```

### Current Monitoring Coverage

- ✅ **7 nodes** - All masters and workers
- ✅ **73 targets** - Everything we care about
- ✅ **30 second** scrape interval
- ✅ **15 days** data retention
- ✅ **ArgoCD metrics** - Track deployments
- ✅ **Kubernetes metrics** - Cluster health

---

## ���️ Architecture Decisions Made

### Storage Strategy
- **Choice**: NFS (nfs-client StorageClass)
- **Why**: 
  - Already available in cluster
  - Supports ReadWriteMany
  - Reliable for monitoring data
- **Alternative**: OpenEBS (kept as backup option)

### Deployment Strategy
- **Choice**: Separate Grafana from Helm chart
- **Why**: 
  - Simpler (no sidecar complexity)
  - Easier to troubleshoot
  - More stable
- **Lesson**: Simplicity > Complex Helm subcharts

### Node Placement
- **Choice**: Exclude worker04 from workloads
- **Why**:
  - worker04 has HDD (slow for monitoring)
  - worker04 out of cluster subnet
  - worker04 reserved for backups
- **Exception**: Node Exporter still runs there for metrics

### GitOps Workflow
- **Choice**: Everything via Git commits
- **Why**:
  - Complete audit trail
  - Easy rollback
  - Team collaboration
  - Disaster recovery

---

## ��� Every File We Created

### Bootstrap Layer (Entry Point)
```
bootstrap/
├── root-app.yaml                    # App-of-Apps master
└── apps/
    ├── infrastructure.yaml          # Manages projects/namespaces
    ├── platform-services.yaml       # Manages platform components
    ├── monitoring.yaml              # Prometheus stack (Helm)
    └── grafana.yaml                 # Grafana standalone
```

### Infrastructure Layer (Foundation)
```
infrastructure/base/
├── projects/
│   ├── infrastructure.yaml          # Infrastructure AppProject
│   ├── platform-services.yaml       # Platform AppProject
│   ├── applications.yaml            # Applications AppProject
│   └── kustomization.yaml
└── namespaces/
    ├── argocd.yaml                  # ArgoCD namespace
    ├── platform-monitoring.yaml     # Monitoring namespace
    └── kustomization.yaml
```

### Platform Services Layer (Monitoring)
```
platform-services/base/
├── monitoring/
│   ├── namespace.yaml               # Namespace definition
│   └── kustomization.yaml
└── grafana/
    ├── deployment.yaml              # Grafana pod + service + config
    └── kustomization.yaml
```

### Documentation
```
docs/
├── PHASE1-COMPLETE.md               # Phase 1 summary
├── PHASE2-COMPLETE.md               # Phase 2 details
├── SESSION-SUMMARY.md               # Session recap
├── TROUBLESHOOTING.md               # Problem solving
├── QUICK-REFERENCE.md               # Command cheat sheet
└── PROJECT-MAP.md                   # This file
```

### Root Files
```
├── README.md                        # Project overview
└── .gitignore                       # Git exclusions
```

---

## ��� What Each Component Does (Simple Explanation)

### ArgoCD
**Like**: A robot that watches your Git repo  
**Does**: When you commit changes, it applies them to Kubernetes  
**Why**: No manual kubectl commands needed

### Prometheus
**Like**: A security camera recording everything  
**Does**: Records metrics every 30 seconds from everything  
**Why**: So you know what happened when things break

### Grafana
**Like**: A TV that shows your security camera footage  
**Does**: Shows beautiful graphs of all your metrics  
**Why**: Humans understand pictures better than numbers

### AlertManager
**Like**: A smoke detector  
**Does**: Sends notifications when something is wrong  
**Why**: You can't watch graphs 24/7

### Node Exporter
**Like**: A health monitor on each server  
**Does**: Reports CPU, memory, disk usage  
**Why**: Know when a server is overloaded

### Kube State Metrics
**Like**: A reporter that tells you about Kubernetes objects  
**Does**: Reports if pods are running, deployments are healthy  
**Why**: Kubernetes doesn't expose this info otherwise

---

## ��� By The Numbers (Current State)

### Cluster
- **Nodes**: 7 total (3 masters, 4 workers)
- **Namespaces**: 10+ (kube-system, argocd, platform-monitoring, etc.)
- **Running Pods**: 50+ across entire cluster

### Monitoring Stack
- **Monitoring Pods**: 12 running
- **Metrics Targets**: 73 scraped
- **Data Points**: ~1.3 million (73 targets × 30s × 15 days)
- **Scrape Interval**: 30 seconds
- **Data Retention**: 15 days

### Storage
- **Prometheus PVC**: 20Gi (bound to NFS)
- **AlertManager PVC**: 5Gi (bound to NFS)
- **Grafana**: No persistence (temporary)

### Network
- **LoadBalancers**: 2 assigned
  - ArgoCD: 10.1.5.184
  - Grafana: 10.1.5.186

### Git Activity
- **Total Commits**: 20+ during implementation
- **Files Tracked**: 30+ YAML/markdown files
- **Lines of Code**: ~3000+ lines
- **Documentation**: ~5000+ lines

---

## ��� Phase 3: Applications & DORA Metrics (NEXT)

### What We'll Build

#### 1. **DORA Metrics Dashboards** (Week 1)

**Dashboard 1: Deployment Frequency**
- **What**: How often we deploy to production
- **Data Source**: ArgoCD sync events
- **Metrics**: 
  - Deployments per day/week/month
  - Deployment trends over time
  - Per-application deployment frequency
- **Goal**: Elite = Multiple deployments per day

**Dashboard 2: Lead Time for Changes**
- **What**: Time from code commit to production
- **Data Source**: Git commits + ArgoCD sync timestamps
- **Metrics**:
  - Commit timestamp → Deployment timestamp
  - Average lead time per application
  - Lead time trends
- **Goal**: Elite = Less than 1 hour

**Dashboard 3: Mean Time to Recovery (MTTR)**
- **What**: How fast we recover from failures
- **Data Source**: Pod restart events, deployment failures
- **Metrics**:
  - Time from failure to recovery
  - Recovery time trends
  - Failure patterns
- **Goal**: Elite = Less than 1 hour

**Dashboard 4: Change Failure Rate**
- **What**: Percentage of deployments that fail
- **Data Source**: ArgoCD sync status
- **Metrics**:
  - Failed deployments / Total deployments
  - Failure rate trends
  - Root cause analysis
- **Goal**: Elite = 0-15% failure rate

#### 2. **First Application Deployment** (Week 1)

**Sample Application Components:**
- **Frontend**: React/Vue web application
- **Backend**: REST API (Node.js/Python/Go)
- **Database**: PostgreSQL/MySQL
- **Configuration**: ConfigMaps, Secrets

**What We'll Create:**
```
applications/
├── dev/
│   └── sample-app/
│       ├── deployment.yaml          # Pod definition
│       ├── service.yaml             # Internal networking
│       ├── ingress.yaml             # External access
│       ├── configmap.yaml           # Configuration
│       ├── servicemonitor.yaml      # Metrics collection
│       └── kustomization.yaml
├── staging/
│   └── sample-app/                  # Same structure
└── production/
    └── sample-app/                  # Same structure
```

**Application Features:**
- Expose `/metrics` endpoint (Prometheus format)
- Health check endpoints (`/health`, `/ready`)
- Structured logging (JSON format)
- Graceful shutdown
- Resource requests/limits defined

#### 3. **CI/CD Pipeline** (Week 2)

**GitHub Actions Workflow:**
```
Code Commit
    ↓
GitHub Actions triggered
    ↓
1. Run tests (unit, integration)
    ↓
2. Build Docker image
    ↓
3. Push to container registry
    ↓
4. Update image tag in Git (dev environment)
    ↓
5. ArgoCD detects change
    ↓
6. Deploy to dev automatically
    ↓
7. Run smoke tests
    ↓
8. Create PR for staging (manual approval)
    ↓
9. Merge → Deploy to staging
    ↓
10. Create PR for production (manual approval)
    ↓
11. Merge → Deploy to production
```

**Files We'll Create:**
```
.github/workflows/
├── ci-dev.yaml                      # Dev deployment pipeline
├── ci-staging.yaml                  # Staging promotion
└── ci-production.yaml               # Production promotion
```

#### 4. **AlertManager Configuration** (Week 2)

**Notification Channels:**
- **Slack**: Instant notifications to team channel
- **Email**: Backup notification method
- **PagerDuty**: For critical production alerts (optional)

**Alert Rules We'll Create:**
```
1. High CPU Usage (>80% for 5 minutes)
2. High Memory Usage (>90% for 5 minutes)
3. Pod Restart Loop (>5 restarts in 10 minutes)
4. Deployment Failed
5. PVC Almost Full (>85%)
6. Node Not Ready
7. Application Response Time >2s
8. Application Error Rate >5%
```

**What We'll Configure:**
- Alert routing rules (where to send what)
- Alert grouping (don't spam with 100 alerts)
- Alert silencing (maintenance windows)
- Alert escalation (critical → immediate)

#### 5. **Application ServiceMonitors** (Week 2)

**What**: Tell Prometheus to scrape application metrics

**Example ServiceMonitor:**
```yaml
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: sample-app
  namespace: dev-apps
spec:
  selector:
    matchLabels:
      app: sample-app
  endpoints:
  - port: metrics
    interval: 30s
    path: /metrics
```

**Application Metrics to Track:**
- Request count
- Request duration
- Error rate
- Active connections
- Queue depth
- Cache hit rate
- Database query time

---

## ��� Phase 3 Step-by-Step Plan

### Week 1: Dashboards & First App

**Day 1-2: Import Grafana Dashboards**
1. Login to Grafana
2. Go to Dashboards → Import
3. Import dashboard IDs:
   - 1860: Node Exporter Full
   - 315: Kubernetes cluster monitoring
   - 6417: Kubernetes Deployment Stats
4. Verify data is showing

**Day 3-4: Create DORA Metrics Dashboard**
1. Create new dashboard in Grafana
2. Add panels for each DORA metric
3. Write PromQL queries
4. Configure time ranges
5. Add documentation

**Day 5: Deploy First Application**
1. Create application YAML files
2. Commit to Git (dev environment)
3. ArgoCD deploys automatically
4. Verify application is running
5. Check metrics are being collected

### Week 2: CI/CD & Alerts

**Day 1-2: GitHub Actions Pipeline**
1. Create workflow YAML
2. Configure secrets (registry credentials)
3. Test build process
4. Test deployment to dev
5. Document pipeline

**Day 3-4: AlertManager Setup**
1. Configure Slack webhook
2. Create alert routing rules
3. Define alert rules in Prometheus
4. Test alerts (trigger intentional failure)
5. Fine-tune thresholds

**Day 5: Integration & Testing**
1. Full end-to-end test
2. Code commit → Build → Deploy → Monitor
3. Trigger alerts and verify notifications
4. Update documentation
5. Team training

---

## ��� Key Concepts to Understand

### GitOps Workflow
```
Developer writes code
    ↓
Commits to GitHub
    ↓
CI/CD builds & tests
    ↓
Updates deployment YAML in Git
    ↓
ArgoCD sees the change
    ↓
ArgoCD deploys to Kubernetes
    ↓
Prometheus collects metrics
    ↓
Grafana shows dashboards
    ↓
AlertManager sends alerts if problems
```

### App-of-Apps Pattern
```
root-app (you apply this once)
    ↓
├── infrastructure (manages AppProjects + namespaces)
├── platform-services (manages monitoring)
├── monitoring (manages Prometheus stack)
└── grafana (manages Grafana)

Future:
└── applications (will manage team apps)
```

### Monitoring Flow
```
Application exposes /metrics
    ↓
ServiceMonitor tells Prometheus about it
    ↓
Prometheus scrapes /metrics every 30s
    ↓
Prometheus stores data (15 days)
    ↓
Grafana queries Prometheus
    ↓
Grafana shows graphs
    ↓
PrometheusRule checks for problems
    ↓
AlertManager sends notifications
```

---

## ��� Success Metrics

### Phase 1 ✅
- [x] ArgoCD deployed and accessible
- [x] App-of-Apps pattern working
- [x] Git as single source of truth
- [x] All applications synced

### Phase 2 ✅
- [x] 73 targets monitored
- [x] Grafana accessible and working
- [x] Metrics being collected
- [x] Storage configured
- [x] Zero failing pods

### Phase 3 ���
- [ ] DORA dashboards showing data
- [ ] First application deployed
- [ ] CI/CD pipeline operational
- [ ] Alerts configured and tested
- [ ] Team trained on workflow

---

## ���️ The Journey So Far
```
Week 1: Foundation
├── Installed ArgoCD ✅
├── Created repository structure ✅
└── Set up App-of-Apps ✅

Week 2-3: Monitoring
├── Deployed Prometheus ✅
├── Deployed Grafana ✅
├── Configured storage ✅
├── Verified metrics collection ✅
└── Troubleshooted issues ✅

Next: Applications & DORA Metrics
├── Import dashboards
├── Create DORA dashboard
├── Deploy first app
├── Set up CI/CD
└── Configure alerts
```

---

## ��� Why We Built It This Way

### Foundation First (Phase 1)
**Why**: Can't deploy apps without the deployment mechanism (ArgoCD)

### Monitoring Second (Phase 2)
**Why**: Need visibility BEFORE deploying apps, not after

### Applications Last (Phase 3)
**Why**: Infrastructure + Monitoring ready = Safe to deploy apps

### GitOps Approach
**Why**: 
- Everything tracked in Git
- Easy rollback if problems
- Team collaboration
- Audit trail for compliance

---

## ��� What You Have Now

### Working GitOps Platform
- ✅ Automatic deployments from Git
- ✅ Complete cluster monitoring
- ✅ Real-time metrics visualization
- ✅ Foundation for DORA metrics
- ✅ Production-ready infrastructure

### Skills Gained
- ✅ ArgoCD App-of-Apps pattern
- ✅ Prometheus/Grafana setup
- ✅ Kubernetes storage management
- ✅ GitOps workflows
- ✅ Troubleshooting complex systems

### Ready For
- ✅ Team application deployments
- ✅ CI/CD pipeline implementation
- ✅ DORA metrics tracking
- ✅ Multi-environment promotion
- ✅ Enterprise scaling

---

## ��� How to Use This Map

**For Onboarding New Team Members:**
1. Start with "The Big Picture"
2. Read Phase 1 & 2 sections
3. Understand "How It Works" sections
4. Review "Every File We Created"
5. Study Phase 3 plan

**For Troubleshooting:**
1. Identify which phase/component
2. Check "What Each Component Does"
3. Review component's specific section
4. Reference [TROUBLESHOOTING.md](./TROUBLESHOOTING.md)

**For Planning:**
1. Review "Phase 3 Step-by-Step Plan"
2. Understand time estimates
3. Check prerequisites
4. Identify dependencies

---

**Last Updated**: November 23, 2025  
**Current Phase**: Phase 2 Complete ✅  
**Next Phase**: Phase 3 - Applications & DORA Metrics  
**Total Build Time**: ~2 weeks
