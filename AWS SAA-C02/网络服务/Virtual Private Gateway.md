# Virtual Private Gateway (VGW) - 虚拟专用网关

> **虚拟专用网关（Virtual Private Gateway, VGW）**是 [[VPC]] 与外部网络（本地数据中心、其他云）之间的 **AWS 侧连接锚点**——建立 [[Site to Site VPN]] 时，它是 VPC 端的 VPN 集中器（云端 VPN 路由器）；使用 [[Direct Connect]] 私有专线时，它也是承载流量进入 VPC 的目标之一。VGW 传统上是"每个 VPC 一个"的连接组件，企业规模扩大后通常会迁移到集中管理的 **[[Transit Gateway]]**。
>
> 相关文档：[[VPC]] | [[Site to Site VPN]] | [[Customer Gateway]] | [[VPC Peering]] | [[Route Table]] | [[CIDR]] | [[Transit Gateway]] | [[Direct Connect]]

---

## 核心概念

### VGW 的双重角色

| 连接方式 | VGW 的角色 |
|---------|-----------|
| **[[Site to Site VPN]]** | 作为 VPC 端的**逻辑 VPN 端点**，与本地侧的**[[Customer Gateway\|客户网关（Customer Gateway, CGW）]]**建立加密 IPsec 隧道，负责流量的加解密处理；完整的隧道冗余、静态/动态（BGP）路由、ECMP 聚合带宽等详见 [[Site to Site VPN]] 独立笔记 |
| **[[Direct Connect]]（DX）** | 作为**私有虚拟接口（Private VIF）**的承载目标之一，让通过专用物理线路传输的流量能够进入指定 VPC，完整内容见 [[Direct Connect]] 独立笔记 |

### 一对一绑定（考试提示）

- **一个传统 VGW 只能附加（Attach）到一个 VPC**——这是 VGW 与后文 [[Transit Gateway]] 最核心的架构差异，也是企业多 VPC 场景下管理复杂度的主要来源
- 一个 VGW 可以同时承载**多个 VPN 连接**（对接多个[[Customer Gateway|客户网关]]），常见于 [[Site to Site VPN]] 笔记中"VPN CloudHub"这类多分支机构组网场景，但**始终只服务一个 VPC**

### 自治系统号（ASN，考试提示）

- 使用动态路由（BGP）时，VGW 拥有一个 **Amazon 侧的私有 ASN**，创建时默认分配（如 64512），也可在创建 VGW 时**自定义指定**
- **考试要点**：若本地网络（[[Customer Gateway|CGW]]）已经使用了与 VGW 默认 ASN**冲突**的私有 ASN 号段，需要在创建 VGW 时**显式指定不同的 ASN**，避免 BGP 邻居关系建立失败，完整的 CGW 侧 ASN 配置要求见 [[Customer Gateway]] 独立笔记

---

## 重要特性与局限

### 内置高可用

- VGW **内部是冗余设计的**——建立 VPN 连接时，AWS 侧会自动提供**两条独立隧道**，分别终结在不同的物理端点上，防止单条路径故障导致连接中断（完整的隧道高可用配置要求见 [[Site to Site VPN]] 独立笔记）

### 不支持传递路由（Transitive Routing，考试最高频考点）

> **考试陷阱**：**这是 VGW 场景下最常被考察的限制**——若 VPC A 通过 VGW 连接了本地机房，即使 VPC A 与 VPC B 之间建立了 [[VPC Peering]] 对等连接，本地机房也**不能**通过 VPC A "借道"访问 VPC B。[[VPC Peering]] 本身同样不支持传递路由，两者的限制是**完全一致的逻辑**——VGW/Peering 提供的都是**严格点对点**的连接，不会自动将"能到达 A"的网络关系传递给"通过 A 能到达的其他网络"。
>
> 若确实需要多方（VPC、本地网络、VPN）之间的**传递性互访**，必须引入原生支持传递路由的 **[[Transit Gateway]]**。

---

## 与路由表的关系

- 私有子网若需要通过 VPN/DX 访问本地网络，其 [[Route Table]] 必须包含指向 **VGW** 的路由
- **路由传播（Route Propagation）**：可在路由表上启用该开关，让 VGW 通过 BGP 从本地网络学到的路由**自动**写入路由表，无需手动维护——完整机制见 [[Route Table]] 笔记中"路由传播"章节

---

## VGW vs Transit Gateway（考试高频对比）

随着企业规模扩大，为每个 VPC 单独管理一个 VGW 会迅速变得复杂——N 个 VPC 都需要连接本地网络时，传统 VGW 模式缺乏集中管理能力，完整的 [[Transit Gateway]] 附件类型、路由表分段和跨账号共享机制见 [[Transit Gateway]] 独立笔记。

| 维度 | Virtual Private Gateway | [[Transit Gateway]] |
|------|--------------------------|-------------------|
| **VPC 关联数量** | 一对一，每个 VPC 需要独立的 VGW | 一对多，单个 Transit Gateway 可连接**成百上千个** VPC |
| **传递路由** | **不支持** | **原生支持**，可作为中央路由器让多个 VPC/VPN/DX 之间互访 |
| **架构模式** | 分散式（每 VPC 一个连接点） | 中心枢纽（星型拓扑，集中管理） |
| **适用规模** | 少量 VPC、简单连接需求 | 大规模多 VPC、多账户、混合云架构 |
| **管理复杂度** | VPC 数量增多时呈线性增长，难以统一维护 | 集中管理路由表，新增 VPC 只需接入枢纽 |

> **考试要点**：题目描述"企业有大量 VPC 都需要连接本地数据中心，且要求彼此间也能互访" → **Transit Gateway** 会取代 VGW 的角色，成为集中式的连接枢纽；题目描述"仅有一两个 VPC 需要连接本地网络的简单场景" → 传统 VGW 已经足够，无需引入 Transit Gateway 的额外复杂度。

---

## 定价

- **VGW 本身免费**，不产生额外费用
- 实际费用来自其承载的连接：[[Site to Site VPN]] 按**连接时长 + 数据传输量**计费；[[Direct Connect]] 按**端口带宽 + 数据传输量**计费

---

## 典型应用场景

| 场景 | 推荐配置 |
|------|---------|
| **单个 VPC 需要通过 VPN 连接本地数据中心** | VGW + [[Site to Site VPN]] |
| **单个 VPC 需要通过专线连接本地数据中心** | VGW + [[Direct Connect]]（Private VIF） |
| **本地网络路由频繁变化，需自动同步** | VGW 启用路由传播 + 动态 BGP VPN |
| **本地 CGW 的 ASN 与 VGW 默认 ASN 冲突** | 创建 VGW 时自定义指定不同的私有 ASN |
| **本地网络需要"借道"某 VPC 访问其 Peering 的另一 VPC** | 不支持（不支持传递路由），需改用 Transit Gateway |
| **大量 VPC 都需要连接本地网络并彼此互访** | 改用 Transit Gateway（取代分散的 VGW 模式） |

---

## 考试重点总结

### SAA-C02 高频考点

1. **VGW 是 AWS 侧的连接锚点**：服务于 Site-to-Site VPN（VPN 集中器）和 Direct Connect（Private VIF 目标）两种连接方式
2. **一个 VGW 只能附加到一个 VPC**：这是与 Transit Gateway 的核心架构差异
3. **一个 VGW 可承载多个 VPN 连接**：但始终只服务单一 VPC，多分支机构场景见 VPN CloudHub
4. **VGW 内部自带隧道冗余**：VPN 连接默认提供两条独立隧道
5. **不支持传递路由**：VPC 通过 VGW 连接本地网络后，本地网络无法借道该 VPC 访问其 Peering 的其他 VPC
6. **VGW 拥有可自定义的私有 ASN**：本地 ASN 冲突时需在创建阶段指定不同值
7. **路由表需启用路由传播才能自动同步 BGP 路由**：否则需手动维护静态路由
8. **VGW 本身免费**：费用产生自其承载的 VPN/DX 连接
9. **规模扩大后应迁移到 Transit Gateway**：获得原生传递路由和集中管理能力
10. **VGW 与 VPC Peering 的"不支持传递路由"是同一逻辑**：两者都是严格点对点连接，不会自动传递可达性

### 场景题解题思路

```
场景分析 → 判断 VGW 相关配置
├── "单个 VPC 需要连接本地数据中心（VPN）" → VGW + Site-to-Site VPN
├── "单个 VPC 需要连接本地数据中心（专线）" → VGW + [[Direct Connect]]
├── "本地网络借道 VPC 访问其 Peering 的另一 VPC" → 不支持，需改用 Transit Gateway
├── "本地 CGW ASN 与 VGW 默认 ASN 冲突" → 创建 VGW 时自定义 ASN
├── "本地路由多变，需自动同步" → 启用路由传播 + 动态 BGP
├── "大量 VPC 都需连接本地网络并彼此互访" → 改用 Transit Gateway
└── "VPN 连接单条隧道故障" → 依赖 VGW 内置的双隧道自动切换到另一条
```

---

## 最佳实践

1. **少量 VPC 的简单连接场景使用 VGW 已足够**：无需为了"面向未来"而提前引入 Transit Gateway 的复杂度
2. **预判企业规模增长趋势，提前规划向 Transit Gateway 迁移的路径**：避免后期大规模重构网络架构
3. **本地网络存在自有 BGP ASN 时，创建 VGW 前先核对是否冲突**：避免 BGP 邻居建立失败后才排查
4. **路由表配合 VGW 时优先启用路由传播**：减少人工维护静态路由的成本和出错概率
5. **需要多方传递性互访时不要试图用多条 Peering/VGW 拼凑**：不支持传递路由的限制无法通过嵌套连接绕过，应直接采用 Transit Gateway
6. **关键业务的 VPN 连接确保本地设备接入两条隧道**：详见 [[Site to Site VPN]] 笔记中的高可用配置要求
