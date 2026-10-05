# Direct Connect - 直连专线

> **AWS Direct Connect（DX）**是本地数据中心与 AWS 之间的**专用物理网络连接**，流量完全绕开公共互联网，提供**稳定、低延迟、高带宽**的混合云连接能力。与 [[Site to Site VPN]] 的"快速但依赖公网、默认加密"形成鲜明对比：Direct Connect **默认不加密**、部署周期长达数周到数月，但延迟更稳定、带宽更大、数据传输成本更低——两者并非互斥的竞品，而是常常组合使用的互补方案。
>
> 相关文档：[[VPC]] | [[Virtual Private Gateway]] | [[Transit Gateway]] | [[Site to Site VPN]] | [[Customer Gateway]] | [[ENI]] | [[Route Table]] | [[CIDR]] | [[AWS Outposts]]

---

## 核心概念

### 物理连接类型

| 类型 | 说明 |
|------|------|
| **专用连接（Dedicated Connection）** | 直接向 AWS 申请，在 **AWS Direct Connect 地点**由 AWS 提供物理端口，带宽为 **1/10/100 Gbps**，一次只能对应一个客户 |
| **托管连接（Hosted Connection）** | 通过 **AWS Direct Connect 合作伙伴**申请，带宽范围更细（**50 Mbps 到 10 Gbps**），部署周期通常比专用连接更短，但带宽调整需经合作伙伴处理 |

> **考试要点**：题目强调"需要 100 Gbps 级别带宽"或"企业希望自行管理物理端口" → **专用连接**；题目强调"带宽需求不大（如 500 Mbps）"或"没有直接进驻 Direct Connect 地点的条件" → **托管连接**（经合作伙伴）。

### 部署周期与 VPN 的对比（考试高频）

- Direct Connect 需要**申请物理线路、机房对接、布线施工**，通常耗时**数周到数月**，无法满足紧急连接需求
- 完整的"部署周期、加密默认状态、带宽稳定性"三维对比详见 [[Site to Site VPN]] 独立笔记的《Site-to-Site VPN vs Direct Connect》章节

> **考试陷阱**：**Direct Connect 默认不加密**——很多人误以为"专用物理线路 = 安全"，但物理隔离≠数据加密；题目描述"通过 Direct Connect 传输敏感数据，同时要求加密" → 在 **Public VIF 上叠加 Site-to-Site VPN** 实现"专线的稳定性 + VPN 的加密性"，而非误认为 Direct Connect 自带加密。

---

## 虚拟接口（VIF）

物理连接建立后，还需要在其上创建**虚拟接口**才能实际承载流量，三种 VIF 类型分别对应不同的访问目标。完整的 VIF 类型、用途对照表和考试要点见 [[ENI]] 独立笔记中的《Direct Connect 虚拟接口（VIF）》章节，核心结论：

| VIF 类型 | 用途 | AWS 侧对接目标 |
|---------|------|---------------|
| **私有 VIF（Private VIF）** | 访问单个 VPC 内的私有资源 | [[Virtual Private Gateway]] 或 Direct Connect Gateway |
| **公有 VIF（Public VIF）** | 访问 S3/DynamoDB 等具有公网端点的 AWS 服务 | AWS 公网服务端点（不经过公共互联网路由） |
| **中转 VIF（Transit VIF）** | 大规模互联成百上千个 VPC | **[[Transit Gateway]]**（通过 Direct Connect Gateway 关联） |

- **同一条物理连接上可同时创建多种 VIF**，VIF 类型的选择完全取决于目标资源的位置和连接规模，与物理线路本身无关

---

## Direct Connect Gateway（考试提示）

- 默认情况下，一条 Direct Connect 连接的私有 VIF **只能关联一个 VGW（对应一个 VPC）**
- **Direct Connect Gateway** 打破这一限制，让**单条 Direct Connect 连接**能够同时关联**多个区域、多个账号**下的多个 VGW（或一个 [[Transit Gateway]]），实现"一条专线触达大量 VPC"

| 关联目标 | 适用场景 |
|---------|---------|
| **多个 VGW** | 单条专线连接多个独立 VPC，各 VPC 之间**不需要**互通 |
| **[[Transit Gateway]]** | 单条专线连接多个 VPC，且这些 VPC **之间也需要**通过 Transit Gateway 互通（更具扩展性的推荐做法） |

> **考试陷阱**：**Direct Connect Gateway 本身不提供 VPC 间的传递路由**——它只是让一条专线能"触达"更多 VGW，各 VPC 之间是否互通仍取决于是否使用了支持传递路由的 Transit Gateway；题目描述"多个 VPC 需要共享同一条专线，且彼此也要互通" → **Direct Connect Gateway + Transit Gateway** 组合，而非误以为 Direct Connect Gateway 单独就能实现 VPC 互通。

---

## 链路聚合组（LAG，Link Aggregation Group）

- 将**多条位于同一 Direct Connect 地点、相同带宽**的物理连接捆绑成一个逻辑连接，实现更高的聚合带宽，并作为单个连接统一管理（一次配置应用到组内所有连接）
- 常见于单条连接带宽不足、又不想管理多条独立连接配置的场景

---

## 高可用与弹性（Resiliency）

- **单条 Direct Connect 连接是单点故障**——物理线路、设备或 Direct Connect 地点本身的故障都会导致连接中断
- **AWS Direct Connect Resiliency Toolkit** 提供多种弹性模型，核心思路是在**多个不同的 Direct Connect 地点**建立**多条独立连接**：

| 弹性模型 | 说明 |
|---------|------|
| **开发/测试** | 单条连接，无高可用保障 |
| **高韧性（High Resiliency）** | 在同一地点的不同设备上建立多条连接，或跨两个地点各一条 |
| **最大韧性（Maximum Resiliency）** | 跨**至少两个不同的 Direct Connect 地点**，每个地点内使用不同设备的独立连接，达到关键业务 SLA 要求 |

> **考试要点**：题目描述"关键业务不能接受 Direct Connect 单点故障" → 评估**最大韧性模型**（跨多个 Direct Connect 地点）；若预算或场景不要求最高等级容灾，也应至少满足**高韧性**而非单条连接裸奔。此外，**Site-to-Site VPN 作为 Direct Connect 的故障备用路径（Failover）**也是常见的低成本高可用组合，完整说明见 [[Site to Site VPN]] 笔记。

---

## 路由与加密

- Direct Connect **要求使用 BGP** 进行动态路由交换（不支持纯静态路由），这与 [[Site to Site VPN]] "静态或 BGP 均可"的灵活性不同
- **MACsec（802.1AE）**：部分 Direct Connect 地点的专用连接支持**二层（物理层）加密**，为需要加密但不希望叠加 VPN 隧道（避免额外的 IPsec 封装开销）的场景提供原生加密选项
- 若所在地点不支持 MACsec 或使用的是托管连接，实现加密的标准做法仍是**在 Public/Private VIF 上叠加 Site-to-Site VPN**

---

## 定价模型

| 计费项 | 说明 |
|--------|------|
| **端口费用（Port Hour）** | 按连接的**带宽等级和时长**收费，与是否有实际流量无关 |
| **数据传出（Data Transfer Out）** | 按 GB 收费，**通常显著低于**通过公共互联网传出的费用 |
| **数据传入（Data Transfer In）** | **免费** |

> **考试要点**：**大规模、持续的数据传出流量场景，Direct Connect 的长期成本通常低于公网传输**——这是"降低跨区域/混合云数据传输成本"类题目在排除 VPN/公网方案后的常见最终答案。

---

## 典型应用场景

| 场景 | 推荐配置 |
|------|---------|
| **紧急/临时需要打通本地与 VPC 的连接** | [[Site to Site VPN]]（而非 Direct Connect，部署周期不满足需求） |
| **大规模、持续、低延迟敏感的数据传输** | Direct Connect |
| **已有 Direct Connect 但需要加密敏感数据** | Public/Private VIF 叠加 Site-to-Site VPN，或使用支持 MACsec 的连接 |
| **单条专线需要连接多个区域/账号的多个 VPC（VPC 间无需互通）** | Direct Connect Gateway + 多个 VGW |
| **单条专线连接的多个 VPC 之间也需要互通** | Direct Connect Gateway + [[Transit Gateway]] |
| **单条连接带宽已达上限** | LAG 捆绑同地点、同带宽的多条连接 |
| **关键业务不能接受专线单点故障** | Direct Connect Resiliency Toolkit（跨多个 Direct Connect 地点部署冗余连接）+ VPN 故障备份 |
| **访问 S3/DynamoDB 等公网端点服务但不经过公共互联网** | 公有 VIF |
| **[[AWS Outposts]] 需要与父区域保持可靠的控制平面连接** | Private VIF（或 Site-to-Site VPN），完整依赖关系见 [[AWS Outposts]] 独立笔记 |

---

## 考试重点总结

### SAA-C02 高频考点

1. **Direct Connect 是专用物理连接，完全绕开公共互联网**：提供比 VPN 更稳定的延迟和更大的带宽
2. **默认不加密**：需要加密时需叠加 VPN（Public/Private VIF 上跑 IPsec）或使用支持 MACsec 的连接
3. **部署周期数周到数月**：紧急连接需求应优先选择 [[Site to Site VPN]]
4. **专用连接 vs 托管连接**：前者直接向 AWS 申请、带宽为 1/10/100 Gbps 固定档位；后者经合作伙伴申请、带宽范围更细
5. **三种 VIF 类型对应不同目标**：私有 VIF（VPC 私有资源）、公有 VIF（AWS 公网端点服务）、中转 VIF（经 Transit Gateway 大规模互联），详见 [[ENI]]
6. **Direct Connect Gateway 让单条专线触达多个 VGW/账号/区域**：但本身不提供 VPC 间传递路由，需搭配 Transit Gateway 才能实现 VPC 互通
7. **LAG 捆绑同地点同带宽的多条连接**：提升聚合带宽，统一管理配置
8. **单条连接是单点故障**：应通过 Resiliency Toolkit 跨多个 Direct Connect 地点建立冗余
9. **要求 BGP 动态路由**：与 VPN 的"静态或 BGP 均可"不同
10. **数据传出成本通常低于公网**：大规模持续传输场景的长期成本优势明显

### 场景题解题思路

```
场景分析 → 判断 Direct Connect 相关配置
├── "紧急/临时需要连接本地与 VPC" → Site-to-Site VPN（而非 Direct Connect）
├── "大规模持续、低延迟敏感传输" → Direct Connect
├── "已有专线需要加密" → 叠加 Site-to-Site VPN 或使用 MACsec
├── "单条专线需连接多区域/账号的多个 VPC（无需互通）" → Direct Connect Gateway + 多 VGW
├── "单条专线连接的多个 VPC 之间也要互通" → Direct Connect Gateway + Transit Gateway
├── "单条连接带宽不足" → LAG 捆绑多条同规格连接
├── "关键业务不能接受专线单点故障" → Resiliency Toolkit（跨多个 DX 地点）+ VPN 备份
└── "降低大规模数据传出成本" → 评估迁移到 Direct Connect
```

---

## 最佳实践

1. **不要用 Direct Connect 满足紧急连接需求**：数周到数月的部署周期决定了它不适合临时/紧急场景，应优先用 VPN 过渡
2. **传输敏感数据时不要假设专线已加密**：明确叠加 VPN 或确认所在地点支持 MACsec
3. **关键业务至少规划两条位于不同 Direct Connect 地点的连接**：避免单一物理线路或地点故障导致完全中断
4. **多 VPC 场景优先通过 Direct Connect Gateway + Transit Gateway 组合**：而非为每个 VPC 单独申请专线或简单堆叠 VGW 关联
5. **将 Site-to-Site VPN 作为 Direct Connect 的标准故障备份路径**：以较低成本获得额外的连接冗余
6. **持续评估数据传出量是否已达到 Direct Connect 更具成本优势的规模**：小流量场景 VPN 可能已经足够，避免过度设计
7. **单条连接带宽逼近上限时优先评估 LAG，而非直接申请全新的独立连接**：降低管理复杂度