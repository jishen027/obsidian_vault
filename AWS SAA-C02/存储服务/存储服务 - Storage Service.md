# AWS 存储服务概览

> AWS 提供全面的存储服务，覆盖从对象存储、块存储到文件存储的所有场景。理解不同存储服务的特性和适用场景是 AWS 架构师认证的核心知识点。

---

## 存储架构的两大支柱

| 存储类型 | 服务 | 架构特点 | 适用场景 |
|---------|------|---------|---------|
| **对象存储** | [[S3 - Object Storage]] | 扁平命名空间，通过 Key 访问对象 | 静态资源、备份归档、大数据分析 |
| **块存储** | [[块储存 - EBS]] | 模拟物理硬盘，挂载为块设备 | 操作系统盘、数据库、企业级应用 |

### 核心区别

- **S3**：通过 REST API 访问，对象级别操作，适合非结构化数据
- **EBS**：通过 iSCSI 协议挂载，块级别操作，适合文件系统级访问

---

## 核心存储服务

### 1. Amazon S3 (Simple Storage Service)

- **类型**：对象存储
- **特点**：无限扩展、99.999999999% (11个9) 持久性
- **存储类别**：Standard、IA、Intelligent-Tiering、Glacier、Glacier Deep Archive
- **相关文档**：[[S3 - Object Storage]]、[[S3 Glacier Archive]]

### 2. Amazon EBS (Elastic Block Store)

- **类型**：块存储
- **特点**：持久化、单可用区冗余、可挂载到 EC2 实例
- **卷类型**：SSD (io2, gp3, gp2)、HDD (st1, sc1)
- **相关文档**：[[块储存 - EBS]]

### 3. S3 Glacier

- **类型**：归档存储
- **特点**：最低成本的存储选项，适合长期归档
- **检索时间**：从几分钟到几小时不等
- **相关文档**：[[S3 Glacier Archive]]

---

## 辅助存储服务

### 1. Amazon EFS (Elastic File System)

- **类型**：文件系统存储 (NFS)
- **特点**：无服务器、多实例共享访问、弹性扩展
- **适用场景**：CMS、容器存储、开发测试环境
- **相关文档**：[[文件储存 - AWS EFS]]

### 2. Amazon FSx

- **类型**：托管文件系统服务
- **提供两种文件系统选项**：
  - **FSx for Windows File Server**：Windows 原生文件系统，SMB 协议
  - **FSx for Lustre**：高性能计算文件系统，Linux 原生
- **适用场景**：Windows 应用迁移、HPC、机器学习

### 3. AWS Storage Gateway

- **类型**：混合存储服务
- **特点**：将本地环境与 AWS 云存储连接
- **三种网关模式**：
  - **文件网关**：S3 兼容的文件接口
  - **卷网关**：iSCSI 挂载的块存储
  - **磁带网关**：虚拟磁带库 (VTL)
- **适用场景**：混合云架构、本地到云的备份

---

## 数据迁移工具

### 物理设备迁移

| 工具 | 描述 | 适用场景 |
|------|------|---------|
| [[AWS Snowball]] | 物理设备，用于大规模数据迁移 |  TB 到 PB 级别的数据迁移，网络不可用或成本过高 |
| **AWS Snowcone** | 更小的边缘计算设备 | 边缘计算 + 小规模数据迁移 |
| **AWS Snowmobile** | 40 英尺拖车，单台可达 100 PB | 超大规模数据迁移（Exabyte 级别） |

### 自动化数据迁移

| 工具 | 描述 | 目标存储 |
|------|------|---------|
| **AWS DataSync** | 自动化将本地数据迁移到 AWS | S3、EFS、FSx |
| **AWS Migration Hub** | 集中跟踪迁移项目进度 | 多服务 |
| **AWS DMS (Database Migration Service)** | 数据库到数据库的迁移 | RDS、DynamoDB 等 |

---

## 存储服务选择指南

### 如何选择存储类型？

```
需要文件级访问？
├── 是 → 多实例共享？
│         ├── 是 → Linux (NFS) → [[文件储存 - AWS EFS]]
│         └── 否 → Windows (SMB) → [[Amazon FSx]] for Windows
└── 否 → 需要块设备挂载？
          ├── 是 → [[块储存 - EBS]]
          └── 否 → 对象/API 访问 → [[S3 - Object Storage]]
```

### 存储成本对比（从高到低）

| 存储类型 | 成本等级 | 适用频率 |
|---------|---------|---------|
| EBS (io2/io1) | 💰💰💰💰💰 | 高频随机 I/O |
| EBS (gp3) | 💰💰💰 | 通用场景 |
| EFS (Standard) | 💰💰💰 | 频繁共享访问 |
| S3 Standard | 💰💰 | 频繁访问 |
| S3 IA / EFS IA | 💰 | 不频繁访问 |
| S3 Glacier | 💰 | 归档 |
| S3 Glacier Deep Archive | 最低 | 长期合规归档 |

---

## 最佳实践

1. **根据访问模式选择存储类型**：热数据用 S3 Standard，冷数据用 Glacier
2. **启用自动数据分层**：利用 Intelligent-Tiering 或 EFS IA 自动优化成本
3. **跨 AZ 部署关键数据**：EBS 快照、S3 版本控制、EFS 多 AZ
4. **启用加密**：所有存储服务都支持 KMS 加密
5. **使用生命周期策略**：自动将旧数据迁移到低成本存储层
6. **混合云场景**：使用 Storage Gateway 或 DataSync 连接本地和云端