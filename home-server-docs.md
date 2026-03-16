# Home Server Documentation

## Network

  Item          Value
  ------------- ---------------------------------
  Router        `192.168.86.1` (Google Nest Wifi)
  Subnet Mask   `255.255.255.0`
  DHCP Pool     `192.168.86.30 → 192.168.86.254`

### Google Nest Wifi Limitations

-   No 2.4 GHz vs 5 GHz band control (auto-switching only)
-   Static IP reservations done via Google Home app (devices are pinned)

------------------------------------------------------------------------

# Infrastructure

## Proxmox Host

  Item            Value
  --------------- ------------------------------
  Hostname        `proxmox`
  Access URL      `https://192.168.86.10:8006`
  Login Realm     `pam`
  Primary Admin   `root@pam`

Proxmox hosts VMs and LXC containers used for the home server
environment.

### Services on Proxmox Host

**Glances** — system monitoring API used by Homepage dashboard.

  Item          Value
  ------------- -------------------------------------------
  Port          `61208`
  URL           `http://192.168.86.10:61208`
  Service       `glances.service` (systemd)
  Mode          API only (`-w --disable-webui`)
  Bind          `0.0.0.0:61208`

Provides CPU, RAM, and disk metrics to Homepage via REST API.

### VMs and Containers

  ID    Type   Name             IP
  ----- ------ ---------------- ------------------
  102   lxc    pihole           `192.168.86.5`
  107   lxc    nginx            `192.168.86.6`
  100   qemu   home-assistant   `192.168.86.11`
  101   lxc    cloudflared      `192.168.86.12`
  103   lxc    media            `192.168.86.13`
  104   lxc    vaultwarden      `192.168.86.14`
  105   lxc    mqtt             `192.168.86.15`
  108   qemu   openclaw         `192.168.86.16`
  106   lxc    racing           `192.168.86.19`

------------------------------------------------------------------------

# Virtual Machines / Containers

## Home Assistant

  Item              Value
  ----------------- -------------------------------
  IP                `192.168.86.11`
  URL               `http://192.168.86.11:8123`
  External Access   `https://home.mcmanus.net.au`
  Platform          Proxmox QEMU VM
  Integration       Sungrow SH10RS inverter

Cloud access is provided through a **Cloudflare Tunnel**. Platform is
a QEMU VM (not LXC like the other services).

------------------------------------------------------------------------

## Cloudflare Tunnel

  Item          Value
  ------------- -------------------------------------------
  IP            `192.168.86.12`
  Platform      LXC container
  Purpose       Secure public access to internal services
  Config File   `/etc/cloudflared/config.yml`

### Exposed Services

  Hostname                     Service
  ---------------------------- ----------------
  `home.mcmanus.net.au`        Home Assistant
  `jellyfin.mcmanus.net.au`    Jellyfin
  `discover.mcmanus.net.au`    Jellyseerr
  `vault.mcmanus.net.au`       Vaultwarden
  `homepage.mcmanus.net.au`    Homepage dashboard
  `racing.mcmanus.net.au`      Racing app
  `deploy.mcmanus.net.au`      Portainer (racing host)


------------------------------------------------------------------------

# Media Server

Docker host located at:

    192.168.86.13

### Media Stack

  Service       Port   URL
  ------------- ------ ---------------------------
  Jellyfin      8096   http://192.168.86.13:8096
  Jellyseerr    5055   http://192.168.86.13:5055
  Radarr        7878   http://192.168.86.13:7878
  Sonarr        8989   http://192.168.86.13:8989
  Bazarr        6767   http://192.168.86.13:6767
  Prowlarr      9696   http://192.168.86.13:9696
  qBittorrent   8080   http://192.168.86.13:8080
  Portainer     9000   http://192.168.86.13:9000  (admin: `admin`)
  FlareSolverr  8191   http://192.168.86.13:8191


------------------------------------------------------------------------

# Password Management

## Vaultwarden

Self‑hosted Bitwarden-compatible password manager.

  Item         Value
  ------------ -----------------------------
  Host         `192.168.86.14`
  Port         `8000`
  URL          `http://192.168.86.14:8000`
  Public URL   `vault.mcmanus.net.au`

------------------------------------------------------------------------

# MQTT

  Item   Value
  ------ -------------------------
  IP     `192.168.86.15`

------------------------------------------------------------------------

# Racing App

Custom application.

  Item           Value
  -------------- ----------------------------------
  Host           `192.168.86.19`
  App Port       `8000`
  Portainer      `9000`
  Public URL     `racing.mcmanus.net.au`

------------------------------------------------------------------------

## OpenClaw

AI agent VM — personal assistant via Telegram, with Home Assistant and Google Calendar integrations (planned).

  Item        Value
  ----------- ---------------------------------------------------
  IP          `192.168.86.16`
  Platform    QEMU VM (Ubuntu 24.04.3)
  Username    `paul`
  Password    the usual
  SSH         `ssh paul@192.168.86.16`
  Dashboard   `http://192.168.86.16:18789` (via SSH tunnel or Tailscale)
  Version     OpenClaw 2026.3.13

### Setup

-   Installed via official install script (npm-based, Node.js 22)
-   Binary at `/home/paul/.npm-global/bin/openclaw`
-   PATH set in `~/.bashrc`
-   Gateway runs as a system daemon (`openclaw onboard --install-daemon`)
-   Workspace at `~/openclaw`
-   Config at `~/.openclaw`

### LLM

-   Provider: OpenAI
-   Model: `gpt-4o`

### Channels

-   **Telegram**: active — bot connected, restricted to Paul's user ID only

### Laptop Access

    ssh -L 18789:localhost:18789 paul@192.168.86.16
    # then open http://localhost:18789 in browser

Or use Tailscale to reach `192.168.86.16:18789` directly.

### TUI

    ssh paul@192.168.86.16
    openclaw tui

### Firewall

Proxmox VM-level firewall enabled (`/etc/pve/firewall/108.fw`).

  Direction   Policy
  ----------- -------
  Inbound     DROP (default)
  Outbound    ACCEPT (default)

Inbound rules:

  Port    Protocol   Source              Purpose
  ------- ---------- ------------------- ---------
  22      TCP        `192.168.86.0/24`   SSH
  18789   TCP        `192.168.86.0/24`   Dashboard

Outbound rules:

  Destination          Port   Protocol   Action   Purpose
  -------------------- ------ ---------- -------- ----------------
  `192.168.86.5`       53     UDP/TCP    ACCEPT   DNS (Pi-hole)
  `192.168.86.11`      8123   TCP        ACCEPT   Home Assistant
  `192.168.86.0/24`    any    any        DROP     Block rest of LAN
  Internet             any    any        ACCEPT   (default policy)

### Integrations

#### Google Calendar
-   Tool: `gog` CLI v0.12.0 (steipete/gogcli), installed at `/usr/local/bin/gog`
-   Account: `mcmanus.paul@gmail.com`
-   Scope: `calendar` (read/write events)
-   Credentials: `~/.config/gogcli/credentials.json`
-   Keyring: file-based, passphrase set via `GOG_KEYRING_PASSWORD` in `~/.bashrc`
-   Default account set via `GOG_ACCOUNT` in `~/.bashrc`

#### Planned

-   Home Assistant (scoped user + MCP add-on)

See `openclaw-plan.md` for full architecture and security notes.

------------------------------------------------------------------------

# Network Services

## Pi-hole

  Item        Value
  ----------- -----------------------------
  IP          `192.168.86.5`
  Admin URL   `http://192.168.86.5/admin`

Provides:

-   DNS filtering
-   Ad blocking
-   Local DNS resolution for internal services

### Local DNS Entries

Pi-hole resolves internal hostnames to the nginx reverse proxy
(`192.168.86.6`) for LAN access without going through Cloudflare.

  Hostname                         Target
  -------------------------------- ----------------
  `radarr.mcmanus.net.au`          `192.168.86.6`
  `sonarr.mcmanus.net.au`          `192.168.86.6`
  `prowlarr.mcmanus.net.au`        `192.168.86.6`
  `bazarr.mcmanus.net.au`          `192.168.86.6`
  `qbittorrent.mcmanus.net.au`     `192.168.86.6`
  `portainer.mcmanus.net.au`       `192.168.86.6`
  `pihole.mcmanus.net.au`          `192.168.86.5`
  `proxmox.mcmanus.net.au`         `192.168.86.10`

Devices must use Pi-hole as their DNS server for these to resolve.

------------------------------------------------------------------------

# Nginx Reverse Proxy

LXC container providing friendly local URLs for media services.

  Item          Value
  ------------- -----------------------------------
  IP            `192.168.86.6`
  Config Dir    `/etc/nginx/sites-enabled/`

### Proxied Services

  Hostname                         Backend
  -------------------------------- ----------------------------
  `radarr.mcmanus.net.au`          `http://192.168.86.13:7878`
  `sonarr.mcmanus.net.au`          `http://192.168.86.13:8989`
  `prowlarr.mcmanus.net.au`        `http://192.168.86.13:9696`
  `bazarr.mcmanus.net.au`          `http://192.168.86.13:6767`
  `qbittorrent.mcmanus.net.au`     `http://192.168.86.13:8080`
  `jellyfin.mcmanus.net.au`        `http://192.168.86.13:8096`
  `overseerr.mcmanus.net.au`       `http://192.168.86.13:5055`
  `portainer.mcmanus.net.au`       `http://192.168.86.13:9000`


Works in combination with Pi-hole local DNS entries that point these
hostnames to `192.168.86.6`.

------------------------------------------------------------------------

# Dashboard

## Homepage

Container running on the media server.

  Item          Value
  ------------- ----------------------------------
  Host          `192.168.86.13`
  Port          `3001`
  Config Path   `/opt/dashboard/homepage-config`

Provides:

-   service dashboard
-   infrastructure links
-   system metrics
-   monitoring widgets

------------------------------------------------------------------------

# Container Update Monitoring

## What's Up Docker (WUD)

  Item       Value
  ---------- --------------------------------
  Host       `192.168.86.13`
  Port       `3000`
  Function   Detect container image updates

------------------------------------------------------------------------

# Storage / Config Locations

## Dashboard Root

    /opt/dashboard

### Homepage Config

    /opt/dashboard/homepage-config

Contains:

    services.yaml
    widgets.yaml
    proxmox.yaml
    settings.yaml
    docker.yaml
    bookmarks.yaml

------------------------------------------------------------------------

# Notifications

Home Assistant sends notifications to:

    notify.mobile_app_pixel_9_pro

Used for:

-   Radarr download notifications
-   Sonarr download notifications

------------------------------------------------------------------------

# Media Automation Flow

    Jellyseerr
          │
          ▼
       Radarr / Sonarr
          │
          ▼
       Prowlarr
          │
          ▼
     qBittorrent
          │
          ▼
      Media Files
          │
          ▼
       Jellyfin

Subtitles handled by:

    Bazarr

------------------------------------------------------------------------

# Access Architecture

## External (Internet)

    Internet
        │
        ▼
    Cloudflare Tunnel
        │
        ▼
    cloudflared (LXC .12)
        │
        ▼
    Internal services

## Internal (LAN)

    LAN device (DNS via Pi-hole .5)
        │
        ▼
    Pi-hole resolves *.mcmanus.net.au → 192.168.86.6
        │
        ▼
    nginx reverse proxy (LXC .6)
        │
        ▼
    Internal services

## Remote (Tailscale)

    Device with Tailscale client
        │
        ▼
    Tailscale network (tailnet: sunfish-harmonic)
        │
        ▼
    Proxmox subnet router (advertises 192.168.86.0/24)
        │
        ▼
    Internal services (SSH, web UIs, etc.)

------------------------------------------------------------------------

# Tailscale

Tailscale provides remote access to the home network when away from the
LAN (e.g. at work). The Proxmox host acts as a **subnet router**,
advertising the entire `192.168.86.0/24` range so all VMs and containers
are reachable by their LAN IPs.

  Item              Value
  ----------------- -------------------------------------------------
  Tailnet           `sunfish-harmonic`
  Account           `mcmanus.paul@gmail.com` (Free plan)
  Subnet Router     Proxmox (`192.168.86.10`)

## Tailnet Machines

  Machine           Tailscale IP       Role / Notes
  ----------------- ------------------ -----------------------------------
  proxmox           `100.100.153.97`   Subnet router, Tailscale SSH
  paul-mac          `100.80.244.20`    macOS client
  pixel-9-pro       `100.72.12.62`     Android client

## Proxmox Tailscale Config

Installed natively on the Proxmox host:

    curl -fsSL https://tailscale.com/install.sh | sh
    tailscale up --advertise-routes=192.168.86.0/24 --ssh

Features enabled:

-   **Tailscale SSH** — SSH into Proxmox at `100.100.153.97` without keys
-   **Subnet routing** — all LAN IPs reachable via Tailscale
-   **Tailscale Serve** — proxies Proxmox web UI:
    `https://proxmox.sunfish-harmonic.ts.net/` → `https://localhost:8006`

## Remote SSH Access

With the Proxmox subnet route approved in the Tailscale admin console,
all containers are accessible via SSH from any Tailscale-connected
device using their LAN IPs:

    ssh root@192.168.86.10   # Proxmox host
    ssh root@192.168.86.12   # cloudflared
    ssh root@192.168.86.13   # media
    ssh root@192.168.86.14   # vaultwarden
    ssh root@192.168.86.5    # pihole
    ssh root@192.168.86.15   # mqtt
    ssh root@192.168.86.19   # racing
    ssh root@192.168.86.6    # nginx

No need to install Tailscale on individual containers — the subnet
router handles routing.

**Prerequisites:**

1.  Subnet route `192.168.86.0/24` must be **approved** in the Tailscale
    admin console (Machines → proxmox → Edit route settings)
2.  ACL policy must allow traffic to subnet destinations — `"dst": ["*"]`
    in the grants section (using `"autogroup:member"` only allows traffic
    to tailnet devices, not to subnets behind them)
3.  Tailscale client must be connected on the remote device
4.  SSH must be running on the target container (enabled by default on
    all LXC containers)

------------------------------------------------------------------------

# Backups

Proxmox backups are configured through the Proxmox web UI for VMs and
containers.

------------------------------------------------------------------------

# Future Improvements

-   Docker auto-update automation

-   Service health monitoring
-   ~~Proxmox API token integration~~ (done)
-   Migrate Cloudflare Tunnel to fully local config management (recreate tunnel to remove remote/dashboard control)
-   Move HA Zigbee into its own LXC container
