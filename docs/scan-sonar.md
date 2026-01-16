# Sonar Universal Scan Template

支援 SonarCloud 和自架 SonarQube 兩種模式的程式碼品質分析。

## 功能

- 支援 SonarCloud（雲端版）
- 支援自架 SonarQube
- 進行程式碼品質與安全性分析
- 產生 SARIF 格式報告並上傳為 artifact

## 使用方式

### SonarCloud 模式

```yaml
jobs:
  code-quality:
    uses: your-org/devops-templates/.github/workflows/scan-sonar-tpl.yml@main
    with:
      mode: "cloud"
      project-key: "my-project-key"
      project-name: "My Project"
      organization: "my-org"
    secrets:
      SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
```

### Self-hosted SonarQube 模式

```yaml
jobs:
  code-quality:
    uses: your-org/devops-templates/.github/workflows/scan-sonar-tpl.yml@main
    with:
      mode: "selfhosted"
      project-key: "my-project-key"
      project-name: "My Project"
    secrets:
      SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
      SONAR_HOST_URL: ${{ secrets.SONAR_HOST_URL }}
```

## 輸入參數

| 參數 | 必填 | 預設值 | 說明 |
|------|------|--------|------|
| `mode` | 是 | - | 掃描模式：`cloud` 或 `selfhosted` |
| `project-key` | 是 | - | Sonar 專案金鑰 |
| `project-name` | 是 | - | Sonar 專案名稱 |
| `organization` | 條件 | - | SonarCloud 組織名稱（cloud 模式必填） |
| `runs_on` | 否 | `ubuntu-latest` | Runner 類型 |

## 必要 Secrets

### SonarCloud 模式

| Secret | 說明 |
|--------|------|
| `SONAR_TOKEN` | SonarCloud 存取權杖 |

### Self-hosted 模式

| Secret | 說明 |
|--------|------|
| `SONAR_TOKEN` | SonarQube 存取權杖 |
| `SONAR_HOST_URL` | SonarQube 伺服器網址 |

## 掃描報告

掃描完成後，SARIF 報告會上傳為 GitHub Actions artifact：
- 名稱：`sonar-report`
- 檔案：`sonar-report.sarif`

## 完整範例

### SonarCloud

```yaml
name: Code Quality

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  sonar:
    uses: your-org/devops-templates/.github/workflows/scan-sonar-tpl.yml@main
    with:
      mode: "cloud"
      project-key: "myorg_myproject"
      project-name: "My Project"
      organization: "myorg"
    secrets:
      SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
```

### Self-hosted SonarQube

```yaml
name: Code Quality

on:
  push:
    branches: [main]

jobs:
  sonar:
    uses: your-org/devops-templates/.github/workflows/scan-sonar-tpl.yml@main
    with:
      mode: "selfhosted"
      project-key: "my-internal-project"
      project-name: "Internal Project"
    secrets:
      SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
      SONAR_HOST_URL: ${{ secrets.SONAR_HOST_URL }}
```

## 設定指南

### SonarCloud 設定

1. 前往 [SonarCloud](https://sonarcloud.io/)
2. 建立新專案或選擇現有專案
3. 取得 Project Key 和 Organization
4. 前往 Account → Security → Generate Token
5. 在 GitHub Secrets 設定 `SONAR_TOKEN`

### Self-hosted SonarQube 設定

1. 登入你的 SonarQube 伺服器
2. 建立專案，取得 Project Key
3. 前往 Administration → Security → Users → Tokens
4. 產生 Token
5. 在 GitHub Secrets 設定：
   - `SONAR_TOKEN`：產生的 Token
   - `SONAR_HOST_URL`：SonarQube 伺服器網址（例：`https://sonar.mycompany.com`）
