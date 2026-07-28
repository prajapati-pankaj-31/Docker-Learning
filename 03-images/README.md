# Chapter 03 - Docker Images

## Goal
Understand what Docker images are and how they are used to create containers.

## What is an image?
A Docker image is a read-only template that contains everything needed to run an application.

## Image vs Container
- Image: blueprint / file system snapshot
- Container: running instance of an image

## Common commands
```bash
docker images
docker pull nginx
docker run -d --name my-nginx nginx
```

## What these commands do
- `docker images`: list locally stored images
- `docker pull nginx`: download the Nginx image from Docker Hub
- `docker run -d --name my-nginx nginx`: start a container from the Nginx image

## Important concepts
- Images are built from Dockerfiles
- Images can be shared via registries such as Docker Hub
- Containers are created from images

## Practice exercise
Try pulling an image and listing it locally:
```bash
docker pull nginx
docker images
```
