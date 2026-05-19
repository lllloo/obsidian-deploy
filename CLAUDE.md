# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

**本檔涵蓋**：repo 工程層——Quartz 部署框架、CI 觸發規則、Quartz 設定行為。
**不涵蓋**：Vault 內容規則、skills、稽核腳本 → 這些已移至 [obsidian-memory](https://github.com/lllloo/obsidian-memory)。

## 專案概覽

此 repo 是 Quartz 4 部署框架，負責把 `obsidian-memory` vault 發佈至 `ob.bugloop.com`。

架構職責分離：
- **obsidian-deploy**（本 repo）— Quartz 框架設定、CI/CD、GitHub Pages 部署
- **obsidian-memory**（另一個 repo）— Vault 筆記本體、skills、稽核腳本

`content/` 目錄**不在本 repo**。CI 會在執行期從 obsidian-memory 自動 checkout 至 `content/`；本機執行 vault 相關指令前須手動 clone：

```bash
git clone https://github.com/lllloo/obsidian-memory.git content
```

## 常用指令

需要 Node.js 22+、npm 10.9.2+（`package.json` engines）。

```bash
npx quartz build --serve    # 本地預覽（localhost:8080）需先有 content/
npm run check               # TypeScript 型別檢查 + Prettier 格式驗證
npm run format              # 自動格式化
npm run test                # 執行 scripts/vault-schema.test.mjs（tsx --test）
npm run vault:check         # 稽核 content/ frontmatter 與檔名（只報告；需 content/）
npm run vault:fix           # 稽核並自動修正（需 content/）
```

`vault:check` / `vault:fix` 執行的是本 repo 的 `scripts/vault-check.mjs`（`--json` 可輸出機器可讀格式）。

## CI 觸發規則

`.github/workflows/deploy.yml` 只在下列路徑 push 到 `main` 時自動觸發：

- `quartz/**`
- `quartz.config.ts` / `quartz.layout.ts`
- `package.json` / `package-lock.json`
- `scripts/**`
- `.github/workflows/deploy.yml`

**Vault 筆記變動不觸發 CI**，需手動 `workflow_dispatch`。成功/失敗皆透過 `DISCORD_WEBHOOK` secret 發送 Discord 通知。

CI 流程：checkout obsidian-memory → `npm run vault:check` → `npx quartz build` → deploy GitHub Pages。

## Quartz 設定重點（`quartz.config.ts`）

- `ignorePatterns`：`private`、`.obsidian`、`CLAUDE.md`、`Inbox`（整個 `Inbox/` 不發佈）、`.env*`、`secrets*` 等敏感資料保險
- `filters: [Plugin.RemoveDrafts()]`：frontmatter 加 `draft: true` 的筆記不發佈
- 日期優先順序：frontmatter → git → filesystem（`CreatedModifiedDate`）
- Wikilink 解析：`shortest`（`CrawlLinks`），連結目標需在 `content/` 下存在
- Plugin pipeline：transformers → filters → emitters
- `CustomOgImages()` 已 comment 掉（會大幅拖慢 build）；需啟用時取消 comment

## 架構說明

- `quartz/` — Quartz 框架原始碼（不需修改）
- `quartz.config.ts` — 站台設定（外觀、plugins、ignorePatterns）
- `quartz.layout.ts` — 版面配置
- `scripts/vault-schema.mjs` — frontmatter schema **真實來源**（欄位白名單、順序、必填、型別，用 zod 定義）；變更欄位規則只改這裡
- `scripts/vault-check.mjs` — vault 稽核腳本（讀取 vault-schema.mjs）；硬規則自動修，語意層問題只 flag
- `scripts/vault-schema.test.mjs` — schema 單元測試（`npm run test` 執行）
- `AGENTS.md` — `CLAUDE.md` 的 symlink，給非 Claude Code 的 agent 工具讀
- `.clipper/vault-clipper.json` — Obsidian Web Clipper 模板（`Inbox/Clippings/` 抓取規則，frontmatter 白名單）；修改後 commit 以跨機器同步
