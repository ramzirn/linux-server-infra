# 05 — Security Hardening

## 1. Objective

The objective of this phase is to strengthen the Ubuntu server's security by applying system updates, enabling automatic security maintenance, verifying security mechanisms, reducing unnecessary services, and protecting remote SSH access.

The goal is to reduce the server's attack surface and establish a stronger security baseline before deploying additional infrastructure components.

Security hardening is an ongoing process. The measures implemented in this phase provide a foundation but do not replace continuous maintenance, monitoring, and security reviews.

## 2. Prerequisites

Before starting this phase, the following requirements should be met:

- An operational Ubuntu Server installation.
- Administrative access through a user account with sudo privileges.
- SSH access configured as described in Phase 01.
- Basic system configuration completed as described in the previous phases.
- An active network connection.

## 3. Security Audit

An initial security assessment was performed using:

```bash
ubuntu-security-status
```

### Observed Results

- Ubuntu 26.04.1 LTS.
- Main and Restricted packages receive security maintenance until 2031.
- Ubuntu Pro is not enabled.
- Extended Security Maintenance (ESM) is available through Ubuntu Pro.

### Key Concepts

- **Ubuntu Security Maintenance:** Security updates provided for supported Ubuntu package repositories during the applicable maintenance period.
- **Ubuntu Pro:** An optional Ubuntu subscription that provides additional security and maintenance coverage.
- **ESM (Extended Security Maintenance):** Extended security maintenance available through Ubuntu Pro for eligible packages and components.

Ubuntu Pro is not required for the current laboratory environment.

The initial assessment confirmed that the system receives standard security maintenance during its applicable support period.

## 4. System Updates & Security Patches

Package repositories were refreshed:

```bash
sudo apt update
```

This command downloads the latest package information from the configured repositories.

Available package upgrades were inspected:

```bash
apt list --upgradable
```

System packages were upgraded:

```bash
sudo apt upgrade
```

A kernel update required a system reboot:

```bash
sudo reboot
```

Remote SSH access was successfully restored after reboot, confirming that the server remained accessible following the kernel update.

### Key Concepts

- **APT:** Ubuntu's package management system.
- **Package Update:** Refreshes the local package metadata.
- **Package Upgrade:** Installs available newer versions of installed packages.
- **Kernel Update:** Installs a newer Linux kernel version containing improvements, fixes, or security patches.

Regular system updates are essential for reducing exposure to known software vulnerabilities.

## 5. Automatic Security Updates

Automatic security maintenance was inspected to determine whether the system was configured to install eligible package updates without manual intervention.

### 5.1 Package Verification

The `unattended-upgrades` package was already installed.

Verification:

```bash
dpkg -l unattended-upgrades
```

The package provides automatic installation of eligible package upgrades according to the configured APT policies.

### 5.2 Automatic Updates Configuration

Configuration file:

`/etc/apt/apt.conf.d/20auto-upgrades`

Current configuration:

```text
APT::Periodic::Update-Package-Lists "1";
APT::Periodic::Unattended-Upgrade "1";
```

This configuration enables:

- Daily package-list updates.
- Daily unattended upgrade processing.

These settings enable the periodic APT maintenance mechanism. They do not, by themselves, define which package origins are eligible for installation.

### 5.3 Security Update Origins

Configuration file:

`/etc/apt/apt.conf.d/50unattended-upgrades`

Configured allowed origins include:

```text
"${distro_id}:${distro_codename}";
"${distro_id}:${distro_codename}-security";
"${distro_id}ESMApps:${distro_codename}-apps-security";
"${distro_id}ESM:${distro_codename}-infra-security";
```

These origins define which package repositories are eligible for unattended upgrades.

Proposed and backports origins are not explicitly enabled in the listed configuration.

The ESM entries are available as configured package origins, but their actual use depends on the availability of the corresponding repositories and Ubuntu Pro entitlement.

### 5.4 Automatic Upgrade Simulation

Automatic upgrades were tested using:

```bash
sudo unattended-upgrade --dry-run --debug
```

The simulation stopped with:

```text
System is on battery power, stopping
upgrade result: False
```

Investigation showed:

- `BAT0` status: `Discharging`.
- `AC` online status: `0`.

This behavior is related to power detection in the VirtualBox laptop environment.

No configuration changes were made to bypass this restriction.

**Important:** Automatic update settings are enabled, but the simulation could not be completed because of the detected power state. Successful automatic upgrade execution has therefore not been confirmed through this test.

Automatic reboot was not explicitly enabled.

## 6. AppArmor

AppArmor provides Mandatory Access Control (MAC), restricting application capabilities according to security profiles.

Unlike traditional discretionary permissions, AppArmor applies additional security policies to processes based on their assigned profiles.

### 6.1 Service Verification

Service status was checked using:

```bash
sudo systemctl status apparmor
```

**Result:** The AppArmor service is active.

### 6.2 Profile Verification

Loaded profiles and enforcement modes were inspected using:

```bash
sudo aa-status
```

### Observed Results

- 186 profiles loaded.
- 110 profiles in enforce mode.
- 2 profiles in complain mode.
- 74 profiles in unconfined mode.
- 3 processes running with enforced profiles.

### Understanding AppArmor Modes

- **Enforce:** The profile is actively enforced. Operations that violate its policy are denied.
- **Complain:** Policy violations are logged, but not blocked by the profile.
- **Unconfined:** The process is not restricted by an enforcing AppArmor profile.

AppArmor is operational and enforcing security policies for processes covered by enforced profiles.

Docker-specific AppArmor profiles were not present because Docker had not yet been installed.

## 7. Services & Attack Surface Reduction

Enabled and currently running services were inspected using:

```bash
systemctl list-unit-files --type=service --state=enabled
```

```bash
systemctl --type=service --state=running
```

These commands provide different information:

- The first lists service unit files configured to start automatically.
- The second lists services currently running.

The `ModemManager` service was identified as unnecessary for this server environment.

### 7.1 ModemManager

ModemManager manages mobile broadband devices and cellular modems.

Since the virtual machine does not require mobile modem connectivity, the service was disabled and stopped:

```bash
sudo systemctl disable --now ModemManager.service
```

The `disable` option prevents the service from starting automatically, while `--now` stops it immediately.

Other services, including `multipathd`, were inspected but left unchanged to avoid unnecessary modifications to system components.

Reducing unnecessary services helps limit the number of components that could potentially introduce security vulnerabilities or expose network interfaces.

## 8. SSH & Brute-force Protection

SSH provides remote administration and is therefore an important security component.

The SSH authentication configuration established during Phase 01 was complemented with Fail2ban to provide additional protection against repeated authentication failures.

### 8.1 Fail2ban Installation

Fail2ban was not initially installed.

Installation:

```bash
sudo apt install fail2ban
```

Service verification:

```bash
sudo systemctl status fail2ban
```

**Result:** The Fail2ban service is active.

### 8.2 SSH Jail Verification

The active Fail2ban jails were inspected using:

```bash
sudo fail2ban-client status
```

**Observed Result:**

```text
Status
|- Number of jail: 1
`- Jail list: sshd
```

The SSH jail is active.

Detailed jail status:

```bash
sudo fail2ban-client status sshd
```

The SSH jail monitors authentication failures associated with the SSH service.

### 8.3 SSH Protection Parameters

The effective SSH jail parameters were inspected using:

```bash
sudo fail2ban-client get sshd maxretry
sudo fail2ban-client get sshd bantime
sudo fail2ban-client get sshd findtime
```

**Observed Configuration:**

| Parameter | Value | Description |
|---|---|---|
| `maxretry` | 5 | Maximum failed attempts within the monitoring window |
| `bantime` | 600 seconds | Ban duration (10 minutes) |
| `findtime` | 600 seconds | Monitoring window (10 minutes) |

Fail2ban monitors SSH authentication failures and temporarily bans IP addresses that exceed the configured retry limit.

These settings provide an additional layer of protection against repeated SSH authentication attempts.

**Note:** An active SSH jail confirms that Fail2ban is monitoring the service. It does not necessarily mean that any IP addresses have been banned.

## 9. Final Security Verification

### 9.1 UFW Firewall

Firewall status was verified using:

```bash
sudo ufw status verbose
```

**Verified Configuration:**

```text
Status: active
Logging: on (low)
Default: deny (incoming), allow (outgoing), disabled (routed)

22/tcp (OpenSSH)           ALLOW IN    Anywhere
22/tcp (OpenSSH (v6))      ALLOW IN    Anywhere (v6)
```

The firewall is active and blocks incoming traffic by default while allowing SSH access.

The firewall configuration is consistent with the SSH access established during the Network & Remote Access phase.

### Understanding the Firewall Policy

- **Incoming:** Denies incoming connections by default unless explicitly allowed.
- **Outgoing:** Allows outgoing connections by default.
- **Routed:** Forwarded traffic is not allowed by default.

The SSH rule permits incoming TCP connections on port 22 through the guest firewall.

Actual remote accessibility also depends on the VirtualBox NAT port-forwarding configuration and the host firewall.

### 9.2 Listening Network Ports

Listening network sockets were inspected using:

```bash
sudo ss -tulpn
```

**Observed Listening Ports:**

| Port | Protocol | Service | Purpose |
|---|---|---|---|
| 22 | TCP | SSH | Remote administration |
| 53 | TCP/UDP | systemd-resolved | Local DNS resolution |
| 68 | UDP | systemd-networkd | DHCP |
| 323 | UDP | chronyd | Chrony command and monitoring interface |

**Important:** UDP 323 is used by Chrony for its command and monitoring interface. It is not the standard NTP service port. NTP normally uses UDP 123.

No unexpected listening services were identified during the inspection.

Listening addresses must also be considered when determining whether a service is accessible from external interfaces.

A service bound to `127.0.0.1`, for example, is not directly reachable through the server's external network interfaces.

### 9.3 SSH Configuration Verification

The effective SSH configuration was inspected using:

```bash
sudo sshd -T | grep -E '^(permitrootlogin|passwordauthentication|pubkeyauthentication|kbdinteractiveauthentication)'
```

**Verified Configuration:**

```text
permitrootlogin no
pubkeyauthentication yes
passwordauthentication no
kbdinteractiveauthentication no
```

The effective SSH security settings confirm that:

- Root SSH login is disabled.
- Public-key authentication is enabled.
- Password authentication is disabled.
- Keyboard-interactive authentication is disabled.

These settings preserve the key-based remote administration model established during the initial network configuration.

## 10. Final Security Status

| Security Component | Status |
|---|---|
| System packages updated | Completed |
| Automatic security update configuration | Enabled |
| Automatic upgrade simulation | Interrupted by power-state detection |
| AppArmor | Active |
| Unnecessary services | ModemManager disabled |
| Fail2ban | Active |
| SSH jail | Active |
| UFW firewall | Active |
| Default incoming policy | Deny |
| SSH key authentication | Enabled |
| SSH password authentication | Disabled |
| Root SSH login | Disabled |
| Listening ports | Inspected |

## 11. Result

The Ubuntu server has undergone its initial security hardening process.

The firewall is active, SSH access is restricted to public-key authentication, AppArmor enforces security policies for protected processes, and Fail2ban provides additional protection against repeated SSH authentication failures.

Unnecessary services have been reviewed, and ModemManager has been disabled.

Automatic security update settings are enabled, although their dry-run simulation was interrupted by VirtualBox power-state detection. Successful unattended upgrade execution was therefore not confirmed during this phase.

These measures establish a stronger security baseline before deploying Docker infrastructure and application services.

Security hardening remains an ongoing process and should continue throughout the lifecycle of the server.