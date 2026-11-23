# 03 — System Inspection & Initial Assessment

## 1. Objective

The objective of this phase is to inspect and assess the Linux server before proceeding with infrastructure configuration and deployment.

This phase establishes an initial system baseline by identifying available resources, understanding the existing environment, inspecting network configuration, and validating connectivity.

System inspection is an important administration task because it helps identify resource constraints, existing services, storage limitations, and potential network conflicts before introducing new workloads.

## 2. Prerequisites

Before starting this phase, the following requirements should be met:

- An Ubuntu Server virtual machine installed and accessible.
- Basic system configuration completed as described in Phase 02.
- An active network connection.
- A user account with `sudo` privileges for commands requiring administrative access.

## 3. Objectives

The main objectives of this phase are:

- Identify the installed Linux distribution and kernel version.
- Inspect available CPU and memory resources.
- Understand the disk and partition layout.
- Identify network interfaces and IP configuration.
- Inspect active system services and processes.
- Identify listening network ports.
- Validate network connectivity and DNS resolution.
- Establish an initial understanding of the server environment.

## 4. Operating System & Kernel Inspection

### Commands

```bash
cat /etc/os-release
uname -r
hostnamectl
```

### Purpose

| Command | Purpose |
|---|---|
| `cat /etc/os-release` | Identifies the Linux distribution and version |
| `uname -r` | Displays the currently running Linux kernel version |
| `hostnamectl` | Displays system information, including hostname, operating system, kernel, and hardware architecture |

### Key Concepts

- **Linux Distribution:** The operating system built around the Linux kernel, including system utilities, package management, and other software.
- **Kernel:** The core component responsible for managing hardware resources, processes, memory, and system operations.
- **Architecture:** Describes the processor architecture used by the system, such as x86_64.

These commands provide an overview of the installed operating system and system identity, establishing the software environment used throughout the project.

## 5. CPU & Memory Inspection

### Commands

```bash
lscpu
nproc
free -h
top
```

### Purpose

| Command | Purpose |
|---|---|
| `lscpu` | Displays CPU architecture, processor model, cores, threads, and virtualization information |
| `nproc` | Returns the number of processing units available to the current process |
| `free -h` | Displays RAM and swap usage in a human-readable format |
| `top` | Provides real-time information about running processes and resource consumption |

### Key Concepts

- **CPU:** The processor responsible for executing instructions.
- **Core:** An individual processing unit within a CPU.
- **Thread:** A logical processing unit exposed by the processor.
- **RAM:** Temporary memory used by the operating system and running applications.
- **Swap:** Disk space used as an extension of memory when physical RAM is under pressure.

### Interpreting Memory Information

The `free -h` command displays several memory values.

The `available` column is particularly important because it estimates how much memory can be made available for new applications without excessive swapping.

This value is generally more useful than looking only at the `free` column, since Linux uses available memory for filesystem caches.

CPU and memory inspection helps evaluate the resources available to the server before deploying workloads.

## 6. Storage Inspection

### Commands

```bash
df -h
lsblk
lsblk -f
sudo du -sh /var /home /opt /tmp
```

### Purpose

| Command | Purpose |
|---|---|
| `df -h` | Displays filesystem capacity, used space, and available space |
| `lsblk` | Lists detected block devices, disks, and partitions |
| `lsblk -f` | Displays filesystem types, UUIDs, and mount points |
| `du -sh` | Estimates disk space usage for selected directories |

### Key Concepts

- **Block Devices:** Devices providing block-based storage access, such as disks and partitions.
- **Partitions:** Logical divisions of a storage device.
- **Filesystems:** Structures used by Linux to organize and store files, such as ext4 and XFS.
- **Mount Points:** Directories where filesystems are attached and made accessible.
- **Loop Devices:** Virtual block devices that allow files to be mounted as filesystems. Ubuntu commonly uses them for Snap application images.

### Understanding the Difference Between Storage Commands

`lsblk` provides an overview of the storage device hierarchy, while `df -h` reports filesystem capacity and usage.

The `du -sh` command estimates the amount of disk space occupied by selected directories.

These commands provide complementary information:

- `lsblk`: What storage devices and partitions exist?
- `df -h`: How much filesystem space is available?
- `du -sh`: Which directories occupy disk space?

Storage inspection provides an overview of the virtual disk, its partitions, filesystem types, and available capacity.

Understanding the storage layout is essential before deploying Docker, which uses disk space to store images, containers, volumes, and logs.

**Note:** Directory sizes reported by `du` do not necessarily match filesystem usage reported by `df`. Filesystem metadata, reserved space, open deleted files, and mounted filesystems can contribute to differences.

## 7. Network Inspection

### Commands

```bash
ip -br addr
ip route
resolvectl status
```

### Purpose

| Command | Purpose |
|---|---|
| `ip -br addr` | Displays network interfaces and their assigned IP addresses in a concise format |
| `ip route` | Displays the system routing table and default gateway |
| `resolvectl status` | Displays DNS configuration and resolver information |

### Key Concepts

- **Network Interface:** A logical or physical interface used for network communication.
- **IP Address:** An address identifying a device on an IP network.
- **Default Gateway:** The router used to reach destinations outside the local network.
- **DNS:** A service that translates domain names into IP addresses.
- **Routing Table:** Information used by the operating system to determine where network packets should be sent.

These commands help identify how the server communicates with its local network and external destinations.

The existing VirtualBox NAT configuration is also taken into account during subsequent network configuration.

## 8. System Services & Processes Inspection

### Commands

```bash
systemctl --type=service --state=running
ps aux
pstree -p
```

### Purpose

| Command | Purpose |
|---|---|
| `systemctl --type=service --state=running` | Lists currently running system services managed by systemd |
| `ps aux` | Displays running processes and their resource information |
| `pstree -p` | Displays running processes in a hierarchical tree structure, including process IDs |

### Key Concepts

- **Service:** A background system component managed by a service manager such as systemd.
- **Process:** An instance of a running program identified by a process ID (PID).
- **PID:** A unique process identifier assigned by the Linux kernel.
- **systemd:** The system and service manager used by Ubuntu.

A service and a process are related but are not the same concept. A systemd service may manage one or more processes.

This inspection identifies the services and processes currently running on the server and helps detect existing resource consumption or services that may occupy network ports.

## 9. Listening Ports Inspection

### Command

```bash
sudo ss -tulnp
```

The `ss` utility displays network socket information.

This command displays listening TCP and UDP sockets, including associated processes when available.

### Command Options

| Option | Meaning |
|---|---|
| `-t` | TCP sockets |
| `-u` | UDP sockets |
| `-l` | Listening sockets |
| `-n` | Numeric addresses and ports |
| `-p` | Associated processes, when available |

### Key Concepts

- **Port:** A logical communication endpoint used by network services.
- **Listening Socket:** A socket waiting for incoming connections or datagrams.
- **TCP:** A connection-oriented transport protocol.
- **UDP:** A connectionless transport protocol.

Identifying listening ports helps determine which services are accepting network connections.

This information is useful when defining firewall policies and deploying applications to avoid port conflicts and unintended network exposure.

**Important:** A listening socket does not automatically mean that a service is accessible from another machine. Actual accessibility also depends on the interface binding, firewall rules, network routing, and virtualization configuration.

## 10. Network Connectivity Validation

Network connectivity was tested using three complementary methods.

### 10.1 IP Connectivity Test

```bash
ping -c 4 8.8.8.8
```

This test attempts to communicate with a public IP address without requiring DNS resolution.

It helps determine whether external IP connectivity is available.

### 10.2 DNS Resolution Test

```bash
ping -c 4 google.com
```

This test checks domain name resolution and attempts to communicate with the resolved IP address.

It helps validate both DNS resolution and network connectivity.

### 10.3 HTTP/HTTPS Connectivity Test

```bash
curl -I https://google.com
```

The `curl` utility sends an HTTP request and displays the response headers.

The `-I` option requests response headers without downloading the full response body.

This test checks application-level connectivity over HTTPS.

### Results

All connectivity tests completed successfully, confirming:

- Successful communication with a public IP address.
- Successful DNS resolution.
- Successful HTTP/HTTPS connectivity.

**Note:** Some external services block ICMP traffic. A failed ping does not necessarily mean that network connectivity is unavailable. HTTP/HTTPS tests can provide additional information.

## 11. Initial Assessment and Interpretation

The previous inspection commands provide complementary information about the server.

The following points should be considered when evaluating the environment:

| Area | Questions to Consider |
|---|---|
| Operating System | Is the expected Ubuntu version installed? |
| CPU | How many processing units are available? |
| Memory | Is enough available RAM left for future services? |
| Storage | Is sufficient disk capacity available for Docker images, containers, and monitoring data? |
| Network | Are the expected interfaces, IP addresses, and routes configured? |
| Services | Are there existing services that should be understood before deployment? |
| Listening Ports | Are any ports already occupied or unexpectedly exposed? |
| Connectivity | Can the server reach external IP addresses and resolve domain names? |

This assessment provides a baseline for understanding the server's capabilities and limitations before deploying additional infrastructure components.

## 12. Result

The initial system inspection is complete.

The server has been assessed across the following areas:

- Operating system and kernel.
- CPU and memory resources.
- Disk, partitions, and filesystems.
- Network interfaces, routing, and DNS configuration.
- Active system services and processes.
- Listening network ports.
- External network and DNS connectivity.

This assessment establishes the initial system baseline required before proceeding with infrastructure configuration and deployment.

The information collected during this phase can also serve as a reference when evaluating the impact of later infrastructure changes.