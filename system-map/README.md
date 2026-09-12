# System Map

See [`system-map.mermaid`](./system-map.mermaid) for the diagram (renders natively on GitHub, or paste into [mermaid.live](https://mermaid.live)).

## Legend

- **Solid green nodes** — currently live and externally accessible (Jellyfin, Nextcloud, Immich, Syncthing, Books)
- **Dashed yellow node** — Servarr stack, only partially built (Gluetun/qBittorrent live, the rest planned), not yet exposed externally
- **Blue nodes** — the external-facing path (Infomaniak DNS, Hetzner Caddy)
- **Dotted lines** — the external access path (client → Infomaniak → Caddy → tunnel)
- **Solid lines from clients** — the internal/LAN path (client → pfSense DNS Resolver directly)

Both paths converge at HAProxy, which routes to the actual backend service based on hostname ACLs.

## Related docs

- [`external-access/README.md`](../external-access/README.md) — full detail on the external path
- [`pfsense/README.md`](../pfsense/README.md) — HAProxy routing detail
- [`docker-host/README.md`](../docker-host/README.md) — what's actually running on docker-host
