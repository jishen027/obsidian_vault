# 大语言模型 - LLM

> **大语言模型 (Large Language Model, LLM)** 是生成式 AI 的文本子集，能生成类人自然语言文本，基于 Transformer 架构和大规模语料训练。理解 LLM 的核心概念是 AIF-C01 考试中权重最高的内容（Domain 2 + 3）。
>
> 相关文档：[[Transformer与Embeddings]] | [[提示工程 - Prompt Engineering]] | [[Amazon Bedrock]] | [[AI与ML概念]]

---

## 生成式 AI 与 LLM 的关系

```
生成式 AI (Generative AI)
├── 文本生成 → LLM（大语言模型）← 最常见、最流行
├── 图像生成 → Stable Diffusion、DALL-E、Amazon Titan Image
├── 音频生成 → 音乐合成、语音克隆
└── 视频生成 → 视频合成模型
```

> LLM 是文本模态，是生成式 AI 中最流行的部分，但常被误认为等同于生成式 AI（实际上是子集）。现代 LLM 正进化为**多模态 (Multimodal)**。

---

## 基础模型 (Foundation Models)

> **基础模型 (Foundation Models, FM)** 是在海量数据上预训练的大型模型，可作为多种下游任务的起点，无需从头训练。

### 基础模型特点

| 特性 | 描述 |
|------|------|
| **大规模预训练** | 在数 TB 文本数据上训练，耗费数百万美元 |
| **通用能力** | 一个模型能胜任翻译、摘要、问答、代码生成等多种任务 |
| **可微调** | 可通过少量数据适配特定领域 |
| **零样本/少样本学习** | 无需额外训练即可执行新任务 |

### 主流基础模型（Amazon Bedrock 提供）

| 模型 | 提供商 | 特点 |
|------|--------|------|
| **Claude** | Anthropic | 长上下文、安全性强 |
| **Llama** | Meta | 开源基础模型 |
| **Mistral** | Mistral AI | 轻量高效 |
| **Titan** | Amazon | AWS 自研系列 |
| **Stable Diffusion** | Stability AI | 图像生成 |
| **Cohere** | Cohere | 企业级文本处理 |

---

## 基础模型规模对比

| 模型 | 层数 | 参数量级 | 特点 |
|------|------|---------|------|
| **GPT-3** | 96 层 | 1750 亿 | 较早的大型模型 |
| **现代大模型** | 数百层 | 数千亿 | 更强推理能力 |

> 参数量越大，节点间连接越多，训练成本越高，但能力也越强。

---

## 定制化基础模型的三种方式（考试重点）

| 方法 | 描述 | 成本 | 适用场景 |
|------|------|------|---------|
| **提示工程 (Prompt Engineering)** | 通过精心设计输入提示引导模型 | 最低（无需训练） | 快速迭代、通用任务 |
| **RAG（检索增强生成）** | 检索外部知识库注入上下文 | 低（无需训练模型） | 需要最新/私有知识 |
| **Fine-tuning（微调）** | 用特定数据集继续训练模型 | 高（需要训练资源） | 特定领域风格/知识 |

### 选择逻辑

```
选择定制化方式
├── "需要最新信息（实时数据）" → RAG
├── "需要私有企业知识" → RAG（Knowledge Base）
├── "需要特定风格/语气" → Fine-tuning
├── "需要特定领域专业知识" → Fine-tuning 或 Domain-specific RAG
└── "快速测试和原型" → 提示工程（Prompt Engineering）
```

---

## Fine-tuning 微调

> **Fine-tuning** 是在预训练基础模型上，使用较小的特定数据集继续训练，使模型适应特定任务或领域。

### 微调类型

| 类型 | 描述 |
|------|------|
| **指令微调 (Instruction Fine-tuning)** | 提供输入-输出对示例，教模型遵循指令 |
| **领域特定微调 (Domain-specific Fine-tuning)** | 使用特定领域知识库微调，增强专业能力 |
| **持续预训练 (Continuous Pre-training)** | 在已有知识上追加训练新知识 |

### Fine-tuning 数据格式

```json
{
  "prompt": "系统提示（定义任务）\n\n### 输入：\n[具体输入内容]\n\n### 响应：",
  "completion": "[期望的模型输出]"
}
```

> 常用 **JSONL 格式**（每行一个 JSON 对象）作为微调数据集格式。

---

## RAG（检索增强生成）

> **RAG (Retrieval-Augmented Generation)** 在 LLM 生成响应之前，先从外部知识库检索相关信息，注入上下文，从而避免模型幻觉、提供最新知识。

### RAG 工作流程

```
用户提问 → 向量化查询 → 搜索向量数据库 → 检索相关文档块
    ↓                                              ↓
生成响应 ← LLM 推理 ← 注入上下文 + 提问 ← 返回相关知识
```

### RAG 核心组件

| 组件 | 描述 | AWS 对应服务 |
|------|------|------------|
| **向量数据库** | 存储文档的向量嵌入 | OpenSearch、Aurora（pgvector）、Pinecone |
| **嵌入模型** | 将文本转为向量 | Amazon Titan Embeddings、Cohere Embed |
| **知识库** | 文档存储 | Amazon S3 |
| **检索器** | 根据查询找到最相关文档 | Amazon Bedrock Knowledge Bases |
| **LLM** | 基于检索结果生成答案 | Amazon Bedrock 各模型 |

### RAG 数据库选择

| 向量数据库 | 特点 |
|-----------|------|
| **Amazon OpenSearch Serverless** | AWS 原生，全托管，支持向量搜索 |
| **Amazon Aurora (pgvector)** | 关系型数据库扩展，适合已用 Aurora 的场景 |
| **Pinecone** | 第三方，易用，多种嵌入支持，丰富云集成 |
| **MongoDB Atlas** | 文档数据库 + 向量搜索 |
| **Redis Enterprise Cloud** | 低延迟，适合实时查询 |

---

## 模型幻觉 (Hallucination)

> **幻觉** 是 LLM 自信地生成错误或虚构信息的现象。

| 原因 | 缓解方法 |
|------|---------|
| 训练数据截止日期之后的信息 | 使用 RAG 提供最新知识 |
| 训练数据中的错误或偏见 | Fine-tuning 修正 |
| 模型对边缘知识的不确定 | 提示中要求引用来源 |
| 温度参数过高 | 降低 Temperature 参数 |

---

## 模型推理参数

| 参数 | 描述 | 影响 |
|------|------|------|
| **Temperature** | 控制输出随机性（0-1） | 低→确定性强，高→创意性强 |
| **Top P** | 概率核采样（0-1） | 控制输出词汇的多样性 |
| **Top K** | 每步只从前 K 个词中选择 | 限制候选词范围 |
| **Max Tokens** | 最大输出长度 | 控制响应长度 |

```
参数调整场景
├── "需要一致性/精确答案" → Temperature 低（接近 0）
├── "需要创意写作/多样输出" → Temperature 高（接近 1）
└── "控制成本/简洁回复" → Max Tokens 设置上限
```

---

## 部署模式（Amazon Bedrock）

| 模式 | 描述 | 成本特点 |
|------|------|---------|
| **On-Demand（按需）** | 无需预置，按调用次数计费（类似 Serverless） | 弹性，适合变化负载 |
| **Provisioned Throughput（预置吞吐）** | 预先购买固定吞吐量 | 固定成本，适合稳定高频使用 |

> 使用**自定义模型（Fine-tuned）** 时必须使用 **Provisioned Throughput**，因为需要持续部署资源。

---

## 考试重点总结

### AIF-C01 高频考点

1. **LLM 是生成式 AI 的文本子集**（非等价关系）
2. **三种定制方式**：提示工程（免训练）/ RAG（实时知识）/ Fine-tuning（专业能力）
3. **RAG 解决幻觉问题**：检索最新/私有知识注入上下文
4. **Temperature 参数**：低=确定，高=创意
5. **部署模式**：On-Demand vs Provisioned Throughput，自定义模型必须用后者

### 场景题解题思路

```
场景分析 → 选择 LLM 定制方式
├── "模型不知道公司内部文档" → RAG（Knowledge Base）
├── "需要模型写特定格式报告" → Fine-tuning（指令微调）
├── "回答总是太随机/不一致" → 降低 Temperature
├── "需要最新新闻信息" → RAG（不能靠 Fine-tuning，数据会过期）
└── "快速验证 AI 方案可行性" → 提示工程（Prompt Engineering）
```
