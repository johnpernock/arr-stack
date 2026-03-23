# The Complete Guide to Quality Profiles and Release Filtering in Radarr & Sonarr

Stop downloading garbage! This guide will teach you how to configure quality profiles and custom formats to ensure you only get high-quality releases worth watching.

## Table of Contents

- [Introduction](#introduction)
- [Understanding Quality Profiles](#understanding-quality-profiles)
- [Setting Up Basic Quality Profiles](#setting-up-basic-quality-profiles)
- [Custom Formats: The Power Tool](#custom-formats-the-power-tool)
- [Filtering Out Bad Releases](#filtering-out-bad-releases)
- [Advanced Scoring and Prioritization](#advanced-scoring-and-prioritization)
- [TRaSH Guides Integration](#trash-guides-integration)
- [Complete Example Configuration](#complete-example-configuration)
- [Testing Your Configuration](#testing-your-configuration)
- [Troubleshooting](#troubleshooting)
- [Progressive Implementation Strategy](#progressive-implementation-strategy)

## Introduction

You've set up your *arr stack, content is downloading, but you're getting:
- Fake "1080p" files that are only 1GB
- Releases with hardcoded subtitles
- Poor quality encodes from bad release groups
- Bloated 50GB files for a 2-hour movie
- CAM/TS/Screener quality garbage

**This guide will fix all of that.**

### What We'll Cover

1. **Quality Profiles** - Define what resolutions and sources you accept
2. **Size Limits** - The first line of defense against fake releases
3. **Custom Formats** - Advanced filtering based on release characteristics
4. **Scoring System** - Prioritize releases you want and block what you don't
5. **TRaSH Guides** - Leverage community wisdom via Profilarr

### The Three-Layer Defense System

Think of quality control as three layers:

1. **Quality Definitions (Size Limits)** - Catches most garbage automatically
2. **Custom Formats (Characteristics)** - Fine-tunes based on codec, groups, audio, etc.
3. **Scoring System (Prioritization)** - Ensures you get the best available release

## Understanding Quality Profiles

### What is a Quality Profile?

A quality profile defines:
- Which quality levels you'll accept (720p, 1080p, 4K, etc.)
- Size limits for each quality (minimum and maximum file size)
- What quality to "cutoff" at (stop upgrading)
- How to score and prioritize releases

### The Quality Hierarchy

Radarr and Sonarr understand quality in this order (lowest to highest):

**Source Quality:**
- WORKPRINT (pre-release)
- CAM (camera recording in theater)
- TELESYNC (TS)
- TELECINE (TC)
- REGIONAL (regional release)
- SCREENER (DVD screener)
- DVDR (full DVD)
- DVD
- WEBDL-480p
- Bluray-480p
- WEBDL-720p
- Bluray-720p
- WEBDL-1080p
- Bluray-1080p
- WEBDL-2160p (4K streaming)
- Bluray-2160p (4K disc)
- Remux-1080p (lossless Bluray rip)
- Remux-2160p (lossless 4K rip)

**General Rule:**
- **Bluray** > **WEBDL** (for same resolution)
- **Remux** > **Bluray encode** (remux is uncompressed)
- **Higher resolution** > **Lower resolution** (usually)

## Setting Up Basic Quality Profiles

Let's create quality profiles that work for most people.

### Profile 1: HD Standard (Most Popular)

**Use Case:** Good quality without huge files

**Configuration:**

1. Settings → Profiles → Add Profile
2. Name: `HD Standard`
3. **Allowed Qualities** (in order of preference):
   - Bluray-1080p ✓
   - WEBDL-1080p ✓
   - Bluray-720p ✓ (fallback)
   - WEBDL-720p ✓ (fallback)
4. **Upgrade Until:** Bluray-1080p
5. **Minimum Custom Format Score:** 0 (we'll configure this later)

### Profile 2: 4K/UHD Quality

**Use Case:** For your 4K TV and favorite movies

**Configuration:**

1. Name: `4K Quality`
2. **Allowed Qualities:**
   - Bluray-2160p ✓
   - WEBDL-2160p ✓
   - Remux-2160p ✓ (if you have the space)
3. **Upgrade Until:** Bluray-2160p (or Remux-2160p if you want perfection)
4. **Minimum Custom Format Score:** 0

### Profile 3: Any HD (Fallback)

**Use Case:** For shows/movies you just want to watch, quality less critical

**Configuration:**

1. Name: `Any HD`
2. **Allowed Qualities:**
   - Bluray-1080p ✓
   - WEBDL-1080p ✓
   - Bluray-720p ✓
   - WEBDL-720p ✓
3. **Upgrade Until:** WEBDL-1080p
4. **Minimum Custom Format Score:** -10000 (accepts anything not explicitly blocked)

## Quality Size Limits: Your First Defense

**This is the most important section.** Size limits prevent 90% of garbage releases.

### Understanding MB/minute

File size is measured in megabytes per minute of runtime:
- 90-minute movie at 50 MB/min = 4.5 GB minimum
- 120-minute movie at 50 MB/min = 6 GB minimum

### Recommended Size Limits

| Quality | Minimum (MB/min) | Preferred (MB/min) | Maximum (MB/min) | Notes |
|---------|------------------|--------------------|--------------------|-------|
| **Bluray-720p** | 20 | 30-50 | 150 | Good for older content |
| **WEBDL-720p** | 10 | 15-30 | 100 | Streaming sources |
| **Bluray-1080p** | 40 | 50-100 | 250 | Sweet spot for quality |
| **WEBDL-1080p** | 15 | 20-40 | 150 | Netflix, Amazon, etc. |
| **Bluray-2160p** | 80 | 100-200 | 400 | 4K from disc |
| **WEBDL-2160p** | 40 | 50-100 | 300 | 4K streaming |
| **Remux-1080p** | 100 | 150-250 | 400 | Uncompressed |
| **Remux-2160p** | 200 | 250-400 | 600 | Uncompressed 4K |

### How to Configure Size Limits

In Radarr/Sonarr:

1. Settings → Quality
2. Click on a quality (e.g., "Bluray-1080p")
3. Set limits:
   - **Minimum:** Multiply runtime by minimum MB/min
   - **Maximum:** Multiply runtime by maximum MB/min
4. For Radarr (movies vary in length):
   - Use the "Preferred" tab
   - Set min: 50 MB/min, max: 250 MB/min for Bluray-1080p

**Example for a 120-minute movie with Bluray-1080p:**
- Minimum: 120 × 50 = 6,000 MB (6 GB)
- Maximum: 120 × 250 = 30,000 MB (30 GB)

This blocks:
- ❌ 2GB "1080p" fake releases
- ❌ 100GB bloated releases
- ✅ 8-20GB high quality encodes

## Custom Formats: The Power Tool

Custom formats let you filter releases based on characteristics like:
- Release group name
- Codec used (x264, x265, AV1)
- Audio quality (DTS-HD, Atmos, AAC)
- Special tags (REPACK, PROPER, Remux)
- Hardcoded subtitles
- Source (which streaming service)

### Creating a Custom Format

Settings → Custom Formats → Add Custom Format

**Each custom format needs:**
1. **Name** - Descriptive label
2. **Conditions** - Regex patterns or tags that match
3. **Score** - Points assigned when matched

### Example: Blocking Bad Release Groups

**Name:** `LQ Release Groups`

**Conditions:**
- Release Title matching regex:
  ```regex
  \b(YIFY|YTS|RARBG|PSA|FGT|STUTTERSHIT|MeGusta|HD2DVD|HDTime|1XBET|4K4U|AKBD)\b
  ```

**Score:** `-10000` (in your quality profile)

**What this does:** Any release from these groups gets -10000 points and will NEVER download.

### Example: Preferring Good Release Groups

**Name:** `Trusted Release Groups`

**Conditions:**
- Release Title matching regex:
  ```regex
  \b(FraMeSToR|HQMUX|DON|CtrlHD|BMF|KRaLiMaRKo|PTer|HiFi|FLUX|CtrlHD|W4F|NTb|WELP)\b
  ```

**Score:** `+100` (in your quality profile)

**What this does:** Releases from trusted groups get +100 points and are strongly preferred.

### Example: Blocking Hardcoded Subtitles

**Name:** `Hardcoded Subs`

**Conditions:**
- Release Title matching regex:
  ```regex
  \b(HC|HCOR|HARDCODED|SUBBED|HQ\.CAM|HD\.CAM)\b
  ```

**Score:** `-10000`

**What this does:** Blocks releases with burned-in subtitles you can't turn off.

### Example: Blocking CAM/TS/Screeners

**Name:** `CAM/TS/Screener`

**Conditions:**
- Release Title matching regex:
  ```regex
  \b(CAM|CAMRIP|TS|TELESYNC|HDTS|TELECINE|HDTC|SCREENER|DVDSCR|DVDSCREENER)\b
  ```

**Score:** `-10000`

**What this does:** Blocks terrible theater recordings and screeners.

### Example: Blocking BR-DISK

**Name:** `BR-DISK`

**Conditions:**
- Release Title matching regex:
  ```regex
  \b(COMPLETE\.BLURAY|UNTOUCHED\.BLURAY|AVC|VC-1)\b
  ```

**Score:** `-10000`

**What this does:** Blocks full Bluray disc images (40-50GB, you probably don't want these).

### Example: Preferring x265/HEVC

**Name:** `x265/HEVC`

**Conditions:**
- Release Title matching regex:
  ```regex
  \b(x265|HEVC|h\.?265)\b
  ```

**Score:** `+25`

**What this does:** Gives small boost to more efficient x265 encodes (smaller files, same quality).

**Note:** Some people prefer x264 for compatibility. Adjust score to taste or set to 0.

### Example: Preferring High-Quality Audio

**Name:** `High Quality Audio`

**Conditions:**
- Release Title matching regex:
  ```regex
  \b(DTS.?HD|DTS.?X|DTSHD|TrueHD|TRUEHD|Atmos|ATMOS|DDP.?7\.1|DD.?EX|FLAC)\b
  ```

**Score:** `+50`

**What this does:** Prefers releases with lossless or high-quality audio.

### Example: Dolby Vision for 4K

**Name:** `Dolby Vision`

**Conditions:**
- Release Title matching regex:
  ```regex
  \b(DV|DoVi|Dolby.?Vision)\b
  ```

**Score:** `+100`

**What this does:** Strongly prefers Dolby Vision HDR for 4K content (if your TV supports it).

### Example: Preferring Remux

**Name:** `Remux`

**Conditions:**
- Release Title matching regex:
  ```regex
  \b(REMUX)\b
  ```

**Score:** `+200`

**What this does:** Gives highest preference to remux (lossless) releases if you have the storage.

## Advanced Scoring and Prioritization

### Understanding the Scoring System

When Sonarr/Radarr finds releases, it calculates a score for each:
```
Total Score = Quality Score + Custom Format Scores
```

**Quality Score:**
- Each quality has an inherent score (Bluray-1080p > WEBDL-1080p > 720p, etc.)
- This is automatic based on quality hierarchy

**Custom Format Scores:**
- Sum of all matching custom format scores
- Can be positive (preferred) or negative (blocked)

**Example Calculation:**
```
Release: "Movie.2024.1080p.BluRay.x265.DTS-HD.FraMeSToR"

Quality: Bluray-1080p (inherent quality score, let's say 100)
Custom Formats:
  + Trusted Groups (FraMeSToR): +100
  + x265/HEVC: +25
  + High Quality Audio (DTS-HD): +50
  
Total Score: 100 + 100 + 25 + 50 = 275
```

vs.

```
Release: "Movie.2024.1080p.BluRay.YIFY"

Quality: Bluray-1080p: 100
Custom Formats:
  + LQ Release Groups (YIFY): -10000
  
Total Score: 100 + (-10000) = -9900

This release is BLOCKED (negative score)
```

### The Cutoff Score

In your quality profile, you set a "Minimum Custom Format Score":
- **0** (default) - Downloads anything not blocked
- **+50** - Only downloads releases scoring 50+ 
- **+100** - Only downloads releases scoring 100+
- **+200** - Very picky, only excellent releases

**Recommendation:**
- Start with `0` to see what's available
- Increase to `+50` or `+100` once you have custom formats configured
- Use `+200` only for favorite movies/shows where you want perfection

### Upgrade Until Quality

Combined with custom format scores, this controls when upgrading stops:

**Scenario 1: Upgrade Based on Quality**
- Upgrade Until: Bluray-1080p
- Downloads WEB-1080p first (available now)
- Automatically upgrades to Bluray-1080p when available

**Scenario 2: Upgrade Based on Score**
- Upgrade Until: Bluray-1080p
- Upgrade Until Custom Score: +100
- Downloads any 1080p with score 0+
- Keeps upgrading until it finds Bluray-1080p with score 100+

### Scoring Philosophy

**Negative Scores (Blocking):**
- Use `-10000` for absolute blocks (bad groups, CAM, hardcoded subs)
- Use `-5000` for strong blocks (BR-DISK if you don't want them)
- Never use small negative scores like `-10`, they don't block effectively

**Positive Scores (Preference):**
- `+5 to +25`: Minor preferences (x265 vs x264)
- `+50 to +100`: Moderate preferences (good audio, trusted groups)
- `+100 to +200`: Strong preferences (Dolby Vision, Remux, top-tier groups)
- `+200+`: Extreme preferences (only if you really mean it)

**Tiered Scoring Example:**
```
Critical Blocks:
  - LQ Groups: -10000
  - CAM/TS: -10000
  - Hardcoded Subs: -10000
  - BR-DISK: -10000

Strong Preferences:
  - Remux: +200
  - Dolby Vision: +100
  - Trusted Groups: +100

Moderate Preferences:
  - High Quality Audio: +50
  - x265/HEVC: +25

Fine Tuning:
  - Repack/Proper: +50
  - Specific streaming service: +10
```

## Filtering Out Bad Releases

Let's put it all together with a complete blocking strategy.

### Must-Have Blocks

These should be in **every** quality profile:

#### 1. LQ Release Groups

Blocks known bad encoders:
```regex
\b(YIFY|YTS|YTS\.MX|RARBG|PSA|FGT|STUTTERSHIT|MeGusta|HD2DVD|HDTime|1XBET|4K4U|AKBD|ALiGN|CAPTCHA|EDGE2020|EVO|eXceSs|FUM|HDGO|HIVE|iNTENSo|JAV|JZVIDEO|LAMA|MKVTV|nhanc3|NOGRP|PiRaTeS|PROTON|QAiZER|RetroBOX|ROBOTS|SALTY|SPASM|TBS|TEPES|TVSmash|VOLTAGE|W4F\.TO|WSTREAM|YOL0|YOLO)\b
```

#### 2. CAM/TS/Screener/Workprint

Blocks theater recordings and pre-release:
```regex
\b(CAM|CAMRIP|TS|TELESYNC|HDTS|PREDVDRIP|TELECINE|HDTC|SCREENER|DVDSCR|DVDSCREENER|WORKPRINT|WORKPR1NT)\b
```

#### 3. Hardcoded Subtitles

Blocks burned-in subs:
```regex
\b(HC|HCOR|HARDCODED|SUBBED|HQ\.CAM|HD\.CAM|KORSUB|KORSUBBED|KORSUBBED|BLURRED)\b
```

#### 4. BR-DISK (Optional)

Blocks full disc images (huge files):
```regex
\b(COMPLETE\.BLURAY|UNTOUCHED\.BLURAY|BLURAY\.COMPLETE|VC-1|AVC)\b
```

### Recommended Preferences

These improve quality of what you download:

#### 1. Trusted Release Groups (Pick from this list)

**General HD (Movies & TV):**
```regex
\b(FraMeSToR|HQMUX|DON|CtrlHD|BMF|KRaLiMaRKo|PTer|HiFi|FLUX|GHOSTS|NCmt|SaL|VietHD|BHDStudio|SA89|SiCFoI|TDD|TayTO|FoRM|SMURF|EPSiLON|TEPES|KRALiMARKO)\b
```

**4K/UHD Specialists:**
```regex
\b(FraMeSToR|BHDStudio|CtrlHD|EA|GUHZER|KasparS|Kitsune|GNOME|TAT|TeaMate|TRiToN|SumVision|Flights|SURCODE|DREDD)\b
```

**Web-DL Specialists:**
```regex
\b(NTb|FLUX|CasStudio|CMRG|KINGS|monkee|NOSiViD|PAAI|QOQ|SiGMA|TOMMY|ViSiON|WELP|iNTENSo|W4F|MVGroup|AlexKTV)\b
```

Score: `+100` or higher

#### 2. Repack/Proper

Prefers fixed releases:
```regex
\b(REPACK|PROPER)\b
```

Score: `+50`

#### 3. x265/HEVC (Optional)

Prefers more efficient codec:
```regex
\b(x265|HEVC|h\.?265)\b
```

Score: `+25` (or 0 if you prefer x264 for compatibility)

#### 4. High Quality Audio

Prefers lossless/HD audio:
```regex
\b(DTS.?HD\.MA|DTS.?HD|DTS.?X|DTSHD|TrueHD|TRUEHD|Atmos|ATMOS|DDP.?7\.1|DD\+.?7\.1|DD.?EX|FLAC)\b
```

Score: `+50`

## TRaSH Guides Integration

Why reinvent the wheel? TRaSH Guides are community-maintained quality profiles and custom formats that are updated regularly.

### What are TRaSH Guides?

[TRaSH Guides](https://trash-guides.info/) provides:
- Pre-made custom formats for all common scenarios
- Recommended scoring
- Regular updates as the scene changes
- Quality profiles optimized for different needs

### Using Profilarr to Import TRaSH Guides

Remember Profilarr from the setup guide? This is where it shines.

**Step 1: Access Profilarr**
- Navigate to `http://[unraid-ip]:5500`

**Step 2: Connect Your Apps**
- Connections → Add Radarr
  - URL: `http://[unraid-ip]:7878`
  - API Key: From Radarr
- Connections → Add Sonarr
  - URL: `http://[unraid-ip]:8989`
  - API Key: From Sonarr

**Step 3: Import TRaSH Custom Formats**
- Custom Formats → Import from TRaSH
- **For Radarr, import these:**
  - [BR-DISK] (block)
  - [LQ] (Low Quality Groups - block)
  - [x265 (HD)] (optional - depends on preference)
  - [Audio - Advanced] (all the HD audio formats)
  - [HDR Formats] (DV, HDR10+, HDR)
  - [Movie Versions] (Theatrical, Extended, etc.)
  - [Streaming Services] (if you prefer Netflix over Amazon, etc.)
  - [Unwanted] (blocks various issues)
  
- **For Sonarr, import these:**
  - [LQ] (Low Quality Groups - block)
  - [x265 (HD)] (optional)
  - [Audio - Advanced]
  - [Streaming Services]
  - [Unwanted]

**Step 4: Import TRaSH Quality Profiles**
- Quality Profiles → Import from TRaSH
- Popular options:
  - **HD Bluray + WEB** (most popular - 1080p focus)
  - **UHD Bluray + WEB** (4K focus)
  - **Remux + WEB 1080p** (best quality, large files)

**Step 5: Customize Scores**
- Review the imported scores
- Adjust to your preferences (more or less strict)
- Set minimum custom format score in profile

**Step 6: Sync to Apps**
- Select which custom formats to sync to Radarr/Sonarr
- Select which quality profiles to sync
- Click "Sync"

**Done!** Your apps now have community-vetted quality control.

### Recommended TRaSH Setups

**Balanced Setup (Most Popular):**
- Quality Profile: HD Bluray + WEB
- Custom Formats: All defaults from TRaSH
- Minimum Score: +100
- Result: Good quality without giant files

**Quality Focused:**
- Quality Profile: Remux + WEB 1080p
- Custom Formats: All defaults from TRaSH
- Minimum Score: +200
- Result: Highest quality, large files

**4K Setup:**
- Quality Profile: UHD Bluray + WEB
- Custom Formats: All defaults + DV HDR10
- Minimum Score: +100
- Result: 4K with HDR preference

## Complete Example Configuration

Let's build a complete "HD Optimal" profile from scratch.

### Profile Settings

**Name:** HD Optimal

**Allowed Qualities (in order):**
1. Bluray-1080p ✓
2. WEBDL-1080p ✓
3. Bluray-720p ✓ (fallback only)

**Upgrade Until:** Bluray-1080p

**Upgrade Until Custom Score:** +100

**Minimum Custom Format Score:** 0 (accepts good releases, blocks bad)

### Size Limits

| Quality | Min | Max |
|---------|-----|-----|
| Bluray-720p | 20 MB/min | 150 MB/min |
| WEBDL-720p | 10 MB/min | 100 MB/min |
| Bluray-1080p | 50 MB/min | 250 MB/min |
| WEBDL-1080p | 20 MB/min | 150 MB/min |

### Custom Formats with Scores

**Critical Blocks (-10000 each):**
- LQ Release Groups
- CAM/TS/Screener
- Hardcoded Subs
- BR-DISK

**Strong Preferences (+100 each):**
- Trusted Release Groups
- Dolby Vision (if 4K)

**Moderate Preferences:**
- High Quality Audio: +50
- Repack/Proper: +50

**Minor Preferences:**
- x265/HEVC: +25

### What This Achieves

**Blocks:**
- ❌ YIFY/YTS releases
- ❌ CAM/TS recordings
- ❌ Hardcoded subtitles
- ❌ Oversized BR-DISK
- ❌ Files under 50 MB/min (fakes)
- ❌ Files over 250 MB/min (bloated)

**Prefers:**
- ✅ Trusted release groups
- ✅ Bluray-1080p over WEB-1080p
- ✅ High quality audio
- ✅ Proper/Repack releases
- ✅ Reasonable file sizes (6-20GB for 2hr movie)

**Example Results:**

```
Movie: The Matrix (1999) - 136 minutes

Release 1: The.Matrix.1999.1080p.BluRay.x264.YIFY.mkv
- Size: 1.4 GB (10 MB/min - REJECTED, too small)
- Group: YIFY (LQ Groups: -10000)
- Score: -10000 ❌ BLOCKED

Release 2: The.Matrix.1999.1080p.BluRay.x264-Random.mkv  
- Size: 8.5 GB (62 MB/min - GOOD)
- Group: Unknown (no custom format match)
- Score: 0 ✅ ACCEPTABLE

Release 3: The.Matrix.1999.1080p.BluRay.x264.DTS-HD.MA.5.1-FraMeSToR.mkv
- Size: 12.8 GB (94 MB/min - GOOD)  
- Group: FraMeSToR (Trusted: +100)
- Audio: DTS-HD MA (HQ Audio: +50)
- Score: +150 ✅✅ PREFERRED

Result: Downloads Release 3, skips 1 & 2
```

## Testing Your Configuration

### Method 1: Interactive Search

The best way to test is using Interactive Search:

**In Radarr:**
1. Go to Movies → Select a movie
2. Click the person icon → Interactive Search
3. This shows all available releases with scores

**What to look for:**
- Releases you DON'T want should have negative scores or be blocked by size
- Releases you DO want should have positive scores
- The highest-scoring release should be your ideal choice

**Example Output:**
```
The.Matrix.1999.1080p.BluRay.x264.DTS-HD-FraMeSToR.mkv
Size: 12.8 GB (✓ within limits)
Score: +150
Custom Formats: Trusted Groups (+100), HQ Audio (+50)
[This should be at the top]

The.Matrix.1999.1080p.BluRay.x264-RARBG.mkv  
Size: 8.5 GB
Score: -10000
Custom Formats: LQ Groups (-10000)
[This should be blocked/at bottom]
```

### Method 2: Test with Known Releases

Search for releases you know exist:

**Good Release Test:**
- Search for a movie
- Use interactive search
- Verify a known-good release scores high
- Verify it's not blocked by size limits

**Bad Release Test:**
- Search for a movie
- Verify YIFY/YTS releases show -10000 score
- Verify they won't download

### Method 3: Monitor Your First Downloads

After setting up:
1. Request a few movies/shows
2. Go to Activity → Queue
3. Watch what gets grabbed
4. Check Activity → History after imports

**Red flags:**
- Downloading YIFY/RARBG/etc. → Your blocks aren't working
- Files < 3GB for 1080p movies → Size limits too loose
- Never finding anything → Too restrictive, lower cutoff score

## Troubleshooting

### Nothing is Downloading

**Problem:** Searches return no results or everything is rejected.

**Solutions:**

1. **Check your Minimum Custom Format Score**
   - Settings → Profiles → Your Profile
   - If set to +200, try lowering to +100 or 0
   - Very few releases score 200+

2. **Check Size Limits**
   - Settings → Quality
   - Your minimums might be too high
   - Try: 40 MB/min for 1080p instead of 50

3. **Check Indexers**
   - System → Status
   - Are your Jackett indexers working?
   - Try adding more indexers

4. **Check Availability**
   - The content might not exist in your quality yet
   - Try searching manually on your indexers

### Still Getting Bad Releases

**Problem:** YIFY or other garbage is downloading.

**Solutions:**

1. **Verify Custom Format is Created**
   - Settings → Custom Formats
   - Check "LQ Release Groups" exists
   - Verify regex is correct

2. **Verify Custom Format is Assigned to Profile**
   - Settings → Profiles → Your Profile
   - Scroll to Custom Formats
   - "LQ Release Groups" should show -10000

3. **Check the Release Name**
   - Activity → History → View the release
   - Does it actually match your regex?
   - You might need to add more groups to the regex

4. **Test Regex**
   - Use regex101.com
   - Paste your regex
   - Test with actual release names
   - Adjust as needed

### Too Many Upgrades

**Problem:** Constantly downloading upgrades, using bandwidth/seedbox space.

**Solutions:**

1. **Set Upgrade Until Custom Score**
   - Settings → Profiles → Your Profile
   - Set to +100 or +150
   - Won't upgrade beyond this score

2. **Lower Custom Format Scores**
   - If minor preferences are +50, reduce to +25
   - Upgrades trigger on small score differences

3. **Disable Upgrades Entirely**
   - Per movie/show: Edit → Unmonitor

### Files Still Too Large/Small

**Problem:** File sizes outside your preferred range.

**Solutions:**

1. **Review Size Limits**
   - Settings → Quality → Click quality level
   - Check min/max are actually set
   - Radarr uses "Preferred" slider for variable length content

2. **Check Custom Format Scores**
   - Remux releases are huge (100+ GB)
   - If scoring Remux +200, it might override size preferences
   - Lower Remux score or remove from allowed qualities

3. **For TV Shows**
   - Use "Per Episode" size limits, not total season size

### Custom Formats Not Working

**Problem:** Releases aren't getting scored by custom formats.

**Solutions:**

1. **Verify Regex Syntax**
   - Must use `\b` for word boundaries
   - Case insensitive (no need for /i flag)
   - Test on regex101.com

2. **Check Format Conditions**
   - Settings → Custom Formats → Your Format
   - Must be "Release Title" condition
   - Not "File Name" or other conditions

3. **Verify Format is in Profile**
   - Settings → Profiles → Your Profile
   - Custom format must be listed
   - Must have a non-zero score

4. **Test with Interactive Search**
   - See scores in real-time
   - Identifies which formats match

## Progressive Implementation Strategy

Don't try to do everything at once. Here's a week-by-week approach:

### Week 1: Foundation

**Goals:**
- Set up basic quality profiles
- Configure size limits
- Block obvious garbage

**Tasks:**
1. Create 1-2 quality profiles (HD Standard, maybe 4K)
2. Set size limits for each quality level
3. Create and assign these custom formats:
   - LQ Release Groups (-10000)
   - CAM/TS/Screener (-10000)
   - Hardcoded Subs (-10000)
4. Test with interactive search
5. Monitor first few downloads

### Week 2: Refinement

**Goals:**
- Add positive scoring
- Fine-tune size limits based on results

**Tasks:**
1. Create Trusted Release Groups (+100)
2. Create HQ Audio (+50)
3. Add x265/HEVC (+25) if desired
4. Review week 1 downloads
5. Adjust size limits if needed

### Week 3: TRaSH Integration

**Goals:**
- Import community standards
- Sync across apps

**Tasks:**
1. Set up Profilarr connections
2. Import TRaSH custom formats
3. Import TRaSH quality profile
4. Customize scores to preference
5. Sync to Radarr and Sonarr

### Week 4: Optimization

**Goals:**
- Fine-tune based on real-world usage
- Set cutoff scores

**Tasks:**
1. Review 3 weeks of downloads
2. Add any missed bad groups to blocks
3. Adjust positive scores if needed
4. Set minimum custom format scores
5. Set upgrade until scores

### Ongoing: Maintenance

**Monthly:**
- Check TRaSH Guides for updates
- Review new bad release groups in community
- Update Profilarr sync

**When Issues Arise:**
- Add bad groups to block list as you find them
- Adjust scores based on what you're getting
- Fine-tune size limits for edge cases

## Advanced Tips

### Conditional Scoring

You can create complex rules:

**Example: Prefer x265 Only for 4K**
- Custom Format: x265 HD → Score: 0 (neutral for 1080p)
- Custom Format: x265 UHD → Score: +50 (positive for 4K)

### Release Group Tiers

Instead of one "Trusted Groups" format, create tiers:

- **Tier 1 (Best):** FraMeSToR, DON, CtrlHD → +150
- **Tier 2 (Good):** W4F, NTb, FLUX → +75
- **Tier 3 (Acceptable):** Most others → 0

### Streaming Service Preferences

If you prefer Netflix over Amazon:

- Custom Format: Netflix → +25
- Custom Format: Amazon → +10
- Custom Format: Disney+ → +15

### Language Preferences

For non-English content:

- Custom Format: Japanese Audio → +100 (for anime)
- Custom Format: English Dub → -100 (if you prefer subs)

### Edition Preferences

For movies with multiple versions:

- Custom Format: Extended Cut → +50
- Custom Format: Theatrical Cut → 0
- Custom Format: Director's Cut → +75

## Conclusion

Quality control in Radarr and Sonarr is all about layers:

1. **Quality Definitions** catch 90% of problems
2. **Custom Formats** fine-tune the remaining 10%
3. **Scoring** ensures you get the best available

**The Golden Rules:**

1. Start with **size limits** - they're your best defense
2. **Block the obvious** - LQ groups, CAM, hardcoded subs
3. **Prefer the good** - trusted groups, good audio/video
4. **Test before deploying** - use interactive search
5. **Iterate and improve** - adjust based on results
6. **Use TRaSH Guides** - don't reinvent the wheel

**Remember:**
- Perfect is the enemy of good
- Too strict = nothing downloads
- Too lenient = garbage downloads
- The goal is content you'll actually watch

Start simple, test often, and refine over time. Your library will thank you!

---

**Additional Resources:**

- [TRaSH Guides](https://trash-guides.info/) - Community quality profiles
- [Radarr Wiki](https://wiki.servarr.com/radarr) - Official documentation
- [Sonarr Wiki](https://wiki.servarr.com/sonarr) - Official documentation
- [Regex101](https://regex101.com/) - Test your regex patterns
- [r/radarr](https://reddit.com/r/radarr) - Community support
- [r/sonarr](https://reddit.com/r/sonarr) - Community support

**Questions?** Open an issue in this repository or check the Unraid forums!
