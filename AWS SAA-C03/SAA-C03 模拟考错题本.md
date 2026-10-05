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
- [[#批次三（22 题）]]
- [[#复习优先级建议]]
- [[#相关知识笔记（已有课程笔记，按类别索引）]]

---

## 模拟考分数追踪

> **目标**：真实考试通过线是缩放分数 **720/1000**（约 72%，比 AIF-C01 的 700/1000 略高），65 题中同样有 15 题不计分且无法分辨。由于及格线更高、且批次一暴露出**四个 Domain 全部低于及格线**，本轮目标定得比 AIF-C01 更保守：**模拟考正确率需稳定在 82% 以上**（而不是 AIF-C01 的 80%）才考虑预约，给缩放分数误差、真实考场紧张情绪留出更大缓冲。

### 目标与预约决策标准（当前，未设定具体考试日期）

**总体目标**：连续 **2-3 次不同批次的模拟考稳定在 ≥ 82%**，且没有任何一个 Domain 长期低于 75%，[[#重复出现的错题（高危薄弱点，优先复习）]] 里的知识点能脱稿复述，再考虑预约真实考试日期。

**各 Domain 目标（按权重排出的优先级，已用批次一+批次二+批次三三次数据更新）**：

| Domain | 权重 | 批次一 | 批次二 | 批次三 | 三批次均值 | 与 82% 目标的差距 | 加权风险（差距 × 权重） | 优先级 |
|--------|------|--------|--------|--------|-----------|-------------------|------------------------|--------|
| Design Cost-Optimized Architectures | 20% | 64% | 64% | 40% | 56% | 26 分 | **5.2（最高，新晋）** | ★★★★ |
| Design Secure Architectures | 30% | 68% | 71% | 63% | 67% | 15 分 | 4.5 | ★★★★ |
| Design Resilient Architectures | 26% | 72% | 54% | 75% | 67% | 15 分 | 3.9 | ★★★ |
| Design High-Performing Architectures | 24% | 57% | 83% | 81% | 74% | 8 分 | 1.92（最低） | ★★ |

> **优先级顺序再次大幅洗牌**：批次三最大的警报是 **Design Cost-Optimized Architectures 从稳定的 64% 骤降到 40%**（本批次 10 题错了 6 题），三批次均值瞬间跌到全场最低的 56%，加权风险从此前的中等一举跳到**最高**，是目前唯一需要立刻正视的"新增薄弱域"。与此同时，**Design Resilient Architectures 从批次二的 54% 大幅回升到 75%**，印证了批次二复盘时的猜测——那次下滑更可能是"广撒网式"的噪音，而非真实能力倒退，本批次三批次均值与 Secure 并列 67%，风险已回落到中等。**Design Secure Architectures 小幅下滑**（71%→63%），仍是四个域里权重最高（30%）的，即使百分比不是最低也不能掉以轻心。**Design High-Performing Architectures 连续两批次稳定在 80%+**（83%→81%），是唯一已经明显达标的域，可以适当降低复习优先级，把腾出的精力集中到 Cost-Optimized 和 Secure。

**预约决策表（随每批次新数据持续更新，参考 AIF-C01 的框架并调高门槛）**：

| 最近一次/最近三次平均分 | 建议 |
|---|---|
| **≥ 85%** | 距离预约已经很近，再确认一次没有 Domain 集中失分即可考虑预约 |
| **82%~84%** | 达到基本目标，但建议再跑 1-2 次批次确认稳定性（不是单次幸运高分），且逐个检查四个 Domain 是否都不低于 75% |
| **75%~81%** | 尚未达标，按上表的加权优先级顺序（High-Performing → Secure → Cost → Resilient）针对性复习，不建议预约 |
| **< 75%（批次一 66% → 批次二 70% → 批次三 66%，仍在此档）** | 差距明显，需要系统性补课，不只是刷题——建议先完整过一遍 [[SAA-C03 复习计划]] 里对应 Domain 的检查清单，再安排下一次模拟考。批次三分数**回落到与批次一持平**，说明批次二的提升还不稳固；当前重点应放在**新晋最高风险的 Design Cost-Optimized Architectures**（三批次均值最低、且批次三断崖式下跌）和 **Design Secure Architectures**（权重最高的域），而不是已经连续两批次稳定 80%+ 的 High-Performing |

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
| 批次三 | 2026-09-24 | 2h6m | — | 65 | 43 | **66%** | ❌ <72%，未达标（较批次二回落 4 个百分点，回到批次一水平） |

> 首次模拟考即暴露出明显差距（66% vs 72% 及格线，差 6 个百分点），且用时 2h10m 已接近 130 分钟的时长上限——时间管理也是需要关注的点，不只是知识掌握程度的问题。批次二提升到 70%，差距缩小到 2 个百分点，但**批次三又回落到 66%**，说明批次二的提升更像是"这一次刚好答对了那批题"，还没有真正转化为稳定的能力——三批次分数呈 66%→70%→66% 的**锯齿形波动**，而不是持续上升的趋势，这是需要正视的信号：接下来的复习应更聚焦于"补齐具体知识空白"（尤其是批次三暴露的 Cost-Optimized 域），而不是简单地"多做几套题"。用时方面，批次三 2h6m 比前两次略快，说明做题速度有小幅改善，但仍需继续关注。

### 各 Domain 表现趋势

| Domain | 批次一（题量/正确率） | 批次二（题量/正确率） | 批次三（题量/正确率） | 走势 |
|--------|----------------------|----------------------|----------------------|------|
| Design High-Performing Architectures | 14 题 / 57% | 24 题 / 83% | 16 题 / 81% | ↑↑ 大幅改善后保持稳定 |
| Design Secure Architectures | 19 题 / 68% | 17 题 / 71% | 27 题 / 63% | ↗↘ 小幅波动，仍需关注 |
| Design Resilient Architectures | 18 题 / 72% | 13 题 / 54% | 12 题 / 75% | ↓↑ V 形反弹，批次二疑似为噪音 |
| Design Cost-Optimized Architectures | 14 题 / 64% | 11 题 / 64% | 10 题 / 40% | ↓↓↓ 断崖式下跌，新晋最高风险域 |

> **⚠️ 域标注可靠性提醒**：批次二的错题里发现至少 3 道题的平台 Domain 标注与实际内容不符，已按内容重新归类并注明原标注（详见 [[#错题分布统计]]）；**批次三复核后未发现明显的域标注错误**，本批次的百分比可信度更高。汇总百分比始终是按平台原始标注计算的，仅供参考大方向。
>
> **趋势解读**：批次三最重要的信号是 **Design Cost-Optimized Architectures 从连续两批次的 64% 骤降到 40%**（10 题错 6 题），这不太可能只是运气问题——结合 [[#错题分布统计]] 看，本批次 Cost 域的错题覆盖了 Snowball 归档路径、ASG 默认终止策略、EFS/S3 存储类别选型、EC2 采购策略组合、跨 AZ 冗余容量规划、Spot Fleet vs Spot Instances 共 6 个不同知识点，属于"广撒网式"的知识空白，与批次二 Resilient 域当时的模式如出一辙——**这也意味着 Resilient 域批次二的骤降大概率同样是噪音而非真实倒退**，本批次回升到 75% 印证了这一点。Secure 域小幅下滑到 63%，错题集中在 IAM 策略评估细节（Q49/Q63/Q64 三道题都是 IAM policy 的不同侧面），值得留意是否会像 S3 存储类别一样发展成新的重复薄弱点。

---

## 错题分布统计

> 累计 **3 个批次、共 63 道错题**（22+19+22）。Domain 列原则上采用平台原始标注；批次二发现 3 道题标注与实际内容明显不符，已按内容重新归类并在括号中注明原标注（参考 AIF-C01 错题本的处理方式）；批次三复核未发现明显标注错误。

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

### 批次三（共 22 题）

| 题号 | Domain | 考点主题 | 状态 |
|------|--------|---------|------|
| Q9 | Design Secure Architectures | 限制 API Gateway 仅内部可信 IP 段调用：资源策略（非安全组/子网） | 📘 |
| Q31 | Design Secure Architectures | 多团队共享同一 S3 桶按前缀隔离：S3 Access Points（非枚举式 Bucket Policy） | 📘 |
| Q36 | Design Secure Architectures | 单租户硬件隔离但无 BYOL 需求：Dedicated Instance（非 Dedicated Host）🚩重复 | 📘 |
| Q41 | Design Secure Architectures | 遗留 SFTP 客户端接入 S3（选二）：Transfer Family + IAM 角色/身份联邦 | 📘 |
| Q49 | Design Secure Architectures | IAM 策略"区域外全部拒绝+指定 IP 段允许终止"的组合评估 | 📘 |
| Q53 | Design Secure Architectures | 私有托管区域无法解析：VPC 需同时启用 DNS 主机名+DNS 解析两个开关 | 📘 |
| Q54 | Design Secure Architectures | 加密数据库需共享给外部账户独立副本：加密快照 + 共享 KMS 密钥策略 | 📘 |
| Q59 | Design Secure Architectures | 密钥自管理但加密过程交给 S3：SSE-C（非 SSE-KMS/SSE-S3/客户端加密） | 📘 |
| Q63 | Design Secure Architectures | 限制提权但保留自由挂载托管策略的能力：权限边界（仅能挂用户/角色，不能挂组） | 📘 |
| Q64 | Design Secure Architectures | `aws:RequestedRegion` 限制的是目标区域，不是调用发起地 | 📘 |
| Q6 | Design Resilient Architectures | ASG 迟迟不终止不健康实例的排查（选三）：宽限期/Impaired 短暂容忍/ELB 健康检查类型 | 📘 |
| Q40 | Design Resilient Architectures | RDS 存储可能耗尽的最小改动方案：启用存储自动扩展（非迁移 Aurora/DynamoDB/建读取副本） | 📘 |
| Q52 | Design Resilient Architectures | RDS Multi-AZ 引擎版本升级：主备同时升级，仍会产生停机 | 📘 |
| Q17 | Design High-Performing Architectures | 欧洲用户访问延迟高：Route 53 延迟路由 + Aurora 跨区域读取副本 | 📘 |
| Q20 | Design High-Performing Architectures | 消除实例重启后的固定引导耗时：EC2 Hibernate（非 User Data/Metadata/AMI） | 📘 |
| Q25 | Design High-Performing Architectures | 容器化迁移遗留 cron 任务：EventBridge Scheduler + ECS Fargate（非 Lambda/Batch/Step Functions） | 📘 |
| Q15 | Design Cost-Optimized Architectures | Snowball 迁移到长期归档最省成本：零日生命周期策略转 Glacier Deep Archive | 📘 |
| Q16 | Design Cost-Optimized Architectures | ASG 默认终止策略优先级顺序：分配策略 > 最旧启动模板 > 最旧启动配置 > 最接近整点计费 | 📘 |
| Q24 | Design Cost-Optimized Architectures | 数百 EC2 并发访问的低频数据：EFS Standard-IA（非 S3 Standard-IA，需要文件系统语义） | 📘 |
| Q34 | Design Cost-Optimized Architectures | 稳态 20 台+峰值 300 台的成本最优组合：稳态买 RI，弹性部分 On-Demand+Spot 混合 | 📘 |
| Q42 | Design Cost-Optimized Architectures | 3 AZ 冗余投入最省实例数达成 HA：3 AZ 各 2 台（N+1 冗余公式） | 📘 |
| Q61 | Design Cost-Optimized Architectures | 每月一次、可容错、按最低价自动选实例：Spot Fleet（非裸 Spot Instances） | 📘 |

---

## 重复出现的错题（高危薄弱点，优先复习）

> AIF-C01 备考的经验是：**同一题库换讲师/换课程后依然出错的知识点，才是真正没有内化的薄弱区**，优先级高于只错一次的题目。批次二开始出现跨批次重复，批次三又新增一例，下表持续更新。

> **🚩 最高优先级：S3 存储类别选型规则，两个批次里已经出错 4 次**（批次一 Q25、批次二 Q8/Q18/Q30），批次三**没有再出现新的 S3 存储类别错题**，是个好现象，但还不能就此判定已经内化，需要再观察 1-2 个批次确认。

| 题目/主题 | 出现方式 | 反复出错的核心症结 | 关联笔记 |
|-----------|---------|-------------------|---------|
| **🚩 S3 存储类别选型（4 次，批次三未新增）** | 批次一 Q25（可再生数据+30天限制）→ 批次二 Q8（已知低频访问选 IA 还是 IT）→ 批次二 Q18（哪些转换不合法）→ 批次二 Q30（短期高频引用数据该用 Standard） | 四次错误覆盖了存储类别决策的四个不同侧面（最短存储期限、IA vs Intelligent-Tiering 的适用边界、合法转换路径、检索费与短生命周期数据的组合效应），说明目前是**逐条记忆孤立规则**，没有建立起"总拥有成本 = 存储单价 + 检索费 + 是否触发最短期限罚金"这套统一的判断框架 | [[S3#生命周期转换的"瀑布模型"：哪些转换合法（考试高频，多选题常考）]] \| [[S3#短期/高频引用数据的存储类别陷阱：min duration + 检索费的组合（考试高频）]] \| [[S3#Standard-IA vs Intelligent-Tiering：访问模式"已知稳定"还是"未知/多变"]] |
| **🆕 EC2 专用硬件选型（2 次）** | 批次一 Q39（有 BYOL 需求 → 误选 Reserved Instance，正确是 Dedicated Host）→ 批次三 Q36（无 BYOL 需求 → 误选 Dedicated Host，正确是 Dedicated Instance） | 两次错误方向刚好相反，说明目前对"Dedicated Host vs Dedicated Instance"的判断标准还没有稳定下来——核心判断依据只有一个：**题干是否出现许可证绑定/BYOL 描述**，出现则选 Host，没出现默认选 Instance（更省钱且同样满足单租户隔离） | [[EC2#实例租赁类型：Dedicated Host vs Dedicated Instance（考试易混淆点）]] |

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
- **关联笔记**：[[Amazon Redshift#S3 集成 - Redshift Spectrum]]

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
- **关联笔记**：[[Route 53#混合云和全球可视化]]

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
- **关联笔记**：[[Amazon Redshift#Redshift Serverless（无服务器数仓，考试易漏点）]] | [[Amazon Redshift#Redshift ML（用 SQL 构建和训练机器学习模型，考试易漏点）]]

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

## 批次三（22 题）

### Batch 3 复盘

> 批次三 66%（43/65），**比批次二回落 4 个百分点，回到批次一的水平**。最重要的发现是 **Design Cost-Optimized Architectures 断崖式下跌**（64%→64%→40%），一举取代 Resilient 成为加权风险最高的域，详见 [[#各 Domain 表现趋势]]；与此同时 **Design Resilient Architectures 从批次二的 54% 反弹回 75%**，印证了批次二的下滑更可能是噪音而非真实倒退。本批次还确认了一个新的**跨批次重复错题**：EC2 专用硬件选型（Dedicated Host vs Dedicated Instance），累计出错 2 次且两次判断方向相反，详见 [[#重复出现的错题（高危薄弱点，优先复习）]]。本批次复核未发现明显的域标注错误，平台标注可信度较高。

**新出现的模式：IAM 策略评估的"组合题"开始扎堆**——本批次一次性出现 3 道独立的 IAM 细节题（Q49 的 Deny+Allow 条件组合、Q63 的权限边界绑定对象限制、Q64 的 `aws:RequestedRegion` 语义），虽然考点各不相同，但都属于"策略 JSON 逐字逐句抠细节"这一类，如果下一批次再次出现类似题目，应该考虑升级为正式的重复薄弱点。

**其余题目**：多数仍是"混淆两个功能相近的服务/方案"模式的延续（S3TA→Glacier vs 直接目标 Glacier Vault、EFS-IA vs S3-IA、Spot Fleet vs 裸 Spot Instances、Hibernate vs User Data/AMI），已逐一补充到对应笔记。

---

### Design Secure Architectures

#### 批次三 · Q9 - 限制 API Gateway 仅内部可信 IP 段调用 📘

- **题干**：企业要为内部用户构建安全的商业智能 API，要求只有一组可信内部 IP 能调用，其余一律拒绝，且运维复杂度最低
- **我的选择（错）**：修改 API Gateway 关联的安全组，只放行特定 IP
- **正确答案**：**为 API 创建资源策略（Resource Policy），显式拒绝除白名单外的所有 IP**
- **核心原因**：**API Gateway 是完全托管服务，不部署在用户自己的子网里，没有安全组这个概念**，"修改 API Gateway 的安全组"这个前提本身就不成立；资源策略是 API Gateway 原生支持的 IAM 风格 JSON 策略，用 `IpAddress`/`NotIpAddress` 条件即可直接实现 IP 白名单，且无需引入任何额外基础设施
- **关联笔记**：[[Amazon API Gateway#IP 白名单访问控制：资源策略 vs 安全组（考试易混淆点）]]

---

#### 批次三 · Q31 - 多团队共享同一 S3 桶按前缀隔离 📘

- **题干**：多个分析服务共用一个 S3 桶，各自写入独立前缀，要求精细化访问控制、且方案可扩展、运维负担最小
- **我的选择（错）**：用单个 Bucket Policy 枚举每个前缀下所有对象的 ARN，按资源级权限分别授权
- **正确答案**：**为每个服务配置独立的 S3 Access Point，各自绑定限定到对应前缀的访问点策略**
- **核心原因**：Access Points 天然按团队/应用隔离，新增服务只需新建一个 Access Point，不影响已有配置；而单一 Bucket Policy 枚举 ARN 的方式**不可扩展**——对象越来越多时策略会变得庞大且难以维护，任何调整都要重新部署整份策略；Macie 只做敏感数据发现分类，不做访问控制，是干扰项
- **关联笔记**：[[S3 Security#S3 Access Points（考试易漏点）]]

---

#### 批次三 · Q36 - 单租户硬件隔离但无 BYOL 需求 📘 🚩重复错题

- **题干**：医疗健康公司出于合规要求，需要把 EC2 实例隔离到单租户硬件，要求最具成本效益
- **我的选择（错）**：Dedicated Hosts
- **正确答案**：**Dedicated Instances**
- **核心原因**：题干**没有提到任何服务器绑定型软件许可（BYOL）需求**，只是单纯的合规隔离要求——Dedicated Instance 已经能提供"硬件专属于单一账户"的隔离级别，且比 Dedicated Host 更便宜；Dedicated Host 的额外价值（控制实例在物理服务器上的具体位置、支持 BYOL）在本题完全用不上，属于不必要的溢价
- **易错点（🚩 累计第 2 次出错）**：批次一 Q39 是"有 BYOL 需求却选了 Reserved Instance"，这次是"没有 BYOL 需求却选了 Dedicated Host"——两次错误方向正好相反，说明目前对这组概念的判断标准还不稳定。**统一判断口诀**：先找题干有没有提到许可证绑定/BYOL，有 → Dedicated Host；没有 → 默认 Dedicated Instance
- **关联笔记**：[[EC2#实例租赁类型：Dedicated Host vs Dedicated Instance（考试易混淆点）]]

---

#### 批次三 · Q41 - 遗留 SFTP 客户端接入 S3（多选，选二）📘

- **题干**：物流公司的 S3 数据湖需要接入仍依赖 SFTP 的遗留供应商系统，要求全托管、无需自建基础设施，且要支持身份联邦以便按供应商精细映射到 S3 桶/前缀
- **我的选择**：配置 S3 桶策略结合 Transfer Family 身份提供商集成（对）+ 用 Route 53 私有托管区域为每个供应商建专属子域名指向 SFTP 终端节点（错）
- **正确答案**：**部署全托管的 AWS Transfer Family SFTP 终端节点，映射 IAM 角色到各供应商** + 上面选对的那项（身份提供商集成）
- **核心原因**：Route 53 私有托管区域只负责 DNS 解析，**不参与、也不能实现访问隔离或权限控制**，题目里的"用子域名做访问隔离"是无意义的干扰描述；真正的访问隔离机制是 Transfer Family 的 **IAM 角色 + S3 前缀映射**，规模化场景再叠加 Cognito/SAML/自定义 IdP 的身份联邦
- **关联笔记**：[[AWS Transfer Family#身份联邦（Identity Provider 集成）]]

---

#### 批次三 · Q49 - IAM 策略"区域外拒绝+指定 IP 允许"组合评估 📘

- **题干**：给出一份 IAM 策略：第一条对**非 us-west-1 区域**的所有 EC2 操作显式 Deny；第二条对来源 IP 在 `10.200.200.0/24` 网段的请求显式 Allow `ec2:TerminateInstances`。问该策略组的实际效果
```json
{
    "Version":"2012-10-17",
    "Id":"EC2TerminationPolicy",
    "Statement":[
        {
            "Effect":"Deny",
            "Action":"ec2:*",
            "Resource":"*",
            "Condition":{
                "StringNotEquals":{
                    "ec2:Region":"us-west-1"
                }
            }
        },
        {
            "Effect":"Allow",
            "Action":"ec2:TerminateInstances",
            "Resource":"*",
            "Condition":{
                "IpAddress":{
                    "aws:SourceIp":"10.200.200.0/24"
                }
            }
        }
    ]
}
```
- **我的选择（错）**：判断为"除 us-west-1 外任意区域，源 IP 匹配时可以终止实例"
- **正确答案**：**只有 us-west-1 区域内的 EC2 实例，且用户源 IP 在该网段时，才能被终止**
- **核心原因**：第一条策略先把"非 us-west-1 区域的所有 EC2 操作"全部显式拒绝，这是**范围限定的前置条件**；第二条的允许只在这个前提**没有被拒绝**的情况下才生效，也就是只剩下 us-west-1 区域内的操作还有机会被第二条策略允许；我把两条策略的作用范围搞反了，误以为第二条"畅通无阻"地覆盖所有非 us-west-1 区域
- **关联笔记**：[[IAM#策略评估逻辑（考试陷阱高发区）]]

---

#### 批次三 · Q53 - 私有托管区域无法解析 📘

- **题干**：创建了私有托管区域并关联到 VPC，但 DNS 查询始终无法解析
- **我的选择（错）**：检查私有托管区域与本地网络路由到同一域名的 Resolver 规则之间是否存在冲突
- **正确答案**：**检查并启用该 VPC 的 DNS 主机名（`enableDnsHostnames`）和 DNS 解析（`enableDnsSupport`）设置**
- **核心原因**：私有托管区域的查询**只能由 VPC 内置的 Amazon DNS 服务器**处理，这两个 VPC 级别的开关是私有托管区域能生效的**前提条件**，非默认方式创建的 VPC 这两项可能未开启；Resolver 规则冲突时是"按最长匹配路由"而不是完全无法解析，NS/SOA 记录是公有托管区域的机制，都不是这题的正确排查方向
- **关联笔记**：[[Route 53#私有托管区域无法解析：先检查 VPC 的两个 DNS 开关（考试易漏点）]]

---

#### 批次三 · Q54 - 加密数据库需共享给外部账户独立副本 📘

- **题干**：需要把存储在 RDS 中的敏感财务数据共享给外部审计方（有自己的 AWS 账户），审计方需要自己独立的一份数据库副本
- **我的选择（错）**：把数据库快照导出到 S3，通过 IAM 角色授权审计方访问该 S3 桶中的对象
- **正确答案**：**创建加密的手动快照，共享该快照，并授权审计方访问对应的 KMS 加密密钥**
- **核心原因**：RDS 快照存储在 AWS 托管的 S3 桶中，**客户没有直接访问权限**，无法把这些底层对象当作普通 S3 对象共享出去，这个前提就不成立；正确路径是直接共享加密快照本身 + 共享 KMS 密钥策略（否则对方拿到快照也无法解密）；读取副本是持续同步机制，不满足"独立的一次性副本"这个语义
- **关联笔记**：[[RDS#跨账户共享加密快照（考试易混淆点）]]

---

#### 批次三 · Q59 - 密钥自管理但加密过程交给 S3 📘

- **题干**：金融公司要求加密密钥必须存储在本地自建的应用程序中，但希望把数据存储和加密过程都交给 S3 完成，同时继续使用现有的密钥
- **我的选择（错）**：客户端加密（数据在客户端本地完成加密后再上传）
- **正确答案**：**SSE-C（服务端加密，客户提供密钥）**
- **核心原因**：题目要求"继续用自己的密钥，但把加密过程交给 S3 完成"——这恰好是 SSE-C 的定位：客户在每次请求中携带自己的密钥，由 **S3 负责实际的加解密操作**；客户端加密则要求客户自己完成整个加密过程后再上传密文，S3 完全不参与加密，不满足"把加密过程交给 S3"这一条；SSE-S3/SSE-KMS 都由 AWS 管理密钥，不满足"密钥必须存储在客户自建系统中"的要求
- **关联笔记**：[[S3 Security#静态加密类型对比]]

---

#### 批次三 · Q63 - 限制提权但保留自由挂载托管策略的能力 📘

- **题干**：希望开发者可以自由尝试挂载各种 AWS 托管策略到自己身上，但不能通过挂载 `AdministratorAccess` 实现提权
- **我的选择（错）**：把开发者放进一个 IAM 组，在组上设置权限边界
- **正确答案**：**为每个开发者（用户）单独设置权限边界**
- **核心原因**：**权限边界只能直接附加到 IAM 用户或角色，不能附加到 IAM 组**，"在组上设置权限边界"这个方案从设计上就不成立；必须为每个开发者的用户身份单独配置权限边界，才能限制其有效权限的天花板，即使误挂了 `AdministratorAccess` 也不会真正获得管理员权限；SCP 需要 AWS Organizations 且题干未提及组织架构，普通 IAM 策略则可能被开发者自己移除，不如权限边界可靠
- **关联笔记**：[[IAM#权限边界（Permission Boundaries）vs 服务控制策略（SCP）]]

---

#### 批次三 · Q64 - `aws:RequestedRegion` 条件键语义 📘

- **题干**：给出一份策略：`Allow ec2:RunInstances`，条件为 `aws:RequestedRegion` 等于 `eu-west-1`，问这份策略允许什么
- **我的选择（错）**：只有当 API 调用**从 eu-west-1 区域发起**时，才允许在 eu-west-1 启动实例
- **正确答案**：**允许在 eu-west-1 区域启动实例，调用可以从世界任何地方发起**
- **核心原因**：`aws:RequestedRegion` 比较的是 **API 调用所指向（目标）的区域**，与调用者物理位置/发起请求所在区域无关；我把"目标区域"和"调用来源"这两个完全不同的维度搞混了——真正限制"调用来源"的是 `aws:SourceIp` 等条件键
- **关联笔记**：[[IAM#`aws:RequestedRegion` 条件键：限定的是目标区域，不是发起请求的位置（考试易混淆点）]]

---

### Design Resilient Architectures

#### 批次三 · Q6 - ASG 迟迟不终止不健康实例的排查（多选，选三）📘

- **题干**：ASG 一直没有终止一个被判定为不健康的 EC2 实例，需要排查可能的原因
- **我的选择**：健康检查宽限期尚未过期（对）+ 该实例可能是 Spot 实例，ASG 不能终止 Spot 实例（错）+ 自定义健康检查判定不健康的实例 ASG 不会终止（错）
- **正确答案**：健康检查宽限期尚未过期 + **该实例可能处于 Impaired 状态**（ASG 会短暂容忍等待恢复）+ **该实例已失败 ELB 健康检查，但 ASG 健康检查类型仍是默认的 EC2**
- **核心原因**：Spot 实例**可以**被 ASG 正常终止；自定义健康检查上报 Unhealthy 后 ASG 同样会终止该实例，这两个都是错误归因；真正会导致"不终止"的是：宽限期未过、Impaired 状态触发短暂容忍等待恢复、以及健康检查类型未显式设为 ELB 导致 ASG 感知不到 ELB 层面的失败
- **关联笔记**：[[Auto Scaling#ASG 不健康实例故障排查清单（考试易漏点）]]

---

#### 批次三 · Q40 - RDS 存储可能耗尽的最小改动方案 📘

- **题干**：手游应用用户量暴涨，工程团队担心 RDS MySQL 存储空间可能耗尽，要求以最少开发和运维工作量解决
- **我的选择（错）**：迁移到 Amazon Aurora（同样提供存储自动扩展）
- **正确答案**：**直接为现有 RDS MySQL 实例启用存储自动扩展**
- **核心原因**：RDS 本身就原生支持存储自动扩展，无需任何迁移即可直接启用；迁移到 Aurora 虽然也能达到同样效果，但涉及明显更高的迁移工作量，违反"最小化开发和运维工作量"的要求；迁移到 DynamoDB 需要重写应用的数据访问逻辑，工作量更大；建读取副本无法解决主实例存储空间不足的问题
- **关联笔记**：[[RDS#存储自动扩展（Storage Autoscaling，考试易漏点）]]

---

#### 批次三 · Q52 - RDS Multi-AZ 引擎版本升级的停机行为 📘

- **题干**：金融公司对 Multi-AZ 部署的 MySQL RDS 实例安排了数据库引擎版本升级维护窗口，问升级期间会发生什么
- **我的选择（错）**：先升级主实例，再升级备用实例，整个过程不产生停机
- **正确答案**：**主实例和备用实例会被同时触发升级，这会产生停机，直到升级完成**
- **核心原因**：Multi-AZ 能规避的是常规**故障转移**场景（硬件故障、AZ 中断）的停机，但**数据库引擎版本升级是明确的例外**——无论是否开启 Multi-AZ，主备实例都会同时升级，停机不可避免，时长随实例规格增大而增加；我把"Multi-AZ 提供高可用"错误地泛化成了"Multi-AZ 下所有维护操作都零停机"
- **关联笔记**：[[RDS#引擎版本升级：Multi-AZ 不能避免停机（考试易混淆点）]]

---

### Design High-Performing Architectures

#### 批次三 · Q17 - 欧洲用户访问延迟高（多选，选二）📘

- **题干**：Web 层在 EC2、数据库用 Aurora，全部部署在 us-east-1，90% 客户在美国和欧洲，欧洲用户反馈应用加载慢
- **我的选择**：在 eu-west-1 新增一套 EC2 web 层，Route 53 启用故障转移路由（错）+ 在 eu-west-1 创建 Aurora 读取副本（对）
- **正确答案**：在 eu-west-1 新增一套 EC2 web 层，Route 53 启用**延迟路由** + eu-west-1 创建 Aurora 读取副本
- **核心原因**：故障转移路由是为可用性设计的（主故障时切到备），**不能用来降低延迟**；延迟路由才是专门根据"哪个区域给用户提供最低延迟"来路由流量的策略，正对应本题"欧洲用户加载慢"这个延迟问题；Aurora 读取副本能把读请求就近分流到欧洲区域，缓解跨大西洋的数据库访问延迟
- **关联笔记**：[[Route 53#路由策略]] | [[Aurora]]

---

#### 批次三 · Q20 - 消除实例重启后的固定引导耗时 📘

- **题干**：某计算机视觉应用每次实例停止再启动，都需要约 3 分钟运行辅助程序才能恢复工作，希望尽量缩短这段引导时间
- **我的选择（错）**：制作一个 AMI，用该 AMI 启动实例
- **正确答案**：**使用 EC2 Hibernate（休眠）**
- **核心原因**：Hibernate 会在实例停止时把内存（RAM）内容保存到 EBS 根卷，下次启动**直接恢复内存状态和正在运行的进程**，从根本上跳过重新初始化的过程；重新制作 AMI 虽然能预装依赖、减少部分安装步骤，但**无法避免进程重新初始化**这个真正的耗时来源，达不到 Hibernate 那种"直接接续运行状态"的效果
- **关联笔记**：[[EC2#缩短应用重启后的"引导时间"：Hibernate 是唯一正解（考试易混淆点）]]

---

#### 批次三 · Q25 - 容器化迁移遗留 cron 任务 📘

- **题干**：制造业分析公司的一批本地 Linux cron 定时脚本（单次执行最长 30 分钟）要迁移上云，要求容器化、无服务器、自动扩缩容、未来能响应事件触发，且最小化重构和运维
- **我的选择（错）**：把镜像部署到 AWS Batch，用 EC2 托管计算环境，在 Batch 里配置 cron 表达式调度
- **正确答案**：**用 EventBridge Scheduler 定义 cron 定时规则，触发 ECS Fargate 任务**
- **核心原因**：EventBridge Scheduler 是全托管的无服务器调度器，原生支持调用 ECS Fargate `RunTask`，天然具备接入事件触发的扩展性；AWS Batch 面向有队列/依赖关系的批量计算作业，且用 EC2 托管计算环境还需要额外配置计算环境，运维负担明显更高，不是"最小化重构和运维"的最优解；Lambda 方案（另一干扰选项）会因 15 分钟超时上限无法覆盖 30 分钟的脚本
- **关联笔记**：[[AWS Fargate#容器化迁移遗留 Cron 任务的最佳组合（考试易漏点）]]

---

### Design Cost-Optimized Architectures

#### 批次三 · Q15 - Snowball 迁移到长期归档最省成本 📘

- **题干**：想用 AWS Snowball 把本地备份数据迁移到 AWS 上的长期归档存储层，要求成本最优
- **我的选择（错）**：创建 Snowball 任务，直接把 S3 Glacier Deep Archive Vault 设为目标
- **正确答案**：**创建 Snowball 任务，目标设为 S3 Bucket，配置零日生命周期策略当天转到 S3 Glacier Deep Archive**
- **核心原因**：**Snowball 任务只能把 S3 Bucket 作为目标，不能直接指定 Glacier/Glacier Deep Archive 的 Vault**；正确路径是先落地到 S3 Bucket，再用零日生命周期策略把数据当天转换到 Glacier Deep Archive——计费时生命周期策略优先生效，最终只按 Glacier Deep Archive 费率计费，不会产生额外的 S3 Standard 停留费用
- **关联笔记**：[[AWS Snowball#Snowball 导入数据后转长期归档（考试易混淆点）]]

---

#### 批次三 · Q16 - ASG 默认终止策略优先级顺序 📘

- **题干**：给出 4 个实例（最旧启动模板/最旧启动配置/最新启动配置/最接近下一个计费小时），ASG 需要从实例数最多的 AZ 终止一个实例，问会终止哪一个
- **我的选择（错）**：拥有最旧启动模板的实例（Instance A）
- **正确答案**：**拥有最旧启动配置的实例（Instance B）**
- **核心原因**：默认终止策略的优先级顺序是：① On-Demand/Spot 的分配策略（本题未涉及）→ ② **最旧的启动模板，但如果存在使用启动配置的实例则优先排除它们**（Instance A 因此被排除考虑）→ ③ **最旧的启动配置**（命中 Instance B）→ ④ 最接近下一个计费小时（优先级最低，未触及）；我把优先级顺序记错了，误以为"最旧启动模板"排在"最旧启动配置"之前生效
- **关联笔记**：[[Auto Scaling#默认终止策略（Default Termination Policy）优先级顺序（考试易漏点）]]

---

#### 批次三 · Q24 - 数百 EC2 并发访问的低频数据 📘

- **题干**：需要为可被数百个 EC2 实例并发访问的低频访问文件寻找最具成本效益的存储方案
- **我的选择（错）**：S3 Standard-IA
- **正确答案**：**Amazon EFS Standard-IA**
- **核心原因**：题目要求"可被数百个 EC2 实例**并发访问**"，这是**文件系统**语义（挂载点、POSIX 兼容、并发读写同一文件系统），S3 是对象存储，没有这种共享文件系统的访问模式；EFS Standard-IA 同时满足"文件系统接口"和"低频访问降本"两个要求，是本题的正确选型
- **关联笔记**：[[AWS EFS#EFS Standard-IA vs S3 Standard-IA（考试易混淆点）]]

---

#### 批次三 · Q34 - 稳态+峰值混合负载的最优采购组合 📘

- **题干**：大数据工作负载 24x7 运行，稳态至少需要 20 台实例才能维持最低性能，峰值需要 300 台，历史数据显示 80% 的时间实际需要 80 台
- **我的选择（错）**：购买 20 台 On-Demand，其余用 ASG 按需配置 Spot 实例
- **正确答案**：**购买 80 台预留实例（覆盖稳态需求），额外用 ASG + 启动模板混合配置 On-Demand 和 Spot 实例应对超出部分**
- **核心原因**：80% 的时间实际需求是 80 台（而非题面最低门槛 20 台），这才是真正的"稳态基线"，应该按 80 台购买 RI 才能最大化节省成本；只买 20 台 On-Demand 意味着 80% 的时间里还有 60 台缺口要用 Spot 覆盖，Spot 可用性没有保障，可能连稳态需求都无法满足；全部买 On-Demand 又比 RI 更贵
- **关联笔记**：[[EC2]]

---

#### 批次三 · Q42 - 3 AZ 冗余投入最省实例数达成 HA 📘

- **题干**：应用需要至少 4 个实例才能维持可接受的用户体验，当前部署在单个 AZ 的 4 台实例上，要求以最低成本实现高可用
- **我的选择（错）**：部署到 2 个 AZ，每个 AZ 4 台（共 8 台）
- **正确答案**：**部署到 3 个 AZ，每个 AZ 2 台（共 6 台）**
- **核心原因**：3 个 AZ 每个放 2 台，任意 1 个 AZ 故障后仍剩 4 台，刚好满足最低要求，总成本只需 6 台；2 个 AZ 若每个只放 2 台，一旦 1 个 AZ 故障就只剩 2 台，低于最低门槛，必须把每个 AZ 增加到 4 台（共 8 台）才能保证故障后仍有 4 台可用，成本明显更高——3 AZ 的"N+1 冗余公式"比 2 AZ 更省实例数
- **关联笔记**：[[Auto Scaling#跨 AZ 容量规划：N+1 冗余公式（考试易漏点）]]

---

#### 批次三 · Q61 - 每月一次、可容错、按最低价自动选实例 📘

- **题干**：公司每月运行一次约 2 小时的大数据工作负载，可以灵活分布到多种规格的服务器上，且要求能容忍服务器故障，寻求最具成本效益的方案
- **我的选择（错）**：直接使用裸的 Spot 实例
- **正确答案**：**使用 Spot Fleet**
- **核心原因**：裸的 Spot 实例被回收后**不会自动补充**，无法满足"容忍服务器故障"这个隐含要求；Spot Fleet 默认持续维持目标容量，个别实例被中断后会自动请求替换，同时可配置 `lowestPrice` 分配策略让 AWS 在多个实例池中自动挑选当前最便宜的组合，把"选实例类型"这个决策也交给 AWS 自动完成
- **关联笔记**：[[EC2#Spot Fleet vs 单独使用 Spot Instances（考试易混淆点）]]

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
├── "免费/收费边界要看清楚触发条件，不要想当然多付钱"
│   批次二 Q46（S3TA 未加速则完全免费，不是仍收标准传输费）
├── "🚩 EC2 专用硬件选型：判断标准只有一个——题干有没有提到 BYOL/许可证绑定（累计 2 次，方向相反）"
│   批次一 Q39（有 BYOL 却选了 RI）、批次三 Q36（无 BYOL 却选了 Dedicated Host）
├── "IAM 策略/条件键要逐字逐句抠细节，不能凭直觉判断范围"
│   批次三 Q49（Deny+Allow 组合评估，范围限定顺序不能搞反）、Q63（权限边界只能挂用户/角色，不能挂组）、Q64（aws:RequestedRegion 限制目标区域而非调用来源）
├── "遇到'紧急/最小改动'类问题，先看有没有'直接开启某个开关'的选项，不要跳去想迁移方案"
│   批次三 Q40（RDS 存储自动扩展 vs 迁移 Aurora/DynamoDB）
├── "组件的能力边界继续被张冠李戴：API Gateway 没有安全组、ASG 不会立即终止 Impaired 实例、Route 53 私有托管区域不做访问隔离"
│   批次三 Q9、Q6、Q41
├── "成本优化题的判断依据是具体的计费/容量机制，批次三集中爆发"
│   批次三 Q15（Snowball 不能直接指定 Glacier Vault）、Q16（ASG 默认终止策略优先级顺序）、Q42（3 AZ 的 N+1 冗余公式比 2 AZ 更省实例数）、Q61（Spot Fleet 自动维持目标容量，裸 Spot 不会）
└── "看似相同的存储/物理隔离概念，实际适用维度不同"
    批次三 Q24（EFS-IA 提供文件系统语义，S3-IA 不提供）
```


---

## 相关知识笔记（已有课程笔记，按类别索引）

> 学习课程已完成，以下是笔记库中已有的完整服务知识笔记索引，按类别分组，方便复习时快速跳转。本错题本不重复整理这些服务本身的功能介绍，只记录模拟考暴露出的具体错题。

| 类别 | 笔记 |
|------|------|
| **计算服务** | [[EC2]] \| [[Auto Scaling]] \| [[AWS Load Balance]] \| [[ECS]] \| [[EKS]] \| [[AWS Fargate]] \| [[AWS Batch]] \| [[AWS Lambda]] \| [[Elastic Beanstalk]] \| [[AWS Amplify]] \| [[AWS Device Farm]] |
| **存储服务** | [[S3]] \| [[S3 Security]] \| [[S3 Glacier Archive]] \| [[EBS]] \| [[AWS EFS]] \| [[AWS FSx]] \| [[AWS Snowball]] \| [[AWS Storage Gateway]] \| [[AWS DataSync]] \| [[AWS Transfer Family]] \| [[Storage Service]] |
| **数据库服务** | [[RDS]] \| [[RDS Proxy]] \| [[Aurora]] \| [[DynamoDB]] \| [[ElastiCache]] \| [[Amazon Redshift]] \| [[Amazon Neptune]] \| [[Amazon DocumentDB]] \| [[Amazon Keyspaces]] \| [[Amazon Timestream]] |
| **网络服务** | [[VPC]] \| [[VPC Peering]] \| [[Transit Gateway]] \| [[Subnet]] \| [[Route Table]] \| [[CIDR]] \| [[Security Group]] \| [[NACL]] \| [[VPC Endpoints]] \| [[VPC Flow Logs]] \| [[VPC Traffic Mirroring]] \| [[Internet Gateway]] \| [[NAT Gateway]] \| [[NAT Instance]] \| [[ENI]] \| [[Virtual Private Gateway]] \| [[Customer Gateway]] \| [[Direct Connect]] \| [[Site to Site VPN]] \| [[Route 53]] \| [[CloudFront]] \| [[AWS Global Accelerator]] \| [[AWS Outposts]] \| [[Bastion Hosts]] |
| **安全和加密** | [[IAM]] \| [[KMS]] \| [[AWS Certificate Manager]] \| [[AWS CloudHSM]] \| [[AWS Secrets Manager]] \| [[AWS WAF]] \| [[AWS Shield]] \| [[AWS Firewall Manager]] |
| **访问管理和身份识别** | [[Cognito]] \| [[AWS Organizations]] \| [[AWS Control Tower]] \| [[AWS Directory Service]] |
| **应用集成服务** | [[SQS]] \| [[SNS]] \| [[Amazon Kinesis]] \| [[Amazon API Gateway]] \| [[AWS Step Functions]] \| [[Amazon EventBridge]] \| [[Amazon MSK]] \| [[Amazon AppFlow]] \| [[Amazon SES]] \| [[Amazon SWF]] \| [[Amazon Connect]] \| [[Amazon Pinpoint (Deprecated)]] |
| **数据分析服务** | [[AWS Glue]] \| [[Amazon Athena]] \| [[Amazon EMR]] \| [[Amazon OpenSearch]] \| [[Amazon QuickSight]] \| [[AWS Lake Formation]] \| [[Amazon Managed Service for Apache Flink]] |
| **监控与审计** | [[CloudWatch]] \| [[CloudTrail]] \| [[AWS Config]] \| [[Amazon GuardDuty]] \| [[Amazon Macie]] \| [[Amazon Inspector]] \| [[AWS X-Ray]] |
| **管理与治理** | [[AWS CloudFormation]] \| [[AWS Systems Manager]] \| [[AWS OpsWorks]] \| [[AWS Service Catalog]] \| [[Cost Explorer]] \| [[AWS Trusted Advisor]] \| [[AWS Artifact]] \| [[AWS Audit Manager]] \| [[AWS Well-Architected Framework]] |
| **灾难恢复和迁移** | [[Disaster Recovery On AWS]] \| [[AWS Elastic Disaster Recovery]] \| [[AWS Backup]] \| [[Database Migration Service]] \| [[AWS Application Discovery Service]] \| [[VMware Cloud on AWS]] |

---

## 使用方法

1. 完成一次模拟考后，把**总分、用时、日期**和**所有错题的完整原文（题干+全部选项+官方解析）**发给我
2. 我会：① 在错题本里建档、②判断每题实际所属 Domain、③解释错误原因、④检查笔记库里有没有对应知识点，没有就补充、⑤累积到 2 次以上的批次后开始做趋势/重复错题分析
3. 参考 AIF-C01 的节奏：一般 5-7 次模拟考、最近三次稳定在 80%+ 就可以考虑预约
