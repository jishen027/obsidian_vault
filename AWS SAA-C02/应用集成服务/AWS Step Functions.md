# AWS Step Functions - Serverless 工作流编排

> **AWS Step Functions** 是全托管的 **Serverless 工作流编排服务**，通过可视化的状态机（State Machine）协调多个 [[AWS Lambda]] 函数或其他 AWS 服务，按预定逻辑顺序、并行或条件分支执行，并自动处理重试、错误捕获和状态跟踪。
>
> 相关文档：[[AWS Lambda]] | [[Amazon SWF]] | [[SQS]] | [[SNS]] | [[DynamoDB]] | [[Amazon API Gateway]] | [[ECS]] | [[Amazon EventBridge]]

---

## 核心概念

### Step Functions 定位（考试要点）

| 服务 | 架构模型 | 状态管理 | 典型场景 |
|------|---------|---------|---------|
| **Step Functions** | 无服务器，直接与 AWS 服务集成 | 服务自动跟踪 | 新项目的工作流编排，AWS 当前推荐方案 |
| [[Amazon SWF]] | 需要自行运行**工作者（Worker）** | 服务自动跟踪 | 遗留系统，需要工作者模型的传统应用 |
| 手动串联多个 [[AWS Lambda]] | 应用代码自行编排 | 需自行实现 | 简单的两三步流程，无需可视化编排和内置重试 |

> **考试提示**：AWS 官方推荐 **Step Functions 作为新工作流编排项目的首选**，SWF 仅用于识别遗留架构；题目若强调"多个 Lambda 函数需要按顺序/并行/条件分支执行，且需要可视化跟踪每一步状态和内置重试"→ **Step Functions**。

### 核心价值

- **可视化工作流**：用 **Amazon States Language（ASL，一种 JSON 格式的状态机定义语言）**描述流程，控制台提供图形化的执行流程和实时状态可视化
- **直接服务集成**：状态机可**直接调用超过 200 个 AWS 服务的 API**（如 DynamoDB、SNS、SQS、ECS、[[AWS Glue]]、[[Amazon SageMaker]]），无需为每个集成步骤都单独编写 Lambda 胶水代码
- **内置错误处理**：原生支持**重试（Retry）**和**捕获（Catch）**，无需在应用代码中手写异常处理逻辑
- **自动状态跟踪**：每次执行自动记录完整的输入/输出和状态历史，便于审计和问题排查

---

## 状态类型（考试高频）

| 状态类型 | 说明 |
|---------|------|
| **Task** | 执行一个工作单元，如调用 Lambda 函数或直接调用其他 AWS 服务 API |
| **Choice** | 基于输入数据做**条件分支**，类似 if/else 逻辑 |
| **Parallel** | 并行执行多个分支，全部完成后再继续 |
| **Map** | 对一个数组中的每个元素**并行或串行地重复执行**同一组步骤（如批量处理 S3 中的多个文件） |
| **Wait** | 暂停指定时间或等到某个时间点 |
| **Pass** | 直接传递或修改数据，不执行任何实际工作，常用于调试或数据转换 |
| **Succeed / Fail** | 显式终止工作流并标记成功/失败 |

---

## 工作流类型：Standard vs Express（考试高频）

| 特性 | Standard Workflows | Express Workflows |
|------|--------------------|--------------------|
| **最长执行时间** | **1 年** | **5 分钟** |
| **执行语义** | **恰好一次（Exactly-once）** | 异步：**至少一次（At-least-once）**；同步：至多一次 |
| **执行历史** | 完整保留，可在控制台查看每一步详情 | 仅发送到 CloudWatch Logs，不在控制台单独保留 |
| **计费方式** | 按**状态转换（State Transition）次数**计费 | 按**执行时长和内存**计费（类似 Lambda） |
| **吞吐量** | 较低（每秒数千次启动） | 极高（每秒数十万次启动） |
| **典型场景** | 长时间运行、需要精确审计、订单处理、人工审批流程 | 高吞吐、短时的事件处理、数据流转换、IoT 数据摄取 |

> **考试陷阱**：题目强调"每条数据只能被处理一次，不能重复"→ **Standard Workflows**（Exactly-once）；题目强调"每秒数十万次高吞吐的短时事件处理，可以容忍偶尔重复"→ **Express Workflows**（成本更低、吞吐更高）。

---

## 错误处理机制

| 机制 | 说明 |
|------|------|
| **Retry** | 在状态定义中配置重试策略：捕获特定错误类型、重试次数、退避系数（Backoff Rate），失败后自动按指数退避重试 |
| **Catch** | 捕获无法通过重试解决的错误，将流程**转移到指定的回退状态**（如通知运维、写入死信队列），而非直接使整个执行失败 |

- 两者可组合使用：先尝试若干次 Retry，仍失败则触发 Catch 进入错误处理分支

---

## 人工审批（Human-in-the-Loop）

- Step Functions **没有像 SWF 那样内置的"人工任务"类型**，而是通过 **回调模式（Callback Pattern）** 实现：状态机调用一个集成服务（如 [[SNS]] 发邮件、[[SQS]] 排队）并**暂停执行，等待外部系统通过 `SendTaskSuccess`/`SendTaskFailure` API 携带 Task Token 恢复流程**
- 典型流程：Step Functions 发送带 Task Token 的通知（如邮件審批链接）→ 人工点击审批 → 后端调用 API 携带 Task Token → 状态机恢复执行

> **考试陷阱**：题目要求"工作流需要等待人工审批才能继续"——**Step Functions 通过 `.waitForTaskToken` 回调模式实现**，而不是像 SWF 那样有原生的 Human Task 类型；若题目明确是遗留系统且已经在用 SWF 的工作者模型，才考虑保留 SWF。

---

## 触发方式

| 触发来源 | 说明 |
|---------|------|
| **[[Amazon API Gateway]]** | API 请求直接触发状态机执行，构建 Serverless API 编排后端 |
| **[[Amazon EventBridge]]（CloudWatch Events）** | 定时任务或事件驱动触发工作流 |
| **[[AWS Lambda]] / SDK 调用** | 应用代码通过 SDK 调用 `StartExecution` 启动工作流 |
| **[[SQS]] 集成** | 通过 Lambda 轮询 SQS 或直接的服务集成触发批处理工作流 |

---

## 与 Amazon SWF 对比（考试重点）

| 特性 | Step Functions | [[Amazon SWF]] |
|------|-----------------|-----------------|
| **架构模型** | 无服务器，直接与 AWS 服务集成 | 需要自行运行工作者（Worker），需管理计算资源 |
| **可视化** | 控制台原生可视化工作流和执行历史 | 无原生可视化，需自行构建监控 |
| **人工审批** | 回调模式（Task Token） | 原生 Human Task 支持 |
| **执行时间** | Standard 最长 1 年，Express 最长 5 分钟 | 最长 1 年 |
| **AWS 推荐程度** | **当前推荐的新项目首选** | 遗留系统维护，不推荐新项目采用 |
| **计费模型** | 按状态转换次数或执行时长计费 | 按活动/决策任务数量 + 工作者运行成本计费 |

---

## 典型应用场景

| 场景 | 推荐配置 |
|------|---------|
| **多步骤订单处理（库存检查、支付、发货通知）** | Step Functions（Standard）+ Lambda/DynamoDB/SNS |
| **数据处理流水线（ETL、批量转换）** | Step Functions + Map 状态并行处理 S3 中的多个对象 |
| **高吞吐、短时的实时数据摄取** | Step Functions（Express Workflows） |
| **需要人工审批的审批流程（如报销、贷款）** | Step Functions + `.waitForTaskToken` 回调模式 |
| **微服务编排（多个 Lambda 函数按顺序/并行调用）** | Step Functions + Choice/Parallel 状态 |
| **遗留系统，已采用工作者模型** | 维持 [[Amazon SWF]]，非新项目不建议迁移 |

---

## 考试重点总结

### SAA-C02 高频考点

1. **Step Functions 是 AWS 当前推荐的工作流编排服务**，SWF 是遗留方案，仅用于识别老系统
2. **状态机用 ASL（JSON）定义**：控制台提供可视化流程图和执行历史
3. **直接服务集成，无需胶水 Lambda**：可直接调用 200+ AWS 服务 API
4. **Standard vs Express**：前者执行时间长（1年）+ 恰好一次 + 完整审计；后者执行时间短（5分钟）+ 高吞吐 + 按时长计费
5. **内置 Retry + Catch**：无需在应用代码中手写重试/异常处理逻辑
6. **Map 状态用于批量并行处理**：对数组中每个元素并行执行相同步骤
7. **人工审批走回调模式（Task Token）**：不是原生 Human Task（那是 SWF 的特色），而是 `waitForTaskToken` + 外部系统回调
8. **恰好一次 vs 至少一次**：Standard 保证 Exactly-once，Express 异步执行是 At-least-once
9. **可被 API Gateway/[[Amazon EventBridge]]/SDK 触发**：多种方式启动执行
10. **执行历史完整保留（Standard）**：便于审计和问题排查，Express 执行历史仅存在 CloudWatch Logs

### 场景题解题思路

```
场景分析 → 选择工作流方案
├── "多个 Lambda 需按顺序/并行/条件执行，需要可视化跟踪" → Step Functions
├── "每条数据必须恰好处理一次，长时间运行" → Step Functions Standard Workflows
├── "每秒数十万次高吞吐、短时事件处理，可容忍偶尔重复" → Step Functions Express Workflows
├── "需要对数组/批量对象并行处理相同逻辑" → Map 状态
├── "步骤失败需要自动重试，多次失败后转入人工处理分支" → Retry + Catch
├── "工作流需要等待人工审批才能继续" → .waitForTaskToken 回调模式
├── "遗留系统已经在用 Worker 模型的工作流" → 维持 Amazon SWF
└── "简单两三步流程，无需可视化和内置重试" → 直接在应用代码中串联 Lambda（无需引入 Step Functions）
```

---

## 最佳实践

1. **新项目优先选择 Step Functions**：不要在新架构中引入已被 AWS 事实上取代的 SWF
2. **优先使用直接服务集成**：能直接调用 DynamoDB/SNS/SQS 等服务 API 时，不必额外包一层 Lambda
3. **高吞吐短时任务选 Express，长时间/强一致性任务选 Standard**：按执行时长和一致性需求选型，而非默认都用 Standard
4. **为关键 Task 配置 Retry + Catch**：避免瞬时故障导致整个工作流失败
5. **用 Map 状态处理批量数据**：避免在单个 Lambda 中用循环处理大量条目，既受限于 15 分钟超时也不便观测单条状态
6. **人工审批场景使用回调模式**：结合 SNS/SQS 发送通知，外部系统回调 Task Token 恢复执行
7. **善用执行历史做审计和调试**：Standard Workflows 的完整执行记录是排查生产问题的第一手资料
