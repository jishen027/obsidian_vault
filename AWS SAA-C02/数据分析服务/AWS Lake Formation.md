# AWS Lake Formation - 数据湖治理与权限中心

> **AWS Lake Formation** 是用于**快速构建、保护和管理数据湖**的服务，在 **[[AWS Glue]] Data Catalog** 的元数据基础之上，提供统一的**细粒度访问控制（Fine-Grained Access Control, FGAC）**层——按数据库、表、列、行甚至单元格级别授权，并让这份权限定义**同时对 [[Amazon Athena]]、Redshift Spectrum、[[Amazon EMR]] 等多个分析服务生效**，而无需在每个服务中分别配置权限。
>
> 相关文档：[[AWS Glue]] | [[Amazon Athena]] | [[Amazon EMR]] | [[Redshift]] | [[S3]] | [[IAM]] | [[Amazon QuickSight]]

---

## 核心概念

### 为什么需要 Lake Formation

- **IAM 权限粒度的局限**：IAM 策略能控制"谁能访问哪个 S3 桶/前缀"，但**无法感知数据的表结构**——做不到"允许某用户查询某张表，但屏蔽其中的薪资列"这类细粒度授权，只能靠应用层逻辑或数据视图变通实现
- **多服务权限管理的重复劳动**：若 Athena、Redshift Spectrum、EMR 都需要访问同一份数据湖，传统做法是在每个服务里各自配置权限，容易出现口径不一致、遗漏授权的问题
- **Lake Formation 的核心价值**：**权限定义一次，多服务统一生效**——基于 Glue Data Catalog 中的表定义，在数据库/表/列/行/单元格级别集中定义授权规则，所有接入的分析服务查询时自动遵循这份权限

### Lake Formation 在数据分析服务中的定位（考试要点）

| 服务 | 类型 | 核心能力 | 典型场景 |
|------|------|---------|---------|
| **Lake Formation** | 数据湖治理与权限中心 | 细粒度访问控制（列/行/单元格级）、跨服务统一授权 | 数据湖需要精细化权限管控、多团队/多租户共享同一数据湖 |
| [[AWS Glue]] | ETL + 元数据目录 | 数据发现、清洗转换、Data Catalog | Lake Formation 依赖的底层元数据和 ETL 基础设施 |
| [[IAM]] | 身份与访问管理 | 粗粒度的资源级权限（能否访问某服务/资源） | 控制"谁能使用 Lake Formation/Athena 等服务"这一层面的权限 |

> **考试陷阱**：题目描述**"需要限制不同用户只能查询表中的特定列，或只能看到特定行的数据"** → 答案是 **Lake Formation** 的细粒度访问控制，而非试图用 **IAM 策略**实现——IAM 策略的最小粒度是资源（如 S3 对象/前缀），**不理解表的列、行结构**；只有 Lake Formation 能在 SQL 查询层面感知并过滤列/行。

---

## 访问控制的四个粒度（考试高频）

| 粒度 | 说明 |
|------|------|
| **表级（Table-Level）** | 授权用户能否访问整张表 |
| **列级（Column-Level）** | 允许/屏蔽表中的特定列（如隐藏薪资、身份证号等敏感字段） |
| **行级（Row-Level）** | 基于行内容过滤（如只允许查看某部门/某区域的数据行） |
| **单元格级（Cell-Level）** | 列级 + 行级的组合，实现"某些用户只能看到特定行的特定列" |

- 权限授予对象可以是 **IAM 用户/角色**，也可以通过 **基于标签的访问控制（LF-TBAC, Tag-Based Access Control）**基于资源标签批量授权，避免为大量表逐一配置权限

> **考试要点**：**LF-TBAC 通过标签而非逐表授权**——给多张表打上相同的业务标签（如 `department=finance`），一次性对该标签授权即可覆盖所有匹配的表，新增符合标签的表会自动继承权限，大幅简化大规模数据湖的权限管理。

---

## 权限模型：Lake Formation 权限 vs IAM 权限

- Lake Formation 引入了独立于 IAM 的**数据权限层（Data Permissions）**，包括 `SELECT`、`INSERT`、`DELETE`、`DESCRIBE` 等针对 Data Catalog 资源（数据库、表、列）的授权动词
- **用户访问数据仍需同时满足两层权限**：
  1. **IAM 层面**：该用户是否有权限调用 Athena/Redshift Spectrum/EMR 等服务的 API
  2. **Lake Formation 层面**：该用户对具体的数据库/表/列/行是否有 `SELECT` 等数据权限
- 这种**双层权限模型**让"服务访问权"和"数据访问权"解耦，便于分别由不同团队（云平台团队管理 IAM，数据治理团队管理 Lake Formation）负责

---

## 数据摄入与治理

- **Blueprints（蓝图）**：提供预置的数据摄入工作流模板，简化从关系型数据库、日志等来源批量导入数据到数据湖的过程
- **数据自动分类和标记**：Lake Formation 可结合 Glue Crawler 识别的元数据，辅助进行敏感数据分类
- **审计与监控**：数据访问活动可与 [[CloudTrail]] 等审计工具集成，追踪谁在何时访问了哪些数据

---

## 与分析服务的集成（考试高频）

| 集成服务 | 说明 |
|---------|------|
| **[[Amazon Athena]]** | 查询时自动遵循 Lake Formation 定义的列/行级权限，无需在 Athena 内单独配置 |
| **Redshift Spectrum** | 通过 Lake Formation 授权后，可对 S3 数据湖中的表实现细粒度访问控制 |
| **[[Amazon EMR]]（EMR on EC2 / EMR Serverless / EMR on EKS）** | Spark 作业可强制执行 Lake Formation 定义的数据库/表/列/行/单元格级权限策略，且已扩展支持 **Hudi、Delta Lake、Iceberg** 等主流数据湖表格式 |

> **考试要点**：Lake Formation 的核心卖点是**"一次授权、处处生效"**——题目强调"多个分析服务需要对同一份数据湖数据执行一致的细粒度权限策略"时，Lake Formation 是标准答案，而非在 Athena、EMR 里分别设置权限。

---

## 与 AWS Glue 的关系

- Lake Formation **构建在 [[AWS Glue]] Data Catalog 之上**——Glue 负责发现和维护表的元数据（schema），Lake Formation 负责在这些元数据对象上叠加细粒度权限
- 两者常被放在同一治理流程中：**Glue Crawler 发现并编目数据 → Lake Formation 对编目后的表/列/行定义访问权限 → Athena/EMR/Redshift Spectrum 查询时自动遵循权限**

---

## 典型应用场景

| 场景 | 推荐配置 |
|------|---------|
| **多团队/多租户共享同一数据湖，需按角色隔离数据** | Lake Formation 列级/行级权限 |
| **敏感字段（薪资、PII）需要对特定用户隐藏** | Lake Formation 列级访问控制 |
| **不同部门只能查看各自业务范围的数据行** | Lake Formation 行级访问控制 |
| **需要对大量表批量、一致地授权，且新表自动继承规则** | LF-TBAC（基于标签的访问控制） |
| **多个分析服务（Athena/EMR/Redshift Spectrum）需要统一权限口径** | 在 Lake Formation 中集中定义一次权限 |
| **需要审计谁访问过哪些敏感数据** | Lake Formation 权限 + CloudTrail 审计 |
| **仅需要控制"谁能使用某个 AWS 服务"这类粗粒度权限** | IAM 策略已足够，无需引入 Lake Formation |

---

## 考试重点总结

### SAA-C02 高频考点

1. **核心定位**：数据湖的细粒度访问控制与治理中心，构建在 Glue Data Catalog 之上
2. **判断依据**：需要**列/行/单元格级别**的数据权限控制时选 Lake Formation；仅需资源级粗粒度权限用 IAM 即可
3. **四种访问控制粒度**：表级、列级、行级、单元格级（列+行组合）
4. **一次授权、处处生效**：权限集中定义在 Lake Formation，Athena/EMR/Redshift Spectrum 查询时统一遵循，无需分别配置
5. **双层权限模型**：访问数据需同时满足 IAM（能否调用服务）+ Lake Formation（能否访问具体数据）两层授权
6. **LF-TBAC 基于标签的访问控制**：按标签批量授权，避免逐表配置，新表自动继承匹配标签的权限
7. **与 Glue 的关系**：Glue 负责元数据发现和编目，Lake Formation 负责在此基础上叠加权限策略
8. **EMR 集成覆盖三种部署模式**：EMR on EC2、EMR Serverless、EMR on EKS 均可强制执行 Lake Formation 权限
9. **支持主流数据湖表格式**：Hudi、Delta Lake、Iceberg 均可应用 Lake Formation 的细粒度权限
10. **Blueprints 简化数据摄入**：提供预置模板，加速从数据库/日志等来源批量导入数据湖

### 场景题解题思路

```
场景分析 → 判断是否用 Lake Formation
├── "需要限制用户只能看到表中特定列（如隐藏薪资字段）" → Lake Formation 列级访问控制
├── "不同部门/租户只能查看各自范围的数据行" → Lake Formation 行级访问控制
├── "多个分析服务需要对同一数据湖执行一致的权限策略" → 在 Lake Formation 集中定义一次权限
├── "需要对大量表批量授权，且新表自动继承规则" → LF-TBAC（基于标签的访问控制）
├── "仅需要控制谁能调用某个 AWS 服务的 API" → IAM 策略已足够，无需 Lake Formation
├── "需要简化从关系型数据库批量导入数据湖的流程" → Lake Formation Blueprints
├── "需要审计谁访问过哪些敏感数据" → Lake Formation 权限 + CloudTrail
└── "EMR Spark 作业需要遵循细粒度数据权限" → EMR（EC2/Serverless/EKS）+ Lake Formation 集成
```

---

## 最佳实践

1. **细粒度数据权限优先用 Lake Formation，而非试图用 IAM 变通实现**：IAM 不理解表的列/行结构
2. **大规模数据湖优先使用 LF-TBAC**：避免为每张新表逐一手动授权，降低管理成本和遗漏风险
3. **多个分析服务统一通过 Lake Formation 授权**：避免在 Athena、EMR、Redshift Spectrum 中重复且可能不一致地配置权限
4. **敏感字段默认屏蔽，按需最小授权**：对薪资、PII 等敏感列采用白名单式授权而非默认可见
5. **结合 Glue Crawler 的自动发现能力**：新数据接入后先由 Glue 编目，再由 Lake Formation 定义权限，形成完整治理流程
6. **善用 Blueprints 加速标准化数据摄入**：减少手工搭建数据湖导入管道的工作量
7. **权限变更结合 CloudTrail 审计**：确保数据访问权限的变更和实际访问行为可追溯
8. **EMR 场景确认部署模式已启用 Lake Formation 集成**：EC2/Serverless/EKS 三种模式的集成方式和支持范围需分别核实
