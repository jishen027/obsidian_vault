# AWS Glue - 无服务器数据集成与 ETL 服务

> **AWS Glue** 是无服务器（Serverless）的**数据集成与 ETL（Extract, Transform, Load）**服务，负责发现、编目、清洗、转换和移动数据，是整个 AWS 数据分析生态中**元数据管理和数据准备**的核心枢纽——[[Amazon Athena]]、Redshift Spectrum、[[Amazon EMR]] 等分析服务通常共享同一份 **Glue Data Catalog** 作为统一的元数据目录。
>
> 相关文档：[[Amazon Athena]] | [[Amazon EMR]] | [[AWS Lake Formation]] | [[Amazon MSK]] | [[Redshift]] | [[Amazon OpenSearch]] | [[Amazon QuickSight]] | [[S3]] | [[DynamoDB]] | [[RDS]] | [[VPC]]

---

## 核心概念

### 为什么需要 Glue

- **数据准备的痛点**：数据湖中的原始数据往往格式各异（CSV、JSON、日志）、schema 未知、需要清洗转换才能被下游分析服务高效查询；手工编写和运维 ETL 脚本/集群成本高
- **Glue 的核心价值**：**自动发现数据结构（Crawler）+ 无服务器执行 ETL 作业（Spark）+ 统一元数据目录（Data Catalog）**三位一体，让数据从"原始文件"变为"可被 SQL 查询的结构化表"这一过程高度自动化
- **在数据分析管道中的定位**：Glue 通常处于数据管道的**上游/中间环节**——负责发现和准备数据，而非最终的查询（[[Amazon Athena]]/Redshift）或可视化（[[Amazon QuickSight]]）

### Glue 在数据分析服务中的定位（考试要点）

| 服务 | 类型 | 核心能力 | 典型场景 |
|------|------|---------|---------|
| **Glue** | 无服务器 ETL + 元数据目录 | 数据发现、清洗转换、统一元数据管理 | 数据湖 ETL 管道、多服务共享的表结构管理 |
| [[Amazon EMR]] | 托管大数据处理平台 | 运行 Spark/Hadoop 等框架，支持复杂自定义逻辑 | 需要精细控制集群、复杂机器学习/大规模处理管道 |
| [[Amazon Athena]] | 无服务器 SQL 查询 | 直接查询 S3 数据 | 临时性 SQL 分析，依赖 Glue Data Catalog 获取表结构 |
| [[Amazon QuickSight]] | BI 可视化 | 交互式仪表盘 | 数据管道的呈现层终点 |

> **考试陷阱**：**Glue 和 EMR 都能运行 Spark 作业，容易被误认为可以互相替代**——Glue 面向**无服务器、开箱即用的 ETL 场景**，无需管理集群，适合标准化的数据转换任务；EMR 面向**需要精细控制集群配置、运行复杂自定义大数据框架逻辑**的场景（如自定义 Hadoop/Flink 管道、机器学习训练）。题目强调"简单快速的 ETL、无需管理基础设施" → Glue；强调"复杂大数据处理生态、需要精细控制" → EMR。

---

## 三大核心组件

### 1. Glue Data Catalog（数据目录）

- **中心化的元数据仓库**：存储数据库、表的 schema 定义（列名、数据类型、数据在 S3 中的位置等），本身**不存储实际数据**
- **多服务共享**：[[Amazon Athena]]、Redshift Spectrum、[[Amazon EMR]] 上的 Hive/Spark 作业可**共享同一份 Data Catalog**，避免每个服务各自重复定义表结构
- 类似传统 Hive Metastore 的托管版本，是数据湖架构中"让数据可被 SQL 查询"的关键一环

### 2. Glue Crawler（爬虫）

- **自动扫描数据源**（如 S3 中的文件），推断数据的 schema（列名、数据类型、分区结构），并自动在 Data Catalog 中创建/更新表定义
- 省去手动编写 DDL 语句定义表结构的工作，尤其适合 schema 会随时间演变的数据源
- **S3 事件驱动爬取**：可配置为仅扫描新增/修改的子目录，而非每次全量扫描整个存储桶，显著缩短爬取时间

> **考试要点**：题目描述**"需要自动发现 S3 中新增数据的表结构，无需手动维护 Schema"** → 答案是 **Glue Crawler**；这也是 [[Amazon Athena]] 能够"无需预先定义表结构即可查询 S3 数据"背后的实现机制。

### 3. Glue Jobs（ETL 作业）

| 引擎类型 | 说明 | 适用场景 |
|---------|------|---------|
| **Apache Spark** | Glue 的旗舰 ETL 引擎，分布式处理大规模数据转换 | 通用的大规模数据清洗、转换、聚合 |
| **Python Shell** | 运行轻量级 Python 脚本，无需 Spark 集群开销 | 小规模数据处理、简单的脚本化任务 |
| **Glue for Ray** | 面向纯 Python 工作负载（如机器学习预处理）的扩展引擎 | 需要 Ray 生态或非 Spark Python 库的场景 |

- **Glue Studio**：可视化界面，通过拖拽方式构建 ETL 作业流程，自动生成底层 Spark 代码，降低编写 ETL 脚本的门槛
- **Job Bookmark（作业书签）**：自动跟踪已处理过的数据，重复运行同一作业时只处理新增/未处理的数据，避免重复处理

---

## 无服务器架构与容量管理

- **按 DPU（Data Processing Unit）计费**：ETL 作业运行时按实际消耗的计算资源和运行时长付费，无需预置或管理任何服务器/集群
- **Glue Flex**：面向**对启动延迟不敏感、可容忍一定调度等待**的非紧急 ETL 作业，用更低的计算成本运行，相比标准执行方式可节省约 **35%** 成本
- **自动伸缩**：Spark 作业可根据数据量自动调整执行器（Executor）数量，无需手动规划集群规模

> **考试要点**：题目强调"ETL 作业对完成时间不敏感，希望降低运行成本" → 使用 **Glue Flex**；强调"需要 ETL 作业尽快完成、对延迟敏感" → 使用标准执行方式。

---

## Zero-ETL 集成

- AWS 数据生态正推动**零 ETL（Zero-ETL）**理念——通过服务间的原生集成，让数据从源系统（如 [[RDS]]/[[DynamoDB]]）**近实时自动复制**到分析目标（如 Redshift），无需用户编写和运维传统的 ETL 作业
- 减少了从"数据产生"到"数据可分析"之间的延迟和运维负担，适合需要近实时分析源系统数据、又不想搭建复杂管道的场景

---

## Schema Registry（模式注册中心）

- 面向**流式数据**（[[Amazon MSK]]/Kafka、[[Amazon Kinesis]] Data Streams）的 schema 管理服务，支持 **Avro、JSON、Protocol Buffers** 等格式
- 确保生产者和消费者对流数据的 schema 保持一致，schema 演变时提供兼容性校验，避免下游消费者因格式变化而解析失败

---

## 安全性

| 机制 | 说明 |
|------|------|
| **IAM 策略** | 控制对 Glue 资源（作业、爬虫、Data Catalog 表）的访问和执行权限 |
| **[[AWS Lake Formation]] 集成** | 提供比 IAM 更细粒度的**行级/列级访问控制**，可统一管理 Glue、Athena、Redshift Spectrum、EMR 等多个服务对同一份数据的访问权限，且已扩展到读写操作均可控制 |
| **静态加密** | Data Catalog 元数据、ETL 作业的中间数据均可使用 [[KMS]] 密钥加密 |
| **VPC 内作业执行** | Glue Job 可配置在 [[VPC]] 内运行，以访问私有子网中的 RDS 等数据源 |

---

## 典型应用场景

| 场景 | 推荐配置 |
|------|---------|
| **数据湖 ETL 管道：清洗转换 S3 原始数据** | Glue Crawler 发现 schema + Glue Job（Spark）转换 |
| **多个分析服务共享统一表结构** | Glue Data Catalog 作为 Athena/Redshift Spectrum/EMR 的共享元数据源 |
| **自动发现新增数据的 Schema** | Glue Crawler 定期/事件驱动扫描 S3 |
| **非紧急、成本敏感的批量 ETL 作业** | Glue Flex，容忍一定调度延迟换取更低成本 |
| **降低非技术人员构建 ETL 的门槛** | Glue Studio 可视化拖拽构建作业流程 |
| **源数据库到分析目标的近实时同步** | Zero-ETL 集成，避免自建传统 ETL 管道 |
| **流数据的 Schema 一致性管理** | Glue Schema Registry |
| **需要精细控制集群、运行复杂自定义大数据逻辑** | 改用 [[Amazon EMR]]，而非 Glue |

---

## 考试重点总结

### SAA-C02 高频考点

1. **核心定位**：无服务器 ETL + 统一元数据目录，是数据准备和多服务共享 schema 的核心枢纽
2. **三大组件**：Data Catalog（元数据存储）、Crawler（自动发现 schema）、Jobs（执行 ETL，主要基于 Spark）
3. **Glue Data Catalog 不存储实际数据**：只存表结构和数据位置指针，是 Athena/Redshift Spectrum/EMR 的共享元数据基础
4. **Glue Crawler 自动推断 Schema**：省去手动定义 DDL 的工作，支持 S3 事件驱动的增量扫描
5. **Glue vs EMR**：都能跑 Spark，但 Glue 无服务器、开箱即用；EMR 面向需要精细控制集群/复杂自定义逻辑的场景
6. **按 DPU 计费，无需管理服务器**：无服务器架构的核心体现
7. **Glue Flex**：非紧急作业可节省约 35% 成本，以调度延迟换取成本优化
8. **Job Bookmark**：自动跟踪已处理数据，重复运行只处理增量，避免数据重复处理
9. **[[AWS Lake Formation]] 提供细粒度访问控制**：行级/列级权限统一管理多个分析服务对同一数据的访问
10. **Schema Registry 面向流数据**：管理 MSK/Kafka、Kinesis 等流数据的 schema 一致性和兼容性

### 场景题解题思路

```
场景分析 → 判断是否用 Glue
├── "需要清洗转换 S3 原始数据，且不想管理 ETL 集群" → AWS Glue（Spark ETL Job）
├── "需要自动发现/更新 S3 数据的表结构" → Glue Crawler
├── "多个分析服务（Athena/Redshift/EMR）需要共享同一份表元数据" → Glue Data Catalog
├── "ETL 作业对完成时间不敏感，希望降低成本" → Glue Flex
├── "希望通过可视化方式构建 ETL 流程，降低编码门槛" → Glue Studio
├── "重复运行同一 ETL 作业，只想处理新增数据" → 启用 Job Bookmark
├── "需要精细控制集群配置或运行复杂自定义大数据逻辑" → 改用 Amazon EMR（而非 Glue）
├── "需要将源数据库近实时同步到分析目标，且不想自建 ETL" → Zero-ETL 集成
└── "需要管理流数据的 Schema 一致性" → Glue Schema Registry
```

---

## 最佳实践

1. **数据湖架构优先用 Glue 而非自建 ETL 集群**：标准化转换场景下无服务器架构大幅降低运维负担
2. **多个分析服务统一使用同一份 Glue Data Catalog**：避免在 Athena、Redshift Spectrum、EMR 之间重复定义表结构
3. **善用 Crawler 的增量扫描能力**：数据量大时配置 S3 事件驱动爬取，而非每次全量扫描
4. **非紧急批量作业评估 Glue Flex**：在可接受调度延迟的前提下降低 ETL 成本
5. **启用 Job Bookmark 避免重复处理**：尤其是定期运行的增量 ETL 管道
6. **结合 [[AWS Lake Formation]] 做细粒度权限管理**：而非仅依赖 IAM 的粗粒度资源级权限
7. **需要复杂自定义处理逻辑时评估 EMR 而非勉强用 Glue**：避免把 Glue 用于超出其设计场景的复杂大数据处理
8. **流数据管道启用 Schema Registry**：提前发现 schema 变化带来的兼容性问题，避免下游消费者解析失败
