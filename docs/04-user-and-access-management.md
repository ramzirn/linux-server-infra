# 03 - User and Access Management

## 1. Overview

This phase focuses on Linux user and group management, administrative privileges, and filesystem permissions.

The objective is to establish a structured access management model that follows the principle of least privilege and allows controlled collaboration between users.

## 2. Objectives

The main objectives of this phase were:

- Inspect existing Linux user accounts.
- Understand system and standard user accounts.
- Create a dedicated administrative user.
- Configure and validate sudo privileges.
- Create and manage Linux groups.
- Configure directory ownership and permissions.
- Understand the SGID permission.
- Test access restrictions on a shared directory.

## 3. User Account Inspection

Existing user accounts were inspected using:

```bash
getent passwd
```

This command retrieves account information from the system's configured user databases.

To display standard user accounts based on their UID range:

```bash
awk -F: '$3 >= 1000 && $3 < 60000 {print $1, $3, $6, $7}' /etc/passwd
```

The current session identity and group membership were inspected using:

```bash
whoami
id
```

Linux distinguishes between system accounts, which are generally used by services, and regular user accounts used for interactive access.

Some service accounts use `/usr/sbin/nologin` to prevent interactive login.

## 4. Administrative User Creation

A dedicated administrative account named `sysadmin` was created:

```bash
sudo adduser sysadmin
```

The user was added to the `sudo` group:

```bash
sudo usermod -aG sudo sysadmin
```

Group membership was verified using:

```bash
groups sysadmin
```

The account was tested by switching to it:

```bash
su - sysadmin
```

Administrative privileges were validated using:

```bash
sudo whoami
sudo -l
```

The `sudo whoami` command returned `root`, confirming that the account could execute commands with administrative privileges.

## 5. Linux Group Management

A dedicated group named `developers` was created to manage access to shared project resources:

```bash
sudo groupadd developers
```

The administrative user was added to the group:

```bash
sudo usermod -aG developers sysadmin
```

Membership was verified using:

```bash
groups sysadmin
getent group developers
```

Groups provide a centralized way to assign permissions to multiple users without configuring access individually for every account.

## 6. Shared Directory Creation

A shared directory was created under `/opt`:

```bash
sudo mkdir -p /opt/projects
```

The directory was assigned to the administrative user and the developers group:

```bash
sudo chown sysadmin:developers /opt/projects
```

The ownership was verified using:

```bash
ls -ld /opt/projects
```

The `/opt/projects` directory will be used to organize future projects and application-related resources.

## 7. Filesystem Permissions

Directory permissions were configured using:

```bash
sudo chmod 2770 /opt/projects
```

The numeric permission value `2770` has the following meaning:

| Value | Description |
|---|---|
| 2 | SGID bit enabled |
| 7 | Owner: read, write and execute |
| 7 | Group: read, write and execute |
| 0 | Others: no permissions |

The resulting permission representation is:

```text
drwxrws---
```

This configuration provides full access to the owner and members of the `developers` group while denying access to other users.

### 7.1 SGID Configuration

The SGID bit was enabled on `/opt/projects`.

For directories, SGID ensures that newly created files and subdirectories inherit the parent directory's group ownership, unless explicitly changed.

This is useful for collaborative environments where several users need to work with shared resources.

## 8. Access Validation

The directory configuration was inspected using:

```bash
ls -ld /opt/projects
```

The expected ownership and permission structure was:

```text
drwxrws--- sysadmin developers /opt/projects
```

Access was also tested using different user accounts.

The administrative user could access the shared directory, while users outside the `developers` group were denied access.

This validated the directory's access restrictions.

## 9. Result

The user and access management configuration is complete.

The server now has:

- A dedicated administrative account named `sysadmin`.
- Validated sudo privileges.
- A dedicated `developers` group.
- A shared project directory at `/opt/projects`.
- Controlled ownership and filesystem permissions.
- SGID enabled for group inheritance.
- Validated access restrictions.

## 10. Conclusion

This phase established a basic Linux access management structure.

The use of dedicated accounts, groups and filesystem permissions provides better control over server resources and supports collaborative administration.

The next phase will focus on network configuration, connectivity, SSH access and network service management.