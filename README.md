# obsidian-deploy

Quartz 4 部署框架，將 [obsidian-memory](https://github.com/lllloo/obsidian-memory) vault 發佈至 [bugloop.com](https://bugloop.com)。

Vault 筆記本體、skills 與稽核腳本由 obsidian-memory 管理；本 repo 只負責 Quartz 設定與 CI/CD。

## 前置需求

- Node.js 22+
- npm 10.9.2+

本地預覽需先 clone vault：

```bash
git clone https://github.com/lllloo/obsidian-memory.git content
```

## 開發指令

```bash
npx quartz build --serve    # 本地預覽（http://localhost:8080）
npm run check               # TypeScript 型別檢查 + Prettier 格式驗證
npm run format              # 自動格式化
```

## 發佈

push 到 `main` 後，若異動路徑含 `quartz/**`、`quartz.config.ts`、`quartz.layout.ts`、`package.json`、`package-lock.json` 或 `.github/workflows/deploy.yml`，CI 自動建置並部署至 GitHub Pages。

**Vault 筆記變動不觸發 CI**，需手動 `workflow_dispatch`。
