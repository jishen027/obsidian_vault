# AWS Amplify - 全栈 Web/移动应用开发平台

> **AWS Amplify** 是面向**前端/移动开发者**的全栈开发平台，包含两大能力：**Amplify Hosting**（基于 Git 的全托管**静态网站/SSR 应用**持续部署与托管，底层由 [[S3]] + [[CloudFront]] 承载）和 **Amplify Libraries/CLI/Studio**（一键式配置和调用 [[Cognito]]（认证）、AppSync/[[Amazon API Gateway]]（API）、[[S3]]（存储）、[[DynamoDB]]（数据）等后端服务，无需逐个手动配置或编写基础设施代码）。它的目标是让前端开发者**几分钟内**从"一份代码"到"一个具备完整认证、API、存储能力的可上线应用"，与面向**传统服务器端应用**的 [[Elastic Beanstalk]] 定位完全不同。
>
> 相关文档：[[S3]] | [[CloudFront]] | [[Cognito]] | [[Amazon API Gateway]] | [[DynamoDB]] | [[AWS Lambda]] | [[Elastic Beanstalk]] | [[AWS CloudFormation]] | [[Route 53 DNS]]

---

## 核心概念

### Amplify Hosting：基于 Git 的全托管前端托管

| 特性 | 说明                                                                          |
|------|------|
| **Git 驱动的 CI/CD** | 连接 GitHub/GitLab/Bitbucket/CodeCommit 仓库，**每次推送代码自动触发构建和部署**，无需手动上传文件或搭建流水线 |
| **分支即环境（Branch-based Deployments）** | 每个 Git 分支可映射为一套**独立部署的环境**（如 `main` → 生产、`dev` → 测试），天然支持功能分支预览             |
| **静态站点 + SSR 双支持** | 既支持纯静态站点（SSG），也支持 **Next.js** 等框架的**服务端渲染（SSR）**                            |
| **原子部署与即时回滚** | 新版本部署是**全有或全无**的原子操作，出现问题可立即回滚到上一个已知正常的版本                                   |
| **自定义域名 + 免费 SSL** | 一键绑定自定义域名，证书通过 **[[AWS Certificate Manager]]** 自动签发管理                       |

> **考试要点**：**Amplify Hosting 本质是把 [[S3]] 静态托管 + [[CloudFront]] 分发这套经典组合，包装成带 Git CI/CD、分支环境和 SSR 支持的全托管服务**——题目描述"需要为前端应用搭建简单的、每次提交代码自动部署的托管环境，并支持多个功能分支的独立预览" → **Amplify Hosting**，而不必手动搭建 S3 + CloudFront + 自建 CI/CD 流水线。

### Amplify Libraries/CLI/Studio：简化后端接入

- **Amplify CLI/Studio**：通过命令行或可视化界面**声明式配置**所需的后端能力（认证、API、存储、数据），Amplify 在背后自动生成并部署对应的 **[[AWS CloudFormation]]** 堆栈，开发者无需逐个手动配置 [[Cognito]]/AppSync/[[DynamoDB]] 等服务
- **Amplify Libraries**：面向 Web（JS）、iOS、Android、Flutter 的开源客户端库，提供**按能力分类的简单 API**（Auth、API、Storage、DataStore、Analytics），屏蔽了直接调用 Cognito SDK、签名 S3 请求等底层细节

### Amplify DataStore（离线优先数据同步）

- 提供**设备本地数据存储**，应用可在**离线状态下正常读写**，网络恢复后自动与云端（AppSync + DynamoDB）**同步**，并内置**冲突解决**机制
- 适合需要良好离线体验的移动应用场景

---

## AWS Amplify vs AWS Elastic Beanstalk（考试高频对比）

| 维度 | AWS Amplify | [[Elastic Beanstalk]] |
|------|-------------|--------------------------|
| **面向对象** | 前端/移动应用（静态站点、SSR、移动 App） | 传统服务器端应用（Java/.NET/PHP/Node.js 等） |
| **底层基础设施** | 无服务器（S3 + CloudFront + 可选 Lambda/AppSync） | **持续运行**的 EC2/ELB/Auto Scaling，本质仍是常驻 IaaS 资源 |
| **部署触发方式** | Git 推送自动触发 | 上传应用版本包（也可结合 CI/CD 工具触发） |
| **后端能力接入** | 内置 Auth/API/Storage 等能力的一键配置 | 需要自行挂载/配置 RDS 等后端资源 |
| **典型场景** | JAMstack 网站、SPA、移动应用后端 | 传统单体 Web 应用、需要长期运行服务器的多层应用 |

> **考试陷阱**：**两者都号称"只需管理代码，基础设施自动处理"，但抽象的对象层次完全不同**——题目描述"需要部署一个传统的多层 Web 应用，后端服务器需要持续运行" → **Elastic Beanstalk**；题目描述"需要快速搭建一个前端应用/移动应用，希望 Git 推送后自动部署，并简单接入用户认证和 API" → **Amplify**。混淆两者是这一知识点的典型失分点，完整的 Beanstalk 架构见 [[Elastic Beanstalk]] 独立笔记。

---

## 典型应用场景

| 场景 | 推荐配置 |
|------|---------|
| **前端应用希望 Git 推送后自动构建部署** | Amplify Hosting |
| **需要多个功能分支各自独立的预览环境** | Amplify Hosting 分支即环境 |
| **需要为前端应用快速接入用户认证、API 和存储** | Amplify Libraries + Cognito/AppSync/S3 |
| **移动应用需要良好的离线读写体验并自动同步云端** | Amplify DataStore |
| **需要部署传统的多层服务器端应用** | 改用 [[Elastic Beanstalk]]（而非 Amplify） |
| **需要对底层基础设施做精细化的自定义控制** | 直接使用 [[AWS CloudFormation]]/CDK，而非 Amplify 的高层封装 |
| **纯静态网站，且团队已有自己的构建/部署流程** | 可直接使用 [[S3]] 静态托管 + [[CloudFront]]，无需 Amplify 的额外封装 |

---

## 考试重点总结

### SAA-C02 高频考点

1. **Amplify 面向前端/移动开发者，Elastic Beanstalk 面向传统服务器端应用**：两者解决的抽象层次不同
2. **Amplify Hosting 底层基于 S3 + CloudFront**：叠加了 Git CI/CD、分支环境、SSR 支持
3. **分支即环境**：每个 Git 分支可独立部署为一套环境，支持功能分支预览
4. **原子部署 + 即时回滚**：新版本部署全有或全无，出问题可快速回退
5. **Amplify CLI/Studio 底层生成 CloudFormation**：本质仍是 IaC，只是提供了更高层的声明式配置体验
6. **Amplify Libraries 屏蔽底层服务调用细节**：提供按能力分类（Auth/API/Storage）的简单客户端 API
7. **Amplify DataStore 提供离线优先的数据同步能力**：内置冲突解决，适合移动场景
8. **Amplify 无持续运行的服务器概念**：底层依赖无服务器组件（S3/CloudFront/Lambda/AppSync）
9. **需要精细控制基础设施时应绕过 Amplify 直接用 CloudFormation/CDK**：Amplify 的高层封装会牺牲部分自定义灵活性
10. **判断依据是应用类型和团队角色**：前端/移动团队追求开发速度 → Amplify；需要部署传统服务器应用 → Elastic Beanstalk

### 场景题解题思路

```
场景分析 → 判断 AWS Amplify 相关配置
├── "前端应用希望 Git 推送后自动构建部署" → Amplify Hosting
├── "需要多个功能分支的独立预览环境" → Amplify Hosting 分支即环境
├── "需要快速接入认证/API/存储，不想手动配置每个服务" → Amplify Libraries + CLI/Studio
├── "移动应用需要离线读写并自动同步" → Amplify DataStore
├── "需要部署传统多层服务器端应用" → 改用 Elastic Beanstalk（而非 Amplify）
├── "需要对基础设施做精细自定义控制" → 改用 CloudFormation/CDK（而非 Amplify）
└── "纯静态站点，已有自己的构建部署流程" → 可直接用 S3 + CloudFront，无需引入 Amplify
```

---

## 最佳实践

1. **前端/移动新项目优先评估 Amplify 以加快交付速度**：避免团队从零手动拼装 S3/CloudFront/Cognito/AppSync
2. **善用分支即环境做功能预览**：在合并到主分支前先在独立环境验证变更
3. **需要深度自定义基础设施时不要勉强套用 Amplify**：应直接使用 CloudFormation/CDK 获得完整控制力
4. **移动应用重视离线体验时评估 Amplify DataStore**：而不是自行实现本地缓存和同步冲突处理逻辑
5. **明确团队场景是"前端/移动应用"还是"传统服务器应用"再选型**：避免把 Amplify 和 Elastic Beanstalk 的适用边界混淆
6. **保留 Amplify 生成的 CloudFormation 堆栈作为可审计的基础设施记录**：即使使用高层工具，底层仍应可追溯、可版本控制