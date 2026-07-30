# Chapter 09 - Node.js + MongoDB with Docker

## Purpose
This chapter demonstrates how to run a Node.js application and MongoDB together using Docker Compose.

## What you will learn
- How to containerize a Node.js app
- How to run MongoDB in a container
- How to connect an app to a database
- How to use Docker Compose for a real-world setup

## Project structure
```text
09-node-mongodb/
├── app.js
├── package.json
├── Dockerfile
├── docker-compose.yml
└── README.md
```

## Step 1: Create the Node.js app
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
- `WORKDIR /app` sets the app folder.
- `COPY package*.json ./` copies dependency files first for better caching.
- `RUN npm install` installs packages.
- `COPY . .` copies the app source.
- `EXPOSE 3000` documents the port.
- `CMD ["npm", "start"]` runs the app.

## Step 4: Create `docker-compose.yml`
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
- `volumes` keeps database data persistent.

## Step 5: Run the project
```bash
docker compose up --build
```

### Why this command is used
- `--build` rebuilds the app image if needed.
- It starts both the Node app and MongoDB.

## Step 6: Verify the app
Open your browser or use curl:
```bash
curl http://localhost:3000/
```

### Expected output
```text
Hello from Node.js + MongoDB Docker app
```

## Step 7: Stop the services
```bash
docker compose down
```

## Common errors and fixes
### Error: MongoDB connection failed
Check if MongoDB is running and the `MONGO_URI` matches the service name.

### Error: Port 3000 already in use
Change the host port:
```yaml
ports:
  - "3001:3000"
```

### Error: `node_modules` missing
Run:
```bash
npm install
```

## Summary
This chapter shows how Docker Compose can be used to run a real-world application stack with a Node.js app and MongoDB.
