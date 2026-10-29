# Practical 11: Creating a CI/CD Pipeline using GitHub Actions, Docker, Docker Hub, and AWS EC2 (Apache2)

## Objective
Build a CI/CD pipeline using GitHub Actions that builds an Apache HTTPD container image on git push, publishes it to Docker Hub, and deploys it on an AWS EC2 instance.

## Prerequisites
- Docker Hub account and Personal Access Token.
- AWS EC2 instance (Ubuntu) with Docker installed and Security Group Port 80 (HTTP) open.

## What We Need To Do
1. Prepare Docker environment on AWS EC2.
2. Store Docker Hub and EC2 SSH credentials in GitHub Secrets.
3. Configure `Dockerfile` based on `httpd:2.4-alpine`.
4. Create GitHub Actions workflow to build, publish, and deploy the container.
5. Verify live containerized deployment.

## How To Do It (Step-by-Step Instructions)

### Step 1: Install Docker on EC2 (if not already installed)
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
- `DOCKERHUB_TOKEN`: Personal Access Token.
- `EC2_HOST`: EC2 Public IP address.
- `EC2_USER`: `ubuntu`
- `EC2_SSH_KEY`: Content of `.pem` private key.

### Step 3: Project Files
**`Dockerfile`**:
```dockerfile
FROM httpd:2.4-alpine
COPY . /usr/local/apache2/htdocs/
EXPOSE 80
```

**`index.html`**:
```html
<!DOCTYPE html>
<html>
<head><title>Docker CI/CD Apache2</title></head>
<body>
    <h1>App running on Docker + Apache2 via GitHub Actions</h1>
</body>
</html>
```

### Step 4: Create GitHub Actions Workflow
Create `.github/workflows/docker-apache-deploy.yml`:
```yaml
name: Docker Apache CI/CD to EC2

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
          tags: ${{ secrets.DOCKERHUB_USERNAME }}/apache-web:latest

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
            docker pull ${{ secrets.DOCKERHUB_USERNAME }}/apache-web:latest
            docker stop apache-app || true
            docker rm apache-app || true
            docker run -d --name apache-app -p 80:80 ${{ secrets.DOCKERHUB_USERNAME }}/apache-web:latest
```

### Step 5: Verify deployment
Commit and push changes to branch `main`. Once finished, open:
```text
http://<EC2-PUBLIC-IP>
```

