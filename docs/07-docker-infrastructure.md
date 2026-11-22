# Phase 07 — Docker Infrastructure

## 1. Objectives

The objective of this phase was to install and configure Docker Engine on the Ubuntu Server and prepare the infrastructure to host multiple containerized applications.

The following topics were covered:

- Docker Engine installation
- Docker service management
- Docker user permissions
- Docker Compose V2
- Container lifecycle management
- Docker networking
- Persistent storage with Docker volumes
- Container restart policies
- Logs inspection and troubleshooting
- System configuration for Redis

## 2. Environment

| Component | Configuration |
|---|---|
| Operating System | Ubuntu Server 26.04.1 LTS |
| Hostname | linux-server |
| Container Engine | Docker Engine |
| Container Runtime | containerd |
| Compose | Docker Compose V2 |
| Test Services | Nginx and Redis |
| Virtualization | VirtualBox |

## 3. Docker Engine Installation

### 3.1 Prepare the system

The required packages were installed:

```bash
sudo apt install ca-certificates curl -y
```

The official Docker GPG key was downloaded and stored in:

```text
/etc/apt/keyrings/docker.asc
```

The official Docker APT repository was configured using:

```text
/etc/apt/sources.list.d/docker.sources
```

The package index was updated:

```bash
sudo apt update
```

### 3.2 Install Docker packages

Docker Engine and its associated components were installed:

```bash
sudo apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin -y
```

Installed components:

- `docker-ce`: Docker Engine.
- `docker-ce-cli`: Docker command-line interface.
- `containerd.io`: Container runtime.
- `docker-buildx-plugin`: Extended image build functionality.
- `docker-compose-plugin`: Docker Compose V2.

### 3.3 Verify the installation

Check Docker service status:

```bash
sudo systemctl status docker
```

Run the official test image:

```bash
sudo docker run hello-world
```

The successful execution confirmed that Docker Engine was operational.

## 4. Docker Service and User Permissions

### 4.1 Docker service

Docker runs as a system service.

Useful commands:

```bash
sudo systemctl status docker
sudo systemctl start docker
sudo systemctl stop docker
sudo systemctl enable docker
```

### 4.2 Docker group

The user was added to the Docker group to execute Docker commands without `sudo`:

```bash
sudo usermod -aG docker ramzile
```

The SSH session was closed and reopened to apply the new group membership.

Verification:

```bash
docker ps
```

**Security consideration:**

Membership in the Docker group provides privileges equivalent to root access in practice. Access to this group must therefore be restricted to trusted users.

## 5. Docker Compose V2

Docker Compose V2 was installed through the Docker Compose plugin.

Verify the version:

```bash
docker compose version
```

Docker Compose allows multiple services to be defined and managed through a single YAML configuration file.

The first practical project was created in:

```text
/home/ramzile/docker-lab/
```

### 5.1 Compose configuration

The following `compose.yaml` file was created:

```yaml
services:
  web:
    image: nginx:alpine
    container_name: lab-nginx
    restart: unless-stopped
    ports:
      - "8080:80"

  redis:
    image: redis:alpine
    container_name: lab-redis
    restart: unless-stopped
    volumes:
      - redis_data:/data

volumes:
  redis_data:
```

### 5.2 Configuration explanation

**Nginx service**

- Uses the `nginx:alpine` image.
- Container name: `lab-nginx`.
- Exposes port 80 inside the container through port 8080 on the server.
- Uses an automatic restart policy.

**Redis service**

- Uses the `redis:alpine` image.
- Container name: `lab-redis`.
- Uses a named volume to persist its data.
- Uses an automatic restart policy.

**Named volume**

The `redis_data` volume is mounted at `/data` inside the Redis container.

## 6. Container Management

### 6.1 Start services

```bash
docker compose up -d
```

The `-d` option starts containers in detached mode.

### 6.2 List running containers

```bash
docker compose ps
```

### 6.3 Stop services

```bash
docker compose stop
```

Stop a specific service:

```bash
docker compose stop redis
```

### 6.4 Start a stopped service

```bash
docker compose start redis
```

### 6.5 Remove containers and networks

```bash
docker compose down
```

This command removes the containers and the Compose-created network.

Named volumes are preserved by default.

**Important:** Avoid using `docker compose down -v` unless volume deletion is explicitly intended. This option removes the project's named volumes and their stored data.

## 7. Docker Networking

### 7.1 Default Compose network

Docker Compose automatically created a network for the project:

```text
docker-lab_default
```

Network inspection:

```bash
docker network ls
```

The network uses the bridge driver.

### 7.2 Container-to-container communication

Services connected to the same Compose network can communicate using their service names.

For example:

```text
redis:6379
```

The Nginx container was used to test DNS resolution:

```bash
docker compose exec web getent hosts redis
```

Expected result:

```text
172.18.0.3 redis redis
```

The assigned IP may differ between executions.

This confirmed that Docker's internal DNS resolution was working.

**Important:** Inside a container, `localhost` refers to that same container, not another service.

## 8. Persistent Storage with Docker Volumes

Containers are replaceable. Data stored only in a container's writable filesystem can be lost when the container is removed.

Docker volumes provide persistent storage independently of the container lifecycle.

### 8.1 Configure the Redis volume

The following configuration was added to `compose.yaml`:

```yaml
volumes:
  - redis_data:/data
```

The volume was declared at the end of the Compose file:

```yaml
volumes:
  redis_data:
```

### 8.2 Test data persistence

A test key was created in Redis:

```bash
docker compose exec redis redis-cli SET demo "donnee-persistante"
```

Retrieve the value:

```bash
docker compose exec redis redis-cli GET demo
```

Expected result:

```text
"donnee-persistante"
```

### 8.3 Recreate containers

The containers were removed:

```bash
docker compose down
```

They were recreated:

```bash
docker compose up -d
```

The stored value was retrieved again:

```bash
docker compose exec redis redis-cli GET demo
```

Expected result:

```text
"donnee-persistante"
```

**Result:** The data remained available after container removal and recreation. This confirmed that the named volume was working correctly.

## 9. Restart Policies

The following restart policy was configured for both services:

```yaml
restart: unless-stopped
```

This policy instructs Docker to restart containers automatically after failures or Docker daemon restarts, unless they have been explicitly stopped.

This is useful when hosting services that should remain available after a server reboot.

## 10. Logs and Troubleshooting

Docker Compose provides commands to inspect application logs.

Display logs for all services:

```bash
docker compose logs
```

Display logs for Nginx:

```bash
docker compose logs web
```

Display logs for Redis:

```bash
docker compose logs redis
```

Follow logs continuously:

```bash
docker compose logs -f
```

During the Redis startup inspection, a memory overcommit warning was identified.

## 11. System Configuration for Redis

Redis recommended enabling Linux memory overcommit to reduce the risk of failures during background saves or replication.

### 11.1 Apply the setting immediately

```bash
sudo sysctl vm.overcommit_memory=1
```

### 11.2 Make the setting persistent

A dedicated sysctl configuration file was created:

```bash
echo 'vm.overcommit_memory = 1' | sudo tee /etc/sysctl.d/99-docker.conf
```

### 11.3 Verify the configuration

```bash
sysctl vm.overcommit_memory
```

Expected result:

```text
vm.overcommit_memory = 1
```

## 12. Security Considerations

The lab was designed for learning and local infrastructure testing.

The following points should be considered before using a similar configuration in production:

- Docker group membership grants highly privileged access.
- Redis should be protected with appropriate authentication and network restrictions.
- Database ports should not be unnecessarily exposed to external networks.
- Production images should use deliberately selected versions rather than relying blindly on mutable tags.
- Persistent data must be backed up.
- Secrets should not be stored directly in public Compose files.
- Staging and production environments should use separate configurations and data.

## 13. Verification and Results

The following checks were completed during this phase:

| Verification | Result |
|---|---|
| Docker Engine installation | Successful |
| Docker service | Active |
| `hello-world` test | Successful |
| Docker commands without sudo | Successful |
| Docker Compose V2 | Verified |
| Nginx container startup | Successful |
| Redis container startup | Successful |
| Compose network creation | Successful |
| Internal DNS resolution | Successful |
| Redis named volume | Created |
| Redis data persistence | Verified |
| Container stop/start | Verified |
| Container recreation | Verified |
| Redis memory overcommit configuration | Applied and made persistent |

## 14. Future Infrastructure Organization

As the number of hosted applications grows, Docker Compose projects can be organized independently.

A possible future directory structure is:

```text
/opt/stacks/
├── odoo/
│   ├── staging/
│   └── production/
├── laravel/
│   ├── staging/
│   └── production/
└── monitoring/
```

Application data can be stored separately:

```text
/srv/app-data/
├── odoo/
│   ├── staging/
│   └── production/
└── laravel/
    ├── staging/
    └── production/
```

This is a proposed future organization, not a configuration deployed during this phase.

## 15. Conclusion

Docker Engine and Docker Compose V2 were successfully installed and tested on the Ubuntu Server.

A multi-container environment was created using Nginx and Redis. Container networking, service discovery, restart policies, lifecycle management, logs inspection, and persistent storage were tested.

The Redis persistence test confirmed that application data remained available after containers were removed and recreated.

The server is now prepared for the next phase: hosting an application behind a reverse proxy.
