# Amazon SageMaker - 通用机器学习平台

> **Amazon SageMaker** 是全托管的**通用机器学习平台**，覆盖数据准备、模型构建、训练、调优、部署和监控的完整机器学习生命周期，让数据科学家和开发者能够针对**特有业务数据训练自定义模型**，区别于 [[Amazon Rekognition]]/[[Amazon Transcribe]]/[[Amazon Comprehend]] 等"开箱即用、面向特定任务"的预训练 AI 服务。**2025 年起，经典 SageMaker 更名为 "Amazon SageMaker AI"**，作为新的一体化平台 **"Amazon SageMaker Unified Studio"**（整合 SageMaker AI、[[Amazon EMR]]、[[AWS Glue]]、[[Amazon Athena]]、[[Redshift]]、Amazon Bedrock 于一体）的核心 ML 组件，但底层能力和考试考点保持不变。
>
> 相关文档：[[Amazon Rekognition]] | [[Amazon Transcribe]] | [[Amazon Comprehend]] | [[Amazon EMR]] | [[AWS Glue]] | [[S3]] | [[EC2]] | [[VPC]] | [[IAM]] | [[KMS]]

---

## 核心概念

### 为什么需要 SageMaker

- **预训练 AI 服务的局限**：Rekognition/Transcribe/Comprehend 等服务针对通用场景做了优化，但**无法覆盖所有特有业务需求**——当任务不属于"图像识别"、"语音转文本"等标准任务范畴，或需要针对私有数据训练专属模型时，需要一个通用的机器学习平台
- **自建 ML 基础设施的痛点**：从零搭建机器学习训练环境涉及计算资源管理、框架安装配置、分布式训练协调、模型部署运维等大量非算法本身的工作
- **SageMaker 的核心价值**：**托管化机器学习基础设施**，让数据科学家专注于算法和模型本身，AWS 负责底层计算资源的预置、扩缩容和运维

### SageMaker 在 AWS AI/ML 服务栈中的定位（考试要点）

| 服务 | 类型 | 灵活性 | 典型场景 |
|------|------|--------|---------|
| **SageMaker** | 通用机器学习平台 | 完全自定义：数据、算法、模型架构均可自行定义 | 针对特有业务数据训练专属模型，标准 AI 服务无法覆盖的场景 |
| [[Amazon Rekognition]] | 预训练视觉 API | 预训练模型 + 轻量定制（Custom Labels） | 标准图像/视频分析任务 |
| [[Amazon Transcribe]] / [[Amazon Comprehend]] | 预训练语音/文本 API | 预训练模型 + 轻量定制 | 标准语音转文本/文本语义分析任务 |
| Amazon Bedrock | 生成式 AI（基础模型）平台 | 调用/微调第三方及 Amazon 基础模型 | 生成式 AI 应用（对话、内容生成），而非传统监督学习任务 |

> **考试陷阱**：题目描述**"需要针对特有业务数据训练自定义机器学习模型，涉及数据准备、算法选择、超参数调优"** → 答案是 **SageMaker**；若题目描述的是标准化任务（图像识别、语音转文本、文本情感分析）且**没有强调自定义训练需求** → 优先考虑对应的预训练 AI 服务（Rekognition/Transcribe/Comprehend），而非"重新发明轮子"用 SageMaker 从零训练。

---

## 机器学习生命周期核心组件

### 数据准备

| 组件 | 说明 |
|------|------|
| **SageMaker Ground Truth** | 托管的数据标注服务，结合人工标注和机器学习辅助标注，为训练数据打标签，支持众包和私有标注团队 |
| **SageMaker Data Wrangler** | 可视化的数据准备工具，简化数据清洗、转换、特征工程的流程 |
| **SageMaker Feature Store** | 集中式的特征存储库，支持训练和推理阶段共享一致的特征定义，避免"训练服务偏差（Training-Serving Skew）" |

### 模型构建与训练

| 组件 | 说明 |
|------|------|
| **SageMaker Studio** | 基于 Web 的集成开发环境（IDE），提供 Notebook、调试、可视化等一体化机器学习开发体验 |
| **内置算法（Built-in Algorithms）** | 提供针对常见任务（分类、回归、聚类等）优化的预置算法，无需自行实现 |
| **自带模型/容器（Bring Your Own Model/Container）** | 支持使用自定义算法代码或 Docker 容器进行训练，兼容 TensorFlow、PyTorch 等主流框架 |
| **托管训练（Managed Training）** | 提交训练任务后，SageMaker 自动预置训练所需的计算实例，任务结束后自动释放，按实际训练时长计费 |
| **分布式训练** | 支持跨多实例的数据并行/模型并行训练，加速大规模模型的训练过程 |
| **托管 Spot 训练（Managed Spot Training）** | 使用 [[EC2]] Spot 实例运行训练任务，可大幅降低训练成本（相比按需实例节省最高约 90%），SageMaker 自动处理 Spot 中断后的检查点恢复 |

> **考试要点**：**训练任务对中断不敏感、可容忍一定延迟时，应优先使用 Managed Spot Training 降低成本**——SageMaker 自动管理训练过程中的检查点保存和 Spot 中断后的恢复，无需用户手动处理容错逻辑，这是"如何降低机器学习训练成本"场景题的标准答案。

### 超参数调优

- **自动模型调优（Automatic Model Tuning / Hyperparameter Tuning）**：自动尝试不同的超参数组合，基于验证指标寻找最优配置，替代人工反复试错调参
- **SageMaker Autopilot**：**AutoML** 能力，自动完成数据预处理、算法选择、模型训练和调优的全流程，适合缺乏深厚机器学习专业知识的场景快速获得可用模型

### 模型部署与推理（考试高频）

| 推理方式 | 说明 | 适用场景 |
|---------|------|---------|
| **实时推理（Real-Time Inference）** | 部署为持久化的 HTTPS 端点，毫秒级低延迟响应单次请求 | 需要即时响应的在线预测（如实时推荐、欺诈检测） |
| **批量转换（Batch Transform）** | 对存储在 [[S3]] 中的大批量数据做离线批量预测，无需维持常驻端点 | 无实时性要求的大规模离线预测任务 |
| **异步推理（Asynchronous Inference）** | 请求进入队列异步处理，适合负载较大、处理时间较长（可达数分钟）的推理请求 | 大文件或长处理时间的推理请求，同时不希望长时间占用同步连接 |
| **无服务器推理（Serverless Inference）** | 自动伸缩计算资源，闲置时可缩容至零，按实际推理量计费 | 流量间歇性、不可预测的推理负载，避免为持久端点付费 |

> **考试陷阱**：**四种推理方式对应不同的延迟和成本权衡**——题目强调"低延迟、持续稳定流量"选实时推理；强调"离线批量处理、无实时性要求"选批量转换；强调"处理时间长、负载大但可接受排队等待"选异步推理；强调"流量间歇、希望空闲时不产生成本"选无服务器推理。四者不可混淆，均是 SAA-C02 高频考点。

### 模型监控与治理

| 组件 | 说明 |
|------|------|
| **SageMaker Model Monitor** | 持续监控生产环境中部署模型的数据质量和预测质量，检测**数据漂移（Data Drift）**，及时发现模型性能衰退 |
| **SageMaker Clarify** | 检测训练数据和模型预测中的**偏差（Bias）**，并提供模型可解释性（Explainability）分析 |
| **SageMaker Model Registry** | 集中管理模型版本，记录模型的血缘和审批状态，支持模型从训练到生产部署的治理流程 |
| **SageMaker Pipelines** | 面向机器学习工作流的编排服务（MLOps），将数据处理、训练、评估、部署等步骤串联为可重复执行的自动化流水线 |

---

## 网络与安全

| 机制 | 说明 |
|------|------|
| **VPC 部署** | Notebook 实例、训练任务、推理端点均可配置运行在 [[VPC]] 私有子网中，访问私有数据源（如 RDS）或避免暴露公网 |
| **IAM 执行角色** | SageMaker 任务通过 IAM 角色获得访问 S3 训练数据、其他 AWS 服务的权限，遵循最小权限原则 |
| **静态加密** | 训练数据、模型产物存储时可使用 [[KMS]] 密钥加密 |
| **传输加密** | 训练/推理过程中的数据传输默认加密 |
| **网络隔离（Network Isolation）** | 训练/推理容器可配置为完全隔离网络访问，防止代码意外访问外部网络，提升安全性 |

---

## 与其他数据/分析服务的集成

| 集成方式 | 说明 |
|---------|------|
| **[[S3]] 作为主要数据源** | 训练数据、模型产物通常存储在 S3，SageMaker 直接读写 |
| **[[Aurora]] 机器学习集成** | Aurora 支持在 SQL 语句中直接调用 SageMaker 端点做实时预测，无需将数据导出到应用层 |
| **SageMaker Unified Studio** | 新一代整合平台，在同一工作空间中衔接 [[Amazon EMR]]/[[AWS Glue]]/[[Amazon Athena]]/[[Redshift]] 的数据处理能力与 SageMaker AI 的建模能力，打通"数据准备 → 模型训练"的完整链路 |
| **AWS Step Functions** | 可编排包含 SageMaker 训练/推理步骤的复杂工作流，实现跨服务的自动化管道 |

---

## 典型应用场景

| 场景 | 推荐配置 |
|------|---------|
| **针对特有业务数据训练专属预测模型** | SageMaker Studio + 内置算法/自带模型训练 |
| **训练任务对中断不敏感，希望降低成本** | Managed Spot Training |
| **缺乏深厚机器学习专业知识，希望快速获得可用模型** | SageMaker Autopilot（AutoML） |
| **需要持续、低延迟的在线预测服务** | 实时推理端点 |
| **大批量历史数据的离线预测** | Batch Transform |
| **推理流量间歇、不可预测，希望空闲零成本** | Serverless Inference |
| **生产模型需要持续监控数据/预测质量衰退** | SageMaker Model Monitor |
| **需要检测模型偏差、提升可解释性（合规要求）** | SageMaker Clarify |
| **需要标准化、可重复的端到端 ML 工作流** | SageMaker Pipelines |
| **标准图像/语音/文本任务，无需自定义训练** | 改用 Rekognition/Transcribe/Comprehend，而非从零用 SageMaker 训练 |

---

## 考试重点总结

### SAA-C02 高频考点

1. **核心定位**：通用机器学习平台，覆盖数据准备到模型部署的完整生命周期，用于训练自定义模型
2. **判断依据**：需要自定义训练模型选 SageMaker；标准化任务（图像/语音/文本）优先选对应的预训练 AI 服务
3. **四种推理部署方式**：实时推理（低延迟持续流量）、批量转换（离线批量）、异步推理（长处理时间大负载）、无服务器推理（间歇流量、零闲置成本）
4. **Managed Spot Training 大幅降低训练成本**：自动处理检查点保存和中断恢复，适合可容忍中断的训练任务
5. **SageMaker Autopilot 是 AutoML 能力**：自动完成算法选择和调优，适合缺乏专业知识快速获得可用模型的场景
6. **Ground Truth 用于数据标注**：结合人工和机器学习辅助标注，生成高质量训练数据
7. **Model Monitor 检测数据漂移**：持续监控生产模型的输入数据和预测质量，及时发现性能衰退
8. **Clarify 检测偏差和提供可解释性**：满足模型公平性和合规审计要求
9. **2025 年更名为 SageMaker AI**：作为 SageMaker Unified Studio（整合 EMR/Glue/Athena/Redshift/Bedrock）的核心 ML 组件，底层能力和考点不变
10. **Feature Store 避免训练服务偏差**：训练和推理阶段共享一致的特征定义

### 场景题解题思路

```
场景分析 → 判断是否用 SageMaker
├── "需要针对特有业务数据训练自定义机器学习模型" → Amazon SageMaker
├── "标准图像识别/语音转文本/文本情感分析任务，无自定义训练需求" → 改用 Rekognition/Transcribe/Comprehend（而非 SageMaker）
├── "训练任务可容忍中断，希望大幅降低成本" → Managed Spot Training
├── "缺乏机器学习专业知识，希望快速获得可用模型" → SageMaker Autopilot（AutoML）
├── "需要持续、低延迟的在线预测服务" → 实时推理端点
├── "大批量历史数据的离线预测，无实时性要求" → Batch Transform
├── "推理请求处理时间长、负载大，可接受排队" → 异步推理（Asynchronous Inference）
├── "推理流量间歇不可预测，希望空闲时零成本" → Serverless Inference
├── "生产模型需要持续监控性能衰退" → SageMaker Model Monitor
├── "需要检测模型偏差、满足合规可解释性要求" → SageMaker Clarify
└── "需要标准化、可重复的端到端 ML 训练部署流程" → SageMaker Pipelines
```

---

## 最佳实践

1. **优先评估预训练 AI 服务是否满足需求**：标准化任务不要盲目用 SageMaker 从零训练，避免不必要的开发和运维成本
2. **可容忍中断的训练任务使用 Managed Spot Training**：显著降低训练成本，SageMaker 自动处理容错
3. **按推理场景的延迟和流量特征选择合适的部署方式**：不要一律使用实时推理端点，间歇流量场景评估 Serverless Inference 降低闲置成本
4. **生产模型务必配置 Model Monitor**：及时发现数据漂移和性能衰退，避免模型在无感知情况下逐渐失效
5. **合规敏感场景启用 Clarify 做偏差检测**：在模型上线前评估公平性，降低合规风险
6. **使用 Feature Store 统一训练和推理特征**：避免训练服务偏差导致的线上线下预测不一致
7. **复杂 ML 工作流用 Pipelines 编排**：而非依赖手动执行或临时脚本串联各步骤，提升可重复性和可维护性
8. **训练/推理任务遵循最小权限和网络隔离原则**：通过 IAM 执行角色精确授权，敏感场景启用网络隔离防止意外的外部访问
