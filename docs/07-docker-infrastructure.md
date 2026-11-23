# 07 — Docker Infrastructure

## 1. Objective

The objective of this phase was to install and configure Docker Engine on the Ubuntu Server and prepare the infrastructure to host multiple containerized applications.

The following topics were covered:

- Docker Engine installation.
- Docker service management.
- Docker user permissions.
- Docker Compose V2.
- Container lifecycle management.
- Docker networking.
- Persistent storage with Docker volumes.
- Container restart policies.
- Log inspection and troubleshooting.
- System configuration for Redis.

This phase introduces the core containerization concepts required for the application hosting and monitoring infrastructure implemented in subsequent phases.

## 2. Environment

| Component | Configuration |
|---|---|
| Operating System | Ubuntu Server 26.04.1 LTS |
| Hostname | `linux-server` |
| Container Engine | Docker Engine |
| Container Runtime | containerd |
| Compose | Docker Compose V2 |
| Test Services | Nginx and Redis |
| Virtualization | VirtualBox |

## 3. Docker Engine Installation

Docker Engine was installed using the official Docker APT repository.

### 3.1 Prepare the System

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

The package index was then updated:

```bash
sudo apt update
```

**Reproducibility note:** The original installation record identifies the key and repository file locations but does not preserve their exact contents or the commands used to create them. For a fully reproducible installation, retain the repository configuration and the corresponding official Docker installation instructions.

### 3.2 Install Docker Packages

Docker Engine and its associated components were installed:

```bash
sudo apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin -y
```

Installed components:

| Package | Purpose |
|---|---|
| `docker-ce` | Docker Engine |
| `docker-ce-cli` | Docker command-line interface |
| `containerd.io` | Container runtime |
| `docker-buildx-plugin` | Extended image build functionality |
| `docker-compose-plugin` | Docker Compose V2 |

### 3.3 Verify the Installation

Check Docker service status:

```bash
sudo systemctl status docker
```

Run the official test image:

```bash
sudo docker run hello-world
```

The successful execution confirmed that Docker Engine was operational and able to run a container.

## 4. Docker Service and User Permissions

### 4.1 Docker Service

Docker runs as a system service managed by systemd.

Useful commands:

```bash
sudo systemctl status docker
sudo systemctl start docker
sudo systemctl stop docker
sudo systemctl enable docker
```

These commands allow administrators to inspect, start, stop, and enable Docker at system startup.

### 4.2 Docker Group

The user was added to the Docker group to execute Docker commands without `sudo`:

```bash
sudo usermod -aG docker ramzile
```

The SSH session was closed and reopened to apply the new group membership.

Verification:

```bash
docker ps
```

The command executed successfully without requiring `sudo`.

### Security Consideration

Membership in the Docker group provides privileges equivalent to root access in practice.

A user with access to the Docker daemon can create privileged containers, mount host directories, and access sensitive host resources.

Docker group membership must therefore be restricted to trusted users.

Running Docker commands without `sudo` is a convenience, not a reduction in administrative privileges.

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

### 5.1 Compose Configuration

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

### 5.2 Configuration Explanation

**Nginx service**

- Uses the `nginx:alpine` image.
- Container name: `lab-nginx`.
- Publishes container port 80 through host port 8080.
- Uses the `unless-stopped` restart policy.

**Redis service**

- Uses the `redis:alpine` image.
- Container name: `lab-redis`.
- Uses a named volume to persist its data.
- Uses the `unless-stopped` restart policy.
- Does not publish its port to the host.

**Named volume**

The `redis_data` volume is mounted at `/data` inside the Redis container.

This allows Redis data stored in that location to remain available independently of the container lifecycle.

## 6. Container Management

Docker Compose provides commands to manage the lifecycle of application services.

### 6.1 Start Services

```bash
docker compose up -d
```

The `-d` option starts containers in detached mode.

### 6.2 List Running Containers

```bash
docker compose ps
```

This command displays the status of the services belonging to the Compose project.

### 6.3 Stop Services

Stop all services:

```bash
docker compose stop
```

Stop a specific service:

```bash
docker compose stop redis
```

The `stop` command stops containers without removing them.

### 6.4 Start a Stopped Service

```bash
docker compose start redis
```

This starts an existing stopped container.

### 6.5 Remove Containers and Networks

```bash
docker compose down
```

This command stops and removes the project's containers and Compose-created network.

Named volumes are preserved by default.

**Important:** Avoid using `docker compose down -v` unless volume deletion is explicitly intended. This option also removes the project's named volumes and their stored data.

### Understanding the Container Lifecycle

These commands have different purposes:

| Command | Effect |
|---|---|
| `docker compose up -d` | Creates and starts services |
| `docker compose ps` | Displays service status |
| `docker compose stop` | Stops containers without removing them |
| `docker compose start` | Starts existing stopped containers |
| `docker compose down` | Stops and removes containers and project network |
| `docker compose logs` | Displays service logs |

Understanding these differences is important when managing persistent application services.

## 7. Docker Networking

### 7.1 Default Compose Network

Docker Compose automatically created a network for the project:

```text
docker-lab_default
```

Network inspection:

```bash
docker network ls
```

The network uses the bridge driver.

By default, Compose creates a project network and connects the project's services to it.

### 7.2 Container-to-Container Communication

Services connected to the same Compose network can communicate using their service names.

For example, Redis is accessible from another service through:

```text
redis:6379
```

The Nginx container was used to test internal DNS resolution:

```bash
docker compose exec web getent hosts redis
```

Observed example:

```text
172.18.0.3 redis redis
```

The assigned IP address may differ between executions.

This confirmed that Docker's internal DNS resolution was working.

### Important Networking Concepts

- **Bridge Network:** A Docker network driver allowing containers to communicate through an isolated virtual network.
- **Service Discovery:** Docker's internal DNS allows services to locate one another by service name.
- **Published Port:** A container port exposed through a port on the Docker host.
- **Internal Container Port:** A port used by applications inside the container network.

Inside a container, `localhost` refers to that same container, not another service.

Services on the same Compose network should communicate using their service names.

Redis does not need a published host port for Nginx or other containers on the same network to reach it.

## 8. Persistent Storage with Docker Volumes

Containers are replaceable. Data stored only in a container's writable filesystem can be lost when the container is removed.

Docker volumes provide persistent storage independently of the container lifecycle.

### 8.1 Configure the Redis Volume

The Redis service uses the following volume configuration:

```yaml
services:
  redis:
    volumes:
      - redis_data:/data

volumes:
  redis_data:
```

The named volume `redis_data` is mounted at `/data` inside the Redis container.

### 8.2 Test Data Persistence

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

### 8.3 Recreate Containers

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

### Important Storage Considerations

- Removing a container does not automatically remove its named volumes.
- Removing a named volume deletes the data stored in that volume.
- Persistent volumes are not a substitute for backups.
- Data persistence also depends on the application writing its data to the mounted directory.

The Redis test demonstrated persistence across container removal and recreation, but it did not constitute a backup or recovery test.

## 9. Restart Policies

The following restart policy was configured for both services:

```yaml
restart: unless-stopped
```

This policy instructs Docker to restart containers automatically after failures or Docker daemon restarts, unless they have been explicitly stopped.

This is useful when hosting services that should remain available after a server reboot.

### Docker Restart Policies

Common restart policies include:

| Policy | Behavior |
|---|---|
| `no` | Does not automatically restart the container |
| `always` | Always attempts to restart the container |
| `on-failure` | Restarts the container when it exits with an error |
| `unless-stopped` | Restarts the container unless it was explicitly stopped |

The configured `unless-stopped` policy provides automatic recovery while respecting an intentional manual stop.

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

These commands are useful for identifying application startup errors, configuration problems, and runtime warnings.

During the Redis startup inspection, a memory overcommit warning was identified.

## 11. System Configuration for Redis

Redis recommended enabling Linux memory overcommit to reduce the risk of failures during background saves or replication.

### 11.1 Apply the Setting Immediately

```bash
sudo sysctl vm.overcommit_memory=1
```

This applies the setting to the running Linux kernel.

### 11.2 Make the Setting Persistent

A dedicated sysctl configuration file was created:

```bash
echo 'vm.overcommit_memory = 1' | sudo tee /etc/sysctl.d/99-docker.conf
```

The file allows the setting to be reapplied during system initialization.

### 11.3 Verify the Configuration

```bash
sysctl vm.overcommit_memory
```

Expected result:

```text
vm.overcommit_memory = 1
```

The setting was applied immediately and configured to persist across reboots.

### Understanding Memory Overcommit

Linux memory overcommit controls how the kernel handles requests for virtual memory.

The value `1` enables a permissive overcommit policy, allowing memory allocation requests to be accepted more readily.

Redis recommends this setting because certain background operations, including persistence and replication, may require additional memory.

**Important:** This is a system-wide kernel setting, not a Redis-specific configuration. It should be understood in the context of the host's memory capacity and workload.

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
- Published Docker ports must be reviewed carefully. The `8080:80` mapping publishes Nginx on all host network interfaces by default unless a specific host IP is supplied.
- Host firewall rules and Docker networking rules should both be considered when assessing external access.

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

A multi-container environment was created using Nginx and Redis. Container networking, service discovery, restart policies, lifecycle management, log inspection, and persistent storage were tested.

The Redis persistence test confirmed that application data remained available after containers were removed and recreated.

The Linux memory overcommit setting was also configured to support Redis background operations.

The server is now prepared for the next phase: application hosting and reverse proxy configuration.