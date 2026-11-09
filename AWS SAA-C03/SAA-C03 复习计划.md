# SAA-C03 复习计划

> 学习课程已完成，当前处于模拟考刷题+查漏补缺阶段。本笔记记录**按 Domain 的复习检查清单**和**每个批次之间的针对性备考重点**，与 [[SAA-C03 模拟考错题本]] 配合使用：错题本记录"考过什么、错在哪"，本笔记记录"下一次考前该重点看什么"。
>
> 目标框架见 [[SAA-C03 模拟考错题本#目标与预约决策标准（当前，未设定具体考试日期）]]：模拟考正确率需连续 2-3 次稳定在 ≥ 82%，且四个 Domain 都不低于 75%，才考虑预约真实考试。

---

## 总体节奏

1. 完成一次模拟考 → 把成绩和错题发给我，更新 [[SAA-C03 模拟考错题本]]
2. 根据错题本的加权风险排序，针对性重读对应的知识笔记（不是从头重新过一遍全部课程）
3. 参考本笔记的"批次备考重点"，针对当前最薄弱的 Domain 做专项复习
4. 安排下一次模拟考，重复以上流程，直到达到预约标准

---

## 按 Domain 的复习检查清单

### Design Secure Architectures（30%，权重最高）

- [ ] IAM：策略评估逻辑（显式拒绝优先）、权限边界 vs SCP（含"权限边界不能挂组"）、`aws:RequestedRegion` 等条件键语义
- [ ] S3 安全：Bucket Policy vs IAM 策略、SSE-S3/SSE-KMS/SSE-C/客户端加密的选型依据、S3 Access Points
- [ ] 网络层访问控制：WAF 地理位置匹配 vs CloudFront Georestriction vs Route 53 地理位置路由的入口判断
- [ ] API Gateway：端点类型、资源策略实现 IP 白名单（不是安全组）
- [ ] VPC 私有托管区域：DNS 主机名/DNS 解析两个开关
- [ ] 加密与密钥管理：ACM 自动续期边界、KMS 跨账户共享、RDS 加密快照跨账户共享
- [ ] 遗留协议接入：AWS Transfer Family（SFTP→S3）+ 身份联邦

### Design Resilient Architectures（26%）

- [ ] Multi-AZ vs 读取副本（同步 vs 异步）、Multi-AZ 引擎升级仍会停机
- [ ] Auto Scaling：健康检查类型与宽限期、不健康实例故障排查清单、AZ 再平衡 vs 不健康替换顺序、Standby+暂停进程做维护
- [ ] RDS Custom vs 标准 RDS、存储自动扩展
- [ ] 无服务器数据管道选型：SQS+Lambda vs 隐藏的自管理 EC2 轮询
- [ ] 全球化架构：Aurora Global Database vs 混用 DynamoDB（最小化重构原则）
- [ ] S3 一致性模型（已是强一致性）

### Design High-Performing Architectures（24%，目前最稳，可降低优先级）

- [ ] Redshift Spectrum vs Athena、Redshift Serverless + Redshift ML vs Athena ML
- [ ] Kinesis Enhanced Fan-Out、图数据库 Neptune 的适用场景
- [ ] Glue DataBrew vs Glue Studio（零代码 vs 代码生成）
- [ ] EC2 Hibernate vs User Data/Metadata/AMI（缩短重启后引导时间）
- [ ] 容器化定时任务：EventBridge Scheduler + Fargate vs Lambda（15分钟上限）/Batch/Step Functions

### Design Cost-Optimized Architectures（20%，当前风险最高）

- [ ] S3 存储类别选型统一框架：最短存储期限 + 检索费 + 访问模式已知/未知 + 合法转换路径（🚩 历史最高频错题）
- [ ] EC2 采购策略：Dedicated Host vs Dedicated Instance（判断依据：是否提到 BYOL/许可证绑定）、RI 覆盖稳态+On-Demand/Spot 覆盖弹性部分、Spot Fleet vs 裸 Spot Instances
- [ ] Auto Scaling 默认终止策略优先级顺序、跨 AZ 的 N+1 冗余容量规划公式
- [ ] EFS Standard-IA vs S3 Standard-IA（文件系统语义 vs 对象存储）
- [ ] Snowball 迁移到长期归档：只能先落 S3 Bucket 再用生命周期策略转 Glacier/Deep Archive
- [ ] Direct Connect 数据传出费用优化（同区域部署减少 DTO）

---

## 批次备考重点（每次模拟考后更新）

### 批次四备考重点（基于批次三 66% 的复盘，2026-09-25 制定）

> 完整数据见 [[SAA-C03 模拟考错题本#批次三（22 题）]]。批次三比批次二回落 4 个百分点（70%→66%），且 **Design Cost-Optimized Architectures 从 64% 骤降到 40%**，是当前最紧急的信号。

**优先级 1 —— Design Cost-Optimized Architectures（新晋加权风险最高的域）**

按上方"按 Domain 的复习检查清单"里 Cost-Optimized 一节逐条过一遍，重点是本批次直接暴露的六个新知识点（Snowball 归档路径、ASG 默认终止策略优先级、EFS-IA vs S3-IA、RI+Spot 稳态/弹性配比、N+1 冗余公式、Spot Fleet vs 裸 Spot），全部已补充到对应笔记：
- [[Auto Scaling#默认终止策略（Default Termination Policy）优先级顺序（考试易漏点）]]
- [[Auto Scaling#跨 AZ 容量规划：N+1 冗余公式（考试易漏点）]]
- [[AWS Snowball#Snowball 导入数据后转长期归档（考试易混淆点）]]
- [[AWS EFS#EFS Standard-IA vs S3 Standard-IA（考试易混淆点）]]
- [[EC2#Spot Fleet vs 单独使用 Spot Instances（考试易混淆点）]]

**优先级 2 —— 修复两个跨批次重复错题**

- 🚩 **EC2 Dedicated Host vs Dedicated Instance（2 次，方向相反）**：批次一"有 BYOL 却选了 RI"，批次三"无 BYOL 却选了 Dedicated Host"。判断口诀：题干提到许可证绑定/BYOL → Host；没提到 → 默认 Instance。见 [[EC2#实例租赁类型：Dedicated Host vs Dedicated Instance（考试易混淆点）]]
- 🚩 **S3 存储类别选型（4 次，批次三未新增，但不能掉以轻心）**：重新过一遍统一判断框架，确认能不看笔记复述最短存储期限+检索费+访问模式+合法转换路径这四个维度

**优先级 3 —— IAM 策略评估细节（新出现的题目集群，警惕演变成重复错题）**

批次三一次性出现 3 道独立的 IAM 细节题（Deny+Allow 组合评估、权限边界不能挂组、`aws:RequestedRegion` 语义），建议自己动手写 2-3 份示例策略并手动评估结果，而不是只重读笔记：
- [[IAM#3. 策略评估逻辑（考试陷阱高发区）]]
- [[IAM#4. 限制型机制：权限边界 vs SCP vs 会话策略]]
- [[IAM#5.2 `aws:RequestedRegion`：限定目标区域，不是发起位置（易混淆点）]]

**优先级 4 —— Design Secure Architectures 整体温习**

批次三降到 63%，虽不是最低分但权重最高（30%），额外看一遍：
- [[Amazon API Gateway#IP 白名单访问控制：资源策略 vs 安全组（考试易混淆点）]]
- [[S3 Security#S3 Access Points（考试易漏点）]]
- [[AWS Transfer Family]]（新出现的服务，第一次考到）

**可以降低优先级 —— Design High-Performing Architectures**

连续两批次稳定在 80%+（83%→81%），暂不需要额外投入时间。

**其他 —— 考试节奏**

三次模拟考用时都在 2h6m～2h10m，接近 130 分钟上限。建议做一次纯计时训练（不追求全对，只练速度），确认能在合理时间内完成 65 题。

---

### 批次五备考重点（基于批次四 64% 的复盘，2026-11-04 制定）

> 完整数据见 [[SAA-C03 模拟考错题本#批次四（23 题，5 处原标注调整）]]。四批次走势 66%→70%→66%→64%，最近两批次连续下滑，且距离批次四已过去一个月。按错题本的决策表，< 75% 应先**系统补课再考**，所以批次五前安排 3 个工作日的专项复习（在 Thaumion Plans「SAA C03 备考计划」中排期），之后再做模拟考。

**复习节奏（工作时间，每天约 2 小时）**

| 日期 | 内容 | 对应 Domain |
|------|------|-------------|
| 11-04 周三 | IAM 策略评估动手写 + VPC 访问控制 + 🚩 Route 53 Resolver | Secure（30%，加权风险最高） |
| 11-05 周四 | 加密与密钥 + 边缘安全（地理限制/OAC+WAF）+ 🚩 EC2 租赁 | Secure |
| 11-06 周五 | Resilient 知识面扫描 + 🚩 S3 存储类别脱稿复述 + 限时节奏练习 | Resilient（26%，波动最大） |
| 11-09 周一 | **模拟考批次五**（严格 130 分钟、不查资料、一次做完） | — |
| 11-10 周二 | 复盘批次五 → 更新错题本 + 本笔记追加批次六重点 | — |

**优先级 1 —— Design Secure Architectures（58%，连续两批次下滑）**

- 动手写 3 份 IAM 示例策略并手动评估：显式 Deny + Allow 组合、"区域外拒绝 + 指定 IP 允许"、`aws:RequestedRegion` → [[IAM#3. 策略评估逻辑（考试陷阱高发区）]]
- 权限边界 vs SCP：边界只能挂用户/角色不能挂组；SCP 影响成员账户 root、不影响 service-linked role（批次三 Q63、批次四 Q63）→ [[IAM]] [[AWS Organizations]]
- 批次四暴露的 VPC 基础题集群：公网实例 NAT 由 Internet Gateway 完成、NAT 网关必须在公有子网、安全组有状态 vs NACL 无状态 → [[Internet Gateway]] [[NAT Gateway]] [[Security Group]] [[NACL]]
- 加密选型对比表（自己画）：SSE-S3（默认对象级唯一密钥）/ SSE-KMS（可审计）/ SSE-C（自管密钥、S3 执行加密）/ 客户端加密；KMS 跨账户 + 加密 RDS 快照共享 → [[S3 Security]] [[KMS]]
- 边缘安全三选一：WAF 地理位置匹配（ALB）/ CloudFront Georestriction / Route 53 地理位置路由；CloudFront + S3 私有内容 = OAC + WAF → [[AWS WAF]] [[CloudFront]]

**优先级 2 —— 三个跨批次重复错题（考前必须能脱稿复述）**

- 🚩 Route 53 Resolver（2 次）：谁发起查询就用谁那一侧的端点——本地查 VPC → 入站；VPC 查本地 → 出站。画一张双向图 → [[Route 53#混合云和全球可视化]]
- 🚩 EC2 租赁（3 次）：有 BYOL/许可证绑定 → Dedicated Host，否则 Dedicated Instance；Launch Template 与 VPC 租赁任一方 dedicated → 结果 dedicated → [[EC2]]
- 🚩 S3 存储类别（4 次，已连续两批次零新增）：不看笔记复述"最短存储期限 + 检索费 + 访问模式已知/未知 + 合法转换瀑布"四个维度，复述不出的部分再回看 → [[S3]]

**优先级 3 —— Design Resilient Architectures（54%，知识面广而分散）**

不追求单点深挖，做一次覆盖面扫描：RDS Multi-AZ 同步 vs 读取副本异步、Multi-AZ 升级仍停机、存储自动扩展；ASG 健康检查/宽限期、Standby + 暂停 ReplaceUnhealthy、再平衡 vs 替换顺序；SQS 标准→FIFO 迁移；EC2 自动恢复保留项；DynamoDB PITR；Aurora Global Database → [[RDS]] [[Auto Scaling]] [[SQS]] [[DynamoDB]] [[Aurora]]

**其他 —— 做题节奏**

四次用时都在 2h6m～2h10m。批次五试行：第一遍每题 ≤ 2 分钟，不确定就标记跳过，目标 110 分钟内做完第一遍，留 20 分钟回看标记题。

---

## 使用方法

1. 每次模拟考前，先过一遍上方"按 Domain 的复习检查清单"里当前风险最高的 1-2 个 Domain
2. 模拟考后把成绩和错题发给我，我会更新 [[SAA-C03 模拟考错题本]] 并在本笔记追加新的"批次备考重点"小节
3. 达到预约标准（连续 2-3 次 ≥ 82%，四个 Domain 都 ≥ 75%）后，本笔记的检查清单可以作为考前最后一轮通读材料
