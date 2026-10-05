# AWS Service Catalog - 自助式 IT 服务目录

> **AWS Service Catalog** 是构建在 **[[AWS CloudFormation]]** 之上的**自助式治理层**——管理员将预先审批的 CloudFormation 模板打包成标准化"产品"，最终用户无需拥有 CloudFormation 或底层资源的直接权限，即可通过自助门户**一键部署**符合企业标准的基础设施，实现"自由自助"与"集中管控"的平衡。
>
> 相关文档：[[AWS CloudFormation]] | [[AWS Control Tower]] | [[AWS Organizations]] | [[IAM]]

---

## 核心概念

### 为什么需要 Service Catalog

- **CloudFormation 直接授权的困境**：若让每个开发者都拥有 CloudFormation 和底层 AWS 资源的创建权限，容易导致配置不一致、绕过安全基线、难以统一治理；但若权限收得太紧，又会拖慢团队的自助能力
- **Service Catalog 的核心价值**：**角色分离**——由平台/管理团队预先准备和审批好标准化的 CloudFormation 模板（产品），最终用户只被授予"部署已批准产品"的权限，而非直接操作 CloudFormation 或底层资源的权限，实现**在可控范围内的自助服务**

### Service Catalog 在治理体系中的定位（考试要点）

| 服务 | 定位 | 授权对象 |
|------|------|---------|
| **[[AWS CloudFormation]]** | 底层的基础设施即代码（IaC）引擎 | 直接使用者需要 CloudFormation 及底层资源的完整权限 |
| **AWS Service Catalog** | 构建在 CloudFormation 之上的**自助服务治理层** | 最终用户只需"部署已批准产品"的权限，无需 CloudFormation 权限 |
| **[[AWS Control Tower]]** | 多账户治理自动化 | Control Tower 的 **Account Factory（账户工厂）本质上就是一个 Service Catalog 产品**，用 Service Catalog 的自助部署机制来标准化新账户的创建流程 |

> **考试陷阱**：**Service Catalog 不是 CloudFormation 的替代品，而是它面向"最终用户自助部署"场景的封装**——题目描述"开发团队需要自行部署基础设施，但不希望给予其直接创建/修改 AWS 资源的权限，同时要保证部署内容符合企业标准" → **Service Catalog**；题目描述"平台团队需要自行编写和管理完整的基础设施模板逻辑" → 直接使用 **CloudFormation**。

---

## 核心组件

### 产品（Product）与组合（Portfolio）

| 概念 | 说明 |
|------|------|
| **产品（Product）** | 一份可部署的 IT 服务——底层由一份 **CloudFormation 模板**定义，可以小到一个 EC2 实例，也可以大到一套完整的多层 Web 应用架构 |
| **组合（Portfolio）** | 产品的集合，附带访问权限和配置信息，是**分配权限的基本单位**——管理员将组合共享/授权给特定用户/角色/OU，而非逐个产品单独授权 |

### 共享方式

- **账户间共享**：将组合直接共享给其他 AWS 账户
- **[[AWS Organizations]] 共享**：共享给组织内特定 OU 甚至整个组织，无需逐账户操作
- **结合 [[AWS CloudFormation]] StackSets**：产品部署时可利用 StackSets 能力批量部署到多账户/多区域

---

## 约束（Constraints，考试高频）

| 约束类型 | 说明 |
|---------|------|
| **模板约束（Template Constraint）** | 限制最终用户在部署产品时可配置的参数范围（如只能选择特定的 EC2 实例类型、限定 IP 范围），让同一份通用 CloudFormation 模板可以按产品/组合分别施加不同的限制，无需为每种限制场景复制模板 |
| **启动约束（Launch Constraint）** | 为产品指定一个专用的 **IAM 角色**，实际的资源创建以该角色的权限执行，而非以最终用户自己的身份权限执行——这是实现"用户无需拥有底层资源权限"的关键机制 |
| **标签更新约束（Tag Update Constraint）** | 控制最终用户在更新产品时能否修改资源标签；**默认不允许**更新标签，需显式配置开启 |

> **考试要点**：**启动约束是 Service Catalog 权限模型的核心**——最终用户本身可能完全没有 EC2/RDS 等资源的创建权限，但只要被授权部署对应的 Service Catalog 产品，实际的资源创建会以启动约束指定的角色权限执行，用户"借用"该角色的权限完成部署，自身权限边界不受影响。

---

## TagOptions

- **TagOption** 是由管理员集中定义的键值对模板，用于生成实际应用到已部署资源上的 AWS 标签，强制推行组织的**标签规范（Tagging Taxonomy）**
- 支持**跨账户共享 TagOptions**——在中心账户定义后，共享账户中的变更会自动同步到接收账户，避免各账户标签标准不一致

---

## 服务操作（Service Actions）

- 允许最终用户在产品部署**之后**，对已配置的服务操作（如重启实例、执行运维脚本）执行**预先批准**的操作，而无需拥有该资源的直接管理权限
- 扩展了 Service Catalog 的自助能力边界，覆盖"部署后运维"而不仅是"首次部署"

---

## 容量限制（考试提示）

| 限制项 | 数值 |
|--------|------|
| 每个产品在每个组合下的约束数量 | 最多 100 个 |
| 每个组合的产品数量 | 最多 150 个 |
| 每个区域的组合数量 | 最多 100 个 |
| 每个产品的版本数量 | 最多 100 个 |

---

## 与 AWS Control Tower Account Factory 的关系（考试要点）

- **Account Factory 并非 Control Tower 独有的技术，而是一个包装在 Service Catalog 之上的产品**——标准的账户创建流程本质是：管理员在 Service Catalog 中定义"账户创建"这一产品，最终用户（或委派的团队）通过自助方式"部署"该产品来创建符合规范的新 AWS 账户
- 这解释了为什么 Control Tower 环境中账户创建流程具备**自助申请 + 自动套用治理基线**的能力——底层复用的正是 Service Catalog 的产品/组合/约束模型
- **Account Factory Customization（AFC）**等进阶能力允许进一步自定义账户创建后的初始化逻辑，同样构建在这套 Service Catalog 基础之上

---

## 安全性

| 机制 | 说明 |
|------|------|
| **[[IAM]] 权限分离** | 最终用户只需组合的访问权限，实际资源创建由启动约束指定的角色代为执行 |
| **模板约束限制参数范围** | 防止用户通过参数配置绕过企业标准（如强制使用加密存储、限定实例类型） |
| **标签强制策略** | 结合 TagOptions 确保所有自助部署的资源都符合标签规范，便于成本分摊和资源治理 |

---

## 典型应用场景

| 场景 | 推荐配置 |
|------|---------|
| **开发团队需要自助部署标准化基础设施，但不应拥有底层资源权限** | Service Catalog 产品 + 启动约束 |
| **需要限制用户在部署时可选择的实例类型/网络配置** | 模板约束 |
| **需要为不同团队/账户分组授权不同的产品集合** | 组合（Portfolio）+ 账户/Organizations 共享 |
| **需要强制所有自助部署的资源遵循统一标签规范** | TagOptions（+ 跨账户共享） |
| **需要允许用户在部署后执行预批准的运维操作** | 服务操作（Service Actions） |
| **需要标准化新 AWS 账户的自助创建流程** | Control Tower Account Factory（底层即 Service Catalog 产品） |
| **平台团队需要自行编写复杂的基础设施编排逻辑** | 直接使用 [[AWS CloudFormation]]，而非 Service Catalog |

---

## 考试重点总结

### SAA-C02 高频考点

1. **核心定位**：构建在 CloudFormation 之上的自助服务治理层，让最终用户无需底层资源权限即可自助部署标准化基础设施
2. **产品 = CloudFormation 模板的封装**：一份可部署的 IT 服务单元
3. **组合是授权的基本单位**：按组合而非逐产品分配访问权限
4. **启动约束是权限分离的关键**：实际资源创建以约束指定的角色权限执行，而非用户自身权限
5. **模板约束限制可配置参数范围**：让通用模板可按场景施加不同限制，避免模板重复
6. **标签更新约束默认不允许**：需显式开启才能让用户更新已部署资源的标签
7. **TagOptions 强制标签规范**：支持跨账户共享，变更自动同步
8. **Account Factory 底层是 Service Catalog 产品**：Control Tower 的账户自助创建能力复用 Service Catalog 的产品/约束模型
9. **支持 Organizations 级共享**：组合可共享给 OU 或整个组织，无需逐账户操作
10. **判断依据**：需要"自助但受控"的部署场景选 Service Catalog；需要"直接编写完整 IaC 逻辑"选 CloudFormation

### 场景题解题思路

```
场景分析 → 判断是否用 Service Catalog
├── "开发团队需要自助部署基础设施，但不应拥有底层资源直接权限" → Service Catalog + 启动约束
├── "需要限制用户部署时可选择的参数范围（实例类型/网络等）" → 模板约束
├── "需要为不同团队/账户分配不同的可部署产品集合" → 组合（Portfolio）+ 共享
├── "需要强制统一的资源标签规范" → TagOptions
├── "需要允许部署后执行预批准的运维操作" → 服务操作（Service Actions）
├── "需要标准化新 AWS 账户的创建流程" → Control Tower Account Factory（Service Catalog 产品）
├── "需要跨多账户/区域批量部署同一套模板" → Portfolio 共享 + CloudFormation StackSets
└── "平台团队需要自行编写完整的基础设施编排逻辑，非最终用户自助场景" → 直接使用 CloudFormation（而非 Service Catalog）
```

---

## 最佳实践

1. **通过启动约束而非直接授权实现权限分离**：让最终用户始终无需持有底层资源的创建权限
2. **善用模板约束复用通用模板**：避免为每种限制场景维护多份几乎相同的 CloudFormation 模板
3. **按团队/环境合理划分组合**：让权限分配粒度与组织架构对齐，而非把所有产品塞进单一组合
4. **强制启用 TagOptions 并跨账户共享**：从源头保证自助部署资源的标签一致性，便于成本分摊和资源盘点
5. **默认关闭标签更新约束，按需开启**：避免用户随意修改治理所需的标签体系
6. **多账户组织善用 Organizations 级共享**：避免逐账户手动配置组合访问权限
7. **新增账户创建流程优先复用 Control Tower Account Factory**：而非重新设计一套独立的 Service Catalog 账户产品
8. **仅面向最终用户自助场景使用 Service Catalog**：平台团队内部的复杂编排逻辑仍应直接维护 CloudFormation 模板
