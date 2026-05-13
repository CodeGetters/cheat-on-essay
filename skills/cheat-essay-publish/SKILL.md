---
name: cheat-essay-publish
description: 登记一篇内容已发布，把 URL/平台 ID/发布时间写入对应预测文件 header 和 state file。这是一个轻量动作——只更新元数据，**不动预测段任何字符**。触发词："已发布"/"I shipped"/"发布链接是 X"/"刚发完 [url]"/"publish registered"。
argument-hint: <prediction-file-or-url>
allowed-tools: Bash(*), Read, Edit, Glob
---

# /cheat-essay-publish — 发布登记

把文章的发布元数据（URL、发布时间）补到预测文件 header 与 state file。**禁止改预测段**——hook 会拦。

## Overview

```
[用户：已发布 https://mp.weixin.qq.com/s/...]
  ↓
[Phase 0: 找到对应的预测文件]   ← 通过 in_progress_session 或匹配
  ↓
[Phase 1: 解析 URL → 提取 biz/mid/idx 等公众号 ID]
  ↓
[Phase 2: 更新 prediction 文件 header（仅 metadata 段）]
  ↓
[Phase 3: 更新 .cheat-state.json，清除 in_progress_session]
```

## Constants

- **VERIFY_BLIND = true** — 提醒用户：从此刻起看到任何后续数据都会破坏盲度声明的诚信

## Inputs

| 必填 | 来源 |
|---|---|
| `<prediction-file>` 或 URL | 用户参数；缺失则用 `.cheat-state.json` 的 `in_progress_session.file` |
| `.cheat-state.json` | 用户项目根 |

## Workflow

### Phase 0: 找到对应的预测文件

按优先级：
1. 用户参数明确给了 prediction 文件路径 → 用它
2. 用户参数只给了 URL → 读 `.cheat-state.json` 的 `in_progress_session.file`
3. 都没有 → 列出 `predictions/*.md` 中 header 没填 `published_at` 的文件，让用户选

**警告路径**：若 `in_progress_session.file` 与用户给的 URL 时间差超过 14 天 → 提示"这个预测写于很久之前，确认是这篇？"

### Phase 1: 解析公众号 URL

公众号文章 URL 格式：`https://mp.weixin.qq.com/s/<token>` 或 `https://mp.weixin.qq.com/s?__biz=X&mid=Y&idx=Z&...`

提取的 platform-specific ID：
- **短链格式** `mp.weixin.qq.com/s/<token>`：直接用 token 作为 `wechat_short_id`
- **长链格式** `mp.weixin.qq.com/s?__biz=...&mid=...&idx=...`：提取 biz / mid / idx 三段
- **微信视频号** `channels.weixin.qq.com/*`：超出本工具范围，提示用户「视频号请用 cheat-on-content（视频版）」

非公众号 URL → 警告"本工具针对公众号设计。你确认这是公众号文章 URL 吗？"，允许用户强制继续（标 `platform: other`）。

发布时间获取：
- 不强求自动抓——公众号文章的发布时间在页面 meta 里但抓取受限
- 询问用户："发布时间是？（默认：现在）" → 接受 ISO 8601 或自然语言（"今天 14:30" / "20 分钟前"）
- 解析失败 → 用 now()

### Phase 2: 更新 prediction 文件 header

**绝不**触碰 `## 预测` 段及之后。只动文件最顶部的 metadata 块。

读文件，定位到 metadata 块（在第一个 `##` 之前的所有行）。检查是否已有这些字段——有则警告"已登记过"并询问是否覆盖；无则追加：

```markdown
**Published at**: 2026-05-12T14:32:00+08:00
**Platform**: wechat
**URL**: https://mp.weixin.qq.com/s/UxehkgUhDMwtTibeQiaXLg
**Article Folder**: articles/2026-05-12_a3f2c1d4_停止期待/
**WeChat Short ID**: UxehkgUhDMwtTibeQiaXLg
```

如果用户给的是长链（含 biz / mid / idx）→ 额外存：
```markdown
**WeChat Biz**: MzI4MTUxNTI0MA==
**WeChat Mid**: 2247532119
**WeChat Idx**: 1
```

**article folder 处理**：到 cheat-essay-publish 这一步，对应的 `articles/<id>/` 目录**应该已经由 cheat-essay-finalize 创建**（含 final.md）。

- 如 article folder 不存在 → 警告"你跳过了 cheat-essay-finalize？建议先跑 finalize 把定稿登记进 article folder 再发"，**询问用户是否跳过登记直接发**：
  - 是 → 自动建一个 article folder（fallback），但不询问稿子一致性，标 `ad_hoc_publish: true`
  - 否 → 让用户先跑 finalize 再回来 publish

用 Edit 工具（不是 Write 重写整个文件）。

**hook 行为预期**：因为只动 metadata 段（在 `## 预测` 之前），immutability hook 应放行。如果 hook 误拦 → 报告 bug，**不要绕过 hook**。

### Phase 3: 更新 state file

```json
{
  "in_progress_session": null,
  "last_published_at": "<ISO timestamp>",
  "last_published_file": "predictions/<filename>",
  "last_published_article_folder": "articles/<...>/",
  "last_published_short_id": "<wechat short token 或 mid 复合 ID>",
  "pending_retros": [
    "predictions/<filename>"
  ],
  "finalized": [
    // 移除 article_folder 匹配本次发布的项；buffer -1
  ]
}
```

**`finalized` 队列处理**（buffer 跟踪关键）：
1. 读 state.finalized[]
2. 找 `article_folder == 本次发布的 article_folder` 的项 → 移除
3. 如果没找到 → 警告"buffer 队列里没有这篇文章。是直接发布没经过 /cheat-essay-finalize 吗？"——不阻塞，但提示用户下次走 /cheat-essay-finalize 让 buffer 跟踪准确

`last_published_short_id` 是 cheat-essay-retro 调 adapter 时的潜在输入（v0.1 manual paste 用不到，留位给未来第三方 API adapter）。

`pending_retros` 是待复盘列表——`cheat-essay-status` 会基于这个列表 + RETRO_WINDOW_DAYS 显示"今天该复盘哪些"。

### Phase 4: 提醒 + 下一步 + buffer 状态

```
✅ 登记完成：predictions/2026-05-12_a3f2c1d4_停止期待.md
   - Published at: 2026-05-12 14:32
   - Platform: wechat
   - URL: https://mp.weixin.qq.com/s/UxehkgUhDMwtTibeQiaXLg

📦 Buffer：N 篇（颜色 + 含义）
   按你的 cadence（X）= N×X 天 buffer
   [如颜色变了，提示"现在该去定稿/暂停定稿"]

⚠️  从此刻起，你看到任何关于这篇文章的阅读/在看/分享数据
    都会破坏盲度声明的诚信。如果不小心看到，告诉我——
    我会在文件里追加一个 integrity warning。

📅 计划复盘：T+3d，约 2026-05-15
   到时间说："复盘 predictions/2026-05-12_..."
   届时按 cheat-essay-retro 引导，去公众号后台抄 6 个数字 + Top 20 评论。
```

Buffer 颜色由 [shared-references/cadence-protocol.md](../../shared-references/cadence-protocol.md) 派生。如本次发布让 buffer 跌入红色（断更风险）→ 高亮警告"今天必须再定稿 ≥1 篇"。

## Key Rules

1. **不动预测段**。即使是修复笔误，也不允许在 publish 时改预测段
2. **不抓数据**。publish 是登记动作，不是数据回收（那是 cheat-essay-retro 的活）
3. **state 字段名固定**。`pending_retros` / `last_published_at` 是其他子 skill（特别是 cheat-essay-status / cheat-essay-retro）依赖的契约
4. **平台默认 wechat**。v0.1.0 本工具只为公众号设计——非公众号 URL 需用户明示强制继续
5. **重复登记需明示**。已有 published_at → 询问"覆盖？"，绝不静默覆盖

## Refusals

- 「我顺手把预测段也改一下」 → 拒绝。请走 `_redo.md` 路径
- 「URL 我等会儿补，先把发布时间记上」 → 允许：URL 字段可后续追加；published_at + platform 必填
- 「跳过 metadata 更新，直接清 in_progress_session」 → 拒绝。元数据是复盘时的关键上下文
- 「我把视频号文章也登记进来」 → 拒绝。视频号是视频形态，本工具是公众号长图文设计——视频号文章请用 cheat-on-content（视频版）

## Integration

- 上游：`/cheat-essay-predict`（写出 prediction 文件并设 in_progress_session）+ `/cheat-essay-finalize`（建 article folder + buffer +1）
- 下游：T+RETRO_WINDOW_DAYS 后 → `/cheat-essay-retro`
- `cheat-essay-status` 用 `pending_retros` 字段计算"今天该复盘哪些"
- 平台字段被 `cheat-essay-retro` 用来路由到对应的 perf-data adapter（v0.1 仅 manual-paste）
