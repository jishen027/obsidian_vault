# Amazon Keyspaces - 兼容 Apache Cassandra 的托管数据库

> **Amazon Keyspaces（for Apache Cassandra）** 是完全托管、无服务器（Serverless）的宽列（Wide-Column）NoSQL 数据库服务，与 **Apache Cassandra** 完全兼容——可直接使用现有的 **CQL（Cassandra Query Language）** 代码、开源驱动和运维工具，无需重写应用即可将本地 Cassandra 工作负载迁移上云。
>
> 相关文档：[[DynamoDB]] | [[RDS]] | [[Aurora]] | [[Amazon Neptune]] | [[Amazon Timestream]] | [[Amazon DocumentDB]] | [[ElastiCache]] | [[KMS]] | [[VPC]]

---

## 核心概念

### 为什么需要 Keyspaces

- **迁移诉求**：许多企业已有基于开源 **Apache Cassandra** 构建的应用（宽列存储、高写入吞吐、多数据中心部署经验），迁移到全新数据模型（如 DynamoDB）意味着大量代码重写
- **Keyspaces 的定位**：提供与 Cassandra **API 级兼容**（CQL、驱动协议）的托管服务，让这些应用**几乎不改代码**即可迁移，同时甩掉自建 Cassandra 集群的运维负担（节点管理、版本升级、修补、容量规划）
- **核心价值**：用云原生的无服务器体验，替换掉开源 Cassandra 集群繁重的自运维成本

### Keyspaces 在数据库家族中的定位（考试要点）

| 服务 | 类型 | 兼容协议/API | 典型场景 |
|------|------|-------------|---------|
| **Keyspaces** | 宽列（Wide-Column）NoSQL | **Cassandra Query Language (CQL)** | 已有 Cassandra 应用迁移上云、宽列数据模型、高吞吐写入 |
| [[DynamoDB]] | 键值/文档 NoSQL | DynamoDB 专有 API | 云原生新应用、无历史包袱、按主键访问的低延迟场景 |
| [[Amazon Neptune]] | 图数据库 | Gremlin/openCypher/SPARQL | 深度关联查询（社交网络、推荐引擎、欺诈检测） |
| [[Amazon Timestream]] | 时序数据库 | 类 SQL / InfluxQL / Flux | 带时间戳数据，按时间范围查询和聚合分析 |
| [[Amazon DocumentDB]] | 文档数据库 | MongoDB 驱动/API 兼容 | 已有 MongoDB 应用迁移、半结构化文档存储 |
| [[RDS]] / [[Aurora]] | 关系型（SQL） | 标准 SQL | 强 ACID 事务、复杂关联查询 |

> **考试陷阱**：题目描述**"已有基于 Apache Cassandra 构建的应用，希望以最小改动量迁移到 AWS 并摆脱自运维集群"** → 答案是 **Amazon Keyspaces**，而不是 DynamoDB——即使两者都是 NoSQL，DynamoDB **不兼容 CQL**，迁移意味着重写数据访问层；只有在**全新构建、无 Cassandra 历史包袱**的场景才应优先考虑 DynamoDB。

---

## 数据模型与兼容性

### 宽列存储模型

- 数据以**表（Table）→ 分区（Partition）→ 行（Row）→ 列（Column）**组织，同一分区内的行可以拥有不同的列集合，schema 相对灵活
- 表通过**分区键（Partition Key）**决定数据的物理分布，可选配**聚簇列（Clustering Columns）**决定分区内的排序，概念上与 DynamoDB 的**分区键+排序键**类似，但语法和数据类型体系遵循 Cassandra 标准

### 协议与工具兼容性

| 兼容项 | 说明 |
|--------|------|
| **CQL（Cassandra Query Language）** | 语法接近 SQL 的查询语言，Keyspaces 原生支持标准 CQL 语句（含批处理、UDT 等） |
| **开源 Cassandra 驱动** | 支持 Apache 2.0 许可的官方 Cassandra 驱动（Java、Python、Node.js 等），应用代码基本无需修改 |
| **CQLSH** | 可通过标准 `cqlsh` 客户端或 AWS 提供的 CQLSH 集成 CloudShell 直接连接管理 |

> **考试要点**：Keyspaces 是**协议兼容**而非**代码复刻**——底层存储引擎和分布式架构是 AWS 自研的，并非直接运行开源 Cassandra 软件；这带来了托管服务的弹性和可用性优势，但也意味着极小部分 Cassandra 高级特性（如某些自定义函数）可能不受支持，题目中若出现"完全一致的开源特性覆盖"描述需留意其局限。

---

## 容量模式与性能

| 模式 | 说明 | 适用场景 |
|------|------|---------|
| **按需模式（On-Demand）** | 按实际读写请求量付费，自动伸缩，无需容量规划 | 流量不可预测或希望免运维容量管理 |
| **预置容量模式（Provisioned）** | 手动指定读写容量单位，可配合 **Application Auto Scaling** 自动调整 | 流量可预测，希望精细控制成本 |
| **预热吞吐（Warm Throughput）** | 为新表或已有表提前"预热"，应对可预见的流量高峰，避免突发流量下的节流 | 大促、活动等已知峰值场景 |

- **无限扩展**：表的吞吐量和存储可随数据量和请求量**近乎无限扩展**，无需像自建 Cassandra 那样手动扩容节点
- **单毫秒级延迟**：读写延迟稳定在个位数毫秒级别，符合 NoSQL 宽列数据库对低延迟的设计目标
- **99.999% 可用性 SLA**：数据自动跨多个可用区复制（类似 Multi-AZ），无需用户手动配置多数据中心拓扑

---

## 变更数据捕获（CDC）

- **Amazon Keyspaces Streams**：捕获表数据的变更事件（新增、修改、删除），类似 DynamoDB Streams 的能力
- **Kinesis Adapter 集成**：可使用 **Kinesis Client Library (KCL)** 处理 Keyspaces CDC 流，复用现有 Kinesis 消费端生态
- **迭代器位置（Iterator Position）**：CDC 流的 `GetRecords` 响应可返回消费位置信息，判断消费者是否已追平最新记录，便于构建可靠的流处理管道

---

## 安全性

| 机制 | 说明 |
|------|------|
| **IAM 身份验证** | 支持使用 IAM 角色/用户进行身份验证，无需管理独立的 Cassandra 用户名密码体系 |
| **静态加密** | 默认使用 AWS 拥有的密钥加密，也可选择客户管理的 [[KMS]] 密钥 |
| **传输加密** | 客户端与服务的连接默认通过 TLS |
| **VPC Endpoint** | 支持通过接口终端节点在 [[VPC]] 内私有访问，无需经过公网 |
| **IPv6 双栈** | 支持 IPv4/IPv6 双栈端点，满足新一代网络合规要求 |

---

## 多区域复制（Multi-Region Replication）

- 支持将表配置为**多区域复制表**，在多个 AWS 区域间自动同步数据，实现全球低延迟读写和跨区域灾难恢复
- 支持在**已有 Keyspace 基础上追加新区域**，无需重建整个数据集
- **用户自定义类型（UDT, User Defined Types）**支持多区域一致性，全球化应用可在不同区域保持相同的数据结构定义

---

## 典型应用场景

| 场景 | 推荐配置 |
|------|---------|
| **已有 Cassandra 应用整体迁移上云** | Keyspaces + 原有 CQL 代码/驱动，几乎零代码改动 |
| **摆脱自建 Cassandra 集群运维负担** | Keyspaces 按需模式，AWS 负责节点、补丁、扩容 |
| **高写入吞吐的宽列数据（IoT 时序、日志）** | Keyspaces + 分区键设计优化写入分布 |
| **已知流量高峰的活动/大促场景** | Keyspaces 预热吞吐（Warm Throughput） |
| **全球化应用的多区域低延迟读写** | Keyspaces 多区域复制表 |
| **变更事件驱动的实时处理管道** | Keyspaces Streams + Kinesis Adapter |
| **全新构建、无历史 Cassandra 代码** | 改用 [[DynamoDB]]，而非 Keyspaces |

---

## 考试重点总结

### SAA-C02 高频考点

1. **核心定位**：与 Apache Cassandra **协议兼容**（CQL + 开源驱动）的全托管无服务器数据库，专为 Cassandra 工作负载迁移设计
2. **Keyspaces vs DynamoDB**：题目强调"已有 Cassandra 应用/CQL 代码"→ Keyspaces；"全新构建、无历史包袱"→ DynamoDB，两者不兼容彼此的 API
3. **宽列存储模型**：表→分区→行→列，分区键决定物理分布，聚簇列决定分区内排序
4. **两种容量模式**：按需模式（自动伸缩免规划）vs 预置容量模式（可配 Auto Scaling）
5. **预热吞吐（Warm Throughput）**：应对可预见流量高峰，避免突发场景下的节流
6. **99.999% 可用性**：数据自动跨可用区复制，无需手动搭建多数据中心
7. **Keyspaces Streams + Kinesis Adapter**：变更数据捕获可复用 KCL 生态处理
8. **IAM 身份验证**：替代传统 Cassandra 用户名密码体系，与 AWS 权限模型统一
9. **支持多区域复制表**：全球化应用的低延迟读写和灾难恢复
10. **底层是 AWS 自研引擎，非直接运行开源 Cassandra**：极少数高级开源特性可能不受支持

### 场景题解题思路

```
场景分析 → 判断是否用 Keyspaces
├── "已有基于 Apache Cassandra 的应用，需要最小改动迁移到云端" → Amazon Keyspaces
├── "希望摆脱自建 Cassandra 集群的节点/补丁/容量运维" → Amazon Keyspaces
├── "全新构建的应用，无 Cassandra 历史代码" → 改用 DynamoDB（而非 Keyspaces）
├── "需要按主键做深度关联/图遍历查询" → 改用 Amazon Neptune（而非 Keyspaces）
├── "已知即将到来的流量峰值（大促/活动）" → 配置 Warm Throughput 预热
├── "需要处理表的变更事件流" → Keyspaces Streams + Kinesis Adapter（KCL）
├── "全球化应用需要多区域低延迟读写" → 多区域复制表
└── "流量不可预测，不想做容量规划" → 按需容量模式
```

---

## 最佳实践

1. **迁移 Cassandra 工作负载优先评估 Keyspaces**：能复用现有 CQL 代码和驱动，大幅降低迁移成本
2. **分区键设计参照 Cassandra 最佳实践**：选择基数高、访问均匀的属性，避免热分区
3. **流量不可预测时选按需模式**：预置容量模式适合流量稳定、希望精细控制成本的场景
4. **已知峰值提前配置 Warm Throughput**：避免大促等场景下因预热不足产生节流
5. **变更事件处理复用 Kinesis 生态**：通过 Kinesis Adapter 对接 KCL，降低自建消费端的开发成本
6. **全球化应用启用多区域复制**：而非在应用层自建跨区域同步逻辑
7. **身份验证优先使用 IAM**：统一在 AWS 权限模型下管理访问，而非维护独立的 Cassandra 凭证体系
8. **VPC 内访问走接口终端节点**：避免流量经过公网
