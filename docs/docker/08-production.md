# Docker Security

> **Module:** 08 - Docker Security (Part 1)
>
> **Difficulty:** Advanced
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

---

# Table of Contents

* Learning Objectives
* Why Docker Security Matters
* Shared Kernel Model
* Threat Model
* Linux Security Features
* Namespaces
* Cgroups
* Linux Capabilities
* seccomp
* AppArmor
* SELinux
* Root vs Non-root Containers
* Rootless Docker
* Security Layers
* Hands-on Labs
* Interview Questions
* Quick Revision

---

# Learning Objectives

After this chapter you should be able to answer:

* Why are containers less isolated than virtual machines?
* Why is running as root dangerous?
* What are Linux Capabilities?
* What is seccomp?
* What is AppArmor?
* What is SELinux?
* What is Rootless Docker?
* How should production containers be secured?

---

# Why Docker Security Matters

Imagine your application has a Remote Code Execution (RCE) vulnerability.

An attacker gains command execution inside your container.

Question:

Can they take over the entire host?

Answer:

**It depends on how well the container is isolated.**

Docker security is about reducing what an attacker can do **after** gaining access.

---

# Mental Model

Think of a container as a room inside a building.

Without security:

```text id="s1a3nz"
Attacker

↓

Room

↓

Entire Building
```

With proper security:

```text id="7mqbg7"
Attacker

↓

Room

↓

Cannot Leave
```

The objective is **containment**.

---

# Shared Kernel Model

One of the biggest differences between VMs and containers.

Virtual Machine:

```text id="vaf7vx"
Application

↓

Guest OS

↓

Guest Kernel

↓

Hypervisor

↓

Host Kernel
```

Each VM has its own kernel.

---

Container:

```text id="1wukc5"
Application

↓

Container

↓

Host Kernel
```

Every container shares the **same Linux kernel**.

---

# Why Does This Matter?

Because a kernel vulnerability affects **all containers**.

Containers are isolated,

but they are **not as strongly isolated as virtual machines**.

This is why kernel security is so important.

---

# Docker Threat Model

Imagine:

```text id="5vkxtm"
Internet

↓

Container

↓

Docker Engine

↓

Linux Kernel

↓

Host
```

Possible attack paths:

* Vulnerable application
* Weak image
* Excessive privileges
* Kernel exploit
* Misconfigured Docker daemon
* Exposed Docker socket

Security aims to block each step.

---

# Security Layers

Think in layers.

```text id="2i7fsv"
Application

↓

Image

↓

Container

↓

Docker

↓

Linux Kernel

↓

Host

↓

Cloud
```

Security at only one layer is never enough.

This is known as **Defense in Depth**.

---

# Linux Namespaces (Security Perspective)

Earlier we studied namespaces for isolation.

Now view them as a security boundary.

Namespaces isolate:

* Processes
* Networking
* Mount points
* IPC
* Hostnames
* Users (with user namespaces)

This prevents containers from directly seeing host resources.

---

# Example

Container:

```bash id="jlwm01"
ps aux
```

Shows only container processes.

Host processes remain hidden.

Namespace isolation reduces information exposure.

---

# Cgroups (Security Perspective)

Cgroups are not just for resource management.

They also help prevent:

* CPU exhaustion
* Memory exhaustion
* Resource abuse

Example:

Without limits,

one container could consume all system memory.

With cgroups:

```text id="jlwm02"
Container

↓

512 MB Limit

↓

Cannot Consume Entire Host
```

---

# Linux Capabilities

This is one of the most important Docker security concepts.

Historically,

Linux had only two privilege levels:

```text id="jlwm03"
Root

Non-root
```

Too coarse.

Linux introduced **Capabilities**.

Instead of giving all root powers,

the kernel divides them into smaller privileges.

---

# Mental Model

Imagine root permissions as a keyring.

```text id="jlwm04"
Root

↓

100 Keys
```

Capabilities allow giving only the required keys.

Example:

```text id="jlwm05"
Application

↓

Needs Network

↓

Gets Network Capability

↓

Nothing Else
```

This follows the **Principle of Least Privilege**.

---

# Common Capabilities

Examples include:

* `CAP_NET_BIND_SERVICE` → Bind to low-numbered ports (below 1024)
* `CAP_CHOWN` → Change file ownership
* `CAP_SETUID` → Change user ID
* `CAP_KILL` → Send signals to processes

Docker removes many powerful capabilities by default.

---

# Dropping Capabilities

Example:

```bash id="jlwm06"
docker run \
--cap-drop ALL \
--cap-add NET_BIND_SERVICE \
myapp
```

Meaning:

Remove every capability.

Give back only the one needed.

This is much safer than running with broad privileges.

---

# Why Not Use --privileged?

Command:

```bash id="jlwm07"
docker run --privileged ubuntu
```

This gives the container **almost unrestricted access** to host resources.

Consequences:

* Device access
* Many kernel capabilities
* Reduced isolation

Production recommendation:

> **Avoid `--privileged` unless absolutely necessary.**

---

# Root User Inside Containers

Many beginners think:

```text id="jlwm08"
Root

↓

Container

↓

Safe
```

Not always.

Root inside a container still has significant power over resources available to that container.

If additional misconfigurations exist,

escaping the container becomes more dangerous.

---

# Run as Non-root

Instead of:

```dockerfile id="jlwm09"
USER root
```

Prefer:

```dockerfile id="jlwm10"
RUN adduser -D appuser

USER appuser
```

Benefits:

* Reduced attack impact
* Better compliance
* Least privilege
* Safer file permissions

---

# Production Example

Bad:

```dockerfile id="jlwm11"
FROM node:22

USER root
```

Better:

```dockerfile id="jlwm12"
FROM node:22-alpine

RUN adduser -D app

USER app
```

A compromise now has fewer privileges.

---

# seccomp

`seccomp` stands for **Secure Computing Mode**.

It limits which **system calls (syscalls)** a process can make to the Linux kernel.

---

# Why seccomp Exists

Application

↓

Makes system call

↓

Kernel

Question:

Should every process be allowed to call every syscall?

No.

Many syscalls are unnecessary for most applications.

Blocking unused syscalls reduces the attack surface.

---

# seccomp Flow

```text id="jlwm13"
Application

↓

System Call

↓

seccomp Filter

↓

Allowed?

↓

YES → Kernel

NO → Blocked
```

Docker ships with a default seccomp profile that blocks many high-risk syscalls while allowing those needed by common applications.

---

# AppArmor

AppArmor is a Linux Security Module.

It restricts **what a program can access**.

Example restrictions:

* Which files may be read
* Which directories may be written
* Which executables may run
* Which capabilities are allowed

Think of it as an additional security policy around your application.

---

# SELinux

SELinux is another Linux Security Module, common on Red Hat-based systems.

Instead of focusing only on user permissions,

SELinux applies **security labels** to processes and files.

Access decisions are based on those labels and policies.

This is why a container may receive **Permission Denied** even when traditional Unix permissions look correct.

---

# AppArmor vs SELinux

| AppArmor                   | SELinux                               |
| -------------------------- | ------------------------------------- |
| Path-based policies        | Label-based policies                  |
| Common on Ubuntu           | Common on RHEL, CentOS, Fedora        |
| Generally simpler to learn | More powerful, steeper learning curve |

Docker can integrate with both, depending on the host OS.

---

# Rootless Docker

Traditional Docker:

```text id="jlwm14"
Docker Daemon

↓

Root
```

Rootless Docker:

```text id="jlwm15"
User

↓

Docker

↓

Containers
```

No root-owned daemon is required for normal operation.

Benefits:

* Smaller attack surface
* Better isolation
* Reduced host risk

Limitations exist for some networking and storage features, so evaluate before adopting it everywhere.

---

# Security Principles

Follow these rules:

* Run as non-root.
* Drop unnecessary capabilities.
* Avoid `--privileged`.
* Use trusted images.
* Keep images updated.
* Apply resource limits.
* Use security profiles.
* Separate secrets from images.

---

# Common Beginner Mistakes

### Mistake 1

Running everything as root.

---

### Mistake 2

Using `--privileged` to "fix" permission problems.

---

### Mistake 3

Assuming containers provide VM-level isolation.

---

### Mistake 4

Ignoring Linux capabilities.

---

### Mistake 5

Trusting any image from a public registry.

---

# Hands-on Labs

## Lab 1

Run:

```bash id="jlwm16"
id
```

inside a container.

Observe the current user.

---

## Lab 2

Modify the Dockerfile to use:

```dockerfile id="’wini17"
USER app
```

Verify the application still works.

---

## Lab 3

Run a container with reduced capabilities.

Observe which operations now fail.

---

# Interview Questions

## Beginner

* Why shouldn't production containers run as root?
* What are Linux Capabilities?

---

## Intermediate

* Difference between AppArmor and SELinux?
* What does seccomp protect?

---

## Senior

Design a secure production container.

Expected discussion:

* Non-root user
* Minimal base image
* Dropped capabilities
* Read-only filesystem (where practical)
* Resource limits
* Healthchecks
* Trusted images
* No secrets baked into the image
* seccomp/AppArmor/SELinux integration

---

# Quick Revision

```text id="’wini18"
Namespaces

↓

Isolation

────────────

Cgroups

↓

Resource Control

────────────

Capabilities

↓

Least Privilege

────────────

seccomp

↓

Restrict Syscalls

────────────

AppArmor / SELinux

↓

Access Control

────────────

Rootless

↓

Reduced Host Risk
```

---

> **Part 1 Complete**

In **Part 2**, we'll cover:

* Image scanning
* CVEs
* Docker Scout
* Trivy
* SBOM
* Image signing
* Docker Content Trust / Notary concepts
* Secrets management
* Read-only root filesystems
* Security hardening checklist
* Real production security architecture
* Incident response

---

# Container Image Security

Question:

When does container security begin?

Many beginners think:

```text id="m2sa81"
Container Starts

↓

Security Begins
```

Wrong.

Security begins much earlier.

Correct flow:

```text id="h7pf30"
Developer

↓

Source Code

↓

Dockerfile

↓

Image Build

↓

Image Scan

↓

Registry

↓

Deployment

↓

Running Container
```

Every stage matters.

---

# Why Image Security Matters

Imagine this Dockerfile:

```dockerfile id="3vz8gm"
FROM ubuntu:18.04
```

Question:

What if Ubuntu 18.04 contains known vulnerabilities?

Every image built from it inherits those vulnerabilities.

The application may be perfectly written,

yet still be insecure.

---

# CVE (Common Vulnerabilities and Exposures)

A **CVE** is a publicly tracked security vulnerability.

Example:

```text id="rf8j31"
OpenSSL

↓

Critical Vulnerability

↓

Assigned CVE

↓

Patch Released
```

Security scanners compare your image against known CVEs.

---

# Image Scanning

Image scanners inspect:

* Operating system packages
* Installed libraries
* Language dependencies (where supported)
* Configuration issues

Goal:

Identify known vulnerabilities **before deployment**.

---

# Typical Image Scanning Flow

```text id="pw1g6m"
Docker Image

↓

Scanner

↓

Package Inventory

↓

CVE Database

↓

Security Report
```

---

# Popular Image Scanners

Common tools include:

* Docker Scout
* Trivy
* Grype
* Snyk Container

Many CI/CD pipelines automatically scan images before pushing them to a registry.

---

# Docker Scout

Docker Scout integrates with the Docker ecosystem.

Typical capabilities:

* Vulnerability detection
* Base image recommendations
* Image comparison
* Policy evaluation

Useful when already working heavily with Docker.

---

# Trivy

One of the most popular open-source scanners.

It can scan:

* Images
* Filesystems
* Git repositories
* Kubernetes manifests
* Infrastructure as Code

Because it is lightweight and easy to automate,

Trivy is widely used in CI pipelines.

---

# Should Every CVE Block Deployment?

No.

Severity matters.

Typical categories:

```text id="81lup0"
Critical

High

Medium

Low
```

Many organizations block deployments only for:

* Critical
* High

while Medium and Low are reviewed separately.

Risk depends on context.

---

# Why Smaller Images Are More Secure

Compare:

Image A

```text id="9b6v5f"
Ubuntu

+ GCC

+ Vim

+ Git

+ Curl

+ Python

+ Build Tools
```

Image B

```text id="s0v1he"
Distroless

↓

Application Only
```

Which has fewer attack opportunities?

Image B.

Fewer packages mean:

* Fewer CVEs
* Smaller attack surface
* Less maintenance

---

# Software Bill of Materials (SBOM)

An SBOM is like an ingredient list for software.

Instead of food ingredients,

it lists:

* Packages
* Libraries
* Versions
* Dependencies

Example:

```text id="yo5wzq"
Application

↓

SBOM

↓

OpenSSL 3.x

Node.js 22

Express 5

SQLite
```

When a new CVE is announced,

the SBOM helps determine whether your image is affected.

---

# Why SBOM Matters

Imagine:

A new OpenSSL vulnerability is published.

Without an SBOM,

you must manually inspect every image.

With an SBOM,

you immediately know:

* Which images contain OpenSSL
* Which version
* Which deployments require updates

This greatly speeds incident response.

---

# Image Signing

Question:

How do you know an image really came from your organization?

Answer:

Digital signatures.

Conceptually:

```text id="f8zt91"
Build Image

↓

Sign Image

↓

Push Registry

↓

Verify Signature

↓

Deploy
```

If the signature is invalid,

deployment can be blocked.

---

# Why Image Signing Exists

Suppose an attacker uploads:

```text id="3zrj8e"
payment-api:latest
```

to a compromised registry.

Without verification,

someone might deploy the malicious image.

Signing proves:

* Origin
* Integrity
* Authenticity

---

# Secrets Management

Earlier we learned:

Never write:

```dockerfile id="31u3lg"
ENV DB_PASSWORD=secret
```

Never commit:

```text id="t12gvl"
.env
```

with production credentials.

Better options:

* Docker Secrets
* Cloud secret managers
* External vault solutions

The application should retrieve secrets securely at runtime.

---

# Read-only Root Filesystem

Many containers never need to modify their own filesystem.

A read-only root filesystem reduces risk.

Conceptually:

```text id="9h1kq4"
Container

↓

Filesystem

↓

Read Only
```

If malware tries to modify application files,

the write operation fails.

Writable locations can still be provided through mounted volumes or temporary filesystems where necessary.

---

# Immutable Infrastructure

Instead of:

```text id="g7o6xb"
SSH

↓

Modify Container

↓

Continue Running
```

Modern practice is:

```text id="z6py4v"
Fix Source Code

↓

Build New Image

↓

Deploy New Container

↓

Delete Old Container
```

Never "patch" running containers.

Replace them.

---

# Runtime Secrets vs Build Secrets

Build Secrets

↓

Needed only while building.

Example:

* Private package registry token
* Git credentials

Runtime Secrets

↓

Needed while the application runs.

Example:

* Database password
* API keys
* TLS private keys

These should be managed differently.

---

# Docker Socket Security

One of the most dangerous mounts:

```text id="d1ql4x"
/var/run/docker.sock
```

Mounting the Docker socket gives a container the ability to communicate with the Docker daemon.

If misused, this can lead to full control over Docker-managed resources on the host.

Only mount it when absolutely necessary and understand the security implications.

---

# Security Hardening Checklist

Before deploying a production container:

* [ ] Trusted base image
* [ ] Pinned image version
* [ ] Minimal packages
* [ ] Multi-stage build
* [ ] Non-root user
* [ ] Drop unnecessary capabilities
* [ ] Default seccomp profile (or stricter if appropriate)
* [ ] AppArmor / SELinux policy applied where supported
* [ ] Resource limits configured
* [ ] Healthcheck configured
* [ ] Secrets managed externally
* [ ] Image scanned
* [ ] Image signed (where supported by your workflow)

---

# Real Production Security Pipeline

```text id="3cmk8q"
Developer

↓

Git Push

↓

CI Pipeline

↓

Unit Tests

↓

Docker Build

↓

SBOM Generation

↓

Image Scan

↓

Image Sign

↓

Private Registry

↓

Deployment

↓

Runtime Monitoring
```

Security is integrated throughout the lifecycle.

---

# Incident Response Example

Scenario:

A critical OpenSSL CVE is announced.

Response:

```text id="c8prx6"
Identify Affected Images

↓

Rebuild Using Patched Base Image

↓

Scan Again

↓

Sign Image

↓

Deploy

↓

Retire Old Image
```

Notice:

The application code may not change.

Only the base image changes.

---

# Common Beginner Mistakes

### Mistake 1

Using outdated base images.

---

### Mistake 2

Ignoring scanner warnings.

---

### Mistake 3

Treating `.env` as secure.

---

### Mistake 4

Installing unnecessary packages.

---

### Mistake 5

Mounting the Docker socket into every container.

---

# Production Best Practices

* Scan every image.
* Update base images regularly.
* Generate SBOMs.
* Sign production images.
* Use immutable deployments.
* Keep secrets outside images.
* Monitor newly published CVEs.

---

# Hands-on Labs

## Lab 1

Scan one of your images using an image scanner such as Trivy or Docker Scout.

Review:

* Critical
* High
* Medium
* Low

Findings.

---

## Lab 2

Replace a large base image with a smaller production-oriented base image.

Compare:

* Image size
* Number of vulnerabilities

---

## Lab 3

Generate an SBOM for one of your images (using your preferred tooling).

Inspect:

* Packages
* Versions
* Dependencies

---

# Advanced Interview Questions

## Q1

Why should images be scanned before deployment?

Expected discussion:

* CVEs
* Supply chain security
* Early detection
* CI integration

---

## Q2

What is an SBOM?

Expected discussion:

* Package inventory
* Dependency visibility
* Faster vulnerability management

---

## Q3

Why are Distroless images generally considered more secure?

Expected discussion:

* Fewer packages
* Smaller attack surface
* Fewer vulnerabilities
* Reduced maintenance

---

## Q4

How would you secure a production container pipeline?

Expected discussion:

* Trusted base images
* Multi-stage builds
* Image scanning
* SBOM generation
* Image signing
* External secrets
* Runtime hardening
* Continuous monitoring

---

# Master Mental Models

## Mental Model 1

```text id="8f5vnc"
Source Code

↓

Dockerfile

↓

Image

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

```text id="p7lzq1"
Security

↓

Build Time

↓

Registry

↓

Deployment

↓

Runtime
```

Security is continuous, not a single step.

---

# Chapter Summary

Docker security combines multiple layers:

* Secure images
* Secure builds
* Secure runtime
* Secure host configuration
* Secure secret management
* Continuous vulnerability management

No single feature makes containers secure.

Real security comes from **layering multiple controls together** and following the principle of least privilege.

---

# Cross References

Previous Chapter:

* Docker Compose

Next Chapter:

➡ **09 - Registries & Image Distribution**

Related Topics:

* Dockerfile
* BuildKit
* Docker Secrets
* Image Registries
* Supply Chain Security

---

# Golden Rules

> **Run as non-root whenever possible.**

> **Never trust an image without verification.**

> **Scan every production image.**

> **Keep secrets outside images.**

> **Patch by rebuilding images, not by modifying running containers.**

> **Security is a continuous process, not a one-time task.**
