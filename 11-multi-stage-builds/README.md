# Chapter 11 - Docker Multi-Stage Builds

## Purpose
This chapter explains Docker multi-stage builds — a powerful feature to create smaller, more secure, and production-ready Docker images by separating build-time dependencies from runtime dependencies.

## What you will learn
- What multi-stage builds are and why they matter
- How multi-stage builds reduce image size
- How to use multiple `FROM` statements in a Dockerfile
- How to copy artifacts between stages with `COPY --from`
- Real-world examples: Node.js app optimization and React production build

---

## What are Multi-Stage Builds?

A **multi-stage build** uses multiple `FROM` statements in a single Dockerfile. Each `FROM` begins a new **stage**. You can selectively copy artifacts (compiled code, binaries, static files) from one stage to another, leaving behind unnecessary build tools and intermediate files.

```
FROM node:18 AS builder       # Stage 1: install all build tools & compile
WORKDIR /app
COPY . .
RUN npm install && npm run build

FROM node:18-alpine AS runner  # Stage 2: only runtime dependencies + built artifacts
WORKDIR /app
COPY --from=builder /app/dist ./dist
CMD ["node", "dist/server.js"]
```

Only the **final stage** is included in the final image. All previous stages are discarded.

---

## Why Use Multi-Stage Builds?

| Benefit | Description |
|---------|-------------|
| **Smaller images** | The final image contains only what's needed to run the app. No compilers, dev dependencies, or build tools. |
| **Better security** | Fewer packages means a smaller attack surface. |
| **Cleaner Dockerfiles** | No need for complex shell scripts to clean up after builds. |
| **Faster deployments** | Smaller images push/pull faster from registries. |
| **Separation of concerns** | Build stage uses a full SDK; runtime stage uses a minimal base image. |

### Size comparison (typical)
| Approach | Image Size |
|----------|-----------|
| Single-stage (full SDK) | ~1.2 GB |
| Single-stage (slim) | ~300 MB |
| Multi-stage (runtime only) | ~150 MB |
| React + Nginx multi-stage | ~23 MB (vs ~1.3 GB dev image) |

---

## Basic Syntax

```dockerfile
# Stage 1: Build stage
FROM <base-image> AS <stage-name>
...
RUN <build-commands>

# Stage 2: Runtime stage
FROM <runtime-base-image>
WORKDIR /app
COPY --from=<stage-name> <source-path> <destination-path>
CMD [<runtime-command>]
```

### Key directives
- `FROM ... AS <name>` — names a stage so you can reference it later
- `COPY --from=<name>` — copies files from a named stage (or even from an external image)
- `COPY --from=0` — references the first stage by index (not recommended; use named stages)

---

## Example 1: Node.js App — Development vs Production

This example shows a Node.js application built with multi-stage to optimize image size.

### Project structure
```
11-multi-stage-builds/node-example/
├── app.js
├── package.json
└── Dockerfile
```

### `app.js`
```js
const express = require('express');

const app = express();
const PORT = process.env.PORT || 3000;

app.get('/', (req, res) => {
  res.json({
    message: 'Hello from multi-stage build!',
    environment: process.env.NODE_ENV || 'development'
  });
});

app.listen(PORT, () => {
  console.log(`Server running on port ${PORT}`);
});
```

### `package.json`
```json
{
  "name": "node-multi-stage",
  "version": "1.0.0",
  "description": "Node.js multi-stage build example",
  "main": "app.js",
  "scripts": {
    "start": "node app.js",
    "build": "echo 'Build step complete'"
  },
  "dependencies": {
    "express": "^4.18.21"
  }
}
```

### `Dockerfile` (Multi-Stage)
```dockerfile
# ==========================================
# STAGE 1: Install dependencies (full SDK)
# ==========================================
FROM node:18 AS dependencies

WORKDIR /app

# Copy only package files first (leverage Docker cache)
COPY package*.json ./

# Install ALL dependencies (including dev)
RUN npm install

# ==========================================
# STAGE 2: Build / Prepare (optional step)
# ==========================================
FROM node:18 AS build

WORKDIR /app

# Copy node_modules from the dependencies stage
COPY --from=dependencies /app/node_modules ./node_modules

# Copy application source code
COPY . .

# Run any build steps (compilation, transpilation, etc.)
RUN npm run build

# ==========================================
# STAGE 3: Production runtime (minimal)
# ==========================================
FROM node:18-alpine AS production

WORKDIR /app

# Install only production dependencies
COPY package*.json ./
RUN npm install --only=production

# Copy application code from build stage
COPY --from=build /app/app.js ./app.js

EXPOSE 3000

# Use node directly (not npm) for better signal handling
CMD ["node", "app.js"]
```

### Build and run
```bash
# Build the multi-stage image
docker build -t node-multi-stage -f Dockerfile .
```

```bash
# Run the container
docker run -d -p 3000:3000 --name node-multi node-multi-stage
```

```bash
# Verify it works
curl http://localhost:3000
# Expected: {"message":"Hello from multi-stage build!","environment":"development"}
```

### Compare sizes
```bash
# Check the final image size
docker image ls node-multi-stage

# Compare with a single-stage build size (if you built one):
# Single-stage: node:18 base image ~ 1GB vs. node:18-alpine base ~ 126MB
```

### How it works
| Stage | Base Image | Purpose |
|-------|-----------|---------|
| `dependencies` | `node:18` | Install all npm dependencies (cached layer) |
| `build` | `node:18` | Copy code and run build steps |
| `production` | `node:18-alpine` | Copy only production deps and built code |

The final image only contains:
- Alpine Linux (minimal)
- Node.js runtime (alpine)
- Production npm packages
- The application code

---

## Example 2: React App — Production Build with Nginx

This example demonstrates building a React application and serving it with Nginx for maximum performance and minimal image size.

### Why Nginx?
- Nginx is a high-performance static file server
- Much more efficient than `npm start` (development server)
- Built React app is just static HTML/JS/CSS — no Node.js runtime needed

### `Dockerfile`
```dockerfile
# ==========================================
# STAGE 1: Build the React application
# ==========================================
FROM node:18-alpine AS build

WORKDIR /app

# Copy package files first (caching optimization)
COPY package*.json ./

# Install all dependencies (including dev)
RUN npm install

# Copy application source
COPY . .

# Build the production bundle
RUN npm run build

# ==========================================
# STAGE 2: Serve with Nginx (minimal)
# ==========================================
FROM nginx:alpine AS production

# Copy the built React app from the build stage
COPY --from=build /app/build /usr/share/nginx/html

# Copy custom nginx config (optional)
COPY nginx.conf /etc/nginx/conf.d/default.conf

# Nginx runs on port 80 by default
EXPOSE 80

# Nginx base image already has CMD to start nginx
```

### `nginx.conf`
```nginx
server {
    listen 80;
    server_name localhost;

    root /usr/share/nginx/html;
    index index.html;

    # Handle Single Page Application routing
    location / {
        try_files $uri $uri/ /index.html;
    }

    # Cache static assets
    location ~* \.(js|css|png|jpg|jpeg|gif|ico|svg)$ {
        expires 1y;
        add_header Cache-Control "public, immutable";
    }
}
```

### How to test with the React app from Chapter 10
If you have the React app from Chapter 10, copy its source files:

```bash
# Create a test directory
mkdir -p 11-multi-stage-builds/react-production/test-app
cp -r 10-react-node/frontend/* 11-multi-stage-builds/react-production/test-app/
```

Then use this Dockerfile in that directory:

```dockerfile
FROM node:18-alpine AS build
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build

FROM nginx:alpine
COPY --from=build /app/build /usr/share/nginx/html
COPY nginx.conf /etc/nginx/conf.d/default.conf
EXPOSE 80
```

### Build and run
```bash
# From inside test-app directory
docker build -t react-production -f Dockerfile .
```

```bash
# Run the production container
docker run -d -p 8080:80 --name react-prod react-production
```

```bash
# Open in browser
# http://localhost:8080
```

### Size comparison
```bash
# Check the production image size
docker image ls react-production

# Compare with the dev image from Chapter 10
# react-production (Nginx): ~23 MB
# react-dev (node:18-alpine): ~1.3 GB
```

### What changed
| Aspect | Dev (Chapter 10) | Production (Multi-stage) |
|--------|-----------------|------------------------|
| Base image | `node:18-alpine` | `nginx:alpine` |
| Image size | ~1.3 GB | ~23 MB |
| Server | `react-scripts` dev server | Nginx |
| Performance | Development only | Production-ready |
| Caching | None | Static asset caching |

---

## Advanced Multi-Stage Techniques

### 1. Copy from an external image
```dockerfile
COPY --from=nginx:alpine /etc/nginx/nginx.conf ./nginx.conf
```

### 2. Use build arguments with stages
```dockerfile
ARG NODE_VERSION=18
FROM node:${NODE_VERSION}-alpine AS build
...
```

### 3. Conditional stages with targets
```bash
# Build only up to a specific stage (for debugging)
docker build --target build -t myapp:debug .
```

### 4. Multiple architectures
```dockerfile
FROM --platform=$BUILDPLATFORM node:18-alpine AS build
...
FROM --platform=$TARGETPLATFORM nginx:alpine AS production
```

---

## Best Practices

| Practice | Why |
|----------|-----|
| **Name your stages** | Use `AS name` for clarity instead of referring by index |
| **Order stages logically** | Dependency installation → Build → Runtime |
| **Leverage cache** | Copy `package*.json` before source code |
| **Use specific tags** | `node:18-alpine` not `node:latest` |
| **Keep runtime minimal** | Use alpine or distroless images for final stage |
| **Use `.dockerignore`** | Exclude `node_modules`, `.git`, etc. from build context |
| **Test with `--target`** | Debug build stage without building the full image |

---

## Common Errors and Fixes

### Error: `COPY --from` reference not found
```text
When using COPY --from, the source path must be inside the build context.
```
**Fix:** Make sure the file you're copying exists in the previous stage and the path is correct.

### Error: Large image despite multi-stage
```text
The image is still large even with multi-stage builds.
```
**Fix:** Check if you accidentally included unnecessary files. Use `.dockerignore` and verify your runtime stage doesn't include dev dependencies.

### Error: `npm start` tries to start dev server
```text
Cannot find module 'react-scripts'
```
**Fix:** In production, use `npm run build` (build stage) and serve static files with Nginx (runtime stage).

---

## Practice Exercises

### Exercise 1: Analyze image sizes
1. Build the Node.js example from this chapter
2. Build a single-stage version using just `node:18` (no multi-stage)
3. Compare the sizes: `docker image ls`
4. Note the difference (usually 2-5x smaller)

### Exercise 2: Convert a single-stage Dockerfile
Take the `backend/Dockerfile` from Chapter 10 and convert it to a multi-stage build:
- Stage 1: Install dependencies
- Stage 2: Copy only production deps and run the app

### Exercise 3: Python multi-stage build
Create a multi-stage Dockerfile for a Python app:
- Stage 1: Use `python:3.11` to install dependencies and compile
- Stage 2: Use `python:3.11-slim` to run the app

### Exercise 4: Production React app
1. Copy the frontend from Chapter 10
2. Create a multi-stage Dockerfile with Nginx
3. Build and run it on port 8080
4. Compare image sizes

---

## Summary

This chapter covered:

- **What multi-stage builds are** — multiple `FROM` statements in a single Dockerfile
- **Why they're important** — smaller images, better security, cleaner builds
- **How to use them** — `FROM ... AS`, `COPY --from`
- **Real examples** — Node.js optimization and React production build with Nginx
- **Best practices** — stage naming, caching, minimal runtime images

Multi-stage builds are essential for production-ready Docker images. They dramatically reduce image size, improve security, and make deployments faster.

### Key commands recap
```bash
# Build with explicit target stage
docker build --target build -t myapp:debug .

# Build the final stage
docker build -t myapp:latest .

# Compare image sizes
docker image ls

# View image history (see stage layers)
docker history myapp:latest
```

