# Chapter 07 - Docker Networks

## Purpose
This chapter explains how Docker networks let containers communicate with each other and with the host.

## What is a Docker network?
A Docker network is a communication bridge that connects containers in a controlled and isolated way.

## Why networks are important
- Allow containers to talk to each other
- Enable service-to-service communication
- Help separate application components logically

## Common commands
```bash
docker network ls
docker network create my-network
docker run -d --name web --network my-network nginx
docker run -d --name db --network my-network postgres
```

## Basic workflow
1. Create a network
2. Connect containers to the same network
3. Use container names as hostnames for communication

## Default networks
Docker provides default networks such as:
- `bridge` - default network for containers
- `host` - shares the host network directly
- `none` - disables networking

## Example
```bash
docker network create app-net
docker run -d --name web --network app-net nginx
docker run -d --name db --network app-net postgres
```

## Practice exercise
Create a custom network, connect two containers to it, and observe that they can communicate.
