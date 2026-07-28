# Chapter 04 - Docker Containers

## Goal
Learn what containers are, how they run, and how to manage them.

## What is a container?
A container is a running instance of a Docker image. It holds the application process and its environment.

## Container lifecycle
- Create and start: `docker run`
- List running containers: `docker ps`
- List all containers: `docker ps -a`
- Stop a container: `docker stop <container-name-or-id>`
- Remove a container: `docker rm <container-name-or-id>`

## Example
```bash
docker run -d --name my-nginx nginx
docker ps
docker stop my-nginx
docker rm my-nginx
```

## Important notes
- Containers are isolated from each other.
- They can be started, stopped, removed, and restarted independently.
- A container uses the image as its base.

## Practice exercise
Run a container, inspect it, and then stop and remove it.
