# Practical 12: Full Stack CI/CD Pipeline using React + Node.js + MongoDB + Docker + GitHub Actions + AWS EC2

## Objective
Implement an end-to-end multi-container CI/CD pipeline that builds React (frontend) and Node.js (backend) Docker images, pushes them to Docker Hub, and automatically launches the full application stack with MongoDB on AWS EC2 using Docker Compose.

## Prerequisites
- Docker Hub account and token.
- AWS EC2 Ubuntu instance with Docker and Docker Compose installed.
- Security Group open on Port 22 (SSH), Port 80 (React UI), and Port 5000 (Node.js API).

## Directory Structure
```text
fullstack-app/
├── client/
│   ├── Dockerfile
│   └── ... (React source files)
├── server/
│   ├── Dockerfile
│   └── ... (Node.js source files)
├── docker-compose.yml
└── .github/
    └── workflows/
        └── fullstack-deploy.yml
```

## What We Need To Do
1. Create multi-stage `Dockerfile` for React and production `Dockerfile` for Node.js.
2. Define a multi-container `docker-compose.yml` file with MongoDB, API server, and Client.
3. Configure Docker and Docker Compose on AWS EC2.
4. Set up repository secrets on GitHub.
5. Create GitHub Actions workflow to build, push images, copy `docker-compose.yml` via SCP, and run `docker-compose up -d`.

## How To Do It (Step-by-Step Instructions)

### Step 1: Client Dockerfile (`client/Dockerfile`)
```dockerfile
FROM node:18-alpine AS build
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build

FROM nginx:alpine
COPY --from=build /app/build /usr/share/nginx/html
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

### Step 2: Server Dockerfile (`server/Dockerfile`)
```dockerfile
FROM node:18-alpine
WORKDIR /app
COPY package*.json ./
RUN npm install --production
COPY . .
EXPOSE 5000
CMD ["node", "index.js"]
```

### Step 3: Docker Compose configuration (`docker-compose.yml`)
```yaml
version: '3.8'

services:
  mongodb:
    image: mongo:6.0
    restart: always
    ports:
      - "27017:27017"
    volumes:
      - mongo-data:/data/db

  server:
    image: ${DOCKERHUB_USERNAME}/mern-server:latest
    restart: always
    ports:
      - "5000:5000"
    environment:
      - MONGO_URI=mongodb://mongodb:27017/appdb
    depends_on:
      - mongodb

  client:
    image: ${DOCKERHUB_USERNAME}/mern-client:latest
    restart: always
    ports:
      - "80:80"
    depends_on:
      - server

volumes:
  mongo-data:
```

### Step 4: Configure EC2 Server
```bash
ssh -i "my-key.pem" ubuntu@<EC2-PUBLIC-IP>
sudo apt update -y
sudo apt install docker.io docker-compose -y
sudo usermod -aG docker ubuntu
mkdir -p /home/ubuntu/app
```

### Step 5: Configure GitHub Secrets
Under **Settings** -> **Secrets and variables** -> **Actions**:
- `DOCKERHUB_USERNAME`: Docker Hub username.
- `DOCKERHUB_TOKEN`: Docker Hub token.
- `EC2_HOST`: EC2 Public IP address.
- `EC2_USER`: `ubuntu`
- `EC2_SSH_KEY`: Content of `.pem` private key.

### Step 6: Create GitHub Actions Workflow
Create `.github/workflows/fullstack-deploy.yml`:
```yaml
name: Full Stack CI/CD Deployment

on:
  push:
    branches: [ main ]

jobs:
  build-and-push:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Log in to Docker Hub
        uses: docker/login-action@v3
        with:
          username: ${{ secrets.DOCKERHUB_USERNAME }}
          password: ${{ secrets.DOCKERHUB_TOKEN }}

      - name: Build & Push React Client
        uses: docker/build-push-action@v5
        with:
          context: ./client
          push: true
          tags: ${{ secrets.DOCKERHUB_USERNAME }}/mern-client:latest

      - name: Build & Push Node.js Server
        uses: docker/build-push-action@v5
        with:
          context: ./server
          push: true
          tags: ${{ secrets.DOCKERHUB_USERNAME }}/mern-server:latest

  deploy-ec2:
    needs: build-and-push
    runs-on: ubuntu-latest
    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Transfer docker-compose.yml to EC2
        uses: appleboy/scp-action@v0.1.7
        with:
          host: ${{ secrets.EC2_HOST }}
          username: ${{ secrets.EC2_USER }}
          key: ${{ secrets.EC2_SSH_KEY }}
          source: "docker-compose.yml"
          target: "/home/ubuntu/app"

      - name: Pull images and launch stack on EC2
        uses: appleboy/ssh-action@v1.0.3
        with:
          host: ${{ secrets.EC2_HOST }}
          username: ${{ secrets.EC2_USER }}
          key: ${{ secrets.EC2_SSH_KEY }}
          script: |
            cd /home/ubuntu/app
            export DOCKERHUB_USERNAME=${{ secrets.DOCKERHUB_USERNAME }}
            docker-compose pull
            docker-compose down
            docker-compose up -d
```

### Step 7: Verify deployment
Commit and push code to `main`. Once finished:
- Access Client: `http://<EC2-PUBLIC-IP>`
- Access API: `http://<EC2-PUBLIC-IP>:5000`

