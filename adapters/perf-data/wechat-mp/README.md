# Adapter: wechat-mp（公众号后台数据抓取）

> **状态：v0.2 候选，v0.1 未实现。** 当前 cheat-essay-retro 默认走 manual paste（用户从后台抄 6 个数字 + 粘评论）。

---

## 规划方向

公众号创作者中心（mp.weixin.qq.com）是强登录态 + 无公开 API 的平台。可行方案按优先级：

### 方案 A：Playwright + 持久化 session（类似 douyin-session）

- 用户首次扫码登录 mp.weixin.qq.com
- Cookie 存在 `.auth/`，后续复用
- 拦截 XHR 抓取文章数据（阅读/在看/分享/完读率/评论）
- **优点**：全自动，数据最全
- **缺点**：维护成本高（微信改版频繁）、Playwright + Chromium 体积大（~500MB）

### 方案 B：第三方数据平台 API

- 新榜 API（按次收费 0.02-0.2 元/次）
- 西瓜数据 API
- **优点**：稳定、无需维护浏览器
- **缺点**：收费、数据可能有延迟、评论抓不全

### 方案 C：RPA 辅助（半自动）

- 用户打开后台页面，adapter 通过 CDP 读取已打开页面的数据
- **优点**：不需要存 cookie、用户可见操作
- **缺点**：需要用户配合打开页面

---

## 当前替代方案

v0.1 用户走 manual paste：

1. 打开公众号后台 → 数据分析 → 内容分析 → 找到文章 → 点详情
2. 抄 6 个数字（阅读/在看/点赞/分享/涨粉/完读率）
3. 粘 Top 20 精选评论

耗时约 2-3 分钟/篇，对低频发布（周更/双周更）完全可接受。

---

## 实现条件

当以下任一条件满足时考虑实现：

- 用户发布频率 ≥ 日更，manual paste 成为瓶颈
- 有稳定的第三方 API 且成本可控
- 微信开放平台提供官方数据接口（目前无）

## 文件清单（待实现）

```
adapters/perf-data/wechat-mp/
├── README.md           # 本文件
├── requirements.txt    # (待定)
├── crawler.py          # (待定)
├── renderer.py         # (待定)
└── run.sh              # cheat-essay-retro 调用的 wrapper
```
