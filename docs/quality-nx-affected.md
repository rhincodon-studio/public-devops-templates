# Nx Affected Quality Check Template

使用 Nx affected 機制，只對受影響的專案執行品質檢查，大幅提升 monorepo CI 效率。

## 功能

- 使用 Nx affected 機制，只對受影響的專案執行檢查
- 自動偵測 PR 或 push 事件，設定正確的比較基準
- 支援 lint、test、build、e2e 等任務
- 可選安裝 Playwright 支援 e2e 測試

## 使用方式

### 基本用法

```yaml
jobs:
  quality:
    uses: your-org/devops-templates/.github/workflows/quality-nx-affected-tpl.yml@main
```

### 包含 e2e 測試

```yaml
jobs:
  quality:
    uses: your-org/devops-templates/.github/workflows/quality-nx-affected-tpl.yml@main
    with:
      tasks: "lint test build e2e"
      install_playwright: true
```

### 完整自訂

```yaml
jobs:
  quality:
    uses: your-org/devops-templates/.github/workflows/quality-nx-affected-tpl.yml@main
    with:
      node_version: "18"
      tasks: "lint test build e2e"
      install_playwright: true
      base_branch: "develop"
      npm_install_args: "--legacy-peer-deps"
```

## 輸入參數

| 參數 | 必填 | 預設值 | 說明 |
|------|------|--------|------|
| `node_version` | 否 | `20` | Node.js 版本 |
| `tasks` | 否 | `lint test build` | 要執行的 Nx 任務，以空格分隔 |
| `install_playwright` | 否 | `false` | 是否安裝 Playwright 瀏覽器 |
| `base_branch` | 否 | `main` | 預設比較基準分支 |
| `npm_install_args` | 否 | `--legacy-peer-deps` | npm ci 額外參數 |
| `runs_on` | 否 | `ubuntu-latest` | Runner 類型 |

## 運作原理

### Nx Affected 機制

`nx affected` 會：
1. 比較 base commit 和 head commit 之間的變更
2. 找出哪些檔案被修改
3. 計算哪些專案受到影響（包含依賴關係）
4. 只對這些專案執行指定的任務

### 比較基準設定

- **Pull Request**：比較 PR 的目標分支（`github.base_ref`）
- **Push**：比較設定的 `base_branch`（預設 `main`）

## 完整範例

### PR 品質檢查

```yaml
name: PR Quality Check

on:
  pull_request:
    branches: [main, develop]

jobs:
  quality:
    uses: your-org/devops-templates/.github/workflows/quality-nx-affected-tpl.yml@main
    with:
      tasks: "lint test build"
```

### 包含 E2E 測試

```yaml
name: Full Quality Check

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  quality:
    uses: your-org/devops-templates/.github/workflows/quality-nx-affected-tpl.yml@main
    with:
      node_version: "20"
      tasks: "lint test build e2e"
      install_playwright: true
      base_branch: "main"
```

### 使用 develop 作為基準

```yaml
name: Feature Branch Check

on:
  pull_request:
    branches: [develop]

jobs:
  quality:
    uses: your-org/devops-templates/.github/workflows/quality-nx-affected-tpl.yml@main
    with:
      base_branch: "develop"
      tasks: "lint test"
```

## 效能優勢

假設 monorepo 有 10 個專案，你只改了 `packages/api`：

| 方式 | 執行範圍 | 時間 |
|------|----------|------|
| 傳統 CI | 全部 10 個專案 | 慢 |
| Nx Affected | 只有 `api` 和依賴它的專案 | 快 |

這就是 Nx affected 機制的核心價值。
