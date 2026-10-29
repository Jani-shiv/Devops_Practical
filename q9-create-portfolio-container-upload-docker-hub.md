# Practical 9: Create a Portfolio Container and Upload into Docker Hub

## Objective
Package a static personal portfolio website into a Docker container image using Nginx and upload it to Docker Hub.

## Prerequisites
- Docker installed on your local machine or an EC2 instance.
- An active Docker Hub account.

## What We Need To Do
1. Create static portfolio HTML files.
2. Write a `Dockerfile` utilizing lightweight Nginx Alpine.
3. Authenticate with Docker Hub from the command line.
4. Build and tag the Docker image.
5. Test the container locally.
6. Push the container image to Docker Hub repository.

## How To Do It (Step-by-Step Instructions)

### Step 1: Create portfolio project folder
```bash
mkdir portfolio && cd portfolio
```

### Step 2: Create `index.html`
```html
<!DOCTYPE html>
<html>
<head>
    <title>My Portfolio</title>
</head>
<body>
    <h1>Welcome to My Portfolio</h1>
    <p>DevOps Engineer | Cloud Enthusiast</p>
</body>
</html>
```

### Step 3: Create `Dockerfile`
```dockerfile
FROM nginx:alpine
COPY index.html /usr/share/nginx/html/index.html
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

### Step 4: Login to Docker Hub
```bash
docker login -u <DOCKERHUB_USERNAME>
```
*(Enter your Docker Hub password or Personal Access Token when prompted)*

### Step 5: Build Docker image
```bash
docker build -t <DOCKERHUB_USERNAME>/portfolio:v1 .
```

### Step 6: Test container locally
```bash
docker run -d -p 8080:80 --name portfolio-test <DOCKERHUB_USERNAME>/portfolio:v1
```
Open `http://localhost:8080` in the browser. Stop and remove the test container:
```bash
docker stop portfolio-test && docker rm portfolio-test
```

### Step 7: Push image to Docker Hub
```bash
docker push <DOCKERHUB_USERNAME>/portfolio:v1
```

### Step 8: Verify on Docker Hub
Log into [hub.docker.com](https://hub.docker.com) and confirm the `portfolio` repository exists with tag `v1`.

