# ZFS Backup Scripts

Automated ZFS snapshot management scripts for Ubuntu/Linux systems with TrueNAS-compatible naming.

## Features

- **Automated snapshot creation** with configurable scheduling
- **Retention management** - automatically clean up old snapshots
- **TrueNAS-compatible naming** format (`auto-YYYY-MM-DD_HH-MM`)
- **Comprehensive logging** to `/var/log/zfs-backup.log`
- **Error handling** with proper exit codes for monitoring
- **Configurable** via environment variables
- **Safe operations** with pre-flight validation and sudo checks

## Requirements

- ZFS utilities (`zfsutils-linux` on Ubuntu/Debian)
- Sudo privileges for ZFS operations
- Bash 4.0 or later

## Installation

1. Clone the repository:
```bash
git clone https://github.com/yourusername/zfs-backup.git
cd zfs-backup
```

2. Make scripts executable:
```bash
chmod +x zfs-backup zfs-backup-cleanup
```

3. Configure your ZFS dataset (see Configuration section below)

4. Test manually with sudo:
```bash
sudo ZFS_DATASET="your/dataset" ./zfs-backup
```

## Configuration

The scripts use environment variables for configuration:

| Variable | Description | Default | Used By |
|----------|-------------|---------|---------|
| `ZFS_DATASET` | ZFS dataset to snapshot | *(required)* | Both scripts |
| `ZFS_KEEP` | Number of snapshots to retain | `7` | zfs-backup-cleanup |

### Important: Configure Your Dataset

**You must set `ZFS_DATASET` to your actual ZFS dataset path.** The scripts do not have a working default value.

To find your datasets:
```bash
zfs list
```

Example output:
```
NAME                    USED  AVAIL  REFER  MOUNTPOINT
rpool                   100G  800G    96K  /rpool
rpool/data              50G   800G   50G   /data
rpool/home              30G   800G   30G   /home
```

## Usage

### Creating Snapshots

```bash
# Create a snapshot of your dataset
sudo ZFS_DATASET="rpool/data" ./zfs-backup
```

### Cleaning Up Old Snapshots

```bash
# Keep only the 7 most recent snapshots
sudo ZFS_DATASET="rpool/data" ./zfs-backup-cleanup

# Keep 14 snapshots
sudo ZFS_DATASET="rpool/data" ZFS_KEEP=14 ./zfs-backup-cleanup
```

## Automated Scheduling with Cron

Add to root's crontab (`sudo crontab -e`):

```cron
# Create daily snapshot at 2 AM
0 2 * * * ZFS_DATASET="rpool/data" /path/to/zfs-backup/zfs-backup

# Clean up old snapshots at 3 AM (keep 7)
0 3 * * * ZFS_DATASET="rpool/data" /path/to/zfs-backup/zfs-backup-cleanup
```

See [example-crontab](example-crontab) for more scheduling examples.

## Logging

All operations are logged to `/var/log/zfs-backup.log` with timestamps.

View recent activity:
```bash
sudo tail -f /var/log/zfs-backup.log
```

Example log output:
```
[2025-10-14 02:00:01] Creating snapshot: rpool/data@auto-2025-10-14_02-00
[2025-10-14 02:00:02] SUCCESS: Snapshot created successfully
[2025-10-14 03:00:01] INFO: Found 10 snapshot(s) for dataset 'rpool/data', keeping 7
[2025-10-14 03:00:01] INFO: Deleting 3 old snapshot(s)
[2025-10-14 03:00:02] SUCCESS: Cleanup completed successfully
```

## How It Works

### zfs-backup
1. Validates sudo privileges
2. Checks dataset exists
3. Creates snapshot with timestamp: `<dataset>@auto-YYYY-MM-DD_HH-MM`
4. Logs results

### zfs-backup-cleanup
1. Validates configuration and sudo privileges
2. Lists all snapshots sorted by creation time (oldest first)
3. Counts snapshots and compares to retention limit
4. Deletes oldest snapshots exceeding the limit
5. Tracks and reports any failures

## Error Handling

Both scripts:
- Exit with code `0` on success
- Exit with code `1` on any error
- Use `set -euo pipefail` for strict error handling
- Validate prerequisites before performing operations
- Log all errors with context

This makes them safe for automated monitoring and alerting.

## Troubleshooting

**"ERROR: Dataset does not exist"**
- Verify dataset path with `zfs list`
- Ensure you're setting `ZFS_DATASET` correctly

**"ERROR: sudo privileges required"**
- Run scripts with `sudo`
- Ensure your user has sudo access
- Check `/etc/sudoers` configuration

**"ERROR: Failed to create/destroy snapshot"**
- Check ZFS pool health: `zpool status`
- Verify sufficient space: `zfs list -o space`
- Review logs: `sudo tail /var/log/zfs-backup.log`

## Contributing

Contributions welcome! Please see [DEVELOPMENT.md](DEVELOPMENT.md) for development guidelines.

## License

MIT License - see [LICENSE](LICENSE) file for details.

## Security

These scripts require sudo/root privileges for ZFS operations. Review the code before running. The scripts:
- Perform only ZFS snapshot operations (create/destroy)
- Do not modify data within datasets
- Include input validation and error checking
- Log all operations for audit trails

## Support

- Report issues: [GitHub Issues](https://github.com/yourusername/zfs-backup/issues)
- Documentation: This README and [DEVELOPMENT.md](DEVELOPMENT.md)
