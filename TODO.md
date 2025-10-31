# TODO - ZFS Backup Scripts Improvements

## High Priority

- [ ] **Filter cleanup to only auto-snapshots**
  - Currently `zfs-backup-cleanup` deletes ANY snapshot, including manual ones
  - Should only delete snapshots matching the `auto-*` pattern
  - Implementation: Add `grep "@auto-"` filter to line 44
  ```bash
  SNAPSHOT_LIST=$(sudo zfs list -H -t snapshot -o name -s creation -d 1 "${DATASET}" 2>/dev/null | grep "@auto-" || true)
  ```

- [ ] **Check for duplicate snapshots**
  - `zfs-backup` doesn't check if a snapshot already exists
  - If run twice in the same minute, it fails
  - Implementation: Check before creating snapshot (before line 39)
  ```bash
  if sudo zfs list -H -o name "${SNAPSHOT}" &>/dev/null; then
      log "WARNING: Snapshot '${SNAPSHOT}' already exists, skipping"
      exit 0
  fi
  ```

- [ ] **Add dry-run mode**
  - Useful for testing without making changes
  - Add `ZFS_DRY_RUN` environment variable support
  - Implementation: Add to configuration section and wrap all zfs commands
  ```bash
  DRY_RUN="${ZFS_DRY_RUN:-false}"
  if [ "${DRY_RUN}" = "true" ]; then
      log "DRY-RUN: Would execute: zfs snapshot ${SNAPSHOT}"
  else
      sudo zfs snapshot "${SNAPSHOT}"
  fi
  ```

## Medium Priority

- [ ] **Prevent concurrent execution**
  - Add lock file to prevent multiple instances running simultaneously
  - Implementation: Use flock
  ```bash
  LOCKFILE="/var/run/zfs-backup-${DATASET//\//-}.lock"
  exec 200>"${LOCKFILE}"
  if ! flock -n 200; then
      log "ERROR: Another instance is already running"
      exit 1
  fi
  ```

- [ ] **Add snapshot verification**
  - After creating a snapshot, verify it actually exists
  - Implementation: Add verification after snapshot creation
  ```bash
  if ! sudo zfs list -H -o name "${SNAPSHOT}" &>/dev/null; then
      log "ERROR: Snapshot created but verification failed"
      exit 1
  fi
  ```

- [ ] **Configuration file support**
  - Allow reading from `/etc/zfs-backup.conf` as alternative to environment variables
  - Implementation: Source config file if it exists
  ```bash
  CONFIG_FILE="${ZFS_CONFIG_FILE:-/etc/zfs-backup.conf}"
  if [ -f "${CONFIG_FILE}" ]; then
      source "${CONFIG_FILE}"
  fi
  ```

- [ ] **Better timestamp precision**
  - If running backups more frequently than once per minute, add seconds
  - Implementation: Update timestamp format
  ```bash
  TIMESTAMP="$(date +auto-%Y-%m-%d_%H-%M-%S)"
  ```

## Lower Priority

- [ ] **Recursive dataset support**
  - Add `ZFS_RECURSIVE` option to handle child datasets
  - Useful for backing up entire dataset trees

- [ ] **Email notifications**
  - Send alerts on failures
  - Use system mail or custom SMTP configuration

- [ ] **Metrics/monitoring integration**
  - Export snapshot counts/ages for Prometheus or other monitoring systems
  - Could output metrics to file or expose via exporter

- [ ] **Advanced retention policies**
  - Support different retention for hourly/daily/weekly/monthly snapshots
  - GFS (Grandfather-Father-Son) style retention scheme

- [ ] **Pre/post hooks**
  - Run commands before/after snapshots for application consistency
  - Example: Flush database buffers, freeze filesystems temporarily
  - Implementation: `ZFS_PRE_HOOK` and `ZFS_POST_HOOK` environment variables
