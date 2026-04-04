# Design: kustomize-diff-test-cases

**Date:** 2026-04-03

## Overview

Create a Git + GitHub repository (`kustomize-diff-test-cases`) that demonstrates the `AdrienneCohea/kustomize-diff@v1` GitHub Action. The repo contains real Kustomize overlays (nginx + ConfigMap) and a workflow that runs the diff action on every push to a non-default branch.

## Repository Structure

```
kustomize-diff-test-cases/
├── .github/
│   └── workflows/
│       └── kustomize-diff.yml
└── .xyz/
    ├── base/
    │   ├── kustomization.yaml      # resources: [deployment.yaml, configmap.yaml]
    │   ├── deployment.yaml         # nginx Deployment, mounts ConfigMap at /usr/share/nginx/html
    │   └── configmap.yaml          # index.html with placeholder content
    └── environments/
        └── prod/
            ├── kustomization.yaml  # bases: [../../base], patchesStrategicMerge: [patch.yaml]
            └── patch.yaml          # sets replicas: 2
```

## Kustomize Overlays

### `.xyz/base`

- **Deployment**: `nginx` image, single replica, mounts the ConfigMap as a volume at `/usr/share/nginx/html`.
- **ConfigMap**: Contains `index.html` with simple placeholder HTML content.
- **kustomization.yaml**: Lists both resources.

### `.xyz/environments/prod`

- Inherits from `../../base`.
- Applies a strategic merge patch that sets `replicas: 2`.
- This gives the diff action a meaningful change to surface when comparing prod against base.

## GitHub Actions Workflow

**File:** `.github/workflows/kustomize-diff.yml`

**Trigger:** `push` to any branch except `main` (the default branch).

**Steps:**
1. `actions/checkout@v4` with `fetch-depth: 0` — required so the action can access the full git history and compare against the remote default branch.
2. `AdrienneCohea/kustomize-diff@v1` with `search-path: .xyz` — scopes overlay discovery to the `.xyz` directory.

## GitHub Repository

- Created under the user's GitHub account.
- Default branch: `main`.
- No branch protection rules required (this is a test/demo repo).

## Success Criteria

- Pushing a branch with a change to any file under `.xyz/` triggers the workflow.
- The workflow produces a human-readable diff report in the Actions UI, with collapsible `::group::` sections per overlay.
- A markdown summary table is written to `$GITHUB_STEP_SUMMARY`.
