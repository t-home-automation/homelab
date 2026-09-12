# Jellyfin

Media server. Image: `lscr.io/linuxserver/jellyfin:latest`. Compose file: [`compose.yaml`](./compose.yaml).

Externally accessible at `jellyfin.alarconrivero.dev` — see [`external-access/README.md`](../../external-access/README.md) for the full request path.

## `.env`

```bash
PUID=1000
PGID=1000
TZ=Europe/Amsterdam
```

PUID/PGID must match the host user (`tiko`, `1000:1000`) that owns `/data` — see [`docker-host/README.md`](../README.md) for the `.env` pattern used across all stacks.

## Docs

- [linuxserver/jellyfin image docs](https://docs.linuxserver.io/images/docker-jellyfin/)
- [Jellyfin official container docs](https://jellyfin.org/docs/general/installation/container/)

## Issues encountered + fixes

### Hardware transcoding (Intel QuickSync)
`/dev/dri:/dev/dri` is the container-side half of GPU passthrough. The Proxmox-side setup (PCI passthrough + `render` group) is documented in [`proxmox-host/README.md`](../../proxmox-host/README.md).

### DLNA/local-discovery ports intentionally disabled
`7359/udp` and `1900/udp` are commented out. Not needed since all access goes through pfSense DNS overrides + fixed URLs rather than LAN broadcast discovery — which also wouldn't reliably cross the Docker bridge / pfSense-to-Deco routing boundary even if enabled.

### "Server Mismatch" warning after a config wipe
If `/config` is wiped/reset (e.g. during an image switch), Jellyfin generates a new Server ID, but browsers cache the old one and show a mismatch warning.

**Fix:** clear the browser's site data for the Jellyfin origin, or unregister its PWA service worker (DevTools → Application → Service Workers → Unregister, then Storage → Clear site data).

### UI not updating live after actions (e.g. delete not disappearing immediately)
Root cause: **HAProxy killing Jellyfin's WebSocket connection** after ~30 seconds (HAProxy's default timeout), which carries Jellyfin's real-time push notifications. Deletes/adds were happening correctly on disk, but the browser never got the live update signal.

**Fix:** increased both `Client timeout` (frontend) and `Server timeout` (backend) in HAProxy's `services_frontend`/`jellyfin_backend` config to `3600000` ms (1 hour). See [`pfsense/README.md`](../../pfsense/README.md) for the HAProxy config this applies to.

> 📝 **Suggestion(s) from Claude** — a blanket 1-hour client timeout across *every* service on `services_frontend` (not scoped just to Jellyfin) is worth revisiting once more services are live and real-world behavior can be observed — consider whether it should be narrower/per-backend.

### Empty folder not cleaned up after deleting the last item in it
Deleting the last episode of a season, or the last song in an album, leaves an empty/stale entry in the library. This is **deliberate Jellyfin behavior**, not a bug — it can't distinguish "genuinely empty" from "mount temporarily failed," so it won't purge metadata on its own.

**Fix:** create a placeholder file (e.g. `.forcerescan`) in the now-empty folder, then trigger a library rescan. Jellyfin will then confirm the mount is accessible and clean up the stale entry.

> 📝 **Suggestion(s) from Claude** — the community plugin **"Hide Empty Folders"** automates this cleanup after every library scan (and via a scheduled task), removing the need for the manual placeholder-file dance if this recurs often.

### Docker image bloat during rebuilds
Switching between images (e.g. `jellyfin/jellyfin` → `lscr.io/linuxserver/jellyfin`) leaves old images behind, consuming disk space. Run periodically during active rebuild phases:
```bash
docker image prune -a
```
(Check `docker ps -a` first to confirm nothing currently stopped-but-wanted references an image before pruning — see [`storage/README.md`](../../storage/README.md) for the disk-resize investigation that surfaced this.)

## TODOs

- *TODO (later): Immich will likely eventually point at the `photos`/`videos` folders currently used by Jellyfin for personal media — revisit folder ownership/structure when that happens.*
