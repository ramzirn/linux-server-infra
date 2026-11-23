# 04 — User and Access Management

## 1. Objective

The objective of this phase is to establish a structured Linux access management model covering user accounts, groups, administrative privileges, and filesystem permissions.

The configuration follows the principle of least privilege and enables controlled collaboration between users.

Linux user and group management is essential for securing system resources and ensuring that users only access the files and administrative operations required for their responsibilities.

## 2. Prerequisites

Before starting this phase, the following requirements should be met:

- An operational Ubuntu Server installation.
- A user account with administrative privileges.
- SSH access to the server, if remote administration is being used.
- Basic familiarity with Linux terminal commands.

## 3. Objectives

The main objectives of this phase are:

- Inspect existing Linux user accounts.
- Understand system and standard user accounts.
- Create a dedicated administrative user.
- Configure and validate sudo privileges.
- Create and manage Linux groups.
- Configure directory ownership and permissions.
- Understand the SGID permission.
- Test access restrictions on a shared directory.

## 4. User Account Inspection

Existing user accounts were inspected using:

```bash
getent passwd
```

This command retrieves account information from the system's configured user databases, which may include local files and other configured identity sources.

Standard user accounts were identified based on their UID range:

```bash
awk -F: '$3 >= 1000 && $3 < 60000 {print $1, $3, $6, $7}' /etc/passwd
```

This command displays selected account information:

- Username.
- User ID (UID).
- Home directory.
- Login shell.

The current session identity and group membership were inspected using:

```bash
whoami
id
```

### Key Concepts

Linux distinguishes between different types of accounts:

- **System Accounts:** Generally used by system services and applications. They commonly have lower UIDs and are not intended for interactive human login.
- **Regular User Accounts:** Generally used for interactive access and administration. On Ubuntu, regular users commonly start at UID 1000.
- **UID:** A numerical identifier assigned to each user account.
- **GID:** A numerical identifier associated with a group.

Some service accounts use `/usr/sbin/nologin` as their login shell to prevent interactive login.

**Note:** The UID range is a convention defined by the system configuration, not an absolute rule. The `awk` command above inspects local accounts in `/etc/passwd`, while `getent passwd` can retrieve accounts from other configured identity sources.

## 5. Administrative User Creation

A dedicated administrative account named `sysadmin` was created:

```bash
sudo adduser sysadmin
```

The command creates the account and guides the administrator through password configuration and optional user information.

The user was added to the `sudo` group:

```bash
sudo usermod -aG sudo sysadmin
```

### Understanding the Command

- `usermod`: Modifies an existing user account.
- `-a`: Appends the specified group to the user's existing supplementary groups.
- `-G`: Specifies supplementary group membership.

The `-a` option is important because using `-G` without `-a` can replace the user's existing supplementary group list.

Group membership was verified using:

```bash
groups sysadmin
```

The account was tested by switching to it:

```bash
su - sysadmin
```

The `-` option starts a login shell and loads the target user's environment.

Administrative privileges were validated using:

```bash
sudo whoami
sudo -l
```

The `sudo whoami` command returned `root`, confirming that the account could execute commands with administrative privileges.

The `sudo -l` command displays the commands the user is permitted to execute through sudo.

**Note:** New group membership is applied to new login sessions. Switching to a login shell with `su - sysadmin` provides a fresh session for testing the configured account.

## 6. Linux Group Management

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

### Key Concepts

Linux groups provide a centralized way to assign permissions to multiple users without configuring access individually for every account.

A user can belong to:

- A primary group, associated with the user's account.
- One or more supplementary groups, providing additional access permissions.

In this configuration, `developers` is a supplementary group used to control access to shared project resources.

## 7. Shared Directory Creation

A shared directory was created under `/opt`:

```bash
sudo mkdir -p /opt/projects
```

The directory was assigned to the administrative user and the `developers` group:

```bash
sudo chown sysadmin:developers /opt/projects
```

Ownership was verified using:

```bash
ls -ld /opt/projects
```

The `/opt/projects` directory will be used to organize future projects and application-related resources.

### Understanding Directory Ownership

Linux filesystem objects have:

- An owner user.
- An owner group.
- A set of permissions controlling access.

The `chown` command changes the ownership of a file or directory.

In this configuration:

- Owner: `sysadmin`
- Group: `developers`

## 8. Filesystem Permissions

Directory permissions were configured using:

```bash
sudo chmod 2770 /opt/projects
```

The numeric permission value `2770` has the following meaning:

| Value | Description |
|---|---|
| 2 | SGID bit enabled |
| 7 | Owner: read, write, and execute |
| 7 | Group: read, write, and execute |
| 0 | Others: no permissions |

The resulting permission representation is:

```text
drwxrws---
```

### Understanding Linux Permissions

Linux permissions are divided into three categories:

- **Owner:** Permissions assigned to the user who owns the directory.
- **Group:** Permissions assigned to members of the directory's group.
- **Others:** Permissions assigned to all other users.

The three basic permissions are:

| Permission | Meaning for a directory |
|---|---|
| Read (`r`) | Allows listing directory contents |
| Write (`w`) | Allows creating, deleting, and renaming entries, subject to other filesystem restrictions |
| Execute (`x`) | Allows accessing and traversing the directory |

This configuration provides full access to the owner and members of the `developers` group while denying access to other users.

## 9. SGID Configuration

The SGID bit was enabled on `/opt/projects` through the `chmod 2770` command.

For directories, SGID ensures that newly created files and subdirectories inherit the parent directory's group ownership, unless explicitly changed.

This is useful in collaborative environments where multiple users need to work with shared resources.

### Important Permission Consideration

SGID controls group inheritance, but it does not automatically guarantee group write access to newly created files.

The resulting permissions also depend on:

- The permissions requested when the file or directory is created.
- The user's `umask`.
- Any additional filesystem permission mechanisms.

For example, a restrictive `umask` may prevent newly created files from being writable by other members of the group.

## 10. Access Validation

The directory configuration was inspected using:

```bash
ls -ld /opt/projects
```

The expected ownership and permission structure was:

```text
drwxrws--- sysadmin developers /opt/projects
```

Access was tested using different user accounts.

The validation covered the following conditions:

- The administrative user could access the shared directory.
- Users outside the `developers` group were denied access.

### Reproducing the Access Tests

As `sysadmin`:

```bash
cd /opt/projects
```

To test access using an account outside the `developers` group, switch to an appropriate existing user:

```bash
su - <username>
cd /opt/projects
```

The second command should return a permission denied error for a user who is neither the directory owner nor a member of its group.

Replace `<username>` with an actual existing account that is not a member of `developers`.

These tests validate directory ownership, group permissions, and access restrictions.

## 11. Result

The user and access management configuration is complete.

The server now has:

- A dedicated administrative account named `sysadmin`.
- Validated sudo privileges.
- A dedicated `developers` group.
- A shared project directory at `/opt/projects`.
- Controlled ownership and filesystem permissions.
- SGID enabled for group inheritance.
- Validated access restrictions.

This establishes a basic Linux access management structure based on dedicated accounts, group-based permissions, and controlled administrative privileges.

The configuration supports collaborative administration while limiting access to shared server resources.