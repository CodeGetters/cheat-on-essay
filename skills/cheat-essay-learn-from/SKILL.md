---
name: cheat-essay-learn-from
description: 从对标账号导入 script + 数据 → 拆 pattern + 派生 base rubric 信号 → 写到 benchmark.md / script_patterns.md / rubric_notes.md。**这是工具最早期信号的来源**——cold-start 用户没自己历史时全靠对标，发过历史的用户也建议至少 1 个对标做 sanity check。触发词："学这个账号"/"拆这几篇对标文章"/"learn from"/"导入对标账号"/"找对标"。
argument-hint: <账号名> [— way: a (default) | b] [— append | --replace]
allowed-tools: Bash(*), Read, Write, Edit, Glob, WebFetch, Skill
---

# /cheat-essay-learn-from — 对标账号导入

工具早期最重要的信号源是**对标账号**——你 init 完没数据，rubric 等权 v0 等于占星。但如果你能找一个你想做成那样的账号，导入 5-10 条它的高/中/低样本，工具就有了 anchor。

后期当你自己 calibration_samples ≥ 10 时，benchmark 影响自然减弱——你的真实数据成为主要信号源。但 benchmark.md **不删**，仍是 cheat-seed brainstorm 的 reference frame。

## Overview

```
[用户：学这个账号 / 启动 cheat-learn-from]
  ↓
[Phase 0: 检查 benchmark 状态]
  ↓
[Phase 1: 选 input 方式（Way a 默认）]
  ↓
[Phase 2: 收集材料]
  Way a: 用户粘 N 条 script 文本 + 数据
  Way b: 用 从 mp.weixin.qq.com 抓 samples/ 目录里的文章正文
  ↓
[Phase 3: 询问每条样本的"印象判断"（高/中/低 + 为啥）]
  ↓
[Phase 4: Claude 拆 pattern + 派生 rubric 信号]
  ↓
[Phase 5: 用户 review → 改 → 落盘]
  ↓
[Phase 6: 写 benchmark.md / script_patterns.md / rubric_notes.md]
  ↓
[Phase 7: 更新 state.benchmark_status]
```

## Constants

- **MIN_SAMPLES = 3** — 最少 3 条样本（少于拆不出 pattern）
- **RECOMMENDED_SAMPLES = 5-10** — 推荐区间，平衡信号量 vs 用户工作量
- **MAX_SAMPLES_PER_RUN = 15** — 单次导入上限——再多 Claude context 不够 + 用户也累
- **DEFAULT_WAY = a** — Way a 简单 + 准确，是 default

## Inputs

| 必填 | 来源 |
|---|---|
| `<账号名>` | 用户参数；缺失则询问 |
| `.cheat-state.json` | 状态文件 |
| Way a: 用户粘的 script 文本 + 数据 | 对话 |
| Way b: `samples/<账号名>/*.mp4` 等文章文件 | 用户提前下载好放进去 |

## Workflow

### Phase 0: 检查 benchmark 状态

读 `.cheat-state.json` 的 `benchmark_status`：

| 状态 | 处理 |
|---|---|
| `none` | 首次导入——继续 Phase 1 |
| `pending` | 用户之前答应等下找——继续 Phase 1 |
| `imported` 已有 benchmark | 询问"你已有 benchmark [当前名]，N 篇样本。要做什么？  a) 追加新文章到当前 benchmark  b) 替换为新 benchmark  c) 只看不改" |

参数解析：
- `--append` → 追加到现有 benchmark
- `--replace <new-name>` → 用新 benchmark 替换（旧的归档到 benchmark.archived/）
- 没标志 + 已有 benchmark → 走上面询问

### Phase 1: 选 input 方式（**两个独立维度**）

每条样本 = **正文** + **数据**。两者怎么拿是独立的——你可以混搭。

#### Phase 1a: 正文 source（怎么拿文章正文）

```
正文怎么拿？

a) **粘文本（最简单，推荐）**
   - 在浏览器打开公众号文章 → 选中正文 → Ctrl/Cmd+C → 粘进对话
   - 大段粘也行——我会自动过滤页眉页脚 / 二维码段 / "我们正在招募伙伴"等套话

b) **给我文章 URL，我抓正文**
   - 你给 https://mp.weixin.qq.com/s/... 形式的 URL
   - 我通过 CDP（Chrome remote-debugging）从公众号页面抓正文
   - 前置：你需要在终端启动调试 Chrome（`open -na "Google Chrome" --args --remote-debugging-port=9222`）
   - 优点：批量给 5 个 URL 一次抓完
   - 缺点：CDP 抓取偶尔受公众号反爬限制（截图、动态加载未触发等），需要回退到 a

c) **跳过正文，只用标题 + 数据 + 印象**
   - 你拿不到正文也懒得抓
   - 后果：pattern 拆不深（只能看标题 / 数据 / 你的印象），但 rubric 信号还行
   - 适合"先快速搭起来，将来补"

回 a / b / c。
```

#### Phase 1b: data source（怎么拿阅读/在看/分享）

```
数据（阅读 / 在看 / 点赞 / 分享 / 涨粉 / 完读率）怎么拿？

a) **手填数字（最简单，公众号场景的默认）**
   - 对标账号文章页面底部可以看到：阅读量、点赞、在看
   - 涨粉率 / 完读率你看不到（除非是自己账号），可以跳过
   - 第三方工具（新榜 / 西瓜）可以看更详细数据，但收费

b) **第三方 API adapter（v0.2 候选，v0.1 暂未实现）**
   - 选 b 自动回退到 a
   - 未来：通过新榜 / 西瓜 API 抓全数据

回 a / b。
```

**最常见的组合**：
- 完全零依赖路径：1a + 1b（粘文本 + 手填）—— 5 分钟搞定 3-5 篇
- 批量加速：1b（URL）+ 1a（手填）—— 你给 URL 我抓正文，数据你粘
- 极简路径：1c + 1a（标题+数据+印象）—— 不需要正文也能跑 rubric 信号

### Phase 2: 收集材料

按 Phase 1a + 1b 的组合走对应路径。

**通用纪律**：每条样本最少要有 (正文 或 抓取结果 或 N/A标记) + 数据 (3-4 项基础：阅读/在看/分享/[完读率可选])。

#### Path A: 粘文本（Phase 1a=a）

```
好。我们一篇一篇来。最少 3 篇，推荐 5-10 篇。

第 1 篇正文，把整段粘下面：
```

收到正文 → 算 article_id（sha256(content)[:12]）→ 进 Phase 2 数据采集。

#### Path B: CDP 抓取（Phase 1a=b）

```
先确认 Chrome 调试模式开了：

[运行 curl -s http://127.0.0.1:9222/json/version 检测]

如果没开：
  ❌ Chrome 调试端口未启动。在终端跑：
     open -na "Google Chrome" --args --remote-debugging-port=9222 --user-data-dir=/tmp/chrome-cdp
  （这会开一个独立 Chrome 实例，不影响你日常浏览器）

开好后给我对标账号文章 URL 列表（每行一条 mp.weixin.qq.com/s/...）：
```

用户给 URL 列表后：
1. 对每个 URL：CDP 新开 tab → `Runtime.evaluate` 抽 `#js_content` 正文 → 关 tab
2. 抓到的正文存为 `samples/<账号名>/<id>/transcript.md`
3. 失败项报告但继续其他
4. 进 Phase 2 数据采集

#### Path C: 跳过正文（Phase 1a=c）

直接进 Phase 2 数据采集——告诉用户"没正文我能拆的 pattern 仅限标题级别 / 你的印象，rubric 信号正常拆"。

#### Phase 2 数据采集（Phase 1b=a 手填）

```
第 1 篇数据：告诉我
- 标题
- 阅读数（公众号文章页底部能看到）
- 在看数（爱心图标旁边）
- 点赞数（拇指图标旁边）
- 分享 / 转发数（如能看到——通常公开页面看不到，仅自己后台能查）

格式随意，能识别就行。例如：
  "标题：怎么停止期待
   阅读：5.2w / 在看 1860 / 点赞 320 / 分享 ?（看不到）"

如果你能再粘 top 5-10 条评论（带赞数）更好——pattern 拆能挖到模因层。
```

收到数据 → 写入 `samples/<账号名>/<id>/meta.md`。

**通用**：继续问第 2 条 / 第 3 条 / ... 用户说"够了"或达到 MAX_SAMPLES_PER_RUN 时进 Phase 3。

### Phase 3: 询问"印象判断"

对每条样本（不管 Way a 或 b），收完数据后**追加问印象**：

```
你看完 / 听完这篇文章的印象，算这个账号的：
  a) 高表现样本（代表作 / 你想做成这样的）
  b) 中表现样本（普通水准 / 不上不下）
  c) 低表现样本（不算这个账号的代表 / 你不想做成这样）

为什么？（一句话——这个判断比数据更能告诉我你想做什么风格）
```

记录 (impression_label, impression_reason) 到内存。

> 注意：印象**可以**和数据冲突——比如某条数据高但用户觉得"不算代表作"。这种冲突本身是有用信号，记录下来。

### Phase 4: Claude 拆 pattern + 派生 rubric 信号

阅读所有 (script, 数据, 印象) → 自己分析：

#### 4a. Script patterns

按 script_patterns.md 的 cheat sheet 框架拆：
- 开头钩子：3 种类型分布（场景代入 / IS 戏仿 / 数据反转）
- 主体结构：几段 / 怎么切
- 句式 / 句长 / 节奏：短句还是长句、有没有标志性句式
- emotional 标记 / 双声道
- 致谢段 / 收尾
- 高频词汇 / 词汇风格

输出 N 个具体 pattern（每个引用具体样本作证据）。

#### 4b. Rubric 信号（**仅定性，不给数值权重**）

对每条样本打 7 维分（用通用维度），然后看：
- 高表现样本（按用户印象）共有哪些维度高？
- 低表现样本共有哪些维度低？
- 哪些维度在高/低样本之间无差异（说明不是关键维度）？

输出**定性方向**（不是数值权重）：
- "ER 看起来重要"（3/3 高样本 ER ≥4）
- "SR 看起来不显著"（高低样本 SR 分布无差异）
- "MS 高的样本评论区有明显模因爆发"

### Phase 5: 用户 review

一次性展示所有结果给用户：

```
我从你给的 N 条对标文章拆出：

📝 N 个 script pattern：
  1. **[Pattern 1 名称]**：[一句话描述] → 证据：[样本 X / Y]
  2. ...
  
🎯 Rubric 定性信号：
  - 看起来重要的维度：ER / QL / MS（每个有 N 条高样本支持）
  - 看起来不显著的维度：SR / NA
  - 给的初始建议：你的对标账号是 [情感+金句驱动型] / [数据驱动型] / [类比讲解型] / ...
  - **不直接给数值权重**——5-10 样本拟合容易过拟合，先用作 tier-2 信号
  
🎨 选题方向感：
  - 主题分布大概：[主题 A 40% / 主题 B 30% / ...]
  - 调性：[一句话]

回 "ok" 我落盘，
或指出哪些 pattern / 信号你不认同（"Pattern X 我觉得不准" / "Rubric 信号 Y 错了"）。
```

用户反馈循环：
- "ok" → Phase 6 落盘
- 用户挑刺 → Claude 改 → 重新展示 → 直至确认

### Phase 6: 落盘

#### 6a. benchmark.md

按 [templates/benchmark.template.md] 格式写到 `<user-channel>/benchmark.md`：
- 账号信息（账号名、URL、调性、粉丝量级——用户提供）
- 导入的样本表
- 基础 rubric 派生（仅定性）
- 选题方向感

如 `--append` → 在现有 benchmark.md 的样本表追加新行 + 重新拆 pattern；不重写整个文件。
如 `--replace` → 把现有 benchmark.md 移到 `benchmark.archived/<旧账号名>_<日期>.md`，写新的。

#### 6b. samples/<账号名>/

为每条样本建子目录：
```
samples/<账号名>/<article-id>/
├── source.mp4 (Way b 才有，Way a 没有)
├── transcript.md (从粘文本写 / CDP 抓取)
└── meta.md (标题 / 数据 / 印象 / 印象理由)
```

#### 6c. script_patterns.md

在 `<user-channel>/script_patterns.md` 加新段：

```markdown
## 对标 [账号名] 借鉴（imported on YYYY-MM-DD，N 篇样本）

> 这些 pattern 来自对标账号——**Imported, untested on my channel**。
> 实拍验证后（≥2 次跑出 + 复盘确认有效）再去掉这个标记，升入正式 pattern。

### Pattern A: [一句话名]
**来自**: [样本 X]
**描述**: [详细]

### Pattern B: ...
```

#### 6d. rubric_notes.md

在 `<user-channel>/rubric_notes.md` 加 / 更新"benchmark-derived initial signals"段：

```markdown
## Benchmark-derived initial signals

> 来自 benchmark.md 的对标账号 [账号名]（N=N 样本，import on YYYY-MM-DD）。
> **仅定性方向，不直接采纳为数值权重**——5-10 样本拟合容易过拟合。
> 等你自己 N≥5 校准样本后正式 bump 时**再决定**是否调权重。

- 看起来重要的维度: ER / QL / ...
- 看起来不显著的维度: SR / NA / ...
- Claude 给的初始建议: [一句话定性]
```

### Phase 7: 更新 state file

```json
{
  "benchmark_status": "imported",
  "benchmark_name": "<账号名>",
  "benchmark_sample_count": <N>
}
```

## Key Rules

1. **Way a 默认**——简单 + 准确。Way b 仅给"懒得粘想批量 URL 抓取"的兜底
2. **必须问印象**——纯看 transcript 拆 pattern 容易抓表面，加用户印象才挖到深层
3. **Rubric 信号仅定性**——不直接给数值权重。5-10 样本拟合过拟合
4. **pattern 默认标 untested**——避免污染用户自己的 pattern 库
5. **不直接下载文章原稿**——下载是用户的事，避免 TOS + 反爬
6. **可重复跑**——`--append` 加新文章，`--replace` 换账号
7. **MIN_SAMPLES = 3**：少于 3 拆不出 pattern，拒绝继续

## Refusals

- 「跳过印象判断，直接拆」 → 拒绝。印象是关键 input
- 「我只能给 1 条样本」 → 拒绝。最少 3 条
- 「直接给我数值权重」 → 拒绝。Phase 4 只给定性信号
- 「能不能不写 transcript 文件，只在内存里拆」 → 不行。transcript 持久化是后续 cheat-retro Phase 4b diff 的依据
- 「帮我下载对标文章」 → 拒绝。引导用户用 yt-dlp / BBDown 等工具自己下

## Integration

- 上游：`/cheat-essay-init` Phase 2.5 在 cold-start 用户时**强烈建议**跑 `/cheat-essay-learn-from`；calibration 用户**可选**
- 上游：`/cheat-essay-status` 在 `benchmark_status=pending` + 距 init >24h 时持续提醒
- 下游：`/cheat-essay-seed` brainstorm 时读 benchmark.md → 知道用户对标方向
- 下游：`script_patterns.md` 加新段，cheat-seed 写 draft 时按 pattern 选结构
- 下游：`rubric_notes.md` 加 benchmark-derived signals 段，cheat-bump 时作为参考之一
- N≥10 提示：`cheat-essay-status` 在用户 calibration_samples ≥10 时提示"你已有足够自己数据，benchmark 影响淡出（保留作 sanity check）"

## benchmark 何时淡出

不是死磕样本数，是 **Claude 判断"用户数据信号是否已超过 benchmark"**：

- **默认参考**：calibration_samples ≥ 10 → benchmark 影响淡出
- **可以更早**：N=5 但用户的 (打分, 实绩) 配对里出现 ≥3 条与 benchmark pattern 不一致的——说明你账号已经走出对标的路径
- **可以更晚**：N=15 但用户的样本都很相似（都做同一类内容），benchmark 仍有信号价值

判断条件 + 默认值都在 cheat-status 触发器 #19 / cheat-seed Phase 0 里实现。

**淡出后**：
- cheat-seed brainstorm 仍读 benchmark.md，但**优先级低于用户自己的 predictions/**
- rubric_notes 的 benchmark signals 段标 `**Status: superseded by user data**`，不删但不再主导
- benchmark.md **不删**——保留作 sanity check（看你账号是否真的偏离对标方向太远）

**任何时候用户主动**：跑 `/cheat-essay-learn-from --replace none` 完全解除 benchmark 影响

## 与其他 skill 的区别

| Skill | 用途 |
|---|---|
| `/cheat-essay-learn-from` | **从对标账号**导入 pattern / rubric 信号（一次性 / 偶尔追加） |
| `/cheat-essay-seed` | brainstorm 选题 + 写 draft（读 benchmark.md 作参考） |
| `/cheat-essay-trends` | 抓今天的热点（与 benchmark 无关） |
| `/cheat-essay-bump` | 升级 rubric（用户自己 N≥5 后用真实数据，不直接用 benchmark signals） |
