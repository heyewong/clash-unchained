# clash-unchained

[English](README.md) | [中文](README_CN.md)

> A global extension script for Clash Verge Rev that adds residential chain proxy and AI traffic routing — works with **any** subscription, zero per-subscription configuration.

## What It Does

AI providers block datacenter IPs. This tool generates a **global Clash Verge extension script** that:

1. Automatically detects your subscription's main proxy group (no need to configure per subscription)
2. Injects a static residential SOCKS5 node with chain proxy (`dialer-proxy`) bound to the detected group
3. Injects an AI routing group and 75+ AI domain rules
4. Adds Tailscale bypass (rules + DNS) — optional

```
┌─────────────────────────────────────────────────────────────────┐
│                    AI Service Traffic                           │
│  Device → LLM-Providers → Residential-US (via Subscription)   │
│       → Residential SOCKS5 → AI Service                        │
└─────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────┐
│                    Normal Traffic                               │
│  Device → Subscription Proxies (unchanged) → Internet          │
└─────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────┐
│                   Tailscale Traffic                             │
│  Device → DIRECT (bypassed)                                    │
└─────────────────────────────────────────────────────────────────┘
```

## Features

- **One Script for All Subscriptions** — Switch between subscriptions freely, no per-sub config
- **Auto-Detect Main Proxy Group** — No need to know or fill in your subscription group name
- **Interactive Setup Wizard** — Paste your residential proxy string, answer 3 questions, done
- **Auto-Install to Clash Verge** — Detects your Clash Verge installation path (macOS/Linux/Windows)
- **75+ Built-in AI Domains** — OpenAI, Claude, Gemini, Copilot, Perplexity, HuggingFace, Poe, and more
- **Tailscale Bypass** — Full three-layer defense: bypass list + DIRECT rules + DNS (`direct-nameserver-follow-policy`)
- **Idempotent** — Safe to run multiple times, won't duplicate injections
- **100% Local** — All processing on your machine

## Quick Start

### 1. Download

| Platform | File |
|----------|------|
| macOS (Apple Silicon) | `clash-unchained-darwin-arm64` |
| macOS (Intel) | `clash-unchained-darwin-amd64` |
| Linux | `clash-unchained-linux-amd64` |
| Windows | `clash-unchained-windows-amd64.exe` |

```bash
chmod +x clash-unchained-*
```

### 2. Run the Setup Wizard

```bash
./clash-unchained
```

The wizard asks you for:
1. **Residential proxy credentials** — Paste the connection string from your provider
2. **Display names** — Node name and AI group name (defaults are fine)
3. **Tailscale** — Enable/disable Tailscale bypass
4. **Output file** — Save path

After confirmation, it generates the global script and asks if you want to **auto-install** it to Clash Verge Rev.

> **Re-run the wizard anytime**: `./clash-unchained -r`

### 3. Activate in Clash Verge

- **If auto-install succeeded**: Go to Clash Verge → **Profiles** → click **Update All Subscriptions**
- **If manual**: Open Clash Verge → **Settings** → **Extensions** → **Global Extension Script** → paste the generated content → save → go to **Profiles** → **Update All Subscriptions**

That's it. Switch between any of your subscriptions freely — the chain proxy works with all of them.

### 4. Verify

```bash
# Your subscription node's IP (baseline)
curl -x http://127.0.0.1:7897 https://api.ipify.org

# Check the running proxy chains
curl -s --unix-socket /tmp/verge/verge-mihomo.sock http://localhost/proxies/LLM-Providers
```

AI domains (e.g., `openai.com`, `anthropic.com`) will show `Chains: Residential-US → LLM-Providers` in Clash Verge logs.

## Advanced: config.yaml

Power users can edit `config.yaml` directly then regenerate:

```bash
./clash-unchained -o clash-global-script.js
```

### `nodes[]` — Proxy Nodes

| Field | Description | Required |
|-------|-------------|----------|
| `name` | Node label shown in Clash UI | Yes |
| `type` | `socks5` (default) | No |
| `server` | Residential proxy server address | Yes |
| `port` | Proxy port | Yes |
| `username` | SOCKS5 username | Yes |
| `password` | SOCKS5 password | Yes |

> `dialer-proxy` is auto-detected at runtime. No need to configure it.

### `proxy_groups[]` — Proxy Groups

| Field | Description | Required |
|-------|-------------|----------|
| `name` | Group label | Yes |
| `type` | `select` | Yes |
| `proxies` | Node names in this group | Yes |

### `ai_domains` — AI Domain Routing

| Field | Description | Default |
|-------|-------------|---------|
| `proxy_group` | Group to route AI traffic through | Required |
| `use_builtin` | Use built-in 75+ domain list | `true` |
| `custom` | Extra domains | - |

### `tailscale` — Tailscale Bypass

| Field | Description | Default |
|-------|-------------|---------|
| `enable` | Enable Tailscale bypass with full DNS config | `false` |

## How It Works

The tool generates a **global extension script** (`Script.js`) that Clash Verge applies to every subscription. Unlike per-subscription scripts, the global script:

1. **Auto-detects** the subscription's main `select` group by scanning `proxy-groups`, skipping AI groups
2. **Injects** the residential SOCKS5 node with `dialer-proxy` bound to the detected group
3. **Prepends** AI domain rules (Tailscale rules above AI rules, if enabled)

This means **one generated script works for all your subscriptions**. Add a new subscription URL in Clash Verge — it gets chain proxy support automatically.

## Changelog (v0.2.0)

- **Breaking**: Changed from per-subscription script to global Script.js
- **Breaking**: Removed `dialer_proxy` config field (auto-detected)
- **Breaking**: Removed `tailscale_bypass` proxy group flag (replaced by `tailscale.enable`)
- **New**: Auto-detect subscription main proxy group at runtime
- **New**: Auto-install to Clash Verge Rev (macOS/Linux/Windows)
- **New**: Idempotency protection (safe to run multiple times)
- **Fix**: `nameserver-policy` key now uses `+.ts.net` (matches subdomains)
- **Fix**: Added `direct-nameserver-follow-policy: true` (critical for Tailscale DNS)
- **Fix**: `fake-ip-filter` includes both `*.ts.net` and `+.ts.net`

## License

MIT
