# Skill: Iterative Experiment Project

> **通用 "agent 过夜迭代实验" skill**，基于 autoresearch `program.md` + nanochat 的经验提炼。
>
> **使用方法**：放到你的迭代实验 repo 根目录（或复制为 `program.md`）。让 agent 读这份文件 + 你的 README + 你的单文件实验代码 + 你的评估脚本，就能自主开始迭代。
>
> **适用场景**：有一个单一目标指标、有一个时间/资源预算、有一个可改的代码文件、有一个可信的评估——任何 4 条都满足的项目。典型例子：训练跑分、算法速度比赛、prompt 调优、参数搜索。

---

## 一、第一次启动（Setup）

读这份 skill 后，你必须先和人类/自己确认以下 6 件事：

1. **确认 run tag**：基于今天日期提一个标签（例 `mar15`）。分支名 `<project>/<tag>` 必须是全新的（不可复用）。
2. **创建隔离分支**：`git checkout -b <project>/<tag>` from master。所有改动只在这个分支做。
3. **完整阅读以下文件**（一个都不能跳）：
   - `README.md` — 项目背景
   - 你的**评估脚本** — 这是 ground truth，**不可修改**
   - 你的**实验代码**（通常是单文件，例 `train.py`）— 你会反复编辑它
   - 任何 constants / configs — 了解哪些是固定的
4. **验证数据/依赖存在**：确认所需数据、tokenizer、模型、API key 都就位。不在 → 告诉人类先跑 setup。
5. **初始化结果表 `results.tsv`**（TSV 而非 CSV，因为描述里可能含逗号）。表头：
   ```
   commit	primary_metric	secondary_metric	status	description
   ```
6. **先跑 baseline**：第一轮不要改任何代码。跑一次当前版本建立基线数字。**绝不跳过 baseline**。

---

## 二、分权宪法（Hard constraints）

### 你可以做 ✅
- 改 `<experiment_file>`（主实验文件，通常一个 `.py`）。架构、超参、优化器、数据处理、训练循环——都是公平游戏。

### 你不可以做 ❌
- 不可修改**评估脚本**（它是 ground truth，改了就作弊）
- 不可修改**固定 constants**（time budget / sequence length / seed 等）
- 不可装新包（只用现有 `pyproject.toml` / `requirements.txt` 里的依赖）
- 不可动**基线 commit**（必须始终能回到起点）
- 不可把 `results.tsv` 加到 git tracking（它是本地只在的、会被你反复重写的状态）

### 软约束
- **资源是 soft constraint**（VRAM / API quota / 时间）。有意义的收益可以适度吃资源，但不能爆。
- **简洁性硬优先**：同样效果，更少代码更好；同样复杂度，更高效果更好。**0.001 的指标改进伴随 20 行 hacky 代码——不值得**；**0.001 改进来自删代码——必须保留**。

---

## 三、实验循环（核心工作流）

每次实验按这 7 步做：

```
1. 假设：我要试什么？为什么觉得它会 work？（一句话写下来）
2. 改代码：只改必要部分，不 side-effect 其他地方
3. 跑：调用实验脚本，等到结束
4. 读结果：提取主指标 + 次指标
5. 判断：keep / discard / crash（见下文规则）
6. 记录：append 一行到 results.tsv
7. commit 或 git reset（keep 则 commit，discard 则 reset）
```

### 判断规则

| 结果 | 主指标 | 代码 | 决定 |
|---|---|---|---|
| ✅ keep | 改进 > noise | 未明显复杂化 | commit，继续 |
| ✅ keep | 不变或略差 | **简化了**（删代码） | commit（简化是胜利） |
| ❌ discard | 改进 < noise | 复杂化 | `git reset` 回 baseline |
| ❌ discard | 变差 | 任意 | `git reset` |
| 💥 crash | N/A | N/A | **先读完整 traceback**（见第四节），修 bug 后重跑 |

### 避免的反 pattern
- **不要**连续跑 10 次实验再 commit。每个独立改动都要有独立 commit。
- **不要**同时改多个东西。一次只测一个 hypothesis。
- **不要**相信"大概 work"。只相信 `results.tsv` 里的数字。

---

## 四、失败模式手册（你 agent 会踩的坑）

以下是**典型 agent 失败模式**（来自 autoresearch 早期真实 bug），必须主动避免：

### 4.1 崩溃时只看最后一行错误
❌ 你会倾向只读 `ValueError: ...` 最后一行。
✅ **强制**读完整 traceback，从最底层 frame 往上看，找 root cause。常见的是 data loader 问题 / dtype 问题 / shape mismatch，而不是表面的 error type。

### 4.2 NaN loss 不退出
❌ 实验跑到一半 loss 变 NaN，但因为没 fast-fail 检查，agent 把"NaN"当成一个数字继续跑，浪费整个时间预算。
✅ 训练循环内加 `if not math.isfinite(loss): raise RuntimeError(f"NaN at step {i}")`。未加的话，把加这条当成第一步改动。

### 4.3 无数据/缺依赖时无限循环
❌ 数据目录为空，dataloader 永远返回空 batch，eval 循环里 step 一直是 0。
✅ 启动时**先校验数据存在且非空**：`assert len(os.listdir(data_dir)) > 0, "No data. Run prepare.py first."`

### 4.4 跳过 section 偷懒
❌ 人类让你做 5 件事，你做了 3 件觉得够了。
✅ 任务分拆成显式编号 checklist，每项必须勾掉或明确说明为什么不做。

### 4.5 格式漂移
❌ 下游 parser 依赖精确格式，你一个小改（加了前缀、变了 case）就全炸。
✅ 改输出格式前 **grep 下游 parser**，确认没人依赖你要改的部分。

### 4.6 reward hacking
❌ 你注意到评估脚本里有个 bug 能被 exploit，你"修"它让分数变好。
✅ 评估脚本是 ground truth，**绝对不改**。发现 bug 告诉人类，不要自己改。

---

## 五、`results.tsv` 格式

每次实验一行。append-only（不要重写历史行）。

```
commit	primary_metric	secondary_metric	status	description
abc1234	1.234567	45.6	keep	baseline
def5678	1.231456	46.2	keep	increase hidden dim 512->640
345abcd	0.000000	0.0	crash	tried lr=0.01, NaN at step 3
678efgh	1.232000	45.1	discard	added extra norm layer, noise-level change + complexity
```

字段说明：
- **commit**: 短 hash（7 chars）
- **primary_metric**: 核心指标，达到 baseline 或更好用实际数字；crash 用 0.000000
- **secondary_metric**: 次要指标（资源消耗 / 速度等）；crash 用 0.0
- **status**: `keep` / `discard` / `crash`（不要发明新状态）
- **description**: 一句话说改了什么。避免逗号（TSV 分隔符是 tab 但有些 parser 不严格）

---

## 六、什么时候停

告诉人类 "完成"，如果以下任一成立：
- 达到人类预设的预算（例如 "跑 10 轮实验"）
- 连续 3 次实验都是 discard（说明容易想到的改动已用完，需要人类介入想方向）
- 发现一个你认为值得单独讨论的 surprise（正面或负面）
- 资源即将耗尽（API quota / 磁盘 / 时间）

**最终交付**：
1. 完整 `results.tsv`
2. 最好的 commit 的 hash + val 数字
3. 一段自然语言总结：我试了什么、什么 work、什么不 work、下一步可以试什么
4. `dev/LOG.md` 里的一条条目（见第七节）

---

## 七、`dev/LOG.md` 条目模板

完成一轮迭代后写一条。四段式：

```markdown
## YYYY-MM-DD: <Title>

### Motivation
为什么想试？从哪里看到这个想法？（1-3 句）

### What changed
具体 diff 粒度：哪个函数、哪个参数、哪个数值。（要能让 3 个月后的我/同事看懂）

### Results
- baseline: 主指标 X ± noise_range
- 改动后: 主指标 Y
- 其他指标对比：...
- 负面结果也写：哪些组合试了不 work

### Conclusion
keep / discard / 部分 keep。默认 on 还是 opt-in？scale 之间是否泛化？
```

**关键**：**负面结果必须公开写**——这是对后来者最有价值的部分。

---

## 八、如果是 AI/ML 训练项目的特别提醒

### 跨 scale 验证
一个改进只在 specific scale 有效 → 疑似 overfit，不要算 keep。至少在 2 个 scale 验证（小 / 目标）。

### 墙钟 > FLOPs
除非在做 scaling laws，否则比较用**固定 wall clock** 预算。架构改动后 tokens/sec 不可比，但墙钟可比。

### 单一主指标 + noise guard
不要同时优化 3 个指标。选一个 primary（训练 loss / eval accuracy / perplexity），选一个 secondary 做 noise guard（例如跨 seed 的 std）。

### 共享文件防冲突
如果结果表在多个 run 并发写，改成 per-run 独立文件，`run_summary.sh` 脚本最后聚合。

---

## 九、和人类的交互协议

### 默认沉默
**不需要**每个实验都 ping 人类。你跑一晚上，早上给一份总结即可。

### 必须 ping 的场景
- 评估脚本里发现疑似 bug（绝不自己改）
- 结果意外地好（可能是 reward hacking）
- 连续 3 次 crash 无法自修
- 资源消耗异常（VRAM 涨 2 倍 / API call 爆 quota）
- 人类明确要求 ping

### 语气
- 不要说 "我认为"。说 "baseline X，改动 Y，数字 Z"
- 负面结果直说："此改动 did not help"，不要粉饰
- 惊喜也直说："此改动加速 30%，建议人类 review"

---

## 十、这份 skill 的作者哲学（为什么这样设计）

1. **人类管规则，agent 管执行**。把 reward / evaluator / baseline 交给人类定，agent 永远不能自己改规则——防止 reward hacking 的物理护栏。
2. **简洁性是一等约束**。agent 默认会堆复杂度，skill 必须显式把"简洁"写进决策函数。
3. **负面结果比正面结果信息量大**。记录负面能让下一个 agent（或下一个你）不重复踩坑。
4. **默认失败**。假设 agent 会漏读、会循环、会偷懒、会无视格式——流程兜底，不寄希望于 agent 足够聪明。
5. **分钟级迭代**。一次改一点、一次 commit 一点。不要累积大 diff。

---

## 附：快速启动 checklist（第一次跑之前勾完）

- [ ] 读完 README
- [ ] 读完 evaluator（知道主指标怎么算）
- [ ] 读完实验代码（至少能指出 3 个可改点）
- [ ] 确认数据 / 依赖 / credentials 都就位
- [ ] 创建隔离分支 `<project>/<run_tag>`
- [ ] 初始化 `results.tsv` 表头
- [ ] 跑 baseline（不改任何代码）
- [ ] baseline 数字 append 到 `results.tsv`
- [ ] commit baseline 对应 hash
- [ ] 写第一条 `dev/LOG.md` 条目（记录 baseline）
- [ ] 开始正式 hypothesis-driven 迭代

---

*这份 skill 基于 [autoresearch](https://github.com/karpathy/autoresearch) 的 `program.md` + [nanochat](https://github.com/karpathy/nanochat) 的 `dev/LOG.md` 提炼，由 `think-like-karpathy` 分析项目整理。适用于任何单一目标 / 可验证 / 可迭代的实验工作流。*
