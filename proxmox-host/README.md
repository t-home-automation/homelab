# Proxmox Host

Physical machine: **HP EliteDesk 800 G3 Mini**, i5-7500T, 16GB RAM, at `192.168.68.62`.

Runs 3 VMs:

| VM | ID | IP | Role |
|---|---|---|---|
| pfSense CE | 100 | LAN `10.0.0.1`, WAN `192.168.68.59` | Router/firewall, DNS, reverse proxy |
| Home Assistant OS | 110 | `10.0.0.20` | See [`home-assistant/`](../home-assistant/README.md) |
| docker-host | 120 | `10.0.0.21` | Runs all Docker workloads — see [`docker-host/`](../docker-host/README.md) |

Both VM IPs are **DHCP static mappings** configured in pfSense (Services → DHCP Server → LAN → Static Mappings) — not netplan static IPs set on the VM itself. pfSense always hands out the same address based on MAC address:

| Hostname | IP | MAC |
|---|---|---|
| docker-host-internal | 10.0.0.21 | `bc:24:11:2a:41:ec` |
| home-assistant | 10.0.0.20 | `bc:24:11:a7:20:c1` |

## docker-host VM specs

From `/etc/pve/qemu-server/120.conf`:

```
agent: 1
balloon: 3072
cores: 4
cpu: host
hostpci0: 0000:00:02
memory: 9216
name: docker-host
net0: virtio=BC:24:11:2A:41:EC,bridge=vmbr1,firewall=1
ostype: l26
scsi0: local-lvm:vm-120-disk-0,iothread=1,size=64G
scsi1: storage-pool:vm-120-disk-0,backup=0,iothread=1,size=1T
scsihw: virtio-scsi-single
```

- **CPU**: 4 cores (max available on this hardware; also the count recommended by [this setup guide](https://www.youtube.com/watch?v=twJDyoj0tDc), which the overall build loosely follows)
- **RAM**: 3GB baseline, ballooning up to 9GB
  - *TODO (later): document why this ballooning range was chosen — decided in an earlier session, reasoning not recorded here.*
- **Root disk**: started at 32GB, resized to 64GB — see [`storage/README.md`](../storage/README.md) for the resize procedure and the storage pool this VM's `/data` disk comes from
- **OS**: `ubuntu-24.04.3-live-server-amd64.iso`

## SSH access

Key generated on the Windows PC using the standard `ssh-keygen` flow, key at `~/.ssh/docker_host_ed25519`. Not shared with other VMs — pfSense and Proxmox are managed via their web UIs instead.

`~/.ssh/config` on the Windows PC:

```
Host docker-host
    HostName docker-host.alarconrivero.dev
    User tiko
    ServerAliveInterval 30
    ServerAliveCountMax 3
    TCPKeepAlive no
    IPQoS throughput
    IdentityFile ~/.ssh/docker_host_ed25519

Host hetzner
    HostName 178.104.56.15
    User root
    IdentityFile ~/.ssh/hetzner_wg_ed25519
```

`ssh docker-host` resolves via pfSense's DNS Resolver override for `docker-host.alarconrivero.dev` → `192.168.68.59` (pfSense WAN IP) → NAT port-forward → `10.0.0.21:22`. This means **SSH access depends on pfSense's DNS Resolver being up** — see [`pfsense/README.md`](../pfsense/README.md) for the NAT rule and firewall alias (`trusted_admin_hosts`) that gates this.

### Suggested SSH hardening (not yet applied)

> 📝 **Suggestion(s) from Claude**
> - Disable password authentication entirely (`PasswordAuthentication no` in `/etc/ssh/sshd_config`) — since key-based auth is already in exclusive use, this closes off brute-force attempts with minimal effort.
> - Disable root SSH login (`PermitRootLogin no`) — likely already effectively true via convention (using `tiko`, not root), worth confirming it's explicit rather than assumed.
> - Fail2ban and a non-default SSH port are lower priority given access is already gated behind the trusted-admin-hosts firewall rule — not urgent, but options if you want additional layers.

## Ubuntu Server install

Most configuration was done **post-install** rather than during the installer.

- *TODO (later): if reinstalling or setting up a new VM in future, record: partitioning method (guided/LVM vs. custom), whether OpenSSH was enabled at install time vs. installed after, and any Featured Server Snaps selected.*
- User: `tiko`, UID/GID `1000:1000`
- Docker installed via the official convenience script:
  ```bash
  sudo sh get-docker.sh
  ```

## Intel iGPU passthrough (for Jellyfin hardware transcoding)

**Proxmox side** — raw PCI passthrough:
- VM 120 → Hardware tab → Add → PCI Device → raw Intel graphics device
- Results in `hostpci0: 0000:00:02` in the VM config (see above)

**docker-host VM side**, after the PCI device was added:
```bash
sudo usermod -aG render tiko
sudo apt install intel-gpu-tools
sudo intel_gpu_top   # verify GPU is accessible/in use
```

Confirms as working — `/dev/dri` shows `card0`, `card1`, and `renderD128` (group: `render`). Jellyfin's compose file maps `/dev/dri:/dev/dri`, and the container inherits access via `tiko`'s `render` group membership (PUID/PGID `1000`).

- *TODO (later): recover the exact verbatim commands from shell history (`history | grep -i render`) if a fully reproducible script is ever needed — the steps above are an accurate reconstruction, not copy-pasted history.*

> 📝 **Suggestion(s) from Claude**
> Keypair generation method for the WireGuard tunnel (see [`external-access/README.md`](../external-access/README.md)) also isn't recorded — check the tunnel's pfSense edit screen, which may still indicate whether the private key was auto-generated or pasted in manually.

## Getting the docker-host compose/env files from GitHub

*TODO (now): set up a way to pull this repo (or just the `docker-host/` compose + env-example files) directly onto docker-host — e.g. `git clone` this repo into `/docker` or a subfolder, or a small pull script — so a fresh docker-host rebuild doesn't require manually recreating every compose file from scratch.*
