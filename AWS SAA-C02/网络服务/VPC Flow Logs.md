# VPC Flow Logs - 网络流日志

> **VPC Flow Logs（流日志）**捕获进出 [[VPC]] 内网络接口（[[ENI]]）的 **IP 流量元数据**（谁访问了谁、用了哪个端口、传输了多少字节、被允许还是被拒绝），是排查连接性故障、安全分析和合规审计的核心工具。它记录的是**流量的"信封信息"**，而非数据包本身的内容——这是它与 **[[VPC Traffic Mirroring|VPC 流量镜像（Traffic Mirroring）]]**最本质的区别，也是最容易被考察的边界。
>
> 相关文档：[[VPC]] | [[ENI]] | [[Security Group]] | [[NACL]] | [[CloudWatch]] | [[Amazon GuardDuty]] | [[AWS Organizations]] | [[S3]] | [[VPC Traffic Mirroring]]

---

## 核心概念

### 为什么需要 Flow Logs

- **默认情况下 VPC 内的网络流量不可见**——无法直接得知某个连接是被安全组/NACL 放行还是拒绝、也无法追溯历史流量模式
- **Flow Logs 的核心价值**：在**不影响网络吞吐或延迟**的前提下，持续记录流经 ENI 的每条流量的元数据，为**故障排查、安全分析、成本审计**提供数据基础

### 三种捕获级别

| 级别 | 覆盖范围 |
|------|---------|
| **VPC 级别** | 捕获该 VPC 内**所有** ENI 的流量 |
| **子网级别** | 捕获该 [[Subnet]] 内**所有**实例 ENI 的流量 |
| **ENI 级别** | 仅捕获**单个**指定网络接口的流量 |

- 无论在哪个级别启用，底层实现都是**为该范围内每个 ENI 单独生成日志记录**——VPC/子网级别只是简化了批量启用的操作，不代表流量在网关层面被统一采样

---

## 日志记录格式（考试高频）

### 默认字段

标准日志记录包含以下核心字段（空格分隔）：

| 字段 | 含义 |
|------|------|
| **version** | Flow Log 版本 |
| **account-id** | AWS 账号 ID |
| **interface-id** | 流量所属的 [[ENI]] ID |
| **srcaddr / dstaddr** | 源/目的 IP 地址 |
| **srcport / dstport** | 源/目的端口 |
| **protocol** | IANA 协议号（如 6=TCP，17=UDP） |
| **packets / bytes** | 该记录聚合窗口内的数据包数/字节数 |
| **start / end** | 聚合窗口的起止时间（Unix 时间戳） |
| **action** | **ACCEPT** 或 **REJECT**——该流量是否被允许通过 |
| **log-status** | 日志记录本身的状态（OK / NODATA / SKIPDATA） |

### 自定义格式（考试提示）

- 除默认字段外，可**自定义选择字段**（如 VPC ID、子网 ID、TCP 标志位、流量方向 direction、pkt-srcaddr 等）——用于满足特定分析需求，减少无关字段带来的存储和查询成本
- **考试要点**：题目描述"需要精确追踪某条流量的 TCP 连接状态（SYN/ACK/FIN）" → 使用**自定义格式**，纳入 `tcp-flags` 字段，默认格式不包含该信息

---

## 目标存储位置

| 目标                                                                        | 特点                                                                           |
| ------------------------------------------------------------------------- | ---------------------------------------------------------------------------- |
| **[[CloudWatch]] Logs**                                                   | 便于**实时查询**（CloudWatch Logs Insights）和设置告警，需要为 Flow Logs 配置具备写入权限的 **IAM 角色** |
| **[[S3]]**                                                                | 适合**长期归档**和大规模离线分析（如结合 [[Amazon Athena]] 查询），成本更低                            |
| **[[Amazon Kinesis#Kinesis Data Firehose（交付流）\| Kinesis Data Firehose]]** | 支持**近实时流式处理**，可进一步转发到第三方 SIEM/分析工具                                           |

> **考试要点**：题目描述"需要长期低成本保留海量流日志用于合规审计" → 投递到 **S3**；描述"需要实时查询和基于流量模式设置 CloudWatch 告警" → 投递到 **CloudWatch Logs**。

---

## Flow Logs 不捕获的流量（考试最高频陷阱）

> **考试陷阱**：**Flow Logs 并非捕获"所有"网络流量**，以下类型默认**不会**出现在日志中：
> - 实例访问 **Amazon DNS 服务器**（Route 53 Resolver，`169.254.169.253`）产生的流量
> - 实例访问**实例元数据服务**（`169.254.169.254`）产生的流量
> - 为 Windows 实例激活许可证而产生的流量
> - 发往/来自 VPC 路由器**保留 IP 地址**（每个子网 CIDR 的第 2 位地址）的流量
> - 被**[[VPC Traffic Mirroring|流量镜像（Traffic Mirroring）]]**捕获的镜像流量本身
>
> 题目描述"启用了 Flow Logs 但日志中看不到实例对 DNS 的查询记录" → **属于预期行为**，并非配置错误。

---

## Flow Logs vs 流量镜像（[[VPC Traffic Mirroring]]，考试高频对比）

| 维度 | VPC Flow Logs | [[VPC Traffic Mirroring]] |
|------|----------------|--------------------------|
| **记录内容** | 流量**元数据**（五元组、字节数、动作） | **完整数据包**（包括 payload 内容） |
| **典型用途** | 连接性排查、访问模式分析、安全审计 | 深度包检测（DPI）、入侵检测、协议级故障排查 |
| **性能开销** | 极低，不影响转发路径 | 需要额外镜像流量到目标（NLB/ENI），开销更高 |
| **实时性** | 近实时（有聚合窗口延迟） | 实时镜像 |

> **考试陷阱**：**题目描述"需要检查数据包的实际内容/载荷（payload）"，Flow Logs 无法满足**——它只记录连接的元数据，不涉及包内容；需要检测数据包内容或做深度协议分析时，应使用 **[[VPC Traffic Mirroring]]** 或第三方 IDS/IPS 方案，而非期待从 Flow Logs 中获取。完整的组件、过滤器、跨 VPC 部署方式见 [[VPC Traffic Mirroring]] 独立笔记。

---

## 与安全组/NACL 的关系

- Flow Logs 记录的 **`action`（ACCEPT/REJECT）**反映的是该流量**是否被安全组或 NACL 放行**，是排查"连接为何失败"的第一手资料
- **典型排障流程**：连接失败 → 查看 Flow Logs 中对应流量记录的 `action` → 若为 **REJECT** → 检查 [[Security Group]] 和 [[NACL]] 规则找出具体拦截点；若日志中**完全没有该记录** → 问题可能出在路由层面（[[Route Table]]）而非访问控制层面

> **考试要点**：**"连接超时但 Flow Logs 中完全查不到相关记录" 通常指向路由问题（如 [[Route Table]] 缺少必要路由），而非安全组/NACL 拦截**——因为只要流量实际到达了 ENI（无论被接受还是拒绝）都会产生记录，完全没有记录说明流量根本没有到达。

---

## 与其他服务的集成

| 服务 | 关系 |
|------|------|
| **[[Amazon GuardDuty]]** | Flow Logs 是 GuardDuty **基础威胁检测**的关键数据源之一，用于识别与已知恶意 IP 的通信、端口扫描等网络层异常，完整能力见 [[Amazon GuardDuty]] 独立笔记 |
| **[[AWS Organizations]] 声明式策略** | 可在组织级别**强制要求**所有账户的 VPC 启用 Flow Logs，确保安全基线不因个别账户遗漏配置而出现盲区，完整机制见 [[AWS Organizations]] 独立笔记 |
| **CloudWatch Logs Insights** | 对投递到 CloudWatch 的 Flow Logs 做交互式查询，快速定位特定 IP/端口的流量模式 |
| **Amazon Athena** | 对投递到 S3 的 Flow Logs 做 SQL 查询，适合大规模历史数据分析 |

---

## 限制与注意事项

- **无法追溯启用之前的流量**：Flow Logs 只记录**启用之后**产生的流量，不能事后补录历史数据
- **不保证捕获每一个数据包**：在**极高流量**场景下，AWS 可能对记录做**采样**处理（`log-status` 显示为 `SKIPDATA` 时表示该窗口内部分数据被跳过）
- **修改安全组/NACL 后可能存在短暂延迟**：流日志的 `action` 反映的是记录当时生效的规则，规则变更后的短时间窗口内数据可能未完全同步

---

## 典型应用场景

| 场景 | 推荐配置 |
|------|---------|
| **实例连接失败，需要判断是安全组/NACL 拦截还是路由问题** | 查看 Flow Logs：有 REJECT 记录 → 检查 SG/NACL；无任何记录 → 检查路由表 |
| **需要长期低成本保留流日志用于合规审计** | 投递到 [[S3]]，结合 Athena 做历史查询 |
| **需要实时查询流量模式并设置告警** | 投递到 [[CloudWatch]] Logs，结合 Logs Insights |
| **需要检测与已知恶意 IP 的通信** | 启用 Flow Logs 作为 [[Amazon GuardDuty]] 的数据源 |
| **需要检查数据包实际内容/做深度协议分析** | 改用 [[VPC Traffic Mirroring]]（Flow Logs 无法满足） |
| **组织级强制所有账户启用 Flow Logs** | [[AWS Organizations]] 声明式策略 |
| **需要追踪 TCP 连接状态（SYN/ACK/FIN）** | 使用自定义格式，纳入 tcp-flags 字段 |

---

## 考试重点总结

### SAA-C02 高频考点

1. **Flow Logs 记录的是流量元数据，不是数据包内容**：需要 payload 检测应改用 [[VPC Traffic Mirroring]]
2. **三种捕获级别**：VPC/子网/ENI，本质都是逐 ENI 生成记录
3. **`action` 字段反映 SG/NACL 的放行结果**：REJECT 记录是排查访问控制拦截的第一手资料
4. **完全没有记录通常指向路由问题**：流量根本没到达 ENI，而非被拦截
5. **默认字段不含 tcp-flags 等信息**：需要时应使用自定义格式
6. **不捕获 DNS 查询、实例元数据访问、Windows 许可证激活等特定流量**：这是预期行为而非故障
7. **可投递到 CloudWatch Logs、S3 或 Kinesis Data Firehose**：分别对应实时查询、长期归档、流式处理三类场景
8. **是 GuardDuty 基础威胁检测的关键数据源**：用于识别网络层异常
9. **不能追溯启用前的历史流量**：只记录启用之后产生的流量
10. **高流量场景可能被采样（SKIPDATA）**：不保证 100% 捕获每个数据包

### 场景题解题思路

```
场景分析 → 判断 Flow Logs 相关排障/配置
├── "连接失败，需判断是安全组/NACL 拦截还是路由问题" → 查 Flow Logs：有 REJECT→查 SG/NACL；无记录→查路由表
├── "需要长期低成本归档流日志" → 投递到 S3 + Athena
├── "需要实时查询和告警" → 投递到 CloudWatch Logs + Logs Insights
├── "需要检测与恶意 IP 的通信" → Flow Logs 作为 GuardDuty 数据源
├── "需要检查数据包实际内容/深度协议分析" → 改用 [[VPC Traffic Mirroring]]
├── "日志中看不到 DNS 查询记录" → 预期行为，非故障
├── "需要追踪 TCP 连接状态" → 自定义格式 + tcp-flags 字段
└── "组织级强制启用 Flow Logs" → AWS Organizations 声明式策略
```

---

## 最佳实践

1. **默认在 VPC 级别启用 Flow Logs 作为安全基线**：低成本获得基础的网络可见性，为后续排障和审计打好基础
2. **结合 AWS Organizations 声明式策略强制组织级启用**：避免个别账户遗漏配置形成监控盲区
3. **按分析需求选择合适的目标存储**：实时排障用 CloudWatch Logs，长期审计用 S3，流式集成第三方工具用 Kinesis Data Firehose
4. **需要 TCP 状态等额外信息时使用自定义格式**：避免事后发现默认字段不够用，需要重新配置并等待新日志累积
5. **将 Flow Logs 接入 [[Amazon GuardDuty]] 增强威胁检测覆盖面**：作为基础威胁检测不可或缺的数据源
6. **排障时先确认"有无记录"再判断"REJECT 还是 ACCEPT"**：这是快速定位问题层级（路由 vs 访问控制）的关键第一步
7. **明确 Flow Logs 无法替代深度包检测工具**：涉及 payload 内容分析的需求应提前规划 [[VPC Traffic Mirroring]] 或第三方方案，而非依赖 Flow Logs
