# Kaniko Build Template

使用 Kaniko 在容器內建置 Docker 映像，無需 Docker daemon。支援推送到 DockerHub 或 GitHub Container Registry (GHCR)。

## 功能

- 使用 Kaniko 在容器內建置映像（無需 Docker daemon）
- 自動產生版本標籤（日期.commit）
- 支援 layer caching 加速建置
- 支援推送到 DockerHub 或 GitHub Container Registry

## 使用方式

### 推送到 DockerHub（預設）

```yaml
jobs:
  build:
    uses: <your-org>/public-devops-templates/.github/workflows/build-kaniko-tpl.yml@v1.3.0
    with:
      repository: "my-repo"
      container_repository: "myorg/my-service"
      working_directory: "services/my-service"
      dockerfile_path: "Dockerfile"
    secrets:
      DOCKERHUB_USERNAME: ${{ secrets.DOCKERHUB_USERNAME }}
      DOCKERHUB_TOKEN: ${{ secrets.DOCKERHUB_TOKEN }}
```

### 推送到 GitHub Container Registry (GHCR)

```yaml
jobs:
  build:
    uses: <your-org>/public-devops-templates/.github/workflows/build-kaniko-tpl.yml@v1.3.0
    permissions:
      contents: read
      packages: write
    with:
      repository: "my-repo"
      container_repository: "myorg/my-service"
      working_directory: "services/my-service"
      dockerfile_path: "Dockerfile"
      registry: "ghcr"
    secrets: inherit
```

> **重要**：使用 GHCR 時，caller workflow 必須設定 `permissions: packages: write`

## 輸入參數

| 參數 | 必填 | 預設值 | 說明 |
|------|------|--------|------|
| `repository` | 是 | - | 儲存庫名稱 |
| `container_repository` | 是 | - | 容器映像儲存庫路徑（例：`myorg/myimage`） |
| `working_directory` | 是 | - | 工作目錄路徑 |
| `dockerfile_path` | 是 | - | Dockerfile 相對路徑 |
| `registry` | 否 | `dockerhub` | 目標 registry：`dockerhub` 或 `ghcr` |
| `cache` | 否 | `true` | 是否啟用 layer cache |
| `runs_on` | 否 | `["self-hosted", "linux"]` | Runner 類型 |

## 必要 Secrets

### DockerHub（registry: dockerhub）

| Secret | 說明 |
|--------|------|
| `DOCKERHUB_USERNAME` | Docker Hub 使用者名稱 |
| `DOCKERHUB_TOKEN` | Docker Hub 存取權杖 |

### GitHub Container Registry（registry: ghcr）

無需額外設定 secrets，自動使用 `github.token`。調用端只需設定 `permissions: packages: write`。

## 版本標籤格式

映像會被標記為：`{YYYY-MM-DD}.{commit_sha}`

例如：`ghcr.io/myorg/my-service:2026-01-16.abc12345`

## GHCR 權限設定

使用 GHCR 時，需要確保：

1. **Caller workflow 設定權限**：
   ```yaml
   permissions:
     contents: read
     packages: write
   ```

2. **Repository 設定**（如果使用 `GITHUB_TOKEN`）：
   - Settings → Actions → General → Workflow permissions
   - 設為 "Read and write permissions"

## 完整範例

### DockerHub

```yaml
name: Build to DockerHub

on:
  push:
    branches: [main]

jobs:
  build:
    uses: <your-org>/public-devops-templates/.github/workflows/build-kaniko-tpl.yml@v1.3.0
    with:
      repository: "my-app"
      container_repository: "mycompany/my-app"
      working_directory: "."
      dockerfile_path: "Dockerfile"
      cache: true
    secrets:
      DOCKERHUB_USERNAME: ${{ secrets.DOCKERHUB_USERNAME }}
      DOCKERHUB_TOKEN: ${{ secrets.DOCKERHUB_TOKEN }}
```

### GitHub Container Registry

```yaml
name: Build to GHCR

on:
  push:
    branches: [main]

jobs:
  build:
    uses: <your-org>/public-devops-templates/.github/workflows/build-kaniko-tpl.yml@v1.3.0
    permissions:
      contents: read
      packages: write
    with:
      repository: "my-app"
      container_repository: "myorg/my-app"
      working_directory: "."
      dockerfile_path: "Dockerfile"
      registry: "ghcr"
    secrets: inherit
```
