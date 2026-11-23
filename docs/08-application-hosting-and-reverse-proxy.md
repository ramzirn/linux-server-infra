# Phase 08 — Application Hosting & Reverse Proxy

## 1. Objective

The objective of this phase was to deploy multiple containerized applications on a single Ubuntu Server and configure an Nginx Reverse Proxy to route incoming HTTP requests to the appropriate application.

Two separate application environments were deployed:

- **Staging:** used for testing and validating application changes.
- **Production:** represents a production-like application environment.

Both applications are accessible through the same Reverse Proxy using different hostnames.

This phase demonstrates how multiple applications can share a single HTTP entry point while running in separate containers.

**Scope clarification:** The Production environment in this lab is a learning environment. It does not represent a fully secured public production deployment.

## 2. Infrastructure Architecture

```text
                      Linux Mint Host
                            |
                   Browser / HTTP Requests
                            |
          staging.local:8081 / production.local:8081
                            |
                     VirtualBox NAT
                            |
                      Ubuntu Server
                        Port 8081
                            |
                   Docker Reverse Proxy
                        Nginx :80
                            |
                     phase8-network
                       /          \
                      /            \
               staging-app    production-app
                    |                |
                 Nginx :80        Nginx :80
                    |                |
               Staging HTML    Production HTML
```

The architecture uses a single Nginx Reverse Proxy to expose two applications through hostname-based routing.

The Reverse Proxy and both application containers communicate through a shared Docker bridge network.

When a client sends an HTTP request, Nginx examines the requested hostname and forwards the request to the corresponding application.

## 3. Prerequisites and Environment

The following components were used:

| Component | Configuration |
|---|---|
| Operating System | Ubuntu Server 26.04.1 LTS |
| Hostname | `linux-server` |
| Container Engine | Docker Engine |
| Orchestration | Docker Compose V2 |
| Reverse Proxy | Nginx |
| Application containers | Nginx Alpine |
| Virtualization | VirtualBox |
| Client system | Linux Mint |

Docker Engine and Docker Compose V2 must already be installed and operational.

The Phase 08 deployment also requires access to the Ubuntu Server through SSH and the ability to create Docker networks and manage containers.

## 4. Directory Structure

The Phase 08 configuration is organized as follows:

```text
/opt/stacks/phase8/
├── applications/
│   ├── staging/
│   │   ├── compose.yaml
│   │   └── html/
│   │       └── index.html
│   │
│   └── production/
│       ├── compose.yaml
│       └── html/
│           └── index.html
│
└── reverse-proxy/
    ├── compose.yaml
    └── nginx/
        └── conf.d/
            └── default.conf
```

Each application has its own Compose project, while the Reverse Proxy is managed independently.

This organization separates application configuration from reverse proxy configuration and makes each component easier to manage.

## 5. Docker Network

A dedicated Docker bridge network was created to allow communication between the Reverse Proxy and both applications.

```bash
docker network create phase8-network
```

The network is external to the three Compose projects.

This means that the network is created independently and referenced by each Compose configuration instead of being created and managed by one individual project.

Docker's internal DNS allows containers connected to this network to communicate using service names or configured network aliases.

The network aliases used in this deployment are:

| Application | Network alias |
|---|---|
| Staging | `staging-app` |
| Production | `production-app` |

**Important:** Staging and Production use separate containers and separate content, but they share the same Docker network. They are separated at the application-container level, not isolated from each other at the network level.

## 6. Staging Application

File: `applications/staging/compose.yaml`

```yaml
services:
  app:
    image: nginx:alpine
    container_name: phase8-staging
    restart: unless-stopped
    volumes:
      - ./html:/usr/share/nginx/html:ro
    networks:
      phase8-network:
        aliases:
          - staging-app

networks:
  phase8-network:
    external: true
```

The application uses Nginx to serve its HTML page.

The bind mount makes the local `html` directory available inside the container at `/usr/share/nginx/html`.

The `:ro` option mounts the directory in read-only mode, preventing the container from modifying the mounted HTML files.

The Docker network alias `staging-app` allows other containers on the shared network to resolve the application by name.

No host port is published for this application. Incoming requests must pass through the Reverse Proxy.

## 7. Production Application

File: `applications/production/compose.yaml`

```yaml
services:
  app:
    image: nginx:alpine
    container_name: phase8-production
    restart: unless-stopped
    volumes:
      - ./html:/usr/share/nginx/html:ro
    networks:
      phase8-network:
        aliases:
          - production-app

networks:
  phase8-network:
    external: true
```

Production follows the same structure as Staging but serves a separate HTML page.

The two environments run in independent containers with separate content.

The `production-app` network alias allows the Reverse Proxy to reach this application through Docker's internal DNS.

Like Staging, Production does not publish a host port.

## 8. Reverse Proxy Configuration

File: `reverse-proxy/compose.yaml`

```yaml
services:
  reverse-proxy:
    image: nginx:alpine
    container_name: phase8-reverse-proxy
    restart: unless-stopped
    ports:
      - "8081:80"
    volumes:
      - ./nginx/conf.d:/etc/nginx/conf.d:ro
    networks:
      - phase8-network

networks:
  phase8-network:
    external: true
```

The Reverse Proxy is the HTTP entry point for both applications.

Port mapping:

| Port | Description |
|---|---|
| `8081` | Port published on the Ubuntu Server |
| `80` | HTTP port inside the Reverse Proxy container |

The mapping `8081:80` forwards incoming connections on server port 8081 to port 80 inside the container.

The Reverse Proxy is the only Phase 08 application container publishing a host port.

By default, this mapping binds to the host's available network interfaces. It is not restricted to loopback. Host firewall rules and network exposure must therefore be considered when deploying a similar configuration outside this lab.

The Nginx configuration directory is mounted read-only inside the container.

## 9. Nginx Virtual Host Configuration

File: `reverse-proxy/nginx/conf.d/default.conf`

```nginx
server {
    listen 80;
    server_name staging.local;

    location / {
        proxy_pass http://staging-app:80;

        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}

server {
    listen 80;
    server_name production.local;

    location / {
        proxy_pass http://production-app:80;

        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

Two Nginx virtual hosts were configured.

The `server_name` directive identifies the hostname requested by the client.

The `proxy_pass` directive forwards the HTTP request to the corresponding application using its Docker network alias.

The `proxy_set_header` directives forward useful information about the original HTTP request:

- `Host`: original requested hostname.
- `X-Real-IP`: client IP address as seen by the Reverse Proxy.
- `X-Forwarded-For`: forwarded client IP information.
- `X-Forwarded-Proto`: original request protocol.

This configuration allows both applications to share the same Reverse Proxy while remaining accessible through separate hostnames.

### How hostname-based routing works

For a request to `staging.local`, the client sends an HTTP request containing the hostname in its `Host` header.

Nginx matches that hostname against its configured virtual hosts and forwards the request to `staging-app:80`.

The same process applies to `production.local`, which is forwarded to `production-app:80`.

Docker's internal DNS resolves these aliases to the corresponding containers on `phase8-network`.

## 10. Local Hostname Configuration

On the Linux Mint host, the following entries were added to `/etc/hosts`:

```text
127.0.0.1 staging.local production.local
```

These entries resolve both hostnames to the local machine.

VirtualBox NAT port forwarding was configured to forward host port `8081` to guest port `8081`.

This allows the host browser to access the applications using:

- http://staging.local:8081
- http://production.local:8081

The request follows this path:

1. Linux Mint resolves the hostname to `127.0.0.1`.
2. The browser connects to host port `8081`.
3. VirtualBox forwards the connection to port `8081` on the Ubuntu Server.
4. Docker forwards the connection to port `80` inside the Reverse Proxy container.
5. Nginx selects the appropriate virtual host using the requested hostname.
6. The request is forwarded to the corresponding application container.

These hostnames are used exclusively for local development and testing. They are not publicly registered domains.

## 11. Deployment Commands

The following steps describe how the three Compose projects are deployed.

### Step 1 — Create the shared network

```bash
docker network create phase8-network
```

This creates the external bridge network used by all three projects.

If the network already exists, it should not be created again. It can be checked with:

```bash
docker network inspect phase8-network
```

### Step 2 — Deploy Staging

```bash
cd /opt/stacks/phase8/applications/staging
docker compose up -d
```

### Step 3 — Deploy Production

```bash
cd /opt/stacks/phase8/applications/production
docker compose up -d
```

### Step 4 — Validate Nginx configuration

```bash
cd /opt/stacks/phase8/reverse-proxy
docker compose run --rm reverse-proxy nginx -t
```

This command creates a temporary container using the Reverse Proxy Compose configuration and runs the Nginx configuration test.

The `nginx -t` command checks the syntax and validity of the Nginx configuration files.

The `--rm` option removes the temporary container after the command finishes.

A successful configuration test should report that the syntax is valid and the configuration test is successful.

### Step 5 — Deploy the Reverse Proxy

```bash
docker compose up -d
```

The Reverse Proxy starts and becomes the entry point for both applications.

## 12. Validation and Testing

### 12.1 Container Status

The running containers were inspected using:

```bash
docker ps
```

All three Phase 08 containers were running:

- `phase8-staging`
- `phase8-production`
- `phase8-reverse-proxy`

### 12.2 Direct Application Testing

Each application was tested directly from inside its container:

```bash
docker exec phase8-staging wget -qO- http://localhost
docker exec phase8-production wget -qO- http://localhost
```

Both applications returned their respective HTML pages.

These tests verified that the application containers were serving their content independently of the Reverse Proxy.

### 12.3 Reverse Proxy Routing

Hostname-based routing was tested using HTTP requests with explicit `Host` headers:

```bash
curl -H "Host: staging.local" http://localhost:8081

curl -H "Host: production.local" http://localhost:8081
```

Both requests returned the correct application content.

This confirmed that Nginx was routing requests to the appropriate application based on the requested hostname.

### 12.4 Browser Testing

The following URLs were tested successfully from the Linux Mint host:

- http://staging.local:8081
- http://production.local:8081

Both applications were accessible through the Reverse Proxy.

### 12.5 Logs

Reverse Proxy logs were inspected using:

```bash
docker compose logs --tail=30 reverse-proxy
```

The logs confirmed successful HTTP requests with status code `200`.

A `404` response for `/favicon.ico` was also observed because no favicon was created.

This does not affect the main application functionality.

### 12.6 Restart Validation

The Ubuntu Server was rebooted.

After reconnecting, all five lab containers were running:

- `phase8-reverse-proxy`
- `phase8-staging`
- `phase8-production`
- `lab-nginx`
- `lab-redis`

Both application URLs remained accessible after the reboot.

This confirmed that the containers configured with restart policies were able to start again after the server reboot and that the Phase 08 applications remained accessible.

## 13. Important Concepts Learned

This phase introduced and demonstrated the following concepts:

- Docker Compose projects and service management.
- Shared Docker bridge networks.
- Docker DNS and network aliases.
- Container-to-container communication.
- Bind mounts and read-only mounts.
- Nginx Reverse Proxy.
- HTTP Host headers and virtual hosts.
- Internal versus published container ports.
- VirtualBox NAT port forwarding.
- Container restart policies.
- Nginx logs and configuration validation.
- Separation of application configuration and reverse proxy configuration.

## 14. Security Considerations

The following security measures and limitations apply to this environment:

- Staging and Production do not publish ports directly on the Ubuntu host.
- Only the Reverse Proxy publishes a port for Phase 08.
- Application HTML directories are mounted read-only.
- The Reverse Proxy configuration is mounted read-only.
- Staging and Production use separate containers and content, but share a Docker network.
- The current setup uses HTTP and local hostnames.
- The Reverse Proxy port mapping listens on all host network interfaces by default unless a specific host IP is supplied.
- Host firewall rules and Docker networking rules must both be considered when assessing external access.
- The environment is intended for learning and testing, not public production deployment.

For a real deployment, additional measures would be required, including HTTPS, proper DNS configuration, appropriate firewall rules, application security, access controls, monitoring and backup procedures.

## 15. Conclusion

Phase 08 was completed successfully.

Two separate application environments were deployed using Docker and connected through a shared Docker bridge network.

An Nginx Reverse Proxy was configured to route incoming HTTP requests according to their requested hostnames.

Both applications were successfully tested directly, through the Reverse Proxy and from the Linux Mint host browser.

Their availability was also verified after a server reboot.

This phase introduced the practical use of Docker networking, internal service discovery, reverse proxy configuration and hostname-based application routing.

The server can now host multiple applications through a single HTTP entry point.