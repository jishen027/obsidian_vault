# Amazon SageMaker - 通用机器学习平台

> **Amazon SageMaker** 是全托管的**通用机器学习平台**，覆盖数据准备、模型构建、训练、调优、部署和监控的完整机器学习生命周期，让数据科学家和开发者能够针对**特有业务数据训练自定义模型**，区别于 [[Amazon Rekognition]]/[[Amazon Transcribe]]/[[Amazon Comprehend]] 等"开箱即用、面向特定任务"的预训练 AI 服务。**2025 年起，经典 SageMaker 更名为 "Amazon SageMaker AI"**，作为新的一体化平台 **"Amazon SageMaker Unified Studio"**（整合 SageMaker AI、[[Amazon EMR]]、[[AWS Glue]]、[[Amazon Athena]]、[[Redshift]]、Amazon Bedrock 于一体）的核心 ML 组件，但底层能力和考试考点保持不变。本笔记同时覆盖 **SAA-C02（架构选型视角）** 与 **AIF-C01（ML 生命周期/MLOps 视角）** 两个考纲的高频考点。
>
> 相关文档：[[Amazon Rekognition]] | [[Amazon Transcribe]] | [[Amazon Comprehend]] | [[Amazon EMR]] | [[AWS Glue]] | [[S3]] | [[EC2]] | [[VPC]] | [[IAM]] | [[KMS]] | [[Amazon Bedrock]] | [[AI与ML概念]] | [[机器学习算法]] | [[数据与分析服务]]

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
| [[Amazon Bedrock]] | 生成式 AI（基础模型）平台 | 调用/微调第三方及 Amazon 基础模型 | 生成式 AI 应用（对话、内容生成），而非传统监督学习任务 |

> **考试陷阱**：题目描述**"需要针对特有业务数据训练自定义机器学习模型，涉及数据准备、算法选择、超参数调优"** → 答案是 **SageMaker**；若题目描述的是标准化任务（图像识别、语音转文本、文本情感分析）且**没有强调自定义训练需求** → 优先考虑对应的预训练 AI 服务（Rekognition/Transcribe/Comprehend），而非"重新发明轮子"用 SageMaker 从零训练。

### 服务选择框架：从简到复杂（AIF-C01 高频考点）

> 在投入成本构建自定义模型之前，应先调查是否有现成服务适用——这是一个"由易到难"的决策顺序，难度和成本依次递增。

```
第一步：AWS 预训练 AI 服务能否满足需求？
├── 能 → 直接使用（Rekognition / Comprehend / Lex 等）
│         支持自定义输出（如 Comprehend 自定义分类器）
│
第二步：现有预训练模型 + 微调？
├── 生成式 AI 场景 → Amazon Bedrock（基础模型 + 迁移学习/Fine-tuning）
├── 其他 ML 场景 → SageMaker JumpStart（预训练模型 + 增量训练）
│
第三步：从头开始训练自定义模型（难度极大，成本极高）
└── 技术上极具挑战性，安全与合规责任极高 → 最后手段
```

### SageMaker vs Amazon Bedrock（AIF-C01 高频考点）

| 特性 | Amazon SageMaker | Amazon Bedrock |
|------|-----------------|----------------|
| **定位** | 构建/训练/部署自定义 ML 模型 | 访问预训练基础模型（LLM） |
| **用户群** | ML 工程师、数据科学家 | AI 工程师、应用开发者 |
| **技术深度** | 深（需要 ML 知识） | 浅（API 调用即可） |
| **模型来源** | 自建、HuggingFace、开源 | AWS 合作商提供的基础模型 |
| **典型场景** | 训练自定义分类器、推荐模型 | 调用 Claude 生成文本、RAG |

```
选择 SageMaker 还是 Bedrock
├── "需要完全控制训练过程" → SageMaker
├── "需要使用自己的数据集从零训练" → SageMaker
├── "快速使用 LLM 构建 AI 应用" → Bedrock
├── "不想管训练基础设施" → Bedrock（Fine-tuning）
└── "需要训练传统 ML 模型（分类、回归）" → SageMaker
```

---

## 机器学习开发生命周期（ML 管道）

> **机器学习管道**是从业务目标到运行已部署模型的一系列相互关联的步骤，是一个**迭代过程**——部分或全部步骤在模型部署后可能会重复进行。

```
1. 确定业务目标
       ↓
2. 收集和准备训练数据
       ↓
3. 训练、优化和评估模型（迭代）
       ↓
4. 部署模型
       ↓
5. 监控模型
       ↓（发现问题时返回步骤 2 或 3）
```

- **确定业务目标**：明确要解决的问题和商业价值，设定可量化的成功标准（没有成功标准就无法评估模型），并进行**成本效益分析**确认 ML 是否是最佳方法——若"构建成本 > 预期收益"，不应启动该 ML 项目
- **SageMaker Studio**：基于 Web 的集成开发环境（IDE），提供 Notebook、代码编辑、实验跟踪等一体化机器学习开发体验（SageMaker Studio Classic 已被新版 Studio 取代）

---

## 数据准备（考试高频）

| 组件 | 说明 |
|------|------|
| **SageMaker Ground Truth** | 托管的数据标注服务，结合**主动学习（Active Learning）**（自动标注高置信度样本）和**人工标注**（处理难例，可结合 Amazon Mechanical Turk 众包）为训练数据打标签 |
| **SageMaker Data Wrangler** | 可视化数据准备工具，300+ 内置转换，无需编写代码 |
| **SageMaker Feature Store** | 集中式特征存储库，支持**在线存储（Online Store，毫秒级延迟，供实时推理读取）**和**离线存储（Offline Store，基于 S3，供批量训练读取）**，训练和推理阶段共享一致的特征定义，避免"训练服务偏差（Training-Serving Skew）" |
| **SageMaker Canvas** | 无代码 ML 工具，上传数据即可训练模型 |
| **[[AWS Glue]] DataBrew** | 可视化数据准备工具，250+ 内置转换，无需代码；转换步骤可保存为**配方（Recipe）**重复用于其他数据集 |

- **训练/验证/测试集划分**：建议比例 **训练集 80% / 验证集 10% / 测试集 10%**
- **特征工程**：精简训练数据中的特征，仅保留推理所需的特征，可组合特征进一步减少数量；减少特征数量 → 减少训练所需的内存量和计算能力
- **[[AWS Glue]]** 作为上游 ETL 服务，将多来源数据收集整理到 S3（详见 [[AWS Glue]] 笔记）；**Glue Data Quality** 使用 DQL 定义数据质量规则并用 ML 检测异常，评估结果可写入 CloudWatch/S3

> **考试陷阱（AIF-C01）**：**Feature Store 的在线/离线存储服务不同目的**——实时推理场景需要毫秒级读取特征，用 Online Store；批量训练需要读取历史大批量特征，用 Offline Store，两者不可混淆。

> **考试提示**：**SageMaker Canvas 空闲时也持续计费**，费用可能远超预期，使用后应及时关闭工作区。

---

## 模型训练

### 训练任务与算法

> 创建训练任务需指定：训练数据 S3 URL、计算资源、模型构件输出 S3 路径、算法（Docker 容器镜像，来自 Amazon ECR）、超参数。

| 组件 | 说明 |
|------|------|
| **内置算法（Built-in Algorithms）** | 预优化的常见 ML 算法（XGBoost、线性回归、KNN 等），无需自行实现 |
| **自定义训练脚本** | 使用自己的 Python 脚本（TensorFlow、PyTorch、MXNet） |
| **自带模型/容器（Bring Your Own Model/Container）** | 完全自定义训练环境，兼容主流框架，存储于 Amazon ECR |
| **分布式训练** | 支持跨多实例的数据并行/模型并行训练，加速大规模模型的训练过程 |

### SageMaker JumpStart 与迁移学习

> **SageMaker JumpStart** 为计算机视觉和 NLP 任务提供经过预训练的 AI 基础模型和特定任务模型（基于大型公用数据集预训练，含 HuggingFace 模型如 Whisper、BERT）。

- **迁移学习（Transfer Learning）**：使用自己的数据集对预训练模型进行**增量训练/微调**，大幅降低成本和开发时间
- 支持快速部署和实验，内置 Jupyter Notebook 示例

### 托管 Spot 训练（Managed Spot Training）

- 使用 [[EC2]] Spot 实例运行训练任务，可大幅降低训练成本（相比按需实例节省最高约 90%），SageMaker 自动处理 Spot 中断后的检查点保存和恢复，无需用户手动处理容错逻辑

> **考试要点**：**训练任务对中断不敏感、可容忍一定延迟时，应优先使用 Managed Spot Training 降低成本**——这是"如何降低机器学习训练成本"场景题的标准答案。

### 实验管理与调优

| 组件 | 说明 |
|------|------|
| **SageMaker Experiments** | 创建、管理、分析和比较大量机器学习实验（可能成千上万个模型版本），可视化比较不同训练运行的关键性能指标 |
| **自动模型调优（Automatic Model Tuning, AMT）** | 配置优化任务，指定算法、超参数范围和目标指标（如最大化 AUC），自动在循环内运行多个训练任务寻找最优超参数组合，替代人工反复试错调参，可结合 Spot 实例降低成本 |
| **SageMaker Autopilot** | **AutoML** 能力，自动完成数据预处理、算法选择、模型训练和调优的全流程，适合缺乏深厚机器学习专业知识的场景快速获得可用模型 |
| **SageMaker Debugger** | 实时分析训练过程，检测梯度问题等异常 |

> **最佳实践**：并行运行多个训练任务（不同算法 + 不同设置），即"运行实验"，结合 Experiments 找到性能最佳的解决方案。

---

## 模型部署与推理（考试高频）

| 推理方式 | 说明 | 适用场景 |
|---------|------|---------|
| **实时推理（Real-Time Inference）** | 部署为持久化的 HTTPS 端点，毫秒级低延迟响应单次请求 | 需要即时响应的在线预测（如实时推荐、欺诈检测） |
| **批量转换（Batch Transform）** | 对存储在 [[S3]] 中的大批量数据做离线批量预测，无需维持常驻端点 | 无实时性要求的大规模离线预测任务 |
| **异步推理（Asynchronous Inference）** | 请求进入队列异步处理，无请求时可缩容至零 | 大文件或长处理时间（可达数分钟）的推理请求，不希望长时间占用同步连接 |
| **无服务器推理（Serverless Inference）** | 自动伸缩计算资源，闲置时可缩容至零，按实际推理量计费 | 流量间歇性、不可预测的推理负载，避免为持久端点付费 |

> **考试陷阱**：**四种推理方式对应不同的延迟和成本权衡**——题目强调"低延迟、持续稳定流量"选实时推理；强调"离线批量处理、无实时性要求"选批量转换；强调"处理时间长、负载大但可接受排队等待"选异步推理；强调"流量间歇、希望空闲时不产生成本"选无服务器推理。四者不可混淆，均是高频考点。

- **所有 SageMaker 托管端点均为完全托管式，支持弹性伸缩**；推理代码和模型构件通常打包为 **Docker 容器**部署，可在 Batch、ECS、EKS、Lambda、EC2 等任何安装容器运行时的 AWS 资源上运行
- **SageMaker 推理推荐器（Inference Recommender）**：测试模型的不同配置选项，帮助选择最佳实例类型和配置
- **推理管道（Inference Pipeline）**：将多个容器（如 sklearn 预处理容器 + 模型推理容器）串联，在**同一 EC2 实例**上按顺序运行，实现低延迟的端到端推理；部署后不可修改，更新需重新部署新版本，管道本身不额外收费

---

## 模型监控与治理

| 组件 | 说明 |
|------|------|
| **SageMaker Model Monitor** | 持续监控生产环境中部署模型的数据质量和预测质量，检测数据/概念漂移，及时发现模型性能衰退 |
| **SageMaker Clarify** | 检测训练数据和模型预测中的**偏差（Bias）**，提供**事后可解释性分析（特征重要性、偏见检测）** |
| **SageMaker Model Registry** | 集中管理模型版本，记录模型的血缘和审批状态，支持模型从训练到生产部署的治理流程 |
| **SageMaker Pipelines** | 面向机器学习工作流的编排服务（MLOps），将数据处理、训练、评估、部署等步骤串联为可重复执行的自动化流水线，支持追踪构件沿袭、条件分支，可用 Python SDK 或 JSON 定义 |

### 数据漂移 vs 概念漂移（AIF-C01 高频考点）

> 模型性能会随时间推移因数据质量、模型质量和模型偏差等原因下降。

| 偏移类型 | 定义 |
|---------|------|
| **数据漂移（Data Drift）** | 与训练数据相比，输入数据分布发生显著变化 |
| **概念漂移（Concept Drift）** | 目标变量的属性发生变化（任何偏移都会导致性能下降） |

**Model Monitor 工作流**：
```
按定义时间表收集端点数据 → 与基准比较 → 根据规则分析 → 在 Studio 中查看结果
     → 发送结果到 CloudWatch → 配置警报 → 触发补救措施（如启动重新训练）
```

---

## MLOps（机器学习运营）

> **MLOps** 是将软件工程最佳实践应用于 ML 模型开发的方法，涉及自动执行手动任务、发布前测试评估代码、自动响应事件。

### 核心原则与好处

- **核心原则**：版本控制（训练数据、代码、模型构件）、自动化（可重复性）、持续监控、自动重新训练（检测到问题时触发）
- **核心好处**：提高工作效率、可重复性、更高可靠性、合规可审计性（对所有输入/输出版本控制，可演示模型构建和部署方式）、提高数据和模型质量

### MLOps 工具生态

| 工具 | 类型 | 用途 |
|------|------|------|
| **AWS CodeCommit** | 源代码存储库 | 存储推理代码 |
| **SageMaker Feature Store** | 特征存储库 | 训练数据特征定义的集中存储 |
| **SageMaker Model Registry** | 模型注册表 | 训练模型和历史记录的集中存储 |
| **SageMaker Pipelines** | ML 管道编排 | 编排 SageMaker 任务，创建可复制的 ML 管道 |
| **AWS Step Functions** | 工作流编排 | 可视化拖放界面定义无服务器工作流，可编排包含 SageMaker 步骤的复杂管道 |
| **Amazon MWAA** | Apache Airflow 托管服务 | 使用 Python 创建工作流，无需管理底层基础设施 |

### 成本追踪

> **AWS 成本分配标签**：为管道使用的所有资源定义标签（如 ML 项目名称），在 **AWS Cost Explorer** 中筛选成本报告，确定项目实际产生的 AWS 费用，用于计算投资回报率（ROI）。

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

## 定价注意事项（考试提示）

> **重要警告**：SageMaker 部分资源在**空闲时也持续计费**，使用完必须及时停止/删除。

| 容易忘关的资源 | 说明 |
|--------------|------|
| **SageMaker Studio 实例** | 内核停止后仍可能计费 |
| **SageMaker Canvas** | 费用可能远超预期，闲置也计费 |
| **实时推理端点** | 按运行时间计费，不用时应删除；间歇流量场景应改用无服务器推理 |

---

## 典型行业案例（AIF-C01 参考）

| 客户 | 使用场景 | 结果 |
|------|---------|------|
| **MasterCard** | 实时欺诈检测评分，SageMaker 训练 + 生成式 AI 增强 | 检测欺诈增加 2 倍，误报减少 10 倍；GenAI 进一步提升效率 20% |
| **Laredo Petroleum** | 监控 1,300+ 口油气井传感器数据，预测性维护 | 实时识别需维护的设备，避免天然气燃烧/泄漏 |
| **Booking.com** | 预订推荐 + AI 行程规划（SageMaker + Bedrock RAG） | 个性化推荐，RAG 保证响应准确且最新 |
| **Pinterest** | 拍照识别商品并推荐相似可售商品（SageMaker Ground Truth + Mechanical Turk） | 图像识别、商品搜索、图片持续重新训练 |

---

## 典型应用场景

| 场景 | 推荐配置 |
|------|---------|
| **针对特有业务数据训练专属预测模型** | SageMaker Studio + 内置算法/自带模型训练 |
| **训练任务对中断不敏感，希望降低成本** | Managed Spot Training |
| **缺乏深厚机器学习专业知识，希望快速获得可用模型** | SageMaker Autopilot（AutoML）或 JumpStart 预训练模型微调 |
| **需要持续、低延迟的在线预测服务** | 实时推理端点 |
| **大批量历史数据的离线预测** | Batch Transform |
| **推理流量间歇、不可预测，希望空闲零成本** | Serverless Inference |
| **生产模型需要持续监控数据/预测质量衰退** | SageMaker Model Monitor |
| **需要检测模型偏差、提升可解释性（合规要求）** | SageMaker Clarify |
| **需要标准化、可重复的端到端 ML 工作流** | SageMaker Pipelines |
| **图像/文本数据需要人工标注** | SageMaker Ground Truth |
| **标准图像/语音/文本任务，无需自定义训练** | 改用 Rekognition/Transcribe/Comprehend，而非从零用 SageMaker 训练 |
| **快速使用 LLM 构建生成式 AI 应用** | 改用 [[Amazon Bedrock]]，而非从零用 SageMaker 训练基础模型 |

---

## 考试重点总结

### SAA-C02 高频考点

1. **核心定位**：通用机器学习平台，覆盖数据准备到模型部署的完整生命周期，用于训练自定义模型
2. **判断依据**：需要自定义训练模型选 SageMaker；标准化任务（图像/语音/文本）优先选对应的预训练 AI 服务
3. **四种推理部署方式**：实时推理（低延迟持续流量）、批量转换（离线批量）、异步推理（长处理时间大负载）、无服务器推理（间歇流量、零闲置成本）
4. **Managed Spot Training 大幅降低训练成本**：自动处理检查点保存和中断恢复，适合可容忍中断的训练任务
5. **SageMaker Autopilot 是 AutoML 能力**：自动完成算法选择和调优，适合缺乏专业知识快速获得可用模型的场景
6. **Ground Truth 用于数据标注**：结合主动学习和人工标注，生成高质量训练数据
7. **Model Monitor 检测数据漂移**：持续监控生产模型的输入数据和预测质量，及时发现性能衰退
8. **Clarify 检测偏差和提供可解释性**：满足模型公平性和合规审计要求
9. **2025 年更名为 SageMaker AI**：作为 SageMaker Unified Studio（整合 EMR/Glue/Athena/Redshift/Bedrock）的核心 ML 组件，底层能力和考点不变
10. **Feature Store 避免训练服务偏差**：训练和推理阶段共享一致的特征定义

### AIF-C01 高频考点

1. **SageMaker vs Bedrock**：自建训练 vs 直接用预训练模型
2. **服务选择框架**：预训练 AI 服务 → JumpStart/Bedrock 微调 → 从头训练（难度和成本依次递增）
3. **迁移学习**：用自己数据对预训练模型进行增量训练，降低成本和时间
4. **Feature Store 在线 vs 离线存储**：在线（毫秒级，供实时推理）vs 离线（S3，供批量训练）
5. **数据漂移 vs 概念漂移**：输入数据分布变化 vs 目标变量属性变化，均导致模型性能下降
6. **MLOps 核心原则**：版本控制、自动化、持续监控、自动重新训练
7. **SageMaker Pipelines vs Step Functions vs MWAA**：均可编排 ML 工作流，前者专为 SageMaker 任务设计，后两者是更通用的工作流编排工具
8. **成本追踪**：成本分配标签 + AWS Cost Explorer 计算 ROI
9. **定价陷阱**：Studio/Canvas/实时端点空闲仍可能计费，需主动清理资源
10. **AWS Glue / DataBrew / Data Wrangler**：分别是通用 ETL 服务、无代码可视化数据准备工具、SageMaker 内置的可视化数据准备工具，三者定位不同但功能有重叠

### 场景题解题思路（是否使用 SageMaker）

```
场景分析 → 判断是否用 SageMaker
├── "需要针对特有业务数据训练自定义机器学习模型" → Amazon SageMaker
├── "标准图像识别/语音转文本/文本情感分析任务，无自定义训练需求" → 改用 Rekognition/Transcribe/Comprehend（而非 SageMaker）
├── "快速使用 LLM 构建生成式 AI 应用" → 改用 Amazon Bedrock（而非从零用 SageMaker 训练）
├── "训练任务可容忍中断，希望大幅降低成本" → Managed Spot Training
├── "缺乏机器学习专业知识，希望快速获得可用模型" → SageMaker Autopilot（AutoML）或 JumpStart
├── "需要持续、低延迟的在线预测服务" → 实时推理端点
├── "大批量历史数据的离线预测，无实时性要求" → Batch Transform
├── "推理请求处理时间长、负载大，可接受排队" → 异步推理（Asynchronous Inference）
├── "推理流量间歇不可预测，希望空闲时零成本" → Serverless Inference
├── "生产模型需要持续监控性能衰退" → SageMaker Model Monitor
├── "需要检测模型偏差、满足合规可解释性要求" → SageMaker Clarify
└── "需要标准化、可重复的端到端 ML 训练部署流程" → SageMaker Pipelines
```

### 场景题解题思路（SageMaker 内部功能选择）

```
场景分析 → 选择 SageMaker 功能
├── "需要给图像/文本打标签" → Ground Truth
├── "需要找最优模型超参数" → Automatic Model Tuning (AMT)
├── "需要管理大量训练实验" → SageMaker Experiments
├── "需要可视化数据清洗（无代码）" → Data Wrangler 或 Glue DataBrew
├── "需要 ETL 从多个来源收集数据" → AWS Glue
├── "需要快速使用预训练模型（HuggingFace 等）" → JumpStart
├── "发现数据漂移/概念漂移" → 触发重新训练模型
└── "需要自动化整个 ML 管道" → SageMaker Pipelines
```

---

## 最佳实践

1. **优先评估预训练 AI 服务/Bedrock 是否满足需求**：标准化任务或生成式 AI 场景不要盲目用 SageMaker 从零训练，遵循"预训练服务 → 微调 → 从零训练"的成本递增顺序
2. **可容忍中断的训练任务使用 Managed Spot Training**：显著降低训练成本，SageMaker 自动处理容错
3. **按推理场景的延迟和流量特征选择合适的部署方式**：不要一律使用实时推理端点，间歇流量场景评估 Serverless Inference 降低闲置成本
4. **生产模型务必配置 Model Monitor**：及时发现数据漂移和性能衰退，避免模型在无感知情况下逐渐失效
5. **合规敏感场景启用 Clarify 做偏差检测**：在模型上线前评估公平性，降低合规风险
6. **使用 Feature Store 统一训练和推理特征**：避免训练服务偏差导致的线上线下预测不一致
7. **复杂 ML 工作流用 Pipelines 编排**：而非依赖手动执行或临时脚本串联各步骤，提升可重复性和可维护性
8. **训练/推理任务遵循最小权限和网络隔离原则**：通过 IAM 执行角色精确授权，敏感场景启用网络隔离防止意外的外部访问
9. **主动管理易被遗忘的计费资源**：Studio 实例、Canvas 工作区、闲置的实时端点应及时停止/删除，避免意外费用
10. **善用成本分配标签追踪 ML 项目 ROI**：结合 AWS Cost Explorer 量化项目实际产生的费用
