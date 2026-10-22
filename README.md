# fe-shared — 三站前端共享代码单一真相源

numeriduel.com / boardduel.com / memoryduel.com 三个前端仓 `public/shared/` 的
**跨站契约文件**以此仓为唯一真相源，替代 2026-10-22 之前的三仓手工复制
（同一逻辑最多 3 份拷贝、各修各的 bug 不回流）。

## 机制

```
eyetoolkit/fe-shared (本仓, shared/*)
        │  raw.githubusercontent.com 免鉴权拉取
        ▼
各站 npm run sync:shared ──► public/shared/ (运行时就地加载, 形态不变)
        │
        ▼
build-gate CI: npm run sync:shared:check (diff 即红)
```

- **sync 脚本**：各前端仓 `scripts/sync-shared.mjs`（三仓同款），按
  `manifest.json` 里本站清单拉取并写入 `public/shared/`
- **CI 门禁**：三仓 build-gate.yml 在构建前跑 `sync:shared --check`——
  本地与真相源不一致即失败，提示跑 `npm run sync:shared`
- **漂移双保险**：tri-sites 的 `check-shared-drift.mjs`（每日 04:00 UTC）
  继续做三站互比，验证 sync 真的在各站生效

## 修改共享文件的正确流程

1. 在本仓改 `shared/` 下文件，commit + push main
2. 三个前端仓各跑 `npm run sync:shared`，各自 commit push
3. CI 的 `sync:shared:check` 会验证各仓确实同步了

**不要**直接改各站 `public/shared/` 里的契约文件——下次 sync 会被覆盖，
CI check 也会红。

## 文件清单（manifest.json 为准）

| 文件 | boardduel | memoryduel | mathduel | 说明 |
|---|---|---|---|---|
| sso-bridge.js | ✓ | ✓ | — | 跨站登录唯一通道（STRICT） |
| identity.js | ✓ | ✓ | ✓ | 昵称/身份，三站统一版 762c0560（STRICT） |
| site-header.js | ✓ | ✓ | — | 全局页头逻辑（STRICT）；导航数据在各自的 site-header.nav.js |
| auth-extras.js | ✓ | ✓ | — | 登录态附加组件（STRICT） |
| icons.svg | ✓ | ✓ | — | 图标 sprite（STRICT） |
| README.md | ✓ | ✓ | — | .tpl 模板维护文档 |
| *.tpl ×4 | ✓ | — | — | robots/ai.txt/404/manifest 占位符模板（sed 生成各站文件） |

**不进本仓的**（站点定制或单站文件，各有其主）：
- `site-header.nav.js`——每站导航数据不同（boardduel 8 款 / memoryduel 3 项）
- `auth-modal.js`——memoryduel 版有 md_session 游客凭证等站点定制（OBSERVE）
- `theme.js` / `daily-client.js` / `duel-room.js` / `share-card.js`——memoryduel 单站
- `qrcode.min.js`——mathduel 单站 vendor 库
- `account.js` / `analytics.js` / `achievements.js` 等——2026-10-22 阶段1 已删（零引用死文件）

## 历史

- 2026-10-22 建仓（shared 收敛阶段4，架构报告第 10 项收官）
- 初始内容取自三仓已一致文件：identity.js 三站统一版、site-header.js
  board/memory 统一版、其余 STRICT 文件
