# Seedbox Configuration Examples

Real-world examples of seedbox setups with complete configurations.

## Table of Contents

- [Ultra.cc with Transmission](#ultracc-with-transmission)
- [Understanding Remote Path Mappings](#understanding-remote-path-mappings)
- [Common Seedbox Providers](#common-seedbox-providers)
- [Alternative Download Clients](#alternative-download-clients)

## Ultra.cc with Transmission

This is a complete, tested configuration using Ultra.cc seedbox service.

### Why Ultra.cc?

- **Excellent support:** Quick response times, knowledgeable staff
- **Easy setup:** One-click app installations
- **Syncthing included:** Pre-installed and easy to configure
- **Reliable:** Great uptime and performance
- **EU location:** Netherlands datacenter with excellent peering
- **Competitive pricing:** Good value for features offered

### Basic Information

**Example Account:**
- Hostname: `server1.ultra.cc`
- Username: `johndoe`
- Control Panel (UCP): `https://server1.ultra.cc/ucp`

### Step-by-Step Configuration

#### 1. Ultra.cc Control Panel Setup

**Enable Transmission:**
1. Log into UCP at `https://server1.ultra.cc/ucp`
2. Navigate to "Applications" or "Apps"
3. Find Transmission and click to enable
4. Note your Transmission port (shown in UCP, e.g., `19091`)

**Configure Transmission:**
1. Click on Transmission to access web interface
2. Settings (wrench icon) → Downloading
3. Set download directory to: `/home/johndoe/files/completed/`
4. Enable "Keep incomplete torrents in": `/home/johndoe/files/incomplete/`
5. Apply settings

**Create Directory Structure:**
Via SSH or UCP file manager:
```bash
mkdir -p /home/johndoe/files/completed/tv
mkdir -p /home/johndoe/files/completed/movies
mkdir -p /home/johndoe/files/incomplete
```

#### 2. Syncthing Configuration

**On Seedbox (Ultra.cc):**
1. Enable Syncthing in UCP
2. Access Syncthing web interface (link in UCP)
3. Actions → Show ID → Copy your Device ID
4. Default Folder → Edit
   - Folder Path: `/home/johndoe/files/completed/`
   - Folder Label: `Seedbox Completed`
   - Under "Sharing" (will add Unraid device after next step)

**On Unraid:**
1. Install `linuxserver/syncthing` from CA
2. Configure container:
   ```
   /config → /mnt/user/appdata/syncthing/
   /data → /mnt/user/data/seedbox-sync/
   ```
3. Access Syncthing at `http://[unraid-ip]:8384`
4. Actions → Show ID → Copy Unraid Device ID

**Connect Devices:**
1. In Seedbox Syncthing:
   - Add Remote Device (paste Unraid Device ID)
   - Device Name: "Unraid Server"
   - Save
2. In Unraid Syncthing:
   - Accept the device connection request
3. In Seedbox Syncthing:
   - Edit "Seedbox Completed" folder
   - Sharing tab → Select "Unraid Server"
   - Save
4. In Unraid Syncthing:
   - Accept the folder share
   - Folder Path: `/data/` (maps to /mnt/user/data/seedbox-sync/)
   - Folder Type: **Receive Only**
   - File Versioning: "Trash Can File Versioning" (optional)
   - Save

**Verify Connection:**
- Both Syncthing interfaces should show devices as "Connected"
- Folder should show "Up to Date"
- Test by creating a file on seedbox, should appear on Unraid

#### 3. Sonarr Configuration

**Add Download Client:**
1. Settings → Download Clients → Add Download Client
2. Select "Transmission"
3. Configure:
   - Name: `Ultra.cc Transmission`
   - Host: `server1.ultra.cc`
   - Port: `19091` (from UCP)
   - Username: `johndoe`
   - Password: `[your Ultra.cc password]`
   - Directory: `tv`
   - Category: (leave blank for Transmission)
4. Test → Should show green checkmark
5. Save

**Configure Remote Path Mapping:**
1. Settings → Download Clients
2. Scroll to "Remote Path Mappings"
3. Click "+" to add mapping:
   - Host: `server1.ultra.cc` (EXACTLY as in download client)
   - Remote Path: `/home/johndoe/files/completed/tv/`
   - Local Path: `/data/seedbox-sync/tv/`
4. Save (checkmark icon)

**Test:**
1. Find a TV show
2. Interactive Search → Grab a release
3. Watch Activity → Queue
4. Should show:
   - Downloading on seedbox
   - Then "Completed - Waiting for Import"
   - Then "Imported"

#### 4. Radarr Configuration

**Add Download Client:**
Same as Sonarr, but use directory `movies`:
1. Settings → Download Clients → Add Download Client
2. Transmission
3. Configure:
   - Name: `Ultra.cc Transmission`
   - Host: `server1.ultra.cc`
   - Port: `19091`
   - Username: `johndoe`
   - Password: `[your Ultra.cc password]`
   - Directory: `movies`
4. Test and Save

**Configure Remote Path Mapping:**
1. Settings → Download Clients → Remote Path Mappings
2. Add mapping:
   - Host: `server1.ultra.cc`
   - Remote Path: `/home/johndoe/files/completed/movies/`
   - Local Path: `/data/seedbox-sync/movies/`
3. Save

### Complete Example Workflow

**User requests "The Matrix" in Jellyseerr:**

1. **Jellyseerr** → sends request to **Radarr**

2. **Radarr**:
   - Searches indexers via Jackett
   - Finds release: `The.Matrix.1999.1080p.BluRay.x264-GROUP.mkv`
   - Sends to `server1.ultra.cc:19091` with directory `movies`

3. **Ultra.cc Transmission**:
   - Receives torrent
   - Downloads to `/home/johndoe/files/completed/movies/`
   - File: `/home/johndoe/files/completed/movies/The.Matrix.1999.1080p.BluRay.x264-GROUP.mkv`

4. **Syncthing**:
   - Detects new file on seedbox
   - Transfers to Unraid: `/mnt/user/data/seedbox-sync/movies/The.Matrix.1999.1080p.BluRay.x264-GROUP.mkv`
   - Shows "Up to Date" when complete

5. **Radarr**:
   - Polls seedbox Transmission
   - Sees download complete at remote path: `/home/johndoe/files/completed/movies/The.Matrix.1999.1080p.BluRay.x264-GROUP.mkv`
   - Remote path mapping translates to local: `/data/seedbox-sync/movies/The.Matrix.1999.1080p.BluRay.x264-GROUP.mkv`
   - Finds file locally
   - Imports to: `/data/media/movies/The Matrix (1999)/The Matrix (1999).mkv`
   - Renames and organizes

6. **Jellyfin**:
   - Scans library
   - Finds new movie
   - Adds to library
   - Available for streaming

7. **Jellyseerr**:
   - Syncs with Jellyfin
   - Shows movie as available
   - Sends notification to requester

## Understanding Remote Path Mappings

This is the most confusing part of seedbox setups. Let's break it down.

### The Problem

Your seedbox and Unraid see the same file at different paths:

- **Seedbox sees:** `/home/johndoe/files/completed/movies/Movie.mkv`
- **Unraid sees:** `/mnt/user/data/seedbox-sync/movies/Movie.mkv`

Sonarr/Radarr need to know: "When the seedbox says the file is at PATH_A, I can find it locally at PATH_B"

### The Solution

Remote Path Mapping tells the *arr apps how to translate paths.

### Visual Example

```
┌─────────────────────────────────────────┐
│         Seedbox (Ultra.cc)              │
│  /home/johndoe/files/completed/movies/  │
│         Movie.mkv                       │
└────────────────┬────────────────────────┘
                 │
                 │ Syncthing
                 │ transfers
                 │
                 ▼
┌─────────────────────────────────────────┐
│              Unraid                     │
│    /mnt/user/data/seedbox-sync/movies/  │
│         Movie.mkv                       │
└─────────────────────────────────────────┘

Remote Path Mapping:
  Host: server1.ultra.cc
  Remote: /home/johndoe/files/completed/movies/
  Local:  /data/seedbox-sync/movies/
  
  (Remember: Docker sees /data/ as /mnt/user/data/seedbox-sync/)
```

### Common Mistakes

❌ **Wrong Host:**
```
Download Client Host: server1.ultra.cc
Mapping Host: 123.45.67.89

These don't match! Use same everywhere.
```

✅ **Correct:**
```
Download Client Host: server1.ultra.cc
Mapping Host: server1.ultra.cc
```

---

❌ **Wrong Paths:**
```
Seedbox actually saves to: /home/johndoe/files/completed/tv/
But you mapped: /home/johndoe/torrents/tv/

These don't match! Check seedbox client settings.
```

✅ **Correct:**
```
Check Transmission settings, it shows: /home/johndoe/files/completed/
Map that exact path.
```

---

❌ **Missing Trailing Slashes:**
```
Remote: /home/johndoe/files/completed/tv
Local: /data/seedbox-sync/tv/

Inconsistent slashes can cause issues.
```

✅ **Correct:**
```
Remote: /home/johndoe/files/completed/tv/
Local: /data/seedbox-sync/tv/

Use trailing slashes consistently.
```

### How to Find Your Paths

**Seedbox Remote Path:**
1. Download a test file
2. Check Transmission "Files" tab
3. Note the full path shown
4. That's your remote path

**Unraid Local Path:**
1. After file syncs via Syncthing
2. Check `/mnt/user/data/seedbox-sync/` on Unraid
3. Note where file appears
4. Convert to Docker path: `/data/seedbox-sync/...`

## Common Seedbox Providers

### Ultra.cc (Recommended)

**Pros:**
- Excellent support
- Pre-installed apps (Syncthing, multiple torrent clients)
- EU servers (Netherlands)
- Easy UCP interface
- Good for beginners

**Cons:**
- Slightly pricier than budget options

**Typical Setup:**
- Transmission or ruTorrent
- Syncthing for transfer
- Port forwarding available

### Whatbox

**Pros:**
- US and EU servers
- Good performance
- Lots of app choices
- Good community

**Cons:**
- Support less responsive than Ultra.cc

**Typical Paths:**
- Home: `/home/username/`
- Downloads: `/home/username/files/`

### Seedhost.eu

**Pros:**
- Very affordable
- EU servers
- Good for budget-conscious

**Cons:**
- Basic support
- Less hand-holding

**Typical Paths:**
- Home: `/home/username/`
- Downloads: `/home/username/downloads/`

### Feral Hosting

**Pros:**
- Unlimited bandwidth
- UK servers
- Good for heavy users

**Cons:**
- Complex setup
- Support tickets only

**Typical Paths:**
- Home: `/home/username/`
- Downloads: `/home/username/private/deluge/data/`

## Alternative Download Clients

### ruTorrent (Popular Alternative)

**Pros:**
- Very feature-rich
- Plugin ecosystem
- RSS capabilities

**Cons:**
- More complex
- Heavier resource usage

**Sonarr/Radarr Setup:**
- Client: rTorrent
- Host: `server1.ultra.cc`
- Port: Check UCP (usually 443 with /rutorrent/RPC2)
- URL Base: `/rutorrent/RPC2`
- Username/Password: Your credentials

**Remote Path Mapping:**
Similar to Transmission, but check ruTorrent settings for actual download path.

### Deluge

**Pros:**
- Good balance of features and simplicity
- Plugin support
- Lightweight

**Cons:**
- Setup can be tricky

**Sonarr/Radarr Setup:**
- Client: Deluge
- Host: `server1.ultra.cc`
- Port: Check UCP (usually 8112 for web)
- Password: Your deluge password

### qBittorrent

**Pros:**
- Very similar to Transmission
- Good web UI
- Category support

**Cons:**
- Categories work differently than Transmission

**Sonarr/Radarr Setup:**
- Client: qBittorrent
- Host: `server1.ultra.cc`
- Port: Check UCP
- Username/Password: Your credentials
- Category: `tv` or `movies`

## Tips for Success

### 1. Start Simple

- Use Transmission first (simplest setup)
- Get it working end-to-end
- Then experiment with other clients if desired

### 2. Document Your Setup

Create a text file with:
```
Seedbox: server1.ultra.cc
Username: johndoe
Transmission Port: 19091
Syncthing Port: 8384
Download Path: /home/johndoe/files/completed/
Local Sync Path: /mnt/user/data/seedbox-sync/
```

### 3. Test in Stages

1. Test seedbox → Can you download manually?
2. Test Syncthing → Do files transfer?
3. Test remote path mapping → Do imports work?
4. Test full workflow → Request → Download → Sync → Import

### 4. Monitor Initially

First few downloads:
- Watch Transmission on seedbox
- Watch Syncthing sync progress
- Watch Sonarr/Radarr activity queue
- Verify files land in Jellyfin

### 5. Set Up Notifications

In Sonarr/Radarr:
- Settings → Connect → Add Connection
- Choose Discord, Telegram, Email, etc.
- Get notified when:
  - Downloads grabbed
  - Downloads imported
  - Failures occur

## Maintenance

### Weekly

- Check Syncthing status (should always be "Up to Date")
- Review failed downloads in Activity → History
- Monitor seedbox disk space in UCP

### Monthly

- Clean up old torrents on seedbox (if not auto-deleting)
- Review Syncthing file versioning (if enabled)
- Check seedbox provider announcements

### As Needed

- Update Transmission/ruTorrent on seedbox
- Update Syncthing on both ends
- Adjust paths if seedbox provider changes structure

## Getting Help

**Seedbox Issues:**
- Contact your provider's support (Ultra.cc support is excellent)
- Check provider's documentation
- Look for provider-specific forums

**Syncthing Issues:**
- [Syncthing Forum](https://forum.syncthing.net/)
- Check logs on both sides
- Verify firewall not blocking

**Path Mapping Issues:**
- Check troubleshooting guide in this repo
- Use Interactive Search to see why files aren't found
- Compare paths character-by-character

---

**Questions?** Open an issue on this repo or contact your seedbox provider's support!
