# Harness GitOps Hydration Demo (OAM + Argo CD + Argo Rollouts)

This repository now contains an end-to-end **PR preview + hydration** demo that combines:

- **Harness CI/CD pipeline** for validation, hydration, and deployment.
- **OAM application definition** as the source-of-intent.
- **Hydrated Kubernetes manifests** generated per PR.
- **Argo CD ApplicationSet** for ephemeral PR environments.
- **Argo Rollouts** progressive delivery (canary + analysis).

## Repository layout

- `base/` - reusable Kubernetes base with Argo Rollout, services, and analysis template.
- `overlays/preprod` and `overlays/prod` - environment overlays via Kustomize.
- `manifests/oam/` - OAM app spec used as hydration source.
- `manifests/argocd/` - Argo CD ApplicationSet for PR previews.
- `harness/pipelines/` - Harness PR hydration pipeline YAML.
- `harness/services/` - Harness service definition.
- `harness/environments/` - Harness environment definition.

## What this demo implements

### 1) Progressive delivery by default

The app runs as an **Argo Rollout** with:

- Canary increments (20% → 50% → 100%).
- Pause windows to observe telemetry.
- Prometheus-based `AnalysisTemplate` gate.
- Hardened container security context and probes.

### 2) OAM as abstraction, hydration as runtime contract

The `manifests/oam/application-pr-preview.yaml` file is the desired intent model.
Harness pipeline step `Hydrate_OAM_to_Kubernetes` renders this into concrete manifests in `hydrated/` on each PR.

### 3) PR environment automation

`manifests/argocd/applicationset-pr-preview.yaml` creates a preview namespace per PR using Argo CD's Pull Request generator.

### 4) Harness PR pipeline controls

`harness/pipelines/pr-hydration-pipeline.yaml` includes:

- Manifest validation (`kustomize build` preprod + prod).
- OAM hydration (`vela dry-run`).
- Optional commit/push of hydrated branch.
- Deployment stage wired to Harness service and environment definitions.

## Harness + community best practices applied

- Git is the source of truth (declarative manifests).
- Immutable container tags passed at deploy time.
- Environment overlays separated from base templates.
- Progressive exposure + metric analysis for safer rollouts.
- Automated PR previews for fast feedback and lower risk.
- Minimal RBAC blast radius via environment namespaces.
- Explicit failure strategy with stage rollback.

## How to adapt quickly for a customer workshop

1. Replace placeholder values:
   - GitHub org/repo and connector refs in `harness/*` and `manifests/argocd/*`.
   - Kubernetes infra definition identifier in the pipeline.
2. Ensure CRDs are installed:
   - Argo Rollouts
   - Argo CD ApplicationSet
   - OAM runtime (KubeVela)
3. Wire metrics source:
   - Update Prometheus query labels in `base/analysis-template.yaml`.
4. Protect production:
   - Add Harness approvals before prod promotion.
   - Add policy checks (OPA/Conftest/SAST) in CI stage.

---

If you want, the next step is to add a **multi-service OAM Application** (frontend + API + worker) and promote with **progressive multi-service dependency waves**.
