# 03 · 常见日志报错排错速查表

> 核心排错心法：**不要盲目重装服务器！** 打开客户端「日志」看报错关键词，30 秒即可定位问题层级。

| 客户端日志 / 现象 | 问题所在层级 | 常见原因与一步排查法 |
| :--- | :--- | :--- |
| `dial tcp <IP>:443: i/o timeout` | 网络 / 防火墙层（压根没连上 443） | 1. 检查客户端 `server` 是否填错了真实 IP（例如误留示例 IP）<br>2. VPS 运行 `ufw status` 确认 `443/tcp` 已 `ALLOW`<br>3. 用 `tcp.ping.pe` 检测 `<IP>:22` 与 `<IP>:443`，区分端口未开还是 IP 被阻断 |
| `REALITY authentication failed` | Reality TLS 握手层（443 已通，暗号未过） | 1. **2026 必查**：是否使用了证书超 8KB 的 `www.microsoft.com`？换成 `www.bing.com:443`<br>2. **2026 必查**：`3x-ui` 的 `最小客户端版本` 是否留空？必须填 `0.0.0`<br>3. 检查 `public-key` 是否误填成了私钥，或 `servername` 误带了 `:443`<br>4. 若服务器使用 Xray `v26.9.8+`，客户端 YAML 需加 `support-x25519mlkem768: true` |
| `invalid REALITY public key`（保存 YAML 即报错） | 客户端配置格式层 | 公钥应为 43 位无空格字符；检查是否漏字、多粘了换行或尖括号 `< >` 未删净 |
| 电脑通了，iPhone 小火箭测速有数字但网页转圈 | Xray 版本与 iOS 客户端兼容层 | 小火箭测速默认是 TCP Ping；若 Xray 为 `v26.9.8+` 强制抗量子握手，在 `3x-ui` 仪表盘将 Xray 切换至 **`v26.7.28`** 即可双端直通 |
| 浏览器通了，但 IDE / PowerShell / Git 走不了代理 | 本地路由模式层 | 在 Clash Verge「设置」中安装「服务模式」并开启 **TUN 模式** |
| PowerShell 报错 `标记“&&”不是有效语句分隔符` | 本地终端误操作 | 此时仍在本地 Windows（`PS C:\>`），请先执行 `ssh root@<IP>` 看到 `root@...:~#` 后再粘贴 Linux 命令 |
