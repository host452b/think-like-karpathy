# 深度洞察：意图 / 实验 / 反常识 / 思维模式

在 `CONCLUSIONS.md` 的基础上继续深挖。按"你说你要的角度"切分：

- **A. 意图三角测量**：4 个 repo 合起来暴露了 Karpathy 2026 的议程
- **B. 实验结果清单**：从 commits 里能捞到的、具体能引用的数字
- **C. 有意思的现象**：只有横看 4 个 repo 才能发现的对偶与矛盾
- **D. 反常识点**：和工程师默认直觉相反、但从 git 历史看确实奏效的选择
- **E. 思维模式目录**：给每种模式起名 + 证据 + 何时适用

---

## A. 意图三角测量

把 4 个 repo 按时间 / 产品身份 / 维护意图摆在一起，画出 Karpathy 2025–2026 的议程结构：

```
时间轴 ──→

2025-10      2025-12         2026-03          2026-04
  │            │                │                │
  ▼            ▼                ▼                ▼
┌──────────┐ ┌──────────────┐ ┌────────────┐ ┌──────────┐
│ nanochat │ │hn-time-capsule│ │autoresearch│ │  jobs    │
│ 严肃主线 │ │ 一日 vibe 玩具│ │预研 + 样板 │ │数据可视化│
└──────────┘ └──────────────┘ └────────────┘ └──────────┘
     │              │                │               │
     ▼              ▼                ▼               ▼
"开源 LLM    "LLM 作为       "agent 能做    "LLM 作为数据
 训练教学     历史判官"        ML 研究吗"     探索工具"
 能做到多简"
     │              │                │               │
     └──────────────┴────────────────┴───────────────┘
                          │
                          ▼
               一个统一议程：
       "2026 的 LLM/agent 能把过去需要
        机构 / 团队 / 几个月的事情，
        压缩成一个人 + 一晚上能做完的事"
```

**每个 repo 都是这个论点的一个样本**：

| Repo | 样本类型 | 论点 |
|---|---|---|
| nanochat | 基础设施 | 单节点 + 可读代码能复现 GPT-2 能力 |
| hn-time-capsule | LLM 应用 | 36 小时 vibe 能把"写个 pipeline 分析 HN 历史"这件事做完 |
| autoresearch | 代理能力 | 代理能在窄场景做出被主线项目采纳的改进 |
| jobs | 数据工具 | 两天能扒完 BLS 全部、打分、上线交互可视化 |

**看穿这个议程后，Karpathy 接下来会做什么是可预测的**：更多单人 × 短时间 × LLM 辅助 × 产生影响的样本，每个样本占据一个"曾经需要机构的活现在一个人做"的格子。他在证明一个时代假设。

**他不会做什么同样明确**：
- 不做 SaaS / 公司级产品（那是反议程）
- 不做重抽象的框架（litgpt 式）
- 不做单纯 demo（所有 4 个 repo 都有真实产出：nanochat 的 leaderboard 记录、jobs 的 live demo、hn-time-capsule 的 blog、autoresearch 被 nanochat 采纳）

---

## B. 实验结果清单（从 commits 里捞到的具体数字）

### B.1 nanochat 的加速拆解（6 个月 3.04h → 1.65h）

| Δ | 贡献技术 | 加速幅度 | commit |
|---|---|---|---|
| 0 | OpenAI 原版 GPT-2 (2019) | 168h | — |
| 1 | d24 baseline (Jan 29) | 3.04h | 348fbb3 |
| 2 | d26 + fp8 | 3.04h → 2.91h (**-4.3%**) | a67eba3 |
| 3 | batch size 1M tokens | 2.91h → 2.76h (**-5.2%**) | 2c062aa |
| 4 | FineWeb-EDU → ClimbMix | 2.76h → 2.02h (**-26.8%**) | 324e69c |
| 5 | autoresearch round 1 | 2.02h → 1.80h (**-10.9%**) | 6ed7d1d |
| 6 | autoresearch round 2 | 1.80h → 1.65h (**-8.3%**) | a825e63 |

**反直觉观察**：架构创新（fp8, batch tuning）的累计贡献 < 数据换血一次 + 代理 2 轮。**数据 > 精度 > 代理 > 批大小**。

### B.2 autoresearch 的 bug fix 分类（35 commits 中 8 条核心修复）

| 类别 | 数量 | 代表 commit |
|---|---|---|
| agent 卡住类（traceback / NaN / 无数据无限循环） | 4 | bd75534, 0be1e4f, ebf3578, 09ebea4 |
| 硬件兼容类（FA3 / ROCm / MLX / macOS） | 4 | 17b480a, 9264224, 7043095, 513fe6f |
| agent 行为约束类（soft language, results.tsv 不 commit） | 2 | ada84e5, 068d93d |

**发现**：agent 相关的 commits 占所有 non-doc commits 的 65%。<em>这个比例反过来证实：自主代理的工程瓶颈不在算法能力，在"不让它呆坐"</em>。

### B.3 hn-time-capsule 的 commit 密度

- 第 1 坐下（12/9 13:27–19:38，6 小时 11 分钟）：10 条 commits
- 平均间隔：37 分钟
- **最短连续间隔**：2 分钟（13:32 → 13:34，删除"oops deprecated file"）
- **最长连续间隔**：4 小时（15:27 → 19:32，期间发现 HN 403 → 加 retry）
- 第 2 坐下（12/10 07:42–08:59，1 小时 17 分钟）：5 条 commits

**意义**：这是"<5 分钟 commit 间隔 + 2 分钟 oops"级别的 LLM-pair-programming 节奏样本。任何想练这种节奏的人都可以把它当 reference。

### B.4 jobs 的关键时间沉默段

- 第 4 commit（3/14 15:39）之后 → 第 5 commit（3/15 20:10）前：**沉默 28.5 小时**
- 恢复后第 1 件事：`Redesign: top header layout with layer-synced stats`
- 第 3 件事：`1cca284 update the readme to minimize misunderstanding`

**推断**：中间 28.5 小时必然收到了公开反馈（可能是 X/HN）。没有哪个单人项目会主动在 36 小时内自己写出"我不是报告，我是工具"的自我更正。

### B.5 dev/LOG.md 的内容分布（nanochat）

- 总长度：1077 行
- 起始日期：2026-01-07（nanochat 已发布 3 个月后才开始写）
- 负面结果条目：明确标记 "Negative" / "Did not help" / "no benefit" 的至少 12 处
- 单次最大扫描（2026-03-24）：5 个 idea 全部失败
- scaling law 硬公式：WD ∝ 1/width² (α ≈ 1.97)

---

## C. 有意思的现象（只有横看 4 个 repo 才能发现的）

### C.1 autoresearch 和 hn-time-capsule 在 CLAUDE.md 上的对偶

- autoresearch：`program.md` 是核心资产，被 9 次修改，README 反复指向它
- hn-time-capsule：v0 就有 `CLAUDE.md`（102 行），30 小时后被主动删除，commit msg："people are free to init their own i think"

**这是一个人、同一个工具（Claude Opus），在不到 90 天里，给出两种完全相反的 agent-instruction 部署策略**。理由很微妙：
- autoresearch 的 agent 是"产品的一部分"（夜跑代理），skill 必须随产品一起分发
- hn-time-capsule 的 agent 是"作者的开发工具"（写代码的辅助），skill 应该随作者一起走

**学到的**：CLAUDE.md 该不该 commit 取决于 agent 在你项目里是"产品一部分"还是"开发工具"。两种都对。

### C.2 "私下开发 → 一次性 push" 在 4 个 repo 全部出现

| Repo | initial commit 行数 | 实际开发时间推测 |
|---|---|---|
| nanochat | 10,292 | 数月（tokenizer + Muon + 8 个脚本 + UI 不是一周能写的） |
| autoresearch | 2,848 | 数周（包括 spawn.sh 的多代理实验） |
| jobs | 656,087（含 html 原始页面） | 数天（scrape + score + site 全链路跑通） |
| hn-time-capsule | 2,073 | 几小时（README 自认 "vibe coded"） |

**没有一个 repo 以小的 MVP commit 开始**。4 个全部遵循"先跑通再 git init"。这和《Hackers and Painters》以来流行的 "commit early, commit often" 指南完全相反。

### C.3 第一条社区 PR 的出现时间

| Repo | 首条社区 PR 时间距 initial commit | PR 内容 |
|---|---|---|
| nanochat | +1 天（Bhaskar）+ +1 天（obxium）+ +1 天（Zach Mueller） | bpb 零除 / logo / speedrun.sh 修复 |
| autoresearch | +1 天（Marcin Bogdanski） | FA3 non-Hopper fallback |
| jobs | 无社区 PR（单作者 repo） | — |
| hn-time-capsule | 无社区 PR | — |

**现象**：Karpathy 明确想长期维护的 repo（nanochat / autoresearch）在 24 小时内就接到社区 PR；声明"不维护"的（jobs / hn-time-capsule）0 PR。**社区嗅觉比 README 声明更灵**。

### C.4 "负面结果"的分布

- nanochat `dev/LOG.md`：大量公开
- autoresearch：无（但 round 1/2 的 agent 否决过哪些实验没开源）
- jobs：README 里的 "What AI Exposure is NOT" 段落是一种负面声明
- hn-time-capsule：无

**唯一严肃记录失败的是 nanochat**。这对应它的唯一"严肃开源项目"身份。

### C.5 "把评测锁死"这条规则的 3 种表达

- nanochat：`runs/speedrun.sh` 中 CORE eval 的跑法被规范化，PR 必须报告 val_bpb 作为 CORE 噪声的 backup
- autoresearch：`prepare.py` 显式 read-only（`evaluate_bpb` 是 ground truth）
- jobs：`score.py` rubric 锁死（虽然鼓励 fork 改 rubric）

**这是 Karpathy 对"reward hacking" 的三层护栏**：①代码层（不可修改）②规则层（必须报告）③鼓励层（要改就 fork，不要改主干）。

### C.6 Twitter 引用作为 git 历史的一部分

- autoresearch README 引用 2 条 tweet（2029701092, 2031135152）
- nanochat README 引用 1 条（DeepWiki / 讨论区）
- hn-time-capsule README 引用 bearblog 短文
- jobs README 引用 live demo karpathy.ai/jobs

**每个 repo 都留了至少一条通向社交媒体的外链**。这些链接是理解项目原始意图的关键证据，但它们**不可被 `git log` 捕获**——未来任何做"纯 repo 考古"的工具都会漏掉这一层。

---

## D. 反常识点（与工程师默认直觉相反，但 Karpathy 这样做）

### D.1 "删除防护代码"
大多数工程师只加不减。nanochat 删除了所有 gradient clipping（2026-01-08 `dev/LOG.md`），原因：
- 测出 clipping 在任何 threshold 都无效
- grad norm 自然从不超过 1.0
- clipping 还会带来 2% all-reduce 开销
- "inactive protection + 2% slowdown" = 纯负收益，**安全感让位于实验证据**

学到的：看到代码里有"从来不触发的保护"，要么真的删掉，要么测出它触发过一次才保留。

### D.2 "5 分钟时间预算而不是固定 token 数"
大多数 benchmark 用固定 tokens（"训练 100B tokens"），便于对比。autoresearch 用 5 分钟墙钟：
- **不便**：不同 GPU 跑出来的 token 数不同
- **便利**：架构换算子 / 词表改动后 token/sec 不可比，但墙钟可比
- **隐含声明**：我们关心"相同预算下谁出结果更好"，不关心"同样 token 数谁 loss 更低"

学到的：选评测口径时，**选"用户真的关心的那个约束"而不是"传统上好对比的那个"**。

### D.3 "首日公开删除自己的 v0 设计"
autoresearch 的 `spawn.sh`（248 行，整个多代理协调层）在 initial commit 当天就被删除：
- 绝大多数作者会把 v0 的"野心版"留几天观察
- Karpathy 几小时内就下决定回撤，commit msg 坦白承认 "dam, erase experimental file from before that snuck through in my purge"

学到的：**承认错误的速度本身就是一种竞争优势**，不给后悔留时间。

### D.4 "一次性 27% 的数据红利值得 breaking change"
nanochat 2026-03-04 `324e69c` 把默认数据集从 FineWeb-EDU 换到 ClimbMix：
- commit msg 明说 "big, breaking change but large upside"
- 所有现有用户必须重新下载数据 shards
- 破坏的是"流畅升级"体验
- 换来的是：同样 CORE 目标下训练时间 2.76h → 2.02h

大多数开源项目会把它作为 opt-in flag 保留兼容性。Karpathy 直接切，**把"积累的 incremental 都还给你"当作默认**。

学到的：**当单次改动的收益超过 20%，值得打破兼容性**。小改要兼容，大改就把所有人一起推上新轨道。

### D.5 "在 commit msg 里写痛苦"
- nanochat `238353c`："document my struggle with fp8 integration yesterday, **it's not working like i thought it would and i suffered**. one day i will return to continue the fight."
- nanochat `a445144`："**dependencies bad bad bad**"
- autoresearch `1e207aa`："**dam**, erase experimental file from before that snuck through in my purge"

大多数 commit msg 是"fix X / add Y"式干瘪描述。Karpathy 在 commit msg 里留情绪。这做到两件事：
1. 让 git 历史可读（未来的你会感谢现在的你）
2. 让协作者意识到"这是人在跟真实问题搏斗"而不是无菌过程

学到的：**可以在 commit msg 里留情绪**，这不是不专业，这是叙事工程。

### D.6 "agent 要被当作初级工程师，不是资深专家"
autoresearch 的 `program.md` 写得像新员工入职手册：
- 必须先读这几个文件
- 不能改这些文件
- 不能装新包
- 遇到崩溃要读完整 traceback
- 用 `results.tsv` tab 分隔，不要用 comma

这和 2024 年主流的 "design a god-level prompt" 不同。Karpathy 假设 agent **会偷懒、会漏读、会陷入循环**，所以流程兜底。

学到的：**给 agent 的 prompt 应该假设它是最差 case，不是最好 case**。

### D.7 "LLM 是受众而不是工具"
jobs 的 `prompt.md`（45k token）是专门给 LLM 消费的 artifact：
- 不是 JSON（给机器）
- 不是文档（给人类全程读）
- 是 "人粘贴给 LLM + 跟它聊" 的一份聚合快照

大多数 repo 把 README / docs 当作 LLM 的 fallback 输入。Karpathy 反过来：**先设计 LLM 友好的 artifact，再让人使用它**。

学到的：**"LLM-native 文件"正在成为项目的一等产出**，和 README / docs 并列。

### D.8 "Ship 之前先跑通所有事"
4 个 repo 的 initial commit 全部是完整链路 push（而不是 MVP）。相反于敏捷开发的 "ship early, ship often"。

为什么这样做而不仅是"发布后再修"：
- Karpathy 的声誉让 initial push 天然会吸引很多眼球
- 把 MVP push 出去等于"公开发布半成品"
- 等整条链路跑通才 push，保护的是**注意力资产**

学到的：**注意力是一次性资源**。如果你的 repo 只会被评估一次（因为读者量级大），那等跑通再发布。如果你的 repo 需要长期迭代积累，早发布早得到反馈。

---

## E. 思维模式目录（命名 + 证据 + 何时适用）

把 Karpathy 的思维模式抽成 12 条可以直接挪用的 pattern。

### E.1 Shadow-drive → Public-drop（先私跑再发布）
- **证据**：4 个 repo initial commit 均为完整功能；`aa42f40` 把 rustbpe 从"内嵌"推到"独立 PyPI 包"同样遵循这条
- **何时适用**：你的 repo 有声誉级别的注意力、只会被初次评估一次
- **何时不适用**：你需要社区快速反馈（比如冷启动 API 设计）

### E.2 Single-dial Scaling（一个旋钮驱动所有）
- **证据**：nanochat `--depth`、autoresearch 5min budget、jobs 0-10 rubric
- **何时适用**：需要 minimize cognitive load 的用户面场景
- **陷阱**：单旋钮会隐藏正交性缺失——所以必须显式说明哪些次级参数"自动派生"

### E.3 Explicit Over Magic（显式优于魔法）
- **证据**：2026-03-04 autocast 在 11 个文件里被删掉，换 `COMPUTE_DTYPE` 全局
- **何时适用**：你需要长期调试的核心路径
- **代价**：更多样板代码，但可 grep、可 debug

### E.4 Guard Rails, Not Railroads（软约束 > 硬约束）
- **证据**：autoresearch 的 "VRAM 是 soft constraint"、nanochat 的 "simpler is better (qualitative)"
- **何时适用**：agent 驱动或 exploratory 场景
- **何时不适用**：安全关键（金融、医疗）——此时要 railroad

### E.5 Subtract Then Decide（先减后判）
- **证据**：删除所有 gradient clipping、删除 spawn.sh、删除 CLAUDE.md、删除 matplotlib from default deps
- **何时适用**：任何存在感模糊的代码（"有了好像也没用"）
- **方法论**：先删，如果 3 个月后没人想起它，永远删掉；有人想起，再加回来并加测试证明价值

### E.6 Wall Clock Over FLOPs（现实优于理论）
- **证据**：nanochat 的 time-to-GPT-2、autoresearch 的 5 分钟、hn-time-capsule 的 36 小时
- **何时适用**：任何最终交付给人类使用的场景
- **反例**：如果你在做 scaling laws 研究，FLOPs 才是正确口径

### E.7 Disk Cache as Interface（磁盘作为阶段边界）
- **证据**：hn-time-capsule 5 阶段 / jobs 6 阶段 / nanochat checkpoint_manager
- **何时适用**：任何含"昂贵阶段 + 便宜阶段"的 pipeline
- **额外益处**：阶段产物可以 grep / git diff，调试极友好

### E.8 Leaderboard as Social Infrastructure（排行榜作为社群引擎）
- **证据**：nanochat `dev/LEADERBOARD.md`，继承自 modded-nanogpt
- **何时适用**：项目已有清晰的单一指标 + 可重复的评估流程
- **危险**：leaderboard 会吸引过拟合 → 必须规定"泛化性"作为 PR 评审标准

### E.9 Module Split by Complexity Type, Not Size（按复杂度类型切模块）
- **证据**：`aa42f40` rustbpe 剥离（效率型复杂度可隐藏，算法型必须暴露）
- **何时适用**：repo 里出现"又大又不用改"的模块
- **方法论**：问自己"这个模块的复杂度来自算法还是性能？"算法 → 留；性能 → 抽离

### E.10 LLM as First-Class Audience（LLM 作为一等受众）
- **证据**：autoresearch `program.md`、jobs `prompt.md`、nanochat `dev/LOG.md`
- **何时适用**：你预期用户会用 LLM 读你的 repo
- **具体做法**：专门准备一份给 LLM 消费的聚合文档（不只是把所有 .md 链一遍）

### E.11 Private Reforks Encouraged, Main Minimal（鼓励私人 fork，主干保持最小）
- **证据**：autoresearch "notable forks" 段落、`results.tsv` 保持 untracked、jobs 鼓励 "fork 来换 prompt"
- **何时适用**：你希望 repo 成为 pattern 而非产品
- **反 pattern**：让所有功能都进主干 → 主干变肿，fork 价值稀释

### E.12 Narrate the Struggle（叙事工程 / 在 commit 里留情绪）
- **证据**：`i suffered`、`dependencies bad bad bad`、`dam`
- **何时适用**：任何希望 git 历史未来可读的场景
- **规则**：情绪要具体（"i suffered" 后面要跟原因），不要当抱怨发泄

---

## F. 收尾：一个可检验的预测

基于 A 节的意图三角测量，可以预测 Karpathy 2026 下半年到 2027 上半年大概率做的事：

1. **autoresearch round 3+**：继续让代理压 nanochat 的 time-to-GPT-2。预测：~1.3h 以内。
2. **一个面向推理的极简 repo**（"nanochat 但用于 serving"？）：他目前没碰过"如何让一个小模型高效 serve"这个格子。
3. **program.md 体例的独立抽取**：很可能以一个小 repo（"karpathy/skill-markdown"？）形式发布，作为 program.md 的模板集合。
4. **不会做**：大模型 training（> 70B）、多节点 orchestration、企业级特性。

如果以上 1-3 在 6 个月内任何一条出现，意图三角测量就有一个实证；如果都不出现，说明我读错了信号。

---

*生成于 2026-04-17。这份文件和 `CONCLUSIONS.md` / 4 份 `report.html` / `CHANGELOG.md` 共同构成完整研究输出。*
