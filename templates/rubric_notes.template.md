# 评分校准笔记

> **本文件是 cheat-on-essay 评分规则进化的载体**。每次复盘实际阅读数据 vs 预测分数后，把判断依据和规律显式写在这里，下次打分前 `/cheat-essay-score` `/cheat-essay-predict` 会先读这个文件再动手。
>
> **核心原则**：规律必须可追溯到具体样本。不写"议题锋度很重要"这种空话，要写"XX 这篇 IE=5 得到验证 / 推翻，因为评论区 top 3 都是 YY 模式"。
>
> 完整生命周期协议见 [shared-references/observation-lifecycle.md](../shared-references/observation-lifecycle.md)。
> 升级流程见 [shared-references/bump-validation-protocol.md](../shared-references/bump-validation-protocol.md)。

---

## Rubric 版本日志

_结构性变更才 bump 版本号；纯观察积累不算。升版后，校准池里的样本必须用新公式重打分。每次升级写一份结构化 evidence memo（见下方各版本 section）。_

**当前版本**: `v0`

**版本速查表**:

| 版本 | 生效日期 | 变更类型 | 驱动样本数 | 驱动 article_ids |
|---|---|---|---|---|
| v0 | [YOUR-INIT-DATE] | 初版占位（cold-start） | 0（先验） | — |

**升级决策原则**:
- 纯权重微调（如 TH×1.5 → ×1.8）→ 不 bump，trigger 重算 composite
- 维度定义细化（如 IE=5 的门槛变严）→ 不 bump，但复盘时标注新门槛
- 新增/删除维度、或定义颠覆性改写 → bump 主版本号

**迁移触发**: 候选筛选时如遇旧版打分的文章进入 top → 当场重读重评；不做全量重评。**校准池（带实绩数据）必须在每次升级时全量重打**。

---

## 当前评分维度 (0-5)

> **起步说明：**
> Cold-start 用户用 v0 等权占位——见 [starter-rubrics/wechat-essay-zero.md](../starter-rubrics/wechat-essay-zero.md)。
> 校准 5 篇之后再决定要不要把表里的等权换成你自己拟合的版本。
> 通用经验版（带权重的起点）见 [starter-rubrics/wechat-essay.md](../starter-rubrics/wechat-essay.md)。

| 维度 | 权重（v0 等权） | 含义 | 平台权重对应 |
|---|---|---|---|
| title_hook (TH) | 1.0 | 标题钩子力 | 打开率（40%） |
| opening_pull (OP) | 1.0 | 开头留人 | 完读率前段（10%） |
| issue_edge (IE) | 1.0 | 议题锋度 | 互动率（30%） |
| emotional_granularity (EG) | 1.0 | 情感颗粒 | 分享率（20%） |
| structural_rhythm (SR) | 1.0 | 结构节奏 | 完读率底层 |
| quote_density (QD) | 1.0 | 金句密度 | 朋友圈二次传播 |
| action_exit (AE) | 1.0 | 行动出口 | 涨粉转化 + 互动 |

**v0 等权综合分公式**：

```
composite = (TH + OP + IE + EG + SR + QD + AE) / 7 × 2.0
```

**通用经验版的差异化权重**（v0 → v1 升级候选起点）：

```
composite = (TH×1.5 + OP×1.2 + IE×1.5 + EG×1.2 + SR + QD + AE) / 8.4 × 2.0
```

---

## 观察记录

> **模板**（每次复盘后追加一条）：
>
> ```
> ### YYYY-MM-DD [标题简称] (id) — [一句话定性，如"验证 IE 主导"]
> - 预测：composite=X.XX，bucket=Y
> - 实绩：阅读 / 在看 / 分享 / 涨粉 / 完读率（带 T+Nd 标注）
> - Top 评论关键词：[简短摘录 + 赞数]
> - 判断：哪个维度被验证 / 推翻？为什么？
> - Rubric 调整：[如果有，写明 "下次打 XX 类文章时改 YY"]
> - 详见：[predictions/<file>.md]
> ```
>
> 删除规则见 [shared-references/observation-lifecycle.md](../shared-references/observation-lifecycle.md)：被吸收为维度 → 删；被推翻 → 删。git history 是档案。

（待第一次复盘后追加；cold-start 期请只追加真实条目，不要复制示例）

---

## 重大跨文章观察（≥2 样本支持但需要更多验证）

> 单样本观察先放在"观察记录"段。≥2 样本同 pattern 才升格到这里。

（暂无——开始记录后会自动累积）

---

## 规律沉淀区（高置信度，打分前必看）

> 每条规律要有 ≥2 样本支持 + 已通过升级验证流程（即被吸收为维度或显式确认）。

（暂无——升级 1-2 次后会有内容）

---

## Benchmark-derived initial signals

> 由 [benchmark.md](benchmark.md) 派生（如有），表示对标账号的高/中/低样本里**哪些维度看起来重要**。
>
> **仅定性方向，不直接采纳为数值权重**——5-10 样本拟合容易过拟合。
> 等你自己 N≥5 校准样本后正式 bump 时再决定是否调权重。
>
> 初始为空——`/cheat-essay-learn-from` 完成后会填这里。

（待 cheat-essay-learn-from 填入）

---

## 待验证假设

> 单样本观察 + 强信号但还没复现的，暂存这里。

- [ ] [示例] 议题型文章 IE=5 > 情感型文章 EG=5 在「在看率」上（等下一篇议题稿发完看）
- [ ] [示例] AE=5 但用标准三连模板 vs AE=5 用对话邀请，涨粉率差几倍

---

## 工具作者预埋的「未来观察」清单（v0.1.0 起）

> 这些是 cheat-on-essay 工具开发时用 5 篇真实样本（议题深度 / 教程实操 / 新闻热点 / 短消息）校准 v0 rubric 后留下的 caveat。
> **它们不是规律也不是被验证的观察**——是"v0 rubric 已知会在某些场景失真"的明牌，等你跑出真实数据来验证或推翻。
> 跑完 5 篇后，请逐条勾选：✅ 被你的数据验证 / ❌ 被推翻 / 🟡 还不够样本判断。
> 详细推导见 [docs/sample-calibration-v0.1.0.md](../docs/sample-calibration-v0.1.0.md)（工具仓库内）。

- [ ] **IE 维度对教程实操类文章会失真**——教程文核心维度是「实操可复用度」「场景刚需度」「完成度」，v0 rubric 没纳入。如果你常写教程类，复盘时 IE 强行打分应标 N/A，看其他 6 维能否解释实际表现
- [ ] **AE 维度被「点赞+在看+转发+星标」标准模板压平**——头部账号普遍用这套，区分度不足。未来 AE 应细化为「行动出口类型」：标准模板 / 对话邀请 / 共同仪式
- [ ] **QD 锚点「密度 OR 分布」需要明确为 AND**——「每千字 ≥2 句」「至少 3 段位有金句」两个条件目前不清楚是 OR 还是 AND，应是 AND
- [ ] **rubric 不抓「热点借势力」**——新闻热点向文章传播 70% 来自时效 + 大号粉丝盘 + 推荐流加权，rubric 无对应维度。如果你常写新闻类，复盘时 composite 与实际表现可能弱相关
- [ ] **rubric 不抓「人格信号」**——头部账号读者跟人不跟文，「这是 XX 写的」可能压过 7 维的所有信号。如果你已有粉丝盘 > 1w，看 IE+EG 之外是否需要补 PI（personal identity）维度
- [ ] **rubric 适用形态边界**——v0 设计目标是 1k-5k 字议题/观点类，不适用短消息卡片（< 500 字）、图集主导文章、纯营销软文。复盘前先确认你这篇形态是否适用

---

## 被拒升级 log

> 提议过但未通过验证的 bump，记录在这里——避免半年后重复提相同的失败方案。

（暂无）

---

## Bucket 方案（**当前: ratio**）

> ⚠️ **bucket 边界是用户账号的属性，不是普适常量**——绝对数桶（"5w 阅读是底部"）只对有粉丝基础的老手成立，对 0 粉新人会让所有文章都落"底部 99%"，bucket 失去排序意义。
>
> 本工具按校准阶段切换三种 bucket 方案。当前生效方案由 `.cheat-state.json` 的 `bucket_scheme` 字段决定。

### 阶段 1：cold-start，比率桶（当前阶段）

`bucket_scheme = "ratio"`

**第 1 篇**：用平台通用默认（实际阅读数）

| Bucket | 范围（实际阅读）| 含义 | 先验概率 |
|---|---|---|---|
| 底部 | < 100 | 几乎没人看见 | 35% |
| 基础盘 | 100 - 1,000 | 朋友圈 + 订阅推送的常态 | 45% |
| 命中 | 1,000 - 10,000 | 第一次被推荐流打开 | 15% |
| 小爆 | 10,000 - 100,000 | 极罕见的零粉首爆 | 4% |
| 10万+ | > 100,000 | 微信算法异常加权 | 1% |

**第 2 篇起**：`baseline = 上一篇实际阅读数`（或最近 3 篇中位数）

| Bucket | 倍数范围 | 含义 |
|---|---|---|
| 退步 | < 0.3 × baseline | 比上一篇明显差 |
| 持平 | 0.3 - 1 × baseline | 与上一篇同档 |
| 命中 | 1 - 3 × baseline | 中度突破 |
| 小爆 | 3 - 10 × baseline | 显著破圈 |
| 大爆 | > 10 × baseline | 量级跃迁 |

详见 [starter-rubrics/wechat-essay-zero.md](../starter-rubrics/wechat-essay-zero.md) 的"Bucket 预测"段。

### 阶段 2：N=5 后切到固定绝对桶（带 ratio 备用）

`bucket_scheme = "absolute_with_ratio"`

跑完 5 篇后，`/cheat-essay-bump --bucket-only` 自动派生：
- `baseline = 5 篇实际阅读的中位数`
- 边界 = baseline × {0.3 / 1 / 3 / 10 / 30}

`/cheat-essay-bump --bucket-only` 落地时会替换本段表格。

### 阶段 3：N≥10 后切到 percentile 桶（推荐长期方案）

`bucket_scheme = "percentile"`

边界 = 你历史样本的 percentile：
- 底部 = bottom 30%
- 基础盘 = 30-60%
- 命中 = 60-85%
- 小爆 = 85-95%
- 大爆 = top 5%

`/cheat-essay-status` 在 N=10 时主动建议切换。这种方案永远自洽——不管账号多大，"top 5%"语义稳定。

---

## 默认复盘窗口

`RETRO_WINDOW_DAYS = 3`

为什么 3 天：微信「看一看」+ 推荐流分发决策一般在 72 小时内基本结束；等更久只引入噪声不增信号。

公众号的「分享 → 朋友圈打开 → 二次传播」长尾比抖音长一些，但 T+3d 已经能捕捉 80% 信号。如果你的账号长尾特别明显（如行业号 / 干货号），可以在 init 时设 `RETRO_WINDOW_DAYS = 7`。

