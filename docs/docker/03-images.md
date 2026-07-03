# Docker Images

> **Module:** 03 - Docker Images (Part 1)
>
> **Difficulty:** Intermediate
>
> **Estimated Reading Time:** 30–40 Minutes
>
> **Prerequisites:**
>
> * Docker Fundamentals
> * Docker Architecture

---

# Table of Contents

* Learning Objectives
* Key Takeaways
* What is a Docker Image?
* Why Images Exist
* Mental Model
* Image vs Container
* Image Internals
* Image Layers
* Union Filesystem Overview
* Image Lifecycle
* Image Naming
* Tags
* Digests
* Production Notes

---

# Learning Objectives

After completing this chapter you should be able to explain:

* What is a Docker Image?
* Why Docker Images are read-only
* Difference between Image and Container
* What are image layers?
* Why layers improve performance
* How Docker stores images
* Why multiple images share storage
* Difference between Tag and Digest

---

# Key Takeaways

Before reading the details, remember these core ideas:

* An Image is a **read-only blueprint**.
* A Container is a **running instance** of an Image.
* Images are made of **multiple immutable layers**.
* Layers are **shared** across images whenever possible.
* Containers never modify image layers.
* Every running container gets its own **Writable Layer**.

These six concepts explain almost everything about Docker Images.

---

# What is a Docker Image?

A Docker Image is a **read-only template** used to create containers.

Think of it as a packaged application that contains:

* Application code
* Runtime
* Libraries
* Dependencies
* Configuration
* Metadata

An image itself does **not** execute.

Only a **Container** executes.

---

# Mental Model

Instead of memorizing definitions, think like this:

```text
Dockerfile
      │
      ▼
Docker Image
      │
      ▼
Container
      │
      ▼
Running Application
```

The Dockerfile describes **how to build**.

The Image is **what gets built**.

The Container is **what actually runs**.

---

# Real Life Analogy

Think of a Docker Image like a class in Object-Oriented Programming.

```text
Class
    ↓
Object
```

Similarly,

```text
Image
    ↓
Container
```

One class can create many objects.

One image can create many containers.

---

# Image vs Container

| Image          | Container                |
| -------------- | ------------------------ |
| Blueprint      | Running instance         |
| Read-only      | Read + Write             |
| Immutable      | Mutable during runtime   |
| Cannot execute | Executes processes       |
| Stored on disk | Running in memory + disk |

Remember:

> **Image = Blueprint**
>
> **Container = Running Process**

---

# Why Are Images Read-Only?

This is one of the most important Docker concepts.

Imagine if images were writable.

Container A modifies:

```text
/usr/bin/node
```

Container B uses the same image.

Suddenly,

Container B also changes.

This would completely break consistency.

Therefore,

Docker makes image layers **immutable**.

Every container receives its own Writable Layer instead.

---

# Internal Structure of an Image

An image is **not one big file**.

It is a stack of layers.

Example:

```text
Application Layer
──────────────────

Node.js Layer
──────────────────

Ubuntu Layer
──────────────────
```

Docker merges these layers into one logical filesystem using a Union Filesystem (covered later in more depth).

---

# Example

Suppose this Dockerfile:

```dockerfile
FROM ubuntu:24.04

RUN apt update

RUN apt install -y nodejs

COPY . /app
```

Conceptually, Docker creates layers like:

```text
Layer 4
Application Files

──────────────

Layer 3
Node.js Installed

──────────────

Layer 2
apt update

──────────────

Layer 1
Ubuntu Base
```

Each instruction (that changes the filesystem) usually creates a new layer.

---

# Why Layers Exist

Layers solve multiple problems at once.

## 1. Storage Efficiency

Suppose you have:

```text
Image A

Ubuntu
Node
App A
```

and

```text
Image B

Ubuntu
Node
App B
```

Without layers:

```text
Ubuntu

Stored Twice
```

With layers:

```text
Ubuntu

Stored Once

Shared by both images
```

Huge storage savings.

---

## 2. Faster Downloads

When pulling an image:

Docker checks layer digests.

If Layer A already exists locally:

```text
Skip Download
```

Only missing layers are downloaded.

---

## 3. Faster Builds

If nothing changed:

Docker reuses cached layers.

Instead of rebuilding:

```text
Ubuntu

Node

npm install
```

Docker simply reuses them.

Only changed layers are rebuilt.

---

# Image Lifecycle

The typical lifecycle is:

```text
Dockerfile
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
Local Image
      │
      ▼
docker run
      │
      ▼
Container
```

This workflow is repeated in almost every CI/CD pipeline.

---

# Image Naming Convention

A complete image name has four parts:

```text
registry/namespace/repository:tag
```

Example:

```text
docker.io/library/nginx:1.27
```

Breakdown:

| Component | Meaning    |
| --------- | ---------- |
| docker.io | Registry   |
| library   | Namespace  |
| nginx     | Repository |
| 1.27      | Tag        |

---

# Tags

A tag is simply a **human-friendly label**.

Examples:

```text
1.0.0

1.2.3

dev

staging

latest
```

Important:

A tag is **mutable**.

Today:

```text
latest
    ↓
Image A
```

Tomorrow:

```text
latest
    ↓
Image B
```

The tag moved.

---

# Digests

Every image also has a digest.

Example:

```text
sha256:7dfc...
```

A digest is generated from the image contents.

Properties:

* Immutable
* Unique
* Cryptographically verifiable

Unlike tags,

a digest **never changes**.

---

# Tag vs Digest

| Tag                           | Digest                                 |
| ----------------------------- | -------------------------------------- |
| Human readable                | Machine generated                      |
| Mutable                       | Immutable                              |
| Easy to remember              | Long hash                              |
| Can point to different images | Always identifies the exact same image |

Production deployments often prefer digests because they guarantee the exact image version.

---

# Production Notes

## Never Deploy Using Only `latest`

Bad:

```text
payment-api:latest
```

Better:

```text
payment-api:2.3.1
```

Best:

```text
payment-api@sha256:...
```

This prevents unexpected deployments when someone updates the `latest` tag.

---

# Mental Model

Remember this relationship:

```text
Dockerfile
        │
        ▼
Image (Read Only)
        │
        ▼
Container (Writable)
```

Everything else in Docker builds upon this model.

---

> **Part 1 Complete**

The next part will cover:

* Copy-on-Write
* Writable Layer
* OverlayFS internals
* Image IDs
* Content-addressable storage
* Image caching
* Multi-stage image concepts
* Image optimization
* Security considerations
* Troubleshooting
* Production case studies

---

# Copy-on-Write (CoW)

One of Docker's biggest optimizations is **Copy-on-Write (CoW)**.

Many beginners memorize the term but don't understand what actually happens.

Let's understand it properly.

---

# Why Copy-on-Write Exists

Suppose you have an image:

```text
Ubuntu
```

Now create 100 containers from it.

Question:

Will Docker make 100 copies of Ubuntu?

No.

That would waste huge disk space.

Instead, Docker allows all containers to **share the same read-only image layers**.

Only when a container modifies a file does Docker create a private copy for that container.

This is called **Copy-on-Write**.

---

# Mental Model

Imagine a library.

There is one original book.

100 students are reading it.

As long as nobody writes in the book,

everyone shares the same copy.

Now one student highlights a paragraph.

Instead of modifying the original,

the library gives that student a personal photocopy.

Exactly the same idea.

---

# Internal Working

Suppose Image contains

```text
/etc/nginx/nginx.conf
```

Container starts.

Initially

```text
Image Layer
        │
        ▼
Container
```

Container reads the file.

No copy is made.

---

Now container modifies

```text
/ etc/nginx/nginx.conf
```

Docker performs:

```text
Read-only Layer

↓

Copy File

↓

Writable Layer

↓

Modify Copy
```

Original image remains untouched.

---

# Visual Flow

```text
                 Image Layer
                     │
          Read Only (Shared)
                     │
      ┌──────────────┴──────────────┐
      ▼                             ▼
 Container A                   Container B
      │                             │
Writable Layer                 Writable Layer
```

Every container has its own writable layer.

---

# Important Rule

Image layers

↓

Never change.

Container writable layer

↓

Changes during runtime.

---

# Example

Container A

Creates

```text
/tmp/test.txt
```

Question

Can Container B see this file?

Answer

No.

Because it exists only inside Container A's writable layer.

---

# Writable Layer

Every running container gets exactly one writable layer.

Structure:

```text
Application Image

↓

Read Only Layers

↓

Writable Layer

↓

Running Container
```

This writable layer stores:

* New files
* Modified files
* Deleted file metadata
* Temporary runtime data

---

# Why Does Data Disappear?

Suppose

```bash
docker run ubuntu
```

Inside container:

```bash
echo "Hello" > /tmp/test.txt
```

Exit.

Delete container.

Run again.

Question

Where is the file?

Gone.

Why?

Because the writable layer was deleted.

Remember:

```text
Container Deleted

↓

Writable Layer Deleted

↓

Runtime Data Lost
```

This is why Docker volumes exist.

---

# Image IDs

Every image has a unique ID.

Example:

```text
sha256:6f1c...
```

Docker doesn't identify images by names internally.

It identifies them using SHA256 hashes.

Names are only for humans.

---

# Why SHA256?

Advantages:

* Unique
* Immutable
* Fast lookup
* Detect corruption
* Verify integrity

Even changing one byte changes the hash.

---

# Content-Addressable Storage

Docker stores layers based on their content.

Suppose two images contain exactly the same Ubuntu layer.

Docker stores it only once.

```text
Image A

↓

Ubuntu Layer

Image B

↓

Ubuntu Layer

↓

Stored Once
```

Huge storage optimization.

---

# Layer Cache

Docker caches every successfully built layer.

Suppose Dockerfile:

```dockerfile
FROM ubuntu:24.04

RUN apt update

RUN apt install -y nodejs

COPY . .

RUN npm install
```

First build:

Everything executes.

Second build:

Nothing changed.

Docker outputs:

```text
Using cache
```

Almost instant.

---

# Cache Invalidation

Now modify only:

```text
index.html
```

Question

Which layers rebuild?

Docker works from top to bottom.

Example:

```dockerfile
FROM ubuntu

RUN apt update

RUN apt install nodejs

COPY . .

RUN npm install
```

If only application code changes,

Docker can reuse:

```text
FROM

RUN apt update

RUN apt install nodejs
```

But it must rebuild:

```text
COPY . .

RUN npm install
```

Because every layer depends on the previous layer.

This is why Dockerfile instruction order is very important.

---

# Good Dockerfile Ordering

Instead of

```dockerfile
COPY . .

RUN npm install
```

Use

```dockerfile
COPY package*.json .

RUN npm install

COPY . .
```

Benefits:

If only source code changes,

Docker reuses the expensive

```text
npm install
```

layer.

Huge build speed improvement.

---

# Image Storage

By default, Docker stores images inside its data directory.

Typically on Linux:

```text
/var/lib/docker
```

Inside this directory Docker manages:

* Image layers
* Metadata
* Build cache
* Snapshots

**Never manually modify this directory.**

Always use Docker commands.

---

# Image History

Every image remembers how it was built.

Command:

```bash
docker history nginx
```

Output contains:

* Layer size
* Instruction
* Creation time

Useful for:

* Optimization
* Security review
* Troubleshooting

---

# Dangling Images

Sometimes builds leave images like:

```text
<none>    <none>
```

These are called dangling images.

Usually created during rebuilds.

Cleanup:

```bash
docker image prune
```

---

# Production Best Practices

## Use Small Base Images

Instead of:

```text
ubuntu
```

Sometimes use:

```text
alpine
```

or

```text
distroless
```

when appropriate.

Smaller images:

* Faster pull
* Faster deployment
* Smaller attack surface

---

## Pin Versions

Avoid:

```dockerfile
FROM node:latest
```

Prefer:

```dockerfile
FROM node:22.17.0-alpine
```

Predictable builds.

---

## Never Store Secrets

Never do:

```dockerfile
ENV PASSWORD=admin123
```

Secrets become part of image metadata.

Use:

* Docker Secrets
* Kubernetes Secrets
* AWS Secrets Manager
* HashiCorp Vault

---

## Scan Images

Every production image should be scanned for vulnerabilities.

Popular tools:

* Docker Scout
* Trivy
* Grype
* AWS ECR Scanning

---

# Common Beginner Mistakes

## Mistake 1

Thinking images are virtual machines.

Wrong.

Images are layered filesystems.

---

## Mistake 2

Thinking image changes while container runs.

Wrong.

Only writable layer changes.

---

## Mistake 3

Using

```dockerfile
FROM latest
```

Bad practice.

---

## Mistake 4

Putting

```dockerfile
COPY . .
```

at the beginning of every Dockerfile.

Destroys build cache efficiency.

---

# Troubleshooting

## Image Pulls Very Slowly

Possible reasons:

* Large image size
* Slow network
* No layer cache
* Registry latency

---

## Image Size Suddenly Increased

Run:

```bash
docker history image-name
```

Find the layer responsible.

Often caused by:

* Installing unnecessary packages
* Large build artifacts
* Cache files

---

## Build Always Rebuilds Everything

Check Dockerfile instruction order.

Poor ordering causes cache invalidation.

---

# Interview Questions

### Beginner

* What is a Docker Image?
* Difference between Image and Container?
* Why are images read-only?

### Intermediate

* Explain Copy-on-Write.
* What is the writable layer?
* Why are Docker images layered?

### Senior

A Docker image suddenly grew from 250 MB to 1.8 GB.

How would you investigate and optimize it?

Expected discussion:

* `docker history`
* Layer analysis
* Multi-stage builds
* Removing unnecessary packages
* Better Dockerfile ordering
* Smaller base images

---

# Key Takeaways

* Images are immutable.
* Containers receive one writable layer.
* Copy-on-Write saves storage and memory.
* Layers enable caching and fast downloads.
* Docker stores identical layers only once.
* Dockerfile order directly affects build performance.
* Production images should be versioned, scanned, and kept as small as practical.

---

> **Part 2 Complete**

The next part will cover:

* OverlayFS deep dive
* Image manifests
* OCI Image Specification
* Multi-stage build internals
* Build cache internals
* Image optimization strategies
* Registry relationship
* Production case studies
* Advanced interview questions

---

# OverlayFS Deep Dive

Earlier we learned that Docker Images are made of multiple layers.

The next question is:

> **How does Docker combine all those layers into one filesystem?**

The answer is **OverlayFS**.

OverlayFS is a Linux Union Filesystem.

Its job is to merge multiple directories into one logical filesystem.

Docker doesn't invent OverlayFS.

It uses the Linux kernel implementation.

---

# Mental Model

Imagine you have three transparent sheets.

Sheet 1

```text
Ubuntu Files
```

Sheet 2

```text
Node.js Files
```

Sheet 3

```text
Application Files
```

When you stack them,

they appear as **one single filesystem**.

OverlayFS does exactly that.

---

# OverlayFS Structure

```text
Application Layer
─────────────────────
Node.js Layer
─────────────────────
Ubuntu Layer
─────────────────────
Writable Layer
─────────────────────
Merged View
```

The application only sees

```text
/
```

It never knows multiple layers exist.

---

# OverlayFS Terminology

OverlayFS mainly works with four directories.

| Directory | Purpose                                   |
| --------- | ----------------------------------------- |
| lowerdir  | Read-only image layers                    |
| upperdir  | Writable layer                            |
| workdir   | Internal OverlayFS workspace              |
| merged    | Final filesystem visible inside container |

Visual

```text
Lower Layers
      │
      ▼

Upper Layer

      │

Work Directory

      │

Merged Filesystem

      │

Container
```

---

# File Read Flow

Suppose application reads

```text
/etc/nginx/nginx.conf
```

OverlayFS checks:

```text
Upper Layer

↓

Exists?

↓

YES

↓

Return Upper Version

↓

NO

↓

Read Lower Layer
```

This lookup is completely transparent.

---

# File Write Flow

Suppose file exists only in image.

Container modifies it.

OverlayFS performs:

```text
Read Lower Layer

↓

Copy File

↓

Upper Layer

↓

Modify Copy

↓

Return Updated File
```

Original image layer never changes.

---

# File Delete Flow

Question

If image layer is read-only,

how can Docker delete a file?

Answer

Docker doesn't delete it.

Instead it creates a special marker called a **Whiteout File**.

Think of it like:

```text
Original File

↓

Hidden

↓

Appears Deleted
```

Actual image layer still contains the file.

---

# Why OverlayFS is Fast

Advantages:

* No full filesystem copy
* Shared layers
* Fast startup
* Less disk usage
* Efficient caching

Without OverlayFS,

every container would require a complete filesystem copy.

---

# OCI Image Specification

Earlier we studied OCI Runtime.

Now let's study OCI Images.

OCI defines how an image should be structured.

Components include:

* Manifest
* Config
* Layers
* Metadata

Because Docker follows OCI,

the same image works on:

* Docker
* containerd
* Podman
* CRI-O
* Kubernetes

---

# Docker Image Manifest

When you run

```bash
docker pull nginx
```

Docker does **not** receive an image immediately.

First,

it downloads the manifest.

Think of the manifest as an index.

Example:

```json
{
  "config":"sha256:abc",
  "layers":[
      "sha256:111",
      "sha256:222",
      "sha256:333"
  ]
}
```

The manifest tells Docker:

* Which config file to use
* Which layers belong to this image
* Layer order

---

# Image Config

Besides layers,

Docker downloads an Image Config.

It stores metadata like:

* Environment variables
* Entrypoint
* CMD
* Working Directory
* Labels
* User
* Exposed Ports

This metadata is part of the image.

---

# Image Pull Flow (Internal)

When executing

```bash
docker pull nginx:1.27
```

Internal flow:

```text
CLI

↓

Daemon

↓

Registry

↓

Download Manifest

↓

Download Config

↓

Download Missing Layers

↓

Verify SHA256

↓

Store Locally

↓

Image Ready
```

Notice:

Layers are downloaded **after** reading the manifest.

---

# Image Push Flow

When pushing

```bash
docker push myapp:v1
```

Docker performs:

```text
CLI

↓

Daemon

↓

Registry

↓

Authenticate

↓

Compare Layer Digests

↓

Upload Missing Layers

↓

Upload Manifest

↓

Push Complete
```

If a layer already exists,

Docker skips uploading it.

---

# Multi-Stage Build (Concept)

Large images often contain:

* Build tools
* Compilers
* Temporary files

Production containers usually don't need them.

Example

Stage 1

```text
Compile Application
```

Stage 2

```text
Copy Final Binary
```

Only the final binary reaches production.

Benefits:

* Smaller image
* Better security
* Faster deployment

We'll cover Multi-Stage Builds in detail in the Dockerfile module.

---

# Image Optimization

Professional Docker images follow these principles.

## Use Small Base Images

Prefer minimal images when suitable.

Example:

```text
node:22-alpine
```

instead of

```text
ubuntu
```

if your application doesn't require a full Ubuntu userspace.

---

## Reduce Layers

Bad

```dockerfile
RUN apt update

RUN apt install curl

RUN apt install git
```

Better

```dockerfile
RUN apt update && \
    apt install -y curl git
```

Fewer layers.

Smaller metadata.

Cleaner history.

---

## Remove Temporary Files

Bad

```dockerfile
RUN apt update
```

Good

```dockerfile
RUN apt update && \
    apt install -y curl && \
    rm -rf /var/lib/apt/lists/*
```

Keeps image smaller.

---

## Use .dockerignore

Never send unnecessary files.

Example

```
.git

node_modules

dist

.env

coverage
```

Smaller build context.

Faster builds.

---

# Image Security

Every image is software.

Software contains vulnerabilities.

Production checklist:

* Use official base images
* Keep base images updated
* Scan every image
* Remove unnecessary packages
* Don't run as root
* Don't store secrets
* Pin image versions

---

# Production Case Study

A team's image size increased from

```text
320 MB

↓

2.1 GB
```

Investigation:

```bash
docker history payment-api
```

Found:

```text
RUN apt install \
chrome \
gcc \
vim \
build-essential
```

Most packages were only required during build.

Solution:

* Multi-stage build
* Remove unnecessary tools
* Switch to Alpine runtime image

Final image:

```text
2.1 GB

↓

290 MB
```

Deployment became significantly faster.

---

# Performance Considerations

Large images cause:

* Slow CI builds
* Slow image pulls
* Longer deployments
* More registry storage
* Higher bandwidth usage

Small images improve:

* Startup time
* Network transfer
* Security review
* Patch management

---

# Common Mistakes

### Installing unnecessary packages

Every package increases:

* Image size
* Attack surface
* Vulnerability count

---

### Ignoring .dockerignore

Large build context

↓

Slow builds

---

### Rebuilding everything

Incorrect Dockerfile ordering

↓

Cache invalidation

↓

Long build times

---

### Using latest

Images become unpredictable.

---

# Troubleshooting Guide

## Why is my image huge?

Check:

```bash
docker history image
```

Look for large layers.

---

## Why does every build take 10 minutes?

Possible causes:

* Cache invalidation
* Huge build context
* No BuildKit
* Wrong Dockerfile order

---

## Why is docker pull slow?

Possible causes:

* Large layers
* Slow registry
* Weak network
* Cold cache

---

# Production Checklist

Before publishing an image:

* [ ] Version is pinned
* [ ] Image scanned
* [ ] No secrets included
* [ ] .dockerignore configured
* [ ] Minimal base image
* [ ] Multi-stage build used (where applicable)
* [ ] Runs as non-root
* [ ] Healthcheck defined
* [ ] Image tested
* [ ] Tagged correctly

---

# Advanced Interview Questions

### Q1

Explain how OverlayFS works internally.

Expected discussion:

* lowerdir
* upperdir
* merged
* workdir
* Copy-on-Write
* Whiteout files

---

### Q2

Why does Docker use layered images?

Expected discussion:

* Storage efficiency
* Cache reuse
* Faster builds
* Faster downloads
* Shared layers

---

### Q3

Explain the complete image pull process.

Expected discussion:

* Manifest
* Config
* Layer download
* Digest verification
* Local storage

---

### Q4

How would you reduce a 2 GB image to under 300 MB?

Expected discussion:

* Multi-stage builds
* Smaller base image
* Remove unnecessary packages
* .dockerignore
* Layer optimization
* Cleanup temporary files

---

# Quick Revision

Remember these rules:

```text
Image
    ↓
Read Only

Container
    ↓
Writable Layer

OverlayFS
    ↓
Merged Filesystem

Manifest
    ↓
Layer Index

Digest
    ↓
Immutable Identity

Tag
    ↓
Human-Friendly Label
```

---

> **Part 3 Complete**

The final part of this chapter will cover:

* BuildKit image internals
* Advanced image caching
* Lazy pulling
* Multi-platform image manifests
* Image lifecycle in CI/CD
* Registry optimization
* Real production architecture
* Complete image troubleshooting playbook
* Chapter summary and master revision sheet

---

# BuildKit and Modern Image Building

Traditional Docker builds every instruction sequentially.

Modern Docker uses **BuildKit**, which significantly improves build performance.

Benefits:

* Parallel build execution
* Better layer caching
* Secret mounts
* SSH forwarding
* Cache import/export
* Smaller build context
* Better logging

Today, BuildKit is the recommended build engine.

---

# BuildKit Architecture

```text
Docker CLI
      │
      ▼
 BuildKit
      │
      ▼
 Build Graph (LLB)
      │
      ▼
Parallel Execution
      │
      ▼
Optimized Image
```

Instead of treating a Dockerfile as just a list of commands,

BuildKit converts it into a dependency graph and optimizes execution wherever possible.

---

# Build Cache Internals

Docker cache is **layer-based**.

Each layer is identified using:

* Parent Layer
* Dockerfile Instruction
* Build Context
* File Contents

If all inputs are identical,

Docker safely reuses the cached layer.

---

## Cache Example

Dockerfile

```dockerfile
FROM node:22-alpine

WORKDIR /app

COPY package*.json .

RUN npm install

COPY . .

RUN npm run build
```

Scenario:

Only

```text
src/index.js
```

changed.

Docker reuses:

* FROM
* WORKDIR
* COPY package*.json
* npm install

Only rebuilds:

* COPY . .
* npm run build

This is why Dockerfile instruction order directly affects build speed.

---

# Cache Mounts

BuildKit also supports cache mounts.

Example:

```dockerfile
RUN --mount=type=cache,target=/root/.npm \
    npm install
```

Benefits:

* Faster package downloads
* Better CI performance
* Less network usage

Very common in production CI pipelines.

---

# Secret Mounts

One of BuildKit's biggest improvements.

Instead of:

```dockerfile
ENV TOKEN=my-secret
```

(which is insecure)

BuildKit supports temporary secrets.

Concept:

```text
Secret

↓

Build Step

↓

Used

↓

Automatically Removed
```

The secret never becomes part of the final image.

---

# SSH Mount

Sometimes builds need private Git repositories.

Example:

```text
Application

↓

Private Git Repository

↓

Build
```

BuildKit can temporarily forward your SSH agent without copying private keys into the image.

---

# Lazy Pulling

Older Docker versions downloaded the complete image before starting.

Modern container runtimes increasingly support lazy pulling concepts (depending on runtime and snapshotter), where image data is fetched as needed.

Benefits:

* Faster startup
* Reduced network usage
* Better scalability

This is especially useful for very large images.

---

# Multi-Platform Images

Earlier we learned:

```text
amd64

arm64
```

Question:

How can one image support both?

Answer:

A **Manifest List**.

Example:

```text
myapp:1.0

        │

        ▼

Manifest List

        │

 ┌──────┴───────┐

 ▼              ▼

amd64        arm64
Image         Image
```

Docker automatically downloads the correct image based on the host architecture.

---

# Image Lifecycle in CI/CD

Modern production pipeline:

```text
Developer

↓

Git Push

↓

CI Pipeline

↓

Unit Tests

↓

docker build

↓

Image Scan

↓

Image Sign

↓

docker push

↓

Registry

↓

Deployment

↓

Container Running
```

Images should be built **once** and promoted through environments.

---

# Build Once, Deploy Everywhere

One of Docker's biggest principles.

Wrong:

```text
Build

↓

Development

Build Again

↓

Testing

Build Again

↓

Production
```

Different builds may produce different artifacts.

Correct:

```text
Build Once

↓

Registry

↓

Development

↓

Testing

↓

Staging

↓

Production
```

Exactly the same image moves through every environment.

---

# Image Promotion

Example

```text
payment-api:2.4.1

↓

Development

↓

Testing

↓

Staging

↓

Production
```

Notice

Image never changes.

Only deployment target changes.

---

# Registry Optimization

Production registries should have:

* Image retention policy
* Vulnerability scanning
* Immutable tags (where possible)
* Lifecycle policies
* Garbage collection
* RBAC
* Audit logs

This keeps storage clean and secure.

---

# Real Production Architecture

```text
Developer

↓

Git

↓

GitLab CI

↓

BuildKit

↓

Docker Image

↓

Security Scan

↓

Image Signing

↓

Private Registry

↓

Kubernetes

↓

Running Pods
```

This architecture is common across modern cloud-native environments.

---

# Complete Image Troubleshooting Playbook

## Problem 1

Image size suddenly increased.

Check:

```bash
docker history image-name
```

Investigate:

* Large packages
* Build artifacts
* Cache files

---

## Problem 2

Build always starts from scratch.

Check:

* Dockerfile instruction order
* Build context changes
* BuildKit enabled?
* Cache disabled?

---

## Problem 3

Image pull is slow.

Check:

* Registry latency
* Layer sizes
* Existing cache
* Network throughput

---

## Problem 4

Wrong version deployed.

Never deploy:

```text
latest
```

Prefer:

```text
2.4.1
```

or

```text
sha256:...
```

---

## Problem 5

Image contains secrets.

Check:

* Dockerfile
* ENV instructions
* COPY commands
* Build arguments
* Image history

Never bake secrets into images.

---

# Production Best Practices

## Image Design

* Keep images immutable.
* Keep images small.
* One responsibility per image.
* Pin dependency versions.
* Scan before publishing.

---

## Security

* Use official base images.
* Run as non-root.
* Remove unnecessary packages.
* Never store credentials.
* Keep images patched.

---

## Performance

* Optimize Dockerfile order.
* Enable BuildKit.
* Use multi-stage builds.
* Minimize build context.
* Reuse cache effectively.

---

# Mental Models

## Mental Model 1

```text
Dockerfile

↓

Image

↓

Registry

↓

Container
```

---

## Mental Model 2

```text
Image

↓

Immutable

Container

↓

Mutable
```

---

## Mental Model 3

```text
Layers

↓

OverlayFS

↓

Merged Filesystem

↓

Running Application
```

---

## Mental Model 4

```text
Manifest

↓

Layer List

↓

Download Missing Layers

↓

Assemble Image
```

---

# Image Optimization Checklist

Before pushing any production image:

* [ ] Small base image
* [ ] Multi-stage build
* [ ] No secrets
* [ ] Version pinned
* [ ] `.dockerignore` configured
* [ ] Image scanned
* [ ] Non-root user
* [ ] Healthcheck included
* [ ] Unnecessary packages removed
* [ ] Correct tags applied

---

# Master Revision Sheet

Remember these concepts in order:

```text
Dockerfile
      │
      ▼
Image
      │
      ▼
Layers
      │
      ▼
OverlayFS
      │
      ▼
Writable Layer
      │
      ▼
Container
      │
      ▼
Registry
      │
      ▼
Production Deployment
```

If you understand this flow, you've understood how Docker images work internally.

---

# Chapter Summary

A Docker Image is **far more than a packaged application**.

Internally it consists of:

* Immutable layers
* OverlayFS
* Image metadata
* Manifests
* Configuration
* Digests
* Tags

Docker combines these technologies to provide:

* Fast builds
* Efficient storage
* Reproducible deployments
* Portable applications

Understanding Docker Images is the foundation for:

* Dockerfile optimization
* BuildKit
* Registries
* Kubernetes
* CI/CD
* Container security

---

# Cross References

Previous Chapter:

* Docker Architecture

Next Chapter:

➡ **04 - Dockerfile**

Related Topics:

* Docker Build
* BuildKit
* OverlayFS
* Docker Registry
* Multi-stage Builds
* Docker Security

---

# Golden Rules

> **Images are immutable.**

> **Containers are ephemeral.**

> **Layers are shared.**

> **Writable layers are temporary.**

> **Build once, deploy everywhere.**

> **Never use `latest` for production deployments.**
