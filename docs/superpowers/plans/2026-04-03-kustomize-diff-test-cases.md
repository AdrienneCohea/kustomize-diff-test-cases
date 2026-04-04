# kustomize-diff-test-cases Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Set up a Git + GitHub repo with Kustomize overlays and a workflow that runs `AdrienneCohea/kustomize-diff@v1` on every push to a non-default branch.

**Architecture:** A base overlay defines a single-replica nginx Deployment and a ConfigMap serving `index.html`. A prod overlay inherits from base and patches replicas to 2. A GitHub Actions workflow triggers on push to any branch except `main` and runs the kustomize-diff action scoped to `.xyz/`.

**Tech Stack:** Kustomize, GitHub Actions, `AdrienneCohea/kustomize-diff@v1`

---

## File Map

| Action | Path | Responsibility |
|--------|------|----------------|
| Create | `.xyz/base/kustomization.yaml` | Declares base resources |
| Create | `.xyz/base/configmap.yaml` | nginx index.html content |
| Create | `.xyz/base/deployment.yaml` | nginx Deployment, 1 replica, mounts ConfigMap |
| Create | `.xyz/environments/prod/kustomization.yaml` | Extends base, applies patch |
| Create | `.xyz/environments/prod/patch.yaml` | Sets replicas: 2 |
| Create | `.github/workflows/kustomize-diff.yml` | CI workflow |

---

### Task 1: Initialize the git repository

**Files:**
- Working directory: `/home/acohea/Documents/Code/kustomize-diff-test-cases`

- [ ] **Step 1: Initialize git repo and set default branch to main**

```bash
cd /home/acohea/Documents/Code/kustomize-diff-test-cases
git init -b main
```

Expected output: `Initialized empty Git repository in .../kustomize-diff-test-cases/.git/`

- [ ] **Step 2: Commit the existing docs directory**

```bash
git add docs/
git commit -m "docs: add design spec and implementation plan"
```

Expected: commit succeeds.

- [ ] **Step 3: Verify**

```bash
git status
```

Expected: `On branch main`, working tree clean.

---

### Task 2: Create the base ConfigMap

**Files:**
- Create: `.xyz/base/configmap.yaml`

- [ ] **Step 1: Create the directory**

```bash
mkdir -p /home/acohea/Documents/Code/kustomize-diff-test-cases/.xyz/base
```

- [ ] **Step 2: Write the ConfigMap**

Create `.xyz/base/configmap.yaml`:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: nginx-index
data:
  index.html: |
    <!DOCTYPE html>
    <html>
    <head><title>kustomize-diff test</title></head>
    <body><h1>Hello from kustomize-diff-test-cases</h1></body>
    </html>
```

- [ ] **Step 3: Verify the file parses as valid YAML**

```bash
python3 -c "import yaml, sys; yaml.safe_load(open('.xyz/base/configmap.yaml'))" && echo "OK"
```

Expected: `OK`

---

### Task 3: Create the base Deployment

**Files:**
- Create: `.xyz/base/deployment.yaml`

- [ ] **Step 1: Write the Deployment**

Create `.xyz/base/deployment.yaml`:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
        - name: nginx
          image: nginx:1.27
          ports:
            - containerPort: 80
          volumeMounts:
            - name: html
              mountPath: /usr/share/nginx/html
      volumes:
        - name: html
          configMap:
            name: nginx-index
```

- [ ] **Step 2: Verify the file parses as valid YAML**

```bash
python3 -c "import yaml, sys; yaml.safe_load(open('.xyz/base/deployment.yaml'))" && echo "OK"
```

Expected: `OK`

---

### Task 4: Create the base kustomization.yaml

**Files:**
- Create: `.xyz/base/kustomization.yaml`

- [ ] **Step 1: Write the kustomization**

Create `.xyz/base/kustomization.yaml`:

```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

resources:
  - configmap.yaml
  - deployment.yaml
```

- [ ] **Step 2: Verify `kustomize build` succeeds on base**

```bash
kustomize build .xyz/base
```

Expected: YAML output containing both the ConfigMap and the Deployment with `replicas: 1`.

- [ ] **Step 3: Commit base overlay**

```bash
git add .xyz/base/
git commit -m "feat: add base nginx overlay with ConfigMap and Deployment"
```

---

### Task 5: Create the prod overlay

**Files:**
- Create: `.xyz/environments/prod/kustomization.yaml`
- Create: `.xyz/environments/prod/patch.yaml`

- [ ] **Step 1: Create the directory**

```bash
mkdir -p /home/acohea/Documents/Code/kustomize-diff-test-cases/.xyz/environments/prod
```

- [ ] **Step 2: Write the prod kustomization**

Create `.xyz/environments/prod/kustomization.yaml`:

```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

resources:
  - ../../base

patches:
  - path: patch.yaml
```

- [ ] **Step 3: Write the prod patch**

Create `.xyz/environments/prod/patch.yaml`:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
spec:
  replicas: 2
```

- [ ] **Step 4: Verify `kustomize build` succeeds on prod overlay**

```bash
kustomize build .xyz/environments/prod
```

Expected: YAML output with the Deployment showing `replicas: 2` (patched from base's `replicas: 1`).

- [ ] **Step 5: Commit prod overlay**

```bash
git add .xyz/environments/prod/
git commit -m "feat: add prod overlay with replicas: 2 patch"
```

---

### Task 6: Create the GitHub Actions workflow

**Files:**
- Create: `.github/workflows/kustomize-diff.yml`

- [ ] **Step 1: Create the directory**

```bash
mkdir -p /home/acohea/Documents/Code/kustomize-diff-test-cases/.github/workflows
```

- [ ] **Step 2: Write the workflow**

Create `.github/workflows/kustomize-diff.yml`:

```yaml
name: kustomize-diff

on:
  push:
    branches-ignore:
      - main

jobs:
  kustomize-diff:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - uses: AdrienneCohea/kustomize-diff@v1
        with:
          search-path: .xyz
```

- [ ] **Step 3: Verify the file parses as valid YAML**

```bash
python3 -c "import yaml, sys; yaml.safe_load(open('.github/workflows/kustomize-diff.yml'))" && echo "OK"
```

Expected: `OK`

- [ ] **Step 4: Commit the workflow**

```bash
git add .github/workflows/kustomize-diff.yml
git commit -m "feat: add kustomize-diff workflow for non-main branches"
```

---

### Task 7: Create GitHub repo and push

**Prerequisites:** `gh` CLI must be authenticated (`gh auth status` should succeed).

- [ ] **Step 1: Confirm gh CLI is authenticated**

```bash
gh auth status
```

Expected: Shows your account is logged in. If not, run `gh auth login` first.

- [ ] **Step 2: Create the GitHub repository**

```bash
gh repo create kustomize-diff-test-cases --public --source=. --remote=origin --push
```

Expected: Output shows repo created and `main` branch pushed. The URL will be printed (e.g. `https://github.com/<your-account>/kustomize-diff-test-cases`).

- [ ] **Step 3: Verify the push**

```bash
gh repo view kustomize-diff-test-cases --web
```

This opens the repo in your browser. Confirm the files are present.

- [ ] **Step 4: Smoke test — push a test branch to trigger the workflow**

```bash
git checkout -b test/smoke-test
git commit --allow-empty -m "test: trigger kustomize-diff workflow"
git push -u origin test/smoke-test
```

- [ ] **Step 5: Watch the workflow run**

```bash
gh run watch
```

Or open the Actions tab in the browser. Expected: workflow runs successfully and produces a diff report (the empty commit won't change overlays, so the summary will show no changes — that's fine for a smoke test).

- [ ] **Step 6: Clean up test branch (optional)**

```bash
git checkout main
git branch -d test/smoke-test
git push origin --delete test/smoke-test
```
