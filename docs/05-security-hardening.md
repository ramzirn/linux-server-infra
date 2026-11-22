# 05 - Security Hardening

## 1. Overview

This phase focuses on strengthening the security of the Ubuntu server by applying security updates, enabling automatic security maintenance, verifying system security mechanisms, reducing unnecessary services, and protecting remote SSH access.

The objective is to reduce the server's attack surface and establish a secure foundation for future Docker infrastructure.

## 2. Security Audit

An initial security audit was performed to inspect the current system security status.

### Ubuntu Security Status

Command:

```bash
ubuntu-security-status
```

Results:
- Ubuntu 26.04.1 LTS.
- Main and Restricted packages receive security maintenance until 2031.
- Ubuntu Pro is not enabled.
- Extended Security Maintenance is available through Ubuntu Pro.

Ubuntu Pro is not required for the current laboratory environment.

## 3. System Updates & Security Patches

Package repositories were refreshed:

```bash
sudo apt update
```

Available package upgrades were inspected:

```bash
apt list --upgradable
```

System packages were upgraded:

```bash
sudo apt upgrade
```

A kernel update required a system reboot.

The server was restarted:

```bash
sudo reboot
```

Remote SSH access was successfully restored after reboot.

## 4. Automatic Security Updates

### 4.1 Package Verification

The `unattended-upgrades` package was already installed.

Verification:

```bash
dpkg -l unattended-upgrades
```

### 4.2 Automatic Updates Configuration

Configuration file:

```bash
/etc/apt/apt.conf.d/20auto-upgrades
```

Current configuration:

```text
APT::Periodic::Update-Package-Lists "1";
APT::Periodic::Unattended-Upgrade "1";
```

This enables:
- Daily package list updates.
- Automatic unattended upgrades.

### 4.3 Security Update Origins

Configuration file:

```bash
/etc/apt/apt.conf.d/50unattended-upgrades
```

Allowed origins include:

```text
"${distro_id}:${distro_codename}";
"${distro_id}:${distro_codename}-security";
"${distro_id}ESMApps:${distro_codename}-apps-security";
"${distro_id}ESM:${distro_codename}-infra-security";
```

Security updates are prioritized without explicitly enabling the proposed, backports, or regular updates repositories.

### 4.4 Automatic Upgrade Simulation

Command:

```bash
sudo unattended-upgrade --dry-run --debug
```

The simulation stopped with the following message:

```text
System is on battery power, stopping
upgrade result: False
```

Investigation showed:
- `BAT0` status: `Discharging`.
- `AC` online status: `0`.

This behavior is related to power detection in the VirtualBox laptop environment.

No configuration changes were made to bypass this restriction.

**Note:** Automatic upgrade settings are enabled, but the simulation could not be completed because of the detected power state.

Automatic reboot was not explicitly enabled.

## 5. AppArmor

AppArmor provides Mandatory Access Control (MAC), restricting the capabilities of applications according to security profiles.

### 5.1 Service Verification

Command:

```bash
sudo systemctl status apparmor
```

Result: The AppArmor service is active.

### 5.2 Profile Verification

Command:

```bash
sudo aa-status
```

Observed results:
- 186 profiles loaded.
- 110 profiles in enforce mode.
- 2 profiles in complain mode.
- 74 profiles in unconfined mode.
- 3 processes running with enforced profiles.

AppArmor is operational and enforcing security policies.

Docker-specific AppArmor profiles were not present because Docker has not yet been installed.

## 6. Services & Attack Surface Reduction

Enabled services and currently running services were inspected.

Commands:

```bash
systemctl list-unit-files --type=service --state=enabled
```

```bash
systemctl --type=service --state=running
```

The `ModemManager` service was identified as unnecessary for this server environment.

### 6.1 ModemManager

ModemManager manages mobile broadband devices and cellular modems.

Since the VM does not require mobile modem connectivity, the service was disabled and stopped.

Command:

```bash
sudo systemctl disable --now ModemManager.service
```

Other services, including `multipathd`, were inspected but left unchanged to avoid unnecessary modifications to system components.

## 7. SSH & Brute-force Protection

### 7.1 Fail2ban Installation

Fail2ban was not initially installed.

Installation:

```bash
sudo apt install fail2ban
```

Service verification:

```bash
sudo systemctl status fail2ban
```

Result: The service is active.

### 7.2 SSH Jail Verification

Fail2ban status:

```bash
sudo fail2ban-client status
```

Observed result:

```text
Status
|- Number of jail: 1
`- Jail list: sshd
```

The SSH jail is active.

Detailed status:

```bash
sudo fail2ban-client status sshd
```

### 7.3 SSH Protection Parameters

The effective configuration was inspected using:

```bash
sudo fail2ban-client get sshd maxretry
sudo fail2ban-client get sshd bantime
sudo fail2ban-client get sshd findtime
```

Observed configuration:

| Parameter | Value | Description |
|---|---|---|
| maxretry | 5 | Maximum failed attempts |
| bantime | 600 seconds | Ban duration (10 minutes) |
| findtime | 600 seconds | Monitoring window (10 minutes) |

Fail2ban monitors SSH authentication failures and temporarily bans IP addresses that exceed the configured retry limit.

## 8. Final Security Verification

### 8.1 UFW Firewall

Command:

```bash
sudo ufw status verbose
```

Verified configuration:

```text
Status: active
Logging: on (low)
Default: deny (incoming), allow (outgoing), disabled (routed)

22/tcp (OpenSSH)           ALLOW IN    Anywhere
22/tcp (OpenSSH (v6))      ALLOW IN    Anywhere (v6)
```

The firewall is active and blocks incoming traffic by default, while allowing SSH access.

### 8.2 Listening Network Ports

Command:

```bash
sudo ss -tulpn
```

Observed listening ports:

| Port | Protocol | Service | Purpose |
|---|---|---|---|
| 22 | TCP | SSH | Remote administration |
| 53 | TCP/UDP | systemd-resolved | Local DNS resolution |
| 68 | UDP | systemd-networkd | DHCP |
| 323 | UDP | chronyd | Time synchronization |

No unexpected listening services were identified.

### 8.3 SSH Configuration Verification

Command:

```bash
sudo sshd -T | grep -E 'permitrootlogin|passwordauthentication|pubkeyauthentication|kbdinteractiveauthentication'
```

Verified configuration:

```text
permitrootlogin no
pubkeyauthentication yes
passwordauthentication no
kbdinteractiveauthentication no
```

SSH security settings are correctly applied:
- Root login is disabled.
- Public key authentication is enabled.
- Password authentication is disabled.
- Keyboard-interactive authentication is disabled.

## 9. Final Security Status

The following security measures have been verified:

| Security Component | Status |
|---|---|
| System packages updated | Completed |
| Automatic security updates | Enabled |
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

## 10. Conclusion

The Ubuntu server has undergone its initial security hardening process.

The firewall is active, SSH access is restricted to public key authentication, AppArmor is enforcing security policies, and Fail2ban provides additional protection against repeated SSH authentication failures.

Unnecessary services have been reviewed, and ModemManager has been disabled.

Automatic security updates are enabled, although their dry-run simulation was interrupted by the VirtualBox power-state detection.

The server now has a stronger security baseline and is ready for the next infrastructure configuration phase.