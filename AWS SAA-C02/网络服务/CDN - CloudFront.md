Amazon CloudFront 核心笔记

1. 全球分发与分发机制

- **边缘网络：** 通过全球边缘节点 (Edge Locations) 提供低延迟服务。
- **缓存逻辑与缓存键：** 默认基于 URL 缓存。若要支持多语言或动态内容，需配置 CloudFront 转发并基于 **查询字符串 (Query Strings)**、**Cookie** 或 **HTTP 标头** 进行缓存。
- **分发类型：**
    - **Web 分发：** 用于 HTTP/HTTPS 网页及静态/动态内容。
    - **RTMP 分发：** 专门用于分发存储在 S3 中的 Adobe RTMP 视频流。

2. 源 (Origins) 的多样性

CloudFront 不仅支持 S3，还支持多种源类型：

- **[[S3 - Object Storage]] 存储桶：** 最常见的对象存储源 。
- **负载均衡器 ([[AWS Load Balance]])：** 用于处理后端 EC2 实例集群的流量。
- **媒体服务：** 包括 AWS MediaPackage 和 MediaStore 。
- **自定义源：** 任何具有公共域名或 IP 的服务器（如本地服务器）。

3. 核心集成与性能优化

- **S3 传输加速 (Transfer Acceleration)：** 这是一个 **S3 存储桶级别** 的配置。它利用 CloudFront 的边缘节点作为入口点，通过亚马逊高速内部网络将数据上传至 S3，显著提升跨国上传速度 。
- **Route 53 路由：** 可将自定义域名（如 `www.example.com`）关联至 CloudFront 分发，实现用户访问的透明引导。

4. 边缘层面的安全性（多层防御）

- **源访问身份 (OAI)：** 创建特殊的虚拟身份并修改 S3 桶策略，确保用户 **只能** 通过 CloudFront 访问文件，无法绕过 CDN 直连 S3 下载 。
- **传输加密 (ACM)：** 在边缘节点部署 SSL/TLS 证书。注意：若要在 CloudFront 上使用第三方证书，必须将其导入 **us-east-1 (N. Virginia)** 区域。
- **DDoS 保护 (AWS Shield)：**
    - **Standard：** 默认开启，防御 3/4 层（如 SYN 洪水）攻击。
    - **Advanced：** 提供 7 层防御（如 HTTP Flood）及全天候专家支持。
- **WAF 集成：** 在边缘拦截 SQL 注入和跨站脚本 (XSS) 攻击，保护后端源服务器 。

3. **CloudFront 缓存行为与策略 (Cache Behavior & Policies)**

- **默认缓存行为：** CloudFront 默认仅根据完整 URL 来缓存内容。这意味着对于 `example.com/home?lang=en` 和 `?lang=zh`，如果未进行特殊配置，它可能无法区分并提供正确的版本。
- **缓存键 (Cache Key)：** 您可以自定义缓存键，将以下元素包含在内以区分缓存副本：
    - **查询字符串 (Query Strings)：** 如错题所示，这是处理多语言或过滤结果的核心配置。
    - **Cookie：** 用于根据用户会话或偏好（如登录状态）提供个性化内容。
    - **HTTP 标头 (Headers)：** 例如根据 `Accept-Language` 标头缓存不同语言的版本 。
- **TTL (Time to Live)：** 控制内容在边缘节点的驻留时间。您可以设置最小、最大和默认 TTL，或通过源服务器的 `Cache-Control` 标头进行控制。
- **失效请求 (Invalidation)：** 如果源站内容提前更新，您可以手动发起“失效”请求，强制边缘节点从源站拉取最新版本，但这通常会产生额外费用 。

6. 成本管理与控制

- **价格类别 (Price Class)：** 允许选择边缘节点的覆盖范围（例如“仅美国/欧洲”），通过限制节点分布来降低月度费用。
- **内容压缩：** CloudFront 可自动压缩（如 Gzip）响应请求的文件，不仅能提升下载速度，还能显著减少通过 AWS 网络流出的流量费。