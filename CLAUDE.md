# Home Server Project

Any changes made to the home server infrastructure (IPs, services, tunnel routes, configs, etc.) must be reflected in `home-server-docs.md`.

## Cloudflare

- API token and zone ID are in `.env`
- Tunnel is **locally managed** via `/etc/cloudflared/config.yml` on `.12` — `config.yml` is authoritative. The old dashboard-managed tunnel has been replaced.
- Tunnel name: `home-server`, ID: `6a704994-c922-41b3-8f21-a9cb08218054`
- To add a new route: edit `config.yml`, restart cloudflared, then create the DNS CNAME via `cloudflared tunnel route dns home-server <hostname>` or via the API.
- DNS records can be managed via API (create, delete, list) using the token in `.env`.
- The API token has DNS permissions only.

## SSH Access

- All VMs/containers accessible via `ssh root@<IP>` using local pem key

## Git

- After every file edit, automatically commit the changes with a concise descriptive message. This ensures a full history of all infrastructure documentation changes.
- Do not wait for the user to ask for a commit — commit immediately after each edit.
- Do not bundle multiple unrelated edits into one commit. Each logical change should be its own commit.
