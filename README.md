# DevOps Templates

可重複使用的 GitHub Actions Workflow Templates，用於建置、掃描、品質檢查和部署。

## 功能總覽

| 類別 | Template | 說明 |
|------|----------|------|
| **Build** | [build-docker-tpl](docs/build-docker.md) | 使用 Docker 建置並推送映像到 DockerHub |
| **Build** | [build-kaniko-tpl](docs/build-kaniko.md) | 使用 Kaniko 建置映像（支援 DockerHub / GHCR） |
| **Quality** | [quality-nx-affected-tpl](docs/quality-nx-affected.md) | Nx monorepo 受影響專案品質檢查 |
| **Security** | [scan-checkmarx-tpl](docs/scan-checkmarx.md) | Checkmarx AST SAST 靜態安全掃描 |
| **Security** | [scan-snyk-tpl](docs/scan-snyk.md) | Snyk 依賴套件與容器安全掃描 |
| **Security** | [scan-sonar-tpl](docs/scan-sonar.md) | SonarCloud / SonarQube 程式碼品質分析 |
| **Security** | [scan-trivy-tpl](docs/scan-trivy.md) | Trivy 依賴套件與容器漏洞掃描 |
| **Deploy** | [deploy-kubernetes-tpl](docs/deploy-kubernetes.md) | Kubernetes 自動化部署 |

## 快速開始

### 1. 引用 Template

在你的 workflow 中使用 `uses` 引用 template：

```yaml
name: CI/CD Pipeline

on:
  push:
    branches: [main]

jobs:
  build:
    uses: your-org/devops-templates/.github/workflows/build-kaniko-tpl.yml@main
    with:
      repository: "my-service"
      container_repository: "myorg/my-service"
      working_directory: "."
      dockerfile_path: "Dockerfile"
    secrets:
      DOCKERHUB_USERNAME: ${{ secrets.DOCKERHUB_USERNAME }}
      DOCKERHUB_TOKEN: ${{ secrets.DOCKERHUB_TOKEN }}
```

### 2. 設定 Secrets

根據你使用的 template，在 GitHub Repository Settings 中設定對應的 secrets。

## 文件

詳細使用說明請參閱 [docs](docs/) 目錄：

- [Docker Build](docs/build-docker.md)
- [Kaniko Build](docs/build-kaniko.md)
- [Nx Affected Quality Check](docs/quality-nx-affected.md)
- [Checkmarx SAST](docs/scan-checkmarx.md)
- [Snyk Security](docs/scan-snyk.md)
- [Sonar Analysis](docs/scan-sonar.md)
- [Trivy Security](docs/scan-trivy.md)
- [Kubernetes Deploy](docs/deploy-kubernetes.md)

## 授權

MIT License
