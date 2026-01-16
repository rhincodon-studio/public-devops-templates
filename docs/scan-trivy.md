# Trivy Security Scan Template

使用 Trivy 掃描依賴套件漏洞和容器映像安全問題。

## 功能

- 掃描專案依賴套件漏洞
- 支援容器映像掃描（選填）
- 檢測 CRITICAL 和 HIGH 等級的安全問題
- 免費且開源

## 使用方式

### 只掃描依賴套件

```yaml
jobs:
  security-scan:
    uses: your-org/devops-templates/.github/workflows/scan-trivy-tpl.yml@main
```

### 同時掃描容器映像

```yaml
jobs:
  security-scan:
    uses: your-org/devops-templates/.github/workflows/scan-trivy-tpl.yml@main
    with:
      image-name: "myorg/myimage:latest"
```

## 輸入參數

| 參數 | 必填 | 預設值 | 說明 |
|------|------|--------|------|
| `image-name` | 否 | - | 容器映像名稱（若提供則執行容器掃描） |
| `runs_on` | 否 | `ubuntu-latest` | Runner 類型 |

## 掃描類型

### 依賴套件掃描（deps-scan）

- 自動執行
- 掃描檔案系統中的依賴套件
- 檢測 CRITICAL 和 HIGH 等級漏洞

### 容器映像掃描（image-scan）

- 僅在提供 `image-name` 時執行
- 掃描容器映像的漏洞

## 完整範例

### 基本依賴掃描

```yaml
name: Security Scan

on:
  push:
    branches: [main]
  pull_request:

jobs:
  trivy:
    uses: your-org/devops-templates/.github/workflows/scan-trivy-tpl.yml@main
```

### 依賴 + 容器掃描

```yaml
name: Full Security Scan

on:
  push:
    branches: [main]

jobs:
  trivy:
    uses: your-org/devops-templates/.github/workflows/scan-trivy-tpl.yml@main
    with:
      image-name: "mycompany/api-server:latest"
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
    uses: your-org/devops-templates/.github/workflows/scan-trivy-tpl.yml@main
    with:
      image-name: "mycompany/api:latest"
```

## Trivy vs Snyk

| 特性 | Trivy | Snyk |
|------|-------|------|
| 費用 | 免費開源 | 免費方案有限制 |
| 設定 | 無需 Token | 需要 API Token |
| 修復建議 | 基本 | 詳細 |
| 支援語言 | 廣泛 | 廣泛 |
| 容器掃描 | 支援 | 支援 |

建議：可以同時使用兩者，獲得更全面的安全掃描結果。
