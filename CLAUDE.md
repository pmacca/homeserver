# Home Server Project

Any changes made to the home server infrastructure (IPs, services, tunnel routes, configs, etc.) must be reflected in `home-server-docs.md`.

## Cloudflare

- API token and zone ID are in `.env`
- Tunnel routes are managed via the **Cloudflare Zero Trust dashboard**, not the local config file on `.12`. Local config edits get overridden by remote config.
- When adding new tunnel routes, add them in the dashboard — it creates DNS CNAME records automatically. Do NOT create DNS CNAMEs via API first (causes conflict).
- The API token only has DNS permissions. Tunnel route changes require the dashboard (or expanding the token scope).
- DNS records can be managed via API (create, delete, list) using the token in `.env`.
- Tunnel ID: `b845f943-d4c8-4a8f-95cc-50fc7dfbe6de`

## SSH Access

- All VMs/containers accessible via `ssh root@<IP>` using local pem key

## Git

- After every file edit, automatically commit the changes with a concise descriptive message. This ensures a full history of all infrastructure documentation changes.
- Do not wait for the user to ask for a commit — commit immediately after each edit.
- Do not bundle multiple unrelated edits into one commit. Each logical change should be its own commit.
