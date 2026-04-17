# Think Like Karpathy

> 通过 git commit 历史、修改热点、commit 节奏，横纵分析 Karpathy 2025–2026 的 4 个开源 repo，学习他的**想法 / 哲学 / 实现 / 解决了什么问题 / 技术精妙的地方**。

## 精华 TL;DR

如果只能读 3 段，读这个。

### 🧠 Top 3 学到的思维方式

**1. Shadow-drive → Public-drop（先私跑再发布）**
4/4 repo 的 initial commit 都是**完整链路一次性入库**——nanochat 10,292 行 / autoresearch 2,848 行 / jobs 656,087 行 / hn-time-capsule 2,073 行。没有一个 repo 以 MVP 形式起步。这和 "ship early, ship often" 的敏捷默认完全相反，保护的是一次性的**注意力资产**。适用场景：任何你的 repo 会被初次评估一次就被判定价值的场合。

**2. Disk Cache as Interface（磁盘作为阶段边界）**
hn-time-capsule 5 阶段 / jobs 6 阶段 / nanochat `runs/` + `checkpoint_manager`——每个阶段的产物都是可失效的文件。**昂贵阶段（LLM 调用、GPU 训练）一旦跑过永不重跑**；改 prompt 只重跑下游、改 parser 只重跑最后一步。附带好处：阶段产物能 `grep`、能 `git diff`，调试极友好。这是可立刻抄的模板。

**3. Narrate the Struggle（在 commit msg 里留情绪）**
`i suffered` / `dependencies bad bad bad` / `dam, erase experimental file from before that snuck through in my purge` / `incredible`——Karpathy 在 commit msg 里留真实情绪。这不是不专业，是**叙事工程**：让 git 历史未来可读，让协作者意识到这是人与真实问题搏斗。规则：情绪要具体（`i suffered` 后面跟 `fp8 integration`），不要当抱怨。

---

### 🤯 Top 3 反常识点

**1. 删除防护代码**
nanochat 2026-01-08 `dev/LOG.md` 记录：在 d12 / d20 上扫了 gradient clipping 的 4 个 threshold，**全部落在 noise 范围**；grad norm 自然从不超过 1.0，clipping 永不触发。结论：删掉整套 clipping 代码（`Deleted all grad-clip code paths`）。大多数工程师只加不减、永远不删"看起来安全的东西"——这里**安全感让位于实验证据**。学到的：看到代码里有"从来不触发的保护"，要么真的删掉，要么测出它触发过一次才保留。

**2. 首日公开删除自己的 v0 设计**
autoresearch initial commit (`b11d6f2`) 包含 248 行的 `spawn.sh`（多-agent 编排脚本）——这是作者的 v0 野心版。但**同一天内被删**（`1e207aa "dam, erase experimental file from before that snuck through in my purge"` + `ae81d55` + `69eb7f9`）。大多数作者会把 v0 留几天观察，Karpathy 几小时内就回撤。学到的：**承认错误的速度本身就是竞争优势**，不给后悔留时间。

**3. LLM 是受众而不是工具**
jobs 的 `prompt.md`（45k token）是专门给 LLM 消费的 artifact——不是 JSON（给机器）、不是文档（给人类全程读），而是 "人粘贴给 LLM + 跟它聊" 的一份聚合快照。autoresearch 的 `program.md` 同理，是给 agent 看的剧本。**先设计 LLM 友好的 artifact，再让人使用它**——这是 2026 年正在成型的一等设计模式，大多数 repo 仍把 LLM 当 fallback 输入。

---

### 📊 Top 3 验证的实验结论

**1. nanochat 6 个月加速的杠杆排序：数据 > fp8 > 代理 > 批大小**

| Δ | 技术 | 单次加速 | commit |
|---|---|---|---|
| FineWeb-EDU → ClimbMix 数据换血 | **-26.8%** | `324e69c` |
| autoresearch round 1 | -10.9% | `6ed7d1d` |
| autoresearch round 2 | -8.3% | `a825e63` |
| batch size → 1M tokens | -5.2% | `2c062aa` |
| d26 + fp8 | -4.3% | `a67eba3` |

**一次数据换血的收益 > 其他所有架构/精度/批大小优化累加**。对任何 pretraining 项目，优先看数据，再看算法。

**2. autoresearch → nanochat 闭环是 2026 年 autonomous AI research 唯一干净的 proof point**
autoresearch round 1（Claude 2 days autonomous on d12，`6ed7d1d`）和 round 2（smear + backout + hyperparam tuning，`a825e63`）的改进**直接 merge 进 nanochat 主线 leaderboard（第 5 + 第 6 行）**，贡献了 18% wall-clock 加速。不是 demo 论文、不是 benchmark 游戏，是 PR 记录。这是未来 1-2 年行业会被反复引用的案例。

**3. initial commit 规模验证了 "private → public drop" 的发布哲学**

| Repo | initial commit 行数 | 文件数 |
|---|---|---|
| jobs | 656,087（含 342 份 BLS raw HTML） | 360 |
| nanochat | 10,292 | 47 |
| autoresearch | 2,848 | 10 |
| hn-time-capsule | 2,073 | 9 |

**4/4 为完整链路**（非 MVP）。且 4/4 都在 initial commit 后 1-3 天内出现 "deprecated file oops / dam erase experimental file" 类 commits——说明私跑阶段代码已包含被误带入的文件，作者发布后第一波清理的是**自己的脏文件**而非 bug。

---

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

- [`reports/`](reports/) — 4 份独立 HTML + ipynb 深度报告（每份按"横纵分析法"结构）
  - [`autoresearch.html`](reports/autoresearch.html) + [`.ipynb`](reports/autoresearch.ipynb) · 35K · 约 8.2K 字
  - [`hn-time-capsule.html`](reports/hn-time-capsule.html) + [`.ipynb`](reports/hn-time-capsule.ipynb) · 27K · 约 7.2K 字
  - [`jobs.html`](reports/jobs.html) + [`.ipynb`](reports/jobs.ipynb) · 31K · 约 7.8K 字
  - [`nanochat.html`](reports/nanochat.html) + [`.ipynb`](reports/nanochat.ipynb) · 45K · 约 13.6K 字
  - HTML 含 **0. 研究计划 / 1. 纵向（时间叙事）/ 2. 横向（竞品对比）/ 3. 横纵交汇 / 4. Changelog**
  - ipynb 是**可运行的证据**——clone 对应 repo 后跑一下，每条论断都能当场验证

- [`skills/`](skills/) — 可复用的 agent skill 文件
  - [`iterative-experiment.md`](skills/iterative-experiment.md) — 把 autoresearch `program.md` + nanochat `dev/LOG.md` 提炼成**通用"agent 过夜迭代实验"skill**，可直接放到你自己 repo 的根目录作为 `program.md`

- [`PLAYBOOK.md`](PLAYBOOK.md) — **实战手册**：如果你要做类似的工程/AI 实验项目，每条建议都带 commit 证据
  - 四条基本宪法 + 启动/写代码/AI 实验/commit 实践/agent 协作/发布/反 pattern 警示 + 一页纸 checklist

- [`CHANGELOG.md`](CHANGELOG.md) — 研究过程的步骤 / 想法 / 结果 / 结论，便于回溯判断路径

- [`analysis.ipynb`](analysis.ipynb) — **可复现 Jupyter notebook**。clone 被分析的 4 个 repo 后，从头到尾运行即可重新采样所有证据，或改查询验证你自己的假设

## 怎么读

- 想快速了解作者思维 → 先读 `CONCLUSIONS.md`
- 想学习可直接挪用的 pattern → 读 `INSIGHTS.md` 的 D + E 章
- 想深入某个 repo → 在浏览器打开 `reports/<repo>.html`，跑 `reports/<repo>.ipynb` 验证
- **想自己做类似项目 → 读 `PLAYBOOK.md`**
- **想让 agent 帮你跑实验 → 把 `skills/iterative-experiment.md` 放进你的 repo 当 `program.md`**
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
