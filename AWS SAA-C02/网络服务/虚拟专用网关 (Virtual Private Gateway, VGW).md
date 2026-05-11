**虚拟专用网关 (Virtual Private Gateway, VGW)** 是 AWS 网络架构中用于建立私有连接的关键逻辑组件，主要作为您的 [[虚拟私有云 - VPC]] 与外部网络（如本地数据中心）之间的桥梁

## 核心定义：AWS 侧的“锚点”
-  **VPN 集中器：** 在建立站点到站点（Site-to-Site）VPN 连接时，VGW 是 VPC 端的逻辑端点 。您可以将其想象为云端的 VPN 路由器。
- **职责分工：** 它负责处理流量的加密和解密，确保数据在公共互联网传输过程中的安全性

## 关键连接方式
- **Site-to-Site VPN：** VGW 通过公共互联网与您本地侧的物理设备——**客户网关 (Customer Gateway, CGW)** 建立加密的 IPsec 隧道 [cite: 429]。它支持 AES 128 位和 256 位加密
- **Direct Connect (DX)：** 当您使用专用物理线路连接本地与 AWS 时，VGW 也是连接的承载目标之一，帮助流量从专用线路进入特定的

## 重要特性与局限
- **高可用性：** VGW 内部是冗余设计的。例如，在创建 VPN 时，它通常会提供两条隧道（Tunnel），以防单条路径发生故障
- **一对一绑定：** 一个传统的 VGW 只能附加（Attach）到**一个** VPC。
- **不支持传递路由 (Transitive Routing)：** 这是考试中的常见考点。例如，如果 VPC A 通过 VGW 连接了本地机房，即使 VPC A 与 VPC B 建立了对等连接（Peering），本地机房也**不能**通过 VPC A 访问 VPC B

## 架构进阶：VGW vs. Transit Gateway
随着企业规模扩大，手动管理多个 VGW 会变得非常复杂
- **VGW 模式：** 每个 VPC 都需要一个独立的 VGW 及其对应的 VPN 隧道
- **Transit Gateway 模式：** 它可以作为一个中央路由器连接成百上千个 VPC 和本地连接，在此时，Transit Gateway 会取代 VGW 的角色，简化整体网络拓扑