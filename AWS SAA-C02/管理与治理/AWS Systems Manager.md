# AWS Systems Manager (SSM) - 统一运维管理中枢

> **AWS Systems Manager（SSM）** 是大规模服务器和资源的**统一运维管理**服务，覆盖安全远程访问、命令执行、补丁管理、配置合规、参数/密钥存储等运维全场景，核心优势是**无需堡垒机、无需开放 SSH/RDP 端口**即可安全管理海量 EC2 实例乃至本地/多云服务器。
>
> 相关文档：[[EC2]] | [[AWS Config]] | [[AWS CloudFormation]] | [[AWS OpsWorks]] | [[AWS Secrets Manager]] | [[IAM]] | [[CloudWatch]] | [[KMS]] | [[VPC]]

---

## 核心概念

### 为什么需要 Systems Manager

- **传统运维方式的痛点**：管理大规模服务器队列需要 SSH 密钥分发、堡垒机维护、开放特定端口，带来安全风险和运维复杂度；批量执行命令、打补丁、检查配置漂移更是缺乏统一工具
- **Systems Manager 的核心价值**：通过安装在实例上的 **SSM Agent** 建立与 AWS 的**出站**连接（无需开放入站端口），所有操作通过 IAM 权限控制并记录审计日志，将命令执行、补丁、配置管理、参数存储统一到一套工具链中
- **托管节点范围**：不局限于 EC2，也支持**本地数据中心服务器**和**其他云的实例**（通过混合激活 Hybrid Activations 注册），实现跨环境的统一运维视图

### SSM 在运维工具链中的定位（考试要点）

| 服务 | 类型 | 场景 | 特点 |
|------|------|------|------|
| **Systems Manager** | 无代理式统一运维管理 | 大规模服务器的安全访问、命令执行、补丁、配置管理 | 无需堡垒机/SSH，IAM 权限控制，跨云跨环境 |
| [[AWS OpsWorks]] | 基于 Chef/Puppet 的配置管理 | 已有 Chef/Puppet Cookbook 资产的团队 | 依赖第三方配置管理工具生态 |
| [[AWS CloudFormation]] | 基础设施即代码 | 资源的声明式创建和管理 | 面向"资源本身"而非"资源运行后的运维" |

> **考试陷阱**：**Systems Manager 面向"已运行实例的运维管理"，CloudFormation 面向"资源的创建和生命周期"**——题目描述"需要在不打开 SSH 端口的情况下远程登录并执行命令" → Systems Manager（Session Manager）；描述"需要声明式地创建整套基础设施" → CloudFormation；两者常配合使用而非互相替代。

---

## Session Manager（会话管理器）

- 无需开放 **SSH（22）/RDP（3389）**入站端口、无需管理 SSH 密钥或堡垒机，即可通过浏览器/CLI 直接获得实例的交互式 Shell
- 访问权限完全由 **IAM 策略**控制，会话活动可记录到 [[CloudWatch]] Logs 或 [[S3]]，满足审计要求
- **考试要点**：题目描述"需要安全地远程访问私有子网中的 EC2 实例，且不希望开放任何入站端口" → **Session Manager** 是标准答案，彻底消除了传统堡垒机方案的攻击面

### Just-in-Time 节点访问（新特性）

- 支持**零常驻权限（Zero Standing Privileges）**模式——运维人员需要先**申请访问**并获得批准，才能建立到目标节点的远程连接，而非长期持有可随时连接的权限
- 进一步收紧了"谁能在什么时候访问哪些节点"的权限窗口，是最小特权原则在运维访问场景的延伸

---

## Run Command（远程命令执行）

- 无需 SSH 登录，直接对**大批量**托管实例**批量、异步**执行预定义或自定义命令（如安装软件、重启服务、执行脚本）
- 可结合 IAM 精细控制哪些用户能对哪些实例执行哪些命令，执行结果和输出可发送到 CloudWatch Logs/S3

---

## Patch Manager（补丁管理）

- 自动化管理操作系统和应用程序的**补丁扫描、批准、安装**，支持定义**补丁基线（Patch Baseline）**（如"仅安装安全补丁"、"发布后 7 天再安装"）
- 结合 **Maintenance Windows（维护时间窗口）**，将补丁安装限定在业务低峰期执行，避免影响生产流量
- 提供补丁合规状态报告，快速识别哪些实例未达到补丁基线要求

---

## State Manager（状态管理）

- 定义并**持续强制**实例应处于的目标配置状态（如"必须安装某个安全代理"、"防病毒软件必须保持运行"），偏离时自动纠正
- 与 [[AWS Config]] 的关系：Config 侧重**检测和记录**配置状态是否合规，State Manager 侧重**主动强制维持**目标状态，两者理念互补

---

## Automation（自动化运维）

- 使用预定义或自定义的 **Automation 文档（Runbook）**，编排跨多个步骤的复杂运维任务（如 AMI 构建、资源修复、灾难恢复演练）
- **与 [[AWS Config]] 的标准组合**：Config 检测到资源配置违反合规规则后，可触发 SSM Automation 文档自动修复，实现"检测 + 修复"的自愈式治理闭环（详见 [[AWS Config]] 笔记）

---

## Parameter Store（参数存储）

| 层级 | 说明 |
|------|------|
| **标准层（Standard）** | 免费，参数值最大 4 KB，适合常规配置项 |
| **高级层（Advanced）** | 收费，支持更大的参数值（最大 8 KB）、参数策略（如自动过期提醒）等高级特性 |

- 支持三种参数类型：**String**（明文字符串）、**StringList**（字符串列表）、**SecureString**（使用 [[KMS]] 加密的敏感值）
- 典型用途：集中存储应用配置、数据库连接字符串、API 端点等**非频繁轮换**的配置和敏感信息

> **考试陷阱**：**Parameter Store 与 [[AWS Secrets Manager]] 都能存储敏感信息，但定位不同**——Parameter Store 更侧重**通用配置管理**（免费或低成本，SecureString 提供基础加密存储），Secrets Manager 专为**凭证类密钥**设计，提供**内置的自动轮换**能力（如自动轮换 RDS 密码）；题目强调"需要自动定期轮换数据库密码"应选 **Secrets Manager**，而非 Parameter Store。

---

## 其他核心能力

| 能力 | 说明 |
|------|------|
| **Inventory（清单）** | 自动收集托管实例的操作系统、已安装软件、网络配置等元数据，便于资产盘点和合规核查 |
| **Distributor** | 将软件包（代理、补丁包等）分发到大规模托管实例，支持版本管理 |
| **OpsCenter** | 集中查看和管理跨账户/跨区域的运维问题（OpsItem），聚合 CloudWatch 告警、Config 合规事件等来源的问题 |
| **Change Manager / Change Calendar** | 为生产变更提供审批工作流和"变更窗口"控制，防止未经授权或不合时宜的变更 |
| **Fleet Manager** | 图形化界面统一查看和管理托管节点队列，包括文件系统浏览、Windows 注册表编辑等 |

---

## 混合与多云管理

- 通过**混合激活（Hybrid Activations）**，本地数据中心服务器或其他云平台的实例安装 SSM Agent 并注册后，即可纳入与 EC2 实例相同的统一运维管理视图
- **2026 年计费调整**：AWS 已取消"高级实例层级"的额外收费，混合/多云节点的注册和大部分核心功能（Session Manager、Run Command、Patch Manager 等）现无需额外的实例注册费用

---

## 安全性

| 机制 | 说明 |
|------|------|
| **IAM 权限控制** | 所有操作（会话、命令执行、参数访问）均由 IAM 策略精细授权 |
| **无需入站端口** | SSM Agent 与 AWS 的连接是出站发起的，无需在安全组开放 SSH/RDP 入站规则 |
| **SecureString 加密** | Parameter Store 的敏感参数使用 [[KMS]] 加密存储 |
| **会话/命令审计** | Session Manager 会话记录、Run Command 执行结果可发送到 CloudWatch Logs/S3，结合 [[CloudTrail]] 提供完整操作审计 |
| **Just-in-Time 访问** | 零常驻权限模式，访问需先申请审批 |

---

## 典型应用场景

| 场景 | 推荐配置 |
|------|---------|
| **安全远程访问私有子网 EC2，不开放 SSH 端口** | Session Manager |
| **批量对大量实例执行运维命令** | Run Command |
| **统一管理操作系统/应用补丁** | Patch Manager + Maintenance Windows |
| **持续强制实例保持目标配置状态** | State Manager |
| **配置检测到违规后自动修复** | Automation + AWS Config 组合 |
| **集中存储非频繁轮换的配置项/敏感值** | Parameter Store（SecureString） |
| **需要自动定期轮换的数据库密码/密钥** | 改用 [[AWS Secrets Manager]]（而非 Parameter Store） |
| **盘点大规模实例的软件/配置清单** | Inventory |
| **管理本地/多云服务器纳入统一运维视图** | 混合激活（Hybrid Activations） |
| **生产变更需要审批流程和变更窗口控制** | Change Manager / Change Calendar |

---

## 考试重点总结

### SAA-C02 高频考点

1. **核心定位**：无代理式统一运维管理，覆盖安全访问、命令执行、补丁、配置管理、参数存储
2. **Session Manager 消除堡垒机需求**：无需开放 SSH/RDP 入站端口，权限完全由 IAM 控制
3. **Run Command 用于批量异步命令执行**：无需逐台登录实例
4. **Patch Manager + Maintenance Windows**：自动化补丁管理并限定在业务低峰期执行
5. **State Manager vs AWS Config**：前者主动强制维持目标状态，后者检测并记录配置合规状态
6. **Automation 是 Config 自动修复的执行引擎**：检测（Config）+ 修复（SSM Automation）构成自愈式治理标准模式
7. **Parameter Store vs [[AWS Secrets Manager]]**：前者通用配置管理（含 SecureString 加密），后者专为凭证设计、支持自动轮换
8. **Parameter Store 两个层级**：标准层免费/4KB，高级层收费/8KB+高级特性
9. **混合激活支持跨云统一管理**：本地/多云服务器可与 EC2 纳入同一运维视图，2026 年起无需额外实例注册费
10. **Just-in-Time 访问实现零常驻权限**：运维访问需先申请审批，而非长期持有连接权限

### 场景题解题思路

```
场景分析 → 判断使用 Systems Manager 的哪个能力
├── "需要安全远程访问 EC2，不希望开放 SSH/RDP 端口" → Session Manager
├── "需要对大批量实例批量执行命令" → Run Command
├── "需要自动化管理操作系统/应用补丁" → Patch Manager + Maintenance Windows
├── "需要持续强制实例保持某个配置状态" → State Manager
├── "配置检测到违规后需要自动修复" → SSM Automation + AWS Config
├── "需要集中存储配置项，含少量敏感值" → Parameter Store（SecureString）
├── "需要自动定期轮换数据库密码等凭证" → 改用 [[AWS Secrets Manager]]（而非 Parameter Store）
├── "需要盘点大规模实例的软件/配置清单" → Inventory
├── "需要管理本地/多云服务器的统一运维" → 混合激活（Hybrid Activations）
└── "生产变更需要审批流程控制" → Change Manager / Change Calendar
```

---

## 最佳实践

1. **默认使用 Session Manager 替代传统堡垒机**：消除 SSH 端口暴露的攻击面，同时获得完整的会话审计记录
2. **高危运维访问启用 Just-in-Time 访问**：贯彻零常驻权限，降低长期持有访问权限的风险
3. **补丁安装结合 Maintenance Windows 避开业务高峰**：减少补丁操作对生产流量的影响
4. **State Manager 与 Config 分工协作**：State Manager 主动维持状态，Config 负责检测合规并触发修复
5. **敏感凭证优先用 [[AWS Secrets Manager]]，通用配置用 Parameter Store**：按数据性质选择合适的存储服务，避免混用
6. **混合云环境统一纳入 SSM 管理视图**：借助混合激活避免本地/多云资源脱离统一运维体系
7. **善用 Inventory 定期盘点资产**：辅助合规审计和软件资产管理，及时发现未授权软件
8. **生产变更走 Change Manager 审批流程**：避免未经审批的高风险操作直接执行
