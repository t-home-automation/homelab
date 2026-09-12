# Home Assistant

VM 110 on Proxmox, `10.0.0.20` (DHCP static mapping, MAC `bc:24:11:a7:20:c1` — see [`proxmox-host/README.md`](../proxmox-host/README.md)).

DNS Resolver override: `ha.alarconrivero.dev` → `10.0.0.1` (via `services_frontend`). HAProxy backend: `ha_backend`. Currently the only service using **"Host matches"** rather than "Host starts with" for its ACL (see [`pfsense/README.md`](../pfsense/README.md)).

**Externally accessible** — `ha.alarconrivero.dev` is one of the six services with a public Infomaniak A record and a Caddy block on the Hetzner side (see [`external-access/README.md`](../external-access/README.md)).

## TODOs

- **(later)** this section is intentionally minimal — most of the actual Home Assistant configuration (integrations, automations, dashboards, any add-ons) hasn't been documented yet. Fill in as needed.
- **(later)** confirm whether the current public-facing setup (via Caddy/tunnel/HAProxy) is the intended long-term access method, or whether Home Assistant's own remote-access features (Nabu Casa, etc.) are also in play/preferred.
