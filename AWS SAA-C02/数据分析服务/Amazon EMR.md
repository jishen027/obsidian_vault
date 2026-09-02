# Amazon EMR - 大数据处理平台

> **Amazon EMR（Elastic MapReduce）** 是托管的大数据处理平台，让用户能够以云原生的方式运行 **Apache Spark、Hadoop、Hive、Presto/Trino、Flink** 等主流大数据处理框架，用于大规模数据的 ETL、机器学习、流处理和交互式分析，而无需自行搭建和运维分布式计算集群。
>
> 相关文档：[[Amazon Athena]] | [[Amazon OpenSearch]] | [[Amazon QuickSight]] | [[AWS Glue]] | [[AWS Lake Formation]] | [[S3]] | [[Redshift]] | [[Amazon Kinesis]] | [[Amazon Managed Service for Apache Flink]] | [[EC2]] | [[EKS]] | [[VPC]]

---

## 核心概念

### 为什么需要 EMR

- **自建大数据集群的痛点**：Hadoop/Spark 生态的集群搭建、节点扩缩容、软件版本管理、故障排查都需要专业的大数据运维能力，自建成本高、周期长
- **EMR 的核心价值**：把开源大数据框架的**部署、扩缩容、故障处理托管化**，同时保留开源生态的灵活性——可直接使用现有的 Spark/Hive 作业代码，无需重写
- **性能优化运行时**：EMR 对 Spark、Trino、Hive 等框架提供**性能优化的运行时（Runtime）**，在保持 100% API 兼容的前提下，处理速度可比开源版本快数倍

### EMR 在数据分析服务中的定位（考试要点）

| 服务 | 类型 | 核心能力 | 典型场景 |
|------|------|---------|---------|
| **EMR** | 托管大数据处理平台 | 运行 Spark/Hadoop/Hive/Flink 等框架，支持复杂 ETL 和机器学习管道 | 大规模数据处理、需要自定义处理逻辑的复杂 ETL、机器学习训练 |
| [[Amazon Athena]] | 无服务器 SQL 查询 | 直接对 S3 数据跑 SQL，无需集群 | 临时性、探索性的 SQL 分析 |
| [[Redshift]] | 数据仓库（OLAP） | 复杂关联查询、BI 报表 | 长期稳定的结构化数据仓库分析 |
| [[Amazon OpenSearch]] | 搜索与分析引擎 | 全文检索、日志实时分析 | 应用搜索、日志/安全监控 |

> **考试陷阱**：题目描述**"需要用 Spark/Hadoop 生态运行复杂的自定义 ETL 逻辑或机器学习训练管道"** → 答案是 **EMR**；若只是"对 S3 数据做临时性 SQL 查询"，Athena 更简单、无需管理集群；若是"结构化数据的长期 BI 报表"，Redshift 更合适。判断依据是**是否需要 Spark/Hadoop 生态的编程灵活性和复杂计算能力**，而非仅仅"处理大数据"这个笼统描述。

---

## 部署模式（考试高频）

| 部署模式 | 说明 | 适用场景 |
|---------|------|---------|
| **EMR on EC2** | 在用户管理的 EC2 集群上运行，可精细控制实例类型、节点配置 | 需要精细控制底层资源、长期稳定运行的大规模集群 |
| **EMR Serverless** | 全托管无服务器，自动预置和伸缩计算资源，无需选择/管理实例 | 负载不可预测、希望免运维、间歇性运行的批处理/流处理作业 |
| **EMR on EKS** | 在 [[EKS]] 集群上以容器化方式运行 Spark 等框架 | 已有 Kubernetes 平台，希望统一容器编排管理大数据和其他工作负载 |

> **考试要点**：**EMR Serverless 无需选择实例类型或管理集群规模**，按实际使用的计算和存储资源计费，作业结束后自动释放资源——题目强调"间歇性大数据作业、不想管理集群容量" → EMR Serverless；强调"需要精细控制节点规格、长期运行的稳定集群" → EMR on EC2。

---

## 集群架构（EMR on EC2）

### 节点类型

| 节点角色 | 职责 |
|---------|------|
| **主节点（Master Node）** | 管理集群，协调任务分配，运行集群管理组件（如 YARN ResourceManager） |
| **核心节点（Core Node）** | 运行任务并存储数据（HDFS），是集群的持久化存储承载者 |
| **任务节点（Task Node）** | 仅运行任务，不存储数据，用于**弹性扩展计算能力**，可安全地随时增减 |

### 弹性伸缩

- **实例组（Instance Groups）/ 实例集群（Instance Fleets）**：定义节点的实例类型和数量，支持**自动伸缩（Auto Scaling）**根据集群负载（如 YARN 内存利用率）动态增减任务节点
- **Spot 实例优化**：任务节点因不存储数据、可随时被中断而不丢数据，是使用 **[[EC2]] Spot 实例**降低大数据处理成本的理想场景；核心节点通常使用按需实例保证数据持久性

> **考试陷阱**：**任务节点适合用 Spot 实例，核心节点不适合**——核心节点存储 HDFS 数据，若使用 Spot 实例被回收会导致数据丢失或集群不稳定；题目问"如何在保证集群稳定性的前提下降低 EMR 成本" → 核心节点用按需/预留实例，任务节点用 Spot 实例。

---

## 支持的处理框架

| 框架 | 用途 |
|------|------|
| **Apache Spark** | 通用的分布式数据处理引擎，支持批处理、流处理、机器学习（MLlib）、图计算，是 EMR 上最常用的框架 |
| **Apache Hadoop（MapReduce + HDFS）** | 经典的批处理框架和分布式文件系统，适合超大规模、离线的批量数据处理 |
| **Apache Hive** | 基于 SQL 语法的数据仓库工具，将 SQL 查询转换为底层 MapReduce/Spark 作业 |
| **Presto / Trino** | 分布式 SQL 查询引擎，支持对多种数据源做交互式低延迟查询 |
| **Apache Flink** | 面向低延迟流处理的框架，适合实时数据管道和事件驱动应用；若不需要与 Spark/Hadoop 共享集群，可考虑改用专为 Flink 设计的 [[Amazon Managed Service for Apache Flink]]（原 Kinesis Data Analytics） |

---

## 存储与数据集成

- **EMRFS（EMR File System）**：让 Spark/Hadoop 作业能像访问 HDFS 一样直接读写 **[[S3]]** 中的数据，实现存储与计算分离——集群可随时终止而不丢失数据，这与自建 Hadoop 集群依赖本地 HDFS 持久化的模式有本质区别
- **[[AWS Glue]] Data Catalog 集成**：EMR 上的 Hive/Spark 作业可共享与 [[Amazon Athena]]、Redshift Spectrum 相同的 Glue Data Catalog 作为统一元数据存储，避免重复定义表结构
- **数据摄入**：常与 [[Amazon Kinesis]]（实时流数据源）、S3（批量数据源）配合，构成完整的数据处理管道

---

## 典型工作流程

1. 原始数据存放在 [[S3]]（数据湖）
2. EMR 集群（EC2/Serverless/EKS）通过 EMRFS 读取 S3 数据，运行 Spark/Hive 作业执行 ETL、清洗、聚合或机器学习训练
3. 处理结果写回 S3（通常转换为 Parquet/ORC 列式格式），或直接加载到 Redshift 供 BI 工具查询
4. 处理完成后，EMR on EC2 集群可**配置为瞬态集群（Transient Cluster）**——作业结束后自动终止，仅为处理过程付费；EMR Serverless 则天然按需付费，无需手动管理集群生命周期

---

## 安全性

| 机制 | 说明 |
|------|------|
| **VPC 部署** | EMR 集群运行在 [[VPC]] 内，通过安全组控制节点间及外部访问 |
| **IAM 角色** | 集群关联服务角色（管理集群资源）和 EC2 实例角色（决定作业可访问的其他 AWS 服务） |
| **静态加密** | 支持对 EMRFS（S3）数据和本地磁盘加密，可使用 KMS 管理密钥 |
| **传输加密** | 支持节点间通信加密（如 Spark 内部通信的 TLS） |
| **Kerberos 身份验证** | 支持与企业级 Kerberos 集成，实现多用户环境下的身份验证和授权 |

---

## 典型应用场景

| 场景 | 推荐配置 |
|------|---------|
| **大规模 ETL：清洗、转换海量原始数据** | EMR + Spark，S3 作为数据湖存储 |
| **机器学习模型训练（大规模特征处理）** | EMR + Spark MLlib |
| **间歇性、负载不可预测的批处理作业** | EMR Serverless，避免管理集群容量 |
| **已有 Kubernetes 平台，统一编排大数据工作负载** | EMR on EKS |
| **降低大规模集群处理成本** | 核心节点用按需实例 + 任务节点用 Spot 实例 |
| **实时流数据处理管道** | EMR + Flink/Spark Streaming，结合 Kinesis 作为数据源 |
| **只需临时对 S3 数据做 SQL 查询** | 改用 [[Amazon Athena]]，而非搭建 EMR 集群 |

---

## 考试重点总结

### SAA-C02 高频考点

1. **核心定位**：托管的 Spark/Hadoop/Hive/Flink 大数据处理平台，用于复杂 ETL、机器学习训练等需要编程灵活性的大规模数据处理
2. **判断依据**：需要 Spark/Hadoop 生态的自定义处理逻辑时选 EMR；仅需 SQL 查询选 Athena/Redshift
3. **三种部署模式**：EMR on EC2（精细控制）、EMR Serverless（免运维、按需）、EMR on EKS（Kubernetes 原生）
4. **节点角色**：主节点管理集群，核心节点存数据（HDFS），任务节点仅计算不存数据
5. **Spot 实例适用于任务节点，不适用于核心节点**：核心节点存储数据，用 Spot 实例可能因中断丢失数据
6. **EMRFS 实现存储计算分离**：数据实际存于 S3，集群可随时终止不丢数据，区别于传统 HDFS 持久化模式
7. **瞬态集群（Transient Cluster）**：作业结束后自动终止，仅为处理时长付费，是控制成本的常见模式
8. **共享 [[AWS Glue]] Data Catalog**：EMR 上的 Hive/Spark 表结构可与 Athena、Redshift Spectrum 共用同一元数据目录
9. **EMR Serverless 无需管理实例/集群规模**：按实际使用资源计费，适合负载不可预测的场景
10. **支持 Kerberos 身份验证**：满足企业多用户环境下的身份验证需求

### 场景题解题思路

```
场景分析 → 判断是否用 EMR
├── "需要用 Spark/Hadoop 运行复杂自定义 ETL 或机器学习训练" → Amazon EMR
├── "只需对 S3 数据做临时性 SQL 查询，无需自定义处理逻辑" → 改用 Amazon Athena（而非 EMR）
├── "负载不可预测，不想管理集群容量" → EMR Serverless
├── "已有 Kubernetes 平台，需要统一编排大数据工作负载" → EMR on EKS
├── "需要精细控制节点规格、长期稳定运行" → EMR on EC2
├── "如何在保证数据安全的前提下降低集群成本" → 核心节点用按需/预留实例，任务节点用 Spot 实例
├── "作业结束后不想继续为闲置集群付费" → 配置瞬态集群，作业完成自动终止
└── "多个分析服务需要共享同一份表元数据" → 统一使用 [[AWS Glue]] Data Catalog
```

---

## 最佳实践

1. **需要复杂计算逻辑时才选择 EMR**：仅做 SQL 查询优先用 Athena/Redshift，避免过度设计
2. **核心节点用按需/预留实例，任务节点用 Spot 实例**：兼顾数据持久性和成本优化
3. **负载不可预测的批处理作业优先 EMR Serverless**：避免手动管理集群规模的运维负担
4. **利用 EMRFS 实现存储计算分离**：数据落在 S3 而非本地 HDFS，集群可按需创建/销毁
5. **短期批处理作业配置为瞬态集群**：作业完成后自动终止，避免为闲置集群持续付费
6. **多个分析服务统一使用 [[AWS Glue]] Data Catalog**：避免在 EMR、Athena、Redshift Spectrum 之间重复定义表结构
7. **处理结果转换为列式格式（Parquet/ORC）**：便于后续 Athena/Redshift Spectrum 高效查询
8. **已有 Kubernetes 运维体系时评估 EMR on EKS**：统一容器编排平台，减少多套集群管理的复杂度
