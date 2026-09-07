# AWS Cost Explorer - 成本资源管理

> **AWS Cost Explorer** 是**免费**的成本可视化与分析工具，让用户通过交互式图表**查看历史支出趋势、按维度拆解成本构成、预测未来花费**，是排查"这个月账单为什么变高了"、"哪个服务/账户/标签花了最多钱"这类问题的第一站。它只负责**呈现和分析**，本身**不提供主动告警能力**——这是它与 **AWS Budgets**（主动阈值告警）最核心的分工差异，也是 SAA-C02 成本管理工具家族中最容易混淆的一组概念。
>
> 相关文档：[[AWS Organizations]] | [[EC2]] | [[AWS Fargate]] | [[CloudWatch]] | [[IAM]] | [[AWS Well-Architected Framework]] | [[AWS Trusted Advisor]]

---

## 核心概念

### 默认报告

- **按服务查看的月度成本**、**按关联账户查看的月度成本**（[[AWS Organizations]] 合并账单场景下，可拆分查看组织内每个成员账户的花费）等开箱即用的默认视图
- 无需任何配置即可立即查看过去的支出情况，适合快速排查异常

### 自定义报告：可按多维度过滤和分组

| 维度 | 说明 |
|------|------|
| **服务（Service）** | 按 EC2、S3、RDS 等具体 AWS 服务拆分成本 |
| **关联账户（Linked Account）** | [[AWS Organizations]] 合并账单下按成员账户拆分 |
| **区域（Region）** | 按资源所在区域拆分 |
| **实例类型/使用类型（Usage Type）** | 更细粒度地看具体资源规格的用量和花费 |
| **购买方式（Purchase Option）** | On-Demand、Reserved、Spot、Savings Plans 各自的花费占比 |
| **成本分配标签（Cost Allocation Tags）** | 按业务自定义标签（如 `Team`、`Project`、`Environment`）拆分，用于成本分摊 |

> **考试要点**：**使用成本分配标签前，必须先在账单控制台中显式激活（Activate）该标签**——题目描述"已经给资源打了标签，但 Cost Explorer 里看不到按标签拆分的数据" → 检查该标签是否已在**成本分配标签**页面激活，而非怀疑标签本身打错。

### 成本预测（Forecasting）

- 基于历史用量模式，使用机器学习**预测未来最长 12 个月**的支出趋势
- **考试要点**：**预测只是"展示趋势"，不会在超出预期时主动通知任何人**——需要"预测到未来会超支就提前告警"这类主动能力时，应使用 **AWS Budgets**（可基于预测值设置告警），而不是误以为 Cost Explorer 的预测功能自带告警。

### 优化建议

| 建议类型 | 说明 |
|---------|------|
| **Savings Plans 购买建议** | 基于历史用量，推荐应承诺的 Savings Plans 额度和类型，预估可节省的费用 |
| **Reserved Instance 购买建议 + 利用率/覆盖率报告** | 推荐应购买的 RI 数量，并展示现有 RI 的**利用率（Utilization，买了多少用了多少）**和**覆盖率（Coverage，用量中有多少被 RI 覆盖）** |
| **EC2 右规格化建议（Rightsizing）** | 识别**过度配置**的 EC2 实例，推荐降配或改用其他实例类型以降低成本 |

> **考试陷阱**：**Cost Explorer 的右规格化建议只聚焦 EC2 实例**——若题目描述"需要跨 EC2、Lambda、EBS、Auto Scaling Group 等多种资源类型的机器学习驱动的右规格化建议" → 应使用**AWS Compute Optimizer**，而非 Cost Explorer；两者都能给出"配置是否过度"的建议，但覆盖的资源范围和分析深度不同。

---

## AWS 成本管理工具家族（考试高频对比）

| 工具 | 核心定位 | 关键特性 |
|------|---------|---------|
| **Cost Explorer** | 历史成本**可视化与分析** | 交互式图表、多维度拆解、预测、优化建议，**免费**，但**不主动告警** |
| **AWS Budgets** | **主动**阈值监控与告警 | 设置成本/用量/RI 及 Savings Plans 利用率的预算阈值，**超出或预测将超出**时通过 [[SNS]] 或邮件告警，还可配置 **Budget Actions** 自动执行响应动作（如附加限制性 IAM 策略、停止实例） |
| **成本与使用报告（Cost and Usage Report, CUR）** | 最细粒度的**原始账单数据导出** | 逐行（Line-Item）级别的完整计费明细，投递到 [[S3]]，供 Athena/QuickSight/Redshift 做深度自定义分析，本身不提供可视化界面 |
| **[[AWS Trusted Advisor]]** | 跨六大类别的**最佳实践检查** | 成本优化只是其中**一个类别**（另有性能、安全、容错、服务配额、卓越运营），侧重"发现闲置/低利用率资源"这类具体检查项，完整能力见 [[AWS Trusted Advisor]] 独立笔记 |
| **AWS Compute Optimizer** | 跨资源类型的**机器学习右规格化建议** | 覆盖 EC2、Auto Scaling Group、EBS、Lambda、ECS on Fargate 等多种资源类型，分析深度和覆盖面均超过 Cost Explorer 的 EC2 右规格化建议 |

> **考试陷阱**：**这是成本管理类题目最高频的失分点**——题目描述"需要在支出超过预算阈值时收到通知，甚至自动采取限制措施" → **AWS Budgets**（而非 Cost Explorer）；题目描述"需要最细粒度的逐行账单数据，导入自己的数据仓库做自定义分析" → **CUR**（而非 Cost Explorer 的聚合视图）；题目描述"需要发现账户中利用率低下/闲置的资源" → **Trusted Advisor**；题目描述"需要跨多种资源类型、机器学习驱动的规格调整建议" → **Compute Optimizer**；题目描述"只是想看看过去的花费趋势、按维度拆解分析" → **Cost Explorer** 本身已经足够。

---

## 与 AWS Organizations 合并账单的关系

- 在 [[AWS Organizations]] 的**合并账单（Consolidated Billing）**模式下，管理账户可以在 Cost Explorer 中**统一查看整个组织**的支出，也可以**按成员账户拆分**分析，完整的合并账单机制见 [[AWS Organizations]] 独立笔记
- Cost Explorer 本身不改变账单的支付主体或折扣共享逻辑，只负责**呈现**这些数据

---

## 典型应用场景

| 场景 | 推荐工具 |
|------|---------|
| **排查某月账单异常升高的原因** | Cost Explorer 按服务/账户/标签拆解分析 |
| **需要在超支或预测将超支时收到告警** | 改用 **AWS Budgets**（而非 Cost Explorer） |
| **需要最细粒度的逐行账单数据做自定义分析** | 改用 **CUR**（而非 Cost Explorer） |
| **需要发现闲置/低利用率资源等最佳实践检查** | 改用 **[[AWS Trusted Advisor]]** |
| **需要跨多资源类型的机器学习右规格化建议** | 改用 **AWS Compute Optimizer** |
| **评估是否应该购买 Savings Plans/Reserved Instance** | Cost Explorer 的购买建议 + 利用率/覆盖率报告 |
| **按团队/项目/环境标签拆分成本用于内部分摊** | Cost Explorer + 已激活的成本分配标签 |
| **多账户组织需要统一查看和拆分整体支出** | Cost Explorer + Organizations 合并账单 |

---

## 考试重点总结

### SAA-C02 高频考点

1. **Cost Explorer 是免费的可视化/分析工具，不提供主动告警**：需要告警应改用 AWS Budgets
2. **默认报告开箱即用，自定义报告支持多维度过滤分组**：服务、账户、区域、使用类型、购买方式、标签
3. **使用成本分配标签前必须先在账单控制台激活**：否则无法按标签拆分数据
4. **预测功能基于机器学习，最长预测 12 个月，但只是展示趋势**：不会主动通知任何人
5. **EC2 右规格化建议范围有限**：跨资源类型的深度建议应使用 Compute Optimizer
6. **Savings Plans/RI 购买建议 + 利用率/覆盖率报告**：评估长期承诺型折扣是否划算的核心依据
7. **CUR 提供最细粒度的原始账单数据，Cost Explorer 提供聚合可视化**：两者是"原始数据"与"呈现分析"的关系，不是竞品
8. **[[AWS Trusted Advisor]] 的成本优化只是六大类别之一**：不要把它等同于专门的成本分析工具
9. **合并账单下 Cost Explorer 可按成员账户拆分整体组织支出**：呈现层面依赖 Organizations 的账单聚合能力
10. **五个工具分工明确，题目描述的动词决定答案**："查看/分析"→ Cost Explorer；"告警/自动响应" → Budgets；"导出原始数据" → CUR；"最佳实践检查" → Trusted Advisor；"机器学习右规格化" → Compute Optimizer

### 场景题解题思路

```
场景分析 → 判断成本管理工具选型
├── "排查账单异常，按维度拆解分析历史支出" → Cost Explorer
├── "支出超过阈值或预测将超支时需要告警/自动响应" → AWS Budgets
├── "需要逐行原始账单数据做自定义深度分析" → Cost and Usage Report (CUR)
├── "需要发现闲置/低利用率资源等最佳实践检查" → [[AWS Trusted Advisor]]
├── "需要跨 EC2/Lambda/EBS/ASG 等多资源类型的机器学习右规格化建议" → AWS Compute Optimizer
├── "评估是否该买 Savings Plans/RI" → Cost Explorer 购买建议 + 利用率/覆盖率报告
└── "按标签拆分成本但看不到数据" → 检查标签是否已在账单控制台激活
```

---

## 最佳实践

1. **提前激活关键的成本分配标签**：避免等到需要按团队/项目拆分成本时才发现数据缺失
2. **需要主动告警时搭配 AWS Budgets，而非只依赖 Cost Explorer 的预测视图**：预测只是参考趋势，不会主动推送通知
3. **定期查看 Savings Plans/RI 利用率和覆盖率报告**：避免购买的承诺型折扣未被充分利用而造成浪费
4. **深度自定义分析场景导出 CUR 到 S3，结合 Athena/QuickSight 处理**：不要试图在 Cost Explorer 界面内完成远超其设计范围的复杂分析
5. **右规格化建议同时参考 Cost Explorer 和 Compute Optimizer**：前者聚焦 EC2，后者覆盖更广的资源类型和更深的机器学习分析
6. **多账户组织善用合并账单 + Cost Explorer 的账户拆分视图**：集中掌握整体支出，同时保留下钻到具体账户的能力
7. **将成本优化检查纳入常规运维流程**：结合 [[AWS Trusted Advisor]] 的闲置资源发现能力，而非只在账单异常后才被动排查