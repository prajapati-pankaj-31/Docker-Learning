# Chapter 10 - React + Node.js with Docker

## Purpose
This chapter demonstrates how to containerize a React frontend and a Node.js backend together using Docker Compose.

## What you will learn
- How to build a simple React app
- How to containerize a React frontend
- How to containerize a Node.js backend
- How to connect frontend and backend using Docker Compose
- How to expose ports correctly

## Project structure
```text
10-react-node/
├── frontend/
│   ├── package.json
│   ├── src/
│   │   └── App.jsx
│   └── Dockerfile
├── backend/
│   ├── package.json
│   ├── server.js
│   └── Dockerfile
├── docker-compose.yml
└── README.md
```

## Step 1: Create the backend
Create `backend/server.js`:
```js
const express = require('express');
const cors = require('cors');

const app = express();
const PORT = process.env.PORT || 5000;

app.use(cors());
app.use(express.json());

app.get('/api/health', (req, res) => {
  res.json({ status: 'ok', message: 'Backend is running' });
});

app.listen(PORT, () => {
  console.log(`Backend running on port ${PORT}`);
});
```

### Why this code is used
- `express` creates the server.
- `cors` allows the frontend to call the backend.
- `/api/health` is a simple health endpoint.

Create `backend/package.json`:
```json
{
  "name": "backend",
  "version": "1.0.0",
  "description": "Node.js backend for React app",
  "main": "server.js",
  "scripts": {
    "start": "node server.js"
  },
  "dependencies": {
    "cors": "^2.8.0",
    "express": "^4.18.21"
  }
}
```

Create `backend/Dockerfile`:
```dockerfile
FROM node:18-alpine
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
EXPOSE 5000
CMD ["npm", "start"]
```

## Step 2: Create the frontend
Create `frontend/package.json`:
```json
{
  "name": "frontend",
  "version": "1.0.0",
  "private": true,
  "scripts": {
    "start": "react-scripts start",
    "build": "react-scripts build"
  },
  "dependencies": {
    "react": "^18.2.0",
    "react-dom": "^18.2.0",
    "react-scripts": "5.0.1"
  }
}
```

Create `frontend/src/App.jsx`:
```jsx
import { useEffect, useState } from 'react';

function App() {
  const [message, setMessage] = useState('Loading...');

  useEffect(() => {
    fetch('http://localhost:5000/api/health')
      .then((res) => res.json())
      .then((data) => setMessage(data.message))
      .catch(() => setMessage('Backend not available'));
  }, []);

  return (
    <div style={{ padding: '2rem', fontFamily: 'Arial' }}>
      <h1>React + Node Docker App</h1>
      <p>{message}</p>
    </div>
  );
}

export default App;
```

Create `frontend/Dockerfile`:
```dockerfile
FROM node:18-alpine
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
EXPOSE 3000
CMD ["npm", "start"]
```

## Step 3: Create `docker-compose.yml`
```yaml
version: "3.8"
services:
  backend:
    build: ./backend
    ports:
      - "5000:5000"
    environment:
      PORT: 5000

  frontend:
    build: ./frontend
    ports:
      - "3000:3000"
    depends_on:
      - backend
```

## Step 4: Run the project
```bash
docker compose up --build
```

### Why this command is used
- It builds both containers.
- It starts the frontend and backend together.

## Step 5: Verify the app
Open the browser at:
```text
http://localhost:3000
```

You should see the React app loading data from the backend.

## Common errors and fixes
### Error: frontend cannot reach backend
Check that both services are running and the backend is listening on port `5000`.

### Error: port already in use
Change the host port in the Compose file:
```yaml
ports:
  - "3001:3000"
```

## Summary
This chapter shows how to containerize a basic React frontend and Node.js backend using Docker Compose.
