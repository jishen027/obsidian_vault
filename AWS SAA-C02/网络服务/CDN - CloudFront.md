# Amazon CloudFront - CDN 服务

> **Amazon CloudFront** 是 AWS 的全球内容分发网络（CDN），通过遍布全球的边缘节点（Edge Locations）缓存内容，为用户提供低延迟、高传输速度的静态与动态内容分发。
>
> 相关文档：[[S3 - Object Storage]] | [[AWS Load Balance]] | [[Route 53 DNS]] | [[IAM]]

---

## 核心概念

### CloudFront 定位

| 服务 | 类型 | 场景 | 特点 |
|------|------|------|------|
| **CloudFront** | CDN | 全球加速静态/动态内容分发 | 边缘缓存、就近接入、降低源站负载 |
| [[S3 - Object Storage]] | 对象存储 | 存储源内容 | 常作为 CloudFront 的源站之一 |
| [[AWS Load Balance]] | 负载均衡 | 区域内流量分发 | 常作为 CloudFront 的动态内容源站 |
| [[Route 53 DNS]] | DNS | 域名解析 | 通过 Alias 记录将自定义域名指向 CloudFront |

### 边缘网络架构

- **边缘节点（Edge Locations）**：分布全球的缓存节点，为终端用户提供最近的接入点，是 CloudFront 实现低延迟的核心
- **区域边缘缓存（Regional Edge Caches）**：位于边缘节点和源站之间的二级缓存层，容量更大、缓存时间更长，减少未命中时穿透回源站的频率
- **请求路径**：用户请求 → 最近边缘节点 → 命中则直接返回；未命中 → 区域边缘缓存 → 仍未命中才回源站拉取

---

## 分发类型

| 类型 | 用途 | 状态 |
|------|------|------|
| **Web 分发** | HTTP/HTTPS 网页、静态资源、动态内容、API 加速 | 主流类型，目前唯一推荐使用的方式 |
| **RTMP 分发** | 早期用于分发 S3 中的 Adobe Flash 流媒体 | **已废弃（Deprecated）**，AWS 已停止支持，考试中如出现仅作为历史知识识别，不应作为答案选择 |

---

## 源（Origins）的多样性

| 源类型 | 说明 |
|--------|------|
| **[[S3 - Object Storage]] 存储桶** | 最常见的对象存储源，通常配合 OAC 限制直连访问 |
| **[[AWS Load Balance]]** | 用于分发后端 EC2 实例集群提供的动态内容 |
| **媒体服务** | AWS MediaPackage、MediaStore 等流媒体源 |
| **自定义源（Custom Origin）** | 任意具有公网域名或 IP 的服务器，包括本地数据中心服务器 |

> **考试要点**：一个 CloudFront 分发可以配置**多个源**，并通过**缓存行为（Cache Behavior）**基于路径模式（如 `/api/*` 走 ALB，`/static/*` 走 S3）将请求路由到不同源站。

---

## 缓存行为与策略（考试高频）

### 默认缓存逻辑

- CloudFront **默认仅基于完整 URL** 缓存内容，不区分查询字符串、Cookie、请求头的差异
- 例如 `example.com/home?lang=en` 和 `example.com/home?lang=zh` 在默认配置下可能被当作**同一缓存对象**，导致用户看到错误语言版本

### 缓存键（Cache Key）自定义

| 维度 | 用途 |
|------|------|
| **查询字符串（Query Strings）** | 处理多语言、分页、过滤结果等按参数区分内容的场景 |
| **Cookie** | 根据登录状态、用户偏好返回个性化内容 |
| **HTTP 请求头（Headers）** | 如根据 `Accept-Language` 缓存不同语言版本 |

> **考试陷阱**：将查询字符串、Cookie、Header 纳入缓存键会**提高缓存命中的精确度**，但也会**降低整体缓存命中率**（因为缓存副本数量增多），需要在个性化和缓存效率之间权衡。

### TTL（生存时间）

- 可设置**最小 TTL、最大 TTL、默认 TTL**，控制内容在边缘节点的驻留时间
- 也可完全由源站的 `Cache-Control` / `Expires` 响应头控制缓存时长

### 失效请求（Invalidation）

- 当源站内容已更新但边缘缓存尚未过期时，可手动发起**失效请求**强制边缘节点回源拉取最新版本
- 失效请求**按数量计费**（超出免费额度后），频繁失效不如**使用带版本号的文件名**（如 `app.v2.js`）经济高效

---

## 安全性（多层防御，考试高频）

### 源访问控制：OAC vs OAI

| 机制 | 状态 | 说明 |
|------|------|------|
| **源访问控制（Origin Access Control, OAC）** | **AWS 当前推荐** | 支持所有 S3 区域、SSE-KMS 加密对象、POST/PUT 方法，安全性和功能均优于 OAI |
| **源访问身份（Origin Access Identity, OAI）** | legacy，仍可用但不再推荐新建 | 创建虚拟身份并写入 S3 桶策略，确保用户只能通过 CloudFront 访问，无法绕过 CDN 直连 S3 |

两者的共同目标：将 S3 Bucket 设为私有，仅允许来自 CloudFront 的请求读取对象，配合 **Block Public Access** 防止用户绕过 CDN 直接访问源站产生的流量成本和缓存失效风险。

### 传输加密（HTTPS/ACM）

- 在边缘节点部署 SSL/TLS 证书，实现用户到边缘节点的 HTTPS 加密
- **考试陷阱**：为 CloudFront 使用 AWS Certificate Manager（ACM）颁发或导入第三方证书时，证书**必须位于 us-east-1（N. Virginia）区域**，无论分发服务的用户在哪个地理区域，否则证书不会出现在 CloudFront 的证书选择列表中

### DDoS 防护（AWS Shield）

| 层级 | 说明 |
|------|------|
| **Standard** | 所有 CloudFront 用户默认自动开启，免费防御第 3/4 层攻击（如 SYN 洪水） |
| **Advanced** | 额外付费订阅，提供第 7 层攻击防护（如 HTTP Flood）、实时攻击可视化、DDoS 响应团队（DRT）支持，并附带费用保护 |

### WAF 集成

- 在 CloudFront 边缘层部署 **AWS WAF**，可拦截 SQL 注入、跨站脚本（XSS）等攻击，在流量到达源站之前完成过滤
- WAF 规则可基于 IP 黑白名单、地理位置、速率限制（Rate-based Rule）等条件精细过滤

---

## 与 Route 53 集成

- 可将自定义域名（如 `www.example.com`）通过 [[Route 53 DNS]] 的 **Alias 记录**指向 CloudFront 分发
- Alias 记录指向 CloudFront 时**查询免费**，且能自动感知分发状态变化，优于使用 CNAME

---

## 成本管理与性能优化

### 价格类别（Price Class）

| 类别 | 覆盖范围 | 成本 |
|------|---------|------|
| **Price Class All** | 全球所有边缘节点 | 最高，覆盖最广、延迟最低 |
| **Price Class 200** | 覆盖大部分区域，不含最贵的边缘节点 | 中等 |
| **Price Class 100** | 仅北美、欧洲等成本最低的边缘节点 | 最低，适合用户集中在特定区域的场景 |

### 内容压缩

- CloudFront 可自动对响应内容进行压缩（如 Gzip、Brotli）
- 不仅提升下载速度，还能显著降低**从 AWS 网络流出的数据传输费用**（按压缩后字节计费）

### S3 Transfer Acceleration（易混淆点）

> **考试陷阱**：S3 Transfer Acceleration 与 CloudFront 分发是**两个不同的功能**，容易混淆。

| 特性 | CloudFront | S3 Transfer Acceleration |
|------|-----------|--------------------------|
| **配置层级** | 独立的分发资源 | **S3 存储桶级别**的开关 |
| **方向** | 主要用于**下行**分发内容给用户 | 主要用于**上行**加速用户向 S3 上传 |
| **机制** | 边缘节点缓存内容后就近提供 | 利用边缘节点作为入口，通过 AWS 高速内部网络转发上传流量到 S3，不缓存 |
| **典型场景** | 静态网站、视频点播、API 加速 | 全球用户向同一 S3 Bucket 高速上传大文件 |

---

## 典型应用场景

| 场景 | 推荐配置 |
|------|---------|
| **静态网站加速** | S3（私有源，OAC）+ CloudFront + Route 53 Alias |
| **动态 API 加速** | ALB/API Gateway 作为源 + CloudFront（按路径配置缓存行为） |
| **视频点播/直播** | MediaPackage/MediaStore + CloudFront |
| **全球用户高速上传** | S3 Transfer Acceleration（非 CloudFront 分发） |
| **多语言/个性化网站** | CloudFront 自定义缓存键（Header/Cookie/Query String） |
| **抵御 Web 攻击** | CloudFront + AWS WAF + Shield Advanced |

---

## 考试重点总结

### SAA-C02 高频考点

1. **定位**：全球 CDN，通过边缘节点缓存降低延迟并减轻源站负载
2. **OAC 优于 OAI**：新架构应使用 OAC，OAI 是仍可用的遗留机制
3. **证书区域限制**：CloudFront 使用的 ACM 证书必须在 us-east-1 区域
4. **默认仅按 URL 缓存**：需要按语言/用户区分内容时必须显式配置缓存键（Query String/Cookie/Header）
5. **缓存键维度增多会降低命中率**：需要在个性化和缓存效率之间权衡
6. **Shield Standard 默认免费开启**，Advanced 需额外付费订阅并提供 7 层防护
7. **CloudFront ≠ S3 Transfer Acceleration**：前者是下行内容分发，后者是 S3 桶级别的上传加速
8. **失效请求收费**：优先用带版本号的文件名代替频繁 Invalidation
9. **多源 + 多缓存行为**：可按路径模式将流量路由到不同源站（S3/ALB 混合架构）
10. **RTMP 分发已废弃**：不应出现在正确答案中

### 场景题解题思路

```
场景分析 → 选择 CloudFront 配置
├── "全球用户访问静态网站，需要低延迟" → CloudFront + S3 源（OAC）
├── "希望用户只能通过 CDN 访问 S3，无法绕过直连" → OAC（或遗留场景下的 OAI）
├── "多语言网站，同一 URL 需返回不同语言内容" → 自定义缓存键（Header/Cookie/Query String）
├── "证书导入后 CloudFront 找不到" → 检查证书是否在 us-east-1 区域
├── "源站内容已更新，边缘节点仍返回旧内容" → 发起 Invalidation，或改用带版本号文件名
├── "需要防御 SQL 注入/XSS" → CloudFront + AWS WAF
├── "需要 7 层 DDoS 防护和专家支持" → AWS Shield Advanced
├── "全球用户需要更快地向同一 S3 Bucket 上传大文件" → S3 Transfer Acceleration（非 CloudFront）
└── "希望降低边缘节点覆盖成本，用户集中在北美/欧洲" → Price Class 100
```

---

## 最佳实践

1. **新架构统一使用 OAC**：功能和安全性优于遗留的 OAI
2. **S3 源启用 Block Public Access**：配合 OAC 确保内容只能经由 CloudFront 访问
3. **按需自定义缓存键**：仅将真正影响内容差异的参数纳入缓存键，避免命中率无谓下降
4. **静态资源使用版本化文件名**：减少对 Invalidation 的依赖，降低成本
5. **ACM 证书统一在 us-east-1 申请**：避免证书列表中找不到目标证书的问题
6. **启用压缩**：同时提升性能并降低流量费用
7. **多源架构按路径拆分缓存行为**：静态内容走 S3，动态内容走 ALB，各自应用最优缓存策略
8. **叠加 WAF + Shield Advanced**：面向公网的关键业务应在边缘层构筑多层防御
9. **合理选择 Price Class**：用户地域集中时降低价格类别以节省成本
