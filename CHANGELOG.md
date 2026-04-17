# 横纵分析报告 Changelog

记录 4 个 repo 研究报告的过程 / 实验 / 想法 / 结果 / 状态 / 结论，方便回溯。

---

## 2026-04-17 T0 — 任务启动

- **请求**：用"横纵分析法"（卡兹克 + 改进版）分析 `autoresearch`, `hn-time-capsule`, `jobs`, `nanochat` 四个 repo，通过 git 提交记录、修改热点来学习作者的想法 / 哲学 / 实现 / 解决的问题 / 技术精妙。
- **附加要求**：每份报告都记录 changelog（实验/想法/结果/状态/结论）。
- **采样数据**（已落 `/tmp/ar-dig/<repo>/`）：
  - `autoresearch`：35 commits (2026-03-06 → 03-25)，热点 `README.md`(15), `program.md`(9), `train.py`(5), `prepare.py`(5)，10 个贡献者（Karpathy + 9 社区）。
  - `hn-time-capsule`：15 commits (2025-12-09 → 12-10，36 小时)，热点 `pipeline.py`(11), `README.md`(5), `CLAUDE.md`(4)。单作者 karpathy。
  - `jobs`：10 commits (2026-03-14 → 03-15，两天)，热点 `site/index.html`(7), `README.md`(7)。单作者 karpathy。
  - `nanochat`：379 commits (2025-10-13 → 2026-04-14，6 个月)，热点 `scripts/base_train.py`(56), `README.md`(45), `nanochat/gpt.py`(36), `dev/LOG.md`(36), `scripts/chat_sft.py`(24)。约 70+ 贡献者，前 5 名：Karpathy(248 合并)、svlandeg(40)、Luke Stanley(8)、Eric Silberstein(6)、Dipesh Babu(4)。
- **计划**：分 4 轮输出 HTML 报告，顺序 `autoresearch → hn-time-capsule → jobs → nanochat`，每个 repo 根目录生成 `report.html`；本文件汇总每轮的关键发现与决策。
- **状态**：采样完成，开始写第 1 份报告。

---

## 报告产出进度

| # | Repo | 状态 | 文件 | 关键结论速览 |
|---|------|------|------|------|
| 1 | autoresearch | ✅ 完成 | `autoresearch/report.html` | 哲学=人改 md/agent 改 py；首日删 spawn.sh 从多代理撤回；S2 全是 agent runtime robustness；最大杠杆=把 skill-as-md 抽成样板 |
| 2 | hn-time-capsule | ✅ 完成 | `hn-time-capsule/report.html` | 36h 两次坐下 vibe code；5 阶段磁盘缓存 pipeline；76da7d5 一条 commit = LLM 应用工程 101；场景 A 无竞品 |
| 3 | jobs | ✅ 完成 | `jobs/report.html` | 1cca284 是核心事件（README 定位反转 AI Exposure → 工具）；prompt.md = LLM-as-audience 新模式；场景 C 5 位竞品 |
| 4 | nanochat | ✅ 完成 | `nanochat/report.html` | 6 个月 168h→1.65h 100× 加速；杠杆序=数据>fp8>代理>批大小；dev/LOG.md 是最稀缺副产品 |

（每完成一份会回填此处 + 追加 T_n 段落记录当次实验/想法/结论）

---

## 2026-04-17 T1 — autoresearch 完成

- **实验**：git log + hotspots 采样；挖 initial commit 发现 spawn.sh（248 行多 agent 编排脚本）同日被删。
- **想法**：这个 repo 的价值重心在 `program.md` 而不是 `train.py`；"skill-as-markdown"是可抽象的样板。
- **结果**：
  - S1 最大事件 = spawn.sh 之死（首日从多代理撤回单代理）。
  - S2 全是 agent runtime robustness（NaN/traceback/无数据/FA3 fallback），非传统 ML 调试。
  - autoresearch 的 round 1/2 被 nanochat leaderboard 直接收录（6ed7d1d, a825e63），形成"预研仓→主仓"闭环。
- **状态**：报告 HTML 已落盘至 `autoresearch/report.html`（纵 4.2k 字 / 横 2.6k / 交汇 1.4k）。
- **结论**：① 分权宪法是核心；② 真正难度是 "agent runtime"；③ 最大杠杆=抽离 skill-as-md kit。
- **遗留盲点**：autoresearch round 1/2 的代理对话 log 未开源，无法还原哪些实验被否。

---

## 2026-04-17 T2 — hn-time-capsule 完成

- **实验**：按分钟时间戳展开 15 条 commits，观察节奏。
- **想法**：这个 repo 最重要的不是产品本身，而是三样东西：①5 阶段磁盘缓存 pipeline ②LLM prompt 三件套（结构约束 + grounding + parser 宽容）③"vibe + 不维护"的发布姿态。
- **结果**：
  - 10 条 commits 挤在 12/9 下午 6 小时内，中位间隔 <10 分钟 — 典型 LLM-pair-programming 节奏。
  - `76da7d5 "tune prompt again"` 在 9 行 diff 里浓缩了 LLM 应用工程的三件套。
  - `bdde346` 删 CLAUDE.md 的立场（"你自己 init 你自己的"）与 autoresearch 的 program.md 立场形成对照——<em>一个是工具资产、一个是用户 dotfile</em>。
  - 场景 A（无直接竞品）：LLM 能力和成本直到 2024 Q4 才跨过门槛，这个位置还空着。
- **状态**：HTML 已落盘至 `hn-time-capsule/report.html`。
- **结论**：① pipeline + prompt 契约 + 不维护声明 = 一日工具模板。② 下一步最大杠杆是补 hindsight-bias 校正。③ 不要扩功能，抽 pipeline 成可复用库才对。
- **遗留盲点**：initial commit 之前的 Claude 对话没记录；LLM API 实际花费未披露。
- **跨 repo 观察**：<b>autoresearch vs hn-time-capsule</b> 是 karpathy 两种 agent 工作流姿态的对照：前者把 agent 指令作为项目核心（人改 md、agent 改 py），后者把 agent 指令视为用户私有配置（CLAUDE.md 被删）。两者都是对的，但对应不同"使用 agent 的意图"——持续迭代 vs 一次性产出。

---

## 2026-04-17 T3 — jobs 完成

- **实验**：按分钟级时间戳展开 11 条 commits，定位到 3/14 15:39 → 3/15 20:10 的 28.5h 沉默段。
- **想法**：沉默段后第一批 commits 明确回应"读者误解"，很可能是 Twitter/X 反馈触发。
- **结果**：
  - `1cca284 "update the readme to minimize misunderstanding"` 是全 repo 最重要事件——从"AI 暴露分析"→"BLS 数据探索工具"的 6 处定位稀释。
  - `score.py` rubric 把"AI 暴露"定义为"工作产物是否原生数字"——窄到可操作，避开"被替代概率"的预测雷。
  - `prompt.md` 是 LLM-as-audience 的新设计模式：给 LLM 一个 45k token 的聚合 Markdown 快照。
  - 场景 C 对比 5 家：Frey-Osborne / OECD / OpenAI GPTs are GPTs / Anthropic Economic Index / McKinsey 系——jobs 不与任何正面竞争。
- **状态**：HTML 已落盘至 `jobs/report.html`。
- **结论**：① 定位反转是学到"如何避开 Frey-Osborne 十年来的媒体化陷阱"的活教材。② prompt.md 值得单独命名为 "single-file conversational dataset" 模式。③ 最大杠杆 = 把自己抽成 "bls-explorer" 模板供生态复用。
- **遗留盲点**：定位反转的外部触发点没直接证据（只能按时间线推测）；LLM 打分 variance 未评估。
- **跨 repo 观察累积**：karpathy 的 repo 都用<b>磁盘缓存 pipeline</b>（hn-time-capsule 5 阶段 / jobs 6 阶段），每一阶段的产物都是可失效的文件；这已经是他的签名架构。

---

## 2026-04-17 T4 — nanochat 完成

- **实验**：按月统计 commits 分布、按关键词挖 leaderboard 对应 commits、读 dev/LOG.md 1077 行。
- **想法**：nanochat 是 4 个 repo 中最复杂、最有教学价值的一个；它是"完整 ChatGPT 链路 × 单节点 × 可读"三交集的独占品。
- **结果**：
  - 月分布：10 月 101 / 11 月 54 / 12 月 35 / 1 月 97 / 2 月 53 / 3 月 33 / 4 月 7——双高峰（发布冲刺 + 科研觉醒期）。
  - Leaderboard 加速拆解：3.04h (Jan 29) → 2.91 (fp8) → 2.76 (1M batch) → 2.02 (ClimbMix, 27%!) → 1.80 (autoresearch r1) → 1.65 (r2)。**杠杆顺序：数据换血 > fp8 > 代理 > 批大小**。
  - initial commit 已入库 10,292 行——即发布前私下已经跑通全链路。
  - `dev/LOG.md`（始于 Jan 7 2026）是最稀缺产物：公开负面结果（LeakyReLU²/partial RoPE 等全失败）、WD scaling law (WD ∝ 1/width²)、fp8 3 周挣扎自白（`238353c "i suffered"`）。
  - 贡献者结构：Karpathy 248 / svlandeg 40 / 其余长尾——健康但 Karpathy-centric。
- **状态**：HTML 已落盘至 `nanochat/report.html`（纵 7.8k / 横 3.8k / 交汇 2k 字）。
- **结论**：① nanochat = 完整 ChatGPT 链路 × 单节点 × 可读 独占位；长期护城河清晰。② 最大杠杆不在再提速 10%，在于把代理注入的 smear/backout 写进 LOG 保住可读性资产。③ dev/LOG.md 是开源训练项目里罕见的"公开 lab notebook"，值得单独被引用。
- **遗留盲点**：smear/backout 具体架构变更没在 LOG 有独立章节；fp8 3 周挣扎期中间过程不可见；autoresearch agent 对话 log 未开源。

---

## 全局综合 · 4 个 repo 读下来学到什么

**Karpathy 的签名架构（4 个 repo 共现）**：
- **磁盘缓存 pipeline**：hn-time-capsule 5 阶段 / jobs 6 阶段。每阶段产物是可失效文件，LLM 昂贵阶段 3 永远可以保不动。
- **LLM-as-audience artifacts**：autoresearch 的 `program.md`（给 agent 看）、jobs 的 `prompt.md`（给人+LLM 对话）、hn-time-capsule 的 CLAUDE.md（被删但象征存在）。这是 2026 新兴设计模式。
- **单一旋钮 / 单一指标**：nanochat 的 `--depth` + CORE；autoresearch 的 5 分钟 + val_bpb；jobs 的 0-10 AI 暴露。

**Karpathy 的发布姿态光谱（4 档）**：
| Repo | 姿态 | agent 指令定位 | 维护意图 |
|------|------|---------------|----------|
| nanochat | 严肃开源项目 + leaderboard | 不涉及 | 长期维护 |
| autoresearch | 样板 + 预研仓 | `program.md` 是核心资产 | 中长期 |
| jobs | 一次性工具 + live demo | 无 agent 指令 | 不维护 |
| hn-time-capsule | 36h vibe code + 发博文 | CLAUDE.md 被主动删 | 明确不维护 |

**跨 repo 关键线索**：autoresearch round 1/2 → nanochat leaderboard #5 #6 的闭环，是 2026 年 autonomous AI research 最干净的 proof point（不是 demo，是 merged 进主线）。

**给自己的三条取经**：
1. 启动新 repo 之前先在本地跑通整条链路，git init 是最后一步——减少 initial commit 后的"deprecated file oops"。
2. 磁盘缓存 pipeline 适用于任何包含 LLM 阶段的数据处理。
3. 如果项目严肃到值得 leaderboard，就把自己的 `dev/LOG.md` 公开。




