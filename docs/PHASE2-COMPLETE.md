# ‚úÖ Phase 2: Platform Services - COMPLETE

**Date Completed**: November 23, 2025  
**Status**: ‚úÖ All objectives achieved

## ÌæØ Objectives

Deploy a complete, production-ready monitoring stack using GitOps, enabling observability across the entire Kubernetes cluster and establishing the foundation for DORA metrics.

---

## Ìæâ Accomplishments

### Complete Monitoring Stack Deployed

**All Components Operational:**

#### 1. **Prometheus** - Metrics Collection & Storage
```yaml
Status:           ‚úÖ Running (2/2 pods)
Pod:              prometheus-kube-prometheus-stack-prometheus-0
Storage:          20Gi NFS (nfs-client StorageClass)
Retention:        15 days
Scrape Interval:  30 seconds
Targets:          73 endpoints monitored
Access:           Port-forward to localhost:9090
```

**Features:**
- Persistent storage for metrics history
- Automatic target discovery via ServiceMonitors
- ArgoCD metrics collection configured
- Node and pod metrics from entire cluster

#### 2. **Grafana** - Visualization & Dashboards
```yaml
Status:           ‚úÖ Running (1/1 pod)
Pod:              grafana-fdf4b5f6-lcxxw
LoadBalancer:     http://10.1.5.186
Credentials:      admin / admin
Datasource:       Prometheus (auto-configured)
Deployment Type:  Standalone (no Helm complexity)
```

**Features:**
- Real-time metrics visualization
- Interactive query builder
- Pre-configured Prometheus datasource
- Ready for dashboard import

**Verified Working:**
- ‚úÖ Query `up` ‚Üí 73 healthy targets
- ‚úÖ Node memory metrics displaying
- ‚úÖ ArgoCD application metrics visible
- ‚úÖ Real-time graph updates

#### 3. **AlertManager** - Alert Management & Routing
```yaml
Status:     ‚úÖ Running (2/2 pods)
Pod:        alertmanager-kube-prometheus-stack-alertmanager-0
Storage:    5Gi NFS (nfs-client StorageClass)
Retention:  120 hours
Access:     Port-forward to localhost:9093
```

**Ready for:**
- Slack notifications
- Email alerts
- PagerDuty integration
- Custom webhook receivers

#### 4. **Node Exporter** - Node-Level Metrics
```yaml
Status:       ‚úÖ Running (7/7 DaemonSet pods)
Coverage:     ALL cluster nodes
Deployment:   DaemonSet
Nodes:        3 masters + 4 workers (including worker04)
```

**Metrics Collected:**
- CPU usage per core
- Memory usage and availability
- Disk I/O and space
- Network traffic
- System load averages

#### 5. **Kube State Metrics** - Kubernetes Object Metrics
```yaml
Status:     ‚úÖ Running (1/1 pod)
Pod:        kube-prometheus-stack-kube-state-metrics
Metrics:    Kubernetes objects state
```

**Metrics Collected:**
- Pod status and phases
- Deployment replica status
- Service endpoints
- ConfigMap and Secret counts
- Resource requests and limits

#### 6. **Prometheus Operator** - Custom Resource Management
```yaml
Status:     ‚úÖ Running (1/1 pod)
Pod:        kube-prometheus-stack-operator
Manages:    ServiceMonitors, PodMonitors, PrometheusRules
```

**Capabilities:**
- Dynamic scrape configuration
- Alert rule management
- Automatic configuration updates

---

## ÌøóÔ∏è Architecture

### Deployment Strategy

**Two-Component Approach:**
1. **kube-prometheus-stack (Helm)**: Core monitoring components
   - Prometheus
   - AlertManager
   - Node Exporter
   - Kube State Metrics
   - Prometheus Operator

2. **Grafana (Standalone)**: Visualization layer
   - Simple Kubernetes Deployment
   - No Helm subchart complexity
   - Single container (no sidecars)
   - Manual datasource configuration

### GitOps Structure
```
bootstrap/apps/
‚îú‚îÄ‚îÄ monitoring.yaml          # Helm: kube-prometheus-stack
‚îÇ                           # (Grafana disabled in chart)
‚îî‚îÄ‚îÄ grafana.yaml            # Standalone Grafana deployment

platform-services/base/
‚îú‚îÄ‚îÄ monitoring/
‚îÇ   ‚îú‚îÄ‚îÄ namespace.yaml      # platform-monitoring namespace
‚îÇ   ‚îî‚îÄ‚îÄ kustomization.yaml
‚îî‚îÄ‚îÄ grafana/
    ‚îú‚îÄ‚îÄ deployment.yaml     # Grafana + ConfigMap + Service
    ‚îî‚îÄ‚îÄ kustomization.yaml
```

### Storage Configuration

**StorageClass**: `nfs-client` (set as default)
```bash
$ kubectl get sc
NAME                   PROVISIONER                                   
nfs-client (default)   k8s-sigs.io/nfs-subdir-external-provisioner
generic-storage        openebs.io/local
openebs-hostpath       openebs.io/local
```

**Persistent Volume Claims:**
```bash
$ kubectl get pvc -n platform-monitoring
NAME                                          STATUS   VOLUME      CAPACITY
prometheus-...-db-prometheus-...-0            Bound    pvc-be1a..  20Gi
alertmanager-...-db-alertmanager-...-0        Bound    pvc-f6e4..  5Gi
```

### Node Placement Strategy

**Worker Nodes 01-03**: Run all stateful workloads
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

**Worker04**: Excluded but still monitored
- **Taints**: 
  - `storage=hdd:NoSchedule`
  - `metallb=disabled:NoSchedule`
- **Purpose**: Dedicated for backups (HDD storage, out of subnet)
- **Exception**: Node Exporter runs here for complete metrics

---

## Ì≥ä Metrics Collection

### Configured Targets (73 Total)

**Kubernetes Cluster Metrics:**
- All pods across all namespaces
- All deployments and replica sets
- All services and endpoints
- All nodes (7 total)

**ArgoCD GitOps Metrics:**
```yaml
- job: argocd-metrics
  target: argocd-metrics.argocd.svc.cluster.local:8082
  
- job: argocd-server-metrics
  target: argocd-server-metrics.argocd.svc.cluster.local:8083
  
- job: argocd-repo-server-metrics
  target: argocd-repo-server.argocd.svc.cluster.local:8084
```

**Infrastructure Metrics:**
- Node Exporter: 7 instances (one per node)
- Kube State Metrics: Kubernetes object states
- Prometheus self-monitoring
- AlertManager metrics

### Sample Queries
```promql
# All targets health (73 targets)
up

# Node memory available
node_memory_MemAvailable_bytes

# Pod status across cluster
kube_pod_status_phase

# CPU usage per node
rate(node_cpu_seconds_total[5m])

# ArgoCD application sync status
argocd_app_info

# Deployment replicas
kube_deployment_status_replicas

# Container memory usage
container_memory_usage_bytes

# Network traffic
rate(node_network_receive_bytes_total[5m])
```

---

## ÌæØ Access Information

### Grafana Web UI
```
URL:      http://10.1.5.186
Username: admin
Password: admin

Note: Password change fails due to no persistence (non-critical)
```

**First Login Steps:**
1. Access http://10.1.5.186 in browser
2. Login with admin/admin
3. Navigate to **Explore** (left menu, compass icon)
4. Select "Prometheus" datasource (default)
5. Run query: `up`
6. Verify 73 targets visible

### Prometheus Web UI
```bash
# Port-forward to access
kubectl port-forward -n platform-monitoring \
  prometheus-kube-prometheus-stack-prometheus-0 9090:9090

# Then open: http://localhost:9090
```

**Useful Pages:**
- `/targets` - View all scrape targets and health
- `/graph` - Query and visualize metrics
- `/alerts` - View active alerts
- `/config` - View Prometheus configuration

### AlertManager Web UI
```bash
# Port-forward to access
kubectl port-forward -n platform-monitoring \
  alertmanager-kube-prometheus-stack-alertmanager-0 9093:9093

# Then open: http://localhost:9093
```

---

## ‚úÖ Verification & Testing

### Health Checks
```bash
# Check all monitoring pods
kubectl get pods -n platform-monitoring

# Expected output:
# NAME                                                        READY   STATUS
# alertmanager-kube-prometheus-stack-alertmanager-0           2/2     Running
# grafana-fdf4b5f6-lcxxw                                      1/1     Running
# kube-prometheus-stack-kube-state-metrics-5f454f8889-cm4pb   1/1     Running
# kube-prometheus-stack-operator-7c8c489f47-zhcbh             1/1     Running
# kube-prometheus-stack-prometheus-node-exporter-* (7 pods)   1/1     Running
# prometheus-kube-prometheus-stack-prometheus-0               2/2     Running
```

### ArgoCD Application Status
```bash
kubectl get applications -n argocd

# Expected output:
# NAME                    SYNC STATUS   HEALTH STATUS
# root-app                Synced        Healthy
# infrastructure          Synced        Healthy
# platform-services       Synced        Healthy
# kube-prometheus-stack   Synced        Healthy
# grafana                 Synced        Healthy
```

### Storage Verification
```bash
# Check PVCs are bound
kubectl get pvc -n platform-monitoring

# Check StorageClass
kubectl get sc nfs-client
```

### Metrics Verification
**In Grafana Explore:**
1. Query: `up` ‚Üí Should return 73 series with value=1
2. Query: `node_memory_MemAvailable_bytes` ‚Üí Should show memory from 7 nodes
3. Query: `argocd_app_info` ‚Üí Should show ArgoCD applications

---

## Ìæì Technical Challenges & Solutions

### Challenge 1: Grafana Sidecar Container Crashes
**Problem**: 
- Helm chart includes sidecar containers (grafana-sc-dashboard, grafana-sc-datasources)
- Sidecars tried to connect to Grafana before it was ready
- Resulted in CrashLoopBackOff

**Solution**:
- Deployed Grafana as standalone Deployment
- Disabled Grafana in kube-prometheus-stack Helm chart
- Manual datasource configuration via ConfigMap
- Single container, no sidecars

**Result**: Stable, simple Grafana deployment

### Challenge 2: PVC Stuck in Pending
**Problem**:
- PVCs wouldn't bind to volumes
- No default StorageClass set
- Multiple StorageClasses available

**Solution**:
```bash
# Set nfs-client as default
kubectl patch storageclass nfs-client \
  -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}'
```

**Result**: Automatic PVC binding

### Challenge 3: Worker04 Node Placement
**Problem**:
- worker04 has HDD storage (unsuitable for monitoring workloads)
- worker04 out of cluster IP subnet range
- worker04 has metallb disabled

**Solution**:
- Added node affinity to exclude worker04 from workloads
- Node Exporter still runs on worker04 (DaemonSet, no affinity)
- Complete cluster metrics coverage maintained

**Result**: Optimal workload distribution

### Challenge 4: ArgoCD Sync Issues
**Problem**:
- Application stuck in "OutOfSync/Degraded"
- Helm chart changes not applying
- Old pods not being replaced

**Solution**:
- Delete and recreate application
- Force sync with prune option
- Separate Grafana from Helm chart
- Simplified deployment model

**Result**: All applications Synced/Healthy

---

## Ì≥à DORA Metrics Foundation

### Ready for Implementation

**1. Deployment Frequency**
- Data Source: ArgoCD sync events
- Metric: `argocd_app_sync_total`
- Calculation: Syncs per day/week/month

**2. Lead Time for Changes**
- Data Source: Git commits + ArgoCD sync time
- Metrics: Git commit timestamp ‚Üí ArgoCD sync timestamp
- Calculation: Time from commit to deployment

**3. Mean Time to Recovery (MTTR)**
- Data Source: Pod restart events
- Metrics: `kube_pod_container_status_restarts_total`
- Calculation: Time from failure to recovery

**4. Change Failure Rate**
- Data Source: Deployment success/failure
- Metrics: `argocd_app_sync_status`
- Calculation: Failed deployments / Total deployments

### Grafana Dashboard Plan (Phase 3)
- Import Kubernetes cluster dashboards
- Create custom DORA metrics dashboard
- Set up alerts for key metrics
- Configure team-specific views

---

## Ì¥ß Configuration Details

### Helm Values (kube-prometheus-stack)
```yaml
prometheus:
  prometheusSpec:
    retention: 15d
    storageSpec:
      volumeClaimTemplate:
        spec:
          storageClassName: nfs-client
          resources:
            requests:
              storage: 20Gi
    scrapeInterval: 30s
    evaluationInterval: 30s

alertmanager:
  enabled: true
  alertmanagerSpec:
    retention: 120h
    storage:
      volumeClaimTemplate:
        spec:
          storageClassName: nfs-client
          resources:
            requests:
              storage: 5Gi

grafana:
  enabled: false  # Deployed separately

nodeExporter:
  enabled: true

kubeStateMetrics:
  enabled: true
```

### Grafana Datasource Configuration
```yaml
apiVersion: 1
datasources:
- name: Prometheus
  type: prometheus
  access: proxy
  url: http://kube-prometheus-stack-prometheus.platform-monitoring.svc.cluster.local:9090
  isDefault: true
  editable: true
  jsonData:
    timeInterval: 30s
```

---

## Ì≥ä Success Metrics

### Quantitative
- ‚úÖ **7 nodes** fully monitored
- ‚úÖ **73 targets** healthy and scraped
- ‚úÖ **12 pods** running (monitoring components)
- ‚úÖ **25Gi storage** provisioned (20Gi + 5Gi)
- ‚úÖ **2 LoadBalancers** allocated
- ‚úÖ **0 failing pods** (100% healthy)
- ‚úÖ **30 second** scrape interval
- ‚úÖ **15 days** metrics retention

### Qualitative
- ‚úÖ GitOps workflow operational
- ‚úÖ Real-time metrics visualization
- ‚úÖ Complete cluster visibility
- ‚úÖ Foundation for DORA metrics
- ‚úÖ Production-ready monitoring
- ‚úÖ Scalable architecture

---

## Ì∫Ä Next Steps (Phase 3)

### Immediate Tasks
1. **Import Kubernetes Dashboards**
   - Node Exporter dashboard
   - Kubernetes cluster overview
   - Pod resources dashboard

2. **Create DORA Metrics Dashboard**
   - Deployment frequency graph
   - Lead time for changes
   - MTTR tracking
   - Change failure rate

3. **Configure AlertManager**
   - Slack webhook integration
   - Email notifications
   - Alert routing rules
   - Severity levels

4. **Deploy First Application**
   - IS team sample app
   - Expose metrics endpoint
   - Configure ServiceMonitor
   - Verify metrics collection

### Future Enhancements
- Add Grafana persistence (PVC)
- High availability Prometheus (2+ replicas)
- Long-term metrics storage (Thanos)
- Log aggregation (Loki)
- Distributed tracing (Tempo)
- Cost monitoring

---

## Ì≥ù Lessons Learned

### What Worked Well
1. **Separate Grafana deployment**: Simpler than Helm subchart
2. **NFS storage**: Reliable for multi-node access
3. **Node affinity**: Effective for heterogeneous clusters
4. **GitOps approach**: All changes tracked and auditable
5. **Iterative troubleshooting**: Systematic problem-solving

### Best Practices Applied
1. Deploy core components (Prometheus) before UI (Grafana)
2. Test each component independently
3. Use labels consistently for organization
4. Document decisions as you go
5. Keep configurations simple and maintainable
6. Verify at each step before proceeding

### Avoid in Future
1. Complex Helm subcharts with many dependencies
2. Assuming default StorageClass exists
3. Large monolithic deployments
4. Insufficient resource requests/limits
5. Manual configuration changes (always use Git)

---

## Ì≥ö Resources & References

**Official Documentation:**
- [Prometheus Operator](https://prometheus-operator.dev/)
- [Grafana Documentation](https://grafana.com/docs/)
- [kube-prometheus-stack Helm Chart](https://github.com/prometheus-community/helm-charts/tree/main/charts/kube-prometheus-stack)

**Internal Documentation:**
- [docs/PHASE1-COMPLETE.md](./PHASE1-COMPLETE.md)
- [docs/SESSION-SUMMARY.md](./SESSION-SUMMARY.md)
- [README.md](../README.md)

---

## Ìæâ Conclusion

Phase 2 successfully deployed a complete, production-ready monitoring stack using GitOps best practices. All 73 metrics targets are healthy, Grafana is accessible and functional, and the foundation for DORA metrics tracking is in place.

The platform is now ready for application deployments and advanced observability features in Phase 3.

---

**Status**: ‚úÖ Phase 2 Complete  
**Next Phase**: Application Deployment & DORA Metrics Dashboards  
**Documentation Updated**: November 23, 2025  
**Verified By**: Platform Engineering Team
