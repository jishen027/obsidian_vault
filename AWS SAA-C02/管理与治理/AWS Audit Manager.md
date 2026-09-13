# AWS Audit Manager - 持续审计证据收集

> **AWS Audit Manager** 持续、自动化地从 AWS 账户中收集**配置快照、API 调用日志、其他安全服务的检查结果**等证据，并将这些证据**映射到具体的合规框架控制项**（如 PCI DSS、GDPR、HIPAA、SOC 2 或自定义框架），最终一键生成可交给审计方的**评估报告**。它解决的问题是"**如何证明我自己的系统持续满足某个合规框架**"，把原本需要人工东拼西凑截图、日志、配置记录的繁琐审计准备工作自动化——这与下载 **AWS 自身**合规证书的 [[AWS Artifact]]、评估**资源配置**是否合规的 [[AWS Config]] 是完全不同的三件事，是 AIF-C01/SAA-C02 都高频出现的三方对比考点。**⚠️ 服务状态提醒：AWS Audit Manager 已于 2026 年 4 月 30 日起进入维护模式，停止向新账户/新区域/新组织开放**，AWS 官方将新的合规管理需求引导至 [[AWS Config]] 的 **Conformance Packs**，完整背景见下方 [[#服务状态变更（重要，2026-04-30 起进入维护模式）]]。
>
> 相关文档：[[AWS Artifact]] | [[AWS Config]] | [[CloudTrail]] | [[Amazon GuardDuty]] | [[IAM]] | [[AWS Organizations]] | [[负责任的AI与安全]] | [[AIF-C01 考试概览]]

---

## 服务状态变更（重要，2026-04-30 起进入维护模式）

> **AWS Audit Manager 正在转入维护模式（Maintenance Mode）**——AWS 官方给出的原因是"通过投资 AWS Config 的合规管理能力，为客户提供更好、更集成的体验"。这是 2026 年下半年 SAA-C01/AIF-C01 场景题中**判断"该不该用 Audit Manager 做新方案"的关键前提**，考试倾向于把 Audit Manager 放进"**存量维护、不建议新建**"的服务分类，与 Kendra、Bedrock Agents Classic 等已进入维护模式的服务性质相同。

### 具体影响范围

| 客户类型 | 4/30/2026 起还能做什么 | 不能做什么 |
|---------|----------------------|-----------|
| **单账户部署（已在某区域启用）** | 继续在该账户/该区域正常使用，包括**创建新的 Assessment** | 不能把 Audit Manager 扩展到**新区域**或**新账户** |
| **组织级部署（在管理账户启用）** | 可以继续把该组织内的账户**添加到已有 Assessment** | 不能扩展到**新区域**，也不能对**新组织**启用 |
| **全新客户（此前从未启用过）** | — | **完全无法**在任何账户/区域新建 Audit Manager |

- **维护模式期间 AWS 仍会做什么**：修复代码缺陷（Bug Fix）、维护现有已支持框架的数据源映射关系
- **维护模式期间 AWS 不会做什么**：**不新增功能、不支持新框架或框架新版本、不新增区域支持**
- **服务不会被关停**：这不是弃用（Deprecation）/下线（Shutdown），存量客户可无限期继续使用现有部署，只是不能"横向扩张"

### AWS 官方推荐的替代路径：AWS Config Conformance Packs

> AWS 明确建议新的合规管理需求转向 [[AWS Config]] 的 **Conformance Packs**（合规包）——将一组 Config 规则打包，可部署到单账户或通过 [[AWS Organizations]] 部署到整个组织，提供合规仪表板和（部分场景下的）自动修复能力。

| 对比维度 | AWS Audit Manager | AWS Config Conformance Packs |
|---------|-------------------|-------------------------------|
| **预置框架数量** | 35 个（含 SOC 2、PCI DSS、HIPAA 等） | 100+ 个预置模板（**但 SOC 2、GDPR 等部分框架目前无对应模板**） |
| **证据来源** | 服务 API 调用 + Security Hub 检查项 + CloudTrail 事件 + Config 规则，共 4 类数据源 | 仅 Config 规则评估产生的**配置项（Configuration Item）**，来源单一但更易查阅 |
| **不合规资源的修复** | **不支持** | **支持**——可定义并触发修复方案（这是 Audit Manager 从未具备的能力） |
| **审计报告导出** | 支持一键导出 PDF 格式的评估报告 | **无直接对应功能**，需通过 Config Advanced Query / `get-resource-config-history` API 等方式自行导出证据 |
| **自定义** | 自定义控制项 + 自定义框架 | 自定义 Config 规则 + 自定义合规包模板 |

> **考试陷阱**：**Conformance Packs 不是 Audit Manager 的"一对一直接替代品"**——题目若描述"团队正在为 SOC 2 审计做准备，且需要完整的框架控制项覆盖和一键导出报告" → 目前 Config Conformance Packs **尚无 SOC 2 模板**，仍需权衡（存量 Audit Manager 客户可继续用，新客户需考虑第三方方案或等待 Config 补齐模板）；题目描述"需要在检测到不合规资源后自动触发修复" → **这是 Conformance Packs 独有、Audit Manager 从不支持的能力**，即使 Audit Manager 未进入维护模式也应选 Config。

### 已启用 Audit Manager 的客户：停用与证据保留

- 可随时通过控制台设置页或 CLI **停用**（Disable）Audit Manager——停用后**停止收集新证据**，但**已收集的证据默认保留 2 年**，之后可随时重新启用以访问历史证据、恢复收集
- 组织级部署需从**组织管理账户**执行停用操作，委派管理员账户默认没有停用权限

---

## 核心概念

### 为什么需要 Audit Manager

- 企业需要定期向内部/外部审计方证明自己的系统**持续满足**某个合规框架（如 PCI DSS、HIPAA），传统做法是审计前人工翻找配置截图、日志、审批记录，**耗时且容易遗漏**
- **Audit Manager 的核心价值**：预先定义好框架和控制项后，**日常持续自动收集证据**，审计季到来时无需临时突击整理，直接导出报告即可

### AWS Audit Manager 在合规工具链中的定位（考试要点）

| 服务 | 核心问题 | 评估对象 | 产出 |
|------|---------|---------|------|
| **AWS Audit Manager** | 我自己的系统如何**持续证明**满足某个合规框架？ | 我方账户的配置、API 活动、其他安全服务检查结果 | 映射到框架控制项的**自动化证据** + 可导出的评估报告 |
| **[[AWS Artifact]]** | AWS 自身的合规认证文件在哪下载？ | AWS 底层基础设施 | 静态 PDF 报告/协议（AWS 已经做好的） |
| **[[AWS Config]]** | 我自己的**资源配置**是否合规？ | 单个资源的配置状态 | 配置合规检测结果（Rule + Remediation） |

> **考试陷阱**：**判断关键是"我是要证据，还是要证书，还是要检测配置"**——题目问"需要持续、自动化地收集证据以简化下一次审计的准备工作" → **AWS Audit Manager**；问"需要下载 AWS 自己的 SOC 2 报告" → **[[AWS Artifact]]**；问"需要检测某个 S3 桶是否加密" → **[[AWS Config]]**。三者常组合出现：**Config 的检测结果本身就是 Audit Manager 可以自动拉取的证据来源之一**，Artifact 的报告则是审计材料中"AWS 那一半"的补充证明。

---

## 核心组件

### 框架（Framework）

- **框架**定义了一整套要满足的合规要求，由若干**控制集（Control Set）**组成，每个控制集下包含若干**控制项（Control）**
- **预置标准框架**：AWS 提供数十种开箱即用的框架，覆盖 **PCI DSS**、**GDPR**、**HIPAA**、**SOC 2**、**CIS AWS Foundations Benchmark**、**NIST 800-53** 等主流标准
- **自定义框架（Custom Framework）**：可基于预置框架裁剪，或完全自定义控制项，适配组织内部特有的合规要求

### 控制项（Control）与数据源映射

- 每个**控制项**对应一条具体的合规要求（如"确保所有存储静态数据的资源已加密"），并预先**映射到自动化数据源**：
  - **[[AWS Config]]** 规则的合规检查结果
  - **[[CloudTrail]]** 中的相关 API 调用事件
  - **AWS Security Hub** 的安全发现（Findings）
  - 其他 AWS 服务的配置元数据
- 无法自动化覆盖的控制项，可**手动上传证据**（如线下审批流程的截图、第三方文档），实现自动化证据和人工证据并存

### 评估（Assessment）

- 基于一个框架**创建评估（Assessment）**，指定评估范围（账户/区域），系统随即开始**持续、自动**采集与该框架控制项相关的证据
- 评估运行期间产生的所有证据按控制项组织，形成**证据文件夹（Evidence Folder）**，方便按控制项逐一审阅

### 评估报告（Assessment Report）

- 审计季到来时，一键将评估中积累的证据**导出为评估报告**（PDF/其他格式），直接交给内部或外部审计方，**无需再临时收集和整理**
- 报告中的证据自带时间戳和来源，具备可追溯性，比人工整理的截图更具说服力

---

## 委派与协作（考试提示）

- 可将特定的**控制项委派（Delegate）**给团队中的其他人员（如各业务线负责人），由其对分配到的证据进行**人工审阅、添加评论、标记合规状态**
- 这种委派机制让"审计证据的技术性收集"与"业务责任人的人工判断"分离，符合大型组织中职责分离（Segregation of Duties）的治理要求

---

## 与其他安全/治理服务的协同

| 集成方式 | 说明 |
|---------|------|
| **[[AWS Config]]** | Config 规则的合规检查结果是 Audit Manager 控制项最主要的自动化证据来源之一 |
| **[[CloudTrail]]** | 提供"谁在什么时候做了什么"的操作日志，作为审计追溯证据 |
| **AWS Security Hub** | 安全发现（如未加密资源、公开访问的存储桶）可直接作为证据映射到相关控制项 |
| **[[AWS Organizations]]** | 支持委派管理员账户，实现组织级多账户的集中评估和证据收集 |

---

## 典型应用场景

| 场景 | 推荐配置 |
|------|---------|
| **需要持续、自动化地为 PCI DSS/HIPAA/SOC 2 等框架收集合规证据** | 基于对应预置框架创建 Assessment |
| **组织有特有的内部合规要求** | 自定义框架（Custom Framework） |
| **审计季需要快速导出可交付审计方的报告** | Assessment Report 一键导出 |
| **部分控制项无法自动化验证，需要人工审核** | 手动上传证据 + 控制项委派给业务责任人 |
| **需要下载 AWS 自身的合规证书** | 改用 [[AWS Artifact]]（而非 Audit Manager） |
| **需要检测/修复单个资源的配置是否合规** | 改用 [[AWS Config]]（而非 Audit Manager） |
| **组织级多账户统一开展审计证据收集** | [[AWS Organizations]] 委派管理员 + 组织级 Assessment |
| **全新客户/全新账户想搭建合规管理方案（2026-04-30 后）** | **无法新建 Audit Manager**，改用 [[AWS Config]] 的 Conformance Packs |
| **存量 Audit Manager 客户想扩展到新区域/新账户** | **不支持**——维护模式下只能在已启用的账户/区域内继续使用 |
| **需要在检测到不合规资源后自动触发修复** | Audit Manager 从不支持修复，改用 [[AWS Config]] Conformance Packs + 修复方案 |

---

## 考试重点总结

### 高频考点（SAA-C02 / AIF-C01 通用）

1. **核心定位**：持续自动收集**我方系统**的合规证据，映射到框架控制项，简化审计准备工作，**不提供 AWS 自身的证书，也不直接修复资源配置**
2. **框架（Framework）→ 控制集（Control Set）→ 控制项（Control）**：三级结构，预置框架覆盖 PCI DSS/GDPR/HIPAA/SOC 2/CIS/NIST 800-53 等主流标准，也支持自定义框架
3. **控制项映射自动化数据源**：Config 规则结果、CloudTrail 事件、Security Hub 发现，无法自动化的部分支持手动上传证据
4. **评估（Assessment）持续运行**：创建后自动、持续采集证据，而非审计前临时一次性抓取
5. **评估报告一键导出**：证据自带时间戳和来源，可直接交付审计方
6. **控制项可委派给业务责任人人工审阅**：技术证据收集与业务判断分离，符合职责分离治理要求
7. **三方对比**：**Audit Manager**（我方证据）vs **[[AWS Artifact]]**（AWS 自身证书）vs **[[AWS Config]]**（我方资源配置检测）——判断依据是"评估对象是谁、产出是证据/证书/检测结果"
8. **服务状态（重要，2026 下半年高频考点）**：Audit Manager 已于 **2026-04-30** 起进入**维护模式**，停止向新账户/新区域/新组织开放；存量客户在已启用范围内可正常使用（含创建新 Assessment），但无法横向扩张；AWS 推荐新需求转向 **[[AWS Config]] Conformance Packs**
9. **Conformance Packs 不是 Audit Manager 的完全等价替代**：目前 **SOC 2、GDPR 等框架尚无对应 Conformance Pack 模板**，且 Config 无一键审计报告导出功能；但 Conformance Packs **独有自动修复能力**，是 Audit Manager 从未支持的

### 场景题解题思路

```
场景分析 → 判断是否使用 AWS Audit Manager
├── "需要持续、自动化收集合规证据，减轻审计准备的人工工作量" → AWS Audit Manager（存量客户）/ 评估 AWS Config Conformance Packs（新客户）
├── "需要针对 PCI DSS/HIPAA/SOC 2 等标准框架收集证据" → 基于预置 Framework 创建 Assessment（存量客户）
├── "组织有特有的内部合规要求" → 自定义框架
├── "审计季需要快速导出可交付的报告" → Assessment Report（Config Conformance Packs 无直接对应功能）
├── "部分合规要求无法自动化验证" → 手动上传证据 + 委派业务责任人审阅
├── "需要下载 AWS 自身的合规证书" → 改用 AWS Artifact
├── "需要检测/修复单个资源的配置，或需要自动修复不合规资源" → 改用 AWS Config（Conformance Packs 支持修复，Audit Manager 不支持）
├── "组织级多账户统一开展审计" → AWS Organizations 委派管理员 + 组织级 Assessment（存量客户）
└── "全新客户/账户，2026-04-30 之后想新建审计证据收集方案" → **无法新建 Audit Manager**，改用 AWS Config Conformance Packs
```

---

## 最佳实践

1. **优先复用预置框架而非从零自定义**：PCI DSS/HIPAA/SOC 2 等主流标准已开箱即用，减少框架搭建成本
2. **尽早创建 Assessment 而非临近审计才启动**：证据是持续累积的，越早开始覆盖的时间窗口越完整
3. **善用控制项委派实现职责分离**：技术团队负责自动化证据的准确性，业务责任人负责人工判断和签署
4. **结合 [[AWS Config]] 提前修复已知不合规配置**：Audit Manager 只负责收集证据呈现现状，实际整改仍需 Config + 自动修复机制
5. **组织级环境使用委派管理员统一管理评估**：避免各账户各自为政导致证据覆盖不完整
6. **审计报告导出前人工复核关键控制项**：尤其是依赖手动证据的控制项，确保材料完整、时效性达标
7. **新项目/新账户优先评估 [[AWS Config]] Conformance Packs 而非新建 Audit Manager**：2026-04-30 起已无法新建，且 Conformance Packs 提供 Audit Manager 不具备的自动修复能力
8. **存量客户停用前先启用 Evidence Finder**：Evidence Finder 会把证据导出到 CloudTrail 数据湖，停用服务后仍可持续访问历史证据，而不仅依赖"停用后 2 年保留期"
9. **框架选型前核对 Conformance Pack 覆盖情况**：SOC 2、GDPR 等框架目前无对应模板，评估迁移前务必确认目标框架是否已被 Config 覆盖

---

## AIF-C01 考试视角（简化版）

> AIF-C01 Domain 5（安全、合规、治理）中，AWS Audit Manager 通常作为 [[负责任的AI与安全]] 笔记里 [[负责任的AI与安全#AI 合规性 (Compliance for AI)|AI 合规性]] 章节的一个选项出现，考查深度远低于对 Bedrock/SageMaker 等 AI 服务本身的要求——**只需记住"Audit Manager = 持续自动收集我方系统的合规证据，2026-04-30 起进入维护模式、新账户改用 AWS Config Conformance Packs"这两句话**，并能与 Artifact、Config 三方区分开。

| 记忆锚点 | 内容 |
|---------|------|
| **一句话定位** | AWS Audit Manager = 持续自动收集**我方系统**的合规证据，简化审计准备 |
| **最容易混淆的对象** | AWS Artifact（下载 **AWS 自身**的证书，而非收集我方证据）、AWS Config（检测**单个资源配置**，而非跨账户汇总证据生成报告） |
| **典型 AI 场景** | 团队把 Bedrock/SageMaker 训练和推理管道纳入 SOC 2 审计范围，需要持续证明这些 AI 工作负载的访问控制、加密、日志留存等符合控制项要求 → 基于 SOC 2 框架创建 **Audit Manager** Assessment，自动收集相关证据 |

> **考试要点**：题目描述"需要持续、自动化地为 AI 系统收集合规审计证据，减少人工整理工作量" → **AWS Audit Manager**；描述"需要下载 AWS 基础设施本身的合规证明" → **[[AWS Artifact]]**；描述"需要检测 AI 训练数据所在的 S3 桶配置是否合规" → **[[AWS Config]]**——这三者是 [[负责任的AI与安全]] 中 AI 合规性章节的高频三方对比题。
