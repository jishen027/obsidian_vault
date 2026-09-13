# Amazon Bedrock AgentCore

> **Amazon Bedrock AgentCore**（2025-07 预览，2025-10 GA）是 AWS 面向生产环境的**智能体基础设施平台**——核心定位是**框架无关（Framework-Agnostic）**和**模型无关（Model-Agnostic）**：无论你用 **Strands Agents**（AWS 自研开源智能体 SDK）、**LangGraph**、**CrewAI**、**LlamaIndex**、**LangChain** 还是完全自定义的代码构建智能体逻辑，也无论调用的是哪个大语言模型（不局限于 Bedrock 托管模型），AgentCore 都提供**安全、可扩展、可观测**的托管运行环境，让你把本地写好的智能体代码**原样部署到生产**，而无需重写或迁移到某个特定框架。它**不是** Bedrock Agents 的更名，而是从"单智能体框架"到"框架无关的智能体生产基础设施"的彻底重新架构，完整的迁移背景见 [[Amazon Bedrock]] 独立笔记中的《Bedrock Agents：Classic 与 AgentCore》章节。
>
> 相关文档：[[Amazon Bedrock]] | [[大语言模型 - LLM]] | [[提示工程 - Prompt Engineering]]

---

## 核心定位：框架无关 + 模型无关（考试重点）

- **AgentCore 本身不是一个"写智能体逻辑"的框架**——真正定义智能体如何推理、规划、调用工具的代码，仍然由 **Strands Agents / LangGraph / CrewAI / LlamaIndex / LangChain** 等框架（或完全自定义代码）编写；AgentCore 提供的是这些智能体代码**运行、伸缩、安全、可观测**所需要的**生产基础设施**
- **模型无关**：AgentCore 不要求必须调用 Bedrock 托管的基础模型，可以搭配任意 LLM 使用
- **考试陷阱**：**"AgentCore 是不是要求必须用 LangGraph 或必须调用 Bedrock 模型" 是常见误解**——题目描述"团队已经用 LangGraph/CrewAI 构建了智能体，希望零改造部署到生产环境" → **AgentCore Runtime 可直接托管现有代码，无需框架迁移**；题目描述"AgentCore 是否是又一个智能体编排框架，与 LangGraph 是竞品关系" → **不是**，AgentCore 是运行这些框架产出代码的**基础设施层**，两者是互补而非竞争关系。

---

## 模块化组件："按需拼装（À La Carte）"（考试重点）

> AgentCore 由多个**独立、可分别启用**的服务组成——可以只用 Runtime 先把智能体跑起来，后续再按需引入 Gateway 接入更多工具，或引入 Identity 支持用户级别的 OAuth 授权，**不要求一次性采用全部组件**。

| 组件                              | 说明                                                                                                                                                                                  |
| ------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Runtime（运行时）**                | **安全、无服务器**的智能体/工具托管环境，处理会话管理、生命周期管理和自动伸缩，可将本地智能体代码"原样"转为云原生部署                                                                                                                      |
| **Gateway（网关）**                 | 把已有的 API/服务转换为符合 **MCP（Model Context Protocol）** 标准的工具，简化工具集成和发现，供智能体统一调用，也负责路由多智能体间的协作                                                                                             |
| **Memory（记忆）**                  | 提供**短期记忆**（单次会话内的上下文）和**长期记忆**（跨会话/跨智能体持久化），**零基础设施管理**                                                                                                                             |
| **Identity（身份管理）**              | 为每个智能体分配**独立身份**，并可与外部身份提供商（**Okta、Entra ID、Cognito** 等）集成，支持用户级别的 OAuth 授权                                                                                                         |
| **Policy Engine（策略引擎）**         | 精确控制智能体可执行的操作范围（2026-03 GA）                                                                                                                                                         |
| **Code Interpreter**            | 智能体可执行代码完成计算/数据处理类任务                                                                                                                                                                |
| **Browser Tool**                | 智能体可操作浏览器完成网页交互类任务                                                                                                                                                                  |
| **Harness（托管智能体编排引擎）**          | **托管的智能体编排循环**——用配置（模型、系统提示、工具、记忆、限制）声明智能体，而不必自己编写推理-调用工具-再推理的编排代码，是 AgentCore 中**最接近 Bedrock Agents Classic 托管体验**的组件（2026-06 GA），完整机制见下方[[#AgentCore Harness：托管编排引擎详解（考试重点）]]独立章节 |
| **Agent Registry / 多智能体编排**     | 管理和编排包含多个协作智能体的复杂流水线                                                                                                                                                                |
| **Evaluations & Observability** | 智能体评估和可观测性工具，可结合 [[Amazon Bedrock]] 笔记中的 Model Evaluation 理念对智能体整体表现做评估                                                                                                             |

> **考试要点**：**"按需拼装"是 AgentCore 组件设计的核心理念**——题目描述"团队只需要一个安全的托管运行环境，暂时不需要复杂的多智能体编排或身份管理" → 只启用 **Runtime** 即可，无需强制引入全部组件；后续需求增长时再逐步添加 Gateway/Identity/Memory 等其他组件。

---

## AgentCore Harness：托管编排引擎详解（考试重点）

> **Harness**（2026-06 GA）解决的是"**智能体的推理-决策循环该由谁来写**"这个问题——它和 **Runtime** 解决的是同一个大问题（如何把智能体安全地跑在生产环境）的**不同层面**，两者不是同一个东西，也不是互斥选项，而是**分层关系**：Harness 是运行在 Runtime **之上/之内**的一层**托管抽象**。

### Harness 的核心机制

- **编排循环由 AWS 托管**：Harness 底层由 **Strands Agents**（AWS 自研开源智能体 SDK）驱动，你**不需要自己编写**"接收输入 → 模型推理 → 判断是否调用工具 → 执行工具 → 把结果喂回模型 → 生成最终回答"这一整套编排逻辑代码
- **配置驱动，而非代码驱动**：你只需要以**配置**的形式声明智能体——使用哪个模型、系统提示词是什么、可调用哪些工具、需要什么样的记忆能力、有哪些限制条件——Harness 负责按这份配置**运行**编排循环
- **改配置 ≠ 重新部署**：绝大多数常见变更（换一个模型、给智能体加一个新工具）在 Harness 模式下只是**修改一个配置字段**，而不需要像"自己写代码"的模式那样重新打包、重新部署

### Harness vs Runtime（考试高频陷阱）

| 维度 | AgentCore Harness | AgentCore Runtime（自带代码模式） |
|------|----------------------|-------------------------------------|
| **编排循环由谁写** | **AWS 托管**（Strands Agents 驱动），你只需配置 | **你自己写**——用任意框架（或不用框架）编写完整的智能体逻辑 |
| **接入方式** | 通过配置声明智能体，或用 **AgentCore CLI** / **boto3** 等 AWS SDK 直接调用 | 用 AgentCore SDK 的 **`BedrockAgentCoreApp`** 入口包装代码，打包为 **ARM64 容器**并推送到 **Amazon ECR**，再部署到 Runtime |
| **修改成本** | 大多数变更是**配置字段修改**，无需重新部署 | 修改编排逻辑需要**重新构建镜像并重新部署** |
| **灵活性** | 较低——受限于 Harness 预设的编排模式和可配置项 | **完全自定义**——可实现任意复杂的编排逻辑、接入任意框架 |
| **底层关系** | **运行在 Runtime 之上/之内**的托管抽象层，其操作在 CloudTrail 中记录为 `AWS::BedrockAgentCore::Runtime` | 更底层的**通用无服务器托管基础设施**（隔离、伸缩、会话、鉴权、可观测性管道） |
| **心智模型类比** | 接近 **Bedrock Agents Classic**（配置一个 Agent + 工具，而非自己写编排代码） | 接近"把自己已经写好的应用容器化后部署到一个托管平台" |

> **考试陷阱**：**"Harness 和 Runtime 是二选一的竞品"是常见误解**——题目描述"团队已经用 LangGraph 写好了完整的智能体编排逻辑，只想把它部署到生产环境，不想改写成配置" → **Runtime**（自带代码模式），用 `BedrockAgentCoreApp` 包装现有代码即可；题目描述"团队没有时间/意愿自己编写智能体的推理-工具调用循环，只想通过配置模型、提示词、工具列表就快速拿到一个能跑的智能体" → **Harness**，由 AWS 托管的 Strands Agents 驱动编排循环，你只需维护配置；题目描述"AgentCore Harness 是不是完全独立于 Runtime 的另一套基础设施" → **不是**，Harness 是构建在 Runtime 之上的托管层，两者是分层关系而非并列的竞品。

### 为什么说 Harness 最接近 Bedrock Agents Classic 的体验

- **Bedrock Agents Classic** 的核心心智模型就是"配置一个 Agent（选模型、写指令）+ 配置 Action Groups（工具），由 AWS 托管整个编排循环"——你从不需要自己写"模型何时调用工具"这类底层编排代码
- **Harness 延续了这一"配置优先"的心智模型**，但底层换成了更现代的 **Strands Agents** 编排引擎，并且天然与 AgentCore 的"按需拼装"体系（Gateway、Memory、Identity 等）集成，而不是 Agents Classic 那种相对封闭的架构
- **考试要点**：题目描述"已经习惯 Agents Classic 那种'配置而非编码'的使用方式，希望在 AgentCore 中找到类似体验" → **Harness**，这是 AgentCore 组件中专门为这类需求设计的组件。

---

## AgentCore CLI（考试提示）

- **`@aws/agentcore`** 命令行工具是当前**官方推荐**的创建、开发、部署 AgentCore 智能体的方式
- 支持广泛的框架集成（Strands、LangGraph、LangChain、Google ADK、OpenAI Agents 以及完全自定义代码）
- 提供**本地开发热重载（Hot Reload）**、**内置评估**、**网关管理**等能力，简化从本地开发到云端部署的流程

---

## 与 Bedrock Agents Classic 的关系（考试高频）

| 维度 | Bedrock Agents Classic | Amazon Bedrock AgentCore |
|------|--------------------------|----------------------------|
| **服务状态** | 2026-07-30 起停止向新客户开放，进入维护模式，模型目录冻结 | 当前 AWS 力推的智能体生产路径 |
| **框架约束** | 仅限 Bedrock 原生的 Agent/Action Groups 编排模型 | **框架无关**——兼容 Strands/LangGraph/CrewAI/LlamaIndex/LangChain 等 |
| **模型约束** | 依赖 Bedrock 托管模型 | **模型无关**——可搭配任意 LLM |
| **智能体协作能力** | 单智能体框架 + 基础工具调用（Action Groups + Lambda） | 原生支持**多智能体系统**，智能体间可共享记忆、委派任务、通过统一网关协作 |
| **迁移路径** | 存量客户可用 **Import-Agent 工具**转换为基于 LangGraph 的实现，典型迁移周期约 2-4 周 | — |

> **考试陷阱**：题目若问"Bedrock Agents 和 AgentCore 是什么关系" → **AgentCore 并非 Agents 的简单改名，而是从"单一 Bedrock 原生框架"到"框架无关的生产基础设施"的架构升级**；题目强调"新建智能体应用" → 优先考虑 **AgentCore**（Agents Classic 已停止新客户接入）；题目描述"已有 Agents Classic 部署，评估迁移" → 使用 **Import-Agent 工具**转换为 LangGraph 实现。完整的 Bedrock Agents 服务状态变化背景见 [[Amazon Bedrock]] 独立笔记。

---

## 典型应用场景

| 场景 | 推荐配置 |
|------|---------|
| **已用 LangGraph/CrewAI/Strands 构建智能体，希望零改造部署到生产** | AgentCore Runtime 直接托管现有代码 |
| **不想自己写编排循环代码，只想配置模型/提示词/工具就跑起一个智能体** | **AgentCore Harness**（Strands Agents 托管驱动，配置优先） |
| **已习惯 Agents Classic"配置而非编码"的体验，希望在 AgentCore 中延续** | AgentCore Harness |
| **需要把已有 API/服务快速转换为智能体可调用的工具** | AgentCore Gateway（自动转换为 MCP 兼容工具） |
| **智能体需要记住跨会话的用户偏好/历史上下文** | AgentCore Memory（短期 + 长期记忆） |
| **需要为每个智能体分配独立身份，并支持用户级 OAuth 授权** | AgentCore Identity（集成 Okta/Entra ID/Cognito） |
| **只需要一个安全托管环境，暂不需要复杂编排** | 仅启用 AgentCore Runtime，按需再添加其他组件 |
| **需要精确控制智能体可执行的操作范围** | AgentCore Policy Engine |
| **智能体需要执行代码/操作浏览器完成任务** | Code Interpreter / Browser Tool |
| **需要构建和编排多个协作智能体的复杂流水线** | Agent Registry / 多智能体编排 |
| **已有 Bedrock Agents Classic 部署，需要评估迁移** | Import-Agent 工具（转换为 LangGraph 实现） |
| **从本地开发到云端部署的完整工作流** | AgentCore CLI（`@aws/agentcore`），支持热重载和内置评估 |

---

## 考试重点总结

### AIF-C01 高频考点

1. **AgentCore 核心定位是框架无关 + 模型无关的智能体生产基础设施**：不是又一个编排框架，而是运行这些框架产出代码的托管环境
2. **兼容主流智能体框架**：Strands Agents（AWS 自研）、LangGraph、CrewAI、LlamaIndex、LangChain 等，也支持完全自定义代码
3. **组件按需拼装（À La Carte）**：可以只用 Runtime，后续再逐步引入 Gateway/Memory/Identity 等其他组件
4. **Runtime 是安全无服务器的托管环境**：处理会话管理、生命周期和自动伸缩
5. **Gateway 把 API 转换为 MCP 兼容工具**：简化工具集成和多智能体间的路由协作
6. **Memory 提供短期 + 长期记忆，零基础设施管理**：跨会话/跨智能体持久化上下文
7. **Identity 为每个智能体分配独立身份**：可集成外部 IdP（Okta/Entra ID/Cognito）支持 OAuth
8. **AgentCore CLI（`@aws/agentcore`）是官方推荐的开发部署方式**：支持本地热重载、内置评估
9. **Harness 是运行在 Runtime 之上的托管编排层（2026-06 GA）**：由 Strands Agents 驱动编排循环，你只需配置模型/提示词/工具，无需自己编写推理-调用工具的循环代码
10. **Harness vs Runtime 的核心区别是"编排循环由谁写"**：Harness 是 AWS 托管、配置驱动；Runtime（自带代码模式）是你自己写完整编排逻辑，用 `BedrockAgentCoreApp` 包装后打包为 ARM64 容器部署
11. **Harness 是 AgentCore 中最接近 Bedrock Agents Classic 体验的组件**：延续"配置而非编码"的心智模型，但底层引擎和集成能力都是现代化的
12. **AgentCore 并非 Bedrock Agents 的改名**：是从单一 Bedrock 原生框架到框架无关生产基础设施的彻底重构
13. **Bedrock Agents Classic 存量迁移用 Import-Agent 工具**：转换为基于 LangGraph 的实现，典型周期 2-4 周

### 场景题解题思路

```
场景分析 → 判断 Amazon Bedrock AgentCore 相关配置
├── "已用 LangGraph/CrewAI/Strands 构建智能体，需零改造部署到生产" → AgentCore Runtime（自带代码模式）
├── "不想自己写编排循环，只想配置模型/提示词/工具快速跑起智能体" → AgentCore Harness（Strands Agents 托管驱动）
├── "习惯 Agents Classic 配置而非编码的体验" → AgentCore Harness
├── "需要把已有 API 转换为智能体可调用的工具" → AgentCore Gateway（MCP 兼容）
├── "智能体需要跨会话记住上下文/用户偏好" → AgentCore Memory
├── "需要为智能体分配独立身份并支持用户级 OAuth" → AgentCore Identity
├── "只需要基础的安全托管环境，暂不需要复杂功能" → 仅启用 Runtime，按需扩展
├── "需要精确限制智能体可执行的操作" → Policy Engine
├── "需要构建多个协作智能体的复杂流水线" → Agent Registry / 多智能体编排
├── "已有 Agents Classic 部署，需要评估迁移" → Import-Agent 工具（转换为 LangGraph）
└── "怀疑 AgentCore 是否要求特定框架/特定模型" → 不要求，AgentCore 框架无关且模型无关
```

---

## 最佳实践

1. **新建智能体应用优先评估 AgentCore，而非已停止新客户接入的 Agents Classic**：避免基于即将维护冻结的平台设计新架构
2. **已有基于主流框架的智能体代码无需重写即可迁移**：优先尝试 AgentCore Runtime 直接托管，而非推倒重来
3. **按实际需求逐步引入组件，而非一次性采用全部能力**：从 Runtime 起步，按需扩展 Gateway/Memory/Identity 等
4. **团队没有自建编排循环的意愿/资源时优先评估 Harness**：配置驱动能显著缩短从想法到可用智能体的周期，待需求超出 Harness 能力边界时再迁移到 Runtime 自带代码模式
5. **涉及用户级授权的场景提前规划 Identity 与现有身份提供商的集成**：避免上线后才发现身份体系割裂
6. **利用 AgentCore CLI 的本地热重载能力加速开发迭代**：减少每次修改都要重新部署到云端验证的等待时间
7. **多智能体协作场景善用 Agent Registry 统一管理编排逻辑**：避免协作关系散落在各个智能体代码中难以维护
8. **存量 Agents Classic 客户尽早规划迁移窗口**：结合 Import-Agent 工具评估 2-4 周的典型迁移周期，避免临近维护模式截止日期才仓促行动