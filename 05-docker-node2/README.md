# Chapter 05 - Docker Node 2

## Purpose
Is project me aap Docker Compose, Docker Hub, image pull/push, multi-image handling aur port mapping ko samjhenge.

## Project structure
- `docker-compose.yml` - multi-service Docker setup
- `Dockerfile` - image build instructions
- `main.js` - Node.js application entry point
- `package.json` - Node.js dependencies and scripts

## What this project teaches
- Docker Hub se image pull aur push kaise karte hain
- Ek hi project me multiple images ko kaise manage karte hain
- Port mapping kaise karte hain
- Docker Compose ke through services ko kaise run karte hain

## Run the project
```bash
docker compose up -d
docker compose ps
docker compose logs
docker compose down
```

## Port mapping examples
Is project me do services use ho rahe hain:
- PostgreSQL -> `5432:5432`
- Redis -> `6379:6379`

Yeh mapping container ke andar ki port ko host machine ki port se connect karti hai.

## Docker Hub push / pull workflow
### 1. Login to Docker Hub
```bash
docker login
```

### 2. Build your own image
```bash
docker build -t <your-dockerhub-username>/docker-node2:1.0 .
```

### 3. Push image to Docker Hub
```bash
docker push <your-dockerhub-username>/docker-node2:1.0
```

### 4. Pull image from Docker Hub
```bash
docker pull <your-dockerhub-username>/docker-node2:1.0
```

## Multi-image handling with Docker Compose
Docker Compose me ek saath multiple services define kar sakte ho:
```yaml
services:
  postgres:
    image: postgres

  redis:
    image: redis
```

Har service apni alag image use karta hai aur alag container me run hota hai.

## GitHub push steps
```bash
git init
git add .
git commit -m "Initial commit"
git branch -M main
git remote add origin <your-github-repo-url>
git push -u origin main
```

## Notes
- `node_modules` aur temporary files ko Git me add mat karo.
- `.gitignore` file isliye banayi gayi hai taaki repository clean rahe.
- Yeh folder GitHub par push karne ke liye ready hai.
