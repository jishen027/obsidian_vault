# Transformer 架构与 Embeddings

> **Transformer** 是由 Google 研究人员提出的神经网络架构（论文《Attention Is All You Need》），通过**多头注意力机制**和**位置编码**实现高效的自然语言处理，是所有现代 LLM 的基础。**Embeddings（嵌入）** 是将文本转换为数值向量的方法，是 RAG 和语义搜索的关键。
>
> 相关文档：[[大语言模型 - LLM]] | [[提示工程 - Prompt Engineering]] | [[Amazon Bedrock]]

---

## Transformer 架构

### 核心特点

| 特性 | 描述 |
|------|------|
| **多头注意力 (Multi-Head Attention)** | 并行关注输入的不同方面，捕获多维度语义关系 |
| **位置编码 (Positional Encoding)** | 为序列中的每个词添加位置信息（Transformer 本身无顺序感知） |
| **自注意力 (Self-Attention)** | 序列内部元素互相关注，理解上下文关系 |
| **交叉注意力 (Cross-Attention)** | 编码器与解码器之间的注意力（用于翻译等任务） |
| **并行计算** | 与 RNN 不同，Transformer 可并行处理整个序列，训练更快 |

### Encoder-Decoder 结构

```
输入文本 → [编码器 Encoder] → 语义表示
                                    ↓
                            [解码器 Decoder] → 输出文本
```

| 组件 | 职责 | 典型用途 |
|------|------|---------|
| **Encoder（编码器）** | 理解输入，生成语义表示 | 文本理解、分类、嵌入 |
| **Decoder（解码器）** | 基于编码器输出生成新序列 | 文本生成、翻译 |
| **Encoder-Decoder** | 两者结合 | 翻译（输入→输出序列） |

---

## Tokenization（词元化）

> **Tokenization** 是将输入文本转换为模型内部词汇表的 Token 序列的过程，是 LLM 处理文本的第一步。

### 工作原理

```
原始文本 → Tokenizer → Token ID 序列 → 模型处理
"Hello World" → [15339, 1917] → 模型推理 → 生成下一个 Token
```

### 关键概念

| 概念 | 描述 |
|------|------|
| **Token** | 文本分割的最小单位，可以是字、词或子词 |
| **词汇表 (Vocabulary)** | 模型训练时建立的所有已知 Token 的集合 |
| **上下文窗口 (Context Window)** | 模型一次能处理的最大 Token 数量 |
| **Tokenizer** | 将文本转换为 Token ID 的工具 |

> LLM 按 **Token** 计费，而非按字符或单词计费。1 Token ≈ 0.75 个英文词（中文每个汉字通常为 1-2 个 Token）。

---

## Embeddings（嵌入向量）

> **Embeddings** 是将文本（或其他数据）转换为数值向量，使计算机能计算语义相似度。语义相近的词/句在向量空间中距离更近。

### 核心用途

| 用途 | 描述 |
|------|------|
| **语义搜索** | 基于含义而非关键词搜索相似内容 |
| **RAG 知识检索** | 将查询和文档转为向量，计算相似度找到相关内容 |
| **聚类分析** | 找到语义相似的文本群组 |
| **推荐系统** | 找到与用户偏好相似的内容 |

### 向量空间示例

```
语义相似的词 → 向量距离近
"苹果" → [0.2, 0.8, 0.3, ...]  (水果)
"香蕉" → [0.2, 0.7, 0.4, ...]  (水果) ← 距离近
"汽车" → [0.9, 0.1, 0.6, ...]  (交通) ← 距离远

蔬菜、肉类、乳制品 → 在向量空间中各自形成簇
```

### AWS 提供的嵌入模型

| 模型 | 提供商 | 特点 |
|------|--------|------|
| **Amazon Titan Embeddings** | Amazon | AWS 原生，与 Bedrock 深度集成 |
| **Cohere Embed** | Cohere | 高质量多语言嵌入 |

---

## 向量数据库 (Vector Database)

> **向量数据库** 专门存储和检索高维向量，支持高效的相似度搜索（最近邻搜索）。是 RAG 架构的核心组件。

### 相似度搜索算法

| 算法 | 全称 | 特点 |
|------|------|------|
| **KNN** | K-Nearest Neighbor | 精确搜索，计算量大 |
| **ANN** | Approximate Nearest Neighbor | 近似搜索，更快但精度略低 |
| **HNSW** | Hierarchical Navigable Small World | 高效 ANN 算法，广泛使用 |

### 常用向量数据库对比

| 数据库 | 类型 | 特点 |
|--------|------|------|
| **Amazon OpenSearch Serverless** | AWS 托管 | 全文搜索 + 向量搜索，AWS 原生集成 |
| **Amazon Aurora (pgvector)** | 关系型 + 向量 | 已有 Aurora 用户的首选扩展 |
| **Pinecone** | 专用向量数据库 | 使用最简单，丰富云集成 |
| **MongoDB Atlas** | 文档 + 向量 | 灵活的文档模型 |
| **Redis Enterprise Cloud** | 内存 + 向量 | 毫秒级低延迟 |

---

## 注意力机制详解

### 自注意力 (Self-Attention)

> 让序列中每个词都能"关注"到其他词，理解上下文关系。

**示例**："银行倒了很多树" vs "他去银行取钱"
- 自注意力帮助模型理解"银行"在不同上下文中的不同含义

### 多头注意力 (Multi-Head Attention)

> 并行运行多个注意力头，每个头关注输入的不同方面。

```
多头注意力 = [注意力头1 + 注意力头2 + ... + 注意力头N] → 合并
                  ↓                ↓
            语法关系         语义关系         ... 等多维度
```

---

## 考试重点总结

### AIF-C01 高频考点

1. **Transformer 是现代 LLM 的基础**（来自《Attention Is All You Need》）
2. **多头注意力机制**：并行捕获不同维度的语义关系
3. **Tokenization**：文本→Token 序列，按 Token 计费
4. **上下文窗口**：模型一次能处理的最大 Token 数
5. **Embeddings**：将文本转为向量，用于语义搜索和 RAG
6. **向量数据库**：存储嵌入向量，支持相似度搜索

### 场景题解题思路

```
场景分析 → 选择向量数据库
├── "AWS 原生，全文+向量搜索" → Amazon OpenSearch Serverless
├── "已有 Aurora，想加向量搜索" → Aurora pgvector
├── "最简单，入门快" → Pinecone
├── "低延迟实时查询" → Redis Enterprise Cloud
└── "灵活文档模型" → MongoDB Atlas
```
