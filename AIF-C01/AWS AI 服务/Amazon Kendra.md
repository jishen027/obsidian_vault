# Amazon Kendra - 企业级智能搜索服务

> **Amazon Kendra** 是全托管的企业级机器学习搜索引擎，使用**语义理解**（而非简单关键词匹配）跨多个企业数据源（SharePoint、Salesforce、S3、数据库等）提供自然语言问答和智能文档检索。**Amazon Kendra 已于 2026 年 6 月 30 日进入维护模式（不再开发新功能），并于 2026 年 7 月 30 日起停止向新客户开放**；AWS 官方建议新工作负载迁移到 **[[Amazon Bedrock]] Knowledge Bases**，其 GenAI Index 也已支持作为 Bedrock Knowledge Bases 的托管检索器（Managed Retriever）以承接过渡。
>
> 相关文档：[[Amazon Bedrock]] | [[Amazon OpenSearch]] | [[Amazon Comprehend]] | [[Amazon Lex]] | [[S3]] | [[IAM]] | [[KMS]]

---

## 核心概念

### 为什么需要 Kendra

- **传统企业搜索的局限**：基于关键词匹配的搜索（如简单的全文索引）无法理解用户问题的语义，检索结果往往是一堆相关文档列表，用户仍需自行翻找答案
- **Kendra 的核心价值**：使用**自然语言理解（NLU）**技术理解问题含义而非仅匹配关键词，能够**直接返回答案或高相关性的文档片段**，而非简单的文档列表
- **典型问题形式**："How do I connect my Echo Plus to my network?" 这类自然语言问题，Kendra 能基于语义理解返回精确答案，而非依赖用户输入恰好匹配的关键词

### Kendra 在 AWS 搜索/检索服务中的定位（考试要点）

| 服务 | 技术路线 | 输出形式 | 典型场景 |
|------|---------|---------|---------|
| **Kendra** | 传统信息检索 + ML 语义理解 | 相关文档/段落（+ 直接答案） | 企业内部文档搜索、自然语言问答（无需 LLM 生成） |
| [[Amazon Bedrock]] Knowledge Bases | 向量嵌入 + LLM（RAG） | LLM 生成的自然语言答案 | 生成式 AI 应用的检索增强生成 |
| [[Amazon OpenSearch]] | 倒排索引 + 向量搜索 | 相关性排序的搜索结果 | 通用全文检索、日志分析，需自行构建应用层语义理解逻辑 |

> **考试陷阱**：**Kendra 与 Bedrock Knowledge Bases 都能做"企业知识库问答"，但技术路线和输出形式不同**——Kendra 基于传统 ML 语义理解，返回匹配的文档/段落；Bedrock Knowledge Bases 基于向量嵌入检索 + LLM 生成，返回**由大语言模型生成的自然语言答案**。题目强调"生成式 AI 应用、需要 LLM 生成的对话式回答"→ Bedrock Knowledge Bases；强调"企业文档检索、返回相关文档而非生成式回答"→ Kendra（但需注意其维护模式状态，新项目应优先评估 Bedrock Knowledge Bases）。

---

## 核心能力

### 三种响应类型

| 响应类型 | 说明 |
|---------|------|
| **FAQ 匹配** | 将用户问题与预先配置的常见问题（FAQ）库匹配，直接返回标准答案 |
| **阅读理解式答案提取** | 从文档中提取能够直接回答问题的**建议答案**片段，而非返回整篇文档 |
| **文档排序** | 按相关性对匹配的文档进行排序，返回最相关的文档/段落列表 |

### 数据源连接器（Connector）

- 提供 **30+ 原生连接器**，覆盖常见企业系统：SharePoint、Salesforce、ServiceNow、Slack、S3、关系数据库等
- 使用连接器只需在 Kendra 索引中添加数据源并选择连接器类型，无需自行开发数据摄取管道
- **访问控制过滤**：Kendra 可基于用户权限过滤搜索结果，确保用户只能搜索到自己有权访问的内容

### 相关性调优（Relevance Tuning）

- 可在**索引级别**或**查询级别**调整特定字段/属性对搜索相关性的影响权重
- 当查询词匹配某个被"加权"的字段时，对应文档在结果中获得**排名提升（Boost）**，提升幅度可自定义
- 查询级别的调优可覆盖索引级别的默认配置，便于针对不同应用场景灵活调整

---

## 索引版本（Index Editions）

| 版本 | 说明 |
|------|------|
| **Developer Edition** | 面向概念验证（PoC），容量较小（约 1 万份文档/3 GB 文本），成本较低 |
| **Enterprise Edition** | 面向生产环境，支持更大容量（约 10 万份文档/30 GB 文本）和高可用性 |
| **GenAI Enterprise Edition（GenAI Index）** | 新一代索引，采用最新的信息检索和语义模型技术，提供更高的开箱即用搜索准确率，并可作为 **Bedrock Knowledge Bases 的托管检索器**，是 Kendra 向生成式 AI 生态迁移过渡的关键能力 |

---

## 与生成式 AI 的关系（考试要点）

- **GenAI Index** 是 Kendra 在维护模式前推出的最后一批重要能力，设计目标是让 Kendra 的语义检索能力可以**在 [[Amazon Bedrock]] Knowledge Bases、Amazon Q Business 等生成式 AI 服务之间迁移复用**
- 这意味着已有 Kendra 索引投入的客户，可以通过 GenAI Index 平滑过渡到以 Bedrock 为核心的 RAG 架构，而不必推倒重来
- **考试提示**：题目若涉及"企业搜索"或"文档问答"场景，需要结合当前服务状态判断——Kendra 仍可能出现在题目选项中（历史考纲/现有部署），但**新架构设计的推荐答案应优先考虑 Bedrock Knowledge Bases**

---

## 安全性

| 机制 | 说明 |
|------|------|
| **IAM 策略** | 控制哪些身份可以管理 Kendra 索引、执行搜索查询 |
| **基于用户权限的结果过滤** | 结合身份提供商（如 SSO/AD）的用户组信息，确保搜索结果符合用户的原始数据访问权限 |
| **静态加密** | 索引数据可使用 [[KMS]] 密钥加密 |
| **传输加密** | API 调用默认通过 HTTPS/TLS |
| **VPC 部署** | 支持通过接口终端节点在 VPC 内私有访问 |

---

## 典型应用场景

| 场景 | 推荐配置 |
|------|---------|
| **企业内部文档/知识库自然语言搜索** | Kendra 索引 + 30+ 连接器接入 SharePoint/Salesforce 等系统 |
| **客服/支持团队的内部知识检索** | Kendra FAQ 匹配 + 阅读理解式答案提取 |
| **需要按用户权限过滤搜索结果的合规场景** | Kendra 基于用户权限的访问控制过滤 |
| **概念验证/小规模试点** | Kendra Developer Edition |
| **生产级、大容量、高可用企业搜索** | Kendra Enterprise Edition（需评估维护模式带来的长期风险） |
| **新建的生成式 AI 问答/RAG 应用** | 优先评估 [[Amazon Bedrock]] Knowledge Bases，而非新建 Kendra 索引 |
| **已有 Kendra 部署，希望平滑过渡到生成式 AI 架构** | 升级到 GenAI Index，作为 Bedrock Knowledge Bases 的托管检索器 |

---

## 考试重点总结

### 高频考点

1. **核心定位**：企业级语义搜索引擎，使用 ML/NLU 理解自然语言问题，而非简单关键词匹配
2. **判断依据**：题目强调"企业文档检索、语义搜索、自然语言问答（非生成式）"→ Kendra；强调"LLM 生成的对话式回答、RAG 应用"→ Bedrock Knowledge Bases
3. **三种响应类型**：FAQ 匹配、阅读理解式答案提取、文档排序，分别对应不同的问答需求
4. **30+ 原生连接器**：覆盖 SharePoint、Salesforce、ServiceNow、S3、数据库等主流企业系统，无需自建数据摄取管道
5. **基于用户权限过滤结果**：搜索结果自动遵循用户原有的数据访问权限，是合规场景的关键能力
6. **相关性调优**：索引级/查询级两个层次调整字段权重，提升关键字段的搜索排名
7. **三种索引版本**：Developer（PoC）、Enterprise（生产）、GenAI Enterprise（新一代语义模型 + 可作 Bedrock 托管检索器）
8. **服务状态变化（重要）**：**2026-06-30 起进入维护模式，2026-07-30 起停止新客户接入**，AWS 推荐新项目迁移至 Bedrock Knowledge Bases
9. **GenAI Index 是过渡桥梁**：让已有 Kendra 投入可迁移复用到 Bedrock Knowledge Bases/Amazon Q Business 等生成式 AI 服务
10. **Kendra vs OpenSearch**：Kendra 内置语义理解和问答能力开箱即用；OpenSearch 是更底层的搜索引擎，需自行构建语义理解层

### 场景题解题思路

```
场景分析 → 判断是否用 Kendra
├── "企业内部文档需要自然语言语义搜索" → Amazon Kendra
├── "需要基于知识库直接生成对话式回答（RAG）" → 改用 Bedrock Knowledge Bases（而非 Kendra）
├── "搜索结果需要遵循用户原有的数据访问权限" → Kendra 基于用户权限的过滤
├── "需要接入 SharePoint/Salesforce 等多个企业系统做统一搜索" → Kendra 原生连接器
├── "特定字段的匹配需要在搜索结果中获得更高排名" → Kendra 相关性调优
├── "新建的生成式 AI 问答应用" → 优先评估 Bedrock Knowledge Bases，而非新建 Kendra
└── "已有 Kendra 索引，希望平滑过渡到生成式 AI 架构" → 升级至 GenAI Index
```

---

## 最佳实践

1. **新项目优先评估 Bedrock Knowledge Bases**：Kendra 已进入维护模式且停止新客户接入，新架构应避免依赖受限产品
2. **已有 Kendra 部署评估迁移路径**：利用 GenAI Index 作为向 Bedrock 生态过渡的桥梁，降低历史投入的沉没成本
3. **善用原生连接器而非自建数据摄取**：覆盖主流企业系统的连接器可大幅缩短集成周期
4. **合规场景务必验证权限过滤生效**：确保搜索结果的访问控制与源系统权限保持一致
5. **通过相关性调优优化关键业务字段的搜索排名**：而非依赖默认相关性算法，尤其是有明确业务优先级的字段
6. **PoC 阶段使用 Developer Edition 控制成本**：验证效果后再评估是否升级到生产级 Enterprise Edition
7. **静态数据启用 KMS 加密**：涉及企业内部敏感文档的索引数据应加密存储
