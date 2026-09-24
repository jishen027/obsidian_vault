# AWS SAA-C03 模拟考错题本

> 记录 AWS Certified Solutions Architect - Associate (SAA-C03) 模拟考试中的错题，按**批次**（每次模拟考/每次提交给我复习的一组题目）分类，同一批次内再按**题目实际所属的 Domain**分组（模拟考平台的原始 Domain 标注如有出入，按内容重新归类，原始标注仅在有出入时于条目内注明）。**题号在不同批次间会重复**，复习时务必以"批次 + 题号"共同定位。每题格式：**题干概要 → 我的选择（错）→ 正确答案 → 核心原因 → 关联笔记**。
>
> 学习课程已完成，笔记库已有完整的服务知识笔记（见下方"相关知识笔记"），本笔记的定位与 [[AIF-C01/AIF-C01 模拟考错题本|AIF-C01 模拟考错题本]] 一致：**只记录模拟考暴露出的具体错题和薄弱点**，不重复整理服务本身的功能介绍。

---

## 考试结构速查

| 域 | 名称 | 权重 |
|----|------|------|
| **Domain 1** | 设计安全架构 (Design Secure Architectures) | 30% |
| **Domain 2** | 设计弹性架构 (Design Resilient Architectures) | 26% |
| **Domain 3** | 设计高性能架构 (Design High-Performing Architectures) | 24% |
| **Domain 4** | 设计成本优化架构 (Design Cost-Optimized Architectures) | 20% |

| 项目 | 详情 |
|------|------|
| 总题数 | 65 题（50 计分 + 15 不计分） |
| 时长 | 130 分钟 |
| 通过分数 | 720/1000（缩放评分） |
| 有效期 | 36 个月 |

> Domain 1（安全）权重最高，其次是 Domain 2（弹性）——两者合计 56%，是模拟考错题复盘时应优先关注的方向。这套权重与 AIF-C01（生成式 AI 相关 Domain 合计约 52%）的"两大权重域集中"结构类似。

---

## 目录

- [[#模拟考分数追踪]]
	- [[#目标与预约决策标准（当前，未设定具体考试日期）]]
	- [[#各 Domain 表现趋势]]
- [[#错题分布统计]]
- [[#重复出现的错题（高危薄弱点，优先复习）]]
- [[#批次一（22 题）]]
- [[#批次二（19 题，含 3 处原标注调整）]]
- [[#复习优先级建议]]

---

## 模拟考分数追踪

> **目标**：真实考试通过线是缩放分数 **720/1000**（约 72%，比 AIF-C01 的 700/1000 略高），65 题中同样有 15 题不计分且无法分辨。由于及格线更高、且批次一暴露出**四个 Domain 全部低于及格线**，本轮目标定得比 AIF-C01 更保守：**模拟考正确率需稳定在 82% 以上**（而不是 AIF-C01 的 80%）才考虑预约，给缩放分数误差、真实考场紧张情绪留出更大缓冲。

### 目标与预约决策标准（当前，未设定具体考试日期）

**总体目标**：连续 **2-3 次不同批次的模拟考稳定在 ≥ 82%**，且没有任何一个 Domain 长期低于 75%，[[#重复出现的错题（高危薄弱点，优先复习）]] 里的知识点能脱稿复述，再考虑预约真实考试日期。

**各 Domain 目标（按权重排出的优先级，已用批次一+批次二两次数据更新）**：

| Domain | 权重 | 批次一 | 批次二 | 两批次均值 | 与 82% 目标的差距 | 加权风险（差距 × 权重） | 优先级 |
|--------|------|--------|--------|-----------|-------------------|------------------------|--------|
| Design Resilient Architectures | 26% | 72% | 54% | 63% | 19 分 | **4.94（最高）** | ★★★★ |
| Design Secure Architectures | 30% | 68% | 71% | 70% | 12 分 | 3.6 | ★★★ |
| Design Cost-Optimized Architectures | 20% | 64% | 64% | 64% | 18 分 | 3.6 | ★★★ |
| Design High-Performing Architectures | 24% | 57% | 83% | 70% | 12 分 | 2.88（最低） | ★★ |

> **优先级顺序发生了明显变化**：批次一时 High-Performing 是加权风险最高的域，但批次二**大幅提升到 83%**（+26 分），把它拉到了风险最低的位置；与此同时 **Design Resilient Architectures 从 72% 骤降到 54%**（-18 分），是四个 Domain 里唯一"逆势变差"的一个，两批次均值 63% 也是最低分，加权后风险跃升为最高——**这是当前最需要警惕的信号**：不是单纯"这个域一直弱"，而是"这个域正在变得更弱"，需要优先排查具体是哪些知识点在拖后腿（详见批次二错题详解和下方复习优先级）。Secure 和 Cost-Optimized 两批次都稳定在 64%~71% 区间，风险中等，按原计划推进即可。

**预约决策表（随每批次新数据持续更新，参考 AIF-C01 的框架并调高门槛）**：

| 最近一次/最近三次平均分 | 建议 |
|---|---|
| **≥ 85%** | 距离预约已经很近，再确认一次没有 Domain 集中失分即可考虑预约 |
| **82%~84%** | 达到基本目标，但建议再跑 1-2 次批次确认稳定性（不是单次幸运高分），且逐个检查四个 Domain 是否都不低于 75% |
| **75%~81%** | 尚未达标，按上表的加权优先级顺序（High-Performing → Secure → Cost → Resilient）针对性复习，不建议预约 |
| **< 75%（批次一 66% → 批次二 70%，仍在此档）** | 差距明显，需要系统性补课，不只是刷题——建议先完整过一遍 [[SAA-C03 复习计划]] 里对应 Domain 的检查清单，再安排下一次模拟考。当前重点应放在 **Design Resilient Architectures**（两批次均值最低、且批次二明显退步），而不是已经大幅改善的 High-Performing |

**预约前最终确认清单**：

- [ ] 最近 2-3 次模拟考正确率都 ≥ 82%
- [ ] 四个 Domain 没有任何一个持续低于 75%（尤其紧盯 High-Performing 和 Secure 这两个加权风险最高的域）
- [ ] [[#重复出现的错题（高危薄弱点，优先复习）]] 中的知识点能不看笔记脱稿复述
- [ ] 模拟考是在接近真实考试的条件下完成的（限时 130 分钟、不查资料、一次做完不中途暂停）——批次一用时 2h10m 已接近上限，需要关注做题速度是否也要针对性提升

### 分数记录

| 批次 | 日期 | 用时 | 题库来源 | 总题数 | 正确数 | 正确率 | 是否达标 |
|------|------|------|---------|-------|-------|--------|---------|
| 批次一 | 2026-09-21 | 2h10m | — | 65 | 43 | **66%** | ❌ <72%，未达标 |
| 批次二 | 2026-09-23 | 2h10m | — | 65 | 46 | **70%** | ❌ <72%，未达标（但比批次一提升 4 个百分点） |

> 首次模拟考即暴露出明显差距（66% vs 72% 及格线，差 6 个百分点），且用时 2h10m 已接近 130 分钟的时长上限——时间管理也是需要关注的点，不只是知识掌握程度的问题。批次二提升到 70%，差距缩小到 2 个百分点，但**两次用时都是 2h10m，卡在时长上限附近**，说明做题速度尚未随正确率一起改善，仍是需要单独关注的风险点。

### 各 Domain 表现趋势

| Domain | 批次一（题量/正确率） | 批次二（题量/正确率） | 走势 |
|--------|----------------------|----------------------|------|
| Design High-Performing Architectures | 14 题 / 57% | 24 题 / 83% | ↑↑ 大幅改善 |
| Design Secure Architectures | 19 题 / 68% | 17 题 / 71% | ↑ 小幅改善 |
| Design Cost-Optimized Architectures | 14 题 / 64% | 11 题 / 64% | → 持平 |
| Design Resilient Architectures | 18 题 / 72% | 13 题 / 54% | ↓↓ 明显退步 |

> **⚠️ 域标注可靠性提醒**：批次二的错题里发现至少 3 道题的平台 Domain 标注与实际内容不符（如把"ASG 跨 AZ 重新平衡机制"标成 High-Performing、把"AWS Glue DataBrew 数据准备工具选型"标成 Resilient、把"Aurora Global Database vs DynamoDB Global Tables"标成 Secure），[[#错题分布统计]] 里已按内容重新归类并注明原标注，但上表的**汇总百分比仍是平台按其原始标注计算的**，可能与真实的内容维度表现有细微出入，仅供参考大方向，不必较真到个位数。
>
> **趋势解读**：批次二最大的变化是 **High-Performing 从 57% 跃升到 83%**（此前的排查/复习明显起了作用），但 **Resilient 从 72% 跌到 54%**，是唯一"变差"的域，且降幅很大，需要认真对待而不是归因于"运气不好"——尤其两批次样本量都不大（13~18 题），单次波动本身可能包含噪音，但结合[[#错题分布统计|错题分布]]看，批次二 Resilient 相关的错题相当分散（ASG 维护、S3 一致性认知、RDS 复制机制等），不是单一知识点导致，值得下一批次重点验证是否只是巧合还是真实退步。

---

## 错题分布统计

> 累计 **2 个批次、共 41 道错题**（22+19）。Domain 列原则上采用平台原始标注；批次二发现 3 道题标注与实际内容明显不符，已按内容重新归类并在括号中注明原标注（参考 AIF-C01 错题本的处理方式）。

### 批次一（共 22 题）

| 题号 | Domain | 考点主题 | 状态 |
|------|--------|---------|------|
| Q1 | Design Secure Architectures | ACM 导入证书到期监控：Config 托管规则 vs CloudWatch 指标 | 📘 |
| Q14 | Design Secure Architectures | API Gateway 内置用户管理认证：Cognito User Pools vs Identity Pools | 📘 |
| Q29 | Design Secure Architectures | 跨账户共享集中管理的 VPC：VPC 共享（子网）vs VPC Peering | 📘 |
| Q44 | Design Secure Architectures | GuardDuty 支持的数据源：VPC Flow Logs/DNS/CloudTrail | 📘 |
| Q56 | Design Secure Architectures | 加密 EBS 卷的覆盖范围（选三）：静态数据+快照+传输中数据全部加密 | 📘 |
| Q60 | Design Secure Architectures | 账户内用户级+跨账户访问控制：S3 Bucket Policy vs IAM Policy vs ACL | 📘 |
| Q2 | Design Resilient Architectures | S3 读后写一致性模型：强一致性 vs 旧版最终一致性认知 | 📘 |
| Q40 | Design Resilient Architectures | 节流/缓冲组合：API Gateway + SQS + Kinesis | 📘 |
| Q47 | Design Resilient Architectures | 需要自定义数据库主机 OS 且要高可用：RDS Custom for Oracle (Multi-AZ) | 📘 |
| Q50 | Design Resilient Architectures | PB 级跨区域一次性复制数据（选二）：S3 sync 命令 + S3 批量复制 | 📘 |
| Q53 | Design Resilient Architectures | Aurora 读写分离：Aurora Replica/reader endpoint，非"读写缓存" | 📘 |
| Q16 | Design High-Performing Architectures | 高度关联数据的复杂查询：Amazon Neptune（图数据库） | 📘 |
| Q18 | Design High-Performing Architectures | 查询 S3 历史数据并与 Redshift 数据交叉引用：Redshift Spectrum | 📘 |
| Q52 | Design High-Performing Architectures | 多消费者读取 Kinesis 吞吐不足：Enhanced Fan-Out | 📘 |
| Q61 | Design High-Performing Architectures | Lambda 最佳实践（选三）：默认在 AWS 拥有的 VPC + 需 NAT 访问公网资源 | 📘 |
| Q63 | Design High-Performing Architectures | Direct Connect 传输数据到 EFS：私有 VIF + PrivateLink 接口终端节点 | 📘 |
| Q65 | Design High-Performing Architectures | S3 流式桥接到 Kinesis 最省开发工作量：AWS DMS（S3 为源） | 📘 |
| Q25 | Design Cost-Optimized Architectures | 可再生数据的 S3 存储类别选型：One Zone-IA + 30 天最短存储期限制 | 📘 |
| Q37 | Design Cost-Optimized Architectures | 混合 On-Demand+Spot 跨实例类型：仅启动模板支持，启动配置不支持 | 📘 |
| Q39 | Design Cost-Optimized Architectures | 沿用现有服务器绑定许可迁移上云：EC2 Dedicated Hosts | 📘 |
| Q46 | Design Cost-Optimized Architectures | 全球用户加速上传大文件到 S3（选二）：S3TA + 分块上传，非 Global Accelerator | 📘 |
| Q62 | Design Cost-Optimized Architectures | 降低 Direct Connect 数据传出费用：可视化工具与数据仓库同区域部署 | 📘 |

### 批次二（共 19 题，3 处原标注调整）

| 题号 | Domain（实际内容） | 考点主题 | 状态 |
|------|-------------------|---------|------|
| Q14 | Design Secure Architectures | 按国家阻断/放行访问：WAF 地理位置匹配（ALB 场景，非 CloudFront） | 📘 |
| Q33 | Design Secure Architectures | 需审计密钥使用且不自带密钥：SSE-KMS vs SSE-S3/SSE-C | 📘 |
| Q36 | Design Secure Architectures | 限制内容仅特定国家可见（选二）：CloudFront Georestriction + Route 53 地理位置路由 | 📘 |
| Q65 | Design Secure Architectures | 从 VPC 解析本地私有 DNS：Route 53 Resolver 出站端点 | 📘 |
| Q9 | Design Resilient Architectures | ASG 内单实例维护最省资源方案（选二）：Standby 状态 + 暂停 ReplaceUnhealthy | 📘 |
| Q10 | Design Resilient Architectures（原标注：High-Performing） | ASG 跨 AZ 再平衡 vs 不健康实例替换：两种流程的先后顺序不同 | 📘 |
| Q15 | Design Resilient Architectures | 近实时告警异常 API 调用：CloudTrail→CloudWatch Logs 指标筛选器（非直连 Kinesis） | 📘 |
| Q29 | Design Resilient Architectures | 全托管、自动扩缩容的传感器数据管道：SQS + Lambda（非 EC2 轮询） | 📘 |
| Q31 | Design Resilient Architectures | 高可用+按内容路由：ALB + 跨 AZ ASG（非 NLB，ASG 本身不分发流量） | 📘 |
| Q32 | Design Resilient Architectures（原标注：Secure） | 部分表全球化+最小化重构：统一用 Aurora（Global Database+普通集群），不混用 DynamoDB | 📘 |
| Q58 | Design Resilient Architectures | RDS Multi-AZ 同步复制 vs 读取副本异步复制：复制方式方向不能搞反 | 📘 |
| Q7 | Design High-Performing Architectures（原标注：Resilient） | 零代码数据准备+剖析+血缘：Glue DataBrew（非 Glue Studio/Athena/AppFlow） | 📘 |
| Q40 | Design High-Performing Architectures | SQL 建模+MPP+无服务器数仓：Redshift Serverless + Redshift ML | 📘 |
| Q43 | Design High-Performing Architectures | 跨区域复制 AMI 会在目标区域同时生成新快照 | 📘 |
| Q8 | Design Cost-Optimized Architectures | 每年仅访问两次但需毫秒级检索：S3 Standard-IA（非 Intelligent-Tiering） | 📘 |
| Q18 | Design Cost-Optimized Architectures | S3 存储类别转换合法性（选二）：瀑布模型，不可逆转换识别 | 📘 |
| Q30 | Design Cost-Optimized Architectures | 24 小时即删除但被频繁引用的数据：S3 Standard（IA 最短存储期+检索费更贵） | 📘 |
| Q46 | Design Cost-Optimized Architectures（原标注：High-Performing） | S3TA 未产生加速效果时的计费：完全不收费（含标准传入流量本就免费） | 📘 |
| Q47 | Design Cost-Optimized Architectures | NFS 兼容+自动分层+最省成本迁移：Storage Gateway 文件网关（非卷网关/EFS/FSx） | 📘 |

---

## 重复出现的错题（高危薄弱点，优先复习）

> AIF-C01 备考的经验是：**同一题库换讲师/换课程后依然出错的知识点，才是真正没有内化的薄弱区**，优先级高于只错一次的题目。批次二开始出现跨批次重复，下表持续更新。

> **🚩 最高优先级：S3 存储类别选型规则，两个批次里已经出错 4 次**（批次一 Q25、批次二 Q8/Q18/Q30），是目前唯一在单个批次内就重复出错 3 次、累计跨批次出错 4 次的知识点——说明"存储类别怎么选"这套规则体系（最短存储期限、检索费、访问模式已知/未知、合法转换路径）还没有真正建立起完整的心智模型，零散记忆单个事实容易在换个问法后又出错。

| 题目/主题 | 出现方式 | 反复出错的核心症结 | 关联笔记 |
|-----------|---------|-------------------|---------|
| **🚩 S3 存储类别选型（4 次）** | 批次一 Q25（可再生数据+30天限制）→ 批次二 Q8（已知低频访问选 IA 还是 IT）→ 批次二 Q18（哪些转换不合法）→ 批次二 Q30（短期高频引用数据该用 Standard） | 四次错误覆盖了存储类别决策的四个不同侧面（最短存储期限、IA vs Intelligent-Tiering 的适用边界、合法转换路径、检索费与短生命周期数据的组合效应），说明目前是**逐条记忆孤立规则**，没有建立起"总拥有成本 = 存储单价 + 检索费 + 是否触发最短期限罚金"这套统一的判断框架 | [[S3#生命周期转换的"瀑布模型"：哪些转换合法（考试高频，多选题常考）]] \| [[S3#短期/高频引用数据的存储类别陷阱：min duration + 检索费的组合（考试高频）]] \| [[S3#Standard-IA vs Intelligent-Tiering：访问模式"已知稳定"还是"未知/多变"]] |

---

## 批次一（22 题）

### Batch 1 初步观察

> 首次模拟考 66%，四个 Domain 全部低于 72% 及格线。22 道错题里，**至少 12 道的核心症结是"混淆两个功能相近但适用场景不同的 AWS 服务/方案"**（如 User Pools vs Identity Pools、VPC 共享 vs Peering、RDS vs RDS Custom、Redshift Spectrum vs Athena、DataSync vs S3+Lambda、Global Accelerator vs S3TA），这与 AIF-C01 备考初期的模式高度相似——说明"先把每组相近服务的边界打清楚"应该是接下来复习的第一优先级，而不是急于刷更多新题。

---

### Design Secure Architectures

#### 批次一 · Q1 - ACM 导入证书到期监控 📘

- **题干**：安全团队希望在第三方 SSL/TLS 证书（导入到 ACM）到期前 30 天收到通知，要求配置和维护成本最低
- **我的选择（错）**：AWS Config 托管规则检查**由 ACM 签发**的证书是否即将到期
- **正确答案**：AWS Config 托管规则检查**导入到 ACM 的第三方证书**是否即将到期，触发 SNS 通知
- **核心原因**：**ACM 只自动续期自己签发的证书，不会自动续期导入的第三方证书**——所以需要监控的对象恰恰是"导入的证书"，而不是 ACM 自己签发、本来就会自动续期的证书；用 CloudWatch `DaysToExpiry` 指标+告警也能做到，但配置工作量明显高于开箱即用的 Config 托管规则
- **关联笔记**：[[AWS Certificate Manager#导入的第三方证书（考试易混淆点：ACM 不会自动续期它）]]

---

#### 批次一 · Q14 - API Gateway 内置用户管理认证 📘

- **题干**：需要为 API Gateway 的 API 调用做授权，公司希望方案提供**内置的用户管理**能力
- **我的选择（错）**：Amazon Cognito Identity Pools
- **正确答案**：Amazon Cognito **User Pools**
- **核心原因**：User Pools 是用户目录，提供注册/登录/密码找回/MFA 等**内置用户管理**能力；Identity Pools 只是把已验证身份换成**临时 AWS 凭证**用于访问 AWS 资源，本身不是认证机制、也不提供用户管理
- **关联笔记**：[[Cognito]]

---

#### 批次一 · Q29 - 跨账户共享集中管理的 VPC 📘

- **题干**：公司用 Organizations 管理多个部门账户，希望为需要高度互联互通的应用提供**共享、集中管理**的 VPC
- **我的选择（错）**：用 VPC Peering 共享一个或多个子网
- **正确答案**：用 **VPC 共享（VPC Sharing）** 共享一个或多个子网
- **核心原因**：VPC 共享（基于 AWS RAM）让 Organizations 下的多个账户在**同一个 VPC 的共享子网**里创建资源，由 owner 账户集中管理网络；VPC Peering 只是连接**两个独立的 VPC**，不提供集中管理能力，而且无论哪种方式，**账户所有者都不能把整个 VPC 作为一个整体共享出去**——只能共享子网
- **关联笔记**：[[VPC#VPC 共享（VPC Sharing，考试易混淆点）]] | [[VPC Peering]]

---

#### 批次一 · Q44 - GuardDuty 支持的数据源 📘

- **题干**：想用 GuardDuty 做威胁检测，GuardDuty 支持哪些数据源？
- **我的选择（错）**：VPC Flow Logs、Amazon API Gateway 日志、S3 访问日志
- **正确答案**：**VPC Flow Logs、DNS 日志、AWS CloudTrail 事件**
- **核心原因**：GuardDuty 的三大核心数据源是固定的——VPC Flow Logs（网络层异常）、DNS 日志（恶意域名/C2 通信）、CloudTrail 事件（账户级异常 API 调用）；API Gateway 日志、S3 访问日志、ELB 日志、CloudFront 日志都**不是** GuardDuty 直接分析的数据源，是常见干扰项
- **关联笔记**：[[Amazon GuardDuty#数据源与分析维度]]

---

#### 批次一 · Q56 - 加密 EBS 卷的覆盖范围（多选，选三）📘

- **题干**：加密的 EBS 卷具备哪些能力？
- **我的选择**：数据静态加密（对）+ 快照加密（对）+ 卷与实例间传输**不**加密（错）
- **正确答案**：静态数据加密 + 快照加密 + **卷与实例间传输的数据也加密**
- **核心原因**：加密 EBS 卷的三个维度是**全有或全无**——静态数据、快照（及由快照创建的新卷）、卷与实例间的传输数据，一旦启用加密**三者同时生效**，不存在"只加密其中一部分"的情况
- **关联笔记**：[[EBS#加密卷的三重覆盖范围（考试高频，多选题常考）]]

---

#### 批次一 · Q60 - 账户内用户级+跨账户访问控制 📘

- **题干**：需要为同账户内特定用户和跨账户请求都提供 S3 访问权限控制，该用什么机制？
- **我的选择（错）**：IAM 策略
- **正确答案**：**S3 Bucket Policy**
- **核心原因**：Bucket Policy 既能授权**本账户内的特定用户**，也能授权**其他 AWS 账户**访问；IAM 策略只能管理**本账户内**的用户权限，无法直接对外部账户授权；ACL 只能对整个外部账户授权（无法精确到用户级别）；Security Group 不适用于 S3
- **关联笔记**：[[S3 Security]]

---

### Design Resilient Architectures

#### 批次一 · Q2 - S3 读后写一致性模型 📘

- **题干**：进程覆盖写入一个已存在的对象后立即读取，S3 会返回什么？
- **我的选择（错）**：在变更完全传播之前，可能返回新数据（暗示存在不确定的传播窗口）
- **正确答案**：**S3 始终返回最新版本的对象**（强一致性）
- **核心原因**：S3 自 2020 年底起对所有 GET/PUT/LIST 操作提供**强读后写一致性**，不存在"变更尚未传播、可能读到旧数据"的窗口期——这是已经改变的行为，若脑子里还停留在"S3 是最终一致性"的旧认知，这类题会全部答错
- **关联笔记**：[[S3#考试重点总结]]

---

#### 批次一 · Q40 - 节流/缓冲组合方案 📘

- **题干**：需要在流量突增时对请求做节流或缓冲，该用哪组服务？
- **我的选择（错）**：ELB + SQS + Lambda
- **正确答案**：**API Gateway + SQS + Kinesis**
- **核心原因**：API Gateway 用令牌桶算法原生支持**节流**；SQS 和 Kinesis 都具备**缓冲**能力，可以平滑流量尖峰；ELB **不具备节流能力**；Lambda 达到并发上限时是直接以 429 错误拒绝请求（属于被节流的对象，而非主动节流/缓冲的工具）
- **关联笔记**：[[Amazon API Gateway#请求节流（Throttling，考试易漏点）]]

---

#### 批次一 · Q47 - 需要自定义数据库主机 OS 且要高可用 📘

- **题干**：公司需要对 Oracle 数据库主机操作系统做特殊定制，同时希望提高数据库层的可用性，且尽量减少底层基础设施运维工作量
- **我的选择（错）**：标准 RDS for Oracle 的 Multi-AZ 配置
- **正确答案**：**RDS Custom for Oracle 的 Multi-AZ 配置**
- **核心原因**：标准 RDS **不允许访问数据库主机操作系统**，无论是否开启 Multi-AZ 都无法满足"自定义 OS"的需求；只有 **RDS Custom** 在托管服务基础上开放了主机/OS 的访问和自定义能力，同时仍支持 Multi-AZ 高可用；自建 EC2 虽然能自定义，但运维工作量远高于 RDS Custom，不满足"最小化运维工作量"的要求
- **关联笔记**：[[RDS#RDS Custom（考试易混淆点：标准 RDS 不允许访问底层主机）]]

---

#### 批次一 · Q50 - PB 级跨区域一次性复制数据（多选，选二）📘

- **题干**：已用 Direct Connect 把 1PB 数据传到 us-west-1 的 S3，现需要一次性复制到 us-east-1 的另一个 S3 桶，且现场不允许使用 Snowball
- **我的选择**：S3 批量复制（对）+ 用 S3 控制台手动复制粘贴（错）
- **正确答案**：`aws s3 sync` 命令 + **S3 批量复制（Batch Replication）**
- **核心原因**：S3 控制台的复制粘贴功能**不适合 PB 级数据量**，只适合少量数据的临时操作；`aws s3 sync` 命令和 S3 批量复制都是专为**批量、一次性**的跨桶复制设计的机制（批量复制做完后可删除复制配置，确保只生效一次）；Snowball 已被明确排除；S3 Transfer Acceleration 不能用于跨桶复制这类场景
- **关联笔记**：[[S3]]

---

#### 批次一 · Q53 - Aurora 读写分离 📘

- **题干**：Aurora Multi-AZ 部署中，数据库读操作导致高 I/O 并增加了写请求的延迟，如何分离读写请求？
- **我的选择（错）**：在 Aurora 数据库上启用读透缓存（read-through caching）
- **正确答案**：**设置读取副本（Aurora Replica），应用改用 reader endpoint**
- **核心原因**：Aurora Replica 连接同一份共享存储卷，专门服务**只读**请求，通过 reader endpoint 自动负载均衡到多个副本，从而把读负载从主实例（writer）分离出去；**Aurora 本身没有内置的读透缓存能力**，这个选项纯属干扰项——要做缓存需要额外引入 ElastiCache 并修改应用代码
- **关联笔记**：[[Aurora]]

---

### Design High-Performing Architectures

#### 批次一 · Q16 - 高度关联数据的复杂查询 📘

- **题干**：需要处理"用户 A 的好友们发布的视频获得了多少点赞"这类高度关联的复杂查询，该用哪个数据库？
- **我的选择（错）**：Amazon Aurora
- **正确答案**：**Amazon Neptune**
- **核心原因**：Neptune 是专为存储和查询**高度关联数据集**设计的图数据库，能以毫秒级延迟查询数十亿级关系，天然适合社交网络这类"好友的好友"式多跳查询；Aurora 是关系型数据库，不是为图遍历查询优化的；OpenSearch 面向搜索/日志分析，Redshift 面向数据仓库分析，都不是这个场景的最佳解
- **关联笔记**：[[Amazon Neptune]]

---

#### 批次一 · Q18 - 查询 S3 历史数据并与 Redshift 数据交叉引用 📘

- **题干**：公司想把 Redshift 中一年以上的历史数据迁到 S3 降低成本，但分析师仍需要能交叉引用这些历史数据和每日报表，要求最少工作量和最低成本
- **我的选择（错）**：通过 Amazon Athena 访问历史数据，Redshift 继续做每日报表，需要交叉引用时导出成文件再分析
- **正确答案**：**使用 Redshift Spectrum**，在 Redshift 集群里创建指向 S3 历史数据的外部表，直接与每日报表数据交叉查询
- **核心原因**：Redshift Spectrum 无需把数据导回 Redshift 集群就能直接查询 S3 数据，且能在**同一个查询引擎**里与现有 Redshift 表做交叉引用；Athena 方案虽然也能查 S3，但和 Redshift 报表是两条独立的查询路径，交叉引用需要手动导出文件再分析，运维繁琐；用 COPY 或 Glue ETL 把数据重新加载回 Redshift 只为了一次性查询，成本明显更高
- **关联笔记**：[[Redshift#S3 集成 - Redshift Spectrum]]

---

#### 批次一 · Q52 - 多消费者读取 Kinesis 吞吐不足 📘

- **题干**：多个消费者应用并行读取同一个 Kinesis Data Streams，工程师发现生产者和消费者之间存在性能延迟，如何改善？
- **我的选择（错）**：把 Kinesis Data Streams 换成 SQS FIFO 队列
- **正确答案**：**启用 Kinesis Data Streams 的 Enhanced Fan-Out（增强扇出）**
- **核心原因**：默认情况下，每个分片 2MB/秒的读取吞吐由**所有消费者共享**，多消费者并行读取时会互相竞争带宽；启用 Enhanced Fan-Out 后每个注册的消费者独享每个分片 2MB/秒的专属吞吐通道；换成 SQS FIFO/Standard 都不适合"多个应用并行消费同一份数据流"这种场景，且 SQS 消息一旦被消费就会从队列移除，不支持多消费者独立、重复读取同一份数据
- **关联笔记**：[[Amazon Kinesis#消费模式：共享吞吐 vs 增强扇出]]

---

#### 批次一 · Q61 - Lambda 最佳实践（多选，选三）📘

- **题干**：以下哪些是使用 Lambda 构建无服务器架构时的正确考虑点？
- **我的选择**：CloudWatch Alarm 监控并发/调用量指标（对）+ 用 Lambda Layer 复用代码（对）+ "建议过度配置函数超时设置以保证性能"（错）
- **正确答案**：以上两个对的选项 + **"Lambda 默认在 AWS 拥有的 VPC 中运行，可直接访问公网/公共 AWS API；一旦启用 VPC，需要通过公有子网中的 NAT 网关才能访问公网资源"**
- **核心原因**：Lambda 函数默认运行在 AWS 自己管理的 VPC 里，天然能访问公网和公共 AWS API；只有需要访问**私有子网内的资源**（如 RDS 实例）时才需要启用 VPC 配置，一旦启用，所有网络流量都要遵循该 VPC 的路由规则，访问公网资源就需要经过 NAT 网关；**我选错的那个选项本身是错误陈述**——AWS 实际建议**不要**过度配置函数超时时间（会导致意外的运行时长和费用），而是应该基于代码实际性能设置合理超时
- **关联笔记**：[[AWS Lambda]]

---

#### 批次一 · Q63 - Direct Connect 传输数据到 EFS 📘

- **题干**：本地服务器每天产生大量视频文件写入 NFS 文件系统，需要在迁移前把新增文件持续复制到 Amazon EFS，要求最具运营效率
- **我的选择（错）**：DataSync agent → 通过公有 VIF 传到 S3 → Lambda 处理事件通知，把文件从 S3 复制到 EFS
- **正确答案**：DataSync agent → 通过**私有 VIF** 连接到 **EFS 的 PrivateLink 接口 VPC 终端节点** → DataSync 每 24 小时执行一次计划任务，直接同步到 EFS
- **核心原因**：DataSync 原生支持**直接**把数据同步到 EFS，走私有 VIF + PrivateLink 接口终端节点即可一步到位；先经过 S3 再用 Lambda 转存到 EFS 是多余的中间环节，运营效率明显更低；VPC Peering 无法用来打通 Direct Connect 的流量到 EFS 端点，S3 网关终端节点也不能用于访问 EFS
- **关联笔记**：[[AWS DataSync#安全性]]

---

#### 批次一 · Q65 - S3 流式桥接到 Kinesis 最省开发工作量 📘

- **题干**：公司想把已有和持续更新的 S3 数据流式传输到 Kinesis Data Streams，要求以最快方式实现
- **我的选择（错）**：配置 S3 的 EventBridge 事件，触发 Lambda 函数把数据发送到 Kinesis
- **正确答案**：使用 **AWS DMS 作为 S3 和 Kinesis Data Streams 之间的桥梁**
- **核心原因**：DMS 原生支持把 **S3 作为源、Kinesis Data Streams 作为目标**，开箱即用完成全量+增量数据的流式迁移，不需要写任何自定义代码；EventBridge/S3 事件通知 + Lambda 的方案需要大量自定义开发去处理数据格式转换和写入 Kinesis 的逻辑，明显不是"最快"的方案；S3 不能直接写入 SNS，SNS 也不能直接发送消息给 Kinesis Data Streams
- **关联笔记**：[[Database Migration Service#灵活的源/目标端点]]

---

### Design Cost-Optimized Architectures

#### 批次一 · Q25 - 可再生数据的 S3 存储类别选型 📘

- **题干**：媒体资产前几天访问频繁、一周后访问骤降但仍需偶尔立即访问，数据可再生，希望降低存储成本
- **我的选择（错）**：7 天后转到 S3 Standard-IA
- **正确答案**：**30 天后转到 S3 One Zone-IA**
- **核心原因**：从 S3 Standard 转到任何 IA 类别都有**最短 30 天存储期限制**，7 天后转换不满足这个前提；数据本身是**可再生的**，不需要 Standard-IA 跨多可用区的冗余保障，One Zone-IA 单可用区存储成本比 Standard-IA 低 20%，更适合这类数据
- **关联笔记**：[[S3]]

---

#### 批次一 · Q37 - 混合 On-Demand+Spot 跨实例类型 📘

- **题干**：希望 Auto Scaling Group 混合使用多种实例类型的 On-Demand 和 Spot 实例，该用启动配置还是启动模板？
- **我的选择（错）**：启动配置或启动模板都可以
- **正确答案**：**只能用启动模板（Launch Template）**
- **核心原因**：启动模板支持跨多种实例类型、混合 On-Demand 和 Spot 采购方式（Mixed Instances Policy）；启动配置（Launch Configuration）从设计上就不支持这个能力，无论怎么配置都做不到
- **关联笔记**：[[Auto Scaling#核心组件]]

---

#### 批次一 · Q39 - 沿用现有服务器绑定许可迁移上云 📘

- **题干**：公司有多个长期的服务器绑定型软件许可，想在迁移上云后继续沿用这些许可，要求最具成本效益
- **我的选择（错）**：EC2 Reserved Instance
- **正确答案**：**EC2 Dedicated Hosts（专用主机）**
- **核心原因**：只有 Dedicated Hosts 让你完整掌控实例在**同一台物理服务器**上的放置，满足按物理核心/插槽计费的服务器绑定型许可的合规要求；Dedicated Instance 虽然硬件隔离，但不保证长期绑定同一台物理服务器，不满足许可合规；On-Demand 和 Reserved Instance 都不解决许可绑定的问题，只是不同的计费模式
- **关联笔记**：[[EC2#实例租赁类型：Dedicated Host vs Dedicated Instance（考试易混淆点）]]

---

#### 批次一 · Q46 - 全球用户加速上传大文件到 S3（多选，选二）📘

- **题干**：海外分支机构反馈上传大视频文件到 S3 存在明显延迟，如何以最具成本效益的方式提升上传速度？
- **我的选择**：分块上传（对）+ 用 AWS Global Accelerator 加速上传（错）
- **正确答案**：**S3 Transfer Acceleration（S3TA）** + 分块上传
- **核心原因**：S3TA 利用 CloudFront 的全球边缘位置把上传流量导入 AWS 优化网络路径，专为加速**向 S3 上传**设计；**Global Accelerator 面向的是计算类端点（ALB/NLB/EC2），不用于加速 S3 上传**——这是两者最容易被搞混的点；分块上传通过并行传输多个分片提升大文件的上传吞吐；Direct Connect/Site-to-Site VPN 都需要数月部署周期，对这个需求是"杀鸡用牛刀"
- **关联笔记**：[[AWS Global Accelerator#与相似服务的边界（考试高频对比）]]

---

#### 批次一 · Q62 - 降低 Direct Connect 数据传出费用 📘

- **题干**：数据仓库和可视化工具之间通过 Direct Connect 连接，查询响应平均 60MB，网页本身 600KB，如何让数据传出（DTO）费用最低？
- **我的选择（错）**：可视化工具部署在本地，通过公网在同区域查询数据仓库
- **正确答案**：**可视化工具和数据仓库部署在同一 AWS 区域，通过 Direct Connect 访问可视化工具本身**
- **核心原因**：DTO 费用按"离开 AWS 传输到本地"的数据量计费——如果可视化工具和数据仓库都在 AWS 区域内，只有 600KB 的网页需要经 Direct Connect 传出到本地，而不是每次查询产生的 60MB 响应；把可视化工具放在本地，则每次查询的 60MB 响应都要产生 DTO 费用；互联网传输费率也普遍高于 Direct Connect，所以本地+互联网的组合是最差选项
- **关联笔记**：[[Direct Connect]]

---

## 批次二（19 题，含 3 处原标注调整）

### Batch 2 复盘

> 批次二 70%（46/65），比批次一提升 4 个百分点，但仍未达到 72% 及格线。最大的发现是 **S3 存储类别选型这个知识点已经跨批次出错 4 次**（本批次单批次内就错了 3 次：Q8、Q18、Q30），正式升级为最高优先级的重复错题，详见 [[#重复出现的错题（高危薄弱点，优先复习）]]。此外，本批次里 **Design Resilient Architectures 相关的错题明显增多且分散**（Q9/Q10/Q15/Q29/Q31/Q32/Q58 共 7 题），但每题考的具体知识点都不同（ASG 维护流程、AZ 再平衡顺序、CloudTrail 告警链路、无服务器管道选型、ALB+ASG 高可用组合、全球数据库架构、RDS 复制方式），属于"广撒网式"的知识空白，而不是单一薄弱点——这与批次二 Resilient 域正确率骤降到 54% 是吻合的。

**次要模式一：题目里出现"悄悄混入自己管理的计算资源"的干扰项**——Q29（SQS+Lambda 而非 EC2 轮询）在考"识别出方案里其实藏了一个需要自己管理的 EC2/自建集群"，只要选项出现"运行在 EC2 上的应用"轮询队列/数据流，就不满足"完全无服务器"的要求，即便其余设计看起来更灵活。

**次要模式二：不能只看"能不能用 SQL 做机器学习"，还要看是否满足题干的其他硬性约束**——Q40 需要同时满足"加载到数据仓库" + "MPP" + "SQL 建模" + "无服务器"四个条件，Athena ML 方案虽然也能"用 SQL 做预测"，但**根本没有把数据加载进任何数仓**，直接违反了题干的第一条要求，且 Athena 不是 MPP 数仓；只有 Redshift Serverless + Redshift ML 同时满足全部四个条件。

**其余题目**：各自独立的知识空白（WAF 地理位置匹配、SSE-KMS 审计、Route 53 Resolver 出站端点、AMI 跨区域复制机制、Storage Gateway 文件网关），已逐一补充到对应笔记。

---

### Design Secure Architectures

#### 批次二 · Q14 - 按国家阻断/放行访问 📘

- **题干**：应用部署在 ALB 后面的 EC2 实例上，因新法规要求只允许公司所在国家访问，需阻断其他两个国家
- **我的选择（错）**：在 VPC 中使用 CloudFront 的 Geo Restriction 功能
- **正确答案**：**在 ALB 所在的 VPC 上配置 AWS WAF**（地理位置匹配规则）
- **核心原因**：CloudFront 的 Geo Restriction **只能附加到 CloudFront 分发**，而这里的入口是 ALB，不是 CloudFront——"在 VPC 中使用 CloudFront"这个描述本身就自相矛盾（CloudFront 运行在边缘节点，不属于任何 VPC）；WAF 的地理位置匹配规则可以直接附加到 ALB，按国家白名单/黑名单控制访问；安全组不具备基于地理位置的过滤能力
- **关联笔记**：[[AWS WAF#地理位置限制三兄弟：WAF Geo Match vs CloudFront Geo Restriction vs Route 53 Geolocation（考试易混淆点）]]

---

#### 批次二 · Q33 - 需审计密钥使用且不自带密钥 📘

- **题干**：医疗数据备份需要加密存储在 S3，公司不想提供自己的加密密钥，但需要审计追踪谁在什么时候使用了密钥
- **我的选择（错）**：客户端加密（客户提供密钥）
- **正确答案**：**SSE-KMS**（AWS KMS 管理密钥的服务端加密）
- **核心原因**：SSE-KMS 由 KMS 托管密钥，同时通过 CloudTrail 提供**每次密钥使用的审计追踪**；SSE-S3 虽然也是服务端管理，但不提供细粒度的密钥使用审计；SSE-C 和客户端加密都需要客户**自己提供密钥**，与"不想提供自己的密钥"这个要求直接矛盾
- **关联笔记**：[[S3 Security#静态加密类型对比]]

---

#### 批次二 · Q36 - 限制内容仅特定国家可见（多选，选二）📘

- **题干**：欧洲足球联赛把美国的直播分销权授予了一家公司，要求只有美国用户能观看，其他国家一律拒绝
- **我的选择**：Route 53 地理位置路由（对）+ Route 53 延迟路由（错）
- **正确答案**：Route 53 地理位置路由 + **CloudFront Georestriction（地理限制）**
- **核心原因**：题目场景是**内容分发/流媒体**，天然适合 CloudFront，其 Georestriction 功能可以直接按国家白名单/黑名单限制访问；Route 53 地理位置路由同样能"限制内容分发到有权限的地区"；延迟路由、加权路由、故障转移路由都是为了性能/可用性优化，**不能**用于按地理位置限制访问权限
- **易错点**：与批次二 Q14 是一对容易混淆的题——Q14 的入口是 ALB（该用 WAF），这题的入口是 CloudFront 流媒体分发（该用 CloudFront 自带的 Georestriction），判断依据始终是"入口资源是什么"
- **关联笔记**：[[AWS WAF#地理位置限制三兄弟：WAF Geo Match vs CloudFront Geo Restriction vs Route 53 Geolocation（考试易混淆点）]]

---

#### 批次二 · Q65 - 从 VPC 解析本地私有 DNS 📘

- **题干**：应用部署在 AWS，需要通过 Site-to-Site VPN 与本地遗留系统通信，要求应用能从 VPC 内解析本地私有 DNS 记录
- **我的选择（错）**：创建 Route 53 私有托管区域，关联到 VPC
- **正确答案**：**创建 Route 53 Resolver 出站端点**，配置转发规则把特定域名的查询转发到本地 DNS 服务器
- **核心原因**：私有托管区域只能解析**托管在 Route 53 里、AWS 侧自己创建**的域名记录，不会自动获取本地真实 DNS 服务器上的记录（除非手动复制，运维繁琐且容易不一致）；出站端点专门用于把 VPC 内对特定域名的查询**转发**到本地 DNS 服务器，是 AWS 官方推荐的混合 DNS 解析方案；入站端点方向相反（本地查询 VPC 内的 AWS 私有域名）
- **关联笔记**：[[Route 53 DNS#混合云和全球可视化]]

---

### Design Resilient Architectures

#### 批次二 · Q9 - ASG 内单实例维护最省资源方案（多选，选二）📘

- **题干**：需要对 ASG 里某个特定实例做维护，但打补丁期间健康检查会短暂失败，导致 ASG 立即替换该实例
- **我的选择**：Standby 状态（对）+ 暂停 ScheduledActions 进程（错）
- **正确答案**：Standby 状态 + **暂停 ReplaceUnhealthy 进程**
- **核心原因**：把实例放入 **Standby** 状态可以保护该实例不被处理流量、也不被当作"不健康"而被替换；暂停 **ReplaceUnhealthy** 进程类型则是从组的层面阻止"不健康即替换"这个自动化行为——两者组合是官方推荐的最省资源维护方案；ScheduledActions 只影响预定的定时伸缩动作，与"维护单个实例、避免被误判替换"完全无关
- **关联笔记**：[[Auto Scaling#暂停扩展流程（Suspend Scaling Processes）]]

---

#### 批次二 · Q10 - ASG 跨 AZ 再平衡 vs 不健康实例替换的顺序 📘（原标注：Design High-Performing Architectures）

- **题干**：手动终止了 AZ A 的两个实例导致资源分布不均衡，随后 AZ B 又检测到一个不健康实例，问 ASG 会如何处理？
- **我的选择（错）**：先创建启动新实例的伸缩活动替换不健康实例，之后再创建终止该不健康实例的活动
- **正确答案**：**不健康实例替换是先终止、再启动**；而 AZ 再平衡是**先启动新实例、再终止旧实例**——两个流程顺序相反
- **核心原因**：这是 ASG 里两套**不同触发场景**、顺序刻意设计相反的机制：不健康实例替换优先**尽快移除**故障实例（先终止后启动）；AZ 再平衡为了**不牺牲可用性/性能**，先在健康 AZ 启动新实例、确认可用后再终止旧实例——题目容易把两者的顺序张冠李戴
- **关联笔记**：[[Auto Scaling#不健康实例的处理流程（Replacement Flow）]]

---

#### 批次二 · Q15 - 近实时告警异常 API 调用模式 📘

- **题干**：日志汇总时发现有大量非法 API 调用，希望以后出现类似情况时能自动触发近实时告警
- **我的选择（错）**：配置 CloudTrail 把事件流式传输到 Kinesis，用 Kinesis 流级别指标触发 Lambda
- **正确答案**：**创建 CloudWatch 指标筛选器处理 CloudTrail 日志中的 API 调用错误码，基于该指标的速率设置告警并发送 SNS 通知**
- **核心原因**：**CloudTrail 不能直接流式传输到 Kinesis**——它的日志交付目的地只有 S3 和 CloudWatch Logs 两种，这个前提错误直接排除该选项；正确路径是先把 CloudTrail 日志送到 CloudWatch Logs，再用指标筛选器统计错误码出现频率，触发告警；Athena+QuickSight 方案是事后报表分析，不满足"近实时自动告警"；Trusted Advisor 只在触及服务配额时才告警，不针对"API 调用模式异常"
- **关联笔记**：[[CloudTrail#与 CloudWatch 集成]]

---

#### 批次二 · Q29 - 全托管、自动扩缩容的传感器数据管道 📘

- **题干**：车企想用完全无服务器、自动扩缩容的组件构建车载传感器数据摄取服务，不希望手动配置容量
- **我的选择（错）**：Kinesis Data Firehose 直接写入自动扩缩容的 DynamoDB 表
- **正确答案**：**SQS 标准队列，由 Lambda 函数批量轮询消费，写入自动扩缩容的 DynamoDB 表**
- **核心原因**：Firehose **不能直接写入 DynamoDB**（只能投递到 S3/Redshift/OpenSearch/Splunk 等目的地），这个前提本身就错误；SQS+Lambda 组合完全无服务器、自动扩缩容，符合要求；凡是方案里出现"运行在 EC2 实例上的应用"来轮询队列/数据流，都不满足"完全无服务器"的要求，即使其余部分设计合理
- **关联笔记**：[[SQS]] | [[AWS Lambda]]

---

#### 批次二 · Q31 - 高可用+按内容路由的负载均衡方案 📘

- **题干**：应用部署在多个 AZ 的 EC2 实例上，要求高可用，并且需要支持按内容路由
- **我的选择（错）**：用 ASG 分发流量到跨 AZ 的 EC2 实例，配置弹性 IP（EIP）掩盖实例故障
- **正确答案**：**用 Application Load Balancer 分发流量到跨 AZ 的 EC2 实例，配置 ASG 掩盖实例故障**
- **核心原因**：**ASG 本身不分发流量**，它只负责维持实例数量和健康状态，"用 ASG 分发流量"这个前提就是错的；按内容路由（基于路径/主机名/请求头等）是 **ALB（七层）**的能力，NLB（四层）做不到；EIP 是静态公网 IP，用于"重新映射"，不是"自动掩盖实例故障"的机制，真正负责这个职责的是 ASG 的健康检查+自动替换
- **关联笔记**：[[AWS Load Balance#ALB (Application Load Balancer)]]

---

#### 批次二 · Q32 - 部分表全球化+最小化重构 📘（原标注：Design Secure Architectures）

- **题干**：游戏公司的 games 表需要全球可访问，users 和 games_played 表只需区域级，要求最小化应用改造
- **我的选择（错）**：games 表用 DynamoDB 全球表，users/games_played 继续用 Aurora
- **正确答案**：**games 表用 Aurora Global Database，users/games_played 继续用普通 Aurora**
- **核心原因**：只要方案里引入了 DynamoDB（NoSQL），涉及该表的所有查询逻辑都要按 DynamoDB 的 API 重写，无论是否"顺便"解决了全球化需求，这就已经违反了"最小化重构"的前提；统一用 Aurora（全球表用 Aurora Global Database，区域表用普通 Aurora 集群）让应用全程面对同一套 SQL API，改造成本最低
- **关联笔记**：[[Aurora#Aurora Global Database vs DynamoDB Global Tables（考试易混淆点：不要为了"全球化"混用两种引擎）]]

---

#### 批次二 · Q58 - RDS Multi-AZ 与读取副本的复制方式 📘

- **题干**：新入职的 DevOps 工程师想了解 RDS Multi-AZ 和读取副本各自的复制能力
- **我的选择（错）**：Multi-AZ 是异步复制、跨至少两个 AZ；读取副本是同步复制，可以是 AZ 内/跨 AZ/跨区域
- **正确答案**：**Multi-AZ 是同步复制、跨至少两个 AZ；读取副本是异步复制**，可以是 AZ 内/跨 AZ/跨区域
- **核心原因**：Multi-AZ 部署创建主实例后**同步复制**到另一个 AZ 的备用实例，保证故障转移时不丢数据；读取副本用引擎原生的**异步复制**机制更新，主要用于读扩展而非零数据丢失的高可用——我的错误选择把两者的复制方式（同步/异步）完全搞反了
- **关联笔记**：[[RDS#Multi-AZ vs 读取副本（考试重点）]]

---

### Design High-Performing Architectures

#### 批次二 · Q7 - 零代码数据准备+剖析+血缘 📘（原标注：Design Resilient Architectures）

- **题干**：医疗分析公司需要零代码界面让数据工程师和业务分析师协作做数据准备，还要求数据血缘追踪、数据剖析、且能跨团队共享转换逻辑
- **我的选择（错）**：用 Glue Studio 可视化画布设计转换工作流
- **正确答案**：**AWS Glue DataBrew**
- **核心原因**：Glue Studio 底层仍然生成并运行 **Spark 脚本**，面向开发者，且**不内置数据剖析能力**；DataBrew 才是真正面向业务分析师的零代码工具，用可复用、可版本化的"Recipe"记录转换步骤（满足血缘追踪和团队共享），并内置列级数据剖析（分布、空值、异常值等）；Athena 需要写 SQL；AppFlow 面向 SaaS 应用数据集成，不是 S3 内部数据准备工具
- **关联笔记**：[[AWS Glue#AWS Glue DataBrew（真正的零代码数据准备工具，考试易混淆点）]]

---

#### 批次二 · Q40 - SQL 建模+MPP+无服务器数仓 📘

- **题干**：零售分析公司需要每日 ETL 加载数仓，还要让分析师用 SQL（而非 Python）构建机器学习模型，要求 MPP 并尽量使用无服务器服务
- **我的选择（错）**：用 Glue 作业把 S3 数据转换后注册为 Glue Data Catalog 里的 Athena 表，分析师用 Athena ML 直接在 S3 数据上做 SQL 建模和预测，不移动数据到数仓
- **正确答案**：**Glue 做 ETL，加载到 Redshift Serverless，用 Redshift ML 做 SQL 建模**
- **核心原因**：题干明确要求"把数据转换后**加载到数据仓库**"——Athena ML 方案恰恰**不把数据加载进任何数仓**，直接违反了这条硬性要求；Athena 是面向 S3 的无服务器查询引擎，不是 MPP 数据仓库，无法提供题目要求的 MPP 规模化聚合/评分能力；Redshift Serverless 才是真正满足"MPP + 无服务器数仓"的组件，配合 Redshift ML 用纯 SQL（`CREATE MODEL`）建模，同时满足数仓加载、MPP、SQL 建模、无服务器四个要求
- **易错点**：Athena ML 和 Redshift ML 都能"用 SQL 做机器学习"，容易觉得两者可以互相替代——但 Athena ML 面向的是**S3 数据的即席查询预测**，Redshift ML 面向的才是**数仓场景下的规模化建模**，题目一旦出现"加载到数据仓库"这个动作，就已经排除了 Athena 路线
- **关联笔记**：[[Redshift#Redshift Serverless（无服务器数仓，考试易漏点）]] | [[Redshift#Redshift ML（用 SQL 构建和训练机器学习模型，考试易漏点）]]

---

#### 批次二 · Q43 - AMI 跨区域复制会同时生成新快照 📘

- **题干**：在区域 A 创建实例、做快照、生成 AMI，把 AMI 复制到区域 B 并在区域 B 启动新实例，问区域 B 此时存在哪些实体？
- **我的选择（错）**：1 个 EC2 实例 + 1 个 AMI（无额外快照）
- **正确答案**：**1 个 EC2 实例 + 1 个 AMI + 1 个快照**
- **核心原因**：AMI 本质基于 EBS 快照构建，跨区域复制 AMI 时 AWS 会在目标区域**自动生成一份新的快照**（快照是区域级资源，不能跨区域共享底层存储），所以区域 B 里除了复制过去的 AMI 和新启动的实例外，还会多出一个此前没有意识到的快照
- **关联笔记**：[[EC2#AMI 生命周期]]

---

### Design Cost-Optimized Architectures

#### 批次二 · Q8 - 每年仅访问两次但需毫秒级检索 📘

- **题干**：审计部门的报告数据每年只访问两次，但需要毫秒级延迟访问，数据量达数百 TB，要求最具成本效益的存储类别
- **我的选择（错）**：S3 Intelligent-Tiering
- **正确答案**：**S3 Standard-IA**
- **核心原因**：访问频率已经明确（每年两次，稳定可预测），不需要为 Intelligent-Tiering 的自动监测/分层功能支付额外的对象监控费；Standard-IA 提供相同的高吞吐低延迟，存储单价比 Intelligent-Tiering 更省，是"访问模式已知且稳定的低频数据"的标准答案
- **关联笔记**：[[S3#Standard-IA vs Intelligent-Tiering：访问模式"已知稳定"还是"未知/多变"]]

---

#### 批次二 · Q18 - S3 存储类别转换合法性（多选，选二）📘

- **题干**：找出以下存储类别转换里**不合法**的两个
- **我的选择**：Intelligent-Tiering → Standard（对，属于不合法）+ Standard-IA → Intelligent-Tiering（错，这其实合法）
- **正确答案**：Intelligent-Tiering → Standard（不合法） + **One Zone-IA → Standard-IA（不合法）**
- **核心原因**：S3 存储类别转换遵循单向"瀑布"模型——**任何类别都不能转换回 Standard**（Standard 只能是起点）；**One Zone-IA 是瀑布链的末端之一，不能"升级"回 Standard-IA 或 Intelligent-Tiering**；而 Standard-IA → Intelligent-Tiering 和 Standard-IA → One Zone-IA 都是**合法**的转换方向，我把这个选项错判成了不合法
- **关联笔记**：[[S3#生命周期转换的"瀑布模型"：哪些转换合法（考试高频，多选题常考）]]

---

#### 批次二 · Q30 - 24 小时即删除但被频繁引用的数据 📘

- **题干**：视频流媒体公司的数据湖有一个暂存区，中间查询结果只保留 24 小时，但被分析管道的其他部分频繁引用
- **我的选择（错）**：S3 One Zone-IA
- **正确答案**：**S3 Standard**
- **核心原因**：Standard-IA 和 One Zone-IA 都有 **30 天最短存储期限**，24 小时就删除的数据仍要按 30 天计费，相当于变相多付了 29 天的存储费；两者还都有**检索费**，而这份数据会被"频繁引用"，检索费会进一步推高成本；S3 Standard 没有最短存储期限制、也没有检索费，综合下来反而是这个场景最省钱的选项——不能只看"访问不算频繁"就反射性选 IA 类别
- **关联笔记**：[[S3#短期/高频引用数据的存储类别陷阱：min duration + 检索费的组合（考试高频）]]

---

#### 批次二 · Q46 - S3TA 未产生加速效果时如何计费 📘（原标注：Design High-Performing Architectures）

- **题干**：科学家用 S3TA 上传 3GB 图像，结果 S3TA 并未产生加速效果，问这次上传该如何收费？
- **我的选择（错）**：仍需支付 S3 标准传输费用
- **正确答案**：**完全不需要支付任何传输费用**
- **核心原因**：S3TA 只在**实际产生加速效果**时才收取加速费用，本次未加速所以不产生 S3TA 费用；同时，从公网向 S3 上传数据（数据传入）本身**从来不收费**，与是否使用加速无关——两个"不收费"的原因叠加，答案是"完全免费"，而不是"至少要付基础传输费"
- **关联笔记**：[[CloudFront#S3 Transfer Acceleration（易混淆点）]]

---

#### 批次二 · Q47 - NFS 兼容+自动分层+最省成本迁移 📘

- **题干**：本地 NFS 存储难以扩展，希望迁移到云端方案，要求保留 NFS 兼容性、支持自动分层到低成本存储、且最具成本效益
- **我的选择（错）**：Storage Gateway Volume Gateway 缓存模式，挂载为块设备后再挂 NFS，快照存 Glacier Deep Archive
- **正确答案**：**Storage Gateway File Gateway**，向工作负载呈现 NFS 兼容的文件共享，文件存储在 S3，用 S3 生命周期策略自动分层
- **核心原因**：File Gateway **原生**提供 NFS 兼容接口，文件直接以对象形式存入 S3，可以用标准的 S3 Lifecycle 策略自动转到低成本存储层级；Volume Gateway 提供的是 **iSCSI 块存储**，要在块设备上层再挂载 NFS 是多此一举的额外封装，且不支持对象级别的 S3 生命周期分层；FSx for Windows 面向 **SMB** 协议，不是 NFS，还需要应用改用 SMB 协议访问文件
- **关联笔记**：[[AWS Storage Gateway#三种网关模式（考试高频）]]

---

## 复习优先级建议

```
按易错模式归类复习（首批次，会随后续批次持续更新）
├── "混淆两个功能相近但适用场景不同的服务/方案"（本批次最集中的模式，12/22 题属于此类）
│   Q14（Cognito User Pools vs Identity Pools）、Q29（VPC 共享 vs VPC Peering）、
│   Q47（RDS vs RDS Custom）、Q18（Redshift Spectrum vs Athena）、
│   Q63（DataSync 直连 EFS vs 绕道 S3+Lambda）、Q46（S3TA vs Global Accelerator）、
│   Q39（Dedicated Host vs Dedicated Instance vs Reserved Instance）、
│   Q37（启动模板 vs 启动配置）、Q53（Aurora Replica vs 不存在的"读透缓存"）
├── "旧版行为认知没有更新"
│   Q2（S3 已是强一致性，不再是最终一致性）
├── "纯记忆型的固定事实清单，不能靠推理"
│   Q44（GuardDuty 三大数据源：VPC Flow Logs/DNS/CloudTrail，不含 API Gateway/S3 日志）、
│   Q56（加密 EBS 卷三个维度全有或全无）
├── "多方案组合题，需要同时判断每个服务各自的能力边界"
│   Q40（API Gateway 节流 + SQS/Kinesis 缓冲，ELB 不节流）、
│   Q61（Lambda 默认在 AWS VPC + 需 NAT 访问公网，不要过度配置超时）
├── "成本优化的判断依据是具体的计费机制，不是直觉"
│   Q62（DTO 按传出量计费，同区域部署减少传出数据量）、
│   Q25（IA 类别转换有 30 天最短存储期限制）
├── "规模效应题：小规模用简单方案，大规模/PB 级换专用批量工具"
│   Q50（PB 级用 sync/批量复制，不用控制台手动复制）
├── "🚩 S3 存储类别选型规则体系，跨批次反复出错（4 次，最高优先级）"
│   批次一 Q25 + 批次二 Q8/Q18/Q30——最短存储期限、检索费、已知/未知访问模式、合法转换路径这四个维度要放在同一个框架里判断，不能孤立记忆
├── "先看流量入口是什么资源，再决定用哪个安全/地理位置控制方案"
│   批次二 Q14（ALB 入口用 WAF，不是 CloudFront）、Q36（CloudFront 入口用 Georestriction）
├── "题目描述的方案里一旦出现自建 EC2/自管理集群，就不是'完全无服务器'"
│   批次二 Q29（SQS+Lambda 而非 EC2 轮询）
├── "能用 SQL 做 ML 不代表满足所有要求，还要逐条核对题干的硬性约束（如是否真的加载进了数仓）"
│   批次二 Q40（Athena ML 没有加载进数仓，也不是 MPP 数仓，Redshift Serverless+ML 才同时满足全部条件）
├── "ASG 内两套顺序相反的机制不要混淆：不健康替换先终止后启动，AZ 再平衡先启动后终止"
│   批次二 Q10
├── "跨引擎/跨技术栈"混用"看似能凑效果，但违背'最小化重构'的要求"
│   批次二 Q32（Aurora vs 混用 DynamoDB）、Q47（File Gateway 原生 NFS+S3 分层，vs Volume Gateway 多此一举的封装）
├── "组件本身的核心职责不能张冠李戴：ASG 不分发流量、CloudTrail 不能直连 Kinesis、Glue Studio 不做数据剖析"
│   批次二 Q31（ALB 分发+ASG 容错）、Q15（CloudTrail→CloudWatch Logs，非 Kinesis）、Q7（DataBrew vs Glue Studio）
└── "免费/收费边界要看清楚触发条件，不要想当然多付钱"
    批次二 Q46（S3TA 未加速则完全免费，不是仍收标准传输费）
```


---

## 相关知识笔记（已有课程笔记，按类别索引）

| 类别 | 笔记 |
|------|------|
| 计算服务 | [[AWS SAA-C02/计算服务/EC2\|EC2]] \| [[AWS SAA-C02/计算服务/Auto Scaling\|Auto Scaling]] \| [[AWS SAA-C02/计算服务/AWS Load Balance\|ELB]] \| [[AWS SAA-C02/计算服务/ECS\|ECS]] \| [[AWS SAA-C02/计算服务/EKS\|EKS]] \| [[AWS SAA-C02/计算服务/AWS Fargate\|Fargate]] \| [[AWS SAA-C02/计算服务/AWS Lambda\|Lambda]] \| [[AWS SAA-C02/计算服务/Elastic Beanstalk\|Elastic Beanstalk]] |
| 存储服务 | [[AWS SAA-C02/存储服务/S3\|S3]] \| [[AWS SAA-C02/存储服务/S3 Security\|S3 Security]] \| [[AWS SAA-C02/存储服务/S3 Glacier Archive\|S3 Glacier]] \| [[AWS SAA-C02/存储服务/EBS\|EBS]] \| [[AWS SAA-C02/存储服务/AWS EFS\|EFS]] \| [[AWS SAA-C02/存储服务/AWS FSx\|FSx]] \| [[AWS SAA-C02/存储服务/AWS Storage Gateway\|Storage Gateway]] \| [[AWS SAA-C02/存储服务/AWS DataSync\|DataSync]] \| [[AWS SAA-C02/存储服务/AWS Snowball\|Snowball]] |
| 数据库服务 | [[AWS SAA-C02/数据库服务/RDS\|RDS]] \| [[AWS SAA-C02/数据库服务/RDS Proxy\|RDS Proxy]] \| [[AWS SAA-C02/数据库服务/Aurora\|Aurora]] \| [[AWS SAA-C02/数据库服务/DynamoDB\|DynamoDB]] \| [[AWS SAA-C02/数据库服务/ElastiCache\|ElastiCache]] \| [[AWS SAA-C02/数据库服务/Redshift\|Redshift]] \| [[AWS SAA-C02/数据库服务/Amazon DocumentDB\|DocumentDB]] \| [[AWS SAA-C02/数据库服务/Amazon Neptune\|Neptune]] \| [[AWS SAA-C02/数据库服务/Amazon Keyspaces\|Keyspaces]] \| [[AWS SAA-C02/数据库服务/Amazon Timestream\|Timestream]] |
| 网络服务 | [[AWS SAA-C02/网络服务/VPC\|VPC]] \| [[AWS SAA-C02/网络服务/Subnet\|Subnet]] \| [[AWS SAA-C02/网络服务/Route Table\|Route Table]] \| [[AWS SAA-C02/网络服务/Security Group\|Security Group]] \| [[AWS SAA-C02/网络服务/NACL\|NACL]] \| [[AWS SAA-C02/网络服务/NAT Gateway\|NAT Gateway]] \| [[AWS SAA-C02/网络服务/Internet Gateway\|Internet Gateway]] \| [[AWS SAA-C02/网络服务/VPC Peering\|VPC Peering]] \| [[AWS SAA-C02/网络服务/Transit Gateway\|Transit Gateway]] \| [[AWS SAA-C02/网络服务/VPC Endpoints\|VPC Endpoints]] \| [[AWS SAA-C02/网络服务/Direct Connect\|Direct Connect]] \| [[AWS SAA-C02/网络服务/Site to Site VPN\|Site-to-Site VPN]] \| [[AWS SAA-C02/网络服务/CloudFront\|CloudFront]] \| [[AWS SAA-C02/网络服务/Route 53 DNS\|Route 53]] \| [[AWS SAA-C02/网络服务/AWS Global Accelerator\|Global Accelerator]] |
| 访问管理与身份 | [[AWS SAA-C02/访问管理和身份识别/IAM\|IAM]] \| [[AWS SAA-C02/访问管理和身份识别/Cognito\|Cognito]] \| [[AWS SAA-C02/访问管理和身份识别/AWS Organizations\|Organizations]] \| [[AWS SAA-C02/访问管理和身份识别/AWS Directory Service\|Directory Service]] \| [[AWS SAA-C02/访问管理和身份识别/AWS Control Tower\|Control Tower]] |
| 安全和加密 | [[AWS SAA-C02/安全和加密/KMS\|KMS]] \| [[AWS SAA-C02/安全和加密/AWS Secrets Manager\|Secrets Manager]] \| [[AWS SAA-C02/安全和加密/AWS Certificate Manager\|ACM]] \| [[AWS SAA-C02/安全和加密/AWS CloudHSM\|CloudHSM]] \| [[AWS SAA-C02/安全和加密/AWS WAF\|WAF]] \| [[AWS SAA-C02/安全和加密/AWS Shield\|Shield]] \| [[AWS SAA-C02/安全和加密/AWS Firewall Manager\|Firewall Manager]] |
| 监控与审计 | [[AWS SAA-C02/监控与审计/CloudWatch\|CloudWatch]] \| [[AWS SAA-C02/监控与审计/CloudTrail\|CloudTrail]] \| [[AWS SAA-C02/监控与审计/AWS X-Ray\|X-Ray]] \| [[AWS SAA-C02/监控与审计/AWS Config\|Config]] \| [[AWS SAA-C02/监控与审计/Amazon GuardDuty\|GuardDuty]] \| [[AWS SAA-C02/监控与审计/Amazon Inspector\|Inspector]] \| [[AWS SAA-C02/监控与审计/Amazon Macie\|Macie]] |
| 应用集成 | [[AWS SAA-C02/应用集成服务/SQS\|SQS]] \| [[AWS SAA-C02/应用集成服务/SNS\|SNS]] \| [[AWS SAA-C02/应用集成服务/Amazon EventBridge\|EventBridge]] \| [[AWS SAA-C02/应用集成服务/AWS Step Functions\|Step Functions]] \| [[AWS SAA-C02/应用集成服务/Amazon Kinesis\|Kinesis]] \| [[AWS SAA-C02/应用集成服务/Amazon MSK\|MSK]] \| [[AWS SAA-C02/应用集成服务/Amazon API Gateway\|API Gateway]] |
| 数据分析 | [[AWS SAA-C02/数据分析服务/AWS Glue\|Glue]] \| [[AWS SAA-C02/数据分析服务/Amazon Athena\|Athena]] \| [[AWS SAA-C02/数据分析服务/Amazon EMR\|EMR]] \| [[AWS SAA-C02/数据分析服务/Amazon OpenSearch\|OpenSearch]] \| [[AWS SAA-C02/数据分析服务/Amazon QuickSight\|QuickSight]] \| [[AWS SAA-C02/数据分析服务/AWS Lake Formation\|Lake Formation]] |
| 灾难恢复与迁移 | [[AWS SAA-C02/灾难恢复和迁移/Disaster Recovery On AWS\|Disaster Recovery]] \| [[AWS SAA-C02/灾难恢复和迁移/AWS Backup\|Backup]] \| [[AWS SAA-C02/灾难恢复和迁移/AWS Elastic Disaster Recovery\|Elastic DR]] \| [[AWS SAA-C02/灾难恢复和迁移/Database Migration Service\|DMS]] |
| 管理与治理 | [[AWS SAA-C02/管理与治理/AWS CloudFormation\|CloudFormation]] \| [[AWS SAA-C02/管理与治理/AWS Systems Manager\|Systems Manager]] \| [[AWS SAA-C02/管理与治理/AWS Well-Architected Framework\|Well-Architected Framework]] \| [[AWS SAA-C02/管理与治理/AWS Trusted Advisor\|Trusted Advisor]] \| [[AWS SAA-C02/管理与治理/Cost Explorer\|Cost Explorer]] |

> ⚠️ 提醒：以上笔记文件夹命名为 **SAA-C02**，但目标考试是 **SAA-C03**——两版考试内容高度重叠，服务层面的知识基本通用，但 C03 相比 C02 更强调无服务器架构（Lambda/Fargate/DynamoDB/Step Functions）和弹性韧性设计的权重。做模拟考时如果遇到笔记里完全没覆盖的新服务或新概念，提醒我一并补充到对应笔记里。

---

## 使用方法

1. 完成一次模拟考后，把**总分、用时、日期**和**所有错题的完整原文（题干+全部选项+官方解析）**发给我
2. 我会：① 在错题本里建档、②判断每题实际所属 Domain、③解释错误原因、④检查笔记库里有没有对应知识点，没有就补充、⑤累积到 2 次以上的批次后开始做趋势/重复错题分析
3. 参考 AIF-C01 的节奏：一般 5-7 次模拟考、最近三次稳定在 80%+ 就可以考虑预约
