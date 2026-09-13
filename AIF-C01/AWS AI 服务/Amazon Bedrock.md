# Amazon Bedrock

> **Amazon Bedrock** 是 AWS 的全托管生成式 AI 服务，提供对来自多个领先 AI 公司的基础模型（Foundation Models）的统一 API 访问，无需管理基础设施。是 AIF-C01 考试权重最高的单一服务（Domain 3 核心）。**2026 年起，经典的 Bedrock Agents 更名为 "Bedrock Agents Classic" 并于 2026-07-30 起停止向新客户开放（进入维护模式，模型目录冻结）**，AWS 力推的新一代智能体平台 **[[Amazon Bedrock AgentCore]]**（2025 年 10 月 GA）成为构建复杂多智能体应用的推荐路径，完整能力见 [[Amazon Bedrock AgentCore]] 独立笔记。
>
> 相关文档：[[Amazon Bedrock AgentCore]] | [[大语言模型 - LLM]] | [[提示工程 - Prompt Engineering]] | [[Transformer与Embeddings]] | [[Amazon SageMaker]] | [[Amazon Kendra]] | [[Amazon Personalize]] | [[Amazon Comprehend]] | [[Amazon OpenSearch]]

---

## 目录

- [[#核心功能概览]]
- [[#可用基础模型（考试重点）]]
- [[#部署模式（考试重点）]]
- [[#Bedrock Knowledge Bases（RAG）]]
- [[#Bedrock Agents：Classic 与 AgentCore（考试重点，服务状态变化）]]
- [[#Bedrock Guardrails（内容安全）]]
- [[#Fine-Tuning a Model（模型微调）]]
- [[#Model Distillation（模型蒸馏）]]
- [[#Bedrock Model Evaluation]]
- [[#Bedrock Prompt Flows（提示流）]]
- [[#CloudWatch 监控]]
- [[#LLM 定价模式（考试重点）]]
- [[#AWS 生成式 AI 三层架构]]
- [[#AWS 专用 AI 硬件与安全（Task Statement 2.3）]]
- [[#PartyRock（Amazon Bedrock 体验平台）]]
- [[#考试重点总结]]

---

## 核心功能概览

| 功能 | 描述 |
|------|------|
| **基础模型访问** | 统一 API 访问 Claude、Llama、Titan、Nova 等多个模型 |
| **Playgrounds** | 无需代码即可在界面测试模型 |
| **Knowledge Bases** | 全托管 RAG，连接私有知识库 |
| **Agents（Classic，维护模式）/ AgentCore** | 构建多步骤自动化 AI 代理，AgentCore 是新一代推荐路径 |
| **Guardrails** | 内容安全过滤层 |
| **Fine-tuning（SFT / RFT）** | 使用自定义数据（监督微调）或奖励函数（强化微调）定制基础模型 |
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
| **On-Demand** | Serverless（按需） | 按 Token 调用量，**无时间承诺** | 变化负载、开发测试 |
| **Provisioned Throughput** | 预置吞吐 | 固定时间费用（**1 个月/6 个月承诺**，也有无承诺的按小时计费选项） | 稳定高频使用、多数自定义模型 |

> **重要（考试高频陷阱）**：① **On-Demand 不需要任何时间承诺**——题目若说"On-Demand 只能搭配时间承诺使用"是把 Provisioned Throughput 的特征错误地安在了 On-Demand 上。② 自定义模型能否用 **On-Demand** 推理**因模型提供商而异**，不能一概而论——例如 **Anthropic** 的模型完成微调后可**立即用于 On-Demand 推理**；而 **Meta Llama 2** 等模型微调后**仅支持 Provisioned Throughput**。AIF-C01 的正确通用结论是"**自定义模型可以用 Provisioned Throughput 或 On-Demand 模式**"（取决于具体模型），而**"自定义模型只能用 Provisioned Throughput"是常见的过度绝对化错误答案**——题目若要求选出"适用于 Bedrock 的准确表述"，应选前者而非后者。

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

> **AgentCore**（2025-07 预览，2025-10 GA）**不是 Bedrock Agents 的简单更名，而是彻底的重新架构**——核心定位是**框架无关（兼容 Strands/LangGraph/CrewAI/LlamaIndex 等）+ 模型无关**的智能体生产基础设施，原生支持构建**多智能体系统**，智能体之间可共享记忆、委派任务，并通过统一网关路由。核心组件（Runtime、Gateway、Memory、Identity、Policy Engine 等）按"按需拼装"理念设计，可分别独立启用。完整组件详解、与主流智能体框架的关系、迁移路径见 [[Amazon Bedrock AgentCore]] 独立笔记。

> **考试陷阱**：题目若问"Bedrock Agents 和 AgentCore 是什么关系" → **AgentCore 并非 Agents 的改名，而是架构上的全面升级**（单一 Bedrock 原生框架 → 框架无关的多智能体生产基础设施）；题目强调"新建智能体应用" → 优先考虑 **AgentCore**（Agents Classic 已停止新客户接入）；题目描述"已有 Agents Classic 部署，评估迁移" → 使用 **Import-Agent 工具**转换为 LangGraph 实现。

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

## Fine-Tuning a Model（模型微调）

> **Fine-tuning** 是在 Bedrock 提供的基础模型上继续训练，产出一个只属于你账户的**自定义模型（Custom Model）**——用于让模型稳定输出特定**风格/格式**、掌握**特定任务模式**，而不必在每次调用时都靠冗长的提示词反复"教"模型。微调背后的原理（迁移学习、RLHF、HHH 原则）和"提示工程 → RAG → Fine-tuning"的通用选择逻辑详见 [[大语言模型 - LLM]] 独立笔记；本节聚焦 **Bedrock 作为产品**实现微调的具体机制，包括两种技术路线——**监督微调（SFT）**和**强化微调（RFT）**。

### 五种自定义方式对比（考试重点）

| 方式 | 数据格式 | 目标 |
|------|---------|------|
| **监督微调（Supervised Fine-Tuning, SFT）** | 提示-响应对（JSONL，**有标签**，标签**来自人工**） | 用人工标注的标准答案教会模型新任务或风格，是最经典的 Fine-tuning 形式 |
| **强化微调（Reinforcement Fine-Tuning, RFT）** | 提示 + **奖励函数（Reward Function）**，**无需**大量人工标注答案 | 让模型针对同一提示生成多个候选回答，由奖励函数打分后用强化学习持续优化，适合"有明确评判标准但没有海量标准答案"的任务 |
| **模型蒸馏（Distillation）** | 只需**提示**，标准答案由**更大的教师模型自动生成**，无需人工标注也无需奖励函数 | 把大模型（教师）在特定任务上的能力"压缩"进一个更小、更快、更便宜的模型（学生），完整机制见下方 [[#Model Distillation（模型蒸馏）]] 独立章节 |
| **Continued Pre-training（持续预训练）** | 原始无标签文本 | 注入领域知识，不改变任务行为 |
| **Import Custom Model（导入自定义模型）** | 外部已训练好的模型权重 | 直接托管使用已有的自训练模型，无需在 Bedrock 内重新训练 |

> **考试陷阱**：**三种"教模型新行为"的方式，区分依据是"标准答案从哪来"**——人工标注 → **SFT**；没有标准答案、靠自动打分规则 → **RFT**；标准答案由**更大的模型自动生成**、你只需准备提示 → **模型蒸馏**。题目描述"有大量标注好的示例问答对，希望模型模仿这种问答风格" → SFT；题目描述"很难穷举标准答案，但能写出一个自动评分规则/奖励函数来判断回答好坏" → RFT；题目描述"希望用更便宜、更快的小模型达到大模型在某项具体任务上的效果，且不想自己标注数据" → **模型蒸馏**。另外，**"教模型说话风格/输出格式" 和 "让模型多懂一些领域知识" 也是两回事**——前者用 SFT/RFT/蒸馏，后者用 **Continued Pre-training**（无标签原始文本），不要混为一谈。

### SFT 数据格式

```json
{"prompt": "系统提示\n\n### 输入：用户问题\n\n### 响应：", "completion": "期望的回答"}
```

- 训练数据以 **JSONL** 格式存放在 [[S3]]，Bedrock 需要具备读取该 S3 位置的 **IAM 角色**权限才能启动微调作业
- 可选提供**验证数据集**，用于在训练过程中评估模型效果，避免仅凭训练集表现做判断
- 支持配置**超参数**：训练轮数（Epochs）、批大小（Batch Size）、学习率倍数（Learning Rate Multiplier），用于在拟合程度和过拟合风险之间调整

### 强化微调（RFT）的工作机制（考试提示）

> RFT 是 Bedrock 较新的模型自定义能力，**全托管**完成整个强化学习流程，无需自建 RL 训练管道、编排逻辑或专用训练基础设施。

| 步骤 | 说明 |
|------|------|
| **1. 生成候选回答** | 模型针对训练集中的每个提示，**生成多个候选回答**（而非只有一个标准答案） |
| **2. 奖励函数打分** | 由**奖励函数（Reward Function）**对每个候选回答**打分**，评判依据由开发者自定义（如代码能否通过测试、答案是否满足业务规则） |
| **3. 策略优化训练** | Bedrock 使用**分组相对策略优化（Group Relative Policy Optimization, GRPO）**这一策略型强化学习算法，根据打分结果持续优化模型 |

- **效果**：AWS 官方数据显示，RFT 平均可比基础模型带来约 **66%** 的准确率提升，让**更小、更便宜的模型**在特定任务上达到接近大模型的表现
- **模型支持范围有限**：RFT 目前仅对**部分模型**开放（含通过 OpenAI 兼容 API 支持的开放权重模型，如 GPT-OSS、Qwen 等），并非所有 Bedrock 基础模型都支持
- **考试要点**：**RFT 不需要海量人工标注的"标准答案"，但需要一个能自动评判回答优劣的奖励函数**——题目描述"任务的正确性可以通过程序自动校验（如代码是否运行成功），但没有足够的人工标注数据" → **RFT** 往往比 SFT 更合适；题目描述"已经有大量高质量的人工标注问答对" → **SFT** 是更直接的选择。

### 并非所有基础模型都支持微调（考试提示）

- **模型自定义能力因模型而异**——并不是 Bedrock 目录里的每个模型都支持 SFT/RFT，具体哪些模型开放该能力需以 Bedrock 控制台的模型详情为准；**RFT 目前开放支持的模型范围比 SFT 更窄**
- **考试要点**：题目若假设"任意 Bedrock 模型都能直接微调"，属于常见的过度概括陷阱——实际选型时必须先确认目标模型是否**明确支持对应的自定义方式（SFT 或 RFT）**，而非默认所有基础模型行为一致。

### 灾难性遗忘（Catastrophic Forgetting，考试提示）

- 用**过窄、过度拟合**的数据集微调，可能导致模型在获得新技能的同时**丢失部分原有的通用能力**（如变得只会按固定模板回答，通用问答能力退化）
- **考试要点**：题目描述"微调后模型在新任务上表现很好，但在其他原本擅长的任务上明显变差" → 这是**灾难性遗忘**的典型表现，缓解手段包括扩充训练数据的多样性、控制训练轮数不要过度拟合。

### 微调后的推理与成本（考试高频）

- **自定义模型（含 SFT、RFT、蒸馏（Distillation）和 Continued Pre-training 产出的模型）通常需要 Provisioned Throughput 才能获得稳定的低延迟推理**，但**能否使用 On-Demand 模式因模型提供商而异**——不是所有自定义模型都被禁止 On-Demand（例如 Anthropic 微调完成后可直接 On-Demand 推理），具体以 Bedrock 控制台当前支持情况为准
- **考试陷阱（双重）**：① **"微调能省钱"是常见的误解**——微调本身可能降低单次调用的提示词长度（少写重复指令），RFT 甚至可能让更小的模型达到大模型的效果，但如果目标模型要求 Provisioned Throughput，还需叠加其**持续承诺费用**，**总体成本是否更低取决于调用量是否足够大、足够稳定**；调用量小或波动大的场景，直接用 On-Demand + 更好的提示工程/RAG 反而更经济。② **"自定义模型只能用 Provisioned Throughput"是过度绝对化的错误表述**——正确的通用结论是"自定义模型可以用 Provisioned Throughput 或 On-Demand（取决于模型）"，题目要求判断"适用于 Bedrock 的准确表述"时应选后者。
- 通过 **Import Custom Model** 导入的自有权重模型明确**支持 On-Demand 模式**，无需强制 Provisioned Throughput（与本节开头"五种自定义方式对比"表中的定位一致），是这一规则中最没有争议的一类

### 何时选择 SFT / RFT（而非 RAG/提示工程）

> 完整的三者选择逻辑决策树见 [[大语言模型 - LLM]] 独立笔记，Bedrock 场景下的核心判断：

```
场景分析 → 是否应该 Fine-tuning，选 SFT 还是 RFT
├── "需要模型稳定输出特定格式/语气/风格，且有大量标注问答对" → SFT
├── "任务正确性可自动校验（如代码/规则），但缺少海量人工标注答案" → RFT
├── "需要模型掌握公司私有的、会频繁更新的知识" → 改用 Knowledge Bases（RAG），而非 SFT/RFT
├── "只是临时/低频调用，希望先低成本验证效果" → 先用 Prompt Engineering，不急于微调
├── "调用量大且稳定，愿意承担 Provisioned Throughput 固定成本" → SFT/RFT 更具性价比
└── "已有自己训练好的模型权重，只想托管使用" → Import Custom Model（支持 On-Demand）
```

---

## Model Distillation（模型蒸馏）

> **Model Distillation（模型蒸馏）**于 2025-05 GA，是把一个**更大、更强但更贵/更慢**的"**教师模型（Teacher Model）**"在特定任务上的能力，"压缩"进一个**更小、更快、更便宜**的"**学生模型（Student Model）**"——目标不是让模型学会全新的知识或技能，而是让**学生模型在某项具体任务上逼近教师模型的表现**，同时大幅降低推理延迟和成本。

### 与 SFT/RFT 的核心区别：标准答案的来源（考试重点）

- **蒸馏只需要提供提示（Prompts）**，不需要人工标注答案（区别于 SFT），也不需要设计奖励函数（区别于 RFT）——**教师模型自动生成"标准答案"**，作为训练学生模型的数据
- **考试陷阱**：**"不需要人工标注数据"是蒸馏最大的卖点，但这不代表蒸馏能创造全新能力**——学生模型的效果**上限是教师模型在该任务上的能力**，蒸馏本质是"模仿/压缩"而非"超越"；题目描述"希望用小模型在某个具体任务上达到接近大模型的效果，且不想自己标注训练数据" → **模型蒸馏**；题目描述"希望模型获得教师模型本身都不具备的全新能力" → 蒸馏无法实现这一目标。

### 单一自动化工作流

```
准备提示（Prompts） → 教师模型自动生成响应
                              ↓
                  （可选）数据合成，进一步丰富/优化教师响应
                              ↓
                     用这些"提示-响应"对训练学生模型
                              ↓
                  产出蒸馏后的自定义学生模型（Provisioned Throughput 推理）
```

- 教师模型生成响应阶段按教师模型的 **On-Demand 费率**计费，学生模型的训练阶段按**模型定制（customization）**费率计费
- 支持的教师/学生模型配对由 Bedrock 官方指定（如 **Nova Premier（教师）→ Nova Pro（学生）**、**Llama 3.3 70B（教师）→ Llama 3.2 1B/3B（学生）**），具体配对以 Bedrock 控制台当前支持列表为准

### 性能收益（考试提示）

- AWS 官方数据：蒸馏后的模型相比原教师模型最高可**快 500%**、**便宜 75%**，在 RAG 等场景下准确率损失**低于 2%**
- **典型场景**：Agent 场景中的**函数调用（Function Calling）**——用蒸馏后的小模型准确完成函数调用判断，同时获得远低于大模型的响应延迟和成本

### 何时选择蒸馏（而非 SFT/RFT）

```
场景分析 → 是否应该用 Model Distillation
├── "希望用更小/更便宜的模型逼近大模型在具体任务上的效果" → Distillation
├── "不想自己标注训练数据，也不想设计奖励函数" → Distillation（教师模型自动生成标签）
├── "Agent 场景需要小模型快速、低成本地完成函数调用判断" → Distillation
├── "需要模型具备教师模型本身都不具备的全新能力" → 蒸馏无法实现，考虑其他定制方式或更强的基础模型
└── "已有大量人工标注数据，希望精确控制训练目标" → 改用 SFT，而非依赖教师模型自动生成的标签
```

---

## Bedrock Model Evaluation

> **Model Evaluation** 让你在正式投入生产前，用**同一批测试提示**系统化比较不同基础模型、不同微调/蒸馏版本、甚至外部系统的输出质量，避免仅凭主观感觉或零散测试就决定"该用哪个模型"。可评估的对象不局限于 Bedrock 原生基础模型，也包括 **SFT/RFT/蒸馏产出的自定义模型**、**Import Custom Model 导入的模型**，帮助验证"微调/蒸馏后模型是否真的比原模型更好"。

### 三种评估方式对比（考试重点）

| 方式 | 评判者 | 特点 | 适合场景 |
|------|-------|------|---------|
| **自动评估（Automatic）** | 内置算法 | 全程自动化，**速度最快、成本最低**，但只能衡量算法可量化的维度 | 大批量、客观性强的指标（准确性、抗干扰性、毒性） |
| **LLM 作为评判者（LLM-as-a-Judge）** | 另一个 LLM | 用大模型给回答打分，**质量接近人工评估但成本和耗时远低于人工**（GA 于 2025-03） | 需要"类人"判断力但预算/时间有限的场景 |
| **人工评估（Human-based）** | 真人评估员（自有团队或 **AWS 托管评估团队**） | **最贴合主观/品牌化标准**，但最慢、最贵 | 友好度、语气、品牌调性等高度主观的指标 |

> **考试要点**：**三者按"客观程度"和"成本"排成一条谱系**——纯算法指标（准确性、毒性等）用**自动评估**；需要类似人类判断力但要控制成本/周期用 **LLM-as-a-Judge**；涉及品牌语气、情感倾向等高度主观且业务敏感的标准，仍需**人工评估**。题目描述"希望以远低于人工评估的成本获得接近人工水准的质量判断" → **LLM-as-a-Judge**，而不是默认答案只能二选一（自动 vs 人工）。

### 自动评估的三大内置维度

| 维度 | 衡量内容 |
|------|---------|
| **准确性（Accuracy）** | 输出与参考答案的匹配程度，可结合 ROUGE、METEOR、BERTScore、精确匹配/F1 等传统 NLP 指标 |
| **抗干扰性（Robustness）** | 对输入做**保持语义不变的微小扰动**（大小写变化、键盘输入错别字、数字转文字、随机改变大小写、增删空白）后，输出质量是否稳定 |
| **毒性（Toxicity）** | 检测输出中是否包含有害/攻击性内容 |

- 可使用 Bedrock 提供的**内置精选数据集**（如 Real Toxicity、BOLD、TREX、WikiText-2、Gigaword、BoolQ、Natural Questions、Trivia QA、Women's Ecommerce Clothing Reviews 等，分别对应摘要、问答、分类等不同任务类型），也可以**自带自定义提示数据集**
- **考试陷阱**：**"抗干扰性（Robustness）"考查的是模型面对拼写错误、大小写变化等"噪音输入"时是否还能给出稳定、正确的回答**——题目描述"用户输入常有打字错误，需要评估模型对此的容错能力" → **Robustness** 指标，而非误判为准确性或毒性维度。

### LLM-as-a-Judge 的评判维度

- **质量类指标**：正确性（Correctness）、完整性（Completeness）、专业的风格与语气（Professional Style and Tone）等
- **负责任 AI 类指标**：有害性（Harmfulness）、是否恰当拒绝作答（Answer Refusal）等
- 本质是用一个（通常更强的）LLM 充当"评委"，对候选回答按上述维度打分，替代部分原本需要人工评估员完成的主观判断

### RAG 评估（RAG Evaluation，考试重点）

> **RAG 评估**于 2025-03 GA，专门针对检索增强生成系统——不只评估 LLM 本身，还要评估**整条 RAG 链路**（检索质量 + 生成质量）。

| 评估范围 | 说明 |
|---------|------|
| **仅评估检索（Retrieval Only）** | 只衡量检索出的文档块是否相关，不涉及最终生成的回答 |
| **端到端评估（Retrieval + Generation）** | 同时评估检索质量和最终生成回答的质量 |

- 支持评估基于 **Bedrock Knowledge Bases** 构建的 RAG，也支持评估**自建/外部的 RAG 系统**
- 关键指标包括：**上下文相关性（Context Relevance）**、**答案有据性/落地性（Groundedness，回答是否真的基于检索到的内容，而非模型编造）**、**引用覆盖率（Citation Coverage）**
- **考试陷阱**：**RAG 评估的"有据性（Groundedness）"指标 和 Guardrails 的"基础真实性过滤"是两回事**——RAG 评估是**上线前/离线**的系统化质量测试工具，用于衡量和比较不同 RAG 配置的整体表现；Guardrails 的基础真实性过滤是**运行时**拦截机制，实时阻止/标记不基于知识库的回答。题目描述"需要系统化衡量和对比不同 RAG 配置的整体回答质量" → **RAG 评估**；题目描述"需要在生产环境实时拦截脱离知识库依据的回答" → **Guardrails**，完整对比见 [[#Bedrock Guardrails（内容安全）]] 章节。

### 自带推理结果（Bring Your Own Inference Responses，考试提示）

- 评估不要求被评对象一定"运行在 Bedrock 上"——可以直接提供**已经生成好的推理结果**（如来自自建模型、第三方服务甚至非 Bedrock 平台的输出）进行评估
- **考试要点**：题目描述"需要在同一套评估框架下，对比 Bedrock 模型和外部自建系统的输出质量" → **Bring Your Own Inference Responses**，说明 Model Evaluation 并非只能评估 Bedrock 原生托管的模型。

### 典型应用场景

```
场景分析 → 选择 Model Evaluation 方式
├── "需要大批量、低成本地衡量准确性/抗干扰性/毒性等客观指标" → 自动评估
├── "需要类人判断力但要控制成本和周期" → LLM-as-a-Judge
├── "需要评估语气/品牌调性等高度主观标准" → 人工评估（自有团队或 AWS 托管团队）
├── "用户输入常有拼写错误，需评估模型容错能力" → 自动评估的 Robustness 指标
├── "需要验证微调/蒸馏后的模型是否真的比原模型更好" → Model Evaluation 对比自定义模型 vs 基础模型
├── "需要衡量 RAG 系统整体（检索+生成）质量" → RAG 评估
├── "需要判断回答是否真的基于检索到的文档（防幻觉）" → RAG 评估的 Groundedness 指标（离线测试，区别于 Guardrails 运行时拦截）
└── "需要对比 Bedrock 模型和外部自建系统的输出" → Bring Your Own Inference Responses
```

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

### 模型规模与价格的关系（考试要点）

- **参数量更小的模型，每百万 Token 的输入/输出单价通常更低**；参数量更大、能力更强的模型单价更高——这是 Bedrock 定价表的普遍规律（例如同一系列内，轻量版模型的价格明显低于旗舰版模型）
- **考试要点**：题目问"哪个表述准确" → **"更小的模型比更大的模型更便宜"为真**；反过来"更大的模型比更小的模型更便宜"为假。选型时可结合任务复杂度权衡——简单任务用更小/更便宜的模型即可满足，无需为额外用不上的能力多付费

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
4. **[[Amazon Bedrock AgentCore]] 是架构升级而非改名**：框架无关 + 模型无关的智能体生产基础设施，支持多智能体协作、共享记忆、统一网关，而非 Agents Classic 的简单单智能体框架，完整组件详解见 [[Amazon Bedrock AgentCore]] 独立笔记
5. **Guardrails** = 内容安全过滤（PII、禁止话题、提示注入防御）
6. **On-Demand vs Provisioned Throughput**：On-Demand 按 Token 计费且**无时间承诺**；Provisioned Throughput 是**固定时间费用**（1/6 个月承诺，也有无承诺按小时计费选项）。自定义模型（SFT/RFT/蒸馏/Continued Pre-training）**能否用 On-Demand 因提供商而异**（如 Anthropic 支持、Meta Llama 2 不支持）——**"自定义模型可用于 Provisioned Throughput 或 On-Demand" 是正确的通用结论，"只能用 Provisioned Throughput" 是过度绝对化的错误答案**；Import Custom Model 明确支持 On-Demand
7. **模型规模与价格**：更小的模型单价更低，更大的模型单价更高——"更小模型更便宜"为真，"更大模型更便宜"为假
7. **SFT（监督微调）需要有标签的提示-响应对（JSONL）**；**RFT（强化微调）不需要海量标注答案，而是靠奖励函数打分 + GRPO 强化学习优化**；Continued Pre-training 用无标签原始文本
8. **自定义权重导入**：Bedrock 支持导入自定义模型权重（On-Demand 模式）
9. **灾难性遗忘（Catastrophic Forgetting）**：训练数据过窄/过拟合会导致模型丢失原有通用能力
10. **Fine-tuning 的真实成本要连同 Provisioned Throughput 一起算**：调用量小/波动大时未必比 On-Demand + 提示工程/RAG 更省钱
11. **RFT 全托管完成强化学习流程**：模型生成多个候选回答 → 奖励函数打分 → GRPO 策略优化，平均带来约 66% 的准确率提升，让小模型逼近大模型效果
12. **RFT 支持的模型范围比 SFT 更窄**：并非所有 Bedrock 模型都开放 RFT，需以控制台实际支持列表为准
13. **模型蒸馏（Distillation）只需提示，标准答案由教师模型自动生成**：区别于 SFT（人工标注）和 RFT（奖励函数），是三者中唯一不需要人工设计标签或评分规则的方式
14. **蒸馏的效果上限是教师模型本身**：目标是让学生模型逼近教师表现、大幅降本提速，而非获得教师都不具备的新能力；官方数据最高提速 500%、降本 75%，准确率损失通常低于 2%
15. **Model Evaluation 三种方式按"客观程度/成本"排列**：自动评估（最快最便宜，限于算法可量化指标）→ LLM-as-a-Judge（接近人工质量，成本远低于人工）→ 人工评估（最贴合主观/品牌标准，最慢最贵）
16. **自动评估三大维度**：准确性（Accuracy）、抗干扰性（Robustness，衡量拼写错误等噪音输入下的稳定性）、毒性（Toxicity）
17. **Model Evaluation 可评估自定义模型**：不局限于基础模型，SFT/RFT/蒸馏/导入的模型都可纳入对比，验证微调效果
18. **RAG 评估（2025-03 GA）评估整条链路**：既能评估 Bedrock Knowledge Bases，也能评估自建 RAG 系统；关键指标含上下文相关性、有据性（Groundedness）、引用覆盖率
19. **RAG 评估的 Groundedness ≠ Guardrails 的基础真实性过滤**：前者是离线/上线前的系统化质量测试，后者是生产环境的实时拦截机制
20. **Bring Your Own Inference Responses**：可直接评估已生成好的推理结果，不要求评估对象运行在 Bedrock 上
21. **按分词付费**：Token = 离散信息单位（文本中的词/图像中的像素）；可提升可扩展性
22. **Nova 2 推理模型**：支持扩展思维（三档思维强度）和百万级 Token 上下文
23. **S3 Vectors 是 Knowledge Bases 的新向量存储选项**：成本敏感的大规模场景可比 OpenSearch Serverless 节省最高约 90%
24. **三层架构**：基础设施层 / ML 服务层 / 应用层
25. **AWS Nitro System**：专用硬件强制安全限制，保护 EC2 上的工作负载和数据
26. **Trainium（训练）vs Inferentia（推理）**：专用 AI 芯片，比 GPU 更高性价比
27. **PartyRock**：基于 Bedrock 的免费学习体验平台

### 场景题解题思路

```
场景分析 → 选择 Bedrock 功能
├── "需要接入公司内部文档" → Knowledge Bases（RAG）
├── "需要构建新的多步骤任务自动化代理" → Bedrock AgentCore（而非已停止新客户接入的 Agents Classic）
├── "已有 Agents Classic 部署，需要评估迁移" → AgentCore Import-Agent 工具（转换为 LangGraph）
├── "需要多个智能体协作、共享记忆、任务委派" → AgentCore（Agents Classic 不支持多智能体协作）
├── "防止输出有害内容" → Guardrails
├── "让模型掌握公司特定写作风格/统一话术，且有大量标注问答对" → 监督微调（SFT）
├── "任务正确性可自动校验，但缺少海量人工标注答案" → 强化微调（RFT，奖励函数 + GRPO）
├── "希望用更小/更便宜的模型逼近大模型效果，且不想自己标注数据" → 模型蒸馏（Distillation，教师模型自动生成标签）
├── "让模型消化大量内部文档但不改变输出风格" → Continued Pre-training（而非 SFT/RFT/蒸馏）
├── "微调后模型在其他任务上表现变差" → 灾难性遗忘（Catastrophic Forgetting），需扩充训练数据多样性
├── "已有自训练模型权重，只想托管使用且调用量小" → Import Custom Model（支持 On-Demand，无需 Provisioned Throughput）
├── "快速测试不同模型效果" → Playgrounds
├── "批量评估多个模型/验证微调效果，且要控制成本" → Model Evaluation（自动评估或 LLM-as-a-Judge）
├── "需要评估语气/品牌调性等主观标准" → Model Evaluation 的人工评估
├── "需要衡量 RAG 系统检索+生成的整体质量，或检查回答是否脱离知识库编造" → RAG 评估（Groundedness 指标）
├── "构建复杂 AI 工作流" → Prompt Flows
├── "大规模向量存储，成本敏感的 RAG 场景" → Knowledge Bases + S3 Vectors
├── "需要模型做多步骤推理并控制推理深度/成本" → Amazon Nova 2（思维强度分档）
├── "学习生成式 AI，免费体验" → PartyRock
├── "降低模型训练成本" → AWS Trainium
└── "降低模型推理成本" → AWS Inferentia
```
