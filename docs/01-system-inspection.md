# 01 - System Inspection & Initial Assessment

## 1. Overview

This document describes the initial inspection and assessment of the Linux server before any system configuration or infrastructure deployment.

The purpose of this phase is to understand the existing server environment, identify available resources, inspect network connectivity, and establish an initial system baseline.

This assessment provides the necessary information to prepare the server for the upcoming configuration, security hardening, and application hosting phases.

## 2. Objectives

The main objectives of this phase are:

- Identify the installed Linux distribution and kernel version.
- Inspect the available CPU and memory resources.
- Understand the disk and partition layout.
- Identify network interfaces and IP configuration.
- Inspect active system services and processes.
- Identify listening network ports.
- Validate network connectivity and DNS resolution.
- Establish an initial understanding of the server environment.

## 3. Operating System & Kernel Inspection

### Commands

```bash
cat /etc/os-release
uname -r
hostnamectl
```

### Purpose

- `cat /etc/os-release`: Identifies the Linux distribution and its version.
- `uname -r`: Displays the currently running Linux kernel version.
- `hostnamectl`: Displays system information, including hostname, operating system, kernel, and hardware architecture.

### Observations

These commands provide an overview of the installed operating system and the current system identity.

The information collected will help identify the software environment used throughout the project.

## 4. CPU & Memory Inspection

### Commands

```bash
lscpu
nproc
free -h
top
```

### Purpose

- `lscpu`: Displays CPU architecture, processor model, cores, threads, and virtualization information.
- `nproc`: Returns the number of available processing units.
- `free -h`: Displays RAM and swap usage in a human-readable format.
- `top`: Provides real-time information about running processes and resource consumption.

### Observations

CPU and memory inspection helps evaluate the resources available to the server.

These resources will be important when deploying multiple Docker containers and monitoring services.

The available memory (`available`) is particularly useful when evaluating how much RAM can be allocated to future workloads.

## 5. Storage Inspection

### Commands

```bash
df -h
lsblk
lsblk -f
sudo du -sh /var /home /opt /tmp
```

### Purpose

- `df -h`: Displays filesystem capacity, used space, and available space.
- `lsblk`: Lists detected block devices, disks, and partitions.
- `lsblk -f`: Displays filesystem types, UUIDs, and mount points.
- `du -sh`: Estimates the disk space used by selected directories.

### Key Concepts

**Block Devices:** Devices that provide block-based storage access, such as disks and partitions.

**Partitions:** Logical divisions of a storage device.

**Filesystems:** Structures used by Linux to organize and store files, such as ext4 and XFS.

**Mount Points:** Directories where filesystems are attached and made accessible.

**Loop Devices:** Virtual block devices that allow files to be mounted as filesystems. Ubuntu commonly uses them for Snap application images.

### Observations

The storage inspection provides an overview of the virtual disk, its partitions, filesystem types, and available capacity.

Understanding the storage layout is important before deploying Docker, which uses disk space to store images, containers, volumes, and logs.

## 6. Network Inspection

### Commands

```bash
ip -br addr
ip route
resolvectl status
```

### Purpose

- `ip -br addr`: Displays network interfaces and their assigned IP addresses in a concise format.
- `ip route`: Displays the system routing table and default gateway.
- `resolvectl status`: Displays DNS configuration and resolver information.

### Key Concepts

**Network Interface:** A logical or physical interface used for network communication.

**IP Address:** An address identifying a device on an IP network.

**Default Gateway:** The router used to reach destinations outside the local network.

**DNS:** A service that translates domain names into IP addresses.

### Observations

The network inspection identifies how the server communicates with its local network and external destinations.

The current VirtualBox networking mode will also be considered during future network configuration.

## 7. System Services & Processes Inspection

### Commands

```bash
systemctl --type=service --state=running
ps aux
pstree -p
```

### Purpose

- `systemctl`: Lists currently running system services managed by systemd.
- `ps aux`: Displays running processes and their resource information.
- `pstree -p`: Displays running processes in a hierarchical tree structure.

### Observations

This inspection helps identify the services and processes currently running on the server.

It also provides an initial understanding of the services that may already occupy system resources or network ports.

## 8. Listening Ports Inspection

### Command

```bash
sudo ss -tulnp
```

### Purpose

Displays listening TCP and UDP sockets, including associated processes when available.

Options:

- `-t`: TCP sockets.
- `-u`: UDP sockets.
- `-l`: Listening sockets.
- `-n`: Numeric addresses and ports.
- `-p`: Associated processes.

### Observations

Identifying listening ports is important for understanding which services are currently exposed or accepting network connections.

This information will be useful when configuring firewall policies and deploying applications.

## 9. Network Connectivity Validation

### Commands

```bash
ping -c 4 8.8.8.8
ping -c 4 google.com
curl -I https://google.com
```

### Purpose

**IP Connectivity Test**

```bash
ping -c 4 8.8.8.8
```

Tests connectivity to a public IP address without requiring DNS resolution.

**DNS Resolution Test**

```bash
ping -c 4 google.com
```

Tests domain name resolution and attempts to communicate with the resolved IP address.

**HTTP/HTTPS Connectivity Test**

```bash
curl -I https://google.com
```

Tests application-level connectivity by establishing an HTTPS request and displaying HTTP response headers.

### Results

The connectivity tests completed successfully.

The server demonstrated:

- Successful communication with a public IP address.
- Successful DNS resolution.
- Successful HTTP/HTTPS connectivity.

## 10. Initial Assessment Summary

The initial inspection phase established a baseline understanding of the Linux server.

The following areas were examined:

| Area | Assessment |
|---|---|
| Operating System | Inspected |
| Kernel Version | Inspected |
| CPU Resources | Inspected |
| Memory Resources | Inspected |
| Disk & Partitions | Inspected |
| Network Interfaces | Inspected |
| Routing & DNS | Inspected |
| Running Services | Inspected |
| Listening Ports | Inspected |
| Network Connectivity | Validated |

Detailed hardware specifications and system configuration values can be recorded separately as the infrastructure documentation evolves.

## 11. Conclusion

The initial system inspection is complete.

The Linux server has been assessed from an operating system, hardware, storage, networking, and service-management perspective.

Network connectivity has also been successfully validated.

This phase establishes the baseline required before making system-level changes.

The next phase will focus on initial Linux system configuration, including hostname configuration, timezone settings, system updates, and essential administration utilities.

