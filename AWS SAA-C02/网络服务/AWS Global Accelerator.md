# AWS Global Accelerator

> **AWS Global Accelerator** 是一种网络层加速服务，通过**两个静态 Anycast IP 地址**将用户流量引入 AWS 全球骨干网络，绕开公共互联网的拥堵路径，把流量路由到延迟最低、最健康的区域端点（ALB/NLB/EC2/Elastic IP）。与 CloudFront 不同，它**不缓存内容**，面向**任意 TCP/UDP 流量**（不局限于 HTTP/HTTPS），核心价值是**加速网络路径 + 秒级跨区域故障转移**。
>
> 相关文档：[[CloudFront]] | [[Route 53 DNS]] | [[AWS Load Balance]] | [[EC2]] | [[Transit Gateway]] | [[Disaster Recovery On AWS]]

---

## 核心概念

### 工作原理

```
用户请求 → 最近的 AWS 边缘位置（通过 Anycast IP 就近接入）
        → 经 AWS 全球私有骨干网络传输（绕开公共互联网拥堵/不稳定路径）
        → 路由到延迟最低/最健康的区域端点（ALB / NLB / EC2 / Elastic IP）
```

- **Anycast IP**：Global Accelerator 分配**两个固定的静态公网 IP**，全球所有用户访问的是同一组 IP，底层由 AWS 网络自动路由到最优区域——IP **永久不变**，即使后端区域端点增减或故障切换
- **AWS 全球骨干网络**：流量一旦进入最近的 AWS 边缘位置，就沿着 AWS 自建的高速专用网络传输到目标区域，而不是继续走公共互联网，减少丢包、抖动和不可预测的路由跳数

### 核心组件

| 组件 | 说明 |
|------|------|
| **加速器（Accelerator）** | 顶层资源，持有两个静态 Anycast IP |
| **监听器（Listener）** | 定义要加速的端口和协议（TCP/UDP） |
| **端点组（Endpoint Group）** | 按**区域**划分的一组端点，每个区域一个端点组，可设置**流量拨号（Traffic Dial）**控制该区域接收的流量百分比 |
| **端点（Endpoint）** | 实际承接流量的资源：[[AWS Load Balance\|ALB/NLB]]、[[EC2]] 实例、Elastic IP |

---

## 与相似服务的边界（考试高频对比）

### AWS Global Accelerator vs [[CloudFront]]

| 维度 | Global Accelerator | CloudFront |
|------|--------------------|------------|
| **是否缓存内容** | ❌ 不缓存，纯网络路径优化 | ✅ 边缘节点缓存静态/动态内容 |
| **支持的协议** | 任意 **TCP/UDP** 流量 | 仅 **HTTP/HTTPS** |
| **典型场景** | 游戏、IoT、VoIP、非 HTTP 协议应用、需要固定 IP 的场景 | 静态网站、视频点播、API 加速等可缓存的 Web 内容 |
| **公网入口 IP** | **固定的 2 个 Anycast IP**，永久不变 | 边缘节点域名，IP 不保证固定 |
| **核心价值** | 加速**网络路径**本身 + 跨区域快速故障转移 | 减少**回源次数**（缓存命中） |

> **考试关键词识别**：题干强调"需要固定的静态 IP 供客户端**防火墙白名单**"或"应用是**非 HTTP 的 TCP/UDP 协议**（如游戏、VoIP）" → **Global Accelerator**；题干强调"静态网站/视频/API 内容，希望**减少回源、降低延迟**" → **CloudFront**。两者也可以叠加使用（不同场景），但不能互相替代。

### AWS Global Accelerator vs [[Route 53 DNS]]（延迟路由/故障转移路由）

| 维度 | Global Accelerator | Route 53 DNS 路由 |
|------|--------------------|--------------------|
| **路由机制** | 网络层（Anycast IP + AWS 骨干网络） | **DNS 层**，返回不同 IP 给不同客户端 |
| **故障转移速度** | **秒级**——不依赖 DNS 缓存过期 | 依赖 **DNS 记录 TTL 过期**才能让客户端拿到新 IP，且部分客户端/中间 DNS 缓存会**忽略或延长 TTL**，故障转移可能耗时数分钟甚至更久 |
| **入口 IP 是否变化** | 固定不变（2 个 Anycast IP） | 不同策略/不同区域可能返回不同 IP |

> **考试陷阱：需要"快速/近乎实时"的跨区域故障转移时，标准答案是 Global Accelerator，不是 Route 53 故障转移路由**——Route 53 的故障转移**受 DNS TTL 和客户端/中间层 DNS 缓存行为制约**，理论上和实践上都可能比 Global Accelerator 慢得多；题目如果强调"故障切换要快""不希望受 DNS 缓存影响"，优先选 Global Accelerator。

### AWS Global Accelerator vs [[Transit Gateway]]

| 维度 | Global Accelerator | Transit Gateway |
|------|--------------------|------------------|
| **解决的问题** | **外部客户端流量**如何最优路径进入 AWS 并找到最佳区域端点 | **AWS 内部**多个 VPC/多账户/本地数据中心之间如何互联 |
| **流量方向** | 面向公网用户的**入站（Ingress）**加速 | VPC 之间、VPC 与本地网络之间的**内部路由** |

> 两者不是竞争关系，是完全不同层面的问题：Global Accelerator 优化"用户怎么进来"，Transit Gateway 优化"进来之后内部网络怎么互通"。

---

## 高可用与流量控制

### 端点权重与流量拨号

- **端点组的流量拨号（Traffic Dial）**：控制该区域端点组接收的流量百分比（0%~100%），可用于**分阶段发布/灰度**——例如新区域先设 10% 验证，再逐步调高
- **端点权重（Endpoint Weights）**：在同一端点组内，进一步按权重分配流量到具体的多个端点

### 健康检查与自动故障转移

- Global Accelerator 持续对每个端点做健康检查，**自动将流量从不健康端点重新路由到健康端点**（同区域内或跨区域），且客户端**始终使用同一组 Anycast IP**，无需更新 DNS 或客户端配置
- 结合多区域部署（如美东 + 欧洲各一套 ALB），可以实现比纯 DNS 故障转移更快、对客户端更透明的**跨区域灾备**方案

### 客户端亲和性（Client Affinity）

- 默认情况下，Global Accelerator 基于**源 IP + 目的 IP + 端口**做流量分配（五元组哈希）
- 可配置为 **Source IP** 亲和性，确保同一客户端 IP 的请求始终路由到同一端点，适合需要会话保持但协议本身不支持 Cookie（如 UDP）的场景

---

## 典型应用场景

| 场景 | 推荐理由 |
|------|---------|
| **多区域部署的低延迟全球应用**（非 HTTP 协议：游戏服务器、VoIP、IoT） | 需要 TCP/UDP 层加速，CloudFront 只支持 HTTP/HTTPS，无法满足 |
| **需要固定公网 IP 供企业客户防火墙白名单** | Global Accelerator 提供永久不变的 2 个 Anycast IP；ALB/CloudFront 的域名对应 IP 可能变化 |
| **多区域高可用架构，要求秒级故障转移** | 不依赖 DNS TTL，比 Route 53 故障转移路由更快 |
| **需要跨区域灰度发布/分阶段迁移流量** | 用端点组的流量拨号逐步调整各区域流量占比 |
| **面向全球用户的 TCP 类企业应用（如金融交易系统）** | 通过 AWS 骨干网络传输，减少公共互联网的丢包和延迟波动 |

---

## 定价模型

| 计费项 | 说明 |
|--------|------|
| **固定费用** | 每个加速器按小时计费（无论是否有流量） |
| **数据传输费用** | 按实际传输的数据量计费，费率因流量是否经过 AWS 骨干网络的不同环节而异 |

> 相比 CloudFront/Route 53，Global Accelerator 的固定小时费用是必须纳入成本考量的因素——不适合低流量、非关键的场景，更适合**关键业务、全球分布、需要高可用保障**的场景。

---

## 考试重点总结

### SAA-C03 高频考点

1. **定位**：网络层加速，静态 Anycast IP + AWS 骨干网络，不缓存内容
2. **协议范围**：任意 TCP/UDP，不限于 HTTP/HTTPS——这是与 CloudFront 最本质的区别
3. **固定 IP**：两个永久不变的 Anycast IP，适合防火墙白名单场景
4. **故障转移速度**：秒级，不依赖 DNS TTL，明显快于 Route 53 故障转移路由
5. **端点类型**：ALB、NLB、EC2 实例、Elastic IP
6. **流量拨号（Traffic Dial）**：按区域端点组控制流量百分比，用于灰度/分阶段发布
7. **与 Transit Gateway 不冲突**：前者管外部流量入口，后者管内部网络互联

### 场景题解题思路

```
场景分析 → 是否该用 Global Accelerator
├── "应用是游戏/VoIP/IoT 等非 HTTP 的 TCP/UDP 协议，需要全球加速" → Global Accelerator
├── "客户企业防火墙要求白名单固定 IP" → Global Accelerator（Anycast IP 永久不变）
├── "多区域部署，要求故障切换要快、不受 DNS 缓存影响" → Global Accelerator
├── "静态网站/视频/API 等 HTTP(S) 内容，希望减少回源提升速度" → CloudFront（不是 Global Accelerator）
├── "只是希望按地理位置/延迟做 DNS 层路由，没有'秒级切换'的硬性要求" → Route 53 路由策略即可，不需要额外引入 Global Accelerator
├── "需要新区域上线时先小流量验证再逐步放量" → 配置端点组的 Traffic Dial
└── "问题是 VPC/账户/本地网络之间怎么互联，不是外部用户怎么进来" → Transit Gateway，不是 Global Accelerator
```

---

## 最佳实践

1. **非 HTTP 协议或需要固定 IP 时才考虑 Global Accelerator**：HTTP(S) 且可缓存的内容优先用 CloudFront，成本更低
2. **搭配多区域端点部署使用**：单区域场景下 Global Accelerator 的加速/故障转移价值有限，主要收益体现在多区域架构
3. **用流量拨号做灰度发布**：新区域/新版本先小比例验证，观察健康检查和实际表现后再逐步调高
4. **对无 Cookie 机制的协议使用 Source IP 亲和性**：弥补 UDP 等协议缺乏应用层会话保持能力的问题
5. **评估固定小时费用是否值得**：低流量、非关键业务场景，Route 53 + 多区域 ALB 的组合可能更具成本效益
