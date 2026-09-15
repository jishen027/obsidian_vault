# AWS AIF-C01 - AI Practitioner 考试概览

> **AWS Certified AI Practitioner (AIF-C01)** 是 AWS 的 AI 基础认证，涵盖传统 ML 流程、AWS 托管 AI 服务、生成式 AI 与大语言模型的基础知识，适合 AI 工程师、数据科学家及希望将 AI 纳入业务决策的架构师。
>
> 相关文档：[[AI与ML概念]] | [[大语言模型 - LLM]] | [[Amazon Bedrock]] | [[Amazon Bedrock AgentCore]] | [[Amazon SageMaker]] | [[Amazon Q]] | [[负责任的AI与安全]]

---

## 考试结构

### 五大考试域

| 域 | 名称 | 权重 |
|----|------|------|
| **Domain 1** | AI 和 ML 基础 (Fundamentals of AI and ML) | 约 20% |
| **Domain 2** | 生成式 AI 基础 (Fundamentals of Generative AI) | 约 24% |
| **Domain 3** | 基础模型的应用 (Applications of Foundation Models) | 约 28% |
| **Domain 4** | 负责任 AI 准则 (Guidelines for Responsible AI) | 约 14% |
| **Domain 5** | AI 解决方案的安全合规治理 (Security, Compliance & Governance) | 约 14% |

> Domain 2 + Domain 3 均为生成式 AI，合计约 52%，是最重要的备考方向。

### 考试格式

| 项目 | 详情 |
|------|------|
| **总题数** | 65 题（50 计分 + 15 不计分） |
| **题型** | 单选、多选、排序、匹配、案例分析 |
| **时长** | 120 分钟（座位时间 150 分钟） |
| **通过分数** | 700/1000（约 70%，采用缩放评分） |
| **有效期** | 36 个月 |
| **考试平台** | Pearson VUE（线上/线下） |

---

## 学习路径

### 前置建议

```
建议学习顺序
├── AWS Cloud Practitioner → 了解 AWS 核心服务与共享责任模型
├── AIF-C01 → AI/ML 基础 + AWS AI 服务 + 生成式 AI
└── （可选）Machine Learning Specialty → 深入 ML 工程
```

### 学习时间估算

| 背景 | 建议时长 |
|------|---------|
| 完全初学者 | 约 20 小时 |
| 有云计算经验 | 约 10 小时 |
| 有 ML/AI 经验 | 约 5 小时 |

> **建议分配**：50% 讲座 + Labs，50% 模拟考题。每天 1-2 小时，约 14 天完成。

---

## 云计算六大优势（跨认证通用基础考点）

> 这是 AWS **所有**认证考试（从 Cloud Practitioner 到 AIF-C01、SAA-C03）中最基础且必考的标准知识点，属于[[#前置建议|前置的 Cloud Practitioner 知识]]，AIF-C01 虽不专门考察，但理解它有助于把握"为什么用云做 AI/ML"这一价值主张的底层逻辑。

| 优势 | 说明 |
|------|------|
| **将资本支出转为可变支出 (Trade capital expense for variable expense)** | 不再需要提前购买昂贵的服务器和数据中心，改为**用多少付多少**——用 OpEx（运营支出）替代 CapEx（资本支出） |
| **受益于巨大的规模效益 (Benefit from massive economies of scale)** | 云服务商通过海量客户的聚集效应，获得更低的采购成本，从而为用户提供**更低的按需购买价格** |
| **停止猜测容量 (Stop guessing capacity)** | 无需提前预估未来容量需求，告别**容量不足**（影响可用性）或**资源浪费**（多余成本）的两难，按需**弹性伸缩（Auto Scaling）** |
| **提高速度和敏捷性 (Increase speed and agility)** | 计算资源的获取和部署从原来的数周/数月缩短到**几分钟内**完成，加快创新和试错节奏 |
| **停止花费资金在数据中心的运行和维护上 (Stop spending money running and maintaining data centers)** | 摆脱布线、搬迁机架、散热管理等基础设施"重活"（**Undifferentiated Heavy Lifting**），把精力集中在真正创造业务价值的事情上 |
| **几分钟内部署到全球 (Go global in minutes)** | 利用云厂商全球分布的 **Region（区域）/ AZ（可用区）**，轻松以低成本实现低延迟的全球化部署 |

> **考试提示**：题目常以场景描述形式出现（如"公司希望避免提前采购硬件的大额投入" → 对应第一条 CapEx→OpEx；"公司希望几分钟内就能在海外为用户提供服务" → 对应"Go global in minutes"），要求判断该场景体现的是六大优势中的哪一条。**Undifferentiated Heavy Lifting（无差别的重活）** 这一说法在生成式 AI 场景下也常被复用——例如"使用 Amazon Bedrock 而非自建基础模型" 同样是把"训练/维护基础模型"这种重活交给 AWS。

---

## 各域考点速览

### Domain 1 - AI 和 ML 基础

- AI / ML / 深度学习 / 生成式 AI 的定义与区别
- 监督学习、无监督学习、强化学习
- 常见算法：回归、分类、聚类、降维
- 神经网络与深度学习基础
- NLP 基础概念
- ML 数据生命周期（CRISP-DM）
- 数据标注与人工审核：**SageMaker Ground Truth**（训练前，标注原始数据，机器自动标注+人工处理难例）vs **Amazon Augmented AI (A2I)**（推理后，对低置信度预测结果做人工复核）

相关笔记：[[AI与ML概念]] | [[Machine Learning]] | [[Deep learning and neural networks]] | [[Model training and evaluation]]

### Domain 2 - 生成式 AI 基础

- 生成式 AI 是深度学习子集，创建**全新原创内容**（非分类/预测）
- 模型输出机制：预测下一个词/Token（概率性过程）
- 参数量 → 规模 → 内存 → 能力（正向关系）；**涌现能力**：模型越大越无需 few-shot
- **预训练 = 自监督学习**：数据量 GB→PB，来源含互联网抓取；策管后仅 **1%~3%** 的分词可用
- 数学基础：概率建模、损失函数、矩阵乘法
- Transformer 架构与注意力机制（2017, "Attention Is All You Need"）
- Tokenization 与 Embeddings（向量、语义接近原则、Q/K/V 注意力）
- **单模态 vs 多模态**：LLM = 单模态；多模态 = 文本 + 图像 + 音频
- **扩散模型**：正向扩散 / 反向扩散 / 稳定扩散（隐空间）；优于 GAN/VAE
- 提示工程（Prompt Engineering）：零样本、**单样本**、少样本、思维链
- **上下文学习 (In-Context Learning)**：在提示词中加入示例，无需重新训练
- **补全 (Completion)**：模型输出；**推理 (Inference)**：提示词→模型→补全的过程

相关笔记：[[大语言模型 - LLM]] | [[Transformer与Embeddings]] | [[提示工程 - Prompt Engineering]]

### Domain 3 - 基础模型应用

- Amazon Bedrock：基础模型访问、Fine-tuning、RAG、Agents、Guardrails
- **Bedrock Agents Classic 于 2026-07-30 起停止向新客户开放（维护模式）**，AWS 力推的新一代智能体生产平台是 **[[Amazon Bedrock AgentCore]]**（框架无关+模型无关，Runtime/Gateway/Memory/Identity/Harness 等按需拼装组件）
- **[[Amazon Q]]**：AWS 生成式 AI 助手统一品牌，按角色拆分为 Q Developer（开发者）/ Q Business（业务知识问答）/ Q Apps（免代码小应用）/ Q in QuickSight（生成式 BI）/ Q in Connect（客服实时辅助）
- **迁移学习**：预训练模型 + 小数据集微调，节省时间和成本
- **项目生命周期**：确定使用案例 → 实验 → 适应调整（高度迭代）→ 评估部署 → 监控
- **基础模型生命周期**：数据选择 → 模型选择 → 预训练 → 微调 → 评估 → 部署 → 反馈
- **RLHF**：HHH 原则（有用/诚实/无害）；对齐人类偏好
- **SageMaker JumpStart**：预构建基础模型、快速微调和部署
- **PartyRock**：基于 Bedrock 的免费学习/原型体验平台
- 模型评估与选择；ROUGE（摘要）/ BLEU（翻译）

相关笔记：[[Amazon Bedrock]] | [[Amazon Bedrock AgentCore]] | [[Amazon SageMaker]] | [[Amazon Q]]

### Domain 4 - 负责任的 AI

- AI 偏见（Bias）与公平性（Fairness）
- 可解释性（Explainability）：内在分析（简单模型）vs 事后分析（复杂模型，本地/全局）
- 数据隐私与 PII 处理
- Guardrails 内容过滤
- **HHH 原则**：有用性 / 诚实性 / 无害性

### Domain 5 - 安全、合规、治理

- AI 安全威胁：**提示词注入** / **数据中毒** / **模型反演漏洞**（三类考试必记）
- **AWS Nitro System**：强制安全限制，保护 EC2 工作负载和数据
- **Trainium（训练）/ Inferentia（推理）**：专用 AI 芯片，高性价比
- IAM 权限控制；CloudWatch 日志监控
- 加密、MFA、持续监控合规框架
- **模型治理工具**：SageMaker **Model Cards**（标准化模型文档）/ **Model Registry**（版本管理+部署前审批工作流）/ **Role Manager**（基于角色人设的最小权限 IAM 策略）/ **Model Dashboard**（组织级已部署模型集中视图）
- **[[Amazon Macie]]**：自动发现 S3 中训练数据集/存储数据里的 PII 和敏感信息，与 Amazon GuardDuty（威胁检测）分工不同
- **[[AWS Artifact]]**：下载 AWS 自身的合规报告/证书（SOC、ISO 等）+ 管理 BAA/DPA 等协议，与持续收集**自有系统**证据的 [[AWS Audit Manager]] 分工不同
- **[[AWS Config]]**：持续评估和记录**我方资源配置**是否符合合规规则（如 S3 桶是否加密）
- **[[AWS Audit Manager]]**：持续自动收集**我方系统**的合规审计证据并生成评估报告，简化审计准备——三者判断依据是"评估对象是谁"：Config 管自己的配置，Artifact 管 AWS 的证书，Audit Manager 管自己的证据；**⚠️ Audit Manager 已于 2026-04-30 起进入维护模式，停止向新账户开放**，AWS 推荐新需求改用 Config 的 Conformance Packs

相关笔记：[[负责任的AI与安全]] | [[Amazon Macie]] | [[AWS Artifact]] | [[AWS Config]] | [[AWS Audit Manager]]

---

## 重要 AWS AI 服务汇总

| 服务                               | 类别          | 核心功能                                                                                        |
| -------------------------------- | ----------- | ------------------------------------------------------------------------------------------- |
| **[[Amazon Bedrock]]**           | 生成式 AI      | 访问基础模型（Claude、Titan、Llama 等）                                                                |
| **[[Amazon Bedrock AgentCore]]** | 生成式 AI 智能体  | 框架/模型无关的智能体生产基础设施（Runtime/Gateway/Memory/Identity/Harness），取代进入维护模式的 Bedrock Agents Classic |
| **[[Amazon SageMaker]]**         | ML 平台       | 完整 ML 训练、部署、监控流程                                                                            |
| **[[Amazon Rekognition]]**       | 计算机视觉       | 图像/视频中的对象、人脸、文字识别                                                                           |
| **[[Amazon Comprehend]]**        | NLP         | 情感分析、实体提取、PII 检测                                                                            |
| **[[Amazon Lex]]**               | 对话 AI       | 语音/文字聊天机器人（Alexa 商业版）                                                                       |
| **[[Amazon Kendra]]**            | 企业搜索        | 语义搜索引擎                                                                                      |
| **[[Amazon Personalize]]**       | 推荐系统        | 个性化推荐（用户-物品交互）                                                                              |
| **[[Amazon Polly]]**             | 文字转语音       | 多语言 TTS，支持 SSML                                                                             |
| **[[Amazon Textract]]**          | 文档处理        | OCR、表单提取、签名检测                                                                               |
| **[[Amazon Transcribe]]**        | 语音转文字       | 自动语音识别 (ASR)                                                                                |
| **[[Amazon Translate]]**         | 翻译          | 神经网络机器翻译                                                                                    |
| **Amazon Fraud Detector**        | 欺诈检测        | 实时在线欺诈检测                                                                                    |
| **[[Amazon Q]]**                 | 生成式 AI 助手家族 | Q Business（企业问答）/ Q Developer（编码）/ Q Apps / Q in QuickSight / Q in Connect                  |

相关笔记：[[AWS AI托管服务]] | [[数据与分析服务]]

---

## 考试重点总结

### 高频考点

1. **生成式 AI vs 传统 ML** 的区别与适用场景
2. **Amazon Bedrock** 的核心功能与部署模式
3. **RAG（检索增强生成）** 的工作原理与使用场景
4. **Fine-tuning vs RAG vs 提示工程** 的选择逻辑
5. **监督/无监督/强化学习** 的定义与典型算法
6. **负责任 AI**：偏见、公平性、Guardrails
7. **各托管 AI 服务** 的核心功能对应场景
8. **Bedrock Agents Classic → [[Amazon Bedrock AgentCore]]**：Agents Classic 已停止新客户接入，AgentCore 是框架/模型无关的智能体生产基础设施
9. **[[Amazon Q]] 产品家族选型**：按"谁在用、做什么"判断对应的 Q 产品（Developer/Business/Apps/QuickSight/Connect）

### 场景题解题思路

```
场景分析 → 选择 AI 解决方案
├── "不想训练模型，直接用 AI" → Amazon Bedrock（基础模型）
├── "需要完整 ML 训练流程" → Amazon SageMaker
├── "图像识别/人脸识别" → Amazon Rekognition
├── "文档文字提取/OCR" → [[Amazon Textract]]
├── "语言理解/情感分析" → Amazon Comprehend
├── "聊天机器人/对话" → Amazon Lex
├── "企业知识库搜索" → [[Amazon Kendra]]
├── "个性化推荐" → [[Amazon Personalize]]
├── "语音转文字" → Amazon Transcribe
├── "已用 LangGraph/CrewAI/Strands 构建智能体，需零改造部署到生产" → [[Amazon Bedrock AgentCore]]
├── "非技术业务人员查询企业内部文档" → [[Amazon Q]] Business
└── "需要模型部署前的审批流程/标准化文档" → SageMaker Model Registry / Model Cards
```
