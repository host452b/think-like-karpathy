# 实战手册：做 AI 工程 / 实验 能从 Karpathy 学到什么

> 基于 4 个 repo (nanochat / autoresearch / jobs / hn-time-capsule) 的 git 考古，提炼成**遇到 X 时做 Y** 的可操作建议。每条都带 commit 证据，不讲抽象哲学。
>
> 想看模式名字（Shadow-drive→Public-drop 等 12 条）读 [`INSIGHTS.md`](INSIGHTS.md)。
> 想看整体结论读 [`CONCLUSIONS.md`](CONCLUSIONS.md)。
> 想动手操作读**这一份**。

---

## 四条基本宪法

先把 Karpathy 工作风格的 4 个隐性前提写在纸上，后面的所有建议都是它们的推论：

1. **简单能读懂 > 功能齐全**（`aa42f40` 剥离 rustbpe 只因为"一个项目里嵌一个项目太 ugly"）
2. **墙钟时间 > 理论 FLOPs**（所有 4 repo 的主指标都按墙钟算）
3. **证据 > 直觉**（nanochat 删 gradient clipping 因为测出从不触发）
4. **叙事可读 > 工程师审美洁癖**（commit msg 里写 "i suffered" / "dependencies bad bad bad"）

---

## 一、项目启动阶段

### 建议 1.1 找一个"独占格子"，不要做通用工具
**Karpathy 的例子**：nanochat = "完整 ChatGPT 链路 × 单节点 × 可读" — 这个三交集的格子在 2025 年确实空着；修改任何一个维度它就不独占了（litgpt 广但不可读，llm.c 单文件但只做 pretrain，nanoGPT 单文件但没有 SFT/RL）。

**可操作**：
- 写下 3 个正交维度，你的项目必须在交集里是**唯一的**
- 不确定是不是独占 → grep GitHub awesome-list；如果已有 5+ 同类 repo，你的定位要更窄
- 不要想着"做得比别人更好"——想着"做别人都没做的这个 specific 组合"

### 建议 1.2 在写代码之前先选发布姿态
**Karpathy 的 4 档对照**：

| 姿态 | 代表 repo | 标配 | 成本 |
|---|---|---|---|
| 严肃开源 | nanochat | leaderboard + `dev/LOG.md` + 长期维护 | 每周 10+ 小时 |
| 样板预研 | autoresearch | `program.md` 是核心 + 中期维护 | 每周 2-5 小时 |
| 工具 demo | jobs | live demo + live README 但不承诺更新 | 基本 0 |
| vibe code | hn-time-capsule | README 首段 "I don't intend to support it" | 0（声明后 issue tracker 会自愈） |

**可操作**：
- 在第 1 行代码前选定 1 档，否则写到一半换姿态极痛苦
- 选错会被 issue tracker 拖死（或反过来：发出去后被严肃对待但你没精力响应）
- 最常见错误：写的时候当"样板"，发的时候硬撑当"严肃开源"

### 建议 1.3 首日决定 5 件事（写 README 之前）
1. **单一指标**是什么？（不能是 3 个，必须是 1 个；其他指标作为 noise guard）
2. **时间预算**是什么？（autoresearch 的 5 分钟；你的项目可能是 10 分钟 / 1 小时 / 1 晚）
3. **什么文件是 read-only**？（autoresearch 的 `prepare.py` 是 reward-hacking 物理护栏）
4. **什么文件给人看、什么给 LLM 看**？（jobs 的 `prompt.md` vs autoresearch 的 `program.md`）
5. **什么 cache 是磁盘 interface**？（hn-time-capsule 5 阶段的产物都在 `./data/<date>/stage_X/`）

不写下来的话，写到中期发现结构不对再重构成本高 3 倍。

---

## 二、写代码阶段

### 建议 2.1 用磁盘缓存 pipeline 模板
**证据**：hn-time-capsule 5 阶段 / jobs 6 阶段 / nanochat 的 `runs/` + `checkpoint_manager.py`。

```
stage_fetch    → ./data/<date>/raw/*.json        便宜（网络）
stage_process  → ./data/<date>/clean/*.md        便宜
stage_analyze  → ./data/<date>/responses/*.txt   ★ 昂贵（LLM/GPU）
stage_parse    → ./data/<date>/parsed/*.json     便宜
stage_render   → ./out/<date>/report.html        便宜

+ 一个 `clean <stage> [--item ID]` 子命令按阶段或按单条抹缓存
```

**可操作**：
- 任何含"昂贵阶段"的 pipeline（LLM 调用、GPU 训练、长爬虫）都必须做成阶段化磁盘 cache
- 每阶段一个 subcommand（argparse 子命令），让重跑只重跑后段
- 阶段产物用人类可读格式（JSON / Markdown / CSV），方便 `grep` / `cat` / `git diff`
- 投入 1 天，回报终身——你的调试期会便宜得离谱

### 建议 2.2 按"复杂度类型"切模块，不按行数
**Karpathy 的规则**（nanochat `aa42f40` 剥离 rustbpe 时明说）：

| 复杂度来自 | 怎么处理 |
|---|---|
| **算法/设计**（每一行 reader 都要读） | **留在主干**。即使 500 行。例：`base_train.py` 339 行 |
| **性能/效率**（没人关心内部实现） | **抽成独立包**。例：rustbpe 476 行 Rust 扔到独立 repo |

**可操作**：
- 面对一个大文件/目录，问："读懂这个项目需要理解它吗？"
  - 是 → 留在主干，哪怕大
  - 否 → 拆成独立 package，让主干精神集中

### 建议 2.3 单旋钮驱动所有（设计 API 时的默认）
**证据**：nanochat 的 `--depth` / autoresearch 的 5 分钟 / jobs 的 0-10 分制。

```python
# 反面（大多数框架）：
train(depth=24, width=1024, heads=16, lr=3e-4, wd=0.1, warmup=1000, ...)

# 正面（nanochat 风格）：
train(depth=24)
# 其他全部从 depth 按 scaling law 自动派生
# 高级用户可以 override，但默认不用想
```

**可操作**：
- 对任何"调参 API"，选一个最主要的旋钮，把其他参数按经验公式派生
- 在 docs 里显式列出派生公式（nanochat 的 `WD ∝ 1/width²` 就是这样）
- 允许 override 但不鼓励

### 建议 2.4 显式优于魔法（特别是训练代码）
**证据**：nanochat 2026-03-04 在 11 个文件里删掉所有 `torch.amp.autocast`，换 `COMPUTE_DTYPE` 全局。commit msg：
> "autocast is 'magic we don't control' — it silently decides which ops run in which precision via internal allowlists."

**可操作**：
- 魔法（上下文管理器 / 装饰器 / 自动转换）在 notebook 里好用，在需要长期调试的核心路径要显式替换
- 写完后问："如果 3 个月后我再读这段，能不能 grep 到每一个 dtype 变化？"——不能就改。

### 建议 2.5 减法优先
**证据**：nanochat `dev/LOG.md` 2026-01-08 删掉所有 gradient clipping：
> "Deleted all grad-clip code paths. The code naturally produces well-behaved gradients. This improves a bit of MFU because we don't have to calculate and sync grad norms."

**可操作**（逆向思维）：
- 扫一遍代码，找"从来不触发的保护 / 从来走不到的 branch / 一直是 False 的 config flag"
- 先删，测试跑绿后观察 1 周。没人投诉 → 永久删除
- 不要因为"删了怕出事"保留——"保留怕出事"的概率等于"删了出事"的概率，而前者带 2-5% 运行开销

---

## 三、AI 实验特有的建议

### 建议 3.1 评测口径优先选"你真的关心的约束"
**对比**：

| 约束 | 何时合适 |
|---|---|
| Fixed tokens (`100B tokens trained`) | 做 scaling laws 研究 |
| Fixed wall clock (`5 min / 2 hours`) | 做"在预算内谁赢" |
| Fixed compute (`1×H100-hour`) | 做跨硬件公平对比 |

**Karpathy 的选择**：nanochat 用 wall clock，因为"users care about 'how long until my model is usable'"。autoresearch 用 wall clock 因为"tokens/sec 在架构改动后不可比"。

**可操作**：在跑第一组实验前，问：**最终用户 / reader 真的会看哪个数字？**—— 用那个。

### 建议 3.2 三层 reward hacking 护栏
**证据**：nanochat / autoresearch / jobs 都有类似结构。

1. **代码层**：评估代码 read-only（`prepare.py` 的 `evaluate_bpb` 是 ground truth；agent / PR 不能改）
2. **规则层**：leaderboard PR 必须同时报告主指标 + noise guard（CORE + val_bpb）
3. **鼓励层**：想改评估 → fork 自己玩，不要改主干

**可操作**：任何有指标的实验平台，至少做 1-2 层。如果只能选一层，选代码层。

### 建议 3.3 实验日志模板（抄 nanochat `dev/LOG.md`）
```markdown
## YYYY-MM-DD: 标题

### 动机 / Motivation
（为什么想试这个？从哪里看到的想法？）

### 改了什么 / What changed
（具体 diff 级别的变化，哪个函数、哪个参数）

### 结果 / Results
（含数字 + noise 范围。对比 baseline。最重要：**负面结果照写**）

### 结论 / Conclusion
（keep / discard / crash。keep 的话默认 on 还是 opt-in？）
```

**可操作**：
- 第一个月强迫自己写 5 条。之后会成习惯
- **负面结果比正面结果信息量大**（nanochat LOG 里有 12+ 条"did not help"）
- 写给"3 个月后的你自己"——那时候你想不起来为什么这里用 0.2 而不是 0.1

### 建议 3.4 改动要验证跨规模泛化
**证据**：nanochat 的 `dev/LEADERBOARD.md` 写明 PR 评审规则：
> "your change must be principled enough that it can easily generalize to other model depths, so that we can sweep out a miniseries."

**可操作**：
- 一个改进只在某 specific scale 有效 → 疑似 overfit，不要 merge
- 至少在 2-3 个 scale 验证（小 / 中 / 目标）
- 把跨 scale 结果放在 PR 描述里

### 建议 3.5 大改值得 breaking change
**Karpathy 的阈值**（从 `324e69c` 反推）：

| 收益区间 | 怎么发 |
|---|---|
| < 5% | 必须 opt-in 保兼容 |
| 5-20% | 看项目阶段 — 早期破 / 成熟保 |
| **> 20%** | **直接 breaking change**，所有用户一起搬 |

**证据**：nanochat 换 ClimbMix 数据集一次性 27%，commit msg 说 "big, breaking change but large upside"。所有用户必须重新下载数据。

**可操作**：不要为了兼容性把每次大胜都压成渐进改动。积累 3 次小改的维护成本 > 1 次大改的迁移成本。

---

## 四、Commit 实践

### 建议 4.1 分钟级迭代节奏（LLM-pair-programming）
**证据**：hn-time-capsule 首日 10 commits 在 6 小时 11 分钟内，中位间隔 < 10 分钟。

```
user 描述需求 (30 秒)
→ agent 写 (2-5 分钟)
→ user 验证 + 小改 (2-5 分钟)
→ commit (30 秒)
→ 下一个需求
```

**可操作**：
- 不要"让 agent 一下做 3 个需求"——diff 会被污染，回退困难
- 不要"攒一天的改动再 commit"——agent 的错误累积到无法追溯
- 把 commit 当作 undo 粒度，而不是"完成一个 milestone"的里程碑

### 建议 4.2 Commit msg 叙事工程
**Karpathy 样例**：
- `i suffered` + 具体上下文（fp8 integration yesterday）
- `dependencies bad bad bad`（有态度 + 可 grep）
- `dam, erase experimental file from before that snuck through in my purge`（承认错误 + 具体）
- `incredible`（罕见情绪标记，后面必须有惊叹理由）

**可操作**：
- 允许情绪，必须具体。`frustrated` 不行，`i suffered fp8 integration` 可以
- 失败也要写。`tried X, did not help, grad norm never exceeds 1.0` 比 `wip` 信息量高 100 倍
- 目标用户：3 个月后的你自己 / 加入项目的新同事 / 做 git 考古的 AI

### 建议 4.3 首日 / 首周留清理时间
**证据**：
- autoresearch: `1e207aa "dam, erase experimental file from before that snuck through in my purge"`
- hn-time-capsule: `739f101 "deprecated file oops"`
- nanochat: 10-16 的 WebUI 一系列收尾 fix

**可操作**：
- 发布后第 1-3 天的 commits 大概率是"清理私跑期的脏文件"，不是 bug。别慌
- 明确预留 24-72 小时不接受 feature PR，只做清理
- 在 README 写 "repo stabilizes in ~1 week after public push"

---

## 五、Agent / LLM 协作

### 建议 5.1 分权宪法（跑在本地的 agent 工作流）
**autoresearch 的模板**：
```
人类 → 改 .md   (context / rules / skill)
agent → 改 .py   (单文件 / 窄场景 / 短迭代)
评估脚本 → 只读 (防 reward hacking)
results.tsv → untracked (避免分支冲突)
baseline commit → 锁死不动
```

**可操作**：
- 新建项目时这 5 条边界在 Day 0 就划好
- agent 不能装包、不能改评估、不能改 baseline
- 实验结果写磁盘（不入 git），因为它一定会被代理反复覆写

### 建议 5.2 把 agent 当初级工程师，不是资深专家
**Karpathy 隐含的 agent 失败模型**：
- agent **会**漏读 README / CLAUDE.md
- agent **会**在错误栈上只看最后一行，跳过 root cause
- agent **会**陷入循环（无数据时反复评估）
- agent **会**偷懒合并 section（你让它出 6 条，它给 4 条）
- agent **会**对格式漂移无感（output 多了一个数字前缀，下游 parser 就炸）

**`program.md` 的应对**（每条都是 defensive default）：
- 强制"**先**读 README + 所有 context 文件"
- 强制"崩了要读**完整** traceback，不是最后一行"
- 明确列出"**不能**装新包 / **不能**改 prepare.py"
- 显式说"6 sections，不是 '大致几节'"
- parser 正则加前缀容错（`76da7d5` 加 `\d+[\.\)]\s*` 前缀）

**可操作**：
- 写 skill 文件时假设 agent 是**最差 case**，不是最好 case
- 每遇到一次 agent 失败模式，往 skill 里加一条具体兜底

### 建议 5.3 为 LLM 单独准备 artifact
**jobs 的 `prompt.md`** 是典型：一份 45k token 的聚合 Markdown，专门给"人粘贴到 LLM 对话里"的用例。不是 JSON、不是 API docs。

**autoresearch 的 `program.md`** 是另一种：给 agent 当 skill。

**CLAUDE.md**（hn-time-capsule 删了）是第三种：agent 指令。

**可操作**（2026 新兴模式）：
- 明确每个 .md 的**受众**：人 / LLM（对话场景）/ agent（执行场景）
- 不要混用（不要让一份 README 同时给人和给 LLM 读）
- 如果你预期项目会被 LLM 读，准备一份专门的聚合快照

### 建议 5.4 agent 产出的改进需要单独解释
**现存问题**：nanochat 的 autoresearch round 1/2 产出了"smear"和"backout"两个架构块，但 `dev/LOG.md` 里没有独立章节解释它们为什么 work。半年后没人能说清。

**可操作**：
- agent 写完代码后，**人类**写一条 LOG.md 条目解释 "why it works"
- 如果你和 agent 都说不清，这个改进要存疑——可能是 overfit specific scale
- 不要让 "代理跑出来的" 成为免解释的通行证

---

## 六、发布阶段

### 建议 6.1 README 第一屏回答 3 个问题
**jobs 的教训**（`1cca284` 的反转）：如果第一屏不说清楚"不是什么"，互联网会用 1 句话帮你总结，而那句话大概率是错的。

README 前 300 字必须回答：
1. 这是什么（一句话）
2. 这**不是**什么（避免被 1 句话误总结）
3. 你会不会维护（一句明确声明）

**反面教材**："AI Exposure of the US Job Market" — 30 小时后被社区质疑，被迫改成"US Job Market / a research tool for exploring BLS data"。
**正面**：hn-time-capsule "99% of this repo was vibe coded in a few hours with Opus 4.5. Code is provided as is and I don't intend to support it."

### 建议 6.2 initial commit 是完整链路，不是 MVP
**证据**：4/4 repo 的 initial commit 都是 complete pipeline。nanochat 47 文件 10,292 行；autoresearch 10 文件 2,848 行。

**为什么这样对**（不是普遍原则，是 Karpathy 特定语境）：
- 有品牌的作者，initial push 只能被评估一次（注意力不可复用）
- 半成品发布 → 观众感知"这个还没 ready" → 以后再完善也吸引不回来
- 没有品牌的作者这条规则要倒过来：尽早发布拿反馈

**可操作**：
- 判断自己的注意力资产：**这次发布能进 Hacker News 首页吗？** 能 → 跑通再发；不能 → 早发早迭代
- 不确定 → 跑通再发

### 建议 6.3 "大数字"会被媒体简化
**Frey-Osborne 踩过的雷**（jobs 主动避开）：一篇 2013 论文严谨说"47% probability of automation"，被媒体压缩成"47% jobs will disappear"，原作者后 10 年都在修正。

**可操作**：
- 不要在 README 第一屏放可被 1 句话引用的大数字（如"80% of jobs"、"300M affected"）
- 如果数字必须放，紧接着写"What this number is NOT"
- jobs 的 "Software developers score 9/10 because AI is transforming their work — but demand for software could easily grow" 是教科书级别的自我 hedging

### 建议 6.4 主动列 notable forks
**Karpathy 做法**：autoresearch 和 nanochat 都有 "notable forks" 章节，列跨硬件 fork（MLX / ROCm / macOS）。

**信号作用**：告诉潜在贡献者"这个 repo 欢迎私分叉，不垄断 narrative"。降低 contributor 心智负担。

**可操作**：至少列 1-3 个 fork，即使你觉得它们不够"正式"。fork 本身就是最好的 vote of confidence。

---

## 七、反 pattern 警示（Karpathy 主动避免的事）

### 7.1 永远不做的（即使有人提 PR）
- 重抽象框架（accelerate / DeepSpeed / Lightning 级别）
- 多节点 orchestration（k8s / slurm）
- 模型架构多样性（MoE / Mamba / Jamba 等并存）
- Enterprise 特性（SSO / RBAC / audit log）
- 详细 CLI argparse help（README 代替）

**理由**：每一条都会扩大维护表面积，稀释独占格子。

### 7.2 唯一例外：硬件多样性
CPU / MPS / ROCm / MLX fallback **值得做**——因为这是**准入门槛**问题，不是能力扩张。门槛每抬高一档用户就流失一大块。

### 7.3 "看起来应该加的"要抵抗
- "加个 log level 吧"——只加 INFO 一条 logging，不做 log level
- "加个配置文件吧"——CLI 参数够用
- "加个 test suite 吧"——只为最 tricky 的部分写 test（nanochat 的 `tests/` 只覆盖 engine / rustbpe）
- "加个 web demo 吧"——有必要再加（nanochat 有因为是 chat，jobs 有因为是可视化；autoresearch 没有）

---

## 八、一页纸 Checklist

### 项目启动（Day 0）
- [ ] 独占格子已定义（3 个维度交集）
- [ ] 发布姿态已选（严肃开源 / 样板 / 工具 demo / vibe code）
- [ ] 单一指标 + 时间预算写进 README
- [ ] read-only 文件清单已列出
- [ ] pipeline 阶段边界（磁盘 cache）已画
- [ ] 每个 .md 的受众已明确（人 / LLM / agent）

### 首次发布（Day 1）
- [ ] initial commit 是完整链路（能跑通整条流水线）
- [ ] README 第一屏 = 是什么 + 不是什么 + 维护意图
- [ ] 预留 24-72 小时清理"oops deprecated"类 commits
- [ ] 有 1-2 条外链（tweet / blog / live demo）
- [ ] 没有可被 1 句话误总结的大数字裸露

### 第一周维护
- [ ] 响应第一批社区 PR（大概率是 hardware / runtime 修复）
- [ ] 警觉 agent runtime 盲区（traceback / NaN / 无数据）
- [ ] 写第一条 `dev/LOG.md` 条目（哪怕只有 3 行）

### 每次 AI 实验
- [ ] LOG.md 四段式：动机 / 改了什么 / 结果（含负面）/ 结论
- [ ] 跨 scale 验证（至少 2 个 depth/size）
- [ ] noise guard（至少 2 个指标同时 report）
- [ ] 不泛化的改进不 merge

### 发 leaderboard / benchmark 前
- [ ] 有至少 2 个 baseline
- [ ] 评测锁死 + 3 层护栏（代码 / 规则 / 鼓励）
- [ ] 泛化性写进 PR 评审标准
- [ ] PR msg 模板要求显式报告 noise 范围

### 使用 agent 辅助时
- [ ] 分权宪法 5 条写进 `program.md` / `CLAUDE.md`
- [ ] agent 每次失败都往 skill 里加一条兜底
- [ ] agent 产出的代码必须有**人类**写一段 "why it works" LOG

---

## 结语：学一个人的风格 vs 抄一个人的风格

Karpathy 的工作方式**不是通用最优解**——它高度依赖：
- 已有品牌（注意力资产可用）
- 深厚技术栈直觉（能直接判断哪里该减哪里该加）
- 长期的"可读"审美训练（写 minGPT / nanoGPT / llm.c 积累的）

所以：
- **能抄**的：pipeline 模板、dev/LOG.md 体例、分权宪法、checklist、commit msg 叙事风格
- **不能抄**的："首日回撤 v0" 这种决策需要足够判断力 / "private-drop 不要 MVP" 只适合有品牌的作者
- **要学其精神**：证据 > 直觉、减法优先、文档是给具体受众写的（人 / LLM / agent）

最重要的一条：**所有这些 pattern 都是为了压低"认真工作"的门槛**。不是为了看起来像专家，不是为了发 HN，不是为了拿 star。

---

*如果你从这份手册挑一个立刻做，选**磁盘缓存 pipeline 模板**——投入 1 天，回报终身。*
