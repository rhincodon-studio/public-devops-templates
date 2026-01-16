# Checkmarx AST SAST Scan Template

使用 Checkmarx AST 進行靜態應用程式安全測試（SAST）。

## 功能

- 使用 Checkmarx AST 進行 SAST 掃描
- 支援自訂失敗策略（依嚴重性等級設定閾值）
- 產生掃描報告並上傳為 artifact

## 使用方式

```yaml
jobs:
  security-scan:
    uses: your-org/devops-templates/.github/workflows/scan-checkmarx-tpl.yml@main
    with:
      project-name: "my-project"
      fail-policy: "high=0;medium=5"
    secrets:
      CX_APIKEY: ${{ secrets.CX_APIKEY }}
      CX_TENANT: ${{ secrets.CX_TENANT }}
```

## 輸入參數

| 參數 | 必填 | 預設值 | 說明 |
|------|------|--------|------|
| `project-name` | 是 | - | Checkmarx 專案名稱 |
| `fail-policy` | 否 | `high=0;medium=5` | 失敗策略，格式：`severity=count` |
| `branch` | 否 | 當前分支 | 分支名稱 |
| `runs_on` | 否 | `ubuntu-latest` | Runner 類型 |

## 必要 Secrets

| Secret | 說明 |
|--------|------|
| `CX_APIKEY` | Checkmarx API 金鑰 |
| `CX_TENANT` | Checkmarx 租戶名稱 |

## 失敗策略說明

`fail-policy` 格式為 `severity=count`，多個條件用分號分隔：

- `high=0` - 發現任何 high 嚴重性問題即失敗
- `medium=5` - 發現超過 5 個 medium 嚴重性問題即失敗
- `high=0;medium=5` - 以上兩個條件都會觸發失敗

## 掃描報告

掃描完成後，報告會上傳為 GitHub Actions artifact：
- 名稱：`cx-results`
- 檔案：`cx-results.json`

## 完整範例

### 嚴格策略

```yaml
name: Security Scan

on:
  push:
    branches: [main]

jobs:
  checkmarx:
    uses: your-org/devops-templates/.github/workflows/scan-checkmarx-tpl.yml@main
    with:
      project-name: "my-secure-app"
      fail-policy: "high=0;medium=0;low=10"
    secrets:
      CX_APIKEY: ${{ secrets.CX_APIKEY }}
      CX_TENANT: ${{ secrets.CX_TENANT }}
```

### 寬鬆策略

```yaml
name: Security Scan

on:
  pull_request:

jobs:
  checkmarx:
    uses: your-org/devops-templates/.github/workflows/scan-checkmarx-tpl.yml@main
    with:
      project-name: "my-app"
      fail-policy: "high=5;medium=20"
    secrets:
      CX_APIKEY: ${{ secrets.CX_APIKEY }}
      CX_TENANT: ${{ secrets.CX_TENANT }}
```
