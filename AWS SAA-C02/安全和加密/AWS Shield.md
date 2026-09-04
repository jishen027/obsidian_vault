# AWS Shield - DDoS 防护服务

> **AWS Shield** 是抵御**分布式拒绝服务（DDoS）攻击**的托管防护服务，提供 **Standard**（所有客户默认免费开启）和 **Advanced**（付费订阅，增强防护 + 专家支持）两个层级，保护 EC2、ELB、CloudFront、Global Accelerator、Route 53 等面向公网的资源免受大规模流量淹没型攻击。
>
> 相关文档：[[AWS WAF]] | [[CloudFront]] | [[AWS Load Balance]] | [[Route 53 DNS]] | [[VPC]] | [[Security Group]] | [[AWS Organizations]] | [[AWS Firewall Manager]]

---

## 核心概念

### Shield 在网络防护体系中的定位（考试高频）

| 服务 | 防护层级 | 防护类型 |
|------|---------|---------|
| **AWS Shield** | 第 3/4 层为主（Advanced 也覆盖第 7 层） | DDoS 洪泛攻击（流量/协议淹没导致服务不可用） |
| **[[AWS WAF]]** | 第 7 层（应用层） | SQL 注入、XSS 等基于请求内容的攻击 |
| **[[Security Group]] / [[NACL]]** | 第 3/4 层 | 基于 IP/端口的访问控制 |

> **考试陷阱**：**Shield 解决"流量淹没导致服务不可用"，WAF 解决"请求内容包含攻击载荷"**——题目描述"网站遭遇大规模流量攻击导致无法访问" → **Shield**；描述"网站被注入 SQL 攻击载荷" → **WAF**；两者面向完全不同的攻击形态，常组合部署（详见 [[AWS WAF]] 笔记中两者在应用层的协同关系）。

---

## Shield Standard（标准版）

- **所有 AWS 客户默认自动启用，完全免费**，无需额外配置
- 提供针对**第 3/4 层**常见 DDoS 攻击（如 SYN 洪水、UDP 反射攻击）的自动检测和缓解
- 保护范围覆盖 **CloudFront、Route 53** 等边缘服务的基础设施层面，是所有面向公网架构的默认安全底线

---

## Shield Advanced（高级版）

### 核心增强能力

| 能力 | 说明 |
|------|------|
| **增强检测与缓解** | 针对更复杂、更大规模的 DDoS 攻击提供比 Standard 更精细的检测和自动缓解能力 |
| **实时攻击可视化** | 通过控制台实时查看攻击的详细指标和趋势，而非事后才能了解攻击情况 |
| **AWS DDoS 响应团队（DRT）支持** | **7×24 小时**可联系 AWS 专家团队，在攻击进行中获得**定制化的缓解策略**协助 |
| **费用保护（Cost Protection）** | 若资源因遭受 DDoS 攻击而**被迫弹性扩容**（如 Auto Scaling 触发扩容应对攻击流量），Shield Advanced 承担由此产生的**额外扩容费用**，避免账单意外飙升 |
| **应用层（第 7 层）防护** | 2025 年起默认采用 **AWS WAF Anti-DDoS 托管规则组**作为应用层防护机制，即 Shield Advanced 的 L7 能力实际构建在 WAF 规则引擎之上（详见 [[AWS WAF]] 笔记） |

### 受保护资源

- **[[EC2]]**（需关联弹性 IP）、**[[AWS Load Balance]]**、**[[CloudFront]]**、**AWS Global Accelerator**、**[[Route 53 DNS]]**
- 需要显式将资源加入 Shield Advanced 的**受保护资源组（Protected Resource Group）**才能获得高级防护和 DRT 支持

### 订阅与计费

- 按**组织**订阅（而非按资源单独付费），订阅后组织内所有符合条件的资源均可享受高级防护
- 月度订阅费叠加数据传输费用，属于**面向大型企业/高风险目标**的安全投入，中小型应用通常 Standard 已能满足基础防护需求

> **考试要点**：**费用保护是 Shield Advanced 区别于单纯"更强防护"的独特价值**——题目描述"担心 DDoS 攻击导致 Auto Scaling 疯狂扩容，产生巨额意外账单" → **Shield Advanced 的费用保护**是精准识别点，Standard 版本不提供这项financial safeguard。

---

## AWS Shield Network Security Director（新特性，预览阶段）

- 2025 年推出的**网络安全态势评估**能力——自动发现账户内的计算、网络、网络安全资源，分析网络拓扑，识别**缺失或配置不当的 [[AWS WAF]]、[[Security Group]]、[[NACL]]** 等网络安全服务，并给出修复建议
- 支持**委派管理员账户**发起跨多账户/多 OU 的**持续网络安全分析**，集中查看组织内每个账户的网络拓扑、安全发现项和修复建议（2025 年底新增多账户分析能力）
- 发现结果可集成到 **AWS Security Hub**（2026 年新增），统一汇总到企业整体的安全态势看板中
- **考试提示**：该能力仍处于预览阶段且持续演进，核心定位是"**主动发现网络安全配置缺陷**"，与 Shield Standard/Advanced 的"**被动防御 DDoS 攻击**"是互补而非替代关系

---

## 与其他服务的协同

| 服务 | 协同关系 |
|------|---------|
| **[[AWS WAF]]** | Shield Advanced 的应用层防护构建在 WAF 规则引擎之上，两者结合覆盖第 3/4/7 层的完整防护 |
| **[[AWS Firewall Manager]]** | 可结合 [[AWS Organizations]] 在组织级统一部署 Shield Advanced 保护策略，确保新资源自动纳入防护，完整的策略类型和自动纳入机制见 [[AWS Firewall Manager]] 独立笔记 |
| **[[Route 53 DNS]] 健康检查** | Shield Advanced 可结合健康检查数据更精准地判断资源是否因攻击而不可用，辅助攻击检测的准确性 |

---

## 典型应用场景

| 场景 | 推荐配置 |
|------|---------|
| **所有面向公网的基础架构的默认防护底线** | Shield Standard（自动启用，免费） |
| **高价值/高风险目标需要专家级 DDoS 响应支持** | Shield Advanced + DRT |
| **担心攻击导致 Auto Scaling 产生意外扩容费用** | Shield Advanced 费用保护 |
| **需要实时查看攻击详情和趋势** | Shield Advanced 实时攻击可视化 |
| **需要发现组织内网络安全配置缺陷（WAF/SG/NACL 配置不当）** | Shield Network Security Director |
| **组织级统一部署 DDoS 防护策略** | Shield Advanced + [[AWS Firewall Manager]] + Organizations |
| **需要拦截请求内容中的攻击载荷（SQL 注入/XSS）** | 改用/叠加 [[AWS WAF]]（而非仅依赖 Shield） |

---

## 考试重点总结

### SAA-C02 高频考点

1. **核心定位**：抵御 DDoS 攻击的托管服务，Standard 免费自动开启，Advanced 付费增强
2. **判断依据**：流量淹没导致服务不可用选 Shield；请求内容含攻击载荷选 WAF
3. **Shield Standard 覆盖第 3/4 层**：SYN 洪水等常见攻击，所有客户默认受保护，无需额外配置
4. **Shield Advanced 增强能力四件套**：增强检测缓解、实时可视化、DRT 支持、费用保护
5. **费用保护是 Advanced 的独特价值**：承担因 DDoS 导致的弹性扩容额外费用，Standard 不提供
6. **Advanced 的 L7 防护构建在 WAF 之上**：2025 年起默认采用 WAF Anti-DDoS 托管规则组
7. **受保护资源覆盖 EC2/ELB/CloudFront/Global Accelerator/Route 53**：需显式加入受保护资源组才享受高级防护
8. **按组织订阅而非按资源计费**：订阅后组织内符合条件的资源均可享受防护
9. **Network Security Director 是主动的安全态势发现工具**：识别 WAF/SG/NACL 配置缺陷，与 Shield 的被动防御互补，支持多账户分析和 Security Hub 集成
10. **Shield + WAF + [[AWS Firewall Manager]] 组合覆盖完整防护体系**：分别对应流量层防护、内容层防护、组织级统一策略管理

### 场景题解题思路

```
场景分析 → 判断使用 Shield 的哪个层级/能力
├── "面向公网架构的默认 DDoS 防护，无预算投入" → Shield Standard（免费自动开启）
├── "高价值目标需要专家实时协助应对复杂 DDoS 攻击" → Shield Advanced + DRT
├── "担心攻击导致自动扩容产生意外费用" → Shield Advanced 费用保护
├── "需要实时查看攻击指标和趋势" → Shield Advanced 实时可视化
├── "需要发现网络中缺失/配置不当的 WAF、安全组、NACL" → Shield Network Security Director
├── "组织级统一部署和强制 DDoS 防护策略" → Shield Advanced + [[AWS Firewall Manager]] + Organizations
├── "需要拦截 SQL 注入/XSS 等应用层攻击内容" → 改用/叠加 AWS WAF（而非仅 Shield）
└── "只需基于 IP/端口做访问控制" → Security Group/NACL（而非 Shield）
```

---

## 最佳实践

1. **所有架构默认依赖 Shield Standard 作为基础防护**：无需额外配置即可获得第 3/4 层的自动防护
2. **高风险/高价值业务评估订阅 Shield Advanced**：尤其是无法承受服务中断或存在明显攻击目标特征的业务
3. **提前将关键资源加入受保护资源组**：避免攻击发生时才发现资源未被 Advanced 覆盖
4. **善用费用保护降低 DDoS 攻击的财务风险**：结合 Auto Scaling 的场景尤其应评估这项能力
5. **Advanced 订阅后结合 WAF 构建应用层防护**：单靠 Shield Advanced 不能替代 WAF 对请求内容的检查能力
6. **组织级部署优先采用 [[AWS Firewall Manager]] 统一策略**：避免各账户防护配置不一致
7. **定期运行 Network Security Director 评估网络安全态势**：主动发现配置缺陷，而非等待攻击发生才暴露问题
8. **重大活动/预期流量高峰前主动联系 DRT（Advanced 客户）**：提前规划缓解策略，而非等攻击发生后被动响应
