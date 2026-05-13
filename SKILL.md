---
name: cheat-on-essay
description: 给所有想把"感觉"变成可校准预测的微信公众号作者。**方法论沿用 cheat-on-content**——打分 → 盲预测 → T+3d 复盘 → 进化 rubric 的循环。**rubric 是循环的内容，不是循环本身**——内置一份公众号文章 v0 等权 rubric（7 维：TH/OP/IE/EG/SR/QD/AE），用户后续靠 cheat-essay-bump 调权重。**强烈建议导入对标账号**作为初始信号源（/cheat-essay-learn-from）。触发词："初始化"/"打分这篇"/"启动预测"/"定稿"/"已发布"/"复盘"/"升级 rubric"/"推荐选题"/"抓热点"/"状态"/"找对标"/"learn from"。**首次使用必须先跑 /cheat-essay-init。**
argument-hint: [draft-path] [— mode: cold-start|calibration]
allowed-tools: Bash(*), Read, Write, Edit, Grep, Glob, Skill, mcp__llm-chat__chat
---

# 公众号作弊器 / Cheat on Essay

> 🎯 **方法论 = cheat-on-content，平台 = 微信公众号文章**
>
> **5 阶段闭环**：写 → 打分 → 盲预测 → 排版定稿（含定稿改稿 v2 重判）→ 发布 → T+3d 复盘 → 进化 rubric。
>
> **rubric 形态**：公众号长图文（一般 1k-3k 字），7 个维度 v0 等权——
> TH（标题钩子）/ OP（开头留人）/ IE（议题锋度）/ EG（情感颗粒）/ SR（结构节奏）/ QD（金句密度）/ AE（行动出口）。
>
> 默认假设：**用户是从零开始的新人**（一篇都没发过）——cold-start 期的预测会简化，
> 只要 7 维打分 + 一句话 bet，不强求 bucket 数字（避免 false precision）。
> 已有 5+ 篇数据的老手走 calibration 模式解锁完整 7 组件预测。

把公众号文章创作变成可校准预测循环：**打分 → 预测 → 定稿 → 发布 → 复盘 → 进化 rubric**。

本文件是**总协议 + 路由器**。具体每个阶段的工作流在 `skills/cheat-essay-*/SKILL.md` 各子 skill 里。

---

## Codex compatibility

Codex 没有 Claude Code 的 slash-command harness。安装到 Codex 后，按自然语言触发同一套路由即可：

- `初始化 cheat-on-essay` → 读取并执行 `skills/cheat-essay-init/SKILL.md`
- `打分这篇 drafts/foo.md` → 读取并执行 `skills/cheat-essay-score/SKILL.md`
- `启动预测 drafts/foo.md` → 读取并执行 `skills/cheat-essay-predict/SKILL.md`
- `定稿 ...` / `已发布 ...` / `复盘 ...` / `升级 rubric` / `状态` → 分别读取对应 `skills/cheat-essay-*/SKILL.md`

执行时遵循本文件的三条原则和路由表；不要依赖 `/cheat-essay-*` 命令是否存在。Claude Code 专用 hook（`.claude/settings.json`）仍只在 Claude Code 里自动触发；Codex 中需要用户主动说 `状态` 查看 buffer、待复盘和候选池。

---

## 三条不可妥协原则（继承自 cheat-on-content，不修改）

任何一条被违反，整个校准循环退化为"凭直觉的自我安慰"。如果用户要求打破其中任何一条，**拒绝执行并说明原因**。

1. **盲预测（Blind prediction）**：预测必须在看到任何实际数据**之前**写完。一旦写完，`## 预测 vN` 段是 immutable——只能往 `## 复盘` 段追加。完整规范：[shared-references/blind-prediction-protocol.md](shared-references/blind-prediction-protocol.md)。**hooks/prediction-immutability.sh 在 harness 层强制执行**。

2. **升级 = 全量重打（Bump = full re-score）**：rubric 升级时，校准池所有有实绩数据的样本必须用新公式重打分；新排序与实际表现排序若在 ≥4/5 样本上不一致，升级被拒；升级必须经跨模型独立审核。完整规范：[shared-references/bump-validation-protocol.md](shared-references/bump-validation-protocol.md)。

3. **rubric 是工作台，不是博物馆**：被新数据推翻或被吸收为正式维度的观察，**删掉**。绝不留"我曾经以为 X，但其实..."的考古层。git history 才是档案。完整规范：[shared-references/observation-lifecycle.md](shared-references/observation-lifecycle.md)。

---

## 路由表（触发词 → 子 skill）

| 用户说 | 调用 | 前置条件 |
|---|---|---|
| "初始化" / "init" / "首次使用" | `/cheat-essay-init` | 无（这是入口） |
| "找对标" / "学这个账号" / "拆这几个对标公号" / "learn from" / "导入对标账号" | `/cheat-essay-learn-from` | 已 init；cold-start 强烈建议；后续可随时 --append / --replace |
| "找选题" / "我不知道写什么" / "seed" / "找前 5 个选题" | `/cheat-essay-seed` | 已 init（cold-start 用户专用一次性种子动作） |
| "打分这篇 [path]" / "score this [path]" | `/cheat-essay-score` | rubric_notes.md 存在 |
| "抽金句" / "找金句" / "挑可截图的句子" / "quote candidates" | `/cheat-essay-quotes` | draft 文件存在；可在任何阶段调，不必经 |
| "启动预测" / "start prediction" / "给这稿子打分并预测" | `/cheat-essay-predict` | 已 init + 有最终稿 |
| "定稿" / "排版定了" / "finalize" / "定稿改了" | `/cheat-essay-finalize` | 对应预测已写（buffer +1，改稿 ≥30% 触发 v2 重判） |
| "已发布" / "I shipped it" / "发布链接是 X" | `/cheat-essay-publish` | 对应预测文件存在（buffer -1） |
| "复盘" / "retro this" / "T+3d 数据来了" | `/cheat-essay-retro` | 对应预测文件存在 + 已过 RETRO_WINDOW_DAYS |
| "升级 rubric" / "bump rubric" / "更新公式" | `/cheat-essay-bump` | 校准池 ≥ MIN_SAMPLES_FOR_BUMP |
| "推荐选题" / "next topic" | `/cheat-essay-recommend` | candidates.md 存在且非空 |
| "抓热点" / "fetch trends" / "今天有什么可做的" | `/cheat-essay-trends` | trend-sources adapter 已配置（日常补充候选池） |
| "状态" / "status" / "看板" | `/cheat-essay-status` | 任意时刻可调 |
| "迁移" / "升级 state" / "schema 版本不对" / "migrate" | `/cheat-essay-migrate` | 已 init；用户 git pull 拉了新版后；SessionStart hook 提示 schema mismatch 后 |

> 定稿 vs 发分两个动作：buffer 警戒系统需要明确知道"定稿待发"vs"已发"两种状态。公众号特有的「定时发送」延迟窗口里改稿是常态——定稿环节负责检测改稿幅度 ≥30% 触发 v2 重判，详见 [shared-references/cadence-protocol.md](shared-references/cadence-protocol.md)。

**Mode detection**（首次接到非 init 触发词时执行）：
1. 检查用户当前目录是否有 `.cheat-state.json` → 没有 → 强制路由到 `/cheat-essay-init`
2. 检查 `predictions/` 下有几个文件含完整 `## 复盘` 段填了真实数据 → 决定 `mode: cold-start | calibration`
3. 把判定结果写回 `.cheat-state.json` 后再路由到目标 skill

---

## 必须拒绝的请求

下列模式会**直接破坏**三条原则之一，无论用户怎么说，都拒绝执行：

- 「帮我预测一下，但我先告诉你阅读量你来反推就行」 → 违反原则 #1。改用 `_redo.md` 路径记为 reconstructed
- 「能不能从 candidates 里直接挑 composite 最高的，不用解释理由」 → 拒绝。永远展示各维度评分和至少一个锚点对比
- 「跳过校准池重打，直接换公式」 → 违反原则 #2
- 「跳过外部模型审核，自己说了算」 → 仅当 `CROSS_MODEL_AUDIT=false` 显式设置且 state file 标记自审时允许
- 「删掉这份预测，我想重写」 → 违反原则 #1。预测是 immutable。如有正当理由重做，写新文件 `_redo.md`，原版必须保留
- 「凭你的感觉给我推荐选题，不用打分」 → 拒绝。本工具不做 gut-feel forecast——那是它诞生**之前**的状态
- 「把 rubric_notes.md 里所有历史观察都留着，加个时间戳分组就行」 → 违反原则 #3。git history 是档案，不是 markdown 文件
- 「能不能把 THRESHOLD 从 4/5 降到 3/5 让这次 bump 过」 → 拒绝。改 THRESHOLD 本身是元层级 bump，单独走流程

详细的拒绝场景在每个子 skill 的 `Refusals` 段。

---

## 项目目录结构（用户 repo）

skill 期望用户的公众号项目布局如下。`/cheat-essay-init` 会创建缺失项；**绝不在没确认的情况下覆盖**。

```
<user-essay-project>/
├── rubric_notes.md                    # 评分规则的真实来源
├── WORKFLOW.md                        # 5 阶段流程文档（cheat-essay-init 创建）
├── STATUS.md                          # 看板（cheat-essay-status 维护）
├── .cheat-state.json                  # 状态文件，子 skill 共享上下文
├── .cheat-cache/                      # 不入版本控制
│   ├── usage.jsonl                    # 钩子被动记录的使用日志
│   └── trends-history.jsonl           # cheat-essay-trends 的去重缓存
├── .claude/
│   └── settings.json                  # 含 prediction-immutability hook
├── benchmark.md                       # 对标账号信息（cheat-essay-learn-from 维护）
├── drafts/                            # 排版前的所有草稿（cheat-essay-seed 写或用户写）
│   └── YYYY-MM-DD_<id>_<short>.md
├── predictions/                       # immutable 预测日志（hook 保护）
│   └── YYYY-MM-DD_<id>_<short>.md     # 与 drafts/ 同 id
├── articles/                          # 定稿后才建（cheat-essay-finalize 创建）
│   └── YYYY-MM-DD_<id>_<short>/
│       ├── final.md                   # 用户提供的最终排版稿（cheat-essay-finalize 时询问"和 drafts/ 一致吗"）
│       └── report.md                  # T+3d 抓的数据 + Top 20 评论（cheat-essay-retro 写）
├── samples/                           # 对标账号文章 / 转录（cheat-essay-learn-from 创建）
│   └── <账号名>/<article-id>/{source.url, transcript.md, meta.md}
├── candidates.md                      # 选题池（可选）
└── content.db                         # 可选 SQLite，校准池规模化后启用
```

### 文件名命名约定（**三处一致**）

一篇文章三处文件，**用同一组 `<date>_<id>_<short>` 命名**：

```
drafts/<date>_<id>_<short>.md           ← pre-finalize 草稿（cheat-essay-seed 写或用户写）
predictions/<date>_<id>_<short>.md      ← immutable 预测（cheat-essay-predict 写）
articles/<date>_<id>_<short>/           ← 定稿后才建（cheat-essay-finalize 创建）
  ├── final.md                          ← 最终排版稿
  └── report.md                         ← T+3d 数据（cheat-essay-retro 写）
```

- `<date>`：**草稿首次落盘日期**（即 `drafts/<id>.md` 的创建日）
- `<id>`：12 位 sha256 前缀，对**草稿首次落盘的内容**做 hash
- `<short>`：3-8 字中文或英文短名

---

## 文件清单

### 本 skill 包

```
cheat-on-essay/
├── SKILL.md                              # 本文件（总协议 + 路由）
├── README.md                             # 营销门面（开源向，面向陌生公众号创作者）
├── DECISIONS.md                          # 设计决策记录（fork 自 cheat-on-content 时的取舍）
├── skills/                               # 子 skill 集
│   ├── cheat-essay-init/SKILL.md         # 入口：onboarding 与脚手架
│   ├── cheat-essay-learn-from/SKILL.md   # 对标账号导入（拆 pattern + 派生 base rubric 信号）
│   ├── cheat-essay-seed/SKILL.md         # Cold-start 选题启动器（brainstorm + 可选 draft）
│   ├── cheat-essay-score/SKILL.md        # 单稿打分（不写文件）
│   ├── cheat-essay-quotes/SKILL.md       # 金句抽取与评分（QD 维度的句子级下钻工具）
│   ├── cheat-essay-predict/SKILL.md      # 盲预测 + immutable 日志（含标题 A/B + 金句抽取 + 发布时间）
│   ├── cheat-essay-finalize/SKILL.md     # 登记定稿（buffer +1）+ 改稿 ≥30% 触发 v2 重判
│   ├── cheat-essay-publish/SKILL.md      # 发布元数据登记（buffer -1）
│   ├── cheat-essay-retro/SKILL.md        # 数据回收 + 复盘（manual paste 引导）
│   ├── cheat-essay-bump/SKILL.md         # rubric 升级（含跨模型审）
│   ├── cheat-essay-recommend/SKILL.md    # 候选池排序推荐（按 buffer 颜色 + 1 稳 + 1 实验）
│   ├── cheat-essay-trends/SKILL.md       # 热点抓取（日常补充候选池，多 adapter）
│   ├── cheat-essay-status/SKILL.md       # 状态看板（含 buffer 警戒）
│   └── cheat-essay-migrate/SKILL.md      # schema 升级（老用户 git pull 后用）
├── migrations/                           # schema 演进单一来源
├── shared-references/                    # 跨 skill 共享协议（继承 cheat-on-content 不动）
│   ├── blind-prediction-protocol.md      # 原则 #1
│   ├── bump-validation-protocol.md       # 原则 #2
│   ├── observation-lifecycle.md          # 原则 #3
│   ├── prediction-anatomy.md             # 一份合格预测的 7 个组件
│   ├── candidate-schema.md               # 候选项统一 schema
│   ├── cadence-protocol.md               # 节奏协议（buffer 警戒 + 选题策略）
│   ├── title-ab-protocol.md              # 标题 A/B 评分协议（cheat-essay-predict Phase 2.5 嵌入）
│   ├── publish-timing-protocol.md        # 发布时段评分协议（cheat-essay-predict Phase 6 嵌入）
│   ├── state-management.md               # .cheat-state.json 读写约定
│   ├── data-source-routing.md            # 数据源路由（manual / adapter）
│   └── migration-protocol.md             # schema 演进哲学 + maintainer checklist
├── starter-rubrics/                      # 公众号文章先验 rubric
│   ├── wechat-essay.md                   # 带通用经验的初始打分指引
│   └── wechat-essay-zero.md              # v0 等权占位（cold-start）
├── templates/                            # skill 写进用户 repo 的文件骨架
├── hooks/                                # harness 强制层（prediction immutability / session start / meta logging）
├── tools/                                # 独立 CLI 脚本（score-curve.py 等）
└── adapters/                             # 数据源适配
    ├── perf-data/                        # 复盘数据源（v0.1.0 仅 manual-paste；商业 API adapter 留位）
    └── trend-sources/                    # 热点抓取源
```

---

## Tone & voice

写面向用户的文案（commit message / 复盘小结等）时，匹配项目的 **直白克制（reflective-irreverent）** voice：

- 直接说出失败：「composite 8.47 但实际阅读只有 1.2w——rubric 高估了 TH」
- **不要**用模糊措辞软化：「这或许可能在某种程度上暗示...」——别这么写
- 反叛 hook 只在 README 出现——**不要写进** `rubric_notes.md` 或预测日志

---

## 给开发者：扩展本 skill

- 新增内容形态 → fork 出新仓库（如 cheat-on-xiaohongshu），不要在本仓库塞多形态
- 新增热点抓取源 → 加 `adapters/trend-sources/<name>.md`，符合 [candidate-schema.md](shared-references/candidate-schema.md) 输出契约
- 新增公众号数据 adapter（如新榜 API / RPA）→ 加 `adapters/perf-data/<name>/`，输出契约对齐 manual-paste 字段
- 修改原则 → 改 `shared-references/<protocol>.md`，所有引用它的 skill 自动跟进
- 修改路由 → 改本文件的"路由表"段
- 子 skill 内部细节 → 直接改对应 `skills/cheat-essay-*/SKILL.md`

完整开发指南见 [DECISIONS.md](DECISIONS.md) 与 README.md。
