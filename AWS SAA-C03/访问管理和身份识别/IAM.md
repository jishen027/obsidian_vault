# AWS IAM - 身份与访问管理

> **AWS Identity and Access Management (IAM)** 是 AWS 账户内身份与访问控制的核心服务，管理"谁（身份）能对哪些资源执行哪些操作"，是几乎所有 AWS 服务权限模型的基础。IAM 本身是**全局性**服务，不局限于特定区域。
>
> 相关文档：[[Cognito]] | [[AWS Directory Service]] | [[AWS Organizations]] | [[KMS]] | [[CloudTrail]] | [[AWS Config]] | [[S3 Security]] | [[VPC]] | [[VPC Endpoints]]

**本笔记结构**：① 身份 → ② 策略（写什么）→ ③ 评估逻辑（怎么判）→ ④ 限制型机制（天花板）→ ⑤ 条件键（怎么收窄）→ ⑥ 角色与跨账户 → ⑦ 联邦 → ⑧ 安全强化 → ⑨ 考试速查

---

## 1. 核心身份（Identities）

| 身份类型 | 说明 |
|---------|------|
| **Root 用户** | 账户创建时默认产生，拥有**最高权限**，不受 IAM 策略限制（但**成员账户的 root 仍受 SCP 约束**，见 [[AWS Organizations]]）。最佳实践：启用 **MFA**、删除所有**访问密钥**、仅在必要操作（如修改账单方案、关闭账户）时使用 |
| **IAM 用户（Users）** | 持久身份，可分配**控制台密码**或**程序化访问密钥（Access Key）**，遵循"最小特权原则"按需授权 |
| **IAM 组（Groups）** | 用户的集合，用于批量挂载策略；**组不是主体（Principal）**——不能被 Assume、不能写进资源策略的 Principal、**不能挂权限边界** |
| **IAM 角色（Roles）** | **临时身份**，没有长期凭证，通过 **AWS STS** 获取 `Access Key ID` + `Secret Access Key` + `Session Token` 临时令牌 |

> **考试要点**：**角色是 AWS 服务间授权和跨账户访问的标准方式**——EC2、Lambda 等应附加 IAM 角色获取权限，而非在代码中硬编码长期访问密钥；"如何让 EC2 安全访问 S3"的标准答案。

---

## 2. 策略（Policies）

### 2.1 策略结构

| 元素 | 作用 |
|------|------|
| `Effect` | `Allow` / `Deny` |
| `Action` / `NotAction` | 操作；`NotAction` 表示"**除这些之外**的所有操作" |
| `Resource` | 资源 ARN |
| `Principal` | **仅资源策略/信任策略中出现**，指定"谁" |
| `Condition` | 可选，基于请求上下文进一步收窄（见第 5 节） |

- **策略是全局性的**：一个用户/角色的权限不因所在区域而改变

### 2.2 策略类型：授予型 vs 限制型（理解评估逻辑的关键）

| 类别 | 类型 | 附加对象 | 作用 |
|------|------|---------|------|
| **授予型**（能给出 Allow） | **身份策略（Identity-Based）** | 用户/组/角色 | "这个身份能做什么" |
| **授予型** | **资源策略（Resource-Based）** | 资源（S3 Bucket Policy、KMS Key Policy、角色信任策略等） | "谁能访问这个资源"，**唯一能直接指定跨账户主体**的策略类型 |
| **限制型**（只收窄、不授予） | **权限边界（Permission Boundary）** | 单个用户/角色 | 该身份的权限天花板 |
| **限制型** | **SCP / RCP** | Organizations 的 Root/OU/账户 | 组织级天花板（SCP 管身份，RCP 管资源） |
| **限制型** | **会话策略（Session Policy）** | `AssumeRole` / `GetFederationToken` 时传入 | 本次会话的天花板 |

身份策略按**管理方式**再分三种：

| 类型 | 说明 |
|------|------|
| **AWS 托管策略** | AWS 预定义并维护，随 AWS 更新自动演进 |
| **客户托管策略** | 自建，可被多个身份复用，独立于身份存在 |
| **内联策略** | 嵌入单个身份，与其**生命周期绑定**（身份删除即删除），不可复用 |

---

## 3. 策略评估逻辑（考试陷阱高发区）

### 3.1 三条铁律

1. **默认隐式拒绝**：没有任何 Allow 就不允许
2. **显式 Deny 永远优先**：任一适用策略里有 Deny，最终就是拒绝
3. **限制型策略是"与"关系**：最终有效权限 = 授予型 Allow ∩ 权限边界 ∩ SCP ∩ 会话策略

### 3.2 完整评估链（手动评估策略时按此顺序走）

```
请求 → ① 任一适用策略有显式 Deny？ → 是 → 拒绝（终止）
     → ② SCP / RCP（若在 Organizations 中）是否 Allow？ → 否 → 拒绝
     → ③ 资源策略是否 Allow？（同账户时：资源策略直接授权给该用户/角色 ARN 即可放行）
     → ④ 身份策略是否 Allow？ → 否 → 拒绝
     → ⑤ 权限边界（若设置）是否 Allow？ → 否 → 拒绝
     → ⑥ 会话策略（若传入）是否 Allow？ → 否 → 拒绝
     → 允许
```

### 3.3 同账户 vs 跨账户

| 场景 | 需要的 Allow |
|------|-------------|
| **同账户** | 身份策略 **或** 资源策略任一 Allow 即可（无显式 Deny，且不被限制型策略拦截） |
| **跨账户** | 调用方账户的身份策略 **和** 目标账户的资源策略**都必须** Allow |

> **考试陷阱**：
> - 同账户内"用户没有任何 IAM 策略，但 Bucket Policy 的 Principal 写了该用户 ARN" → **可以访问**；换成跨账户 → **不能**，还需调用方身份策略授权
> - "IAM 策略已授权但跨账户访问依然失败" → 通常是**目标账户资源策略**一侧未授权

---

## 4. 限制型机制：权限边界 vs SCP vs 会话策略

| 机制 | 作用范围 | 关键规则 |
|------|---------|---------|
| **权限边界** | 单个 IAM **用户/角色** | **不能挂到组**；即使附加了 `AdministratorAccess`，有效权限也不超出边界 |
| **[[AWS Organizations]] SCP** | 整个组织/OU/账户 | 需启用 Organizations **全部功能**（仅合并账单不行）；**不影响管理账户**；**影响成员账户 root**；**不影响 service-linked role** |
| **会话策略** | 单次 AssumeRole/联合会话 | 只能收窄角色权限策略，不能扩大 |

> **考试要点**：题目强调"限制**单个用户/角色**，即使被误授予管理员也不能超限" → **权限边界**；强调"限制**整个账户/组织**所有身份" → **SCP**。三者可叠加，取交集。
>
> **考试陷阱：权限边界不能挂组**——"把开发者放进组，再给组设权限边界"是**无效方案**；正确做法是给**每个用户/角色单独**设置边界。退而求其次用普通 IAM 策略不如边界安全（用户可能修改/移除策略提权）。

---

## 5. 条件键与策略变量

### 5.1 常考条件键速查

| 条件键 | 比较的是什么 | 典型用途 |
|--------|-------------|---------|
| `aws:RequestedRegion` | API 调用的**目标区域** | 区域护栏 |
| `aws:SourceIp` | 调用方**公网**源 IP | 只允许公司 IP 段 |
| `aws:ViaAWSService` | 请求是否由 AWS 服务代为发起 | 配合 SourceIp 避免误拒 |
| `aws:SourceVpce` / `aws:SourceVpc` / `aws:VpcSourceIp` | 经 VPC Endpoint 的请求来源 | 桶只允许从某个 VPC/终端节点访问 |
| `aws:MultiFactorAuthPresent` | 是否经过 MFA | 敏感操作要求 MFA |

### 5.2 `aws:RequestedRegion`：限定目标区域，不是发起位置（易混淆点）

- `"StringEquals": {"aws:RequestedRegion": "eu-west-1"}` + `Allow` EC2 操作 = "**该操作只能在 eu-west-1 执行**"，调用者在哪里发起都无所谓
- 限制"调用来源"要用 `aws:SourceIp` 等，两者维度完全不同
- **解题步骤**：先确认限制的是**目标区域**，再单独判断调用来源是否受限（没有 SourceIp 类条件就不受限）

**区域限制的标准写法：`Deny` + `StringNotEquals` + `NotAction`（考试原题常见形态，常用作 SCP）**

```json
{
  "Effect": "Deny",
  "NotAction": ["iam:*", "sts:*", "cloudfront:*", "route53:*", "support:*", "organizations:*"],
  "Resource": "*",
  "Condition": {"StringNotEquals": {"aws:RequestedRegion": ["eu-west-1", "eu-west-2"]}}
}
```

- 含义：**除列出的全局服务外**，目标区域不在 eu-west-1/eu-west-2 的操作一律拒绝
- **必须用 `NotAction` 豁免全局服务**：IAM、STS（全局端点）、CloudFront、Route 53、Support、Organizations 的请求区域固定为 `us-east-1`，不豁免会被误拒
- `Deny` + `NotAction` **不等于**允许这些操作——放行仍需其他策略的 Allow

### 5.3 `aws:SourceIp` 的两个陷阱

1. **误拒 AWS 服务代你发起的调用**：`Deny` + `NotIpAddress aws:SourceIp` 时，CloudFormation、Athena 等以你的身份调用其他服务，源 IP 是 AWS 地址 → 加 `"Bool": {"aws:ViaAWSService": "false"}`，让 Deny 只针对直接调用
2. **经 VPC Endpoint 的请求没有公网源 IP**：SourceIp 匹配不到 → 改用 `aws:SourceVpce` / `aws:SourceVpc` / `aws:VpcSourceIp`

> **考试关键词识别**："只允许公司 IP 段访问，但 CloudFormation 部署失败" → 缺 `aws:ViaAWSService`；"S3 Bucket 只允许从某个 VPC 访问" → 桶策略用 `aws:SourceVpce` / `aws:SourceVpc`。

### 5.4 策略变量

- 在策略中引用运行时值（如 `${aws:username}`），实现"用户只能访问以自己用户名命名的 S3 前缀"这类可复用模板，无需为每个用户单独写策略

---

## 6. 角色与跨账户访问（STS / AssumeRole）

- **AWS STS** 负责发放有时效性的**临时凭证**（默认数小时过期，比长期密钥更安全）
- 角色由两部分组成，职责分离：
  - **信任策略（Trust Policy）**：**谁**可以扮演这个角色（本质是一种资源策略）
  - **权限策略（Permission Policy）**：扮演后**能做什么**

### 跨账户访问的两种方式（考试高频对比）

| 方式 | 做法 | 调用方的原有权限 | 适用场景 |
|------|------|----------------|---------|
| **AssumeRole** | 账户 B 建角色，信任策略信任账户 A；A 的用户 `sts:AssumeRole` 获取临时凭证 | **放弃**——会话中只有该角色的权限 | 通用标准模式；目标服务不支持资源策略时 |
| **资源策略** | 账户 B 在资源上（如 Bucket Policy）授权账户 A 的主体 | **保留** | 需要同时操作两边资源 |

> **考试陷阱：扮演角色会放弃原有权限**——"从账户 A 的 DynamoDB 读数据并写入账户 B 的 S3" → 用账户 B 的 **S3 Bucket Policy** 授权账户 A 的角色（保留对 A 的 DynamoDB 权限），而不是 AssumeRole 进 B。

---

## 7. 身份联邦（Federation）

### SAML 2.0（企业级 SSO）

- **工作流**：企业 IdP（如 AD）验证用户 → 发送 **SAML 断言**给 AWS → **STS** 发放临时令牌 → 登录 AWS 控制台，无需在 AWS 创建 IAM 用户

### AWS IAM Identity Center（现代化 SSO 方案）

- 原名 **AWS SSO**，AWS 官方推荐的**多账户、多应用统一身份管理**方案，构建在 [[AWS Organizations]] 之上
- 可连接 AD、Okta 等第三方 IdP 或使用内置身份存储，登录一次访问所有被授权的成员账户和应用
- 通过**权限集（Permission Set）**集中管理，优于逐账户手动配置 SAML 联邦

### 身份工具选型

| 工具 | 面向 |
|------|------|
| **IAM** | **AWS 账户内部**身份（管理员、运维、AWS 资源本身） |
| **IAM Identity Center** | **企业员工**跨多账户/多应用的统一 SSO |
| **[[Cognito]]** | **外部 App 终端用户**（移动/Web 应用消费者） |
| **[[AWS Directory Service]]（Managed Microsoft AD）** | 把**本地 AD 域**扩展到云端，供依赖 Windows AD 的应用使用 |

---

## 8. 安全强化与审计

| 工具 | 用途 |
|------|------|
| **MFA** | 虚拟 MFA（手机 App）/ 硬件 MFA / U2F·FIDO 安全密钥（防钓鱼最强）；配合 `aws:MultiFactorAuthPresent` 保护敏感操作 |
| **IAM Access Analyzer** | 持续分析资源策略，发现**意外授予外部账户/公网**的访问；也能基于访问活动生成最小权限策略 |
| **凭证报告（Credential Report）** | 账户级 CSV，列出所有用户密码/密钥状态和最后使用时间 → 审计未轮换/未使用的凭证 |
| **访问顾问（Access Advisor）** | 显示某身份**实际用过**哪些服务 → 收紧多余权限 |

---

## 9. 考试速查

### 9.1 场景 → 答案

```
场景分析 → 判断使用 IAM 的哪个机制
├── "AWS 资源需要访问其他 AWS 服务" → IAM 角色（而非硬编码密钥）
├── "跨账户访问另一账户的资源" → 目标账户角色 + 信任策略 + AssumeRole
├── "跨账户且需同时操作两边资源" → 目标资源的资源策略（保留原有权限）
├── "IAM 策略已授权但跨账户访问失败" → 检查目标账户资源策略
├── "同账户内无 IAM 策略但 Bucket Policy 授权了该用户" → 可以访问
├── "允许和拒绝冲突" → 显式 Deny 始终优先
├── "限制单个用户/角色权限上限，即使误授管理员" → 权限边界（不能挂组）
├── "限制整个组织/账户权限上限" → SCP
├── "限制只能在指定区域操作" → Deny + NotAction(全局服务) + StringNotEquals aws:RequestedRegion
├── "只允许公司 IP，但 CloudFormation 等服务调用失败" → 加 aws:ViaAWSService
├── "S3 只允许从某个 VPC 访问" → aws:SourceVpce / aws:SourceVpc
├── "企业员工用现有 AD 账号登录多个 AWS 账户" → IAM Identity Center / SAML 联邦
├── "发现被意外授予外部访问的资源" → IAM Access Analyzer
├── "审计长期未使用/未轮换的凭证" → 凭证报告
└── "外部 App 终端用户注册登录" → Cognito
```

### 9.2 SAA-C03 高频考点

1. **角色是服务间授权和跨账户访问的标准方式**，不要硬编码密钥
2. **组不是主体**：不能 Assume、不能写进 Principal、不能挂权限边界
3. **评估顺序**：显式 Deny → SCP/RCP → 资源策略 → 身份策略 → 权限边界 → 会话策略
4. **同账户一边 Allow 即可，跨账户两边都要 Allow**
5. **限制型机制只收窄不授予**：权限边界（单身份）、SCP（组织）、会话策略（单会话），取交集
6. **AssumeRole 放弃原有权限，资源策略保留原有权限**
7. **`aws:RequestedRegion` 限的是目标区域**；区域护栏要用 `NotAction` 豁免全局服务
8. **`aws:SourceIp` 陷阱**：服务代调用用 `aws:ViaAWSService`，经 VPC Endpoint 用 `aws:SourceVpce`
9. **信任策略 vs 权限策略**：谁能扮演 vs 扮演后能做什么
10. **IAM Identity Center** 是现代化多账户 SSO 方案

---

## 最佳实践

1. **Root 用户仅用于必要操作**：启用 MFA、删除访问密钥
2. **优先使用角色而非长期密钥**：服务间授权、跨账户访问都用临时凭证
3. **最小特权**：结合 Access Advisor 收紧未使用的权限
4. **用策略变量减少重复策略**：如按用户名限定 S3 前缀
5. **高权限角色叠加权限边界**：防止误操作或提权
6. **多账户环境用 IAM Identity Center**
7. **定期审查凭证报告和 Access Analyzer 发现项**
8. **敏感操作要求 MFA**（`aws:MultiFactorAuthPresent`）
9. **优先使用可复用的托管策略**，内联策略仅用于与特定身份强绑定的场景
