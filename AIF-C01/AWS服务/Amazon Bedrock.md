# Amazon Bedrock

> **Amazon Bedrock** 是 AWS 的全托管生成式 AI 服务，提供对来自多个领先 AI 公司的基础模型（Foundation Models）的统一 API 访问，无需管理基础设施。是 AIF-C01 考试权重最高的单一服务（Domain 3 核心）。
>
> 相关文档：[[大语言模型 - LLM]] | [[提示工程 - Prompt Engineering]] | [[Transformer与Embeddings]] | [[Amazon SageMaker]]

---

## 核心功能概览

| 功能 | 描述 |
|------|------|
| **基础模型访问** | 统一 API 访问 Claude、Llama、Titan 等多个模型 |
| **Playgrounds** | 无需代码即可在界面测试模型 |
| **Knowledge Bases** | 全托管 RAG，连接私有知识库 |
| **Agents** | 构建多步骤自动化 AI 代理 |
| **Guardrails** | 内容安全过滤层 |
| **Fine-tuning** | 使用自定义数据微调基础模型 |
| **Model Evaluation** | 评估和比较模型性能 |
| **Prompt Management** | 管理和版本化提示模板 |
| **Prompt Flows** | 可视化构建 LLM 工作流 |

---

## 可用基础模型（考试重点）

| 提供商 | 模型 | 特点 |
|--------|------|------|
| **Anthropic** | Claude 系列 | 长上下文、安全对齐、强推理 |
| **Meta** | Llama 系列 | 开源，轻量高效 |
| **Mistral AI** | Mistral 系列 | 高性价比 |
| **Amazon** | Titan 系列（文本、嵌入、图像） | AWS 原生，合规性好 |
| **Stability AI** | Stable Diffusion | 图像生成 |
| **Cohere** | Command、Embed | 企业文本处理、嵌入向量 |
| **AI21 Labs** | Jurassic 系列 | 多语言文本生成 |

> **考试提示**：选择模型时关注使用场景（文本生成选 Claude/Llama，图像生成选 Stable Diffusion，嵌入向量选 Titan Embeddings 或 Cohere Embed）。

---

## 部署模式（考试重点）

| 模式 | 别称 | 计费 | 适合场景 |
|------|------|------|---------|
| **On-Demand** | Serverless（按需） | 按 Token 调用量 | 变化负载、开发测试 |
| **Provisioned Throughput** | 预置吞吐 | 固定时间费用 | 稳定高频使用、自定义模型 |

> **重要**：使用 **Fine-tuned 自定义模型** 进行推理时，**必须使用 Provisioned Throughput**，不支持 On-Demand 模式。

---

## Bedrock Knowledge Bases（RAG）

> **Knowledge Bases** 是 Bedrock 的全托管 RAG 解决方案，自动处理文档解析、向量化、存储和检索全流程。

### RAG 工作流程

```
上传文档到 S3 → 同步数据源 → 自动向量化 → 存入向量数据库
                                                    ↓
用户提问 → 查询向量数据库 → 检索相关文档块 → 注入 LLM 上下文 → 生成回答
```

### 支持的向量数据库

| 数据库 | 特点 |
|--------|------|
| **Amazon OpenSearch Serverless** | AWS 原生，推荐首选 |
| **Amazon Aurora（pgvector）** | 关系型数据库用户 |
| **Pinecone** | 专用向量数据库，简单易用 |
| **MongoDB Atlas** | 文档型数据库 |
| **Redis Enterprise Cloud** | 低延迟 |

### 数据导入注意事项

- 支持文档格式：PDF、TXT、HTML、CSV、Word 等
- 需要定期**同步 (Sync)** 数据源以更新知识库
- 使用嵌入模型将文档转为向量（默认 Amazon Titan Embeddings）

---

## Bedrock Agents

> **Bedrock Agents** 允许构建能够规划和执行多步骤任务的 AI 代理，自动调用外部 API 或 Lambda 函数完成复杂任务。

### 核心组件

| 组件 | 描述 |
|------|------|
| **Agent（代理）** | 核心 LLM，负责规划和推理 |
| **Action Groups（动作组）** | 定义代理能调用的函数/API |
| **Lambda 函数** | Action Groups 的实际执行后端 |
| **Knowledge Base** | 代理可查询的知识库 |
| **Guardrails** | 内容安全过滤 |

### Agent 工作原理

```
用户请求 → Agent 分析任务
              ↓
    决定调用哪个 Action Group
              ↓
    调用 Lambda 函数（可能查询数据库）
              ↓
    汇总结果，可能再次调用其他 Action
              ↓
    生成最终回答返回用户
```

---

## Bedrock Guardrails（内容安全）

> **Guardrails** 是 Bedrock 的内容安全过滤层，可拦截输入和输出中的有害内容。

### 过滤类型

| 类型 | 描述 |
|------|------|
| **内容过滤** | 过滤仇恨、暴力、色情等有害内容 |
| **禁止话题过滤** | 定义禁止讨论的话题（最多 30 个），可提供 5 个示例短语 |
| **词汇过滤** | 屏蔽特定词汇 |
| **PII 过滤** | 检测和遮蔽个人身份信息（姓名、电话、邮件等），支持屏蔽或脱敏 |
| **基础真实性过滤** | 验证响应是否基于提供的参考来源（事实性检查） |

### 使用场景

```
场景分析 → 使用 Guardrails
├── "防止 LLM 讨论竞争对手产品" → 禁止话题过滤
├── "防止用户输入覆盖系统指令（提示注入）" → 输入过滤
├── "防止输出包含用户个人信息" → PII 过滤
└── "确保回答基于知识库内容（防幻觉）" → 基础真实性过滤
```

---

## Fine-tuning（Bedrock 自定义模型）

### Fine-tuning vs 持续预训练

| 方式 | 数据格式 | 目标 |
|------|---------|------|
| **Fine-tuning（微调）** | 提示-响应对（JSONL） | 教会模型新任务或风格 |
| **Continued Pre-training** | 原始文本 | 注入领域知识 |
| **Import Custom Model** | 导入外部模型权重 | 使用已有自训练模型 |

### 微调数据格式

```json
{"prompt": "系统提示\n\n### 输入：用户问题\n\n### 响应：", "completion": "期望的回答"}
```

> 自定义模型必须使用 **Provisioned Throughput** 进行推理，无法使用 On-Demand 模式。

---

## Bedrock Model Evaluation

> **模型评估** 允许系统化比较不同基础模型或微调版本的性能。

### 评估方式

| 方式 | 描述 |
|------|------|
| **自动评估** | 使用内置指标自动对比（ROUGE、METEOR、BERTScore 等） |
| **人工评估** | 引入人类评估员对输出质量打分 |

---

## Bedrock Prompt Flows（提示流）

> **Prompt Flows** 是可视化的 LLM 工作流构建器，支持条件逻辑、循环、并行处理。

### 支持节点类型

| 节点类型 | 功能 |
|---------|------|
| **Prompt 节点** | 调用 LLM 生成文本 |
| **条件节点** | 基于条件路由不同分支 |
| **Iterator 节点** | 遍历列表项 |
| **Lambda 节点** | 调用 AWS Lambda 函数 |
| **Knowledge Base 节点** | 查询知识库 |
| **Agent 节点** | 调用 Bedrock Agent |
| **S3 节点** | 读写 S3 数据 |

---

## CloudWatch 监控

- Bedrock 调用日志可发送到 **CloudWatch Log Groups**
- 日志包含：调用的模型、输入/输出内容、Token 使用量
- 可用于成本分析、使用量监控、安全审计

---

## 考试重点总结

### AIF-C01 高频考点

1. **Bedrock = 基础模型统一 API 访问**（无需管理基础设施）
2. **Knowledge Bases** = 全托管 RAG（解决私有知识访问问题）
3. **Agents** = 多步骤任务自动化代理
4. **Guardrails** = 内容安全过滤（PII、禁止话题、提示注入防御）
5. **On-Demand vs Provisioned Throughput**：自定义模型必须用后者
6. **Fine-tuning 数据格式**：JSONL，提示-响应对

### 场景题解题思路

```
场景分析 → 选择 Bedrock 功能
├── "需要接入公司内部文档" → Knowledge Bases（RAG）
├── "需要自动完成多步骤任务" → Bedrock Agents
├── "防止输出有害内容" → Guardrails
├── "让模型掌握公司特定写作风格" → Fine-tuning
├── "快速测试不同模型效果" → Playgrounds
├── "批量评估多个模型" → Model Evaluation
└── "构建复杂 AI 工作流" → Prompt Flows
```
