Amazon Cognito 是 AWS 安全与身份体系中专门为移动和 Web 应用程序设计的身份管理服务

## Cognito 主要通过两个核心组件平衡， 身份验证和授权
- 用户池 - User pools: 专注于身份验证 Authentication
- 身份池 - Identity Pools: 专注于授权，提供AWS 凭证能使其访问S3 或者DynamoDB等资源

## 在安全和身份体系的宏观定位
- 身份联邦 - Identity Federation ： Cognito 时实现身份联邦的关键工具，允许外部用户，如Google，Facebook 等无需创建AMI 用户的情况下访问AWS 资源
- 最小特权原则： 通过身份池和特定IAM角色关联，可以精确控制移动端用户的数据访问权限
- 目录集成： 隶属于AWS Directory Service家住，能与其它身份提供商协同工作，简化了跨平台的身份管理逻辑