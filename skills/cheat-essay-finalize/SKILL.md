---
name: cheat-essay-finalize
description: 登记一篇文章已完成定稿（排版 + 配图 + 标题确定）。**建 article folder + 询问最终定稿是否与 drafts/<id>.md 一致 + buffer +1**。与 cheat-essay-publish 配对：定稿了进队列，发了出队列。触发词："定稿"/"finalize"/"排版定了"/"已定稿 X"/"准备发了"。
argument-hint: <draft-path-or-id>
allowed-tools: Bash(*), Read, Write, Edit, Glob
---

# /cheat-essay-finalize — 登记定稿完成 + 建 article folder + (改稿则) 触发 v2 预测

把文章从"已写预测、未定稿"状态推进到"已定稿、未发布"状态。这一步：
1. **建 `articles/<同 id>/`** 目录（之前没有的话）
2. **询问用户**："最终排版/标题/配图都定了的稿子和 `drafts/<id>.md` 一致吗？"
3. 算 diff——超过 V2_TRIGGER_THRESHOLD (默认 30%) → **delegate 到 `/cheat-essay-predict — mode: v2`** 在原 prediction 文件 append `## 预测 v2` 段
4. 把 article folder 加进 state.finalized 队列，buffer +1

cheat-essay-finalize 自己**不**写预测内容——所有预测落盘逻辑在 cheat-essay-predict。finalize 只负责检测改稿 + 派发。

为什么单独一个 skill：
- buffer 警戒系统需要明确区分"定稿了" vs "发了"。文章可以批量定稿（一天攒 3 篇排版），分散发（按 cadence 每周/日发 1 篇）
- "最终定稿" ≠ "draft 草稿"是常态——公众号「定时发送」的 24h 窗口里改标题、调段落、改金句几乎是必然的。这一步是把 diff 显式化、触发 v2 重判、采集"用户改稿 pattern"信号的入口
- v2 预测 vs v1 预测的差异本身就是 rubric 升级证据——比如 v1 给 TH=4，v2 给 TH=5（用户改稿改了更锋利的标题），就告诉 rubric "这个用户的 TH 阈值跟我现在公式不一致"
- 公众号场景的"定稿"包含：标题敲定（A/B 备选选一个）/ 摘要写好 / 配图选好 / 排版调好 / 内文校对完成。这五项中任何一项让稿子内容（不仅是表面）发生明显变化都算"改稿"

## Overview

```
[用户：定稿 drafts/2026-05-12_abc123_停止期待.md]
  ↓
[Phase 0: 解析路径 + 验证 prediction 已存在]
  ↓
[Phase 1: 检查是否已登记（避免重复）]
  ↓
[Phase 2: 建 articles/<id>/ + 询问"定稿一致吗？"]
  ↓
[Phase 3: 写 articles/<id>/final.md + (改稿) 触发 v2 重判]
  ↓
[Phase 4: append state.finalized]
  ↓
[Phase 5: 输出 buffer 状态]
```

## Constants

- **REQUIRE_PREDICTION = true** — 定稿前必须先有 v1 prediction 文件
- **V2_TRIGGER_THRESHOLD = 0.30** — 稿子 diff 超过 30%（行级 unified diff 行数 / 原稿子行数）→ 默认建议 v2 重判；低于 30% 询问用户是否仍要 v2
- **DIFF_METRIC = lines** — 用 `diff -u | grep '^[+-]' | wc -l` 算改动行数 / 原文件行数

## Inputs

| 必填 | 来源 |
|---|---|
| `<draft-path-or-id>` | 用户参数；缺失则询问 |
| `.cheat-state.json` | 状态文件 |
| `drafts/*.md` | pre-finalize 草稿 |
| `predictions/*.md` | 验证对应预测存在 |

## Workflow

### Phase 0：解析 + 验证

1. 解析用户给的路径——支持几种形态：
   - 完整路径 `drafts/2026-05-12_abc123_停止期待.md`
   - 简写 `2026-05-12_abc123_停止期待`
   - id 简写 `abc123` → glob `drafts/*_abc123_*.md` 找匹配
2. 验证 `drafts/<id>.md` 存在：不存在 → 报错"找不到 pre-finalize 草稿"
3. 验证有对应 prediction `predictions/<同名>.md`：
   - 不存在 → **拒绝登记**，提示"先跑 /cheat-essay-predict 写预测，否则违反盲预测原则——你不能定稿后才写预测，那等于已经看到了最终标题 + 配图布局再写预测"
   - 存在 → 通过

### Phase 1：检查重复

读 `.cheat-state.json`，检查 `finalized[]` 是否已含此 id：
- 已存在 → 警告"已登记过（X 天前）。是要重新登记，还是要用 /cheat-essay-publish 发布？"
- 不存在 → 进入 Phase 2

### Phase 2：建 article folder + 询问稿子一致性

1. 建目录 `articles/<id>_<short>/`（同 drafts/ + predictions/ 的命名）
2. **询问用户**：

```
定稿 「<title>」 的时候，最终的稿子和 drafts/<id>.md 一致吗？

「最终稿」 = 标题敲定 + 摘要 + 配图 + 排版 + 内文校对 全部完成的版本。

a) 一致——按草稿排版的，标题没换，金句没改
b) 改了一些——你能给我看下最终稿吗？我重新打分一次（v2 预测）
   特别提示：哪怕只是换了标题，TH 维度可能变 +1，建议走 v2
c) 大改了，基本是另一篇 → 走 _redo 流程：
   drafts/<id>_redo.md → 重新 cheat-essay-predict → 再 cheat-essay-finalize
   （原 prediction 留档脱钩）
```

### Phase 3：写 articles/<id>/final.md + (b 路径) 触发 v2 预测

**a 路径（一致）**：
- `cp drafts/<id>.md → articles/<id>/final.md`
- `script_consistency = consistent`
- 不重判，进 Phase 4

**b 路径（改了）**：
1. 询问用户最终稿——粘贴文本 / 文件路径
2. 若用户提供 → 写入 `articles/<id>/final.md`
3. 若用户没保留（直接在公众号后台改的，未保留 md）→ 标 `script_lost`，写占位文件 + 警告"v2 重判跳过——下次建议在公众号后台修改前先 cp 一份草稿到本地"，进 Phase 4
4. 提供了的话：算 diff
   ```bash
   added=$(diff -u drafts/<id>.md articles/<id>/final.md | grep -c '^+')
   removed=$(diff -u drafts/<id>.md articles/<id>/final.md | grep -c '^-')
   total_orig=$(wc -l < drafts/<id>.md)
   diff_pct=$(( (added + removed) * 100 / total_orig ))
   ```
5. **判定 v2 触发**：
   - `diff_pct >= 30` → 默认建议 v2 重判，**主动调用** `/cheat-essay-predict — mode: v2 — prediction-file: predictions/<id>.md` 传 `articles/<id>/final.md` 作 input。cheat-essay-predict 走 v2 模式 append `## 预测 v2`
   - `diff_pct < 30` 但标题变了 → 即使行数 diff 小，标题变化必触发 v2（TH 维度变化对预测影响大）。询问用户："标题变了，TH 维度可能变。要重判吗？默认 yes"
   - `diff_pct < 30` 且标题没变 → 询问用户："只改了 N% 的内容（标题没变），要重判吗？默认不（v1 预测仍有效）"。用户说要 → 同上调用；用户说不 → 跳过 v2，继续 Phase 4
6. cheat-essay-predict 完成 v2 落盘后，控制权回到 cheat-essay-finalize 进 Phase 4

**c 路径（大改）**：
- 不写 `articles/<id>/final.md`，提示走 `_redo` 流程
- 退出 cheat-essay-finalize（不进 Phase 4）

### Phase 4：state 更新

```json
{
  "finalized": [
    ...,
    {
      "article_folder": "articles/2026-05-12_abc123_停止期待/",
      "prediction_file": "predictions/2026-05-12_abc123_停止期待.md",
      "drafts_path": "drafts/2026-05-12_abc123_停止期待.md",
      "finalized_at": "<ISO timestamp>",
      "script_consistency": "consistent" | "modified" | "lost",
      "script_diff_pct": <0-100 int 或 null>,
      "title_changed": <bool>,
      "v2_prediction_written": <true/false>,
      "script_hash_at_finalize": "<sha256:12 of articles/<id>/final.md>"
    }
  ]
}
```

`v2_prediction_written: true` 表示 prediction 文件里现在有 `## 预测 v2` 段，cheat-essay-retro 应读 v2 算偏差；`false` 表示沿用 v1。

`title_changed` 单独记录——因为公众号 TH 权重高，标题变化值得单独追踪。

### Phase 5：输出 buffer 状态

读完 state 后立即算 buffer + 颜色（按 [cadence-protocol.md](../../shared-references/cadence-protocol.md) 的派生规则）：

```
✅ 已登记定稿：articles/2026-05-12_abc123_停止期待/
   预测文件：predictions/2026-05-12_abc123_停止期待.md
   v2 预测：<已生成 / 沿用 v1>

📦 当前 buffer：3 篇（🟢 绿色，正常）
   按你的 cadence（每周更）= 3 周 buffer，节奏稳定。

下一步：定稿其他候选 / 等下个发布日 / 不动
```

如果 buffer 颜色变了（如从绿到蓝）→ 高亮提醒：
```
📦 当前 buffer：6 篇（🔵 蓝色，**积压**）
⚠️  建议暂停定稿，全力发布存货 + 复盘。
   按你的 cadence（日更）= 6 天预备，已超过健康上限。
```

## Key Rules

1. **不写 prediction**——定稿了 ≠ 发了。预测在 /cheat-essay-predict 锁，定稿只是事件
2. **不动 article folder 内容**——final.md / 配图 都不改
3. **必须先有 prediction**——否则违反盲预测（定稿后才写预测 = 已看到最终标题 + 排版决策的事后判断）
4. **buffer 计算实时**——每次 finalize / publish 后立刻重算，state.finalized 是真值
5. **支持批量**：用户可以一天连说 "定稿 X / 定稿 Y / 定稿 Z" 三次连续登记
6. **标题变化是 v2 强信号**——即使行数 diff < 30%，标题变了就强烈建议 v2（TH 维度权重高）

## Refusals

- 「定稿 X，但我从来没跑过 cheat-essay-predict」 → 拒绝。v1 预测**必须定稿前写**——定稿后才写预测会被最终标题 / 配图 / 排版诱导事后修改。请先 /cheat-essay-predict 写 v1 再来 /cheat-essay-finalize。（v2 重判是另一回事——v1 已存在 + 定稿改稿才允许）
- 「我没有 draft，我直接在公众号后台写的」 → 询问用户 → 帮他从公众号后台 copy 一份内容到 drafts/，再走预测 → 再定稿。如果坚持不走完整流程，登记时标 `ad_hoc: true`，但 rubric 校准价值打折
- 「我改稿了但你直接覆盖 v1 吧，别留 v2 段」 → 拒绝。v1 是档案，v2 才是当前判断——append 不覆盖。两段一起留是 rubric 学习的关键证据
- 「定稿了我顺便也算发布了」 → 拒绝。finalize 和 publish 是两件事。即使你已经点了「发送」，请先 finalize 再 publish——buffer 计数依赖这两个动作分开

## Integration

- 上游：`/cheat-essay-predict` 写完 prediction → 用户排版定稿 → `/cheat-essay-finalize` 登记
- 下游：`/cheat-essay-publish` 发布时把对应项从 state.finalized 移除
- `/cheat-essay-status` 看板的 buffer 数字直接来自 `state.finalized.length`
- `/cheat-essay-recommend` 看 buffer 颜色调推荐策略
- SessionStart hook 看 buffer 颜色决定报告第一行

## state.finalized 数据结构

```json
{
  "finalized": [
    {
      "article_folder": "articles/2026-05-12_abc123_停止期待/",
      "prediction_file": "predictions/2026-05-12_abc123_停止期待.md",
      "finalized_at": "2026-05-12T18:30:00+08:00",
      "ad_hoc": false  // true if user finalized without going through full flow
    }
  ]
}
```

按 `finalized_at` 升序——最早定稿的在前面。`/cheat-essay-status` 显示最早一项的 days-since-finalize 警告（避免文章定稿了 7 天没发——议题时效流失）。
