# The Complete Guide to Setting Up Your *arr Stack on Unraid

If you're running Unraid as your home server OS, you've probably heard about the powerful *arr suite of applications. These tools work together to automate your media management, from finding and downloading content to organizing it perfectly for your media server. In this guide, I'll walk you through setting up a complete *arr stack that just works.

## What is the *arr Stack?

The *arr stack refers to a collection of applications that automate media management:

- **Sonarr** - Manages TV shows
- **Radarr** - Manages movies
- **Jackett** - Indexer proxy that translates queries for torrent trackers
- **Jellyseerr** - Request management and media discovery built specifically for Jellyfin
- **Profilarr** - Syncs and manages custom formats and quality profiles across *arr apps
- **Jellyfin** - Open-source media server for streaming your content

You'll also need a download client. This guide covers two options:
- **Local downloads**: Transmission with VPN on your Unraid server
- **Remote seedbox**: Using a seedbox in another country with Syncthing for automatic file transfers (I highly recommend Ultra.cc for their excellent service and support)

## Prerequisites

Before we begin, make sure you have:

- Unraid server up and running
- Community Applications plugin installed
- Basic understanding of Docker containers
- Sufficient storage space configured
- **Choose one:** 
  - A VPN solution if downloading locally (highly recommended)
  - OR a seedbox service in another country (for better privacy and speeds)

## Step 1: Planning Your Folder Structure

This is crucial and often overlooked. A proper folder structure prevents permission issues and makes everything work smoothly. Here's the structure I recommend:

**For Local Downloads:**
```
/mnt/user/data/
├── torrents/
│   ├── movies/
│   ├── tv/
│   └── music/
├── usenet/
│   ├── movies/
│   ├── tv/
│   └── music/
└── media/
    ├── movies/
    ├── tv/
    └── music/
```

**For Seedbox Setup with Syncthing:**
```
/mnt/user/data/
├── seedbox-sync/        # Syncthing receives files here
│   ├── movies/
│   ├── tv/
│   └── music/
└── media/               # Final organized media location
    ├── movies/
    ├── tv/
    └── music/
```

Create these folders in Unraid by going to **Shares** and creating a share called `data`. Then use the file browser or SSH to create the subdirectories.

**Important:** Don't use separate shares for downloads and media. Keep everything under one parent directory to avoid Docker mapping issues and enable hardlinks (which save space).

## Step 2A: Local Downloads - Install Transmission (Optional)

**Skip this step if you're using a seedbox with Syncthing (covered in Step 2B).**

1. Open **Apps** tab in Unraid
2. Search for "transmission"
3. I recommend the `linuxserver/transmission` container or `haugene/transmission-openvpn` if you want built-in VPN support
4. Configure the container:

```
Container Path: /data → Host Path: /mnt/user/data/
Container Path: /config → Host Path: /mnt/user/appdata/transmission/
```

5. Set your VPN credentials if using the VPN version
6. Click **Apply**

Once running, access Transmission at `http://[unraid-ip]:9091`

**Initial Setup:**
- Default login: Check your container's environment variables (usually admin/password or transmission/transmission)
- Web Interface Settings:
  - Go to Settings (wrench icon) → Downloading
  - Download to: `/data/torrents/`
  - Incomplete directory: `/data/torrents/incomplete/` (enable if desired)
- Network Settings:
  - Enable Port forwarding if using VPN
- Privacy:
  - Enable "Require encryption" for better security

**Important Notes:**
- Transmission uses a cleaner, simpler interface than some alternatives
- Works seamlessly with *arr apps
- Very reliable and lightweight
- If using VPN container, verify your VPN is connected before downloading

## Step 2B: Seedbox Setup - Install Syncthing (Recommended)

**Skip Step 2A if using this method.**

Using a seedbox in another country provides better privacy, speeds, and keeps torrenting off your home IP. We'll use Syncthing to automatically transfer completed downloads from your seedbox to Unraid.

**Choosing a Seedbox Provider:**

I personally use and highly recommend **Ultra.cc** for seedbox services. Here's why:
- Excellent customer support that actually responds quickly
- Reliable service with great uptime
- Competitive pricing with multiple plan options
- Easy-to-use control panel with one-click app installs
- Syncthing comes pre-installed and easy to configure
- Located in the Netherlands (EU) with great peering
- Support team is knowledgeable and helpful with setup questions

Other reputable providers include Whatbox, Seedhost, and Feral Hosting, but I've had the best experience with Ultra.cc's service and support.

**On Your Seedbox (Ultra.cc example):**

1. Log into your Ultra.cc control panel (UCP)
2. Navigate to "Apps" or "Applications"
3. Enable Syncthing (usually just a toggle switch)
4. Click on Syncthing to access the web interface
5. Note your Seedbox Device ID (Actions → Show ID)
6. Create a folder to share:
   - Most seedboxes have a default `/home/username/files/` or `/home/username/Downloads/` directory
   - On Ultra.cc, typically use `/home/username/files/completed/`
   - This is where your torrent client will save completed downloads

**On Unraid:**

1. Search for "syncthing" in Apps
2. Install `linuxserver/syncthing`
3. Configure:

```
Container Path: /config → Host Path: /mnt/user/appdata/syncthing/
Container Path: /data → Host Path: /mnt/user/data/seedbox-sync/
```

4. Access at `http://[unraid-ip]:8384`

**Configure Syncthing Connection:**

1. In Unraid Syncthing, click **Actions → Show ID** and copy your Device ID
2. In Seedbox Syncthing (Ultra.cc):
   - Add Remote Device (paste Unraid Device ID)
   - Name it "Unraid Server"
   - Save
3. In Unraid Syncthing:
   - Accept the connection request from your seedbox
4. Share the seedbox completed folder:
   - In Seedbox Syncthing, edit the shared folder
   - Under "Sharing," enable sharing with Unraid
5. In Unraid Syncthing:
   - Accept the folder share
   - Set folder path to `/data/` (which maps to `/mnt/user/data/seedbox-sync/` on Unraid)
   - Set folder type to "Receive Only" (important!)
   - Under "File Versioning," optionally enable "Trash Can" versioning

**Configure Your Seedbox's Download Client:**

On Ultra.cc (or your seedbox), configure your torrent client. Most seedboxes offer multiple options:
- **Transmission** (recommended - simple and reliable)
- **rTorrent/ruTorrent** (advanced, very popular)
- **Deluge** (good middle ground)

Configure completed downloads to save to the folder you're syncing with Unraid. On Ultra.cc with Transmission:
- Access Transmission web interface from UCP
- Settings → Downloading
- Set download directory to `/home/username/files/completed/`

Files will now automatically transfer to your Unraid server!

**Download Client Configuration for *arr Apps:**

You have two options:

**Option A - Direct seedbox connection (requires accessible ports):**
- Connect Sonarr/Radarr directly to your seedbox's torrent client
- Ultra.cc provides easy access to all client ports
- Best for immediate status updates

**Option B - Local monitoring (simpler):**
- Set up a "Remote Path Mapping" in Sonarr/Radarr
- Tell the *arr apps that downloads are in `/data/seedbox-sync/`
- They'll import files once Syncthing completes the transfer
- Slight delay but no port configuration needed

We'll configure Option A (direct connection with remote path mappings) in the Sonarr/Radarr steps below, which gives you the best of both worlds.

## Step 3: Install Jackett (Indexer Proxy)

Jackett acts as a proxy server that translates queries from your *arr apps into tracker-specific formats.

1. Search for "jackett" in Apps
2. Install `linuxserver/jackett`
3. Configure:

```
Container Path: /config → Host Path: /mnt/user/appdata/jackett/
Container Path: /downloads → Host Path: /mnt/user/data/torrents/
```

4. Access at `http://[unraid-ip]:9117`

**Configuration:**
- Click **Add Indexer**
- Search for your preferred indexers (1337x, RARBG, The Pirate Bay, etc.)
- For each indexer, click the wrench icon to configure
- Test the indexer and save
- Copy the Torznab Feed URL (you'll need this for Sonarr/Radarr)

**Important:** Each indexer you add will have its own Torznab feed URL and API key. You'll add these individually to Sonarr and Radarr.

## Step 4: Install Sonarr (TV Shows)

1. Search for "sonarr" in Apps
2. Install `linuxserver/sonarr`
3. Configure:

```
Container Path: /config → Host Path: /mnt/user/appdata/sonarr/
Container Path: /data → Host Path: /mnt/user/data/
```

4. Access at `http://[unraid-ip]:8989`

**Configuration:**

**Media Management:**
- Settings → Media Management
- Enable "Rename Episodes"
- Set your preferred naming format (I use `{Series Title} - S{season:00}E{episode:00} - {Episode Title}`)
- Enable "Unmonitor Deleted Episodes"

**Root Folders:**
- Settings → Media Management → Root Folders
- Add: `/data/media/tv`

**Download Clients:**

**For Local Transmission:**
- Settings → Download Clients → Add Download Client
- Select Transmission
- Host: Your Unraid IP
- Port: 9091
- Username/Password: Your Transmission credentials
- Category: `tv` (Transmission will create subdirectory)
- Test and Save

**For Seedbox Setup:**
- Settings → Download Clients → Add Download Client
- Select your seedbox's client type (Transmission, Deluge, rTorrent, etc.)
- **For Ultra.cc with Transmission:**
  - Name: Ultra.cc Transmission
  - Host: Your Ultra.cc server hostname (e.g., `yourserver.ultra.cc`)
  - Port: Check Ultra.cc UCP for your Transmission port (usually unique per user)
  - Username: Your Ultra.cc username
  - Password: Your Ultra.cc password
  - Directory: Leave blank or set to `tv` (optional)
- **Do NOT configure Remote Path Mappings here** (we'll do this separately)
- Test and Save

**Configure Remote Path Mappings for Seedbox:**

This is crucial for seedbox setups - it tells Sonarr where to find files locally after Syncthing transfers them.

- Settings → Download Clients
- Scroll to the bottom section "Remote Path Mappings"
- Click the "+" button to add a new mapping
- Configure as follows:
  - **Host:** Your seedbox hostname (must match exactly what you used in download client)
  - **For Ultra.cc example:** `yourserver.ultra.cc`
  - **Remote Path:** The full path on your seedbox where files are saved
    - Example for Ultra.cc: `/home/yourusername/files/completed/tv/`
    - This is what your seedbox's download client reports
    - Check your seedbox's Transmission settings to confirm the exact path
  - **Local Path:** Where Syncthing syncs those files on Unraid
    - Example: `/data/seedbox-sync/tv/`
    - This must match your Syncthing folder configuration
- Click the checkmark to save

**How it works:**
1. Sonarr sends download to seedbox: "Save to /home/yourusername/files/completed/tv/"
2. Seedbox downloads the file
3. Syncthing transfers file to Unraid at `/data/seedbox-sync/tv/`
4. Sonarr checks download status, sees path `/home/yourusername/files/completed/tv/Show.Name.S01E01.mkv`
5. Remote path mapping translates this to `/data/seedbox-sync/tv/Show.Name.S01E01.mkv`
6. Sonarr finds the file locally and imports it

**Testing Your Mapping:**
- System → Status → More Info
- Look for "Remote Path Mappings" section
- Should show your configured mapping
- Grab a test episode and watch the Activity queue to ensure imports work

**Add Indexers from Jackett:**
- Settings → Indexers → Add Indexer → Custom → Torznab
- For each indexer in Jackett:
  - Name: Give it a descriptive name (e.g., "1337x")
  - URL: Copy from Jackett (click "Copy Torznab Feed" button)
  - API Key: Copy from Jackett (top right corner)
  - Categories: 5030, 5040 (TV categories)
  - Test and Save
- Repeat for each indexer you want to use

## Step 5: Install Radarr (Movies)

The process is nearly identical to Sonarr:

1. Install `linuxserver/radarr`
2. Configure:

```
Container Path: /config → Host Path: /mnt/user/appdata/radarr/
Container Path: /data → Host Path: /mnt/user/data/
```

3. Access at `http://[unraid-ip]:7878`

**Configuration:**
- Set up media management and renaming (similar to Sonarr)
- Add Root Folder: `/data/media/movies`
- Add download client:
  - **For Local Transmission:**
    - Select Transmission
    - Host: Your Unraid IP
    - Port: 9091
    - Username/Password: Your Transmission credentials
    - Category: `movies`
    - Test and Save
  - **For Seedbox (Ultra.cc example):**
    - Select Transmission (or your seedbox's client)
    - Host: `yourserver.ultra.cc`
    - Port: Your Ultra.cc Transmission port (check UCP)
    - Username: Your Ultra.cc username
    - Password: Your Ultra.cc password
    - Directory: Leave blank or `movies`
    - Test and Save
    
**Configure Remote Path Mappings for Seedbox:**

If using a seedbox, you must configure remote path mappings in Radarr as well:

- Settings → Download Clients
- Scroll to the bottom section "Remote Path Mappings"
- Click the "+" button to add a new mapping
- Configure as follows:
  - **Host:** Your seedbox hostname (must match download client exactly)
  - **For Ultra.cc:** `yourserver.ultra.cc`
  - **Remote Path:** The full path on your seedbox where movies are saved
    - Example for Ultra.cc: `/home/yourusername/files/completed/movies/`
    - Verify this path in your seedbox's download client settings
  - **Local Path:** Where Syncthing syncs those files on Unraid
    - Example: `/data/seedbox-sync/movies/`
    - Must match your Syncthing configuration
- Click the checkmark to save

**Important Notes:**
- The Host field must match EXACTLY between download client and remote path mapping
- If your seedbox hostname is `server1.ultra.cc`, use that everywhere (not the IP)
- Or if you used the IP, use that everywhere consistently
- Remote paths must include trailing slashes: `/home/user/files/` not `/home/user/files`
- Test with a movie to verify the mapping works correctly

- Add indexers from Jackett:
  - Settings → Indexers → Add Indexer → Custom → Torznab
  - Use the same process as Sonarr
  - Categories: 2000 (Movies category)
  - Add each indexer you want to use

## Step 6: Install Jellyseerr (Request Management)

Jellyseerr is a fork of Overseerr built specifically for Jellyfin, providing a beautiful interface for requesting movies and TV shows, with user management and approval workflows.

1. Install `fallenbagel/jellyseerr`
2. Configure:

```
Container Path: /app/config → Host Path: /mnt/user/appdata/jellyseerr/
```

3. Access at `http://[unraid-ip]:5055`

**Initial Setup:**
- On first launch, you'll go through a setup wizard
- Sign in with your Jellyfin account:
  - Jellyfin URL: `http://[unraid-ip]:8096`
  - Enter your Jellyfin credentials
  - Select which libraries to sync
- Connect Sonarr:
  - Name: Sonarr
  - Hostname/IP: Your Unraid IP
  - Port: 8989
  - API Key: From Sonarr Settings → General
  - Base URL: Leave blank (unless using reverse proxy)
  - Test and Save
- Connect Radarr:
  - Same process with port 7878 and Radarr API key

**User Management:**
- Users can import their Jellyfin accounts into Jellyseerr
- Request content through the beautiful web interface
- Set up approval requirements for different users
- Email/Discord/Telegram notifications for requests and availability
- Browse trending content with trailers and ratings from TMDB

**Configuration Tips:**
- Settings → Users → Import Jellyfin users
- Settings → Users → Set default permissions for new users
- Settings → Notifications → Configure Discord/Email/Telegram/Slack notifications
- Settings → General → Enable auto-approve for trusted users
- Settings → General → Configure request limits (daily/weekly/monthly)

**Why Jellyseerr over Overseerr?**
- Native Jellyfin integration (Overseerr is Plex-focused)
- Better library syncing with Jellyfin
- Jellyfin authentication support
- Same beautiful interface and features

## Step 7: Install Jellyfin (Media Server)

Jellyfin is a free, open-source media server that lets you stream your organized media to any device.

1. Install `linuxserver/jellyfin`
2. Configure:

```
Container Path: /config → Host Path: /mnt/user/appdata/jellyfin/
Container Path: /data → Host Path: /mnt/user/data/media/
```

3. Access at `http://[unraid-ip]:8096`

**Initial Setup:**
- Create your admin account
- Add media libraries:
  - Movies: `/data/movies`
  - TV Shows: `/data/tv`
  - Music: `/data/music` (if applicable)
- Configure metadata providers (TMDB, TheTVDB, etc.)
- Set up hardware acceleration if your Unraid server supports it:
  - Dashboard → Playback → Transcoding
  - Enable hardware acceleration (Intel QuickSync, NVIDIA NVENC, or AMD AMF)

**Additional Configuration:**
- Dashboard → Networking → Enable remote access (if desired)
- Dashboard → Users → Create accounts for family/friends
- Dashboard → Plugins → Install useful plugins:
  - TMDb Box Sets (for movie collections)
  - Playback Reporting (watch statistics)
  - Reports (detailed library reports)

**Jellyfin Apps:**
Jellyfin has apps for nearly every platform:
- Web browser (best experience)
- Android/iOS
- Android TV/Fire TV/Apple TV/Roku
- Windows/Mac/Linux desktop apps
- Smart TVs (Samsung, LG, etc.)

**Why Jellyfin?**
- Completely free and open-source (no premium features locked behind paywall)
- No tracking or phone-home
- Works perfectly with Jellyseerr
- Active development community
- Full control over your media server

## Step 8: Install Profilarr (Profile Manager)

Profilarr helps you sync custom formats, quality profiles, and naming schemes across multiple Sonarr/Radarr instances, or automatically apply TRaSH Guides recommendations.

1. Search for "profilarr" in Apps (or install via Docker with `grostim/profilarr`)
2. Configure:

```
Container Path: /config → Host Path: /mnt/user/appdata/profilarr/
```

3. Access at `http://[unraid-ip]:5500`

**Configuration:**

**Add Your Apps:**
- Navigate to Connections
- Add Radarr:
  - Name: Radarr
  - URL: `http://[unraid-ip]:7878`
  - API Key: From Radarr
  - Test and Save
- Add Sonarr with the same process

**Sync Quality Profiles:**
- Go to Quality Profiles
- Create or import profiles based on TRaSH Guides
- Select which apps should receive each profile
- Click Sync to apply

**Custom Formats:**
- Profilarr can automatically import and sync custom formats
- Navigate to Custom Formats section
- Import from TRaSH Guides (recommended)
- Common formats include:
  - Proper/Repack detection
  - Release group preferences
  - Audio codec preferences (DTS, TrueHD, etc.)
  - HDR formats
  - Unwanted content filters

**Benefits:**
- Maintain consistent quality across apps
- Easy updates when TRaSH Guides change
- Manage multiple instances easily
- Apply best practices without manual configuration

## Seedbox Configuration Example (Complete Walkthrough)

To help clarify the seedbox setup, here's a complete example with real paths:

**Scenario:** Using Ultra.cc seedbox with Transmission

**Seedbox Details:**
- Hostname: `server1.ultra.cc`
- Username: `johndoe`
- Transmission Port: `19091` (check your UCP for your specific port)
- Completed downloads path: `/home/johndoe/files/completed/`

**Step 1: Configure Seedbox Transmission (via Ultra.cc UCP)**
- Enable Transmission in UCP if not already enabled
- Access Transmission web interface from UCP
- Settings → Downloading
- Set download directory to: `/home/johndoe/files/completed/`
- Create subdirectories for organization:
  - `/home/johndoe/files/completed/tv/`
  - `/home/johndoe/files/completed/movies/`

**Step 2: Configure Syncthing**
- Enable Syncthing in Ultra.cc UCP
- Seedbox shares folder: `/home/johndoe/files/completed/`
- Unraid receives to: `/mnt/user/data/seedbox-sync/`
- Set folder to "Receive Only" on Unraid
- Verify both devices show as "Connected" and "Up to Date"

**Step 3: Configure Sonarr Download Client**
- Name: Ultra.cc Transmission
- Host: `server1.ultra.cc`
- Port: `19091`
- Username: `johndoe`
- Password: `[your Ultra.cc password]`
- Directory: `tv` (optional - Transmission will create subdirectory)
- Save (don't configure remote path here)

**Step 4: Configure Sonarr Remote Path Mapping**
- Settings → Download Clients → Remote Path Mappings → Add
- Host: `server1.ultra.cc` (EXACTLY as entered in download client)
- Remote Path: `/home/johndoe/files/completed/tv/`
- Local Path: `/data/seedbox-sync/tv/`
- Save

**Step 5: Configure Radarr (Same Process)**
- Download Client: Same as Sonarr but already configured
- Remote Path Mapping:
  - Host: `server1.ultra.cc`
  - Remote Path: `/home/johndoe/files/completed/movies/`
  - Local Path: `/data/seedbox-sync/movies/`

**What Happens When You Request a Movie:**
1. User requests "The Matrix" in Jellyseerr
2. Radarr searches Jackett indexers, finds release
3. Radarr sends to `server1.ultra.cc:19091` (Transmission) with directory `movies`
4. Ultra.cc Transmission saves to `/home/johndoe/files/completed/movies/The.Matrix.1999.1080p.mkv`
5. Syncthing detects new file and transfers to `/mnt/user/data/seedbox-sync/movies/The.Matrix.1999.1080p.mkv`
6. Radarr polls Ultra.cc, sees file completed at remote path
7. Remote path mapping translates to local path `/data/seedbox-sync/movies/The.Matrix.1999.1080p.mkv`
8. Radarr finds file locally, imports to `/data/media/movies/The Matrix (1999)/The Matrix (1999).mkv`
9. Jellyfin scans and movie appears in library

**Why This Setup Works Great:**
- Ultra.cc handles all the torrenting with their fast servers and great peering
- Syncthing automatically transfers completed files to your Unraid server
- Remote path mapping ensures Sonarr/Radarr can track and import files correctly
- Your home IP never touches any torrent swarms
- Ultra.cc support can help if you run into any issues with their services

## Quality Profiles and Optimization

### Setting Up Quality Profiles

In both Sonarr and Radarr:
1. Settings → Profiles
2. Edit or create quality profiles based on your preferences
3. I recommend having profiles like:
   - **1080p**: For most content
   - **4K**: Only if you have the storage and bandwidth
   - **Any**: Accepts whatever's available

### Custom Formats (Advanced)

Radarr and Sonarr support custom formats to fine-tune what you download. For example:
- Prefer releases with proper audio codecs
- Avoid releases with hardcoded subtitles
- Prefer specific release groups

This is covered in detail in TRaSH Guides (worth looking up).

## Common Issues and Solutions

**Problem: Downloads complete but don't import**
- Check folder permissions. Run: `chmod -R 777 /mnt/user/data/` (or be more specific)
- Ensure PUID and PGID are set correctly in Docker containers (usually 99:100 for Unraid)
- For seedbox setup: Verify remote path mappings are correct in Sonarr/Radarr

**Problem: Hardlinks not working (duplicate files)**
- Verify downloads and media are under the same parent directory (`/data/`)
- Don't use separate Docker volumes for downloads and media
- Note: Hardlinks won't work with seedbox setups as files are copied from remote server

**Problem: Can't connect apps together**
- Use your Unraid server's IP, not `localhost` or `127.0.0.1`
- Check that container ports aren't conflicting
- Verify API keys are correct

**Problem: Torrents not starting**
- **Local:** Check Transmission is running and VPN is connected if using VPN container
- **Local:** Verify credentials are correct
- **Seedbox:** Check your seedbox's Transmission is running (check Ultra.cc UCP status)
- **Seedbox:** Verify you can connect to seedbox from Unraid
- Check indexer status in Jackett

**Problem: Jackett indexers not working in Sonarr/Radarr**
- Verify the Torznab URL is correct (copy directly from Jackett)
- Check API key matches
- Make sure correct categories are set (5030,5040 for TV; 2000 for movies)
- Test the indexer in Jackett first before adding to *arr apps

**Problem: Jellyseerr not finding media**
- Verify Jellyfin library paths match your actual media paths
- Sync libraries in Jellyseerr settings
- Check that Sonarr/Radarr root folders are correctly configured
- Ensure Jellyfin has finished scanning the libraries

**Problem: Syncthing not transferring files from seedbox**
- Check both Syncthing web interfaces show "Up to Date"
- Verify folder paths are correct on both ends
- Check that devices are connected (green checkmark)
- Ensure seedbox folder is set to "Send Only" and Unraid folder is "Receive Only"
- Check firewall rules aren't blocking Syncthing ports (22000 TCP/UDP)
- Review Syncthing logs for errors

**Problem: Files transfer from seedbox but *arr apps don't import**
- **Most common cause:** Incorrect remote path mappings
- Verify remote path mappings match your actual seedbox paths EXACTLY
- Check Host field matches between download client and remote path mapping (use IP everywhere or hostname everywhere, not mixed)
- Ensure paths include or exclude trailing slashes consistently
- Common path issues:
  - Seedbox uses `/home/user/torrents/complete/` but you mapped `/home/user/files/`
  - Download client shows path with category: `/rutorrent/downloads/tv/` 
  - Syncthing folder doesn't match local path in mapping
- **How to diagnose:**
  - Go to Activity → Queue in Sonarr/Radarr
  - Look at a completed download
  - Note the "Output Path" shown
  - That's the remote path - it should match your mapping
  - If not, update your remote path mapping
- Check that files have finished syncing (Syncthing shows "Up to Date")
- Ensure file permissions are correct after sync: `chmod -R 777 /mnt/user/data/seedbox-sync/`
- Look at Sonarr/Radarr logs (System → Logs) for import errors
- Test the remote path mapping: System → Status → More Info → Remote Path Mappings

## Security Considerations

1. **Use a VPN or Seedbox** 
   - **Local downloads:** VPN is essential if torrenting copyrighted content
   - **Seedbox:** Keeps torrenting off your home IP entirely (recommended)
2. **Set up authentication** - Enable auth on all *arr apps and Jellyseerr
3. **Use reverse proxy** - Consider setting up nginx or Traefik for HTTPS access
4. **Secure Syncthing** 
   - Use device authentication (don't use introducer mode)
   - Consider using "Untrusted" encryption for extra security
   - Don't expose Syncthing ports to the internet unnecessarily
5. **Keep apps updated** - Unraid makes this easy with update notifications
6. **Regular backups** - Backup your appdata folder regularly
7. **Jellyfin security**
   - Use strong passwords for all accounts
   - Enable two-factor authentication if exposing externally
   - Review login attempts regularly

## Maintenance Tips

- **Update regularly**: Check for updates weekly
- **Monitor disk space**: Set up disk space alerts (especially important with seedbox transfers)
- **Clean up failed downloads**: 
  - Check your download client periodically
  - For seedbox: Monitor disk space on both seedbox and Unraid
- **Review wanted lists**: Sonarr/Radarr can accumulate a lot of wanted items
- **Backup configs**: Your appdata folder contains all configs - back it up!
- **Syncthing maintenance** (if using seedbox):
  - Check sync status weekly
  - Review file versioning to prevent bloat
  - Monitor seedbox disk space to prevent sync failures
  - Periodically verify files synced correctly
- **Jellyfin maintenance**:
  - Schedule library scans appropriately
  - Clean up old transcoding temp files
  - Review user activity for issues
  - Update plugins regularly

## Conclusion

With your *arr stack properly configured, you now have a powerful, automated media management system. New episodes and movies will be automatically downloaded (either locally or via your seedbox), synced to your Unraid server, renamed, and organized without any manual intervention. Jellyseerr provides a beautiful interface for users to request content, while Profilarr ensures your quality profiles and custom formats stay consistent and optimized across all your apps. Jellyfin gives you a completely open-source streaming solution with apps for every platform.

The initial setup takes some time, but once it's running, it's incredibly hands-off. Take the time to configure quality profiles through Profilarr, set up your indexers in Jackett properly, configure Jellyseerr permissions to match your needs, and if using a seedbox, ensure Syncthing is properly configured for automatic transfers. After that, you'll rarely need to touch it again.

**Quick Reference - Default Ports:**
- Transmission: 9091 (if using local downloads)
- Syncthing: 8384 (if using seedbox)
- Jackett: 9117
- Sonarr: 8989
- Radarr: 7878
- Jellyseerr: 5055
- Profilarr: 5500
- Jellyfin: 8096

**Workflow Summary:**

*With Local Downloads:*
User requests in Jellyseerr → Sonarr/Radarr searches Jackett → Sends to Transmission → Downloads via VPN → Imports to media folder → Appears in Jellyfin

*With Seedbox (Ultra.cc):*
User requests in Jellyseerr → Sonarr/Radarr searches Jackett → Sends to Ultra.cc Transmission → Downloads on Ultra.cc → Syncthing transfers to Unraid → Imports to media folder → Appears in Jellyfin

Happy streaming!

---

*Have questions or running into issues? The Unraid forums and r/unraid on Reddit are excellent resources for troubleshooting.*
