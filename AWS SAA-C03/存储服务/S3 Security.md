# Amazon S3 Security - 访问控制与加密

> **S3 安全**横跨身份访问控制、资源策略、网络隔离、静态/传输加密和审计合规等多个维度，是 SAA-C02 考试中最高频的主题之一。本笔记深入展开 [[S3]] 中"安全性"一节未详述的机制。
>
> 相关文档：[[S3]] | [[IAM]] | [[KMS]] | [[VPC]] | [[CloudTrail]] | [[Amazon GuardDuty]] | [[Amazon Macie]]

---

## 核心概念

### 访问控制的三层机制（考试重点）

| 机制 | 类型 | 绑定对象 | 特点 |
|------|------|---------|------|
| **IAM 策略** | 基于身份（Identity-based） | 用户/角色/组 | 控制该身份**能访问哪些资源**，适合管理内部用户权限 |
| **Bucket Policy** | 基于资源（Resource-based） | Bucket | 控制**谁能访问这个 Bucket**，无需对方扮演本账户角色即可实现跨账户授权（对方账户仍需用自己的 IAM 策略把权限分配给具体用户/角色） |
| **ACL（访问控制列表）** | 基于资源（旧机制） | Bucket / 单个对象 | **AWS 已不推荐使用**，仅在极少数需要跨账户对象级授权的场景保留 |

> **考试要点**：**同一账户内**，IAM 策略和 Bucket Policy 是**并集**关系——只要任意一个允许访问且没有显式拒绝，请求就会通过。二者冲突时，**显式 Deny 始终优先**于任何 Allow。
>
> **跨账户时是交集**：Bucket Policy 允许账户 B 访问后，账户 B 中的用户/角色**还需要自己账户的 IAM 策略也允许**，请求才会通过。

### 何时用 IAM 策略 vs Bucket Policy

```
选择访问控制方式
├── "只在本账户内，管理用户/角色能访问哪些 Bucket" → IAM 策略
├── "跨账户授权，且不想要求对方创建 IAM 角色" → Bucket Policy
├── "需要基于请求来源（IP、VPC、协议）做限制" → Bucket Policy（Condition 键）
└── "需要匿名公开访问（如静态网站）" → Bucket Policy
```

---

## Bucket Policy 深入解析

### 策略结构

Bucket Policy 是附加在 Bucket 上的 JSON 文档，核心字段：

| 字段 | 说明 |
|------|------|
| **Principal** | 谁受此策略影响（账户、用户、角色、`*` 表示所有人） |
| **Action** | 允许/拒绝的 S3 API 操作（如 `s3:GetObject`） |
| **Resource** | 目标 Bucket/对象 ARN |
| **Condition** | 附加限制条件（IP、VPC、加密方式、传输协议等） |

### 常见 Condition 键（考试高频）

| Condition 键 | 用途 |
|-------------|------|
| **`aws:SecureTransport`** | 拒绝非 HTTPS 请求，强制加密传输 |
| **`aws:SourceIp`** | 限制仅允许特定 IP/CIDR 段访问 |
| **`aws:SourceVpce`** | 限制仅允许通过特定 **VPC Endpoint** 访问 |
| **`s3:x-amz-server-side-encryption`** | 拒绝未加密的上传请求（强制服务端加密） |
| **`aws:MultiFactorAuthPresent`** | 要求请求必须携带 MFA 验证（配合 MFA Delete） |

> **考试陷阱**：题目描述"要求所有上传对象必须加密，未加密的 PUT 请求应被拒绝"——正确做法是在 Bucket Policy 中添加 `Deny` 规则，条件为 `s3:x-amz-server-side-encryption` **不存在**，而不是仅仅依赖"默认加密"设置（默认加密不会拒绝显式指定为非加密的请求）。

---

## Block Public Access（考试高频）

S3 提供账户级和 Bucket 级的"公共访问阻止"开关，**独立于**任何 Bucket Policy 或 ACL 生效，是防止数据泄露的最后一道防线。

| 设置项 | 作用 |
|--------|------|
| **阻止通过新 ACL 授予的公共访问** | 忽略新增的公开 ACL |
| **阻止通过任意 ACL 授予的公共访问** | 忽略所有公开 ACL（含已存在的） |
| **阻止通过新公共 Bucket Policy 授予的访问** | 拒绝设置允许公开访问的新策略 |
| **阻止通过任意 Bucket Policy 授予的公共和跨账户访问** | 忽略所有允许公开访问的策略 |

> **考试要点**：即使 Bucket Policy 显式允许 `Principal: "*"`，只要 Block Public Access 处于开启状态，公共访问请求依然会被**拒绝**。新建 Bucket 默认**全部开启**这四项设置。

---

## 加密（考试高频）

### 静态加密类型对比

| 类型 | 密钥管理方 | 特点 | 适用场景 |
|------|-----------|------|---------|
| **SSE-S3** | AWS 全权管理（AES-256） | 默认加密方式，无需额外配置；**每个对象在写入时都已自动使用唯一的数据密钥加密**（该密钥本身又被定期轮换的根密钥加密） | 无特殊合规要求的通用场景 |
| **SSE-KMS** | [[KMS]] 管理，客户可控密钥策略 | 支持密钥轮换、细粒度权限、**CloudTrail 审计每次密钥使用** | 需要审计、合规、密钥访问控制的场景 |
| **SSE-C** | 客户提供密钥，AWS 仅负责加解密过程 | AWS **不存储**密钥，请求中必须携带 | 客户自主管理密钥且不信任 AWS 存储密钥 |
| **客户端加密** | 客户端本地完成 | 数据上传前已加密，AWS 只存储密文 | 最高安全要求，AWS 完全不接触明文/密钥 |

### "每个文件用不同密钥加密"不需要额外设计（考试易混淆点）

> 需求描述"希望每个文件都用不同的密钥加密，以提供最大安全性"时，第一反应不应该是去设计方案让每个对象拿到不同密钥——**SSE-S3 默认就已经为每个对象分配唯一的加密密钥**，不需要任何额外配置（不需要拆分成多个桶、不需要用 KMS 加密上下文人为构造"每文件不同密钥"、也不需要客户端加密）。

- **常见干扰项 1**：把数据按逻辑拆分到不同的 S3 桶，每个桶单独用 SSE-S3 加密——多余的复杂度，SSE-S3 在单个桶内已经做到了对象级别的唯一密钥
- **常见干扰项 2**：用 SSE-KMS 的"加密上下文（Encryption Context）"试图为每个对象生成不同密钥——加密上下文只是附加的校验/审计信息，**不会**也不是用来生成唯一密钥的机制
- **常见干扰项 3**：用客户端加密或多区域密钥（Multi-Region Keys）手动管理每个文件的密钥——这是服务端加密场景下完全不必要的额外运维负担

> **考试关键词识别**：题目描述"数据已从多密钥的本地环境迁移到 S3，希望继续保持每个文件独立密钥加密，且不想承担额外的拆分/管理开销" → **单个 S3 桶 + SSE-S3 即可**，这是最省事且已经满足要求的方案。

---

### SSE-KMS 的额外考量

- 每次 GET/PUT 都会调用 KMS API，**大量小对象访问可能触及 KMS 请求限流（每秒配额）**
- **S3 Bucket Keys** 可减少调用 KMS 的次数（在 Bucket 级别生成数据密钥的信封密钥），显著降低 KMS 请求量和成本
- 可通过 IAM/KMS 密钥策略精细控制**谁能解密**，实现比 SSE-S3 更强的访问控制

### 强制加密的两种方式

| 方式 | 说明 |
|------|------|
| **默认加密（Default Encryption）** | Bucket 级别设置，未指定加密方式的请求自动应用；但不会**拒绝**显式声明为不加密的请求 |
| **Bucket Policy 显式 Deny** | 通过 Condition 拒绝不符合加密要求的 PUT 请求，是唯一能**强制阻止**未加密上传的方式 |

---

## 传输加密

- 强制 HTTPS：在 Bucket Policy 中使用 `aws:SecureTransport: false` 条件 Deny 所有非 HTTPS 请求
- S3 所有 API 端点均同时支持 HTTP 和 HTTPS，**默认不强制加密传输**，需显式配置策略

---

## 网络层访问控制

### VPC Endpoint + Bucket Policy 组合限制

- 通过 **Gateway [[VPC Endpoints|VPC Endpoint]]** 让 VPC 内资源私有访问 S3，不经过公网
- 配合 Bucket Policy 的 `aws:SourceVpce` 条件，可实现"**仅允许通过该 VPC Endpoint 的请求访问 Bucket**"，即使凭证泄露，公网发起的请求依然会被拒绝

> **考试要点**：这是"限制 S3 Bucket 仅能被特定 VPC 内的 EC2 实例访问，即使拥有正确凭证也不能从公网访问"这类场景题的标准答案组合。

### CORS（跨源资源共享）

- 浏览器的**同源策略**默认会阻止网页从一个域（如 `example.com`）用 JavaScript 直接向另一个域（如 S3 Bucket 的域名）发起请求
- S3 需在 Bucket 上配置 **CORS 规则**（JSON/XML），显式声明允许哪些来源域（`AllowedOrigin`）、方法（`AllowedMethod`：GET/PUT/POST 等）、请求头（`AllowedHeader`）
- CORS 与 Bucket Policy/IAM 是**互补而非替代**关系：CORS 只解决浏览器端的跨域请求许可问题，不授予任何实际的 S3 访问权限——即便 CORS 规则允许，请求仍必须先通过身份/资源策略的鉴权

> **考试陷阱**：题目描述"托管在 CloudFront/其他域名的网页，通过 JavaScript（`fetch`/`XMLHttpRequest`）直接向 S3 上传文件或加载资源时被浏览器拦截，报 CORS 错误"——正确答案是在 S3 Bucket 上**配置 CORS 规则**，而不是修改 Bucket Policy 或 IAM 策略（这两者无法解决浏览器同源策略的拦截）。

---

## Object Lock（WORM 合规锁定，考试高频）

用于满足**一次写入多次读取（WORM）**合规要求，防止对象在保留期内被删除或覆盖。

| 模式 | 说明 | 适用场景 |
|------|------|---------|
| **治理模式（Governance）** | 默认阻止删除/覆盖，但拥有特殊 IAM 权限（`s3:BypassGovernanceRetention`）的用户可绕过 | 内部数据保护，允许授权人员紧急处理 |
| **合规模式（Compliance）** | **任何人（包括 Root 账户）都无法**删除或缩短保留期，直到期满 | 监管强制的法律合规场景（如金融、医疗记录） |
| **法律保留（Legal Hold）** | 无固定到期时间，独立于保留期存在，需显式移除 | 诉讼/调查期间需无限期保留的证据数据 |

> **考试陷阱**：Object Lock **依赖版本控制**，启用 Object Lock 必须同时启用版本控制（且之后不能暂停版本控制）。早期 Object Lock 只能在创建 Bucket 时启用，旧题答案可能仍以此为准；**自 2023 年 11 月起，已可对已存在且开启版本控制的 Bucket 启用 Object Lock**。

---

## MFA Delete

- 要求删除对象版本或**禁用版本控制**时必须提供有效的 MFA 一次性密码
- 只能通过 **Bucket 拥有者（Root 账户）** 使用 CLI/API 启用（控制台不支持配置该项）
- 用于防止误删除或恶意删除关键数据

---

## S3 Access Points（考试易漏点）

> 当**同一个 Bucket**需要被多个团队/应用共享，且每个团队只应看到自己的前缀（prefix），而不是维护一份越来越复杂的 Bucket Policy 时，**S3 Access Points** 是专为此设计的机制。

### 是什么

Access Point 是挂在某个 Bucket 上的**具名网络入口**。一个 Bucket 可以挂很多个 Access Point，每个 Access Point 有：

| 组成部分 | 说明 |
|---------|------|
| **独立的名称 / ARN / 主机名** | ARN 形如 `arn:aws:s3:<region>:<account-id>:accesspoint/<name>`，应用通过它而不是 Bucket 名访问数据 |
| **Access Point 别名（Alias）** | 自动生成形如 `<name>-<随机串>-s3alias` 的别名，**凡是接受 Bucket 名的地方都可直接使用**，老应用无需改代码 |
| **独立的 Access Point Policy** | 与 Bucket Policy 语法相同的资源策略，只作用于经由该入口的请求，可限定到某个前缀（如 `finance/*`） |
| **网络来源（Network Origin）** | `Internet` 或 `VPC`；设为 VPC 时只接受来自指定 VPC 的请求，**创建后不可修改** |
| **独立的 Block Public Access 设置** | 每个 Access Point 可单独开启阻止公共访问 |

```
                      ┌── AP: finance-ap   (策略: 仅 Finance 角色, 仅 finance/*)  ──┐
应用/团队 ──请求────▶ ├── AP: analytics-ap (策略: 只读 GetObject, 仅 logs/*)      ├──▶ 同一个 S3 Bucket
                      └── AP: vpc-only-ap  (Network Origin = VPC-A)              ──┘
```

### 权限评估：Bucket Policy 与 Access Point Policy 必须都允许

经由 Access Point 的请求需要**同时**通过 Access Point Policy 和底层 Bucket Policy（以及调用者的 IAM 策略）。为避免在两处重复维护规则，标准做法是**在 Bucket Policy 中把访问控制委托给 Access Point**：

```json
{
  "Effect": "Allow",
  "Principal": { "AWS": "*" },
  "Action": "*",
  "Resource": ["arn:aws:s3:::my-bucket", "arn:aws:s3:::my-bucket/*"],
  "Condition": { "StringEquals": { "s3:DataAccessPointAccount": "111122223333" } }
}
```

含义：凡是经由本账户自有 Access Point 进来的请求，Bucket 层都放行，具体权限由各 Access Point Policy 决定。Bucket Policy 只需写一次，之后新增团队只需新建 Access Point。

> 进一步收紧：可在 Bucket Policy 中添加 Deny，配合 `s3:DataAccessPointArn` / `s3:AccessPointNetworkOrigin` 条件键，**禁止绕过 Access Point 直接访问 Bucket**，或要求只能经由 VPC 类型的 Access Point 访问。

### VPC 限定的 Access Point

- Network Origin 设为 `VPC` 后，只有从该 VPC 内（通过 [[VPC Endpoints|S3 VPC Endpoint]]）发出的请求能使用该入口，公网请求一律拒绝
- 与"Bucket Policy + `aws:SourceVpce`"的效果类似，但粒度是**每个 Access Point 一个 VPC**，适合多个 VPC/团队分别访问同一个 Bucket 的场景
- VPC Endpoint 的 Endpoint Policy 也需要允许访问该 Access Point，否则请求会在 Endpoint 处被拒

### 限制与注意事项

- Access Point 只支持**对象级操作**（GetObject、PutObject、ListObjects 等）以及部分 Bucket 操作，不能用来做删除 Bucket、修改 Bucket 配置等管理操作
- Access Point 与 Bucket 必须在**同一 Region**
- 支持**跨账户 Access Point**：Bucket 拥有者可以允许其他账户在其 Bucket 上创建 Access Point（需 Bucket Policy 授权）

### 两种衍生的 Access Point（考试会对比）

| 类型 | 作用 | 典型考法 |
|------|------|---------|
| **S3 Multi-Region Access Points** | 一个**全局端点**对应多个 Region 的 Bucket，借助 AWS Global Accelerator 网络把请求路由到**延迟最低**的 Bucket；配合 [[S3]] 跨区域复制（CRR）保持数据同步，支持主备故障转移 | "全球用户访问多 Region 数据，要求单一端点 + 最低延迟 + Region 故障时自动切换" |
| **S3 Object Lambda Access Points** | 在 GET 请求返回前调用 [[Lambda]] 函数**即时改写数据**（如脱敏 PII、调整图片尺寸、格式转换），无需存储多份副本 | "同一份数据给不同应用返回不同视图（如对分析团队隐去敏感字段），且不想复制数据" |

> 注：AWS 已于 2025 年 11 月起停止向新客户开放 S3 Object Lambda，但它仍可能出现在考题中，需能识别。

> **考试关键词识别**：题目描述"多个应用/服务共享同一个 S3 Bucket，每个只能访问自己的前缀，要求方案可扩展、易维护、运维负担最小" → **S3 Access Points**（为每个应用创建独立访问点+独立策略），而不是硬编码堆砌一份枚举所有对象 ARN 的 Bucket Policy，也不是用 Macie（Macie 只做敏感数据发现分类，不做访问控制）。

---

## 跨账户访问模式对比

| 模式 | 机制 | 适用场景 |
|------|------|---------|
| **Bucket Policy 授权** | 直接在策略中指定对方账户 Principal | 简单场景，对方无需扮演角色，用自身凭证直接访问（仍需对方 IAM 策略允许） |
| **IAM 角色 + AssumeRole** | 对方通过 STS 临时凭证扮演本账户角色 | 需要精细审计、临时访问、遵循最小权限原则 |
| **S3 Access Points** | 为不同团队/应用创建独立访问入口，各自绑定策略 | 大规模共享 Bucket，简化多团队权限管理 |
| **跨账户 ACL（已弃用）** | 对象级 ACL 授权其他账户 | 遗留系统，不建议新架构使用 |

---

## 审计与威胁检测

| 工具 | 用途 |
|------|------|
| **[[CloudTrail]] 数据事件** | 记录每次对象级 API 调用（GetObject/PutObject 等），默认关闭，需显式启用（产生额外费用） |
| **S3 Access Logging** | 记录访问 Bucket 的请求日志，存储到另一 Bucket |
| **IAM Access Analyzer for S3** | 自动检测意外暴露给外部账户或公网的 Bucket |
| **[[Amazon Macie]]** | 使用机器学习自动发现 S3 中的敏感数据（PII、密钥等）并评估风险，完整能力见 [[Amazon Macie]] 独立笔记 |
| **[[Amazon GuardDuty]] S3 Protection** | 检测异常 API 调用模式，识别潜在的凭证泄露或恶意访问 |

---

## 考试重点总结

### SAA-C02 高频考点

1. **IAM 策略 vs Bucket Policy**：并集生效，显式 Deny 优先于任何 Allow
2. **Block Public Access**：独立于策略生效，是防止意外公开的最后防线，即使 Policy 允许公开也会被拒绝
3. **强制加密上传**：必须用 Bucket Policy 的 Deny 规则，默认加密无法拒绝显式非加密请求
4. **强制 HTTPS**：`aws:SecureTransport` 条件
5. **私有网络限制**：VPC Endpoint + `aws:SourceVpce` 条件组合，实现仅限特定 VPC 访问
6. **CORS**：解决浏览器跨域请求拦截问题，与 IAM/Bucket Policy 是互补关系，不能替代鉴权
7. **SSE-KMS vs SSE-S3**：需要审计密钥使用或精细密钥权限控制时选 SSE-KMS，注意 KMS 请求限流问题（用 S3 Bucket Keys 缓解）
8. **Object Lock**：治理模式可绕过、合规模式任何人不可绕过，必须配合版本控制
9. **MFA Delete**：只能通过 Root 账户 CLI/API 启用，防止误删/恶意删除
10. **跨账户访问**：优先考虑 Bucket Policy 或 IAM 角色 AssumeRole，而非已弃用的 ACL
11. **S3 Access Points**：多团队共享同一 Bucket 时，每个团队一个入口 + 独立策略；Bucket Policy 通过 `s3:DataAccessPointAccount` 委托授权；Multi-Region AP 提供全局单一端点

### 场景题解题思路

```
场景分析 → 选择 S3 安全机制
├── "跨账户授权访问，对方无需配置 IAM" → Bucket Policy
├── "即使策略允许公开，也要确保不能被公网访问" → Block Public Access
├── "上传的对象必须强制加密，拒绝未加密请求" → Bucket Policy Deny（条件：无加密头）
├── "所有请求必须走 HTTPS" → Bucket Policy Deny（aws:SecureTransport: false）
├── "仅允许特定 VPC 内的实例访问，即使凭证正确" → VPC Endpoint + aws:SourceVpce 条件
├── "网页 JS 直接访问 S3 被浏览器拦截，报跨域错误" → 配置 CORS 规则
├── "需要审计每次密钥使用，或精细控制谁能解密" → SSE-KMS
├── "监管要求对象在保留期内任何人都不能删除" → Object Lock 合规模式
├── "内部场景允许特殊权限人员紧急删除" → Object Lock 治理模式
├── "防止误删除关键数据版本" → MFA Delete
├── "多个团队共享同一 Bucket，各自只能访问自己的前缀，易扩展" → S3 Access Points
├── "多 Region Bucket 对全球用户提供单一端点 + 就近路由" → S3 Multi-Region Access Points
├── "不复制数据，按调用方返回脱敏/改写后的对象" → S3 Object Lambda Access Points
└── "自动发现存储的敏感数据（PII）" → [[Amazon Macie]]
```

---

## 最佳实践

1. **默认保持 Block Public Access 全部开启**：仅在明确需要公开访问（如静态网站）时针对性关闭
2. **优先使用 Bucket Policy 而非 ACL**：ACL 是遗留机制，新架构应完全避免
3. **强制加密和 HTTPS 用 Deny 策略而非仅依赖默认设置**：默认加密不能阻止显式的非加密请求
4. **对合规数据启用 Object Lock 合规模式**：确保保留期内数据不可篡改
5. **敏感数据场景启用 SSE-KMS 并配合 S3 Bucket Keys**：兼顾审计能力与成本
6. **跨账户访问优先选择 IAM 角色 AssumeRole**：比 Bucket Policy 更利于审计和临时权限管理
7. **启用 CloudTrail 数据事件 + [[Amazon Macie]]**：对存放敏感数据的 Bucket 做持续的访问审计和内容扫描
8. **定期运行 IAM Access Analyzer for S3**：主动发现意外的公开或跨账户暴露
