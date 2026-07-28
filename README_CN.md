# clash-unchained

[English](README.md) | [中文](README_CN.md)

> 生成 Clash Verge Rev 全局扩展脚本，自动为所有订阅增加住宅链式代理与 AI 流量分流。

## 它做什么

大模型时代，AI 服务商屏蔽数据中心 IP。本工具生成一段**全局扩展脚本**，一次配置，所有订阅自动适配：

1. **自动检测**当前订阅的主代理组，无需手动填写组名
2. **注入**住宅静态 SOCKS5 节点，`dialer-proxy` 绑定到检测到的主组
3. **注入** AI 路由组和 75+ AI 域名规则
4. **可选** Tailscale 绕过（三层防线：bypass 规则 + DIRECT + DNS）

```
┌─────────────────────────────────────────────────────────────────┐
│                       AI 服务流量                               │
│  设备 → LLM-Providers → Residential-US（经订阅节点）→ AI 服务  │
└─────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────┐
│                       普通流量                                  │
│  设备 → 订阅节点（不变）→ 互联网                                │
└─────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────┐
│                     Tailscale 流量                              │
│  设备 → DIRECT（直连）                                          │
└─────────────────────────────────────────────────────────────────┘
```

## 特性

- **一次生成，所有订阅通用** — 切订阅无需重新配置
- **自动检测主代理组** — 不再需要填写"🚀节点选择"或"蓝莓桥"
- **交互式配置向导** — 粘贴住宅代理连接串，回答 3 个问题即可
- **自动安装到 Clash Verge** — 自动检测安装路径（macOS/Linux/Windows）
- **75+ 内置 AI 域名** — OpenAI、Claude、Gemini、Copilot 等
- **Tailscale 三层防线** — bypass 列表 + DIRECT 规则 + DNS 配置
- **幂等保护** — 多次运行不会重复注入
- **完全本地** — 配置信息不会外泄

## 快速开始

### 1. 下载

| 平台 | 文件名 |
|------|--------|
| macOS（Apple Silicon） | `clash-unchained-darwin-arm64` |
| macOS（Intel） | `clash-unchained-darwin-amd64` |
| Linux | `clash-unchained-linux-amd64` |
| Windows | `clash-unchained-windows-amd64.exe` |

```bash
chmod +x clash-unchained-*
```

### 2. 运行配置向导

```bash
./clash-unchained
```

向导问答：
1. **住宅代理信息** — 粘贴供应商提供的连接串（host:port:user:pass）
2. **显示名称** — 节点名和 AI 组名（默认即可）
3. **Tailscale** — 是否启用绕过
4. **输出文件** — 保存路径

确认后生成全局脚本，并询问是否**自动安装**到 Clash Verge Rev。

> **随时重新配置**：`./clash-unchained -r`

### 3. 在 Clash Verge 中激活

- **如果自动安装成功**：打开 Clash Verge → **订阅** → 点击**更新所有订阅**
- **如果手动安装**：设置 → 扩展 → 全局扩展脚本 → 粘贴生成的内容 → 保存 → 订阅 → 更新所有订阅

此后任意切订阅，链式代理都自动生效。

### 4. 验证

```bash
# 订阅节点的 IP（基准值）
curl -x http://127.0.0.1:7897 https://api.ipify.org

# 查看运行时代理组状态
curl -s --unix-socket /tmp/verge/verge-mihomo.sock http://localhost/proxies/LLM-Providers
```

AI 域名（如 `openai.com`）在 Clash Verge 日志中会显示 `Chains: Residential-US → LLM-Providers`。

## 高级配置

编辑 `config.yaml` 后重新生成：

```bash
./clash-unchained -o clash-global-script.js
```

### `nodes[]` — 住宅节点

| 字段 | 说明 | 是否必填 |
|------|------|----------|
| `name` | 节点名称，显示在 Clash UI 中 | 必填 |
| `type` | `socks5`（默认） | 否 |
| `server` | 住宅代理服务器地址 | 必填 |
| `port` | 代理端口 | 必填 |
| `username` | SOCKS5 用户名 | 必填 |
| `password` | SOCKS5 密码 | 必填 |

> `dialer-proxy` 自动检测，无需配置。

### `proxy_groups[]` — 代理组

| 字段 | 说明 | 是否必填 |
|------|------|----------|
| `name` | 组名 | 必填 |
| `type` | `select` | 必填 |
| `proxies` | 组内节点名列表 | 必填 |

### `ai_domains` — AI 域名路由

| 字段 | 说明 | 默认值 |
|------|------|--------|
| `proxy_group` | AI 流量路由到哪个组 | 必填 |
| `use_builtin` | 使用内置 75+ AI 域名列表 | `true` |
| `custom` | 额外自定义域名 | - |

### `tailscale` — Tailscale 绕过

| 字段 | 说明 | 默认值 |
|------|------|--------|
| `enable` | 启用 Tailscale 绕过（含完整 DNS 配置） | `false` |

## 工作原理

工具生成一段**全局扩展脚本**（`profiles/Script.js`），Clash Verge 在每次刷新订阅时自动执行：

1. **自动检测** — 扫描 `proxy-groups`，跳过 AI 组名，取第一个 `select` 组作为链式出口
2. **注入节点** — 住宅 SOCKS5 节点，`dialer-proxy` 绑定到检测出的主组
3. **注入规则** — AI 域名规则（启用 Tailscale 时在最前面插入绕过规则）

**一个脚本通用所有订阅。** 新增订阅 URL 后自动获得链式代理支持。

## 更新记录 (v0.2.0)

- **Breaking**：从 per-subscription 脚本改为全局 Script.js
- **Breaking**：移除 `dialer_proxy` 配置字段（自动检测）
- **Breaking**：移除 `tailscale_bypass` 代理组标记（改为 `tailscale.enable`）
- **新增**：运行时自动检测订阅主代理组
- **新增**：自动安装到 Clash Verge Rev（macOS/Linux/Windows）
- **新增**：幂等保护（多次运行不重复注入）
- **修复**：`nameserver-policy` key 改为 `+.ts.net`（匹配子域名）
- **修复**：补充 `direct-nameserver-follow-policy: true`（Tailscale DNS 必备）
- **修复**：`fake-ip-filter` 同时包含 `*.ts.net` 和 `+.ts.net`

## License

MIT
