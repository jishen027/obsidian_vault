# Amazon AppFlow - SaaS 数据集成服务

> **Amazon AppFlow** 是全托管的**集成服务**，通过**预置连接器（Connector）**在 **SaaS 应用**（Salesforce、ServiceNow、Slack、Zendesk、SAP、Google Analytics 等）与 **AWS 服务**（[[S3]]、[[Redshift]] 等）之间**双向**传输数据，传输过程中支持**字段映射、过滤、脱敏、校验**等数据处理，全程**无需编写自定义集成代码**。它解决的是"把 SaaS 里的业务数据搬到 AWS 做分析/处理，或把 AWS 处理结果写回 SaaS"这类问题，与负责"路由离散事件通知"的 [[Amazon EventBridge]]、负责"文件/对象传输"的 [[AWS DataSync]]、负责"数据库到数据库迁移"的 [[Database Migration Service]] 分属完全不同的数据集成层次。
>
> 相关文档：[[Amazon EventBridge]] | [[AWS Glue]] | [[AWS DataSync]] | [[Database Migration Service]] | [[S3]] | [[Redshift]] | [[KMS]] | [[IAM]]

---

## 核心概念

### 流（Flow）：AppFlow 的核心单元

一个 **Flow** 完整定义了一次数据集成任务：

| 要素 | 说明 |
|------|------|
| **源连接器（Source Connector）** | 数据的来源——可以是 SaaS 应用（如 Salesforce）或 AWS 服务 |
| **目标连接器（Destination Connector）** | 数据的去向——同样可以是 SaaS 应用或 AWS 服务 |
| **字段映射（Mapping）** | 定义源字段如何对应到目标字段 |
| **过滤条件（Filter）** | 只传输满足特定条件的记录（如"仅同步过去 24 小时内更新的客户记录"） |
| **数据转换** | 传输过程中对数据做**校验（Validation）**、**脱敏（Masking）**、**聚合（Aggregation）**等处理 |

### 双向数据流（考试提示）

- 数据流向**不局限于"SaaS → AWS"**——同样支持"**AWS → SaaS**"，例如把机器学习模型在 [[S3]]/[[Redshift]] 中处理完的结果**写回** Salesforce，供业务人员在原有工具中直接使用
- **考试要点**：题目描述"需要把 AWS 侧分析处理后的结果同步回 SaaS 应用" → AppFlow 的**双向**能力天然支持，不需要额外开发反向同步的自定义代码。

### 触发方式

| 触发类型 | 说明 |
|---------|------|
| **按需（On-Demand）** | 手动触发一次性执行 |
| **计划（Scheduled）** | 按预设的周期性计划（类似 Cron）自动执行 |
| **事件驱动（Event-Driven）** | 由源 SaaS 应用内部发生的变更事件触发（如 Salesforce 记录变更） |

### 私有连接（考试提示）

- 支持通过 **AWS PrivateLink** 在 AppFlow 与部分 SaaS 应用/VPC 内私有资源之间建立连接，数据传输**不经过公共互联网**，降低敏感业务数据的暴露面
- 传输的数据可通过 **[[KMS]]** 在静态存储时加密

---

## Amazon AppFlow vs Amazon EventBridge（考试高频陷阱）

| 维度 | Amazon AppFlow | [[Amazon EventBridge]]（合作伙伴事件总线） |
|------|-----------------|----------------------------------------------|
| **传输内容** | **批量/成规模的业务数据记录**（如客户对象、订单记录） | **离散的事件通知**（如"某条工单被创建"这一件事发生了） |
| **典型触发方式** | 按需/计划/源应用变更事件，**批量**处理 | 单个事件发生时**实时**路由 |
| **是否支持数据转换** | **支持**（映射、过滤、脱敏、校验、聚合） | 不支持，只做事件内容的**模式匹配和路由**，不改变事件负载 |
| **典型场景** | 定期把 Salesforce 全部/增量客户数据同步到 S3/Redshift 做分析 | Zendesk 工单创建后**立即**触发 Lambda 发送通知 |

> **考试陷阱**：**两者都能"对接第三方 SaaS"，但解决的问题层次完全不同**——题目描述"需要定期把 SaaS 应用中的业务数据整体/增量同步到数据仓库做分析" → **AppFlow**；题目描述"需要在 SaaS 应用发生某个具体事件时实时触发下游自动化响应" → **[[Amazon EventBridge]]** 的合作伙伴事件总线，完整机制见 [[Amazon EventBridge]] 独立笔记。两者甚至可以组合使用：AppFlow 负责批量数据落地，EventBridge 负责实时事件触发。

---

## 与相关数据集成/迁移服务的边界（考试高频）

| 服务 | 数据类型 | 典型场景 |
|------|---------|---------|
| **Amazon AppFlow** | SaaS 应用的**结构化业务记录**（通过应用 API） | Salesforce/ServiceNow 等 SaaS 数据与 AWS 服务之间的双向同步 |
| **[[AWS Glue]]** | 已在 AWS/数据湖中的数据，**通用 ETL** | 对 S3/数据库中的数据做基于 Spark 的复杂转换、编目，完整能力见 [[AWS Glue]] 独立笔记 |
| **[[AWS DataSync]]** | **文件/对象**数据 | 本地文件系统 ↔ S3/EFS/FSx 的持续同步，完整边界见 [[AWS DataSync]] 独立笔记 |
| **[[Database Migration Service]]** | **数据库引擎**内部的数据 | 数据库到数据库的迁移/持续复制，完整边界见 [[Database Migration Service]] 独立笔记 |

> **考试陷阱**：**四者极易在"数据集成/迁移"类题目中被混淆，判断依据是"数据来源的形态"**——来自 **SaaS 应用 API**（如 Salesforce）用 **AppFlow**；来自**本地文件系统**用 **DataSync**；来自**数据库引擎**用 **DMS**；数据**已经在 AWS 内部**、需要复杂转换和编目用 **Glue**。常见的组合模式是：**AppFlow 把 SaaS 数据落地到 S3 → Glue 对其做进一步的清洗、转换和编目 → 供 Athena/Redshift 分析**，三者是流水线中的不同环节，而非互相替代。

---

## 典型应用场景

| 场景 | 推荐配置 |
|------|---------|
| **定期把 Salesforce 客户数据同步到 S3/Redshift 做分析** | Amazon AppFlow（计划触发） |
| **需要把 AWS 侧处理结果写回 SaaS 应用供业务人员使用** | Amazon AppFlow（AWS → SaaS 方向） |
| **只同步满足特定条件的记录，并对敏感字段脱敏** | AppFlow 的过滤条件 + 数据脱敏 |
| **SaaS 应用内部发生特定事件时需要实时触发自动化响应** | 改用 [[Amazon EventBridge]] 合作伙伴事件总线（而非 AppFlow） |
| **希望传输过程不经过公共互联网** | AppFlow + AWS PrivateLink |
| **数据落地后还需要复杂的 Spark ETL 和数据编目** | AppFlow 负责落地 + [[AWS Glue]] 负责后续深度处理 |
| **需要同步的是本地文件而非 SaaS 应用数据** | 改用 [[AWS DataSync]]（而非 AppFlow） |
| **需要迁移/同步的是数据库而非 SaaS 应用数据** | 改用 [[Database Migration Service]]（而非 AppFlow） |

---

## 考试重点总结

### SAA-C02 高频考点

1. **AppFlow 专为 SaaS 应用与 AWS 服务之间的数据集成设计**：无需编写自定义 API 对接代码
2. **支持双向数据流**：SaaS → AWS 和 AWS → SaaS 均可，不局限于单一方向
3. **传输过程内置数据转换能力**：映射、过滤、脱敏、校验、聚合
4. **三种触发方式**：按需、计划、源应用事件驱动
5. **AppFlow 传输批量业务数据记录，EventBridge 路由离散事件通知**：这是两者最核心也最易混淆的区别
6. **支持 AWS PrivateLink 私有连接**：避免敏感数据经过公共互联网
7. **与 DataSync 的区别是数据形态**：AppFlow 面向 SaaS API 数据，DataSync 面向文件/对象
8. **与 DMS 的区别是数据来源**：AppFlow 面向 SaaS 应用，DMS 面向数据库引擎
9. **常与 AWS Glue 组合使用**：AppFlow 负责从 SaaS 落地数据，Glue 负责后续复杂 ETL 和编目
10. **判断依据是数据来源形态**：SaaS API → AppFlow；文件系统 → DataSync；数据库 → DMS；已在 AWS 内部需复杂转换 → Glue

### 场景题解题思路

```
场景分析 → 判断 Amazon AppFlow 相关配置
├── "定期同步 SaaS 应用数据到 S3/Redshift 做分析" → Amazon AppFlow
├── "需要把 AWS 处理结果写回 SaaS 应用" → AppFlow（AWS → SaaS 方向）
├── "只需同步部分记录并对敏感字段脱敏" → AppFlow 过滤 + 脱敏
├── "SaaS 内部事件需要实时触发自动化响应" → 改用 EventBridge 合作伙伴事件总线
├── "传输不能经过公共互联网" → AppFlow + PrivateLink
├── "数据落地后需要复杂 ETL/编目" → AppFlow + AWS Glue 组合
├── "需要同步的是本地文件" → 改用 AWS DataSync
└── "需要迁移/同步的是数据库" → 改用 Database Migration Service
```

---

## 最佳实践

1. **按数据来源形态而非"要不要集成 SaaS"来选型**：SaaS API 用 AppFlow，文件用 DataSync，数据库用 DMS，避免混用
2. **需要实时事件响应时不要误用 AppFlow**：批量/计划同步场景才是 AppFlow 的设计目标，实时事件应改用 EventBridge
3. **传输敏感业务数据时优先启用 PrivateLink**：减少数据经过公共互联网的暴露面
4. **善用内置的过滤和脱敏能力**：避免把不必要的敏感字段全量同步到下游，之后再想办法清理
5. **大规模数据落地后交给 AWS Glue 做深度处理**：不要试图在 AppFlow 的映射/过滤能力之外硬塞复杂的 Spark 级转换逻辑
6. **双向同步场景明确设计好数据一致性策略**：避免 AWS 与 SaaS 两侧因同步延迟产生数据冲突
7. **合理选择触发方式**：高频变更的数据优先评估事件驱动触发，而非固定的计划任务造成同步延迟或资源浪费