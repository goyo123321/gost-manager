# GOST 一键管理脚本

[![License](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)
[![GOST](https://img.shields.io/badge/GOST-v2%20%7C%20v3-blue)](https://github.com/ginuerzh/gost)

> 一个跨平台的 GOST 代理一键安装、配置、管理脚本，支持 Linux / FreeBSD / macOS，自动检测系统架构并下载匹配版本。内置安全加固：敏感文件 600 权限、密码 URL 编码、下载 SHA256 尽力校验、启动命令安全序列化。

## ✨ 特性

- **一键安装**：自动检测系统（Linux、FreeBSD、Darwin）和 CPU 架构（amd64、arm64、armv7、386），从 GitHub 下载匹配的 GOST 二进制文件。
- **双版本支持**：
  - **GOST v2**：智能识别版本号，≥2.12.0 使用 `tar.gz` 新格式，≤2.11.5 使用 `gz` 旧格式（自动处理 armv8 等命名差异）。
  - **GOST v3**：仅显示稳定版（自动过滤 `nightly`、`rc`、`alpha`、`beta`），默认安装最新稳定版。
- **代理配置向导**：交互式设置端口、协议（HTTP / SOCKS5 / 自适应 / Shadowsocks / WebSocket / 链式代理 / DNS 代理 / 自定义命令）。
- **后台运行**：使用 `nohup` 将 GOST 放入后台，日志输出到 `~/GOST/gost.log`。
- **开机自启**：通过 `crontab @reboot` 实现开机自启，并内置进程保活（每 5 分钟检查一次，自动重启）。
- **状态查看**：显示已安装版本、运行状态、进程 PID。
- **一键卸载**：停止进程、删除文件、清除 crontab 任务。
- **在线更新**：脚本自身支持从远程仓库更新，更新前校验脚本头部与关键函数。
- **安全加固**：
  - `umask 077`，`start_cmd.txt` / `sub.txt` / `gost.log` / `gost.pid` 权限 600，`keepalive.sh` 权限 700。
  - 用户名/密码写入 GOST URL 前自动 URL 编码，避免 `?`、`&`、`#`、空格等破坏 URL 解析。
  - 下载二进制时尽力进行 SHA256 校验（校验文件不可用或无匹配条目时仅警告，不阻断安装）。
  - 启动命令使用 `printf '%q'` 序列化并以数组执行，避免 word splitting；保活脚本通过 `eval` 还原。
  - 脚本自更新前校验 shebang、关键函数和当前文件可写性，异常内容直接放弃更新。

### GOST-V2[GitHub仓库](https://github.com/ginuerzh/gost)
### [GOST-v2文档](https://v2.gost.run/getting-started/)
### GOST-V3[GitHub仓库](https://github.com/go-gost/gost)
### [GOST-v3文档](https://gost.run/getting-started/quick-start/)

## 🚀 快速开始

### 一键安装

```bash
curl -fsSL https://raw.githubusercontent.com/goyo123321/gost-manager/refs/heads/main/gost-manager.sh -o ~/gost-manager.sh && chmod +x ~/gost-manager.sh && bash ~/gost-manager.sh
```

或者使用 wget：

```bash
wget -q https://raw.githubusercontent.com/goyo123321/gost-manager/refs/heads/main/gost-manager.sh -O ~/gost-manager.sh && chmod +x ~/gost-manager.sh && bash ~/gost-manager.sh
```

运行过的快速命令：

```bash
~/gost-manager.sh
```

1. 运行脚本后，显示主菜单。
2. 选择 `1) 安装 GOST` → 选择版本（v2 或 v3）→ 选择具体版本号（直接回车默认安装第一个版本）。
3. 安装完成后询问是否配置代理（直接回车默认为“是”）。
4. 按提示输入：
   - 端口：例如 `1080`
   - 认证：询问是否需要认证，默认开启（账号 `admin` / 密码 `123456`），也可选择无认证
   - 协议：推荐 `3) 自适应 (HTTP/SOCKS5)`
   - 自定义 DNS（可选，支持 UDP/TCP/DoT/DoH 格式）
5. 询问是否开启开机自启（输入 `y` 开启，直接回车默认不开启）。
6. 配置完成，代理即启动，并显示代理链接；节点信息同步保存到 `~/sub.txt`（权限 600）。

> **仓库地址一致性提醒**：脚本内 `update_script` 的下载地址必须与 README 中的安装命令一致。当前统一使用 `goyo123321/gost-manager`。若你 fork 或迁移仓库，请同时修改脚本内 `update_script` 的 `url` 变量，否则“5) 更新脚本”会拉取到错误内容。

## 📖 使用说明

### 主菜单选项

| 选项 | 功能 |
|------|------|
| 1 | 安装 GOST（可选择 v2 或 v3，并从列表中挑选具体版本） |
| 2 | 配置代理（重新设置代理类型、端口、认证、DNS 等） |
| 3 | 查看当前状态（已安装版本、进程 PID、运行状态） |
| 4 | 卸载 GOST（停止进程、删除目录、清除自启任务） |
| 5 | 更新脚本本身（从 GitHub 拉取最新版本，校验后覆盖） |
| 6 | 查看节点信息（显示保存的代理链接） |
| 7 | 停止 GOST 进程 |
| 8 | 查看运行日志（支持实时跟踪） |
| 0 | 退出脚本 |

### 代理协议说明

| 协议 | 说明 | 示例 URL |
|------|------|----------|
| 1) HTTP | 标准 HTTP 代理 | `http://admin:123456@192.168.1.1:8080` |
| 2) SOCKS5 | 标准 SOCKS5 代理 | `socks5://admin:123456@192.168.1.1:1080` |
| 3) 自适应 | 同一端口同时支持 HTTP 和 SOCKS5（推荐） | `http://admin:123456@192.168.1.1:8888` 或 `socks5://...` |
| 4) Shadowsocks | 基于 shadowsocks 协议，支持 AEAD 加密 | `ss://aes-256-gcm:password@192.168.1.1:8388#GOST-SS` |
| 5) WebSocket | 纯 WS 隧道，支持自定义路径 | `ws://192.168.1.1:8080/ws` |
| 5) v3 组合 | HTTP over WS | `http+ws://192.168.1.1:8080/ws` |
| 5) v3 组合 | SOCKS5 over WS | `socks5+ws://admin:123456@192.168.1.1:8080/ws` |
| 5) v3 组合 | SS over WS | `ss+ws://aes-256-gcm:123456@192.168.1.1:8080/ws` |
| 6) 链式代理 | 本地 HTTP/SOCKS5 → 远程 WS/WSS | `http://admin:123456@:3128 -> socks5+ws://node:443?path=/ws` |
| 7) 自定义命令 | 直接输入完整 GOST 命令行 | `gost -L http://:8080?dns=8.8.8.8` |
| 8) DNS 代理 | DoH 本地解析，或转发给远程隧道解析 | `dns://127.0.0.1:10053?dns=https://doh.pub/dns-query&cache=60s` |

> 上表中的 `+ws` 组合 URL 为说明性写法，客户端实际连接时需支持对应传输层，并正确填写 `path`。
> **SS over WS** 生成的 `ss://` Base64 链接不包含 WS 传输参数，客户端需手动配置 `path`（脚本会在输出中提示）。

### 开机自启机制

- 在 `~/GOST/` 目录下生成 `keepalive.sh`（权限 700）。
- 通过 `crontab` 添加（路径带引号，兼容含空格的工作目录）：
  - `@reboot`：系统启动时运行保活脚本。
  - `*/5 * * * *`：每 5 分钟检查一次进程，若未运行则自动启动。
- 保活脚本读取 `start_cmd.txt`（权限 600，内容为 `printf '%q'` 序列化后的命令），重启后自动恢复最后一次启动命令。

## 🔐 安全说明

| 项目 | 行为 |
|------|------|
| 文件权限 | 脚本启动即 `umask 077`；关键文件写入后 `chmod 600`，保活脚本 `chmod 700`。 |
| 密码编码 | 用户名/密码写入 URL 前统一 `urlencode`，`%XX` 形式；输入仍禁止 `:@/` 以避免歧义。 |
| 下载校验 | v3 尝试 `gost_<ver>_checksums.txt`，v2 尝试 `checksums.txt`；无 `sha256sum`、无法获取校验文件或无匹配条目时仅警告并继续。校验值不匹配则中止安装并删除文件。 |
| 命令执行 | 启动统一走 `start_gost`，命令以数组传入，`nohup "${cmd[@]}"` 执行；落盘使用 `printf '%q '`。 |
| 自更新 | 先校验 shebang 与 `show_menu` 关键字，再检查 `$0` 可写，最后 `cat > "$0"`。 |
| 已知限制 | GOST 通过命令行参数接收密码，**同机用户仍可通过 `ps` 看到进程参数**。建议以低权限用户运行，避免多人共享账号；脚本未实现配置文件方式启动。 |

## 🛠️ 手动管理

### 手动启动代理

```bash
cd "$HOME/GOST"
nohup ./gost -L "admin:123456@:1080" > gost.log 2>&1 &
```

> 密码含 `?`、`&`、`#`、空格等字符时，请先 URL 编码，或改用脚本配置以避免 URL 解析错误。

### 手动停止代理

```bash
pkill -f "$HOME/GOST/gost"
```

> 不要写成 `pkill -f "~/GOST/gost"`，双引号内的 `~` 不会展开为家目录。

### 查看日志

```bash
tail -f "$HOME/GOST/gost.log"
```

### 查看运行状态

```bash
ps aux | grep '[g]ost'
```

## 🌍 支持的系统与环境

| 系统 | 版本 | 架构 |
|------|------|------|
| Linux (glibc) | 任意 | amd64, arm64, armv7, 386 |
| Linux (musl, 如 Alpine) | 任意 | amd64, arm64, armv7, 386 * |
| FreeBSD | 12+ | amd64, 386 |
| macOS (Darwin) | 10.15+ | amd64, arm64 |
| Windows (WSL/原生) | 理论支持（未测试） | amd64, 386 |

注：Alpine Linux 使用 musl libc，而部分 GOST v2.12.0+ 官方构建可能依赖 glibc。若在 Alpine 上运行 v2.12.0 报动态链接错误，可安装 `gcompat`（`apk add gcompat`），或改用 v2.11.5 等通常为静态链接的版本。脚本已内置多 URL 备选，但不保证所有版本在所有 libc 下均可运行。

## ❓ 常见问题

### Q1: 安装时提示“下载失败”怎么办？

- 检查网络是否能够访问 GitHub（`raw.githubusercontent.com` 和 `api.github.com`）。
- 如果身处中国大陆，可尝试更换网络或使用代理。
- 脚本内置了多个备选 URL，如果全部失败，可手动下载二进制文件放入 `~/GOST/` 目录并命名为 `gost`，然后运行脚本的配置代理步骤。

### Q2: 代理启动失败，日志显示“Exec format error”？

- 表示下载的二进制与系统架构不匹配。请运行 `uname -m` 查看架构，并确保脚本正确识别（安装时通常会显示架构信息）。如果不匹配，请手动下载对应架构的版本。

### Q3: 开机自启不生效？

- 确认 crontab 是否支持 `@reboot`（Serv00 等部分环境可能限制）。如果不支持，可改用 `~/.profile` 或 `~/.bashrc` 中添加启动命令。
- 检查 `~/GOST/keepalive.sh` 是否有执行权限（应为 700）。
- 查看 crontab 日志：`grep CRON /var/log/syslog`（需要 root 权限）。

### Q4: 如何卸载脚本和所有文件？

- 在主菜单中选择 `4) 卸载 GOST`，会删除 `~/GOST/` 目录并清除 crontab 任务。
- 如需删除脚本本身，直接删除 `~/gost-manager.sh` 即可。

### Q5: 脚本更新后如何使用新功能？

- 选择主菜单 `5) 更新脚本`，会自动下载最新版本，校验通过后覆盖当前脚本并退出。重新运行脚本即可使用新功能。
- 若提示“下载内容异常”或“当前脚本路径不可写”，说明拉取到的是错误页面或文件不可写；请重试，或按提示手动 `curl` 覆盖。

### Q6: SHA256 校验失败怎么办？

- 脚本会删除已下载文件并中止安装，不会执行未通过校验的二进制。
- 常见原因：网络代理篡改、下载不完整、校验文件与资产不匹配。请更换网络后重试。
- 若上游未提供校验文件或格式不匹配，脚本会显示“跳过校验”并继续，这属于预期行为。

### Q7: 密码含特殊字符会怎样？

- 输入时禁止 `:@/`，其余字符（如 `?`、`&`、`#`、空格、中文）会由脚本自动 URL 编码后写入 GOST 参数。
- 显示的代理链接中密码可能呈现为 `%XX` 形式，这是正常的；客户端一般会自动解码。
- 手动启动时请自行编码，或直接使用脚本配置。

### Q8: 从旧版升级后，旧文件权限还是 644？

- 加固版只对“新写入”的文件设置 600。升级后请重新执行一次 `2) 配置代理`，即可重写并收紧权限。
- 也可手动执行：`chmod 600 ~/GOST/start_cmd.txt ~/GOST/gost.log ~/GOST/gost.pid ~/sub.txt 2>/dev/null; chmod 700 ~/GOST/keepalive.sh 2>/dev/null`。

### Q9: 自定义命令中的引号和空格怎么处理？

- 脚本会用 `eval` 将输入解析为数组。含空格的参数请用引号包裹，例如：
  `gost -L "http://user:pass@:8080?dns=https://1.1.1.1/dns-query"`
- 不确定时，优先使用菜单内的结构化配置项。

## 📝 版本历史

| 版本 | 日期 | 更新内容 |
|------|------|----------|
| v3.1 | 2026-10-22 | 安全加固：`umask 077` + 关键文件 600；密码 URL 编码；下载 SHA256 尽力校验；启动命令数组化 + `printf '%q'` 序列化；自更新增加内容与可写性校验；修复 cron 路径含空格、`version_ge` 对 prerelease 报错、v2 版本列表未过滤 prerelease 等问题；统一仓库地址为 `goyo123321/gost-manager` |
| v3.0 | 2026-06-13 | 全面适配 v3 语法；新增链式代理、WebSocket 组合模式、自定义命令；修复多项 bug |
| v2.5 | 2026-06-08 | 增加 v3 稳定版过滤（去除 nightly/rc）；默认配置代理（Y/n）；版本选择支持默认第一个 |
| v2.4 | 2026-06-07 | 修复 v2.11.5 下载失败（增加 armv8 备选 URL）；适配 FreeBSD 系统 |
| v2.3 | 2026-06-06 | 增加脚本自更新功能；优化架构检测 |
| v2.2 | 2026-06-05 | 支持 GOST v3 安装 |
| v2.1 | 2026-06-04 | 新增开机自启及进程保活 |
| v2.0 | 2026-06-03 | 初始发布，支持 v2 全版本 |

## 🤝 贡献

欢迎提交 Issue 或 Pull Request。如果你有更好的建议或发现 Bug，请到 GitHub 仓库反馈。

## 📄 许可证

本项目采用 MIT 许可证。GOST 本身遵循其各自的开源协议。

---

轻松管理 GOST 代理，一键搞定！
