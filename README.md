# Think Like Karpathy

> 通过 git commit 历史、修改热点、commit 节奏，横纵分析 Karpathy 2025–2026 的 4 个开源 repo，学习他的**想法 / 哲学 / 实现 / 解决了什么问题 / 技术精妙的地方**。

## 被分析的 4 个 repo（均为 [@karpathy](https://github.com/karpathy) 的作品）

| Repo | 周期 | commits | 一句话速记 |
|---|---|---|---|
| [nanochat](https://github.com/karpathy/nanochat) | 2025-10 → 2026-04 (6 个月) | 380 | 单节点 + 可读代码 6 个月把 time-to-GPT-2 从 3.04h 压到 1.65h |
| [autoresearch](https://github.com/karpathy/autoresearch) | 2026-03 (3 周) | 35 | 让 agent 过夜跑实验；"人改 md / agent 改 py"的分权宪法 |
| [jobs](https://github.com/karpathy/jobs) | 2026-03 (2 天) | 11 | BLS 342 职业 treemap + LLM 可换上色层；README 定位反转最值得学 |
| [hn-time-capsule](https://github.com/karpathy/hn-time-capsule) | 2025-12 (36 小时) | 15 | 一次 vibe-code 的 LLM 应用工程教科书 |

## 这个仓库里是什么

**不是** karpathy 的代码 fork —— 仓库里只包含"对他 repo 的分析产出"。

- [`CONCLUSIONS.md`](CONCLUSIONS.md) — 跨 4 repo 的综合结论（~6K 字 · 6 章）
  - Karpathy 2025–2026 的开发宪法（4 repo 共现的 5 个模式）
  - 发布姿态光谱（4 档对应图）
  - `autoresearch → nanochat` 闭环的意义
  - 你能做什么（5 条按 ROI 排序）

- [`INSIGHTS.md`](INSIGHTS.md) — 深度洞察（~19K 字 · 6 章）
  - **A.** 意图三角测量：4 repo 合起来的 2026 议程
  - **B.** 实验结果清单（含 nanochat 的 6 个月加速拆解表）
  - **C.** 只有横看才能发现的有意思现象
  - **D.** 反常识点（8 条）— 删除防护代码 / 墙钟不 FLOPs / 首日回撤 / 大改破兼容 / commit 留情绪 …
  - **E.** 思维模式目录（12 条命名 pattern）— Shadow-drive→Public-drop / Single-dial Scaling / Explicit Over Magic …
  - **F.** 可验证的未来预测

- [`reports/`](reports/) — 4 份独立 HTML 深度报告（每份按"横纵分析法"结构）
  - [`autoresearch.html`](reports/autoresearch.html) · 35K · 约 8.2K 字
  - [`hn-time-capsule.html`](reports/hn-time-capsule.html) · 27K · 约 7.2K 字
  - [`jobs.html`](reports/jobs.html) · 31K · 约 7.8K 字
  - [`nanochat.html`](reports/nanochat.html) · 45K · 约 13.6K 字
  - 每份报告含 **0. 研究计划 / 1. 纵向（时间叙事）/ 2. 横向（竞品对比）/ 3. 横纵交汇 / 4. Changelog**

- [`CHANGELOG.md`](CHANGELOG.md) — 研究过程的步骤 / 想法 / 结果 / 结论，便于回溯判断路径

## 怎么读

- 想快速了解作者思维 → 先读 `CONCLUSIONS.md`
- 想学习可直接挪用的 pattern → 读 `INSIGHTS.md` 的 D + E 章
- 想深入某个 repo → 在浏览器打开 `reports/<repo>.html`
- 想看我们是如何做这件事的 → `CHANGELOG.md`

## 方法论

本研究采用"横纵分析法 v2"（基于[数字生命卡兹克](https://x.com)原版改进，加了研究计划 / 类型前置判断 / 信息缺口处理 / 用户视角可追溯 / 交汇拆三问 / 输出长度处理）。

每份报告遵循：
- **纵向**：沿时间轴完整还原演进（不写流水账，写叙事）
- **横向**：选 3–5 个同类竞品深入对比（不是功能表）
- **交汇**：路径依赖判断 + 差距可追赶性 + 决策建议（最应该做 / 最不应该做）

## 数据来源

- 公开 git history（`git log` / `--name-only` / `--shortstat` / 关键 commit diff）
- 每 repo 的 `README.md` / `dev/LOG.md` / `dev/LEADERBOARD.md`
- 关联 tweet / 博文（README 中的外链，如 Karpathy bearblog、x.com/karpathy）
- 不使用任何作者内部沟通；结论均基于可 `git blame` 溯源的公开信息

## 数据截止

2026-04-17。每份报告的 "数据截止" 时间写在文末 footer。

## 关于"反常识点"

如果只读一节，读 `INSIGHTS.md` 的 **D 章**——8 个与工程师默认直觉相反、但 Karpathy 实际这样做、并且从 git 历史看确实奏效的选择。例子：

- "删除防护代码"（gradient clipping 测出从未触发 → 直接删）
- "首日公开删除自己的 v0 设计"（autoresearch 的 spawn.sh 首日死）
- "在 commit msg 里写痛苦"（`i suffered` / `dependencies bad bad bad` / `dam`）
- "LLM 是受众而不是工具"（`prompt.md` 专门给 LLM 消费）
- "Ship 之前先跑通所有事"（initial commit 全部是完整链路，非 MVP）

---

*如发现报告中的事实错误，欢迎开 issue 纠正；作者方法判断可商榷，但事实必须基于可追溯证据。*
