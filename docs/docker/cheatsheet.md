# Docker Cheat Sheet

> **Module:** 12 - Docker Cheat Sheet
>
> **Purpose:** Quick revision for daily work, interviews, and production troubleshooting.

---

# Docker Workflow

```text
Write Dockerfile
        │
        ▼
docker build
        │
        ▼
Docker Image
        │
        ▼
docker push
        │
        ▼
Registry
        │
        ▼
docker pull
        │
        ▼
docker run
        │
        ▼
Running Container
```

---

# Docker Architecture

```text
Docker CLI
      │
      ▼
Docker API
      │
      ▼
dockerd
      │
      ▼
containerd
      │
      ▼
runc
      │
      ▼
Linux Kernel
```

---

# Image vs Container

| Image      | Container          |
| ---------- | ------------------ |
| Blueprint  | Running instance   |
| Read-only  | Read + Write layer |
| Built once | Can run many times |
| Immutable  | Temporary          |

---

# Image Commands

```bash
docker build -t app:v1 .
docker images
docker history app:v1
docker inspect app:v1
docker image rm app:v1
docker image prune
```

---

# Container Commands

```bash
docker run nginx

docker run -d nginx

docker ps

docker ps -a

docker stop <container>

docker start <container>

docker restart <container>

docker rm <container>

docker exec -it <container> sh

docker logs <container>

docker inspect <container>

docker stats
```

---

# Dockerfile Instructions

| Instruction | Purpose                      |
| ----------- | ---------------------------- |
| FROM        | Base image                   |
| RUN         | Build-time command           |
| COPY        | Copy files                   |
| ADD         | Copy + archive extraction    |
| WORKDIR     | Working directory            |
| ENV         | Runtime environment variable |
| ARG         | Build-time variable          |
| EXPOSE      | Document listening port      |
| USER        | Run as user                  |
| CMD         | Default command              |
| ENTRYPOINT  | Main executable              |

---

# Storage

| Storage Type     | Best Use                |
| ---------------- | ----------------------- |
| Writable Layer   | Temporary runtime files |
| Named Volume     | Databases               |
| Bind Mount       | Development             |
| Anonymous Volume | Temporary persistence   |
| tmpfs            | RAM-only storage        |

---

# Volume Commands

```bash
docker volume create data

docker volume ls

docker volume inspect data

docker volume rm data

docker volume prune
```

---

# Networking

```text
Container

↓

Network Namespace

↓

veth Pair

↓

docker0 Bridge

↓

Host

↓

Internet
```

---

# Network Drivers

| Driver              | Use Case                     |
| ------------------- | ---------------------------- |
| Bridge              | Default                      |
| User-defined Bridge | Production applications      |
| Host                | High performance             |
| None                | No networking                |
| Macvlan             | Physical network integration |
| IPvlan              | Large L2/L3 networks         |
| Overlay             | Multi-host clusters          |

---

# Network Commands

```bash
docker network ls

docker network inspect bridge

docker network create app-net

docker network rm app-net
```

---

# Compose Commands

```bash
docker compose up

docker compose up -d

docker compose down

docker compose stop

docker compose start

docker compose restart

docker compose logs

docker compose logs -f

docker compose ps

docker compose exec app sh
```

---

# Compose Flow

```text
compose.yaml

↓

Networks

↓

Volumes

↓

Images

↓

Containers

↓

Application
```

---

# Security Checklist

* Run as non-root
* Use minimal base image
* Multi-stage builds
* Scan images
* Drop unnecessary capabilities
* Avoid `--privileged`
* Keep secrets outside images
* Apply healthchecks
* Pin image versions

---

# Registry Workflow

```text
docker build

↓

docker tag

↓

docker push

↓

Registry

↓

docker pull

↓

docker run
```

---

# Tag vs Digest

| Tag            | Digest             |
| -------------- | ------------------ |
| Mutable        | Immutable          |
| Easy to read   | Cryptographic hash |
| Human-friendly | Production-safe    |

---

# Troubleshooting Flow

```text
Problem

↓

docker ps

↓

docker logs

↓

docker inspect

↓

docker exec

↓

Network

↓

Volumes

↓

Resources

↓

Root Cause

↓

Fix

↓

Validate
```

---

# Common Errors

| Error                  | First Check                 |
| ---------------------- | --------------------------- |
| Restarting             | `docker logs`               |
| Exited                 | Main process ended          |
| OOMKilled              | `docker stats`              |
| Permission denied      | UID/GID, mounts             |
| No space left          | `docker system df`          |
| Port already allocated | `docker ps`, `ss -tulpn`    |
| DNS failure            | Network, `/etc/resolv.conf` |
| Cannot pull image      | Login, tag, registry        |

---

# Cleanup Commands

```bash
docker image prune

docker container prune

docker volume prune

docker network prune

docker system prune
```

⚠️ Review what will be removed before running cleanup commands on production systems.

---

# Interview Revision

Remember these topics in order:

```text
Docker

↓

Architecture

↓

Images

↓

Dockerfile

↓

Volumes

↓

Networking

↓

Compose

↓

Security

↓

Registries

↓

Troubleshooting
```

---

# Docker Golden Rules

* Containers are disposable.
* Images are immutable.
* Volumes persist data.
* Build once, deploy everywhere.
* Never use `latest` in production.
* Prefer service names over IP addresses.
* Run containers as non-root.
* Scan every production image.
* Read logs before making changes.
* Fix root causes, not symptoms.

---

# One-Minute Revision

```text
Dockerfile
      │
      ▼
Image
      │
      ▼
Registry
      │
      ▼
Pull
      │
      ▼
Container
      │
      ├── Volume
      ├── Network
      ├── Security
      ▼
Compose
      ▼
Production
      ▼
Troubleshooting
```

---

# What's Next?

After mastering Docker, the recommended learning path is:

```text
Docker

↓

Kubernetes

↓

Helm

↓

Ingress

↓

Observability

↓

GitOps

↓

Production Kubernetes
```

Everything you've learned about images, containers, networking, storage, and security will directly help you understand Kubernetes.
