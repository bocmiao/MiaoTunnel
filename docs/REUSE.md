# 复用、许可证与供应链决策

## 总体策略

复用稳定的网络与证书组件，以独立进程集成；MiaoTunnel 只编写身份、配置编排、状态收敛、审计、诊断和安装体验。

## 组件矩阵

| 组件 | 用法 | 许可证 | 决策 | 主要风险与控制 |
| --- | --- | --- | --- | --- |
| [FRP](https://github.com/fatedier/frp) | `frps`/`frpc` 独立进程 | Apache-2.0 | 采用 | 固定已验证版本；插件拒绝越权代理；配置适配层隔离 |
| [Caddy](https://github.com/caddyserver/caddy) | HTTPS 入口与证书 | Apache-2.0 | 采用 | 管理 API 不暴露公网；只生成显式域名；持久化 `/data` |
| [SQLite](https://sqlite.org/) | 单机控制面数据库 | Public Domain | 采用 | 单写者、事务、备份恢复测试；不假装支持集群 |
| Go `net/http`、`html/template`、`embed` | API 与服务端页面 | BSD-3-Clause | 采用 | 先不用 SPA 和大型 Web 框架 |
| `golang.org/x/crypto` | Argon2id 等密码学辅助 | BSD-3-Clause | 按需采用 | 固定版本并跟进 Go 安全公告 |
| [go-sqlite3](https://github.com/mattn/go-sqlite3) | `database/sql` 驱动 | MIT | 候选 | 需要 CGO；只在 Linux 服务端构建，先做交叉编译 spike |
| [pquerna/otp](https://github.com/pquerna/otp) | TOTP | Apache-2.0 | v0.2 候选 | v0.1 不提前引入；公开测试前补齐 |
| Wiredoor | UX、威胁和部署参考 | Apache-2.0 | 不 fork | 只借鉴概念；复制代码时逐文件记录 NOTICE |
| rathole | 备用数据面 | Apache-2.0 | 暂不集成 | 只有 FRP 出现可复现瓶颈才做适配 |
| Pangolin | 产品边界参考 | AGPL-3.0/商业混合 | 不复制代码 | 避免把不同版本代码和素材误纳入项目 |

## 为什么独立进程集成 FRP/Caddy

- 升级、回滚和漏洞响应边界清楚；
- 不依赖不稳定的内部 Go API；
- 可以保留第三方原许可证和 NOTICE；
- 数据面替换成本受适配器限制；
- GPLv3 项目可以分发 Apache-2.0 组件，但仍需保留其许可证与 NOTICE。

[Apache 软件基金会的说明](https://www.apache.org/licenses/GPL-compatibility.html)明确：Apache-2.0 代码可纳入 GPLv3 项目；反方向不成立。这里是工程合规记录，不构成法律意见。

## 分发要求

发布包至少包含：

- MiaoTunnel 的 `LICENSE`（GPLv3）；
- `THIRD_PARTY_NOTICES.md`，列出组件、版本、来源、许可证和是否修改；
- FRP、Caddy 等 Apache-2.0 许可证文本及其上游 NOTICE（若存在）；
- 源码获取方式、构建步骤、对应提交和依赖锁文件；
- 二进制 SHA-256 校验值；稳定版再增加签名。

容器镜像标签不得只用 `latest`。Compose 示例应固定到经验证的小版本或 digest，并由 Renovate/Dependabot 提 PR，不能自动上线。

## 不直接复制的内容

- 竞品名称、logo、截图、文案和品牌素材；
- Pangolin 企业版或许可不明目录；
- 商业产品页面的代码与视觉资产；
- 未记录来源的配置片段；
- 为“兼容未来”提前复制的大型 SDK 或插件系统。

## 供应链基线

- 合并前运行单元测试、`go vet`、govulncheck、镜像扫描和 secret scan；
- 发布由 GitHub Actions 从 tag 构建，禁止开发机手工上传稳定包；
- 生成 SBOM；构建日志记录 Go、FRP、Caddy 版本；
- agent 不静默自更新；管理员明确选择版本并可回滚；
- 依赖升级先经过端到端隧道、证书续期和配置回滚测试。

## 替换数据面的触发条件

满足以下至少一项且有可复现证据，才评估 rathole 或自研：

1. FRP 无法实现每设备和每服务隔离；
2. 严重安全问题不能通过升级、配置或上游修复解决；
3. 目标硬件上的资源或性能基准持续不达标；
4. 协议升级无法保持服务器与 agent 的可控兼容窗口。

“代码看起来更简洁”或“理论上更快”不是替换依据。
