# Amazon Bedrock

> **Amazon Bedrock** 是 AWS 的全托管生成式 AI 服务，提供对来自多个领先 AI 公司的基础模型（Foundation Models）的统一 API 访问，无需管理基础设施。是 AIF-C01 考试权重最高的单一服务（Domain 3 核心）。**2026 年起，经典的 Bedrock Agents 更名为 "Bedrock Agents Classic" 并于 2026-07-30 起停止向新客户开放（进入维护模式，模型目录冻结）**，AWS 力推的新一代智能体平台 **Amazon Bedrock AgentCore**（2025 年 10 月 GA）成为构建复杂多智能体应用的推荐路径。
>
> 相关文档：[[大语言模型 - LLM]] | [[提示工程 - Prompt Engineering]] | [[Transformer与Embeddings]] | [[Amazon SageMaker]] | [[Amazon Kendra]] | [[Amazon Personalize]] | [[Amazon Comprehend]] | [[Amazon OpenSearch]]

---

## 核心功能概览

| 功能 | 描述 |
|------|------|
| **基础模型访问** | 统一 API 访问 Claude、Llama、Titan、Nova 等多个模型 |
| **Playgrounds** | 无需代码即可在界面测试模型 |
| **Knowledge Bases** | 全托管 RAG，连接私有知识库 |
| **Agents（Classic，维护模式）/ AgentCore** | 构建多步骤自动化 AI 代理，AgentCore 是新一代推荐路径 |
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
| **Amazon** | Titan 系列（文本、嵌入、图像）、**Nova 系列**（含 Nova 2 推理模型） | AWS 原生，合规性好 |
| **Stability AI** | Stable Diffusion | 图像生成 |
| **Cohere** | Command、Embed | 企业文本处理、嵌入向量 |
| **AI21 Labs** | Jurassic 系列 | 多语言文本生成 |
| **OpenAI**（有限预览） | GPT-5.5、GPT-5.4、Codex | 通过标准 Bedrock API 调用，继承 AWS 安全控制 |

> **考试提示**：选择模型时关注使用场景（文本生成选 Claude/Llama，图像生成选 Stable Diffusion，嵌入向量选 Titan Embeddings 或 Cohere Embed）。

### Amazon Nova 2（新一代推理模型）

> **Amazon Nova 2** 于 2025 年 AWS re:Invent 发布，包含 **Nova 2 Lite**（已 GA）和 **Nova 2 Pro**（预览）两个型号，是支持**扩展思维（Extended Thinking）**的推理模型。

- **扩展思维**：支持逐步推理和任务分解，可配置**三档思维强度**（低/中/高），在速度、智能程度和成本之间灵活权衡
- **超长上下文**：支持**百万级 Token 上下文窗口**
- **考试要点**：题目强调"需要模型进行多步骤推理、可控制推理深度与成本的权衡"时，Nova 2 的思维强度分档是典型考点

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
| **[[Amazon OpenSearch]] Serverless** | AWS 原生，长期作为推荐首选 |
| **Amazon S3 Vectors**（新） | S3 原生向量存储，Bedrock 自动创建 S3 向量桶和索引，大规模部署可节省最高约 **90%** 成本，适合成本敏感的大规模 RAG/语义搜索/Agent 记忆场景 |
| **[[Aurora]]（pgvector）** | 关系型数据库用户 |
| **Pinecone** | 专用向量数据库，简单易用 |
| **MongoDB Atlas** | 文档型数据库 |
| **Redis Enterprise Cloud** | 低延迟 |

> **考试要点**：**S3 Vectors 是成本优化优先场景的新选择**——题目强调"大规模向量存储、成本敏感"时应考虑 S3 Vectors；强调"需要成熟的全文+向量混合搜索能力"仍首选 OpenSearch Serverless。

### 数据导入注意事项

- 支持文档格式：PDF、TXT、HTML、CSV、Word 等
- 需要定期**同步 (Sync)** 数据源以更新知识库
- 使用嵌入模型将文档转为向量（默认 Amazon Titan Embeddings）

---

## Bedrock Agents：Classic 与 AgentCore（考试重点，服务状态变化）

> **Bedrock Agents（2023-11 发布）现称 "Bedrock Agents Classic"，已于 2026-07-30 起停止向新客户开放并进入维护模式**——现有工作负载不受影响，但**不再新增功能，模型目录已冻结**（2026-07-30 之后 Bedrock 新增的基础模型仅通过 AgentCore 可用）。AWS 为存量客户提供 **AgentCore Import-Agent 工具**，可将 Agents Classic 配置转换为基于 LangGraph 的实现，多数工作负载迁移周期约 2-4 周。

### Bedrock Agents Classic 核心组件（仍适用于存量部署）

| 组件 | 描述 |
|------|------|
| **Agent（代理）** | 核心 LLM，负责规划和推理 |
| **Action Groups（动作组）** | 定义代理能调用的函数/API |
| **Lambda 函数** | Action Groups 的实际执行后端 |
| **Knowledge Base** | 代理可查询的知识库 |
| **Guardrails** | 内容安全过滤 |

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

### Amazon Bedrock AgentCore（新一代智能体平台，考试重点）

> **AgentCore**（2025-07 预览，2025-10 GA）**不是 Bedrock Agents 的简单更名，而是彻底的重新架构**——Agents Classic 是单智能体框架 + 基础工具调用，AgentCore 支持构建**多智能体系统**，智能体之间可共享记忆、委派任务，并通过统一网关路由。

| AgentCore 组件                    | 说明                                                                         |
| ------------------------------- | -------------------------------------------------------------------------- |
| **Runtime（运行时）**                | 托管的智能体执行环境                                                                 |
| **Gateway（网关）**                 | 统一网关，路由多智能体间的调用和协作                                                         |
| **Memory（记忆）**                  | 跨会话/跨智能体共享的记忆能力                                                            |
| **Identity（身份管理）**              | 智能体身份和权限管理                                                                 |
| **Policy Engine（策略引擎）**         | 精确控制智能体可执行的操作（2026-03 GA）                                                  |
| **Code Interpreter**            | 智能体可执行代码完成任务                                                               |
| **Browser Tool**                | 智能体可操作浏览器完成网页任务                                                            |
| **Managed Harness（托管工具集）**      | 处理智能体工作负载的部署、伸缩、安全，是 AgentCore 中**最接近 Agents Classic 托管体验**的组件（2026-04 新增） |
| **Agent Registry / 多智能体编排**     | 管理和编排复杂的多智能体流水线                                                            |
| **Evaluations & Observability** | 智能体评估和可观测性工具                                                               |

> **考试陷阱**：题目若问"Bedrock Agents 和 AgentCore 是什么关系" → **AgentCore 并非 Agents 的改名，而是架构上的全面升级**（单智能体 → 多智能体协作系统）；题目强调"新建智能体应用" → 优先考虑 **AgentCore**（Agents Classic 已停止新客户接入）；题目描述"已有 Agents Classic 部署，评估迁移" → 使用 **Import-Agent 工具**转换为 LangGraph 实现。

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

## LLM 定价模式（考试重点）

> 使用 LLM 有两种主要定价模式，各有不同的成本和责任分工。

| 定价模式 | 说明 | 成本构成 | 适合场景 |
|---------|------|---------|---------|
| **自托管（Hosted on Own Infrastructure）** | 在自己的服务器或 EC2 上运行 LLM | 计算资源费 + 可能的许可证费 + 维护成本 | 有合规/数据主权要求、需高度定制 |
| **按分词付费（Per-Token Pricing）** | 按处理的 Token 数量计费 | 仅按实际使用量付费，无固定基础设施成本 | 弹性需求、快速上市、无需维护基础设施 |

### 按分词计费详解

> **分词（Token）** 是供应商用来对 API 调用进行定价的单位。每个 Token 代表一个离散的信息单位：
> - 文本中的字符或单词（输入 + 输出均计费）
> - 图像中的像素（多模态输入）

> **优势**：按分词付费 + AWS 托管 = 可增加可扩展性，无需投资和维护基础设施。

---

## AWS 生成式 AI 三层架构

> AWS 将生成式 AI 基础设施划分为三个层次，每层都有安全考量。

```
┌─────────────────────────────────────────┐
│ 顶层：应用层（Applications）             │
│  LLM 应用程序：代码生成、内容生成、       │
│  RAG、提示词工程、控制面板等             │
├─────────────────────────────────────────┤
│ 中间层：ML/AI 服务层                    │
│  Amazon Bedrock / SageMaker              │
│  基础模型访问、微调、知识库、代理等      │
├─────────────────────────────────────────┤
│ 底层：基础设施层（Infrastructure）       │
│  训练/推理硬件、专用 AI 加速器、        │
│  AWS 全球网络、安全保障                 │
└─────────────────────────────────────────┘
```

### AI 系统三个关键组件

| 组件 | 描述 | 安全注意事项 |
|------|------|------------|
| **输入 (Input)** | 提示词、上下文、用户数据 | 防止提示注入、数据投毒 |
| **模型 (Model)** | 基础模型、权重、参数 | 防止模型反演、未授权访问 |
| **输出 (Output)** | 补全内容、生成结果 | 内容过滤、幻觉检测 |

---

## AWS 专用 AI 硬件与安全（Task Statement 2.3）

### 专用 AI 加速器（降低成本）

| 硬件 | 用途 | 优势 |
|------|------|------|
| **AWS Trainium** | 模型**训练**加速 | 比标准 GPU 更高性价比的训练成本 |
| **AWS Inferentia** | 模型**推理**加速 | 比标准 GPU 更高性价比的推理成本 |
| **GPU（P4/P5/G5/G6）** | 通用 ML 训练和推理 | 高性能但成本较高 |

### AWS Nitro System 安全保障

> **AWS Nitro System** 具有专用硬件和相关固件，旨在**强制执行安全限制**，确保没有人可以访问您在 EC2 实例上运行的工作负载或数据。

| 特性 | 说明 |
|------|------|
| **工作负载隔离** | 即使 AWS 员工也无法访问您的数据和模型权重 |
| **适用范围** | 所有基于 Nitro 的实例，包括 Trainium、Inferentia、P4/P5/G5/G6 |
| **保护对象** | AI 模型权重 + 处理这些模型的数据 |

> **关键**：Nitro System 保护范围适用于所有基于 Nitro 的应用程序和实例，确保未经授权的个人不能访问敏感 AI 数据。

---

## PartyRock（Amazon Bedrock 体验平台）

> **PartyRock** 是基于 Amazon Bedrock 构建的免费体验平台，用于**学习和测试**生成式 AI 应用程序。

| 特性 | 说明 |
|------|------|
| **无需代码** | 可视化界面构建生成式 AI 应用 |
| **学习基础技术** | 了解基础模型如何响应不同提示词 |
| **示例应用** | 创建播放列表、问答游戏、食谱生成等 |
| **适合场景** | 学习、原型验证、演示 |

> **网址**：[partyrock.aws](https://partyrock.aws) — 免费试用，无需 AWS 账户

---

## 考试重点总结

### AIF-C01 高频考点

1. **Bedrock = 基础模型统一 API 访问**（无需管理基础设施）
2. **Knowledge Bases** = 全托管 RAG（解决私有知识访问问题）
3. **Agents Classic 已进入维护模式**：2026-07-30 起停止新客户接入，模型目录冻结，新项目应使用 **AgentCore**
4. **AgentCore 是架构升级而非改名**：支持多智能体协作、共享记忆、统一网关，而非 Agents Classic 的简单单智能体框架
5. **Guardrails** = 内容安全过滤（PII、禁止话题、提示注入防御）
6. **On-Demand vs Provisioned Throughput**：自定义模型必须用后者
7. **Fine-tuning 数据格式**：JSONL，提示-响应对
8. **自定义权重导入**：Bedrock 支持导入自定义模型权重（On-Demand 模式）
9. **按分词付费**：Token = 离散信息单位（文本中的词/图像中的像素）；可提升可扩展性
10. **Nova 2 推理模型**：支持扩展思维（三档思维强度）和百万级 Token 上下文
11. **S3 Vectors 是 Knowledge Bases 的新向量存储选项**：成本敏感的大规模场景可比 OpenSearch Serverless 节省最高约 90%
12. **三层架构**：基础设施层 / ML 服务层 / 应用层
13. **AWS Nitro System**：专用硬件强制安全限制，保护 EC2 上的工作负载和数据
14. **Trainium（训练）vs Inferentia（推理）**：专用 AI 芯片，比 GPU 更高性价比
15. **PartyRock**：基于 Bedrock 的免费学习体验平台

### 场景题解题思路

```
场景分析 → 选择 Bedrock 功能
├── "需要接入公司内部文档" → Knowledge Bases（RAG）
├── "需要构建新的多步骤任务自动化代理" → Bedrock AgentCore（而非已停止新客户接入的 Agents Classic）
├── "已有 Agents Classic 部署，需要评估迁移" → AgentCore Import-Agent 工具（转换为 LangGraph）
├── "需要多个智能体协作、共享记忆、任务委派" → AgentCore（Agents Classic 不支持多智能体协作）
├── "防止输出有害内容" → Guardrails
├── "让模型掌握公司特定写作风格" → Fine-tuning
├── "快速测试不同模型效果" → Playgrounds
├── "批量评估多个模型" → Model Evaluation
├── "构建复杂 AI 工作流" → Prompt Flows
├── "大规模向量存储，成本敏感的 RAG 场景" → Knowledge Bases + S3 Vectors
├── "需要模型做多步骤推理并控制推理深度/成本" → Amazon Nova 2（思维强度分档）
├── "学习生成式 AI，免费体验" → PartyRock
├── "降低模型训练成本" → AWS Trainium
└── "降低模型推理成本" → AWS Inferentia
```
