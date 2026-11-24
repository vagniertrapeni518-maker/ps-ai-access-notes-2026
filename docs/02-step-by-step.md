# 02 · 零基础分步部署清单（Ubuntu 22.04 + 3x-ui v3.11 + Clash Verge / iOS）

> 本文命令已在 `Ubuntu 22.04 64-bit` + `3x-ui v3.11.0` + `Clash Verge Rev` + `iOS Shadowrocket` 实机验证通过。
> 所有 `<尖括号>` 均为占位符，执行时请替换为自己的真实值。

---

## 第一步：通过 SSH 登录服务器并初始化

在 Windows 打开自带 **PowerShell / 终端**（`Win + X` → 终端），执行：

```powershell
ssh root@<YOUR_VPS_IP>
```

> **新手提示**：
> - 看到 `PS C:\...>` 说明仍在本地 Windows，看到 `root@...:~#` 才算登进云端 Linux 服务器。
> - Linux 输入密码时**屏幕不会显示任何字符或星号**，粘贴一次直接回车即可。

登录后，执行一键初始化（安装基础工具 + 开启 BBR 加速 + 校验时间同步）：

```bash
apt update && DEBIAN_FRONTEND=noninteractive apt install -y curl wget ufw ca-certificates && printf "net.core.default_qdisc=fq\nnet.ipv4.tcp_congestion_control=bbr\n" > /etc/sysctl.d/99-bbr.conf && sysctl --system && sysctl net.ipv4.tcp_congestion_control && timedatectl | grep -i synchronized
```

确认输出包含：
- `net.ipv4.tcp_congestion_control = bbr`
- `System clock synchronized: yes`

建议顺手运行 `passwd` 修改邮件里的明文初始 root 密码。

---

## 第二步：安装 `3x-ui` 面板并配置最小端口防火墙

运行 `MHSanaei/3x-ui` 官方安装脚本（运行前建议前往其 GitHub 官方仓库核对最新指令）：

```bash
bash <(curl -Ls https://raw.githubusercontent.com/mhsanaei/3x-ui/master/install.sh)
```

安装交互建议：
1. **Database Selection**：输入 `1`（SQLite，最省内存）。
2. **Customize Panel Port settings**：输入 `n`（自动生成冷门随机端口、随机路径与强密码）。
3. **SSL Certificate Setup**：输入 `2`（Let's Encrypt IP 免费 HTTPS 证书，确认 IPv4 输入 `y`，IPv6 回车跳过，ACME 端口默认 `80` 回车）。

安装完成后，记录屏幕输出的 `Access URL`、`port`、`username`、`password` 至本地密码管理器（若忘记密码，可运行 `x-ui` 选 `7` 重置密码再选 `14` 重启面板）。

**放行防火墙（⚠️ 务必最先放行 22 端口防锁死！）**：

```bash
ufw allow 22/tcp && ufw allow 80/tcp && ufw allow 443/tcp && ufw allow <YOUR_PANEL_PORT>/tcp && ufw --force enable && ufw status
```

---

## 第三步：在 `3x-ui v3.11+` 网页后台创建节点

打开 `Access URL` 登录并切换简体中文，按以下顺序操作：

### 1. 兼容 iOS 与桌面端的关键一步：切换 Xray 内核至 `v26.7.28`
- 若面板默认内核为 `v26.9.8+`（如 `v26.10.21`），其强制要求 `X25519MLKEM768` 抗量子握手，会导致部分 iOS 客户端直连扫码无法走流量。
- 在仪表盘左上角点击 **`Xray · 运行中 v26.xx`** → 切换为 **`v26.7.28`**（`< v26.9.8` 稳定版），等待 5–10 秒自动重启。

### 2. 添加入站
进入左侧 **「入站列表 → 添加入站」**：
- **基础配置**：备注填 `VPS-VLESS-Reality`，协议 `vless`，端口 `443`。
- **协议**：保持默认（解密/加密均为 `none`）。
- **传输**：保持 **`RAW`**（新版 TCP）。
- **安全**：选 **`Reality`**：
  - **uTLS**：`chrome`
  - **目标**：`www.bing.com:443`（⚠️ 勿用 `www.microsoft.com:443`，2026 年其证书超 8192 字节会认证失败）
  - **SNI**：`www.bing.com`（按回车形成标签）
  - **最小客户端版本**：**务必填 `0.0.0`**（防止 Xray 拒收 Clash Verge / Mihomo）
  - 点击 **「获取新证书」** 自动生成公钥、私钥与 Short IDs（`mldsa65 Seed` 留空）。
- 点击 **「创建」**。

### 3. 添加客户端并关联入站
进入左侧 **「客户端（小人头图标）→ 添加客户端」**：
- 在 **「凭据」** 页复制自动生成的 `UUID`；
- 切回 **「基本」** 页，在最底部 **「关联入站」** 选中刚创建的入站 → 点击 **「创建」**。

---

## 第四步：多端接入

### Windows：Clash Verge Rev（本地 YAML 直导，拒绝在线转换）
1. 复制本仓库 [templates/clash-verge-local.yaml](../templates/clash-verge-local.yaml)，替换其中 `server`、`uuid`、`public-key`（**注意是公钥不是私钥**）、`short-id` 4 项占位符。
2. 在 Clash Verge 点击 **「订阅 → 新建 → 类型选 Local」** → 右键「编辑文件」粘贴保存并选中启用。
3. 在「代理」页测试延迟变绿；如需命令行工具与 IDE 同步走代理，在「设置」中安装服务模式并开启 **TUN 模式**。

### iOS：Shadowrocket / Android：v2rayNG
1. 在 `3x-ui` 左侧 **「客户端」** 点击二维码图标 → 展开下面带 **`Vless ... (443)`** 的一行（不要扫上面的订阅信息）。
2. 打开手机端代理 App 内的扫一扫功能（勿用系统相机或微信）直接扫码导入，路由模式选「配置」即可使用。
