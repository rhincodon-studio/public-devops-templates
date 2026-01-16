# Snyk Security Scan Template

使用 Snyk 進行依賴套件和容器映像安全掃描。

## 功能

- 掃描專案依賴套件的已知漏洞
- 支援容器映像掃描（選填）
- 檢測漏洞並提供修復建議

## 使用方式

### 只掃描依賴套件

```yaml
jobs:
  security-scan:
    uses: your-org/devops-templates/.github/workflows/scan-snyk-tpl.yml@main
    secrets:
      SNYK_TOKEN: ${{ secrets.SNYK_TOKEN }}
```

### 同時掃描容器映像

```yaml
jobs:
  security-scan:
    uses: your-org/devops-templates/.github/workflows/scan-snyk-tpl.yml@main
    with:
      image-name: "myorg/myimage:latest"
      org: "my-snyk-org"
    secrets:
      SNYK_TOKEN: ${{ secrets.SNYK_TOKEN }}
```

## 輸入參數

| 參數 | 必填 | 預設值 | 說明 |
|------|------|--------|------|
| `image-name` | 否 | - | 容器映像名稱（若提供則執行容器掃描） |
| `org` | 否 | - | Snyk 組織 ID |
| `runs_on` | 否 | `ubuntu-latest` | Runner 類型 |

## 必要 Secrets

| Secret | 說明 |
|--------|------|
| `SNYK_TOKEN` | Snyk API 權杖 |

## 掃描類型

### 依賴套件掃描（deps-scan）

- 自動執行
- 掃描所有專案的依賴套件
- 使用 `snyk test --all-projects`

### 容器映像掃描（image-scan）

- 僅在提供 `image-name` 時執行
- 掃描容器映像的漏洞
- 使用 `snyk container test`

## 完整範例

### 基本依賴掃描

```yaml
name: Security Scan

on:
  push:
    branches: [main]
  pull_request:

jobs:
  snyk:
    uses: your-org/devops-templates/.github/workflows/scan-snyk-tpl.yml@main
    secrets:
      SNYK_TOKEN: ${{ secrets.SNYK_TOKEN }}
```

### 依賴 + 容器掃描

```yaml
name: Full Security Scan

on:
  push:
    branches: [main]

jobs:
  snyk:
    uses: your-org/devops-templates/.github/workflows/scan-snyk-tpl.yml@main
    with:
      image-name: "mycompany/api-server:latest"
      org: "mycompany"
    secrets:
      SNYK_TOKEN: ${{ secrets.SNYK_TOKEN }}
```

### 搭配建置流程

```yaml
name: Build and Scan

on:
  push:
    branches: [main]

jobs:
  build:
    uses: your-org/devops-templates/.github/workflows/build-docker-tpl.yml@main
    with:
      module: "api"
      workdir: "./api"
      image_name: "mycompany/api"
    secrets:
      DOCKERHUB_USERNAME: ${{ secrets.DOCKERHUB_USERNAME }}
      DOCKERHUB_TOKEN: ${{ secrets.DOCKERHUB_TOKEN }}

  security:
    needs: build
    uses: your-org/devops-templates/.github/workflows/scan-snyk-tpl.yml@main
    with:
      image-name: "mycompany/api:latest"
    secrets:
      SNYK_TOKEN: ${{ secrets.SNYK_TOKEN }}
```

## 取得 Snyk Token

1. 登入 [Snyk](https://snyk.io/)
2. 前往 Account Settings
3. 複製 API Token
4. 在 GitHub Repository Settings → Secrets 新增 `SNYK_TOKEN`
