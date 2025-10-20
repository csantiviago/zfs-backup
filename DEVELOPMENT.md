# DEVELOPMENT.md

This file provides development guidance and technical documentation for the ZFS Backup Scripts project.

## Overview

This is a ZFS snapshot management system consisting of Bash scripts for automating ZFS snapshot creation and retention cleanup on Ubuntu/Linux systems.

## Scripts

### zfs-backup
Creates ZFS snapshots with TrueNAS-compatible naming format (`auto-YYYY-MM-DD_HH-MM`).

**Features:**
- Error handling with `set -euo pipefail`
- Early sudo privilege validation (checks before any ZFS operations)
- Dataset existence validation
- Comprehensive logging to `/var/log/zfs-backup.log`
- Environment variable configuration support

**Configuration:**
- `ZFS_DATASET` - Dataset path (required, no default)

### zfs-backup-cleanup
Manages snapshot retention by deleting old snapshots beyond the configured retention period.

**Features:**
- Chronological ordering (deletes oldest first)
- Early sudo privilege validation (checks before any ZFS operations)
- Input validation for retention count
- Error handling and failed deletion tracking
- Comprehensive logging to `/var/log/zfs-backup.log`
- Environment variable configuration support

**Configuration:**
- `ZFS_DATASET` - Dataset path (required, no default)
- `ZFS_KEEP` - Number of snapshots to retain (default: 7)

## Architecture

Both scripts include:
1. Strict error handling (`set -euo pipefail`)
2. Early sudo privilege validation using `sudo -v` (checks BEFORE any ZFS operations)
3. Pre-flight validation (dataset exists, sudo access available)
4. Timestamped logging to both stderr and `/var/log/zfs-backup.log`
5. Proper exit codes for success/failure monitoring

The cleanup script:
1. Lists all snapshots for the dataset sorted by creation time
2. Counts total snapshots
3. If count exceeds `KEEP` limit, calculates how many to delete
4. Deletes oldest snapshots first
5. Tracks and reports any failures

## Running Scripts

All scripts require sudo privileges for ZFS operations:

```bash
# Create a snapshot
sudo ./zfs-backup

# Clean up old snapshots
sudo ./zfs-backup-cleanup

# Override default configuration via environment variables
sudo ZFS_DATASET="rpool/data/mydata" ZFS_KEEP=14 ./zfs-backup-cleanup
```

## Scheduling with Cron

Add to root's crontab (`sudo crontab -e`):

```cron
# Create snapshot daily at 2 AM
0 2 * * * /home/cadu/NAS/devel/bash/zfs-backup/zfs-backup

# Clean up old snapshots daily at 3 AM
0 3 * * * /home/cadu/NAS/devel/bash/zfs-backup/zfs-backup-cleanup
```

## Logging

Scripts log all operations to `/var/log/zfs-backup.log` with timestamps. Log entries include:
- Snapshot creation/deletion operations
- Success/failure status
- Error messages with context
- Snapshot counts and retention information

View logs:
```bash
sudo tail -f /var/log/zfs-backup.log
```

## Dependencies

- `zfs` command-line tools must be installed
- User needs sudo access for `zfs snapshot` and `zfs destroy` commands
- Write access to `/var/log/zfs-backup.log` (handled via sudo)
- Standard utilities: `date`, `grep`, `awk`, `wc`, `head`, `tee`
