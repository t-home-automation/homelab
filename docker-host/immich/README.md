# Immich

Status: **live/populated**, running on docker-host.

Externally accessible at `immich.alarconrivero.dev` — see [`external-access/README.md`](../../external-access/README.md) for the full request path.

Currently uses its own `/data/immich` folder. The `/data/photos` and `/data/videos` folders (currently used by Jellyfin for personal media) will likely be pointed at by Immich in the future instead — not yet done.

## TODOs

- **(later)** review/reorganize this setup — flagged during docs pass, no specific issue identified yet
- **(later)** decide on and execute the folder reorganization (Immich → `photos`/`videos`) once ready
- Compose file for this stack not yet captured in this repo — *TODO (now, if not already in `/docker/immich/`): locate/document its actual compose.yaml and `.env` alongside the other stacks*
