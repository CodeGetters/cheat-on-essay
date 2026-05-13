<h1 align="center">
  <img src="logo.svg" alt="Cheat on Essay" width="720">
</h1>

<h2 align="center">Cheat on Essay</h2>

<p align="center">
公众号作弊器 — 把每一篇推送变成可校准的实验。
</p>

<p align="center">
你正在读这段话——这个 skill 预测过了。<br>
它把作者的每一次"我感觉这篇会爆"变成可校准的实验。<br>
它把每一次盲预测、每一条 top 评论、每一次 T+3d 复盘，编织成只属于你的爆款公式。<br>
你停下来思考"这是不是真的"——也在它的预测里。
</p>

<p align="center">
  <a href="../README.md"><strong>English</strong></a>
  &nbsp;·&nbsp;
  <strong>简体中文</strong>
</p>

<p align="center">
<a href="../CHANGELOG.md"><img src="https://img.shields.io/badge/version-v0.1.0-orange" alt="Version"></a>
&nbsp;
<a href="../LICENSE"><img src="https://img.shields.io/badge/license-MIT-blue.svg" alt="License"></a>
</p>

---

## 🎬 它真正在干什么

大部分公众号作者活在同一个赌局里：

> 推送 → 数据出来 → 学不到东西 → 下次继续赌

跑过 200 篇的作者跟跑过 1 篇的差距不到 10%——因为他们没在每次推送后**记账**。

**公众号外挂**让每一次判断都被记录、被复盘、被吸收进下一次：

📊 7 维打分 → 🏷️ 标题 A/B 评分 → 🎯 盲预测 → 📐 定稿 → 🚀 发布 → 📈 T+3 天复盘 → 🧬 进化你的评分公式

这不是 motivation，是 **compounding**——你不复盘的每一篇，都是在折损"看见自己"的能力。

跑一个月 = 你有了一份**只属于你账号的爆款公式**。
跑三个月 = 你比刚开始的自己强 10 倍。

---

## 🌀 起源

这是 [cheat-on-content](https://github.com/XBuilderLAB/cheat-on-content)（视频版）的姊妹仓库。同一套方法论——打分 → 盲预测 → 复盘 → 进化 rubric——平移到公众号长图文场景。

但**rubric 必须重写**：视频靠完播率和情感共鸣（ER/HP/QL/NA/AB/SR/SAT 7 维），公众号靠标题钩子和议题锋度。这版 rubric 重新设计了 7 维：

- **TH** 标题钩子力（决定 40% 打开率）
- **OP** 开头留人（决定完读率前段）
- **IE** 议题锋度（决定 30% 互动率）
- **EG** 情感颗粒（决定 20% 分享率）
- **SR** 结构节奏（完读率底层）
- **QD** 金句密度（朋友圈截图驱动）
- **AE** 行动出口（涨粉转化 + 互动）

---

## ⚖️ 它和别的"创作工具"哪里不一样

| 别人 | 这个 |
|---|---|
| 给你"灵感" | 让你**自己的灵感被量化** |
| AI 帮你写 | AI 帮你**判**——稿子还是你的 |
| 一发发 3 个标题 A/B 测 | 一发就**赌**——把判断写下来，数据出来对账 |
| 静态数据看板 | **会进化的评分公式**——你三个月后的 rubric 已经不是初始版 |
| AI 帮你生成标题 | AI 帮你**给候选标题打分**——你的判断对象必须是你写的 |

一句话：别的工具帮你"产出更多"，这个工具帮你"判得更准"。

---

## 🤔 那 ChatGPT / 豆包 / DeepSeek 不是也能干这个？

那是**通用助手**——对所有人说同样的话。你问"我这篇会爆吗"，它的答案是从全网平均经验拟合出来的，跟你的账号没关系。明天再问一遍，答案还是上次那个——**它不记得你，更不会因为你而变**。

这套是**你自己的运营专家**，只服务你这一个公众号：

- 评分公式从**你的**历史数据反推，不是通用训练分布
- 每发一篇它就更新一次对你账号的理解——三个月后判断准度比刚开始强 10 倍（**自动进化**）
- 它知道你的对标账号、你的发布 cadence、你最近三篇为什么扑——这些 ChatGPT 第一句话就忘了

通用 LLM 帮所有人；这套帮你**这个**号。

---

## 🛡️ 它怎么让循环真的能进化

📝 **每篇都留底**：发布前打分、写预测、给候选标题评分、定发布时段，全程存档。三天后回来对账——你哪里准、哪里偏，**一目了然**，不再是模糊的"感觉这次没发好"。

🔁 **越用越准**：连续三次同方向偏差，或单次 ≥10x 极端偏差，工具自动催你升级评分公式。**你不主动它也催**。

🛡️ **升级有刹车**：换公式必须用新公式重判所有历史样本，能比旧公式更准才放行；还要跨模型独立审一次——**防你自己骗自己**。

🪒 **rubric 是工作台不是博物馆**：被推翻的观察删，被吸收的也删。永远只放当下最有用的。

---

## 📦 安装

```bash
git clone https://github.com/<your-org>/cheat-on-essay.git
cd cheat-on-essay
bash install.sh
```

14 个子 skill 软链接到你 agent 的 skill 目录。装一次，所有公众号项目都能用。

**支持的 agent**：Claude Code（默认）· Codex（`bash install.sh --codex`）· 两个都装（`bash install.sh --all`）

> 冻结版本：`bash install.sh --copy` / `bash install.sh --codex --copy`
>
> 卸载：`bash uninstall.sh` / `bash uninstall.sh --codex`（不动你的内容数据）

---

## 🚀 第一次跑

在你的公众号项目目录里打开支持 skill 的 agent，说：

```
初始化 cheat-on-essay
```

5-6 个问题搞定 onboarding。**强烈建议导对标账号**——5-10 篇样本 → 工具立刻有 anchor，不然前 5 篇预测精度 ±50%。

---

## ⚡ 日常用法

```
打分这篇 drafts/<...>.md             → 7 维评分（探索）
抽金句 drafts/<...>.md                → 5 维传播力评分 + 候选金句库
启动预测 drafts/<...>.md              → 7 维 + 标题 A/B + 发布时段 + 盲预测
定稿 drafts/<...>.md                  → 建 article folder + 改稿 ≥30% 触发 v2 重判
已发布 https://mp.weixin.qq.com/s/... → buffer -1
复盘 articles/<...>/                  → T+3d 6 数字 + 20 评论 + 复盘
状态 / 抓热点 / 找选题 / 升级 rubric / 找对标
```

支持 hook 的 agent 每次开会话自动报告 buffer + 待复盘 + top 候选——你不用主动问。其他 agent 直接说 `状态` 即可。

完整工作流 + 子 skill 细节见 [SKILL.md](../SKILL.md)。

---

## ⚠️ rubric 适用边界（必读）

v0 设计目标是**议题观点 / 行业分析 / 深度知识 / 个人随笔类长图文**（1k-5k 字）。

| 形态 | rubric 适用度 |
|---|---|
| 议题观点 / 时评 / 行业分析 | ✅ 完全适用 |
| 深度知识 / 长文科普 | ✅ 完全适用 |
| 个人随笔 / 情感叙事 | ✅ 完全适用 |
| 教程实操 / 工具分享 | ⚠️ 部分适用（IE 维度建议标 N/A） |
| 新闻速递 / 热点点评 | ⚠️ 部分适用（rubric 抓不到时效 + 大号 + 推荐流加权） |
| < 500 字短消息卡片 | ❌ 不适用 |
| 图集主导 / 纯营销软文 | ❌ 不适用 |

工具仍然能跑，但分数解释力会打折——见 [DECISIONS.md](../DECISIONS.md) §5 与 [docs/sample-calibration-v0.1.0.md](sample-calibration-v0.1.0.md) 的样本校准分析。

---

## 📊 复盘需要你提供的数据

v0.1.0 默认走 **manual paste**——公众号生态没有可用的免费第三方 API。

复盘时你需要从公众号后台抄：

```
✅ 阅读数（总）
✅ 在看数 + 点赞数
✅ 分享次数
✅ 涨粉数（24h 或 48h）
🟡 完读率（如后台显示）
✅ Top 20 评论（带赞数）
```

后台路径：手机微信 → 订阅号助手 → 数据 → 内容分析 / 或 mp.weixin.qq.com → 数据分析 → 内容分析。

**评论才是真信号**——「分享率 8%」不能告诉你为什么，但 20 条评论关键词能。复盘工具会强制要求评论 ≥5 条。

未来 v0.2 候选：接入新榜 / 西瓜助手等第三方付费 API。

---

## 📈 Star History

<a href="https://star-history.com/#<your-org>/cheat-on-essay&Date">
  <img src="https://api.star-history.com/svg?repos=<your-org>/cheat-on-essay&type=Date" alt="Star History Chart" width="720">
</a>

---

## 📜 License

MIT。商用、改造、闭源接入都行。

---

*这是作弊吗？计算器也是。Google 也是。*
*未来从不奖励努力——它只奖励先看见规律的人。*

*你看到这一行——也是它预测的。*
