# 02 - Initial System Configuration

## 1. Overview

This phase covers the initial configuration of the Linux server after completing the system inspection.

The objective is to establish a clean, properly configured and maintainable server environment before moving on to user management, networking and security hardening.

## 2. Objectives

The main objectives of this phase were:

- Configure a meaningful server hostname.
- Set the correct system timezone.
- Update the operating system.
- Install essential system administration tools.
- Enable automatic security updates.

## 3. Hostname Configuration

The server hostname was configured as:

```text
linux-server
```

The hostname identifies the machine across the system and helps administrators recognize it in terminal sessions and network environments.

Command used:

```bash
sudo hostnamectl set-hostname linux-server
```

Configuration was verified with:

```bash
hostnamectl
hostname
```

## 4. Timezone Configuration

The server timezone was configured to use the local Algerian timezone:

```text
Africa/Algiers
```

Command used:

```bash
sudo timedatectl set-timezone Africa/Algiers
```

Configuration was verified with:

```bash
timedatectl
```

Correct timezone configuration is important for system logs, scheduled tasks and monitoring.

## 5. System Updates and Package Maintenance

The APT package index was refreshed:

```bash
sudo apt update
```

Installed packages were checked for available updates:

```bash
sudo apt upgrade
```

No package upgrades were required at the time of execution. This indicated that the installed packages were already up to date according to the configured repositories.

Unused dependencies were also checked and cleaned up using:

```bash
sudo apt autoremove
```

## 6. Essential Administration Tools

The following administration utilities were installed:

```bash
sudo apt install curl wget git vim htop tree net-tools unzip
```

Their main purposes are:

| Package | Purpose |
|---|---|
| curl | HTTP requests and resource transfers |
| wget | Downloading files from remote servers |
| git | Version control and project management |
| vim | Terminal-based text editing |
| htop | Interactive process and resource monitoring |
| tree | Directory structure visualization |
| net-tools | Traditional network utilities |
| unzip | Extracting ZIP archives |

## 7. Automatic Security Updates

The `unattended-upgrades` package was installed to automate the installation of configured updates, including security updates.

Installation:

```bash
sudo apt install unattended-upgrades
```

Automatic updates were enabled using:

```bash
sudo dpkg-reconfigure -plow unattended-upgrades
```

The configuration was inspected with:

```bash
cat /etc/apt/apt.conf.d/20auto-upgrades
```

The APT periodic configuration controls package-list refreshes and unattended upgrades.

Automatic updates apply according to the configured package origins and update policies. They do not necessarily install every available type of update.

## 8. Validation

The following commands were used to validate the system configuration:

```bash
hostnamectl
timedatectl
sudo apt update
sudo apt upgrade
apt list --upgradable
```

The installed administration utilities and unattended-upgrades configuration were also checked.

## 9. Result

The initial system configuration is complete.

The server now has:

- A clearly defined hostname.
- The correct local timezone.
- An updated package state.
- Essential administration utilities.
- Automatic security updates enabled.

These configurations provide a stable starting point for the next stages of server administration.

## 10. Conclusion

This phase established the initial operational configuration of the Linux server.

The next phase will focus on user and access management, including administrative privileges and the principles of least privilege.