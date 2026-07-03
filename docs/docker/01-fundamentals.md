# Docker Fundamentals

> **Module:** 01 - Docker Fundamentals
> **Difficulty:** Beginner
> **Estimated Reading Time:** 45–60 Minutes
> **Prerequisites:** Basic Linux Commands (Recommended)

---

# Table of Contents

* Learning Objectives
* Introduction
* What is Docker?
* Why Docker Was Created
* Problems Before Docker
* Evolution of Application Deployment
* Virtual Machines vs Containers
* What is a Container?
* What is Containerization?
* Docker Terminology
* Docker Ecosystem
* Docker Workflow
* Docker Lifecycle
* Real World Examples
* Production Use Cases
* Advantages
* Limitations
* Common Misconceptions
* Best Practices
* Mental Models
* Quick Revision
* Interview Questions
* Summary
* Next Chapter

---

# Learning Objectives

After completing this chapter, you should be able to answer:

* What is Docker?
* Why was Docker created?
* What problems does Docker solve?
* Difference between Virtual Machines and Containers
* What is Containerization?
* Why do containers start faster than VMs?
* Why are containers lightweight?
* Why is Docker not a Virtual Machine?
* What is an Image?
* What is a Container?

---

# Introduction

Docker is one of the most important technologies in modern DevOps.

Today almost every company uses Docker directly or indirectly.

Examples:

* Microservices
* Kubernetes
* CI/CD Pipelines
* Cloud Deployments
* Machine Learning
* Data Engineering

Understanding Docker properly makes learning Kubernetes much easier because Kubernetes runs **containers**, not virtual machines.

---

# What is Docker?

## Simple Definition

Docker is a platform that allows developers to package an application together with everything it needs to run.

That includes:

* Application code
* Runtime
* Libraries
* Dependencies
* Configuration
* Operating System files (user space)

This package is called a **Docker Image**.

When an image runs, it becomes a **Container**.

---

# Real Life Analogy

Imagine you built a Node.js application.

It needs:

* Node.js 22
* npm packages
* Environment variables
* Configuration files

Instead of asking everyone to install these manually, Docker packs everything into one portable package.

Think of it like:

```
Application
        +
Dependencies
        +
Runtime
        +
Configuration
        =
Docker Image
```

---

# Why Docker Was Created

Before Docker, developers often heard:

> "It works on my machine."

This was one of the biggest software deployment problems.

Example:

Developer Machine

```
Ubuntu 22

Node.js 20

MySQL 8

Works
```

Production Server

```
Ubuntu 18

Node.js 16

MySQL 5.7

Fails
```

Same code.

Different environment.

Different result.

Docker solves this by making the runtime environment consistent.

---

# Problems Before Docker

## Problem 1

Dependency Conflicts

Example:

Application A

Needs

```
Python 3.12
```

Application B

Needs

```
Python 3.8
```

Installing both on the same server becomes difficult.

Containers isolate dependencies.

---

## Problem 2

Environment Differences

Development

↓

Testing

↓

Production

Different software versions caused failures.

---

## Problem 3

Slow Deployment

Setting up a new server required:

* Install OS packages
* Install runtime
* Configure application
* Configure dependencies

This could take hours.

Docker images reduce deployment to:

```
docker pull

docker run
```

---

## Problem 4

Resource Wastage

Before containers, many companies used Virtual Machines.

Each VM required:

* Full Operating System
* Dedicated Memory
* Dedicated Storage

Very expensive.

---

# Evolution of Application Deployment

```
Physical Server

↓

Virtual Machines

↓

Containers

↓

Kubernetes
```

Docker became the bridge between traditional infrastructure and cloud-native architecture.

---

# Virtual Machines vs Containers

## Virtual Machine

```
Application

↓

Libraries

↓

Guest Operating System

↓

Hypervisor

↓

Host Operating System

↓

Hardware
```

Every VM has its own kernel and operating system.

Advantages:

* Strong isolation
* Multiple operating systems

Disadvantages:

* Heavy
* Slow startup
* High resource usage

---

## Container

```
Application

↓

Libraries

↓

Container Runtime

↓

Host Linux Kernel

↓

Hardware
```

Containers share the host kernel.

Advantages:

* Lightweight
* Fast startup
* Lower memory usage
* Higher density

---

# Comparison

| Feature      | Virtual Machine | Container         |
| ------------ | --------------- | ----------------- |
| Guest OS     | Yes             | No                |
| Own Kernel   | Yes             | No                |
| Startup Time | Minutes         | Seconds (or less) |
| Size         | GB              | MB                |
| Memory Usage | High            | Low               |
| Performance  | Lower           | Near Native       |
| Portability  | Moderate        | Excellent         |

---

# What is a Container?

A container is simply an **isolated Linux process** running on the host system.

It has:

* Its own filesystem
* Its own process namespace
* Its own network namespace
* Its own hostname
* Resource limits

But it **shares the host kernel**.

This is one of the most important concepts in Docker.

---

# What is Containerization?

Containerization is the process of packaging an application together with everything required to run it.

```
Code

+

Dependencies

+

Runtime

+

Configuration

↓

Container
```

This ensures the application behaves the same everywhere.

---

# Docker Terminology

## Docker Image

Blueprint.

Read-only.

Cannot run by itself.

---

## Docker Container

Running instance of an image.

Can start.

Can stop.

Can restart.

Can be deleted.

---

## Dockerfile

Text file containing instructions to build an image.

Example:

```
FROM node:22

COPY . .

RUN npm install

CMD ["npm","start"]
```

---

## Docker Engine

The software responsible for building and running containers.

---

## Docker Daemon

Background service that performs Docker operations.

---

## Docker CLI

Command-line interface used by users.

Example:

```
docker run nginx
```

---

## Docker Hub

Public registry used to store Docker images.

---

# Docker Workflow

```
Dockerfile

↓

docker build

↓

Image

↓

docker run

↓

Container
```

This workflow is used in almost every project.

---

# Docker Lifecycle

```
Dockerfile

↓

Build Image

↓

Store Image

↓

Run Container

↓

Stop Container

↓

Remove Container

↓

Rebuild Image
```

---

# Real World Example

Imagine an e-commerce company.

Services:

```
Frontend

Backend

Authentication

Payment

Redis

MySQL
```

Each service runs inside its own container.

Benefits:

* Independent deployment
* Easier scaling
* Better isolation

---

# Production Use Cases

Docker is widely used for:

* Microservices
* CI/CD
* Development Environments
* Testing
* API Services
* Machine Learning
* Batch Jobs
* Data Processing
* Local Kubernetes Clusters

---

# Advantages

* Consistent environment
* Faster deployments
* Better resource utilization
* Easy scaling
* Portable
* Versioned images
* Simple rollback
* Cloud-native

---

# Limitations

Docker is not a replacement for everything.

Examples:

* Shared kernel means weaker isolation than VMs
* Windows containers differ from Linux containers
* Containers are ephemeral by default
* Persistent data requires volumes

---

# Common Misconceptions

## Myth 1

Docker is a Virtual Machine.

❌ False

Docker uses Linux kernel features.

---

## Myth 2

Docker replaces Kubernetes.

❌ False

Kubernetes manages containers.

Docker creates and runs containers.

---

## Myth 3

Docker makes applications faster.

❌ Not necessarily.

Docker mainly provides consistency, portability, and isolation.

---

# Best Practices

* One application per container
* Use official base images
* Version your images
* Keep images small
* Never store secrets inside images
* Use volumes for persistent data
* Avoid using `latest` in production
* Rebuild images instead of modifying running containers

---

# Mental Models

## Mental Model 1

Think:

```
Docker

↓

Image

↓

Container
```

Not:

```
Docker

↓

Virtual Machine
```

---

## Mental Model 2

Image is like a recipe.

Container is the cooked food.

Many containers can be created from one image.

---

## Mental Model 3

```
Image

↓

Read Only

↓

Container

↓

Writable Layer
```

---

# Quick Revision

Remember these points:

* Docker packages applications with dependencies.
* Containers share the host kernel.
* Images are read-only.
* Containers are running instances of images.
* Docker solves environment consistency problems.
* Containers are lightweight because they do not contain a full operating system.

---

# Interview Questions

## Beginner

1. What is Docker?
2. Why was Docker created?
3. Difference between Image and Container?
4. Difference between Docker and Virtual Machine?
5. Why are containers lightweight?

---

## Intermediate

1. Why do containers start faster than VMs?
2. Why do containers share the host kernel?
3. What problems does Docker solve?
4. What is containerization?
5. Why is Docker important for CI/CD?

---

## Scenario

Your application works on your laptop but fails on production because of dependency differences.

How can Docker solve this problem?

Expected Answer:

Docker packages the application together with its dependencies into an image, ensuring the same environment is used across development, testing, and production.

---

# Summary

Docker is not just another tool.

It is a standardized way of packaging, distributing, and running applications consistently across different environments.

Understanding Docker Fundamentals is the foundation for everything that follows:

* Architecture
* Images
* Dockerfile
* Volumes
* Networking
* Compose
* Kubernetes

Without this foundation, advanced Docker topics become difficult to understand.

---

# Cross References

Next Chapter:

➡ **02 - Docker Architecture**

Related Topics:

* Docker Images
* Dockerfile
* Linux Namespaces
* cgroups
* Container Runtime

---

> **Golden Rule**

> **Docker is not virtualization. Docker is process isolation built on Linux kernel features.**
