# Documentation Guide

This directory contains the technical documentation for the **Linux Server Infrastructure — DevOps Learning Lab** project.

The documentation is organized into nine progressive phases. Each phase covers a specific part of building, configuring, securing, and monitoring a Linux server infrastructure.

The recommended approach is to follow the phases in order, as later phases build on the configuration and concepts introduced earlier.

## Learning Path

| Phase | Topic | Description |
|---|---|---|
| 01 | [Network and Remote Access](01-network-and-remote-access.md) | Set up the virtual machine network, configure SSH access, and establish secure remote administration. |
| 02 | [Initial System Configuration](02-initial-system-configuration.md) | Configure the hostname, timezone, system updates, essential tools, and automatic security updates. |
| 03 | [System Inspection](03-system-inspection.md) | Inspect system resources, storage, network configuration, running services, and listening ports. |
| 04 | [User and Access Management](04-user-and-access-management.md) | Create users and groups, configure administrative privileges, and manage directory permissions. |
| 05 | [Security Hardening](05-security-hardening.md) | Apply security measures using UFW, Fail2ban, AppArmor, SSH hardening, and service management. |
| 06 | [System Services and Storage Management](06-system-services-and-storage-management.md) | Inspect system services, understand filesystem mounts and LVM, and extend available disk capacity. |
| 07 | [Docker Infrastructure](07-docker-infrastructure.md) | Install and configure Docker, understand container management, and prepare the environment for containerized services. |
| 08 | [Application Hosting and Reverse Proxy](08-application-hosting-and-reverse-proxy.md) | Deploy separate staging and production applications using Docker and configure Nginx as a reverse proxy. |
| 09 | [Monitoring and Observability](09-monitoring-and-observability.md) | Deploy Prometheus, Node Exporter, cAdvisor, Grafana, and Alertmanager to monitor the server and containers. |

## Environment and Prerequisites

The project was implemented using the following environment:

| Component | Configuration |
|---|---|
| Host operating system | Linux Mint |
| Virtualization | Oracle VirtualBox |
| Guest operating system | Ubuntu Server 26.04.1 LTS |
| Server hostname | `linux-server` |
| Container platform | Docker Engine and Docker Compose |
| Application server | Nginx |
| Monitoring stack | Prometheus, Node Exporter, cAdvisor, Grafana, Alertmanager |

To reproduce the project, you will need:

- A computer capable of running a virtual machine.
- Oracle VirtualBox or an equivalent virtualization platform.
- An Ubuntu Server installation.
- An Internet connection for package installation and image downloads.
- Sufficient CPU, RAM, and disk space for the virtual machine and Docker services.
- Basic familiarity with the Linux command line. Beginners can also use the documentation to learn the commands progressively.

Some steps require administrative privileges through `sudo`.

## How to Use This Documentation

1. Start with Phase 01 and follow the recommended order.
2. Read the objective and explanations before executing commands.
3. Execute commands in the appropriate environment: host machine or Ubuntu Server virtual machine.
4. Compare your results with the documented outputs where available.
5. Make sure each phase is working before moving to the next one.
6. Keep track of configuration changes and adapt commands when your environment differs from the documented setup.

**Important:** These documents describe the actual implementation of this learning lab. Some commands, IP addresses, paths, and configuration values are specific to the original environment and may need to be adapted.

## Important Notes

### Lab Environment

This project is designed as a practical learning environment, not as a production-ready infrastructure.

Some configurations are intentionally simplified to make the concepts easier to understand and practice.

### Security

The project introduces several important security mechanisms, including SSH key authentication, UFW, Fail2ban, AppArmor, and restricted container access.

However, completing these phases does not guarantee production-level security.

### Known Limitations and Future Improvements

Some components can be improved beyond the current implementation. Examples include:

- Configuring HTTPS and proper DNS for application hosting.
- Improving Docker monitoring dashboards.
- Configuring external alert notifications.
- Reviewing container privileges and applying additional security restrictions.
- Implementing a backup and recovery strategy.

These improvements can be explored as future extensions of the project.

## Project Status

The nine phases document the implementation of the Linux server infrastructure, from initial system configuration to application hosting and monitoring.

The core learning environment is deployed and its main components have been tested. Some optional improvements remain documented as future work.

---

**Start here:** [Phase 01 — Network and Remote Access](01-network-and-remote-access.md)