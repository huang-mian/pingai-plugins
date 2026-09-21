# pingai-plugins

PingAI 的 plugin / skill / MCP **内容资产仓库**（私有）。

> 这是「内容资产」仓库，**不放工程代码**。MCP 网关服务、后端接口、admin 管理页等
> 代码在 `pingai-core`，不进本仓库。

## 两个区（边界约定）

| | ① 自研区 `skills/` + `mcp/` | ② 第三方区 `mirror/` |
|---|---|---|
| 内容 | PingAI 自己的 skill / MCP | 第三方开源仓库镜像 |
| 接分发？ | ✅ 是，同步进 `app.plugins` 表 | ❌ 否，纯备份防失效 |
| 维护 | 运营 / 内容团队直接编辑 | 脚本定期 pull 上游 release |
| 授权 | PingAI 自有 | 保留上游 license |
| 更新 | 编辑 + 升版本号 | 定期 fetch |

## 目录结构

```
.
├── skills/          # ① 自研 skill（内容源）
│   └── <skill-id>/
│       ├── skill.md       # 提示词 / 工作流主体
│       └── manifest.json  # id / 名称 / 描述 / 分类 / 版本 / install_cmd
├── mcp/             # ① 自研 / 托管 MCP 定义（若走 C 方向）
│   └── <mcp-id>/
│       ├── server.json    # transport / url / config
│       └── manifest.json
├── mirror/          # ② 第三方开源备份（纯镜像，不接分发）
│   └── <upstream-org>-<repo>/
├── catalog/
│   └── catalog.json       # 自研区编译产物（脚本生成，勿手改）
└── scripts/
    └── sync-to-db.ts      # 仓库 → app.plugins 表的同步脚本
```

## 分发链路

```
自研区（skills/ + mcp/，唯一真相源）
   └─ scripts/sync-to-db.ts 编译 → catalog/catalog.json
        └─ 迁移/seed 灌进 app.plugins 表
             └─ 后端 GET /v1/me/plugins 读表分发 → 桌面端写 ~/.claude.json
```

> 当前后端目录是 `pingai-backend/src/v1/me/plugins.ts` 里的**硬编码 mock**。
> 迁表第一步就是把那 18 条自研 IG skill + 4 条 MCP 从 mock 搬进本仓库的
> `skills/` / `mcp/`，让 mock 有个真实归宿。

## MCP 定位（未拍板）

MCP 定位见 `pingai-core/docs/mcp-positioning.md`（待拍板，内部三种分裂说法）。
本仓库结构先按「内容源」设计，MCP 的 `server.json` 字段等定位定了再补。
