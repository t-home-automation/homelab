# Homepage

Dashboard ([gethomepage.dev](https://gethomepage.dev)) — status: **runbook written, not yet deployed.**

Planned setup: local-only exposure, DNS override → `home.alarconrivero.dev` → LAN IP (matches the "local-only for admin/sensitive services" pattern used elsewhere in this homelab).

## TODOs

- **(later)** confirm qBittorrent WebUI port (now settled: `8090`, per [`servarr/README.md`](../servarr/README.md)) and LazyLibrarian port, then execute the deployment runbook

> 📝 **Suggestion(s) from Claude**
> - Use Homepage's built-in **service widgets** (native integrations for Jellyfin, qBittorrent, Sonarr/Radarr, etc. showing live stats — active downloads, library counts, currently playing) instead of just static links. Docs: [gethomepage.dev/configs/services](https://gethomepage.dev/configs/services/)
> - Use the **Docker integration** — auto-discovers running containers via the Docker socket, shows status/health directly, reduces manual config drift as more services get added. Docs: [gethomepage.dev/configs/docker](https://gethomepage.dev/configs/docker/)
> - A **bookmarks/quick-links section** for non-Docker things too (pfSense UI, Proxmox UI, Hetzner console, Infomaniak panel) — these came up repeatedly as "where do I even log into that" during doc-writing, so Homepage could double as a central login hub.
> - A **Gluetun/VPN status widget** — shows connection status directly, useful given how much attention port-forwarding health has gotten in this build. Docs: [gethomepage.dev/configs/services/gluetun](https://gethomepage.dev/configs/services/gluetun/)
