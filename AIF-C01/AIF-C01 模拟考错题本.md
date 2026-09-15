# AIF-C01 模拟考错题本

> 记录模拟考试中的错题，按**批次**（每次提交给我复习的一组题目）分类，同一批次内再按 Domain 分组。**注意：题号在不同批次间会重复**（如两个批次都出现过"Q2""Q46"，但内容完全不同），复习时务必以"批次 + 题号"共同定位，不要只记题号。每题格式：**题干概要 → 我的选择（错）→ 正确答案 → 核心原因 → 关联笔记**。
>
> 相关文档：[[AIF-C01 考试概览]]

## 目录

- [[#错题分布统计]]
- [[#重复出现的错题（高危薄弱点，优先复习）]]
- [[#批次一（12 题）]]
- [[#批次二（15 题）]]
- [[#复习优先级建议]]

---

## 错题分布统计

### 批次一（共 12 题）

| 题号 | Domain（原始标注） | 考点主题 | 状态 |
|------|-------------------|---------|------|
| Q2 | Fundamentals of AI and ML | 监督 vs 无监督 vs 半监督学习识别 | |
| Q18 | Fundamentals of AI and ML | 提升模型准确率：Epoch vs 学习率 vs 批大小 vs 正则化 | |
| Q26 | Fundamentals of AI and ML | Shapley 值 vs PDP（局部 vs 全局解释） | 📘 已补充 |
| Q30 | Fundamentals of AI and ML | 推理参数：Top K vs Top P vs Temperature vs Stop Sequences | |
| Q32 | Applications of Foundation Models | SageMaker Ground Truth vs Feature Store vs Canvas vs JumpStart | |
| Q35 | Fundamentals of Generative AI | 嵌入模型：BERT vs Word2Vec vs PCA vs SVD | 📘 已补充 |
| Q38 | Security, Compliance, and Governance（疑似错标，实际属 Domain 3 模型评估） | 翻译质量评估指标：BLEU vs BERTScore vs ROUGE vs Accuracy | 📘 已补充 |
| Q39 | Fundamentals of AI and ML（疑似错标，实际属 Domain 2 提示工程） | 提示四要素 vs 超参数/参数 | 📘 已补充 |
| Q46 | Applications of Foundation Models | AI 托管服务职责区分：Comprehend / Polly / Transcribe / Rekognition | |
| Q47 | Applications of Foundation Models | Bedrock 微调后模型的部署方式：Provisioned Throughput vs On-Demand | |
| Q50 | Fundamentals of AI and ML | 模型选型的第一步：狭窄定义用例 | |
| Q62 | Security, Compliance, and Governance（疑似错标，实际属 Domain 2 提示工程） | 少样本示例的输入-输出配对：意图识别场景 | 📘 已补充 |

### 批次二（共 15 题）

| 题号 | Domain（原始标注） | 考点主题 | 状态 |
|------|-------------------|---------|------|
| Q2 | Fundamentals of AI and ML | ML 实施的首要挑战：数据质量 vs 算力 vs 算法 vs 应用场景 | 📘 已补充 |
| Q5 | Security, Compliance, and Governance（疑似错标，实际属 Domain 2 提示工程） | 对齐聊天机器人语气/风格：迭代调整提示词 vs 调 Temperature | 📘 已补充 |
| Q7 | Fundamentals of AI and ML | 偏差-方差权衡：方向与定义不能搞反 | 📘 已补充 |
| Q10 | Fundamentals of Generative AI | 生成式模型的能力：生成新内容 vs 分类（判别式模型的活） | |
| Q12 | Applications of Foundation Models | 多模态嵌入模型 vs 多模态生成模型（🔁 与更早的独立提问重复出错） | 📘 之前已补充 |
| Q14 | Fundamentals of Generative AI | Amazon Q Developer 能力边界：能查/能问，不能部署/不能改/不能可视化 | 📘 已补充 |
| Q23 | Fundamentals of AI and ML | 推理参数：限制响应长度该用哪个参数（Response Length） | |
| Q28 | Fundamentals of AI and ML | 半监督学习示例：欺诈识别 vs 情感分析 vs 聚类 vs 降维 vs 神经网络 | 📘 已补充 |
| Q43 | Fundamentals of AI and ML | 确定性 vs 概率性 ML 模型（🔁 与更早的独立提问重复出错） | 📘 之前已补充 |
| Q46 | Guidelines for Responsible AI | 自动化安全评估该用哪个 AWS 服务：Amazon Inspector | 📘 已补充 |
| Q51 | Fundamentals of AI and ML | 结构化 vs 非结构化数据的特征工程差异 | 📘 已补充 |
| Q52 | Fundamentals of Generative AI | RAG 的最佳适用场景：客服/医疗问答 vs 原创内容/图像生成/推荐 | 📘 已补充 |
| Q57 | Applications of Foundation Models | 模型选型 + 有害内容防护：Model Evaluation + Guardrails | |
| Q58 | Fundamentals of Generative AI | 基础模型的训练范式：无标签数据 + 自监督学习 | |
| Q63 | Security, Compliance, and Governance（疑似错标，实际属通用云基础知识） | 云计算三项优势（选三）：CapEx→OpEx 方向易被反转 | |

> **关于"疑似错标"**：批次一 Q38/Q39/Q62、批次二 Q5/Q63 的题干内容明显分别属于"模型评估指标""提示词构成""少样本提示""提示工程""云计算基础"，均不是 Security, Compliance, and Governance 的典型考点——这大概率是题库本身的标签错误，复习时以**题目实际内容**为准，不必按此标签去对应的 Domain 5 笔记里找。

---

## 重复出现的错题（高危薄弱点，优先复习）

> 以下两道题**在不同时间被问到两次、两次都答错**（一次是更早的独立提问，另一次是批次二的模拟考），说明这两个知识点是**真正没有内化、而非一时疏忽**的薄弱区，考前应重点二次确认，优先级高于本笔记里只错过一次的题目。

| 题目 | 反复出错的核心症结 | 关联笔记 |
|------|-------------------|---------|
| **多模态嵌入模型 vs 多模态生成模型**（批次二 Q12） | 容易把"理解/检索多模态查询"（该用嵌入模型，成本低）和"生成全新多模态内容"（生成模型，更重更贵）这两个目标混为一谈，看到"多模态"就条件反射选生成模型 | [[Transformer与Embeddings#多模态嵌入模型 vs 多模态生成模型（考试重点）]] |
| **确定性 vs 概率性 ML 模型**（批次二 Q43） | 容易被"只能是……"这类过度绝对化的选项吸引，或误以为"监督学习=确定性、无监督学习=概率性"存在因果关系 | [[Machine Learning#确定性模型 vs 概率性模型（考试重点）]] |

---

## 批次一（12 题）

### Fundamentals of AI and ML（批次一）

#### 批次一 · Q2 - 监督学习 vs 无监督学习 vs 半监督学习识别（多选，选二）

- **题干**：以下哪些属于监督学习？（Document classification / Clustering / Linear regression / Neural network / Association rule learning）
- **我的选择（错）**：Document classification（错）+ Linear regression（对）
- **正确答案**：Linear regression + **Neural network**
- **核心原因**：
  - **Linear regression**、**Neural network** 都是用**带标签数据**训练、预测已知目标的监督学习算法
  - **Document classification（文档分类）** 属于**半监督学习**——当文档数量过大、无法全部人工标注时，先用少量标注数据 + 大量无标注数据训练，本质是监督+无监督的结合，不能简单归为监督学习
  - **Clustering（聚类）**、**Association rule learning（关联规则）** 都是**无监督学习**——前者按相似性分组，后者挖掘"经常同时出现"的规律（如 Apriori 购物篮分析），二者都不依赖标签
- **易错点**：这道题的陷阱在于"半监督学习"这个**第三选项**容易被误判成监督学习的一种（因为它确实用到了部分标签），但只要用到无标签数据做训练，就不是纯监督学习
- **关联笔记**：[[Machine Learning#监督学习算法]] | [[Machine Learning#无监督学习算法]] | [[Machine Learning#半监督学习 (Semi-Supervised Learning)（考试重点：勿与自监督混淆）]]（本知识点在批次二 Q28 再次考查，已补充完整的半监督学习定义与例题清单）

---

#### 批次一 · Q18 - 提升模型准确率：应该调整哪个超参数

- **题干**：模型部署后预测准确率不理想，应该怎么做来提升准确率？
- **我的选择（错）**：增加正则化（Increase regularization）
- **正确答案**：**增加 Epoch 数量**（让模型对训练数据训练更多轮）
- **核心原因**：
  - **Epoch 增多** → 模型有更多机会学习训练数据中的复杂模式和关系，直到准确率/误差率达到可接受水平——这是最直接针对"准确率不够"的调整方向
  - **正则化（我的错误选择）** 是用来**解决过拟合**的（惩罚模型复杂度，让模型更"收敛"更保守）；但题目描述的是"准确率不理想"，并未说明是过拟合还是欠拟合——如果模型本身**欠拟合**（还没学够），继续增加正则化反而会**进一步压低准确率**，属于用错方向的药
  - **降低批大小**：会让训练过程更"嘈杂"（每次用更少样本更新权重），可能有助于跳出局部最优，但不保证提升准确率，甚至可能减慢收敛、让训练更不稳定
  - **降低学习率**：让优化过程更平稳、避免"跨过"最优点，但过小会导致训练过慢、容易卡在局部最优，本质上是**微调阶段**才做的精细调整，而非直接的"准确率提升手段"
- **考试关键词识别**：题目若只说"准确率不够"没有说明"训练集好、测试集差"（过拟合特征）→ 默认先考虑"训练不充分"这个更基础的方向（增加 Epoch），而不是想当然地上正则化
- **关联笔记**：[[Machine Learning#模型参数与超参数]]（Epoch/学习率/批大小定义）| [[Machine Learning#过拟合与欠拟合（考试重点）]]（何时才该用正则化，含新补充的偏差-方差方向速查表）

---

#### 批次一 · Q26 - Shapley 值 vs PDP（局部 vs 全局解释）📘

- **我的选择（错）**：认为 Shapley 值是全局、PDP 是局部（把两者搞反了）
- **正确答案**：**Shapley 值 = 局部解释**（单个实例的特征贡献）；**PDP = 全局解释**（某特征在整个数据集范围内的边际影响）
- **核心原因**：Shapley 值基于博弈论，逐个实例计算每个特征的边际贡献，回答"为什么这一条预测是这个结果"；PDP 固定其他特征，扫描某个特征的取值范围，回答"这个特征整体上如何影响模型输出"
- **完整解析与记忆技巧已补充至**：[[负责任的AI与安全#Shapley 值 vs PDP（局部解释 vs 全局解释）]]

---

#### 批次一 · Q30 - 推理参数：限制候选词数量该调哪个参数

- **题干**：公司想调节模型为下一个词考虑的"最可能候选词数量"，该用哪个推理参数？
- **我的选择（错）**：Top P
- **正确答案**：**Top K**
- **核心原因**：
  - **Top K** 按**固定数量**限制候选词——只从概率最高的 **K 个**词元中采样，题目描述的"regulate the number of most-likely candidates"（数量）精确对应 Top K 的定义
  - **Top P（我的错误选择）** 按**累积概率百分比**动态决定候选集大小，题目若说"regulate the percentage/proportion"才对应 Top P——这是这道题的核心陷阱：**Top K 用"数量"表述，Top P 用"百分比/累积概率"表述**，题干用词决定了该选哪个
  - **Temperature**：调节整体概率分布的"随机性"，不是筛选候选词数量/范围的机制
  - **Stop sequences**：指定生成到某字符串就停止，与候选词数量无关
- **关联笔记**：[[提示工程 - Prompt Engineering#核心参数对比]]（表格中 Top K "通常几十到上百"、按"固定数量"筛选 vs Top P 按"累积概率"筛选的定义差异，是本题唯一考点）

---

#### 批次一 · Q50 - 确保选出最佳模型：研究团队该先做什么

- **题干**：为了确保为 AI 应用选出最佳模型，应该让研究团队做什么？
- **我的选择（错）**：识别潜在的数据源（Identify potential data sources）
- **正确答案**：**狭窄地定义应用的使用案例**（Define the use case of the application narrowly）
- **核心原因**：
  - **明确且具体的使用案例**能让研究团队清楚知道模型到底要完成什么任务，这是选型的**前提和基础**——没有清晰的目标，后续的数据、成本、架构选择都无从谈起
  - **识别数据源（我的错误选择）** 确实重要，但它是**支持数据收集**的准备步骤，并**不直接决定该选哪个模型**——是"知道要做什么"之后的**下一步**，而非第一步
  - **广泛定义目标受众**：范围定义越宽泛，需求越模糊，反而更难精准适配模型
  - **确定成本约束**：影响的是"有多少资源可用于训练"，而非"哪个模型架构/类型最适合这个任务"
- **考试关键词识别**：题目问"**确保选出最佳模型**"的**首要前提** → 优先考虑"任务/使用案例是否定义清楚"，而不是"数据/成本/受众"这类**支持性**但非决定性的因素
- **关联笔记**：[[Amazon SageMaker#项目生命周期]]（如有，可与"确定使用案例 → 实验 → 评估部署"的生命周期对照——"定义使用案例"永远是第一步）

---

### Fundamentals of Generative AI（批次一）

#### 批次一 · Q35 - 嵌入模型：区分一词多义应选哪个 📘

- **我的选择（错）**：Word2Vec
- **正确答案**：**BERT**（基于双向 Transformer 的动态/上下文嵌入）
- **核心原因**：Word2Vec/PCA/SVD 都产出**静态嵌入**（一个词永远对应同一个向量），无法区分上下文；BERT 同时看词的前后文，为同一个词在不同句子中生成不同向量
- **完整解析已补充至**：[[Transformer与Embeddings#静态嵌入 vs 上下文嵌入（考试重点：区分一词多义）]]

---

### Applications of Foundation Models（批次一）

#### 批次一 · Q32 - 构建高质量训练数据集应选哪个 SageMaker 服务

- **题干**：需要大规模、高质量、带标签的数据集来训练模型，该用哪个 SageMaker 服务？
- **我的选择（错）**：Amazon SageMaker Feature Store
- **正确答案**：**Amazon SageMaker Ground Truth**
- **核心原因**：
  - **Ground Truth** 是专门的**数据标注**服务，结合机器自动标注（主动学习）+ 人工标注（处理难例，可用 Mechanical Turk/私有/供应商劳动力）产出高质量标注数据集，直接对应题目"构建高质量、带标签数据集"的需求
  - **Feature Store（我的错误选择）** 是**特征存储库**——存的是已经处理好的**特征**（训练/推理阶段共享一致定义，避免训练服务偏差），而不是"给原始数据打标签"，两者所处的数据管道阶段完全不同（Feature Store 在标注**之后**，用于存储已构建好的特征）
  - **JumpStart**：提供预训练基础模型和快速微调，不涉及数据标注
  - **Canvas**：无代码建模界面，面向没有 ML 经验的用户直接建模，不是数据标注工具
- **考试关键词识别**："打标签 / labeled dataset / 标注" → 优先联想 **Ground Truth**；"存储/复用特征、训练推理一致性" → 才是 **Feature Store**
- **关联笔记**：[[Amazon SageMaker#SageMaker Ground Truth（数据标注，考试高频）]]（含与 Feature Store、Amazon A2I 的对比表格，务必重新过一遍这几个易混淆服务的分工）

---

#### 批次一 · Q46 - AI 托管服务职责区分（多选，选二）

- **题干**：以下哪些关于 Amazon ML 服务的表述正确？（Comprehend 语音转文字 / Polly 自然语音 / Comprehend 文本洞察 / Rekognition 提取关键短语 / Transcribe 构建对话界面）
- **我的选择（错）**：Amazon Polly（对）+ Amazon Transcribe 构建对话界面（错）
- **正确答案**：**Amazon Polly**（文字转自然语音）+ **Amazon Comprehend**（用 ML 从文本中挖掘洞察和关系）
- **核心原因（逐项纠错）**：
  - ❌ "Comprehend 做语音转文字" → 语音转文字是 **Amazon Transcribe** 的职责，Comprehend 是文本分析
  - ❌ "Transcribe 构建对话界面（语音+文字）" → 构建对话界面（聊天机器人）是 **Amazon Lex** 的职责，Transcribe 只负责语音转文字这一单一环节
  - ❌ "Rekognition 提取关键短语、按主题整理文本" → 这是**文本**处理任务，属于 Comprehend；Rekognition 专注**图像/视频**分析
  - ✅ "Polly 生成多语言自然语音" → 正确，Polly = 深度学习 TTS
  - ✅ "Comprehend 用 ML 挖掘文本中的洞察和关系" → 正确，这正是 Comprehend 的核心定位（NLP，无需 ML 经验）
- **易错点**：五个 AWS AI 服务（Comprehend / Polly / Transcribe / Rekognition / Lex）功能名字相近，考试常把"服务 A 的职责"错配给"服务 B"作为干扰项，必须精确记住每个服务的**输入输出模态**：Transcribe（语音→文字）、Polly（文字→语音）、Comprehend（文字→洞察/情感/实体）、Rekognition（图像/视频→识别结果）、Lex（语音+文字→对话交互）
- **关联笔记**：[[AWS AI托管服务]]（含五个服务的完整对比表和场景速查表，建议整体重新过一遍）

---

#### 批次一 · Q47 - Bedrock 微调后模型的部署方式

- **题干**：公司想用自己的带标签医疗文本数据微调 Bedrock 基础模型，应该用什么方式来测试和部署？
- **我的选择（错）**：Batch inference（批量推理）
- **正确答案**：**Provisioned Throughput**（预置吞吐量）
- **核心原因**：
  - 微调（Fine-tuning）产出的自定义模型，按 AWS 官方博客的表述，**测试和部署都必须购买 Provisioned Throughput**——这是为微调这种"计算密集、持续负载"场景设计的容量预留模式
  - **Batch inference（我的错误选择）** 是用来**对存储在 S3 中的大批量数据异步跑推理**、提高大数据集推理效率的功能，**不能用来完成微调过程本身**，是典型的干扰项
  - **Bedrock Playground**：只是一个用于**实验/测试提示词和推理参数**的控制台环境，不具备微调功能
  - **On-Demand**：按 Token 用量付费、无时间承诺，但**不支持**自定义模型的测试和部署（微调场景下需要 Provisioned Throughput）
- **⚠️ 与本笔记库现有内容的重要差异（复习时注意甄别）**：[[Amazon Bedrock]] 笔记中记录的是**更新、更细致**的规则——"自定义模型能否用 On-Demand 因模型提供商而异"（例如 **Anthropic** 的微调模型可直接用 On-Demand 推理，而 **Meta Llama 2** 等仅支持 Provisioned Throughput），并明确指出"自定义模型只能用 Provisioned Throughput"是**过度绝对化的错误答案**。本题的"Exam Alert"给出的是**更早期/更笼统**的规则（"测试部署自定义模型强制要求 Provisioned Throughput"）。**应试策略**：如果题目没有提及具体模型提供商、只是泛泛问"微调后应该用什么模式测试部署"，选 **Provisioned Throughput** 仍是当前题库的标准答案；但如果题目具体点名 Anthropic 等已知支持 On-Demand 的提供商，则需按 [[Amazon Bedrock]] 笔记中的更新规则判断。
- **关联笔记**：[[Amazon Bedrock#部署模式（考试重点）]]

---

### Security, Compliance, and Governance for AI Solutions（批次一，含疑似错标）

> 以下三题模拟考系统标注为 Domain 5，但题目实际内容分别属于 Domain 3（模型评估指标）和 Domain 2（提示工程），复习时按内容归类，不按系统标签归类。

#### 批次一 · Q38 - 翻译质量评估指标：BLEU vs BERTScore 📘

- **我的选择（错）**：BERT score
- **正确答案**：**BLEU**（机器翻译评估的行业标准指标）
- **核心原因**：BLEU 基于 n-gram 精确率匹配，是翻译任务的标准指标；BERTScore 虽然更"先进"（语义相似度），但不是翻译评估场景下的标准答案；ROUGE 面向摘要；Accuracy 不适用于生成任务
- **完整解析已补充至**：[[Model training and evaluation#NLP 评估指标（考试重点）]]（含"考试陷阱：BLEU vs BERTScore"callout）

---

#### 批次一 · Q39 - 提示的四要素 vs 超参数/参数 📘

- **我的选择（错）**：Instructions, Parameters, Input data, Output Indicator
- **正确答案**：**Instructions, Context, Input data, Output Indicator**
- **核心原因**：提示词的四要素（指令/上下文/输入数据/输出指示符）属于**推理阶段**的提示词内容；超参数（学习率等）和模型参数（权重）属于**训练阶段**，两个层面不能混用
- **完整解析已补充至**：[[提示工程 - Prompt Engineering#考试陷阱：提示的四要素 ≠ 超参数/模型参数]]

---

#### 批次一 · Q62 - 少样本示例的输入-输出配对：意图识别场景 📘

- **我的选择（错）**：用户输入 + 正确的模型回复（关注输入到回复的匹配）
- **正确答案**：**用户输入 + 正确的用户意图**
- **核心原因**：少样本示例的"输出"必须是任务的**目标产物**——意图识别任务的目标产物是**意图标签**，而不是"客服该怎么回复"（那是响应生成任务，是另一回事）
- **完整解析已补充至**：[[提示工程 - Prompt Engineering#考试陷阱：少样本示例中"输入-输出"到底是哪两样东西]]

---

## 批次二（15 题）

### Fundamentals of AI and ML（批次二）

#### 批次二 · Q2 - ML 实施的首要挑战 📘

- **题干**：ML 项目实施中，最好/最主要的挑战是什么？
- **我的选择（错）**：缺少可用的 ML 算法（Lack of available machine learning algorithms）
- **正确答案**：**收集和准备高质量训练数据的难度**
- **核心原因**：数据清洗、去噪、标注、验证是最耗时且最容易被低估的环节；算力、算法数量、应用场景广度都不是主要瓶颈——"Garbage in, garbage out"，数据质量决定模型质量上限
- **完整解析已补充至**：[[AI与ML概念#ML 实施的首要挑战（考试重点）]]

---

#### 批次二 · Q7 - 偏差-方差权衡：方向与定义不能搞反 📘

- **题干**：什么是机器学习中的偏差-方差权衡（bias-variance trade-off）？
- **我的选择（错）**：认为"高偏差导致过拟合，高方差导致欠拟合"（方向反了）
- **正确答案**：**偏差 = 模型假设过于简单/错误带来的误差 → 导致欠拟合**；**方差 = 模型复杂度/对训练数据过度敏感带来的误差 → 导致过拟合**
- **核心原因**：这道题的干扰项做了两层反转——一层是把"偏差/方差"和"欠拟合/过拟合"的对应方向调换，另一层是把"偏差=简单假设的误差、方差=复杂度的误差"这组**定义本身**对调，必须同时守住两条映射才能选对
- **完整解析已补充至**：[[Machine Learning#偏差-方差权衡（考试重点，勿与"过拟合/欠拟合"的方向搞反）]]

---

#### 批次二 · Q23 - 推理参数：限制响应长度该用哪个参数

- **题干**：公司想设置模型响应中返回的**词元数量上限**，该用哪个推理参数？
- **我的选择（错）**：Stop sequence
- **正确答案**：**Response length**（即 Max Tokens，Bedrock 推理参数 API 中的命名）
- **核心原因**：
  - **Response length** 直接定义"返回响应的最小/最大词元数"，精确对应"设置词元数量上限"的需求
  - **Stop sequence（我的错误选择）** 是指定**特定字符串**，模型生成到该字符串就停止——它依赖"匹配到某个标记"，而不是"数到第 N 个词元就停止"，两种"停止生成"的触发机制完全不同
  - **Top P / Top K**：都是筛选"下一个词元候选范围"的采样参数，与"生成多长"无关
- **考试关键词识别**：题目描述"设置返回词元数量的**上限/下限**" → **Response Length（Max Tokens）**；题目描述"生成到某个**特定字符串**就停止" → **Stop Sequences**——一个按"数量"控制，一个按"内容匹配"控制，不要混淆
- **关联笔记**：[[提示工程 - Prompt Engineering#核心参数对比]]（表格中已将 Max Tokens 与 Bedrock API 命名的 Response Length 标注为同一概念的两种叫法）

---

#### 批次二 · Q28 - 半监督学习示例（多选，选二）📘

- **题干**：以下哪些是半监督学习的例子？（Clustering / Fraud identification / Sentiment analysis / Dimensionality reduction / Neural network）
- **我的选择（错）**：Fraud identification（对）+ Neural network（错）
- **正确答案**：Fraud identification + **Sentiment analysis**
- **核心原因**：欺诈识别和情感分析都是"只有一小部分数据被专家标注，其余海量数据无标签"的典型半监督场景；Neural network 是复杂的**监督学习**技术；Clustering、Dimensionality reduction 都是**无监督学习**
- **完整解析（含半监督学习的完整定义、流程和三个高频例题）已补充至**：[[Machine Learning#半监督学习 (Semi-Supervised Learning)（考试重点：勿与自监督混淆）]]

---

#### 批次二 · Q43 - 确定性 vs 概率性 ML 模型 📘

- **我的选择（错）**：认为 ML 模型只能是概率性的（Machine Learning models can only be probabilistic）
- **正确答案**：**ML 模型可以是确定性、概率性，或两者混合**
- **核心原因**：决策树是确定性模型（同输入必同输出）；贝叶斯网络是概率性模型（给出概率分布）；神经网络、随机森林则是两者的混合体——"只能是……"这类绝对化表述都是错误答案
- **完整解析已补充至**：[[Machine Learning#确定性模型 vs 概率性模型（考试重点）]]

---

#### 批次二 · Q51 - 结构化 vs 非结构化数据的特征工程差异 📘

- **题干**：结构化数据和非结构化数据的特征工程任务，关键区别是什么？
- **我的选择（错）**：认为结构化数据不需要特征工程（已经是可用格式），只有非结构化数据才需要大量预处理
- **正确答案**：**结构化数据的特征工程常涉及归一化、处理缺失值；非结构化数据涉及分词、向量化**
- **核心原因**：两类数据都**需要**特征工程，只是处理手法不同——结构化数据（表格/数值）关注"数值是否干净统一"，非结构化数据（文本/图像/音频）关注"如何把内容转成数值向量"；"结构化数据不需要特征工程"本身就是错误前提
- **完整解析已补充至**：[[AI与ML概念#结构化 vs 非结构化数据的特征工程差异（考试重点）]]

---

### Fundamentals of Generative AI（批次二）

#### 批次二 · Q10 - 生成式模型的能力：生成新内容 vs 分类

- **题干**：生成模型分析动物图像（记录耳型、眼型、尾部特征、皮肤纹理等变量）后，能执行以下哪项任务？
- **我的选择（错）**：可以对多个物种（猫、狗等）做分类
- **正确答案**：**可以重新生成训练集中不存在的全新动物图像**
- **核心原因**：
  - 生成模型学习特征及其相互关系后，能**创造全新内容**——这正是"生成式"这个词的核心含义，题干描述的"记录变量、学习不同动物的普遍特征"本质是在为"生成新样本"做准备，而非为分类做准备
  - **分类（单类或多类，我的错误选择）** 是**判别式模型（Discriminative Model）**的能力——判别式模型学习"已知输入特征"与"未知类别标签"之间的映射关系（如像素排列→类别名称），而生成模型的目标从一开始就不是分类
  - **识别训练集中的任意图像**：生成模型不是图像匹配/检索算法，无法做到"识别这张图是不是训练集里见过的"
- **考试关键词识别**：题目强调"生成式 AI / Generative Model" + "学习特征后创造新内容" → 生成新样本；题目若强调"判断属于哪个类别/预测标签" → 那是判别式模型的领域，与生成模型无关
- **关联笔记**：[[大语言模型 - LLM]] 开篇的"生成式 AI vs 传统 AI"对比表（核心目标：分类/预测 vs 生成全新内容），是这道题唯一需要的知识点

---

#### 批次二 · Q14 - Amazon Q Developer 能力边界（多选，选二）📘

- **题干**：以下哪些代表 Amazon Q Developer 的能力？（部署云基础设施 / 修改资源做成本优化 / 理解和管理云基础设施 / 用自然语言回答成本问题 / 可视化成本数据）
- **我的选择（错）**：理解和管理云基础设施（对）+ 部署云基础设施（错）
- **正确答案**：理解和管理云基础设施 + **用自然语言回答 AWS 账户特定的成本问题**
- **核心原因**：Q Developer 的云资源能力被严格限定在"**查询/理解/问答**"层面（可以用自然语言列出资源、可以基于 Cost Explorer 数据回答成本问题），但**不能部署基础设施、不能自动修改资源做成本优化、也不能可视化成本数据**——可视化仍需去 AWS Cost Explorer 本身完成
- **完整解析已补充至**：[[Amazon Q Developer#云资源理解与成本问答（考试重点：能"问"不能"做"）]]

---

#### 批次二 · Q52 - RAG 的最佳适用场景（多选，选二）

- **题干**：以下哪些是在 Amazon Bedrock 中使用 RAG（检索增强生成）的最佳适用场景？（客服聊天机器人 / 匹配购物者偏好的商品推荐 / 根据文本生成图像 / 原创内容创作 / 医疗查询聊天机器人）
- **我的选择（错）**：Customer service chatbot（对）+ Original content creation（错）
- **正确答案**：Customer service chatbot + **Medical queries chatbot**
- **核心原因**：
  - **客服聊天机器人**、**医疗查询聊天机器人** 都需要基于企业自有的、会持续更新的私有知识（产品文档/医学文献）给出**有据可查、准确**的回答，正是 RAG 要解决的"模型不知道的私有/实时知识"问题
  - **原创内容创作（我的错误选择）**：目标是创造全新、有创意的内容，不需要"检索已有正确答案"，是纯生成任务，与 RAG 的检索定位相反
  - **根据文本生成图像**：是扩散模型等生成任务，与"检索文本知识库回答问题"无关
  - **匹配购物者偏好的商品推荐**：属于**推荐系统**的任务范畴（协同过滤/个性化排序），不是"检索知识回答问题"
- **关联笔记**：[[Amazon Bedrock#RAG 适用 vs 不适用场景（考试重点）]]（含完整的适用/不适用场景对照表）

---

#### 批次二 · Q58 - 基础模型的训练范式：无标签数据 + 自监督学习

- **题干**：关于生成式 AI 中的基础模型（Foundation Models），以下哪项表述正确？
- **我的选择（错）**：FMs use labeled training data sets for supervised learning
- **正确答案**：**FMs use unlabeled training data sets for self-supervised learning**
- **核心原因**：基础模型使用**自监督学习**——从海量**无标签**数据中自动构造隐式标签（如遮盖词元、预测下一个词元），完全不依赖人工标注的训练数据，这是"预训练"阶段的核心特征
- **关联笔记**：[[Machine Learning#自监督学习 (Self-Supervised Learning)]]（已明确标注"基础模型预训练阶段通常采用自监督学习"这一考试关联点，可与本题直接对照）

---

### Applications of Foundation Models（批次二）

#### 批次二 · Q12 - 多模态嵌入模型 vs 多模态生成模型 📘（🔁 重复出错，见上方"高危薄弱点"）

- **我的选择（错）**：多模态生成模型（Multi-modal generative model）
- **正确答案**：**多模态嵌入模型（Multi-modal embedding model）**
- **核心原因**：任务是"理解/解读"文本+图像混合查询，而非"生成全新内容"——嵌入模型（如 Amazon Titan Multimodal Embeddings）把不同模态数据对齐到同一向量空间，用相似度匹配即可完成理解，计算成本远低于生成模型；生成模型能力更强但更贵，用在"只需要理解"的场景上是过度设计
- **完整解析已补充至**：[[Transformer与Embeddings#多模态嵌入模型 vs 多模态生成模型（考试重点）]]

---

#### 批次二 · Q57 - 模型选型 + 有害内容防护（多选，选二）

- **题干**：团队需要为公开应用选择最合适的 LLM，同时担心生成有害/不当内容，应该用哪两个 AWS 方案？
- **我的选择（错）**：Model Evaluation on Amazon Bedrock（对）+ Amazon SageMaker Model Monitor（错）
- **正确答案**：Model Evaluation on Amazon Bedrock + **Guardrails for Amazon Bedrock**
- **核心原因**：
  - **Model Evaluation** 帮助在多个基础模型间横向比较，为特定用例选出效果最好的模型，直接对应"选型"这一诉求
  - **Guardrails（正确答案，我漏选）** 是专门为生成式 AI 应用实现内容安全防护的服务——支持跨多个基础模型统一应用护栏、标准化安全和隐私控制，直接对应"担心生成有害内容"这一诉求
  - **SageMaker Model Monitor（我的错误选择）** 用于监控**已上线模型在生产环境中的数据/预测质量漂移**，既不涉及"选型"也不涉及"内容审核"，是典型的功能名称相近但用途完全不同的干扰项
  - **Amazon Comprehend**、**SageMaker Clarify**：分别是文本洞察分析、偏见检测工具，都不直接服务于"模型选型"或"生成内容审核"这两个目标
- **考试关键词识别**："在多个基础模型中挑选最合适的" → **Model Evaluation**；"防止生成有害/不当内容" → **Guardrails**；两者经常在同一道题里成对出现，因为它们分别对应"上线前选型"和"上线后内容防护"这两个相邻但不同的阶段
- **关联笔记**：[[Amazon Bedrock#Bedrock Model Evaluation]] | [[Amazon Bedrock#Bedrock Guardrails（内容安全）]]

---

### Guidelines for Responsible AI（批次二）

#### 批次二 · Q46 - 自动化安全评估该用哪个 AWS 服务 📘

- **我的选择（错）**：AWS Audit Manager
- **正确答案**：**Amazon Inspector**
- **核心原因**：Amazon Inspector 是**自动化安全评估服务**，自动扫描已部署的应用（EC2/容器/Lambda）以发现**漏洞**和对安全最佳实践的偏离；AWS Config 评估的是**资源配置**是否合规，AWS Audit Manager 收集的是**审计证据**，AWS Artifact 提供的是 **AWS 自身的合规报告**——四者评估对象完全不同，不能因为都带有"安全/合规"字样就互相替代
- **完整解析（含四服务对比表）已补充至**：[[负责任的AI与安全#AWS 合规相关资源]]

---

### 疑似错标（批次二，实际属通用云基础知识）

#### 批次二 · Q63 - 云计算三项优势（多选，选三）

- **题干**：以下哪些是云计算相对于本地基础设施的优势？（提前规划容量 / 规模效益 / 几分钟内部署全球应用 / 将资本支出转为可变支出 / 花钱建维护数据中心 / 将可变支出转为资本支出）
- **我的选择**：Benefit from massive economies of scale（对）+ Go global in minutes（对）+ **Trade variable expense for capital expense**（错，选反了）
- **正确答案**：Benefit from massive economies of scale + Go global in minutes + **Trade capital expense for variable expense**
- **核心原因**：这道题的陷阱就是把"六大优势"之一的表述**主谓颠倒**放进选项——正确说法是"**将资本支出（CapEx）转为可变支出（OpEx）**"（Trade capital expense **for** variable expense），干扰项把方向倒过来变成"将可变支出转为资本支出"（Trade variable expense for capital expense），意思完全相反，读题时必须逐字确认"trade A for B"中 A、B 谁在前
- **易错点**：另外两个错误选项（"提前规划容量"、"花钱建维护数据中心"）都是云计算要**摆脱**的旧模式描述，容易被当成"优势"选中，但它们描述的其实是传统本地基础设施的**痛点**，不是云计算带来的好处
- **关联笔记**：[[AIF-C01 考试概览#云计算六大优势（跨认证通用基础考点）]]（已收录完整六大优势表格，可据此复查这道题的"CapEx/OpEx 方向反转"陷阱）

---

## 复习优先级建议

```
按易错模式归类复习
├── "反复出错，说明真没记住" → 批次二 Q12（多模态嵌入/生成）、Q43（确定性/概率性模型）★★★
├── "把局部/全局、静态/动态等对立概念搞反" → 批次一 Q26（Shapley/PDP）、Q35（BERT/Word2Vec）
├── "把两组定义或方向同时对调（双重陷阱）" → 批次二 Q7（偏差=简单假设/方差=复杂度，且分别对应欠拟合/过拟合）、Q63（CapEx→OpEx 方向反转）
├── "混淆同类但职责不同的 AWS 服务" → 批次一 Q32（Ground Truth/Feature Store）、Q46（Comprehend/Transcribe/Rekognition）、批次二 Q46（Inspector/Config/Audit Manager/Artifact）
├── "把训练阶段概念和推理阶段概念混为一谈" → 批次一 Q39（超参数 vs 提示要素）、Q18（Epoch/正则化选择方向错误）
├── "少样本/提示工程的输入输出到底指什么" → 批次一 Q62（意图识别配对）、Q30（Top K vs Top P 用词陷阱）、批次二 Q23（Response Length vs Stop Sequence）、Q5（语气风格该调提示词而非 Temperature）
├── "评估指标/服务选型：更先进/更像 ≠ 更标准" → 批次一 Q38（BLEU vs BERTScore）、批次二 Q57（Model Evaluation + Guardrails 的组合）
├── "生成式 vs 判别式、监督 vs 半监督的边界" → 批次二 Q10（生成 vs 分类）、Q28（半监督学习示例）、批次一 Q2（半监督识别）
├── "AI 助手/服务的能力边界：能查不能做" → 批次二 Q14（Q Developer 云资源/成本问答）
├── "数据/特征工程类基础题" → 批次二 Q2（ML 首要挑战=数据质量）、Q51（结构化/非结构化特征工程）
└── "解题第一步该做什么" → 批次一 Q50（先定义使用案例）
```
