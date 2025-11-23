# Troubleshooting Guide

## Common Issues

### ArgoCD Application Not Syncing

**Symptoms:**
- Application stuck in "OutOfSync"
- Sync operation fails

**Diagnosis:**
```bash
# Check application status
argocd app get <app-name>

# Check sync status
argocd app sync <app-name> --dry-run

# View application logs
kubectl logs -n argocd deployment/argocd-application-controller
```

**Solutions:**
1. Check YAML syntax errors
2. Verify RBAC permissions
3. Check resource quotas
4. Review application events

---

### Image Pull Errors

**Symptoms:**
- Pods in ImagePullBackOff state

**Diagnosis:**
```bash
kubectl describe pod <pod-name> -n <namespace>
kubectl get events -n <namespace> --sort-by='.lastTimestamp'
```

**Solutions:**
1. Verify image registry credentials
2. Check image tag exists
3. Review network policies
4. Verify registry access

---

### Out of Sync Status

**Symptoms:**
- ArgoCD shows "OutOfSync" but resources match

**Solutions:**
```bash
# Force refresh
argocd app sync <app-name> --force

# Check for ignored differences
argocd app diff <app-name>
```

---

### Promotion Issues

**Problem:** Changes not appearing after promotion

**Solution:**
```bash
# Verify files copied correctly
diff infrastructure/overlays/dev/app.yaml infrastructure/overlays/stage/app.yaml

# Check Git commit/push
git log --oneline -n 5
git status

# Trigger ArgoCD refresh
argocd app sync <app-name>
```

---

## Emergency Procedures

### Rollback Application
```bash
# Via ArgoCD
argocd app rollback <app-name> <revision>

# Via Git
git revert <commit-hash>
git push
```

### Cluster Recovery

1. Check cluster health
2. Review etcd status
3. Verify control plane
4. Check worker nodes
5. Review GitOps repo as source of truth

---

## Getting Help

1. Check this documentation
2. Review GitHub issues
3. Contact Platform Team
4. Create incident ticket

