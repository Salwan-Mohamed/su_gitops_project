# Grafana Dashboard Reference

**Access**: http://10.1.5.186  
**Credentials**: admin / admin  
**Status**: ‚úÖ Persistent storage configured - dashboards survive restarts

---

## Ì≥Å Dashboard Organization

### Ì≥ä Kubernetes Infrastructure
**Purpose**: Monitor cluster health and node performance

1. **Node Exporter Full (1860)**
   - CPU usage per node (all 7 nodes)
   - Memory usage and availability
   - Disk I/O and capacity
   - Network traffic
   - **Use Case**: Identify resource bottlenecks

2. **Kubernetes Cluster Monitoring (315)**
   - Overall cluster capacity
   - Pod counts across namespaces
   - Deployment health status
   - Resource requests vs limits
   - **Use Case**: Cluster-wide health overview

3. **Kubernetes Deployment Stats (8588)**
   - Deployment replica status
   - StatefulSet health
   - DaemonSet coverage
   - Rollout history
   - **Use Case**: Application deployment health

---

### ÌæØ DORA Metrics
**Purpose**: Track DevOps performance metrics

1. **ArgoCD DORA Metrics (13332)**
   - Deployment Frequency (deployments/day)
   - Change Failure Rate (%)
   - Application sync status
   - Deployment trends over time
   - **Use Case**: Measure DORA performance levels

**DORA Performance Levels:**
- ÌøÜ **Elite**: Deploy on demand (multiple/day), <1hr lead time, <1hr MTTR, <15% failure
- Ìµá **High**: Weekly-monthly deploys, <1 week lead time, <1 day MTTR, 16-30% failure
- Ìµà **Medium**: Monthly-every 6mo, 1-6mo lead time, 1 day-1 week MTTR, 31-45% failure
- Ìµâ **Low**: <Every 6mo, >6mo lead time, >1 week MTTR, >45% failure

**Current Status**: Elite level (0% failure rate, multiple deploys/day during setup)

---

## Ì¥ç Useful Prometheus Queries

### Application Health
```promql
# All ArgoCD applications and their sync status
argocd_app_info

# Only synced applications
argocd_app_info{sync_status="Synced"}

# Out of sync applications (needs attention)
argocd_app_info{sync_status="OutOfSync"}
```

### DORA Metrics

**Deployment Frequency (last 24 hours):**
```promql
sum(increase(argocd_app_sync_total{phase="Succeeded"}[24h]))
```

**Deployment Frequency (per hour rate):**
```promql
sum(rate(argocd_app_sync_total{phase="Succeeded"}[1h])) * 3600
```

**Change Failure Rate (%):**
```promql
(sum(increase(argocd_app_sync_total{phase="Failed"}[24h])) / 
 sum(increase(argocd_app_sync_total[24h])) * 100) or vector(0)
```

**Successful Deployments:**
```promql
sum(increase(argocd_app_sync_total{phase="Succeeded"}[24h]))
```

**Failed Deployments:**
```promql
sum(increase(argocd_app_sync_total{phase="Failed"}[24h]))
```

### Infrastructure Health

**Node Memory Available:**
```promql
node_memory_MemAvailable_bytes
```

**Node CPU Usage (%):**
```promql
100 - (avg by (instance) (rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100)
```

**Pod Restart Rate (indicator of issues):**
```promql
sum(rate(kube_pod_container_status_restarts_total[5m]))
```

**Pods by Phase:**
```promql
count(kube_pod_status_phase{phase="Running"}) by (namespace)
```

**Disk Usage Per Node:**
```promql
node_filesystem_avail_bytes{mountpoint="/"}
```

### Monitoring Stack Health

**Prometheus Targets Up:**
```promql
up
```

**Prometheus Scrape Duration:**
```promql
scrape_duration_seconds
```

**Number of Time Series:**
```promql
prometheus_tsdb_head_series
```

---

## Ì∫® Common Troubleshooting Queries

**Find Crashing Pods:**
```promql
rate(kube_pod_container_status_restarts_total[15m]) > 0
```

**High Memory Pods:**
```promql
container_memory_usage_bytes > 1000000000
```

**High CPU Pods:**
```promql
rate(container_cpu_usage_seconds_total[5m]) > 0.8
```

**Pods Not Ready:**
```promql
kube_pod_status_ready{condition="false"} == 1
```

---

## Ì≥ù Dashboard Maintenance

### Adding New Dashboards
1. Click **"+"** ‚Üí **Import**
2. Enter dashboard ID from grafana.com
3. Select appropriate folder
4. Choose **Prometheus** as datasource
5. Click **Import**

### Creating Custom Dashboards
1. Click **"+"** ‚Üí **Create Dashboard**
2. Add panel with PromQL query
3. Choose visualization type
4. Save to appropriate folder

### Backup Dashboards
Dashboards are automatically persisted on NFS volume:
- Location: `/var/lib/grafana` in pod
- PVC: `grafana-pvc` (10Gi)
- Survives pod restarts ‚úÖ

---

## ÌæØ Quick Links

- **Explore Metrics**: http://10.1.5.186/explore
- **All Dashboards**: http://10.1.5.186/dashboards
- **Prometheus Direct**: http://10.1.5.185:9090
- **AlertManager**: http://10.1.5.185:9093

---

**Last Updated**: December 1, 2025  
**Status**: ‚úÖ Production Ready with Persistent Storage
