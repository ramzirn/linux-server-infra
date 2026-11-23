# 01 — Network & Remote Access

## 1. Objective

The objective of this phase is to establish network connectivity and secure remote administration of an Ubuntu Server virtual machine from a Linux Mint host machine.

The server runs inside Oracle VirtualBox using NAT networking. Remote administration is provided through SSH (Secure Shell) with Ed25519 public-key authentication.

This phase introduces essential concepts such as virtual machine networking, port forwarding, SSH authentication, firewall configuration, and basic network connectivity testing.

## 2. Prerequisites

Before starting this phase, the following requirements must be met:

- A Linux Mint host machine or another compatible Linux distribution.
- Oracle VirtualBox installed on the host.
- An Ubuntu Server virtual machine installed and accessible through its VirtualBox console.
- An active Internet connection.
- Basic familiarity with Linux terminal commands.

The initial SSH configuration must be performed from the virtual machine console or another available administrative access method.

## 3. Environment

The following environment was used for this implementation:

| Component | Configuration |
|---|---|
| Host machine | Linux Mint |
| Virtualization | Oracle VirtualBox |
| Guest operating system | Ubuntu Server 26.04.1 LTS |
| Hostname | `linux-server` |
| Server user | `ramzile` |
| Network mode | NAT |
| Guest IP address | `10.0.2.15/24` |
| Default gateway | `10.0.2.2` |
| SSH port (guest) | `22` |

## 4. Network Architecture

The Ubuntu server runs inside a VirtualBox virtual machine configured with NAT (Network Address Translation).

NAT allows the guest virtual machine to access external networks through the host machine without requiring a separate network interface configuration accessible directly from the local network.

To establish SSH access from the host, a VirtualBox port forwarding rule was configured.

### Port forwarding configuration

| Parameter | Value |
|---|---|
| Name | SSH |
| Protocol | TCP |
| Host IP | `127.0.0.1` |
| Host port | `2222` |
| Guest IP | `10.0.2.15` |
| Guest port | `22` |

### Connection flow

```text
Linux Mint Host
127.0.0.1:2222
       |
       v
VirtualBox NAT
Port Forwarding
       |
       v
Ubuntu Server
10.0.2.15:22
       |
       v
SSH Service
```

The host-side binding is restricted to `127.0.0.1`, which means the forwarded SSH port is accessible through the host's loopback interface rather than being exposed directly on the host's network interfaces.

**Important distinction:**

- `127.0.0.1:2222` is the address and port used by the Linux Mint host.
- `10.0.2.15:22` is the address and port of the SSH service inside Ubuntu.
- VirtualBox forwards incoming connections from the host-side port to the guest SSH service.

## 5. SSH Service Configuration

SSH (Secure Shell) provides encrypted remote access to Linux systems. It allows administrators to execute commands and manage a server remotely.

The SSH server was installed and configured on Ubuntu.

The service status was checked:

```bash
sudo systemctl status ssh
```

The listening port was inspected:

```bash
sudo ss -tlnp | grep ':22'
```

The service was confirmed running and listening on port 22.

The server was listening on both IPv4 and IPv6 interfaces.

**Note:** The SSH service listens on port 22 inside the Ubuntu guest. The host accesses it through the VirtualBox forwarded port 2222.

## 6. SSH Key Authentication

SSH supports several authentication methods. This project uses public-key authentication with the Ed25519 algorithm.

Ed25519 is a modern public-key cryptographic algorithm supported by OpenSSH.

### 6.1 Generate an SSH key pair

An Ed25519 SSH key pair was generated on the Linux Mint host:

```bash
ssh-keygen -t ed25519 -C "linux-server-access" -f ~/.ssh/linux_server_ed25519
```

The command generates two files:

| File | Purpose |
|---|---|
| `~/.ssh/linux_server_ed25519` | Private key |
| `~/.ssh/linux_server_ed25519.pub` | Public key |

The private key must remain on the host machine. It must never be shared or committed to a Git repository.

### 6.2 Install the public key on the server

The public key must be added to the Ubuntu user's authorized keys file:

`/home/ramzile/.ssh/authorized_keys`

For an initial setup where password-based SSH authentication is still available, the public key can be transferred using:

```bash
ssh-copy-id -i ~/.ssh/linux_server_ed25519.pub \
-p 2222 ramzile@127.0.0.1
```

This command installs the public key in the remote user's `authorized_keys` file.

The equivalent manual method is to append the public key to the file while connected through the virtual machine console.

### 6.3 Configure SSH directory permissions

The SSH directory and authorized keys file must have restrictive permissions.

| Path | Permission | Purpose |
|---|---|---|
| `/home/ramzile/.ssh` | `700` | Only the owner can access the directory |
| `/home/ramzile/.ssh/authorized_keys` | `600` | Only the owner can read and modify the file |

The permissions can be configured with:

```bash
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
```

The directory and file should also belong to the `ramzile` user.

**Security note:** OpenSSH may reject authentication if the user's home directory or SSH files have inappropriate ownership or permissions.

## 7. SSH Client Configuration

An SSH client configuration was created on the Linux Mint host to simplify remote connections.

**File:** `~/.ssh/config`

```sshconfig
Host linux-server
    HostName 127.0.0.1
    User ramzile
    Port 2222
    IdentityFile ~/.ssh/linux_server_ed25519
    IdentitiesOnly yes
```

### Configuration explanation

| Parameter | Purpose |
|---|---|
| `Host` | Defines a shortcut name for the connection |
| `HostName` | Specifies the destination address |
| `User` | Specifies the remote Linux user |
| `Port` | Specifies the host-side forwarded SSH port |
| `IdentityFile` | Specifies the private key used for authentication |
| `IdentitiesOnly yes` | Restricts authentication to the explicitly configured identity |

The SSH client configuration file permissions were restricted:

```bash
chmod 600 ~/.ssh/config
```

Remote access can now be established using the configured host alias:

```bash
ssh linux-server
```

This avoids specifying the IP address, port, username, and private key each time.

## 8. SSH Security Hardening

After confirming that SSH access worked with public-key authentication, the SSH server configuration was hardened.

### 8.1 Back up the original configuration

A backup of the original SSH server configuration was created:

```bash
sudo cp /etc/ssh/sshd_config /etc/ssh/sshd_config.backup
```

This provides a recovery point before modifying the SSH configuration.

### 8.2 Configure SSH security directives

The following security settings were applied in `/etc/ssh/sshd_config`:

```sshconfig
PermitRootLogin no
PubkeyAuthentication yes
PasswordAuthentication no
KbdInteractiveAuthentication no
```

### Directive explanations

| Directive | Configuration | Purpose |
|---|---|---|
| `PermitRootLogin` | `no` | Prevents direct SSH login as root |
| `PubkeyAuthentication` | `yes` | Enables public-key authentication |
| `PasswordAuthentication` | `no` | Disables SSH password authentication |
| `KbdInteractiveAuthentication` | `no` | Disables keyboard-interactive authentication |

Disabling direct root login encourages administrators to connect using a regular user account and use `sudo` when administrative privileges are required.

Disabling password-based authentication reduces exposure to password-guessing attacks. However, it also means that a working alternative authentication method must be confirmed before applying the change.

### 8.3 Check additional SSH configuration files

An additional SSH configuration file was identified:

`/etc/ssh/sshd_config.d/50-cloud-init.conf`

It contained a password authentication setting that affected the effective SSH configuration.

The setting was changed to:

```sshconfig
PasswordAuthentication no
```

OpenSSH can load additional configuration files through the `Include` directive. The effective configuration depends on the directives and their order, so checking only `/etc/ssh/sshd_config` is not sufficient.

### 8.4 Verify the effective SSH configuration

The effective SSH settings were inspected using:

```bash
sudo sshd -T | grep -E '^(permitrootlogin|passwordauthentication|pubkeyauthentication|kbdinteractiveauthentication)'
```

Expected and verified output:

```text
permitrootlogin no
passwordauthentication no
pubkeyauthentication yes
kbdinteractiveauthentication no
```

### 8.5 Validate and reload SSH

The SSH configuration syntax was validated:

```bash
sudo sshd -t
```

No output indicates that the configuration syntax check completed successfully.

The SSH service was then reloaded:

```bash
sudo systemctl reload ssh
```

A new SSH connection was successfully established from a second terminal after the reload, confirming that key-based authentication remained operational.

**Important:** Keep the existing SSH session open while testing a new connection. If the new connection fails, the existing session or VirtualBox console can be used to restore access.

## 9. Firewall Configuration

UFW (Uncomplicated Firewall) was enabled on the Ubuntu server to control incoming and outgoing network traffic.

UFW provides a simplified interface for managing Linux firewall rules.

The firewall status and policies were checked:

```bash
sudo ufw status verbose
```

### Active firewall configuration

| Setting | Configuration |
|---|---|
| Firewall status | Active |
| Default incoming policy | Deny |
| Default outgoing policy | Allow |
| Default routed policy | Disabled |
| SSH IPv4 | Allowed |
| SSH IPv6 | Allowed |

Active rules were verified:

```bash
sudo ufw status numbered
```

Verified output:

```text
[1] OpenSSH       ALLOW IN    Anywhere
[2] OpenSSH (v6)  ALLOW IN    Anywhere (v6)
```

Only port 22 needs to be allowed by UFW on the Ubuntu guest. Port 2222 is the host-side VirtualBox forwarding port and is not a guest firewall port.

**Security note:** The firewall rules permit SSH traffic on the guest. The VirtualBox NAT configuration determines how the host reaches that service.

## 10. Network Connectivity Tests

Network connectivity was tested directly from the Ubuntu server.

These tests verify different parts of the network configuration.

### 10.1 Default gateway connectivity

```bash
ping -c 4 10.0.2.2
```

**Result:** 4 packets received, 0% packet loss.

This confirms connectivity between the virtual machine and its VirtualBox NAT gateway.

### 10.2 External IP connectivity

```bash
ping -c 4 8.8.8.8
```

**Result:** 4 packets received, 0% packet loss.

This confirms external IP connectivity without relying on DNS resolution.

### 10.3 DNS resolution and connectivity

```bash
ping -c 4 google.com
```

**Result:** DNS resolution successful, 4 packets received, 0% packet loss.

This confirms that the server can resolve a domain name and reach the corresponding destination.

Together, these tests confirmed connectivity to the VirtualBox gateway, external IP connectivity, and DNS resolution.

**Note:** Some external services block ICMP traffic. A failed ping does not always mean that the network connection is unavailable.

## 11. Troubleshooting

The following checks can help diagnose common issues during this phase.

| Problem | Possible cause | Suggested check |
|---|---|---|
| SSH connection refused | SSH service stopped or incorrect forwarding rule | `sudo systemctl status ssh` |
| SSH connection timed out | Network or firewall issue | Check VirtualBox NAT forwarding and UFW |
| Permission denied (publickey) | Incorrect key, ownership, or permissions | Inspect the SSH key and `authorized_keys` permissions |
| Password authentication still available | Conflicting SSH configuration directives | Inspect configuration files and run `sudo sshd -T` |
| Domain name cannot be resolved | DNS configuration issue | Test an external IP and inspect DNS settings |
| SSH connection fails after configuration changes | Incorrect SSH settings | Use the existing session or VirtualBox console to recover access |

For SSH connection troubleshooting, the verbose client mode can provide additional diagnostic information:

```bash
ssh -v linux-server
```

## 12. Final Validation

The following components and configurations were successfully verified:

- [x] Ubuntu server network connectivity.
- [x] VirtualBox NAT networking.
- [x] SSH service running and listening on port 22.
- [x] VirtualBox SSH port forwarding from host to guest.
- [x] Ed25519 public-key authentication.
- [x] SSH client configuration using `ssh linux-server`.
- [x] Root SSH login disabled.
- [x] Password-based SSH authentication disabled.
- [x] Public-key authentication enabled.
- [x] UFW active with restrictive incoming traffic rules.
- [x] Successful gateway, external IP, and DNS connectivity tests.
- [x] Successful SSH reconnection after applying security settings.

## 13. Result

The Ubuntu server is accessible remotely from the Linux Mint host through SSH using Ed25519 public-key authentication.

Root login and password-based SSH authentication are disabled, while UFW restricts incoming traffic.

The network and remote administration layer is configured and validated, providing the foundation for the subsequent infrastructure configuration phases.

This phase introduces the fundamental concepts required to administer a Linux server remotely and securely.