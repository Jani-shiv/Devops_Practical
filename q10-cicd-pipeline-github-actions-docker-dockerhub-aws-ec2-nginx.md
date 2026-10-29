# Practical 10: Creating a CI/CD Pipeline using GitHub Actions, Docker, Docker Hub, and AWS EC2 (Nginx)

## Objective
Build a CI/CD pipeline using GitHub Actions that builds a Dockerized Nginx web image on git push, publishes it to Docker Hub, and pulls and deploys the container on an AWS EC2 instance.

## Prerequisites
- Docker Hub account and Personal Access Token.
- AWS EC2 instance (Ubuntu) with Docker installed and Security Group Port 80 (HTTP) open.

## What We Need To Do
1. Install Docker on the EC2 instance and grant user privileges.
2. Store Docker Hub and EC2 SSH secrets in GitHub repository secrets.
3. Add `Dockerfile` and web assets to the repository.
4. Create GitHub Actions workflow to build, push to Docker Hub, SSH into EC2, and run the container.
5. Verify live containerized deployment via EC2 public IP.

## How To Do It (Step-by-Step Instructions)

### Step 1: Install Docker on EC2
SSH into your EC2 instance:
```bash
ssh -i "my-key.pem" ubuntu@<EC2-PUBLIC-IP>
sudo apt update -y
sudo apt install docker.io -y
sudo usermod -aG docker ubuntu
newgrp docker
sudo systemctl enable docker
sudo systemctl start docker
```

### Step 2: Configure GitHub Repository Secrets
Under **Settings** -> **Secrets and variables** -> **Actions** -> **New repository secret**:
- `DOCKERHUB_USERNAME`: Your Docker Hub username.
- `DOCKERHUB_TOKEN`: Docker Hub Personal Access Token.
- `EC2_HOST`: EC2 Public IP address.
- `EC2_USER`: `ubuntu`
- `EC2_SSH_KEY`: Content of `.pem` private key file.

### Step 3: Project Files
**`Dockerfile`**:
```dockerfile
FROM nginx:alpine
COPY . /usr/share/nginx/html
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

**`index.html`**:
```html
<!DOCTYPE html>
<html>
<head><title>Docker CI/CD Nginx</title></head>
<body>
    <h1>App running on Docker + Nginx via GitHub Actions</h1>
</body>
</html>
```

### Step 4: Create GitHub Actions Workflow
Create `.github/workflows/docker-nginx-deploy.yml`:
```yaml
name: Docker Nginx CI/CD to EC2

on:
  push:
    branches: [ main ]

jobs:
  build-and-push:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout repository code
        uses: actions/checkout@v4

      - name: Log in to Docker Hub
        uses: docker/login-action@v3
        with:
          username: ${{ secrets.DOCKERHUB_USERNAME }}
          password: ${{ secrets.DOCKERHUB_TOKEN }}

      - name: Build and Push Docker image
        uses: docker/build-push-action@v5
        with:
          context: .
          push: true
          tags: ${{ secrets.DOCKERHUB_USERNAME }}/nginx-web:latest

  deploy-to-ec2:
    needs: build-and-push
    runs-on: ubuntu-latest
    steps:
      - name: Deploy container to EC2 via SSH
        uses: appleboy/ssh-action@v1.0.3
        with:
          host: ${{ secrets.EC2_HOST }}
          username: ${{ secrets.EC2_USER }}
          key: ${{ secrets.EC2_SSH_KEY }}
          script: |
            docker pull ${{ secrets.DOCKERHUB_USERNAME }}/nginx-web:latest
            docker stop nginx-app || true
            docker rm nginx-app || true
            docker run -d --name nginx-app -p 80:80 ${{ secrets.DOCKERHUB_USERNAME }}/nginx-web:latest
```

### Step 5: Verify deployment
Commit and push code to branch `main`. When the workflow completes, open:
```text
http://<EC2-PUBLIC-IP>
```

