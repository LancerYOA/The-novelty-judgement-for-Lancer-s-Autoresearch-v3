# 创新性评估方法论参考

## 一、OpenNovelty核心方法（内化要点）

### 1.1 四阶段流水线架构
OpenNovelty是一个LLM驱动的学术新颖性评估系统，其核心设计哲学是**透明、证据驱动、可验证**。

**Phase I — 信息提取**：
- 从论文中提取核心任务（Core Task）和1-3个贡献声明（Contribution Claims）
- 为每个任务/贡献生成3个检索查询变体（Query Variants），遵循信息检索的查询扩展最佳实践
- 关键输出：`core_task`（一句话描述）+ `contributions`（结构化贡献列表）

**Phase II — 文献检索**：
- 语义检索相关论文（Core Task Top-50, Contribution Top-10）
- 质量过滤（精确匹配、时间过滤）+ 去重（canonical_id + 标题规范化）
- 构建引用索引（citation_index）

**Phase III — 深度分析**：
- 构建分层分类体系（Taxonomy）：将Top-50论文组织成层次化的子领域树
- 合成综述（Survey）：生成叙述性文本描述分类体系
- 文本相似度检测：token级模糊匹配（≥30连续词）
- 全文比较验证：**Refutation Assessment**（核心！）
- 贡献分析（Contribution Analysis）

**Phase IV — 报告生成**：
- 模板化渲染Markdown/PDF报告

### 1.2 Refutation Assessment（反驳评估）——最核心的方法

这是OpenNovelty最关键的创新评估方法。对于每个贡献声明，系统判定候选论文是否能"反驳"
原始论文的新颖性声明：

- **can_refute**：候选论文证明类似的先驱工作已存在
  - 需提供详细证据：3-5句总结 + 证据对（原文引用 vs 候选论文引用）
  - 每个引用必须是原文逐字摘录（≤90词），并标注段落位置

- **cannot_refute**：候选论文不能挑战新颖性
  - 仅需简要说明技术差异（1-2句）

- **unclear**：因缺乏细节难以比较

**关键约束**：
- 不得通过聚焦次要差异来制造人为反驳
- 证据引用必须来自全文上下文，而非贡献描述
- 每个引用都必须能在原文中逐字找到

### 1.3 分层分类体系（Taxonomy）

- 将核心任务相关的Top-50论文组织成层次化的子领域树
- 确定原始论文在分类体系中的位置（taxonomy_path）
- 识别"兄弟论文"（同一叶节点下的论文）进行详细比较
- 如果没有兄弟论文，进行子主题级别的比较
- 如果在分类体系中完全孤立，记录结构性隔离——这可能是高创新性的信号

### 1.4 对比分析策略

- **兄弟论文比较**：对同一分类叶节点下的论文进行详细区分
- **子主题回退**：当无兄弟论文但有兄弟子主题时，比较子主题
- **结构隔离**：当既无兄弟论文也无兄弟子主题时，表明论文在领域中具有独特位置

---

## 二、第一性原理检验框架

### 2.1 信息流动检验
- 核心问题：信息在模型/系统中的流动是否有瓶颈？
- 参考：ResNet通过shortcut connection为梯度创建高速公路
- 检验方法：画出信息流图，标注瓶颈点，问"有没有更直接的路径？"

### 2.2 注意力分配检验
- 核心问题：系统是否在浪费计算资源关注不重要的输入？
- 参考：Attention机制的核心是"只看重要的"
- 检验方法：分析输入的重要性分布，问"能否让模型学会忽略不重要的？"

### 2.3 监督信号检验
- 核心问题：是否充分利用了数据内部蕴含的监督信号？
- 参考：BERT的MLM利用上下文作为自监督信号
- 检验方法：审视数据的时间序列、上下文、共现关系，问"这些结构本身就是label吗？"

### 2.4 降维检验
- 核心问题：复杂问题能否被转化为更简单的等价问题？
- 参考：Word2Vec将V分类问题转化为K个二分类问题
- 检验方法：问"这个问题的本质是什么？有没有更简单的等价表述？"

### 2.5 鲁棒性检验
- 核心问题：系统是否过度依赖某个特定组件？
- 参考：Dropout强迫网络不依赖任何单一神经元
- 检验方法：问"如果去掉某个组件，系统还能工作吗？"

### 2.6 步骤化检验
- 核心问题：直接输出答案的过程中是否丢失了中间推理？
- 参考：CoT通过生成中间步骤为最终答案积攒计算量
- 检验方法：问"有没有中间步骤可以显式化？"

---

## 三、创新反模式识别

### 3.1 "创新性通胀"
- 表现：仅换了数据集/任务就声称创新
- 检验：核心方法是否有本质变化？如果在另一个数据集上也能直接运行，则非创新

### 3.2 "组合即创新"
- 表现：将A方法和B方法简单组合
- 检验：组合是否产生了1+1>2的效果？是否有理论解释为何组合有效？

### 3.3 "术语包装"
- 表现：用新术语/名称包装旧思想
- 检验：去掉新术语后，方法是否与已有工作本质相同？

### 3.4 "基准刷分"
- 表现：在特定benchmark上通过超参搜索达到SOTA
- 检验：性能提升是否来自真正的方法创新？能否在多个benchmark上一致提升？

### 3.5 "框架堆砌"
- 表现：设计了复杂的多模块框架，但每个模块都是标准组件
- 检验：去掉任何一个模块后效果如何？真正起作用的是哪个组件？

### 3.6 "问题构造"
- 表现：构造了一个没人关心的问题，然后完美解决它
- 检验：这个问题在实际场景中真的存在吗？有人遇到过这个问题吗？

---

## 四、跨学科创新的评估要点

### 4.1 概念桥梁
- 是否在两个或多个学科之间建立了新的概念联系？
- 参考：语义熵在语言学、信息论和深度学习之间搭建桥梁

### 4.2 方法迁移
- 是否将一个领域的成熟方法创造性地应用到另一个领域？
- 参考：对比学习从NLP到CV的迁移（SimCLR, MoCo）

### 4.3 视角转换
- 是否从全新的学科视角审视一个老问题？
- 参考：从物理学的能量景观视角理解神经网络优化

---

## 五、基于文献的创新性量化指标

### 5.1 文献空白度
- 使用OpenNovelty的refutation结果：`cannot_refute`的比例越高，创新性越强
- 分类体系中的隔离度：在taxonomy中越孤立，可能越创新

### 5.2 概念新颖度
- 是否引入了新术语/概念？（需区分"术语包装"和真正的概念创新）
- 新概念是否有清晰的定义和理论基础？

### 5.3 方法差异度
- 与最相近的先驱工作在方法层面的本质差异有多大？
- 差异是在"做什么"层面（问题选择）还是"怎么做"层面（方法设计）？

### 5.4 影响力预估
- 方法的通用性：能应用到多少不同的场景？
- 方法的简洁性：越简洁的方法越容易被广泛采用
- 方法的可解释性：越好理解的方法越容易获得学术认可

---

## 六、什么是好问题？什么是好创新？

### 6.1 好问题的特征
1. **重要性**：解决它会对领域产生实质影响
2. **困难性**：不是trivial的问题，需要非平凡的洞察
3. **可定义性**：问题边界清晰，可以明确评估是否解决
4. **时机性**：现在解决它的条件已经成熟（数据、算力、理论基础）
5. **连接性**：解决它会打开新的问题空间

### 6.2 好创新的特征
1. **贴近本质**：直击问题的根本，而非表面
2. **简洁优雅**：方法简单到让人惊讶它为什么有效
3. **解决盲区**：关注到别人忽视的角度
4. **推动进步**：不仅解决当前问题，还为后续研究铺路
5. **违反直觉但正确**：挑战常识但经得起验证（如Dropout的"随机拔网线"）
6. **可组合性**：核心思想可以与其他技术自由组合

### 6.3 创新的层次
1. **范式创新**（最高）：改变整个领域的思维方式（如Attention机制）
2. **方法论创新**：提出全新的解决路径（如ResNet的残差连接）
3. **视角创新**：从新角度审视老问题（如语义熵）
4. **技术创新**：在现有框架内引入新技术（如LoRA）
5. **应用创新**：将已有方法创造性地应用到新场景
6. **增量创新**（最低）：对现有方法的改进和优化

---

## 七、学术大佬的创新性判断智慧

### 7.1 Richard Hamming — "You and Your Research"（1986, Bell Labs）

Hamming在Bell Labs与Feynman、Shannon、Fermi等巨擘共事40年后，总结出以下关于伟大研究的洞察：

**选择重要的问题**：
- "如果你不在重要的问题上工作，你不太可能做出重要的工作。"
- 重要问题不仅意味着结果的影响力大，还必须有"合理的进攻方式"（reasonable attack）
- 伟大的科学家会维持一个10-20个重要问题的清单，一旦发现新技术可能适用于某个问题，立刻转向进攻

**重要问题的定义**：
- 不仅看解决方案的影响力，还看你是否有可行的进攻路径
- Bell Labs没有人研究时间旅行或反重力——不是因为它们不重要，而是因为没有进攻方式
- 好的问题 = 重要性 × 可进攻性

**突破常常来自局外人**：
- 考古学中的碳定年法来自物理学
- 第一架飞机由自行车专家制造
- 跨学科的迁移往往产生最大突破

**简洁的力量**：
- Hamming强调把孤立问题转化为一般性问题（Transform isolated problems into general problems）
- 这与ResNet、Attention等案例高度一致——解决一个具体问题时，找到的方法具有普遍价值

**评估启示**：
- 评估idea时，问两个问题：(1) 这个问题重要吗？(2) 你有合理的进攻方式吗？
- 如果两者都满足，即使idea看起来简单，也可能是伟大的工作

### 7.2 Michael Black — "Novelty in Science: A Guide to Reviewers"

Michael Black（Max Planck Institute，CVPR领域顶级学者）对创新性评估有深刻洞察：

**简单≠缺乏创新**：
- "简单的idea经常被误认为缺乏创新性，但恰恰相反——简单往往就是创新"
- 审稿人常见错误评价："这个idea太简单了，它只是改了loss函数中的一项"
- 但如果之前没有人想到改这一项，那它就是创新的——洞察力在于意识到一个小改变能产生大效果
- "我重视简洁超过不必要的复杂性——越简单越好"

**用"美"替代"创新性"**：
- Black建议用"beauty"（美）取代"novelty"作为评审标准
- 因为"美"剥离了"技术复杂性"和"难度"的暗示，更接近科学创新的本质
- 一幅毕加索的简笔画可以和伦勃朗的繁复画作一样美——论文也一样

**创新性的多种形式**：
- 新数据集可以是创新的（即使方法都是已有的）
- 旧方法的新用法可以是创新的（如果没人想到这么用）
- 用简单算法替代复杂算法也是创新（因为提供了洞察）
- "创新性以美展现自己的方式一样多"

**"显而易见"的陷阱**：
- 好的idea在被提出后看起来"显而易见"——这恰恰是好idea的标志
- "惊讶是一种短暂的情感。听到一个好idea，先有惊讶，然后越好的idea越显得理所当然"
- 评估创新性必须在idea存在之前的状态下评估，而非事后

**"简单"与"平凡"的区分**：
- 简单（simple）≠ 平凡（trivial）
- 平凡意味着不重要
- 如果一个简单idea效果比SOTA好，它很可能不是平凡的——作者发现了什么
- 真正应该警惕的是"不重要"而非"不复杂"

### 7.3 NeurIPS 2025 审稿指南的核心评价维度

NeurIPS 2025年的官方审稿指南强调以下关键判断准则：

**Significance（重要性）**：
- 研究结果对社区是否有影响？
- 研究者或实践者是否会使用这些idea或在其基础上构建？
- 是否以更好的方式解决了一个困难任务？
- 是否以可证明的方式推进了知识/理解？

**Originality（原创性）**：
- 是否提供了新的洞察、加深了理解、或突出了现有方法的重要属性？
- 原创性不一定要引入全新方法——评估现有方法的新洞察也同样有价值
- 提升效率、公平性等的工作也具有同等价值

**关键引用**：NeurIPS指出"Solid, technical papers that explore new territory or point out 
new directions for research are preferable to papers that advance the state of the art, 
but only incrementally."——探索新领域或指出新方向的工作优于增量提升SOTA的工作。

### 7.4 ICML 审稿教程的创新性评估示例

ICML 2023审稿教程给出了一个优秀创新性评价的范例：

**差的评价**："This paper proposes a new model for doing X, and it beats the state of the art."
（不够具体：到底什么是新颖的？什么是之前没有做过的？）

**好的评价**："This paper is the first to connect the theory of spectral semi-snippets with 
graph representation learning. This theory allows to compactly represent motifs... This idea 
is to my knowledge completely novel in the graph representation literature and may inspire 
further such models."
（具体指出了创新点在哪里、为什么新颖、以及潜在影响）

**评估启示**：好的创新性判断必须具体回答"之前没有什么"和"现在有什么"

### 7.5 ICLR 审稿中的创新性被拒原因分析

根据对ICLR 2022-2025审稿数据的研究：

**最常见的创新性缺陷**：
1. 与先前工作重叠（overlap with prior work）
2. 缺乏原创性（lack of originality）
3. 缺乏清晰的贡献（lack of clear contribution）

**影响接受率的关键因素**：
- 论文评分与"贡献"维度相关性最强，其次是"soundness"
- 高分论文的共同特征：被认可的创新性（novelty）和优雅的方法论（elegant and efficient models）
- 低分论文的共同缺陷：写作不清（unclear wording）和实验缺陷（weak baselines）

**"探索新领域"的论文获得更高分**：
- 技术创新性在论文包含较少表格和图形时评分更高（简洁的视觉设计凸显idea的原创性）
- 实证贡献在包含更多表格和图形时评分更高
- 启示：如果你的核心贡献是idea本身，保持呈现方式的简洁

### 7.6 Yann LeCun 对ResNet的评价

LeCun在评价何恺明的ResNet时指出：
- "The concept is simple and used in every deep learning system today (including transformers)"
- 核心是将每层从 y=f(x,w) 改为 y=x+f(x,w)
- ResNet论文已成为全科学领域引用最多的论文（253,000+ 引用）
- LeCun强调这是"open research"的力量——简单的idea + 开放的分享 = 最大影响

### 7.7 知乎SIGGRAPH资深研究者的经验

来自图形学领域的实战经验：

**Hack vs Innovation**：
- Hack = 用了某个方法取得不错效果，但讲不出道理
- Innovation = 方法背后有可解释的原理
- "生搬硬套的Hack是站不住脚的"——数学上能自洽但和物理机制无关不够

**idea的新颖程度直接代表问题理解的深度**：
- "你想到的idea的新颖程度就直接代表着你对这个问题理解的深入程度"
- 深入理解问题 → 回溯到最根本的idea → 在根节点处创新

**真正的挑战检验**：
- "如果一个idea里面没有任何挑战，那这个idea的创新性将大打折扣"
- 没有挑战的idea很容易被别人想到
- 挑战应该是真实存在的，不是人为编造的

---

## 八、顶会评分标准速查

### NeurIPS 评分系统
- **10**: Top 5% accepted papers. Truly groundbreaking work
- **9**: Top 15%. Excellent submission; strong accept
- **8**: Top 50%. Very good submission; clear accept
- **6**: Marginally above acceptance threshold
- **5**: Marginally below acceptance threshold
- **4**: Okay submission, but not good enough; reject

### 创新性评估的黄金标准（综合各顶会）
1. 是否探索了新领域或指出了新方向？（优于增量SOTA）
2. 之前没有什么？现在有什么？（具体差异）
3. 简单但有效的idea是否被误判为"不够创新"？
4. 创新性是否在idea提出之前的状态下评估？
5. 方法背后是否有可解释的原理（非Hack）？
6. 问题是否重要且有合理的进攻方式？（Hamming标准）
7. idea是否具有"美"——简洁、优雅、深刻？（Black标准）

---

## 九、2025-2026 最新 AI 同行评审系统原理集萃

本节整合本 skill 借鉴的 7 个最新 AI peer-review 系统的核心原理，既是设计出处的交代，也是评估者的知识背景。

### 9.1 Stanford Agentic Reviewer（Yixing Jiang + Andrew Ng, paperreview.ai）

**架构 Pipeline**：PDF → Markdown (LandingAI Agentic Document Extraction) → 多 specificity 查询生成 → Tavily API 检索 arXiv → 相关工作摘要 → 综合 review。

**7 维 Rubric**：Originality、Importance of research question、Claims well supported、Soundness of experiments、Clarity of writing、Value to research community、Contextualization relative to prior work。

**融合**：线性回归模型映射 7 维到最终 score。

**性能**（ICLR 2025，300 篇投稿，150 训/147 测）：Spearman ρ = 0.42（vs human-human 0.41，接近人类一致性水平）；AUC = 0.75（vs human 0.84）。

**关键局限**：Jiang 和 Ng 明确指出 **"LLMs are markedly less likely to comment on novelty in the way humans do"**——AI 在其他维度接近人类，**唯独 novelty 是显著短板**。这是本 skill 存在的根本理由：专门针对 AI 最薄弱的 novelty 维度做深度优化。

**对本 skill 的启发**：Phase 1.0 的 "Multi-Specificity Query Expansion" 和 "Refutation Chain" 直接借鉴 Stanford 的查询策略。

### 9.2 DeepReview（arxiv 2503.08569, DeepReviewer-14B）

**架构**：3 阶段 z₁ → z₂ → z₃。
- **z₁ Novelty Verification**：问题生成 + 论文分析 + 文献检索（Semantic Scholar/OpenScholar 检索 ~60 篇，rerank Top 10）
- **z₂ Multi-dim Review**：将作者 rebuttal 转化为可执行建议，同时综合多份 review
- **z₃ Reliability Verification**：四阶段验证链（方法论 + 实验 + 综合分析），每条评论附 confidence level

**基础模型**：Phi-4 14B + LongRoPE 扩展到 256K 上下文。

**数据**：DeepReview-13K（13,378 samples from ICLR 2024-2025, 平均 10,178 tokens/paper, 33.24% acceptance rate）。

**性能**：vs GPT-o1 Rating MSE ↓ **65.83%**；vs AgentReview (GPT-4o) 在 constructive value 上 **99.02% win rate**，analytical depth **99.01% win rate**。

**局限**：Synthetic data 可能偏离真实人类评审的细腻差别；adversarial robustness 未完全免疫。

**对本 skill 的启发**：Phase 1.0 Pipeline 的五步结构（查询 → 检索 → Rerank → Refutation → 自检）**直接对应 DeepReview 的 z₁ 范式**。Phase 2.5 多视角自检类似 z₃ 的可靠性验证思路。

### 9.3 AgentReview（EMNLP 2024, Ahren09）

**设计**：5 阶段 × 3 角色（Reviewer / Author / AC）× 基于 chatarena 框架，32% 接受率固定以匹配 ICLR 实际。

**数据**：ICLR 2020-2023 真实 reviews。

**核心发现**：**37.1% 决策变化来自 reviewer bias**——social influence theory（reviewer 间互相影响）、altruism fatigue（审稿疲劳）、authority bias（对知名作者/机构的偏好）。

**对本 skill 的启发**：Phase 2.5 Optimist / Pessimist / Meta-Reviewer 三角色模拟**直接借鉴此设计**，目的是降低单视角盲区。

### 9.4 debashis1983/agentic-paper-review（最高人类一致性的开源系统）

**架构**：9-Node LangGraph workflow with **Plan-Execute-Reflect** pattern。

**数据**：46,748 ICLR 2025 reviews from OpenReview。

**ML-Calibrated Scoring**（最关键的实证发现）：
```
Rating = -0.3057 + 0.7134×Soundness + 0.4242×Presentation + 1.0588×Contribution
```
权重占比：**Contribution 48.2% / Soundness 32.5% / Presentation 19.3%**。

**性能**：Spearman ρ = **0.7382**（显著高于 Stanford 的 0.42，是目前开源系统最高）。

**核心洞察**：**"Humans reward bold ideas 2.5x more than clear writing"**（1.0588 / 0.4242 ≈ 2.5）。这个数据驱动的发现颠覆了"写得清楚最重要"的直觉——真正决定接受的是**贡献的大胆程度**。

**对本 skill 的启发**：
1. B0.5 反偏见检查中的 "Bold-Idea 保守偏见"**直接引用此研究**——提醒评估者不要用 feasibility 顾虑拉低 novelty 评分
2. 验证了本 skill 把"方法论创新"设为最高权重（25%）的合理性

### 9.5 NovBench（arxiv 2604.11543, 2026 最新专门 novelty benchmark）

**设计**：第一个专门评估 LLM novelty assessment 能力的 benchmark，1,684 paper-review pairs from **COLING 2020 + EMNLP 2023**。

**4 维元评估 Rubric**（评估"评估"的框架）：
- **Relevance (1-5)**：语义对齐度（Information Matching Score）
- **Correctness (0-1)**：情感分布（positive / neutral / negative）vs 人类分布的接近度
- **Coverage (0-1)**：是否覆盖人类评审者识别的关键点
- **Clarity (0-1)**：关键词覆盖 + 句长 + 语言模型流畅度

**测试结论**：
| 策略 | 最适合 | 最损伤 |
|------|--------|--------|
| Zero-shot | Relevance (max 3.70/5) | — |
| Few-shot | Coverage, Correctness | Relevance |
| RAG | Clarity | Relevance（retrieval 可能误导） |

**识别的 5 类 LLM 系统性偏见**（本 skill B0.5 反偏见检查的来源）：
1. **Over-positive bias**：GPT-4o 倾向 positive，较少负面
2. **Missing multi-innovations**：只抓最显眼 1 个创新
3. **Sentiment skew**：specialist 模型反向过度负面（SEA-S）
4. **RAG misguidance**：被 retrieval 结果误导，高估相似性
5. **Instruction-following deficiency**：finetuned 模型常偏离指令

**对本 skill 的启发**：B0.5 三条反偏见检查（Over-Positive、Missing Multi-Innovations、Bold 保守）直接基于此研究。

### 9.6 OpenReviewer（maxidl, Llama-OpenReviewer-8B, arxiv 2412.11948）

**训练**：在 79,000 expert reviews from top ML conferences 上 fine-tune。

**关键观察**：OpenReviewer 比 GPT-4/Claude-3.5 **considerably more critical and realistic**——暗示通用 LLM 有**系统性 over-positive 倾向**，需要专门训练或反偏见机制来校正。

**对本 skill 的启发**：强化了 B0.5 的第一条（Over-Positive 反偏见）的必要性。

### 9.7 Russ Poldrack's ai-peer-review（Stanford 神经科学）

**设计**：三阶段 —— individual reviews（多模型并行） → aggregation → **blind meta-review**（隐去 model ID 后再综合，防 authority bias）。

**支持模型**：GPT-4o/Claude 3.7 Sonnet/Gemini 2.5 Pro/DeepSeek R1/Llama 4 Maverick。

**对本 skill 的启发**：Phase 2.5 的 Meta-Reviewer 采用 **blind synthesis** 机制——不参照前两轮角色标签。

### 9.8 OpenAIReview（openaireview.org）

**设计哲学**：区分两种 review 目的 —— **Quality review**（帮助作者改进，inspiration 来自 Refine）vs **Gatekeeping review**（决定接受/拒绝）。OpenAIReview 明确定位为 quality review。

**对本 skill 的启发**：本 skill 应明确定位为 **quality review 工具**——Phase 3"创新性增强建议"是核心输出，而非接受/拒绝决策。

### 9.9 本 skill 的差异化优势（相对于纯 AI 系统）

| 对比维度 | 纯 AI 系统（如 Stanford、DeepReview） | 本 novelty-assessment skill |
|---------|---------------------|-------|
| 人在回路 | 单次输出，无中间干预 | **4 个检查点**（A/A2/A3/B0/B0.5）强制用户确认 |
| 可解释性 | 7 维线性回归黑箱 | 每维评分附理由 + refutation 判定 + **锚点对标** |
| 反偏见主动性 | 被动（由 fine-tune 解决） | **B0.5 主动反制** NovBench 5 类偏见 |
| 方法论深度 | 主要依赖 rubric 模板 | 集成 Hamming / Black / Karpathy 等顶级研究者判断智慧 |
| 多视角模拟 | 有的有，有的无 | **Phase 2.5** 三角色 blind synthesis |
| 锚点校准 | 几乎都无 | 经典案例表（Dropout=9.5, ResNet=9.5, Attention=10, LoRA=8.5 等）提供**可复现的评分基准** |
| Novelty 专门化 | 通用 review，novelty 仅 1 维 | **全 skill 聚焦 novelty**，6 维度均围绕 novelty 展开 |

### 9.10 性能参考（与上述系统对标）

| 系统 | Spearman ρ (vs human) | 数据 | 策略 |
|------|-----------------------|------|------|
| Human-Human (ICLR 2025) | 0.41 | — | 基准 |
| Stanford Agentic Reviewer | **0.42** | 147 ICLR 2025 test | 7 维线性回归融合 |
| debashis1983 | **0.7382** | 46,748 ICLR 2025 训练 | ML-calibrated 3 维回归 |
| DeepReview-14B | — (MSE ↓65.83% vs GPT-o1) | DeepReview-Bench 1,286 | 3 阶段 z₁→z₂→z₃ |

**本 skill 的目标定位**：不是追求人机一致性分数（我们没法 fine-tune），而是提供**更透明、更可解释、更少偏见**的 novelty 评估流程，**补足纯 AI 系统最弱的 novelty 评估能力**。
