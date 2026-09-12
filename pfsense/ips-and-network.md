# IPs & Network Reference

Quick lookup for every static IP/hostname across the homelab. See [`README.md`](./README.md) for the systems (DNS Resolver, HAProxy, firewall) that reference these.

## Core infrastructure

| Host | IP | Notes |
|---|---|---|
| Proxmox host | `192.168.68.62` | Mini PC, on the Deco mesh network |
| pfSense LAN | `10.0.0.1` | |
| pfSense WAN | `192.168.68.59` | On the Deco mesh network |
| Home Assistant OS (VM 110) | `10.0.0.20` | DHCP static mapping, MAC `bc:24:11:a7:20:c1` |
| docker-host (VM 120) | `10.0.0.21` | DHCP static mapping, MAC `bc:24:11:2a:41:ec` |
| Hetzner VPS (wireguard-gateway) | `178.104.56.15` | Public IP |
| WireGuard tunnel — pfSense side | `10.99.0.1/30` | Interface: `WG_HETZNER` / `tun_wg0` |
| WireGuard tunnel — Hetzner side | `10.99.0.2/30` | |

## Client devices (trusted admin hosts)

| Device | IP | Notes |
|---|---|---|
| PC | `192.168.68.60` | Static IP set in Deco app |
| Phone | `192.168.68.53` | Static IP set in Deco app |
| MacBook Air M4 | *(not yet assigned)* | Arriving this week — TODO: add static IP + add to `trusted_admin_hosts` |

## Servarr network (`servarrnetwork`, `172.39.0.0/24`)

| Service | IP |
|---|---|
| gluetun | `172.39.0.2` |
| sonarr | `172.39.0.3` *(not yet deployed)* |
| radarr | `172.39.0.4` *(not yet deployed)* |
| lidarr | `172.39.0.5` *(not yet deployed)* |
| bazarr | `172.39.0.6` *(not yet deployed)* |
| seerr | `172.39.0.7` *(not yet deployed)* |
| jellystat-db | `172.39.0.8` *(reserved, unused)* |
| jellystat | `172.39.0.9` *(reserved, unused)* |

## DNS Resolver Host Overrides (`alarconrivero.dev`)

| Host | Points to | Notes |
|---|---|---|
| bazarr | `10.0.0.1` | via services_frontend — not yet deployed |
| books | `10.0.0.1` | CalibreWeb — live |
| docker-host | `192.168.68.59` | SSH port-forward target |
| ha | `10.0.0.1` | Home Assistant |
| immich | `10.0.0.1` | via services_frontend — live |
| jellyfin | `10.0.0.1` | via services_frontend — live |
| lidarr | `10.0.0.1` | via services_frontend — not yet deployed |
| nextcloud | `10.0.0.1` | via services_frontend — live |
| pfsense | `192.168.68.59` | pfSense management |
| prowlarr | `10.0.0.1` | via services_frontend — not yet deployed |
| proxmox | `192.168.68.59` | Proxmox management |
| qbittorrent | `10.0.0.1` | via services_frontend — live, not yet exposed via Caddy |
| radarr | `10.0.0.1` | via services_frontend — not yet deployed |
| shelfmark | `10.0.0.1` | via services_frontend — not yet deployed |
| sonarr | `10.0.0.1` | via services_frontend — not yet deployed |
| syncthing | `10.0.0.1` | via services_frontend — live |

## Externally accessible (public internet, via Hetzner Caddy)

`ha`, `nextcloud`, `immich`, `syncthing`, `jellyfin`, `books` — all resolve via individual Infomaniak A records to `178.104.56.15`. See [`external-access/README.md`](../external-access/README.md).
