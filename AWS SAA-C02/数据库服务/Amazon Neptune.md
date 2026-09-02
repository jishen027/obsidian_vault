# Amazon Neptune - 图数据库服务

> **Amazon Neptune** 是完全托管的**图数据库（Graph Database）**服务，专为存储和查询高度关联的数据（如社交网络、推荐引擎、欺诈检测、知识图谱）设计，原生支持 **属性图（Property Graph）** 与 **RDF** 两种图模型，可分别用 Gremlin/openCypher 与 SPARQL 查询。
>
> 相关文档：[[RDS]] | [[Aurora]] | [[DynamoDB]] | [[Redshift]] | [[Amazon Keyspaces]] | [[Amazon Timestream]] | [[Amazon DocumentDB]] | [[ElastiCache]] | [[KMS]] | [[VPC]]

---

## 核心概念

### 为什么需要图数据库

- **关系型数据库的短板**：查询"多跳关联"（如"朋友的朋友喜欢的商品"）需要多次 **JOIN**，跳数越多、JOIN 越多，查询性能急剧下降
- **图数据库的优势**：数据以**节点（Vertex）**和**边（Edge）**的形式原生存储，遍历关联关系是**常数时间的指针跳转**，而非昂贵的 JOIN 运算，适合数据之间的"关系"本身就是查询重点的场景
- **核心价值**：把"关系"作为一等公民建模，而不是像关系型数据库那样通过外键间接表达

### Neptune 在数据库家族中的定位（考试要点）

| 服务 | 类型 | 数据模型 | 典型场景 |
|------|------|---------|---------|
| **Neptune** | 图数据库 | 节点 + 边（Property Graph / RDF） | 社交网络、推荐引擎、欺诈检测、知识图谱、网络/IT 拓扑分析 |
| [[RDS]] / [[Aurora]] | 关系型（SQL） | 表 + 外键 | 强 ACID 事务、结构化业务数据、少量层级关系 |
| [[DynamoDB]] | NoSQL 键值/文档 | Item + 属性 | 按主键访问、海量并发、低延迟 |
| [[Amazon Keyspaces]] | 宽列 NoSQL（Cassandra 兼容） | 表 + 分区 + 列 | 已有 Cassandra 应用迁移、宽列高吞吐写入 |
| [[Amazon Timestream]] | 时序数据库 | 带时间戳、按时间范围查询 | IoT 传感器数据、监控指标、时序分析 |
| [[Amazon DocumentDB]] | 文档数据库（MongoDB 兼容） | 兼容 MongoDB 驱动、JSON 文档 | 已有 MongoDB 应用迁移、半结构化文档存储 |
| [[Redshift]] | 数据仓库（OLAP） | 列式存储 | 海量历史数据的聚合分析、BI 报表 |

> **考试陷阱**：题目描述**"需要查询多层级、深度关联的关系"**（如"找出用户的二度人脉"、"识别关联账户的欺诈网络"）→ 首选 **Neptune**；若只是"数据之间存在少量外键关联"，仍应使用 **RDS/Aurora**，不要看到"关系"两个字就联想到 Neptune——判断依据是**关联的深度和复杂度**，而非是否存在关联。

---

## 查询语言支持

| 图模型 | 查询语言 | 说明 |
|--------|---------|------|
| **属性图（Property Graph）** | **Apache TinkerPop Gremlin** | 命令式的图遍历语言，逐步描述"从节点 A 出发，沿某类边走几跳" |
| **属性图（Property Graph）** | **openCypher** | 声明式查询语言（源自 Neo4j），语法更接近 SQL，"描述想要的图形模式"而非遍历步骤 |
| **RDF（Resource Description Framework）** | **SPARQL** | W3C 标准的语义网查询语言，用于三元组（主语-谓语-宾语）数据模型，适合知识图谱、本体推理 |

> **考试要点**：**同一个 Neptune 集群内，Property Graph（Gremlin/openCypher）与 RDF（SPARQL）数据是相互隔离的**——不能在同一个数据库实例中混合查询两种模型的数据，需要在建库时确定使用哪种图模型。

---

## 架构与高可用

### 集群架构

- 与 [[Aurora]] 类似的**存储计算分离**架构：**1 个主实例（读写）+ 最多 15 个只读副本**，共享同一份跨可用区复制的分布式存储卷
- 存储自动扩展，最大支持 **128 TiB**
- 主实例故障时，Neptune 自动将某个只读副本提升为新主实例，**故障转移通常在 30 秒内完成**

### Neptune Serverless

- 按需自动伸缩计算容量（NCU，Neptune Capacity Unit），根据实际负载在配置的最小/最大范围内**实时扩缩容**，无需手动指定实例规格
- 适合**负载不可预测、有明显波峰波谷、或不想精细管理容量规划**的场景，与 [[Aurora]] Serverless 的设计理念一致

---

## Neptune Analytics（图分析引擎）

- 独立于 Neptune Database 的**内存优化图分析引擎**，专为**大规模图数据的快速分析**（而非事务型的增删改查）设计
- 支持**图算法库**（如 PageRank、最短路径、社区发现、相似度计算），可直接对图数据跑复杂分析而无需导出到其他分析工具
- 支持**停止/启动（Stop/Start）**能力，闲置时段暂停计算资源以降低成本
- 与 **Amazon Bedrock Knowledge Bases** 集成，为生成式 AI 提供 **GraphRAG**（基于图谱的检索增强生成）能力，通过实体关系提升 RAG 答案的准确性和可解释性
- 支持与开源图机器学习库 **GraphStorm** 集成，用于在大规模图上做节点分类、链接预测等图机器学习任务

> **考试陷阱**：**Neptune Database（事务型图数据库）与 Neptune Analytics（图分析引擎）是两个不同的产品**——题目强调"高并发的图数据增删改查"选 Neptune Database；强调"对整张图跑 PageRank/最短路径等分析算法"选 Neptune Analytics。

---

## 安全性

| 机制 | 说明 |
|------|------|
| **VPC 部署** | Neptune 集群只能部署在 [[VPC]] 内，通过安全组控制网络访问，不支持公网直接访问 |
| **静态加密** | 使用 [[KMS]] 管理的密钥对存储数据加密，且**只能在创建集群时启用**，无法对已有未加密集群直接开启 |
| **传输加密** | 客户端与集群间的连接默认使用 SSL/TLS |
| **IAM 数据库认证** | 支持使用 IAM 凭证代替数据库用户名/密码进行身份验证 |
| **标签级访问控制（TBAC）** | 支持在 IAM 策略/SCP 中使用资源标签和 IAM 主体标签作为条件，精细化控制对 Neptune 数据面操作的访问权限 |

---

## 备份与恢复

| 功能 | 说明 |
|------|------|
| **自动快照** | 默认启用，保留期内可做时间点恢复（Point-in-Time Recovery） |
| **手动快照** | 用户手动触发，可长期保留、跨区域复制 |
| **AWS Backup 集成** | 支持通过 AWS Backup 统一管理 Neptune 备份策略，包括**逻辑气隙保管库（Logically Air-Gapped Vault）**，为勒索软件等场景提供额外隔离的恢复点 |

---

## 最新动态（2025-2026）

| 时间 | 更新内容 |
|------|---------|
| 2025-06 | Neptune Analytics 集成开源图机器学习库 **GraphStorm** |
| 2025-07 | Graph Explorer 新增对 **Gremlin/openCypher 原生查询**的支持，可在可视化界面中直接编写查询 |
| 2025-08 | Neptune Analytics 支持**停止/启动（Stop/Start）**能力，闲置时按需暂停计算资源降低成本 |
| 2025-10 | Neptune Database 集成 **GraphStorm**，支持规模化图机器学习 |
| 2026 年初 | **Amazon Bedrock Knowledge Bases GraphRAG**（基于 Neptune Analytics）正式 GA，为生成式 AI 应用提供图谱增强检索 |
| 2026-01 | Neptune Analytics 扩展至 7 个新区域（含首尔、大阪、香港、斯德哥尔摩、巴黎、圣保罗、美国西部N.加州） |
| 2026-01 | Neptune Database 支持 **Graviton3 (R7g) / Graviton4 (R8g)** 实例，多区域上线 |
| 2026-02 | Neptune Analytics 进一步扩展至中东、非洲、加拿大等 7 个区域 |
| 2026-02 | AWS Backup 扩展对 Neptune 的支持至 5 个新区域 |
| 2026-03 | Neptune Database 新增**原生空间数据（Spatial Data）支持**，内置 11 个符合 ISO 13249-3 标准的空间函数，可与 Esri ArcGIS 等 GIS 系统集成 |
| 2026-03 | Neptune Database 落地印度海得拉巴区域 |
| 2026-06 | Neptune 支持 **IPv6 双栈（Dual-Stack）** 网络模式，集群可同时接受 IPv4/IPv6 连接 |
| 2026-07 | Neptune 支持**标签级访问控制（TBAC）** |
| 2026-08 | AWS Backup 逻辑气隙保管库支持扩展至 Neptune 的更多区域 |

> **考试提示**：SAA-C02 考纲一般不会深入考查 Neptune 的最新特性细节，但需要能在场景题中**识别出"深度关联查询/知识图谱/推荐引擎"等关键词并正确选择 Neptune**；GraphRAG、空间数据等新能力属于加分了解内容，非高频考点。

---

## 典型应用场景

| 场景 | 推荐配置 |
|------|---------|
| **社交网络关系分析（好友推荐、影响力分析）** | Neptune Database + Gremlin/openCypher |
| **欺诈检测（识别关联账户网络）** | Neptune Database，多跳关联查询实时识别可疑关系 |
| **推荐引擎（协同过滤、关联推荐）** | Neptune Database，基于用户-商品图遍历生成推荐 |
| **知识图谱、语义搜索** | Neptune Database + SPARQL（RDF 模型） |
| **网络/IT 基础设施拓扑分析** | Neptune Database，建模设备与连接关系 |
| **对整张图做算法级分析（PageRank、社区发现）** | Neptune Analytics |
| **生成式 AI 应用的知识增强检索** | Neptune Analytics + Amazon Bedrock Knowledge Bases（GraphRAG） |
| **负载波动大、不想手动管理容量** | Neptune Serverless |

---

## 考试重点总结

### SAA-C02 高频考点

1. **核心定位**：全托管图数据库，用节点+边原生建模关联关系，遍历关联比关系型数据库的多表 JOIN 更高效
2. **识别关键词**：社交网络、推荐引擎、欺诈检测、知识图谱、多跳/深度关联查询 → 首选 Neptune
3. **两种图模型互相隔离**：Property Graph（Gremlin/openCypher）与 RDF（SPARQL）不能在同一实例中混合查询
4. **架构类似 Aurora**：存储计算分离，1 主 + 最多 15 只读副本，共享分布式存储，故障转移约 30 秒
5. **Neptune Database vs Neptune Analytics**：前者面向事务型增删改查，后者面向图算法级分析（PageRank、最短路径等）
6. **必须部署在 VPC 内**：不支持公网直接访问
7. **静态加密只能在创建时启用**：无法对已存在的未加密集群事后开启
8. **Neptune Serverless**：按 NCU 自动伸缩，适合负载不可预测的场景
9. **支持 GraphRAG**：Neptune Analytics 与 Bedrock Knowledge Bases 集成，为生成式 AI 提供图谱增强检索
10. **存储上限 128 TiB**，最大 15 个只读副本，与 Aurora 的存储扩展逻辑相似

### 场景题解题思路

```
场景分析 → 判断是否用 Neptune
├── "需要多跳/深度关联查询（好友的好友、关联欺诈账户）" → Amazon Neptune
├── "数据之间只有少量外键关联，仍以结构化事务为主" → RDS/Aurora（而非 Neptune）
├── "需要构建知识图谱、语义搜索" → Neptune + SPARQL（RDF 模型）
├── "需要属性图遍历（推荐引擎、社交网络）" → Neptune + Gremlin/openCypher
├── "需要对整张图跑 PageRank/最短路径等分析算法" → Neptune Analytics（而非 Neptune Database）
├── "生成式 AI 应用需要图谱增强的检索增强生成" → Neptune Analytics + Bedrock Knowledge Bases（GraphRAG）
├── "图数据负载波动大，不想手动管理容量" → Neptune Serverless
└── "海量历史数据的聚合分析报表" → 改用 Redshift（而非 Neptune）
```

---

## 最佳实践

1. **判断依据是关联的深度和复杂度**：只有少量外键关联时不要盲目上 Neptune，评估是否真的需要多跳遍历能力
2. **建库前确定图模型**：Property Graph 与 RDF 无法在同一实例内混用，需提前根据查询需求选定
3. **事务型负载用 Neptune Database，分析型负载用 Neptune Analytics**：不要用错产品形态
4. **创建集群时就启用静态加密**：加密无法对已有集群事后追加
5. **负载不可预测时优先 Neptune Serverless**：避免过度配置或频繁手动调整实例规格
6. **只读副本用于读扩展和高可用**：结合应用层读写分离，将读流量分散到只读副本
7. **结合 AWS Backup 统一管理备份策略**：对关键图数据考虑启用逻辑气隙保管库，提升勒索软件等场景下的恢复韧性
8. **生成式 AI 场景优先评估 GraphRAG**：相比纯向量检索，图谱关系能提升 RAG 答案的准确性和可解释性
