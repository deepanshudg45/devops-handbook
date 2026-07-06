Docker Commands Reference

Purpose: Complete Docker command reference for learning, daily work, troubleshooting, and interviews.

⸻

Table of Contents

1. Docker Information
2. Image Commands
3. Container Commands
4. Dockerfile & Build Commands
5. Volume Commands
6. Network Commands
7. Docker Compose Commands
8. Registry Commands
9. System Cleanup Commands
10. Logs & Debugging
11. Resource Monitoring
12. Security Commands
13. Backup & Restore
14. Useful One-Liners
15. Command Cheat Sheet

⸻

1. Docker Information

Check Docker Version

docker --version

Use Case: Verify installed Docker version.

⸻

Detailed Docker Information

docker version

Use Case: Check both Client and Server versions.

⸻

Docker System Information

docker info

Use Case: View storage driver, cgroup version, CPUs, memory, root directory, registry configuration, etc.

⸻

Docker Help

docker --help

Use Case: List all Docker commands.

⸻

2. Image Commands

List Images

docker images

or

docker image ls

Use Case: View locally available images.

⸻

Pull Image

docker pull nginx

Use Case: Download an image from a registry.

⸻

Build Image

docker build -t myapp:v1 .

Use Case: Build an image from a Dockerfile.

⸻

Build Without Cache

docker build --no-cache -t myapp:v1 .

Use Case: Ignore cached layers during build.

⸻

Tag Image

docker tag myapp:v1 username/myapp:v1

Use Case: Prepare image for pushing to a registry.

⸻

Push Image

docker push username/myapp:v1

Use Case: Upload an image to a registry.

⸻

Remove Image

docker rmi myapp:v1

Use Case: Delete a local image.

⸻

Inspect Image

docker inspect myapp:v1

Use Case: View image metadata.

⸻

View Image History

docker history myapp:v1

Use Case: Inspect image layers.

⸻

Save Image

docker save -o image.tar myapp:v1

Use Case: Export an image for offline transfer.

⸻

Load Image

docker load -i image.tar

Use Case: Import a previously saved image.

⸻

3. Container Commands

Run Container

docker run nginx

Use Case: Start a container.

⸻

Run in Background

docker run -d nginx

Use Case: Run services like Nginx, MySQL, Redis.

⸻

Interactive Container

docker run -it ubuntu bash

Use Case: Open a shell inside a container.

⸻

Name a Container

docker run --name web nginx

Use Case: Easier management than random names.

⸻

Publish Port

docker run -p 8080:80 nginx

Use Case: Access a container from the host.

⸻

Mount Volume

docker run -v data:/var/lib/mysql mysql

Use Case: Persist database data.

⸻

Bind Mount

docker run -v $(pwd):/app node

Use Case: Live code editing during development.

⸻

Environment Variable

docker run -e DB_HOST=mysql app

Use Case: Pass runtime configuration.

⸻

List Running Containers

docker ps

Use Case: View active containers.

⸻

List All Containers

docker ps -a

Use Case: Include stopped containers.

⸻

Stop Container

docker stop web

Use Case: Gracefully stop a running container.

⸻

Start Container

docker start web

Use Case: Restart a previously stopped container.

⸻

Restart Container

docker restart web

Use Case: Apply configuration changes.

⸻

Kill Container

docker kill web

Use Case: Immediately terminate a hung container.

⸻

Remove Container

docker rm web

Use Case: Delete a stopped container.

⸻

Force Remove

docker rm -f web

Use Case: Remove a running container.

⸻

Rename Container

docker rename old-name new-name

⸻

Pause Container

docker pause web

Use Case: Temporarily suspend execution.

⸻

Resume Container

docker unpause web

⸻

4. Dockerfile & Build Commands

Build with Specific Dockerfile

docker build -f Dockerfile.prod -t app .

⸻

Multi-stage Build Target

docker build --target production -t app .

⸻

Build Argument

docker build --build-arg NODE_VERSION=22 .

⸻

Show Build Progress

docker build --progress=plain .

⸻

5. Volume Commands

Create Volume

docker volume create data

⸻

List Volumes

docker volume ls

⸻

Inspect Volume

docker volume inspect data

⸻

Remove Volume

docker volume rm data

⸻

Remove Unused Volumes

docker volume prune

⸻

6. Network Commands

List Networks

docker network ls

⸻

Create Network

docker network create app-network

⸻

Inspect Network

docker network inspect app-network

⸻

Connect Container

docker network connect app-network web

⸻

Disconnect Container

docker network disconnect app-network web

⸻

Remove Network

docker network rm app-network

⸻

7. Docker Compose Commands

Start Services

docker compose up

⸻

Start in Background

docker compose up -d

⸻

Stop Services

docker compose stop

⸻

Remove Stack

docker compose down

⸻

Remove Including Volumes

docker compose down -v

⸻

Restart Services

docker compose restart

⸻

View Running Services

docker compose ps

⸻

View Logs

docker compose logs

⸻

Follow Logs

docker compose logs -f

⸻

Execute Shell

docker compose exec app sh

⸻

Scale Service

docker compose up --scale backend=3

⸻

8. Registry Commands

Login

docker login

⸻

Logout

docker logout

⸻

Push Image

docker push username/app:v1

⸻

Pull Image

docker pull username/app:v1

⸻

9. System Cleanup Commands

Disk Usage

docker system df

⸻

Remove Unused Images

docker image prune

⸻

Remove Stopped Containers

docker container prune

⸻

Remove Unused Networks

docker network prune

⸻

Remove Everything Unused

docker system prune

⸻

Remove Everything Including Volumes

docker system prune -a --volumes

⚠️ Use with caution. This removes unused images, containers, networks, build cache, and volumes.

⸻

10. Logs & Debugging

View Logs

docker logs web

⸻

Follow Logs

docker logs -f web

⸻

Last 100 Lines

docker logs --tail 100 web

⸻

Execute Shell

docker exec -it web sh

⸻

Execute Bash

docker exec -it ubuntu bash

⸻

Inspect Container

docker inspect web

⸻

View Running Processes

docker top web

⸻

View Events

docker events

⸻

11. Resource Monitoring

Live Resource Usage

docker stats

⸻

Single Container

docker stats web

⸻

12. Security Commands

Run as Different User

docker run --user 1000:1000 app

⸻

Read-only Filesystem

docker run --read-only app

⸻

Drop All Capabilities

docker run --cap-drop ALL app

⸻

Add Required Capability

docker run --cap-add NET_BIND_SERVICE app

⸻

13. Backup & Restore

Backup Volume

docker run --rm \
-v data:/source \
-v $(pwd):/backup \
alpine \
tar czf /backup/data.tar.gz -C /source .

⸻

Restore Volume

docker run --rm \
-v data:/target \
-v $(pwd):/backup \
alpine \
tar xzf /backup/data.tar.gz -C /target

⸻

14. Useful One-Liners

Remove All Stopped Containers

docker rm $(docker ps -aq)

⸻

Remove Dangling Images

docker image prune

⸻

Stop All Running Containers

docker stop $(docker ps -q)

⸻

Remove All Containers

docker rm -f $(docker ps -aq)

⸻

Show Container IP

docker inspect -f '{{range.NetworkSettings.Networks}}{{.IPAddress}}{{end}}' web

⸻

15. Command Cheat Sheet

Task	Command
Build Image	docker build -t app .
Run Container	docker run app
Background Run	docker run -d app
List Containers	docker ps
List Images	docker images
View Logs	docker logs app
Execute Shell	docker exec -it app sh
Inspect	docker inspect app
Monitor Resources	docker stats
Create Volume	docker volume create data
Create Network	docker network create app-net
Start Compose	docker compose up -d
Stop Compose	docker compose down
Push Image	docker push user/app:v1
Pull Image	docker pull user/app:v1
Cleanup	docker system prune

⸻

Best Practices

* Use explicit image tags instead of latest in production.
* Prefer named volumes for persistent data.
* Use bind mounts primarily for local development.
* Always inspect logs before restarting containers.
* Avoid docker system prune on production without reviewing what will be removed.
* Run containers as non-root whenever possible.
* Use docker compose for multi-container applications.
* Scan and update images regularly.
* Use docker inspect before making assumptions about configuration.
* Keep this file as your daily command reference.
