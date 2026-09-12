# Master TODO List

Consolidated from every section. See each section's own README for full context on any item.

## TODO (now) — pick up soon

- [ ] Set up a way to pull this repo's compose/env-example files directly onto docker-host (e.g. `git clone`) — [`proxmox-host/README.md`](./proxmox-host/README.md), [`docker-host/README.md`](./docker-host/README.md)
- [ ] Add MacBook Air M4's static IP to `trusted_admin_hosts` once it arrives — [`pfsense/README.md`](./pfsense/README.md)
- [ ] Locate/document Nextcloud's, Immich's, and Syncthing's actual compose.yaml + `.env` files — [`docker-host/nextcloud/`](./docker-host/nextcloud/README.md), [`docker-host/immich/`](./docker-host/immich/README.md), [`docker-host/syncthing/`](./docker-host/syncthing/README.md)
- [ ] Confirm Nextcloud is genuinely routed through HAProxy end-to-end, not still reachable via a bypassing direct host-port binding — [`docker-host/nextcloud/README.md`](./docker-host/nextcloud/README.md)

## TODO (later) — deferred, no urgency

- [ ] Record Ubuntu installer choices next time a VM is built from scratch (partitioning, OpenSSH-at-install, Featured Server Snaps) — [`proxmox-host/README.md`](./proxmox-host/README.md)
- [ ] Document why the docker-host VM's RAM ballooning range (3GB→9GB) was chosen — [`proxmox-host/README.md`](./proxmox-host/README.md)
- [ ] Recover verbatim iGPU passthrough commands from shell history, if ever needed — [`proxmox-host/README.md`](./proxmox-host/README.md)
- [ ] Confirm how the WireGuard tunnel keypair was originally generated — [`external-access/README.md`](./external-access/README.md)
- [ ] Expand the storage pool allocation beyond 1TB as Immich/Nextcloud/media grow — [`storage/README.md`](./storage/README.md)
- [ ] Add `sonarr`, `radarr`, `prowlarr`, `bazarr`, `lidarr`, `nzbget`, `flaresolverr`, `seerr`, `shelfmark` to the Servarr compose stack — [`docker-host/servarr/README.md`](./docker-host/servarr/README.md)
- [ ] Swap back to `qb-port-sync` (ADXGlock) once its PR #1 merges/releases — [`docker-host/servarr/README.md`](./docker-host/servarr/README.md)
- [ ] Decide which servarr services genuinely need external (Caddy) exposure vs. LAN-only access — [`docker-host/servarr/README.md`](./docker-host/servarr/README.md)
- [ ] Deploy Homepage dashboard once qBittorrent/LazyLibrarian ports are confirmed — [`docker-host/homepage/README.md`](./docker-host/homepage/README.md)
- [ ] Review/reorganize Nextcloud and Immich setups — [`docker-host/nextcloud/`](./docker-host/nextcloud/README.md), [`docker-host/immich/`](./docker-host/immich/README.md)
- [ ] Execute the photos/videos → Immich folder reorganization — [`docker-host/immich/README.md`](./docker-host/immich/README.md)
- [ ] Clean up flagged certificates and unused CA — [`external-access/README.md`](./external-access/README.md)
- [ ] Confirm Hetzner Cloud Console project URL and Infomaniak Manager domain-specific URL — [`pfsense/secrets-links.md`](./pfsense/secrets-links.md)
- [ ] Consider consolidating the six Infomaniak A records into a wildcard-CNAME structure — [`external-access/README.md`](./external-access/README.md)
- [ ] HAProxy ACL match-type consistency cleanup ("starts with" vs "matches") — [`pfsense/README.md`](./pfsense/README.md)
- [ ] Fill in actual Home Assistant configuration detail (integrations, automations, add-ons) — [`home-assistant/README.md`](./home-assistant/README.md)
- [ ] TP-Link Deco SYN/RST injection issue — revisit if it recurs (firmware update, wired-bypass test, or vendor bug report)
- [ ] Consider whether `/docker` and `/data` configs should be git-tracked directly, separate from this docs repo — [`docker-host/README.md`](./docker-host/README.md)

## 📝 Suggestions from Claude (not decided, just proposed)

- SSH hardening: disable password auth, disable root login — [`proxmox-host/README.md`](./proxmox-host/README.md)
- Homepage: use native service widgets, Docker integration, bookmarks section, Gluetun status widget — [`docker-host/homepage/README.md`](./docker-host/homepage/README.md)
- Split `services_frontend` into public vs. internal-only frontends — [`docker-host/servarr/README.md`](./docker-host/servarr/README.md)
- Revisit whether the blanket 1-hour HAProxy client timeout should be scoped more narrowly — [`docker-host/jellyfin/README.md`](./docker-host/jellyfin/README.md), [`pfsense/README.md`](./pfsense/README.md)
- Install "Hide Empty Folders" Jellyfin plugin to automate the empty-library-folder cleanup — [`docker-host/jellyfin/README.md`](./docker-host/jellyfin/README.md)
- LXC backup coverage, backup restore testing, off-cadence critical-config backups, backup failure alerting — [`storage/README.md`](./storage/README.md)
- Consider narrowing the broad `192.168.68.0/22 → self:443` WAN firewall rule if it's not meant to cover the whole mesh — [`pfsense/README.md`](./pfsense/README.md)
- Decide whether to keep or remove the WG_HETZNER ICMP test rule — [`pfsense/README.md`](./pfsense/README.md)
- Add a Post-Renew Action for the pfSense WebGUI cert, if `LE-public-alarconrivero` is ever reused there — [`external-access/README.md`](./external-access/README.md)
