# OpenClaw Planning

## Overview

Run OpenClaw as a personal AI agent on the homeserver, accessible via WhatsApp, with limited access to Home Assistant (via MCP) and Google Calendar.

---

## Goals

- [ ] WhatsApp as the primary interface
- [ ] Home Assistant integration (limited, scoped entities only)
- [ ] Google Calendar read/write (event creation)
- [ ] Expandable to future use cases

---

## Proposed Infrastructure

| ID  | Type | Name      | IP               |
|-----|------|-----------|------------------|
| 108 | qemu | openclaw  | 192.168.86.16    |

- Standalone QEMU VM (same as HA) — stronger kernel isolation than LXC
- Docker inside the VM for app-level isolation
- Telegram uses outbound polling — no inbound tunnel needed

---

## Architecture

```
Telegram (phone)          Browser dashboard / TUI (laptop)
        │                         │
        ▼                         ▼ (SSH tunnel or Tailscale → localhost:18789)
  OpenClaw Gateway (QEMU VM .20, Docker, port 18789)
        │
        ├──▶ Home Assistant (.11:8123)  [scoped user + long-lived token]
        │
        ├──▶ Google Calendar API  [calendar.events scope only]
        │
        └──▶ Internet (OpenAI API + Telegram polling)

Firewall: outbound internet + .11:8123 only
          ALL other LAN access blocked
```

## Laptop Access

- **Browser dashboard**: `openclaw dashboard` → `http://localhost:18789`
  - Access via SSH tunnel: `ssh -L 18789:localhost:18789 root@192.168.86.16`
  - Or via Tailscale direct to `.20`
- **TUI**: `ssh root@192.168.86.16` then `openclaw tui`
- **macOS menubar app**: native companion app (Beta)

---

## Security Approach

### VM Isolation
- Dedicated QEMU VM on Proxmox (full kernel isolation — stronger than LXC for an internet-facing AI agent with known CVEs)
- Docker inside the VM for app-level isolation
- No SSH keys to other containers stored inside

### Network Firewall (Proxmox level)
- Allow: outbound internet (443/80)
- Allow: `192.168.86.11:8123` (Home Assistant only)
- Block: all other LAN (`192.168.86.0/24`)

### WhatsApp Gateway
- `allowedNumbers` restricted to your number only
- Prevents any other contact from triggering tool calls

### Home Assistant
- Dedicated HA user: `openclaw` (non-admin)
- Long-lived access token for that user only
- Expose only approved entity domains (e.g. lights, climate, sensors)
- Use the OpenClawHomeAssistant MCP add-on

### Google Calendar
- Dedicated Google Cloud project (not shared with other services)
- OAuth scope: `calendar.events` only (not full `calendar`)
- Tokens stored in `~/.openclaw` inside the container

### OpenClaw Config Lockdown
- `terminal: false` — disable shell access
- `filesystem` mount: dedicated workspace dir only (not `/`)
- No community ClawHub skills (review source before installing any)

---

## Open Questions

- [x] LLM backend: **OpenAI (GPT-4o)** — upgrade path to Claude Sonnet 4.6 once stable
- [x] Messaging: **Telegram** (via BotFather) for initial setup and testing
  - Future: migrate to WhatsApp Web bridge once stable (family is on WhatsApp)
  - WhatsApp Business API ruled out (no business account)
- [x] HA entities: expose all (personal use, Telegram locked to your user ID)
- [x] External access: **Cloudflare Tunnel** route for webhook URL (Telegram uses outbound polling by default — tunnel may not be needed at all)
- [x] Backup: not needed for test phase — add to Future Improvements when stable

---

## Implementation Steps

- [x] Create QEMU VM (ID 108, IP 192.168.86.16) on Proxmox — Ubuntu 24.04.3
- [x] Install Docker (v29.3.0)
- [x] Install OpenClaw 2026.3.13 via official install script
- [x] Run `openclaw onboard` — gateway daemon installed, gpt-4o selected
- [x] Configure Telegram channel — bot active, restricted to Paul's user ID
- [x] Document in `home-server-docs.md`
- [ ] Set Proxmox firewall rules on the VM
- [ ] Create scoped HA user + install OpenClawHomeAssistant MCP add-on
- [ ] Configure Google Calendar OAuth (dedicated GCP project)
- [ ] Migrate to WhatsApp Web bridge (future — family on WhatsApp)

---

## References

- OpenClaw docs: https://docs.openclaw.ai
- HA add-on: https://github.com/techartdev/OpenClawHomeAssistant
- Security guide: https://docs.openclaw.ai/gateway/security
