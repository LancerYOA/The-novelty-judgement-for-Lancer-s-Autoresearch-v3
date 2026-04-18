---
name: novelty-assessment
description: >
  研究创新性（Novelty）深度评估技能：对研究问题和idea进行严苛的10分制创新性评估，
  从文献空白、问题重要性、方法论创新、第一性原理、跨领域适用性、知识体系影响、
  应用价值和学术影响力等多个维度，生成结构化的中文创新性分析报告，并提供改进方向。
  当用户提到'创新性评估'、'novelty assessment'、'创新性分析'、'idea评估'、
  '这个idea创新吗'、'评估创新点'、'novelty check'、'创新性验证'时触发此技能。
  也适用于用户提交文献调研报告和意义分析报告后，要求评估研究创新性的场景。
  即使用户只是问'这个idea怎么样'、'这个方向有创新性吗'、'帮我看看这个研究的创新点够不够'
  这样简短的问题，也应该触发此技能。与文献调研技能不同，本技能的核心不是找论文，
  而是从多维度严格判断一个研究idea的创新性水平，并给出量化评分和改进建议。
---

# 研究创新性（Novelty）深度评估技能

## 核心理念

> **在执行评估前**，请先阅读 `methodology.md`（与本文件同目录）获取完整的方法论参考：
> - 第七节"学术大佬的创新性判断智慧"（Hamming、Michael Black、NeurIPS/ICLR审稿标准）
> - 第八节"顶会评分标准速查"
> - 第九节"2025-2026 最新 AI 同行评审系统原理集萃"（Stanford Agentic Reviewer、DeepReview、AgentReview、debashis1983、NovBench、OpenReviewer、poldrack/ai-peer-review、OpenAIReview——本 skill 的 Phase 1.0 / B0.5 / Phase 2.5 均来自这些系统的设计借鉴）
>
> 如果 `methodology.md` 不可用，本文件的评估流程和评分标准已自包含，可独立执行。

**本 skill 的定位**：Quality Review（帮助作者提升创新性），非 Gatekeeping（非接受/拒绝决策）。Phase 3"创新性增强建议"是核心输出。

本技能的评估哲学源于以下洞察：

**真正伟大的创新往往是做减法或利用直觉**——Dropout（随机关闭神经元）、ReLU（截断负数）、
Attention（只看重要的）、Word2Vec负采样（大分类变小对比）、LoRA（大矩阵拆小矩阵）、
ResNet残差连接（加一条捷径）。这些改变行业轨迹的idea，形式上极度简单，
却抓住了问题的本质。评估创新性时，必须识别这种"简单到令人发指但直击本质"的品质。

**语义熵（Semantic Entropy）的启示**——好的创新往往在概念层面开创新的建模视角，
如从语义角度而非词汇层面考虑模型输出的不确定性，在语言学、信息论和深度学习之间
搭建桥梁。这种跨学科的概念创新，是创新性评估中最应被重视的维度。

**第一性原理（First Principles）**——最顶级的突破来自对问题本质的最简单抽象。
评估时要问：这个idea是否触及了问题的根本？是否从最基础的物理/数学/逻辑出发，
而非在现有框架上缝缝补补？

---

## 评估流程

### Phase 0：信息收集与理解

**⚠️ 去锚定指令（Anti-Anchoring · 最先执行）**：

若用户提交的材料中包含**自我评估分数或等级判断**（如"综合推荐指数 8.1/10"、"创新强度中等偏上"、"预计能中 NeurIPS"等），评估者必须：
1. **首先完全忽略这些数字和等级判断**，不把它们作为评分起点
2. 独立完成 Phase 0.5 / 1.0 / 1 / 2 全流程后，再回头对照用户的自评
3. 材料中的自评仅作为"用户认为的贡献点"的**参考清单**（用于校验 Contribution Claims 是否完整），而**不影响评分数值**
4. 若最终评分与用户自评差距 >1.5 分，在报告"关键发现"段显式说明差距来源（而非向用户自评靠拢）

本条目的：防止 LLM 评估者的锚定效应——一个没看到用户"8.1"自评的独立评估者可能给 4-5 分，但被锚定的评估者可能不自觉向 6-7 分偏移。

1. **接收输入材料**：
   - 用户提交的研究问题/idea描述
   - 文献调研报告（如有）
   - 意义分析报告（如有）
   - 相关论文或代码（如有）

2. **核心信息提取**（借鉴OpenNovelty Phase 1）：
   - 提取**核心任务（Core Task）**：用一句话描述这个研究要解决什么问题
   - 提取**贡献声明（Contribution Claims）**：列出1-3个主要创新点
   - 为每个创新点生成**检索查询变体**：用于后续文献验证

3. **如果用户未提供文献调研报告**，使用web_search进行快速文献扫描：
   - 搜索核心任务相关的顶会/顶刊论文（如 `"{核心任务关键词}" site:arxiv.org`）
   - 搜索每个贡献声明的相关先驱工作（如 `"{方法名}" {领域} paper`）
   - 至少执行5-8次搜索以覆盖主要相关工作
   - **如果web搜索不可用或结果质量差**：基于自身知识库进行分析，在报告中注明"未经文献实时验证"，并建议用户补充文献调研

4. **多idea/多语言处理**：
   - 如果用户同时提交多个idea：逐个评估，每个idea独立出报告
   - 如果用户用英文描述idea：评估过程仍用中文输出，但保留英文术语原文

### ⏸️ 检查点A：确认理解

**在进入评估前，先向用户确认**：
> 我理解的核心任务是：[Core Task一句话]
> 主要创新点：[1-3个Contribution Claims]
> 我会从以下角度重点评估：[最相关的2-3个维度]
> 这样理解对吗？有需要补充的吗？

等用户确认后再继续。如果用户补充了新信息，更新核心信息后再进入评估。

### Phase 0.5：快速筛选（可选）

如果从Phase 0的信息来看，idea满足以下**任意一个**条件，启动快速筛选模式：
- 核心方法是直接调用现有工具/API（如"用GPT-4做X"）且用户未描述任何方法层面的改进
- 文献扫描中找到≥3篇高度相似的已发表工作（方法+场景+目标均接近）
- 仅更换数据集/任务场景，核心方法与已有工作完全一致
- **核心方法可分解为 2-3 个独立组件**，且每个组件在近 2 年文献中都有独立先驱（即使具体组合方式未见过）——本条捕获"组合型创新"，避免 skill 把"三合一拼命题"误判为高分创新

**快速筛选输出**（替代完整报告）：
```
## 快速创新性判断
- 核心问题：[一句话]
- 初步判断：创新性偏低（预估3-4分），原因：[1-2句]
- 关键相似工作：[1-2篇最相关论文/工作名称]
- 主要瓶颈：[最弱维度]
- 提升方向：[2-3条建议]
想要完整的六维度分析报告吗？或者你的方法是否有我遗漏的技术创新点？
```

**注意**：如果用户描述极其简短（<30字且无技术细节），在快速筛选前先追问：
> 能否再补充一些技术细节？比如你打算用什么方法、与现有工作的核心区别是什么？

### ⏸️ 检查点A2：快速筛选确认（仅在触发Phase 0.5时使用）

输出快速判断后，**必须等待用户响应**，不要自动进入Phase 1。明确询问：
> 这个快速判断是否准确？以下三选一：
> (1) 准确，停止评估
> (2) 我有遗漏的技术细节要补充：[请说明]
> (3) 仍想要完整六维度报告

只有用户明确选择(3)或在补充细节后要求继续，才进入Phase 1。这避免了"快速判断不准但仍硬上完整报告"的资源浪费。

### Phase 0.8：Research Taste Gate（值不值得做的前置硬门槛）

**核心理念**：在谈 novelty 之前，先判断这个题**是否值得投入资源**。科研大佬看题的第一反应不是"名字新不新"或"能不能投顶会"，而是问题是否重要、为什么是现在、是否有 reasonable attack、是否有 decisive test、失败后还剩什么。

本 skill 的最终输出因此**必须同时给出两类结论**（不能合并为单一分数）：
- **Novelty Score**（新颖性总分，由 Phase 2 六维度加权产生）
- **Worth-Doing Verdict**（值不值得投入，综合 Taste Gate 与 Novelty Score 得出）

常见错配一旦忽略 Taste Gate 就会漏掉：
- 高 novelty + 低 attackability → 值得追问但不值得立即投入
- 低 novelty + 高 attackability → 适合练手但发表潜力有限
- 高 novelty + 无 decisive test → 极可能是 "plausible but empty"

#### 必答八问（Taste Gate Checklist）

1. **What**：你到底想做什么？不用术语，普通同行能听懂吗？
2. **Today**：今天别人怎么做？现有做法卡在哪？
3. **New lever**：你抓到的新杠杆是什么？
4. **Why now**：新的数据/算力/工具/理论视角/制度机会，哪一个让它"现在可做"？
5. **Who cares**：谁会在乎？研究社区、工程团队、产业用户，还是其他学科？
6. **Reasonable attack**：有没有一条可执行的攻击路线？
7. **Decisive test**：什么最小实验/toy case/theorem/ablation 一做就知道值不值？
8. **Residual value**：如果主假设失败，还剩什么可发表的知识价值？

#### Taste Gate 评分（1-10，独立于 Novelty Score）

- **9-10**：问题重要、timely、攻击路线清晰，失败也有残值；值得重投入
- **7-8**：很值得做，至少值得一个完整项目/论文周期
- **5-6**：可做小规模 probe，但还不够 thesis 级或主线级
- **3-4**：概念上有趣，但 why-now 或 attackability 太弱
- **1-2**：不建议投入；即使新，也大概率不值得

#### 直接触发的硬限制（作用于后续 Phase 1/2 评分）

- **没有 reasonable attack** → Worth-Doing Verdict 最高只能是"谨慎试探"
- **说不清 who cares** → Phase 1 维度五（应用价值）不得高于 5 分
- **说不出 decisive test** → 后续新增的"可进攻性与证据可信度"输出字段上限 ≤ 5
- **主假设失败后毫无残值** → 不适合高投入长期项目，Worth-Doing Verdict 不得 ≥ "值得做完整项目"

#### 信息不完整时的处理原则

若用户材料不足以回答八问中的 ≥3 个：**不阻塞、不强行追问**，而是做保守外推并显式标注：
- 哪些回答来自假设（非用户原文）
- 哪些结论置信度低
- 哪个关键信息一旦补充，最可能改变判断

### ⏸️ 检查点A2.5：Taste Gate 确认

八问回答完毕后，向用户展示：
> **Taste Gate 打分：X/10**，主要依据：[1-2 句]
> - Who cares（谁会在乎）：[一句]
> - 最便宜 Decisive Test：[一句或"未找到明确的"]
> - 失败后残值：[一句或"几乎无残值"]
> - 触发的硬限制（若有）：[列出对后续评分的封顶影响]
>
> 以下三选一：
> (1) 认可本 Taste Gate 判断，进入 Phase 1 文献核查与六维度评分
> (2) 我有八问中某问的补充信息：[请说明]
> (3) Taste Gate 太低，不希望继续——停止评估

只有用户选择 (1) 或补充信息后要求继续，才进入 Phase 1.0 / Phase 1。

### Phase 0.9：Feasibility Assessment（可行性独立评估 · 借鉴 AI-Scientist-v2 Experiments 字段）

**设计来源**：AI-Scientist-v2 `perform_ideation_temp_free.py:36` 的 `Experiments` 字段要求"simple and feasible"；Stanford Agentic Reviewer 在 7 维 rubric 中隐式用 "Claims well supported" 评估可行性。本 skill 在 V4 之前把 feasibility 混入 Taste Gate "Reasonable attack" 和 Risk Factors"Feasibility 风险"——两处都不是专门评分，评估者倾向**用 novelty 顾虑下调 feasibility**（NovBench 发现的 Bold-Idea 保守偏见的变体）。R25 把 Feasibility 剥离为**独立一维评分**，与 Novelty 正交。

#### 与其他字段的层级切分（避免重叠）

| 层级 | 字段 | 作用 | 粒度 |
|------|------|------|------|
| 战略层 | Taste Gate "Reasonable attack" | 有无可执行的攻击路线（二元判断） | 定性 |
| **战术层** | **Feasibility Score（新）** | **合理成本内能否做出来（量化打分 1-10）** | **定量** |
| 战术细节 | Risk Factors 中的 Feasibility 风险 | 具体可能崩溃的环节 | 条目 |

三者正交：战略判断"值得不值得打"，战术评分"打得下来打不下来"，细节记录"哪一步最可能翻车"。

#### Feasibility Score 的 4 子项（每项 1-10，取均值为最终 Feasibility Score）

| 子项 | 10 分（理想） | 1 分（极难） |
|-----|------------|------------|
| **F1 算力需求** | 单卡 A100 一天内完成 | 千卡月级（超出一般学术组能力） |
| **F2 数据可得性** | 公开 benchmark 即可（如 GSM8K、ImageNet） | 需自建大规模新数据集（标注成本 > $100K 或法律受限） |
| **F3 时间成本** | 一名 PhD 一周出 decisive test 结果 | 一名 PhD 一年仍无法完成最小可验证版本 |
| **F4 团队能力门槛** | 典型 ML PhD 可单独执行 | 需顶尖跨领域专家团队（如数学家+物理学家+系统工程师协作） |

**Feasibility Score = (F1 + F2 + F3 + F4) / 4**，四舍五入到一位小数。

#### 评分标准（1-10 档位）

- **9-10**：极易实现，小组 1-2 人 1-2 周内可完成 decisive test；典型例子：fine-tune 小模型测新 loss
- **7-8**：合理可行，单个 PhD 1-3 月完成；典型例子：在公开 benchmark 上训练新架构
- **5-6**：中等难度，需整组 3-6 个月；典型例子：构建新数据集 + 训练 + 评估
- **3-4**：高难度，需大组 6-12 个月或需要特殊资源（大算力、罕见数据）
- **1-2**：极难实现，需要当前条件下不具备的资源（如千卡 × 月级，或需要收集需 IRB 审批的敏感数据）

#### 特殊说明

- Feasibility ≠ Novelty。Feasibility 9 且 Novelty 2 是合法状态（如 "fine-tune GPT-4 做情感分析"——极易做但毫无新意）；Feasibility 3 且 Novelty 9 也是合法状态（如 "要求跨物种脑电数据验证新认知理论"——极难做但若成功极创新）
- Feasibility 评分**不允许受 Novelty 拖累**（反 Bold-Idea 保守偏见）。若 idea 技术上可做，即使被判 `directly_refuted`（即没 novelty），Feasibility 仍应按实际成本打分
- 若用户描述不足以判断某子项（如未说明数据源），该子项按"信息不足中位 5 分"处理，并在报告中标注"F{X} 因信息不足估计"

#### 输出到 Dashboard

Feasibility Score 直接进入 Decision Dashboard 的新字段 `Feasibility Score`，并与 Novelty Score 组合成 **Risk-Reward Quadrant**（见 Phase 2 决策表）。

### ⏸️ 检查点 A2.8：Feasibility 确认（可跳过）

Feasibility 评分后向用户展示：
> **Feasibility Score：X / 10**（F1=算力 X，F2=数据 X，F3=时间 X，F4=团队 X）
> 最大 feasibility 风险：[一句]
>
> 三选一：
> (1) 认可评分，进入 Phase 1.0 / 1
> (2) 我有信息可修正某子项：[说明]
> (3) 跳过本评估（默认 Feasibility = 5，在报告中标注"未评估"）

用户选 (3) 时，Risk-Reward Quadrant 不生成，Dashboard 标注 "Feasibility 未评估"。

### Phase 1.0：结构化Novelty Verification Pipeline（DeepReview风格·深度模式）

本阶段是Phase 0第3步"快速文献扫描"的深化版，参考 **DeepReview (arxiv 2503.08569) z₁ Novelty Verification** 设计。
适用场景：idea涉及多个技术点、用户要求高置信度评估、或Phase 0.5未触发快速筛选时。
如果 Phase 0 已做充分检索或用户只要快速判断，可**跳过本阶段**直接进入 Phase 1，并在报告中降低证据权重。

#### Step 1：查询扩展（Multi-Specificity Query Generation）

对每个 Contribution Claim，生成 **3-5 个不同特异性层次** 的检索查询（借鉴 Stanford Agentic Reviewer 的 multi-specificity 策略）：

| 层次 | 示例（claim: "用 RL 优化 CoT 展开粒度"） |
|------|----------------------------------|
| 高特异性（精确） | `"reinforcement learning" "chain-of-thought" "granularity"` |
| 方法通用 | `"RL-based reasoning control" LLM` |
| 问题层面 | `"adaptive chain of thought length"` |
| 先驱定位 | `"DeepSeek R1" OR "OpenAI o1" CoT RL` |
| 跨领域迁移 | `"adaptive computation" "information sufficiency"` |

每个 claim 生成 3-5 个 query，覆盖**核心技术词 + 变体/同义词 + 先驱标志**。

#### Step 2：Top-K 文献检索

对每个 query 执行 web_search（或 Semantic Scholar / arXiv / OpenScholar API 如可用）：
- 每个 query 取前 **10-15 条结果**
- 合并去重后，全部 claim 总计目标检索 **20-60 篇**候选论文
- 记录：`title + year + venue + abstract + matched_query`

**降级策略**：若 web_search 不可用或结果 <5 篇，明确标注"证据不足模式"并继续，同时将最终 claim audit 判定**置信度降一档**，默认偏向 `evidence_insufficient` 状态（避免在无证据情况下误给 `directly_refuted` 或 `not_refuted_yet`）。

#### Step 3：相关性 Rerank → Top-10

按以下加权打分，选出 **Top-10** 最相关论文进入深度对比：

| 维度 | 权重 | 评分依据 |
|------|------|---------|
| 方法重叠度 | 30% | 核心方法是否与 claim 一致或接近 |
| 问题定义重叠 | 30% | 解决的问题是否相同/重叠 |
| 发表年份 | 20% | 近 3 年优先（2023-2026） |
| 影响力 | 20% | 顶会（NeurIPS/ICLR/ICML/ACL/CVPR）或高引优先 |

#### Step 4：Refutation Chain 深度对比

对 Top-10 中与每个 Contribution Claim **最相关的 3-5 篇**，逐一做 Refutation 判定，格式统一如下：

```
Claim: [原claim一句话]
候选先驱: [Paper A (First-Author, Year, Venue)]
方法对比: 我方法=X / 候选方法=Y / 本质差异=Z
判定: directly_refuted / partially_refuted / not_refuted_yet / evidence_insufficient
证据: [具体差异点或共同点, ≤2 句]
差异层级: 概念骨架 / 机制 / 仅实验组合（越靠前 delta 越硬）
```

聚合成 **Refutation 汇总表**，作为 Phase 1 维度一（问题重要性与文献空白）的**直接评分依据**。

#### Step 5：Pipeline 自检（通过才进 Phase 1）

- ✅ 每个 Contribution Claim 至少有 **3 篇** 候选对比
- ✅ 至少 **2/3** 的候选有明确判定（非全 unclear）
- ✅ 至少 **1 篇近 3 年**（2023-2026）工作纳入对比

任一失败则标注"证据不足"并在 Phase 1 维度一扣分 1 档（而非硬打高分）。

### ⏸️ 检查点 A3：Pipeline 结果确认（仅深度模式）

完成 Pipeline 后，向用户展示：
> 文献 Pipeline 完成：共检索 N 篇，Top-10 关键候选先驱：[Paper A/B/C 一行摘要]。
> Claim Audit 初步判定：M 篇 directly_refuted / K 篇 partially_refuted / L 篇 not_refuted_yet / P 篇 evidence_insufficient。
> 是否继续 Phase 1 评分？或有需要补充/修正的候选？

等用户确认后再进入 Phase 1。

### ⏸️ 检查点 A4：术语剥离测试（Terminology Stripping Test · Phase 1 前强制）

Phase 1 评分**之前**必须执行的强制步骤。来源：反制"Smart Plagiarism"和"术语包装"两大反模式（SKILL.md 反模式节第 586/590 行）。

**执行步骤**：

1. **列出所有新造术语**：把 idea 中所有作者自造术语列成清单（如 "Post-Answer Verification Collapse"、"Commit-Gated Challenger"、"Semantic Gate" 等）
2. **朴素白话替换**：每个新术语改写成 10 岁学生都能看懂的描述（如 "长上下文下验证率下降"、"选择性调用第二个 LLM 重新检查"、"语义相似度阈值判断"）
3. **重做 refutation 检验**：在朴素白话版本上重新检索文献，看是否能找到概念层等价的已有工作

**判定规则**：
- 若剥离后 claim audit 结果从 `not_refuted_yet` → `partially_refuted` / `directly_refuted`，或 gap 明显缩小，则该 claim **存在术语包装风险**
- 命中术语包装风险 → **维度四（知识体系影响）上限 ≤ 4 分**（术语不构成真实的概念贡献）
- 在综合评定"关键发现"段落必须标注 `[术语包装存疑]` 标签

**免除条件**：若术语对应的是全新的**可测量量**（如"语义熵"引入了 entropy over meanings 的可计算度量），而非仅为旧概念换皮，则不触发本处罚。

### Phase 1：六维度创新性评估（10分制）

对每个维度进行独立评分（1-10分），并给出详细理由。

#### 维度一：问题重要性与文献空白（权重 20%）

**评估要点**：
- 这是否是领域内的关键难题？长期存在但未被解决的痛点？
- 文献中是否存在明确的研究空白（Gap）？
- 该问题的影响范围和深度如何？
- 是否有其他研究者已经从类似角度攻克了该问题？

**⚠️ 内部构成约束（防止"重要问题 + 标准方法"获得虚高分）**：
本维度的 10 分内部构成须按如下分配，避免把"问题很重要"单点拉高整体分数：
- **问题重要性分量**：最多占本维度 **30%**（≤3 分）——即使问题是领域核心痛点（产业热点、长期难题），单凭"问题重要"也不应让本维度超过 3 分
- **文献空白真实程度**：占 **40%**（≤4 分）——要求"解法切入角度"无先驱，而非仅仅"该问题没被这样包装"
- **解法新颖性与切入角度**：占 **30%**（≤3 分）——核心方法的新颖程度，包括与 refutation 判定的一致性

**硬性上限**：单纯"问题很重要但解法是标准方法（组合型、已有技术拼装）"的情况，本维度 **不应超过 5 分**。

**评分标准**：
- **9-10分**：解决了领域内公认的核心难题，文献中几乎无人触及此角度，填补了重大理论空白
- **7-8分**：针对重要但非核心的难题，或从全新视角审视已有问题，文献中仅有零散尝试
- **5-6分**：问题有一定价值，但已有较多相关研究，创新空间受限
- **3-4分**：问题价值有限，或已被充分研究，仅做增量改进
- **1-2分**：问题缺乏研究意义，或已被完全解决

**关键检验 — 四状态 Claim Audit**（借鉴 research-grade 修订，替代旧版二元 `can_refute / cannot_refute`）：

旧版的二元判定存在两个严重问题：
- 把"当前没找到直接反例"误写成"已经证明绝对新颖"
- 让"实验没做过"被误当成"概念真的新"

**新的四状态规则**（对每个 Contribution Claim 逐条判定）：

- **`directly_refuted`** — 已有工作在问题定义 + 概念骨架 + 关键机制上都高度重合。这类 claim 不应再作为主创新点。
- **`partially_refuted`** — 祖先思想已经存在，但在以下至少一项上仍可能有实质增量：新问题表述 / 新可测量量 / 新理论桥梁 / 新机制交互 / 新约束下成立。这类 claim 只能拿中档分，除非增量讲得极硬。
- **`not_refuted_yet`** — 目前没有找到足以直接击穿 claim 的强先驱。**注意**：这**不等于**"已经证明绝对新颖"，只是"当前证据下尚未被击穿"——仍要看 delta 是否硬、是否经得起最强先驱对比。
- **`evidence_insufficient`** — 检索不充分、claim 表述过模糊或候选文献质量差。这种情况不能吹高分，只能降低置信度（置信度至少降一档，总分向中位收敛）。

**⚠️ `directly_refuted` 判定的严格化要求**（R20 修订，解决判定过宽导致的分数不稳定）：

此前的 `directly_refuted` 判定过于宽松，导致**方向性重叠就被判定为直接反驳**（如"RL + LLM 推理"这种大方向，任何新工作都会被 R1/o1 判定 `directly_refuted`，分数塌缩）。此修订要求证据更严格。

**判定 `directly_refuted` 必须同时满足以下三条**（缺一不可，少一条只能判 `partially_refuted`）：

1. **问题层重合**：先驱解决的问题与本 claim 声称解决的问题**在定义上相同或包含关系**（不是"同一方向"，是"同一具体问题"）。
   - ✅ 重合："DeepSeek-R1 用 RL 训练 LLM 在数学推理 benchmark 上自动生成长 CoT 并提升准确率" vs 本 claim "用 RL 训练 LLM 在数学推理 benchmark 上自动生成长 CoT 并提升准确率" — 问题完全相同
   - ❌ 不算重合："DeepSeek-R1 用 RL 训练推理" vs 本 claim "用 RL 学习推理跳过/展开的粒度决策" — 同一方向，但前者目标是"训练出会推理的模型"，后者目标是"训练出会决定何时推理的模型"，具体问题不同（前者 ≠ 后者的超集）

2. **概念骨架重合**：先驱的概念框架与本 claim 的概念框架**在关键概念集合上有 ≥80% 重叠**（不是"都提到同个领域"，是"核心概念几乎完全相同"）。
   - ✅ 重合：先驱="GRPO + rule-based reward + length penalty" vs 本 claim="GRPO + 正确性 reward + 长度 penalty" — 三个核心概念完全对应
   - ❌ 不算重合：先驱="GRPO + rule-based reward" vs 本 claim="GRPO + 复合 reward（正确性 + 粒度决策 reward）" — 概念集合中的"粒度决策 reward"在先驱中**不存在或非焦点**

3. **关键机制重合**：先驱的**实现机制**（训练算法 / 架构模块 / loss 函数 / 推理过程）与本 claim 的实现机制**在细节上基本一致**（允许 ≤20% 的机制层差异）。
   - ✅ 重合：先驱的训练过程 = "采样 → rule-based reward 评分 → GRPO 更新"；本 claim = "采样 → rule-based reward 评分 → GRPO 更新"
   - ❌ 不算重合：先驱的训练过程 = "采样 → 规则 reward → GRPO"；本 claim 机制 = "采样 → 规则 reward + 学习型粒度 gate → 双优化器更新" — 机制层差异 >20%

**如果三条中有任何一条不能在**"具体引用的先驱论文方法描述"**和**"本 claim 的具体方法描述"**之间做出显式对比**，则**必须降级为 `partially_refuted`**，在证据字段中**显式标注"哪一层未满足三条判定"**。

**模糊输入的保守处理**：当用户提交的 idea 描述本身过于简短（<50 字）或方法细节缺失时，评估者无法在"机制层"做细粒度对比——此时**不得判 `directly_refuted`**，应判 `partially_refuted` 并显式标注"[判定存疑：因用户描述不足以做机制层对比，保守判定]"。

**反稳定性陷阱**：
- 不允许凭"我记得类似工作存在"的模糊印象判定 `directly_refuted`——必须能写出具体先驱论文的 first author + year + venue
- 不允许因"同一研究方向"就触发 `directly_refuted`——方向重叠不是 claim 重叠
- 如果评估者在多轮独立运行时对同一 claim 的判定在 `directly_refuted` / `partially_refuted` 之间摇摆，**默认取 `partially_refuted`**（保守判定，避免稳定性问题）

本 R20 修订后，evaluator 在两个独立 run 之间对同一 claim 的判定一致性应从之前的约 60% 提升到 ≥85%，分数波动范围应从 ±1.0 缩小到 ±0.5。

**状态 → 分数映射（作用于本维度）**：

| 主 claim 状态 | 本维度（文献空白真实程度分量）分数上限 |
|--------------|------------------------------------|
| `directly_refuted` | ≤ 3（Novelty Score 通常不应超过 4.5，除非另有独立主 claim） |
| `partially_refuted` | ≤ 6（除非增量是"新可测量量 / 新理论桥梁"这类硬 delta） |
| `not_refuted_yet` | 可达 8+，但需写清"最强先驱是谁、本工作到底比它新在哪个层级（概念/机制/实验组合）" |
| `evidence_insufficient` | 置信度标记 Low，不得高于 6 |

- 示例：
  - `directly_refuted`："创新点'用RL优化推理链'→ DeepSeek-R1 (2025) 已实现 RL 驱动的推理优化，问题/概念/机制三层都高度重叠"
  - `partially_refuted`："创新点'Post-Answer Verification Collapse + Commit-Gated Challenger 组合' → 剥离术语后 = 'CCR + Reinforcement Inference' 已覆盖概念层；仅组合方式与 ablation 设计未见过（实验层新，概念层不新）"
  - `not_refuted_yet`："创新点'图上变分推断视角'→ GIB (2020) 用信息瓶颈做图表示学习，但未建立与变分推断的等价关系——在当前检索证据下，概念桥梁未被击穿"
  - `evidence_insufficient`："创新点'多模态融合框架'→ 找到相似架构但应用领域不同，且未能做到同领域充分检索——置信度 Low"

**Disruption vs Novelty 双指标视角**（2024年计量学新发现）：
研究表明disruption（颠覆性）和novelty（新颖性）是**本质不同**的两个维度：
- **Disruption**：是否颠覆了现有范式？引用此工作的后续论文是否**不再引用**该工作引用的前驱？（即切断了学科的"祖先链"）
- **Novelty**：是否建立了**新的跨领域连接**？是否重组了之前未被连接的知识单元？
- **同时具备才是顶级创新**（Attention同时disruptive + novel；很多增量工作novel但不disruptive）
- **宏观背景**：Nature 2023研究发现全领域论文正变得**越来越不disruptive**——评估时要区分"增量式新"和"颠覆式新"，给后者更高权重

#### 维度二：方法论创新与第一性原理（权重 25%）

**评估要点**：
- 方法是否触及问题的本质？是否从第一性原理出发？
- 是否引入了全新的数学/物理/逻辑框架？
- 还是在现有方法上做增量修改（换Loss、加模块、调结构）？
- 方法的简洁性与优雅性如何？（简单而深刻 > 复杂而表面）

**评分标准**：
- **9-10分**：提出了全新的范式或理论框架，如Attention机制之于序列建模、ResNet之于深层网络。方法简洁但深刻改变了领域思维方式
- **7-8分**：在现有框架内提出了显著不同的新方法，有明确的理论依据，如LoRA之于大模型微调
- **5-6分**：对现有方法进行了有意义的改进或组合，有一定新意但未突破框架
- **3-4分**：仅是现有方法的简单组合或微调，缺乏理论深度
- **1-2分**：没有方法论创新，纯粹的工程实现或参数调优

**第一性原理检验**：
- 信息的流动是否通畅？（参考ResNet的梯度高速公路）
- 是否真的需要关注所有输入？（参考Attention的选择性聚焦）
- 能否把复杂问题拆解成几步走？（参考CoT的思维链）
- 是否利用了数据内部的监督信号？（参考BERT的自监督）
- 是否把大问题降维成小问题？（参考Word2Vec的负采样）
- **Fertile Ground 视角**：这个 idea 能 chain 出多少后续工作？如果解决了，会打开什么新的问题空间？一次性的改进 vs 能长出 10 篇后续论文的 idea，差别巨大（该视角为社区常见经验判断，非某位学者的已发表"标准"）

#### 维度三：跨领域适用性与推广潜力（权重 15%）

**评估要点**：
- 创新点是否仅限于特定任务/数据集，还是具有普适性？
- 方法能否迁移到其他领域？
- 核心思想是否具有"元方法"（meta-method）的特质？

**评分标准**：
- **9-10分**：核心思想具有跨领域的普适价值，如Dropout的随机正则化思想后来影响了数据增强、集成学习等多个方向
- **7-8分**：方法可以自然扩展到同类问题的多个子领域
- **5-6分**：主要适用于目标领域，但有一定的迁移可能性
- **3-4分**：高度依赖特定场景或数据特征
- **1-2分**：完全针对特定任务定制，无推广可能

#### 维度四：对学科知识体系的影响（权重 15%）

**评估要点**：
- 是否建立了新的概念、理论框架或模型？
- 是否有可能推动新的学科分支的形成？
- 是否改变了领域内的研究范式或思维方式？

**评分标准**：
- **9-10分**：开创性地定义了新的研究范式或概念体系，如"语义熵"概念在语言学、信息论和深度学习之间搭建桥梁
- **7-8分**：显著丰富了现有理论体系，提出了新的分析视角或解释框架
- **5-6分**：对现有知识体系有补充价值，但未引入根本性的新概念
- **3-4分**：在已有框架内运作，未对知识体系产生明显影响
- **1-2分**：对学科知识体系无任何贡献

#### 维度五：应用价值与市场需求（权重 10%）

**评估要点**：
- 创新点能否解决实际工程/产业/社会问题？
- 是否有明确的应用场景和用户需求？
- 实现成本和部署难度如何？

**评分标准**：
- **9-10分**：直接解决了重大实际需求，如LoRA让普通公司也能微调大模型
- **7-8分**：有明确且重要的应用前景
- **5-6分**：有一定应用价值但场景受限
- **3-4分**：偏理论研究，应用前景不明
- **1-2分**：无明显应用价值

#### 维度六：学术影响力潜力（权重 15%）

**评估要点**：
- 预估论文的引用潜力和后续研究关注度
- 是否可能引发跟进研究热潮？
- 成果是否适合在顶会/顶刊发表？
- 叙事是否足够清晰有力，能被广泛传播？

**评分标准**：
- **9-10分**：极有可能成为领域里程碑论文，引发大量跟进研究
- **7-8分**：有望在顶会/顶刊发表并获得显著关注
- **5-6分**：可在优秀会议/期刊发表，但影响力有限
- **3-4分**：仅能在一般期刊发表
- **1-2分**：缺乏学术影响力

### Phase 1.5：Reflection Pass（反思精化 · 借鉴 AI-Scientist-v2 `num_reflections` 机制）

**设计来源**：Sakana AI `AI-Scientist-v2/ai_scientist/perform_ideation_temp_free.py:128-266` 的 Reflection Loop——每轮 idea 生成后强制进入"思考→可选文献查询→修正"循环，平均让 novelty 判断的准确率提升显著。本 skill 把它改造为"评分落地前的主动质疑"环节，强制评估者回头审视最弱证据点。

#### 触发模式（两档）

- **轻量 Reflection（默认）**：所有评估必须执行，输出 ≤200 字，无需二次检索
- **深度 Reflection（可选）**：满足以下任一条件时必须升级
  - 六维度极差 ≥5（例：方法论 4 分 vs 应用 9 分）
  - Claim Audit 中 ≥1 条判定为 `evidence_insufficient`
  - 用户明确要求高置信度或"做 reflection pass"
  - Phase 0.5 快速筛选未触发但 Claim 数量 ≥3（潜在 Missing Multi-Innovations 风险高）

#### 执行步骤（三步 Pass）

**Step 1 — 查漏 Pass（自我审视最弱评分证据）**：

逐一审视 Phase 1 六维度评分，对每个维度问一句话：
> 这个分数的**最弱证据**是什么？如果一个强敌 reviewer 只攻击这一条，我能防守住吗？

记录每个维度的"最弱证据"（一句话），**不**要求此步调分。

**Step 2 — 次要 Claim 扫描（反 Missing Multi-Innovations 偏见 · NovBench）**：

主动提出一个问题：
> 除了主 Contribution Claims 之外，用户的描述里是否隐含了 **2nd/3rd tier 创新点**（小改动、次要组件、辅助数据、分析视角）？

如找到候选次要 claim，补录到 Claim Audit 四状态表，至少判定其中一条。

**Step 3 — 定向补检索（仅深度模式 + web_search 可用时）**：

针对 Step 1 最弱证据点、Step 2 新发现的次要 claim，最多执行 **2-3 个新 query**（每 claim ≤1 个）。若 web 不可用，跳过本步并在"反思历史"中标注"未做二次检索"。

#### 修正规则（硬约束）

- 允许调整范围：**每维度 ≤ ±0.3 分**；允许在 Risk Factors 清单中新增 1-2 条；允许升/降一个 Claim Audit 状态（但必须写明新证据）
- **禁止**：凭"再想一想"调分——必须有具体"新发现的证据点"或"被主评分漏掉的次要 claim"才能调整
- **禁止**：Reflection 后重跑 B0.8/B0.9（避免循环），只更新维度分和 Risk Factors，让 B0 自检与 B0.9 自然适配新数据
- **超出 ±0.3 的调整**：说明最初评分存在严重错误，必须**完全重新评估**该维度（不算 Reflection，算回退重评）

#### 输出格式（必须追加到报告"反思修正历史"段）

**轻量模式输出**（≤200 字）：
```
### Reflection Pass（轻量 · Phase 1.5）
- 最弱证据点（6 维度各一句）：D1: ... / D2: ... / D3: ... / D4: ... / D5: ... / D6: ...
- 次要 Claim 扫描：[发现 1 条 / 未发现次要 claim]
- 修正记录：[无调整 / 调整 X 维度 ±Y 分，因 Z 证据]
```

**深度模式输出**（≤500 字）：
```
### Reflection Pass（深度 · Phase 1.5）
- 触发原因：[极差 ≥5 / evidence_insufficient / 用户要求 / Multi-claim]
- 最弱证据点分析（6 维度详述，每维 30-50 字）：...
- 次要 Claim 发现：[列出新发现的 2nd/3rd tier claim + 它们的 Claim Audit 状态]
- 二次检索结果：[检索到 N 篇新文献 / web 不可用跳过]
- 修正清单：
  - D{X}: {原分} → {新分}（证据：{一句话}）
  - Risk Factors 新增：{一句话}
- 置信度变化：{Medium → High / 维持 / 降至 Low}
```

#### 与 Phase 2.5 的关系

Phase 1.5 Reflection 是**评估者自省**（单视角深化），Phase 2.5 多视角自检是**Optimist/Pessimist 对抗**（多视角交锋）。两者正交互补：
- Phase 1.5 补**证据深度**（回头找漏的证据）
- Phase 2.5 补**视角广度**（换立场重看）

若 Phase 1.5 已触发深度模式，Phase 2.5 可降级为轻量模式（只写一行 Meta 结论），避免冗长。反之亦然。

### ⏸️ 检查点 B0.2：Reflection 结果确认（仅深度模式）

完成深度 Reflection 后，向用户展示：
> **反思发现**：[Step 1 最关键的 1 条最弱证据] + [Step 2 最关键的 1 条次要 claim（若有）]
> **修正**：{没有修正 / D{X} 从 A 分调到 B 分，因 Z}
> **新的 Confidence**：{High / Medium / Low}
>
> 三选一：
> (1) 接受 Reflection 修正，继续 B0 自检
> (2) 我有额外信息可进一步修正：...
> (3) Reflection 过度了，撤销修正

用户选 (3) 时，恢复 Reflection 前的分数并进入 B0，同时在报告中标注"用户撤销 Reflection 修正"。

### ⏸️ 检查点B0：评分自检（Phase 1结束→Phase 2开始之前）

完成六维度评分后，**先做内部自检**再向用户汇总，按以下三条逐一核查：

1. **极端分检验**（**R22 修订**：对高分和低分采用不对称要求，鼓励识别真正优秀的创新）：
   - **低分极端（任意维度 <3 分）**：需提供至少 2 条独立证据支撑（论文引用、第一性原理推导、对标案例任选其二）。无法补足 → 向中位收敛 1 分。
   - **高分极端（任意维度 >8 分）**：需提供至少 1 条独立证据支撑（对标附录经典锚点表中的具体案例，说明为何评分与锚点一致）。无法补足 → 向 8 分收敛 0.5 分（而非回退 1 分，因为原规则会直接抹除高分识别能力）。
   - **极高分 (>9 分)**：保留原严格要求——需至少 2 条证据，且评估者必须显式对标 Attention/ResNet/Dropout 等顶级锚点，说明待评估 idea 在哪个维度接近这种级别。无法补足 → 向 9 分收敛 1 分。
   - **动机**：旧版"低分高分一视同仁"导致高分维度难以立足，评估者因"补证据麻烦"而向中位收敛，扼杀了识别优秀创新的能力。
2. **方差检验**：六维度分数的极差（max - min）是否>5分？若是，须显式说明为何同一idea在不同维度评价差异悬殊（例：方法论极弱但应用价值极高的idea，是工程驱动而非研究驱动）。
3. **锚点对标**：将待评估idea与附录"经典创新案例参考锚点"表对照——总分预估应落在哪个区间（例：6.0-7.0区间应类比"典型顶会论文"）？若计算总分与锚点直觉差距>1.5分，回到最弱维度重新审视评分理由。
4. **多锚点对标**（**R22 修订**：从单一低分 one-shot 扩展到双锚点对标，防止评分被单个 4.65 分锚点拉向中低档）：

   **锚点 A（低-中档）**：§输出示例节 one-shot 案例（RL 优化 CoT 展开粒度，总分 4.65）——代表"组合型增量，在已有范式内的修改"
   **锚点 B（中-高档）**：参考 `methodology.md` §十二 或本文件附录经典案例表——代表"典型顶会论文（5.5-7.0）"档，如 LoRA (8.5)、Semantic Entropy (8.5)、Mamba (8.0) 的**中低等价物**（非这些顶级案例本身，而是"有跨学科桥梁或显著理论贡献但未达 Attention 级别"的工作）

   **对标执行**：评估者须**先**判断待评估 idea 更接近哪个锚点，**再**应用对应的偏差约束。

   **与锚点 A 相似的判定条件**（任一满足）：
   - 都是"组合型贡献"且**无理论桥梁支撑 1+1>2**
   - 都是"纯增量改进"（无新概念、无跨学科桥梁、无新可测量量）
   - Claim Audit 中 `directly_refuted` 占 ≥50% 的 claim
   - 都依赖 toy/有限实验验证且无工业合作
   - **分数偏差约束（A 档）**：总分应落在 **3.2-6.2** 区间

   **与锚点 B 相似的判定条件**（任一满足）：
   - 具备 **≥1** 个以下 B0.9 正向特质：跨学科桥梁、新可测量量、Fertile Ground ≥3 后续方向、朴素但美
   - Claim Audit 中 ≥1 个主 claim 为 `not_refuted_yet` 且 delta 在概念层或机制层
   - 触发 B0.9 加分 ≥+0.5
   - 有明确的理论贡献或新视角（即使不是范式级）
   - **分数偏差约束（B 档）**：总分应落在 **5.0-7.5** 区间

   **双档都不满足时**（idea 既非纯增量也无正向特质）：不强制锚点约束，按六维度实际计算，但要在报告中写一句话说明"未匹配任一锚点"。

   **偏差处理规则**：
   - 若最终分落在目标档区间内 → 直接通过
   - 若偏差 < 0.5 分且 B0.9 加分支撑充足 → 通过，无需调整
   - 若偏差 ≥ 0.5 分但有明确差异支撑（至少 2 条具体维度上的根本差异）→ 保留原分
   - 若偏差 ≥ 0.5 分且无差异支撑 → 向目标档区间最近边界收敛

   **R22 设计动机**：旧版只用单一 4.65 分锚点 + ±1.5 约束，效果是把所有 "像 one-shot 的 idea" 强制压在 3.2-6.2，让真正具备跨学科桥梁或理论贡献的工作无法突破 6.2。新版双档机制让"中高档 idea"有独立对标参照，不被误拖到低档。

通过自检后再进入Phase 2加权计算，避免"评分膨胀/萎缩"和"维度间矛盾"两类常见错误。

### ⏸️ 检查点B0.5：LLM评估偏见反制（NovBench 2026启发）

B0 通过后，额外做 **3 条反偏见检查**。来源：NovBench (arxiv 2604.11543) 在 1,684 paper-review 上的实证发现——通用 LLM 做 novelty 评估有 3 类系统性偏见，必须主动反制：

1. **过度乐观偏见（Over-Positive）** — 通用 LLM 倾向给予更多赞扬、较少负面评价（GPT-4o 尤甚）。
   - 检查（**R21 修订**：改为以下 3 条中**至少 2 条同时满足**才触发反偏见动作，防止阈值过敏感误伤合理中高分 idea）：
     - **≥3 个维度落在 7+ 区间**（真正的多维度乐观信号）
     - **六维度加权后的均分 ≥7.0**（原 6.5 上调至 7.0，因为"典型顶会论文 (5.5-7.0)"区间的均分本来就能达到 6.5+，不应被误判）
     - **最低维度 ≥6**（原 ≥5 上调至 ≥6，因为六维度都 ≥5 仅仅是"无明显短板"而非"过度乐观"）
     - 额外一条作为硬条件检查（无论 2/3 是否触发都必须执行）：**是否主动列出了至少 3 个具体弱点？** 如果没有，即使主规则未触发也需补足弱点分析。
   - **R21 设计动机**：旧版三条阈值独立触发任一即启动，导致几乎所有 ≥5 分的 idea 都触发"过度乐观"反偏见，造成整个评分系统向下偏移。新版要求 2/3 同时触发，是为了**只拦截真正过度乐观的情况**，保留合理中高分空间。
   - 反偏见动作（触发后必须全部执行）：
     - 对任何 ≥7 分的维度，必须补 1 句"**审稿人最可能的攻击点**"
     - 对任何 ≥8 分的维度，必须补 1 句"**潜在被挑战点**"（"审稿人可能质疑..."）
     - 强制重新检查：是否遗漏了"组合型创新"、"术语包装"、"toy-only evidence" 三个反模式？若发现，回到 Phase 1 应用相应处罚规则

2. **多创新点遗漏（Missing Multi-Innovations）** — LLM 倾向只抓最显眼的 1 个创新点，忽视 2nd/3rd tier 创新。
   - 检查：我列的 Contribution Claims 是否穷尽了原文所有主张（方法+数据+分析+理论均扫一遍）？
   - 反偏见：主动问一遍"**除了最大创新点，还有什么新改动/新组件/新数据/新分析？**"——至少尝试补 1 条次要创新。

3. **Bold-Idea 保守偏见（Conservatism vs Bold）** — 社区观察（含 debashis1983 等开源项目的工程实验报告）普遍指出：human reviewer 对 bold ideas 的权重显著高于 writing clarity，但 LLM reviewer 倾向用 feasibility 顾虑拉低 novelty。具体回归系数为项目自报，非学界公认定论，仅作为"方向性提醒"使用。
   - 检查：我是否因"技术不成熟""实现复杂"就降低了 novelty 评分？
   - 反偏见：Novelty 评分 **只看"是否新"**，feasibility 是另一个独立维度——不要相互污染。

三条反偏见检查过关后，再进入 Phase 2 加权计算。在报告的"关键发现"段末尾，附一行：**"✅ 已执行 B0.5 反偏见自检"**，向用户显示你主动反制了 LLM 评估的系统性偏差。

### ⏸️ 检查点 B0.7：证据等级调节（Evidence Quality Gate）

在 Phase 2 加权之前，做一次证据等级核查：

**Toy-Only Evidence 折扣**：若支撑核心 claim 的实验 **全部** 来自 toy / synthetic 环境（确定性 verifier、人造任务、无真实 LLM reasoning trace、无工业合作数据、无 public benchmark 的真实效果验证），则：
- **维度五（应用价值）上限 ≤ 6 分**（除非有工业合作数据或真实场景 A/B 测试结果）
- **维度三（跨领域适用性）扣 1 分**（推广性未经真实场景验证，属于推测）
- 在综合评定"关键发现"段落必须标注 `[toy-only evidence]` 标签

**非 toy-only 豁免**：若存在任一：(a) 真实 LLM 上的 end-to-end 验证；(b) 工业合作或生产环境数据；(c) 已发表 benchmark 的 SOTA 对比——则豁免本条。

本检查点意图：防止"纸面上漂亮但没人真的跑过"的 claim 在应用价值/跨领域适用性上获得不成比例的高分。

### ⏸️ 检查点 B0.8：处罚叠加上限（Penalty Stacking Cap · 防止累积过度下调）

**设计动机**：skill 包含多条独立处罚规则（组合型贡献、术语包装、Toy-Only、Claim Audit 状态映射、Phase 0.8 硬限制等）。每条规则单独是合理的，但**一个 idea 同时命中 3-4 条时**会被叠加下调至 3-5 分区间——这会让"严苛但有一定价值的增量工作"获得与"纯重复复现"相同的分数，评分失去区分度（压缩了 4-7 分这一中间档）。

**硬性规则**：本维度评分**任何单条硬性 cap（上限封顶）最多只能作用于同一个维度一次**，且**同一份评估中，直接下调分数的硬性 cap 最多生效 2 条**。

**执行顺序**：当识别出同时触发 ≥3 条下调型硬处罚时，按以下优先级选出**最相关的 2 条**执行，其余**降级为"软提示"**（不影响维度分数，只在报告标签中列出供用户参考）：

| 优先级 | 处罚类型 | 理由（为什么最应保留） |
|-------|---------|----------------------|
| P1（最高） | Claim Audit `directly_refuted` → 维度一 ≤3 | 三层重合是最硬的先驱证据，不能妥协 |
| P2 | 组合型贡献 → 维度二 ≤4 | 组合无 1+1>2 价值的"拼装"属实质性缺乏方法创新 |
| P3 | 术语包装 → 维度四 ≤4 | 朴素白话剥离后若 gap 明显缩小，说明知识体系贡献虚高 |
| P4 | Toy-Only → 维度五 ≤6 | 无真实场景验证时的应用价值上限约束 |
| P5（最低） | Phase 0.8 单项硬限制（who cares/decisive test/residual value） | 反映未展开的初稿状态，应用到 Verdict 即可，不必再次下压六维度分数 |

**软提示（降级处罚）的处理**：被降级的处罚不直接扣分，但必须：
1. 在报告"关键发现"段落的标签行中列出（如 `[软提示: toy-only evidence]`）
2. 在 Worth-Doing Verdict 的依据中一句话提及
3. 在"创新性增强路径"中至少包含一条针对该软提示的改进建议

**豁免例外**：若一个 idea 的 Novelty Score 已落在 ≤ 3 的"无创新"档（如纯 API 调用复现型），无需启用本 cap——此时多层处罚叠加反映的是"这确实是低创新" 的事实，不存在"误伤"。

**反面保障**：B0.8 不得被用来"救援"真正低质量 idea。判断原则——**如果一个 idea 即使只触发 P1 单条 cap 也应评为 3-4 分（如纯 API 复现 + 方向完全撞车），那么 B0.8 不启用**；**如果触发了 3-4 条 cap 但其中至少一条是边际触发（判断存疑），那么 B0.8 启用**。

本检查点意图：让"严苛"聚焦在最有证据的单个短板上，而非把多个软性判断叠加成"压榨式低分"，同时维持对纯复现型 idea 的严苛评估。

### ⏸️ 检查点 B0.9：正向校准（Positive Calibration · 防止"只扣不加"失衡）

**设计动机**：此前 skill 含 10+ 条下调机制（硬 cap、refutation 降档、反模式处罚、极端分向中位收敛），但**没有任何"向上校准"的通道**。这导致一个结构性偏差——**具有真正优秀特质的 idea 无法被识别并上浮**，评分系统实际压缩到 3-5 区间而非宣称的 1-10。B0.9 修正这一结构性偏差：如果一个 idea **具备明确的正向特质**（而不是默认就该上浮），允许总分获得加分回报。

**硬性规则**：加分基于**客观可核查的正向特质**，每条独立判定。若同时满足多条，加分可叠加，但**加分总和上限 +1.5 分**（不得超过）。加分作用于 Phase 2 的 weighted total，不单独调整维度分。

**加分触发条件（严格，不得宽泛套用）**：

| # | 正向特质 | 判定标准 | 加分 |
|---|---------|---------|------|
| **B+1** | **跨学科概念桥梁** | 显式建立 ≥2 学科间从未被连接过的概念映射（例：语义熵连接信息论×NLP×认知不确定性）。判定关键：两学科都必须是"母学科"级别（信息论、控制论、统计物理、经济学、认知科学、神经科学等），"GNN + CV"不算跨学科。必须能写出"此工作揭示了 X 学科的 Y 概念等价于 Z 学科的 W 概念"的一句话。 | +0.5 |
| **B+2** | **引入新可测量量** | 定义了一个前所未见、可数值化、可在实验中测量的新度量（如"语义熵" over meanings、ResNet 提出的 shortcut 与梯度传播率）。**关键**：不是给旧指标换名字（即通过 A4 术语剥离测试），也不是复合已有指标（"accuracy + F1"不算）。必须有独立的测量方法和该度量特有的行为规律。 | +0.5 |
| **B+3** | **Fertile Ground 验证** | 明确列举 ≥3 个独立的后续研究方向（不是"可扩展到 X 数据集"这种琐碎外推，而是"本工作打开了 Y 子问题空间，可衍生 A/B/C 三类研究"）。判定标准：这 3 个后续方向如果分别被实现，每个都可发 ≥1 篇独立顶会论文。 | +0.3 |
| **B+4** | **Disruption + Novelty 双指标** | 同时具备：(a) Disruption——后续工作引用本工作时倾向**不再引用**本工作引用的前驱（切断祖先链）；(b) Novelty——连接了此前未被连接的知识单元。本条极难触发，**仅当评估者能对标 Attention/ResNet/Dropout 级别的改变领域轨迹特征时才适用**。 | +0.5 |
| **B+5** | **朴素但美（Michael Black 标准）** | 方法简单到"显而易见但此前无人做过"，且有明确的第一性原理支撑（如 ResNet 的 y = F(x) + x）。判定关键：剥离后**仍**保留简洁性（不是复杂方法的包装）；且有实验/理论支撑"这个简单改动抓住了问题本质"。 | +0.3 |

**硬性保护**：
1. **正向校准不得救援低质量 idea**：若 idea 的 Novelty Score 原始值（加分前）**低于 4.0**，则 B0.9 **最多只能加 +0.3**（即上限降为 +0.3，防止把 3.5 分 idea 通过加分抬到 5 分）。
2. **正向校准不得违反 refutation 事实**：若主 claim 已被 `directly_refuted`，则即使满足 B+1~B+5 的某些条件，加分上限降为 +0.5（因为此时"正向特质"其实是复述先驱工作的优点，不属于本工作贡献）。
3. **加分必须写证据**：每条触发的加分必须在报告中提供具体的**一句话证据**（例："B+1 跨学科桥梁：本工作建立了信息论 MI(X;T) 与 VAE 变分下界的等价关系，此前未见"）。仅标注条款编号不算证据。

**触发失败的处理**：如果一个 idea 未触发 B+1~B+5 中的任一条，总分**不加分**，但在报告"关键发现"段落必须给出**一句话解释**："未触发正向校准的原因：[具体判断]"——这防止评估者默认不去检查正向特质、直接扣分。

**B0.9 执行顺序**：
1. 完成 B0.8（处罚叠加 cap）后，得到六维度分数
2. 计算 Phase 2 加权初始总分
3. 逐条检查 B+1~B+5，记录触发条款与证据
4. 应用硬性保护（低分限制、refutation 限制）
5. 得到最终 Novelty Score = 初始总分 + B0.9 加分

本检查点意图：让评估器既能严苛地打压"组合复现"，也能识别和奖励"真正触及本质"的工作，避免整个评分系统坍缩到 3-5 分这一压缩档。

### Phase 2：综合评定

1. **加权总分计算**：
   - 基础总分 = Σ(各维度得分 × 权重)
   - **应用 B0.9 正向校准加分**：最终 Novelty Score = 基础总分 + B0.9 加分（≥0，上限 +1.5）
   - 四舍五入到一位小数
   - **示例**：若六维度得分为 6, 7, 5, 6, 8, 6 → 基础总分 = 6×0.20 + 7×0.25 + 5×0.15 + 6×0.15 + 8×0.10 + 6×0.15 = 1.20 + 1.75 + 0.75 + 0.90 + 0.80 + 0.90 = 6.3；若触发 B+1（跨学科桥梁）+ B+3（Fertile Ground），加分 +0.8 → 最终 **7.1 / 10**
   - **权重速查**：维度1=20%, 维度2=25%, 维度3=15%, 维度4=15%, 维度5=10%, 维度6=15%（合计100%）

2. **总体评级**：
   - **8.0-10.0分**：⭐⭐⭐ 卓越创新（Exceptional Novelty）——具有改变领域轨迹的潜力
   - **6.0-7.9分**：⭐⭐ 显著创新（Significant Novelty）——重要且有价值的创新贡献
   - **4.0-5.9分**：⭐ 一般创新（Moderate Novelty）——有一定新意但创新程度有限
   - **2.0-3.9分**：⚠️ 低创新（Low Novelty）——增量改进，创新性不足
   - **0.0-1.9分**：❌ 缺乏创新（Insufficient Novelty）——无明显创新贡献

3. **关键发现总结**：
   - 最强创新维度（亮点）
   - 最弱创新维度（短板）
   - 与同领域代表性工作的对比定位

4. **Risk-Reward Quadrant（R25 新增 · Novelty × Feasibility 2×2 映射）**：

   | | **Novelty ≥6** | **Novelty <6** |
   |--|--|--|
   | **Feasibility ≥6** | 🦄 **Unicorn**（罕见，值得全力投入） | 🔧 **Incremental-Practical**（适合 workshop / 工程论文 / 练手） |
   | **Feasibility <6** | ⚠️ **Plausible-Empty**（Taste Gate 核心警示 · "想得美做不出来"） | 🚫 **Avoid**（不值得投入） |

   象限解释：
   - **Unicorn**：既新又可做——科研金矿，全力投入
   - **Plausible-Empty**：novel 但极难做——若 Taste Gate 高且 residual value 存在仍可尝试小 probe；否则应避开
   - **Incremental-Practical**：不新但可做——适合新手练手、工程报告、已发表方法的新应用
   - **Avoid**：既不新又难做——除非有特殊动机，否则不投入

5. **Worth-Doing Verdict（综合 Taste Gate + Novelty Score + Feasibility Score + Quadrant 得出）**：

   按以下三维决策表给出最终 verdict（四档之一）：

   | Taste Gate | Novelty | Feasibility | Quadrant | Verdict |
   |-----------|---------|-------------|----------|---------|
   | ≥8 | ≥7 | ≥6 | Unicorn | **强烈建议投入** |
   | ≥7 | ≥6 | ≥6 | Unicorn | **值得做完整项目** |
   | ≥7 | ≥6 | <6 | Plausible-Empty | **值得尝试但小心陷阱**（先做小 probe 验证 feasibility，成功再升） |
   | 5-6 任一 | 任意 | ≥7 | — | **只值得做小 probe** |
   | 任意 | <4 | ≥7 | Incremental-Practical | **workshop 可考虑**（或跳过） |
   | 任意 | <6 | <4 | Avoid | **不建议投入主线** |
   | <5 任一 或 触发 Phase 0.8 硬限制 | — | — | — | **不建议投入主线** |

   **置换规则**：
   - Novelty 高 × Feasibility 低 → Quadrant = Plausible-Empty，Verdict 以 Taste Gate 和 residual value 共同决定；若 Taste Gate ≥7 且 residual value 存在，可给"值得尝试但小心陷阱"；否则"不建议投入主线"
   - Novelty 低 × Feasibility 高 → Quadrant = Incremental-Practical，至多给"workshop 可考虑"或"只值得做小 probe"
   - 两高一低或两低一高时以更严的那一边为准（保守）

   **Verdict 陈述须包含**：
   - 最终档位 + Quadrant 标签 + 一句主理由
   - 若上调/下调档位（偏离决策表），必须写明理由

### Phase 2.5：内部多视角自检（AgentReview启发·可选深度模式）

**触发条件**（任一满足即建议执行）：
- 总分落在 ⭐⭐⭐（≥8.0）或 ⭐（4.0-5.9）或 ❌（<4.0）等边缘/极端等级
- Refutation 判定出现 `unclear` 比例 >1/3
- 用户明确要求"做审稿模拟"或"多视角评估"

**设计来源**：AgentReview (EMNLP 2024, arxiv 2406.12708) 在 ICLR 2020-2023 数据上发现 **37.1% 决策变化来自单一 reviewer 的 bias**（social influence、altruism fatigue、authority bias）。通过三角色内部模拟可降低单视角盲区。

**三轮角色切换**（同一 assistant 依次扮演三角色，**各角色输出彼此隔离**）：

**Round 1 — Optimist Reviewer**（argue for acceptance，≤100 字）
从最支持接受的视角：2-3 条最强创新亮点 + 最被低估的贡献 + 最能说服 AC 的"独特价值主张"。

**Round 2 — Pessimist Reviewer**（argue for rejection，≤100 字）
从最反对的视角：2-3 条致命弱点 + 最易被判为 `directly_refuted` 或 `partially_refuted` 的 claim + 与已有工作最可疑的重叠点。

**Round 3 — Meta-Reviewer（Blind Synthesizer）**
借鉴 poldrack/ai-peer-review 的 **blind meta-review** 机制：**不参照**前两轮的角色标签，仅基于两份内容综合——
- Optimist 的亮点是否真实可靠，还是过度乐观？
- Pessimist 的批评是否有力（而非鸡蛋里挑骨头）？
- 最终总分需要 **±0.5 范围** 内的调整吗？

**输出格式**（附加到报告"关键发现"段之后）：
```
### 多视角自检（Phase 2.5）
- Optimist 提示：[亮点/上调理由 1 句]
- Pessimist 警示：[弱点/下调理由 1 句]
- Meta 综合：总分净调整 = ±X.X / 维持不变
- 置信度：High / Medium / Low（基于两方论据的质量分歧度）
```

此机制**作为启发式红队自检**使用，设计灵感来自 AgentReview (EMNLP 2024) 的多角色流程与 Poldrack ai-peer-review 的 blind meta-review 思路，但本处并不宣称已有独立实证收益数据。把它当成"强迫系统从对立立场重看同一个 idea"的手段即可。

### Phase 3：创新性增强建议

根据评估结果，**只针对评分最低的2-3个维度**提供改进建议，每个维度2-3条，共4-6条建议。从以下方向选择最相关的：

1. **深化第一性原理**：如何让方法更贴近问题本质？
2. **扩大研究空白**：如何找到更独特的研究切入点？
3. **增强方法论深度**：如何让方法更简洁、更优雅？
4. **拓展适用范围**：如何提升方法的普适性？
5. **强化理论贡献**：如何构建新的概念或框架？
6. **提升叙事能力**：如何让论文更有说服力和传播力？

每条建议格式：**当前问题** → **改进方向** → **预期效果**（具体到可执行的下一步）

### ⏸️ 检查点B：报告交付

报告输出后，主动询问用户：
> 对评估结果有异议或想深入探讨的维度吗？
> 需要我针对某条改进建议展开更详细的方案吗？

---

## 输出格式

生成一份结构化的中文创新性分析报告，使用Markdown格式，包含以下部分。

**长度控制**（避免过长，保持精炼可用）：
- 整份报告控制在 **1500-2500字** 之间
- 每个维度评分详析：**150-300字**（核心论据 + refutation判定 + 1个对标案例）
- 关键发现：**100字内**
- 增强建议：每条 **50-80字**
- 参考文献：列出 **3-8个** 关键引用即可

**优先质量**：宁可在某个关键维度深挖到400字（如方法论创新），也不要为凑字数在不重要维度灌水。


```
# 🔬 研究创新性评估报告

## 📌 核心判断速览（Decision Dashboard）

> 这一段是整份报告最重要的一屏——任何人扫一眼就应该能判断"这事到底值不值得做、新不新"。
> 格式固定，**不可省略任一字段**。

| 字段 | 值 | 一句话依据 |
|------|-----|----------|
| **Worth-Doing Verdict** | 强烈建议投入 / 值得做完整项目 / 只值得做小 probe / 不建议投入主线 | ... |
| **Novelty Score** | X.X / 10 | Phase 2 六维度加权 |
| **Taste Gate** | X / 10 | 见 Phase 0.8 八问 |
| **Feasibility Score** | X / 10 | Phase 0.9 四子项（算力/数据/时间/团队）均值；与 Novelty 正交 |
| **Risk-Reward Quadrant** | Unicorn / Plausible-Empty / Incremental-Practical / Avoid | 基于 Novelty × Feasibility 2×2 映射（见 Phase 2 决策表） |
| **Confidence** | High / Medium / Low | 取决于证据充分度与 claim audit 状态分布 |
| **Strongest Prior** | [论文名 / First Author / Year / Venue] | 最可能击穿本工作的先驱，或"未识别出强先驱" |
| **Risk Factors** | 3-5 条清单（见"Risk Factors 明细"段） | 借鉴 AI-Scientist-v2 (Sakana AI 2024) FinalizeIdea 的 `Risk Factors` 字段——穷尽式列举所有潜在风险，而非只看最致命一条 |
| **Biggest Killer Risk** | [一句] | Risk Factors 中最致命单项的一句提炼（必须是 Risk Factors 清单中某一条，不得另造） |
| **Cheapest Decisive Test** | 3 步结构（Setup / Measurement / Success Criterion） | 借鉴 AI-Scientist-v2 `Experiments` 字段（perform_ideation_temp_free.py:36 "simple and feasible"）——一句话无法让用户直接执行，必须结构化 |
| **Residual Value if Fails** | [一句或"几乎无"] | 主假设失败后仍可发表的残值 |

**硬规则**：
- Strongest Prior 不能写"无"。要么给出具体论文/作者，要么明说"检索不充分，未识别"——但后者会触发 Confidence = Low。
- **Risk Factors 必须 ≥3 条**。每条格式固定为 `风险点 | 触发条件 | 对 Novelty/Feasibility 的影响`。若少于 3 条，Confidence 强制 = Low，且必须在"关键发现"段说明"为何风险识别不足"（AI-Scientist-v2 设计哲学：穷尽式列举比挑出最严重风险更有价值，前者便于 reviewer/author 展开讨论）。
- Biggest Killer Risk 必须是**单一最尖锐风险**，且必须**是 Risk Factors 中某一条的浓缩**——不允许另造一条 Dashboard 里没列的风险（保证 Dashboard 内部一致性）。
- **Cheapest Decisive Test 的 3 步必须全部写出**：①Setup（实验配置一句）；②Measurement（测什么指标，要具体到数值化标准）；③Success Criterion（阈值/对比方式/定性判定准则）。三步中任一步写不出（写"待定""不清楚""需要探索"），Worth-Doing Verdict 最高只能是"只值得做小 probe"，且 Confidence 不得高于 Medium。

## 一、研究概要
- 核心任务：[一句话描述]
- 主要贡献声明：[列出1-3个创新点]
- 所属领域：[领域标签]

## 二、Taste Gate（值不值得做）
- **Taste Gate 分数：X/10**
- 结论：强烈建议投入 / 值得做完整项目 / 只值得做小 probe / 不建议投入主线
- 关键依据：
  - Who cares（谁会在乎）：[一句]
  - Why now（为什么现在）：[一句]
  - Reasonable attack（攻击路线）：[一句或"未识别"]
  - Decisive test（最小决定性验证）：[一句或"未识别"]
  - Residual value（失败后残值）：[一句或"几乎无"]
- 触发的硬限制（若有）：[列出对后续评分的封顶，或"无"]

## 二·附、Feasibility Assessment（Phase 0.9 · R25 新增）
- **Feasibility Score：X/10**（四子项均值）
  - F1 算力需求（1-10）：[值 + 一句话依据]
  - F2 数据可得性（1-10）：[值 + 一句话依据]
  - F3 时间成本（1-10）：[值 + 一句话依据]
  - F4 团队能力（1-10）：[值 + 一句话依据]
- **最大 feasibility 风险**：[一句话，对应 F1-F4 中最低分子项]
- **Risk-Reward Quadrant**：🦄 Unicorn / ⚠️ Plausible-Empty / 🔧 Incremental-Practical / 🚫 Avoid（基于 Novelty × Feasibility 2×2 映射）

## 三、六维度评分详析

### 3.1 问题重要性与文献空白（X/10）
[详细分析 + 文献证据 + refutation检验结果]

### 3.2 方法论创新与第一性原理（X/10）
[详细分析 + 第一性原理检验 + 与经典创新案例的对标]

### 3.3 跨领域适用性与推广潜力（X/10）
[详细分析]

### 3.4 对学科知识体系的影响（X/10）
[详细分析]

### 3.5 应用价值与市场需求（X/10）
[详细分析]

### 3.6 学术影响力潜力（X/10）
[详细分析]

## 四、综合评定
- 加权总分（Novelty Score）：X.X / 10
- 总体评级：[等级 + 星标]
- **各维度得分柱状图**（统一用以下ASCII格式输出，█代表1分）：
  ```
  问题重要性  (20%) │██████░░░░│ 6/10
  方法论创新  (25%) │███████░░░│ 7/10
  跨领域适用  (15%) │█████░░░░░│ 5/10
  知识体系    (15%) │██████░░░░│ 6/10
  应用价值    (10%) │████████░░│ 8/10
  学术影响    (15%) │██████░░░░│ 6/10
  ─────────────────────────────────
  加权总分：6.3/10  ⭐⭐ 显著创新
  ```
- **Worth-Doing Verdict**：[强烈建议投入 / 值得做完整项目 / 只值得做小 probe / 不建议投入主线]
  - 主理由：[1 句，引用 Taste Gate 与 Novelty Score 的综合判断]
  - 若偏离决策表（表见 Phase 2）：写明升/降档理由

## 五、关键发现
- 最强创新维度：[维度名] — [原因]
- 最弱创新维度：[维度名] — [原因]
- 领域定位：[与代表性工作的对比]

## 六、创新性增强路径
[针对最弱2-3个维度，共4-6条改进建议，每条说明：
 当前问题 → 改进方向 → 预期效果]

## 七、参考文献与证据链
[列出评估过程中参考的关键文献]

## 七·附：Reflection Pass（Phase 1.5 · 借鉴 AI-Scientist-v2 num_reflections）

按轻量或深度模式输出（见 Phase 1.5 章节规范），必须包含：
- 最弱证据点（6 维度各一句）
- 次要 Claim 扫描结果
- 修正清单（若无修正，显式写"无修正"）
- 置信度更新

## 八、Risk Factors 明细（Dashboard 字段的展开版）

逐条展开 Dashboard 中的 `Risk Factors` 字段（≥3 条）：

1. **[风险点一句提炼]** | **触发条件**：[什么情况下这个风险会变为现实] | **影响**：[对 Novelty 分数下调 N 分 / 对 Feasibility 限制 / 对 Verdict 降档]
2. **[风险点]** | **触发条件**：... | **影响**：...
3. **[风险点]** | **触发条件**：... | **影响**：...

## 九、Cheapest Decisive Test 设计（Dashboard 字段的展开版）

借鉴 AI-Scientist-v2 `Experiments` 字段 + Hamming "reasonable attack" 思想，必须同时给出三步：

- **Setup**：[实验配置一句——数据集/模型规模/训练预算]
- **Measurement**：[测什么指标——必须数值化，如"Pearson 相关性"、"accuracy 差值"、"loss 收敛曲线的斜率"]
- **Success Criterion**：[阈值或对比或定性判定——如"相关性 ≥0.5"、"超越 baseline ≥3 pts"、"出现/不出现某行为"]

**若三步任一步写不出**，在 Dashboard 的 `Cheapest Decisive Test` 字段中明示"[此步未能写出]"，Verdict 自动降档至"只值得做小 probe"。

## 十、结构化输出（JSON 镜像 · 便于下游工具链解析）

借鉴 AI-Scientist-v2 `extract_json_between_markers`（perform_ideation_temp_free.py + perform_llm_review.py），在 Markdown 报告末尾**必须**追加一段 JSON 代码块。字段与上文数据**必须一致**（不一致则评估无效——评估者需自纠）。

```json
{
  "dashboard": {
    "worth_doing_verdict": "...",
    "novelty_score": 0.0,
    "taste_gate": 0,
    "feasibility_score": 0.0,
    "feasibility_breakdown": {
      "f1_compute": 0,
      "f2_data": 0,
      "f3_time": 0,
      "f4_team": 0
    },
    "risk_reward_quadrant": "Unicorn / Plausible-Empty / Incremental-Practical / Avoid",
    "confidence": "High / Medium / Low",
    "strongest_prior": {
      "paper_name": "...",
      "first_author": "...",
      "year": 0,
      "venue": "..."
    },
    "risk_factors": [
      {"risk": "...", "trigger": "...", "impact": "Novelty -N / Feasibility limit / Verdict downgrade"},
      {"risk": "...", "trigger": "...", "impact": "..."},
      {"risk": "...", "trigger": "...", "impact": "..."}
    ],
    "biggest_killer_risk": "Risk Factors 中最致命单项的一句提炼",
    "cheapest_decisive_test": {
      "setup": "...",
      "measurement": "...",
      "success_criterion": "..."
    },
    "residual_value_if_fails": "..."
  },
  "six_dimensions": {
    "problem_importance": 0,
    "methodology_innovation": 0,
    "cross_domain": 0,
    "knowledge_system_impact": 0,
    "application_value": 0,
    "academic_impact": 0
  },
  "base_weighted_score": 0.0,
  "b09_bonus": {
    "total": 0.0,
    "triggers": ["B+1 / B+2 / B+3 / B+4 / B+5 中触发的条款"],
    "evidence_per_trigger": ["触发条款对应的一句话证据"]
  },
  "final_novelty_score": 0.0,
  "claim_audit": [
    {"claim": "...", "status": "directly_refuted / partially_refuted / not_refuted_yet / evidence_insufficient", "prior_ref": "..."}
  ],
  "reflection_history": {
    "mode": "lightweight / deep",
    "trigger_reason": "default / variance_ge5 / evidence_insufficient / user_request / multi_claim",
    "weakest_evidence_per_dim": {
      "d1": "...", "d2": "...", "d3": "...", "d4": "...", "d5": "...", "d6": "..."
    },
    "secondary_claims_found": [
      {"claim": "...", "status": "directly_refuted / partially_refuted / not_refuted_yet / evidence_insufficient"}
    ],
    "score_adjustments": [
      {"dim": "d1-d6", "before": 0, "after": 0, "evidence": "..."}
    ],
    "risk_factors_added": ["..."],
    "confidence_change": "High → Medium / 维持 / Medium → High / Low → Medium",
    "secondary_search_performed": true
  },
  "meta": {
    "skill_version": "R24-reflection-loop",
    "evaluation_date": "YYYY-MM-DD"
  }
}
```

**硬自检**（生成 JSON 后必须逐项核对）：
- JSON 中 `novelty_score` 必须 = Markdown 第四节加权总分
- JSON 中 `risk_factors` 条数必须 = Markdown 第八节条数（且 ≥3）
- JSON 中 `cheapest_decisive_test` 三字段必须全部非空
- JSON 中 `claim_audit` 每条状态必须落在四状态枚举内
- JSON 中 `reflection_history.mode` 必须是 `lightweight` 或 `deep`；若 `mode=deep` 则 `weakest_evidence_per_dim` 六字段全非空
- JSON 中 `reflection_history.score_adjustments` 每条的 `before` 与 `after` 差 **≤0.3**（超出即意味着 Reflection 变成了重评，必须修正）
- JSON 中 `feasibility_score` 必须等于 `feasibility_breakdown` 四子项均值（误差 ≤0.05）；四子项每项必须在 1-10 区间
- JSON 中 `risk_reward_quadrant` 必须与 `novelty_score` / `feasibility_score` 的 2×2 映射一致（Novelty≥6 × Feasibility≥6 = Unicorn；Novelty≥6 × Feasibility<6 = Plausible-Empty；Novelty<6 × Feasibility≥6 = Incremental-Practical；Novelty<6 × Feasibility<6 = Avoid）

若自检不通过，评估者必须修正后重新输出，不得带着错误数据交付用户。
```

---

## 输出示例（One-Shot 参考）

以下是对「用 RL 优化 LLM 推理链（CoT）的跳过/展开决策」这一典型 idea 的**浓缩完整示例**（≈500 字），供执行时作为 one-shot 参考。实际输出应是此示例的 3-4 倍详细度。

```
# 🔬 研究创新性评估报告

## 📌 核心判断速览

| 字段 | 值 | 一句话依据 |
|------|-----|----------|
| Worth-Doing Verdict | 只值得做小 probe | Taste Gate 中等 + Novelty 偏低 + 主 claim 被直接击穿 + Feasibility 高（但 novelty 不足挽救）|
| Novelty Score | 4.65 / 10 | 六维度加权，受 R1/o1 先驱拖累 |
| Taste Gate | 5 / 10 | 问题重要但 why-now 弱，attackability 中等 |
| Feasibility Score | 7.3 / 10 | F1=8(单A100可跑) / F2=9(GSM8K公开) / F3=7(PhD 1月) / F4=5(需RL调参经验)|
| Risk-Reward Quadrant | 🔧 Incremental-Practical | Novelty<6 × Feasibility≥6 → 适合 workshop 或练手，非 Unicorn |
| Confidence | Medium | 主要先驱已清晰，次级 claim 证据一般 |
| Strongest Prior | DeepSeek-R1 (2025) | GRPO 训练范式在问题/概念/机制三层与本 claim 高度重叠 |
| Risk Factors | 3 条（见"八、Risk Factors 明细"） | 全部为 Novelty 风险，Feasibility 风险中等 |
| Biggest Killer Risk | 复合 reward 两项对应 R1 的 rule-based reward + length penalty，无独立机制（Risk Factors #1 浓缩） | 这条是 Dashboard 清单第 1 条的提炼 |
| Cheapest Decisive Test | 3 步（见"九、Cheapest Decisive Test 设计"） | Setup/Measure/Criterion 三步齐全 |
| Residual Value if Fails | 若跳过/展开策略学不到，仍能发一篇"推理粒度可学性"的负结果 |

## 一、研究概要
- 核心任务：用 RL 让 LLM 自主决定推理链的展开/跳过粒度
- 主要贡献声明：①基于答案正确性+步骤简洁性的复合 reward；②推理粒度可学习
- 所属领域：LLM 推理 / RL 对齐

## 二、六维度评分详析

### 2.1 问题重要性与文献空白（5/10）
推理成本优化是真实痛点，但"RL 驱动推理优化"已被 DeepSeek-R1 (2025)、o1 (2024) 大量探索。Claim Audit 判定 `directly_refuted`——核心思路与 R1 的 GRPO 训练范式在问题/概念/机制三层都高度重叠。

### 2.2 方法论创新与第一性原理（4/10）
复合 reward 属于对 R1 范式的增量修改，未触及推理的第一性问题（信息何时充分？）。Fertile Ground 检验：难 chain 出独立后续工作。对标 Michael Black 标准：简单但**不美**，更偏工程组合。

### 2.3 跨领域适用性（6/10）
粒度决策思想可迁移到代码生成、工具调用等场景，但需重新设计 reward。

### 2.4 学科知识体系影响（3/10）
未建立新概念或框架，在 RLHF/RLAIF 既有范式内运作。

### 2.5 应用价值（7/10）
推理成本降低直接对应 API 账单，工业需求明确。

### 2.6 学术影响力（4/10）
可投 workshop 或二流会议；难在 NeurIPS/ICLR 获高分，受先前工作重叠拖累。

## 三、综合评定
各维度柱状图：
```
问题重要性  (20%) │█████░░░░░│ 5/10
方法论创新  (25%) │████░░░░░░│ 4/10
跨领域适用  (15%) │██████░░░░│ 6/10
知识体系    (15%) │███░░░░░░░│ 3/10
应用价值    (10%) │███████░░░│ 7/10
学术影响    (15%) │████░░░░░░│ 4/10
─────────────────────────────────
加权总分：4.65/10  ⭐ 一般创新
```
B0 自检：极差 = 7-3 = 4（未超 5，通过）；无极端分；锚点对标≈"一般改进性工作"，符合。

## 四、关键发现
- 最强维度：应用价值（推理成本是真实商业痛点）
- 最弱维度：知识体系影响（无新概念/框架）
- 领域定位：R1 范式内的增量微调，非独立贡献

## 五、创新性增强路径（针对最弱 2 维度，4 条）
1. **方法论**：当前=复合 reward → 改进=引入"推理不确定性"作为隐式 reward（参考语义熵）→ 预期=触及第一性问题
2. **方法论**：当前=RL 范式同 R1 → 改进=尝试 RL-free 的推理粒度自适应（如贝叶斯停止准则）→ 预期=跳出 refutation 范围
3. **知识体系**：当前=无概念贡献 → 改进=提出"推理经济性"度量体系 → 预期=新分析视角
4. **知识体系**：当前=RLHF 框架内 → 改进=建立推理粒度与信息论熵率的关系 → 预期=跨学科桥梁

## 六、参考文献
- DeepSeek-R1 (2025)；OpenAI o1 (2024)；语义熵 (Farquhar et al. 2024)；GRPO paper

## 七·附、Reflection Pass（轻量 · Phase 1.5）
- **最弱证据点（6 维度）**：
  - D1: 假设 R1 GRPO 范式重合度是 100%，未查证最新 o1 pro 报告
  - D2: 组合型处罚施加较重，未检查是否有"reward shaping 防注入"的独立机制创新
  - D3: "推理粒度可迁移代码生成"是推测，无 toy 验证
  - D4: 未检查"推理粒度"是否已被某综述视为独立概念
  - D5: 应用价值主要基于直觉（"推理 token 贵"），未量化
  - D6: 引用潜力估计粗糙（workshop-only）
- **次要 Claim 扫描**：用户描述"自动学会什么时候展开"暗含"跳过决策"的二元控制信号——是否与 Early Exit / Adaptive Computation 文献（Graves 2016, ACT; Schuster et al. 2022, Confident Adaptive Language Models）重合？→ **补录次要 claim**：`partially_refuted`（ACT/CALM 已有自适应深度概念，但未用 RL 学习）
- **修正清单**：无分数调整（次要 claim 与主 claim 叠加后不改变维度分），但 Risk Factors 清单新增 1 条："ACT/CALM 等自适应计算文献可能被 reviewer 引为先驱"
- **置信度变化**：维持 Medium

## 七、Risk Factors 明细
1. **复合 reward 的 length penalty 可能直接注入长度偏好（而非让模型学会决策）** | **触发条件**：训练后分析发现 reward 组件权重已把 length penalty 直接映射到 CoT 长度，模型没学会"根据难度自适应" | **影响**：Novelty 下调 1-2 分（方法论维度），Verdict 降档
2. **GRPO 对新复合 reward 的稳定性未知** | **触发条件**：训练 loss 不收敛、梯度方差 > 10x baseline、或需要大量 hyperparam search | **影响**：Feasibility 风险，时间成本放大 3-5 倍
3. **与 o1/R1 的差异仅在 reward 设计，可能被 reviewer 判为同一范式内的 reward engineering** | **触发条件**：NeurIPS/ICLR reviewer 找到 o1 技术报告中类似 reward 设计的模糊描述 | **影响**：Novelty 再下调 0.5-1 分，Verdict 变为"不建议投入主线"

## 八、Cheapest Decisive Test 设计
- **Setup**：小模型（1B param，如 Qwen2.5-1.5B）在 GSM8K 子集上跑 1-epoch GRPO，复合 reward = α·正确性 + β·(-|CoT 长度 - 难度_理想|)
- **Measurement**：训练后在 held-out set 上抽样 100 条推理轨迹，人工标注"展开粒度是否与题目难度相关"，计算 Pearson 相关系数
- **Success Criterion**：相关性 ≥ 0.5 → 粒度真被学到（值得推进）；0.3-0.5 → 模糊（需更大实验）；<0.3 → 只是 reward shaping 直接注入，Verdict 降为"不建议投入"

## 九、结构化输出（JSON 镜像）
```json
{
  "dashboard": {
    "worth_doing_verdict": "只值得做小 probe",
    "novelty_score": 4.65,
    "taste_gate": 5,
    "feasibility_score": 7.3,
    "feasibility_breakdown": {
      "f1_compute": 8,
      "f2_data": 9,
      "f3_time": 7,
      "f4_team": 5
    },
    "risk_reward_quadrant": "Incremental-Practical",
    "confidence": "Medium",
    "strongest_prior": {
      "paper_name": "DeepSeek-R1",
      "first_author": "DeepSeek Team",
      "year": 2025,
      "venue": "Technical Report / arxiv"
    },
    "risk_factors": [
      {"risk": "复合 reward length penalty 直接注入长度偏好", "trigger": "训练后 reward 组件分析显示直接映射", "impact": "Novelty -1-2"},
      {"risk": "GRPO 对新复合 reward 稳定性未知", "trigger": "loss 不收敛 / 梯度方差 >10x baseline", "impact": "Feasibility 风险 / 时间 3-5x"},
      {"risk": "与 o1/R1 差异仅在 reward 设计", "trigger": "reviewer 找到 o1 技术报告类似描述", "impact": "Novelty -0.5-1 / Verdict 降档"}
    ],
    "biggest_killer_risk": "复合 reward length penalty 直接注入长度偏好（Risk Factors #1）",
    "cheapest_decisive_test": {
      "setup": "Qwen2.5-1.5B 在 GSM8K 子集跑 1-epoch GRPO，复合 reward = α·正确性 + β·(-|len - 难度_理想|)",
      "measurement": "100 条推理轨迹人工标注'粒度 vs 难度'相关性，计算 Pearson",
      "success_criterion": "≥0.5 推进 / 0.3-0.5 模糊 / <0.3 降 Verdict"
    },
    "residual_value_if_fails": "可发一篇'推理粒度可学性'的负结果 workshop 论文"
  },
  "six_dimensions": {
    "problem_importance": 5,
    "methodology_innovation": 4,
    "cross_domain": 6,
    "knowledge_system_impact": 3,
    "application_value": 7,
    "academic_impact": 4
  },
  "base_weighted_score": 4.65,
  "b09_bonus": {
    "total": 0.0,
    "triggers": [],
    "evidence_per_trigger": ["未触发：复合 reward 属 R1 范式内增量，无跨学科桥梁/新可测量量/Fertile Ground ≥3"]
  },
  "final_novelty_score": 4.65,
  "claim_audit": [
    {"claim": "用 RL 优化 LLM 推理链展开/跳过决策", "status": "directly_refuted", "prior_ref": "DeepSeek-R1 (2025)"}
  ],
  "reflection_history": {
    "mode": "lightweight",
    "trigger_reason": "default",
    "weakest_evidence_per_dim": {
      "d1": "R1 范式 100% 重合未查最新 o1 pro",
      "d2": "组合处罚重,未查 reward 防注入机制创新",
      "d3": "代码生成迁移是推测,无 toy",
      "d4": "推理粒度独立概念未查证",
      "d5": "token 成本无量化",
      "d6": "workshop-only 估计粗"
    },
    "secondary_claims_found": [
      {"claim": "跳过决策的二元自适应控制信号", "status": "partially_refuted"}
    ],
    "score_adjustments": [],
    "risk_factors_added": ["ACT/CALM 自适应计算文献可能被 reviewer 引为先驱"],
    "confidence_change": "维持 Medium",
    "secondary_search_performed": false
  },
  "meta": {
    "skill_version": "R25-feasibility-quadrant",
    "evaluation_date": "2026-04-18"
  }
}
```
```

**使用此示例时的注意**：实际报告应按「长度控制」章节的字数要求（1500-2500 字）扩展，此处仅为**结构参考**和**评分档位校准**。R23 新增的 Risk Factors 明细、Decisive Test 3 步、JSON 镜像三段是**必填**（不是可选）。R24 新增的 Reflection Pass 七·附段落也是**必填**（轻量模式 ≤200 字，深度模式 ≤500 字）。R25 新增的 Feasibility Score 四子项 + Risk-Reward Quadrant + 决策表三维升级也是**必填**——Feasibility 和 Novelty 正交，不得互相拖累。

---

## 评估原则与注意事项

### 严苛但公正
- 评分要严苛，不要"分数膨胀"——大多数研究应落在4-7分区间
- 真正能达到9-10分的创新极其罕见（参考标准：Attention、ResNet、Dropout级别）
- 但也不要过度苛刻——好的增量工作值得5-6分的肯定
- **NeurIPS标准**：探索新领域或指出新方向的工作优于增量提升SOTA的工作
- **近年顶会审稿趋势**：缺少 SOTA 结果本身不构成拒绝理由——只要能 convincing 地展示 new / relevant / impactful knowledge，即可被认可（以具体会议当期指引为准，不做"所有 ICLR 会议均如此"的普遍性断言）
- **宏观趋势警示**（Nature 2023）：全领域论文正变得越来越不disruptive。评估当代idea时，要区分"真正改变学科轨迹的创新"和"在既有框架内优化的增量工作"，对前者给予更高分

### Hamming标准：重要性 × 可进攻性
- Richard Hamming（Bell Labs, 与Shannon/Feynman共事40年）的核心判断：
  - 好问题 = 问题的重要性 × 是否有合理的进攻方式（reasonable attack）
  - "如果你不在重要的问题上工作，你不太可能做出重要的工作"
  - 伟大科学家维持10-20个重要问题清单，发现新技术时立刻出击
  - 突破常来自局外人——碳定年来自物理学，飞机来自自行车专家

### Michael Black标准：美 > 复杂性
- Michael Black（MPI，CVPR顶级学者）的审稿哲学：
  - **简单≠缺乏创新**："只改了loss函数中的一项"——如果没人想到，它就是创新
  - 用"美"（beauty）替代"创新性"来评估——美剥离了对复杂性的盲目追求
  - **简单 vs 平凡**：平凡=不重要，简单但有效=可能是伟大的发现
  - **"显而易见"陷阱**：好idea被提出后看起来理所当然——这恰恰是好idea的标志
  - 评估创新性必须在idea存在之前的状态下评估，而非事后回看
  - 创新性有多种形式：新数据集、旧方法新用法、简单替代复杂——都可能是真正的创新

### 顶会审稿实证发现
- ICLR论文最常见创新性缺陷：与先前工作重叠 > 缺乏原创性 > 贡献不清晰
- 论文评分与"贡献"维度相关性最强，其次是"soundness"
- 高分论文共同特征：创新性 + 优雅方法论（elegant and efficient models）
- 低分论文共同缺陷：写作不清 + 实验缺陷（weak baselines）

### 证据驱动
- 所有判断必须有文献或逻辑支撑
- 对 `directly_refuted` / `partially_refuted` 的判定要提供具体的先驱工作引用和"重叠层级"说明（概念/机制/实验）
- 对 `not_refuted_yet` 的判定要说明"最强先驱是谁、本工作比它新在哪个层级"，不允许省略这一步
- 对 `evidence_insufficient` 的判定要显式标注置信度 Low 并不得推高分数

### 建设性导向
- 评估的目的不是打击，而是帮助研究者提升创新性
- 改进建议必须具体、可操作
- 指出问题时同时提供解决思路

### 上下文敏感
- 考虑研究所处的发展阶段（早期探索 vs 成熟方向）
- 考虑不同学科对"创新"的定义差异
- 考虑实际约束（算力、数据、时间）

### 反模式识别
- 警惕"创新性通胀"：仅仅是换了个数据集/任务就声称创新
- 警惕"组合即创新"：简单堆叠现有方法并不构成真正创新
  - **硬性处罚规则**（执行层）：若核心 novelty claim 的本质是"将 N≥2 个已有组件组合"，且每个组件在近 2 年文献中均可找到独立先驱，则：
    - **维度二（方法论创新）上限 ≤ 4 分**（封顶，除非组合产生了明显的 1+1>2 理论突破并有实证支撑）
    - **维度一（文献空白）扣 1-2 分**——"没人把这几个东西拼一起"的空白价值远低于"没人做过这个问题"
    - 在综合评定"关键发现"段落必须标注 `[组合型贡献]` 标签
    - 不满足这些条件的评分需要在报告中显式给出"组合如何产生了非增量价值"的论证
- 警惕"术语包装"：用新术语包装旧思想
- 警惕"基准刷分"：在特定benchmark上微调到SOTA不等于创新
- 警惕"Hack当创新"：数学上能自洽但和物理/逻辑机制无关的方法站不住脚
- 警惕"复杂性崇拜"：故意让方法显得更复杂以通过审稿是反科学的——简单但有效才是最高境界
- 警惕"Smart Plagiarism"（LLM时代特有）：方法论与已有工作高度重叠，但通过术语转换、结构重排、符号替换规避传统重复检测。检验方法：抽离术语和符号后，核心数学/算法结构是否与已有工作等价？来源：Stanford 100+ NLP研究者研究（arxiv 2409.04109）发现LLM生成idea的novelty得分显著高于人类，但feasibility偏低，需警惕"看起来新颖但实质重复"
- 警惕"Prompt即创新"（LLM时代特有）：仅依靠prompt engineering调用现有模型不构成方法论创新，除非：(a)揭示了模型能力边界的新发现；(b)建立了新的理论框架；(c)设计的prompt范式具有跨模型/跨任务的普适价值

### 深度检验
- "idea的新颖程度直接代表你对问题理解的深入程度"
- 真正的创新来自对问题脉络的深刻理解——回溯到研究的根节点
- 如果一个idea没有任何真实挑战（非人为编造），它的创新性就大打折扣

### LLM时代的特别注意事项（2024-2026）
- **LLM-generated idea的novelty膨胀现象**：Stanford研究发现LLM生成的idea在novelty评分上显著高于人类专家（p<0.05），但**feasibility偏低**。评估此类idea时要交叉检验：novelty高但feasibility低 → 可能是"看起来新颖但实质不可行"
- **"novel but not disruptive"陷阱**：LLM擅长生成"重组式novelty"（连接已有概念），但很难生成"disruptive novelty"（颠覆范式）。评估时优先奖励真正颠覆的工作，而非仅"重组得巧妙"
- **AI辅助研究时代人类的独特贡献**：
  - 问题品味（research taste）——选择什么问题值得做
  - 跨学科直觉——建立LLM不熟悉的概念桥梁
  - 真实挑战识别——区分"人为编造的问题"和"真实存在的痛点"
  - 第一性原理思考——回到物理/数学/逻辑的根基
- **ICLR 等顶会近年普遍要求披露 LLM 使用**（具体政策各会议不同且时有更新，评估时应以用户投稿会议当期 call-for-papers 为准，不做"自动拒稿"式的硬断言）：评估中如怀疑 idea 由 LLM 生成，应主动追问"人类贡献部分是什么"

---

## 附录：经典创新案例参考锚点

以下案例作为评分的参考锚点，帮助校准评分尺度：

| 案例 | 核心创新 | 参考评分 | 原因 |
|------|----------|----------|------|
| Dropout | 随机关闭神经元防止过拟合 | 9.5 | 极简但深刻，改变了正则化范式 |
| ResNet残差连接 | y = F(x) + x | 9.5 | 一个加法解决梯度消失，开启深层网络时代 |
| Attention机制 | 加权求和实现可微检索 | 10 | 打破时序依赖，奠定Transformer基础 |
| ReLU | max(0, x) | 9.0 | 极简非线性，解决梯度消失+计算加速 |
| Word2Vec负采样 | 多分类→二分类 | 9.0 | 降维思想+对比学习的先驱 |
| BERT MLM | 挖空填词的自监督 | 9.0 | 利用数据内部监督信号 |
| LoRA | 低秩矩阵分解微调 | 8.5 | 穷人的核武器，民主化大模型微调 |
| CoT思维链 | "请一步步思考" | 8.5 | 用Prompt释放推理能力 |
| 语义熵 | 从语义层面度量不确定性 | 8.5 | 跨学科概念桥梁，新建模视角 |
| DeepSeek-R1 (2025) | RL驱动的推理能力涌现 | 8.5 | 证明纯RL + 简单reward就能涌现推理，挑战SFT范式 |
| Mamba/SSM (2024) | 线性复杂度序列建模 | 8.0 | 从控制论借鉴状态空间模型，挑战Transformer主导 |
| Constitutional AI (2024) | 用AI反馈替代人类反馈 | 8.0 | RLAIF降低RLHF对齐成本，可扩展性创新 |
| 典型的顶会论文 | 在某方面有明确进步 | 5.5-7.0 | 有价值但非范式变革 |
| 一般改进性工作 | 增量性能提升 | 3.5-5.0 | 有贡献但创新有限 |
| LLM-generated "plausible" idea | 看似新颖但实质重组 | 2.5-4.0 | Smart Plagiarism模式，novelty得分高但缺少真实挑战 |
