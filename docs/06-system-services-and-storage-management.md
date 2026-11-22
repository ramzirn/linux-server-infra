# Phase 06 — System Services & Storage Management

## 1. Objective

The objective of this phase was to inspect the server's system configuration, understand its storage architecture, review filesystem mount points, and improve the available disk capacity.

This phase also included a basic inspection of system logs.

## 2. Default System Target

The default systemd target was checked using:

```bash
systemctl get-default
```

Result:

```text
graphical.target
```

The server currently boots into `graphical.target`, which provides a graphical environment.

For a dedicated production server, `multi-user.target` is generally preferred because it starts the system without a graphical desktop environment.

No changes were made to the default target.

## 3. Disk Inspection

The disk layout was inspected using:

```bash
lsblk
```

Initial storage architecture:

| Device | Size | Description |
|---|---:|---|
| `/dev/sda` | 29.3 GB | Main virtual disk |
| `/dev/sda1` | 1 MB | Technical boot partition |
| `/dev/sda2` | 2 GB | Boot partition |
| `/dev/sda3` | 27.3 GB | Partition containing the LVM physical volume |
| `ubuntu-vg/ubuntu-lv` | 13.64 GB | Root logical volume |

The system uses LVM (Logical Volume Manager) to manage its storage.

## 4. Initial Disk Usage

Disk usage was inspected using:

```bash
df -h
```

Before the storage extension, the root filesystem had the following capacity:

| Metric | Initial value |
|---|---:|
| Total size | 14 GB |
| Used space | 5.5 GB |
| Available space | 7.3 GB |
| Usage | 43% |

The initial root filesystem capacity was limited for future application hosting and Docker workloads.

## 5. LVM Inspection

The volume group was inspected using:

```bash
sudo vgs
```

The logical volume was inspected using:

```bash
sudo lvs
```

Initial LVM configuration:

| Component | Size |
|---|---:|
| Volume group (`ubuntu-vg`) | 27.29 GB |
| Root logical volume (`ubuntu-lv`) | 13.64 GB |
| Available volume group space | 13.65 GB |

The volume group had sufficient free space to extend the root logical volume without increasing the virtual disk size.

## 6. Root Filesystem Extension

The root logical volume was extended from 13.64 GB to 24 GB.

### Step 1 — Extend the logical volume

```bash
sudo lvextend -L 24G /dev/ubuntu-vg/ubuntu-lv
```

The operation completed successfully.

### Step 2 — Resize the filesystem

```bash
sudo resize2fs /dev/ubuntu-vg/ubuntu-lv
```

The ext4 filesystem was successfully resized online, without requiring a reboot.

No existing data was intentionally removed during this operation.

## 7. Filesystem and Mount Points

The mounted filesystems were inspected using:

```bash
findmnt
```

Important mount points identified:

| Mount Point | Filesystem | Purpose |
|---|---|---|
| `/` | ext4 | Main root filesystem |
| `/boot` | ext4 | Boot files |
| `/run` | tmpfs | Temporary runtime data |
| `/tmp` | tmpfs | Temporary files |
| `/dev` | devtmpfs | Device management |
| `/proc` | proc | Process and kernel information |
| `/sys` | sysfs | Kernel and hardware information |

Snap applications also use read-only SquashFS mounts under `/snap`.

### Persistent Mount Configuration

The `/etc/fstab` file was inspected:

```bash
cat /etc/fstab
```

The configuration contains entries for:

- The root filesystem (`/`), identified through its LVM device ID.
- The `/boot` partition, identified through its UUID.
- The swap file (`/swap.img`).

The file was inspected but not modified.

## 8. System Logs

System journal disk usage was inspected using:

```bash
journalctl --disk-usage
```

Result:

```text
Archived and active journals take up 97M in the file system.
```

The system journals currently occupy approximately 97 MB.

No journal cleanup or retention configuration changes were performed.

## 9. Final Storage Verification

The root filesystem was checked again:

```bash
df -h /
```

Final result:

| Metric | Final value |
|---|---:|
| Total size | 24 GB |
| Used space | 5.5 GB |
| Available space | 17 GB |
| Usage | 25% |

The volume group was also checked:

```bash
sudo vgs
```

Final result:

| Component | Final value |
|---|---:|
| Volume group size | 27.29 GB |
| Root logical volume | 24 GB |
| Remaining free space | 3.29 GB |

The extension was successful, and the root filesystem can now use the additional capacity.

## 10. Conclusion

Phase 06 was completed successfully.

The server's storage architecture was inspected, and the root logical volume was extended from 13.64 GB to 24 GB using LVM.

The final configuration provides approximately 17 GB of available root filesystem space, with 3.29 GB remaining free in the volume group.

The filesystem mount configuration and system journal usage were also reviewed.

No unnecessary system service or filesystem configuration changes were introduced.

The server is now better prepared for the upcoming Docker infrastructure phase.