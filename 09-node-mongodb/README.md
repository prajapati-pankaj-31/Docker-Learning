# Chapter 09 - Node.js + MongoDB with Docker

## Purpose
This chapter demonstrates how to run a Node.js application and MongoDB together using Docker Compose in a structure that is close to a real-world development setup.

## What you will learn
- How to containerize a Node.js app
- How to run MongoDB in a container
- How to connect the app to MongoDB
- How to use Docker Compose for a real-world stack
- How to keep the build clean and efficient with a Docker ignore file

## Project structure
```text
09-node-mongodb/
├── app.js
├── package.json
├── Dockerfile
├── docker-compose.yml
├── .dockerignore
└── README.md
```

## Prerequisites
Make sure Docker Desktop is running before you start.

```bash
docker --version
docker compose version
```

### Why these commands are used
- `docker --version` confirms Docker is installed.
- `docker compose version` confirms Compose is available.

## Step 1: Create the Node.js application
Create `app.js`:
```js
const express = require('express');
const mongoose = require('mongoose');

const app = express();
const PORT = process.env.PORT || 3000;
const MONGO_URI = process.env.MONGO_URI || 'mongodb://mongo:27017/appdb';

app.use(express.json());

app.get('/', (req, res) => {
  res.send('Hello from Node.js + MongoDB Docker app');
});

app.get('/health', (req, res) => {
  res.json({ status: 'ok' });
});

mongoose.connect(MONGO_URI)
  .then(() => {
    console.log('MongoDB connected');
    app.listen(PORT, () => {
      console.log(`Server running on port ${PORT}`);
    });
  })
  .catch((err) => {
    console.error('MongoDB connection error:', err);
  });
```

### Why this code is used
- `express` creates the web server.
- `mongoose` connects the app to MongoDB.
- `MONGO_URI` uses the service name `mongo` because both containers are on the same Docker network.
- The `/health` endpoint helps verify that the app is running.

## Step 2: Create `package.json`
```json
{
  "name": "node-mongodb-docker",
  "version": "1.0.0",
  "description": "Node.js app with MongoDB using Docker",
  "main": "app.js",
  "scripts": {
    "start": "node app.js"
  },
  "dependencies": {
    "express": "^4.18.21",
    "mongoose": "^8.0.1"
  }
}
```

## Step 3: Create `Dockerfile`
```dockerfile
FROM node:18-alpine
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
EXPOSE 3000
CMD ["npm", "start"]
```

### Why this Dockerfile is used
- `FROM node:18-alpine` uses a lightweight Node base image.
- `WORKDIR /app` sets a clean working directory inside the container.
- `COPY package*.json ./` copies dependency files first for better caching.
- `RUN npm install` installs dependencies in a clean and repeatable way.
- `COPY . .` copies the application code into the image.
- `EXPOSE 3000` documents the port the app uses.
- `CMD ["npm", "start"]` defines the default command to run the app.

## Step 4: Create `.dockerignore`
```text
node_modules
npm-debug.log
.git
.env
```

### Why this file is used
It prevents unnecessary files from being sent to the Docker build context, which keeps builds faster and cleaner.

## Step 5: Create `docker-compose.yml`
```yaml
version: "3.8"
services:
  app:
    build: .
    ports:
      - "3000:3000"
    environment:
      PORT: 3000
      MONGO_URI: mongodb://mongo:27017/appdb
    depends_on:
      - mongo

  mongo:
    image: mongo:6
    ports:
      - "27017:27017"
    volumes:
      - mongo-data:/data/db

volumes:
  mongo-data:
```

### Why this Compose file is used
- `app` builds from the local Dockerfile.
- `mongo` runs the MongoDB container.
- `depends_on` ensures MongoDB starts before the app.
- `volumes` keeps database data persistent across container restarts.

## Step 6: Run the project
```bash
docker compose up --build
```

### Why this command is used
- `up` starts the services.
- `--build` rebuilds the image so the latest code is included.
- This starts both the Node app and MongoDB together.

## Step 7: Verify the application
Open your browser or run:
```bash
curl http://localhost:3000/
```

### Expected output
```text
Hello from Node.js + MongoDB Docker app
```

You can also verify the health endpoint:
```bash
curl http://localhost:3000/health
```

### Expected output
```json
{"status":"ok"}
```

## Step 8: Stop the services
```bash
docker compose down
```

### Why this command is used
It stops and removes the containers created by Compose.

## Common errors and fixes
### Error: MongoDB connection failed
Check whether MongoDB is running and whether `MONGO_URI` uses the correct service name.

### Error: Port 3000 already in use
Change the host port in the Compose file:
```yaml
ports:
  - "3001:3000"
```

### Error: `node_modules` missing
Run:
```bash
npm install
```

### Error: Docker build is slow
Ensure `.dockerignore` is present so unnecessary files are excluded.

## Best practices
- Keep your Dockerfile simple and readable.
- Use `.dockerignore` to reduce build context size.
- Use environment variables for configuration.
- Use named volumes for persistent data.
- Prefer Compose for multi-container applications.

## Summary
This chapter shows how Docker Compose can be used to run a real-world application stack with a Node.js app and MongoDB in a professional, maintainable way.
