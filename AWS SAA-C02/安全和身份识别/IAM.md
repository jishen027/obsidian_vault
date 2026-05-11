IAM 核心身份 - Identities

- **Root 用户：** 账号创建时默认产生，拥有最高权限。最佳实践：启用 **MFA**、删除所有 **访问密钥 (Access Keys)**、仅在必要时（如修改账单方案）使用。
- **IAM 用户 (Users)：** 持久身份。可分配**控制台密码**或**程序化访问密钥**。遵循“最小特权原则”。
- **IAM 组 (Groups)：** 用户的集合。注意：组**不是**“可信实体 (Principal)”，不能被直接假设，仅用于简化权限分配。
- **IAM 角色 (Roles)：** 临时身份，不具备长期凭证。通过 **AWS STS** 获取包含 `Access Key ID`、`Secret Access Key` 和 `Session Token` 的临时令牌。

授权与访问控制

- **IAM 策略 (Policies)：** 使用 **JSON** 格式编写。IAM 策略是**全局性**的，不局限于特定区域。
    - **冲突解决：** 显式的 **Deny (拒绝)** 优先级始终高于 Allow (允许)。
- **权限边界 (Permission Boundaries)：** 用于设置身份权限的“天花板”，即使附加了 `AdministratorAccess`，也无法超越此边界定义的范围。

身份识别的扩展与集成 (核心补充)

- **AWS STS (Security Token Service)：** IAM 权限分配的技术支撑，专门负责发放有时效性的临时凭证（默认几小时后过期）。
- **身份联邦 (Federation) 与 SAML 2.0：**
    - **SAML 2.0：** 企业级 **SSO** 的行业标准协议。
    - **工作流：** 本地系统验证用户 -> 发送 SAML 断言给 AWS -> **STS** 发放临时令牌 -> 用户登录控制台。
- **辅助身份工具：**
    - **[[Cognito]]：** 面向**外部 App 注册用户**。
    - **Managed Microsoft AD：** 将本地 AD 域扩展到云端 VPC