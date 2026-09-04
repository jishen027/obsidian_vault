# AWS Directory Service - 托管目录服务

> **AWS Directory Service** 是托管的目录服务家族，为需要 **Microsoft Active Directory（AD）**兼容目录的场景提供多种部署形态——从完整托管的原生 AD、到连接本地 AD 的代理、再到轻量级的独立目录，解决企业将依赖 AD 的应用和工作负载迁移/扩展到云端的目录集成需求。
>
> 相关文档：[[IAM]] | [[Cognito]] | [[AWS FSx]] | [[RDS]] | [[VPC]] | [[CloudWatch]]

---

## 核心概念

### 为什么需要 Directory Service

- **企业遗留系统的目录依赖**：大量企业应用（Windows 文件共享、SQL Server、SharePoint、内部业务系统）依赖 **Microsoft Active Directory** 做身份验证和组策略管理，迁移上云时不能简单丢弃这套体系
- **Directory Service 的核心价值**：提供**与 AD 兼容或直接对接 AD** 的托管目录，让这些应用无需重写身份验证逻辑即可在 AWS 云端或混合环境中继续使用 AD 生态（Kerberos 认证、组策略、LDAP、域信任）

### 三种目录类型（考试高频）

| 类型 | 定位 | 说明 |
|------|------|------|
| **AWS Managed Microsoft AD** | 云端原生的完整 AD | 运行在真实的 Windows Server AD 上，支持**组策略、域信任、Schema 扩展、Kerberos、LDAP**等完整 AD 特性，可与本地 AD 建立**双向信任关系** |
| **AD Connector** | 连接本地 AD 的代理 | **不复制目录数据**，只是一个代理/网关，将 AWS 服务的认证请求转发到**本地已有的 AD**，本地 AD 保持唯一权威数据源 |
| **Simple AD** | 轻量级独立目录 | 基于 **Samba 4** 构建、兼容 AD 的独立目录，功能子集（用户/组管理、组策略、Kerberos SSO），**已于 2026-07-30 起停止向新客户开放**，现有客户不受影响 |

> **考试陷阱**：**三者的核心判断依据是"数据权威源在哪里"以及"是否需要完整 AD 特性"**——已有本地 AD、希望云端应用直接认证到本地目录且不复制数据 → **AD Connector**；需要在云端运行一套完整、独立管理的 AD（可选与本地建立信任）→ **AWS Managed Microsoft AD**；只是轻量级、临时性或成本敏感的简单目录需求，且不需要与本地 AD 关联 → **Simple AD**（但需注意其已停止新客户接入）。

---

## AWS Managed Microsoft AD 深入

### 版本

| 版本 | 说明 |
|------|------|
| **Standard Edition** | 面向中小型部署，目录对象数量和存储容量有限 |
| **Enterprise Edition** | 面向大型企业部署，支持更大规模的对象数量和存储 |

### 核心能力

- **域信任（Trust Relationships）**：可与**本地 AD 域**建立单向或双向信任关系，实现云端和本地用户互相访问对方环境中的资源，无需迁移或复制账户
- **目录共享（Directory Sharing）**：一个 Managed Microsoft AD 目录可**跨多个 AWS 账户共享**，供不同账户内的 EC2 实例加入同一域、使用同一套 AD 凭证认证——Standard Edition 最多共享给 25 个账户，Enterprise Edition 最多 500 个账户
- **Hybrid Edition（2025 新增）**：将**本地或多云环境中已有的 AD 域**扩展到 AWS Managed Microsoft AD，自动处理跨环境的复制和维护，进一步简化混合云场景下的目录一致性管理
- **多区域复制**：支持将目录复制到多个 AWS 区域，供全球分布的应用就近认证，降低跨区域认证延迟
- **安全日志集成**：目录的安全日志可转发到 [[CloudWatch]] Logs，用于安全监控、审计和合规报告

### 与其他 AWS 服务的集成

| 集成服务 | 用途 |
|---------|------|
| **Amazon WorkSpaces** | 云端桌面使用 AD 凭证登录 |
| **[[RDS]] / [[AWS FSx]] for Windows File Server** | 数据库/文件服务器直接使用 AD 做身份验证和访问控制 |
| **EC2 域加入（Domain Join）** | Windows EC2 实例加入 AD 域，统一管理策略和身份 |
| **IAM Identity Center** | 可作为身份源之一，供企业员工用 AD 凭证做 AWS 多账户 SSO |

---

## AD Connector 深入

- 本质是一个**目录代理（Proxy）**，不在 AWS 侧存储任何用户/密码数据，所有认证请求都转发回**本地已有的 Active Directory**
- **典型用途**：让 WorkSpaces 连接到本地 AD、启用 EC2 域加入、使用 AD 凭证登录 AWS 管理控制台、与 IAM Identity Center 集成，同时保持"本地 AD 是唯一真相来源"
- **优势**：无需在云端维护第二份目录数据副本，避免数据同步和一致性问题；**局限**：依赖与本地网络的持续连通性（通常通过 [[VPC]] 的 VPN/Direct Connect），本地网络中断会影响云端认证请求

---

## Simple AD 深入（服务状态变化）

- 基于开源 **Samba 4** 实现的 AD 兼容目录，提供用户账户/组成员管理、组策略、EC2 安全连接、基于 Kerberos 的 SSO 等基础功能
- **不支持**域信任、Schema 扩展等 AWS Managed Microsoft AD 才具备的完整 AD 特性
- **考试提示**：Simple AD 已于 **2026-07-30 起停止向新客户开放**，规划新架构时若涉及"轻量级独立目录"需求，应优先评估 AWS Managed Microsoft AD 或确认 Simple AD 的当前可用性

---

## Directory Service Data API（新特性）

- 允许通过 **AWS CLI、API、管理控制台**直接对目录中的用户和组执行**增删改查（CRUD）**操作，无需在域内的 Windows 主机上运行传统的 AD 管理工具（如 ADUC）
- 简化了自动化脚本和 IaC 场景下对目录对象的编程式管理

---

## 与 IAM / Cognito 的定位区分（考试要点）

| 服务 | 面向对象 | 核心场景 |
|------|---------|---------|
| **AWS Directory Service** | 依赖 **Microsoft AD 生态**的传统企业应用/员工 | Windows 域环境的云端延伸或原生托管 |
| **[[IAM]]** | AWS 账户内部的身份（员工、角色、服务） | 管控对 AWS 控制台和 API 本身的访问权限 |
| **[[Cognito]]** | 面向**外部 App 的终端消费者** | 移动/Web 应用的海量用户注册登录 |

> **考试陷阱**：题目描述**"企业已有 Windows AD 环境，需要将依赖 AD 的应用（如 SharePoint、SQL Server 认证）迁移或扩展到云端"** → 答案是 **AWS Directory Service**（具体选哪种类型取决于是否需要复制数据/完整 AD 特性）；这与 [[IAM]]（AWS 账户内部身份）、[[Cognito]]（应用终端用户）是完全不同维度的需求，三者不可混淆。

---

## 安全性

| 机制 | 说明 |
|------|------|
| **VPC 部署** | 目录部署在 [[VPC]] 私有子网中，通过安全组控制访问 |
| **多可用区高可用** | AWS Managed Microsoft AD 默认跨两个可用区部署域控制器，提供高可用性 |
| **传输加密** | 域内通信遵循标准 AD 协议的加密机制（Kerberos） |
| **安全日志转发** | 可将安全事件日志转发到 [[CloudWatch]] Logs 做集中监控 |

---

## 典型应用场景

| 场景 | 推荐配置 |
|------|---------|
| **已有本地 AD，希望云端应用直接认证到本地目录** | AD Connector |
| **需要在云端运行独立、完整功能的 AD 域** | AWS Managed Microsoft AD |
| **需要本地 AD 与云端 AD 建立双向信任互访** | AWS Managed Microsoft AD + 域信任 |
| **多账户环境下多个 EC2 需加入同一 AD 域** | AWS Managed Microsoft AD + 目录共享 |
| **混合云/多云环境需要统一 AD 域并自动同步** | AWS Managed Microsoft AD Hybrid Edition |
| **RDS/FSx for Windows 需要 AD 身份验证** | AWS Managed Microsoft AD 或 AD Connector 对接 |
| **轻量级、独立、成本敏感的简单目录需求** | Simple AD（需确认新客户接入状态） |
| **自动化脚本管理目录用户/组** | Directory Service Data API |

---

## 考试重点总结

### SAA-C02 高频考点

1. **核心定位**：托管的 AD 兼容目录服务家族，服务于依赖 Windows AD 生态的企业应用
2. **三种类型判断依据**：数据权威源位置 + 是否需要完整 AD 特性——AD Connector（代理，本地为准）、Managed Microsoft AD（云端原生完整 AD）、Simple AD（轻量独立，已停止新客户接入）
3. **AD Connector 不复制数据**：仅做认证请求转发，依赖与本地网络的持续连通性
4. **Managed Microsoft AD 支持域信任**：可与本地 AD 建立单向/双向信任，无需迁移账户即可互访资源
5. **目录共享支持跨账户**：Standard 最多 25 个账户，Enterprise 最多 500 个账户
6. **Hybrid Edition 简化混合云目录一致性**：自动处理本地/多云 AD 域与 AWS 侧的复制维护
7. **Simple AD 功能是子集**：不支持域信任、Schema 扩展，已于 2026-07-30 停止新客户接入
8. **与 RDS/FSx/WorkSpaces/EC2 深度集成**：这些服务的 Windows 身份验证场景是高频考点
9. **Directory Service vs IAM vs Cognito**：分别面向 AD 生态应用/AWS 账户内部身份/应用终端用户，三个完全不同的维度
10. **安全日志可转发到 CloudWatch Logs**：用于审计和合规监控

### 场景题解题思路

```
场景分析 → 判断使用哪种 Directory Service
├── "已有本地 AD，希望云端应用无需复制数据即可认证到本地目录" → AD Connector
├── "需要在云端运行独立、完整功能的 AD（组策略/域信任/Schema 扩展）" → AWS Managed Microsoft AD
├── "需要本地 AD 与云端环境建立双向信任互访资源" → Managed Microsoft AD + 域信任
├── "多个 AWS 账户的 EC2 需要加入同一 AD 域" → Managed Microsoft AD + 目录共享
├── "混合云/多云环境需要自动同步的统一 AD" → Managed Microsoft AD Hybrid Edition
├── "轻量级、独立、成本敏感，且不涉及本地 AD" → Simple AD（确认接入状态）
├── "需要通过脚本/API 自动化管理目录用户组" → Directory Service Data API
├── "企业员工需要访问 AWS 控制台/API" → 改用 IAM/IAM Identity Center（而非 Directory Service 本身）
└── "移动/Web 应用的终端用户注册登录" → 改用 Cognito（而非 Directory Service）
```

---

## 最佳实践

1. **按数据权威源需求选择目录类型**：本地 AD 为权威源用 AD Connector，需要云端独立管理用 Managed Microsoft AD
2. **AD Connector 场景确保本地网络连通性可靠**：认证请求依赖 VPN/Direct Connect 持续可用，应评估网络冗余方案
3. **多账户场景善用目录共享**：避免为每个账户重复搭建独立目录，统一域管理降低运维复杂度
4. **混合云场景优先评估 Hybrid Edition**：相比手动维护跨环境复制，自动化方案更可靠
5. **新架构避免依赖 Simple AD**：已停止新客户接入，轻量级需求可考虑 Managed Microsoft AD 的 Standard Edition
6. **启用安全日志转发到 CloudWatch**：便于集中审计和满足合规要求
7. **域控制器部署遵循高可用最佳实践**：确保跨多可用区部署，避免单点故障影响认证服务
8. **明确区分三类身份服务的适用边界**：Directory Service（AD 生态应用）、IAM（AWS 账户内部）、Cognito（应用终端用户），避免选型混淆
