# SU GitOps Platform Project

Enterprise GitOps implementation for Kubernetes infrastructure and application management using ArgoCD.

## ��� Project Overview

**Goal**: Implement a complete GitOps platform with DORA metrics tracking for enterprise Kubernetes deployment across development, staging, and production environments.

**Current Status**: Phase 2 Complete ✅

---

## ��� Implementation Status

### ✅ Phase 1: Foundation (Complete)
- [x] ArgoCD installation and configuration
- [x] App-of-Apps pattern implementation
- [x] Project structure setup
- [x] AppProject definitions (infrastructure, platform-services, applications)
- [x] Namespace management
- [x] GitOps workflow established

### ✅ Phase 2: Platform Services (Complete)
- [x] Prometheus metrics collection (20Gi storage)
- [x] Grafana visualization dashboard (http://10.1.5.186)
- [x] AlertManager alert routing (5Gi storage)
- [x] Node Exporter on all 7 cluster nodes
- [x] Kube State Metrics for Kubernetes objects
- [x] Prometheus Operator for CRD management
- [x] Storage configuration (nfs-client StorageClass)
- [x] Node affinity rules (excluding worker04)
- [x] 73 metrics targets monitored
- [x] ArgoCD metrics collection configured

### ��� Phase 3: Application Deployment (Upcoming)
- [ ] DORA metrics dashboards
- [ ] First IS team application deployment
- [ ] CI/CD pipeline (GitHub Actions)
- [ ] Application metrics collection
- [ ] Alert rules and notifications
- [ ] Multi-environment promotion (dev → staging → prod)

---

## ���️ Architecture

### Cluster Configuration
```
Kubernetes Cluster (v1.x)
├── Control Plane (3 nodes)
│   ├── master01
│   ├── master02
│   └── master03
├── etcd (3 external nodes)
└── Worker Nodes (4 nodes)
    ├── worker01 (workloads)
    ├── worker02 (workloads)
    ├── worker03 (workloads)
    └── worker04 (backup - HDD storage, excluded from workloads)
```

### GitOps Structure
```
su_gitops_project/
├── bootstrap/
│   ├── root-app.yaml              # ArgoCD App-of-Apps
│   └── apps/
│       ├── infrastructure.yaml    # AppProjects & namespaces
│       ├── platform-services.yaml # Platform components
│       ├── monitoring.yaml        # Prometheus stack (Helm)
│       └── grafana.yaml          # Grafana visualization
├── infrastructure/
│   └── base/
│       ├── projects/             # AppProject definitions
│       │   ├── infrastructure.yaml
│       │   ├── platform-services.yaml
│       │   └── applications.yaml
│       └── namespaces/
│           ├── argocd.yaml
│           └── platform-monitoring.yaml
├── platform-services/
│   └── base/
│       ├── monitoring/           # Monitoring namespace
│       └── grafana/             # Grafana deployment
│           ├── deployment.yaml
│           └── kustomization.yaml
├── applications/                 # Application deployments (Phase 3)
│   ├── dev/
│   ├── staging/
│   └── production/
└── docs/
    ├── PHASE1-COMPLETE.md
    ├── PHASE2-COMPLETE.md
    └── SESSION-SUMMARY.md
```

---

## ��� Quick Start

### Prerequisites
- Kubernetes cluster (3 masters, 3 etcd, 4 workers)
- ArgoCD installed
- kubectl configured
- GitHub repository access
- MetalLB for LoadBalancer services
- NFS provisioner or OpenEBS for storage

### Deploy the Platform

1. **Bootstrap ArgoCD App-of-Apps**
```bash
kubectl apply -f bootstrap/root-app.yaml
```

2. **Verify Deployment**
```bash
# Check all ArgoCD applications
kubectl get applications -n argocd

# Expected output:
# NAME                    SYNC STATUS   HEALTH STATUS
# root-app                Synced        Healthy
# infrastructure          Synced        Healthy
# platform-services       Synced        Healthy
# kube-prometheus-stack   Synced        Healthy
# grafana                 Synced        Healthy
```

3. **Access Monitoring**
```bash
# ArgoCD UI
http://10.1.5.184
Username: admin
Password: <from secret>

# Grafana UI
http://10.1.5.186
Username: admin
Password: admin
```

---

## ��� Monitoring Stack

### Components

**Prometheus** - Metrics Collection
- Storage: 20Gi NFS (nfs-client)
- Retention: 15 days
- Scrape interval: 30 seconds
- Targets: 73 monitored endpoints

**Grafana** - Visualization
- LoadBalancer: http://10.1.5.186
- Datasource: Prometheus (auto-configured)
- Credentials: admin/admin

**AlertManager** - Alert Routing
- Storage: 5Gi NFS
- Retention: 120 hours
- Ready for Slack/Email notifications

**Node Exporter** - Node Metrics
- DaemonSet: 7/7 pods (all nodes)
- Metrics: CPU, memory, disk, network

**Kube State Metrics** - K8s Object Metrics
- Monitors: Pods, Deployments, Services, etc.

**Prometheus Operator** - CRD Management
- Manages: ServiceMonitors, PodMonitors, PrometheusRules

### Useful Queries
```promql
# Check all targets health
up

# Node memory available
node_memory_MemAvailable_bytes

# Pod status across cluster
kube_pod_status_phase

# ArgoCD application info
argocd_app_info

# Deployment replicas
kube_deployment_status_replicas
```

---

## ��� Configuration

### Storage
- **Default StorageClass**: `nfs-client`
- **Access Mode**: ReadWriteOnce (RWO)
- **Provisioner**: NFS Subdir External Provisioner

### Node Affinity
All platform services avoid `worker04`:
```yaml
affinity:
  nodeAffinity:
    requiredDuringSchedulingIgnoredDuringExecution:
      nodeSelectorTerms:
      - matchExpressions:
        - key: kubernetes.io/hostname
          operator: NotIn
          values:
          - worker04
```

**Reason**: worker04 reserved for backups (HDD storage, out of subnet range)

### ArgoCD Sync Policy
```yaml
syncPolicy:
  automated:
    prune: true
    selfHeal: true
  syncOptions:
    - CreateNamespace=true
  retry:
    limit: 3
```

---

## ��� DORA Metrics (Coming in Phase 3)

**Planned Tracking:**
1. **Deployment Frequency** - How often we deploy
2. **Lead Time for Changes** - Time from commit to production
3. **Mean Time to Recovery (MTTR)** - Time to recover from failures
4. **Change Failure Rate** - % of deployments causing failures

**Data Sources:**
- Git commits (GitHub API)
- ArgoCD sync events
- Prometheus pod restart metrics
- Custom application metrics

---

## ��� Team Structure

**Platform Engineering Team**
- Infrastructure management
- GitOps platform maintenance
- Monitoring and observability

**IS Team** (7-10 applications initially)
- Application development
- Application deployment via GitOps

**Planned: DevOps Team**
- CI/CD pipeline management
- Deployment automation

**Other Teams** (Future)
- System Engineers
- Network Automation
- Helpdesk

---

## ��� Documentation

- [Phase 1 Complete](docs/PHASE1-COMPLETE.md) - Foundation setup
- [Phase 2 Complete](docs/PHASE2-COMPLETE.md) - Monitoring stack
- [Session Summary](docs/SESSION-SUMMARY.md) - Latest session recap

---

## ��� Troubleshooting

### Check ArgoCD Application Status
```bash
kubectl get applications -n argocd
kubectl describe application <app-name> -n argocd
```

### View Pod Logs
```bash
kubectl logs -n <namespace> <pod-name>
kubectl logs -n <namespace> <pod-name> -c <container-name>
```

### Force ArgoCD Sync
```bash
# Via kubectl
kubectl patch application <app-name> -n argocd \
  --type merge -p '{"operation":{"initiatedBy":{"username":"admin"},"sync":{"revision":"HEAD"}}}'

# Via ArgoCD UI
# Click "SYNC" button on application
```

### Check Storage
```bash
# List PVCs
kubectl get pvc -n platform-monitoring

# Check StorageClasses
kubectl get storageclass

# View PV details
kubectl get pv
```

---

## ��� Next Steps

### Phase 3 Priorities
1. Import pre-built Kubernetes dashboards in Grafana
2. Create custom DORA metrics dashboard
3. Deploy first IS team application
4. Implement GitHub Actions CI/CD pipeline
5. Configure AlertManager notifications (Slack/Email)
6. Set up application-specific ServiceMonitors

### Future Enhancements
- Multi-cluster deployment (dev/staging/prod clusters)
- Disaster recovery automation
- Cost monitoring and optimization
- Security scanning integration
- Backup automation for worker04

---

## ��� Support

**Repository**: https://github.com/Salwan-Mohamed/su_gitops_project

**Access Points:**
- ArgoCD UI: http://10.1.5.184
- Grafana UI: http://10.1.5.186

**Key Commands:**
```bash
# Check all applications
kubectl get applications -n argocd

# View monitoring pods
kubectl get pods -n platform-monitoring

# Port-forward Prometheus
kubectl port-forward -n platform-monitoring \
  prometheus-kube-prometheus-stack-prometheus-0 9090:9090
```

---

## ��� Success Metrics

- ✅ 7-node cluster fully monitored
- ✅ 73 healthy metrics targets
- ✅ GitOps workflow operational
- ✅ All ArgoCD apps Synced/Healthy
- ✅ 25Gi storage provisioned
- ✅ Zero failing pods
- ✅ Real-time metrics visualization

---

## ��� Achievements

**Phase 2 Completion:**
- Complete monitoring stack deployed
- Enterprise-grade observability
- Foundation for DORA metrics
- Production-ready platform
- Full GitOps automation

**Next**: Phase 3 - Application deployment and DORA metrics dashboards

---

**Last Updated**: November 23, 2025  
**Status**: Phase 2 Complete ✅  
**Maintainer**: Platform Engineering Team
