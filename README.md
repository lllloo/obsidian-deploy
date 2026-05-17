# obsidian-deploy

Quartz 4 部署框架，將 [obsidian-memory](https://github.com/lllloo/obsidian-memory) vault 發佈至 [ob.bugloop.com](https://ob.bugloop.com)。

Vault 筆記本體、skills、稽核腳本均由 obsidian-memory 管理，本 repo 只負責 Quartz 設定與 CI/CD。

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
npm run test                # 執行所有測試（tsx --test）
npm run vault:check         # 稽核 content/ frontmatter 與檔名（需先 clone vault）
npm run vault:fix           # 稽核並自動修正（需先 clone vault）
```

## 發佈

push 到 `main` 後，若異動路徑含 `quartz/**`、`quartz.config.ts`、`quartz.layout.ts`、`package.json`、`package-lock.json` 或 `.github/workflows/deploy.yml`，CI 自動建置並部署至 GitHub Pages。

**Vault 筆記變動不觸發 CI**，需手動 `workflow_dispatch`。

## Web Clipper 模板

`.clipper/vault-clipper.json` 是 [Obsidian Web Clipper](https://obsidian.md/clipper) 的 template 匯入檔。匯入方式：擴充套件 → Settings → Templates → Import → 選此檔。修改後 commit 以跨機器同步。
