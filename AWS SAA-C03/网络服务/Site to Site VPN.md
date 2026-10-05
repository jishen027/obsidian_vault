# Site-to-Site VPN - 站点到站点 VPN

> **AWS Site-to-Site VPN** 在 [[VPC]]（通过 [[Virtual Private Gateway]] 或 [[Transit Gateway]]）与本地数据中心（通过**[[Customer Gateway|客户网关 Customer Gateway, CGW]]**）之间，经**公共互联网**建立**加密的 IPsec 隧道**——相比需要数周部署周期的 [[Direct Connect]] 专线，VPN 可在**几分钟到几小时内**完成配置并投入使用，是"快速、加密、经济"的混合云连接方案。
>
> 相关文档：[[Virtual Private Gateway]] | [[Customer Gateway]] | [[Transit Gateway]] | [[Direct Connect]] | [[VPC]] | [[Route Table]] | [[VPC Peering]] | [[CIDR]] | [[AWS Outposts]]

---

## 核心概念

### 为什么需要 Site-to-Site VPN

- **[[Direct Connect]] 部署周期长**：申请物理专线、机房对接通常需要**数周甚至数月**，无法满足紧急或临时的混合云连接需求
- **VPN 的核心价值**：复用**已有的公共互联网**链路，通过 IPsec 加密隧道**快速**建立起加密的私有通信通道，是验证混合云架构、灾备场景、或作为 Direct Connect 上线前**过渡方案**的常见选择

### 核心组件

| 组件 | 角色 | 说明 |
|------|------|------|
| **[[Virtual Private Gateway]]（VGW）** 或 **[[Transit Gateway]]** | AWS 侧的 VPN 端点 | 附加到 VPC（或作为 Transit Gateway 的 VPN 附件），负责隧道的加解密处理 |
| **[[Customer Gateway\|客户网关（Customer Gateway, CGW）]]** | 本地侧的 VPN 端点 | 本地数据中心的物理设备或软件（防火墙、路由器等），需要一个**静态公网 IP**，AWS 侧的 CGW 资源与本地物理设备的区分详见 [[Customer Gateway]] 独立笔记 |
| **VPN 连接（VPN Connection）** | 逻辑连接 | 连接 VGW/Transit Gateway 与 CGW，底层由**两条独立隧道**组成 |

---

## 隧道与高可用（考试高频）

- **每个 VPN 连接默认包含 2 条独立的 IPsec 隧道**，分别终结在 AWS 侧**不同的物理端点**上，实现内置的高可用——单条隧道故障时可自动切换到另一条
- **考试要点**：**客户侧设备应同时配置两条隧道**（而非只接入其中一条），才能真正享受到 AWS 提供的高可用能力；若本地设备只配置了单隧道，一旦该隧道所在的 AWS 端点维护或故障，整个连接会中断
- 每条隧道的**带宽上限约为 1.25 Gbps**——若需要更高聚合带宽，需结合下文的 **ECMP（等价多路径）**方案或改用 [[Direct Connect]]

---

## 路由方式：静态 vs 动态（BGP）

| 类型 | 说明 | 适用场景 |
|------|------|---------|
| **静态路由（Static Routing）** | 手动指定需要通过隧道路由的 CIDR 范围 | 本地设备**不支持 BGP** 时的简化方案，配置简单但拓扑变化需手动更新 |
| **动态路由（Dynamic Routing, BGP）** | [[Customer Gateway\|CGW]] 与 AWS 侧通过 **BGP** 协议自动交换路由信息 | **AWS 推荐**方式，本地网络拓扑变化时路由**自动**同步，无需手动维护，也是使用 Transit Gateway 路由传播、ECMP 等高级特性的**前提条件** |

> **考试陷阱**：**"本地网络拓扑经常变化，静态路由维护困难/容易遗漏"是识别"应使用动态路由（BGP）"的标准题干**——这与 [[Route Table]] 笔记中"路由传播（Route Propagation）"的适用场景完全一致：启用路由传播 + 动态 BGP VPN，让本地路由变化自动同步到 AWS 路由表。

---

## 与 Transit Gateway 结合：ECMP 与聚合带宽

- 通过 **[[Transit Gateway]]** 作为 VPN 的 AWS 侧附件（而非直接挂载单个 VGW），可以创建**多条 VPN 连接**并启用 **ECMP（Equal-Cost Multi-Path，等价多路径）**，将多条隧道的流量**负载均衡**分摊，从而获得**超过单隧道 1.25 Gbps** 的聚合带宽，完整的附件类型和路由表机制见 [[Transit Gateway]] 独立笔记
- **考试要点**：题目描述"单条 VPN 隧道带宽不足，需要提升 VPN 连接的整体吞吐" → 结合 **Transit Gateway + 多条 VPN 连接 + ECMP**，而非单纯期待某条隧道自动扩容（VPN 隧道本身没有类似 [[NAT Gateway]] 的自动带宽扩展能力）

### 加速版 Site-to-Site VPN（Accelerated VPN）

- 利用 **AWS Global Accelerator** 的全球边缘网络，将本地流量引导至**最近的 AWS 边缘位置**再进入 AWS 骨干网络，减少公共互联网传输的距离和抖动，提升连接的**稳定性和延迟一致性**
- **仅支持挂载在 Transit Gateway 上的 VPN 连接**，无法直接用于挂载在传统 VGW 上的 VPN

---

## 多站点连接：VPN CloudHub

- 使用**单个 VGW**配合**多个[[Customer Gateway|客户网关（CGW）]]**，可以让**多个远程站点（分支机构）**不仅能各自连接到 VPC，还能借助 AWS 作为中转，**彼此之间互相通信**（Hub-and-Spoke 拓扑）
- 依赖**动态路由（BGP）**实现路由的自动分发，是"多分支机构 + 低成本快速组网"场景下、无需引入 Transit Gateway 的轻量级方案

---

## 加密与安全性

- 隧道基于 **IPsec** 协议，支持 **AES 128 位和 256 位**加密，保障数据在公共互联网传输过程中的机密性和完整性
- 与 **[[Direct Connect]]** 的核心差异：Direct Connect 专线本身**默认不加密**（需要额外叠加 VPN），而 Site-to-Site VPN **默认自带加密**——这是两者定位的本质区别之一（延迟稳定性 vs 传输安全性的默认取舍）

---

## Site-to-Site VPN vs [[Direct Connect]]（考试高频对比）

| 维度 | Site-to-Site VPN | [[Direct Connect]] |
|------|-------------------|-----------------|
| **部署周期** | 快（分钟到小时级） | 慢（数周到数月） |
| **底层链路** | 公共互联网 | 专用物理线路 |
| **默认加密** | **是**（IPsec） | **否**（需叠加 VPN 才能加密） |
| **延迟/带宽一致性** | 受公网状况影响，波动较大 | 稳定、低延迟、可预测 |
| **典型带宽** | 单隧道约 1.25 Gbps（可用 ECMP 聚合） | 可达数十 Gbps |
| **成本模型** | 按连接时长 + 数据传输计费，成本较低 | 端口费 + 数据传输费，前期投入更高 |
| **适用场景** | 快速上线、灾备、成本敏感、临时连接 | 大规模稳定数据传输、低延迟关键业务 |

> **考试要点**：**两者并非互斥，常组合使用**——用 VPN 作为 [[Direct Connect]] 的**加密补充**（在 DX 的 Public VIF 上叠加 VPN 隧道，兼得专线的稳定性和 VPN 的加密性），或作为 Direct Connect **主链路故障时的备用路径（Failover）**，这是"如何为关键业务设计高可用且加密的混合云连接"这类综合题的标准答案组合。

---

## 典型应用场景

| 场景 | 推荐配置 |
|------|---------|
| **需要快速建立本地到 AWS 的加密连接** | Site-to-Site VPN（分钟到小时级即可上线） |
| **本地网络拓扑经常变化，路由需自动同步** | 动态路由（BGP），而非静态路由 |
| **单条隧道带宽不足，需要更高聚合吞吐** | Transit Gateway + 多条 VPN 连接 + ECMP |
| **需要更稳定、抖动更小的 VPN 连接体验** | Accelerated Site-to-Site VPN（仅限 Transit Gateway 附件） |
| **多个分支机构需要互相通信，且都要连接 VPC** | VPN CloudHub（单 VGW + 多 CGW + BGP） |
| **已有 Direct Connect 但需要加密传输** | [[Direct Connect]] Public VIF 上叠加 Site-to-Site VPN |
| **为 Direct Connect 提供故障时的备用连接** | Site-to-Site VPN 作为 Failover 路径 |
| **大规模稳定数据传输、延迟敏感型业务** | 改用 [[Direct Connect]]（而非仅依赖 VPN） |

---

## 考试重点总结

### SAA-C02 高频考点

1. **核心组件**：VGW/Transit Gateway（AWS 侧）+ [[Customer Gateway]]（本地侧）+ 两条 IPsec 隧道
2. **每个 VPN 连接默认双隧道**：本地设备应同时配置两条隧道才能获得真正的高可用
3. **单隧道带宽约 1.25 Gbps**：需要更高带宽应结合 Transit Gateway + ECMP，而非期待自动扩容
4. **动态路由（BGP）是 AWS 推荐方式**：本地拓扑变化时自动同步路由，是路由传播机制的前提
5. **Accelerated VPN 仅支持 Transit Gateway 附件**：利用 Global Accelerator 降低延迟抖动
6. **VPN CloudHub 支持多分支机构互连**：单 VGW + 多 CGW + BGP 动态路由
7. **VPN 默认加密，Direct Connect 默认不加密**：这是两者最本质的安全属性差异
8. **VPN 部署快、Direct Connect 部署慢**：分钟/小时级 vs 数周/数月级
9. **VPN 与 Direct Connect 常组合而非互斥**：VPN 可作为 DX 的加密补充或故障备用路径
10. **VPN 不支持传递路由**：与 [[VPC Peering]] 的限制逻辑一致，完整说明见 [[Virtual Private Gateway]] 笔记

### 场景题解题思路

```
场景分析 → 判断 Site-to-Site VPN 相关配置
├── "需要快速建立加密的本地-AWS 连接" → Site-to-Site VPN
├── "本地拓扑多变，静态路由难维护" → 改用动态路由（BGP）
├── "单隧道带宽不够，需要更高聚合吞吐" → Transit Gateway + 多 VPN 连接 + ECMP
├── "希望降低 VPN 连接的延迟抖动" → Accelerated Site-to-Site VPN
├── "多个分支机构需要互相通信并连接 VPC" → VPN CloudHub
├── "已有 Direct Connect 但需要加密" → DX Public VIF + Site-to-Site VPN
├── "需要为 Direct Connect 提供故障备用路径" → Site-to-Site VPN 作为 Failover
└── "大规模稳定传输、低延迟关键业务" → 改用 Direct Connect
```

---

## 最佳实践

1. **本地设备务必同时接入两条隧道**：仅使用单隧道等于放弃 AWS 内置的高可用能力
2. **优先使用动态路由（BGP）而非静态路由**：减少本地网络变更时的手动维护成本和出错概率
3. **带宽需求增长时优先评估 Transit Gateway + ECMP**：而非新建更多独立、难以统一管理的 VPN 连接
4. **对延迟敏感的场景评估 Accelerated VPN**：但需确认已采用 Transit Gateway 附件模式
5. **关键业务同时规划 VPN 和 Direct Connect 作为互为备份**：分别覆盖"专线故障"和"临时快速恢复连接"两种场景
6. **Direct Connect 传输敏感数据时叠加 VPN 加密**：不要假设专线本身已提供加密保护
7. **VPN 作为长期主链路前评估其公网带宽和延迟波动是否满足业务 SLA**：不满足时应尽早规划迁移到 Direct Connect
