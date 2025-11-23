# Troubleshooting Guide

Common issues and solutions for the GitOps monitoring platform.

---

## Ì¥ç Monitoring Stack Issues

### Grafana Not Accessible

**Symptoms:**
- Cannot access http://10.1.5.186
- Connection timeout
- LoadBalancer pending

**Diagnosis:**
```bash
# Check Grafana pod status
kubectl get pods -n platform-monitoring -l app=grafana

# Check service
kubectl get svc -n platform-monitoring grafana

# Check events
kubectl describe svc -n platform-monitoring grafana
```

**Solutions:**

1. **Pod not running:**
```bash
kubectl logs -n platform-monitoring -l app=grafana
kubectl describe pod -n platform-monitoring -l app=grafana
```

2. **LoadBalancer pending:**
```bash
# Check MetalLB is running
kubectl get pods -n metallb-system

# Check IP pool configuration
kubectl get ipaddresspools -n metallb-system
```

3. **Port-forward as workaround:**
```bash
kubectl port-forward -n platform-monitoring svc/grafana 3000:80
# Access: http://localhost:3000
```

---

### Prometheus Not Collecting Metrics

**Symptoms:**
- No data in Grafana queries
- Targets showing as down
- Missing metrics

**Diagnosis:**
```bash
# Check Prometheus pod
kubectl get pods -n platform-monitoring prometheus-kube-prometheus-stack-prometheus-0

# Port-forward and check targets
kubectl port-forward -n platform-monitoring \
  prometheus-kube-prometheus-stack-prometheus-0 9090:9090
# Open: http://localhost:9090/targets
```

**Solutions:**

1. **Check Prometheus config:**
```bash
kubectl logs -n platform-monitoring \
  prometheus-kube-prometheus-stack-prometheus-0 -c prometheus
```

2. **Verify ServiceMonitors:**
```bash
kubectl get servicemonitors -n platform-monitoring
```

3. **Check network policies:**
```bash
kubectl get networkpolicies -n platform-monitoring
```

---

### PVC Stuck in Pending

**Symptoms:**
- Prometheus/AlertManager pods stuck in Pending
- PVC status shows Pending
- No volume available

**Diagnosis:**
```bash
# Check PVC status
kubectl get pvc -n platform-monitoring

# Check events
kubectl describe pvc <pvc-name> -n platform-monitoring

# Check StorageClass
kubectl get sc
```

**Solutions:**

1. **Set default StorageClass:**
```bash
kubectl patch storageclass nfs-client \
  -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}'
```

2. **Check provisioner is running:**
```bash
# For NFS
kubectl get pods -n nfs-provisioner

# For OpenEBS
kubectl get pods -n openebs
```

3. **Manually create PV (if needed):**
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: prometheus-pv
spec:
  capacity:
    storage: 20Gi
  accessModes:
    - ReadWriteOnce
  storageClassName: nfs-client
  nfs:
    server: <nfs-server>
    path: /path/to/storage
```

---

## Ì¥Ñ ArgoCD Sync Issues

### Application Stuck OutOfSync

**Symptoms:**
- Application shows "OutOfSync"
- Changes not applying
- Sync fails

**Diagnosis:**
```bash
# Check application status
kubectl get application <app-name> -n argocd

# Get detailed status
kubectl describe application <app-name> -n argocd

# Check ArgoCD logs
kubectl logs -n argocd -l app.kubernetes.io/name=argocd-application-controller
```

**Solutions:**

1. **Force hard refresh:**
```bash
kubectl patch application <app-name> -n argocd \
  --type merge \
  -p '{"operation":{"initiatedBy":{"username":"admin"},"sync":{"revision":"HEAD","prune":true}}}'
```

2. **Delete and let root-app recreate:**
```bash
kubectl delete application <app-name> -n argocd
# Wait 30 seconds for root-app to recreate
```

3. **Check Git repository access:**
```bash
# Test from ArgoCD repo-server pod
kubectl exec -it -n argocd <repo-server-pod> -- git ls-remote <repo-url>
```

---

### Application Degraded/Progressing

**Symptoms:**
- Health status shows "Degraded" or "Progressing"
- Some resources not healthy
- Sync completed but issues remain

**Diagnosis:**
```bash
# Check which resources are unhealthy
kubectl get application <app-name> -n argocd -o yaml | grep -A 10 status

# Check resource events
kubectl get events -n <namespace> --sort-by='.lastTimestamp'
```

**Solutions:**

1. **Identify unhealthy resource:**
```bash
# Get application resource tree
argocd app get <app-name> --show-operation

# Check specific resource
kubectl describe <resource-type> <resource-name> -n <namespace>
```

2. **Common fixes:**
```bash
# For stuck deployments
kubectl rollout status deployment/<name> -n <namespace>
kubectl rollout restart deployment/<name> -n <namespace>

# For failed jobs
kubectl delete job <job-name> -n <namespace>
```

---

## Ì¥í Pod Issues

### CrashLoopBackOff

**Symptoms:**
- Pod restarting continuously
- Status shows CrashLoopBackOff
- Container exits immediately

**Diagnosis:**
```bash
# Check pod status
kubectl get pod <pod-name> -n <namespace>

# View logs
kubectl logs <pod-name> -n <namespace>
kubectl logs <pod-name> -n <namespace> --previous

# Check events
kubectl describe pod <pod-name> -n <namespace>
```

**Common Causes & Solutions:**

1. **Configuration error:**
```bash
# Check ConfigMaps/Secrets
kubectl get configmap -n <namespace>
kubectl describe configmap <name> -n <namespace>
```

2. **Resource limits:**
```bash
# Check resource requests/limits
kubectl get pod <pod-name> -n <namespace> -o yaml | grep -A 5 resources
```

3. **Volume mount issues:**
```bash
# Check volume mounts
kubectl describe pod <pod-name> -n <namespace> | grep -A 10 Volumes
```

---

### Pod Stuck in Pending

**Symptoms:**
- Pod never starts
- Status remains Pending
- No node assigned

**Diagnosis:**
```bash
# Check why pod is pending
kubectl describe pod <pod-name> -n <namespace> | grep -A 10 Events
```

**Common Causes & Solutions:**

1. **Insufficient resources:**
```bash
# Check node resources
kubectl top nodes
kubectl describe nodes
```

2. **Node affinity/taints:**
```bash
# Check node labels
kubectl get nodes --show-labels

# Check taints
kubectl describe node <node-name> | grep Taints
```

3. **PVC not bound:**
```bash
# Check PVCs
kubectl get pvc -n <namespace>
```

---

## Ìºê Network Issues

### Service Not Accessible

**Symptoms:**
- Cannot reach service
- Connection refused
- Timeouts

**Diagnosis:**
```bash
# Check service
kubectl get svc <service-name> -n <namespace>

# Check endpoints
kubectl get endpoints <service-name> -n <namespace>

# Test from within cluster
kubectl run test-pod --rm -it --image=curlimages/curl -- \
  curl http://<service-name>.<namespace>.svc.cluster.local:<port>
```

**Solutions:**

1. **Service selector mismatch:**
```bash
# Compare service selector with pod labels
kubectl get svc <service-name> -n <namespace> -o yaml | grep selector
kubectl get pods -n <namespace> --show-labels
```

2. **Port misconfiguration:**
```bash
# Check service ports
kubectl describe svc <service-name> -n <namespace>

# Check container ports
kubectl get pod <pod-name> -n <namespace> -o yaml | grep containerPort
```

---

### LoadBalancer IP Not Assigned

**Symptoms:**
- LoadBalancer shows `<pending>`
- External-IP never gets assigned
- Cannot access from outside cluster

**Diagnosis:**
```bash
# Check service
kubectl get svc <service-name> -n <namespace>

# Check MetalLB
kubectl get pods -n metallb-system
kubectl logs -n metallb-system -l app=metallb
```

**Solutions:**

1. **MetalLB not running:**
```bash
kubectl get pods -n metallb-system
# If not running, reinstall MetalLB
```

2. **IP pool exhausted:**
```bash
# Check IP pool
kubectl get ipaddresspools -n metallb-system

# Check allocated IPs
kubectl get svc --all-namespaces | grep LoadBalancer
```

3. **Worker04 metallb disabled:**
```bash
# Verify service is not scheduled on worker04
kubectl get pods -n <namespace> -o wide
```

---

## Ì≥ä Grafana Issues

### Datasource Not Working

**Symptoms:**
- "Data source not found" error
- Queries return no data
- Connection test fails

**Diagnosis:**
```bash
# Check Prometheus service
kubectl get svc -n platform-monitoring kube-prometheus-stack-prometheus

# Test connection from Grafana pod
kubectl exec -it -n platform-monitoring <grafana-pod> -- \
  curl http://kube-prometheus-stack-prometheus.platform-monitoring.svc.cluster.local:9090
```

**Solutions:**

1. **Update datasource URL:**
   - Go to Configuration ‚Üí Data Sources
   - Click Prometheus
   - Update URL to: `http://kube-prometheus-stack-prometheus.platform-monitoring.svc.cluster.local:9090`
   - Click "Save & Test"

2. **Check ConfigMap:**
```bash
kubectl get configmap grafana-datasources -n platform-monitoring -o yaml
```

---

### Password Change Fails

**Symptoms:**
- Cannot change admin password
- "Network error" when saving
- Reverts to old password

**Cause:** Grafana has no persistence (no PVC)

**Workaround:**
- Use admin/admin for now
- Phase 3: Add persistence with PVC

---

## Ì¥ê Storage Issues

### NFS Mount Failures

**Symptoms:**
- Pods can't mount volumes
- "Mount failed" errors
- PVC bound but pod pending

**Diagnosis:**
```bash
# Check PVC
kubectl describe pvc <pvc-name> -n <namespace>

# Check pod events
kubectl describe pod <pod-name> -n <namespace> | grep -A 10 Events

# Check NFS provisioner
kubectl get pods -n nfs-provisioner
kubectl logs -n nfs-provisioner <provisioner-pod>
```

**Solutions:**

1. **NFS server accessibility:**
```bash
# From any node
showmount -e <nfs-server-ip>
mount -t nfs <nfs-server>:/export /mnt/test
```

2. **Check PV:**
```bash
kubectl get pv
kubectl describe pv <pv-name>
```

---

## Ì≥ã Useful Debug Commands

### General Debugging
```bash
# Get all resources in namespace
kubectl get all -n <namespace>

# Watch resources
kubectl get pods -n <namespace> -w

# Get events sorted by time
kubectl get events -n <namespace> --sort-by='.lastTimestamp'

# Get resource YAML
kubectl get <resource-type> <name> -n <namespace> -o yaml

# Exec into pod
kubectl exec -it <pod-name> -n <namespace> -- /bin/sh
```

### Monitoring Specific
```bash
# Check Prometheus targets
kubectl port-forward -n platform-monitoring \
  prometheus-kube-prometheus-stack-prometheus-0 9090:9090
# Open: http://localhost:9090/targets

# Check AlertManager status
kubectl port-forward -n platform-monitoring \
  alertmanager-kube-prometheus-stack-alertmanager-0 9093:9093
# Open: http://localhost:9093

# View Grafana config
kubectl get configmap -n platform-monitoring grafana-datasources -o yaml
```

---

## Ì∂ò Emergency Procedures

### Complete Monitoring Stack Restart
```bash
# Delete all monitoring pods
kubectl delete pods -n platform-monitoring --all

# ArgoCD will recreate them automatically
# Wait 2-3 minutes and verify
kubectl get pods -n platform-monitoring
```

### Reset Grafana
```bash
# Delete Grafana pod
kubectl delete pod -n platform-monitoring -l app=grafana

# Wait for recreation
kubectl get pods -n platform-monitoring -l app=grafana -w
```

### Reset ArgoCD Application
```bash
# Delete application
kubectl delete application <app-name> -n argocd

# root-app will recreate it
# Wait 30 seconds
kubectl get application <app-name> -n argocd
```

---

## Ì≥û Getting Help

**Check Logs:**
```bash
# ArgoCD
kubectl logs -n argocd -l app.kubernetes.io/name=argocd-application-controller

# Prometheus
kubectl logs -n platform-monitoring prometheus-kube-prometheus-stack-prometheus-0 -c prometheus

# Grafana
kubectl logs -n platform-monitoring -l app=grafana
```

**Documentation:**
- [Prometheus Operator Docs](https://prometheus-operator.dev/docs/)
- [ArgoCD Troubleshooting](https://argo-cd.readthedocs.io/en/stable/user-guide/troubleshooting/)
- [Kubernetes Debugging](https://kubernetes.io/docs/tasks/debug/)

---

**Last Updated**: November 23, 2025
