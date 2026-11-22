# 01 — Network & Remote Access

## 1. Objective

The objective of this phase is to configure reliable network connectivity and secure remote administration of the Ubuntu server from the Linux Mint host machine.

The server is accessed remotely through SSH using public-key authentication.

## 2. Environment

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

## 3. Network Architecture

The Ubuntu server runs inside a VirtualBox virtual machine configured with NAT networking.

SSH access is provided through VirtualBox port forwarding.

**Port forwarding configuration:**

| Parameter | Value |
|---|---|
| Name | SSH |
| Protocol | TCP |
| Host IP | `127.0.0.1` |
| Host port | `2222` |
| Guest IP | `10.0.2.15` |
| Guest port | `22` |

Connection flow:

`Linux Mint (127.0.0.1:2222) → VirtualBox NAT → Ubuntu (10.0.2.15:22)`

The host-side binding is restricted to localhost. The SSH forwarding port is therefore not exposed directly to the local network.

## 4. SSH Service Configuration

The SSH server was installed and configured on Ubuntu.

The service status was checked using:

```bash
sudo systemctl status ssh
```

The SSH service is running and listening on port 22.

The listening sockets were verified with:

```bash
sudo ss -tlnp | grep ':22'
```

The server listens on IPv4 and IPv6 interfaces.

## 5. SSH Key Authentication

An Ed25519 SSH key pair was generated on the Linux Mint host:

```bash
ssh-keygen -t ed25519 -C "linux-server-access" -f ~/.ssh/linux_server_ed25519
```

The generated files are:

- `~/.ssh/linux_server_ed25519` — private key.
- `~/.ssh/linux_server_ed25519.pub` — public key.

The public key was added to the Ubuntu user's authorized keys file:

```text
/home/ramzile/.ssh/authorized_keys
```

The server-side SSH permissions were configured as follows:

| Path | Permission |
|---|---|
| `~/.ssh` | `700` |
| `~/.ssh/authorized_keys` | `600` |

The private key remains on the host machine and must never be shared or copied into the repository.

## 6. SSH Client Configuration

To simplify remote connections, an SSH client configuration was created on the Linux Mint host.

File: `~/.ssh/config`

```sshconfig
Host linux-server
    HostName 127.0.0.1
    User ramzile
    Port 2222
    IdentityFile ~/.ssh/linux_server_ed25519
    IdentitiesOnly yes
```

The configuration file permissions were restricted:

```bash
chmod 600 ~/.ssh/config
```

Remote access can now be established using:

```bash
ssh linux-server
```

## 7. SSH Security Hardening

A backup of the original SSH configuration was created:

```bash
sudo cp /etc/ssh/sshd_config /etc/ssh/sshd_config.backup
```

The following security settings were applied in `/etc/ssh/sshd_config`:

```sshconfig
PermitRootLogin no
PubkeyAuthentication yes
PasswordAuthentication no
KbdInteractiveAuthentication no
```

An additional configuration file was discovered:

`/etc/ssh/sshd_config.d/50-cloud-init.conf`

It contained an overriding password authentication setting. This was changed to:

```sshconfig
PasswordAuthentication no
```

### Effective SSH configuration

The effective settings were verified using:

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

The SSH configuration syntax was checked:

```bash
sudo sshd -t
```

The SSH service was then reloaded:

```bash
sudo systemctl reload ssh
```

A new SSH connection was successfully established from a second terminal after the reload, confirming that key-based authentication remained operational.

## 8. Firewall Configuration

UFW (Uncomplicated Firewall) was enabled on the Ubuntu server.

Firewall status was checked with:

```bash
sudo ufw status verbose
```

The active firewall policy is:

| Setting | Configuration |
|---|---|
| Firewall status | Active |
| Default incoming policy | Deny |
| Default outgoing policy | Allow |
| Default routed policy | Disabled |
| SSH IPv4 | Allowed |
| SSH IPv6 | Allowed |

The SSH rules were verified with:

```bash
sudo ufw status numbered
```

The active rules are:

```text
[1] OpenSSH       ALLOW IN    Anywhere
[2] OpenSSH (v6)  ALLOW IN    Anywhere (v6)
```

Only port 22 needs to be allowed by UFW on the Ubuntu guest. Port 2222 is the host-side VirtualBox forwarding port.

## 9. Network Connectivity Tests

Network connectivity was tested directly from the Ubuntu server.

### 9.1 Default gateway

```bash
ping -c 4 10.0.2.2
```

Result: 4 packets received, 0% packet loss.

### 9.2 External IP connectivity

```bash
ping -c 4 8.8.8.8
```

Result: 4 packets received, 0% packet loss.

### 9.3 DNS resolution and connectivity

```bash
ping -c 4 google.com
```

Result: DNS resolution successful, 4 packets received, 0% packet loss.

These tests confirm communication with the VirtualBox gateway, external IP connectivity, and DNS resolution.

## 10. Final Validation

The following items were successfully verified:

- Ubuntu server network connectivity.
- VirtualBox NAT networking.
- SSH service running on port 22.
- SSH port forwarding from the host to the guest.
- Ed25519 public-key authentication.
- Simplified SSH connection using `ssh linux-server`.
- Root SSH login disabled.
- Password-based SSH authentication disabled.
- Public-key authentication enabled.
- UFW active with restrictive incoming traffic rules.
- Successful network and DNS connectivity tests.
- Successful SSH reconnection after applying the security configuration.

## 11. Result

The Ubuntu server is now reachable remotely from the Linux Mint host through SSH.

Remote access uses Ed25519 public-key authentication, password-based SSH authentication is disabled, and UFW restricts incoming connections.

The server is ready for the next infrastructure configuration phases.