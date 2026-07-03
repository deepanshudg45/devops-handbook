# Docker Interview Guide

> **Module:** 11 - Docker Interview Guide (Part 1)
>
> **Difficulty:** Beginner → Senior
>
> **Purpose:** Prepare for real Docker interviews by understanding not only the answers but also what the interviewer is actually testing.

---

# Table of Contents

* How to Use This Guide
* Interview Strategy
* Beginner Questions
* Intermediate Questions
* Advanced Questions
* Senior Architecture Questions
* Scenario-Based Questions
* Rapid Fire Round
* Common Mistakes
* Final Revision Checklist

---

# How to Use This Guide

Don't memorize answers.

Instead:

```text id="q1f4vd"
Question

↓

Understand WHY

↓

Explain in Your Own Words

↓

Draw Diagram

↓

Give Real Example
```

Interviewers can easily recognize memorized answers.

They value understanding.

---

# What Interviewers Actually Test

Most Docker questions fall into one of four categories.

```text id="8m2gks"
Knowledge

↓

Understanding

↓

Practical Experience

↓

Problem Solving
```

Example:

Question:

> What is a Docker Volume?

They're not checking whether you know the definition.

They're checking whether you know:

* Why it exists
* When to use it
* What happens if you don't use it

---

# Beginner Level (0–1 Year)

These questions verify fundamentals.

---

## Q1

### What is Docker?

Expected Answer

Docker is a containerization platform that packages an application along with its dependencies into a portable container.

Interviewer is checking:

* Basic understanding
* Difference between application and container

---

## Q2

### Difference between Container and Image?

Expected Discussion

| Image      | Container          |
| ---------- | ------------------ |
| Blueprint  | Running instance   |
| Read-only  | Running process    |
| Built once | Started many times |

Example:

Dockerfile

↓

Image

↓

Container

---

## Q3

### What is a Dockerfile?

Expected Discussion

A Dockerfile is a text file containing instructions used to build a Docker image.

Mention:

* FROM
* COPY
* RUN
* CMD
* ENTRYPOINT

---

## Q4

### What is Docker Hub?

Expected Discussion

Public image registry.

Used for:

* Pulling images
* Publishing images
* Sharing images

---

## Q5

### Difference between Docker and Virtual Machine?

Expected Discussion

Docker

* Shares host kernel
* Lightweight
* Faster startup

VM

* Own operating system
* Own kernel
* Heavier

---

## Q6

### What happens when you run:

```bash id="i01"
docker run nginx
```

Expected Flow

```text id="i02"
Check Local Image

↓

Pull If Missing

↓

Create Container

↓

Create Writable Layer

↓

Create Network

↓

Start Process
```

This is one of the most common interview questions.

---

# Intermediate Level (1–3 Years)

Interviewers now expect practical knowledge.

---

## Q7

### Difference between CMD and ENTRYPOINT?

Expected Discussion

CMD

* Default command
* Easily overridden

ENTRYPOINT

* Main executable
* Makes the container behave like an executable

Best answer includes an example.

---

## Q8

### Difference between COPY and ADD?

Expected Discussion

COPY

* Copies files

ADD

* Copies files
* Can extract local archives automatically
* Can fetch remote URLs (though COPY + curl/wget is usually preferred for clarity)

Production recommendation:

Prefer COPY unless ADD provides a specific benefit.

---

## Q9

### What is a Docker Volume?

Expected Discussion

Purpose:

Persistent storage.

Mention:

* Named Volume
* Bind Mount
* Writable Layer
* tmpfs

---

## Q10

### Difference between Bind Mount and Named Volume?

Expected Discussion

Bind Mount

* Host controlled
* Excellent for development

Named Volume

* Docker managed
* Preferred for databases and persistent application data

---

## Q11

### Explain Docker Networking.

Expected Discussion

Mention:

* Network Namespace
* veth Pair
* Linux Bridge
* docker0
* NAT
* DNS

Drawing a packet flow diagram earns extra points.

---

## Q12

### Difference between EXPOSE and -p?

Expected Discussion

EXPOSE

* Documentation
* Metadata

-p

* Actual host port publishing

---

# Interview Tip

Whenever possible,

draw:

```text id="tip1"
Browser

↓

Host

↓

Docker

↓

Container
```

Visual explanations are often clearer than long verbal answers.

---

# Common Beginner Mistakes

❌ Saying Docker is a Virtual Machine.

❌ Saying Images are containers.

❌ Saying Volumes are inside containers.

❌ Saying EXPOSE publishes ports.

❌ Saying `latest` is always the newest version.

---

# Self-Evaluation

If you cannot explain these topics without looking at notes,

review:

* Docker Fundamentals
* Images
* Dockerfile
* Volumes
* Networking

These form the foundation for every advanced Docker interview.

---

> **Part 1 Complete**

In **Part 2** we'll cover:

* Advanced Docker interview questions
* Docker Compose interview questions
* Security interview questions
* Registry interview questions
* Production troubleshooting scenarios
* Whiteboard architecture questions
* Real HR + technical interview flow

---

# Advanced Level Interview Questions (3–5 Years)

At this level,

interviewers don't want definitions.

They want to know whether you understand **how Docker actually works internally**.

---

# Q13

## Explain Docker Architecture.

### What Interviewer Wants

Can you explain what happens behind the scenes?

---

### Expected Discussion

```text id="q13a"
Docker CLI

↓

Docker API

↓

Docker Daemon

↓

containerd

↓

runc

↓

Linux Kernel
```

Mention:

* CLI sends REST API requests
* dockerd manages images, networks and volumes
* containerd manages container lifecycle
* runc creates containers using OCI runtime specifications
* Linux Kernel provides namespaces and cgroups

---

### Bonus Points

Explain why Docker is called a **client-server architecture**.

---

# Q14

## What happens internally during docker run?

Expected discussion:

```text id="q14a"
Check Image

↓

Pull if Missing

↓

Create Writable Layer

↓

Create Network Namespace

↓

Create veth Pair

↓

Attach Bridge

↓

Mount Volumes

↓

Apply cgroups

↓

Apply Namespaces

↓

Start PID 1
```

This is one of the strongest answers you can give.

---

# Q15

## Why are containers lightweight?

Expected discussion:

Containers do **not** include:

* Guest OS
* Separate kernel

Instead they share:

```text id="q15a"
Host Kernel
```

Therefore:

* Faster startup
* Lower RAM usage
* Less disk usage

---

# Q16

## Explain OverlayFS.

Expected discussion:

Docker images are built from layers.

OverlayFS merges:

```text id="q16a"
Lower Layers

+

Upper Writable Layer

↓

Merged View
```

Benefits:

* Faster builds
* Layer reuse
* Smaller images

---

# Q17

## Why are Docker Images Immutable?

Expected discussion:

Images should never change.

Instead:

```text id="q17a"
Old Image

↓

New Build

↓

New Image
```

Benefits:

* Reproducibility
* Rollback
* Versioning
* Predictability

---

# Q18

## Difference Between RUN, CMD and ENTRYPOINT

Expected Discussion

RUN

↓

Build Time

CMD

↓

Default Runtime Command

ENTRYPOINT

↓

Main Executable

Example:

```dockerfile id="q18docker"
FROM alpine

RUN apk add curl

ENTRYPOINT ["curl"]

CMD ["https://example.com"]
```

---

# Q19

## Difference Between Bridge and Host Networking

Expected Discussion

Bridge

* Network Namespace
* docker0
* NAT
* Isolation

Host

* Shared host network
* No NAT
* Faster
* Port conflicts possible

Mention when to use each.

---

# Q20

## Why Isn't Host Networking Default?

Expected discussion:

Although faster,

Host networking reduces:

* Isolation
* Security
* Port flexibility

Docker chooses Bridge because it balances:

Performance

*

Security

*

Multi-container support

---

# Docker Compose Interview Questions

---

# Q21

## Why Use Docker Compose?

Expected Discussion

Without Compose:

Multiple

```bash id="q21cmd"
docker run
```

commands.

With Compose:

One

```bash id="q21cmd2"
docker compose up
```

Benefits:

* Easier management
* Declarative configuration
* Networks
* Volumes
* Service discovery

---

# Q22

## Difference Between build and image?

Expected discussion

```yaml id="q22yaml"
build:
```

↓

Build locally

```yaml id="q22yaml2"
image:
```

↓

Use existing image

---

# Q23

## What does depends_on do?

Correct answer:

Controls startup order.

Wrong answer:

"Waits until the database is ready."

Mention:

Healthchecks

↓

Retry Logic

↓

Production Best Practice

---

# Q24

## Difference Between docker compose stop and down?

stop

↓

Stops containers

down

↓

Stops

*

Removes containers

*

Removes networks

Optionally removes named volumes with `-v`

---

# Docker Security Interview Questions

---

# Q25

## Why Should Containers Run as Non-root?

Expected discussion

Least Privilege.

If application compromised,

attacker gains fewer privileges.

Mention:

```dockerfile id="q25docker"
USER app
```

---

# Q26

## What Are Linux Capabilities?

Expected discussion

Instead of giving:

```text id="q26a"
Root

↓

Everything
```

Give only required privileges.

Example:

```text id="q26b"
CAP_NET_BIND_SERVICE
```

---

# Q27

## Why is --privileged Dangerous?

Expected discussion

Container receives:

* Many capabilities
* Device access
* Reduced isolation

Production recommendation:

Avoid unless absolutely required.

---

# Q28

## What is seccomp?

Expected discussion

System Call Filtering.

```text id="q28a"
Application

↓

Syscall

↓

seccomp

↓

Allowed?

↓

Kernel
```

---

# Docker Registry Interview Questions

---

# Q29

## Difference Between Tag and Digest?

Expected discussion

Tag

↓

Mutable

Digest

↓

Immutable

Production:

Prefer digests or pinned version tags.

---

# Q30

## Why Should Production Avoid latest?

Expected discussion

`latest`

↓

Can change

↓

Unexpected deployment

↓

Different application version

Use explicit versions.

---

# Q31

## Explain docker push.

Expected discussion

```text id="q31a"
Read Manifest

↓

Compare Layers

↓

Upload Missing Layers

↓

Upload Manifest
```

---

# Q32

## Explain docker pull.

Expected discussion

```text id="q32a"
Download Manifest

↓

Reuse Local Layers

↓

Download Missing Layers

↓

Verify Digest
```

---

# Troubleshooting Questions

---

# Q33

## Container Keeps Restarting.

How Will You Debug?

Expected answer

```text id="q33a"
docker ps

↓

docker logs

↓

docker inspect

↓

Dependencies

↓

Resources

↓

Fix
```

---

# Q34

## Application Cannot Reach Database.

Expected discussion

Check:

* Network
* DNS
* Service Name
* Port
* Credentials
* Health

---

# Q35

## Container Shows Permission Denied.

Expected discussion

Check:

* UID
* GID
* Volume permissions
* SELinux/AppArmor
* Read-only mounts

---

# Whiteboard Architecture Question

Interviewer:

Design Docker architecture for:

* React
* Node.js
* Redis
* PostgreSQL
* Nginx

Expected drawing:

```text id="arch1"
Internet

↓

Nginx

↓

Frontend

↓

Backend

↓

Redis

↓

PostgreSQL
```

Mention:

* User-defined networks
* Named volumes
* Healthchecks
* Restart policies
* Secrets
* Reverse proxy
* Internal-only database

---

# Follow-up Questions

Interviewers often continue with:

* Why Redis?
* Why separate networks?
* Why Named Volume?
* Why not expose PostgreSQL?
* How will you scale Backend?
* How will you update without downtime?

Practice answering the "why," not just the "what."

---

# Common Advanced Mistakes

❌ Thinking Compose is an orchestrator like Kubernetes.

❌ Using `latest` in production.

❌ Running everything as root.

❌ Using Bind Mounts for production application code.

❌ Depending only on `depends_on`.

---

# Interview Advice

When answering:

Don't say only:

> "Docker uses namespaces."

Say:

```text id="tip2"
Namespaces

↓

Isolation

↓

Security

↓

Independent Processes

↓

Separate Networking
```

Always explain:

* What
* Why
* How
* Real Example

That is what impresses interviewers.

---

> **Part 2 Complete**

In **Part 3 (Final)** we'll cover:

* Senior (5+ years) architecture questions
* Real production design questions
* Kubernetes transition questions
* HR + technical interview flow
* 50 rapid-fire Docker questions
* Complete revision sheet
* Docker interview cheat sheet

# Docker Interview Guide

> **Module:** 11 - Docker Interview Guide (Part 3)

---

# Senior Level (5+ Years)

At senior level,

the interviewer stops asking command-based questions.

Instead they ask:

> "Design something."

or

> "Production broke."

They want to understand your decision-making.

---

# Q36

## Design a Docker Architecture for an E-commerce Application

Requirements

* React Frontend
* Node Backend
* Redis
* PostgreSQL
* Nginx
* HTTPS
* Logging
* Monitoring

---

### Expected Whiteboard

```text id="q36a"
                Internet
                    │
                    ▼
            Load Balancer
                    │
                    ▼
                 Nginx
                    │
        ┌───────────┴───────────┐
        ▼                       ▼
   React Frontend         Node Backend
                                  │
                     ┌────────────┴────────────┐
                     ▼                         ▼
                  Redis                  PostgreSQL
```

---

### Strong Answer Includes

Networking

* Separate frontend/backend networks
* Database on private network
* Only Nginx exposed

Storage

* Named volumes
* Database backups

Security

* Non-root containers
* Secrets
* Healthchecks
* Image scanning

Operations

* Centralized logs
* Monitoring
* Restart policies

---

# Q37

## How Would You Reduce Docker Image Size?

Expected discussion

* Multi-stage builds
* Smaller base image
* `.dockerignore`
* Remove build dependencies
* Copy only required artifacts
* Avoid unnecessary packages

Bonus

Mention BuildKit caching.

---

# Q38

## How Would You Secure Production Containers?

Expected discussion

Application

↓

Non-root

↓

Capabilities dropped

↓

Read-only filesystem

↓

Secrets external

↓

Minimal image

↓

Image scan

↓

Image signing

↓

Runtime monitoring

---

# Q39

## How Would You Debug a Slow Docker Build?

Expected discussion

Questions:

* Build cache invalidated?
* Dockerfile order?
* Large context?
* Missing `.dockerignore`?
* Dependencies changing?

Tools

```bash id="q39cmd"
docker history

docker build --progress=plain
```

---

# Q40

## Production Server Suddenly Runs Out of Disk Space

Expected Investigation

```text id="q40a"
df -h

↓

docker system df

↓

Images

↓

Volumes

↓

Logs

↓

Build Cache
```

Expected Fix

* Remove unused images
* Remove build cache
* Rotate logs
* Review retention policy

---

# Q41

## How Would You Migrate a Dockerized Application to Another Server?

Expected discussion

Application

↓

Push Image

↓

Backup Volume

↓

Restore Volume

↓

Pull Image

↓

Start Compose

↓

Verify

Don't say:

> "Copy container."

Containers are disposable.

Images + Data move.

---

# Q42

## Explain Docker Security End-to-End

Strong Answer

```text id="q42a"
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

Pull

↓

Runtime

↓

Monitoring
```

Mention:

* Least Privilege
* Secrets
* Rootless (where applicable)
* seccomp
* Capabilities

---

# Q43

## Why Does Kubernetes Still Need Container Images?

Expected discussion

Kubernetes does **not** build applications.

It schedules containers.

Images remain the deployment artifact.

Mention OCI image compatibility.

---

# Q44

## Explain Docker Networking Internally

Strong Answer

```text id="q44a"
Network Namespace

↓

veth Pair

↓

Linux Bridge

↓

NAT

↓

iptables

↓

Application
```

---

# Q45

## Explain Docker Storage Internally

Strong Answer

```text id="q45a"
OverlayFS

↓

Writable Layer

↓

Named Volume

↓

Host Filesystem
```

Mention:

* Copy-on-write
* Persistence
* Bind mounts
* tmpfs

---

# HR + Technical Interview Flow

Typical interview sequence:

```text id="flow1"
Resume

↓

Docker Fundamentals

↓

Dockerfile

↓

Networking

↓

Volumes

↓

Compose

↓

Security

↓

Troubleshooting

↓

Architecture

↓

Behavioral Questions
```

The deeper you go,

the more "why" questions appear.

---

# Weak Answer vs Strong Answer

Question

Why use Docker Volumes?

Weak

> "Because data persists."

Strong

> "Containers have a writable layer that disappears when the container is removed. Volumes separate the data lifecycle from the container lifecycle, making backups, upgrades, and migrations much easier."

Notice:

The second answer explains **why**.

---

Question

Why not use `latest`?

Weak

> "Because it is bad."

Strong

> "`latest` is mutable. It can point to different image versions over time, making deployments non-reproducible. Using version tags or digests ensures predictable deployments and simpler rollbacks."

---

Question

Why Multi-stage Build?

Weak

> "Smaller image."

Strong

> "It separates the build environment from the runtime environment, removes unnecessary build tools from the final image, reduces attack surface, decreases image size, and improves deployment speed."

---

# Rapid Fire (50 Questions)

Answer each in **under 30 seconds**.

### Fundamentals

1. What is Docker?
2. Image vs Container?
3. Dockerfile?
4. Docker Hub?
5. Container lifecycle?

### Images

6. Layer?
7. OverlayFS?
8. Build cache?
9. Multi-stage build?
10. BuildKit?

### Dockerfile

11. RUN?
12. CMD?
13. ENTRYPOINT?
14. COPY?
15. ADD?

### Volumes

16. Named Volume?
17. Bind Mount?
18. Anonymous Volume?
19. tmpfs?
20. Writable Layer?

### Networking

21. Bridge?
22. docker0?
23. veth?
24. Network Namespace?
25. Host Network?
26. Overlay?
27. NAT?
28. EXPOSE?
29. `-p`?
30. Docker DNS?

### Compose

31. Service?
32. `build`?
33. `image`?
34. `depends_on`?
35. Healthcheck?
36. Restart policy?

### Security

37. Non-root?
38. Capabilities?
39. seccomp?
40. AppArmor?
41. SELinux?
42. Rootless?

### Registry

43. Docker Hub?
44. ECR?
45. Tag?
46. Digest?
47. Image signing?
48. SBOM?

### Troubleshooting

49. Container restarting?
50. OOMKilled?

If you can answer all 50 confidently,

your Docker fundamentals are strong.

---

# Final Revision Flow

Before any interview, revise in this order:

```text id="rev1"
Fundamentals

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

This follows the natural flow of how Docker works.

---

# Self-Assessment Checklist

Rate yourself (1–5):

| Topic           | Score |
| --------------- | :---: |
| Docker Basics   |   ⬜   |
| Architecture    |   ⬜   |
| Images          |   ⬜   |
| Dockerfile      |   ⬜   |
| Volumes         |   ⬜   |
| Networking      |   ⬜   |
| Compose         |   ⬜   |
| Security        |   ⬜   |
| Registries      |   ⬜   |
| Troubleshooting |   ⬜   |

Any topic scoring below **4** should be revised.

---

# Interview Success Strategy

When answering:

1. Define the concept briefly.
2. Explain **why** it exists.
3. Explain **how** it works.
4. Give a production example.
5. Mention trade-offs if applicable.

Example structure:

```text id="strategy1"
What

↓

Why

↓

How

↓

Example

↓

Trade-off
```

This structure works for almost every technical interview.

---

# Docker Master Revision Sheet

```text id="master1"
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
      ├──────────► Volume
      │
      ├──────────► Network
      │
      ├──────────► Security
      │
      ▼
Compose
      │
      ▼
Production
      │
      ▼
Troubleshooting
```

---

# Final Golden Rules

> Understand concepts instead of memorizing commands.

> Always explain **why**, not just **what**.

> Containers are temporary; data is not.

> Build once, deploy everywhere.

> Production systems should be reproducible, secure, and observable.

> A calm, structured debugging process is more valuable than memorizing dozens of commands.

---

# Congratulations

You have completed the **Docker Master Handbook**.

It now contains:

* ✅ 11 complete modules
* ✅ Mental models
* ✅ Internal architecture
* ✅ Production concepts
* ✅ Security
* ✅ Registries
* ✅ Troubleshooting
* ✅ Interview preparation
* ✅ Hands-on labs
* ✅ Revision guides

This gives you a solid Docker foundation for DevOps, Platform Engineering, Cloud Engineering, and Kubernetes.

The natural next step is to build the same level of mastery in **Kubernetes**, where many of these Docker concepts become building blocks for clustered applications.

