# PingAI 插件内容 · 加密分发仓

本仓是 PingAI 插件/技能内容（skill body 等）的**加密分发（CDN）仓库** —— 也是
`CONTENT_CDN_BASE_URL` 默认指向的 `main` 分支 raw 根。

## 里面是什么

只有生成器摊的**密文树**：

```
skills/<id>/content/
  ├── index.json    # 该技能内容包的清单（也是密文）
  └── SKILL.md      # 技能正文（密文）
```

`index.json` / `SKILL.md` 的**内容**一律是 AES-256-GCM 密文（base64），`curve` 如下：

```
加密:  sha256(明文) —— 生成器先算 hash 再加密
       明文正文 --AES-256-GCM(每插件派生 key)--> 密文(base64)  ← 本仓存这个
解密:  客户端下载密文 --AES-256-GCM(相同派生 key)--> 明文
       再 sha256(明文) === 目录 pin 的 content_sha256 → 通过才注入
```

- **每插件一把 key**：由服务端专用 `CONTENT_KEY_SECRET` 按 `pluginId:version` HMAC-SHA256 派生，
  不写 git、不落库、不下内置客户端——客户端只在**订阅门禁通过**时按次拿到。
- **明文正文一律私有保有、绝不进本仓**（详见 `.gitignore`：只有 `skills/<id>/content/**` 能进树）。

## 用途

- 后端在目录响应里下发 `components[].content_url = <CONTENT_CDN_BASE_URL>/skills/<id>/content`。
- 客户端据此从本仓（GitHub raw）拉取密文，本地用订阅门禁下发的 `content_key` 解密、校验 sha、注入宿主。

## 改/发布姿势

明文源在**本地私有目录**维护（`pingai-core/pingai-plugins/skills/<id>/SKILL.md`）。要发布：

```bash
# pingai-backend 下，用与部署后端【同一把】CONTENT_KEY_SECRET
CONTENT_KEY_SECRET=<与后端一致> npx tsx _gen-plugins-sync.mts \
  --repo <明文本地源> --dist <本仓clone根>
# → 摊出 skills/<id>/content/* 密文树到 clone，提交 + 推送 main
```

🔴 顺序：**先发布密文到本仓**，**再**在部署后端开 `CONTENT_CDN_BASE_URL`——否则客户端
GitHub 404 会静默退 `no-index` 本地占位（看似上映、其实没东西可下）。