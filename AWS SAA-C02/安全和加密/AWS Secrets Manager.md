# AWS Secrets Manager - 凭证管理与自动轮换

> **AWS Secrets Manager** 是专为**凭证类密钥**（数据库密码、API 密钥、第三方服务令牌等）设计的托管服务，核心差异化能力是**内置的自动轮换（Automatic Rotation）**——无需人工介入即可周期性更换凭证并同步给所有依赖方，从根源上减少长期静态凭证泄露的风险。
>
> 相关文档：[[KMS]] | [[AWS Systems Manager]] | [[RDS Proxy]] | [[RDS]] | [[IAM]] | [[AWS Lambda]] | [[VPC]]

---

## 核心概念

### 为什么需要 Secrets Manager

- **长期静态凭证的风险**：应用代码或配置文件中硬编码数据库密码等凭证，一旦泄露就是**永久有效**的攻击入口，且人工定期轮换凭证既繁琐又容易遗漏
- **Secrets Manager 的核心价值**：**集中存储 + 自动轮换 + 按需检索**——应用运行时通过 API 动态获取最新凭证（而非硬编码），Secrets Manager 按计划自动生成新凭证、更新目标服务（如数据库）、并让所有取用方无缝拿到最新版本
- **静态加密**：所有存储的密钥值底层均使用 [[KMS]] 加密，Secrets Manager 本身不是加密服务，而是在 KMS 之上构建的**凭证生命周期管理层**

### Secrets Manager 在安全工具链中的定位（考试高频）

| 服务 | 核心定位 | 自动轮换 | 典型场景 |
|------|---------|---------|---------|
| **Secrets Manager** | 凭证（Secret）的集中管理 | ✅ **内置托管轮换** | 数据库密码、API 密钥等需要定期轮换的凭证 |
| **[[AWS Systems Manager]] Parameter Store** | 通用配置项/参数存储 | ❌ 无内置轮换（需自行实现） | 非敏感或低频变更的配置项，SecureString 可选加密 |
| **[[KMS]]** | 加密密钥本身的管理 | 密钥材料轮换（非凭证轮换） | 为 Secrets Manager/S3/EBS 等提供底层加密能力 |

> **考试陷阱**：**三者常在选项中一起出现，判断依据是"存的是什么"和"是否需要自动轮换"**——存的是**数据库密码/API 密钥且需要定期自动轮换** → Secrets Manager；存的是**普通配置项**（端点地址、功能开关）→ Parameter Store（成本更低）；管理的是**加密密钥本身**而非凭证内容 → KMS。题目强调"自动轮换数据库密码，轮换期间不能中断连接"是 Secrets Manager 的经典识别特征。

---

## 自动轮换（Automatic Rotation，考试高频）

### 托管轮换（Managed Rotation）

- AWS 为常见服务提供**预置的轮换 Lambda 函数**，覆盖 [[RDS]]（MySQL/PostgreSQL/MariaDB/Oracle/SQL Server）、Redshift、DocumentDB 等，只需在控制台启用轮换并设置周期，无需自行编写轮换逻辑
- 轮换过程自动完成"生成新密码 → 更新数据库 → 更新 Secret 存储"的完整链路

### 自定义轮换（Custom Rotation）

- 面向 AWS 预置模板未覆盖的服务（如第三方 SaaS API 密钥），可编写自定义 Lambda 函数实现轮换逻辑，遵循标准的**四步协议**：
  1. **createSecret**：生成新的凭证值
  2. **setSecret**：在目标系统（如数据库）中设置新凭证
  3. **testSecret**：验证新凭证可正常工作
  4. **finishSecret**：将新版本标记为当前生效版本，完成切换
- 四步协议设计确保轮换过程中**旧凭证在新凭证验证通过前不会失效**，实现零停机轮换

### 零接触轮换（Zero-Touch Rotation，新特性）

- 面向**第三方/非 AWS 服务**的凭证，进一步减少人工介入和对凭证明文的直接接触，提供自动化轮换、集中可见性等能力，扩展了轮换能力覆盖的边界，不再局限于 AWS 原生服务

> **考试要点**：**轮换期间业务连接不中断是 Secrets Manager 的关键卖点**——尤其是结合 **[[RDS Proxy]]** 使用时，RDS Proxy 会自动感知 Secrets Manager 中凭证的轮换并平滑切换连接池使用的凭证，应用侧完全无感知，这是"如何在不中断数据库连接的前提下轮换密码"这类场景题的标准答案组合。

---

## 版本与暂存标签（Staging Labels）

- 每次轮换或更新会生成 Secret 的**新版本**，通过**暂存标签**标记版本所处阶段：
  - **AWSCURRENT**：当前生效、应用应使用的版本
  - **AWSPENDING**：轮换过程中正在测试的新版本
  - **AWSPREVIOUS**：上一个生效版本，轮换刚完成时仍可用于短暂的回退场景
- 这种多版本并存机制是四步轮换协议能够实现"零停机切换"的底层支撑

---

## 多区域复制（Multi-Region Replication，考试要点）

- Secrets Manager 支持将 Secret **自动复制**到多个 AWS 区域，且**即使主区域的 Secret 因轮换而更新，副本区域也会保持同步**
- **典型场景**：全球分布式应用需要在多个区域读取同一份数据库凭证；灾难恢复场景下，次要区域可直接使用已同步的凭证副本，无需等待手动复制
- 2025 年更新进一步降低了跨区域复制延迟（多数区域可做到亚秒级同步），并增强了自动故障转移能力

> **考试陷阱**：**多区域复制是 Secrets Manager 相较于 Parameter Store 的差异化能力之一**——Parameter Store 本身不提供跨区域自动同步凭证值的原生机制；题目强调"全球应用需要在多个区域访问同一份自动同步的数据库凭证"应识别为 Secrets Manager 的多区域复制特性。

---

## 访问控制与集成

| 机制 | 说明 |
|------|------|
| **IAM 策略 + 资源策略** | 精细控制哪些身份可以读取/管理哪些 Secret，资源策略支持跨账户共享 Secret |
| **[[RDS Proxy]] 集成** | 数据库凭据存储在 Secrets Manager，RDS Proxy 自动获取并使用，轮换时**无需中断已建立的连接**（详见 [[RDS Proxy]] 笔记） |
| **ECS/EKS 集成** | 容器可通过环境变量注入或 Secrets Store CSI Driver 等方式，在启动时安全获取存储在 Secrets Manager 中的凭证，避免将敏感值写入容器镜像或任务定义明文 |
| **[[AWS Lambda]] 轮换函数** | 无论是托管轮换还是自定义轮换，底层执行体都是 Lambda 函数 |

---

## 计费与容量考量

- 按**每个 Secret 的月度存储费用**加上**API 调用次数**计费，成本显著高于 Parameter Store（Parameter Store 标准层免费）
- **考试要点**：**成本敏感、无需自动轮换的通用配置项，不应使用 Secrets Manager**——题目描述"只是存储一些不敏感的配置参数，且预算有限"，更合适的答案是 Parameter Store，而非为了统一工具链而滥用 Secrets Manager 存储所有配置

---

## 安全性

| 机制 | 说明 |
|------|------|
| **静态加密** | 所有 Secret 值默认使用 [[KMS]] 加密存储 |
| **传输加密** | API 调用默认通过 HTTPS/TLS |
| **VPC Endpoint** | 支持通过接口终端节点在 [[VPC]] 内私有访问 Secrets Manager API，无需经过公网 |
| **审计** | 可与 CloudTrail 集成，记录每一次 Secret 的访问和轮换操作 |
| **细粒度资源策略** | 支持基于条件的跨账户/跨角色精细授权 |

---

## 典型应用场景

| 场景 | 推荐配置 |
|------|---------|
| **数据库密码需要定期自动轮换** | Secrets Manager 托管轮换（RDS/Redshift/DocumentDB） |
| **第三方 SaaS API 密钥的自定义轮换** | Secrets Manager 自定义轮换 Lambda（四步协议） |
| **轮换期间不能中断数据库连接** | Secrets Manager + [[RDS Proxy]] 组合 |
| **全球应用需要多区域同步访问同一凭证** | 多区域复制（Multi-Region Replication） |
| **容器化应用需要安全获取运行时凭证** | ECS/EKS 集成 Secrets Manager，避免凭证写入镜像 |
| **第三方/非 AWS 服务的凭证轮换** | 零接触轮换（Zero-Touch Rotation） |
| **仅需存储不敏感、低成本的配置项** | 改用 Parameter Store（而非 Secrets Manager） |
| **需要管理的是加密密钥本身而非凭证** | 改用 [[KMS]]（而非 Secrets Manager） |

---

## 考试重点总结

### SAA-C02 高频考点

1. **核心定位**：专为凭证设计的托管服务，核心差异化能力是内置自动轮换
2. **判断依据**：需要"自动轮换凭证"选 Secrets Manager；仅需"存储配置项"选 Parameter Store（成本更低）；管理"加密密钥本身"选 KMS
3. **托管轮换覆盖 RDS/Redshift/DocumentDB**：预置 Lambda 函数，无需自行编写轮换逻辑
4. **自定义轮换遵循四步协议**：createSecret → setSecret → testSecret → finishSecret，确保零停机切换
5. **暂存标签支持多版本并存**：AWSCURRENT/AWSPENDING/AWSPREVIOUS 是轮换过程平滑切换的关键机制
6. **结合 RDS Proxy 实现轮换不中断连接**：这是"数据库密码轮换零停机"场景题的标准答案组合
7. **多区域复制是差异化能力**：Secret 自动同步到多区域，且轮换后副本保持同步，Parameter Store 无此原生能力
8. **计费显著高于 Parameter Store**：按 Secret 数量 + API 调用计费，成本敏感场景需权衡是否真的需要自动轮换
9. **底层加密依赖 KMS**：Secrets Manager 本身不是加密服务，而是凭证生命周期管理层
10. **零接触轮换扩展到第三方服务**：不再局限于 AWS 原生服务的凭证轮换

### 场景题解题思路

```
场景分析 → 判断是否用 Secrets Manager
├── "数据库密码需要定期自动轮换" → Secrets Manager 托管轮换
├── "第三方 API 密钥需要自定义轮换逻辑" → Secrets Manager 自定义轮换 Lambda
├── "轮换密码时不能中断已有数据库连接" → Secrets Manager + RDS Proxy
├── "全球应用需要多区域同步访问同一凭证" → 多区域复制
├── "容器化应用需要安全获取运行时凭证，避免写入镜像" → ECS/EKS 集成 Secrets Manager
├── "只需存储不敏感的配置参数，预算有限" → 改用 Parameter Store（而非 Secrets Manager）
├── "管理的是加密密钥本身，而非凭证内容" → 改用 KMS（而非 Secrets Manager）
└── "第三方/非 AWS 服务凭证需要自动化轮换和集中可见性" → 零接触轮换（Zero-Touch Rotation）
```

---

## 最佳实践

1. **数据库等原生支持的服务优先使用托管轮换**：无需自行开发和维护轮换 Lambda 函数
2. **应用代码通过 API 动态获取凭证，杜绝硬编码**：从根源消除长期静态凭证泄露的风险
3. **高可用数据库场景结合 RDS Proxy**：实现凭证轮换对应用完全透明，避免连接中断
4. **全球分布式应用启用多区域复制**：避免跨区域手动同步凭证带来的一致性和运维负担
5. **成本敏感、无需轮换的配置项改用 Parameter Store**：避免为通用配置支付 Secrets Manager 的额外成本
6. **容器工作负载避免将凭证写入镜像或环境变量硬编码**：运行时从 Secrets Manager 动态获取
7. **结合资源策略实现最小权限的跨账户共享**：避免为跨账户访问过度放宽 IAM 权限
8. **定期审查 CloudTrail 中的 Secret 访问记录**：识别异常的高频访问或未预期的调用方
