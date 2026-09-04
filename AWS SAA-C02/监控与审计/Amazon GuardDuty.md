# Amazon GuardDuty - 智能威胁检测

> **Amazon GuardDuty** 是全托管的**智能威胁检测**服务，持续分析 [[CloudTrail]] 事件、[[VPC]] 流日志、DNS 日志等多种数据源，结合机器学习、异常检测和威胁情报，自动识别账户内的**凭证被盗、恶意侦察、恶意软件、异常 API 调用**等安全威胁，无需部署或管理任何检测基础设施。
>
> 相关文档：[[CloudTrail]] | [[VPC]] | [[AWS Config]] | [[AWS WAF]] | [[AWS Shield]] | [[IAM]] | [[AWS Organizations]] | [[EKS]] | [[Amazon Inspector]] | [[Amazon Macie]]

---

## 核心概念

### 为什么需要 GuardDuty

- **人工审查日志的局限**：CloudTrail/VPC Flow Logs 等原始日志量巨大，人工排查异常几乎不可能覆盖全部攻击面，且许多攻击模式（凭证被盗后的横向移动、恶意软件通信）需要跨多个数据源关联分析才能识别
- **GuardDuty 的核心价值**：**无需部署代理、无需管理基础设施**，直接分析账户已有的日志和网络流数据，结合 AWS 安全团队维护的**威胁情报**（已知恶意 IP、域名）和**机器学习异常检测**模型，自动生成带严重程度评级的安全发现（Findings）
- **开箱即用**：启用后立即开始分析历史可用数据，无需预先配置规则或训练模型，是"零基础设施"的威胁检测方案

### GuardDuty 在安全监控体系中的定位（考试要点）

| 服务 | 核心问题 | 数据类型 | 典型场景 |
|------|---------|---------|---------|
| **GuardDuty** | 是否存在安全威胁/攻击行为？ | 跨数据源关联分析后的威胁发现（Findings） | 凭证被盗、恶意软件、异常 API 调用、攻击者侦察 |
| [[CloudTrail]] | 谁在什么时候做了什么？ | 原始 API 调用事件日志 | 审计、取证，是 GuardDuty 的**输入数据源之一** |
| [[AWS Config]] | 资源配置长什么样？ | 配置项快照 | 配置合规检测，与威胁检测是不同维度 |
| [[AWS WAF]] / [[AWS Shield]] | 如何**拦截**已知攻击模式/流量？ | 实时流量过滤 | 主动防御，GuardDuty 是**检测**而非拦截 |
| **[[Amazon Inspector]]** | 资源本身存在哪些**已知漏洞**？ | 软件 CVE、网络可达性、代码缺陷 | 评估已知弱点，与 GuardDuty 的"实时攻击检测"是漏洞管理生命周期中互补的前后阶段，完整对比见 [[Amazon Inspector]] 独立笔记 |
| **[[Amazon Macie]]** | 我的**数据内容**有多敏感、暴露风险多大？ | S3 对象内容（PII、密钥等） | 数据安全维度，与 GuardDuty 的"行为威胁检测"互补，完整能力见 [[Amazon Macie]] 独立笔记 |

> **考试陷阱**：**GuardDuty 是检测和告警，不是主动拦截**——题目描述"需要识别账户内是否存在被入侵的迹象、异常 API 调用" → **GuardDuty**；描述"需要主动拦截恶意流量/请求" → WAF/Shield/Security Group 等执行拦截的服务；描述"需要发现资源上是否存在已知 CVE/未打补丁的软件" → [[Amazon Inspector]]（评估弱点）而非 GuardDuty（检测攻击行为）；GuardDuty 发现威胁后通常需要**结合其他机制**（Lambda 自动化响应、安全团队介入）才能实际处置。

---

## 数据源与分析维度

| 数据源 | 用于检测 |
|--------|---------|
| **[[CloudTrail]] 管理事件** | 异常的账户级 API 调用行为（如异地登录、权限提升尝试） |
| **[[CloudTrail]] S3 数据事件** | S3 对象级别的异常访问模式，识别潜在的数据泄露或凭证滥用（S3 Protection） |
| **[[VPC]] 流日志** | 网络层面的异常流量，如与已知恶意 IP 的通信、端口扫描 |
| **DNS 日志** | 恶意域名查询、命令与控制（C2）通信特征 |
| **EKS 审计日志 + 运行时监控** | Kubernetes 集群的异常操作和容器运行时的可疑进程/恶意软件执行（EKS Protection） |
| **RDS 登录活动** | 数据库异常登录尝试，识别潜在的凭证滥用（RDS Protection） |
| **Lambda 网络活动** | 函数执行期间的异常网络连接（Lambda Protection） |

- GuardDuty **不需要用户手动接入或配置这些日志源**——只要相应的 AWS 服务在使用，启用 GuardDuty 后即可自动分析，无需额外部署日志收集管道

---

## 核心防护模块（考试高频）

| 模块 | 说明 |
|------|------|
| **基础威胁检测** | 分析 CloudTrail/VPC Flow Logs/DNS 日志，识别异常 API 调用、可疑网络通信等基础威胁模式 |
| **S3 Protection** | 检测 S3 对象级 API 调用中的异常模式，识别潜在的凭证泄露或恶意访问 |
| **EKS Protection** | 分析 EKS 审计日志识别 Kubernetes 层面的可疑操作；结合**运行时监控（Runtime Monitoring）**进一步获得容器级可见性，检测恶意进程和运行时行为 |
| **RDS Protection** | 监控数据库登录活动，识别异常或暴力破解式的登录尝试 |
| **Lambda Protection** | 检测 Lambda 函数网络活动中的异常连接模式 |
| **Malware Protection** | 扫描 EBS 卷和 S3 对象，识别 EC2 实例、容器工作负载、S3 存储桶中的恶意软件；**Malware Protection for AWS Backup** 进一步在恢复前扫描备份数据，确保恢复内容不包含恶意软件 |

> **考试要点**：题目提到具体资源类型的威胁检测（"S3 异常访问"、"EKS 容器恶意进程"、"RDS 异常登录"、"Lambda 异常网络连接"）时，应对应到相应的**防护模块**，而非笼统地回答"GuardDuty"——这些模块可**按需独立启用**，并非全部默认开启。

---

## Extended Threat Detection（扩展威胁检测，考试提示）

- 自动关联**跨多个数据源、跨时间线**的信号，识别单一数据源难以发现的**多阶段攻击（Multi-Stage Attack）**序列（如"异常登录 → 权限侦察 → 横向移动 → 数据外泄"的完整攻击链）
- **对所有 GuardDuty 客户自动启用，无需额外付费**
- **覆盖范围持续扩展**：最初面向账户级攻击序列，2025 年扩展支持 **EKS 集群**的多阶段攻击关联分析，随后进一步扩展支持 **EC2** 和运行在 EC2/Fargate 上的 **ECS** 集群
- **考试要点**：题目强调"需要识别横跨多个阶段、多个信号的复杂攻击链，而非孤立的单点异常" → **Extended Threat Detection** 是精准识别点，这是 GuardDuty 相比"单一规则告警"工具的核心差异化能力

---

## 发现（Findings）与严重程度

- 每个检测结果称为一个 **Finding**，包含威胁类型、涉及资源、严重程度评级（**低/中/高**）等信息
- 支持配置**抑制规则（Suppression Rules）**，对已知的误报或低风险场景自动归档 Finding，减少告警噪音
- 支持自定义**受信任 IP 列表（Trusted IP List）**和**威胁 IP 列表（Threat IP List）**，分别用于排除已知安全的内部 IP、补充组织自有的威胁情报

---

## 多账户管理

- 支持指定**委派管理员账户**，结合 [[AWS Organizations]] 实现组织级的集中威胁检测管理——委派管理员账户可以查看和管理组织内所有成员账户的 GuardDuty Findings，无需登录每个账户分别检查
- 新加入组织的账户可配置为**自动启用** GuardDuty，确保安全覆盖不因遗漏配置而出现盲区

---

## 响应与集成（考试高频）

| 集成方式 | 说明 |
|---------|------|
| **[[Amazon EventBridge]]** | GuardDuty Findings 可作为事件源，触发自动化响应（如通过 Lambda 自动隔离受威胁的 EC2 实例、撤销可疑凭证） |
| **AWS Security Hub** | Findings 可汇总到 Security Hub，与其他安全服务（Shield Network Security Director 等）的发现统一到中心化的安全态势看板 |
| **Amazon Detective** | 对 GuardDuty Finding 做进一步的**根因调查**，可视化攻击时间线和资源关系图，辅助安全团队深入分析 |

> **考试要点**：题目描述**"检测到威胁后需要自动化响应（如自动隔离实例）"** → GuardDuty Finding + **EventBridge** + Lambda 的标准自动化响应模式；描述**"需要对发现的威胁做深入根因调查"** → 结合 **Amazon Detective**，GuardDuty 本身负责检测和告警，不提供深度调查分析能力。

---

## 安全性

| 机制 | 说明 |
|------|------|
| **IAM 权限控制** | 控制哪些身份可以查看/管理 GuardDuty Findings 和配置 |
| **无需额外部署代理**（基础检测） | 分析已有的 AWS 服务日志，不侵入现有工作负载；Runtime Monitoring 等高级能力需部署轻量级代理获得进程级可见性 |
| **委派管理员职责分离** | 组织级威胁检测管理可下放到专用安全账户，遵循最小权限原则 |

---

## 典型应用场景

| 场景 | 推荐配置 |
|------|---------|
| **检测账户内的异常 API 调用/凭证被盗迹象** | 启用 GuardDuty 基础威胁检测 |
| **检测 S3 存储桶的异常访问模式** | S3 Protection |
| **检测 EKS 集群的容器级恶意进程/异常运行时行为** | EKS Protection + Runtime Monitoring |
| **检测 RDS 数据库的异常/暴力破解登录** | RDS Protection |
| **检测 Lambda 函数的异常网络连接** | Lambda Protection |
| **扫描 EC2/S3 中的恶意软件** | Malware Protection |
| **确保恢复的备份数据不含恶意软件** | Malware Protection for AWS Backup |
| **识别跨多阶段、多信号的复杂攻击链** | Extended Threat Detection（自动启用） |
| **组织级集中管理多账户的威胁检测** | 委派管理员 + AWS Organizations |
| **检测到威胁后自动触发响应动作** | GuardDuty + EventBridge + Lambda |
| **需要对威胁做深入根因调查和可视化分析** | 结合 Amazon Detective |
| **需要主动拦截攻击流量/请求，而非仅检测** | 改用/叠加 AWS WAF、AWS Shield（而非仅依赖 GuardDuty） |

---

## 考试重点总结

### SAA-C02 高频考点

1. **核心定位**：全托管智能威胁检测服务，分析多数据源识别安全威胁，检测而非主动拦截
2. **判断依据**：需要"发现是否存在威胁/入侵迹象"选 GuardDuty；需要"主动拦截攻击"选 WAF/Shield
3. **无需部署基础设施**：直接分析已有的 CloudTrail/VPC Flow Logs/DNS 日志，开箱即用
4. **多个防护模块按需启用**：S3/EKS/RDS/Lambda Protection 分别针对不同资源类型的威胁，并非全部默认开启
5. **Malware Protection 覆盖 EC2/S3/备份**：扫描 EBS 卷和 S3 对象，还可在恢复前扫描备份数据
6. **Extended Threat Detection 自动启用且免费**：识别跨数据源、跨时间线的多阶段攻击链，是核心差异化能力
7. **Findings 支持抑制规则降噪**：结合受信任/威胁 IP 列表进一步精细化检测
8. **委派管理员实现组织级集中管理**：结合 Organizations，新账户可配置自动启用
9. **响应依赖 EventBridge 集成**：Findings 触发自动化响应（如隔离实例）的标准模式
10. **深度调查依赖 Amazon Detective**：GuardDuty 负责检测告警，Detective 负责根因分析和攻击链可视化

### 场景题解题思路

```
场景分析 → 判断使用 GuardDuty 的哪个能力
├── "需要检测账户内的异常 API 调用/凭证被盗迹象" → GuardDuty 基础威胁检测
├── "需要检测 S3/EKS/RDS/Lambda 特定资源的异常行为" → 对应的 Protection 模块
├── "需要扫描 EC2/S3 中的恶意软件" → Malware Protection
├── "需要确保恢复的备份数据安全" → Malware Protection for AWS Backup
├── "需要识别跨多阶段、多信号的复杂攻击链" → Extended Threat Detection
├── "需要组织级集中管理多账户威胁检测" → 委派管理员 + Organizations
├── "检测到威胁后需要自动化响应（如隔离实例）" → GuardDuty + EventBridge + Lambda
├── "需要对威胁做深入根因调查" → 结合 Amazon Detective
└── "需要主动拦截攻击流量，而非仅检测发现" → 改用/叠加 AWS WAF、AWS Shield
```

---

## 最佳实践

1. **默认启用 GuardDuty 作为账户的基础威胁检测能力**：无需额外基础设施投入即可获得可观的安全覆盖
2. **按实际使用的资源类型启用对应防护模块**：使用 EKS/RDS/Lambda 的账户应针对性开启相应 Protection，而非遗漏
3. **组织级环境使用委派管理员 + 自动启用**：确保新创建账户不会因遗漏配置而形成检测盲区
4. **善用抑制规则减少告警疲劳**：对已确认的误报或低风险场景配置抑制，让团队聚焦真正的高风险 Finding
5. **结合 EventBridge 构建自动化响应管道**：对高严重程度 Finding 实现自动隔离/告警，而非仅依赖人工响应
6. **重大安全事件调查时结合 Amazon Detective**：利用其攻击时间线和关系图能力加速根因分析
7. **补充组织自有威胁情报到 Threat IP List**：结合 AWS 官方威胁情报和内部已知风险 IP，提升检测覆盖面
8. **GuardDuty 与 WAF/Shield 组合形成检测+防御闭环**：GuardDuty 负责发现，WAF/Shield 负责拦截，缺一不可
