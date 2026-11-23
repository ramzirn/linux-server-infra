# Linux Server Infrastructure — DevOps Learning Lab

A hands-on Linux infrastructure project covering system administration, security hardening, containerization, application hosting, and monitoring.

This repository documents the progressive implementation of a Linux server environment, from initial system configuration to Docker-based application deployment and centralized monitoring.

The project is designed as a practical learning lab for people interested in Linux system administration, infrastructure management, Docker, and DevOps fundamentals.

## Project Overview

The infrastructure is built around an Ubuntu Server virtual machine running in Oracle VirtualBox and administered remotely from a Linux Mint host.

The project follows nine progressive phases, covering:

- Linux server installation and initial configuration.
- Network configuration and secure SSH remote access.
- User, group, and permission management.
- Linux security hardening.
- Storage and filesystem management using LVM.
- Docker infrastructure and container management.
- Application hosting using Docker and Nginx Reverse Proxy.
- Infrastructure monitoring and alerting with Prometheus and Grafana.

Each phase introduces practical concepts and builds upon the configuration established in previous phases.

## Infrastructure Architecture

The project is organized into three main areas:

**1. Linux System Administration**
- Ubuntu Server.
- SSH remote administration.
- User and permission management.
- UFW firewall and Fail2ban.
- AppArmor.
- LVM storage management.

**2. Application Infrastructure**
- Docker Engine and Docker Compose.
- Independent Staging and Production application environments.
- Nginx Reverse Proxy.
- Docker bridge networking.
- Containerized services.

**3. Monitoring and Observability**
- Prometheus for metrics collection and storage.
- Node Exporter for Linux system metrics.
- cAdvisor for container metrics.
- Grafana for visualization.
- Alertmanager for alert management.

### High-Level Architecture

```text
                  LINUX MINT HOST
                        |
                    VirtualBox
                        |
                  UBUNTU SERVER
                        |
          +-------------+-------------+
          |             |             |
       SYSTEM       APPLICATION    MONITORING
    ADMINISTRATION INFRASTRUCTURE INFRASTRUCTURE
          |             |             |
       SSH / UFW     Docker        Prometheus
       Fail2ban      Compose       Node Exporter
       AppArmor      Nginx         cAdvisor
       Users         Staging       Grafana
       Permissions   Production    Alertmanager
       LVM           Reverse Proxy
```

## Technology Stack

| Technology | Purpose |
|---|---|
| Ubuntu Server 26.04.1 LTS | Server operating system |
| Oracle VirtualBox | Virtualization platform |
| Linux Mint | Host operating system and administration workstation |
| OpenSSH | Secure remote administration |
| UFW | Firewall management |
| Fail2ban | Protection against repeated authentication attempts |
| AppArmor | Mandatory access control |
| LVM | Logical volume and storage management |
| Docker Engine | Containerization |
| Docker Compose | Multi-container deployment |
| Nginx | Web server and reverse proxy |
| Prometheus | Metrics collection and storage |
| Node Exporter | Linux system metrics |
| cAdvisor | Container metrics |
| Grafana | Monitoring dashboards and visualization |
| Alertmanager | Alert management |

## Project Documentation

The project is divided into nine phases. Each phase has its own technical documentation in the `docs/` directory.

| Phase | Documentation | Main Topics |
|---|---|---|
| 01 | [Network and Remote Access](docs/01-network-and-remote-access.md) | VirtualBox networking, SSH, authentication keys, UFW |
| 02 | [Initial System Configuration](docs/02-initial-system-configuration.md) | Hostname, timezone, APT, essential tools, automatic updates |
| 03 | [System Inspection](docs/03-system-inspection.md) | CPU, memory, storage, networking, services, processes |
| 04 | [User and Access Management](docs/04-user-and-access-management.md) | Users, groups, sudo, ownership, permissions, SGID |
| 05 | [Security Hardening](docs/05-security-hardening.md) | SSH hardening, UFW, Fail2ban, AppArmor, service inspection |
| 06 | [System Services and Storage Management](docs/06-system-services-and-storage-management.md) | systemd, filesystems, mount points, LVM, disk capacity |
| 07 | [Docker Infrastructure](docs/07-docker-infrastructure.md) | Docker Engine, Compose, containers, networks, persistent storage |
| 08 | [Application Hosting and Reverse Proxy](docs/08-application-hosting-and-reverse-proxy.md) | Staging and Production environments, Docker networking, Nginx reverse proxy |
| 09 | [Monitoring and Observability](docs/09-monitoring-and-observability.md) | Prometheus, Node Exporter, cAdvisor, Grafana, Alertmanager |

For a guided introduction and the recommended learning order, see the [Documentation Guide](docs/README.md).

## Repository Structure

```text
linux-server-infra/
│
├── README.md
│
└── docs/
    ├── README.md
    ├── 01-network-and-remote-access.md
    ├── 02-initial-system-configuration.md
    ├── 03-system-inspection.md
    ├── 04-user-and-access-management.md
    ├── 05-security-hardening.md
    ├── 06-system-services-and-storage-management.md
    ├── 07-docker-infrastructure.md
    ├── 08-application-hosting-and-reverse-proxy.md
    └── 09-monitoring-and-observability.md
```

The repository contains the technical documentation. The operational configuration files and deployed services are maintained on the Ubuntu Server in their respective directories.

## Environment and Prerequisites

### Implementation Environment

The project was implemented using:

| Component | Configuration |
|---|---|
| Host OS | Linux Mint |
| Virtualization | Oracle VirtualBox |
| Guest OS | Ubuntu Server 26.04.1 LTS |
| Server hostname | `linux-server` |
| Container platform | Docker Engine and Docker Compose |

### Requirements for Reproducing the Lab

To follow the project, you will need:

- A computer capable of running a virtual machine.
- Oracle VirtualBox or an equivalent virtualization platform.
- An Ubuntu Server virtual machine.
- An Internet connection for package installation and container image downloads.
- Sufficient CPU, memory, and disk space for the virtual machine and deployed services.
- Basic familiarity with the Linux command line.

Beginners can follow the documentation progressively. Basic networking and Linux command-line knowledge will make the experience easier.

Some operations require administrative privileges through `sudo`.

## How to Use This Repository

The recommended approach is to follow the nine phases in numerical order.

1. Start with the [Documentation Guide](docs/README.md).
2. Follow each phase sequentially.
3. Read the objectives and explanations before executing commands.
4. Pay attention to configuration files and command outputs.
5. Perform the documented verification steps before moving to the next phase.
6. Adapt environment-specific values, such as IP addresses, paths, and ports, to your own setup.

The documentation can also be used as a reference for individual topics such as SSH hardening, LVM, Docker networking, Nginx reverse proxy, and Prometheus monitoring.

**Important:** The documents describe the actual implementation of this lab. Some commands and configuration values are specific to the original environment and may require adaptation.

## Security and Production Considerations

This repository represents a learning and testing environment. It is not intended to be deployed directly as a production infrastructure.

The project introduces several important security practices, including:

- SSH key-based authentication.
- Firewall configuration.
- Fail2ban and AppArmor.
- Restricted access to application containers.
- Local access to the Grafana interface through SSH tunneling.

However, additional measures would be required before using a similar environment in production:

- Proper DNS and HTTPS configuration.
- Secure secrets and credential management.
- Regular backup and recovery procedures.
- Further container security hardening.
- External monitoring notification channels.
- Detailed network and firewall access reviews.
- Application-level security controls.
- High availability and disaster recovery planning.

Known limitations and possible improvements are documented in the relevant phase documents.

## Project Status

The nine phases have been implemented and documented.

The core infrastructure is deployed, and its main components have been tested, including remote server access, Docker-based application hosting, monitoring data collection, Grafana visualization, and alert validation.

Some improvements remain possible, particularly Docker dashboard compatibility, external alert notifications, and additional container security hardening.

The repository represents a completed practical learning lab that can be extended through future infrastructure and DevOps projects.

## Author

A practical infrastructure project developed to strengthen hands-on skills in Linux system administration, networking, security, Docker, application hosting, and monitoring.