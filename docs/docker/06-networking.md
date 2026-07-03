# Docker Networking

> **Module:** 06 - Docker Networking (Part 1)
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

---

# Table of Contents

* Learning Objectives
* Why Docker Networking Exists
* Networking Mental Model
* Linux Networking Basics
* Network Namespaces
* veth Pair
* Linux Bridge
* Docker Bridge
* Packet Flow
* NAT
* iptables
* Docker DNS
* Network Drivers Overview
* Common Mistakes
* Production Best Practices
* Hands-on Labs
* Quick Revision

---

# Learning Objectives

After completing this chapter you should be able to answer:

* Why does every container get its own IP?
* What is a Network Namespace?
* What is a veth pair?
* How does a container reach the Internet?
* What is docker0?
* How does Docker DNS work?
* Why can containers communicate using names?
* How does packet flow work inside Docker?

---

# Key Takeaways

Remember these before reading further.

* Every container gets its own network stack.
* Docker uses Linux Network Namespaces.
* Containers connect to the host using **veth pairs**.
* Docker Bridge connects containers together.
* NAT allows containers to access external networks.
* Docker provides built-in DNS for user-defined networks.

---

# Why Docker Networking Exists

Imagine two containers.

```text
Frontend

Backend
```

Question:

How will they communicate?

Without networking,

containers would be isolated processes.

Docker Networking provides:

* Container-to-container communication
* Container-to-host communication
* Internet access
* Service discovery
* Network isolation

---

# Real Production Example

Typical application:

```text
Internet
      │
      ▼
Nginx
      │
      ▼
Backend API
      │
      ▼
Redis
      │
      ▼
PostgreSQL
```

Each service runs in its own container.

Docker networking allows these containers to communicate securely.

---

# Mental Model

Think of containers like apartments.

Each apartment has:

* Its own room
* Its own door
* Its own electricity

But everyone is connected through the building's internal network.

Container

↓

Apartment

Docker Bridge

↓

Apartment Building Network

---

# Linux Networking Basics

Docker networking is built on Linux networking.

Important components:

* Network Namespace
* veth Pair
* Linux Bridge
* Routing Table
* iptables
* NAT

Docker combines these Linux features into an easy-to-use networking model.

---

# Network Namespace

Earlier we learned about Namespaces.

Network Namespace isolates:

* Network interfaces
* IP addresses
* Routing tables
* Firewall rules
* ARP table

Every container gets its own Network Namespace.

---

# Example

Container A

```text
eth0

172.17.0.2
```

Container B

```text
eth0

172.17.0.3
```

Both have an interface named **eth0**.

How?

Because each container has its own Network Namespace.

---

# Verify Yourself

Run:

```bash
docker exec -it nginx ip addr
```

You'll see something like:

```text
eth0

172.17.0.2
```

Run the same command in another container.

It also has:

```text
eth0
```

Different namespace.

Different network stack.

---

# Problem

Containers are isolated.

How do they communicate with the outside world?

Answer:

Using a **veth pair**.

---

# What is a veth Pair?

A **Virtual Ethernet Pair** is like a virtual network cable.

Imagine a physical Ethernet cable.

One end goes into your laptop.

The other goes into your router.

A veth pair works exactly the same way.

One end is inside the container.

The other end is on the host.

---

# Visual Representation

```text
Container Namespace

eth0
 │
 │
 │
vethXXXX
========================
vethYYYY
 │
 │
 │
Host Namespace
```

Packets entering one end immediately appear on the other end.

---

# Linux Bridge

Now suppose you have three containers.

Question:

How do they all connect together?

Answer:

A Linux Bridge.

Think of a bridge as a virtual network switch.

---

# Docker Bridge

Docker automatically creates a Linux bridge called:

```text
docker0
```

View it:

```bash
ip link show docker0
```

Conceptually:

```text
               docker0

          ┌─────────────┐
          │             │
Container A        Container B
      │                  │
     veth              veth
          │             │
          └─────────────┘
```

Every container attached to the default bridge network connects to `docker0`.

---

# Packet Flow

Suppose:

Container A

↓

Container B

Packet journey:

```text
Container A

↓

eth0

↓

veth Pair

↓

docker0 Bridge

↓

veth Pair

↓

eth0

↓

Container B
```

Everything happens inside the Linux kernel.

No physical switch is involved.

---

# Internet Access

Now Container A wants to reach:

```text
google.com
```

Flow:

```text
Container

↓

eth0

↓

veth

↓

docker0

↓

Host Network

↓

NAT

↓

Internet
```

Docker uses **NAT (Network Address Translation)** so containers can access external networks using the host's IP address.

---

# Why NAT?

Containers usually have private IP addresses.

Example:

```text
172.17.0.2
```

These addresses are not routable on the Internet.

NAT translates:

```text
172.17.0.2

↓

Host Public IP
```

The outside world only sees the host IP.

---

# iptables

Docker automatically programs Linux firewall rules using **iptables** (or the equivalent backend on systems using nftables).

Responsibilities include:

* NAT
* Port forwarding
* Connection tracking
* Network isolation

You usually don't configure these rules manually for basic Docker networking.

---

# Docker DNS

Question:

Why can one container access another using:

```text
backend
```

instead of:

```text
172.18.0.5
```

Because Docker provides an embedded DNS server on **user-defined bridge networks**.

When a container joins such a network,

Docker automatically registers its name.

Example:

```text
frontend

↓

backend

↓

postgres
```

No manual DNS server required.

---

# Network Drivers

Docker supports multiple network drivers.

| Driver              | Purpose                                        |
| ------------------- | ---------------------------------------------- |
| Bridge              | Default single-host networking                 |
| User-defined Bridge | Custom bridge with built-in DNS                |
| Host                | Shares host network stack                      |
| None                | No networking                                  |
| Macvlan             | Container gets MAC address on physical network |
| IPvlan              | Lightweight L2/L3 networking                   |
| Overlay             | Multi-host container networking                |

Each driver solves a different problem.

We'll study each one separately.

---

# Common Beginner Mistakes

### Mistake 1

Thinking every container can communicate automatically.

Not always.

It depends on the network configuration.

---

### Mistake 2

Using IP addresses instead of container names on user-defined bridge networks.

IPs may change.

Names are more reliable.

---

### Mistake 3

Thinking `docker0` exists inside the container.

It exists on the host.

---

### Mistake 4

Ignoring DNS and hardcoding addresses.

Always prefer service names where possible.

---

# Hands-on Labs

## Lab 1

Run two containers.

Inspect their IP addresses.

Observe that both have:

```text
eth0
```

but different IPs.

---

## Lab 2

On the host:

```bash
ip link show docker0
```

Observe the Docker bridge.

---

## Lab 3

Inspect a container:

```bash
docker inspect nginx
```

Locate:

* IP Address
* Gateway
* Network Name

---

# Quick Revision

```text
Container

↓

Network Namespace

↓

eth0

↓

veth Pair

↓

docker0

↓

Host Network

↓

NAT

↓

Internet
```

---

# Golden Rules

> Every container gets its own Network Namespace.

> Containers connect to the host using a veth pair.

> `docker0` is a Linux bridge acting like a virtual switch.

> NAT allows containers with private IPs to access external networks.

> Prefer container names over IP addresses on user-defined bridge networks.

---

> **Part 1 Complete**

In **Part 2** we will deep dive into:

* Default Bridge Network
* User-defined Bridge Network
* DNS Resolution
* Container Discovery
* Port Mapping (`-p`)
* `EXPOSE` vs `-p`
* Complete packet flow from browser to container
* Real production architecture
* Advanced troubleshooting

---

# Default Bridge Network

When Docker is installed,

it automatically creates a network called:

```text id="2od2gk"
bridge
```

You can verify it:

```bash id="e22gbo"
docker network ls
```

Example output:

```text id="5tqdbf"
NETWORK ID     NAME      DRIVER

xxxxx          bridge    bridge

xxxxx          host      host

xxxxx          none      null
```

Every container starts on this network **unless another network is specified**.

---

# Internal Architecture

```text id="f9y9l4"
Container A

↓

veth

↓

docker0 Bridge

↓

veth

↓

Container B
```

Everything is connected through:

```text id="lxzsud"
docker0
```

---

# Default IP Range

Usually Docker assigns:

```text id="v7ltx9"
172.17.0.0/16
```

Example:

```text id="2n6pkm"
Host Gateway

172.17.0.1

Container A

172.17.0.2

Container B

172.17.0.3
```

Docker automatically manages IP allocation.

---

# Verify

```bash id="a57ueo"
docker inspect nginx
```

Look for:

```text id="vbx9g8"
IPAddress
```

Example:

```text id="a5n8yu"
172.17.0.2
```

---

# Can Containers Communicate?

Yes...

but with an important limitation.

Containers on the default bridge can communicate **using IP addresses** if connectivity allows.

However,

they **do not automatically get DNS-based name resolution for container names**.

Example:

```text id="n1cf79"
172.17.0.2

↓

Works
```

```text id="twl7hv"
http://backend

↓

Fails
```

unless additional mechanisms (like legacy links) are used.

This limitation is why user-defined bridge networks are preferred.

---

# Why User-defined Bridge Exists

Docker developers realized:

Managing IP addresses manually is difficult.

Imagine:

```text id="rth4fr"
Frontend

172.17.0.5

Backend

172.17.0.7

Redis

172.17.0.8
```

Tomorrow:

Restart Backend.

Now:

```text id="dj9kyr"
172.17.0.11
```

Frontend breaks.

Bad design.

Solution:

Docker DNS.

---

# User-defined Bridge Network

Create:

```bash id="1ecjlwm"
docker network create app-network
```

Run containers:

```bash id="fdw2ym"
docker run -d \
--name backend \
--network app-network \
nginx
```

Another container:

```bash id="8r92mj"
docker run -it \
--network app-network \
busybox
```

Now:

```bash id="5jlwmf"
ping backend
```

Works.

No IP required.

---

# Internal Architecture

```text id="dyjlwm"
          app-network

        Linux Bridge

       ┌───────────────┐

Frontend

Backend

Redis

Postgres
```

Docker automatically maintains DNS records.

---

# Docker Embedded DNS

Every container on a user-defined bridge gets:

```text id="nghjlwm"
127.0.0.11
```

inside:

```text id="bjjlwm"
/etc/resolv.conf
```

Docker's embedded DNS server resolves:

```text id="ykjlwm"
backend

↓

172.18.0.3
```

Application never sees the DNS lookup.

---

# Name Resolution Flow

```text id="u2jlwm"
Frontend

↓

backend

↓

Docker DNS

↓

172.18.0.3

↓

Backend Container
```

Simple.

Reliable.

Production-ready.

---

# Why Names Are Better Than IPs

IPs change.

Names don't.

Example:

```text id="4jlwm2"
Backend

↓

Restart

↓

New IP

↓

Same Name
```

Application keeps working.

---

# Production Rule

Never write:

```text id="4cxjlwm"
http://172.18.0.4
```

Instead:

```text id="svjlwm"
http://backend:8080
```

Much safer.

---

# Port Mapping

Question

Container listens on:

```text id="5jlwmz"
80
```

Can browser access it?

No.

Container port exists inside the container's network namespace.

Need mapping.

---

# Publishing Ports

Command:

```bash id="mfjlwm"
docker run \
-p 8080:80 \
nginx
```

Meaning:

```text id="0jlwmx"
Host

8080

↓

Container

80
```

---

# Packet Flow

Browser

↓

Host

```text id="owjlwm"
localhost:8080
```

↓

iptables/NAT

↓

Container

```text id="jlwmr"
80
```

↓

Nginx

---

# Visual Flow

```text id="jlwmf"
Browser

↓

Host Port 8080

↓

Docker NAT

↓

Container Port 80

↓

Application
```

---

# Why Doesn't EXPOSE Publish?

Earlier we studied:

```dockerfile
EXPOSE 80
```

Question

Can browser access it?

No.

Because:

```text id="jlwmh"
EXPOSE

↓

Documentation Only
```

Publishing happens only through:

```bash id="jlwmj"
-p
```

---

# EXPOSE vs -p

| EXPOSE                    | -p                               |
| ------------------------- | -------------------------------- |
| Dockerfile instruction    | Runtime option                   |
| Documentation             | Actual port mapping              |
| Doesn't change networking | Creates host-to-container access |
| Optional                  | Required for external access     |

This is one of the most common interview questions.

---

# Host → Container Flow

```text id="jlwmk"
Browser

↓

Host Network

↓

iptables DNAT

↓

Container IP

↓

Application
```

Docker automatically creates the required NAT rules.

---

# Container → Internet Flow

```text id="jlwml"
Container

↓

docker0

↓

Host

↓

SNAT/MASQUERADE

↓

Internet
```

The external server only sees the host's public IP.

---

# Container → Container Flow

Same network:

```text id="jlwmm"
Container A

↓

Bridge

↓

Container B
```

No NAT required.

Just Layer 2 switching through the Linux bridge.

---

# Bridge vs User-defined Bridge

| Default Bridge                  | User-defined Bridge        |
| ------------------------------- | -------------------------- |
| Auto-created                    | User creates it            |
| Basic connectivity              | Better isolation           |
| No automatic container-name DNS | Built-in DNS               |
| Rarely used in production       | Preferred for applications |

---

# Production Architecture

Example:

```text id="jlwn1"
Internet

↓

Nginx

↓

frontend-network

↓

Frontend

↓

backend-network

↓

Backend

↓

database-network

↓

Postgres
```

Notice:

Not every container shares one network.

Multiple networks improve isolation.

---

# Multi-Network Containers

A container can join multiple networks.

Example:

```text id="jlwn2"
Frontend

↓

frontend-network

──────────────

Backend

↓

frontend-network

↓

backend-network

──────────────

Database

↓

backend-network
```

Backend acts as a bridge between application tiers.

---

# Common Beginner Mistakes

### Mistake 1

Using IP addresses.

---

### Mistake 2

Putting every container into one network.

---

### Mistake 3

Thinking EXPOSE publishes ports.

---

### Mistake 4

Publishing database ports unnecessarily.

MySQL usually doesn't need public access.

---

# Production Best Practices

* Use user-defined bridge networks.
* Communicate using service names.
* Publish only required ports.
* Keep databases on private networks.
* Separate frontend and backend traffic.

---

# Hands-on Labs

## Lab 1

Create:

```bash id="jlwn3"
docker network create demo
```

Run two containers.

Verify:

```bash id="jlwn4"
ping <container-name>
```

works.

---

## Lab 2

Publish:

```bash id="jlwn5"
-p 8080:80
```

Verify browser access.

---

## Lab 3

Remove `-p`.

Try accessing the application.

Observe why it fails from the host while still working from another container on the same network.

---

# Interview Questions

## Beginner

* What is a Bridge Network?
* Difference between EXPOSE and `-p`?
* Why does every container get its own IP?

---

## Intermediate

* Explain Docker DNS.
* Why are user-defined bridge networks preferred?
* Explain packet flow from browser to container.

---

## Senior

Design networking for:

* Frontend
* Backend
* Redis
* PostgreSQL

Requirements:

* Internet users access only the frontend.
* Backend is not directly accessible from the Internet.
* Database is private.
* Services communicate using names, not IPs.

Expected discussion:

* Multiple user-defined bridge networks
* Minimal exposed ports
* DNS-based service discovery
* Network isolation

---

# Quick Revision

```text id="jlwn6"
Container

↓

Network Namespace

↓

veth

↓

Linux Bridge

↓

Docker DNS

↓

Container Name

↓

Application
```

---

> **Part 2 Complete**

In **Part 3**, we'll cover the remaining Docker network drivers:

* Host Network
* None Network
* Macvlan
* IPvlan (L2 & L3)
* Overlay Network
* Swarm networking
* VXLAN basics
* Cross-host packet flow
* Real production architectures
* Troubleshooting
* Performance comparison
* Complete networking revision sheet

---

# Host Network

Host networking is the simplest networking mode.

Instead of creating:

* Network Namespace
* veth Pair
* Bridge
* NAT

Docker lets the container use the **host's network stack directly**.

Command:

```bash
docker run --network host nginx
```

---

# Mental Model

Bridge Mode

```text
Container

↓

Own Network Namespace

↓

veth

↓

docker0

↓

Host Network
```

Host Mode

```text
Container

↓

Host Network Stack
```

No bridge.

No veth.

No NAT.

---

# Internal Working

Normally Docker creates:

* eth0
* Private IP
* Routing table

Host networking skips all of this.

Container and host share:

* Interfaces
* IP address
* Routing table
* Network stack

---

# Why Doesn't It Have Its Own IP?

Question:

Why doesn't a Host Network container have its own IP?

Because Docker **does not create a separate Network Namespace**.

The application runs directly inside the host's networking namespace.

Host IP

↓

Container uses same IP

No virtual interface is created.

---

# Why is -p Unnecessary?

Example

```bash
docker run --network host nginx
```

Nginx listens on:

```text
80
```

Since it is already using the host network,

port 80 is already the host's port 80.

Nothing to map.

Therefore:

```bash
-p
```

has no effect in Host mode.

---

# Why is Host Mode Faster?

Bridge mode adds:

* veth traversal
* Bridge forwarding
* NAT

Host mode removes these layers.

Packet path becomes shorter.

```text
Application

↓

Host Network

↓

Internet
```

Less processing.

Lower latency.

Higher throughput.

---

# Why Isn't Host Mode Default?

Because speed isn't the only goal.

Docker also values:

* Isolation
* Security
* Multi-tenancy

Host networking sacrifices isolation.

Example:

Two containers

Both start:

```text
Port 80
```

Impossible.

Only one process can bind the same host port.

---

# Port Conflict Example

Container A

```text
nginx

↓

Port 80
```

Container B

```text
Apache

↓

Port 80
```

Second container fails.

Reason:

Both are trying to bind the same host port.

---

# Production Use Cases

Host networking is appropriate when:

* Very low latency is required
* High packet throughput is important
* Monitoring agents need host network visibility
* Certain network appliances need direct access

Examples:

* Node Exporter
* Packet capture tools
* IDS/IPS solutions
* High-performance proxies (depending on requirements)

---

# Host Network Summary

Advantages

* Fastest networking
* No NAT
* No bridge
* Lowest latency

Disadvantages

* No isolation
* Port conflicts
* Reduced security
* Harder multi-tenant deployments

---

# None Network

Sometimes applications should have **no network access at all**.

Example:

```bash
docker run --network none alpine
```

Inside container:

```bash
ip addr
```

Only shows:

```text
lo
```

(loopback interface)

No eth0.

No Internet.

No bridge.

---

# Why Use None?

Examples:

* Offline batch processing
* Cryptographic operations
* Sensitive workloads
* Security testing

Principle:

If networking isn't needed,

don't provide it.

---

# Macvlan Network

Macvlan is completely different.

Instead of using Docker's bridge,

the container appears directly on the physical network.

Each container gets:

* Its own MAC address
* Its own IP address

Just like a physical server.

---

# Architecture

```text
Physical Switch

        │

────────┼────────

 │      │      │

Host   Container A   Container B

MAC1      MAC2         MAC3
```

From the switch's perspective,

containers look like independent machines.

---

# Why Use Macvlan?

Suppose legacy software expects:

One application

↓

One physical machine

Macvlan satisfies this expectation.

Useful when applications must be reachable directly from the physical network.

---

# Advantages

* Direct network presence
* No NAT
* Independent MAC address
* Easy integration with legacy environments

---

# Disadvantages

* More complex configuration
* Switch port security may block multiple MAC addresses
* Host-to-container communication requires additional configuration

---

# Production Use Cases

* Industrial systems
* Legacy enterprise applications
* Network appliances
* Monitoring devices
* Broadcast-dependent workloads

---

# IPvlan

IPvlan is similar to Macvlan,

but lighter.

Difference:

Macvlan

↓

Each container gets a unique MAC address.

IPvlan

↓

Containers share the parent's MAC address.

Only IP addresses differ.

---

# Why Was IPvlan Created?

Large networks may have thousands of containers.

Thousands of MAC addresses increase pressure on physical switches.

IPvlan reduces this by sharing MAC addresses.

---

# IPvlan L2

Architecture:

```text
Switch

↓

Host NIC (One MAC)

↓

Containers

↓

Different IPs

↓

Same MAC
```

Layer 2 communication.

---

# IPvlan L3

Now Docker performs routing.

Instead of Layer 2 switching,

traffic is routed at Layer 3.

Useful for larger routed environments.

---

# Macvlan vs IPvlan

| Feature              | Macvlan              | IPvlan   |
| -------------------- | -------------------- | -------- |
| MAC Address          | Unique per container | Shared   |
| IP Address           | Unique               | Unique   |
| Switch Load          | Higher               | Lower    |
| Legacy Compatibility | Excellent            | Moderate |
| Scalability          | Lower                | Higher   |

---

# Overlay Network

Everything so far assumed:

One Docker Host.

Question

What if:

Host A

↓

Container A

needs to communicate with

Host B

↓

Container B?

Bridge networking can't help.

Solution:

Overlay Network.

---

# Overlay Architecture

```text
Host A

↓

Container A

↓

Overlay Tunnel

↓

Host B

↓

Container B
```

Docker creates a virtual network spanning multiple hosts.

---

# VXLAN (Concept)

Overlay networking typically uses VXLAN encapsulation.

Think:

Original packet

↓

Wrapped inside another packet

↓

Sent across the physical network

↓

Unwrapped

↓

Delivered

Applications never notice.

---

# Production Example

```text
Server 1

Frontend

────────────

Server 2

Backend

────────────

Server 3

Database
```

All containers communicate as if they are on one logical network.

---

# Why Overlay Exists

Microservices often run on multiple servers.

Overlay networking provides:

* Service discovery
* Cross-host communication
* Container mobility
* Network abstraction

---

# Performance Comparison

| Driver              | Performance | Isolation | Typical Use                           |
| ------------------- | ----------- | --------- | ------------------------------------- |
| Bridge              | High        | High      | Default single-host workloads         |
| User-defined Bridge | High        | High      | Recommended application networking    |
| Host                | Very High   | Low       | High-performance networking           |
| None                | N/A         | Maximum   | Offline workloads                     |
| Macvlan             | Very High   | Medium    | Legacy & physical network integration |
| IPvlan              | Very High   | Medium    | Large Layer 2/3 environments          |
| Overlay             | Moderate    | High      | Multi-host clusters                   |

---

# Production Decision Guide

Need one host?

↓

Bridge / User-defined Bridge

Need lowest latency?

↓

Host

Need complete network isolation?

↓

None

Need physical network visibility?

↓

Macvlan

Need many containers with lower switch overhead?

↓

IPvlan

Need multiple Docker hosts?

↓

Overlay

---

# Real Production Architecture

```text
                 Internet
                     │
             Load Balancer
                     │
        ┌────────────┴────────────┐
        │                         │
     Docker Host A           Docker Host B
        │                         │
   User-defined Bridge      User-defined Bridge
        │                         │
      Frontend                Backend
           \                   /
            \                 /
             ─── Overlay ─────
                    │
               Database Tier
```

---

# Troubleshooting Guide

## Container Can't Reach Internet

Check:

```bash
docker network inspect bridge
```

Possible causes:

* Missing gateway
* NAT rules
* Host firewall
* DNS issues

---

## Container Can't Resolve Another Container

Questions:

* Same user-defined network?
* Correct container name?
* DNS available?

Remember:

Default bridge doesn't provide automatic container-name DNS.

---

## Host Can't Access Container

Check:

* Was `-p` used?
* Is the application listening on the expected interface and port?
* Is a firewall blocking traffic?

---

## Overlay Network Problems

Investigate:

* Host-to-host connectivity
* VXLAN ports
* Cluster configuration
* Firewall rules

---

# Advanced Interview Questions

### Q1

Explain packet flow in Bridge mode.

Expected discussion:

* Network Namespace
* veth pair
* Linux bridge
* Routing
* NAT

---

### Q2

Why is Host networking faster?

Expected discussion:

* No Network Namespace
* No veth
* No bridge forwarding
* No NAT

---

### Q3

Difference between Macvlan and IPvlan?

Expected discussion:

* MAC addresses
* Switch scalability
* Layer 2 behavior
* Enterprise networking

---

### Q4

When would you choose Overlay instead of Bridge?

Expected discussion:

* Multiple Docker hosts
* Cross-host communication
* Service discovery
* Cluster networking

---

# Master Revision Sheet

```text
Bridge
↓

Single Host

────────────

User-defined Bridge
↓

DNS + Isolation

────────────

Host
↓

Maximum Performance

────────────

None
↓

No Network

────────────

Macvlan
↓

Physical Network Presence

────────────

IPvlan
↓

Shared MAC

────────────

Overlay
↓

Multi-Host Networking
```

---

# Chapter Summary

Docker networking is built on Linux networking primitives:

* Network Namespaces
* veth pairs
* Linux bridges
* Routing
* NAT
* DNS

Docker packages these into different network drivers, each designed for a specific use case.

Choosing the correct driver depends on:

* Performance requirements
* Isolation needs
* Physical network integration
* Number of hosts
* Operational complexity

---

# Cross References

Previous Chapter:

* Docker Volumes

Next Chapter:

➡ **07 - Docker Compose**

Related Topics:

* Linux Networking
* Kubernetes CNI
* VXLAN
* iptables
* Network Namespaces

---

# Golden Rules

> **Use User-defined Bridge for most application deployments.**

> **Use Host networking only when you truly need direct host networking.**

> **Never expose databases unnecessarily.**

> **Prefer service names over IP addresses.**

> **Understand packet flow before troubleshooting networking issues.**

> **Networking problems are easier to solve when you think in layers: Namespace → Interface → Bridge → Routing → NAT → Application.**
