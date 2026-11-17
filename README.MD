# Linux Server Infrastructure

## Project Overview

**Linux Server Infrastructure** is a hands-on infrastructure project focused on building, configuring, securing, and maintaining a Linux server in a virtualized environment.

The objective is to transform a fresh Linux installation into a stable, secure, maintainable, and production-ready server environment capable of hosting containerized applications and infrastructure services.

The server runs as a virtual machine using Oracle VirtualBox.

This project covers the complete server preparation lifecycle, from initial system configuration and Linux administration to network management, security hardening, containerization, monitoring, and automation.

All implementation steps, configurations, technical decisions, and procedures are documented in this repository.

## Project Objectives

The main objectives are to:

- Build and configure a Linux server from a fresh installation.
- Establish a reliable and maintainable server environment.
- Implement secure remote administration.
- Understand and manage Linux users, permissions, processes, and services.
- Configure network connectivity and firewall rules.
- Apply essential server security hardening practices.
- Prepare the server to host containerized applications using Docker.
- Manage application deployments with Docker Compose.
- Configure a reverse proxy to expose hosted applications.
- Implement infrastructure monitoring and observability.
- Automate system administration and maintenance tasks.
- Establish backup and troubleshooting procedures.
- Maintain clear and reproducible technical documentation.

## Project Scope & Implementation Roadmap

The project is divided into the following implementation phases.

### Phase 1 — System Inspection & Initial Assessment

Establish a baseline of the newly installed Linux server.

- Identify the operating system and kernel version.
- Inspect CPU, RAM, and disk resources.
- Identify network interfaces and IP configuration.
- Inspect running processes and system services.
- Verify internet connectivity.
- Identify the current system configuration and limitations.

**Expected result:** A documented understanding of the initial server environment.

### Phase 2 — Initial System Configuration

Prepare the operating system for administration and hosting workloads.

- Configure the server hostname.
- Configure timezone and system time.
- Update system packages.
- Install essential administration utilities.
- Review package management and repositories.
- Configure automatic security updates where appropriate.

**Expected result:** A clean, updated, and properly configured Linux system.

### Phase 3 — User & Access Management

Establish a controlled server administration environment.

- Create a dedicated administrative user.
- Configure sudo privileges.
- Understand Linux users and groups.
- Configure SSH remote access.
- Set up SSH key-based authentication.
- Review file ownership and permissions.
- Restrict unnecessary administrative access.

**Expected result:** Secure and controlled server administration.

### Phase 4 — Network Configuration

Configure and validate server networking.

- Inspect and understand network interfaces.
- Configure IP addressing.
- Understand DNS resolution and default gateways.
- Verify connectivity between the host and the VM.
- Test network connectivity and name resolution.
- Identify required application ports.
- Configure firewall rules using UFW.

**Expected result:** A reachable server with properly configured network access.

### Phase 5 — Security Hardening

Apply essential Linux server security practices.

- Harden SSH configuration.
- Disable unnecessary authentication methods.
- Configure UFW firewall policies.
- Configure Fail2ban where appropriate.
- Enable security updates.
- Review exposed ports and running services.
- Apply the principle of least privilege.
- Review basic security and system logs.

**Expected result:** A hardened server with a reduced attack surface.

### Phase 6 — System Services & Storage Management

Prepare the server for application workloads and long-term maintenance.

- Understand systemd and service management.
- Inspect and manage system services.
- Understand filesystem structure and disk usage.
- Organize application and configuration directories.
- Manage system logs using journalctl.
- Understand process and resource management.
- Establish basic maintenance procedures.

**Expected result:** An organized and maintainable server environment.

### Phase 7 — Docker Environment Preparation

Prepare the server to host containerized workloads.

- Install Docker Engine.
- Install and configure Docker Compose.
- Understand Docker images and containers.
- Configure Docker networks.
- Manage persistent volumes.
- Understand container lifecycle and logs.
- Apply basic Docker security practices.
- Validate Docker functionality.

**Expected result:** A functional Docker environment ready for application deployment.

### Phase 8 — Application Hosting & Reverse Proxy

Deploy and expose applications through the server.

- Deploy multiple containerized applications.
- Define services using Docker Compose.
- Configure application-specific networks and volumes.
- Install and configure Nginx or Traefik.
- Configure reverse proxy routing.
- Manage application environment variables.
- Validate application accessibility.
- Understand internal and external service communication.

**Expected result:** A server capable of hosting and exposing multiple applications.

### Phase 9 — Monitoring & Observability

Implement visibility into server and application health.

- Install Prometheus.
- Configure Node Exporter.
- Install Grafana.
- Collect server resource metrics.
- Monitor CPU, RAM, disk, and network usage.
- Create monitoring dashboards.
- Explore basic alerting mechanisms.

**Expected result:** A monitored infrastructure with visibility into system health and resource consumption.

### Phase 10 — Automation, Backup & Maintenance

Improve infrastructure consistency and operational efficiency.

- Develop Bash administration scripts.
- Automate repetitive maintenance tasks.
- Introduce Ansible for server configuration management.
- Automate application deployment where appropriate.
- Define backup procedures.
- Test backup restoration.
- Document common troubleshooting procedures.

**Expected result:** A more reproducible, automated, and maintainable server infrastructure.

## Technology Stack

| Category | Technology |
|---|---|
| Virtualization | Oracle VirtualBox |
| Operating System | Ubuntu Server |
| Remote Administration | OpenSSH |
| Firewall | UFW |
| Intrusion Prevention | Fail2ban |
| Containerization | Docker Engine |
| Container Management | Docker Compose |
| Reverse Proxy | Nginx / Traefik |
| Monitoring | Prometheus |
| Visualization | Grafana |
| Automation | Bash / Ansible |
| Version Control | Git / GitHub |

## Target Infrastructure Architecture

```text
                  HOST MACHINE
                       |
                   VirtualBox
                       |
                 UBUNTU SERVER
                       |
          +------------+------------+
          |            |            |
         SSH          UFW         Storage
          |            |            |
          +------------+------------+
                       |
                  DOCKER ENGINE
                       |
                DOCKER NETWORKS
                       |
          +------------+------------+
          |            |            |
       App 1         App 2       Monitoring
          |            |            |
          +------------+------------+
                       |
                 REVERSE PROXY
                       |
                APPLICATION ACCESS
```

The architecture will evolve as each implementation phase is completed.

## Expected Deliverables

At the end of the project, the repository should contain:

- A fully configured Linux server.
- A documented server architecture.
- Secure remote administration.
- Network and firewall configuration.
- A hardened operating system.
- A functional Docker environment.
- Multiple deployed containerized applications.
- A configured reverse proxy.
- A monitoring and observability stack.
- Server administration and automation scripts.
- Ansible configuration and playbooks.
- Backup and maintenance procedures.
- Technical documentation and troubleshooting guides.
- A final technical project report in PDF format.

## Repository Structure

```text
linux-server-infra/
│
├── README.md
│
├── docs/
│   ├── architecture.md
│   ├── system-inspection.md
│   ├── system-configuration.md
│   ├── user-management.md
│   ├── network-configuration.md
│   ├── security-hardening.md
│   ├── storage-management.md
│   ├── docker-setup.md
│   ├── application-deployment.md
│   ├── monitoring.md
│   ├── automation.md
│   └── troubleshooting.md
│
├── scripts/
│   ├── system/
│   ├── security/
│   ├── maintenance/
│   └── backup/
│
├── docker/
│   ├── applications/
│   └── monitoring/
│
├── nginx/
│
├── ansible/
│   ├── inventory/
│   ├── playbooks/
│   └── roles/
│
└── screenshots/
```

## Project Philosophy

This project follows a practical, structured, and documentation-driven approach.

Every implementation phase should be understood, configured, tested, and documented before moving to the next one.

The goal is not simply to install infrastructure tools, but to understand how a Linux server operates, how its components interact, and how to maintain a reliable environment following professional system administration and DevOps practices.

**Inspect. Configure. Secure. Deploy. Monitor. Automate. Maintain.**


**Project Status:** In Progress  
**Repository:** `linux-server-infra`