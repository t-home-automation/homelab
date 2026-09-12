# Nextcloud

Status: **live/populated**, running on docker-host. Currently bound to host port `8080` internally (confirmed via `docker port nextcloud` → `0.0.0.0:8080->80/tcp`), which is why qBittorrent's WebUI had to be moved to port `8090` in the Servarr stack to avoid a conflict.

Externally accessible at `nextcloud.alarconrivero.dev` — see [`external-access/README.md`](../../external-access/README.md) for the full request path (Caddy → WireGuard tunnel → pfSense `services_frontend` → `nextcloud_backend`).

## TODOs

- **(later)** review/reorganize this setup — flagged during docs pass, no specific issue identified yet, just noted as needing a look
- **(later)** confirm whether Nextcloud is genuinely routed through HAProxy's `services_frontend` correctly end-to-end, or whether it's also still reachable via the direct host-port binding (`10.0.0.21:8080`) in a way that bypasses the intended routing
- Compose file for this stack not yet captured in this repo — *TODO (now, if not already in `/docker/nextcloud/`): locate/document its actual compose.yaml and `.env` alongside the other stacks*
