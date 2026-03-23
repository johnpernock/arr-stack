# Maintenance Guide

Keep your *arr stack running smoothly with regular maintenance tasks.

## Table of Contents

- [Daily Tasks](#daily-tasks)
- [Weekly Tasks](#weekly-tasks)
- [Monthly Tasks](#monthly-tasks)
- [Quarterly Tasks](#quarterly-tasks)
- [Updating Applications](#updating-applications)
- [Backup Strategy](#backup-strategy)
- [Performance Optimization](#performance-optimization)

## Daily Tasks

### Monitor Download Activity

**What to check:**
- Activity → Queue in Sonarr/Radarr
- Any stuck downloads?
- Any failed imports?

**Time:** 2-3 minutes

**How:**
1. Open Sonarr at `http://[unraid-ip]:8989`
2. Click Activity → Queue
3. Look for:
   - Red X (failed)
   - Orange warning (stalled)
   - Items stuck for hours

**Action items:**
- Failed downloads: Remove and search again
- Stalled downloads: Check download client
- Stuck imports: Check logs for errors

### Check Jellyseerr Requests

**What to check:**
- New user requests
- Pending approvals (if enabled)

**Time:** 1-2 minutes

**How:**
1. Open Jellyseerr at `http://[unraid-ip]:5055`
2. Check Requests tab
3. Approve/deny as needed

## Weekly Tasks

### Review Download History

**What to check:**
- What downloaded this week
- Quality of releases
- Any bad releases getting through

**Time:** 5-10 minutes

**How:**
1. Activity → History
2. Sort by date (last 7 days)
3. Check file sizes and quality
4. Look for patterns

**Action items:**
- Add bad groups to block list
- Adjust size limits if seeing issues
- Update custom formats if needed

### Check Disk Space

**What to check:**
- Seedbox storage (if using)
- Unraid array storage
- Download client space

**Time:** 2-3 minutes

**How:**

**For Unraid:**
```bash
df -h /mnt/user/data/
```

**For Seedbox:**
- Check provider control panel
- Or SSH: `df -h ~/files/`

**Action items:**
- Clean up old torrents if needed
- Delete old media you don't watch
- Upgrade storage if consistently >80% full

### Verify Syncthing Status (Seedbox Users)

**What to check:**
- Both devices connected
- Folders showing "Up to Date"
- No sync errors

**Time:** 2 minutes

**How:**
1. Open both Syncthing interfaces
2. Verify devices show green (connected)
3. Check folders show "Up to Date"
4. Review any errors in logs

**Action items:**
- Restart Syncthing if disconnected
- Clear out sync conflicts if any
- Check network if persistent issues

### Clean Up Failed Downloads

**What to check:**
- Failed downloads in download client
- Incomplete/corrupt files

**Time:** 5 minutes

**How:**

**Transmission:**
1. Open web UI
2. Look for stopped/errored torrents
3. Remove failed ones
4. Delete associated files

**Seedbox:**
- Check your seedbox client
- Remove old failed downloads
- Clean incomplete folder

### Review Jackett Indexer Status

**What to check:**
- Which indexers are up/down
- Response times
- Error rates

**Time:** 3 minutes

**How:**
1. Open Jackett
2. Check status column for each indexer
3. Test any showing errors
4. Review logs if issues

**Action items:**
- Disable permanently down indexers
- Add new indexers if several are down
- Update cookies/credentials if needed

## Monthly Tasks

### Update Applications

**What to update:**
- All Docker containers
- Jackett indexers (built-in update)
- TRaSH Guides via Profilarr

**Time:** 15-20 minutes

**How:**

**Docker Containers (Unraid):**
1. Go to Docker tab
2. Click "Check for Updates"
3. Update one at a time
4. Test after each update

**Recommended order:**
1. Jackett (least critical)
2. Transmission/Syncthing
3. Sonarr
4. Radarr
5. Jellyseerr
6. Profilarr
7. Jellyfin (be careful, test playback after)

**Jackett Indexers:**
1. Open Jackett
2. Some indexers auto-update
3. Check for "Update available" messages
4. Follow prompts

**TRaSH Guides:**
1. Open Profilarr
2. Check for TRaSH updates
3. Review changes before syncing
4. Sync to apps

### Backup Configuration

**What to backup:**
- Unraid appdata folder
- Configuration files
- Custom scripts

**Time:** 10-15 minutes (automated after first setup)

**How:**

**Manual Method:**
```bash
tar -czf /mnt/user/backups/appdata-$(date +%Y%m%d).tar.gz /mnt/user/appdata/
```

**Automated with Community Applications:**
- Install "CA Backup/Restore Appdata" plugin
- Configure automatic backups
- Schedule weekly

**What to include:**
- `/mnt/user/appdata/sonarr/`
- `/mnt/user/appdata/radarr/`
- `/mnt/user/appdata/jackett/`
- `/mnt/user/appdata/jellyseerr/`
- `/mnt/user/appdata/profilarr/`
- `/mnt/user/appdata/jellyfin/`
- `/mnt/user/appdata/transmission/`
- `/mnt/user/appdata/syncthing/`

### Review Quality Profiles

**What to review:**
- Are you getting quality you want?
- Too strict (nothing downloading)?
- Too lenient (garbage downloading)?

**Time:** 10 minutes

**How:**
1. Review last month's downloads
2. Check file sizes vs runtime
3. Look at release groups downloaded
4. Interactive search on a few titles

**Action items:**
- Adjust size limits
- Update custom format scores
- Add/remove blocked groups
- Update TRaSH Guide scores

### Clean Up Jellyfin

**What to clean:**
- Transcode temp files
- Old metadata
- Unused thumbnails

**Time:** 5 minutes

**How:**
1. Dashboard → Scheduled Tasks
2. Run "Clean Transcoding Files"
3. Run "Clean Cache"
4. Optionally scan libraries

### Monitor Resource Usage

**What to check:**
- CPU usage
- RAM usage
- Disk I/O
- Network usage

**Time:** 5 minutes

**How:**

**Unraid Dashboard:**
- Check CPU graph
- Check memory usage
- Check array status

**Action items:**
- If CPU constantly high: reduce concurrent downloads
- If RAM high: restart heavy containers
- If I/O high during Syncthing: limit sync speed

## Quarterly Tasks

### Review and Optimize

**What to review:**
- Are you using all your apps?
- Any features you're not utilizing?
- Any pain points to address?

**Time:** 30-60 minutes

**How:**
1. Review your workflow
2. Check which features you actually use
3. Research new features added to apps
4. Look for optimization opportunities

**Examples:**
- Set up notifications if you haven't
- Configure webhooks for automation
- Enable hardware transcoding in Jellyfin
- Set up reverse proxy for remote access

### Database Maintenance

**What to maintain:**
- Sonarr/Radarr databases
- Jellyfin database

**Time:** 10-15 minutes

**How:**

**Sonarr/Radarr:**
1. System → Backup
2. Create backup first
3. System → Tasks → Housekeeping
4. Let it clean up old history

**Jellyfin:**
1. Dashboard → Scheduled Tasks
2. Run database cleanup tasks
3. Optimize database (if available)

### Review Security

**What to check:**
- Are passwords still strong?
- Is VPN working (if local)?
- Are ports exposed that shouldn't be?
- Any suspicious activity?

**Time:** 15 minutes

**How:**
1. Check Unraid security settings
2. Review VPN logs (if local)
3. Check Jellyfin login history
4. Review Sonarr/Radarr API usage logs

**Action items:**
- Rotate passwords
- Update VPN if needed
- Enable 2FA where available
- Review API key usage

### Major Updates

**What to update:**
- Unraid OS (if updates available)
- Major app version changes

**Time:** 30-60 minutes

**How:**
1. Backup everything first
2. Read release notes
3. Update Unraid OS if available
4. Update major app versions one at a time
5. Test thoroughly after each

**Be careful with:**
- Jellyfin major versions (test playback)
- Sonarr/Radarr v3 → v4 (if/when it happens)
- Any breaking changes mentioned in release notes

## Updating Applications

### Update Strategy

**Golden Rules:**
1. **Always backup before major updates**
2. **Update one at a time**
3. **Test after each update**
4. **Read release notes first**
5. **Have a rollback plan**

### How to Update Docker Containers

**Method 1: Unraid WebUI**
1. Docker tab
2. Click container
3. "Check for Updates"
4. Click update icon if available
5. Container auto-restarts

**Method 2: CA Auto Update**
1. Install "CA Auto Update Applications" plugin
2. Configure which containers to auto-update
3. Set schedule
4. Review logs after updates

### Rolling Back Updates

If an update breaks something:

1. **Stop the container**
2. **Edit container**
3. **Change Repository**
   - Add tag for previous version
   - Example: `linuxserver/sonarr:3.0.8` instead of `latest`
4. **Apply**
5. **Restore from backup if needed**

### Checking for Updates

**Weekly check recommended:**
- Docker containers: Check for Updates button
- Jackett: Built-in update notification
- Unraid plugins: Plugins tab

**Subscribe to announcements:**
- [Sonarr Discord](https://discord.gg/sonarr)
- [Radarr Discord](https://discord.gg/radarr)
- [Jellyfin Blog](https://jellyfin.org/blog)
- [Unraid Forums](https://forums.unraid.net/)

## Backup Strategy

### What to Backup

**Critical (backup before any major change):**
- Sonarr database and config
- Radarr database and config
- Jellyseerr config
- Profilarr config

**Important (backup monthly):**
- Jackett config
- Transmission config
- Syncthing config

**Nice to have (backup quarterly):**
- Jellyfin database (can rebuild from media)
- Custom scripts

### How to Backup

**Automated Appdata Backup:**
```bash
#!/bin/bash
# Save as /mnt/user/scripts/backup-appdata.sh

BACKUP_DIR="/mnt/user/backups/appdata"
DATE=$(date +%Y%m%d)

mkdir -p $BACKUP_DIR

tar -czf $BACKUP_DIR/sonarr-$DATE.tar.gz /mnt/user/appdata/sonarr/
tar -czf $BACKUP_DIR/radarr-$DATE.tar.gz /mnt/user/appdata/radarr/
tar -czf $BACKUP_DIR/jellyseerr-$DATE.tar.gz /mnt/user/appdata/jellyseerr/
tar -czf $BACKUP_DIR/jackett-$DATE.tar.gz /mnt/user/appdata/jackett/

# Keep only last 7 days
find $BACKUP_DIR -name "*.tar.gz" -mtime +7 -delete
```

**Schedule with User Scripts plugin:**
1. Install "User Scripts" plugin
2. Add new script
3. Paste backup script
4. Schedule: Weekly

**Cloud Backup (Optional):**
- Use rclone to sync backups to cloud
- Google Drive, Dropbox, Backblaze B2
- Encrypt before upload

### Restore from Backup

**To restore Sonarr (example):**
1. Stop Sonarr container
2. Delete `/mnt/user/appdata/sonarr/`
3. Extract backup:
   ```bash
   tar -xzf /mnt/user/backups/appdata/sonarr-YYYYMMDD.tar.gz -C /
   ```
4. Start Sonarr container
5. Verify data restored

## Performance Optimization

### Reduce CPU Usage

**If CPU constantly high:**

1. **Limit concurrent downloads**
   - Transmission: Settings → Downloading
   - Max active torrents: 5-10

2. **Reduce scan frequency**
   - Sonarr/Radarr: Settings → Media Management
   - Rescan folder: Longer interval

3. **Disable thumbnail generation**
   - Jellyfin: Dashboard → Playback
   - Uncheck "Enable video image extraction"

### Reduce RAM Usage

**If running out of memory:**

1. **Restart heavy containers monthly**
   - Jellyfin especially
   - Radarr/Sonarr if huge libraries

2. **Reduce library size in memory**
   - Jellyfin: Fewer libraries
   - Sonarr/Radarr: Archive old shows/movies

### Reduce Disk I/O

**If array slow during syncing:**

1. **Limit Syncthing bandwidth**
   - Actions → Settings → Connections
   - Download Rate Limit: 10 MB/s (adjust as needed)

2. **Schedule intensive tasks**
   - Jellyfin scans during off-hours
   - Backup scripts at night

### Optimize Network

**For better download speeds:**

1. **Enable port forwarding**
   - Transmission: Settings → Network
   - Check "Use port forwarding"

2. **Adjust connection limits**
   - Transmission: Settings → Peers
   - Max peers per torrent: 50-100

## Monitoring

### Set Up Monitoring (Optional but Recommended)

**Uptime Kuma:**
- Monitors all apps
- Alerts if any go down
- Dashboard of health

**Grafana + Prometheus:**
- Advanced metrics
- Historical data
- Performance trends

**HOMARR:**
- Beautiful dashboard
- Shows all app statuses
- Quick access to all apps

### Key Metrics to Watch

**Storage:**
- Seedbox: <80% full
- Unraid array: <90% full

**Performance:**
- CPU: <50% average
- RAM: <80% used
- Network: No constant saturation

**App Health:**
- All containers running
- No errors in logs
- API responsive

## Troubleshooting Maintenance

### Container Won't Update

**Solutions:**
1. Stop container
2. Force update in Apps tab
3. Remove and re-add container
4. Check Docker image pull logs

### Backup Script Failing

**Solutions:**
1. Check disk space
2. Verify paths exist
3. Check permissions
4. Review script logs

### High Resource Usage After Update

**Solutions:**
1. Check release notes for known issues
2. Clear cache in app
3. Restart container
4. Roll back if persistent

## Maintenance Checklist

Print this out or keep digitally:

**Daily (2-3 min):**
- [ ] Check Activity → Queue
- [ ] Review any failed downloads

**Weekly (15-20 min):**
- [ ] Review download history
- [ ] Check disk space
- [ ] Verify Syncthing (if seedbox)
- [ ] Clean failed downloads
- [ ] Check Jackett status

**Monthly (45-60 min):**
- [ ] Update all containers
- [ ] Backup configurations
- [ ] Review quality profiles
- [ ] Clean Jellyfin cache
- [ ] Monitor resource usage

**Quarterly (1-2 hours):**
- [ ] Deep review and optimization
- [ ] Database maintenance
- [ ] Security review
- [ ] Major updates (if any)

---

**Remember:** Maintenance prevents problems! Spending an hour per month saves hours of troubleshooting later.
