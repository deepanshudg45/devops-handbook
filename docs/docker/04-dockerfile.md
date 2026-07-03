# Dockerfile

> **Module:** 04 - Dockerfile (Part 1)
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

---

# Table of Contents

* Learning Objectives
* Key Takeaways
* What is a Dockerfile?
* Why Dockerfiles Exist
* Build Process Overview
* Docker Build Internals
* Build Context
* Dockerfile Lifecycle
* Dockerfile Syntax
* Dockerfile Instructions Overview
* FROM
* WORKDIR
* COPY
* ADD
* LABEL
* USER
* Production Notes
* Common Mistakes
* Mental Models
* Hands-on Lab
* Quick Revision

---

# Learning Objectives

After completing this chapter, you should be able to answer:

* What is a Dockerfile?
* How does Docker build an image?
* What is Build Context?
* Difference between COPY and ADD?
* Why does every Dockerfile start with FROM?
* Why is WORKDIR preferred over cd?
* Why does COPY create a new image layer?
* How should a production Dockerfile be written?

---

# Key Takeaways

Remember these principles before diving into the details.

* Dockerfile is a **recipe**, not an image.
* `docker build` converts a Dockerfile into an Image.
* Every filesystem-changing instruction usually creates a new layer.
* Docker executes instructions **top to bottom**.
* Docker cache depends heavily on instruction order.
* A good Dockerfile is **small, secure, reproducible, and cache-friendly**.

---

# What is a Dockerfile?

A Dockerfile is a plain text file containing instructions that tell Docker how to build an image.

Think of it as a build recipe.

Example:

```dockerfile
FROM node:22-alpine

WORKDIR /app

COPY package*.json .

RUN npm install

COPY . .

CMD ["npm","start"]
```

Docker reads these instructions one by one and produces a Docker Image.

---

# Mental Model

Never think:

```text
Dockerfile

↓

Runs Application
```

Instead think:

```text
Dockerfile

↓

docker build

↓

Docker Image

↓

docker run

↓

Running Container
```

A Dockerfile **never runs**.

It only **describes how an image should be created**.

---

# Why Dockerfiles Exist

Without Dockerfile:

Developer A manually installs:

* Node.js
* npm
* Dependencies
* Configuration

Developer B repeats the same work.

Production repeats it again.

This creates inconsistency.

Dockerfile solves this by making builds reproducible.

Same Dockerfile

↓

Same Image

↓

Same Environment

---

# Docker Build Process

High-level flow:

```text
Dockerfile
      │
      ▼
Build Context
      │
      ▼
docker build
      │
      ▼
Docker Engine
      │
      ▼
Instruction Execution
      │
      ▼
Layer Creation
      │
      ▼
Image Manifest
      │
      ▼
Docker Image
```

---

# What Happens During docker build?

Suppose we execute:

```bash
docker build -t payment-api:v1 .
```

Internally Docker performs:

1. Reads the Dockerfile.
2. Sends the Build Context to the Docker daemon.
3. Reads instructions from top to bottom.
4. Executes each instruction.
5. Creates layers where required.
6. Creates image metadata.
7. Generates the final image.

---

# Docker Build Internals

Detailed flow:

```text
Docker CLI

↓

Docker Daemon

↓

Read Dockerfile

↓

Load Build Context

↓

Execute FROM

↓

Execute Remaining Instructions

↓

Create Layers

↓

Create Image Config

↓

Generate Manifest

↓

Store Image
```

---

# What is Build Context?

This is one of the most misunderstood Docker concepts.

Command:

```bash
docker build .
```

Question:

What does the `.` mean?

It is **not** "current working directory" in the Dockerfile.

It means:

> "Send this directory and its contents to the Docker daemon as the Build Context."

---

# Build Context Example

Project:

```text
project/

├── Dockerfile
├── package.json
├── package-lock.json
├── src/
├── public/
└── README.md
```

Command:

```bash
docker build .
```

Docker packages almost everything in this directory (except files excluded by `.dockerignore`) and sends it to the daemon.

Only then can `COPY` access those files.

---

# Why .dockerignore Matters

Suppose your project contains:

```text
.git/

node_modules/

coverage/

.env

logs/
```

If these are not ignored,

Docker sends them too.

Problems:

* Larger build context
* Slower builds
* Bigger cache invalidation
* Possible secret leakage

Always maintain a proper `.dockerignore`.

---

# Dockerfile Lifecycle

```text
Write Dockerfile

↓

docker build

↓

Image Created

↓

docker run

↓

Container Running

↓

Modify Dockerfile

↓

Rebuild Image
```

Notice:

Editing a Dockerfile never changes an existing image.

You must rebuild.

---

# Dockerfile Syntax

General format:

```dockerfile
INSTRUCTION arguments
```

Examples:

```dockerfile
FROM ubuntu:24.04

WORKDIR /app

COPY . .

RUN npm install

CMD ["npm","start"]
```

Instructions are case-insensitive, but uppercase is the accepted convention.

---

# Dockerfile Instructions Overview

Common instructions:

| Instruction | Purpose                          |
| ----------- | -------------------------------- |
| FROM        | Base image                       |
| RUN         | Execute command during build     |
| COPY        | Copy files                       |
| ADD         | Copy files + additional features |
| WORKDIR     | Set working directory            |
| ENV         | Set environment variables        |
| ARG         | Build-time variables             |
| EXPOSE      | Document listening port          |
| USER        | Run as specific user             |
| CMD         | Default command                  |
| ENTRYPOINT  | Fixed executable                 |
| LABEL       | Metadata                         |
| VOLUME      | Declare mount point              |

We will study every instruction in depth.

---

# FROM

Every Dockerfile starts with `FROM`.

Example:

```dockerfile
FROM ubuntu:24.04
```

Meaning:

Start this image using Ubuntu 24.04 as the base.

Without `FROM`, Docker has no starting filesystem.

---

## Why FROM Creates the Foundation

Think of it like constructing a building.

You don't start from the third floor.

You first need the foundation.

Similarly,

`FROM`

↓

Base Image

↓

Everything else builds on top.

---

# WORKDIR

Example:

```dockerfile
WORKDIR /app
```

Instead of repeatedly writing:

```dockerfile
RUN cd /app && npm install
```

Docker remembers the working directory for subsequent instructions.

Benefits:

* Cleaner Dockerfiles
* Easier maintenance
* Less repetition

---

# COPY

Example:

```dockerfile
COPY . .
```

Meaning:

Copy files from the Build Context into the image.

Important:

`COPY` copies files **during build**, not during container runtime.

---

# Why COPY Creates a New Layer

COPY changes the filesystem.

Docker records that filesystem change.

Therefore,

a new image layer is created.

This is one reason Docker can cache COPY instructions.

---

# ADD

Example:

```dockerfile
ADD archive.tar.gz /app/
```

ADD can:

* Copy local files
* Automatically extract local tar archives

Historically it also supported remote URLs, but modern best practice is to download remote content explicitly with tools like `curl` or `wget` inside a controlled build step when needed.

---

# COPY vs ADD

Prefer:

```dockerfile
COPY
```

Use:

```dockerfile
ADD
```

Only when you specifically need its additional functionality (such as extracting a local tar archive).

General production recommendation:

> **If COPY works, use COPY.**

---

# LABEL

Labels add metadata.

Example:

```dockerfile
LABEL maintainer="Harsh"

LABEL version="1.0"

LABEL application="payment-api"
```

Useful for:

* Ownership
* Versioning
* Automation
* Image discovery

---

# USER

Example:

```dockerfile
USER node
```

Without USER,

containers usually run as root.

Production recommendation:

Always run applications as a non-root user whenever possible.

This significantly reduces security risk.

---

# Production Pitfall

❌ Bad

```dockerfile
FROM node:latest
```

Why?

Tomorrow `latest` may point to a completely different version.

✅ Better

```dockerfile
FROM node:22.17.0-alpine
```

Pinned versions create reproducible builds.

---

# Common Mistakes

### Mistake 1

Using `latest` everywhere.

---

### Mistake 2

Putting:

```dockerfile
COPY . .
```

before dependency installation.

This destroys build cache efficiency.

---

### Mistake 3

Sending huge Build Contexts.

Always configure `.dockerignore`.

---

### Mistake 4

Running production containers as root.

Avoid whenever possible.

---

# Hands-on Lab

Create this Dockerfile:

```dockerfile
FROM nginx:1.27

COPY index.html /usr/share/nginx/html/

EXPOSE 80

CMD ["nginx","-g","daemon off;"]
```

Tasks:

1. Build the image.
2. Run the container.
3. Modify `index.html`.
4. Rebuild.
5. Observe which layers are reused.

This exercise demonstrates Docker layer caching in practice.

---

# Quick Revision

Remember:

```text
Dockerfile

↓

Instructions

↓

Layers

↓

Image

↓

Container
```

And:

```text
FROM

↓

Foundation

COPY

↓

Filesystem Change

↓

New Layer

WORKDIR

↓

Current Working Directory

USER

↓

Execution Identity
```

---

> **Part 1 Complete**

In the next part we will cover:

* RUN (deep dive)
* ENV
* ARG
* CMD
* ENTRYPOINT
* EXPOSE
* VOLUME
* SHELL
* ONBUILD
* STOPSIGNAL
* HEALTHCHECK
* Build Cache Internals
* Layer Reuse
* Cache Invalidation
* Production Dockerfile Design
* Multi-stage Build Preview
* Advanced Interview Questions

---

# RUN Instruction (Deep Dive)

The `RUN` instruction executes commands **during the image build process**.

Syntax:

```dockerfile
RUN <command>

# or

RUN ["executable","arg1","arg2"]
```

Example:

```dockerfile
FROM ubuntu:24.04

RUN apt-get update

RUN apt-get install -y nginx
```

Every `RUN` instruction executes while Docker is building the image.

After execution, Docker saves the filesystem changes as a **new image layer**.

---

# Mental Model

Think of it like this:

```text
RUN

↓

Execute Command

↓

Filesystem Changes

↓

New Layer

↓

Saved Forever
```

Once a layer is created,

it becomes part of the image.

---

# Why RUN Creates Layers

Example:

```dockerfile
RUN mkdir /app
```

Docker modifies the filesystem.

Since image layers are immutable,

Docker creates a brand-new layer representing this change.

---

# Layer Example

```dockerfile
FROM ubuntu

RUN apt update

RUN apt install -y nginx

RUN mkdir /app
```

Produces:

```text
Layer 1
Ubuntu

↓

Layer 2
apt update

↓

Layer 3
nginx installed

↓

Layer 4
/app created
```

---

# Combining RUN Instructions

Bad

```dockerfile
RUN apt update

RUN apt install -y nginx

RUN apt install -y curl
```

Creates three separate layers.

Better

```dockerfile
RUN apt-get update && \
    apt-get install -y nginx curl && \
    rm -rf /var/lib/apt/lists/*
```

Advantages:

* Fewer layers
* Smaller image
* Cleaner cache
* Better performance

---

# Production Pitfall

Never do:

```dockerfile
RUN apt-get update
```

alone.

Why?

The package index becomes stale if the layer is reused later.

Instead combine update and install in the same instruction.

Correct:

```dockerfile
RUN apt-get update && \
    apt-get install -y nginx
```

---

# ENV Instruction

ENV defines **environment variables inside the image**.

Syntax:

```dockerfile
ENV KEY=value
```

Example:

```dockerfile
ENV NODE_ENV=production

ENV PORT=3000
```

Now every container created from this image inherits these values unless overridden.

---

# ENV vs ARG

One of the most common interview questions.

| ENV                                | ARG                                              |
| ---------------------------------- | ------------------------------------------------ |
| Runtime + Build                    | Build Only                                       |
| Stored in image                    | Not persisted as runtime env by default          |
| Available inside running container | Not available after build unless copied into ENV |

---

# ARG Instruction

ARG defines **build-time variables**.

Example:

```dockerfile
ARG VERSION=22

FROM node:${VERSION}
```

During build:

```bash
docker build --build-arg VERSION=24 .
```

Docker substitutes the value only during the build.

Once the image is created,

the ARG is no longer available unless explicitly copied into an ENV.

---

# ENV + ARG Together

A common production pattern:

```dockerfile
ARG NODE_VERSION=22

FROM node:${NODE_VERSION}

ENV NODE_ENV=production
```

`NODE_VERSION`

↓

Build time.

`NODE_ENV`

↓

Runtime.

---

# EXPOSE

Example:

```dockerfile
EXPOSE 8080
```

Question

Does EXPOSE publish the port?

No.

This is a very common misconception.

EXPOSE is only **documentation**.

It tells people and tools:

> "This application is expected to listen on port 8080."

It does **not** bind the port to the host.

---

# Publishing Ports

Actual publishing happens using:

```bash
docker run -p 8080:8080 image
```

Remember:

```text
EXPOSE

↓

Documentation

-p

↓

Actual Port Mapping
```

---

# CMD

CMD defines the **default command** when the container starts.

Example:

```dockerfile
CMD ["nginx","-g","daemon off;"]
```

When container starts,

Docker executes this command.

---

# Why daemon off?

Normally,

Nginx runs in the background.

Inside containers,

the main process must remain in the foreground.

Otherwise:

```text
PID 1 exits

↓

Container exits
```

Therefore:

```dockerfile
daemon off;
```

keeps Nginx attached to PID 1.

---

# ENTRYPOINT

ENTRYPOINT defines the executable that always runs.

Example:

```dockerfile
ENTRYPOINT ["python3"]
```

Now:

```bash
docker run image app.py
```

actually becomes:

```text
python3 app.py
```

ENTRYPOINT is harder to override than CMD.

---

# CMD vs ENTRYPOINT

Example:

```dockerfile
ENTRYPOINT ["python3"]

CMD ["app.py"]
```

Default execution:

```text
python3 app.py
```

User can override:

```bash
docker run image test.py
```

Result:

```text
python3 test.py
```

ENTRYPOINT stays.

CMD changes.

---

# When to Use CMD

Use CMD when you want a sensible default that users can replace easily.

Examples:

* Development images
* Utility containers
* Test containers

---

# When to Use ENTRYPOINT

Use ENTRYPOINT when the image should always execute a particular application.

Examples:

* Database images
* CLI tools
* Specialized application containers

---

# VOLUME

Example:

```dockerfile
VOLUME ["/data"]
```

This tells Docker:

> "/data is expected to be persistent."

Important:

Creating a VOLUME in the Dockerfile **does not automatically back up your data**.

It simply declares a mount point.

In production,

volumes are usually managed outside the Dockerfile.

---

# SHELL

Default shell:

Linux:

```text
/bin/sh -c
```

You can change it.

Example:

```dockerfile
SHELL ["/bin/bash","-c"]
```

Useful when your build requires Bash-specific features.

---

# HEALTHCHECK

One of the most useful production instructions.

Example:

```dockerfile
HEALTHCHECK CMD curl -f http://localhost:3000/health || exit 1
```

Docker periodically executes this command.

Possible states:

```text
Starting

↓

Healthy

↓

Unhealthy
```

Important:

A running container is **not necessarily a healthy container**.

---

# STOPSIGNAL

Default:

```text
SIGTERM
```

You can customize it.

Example:

```dockerfile
STOPSIGNAL SIGINT
```

Useful for applications that handle different signals differently.

---

# ONBUILD

Used mainly for creating base images.

Example:

```dockerfile
ONBUILD COPY . .
```

This instruction doesn't execute immediately.

It executes when another Dockerfile uses this image as its base.

Not commonly used in modern production workflows.

---

# Dockerfile Instruction Execution Order

Docker executes instructions from top to bottom.

Example:

```dockerfile
FROM ubuntu

WORKDIR /app

COPY package.json .

RUN npm install

COPY . .

CMD ["npm","start"]
```

Changing `package.json`

↓

Invalidates

```text
COPY package.json

↓

RUN npm install

↓

COPY . .

↓

CMD metadata remains
```

Changing only application source code

↓

Reuses

```text
npm install
```

layer.

Huge performance improvement.

---

# Build Cache Invalidation

Docker cache works sequentially.

Once one layer changes,

every following filesystem-changing layer is rebuilt.

Think:

```text
Layer 1 ✅

↓

Layer 2 ✅

↓

Layer 3 ❌ Changed

↓

Layer 4 Rebuilt

↓

Layer 5 Rebuilt
```

This is why instruction ordering matters so much.

---

# Production Dockerfile Principles

A good Dockerfile should be:

* Small
* Secure
* Deterministic
* Cache-friendly
* Readable
* Reproducible
* Fast to build

---

# Production Checklist

Before merging a Dockerfile:

* [ ] Pinned base image version
* [ ] Multi-stage build where applicable
* [ ] Non-root user
* [ ] `.dockerignore` configured
* [ ] Secrets not baked into image
* [ ] HEALTHCHECK defined
* [ ] Minimal packages installed
* [ ] Cache optimized
* [ ] ENTRYPOINT/CMD reviewed

---

# Interview Questions

### Beginner

* Difference between RUN and CMD?
* Difference between COPY and ADD?
* What does EXPOSE do?

### Intermediate

* Difference between CMD and ENTRYPOINT?
* Difference between ENV and ARG?
* Why does COPY create a new layer?

### Senior

A Docker build takes 20 minutes.

How would you optimize the Dockerfile?

Expected discussion:

* Dockerfile ordering
* Layer caching
* Multi-stage builds
* BuildKit
* `.dockerignore`
* Package caching
* Smaller base images

---

# Quick Revision

```text
RUN
↓

Build Time

CMD
↓

Default Runtime Command

ENTRYPOINT
↓

Fixed Executable

ARG
↓

Build Time Variable

ENV
↓

Runtime Variable

EXPOSE
↓

Documentation

-p
↓

Port Publishing

HEALTHCHECK
↓

Application Status
```

---

> **Part 2 Complete**

In the next and final part of `04-dockerfile.md`, we'll cover:

* Multi-stage builds (deep dive)
* Distroless images
* Dockerfile security
* BuildKit advanced features
* Secrets during build
* SSH mounts
* Real production Dockerfiles (Node.js, Java, Python, Go)
* Dockerfile anti-patterns
* Optimization case studies
* Complete troubleshooting guide
* Master revision sheet
* Chapter summary

---

# Multi-Stage Builds (Deep Dive)

One of the biggest improvements in modern Dockerfiles is the use of **Multi-Stage Builds**.

Before understanding Multi-Stage Builds, let's first understand the problem they solve.

---

# The Problem

Suppose you're building a Go application.

To compile it, you need:

* Go Compiler
* Build Tools
* Dependencies

But after compilation, the application only needs the final binary.

Question:

Should the production image still contain:

* Go Compiler?
* Git?
* Build Tools?

No.

They're unnecessary and increase:

* Image size
* Attack surface
* Vulnerability count

---

# Traditional Dockerfile

```dockerfile
FROM golang:1.24

WORKDIR /app

COPY . .

RUN go build -o app

CMD ["./app"]
```

Final image contains:

* Go compiler
* Source code
* Cache
* Build dependencies
* Final binary

Very large.

---

# Multi-Stage Solution

```dockerfile
FROM golang:1.24 AS builder

WORKDIR /app

COPY . .

RUN go build -o app

FROM alpine:3.22

WORKDIR /app

COPY --from=builder /app/app .

CMD ["./app"]
```

Now the final image contains only:

* Alpine Linux
* Compiled binary

Nothing else.

---

# Mental Model

Think of a factory.

Stage 1

```text
Raw Materials

↓

Machine

↓

Finished Product
```

Stage 2

```text
Finished Product

↓

Shipping Box
```

Customers only receive the shipping box.

Not the factory.

Exactly the same happens in Multi-Stage Builds.

---

# Internal Working

```text
Stage 1

Source Code

↓

Compile

↓

Binary

──────────────

Stage 2

↓

Copy Binary

↓

Production Image
```

Docker completely discards Stage 1 unless explicitly retained.

---

# Why Multi-Stage Builds Matter

Benefits:

* Smaller images
* Faster deployments
* Lower registry storage
* Reduced attack surface
* Cleaner production images

This is the recommended approach for most compiled languages.

---

# Real Example (Node.js)

Bad Dockerfile

```dockerfile
FROM node:22

WORKDIR /app

COPY . .

RUN npm install

RUN npm run build

CMD ["npm","start"]
```

Problems:

* Development dependencies remain
* Build cache remains
* Source files remain
* Large image

---

Better

```dockerfile
FROM node:22 AS builder

WORKDIR /app

COPY package*.json .

RUN npm ci

COPY . .

RUN npm run build

FROM nginx:1.27-alpine

COPY --from=builder /app/dist /usr/share/nginx/html
```

Production image now contains only static website files.

No Node.js runtime required.

---

# Stage Naming

Example

```dockerfile
FROM node:22 AS builder

...

FROM alpine AS runtime
```

Meaning:

```text
builder

↓

runtime
```

Using names is better than numeric stage references because it's easier to read and maintain.

---

# COPY --from

Example

```dockerfile
COPY --from=builder /app/app .
```

Meaning:

Copy

```text
/app/app
```

from

```text
builder stage
```

into

current stage.

Nothing else is copied.

---

# Distroless Images

Modern production often uses **Distroless** images.

Question:

What is Distroless?

A minimal runtime image containing only what the application needs.

No:

* Shell
* Package Manager
* apt
* yum
* bash

Benefits:

* Very small
* More secure
* Fewer vulnerabilities

Example:

```dockerfile
FROM gcr.io/distroless/static
```

---

# Alpine vs Distroless

| Alpine                    | Distroless         |
| ------------------------- | ------------------ |
| Has shell                 | No shell           |
| Package manager available | No package manager |
| Good for debugging        | Production focused |
| Slightly larger           | Usually smaller    |

Rule:

Use Alpine if debugging inside the container is important.

Use Distroless for hardened production images when debugging through the container shell isn't required.

---

# Dockerfile Security

A Dockerfile is part of your application's security.

Poor Dockerfiles create insecure containers.

Good Dockerfiles reduce risk before deployment even starts.

---

# Security Best Practices

## Use Official Images

Prefer:

```dockerfile
FROM node:22-alpine
```

Avoid random community images unless they are trusted and maintained.

---

## Pin Versions

Bad

```dockerfile
FROM python:latest
```

Good

```dockerfile
FROM python:3.13.5-alpine
```

Pinned versions create reproducible builds.

---

## Never Run as Root

Bad

```dockerfile
USER root
```

Better

```dockerfile
RUN adduser -D appuser

USER appuser
```

If the application is compromised,

damage is limited.

---

## Don't Install Unnecessary Packages

Bad

```dockerfile
RUN apt install \
vim \
nano \
curl \
gcc \
make \
python3
```

Question:

Does production need all of these?

Usually not.

Every package increases:

* Image size
* Vulnerability count
* Maintenance effort

Install only what is required.

---

# Secrets During Build

Never write:

```dockerfile
ENV DB_PASSWORD=mysecret
```

or

```dockerfile
COPY .env .
```

These secrets become part of the image or build context.

Instead use BuildKit secrets.

Concept:

```text
Secret

↓

Temporary Build Mount

↓

Build Finished

↓

Secret Removed
```

The final image never contains the secret.

---

# SSH Mount

Sometimes builds need private Git repositories.

Example:

```text
Private Git Repository

↓

Build

↓

Application
```

Instead of copying SSH keys into the image,

BuildKit temporarily forwards the SSH agent.

Benefits:

* Keys never become part of image layers
* Better security
* Easier CI/CD integration

---

# Dockerfile Anti-Patterns

## Anti-Pattern 1

```dockerfile
COPY . .
```

before

```dockerfile
npm install
```

Every source code change invalidates dependency cache.

---

## Anti-Pattern 2

Installing editors:

```dockerfile
vim

nano

htop
```

Production containers rarely need them.

---

## Anti-Pattern 3

Using latest everywhere.

Bad for reproducibility.

---

## Anti-Pattern 4

Putting secrets inside Dockerfile.

Never.

---

## Anti-Pattern 5

One giant RUN instruction with unrelated operations.

Combine related commands, but don't sacrifice readability for the sake of having fewer layers.

---

# Real Production Dockerfile (Node.js)

```dockerfile
FROM node:22-alpine AS builder

WORKDIR /app

COPY package*.json ./

RUN npm ci

COPY . .

RUN npm run build

FROM node:22-alpine

WORKDIR /app

COPY --from=builder /app/dist ./dist
COPY package*.json ./

RUN npm ci --omit=dev

USER node

EXPOSE 3000

HEALTHCHECK CMD wget --spider http://localhost:3000/health || exit 1

CMD ["node","dist/index.js"]
```

Features:

* Multi-stage build
* Production dependencies only
* Non-root user
* Healthcheck
* Smaller runtime image

---

# Dockerfile Troubleshooting

## Build Always Rebuilds

Possible causes:

* Wrong instruction order
* Build context changes
* Cache disabled

---

## Image Too Large

Check:

```bash
docker history image-name
```

Investigate large layers.

---

## Secrets Found in Image

Check:

```bash
docker history image-name
```

and inspect Dockerfile for:

* ENV
* COPY
* ARG misuse

---

## Application Exits Immediately

Check:

* CMD
* ENTRYPOINT
* Main process
* Foreground execution

Example:

Nginx needs:

```dockerfile
CMD ["nginx","-g","daemon off;"]
```

Otherwise PID 1 exits and the container stops.

---

# Production Dockerfile Checklist

Before publishing an image:

* [ ] Base image pinned
* [ ] Multi-stage build used
* [ ] Build cache optimized
* [ ] `.dockerignore` configured
* [ ] Secrets excluded
* [ ] Non-root user configured
* [ ] Healthcheck added
* [ ] Minimal packages installed
* [ ] Production dependencies only
* [ ] Image scanned
* [ ] Tags versioned

---

# Interview Questions

## Beginner

* What is a Dockerfile?
* Difference between RUN and CMD?
* Why use WORKDIR?

---

## Intermediate

* Difference between CMD and ENTRYPOINT?
* Difference between COPY and ADD?
* Why does Docker cache layers?

---

## Senior

### Q1

How would you optimize a 2 GB Docker image?

Expected discussion:

* Multi-stage builds
* Smaller base image
* Remove unnecessary packages
* `.dockerignore`
* Layer ordering
* BuildKit

---

### Q2

How would you securely use a private Git repository during image build?

Expected discussion:

* BuildKit SSH mounts
* Avoid copying SSH keys
* Temporary credential forwarding

---

### Q3

A Docker build takes 25 minutes in CI.

How would you improve it?

Expected discussion:

* Dockerfile ordering
* Layer cache
* BuildKit
* Cache mounts
* Multi-stage builds
* Smaller build context

---

# Master Mental Models

## Mental Model 1

```text
Dockerfile

↓

Recipe

↓

Image

↓

Container
```

---

## Mental Model 2

```text
FROM

↓

Foundation

RUN

↓

Modify Filesystem

COPY

↓

Application Files

CMD

↓

Default Command
```

---

## Mental Model 3

```text
Stage 1

↓

Compile

↓

Stage 2

↓

Runtime Image
```

Only the runtime image is deployed.

---

# Chapter Summary

A Dockerfile is not just a list of commands.

It is an **Infrastructure as Code** document that defines:

* How an image is built
* What software is installed
* How the application starts
* Which user runs the application
* How the application is monitored
* How efficiently Docker can cache the build

A well-written Dockerfile results in:

* Faster builds
* Smaller images
* Better security
* Easier maintenance
* Predictable deployments

Poor Dockerfiles lead to:

* Slow CI pipelines
* Large images
* Security vulnerabilities
* Cache misses
* Difficult troubleshooting

---

# Cross References

Previous Chapters:

* Docker Fundamentals
* Docker Architecture
* Docker Images

Next Chapter:

➡ **05 - Docker Volumes**

Related Topics:

* BuildKit
* OverlayFS
* Docker Registry
* Docker Security
* Multi-Stage Builds

---

# Golden Rules

> **Build once, deploy everywhere.**

> **Use Multi-Stage Builds for production.**

> **Never store secrets in Dockerfiles or images.**

> **Run containers as non-root whenever possible.**

> **Optimize Dockerfile order for maximum cache reuse.**

> **A Dockerfile is Infrastructure as Code—treat it with the same quality standards as application code.**
