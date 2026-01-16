# Kubernetes Deploy Template

自動化部署應用程式到 Kubernetes 集群。

## 功能

- 使用 k8s-deploy 工具更新 kustomize overlay 配置
- 支援多服務同時部署
- 自動提交版本變更到 Git
- 支援自訂 deploy 路徑

## 使用方式

```yaml
jobs:
  deploy:
    uses: your-org/devops-templates/.github/workflows/deploy-kubernetes-tpl.yml@main
    with:
      version: "2024-01-15.abc123"
      services: '["api-server", "web-app"]'
      overlays: '["cluster-a/api-server", "cluster-a/web-app"]'
    secrets:
      KUBECONFIG: ${{ secrets.KUBECONFIG }}
```

## 輸入參數

| 參數 | 必填 | 預設值 | 說明 |
|------|------|--------|------|
| `version` | 是 | - | 要部署的版本號 |
| `services` | 是 | - | 服務名稱列表，JSON 陣列格式 |
| `overlays` | 是 | - | Kustomize overlay 路徑列表，JSON 陣列格式 |
| `k8s-deploy-image` | 否 | `myorg/k8s-deploy:latest` | k8s-deploy CLI 容器映像 |
| `deploy-path` | 否 | `/workspace/deploy` | deploy 目錄路徑 |
| `runs_on` | 否 | `ubuntu-latest` | Runner 類型 |

## 必要 Secrets

| Secret | 說明 |
|--------|------|
| `KUBECONFIG` | Base64 編碼的 kubeconfig 檔案內容 |

## 參數格式說明

### services（JSON 陣列）

```json
["service-a", "service-b", "service-c"]
```

### overlays（JSON 陣列）

```json
["cluster-name/service-a", "cluster-name/service-b", "cluster-name/service-c"]
```

> **注意**：`services` 和 `overlays` 的順序必須對應

## 完整範例

### 單一服務部署

```yaml
name: Deploy to Production

on:
  workflow_dispatch:
    inputs:
      version:
        description: 'Version to deploy'
        required: true

jobs:
  deploy:
    uses: your-org/devops-templates/.github/workflows/deploy-kubernetes-tpl.yml@main
    with:
      version: ${{ github.event.inputs.version }}
      services: '["api-server"]'
      overlays: '["prod-cluster/api-server"]'
    secrets:
      KUBECONFIG: ${{ secrets.KUBECONFIG }}
```

### 多服務部署

```yaml
name: Deploy All Services

on:
  push:
    branches: [main]

jobs:
  build:
    # ... 建置步驟
    outputs:
      version: ${{ steps.version.outputs.version }}

  deploy:
    needs: build
    uses: your-org/devops-templates/.github/workflows/deploy-kubernetes-tpl.yml@main
    with:
      version: ${{ needs.build.outputs.version }}
      services: '["api-server", "web-app", "worker"]'
      overlays: '["prod/api-server", "prod/web-app", "prod/worker"]'
    secrets:
      KUBECONFIG: ${{ secrets.KUBECONFIG }}
```

### 多環境部署

```yaml
name: Deploy to Staging

on:
  push:
    branches: [develop]

jobs:
  deploy-staging:
    uses: your-org/devops-templates/.github/workflows/deploy-kubernetes-tpl.yml@main
    with:
      version: "staging-${{ github.sha }}"
      services: '["api-server"]'
      overlays: '["staging-cluster/api-server"]'
    secrets:
      KUBECONFIG: ${{ secrets.STAGING_KUBECONFIG }}
```

## KUBECONFIG 設定

### 產生 Base64 編碼的 kubeconfig

```bash
cat ~/.kube/config | base64 -w 0
```

或在 macOS：

```bash
cat ~/.kube/config | base64
```

### 設定 GitHub Secret

1. 前往 Repository Settings → Secrets and variables → Actions
2. 新增 secret：`KUBECONFIG`
3. 貼上 Base64 編碼的內容

## 目錄結構

預期的 deploy 目錄結構：

```
deploy/
├── base/
│   └── kustomization.yaml
└── overlays/
    ├── prod-cluster/
    │   ├── api-server/
    │   │   └── kustomization.yaml
    │   └── web-app/
    │       └── kustomization.yaml
    └── staging-cluster/
        └── api-server/
            └── kustomization.yaml
```
