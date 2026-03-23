# Quick Start Cheat Sheet

Fast reference for common tasks and configurations.

## 🚀 Installation Order

1. **Transmission** (or skip if using seedbox) - Port 9091
2. **Syncthing** (if using seedbox) - Port 8384
3. **Jackett** - Port 9117
4. **Sonarr** - Port 8989
5. **Radarr** - Port 7878
6. **Jellyseerr** - Port 5055
7. **Jellyfin** - Port 8096
8. **Profilarr** - Port 5500

## 📁 Essential Folder Structure

```
/mnt/user/data/
├── seedbox-sync/      # Syncthing receives here (seedbox users)
│   ├── tv/
│   └── movies/
├── torrents/          # Local downloads (local users)
│   ├── tv/
│   └── movies/
└── media/             # Final organized media
    ├── tv/
    └── movies/
```

## 🔧 Critical Docker Mappings

All containers need:
```
/config → /mnt/user/appdata/[app-name]/
/data → /mnt/user/data/
```

## ⚙️ Essential Configurations

### Sonarr/Radarr Download Client

**Local Transmission:**
- Host: `[unraid-ip]`
- Port: `9091`

**Seedbox:**
- Host: `server1.ultra.cc` (your seedbox hostname)
- Port: Check your UCP
- Remote Path Mapping REQUIRED

### Remote Path Mapping (Seedbox)

```
Host: server1.ultra.cc (match download client exactly!)
Remote Path: /home/username/files/completed/tv/
Local Path: /data/seedbox-sync/tv/
```

### Jackett Indexers in Sonarr/Radarr

1. Settings → Indexers → Add → Torznab
2. Copy Torznab Feed URL from Jackett
3. Copy API Key from Jackett
4. Categories:
   - TV: `5030,5040`
   - Movies: `2000`

## 🎯 Quality Profile Quick Setup

### Size Limits (Settings → Quality)

| Quality | Min MB/min | Max MB/min |
|---------|------------|------------|
| 1080p Bluray | 50 | 250 |
| 1080p WEB | 20 | 150 |
| 4K Bluray | 100 | 400 |

### Must-Block Custom Formats

**LQ Groups** (-10000):
```regex
\b(YIFY|YTS|RARBG|PSA|FGT|MeGusta)\b
```

**CAM/TS** (-10000):
```regex
\b(CAM|TS|TELESYNC|SCREENER)\b
```

**Hardcoded Subs** (-10000):
```regex
\b(HC|HARDCODED)\b
```

## 🔍 Testing Commands

**Test connection to seedbox:**
```bash
telnet server1.ultra.cc [port]
```

**Check Syncthing status:**
- Both sides should show "Connected" and "Up to Date"

**Test file transfer:**
```bash
# On seedbox, create test file:
touch /home/username/files/completed/test.txt

# Should appear on Unraid:
ls /mnt/user/data/seedbox-sync/test.txt
```

## 🐛 Quick Troubleshooting

**Nothing downloading?**
1. Check Jackett indexers (test each)
2. Check size limits (too strict?)
3. Lower minimum custom format score to 0

**Files won't import?**
1. Check remote path mapping (exact match?)
2. Check permissions: `chmod -R 777 /mnt/user/data/`
3. Check Syncthing is "Up to Date"

**Syncthing not syncing?**
1. Check both devices show "Connected"
2. Verify folder is shared to correct device
3. Check firewall (port 22000)

## 📊 Default Credentials

| App | Default User | Default Pass | Change? |
|-----|--------------|--------------|---------|
| Transmission | varies | varies | ✅ Yes |
| Jackett | none | none | Set auth ✅ |
| Sonarr | none | none | Set auth ✅ |
| Radarr | none | none | Set auth ✅ |
| Jellyfin | (set on first run) | - | - |
| Jellyseerr | (import from Jellyfin) | - | - |

## 🔐 Security Checklist

- [ ] Set authentication on all apps
- [ ] Use VPN (local) or seedbox (remote)
- [ ] Don't expose ports to internet without reverse proxy
- [ ] Use strong passwords
- [ ] Enable Jellyfin 2FA if exposing externally

## 📱 Useful Links

**Quick Access (Replace [unraid-ip]):**
- Transmission: `http://[unraid-ip]:9091`
- Syncthing: `http://[unraid-ip]:8384`
- Jackett: `http://[unraid-ip]:9117`
- Sonarr: `http://[unraid-ip]:8989`
- Radarr: `http://[unraid-ip]:7878`
- Jellyseerr: `http://[unraid-ip]:5055`
- Jellyfin: `http://[unraid-ip]:8096`
- Profilarr: `http://[unraid-ip]:5500`

**Documentation:**
- [Setup Guide](docs/setup-guide.md)
- [Quality Profiles](docs/quality-profiles-guide.md)
- [Troubleshooting](docs/troubleshooting.md)
- [Seedbox Examples](docs/seedbox-examples.md)

**External Resources:**
- [TRaSH Guides](https://trash-guides.info/)
- [Sonarr Wiki](https://wiki.servarr.com/sonarr)
- [Radarr Wiki](https://wiki.servarr.com/radarr)

## ⏱️ Maintenance Schedule

**Daily:** Check Activity → Queue (2 min)

**Weekly:**
- Review downloads (5 min)
- Check disk space (2 min)
- Verify Syncthing (2 min)

**Monthly:**
- Update containers (15 min)
- Backup configs (10 min)
- Review quality profiles (10 min)

## 🎬 Complete Workflow Example

```
1. User opens Jellyseerr → Requests "The Matrix"
2. Jellyseerr → Radarr
3. Radarr → Searches Jackett
4. Jackett → Queries indexers
5. Radarr → Finds best release, sends to seedbox
6. Seedbox → Downloads file
7. Syncthing → Transfers to Unraid
8. Radarr → Detects via remote path mapping
9. Radarr → Imports to /data/media/movies/
10. Jellyfin → Scans, finds movie
11. Jellyseerr → Syncs, shows "Available"
12. User → Watches movie!
```

## 💡 Pro Tips

1. **Start simple** - Get basic setup working before quality profiles
2. **Use Interactive Search** - Best way to debug quality issues
3. **TRaSH Guides via Profilarr** - Don't reinvent the wheel
4. **Test with one movie/show** - Before bulk adding
5. **Read the logs** - They tell you exactly what's wrong
6. **Document your setup** - Future you will thank you

---

**Need more detail?** See the full guides in the [docs](docs/) folder!
