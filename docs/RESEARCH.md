# MiaoTunnel 竞品与 GitHub 深度调研

更新日期：2026-09-28

## 1. 结论先行

MiaoTunnel 不应再造一个通用隧道内核，也不应直接复制一个已有的全栈项目。更合理的产品定义是：

> 面向 NAS、家庭实验室和小团队的自托管发布器，用成熟数据面完成连接，用中文向导、安全默认值和分层诊断降低使用门槛。

首版建议：Go 单体控制面 + 独立 `miao-agent` + FRP 数据面 + Caddy HTTPS 入口 + SQLite。v0.1 只发布 HTTP/HTTPS 服务；裸 TCP、UDP、P2P、多租户和高可用均延后。

关键依据：

- FRP 已覆盖传输、复用、重连、TLS、带宽限制和服务端插件，没有必要重写；
- FRP 的 `Login`、`NewProxy` 等插件回调足以实现每设备授权和服务白名单；
- Caddy 能自动签发证书并原子更新配置，但必须只为控制面明确批准的域名创建路由；
- Wiredoor 与原始构想高度重叠，证明需求成立，也说明“WireGuard + NGINX + TypeScript/Vue 全栈”不是最轻的首版；
- 国内产品的优势主要在一键安装、控制台、套餐节点和教程，开源自托管项目仍有“数据归属 + NAS 模板 + 可诊断”的空间。

## 2. 数据口径

- GitHub 数据来自仓库与 Releases API，快照时间为 2026-09-28；Star、Issue 和版本会继续变化。
- 功能只采用项目 README、官方文档或官方产品页；不把第三方评测当作事实来源。
- 性能描述若来自项目自身，一律标注为“项目自述”，不同测试环境之间不横向比较。
- “活跃度”只作为维护风险信号，不等于代码质量或安全性。
- 商业价格为当日页面快照，不作为长期承诺。

## 3. 开源项目数据

| 项目 | Star | Fork | Open issue | 语言 | 许可证 | 最新发布 | 最近 push | 判断 |
| --- | ---: | ---: | ---: | --- | --- | --- | --- | --- |
| [FRP](https://github.com/fatedier/frp) | 109,659 | 15,227 | 46 | Go | Apache-2.0 | v0.71.0 / 2026-08-14 | 2026-09-15 | 最适合做首版数据面 |
| [Headscale](https://github.com/juanfont/headscale) | 44,175 | 2,588 | 142 | Go | BSD-3-Clause | v0.29.4 / 2026-09-23 | 2026-09-27 | 适合私网组网，不是公开入口核心 |
| [NPS](https://github.com/ehang-io/nps) | 34,235 | 6,072 | 527 | Go | GPL-3.0 | v0.26.10 / 2021-04-08 | 2024-05-30 | 功能相关，但发布陈旧、维护风险高 |
| [NetBird](https://github.com/netbirdio/netbird) | 29,564 | 1,712 | 1,415 | Go | 混合许可证 | v0.79.0 / 2026-09-18 | 2026-09-26 | 零信任组网完整，但范围过大 |
| [Pangolin](https://github.com/fosrl/pangolin) | 22,937 | 790 | 113 | TypeScript | CE 为 AGPL-3.0；另有商业版 | 1.23.0 / 2026-09-16 | 2026-09-28 | 最接近完整产品竞品，明显偏重 |
| [localtunnel](https://github.com/localtunnel/localtunnel) | 22,488 | 1,569 | 167 | JavaScript | MIT | 未使用 GitHub Release | 2025-08-29 | 临时开发隧道体验参考 |
| [Chisel](https://github.com/jpillora/chisel) | 16,591 | 1,609 | 245 | Go | MIT | v1.12.0 / 2026-08-29 | 2026-09-01 | 单文件和受限网络参考 |
| [rathole](https://github.com/rathole-org/rathole) | 14,263 | 835 | 98 | Rust | Apache-2.0 | v0.5.0 / 2023-10-01 | 2026-08-23 | 精简备选数据面；正式发布较旧 |
| [bore](https://github.com/ekzhang/bore) | 11,510 | 528 | 17 | Rust | MIT | v0.6.0 / 2025-06-09 | 2026-02-04 | 极简 TCP 工具，不是管理产品 |
| [GOST](https://github.com/go-gost/gost) | 7,545 | 832 | 103 | Go | MIT | v3.3.0 / 2026-08-30 | 2026-09-22 | 协议丰富，但容易越界成通用代理 |
| [zrok](https://github.com/openziti/zrok) | 4,723 | 224 | 126 | Go | Apache-2.0 | v2.0.5 / 2026-09-27 | 2026-09-27 | 公私分享模型好，OpenZiti 体系偏重 |
| [sish](https://github.com/antoniomika/sish) | 4,738 | 335 | 25 | Go | MIT | v2.23.0 / 2026-06-05 | 2026-06-25 | 无客户端 SSH 隧道体验参考 |
| [Piko](https://github.com/andydunstall/piko) | 2,193 | 88 | 3 | Go | MIT | v0.10.0 / 2026-05-08 | 2026-09-22 | 面向集群/生产流量，非首版优先级 |
| [Wiredoor](https://github.com/wiredoor/wiredoor) | 1,621 | 78 | 9 | TypeScript | Apache-2.0 | v1.8.0 / 2026-09-22 | 2026-09-22 | 最直接竞品与设计参照 |
| [boringproxy](https://github.com/boringproxy/boringproxy) | 1,385 | 132 | 60 | Go | MIT | v0.10.0 / 2023-01-04 | 2024-07-06 | 简洁 Web 发布体验参考，维护偏慢 |

补充基础设施：[Caddy](https://github.com/caddyserver/caddy) 在同一快照下为 76,125 Star、5,010 Fork、Apache-2.0，最新发布 v2.11.4（2026-06-03）。它不是竞品，而是候选 HTTPS 入口。

## 4. 代表项目拆解

### 4.1 FRP：复用数据面，不复刻它的用户体验

可直接复用：

- TCP、UDP、HTTP、HTTPS、QUIC/KCP、连接池、TCP 复用、健康检查和带宽限制；
- 强制 TLS、token/OIDC 鉴权、端口白名单、Prometheus 指标；
- 本地管理 API 和动态代理持久化；
- [服务端插件](https://github.com/fatedier/frp/blob/dev/doc/server_plugin.md)在 `Login`、`NewProxy`、`CloseProxy`、`Ping`、`NewWorkConn`、`NewUserConn` 阶段允许、拒绝或改写请求；
- 全局与每代理 metadata 可传给授权插件。

不足：上游也承认配置、认证、证书与 API 管理仍不够现代。MiaoTunnel 的价值正是补齐注册、撤销、域名、安全默认值和诊断，不修改 FRP 协议。

### 4.2 Wiredoor：需求验证和 UX 标杆，不直接整仓 fork

Wiredoor 与 MiaoTunnel 旧方案几乎一一对应：自托管入口、WireGuard、HTTP/TCP/UDP、NGINX、Let's Encrypt、OAuth2/IP 规则、Web 控制台、CLI、Docker/Kubernetes、Prometheus/Grafana。

其公开部署要求包括公网 Linux、Docker/Compose、80/443 TCP、443 UDP、51820 UDP，以及一段 TCP/UDP 业务端口。远端节点主动建立 WireGuard。官方安全文档强调节点独立 token、网关节点的额外风险、最小子网范围和反向代理后的真实 IP 问题。

不直接 fork 的原因：

- 主仓为 Express + TypeORM + SQLite/MySQL + Vue/Vite + NGINX + WireGuard，首版维护面过大；
- WireGuard 依赖 UDP，在部分受限网络中不如 FRP 的 TCP 起步稳妥；
- 直接 fork 会长期承担上游同步、前后端依赖和迁移成本；
- 产品差异会退化成汉化与打包，难以形成独立边界。

应借鉴：节点/网关区分、分层排障、OAuth2/IP 规则、Docker 开箱部署和清晰的安全警告。

### 4.3 Pangolin、zrok、NetBird：身份产品的上限参考

- Pangolin 已扩展到 SASE、PAM、AI Gateway、RBAC 和审计，说明完整身份平台需求真实，也说明范围很容易失控；
- zrok 的 public/private share 模型和命令体验值得借鉴，但 OpenZiti 依赖不适合最小首版；
- NetBird/Headscale 擅长设备组网。若未来重点转为“任何设备访问整个私网”，应集成或推荐它们，而不是把 MiaoTunnel 变成另一个 mesh VPN。

### 4.4 sish、bore、Chisel：首次使用体验参考

- sish 证明“只有 SSH 客户端也能开隧道”的低门槛价值；
- bore 证明极小命令面有利于理解和排障；
- Chisel 证明基于 HTTP/WebSocket 的单文件工具适合受限网络。

这些项目适合临时开发隧道，不具备 MiaoTunnel 需要的长期设备身份、自动证书、审计和安全发布生命周期。

## 5. 托管产品可借鉴之处

| 产品 | 官方能力与限制 | 应借鉴 | 不照搬 |
| --- | --- | --- | --- |
| [Cloudflare Tunnel](https://developers.cloudflare.com/cloudflare-one/networks/connectors/cloudflare-tunnel/) | 连接器主动出站；源站无需公网入站；通过 QUIC/HTTP2 连接 Cloudflare | 出站式安装、冗余连接、清晰健康状态 | 平台锁定、全球边缘网络不是自托管项目能复制的 |
| [ngrok](https://ngrok.com/docs/universal-gateway/) | HTTP/HTTPS/TCP/TLS 端点；TLS、mTLS、策略能力 | 首次运行速度、即时 URL、检查与策略 | SaaS 控制面和套餐模式 |
| [Tailscale Funnel](https://tailscale.com/docs/features/tailscale-funnel) | 只使用 tailnet 域名；仅 443/8443/10000；TLS；存在不可配置带宽限制 | 用限制换简单、公开动作需要双重确认 | 依赖 tailnet 与托管控制面 |
| [Pangolin](https://docs.pangolin.net/) | 浏览器公开访问、私有客户端访问、SSO/OAuth/OTP、自动证书 | 公私模式、身份前置、资源而非端口的表述 | SASE 全家桶和企业功能 |

共同规律：用户不愿理解隧道配置；他们要的是“选择设备和服务，得到一个可访问地址，并知道失败在哪一层”。

## 6. 国内产品与价格快照

| 产品 | 2026-09-28 官方页面信息 | 产品启示 |
| --- | --- | --- |
| [cpolar](https://www.cpolar.com/pricing) | 免费/基础档标示 2 Mbps；Pro ¥149/年、3 Mbps、5 个保留域名、2 个 TCP 地址；Business ¥204/年并增加 IP 白名单、通配域名等；NAS 方案从 ¥600/年起 | 套餐把带宽、固定域名、TCP 地址和安全规则作为主要价值 |
| [NATAPP](https://natapp.cn/tunnel/buy) | 支持 HTTP/HTTPS/TCP；示例 HK_2 ¥12/月，100M 带宽并按流量计费 2 元/GB | 低价节点和固定域名降低尝试成本，但长期依赖服务商 |
| [花生壳](https://hsk.oray.com/) | HTTP/HTTPS/TCP/UDP、Docker 客户端、访问时间/地区/IP/浏览器/OS 规则、诊断与 OpenAPI | 模板、诊断、细粒度访问规则是成熟产品标配 |
| [SakuraFrp](https://www.natfrp.com/pricing/) | 免费 10 Mibps、2 隧道、5 GiB/月；Silver VIP ¥20/月、36 Mibps、10 隧道与更高流量；提供 NAS 教程和自动 HTTPS | 节点选择、流量透明、NAS 教程和状态信息很重要 |
| [OpenFrp](https://www.openfrp.net/) | 多配置、免手写配置、自动 TLS/HTTPS、游戏 TCP/UDP 场景 | 国内用户期待“一键启动”和场景化配置 |

价格可能随时调整，表格只用于理解商业模型。MiaoTunnel 不应承诺“免费带宽”或“更快线路”：自托管速度主要由用户 VPS、运营商路由与本地上行决定。

## 7. 市场空位与目标用户

优先用户：

1. 有公网 VPS 和 NAS/家庭服务器，但不想手写 FRP/Caddy 配置的人；
2. 需要长期发布 Home Assistant、相册、开发预览或 webhook 的个人开发者；
3. 需要把一个内部 Web 工具安全交给少量同事访问的小团队。

不是首版用户：需要全球边缘加速、企业 SSO/合规、多租户计费、整网 VPN、游戏 UDP 或匿名临时隧道的人。

可形成差异的五件事：

- 中文安装和故障解释；
- 绿联、群晖、飞牛、Unraid 等 NAS 场景模板；
- 默认受保护，公开访问需要明确确认；
- 自托管、可导出、不依赖某个云账号；
- 从 DNS、证书、网关、隧道、agent 到目标端口的逐层诊断。

## 8. 最终技术决策

候选方案用 1–5 分评估。权重是本项目的决策模型，不是外部测评。

| 方案 | 成熟度 20% | 安全钩子 20% | 运维简单 20% | 网络适应 15% | 可差异化 15% | 许可 10% | 加权结果 |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| FRP + 自研薄控制面 | 5 | 4 | 4 | 5 | 5 | 5 | **4.60/5** |
| fork Wiredoor | 4 | 4 | 2 | 3 | 2 | 5 | 3.25/5 |
| rathole + 自研控制面 | 3 | 3 | 4 | 3 | 5 | 5 | 3.70/5 |
| 从零自研协议 | 1 | 2 | 1 | 2 | 4 | 5 | 2.20/5 |

采用第一项，并设置边界：

- FRP 作为独立进程和可替换适配器，不复制其源码；
- MiaoTunnel 控制面负责身份、期望状态、域名、Caddy 配置、审计和诊断；
- FRP 插件只做登录与代理创建授权，v0.1 不把每个访客连接都同步依赖控制面；
- 首版以 TCP 传输连接网关，QUIC/KCP 只在真实网络数据证明需要后增加；
- 不提供正向代理、通用出口、匿名中继或绕过网络控制的功能。

## 9. 仍需用原型验证的假设

| 假设 | 验证方法 | 通过标准 |
| --- | --- | --- |
| Caddy 可通过受限管理通道安全原子更新 | Unix socket/隔离网络 spike，反复增删 100 条路由 | 管理端口不暴露；更新失败保留旧配置 |
| FRP 插件可可靠拦截伪造 agent 与越权域名 | 伪造 token、service ID、domain、proxy type | 全部拒绝并产生脱敏审计事件 |
| 控制面重启不影响已建立 HTTP 数据面 | 传输中重启 miao-server | 既有连接继续或可接受地重试；新隧道被拒绝而非放行 |
| NAS 容器网络可稳定访问局域网目标 | 绿联/群晖至少各一台实测 | 能检测目标、解释 host/bridge 网络问题 |
| 一台低配 VPS 足以承载家庭场景 | 1 vCPU/1 GiB、50 个服务、200 并发请求压测 | 无错误，资源曲线和吞吐结果公开；不预设营销数字 |

## 10. 一手资料索引

- [FRP README](https://github.com/fatedier/frp) 与 [Server Plugin](https://github.com/fatedier/frp/blob/dev/doc/server_plugin.md)
- [Wiredoor 文档](https://docs.wiredoor.net/) 与 [源码](https://github.com/wiredoor/wiredoor)
- [Pangolin 文档](https://docs.pangolin.net/) 与 [源码](https://github.com/fosrl/pangolin)
- [Caddy Automatic HTTPS](https://caddyserver.com/docs/automatic-https)、[API](https://caddyserver.com/docs/api)、[basic_auth](https://caddyserver.com/docs/caddyfile/directives/basic_auth)
- [Apache-2.0 与 GPLv3 兼容说明](https://www.apache.org/licenses/GPL-compatibility.html)
- [SQLite 公共领域声明](https://sqlite.org/copyright.html)
