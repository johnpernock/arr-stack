# CLAUDE.md — arr-stack

Comprehensive guide for setting up a fully automated media stack on Unraid. Documentation only — no application code. Full details in `docs/setup-guide.md` and `README.md`.

---

## What this covers

A complete media automation stack for Unraid covering two setups:

- **Local downloads** — Transmission with VPN running on the Unraid server
- **Remote seedbox** — Ultra.cc seedbox + SyncThing for automatic file transfer to local Media share

---

## Stack components

| Application | Purpose |
|-------------|---------|
| Sonarr | TV show management and automation |
| Radarr | Movie management and automation |
| Jackett | Indexer proxy for torrent trackers |
| Jellyseerr | Media request interface (web UI) |
| Jellyfin | Open-source media server with hardware transcoding |
| Profilarr | TRaSH Guides quality profile sync for Radarr/Sonarr |
| Transmission | Torrent client (local download setup only) |
| SyncThing | File sync from remote seedbox to local Media share |

---

## Unraid context

All containers run on the Unraid server (`192.168.1.x`). See `unraid-guide/` for server hardware and share layout.

- Media share: `/mnt/user/Media` — where Sonarr/Radarr/Jellyfin read files
- Docker appdata: `/mnt/user/appdata` — where container configs live
- SyncThing syncs from remote seedbox into the Media share; Sonarr/Radarr pick up completed downloads via remote path mapping

---

## Documentation structure

```
arr-stack/
├── docs/
│   ├── setup-guide.md           — Full step-by-step install for all containers
│   ├── quality-profiles-guide.md — TRaSH Guides integration via Profilarr
│   ├── seedbox-examples.md      — Remote seedbox + SyncThing configuration
│   ├── maintenance.md           — Ongoing upkeep tasks
│   └── troubleshooting.md       — Common issues and fixes
├── QUICK_START.md               — Fast path for experienced users
└── README.md                    — Overview and table of contents
```

---

## Key notes

- **Profilarr is on beta branch** — may have breaking updates on container pulls
- **Jellyseerr (Seerr) is on develop branch** — same caution applies
- **SyncThing is the seedbox bridge** — syncs downloads remotely, not a local torrent client in the seedbox setup; Radarr/Sonarr import from the synced folder
- **Jellyfin uses Intel iGPU (i915 SR-IOV)** for hardware transcoding — the i915-sriov plugin on Unraid enables iGPU sharing
