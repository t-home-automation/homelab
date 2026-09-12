# Homelab Documentation

Documentation for Toto's self-hosted homelab: media independence, data sovereignty, and a personal cloud running on a Proxmox mini PC, fronted by a Hetzner VPS + WireGuard tunnel for external access.

This repo reflects the setup **as of September 2026**. It was written retroactively, section by section, confirmed live against the actual running configuration rather than assumed from memory — so it should be trusted over any earlier notes/chat history.

## Sections

- [`proxmox-host/`](./proxmox-host/README.md) — the Proxmox mini PC, VM creation, SSH access, Ubuntu setup, iGPU passthrough
- [`docker-host/`](./docker-host/README.md) — the docker-host VM: folder structure, `.env` pattern, and each running/planned service (Jellyfin, Servarr, Homepage, Nextcloud, Immich, Syncthing)
- [`storage/`](./storage/README.md) — ZFS storage pool, scheduled + manual backups
- [`pfsense/`](./pfsense/README.md) — DNS resolver, HAProxy, firewall rules, aliases, NAT; includes a quick IP reference and a secrets-locations guide
- [`external-access/`](./external-access/README.md) — WireGuard tunnel to Hetzner, Caddy, Infomaniak DNS, Let's Encrypt/ACME certificate setup
- [`home-assistant/`](./home-assistant/README.md) — Home Assistant OS VM (currently minimal — mostly TODO)
- [`system-map/`](./system-map/README.md) — a Mermaid diagram of the whole system, plus a short legend

## Conventions used throughout

- **TODO (now)** — small, unresolved items meant to be picked up soon, right after this doc pass
- **TODO (later)** — deferred, no urgency
- **📝 Suggestion(s) from Claude** — ideas Claude proposed during doc-writing; not decided or in progress, just flagged for consideration
- Compose files in this repo are the **real, current, working versions** — copy them directly rather than reconstructing from prose
- `.env` / `secrets.env` files are **never committed** — each relevant README documents their structure and what needs filling in, with placeholder values only

## Master TODO list

See [`todos.md`](./todos.md) for every outstanding item across the whole repo in one place.
