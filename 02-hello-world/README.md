# Chapter 02 - Hello World with Docker

## Goal
Learn how to run your first container using the official Hello World image.

## Step 1: Check Docker installation
```bash
docker --version
docker info
```

## Step 2: Run the Hello World container
```bash
docker run hello-world
```

## What happens here?
- Docker downloads the `hello-world` image from Docker Hub.
- It creates a container from that image.
- The container prints a short message showing that Docker is working.

## Important notes
- `docker run` creates and starts a container.
- If the image is not present locally, Docker pulls it automatically.
- This is the simplest way to verify Docker is installed correctly.

## Practice exercise
Try the command below and observe the output:
```bash
docker run hello-world
```
