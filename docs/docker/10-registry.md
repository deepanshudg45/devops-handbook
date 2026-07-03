# Docker Registries & Image Distribution

> **Module:** 09 - Registries & Image Distribution (Part 1)
>
> **Difficulty:** Intermediate → Advanced
>
> **Estimated Reading Time:** 90–120 Minutes
>
> **Prerequisites:**
>
> * Docker Fundamentals
> * Docker Architecture
> * Docker Images
> * Dockerfile
> * Docker Volumes
> * Docker Networking
> * Docker Compose
> * Docker Security

---

# Table of Contents

* Learning Objectives
* Why Registries Exist
* Registry Architecture
* Docker Hub
* Private Registries
* Image Push Flow
* Image Pull Flow
* Tags
* Digests
* Authentication
* Image Lifecycle
* Hands-on Labs
* Interview Questions
* Quick Revision

---

# Learning Objectives

After this chapter, you should be able to answer:

* What is a Docker Registry?
* Why do registries exist?
* Difference between Docker Hub and a private registry?
* What happens during `docker push`?
* What happens during `docker pull`?
* Why are image digests important?
* Why shouldn't production use `latest`?

---

# Why Docker Registries Exist

Imagine every developer builds an image locally.

Developer A:

```text id="4j9x2r"
payment-api:v1
```

Developer B:

```text id="t3w1y8"
payment-api:v1
```

Production server:

No image.

How does production get the correct image?

Answer:

A registry.

---

# Mental Model

Think of Git.

Code lives in:

```text id="9m0s7v"
Git Repository
```

Images live in:

```text id="m7g5qe"
Container Registry
```

Just as Git stores source code,

registries store container images.

---

# Registry Architecture

```text id="6v3cn4"
Developer

↓

docker build

↓

Image

↓

docker push

↓

Registry

↓

docker pull

↓

Production
```

The registry becomes the central source of truth.

---

# Why Not Copy Images Manually?

Imagine copying images using USB drives.

Problems:

* No version control
* No integrity checks
* No collaboration
* No automation

Registries solve all of these.

---

# Docker Hub

The default public registry.

When you run:

```bash id="1qx4pc"
docker pull nginx
```

Docker automatically contacts Docker Hub unless another registry is specified.

---

# Docker Hub Structure

Example:

```text id="f7lk92"
docker.io/library/nginx:1.27
```

Breakdown:

```text id="2pw0ec"
Registry

↓

docker.io

Namespace

↓

library

Repository

↓

nginx

Tag

↓

1.27
```

---

# Repository

Think of a repository as a folder containing different versions of the same image.

Example:

```text id="3x8sbl"
payment-api

├── v1.0

├── v1.1

├── v2.0

└── latest
```

---

# Tags

Tags are human-friendly labels.

Example:

```text id="4m6gpd"
payment-api

↓

v1.0

v1.1

v2.0
```

Tags help identify versions.

But remember:

Tags are **mutable**.

They can point to different images over time.

---

# Digests

Every image also has an immutable SHA256 digest.

Example:

```text id="9qy3fd"
sha256:abc123...
```

Unlike tags,

digests never change.

Production deployments often use digests for maximum reproducibility.

---

# Tag vs Digest

| Tag              | Digest             |
| ---------------- | ------------------ |
| Human-readable   | Machine identifier |
| Mutable          | Immutable          |
| Easy to remember | Cryptographic hash |
| Can change       | Never changes      |

---

# Why Not Use latest?

Question:

What does:

```text id="3sh8tn"
latest
```

actually mean?

Answer:

Whatever image the publisher currently points `latest` to.

It is **not** guaranteed to be the newest version numerically, and it can change unexpectedly.

Production systems should use explicit version tags or digests.

---

# Private Registry

Public registries are useful,

but many companies build proprietary software.

They don't want their images publicly available.

Solution:

Private Registry.

---

# Architecture

```text id="8crw5a"
Developer

↓

CI Pipeline

↓

Private Registry

↓

Production Servers
```

Only authorized users and systems can pull images.

---

# Registry Authentication

Before pushing:

```bash id="e5m0w2"
docker login
```

Docker stores authentication credentials (or uses an external credential helper if configured).

After authentication:

```bash id="r4z7pd"
docker push
```

becomes possible.

---

# docker push Flow

Internally:

```text id="0h2k7e"
Image

↓

Read Manifest

↓

Compare Layers

↓

Upload Missing Layers

↓

Upload Manifest

↓

Registry Stores Image
```

Docker uploads only missing layers.

Shared layers are reused.

---

# docker pull Flow

```text id="b6p4zu"
Registry

↓

Download Manifest

↓

Check Local Layers

↓

Download Missing Layers

↓

Verify Digest

↓

Image Ready
```

Again,

Docker downloads only what is needed.

---

# Layer Reuse

Suppose:

Image A

```text id="v1n7sy"
Ubuntu

Node

App V1
```

Image B

```text id="zt9c4u"
Ubuntu

Node

App V2
```

Ubuntu and Node layers already exist locally.

Docker downloads only the new application layer.

Huge bandwidth savings.

---

# Registry Naming

Typical format:

```text id="7sq3kp"
registry.example.com/team/payment-api:v2.1.0
```

Components:

```text id="m1fj9d"
Registry

↓

Namespace

↓

Repository

↓

Tag
```

---

# Image Lifecycle

```text id="h8wr4c"
Developer

↓

Build

↓

Scan

↓

Push

↓

Registry

↓

Pull

↓

Deploy

↓

Run
```

Every production image follows this lifecycle.

---

# Production Pitfall

❌ Deploy:

```text id="jlwm01"
payment-api:latest
```

Today it works.

Tomorrow someone pushes a new image tagged `latest`.

Without changing your deployment configuration,

your next deployment may run different code.

---

✅ Better:

```text id="jlwm02"
payment-api:2.4.1
```

or even:

```text id="jlwm03"
payment-api@sha256:...
```

---

# Hands-on Labs

## Lab 1

Build an image.

Tag it:

```bash id="jlwm04"
docker tag app:v1 myuser/app:v1
```

---

## Lab 2

Log in to a registry.

Push the image.

Verify it appears in the repository.

---

## Lab 3

Delete the local image.

Pull it again.

Observe that Docker reconstructs the image from downloaded layers.

---

# Interview Questions

## Beginner

* What is a Docker Registry?
* Difference between Repository and Registry?
* Difference between Tag and Digest?

---

## Intermediate

* Explain the `docker push` process.
* Explain the `docker pull` process.
* Why shouldn't production use `latest`?

---

## Quick Revision

```text id="jlwm05"
Build

↓

Tag

↓

Push

↓

Registry

↓

Pull

↓

Run
```

---

# Golden Rules

> Registries are the source of truth for container images.

> Tags are convenient; digests are immutable.

> Push images once, pull them everywhere.

> Never rely on `latest` for production deployments.

---

> **Part 1 Complete**

In **Part 2** we'll cover:

* Private Registry
* AWS ECR
* GitHub Container Registry (GHCR)
* GitLab Container Registry
* Harbor
* Image signing
* Multi-architecture images
* OCI Distribution Specification
* Registry garbage collection
* Production architectures
* Troubleshooting
* Interview questions

---

# Private Registries

A public registry is convenient.

A company registry is secure.

Question:

Why would a company never publish its production images publicly?

Because images may contain:

* Proprietary applications
* Internal APIs
* Company architecture
* Business logic

These should remain private.

---

# Private Registry Architecture

```text id="pv1k2m"
Developer

↓

Git Push

↓

CI Pipeline

↓

Build Image

↓

Private Registry

↓

Production Servers

↓

Pull Image
```

The registry becomes the company's internal image repository.

---

# Self-Hosted Registry

Docker provides an official Registry implementation.

Typical deployment:

```text id="fq0zr8"
Docker Host

↓

Registry Container

↓

Persistent Volume

↓

Image Storage
```

Many organizations start here for internal environments.

---

# Why Private Registries?

Benefits:

* Access control
* Faster internal downloads
* Security
* Image retention
* Vulnerability scanning
* Audit logs

---

# AWS ECR (Elastic Container Registry)

Amazon ECR is AWS's managed container registry.

Typical architecture:

```text id="n7d0au"
Developer

↓

CI/CD

↓

Amazon ECR

↓

Amazon ECS / EKS / EC2
```

Advantages:

* Fully managed
* IAM integration
* Lifecycle policies
* Regional availability
* Vulnerability scanning (supported)

---

# Why Companies Like ECR

No registry server to maintain.

AWS handles:

* Availability
* Scaling
* Storage
* Authentication integration

Developers focus on building applications.

---

# Authentication with ECR

Unlike Docker Hub,

authentication is usually obtained through AWS credentials and a temporary authorization token.

Flow:

```text id="m2x7wp"
AWS Credentials

↓

Temporary Login Token

↓

docker login

↓

Push / Pull
```

This avoids using long-lived registry passwords.

---

# GitHub Container Registry (GHCR)

GitHub provides its own container registry.

Typical flow:

```text id="g0v8hl"
GitHub Repository

↓

GitHub Actions

↓

GHCR

↓

Deployment
```

Advantages:

* Integrated with GitHub
* Supports private repositories
* Fine-grained permissions
* Works well with GitHub Actions

---

# Common GHCR Use Cases

* Open-source projects
* Internal development
* CI/CD pipelines
* Multi-platform images

---

# GitLab Container Registry

GitLab also provides an integrated registry.

Architecture:

```text id="h3c9rw"
Git Push

↓

GitLab CI

↓

Build Image

↓

GitLab Registry

↓

Deploy
```

Since the registry is built into GitLab,

CI pipelines can push images without requiring a separate registry server.

---

# Typical GitLab Pipeline

```text id="u8r5fm"
Developer

↓

Commit

↓

Pipeline

↓

Build

↓

Scan

↓

Push Registry

↓

Deploy
```

Very common in enterprise environments.

---

# Harbor

Harbor is an enterprise-grade open-source registry.

Key features:

* Role-Based Access Control (RBAC)
* Image signing support
* Replication
* Vulnerability scanning
* Project isolation
* Audit logs
* Garbage collection

Large organizations often choose Harbor when they need full control over their registry infrastructure.

---

# Registry Comparison

| Registry                  | Managed     | Best Use Case               |
| ------------------------- | ----------- | --------------------------- |
| Docker Hub                | Yes         | Public images               |
| Private Docker Registry   | Self-hosted | Small internal environments |
| AWS ECR                   | Yes         | AWS workloads               |
| GitHub Container Registry | Yes         | GitHub-centric projects     |
| GitLab Container Registry | Yes         | GitLab CI/CD                |
| Harbor                    | Self-hosted | Enterprise private registry |

---

# Image Signing

Question:

How do you know an image wasn't modified?

Answer:

Digital signatures.

Concept:

```text id="l9p6yk"
Image

↓

Sign

↓

Registry

↓

Verify

↓

Deploy
```

If verification fails,

deployment can be rejected.

---

# Why Image Signing Matters

Imagine:

```text id="e6m4qs"
payment-api:v2
```

stored in a registry.

An attacker somehow replaces it.

Without verification,

deployment proceeds.

With signature verification,

the image is rejected because the signature no longer matches.

---

# OCI Distribution Specification

Docker images are no longer tied only to Docker.

The **OCI Distribution Specification** defines how compliant registries and clients exchange container images.

Benefits:

* Vendor neutrality
* Interoperability
* Standard APIs

Because of OCI,

many tools can interact with the same registry ecosystem.

---

# Multi-Architecture Images

Suppose you build:

```text id="r3b0hf"
amd64
```

Question:

Will it run on:

```text id="x2m7aj"
arm64
```

Not necessarily.

Different CPU architectures require compatible binaries.

---

# Manifest Lists

Modern registries store:

```text id="t6q8cn"
payment-api:2.0

↓

Manifest List

↓

amd64 Image

↓

arm64 Image

↓

arm/v7 Image
```

When pulling,

Docker automatically downloads the correct image for the host architecture.

---

# Why Multi-Architecture Matters

Examples:

Developer laptop

↓

Apple Silicon (arm64)

Production

↓

Intel Servers (amd64)

One tag,

multiple images,

automatic selection.

---

# Registry Garbage Collection

Over time,

registries accumulate:

* Old tags
* Deleted images
* Unreferenced layers

Garbage collection removes unused data to recover storage.

Conceptually:

```text id="q5z8lt"
Registry

↓

Unused Layers

↓

Garbage Collection

↓

Storage Reclaimed
```

---

# Lifecycle Policies

Enterprise registries often support automatic cleanup.

Example policy:

```text id="v1k9jh"
Keep

Last 20 Images

Delete

Everything Older
```

Benefits:

* Lower storage costs
* Cleaner repositories
* Easier maintenance

---

# Image Promotion

One image should move through environments.

Not be rebuilt.

Correct flow:

```text id="p8g2fd"
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

This guarantees every environment runs the exact same artifact.

---

# Production Registry Architecture

```text id="z4n1cm"
Developer

↓

Git Push

↓

CI Pipeline

↓

Build

↓

SBOM

↓

Image Scan

↓

Image Sign

↓

Private Registry

↓

Production Cluster
```

Notice:

The registry is the central distribution point.

---

# Registry Security Best Practices

* Require authentication.
* Use least-privilege access.
* Scan images before publishing.
* Sign production images.
* Remove unused images.
* Enable audit logging.
* Use immutable tags where supported.

---

# Common Beginner Mistakes

### Mistake 1

Using `latest` everywhere.

---

### Mistake 2

Pushing development images into production repositories.

---

### Mistake 3

Skipping image scanning.

---

### Mistake 4

Never cleaning old images.

---

### Mistake 5

Granting everyone push access.

---

# Hands-on Labs

## Lab 1

Push an image to:

* Docker Hub **or**
* GitHub Container Registry **or**
* GitLab Container Registry

Verify the repository contains the expected tag.

---

## Lab 2

Create two version tags:

```text id="j7w0qn"
v1.0

v1.1
```

Pull both and inspect their image IDs.

---

## Lab 3

Explore a registry's lifecycle or retention policy settings.

Understand how old images are removed automatically.

---

# Advanced Interview Questions

## Q1

Difference between Docker Hub and AWS ECR?

Expected discussion:

* Public vs managed private registry
* IAM integration
* Enterprise features
* AWS ecosystem

---

## Q2

What is a multi-architecture image?

Expected discussion:

* Manifest list
* Different CPU architectures
* Automatic selection during pull

---

## Q3

Why should production use image digests instead of mutable tags?

Expected discussion:

* Immutability
* Reproducibility
* Supply chain integrity

---

## Q4

Design a secure image distribution pipeline.

Expected discussion:

* Build
* Scan
* SBOM
* Sign
* Private registry
* Deployment
* Runtime monitoring

---

# Master Mental Models

## Mental Model 1

```text id="a3t8km"
Build

↓

Scan

↓

Sign

↓

Registry

↓

Deploy
```

---

## Mental Model 2

```text id="w6n1qy"
Registry

↓

Repositories

↓

Tags

↓

Digests

↓

Images
```

---

## Mental Model 3

```text id="d4m7ps"
One Image

↓

Many Environments

↓

Same Artifact
```

---

# Chapter Summary

Container registries are the distribution backbone of modern container platforms.

They provide:

* Centralized storage
* Version management
* Access control
* Vulnerability scanning
* Image signing
* Multi-architecture support
* Integration with CI/CD

In production,

registries become the trusted source from which every deployment retrieves container images.

---

# Cross References

Previous Chapter:

* Docker Security

Next Chapter:

➡ **10 - Real Production Troubleshooting Labs**

Related Topics:

* Docker Images
* BuildKit
* CI/CD
* Kubernetes
* Supply Chain Security

---

# Golden Rules

> **Build once, deploy everywhere.**

> **Treat the registry as the single source of truth for images.**

> **Use immutable versions or digests for production deployments.**

> **Scan and sign images before publishing.**

> **Never promote unverified images into production.**
