# adapters/trend-sources/wechat-hot.md — 公众号看一看 / 第三方聚合热门

**状态：experimental（v0.1.0）—— 公众号没有公开热门 API，本 adapter 是一个占位 + 实现建议**。

---

## 目标

抓"公众号生态的当前热门文章"，输出 candidate items 进 candidates.md。

## 为什么需要这个 adapter

- 微博热搜 / 知乎热榜的热点跟你的公众号场景**不完全对应**——你的读者可能不刷微博但订阅了某些公号
- 公众号「看一看 → 朋友♡」其实是你最直接的同温层信号——可惜微信不开放数据
- 第三方聚合（新榜 / 微小宝 / 西瓜助手）覆盖大量公众号文章 + 周期性更新热门榜

## 数据源候选（按可行性排序）

### 1. 第三方商业 API（推荐生产用，需自付费）

| 服务 | 接口 | 价格 |
|---|---|---|
| 新榜 | `api.newrank.cn`，按次 0.02-0.2 元 | 商业方案 |
| 西瓜助手 | 类似 | 商业方案 |
| 飞瓜 | 类似 | 商业方案 |

**配置方式**（用户自接）：
- 把 API key 放在 `.cheat-secrets.json`（gitignore）
- 实现一个抓函数返回符合 [candidate-schema.md](../../shared-references/candidate-schema.md) 的 items
- 调用接口取「最近 24h 公众号文章 top N」

### 2. 公众号「看一看精选」（你自己的）

如果你愿意手动定期截图 / 复制看一看推荐流：
- 走 `manual-paste` adapter，不要走这个 wechat-hot
- 命中率高（是你账号生态的真实信号）但需要人工时间

### 3. 公开聚合站点（experimental，稳定性低）

- 一些公开站点（如 toutiao / 嗅探类聚合）会汇总各平台热门文章
- 这些站点本身没有官方 API，需要 WebFetch 解析 HTML
- 稳定性低，站点改版必失败——**不推荐生产依赖**

## v0.1.0 实现状态

⚠️ **未实现 fetch 函数**。如果你启用了 `wechat-hot` 但 adapter 未实现：
- `/cheat-essay-trends` 会优雅降级，输出 "wechat-hot: 跳过（adapter 未实现，回退到 manual-paste）"
- 不阻塞整个抓取流程

## 用户自定义实现指引

如果你想接入自己的第三方 API：

1. 在 `adapters/trend-sources/` 下复制本文件为 `<你的源>.md`
2. 实现 fetch 逻辑（Python / Node / Bash 都行），输出 items 数组
3. 每个 item 必须含字段：`title` / `url` / `snapshot_text` / `source_type` / `snapshot_at`（ISO 8601）
4. 写入 `.cheat-state.json` 的 `enabled_trend_sources` 数组
5. 跑 `/cheat-essay-trends --sources <你的源>` 验证

## 失败模式 + 降级

| 错误 | 处理 |
|---|---|
| API key 缺失 | skip，提示"配置 API key 见 wechat-hot.md" |
| API 配额耗尽 | skip，附配额恢复时间 |
| 网络超时 | retry 1 次，仍失败 skip |
| 返回格式变化 | 报告"schema 不匹配，请更新 adapter" |

## 稳定性等级

★★☆☆☆（v0.1.0 占位，未真实校准）

未来路线：
- v0.2 候选：内置一个第三方 API 的官方支持 adapter（如新榜）
- v0.3 候选：尝试解析公开聚合站点（接受高维护成本）
