# Amazon DocumentDB - 兼容 MongoDB 的文档数据库

> **Amazon DocumentDB（with MongoDB compatibility）** 是完全托管、与 **MongoDB** 兼容的**文档数据库**服务，支持现有 MongoDB 驱动、工具和应用代码，让 JSON 风格的半结构化文档数据获得云原生的可扩展性、高可用性和运维托管能力，而无需自建和运维 MongoDB 集群。
>
> 相关文档：[[DynamoDB]] | [[Amazon Keyspaces]] | [[Amazon Neptune]] | [[Amazon Timestream]] | [[RDS]] | [[Aurora]] | [[KMS]] | [[VPC]]

---

## 核心概念

### 为什么需要 DocumentDB

- **迁移诉求**：许多应用基于开源 **MongoDB** 构建（JSON 文档模型、灵活 schema），迁移到全新数据模型意味着大量代码重写；DocumentDB 提供**驱动级兼容**，让这些应用几乎不改代码即可迁移
- **DocumentDB 的定位**：与 [[Amazon Keyspaces]] 之于 Cassandra 的关系类似——都是 AWS 自研引擎、对开源产品的**协议/API 兼容层**，而非直接运行开源软件，用云原生托管体验替代自建集群的运维负担
- **核心价值**：摆脱 MongoDB 集群自运维（分片管理、副本集配置、版本升级、容量规划），同时保留原有应用代码和开发者熟悉的查询语法

### DocumentDB 在数据库家族中的定位（考试要点）

| 服务 | 类型 | 兼容协议/API | 典型场景 |
|------|------|-------------|---------|
| **DocumentDB** | 文档（Document）数据库 | **MongoDB 驱动/API 兼容**（3.6-8.0） | 已有 MongoDB 应用迁移上云、半结构化 JSON 文档存储 |
| [[Amazon Keyspaces]] | 宽列 NoSQL | CQL（Cassandra 兼容） | 已有 Cassandra 应用迁移 |
| [[DynamoDB]] | 键值/文档 NoSQL | DynamoDB 专有 API | 云原生新应用、按主键访问的低延迟场景 |
| [[RDS]] / [[Aurora]] | 关系型（SQL） | 标准 SQL | 强 ACID 事务、复杂关联查询 |

> **考试陷阱**：题目描述**"已有基于 MongoDB 构建的应用，希望以最小改动量迁移到 AWS 并摆脱自运维集群"** → 答案是 **Amazon DocumentDB**，而非 DynamoDB——即使两者都是面向文档/JSON 的 NoSQL 服务，DynamoDB **不兼容 MongoDB 驱动和查询语法**，迁移意味着重写数据访问层；只有**全新构建、无 MongoDB 历史包袱**的场景才应优先考虑 DynamoDB。这与 [[Amazon Keyspaces]] vs DynamoDB 的判断逻辑完全一致：**看是否已有兼容特定开源协议的历史应用**。

---

## 架构

### 存储与计算分离

- 与 [[Aurora]] 同源的架构理念：**计算层（实例）**与**存储层（分布式存储卷）**解耦，存储层跨多个可用区自动复制（通常 6 份副本），无需用户管理底层存储扩展
- 存储**自动扩展**，最高可达 **64 TiB**，按实际使用量增长，无需预先分配
- 一个集群包含 **1 个主实例（读写）+ 最多 15 个只读副本**，只读副本共享同一份底层存储，几乎无复制延迟

### 两种集群模式

| 集群模式 | 说明 | 适用场景 |
|---------|------|---------|
| **实例集群（Instance-Based Cluster）** | 需手动选择实例规格，与传统 Aurora/RDS 集群管理方式一致 | 负载可预测，希望精细控制实例规格和成本 |
| **弹性集群（Elastic Clusters）** | 无需选择/管理/升级实例，自动分片（Sharding），支持**百万级读写/秒**和 **PB 级存储**，可**启动/停止（Start/Stop）**集群，支持**可读辅助节点（Readable Secondaries）** | 超大规模、需要水平分片扩展、不想管理实例规格的场景 |

> **考试要点**：**弹性集群使用兼容 MongoDB 5.0 的 Wire Protocol**，其"自动分片、免实例管理"的能力类似于把 DynamoDB 的无服务器体验带到了 MongoDB 兼容层——题目强调"MongoDB 兼容 + 免容量管理 + 超大规模水平扩展"时，答案是 DocumentDB **Elastic Clusters**，而非普通实例集群。

---

## MongoDB 兼容性

| 兼容项 | 说明 |
|--------|------|
| **驱动兼容** | 支持绝大多数 MongoDB 官方驱动（Java、Python、Node.js 等），应用代码基本无需修改 |
| **API/查询语法兼容** | 支持 MongoDB 3.6、4.0、5.0、6.0、7.0、8.0 版本的绝大部分查询、聚合管道（Aggregation Pipeline）语法 |
| **工具兼容** | 兼容 `mongosh`、MongoDB Compass 等常见管理工具 |

> **考试要点**：DocumentDB 是**协议兼容**而非**代码复刻**——底层存储引擎是 AWS 自研的、构建在分布式存储之上，并非直接运行开源 MongoDB 软件；这带来了托管服务的弹性和可用性优势，但**并非 100% 覆盖所有 MongoDB 特性**（例如某些高级聚合操作符或存储引擎特定功能可能不受支持），题目中若强调"完全等同开源 MongoDB 的全部特性"需留意其局限，与 [[Amazon Keyspaces]] 相对于 Cassandra 的兼容性局限逻辑一致。

---

## 高可用与灾难恢复

| 机制 | 说明 |
|------|------|
| **多可用区部署** | 存储层自动跨多个可用区复制（通常 6 份），单个可用区故障不影响数据持久性 |
| **只读副本自动故障转移** | 主实例故障时，DocumentDB 自动将某个只读副本提升为新主实例，故障转移通常在 **30 秒左右**完成 |
| **全局集群（Global Clusters）** | 支持跨多个 AWS 区域的低延迟只读副本和快速灾难恢复，主区域故障时可将某个次要区域快速提升为主区域 |
| **自动/手动快照** | 自动快照默认启用，支持时间点恢复（PITR）；也支持手动快照长期保留和跨区域复制 |

---

## 安全性

| 机制 | 说明 |
|------|------|
| **VPC 部署** | DocumentDB 集群只能部署在 [[VPC]] 内，通过安全组控制网络访问，不支持公网直接访问 |
| **静态加密** | 使用 [[KMS]] 管理的密钥对存储数据加密，**只能在创建集群时启用**，无法对已有未加密集群直接开启（与 Aurora/Neptune 的加密限制一致） |
| **传输加密** | 客户端与集群间的连接默认使用 TLS |
| **IAM 权限控制** | 通过 IAM 策略控制对 DocumentDB 管理面 API（创建/删除集群等）的访问；数据面身份验证仍使用 DocumentDB 原生用户名/密码 |

---

## 典型应用场景

| 场景 | 推荐配置 |
|------|---------|
| **已有 MongoDB 应用整体迁移上云** | DocumentDB 实例集群 + 原有驱动代码，几乎零改动 |
| **摆脱自建 MongoDB 副本集/分片集群运维** | DocumentDB，AWS 负责实例、补丁、存储扩展 |
| **超大规模、需要水平分片的文档工作负载** | DocumentDB Elastic Clusters，免实例管理 |
| **半结构化 JSON 数据、灵活 schema 的应用** | DocumentDB，无需预定义严格表结构 |
| **全球化应用的低延迟读 + 快速容灾** | DocumentDB Global Clusters |
| **内容管理系统、目录服务、用户画像存储** | DocumentDB，文档模型天然贴合嵌套/半结构化数据 |
| **全新构建、无历史 MongoDB 代码** | 改用 [[DynamoDB]]，而非 DocumentDB |

---

## 考试重点总结

### SAA-C02 高频考点

1. **核心定位**：与 MongoDB **驱动/API 兼容**的全托管文档数据库，专为 MongoDB 工作负载迁移设计，底层是 AWS 自研引擎
2. **DocumentDB vs DynamoDB**：题目强调"已有 MongoDB 应用/驱动代码"→ DocumentDB；"全新构建、无历史包袱"→ DynamoDB
3. **架构与 Aurora 同源**：存储计算分离，1 主 + 最多 15 只读副本，共享分布式存储，故障转移约 30 秒
4. **存储自动扩展至 64 TiB**：无需预先分配容量
5. **实例集群 vs 弹性集群（Elastic Clusters）**：前者手动管理实例规格，后者自动分片、免实例管理，支持百万级读写/秒和 PB 级存储
6. **弹性集群支持启动/停止和可读辅助节点**：进一步降低成本和提升读扩展能力
7. **全局集群（Global Clusters）**：跨区域低延迟读 + 快速灾难恢复，与 Aurora Global Database 理念一致
8. **静态加密只能在创建时启用**：无法对已有集群事后追加，与 Aurora/Neptune 规则一致
9. **必须部署在 VPC 内**：不支持公网直接访问
10. **协议兼容非 100% 特性覆盖**：极少数 MongoDB 高级特性可能不受支持

### 场景题解题思路

```
场景分析 → 判断是否用 DocumentDB
├── "已有基于 MongoDB 构建的应用，需要最小改动迁移到云端" → Amazon DocumentDB
├── "希望摆脱自建 MongoDB 副本集/分片集群的运维负担" → Amazon DocumentDB
├── "全新构建的应用，无 MongoDB 历史代码" → 改用 DynamoDB（而非 DocumentDB）
├── "MongoDB 兼容 + 超大规模水平分片 + 不想管理实例" → DocumentDB Elastic Clusters
├── "负载可预测，希望精细控制实例规格" → DocumentDB 实例集群
├── "需要按主键做深度关联/图遍历查询" → 改用 Amazon Neptune（而非 DocumentDB）
├── "全球化应用需要低延迟读 + 快速容灾" → DocumentDB Global Clusters
└── "已有 Cassandra（而非 MongoDB）应用迁移" → 改用 Amazon Keyspaces（而非 DocumentDB）
```

---

## 最佳实践

1. **迁移 MongoDB 工作负载优先评估 DocumentDB**：能复用现有驱动代码和查询语法，大幅降低迁移成本
2. **超大规模/分片需求优先考虑 Elastic Clusters**：避免手动管理分片和实例规格的复杂度
3. **只读副本用于读扩展和高可用**：结合应用层读写分离，将读流量分散到只读副本
4. **创建集群时就启用静态加密**：加密无法对已有集群事后追加
5. **全球化应用启用 Global Clusters**：而非在应用层自建跨区域同步逻辑
6. **迁移前验证聚合管道等高级特性兼容性**：并非所有开源 MongoDB 特性都受支持，迁移前应做兼容性评估
7. **非生产/间歇性负载考虑 Elastic Clusters 的启动/停止能力**：降低闲置时段的成本
8. **VPC 内部署并结合安全组精细控制访问**：DocumentDB 不支持公网直接访问，需规划好网络拓扑
