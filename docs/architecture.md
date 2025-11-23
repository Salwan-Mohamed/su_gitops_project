# GitOps Platform Architecture

## Architecture Overview

### Current State
- Single Kubernetes HA cluster
- Single ArgoCD instance (transitioning to multi-instance)
- Folder-based repository structure
- Environment promotion: dev → stage → production

### ArgoCD Strategy

**Phase 1: Single ArgoCD (Current)**
- One ArgoCD instance managing all environments
- RBAC for team separation
- Used for platform stabilization

**Phase 2: Cockpit & Fleet (Future Production)**
- Central ArgoCD (Cockpit): Platform team manages infrastructure
- Team ArgoCD instances (Fleet): Each team gets dedicated instance
- Better isolation and autonomy

### Repository Strategy

**Folder-based Monorepo**
```
infrastructure/
├── base/           # Common configurations
└── overlays/
    ├── dev/        # Development
    ├── stage/      # Staging  
    └── production/ # Production
```

**Advantages:**
- Easy promotion between environments (cp command)
- Simple diff between stages
- Single source of truth
- Git-based traceability

### Environment Promotion
```
Dev → Stage → Production

# Promote from dev to stage
cp infrastructure/overlays/dev/app.yaml infrastructure/overlays/stage/app.yaml
git add . && git commit -m "Promote app to stage"
git push
```

### Network Architecture

- **Load Balancer**: MetalLB (10.1.5.184 for ArgoCD)
- **CNI**: [To be documented]
- **Storage**: OpenEBS + NFS Provisioner
- **Ingress**: [To be implemented]

### Security Architecture

- **RBAC**: Kubernetes native RBAC
- **Network Policies**: Zero-trust segmentation
- **Secrets**: Sealed Secrets (planned)
- **Image Security**: Scanning in CI/CD (planned)
- **Policy Enforcement**: OPA/Kyverno (planned)

### Monitoring & Observability

**Planned Stack:**
- Prometheus: Metrics collection
- Grafana: Visualization & dashboards
- Loki: Log aggregation
- AlertManager: Alert routing

**DORA Metrics:**
- Deployment frequency tracking
- Lead time measurement
- MTTR calculation
- Change failure rate monitoring

---

## Infrastructure Components

### Core Infrastructure
- Namespaces (per environment)
- RBAC policies
- Network policies
- Resource quotas
- Pod security policies

### Platform Services
- Monitoring (Prometheus/Grafana)
- Logging (Loki)
- Ingress controller
- Certificate manager (cert-manager)
- Secret management
- Service mesh (future)

### Team Workloads
- IS Team: Business applications
- DevOps Team: CI/CD tools
- System Team: VDI/DC management
- Network Team: Automation tools
- Helpdesk Team: Support systems

---

## Disaster Recovery

**Backup Strategy:**
- etcd snapshots (automated)
- GitOps repo as source of truth
- Persistent volume backups
- Regular restore testing

**RTO/RPO:**
- RTO: < 4 hours
- RPO: < 15 minutes

---

## Scalability Considerations

- Horizontal Pod Autoscaling (HPA)
- Vertical Pod Autoscaling (VPA)
- Cluster Autoscaling (future)
- Multi-cluster federation (future)

---

**Last Updated**: 2024-11
**Document Owner**: Platform Engineering Team
