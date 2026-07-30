# Chapter 08 - Docker Compose

## Purpose
This chapter explains how Docker Compose helps you run multi-container applications using a single configuration file.

## What is Docker Compose?
Docker Compose is a tool for defining and running multi-container Docker applications.

It is useful when your app needs:
- a web container
- a database container
- a cache container

Instead of running each container manually, you define everything in a `docker-compose.yml` file and run it with one command.

## Why Docker Compose is important
- Simplifies multi-container setup
- Makes environments reproducible
- Helps developers work with the same configuration
- Reduces manual Docker commands

## Basic structure of a Compose file
```yaml
version: "3.8"
services:
  web:
    image: nginx
    ports:
      - "8080:80"
```

### Explanation
- `version` defines the Compose file format version.
- `services` contains one or more services.
- `web` is a service name.
- `image` tells Docker which image to use.
- `ports` maps host port to container port.

## Create your first Compose file
Create a file named `docker-compose.yml` with the following content:

```yaml
version: "3.8"
services:
  web:
    image: nginx
    ports:
      - "8080:80"
```

## Run the Compose file
```bash
docker compose up -d
```

### Why this command is used
- `up` starts the services.
- `-d` runs them in detached mode (background).

## Check running services
```bash
docker compose ps
```

## View logs
```bash
docker compose logs
```

## Stop the services
```bash
docker compose down
```

### Why this command is used
- It stops and removes the containers created by Compose.

## Compose with a database example
Here is a more practical example using PostgreSQL and Redis:

```yaml
version: "3.8"
services:
  postgres:
    image: postgres
    environment:
      POSTGRES_USER: postgres
      POSTGRES_DB: review
      POSTGRES_PASSWORD: password
    ports:
      - "5432:5432"

  redis:
    image: redis
    ports:
      - "6379:6379"
```

## Important Compose concepts
### Services
A service is a containerized application component.

Example:
```yaml
services:
  web:
    image: nginx
```

### Networks
Compose creates a default network for all services so they can communicate by service name.

### Volumes
You can attach persistent storage using volumes.

```yaml
services:
  db:
    image: postgres
    volumes:
      - db-data:/var/lib/postgresql/data

volumes:
  db-data:
```

## Build a custom image with Compose
If you want to build an image from a Dockerfile, use the `build` section:

```yaml
version: "3.8"
services:
  app:
    build:
      context: .
      dockerfile: Dockerfile
    ports:
      - "3000:3000"
```

## Commands you should know
```bash
docker compose up
```
Runs services in the foreground.

```bash
docker compose up -d
```
Runs services in the background.

```bash
docker compose ps
```
Lists running services.

```bash
docker compose logs
```
Shows logs.

```bash
docker compose down
```
Stops and removes containers.

```bash
docker compose restart
```
Restarts services.

```bash
docker compose stop
```
Stops services without removing them.

```bash
docker compose rm
```
Removes stopped containers.

## Practical exercise
Create a `docker-compose.yml` file with:
- one `nginx` service
- one `postgres` service

Then run:
```bash
docker compose up -d
```

After that, check:
```bash
docker compose ps
```

## Expected output
When everything runs correctly, you will see the containers created and started by Compose.

## Common errors and fixes
### Error: docker compose command not found
Install Docker Desktop or update Docker to a recent version.

### Error: port already in use
Change the host port in the Compose file:
```yaml
ports:
  - "8081:80"
```

### Error: container exits immediately
Check the logs:
```bash
docker compose logs
```

## Summary
Docker Compose is one of the most important tools in modern Docker workflows. It allows you to define, run, and manage multi-container applications with a single YAML file.
