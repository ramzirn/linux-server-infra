# 06 — System Services & Storage Management

## 1. Objective

The objective of this phase is to inspect the server's system configuration, understand its storage architecture, review filesystem mount points, and increase the available disk capacity.

A basic inspection of system logs was also performed.

This phase introduces important Linux storage concepts, particularly Logical Volume Management (LVM), filesystem resizing, and persistent mount configuration.

## 2. Prerequisites

Before starting this phase, the following requirements should be met:

- An operational Ubuntu Server installation.
- Administrative access through a user account with sudo privileges.
- Basic familiarity with Linux storage and filesystem commands.
- A working system with no known storage-related errors.

**Important:** Storage modifications should always be preceded by an appropriate backup when working with important data. The operations described in this phase were performed in a laboratory virtual machine.

## 3. Default System Target

The default systemd target was checked using:

```bash
systemctl get-default
```

Result:

```text
graphical.target
```

The server is configured to boot into `graphical.target`, which includes the multi-user system environment and can also start the graphical login environment when the required components are installed.

For a dedicated server without a graphical desktop, `multi-user.target` is generally preferred because it starts the system in multi-user mode without requiring the graphical login environment.

No changes were made to the default target.

### Key Concepts

- **systemd:** The system and service manager used by Ubuntu.
- **Target:** A systemd unit used to group other units and define a system operating state.
- **graphical.target:** A target associated with multi-user operation and graphical login services.
- **multi-user.target:** A target providing multi-user operation without requiring a graphical login environment.

The existing configuration was inspected but left unchanged.

## 4. Disk Inspection

The disk layout was inspected using:

```bash
lsblk
```

### Initial Storage Architecture

| Device | Size | Description |
|---|---:|---|
| `/dev/sda` | 29.3 GB | Main virtual disk |
| `/dev/sda1` | 1 MB | Small boot-related partition |
| `/dev/sda2` | 2 GB | Boot partition |
| `/dev/sda3` | 27.3 GB | Partition containing the LVM physical volume |
| `ubuntu-vg/ubuntu-lv` | 13.64 GB | Root logical volume |

The system uses **LVM (Logical Volume Manager)** to manage its storage.

### Understanding the LVM Architecture

LVM separates storage management into several layers:

- **Physical Disk:** The virtual disk `/dev/sda`.
- **Physical Volume (PV):** The LVM partition `/dev/sda3`.
- **Volume Group (VG):** `ubuntu-vg`, which provides a pool of storage capacity.
- **Logical Volume (LV):** `ubuntu-lv`, which contains the root filesystem.

The relationship can be represented as follows:

```text
/dev/sda (29.3 GB)
│
├── /dev/sda1 (1 MB)
│
├── /dev/sda2 (2 GB)
│
└── /dev/sda3 (27.3 GB)
    │
    └── Physical Volume (PV)
        │
        └── Volume Group (VG): ubuntu-vg
            │
            └── Logical Volume (LV): ubuntu-lv
                │
                └── Root Filesystem (/)
```

LVM provides flexibility by allowing logical volumes to be resized without requiring the entire disk to be repartitioned.

## 5. Initial Disk Usage

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

**Note:** The values displayed by `df -h` are rounded. Used and available space do not necessarily add up to the total displayed capacity because of filesystem metadata and reserved space.

## 6. LVM Inspection

The volume group was inspected using:

```bash
sudo vgs
```

The logical volume was inspected using:

```bash
sudo lvs
```

### Initial LVM Configuration

| Component | Size |
|---|---:|
| Volume group (`ubuntu-vg`) | 27.29 GB |
| Root logical volume (`ubuntu-lv`) | 13.64 GB |
| Available volume group space | 13.65 GB |

The volume group had sufficient free space to extend the root logical volume without increasing the virtual disk size.

### Understanding LVM Capacity

Two different storage values must be considered:

- **Free space in the volume group:** Storage capacity available for creating or extending logical volumes.
- **Available filesystem space:** Space currently available for storing files inside a mounted filesystem.

The volume group had approximately 13.65 GB of unused capacity, while the root filesystem had approximately 7.3 GB available for storing additional data.

These values describe different storage layers.

## 7. Root Filesystem Extension

The root logical volume was extended from 13.64 GB to 24 GB.

The operation was performed in two steps.

### Step 1 — Extend the Logical Volume

```bash
sudo lvextend -L 24G /dev/ubuntu-vg/ubuntu-lv
```

The `lvextend` command increases the size of an existing logical volume.

The `-L 24G` option sets the target logical volume size to 24 GB.

The operation completed successfully.

### Step 2 — Resize the Filesystem

```bash
sudo resize2fs /dev/ubuntu-vg/ubuntu-lv
```

The `resize2fs` utility resizes an ext2/ext3/ext4 filesystem.

The ext4 filesystem was successfully resized online, without requiring a reboot.

No existing data was intentionally removed during this operation.

### Understanding the Two Operations

Extending an LVM logical volume and resizing its filesystem are two separate operations:

1. `lvextend` increases the storage capacity of the logical volume.
2. `resize2fs` expands the filesystem so that it can use the additional capacity.

Increasing the logical volume size alone does not automatically mean that the filesystem has been resized.

**Note:** `resize2fs` is appropriate here because the root filesystem uses ext4. Other filesystem types require their own resizing tools and procedures.

## 8. Filesystem and Mount Points

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

### Key Concepts

- **Mount Point:** A directory through which a filesystem is made accessible.
- **ext4:** A commonly used Linux filesystem.
- **tmpfs:** A temporary filesystem generally backed by memory and potentially swap.
- **proc:** A virtual filesystem exposing process and kernel information.
- **sysfs:** A virtual filesystem exposing information about devices and the kernel.
- **devtmpfs:** A filesystem providing device nodes.
- **SquashFS:** A compressed, read-only filesystem commonly used by Snap applications.

The `findmnt` command provides an overview of mounted filesystems and their relationships.

The listed mount points are the important examples identified during the inspection, not an exhaustive list of every mount in the system.

### 8.1 Persistent Mount Configuration

The `/etc/fstab` file was inspected:

```bash
cat /etc/fstab
```

The configuration contains entries for:

- The root filesystem (`/`), using its LVM device mapping.
- The `/boot` partition, identified through its UUID.
- The swap file (`/swap.img`).

The file was inspected but not modified.

### Understanding `/etc/fstab`

The `/etc/fstab` file defines filesystems and swap areas that should be configured during system initialization.

It allows the system to identify persistent storage using device paths, UUIDs, or other supported identifiers.

Inspecting this file helps understand how the system's filesystems are configured to be mounted.

## 9. System Logs

System journal disk usage was inspected using:

```bash
journalctl --disk-usage
```

Result:

```text
Archived and active journals take up 97M in the file system.
```

The system journals currently occupy approximately 97 MB.

### Key Concepts

The systemd journal collects system logs, including information about:

- System startup and shutdown.
- Service activity.
- Kernel messages.
- System errors and warnings.
- Application events logged through the journal.

No journal cleanup or retention configuration changes were performed.

## 10. Final Storage Verification

The root filesystem was checked again:

```bash
df -h /
```

### Final Filesystem Result

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

### Final LVM Result

| Component | Final value |
|---|---:|
| Volume group size | 27.29 GB |
| Root logical volume | 24 GB |
| Remaining free space | 3.29 GB |

The extension was successful, and the root filesystem can now use the additional capacity.

The volume group retains approximately 3.29 GB of unused capacity for potential future logical volume allocation.

## 11. Conclusion

Phase 06 was completed successfully.

The server's storage architecture was inspected, and the root logical volume was extended from 13.64 GB to 24 GB using LVM.

The final configuration provides approximately 17 GB of available root filesystem space, with 3.29 GB remaining free in the volume group.

Filesystem mount points, persistent mount configuration, the default systemd target, and system journal usage were also reviewed.

No unnecessary system service or filesystem configuration changes were introduced.

The server now has additional storage capacity for the upcoming Docker infrastructure phase.