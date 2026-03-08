# Kubernetes Deploy Pipeline

自動化 Kubernetes 部署流程，支援 CI 觸發、手動觸發和直接修改 manifest 三種方式。

## 架構概覽

```
rs-apps (CI)                              rs-manifest (CD)
┌──────────────────┐                      ┌──────────────────────────────┐
│ build-image      │                      │ update-tag                   │
│ (Kaniko)         │                      │   - 更新 values.yaml         │
│       ↓          │                      │   - commit & push            │
│ trigger-deploy   │ ──repository_────→   │   - output: commit_sha       │
│                  │    dispatch          │       ↓ (ref=commit_sha)     │
└──────────────────┘                      │ deploy                       │
                                          │   - checkout 最新 main HEAD  │
                                          │   - helm upgrade             │
                                          │       ↓                      │
                                          │ health-check                 │
                                          │   - kubectl rollout status   │
                                          └──────────────────────────────┘
```

## 觸發方式

| 觸發方式 | update-tag job | deploy job | 使用場景 |
|----------|----------------|------------|----------|
| **repository_dispatch** | ✅ 執行 | ✅ 執行 | CI pipeline 自動觸發 |
| **workflow_dispatch** | ✅ 執行 | ✅ 執行 | 手動指定版本部署 |
| **push** | ⏭️ 跳過 | ✅ 執行 | 直接修改 values.yaml |

## Templates

### 1. deploy-trigger-deploy-tpl.yml

在 CI build 成功後觸發 manifest repo 的部署 workflow。

**使用方式：**

```yaml
trigger-deploy:
  needs: build-image
  if: github.ref == 'refs/heads/main' && github.event_name == 'push'
  uses: rhincodon-studio/public-devops-templates/.github/workflows/deploy-trigger-deploy-tpl.yml@main
  with:
    app: my-app
    cluster: MY-CLUSTER
    namespace: my-app
    manifest_repo: myorg/manifest-repo
  secrets:
    MANIFEST_REPO_TOKEN: ${{ secrets.MANIFEST_REPO_TOKEN }}
```

**輸入參數：**

| 參數 | 必填 | 預設值 | 說明 |
|------|------|--------|------|
| `app` | 是 | - | 應用程式名稱 |
| `cluster` | 是 | - | 目標叢集名稱 |
| `namespace` | 是 | - | Kubernetes namespace |
| `manifest_repo` | 是 | - | manifest repo 路徑 (org/repo) |
| `image_tag` | 否 | 自動產生 | 映像標籤 (預設: YYYY-MM-DD.commit) |
| `event_type` | 否 | `deploy-app` | repository_dispatch 事件類型 |

**必要 Secrets：**

| Secret | 說明 |
|--------|------|
| `MANIFEST_REPO_TOKEN` | 有權限觸發 manifest repo workflow 的 GitHub PAT |

---

### 2. deploy-update-tag-kubernetes-tpl.yml

更新 Helm chart 的 `values.yaml` 中的 image tag。

**使用方式：**

```yaml
update-tag:
  uses: rhincodon-studio/public-devops-templates/.github/workflows/deploy-update-tag-kubernetes-tpl.yml@main
  with:
    app: my-app
    cluster: MY-CLUSTER
    namespace: my-app
    image_tag: "2026-01-16.abc12345"
  secrets:
    GIT_TOKEN: ${{ secrets.GIT_TOKEN }}
```

**輸入參數：**

| 參數 | 必填 | 預設值 | 說明 |
|------|------|--------|------|
| `app` | 是 | - | 應用程式名稱 |
| `cluster` | 是 | - | 目標叢集名稱 |
| `namespace` | 是 | - | Kubernetes namespace |
| `image_tag` | 是 | - | 要更新的映像標籤 |
| `values_path` | 否 | 自動計算 | values.yaml 路徑 |
| `triggered_by` | 否 | `manual` | 觸發來源 |
| `commit_sha` | 否 | `N/A` | 上游 commit SHA |

**必要 Secrets：**

| Secret | 說明 |
|--------|------|
| `GIT_TOKEN` | 有 repo write 權限的 GitHub PAT |

**輸出：**

| 輸出 | 說明 |
|------|------|
| `changed` | 是否有變更 (true/false) |
| `commit_sha` | tag 更新後的 commit SHA（可傳給 deploy job 的 `ref`） |

---

### 3. deploy-kubernetes-tpl.yml

偵測變更的 Helm chart 並部署到 Kubernetes。

**使用方式：**

```yaml
# 自動偵測變更
deploy:
  uses: rhincodon-studio/public-devops-templates/.github/workflows/deploy-kubernetes-tpl.yml@main
  with:
    base_sha: ${{ github.event.before }}
    head_sha: ${{ github.sha }}
  secrets: inherit

# 手動指定 chart
deploy:
  uses: rhincodon-studio/public-devops-templates/.github/workflows/deploy-kubernetes-tpl.yml@main
  with:
    charts: "deploy/my-app/helm/MY-CLUSTER/my-app"
  secrets: inherit
```

**輸入參數：**

| 參數 | 必填 | 預設值 | 說明 |
|------|------|--------|------|
| `base_sha` | 否 | `HEAD~1` | 比較的基準 commit SHA |
| `head_sha` | 否 | `HEAD` | 比較的目標 commit SHA |
| `charts` | 否 | - | 手動指定 chart 路徑（空格分隔） |
| `ref` | 否 | default branch | 要 checkout 的 Git ref（見下方說明） |
| `k8s_deploy_image` | 是 | - | k8s-deploy 容器映像 |
| `kubeconfig_dir` | 否 | `$HOME/.kube` | kubeconfig 目錄 |
| `helm_timeout` | 否 | `5m` | Helm 部署超時時間 |
| `runs_on` | 否 | `["self-hosted", "linux"]` | Runner 類型 |

> **`ref` 參數說明：** 未指定時，checkout 會使用 repository 的 default branch 最新 HEAD，
> 而非 `github.sha`。這避免了 `repository_dispatch` 觸發時 `github.sha` 指向
> `update-tag` push 之前的舊 commit，導致部署到錯誤版本的問題。
> 也可以傳入 `update-tag` 的 `commit_sha` output 來精確指定 checkout 的 commit。

**必要設定：**

- Self-hosted runner 需預先設定 kubeconfig 檔案
- 格式：`$HOME/.kube/<cluster-name>`

---

## 完整範例

### CI Workflow (rs-apps)

```yaml
name: 🧩 my-app

on:
  push:
    branches: [main]
    paths:
      - 'apps/my-app/**'
  pull_request:
    paths:
      - 'apps/my-app/**'
  workflow_dispatch:

jobs:
  build-image:
    uses: rhincodon-studio/public-devops-templates/.github/workflows/build-kaniko-tpl.yml@main
    permissions:
      contents: read
      packages: write
    with:
      repository: my_app
      registry: ghcr
      container_repository: myorg/my-app
      working_directory: .
      dockerfile_path: apps/my-app/Dockerfile
    secrets: inherit

  trigger-deploy:
    needs: build-image
    if: github.ref == 'refs/heads/main' && github.event_name == 'push'
    uses: rhincodon-studio/public-devops-templates/.github/workflows/deploy-trigger-deploy-tpl.yml@main
    with:
      app: my-app
      cluster: MY-CLUSTER
      namespace: my-app
      manifest_repo: myorg/manifest-repo
    secrets:
      MANIFEST_REPO_TOKEN: ${{ secrets.MANIFEST_REPO_TOKEN }}
```

### CD Workflow (manifest-repo)

```yaml
name: 🚀 Deploy to Kubernetes

on:
  repository_dispatch:
    types: [deploy-app]

  push:
    branches: [main]
    paths:
      - 'deploy/**/helm/**'

  workflow_dispatch:
    inputs:
      app:
        description: 'Application name'
        required: true
        type: choice
        options:
          - my-app
          - other-app
      cluster:
        description: 'Target cluster'
        required: true
        type: choice
        options:
          - MY-CLUSTER
      namespace:
        description: 'Kubernetes namespace'
        required: true
        type: choice
        options:
          - my-app
          - other-app
      image_tag:
        description: 'Image tag to deploy'
        required: true
        type: string

jobs:
  update-tag:
    if: github.event_name == 'repository_dispatch' || github.event_name == 'workflow_dispatch'
    uses: rhincodon-studio/public-devops-templates/.github/workflows/deploy-update-tag-kubernetes-tpl.yml@main
    with:
      app: ${{ github.event_name == 'repository_dispatch' && github.event.client_payload.app || inputs.app }}
      cluster: ${{ github.event_name == 'repository_dispatch' && github.event.client_payload.cluster || inputs.cluster }}
      namespace: ${{ github.event_name == 'repository_dispatch' && github.event.client_payload.namespace || inputs.namespace }}
      image_tag: ${{ github.event_name == 'repository_dispatch' && github.event.client_payload.image_tag || inputs.image_tag }}
      triggered_by: ${{ github.event_name == 'repository_dispatch' && github.event.client_payload.triggered_by || 'manual' }}
      commit_sha: ${{ github.event_name == 'repository_dispatch' && github.event.client_payload.commit_sha || 'N/A' }}
    secrets:
      GIT_TOKEN: ${{ secrets.GIT_TOKEN }}

  deploy:
    needs: [update-tag]
    if: |
      always() &&
      (needs.update-tag.result == 'success' && needs.update-tag.outputs.changed == 'true') ||
      (github.event_name == 'push')
    uses: rhincodon-studio/public-devops-templates/.github/workflows/deploy-kubernetes-tpl.yml@main
    with:
      base_sha: ${{ github.event_name == 'push' && github.event.before || '' }}
      head_sha: ${{ github.event_name == 'push' && github.sha || '' }}
      ref: ${{ needs.update-tag.outputs.commit_sha || '' }}
      charts: ${{ github.event_name != 'push' && format('deploy/{0}/helm/{1}/{2}', ...) || '' }}
    secrets: inherit
```

> **注意：** `ref` 傳入 `update-tag` 的 `commit_sha` output，確保 deploy job
> checkout 的是包含新 image tag 的 commit，而非 `repository_dispatch` 觸發時的舊 `github.sha`。

---

## 目錄結構

```
manifest-repo/
├── .github/
│   └── workflows/
│       └── deploy.yml
└── deploy/
    └── my-app/
        └── helm/
            └── MY-CLUSTER/
                └── my-app/
                    ├── Chart.yaml
                    ├── values.yaml
                    └── templates/
                        ├── app-deploy.yaml
                        └── ingress.yaml
```

---

## 必要 Secrets 設定

### CI Repo (rs-apps)

| Secret | 說明 |
|--------|------|
| `MANIFEST_REPO_TOKEN` | 有權限觸發 manifest repo workflow 的 GitHub PAT (需要 `repo` scope) |

### CD Repo (manifest-repo)

| Secret | 說明 |
|--------|------|
| `GIT_TOKEN` | 有 repo write 權限的 GitHub PAT (用於 commit & push values.yaml) |

### Self-hosted Runner

- 預先設定 kubeconfig 檔案於 `$HOME/.kube/<cluster-name>`
- 安裝 `helm` CLI

---

## 版本標籤格式

映像標籤自動產生格式：`YYYY-MM-DD.{commit_sha_8}`

例如：`2026-01-16.abc12345`
