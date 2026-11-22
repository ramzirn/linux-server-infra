# Phase 08 — Application Hosting & Reverse Proxy

## 1. Overview

The objective of this phase was to deploy multiple applications on a single Ubuntu Server using Docker and configure an Nginx Reverse Proxy to route incoming HTTP requests to the correct application.

Two independent environments were deployed:

- **Staging:** used for testing and validation.
- **Production:** represents the production application environment.

Both applications are accessible through the Reverse Proxy using different hostnames.

## 2. Infrastructure Architecture

```text
                    Linux Mint Host
                          |
                 Browser / HTTP requests
                          |
              staging.local:8081
              production.local:8081
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
                    /       \
                   /         \
          staging-app     production-app
               |                |
          Nginx :80         Nginx :80
               |                |
          Staging HTML     Production HTML
```

## 3. Directory Structure

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

## 4. Docker Network

A dedicated Docker bridge network was created to allow communication between the Reverse Proxy and the two applications.

```bash
docker network create phase8-network
```

The network is external to the three Compose projects, allowing all of them to share it.

## 5. Staging Application

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

The bind mount makes the local `html` directory available inside the container in read-only mode.

The Docker network alias `staging-app` allows other containers on the shared network to resolve the application by name.

## 6. Production Application

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

The two environments are independent containers with separate content.

## 7. Reverse Proxy Configuration

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

Port mapping:

- `8081`: port exposed on the Ubuntu Server.
- `80`: HTTP port inside the Reverse Proxy container.

The Reverse Proxy is the only Phase 08 application container exposing a host port.

## 8. Nginx Virtual Host Configuration

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

The `server_name` directive identifies the requested hostname.

The `proxy_pass` directive forwards the request to the corresponding application using Docker DNS.

The `proxy_set_header` directives preserve useful information about the original HTTP request.

## 9. Local Hostname Configuration

On the Linux Mint host, the following entries were added to `/etc/hosts`:

```text
127.0.0.1 staging.local production.local
```

VirtualBox NAT port forwarding was configured to forward host port `8081` to guest port `8081`.

This allows the host browser to access the applications using:

- http://staging.local:8081
- http://production.local:8081

These hostnames are for local development and testing, not publicly registered domains.

## 10. Deployment Commands

Create the shared network:

```bash
docker network create phase8-network
```

Deploy Staging:

```bash
cd /opt/stacks/phase8/applications/staging
docker compose up -d
```

Deploy Production:

```bash
cd /opt/stacks/phase8/applications/production
docker compose up -d
```

Validate Nginx configuration:

```bash
cd /opt/stacks/phase8/reverse-proxy
docker compose run --rm reverse-proxy nginx -t
```

Deploy Reverse Proxy:

```bash
docker compose up -d
```

## 11. Validation and Testing

### Container status

```bash
docker ps
```

All three Phase 08 containers were running.

### Direct application testing

```bash
docker exec phase8-staging wget -qO- http://localhost
docker exec phase8-production wget -qO- http://localhost
```

Both applications returned their respective HTML pages.

### Reverse Proxy routing

```bash
curl -H "Host: staging.local" http://localhost:8081

curl -H "Host: production.local" http://localhost:8081
```

Both requests returned the correct application content.

### Browser testing

The following URLs were tested successfully:

- http://staging.local:8081
- http://production.local:8081

### Logs

```bash
docker compose logs --tail=30 reverse-proxy
```

The logs confirmed successful HTTP requests with status code `200`.

A `404` response for `/favicon.ico` was observed because no favicon was created. This does not affect application functionality.

### Restart validation

The Ubuntu Server was rebooted.

After reconnecting, all five lab containers were running:

- `phase8-reverse-proxy`
- `phase8-staging`
- `phase8-production`
- `lab-nginx`
- `lab-redis`

Both application URLs remained accessible.

## 12. Important Concepts Learned

- Docker Compose projects and service management.
- Shared Docker bridge networks.
- Docker DNS and network aliases.
- Container-to-container communication.
- Bind mounts and read-only volumes.
- Nginx Reverse Proxy.
- HTTP Host headers and virtual hosts.
- Internal versus published container ports.
- VirtualBox NAT port forwarding.
- Container restart policies.
- Nginx logs and configuration validation.

## 13. Security Considerations

- Staging and Production do not publish ports directly on the Ubuntu host.
- Only the Reverse Proxy publishes a port for Phase 08.
- Application HTML directories are mounted read-only.
- The Reverse Proxy configuration is mounted read-only.
- The current setup uses HTTP and local hostnames; it is a learning environment, not a public production deployment.

For a real deployment, HTTPS, proper DNS, firewall rules, application security and access controls must be configured.

## 14. Phase Status

**Status: Completed**

The two applications were successfully deployed and routed through a single Nginx Reverse Proxy.

Browser access, Docker networking, logs and restart behavior were validated.