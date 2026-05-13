# [文章标题] — 预测日志

> **本模板由 `/cheat-essay-predict` 自动填充**。
> 所有预测都用统一格式（7 个核心组件 + 标题 A/B 段 + 发布时段 metadata + 复盘段）——前 5 篇和成熟期都一样，区别仅是 header 里的 confidence 等级。
>
> 完整规范见 [shared-references/prediction-anatomy.md](../shared-references/prediction-anatomy.md)。
> 公众号独有两段协议见：[title-ab-protocol.md](../shared-references/title-ab-protocol.md) + [publish-timing-protocol.md](../shared-references/publish-timing-protocol.md)。

---

**Article ID**: `<12 位 hash>`  ← e.g. ab61ed09f0a1（对 `drafts/<id>.md` 首次落盘内容做 sha256，取前 12）
**Title**: `<完整标题>`（用户在 Phase 2.5 标题 A/B 中最终选定的标题）
**Rubric Version**: **`<v0/v1/v2/...>`**
**Skill Package**: `cheat-on-essay@<version>`
**Content Form**: `<state.content_form>`  ← e.g. `opinion-essay` / `personal-narrative` / ...
**预测时间**: `<YYYY-MM-DD>`（基于最终稿）
**Script Path**: `drafts/<YYYY-MM-DD>_<id>_<short>.md`
**Script Hash**: `sha256:<12 位>` (predict 时 hash draft 内容；cheat-essay-finalize 时再 hash，不一致则复盘段加 integrity warning)
**Word Count**: `<N>`  ← e.g. 2200 字
**Typical Word Count (state)**: `<state.typical_word_count>`  ← e.g. 2200（用户 init 时设的目标字数）
**Calibration Samples (at predict time)**: `<state.calibration_samples>`  ← e.g. 3
**Confidence**: `<emoji + 标签>`  ← e.g. 🟡 偏低 (中枢 ±40%，可作为参考之一)。从 calibration_samples 派生，见 state-management.md
**Prediction Basis**: `pre_finalize` | `post_finalize_pre_publish`  ← v1 默认 pre_finalize；v2 由 cheat-essay-finalize 触发 = post_finalize_pre_publish
**Scored By**: `claude` | `claude+user_override`  ← Claude 自打分；如用户在 review 阶段挑刺改了字段，标 `+user_override`
**User Override**: `<如有覆盖，列出哪些字段被改了+原值与新值>` | `none`
  ← 例：`IE: claude=4 → user=3 (用户认为 '议题不够锋利')` `中枢: claude=5w → user=3w`
  ← 复盘时这个字段帮诊断：用户哪个维度直觉跟 Claude 系统性偏离，被实绩验证 → rubric 可能漏了什么
**Publish Timing**: 推荐 `<时段>` / 用户计划 `<时段>`  ← e.g. 推荐晚 20-22 / 用户计划晚 21:30
**Timing Rubric**: `<推荐理由一句话>`  ← e.g. content_form=personal-narrative + EG 颗粒度强 → 晚档情绪开放期匹配
**Timing Confidence**: `<emoji>`  ← 🟢 已校准 / 🟡 v0 起步 / 🔴 用户选反向时段
**预测时数据状态**: **blind**（未看任何阅读/在看/点赞/分享数据）

---

## 输入快照

**7 维分 (v0 等权)**: `<TH=X / OP=X / IE=X / EG=X / SR=X / QD=X / AE=X>` → composite=**`<X.XX>`**

> 示例：TH=4 / OP=5 / IE=3 / EG=5 / SR=4 / QD=4 / AE=5 → composite=**8.57**（v0 等权 / 7 × 2.0）

**用户改写要点 vs Claude 草稿**（如有）:
- **开头**：[user 砍掉了什么 / 加了什么]
- **砍掉**：[具体段落 / 概念名 / 铺垫]
- **保留**：[关键的金句 / 致谢段 / 主体结构]
- **节奏**：比草稿 [紧 / 松] 约 N%

> 如用户从零写没用 cheat-essay-seed，写"用户原创稿，无 Claude 草稿对照"。

---

## 标题 A/B 评分

**候选数**: `<N>`（推荐 2-3 个候选；只给 1 个标 `single_candidate: true`）
**选择者**: user（最终选择由用户做，工具仅推荐）

| 候选 | TH 评分 | 优势 | 短板 | 推荐? |
|---|---|---|---|---|
| `<标题 1>` | `<1-5>` | `<一句话>` | `<一句话>` | `<推荐 / 强烈推荐 / 不推荐>` |
| `<标题 2>` | `<1-5>` | `<一句话>` | `<一句话>` | `<推荐 / 强烈推荐 / 不推荐>` |
| `<标题 3>` | `<1-5>` | `<一句话>` | `<一句话>` | `<推荐 / 强烈推荐 / 不推荐>` |

**推荐**: 候选 #`<X>`（评分 `<Y>`，`<一句话理由>`）

**用户最终选择**: 候选 #`<X>` / 「`<最终标题>`」

> 标 `clickbait_risk: high` 如选了 TH=5 但其他维度普遍低 — 复盘时记得检查推荐流是否被算法压
> 详见 [title-ab-protocol.md](../shared-references/title-ab-protocol.md)

---

## 预测 v1

> ⚠️ **本段是 immutable**——`hooks/prediction-immutability.sh` 会拦截对本段的 Edit。
> 写完不可改。如要重做请创建 `<本文件名>_redo.md`，原文件保留。

**Bucket**: `<X-Yk/w>`  ← e.g. `1w-10w`

**内心概率分布**（公众号阅读 5 档）:
- `<1k` → X%
- `1k-1w` → X%
- **`<headline bucket>` → X%**（中枢 ~`<X>w`）
- `1w-10w` 或 `10w-100w` → X%
- `10w+` → X%

> 加起来必须 100%。
> Confidence 低（calibration_samples 少）时**应该更平**（如 30/30/20/15/5），不是更尖（如 5/40/45/8/2）——诚实地反映不确定。

**辅助预测**（区间，不是点估计）:
- 在看率：`<X-Y%>`（推断 IE 与 EG 综合）
- 分享率：`<X-Y%>`（推断 QD 与 EG 颗粒度）
- 涨粉率：`<X-Y%>`（推断 AE 与 全文综合质量）

**一句话 reason**:
> [核心驱动因素 + 最强反例约束 + 中枢预测]

---

## 推理因素

| 因素 | 方向 | 置信度 | 说明 |
|---|---|---|---|
| `<dim or feature>` | 强 + / 中 + / 弱 ? / 强 - | 高 / 中 / 低 | [≤30 字理由] |

> 置信度三档：高（强证据 + 多锚点支持）、中（有理由但样本少）、低（凭直觉）。复盘时如果"低置信度"因素被验证 → 直觉强；"高置信度"因素被推翻 → rubric 有 bug。

---

## 锚点对比

| 对照样本 | composite | 实绩阅读 | 异同 |
|---|---|---|---|
| `<样本名>` | `<X.XX>` | `<X>` | [关键差异维度] |

> **校准池不够时**（< 2 个 composite ±0.5 邻近样本）：
> 写"校准池只有 N 个样本，无 composite 邻近样本。**锚点对比 N/A**——注意本次预测 confidence 是 🟡 偏低 / 🔴 极低，bucket 中枢仅供参考。"
> **不要直接删这段**——告诉读者锚点为何缺，比静默跳过诚实。
>
> **筛锚点优先按字数**：本篇 `<X>` 字，优先找 `<X*0.7>` - `<X*1.3>` 字范围的样本，避免跨字数得出虚假结论。

---

## 反事实场景（复盘用）

**如果爆 `10w+`**（X% 预期）:
- [验证什么假设——通常是 TH+EG 双 5 或 QD 主导]
- [推翻什么假设——如 IE=3 实际低估]
- [可能新增什么 rubric 维度——如热点借势 HM]

**如果落在 `headline bucket`**（X% 预期）:
- [基准线验证什么]

**如果跌到 `<1k`**（X% 预期）:
- [推翻什么核心判断——通常是 TH 过度乐观或议题悬空]

**如果 `<100`**（X% 预期）:
- [极端场景的可能解释——账号断流 / 算法限流 / 标题党触发降权]

> 实际落在哪个 bucket → 告诉你 rubric 的哪个假设被测试。**不可省略**。

---

## 关键校准假设

[把这次预测当成一次实验，明确写下"如果 X 发生，证明 Y"]

[找一个对照样本（最好是上一篇预测）]

两篇 [同 composite / 邻近 composite]，差异：
- 本篇：[关键维度对比]
- 对照：[关键维度对比]

**我押**：[本篇 vs 对照 = X 倍 / 高 N 阅读]

如果反过来 → [推翻什么 rubric 假设]
如果差距 < N → [rubric 基本 OK / 噪声范围]

> **校准池只有 0-1 篇时**：写"无可对照样本——但仍写下我对这次的核心赌注（即使没有锚点）："然后写 1-2 条这次想测的事。

---

## 复盘

> ⚠️ **以下段落由 `/cheat-essay-retro` 在 T+`RETRO_WINDOW_DAYS` 天后追加**。
> hook 允许追加本段；不允许改预测段任何字符。

（待填——T+RETRO_WINDOW_DAYS 天后跑 `/cheat-essay-retro <对应 article folder>`）
