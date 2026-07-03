# Docker Volumes

> **Module:** 05 - Docker Volumes (Part 1)
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

---

# Table of Contents

* Learning Objectives
* Key Takeaways
* Why Docker Volumes Exist
* The Ephemeral Container Problem
* Writable Layer
* Why Data Gets Lost
* What is a Docker Volume?
* Volume Architecture
* Docker Storage Overview
* Types of Docker Storage
* Copy-on-Write Relationship
* Production Use Cases
* Common Mistakes
* Mental Models
* Hands-on Lab
* Quick Revision

---

# Learning Objectives

After completing this chapter, you should be able to answer:

* Why do Docker Volumes exist?
* Why is container data temporary?
* What happens to the Writable Layer?
* What is the difference between a Volume and the Writable Layer?
* Why are Volumes stored outside containers?
* Why are databases almost always deployed using Volumes?
* How does Docker manage persistent storage?

---

# Key Takeaways

Remember these principles first.

* Containers are **ephemeral**.
* Images are **immutable**.
* Writable Layers are **temporary**.
* Volumes are **persistent**.
* Containers come and go.
* Data should survive container replacement.

These six points explain almost every storage-related decision in Docker.

---

# The Biggest Beginner Mistake

Many beginners think:

```text
Container

↓

Stores Data Forever
```

This is wrong.

A container is **not** permanent storage.

Containers are designed to be:

* Created
* Started
* Stopped
* Deleted
* Recreated

at any time.

Therefore,

important data must live **outside** the container.

---

# Why Docker Volumes Exist

Let's understand the problem first.

Suppose you run MySQL.

```bash
docker run mysql
```

Database starts.

Now create a database.

```sql
CREATE DATABASE company;
```

Insert customer records.

Everything works.

Now someone removes the container.

```bash
docker rm mysql
```

Question:

Where is the database?

Answer:

Gone.

Why?

Because the database lived inside the container's Writable Layer.

---

# The Ephemeral Container Problem

Container lifecycle:

```text
Image
    │
    ▼
Container Created
    │
    ▼
Application Writes Data
    │
    ▼
Writable Layer
    │
    ▼
Container Deleted
    │
    ▼
Writable Layer Deleted
    │
    ▼
Data Lost
```

Docker is behaving exactly as designed.

The mistake is storing persistent data in the wrong place.

---

# Writable Layer Review

Earlier we learned:

Every container receives one Writable Layer.

Structure:

```text
Image Layers
(Read Only)

↓

Writable Layer

↓

Running Container
```

Everything created during runtime goes into the Writable Layer.

Examples:

* Uploaded files
* Log files
* Database files
* Cache
* Temporary files

Unless another storage mechanism is used.

---

# Why Does Docker Delete the Writable Layer?

Think from Docker's perspective.

A container is a temporary runtime object.

When the container is deleted,

its runtime filesystem is no longer needed.

So Docker removes:

* Process metadata
* Network namespace
* Writable Layer

Result:

All runtime data disappears.

This behavior keeps containers lightweight and disposable.

---

# The Solution

Instead of storing data **inside** the container,

store it **outside** the container.

Visual:

```text
Before

Container

↓

Writable Layer

↓

Database
```

Problem:

Delete container

↓

Database deleted.

---

Correct design:

```text
Container

↓

Volume

↓

Host Storage
```

Now,

container can be deleted.

Volume remains.

---

# What is a Docker Volume?

A Docker Volume is a Docker-managed storage location that exists independently of any container.

Important characteristics:

* Persistent
* Reusable
* Independent
* Managed by Docker

A volume continues to exist even if every container using it is removed (unless the volume itself is explicitly removed).

---

# Mental Model

Think of a mobile phone.

Applications:

* WhatsApp
* Instagram
* Telegram

All applications can be uninstalled.

But your SD card remains.

Docker Volume

=

SD Card

Container

=

Application

Delete the application.

SD card still exists.

---

# Volume Architecture

```text
                Docker Host
──────────────────────────────────

Docker Engine

       │

       ▼

 Docker Volume

       │

       ▼

 Host Filesystem

──────────────────────────────────

       ▲

       │

Container A

Container B

Container C
```

Notice

The Volume belongs to Docker,

not to the container.

---

# Relationship Between Image, Container and Volume

```text
Dockerfile

↓

Image

↓

Container

↓

Volume

↓

Persistent Data
```

Images create containers.

Containers use volumes.

Volumes store data.

Each component has a different responsibility.

---

# Docker Storage Overview

Docker provides multiple storage mechanisms.

```text
Container

├── Writable Layer

├── Volumes

├── Bind Mounts

└── tmpfs
```

Each one solves a different problem.

We'll study every type in detail in the next sections.

---

# Writable Layer vs Volume

| Writable Layer          | Volume                     |
| ----------------------- | -------------------------- |
| Temporary               | Persistent                 |
| Deleted with container  | Independent                |
| Copy-on-Write           | Direct storage             |
| Slower for heavy writes | Better for persistent data |
| Container specific      | Can be shared              |

This distinction is extremely important.

---

# Copy-on-Write Relationship

Remember:

Writable Layer uses Copy-on-Write.

Volumes do **not**.

Visual:

```text
Container

↓

Writable Layer

↓

Copy-on-Write

──────────────

Volume

↓

Direct Storage
```

This is one reason databases prefer Volumes.

---

# Why Databases Use Volumes

Imagine MySQL storing millions of records.

If everything lived inside the Writable Layer:

* Container deletion destroys data.
* Heavy Copy-on-Write activity can reduce efficiency.
* Backup becomes harder.

Using Volumes solves these issues.

Production databases should always store their data on persistent storage.

---

# Real Production Examples

Applications that almost always require Volumes:

* MySQL
* PostgreSQL
* MongoDB
* Redis (depending on persistence settings)
* Jenkins
* Nexus Repository
* SonarQube
* Grafana
* Prometheus
* GitLab
* Elasticsearch

These applications manage valuable state that must survive container replacement.

---

# Why Containers Are Designed This Way

Docker follows the philosophy of **Immutable Infrastructure**.

Instead of modifying a running container,

the preferred workflow is:

```text
Fix Code

↓

Build New Image

↓

Create New Container

↓

Attach Existing Volume

↓

Continue Running
```

Containers are replaceable.

Data is not.

---

# Production Pitfall

❌ Bad

```text
Application

↓

Stores customer uploads

↓

Writable Layer
```

Delete container.

Customer files disappear.

---

✅ Correct

```text
Application

↓

Docker Volume

↓

Persistent Storage
```

---

# Common Mistakes

## Mistake 1

Thinking containers are permanent.

---

## Mistake 2

Using the Writable Layer for databases.

---

## Mistake 3

Deleting a container without understanding where data is stored.

---

## Mistake 4

Treating Volumes as backups.

Volumes provide persistence.

They are **not** a backup strategy.

You still need proper backups.

---

# Hands-on Lab

Run:

```bash
docker run -it --name demo ubuntu
```

Inside the container:

```bash
echo "Hello Docker" > /tmp/test.txt
```

Exit.

Remove the container.

Create another container.

Check:

```bash
cat /tmp/test.txt
```

Question:

Why is the file missing?

Think about the Writable Layer before reading the answer.

---

# Self-Assessment

Try answering these without looking back:

1. Why are containers called ephemeral?
2. What is the Writable Layer?
3. Why is a Volume independent of a container?
4. Why do databases require Volumes?
5. What happens to runtime data when a container is deleted?

If you can answer all five confidently, your storage foundation is strong.

---

# Quick Revision

```text
Image
    ↓
Read Only

Container
    ↓
Writable Layer
    ↓
Temporary

Volume
    ↓
Persistent

Container Deleted
    ↓
Writable Layer Deleted

Volume
    ↓
Still Exists
```

---

# Key Mental Models

## Mental Model 1

```text
Image

↓

Container

↓

Volume
```

Containers execute applications.

Volumes protect data.

---

## Mental Model 2

```text
Container

=

Application

Volume

=

Hard Disk
```

Applications can be reinstalled.

The hard disk remains.

---

## Golden Rules

> Containers are temporary.

> Data is permanent.

> Never store important data only in the Writable Layer.

> Build new containers instead of modifying old ones.

> Volumes exist to separate **application lifecycle** from **data lifecycle**.

---

> **Part 1 Complete**

In Part 2 we will study the four storage types in depth:

* Anonymous Volumes
* Named Volumes
* Bind Mounts
* tmpfs Mounts

We'll cover:

* Internal working
* Linux implementation
* Production use cases
* Advantages & disadvantages
* Performance
* Security
* Edge cases
* Real production architecture
* Troubleshooting
* Interview questions

---

# Docker Storage Types

Docker provides four major storage mechanisms.

```text
Docker Storage

├── Writable Layer
│
├── Named Volumes
│
├── Anonymous Volumes
│
├── Bind Mounts
│
└── tmpfs Mounts
```

Each one solves a different problem.

Choosing the wrong storage type can lead to:

* Data loss
* Poor performance
* Security issues
* Difficult deployments

---

# 1. Named Volumes ⭐⭐⭐⭐⭐

This is the most commonly used storage type in production.

Syntax

```bash
docker volume create mysql-data
```

Use it:

```bash
docker run \
-v mysql-data:/var/lib/mysql \
mysql
```

---

# Internal Working

Docker creates a managed storage location.

Conceptually:

```text
Docker Host

↓

Docker Volume

↓

mysql-data

↓

Container

↓

/var/lib/mysql
```

Docker manages everything.

You don't need to know the host path.

---

# Mental Model

Think of it like renting a locker.

Locker Name

↓

"MySQL-Data"

No matter who uses it,

the locker remains.

Containers come and go.

Locker stays.

---

# Why Named?

Because Docker stores the volume with a name.

Example

```text
mysql-data

postgres-data

jenkins-home

grafana-storage
```

Easy to identify.

Easy to reuse.

---

# Advantages

* Docker managed
* Easy backup
* Easy migration
* Persistent
* Can be shared
* Works well with Compose
* Works well with Swarm

---

# Disadvantages

* Files aren't immediately visible in your project directory.
* Requires Docker commands for management.

---

# Production Use Cases

Almost every stateful service:

* MySQL
* PostgreSQL
* MongoDB
* Redis persistence
* Jenkins
* Nexus
* SonarQube
* Prometheus
* Grafana

---

# Real Production Example

```yaml
services:

  mysql:

    image: mysql

    volumes:

      - mysql-data:/var/lib/mysql

volumes:

  mysql-data:
```

This is how many production Compose files are written.

---

# Lifecycle

```text
Container Deleted

↓

Volume Exists

↓

New Container

↓

Same Data
```

Exactly what we want.

---

# Production Pitfall

Deleting the container:

```bash
docker rm mysql
```

does **not** delete the named volume.

Deleting the volume:

```bash
docker volume rm mysql-data
```

does.

Always know which command you're running.

---

# 2. Anonymous Volumes

Syntax

```bash
docker run \
-v /var/lib/mysql \
mysql
```

Notice:

No volume name.

Docker automatically generates one.

Example

```text
0a84a5b7...

f91a33cd...
```

Random names.

---

# Internal Working

Docker creates:

```text
Random Volume

↓

Container

↓

Application
```

Container doesn't care.

Docker keeps track internally.

---

# Why Anonymous Volumes Exist?

Sometimes applications need persistent writable storage,

but you don't care about managing it manually.

Docker automatically creates a volume.

Common in official images.

---

# Example

Official MySQL image declares:

```dockerfile
VOLUME /var/lib/mysql
```

If you don't provide a volume,

Docker automatically creates an anonymous volume.

---

# Problem

Suppose

```bash
docker rm mysql
```

Question

Where is the anonymous volume?

Still exists.

Many beginners don't realize this.

Eventually:

```bash
docker volume ls
```

shows dozens of unused anonymous volumes.

Disk fills up.

---

# Cleanup

```bash
docker volume prune
```

Removes unused volumes.

Be careful.

Never run this blindly on production.

---

# When Should You Use Anonymous Volumes?

Mostly when:

* Testing
* Temporary containers
* Official images managing internal data

Rarely used intentionally in production.

---

# 3. Bind Mounts ⭐⭐⭐⭐⭐

This is the second most important storage type.

Syntax

```bash
docker run \
-v $(pwd):/app
```

or

```bash
docker run \
--mount type=bind,source=$(pwd),target=/app
```

---

# Internal Working

Instead of Docker creating storage,

Docker maps an existing host directory.

```text
Host Directory

↓

Docker

↓

Container
```

Both see the same files.

---

# Mental Model

Think of a shared folder.

Host

↓

Shared Folder

↓

Container

Everyone accesses the same files.

---

# Real Example

Project

```text
project/

index.html

style.css

app.js
```

Run

```bash
docker run \
-v $(pwd):/usr/share/nginx/html \
nginx
```

Modify

```text
index.html
```

Browser updates immediately.

No rebuild required.

---

# Why Developers Love Bind Mounts

Development workflow:

```text
VS Code

↓

Save File

↓

Host Filesystem

↓

Bind Mount

↓

Container

↓

Instant Change
```

Very fast.

---

# Advantages

* Immediate file changes
* Easy debugging
* Excellent for development
* No image rebuild required

---

# Disadvantages

* Host dependent
* Security risk
* Different paths on Windows/Linux/macOS
* Harder portability

---

# Production Warning

Avoid using bind mounts for application code in production unless you have a specific operational reason.

Production should typically deploy immutable images.

---

# Good Production Uses

* Log collection
* Configuration files
* TLS certificates
* Reverse proxy configuration

Example:

```text
/etc/nginx/nginx.conf
```

mounted from host.

---

# 4. tmpfs Mounts

Question

Need storage,

but never write to disk?

Use tmpfs.

---

# Internal Working

```text
RAM

↓

tmpfs

↓

Container
```

No disk.

Everything stored in memory.

---

# Characteristics

* Extremely fast
* Non-persistent
* Automatically deleted
* Never written to disk

---

# Use Cases

* Temporary secrets
* Session storage
* Encryption keys
* Build scratch space
* Temporary cache

---

# Why Faster?

Disk

↓

SSD/HDD

↓

Filesystem

↓

Application

tmpfs

↓

RAM

↓

Application

No disk latency.

---

# Security Benefit

When the container stops,

RAM is cleared.

Sensitive temporary data disappears automatically.

---

# Storage Comparison

| Feature             | Named Volume | Anonymous Volume | Bind Mount | tmpfs                |
| ------------------- | ------------ | ---------------- | ---------- | -------------------- |
| Persistent          | ✅            | ✅                | ✅          | ❌                    |
| Docker Managed      | ✅            | ✅                | ❌          | ✅                    |
| Host Path Visible   | ❌            | ❌                | ✅          | ❌                    |
| Best for Production | ✅            | ⚠️ Rarely        | ⚠️ Depends | ✅ Specific use cases |
| Fastest             | Good         | Good             | Depends    | ⭐ Fastest            |

---

# Which Should I Use?

## Database

```text
Named Volume
```

---

## Local Development

```text
Bind Mount
```

---

## Temporary Secrets

```text
tmpfs
```

---

## Testing

```text
Anonymous Volume
```

---

# Production Decision Flow

```text
Need Persistent Data?

        │

       YES

        │

Need Host Files?

   │            │

 YES           NO

 │             │

Bind       Named Volume


──────────────

Need Persistence?

        │

       NO

        │

Need RAM Only?

   │           │

 YES          NO

 │            │

tmpfs   Writable Layer
```

---

# Common Beginner Mistakes

### Mistake 1

Using Bind Mounts everywhere.

---

### Mistake 2

Using Writable Layer for databases.

---

### Mistake 3

Thinking Anonymous Volumes disappear automatically.

---

### Mistake 4

Running

```bash
docker volume prune
```

on production without checking.

---

# Production Best Practices

* Databases → Named Volumes
* Development source code → Bind Mounts
* Secrets → tmpfs or secret management solutions
* Don't depend on Writable Layer
* Backup important volumes regularly

---

# Hands-on Lab

## Lab 1

Create a named volume.

Mount it.

Delete the container.

Create a new container using the same volume.

Verify the data still exists.

---

## Lab 2

Run a container using a bind mount.

Edit a file on the host.

Observe the change immediately inside the container.

---

## Lab 3

Create a tmpfs mount.

Write a file.

Stop the container.

Verify the file is gone after restarting a fresh container.

---

# Interview Questions

## Beginner

* What is a Docker Volume?
* Difference between Writable Layer and Volume?
* Why are Volumes needed?

---

## Intermediate

* Difference between Named and Anonymous Volumes?
* Difference between Bind Mount and Named Volume?
* Why do databases prefer Named Volumes?

---

## Senior

A production database container was deleted accidentally.

Data is still available.

Explain how this is possible.

Expected discussion:

* Named Volume
* Container lifecycle
* Volume independence
* Persistent storage
* Data recovery by attaching the volume to a new container

---

# Quick Revision

```text
Writable Layer

↓

Temporary

──────────────

Named Volume

↓

Production

──────────────

Anonymous Volume

↓

Automatic

──────────────

Bind Mount

↓

Development

──────────────

tmpfs

↓

RAM
```

---

> **Part 2 Complete**

In **Part 3 (Final)** we'll cover:

* Volume internals in Linux
* `/var/lib/docker/volumes`
* Volume drivers
* NFS, EFS, CIFS, Cloud storage
* Volume backup & restore
* Volume migration
* Volume performance
* Security
* SELinux/AppArmor considerations
* Production architectures
* Docker Compose volumes
* Complete troubleshooting guide
* Advanced interview questions
* Master revision sheet
* Chapter summary

# Docker Volume Internals

Earlier we learned **what** a Volume is.

Now let's learn **how Docker actually stores it**.

---

# Where are Docker Volumes Stored?

By default (Linux):

```text
/var/lib/docker/volumes/
```

Example:

```text
/var/lib/docker/

├── containers/
├── image/
├── network/
├── overlay2/
├── plugins/
├── volumes/
│
├── mysql-data/
│      └── _data/
│
├── postgres-data/
│      └── _data/
│
└── jenkins-home/
       └── _data/
```

Every Named Volume has its own directory.

Docker mounts this directory into the container.

---

# Internal Flow

Example:

```bash
docker run \
-v mysql-data:/var/lib/mysql \
mysql
```

Internally:

```text
Container

↓

/var/lib/mysql

↓

Volume

↓

mysql-data

↓

Host Directory

↓

/var/lib/docker/volumes/mysql-data/_data
```

Application thinks it's writing inside:

```text
/var/lib/mysql
```

Actually,

Linux writes into Docker's managed storage.

---

# How Mounting Works

Container:

```text
/app/data
```

↓

Docker Mount

↓

Host

```text
/var/lib/docker/volumes/data/_data
```

Linux performs a mount operation.

The application doesn't know the difference.

---

# Volume Drivers

Until now we've used the default:

```text
local
```

Docker also supports plugins called **Volume Drivers**.

Think of them as different storage backends.

Examples:

* local
* NFS
* CIFS / SMB
* Amazon EFS
* Azure Files
* Portworx
* Longhorn
* Ceph
* NetApp
* VMware vSAN

---

# Mental Model

Instead of:

```text
Container

↓

Local Disk
```

You can have:

```text
Container

↓

Docker Volume Driver

↓

Network Storage

↓

Cloud Storage
```

Docker doesn't care where the storage actually lives.

The driver handles that.

---

# NFS Volumes

Large companies often use shared storage.

Example:

```text
App Server 1

↓

NFS

↑

App Server 2
```

Both servers access the same files.

Useful for:

* Shared uploads
* Reports
* Static assets

---

# AWS EFS

In AWS,

a common production setup is:

```text
Container

↓

Docker Volume

↓

EFS Driver

↓

Amazon EFS
```

Benefits:

* Multiple EC2 instances
* Shared storage
* Automatic scaling
* Highly available

Common for:

* WordPress
* Shared uploads
* CMS
* Jenkins

---

# Kubernetes Relationship

Docker Volumes inspired Kubernetes Persistent Volumes.

Conceptually:

```text
Docker

↓

Volume

────────────

Kubernetes

↓

Persistent Volume (PV)

↓

Persistent Volume Claim (PVC)
```

If Docker Volumes are clear,

Kubernetes storage becomes much easier.

---

# Volume Backup

Question

How do we back up a Docker Volume?

One simple approach:

Create a temporary container.

Mount the volume.

Archive its contents.

Conceptually:

```text
Volume

↓

Temporary Container

↓

Archive

↓

Backup File
```

Production teams usually automate this process using backup tools or storage snapshots.

---

# Volume Restore

Restore process:

```text
Backup

↓

Temporary Container

↓

Volume

↓

Application
```

Always test restore procedures.

A backup that cannot be restored is not useful.

---

# Volume Migration

Scenario

Moving MySQL from Server A to Server B.

Typical workflow:

```text
Server A

↓

Backup Volume

↓

Transfer Backup

↓

Server B

↓

Restore Volume

↓

Start Container
```

The container image can remain the same.

Only the persistent data moves.

---

# Performance Considerations

Different storage options have different characteristics.

| Storage         | Speed              | Persistence | Typical Use                 |
| --------------- | ------------------ | ----------- | --------------------------- |
| Writable Layer  | Medium             | ❌           | Temporary runtime data      |
| Named Volume    | High               | ✅           | Databases, application data |
| Bind Mount      | Depends on host    | ✅           | Development, configs        |
| tmpfs           | Very High          | ❌           | Temporary memory-only data  |
| Network Storage | Depends on network | ✅           | Shared enterprise storage   |

---

# Why Databases Prefer Volumes

Database workloads involve frequent writes.

Using a Named Volume provides:

* Better persistence
* Cleaner lifecycle
* Easier backup
* Simpler migration

The key reason is **persistence**, not simply avoiding Copy-on-Write.

---

# Security Considerations

Volumes may contain:

* Customer data
* Database files
* API uploads
* Logs
* Secrets (if misused)

Protect them.

Production recommendations:

* Restrict host access
* Encrypt storage where appropriate
* Backup regularly
* Limit permissions
* Monitor storage usage

---

# SELinux Considerations

On SELinux-enabled systems (for example, many Red Hat-based distributions), bind mounts may require additional labeling so the container can access the files.

Example:

```bash
-v /host/path:/container/path:Z
```

or

```bash
-v /host/path:/container/path:z
```

These options adjust SELinux labels for container access.

Without proper labeling,

containers may receive **Permission Denied** errors even when Linux file permissions appear correct.

---

# Read-Only Mounts

Sometimes applications should only read files.

Example:

```bash
-v config:/etc/app:ro
```

Benefits:

* Prevent accidental modification
* Better security
* Protect configuration files

---

# Docker Compose Example

```yaml
services:

  postgres:

    image: postgres:17

    volumes:

      - postgres-data:/var/lib/postgresql/data

volumes:

  postgres-data:
```

Compose automatically creates the volume if it doesn't already exist.

---

# Real Production Architecture

```text
                    Users
                      │
                      ▼
               Application Container
                      │
                      ▼
               Docker Named Volume
                      │
                      ▼
            High-Performance Storage
                      │
                      ▼
          Daily Backup + Monitoring
```

Notice the separation:

Application lifecycle ≠ Data lifecycle

---

# Disaster Recovery Scenario

Production server crashes.

Recovery process:

```text
New Server

↓

Install Docker

↓

Restore Volume

↓

Pull Image

↓

Run Container

↓

Application Restored
```

Because data and application are separate,

recovery becomes much easier.

---

# Troubleshooting Guide

## Volume Not Mounted

Check:

```bash
docker inspect <container>
```

Look under:

```text
Mounts
```

---

## Data Missing

Questions to ask:

* Was the correct volume mounted?
* Was the container recreated without the volume?
* Was the volume accidentally removed?

---

## Permission Denied

Possible causes:

* File ownership mismatch
* Incorrect UID/GID
* SELinux labels
* Read-only mount

---

## Disk Full

Investigate:

```bash
docker volume ls

docker system df
```

Look for:

* Unused volumes
* Large database files
* Old backups

---

# Production Best Practices

✅ Use Named Volumes for stateful services.

✅ Use Bind Mounts mainly for development and selected configuration use cases.

✅ Keep containers stateless whenever possible.

✅ Back up important volumes regularly.

✅ Monitor storage growth.

✅ Test recovery procedures.

---

# Hands-on Labs

## Lab 1

Create a Named Volume.

Store data.

Delete the container.

Verify data still exists with a new container.

---

## Lab 2

Create a Bind Mount.

Edit a file on the host.

Observe the change immediately inside the container.

---

## Lab 3

Create a Read-Only Mount.

Attempt to modify a file.

Observe the permission failure.

---

## Lab 4

Inspect a running container.

Identify:

* Mount source
* Destination
* Mount type

---

# Advanced Interview Questions

## Q1

Explain the complete lifecycle of a Docker Volume.

Expected discussion:

* Creation
* Mount
* Runtime
* Container deletion
* Volume persistence
* Explicit removal

---

## Q2

Why doesn't deleting a container delete its Named Volume?

Expected discussion:

Volumes are independent Docker objects with their own lifecycle.

---

## Q3

How would you migrate a production MySQL container to another server?

Expected discussion:

* Backup volume
* Transfer backup
* Restore volume
* Pull image
* Recreate container
* Verify application

---

## Q4

When would you choose Bind Mount instead of a Named Volume?

Expected discussion:

* Local development
* Live code editing
* Configuration files
* Host-managed assets

---

# Master Revision Sheet

```text
Container
    │
    ▼
Writable Layer
    │
 Temporary

────────────────────

Named Volume
    │
 Persistent

────────────────────

Bind Mount
    │
 Host Directory

────────────────────

tmpfs
    │
 RAM Only

────────────────────

Volume Driver
    │
 Local / NFS / EFS / Cloud
```

---

# Chapter Summary

Docker Volumes solve one fundamental problem:

> **Containers are temporary, but data is not.**

Understanding this principle makes storage decisions much easier.

By mastering:

* Writable Layer
* Named Volumes
* Anonymous Volumes
* Bind Mounts
* tmpfs
* Volume Drivers
* Backup & Restore
* Security
* Performance

you now have the knowledge needed to design storage for most Docker-based production workloads.

---

# Cross References

Previous Chapter:

* Dockerfile

Next Chapter:

➡ **06 - Docker Networking**

Related Topics:

* Docker Compose
* Kubernetes Persistent Volumes
* AWS EFS
* OverlayFS
* Docker Security

---

# Golden Rules

> **Containers are disposable.**

> **Volumes are persistent.**

> **Never rely on the Writable Layer for important data.**

> **Always separate application lifecycle from data lifecycle.**

> **Backups are mandatory. Persistence is not the same as backup.**
