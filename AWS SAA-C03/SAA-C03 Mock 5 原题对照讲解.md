# SAA-C03 Mock 5 错题：原题对照讲解

> 批次五（2026-11-10）共 24 题：23 道错题 + 1 道未作答（Q65）。成绩 **63%（41/65）**。每题先列英文原题和选项（✅ 正确答案、❌ 你的选择），再用中文讲解。
> 完整的分类统计和趋势见 [[SAA-C03 模拟考错题本#批次五（23 题，4 处原标注调整）]]。

**选项标记**：✅ **正确答案** = 正确但你没选 · ✅ **你选对了** · ❌ **你的选择** = 选错了 · ▫️ = 其他选项

**🚩 原题重现**：Q18（= 批次一 Q65，选了同一个错误选项）、Q21（= 批次一 Q50）

---

## Q14 · Cost-Optimized（平台标注）

### 原题

> You have developed a new REST API leveraging the Amazon API Gateway, AWS Lambda and Amazon Aurora database services. Most of the workload on the website is read-heavy. The data rarely changes and it is acceptable to serve users outdated data for about 24 hours. Recently, the website has been experiencing high load and the costs incurred on the Aurora database have been very high.
>
> How can you easily reduce the costs while improving performance, with minimal changes?

- ▫️ Enable AWS Lambda In Memory Caching
- ▫️ Switch to using an Application Load Balancer
- ❌ **你的选择** Add Amazon Aurora Read Replicas
- ✅ **正确答案** Enable Amazon API Gateway Caching

### 讲解

**题眼**：题干说了“可以接受约 24 小时的旧数据”+“降本”+“改动最少”

**错在哪**：你选了加 Aurora 读副本。读副本确实能分担读压力，但它是**再加一台数据库实例**，成本只会更高，直接违反了“reduce the costs”。

**正确思路**：“可以接受旧数据”就是在提示加缓存。API Gateway 缓存放在最前面，命中缓存时 Lambda 和 Aurora 都不会被调用，所以既降本又提速，而且只需在 Stage 上打开一个开关，代码不用改。

**其他选项**：
- Lambda In Memory Caching：Lambda 没有这个原生功能，是干扰项
- 换成 ALB：负载均衡解决不了数据库读压力

> 💡 **口诀**：可接受旧数据 + 读多写少 + 最少改动 → API Gateway Caching（默认 TTL 300 秒，最大 3600 秒）

---

## Q15 · Resilient（平台标注）

### 原题

> A company has migrated its application from a monolith architecture to a microservices based architecture. The development team has updated the Amazon Route 53 simple record to point "myapp.mydomain.com" from the old Load Balancer to the new one.
>
> The users are still not redirected to the new Load Balancer. What has gone wrong in the configuration?

- ✅ **正确答案** The Time To Live (TTL) is still in effect
- ▫️ The health checks are failing
- ❌ **你的选择** The Alias Record is misconfigured
- ▫️ The CNAME Record is misconfigured

### 讲解

**题眼**：改了 Simple 记录，用户还是去旧的 LB

**错在哪**：你选了“Alias 记录配置错误”。题干只说更新了记录，没有任何信息表明记录本身写错了。

**正确思路**：DNS 解析器会按 TTL 缓存旧答案，在 TTL 到期之前，客户端拿到的都是旧 LB 的地址。正确做法是：改之前先把 TTL 调小（如 300 秒），改完确认没问题再调回去。

**其他选项**：
- 健康检查失败：Simple 路由**不支持健康检查**，直接排除
- CNAME 配置错误：同样没有依据

> 💡 **口诀**：改了 DNS 不生效，先想 TTL

---

## Q18 · High-Performing（平台标注） 🚩原题重现

### 原题

> A Big Data analytics company writes data and log files in Amazon S3 buckets. The company now wants to stream the existing data files as well as any ongoing file updates from Amazon S3 to Amazon Kinesis Data Streams.
>
> As a Solutions Architect, which of the following would you suggest as the fastest possible way of building a solution for this requirement?

- ▫️ Amazon S3 bucket actions can be directly configured to write data into Amazon Simple Notification Service (Amazon SNS). Amazon SNS can then be used to send the updates to Amazon Kinesis Data Streams
- ✅ **正确答案** Leverage AWS Database Migration Service (AWS DMS) as a bridge between Amazon S3 and Amazon Kinesis Data Streams
- ❌ **你的选择** Configure Amazon EventBridge events for the bucket actions on Amazon S3. An AWS Lambda function can then be triggered from the Amazon EventBridge event that will send the necessary data to Amazon Kinesis Data Streams
- ▫️ Leverage Amazon S3 event notification to trigger an AWS Lambda function for the file create event. The AWS Lambda function will then send the necessary data to Amazon Kinesis Data Streams

### 讲解

**题眼**：已有数据 + 持续更新，S3 → Kinesis，要求“fastest possible way of building”

**错在哪**：你选了 EventBridge + Lambda。**这和批次一 Q65 选的是同一个错误选项。**Lambda 方案可行，但读文件、解析、写 Kinesis、补历史数据都要自己写代码，不是最快的。

**正确思路**：DMS 原生支持 **S3 作为源、Kinesis Data Streams 作为目标**，用 Full load（全量加载：先把已有的数据整体搬一遍）+ CDC（Change Data Capture，变更数据捕获：之后持续捕获源端新增和修改的数据，并实时同步过去）一次覆盖历史数据和后续变更，开箱即用，不写代码。

**其他选项**：
- S3 事件通知 + Lambda：同样要自己写代码
- S3 → SNS → Kinesis：S3 不能“直接写入” SNS，SNS 也不能把消息发到 Kinesis Data Streams

> 💡 **口诀**：S3 → Kinesis，历史 + 持续变更 = DMS

---

## Q20 · Secure（平台标注）

### 原题

> A global pharmaceutical company operates a hybrid cloud network. Its primary AWS workloads run in the us-west-2 Region, connected to its on-premises data center via an AWS Direct Connect connection. After acquiring a biotech firm headquartered in Europe, the company must integrate the biotech’s workloads, which are hosted in several VPCs in the eu-central-1 Region and connected to the biotech's on-premises facility through a separate Direct Connect link. All CIDR blocks are non-overlapping, and the business requires full connectivity between both data centers and all VPCs across the two Regions. The company also wants a scalable solution that minimizes manual network configuration and long-term operational overhead.
>
> Which solution will best meet these requirements?

- ▫️ Deploy EC2-based VPN appliances in each VPC. Configure a full mesh VPN topology between all VPCs and data centers using CloudHub-style routing across Regions
- ▫️ Establish inter-Region VPC peering between each VPC in the us-west-2 and eu-central-1 Regions. Use static routing tables in each VPC to define peer relationships and enable cross-Region communication
- ❌ **你的选择** Create private VIFs (virtual interfaces) in each Region and associate them directly with foreign-region VPCs using routing table entries and BGP. Use VPC endpoints in each account to forward cross-Region traffic
- ✅ **正确答案** Connect both Direct Connect links to a shared Direct Connect gateway. Attach each Region's virtual private gateway (VGW) to the Direct Connect gateway, enabling transitive routing between the VPCs and the on-premises networks across Regions

### 讲解

**题眼**：两个 Region、两条 Direct Connect、多个 VPC 全互通，要求可扩展、运维最少

**错在哪**：你选了“Private VIF 直接关联到其他 Region 的 VPC + VPC Endpoint 转发”。这里有两处错：Private VIF 只能连**同一个 Region** 的 VGW/TGW；VPC Endpoint 是访问 AWS 服务用的，不能转发跨区域的私网流量。

**正确思路**：Direct Connect Gateway 是**全局资源**：两条专线的 Private VIF 都接到同一个 DX Gateway，再把两个 Region 的 VGW 关联上去，本地和所有 VPC 就能跨 Region 互通。

**其他选项**：
- 跨区域 VPC Peering：不支持传递路由，VPC 一多，对等连接和路由表的数量会爆炸
- EC2 上自建 VPN 全网状：运维负担最重

> 💡 **口诀**：跨 Region + Direct Connect → DX Gateway（全局）。本题是平台标成 Secure 的网络题，按内容应归到 High-Performing

---

## Q21 · Resilient（平台标注） 🚩原题重现

### 原题

> An e-commerce company has copied 1 petabyte of data from its on-premises data center to an Amazon S3 bucket in the us-west-1 Region using an AWS Direct Connect link. The company now wants to set up a one-time copy of the data to another Amazon S3 bucket in the us-east-1 Region. The on-premises data center does not allow the use of AWS Snowball.
>
> As a Solutions Architect, which of the following options can be used to accomplish this goal? (Select two)

- ▫️ Copy data from the source Amazon S3 bucket to a target Amazon S3 bucket using the S3 console
- ▫️ Use AWS Snowball Edge device to copy the data from one Region to another Region
- ✅ **正确答案** Copy data from the source bucket to the destination bucket using the aws S3 sync command
- ✅ **你选对了** Set up Amazon S3 batch replication to copy objects across Amazon S3 buckets in another Region using S3 console and then delete the replication configuration
- ❌ **你的选择** Set up Amazon S3 Transfer Acceleration (Amazon S3TA) to copy objects across Amazon S3 buckets in different Regions using S3 console

### 讲解

**题眼**：1PB、us-west-1 → us-east-1、一次性复制、不能用 Snowball

**错在哪**：你选了 Batch Replication（对）+ S3TA（错）。S3 Transfer Acceleration 加速的是**客户端到桶**的远距离上传下载，不能用来做桶到桶复制。**这是批次一 Q50 的原题**，上次第二个选项错选了“控制台复制”。两次都只记住了 Batch Replication，没选 `s3 sync`。

**正确思路**：桶到桶一次性复制有两种原生方式：`aws s3 sync`（用 CopyObject API，失败了可以重跑）；S3 Batch Replication（专门复制**已有**对象，做完把复制配置删掉）。

**其他选项**：
- S3 控制台复制：PB 级数据不现实
- Snowball Edge：题干明确排除，而且这是 Region 到 Region 的复制，用不上物理设备

> 💡 **口诀**：一次性桶到桶 = s3 sync + Batch Replication

---

## Q25 · Cost-Optimized（平台标注）

### 原题

> A healthcare provider is experiencing rapid data growth in its on-premises servers due to increased patient imaging and record retention requirements. The organization wants to extend its storage capacity to AWS in a way that preserves quick access to critical records, including from its local file systems. The company must optimize bandwidth usage during migration and avoid any retrieval fees or delays when accessing the data in the cloud. The provider wants a hybrid cloud solution that requires minimal application reconfiguration, allows frequent local access to key datasets, and ensures that cloud storage costs remain predictable without paying extra for data retrieval.
>
> Which AWS solution best meets these requirements?

- ▫️ Set up Amazon S3 Standard-Infrequent Access (S3 Standard-IA) as the primary storage tier. Configure the on-premises file server to replicate changes to the S3 bucket using AWS DataSync for asynchronous updates
- ▫️ Deploy AWS Storage Gateway using stored volumes. Retain the full dataset on-premises and asynchronously back up point-in-time snapshots to Amazon S3. Configure applications to read from the local volume and recover data from the cloud if needed
- ❌ **你的选择** Implement Amazon FSx for Windows File Server and configure on-premises servers to mount the file system using a VPN connection. Store all primary data in FSx and use it as the central NAS replacement
- ✅ **正确答案** Deploy AWS Storage Gateway using cached volumes. Store frequently accessed data locally, while writing all primary data asynchronously to Amazon S3

### 讲解

**题眼**：本地容量不够，要扩展到 AWS；常用数据要本地快速访问；节省带宽；没有取回费；应用改动最少

**错在哪**：你选了 FSx for Windows 通过 VPN 挂载作为主 NAS。这样每次读写都要过 VPN，**不省带宽、延迟高**，和“频繁本地访问”冲突。

**正确思路**：Storage Gateway **缓存卷**：主数据放在 S3，本地只缓存常用的那部分。本地存储压力变小了，热数据访问又快，数据在 S3 里也没有取回费。

**其他选项**：
- 存储卷（stored volumes）：完整数据留在本地，只把快照备份到 S3，**本地存储不会减少**，正好违反“扩展容量”
- S3 Standard-IA + DataSync：IA 有取回费，DataSync 也不能提供实时的本地访问

> 💡 **口诀**：缓存卷 = 主数据在云、本地放热数据；存储卷 = 主数据在本地、云端放备份

---

## Q29 · Resilient（平台标注）

### 原题

> Your company is deploying a website running on AWS Elastic Beanstalk. The website takes over 45 minutes for the installation and contains both static as well as dynamic files that must be generated during the installation process.
>
> As a Solutions Architect, you would like to bring the time to create a new instance in your AWS Elastic Beanstalk deployment to be less than 2 minutes. Which of the following options should be combined to build a solution for this requirement? (Select two)

- ✅ **正确答案** Use Amazon EC2 user data to customize the dynamic installation parts at boot time
- ✅ **你选对了** Create a Golden Amazon Machine Image (AMI) with the static installation components already setup
- ❌ **你的选择** Use AWS Elastic Beanstalk deployment caching feature
- ▫️ Use Amazon EC2 user data to install the application at boot time
- ▫️ Store the installation files in Amazon S3 so they can be quickly retrieved

### 讲解

**题眼**：安装要 45 分钟，有静态文件也有动态文件，目标是新实例 2 分钟内就绪（选二）

**错在哪**：你选了 Golden AMI（对）+ “Elastic Beanstalk deployment caching”（错）。**这个功能根本不存在**，是编出来的干扰项。

**正确思路**：把静态和动态分开处理：静态部分提前做进 Golden AMI；动态部分在启动时用 EC2 User Data 生成，只做少量的定制工作。

**其他选项**：
- 用 User Data 安装整个应用：还是要 45 分钟
- 把安装文件放到 S3：下载快了一点，但安装本身仍然很慢

> 💡 **口诀**：静态的做进 AMI，动态的交给 User Data。原标注 Resilient，按内容应归到 High-Performing

---

## Q30 · High-Performing（平台标注）

### 原题

> A company's business logic is built on several microservices that are running in the on-premises data center. They currently communicate using a message broker that supports the MQTT protocol. The company is looking at migrating these applications and the message broker to AWS Cloud without changing the application logic.
>
> Which technology allows you to get a managed message broker that supports the MQTT protocol?

- ✅ **正确答案** Amazon MQ
- ▫️ Amazon Simple Queue Service (Amazon SQS)
- ❌ **你的选择** Amazon Simple Notification Service (Amazon SNS)
- ▫️ Amazon Kinesis Data Streams

### 讲解

**题眼**：现有消息代理用 MQTT；迁移上云；不改应用逻辑；要托管服务

**错在哪**：你选了 SNS。SNS 是 AWS 自己的 pub/sub API，**不支持 MQTT**，应用必须改代码。

**正确思路**：Amazon MQ 是托管的 ActiveMQ / RabbitMQ，支持 JMS、AMQP、STOMP、MQTT、OpenWire、WebSocket 这些行业标准协议，应用只需要换一下连接地址。

**其他选项**：
SQS、SNS、Kinesis 都是 AWS 专有 API，都不支持 MQTT

> 💡 **口诀**：看到 MQTT / AMQP / JMS + 不改代码 → Amazon MQ

---

## Q31 · Resilient（平台标注）

### 原题

> A CRM company has a software as a service (SaaS) application that feeds updates to other in-house and third-party applications. The SaaS application and the in-house applications are being migrated to use AWS services for this inter-application communication.
>
> As a Solutions Architect, which of the following would you suggest to asynchronously decouple the architecture?

- ▫️ Use Elastic Load Balancing (ELB) for effective decoupling of system architecture
- ❌ **你的选择** Use Amazon Simple Queue Service (Amazon SQS) to decouple the architecture
- ✅ **正确答案** Use Amazon EventBridge to decouple the system architecture
- ▫️ Use Amazon Simple Notification Service (Amazon SNS) to communicate between systems and decouple the architecture

### 讲解

**题眼**：SaaS 应用要把更新推送给内部和**第三方**应用，要求异步解耦

**错在哪**：你选了 SQS。SQS 能做解耦，但它**不能直接接入第三方 SaaS 事件源**，题目里的 SaaS 这个条件没有满足。

**正确思路**：EventBridge 是唯一原生支持第三方 SaaS 合作伙伴事件源的服务，还能按事件内容写规则，把事件路由给不同的目标。

**其他选项**：
- SNS：同样没有 SaaS 集成
- ELB：同步调用，不算异步解耦

> 💡 **口诀**：SaaS / 第三方事件 → EventBridge

---

## Q32 · Cost-Optimized（平台标注）

### 原题

> As an e-sport tournament hosting company, you have servers that need to scale and be highly available. Therefore you have deployed an Elastic Load Balancing (ELB) with an Auto Scaling group (ASG) across 3 Availability Zones (AZs). When e-sport tournaments are running, the servers need to scale quickly. And when tournaments are done, the servers can be idle. As a general rule, you would like to be highly available, have the capacity to scale and optimize your costs.
>
> What do you recommend? (Select two)

- ❌ **你的选择** Set the minimum capacity to 3
- ✅ **正确答案** Set the minimum capacity to 2
- ▫️ Set the minimum capacity to 1
- ▫️ Use Dedicated hosts for the minimum capacity
- ✅ **你选对了** Use Reserved Instances (RIs) for the minimum capacity

### 讲解

**题眼**：ASG 跨 3 个 AZ；比赛时要快速扩容、平时空闲；要求高可用 + 能扩展 + 成本最优（选二）

**错在哪**：你选了最小容量 3（错）+ RI（对）。设成 3 当然高可用，但题干要求“optimize your costs”，2 台已经能满足高可用，第 3 台就是多花的钱。

**正确思路**：最小容量 2：分布在两个 AZ，一个 AZ 故障时还有一台在服务，ASG 会在第三个 AZ 补一台回来。最小容量这部分实例会一直运行，所以用 RI 最划算。

**其他选项**：
- 最小容量 1：单点故障
- Dedicated Host：没有许可证绑定的需求

> 💡 **口诀**：高可用的最小值 = 2 台、跨 2 个 AZ；常驻的那部分用 RI

---

## Q33 · Cost-Optimized（平台标注）

### 原题

> A genomics research firm is processing sporadic bursts of data-intensive workloads using Amazon EC2 instances. The shared storage must support unpredictable spikes in file operations, but the average daily throughput demand remains relatively low. The team has selected Amazon Elastic File System (EFS) for its scalability and wants to ensure optimal cost and performance during bursts without provisioning throughput manually.
>
> Which approach should the team take to best meet these requirements?

- ▫️ Enable EFS Infrequent Access (IA) storage class to reduce storage cost while continuing to benefit from burst throughput mode
- ▫️ Change the throughput mode to provisioned and configure the desired throughput value to support burst workloads
- ❌ **你的选择** Switch the EFS storage class to EFS One Zone to reduce cost, which will automatically enable burst throughput mode
- ✅ **正确答案** Enable EFS burst throughput mode on the file system using the General Purpose performance mode and EFS Standard storage class

### 讲解

**题眼**：偶发的突发负载、平均吞吐低；用 EFS；不想手动配置吞吐

**错在哪**：你选了“切换到 One Zone，会自动启用突发吞吐模式”。One Zone 是**存储类别**，突发 / 预置是**吞吐模式**，两者互相独立，切换 One Zone 不会改变吞吐模式。One Zone 还会失去多 AZ 冗余。

**正确思路**：General Purpose 性能模式 + Standard 存储类别 + Bursting 吞吐模式：通过积累信用来应对突发，不需要手动配置。

**其他选项**：
- Provisioned 模式：要手动设吞吐值，不管用不用都要付费
- IA 存储类别：只能降低存储费，和吞吐无关，访问时还要另外收费

> 💡 **口诀**：存储类别（Standard / One Zone / IA）和吞吐模式（Bursting / Provisioned / Elastic）是两个独立的维度

---

## Q34 · Resilient（平台标注）

### 原题

> An e-commerce company wants to migrate its on-premises application to AWS. The application consists of application servers and a Microsoft SQL Server database. The solution should result in the maximum possible availability for the database layer while minimizing operational and management overhead.
>
> As a solutions architect, which of the following would you recommend to meet the given requirements?

- ❌ **你的选择** Migrate the data to Amazon RDS for SQL Server database in a cross-region Multi-AZ deployment
- ✅ **正确答案** Migrate the data to Amazon RDS for SQL Server database in a Multi-AZ deployment
- ▫️ Migrate the data to Amazon EC2 instance hosted SQL Server database. Deploy the Amazon EC2 instances in a Multi-AZ configuration
- ▫️ Migrate the data to Amazon RDS for SQL Server database in a cross-region read-replica configuration

### 讲解

**题眼**：SQL Server；数据库层可用性最高；运维最少

**错在哪**：你选了“跨区域 Multi-AZ”。**RDS Multi-AZ 只在一个 Region 内，不存在 cross-region Multi-AZ**，这是编出来的选项。

**正确思路**：RDS for SQL Server Multi-AZ：自动故障转移，终端节点不变，补丁和复制都由 AWS 托管。

**其他选项**：
- 跨区域读副本：用来扩展读取，不提高可用性
- 在 EC2 上自建：要自己打 OS 和数据库补丁

> 💡 **口诀**：Multi-AZ 永远在同一个 Region 内；跨 Region 只有读副本和 Aurora Global Database

---

## Q37 · High-Performing（平台标注）

### 原题

> A retail company is using AWS Site-to-Site VPN connections for secure connectivity to its AWS cloud resources from its on-premises data center. Due to a surge in traffic across the VPN connections to the AWS cloud, users are experiencing slower VPN connectivity.
>
> Which of the following options will maximize the VPN throughput?

- ✅ **正确答案** Create an AWS Transit Gateway with equal cost multipath routing and add additional VPN tunnels
- ▫️ Use AWS Global Accelerator for the VPN connection to maximize the throughput
- ▫️ Use Transfer Acceleration for the VPN connection to maximize the throughput
- ❌ **你的选择** Create a virtual private gateway with equal cost multipath routing and multiple channels

### 讲解

**题眼**：Site-to-Site VPN 流量激增变慢，要最大化吞吐

**错在哪**：你选了“支持 ECMP 的虚拟私有网关”。**VGW 不支持 ECMP**，这个选项描述的功能不存在。

**正确思路**：单条 VPN 隧道的上限约 1.25 Gbps。要突破这个上限，只能用 **Transit Gateway + ECMP**，在多条隧道之间做等价多路径（需要开启动态路由 / BGP）。

**其他选项**：
- Global Accelerator：加速型 VPN 改善的是网络路径质量，不能提高吞吐上限
- Transfer Acceleration：这是 S3 的功能，和 VPN 无关

> 💡 **口诀**：VPN 吞吐不够 → TGW + ECMP + 多隧道

---

## Q40 · Resilient（平台标注）

### 原题

> A media company operates a video rendering pipeline on Amazon EKS, where containerized jobs are scheduled using Kubernetes deployments. The application experiences bursty traffic patterns, particularly during peak streaming hours. The platform uses the Kubernetes Horizontal Pod Autoscaler (HPA) to scale pods based on CPU utilization. A solutions architect observes that the total number of EC2 worker nodes remains constant during traffic spikes, even when all nodes are at maximum resource utilization. The company needs a solution that enables automatic scaling of the underlying compute infrastructure when pod demand exceeds cluster capacity.
>
> Which solution should the architect implement to resolve this issue with the least operational overhead?

- ✅ **正确答案** Deploy the Kubernetes Cluster Autoscaler to the EKS cluster. Configure it to integrate with the existing EC2 Auto Scaling group to automatically launch or terminate nodes based on pending pod demands
- ▫️ Use AWS Fargate to replace the EKS worker nodes with serverless compute profiles, allowing Fargate to scale pods automatically without managing EC2 infrastructure
- ❌ **你的选择** Enable Amazon EC2 Auto Scaling with custom CloudWatch alarms based on cluster-wide CPU and memory usage to dynamically adjust node count in the EKS worker node group
- ▫️ Implement an AWS Lambda function that runs every 10 minutes and checks EKS pod scheduling status. Trigger node scaling manually using the eksctl CLI or AWS SDK if unschedulable pods are detected

### 讲解

**题眼**：EKS 上 HPA 能扩 Pod，但节点数不变；Pod 需求超过集群容量时要自动扩节点；运维最少

**错在哪**：你选了 EC2 Auto Scaling + CloudWatch CPU/内存告警。EC2 层面的指标**反映不出 Pod 有没有因为资源不够而调度失败**，比如资源碎片化时，CPU 不高，Pod 照样处于 Pending。

**正确思路**：Kubernetes Cluster Autoscaler 直接监控“调度不上的 Pod”，自动调整对应 ASG 的期望容量，也能在节点空闲时缩容。（现在也可以用 Karpenter）

**其他选项**：
- Lambda 定时检查：自己写轮询脚本，运维负担重
- 换成 Fargate：需要改造应用，而且 GPU 等工作负载不兼容

> 💡 **口诀**：HPA 扩 Pod，Cluster Autoscaler / Karpenter 扩节点

---

## Q42 · Secure（平台标注）

### 原题

> A junior developer has downloaded a sample Amazon S3 bucket policy to make changes to it based on new company-wide access policies. He has requested your help in understanding this bucket policy.
>
> As a Solutions Architect, which of the following would you identify as the correct description for the given policy?

```json
{
 "Version": "2012-10-17",
 "Id": "S3PolicyId1",
 "Statement": [
   {
     "Sid": "IPAllow",
     "Effect": "Allow",
     "Principal": "*",
     "Action": "s3:*",
     "Resource": "arn:aws:s3:::examplebucket/*",
     "Condition": {
        "IpAddress": {"aws:SourceIp": "54.240.143.0/24"},
        "NotIpAddress": {"aws:SourceIp": "54.240.143.188/32"}
     }
   }
 ]
}
```

- ▫️ It ensures Amazon EC2 instances that have inherited a security group can access the bucket
- ✅ **正确答案** It authorizes an entire Classless Inter-Domain Routing (CIDR) except one IP address to access the Amazon S3 bucket
- ❌ **你的选择** It ensures the Amazon S3 bucket is exposing an external IP within the Classless Inter-Domain Routing (CIDR) range specified, except one IP
- ▫️ It authorizes an IP address and a Classless Inter-Domain Routing (CIDR) to access the S3 bucket

### 讲解

**题眼**：同一个 Condition 块里同时有 IpAddress 和 NotIpAddress

**错在哪**：你选了“让 S3 桶暴露该 CIDR 内的外部 IP”。桶策略控制的是**谁能访问桶**，S3 桶本身没有“暴露 IP”这种概念。

**正确思路**：同一个 Condition 块里的多个条件是 **AND** 关系：源 IP 在 54.240.143.0/24 内，**并且**不是 .188，才放行。也就是整段 CIDR 都能访问，唯独排除一个 IP。

**其他选项**：
- “授权一个 IP 和一个 CIDR”：把 NotIpAddress 读反了
- “继承了安全组的 EC2”：桶策略和安全组没有关系

> 💡 **口诀**：读策略按 Effect → Principal → Action → Resource → Condition 逐段翻译；同一个块里的条件取 AND

---

## Q44 · Cost-Optimized（平台标注）

### 原题

> You are working as a Solutions Architect for a photo processing company that has a proprietary algorithm to compress an image without any loss in quality. Because of the efficiency of the algorithm, your clients are willing to wait for a response that carries their compressed images back. You also want to process these jobs asynchronously and scale quickly, to cater to the high demand. Additionally, you also want the job to be retried in case of failures.
>
> Which combination of choices do you recommend to minimize cost and comply with the requirements? (Select two)

- ❌ **你的选择** Amazon EC2 On-Demand Instances
- ▫️ Amazon EC2 Reserved Instances (RIs)
- ✅ **你选对了** Amazon Simple Queue Service (Amazon SQS)
- ✅ **正确答案** Amazon EC2 Spot Instances
- ▫️ Amazon Simple Notification Service (Amazon SNS)

### 讲解

**题眼**：客户愿意等结果；要求异步、快速扩展、失败重试；**成本最低**（选二）

**错在哪**：你选了 SQS（对）+ On-Demand（错）。题干写了 minimize cost，而 On-Demand 不是最便宜的。

**正确思路**：SQS 负责缓冲和重试：消息的可见性超时到期后会重新出现在队列里，所以 Spot 实例被中断时，没做完的任务会被别的实例重新处理。客户又愿意等，这正是 Spot 的典型使用场景。

**其他选项**：
- RI：需要 1 到 3 年承诺，不适合这种负载不可预测的场景
- SNS：只推送消息，不提供缓冲和重试

> 💡 **口诀**：可中断 + 有队列能重试 + 最省钱 → SQS + Spot

---

## Q49 · Secure（平台标注）

### 原题

> A retail company uses AWS Cloud to manage its technology infrastructure. The company has deployed its consumer-focused web application on Amazon EC2-based web servers and uses Amazon RDS PostgreSQL database as the data store. The PostgreSQL database is set up in a private subnet that allows inbound traffic from selected Amazon EC2 instances. The database also uses AWS Key Management Service (AWS KMS) for encrypting data at rest.
>
> Which of the following steps would you recommend to facilitate end-to-end security for the data-in-transit while accessing the database?

- ❌ **你的选择** Create a new network access control list (network ACL) that blocks SSH from the entire Amazon EC2 subnet into the database
- ✅ **正确答案** Configure Amazon RDS to use SSL for data in transit
- ▫️ Use IAM authentication to access the database instead of the database user's access credentials
- ▫️ Create a new security group that blocks SSH from the selected Amazon EC2 instances into the database

### 讲解

**题眼**：问的是 **data-in-transit**（传输中的数据）加密

**错在哪**：你选了“新建 NACL 阻止 SSH”。NACL 和安全组只做访问控制，不提供加密；而且 **RDS 根本不能 SSH 登录**。

**正确思路**：在 RDS 上启用 SSL/TLS，让客户端和数据库之间的连接加密。题干里的 KMS 只负责静态加密（at rest），传输加密要另外配置。

**其他选项**：
- IAM 数据库认证：只是换了一种认证方式，不提供传输加密
- 用安全组阻止 SSH：同理，RDS 没有 SSH

> 💡 **口诀**：传输中加密 → SSL/TLS；静态加密 → KMS

---

## Q51 · Resilient（平台标注）

### 原题

> You are working for a software as a service (SaaS) company as a solutions architect and help design solutions for the company's customers. One of the customers is a bank and has a requirement to whitelist a public IP when the bank is accessing external services across the internet.
>
> Which architectural choice do you recommend to maintain high availability, support scaling-up to 10 instances and comply with the bank's requirements?

- ▫️ Use a Classic Load Balancer with an Auto Scaling Group
- ✅ **正确答案** Use a Network Load Balancer with an Auto Scaling Group
- ❌ **你的选择** Use an Application Load Balancer with an Auto Scaling Group
- ▫️ Use an Auto Scaling Group with Dynamic Elastic IPs attachment

### 讲解

**题眼**：银行需要把**固定公网 IP** 加入白名单；要求高可用、可扩展到 10 台

**错在哪**：你选了 ALB + ASG。**ALB 只提供 DNS 名，IP 会变**，银行没法写进白名单。

**正确思路**：NLB 在每个 AZ 都有固定 IP，还可以绑定 EIP，后面接 ASG 一样可以扩展。

**其他选项**：
- CLB：同样只提供 DNS
- “ASG 动态绑定 EIP”：不存在这个功能

> 💡 **口诀**：要固定 IP → NLB；必须用 ALB → 前面加 Global Accelerator（批次四 Q60）

---

## Q54 · Resilient（平台标注）

### 原题

> An IT company has a large number of clients opting to build their application programming interface (API) using Docker containers. To facilitate the hosting of these containers, the company is looking at various orchestration services available with AWS.
>
> As a Solutions Architect, which of the following solutions will you suggest? (Select two)

- ❌ **你的选择** Use Amazon Elastic Container Service (Amazon ECS) with Amazon EC2 for serverless orchestration of the containerized services
- ✅ **你选对了** Use Amazon Elastic Container Service (Amazon ECS) with AWS Fargate for serverless orchestration of the containerized services
- ▫️ Use Amazon EMR for serverless orchestration of the containerized services
- ▫️ Use Amazon SageMaker for serverless orchestration of the containerized services
- ✅ **正确答案** Use Amazon Elastic Kubernetes Service (Amazon EKS) with AWS Fargate for serverless orchestration of the containerized services

### 讲解

**题眼**：Docker 容器 API；要求**无服务器**编排（选二）

**错在哪**：你选了 ECS + Fargate（对）+ ECS + EC2（错）。底层是 EC2 就要自己管理实例，不算无服务器。

**正确思路**：Fargate 可以配合 ECS 用，也可以配合 EKS 用，所以答案是 ECS + Fargate 和 EKS + Fargate。

**其他选项**：
EMR 是大数据服务，SageMaker 是机器学习服务，都不是容器编排

> 💡 **口诀**：Serverless 容器 = Fargate（ECS 和 EKS 都能用）。原标注 Resilient，按内容应归到 High-Performing

---

## Q57 · Resilient（平台标注）

### 原题

> The engineering team at a social media company has recently migrated to AWS Cloud from its on-premises data center. The team is evaluating Amazon CloudFront to be used as a CDN for its flagship application. The team has hired you as an AWS Certified Solutions Architect – Associate to advise on Amazon CloudFront capabilities on routing, security, and high availability.
>
> Which of the following would you identify as correct regarding Amazon CloudFront? (Select three)

- ✅ **正确答案** Use field level encryption in Amazon CloudFront to protect sensitive data for specific content
- ▫️ Use geo restriction to configure Amazon CloudFront for high-availability and failover
- ✅ **你选对了** Amazon CloudFront can route to multiple origins based on the content type
- ▫️ Amazon CloudFront can route to multiple origins based on the price class
- ✅ **你选对了** Use an origin group with primary and secondary origins to configure Amazon CloudFront for high-availability and failover
- ▫️ Use AWS Key Management Service (AWS KMS) encryption in Amazon CloudFront to protect sensitive data for specific content

### 讲解

**题眼**：CloudFront 的路由、安全、高可用能力，选三项正确的

**错在哪**：你选对了“按内容类型路由到多个源”和“Origin Group 主备故障转移”，但漏选了**字段级加密**。

**正确思路**：CloudFront 保护特定敏感字段用的是 Field-Level Encryption：在边缘节点用公钥加密 POST 请求里指定的字段（最多 10 个），只有拥有私钥的后端才能解密。

**其他选项**：
- “CloudFront 用 KMS 加密”：干扰项
- 地理限制：用来封锁国家，和高可用无关
- Price Class：决定用哪些边缘站点，不决定路由到哪个源

> 💡 **口诀**：CloudFront 高可用 = Origin Group；保护敏感字段 = Field-Level Encryption

---

## Q61 · High-Performing（平台标注）

### 原题

> A social media company wants the capability to dynamically alter the size of a geographic area from which traffic is routed to a specific server resource.
>
> Which feature of Amazon Route 53 can help achieve this functionality?

- ▫️ Latency-based routing
- ▫️ Weighted routing
- ❌ **你的选择** Geolocation routing
- ✅ **正确答案** Geoproximity routing

### 讲解

**题眼**：要能动态**调整地理区域的大小**

**错在哪**：你选了 Geolocation。Geolocation 按用户所在的洲 / 国家固定对应到资源，区域大小不能调。

**正确思路**：Geoproximity 有 **bias** 参数：正值（1~99）扩大一个资源覆盖的区域，负值（-1~-99）缩小。

**其他选项**：
Latency 按延迟路由，Weighted 按比例分配，都和地理区域大小无关

> 💡 **口诀**：调整区域大小 = bias = Geoproximity

---

## Q62 · Cost-Optimized（平台标注）

### 原题

> You have an Amazon S3 bucket that contains files in two different folders - s3://my-bucket/images and s3://my-bucket/thumbnails. When an image is first uploaded and new, it is viewed several times. But after 45 days, analytics prove that image files are on average rarely requested, but the thumbnails still are. After 180 days, you would like to archive the image files and the thumbnails. Overall you would like the solution to remain highly available to prevent disasters happening against a whole Availability Zone (AZ).
>
> How can you implement an efficient cost strategy for your Amazon S3 bucket? (Select two)

- ▫️ Create a Lifecycle Policy to transition all objects to Amazon S3 Standard IA after 45 days
- ✅ **你选对了** Create a Lifecycle Policy to transition objects to Amazon S3 Standard IA using a prefix after 45 days
- ❌ **你的选择** Create a Lifecycle Policy to transition objects to Amazon S3 Glacier using a prefix after 180 days
- ✅ **正确答案** Create a Lifecycle Policy to transition all objects to Amazon S3 Glacier after 180 days
- ▫️ Create a Lifecycle Policy to transition objects to Amazon S3 One Zone IA using a prefix after 45 days

### 讲解

**题眼**：45 天后 images 冷了，thumbnails 还热；180 天后**两个文件夹都要**归档；要能扛住 AZ 故障（选二）

**错在哪**：你选了 45 天按前缀转 IA（对）+ 180 天**按前缀**转 Glacier（错）。180 天时 images 和 thumbnails 都要归档，只按 images 前缀过滤就漏掉了 thumbnails。

**正确思路**：45 天：用 images/ 前缀，只把图片转到 Standard-IA；180 天：**所有对象**转到 Glacier。

**其他选项**：
- 45 天全部转 IA：缩略图还在频繁访问，转过去会产生取回费
- One Zone-IA：扛不住 AZ 故障，违反题干的高可用要求

> 💡 **口诀**：每条规则都要问一句：作用于**哪些对象**？

---

## Q63 · Cost-Optimized（平台标注）

### 原题

> An e-commerce analytics company is preparing to archive several years of transaction records and customer analytics reports in Amazon S3 for long-term storage. To meet compliance requirements, the archived data must be encrypted at rest. Additionally, the solution must be cost-effective and ensure that key rotation occurs automatically every 12 months to comply with the company’s internal data governance policy.
>
> Which solution will meet these requirements with the least operational overhead?

- ✅ **正确答案** Use AWS Key Management Service (KMS) to create a customer managed key with automatic rotation enabled. Configure the S3 bucket’s default encryption to use the customer managed key. Migrate the data to the S3 bucket
- ▫️ Encrypt the data locally using client-side encryption libraries and upload the encrypted files to S3. Create a KMS key with imported key material and configure key rotation settings
- ▫️ Use AWS CloudHSM to generate encryption keys. Configure S3 to use these custom encryption keys via client-side encryption and rotate the keys annually using an on-premises key management workflow
- ❌ **你的选择** Use Amazon S3 server-side encryption with S3-managed keys (SSE-S3). Upload data with default encryption enabled. Rely on the built-in key management and rotation behavior of SSE-S3

### 讲解

**题眼**：静态加密；成本低；**每 12 个月自动轮换密钥**；运维最少

**错在哪**：你选了 SSE-S3。SSE-S3 的密钥完全由 AWS 管理，你**无法控制或证明“每 12 个月轮换一次”**，不满足内部治理要求，也没法审计。

**正确思路**：用 KMS 客户托管密钥并开启自动轮换（默认每年一次），再把它设成桶的默认加密，之后就全自动了。

**其他选项**：
- 客户端加密 + 导入密钥材料：导入密钥材料的 KMS 密钥**不支持自动轮换**
- CloudHSM + 本地轮换流程：运维负担最重

> 💡 **口诀**：对密钥有控制、轮换、审计要求 → KMS 客户托管密钥。原标注 Cost，按内容应归到 Secure

---

## Q65 · Secure（平台标注） ⏭️未作答

### 原题

> A developer in your company has set up a classic 2 tier architecture consisting of an Application Load Balancer and an Auto Scaling group (ASG) managing a fleet of Amazon EC2 instances. The Application Load Balancer is deployed in a subnet of size `10.0.1.0/24` and the Auto Scaling group is deployed in a subnet of size `10.0.4.0/22`.
>
> As a solutions architect, you would like to adhere to the security pillar of the well-architected framework. How do you configure the security group of the Amazon EC2 instances to only allow traffic coming from the Application Load Balancer?

- ▫️ Add a rule to authorize the CIDR `10.0.1.0/24`
- ▫️ Add a rule to authorize the CIDR `10.0.4.0/22`
- ▫️ Add a rule to authorize the security group of the Auto Scaling group
- ✅ **正确答案** Add a rule to authorize the security group of the Application Load Balancer

### 讲解

**题眼**：只允许来自 ALB 的流量；安全支柱（最小权限）

**错在哪**：这题你没作答（时间不够）。

**正确思路**：EC2 安全组里加一条入站规则，**来源选 ALB 的安全组**。安全组可以引用另一个安全组，只放行挂了这个安全组的 ALB 节点；ALB 扩缩容、IP 变化都不受影响。

**其他选项**：
- 授权 10.0.1.0/24：会放行这个子网里的**所有资源**，不只是 ALB
- 授权 10.0.4.0/22：这是 EC2 自己的子网，方向反了
- 授权 ASG 的安全组：等于只允许实例之间互访，ALB 的流量还是进不来

> 💡 **口诀**：“只允许来自某个 AWS 资源” → 安全组引用安全组，不用 CIDR

---
