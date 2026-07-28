# Chapter 05 - Docker with Node.js

## Goal
Create a simple Node.js application and run it inside a Docker container.

## Project files
- `app.js`: simple HTTP server
- `package.json`: Node.js app metadata and start script
- `Dockerfile`: instructions to build the image
- `.dockerignore`: files to exclude from the Docker build context

## Build the image
```bash
docker build -t docker-node-app .
```

## Run the container
```bash
docker run -p 3000:3000 docker-node-app
```

## What you will learn
- How Dockerfiles define image build steps
- How to expose a port from a container
- How to run a Node.js app inside a container

## Practice exercise
Build and run the app locally, then open http://localhost:3000 in your browser.
