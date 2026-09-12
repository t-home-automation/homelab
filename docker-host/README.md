# Docker Host

VM 120 on Proxmox, `10.0.0.21`, Ubuntu 24.04.3, user `tiko` (UID/GID `1000:1000`). See [`proxmox-host/README.md`](../proxmox-host/README.md) for VM specs, SSH access, and iGPU passthrough setup.

## Subsections

- [`jellyfin/`](./jellyfin/README.md) — media server
- [`servarr/`](./servarr/README.md) — gluetun/qBittorrent/port-sync (live), remaining *arr apps (planned)
- [`homepage/`](./homepage/README.md) — dashboard (not yet deployed)
- [`nextcloud/`](./nextcloud/README.md) — file sync/cloud (live, needs review)
- [`immich/`](./immich/README.md) — photo management (live, needs review)
- [`syncthing/`](./syncthing/README.md) — file sync (live)

## Folder structure

```
/data/
├── books
├── downloads/
│   ├── qbittorrent/
│   │   ├── completed
│   │   ├── incomplete
│   │   └── torrents
│   └── nzbget/
│       ├── completed
│       ├── intermediate
│       ├── nzb
│       ├── queue
│       └── tmp
├── movies
├── music
├── shows
├── immich       # populated, active — TODO (later): review/reorganize
├── nextcloud    # populated, active
├── syncthing    # populated, active
├── photos       # currently used by Jellyfin; personal photos — Immich will likely point here later
├── videos       # currently used by Jellyfin; personal videos
└── lost+found

/docker/
├── .env.template                      # template only, not read by Compose — copy the 3 lines into
│                                       # any new stack's own .env
├── jellyfin/
│   ├── compose.yaml
│   └── .env
└── servarr/
    ├── compose.yaml
    ├── .env
    └── gluetun/
        └── auth/
            └── config.toml
```

`/data` lives on a 1TB partition of a larger 4TB ZFS pool — see [`storage/README.md`](../storage/README.md) for the storage pool and backup setup.

**Notes:**
- No `youtube` folder exists (was mentioned once, then dropped — not in use).
- No git tracking currently on `/docker` or `/data` — *TODO (later): consider whether the actual compose files/configs should be git-tracked directly, separate from this documentation repo.*
- *TODO (now): see [`proxmox-host/README.md`](../proxmox-host/README.md) — set up a way to pull this repo's compose/env-example files onto docker-host directly (e.g. `git clone`), rather than manually recreating them from scratch on a rebuild.*

## `.env` setup and linking pattern

**Current approach: one standalone `.env` per stack, no symlinking.** Each stack folder (`jellyfin/`, `servarr/`, and any future one) has its own real `.env` file containing everything Compose needs — both non-secret shared values (`PUID`, `PGID`, `TZ`) and, where relevant, stack-specific config and secrets together in a single file.

This was arrived at after trying a symlinked shared root `.env` + per-stack `secrets.env` split — that approach worked but added complexity (remembering which file held which variable, `env_file:` vs. `${...}` interpolation confusion) for limited benefit at this scale. Consolidating to one file per stack was simpler and matches the pattern used by the reference guide this build loosely follows ([TechHutTV's homelab repo](https://github.com/TechHutTV/homelab)).

A `/docker/.env.template` file exists purely as a copy-paste starting point for new stacks (not read by Compose — just a reference):
```bash
PUID=1000
PGID=1000
TZ=Europe/Amsterdam
```

### Two Compose mechanisms — don't confuse them

1. **Variable interpolation** (`${PUID}` written directly in a compose file's `environment:` block) — resolved automatically by `docker compose` using whatever file is literally named `.env` in the **same directory** you run the command from. No `env_file:` needed for this.
2. **`env_file:`** — injects a file's contents directly as environment variables **inside a container** at start time. Does *not* feed into `${...}` interpolation.

Both mechanisms are used across the compose files in this repo — check each service's block to see which applies.

### Known gotchas hit during setup

- A file referenced via `env_file:` (e.g. an old `secrets.env`) is **not** the same as the directory's `.env` for interpolation purposes — `${VAR}` will resolve blank if `VAR` only exists in an `env_file:`-referenced file, not the literal `.env`.
- Quoted values in an `env_file:`-sourced file are taken **literally, quotes included** — e.g. `PASS="abc"` sets the value to `"abc"` (with quote characters), not `abc`. Don't quote values unless a specific tool explicitly expects it.
- Variable naming must match **exactly** what a given tool's documentation expects — a mismatched name (e.g. `GLUETUN_CONTROL_PASS` vs. the tool's expected `GLUETUN_PASS`) fails silently as a blank/default value, not an obvious error.
