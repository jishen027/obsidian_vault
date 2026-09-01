# Amazon EKS - Elastic Kubernetes Service

> **Amazon EKS (Elastic Kubernetes Service)** 是 AWS 上的**全托管 Kubernetes 服务**，负责管理 Kubernetes 控制平面（Control Plane）的高可用性、打补丁和升级，让用户专注于运行标准的 K8s 工作负载，同时可与 [[ECS]]、[[AWS Fargate]] 一样选择底层计算方式。
>
> 相关文档：[[ECS]] | [[AWS Fargate]] | [[EC2]] | [[Auto Scaling]] | [[AWS Load Balance]] | [[IAM]] | [[虚拟私有云 - VPC]]

---

## 核心概念

### EKS 定位（考试要点）

| 服务 | 编排器 | 学习曲线 | 兼容性 | 典型场景 |
|------|--------|---------|--------|---------|
| **EKS** | 标准 **Kubernetes** | 高（需要 K8s 知识） | 与开源 K8s 生态完全兼容，可无缝迁移已有 K8s 工作负载 | 已在用 K8s、需要跨云可移植性、依赖 K8s 生态工具（Helm、Operator 等） |
| [[ECS]] | AWS 专有编排器 | 低 | 仅限 AWS，非标准 K8s | 新项目、追求简单快速上手、不需要 K8s 生态 |

> **考试陷阱**：题目描述"团队已经在其他环境（本地/其他云）使用 Kubernetes，希望迁移到 AWS 并保持一致的编排工具和运维方式"→ **EKS**；题目只强调"想要简单快速地在 AWS 上运行容器，团队不熟悉 K8s"→ **ECS**（学习曲线更低）。

### 核心组件

| 组件 | 描述 |
|------|------|
| **控制平面（Control Plane）** | AWS 托管的 Kubernetes API Server、etcd 等组件，跨多可用区自动高可用，用户不可直接访问底层节点 |
| **数据平面（Data Plane）** | 实际运行 Pod 的计算资源，可以是 EC2 节点组、Fargate，或自建节点 |
| **节点组（Node Group）** | 一组用于运行 Pod 的 EC2 实例，由 EKS 托管（Managed Node Group）或用户自管理（Self-Managed） |
| **Pod** | Kubernetes 最小调度单元，包含一个或多个容器 |
| **命名空间（Namespace）** | 逻辑隔离单元，用于在同一集群内划分不同团队/环境的资源 |

---

## 计算方式：托管节点组 vs 自管理节点 vs Fargate

| 方式 | 管理粒度 | 说明 |
|------|---------|------|
| **托管节点组（Managed Node Group）** | AWS 半托管 | AWS 自动处理节点的创建、更新、终止生命周期，底层仍是用户账户内的 EC2 实例 |
| **自管理节点（Self-Managed Nodes）** | 用户完全管理 | 用户自行创建和维护 EC2 [[Auto Scaling]] 组作为工作节点，灵活性最高但运维负担最重 |
| **[[AWS Fargate]]** | 无服务器 | 无需管理任何节点，按 Pod 声明的 CPU/内存计费，AWS 自动分配计算资源 |

> **考试要点**：EKS 与 [[ECS]] 一样，都可以选择 **Fargate 作为无服务器计算层**——"无需管理服务器的容器化工作负载"这一考点在 ECS 和 EKS 场景下都可能出现，需结合题目是否强调"标准 K8s API/工具链"来判断选 ECS 还是 EKS。

---

## 网络与负载均衡

- **VPC CNI 插件**：EKS 默认使用 Amazon VPC CNI，每个 Pod 获得 [[虚拟私有云 - VPC]] 内的真实私有 IP，而非叠加网络（Overlay），性能更接近原生 EC2 网络
- **AWS Load Balancer Controller**：在集群内运行的控制器，可根据 Kubernetes Ingress/Service 资源自动创建和配置 [[AWS Load Balance]]（ALB/NLB），实现 K8s 原生的负载均衡声明方式
- **安全组**：可结合 `awsvpc`-类似的机制，为 Pod 级别关联安全组（Security Groups for Pods），实现细粒度网络隔离

---

## 安全性

| 机制 | 说明 |
|------|------|
| **IAM Roles for Service Accounts (IRSA)** | 将 [[IAM]] 角色绑定到 Kubernetes Service Account，实现 **Pod 级别的最小权限**，无需在节点级别共享宽泛的 IAM 权限 |
| **集群访问控制（aws-auth ConfigMap / EKS Access Entries）** | 将 IAM 用户/角色映射到 Kubernetes RBAC 权限，控制谁能通过 `kubectl` 访问集群资源 |
| **控制平面日志** | 可选启用并发送到 CloudWatch Logs（API Server、审计日志、认证器日志等） |
| **静态加密** | 支持通过 [[KMS]] 对 Kubernetes Secrets 做信封加密（Envelope Encryption） |

> **考试陷阱**：**IRSA 是 EKS 场景下实现"Pod 级别最小权限"的标准答案**，类比于 ECS 中的"任务级 IAM 角色"——两者思路一致，都是避免整个节点/实例共享同一套宽泛权限。

---

## EKS vs ECS vs Fargate（考试高频）

| 特性 | EKS | ECS | Fargate |
|------|-----|-----|---------|
| **本质** | 托管 Kubernetes 编排器 | AWS 专有容器编排器 | ECS/EKS 之下的无服务器计算层 |
| **学习曲线** | 高 | 低 | 不适用（计算层而非编排器） |
| **跨云可移植性** | 高（标准 K8s） | 无（AWS 专有） | 不适用 |
| **控制平面费用** | 每个集群按小时收费 | 免费（只付底层资源费用） | 按声明的 CPU/内存计费 |
| **适用场景** | 已有 K8s 经验、需要跨云一致性 | 新项目、追求简单快速 | 两者都可选用，追求零运维 |

---

## 典型应用场景

| 场景 | 推荐配置 |
|------|---------|
| **已有 Kubernetes 工作负载迁移上云** | EKS（保留原有 Helm Chart、Operator 等生态工具） |
| **多云/混合云需要一致的编排工具** | EKS |
| **团队不熟悉 K8s，追求简单上手** | 改用 [[ECS]] |
| **需要 Pod 级别无服务器计算** | EKS + Fargate |
| **需要 Pod 级别最小权限的 IAM 控制** | EKS + IRSA |
| **依赖复杂的 K8s 生态（Service Mesh、Operator）** | EKS |

---

## 考试重点总结

### SAA-C02 高频考点

1. **EKS 定位**：全托管 Kubernetes 控制平面，兼容标准 K8s 生态
2. **EKS vs ECS 选型**：已有 K8s 经验/需要跨云可移植性选 EKS；追求简单快速选 ECS
3. **三种计算方式**：托管节点组、自管理节点、Fargate（无服务器）
4. **控制平面收费**：EKS 集群控制平面按小时计费，区别于 ECS 服务本身免费
5. **VPC CNI**：Pod 直接获得 VPC 内真实私有 IP，而非 Overlay 网络
6. **IRSA 实现 Pod 级最小权限**：将 IAM 角色绑定到 Kubernetes Service Account，类比 ECS 任务级 IAM 角色
7. **AWS Load Balancer Controller**：根据 K8s Ingress/Service 自动创建 ALB/NLB
8. **EKS 同样支持 Fargate**：无服务器容器计算不是 ECS 独有能力
9. **控制平面高可用**：AWS 自动跨多可用区管理 API Server、etcd，用户不可直接访问
10. **Secrets 加密**：可通过 KMS 对 Kubernetes Secrets 做信封加密

### 场景题解题思路

```
场景分析 → 选择容器编排服务
├── "已有 Kubernetes 工作负载，需要保持一致的编排/工具链" → EKS
├── "多云或混合云环境需要标准化的容器编排" → EKS
├── "团队不熟悉 K8s，追求简单快速部署容器" → ECS（而非 EKS）
├── "需要 Pod/任务级别的无服务器计算" → EKS/ECS + Fargate（均可）
├── "需要 Pod 级别的最小 IAM 权限" → EKS + IRSA
├── "需要根据 K8s Ingress 资源自动创建负载均衡器" → EKS + AWS Load Balancer Controller
└── "需要跨云迁移容器工作负载" → EKS（标准 K8s 兼容性优势）
```

---

## 最佳实践

1. **已有 K8s 经验或生态依赖时优先 EKS**：避免重写 Helm Chart、Operator 等已有资产
2. **无特殊需求的新项目优先评估 ECS**：学习曲线更低，运维更简单
3. **优先使用托管节点组而非自管理节点**：减少节点生命周期管理的运维负担
4. **Pod 级权限使用 IRSA**：避免节点级别共享过宽的 IAM 权限
5. **善用 AWS Load Balancer Controller**：用 K8s 原生声明方式管理负载均衡，而非手动创建 ALB/NLB
6. **敏感数据的 Secrets 启用 KMS 加密**：不要依赖默认的 Base64 编码作为安全手段
7. **可容忍中断的工作负载评估 Fargate Spot**：与 ECS 一样可用于 EKS 场景下的成本优化
