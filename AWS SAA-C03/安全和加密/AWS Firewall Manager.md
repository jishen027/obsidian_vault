# AWS Firewall Manager - 组织级安全策略集中管理

> **AWS Firewall Manager** 是**组织级**的安全策略编排服务，让管理员在一个地方**集中定义**防火墙/安全策略（[[AWS WAF]]、[[AWS Shield]] Advanced、VPC 安全组、AWS Network Firewall、Route 53 Resolver DNS Firewall、第三方防火墙），并**自动强制应用**到 [[AWS Organizations]] 内所有符合条件的账户和资源——包括**新创建的资源**，无需逐账户逐资源手动配置。
>
> 相关文档：[[AWS WAF]] | [[AWS Shield]] | [[AWS Organizations]] | [[VPC]] | [[Security Group]] | [[CloudFront]] | [[AWS Load Balance]]

---

## 核心概念

### 为什么需要 Firewall Manager

- **多账户场景的治理难题**：一个大型组织可能有数十上百个账户，若每个账户各自配置 WAF 规则、安全组、Shield Advanced 保护，极易出现**配置不一致、防护盲区、新账户/新资源遗漏保护**等问题
- **Firewall Manager 的核心价值**：管理员在**管理账户或委派管理员账户**中定义一套策略，Firewall Manager 自动扫描组织内所有账户，将策略部署到匹配范围（按 OU、账户、标签筛选）的资源上，并**持续监控合规状态**，对不合规资源自动修复或告警

### 前置条件

- 必须先启用 [[AWS Organizations]] 的**完整功能模式（All Features）**
- 需要指定一个**关联账户（Firewall Manager Administrator）**——可以是管理账户本身，但**推荐使用委派管理员账户**，遵循最小权限和职责分离原则（与 [[AWS Organizations]] 笔记中"委派管理员"章节的治理理念一致）

---

## 六种策略类型（考试高频）

| 策略类型 | 强制部署到 | 典型用途 |
|---------|-----------|---------|
| **AWS WAF 策略** | [[CloudFront]]、[[AWS Load Balance]]、API Gateway 等 | 组织级统一 Web ACL 规则基线，完整规则类型见 [[AWS WAF]] 独立笔记 |
| **[[AWS Shield]] Advanced 策略** | EC2（弹性 IP）、ALB、CloudFront、Global Accelerator | 确保符合条件的资源自动纳入 Shield Advanced 高级防护 |
| **[[Security Group]] 策略** | VPC 安全组 | 审计并强制统一的安全组规则基线（如禁止 0.0.0.0/0 开放高危端口） |
| **AWS Network Firewall 策略** | VPC | 在 VPC 层面部署无状态/有状态的网络层过滤规则 |
| **Route 53 Resolver DNS Firewall 策略** | VPC 的 DNS 查询 | 拦截对已知恶意域名的 DNS 解析请求，防御 DNS 层面的数据外泄和 C2 通信 |
| **第三方防火墙策略** | VPC | 通过 AWS Marketplace 订阅集成 Palo Alto Networks Cloud NGFW、Fortinet CNFaaS 等第三方防火墙方案 |

> **考试要点**：**一个 Firewall Manager 策略只对应一种策略类型**——若组织需要同时统一管理 WAF 规则和安全组基线，需要创建**多个独立策略**，而非试图用单个策略覆盖所有类型。

---

## 安全组策略的两种审计模式（考试提示）

| 模式 | 说明 |
|------|------|
| **通用安全组策略（Usage Audit）** | 审计组织内**未被使用**或**冗余**的安全组，识别可清理的资源，降低管理复杂度 |
| **内容审计安全组策略（Content Audit）** | 审计安全组规则内容是否符合预设基线（如禁止危险端口对公网开放），可选择**仅告警**或**自动修复**（撤销违规规则） |

---

## 策略应用范围与自动化

### 资源选择维度

- 可按 **组织单元（OU）、具体账户、资源标签** 精确控制策略应用范围，实现分层治理（如仅对生产 OU 强制最严格的 WAF 规则）
- 支持**排除列表**，为特殊场景的账户/资源设置例外

### 新资源自动纳入（核心卖点）

- Firewall Manager **持续监控**组织内新创建的资源（新 VPC、新 ALB、新 CloudFront 分配等），一旦匹配策略范围，**自动应用防护配置**，无需管理员手动介入——这是相较于"一次性批量部署"最关键的差异化价值

### 合规仪表盘

- 提供组织级的**合规状态可视化**，集中查看哪些账户/资源符合策略要求、哪些不合规，并可深入查看具体的违规详情

---

## 与其他服务的关系（考试易混淆点）

| 服务                        | 与 Firewall Manager 的关系                                                                                        |
| ------------------------- | ------------------------------------------------------------------------------------------------------------- |
| **[[AWS Organizations]]** | Firewall Manager 依赖 Organizations 的账户结构和委派管理员机制，是策略强制推行的组织基础                                                  |
| **[[AWS WAF]]**           | Firewall Manager 是 WAF 规则的**组织级分发和强制工具**，WAF 本身的规则语法、Web ACL 概念见 [[AWS WAF]] 独立笔记                             |
| **[[AWS Shield]]**        | Firewall Manager 确保符合条件的资源自动订阅 Shield Advanced 保护，而非替代 Shield Advanced 本身的防护能力                                |
| **[[AWS Config]]**        | Firewall Manager 的合规检测理念与 [[AWS Config]] 的配置合规评估类似，但 Firewall Manager **专注于网络安全策略**的部署与强制，Config 覆盖更广泛的资源配置合规 |

> **考试陷阱**：**Firewall Manager 本身不是一种防护技术，而是"策略编排和强制推行"的管理层**——题目描述"需要在几十个账户间统一部署一致的 WAF/安全组/Shield 策略，并确保新建资源自动纳入" → **Firewall Manager**；题目描述"需要定义具体的 WAF 拦截规则或 Web ACL" → 直接使用 **[[AWS WAF]]**（Firewall Manager 只是分发这些规则的载体，不替代 WAF 本身的规则定义）。

---

## 典型应用场景

| 场景 | 推荐配置 |
|------|---------|
| **多账户组织需要统一强制 WAF 规则基线** | Firewall Manager WAF 策略 + [[AWS Organizations]] |
| **确保新建的 CloudFront/ALB 自动订阅 Shield Advanced** | Firewall Manager Shield Advanced 策略 |
| **审计并清理组织内闲置/冗余安全组** | Firewall Manager 通用安全组策略（Usage Audit） |
| **强制禁止安全组开放高危端口到公网** | Firewall Manager 内容审计安全组策略（Content Audit，自动修复） |
| **在 VPC 层面统一部署网络层过滤规则** | Firewall Manager Network Firewall 策略 |
| **拦截对已知恶意域名的 DNS 解析** | Firewall Manager Route 53 Resolver DNS Firewall 策略 |
| **希望使用 Palo Alto/Fortinet 等第三方防火墙但仍需组织级集中管理** | Firewall Manager 第三方防火墙策略 |
| **仅需为单个账户配置 WAF 规则，无多账户强制需求** | 直接使用 [[AWS WAF]]，无需引入 Firewall Manager |

---

## 考试重点总结

### SAA-C02 高频考点

1. **核心定位**：组织级安全策略集中定义与自动强制推行工具，而非独立的防护技术本身
2. **前置条件**：依赖 [[AWS Organizations]] 完整功能模式 + 指定关联账户（推荐委派管理员）
3. **六种策略类型**：WAF、Shield Advanced、安全组、Network Firewall、Route 53 Resolver DNS Firewall、第三方防火墙
4. **一策略一类型**：需要同时管理多种防护时创建多个独立策略
5. **安全组策略两种模式**：通用审计（清理冗余）vs 内容审计（规则基线合规，可自动修复）
6. **新资源自动纳入是核心卖点**：持续监控组织内新建资源并自动应用匹配策略，无需人工介入
7. **资源范围可按 OU/账户/标签精细控制**：支持排除列表处理特殊例外
8. **合规仪表盘提供组织级可视化**：集中查看策略合规状态，无需逐账户核查
9. **Firewall Manager 不替代具体防护服务的规则定义能力**：具体 WAF 规则语法见 [[AWS WAF]]，Shield Advanced 具体能力见 [[AWS Shield]]
10. **与 Organizations 委派管理员机制配合**：遵循最小权限原则，管理账户不应作为日常操作账户

### 场景题解题思路

```
场景分析 → 判断是否用 Firewall Manager
├── "多账户组织需要统一强制部署 WAF/安全组/Shield 策略" → Firewall Manager
├── "确保新建资源自动纳入既定安全策略，无需人工介入" → Firewall Manager（自动纳入新资源）
├── "需要审计并清理组织内冗余安全组" → Firewall Manager 通用安全组策略
├── "需要强制禁止安全组开放危险端口到公网" → Firewall Manager 内容审计安全组策略
├── "需要在 VPC 层统一部署网络层过滤规则" → Firewall Manager Network Firewall 策略
├── "需要拦截对恶意域名的 DNS 查询" → Firewall Manager Route 53 Resolver DNS Firewall 策略
├── "只需为单一账户配置具体的 WAF 拦截规则" → 直接用 [[AWS WAF]]，无需 Firewall Manager
└── "只需为单一账户订阅 Shield Advanced" → 直接用 [[AWS Shield]] Advanced，无需 Firewall Manager
```

---

## 最佳实践

1. **使用委派管理员账户而非管理账户日常操作 Firewall Manager**：遵循最小权限和职责分离，与 [[AWS Organizations]] 治理理念一致
2. **按 OU 分层设计策略范围**：生产/开发/测试环境应用不同严格程度的策略基线
3. **安全组内容审计策略优先启用自动修复**：对明确违规的高危配置（如开放 22/3389 端口到 0.0.0.0/0）及时自动撤销，缩短暴露窗口
4. **善用标签精细化控制策略范围**：避免"一刀切"策略误伤有特殊需求的资源，同时保留排除列表处理真正的例外
5. **定期查看合规仪表盘**：主动发现不合规账户/资源，而非等安全事件发生后才排查
6. **新建组织架构时尽早规划 Firewall Manager 策略**：确保后续新增账户/资源从一开始就纳入统一防护，避免后期补救成本更高
7. **多种防护需求分别创建独立策略**：不要试图用单一策略覆盖 WAF + 安全组 + Shield 等多种类型
