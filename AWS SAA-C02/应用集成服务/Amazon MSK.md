# Amazon MSK - 托管 Apache Kafka 流式服务

> **Amazon MSK（Managed Streaming for Apache Kafka）** 是全托管的 **Apache Kafka** 服务，让用户能够构建和运行基于开源 Kafka 的实时流数据管道，兼容标准 Kafka API、客户端和生态工具（Kafka Connect、Kafka Streams 等），无需自行部署、运维和修补 Kafka/ZooKeeper 集群。
>
> 相关文档：[[Amazon Kinesis]] | [[Amazon Managed Service for Apache Flink]] | [[Amazon EMR]] | [[AWS Glue]] | [[S3]] | [[Redshift]] | [[VPC]] | [[KMS]]

---

## 核心概念

### 为什么需要 MSK

- **迁移诉求**：许多企业已有基于开源 **Apache Kafka** 构建的流数据管道（生产者/消费者代码、Kafka Connect 连接器、Kafka Streams 应用），迁移到全新的流服务意味着重写代码和连接器配置
- **MSK 的定位**：与 [[Amazon Keyspaces]] 之于 Cassandra、[[Amazon DocumentDB]] 之于 MongoDB 的关系类似——提供**协议/API 兼容层**，让已有 Kafka 工作负载几乎不改代码即可迁移上云，同时摆脱自建 Kafka/ZooKeeper 集群的运维负担（Broker 扩缩容、版本升级、磁盘管理、监控告警）
- **核心价值**：保留 Kafka 生态的成熟工具链（Debezium、Kafka Connect 连接器、各语言客户端库）和运维人员已有的 Kafka 知识，同时获得云原生的托管体验

### MSK 在流处理生态中的定位（考试要点）

| 服务 | 协议/生态 | 消费模型 | 典型场景 |
|------|----------|---------|---------|
| **Amazon MSK** | 兼容 **Apache Kafka** 协议、客户端、生态工具 | 发布/订阅日志模型，多消费者组各自独立按偏移量（Offset）消费 | 已有 Kafka 工作负载迁移、需要 Kafka 生态工具/连接器的场景 |
| [[Amazon Kinesis]] Data Streams | AWS 专有 API | 拉取模型，多消费者独立重放同一数据 | 云原生新应用，无 Kafka 历史包袱，希望更简单的托管体验 |
| Amazon MQ | 兼容 ActiveMQ/RabbitMQ 协议 | 传统消息队列/主题模型 | 已有基于 JMS/AMQP 等传统消息中间件的应用迁移 |

> **考试陷阱**：题目描述**"已有基于 Apache Kafka 构建的流数据管道，希望以最小改动量迁移到 AWS 并摆脱自运维集群"** → 答案是 **Amazon MSK**，而非 Kinesis Data Streams——即使两者都是发布/订阅式流服务，Kinesis **不兼容 Kafka 客户端和协议**，迁移意味着重写生产者/消费者代码和连接器；只有**全新构建、无 Kafka 历史包袱**的场景才应优先考虑更简单的 Kinesis Data Streams。这与 [[Amazon Keyspaces]]/[[Amazon DocumentDB]] 的判断逻辑完全一致：**看是否已有兼容特定开源协议的历史应用**。

---

## 部署模式（考试高频）

| 模式 | 说明 | 适用场景 |
|------|------|---------|
| **MSK Provisioned（预置集群）** | 手动指定 Broker 数量、实例类型和存储容量，精细控制集群配置 | 负载可预测，需要精细调优性能和成本的长期稳定工作负载 |
| **MSK Serverless** | 全自动伸缩计算和存储，无需预置或管理 Broker 容量，按实际**流入/流出的数据量**计费 | 负载不可预测、希望免运维、间歇性或增长趋势不确定的工作负载 |

### Broker 类型

| Broker 类型 | 特点 |
|------------|------|
| **Standard Broker** | 传统的 MSK Broker 类型，存储与计算耦合在同一实例上 |
| **Express Broker**（新一代） | 存储与计算解耦，扩容更快、吞吐更高，开箱即用地应用 Kafka 最佳实践配置，简化性能调优 |

---

## 架构基础

### 核心概念

- **主题（Topic）与分区（Partition）**：数据按 Topic 组织，每个 Topic 可拆分为多个 Partition 实现并行读写；同一 Partition 内消息严格有序，跨 Partition 不保证全局顺序
- **消费者组（Consumer Group）**：多个消费者可组成一个 Group 共同消费一个 Topic，组内每个 Partition 只由一个消费者处理；不同 Consumer Group 之间**相互独立、各自维护偏移量**，可重复消费同一份数据
- **偏移量（Offset）**：消费者在 Partition 内的读取位置标记，支持从任意历史偏移量重新消费（类似 Kinesis 的重放能力，但这是 Kafka 原生设计）

### KRaft 模式（去 ZooKeeper 化）

- Kafka 3.7+ 版本的 MSK 集群支持 **KRaft（Kafka Raft）模式**，用 Kafka 内置的共识协议替代传统的外部 **ZooKeeper** 做元数据管理
- **考试要点**：KRaft 模式下集群可扩展到最多 **60 个 Broker**，相比传统 ZooKeeper 模式的 **30 个 Broker** 上限翻倍，简化了架构（无需单独运维 ZooKeeper 组件）

---

## MSK Connect

- 托管版本的 **Kafka Connect**，用于在 MSK 集群与外部数据源/数据目标之间做数据集成（如从 RDS 通过 Debezium 做变更数据捕获，或将数据导出到 S3）
- 无需自行部署和管理 Kafka Connect Worker 节点，AWS 负责连接器的运行和伸缩

## MSK Replicator

- 托管的跨区域/同区域数据复制能力，替代自建 **MirrorMaker 2**，简化 Kafka 集群间的数据复制配置
- 典型用途：跨区域灾难恢复、多区域低延迟读取、将本地 Kafka 集群数据持续复制到 AWS 云端

---

## 安全性

| 机制 | 说明 |
|------|------|
| **VPC 部署** | MSK 集群运行在 [[VPC]] 内的私有子网中，通过安全组控制网络访问，默认不暴露公网 |
| **IAM 访问控制** | 支持使用 **IAM 策略**对 Kafka Topic/Group 级别的操作（生产、消费）进行授权，替代传统 Kafka 的 ACL 管理方式 |
| **传统 Kafka 认证** | 同时支持 **SASL/SCRAM**、**mTLS（双向 TLS）**等 Kafka 原生认证机制，便于已有 Kafka 客户端平滑迁移 |
| **静态加密** | 使用 [[KMS]] 管理的密钥对 Broker 存储的数据加密 |
| **传输加密** | Broker 间及客户端到 Broker 的通信默认支持 TLS 加密 |

---

## 与其他服务的集成

| 集成服务 | 用途 |
|---------|------|
| **[[Amazon Managed Service for Apache Flink]]** | 可直接以 MSK 作为输入源，对 Kafka 流数据做实时窗口聚合、有状态计算 |
| **[[AWS Glue]] Schema Registry** | 管理 MSK 流数据的 schema（Avro/JSON/Protobuf），确保生产者消费者 schema 一致性 |
| **[[Amazon EMR]]** | 可运行 Kafka Streams 或 Spark Structured Streaming 作业消费 MSK 数据做复杂处理 |
| **MSK Connect + [[S3]]** | 通过连接器将 Kafka 流数据持续导出到 S3 数据湖 |

---

## 典型应用场景

| 场景 | 推荐配置 |
|------|---------|
| **已有 Kafka 应用整体迁移上云** | MSK Provisioned + 原有 Kafka 客户端代码，几乎零改动 |
| **摆脱自建 Kafka/ZooKeeper 集群运维负担** | MSK，AWS 负责 Broker、补丁、扩容 |
| **负载不可预测，不想管理集群容量** | MSK Serverless |
| **需要 Kafka Connect 生态的连接器（CDC、S3 导出等）** | MSK Connect |
| **跨区域灾备或多区域数据复制** | MSK Replicator，替代自建 MirrorMaker 2 |
| **需要处理 Kafka 流数据的实时窗口聚合** | MSK + [[Amazon Managed Service for Apache Flink]] |
| **全新构建、无历史 Kafka 代码** | 改用 [[Amazon Kinesis]] Data Streams，而非 MSK |

---

## 考试重点总结

### SAA-C02 高频考点

1. **核心定位**：与 Apache Kafka **协议/客户端兼容**的全托管流服务，专为 Kafka 工作负载迁移设计
2. **MSK vs Kinesis Data Streams**：题目强调"已有 Kafka 应用/客户端代码"→ MSK；"全新构建、无历史包袱"→ Kinesis，两者不兼容彼此的协议
3. **两种部署模式**：Provisioned（手动管理 Broker，精细控制）vs Serverless（自动伸缩，按用量计费）
4. **Standard vs Express Broker**：Express 存算分离、扩容更快、开箱即用最佳实践配置
5. **KRaft 模式取代 ZooKeeper**：Kafka 3.7+ 支持，Broker 上限从 30 提升到 60，简化架构
6. **消费者组独立消费**：多个 Consumer Group 可各自独立、重复消费同一 Topic 数据，互不影响
7. **MSK Connect**：托管版 Kafka Connect，简化与外部系统（数据库 CDC、S3）的集成
8. **MSK Replicator**：托管跨区域/同区域复制，替代自建 MirrorMaker 2
9. **IAM 可用于 Kafka 级别授权**：除传统 SASL/SCRAM、mTLS 外，也支持用 IAM 策略控制 Topic/Group 访问
10. **默认部署在 VPC 私有子网**：不像 Kinesis 是完全托管的公共 API 端点，MSK 集群位于用户 VPC 内

### 场景题解题思路

```
场景分析 → 判断是否用 MSK
├── "已有基于 Apache Kafka 构建的应用，需要最小改动迁移到云端" → Amazon MSK
├── "希望摆脱自建 Kafka/ZooKeeper 集群的运维负担" → Amazon MSK
├── "全新构建的应用，无 Kafka 历史代码" → 改用 Kinesis Data Streams（而非 MSK）
├── "负载不可预测，不想管理 Broker 容量" → MSK Serverless
├── "需要 Kafka Connect 连接器做 CDC 或数据导出" → MSK Connect
├── "需要跨区域复制 Kafka 数据用于灾备" → MSK Replicator（而非自建 MirrorMaker 2）
├── "需要对 Kafka 流数据做实时窗口聚合" → MSK + Amazon Managed Service for Apache Flink
└── "集群需要扩展到 30 个 Broker 以上" → 使用 KRaft 模式（Kafka 3.7+）
```

---

## 最佳实践

1. **迁移 Kafka 工作负载优先评估 MSK**：能复用现有客户端代码和 Kafka Connect 连接器，大幅降低迁移成本
2. **负载不可预测时优先 MSK Serverless**：避免预置容量估算不准导致的浪费或瓶颈
3. **新集群优先选择 Express Broker**：存算分离架构提供更快扩容速度和开箱即用的性能调优
4. **需要超过 30 个 Broker 时启用 KRaft 模式**：同时简化架构，无需单独运维 ZooKeeper
5. **优先使用 IAM 做访问控制**：相比手动管理 SASL/SCRAM 凭证，IAM 策略更易与现有 AWS 权限体系集成
6. **跨系统集成优先使用 MSK Connect**：而非自行部署和运维 Kafka Connect Worker
7. **跨区域容灾使用 MSK Replicator**：避免自建和运维 MirrorMaker 2 的复杂度
8. **流数据 schema 管理接入 Glue Schema Registry**：确保生产者消费者 schema 演变时的兼容性
