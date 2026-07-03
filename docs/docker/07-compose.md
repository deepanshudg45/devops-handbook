# Docker Compose

> **Module:** 07 - Docker Compose (Part 1)
>
> **Difficulty:** Intermediate
>
> **Estimated Reading Time:** 60–90 Minutes
>
> **Prerequisites:**
>
> * Docker Fundamentals
> * Docker Architecture
> * Docker Images
> * Dockerfile
> * Docker Volumes
> * Docker Networking

---

# Table of Contents

* Learning Objectives
* Why Docker Compose Exists
* Problems Without Compose
* What is Docker Compose?
* Compose Architecture
* compose.yaml
* Services
* Networks
* Volumes
* Environment Variables
* Build vs Image
* Container Dependencies
* Project Lifecycle
* Hands-on Lab
* Interview Questions
* Quick Revision

---

# Learning Objectives

After this chapter, you should be able to answer:

* Why Docker Compose was created
* Difference between `docker run` and Docker Compose
* What is a Service?
* What is `compose.yaml`?
* How are multiple containers managed together?
* How does Compose automatically create networks?
* How does Compose handle volumes?
* What happens internally when running `docker compose up`?

---

# Key Takeaways

Before diving into details, remember these principles:

* Docker Compose is **not** a container runtime.
* Compose is an orchestration tool for **single-host** multi-container applications.
* One Compose file can define:

  * Services
  * Networks
  * Volumes
  * Secrets
  * Configurations
* Compose automatically creates and connects all required resources.

---

# Why Docker Compose Exists

Suppose your application contains:

```text
Frontend

Backend

Redis

PostgreSQL

Nginx
```

Without Compose, you would run:

```bash
docker network create app-network

docker volume create postgres-data

docker run ...

docker run ...

docker run ...

docker run ...

docker run ...
```

Imagine maintaining 20 services like this.

It quickly becomes difficult.

---

# The Problem Without Compose

Problems include:

* Long commands
* Wrong startup order
* Missing networks
* Missing volumes
* Difficult maintenance
* Hard onboarding for new developers

Example:

A new developer joins.

Instead of running one command,

they need to remember 10–15 different Docker commands.

---

# The Solution

Docker Compose lets you describe your entire application in one file.

```text
compose.yaml

↓

docker compose up

↓

Everything Starts
```

Infrastructure becomes **declarative** instead of imperative.

---

# What is Docker Compose?

Docker Compose is a tool that manages **multi-container applications**.

Instead of typing many `docker run` commands,

you describe your application in YAML.

Compose creates:

* Containers
* Networks
* Volumes

and connects them automatically.

---

# Mental Model

Think of a movie director.

Actors alone cannot produce a movie.

Someone coordinates:

* Actors
* Camera
* Lights
* Sound

Docker Compose plays the same role.

Containers are the actors.

Compose coordinates them.

---

# Compose Architecture

```text
Developer

↓

compose.yaml

↓

docker compose up

↓

Docker Engine

↓

Images

↓

Networks

↓

Volumes

↓

Running Containers
```

Compose communicates with Docker Engine using the same Docker API that the Docker CLI uses.

---

# compose.yaml

This is the heart of Docker Compose.

Typical structure:

```yaml
services:

volumes:

networks:

secrets:

configs:
```

Everything about the application lives in this file.

---

# Why YAML?

YAML is:

* Human-readable
* Simple
* Declarative
* Widely used in DevOps

You'll also encounter YAML in:

* Kubernetes
* GitHub Actions
* GitLab CI
* Ansible
* Argo CD

Learning it now helps later.

---

# Basic Compose Example

```yaml
services:

  nginx:

    image: nginx:1.27

    ports:

      - "8080:80"
```

Start:

```bash
docker compose up
```

Stop:

```bash
docker compose down
```

Compose handles the container lifecycle automatically.

---

# Services

A **Service** describes how a container should run.

Example:

```yaml
services:

  backend:

    image: node:22
```

Think:

```text
Service

↓

Container Template
```

Running Compose creates one or more containers from each service definition.

---

# Build vs Image

Compose supports two ways of creating a service.

## Using an Existing Image

```yaml
services:

  nginx:

    image: nginx:1.27
```

Docker downloads the image if necessary.

---

## Building From Source

```yaml
services:

  backend:

    build: .
```

Compose runs:

```bash
docker build
```

before creating the container.

---

## Build Context

Example:

```yaml
build:

  context: .

  dockerfile: Dockerfile
```

Meaning:

* Build context = current directory
* Dockerfile = `./Dockerfile`

You can also specify another Dockerfile:

```yaml
build:

  context: .

  dockerfile: docker/prod.Dockerfile
```

Useful when maintaining separate development and production Dockerfiles.

---

# Volumes in Compose

Example:

```yaml
services:

  postgres:

    image: postgres:17

    volumes:

      - postgres-data:/var/lib/postgresql/data

volumes:

  postgres-data:
```

Compose automatically:

* Creates the volume (if missing)
* Mounts it
* Reuses it on future runs

---

# Networks in Compose

Example:

```yaml
services:

  frontend:

    image: nginx

  backend:

    image: node
```

If you don't define a network,

Compose automatically creates one.

Conceptually:

```text
compose-project_default

↓

Frontend

↓

Backend
```

Containers can communicate using service names.

---

# Automatic DNS

Suppose:

```yaml
services:

  backend:

  postgres:
```

Inside the backend container:

```text
postgres
```

automatically resolves to the PostgreSQL container.

No manual IP configuration is needed.

---

# Environment Variables

Example:

```yaml
services:

  backend:

    environment:

      NODE_ENV: production

      PORT: 3000
```

Inside the container:

```bash
env
```

Output:

```text
NODE_ENV=production

PORT=3000
```

Environment variables configure runtime behavior without rebuilding the image.

---

# depends_on

Example:

```yaml
services:

  backend:

    depends_on:

      - postgres
```

Meaning:

Compose starts the PostgreSQL container before starting the backend container.

**Important:**

`depends_on` controls **startup order**, **not application readiness**.

If PostgreSQL takes 20 seconds to become ready,

the backend may still fail unless you implement:

* Healthchecks
* Retry logic
* Wait-for scripts

This is a very common production interview question.

---

# Project Lifecycle

```text
compose.yaml

↓

docker compose up

↓

Networks Created

↓

Volumes Created

↓

Images Built/Pulled

↓

Containers Started

↓

Application Running
```

Everything is managed as a single project.

---

# Production Pitfall

Many beginners assume:

```yaml
depends_on
```

means

> "Database is ready."

This is incorrect.

It only means:

> "Start this container first."

Always combine dependencies with proper health checks or application retries.

---

# Hands-on Lab

Create a Compose project with:

* Nginx
* PostgreSQL

Requirements:

* PostgreSQL uses a named volume.
* Nginx publishes port `8080`.
* Both services communicate over the default Compose network.

Run:

```bash
docker compose up -d
```

Verify:

* Network created
* Volume created
* Containers running
* Service name resolution works

---

# Interview Questions

## Beginner

* What is Docker Compose?
* Why use Compose instead of multiple `docker run` commands?
* What is a Service?

---

## Intermediate

* Difference between `build:` and `image:`?
* How does Compose create networks?
* What does `depends_on` actually do?

---

## Quick Revision

```text
compose.yaml

↓

Services

↓

Networks

↓

Volumes

↓

docker compose up

↓

Complete Application
```

---

# Golden Rules

> One Compose file should describe the complete application.

> Prefer service names instead of IP addresses.

> Use named volumes for persistent data.

> `depends_on` controls startup order, not readiness.

> Compose is ideal for single-host development and testing environments.

---

> **Part 1 Complete**

In **Part 2** we'll cover:

* Every Compose key in detail
* Ports
* Restart policies
* Healthchecks
* Secrets
* Configs
* Profiles
* Multiple Compose files
* Override files
* Resource limits
* Logging
* Production best practices
* Advanced troubleshooting
* Complete production Compose example

# Docker Compose

> **Module:** 07 - Docker Compose (Part 1)
>
> **Difficulty:** Intermediate
>
> **Estimated Reading Time:** 60–90 Minutes
>
> **Prerequisites:**
>
> * Docker Fundamentals
> * Docker Architecture
> * Docker Images
> * Dockerfile
> * Docker Volumes
> * Docker Networking

---

# Table of Contents

* Learning Objectives
* Why Docker Compose Exists
* Problems Without Compose
* What is Docker Compose?
* Compose Architecture
* compose.yaml
* Services
* Networks
* Volumes
* Environment Variables
* Build vs Image
* Container Dependencies
* Project Lifecycle
* Hands-on Lab
* Interview Questions
* Quick Revision

---

# Learning Objectives

After this chapter, you should be able to answer:

* Why Docker Compose was created
* Difference between `docker run` and Docker Compose
* What is a Service?
* What is `compose.yaml`?
* How are multiple containers managed together?
* How does Compose automatically create networks?
* How does Compose handle volumes?
* What happens internally when running `docker compose up`?

---

# Key Takeaways

Before diving into details, remember these principles:

* Docker Compose is **not** a container runtime.
* Compose is an orchestration tool for **single-host** multi-container applications.
* One Compose file can define:

  * Services
  * Networks
  * Volumes
  * Secrets
  * Configurations
* Compose automatically creates and connects all required resources.

---

# Why Docker Compose Exists

Suppose your application contains:

```text
Frontend

Backend

Redis

PostgreSQL

Nginx
```

Without Compose, you would run:

```bash
docker network create app-network

docker volume create postgres-data

docker run ...

docker run ...

docker run ...

docker run ...

docker run ...
```

Imagine maintaining 20 services like this.

It quickly becomes difficult.

---

# The Problem Without Compose

Problems include:

* Long commands
* Wrong startup order
* Missing networks
* Missing volumes
* Difficult maintenance
* Hard onboarding for new developers

Example:

A new developer joins.

Instead of running one command,

they need to remember 10–15 different Docker commands.

---

# The Solution

Docker Compose lets you describe your entire application in one file.

```text
compose.yaml

↓

docker compose up

↓

Everything Starts
```

Infrastructure becomes **declarative** instead of imperative.

---

# What is Docker Compose?

Docker Compose is a tool that manages **multi-container applications**.

Instead of typing many `docker run` commands,

you describe your application in YAML.

Compose creates:

* Containers
* Networks
* Volumes

and connects them automatically.

---

# Mental Model

Think of a movie director.

Actors alone cannot produce a movie.

Someone coordinates:

* Actors
* Camera
* Lights
* Sound

Docker Compose plays the same role.

Containers are the actors.

Compose coordinates them.

---

# Compose Architecture

```text
Developer

↓

compose.yaml

↓

docker compose up

↓

Docker Engine

↓

Images

↓

Networks

↓

Volumes

↓

Running Containers
```

Compose communicates with Docker Engine using the same Docker API that the Docker CLI uses.

---

# compose.yaml

This is the heart of Docker Compose.

Typical structure:

```yaml
services:

volumes:

networks:

secrets:

configs:
```

Everything about the application lives in this file.

---

# Why YAML?

YAML is:

* Human-readable
* Simple
* Declarative
* Widely used in DevOps

You'll also encounter YAML in:

* Kubernetes
* GitHub Actions
* GitLab CI
* Ansible
* Argo CD

Learning it now helps later.

---

# Basic Compose Example

```yaml
services:

  nginx:

    image: nginx:1.27

    ports:

      - "8080:80"
```

Start:

```bash
docker compose up
```

Stop:

```bash
docker compose down
```

Compose handles the container lifecycle automatically.

---

# Services

A **Service** describes how a container should run.

Example:

```yaml
services:

  backend:

    image: node:22
```

Think:

```text
Service

↓

Container Template
```

Running Compose creates one or more containers from each service definition.

---

# Build vs Image

Compose supports two ways of creating a service.

## Using an Existing Image

```yaml
services:

  nginx:

    image: nginx:1.27
```

Docker downloads the image if necessary.

---

## Building From Source

```yaml
services:

  backend:

    build: .
```

Compose runs:

```bash
docker build
```

before creating the container.

---

## Build Context

Example:

```yaml
build:

  context: .

  dockerfile: Dockerfile
```

Meaning:

* Build context = current directory
* Dockerfile = `./Dockerfile`

You can also specify another Dockerfile:

```yaml
build:

  context: .

  dockerfile: docker/prod.Dockerfile
```

Useful when maintaining separate development and production Dockerfiles.

---

# Volumes in Compose

Example:

```yaml
services:

  postgres:

    image: postgres:17

    volumes:

      - postgres-data:/var/lib/postgresql/data

volumes:

  postgres-data:
```

Compose automatically:

* Creates the volume (if missing)
* Mounts it
* Reuses it on future runs

---

# Networks in Compose

Example:

```yaml
services:

  frontend:

    image: nginx

  backend:

    image: node
```

If you don't define a network,

Compose automatically creates one.

Conceptually:

```text
compose-project_default

↓

Frontend

↓

Backend
```

Containers can communicate using service names.

---

# Automatic DNS

Suppose:

```yaml
services:

  backend:

  postgres:
```

Inside the backend container:

```text
postgres
```

automatically resolves to the PostgreSQL container.

No manual IP configuration is needed.

---

# Environment Variables

Example:

```yaml
services:

  backend:

    environment:

      NODE_ENV: production

      PORT: 3000
```

Inside the container:

```bash
env
```

Output:

```text
NODE_ENV=production

PORT=3000
```

Environment variables configure runtime behavior without rebuilding the image.

---

# depends_on

Example:

```yaml
services:

  backend:

    depends_on:

      - postgres
```

Meaning:

Compose starts the PostgreSQL container before starting the backend container.

**Important:**

`depends_on` controls **startup order**, **not application readiness**.

If PostgreSQL takes 20 seconds to become ready,

the backend may still fail unless you implement:

* Healthchecks
* Retry logic
* Wait-for scripts

This is a very common production interview question.

---

# Project Lifecycle

```text
compose.yaml

↓

docker compose up

↓

Networks Created

↓

Volumes Created

↓

Images Built/Pulled

↓

Containers Started

↓

Application Running
```

Everything is managed as a single project.

---

# Production Pitfall

Many beginners assume:

```yaml
depends_on
```

means

> "Database is ready."

This is incorrect.

It only means:

> "Start this container first."

Always combine dependencies with proper health checks or application retries.

---

# Hands-on Lab

Create a Compose project with:

* Nginx
* PostgreSQL

Requirements:

* PostgreSQL uses a named volume.
* Nginx publishes port `8080`.
* Both services communicate over the default Compose network.

Run:

```bash
docker compose up -d
```

Verify:

* Network created
* Volume created
* Containers running
* Service name resolution works

---

# Interview Questions

## Beginner

* What is Docker Compose?
* Why use Compose instead of multiple `docker run` commands?
* What is a Service?

---

## Intermediate

* Difference between `build:` and `image:`?
* How does Compose create networks?
* What does `depends_on` actually do?

---

## Quick Revision

```text
compose.yaml

↓

Services

↓

Networks

↓

Volumes

↓

docker compose up

↓

Complete Application
```

---

# Golden Rules

> One Compose file should describe the complete application.

> Prefer service names instead of IP addresses.

> Use named volumes for persistent data.

> `depends_on` controls startup order, not readiness.

> Compose is ideal for single-host development and testing environments.

---

> **Part 1 Complete**

In **Part 2** we'll cover:

* Every Compose key in detail
* Ports
* Restart policies
* Healthchecks
* Secrets
* Configs
* Profiles
* Multiple Compose files
* Override files
* Resource limits
* Logging
* Production best practices
* Advanced troubleshooting
* Complete production Compose example

---

# Build Section (Advanced)

Earlier we used:

```yaml
services:
  backend:
    build: .
```

This is the shortest form.

In production, the `build` section is usually more detailed.

Example:

```yaml
services:
  backend:
    build:
      context: .
      dockerfile: docker/prod.Dockerfile
      target: production
      args:
        NODE_VERSION: "22"
```

---

# Understanding Each Field

## context

```yaml
context: .
```

Defines the **Build Context** sent to Docker.

Remember from the Dockerfile chapter:

Everything inside the build context (except files excluded by `.dockerignore`) is sent to the Docker daemon.

---

## dockerfile

```yaml
dockerfile: docker/prod.Dockerfile
```

Compose doesn't require the Dockerfile to be named `Dockerfile`.

Useful examples:

```text
docker/

├── dev.Dockerfile
├── prod.Dockerfile
└── test.Dockerfile
```

---

## target

Suppose your Dockerfile contains:

```dockerfile
FROM node:22 AS builder

...

FROM node:22-alpine AS production
```

Compose:

```yaml
target: production
```

Only builds until the `production` stage.

Very useful with Multi-Stage Builds.

---

## args

Build-time variables.

Example:

```yaml
args:
  NODE_VERSION: "22"
```

Dockerfile:

```dockerfile
ARG NODE_VERSION

FROM node:${NODE_VERSION}
```

Remember:

`ARG`

↓

Build time only

---

# BuildKit with Compose

Compose automatically works with BuildKit when BuildKit is enabled.

Benefits:

* Faster builds
* Parallel execution
* Better caching
* Secret mounts
* SSH forwarding

You don't need to change your Compose file to benefit from BuildKit.

---

# YAML Anchors

Large Compose files often repeat configuration.

Instead of copying:

```yaml
restart: unless-stopped

logging:
  driver: json-file
```

multiple times,

YAML supports anchors.

Example:

```yaml
x-defaults: &defaults
  restart: unless-stopped

services:

  app1:
    <<: *defaults

  app2:
    <<: *defaults
```

Now both services inherit the same configuration.

This is a YAML feature, not a Docker Compose feature.

---

# Extension Fields

Compose ignores keys beginning with:

```text
x-
```

Example:

```yaml
x-common-env:
  environment:
    NODE_ENV: production
```

You can reference these values elsewhere using YAML anchors.

Useful for:

* Shared configuration
* Cleaner files
* Large projects

---

# Project Name

Compose groups resources into a **project**.

If your directory is:

```text
payment-api/
```

Compose creates:

```text
payment-api_backend_1

payment-api_postgres_1

payment-api_default
```

Everything belongs to one project.

---

# Changing the Project Name

Run:

```bash
docker compose -p ecommerce up
```

Now resources become:

```text
ecommerce_backend_1

ecommerce_default
```

Useful when running multiple copies of the same application.

---

# Compose Lifecycle Commands

## Create & Start

```bash
docker compose up
```

Creates resources (if needed) and starts the application.

---

## Detached Mode

```bash
docker compose up -d
```

Runs in the background.

Most common for long-running services.

---

## Stop

```bash
docker compose stop
```

Stops containers.

Networks and volumes remain.

---

## Start Again

```bash
docker compose start
```

Starts previously stopped containers.

No recreation.

---

## Restart

```bash
docker compose restart
```

Restarts services without removing them.

---

## Remove Everything

```bash
docker compose down
```

Removes:

* Containers
* Networks

Keeps named volumes unless explicitly requested.

---

## Remove Volumes Too

```bash
docker compose down -v
```

Removes:

* Containers
* Networks
* Named volumes

⚠️ Be careful.

Database data may be deleted.

---

## View Running Services

```bash
docker compose ps
```

Shows:

* Status
* Ports
* Health
* Container names

---

## Execute Commands

```bash
docker compose exec backend bash
```

Opens a shell inside the running container.

---

## View Logs

Entire project:

```bash
docker compose logs
```

Single service:

```bash
docker compose logs backend
```

Live logs:

```bash
docker compose logs -f
```

---

# Internal Working of docker compose up

When you execute:

```bash
docker compose up
```

Compose performs the following sequence:

```text
Read compose.yaml
        │
        ▼
Validate YAML
        │
        ▼
Create Networks
        │
        ▼
Create Volumes
        │
        ▼
Build Images (if required)
        │
        ▼
Pull Missing Images
        │
        ▼
Create Containers
        │
        ▼
Attach Networks
        │
        ▼
Mount Volumes
        │
        ▼
Start Containers
        │
        ▼
Application Running
```

This entire process is automated.

---

# Real Production Example

Imagine an e-commerce platform.

```text
                Internet
                     │
             Reverse Proxy (Nginx)
                     │
      ┌──────────────┴──────────────┐
      │                             │
Frontend                     Backend API
      │                             │
      ├──────────────┐              │
      ▼              ▼              ▼
   Redis        PostgreSQL      RabbitMQ
```

Compose manages:

* Networks
* Volumes
* Service discovery
* Startup order
* Healthchecks

from one YAML file.

---

# Development vs Production

## Development

Usually includes:

* Bind mounts
* Hot reload
* Debug ports
* Development profiles

---

## Production

Usually includes:

* Immutable images
* Named volumes
* Healthchecks
* Restart policies
* Secrets
* Minimal exposed ports

---

# Compose Limitations

Docker Compose is excellent for:

* Local development
* Testing
* Single-host deployments
* Small production environments

It is **not** a replacement for a full orchestration platform.

Compose does **not** provide features like:

* Automatic scheduling across multiple hosts
* Self-healing clusters
* Automatic scaling across nodes
* Rolling updates across a cluster

Those responsibilities belong to platforms like Kubernetes or Docker Swarm.

---

# Troubleshooting Guide

## Service Can't Reach Database

Check:

* Same network?
* Correct service name?
* Database healthy?
* Correct environment variables?

---

## Image Keeps Rebuilding

Possible causes:

* Dockerfile changed
* Build context changed
* Cache invalidated

---

## Volume Data Missing

Check:

* Named volume exists?
* Mounted correctly?
* Accidentally ran:

```bash
docker compose down -v
```

---

## Service Restarts Continuously

Investigate:

```bash
docker compose logs
```

Possible causes:

* Application crash
* Wrong environment variables
* Database unavailable
* Failed healthcheck

---

# Production Best Practices

* Keep one responsibility per service.
* Use pinned image versions.
* Use named volumes for persistent data.
* Use healthchecks for critical services.
* Use restart policies.
* Prefer service names over IP addresses.
* Separate development and production configurations.
* Store secrets outside the Compose file.

---

# Hands-on Labs

## Lab 1

Create a complete Compose project:

* React frontend
* Node.js backend
* PostgreSQL
* Redis
* Nginx

Requirements:

* Named volumes
* Healthchecks
* Restart policies
* Internal networking

---

## Lab 2

Create:

```text
compose.yaml

compose.dev.yaml

compose.prod.yaml
```

Verify that the second file overrides the first.

---

## Lab 3

Use YAML anchors to avoid duplicate configuration.

---

# Advanced Interview Questions

## Q1

What happens internally when `docker compose up` is executed?

Expected discussion:

* Parse YAML
* Validate
* Create networks
* Create volumes
* Build/Pull images
* Create containers
* Attach networking
* Mount volumes
* Start containers

---

## Q2

Difference between:

```bash
docker compose stop
```

and

```bash
docker compose down
```

Expected discussion:

`stop`

* Stops containers
* Resources remain

`down`

* Removes containers
* Removes networks
* Optionally removes volumes with `-v`

---

## Q3

Why shouldn't production code rely on bind mounts?

Expected discussion:

* Host dependency
* Reduced portability
* Immutable infrastructure principles
* Easier deployments with images

---

## Q4

When would you use multiple Compose files?

Expected discussion:

* Development vs production
* CI environments
* Local overrides
* Different deployment configurations

---

# Master Mental Models

## Mental Model 1

```text
Dockerfile
      │
      ▼
Image
      │
      ▼
Compose
      │
      ▼
Application Stack
```

---

## Mental Model 2

```text
compose.yaml

↓

Services

↓

Containers

↓

Networks

↓

Volumes

↓

Running Application
```

---

## Mental Model 3

```text
docker compose up

↓

Everything Starts

────────────

docker compose down

↓

Everything Stops
```

---

# Complete Compose Checklist

Before committing a Compose file:

* [ ] Pinned image versions
* [ ] Named volumes for persistent data
* [ ] User-defined networking (or Compose default network)
* [ ] Healthchecks configured
* [ ] Restart policies configured
* [ ] Secrets not hardcoded
* [ ] `.env` not committed
* [ ] Development and production configs separated
* [ ] Service names used instead of IP addresses

---

# Chapter Summary

Docker Compose is a declarative tool for managing **multi-container applications on a single host**.

Instead of manually creating:

* Networks
* Volumes
* Containers

Compose describes the entire application in one YAML file and automates the lifecycle.

Compose is ideal for:

* Local development
* Integration testing
* Learning microservices
* Small to medium single-host deployments

It also introduces concepts—services, declarative configuration, health checks, and networking—that you'll see again in Kubernetes.

---

# Cross References

Previous Chapter:

* Docker Networking

Next Chapter:

➡ **08 - Docker Security**

Related Topics:

* Dockerfile
* BuildKit
* Docker Secrets
* Volumes
* Networks
* Kubernetes Pods

---

# Golden Rules

> **One Compose file should describe one application stack.**

> **Build images once; run them through Compose.**

> **Use service names instead of IP addresses.**

> **Separate development and production configurations.**

> **Compose is for single-host orchestration; Kubernetes is for multi-node orchestration.**
