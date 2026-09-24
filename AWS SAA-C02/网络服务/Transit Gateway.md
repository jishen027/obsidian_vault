# Transit Gateway - 传输网关

> **AWS Transit Gateway** 是**中心枢纽（星型拓扑）**式的区域级网络中转服务，可同时连接成百上千个 [[VPC]]、[[Site to Site VPN]] 连接、[[Direct Connect]]、甚至其他区域的 Transit Gateway，并**原生支持传递路由（Transitive Routing）**——这是它与 [[VPC Peering]]（不支持传递路由，点对点连接呈平方级增长）和传统 [[Virtual Private Gateway]]（一对一绑定单个 VPC）的核心区别，也是大规模混合云/多账户架构的标准连接枢纽。
>
> 相关文档：[[VPC]] | [[VPC Peering]] | [[Virtual Private Gateway]] | [[Site to Site VPN]] | [[Direct Connect]] | [[Customer Gateway]] | [[Route Table]] | [[CIDR]] | [[AWS Global Accelerator]]

> **易混淆点**：Transit Gateway 解决的是 **AWS 内部**（VPC 之间、VPC 与本地网络之间）怎么互联的问题；[[AWS Global Accelerator]] 解决的是**外部客户端流量**怎么以最优路径进入 AWS 并找到最佳区域端点的问题——两者面向的流量方向和层面完全不同，不是互相替代关系。

---

## 核心概念

### 为什么需要 Transit Gateway

- **VPC Peering 的瓶颈**：N 个 VPC 若要全互联，需要 N×(N-1)/2 条点对点连接，且**不支持传递路由**，管理复杂度随规模呈**平方级增长**，完整限制见 [[VPC Peering]] 独立笔记
- **传统 VGW 的瓶颈**：一个 VGW **只能附加到一个 VPC**，企业有大量 VPC 都需要连接本地网络时，需要为每个 VPC 单独维护 VGW 和 VPN 连接，完整对比见 [[Virtual Private Gateway]] 独立笔记
- **Transit Gateway 的核心价值**：作为**单一的中心路由器**，所有 VPC/VPN/DX 只需连接到这一个枢纽即可**互相访问**（受路由表控制），新增一个 VPC 只需新建一条"附件（Attachment）"连接到枢纽，而非与所有现有 VPC 逐一建立连接

---

## 附件类型（Attachment）

| 附件类型 | 连接对象 |
|---------|---------|
| **VPC 附件** | 将 [[VPC]] 内指定子网连接到 Transit Gateway |
| **VPN 附件** | 将 [[Site to Site VPN]] 连接挂载到 Transit Gateway（而非传统 VGW），支持 **ECMP** 聚合多条隧道带宽，完整机制见 [[Site to Site VPN]] 独立笔记 |
| **Direct Connect Gateway 附件** | 通过 **Direct Connect Gateway** 关联，让单条 [[Direct Connect]] 专线可经由 Transit Gateway 触达**多个区域**的多个 VPC，完整机制见 [[Direct Connect]] 独立笔记 |
| **Transit Gateway 对等连接（Peering Attachment）** | 连接**另一个区域**的 Transit Gateway，实现跨区域的网络互联 |
| **Connect 附件** | 通过 **GRE 隧道**集成第三方 SD-WAN 设备/软件路由器，将其纳入 Transit Gateway 的路由体系 |

---

## Transit Gateway 路由表（考试高频）

- Transit Gateway 拥有**独立于** [[VPC]] [[Route Table|路由表]]的**专属路由表**——每个附件在关联到 Transit Gateway 时，需要**关联（Associate）**到某个 Transit Gateway 路由表，决定该附件能查到哪些路由
- **支持创建多个 Transit Gateway 路由表**，实现网络分段（Segmentation）——例如让"生产环境 VPC"和"开发环境 VPC"分别关联不同的路由表，彼此**默认不可达**，即使它们连接到同一个 Transit Gateway
- **路由传播（Route Propagation）**：附件的路由可以**自动传播**到指定的 Transit Gateway 路由表（VPC 附件传播其 CIDR、VPN/DX 附件通过 BGP 传播动态路由），减少手动维护

> **考试要点**：**"需要让某些 VPC 互通，同时让另一些 VPC 彼此隔离，都连接同一个 Transit Gateway" → 通过创建多个 Transit Gateway 路由表并分别关联/传播不同附件实现**，而不是为隔离的 VPC 单独建一个新的 Transit Gateway——这是 Transit Gateway 支持多租户网络分段的核心机制。

---

## 共享服务架构（Shared Services VPC Pattern）

- 常见架构模式：将 DNS、Active Directory、监控代理等**共享服务**部署在一个独立的"共享服务 VPC"中，其余业务 VPC 通过 Transit Gateway 路由表**只允许访问共享服务 VPC**，业务 VPC 之间**互相隔离**
- 通过精细设计 Transit Gateway 路由表的关联和传播关系即可实现，无需额外的网络设备

---

## 跨区域连接（Transit Gateway Peering）

- Transit Gateway 之间的对等连接**支持跨区域**，流量在 AWS 骨干网络中**自动加密**传输
- **考试陷阱**：**Transit Gateway Peering 连接不支持路由传播，必须手动在双方路由表中添加静态路由**——这与 VPC 附件的自动路由传播不同，是"配置了 TGW Peering 但对端网络仍不可达"这类场景题的高频排查点

---

## 跨账号共享（AWS RAM）

- 通过 **AWS Resource Access Manager（RAM）**，可以将一个 Transit Gateway **共享给其他 AWS 账号或整个组织（Organization）**，其他账号可以直接创建附件连接到该 Transit Gateway，**无需在每个账号中重复创建**独立的 Transit Gateway
- 这是多账号架构（结合 [[AWS Organizations]]）中实现**集中式网络枢纽**的标准做法——网络团队在中心账号管理唯一的 Transit Gateway，业务账号只需连接附件

---

## 带宽与性能

- **每个 VPC 附件**支持高达 **50 Gbps** 的突发带宽，无需额外配置即可自动扩展
- VPN 附件受限于 Site-to-Site VPN 本身**单隧道约 1.25 Gbps** 的上限，如需更高聚合带宽需结合 **ECMP** 跨多条隧道负载均衡，详见 [[Site to Site VPN]] 独立笔记

---

## 网络可观测性：Transit Gateway Network Manager

- 提供**全局网络拓扑可视化**，集中查看跨区域的 Transit Gateway、VPC、[[Direct Connect]]、VPN 连接的拓扑关系和健康状态，尤其适合管理复杂的全球化混合云网络

---

## 定价

- 按**每个附件的小时费**（Attachment-hour）+ **数据处理量**计费——附件数量越多、流经的数据量越大，成本越高，规划架构时应评估是否所有 VPC 都真的需要接入枢纽

---

## Transit Gateway vs VPC Peering vs Virtual Private Gateway（考试高频对比）

| 维度 | Transit Gateway | [[VPC Peering]] | [[Virtual Private Gateway]] |
|------|-------------------|--------------------|--------------------------------|
| **拓扑** | 星型枢纽，中心化管理 | 点对点（Mesh 需 N² 级连接） | 一对一绑定单个 VPC |
| **传递路由** | **原生支持** | **不支持** | **不支持** |
| **适用规模** | 大规模（几十到上千个）VPC/VPN/DX 混合互联 | 少量 VPC（2-10 个左右）直连 | 单 VPC 连接本地网络的简单场景 |
| **跨区域** | 支持（Transit Gateway Peering） | 支持（Inter-Region Peering） | 不适用（VGW 本身是区域级资源） |
| **成本** | 按附件小时 + 数据处理量，规模大时更具性价比 | 连接免费，仅数据传输费（跨区域） | 免费，费用产生自其承载的 VPN/DX 连接 |
| **跨账号共享** | 支持（AWS RAM） | 支持（跨账号 Peering） | 不适用 |
| **典型场景** | 企业级多账户、多 VPC、混合云架构 | 两三个 VPC 之间的简单直连 | 单 VPC 的 VPN/DX 连接 |

> **考试要点**：**判断依据是"连接规模"和"是否需要传递路由/网络分段"**——题目描述"仅需连接两三个 VPC" → VPC Peering 已经足够；描述"需要连接数十上百个 VPC，或需要通过中心枢纽统一管理路由、实现网段隔离" → **Transit Gateway**；描述"单个 VPC 连接本地数据中心的简单 VPN/DX 场景" → 传统 [[Virtual Private Gateway]] 已经足够。

---

## 典型应用场景

| 场景 | 推荐配置 |
|------|---------|
| **大规模多 VPC/多账户需要互联并支持传递访问** | Transit Gateway |
| **需要让部分 VPC 互通，另一部分彼此隔离** | 多个 Transit Gateway 路由表 + 差异化关联/传播 |
| **共享服务（DNS/AD/监控）需要被所有业务 VPC 访问，业务 VPC 间彼此隔离** | Shared Services VPC + Transit Gateway 路由表分段 |
| **单条 Direct Connect 专线需要触达多区域的多个 VPC** | [[Direct Connect]] Gateway + Transit Gateway |
| **需要跨区域连接两个 Transit Gateway** | Transit Gateway Peering（需手动添加静态路由） |
| **多账户组织需要共享同一个网络枢纽，避免重复建设** | AWS RAM 共享 Transit Gateway |
| **VPN 单隧道带宽不足** | Transit Gateway + 多条 VPN 连接 + ECMP |
| **需要集中查看全球网络拓扑和健康状态** | Transit Gateway Network Manager |
| **仅需连接两三个 VPC，无传递路由需求** | 改用 [[VPC Peering]]（而非 Transit Gateway） |

---

## 考试重点总结

### SAA-C02 高频考点

1. **核心定位**：星型枢纽式区域级网络中转服务，原生支持传递路由
2. **五种附件类型**：VPC、VPN、[[Direct Connect]] Gateway、Transit Gateway Peering、Connect（GRE/SD-WAN）
3. **拥有独立于 VPC 路由表的专属路由表**：可创建多个路由表实现网络分段
4. **路由传播自动同步附件路由**：VPC 附件传播 CIDR，VPN/DX 附件通过 BGP 传播
5. **Transit Gateway Peering 不支持路由传播**：必须手动添加静态路由，这与 VPC 附件不同
6. **跨区域 Peering 流量自动加密**：无需额外配置加密隧道
7. **AWS RAM 支持跨账号/跨组织共享**：避免每个账号重复创建 Transit Gateway
8. **每个 VPC 附件支持高达 50 Gbps 突发带宽**：VPN 附件仍受限于隧道本身的带宽上限
9. **按附件小时 + 数据处理量计费**：附件数量和流量规模直接影响成本
10. **判断依据是连接规模和是否需要传递路由/网段隔离**：少量 VPC 简单直连选 Peering，大规模/需要传递路由/分段选 Transit Gateway

### 场景题解题思路

```
场景分析 → 判断是否使用 Transit Gateway
├── "大规模多 VPC/多账户需要互联并支持传递访问" → Transit Gateway
├── "部分 VPC 需互通，部分需隔离，共用同一枢纽" → 多个 TGW 路由表分段
├── "共享服务 VPC 模式，业务 VPC 间彼此隔离" → Shared Services VPC + TGW 路由表
├── "单条专线需要触达多区域多个 VPC" → [[Direct Connect]] Gateway + Transit Gateway
├── "需要跨区域连接两个 TGW" → TGW Peering（记得手动加静态路由）
├── "多账户需共享同一网络枢纽" → AWS RAM 共享 Transit Gateway
├── "VPN 单隧道带宽不足" → TGW + 多 VPN 连接 + ECMP
└── "仅需连接两三个 VPC，无传递路由需求" → 改用 VPC Peering
```

---

## 最佳实践

1. **大规模、持续增长的多 VPC 架构优先采用 Transit Gateway**：避免后期从 Peering Mesh 迁移的重构成本
2. **善用多个 Transit Gateway 路由表实现网络分段**：而非为需要隔离的环境单独建立多个 Transit Gateway
3. **多账户组织通过 AWS RAM 集中共享单一 Transit Gateway**：网络团队统一管理，业务账号只需创建附件
4. **Transit Gateway Peering 后立即核对路由表是否已手动添加静态路由**：这一步不会自动完成
5. **共享服务架构优先使用 Shared Services VPC + 路由表分段模式**：集中管理 DNS/AD 等基础设施服务
6. **持续监控附件数量和数据处理量对成本的影响**：并非所有 VPC 都必须接入枢纽，评估实际互联需求
7. **复杂全球化网络优先启用 Transit Gateway Network Manager**：获得统一的拓扑可视化，加快故障定位
8. **VPN 带宽需求增长时优先在 Transit Gateway 层面用 ECMP 扩容**：而非新建大量难以统一管理的独立 VPN 连接
