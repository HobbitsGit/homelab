# Unraid Homelab — Maintenance Guide

This document covers routine maintenance tasks for the Unraid media server stack, including updates, backups to local SSD and Backblaze, and restore procedures.

---

## Table of Contents

- [Routine Maintenance Schedule](#routine-maintenance-schedule)
- [Updating Containers](#updating-containers)
- [Updating Unraid OS](#updating-unraid-os)
- [Backup Strategy Overview](#backup-strategy-overview)
- [Appdata Backup (Local SSD)](#appdata-backup-local-ssd)
- [Unraid Flash Backup](#unraid-flash-backup)
- [Backblaze B2 Backup](#backblaze-b2-backup)
- [Restoring from Backup](#restoring-from-backup)
  - [Restore Appdata from Local SSD](#restore-appdata-from-local-ssd)
  - [Restore Appdata from Backblaze](#restore-appdata-from-backblaze)
  - [Restore Unraid Flash/Config](#restore-unraid-flashconfig)
- [Parity Checks](#parity-checks)
- [Drive Health Monitoring](#drive-health-monitoring)
- [UPS Management](#ups-management)

---

## Routine Maintenance Schedule

| Task | Frequency | How |
|------|-----------|-----|
| Parity check | Monthly (1st, 3:00 AM) | Automated via Scheduler |
| Appdata backup to SSD | Weekly (Sunday, 3:00 AM) | Appdata Backup plugin |
| Backblaze backup | Ongoing/continuous | Configured via plugin |
| Docker container updates | Weekly | Manual or notification-triggered |
| Unraid OS updates | As released | Manual after reviewing changelog |
| SMART drive check | Monthly | Via Unraid Disk Settings |
| NVMe TRIM | Monthly (15th) | Automated via Scheduler |
| Review notification emails | Ongoing | Check Gmail → forwarded to Proton |

---

## Updating Containers

### Method 1 — Via Unraid Dashboard (Recommended)

1. Go to the **Docker** tab
2. Look for containers showing an update available badge
3. Click the container icon → **Check for Updates**
4. If an update is available, click **Update**
5. The container will pull the new image and restart automatically

### Method 2 — Update All at Once

1. Go to **Docker** tab
2. Click **Check for Updates** at the top of the page
3. Review the list of available updates
4. Click **Update All** — or update individually to be safe

### Best Practice for Updating

- Update **one container at a time** when possible
- After updating, verify the container starts and the web UI is accessible
- Check logs for errors after updating: click the container icon → **Logs**
- For critical containers (Gluetun, SABnzbd, qBittorrent), verify VPN is still working after update:

```bash
docker exec qbittorrent curl -s https://api.ipify.org
docker exec sabnzbd curl -s https://api.ipify.org
```

Both should return a VPN IP, not your real home IP.

### Container Update Order

Update in this order to avoid dependency issues:

1. Gluetun (VPN gateway)
2. qBittorrent
3. SABnzbd
4. Prowlarr
5. Radarr
6. Sonarr
7. Jellyfin
8. Jellyseerr
9. FlareSolverr
10. All others

---

## Updating Unraid OS

1. Go to **Tools → Update OS**
2. Review the release notes before updating
3. Unraid will download and stage the update
4. Schedule the update for a low-activity time
5. The server will reboot to apply the update
6. After reboot, verify all containers start correctly
7. Run a quick parity check if prompted

> **Note:** Always back up your Unraid flash drive before a major OS version update.

---

## Backup Strategy Overview

This setup uses a 3-2-1 backup approach:

| Copy | Location | What |
|------|----------|------|
| 1 | Array (protected by parity) | All media files |
| 2 | 250GB SSD (Unassigned Device) | Appdata, flash config |
| 3 | Backblaze B2 (offsite) | Appdata, important documents |

**What is backed up:**
- Docker appdata (container configs, databases)
- Unraid flash/USB config
- Important documents share
- VM configs (if applicable)

**What is NOT backed up locally (too large):**
- Media files (movies, TV shows) — these can be re-downloaded
- Photos — backed up to Backblaze directly

---

## Appdata Backup (Local SSD)

The **Appdata Backup** plugin handles this automatically.

### Schedule

- **Frequency:** Weekly, Sunday at 3:00 AM
- **Destination:** `/mnt/disks/Backup/appdata`
- **Retention:** 30 days

### Manual Backup

To run a backup manually at any time:

1. Go to **Settings → Appdata Backup**
2. Click **Start Backup**
3. Monitor the log for completion

### Verifying the Backup

1. Go to **Main** tab → scroll down to Unassigned Devices
2. Check the **Used** column on the Backup SSD — it should show some used space
3. Or browse to `/mnt/disks/Backup/appdata` via the Files tab to see backup archives

### What Gets Backed Up

- `/mnt/cache/appdata/` — all Docker container config data
- Unraid flash drive configuration
- VM metadata

---

## Unraid Flash Backup

The Unraid flash (USB boot drive) stores your array configuration, Docker templates, and settings. Losing it without a backup means reconfiguring from scratch.

### Automatic Flash Backup (Appdata Backup Plugin)

The Appdata Backup plugin is configured to back up the flash drive automatically as part of its weekly run. This goes to `/mnt/disks/Backup/appdata`.

### Manual Flash Backup

1. Go to **Main** tab
2. Click on **Flash** in the Boot Device section
3. Click **Flash Backup** → **Download**
4. Save the ZIP file somewhere safe (external drive, cloud storage)

### Unraid Connect Flash Backup

If using Unraid Connect:
1. Go to **Tools → Unraid Connect**
2. Enable **Flash Backup** in the settings
3. Unraid will automatically back up your flash config to Lime Technology's servers

---

## Backblaze B2 Backup

### Setup Overview

Backblaze B2 provides offsite backup for critical data. Configure it using one of these methods:

**Option A — Duplicati (Docker container)**
- Runs as a Docker container
- Backs up specific folders to B2
- Supports encryption and scheduling
- Recommended for appdata and documents

**Option B — Rclone (via User Scripts plugin)**
- Command-line based
- Very flexible and fast
- Good for large file sets

### Recommended Backup Targets to Backblaze

| Folder | Priority |
|--------|----------|
| `/mnt/disks/Backup/appdata` | High — offsite copy of local SSD backup |
| `/mnt/user/Documents` | High — irreplaceable personal files |
| `/mnt/user/Photos` | High — personal photos |
| `/mnt/user/data/media` | Low — re-downloadable, large |

### Monitoring Backblaze Backups

- Log into **backblaze.com** to verify files are being uploaded
- Check your backup container logs weekly
- Set up email alerts in Backblaze for failed uploads

---

## Restoring from Backup

### Restore Appdata from Local SSD

Use this when a container's config gets corrupted or you need to roll back a container.

**Full appdata restore:**

1. Stop the affected container:
   - Docker tab → click container → **Stop**
2. Delete or rename the corrupted appdata folder:
   ```bash
   mv /mnt/cache/appdata/CONTAINERNAME /mnt/cache/appdata/CONTAINERNAME.bak
   ```
3. Find the backup archive on the SSD:
   ```bash
   ls /mnt/disks/Backup/appdata/
   ```
4. Extract the backup:
   ```bash
   tar -xzf /mnt/disks/Backup/appdata/BACKUP_FILE.tar.gz -C /mnt/cache/appdata/
   ```
5. Start the container and verify it works

**Single container restore via Appdata Backup plugin:**

1. Go to **Settings → Appdata Backup**
2. Look for a restore option in the plugin UI
3. Select the container and backup date to restore from

### Restore Appdata from Backblaze

Use this if the local SSD backup is also unavailable (catastrophic failure).

1. Install Duplicati or Rclone on a working system
2. Connect to your Backblaze B2 bucket
3. Download the relevant backup archive
4. Follow the same extraction steps as above

### Restore Unraid Flash/Config

**If USB drive fails:**

1. Get a new USB drive (minimum 1GB, use a quality brand)
2. Download the Unraid OS installer from **unraid.net**
3. Flash it to the new USB drive using the Unraid USB Creator tool
4. Copy your flash backup ZIP contents to the new USB drive's `/config` folder
5. Boot from the new USB drive
6. Your license is tied to the USB GUID — you may need to transfer it at **keys.unraid.net**

**If config gets corrupted (drive is fine):**

1. Boot into Unraid
2. Go to **Tools → New Config** (use with caution — this resets array assignments)
3. Or restore specific config files from your flash backup ZIP

---

## Parity Checks

### Scheduled Check

- **Frequency:** Monthly, 1st of the month at 3:00 AM
- **Type:** Read-only (non-correcting) — reports errors without fixing
- **Configured in:** Settings → Scheduler → Parity Check

### Manual Parity Check

1. Go to **Main** tab
2. Click **Check** next to the array
3. Choose **Read-Only** for routine checks
4. Choose **Correcting** only if a previous check found errors

### Interpreting Results

| Result | Meaning |
|--------|---------|
| 0 errors | Everything is healthy ✅ |
| Errors found (first time) | Run a correcting check to fix |
| Errors found repeatedly | Possible failing drive — check SMART data |

### Parity Check and Performance

Parity checks significantly impact performance:
- Array read/write speeds are reduced by ~50%
- Jellyfin streaming and transcoding may be affected
- Scheduled for 3:00 AM to minimize impact
- A 28TB parity check typically takes 8-16 hours

---

## Drive Health Monitoring

### SMART Data

1. Go to **Main** tab
2. Click on any drive
3. Click **SMART** to view drive health data

Key values to watch:
- **Reallocated Sectors Count** — should be 0
- **Pending Sector Count** — should be 0
- **Uncorrectable Sector Count** — should be 0
- **Temperature** — ideally below 50°C

### Temperature Monitoring

Unraid shows drive temperatures on the Main tab. Healthy ranges:
- **HDD:** 30–50°C normal, above 55°C investigate
- **NVMe SSD:** 30–70°C normal, above 75°C investigate

### Notifications

Unraid is configured to send email alerts for:
- Drive SMART warnings
- Array errors
- Parity check completion/results
- Temperature warnings
- Plugin/OS update availability

Alerts go to Gmail → forwarded automatically to Proton Mail.

---

## UPS Management

### CyberPower CP1500PFCLCD

The UPS protects the NAS, router, switch, and access point from power loss.

### Unraid UPS Integration

Go to **Settings → UPS Settings** to view and configure:
- Current battery level
- Estimated runtime
- Shutdown threshold (set to 25-30% battery)

### What Happens During a Power Outage

1. Power fails → UPS switches to battery
2. Unraid monitors battery percentage
3. When battery reaches the configured threshold, Unraid initiates a clean shutdown
4. All containers stop gracefully, array spins down safely
5. Power returns → manually power on the server

### UPS Runtime Estimate

At typical load (~130W), the CP1500PFCLCD provides approximately 20-30 minutes of runtime — enough time for a clean shutdown plus some buffer.

### Checking UPS Status

The LCD display on the UPS shows:
- Input/output voltage
- Load percentage
- Battery level
- Estimated runtime

---

## Quick Reference — Common Commands

```bash
# Check VPN is working for download clients
docker exec qbittorrent curl -s https://api.ipify.org
docker exec sabnzbd curl -s https://api.ipify.org

# Check disk usage
df -h /mnt/user/data/

# List contents of media folders
ls /mnt/user/data/media/movies/
ls /mnt/user/data/media/tv/

# View container logs
docker logs CONTAINERNAME --tail 50

# Restart a container
docker restart CONTAINERNAME

# Check all running containers
docker ps

# Force Jellyfin library scan (run in terminal)
# Or use Dashboard → Libraries → Scan All Libraries in the web UI

# Check NVMe health
nvme smart-log /dev/nvme0

# Check array drive SMART
smartctl -a /dev/sdX
```

---

*Last updated: Based on initial server build — update this document as configuration changes.*
