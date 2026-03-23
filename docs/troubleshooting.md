# Troubleshooting Guide

Common issues and solutions for your *arr stack.

## Table of Contents

- [Download Issues](#download-issues)
- [Import Issues](#import-issues)
- [Seedbox and Syncthing Issues](#seedbox-and-syncthing-issues)
- [Quality Profile Issues](#quality-profile-issues)
- [Indexer and Jackett Issues](#indexer-and-jackett-issues)
- [Application Connection Issues](#application-connection-issues)
- [Jellyfin and Jellyseerr Issues](#jellyfin-and-jellyseerr-issues)

## Download Issues

### Downloads complete but don't import

**Symptoms:**
- Files download successfully
- Show in download client as complete
- Never appear in Jellyfin
- Stuck in Activity queue

**Solutions:**

1. **Check folder permissions**
   ```bash
   chmod -R 777 /mnt/user/data/
   ```
   Or be more specific:
   ```bash
   chmod -R 777 /mnt/user/data/seedbox-sync/
   chmod -R 777 /mnt/user/data/media/
   ```

2. **Verify PUID and PGID**
   - Check Docker container environment variables
   - Should be 99:100 for Unraid
   - Edit container → Add Variable: PUID=99, PGID=100

3. **For seedbox setup: Check remote path mappings**
   - Settings → Download Clients → Remote Path Mappings
   - Host must match exactly
   - Paths must be correct

4. **Check Activity → Queue for errors**
   - Look for specific error messages
   - Common: "No files found eligible for import"
   - Check System → Logs for details

### Torrents not starting

**Local Transmission:**
- Verify Transmission is running (Docker container status)
- Check VPN connection if using VPN container
- Verify credentials are correct in Sonarr/Radarr
- Check Transmission logs for errors

**Seedbox:**
- Verify seedbox Transmission is running (check provider control panel)
- Test connection from Unraid: `telnet seedbox.ultra.cc [port]`
- Verify credentials match between apps
- Check seedbox disk space (full disk = won't start)

### Downloads are slow

**Local:**
- Check VPN speed (try different VPN server)
- Check ISP throttling
- Verify port forwarding is working
- Check Transmission connection settings

**Seedbox:**
- This shouldn't happen with good seedbox
- Check seedbox provider status page
- Contact seedbox support
- Verify you're not hitting seedbox data limits

## Import Issues

### Files import but create duplicates

**Problem:** Both download file AND imported file exist (double disk usage)

**Cause:** Hardlinks not working

**Solutions:**

1. **Verify folder structure**
   - Downloads and media must be under same parent directory
   - ✅ Good: `/mnt/user/data/torrents/` and `/mnt/user/data/media/`
   - ❌ Bad: `/mnt/user/downloads/` and `/mnt/user/media/`

2. **Check Docker volume mappings**
   - Must use same volume root
   - ✅ Good: `/data` → `/mnt/user/data/`
   - ❌ Bad: `/downloads` → `/mnt/user/downloads/`, `/media` → `/mnt/user/media/`

3. **For seedbox: Hardlinks won't work**
   - Files are copied from remote server
   - This is expected behavior
   - Use Sonarr/Radarr to delete files after import
   - Settings → Media Management → Enable "Delete empty folders"

### Wrong naming after import

**Problem:** Files renamed incorrectly or not at all

**Solutions:**

1. **Check naming settings**
   - Settings → Media Management
   - Verify "Rename Episodes/Movies" is enabled
   - Check your naming format

2. **Standard Episode Format:**
   ```
   {Series Title} - S{season:00}E{episode:00} - {Episode Title}
   ```

3. **Standard Movie Format:**
   ```
   {Movie Title} ({Release Year})
   ```

4. **Test naming:**
   - Select a movie/episode
   - Preview in file browser
   - Adjust format as needed

## Seedbox and Syncthing Issues

### Syncthing not transferring files

**Symptoms:**
- Files download on seedbox
- Never appear on Unraid
- Syncthing shows "Up to Date" but files missing

**Solutions:**

1. **Check device connection**
   - Both Syncthing interfaces should show devices as "Connected"
   - Green checkmark = connected
   - Gray = disconnected

2. **Verify folder sharing**
   - Seedbox Syncthing: Folder should be shared with Unraid
   - Unraid Syncthing: Should show folder from seedbox
   - Check folder IDs match

3. **Check folder type**
   - Seedbox folder: "Send Only" or "Send & Receive"
   - Unraid folder: "Receive Only"
   - Don't use "Send & Receive" on both (causes conflicts)

4. **Check firewall**
   - Syncthing uses port 22000 TCP/UDP
   - Verify not blocked on either end
   - Check Unraid firewall settings

5. **Check paths**
   - Seedbox path: Must match what download client reports
   - Unraid path: Must match Docker mapping
   - Look for typos, trailing slashes

6. **Review Syncthing logs**
   - Actions → Logs (in Syncthing interface)
   - Look for sync errors
   - Common: Permission denied, path not found

### Files sync but *arr apps don't import

**This is the most common seedbox issue.**

**Diagnosis:**

1. **Check remote path mapping**
   - Settings → Download Clients → Remote Path Mappings
   - Host field must match download client host EXACTLY
   - Example: If download client uses `server1.ultra.cc`, mapping must too
   - Don't mix IP addresses and hostnames

2. **Verify paths are correct**
   - Go to Activity → Queue
   - Click on stuck download
   - Note the "Output Path" shown
   - This path should match your remote path mapping

3. **Common path mismatches:**
   ```
   Download client reports: /home/user/files/completed/tv/
   But you mapped: /home/user/torrents/completed/tv/
   
   FIX: Update remote path mapping to match actual path
   ```

4. **Check Syncthing sync is complete**
   - File might still be transferring
   - Wait for Syncthing to show "Up to Date"
   - Check file size matches on both ends

5. **Verify file permissions after sync**
   ```bash
   chmod -R 777 /mnt/user/data/seedbox-sync/
   ```

6. **Check Sonarr/Radarr logs**
   - System → Logs
   - Search for the file name
   - Look for "No files eligible for import" or similar
   - Tells you exactly what's wrong

### Syncthing using too much bandwidth

**Solutions:**

1. **Set bandwidth limits**
   - Actions → Settings → Connections
   - Set "Download Rate" limit
   - Set "Upload Rate" limit (important if seedbox sends a lot)

2. **Change sync schedule**
   - Advanced → Folder → Edit
   - Set scan interval to longer (e.g., 3600 seconds = 1 hour)
   - Files sync less frequently

3. **Use folder versioning carefully**
   - File Versioning can create many copies
   - Use "Trash Can" type with low max age
   - Or disable if not needed

## Quality Profile Issues

### Nothing downloads - everything rejected

**Symptoms:**
- Searches find releases
- All rejected
- Queue always empty

**Solutions:**

1. **Lower minimum custom format score**
   - Settings → Profiles → Your Profile
   - Minimum Custom Format Score: Try 0 instead of +100
   - Save and test

2. **Check size limits**
   - Settings → Quality
   - Your minimums might be too high
   - Recommended: 40 MB/min for 1080p (not 100)

3. **Verify quality profile allows qualities**
   - Settings → Profiles
   - Check allowed qualities are selected
   - Bluray-1080p and WEBDL-1080p should be checked

4. **Check cutoff**
   - Cutoff quality should be achievable
   - Don't set to Remux if you don't have Remux indexers

5. **Use Interactive Search to diagnose**
   - Movies/Series → Select item → Interactive Search
   - See why each release is rejected
   - Adjust accordingly

### Still downloading bad releases (YIFY, etc.)

**Symptoms:**
- Bad releases still getting through
- YIFY, RARBG, etc. downloading despite blocks

**Solutions:**

1. **Verify custom format exists**
   - Settings → Custom Formats
   - "LQ Release Groups" should exist
   - Check regex is correct

2. **Verify custom format is assigned to profile**
   - Settings → Profiles → Your Profile
   - Scroll to Custom Formats section
   - "LQ Release Groups" should show -10000 score

3. **Check release name matches regex**
   - Sometimes groups use variations
   - Add variations to regex:
   ```regex
   \b(YIFY|YTS|YTS\.MX|YTS\.AG|RARBG|RARBG\.TO)\b
   ```

4. **Test with Interactive Search**
   - Should show -10000 score for YIFY releases
   - If showing 0, custom format isn't matching

### Files too large or too small

**Solutions:**

1. **Check size limits are set**
   - Settings → Quality → Click quality name
   - Min and Max should be filled in
   - Can't be 0

2. **For Radarr: Use Preferred slider**
   - Movies vary in length
   - Use "Preferred" size limits
   - Set to MB per minute (e.g., 50-250 for 1080p)

3. **For Sonarr: Check per-episode vs season**
   - Size limits are per episode
   - Not for entire season

4. **Remux override:**
   - If you're getting huge Remux files you don't want
   - Check Custom Formats
   - Remux format might have high positive score
   - Lower it or remove from allowed qualities

## Indexer and Jackett Issues

### Jackett indexers not working

**Solutions:**

1. **Test indexer in Jackett**
   - Open Jackett web UI
   - Click "Test" button on indexer
   - Should return results
   - If fails: indexer is down or blocked

2. **Check Torznab URL is correct**
   - In Sonarr/Radarr: Settings → Indexers → Your indexer
   - URL should be copied exactly from Jackett
   - Click "Copy Torznab Feed" in Jackett
   - Paste into Sonarr/Radarr

3. **Verify API key matches**
   - Jackett API key: Top right corner of Jackett UI
   - Copy and paste into indexer settings
   - Don't type manually (easy to make mistake)

4. **Check categories**
   - TV indexers need categories: 5030, 5040
   - Movie indexers need category: 2000
   - Set in indexer settings in Sonarr/Radarr

5. **Indexer might be down**
   - Public trackers go down frequently
   - Check indexer's website directly
   - Remove or disable if permanently down

### Not finding releases you know exist

**Solutions:**

1. **Check indexer has the content**
   - Search manually on indexer's website
   - Verify it exists

2. **Check quality profile**
   - Release might be wrong quality
   - Try expanding allowed qualities temporarily

3. **Check release date vs monitoring**
   - Sonarr won't grab old episodes if "Unmonitor old episodes" is on
   - Manually search instead

4. **Check search terms**
   - Series → Edit → Use alternate titles if needed
   - Some series have different names on different trackers

## Application Connection Issues

### Can't connect apps together

**Symptoms:**
- Adding download client fails
- Adding indexer fails
- Jellyseerr can't connect to Sonarr/Radarr

**Solutions:**

1. **Use server IP, not localhost**
   - ❌ Wrong: `localhost:8989`
   - ❌ Wrong: `127.0.0.1:8989`
   - ✅ Right: `192.168.1.100:8989` (your actual Unraid IP)

2. **Verify ports aren't conflicting**
   - Check Docker container ports
   - Each app needs unique port
   - Edit container if conflict found

3. **Check API keys are correct**
   - Copy from Settings → General
   - Paste exactly (no extra spaces)
   - Keys are case-sensitive

4. **Verify apps are running**
   - Check Docker container status
   - All should show "running"
   - Restart if stopped

5. **Check Unraid network settings**
   - Some setups need bridge mode
   - Some need host mode
   - Try switching if having issues

## Jellyfin and Jellyseerr Issues

### Jellyseerr not finding media

**Solutions:**

1. **Sync libraries**
   - Jellyseerr Settings → Jellyfin
   - Click "Sync Libraries"
   - Wait for completion

2. **Verify Jellyfin library paths**
   - Jellyfin → Dashboard → Libraries
   - Paths should match your media folders
   - `/data/movies` and `/data/tv`

3. **Check Sonarr/Radarr root folders**
   - Must match Jellyfin library paths
   - Sonarr root folder: `/data/media/tv`
   - Radarr root folder: `/data/media/movies`

4. **Force library scan in Jellyfin**
   - Dashboard → Libraries → Scan All Libraries
   - Then sync again in Jellyseerr

### Jellyfin not showing new media

**Solutions:**

1. **Check library auto-scan settings**
   - Dashboard → Library → Enable real-time monitoring
   - Or set scan interval

2. **Manual scan**
   - Dashboard → Libraries → Scan Library
   - Should find new content

3. **Check file permissions**
   ```bash
   chmod -R 777 /mnt/user/data/media/
   ```

4. **Verify paths are correct**
   - Media must be in Jellyfin library path
   - Check Dashboard → Libraries

### Jellyfin transcoding fails

**Solutions:**

1. **Enable hardware acceleration**
   - Dashboard → Playback → Transcoding
   - Select your hardware (Intel QuickSync, NVIDIA, AMD)
   - Requires proper Docker configuration

2. **Check FFmpeg installation**
   - Should be included in LinuxServer.io image
   - Update container if outdated

3. **Check client compatibility**
   - Some files don't need transcoding
   - Force direct play if possible
   - Check codec support on client

## General Debugging Steps

When something isn't working:

1. **Check logs first**
   - Every app has logs: System → Logs
   - Filter by level (Error, Warning)
   - Look for recent entries

2. **Test connections**
   - Use "Test" button wherever available
   - Tells you immediately if connection works

3. **Check one thing at a time**
   - Don't change multiple settings
   - Make one change, test, repeat

4. **Use Interactive Search**
   - Best debugging tool for download issues
   - Shows exactly why releases are rejected

5. **Restart apps**
   - Sometimes settings don't apply until restart
   - Docker → Stop → Start

6. **Check Unraid system**
   - Disk space full?
   - Docker service running?
   - Array started?

## Getting Help

If you're still stuck:

1. **Check logs and gather info**
   - Copy relevant log entries
   - Note exact error messages
   - Document what you've tried

2. **Search existing issues**
   - [This repo's issues](https://github.com/johnpernock/arr-stack/issues)
   - Application Discord servers
   - r/unraid, r/sonarr, r/radarr

3. **Ask for help with details**
   - What you're trying to do
   - What's happening instead
   - Relevant logs
   - What you've already tried

4. **Application-specific support**
   - [Sonarr Discord](https://discord.gg/sonarr)
   - [Radarr Discord](https://discord.gg/radarr)
   - [Jellyfin Forum](https://forum.jellyfin.org/)
   - [Unraid Forum](https://forums.unraid.net/)

---

**Remember:** Most issues are configuration problems, not bugs. Double-check settings, paths, and permissions first!
