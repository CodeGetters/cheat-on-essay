---
name: cheat-essay-retro
description: T+N 天数据回收 + 复盘 + 把新观察写入 rubric_notes.md。这是校准循环的反馈环节——不复盘的预测等于占星。**公众号场景默认走 manual paste**：用户从公众号后台抄 6 个数字 + 粘 Top 20 评论。触发词："复盘 [path]"/"retro this"/"T+3d 数据来了"/"抓数据 [path]"/"把这篇复盘了"。
argument-hint: <prediction-file> [— window: 3|5|7] [— source: manual|adapter]
allowed-tools: Bash(*), Read, Edit, Write, Glob, Grep, Skill
---

# /cheat-essay-retro — 数据回收与复盘

抓 T+N 天的实际表现 → 对比预测 → 提炼新观察 → 写入 rubric_notes.md。**只追加 `## 复盘` 段，绝不改预测段**。

## Overview

```
[用户：复盘 predictions/2026-05-12_...]
  ↓
[Phase 0: 校验 immutability + 校验时间窗口]
  ↓
[Phase 1: 抓数据（manual paste 引导 / adapter 占位）]
  ↓
[Phase 2: 写实绩段 + top 评论关键词聚类]
  ↓
[Phase 3: 验证/推翻预测的各假设（含 v0 七维 + 标题 A/B + 发布时段）]
  ↓
[Phase 4: 提炼新观察]
  ↓
[Phase 5: 落盘（追加到 ## 复盘 段）]
  ↓
[Phase 6: 写入 rubric_notes.md 的"观察记录"段 + script_patterns.md]
  ↓
[Phase 7: 检测是否触发 bump 候选 → 提示用户跑 /cheat-essay-bump]
```

## Constants

- **RETRO_WINDOW_DAYS = 3** — 默认 T+3d。公众号长尾比短视频长一些，干货 / 行业号可设 7
- **DATA_SOURCE = manual** — manual（默认）：用户从后台粘数字 + 评论；adapter（v0.2 候选）：调第三方 API
- **AUTO_PROPOSE_BUMP = true** — Claude 判断是否系统性偏差时自动提议 /cheat-essay-bump
  - **默认参考**：连续 ≥3 次同向偏差（high/low）→ 提议
  - **但 Claude 可以更早提议**：1 次极端偏差（如中枢 5w 实绩 0.5w 这种 ≥10x），即使没有"连续"也提议
  - **也可以更晚**：3 次同向但每次偏差都很小（<25%），可能只是噪声不是系统性
- **TOP_COMMENTS_N = 20** — 抓 / 粘 top N 高赞评论
- **REQUIRE_NUMBERS = ["reads", "views_again", "likes", "shares", "new_followers", "completion_rate"]** — 必填 6 个数字（完读率如后台不显示可标 N/A）

> 💡 调用时覆盖：`/cheat-essay-retro <file> — window: 7 — source: adapter`

## Inputs

| 必填 | 来源 |
|---|---|
| `<prediction-file>` 或 `<article-folder>` | 用户参数；缺失则从 `.cheat-state.json` 的 `pending_retros[0]` |
| `rubric_notes.md` | 用户项目根 |
| `.cheat-state.json` | 状态文件 |

### 入参解析（同 cheat-essay-predict 双形态接受）

用户给的可能是：
- **`predictions/2026-05-12_<id>_<short>.md`** → 直接用这个 prediction 文件
- **`articles/2026-05-12_<id>_<short>/`** → 找对应的 prediction 文件（按 id 匹配）+ 把 report.md 写到该 article folder 里
- 缺省 → 从 `pending_retros[0]` 取最早的

## Workflow

### Phase 0: 校验

1. 读 `<prediction-file>`，确认存在
2. **识别有效预测段**：扫所有 `## 预测...` 段（可能含 `## 预测`、`## 预测 v1`、`## 预测 v2` 等）：
   - 取**最后一个**`## 预测 vN` 作为本次校准的依据（v2 存在则用 v2；只有 v1 则用 v1；legacy 单段 `## 预测` 直接用）
   - state.finalized 对应项的 `v2_prediction_written` 应与"是否存在 v2 段"一致——不一致则警告（state 与文件脱节）
3. **校验 immutability**：在内存 cache 住所有 `## 预测...` 段的内容（用于 Phase 5 后核对——**全部段不可改**，不只是有效段）
4. 校验文件 header 有 `Published at` → 没登记的不能复盘，提示用户先 `/cheat-essay-publish`
5. 校验时间窗口：今天 - published_at >= RETRO_WINDOW_DAYS。不够 → 提示"还差 X 天"，询问用户是否仍坚持复盘（标 `early_retro: true`）
6. 校验已有复盘段是否已填——已填则询问"是补充还是修正？"
   - 补充 → 在已有复盘段下追加新子段，标日期
   - 修正预测段（用户错觉）→ 拒绝

### Phase 1: 抓数据（manual paste 引导）

公众号场景下 99% 走 manual——v0.1.0 没有可用的免费 API adapter。

#### Path A：`DATA_SOURCE=manual`（默认）

**Step 1：引导用户到公众号后台找数据**

```
📊 现在去公众号后台抄数据。

打开手机微信 → 「订阅号助手」 → 进你的公众号 → 数据 → 内容分析
或电脑端 → mp.weixin.qq.com → 数据分析 → 内容分析 → 找到这篇文章 → 点详情

需要这 6 个数字（顺序无所谓，能识别就行）：
  ✅ 阅读数（总）
  ✅ 在看数
  ✅ 点赞数
  ✅ 分享次数（转发到朋友/朋友圈/群聊的总次数）
  ✅ 涨粉数（24h 或 48h，看你的发布时间）
  🟡 完读率（"阅读完成率"——后台可见就给，看不到标 N/A）

粘格式随意，例如：
  阅读 5.2w 在看 1860 点赞 320 分享 580 涨粉 47 完读 38%
  或
  reads: 52000, views_again: 1860, likes: 320, shares: 580, new_followers: 47, completion: 38%
```

**Step 2：强制要求 top 20 评论**

```
🗨️ 接下来粘 Top 20 评论（带赞数）。

为什么必须粘评论：
  - "分享率 8%" 不能告诉你为什么火，20 条评论关键词能
  - "她不一样" 这种模因爆发只能从评论看出
  - 公众号「精选留言」其实就是你的免费焦点小组

从公众号后台 → 留言管理 → 按赞数排序 → 复制前 20 条。
或者公众号文章页底部「精选留言」直接复制。

格式：每行一条，带赞数，例如：
  「我也是这样」（1200 赞）
  「不能再赞同了，第三段直接破防」（840 赞）
  ...
```

用户拒绝 / 给少于 5 条 → **拒绝继续**：

```
评论才是真信号——没评论的复盘 = 看体温计判断病情。

请粘 top 20 给我。如果实在拿不到（评论被关 / 是别人账号 / 已删 等），
告诉我原因，我帮你标 `comments_unavailable`，但这次复盘价值打折。
```

**Step 3：写到 article folder 的 report.md**

```markdown
# Report — <article 标题>

**复盘时间**: 2026-05-15T10:00:00+08:00
**数据回收 T+N**: T+3d
**数据来源**: manual paste from 公众号后台

## 数字
- 阅读：52,000
- 在看：1,860 (3.58%)
- 点赞：320 (0.62%)
- 分享：580 (1.12%)
- 涨粉：47 (0.09%)
- 完读率：38%

## Top 20 评论（带赞数）
| 排名 | 评论 | 赞数 |
|---|---|---|
| 1 | 我也是这样 | 1200 |
| ... | ... | ... |
```

#### Path B：`DATA_SOURCE=adapter`（v0.2 候选，v0.1 自动回退）

v0.1.0 暂不实现。如果用户配置 `data_collection: adapter`：
- 提示："adapter 在 v0.1 未实现。回退到 manual paste。"
- 走 Path A

未来 v0.2 候选 adapters：
- 新榜 API（按次收费 0.02-0.2 元）
- 西瓜助手 API
- RPA / Playwright 模拟登录创作者中心（高维护成本）

任何 adapter 失败 → 优雅降级到 manual。

### Phase 2: 写实绩段 + top 评论分析

**实绩数据格式**（参考 [prediction-anatomy.md](../../shared-references/prediction-anatomy.md) 的复盘段格式）：

```markdown
### 实绩数据
- 阅读：5.2w（落在 `1w-10w` 桶内中位，相对中枢 3w **+73%**）
- 在看：1860（在看率 3.58%，**优秀**——基准 1.5%）
- 点赞：320（点赞率 0.62%）
- 分享：580（分享率 1.12%，**中等**——基准 3%）
- 涨粉：47（涨粉率 0.09%，**中等**——基准 0.5%）
- 完读率：38%（**偏低**——平台权重 10%）
```

每个比率必须算出来 + 标对比基准（中等 / 优秀 / 顶级，参考 wechat-essay-zero.md 的辅助验证维度表）。这些比率是单纯阅读数无法暴露的真信号。

**top 评论关键词聚类**：
- 把粘进来的 N 条评论分 3-5 类（高赞模因 / 概念引用 / 自我代入"我也是" / 议题展开 / 转发暴露暗示 / 离题噪声 等）
- 每类列代表性评论（带赞数）
- 报告比例（"35% 是 EG 自我代入、22% 是议题展开、18% 是金句引用"）
- **特别留意**：评论里有没有出现"我转发了"/"我截图发朋友圈了"等转发暴露——这是 QD 维度的强证据

### Phase 3: 验证/推翻

对 prediction 文件里的每一项（7 维评分、标题 A/B、发布时段、推理因素、关键校准假设、反事实场景），逐项判定。

**特别针对公众号场景**——除了 7 维评分，还要验证：

#### 3a. 7 维评分验证

```markdown
### 7 维评分验证

**验证 ✅**:
- EG=5 被验证：35% 评论是"我也是"类自我代入，是 EG 颗粒度强的强证据
- AE=5 被验证：留言区有大量"留言告诉我那段对话的最后一句话"模因回应——AE 对话邀请生效
- 关键校准假设完全成立：本篇 5.2w / 上篇 1.8w = 2.9x，超我押的 1.5x

**推翻 ❌**:
- 中枢 3w 被超出 +73%
- TH=4 可能低估：阅读 5.2w 远超中枢预期，标题分享暴露暗示比预想强
- 反事实推理里"如果 <1w 推翻 IE=3 起点"——实际落到 1w-10w 偏上，IE=3 起点正确但 TH 低估
```

#### 3b. 标题 A/B 验证（**公众号独有**）

```markdown
### 标题 A/B 验证

预测时 3 个候选：
- 「停止期待，是我对他最后的温柔」TH=4，被推荐
- 「他给的不是爱，是配合你期待演的戏」TH=5，强烈推荐
- 「我们之间的关系，本质是 OOO」TH=2，不推荐

用户最终选了：候选 #2（推荐采纳）

实际表现：阅读 5.2w + 朋友圈/群分享率 1.12% + 涨粉 47

判定：✅ 推荐准确
  - 候选 #2 的 TH=5 与实际朋友圈传播力匹配
  - 评论里 12% 出现"刚转发了你这条"提示——TH 的分享暴露驱动强
  - 推荐 vs 用户选择一致 → rubric TH 维度信号正确

如果用户选了 #1 或 #3（不推荐）但跑得很好 → 反 pattern，写到 rubric_notes.md「未来观察」
```

#### 3c. 发布时段验证（**公众号独有**）

```markdown
### 发布时段验证

预测时推荐：晚 20-22（personal-narrative + EG 颗粒度强）
用户实际发：晚 21:30（推荐采纳）

实际表现：完读率 38%（偏低）

判定：⚠️ 时段推荐部分准确
  - 在看率 3.58% 优秀 → 睡前情绪开放期推断正确
  - 完读率 38% 偏低 → 可能与稿子 2200 字偏长有关，不是时段问题
  - 时段 Confidence 维持 🟡（v0 起步表）→ 还需更多样本校准

如果用户选了凌晨 / 工作时间但跑得很好 → 反 pattern，可能用户的读者画像与默认表不同 → 提示在 init 配置 reader_profile
```

**关键纪律**：
- 每条验证 / 推翻必须引用具体数据（"在看率 3.58%"），不许写"基本符合"这种含糊措辞
- 反事实的"如果落在 X bucket 意味着什么"——实际落在的那个 bucket 直接告诉你哪个 rubric 假设被测试了，明确写出来

### Phase 4: 提炼新观察（**三类，分别写入对应文件**）

#### 4a. Rubric 观察（写入 rubric_notes.md）

打分维度 / 公式 / bucket 边界相关的观察：

```markdown
### 需要写进 rubric_notes.md 的新观察

1. **TH 在含「分享暴露暗示」标题的真实权重应 ≥ ×1.8**：候选 #2 的 5w 实绩 vs 候选 #1 同题预期约 2-3w，证据强
2. **完读率与字数强相关**：2200 字 38% 完读 vs 历史 1500 字 56% 完读 → 字数权重纳入 SR 维度
3. **公众号头部账号读者跟人不跟文（待验证）**：检查工具作者预埋观察清单的 PI 维度——本篇账号粉丝量 ?w，需对比无名账号同议题文章的转发率
4. ……
```

每条观察必须可追溯到具体数据点（不写"标题很重要"——写"候选 #2 TH=5 vs #1 TH=4 的预期流量差是关键校准假设的 2.9x"）。

**如果触发了「未来观察清单」中的某条**（rubric_notes.md template 里 cheat-essay-init 写的预埋清单）→ 显式标注"☑ 触发预埋观察 #N（教程 IE 失真 / AE 模板压平 / QD 锚点 / 热点借势 / 人格信号 / 形态边界）"——这是 rubric 第一次 bump 的关键证据。

#### 4b. 写作 Pattern 观察（写入 script_patterns.md）

Diff `drafts/<id>.md`（pre-finalize 草稿，可能是 cheat-essay-seed 写或用户写）vs `articles/<id>/final.md`（实际定稿——cheat-essay-finalize 时用户提供的版本），找出**改动且对流量有明显影响**的部分：

| 用户做了什么 | 流量影响 | 是否提议追加 pattern |
|---|---|---|
| 砍掉某段 | 实绩 ≥ 中枢 → "砍掉没伤流量"——验证那段冗余 | 是，加到 script_patterns.md "用户改稿历史观察"表 |
| 换了标题（v2 触发） | 实绩超中枢 → 新标题更锋利 | 是，候选 Pattern N，标 ≥1 样本待验证 |
| 改了开头（加具体场景） | 完读率提升 vs 历史 | 是，候选 Pattern N |
| 加了配图 | 在看率显著上升 | 是，候选 Pattern N |
| 没动结构 / 改动与流量无关 | — | 不追加 |

输出格式：

```markdown
### 需要写进 script_patterns.md 的新 pattern 候选

1. **用户改稿模式**: 砍掉 [X 段] / 改了标题 [原标题→新标题]
   - 流量影响：实绩 [N] vs 中枢 [M]，[偏差 / 命中]
   - 建议：追加到 script_patterns.md 的"用户改稿历史观察"表

2. **新 pattern 候选 N**：[一句话描述]
   - 单样本支持
   - 触发条件：[何时该用]
   - 建议：追加到 script_patterns.md 末尾的"新发现的 Pattern"段，标 ≥1 样本待验证
```

询问用户："要把这些追加到 script_patterns.md 吗？(yes / no / 选择哪几条)"。**用户确认后才追加**——避免把单点观察直接写成正式 pattern。

> **rubric 进化 ≠ 写作进化**——两者解耦：
> - rubric_notes.md 学的是"哪些维度真的预测流量"
> - script_patterns.md 学的是"什么写法真的能起作用"
> 可能有交叉（如 EG 维度与"具体场景"pattern），但记录在两个文件里是因为**作用域不同**——rubric 改了影响所有未来打分，pattern 改了影响所有未来 draft。

如果 `articles/<id>/final.md` **缺失**（cheat-essay-finalize 时用户标 `script_lost`） → 跳过 4b，没法 diff。
如果 `script_consistency = "consistent"`（用户定稿没改）→ 4b 仍然有意义（diff 也许是空），但可以快速跳过细查。
如果 `script_consistency = "modified"`（用户定稿改了）→ **4b 是核心**，重点学这次改动 → 流量影响。

#### 4c. 标题 / 时段 / 评论模因专项观察（**公众号独有**）

```markdown
### 公众号专项观察

1. **标题分享暴露驱动**：候选 #2 的标题在转发到朋友圈时不暴露读者的真实处境（"演的戏"是评论别人，不是承认自己）→ 这是公众号文章 vs 视频脚本的关键差异，TH 维度的子信号
2. **评论模因爆发观察**：本次"我截图发朋友圈了"出现 12 次（带赞数）→ QD 维度的强证据，金句被高频引用
3. **完读率与发布时段的交互**：晚档发但完读率偏低 → 可能"睡前接收情绪开放"和"睡前注意力衰减"是两件事，未来观察记录
```

### Phase 5: 落盘到 ## 复盘 段

用 Edit 工具，**仅追加**到现有 `## 复盘` 段（如有占位 `（待填）` 行先删除）：

```markdown
## 复盘

**复盘时间**: 2026-05-15（发布 T+3d）
**抓取时间**: 2026-05-15 10:00
**数据来源**: manual paste from 公众号后台

### 实绩数据
[Phase 2 内容]

### Top 评论关键词
[Phase 2 内容]

### 7 维评分验证
[Phase 3a 内容]

### 标题 A/B 验证
[Phase 3b 内容]

### 发布时段验证
[Phase 3c 内容]

### 需要写进 rubric_notes.md 的新观察
[Phase 4a 内容]

### 公众号专项观察
[Phase 4c 内容]
```

**写完后再次校验**：读取保存后的文件，对比**所有** `## 预测...` 段（v1 / v2 / legacy）的合并哈希应等于 Phase 0 cache 的合并哈希。**任一段被改 → 报错并回滚**。

### Phase 6: 写入 rubric_notes.md + script_patterns.md

#### 6a. rubric_notes.md（Phase 4a 的输出）

按 [observation-lifecycle.md](../../shared-references/observation-lifecycle.md) 的"观察记录模板"格式，追加到 `rubric_notes.md` 的 `## 观察记录` 段：

```markdown
### YYYY-MM-DD [标题简称] (id) — [一句话定性]
- 预测：composite=X.XX，bucket=Y
- 实绩：阅读 / 在看 / 点赞 / 分享 / 涨粉 / 完读率（带 T+Nd 标注）
- Top 评论关键词：[简短摘录 + 赞数]
- 判断：哪个维度被验证/推翻？为什么？
- 标题 A/B：用户选哪个 / 推荐对不对
- 发布时段：实际选哪个 / 推荐对不对
- 触发预埋观察：☐ 教程 IE / ☐ AE 模板 / ☐ QD 锚点 / ☐ 热点借势 / ☐ 人格信号 / ☐ 形态边界
- Rubric 调整：[如果有，写明 "下次打 XX 类文章时改 YY"]
- 详见：[predictions/<file>.md]
```

**检测跨样本 pattern**：扫描已有"观察记录"，看新观察是否与某条已有观察形成 ≥2 样本支持。命中则按 [observation-lifecycle.md](../../shared-references/observation-lifecycle.md) 升级到"重大跨文章观察"段。

#### 6b. script_patterns.md（Phase 4b 的输出，**用户确认后才写**）

如 Phase 4b 用户回 "yes" 或选择性确认了某几条：
- "用户改稿模式" → 追加到 script_patterns.md 的"用户改稿历史观察"表
- "新 pattern 候选 N" → 追加到末尾"新发现的 Pattern"段，**显式标 ≥1 样本待验证**

新 pattern 候选的格式（同 [script_patterns.template.md](../../templates/script_patterns.template.md) 的 Pattern 11/12 示例）：

```markdown
### Pattern N（来自 [文章简称]，单样本待验证）

**现象**：[Phase 4b 描述]

**原理**：[为什么有效——基于这一次观察的猜测]

**触发条件**：[何时该用]

**待验证**：需要 ≥2 样本支持才能升正式 pattern。
```

跨样本 pattern 升正式：扫描"新发现的 Pattern"段，看是否有 ≥2 样本支持同一现象 → 升到核心 pattern 库 + 删 "待验证" 标记。

如用户在 Phase 4b 全否（"no"）→ 跳过 6b，rubric_notes.md 仍照写。

### Phase 7: 检测 bump 触发

读 `.cheat-state.json` 的 `consecutive_directional_errors` 字段，按本次复盘判定向更新：
- 本次预测高估（实绩 < 中枢 -25%） → push `["high"]` + 记录 deviation_magnitude（如 0.5x / 0.3x）
- 本次预测低估（实绩 > 中枢 +25%） → push `["low"]` + 记录 deviation_magnitude
- 在 ±25% 内 → 不 push

**Claude 判断是否提议 bump**（不是固定门槛）：

```
判断维度：
1. 连续同向次数（参考默认：≥3）
2. 单次偏差幅度（参考默认：>2x 或 <0.5x 算极端）
3. 偏差是否能解释为单一维度漏判（如 TH 或 EG 一致偏离）
4. 用户是否在复盘里反复提到同一现象
5. 是否触发了预埋观察清单中的同一条（≥2 次触发同一条 → 强信号）

任一足够强 → 提议 bump：
- 3 次连续同向，每次都中等偏差 → 提议
- 1 次极端偏差（如 ≥10x），即使没连续 → 提议（"一次性强信号"）
- 2 次同向 + 评论区出现一致的反向证据 → 提议（"评论 + 数据双信号"）
- 同一条预埋观察清单被 ≥2 次触发 → 提议（"预埋观察成立"）

不提议的情况：
- 3 次同向但每次都很小（<25%）→ 可能只是噪声
- 偏差跨多个维度无清晰方向 → bump 不知道改什么
```

提议时输出：

```
🚨 检测到 [系统性偏差信号] / [极端单点偏差] / [预埋观察成立] 。

[简短描述：连续 N 次 / 1 次极端 / 评论双信号 / 第 N 次触发预埋观察 X 等]

这可能是 rubric 系统性偏差的信号。建议：
- 跑 /cheat-essay-bump 看是否需要升级公式
- 或先看 /cheat-essay-status 详细分析

注：本次提议是 [default-aligned: 满足 ≥3 同向] / [judgment-driven: 1 次 10x 强偏差] / [pre-embedded: 触发预埋观察清单 #N]
```

更新 state file：
```json
{
  "calibration_samples": <+1>,
  "pending_retros": [<剔除本次>],
  "last_retro_at": "<ISO>",
  "consecutive_directional_errors": [...]
}
```

## Key Rules

1. **预测段 immutable**。Phase 0 cache + Phase 5 校验是双保险。任何 hash 不一致 → 报错回滚
2. **数据来源必须标注**。`数据来源: manual paste from 公众号后台` 或 `数据来源: adapter:newrank`（v0.2+）写进复盘段
3. **观察可追溯**。每条新观察引用具体数据点
4. **不在复盘里 bump**。Phase 7 只**提议** bump，实际升级走 `/cheat-essay-bump`——避免一次操作做两件事
5. **早复盘标记**。RETRO_WINDOW_DAYS 不到就复盘 → state file 记 `early_retro: true`，bump 时这种样本权重降级
6. **必填 6 个数字**。少于 6 个 + 评论少于 5 条 → 拒绝继续（除非用户给明确不可得理由）
7. **必查预埋观察清单**。每次 retro 都要核对 rubric_notes.md 的「工具作者预埋的未来观察清单」段是否被触发——这是 v0.1.0 收集 rubric 缺陷证据的关键

## Refusals

- 「这条数据已经看过了，但你假装没看，按预测时的盲度做复盘」 → 复盘本来就是看完数据再做的；这个表述本身没有违规，但要确认用户没在 prediction 写之前透露过数据
- 「把预测段的概率分布改一下，让复盘看起来更准」 → 拒绝。原则 #1
- 「跳过观察提炼，直接结束」 → 拒绝。新观察是 rubric 进化的唯一燃料；缺它复盘退化为"看一眼"
- 「直接 bump，不要单独走 /cheat-essay-bump」 → 拒绝。bump 流程有完整的跨模型审 + cleanup pass，retro 是触发器不是执行器
- 「我只能给你阅读数，其他数据都看不到」 → 询问原因：是看不到（公众号后台权限问题 / 别人账号 / 已删等）还是懒得抄。看不到 → 允许标 `partial_data: true` 但复盘价值降级；懒得抄 → 拒绝
- 「评论也粘不了，公众号没人评论」 → 允许，标 `comments_unavailable: zero_comments`——本身就是有用信号（评论 0 条说明 EG 颗粒度或 AE 出口出了问题）

## Integration

- 前置：`/cheat-essay-publish` 已登记 + 时间窗口达到
- 下游：累计 `consecutive_directional_errors` 满 3（或单次极端偏差 / 预埋观察成立）→ 触发 `/cheat-essay-bump` 提议
- 状态字段更新：`calibration_samples` +1（这是 cheat-essay-status 显示进度的关键）
- pending_retros：剔除本条
- 与 [observation-lifecycle.md](../../shared-references/observation-lifecycle.md) 紧耦合：每次复盘是观察新增的入口
- 与 rubric_notes.md 的预埋观察清单紧耦合：retro 是触发清单检查的入口
