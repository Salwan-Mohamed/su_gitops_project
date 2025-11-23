# Quick Reference Guide

Fast access to common commands and information.

---

## ��� Access URLs
```
ArgoCD UI:      http://10.1.5.184
Grafana UI:     http://10.1.5.186
GitHub Repo:    https://github.com/Salwan-Mohamed/su_gitops_project
```

---

## ��� Credentials
```
ArgoCD:
  Username: admin
  Password: <from secret: kubectl get secret argocd-initial-admin-secret -n argocd -o jsonpath='{.data.password}' | base64 -d>

Grafana:
  Username: admin
  Password: admin
```

---

## ��� Essential Commands

### Monitoring
```bash
# Check all monitoring pods
kubectl get pods -n platform-monitoring

# Access Grafana
open http://10.1.5.186

# Port-forward Prometheus
kubectl port-forward -n platform-monitoring prometheus-kube-prometheus-stack-prometheus-0 9090:9090

# Port-forward AlertManager
kubectl port-forward -n platform-monitoring alertmanager-kube-prometheus-stack-alertmanager-0 9093:9093
```

### ArgoCD
```bash
# Check all applications
kubectl get applications -n argocd

# Check specific app
kubectl get application <app-name> -n argocd

# Force sync
kubectl patch application <app-name> -n argocd --type merge -p '{"operation":{"sync":{}}}'

# Access ArgoCD UI
open http://10.1.5.184
```

### Troubleshooting
```bash
# Check pod logs
kubectl logs -n <namespace> <pod-name>

# Describe resource
kubectl describe <resource-type> <name> -n <namespace>

# Get events
kubectl get events -n <namespace> --sort-by='.lastTimestamp'

# Exec into pod
kubectl exec -it <pod-name> -n <namespace> -- /bin/sh
```

---

## ��� Useful Prometheus Queries
```promql
# All targets health
up

# Node memory
node_memory_MemAvailable_bytes

# Pod status
kube_pod_status_phase

# CPU usage
rate(node_cpu_seconds_total[5m])

# ArgoCD apps
argocd_app_info

# Pod restarts
kube_pod_container_status_restarts_total
```

---

## ���️ Project Structure
```
su_gitops_project/
├── bootstrap/
│   ├── root-app.yaml
│   └── apps/
├── infrastructure/base/
│   ├── projects/
│   └── namespaces/
├── platform-services/base/
│   ├── monitoring/
│   └── grafana/
├── applications/
└── docs/
```

---

## ��� Phase Status

- ✅ Phase 1: Foundation (Complete)
- ✅ Phase 2: Platform Services (Complete)
- ��� Phase 3: Applications & DORA Metrics (Next)

---

**Quick Links:**
- [Full README](../README.md)
- [Phase 2 Docs](./PHASE2-COMPLETE.md)
- [Troubleshooting](./TROUBLESHOOTING.md)
