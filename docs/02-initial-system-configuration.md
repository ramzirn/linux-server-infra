# 02 — Initial System Configuration

## 1. Objective

The objective of this phase is to establish a properly configured and maintainable Ubuntu Server environment before proceeding with system inspection, user management, security hardening, and infrastructure deployment.

This phase introduces essential Linux administration tasks, including hostname configuration, timezone management, package maintenance, administration tools, and automatic security updates.

## 2. Prerequisites

Before starting this phase, the following requirements should be met:

- An Ubuntu Server virtual machine installed and accessible.
- Administrative privileges through a user account with `sudo` access.
- An active network connection.
- Basic familiarity with Linux terminal commands.

The server should already have basic network connectivity established, as described in Phase 01.

## 3. Objectives

The main objectives of this phase are:

- Configure a meaningful server hostname.
- Set the correct system timezone.
- Update the operating system.
- Install essential system administration tools.
- Enable automatic security updates.

## 4. Hostname Configuration

The server hostname was configured as:

```text
linux-server
```

A hostname identifies a machine across the operating system and network environment. It also makes the server easier to recognize in terminal sessions and administration tools.

### Configuration

```bash
sudo hostnamectl set-hostname linux-server
```

### Verification

```bash
hostnamectl
hostname
```

The hostname was confirmed as `linux-server`.

**Note:** Changing the hostname does not automatically configure DNS records or guarantee that the machine can be reached through that name from other devices.

## 5. Timezone Configuration

The server timezone was configured to use the local Algerian timezone:

```text
Africa/Algiers
```

A timezone determines how system timestamps are displayed locally. Correct timezone configuration is useful for interpreting logs, scheduled tasks, and monitoring events.

### Configuration

```bash
sudo timedatectl set-timezone Africa/Algiers
```

### Verification

```bash
timedatectl
```

The output can be used to verify the configured timezone and current system clock information.

**Important distinction:** The timezone controls how timestamps are represented locally. It does not independently synchronize the system clock. Time synchronization is handled by separate time synchronization services.

## 6. System Updates and Package Maintenance

Ubuntu uses APT (Advanced Package Tool) to manage software packages.

APT provides commands to refresh package information, install updates, and remove unnecessary dependencies.

### 6.1 Refresh the package index

The package index was refreshed using:

```bash
sudo apt update
```

This command downloads the latest package information from the configured repositories.

It does not install or upgrade packages by itself.

### 6.2 Apply available package upgrades

Available package upgrades were checked and applied using:

```bash
sudo apt upgrade
```

No package upgrades were required at the time of execution, indicating that the installed packages were already up to date according to the configured repositories.

### 6.3 Remove unused dependencies

Unused dependencies were checked and removed using:

```bash
sudo apt autoremove
```

This command identifies automatically installed packages that are no longer required by installed packages.

Review the proposed removals before confirming the operation.

**Important:** Package availability and update results depend on the configured repositories and their current state.

## 7. Essential Administration Tools

The following administration utilities were installed:

```bash
sudo apt install curl wget git vim htop tree net-tools unzip
```

These tools provide basic utilities required for day-to-day server administration and infrastructure management.

| Package | Purpose |
|---|---|
| `curl` | Performing HTTP requests and transferring resources |
| `wget` | Downloading files from remote servers |
| `git` | Version control and source code management |
| `vim` | Terminal-based text editing |
| `htop` | Interactive process and resource monitoring |
| `tree` | Visualizing directory structures |
| `net-tools` | Providing traditional network utilities such as `ifconfig` and `netstat` |
| `unzip` | Extracting ZIP archives |

### Basic verification

The installed utilities can be checked using:

```bash
which curl wget git vim htop tree ifconfig unzip
```

For example, the following commands demonstrate some of their basic uses:

```bash
htop
```

```bash
tree
```

```bash
git --version
```

These tools will be useful throughout the subsequent infrastructure phases.

## 8. Automatic Security Updates

Keeping the operating system updated is an important part of Linux server maintenance.

Ubuntu provides the `unattended-upgrades` package to automate the installation of updates from configured package origins.

### 8.1 Install unattended-upgrades

The package was installed using:

```bash
sudo apt install unattended-upgrades
```

### 8.2 Enable automatic updates

Automatic updates were enabled using:

```bash
sudo dpkg-reconfigure -plow unattended-upgrades
```

This command launches the package configuration interface, allowing automatic updates to be enabled.

### 8.3 Inspect the periodic APT configuration

The APT periodic configuration was inspected:

```bash
cat /etc/apt/apt.conf.d/20auto-upgrades
```

The configuration controls periodic package-list refreshes and unattended upgrades.

A typical enabled configuration contains:

```text
APT::Periodic::Update-Package-Lists "1";
APT::Periodic::Unattended-Upgrade "1";
```

The value `1` enables the corresponding daily periodic operation.

### 8.4 Understand update policies

The package origins eligible for unattended upgrades are controlled separately through:

```bash
cat /etc/apt/apt.conf.d/50unattended-upgrades
```

This file defines which repository origins are allowed to provide automatic updates.

Automatic updates are therefore controlled by both:

- `/etc/apt/apt.conf.d/20auto-upgrades`: periodic update scheduling.
- `/etc/apt/apt.conf.d/50unattended-upgrades`: eligible package origins and update policies.

**Important:** Enabling unattended upgrades does not mean that every available package update will automatically be installed. Actual behavior depends on the configured origins and policies.

## 9. Validation

The following commands were used to validate the system configuration and package state.

### 9.1 Hostname

```bash
hostnamectl
hostname
```

Expected hostname:

```text
linux-server
```

### 9.2 Timezone

```bash
timedatectl
```

Expected timezone:

```text
Africa/Algiers
```

### 9.3 Package state

```bash
sudo apt update
sudo apt upgrade
apt list --upgradable
```

These commands allow administrators to inspect package repository information and determine whether upgrades are available.

### 9.4 Administration utilities

The installed tools can be checked with:

```bash
which curl wget git vim htop tree ifconfig unzip
```

### 9.5 Automatic update configuration

```bash
dpkg -l unattended-upgrades
```

```bash
cat /etc/apt/apt.conf.d/20auto-upgrades
```

The configuration of allowed package origins can also be inspected in:

```bash
cat /etc/apt/apt.conf.d/50unattended-upgrades
```

## 10. Troubleshooting

| Problem | Possible cause | Suggested check |
|---|---|---|
| APT cannot reach repositories | Network or DNS connectivity issue | Check network connectivity and DNS resolution |
| Package installation fails | Repository configuration or package availability | Run `sudo apt update` and inspect the error |
| Hostname does not appear as expected | Configuration not applied or incorrect hostname | Run `hostnamectl` |
| Incorrect displayed local time | Incorrect timezone configuration | Run `timedatectl` |
| Automatic updates are not running | Incorrect periodic configuration or package origin policy | Inspect `20auto-upgrades` and `50unattended-upgrades` |

## 11. Result

The initial system configuration is complete.

The server now has:

- A clearly defined hostname: `linux-server`.
- The configured local timezone: `Africa/Algiers`.
- An up-to-date package state at the time of execution.
- Essential system administration utilities.
- The `unattended-upgrades` package installed and automatic updates enabled.

This establishes a stable operational baseline for the subsequent infrastructure configuration phases.