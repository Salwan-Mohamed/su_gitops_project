<!-- Copilot / AI agent instructions for the SU GitOps repo -->
# Copilot Instructions — SU GitOps Platform

This file contains concise, actionable guidance for AI coding agents working in this repository. Focus on observable patterns and conventions so you can be immediately productive.

1) Big picture (what this repo is for)
- Purpose: single-repo GitOps platform for a Kubernetes cluster managed by ArgoCD. See `README.md` and `docs/architecture.md` for goals and phases.
- Top-level layout: `bootstrap/` (ArgoCD App-of-Apps), `infrastructure/` (kustomize bases & overlays), `platform-services/` (monitoring, grafana), `applications/` (team apps), `docs/`.

2) Key components and where to find them
- ArgoCD App-of-Apps entry: `bootstrap/root-app.yaml` — points to `bootstrap/apps` and uses `syncPolicy.automated.selfHeal=true` and `CreateNamespace=true`.
- Per-app ArgoCD applications: `bootstrap/apps/*.yaml` (e.g., `infrastructure.yaml`, `platform-services.yaml`, `monitoring.yaml`). Note: `infrastructure.yaml` enables `prune: true` and `ApplyOutOfSyncOnly=true`.
- Kustomize base for cluster infra: `infrastructure/base/kustomization.yaml` (namespaces, RBAC, network policies).
- Platform monitoring: `platform-services/base/monitoring/` and `platform-services/base/grafana/`.
- AppProject & environment definitions: `infrastructure/base/projects/` and `infrastructure/overlays/{dev,stage,production}`.

3) Repository & deployment conventions (explicit rules)
- Branch and revision: `main` is the canonical targetRevision used by ArgoCD apps (see `bootstrap/root-app.yaml`).
- Promotion model: environments are promoted by copying overlay manifests (dev → stage → production). See `docs/architecture.md` for the suggested `cp` + `git commit` workflow.
- Labels: resources use `managed-by: gitops` and optionally `component` labels. Kustomize uses `labels: pairs:` in `infrastructure/base/kustomization.yaml`.
- Safety defaults: `bootstrap/root-app.yaml` has `prune: false` (safety at root) while sub-apps (infrastructure) may have `prune: true` — be careful when changing prune settings.
- Secret handling: some ArgoCD apps ignore differences for Secret data (see `ignoreDifferences` in `bootstrap/apps/infrastructure.yaml`) — do not attempt to reconcile raw secret `data` fields.

4) ArgoCD sync options to respect
- `CreateNamespace=true` is used widely — don't remove it unless you intentionally change namespace creation flow.
- `ApplyOutOfSyncOnly=true` used on infra app to reduce noisy re-applies.
- Retry/backoff and limits are configured in root and child apps — prefer small, idempotent changes and respect retry semantics.

5) Runtime & infra expectations (developer prerequisites)
- Assumes an existing HA Kubernetes cluster and ArgoCD instance. Local development/testing: use `kubectl` pointed at a cluster with ArgoCD installed.
- Storage: default StorageClass is `nfs-client` in this project; Prometheus and AlertManager expect NFS-backed PVCs.
- MetalLB provides LoadBalancer IPs used in docs (`10.1.5.184` ArgoCD, `10.1.5.186` Grafana) — don't hardcode these addresses for different clusters.

6) Typical commands (copy-paste ready)
- Bootstrap ArgoCD App-of-Apps:
  - `kubectl apply -f bootstrap/root-app.yaml`
- Inspect ArgoCD apps:
  - `kubectl get applications -n argocd`
  - `kubectl describe application <app> -n argocd`
- Force sync (prefer ArgoCD UI or `argocd` CLI); minimal kubectl patch example is in `README.md`.
- Prometheus port-forward for ad-hoc queries:
  - `kubectl port-forward -n platform-monitoring prometheus-*-prometheus-0 9090:9090`

7) Patterns to follow when editing the repo
- Use the overlays model: change `infrastructure/overlays/dev` first; promote to stage/production by copying overlays and committing.
- Keep `repoURL` and `targetRevision` intact unless performing a global branch strategy change.
- When updating ArgoCD `Application` manifests, prefer minimal diffs: change `path` or `targetRevision` rather than toggling many sync options.
- When adding new platform components, place kustomize bases under `platform-services/base/` and add overlays for environments under `platform-services/overlays/`.

8) What to avoid / gotchas
- Do not remove `ignoreDifferences` rules for Secrets without a migration plan — ArgoCD will report spurious differences.
- Avoid enabling `prune: true` on root-level App without human review — the repo has `prune: false` at the root for safety.
- Do not hardcode cluster-specific IPs or credentials into manifests; use cluster-specific overlays or external secret management.

9) Places to look for more context (quick references)
- High-level overview: `README.md`
- Architecture & promotion: `docs/architecture.md`
- ArgoCD bootstrapping: `bootstrap/root-app.yaml`, `bootstrap/apps/*.yaml`
- Kustomize conventions: `infrastructure/base/kustomization.yaml`
- Monitoring: `platform-services/base/` (Prometheus, Grafana)
- CI ideas: `.github/` and `ci-cd/github-actions/` (GitHub Actions planned)

10) If you modify generated or operational manifests
- Run the minimal checks: `kubectl apply --server-dry-run=client -f <file>` and validate kustomize build: `kustomize build <dir>` or `kubectl kustomize <dir>`.

If anything here is unclear or you want me to expand a section (examples, more command snippets, or automated checks), tell me which area to iterate on next.
