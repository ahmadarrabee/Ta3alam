# Backup & Recovery Procedures

Owner: Ahmad Arrabee; execution by authorized local technical support. Backups contain private learner data and follow the [privacy retention policy](PRIVACY.md).

## Physical USB security

Use dedicated, labeled drives; record custody, snapshot dates, and verification results in a restricted local log. Keep drives in a locked cabinet, restrict handling to authorized staff, and never lend them to learners or connect them to public computers. Prefer encrypted storage with recovery keys kept separately by Ahmad; the scripts do **not** encrypt backups. Limit filesystem access and never upload snapshots. For unattended backups an encrypted drive must already be unlocked and mounted securely. Safely unmount before removal and keep a rotated verified copy physically separate from the server. Report loss immediately.

## Daily 02:00 backup

Confirm the real Kolibri data directory in **Device > Info**, the service identity/configuration, the checkout path, the USB mount, and server local time (`timedatectl`). `/home/server/.kolibri` below is an example, not an assumed service data path. On the Ubuntu server, verify a manual run:

```bash
sudo bash /opt/offline-learning-hub/maintenance/backup_kolibri.sh /home/server/.kolibri /media/server/BACKUP_USB
```

Then use `sudo crontab -e` to add:

```cron
0 2 * * * /bin/bash /opt/offline-learning-hub/maintenance/backup_kolibri.sh /home/server/.kolibri /media/server/BACKUP_USB >> /var/log/kolibri-backup.log 2>&1
```

This is 02:00 AM every day in the **server's local timezone**; configure it to `Asia/Amman` for the Jordan deployment. Cron does not catch up a missed run when the laptop is off. Leave the server powered on and USB mounted with enough free space; review the log and newest completed snapshot each morning.

The wrapper requires systemd, Bash, `flock`, and Python 3. It locks against overlapping wrapper runs, checks that the service exists, stops Kolibri, and calls `maintenance/auto_backup.py`. An exit trap attempts to restart Kolibri only if it was active initially, including after a copy failure; verify service health after every run. Stop any other writers too. Do not run manual backups or restores concurrently with cron.

Snapshots are stored in `<USB mount>/kolibri_backups/backup_<timestamp>_<suffix>`. A failed copy retains `.partial`; never restore it as a completed snapshot. The script checks that the USB is a mount point and never overwrites or automatically prunes old snapshots. Staff must apply the retention policy, remove expired copies securely, and manage capacity. Symlinks are preserved rather than followed: back up externally linked content separately. A completed copy alone is not proof of recoverability.

## Restore a completed snapshot

1. Arrange downtime, disconnect learners, and temporarily disable the scheduled backup. Confirm no backup is running. Identify a completed snapshot and record its timestamp and Kolibri version. Use a matching version first, with any later upgrade handled separately.
2. Confirm the actual data path, its owner/group (`stat`), any service `User`/`Group` settings (`systemctl cat kolibri`), and permissions. Unlock/mount the USB and verify the source. Do not paste paths blindly.
3. Stop Kolibri and all other writers:

   ```bash
   sudo systemctl stop kolibri
   systemctl is-active kolibri
   ```

   Confirm it is inactive before copying.
4. Keep a separate, secured rollback copy of the existing data. The following Bash example deliberately requires replacement paths and identities. Check free space and ensure the rollback destination does not already exist:

   ```bash
   data_dir=/actual/kolibri/data
   snapshot=/media/server/BACKUP_USB/kolibri_backups/backup_COMPLETED_TIMESTAMP
   rollback_dir=/secure/local/kolibri-before-restore-UNIQUE_TIMESTAMP
   service_user=ACTUAL_KOLIBRI_USER
   service_group=ACTUAL_KOLIBRI_GROUP
   sudo cp -a -- "$data_dir" "$rollback_dir"
   ```

   Check that this succeeds and inspect the rollback copy before continuing. Keep rollback data outside the portal's document root.
5. Move the stopped live directory to a separate, unused sibling path, then recreate it and copy the snapshot **contents**, including hidden files. This avoids merging stale database files:

   ```bash
   sudo mv -- "$data_dir" "${data_dir}.pre-restore-UNIQUE_TIMESTAMP"
   sudo mkdir -- "$data_dir"
   sudo cp -a -- "$snapshot"/. "$data_dir"/
   sudo chown -R -- "$service_user:$service_group" "$data_dir"
   ```

   Stop on any error. Restore the recorded directory access modes/ACLs for this installation; do not grant world write access or apply blanket executable permissions. Verify external symlink targets separately. If the data path is itself a mount point or symlink, have technical support plan restoration into that layout instead of moving it.
6. Reapply deletions recorded since the snapshot before learners reconnect. Restart and inspect:

   ```bash
   sudo systemctl start kolibri
   systemctl is-active kolibri
   sudo journalctl -u kolibri -n 100 --no-pager
   ```

7. Test authorized sign-in, classes, recorded progress, Arabic lessons, and a new saved exercise. If verification fails, stop Kolibri, retain the failed restore for diagnosis, and recover the secured pre-restore copy with its matching software. Re-enable cron only after validation. Securely expire rollback copies under the same retention rules.

## Quarterly recovery verification

Every quarter, Ahmad and local technical support restore a selected completed snapshot onto a spare, isolated installation using matching Kolibri software. Include the newest backup and rotate through older retained snapshots and USB drives. Keep production untouched and prevent duplicate learner data from reaching any external network.

Record snapshot/drive identity, backup and software versions, tester/date, restore duration, file-copy integrity checks, account/class counts, sample progress, lesson playback, permissions, and a saved exercise test. Record pass/fail and corrective actions; repeat after failures, replacement hardware, or major upgrades. Remove test learner data securely afterward. Treat a failed verification as an operational incident and create a new verified backup once corrected.
