# Amazon Textract - 智能文档处理服务

> **Amazon Textract** 是全托管的**智能文档处理（IDP）**服务，超越传统 OCR——不仅提取文档中的文字，还能理解文档的**结构**（表单字段、表格、签名、身份证件字段等），从扫描件、PDF、图像中自动提取结构化数据，无需自定义模板或规则即可处理各类业务文档。
>
> 相关文档：[[Amazon Comprehend]] | [[Amazon Comprehend Medical]] | [[Amazon Rekognition]] | [[S3]] | [[AWS Lambda]] | [[IAM]] | [[KMS]]

---

## 核心概念

### 为什么需要 Textract

- **传统 OCR 的局限**：普通 OCR 只能识别图像中的字符，输出一堆无结构的纯文本，仍需人工或自定义规则去解析"这段文字是发票号"还是"这段文字是收件人姓名"
- **Textract 的核心价值**：使用机器学习**理解文档的版面结构**，直接识别表单的键值对（Key-Value）、表格的行列关系、身份证件的标准字段，输出**保留结构和坐标信息**的结果，大幅减少后续人工解析的工作量
- **在 AWS AI/ML 服务栈中的定位**：与 [[Amazon Rekognition]] 同属计算机视觉范畴，但 Textract **专注于文档场景**（表单/表格/证件），而 Rekognition 面向通用图像/视频分析（物体、人脸、场景）

### Textract 在 SAA-C02/AIF-C01 考试中的定位（考试要点）

| 服务 | 核心能力 | 输出形式 | 典型场景 |
|------|---------|---------|---------|
| **Textract** | 文档结构化提取（OCR + 版面理解） | 结构化文本、表单键值对、表格、证件字段 | 数字化纸质表单、发票/收据自动录入、身份证件核验 |
| [[Amazon Rekognition]] | 通用图像/视频分析 | 物体标签、人脸属性、场景信息 | 图像识别、内容审核、人脸比对，非文档结构化场景 |
| [[Amazon Comprehend]] | 文本语义分析 | 情感、实体、关键短语 | 对已提取的纯文本做语义理解，通常是 Textract 的**下游** |

> **考试陷阱**：题目描述**"需要从扫描文档/图像中提取文字并识别表单结构（如键值对、表格）"** → 答案是 **Textract**，而非 Rekognition——Rekognition 的文字检测（Text Detection）只能识别图像中的文字内容（如路牌文字），**不理解文档的表单/表格结构**；题目若强调"发票/收据/身份证件的结构化字段提取"，进一步应考虑 Textract 的专用 API（AnalyzeExpense/AnalyzeID），而非通用的 AnalyzeDocument。

---

## 核心 API（考试高频）

| API | 说明 |
|------|------|
| **DetectDocumentText** | 基础 OCR，仅提取文档中的原始文字内容和位置坐标，不理解结构 |
| **AnalyzeDocument** | 在 OCR 基础上识别**表单键值对（Forms）**和**表格（Tables）**结构，支持自定义 **Queries**（用自然语言提问获取特定字段，如"客户姓名是什么？"） |
| **AnalyzeExpense** | 专为**发票/收据**优化，理解上下文自动提取供应商名称、发票编号等 40+ 规范化字段（含摘要字段和明细行字段） |
| **AnalyzeID** | 专为**身份证件**（护照、驾驶执照等）优化，无需模板配置即可自动提取到期日期、出生日期等标准字段 |
| **AnalyzeLending** | 专为贷款文件处理场景优化的文档分类与提取能力 |

> **考试要点**：**根据文档类型选择专用 API 而非通用 API**——处理发票/收据用 AnalyzeExpense，处理身份证件用 AnalyzeID，两者针对特定文档类型做了字段级优化，比用通用的 AnalyzeDocument 手动解析更准确、更省开发工作量。

### Queries 功能

- **AnalyzeDocument** 支持 **Queries（查询）**功能：以自然语言提问的方式指定需要提取的信息（如"总金额是多少？"），无需预先知道文档的固定版面结构
- 支持基于自有数据**自定义调优 Queries**，提升在特定业务文档类型上的提取准确率，同时保持对数据的控制权

---

## 支持格式与输出

- **支持文件格式**：JPEG、PNG、PDF、TIFF
- **输出内容**：提取的文字、表单键值对、表格结构，均附带**位置坐标（Bounding Box）**，便于在原始文档中定位每个提取结果
- **签名检测**：可检测文档中签名的存在和位置，用于合规审核场景（如验证合同是否已签署），但不做签名笔迹身份核验

---

## 常见组合工作流

```
扫描文档/图像 → Amazon Textract（提取文字/结构） → Amazon Comprehend（情感分析/PII 检测）→ 结果
```

- **典型应用**：从合同、报告等文档提取文字后，进一步用 [[Amazon Comprehend]] 做情感分析、实体提取或 PII 检测
- **医疗场景变体**：提取医疗文档文字后，改用 **[[Amazon Comprehend Medical]]** 做医学实体和 PHI 识别，而非通用 Comprehend

---

## 安全性

| 机制 | 说明 |
|------|------|
| **IAM 策略** | 控制哪些身份可以调用 Textract API |
| **静态加密** | 处理结果和存储的文档可使用 [[KMS]] 密钥加密 |
| **传输加密** | API 调用默认通过 HTTPS/TLS |
| **VPC Endpoint** | 支持通过接口终端节点在 VPC 内私有调用 API |

---

## 架构与集成

| 集成方式 | 说明 |
|---------|------|
| **[[S3]] 触发异步处理** | 文档上传到 S3 后，通过事件通知触发 [[AWS Lambda]] 调用 Textract 异步分析大批量/多页文档 |
| **同步 vs 异步 API** | 单页、小文件适合**同步调用**立即返回结果；多页 PDF 等大文件应使用**异步 API**，提交任务后通过 SNS 通知获取结果 |
| **下游语义分析** | 提取结果传递给 [[Amazon Comprehend]]/[[Amazon Comprehend Medical]] 做进一步的情感分析、实体提取或医学信息识别 |

---

## 典型应用场景

| 场景 | 推荐配置 |
|------|---------|
| **数字化纸质申请表/合同** | AnalyzeDocument（Forms + Tables） |
| **自动录入发票/收据数据** | AnalyzeExpense |
| **身份证件信息核验（KYC）** | AnalyzeID |
| **验证合同是否包含签名** | AnalyzeDocument 签名检测 |
| **贷款文件的分类与字段提取** | AnalyzeLending |
| **在大量文档中按自然语言问题查找特定信息** | AnalyzeDocument Queries |
| **提取文字后进一步做情感/实体分析** | Textract + Comprehend 组合管道 |
| **提取医疗文档文字后识别医学实体** | Textract + Comprehend Medical 组合管道 |
| **只需识别图像中的文字（如路牌），无需理解文档结构** | 改用 [[Amazon Rekognition]] 文字检测，而非 Textract |

---

## 考试重点总结

### 高频考点

1. **核心定位**：智能文档处理服务，超越传统 OCR，理解表单/表格/证件的结构化信息
2. **判断依据**：需要"理解文档结构"（键值对、表格、证件字段）时选 Textract；只需识别图像中零散文字（路牌等）选 Rekognition
3. **五种核心 API**：DetectDocumentText（基础 OCR）、AnalyzeDocument（表单/表格 + Queries）、AnalyzeExpense（发票/收据）、AnalyzeID（身份证件）、AnalyzeLending（贷款文件）
4. **专用 API 优先于通用 API**：处理发票用 AnalyzeExpense，处理证件用 AnalyzeID，比手动解析 AnalyzeDocument 结果更准确高效
5. **Queries 支持自然语言提问**：无需预知文档固定版面即可提取指定字段，且支持自定义调优
6. **同步 vs 异步**：单页小文件同步调用；多页大文件异步处理 + SNS 通知
7. **输出附带坐标信息**：便于在原始文档中定位每个提取结果，支持高亮展示等场景
8. **Textract + Comprehend 标准组合**：文档结构化提取 + 语义分析，构成"文档 → 文字 → 洞察"的完整管道
9. **医疗文档场景改用 Comprehend Medical**：而非通用 Comprehend，专为医学术语和 PHI 优化
10. **签名检测只识别存在与位置**：不做签名笔迹的身份验证，不可与身份核验混淆

### 场景题解题思路

```
场景分析 → 判断是否用 Textract
├── "需要数字化纸质表单/合同的结构化字段" → Amazon Textract（AnalyzeDocument）
├── "需要自动录入发票/收据数据" → AnalyzeExpense
├── "需要提取护照/驾驶执照等身份证件信息" → AnalyzeID
├── "需要验证文档是否包含签名" → AnalyzeDocument 签名检测
├── "需要按自然语言问题从文档中查找特定信息" → AnalyzeDocument Queries
├── "文档为多页 PDF 或大文件" → 使用异步 API + SNS 通知
├── "提取文字后需要做情感分析/PII 检测" → Textract + Comprehend 组合
├── "提取的是医疗文档，需要识别医学实体" → Textract + Comprehend Medical 组合
└── "只需识别图像中零散文字，无需理解文档结构" → 改用 Amazon Rekognition（而非 Textract）
```

---

## 最佳实践

1. **按文档类型选择专用 API**：发票用 AnalyzeExpense、证件用 AnalyzeID，而非统一用通用 API 再自行解析
2. **大文件/多页文档使用异步处理**：避免同步调用超时，结合 SNS 通知获取处理结果
3. **善用 Queries 减少版面适配工作**：面对版面多变的文档类型，用自然语言提问比硬编码字段位置更灵活
4. **业务专属文档类型评估自定义调优 Queries**：在保持数据控制权的前提下提升特定文档的提取准确率
5. **结合 S3 事件通知构建自动化处理管道**：文档上传后自动触发提取，避免人工介入
6. **下游语义分析选对专用服务**：通用文档接 Comprehend，医疗文档接 Comprehend Medical
7. **利用坐标信息增强用户体验**：在前端展示提取结果时高亮原文档对应位置，提升可信度和可核验性
8. **签名检测场景明确其能力边界**：仅用于确认签名"存在"，不能替代身份核验流程
