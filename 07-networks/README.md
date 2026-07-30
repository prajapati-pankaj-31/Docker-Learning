# Chapter 07 - Docker Networks

## Purpose
This chapter moves beyond the basics and explains how Docker networking works in real-world applications, including container-to-container communication, port publishing, custom networks, and troubleshooting.

## 1. What is a Docker network?
A Docker network is a virtual communication layer that allows containers to connect with each other and with the outside world in a controlled way.

Why it matters:
- Containers need networking to communicate with each other.
- Services such as web apps, databases, and queues often run in separate containers.
- Networks help isolate services while still allowing controlled communication.

## 2. Types of Docker networks
Docker provides several network types:

- `bridge` - default network for containers on a single host
- `host` - container shares the host network directly
- `none` - container has no network access
- `overlay` - used for multi-host communication in swarm mode

### Example
```bash
docker network ls
```
This shows all existing Docker networks on your machine.

## 3. Create and inspect a custom network
```bash
docker network create app-net
docker network ls
docker network inspect app-net
```

### What each command does
- `docker network create app-net` creates a custom bridge network.
- `docker network ls` lists all available networks.
- `docker network inspect app-net` shows detailed network information such as containers connected to it and IP addresses.

## 4. Connect containers to the same network
```bash
docker run -d --name web --network app-net nginx
docker run -d --name db --network app-net postgres
```

### Why this is useful
Both containers are on the same network, so they can communicate using container names as hostnames.

## 5. Port publishing (host to container)
If you want to access a container from your host machine, you need to publish ports.

```bash
docker run -d --name web-server -p 8080:80 nginx
```

### Explanation
- `-p 8080:80` publishes container port `80` to host port `8080`.
- You can access the app in the browser at `http://localhost:8080`.

## 6. Container-to-container communication
Containers in the same network can communicate by name.

```bash
docker run -d --name web --network app-net nginx
docker run -d --name db --network app-net postgres
```

Inside the network, a container can reach another container using its name.

Example idea:
- `web` can connect to `db`
- `db` can be reached as `db` rather than using an IP address

## 7. Environment variables and network configuration
Environment variables help configure services without editing code.

```bash
docker run -d --name postgres-container \
  -e POSTGRES_USER=postgres \
  -e POSTGRES_PASSWORD=password \
  -e POSTGRES_DB=appdb \
  --network app-net \
  postgres
```

### Why this is important
- Keeps configuration flexible
- Helps avoid hardcoding sensitive values
- Makes container setup cleaner and reusable

## 8. Remove and clean up networks
```bash
docker rm -f web db
docker network rm app-net
```

### What each command does
- `docker rm -f web db` stops and removes the containers.
- `docker network rm app-net` removes the custom network.

## 9. Useful advanced commands
```bash
docker ps
docker ps -a
docker inspect <container-name>
docker logs <container-name>
```

### Explanation
- `docker ps` lists running containers.
- `docker ps -a` lists all containers including stopped ones.
- `docker inspect` shows detailed configuration of a container.
- `docker logs` displays logs for debugging.

## 10. Troubleshooting common networking issues
### Problem: container cannot reach another container
Check:
- Are both containers on the same network?
- Are you using the correct container name?
- Is the service listening on the expected port?

### Problem: port not accessible from host
Check:
- Is the container exposing the correct port?
- Is the host port already in use?
- Did you publish the port correctly using `-p`?

### Problem: Docker network not found
```bash
docker network ls
```
Use this to confirm whether the network exists.

## 11. Practical example: web + database
```bash
docker network create app-net
docker run -d --name db --network app-net -e POSTGRES_USER=postgres -e POSTGRES_PASSWORD=password postgres
docker run -d --name web --network app-net -p 8080:80 nginx
```

### What this example shows
- A custom Docker network is created.
- A database container and a web container are attached to the same network.
- The web container is accessible from the host through port `8080`.

## 12. Best practices for Docker networking
- Use custom networks for multi-container applications.
- Avoid hardcoding IP addresses when possible.
- Use container names instead of IPs for communication inside the same Docker network.
- Publish only the ports you need.
- Keep services isolated and clearly separated.

## 13. Practice exercise
Try this hands-on exercise:

```bash
docker network create demo-net
docker run -d --name web --network demo-net -p 8080:80 nginx
docker run -d --name alpine-test --network demo-net alpine sh -c "sleep 3600"
```

Then inspect the network:
```bash
docker network inspect demo-net
docker ps
docker logs web
```

## Summary
Docker networking is essential for building real-world applications. Understanding custom networks, port publishing, container communication, and debugging will make you much more confident with Docker in production-style setups.
