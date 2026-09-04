# AWS Control Tower - 多账户着陆区自动化

> **AWS Control Tower** 构建在 **[[AWS Organizations]]** 之上，提供开箱即用的**着陆区（Landing Zone）**自动化能力——一键搭建符合最佳实践的多账户环境骨架、批量应用护栏（Guardrail）、标准化新账户创建流程，让企业无需从零手动拼装 Organizations、SCP、Config、CloudTrail 等组件即可获得规范的多账户治理基线。
>
> 相关文档：[[AWS Organizations]] | [[AWS Service Catalog]] | [[IAM]] | [[AWS Config]] | [[CloudTrail]] | [[AWS CloudFormation]]

---

## 核心概念

### 为什么需要 Control Tower

- **手动搭建 Organizations 治理框架的复杂度**：从零实现一套完善的多账户治理需要综合运用 Organizations（OU/SCP）、Config（合规规则）、CloudTrail（组织级审计）、IAM Identity Center（统一登录）等多个服务，涉及大量重复性的最佳实践配置工作
- **Control Tower 的核心价值**：**自动化编排**上述服务，几分钟内即可获得一套预配置好的、符合 AWS 官方推荐架构的多账户环境——不是替代 Organizations，而是在其之上提供**开箱即用的自动化和护栏管理层**

### Control Tower 与 Organizations 的关系（考试要点）

| 服务 | 定位 | 关系 |
|------|------|------|
| **[[AWS Organizations]]** | 底层的多账户管理基础设施 | SCP、RCP、OU、合并账单等原语能力 |
| **AWS Control Tower** | 构建在 Organizations 之上的自动化编排层 | 自动创建标准化的 OU 结构、批量部署护栏、提供账户创建自助服务和合规仪表盘 |

> **考试陷阱**：**Control Tower 不是 Organizations 的替代品，而是它的自动化封装**——题目描述"需要从零手动精细控制每一个 SCP 和 OU 的细节"，直接用 Organizations 更灵活；题目描述"希望快速获得一套符合最佳实践的多账户环境，不想手动拼装" → **Control Tower**。已经使用 Organizations 的账户也可以后续"注册（Enroll）"到 Control Tower 获得自动化护栏管理。

---

## 着陆区（Landing Zone）

- **着陆区**是 Control Tower 自动搭建的多账户环境骨架，默认包含：
  - **管理账户（Management Account）**：组织的顶层账户
  - **日志归档账户（Log Archive Account）**：集中存放组织级 CloudTrail 日志和 Config 快照
  - **审计账户（Audit/Security Account）**：供安全团队集中查看跨账户的合规和安全状态，通常配置为委派管理员
- 着陆区版本持续迭代（如 2025 年发布的 4.0 版本引入了服务专属资源隔离，改善了 Config/CloudTrail 等基础服务之间的隔离性），版本升级由 AWS 托管，无需用户手动重建

---

## 护栏（Guardrails / Controls，考试高频）

### 护栏的两种作用方式

| 类型 | 底层机制 | 作用时机 |
|------|---------|---------|
| **预防性护栏（Preventive）** | 底层实现为 **[[AWS Organizations]] SCP** | **事前阻止**——直接拦截被禁止的操作，操作根本无法执行 |
| **检测性护栏（Detective）** | 底层实现为 **[[AWS Config]] 规则** | **事后发现**——持续评估资源配置，发现违反策略的情况后标记为非合规 |

> **考试要点**：**预防性护栏能真正阻止操作，检测性护栏只能事后发现并标记**——这与 [[AWS Config]] 笔记中"主动型规则不会阻止部署"的结论呼应：真正的强制拦截依赖 SCP（即 Control Tower 的预防性护栏），配置检测依赖 Config 规则（即检测性护栏）。

### 三种控制级别

| 级别 | 说明 |
|------|------|
| **强制性控制（Mandatory）** | 保护 Control Tower 部署的核心资源本身，自动启用，不可关闭 |
| **强烈推荐控制（Strongly Recommended）** | 落实多账户环境的常见最佳实践，应用到 OU 级别（该 OU 下所有账户生效） |
| **可选控制（Elective）** | 面向企业常见的可选限制场景（如禁止特定区域的资源创建），按需在 OU 级别启用 |

- Control Tower 的**控制目录（Control Catalog）**已扩展至数百项预配置控制，持续新增覆盖安全、成本、持久性、运维最佳实践的托管规则

---

## Account Factory（账户工厂）

| 方式 | 说明 |
|------|------|
| **Account Factory（控制台/自助服务）** | 通过标准化的自助服务门户创建新账户，自动完成账户创建、加入指定 OU、应用护栏基线，无需手动逐项配置 |
| **Account Factory for Terraform（AFT）** | 面向偏好 **基础设施即代码（IaC）** 工作流的团队，通过 Terraform 模块和 GitOps 流水线实现账户创建的**程序化、可版本控制**管理，每次账户请求通过代码变更触发，支持 GitHub 和 GitLab |

> **考试要点**：题目强调"团队希望通过标准化门户快速自助创建符合规范的新账户" → Account Factory；强调"团队已采用 IaC/GitOps 工作流，希望账户创建也纳入代码化管理" → **Account Factory for Terraform（AFT）**。

> **考试提示**：**Account Factory 底层本质是一个 [[AWS Service Catalog]] 产品**——账户创建这一自助流程复用了 Service Catalog 的产品/组合/约束模型，而非 Control Tower 独有的技术；理解这层关系有助于回答"Control Tower 与 Service Catalog 是什么关系"这类考题，详见 [[AWS Service Catalog]] 独立笔记。

---

## 合规仪表盘（Dashboard）

- 提供组织级的**合规总览**，展示当前启用了哪些护栏、各账户/OU 的护栏遵从状态，以及是否存在**配置漂移（Drift）**——即账户配置偏离了 Control Tower 期望的基线状态
- 检测到漂移后，管理员可选择**重新基线化（Re-baseline）**该账户，将其恢复到符合 Control Tower 治理要求的状态

---

## 典型应用场景

| 场景 | 推荐配置 |
|------|---------|
| **企业首次搭建多账户环境，希望遵循最佳实践** | AWS Control Tower 一键部署着陆区 |
| **已有 Organizations 环境，希望补充自动化护栏管理** | 将现有账户/组织注册（Enroll）到 Control Tower |
| **需要真正阻止某类操作，而非仅事后发现** | 预防性护栏（底层为 SCP） |
| **需要持续监控账户配置是否符合基线，事后告警** | 检测性护栏（底层为 Config 规则） |
| **业务团队需要自助申请新 AWS 账户，且自动套用合规基线** | Account Factory |
| **账户创建流程需要纳入 IaC/GitOps 代码化管理** | Account Factory for Terraform（AFT） |
| **需要发现哪些账户偏离了治理基线** | 合规仪表盘 + 配置漂移检测 |
| **需要精细自定义每一条 SCP/OU 细节，而非标准化模板** | 直接使用 [[AWS Organizations]]，而非依赖 Control Tower 默认模板 |

---

## 考试重点总结

### SAA-C02 高频考点

1. **核心定位**：构建在 Organizations 之上的自动化编排层，提供开箱即用的多账户治理最佳实践，而非替代 Organizations
2. **着陆区三大核心账户**：管理账户、日志归档账户、审计账户，构成标准化的多账户骨架
3. **预防性护栏 = SCP**：事前阻止违规操作，操作根本无法执行
4. **检测性护栏 = Config 规则**：事后发现配置违规，标记非合规但不阻止
5. **三种控制级别**：强制性（不可关闭）、强烈推荐（OU 级最佳实践）、可选（按需启用）
6. **Account Factory 简化标准化账户创建**：自助门户自动完成创建、OU 分配、护栏应用
7. **AFT 面向 IaC/GitOps 工作流**：账户创建通过 Terraform + Git 流水线程序化管理
8. **合规仪表盘检测配置漂移**：发现账户偏离治理基线后可重新基线化
9. **已有 Organizations 环境可注册到 Control Tower**：无需推倒重来，可逐步引入自动化护栏
10. **需要精细自定义时直接用 Organizations**：Control Tower 提供的是标准化模板，极致灵活性场景仍需底层 Organizations 原语

### 场景题解题思路

```
场景分析 → 判断是否用 Control Tower
├── "首次搭建多账户环境，希望快速获得最佳实践基线" → AWS Control Tower
├── "需要真正阻止某类高危操作（而非仅告警）" → 预防性护栏（SCP）
├── "需要持续检测账户配置是否偏离基线" → 检测性护栏（Config 规则）
├── "业务团队需要自助申请新账户并自动套用合规配置" → Account Factory
├── "账户创建流程需要代码化、可版本控制管理" → Account Factory for Terraform（AFT）
├── "需要发现哪些账户配置已偏离治理基线" → 合规仪表盘 + 配置漂移检测
├── "已有 Organizations 环境，希望补充自动化护栏" → 注册现有账户到 Control Tower
└── "需要精细控制每条 SCP/OU 的具体细节" → 直接使用 AWS Organizations（而非依赖 Control Tower 默认模板）
```

---

## 最佳实践

1. **新建多账户环境优先评估 Control Tower**：相比手动拼装 Organizations + Config + CloudTrail，能更快获得符合最佳实践的基线
2. **日志归档账户和审计账户与业务账户分离**：遵循最小权限和职责分离，避免安全/日志功能与业务负载混用同一账户
3. **优先启用预防性护栏应对高风险操作**：检测性护栏只能事后发现，无法真正阻止已发生的违规操作
4. **IaC/GitOps 团队采用 Account Factory for Terraform**：让账户创建流程与其余基础设施代码保持一致的管理方式
5. **定期检查合规仪表盘的漂移状态**：主动发现并重新基线化偏离治理要求的账户，而非被动等待审计发现
6. **已有 Organizations 环境不必推倒重来**：评估注册到 Control Tower 的迁移路径，逐步获得自动化护栏能力
7. **极致定制化需求下降级到 Organizations 原语**：Control Tower 的标准化模板不满足时，直接操作底层 SCP/OU 获得更大灵活性
8. **关注着陆区版本升级公告**：AWS 持续演进着陆区架构（如资源隔离改进），及时了解版本变化对现有环境的影响
