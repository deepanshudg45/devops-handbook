# Real Production Troubleshooting Labs

> **Module:** 10 - Production Troubleshooting Labs (Part 1)
>
> **Difficulty:** Advanced
>
> **Estimated Reading Time:** 120–180 Minutes
>
> **Prerequisites:**
>
> * Modules 1–9

---

# Table of Contents

* Troubleshooting Mindset
* Debugging Workflow
* Golden Rules
* Incident 1 – Container Keeps Restarting
* Incident 2 – Application Exits Immediately
* Incident 3 – Port Already Allocated
* Incident 4 – Cannot Connect to Database
* Incident 5 – DNS Resolution Failure
* Incident 6 – Image Pull Failed
* Common Investigation Commands
* Interview Questions
* Quick Revision

---

# Before We Start

A common beginner mistake is trying random commands until something works.

Professional engineers follow a process.

Instead of:

```text id="q1m8xe"
Guess

↓

Run Commands

↓

Hope
```

Think:

```text id="v4r7ka"
Observe

↓

Collect Evidence

↓

Form Hypothesis

↓

Verify

↓

Fix

↓

Validate
```

This mindset prevents unnecessary downtime.

---

# Golden Troubleshooting Workflow

Whenever Docker breaks, ask these questions in order:

1. Is the container running?
2. If not, why did it stop?
3. Can I read the logs?
4. Is the application healthy?
5. Is networking working?
6. Is storage mounted correctly?
7. Are resources exhausted?
8. Has anything changed recently?

Following the same sequence every time reduces missed clues.

---

# Investigation Commands

These are your first-response commands.

Check running containers:

```bash id="cmd01"
docker ps
```

Check all containers:

```bash id="cmd02"
docker ps -a
```

View logs:

```bash id="cmd03"
docker logs <container>
```

Inspect configuration:

```bash id="cmd04"
docker inspect <container>
```

Check resource usage:

```bash id="cmd05"
docker stats
```

Open a shell:

```bash id="cmd06"
docker exec -it <container> sh
```

These six commands solve a surprisingly large percentage of Docker problems.

---

# Incident 1

## Container Keeps Restarting

Symptoms:

```text id="i1sym"
STATUS

Restarting (1) ...

Restarting (2) ...

Restarting (3) ...
```

---

## Investigation

Step 1

```bash id="i1cmd1"
docker logs app
```

Possible output:

```text id="i1log"
Error:

Cannot connect to database
```

---

Step 2

Inspect restart policy.

```bash id="i1cmd2"
docker inspect app
```

Look for:

```text id="i1restart"
RestartPolicy
```

---

Step 3

Check dependencies.

Database running?

Network correct?

Environment variables correct?

---

## Root Cause

The container isn't the problem.

The application crashes,

Docker restarts it.

Docker is doing exactly what the restart policy requested.

---

## Fix

Solve the application problem.

Don't disable restart policies just to hide symptoms.

---

# Incident 2

## Container Exits Immediately

Example:

```bash id="i2cmd1"
docker run ubuntu
```

Container exits instantly.

---

## Why?

Ubuntu has no long-running foreground process.

Docker exits when PID 1 exits.

---

## Investigation

```bash id="i2cmd2"
docker ps -a
```

Status:

```text id="i2status"
Exited (0)
```

Meaning:

The process completed successfully.

---

## Fix

Run an interactive shell:

```bash id="i2cmd3"
docker run -it ubuntu bash
```

or start an application that stays in the foreground.

---

# Incident 3

## Port Already Allocated

Error:

```text id="i3err"
Bind for 0.0.0.0:80 failed

Port already allocated
```

---

## Investigation

Find who owns the port.

Linux:

```bash id="i3cmd1"
ss -tulpn
```

or

```bash id="i3cmd2"
lsof -i :80
```

---

Check Docker:

```bash id="i3cmd3"
docker ps
```

Maybe another container already uses port 80.

---

## Fix

Options:

* Stop conflicting service.
* Choose another host port.
* Use a reverse proxy if multiple services must be exposed.

---

# Incident 4

## Backend Cannot Reach Database

Symptoms:

Application logs:

```text id="i4log"
Connection refused
```

---

## Investigation

Check database container.

```bash id="i4cmd1"
docker ps
```

Running?

---

Verify network.

```bash id="i4cmd2"
docker network inspect app-network
```

Are both containers attached?

---

Verify DNS.

Inside backend:

```bash id="i4cmd3"
ping postgres
```

If name resolution fails,

check whether both containers are on the same user-defined network.

---

Verify environment variables.

```bash id="i4cmd4"
env
```

Is:

```text id="i4env"
DB_HOST=postgres
```

correct?

---

## Root Causes

Most common:

* Wrong hostname
* Wrong network
* Database still starting
* Wrong credentials
* Firewall rules

---

# Incident 5

## DNS Resolution Failure

Application:

```text id="i5err"
Unknown host
```

---

## Investigation

Inside container:

```bash id="i5cmd1"
cat /etc/resolv.conf
```

Check:

```text id="i5dns"
127.0.0.11
```

Docker DNS should be present on a user-defined bridge network.

---

Check:

```bash id="i5cmd2"
docker network inspect app-network
```

Confirm both services belong to the same network.

---

## Fix

Use a user-defined bridge network.

Use service names,

not IP addresses.

---

# Incident 6

## Image Pull Failed

Example:

```text id="i6err"
pull access denied
```

---

## Investigation

Question 1

Authenticated?

```bash id="i6cmd1"
docker login
```

---

Question 2

Repository name correct?

Example:

```text id="i6repo"
company/payment-api
```

not

```text id="i6repo2"
company/paymentapi
```

---

Question 3

Tag exists?

```text id="i6tag"
v1.4.2
```

Maybe only:

```text id="i6tag2"
v1.4.1
```

exists.

---

Question 4

Private registry permissions?

---

## Fix

Correct:

* Registry URL
* Repository
* Authentication
* Tag

---

# Common Investigation Checklist

Whenever something breaks:

```text id="check01"
Container Running?

↓

Application Logs

↓

docker inspect

↓

Network

↓

Volumes

↓

Environment Variables

↓

Resources

↓

Host Logs
```

Never skip directly to random fixes.

---

# Hands-on Labs

## Lab 1

Create a container that exits immediately.

Determine why.

Fix it.

---

## Lab 2

Run two containers using the same host port.

Observe the error.

Resolve it.

---

## Lab 3

Break database connectivity intentionally.

Restore communication.

Document each troubleshooting step.

---

# Interview Questions

## Beginner

* Why does a container restart repeatedly?
* Why does Ubuntu exit immediately?

---

## Intermediate

* Explain how you investigate a networking issue.
* How would you troubleshoot a failed image pull?

---

## Senior

A production application stopped working after deployment.

Explain your complete investigation workflow.

Expected discussion:

* Gather symptoms
* Verify container status
* Check logs
* Inspect configuration
* Validate networking
* Validate storage
* Check recent changes
* Confirm fix
* Monitor after recovery

---

# Quick Revision

```text id="rev01"
Observe

↓

Logs

↓

Inspect

↓

Network

↓

Storage

↓

Resources

↓

Fix

↓

Validate
```

---

# Golden Rules

> Never guess.

> Read logs before changing anything.

> Verify one hypothesis at a time.

> Docker usually reports symptoms; the root cause is often inside the application or its dependencies.

> A systematic process beats random experimentation every time.

---

> **Part 1 Complete**

In **Part 2**, we'll cover more real incidents:

* OOMKilled containers
* Disk full due to Docker
* Volume permission denied
* Bind mount issues
* Read-only filesystem errors
* Container can't reach the Internet
* Slow Docker builds
* Build cache problems
* Image size explosion
* Docker daemon unavailable
* Docker socket permission issues
* Real production debugging workflow

---

# Incident 7

## Container OOMKilled

One of the most common production failures.

Symptoms:

```text id="oom1"
Container exits unexpectedly.

Status:

OOMKilled
```

---

## What is OOM?

OOM = **Out Of Memory**

The Linux kernel has an **OOM Killer**.

When the system runs out of memory,

the kernel selects one or more processes to terminate.

Docker reports this as:

```text id="oom2"
OOMKilled
```

---

## Investigation

Check:

```bash id="oomcmd1"
docker inspect app
```

Look for:

```text id="oom3"
OOMKilled: true
```

---

Monitor memory:

```bash id="oomcmd2"
docker stats
```

Observe:

* Memory usage
* CPU usage

---

Inside application logs:

```text id="oom4"
Java Heap Space

MemoryError

Killed
```

---

## Root Causes

Most common:

* Memory leak
* Container memory limit too low
* Huge file processing
* Infinite caching
* Large JVM heap

---

## Fix

* Find memory leak
* Increase memory limit (only if justified)
* Tune application memory usage
* Process data in smaller batches

Never "fix" every OOM by only increasing RAM.

---

# Incident 8

## Docker Disk Full

Symptoms

```text id="disk1"
No space left on device
```

---

## Investigation

Check Docker usage:

```bash id="diskcmd1"
docker system df
```

Check volumes:

```bash id="diskcmd2"
docker volume ls
```

Check images:

```bash id="diskcmd3"
docker image ls
```

Check host disk:

```bash id="diskcmd4"
df -h
```

---

## Common Causes

* Old images
* Unused volumes
* Build cache
* Container logs
* Large database files

---

## Cleanup Carefully

Unused images:

```bash id="diskcmd5"
docker image prune
```

Unused containers:

```bash id="diskcmd6"
docker container prune
```

Unused volumes:

```bash id="diskcmd7"
docker volume prune
```

Everything unused:

```bash id="diskcmd8"
docker system prune
```

⚠️ Never run cleanup commands blindly in production.

Always verify what will be removed.

---

# Incident 9

## Permission Denied

Example

```text id="perm1"
Permission denied
```

while writing to:

```text id="perm2"
/app/data
```

---

## Investigation

Inside container:

```bash id="permcmd1"
id
```

Check current user.

---

Check ownership:

```bash id="permcmd2"
ls -l
```

---

Check bind mount permissions.

---

If using SELinux,

verify labels are correct.

---

## Root Causes

* Wrong UID/GID
* Host ownership mismatch
* Read-only mount
* SELinux policy
* Non-root user lacks access

---

## Fix

Prefer fixing ownership and permissions rather than running the container as root.

---

# Incident 10

## Bind Mount Doesn't Show Expected Files

Example

```bash id="bind1"
-v ./app:/app
```

Container starts,

but files appear missing.

---

## Investigation

Verify:

* Correct host path
* Directory exists
* Relative path resolves correctly
* Expected working directory

Inspect mounts:

```bash id="bind2"
docker inspect app
```

Look under:

```text id="bind3"
Mounts
```

---

## Root Cause

A bind mount hides any files already present at the target path inside the image.

Example:

Image contains:

```text id="bind4"
/app/index.js
```

Host directory is empty.

After mounting,

the container sees the empty host directory.

---

# Incident 11

## Read-only Filesystem

Application log:

```text id="ro1"
Read-only file system
```

---

## Investigation

Inspect:

```bash id="rocmd1"
docker inspect app
```

Look for:

* ReadOnlyRootFilesystem
* Mount options

---

## Root Cause

Application tries to write into a read-only filesystem.

---

## Fix

Store writable data in:

* Named volumes
* Bind mounts
* tmpfs

Never write application data into immutable image layers.

---

# Incident 12

## Container Can't Reach Internet

Example

```bash id="net1"
apt update
```

Fails.

---

## Investigation

Test:

```bash id="net2"
ping 8.8.8.8
```

Works?

---

Test DNS:

```bash id="net3"
ping google.com
```

If IP works but hostname fails,

problem is DNS.

---

Inspect network:

```bash id="net4"
docker network inspect bridge
```

---

Check host connectivity.

If the host itself has no Internet access,

containers won't either.

---

## Root Causes

* Broken DNS
* Missing gateway
* Firewall
* Corporate proxy
* Host networking issue

---

# Incident 13

## Docker Build Suddenly Becomes Slow

Yesterday:

30 seconds

Today:

8 minutes

---

## Investigation

Questions:

* Cache invalidated?
* Large files added?
* `.dockerignore` missing?
* Dependencies changing every build?

---

Inspect Dockerfile order.

Example:

Bad

```dockerfile id="slow1"
COPY . .

RUN npm install
```

Every source code change invalidates dependency caching.

---

Better

```dockerfile id="slow2"
COPY package*.json .

RUN npm install

COPY . .
```

Dependency layers stay cached.

---

# Incident 14

## Image Size Explodes

Yesterday:

300 MB

Today:

2.8 GB

---

## Investigation

Questions:

* Installed build tools?
* Forgot Multi-stage Build?
* Added unnecessary packages?
* Large files copied?

---

Useful command:

```bash id="img1"
docker history image
```

Shows image layers.

---

## Fix

* Multi-stage builds
* `.dockerignore`
* Smaller base image
* Remove temporary files
* Combine related `RUN` steps where appropriate

---

# Incident 15

## Cannot Connect to Docker Daemon

Example

```text id="daemon1"
Cannot connect to the Docker daemon
```

---

## Investigation

Check service:

```bash id="daemon2"
systemctl status docker
```

Verify daemon running.

---

Check socket:

```bash id="daemon3"
ls -l /var/run/docker.sock
```

---

Verify user permissions.

---

## Common Causes

* Docker service stopped
* Wrong user group
* Socket permissions
* Daemon startup failure

---

# Incident 16

## Permission Denied Accessing Docker

Example

```text id="dockperm1"
permission denied

docker.sock
```

---

## Investigation

Check groups:

```bash id="dockperm2"
groups
```

Verify membership in:

```text id="dockperm3"
docker
```

---

If recently added to the group,

log out and back in for group membership to refresh.

---

# Production Debugging Workflow

A professional engineer follows a predictable process.

```text id="workflow1"
User Reports Issue
        │
        ▼
Reproduce Problem
        │
        ▼
Check Container Status
        │
        ▼
Read Logs
        │
        ▼
Inspect Configuration
        │
        ▼
Check Networks
        │
        ▼
Check Volumes
        │
        ▼
Check Resources
        │
        ▼
Identify Root Cause
        │
        ▼
Implement Fix
        │
        ▼
Verify Recovery
        │
        ▼
Monitor
```

The key is consistency.

---

# Production Best Practices

* Never troubleshoot by guessing.
* Read logs before changing configuration.
* Verify one hypothesis at a time.
* Keep deployments reproducible.
* Monitor resource usage continuously.
* Test backups and recovery regularly.
* Document every production incident.

---

# Hands-on Labs

## Lab 4

Create a container with a very small memory limit.

Run a memory-intensive program.

Observe the OOMKilled state.

---

## Lab 5

Fill a Docker volume with large files in a test environment.

Observe storage usage with:

```bash id="lab5"
docker system df
```

---

## Lab 6

Create a bind mount with incorrect permissions.

Diagnose and fix the issue without switching the container to run as root.

---

## Lab 7

Optimize a slow Dockerfile by improving layer caching and using a `.dockerignore` file.

Measure the build time before and after.

---

# Advanced Interview Questions

## Q1

A container restarts every few seconds.

How would you investigate?

Expected discussion:

* Container status
* Logs
* Restart policy
* Dependencies
* Healthchecks
* Root cause analysis

---

## Q2

Production server reports:

```text id="adv1"
No space left on device
```

Expected discussion:

* `docker system df`
* Images
* Volumes
* Build cache
* Logs
* Cleanup strategy

---

## Q3

Application cannot write to a mounted directory.

Expected discussion:

* User ID
* Group ID
* File ownership
* Mount type
* SELinux/AppArmor if applicable

---

## Q4

Docker build suddenly becomes very slow.

Expected discussion:

* Cache invalidation
* Build context
* Dockerfile order
* `.dockerignore`
* Dependency caching

---

# Master Troubleshooting Mental Model

```text id="mental1"
Symptoms
    │
    ▼
Evidence
    │
    ▼
Hypothesis
    │
    ▼
Verification
    │
    ▼
Root Cause
    │
    ▼
Fix
    │
    ▼
Validation
```

Always work from evidence, not assumptions.

---

# Chapter Summary

Production troubleshooting is less about memorizing commands and more about following a disciplined investigation process.

Across all incidents, the recurring themes are:

* Observe before acting.
* Collect evidence.
* Read logs.
* Verify configuration.
* Check networking, storage, and resources.
* Confirm the fix.
* Learn from every incident.

These habits are what distinguish experienced engineers from those who rely on trial and error.

---

# Cross References

Previous Chapter:

* Docker Registries & Image Distribution

Related Topics:

* Docker Networking
* Docker Volumes
* Docker Security
* BuildKit
* Linux System Administration

---

# Golden Rules

> **Symptoms are not the root cause.**

> **Logs are evidence, not opinions.**

> **Fix the underlying issue, not just the visible error.**

> **Reproduce, investigate, verify, then change.**

> **Every production incident should improve your future troubleshooting process.**

---

# Production War Stories (Part 3)

These are real-world scenarios inspired by common production incidents.

The goal isn't to memorize fixes.

The goal is to develop a systematic debugging mindset.

---

# Incident 17

## Healthcheck Always Failing

Container status:

```text
unhealthy
```

Application works when tested manually.

---

## Investigation

Check:

```bash
docker inspect app
```

Look for:

```text
Healthcheck
```

Verify:

* Correct endpoint
* Correct port
* Enough startup time
* Timeout values

Example mistake:

```yaml
healthcheck:
  test: ["CMD", "curl", "-f", "http://localhost:3000/health"]
```

Application actually listens on:

```text
8080
```

---

## Root Cause

Healthcheck configuration is incorrect.

Application itself is healthy.

---

## Lesson

A failing healthcheck does **not** always mean a failing application.

---

# Incident 18

## Container Starts Before Database

Symptoms

```text
Connection refused
```

Only during startup.

After restart,

everything works.

---

## Investigation

Database logs:

```text
Starting...

Initializing...

Ready
```

Backend logs:

```text
Connection refused
```

Backend started first.

---

## Root Cause

Application assumes database is immediately available.

---

## Fix

Implement:

* Healthchecks
* Retry logic
* Exponential backoff

Never rely only on startup order.

---

# Incident 19

## Wrong Image Deployed

Production suddenly behaves differently.

CI pipeline succeeded.

---

## Investigation

Check image:

```bash
docker inspect app
```

Verify:

```text
Image ID

Digest

Tag
```

---

Possible issue:

```text
latest
```

was deployed.

Pipeline picked a newer image than expected.

---

## Fix

Deploy immutable versions:

```text
payment-api:2.4.1
```

or

```text
payment-api@sha256:...
```

---

# Incident 20

## Configuration Changed But Container Didn't

Developer edits:

```text
.env
```

Application still uses old values.

---

## Investigation

Check:

```bash
docker compose config
```

Then inspect:

```bash
docker inspect app
```

Environment variables are loaded only when the container starts.

---

## Fix

Recreate the service:

```bash
docker compose up -d --force-recreate
```

---

# Incident 21

## Wrong Volume Mounted

Application starts.

Expected files missing.

---

## Investigation

```bash
docker inspect app
```

Check:

```text
Mounts
```

Verify:

* Source
* Destination
* Mount type

---

## Root Cause

Volume attached to the wrong target directory.

---

# Incident 22

## Container Uses 100% CPU

Symptoms

```text
docker stats
```

shows:

```text
CPU

100%
```

---

## Investigation

Inside container:

```bash
top
```

or

```bash
ps aux
```

Find:

* Infinite loop
* Runaway worker
* Recursive process
* Heavy computation

---

## Fix

Solve the application problem.

CPU is a symptom,

not the root cause.

---

# Incident 23

## Time Difference Between Host and Container

Application generates incorrect timestamps.

---

## Investigation

Inside container:

```bash
date
```

Compare with host.

Check:

* Timezone
* UTC handling
* Mounted timezone files (if intentionally used)

---

## Lesson

Store timestamps in UTC.

Convert only for display.

---

# Incident 24

## Container Works Locally But Fails in CI

---

## Investigation

Compare:

* Environment variables
* Build arguments
* Docker version
* BuildKit enabled?
* Build context
* `.dockerignore`

---

## Root Cause

Local and CI environments differ.

---

## Lesson

Keep builds reproducible.

Avoid relying on local machine state.

---

# Incident 25

## Container Escaped Expectations

Application unexpectedly accesses host resources.

---

## Investigation

Review:

```bash
docker inspect app
```

Check:

* Privileged mode
* Mounted Docker socket
* Host networking
* Capabilities
* Mounted host directories

---

## Lesson

Every additional privilege increases risk.

Use the Principle of Least Privilege.

---

# Universal Debugging Checklist

Before changing anything, answer:

* Is the container running?
* What changed recently?
* What do the logs say?
* Is networking working?
* Is storage mounted correctly?
* Is DNS resolving?
* Is the application healthy?
* Are CPU and memory normal?
* Are environment variables correct?
* Is the correct image deployed?
* Is the correct version running?

If you can't answer these questions,

you're probably debugging too early.

---

# The 5-Minute Production Workflow

When an alert arrives:

**Minute 1**

* Read the alert.
* Identify affected service.

**Minute 2**

* Check container status.
* Read logs.

**Minute 3**

* Verify networking, storage, and environment variables.

**Minute 4**

* Form one hypothesis.
* Test it.

**Minute 5**

* Apply the smallest safe fix.
* Verify recovery.
* Continue monitoring.

---

# Docker Command Cheat Sheet

```bash
docker ps
docker ps -a
docker logs <container>
docker inspect <container>
docker exec -it <container> sh
docker stats
docker top <container>
docker network ls
docker network inspect <network>
docker volume ls
docker volume inspect <volume>
docker image ls
docker history <image>
docker system df
docker events
```

---

# Final Mental Model

```text
Alert

↓

Observe

↓

Logs

↓

Inspect

↓

Network

↓

Storage

↓

Resources

↓

Configuration

↓

Hypothesis

↓

Verification

↓

Fix

↓

Validation

↓

Postmortem
```

---

# Final Golden Rules

> Never panic during production incidents.

> Logs are your first source of truth.

> Make one change at a time.

> Roll back if the risk of the next change is unknown.

> Every incident should produce documentation or automation so it doesn't happen the same way twice.

---

# Docker Mastery Complete

If you understand everything in Modules **1–10**, you have a strong foundation in:

* Docker internals
* Image creation
* Storage
* Networking
* Compose
* Security
* Registries
* Production operations
* Troubleshooting

The next logical step is **Kubernetes**, where Docker concepts scale from one host to many hosts.
