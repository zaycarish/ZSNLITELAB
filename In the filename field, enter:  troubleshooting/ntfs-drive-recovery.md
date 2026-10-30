# Case Study 001: NTFS Drive Mount Troubleshooting

**System:** ZSNLITE — Ubuntu Linux Server  
**Hardware:** Dell OptiPlex 3880  
**Category:** Linux Administration / Storage  
**Filesystem:** NTFS  
**Status:** Mount verified — read/write

## 1. Problem Overview

While maintaining my self-hosted media server, I investigated a mounting issue involving an external NTFS-formatted storage drive.

The drive is used to store media files accessed by services running on my Ubuntu Linux server, including Jellyfin.

The troubleshooting objective was to inspect the drive's filesystem, verify its condition, and confirm that Ubuntu could mount it with read/write access.

## 2. Environment

- **Operating System:** Ubuntu Linux
- **Server:** Dell OptiPlex 3880
- **Storage:** External NTFS drive
- **Mount point:** `/media/M1`
- **Relevant tools:** `findmnt`, `ntfsfix`

## 3. Initial Investigation

I started by checking whether the filesystem was mounted at the expected location.

```bash
findmnt /media/M1
```

The command initially returned no output, indicating that no filesystem was mounted at that location at the time of the check.

I then inspected the NTFS filesystem using a non-modifying diagnostic command:

```bash
sudo ntfsfix --no-action /dev/disk/by-uuid/FA3CF00C3CEFC22D
```

The diagnostic reported:

```text
Mounting volume... OK
Processing of $MFT and $MFTMirr completed successfully.
Checking the alternate boot sector... OK
NTFS volume version is 3.1.
NTFS partition was processed successfully.
```

These results indicated that the NTFS metadata checks completed without reporting errors.

The `--no-action` option was important because I had not yet backed up the drive and wanted to inspect its condition without requesting filesystem modifications.

## 4. Mount Verification

After further troubleshooting, I checked the mounted filesystem again:

```bash
findmnt -no SOURCE,FSTYPE,OPTIONS /media/M1
```

The command returned:

```text
/dev/sdb1 fuseblk rw,relatime,user_id=0,group_id=0,default_permissions,allow_other,blksize=4096
```

### Understanding the Output

- `/dev/sdb1` — The block device containing the filesystem.
- `fuseblk` — The filesystem is mounted through a FUSE-based driver.
- `rw` — The filesystem is mounted with read/write access.
- `relatime` — A filesystem access-time optimization.
- `allow_other` — Allows access by users other than the mounting user, subject to applicable permissions.

## 5. Findings

The investigation established that:

1. The expected mount point initially had no active filesystem.
2. The non-modifying NTFS diagnostic completed successfully.
3. The drive was subsequently visible at `/media/M1`.
4. The final mount options included `rw`, confirming that the filesystem was mounted in read/write mode.

**Important distinction:** A read/write mount does not by itself prove that every application can write to the drive. Application permissions and actual file operations must be checked separately.

## 6. Lessons Learned

This incident helped reinforce several Linux administration concepts:

- Checking mount status before attempting repairs.
- Understanding the difference between a block device and a mount point.
- Using filesystem diagnostic tools cautiously when backups are unavailable.
- Reading Linux mount options.
- Verifying system state after troubleshooting.
- Avoiding unnecessary filesystem modifications.

## 7. Future Improvements

- Establish a reliable backup strategy for external media drives.
- Document persistent mount configuration.
- Verify application-level read/write permissions.
- Monitor drive health and storage utilization.
- Record the exact remediation steps during future incidents.

## 8. Conclusion

This troubleshooting exercise provided practical experience diagnosing NTFS storage behavior on Ubuntu Linux.

By inspecting filesystem metadata and verifying mount status, I was able to confirm that the external drive was ultimately mounted with read/write access.

The incident reinforced the importance of using diagnostic commands, understanding their output, and verifying results rather than assuming a problem has been resolved.
