# Where to Find/Generate Each Credential

No actual secrets live in this repo. This is a map of **where to go get or regenerate** each credential used across the homelab.

| Credential | Used by | Where to get/generate it |
|---|---|---|
| ProtonVPN WireGuard private key | Gluetun (`servarr/.env`) | [account.protonvpn.com/downloads](https://account.protonvpn.com/downloads) → WireGuard configuration → generate a config with NAT-PMP/port forwarding enabled → copy `PrivateKey` from the downloaded `.conf` |
| Hardcover API key | Shelfmark (not yet set up) | [hardcover.app/account/api](https://hardcover.app/account/api) |
| Prowlarr API key | Prowlarr (not yet deployed) | Prowlarr WebUI → Settings → General → Security → API Key |
| qBittorrent API key | qBittorrent port-forward sync tool | qBittorrent WebUI → Preferences → WebUI → API Key → Generate (needs qBittorrent ≥ 5.2.0) |
| Infomaniak API token (`letsencrypt-pfsense-2026`) | pfSense ACME (DNS-01 challenge for the Let's Encrypt cert) | [manager.infomaniak.com/v3/ng/accounts/token/list](https://manager.infomaniak.com/v3/ng/accounts/token/list) — needs `dns:read`, `dns:write`, `domain:read` scopes (see [`external-access/README.md`](../external-access/README.md) for why each scope is needed) |
| WireGuard keypair (Hetzner tunnel) | pfSense `WG_HETZNER` tunnel + Hetzner `/etc/wireguard/*.conf` | *TODO (later): origin/generation method not recorded — check pfSense's tunnel edit screen, which may indicate whether it was auto-generated* |
| SSH key for docker-host | `~/.ssh/docker_host_ed25519` on the Windows PC | Generated locally via standard `ssh-keygen` flow — not stored anywhere remote |
| SSH key for Hetzner | `~/.ssh/hetzner_wg_ed25519` on the Windows PC | Same as above |

## Panel / console links

| Service | URL |
|---|---|
| Hetzner Cloud Console | `https://console.hetzner.cloud/` *(exact project URL not yet recorded — TODO later)* |
| Infomaniak Manager | `https://manager.infomaniak.com/` |
| Infomaniak API tokens | `https://manager.infomaniak.com/v3/ng/accounts/token/list` |
| pfSense WebGUI | `https://pfsense.alarconrivero.dev:8443` (LAN/trusted-admin-hosts only) |
| Proxmox WebUI | `https://proxmox.alarconrivero.dev` |
