# Servarr Stack

Compose file: [`compose.yaml`](./compose.yaml). Network: `servarrnetwork` (`172.39.0.0/24`), a custom bridge network — gives containers name-based resolution and a fixed, collision-free IP range separate from Docker's default auto-assigned subnets.

## Status

**Live:** `gluetun`, `qbittorrent`, `qbittorrent-port-forward-gluetun-server` (port-sync).
**Not yet built:** `sonarr`, `radarr`, `prowlarr`, `bazarr`, `lidarr`, `nzbget`, `flaresolverr`, `seerr`, `shelfmark` — DNS overrides and HAProxy backends for these already exist in pfSense (provisioned ahead of time), but the containers themselves aren't deployed. See TODOs below.

## `.env`

Single consolidated file — see [`docker-host/README.md`](../README.md) for why this stack uses one `.env` rather than a split `.env`/`secrets.env`.

```bash
# User related values
PUID=1000
PGID=1000
TZ=Europe/Amsterdam

# VPN provider config
VPN_SERVICE_PROVIDER=protonvpn
VPN_TYPE=wireguard
VPN_PORT_FORWARDING=on
PORT_FORWARD_ONLY=on
SERVER_COUNTRIES=Netherlands
VPN_PORT_FORWARDING_STATUS_FILE=/gluetun/forwarded_port
HEALTH_VPN_DURATION_INITIAL=120s

# Static IPs on servarrnetwork (172.39.0.0/24)
# Full IP reference: see pfsense/ips-and-network.md
SET_IP_GLUETUN=172.39.0.2
SET_IP_SONARR=172.39.0.3
SET_IP_RADARR=172.39.0.4
SET_IP_LIDARR=172.39.0.5
SET_IP_BAZARR=172.39.0.6
SET_IP_SEERR=172.39.0.7
#SET_IP_JELLYSTAT_DB=172.39.0.8
#SET_IP_JELLYSTAT=172.39.0.9

# --- Secrets (this file must be gitignored) ---
WIREGUARD_PRIVATE_KEY=CHANGEME   # from ProtonVPN's WireGuard config generator (NAT-PMP enabled)
HARDCOVER_API_KEY=CHANGEME       # reserved for Shelfmark (not yet set up) — https://hardcover.app/account/api
PROWLARR_API_KEY=CHANGEME        # Prowlarr WebUI → Settings → General → Security → API Key
QBT_API_KEY=CHANGEME             # qBittorrent WebUI → Preferences → WebUI → API Key → Generate (needs >= 5.2.0)

# Unused currently — Gluetun's /v1/portforward route is set to auth="none"
# since the active port-forward tool doesn't support credentials. Keep for
# when we switch back to qb-port-sync (pending PR #1 merge).
#GLUETUN_USER=admin
#GLUETUN_PASS=CHANGEME

# Unused currently — the active port-forward tool uses QBT_API_KEY only.
# Keep for the same qb-port-sync fallback scenario as above.
#QBITTORRENT_USER=admin
#QBITTORRENT_PASS=CHANGEME
```

## `gluetun/auth/config.toml`

Required for Gluetun's control server API (used by the port-forward sync tool):

```toml
[[roles]]
name = "port-forward"
routes = ["GET /v1/portforward"]
auth = "none"
```

`auth = "none"` is deliberate — the active port-forward tool (`kirari04/qbittorrent-port-forward-gluetun-server`) has no credential-passing mechanism, so this route must allow unauthenticated access. This does mean `GET /v1/portforward` is reachable by anything that can reach `10.0.0.21:8000` on the LAN — a minor, deliberate exposure trade-off, not an oversight.

## Container sections

### Gluetun (VPN + port forwarding)

ProtonVPN, WireGuard, Netherlands servers, NAT-PMP port forwarding enabled. Runs the shared network namespace that `qbittorrent` and the port-sync tool both attach to via `network_mode: service:gluetun`.

**Port forwarding, why it matters:** without a forwarded port, qBittorrent can only make outgoing connections to peers — incoming connections (needed for good download speed and any seeding) are blocked by ProtonVPN's own NAT. `VPN_PORT_FORWARDING=on` + `PORT_FORWARD_ONLY=on` gets Gluetun a dynamically-assigned forwarded port from Proton on each connection.

**The torrenting port (`6881`) is *not* mapped in `ports:`** — deliberately. Docker's host port mapping only affects traffic reaching the container via the host's own network interfaces; incoming P2P connections via the VPN tunnel arrive through the tunnel interface directly, bypassing Docker's port mapping entirely. Mapping it would be a no-op.

### qBittorrent

Runs via `network_mode: service:gluetun` (all traffic forced through the VPN tunnel). WebUI on port `8090` (swapped from the default `8080` — that port is already used by Nextcloud on this host).

**`WEBUI_PORT` gotcha:** per the linuxserver image's own docs, changing the WebUI port requires updating **both** the host-side port mapping *and* `WEBUI_PORT` to the *same* new number (not remapped, e.g. not `8090:8080`) — otherwise a CSRF/Referer check inside qBittorrent's WebUI rejects requests.

### qBittorrent Port Sync

**Currently active: `kirari04/qbittorrent-port-forward-gluetun-server`**, using qBittorrent's **API key** (Bearer token auth), which sidesteps a real compatibility bug in the alternative tool (see below).

**Why not the more commonly-referenced `qb-port-sync` (ADXGlock)?** qBittorrent ≥5.2.0 changed its login endpoint (`POST /api/v2/auth/login`) to return `204 No Content` on success instead of the old `200 Ok.`. `qb-port-sync` only recognizes `200` as success, so it fails to authenticate even with correct credentials — a confirmed, widespread compatibility issue (also hit other tools like Whisparr against qBittorrent 5.2.0). There's an **open, unmerged PR** fixing this: [ADXGlock/qb-port-sync#1](https://github.com/ADXGlock/qb-port-sync/pull/1) — a small, single-commit fix, but unreviewed. The commented-out fallback block in `compose.yaml` is ready to re-enable once that PR merges/releases.

Kirari04's tool avoids the whole issue by using an **API key** sent as `Authorization: Bearer <key>` and never calling the login endpoint at all.

**Env vars it actually supports** (from its own docs — don't add others, they're silently ignored): `QBT_API_KEY`, `QBT_ADDR`, `GTN_ADDR`. No username/password support of any kind, for either qBittorrent or Gluetun's control server — this is *why* the Gluetun `config.toml` route above is set to `auth = "none"`.

## pfSense cross-links

- [HAProxy backend `qbittorrent_backend`](../../pfsense/README.md#haproxy) — currently only reachable via internal `services_frontend` ACL, not exposed via Caddy externally
- [DNS Resolver override for `qbittorrent.alarconrivero.dev`](../../pfsense/ips-and-network.md)
- Same pattern applies for `sonarr`, `radarr`, `prowlarr`, `bazarr`, `lidarr`, `shelfmark` once each is actually deployed

## TODOs

- **(now)** confirm which service actually uses `HARDCOVER_API_KEY` — ~~resolved: reserved for **Shelfmark**, not yet set up~~
- **(later)** add `sonarr`, `radarr`, `prowlarr`, `bazarr`, `lidarr`, `nzbget`, `flaresolverr`, `seerr`, `shelfmark` to this compose file
- **(later)** swap back to `qb-port-sync` (ADXGlock) once [PR #1](https://github.com/ADXGlock/qb-port-sync/pull/1) merges/releases — the commented block in `compose.yaml` is ready
- **(later)** decide which of these services genuinely need external (Caddy) exposure vs. LAN/tunnel-only access

> 📝 **Suggestion(s) from Claude**
> Given the plan to use Homepage for status visibility and something like Seerr for actual media requests, the *arr apps and qBittorrent's own WebUIs may never need public URLs at all — only Jellyfin, Seerr, and things like Nextcloud/Immich/Syncthing genuinely need external access. Worth considering splitting `services_frontend` into a public-facing frontend and a separate internal-only frontend for admin UIs (arr apps, qBittorrent), rather than one shared frontend that *could* expose everything if a Caddy block were ever added by habit.
