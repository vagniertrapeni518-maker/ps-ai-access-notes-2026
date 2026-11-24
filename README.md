# 2026 零基础海外 Linux VPS 与多端兼容避坑实战手册

> 一份由零基础学习者与 AI 结对实战、逐行真机验证整理而成的开源笔记。
> 专注解决 **2026 年新版 `3x-ui v3.11+`、`Xray v26.x` 与桌面端（Clash Verge）/ iOS 端兼容性暗坑**，附开箱即用的脱敏本地配置模板与日志排错速查表。
> 版本信息基于 2026-11 实测，可能随上游更新失效。

## ⚠️ 合规与安全声明

1. 本仓库仅用于 **Linux 服务器运维学习、网络安全基线实践与个人合法网络环境研究**，请严格遵守所在地区法律法规。
2. 仓库内所有 IP、端口、UUID、密钥均为占位符，**切勿将个人真实凭证提交到任何公开仓库或粘贴到陌生第三方在线转换网站**。

---

## 🔥 为什么 2026 年看旧教程经常连不上？（实测 3 大新坑速查）

很多 2024–2025 年的老教程已经跑不通最新版面板与内核，本次实测踩通的 3 个高频新坑汇总如下：

| 现象 / 日志报错 | 2026 真实原因 | 一分钟解法 |
| :--- | :--- | :--- |
| `REALITY authentication failed` | 常见伪装目标 `www.microsoft.com` 证书超过 8192 字节解析上限 | 更换为证书合规的大厂 TLS 1.3 站点（如 `www.bing.com:443` / `www.cloudflare.com:443`） |
| `REALITY authentication failed`（仅 Clash Verge / Mihomo 报错） | Xray `v26.7.11+` 留空 `minClientVer` 时会拒收 Mihomo 内核；`v26.9.8+` 强制校验抗量子密钥共享 | 1. 服务端 `最小客户端版本` 显式填 `0.0.0`<br>2. 客户端 `reality-opts` 增加 `support-x25519mlkem768: true` |
| 电脑通了，iOS 端测速正常但网页一直转圈 | Xray `v26.9.8+` 强制 `X25519MLKEM768`，部分 iOS 客户端直连扫码尚未默认携带该扩展 | 在 `3x-ui` 仪表盘将 Xray 内核切换至 **`v26.7.28`**（`< v26.9.8` 稳定版），实现电脑 + 手机双端兼容 |

---

## 🛠️ 推荐技术栈（入门低成本实测组合）

- **VPS 机型**：海外低价入门 KVM VPS（1C1G 促销款年付约 $15–22，优选美西机房）
- **操作系统**：`Ubuntu 22.04 64-bit`（对 1GB 内存模板友好）
- **基础优化**：开启内核 **BBR** + 校验 `timedatectl` 时间同步 + `ufw` 最小端口放行（先放 22 端口防锁死）
- **管理面板**：`MHSanaei/3x-ui`（v3.11+ 独立客户端管理，内置 Fail2ban 与 Let's Encrypt IP HTTPS 证书）
- **桌面客户端**：Clash Verge Rev（采用**本地 YAML 模板直导**，拒绝第三方在线订阅转换）
- **移动端**：iOS `Shadowrocket` / Android `v2rayNG`

---

## 📂 仓库目录导航

| 文件 | 说明 |
| :--- | :--- |
| [docs/01-vps-selection.md](docs/01-vps-selection.md) | VPS 选型对比、预算分层、国内外云隔离原则与支付防盗刷建议 |
| [docs/02-step-by-step.md](docs/02-step-by-step.md) | 零基础手把手分步手册（SSH → BBR → 3x-ui v3.11 → ufw → Clash Verge / iOS） |
| [docs/03-troubleshooting.md](docs/03-troubleshooting.md) | 按客户端日志报错反查表（`i/o timeout`、`REALITY authentication failed`、手机转圈等） |
| [templates/clash-verge-local.yaml](templates/clash-verge-local.yaml) | 100% 脱敏的 Clash Verge Rev 本地 YAML 配置模板（含 AI 分流 + 国内直连） |

---

## 🤝 欢迎交流

如果这份踩坑记录帮你省了 2 小时，欢迎点个 **Star ⭐** 收藏备用；遇到 2026 新版本报错也欢迎携带**脱敏日志**提交 Issue 一起完善。
