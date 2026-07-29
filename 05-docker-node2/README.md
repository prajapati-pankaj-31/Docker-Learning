# Chapter 05 - Docker Node 2

## Purpose
This project helps you understand Docker Compose, Docker Hub image push/pull, multi-image handling, and port mapping.

## Project structure
- `docker-compose.yml` - Defines multiple services and their container configuration
- `Dockerfile` - Instructions to build a custom Docker image
- `main.js` - Node.js application entry point
- `package.json` - Node.js dependencies and scripts

## What you will learn
- How to pull and push images from Docker Hub
- How to manage multiple images in one project
- How to map container ports to host ports
- How to run services using Docker Compose

## Start the project
```bash
docker compose up -d
```
Use this command to build (if needed) and start all services in the background.

```bash
docker compose ps
```
Use this command to list all running containers.

```bash
docker compose logs
```
Use this command to view logs from the running services.

```bash
docker compose down
```
Use this command to stop and remove the containers created by Compose.

## Port mapping example
This project uses two services:
- PostgreSQL -> `5432:5432`
- Redis -> `6379:6379`

This means the container port is exposed to the host machine so you can access it from your local system.

## Docker Hub workflow
### 1. Log in to Docker Hub
```bash
docker login
```
Use this command to authenticate your Docker CLI with Docker Hub.

### 2. Build a custom image
```bash
docker build -t <your-dockerhub-username>/docker-node2:1.0 .
```
Use this command to build an image from the current directory.

### 3. Push the image to Docker Hub
```bash
docker push <your-dockerhub-username>/docker-node2:1.0
```
Use this command to upload the image to Docker Hub.

### 4. Pull the image from Docker Hub
```bash
docker pull <your-dockerhub-username>/docker-node2:1.0
```
Use this command to download the image from Docker Hub to your local machine.

## Multi-image handling with Docker Compose
You can define multiple services in one Compose file:
```yaml
services:
  postgres:
    image: postgres

  redis:
    image: redis
```
Each service uses its own image and runs in its own container.

## GitHub push steps
```bash
git init
git add .
git commit -m "Initial commit"
git branch -M main
git remote add origin <your-github-repo-url>
git push -u origin main
```
Use these commands to initialize Git, add project files, commit changes, and push the repository to GitHub.

## Notes
- Do not add `node_modules` or temporary files to Git.
- The `.gitignore` file keeps the repository clean.
- This folder is ready to be pushed to GitHub.
