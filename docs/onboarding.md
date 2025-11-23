# Team Onboarding Guide

## Platform Engineer Onboarding

### Prerequisites
- kubectl access to cluster
- GitHub account with repo access
- ArgoCD UI access
- VS Code with GitOps extensions

### Setup Steps

1. **Clone Repository**
```bash
git clone https://github.com/Salwan-Mohamed/su_gitops_project.git
cd su_gitops_project
```

2. **Configure kubectl**
```bash
# Verify cluster access
kubectl get nodes
kubectl get namespaces
```

3. **Configure ArgoCD**
```bash
# Install ArgoCD CLI
# For Linux/WSL
curl -sSL -o argocd-linux-amd64 https://github.com/argoproj/argo-cd/releases/latest/download/argocd-linux-amd64
sudo install -m 555 argocd-linux-amd64 /usr/local/bin/argocd

# Login
argocd login 10.1.5.184 --username admin --insecure
```

4. **Verify Access**
```bash
# Check ArgoCD applications
argocd app list

# Check cluster resources
kubectl get applications -n argocd
```

---

## Developer Onboarding (IS Team)

### Access Requirements
- Read access to GitOps repository
- Namespace access (dev/stage/production)
- ArgoCD UI access (view only initially)

### Application Deployment Workflow

1. **Create Application Structure**
```bash
applications/is-team/your-app/
├── base/
│   ├── deployment.yaml
│   ├── service.yaml
│   └── kustomization.yaml
└── overlays/
    ├── dev/
    ├── stage/
    └── production/
```

2. **Deploy to Dev**
```bash
# Create PR with your app config
git checkout -b feature/new-app
# Add your manifests
git add applications/is-team/your-app/
git commit -m "Add new application"
git push origin feature/new-app
```

3. **Promotion Process**
```bash
# After testing in dev
cp -r applications/is-team/your-app/overlays/dev/* applications/is-team/your-app/overlays/stage/
# Commit and create PR
```

---

## DevOps Engineer Onboarding

### CI/CD Pipeline Setup

1. **GitHub Actions Access**
- Secrets management
- Workflow permissions
- Runner configuration

2. **Pipeline Integration**
```yaml
# .github/workflows/deploy.yaml
name: Deploy Application
on:
  push:
    branches: [main]
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Update manifests
        run: |
          # Update image tags
          # Commit to GitOps repo
```

---

## System/Network/Helpdesk Teams

### Future Multi-Instance Setup

Each team will receive:
- Dedicated ArgoCD instance
- Team-specific namespaces
- RBAC policies
- Documentation and training

---

## Support Channels

- **Platform Team**: Slack #platform-engineering
- **Documentation**: This repository
- **Issues**: GitHub Issues
- **Emergency**: On-call rotation

