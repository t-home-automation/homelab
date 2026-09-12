# Storage & Backups

## Storage pool

- External HDD, **4TB total**, connected directly to the Proxmox mini PC
- Formatted as a **ZFS pool**
- Currently only **1TB carved out** and passed through to the docker-host VM as `/dev/sdb1` → `/data`
- Device-level backup of the pool/disk itself is **off** — only VM-level snapshots are backed up (see below), not the underlying disk/pool

*TODO (later): decide on expanding the allocation beyond 1TB as Immich/Nextcloud/media grow — 3TB currently unused.*

## docker-host root disk resize (Sept 2026)

Root disk was resized from 30GB → 64GB. Procedure:

1. Shut down the VM cleanly (`sudo shutdown -h now`)
2. Proxmox UI: VM → Hardware → disk → Resize disk → add the desired amount (not new total)
3. Boot the VM back up
4. Inside Ubuntu, grow through the full chain:
   ```bash
   sudo growpart /dev/sda 3          # grow the partition
   sudo pvresize /dev/sda3           # extend the LVM physical volume
   sudo lvextend -l +100%FREE /dev/mapper/ubuntu--vg-ubuntu--lv   # extend the logical volume
   sudo resize2fs /dev/mapper/ubuntu--vg-ubuntu--lv               # grow the filesystem
   ```
5. Confirm: `df -h /`

While investigating disk usage that prompted this resize, `docker image prune -a` reclaimed several GB of orphaned images from earlier container rebuilds — worth running periodically during active rebuild phases (see [`docker-host/jellyfin/README.md`](../docker-host/jellyfin/README.md)).

## Scheduled backups (Proxmox VM snapshots)

- **Sunday 3:00 AM** — Proxmox creates VM snapshots (`vzdump`, mode: snapshot, compression: zstd), stored on a dedicated ZFS dataset: **`backups-zfs`** (on the same 4TB pool as the storage pool above)
- **Sunday 4:00 AM** — a cron job runs `rclone`, **mirroring** `backups-zfs` to Proton Drive: anything removed locally gets removed from Proton, anything new gets added — so Proton always reflects the current local retention state exactly
- **Retention:** 4 weekly copies, plus 1 snapshot from each month retained for 6 months

## Manual backups

A bash function in `~/.bashrc` **on the Proxmox host**:

```bash
manual-backup() {
  vzdump "$1" --storage backups-zfs --mode snapshot --compress zstd \
    --notes-template "MANUAL: $2" --protected
}
```

**Usage:** `manual-backup <VMID> "<description>"`

Runs `vzdump` against the given VM, stores to `backups-zfs`, tags with a custom note, and — critically — passes `--protected`, which exempts it from the scheduled retention/pruning rotation (unlike the automatic weekly snapshots).

## TODOs

> 📝 **Suggestion(s) from Claude**
> - **LXC backup coverage** — if any LXC containers get added to Proxmox in the future (separate resource type from VMs), they'd need their own inclusion in the snapshot schedule — easy to overlook since they're not automatically covered by VM-focused backup config.
> - **Backup restore testing** — periodically test-restoring a snapshot (even to a scratch VM) to confirm backups are genuinely restorable, not just "created successfully."
> - **Off-cadence critical-config backups** — small, fast, high-value backups of things like the pfSense config export or the `.env` files documented across this repo, on a tighter cadence than the weekly VM snapshot cycle, given how much manual reconstruction work went into rebuilding some of this configuration.
> - **Failure alerting** — if the Sunday cron job silently fails (rclone auth expiring, disk full, etc.), there's currently no way to notice short of manually checking. A simple webhook/notification on failure would close this gap.
