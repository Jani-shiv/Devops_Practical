# Practical 11: Creating a CI/CD Pipeline using GitHub Actions, Docker, Docker Hub, and AWS EC2 (Apache2)

## Objective
Build a CI/CD pipeline using GitHub Actions that builds an Apache HTTPD container image, tags it with the immutable Git commit SHA, publishes it to Docker Hub, and deploys it on an AWS EC2 instance with reliable execution permissions and status verification.

## Prerequisites
- Docker Hub account and Personal Access Token (PAT).
- AWS EC2 instance (Ubuntu) with Docker installed and Security Group Port 80 (HTTP) and Port 22 (SSH) open.

## What We Need To Do
1. Configure Docker daemon on EC2 and verify deployment user execution.
2. Store Docker Hub and EC2 SSH credentials in GitHub Secrets.
3. Configure `Dockerfile` based on `httpd:2.4-alpine` with healthcheck.
4. Create GitHub Actions workflow to build, tag with Git SHA, push to Docker Hub, and deploy on EC2.
5. Verify live containerized deployment.

---

## How To Do It (Step-by-Step Instructions)

### Step 1: Install and Configure Docker on AWS EC2
```bash
ssh -i "my-key.pem" ubuntu@<EC2-PUBLIC-IP>
sudo apt update -y
sudo apt install -y docker.io curl
sudo usermod -aG docker ubuntu
sudo systemctl enable docker
sudo systemctl restart docker
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
COPY index.html /usr/local/apache2/htdocs/index.html
EXPOSE 80
HEALTHCHECK --interval=10s --timeout=3s --retries=3 \
  CMD wget --quiet --tries=1 --spider http://localhost/ || exit 1
```

**`index.html`**:
```html
<!DOCTYPE html>
<html>
<head><title>Docker CI/CD Apache2</title></head>
<body>
    <h1>Production App running on Docker + Apache2 via GitHub Actions</h1>
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
    outputs:
      image_tag: ${{ steps.vars.outputs.sha_short }}
    steps:
      - name: Checkout repository code
        uses: actions/checkout@v4

      - name: Calculate short commit SHA
        id: vars
        run: echo "sha_short=$(echo ${{ github.sha }} | cut -c1-7)" >> $GITHUB_OUTPUT

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
          tags: |
            ${{ secrets.DOCKERHUB_USERNAME }}/apache-web:${{ steps.vars.outputs.sha_short }}
            ${{ secrets.DOCKERHUB_USERNAME }}/apache-web:latest

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
            set -e
            IMAGE="${{ secrets.DOCKERHUB_USERNAME }}/apache-web:${{ needs.build-and-push.outputs.image_tag }}"
            echo "Pulling image: ${IMAGE}"
            sudo docker pull "${IMAGE}"

            echo "Stopping old container if present..."
            sudo docker stop apache-app || true
            sudo docker rm apache-app || true

            echo "Running new container..."
            sudo docker run -d --name apache-app --restart always -p 80:80 "${IMAGE}"

            echo "Verifying container status..."
            sleep 5
            if [ "$(sudo docker inspect -f '{{.State.Status}}' apache-app)" != "running" ]; then
              echo "Deployment failed! Container status is not running." >&2
              exit 1
            fi
            echo "Deployment verified successfully!"
```

### Step 5: Verify Deployment
Commit and push changes to branch `main`. Once finished, access `http://<EC2-PUBLIC-IP>`.
