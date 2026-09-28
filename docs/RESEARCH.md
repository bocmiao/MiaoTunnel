# MiaoTunnel 竞品与 GitHub 调研

更新日期：2026-09-28

## 调研范围

调研以项目官方 GitHub 仓库和官方文档为主，分为四类：

1. 自托管反向隧道；
2. 开发者临时公开服务；
3. 零信任组网与私网接入；
4. 商业托管隧道。

优秀项目清单可参考 [awesome-tunneling](https://github.com/anderspitman/awesome-tunneling)。

## 代表项目

| 项目 | 类型 | 值得借鉴 | MiaoTunnel 不直接照搬的部分 |
| --- | --- | --- | --- |
| [FRP](https://github.com/fatedier/frp) | 自托管反向代理 | TCP、UDP、HTTP、HTTPS、P2P、QUIC、监控、插件等能力成熟 | 配置项多；证书、域名和生命周期管理仍需要用户整合 |
| [rathole](https://github.com/rathole-org/rathole) | 高性能反向隧道 | Rust、低资源占用、Noise/TLS、按服务令牌、热加载 | 控制面、域名、自动 HTTPS 和用户管理不是重点 |
| [NPS](https://github.com/ehang-io/nps) | 反向代理与 Web 管理 | 服务端管理、多协议、国内用户认知度 | 复用前需单独评估维护状态、安全边界和现代化改造成本 |
| [Pangolin](https://github.com/fosrl/pangolin) | 身份感知的自托管访问平台 | WireGuard、可视化管理、身份与访问控制、自动证书；是最接近的产品竞品 | 产品范围已扩展到 SASE/零信任平台，部署和概念比家庭用户场景更重 |
| [bore](https://github.com/ekzhang/bore) | 极简 TCP 隧道 | 命令简单、实现小、适合验证核心链路 | 缺少 HTTP 域名、证书、管理、审计和多用户能力 |
| [chisel](https://github.com/jpillora/chisel) | 基于 HTTP/WebSocket 的 TCP 隧道 | 单文件、SSH 加密、穿越受限网络能力强 | 更偏底层工具，不是完整管理产品 |
| [zrok](https://github.com/openziti/zrok) | 公共和私有资源分享 | 公私分享模型、易用 CLI、OpenZiti 安全网络 | 底层体系和部署模型较复杂 |
| [Headscale](https://github.com/juanfont/headscale) | Tailscale 控制面替代 | 自托管设备组网、成熟客户端生态 | 重点是私网 mesh，不是面向公众的自定义域名反向代理 |
| [NetBird](https://github.com/netbirdio/netbird) | WireGuard 零信任组网 | P2P、回退中继、访问策略、团队管理 | 系统范围较大，不适合作为极简首版 |
| [Cloudflare Tunnel](https://developers.cloudflare.com/cloudflare-one/networks/connectors/cloudflare-tunnel/) | 商业托管隧道 | 出站连接、无公网源站、全球入口、WAF/Access 集成 | 依赖 Cloudflare 账户、网络与产品边界，不能完全自托管 |
| [ngrok](https://ngrok.com/docs/guides/share-localhost/tunnels) | 商业开发者隧道 | 极佳的首次使用体验、自动 HTTPS、调试与策略能力 | SaaS 依赖、套餐限制和长期成本 |
| [Tailscale Funnel](https://tailscale.com/docs/features/tailscale-funnel) | 基于 tailnet 的公开入口 | 与私网设备整合、自动 HTTPS、双重启用 | 公开入口和端口受平台规则约束，控制面不是自托管 |

## 市场空位

底层转发能力已经很成熟，真正仍有空间的是“把自托管变简单”：

- 安装时自动检查 DNS、端口、时间同步和防火墙；
- 可视化创建服务，而不是手写两端配置；
- 自动签发 HTTPS，并清楚区分公开服务与私有服务；
- 设备注册、撤销、轮换和审计默认可用；
- 对群晖、绿联、飞牛、Unraid、Home Assistant 等家庭服务器提供模板；
- 故障提示说人话：是 DNS、证书、隧道、目标端口还是客户端离线。

## 首版建议

### 选择：复用 FRP 数据面

原因：

- 协议覆盖足够，尤其适合后续增加 TCP、UDP 和 P2P；
- Go 技术栈便于与控制面和 agent 统一；
- 项目成熟，部署资料多；
- 通过适配器隔离后，未来可换 rathole 或自研数据面。

### 不选择：立即自研协议

首版自研协议会同时引入多路复用、重连、流控、心跳、TLS、NAT、兼容性和压测问题，但这些并不是用户最急迫的痛点。先验证部署体验和安全管理更合理。

### 入口选择：Caddy

Caddy 负责公开 HTTP/HTTPS、证书与域名路由；隧道数据面只负责可靠转发。自动证书必须通过控制面的域名授权接口限制，不能开放任意域名签发。

## 差异化路线

1. **NAS 优先**：把家庭服务发布做成模板；
2. **中文故障诊断**：一页展示域名、证书、网关、agent 和目标服务状态；
3. **默认安全**：一次性注册、设备撤销、最小日志、禁止匿名中继；
4. **单机可用**：一台廉价 VPS + Docker Compose 即可运行；
5. **可迁移**：用户可导出配置、证书元数据和审计记录，不绑定托管平台。

## 主要风险

| 风险 | 首版措施 |
| --- | --- |
| 被用作开放代理或攻击跳板 | 不支持匿名注册；端口白名单；限速；设备与服务可撤销 |
| 注册令牌泄露 | 短有效期、一次性、哈希保存、使用后立即失效 |
| 公网管理面被撞库 | 强密码、登录限速、安全 Cookie；2FA 在公开测试前完成 |
| 证书签发被滥用 | 仅允许已验证域名或平台子域名；ACME 限速与审计 |
| 日志泄露凭据 | 默认只记元数据，不记录正文、Cookie 或授权头 |
| 自动更新供应链风险 | v0.1 不做静默自更新；发布包校验，后续增加签名 |
| 第三方许可证不兼容 | 数据面和入口以独立进程集成；保留第三方声明并逐项审计 |
