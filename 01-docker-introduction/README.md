# Chapter 01 - Docker Introduction

## What is Docker?
Docker is a platform for building, shipping, and running applications in containers.

## Why Docker is useful
- Consistent environments across machines
- Faster setup and deployment
- Lightweight compared to virtual machines
- Easy to share applications with teammates

## Important concepts
- Image: a read-only template used to create containers
- Container: a running instance of an image
- Docker Engine: the runtime that manages containers
- Dockerfile: a file that defines how an image is built

## Basic commands to practice
```bash
docker --version
docker info
docker run hello-world
```

## Learning goal for this chapter
Understand the difference between images and containers and learn how Docker helps create repeatable development environments.

## Notes
- A container is created from an image.
- A Dockerfile is used to build custom images.
- Docker makes it easier to run the same app in development, testing, and production.
