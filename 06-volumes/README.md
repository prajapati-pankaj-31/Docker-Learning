# Chapter 06 - Docker Volumes

## Purpose
This chapter explains how Docker volumes are used to persist data outside the container lifecycle.

## What is a volume?
A Docker volume is a storage mechanism that keeps data even when a container is removed or recreated.

## Why volumes are important
- Preserve data across container restarts
- Share data between containers
- Keep application data separate from container filesystem

## Common commands
```bash
docker volume ls
docker volume create my-volume
docker run -d --name my-data-container -v my-volume:/data alpine sh -c "sleep 3600"
```

## Basic workflow
1. Create a volume
2. Mount it to a container
3. Write data inside the container
4. Remove the container and verify the data still exists in the volume

## Bind mount example
```bash
docker run -d --name web-app -p 80:80 -v /path/on/host:/usr/share/nginx/html nginx
```

## Volume vs bind mount
- Volume: managed by Docker, easier for container use
- Bind mount: maps a host folder directly into the container

## Practice exercise
Create a volume, mount it into a container, and verify that data persists.
