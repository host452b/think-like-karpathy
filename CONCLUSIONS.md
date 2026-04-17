# 综合研究结论（跨 4 repo）

读完 `autoresearch` / `hn-time-capsule` / `jobs` / `nanochat` 后，把"单个 repo 内部结论"之外的、跨 repo 才能看见的东西落纸。

详细每 repo 分析见对应目录下 `report.html`，过程日志见 `CHANGELOG.md`。

---

## 一、Karpathy 2025–2026 的"开发宪法"（4 repo 共现的模式）

| 模式 | 出现处 | 本质 |
|---|---|---|
| **磁盘缓存 pipeline** | hn-time-capsule 5 阶段 / jobs 6 阶段 / nanochat 的 `runs/` | 每阶段产物是文件 → 昂贵阶段（LLM 调用、GPU 训练）一旦跑过永不重跑 |
| **LLM-as-audience artifacts** | autoresearch `program.md` / jobs `prompt.md` / hn-time-capsule CLAUDE.md | repo 里有专门给 LLM/agent 看的文件，不是给机器解析 |
| **单一旋钮 + 单一指标** | nanochat `--depth` + CORE / autoresearch 5min + val_bpb / jobs 0-10 AI exposure | 其他超参全部从这一个派生，评测是一个数字 |
| **initial commit = 私下跑完后一次性 push** | 4 个 repo 全部 | git 历史不反映开发过程，反映的是"发布策略" |
| **按复杂度类型切模块** | nanochat `aa42f40` 剥离 rustbpe | 效率型复杂度可隐藏（独立包），算法型复杂度必须暴露 |

---

## 二、发布姿态光谱（4 repo 正好对应 4 档）

```
重 ← 不同"使用 agent 的意图" → 轻
─────────────────────────────────────────────────
 nanochat      autoresearch    jobs         hn-time-capsule
 严肃开源       样板+预研仓      一次性工具    36h vibe code
 leaderboard   program.md=     live demo    CLAUDE.md 被删
 长期维护       核心资产         只 README    明确不维护
```

**使用方法**：判断自己下一个项目属于哪档 → 决定要不要公开 `dev/LOG.md` / 要不要保留 `CLAUDE.md` / 要不要搞 leaderboard / 要不要声明 "I don't intend to support it"。选错了姿态会让维护心智负担失控。

---

## 三、最重要的一条 meta-insight

**autoresearch round 1/2 → nanochat leaderboard #5 #6 的闭环**是 2026 年 autonomous AI research 唯一一个干净的 proof point：

- 不是 demo 论文
- 不是 benchmark 游戏
- 是代理产出的代码被 merge 进主线项目的 PR 记录（commits `6ed7d1d`, `a825e63`）
- 从 2.02h 压到 1.65h（18% wall-clock 加速）

这是未来 1-2 年行业会被反复引用的案例。

---

## 四、每个 repo 的一句话速记

| repo | 一句话 |
|---|---|
| **autoresearch** | "人改 md / agent 改 py"的分权宪法 + 全部早期 commits 都在堵 agent runtime 的呆坐盲区 |
| **hn-time-capsule** | 36h 两次坐下 vibe code；`76da7d5 tune prompt again` 一条 commit 示范 LLM 应用工程三件套（结构约束 / grounding / parser 宽容） |
| **jobs** | `1cca284` 的 README 定位反转（AI Exposure 分析 → BLS 探索工具）比任何代码都有价值；`prompt.md` = "LLM-as-audience" 新设计模式 |
| **nanochat** | 6 个月 3.04h→1.65h 的速度战争；杠杆顺序 = **数据换血 > fp8 > 代理注入 > 批大小自动化**；`dev/LOG.md` 是最稀缺产物 |

---

## 五、你能做什么（5 条，按 ROI 排序）

### 1. 立刻偷 5 阶段 pipeline 模板
下一个涉及 LLM/GPU 的数据处理项目，直接抄 hn-time-capsule 的 `fetch→prompt→analyze→parse→render` 骨架。每阶段一个 subcommand + 一个 `clean` 子命令。
- **成本**：1 天
- **回报**：调试期几乎免费（昂贵阶段的缓存永远可保不动）

### 2. 给你的 repo 起一份 `dev/LOG.md`
抄 nanochat 的实验日志体例：
```
## YYYY-MM-DD: 标题

### 动机 / Motivation
### 改了什么 / What changed
### 结果 / Results
### 结论 / Conclusion
```
**关键：公开负面结果**。nanochat 的 LOG 有整章是"这 5 个想法试了都失败"——对后来者最有价值。

### 3. NVIDIA 特有的机会（你在 NVIDIA）
nanochat 明确已经绑向 **ClimbMix + fp8 + H100**，你的团队离 ClimbMix / NeMo Curator / H100/Blackwell 内部数据非常近。能做：
- 提交 round 3 speedrun PR（ClimbMix 之后还有数据红利空间）
- 给 Blackwell 更新 kernel（当前 fp8 路径还是 Hopper-centric）
- 推动下一代 pretraining 数据集进 speedrun

这 3 件都是 nanochat 社区"等着有人做"的事，而 Karpathy 本人拿不到 NVIDIA 内部数据。

### 4. 把 `program.md` skill-as-markdown 样板用到自己项目
任何希望 agent 帮你过夜迭代的私有项目，抄 autoresearch 的分权宪法：
- 人类改 md（给 context 和规则）
- agent 改 `.py`（单文件实验）
- 评估脚本锁死只读（防 reward hacking）
- 结果表 `results.tsv` 留 untracked（避免分支冲突）

这个范式 2026 年还没被广泛使用，先用的人有红利。

### 5. 用 hn-time-capsule 式 "36h 一日工具"练手
不是为了产品——为了训练"如何用 Claude/Codex 把一个想法在一个周末变成可展示 artifact"的肌肉。

关键姿态是最后写清 "Vibe code alert, I don't intend to support it"——这决定了你的维护心智负担。没有这一句，issue tracker 会把你拖死。

---

## 六、留给自己的问题

看完 4 个 repo 后还没想清楚的问题（可能值得后续深入）：

1. **代理改进的可解释化**：autoresearch 产出的 smear/backout 如果成为 nanochat 架构常驻块，6 个月后还能有人解释为什么吗？LOG.md 得专门补一章。
2. **硬件生态绑死 vs 多样化**：nanochat 当前把 NVIDIA 当默认，社区 fork 要走多远才能在 AMD/Apple/TPU 上也能速通？
3. **LLM-as-audience 模式的边界**：哪些 artifact 真的需要单独给 LLM 准备一份？哪些是过度工程？（例如 `prompt.md` 45k token 的聚合是合理的，但把每篇博客都 dump 成 LLM 快照就过了。）

---

*生成于 2026-04-17。基于公开 git history / README / dev/LOG.md / dev/LEADERBOARD.md。*
