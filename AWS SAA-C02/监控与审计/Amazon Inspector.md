# Amazon Inspector - 自动化漏洞管理

> **Amazon Inspector** 是全托管的**自动化漏洞评估**服务，持续扫描 [[EC2]]、容器镜像（ECR）、[[AWS Lambda]] 函数以及应用源代码，识别**已知软件漏洞（CVE）、网络可达性风险、代码安全缺陷**，并按**风险评分**自动排序，帮助团队聚焦真正需要优先修复的问题，而非人工逐一排查。
>
> 相关文档：[[Amazon GuardDuty]] | [[Amazon Macie]] | [[EC2]] | [[AWS Lambda]] | [[AWS Config]] | [[AWS Organizations]] | [[CloudWatch]] | [[IAM]]

---

## 核心概念

### 为什么需要 Inspector

- **传统漏洞扫描的局限**：手动运行扫描工具、逐台服务器登录检查软件版本，在动态伸缩的云环境中几乎不可持续——实例频繁创建销毁，静态的扫描计划无法覆盖不断变化的资源清单
- **Inspector 的核心价值**：**自动发现**账户内符合条件的资源（无需手动指定扫描目标），持续监控其软件清单和配置，一旦出现新的 CVE 或资源发生变更立即重新评估，实现"**持续**"而非"**一次性**"的漏洞管理

### Inspector 在安全监控体系中的定位（考试高频）

| 服务 | 核心问题 | 关注点 | 典型场景 |
|------|---------|--------|---------|
| **Amazon Inspector** | 资源本身**存在哪些已知漏洞/弱点**？ | 软件 CVE、网络可达性、代码缺陷（**评估已知风险**） | 未打补丁的软件包、意外暴露公网的端口、代码中的硬编码密钥 |
| **[[Amazon GuardDuty]]** | 是否**正在发生**攻击/异常行为？ | 跨数据源关联的威胁信号（**检测主动攻击**） | 凭证被盗、恶意软件通信、异常 API 调用 |
| **[[AWS Config]]** | 资源配置**是否符合**预设规则？ | 配置项快照与合规规则比对 | 资源配置漂移、合规基线检查 |

> **考试陷阱**：**Inspector 回答"我有没有已知的弱点"，GuardDuty 回答"有没有人正在利用弱点攻击我"**——题目描述"需要识别 EC2 实例上未打补丁的软件包、已知 CVE" → **Inspector**；题目描述"需要检测账户是否已经出现异常登录或恶意软件通信" → **GuardDuty**；两者是漏洞管理生命周期中**互补的前后两个阶段**（Inspector 侧重预防性评估，GuardDuty 侧重实时检测），常组合部署。

---

## 扫描目标与方式（考试高频）

| 扫描类型 | 目标 | 扫描方式 |
|---------|------|---------|
| **EC2 扫描** | [[EC2]] 实例操作系统及已安装软件包 | **代理式（Agent-based）**通过 SSM Agent 采集软件清单，或**无代理（Agentless）**直接对 EBS 快照做深度分析，无需在实例上安装/运行任何代理 |
| **容器镜像扫描（ECR）** | 推送到 Elastic Container Registry 的容器镜像 | 镜像**推送时自动扫描**，也支持对已存量镜像的持续重扫描（新 CVE 公布后自动触发） |
| **Lambda 扫描** | [[AWS Lambda]] 函数代码及其依赖库 | 分析函数部署包中的第三方依赖库是否存在已知漏洞 |
| **代码安全扫描（Code Security，2025 新特性）** | 应用源代码、基础设施即代码（IaC）模板 | 原生集成 **GitHub、GitLab**，在代码提交/合并请求阶段即完成扫描，将安全左移到开发流程早期 |

> **考试要点**：**代理式 vs 无代理扫描是 EC2 扫描的重要区分点**——无代理扫描（Agentless）**无需在实例上安装/维护任何软件**，直接分析 EBS 快照即可发现漏洞，降低了对生产工作负载性能的干扰；2026 年代理式扫描（VM Scanner）也已扩展支持 WordPress、Apache HTTP Server、Python 包、Ruby Gem 等更广泛的软件生态，与无代理扫描的检测覆盖面基本对齐，企业可根据是否已有 SSM Agent 部署基础灵活选择。

---

## 代码安全（Code Security）三大能力

| 能力 | 说明 |
|------|------|
| **静态应用安全测试（SAST）** | 分析应用源代码本身，识别注入漏洞、不安全的加密用法等代码层面的安全缺陷 |
| **软件成分分析（SCA，Software Composition Analysis）** | 分析代码引用的第三方开源依赖库，识别已知存在漏洞的依赖版本 |
| **基础设施即代码扫描（IaC Scanning）** | 分析 CloudFormation、Terraform 等 IaC 模板，在资源实际部署前发现不安全的配置（如过度宽松的安全组规则），实现部署前拦截 |

- 三项能力通过与 **GitHub/GitLab 原生集成**在开发者的日常工作流（提交、Pull Request）中直接呈现结果，是"**安全左移（Shift Left）**"理念的具体落地——将漏洞发现提前到编码阶段，而非等到资源已部署到生产环境后才通过 EC2/ECR 扫描发现

---

## 风险评分与优先级排序

- Inspector 为每个 Finding 计算综合**风险评分**，融合以下维度而非仅依赖传统的 CVSS 基础评分：
  - **CVSS 评分**：漏洞本身的严重程度
  - **EPSS（Exploit Prediction Scoring System）**：该漏洞**在实际环境中被利用的概率**
  - **网络可达性（Network Reachability）**：存在漏洞的资源是否**可从公网访问**——同一漏洞若存在于隔离子网中的实例上，风险评分显著低于暴露在公网的实例
  - **已知被利用漏洞（Known Exploited Vulnerabilities）**：是否已被 CISA 等机构确认为**真实野外利用**的漏洞
- **Windows KB 聚合发现（2026 新特性）**：将同一 Windows 补丁（KB）关联的多个 CVE **合并为一条 Finding**，展示最高 CVSS/EPSS 评分及相关补丁链接，避免同一补丁产生的大量重复告警

> **考试要点**：**Inspector 的风险评分不是单纯的 CVSS 数字，而是结合实际可利用性和网络暴露面的综合排序**——题目描述"团队被海量漏洞告警淹没，希望优先处理真正高风险的问题" → Inspector 的风险评分机制（结合 EPSS 和网络可达性）正是为解决这类"告警疲劳"设计的核心能力。

---

## 多账户管理

- 支持指定**委派管理员账户**，结合 [[AWS Organizations]] 实现组织级的集中漏洞管理——委派管理员可查看和管理组织内所有成员账户的 Finding
- 新加入组织的账户可配置为**自动启用** Inspector，避免因遗漏配置产生扫描盲区，这一治理模式与 [[Amazon GuardDuty]] 的委派管理员机制一致

---

## 抑制规则与集成

| 集成方式 | 说明 |
|---------|------|
| **抑制规则（Suppression Rules）** | 对已确认的误报或可接受风险（如已有其他补偿性控制措施的漏洞）自动归档 Finding，减少噪音 |
| **[[Amazon EventBridge]]** | Finding 可作为事件源触发自动化响应（如自动触发补丁流程、通知负责团队） |
| **AWS Security Hub** | Finding 可汇总到 Security Hub，与 [[Amazon GuardDuty]]、Shield Network Security Director 等其他安全服务的发现统一呈现在中心化的安全态势看板 |
| **[[AWS Systems Manager]] Patch Manager** | Inspector 发现漏洞后，可结合 Patch Manager 完成实际的补丁修复闭环 |

---

## 典型应用场景

| 场景 | 推荐配置 |
|------|---------|
| **持续监控 EC2 实例的已知 CVE，且不希望安装代理干扰性能** | Inspector 无代理（Agentless）EC2 扫描 |
| **已有 SSM Agent 部署，希望覆盖更广泛的应用软件生态** | Inspector 代理式（VM Scanner）EC2 扫描 |
| **容器镜像推送到 ECR 前需要自动漏洞扫描** | Inspector ECR 镜像扫描 |
| **Lambda 函数的第三方依赖库存在已知漏洞** | Inspector Lambda 扫描 |
| **在代码提交/PR 阶段就发现安全缺陷，而非部署后才发现** | Inspector Code Security（SAST/SCA/IaC）+ GitHub/GitLab 集成 |
| **海量漏洞告警需要按真实风险优先级排序** | Inspector 风险评分（CVSS + EPSS + 网络可达性） |
| **组织级集中管理多账户的漏洞发现** | 委派管理员 + AWS Organizations |
| **发现漏洞后自动触发修复流程** | Inspector + EventBridge + Systems Manager Patch Manager |
| **需要判断资源是否正在遭受攻击，而非评估已知弱点** | 改用/叠加 [[Amazon GuardDuty]]（而非仅依赖 Inspector） |

---

## 考试重点总结

### SAA-C02 高频考点

1. **核心定位**：全托管自动化漏洞评估服务，识别已知 CVE、网络暴露风险、代码安全缺陷
2. **判断依据**：评估"是否存在已知弱点"选 Inspector；检测"是否正在遭受攻击"选 [[Amazon GuardDuty]]，两者互补覆盖漏洞管理生命周期的不同阶段
3. **扫描范围覆盖 EC2、ECR 容器镜像、Lambda 函数、应用源代码**：四类目标各有独立的扫描机制
4. **EC2 扫描分代理式与无代理两种方式**：无代理扫描直接分析 EBS 快照，无需在实例上安装任何软件
5. **Code Security 是"安全左移"的落地**：SAST + SCA + IaC 扫描，原生集成 GitHub/GitLab，在开发阶段而非部署后发现问题
6. **风险评分融合 CVSS + EPSS + 网络可达性**：不是单纯的 CVSS 数字，公网可达的漏洞风险评分显著更高
7. **自动发现资源，无需手动指定扫描目标**：符合条件的新资源自动纳入持续扫描
8. **委派管理员支持组织级集中管理**：新账户可配置自动启用，避免扫描盲区
9. **抑制规则降低告警噪音**：对已知可接受风险的 Finding 自动归档
10. **响应闭环依赖 EventBridge + Patch Manager**：Finding 触发自动化通知/修复流程的标准模式

### 场景题解题思路

```
场景分析 → 判断使用 Inspector 的哪个能力
├── "需要发现 EC2 上未打补丁的软件包/已知 CVE，且不想装代理" → Inspector 无代理 EC2 扫描
├── "需要覆盖 WordPress/Apache/Python/Ruby 等更广泛的软件生态" → Inspector 代理式（VM Scanner）扫描
├── "容器镜像推送到 ECR 前需要自动扫描" → Inspector ECR 镜像扫描
├── "Lambda 函数依赖库存在已知漏洞" → Inspector Lambda 扫描
├── "希望在代码提交/PR 阶段就发现安全缺陷" → Inspector Code Security + GitHub/GitLab
├── "海量漏洞告警，需要按真实风险优先级处理" → Inspector 风险评分（CVSS+EPSS+网络可达性）
├── "组织级集中管理多账户漏洞发现" → 委派管理员 + Organizations
├── "发现漏洞后需要自动触发修复流程" → Inspector + EventBridge + Patch Manager
└── "需要判断是否正在遭受攻击，而非评估已知弱点" → 改用/叠加 Amazon GuardDuty
```

---

## 最佳实践

1. **优先启用无代理 EC2 扫描降低运维负担**：无需部署和维护代理软件即可获得漏洞可见性，性能干扰更小
2. **已有成熟 SSM Agent 部署的环境可叠加代理式扫描**：获得更广泛的应用软件生态覆盖
3. **CI/CD 流程中尽早接入 Code Security**：将 SAST/SCA/IaC 扫描左移到 PR 阶段，修复成本远低于生产环境发现后再处理
4. **优先处理高风险评分（而非仅高 CVSS）的 Finding**：结合 EPSS 和网络可达性，聚焦真正暴露在公网且已知被利用的漏洞
5. **组织级环境使用委派管理员 + 自动启用**：确保新创建账户从一开始就纳入扫描覆盖
6. **善用抑制规则聚焦真正需要处理的问题**：对已有补偿性控制措施的可接受风险及时归档，减少告警疲劳
7. **结合 EventBridge 构建自动化修复管道**：对高风险 Finding 自动触发 Patch Manager 修复流程或通知负责团队
8. **Inspector 与 [[Amazon GuardDuty]] 组合覆盖漏洞管理全生命周期**：Inspector 负责"提前发现弱点"，GuardDuty 负责"实时检测攻击"，两者互补而非替代
