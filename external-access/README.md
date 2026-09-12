# WireGuard Tunnel & External Access

Full path for a public request reaching an internal service, end to end:

```
Client → Infomaniak DNS → Hetzner Caddy (public HTTPS, auto-cert)
       → WireGuard tunnel (10.99.0.2 ↔ 10.99.0.1)
       → pfSense services_frontend (HAProxy, public cert)
       → HAProxy ACL routing → internal backend
```

See [`system-map/`](../system-map/system-map.mermaid) for the visual version.

## Hetzner VPS

- Hostname: `wireguard-gateway`
- IP: `178.104.56.15`
- CX23, Nuremberg, KVM virtualization
- Ubuntu 24.04.5 LTS, 2 CPU cores, 3.7GB RAM, no swap
- SSH: `ssh hetzner` (see [`proxmox-host/README.md`](../proxmox-host/README.md) for the SSH config block)

## WireGuard tunnel

### pfSense side
- Tunnel: `tun_wg0`, description "WG to Hetzner", assigned to interface `WG_HETZNER` (opt1)
- Tunnel address: `10.99.0.1/30`, listen port `51820`
- Peer: "Hetzner gateway" — Allowed IPs `10.99.0.2/32`, Endpoint `178.104.56.15:51820`
- Package: `pfSense-pkg-WireGuard 0.2.13_4`, Keep Configuration enabled

### Hetzner side (`/etc/wireguard/wg0.conf`)
```ini
[Interface]
PrivateKey = <REDACTED>
Address = 10.99.0.2/30
ListenPort = 51820

[Peer]
PublicKey = n94YWIFTyl4thLc2F1wmpIQU2znXGtnBOPI8SPz+0nQ=
PresharedKey = <REDACTED>
AllowedIPs = 10.0.0.0/24, 10.99.0.1/32
```

A **pre-shared key** is used in addition to the standard asymmetric handshake — extra layer of security. `AllowedIPs` on this side is broader (`10.0.0.0/24`, not just the tunnel endpoint) so Caddy can route to anything on the internal LAN, not just `10.99.0.1` itself.

*TODO (later): keypair generation method not recorded — check pfSense's tunnel edit screen for whether it was auto-generated or pasted in manually, for reproducibility if this ever needs rebuilding.*

## Caddy (Hetzner)

Installed **natively** (not Docker) — binary at `/usr/bin/caddy`, running as a systemd service (`caddy.service`, enabled at boot). Config: `/etc/caddy/Caddyfile`.

```caddyfile
ha.alarconrivero.dev {
    reverse_proxy https://10.99.0.1:443 {
        header_up Host {host}
        transport http {
            tls_server_name ha.alarconrivero.dev
        }
    }
}
# ...identical block, different hostname, for each of:
# nextcloud.alarconrivero.dev, immich.alarconrivero.dev,
# syncthing.alarconrivero.dev, jellyfin.alarconrivero.dev,
# books.alarconrivero.dev
```

**Why `tls_server_name` is required per-block:** the public wildcard cert HAProxy uses doesn't cover bare IPs. Since Caddy reverse-proxies to pfSense's tunnel IP (`10.99.0.1`) rather than a hostname, it must explicitly set the SNI hostname via `tls_server_name` so HAProxy knows which cert/backend to serve.

**`header_up Host {host}`** preserves the original hostname so HAProxy's ACL-based routing can correctly match and route to the right backend.

**Caddy manages its own Let's Encrypt cert automatically** for its public-facing side — this is separate from the `LE-public-alarconrivero` cert used internally by HAProxy (see below).

**Currently exposed:** `ha`, `nextcloud`, `immich`, `syncthing`, `jellyfin`, `books` — six services. Not yet added: `qbittorrent`, `prowlarr`, `sonarr`, `radarr`, `bazarr`, `lidarr`, `shelfmark` (consistent with those containers not being deployed yet — see [`docker-host/servarr/README.md`](../docker-host/servarr/README.md)).

*TODO (later): add corresponding Caddyfile blocks as those services come online — but see the Suggestion in the Servarr README first about whether they should be exposed publicly at all.*

## Infomaniak DNS

Public DNS provider for `alarconrivero.dev`. Manager: `https://manager.infomaniak.com/`.

Six A records, each pointing individually at `178.104.56.15` (5 min TTL): `ha`, `nextcloud`, `immich`, `syncthing`, `jellyfin`, `books`.

Also present: standard mail records (MX/SPF/DKIM/DMARC via Infomaniak) and a ProtonMail verification TXT record — mail routing appears split between Infomaniak (inbound MX) and Proton (verified, possibly for mailbox hosting/aliasing) — not investigated further.

> 📝 **Suggestion(s) from Claude** — six individual A records all pointing at the same IP means a future IP change requires updating all six manually. Given a wildcard cert (`*.alarconrivero.dev`) already exists, a single CNAME structure (e.g. `*.alarconrivero.dev` → one `gateway` A record) would mean only one record ever needs updating. Not urgent, worth considering during a future cleanup pass.

### API token

Token `letsencrypt-pfsense-2026`, created via `https://manager.infomaniak.com/v3/ng/accounts/token/list`, unlimited expiration (auto-deactivates after 1 year of no use per Infomaniak's own policy — worth knowing as a "why did this suddenly break" possibility). Scopes: `dns:read`, `dns:write`, `domain:read`.

**Why all three scopes are needed** (confirmed against the `acme.sh` `dns_infomaniak` plugin source):
- `domain:read` — looks up which DNS zone a domain belongs to (`GET /2/domains/{domain}/zones`) before the challenge record can be written
- `dns:write` — creates/removes the `_acme-challenge` TXT record
- `dns:read` — verifies/polls for the TXT record after creation, confirming propagation before validation proceeds

## Certificates

### `LE-public-alarconrivero` (the public cert HAProxy uses)

- Issued via pfSense's ACME package, DNS-01 challenge through Infomaniak (using the token above)
- SAN list: `alarconrivero.dev` + `*.alarconrivero.dev` (wildcard)
- Account Key: `LetsEncrypt-Production-New` (Let's Encrypt Production — rate-limited)
- Valid until Dec 9, 2026 (standard 90-day Let's Encrypt lifetime)

**Scheduled renewal:** ✅ enabled and verified working (as of this doc pass — was previously disabled simply because the setup hadn't reached that step yet, not a deliberate choice). Post-Renew Action configured: Shell Command → `/usr/local/etc/rc.d/haproxy.sh restart` — necessary because HAProxy won't pick up a renewed cert on its own without a restart. Verified working via a manual test renewal (HAProxy showed ~1 minute uptime immediately after, confirming the restart action fired correctly).

> 📝 **Suggestion(s) from Claude** — if this cert is ever also used for pfSense's own WebGUI (not currently the case), add a second Post-Renew Action: Shell Command → `/etc/rc.restart_webgui`.

### Other certificates on this pfSense (System → Certificate Manager)

| Name | Issuer | In use | Notes |
|---|---|---|---|
| GUI default (×2) | self-signed | — | Likely leftover pfSense defaults — cleanup candidates |
| pfsense-webgui-2026 | Home-Internal-CA | `webConfigurator` | Separate internal-CA cert for pfSense's own admin login — keep |
| proxmox-haproxy-2026 | Home-Internal-CA | — | No "In Use" marking despite the name — check if genuinely unused before removing |
| LE-public-alarconrivero | Let's Encrypt | `HAProxy (2)`, `Acme (1)` | The active public cert, described above |

**Certificate Authorities:** `Home-Internal-CA` (self-signed, backs the two internal-CA certs above) + three Let's Encrypt intermediate/root CAs, one of which (`YR1`) has 0 certificates using it — likely a fully-rotated-out Let's Encrypt intermediate generation, safe cleanup candidate.

> 📝 **Suggestion(s) from Claude** — two parallel cert systems exist here: the **public Let's Encrypt chain** (for HAProxy-fronted external services) and a **separate internal self-signed CA** (`Home-Internal-CA`, for direct admin-UI-only access like pfSense's own WebGUI). Worth keeping this distinction clear in your head — they serve different purposes and aren't interchangeable.

## TODOs

- **(later)** confirm/clean up the certificate cleanup candidates listed above
- **(later)** confirm Hetzner Cloud Console project URL and Infomaniak Manager domain-specific URL (see [`pfsense/secrets-links.md`](../pfsense/secrets-links.md))
