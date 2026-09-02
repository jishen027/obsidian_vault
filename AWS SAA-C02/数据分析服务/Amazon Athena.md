# Amazon Athena - 无服务器交互式查询服务

> **Amazon Athena** 是无服务器（Serverless）的交互式查询服务，基于开源 **Presto** 引擎，允许用户直接对 **S3** 中的数据使用**标准 SQL** 进行查询分析，**无需 ETL、无需将数据加载到数据库、无需管理任何基础设施**，按实际扫描的数据量付费。
>
> 相关文档：[[S3]] | [[Redshift]] | [[Amazon OpenSearch]] | [[Amazon EMR]] | [[Amazon QuickSight]] | [[AWS Glue]] | [[AWS Lake Formation]] | [[DynamoDB]] | [[RDS]] | [[Aurora]] | [[Amazon Kinesis]] | [[KMS]]

---

## 核心概念

### 为什么需要 Athena

- **数据湖分析的痛点**：S3 中存储着海量原始数据（日志、点击流、CSV/JSON/Parquet 文件），若要用 SQL 分析，传统做法需要先 ETL 加载到数据库，耗时且增加存储成本
- **Athena 的核心价值**：**直接对 S3 中的原始文件跑 SQL 查询**，无需数据搬迁，查询结束后立即释放资源，真正做到"用完即走"，零基础设施管理
- **Schema-on-Read**：数据本身保持原始格式存放在 S3，只在查询时通过**元数据目录（Data Catalog）**定义表结构，而非像传统数据库那样"写入时就要求固定 schema"

### Athena 在数据分析服务中的定位（考试要点）

| 服务 | 类型 | 数据位置 | 计费方式 | 典型场景 |
|------|------|---------|---------|---------|
| **Athena** | 无服务器 SQL 查询引擎 | 数据留在 S3，查询时读取 | 按**扫描的数据量**计费 | 临时性、探索性查询，日志/数据湖即席分析 |
| [[Redshift]] | 数据仓库（OLAP 集群） | 数据加载进 Redshift 集群 | 按**集群运行时间**计费 | 高频、复杂的 BI 报表和长期数据仓库需求 |
| Redshift Spectrum | Redshift 的 S3 查询扩展 | 数据留在 S3，通过 Redshift 集群查询 | 按扫描的数据量计费，需要现有 Redshift 集群 | 已有 Redshift 集群，偶尔需要联合查询 S3 冷数据 |
| [[DynamoDB]] | NoSQL 键值/文档 | 数据存于 DynamoDB | 按容量/请求计费 | 高并发、低延迟的主键查询 |

> **考试陷阱**：题目描述**"需要对 S3 中的日志文件做临时性、一次性的 SQL 分析，且没有现成的数据仓库集群"** → 答案是 **Athena**；若题目强调**"已经有 Redshift 集群，需要偶尔联合查询 S3 中的冷数据"** → 答案是 **Redshift Spectrum**（复用现有集群，而非额外引入 Athena）；若强调**"高频、复杂的多表关联分析报表，长期稳定运行"** → 答案是 **Redshift** 全量加载数据到集群中查询，而非依赖 Athena 反复扫描原始文件。三者的判断核心是**查询频率、数据是否已有集群承载、以及是否需要长期稳定的低延迟分析**。

---

## 工作机制

### 与 AWS Glue Data Catalog 集成

- Athena **不存储数据**，也**不存储表结构**——表定义（Schema）保存在 **[[AWS Glue]] Data Catalog** 中，本质是一份指向 S3 数据位置的元数据映射
- 可通过 **Glue Crawler** 自动爬取 S3 中的数据并推断出表结构，省去手动定义 DDL 的工作
- 多个分析服务（Athena、Redshift Spectrum、[[Amazon EMR]]）可**共享同一份 Glue Data Catalog**，避免重复定义元数据

### 查询性能优化

| 优化手段 | 效果 |
|---------|------|
| **列式存储格式（Parquet / ORC）** | 相比 CSV/JSON 等行式格式，可大幅减少扫描的数据量，直接降低查询成本和延迟 |
| **数据分区（Partitioning）** | 按日期、区域等维度对 S3 数据分目录存放，查询时 Athena 可跳过不相关分区，减少扫描范围 |
| **数据压缩** | 压缩后的文件（如 Snappy、GZIP）进一步减少扫描的数据量 |
| **列裁剪（Columnar Projection）** | 只读取查询涉及的列，而非整行数据，列式格式下效果尤为明显 |

> **考试要点**：Athena **按扫描的数据量计费**（每 TB 固定费用），因此**列式存储 + 分区 + 压缩**是 SAA-C02 场景题中"如何降低 Athena 查询成本"的标准答案组合——把 CSV 转换成 Parquet 并按日期分区，通常能将扫描量降低一个数量级以上。

### 联合查询（Federated Query）

- 通过**数据源连接器（Connector）**，Athena 可查询 S3 之外的数据源——包括 [[RDS]]/[[Aurora]]、[[DynamoDB]]、DocumentDB 等关系型/非关系型数据库，以及本地数据源
- 支持在**一条 SQL 中联合查询多个异构数据源**，无需先把数据统一搬迁到 S3
- 官方及社区提供数十种连接器，覆盖主流数据库和第三方云服务

### 容量预留（Capacity Reservations）

- 支持按 **DPU（Data Processing Unit）**预留专属计算容量，适合对**查询优先级、并发控制**有严格要求的交互式工作负载
- 相比默认的按需（On-Demand）模式，容量预留提供更可预测的性能和成本，适合有一定基础查询量的场景

---

## 安全性

| 机制 | 说明 |
|------|------|
| **IAM 策略** | 控制哪些身份可以对 Athena 执行查询、访问哪些 [[AWS Glue]] Data Catalog 表 |
| **S3 存储桶策略/ACL** | Athena 查询的底层权限最终取决于对 S3 数据的读取权限 |
| **静态加密** | 查询结果默认写入 S3 的结果存储桶，可使用 [[KMS]] 密钥加密查询结果和源数据 |
| **传输加密** | 客户端与 Athena 的连接默认通过 TLS |
| **细粒度访问控制（Fine-Grained Access Control）** | 结合 **[[AWS Lake Formation]]** 可实现基于列、行级别的数据访问权限控制 |

---

## 典型应用场景

| 场景 | 推荐配置 |
|------|---------|
| **S3 日志/点击流的临时性即席分析** | Athena 直接查询，无需搭建数据仓库 |
| **数据湖架构中的 SQL 查询层** | S3 + [[AWS Glue]] Data Catalog + Athena |
| **降低查询成本和延迟** | 将原始数据转换为 Parquet/ORC，并按常用查询维度分区 |
| **需要联合查询 S3 与关系型/NoSQL 数据库** | Athena Federated Query |
| **已有 Redshift 集群，偶尔查询 S3 冷数据** | Redshift Spectrum（而非额外引入 Athena） |
| **高频、复杂的长期 BI 报表分析** | 改用 [[Redshift]]，数据全量加载进集群 |
| **对查询优先级/并发有严格要求的交互式负载** | Athena Capacity Reservations |

---

## 考试重点总结

### SAA-C02 高频考点

1. **核心定位**：无服务器 SQL 查询引擎，直接查询 S3 数据，无需 ETL、无需管理基础设施
2. **计费方式**：按**扫描的数据量**付费，而非按集群运行时间——这是与 Redshift 的关键区别
3. **Schema-on-Read**：表结构保存在 **Glue Data Catalog** 中，Athena 本身不存储数据也不存储 schema
4. **降低成本三件套**：列式存储（Parquet/ORC）+ 数据分区 + 压缩，可大幅减少扫描量和查询成本
5. **Athena vs Redshift Spectrum**：都能查 S3，但 Spectrum 依赖现有 Redshift 集群，Athena 完全无服务器、独立运行
6. **Athena vs Redshift**：临时性/探索性/低频查询选 Athena；高频、复杂、长期稳定的数仓分析选 Redshift
7. **联合查询（Federated Query）**：通过连接器可查询 RDS/DynamoDB 等 S3 之外的数据源，一条 SQL 跨异构数据源
8. **Glue Crawler 自动推断 Schema**：省去手动定义表结构的工作，多个分析服务可共享同一份 Data Catalog
9. **容量预留（DPU）**：面向需要可预测性能和并发控制的交互式工作负载
10. **权限最终取决于 S3 权限**：IAM 策略 + S3 Bucket Policy 共同决定查询能访问哪些数据

### 场景题解题思路

```
场景分析 → 判断是否用 Athena
├── "需要对 S3 中的日志/数据做临时性 SQL 分析，无现成数仓" → Amazon Athena
├── "已有 Redshift 集群，偶尔需要联合查询 S3 冷数据" → Redshift Spectrum（而非新建 Athena 方案）
├── "高频、复杂的多表关联 BI 报表，长期稳定运行" → 改用 Redshift（数据加载进集群）
├── "查询成本过高，扫描数据量过大" → 转换为 Parquet/ORC + 按查询维度分区 + 压缩
├── "需要在一条 SQL 中联合查询 S3 与 RDS/DynamoDB" → Athena Federated Query
├── "希望自动推断 S3 数据的表结构" → [[AWS Glue]] Crawler + Data Catalog
├── "交互式查询需要保证并发和优先级" → Athena Capacity Reservations（DPU 预留）
└── "需要按行/列级别做细粒度数据访问控制" → 结合 [[AWS Lake Formation]]
```

---

## 最佳实践

1. **原始数据优先转换为列式格式**：Parquet/ORC 相比 CSV/JSON 能大幅降低扫描量和查询成本
2. **按常用查询维度做数据分区**：如按日期分区，让 Athena 能跳过无关分区，减少扫描范围
3. **启用压缩进一步降低成本**：压缩格式在列式存储基础上叠加，进一步减少扫描的数据量
4. **善用 [[AWS Glue]] Crawler 自动维护 Schema**：数据结构变化频繁时，避免手动维护 DDL 的负担
5. **临时性查询优先 Athena，长期高频分析优先 Redshift**：按查询频率和复杂度选型，避免过度设计
6. **已有 Redshift 集群时优先复用 Spectrum**：避免为偶发的 S3 查询额外引入新的分析服务
7. **跨数据源分析场景使用 Federated Query**：避免先手动 ETL 汇总到 S3 再查询的额外开销
8. **查询结果桶单独配置生命周期策略**：避免 Athena 查询结果长期堆积产生不必要的存储成本
