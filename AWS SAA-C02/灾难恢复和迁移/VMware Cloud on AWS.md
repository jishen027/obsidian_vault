# VMware Cloud on AWS - 在 AWS 上运行 VMware

> **VMware Cloud on AWS** 是由 AWS 和 VMware（现 Broadcom）联合打造的托管服务，把完整的 **VMware 软件定义数据中心（SDDC）技术栈**（vSphere、vSAN、NSX）运行在**专属的 AWS 裸机（Bare Metal）基础设施**上——企业可以用**完全相同的 vCenter 工具、技能和虚拟机格式（VMDK）**，把本地 VMware 工作负载**原样搬到 AWS**，**不需要任何格式转换或重新架构**。这是与 [[AWS Elastic Disaster Recovery]]/Application Migration Service（把工作负载**转换**为原生 EC2 实例）截然不同的迁移路径——前者追求"零改造搬迁"，后者追求"迁移后即享原生 AWS 弹性"。
>
> 相关文档：[[AWS Elastic Disaster Recovery]] | [[AWS Application Discovery Service]] | [[Database Migration Service]] | [[EC2]] | [[VPC]] | [[Disaster Recovery On AWS]]

---

## 核心概念

### 软件定义数据中心（SDDC）

- SDDC 是一组**专属（非多租户）的 AWS 裸机主机**，直接运行 **VMware ESXi 虚拟化层**、**vSAN**（软件定义存储）和 **NSX**（软件定义网络）
- 底层硬件的**采购、部署、打补丁、故障更换**完全由 AWS/VMware 托管，客户只需要像管理本地 VMware 环境一样管理其上的虚拟机
- **考试要点**：**VMware Cloud on AWS 提供的是专属裸机容量，而非共享的多租户虚拟化层**——这天然满足许多**软件许可（BYOL）**和**合规隔离**要求，题目描述"现有 VMware 许可证要求专属物理主机" → VMware Cloud on AWS 的裸机 SDDC 天然满足，无需额外申请 EC2 Dedicated Host。

### 零改造迁移（考试高频）

- 虚拟机以**原生 VMDK 格式**直接运行，**不需要转换成 AMI 或重新打包**，管理员继续使用**熟悉的 vCenter/vSphere 工具链**
- **考试陷阱**：**这是"最小化迁移风险和改造工作量"类题目的标准答案**——题目描述"企业希望尽快把大量 VMware 虚拟机迁移上云，但不希望改变现有的运维工具和虚拟机格式，也不想承担应用重新验证的风险" → **VMware Cloud on AWS**；若题目强调"迁移后需要充分利用原生 AWS 弹性伸缩、Serverless 等能力，愿意做格式转换和架构调整" → 应改用 [[AWS Elastic Disaster Recovery]]/Application Migration Service 迁移为原生 EC2 实例。

### 与原生 AWS 服务的低延迟互通

- SDDC 通过 **ENI** 连接到客户的 [[VPC]]，SDDC 内的虚拟机可以**低延迟、直接访问**原生 AWS 服务（[[S3]]、[[RDS]]、[[AWS Lambda]] 等）
- 这让企业可以**渐进式现代化**：先把 VMware 工作负载原样搬到云上，再按需逐步把部分组件迁移/重构为原生 AWS 服务，而不必"一步到位"完成全部重构

### 混合关联模式（Hybrid Linked Mode）

- 将**本地 vCenter** 和 **云端 SDDC 的 vCenter** 关联成**单一管理视图**，管理员可以在一个界面同时管理本地和云端的虚拟机清单、权限和策略
- **考试要点**：**混合关联模式本身只是"统一管理视图"，真正实现虚拟机跨环境迁移的是 vMotion**——不要把两者混为一谈。

### 实时 vMotion 迁移（零停机，考试高频）

- 借助 **vMotion**，可以在**本地 VMware 环境**和**云端 SDDC** 之间**实时、零停机**地迁移运行中的虚拟机，业务连接不中断
- **考试陷阱**：**这是 VMware Cloud on AWS 相较 AWS 原生迁移工具最大的差异化优势**——[[AWS Elastic Disaster Recovery]] 等工具依赖持续复制 + 择机**割接（Cutover）**，割接瞬间仍有短暂切换；而 vMotion 迁移在**虚拟化层面**完成，实现真正的**零停机热迁移**。题目描述"数据库/应用在迁移过程中一刻都不能中断，且希望保留原有虚拟化环境" → vMotion（VMware Cloud on AWS），而非依赖 DRS 的持续复制加割接模式。

---

## 典型用途

| 用途 | 说明 |
|------|------|
| **数据中心退出（Data Center Evacuation）** | 本地数据中心租约到期或希望停止自建机房运维，需要在有限时间窗口内整体迁移大量 VMware 工作负载 |
| **灾难恢复目标** | 结合 **VMware Site Recovery**，把 VMware Cloud on AWS 的 SDDC 作为本地 VMware 环境的灾难恢复站点，与 [[AWS Elastic Disaster Recovery]]（面向原生 EC2 恢复）是两条并行但不同的 DR 路径 |
| **云容量扩展（Cloud Bursting）** | 本地 VMware 容量不足时，临时扩展到云端 SDDC 承接峰值负载 |
| **渐进式云现代化** | 先原样搬迁，再逐步将部分组件重构为原生 AWS 服务，降低一次性重构的风险 |

> **考试陷阱**：**VMware Cloud on AWS 也可以作为灾难恢复方案，但它和 [[AWS Elastic Disaster Recovery]] 面向的恢复目标不同**——前者的恢复目标仍是**运行在 VMware SDDC 中的虚拟机**（保留原有虚拟化环境）；后者的恢复目标是**原生 EC2 实例**。题目描述"希望灾难恢复站点继续使用 VMware 工具管理，而非转换为原生 EC2" → VMware Cloud on AWS + VMware Site Recovery；题目描述"希望灾难恢复后直接是可弹性伸缩的原生 EC2 环境" → [[AWS Elastic Disaster Recovery]]。

---

## 与迁移规划工具的关系

- 大规模 VMware 环境迁移前，仍建议先用 **[[AWS Application Discovery Service]]** 的无代理发现（本身就基于 VMware vCenter 环境采集）摸清资产清单和依赖关系，再决定是采用 **VMware Cloud on AWS（零改造）** 还是 **[[AWS Elastic Disaster Recovery]]/Application Migration Service（转换为原生 EC2）**
- 数据库层面若计划在迁移后**切换到不同的数据库引擎**（如从自建 Oracle 迁移到 Aurora），仍需要 **[[Database Migration Service]]** 单独处理数据/Schema 迁移，VMware Cloud on AWS 本身不涉及数据库内部的迁移逻辑

---

## 计费模式（考试提示）

- 按**专属裸机主机**计费，而非按虚拟机数量计费——即使 SDDC 内运行数十上百台虚拟机，计费单位始终是底层的裸机主机
- 支持**按需付费**或以 **1 年/3 年订阅**换取更低的小时单价，与 EC2 的按需/预留实例定价理念类似

---

## 典型应用场景

| 场景 | 推荐方案 |
|------|---------|
| **大量 VMware 虚拟机需要快速上云，且不希望改造格式或工具链** | VMware Cloud on AWS |
| **迁移过程要求零停机，业务连接不能中断** | VMware Cloud on AWS + vMotion |
| **希望迁移后直接获得原生 AWS 弹性伸缩/Serverless 能力，可接受格式转换** | 改用 [[AWS Elastic Disaster Recovery]]/Application Migration Service |
| **软件许可证要求专属物理主机** | VMware Cloud on AWS 的专属裸机 SDDC |
| **灾难恢复站点需要继续使用 VMware 工具管理** | VMware Cloud on AWS + VMware Site Recovery |
| **本地容量不足，临时扩展承接峰值负载** | VMware Cloud on AWS 云容量扩展 |
| **希望统一管理本地和云端 VMware 虚拟机清单** | Hybrid Linked Mode |
| **大规模迁移前需要先摸清资产和依赖关系** | 先用 [[AWS Application Discovery Service]] 做发现规划 |

---

## 考试重点总结

### SAA-C02 高频考点

1. **VMware Cloud on AWS 运行在专属裸机主机上**：非多租户虚拟化层，天然满足软件许可和合规隔离需求
2. **零改造迁移是核心卖点**：虚拟机保持原生 VMDK 格式，继续使用熟悉的 vCenter 工具链
3. **通过 ENI 连接 VPC，可低延迟访问原生 AWS 服务**：支持渐进式云现代化，而非"一步到位"重构
4. **Hybrid Linked Mode 只是统一管理视图**：真正实现零停机跨环境迁移的是 vMotion
5. **vMotion 支持真正的零停机热迁移**：与 AWS 原生迁移工具"持续复制 + 择机割接"的模式有本质区别
6. **可作为灾难恢复目标，但恢复目标仍是 VMware 虚拟机**：与 AWS Elastic Disaster Recovery 恢复为原生 EC2 实例是两条不同路径
7. **按专属裸机主机计费，而非按虚拟机数量**：即使 SDDC 内运行大量虚拟机，计费单位不变
8. **迁移前仍建议先用 AWS Application Discovery Service 做发现**：VMware Cloud on AWS 只是执行阶段的一种选择，不替代规划阶段
9. **数据库引擎切换类迁移仍需 Database Migration Service**：VMware Cloud on AWS 不处理数据库内部的数据/Schema 转换
10. **选型核心依据是"是否接受格式转换和架构调整"**：不接受/追求最小改造风险 → VMware Cloud on AWS；接受转换以换取原生 AWS 能力 → Elastic Disaster Recovery/Application Migration Service

### 场景题解题思路

```
场景分析 → 判断 VMware Cloud on AWS 相关配置
├── "大量 VMware 虚拟机需快速上云，不希望改造格式/工具链" → VMware Cloud on AWS
├── "迁移过程要求零停机，业务连接不能中断" → VMware Cloud on AWS + vMotion
├── "希望迁移后直接获得原生 AWS 弹性能力，可接受格式转换" → 改用 Elastic Disaster Recovery/Application Migration Service
├── "软件许可证要求专属物理主机" → VMware Cloud on AWS 专属裸机 SDDC
├── "灾难恢复站点需继续用 VMware 工具管理" → VMware Cloud on AWS + VMware Site Recovery
├── "本地容量不足需临时扩展" → VMware Cloud on AWS 云容量扩展
├── "需要统一管理本地和云端虚拟机清单" → Hybrid Linked Mode
└── "大规模迁移前需先摸清资产和依赖关系" → AWS Application Discovery Service（规划阶段）
```

---

## 最佳实践

1. **大规模迁移前先用 AWS Application Discovery Service 摸清资产和依赖关系**：再决定采用零改造迁移还是转换为原生 EC2
2. **明确业务是否真的需要零停机迁移**：若可接受短暂割接窗口，评估 Elastic Disaster Recovery 是否已经足够，避免为不必要的零停机需求引入更高成本
3. **迁移落地后规划渐进式现代化路线图**：利用 ENI 连接原生 AWS 服务的能力，逐步而非一次性完成组件重构
4. **软件许可证有专属主机要求时优先评估 VMware Cloud on AWS，而非额外申请 EC2 Dedicated Host**：裸机 SDDC 天然满足此类合规需求
5. **灾难恢复方案选型时明确恢复目标环境**：需要保留 VMware 管理体验选 VMware Cloud on AWS + Site Recovery，需要原生 EC2 弹性选 Elastic Disaster Recovery
6. **利用 1 年/3 年订阅降低长期使用成本**：稳定的长期工作负载不应只使用按需计费
7. **数据库层面的迁移需求单独规划 Database Migration Service**：不要假设 VMware Cloud on AWS 会连带处理数据库内部的迁移逻辑