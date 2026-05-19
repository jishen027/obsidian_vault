# Amazon SageMaker

> **Amazon SageMaker** 是 AWS 的全托管机器学习平台，覆盖 ML 的完整生命周期：数据准备、模型训练、调参、评估、部署和监控。与 Bedrock 不同，SageMaker 面向需要**自建和训练模型**的 ML 工程师和数据科学家。
>
> 相关文档：[[Amazon Bedrock]] | [[AI与ML概念]] | [[机器学习算法]] | [[数据与分析服务]]

---

## SageMaker vs Bedrock（考试重点）

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

## SageMaker 核心组件

### 开发环境

| 组件 | 描述 |
|------|------|
| **SageMaker Studio** | 集成 IDE，支持 Jupyter Notebooks、代码编辑、实验跟踪 |
| **SageMaker Notebooks** | 托管 Jupyter Notebooks，按使用计费 |
| **SageMaker JumpStart** | 预构建 ML 解决方案和基础模型的起点（含 HuggingFace 模型） |

> **注意**：SageMaker Studio Classic 已被新版 SageMaker Studio 取代，新版体验差异较大。

### 数据准备

| 组件 | 描述 |
|------|------|
| **SageMaker Data Wrangler** | 可视化数据准备工具，数据清洗、转换、可视化 |
| **SageMaker Ground Truth** | 数据标注服务，支持图像标注、文本分类等人工标注工作流 |
| **SageMaker Feature Store** | 集中存储、共享、发现 ML 特征的在线/离线存储 |

> **SageMaker Canvas**：无代码 ML 工具，上传数据即可训练模型，但**费用较高**，即使空闲也持续计费，建议谨慎使用。

### 训练

| 组件 | 描述 |
|------|------|
| **内置算法 (Built-in Algorithms)** | 预优化的常见 ML 算法（XGBoost、线性回归、KNN 等） |
| **自定义训练脚本** | 使用自己的 Python 脚本（TensorFlow、PyTorch、MXNet） |
| **自定义 Docker 容器** | 完全自定义训练环境 |

### 调参与优化

| 组件 | 描述 |
|------|------|
| **Automatic Model Tuning（超参数调优）** | 自动搜索最优超参数组合，可使用 Spot 实例降低成本 |
| **SageMaker Debugger** | 实时分析训练过程，检测梯度问题 |

### 部署与推理

| 组件 | 描述 |
|------|------|
| **实时端点 (Real-time Endpoint)** | 持续运行的推理端点，低延迟响应 |
| **批量转换 (Batch Transform)** | 一次性处理大量数据的批量推理任务 |
| **推理管道 (Inference Pipeline)** | 将多个容器串联成推理流水线（如预处理→推理→后处理） |
| **多模型端点 (Multi-model Endpoint)** | 一个端点托管多个模型，节省资源 |

### 监控

| 组件 | 描述 |
|------|------|
| **SageMaker Model Monitor** | 监控生产模型的数据漂移和质量下降 |
| **CloudWatch 集成** | 追踪训练/推理指标和日志 |

---

## SageMaker ML 流程

### 完整流程

```
数据准备 (Data Wrangler / Ground Truth)
    ↓
特征工程 (Feature Store)
    ↓
模型训练 (Training Jobs)
    ↓
超参数调优 (Automatic Model Tuning)
    ↓
模型评估 (Model Evaluation)
    ↓
模型部署 (Endpoint / Batch Transform)
    ↓
模型监控 (Model Monitor)
```

---

## SageMaker Feature Store

> **Feature Store** 提供在线（低延迟）和离线（大批量）特征存储，支持多个 ML 模型共享特征。

| 模式 | 用途 | 特点 |
|------|------|------|
| **在线存储 (Online Store)** | 实时推理时快速读取特征 | 毫秒级延迟 |
| **离线存储 (Offline Store)** | 批量训练时读取历史特征 | 基于 S3，适合大数据量 |

### 流式摄取

- 使用 **Put Record API** 将实时数据推送到 Feature Store
- 支持 Kafka、Kinesis、Spark 等流式数据源
- 毫秒级延迟，高吞吐量

---

## SageMaker Ground Truth（数据标注）

> **Ground Truth** 提供数据标注工作流，支持图像分类、对象检测、文本分类等任务。

| 特性 | 描述 |
|------|------|
| **人工标注** | 通过标注人员界面完成标注任务 |
| **主动学习** | 自动标注置信度高的样本，人工处理难例 |
| **CORS 注意** | 标注图像时需要启用 S3 CORS（解决图像方向显示问题） |

---

## SageMaker JumpStart

> **JumpStart** 提供预构建的 ML 解决方案、基础模型（含 HuggingFace 模型），可直接部署或微调。

- 访问 HuggingFace 上的 Whisper、Bert 等模型
- 支持快速部署和实验
- 内置 Jupyter Notebook 示例

---

## 定价注意事项

> **重要警告**：SageMaker 部分服务在空闲时也持续计费，使用完必须及时停止实例。

| 容易忘关的资源 | 说明 |
|--------------|------|
| **SageMaker Studio 实例** | 内核停止后仍可能计费 |
| **SageMaker Canvas** | 费用极高（课程中曾有数百加元意外费用），不建议使用 |
| **实时端点** | 按运行时间计费，不用时应删除 |

---

## 推理管道 (Inference Pipeline)

> 将多个容器串联处理推理请求，支持在**同一 EC2 实例**上按顺序运行所有容器（低延迟）。

| 特性 | 描述 |
|------|------|
| **不可变性** | 部署后不可修改，更新需重新部署新版本 |
| **无额外成本** | 推理管道本身不额外收费 |
| **支持容器类型** | sklearn 容器、Spark ML 容器、自定义容器 |

---

## 考试重点总结

### AIF-C01 高频考点

1. **SageMaker vs Bedrock**：自建训练 vs 直接用预训练模型
2. **Data Wrangler**：可视化数据准备
3. **Ground Truth**：数据标注服务
4. **Automatic Model Tuning**：超参数自动调优
5. **Feature Store**：在线（实时推理）vs 离线（批量训练）
6. **部署模式**：实时端点、批量转换、推理管道
7. **Model Monitor**：检测生产模型数据漂移

### 场景题解题思路

```
场景分析 → 选择 SageMaker 功能
├── "需要给图像打标签" → Ground Truth
├── "需要找最优模型参数" → Automatic Model Tuning（超参数调优）
├── "需要低延迟实时推理" → 实时端点
├── "需要处理大批量数据" → Batch Transform（批量转换）
├── "需要检测模型质量下降" → Model Monitor
├── "需要可视化数据清洗" → Data Wrangler
└── "需要快速使用 HuggingFace 模型" → JumpStart
```
