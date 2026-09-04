# AWS WAF - Web 应用防火墙

> **AWS WAF（Web Application Firewall）** 是应用层（**OSI 第 7 层**）的 Web 防火墙服务，通过可配置的规则检查 HTTP/HTTPS 请求，拦截 **SQL 注入、跨站脚本（XSS）**等常见 Web 攻击，可直接附加到 [[CloudFront]]、[[AWS Load Balance]]、[[Amazon API Gateway]]、AppSync、[[Cognito]] 用户池等面向公网的入口，在流量到达应用/源站之前完成过滤。
>
> 相关文档：[[AWS Shield]] | [[CloudFront]] | [[AWS Load Balance]] | [[Amazon API Gateway]] | [[VPC]] | [[Security Group]] | [[NACL]] | [[CloudWatch]] | [[AWS Firewall Manager]] | [[AWS Organizations]]

---

## 核心概念

### WAF 在网络防护体系中的定位（考试高频）

| 服务 | 防护层级 | 防护类型 | 典型场景 |
|------|---------|---------|---------|
| **AWS WAF** | 第 7 层（应用层） | SQL 注入、XSS、HTTP 请求内容异常、机器人流量 | 检查请求**内容**是否包含攻击特征 |
| **[[AWS Shield]]** | 第 3/4 层（网络/传输层）为主，Advanced 也覆盖第 7 层 | DDoS 洪泛攻击（Standard 免费自动防护，Advanced 付费增强） | 抵御大规模流量淹没型攻击 |
| **[[Security Group]]** | 第 3/4 层，实例级 | 基于 IP/端口的有状态访问控制 | 谁能连接到这台实例的哪个端口 |
| **[[NACL]]** | 第 3/4 层，子网级 | 基于 IP/端口的无状态访问控制 | 子网边界的粗粒度流量过滤 |

> **考试陷阱**：**WAF 检查的是请求的"内容"，Security Group/NACL 检查的是"来源和端口"**——题目描述"需要拦截包含 SQL 注入攻击载荷的 HTTP 请求" → **WAF**（安全组无法理解 HTTP 请求体的内容）；描述"需要限制只有特定 IP 段能访问某端口" → Security Group/NACL；描述"需要抵御大规模流量淹没导致服务不可用" → [[AWS Shield]]（而非 WAF）。三者常组合部署形成纵深防御。

### 为什么需要 WAF

- **传统网络层控制的局限**：Security Group/NACL 只能基于 IP、端口做粗粒度过滤，无法理解 HTTP 请求的语义内容，无法拦截"来自合法 IP、发往合法端口，但请求体包含攻击载荷"的应用层攻击
- **WAF 的核心价值**：在应用层解析 HTTP 请求内容，识别 SQL 注入、XSS 等攻击特征，并支持基于**速率、地理位置、IP 信誉**等维度的精细化流量控制

---

## 核心组件

### Web ACL（Web 访问控制列表）

- WAF 的核心资源，包含一组**规则（Rule）**和**默认动作（Default Action，Allow/Block）**，附加到具体的资源（CloudFront 分配、ALB、API Gateway 等）后生效
- 一个 Web ACL 可同时保护多个资源，规则按顺序评估，也可配置规则组的优先级

### 规则（Rules）与规则组（Rule Groups）

| 类型 | 说明 |
|------|------|
| **自定义规则（Custom Rules）** | 用户自行定义的匹配条件，如字符串匹配、正则匹配、地理位置匹配、IP 集合匹配、请求大小限制等 |
| **速率限制规则（Rate-Based Rule）** | 基于**聚合键（Aggregation Key）**统计单位时间内的请求次数，超过阈值自动拦截，聚合键可组合 IP、请求头、Cookie、查询参数、Label 命名空间等**最多 5 个维度**，实现如"按 IP + 按 URI 路径"的精细限流 |
| **AWS 托管规则组（AWS Managed Rules）** | AWS 预置并持续更新的规则集合，覆盖 OWASP Top 10 常见漏洞、已知恶意 IP、SQL 注入特征库等，开箱即用 |
| **市场规则组（Marketplace Rule Groups）** | 第三方安全厂商在 AWS Marketplace 提供的专业规则集，可直接订阅使用 |

> **考试要点**：**Rate-Based Rule 是应对暴力破解/应用层 DDoS 的标准工具**——题目描述"需要限制单个 IP 在短时间内的请求次数，防止暴力破解登录接口"，答案是配置 **速率限制规则**，而非试图用 Security Group 实现（Security Group 无法感知请求频率）。

### Bot Control（机器人流量控制）

- 托管规则组，提供对常见爬虫、扫描器等机器人流量的**可见性和控制**——可选择拦截/限速侵入性机器人（爬虫、扫描器），同时放行良性机器人（搜索引擎、监控探针）

### 欺诈防护规则组

| 规则组 | 说明 |
|--------|------|
| **账户接管防护（ATP, Account Takeover Prevention）** | 检测登录请求是否匹配已泄露凭证库，识别撞库攻击（Credential Stuffing）和跨多 IP 的分布式登录尝试 |
| **账户创建欺诈防护（ACFP, Account Creation Fraud Prevention）** | 分析注册请求，识别虚假邮箱、一次性域名、批量注册等机器人驱动的账户创建欺诈 |

### CAPTCHA 与 Challenge 动作

- 除了简单的 Allow/Block，WAF 规则的动作还可设置为 **CAPTCHA** 或 **Challenge（无感质询）**，在放行前要求客户端完成人机验证或静默的浏览器环境验证，用于应对介于"明显合法"和"明显恶意"之间的可疑流量

---

## 与 AWS Shield 的协同（考试提示）

- **AWS WAF Anti-DDoS 托管规则组**（2025 年推出）专为应用层（第 7 层）DDoS 防护设计，**[[AWS Shield]] Advanced 已将其采纳为默认的应用层防护机制**，并逐步过渡为唯一的应用层防护方案
- 这意味着 Shield Advanced 的第 7 层防护能力实际上是**构建在 WAF 规则引擎之上**的，进一步说明 WAF 与 Shield 并非完全独立、而是在应用层防护上深度协同，完整的 Shield Standard/Advanced 能力见 [[AWS Shield]] 独立笔记

---

## 部署位置

- **[[CloudFront]]**：在边缘节点拦截攻击流量，请求根本不会到达源站，是全球分发应用的推荐部署点
- **[[AWS Load Balance]]（Application Load Balancer）**：区域级部署，保护未使用 CloudFront 的区域性 Web 应用
- **[[Amazon API Gateway]]**：保护 REST/HTTP API 端点
- **AWS AppSync**：保护 GraphQL API
- **Amazon [[Cognito]] 用户池**：保护登录/注册端点，天然适配 ATP/ACFP 欺诈防护规则组
- **AWS Verified Access / App Runner**：保护零信任访问入口和托管应用运行时

---

## 日志与可观测性

- 支持将请求级日志投递到 **Amazon Kinesis Data Firehose**（进而落地 S3/OpenSearch 等），记录每条请求匹配了哪条规则、最终动作是什么，用于安全分析和规则调优
- 可与 [[CloudWatch]] 集成查看请求量、拦截量等指标，并据此调整规则阈值

---

## 组织级统一管理：AWS Firewall Manager

- 面向多账户/多资源的组织，通过 **[[AWS Firewall Manager]]** 可以**集中定义并强制推行**统一的 WAF 规则策略到组织内所有账户的所有符合条件的资源，无需逐账户逐资源手动配置
- 常与 [[AWS Organizations]] 结合，确保新创建的资源自动纳入统一的 WAF 保护基线，完整的策略类型和自动纳入机制见 [[AWS Firewall Manager]] 独立笔记

---

## 安全性

| 机制 | 说明 |
|------|------|
| **IAM 策略** | 控制哪些身份可以创建/修改 Web ACL 和规则 |
| **规则评估顺序** | 规则按配置顺序依次评估，需注意规则优先级避免误拦截合法流量 |
| **WCU 容量限制** | 每个 Web ACL 的规则复杂度受 **Web ACL 容量单位（WCU）**限制，规则越复杂消耗的 WCU 越多 |

---

## 典型应用场景

| 场景 | 推荐配置 |
|------|---------|
| **防御 SQL 注入/XSS 等常见 Web 攻击** | WAF + AWS 托管规则组 |
| **限制单个 IP 的高频请求，防止暴力破解** | 速率限制规则（Rate-Based Rule） |
| **需要拦截恶意爬虫，同时放行搜索引擎** | Bot Control 托管规则组 |
| **登录接口需要防御撞库/凭证填充攻击** | 账户接管防护（ATP） |
| **注册接口需要防御虚假批量注册** | 账户创建欺诈防护（ACFP） |
| **可疑但不确定是否恶意的流量** | CAPTCHA / Challenge 动作 |
| **全球分发应用需要在边缘层拦截攻击** | CloudFront + WAF |
| **组织内多账户需要统一强制 WAF 策略** | [[AWS Firewall Manager]] + Organizations |
| **需要抵御大规模流量淹没型 DDoS** | 改用/叠加 [[AWS Shield]]（而非仅依赖 WAF） |
| **只需基于 IP/端口做访问控制，无需理解请求内容** | Security Group/NACL 已足够，无需引入 WAF |

---

## 考试重点总结

### SAA-C02 高频考点

1. **核心定位**：应用层（第 7 层）Web 防火墙，检查 HTTP 请求内容，区别于 Security Group/NACL 的网络层访问控制
2. **判断依据**：需要理解请求"内容"（SQL 注入/XSS 特征）选 WAF；只需基于 IP/端口过滤选 Security Group/NACL；抵御流量淹没型 DDoS 选 [[AWS Shield]]
3. **Web ACL 是核心资源**：包含规则集合和默认动作，可同时保护多个资源
4. **Rate-Based Rule 是限流/防暴力破解的标准工具**：支持最多 5 个聚合键组合，实现精细化限流
5. **AWS 托管规则组开箱即用**：覆盖 OWASP Top 10 等常见漏洞特征，无需自行编写全部规则
6. **Bot Control 区分良性/恶意机器人**：拦截爬虫/扫描器，放行搜索引擎等合法机器人
7. **ATP/ACFP 专为登录/注册场景设计**：分别防御撞库攻击和批量虚假注册
8. **CAPTCHA/Challenge 是 Allow/Block 之外的第三类动作**：应对不确定性流量
9. **WAF 可部署在 CloudFront/ALB/API Gateway/AppSync/Cognito 等多个入口**：按架构选择合适的附加点，CloudFront 部署可在边缘拦截、减轻源站压力
10. **[[AWS Firewall Manager]] 实现组织级统一策略推行**：结合 Organizations 确保新资源自动纳入保护基线

### 场景题解题思路

```
场景分析 → 判断是否用 WAF
├── "需要拦截 SQL 注入/XSS 等应用层攻击" → AWS WAF + 托管规则组
├── "需要限制单个 IP 短时间内的请求次数" → 速率限制规则（Rate-Based Rule）
├── "需要区分并拦截恶意爬虫，放行搜索引擎" → Bot Control
├── "登录接口遭遇撞库/凭证填充攻击" → 账户接管防护（ATP）
├── "注册接口出现批量虚假账户创建" → 账户创建欺诈防护（ACFP）
├── "流量可疑但不确定是否恶意" → CAPTCHA/Challenge 动作
├── "全球应用需要在边缘层拦截攻击、减轻源站压力" → CloudFront + WAF
├── "多账户组织需要统一强制 WAF 防护策略" → [[AWS Firewall Manager]] + Organizations
├── "需要抵御大规模流量淹没型 DDoS 攻击" → 改用/叠加 [[AWS Shield]]（而非仅 WAF）
└── "只需基于 IP/端口做访问控制" → Security Group/NACL 已足够，无需引入 WAF
```

---

## 最佳实践

1. **优先使用 AWS 托管规则组覆盖常见攻击特征**：无需从零编写规则应对 OWASP Top 10 等已知漏洞类型
2. **登录/注册端点叠加 ATP/ACFP 规则组**：这类场景的欺诈模式高度专业化，托管规则组比自定义规则更有效
3. **CloudFront 场景优先在边缘层部署 WAF**：在攻击流量到达源站前拦截，同时降低源站负载
4. **速率限制规则按业务场景选择合适的聚合键组合**：避免过于宽泛（误伤正常用户）或过于狭窄（无法有效限流）
5. **不确定流量使用 CAPTCHA/Challenge 而非直接 Block**：减少对合法用户的误伤，同时过滤自动化攻击
6. **多账户组织使用 [[AWS Firewall Manager]] 统一策略**：避免各账户 WAF 配置不一致导致的防护盲区
7. **结合日志持续调优规则**：定期分析 Kinesis Data Firehose 投递的请求日志，识别误报和漏报并调整规则
8. **WAF 与 [[AWS Shield]]、Security Group、NACL 组合形成纵深防御**：不要指望单一服务覆盖所有层级的攻击类型
