# AWS AIF-C01 - AI Practitioner 考试概览

> **AWS Certified AI Practitioner (AIF-C01)** 是 AWS 的 AI 基础认证，涵盖传统 ML 流程、AWS 托管 AI 服务、生成式 AI 与大语言模型的基础知识，适合 AI 工程师、数据科学家及希望将 AI 纳入业务决策的架构师。
>
> 相关文档：[[AI与ML概念]] | [[大语言模型 - LLM]] | [[Amazon Bedrock]] | [[Amazon SageMaker]] | [[负责任的AI与安全]]

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

## 各域考点速览

### Domain 1 - AI 和 ML 基础

- AI / ML / 深度学习 / 生成式 AI 的定义与区别
- 监督学习、无监督学习、强化学习
- 常见算法：回归、分类、聚类、降维
- 神经网络与深度学习基础
- NLP 基础概念
- ML 数据生命周期（CRISP-DM）

相关笔记：[[AI与ML概念]] | [[机器学习算法]] | [[深度学习与神经网络]]

### Domain 2 - 生成式 AI 基础

- 生成式 AI 是深度学习子集，创建**全新原创内容**（非分类/预测）
- 模型输出机制：预测下一个词/Token（概率性过程）
- 参数量 → 规模 → 内存 → 能力（正向关系）
- 数学基础：概率建模、损失函数、矩阵乘法
- Transformer 架构与注意力机制（2017, "Attention Is All You Need"）
- Tokenization 与 Embeddings（向量数据库）
- 模态（Modalities）：文本、图像、音频、视频
- 提示工程（Prompt Engineering）：零样本、**单样本**、少样本、思维链
- **上下文学习 (In-Context Learning)**：在提示词中加入示例，无需重新训练
- **补全 (Completion)**：模型输出；**推理 (Inference)**：提示词→模型→补全的过程

相关笔记：[[大语言模型 - LLM]] | [[Transformer与Embeddings]] | [[提示工程 - Prompt Engineering]]

### Domain 3 - 基础模型应用

- Amazon Bedrock：基础模型访问、Fine-tuning、RAG
- Bedrock Agents、Knowledge Bases、Guardrails
- 模型评估与选择
- SageMaker：ML 完整流程

相关笔记：[[Amazon Bedrock]] | [[Amazon SageMaker]]

### Domain 4 - 负责任的 AI

- AI 偏见（Bias）与公平性（Fairness）
- 可解释性（Explainability）与透明度
- 数据隐私与 PII 处理
- Guardrails 内容过滤

### Domain 5 - 安全、合规、治理

- AI 安全威胁（Prompt Injection、数据投毒）
- IAM 权限控制
- CloudWatch 日志监控
- 数据加密与合规要求

相关笔记：[[负责任的AI与安全]]

---

## 重要 AWS AI 服务汇总

| 服务 | 类别 | 核心功能 |
|------|------|---------|
| **Amazon Bedrock** | 生成式 AI | 访问基础模型（Claude、Titan、Llama 等） |
| **Amazon SageMaker** | ML 平台 | 完整 ML 训练、部署、监控流程 |
| **Amazon Rekognition** | 计算机视觉 | 图像/视频中的对象、人脸、文字识别 |
| **Amazon Comprehend** | NLP | 情感分析、实体提取、PII 检测 |
| **Amazon Lex** | 对话 AI | 语音/文字聊天机器人（Alexa 商业版） |
| **Amazon Kendra** | 企业搜索 | 语义搜索引擎 |
| **Amazon Personalize** | 推荐系统 | 个性化推荐（用户-物品交互） |
| **Amazon Polly** | 文字转语音 | 多语言 TTS，支持 SSML |
| **Amazon Textract** | 文档处理 | OCR、表单提取、签名检测 |
| **Amazon Transcribe** | 语音转文字 | 自动语音识别 (ASR) |
| **Amazon Translate** | 翻译 | 神经网络机器翻译 |
| **Amazon Fraud Detector** | 欺诈检测 | 实时在线欺诈检测 |
| **Amazon Q** | AI 助手 | 企业级 AI 聊天助手 |

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

### 场景题解题思路

```
场景分析 → 选择 AI 解决方案
├── "不想训练模型，直接用 AI" → Amazon Bedrock（基础模型）
├── "需要完整 ML 训练流程" → Amazon SageMaker
├── "图像识别/人脸识别" → Amazon Rekognition
├── "文档文字提取/OCR" → Amazon Textract
├── "语言理解/情感分析" → Amazon Comprehend
├── "聊天机器人/对话" → Amazon Lex
├── "企业知识库搜索" → Amazon Kendra
├── "个性化推荐" → Amazon Personalize
└── "语音转文字" → Amazon Transcribe
```
