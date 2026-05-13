# Changelog

All notable changes to **cheat-on-essay** will be documented here.

格式参考 [Keep a Changelog](https://keepachangelog.com/zh-CN/1.1.0/)，版本号遵循 [SemVer](https://semver.org/lang/zh-CN/)。

---

## [0.1.0] — 2026-05-12

### Added — Initial release

**cheat-on-essay** fork 自 [cheat-on-content](https://github.com/XBuilderLAB/cheat-on-content)（视频版）。同一套校准方法论（盲预测 → T+3d 复盘 → 进化 rubric）平移到微信公众号长图文场景。

#### 核心改动 vs cheat-on-content

- **rubric 重写**：7 维从 ER/HP/QL/NA/AB/SR/SAT（视频）→ TH/OP/IE/EG/SR/QD/AE（公众号）
  - **TH** 标题钩子力（决定 40% 打开率）
  - **OP** 开头留人（决定完读率前段）
  - **IE** 议题锋度（决定 30% 互动率）
  - **EG** 情感颗粒（决定 20% 分享率）
  - **SR** 结构节奏（完读率底层）
  - **QD** 金句密度（朋友圈截图驱动）
  - **AE** 行动出口（涨粉转化 + 互动）
- **目录命名**：scripts/→drafts/、videos/→articles/（公众号场景的语义对应）
- **数据回收方式**：默认 manual paste（公众号无可用免费 API；adapter 接口留位等 v0.2）
- **三个公众号独有机制**：
  - 标题 A/B 评分（嵌入 cheat-essay-predict Phase 2.5）—— 用户给 2-3 个候选，工具打分 + 推荐
  - 金句抽取与评分（独立子 skill `cheat-essay-quotes`）—— 5 维传播力评分
  - 发布时段评分（嵌入 cheat-essay-predict Phase 6）—— 三档黄金时段 + 议题类型映射
- **cheat-shoot → cheat-essay-finalize**：从"拍了"语义换成"定稿"——公众号「定时发送」窗口内改稿是常态
- **状态 schema 1.0**：与 cheat-on-content 1.2 不兼容，加 `skill_package: "cheat-on-essay"` 字段防同目录混装

#### 子 skill 清单（14 个）

```
cheat-essay-init / learn-from / seed / score / quotes / predict /
finalize / publish / retro / bump / recommend / trends / status / migrate
```

#### 协议文档

- 继承自 cheat-on-content（不动）：blind-prediction-protocol、bump-validation-protocol、observation-lifecycle、prediction-anatomy、candidate-schema、cadence-protocol、state-management、data-source-routing、migration-protocol
- 新增公众号专属：
  - [title-ab-protocol.md](shared-references/title-ab-protocol.md) — 标题 A/B 评分协议
  - [publish-timing-protocol.md](shared-references/publish-timing-protocol.md) — 发布时段评分协议

#### Starter rubrics

- [starter-rubrics/wechat-essay-zero.md](starter-rubrics/wechat-essay-zero.md) — v0 等权占位（cold-start）
- [starter-rubrics/wechat-essay.md](starter-rubrics/wechat-essay.md) — 通用经验版（v0→v1 bump 起点；带通用经验权重 TH×1.5 / IE×1.5 / OP×1.2 / EG×1.2 等）

#### 工具作者预埋的 v0 已知缺陷（写进 rubric_notes.template.md 的「未来观察清单」）

跑完 5 篇后用户应核对：

- [ ] IE 维度对教程实操类文章会失真
- [ ] AE 维度被「点赞+在看+转发+星标」标准模板压平
- [ ] QD 锚点「密度 OR 分布」需要明确为 AND
- [ ] rubric 不抓「热点借势力」（新闻类文章）
- [ ] rubric 不抓「人格信号」（头部账号读者跟人不跟文）
- [ ] rubric 适用形态边界（短消息卡片 / 图集 / 软文不适用）

详见 [docs/sample-calibration-v0.1.0.md](docs/sample-calibration-v0.1.0.md)——基于 5 篇真实公众号文章的 v0 rubric 校准分析。

#### v0.1.0 限制

- 数据回收**仅 manual paste**（adapter 接口在 `adapters/perf-data/` 留位但未实现）
- 热点抓取（cheat-essay-trends）所有 adapter 标 `experimental`——从 cheat-on-content 继承，公众号场景命中率未校准
- 未做小红书（小红书计划独立仓库 cheat-on-xiaohongshu，复用 shared-references/）

#### 三条不可妥协原则（继承自 cheat-on-content，永不动）

1. **盲预测**：predictions/ 下 `## 预测 vN` 段 immutable，hook 阻塞 PreToolUse(Edit|Write)
2. **Bump = 全量重打**：rubric 升级必须重打校准池 + ≥4/5 排序一致 + 跨模型独立审
3. **rubric 是工作台**：被推翻 / 被吸收的观察 → 删，git history 是档案

---

## [Future Roadmap]

### v0.2 候选

- **adapters/perf-data/newrank/** — 接入新榜 API（按次付费），从 manual paste 升级到自动抓取
- **standalone rubric for tutorials** — 教程实操形态的独立 rubric（解决 IE 维度失真）
- **HM dimension** — 新增热点借势力维度（覆盖新闻速递形态）

### v0.3 候选

- **PI dimension** — 人格信号维度（覆盖头部账号「读者跟人不跟文」）
- **standalone rubric for news** — 新闻热点形态的独立 rubric

### v1.0 目标

- 跑通 50+ 真实用户的 5 篇校准 → 验证 v0 rubric 在多账号上是否收敛到稳定形态
- 写完 [cheat-on-xiaohongshu](https://github.com/<your-org>/cheat-on-xiaohongshu) 姊妹仓库（小红书图文场景）
