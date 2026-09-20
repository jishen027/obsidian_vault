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
	- [[#各 Domain 表现趋势]]
- [[#错题分布统计]]
- [[#重复出现的错题（高危薄弱点，优先复习）]]
- [[#复习优先级建议]]

---

## 模拟考分数追踪

> **目标**：真实考试通过线是缩放分数 **720/1000**（比 AIF-C01 的 700/1000 略高），65 题中同样有 15 题不计分且无法分辨。建议模拟考正确率稳定在 **80% 以上**再考虑预约，参考 AIF-C01 备考时的经验：连续 2-3 次不同题库都 ≥ 80%、没有 Domain 集中失分、高危薄弱点能脱稿复述，是比较可靠的预约信号。

### 分数记录

| 批次 | 日期 | 用时 | 题库来源 | 总题数 | 正确数 | 正确率 | 是否达标 |
|------|------|------|---------|-------|-------|--------|---------|
| | | | | | | | |

> 表格待填——完成第一次模拟考后，把总分/用时/日期发给我，我会在这里建立第一行记录，后续每次批次都会追加。

### 各 Domain 表现趋势

> 待累积至少 2-3 个批次的 Domain 明细数据后再建立趋势表和解读（参考 [[AIF-C01/AIF-C01 模拟考错题本#各 Domain 表现趋势（考试平台官方数据）]] 的格式：按批次列出各 Domain 正确率，并标注模拟考平台的 Domain 标注是否可靠）。

---

## 错题分布统计

> 待第一批次错题录入后开始统计。格式参考：按批次分表，列出 题号 / Domain（实际内容）/ 考点主题 / 状态。

---

## 重复出现的错题（高危薄弱点，优先复习）

> 待发现跨批次重复出错的知识点后建立此表——AIF-C01 备考的经验是：**同一题库换讲师/换课程后依然出错的知识点，才是真正没有内化的薄弱区**，优先级高于只错一次的题目。

---

## 复习优先级建议

> 待累积足够错题后，按易错模式（而非单纯按 Domain）归类整理，参考 [[AIF-C01/AIF-C01 模拟考错题本#复习优先级建议]] 的格式——例如"混淆同类但职责不同的 AWS 服务""把两组定义或方向同时对调""评估更先进的方案是否真的更适合"这类跨题目共性模式，比逐题记忆更有效。

---

## 相关知识笔记（已有课程笔记，按类别索引）

| 类别 | 笔记 |
|------|------|
| 计算服务 | [[AWS SAA-C02/计算服务/EC2\|EC2]] \| [[AWS SAA-C02/计算服务/Auto Scaling\|Auto Scaling]] \| [[AWS SAA-C02/计算服务/AWS Load Balance\|ELB]] \| [[AWS SAA-C02/计算服务/ECS\|ECS]] \| [[AWS SAA-C02/计算服务/EKS\|EKS]] \| [[AWS SAA-C02/计算服务/AWS Fargate\|Fargate]] \| [[AWS SAA-C02/计算服务/AWS Lambda\|Lambda]] \| [[AWS SAA-C02/计算服务/Elastic Beanstalk\|Elastic Beanstalk]] |
| 存储服务 | [[AWS SAA-C02/存储服务/S3\|S3]] \| [[AWS SAA-C02/存储服务/S3 Security\|S3 Security]] \| [[AWS SAA-C02/存储服务/S3 Glacier Archive\|S3 Glacier]] \| [[AWS SAA-C02/存储服务/EBS\|EBS]] \| [[AWS SAA-C02/存储服务/AWS EFS\|EFS]] \| [[AWS SAA-C02/存储服务/AWS FSx\|FSx]] \| [[AWS SAA-C02/存储服务/AWS Storage Gateway\|Storage Gateway]] \| [[AWS SAA-C02/存储服务/AWS DataSync\|DataSync]] \| [[AWS SAA-C02/存储服务/AWS Snowball\|Snowball]] |
| 数据库服务 | [[AWS SAA-C02/数据库服务/RDS\|RDS]] \| [[AWS SAA-C02/数据库服务/RDS Proxy\|RDS Proxy]] \| [[AWS SAA-C02/数据库服务/Aurora\|Aurora]] \| [[AWS SAA-C02/数据库服务/DynamoDB\|DynamoDB]] \| [[AWS SAA-C02/数据库服务/ElastiCache\|ElastiCache]] \| [[AWS SAA-C02/数据库服务/Redshift\|Redshift]] \| [[AWS SAA-C02/数据库服务/Amazon DocumentDB\|DocumentDB]] \| [[AWS SAA-C02/数据库服务/Amazon Neptune\|Neptune]] \| [[AWS SAA-C02/数据库服务/Amazon Keyspaces\|Keyspaces]] \| [[AWS SAA-C02/数据库服务/Amazon Timestream\|Timestream]] |
| 网络服务 | [[AWS SAA-C02/网络服务/VPC\|VPC]] \| [[AWS SAA-C02/网络服务/Subnet\|Subnet]] \| [[AWS SAA-C02/网络服务/Route Table\|Route Table]] \| [[AWS SAA-C02/网络服务/Security Group\|Security Group]] \| [[AWS SAA-C02/网络服务/NACL\|NACL]] \| [[AWS SAA-C02/网络服务/NAT Gateway\|NAT Gateway]] \| [[AWS SAA-C02/网络服务/Internet Gateway\|Internet Gateway]] \| [[AWS SAA-C02/网络服务/VPC Peering\|VPC Peering]] \| [[AWS SAA-C02/网络服务/Transit Gateway\|Transit Gateway]] \| [[AWS SAA-C02/网络服务/VPC Endpoints\|VPC Endpoints]] \| [[AWS SAA-C02/网络服务/Direct Connect\|Direct Connect]] \| [[AWS SAA-C02/网络服务/Site to Site VPN\|Site-to-Site VPN]] \| [[AWS SAA-C02/网络服务/CloudFront\|CloudFront]] \| [[AWS SAA-C02/网络服务/Route 53 DNS\|Route 53]] |
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
