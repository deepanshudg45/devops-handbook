# Docker Architecture

> **Module:** 02 - Docker Architecture
>
> **Difficulty:** Intermediate
>
> **Estimated Reading Time:** 90–120 Minutes
>
> **Prerequisites:** Docker Fundamentals

---

# Table of Contents

1. Learning Objectives
2. Why Docker Architecture Matters
3. High-Level Architecture
4. Docker Components
5. Docker CLI
6. Docker Engine
7. Docker Daemon
8. Docker REST API
9. containerd
10. runc
11. OCI Standards
12. Linux Kernel Features
13. Namespaces
14. cgroups
15. Union File System
16. OverlayFS
17. Complete Request Flow
18. Container Lifecycle
19. Production Architecture
20. Common Misconceptions
21. Best Practices
22. Interview Questions
23. Quick Revision

---

# Learning Objectives

After this chapter you should be able to answer:

* What happens internally when we run `docker run nginx`?
* What is Docker Engine?
* Difference between Docker CLI and Docker Daemon?
* What is containerd?
* What is runc?
* What are Linux Namespaces?
* What are cgroups?
* Why doesn't Docker need a Guest OS?
* What is OverlayFS?
* How does Docker isolate containers?

---

# Why Docker Architecture Matters

Many people know Docker commands.

Very few understand **what actually happens behind the scenes**.

Example:

```bash
docker run nginx
```

Looks like a single command.

Internally, dozens of things happen before Nginx starts.

Understanding this makes debugging much easier.

---

# High-Level Architecture

```text
                 User
                  │
                  ▼
          Docker CLI
                  │
          REST API Request
                  │
                  ▼
          Docker Daemon
                  │
      ┌───────────┼────────────┐
      │           │            │
      ▼           ▼            ▼
 Image Mgmt   Network Mgmt  Volume Mgmt
      │
      ▼
          containerd
                │
                ▼
              runc
                │
                ▼
 Linux Kernel (Namespaces + cgroups)
                │
                ▼
           Running Container
```

---

# Docker Components

Docker consists of multiple components working together.

| Component     | Responsibility                 |
| ------------- | ------------------------------ |
| Docker CLI    | Accepts user commands          |
| Docker Daemon | Executes Docker operations     |
| Docker Engine | Complete Docker platform       |
| Docker API    | Communication layer            |
| containerd    | Container lifecycle management |
| runc          | Creates Linux containers       |
| Linux Kernel  | Isolation and resource control |

---

# Docker CLI

CLI stands for **Command Line Interface**.

Examples:

```bash
docker run nginx

docker ps

docker build .

docker logs app
```

Important:

CLI itself does **not** create containers.

It only sends requests.

Think of it as a remote control.

---

# Mental Model

```text
Remote Control

↓

Television
```

The remote doesn't play the movie.

The television does.

Similarly,

```text
Docker CLI

↓

Docker Daemon
```

---

# Docker Engine

Docker Engine is the complete runtime platform.

It includes:

* Docker Daemon
* REST API
* CLI (client)
* Image management
* Network management
* Volume management

Think of Docker Engine as the "brain" of Docker.

---

# Docker Daemon

Docker Daemon (`dockerd`) is a background service.

It is always running.

Its responsibilities include:

* Build images
* Run containers
* Stop containers
* Remove containers
* Create networks
* Manage volumes
* Pull images
* Push images

---

# Why Do We Need the Daemon?

Suppose Docker CLI directly interacted with Linux.

Then every Docker command would need to know:

* Networking
* Storage
* Image management
* Logging
* Security
* Runtime

Instead,

everything is centralized inside the daemon.

---

# Docker API

Communication between CLI and Daemon happens using the Docker REST API.

Flow:

```text
docker run nginx

↓

CLI

↓

REST API

↓

Docker Daemon
```

The CLI is simply an API client.

This is why GUI tools like Docker Desktop also work—they use the same API.

---

# What is containerd?

Many beginners think Docker directly creates containers.

Not true.

Docker delegates container management to **containerd**.

Responsibilities:

* Pull images
* Store images
* Manage snapshots
* Start containers
* Stop containers
* Handle container lifecycle

---

# What is runc?

containerd still doesn't create Linux processes.

That job belongs to **runc**.

runc is a low-level runtime.

Responsibilities:

* Create namespaces
* Apply cgroups
* Mount filesystem
* Start PID 1
* Launch the container process

Think:

```text
Docker

↓

containerd

↓

runc

↓

Linux Kernel
```

---

# OCI (Open Container Initiative)

Before OCI, every company implemented containers differently.

OCI created standards.

Main standards:

* Runtime Specification
* Image Specification
* Distribution Specification

Because Docker follows OCI standards, the same image can run on Kubernetes, Podman, CRI-O, and other OCI-compliant runtimes.

---

# Linux Kernel Features

Docker is possible because Linux already provides:

* Namespaces
* cgroups
* Capabilities
* seccomp
* OverlayFS
* Netfilter (iptables/nftables)

Docker combines these features into a simple developer experience.

---

# Linux Namespaces

Namespaces provide **isolation**.

Each container gets its own view of system resources.

Common namespaces:

| Namespace | Isolates          |
| --------- | ----------------- |
| PID       | Processes         |
| NET       | Networking        |
| MNT       | Mount points      |
| UTS       | Hostname          |
| IPC       | Shared memory     |
| USER      | Users & Groups    |
| CGROUP    | cgroup visibility |

---

# Example: PID Namespace

Container A:

```text
PID 1

PID 2

PID 3
```

Container B:

```text
PID 1

PID 2

PID 3
```

Both have PID 1.

How?

Because each container has its own PID namespace.

---

# Network Namespace

Every container gets:

* Virtual interface
* IP Address
* Routing table
* Firewall rules

This is why two containers can both listen on port 80 internally.

---

# Mount Namespace

Each container gets its own filesystem view.

Changing files inside one container doesn't automatically affect another.

---

# UTS Namespace

Allows containers to have different hostnames.

Example:

```bash
hostname
```

Container A:

```text
frontend
```

Container B:

```text
backend
```

Same Linux host.

Different hostnames.

---

# cgroups

Namespaces isolate.

cgroups limit resources.

Example:

```text
CPU

Memory

Disk I/O

PIDs

Network
```

Without cgroups,

one container could consume all host resources.

---

# Example

Container A:

```text
Memory Limit

512 MB
```

Container B:

```text
Memory Limit

2 GB
```

Kernel enforces these limits.

---

# OverlayFS

Images contain layers.

OverlayFS combines those layers into one filesystem.

Example:

```text
Layer 1

Ubuntu

↓

Layer 2

Node.js

↓

Layer 3

Application

↓

Merged View
```

Container sees only one filesystem.

Internally,

layers remain separate.

---

# Why Layers?

Benefits:

* Reuse
* Smaller downloads
* Faster builds
* Better cache
* Lower storage usage

---

# Complete docker run Flow

Command:

```bash
docker run nginx
```

Internal Flow:

```text
User

↓

Docker CLI

↓

REST API

↓

Docker Daemon

↓

Check Image

↓

Pull if Missing

↓

containerd

↓

runc

↓

Namespaces

↓

cgroups

↓

OverlayFS Mount

↓

Network Setup

↓

PID 1 Starts

↓

Running Container
```

This entire process happens in a few seconds.

---

# Container Lifecycle

```text
Image

↓

Create

↓

Start

↓

Running

↓

Paused (Optional)

↓

Stopped

↓

Removed
```

Understanding this lifecycle helps during troubleshooting.

---

# Production Architecture

In production:

```text
Developer

↓

Git Push

↓

CI/CD

↓

docker build

↓

Registry

↓

Production Server

↓

Docker Daemon

↓

containerd

↓

runc

↓

Linux Kernel

↓

Running Containers
```

This same architecture is used inside Kubernetes (with OCI-compatible runtimes).

---

# Common Misconceptions

### Myth 1

Docker creates containers.

❌

Docker orchestrates container creation.

runc actually creates the container.

---

### Myth 2

Docker CLI runs containers.

❌

CLI only sends requests.

Daemon performs the work.

---

### Myth 3

Docker has its own kernel.

❌

Containers always use the host Linux kernel.

---

# Best Practices

* Understand architecture before memorizing commands.
* Treat Docker as a client-server application.
* Learn Linux internals—they explain Docker behavior.
* Debug from the daemon outward, not from random guesses.
* Use OCI-compliant images and runtimes.

---

# Mental Models

## Mental Model 1

```text
CLI

↓

Request

↓

Daemon

↓

containerd

↓

runc

↓

Linux

↓

Container
```

---

## Mental Model 2

Docker is like a construction company.

```text
Customer

↓

Reception (CLI)

↓

Project Manager (Daemon)

↓

Site Manager (containerd)

↓

Construction Worker (runc)

↓

Building (Container)
```

Each component has a different responsibility.

---

## Mental Model 3

Remember:

```text
Namespaces

↓

Isolation

cgroups

↓

Limits
```

Isolation and resource control are different concepts.

---

# Quick Revision

* Docker uses a client-server architecture.
* CLI sends API requests.
* Daemon manages Docker operations.
* containerd manages container lifecycle.
* runc creates Linux containers.
* Namespaces provide isolation.
* cgroups enforce resource limits.
* OverlayFS combines image layers.
* Containers share the host kernel.

---

# Interview Questions

### Beginner

* What is Docker Engine?
* What is Docker Daemon?
* Difference between CLI and Daemon?
* What is containerd?

### Intermediate

* Explain the flow of `docker run nginx`.
* Difference between Namespaces and cgroups?
* Why doesn't Docker require a Guest OS?
* What is runc?

### Senior

Design the internal architecture of Docker from the CLI command until the application process starts.

Explain each component's responsibility.

---

# Summary

Docker Architecture is built on a layered design:

```text
User
 ↓
Docker CLI
 ↓
REST API
 ↓
Docker Daemon
 ↓
containerd
 ↓
runc
 ↓
Linux Kernel
 ↓
Container
```

Every Docker feature—images, networking, volumes, Compose, BuildKit, Kubernetes—ultimately depends on this architecture.

Master this flow once, and the rest of Docker becomes much easier to understand.

---

# Cross References

Previous Chapter:

* Docker Fundamentals

Next Chapter:

➡ **03 - Docker Images**

Related Topics:

* Docker Build
* Dockerfile
* OverlayFS
* Linux Kernel
* Kubernetes Container Runtime

