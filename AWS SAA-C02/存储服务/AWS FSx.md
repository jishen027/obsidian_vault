# Amazon FSx

> **Amazon FSx** 是一组**完全托管的第三方文件系统**服务，让用户可以在 AWS 上运行熟悉的商业和开源文件系统（Windows、Lustre、NetApp ONTAP、OpenZFS），而无需自行搭建和维护底层基础设施。
>
> 相关文档：[[AWS EFS]] | [[EBS]] | [[S3]] | [[VPC]] | [[Storage Service]] | [[AWS Directory Service]]

---

## 核心概念

### FSx 产品家族对比（考试重点）

| 文件系统 | 协议 | 原生平台 | 核心场景 |
|---------|------|---------|---------|
| **FSx for Windows File Server** | SMB | Windows | Windows 应用迁移、AD 集成的企业文件共享 |
| **FSx for Lustre** | Lustre / POSIX | Linux | 高性能计算 (HPC)、机器学习、视频处理 |
| **FSx for NetApp ONTAP** | NFS / SMB / iSCSI | 多平台 | 从本地 NetApp 迁移、多协议统一存储 |
| **FSx for OpenZFS** | NFS | Linux | 需要 ZFS 特性（快照、克隆）的 Linux 工作负载 |

### 为什么选择 FSx 而不是 EFS

| 特性 | [[AWS EFS]] | Amazon FSx |
|------|----------|-----------|
| **协议** | 仅 NFS（Linux） | SMB（Windows）、Lustre、NFS、iSCSI（因产品而异） |
| **底层文件系统** | AWS 自研的 NFS 实现 | 直接使用第三方原生文件系统（如 Windows Server、Lustre） |
| **适用操作系统** | 仅 Linux | Windows（FSx for Windows）、Linux（Lustre/OpenZFS）、多平台（ONTAP） |
| **AD 集成** | 不支持 | FSx for Windows 原生支持 Microsoft Active Directory |
| **高性能计算** | 不擅长 | FSx for Lustre 专为 HPC / ML 设计 |

> **一句话总结**：需要 **Linux + NFS** 共享存储 → 首选 [[AWS EFS]]；需要 **Windows 原生文件系统 (SMB)** 或 **高性能计算 (HPC/ML)** → 选择对应的 **FSx** 产品。

---

## FSx for Windows File Server

### 核心特性

- 提供**原生 Windows 文件系统**，完全兼容 **SMB 协议**和 **NTFS**
- 与 **Microsoft Active Directory (AD)** 原生集成：
  - 支持 [[AWS Directory Service]]（AWS Managed Microsoft AD）或自建 AD
  - 保留 Windows 权限模型（ACL）
- 支持 Windows 原生功能：
  - **卷影副本 (Shadow Copies)**：文件级时间点恢复，用户可自助还原
  - **数据去重 (Data Deduplication)**：降低存储成本
  - **DFS 命名空间 (Distributed File System)**：跨多个文件系统整合命名空间
- 可挂载到 **EC2 (Windows/Linux)**，也可从**本地数据中心**通过 Direct Connect / VPN 访问

### 存储类型

| 存储类型 | 特点 | 适用场景 |
|---------|------|---------|
| **SSD** | 高性能、低延迟 | 数据库、延迟敏感应用 |
| **HDD** | 低成本 | 通用文件共享、大容量归档 |

### 部署类型

| 部署类型 | 可用性 | 说明 |
|---------|--------|------|
| **Single-AZ** | 单可用区 | 成本更低，无自动故障转移 |
| **Multi-AZ** | 跨可用区自动故障转移 | 生产环境推荐，主备架构 + 自动同步 |

### 适用场景

- **Windows 应用迁移上云**：如 .NET 应用、SQL Server 备份文件共享
- **企业用户主目录/部门共享盘**：需要 AD 权限控制
- **需要 SMB 协议**的传统企业应用

---

## FSx for Lustre

### 核心特性

- **Lustre** 是专为**高性能计算 (HPC)** 设计的开源并行文件系统
- 提供**亚毫秒级延迟**、数百 GB/s 吞吐量、数百万 IOPS
- 原生 **POSIX 兼容**，可像本地磁盘一样挂载使用
- **与 [[S3]] 无缝集成**：
  - 可将 S3 存储桶中的对象**延迟加载 (lazy load)** 为 Lustre 文件系统中的文件
  - 计算完成后可将结果**写回 S3**

### 部署类型对比（考试高频考点）

| 部署类型 | 数据持久性 | 适用场景 |
|---------|-----------|---------|
| **Scratch (临时型)** | **不复制数据**，节点故障可能丢失数据 | 短期高性能计算，成本最低，最高突发吞吐 |
| **Persistent (持久型)** | 数据在 AZ 内自动复制，节点故障自动更换 | 长期运行的工作负载，需要高可用性 |

### 适用场景

- **高性能计算 (HPC)**：基因组学、金融建模、地震分析
- **机器学习训练**：大规模并行读取训练数据集
- **视频处理与渲染**：媒体转码、CG 渲染农场
- **大数据分析**：需要与 S3 数据湖直接联动计算的场景

### 场景关键词识别

```
"高性能计算 / HPC" → FSx for Lustre
"机器学习训练，需要高吞吐并行读取" → FSx for Lustre
"直接对 S3 中的数据进行计算，无需先复制" → FSx for Lustre（S3 集成）
"短期计算任务，成本优先" → Scratch 文件系统
"长期运行，需要高可用" → Persistent 文件系统
```

---

## FSx for NetApp ONTAP（简述）

- 提供 NetApp **ONTAP** 文件系统的完全托管版本
- 同时支持 **NFS、SMB、iSCSI** 多协议访问
- 适合**从本地 NetApp 环境迁移**到 AWS，或需要 NetApp 特有功能（如 Snapshot、FlexClone、精简配置）的场景
- 支持**跨协议**统一访问同一份数据（Linux 和 Windows 客户端共享同一文件系统）

## FSx for OpenZFS（简述）

- 提供开源 **OpenZFS** 文件系统的完全托管版本
- 支持 **NFS** 协议，专为 Linux 工作负载设计
- 提供 ZFS 特有能力：**快照、克隆、压缩**，适合需要精细存储管理的场景

---

## 高可用性与持久性

- **FSx for Windows**：Multi-AZ 部署提供自动故障转移；Single-AZ 内部件级冗余
- **FSx for Lustre**：Persistent 部署在单个 AZ 内自动复制数据；Scratch 部署不提供数据复制
- **FSx for ONTAP / OpenZFS**：支持 Multi-AZ 部署实现高可用
- 与 [[AWS EFS]] 的"原生跨多 AZ"不同，FSx 各产品的高可用能力**因部署类型和产品而异**，需按需选择

---

## 安全与访问控制

- 通过 **[[VPC]]** 内部网络访问，流量不经过公网
- 通过**安全组**控制访问（参考 [[Security Group]]）
- **静态数据加密**：与 AWS KMS 集成，支持客户托管密钥 (CMK)
- **传输中加密**：SMB/NFS 流量可加密传输
- **FSx for Windows**：权限模型基于 AD，与本地 Windows 权限体系一致

---

## 数据保护与备份

- 支持**自动化每日备份**（增量备份），可自定义备份窗口和保留期
- 支持通过 **AWS Backup** 集中管理 FSx 备份
- 支持**手动备份**用于关键操作前的快照
- 备份可用于**跨区域恢复**，实现灾难恢复

---

## 场景题解题思路

```
场景分析 → 选择文件存储服务
├── "Linux + NFS + 多实例共享" → EFS
├── "Windows 应用迁移 + AD 集成 + SMB" → FSx for Windows File Server
├── "高性能计算 / 机器学习 / 与 S3 联动计算" → FSx for Lustre
├── "从本地 NetApp 迁移 + 多协议" → FSx for NetApp ONTAP
├── "需要 ZFS 快照/克隆特性的 Linux 工作负载" → FSx for OpenZFS
├── "短期 HPC 任务，成本优先，可接受数据丢失风险" → Lustre Scratch
└── "长期 HPC 任务，需要高可用" → Lustre Persistent
```

**关键总结：**
- **EFS** = Linux 原生 NFS 共享存储的默认选择
- **FSx for Windows** = Windows 生态（SMB + AD）的默认选择
- **FSx for Lustre** = 高性能计算 / 机器学习的默认选择，且能与 S3 无缝联动
- **FSx for ONTAP / OpenZFS** = 需要特定第三方文件系统特性时的选择

---

## 最佳实践

1. **按操作系统和协议选型**：Windows 环境优先 FSx for Windows，而非强行用 EFS
2. **HPC/ML 场景直接对接 S3**：使用 FSx for Lustre 避免额外的数据搬运步骤
3. **生产环境使用 Multi-AZ 部署**：避免单点故障
4. **合理选择 Lustre 部署类型**：临时计算用 Scratch 降低成本，长期任务用 Persistent 保证可靠性
5. **启用自动化备份**：结合 AWS Backup 实现统一的备份策略管理
6. **通过安全组和 KMS 加密**：保障传输与静态数据安全
