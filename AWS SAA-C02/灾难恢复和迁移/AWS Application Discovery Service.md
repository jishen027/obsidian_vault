# AWS Application Discovery Service - 应用发现服务

> **AWS Application Discovery Service** 是迁移**规划**阶段使用的发现工具——自动收集本地数据中心服务器的**硬件规格、资源利用率、运行进程和服务器间网络连接**等信息，帮助企业在大规模迁移前建立**准确的资产清单和应用依赖关系图**，而不是依赖手工整理、往往不完整的服务器清单。它本身**不迁移任何数据或服务器**，收集到的数据会汇总到 **AWS Migration Hub**，为后续实际执行迁移的工具（[[Database Migration Service]]、[[AWS Elastic Disaster Recovery]] 等）提供规划依据。
>
> 相关文档：[[AWS Elastic Disaster Recovery]] | [[Database Migration Service]] | [[VMware Cloud on AWS]] | [[AWS Backup]] | [[Disaster Recovery On AWS]] | [[EC2]] | [[VPC]]

---

## 核心概念

### 为什么需要 Application Discovery Service

- **大规模迁移最容易低估的风险**：企业往往**不清楚自己到底有多少台服务器、这些服务器之间如何互相调用**——手工盘点耗时且容易遗漏，一旦迁移时漏掉某个有隐藏依赖的服务器，会导致迁移后应用故障
- **核心价值**：自动化、持续地采集服务器规格、利用率和**网络连接数据**，让迁移团队能够基于真实数据做**应用分组（哪些服务器应该一起迁移）**、**迁移波次规划（Wave Planning）**和**目标实例规格（Right-Sizing）**决策

### 两种数据采集方式（考试高频对比）

| 采集方式 | 部署形式 | 采集内容 | 是否支持网络依赖关系 |
|---------|---------|---------|---------------------|
| **无代理发现（Agentless Discovery）** | 在 **VMware vCenter** 环境中部署一个轻量级虚拟设备（Agentless Collector），无需在每台虚拟机上安装任何软件 | 服务器规格（CPU/内存/磁盘）、资源利用率 | **不支持**——只能看到虚拟化层面的资源信息，无法感知进程级的网络连接 |
| **基于代理发现（Agent-based Discovery）** | 在**每台服务器**（物理机或虚拟机）上安装 **Application Discovery Agent** | 更详细的系统配置、运行中的进程、**服务器间的网络连接数据** | **支持**——是唯一能够精确绘制"哪个应用调用了哪个应用"依赖关系图的方式 |

> **考试陷阱**：**"需要绘制应用间的网络依赖关系，确定迁移分组和先后顺序"这类需求，必须使用基于代理的发现，而非无代理发现**——无代理方式虽然部署简单（不用碰每台虚拟机），但只能拿到"这台机器配置多大、用了多少资源"，看不到"这台机器和哪些其他机器有网络通信"；题目描述"不方便在每台服务器上安装软件，只需要了解资源利用率" → 无代理发现；题目描述"需要搞清楚应用之间复杂的调用关系，以便安全分批迁移" → 基于代理发现。

### 采集的数据类型

- **静态配置**：CPU 核数、内存大小、磁盘容量、操作系统版本
- **性能/利用率历史**：CPU、内存、磁盘、网络的历史使用率趋势，用于迁移后**右规格化（Right-Sizing）**推荐目标 EC2 实例类型，避免"照搬本地配置"导致过度配置或性能不足
- **网络连接（仅 Agent-based）**：记录服务器之间实际发生的网络连接，用于反推**应用依赖关系**

---

## 与 AWS Migration Hub 的关系

- Application Discovery Service 采集到的数据会**自动导入 Migration Hub**，在 Migration Hub 中可以将发现的服务器**分组为"应用"**，并在一个统一的界面跟踪后续使用 [[Database Migration Service]]、[[AWS Elastic Disaster Recovery]]/Application Migration Service 等工具执行迁移的**进度**
- **考试要点**：**Application Discovery Service 负责"发现和规划"，Migration Hub 负责"统一跟踪迁移进度"，两者是迁移生命周期中前后相接的不同阶段**，不要混淆两者的职责边界。

---

## 与相关迁移服务的边界（考试陷阱）

| 服务 | 所处阶段 | 核心职责 |
|------|---------|---------|
| **AWS Application Discovery Service** | **规划阶段（迁移前）** | 采集服务器清单、利用率、网络依赖，**不迁移任何数据** |
| **AWS Migration Evaluator** | **规划阶段（迁移前）** | 基于采集到的数据（同样可用 Agentless Collector）生成**成本对比和商业论证（TCO/Business Case）**，聚焦"迁移到 AWS 能省多少钱"而非应用依赖关系 |
| **[[Database Migration Service]]** | **执行阶段** | 实际迁移**数据库内的数据** |
| **[[AWS Elastic Disaster Recovery]] / Application Migration Service** | **执行阶段** | 实际迁移**整台服务器**并转换为**原生 EC2 实例**（块级复制 + 按需启动） |
| **[[VMware Cloud on AWS]]** | **执行阶段** | 把 VMware 虚拟机**原样搬迁**（零改造，保留 VMDK 格式和 vCenter 工具链），不转换为原生 EC2 |

> **考试陷阱**：**Application Discovery Service 与 Migration Evaluator 都可能用到类似的 Agentless Collector 采集工具，但产出目的完全不同**——题目描述"需要绘制应用依赖关系，规划迁移分组和顺序" → **Application Discovery Service**；题目描述"需要向管理层证明迁移上云能节省多少成本，生成数据驱动的商业论证" → **Migration Evaluator**；题目描述"规划阶段已完成，现在需要真正把服务器/数据库迁移过去" → 改用 [[Database Migration Service]] 或 [[AWS Elastic Disaster Recovery]]，Application Discovery Service 本身不具备迁移执行能力。

---

## 典型应用场景

| 场景 | 推荐配置 |
|------|---------|
| **大规模迁移前需要摸清本地服务器的真实资产清单** | AWS Application Discovery Service |
| **不方便在每台服务器安装软件，只需了解资源规格和利用率** | 无代理发现（Agentless Collector，适用于 VMware 环境） |
| **需要绘制应用之间的网络依赖关系，规划迁移分组和顺序** | 基于代理发现（Application Discovery Agent） |
| **需要向管理层论证迁移上云的成本效益** | 改用 AWS Migration Evaluator（而非 Application Discovery Service） |
| **发现阶段完成后需要统一跟踪多个迁移工具的执行进度** | 数据导入 AWS Migration Hub 集中管理 |
| **规划完成，需要实际迁移数据库/整台服务器** | 改用 [[Database Migration Service]] / [[AWS Elastic Disaster Recovery]]（执行阶段工具） |

---

## 考试重点总结

### SAA-C02 高频考点

1. **Application Discovery Service 只负责发现和规划，不迁移任何数据**：是迁移生命周期中"规划阶段"的工具，而非"执行阶段"
2. **两种采集方式**：无代理发现（VMware 环境，仅规格/利用率）、基于代理发现（每台服务器安装 Agent，含网络依赖数据）
3. **只有基于代理发现才能采集网络连接数据**：绘制应用依赖关系图必须选择这种方式
4. **无代理发现部署更简单，但看不到进程级网络依赖**：适合只需要资源利用率数据的场景
5. **采集的利用率数据用于迁移后的右规格化推荐**：避免迁移到 AWS 后配置过度或不足
6. **发现的数据自动汇入 AWS Migration Hub**：在 Hub 中分组为"应用"并跟踪迁移进度
7. **与 Migration Evaluator 的区别在于产出目的**：Discovery Service 聚焦技术依赖关系，Migration Evaluator 聚焦成本商业论证
8. **与 Database Migration Service/Elastic Disaster Recovery 的区别在于所处阶段**：前者是规划工具，后两者是执行工具
9. **迁移分组/波次规划高度依赖准确的依赖关系数据**：手工盘点容易遗漏隐藏依赖，导致迁移后应用故障
10. **Application Discovery Service 本身不产生任何迁移执行动作**：题目若描述"需要真正搬迁"，答案必须转向执行阶段的工具

### 场景题解题思路

```
场景分析 → 判断 Application Discovery Service 相关配置
├── "需要摸清本地服务器资产清单和依赖关系再规划迁移" → AWS Application Discovery Service
├── "不方便装 Agent，只需资源利用率数据" → 无代理发现（Agentless Collector）
├── "需要绘制应用间网络依赖，规划迁移分组" → 基于代理发现（Application Discovery Agent）
├── "需要向管理层论证迁移上云的成本收益" → 改用 Migration Evaluator
├── "需要统一跟踪多个迁移工具的执行进度" → 数据导入 Migration Hub
└── "规划完成，需要真正执行迁移" → Database Migration Service（数据库）/ Elastic Disaster Recovery（整机）
```

---

## 最佳实践

1. **大规模迁移前务必先做发现，而非直接凭经验估算服务器清单**：避免遗漏隐藏依赖导致迁移后应用故障
2. **需要应用依赖关系图时选择基于代理发现，而非图省事只用无代理方式**：无代理方式无法满足依赖关系规划需求
3. **善用采集到的利用率历史数据做右规格化**：不要简单照搬本地服务器配置作为目标 EC2 实例类型
4. **将发现数据导入 Migration Hub 统一管理**：避免规划数据和实际迁移进度分散在不同工具中难以追踪
5. **需要成本论证材料时同时评估 Migration Evaluator**：不要试图用 Application Discovery Service 的数据自行拼凑商业论证
6. **明确规划工具与执行工具的边界**：完成发现和分组规划后，及时切换到 Database Migration Service/Elastic Disaster Recovery 等执行阶段工具，不要停留在规划阶段