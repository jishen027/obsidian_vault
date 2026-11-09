# SAA-C03 复习测验记录

> 和 Claude 做的针对性复习测验记录，不改动 [[SAA-C03 模拟考错题本]]。题目为错题本原题的英文变体。

## 2026-11-06 · Design Secure Architectures（12 题，对 10 题，83%）

| 考点 | 结果 | 备注 |
|------|------|------|
| ACM 导入证书到期监控 | ✅ | |
| **Cognito User Pool vs Identity Pool** | ❌ | 🚩 第 2 次错，且与批次一 Q14 选了**完全相同**的错误选项（Identity Pool 做 authorizer） |
| **GuardDuty 基础数据源** | ❌ | 🚩 第 2 次错，且与批次一 Q44 选了**完全相同**的错误组合（含 API Gateway / S3 访问日志） |
| Bucket Policy 同账户用户级 + 跨账户 | ✅ | |
| ALB 入口按国家拦截 → WAF Geo Match | ✅ | |
| 审计密钥使用 → SSE-KMS | ✅ | |
| 密钥自管 + 加密交给 S3 → SSE-C | ✅ | |
| 权限边界防提权（不能挂组） | ✅ | |
| SCP 影响成员账户 root | ✅ | |
| NAT 网关放公有子网 | ✅ | |
| NACL 无状态，出站放行临时端口 | ✅ | |
| `aws:RequestedRegion` = 目标区域 | ✅ | |

### 本次重复错题的纠正口诀

- **Cognito**：User Pool 管“你是谁”（注册/登录/找回密码/MFA/用户目录，可直接做 API Gateway authorizer）；Identity Pool 管“你能拿什么 AWS 凭证”（把已认证身份换成临时 IAM 凭证，本身不管理用户）。
- **GuardDuty**：只看三件事——谁调了 API（CloudTrail 管理事件）、流量去了哪（VPC Flow Logs）、域名查了谁（DNS 日志）。选项里出现任何应用层访问日志（API Gateway、S3 server access、ELB、CloudFront、WAF）→ 整个选项排除。

## 2026-11-06 · 混合复习：复测 + 其他 Domain（14 题）

- ✅ **Cognito（Identity Pool 反向场景）复测通过**、✅ **GuardDuty 复测通过**——今天两个重复错点都已换角度答对。
- ❌ **EC2 Hibernate**：选了“每晚做 AMI”。AMI 只保存磁盘，不保存内存；要让内存里的数据在重启后还在，只有 Hibernate（把 RAM 写到加密的 EBS 根卷，启动时恢复）。口诀：**保留内存状态 / 消除启动预热 → Hibernate**。（批次三 Q20 同一考点，第 2 次错）
- ❓ **FSx for NetApp ONTAP（SMB + NFS 同一份数据）**：待确认对错。口诀：**多协议（SMB + NFS）→ ONTAP；只有 SMB → FSx for Windows；只有 NFS → EFS；高性能计算 → Lustre**。（批次四 Q16 同一考点）
- 其余（Kinesis Enhanced Fan-Out、Redshift Spectrum、S3TA、Global Accelerator 固定 IP、Alias 记录、EFS-IA、ASG 终止策略、Standby、RDS 存储自动扩展、NLB 跨 AZ 计算）未报错。

## 2026-11-06 · S3 存储类别事实回忆（12 题，对 6 题，50%）

> 场景题（价格排序、单 AZ、季度访问选 Glacier IR）都对，但**具体数字和计费规则记不牢**。

| 错误 | 我的答案 | 正确答案 |
|------|---------|---------|
| IA 最短存储期 | 90 天 | **30 天**（90 天是 Glacier IR / Flexible） |
| 哪些类别毫秒级取回 | 只有 Standard 和 IT | **除 Glacier Flexible 和 Deep Archive 外全部毫秒级**（含 Standard-IA、One Zone-IA、Glacier IR） |
| Glacier Flexible 取回时间 | Expedited 毫秒级 | **Expedited 1–5 分钟 / Standard 3–5 小时 / Bulk 5–12 小时** |
| Intelligent-Tiering 计费 | 回到高频层要收取回费 | **无取回费，按对象每月收监控和自动化费** |
| IT 对 <128 KB 对象 | 按 128 KB 计入 IA 层 | **不参与自动分层，始终按 Frequent Access 层计费** |
| IA / Glacier IR 另一条计费规则 | PUT >5 GB 收费 | **最小计费对象 128 KB** |

### 速记表

| 类别 | 最短存储期 | 取回 | 取回费 | AZ |
|------|-----------|------|-------|----|
| Standard | 无 | 毫秒 | 无 | ≥3 |
| Intelligent-Tiering | 无 | 毫秒（可选归档层除外） | 无，但按对象收监控费；<128 KB 不分层 | ≥3 |
| Standard-IA | 30 天 | 毫秒 | 有，最小计费 128 KB | ≥3 |
| One Zone-IA | 30 天 | 毫秒 | 有，最小计费 128 KB | 1 |
| Glacier Instant Retrieval | 90 天 | 毫秒 | 有（较高），最小计费 128 KB | ≥3 |
| Glacier Flexible Retrieval | 90 天 | 1–5 分 / 3–5 时 / 5–12 时 | 有（Bulk 免费） | ≥3 |
| Glacier Deep Archive | 180 天 | 12 时 / 48 时 | 有 | ≥3 |

- 数字口诀：**30 / 90 / 180**（IA / Glacier / Deep Archive）
- 时间口诀：**Flexible 是“1-5 分、3-5 时、5-12 时”，Deep Archive 是“12、48”**
- 规则口诀：**128 KB 出现两次**——IA 和 Glacier IR 最小按 128 KB 计费；IT 小于 128 KB 不分层
- 待办：明天再测一次这 6 个点

## 2026-11-06 13:44 · S3 弱点复测（8 题，对 6 题，75%，较上一轮 50% 提升）

- ✅ 已掌握：IA 最短 30 天、哪些类别需要 restore、One Zone-IA 毫秒级、Flexible Bulk 5–12 小时、IT 无取回费、IA 最小计费 128 KB
- ❌ **IT 对 <128 KB 对象**（第 2 次错，两次都和 IA 的“按 128 KB 计费”混淆）→ IT 是“不搬家”：小对象留在 Frequent Access 层、按 Frequent 价格计费、不收监控费；IA 才是“按 128 KB 收钱”
- ❌ **Glacier Flexible 10 分钟内取回**（第 2 次错，选了 Standard）→ Glacier 里的 “Standard” 是 3–5 **小时**；只有 Expedited（加急）是 1–5 **分钟**
- 待办：明天复测这 2 个点

## 2026-11-06 14:02 · 专项复习 1（重做）：IAM 评估 / VPC 访问控制 / Route 53 Resolver（12 题，对 10 题，83%）

- ✅ IAM 评估全对（显式 Deny 优先、跨账户两边都要允许、权限边界和 SCP 取交集、多条件组合推理）、Route 53 Resolver 全对、安全组引用安全组、安全组有状态
- ❌ **NACL 规则顺序**：选了“Deny 规则号更大”。NACL 按规则号**从小到大**依次匹配，**第一条命中就停止**；Deny 必须编号更小才能排在 Allow 前面生效。（安全组没有 Deny、不看顺序；NACL 有 Deny、看顺序）
- ❌ **私有子网访问 S3 最省钱且锁桶**：选了接口终端节点 + `aws:SourceIp`。S3/DynamoDB 用**网关终端节点（免费，改路由表）**；锁桶用 `aws:SourceVpce`。走 VPC 终端节点的流量没有公网源 IP，`aws:SourceIp` 不起作用

## 2026-11-06 14:47 · 专项复习 2（重做）：加密与密钥 / 边缘安全 / EC2 租赁（12 题，对 10 题，83%）

- ✅ EBS 已有卷加密走快照、共享加密 RDS 快照需客户托管密钥、导入密钥材料不能自动轮换、CloudHSM、Secrets Manager 自动轮换、OAC、Shield Advanced、WAF 限流规则、CloudFront 证书在 us-east-1、License Manager
- ❌ **SSE-KMS 请求费太高**：选了“改用 SSE-S3”，违反了题干“keeping SSE-KMS”的硬性约束。正解 **S3 Bucket Keys**：用桶级短期密钥，大幅减少对 KMS 的调用次数，仍是 SSE-KMS。（错题本已有的模式：先逐条核对题干约束再选）
- ❌ **CloudFront 付费内容、多个文件、不改 URL**：选了 S3 presigned URL（会绕过 CloudFront）。**单个文件 → Signed URL；多个文件且不改 URL → Signed Cookies**

### 今日汇总（11/6）

| 轮次 | 正确率 |
|------|-------|
| Secure 错题变体 | 10/12（83%） |
| 混合复习 | 约 12/14 |
| S3 事实回忆 | 6/12（50%） |
| S3 弱点复测 | 6/8（75%） |
| 专项复习 1 重做 | 10/12（83%） |
| 专项复习 2 重做 | 10/12（83%） |

## 2026-11-07 22:26 · 复测 11/6 全部错题（14 题，对 13 题，93%）

- ✅ 已攻克：Cognito User/Identity Pool、GuardDuty 数据源、EC2 Hibernate、FSx ONTAP、IT <128 KB、Glacier Flexible Expedited、Deep Archive 太慢退到 Flexible、IA 30 天毫秒级、NACL 规则顺序、网关终端节点、aws:SourceVpce、S3 Bucket Keys
- ❌ **CloudFront Signed URL vs Signed Cookies**（第 2 次错，方向反了：昨天该选 Cookies 选了 presigned URL，今天该选 URL 选了 Cookies）
  - 判断标准只看两点：**几个文件？要不要发一个链接？**
  - **一个文件 / 发给客户一个下载链接 → Signed URL**
  - **很多文件 / 登录后整站可看 / 不想改每个 URL → Signed Cookies**
  - 待办：11/8 再测一次

## 2026-11-09 12:12 · Mock 5 前热身（8 题，对 8 题，100%）

- ✅ 全对：Glacier Instant Retrieval（季度访问 + 毫秒级 + 多 AZ）、Route 53 Resolver Inbound（本地 → AWS）、SQS FIFO + Message Group ID、Multi-AZ 实例的备用不可读 → Read Replica、Spot、DynamoDB 网关终端节点、Failover 路由 + 健康检查、io2 Block Express（gp3 上限 16,000 IOPS）
- 两个反复错的点（S3 存储类别、Resolver 方向）本轮都对
- 注：热身题难度低于模考，不作为真实水平参考
