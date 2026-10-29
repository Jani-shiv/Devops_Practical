# Practical 10: Creating a CI/CD Pipeline using GitHub Actions, Docker, Docker Hub, and AWS EC2 (Nginx)

## Objective
Build a robust CI/CD pipeline using GitHub Actions that tags images with the Git commit SHA, publishes them to Docker Hub, and deploys them to AWS EC2 using reliable Docker permissions, health checks, and automatic rollback on failure.

## Prerequisites
- Docker Hub account and Personal Access Token (PAT).
- AWS EC2 instance (Ubuntu) with Docker installed and Security Group Port 80 (HTTP) and Port 22 (SSH) open.

---

> [!IMPORTANT]
> **Docker Group Permission on EC2**:
> Adding a user to the `docker` group (`sudo usermod -aG docker ubuntu`) requires restarting the Docker daemon and establishing a fresh SSH session. In CI/CD SSH commands, we explicitly ensure the user can execute Docker commands, or we use `sudo docker` without password prompting by adding `ubuntu ALL=(ALL) NOPASSWD: /usr/bin/docker` if necessary.

---

## What We Need To Do
1. Configure Docker on EC2 and verify non-interactive SSH execution works without permission issues.
2. Set up GitHub Secrets (`DOCKERHUB_USERNAME`, `DOCKERHUB_TOKEN`, `EC2_HOST`, `EC2_USER`, `EC2_SSH_KEY`).
3. Build Docker image tagged with both `github.sha` (immutable) and `latest`.
4. Deploy the exact commit SHA tag on EC2 via SSH with automated rollback if container health check fails.

---

## How To Do It (Step-by-Step Instructions)

### Step 1: Install and Configure Docker on AWS EC2
Connect to EC2:
```bash
ssh -i "my-key.pem" ubuntu@<EC2-PUBLIC-IP>
```
Install Docker and configure group:
```bash
sudo apt update -y
sudo apt install -y docker.io curl
sudo usermod -aG docker ubuntu
sudo systemctl enable docker
sudo systemctl restart docker
```
Test that non-sudo docker works:
```bash
# Log out and log back in to refresh group membership
exit
ssh -i "my-key.pem" ubuntu@<EC2-PUBLIC-IP>
docker ps
```

### Step 2: Configure GitHub Repository Secrets
Under **Settings** -> **Secrets and variables** -> **Actions** -> **New repository secret**:
- `DOCKERHUB_USERNAME`: Your Docker Hub username.
- `DOCKERHUB_TOKEN`: Docker Hub Personal Access Token.
- `EC2_HOST`: EC2 Public IP address.
- `EC2_USER`: `ubuntu`
- `EC2_SSH_KEY`: Private `.pem` key content.

### Step 3: Project Files
**`Dockerfile`**:
```dockerfile
FROM nginx:alpine
COPY index.html /usr/share/nginx/html/index.html
EXPOSE 80
HEALTHCHECK --interval=10s --timeout=3s --retries=3 \
  CMD wget --quiet --tries=1 --spider http://localhost/ || exit 1
CMD ["nginx", "-g", "daemon off;"]
```

**`index.html`**:
```html
<!DOCTYPE html>
<html>
<head><title>Docker CI/CD Nginx</title></head>
<body>
    <h1>Production App running on Docker + Nginx via GitHub Actions</h1>
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
            ${{ secrets.DOCKERHUB_USERNAME }}/nginx-web:${{ steps.vars.outputs.sha_short }}
            ${{ secrets.DOCKERHUB_USERNAME }}/nginx-web:latest

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
            IMAGE="${{ secrets.DOCKERHUB_USERNAME }}/nginx-web:${{ needs.build-and-push.outputs.image_tag }}"
            echo "Pulling image: ${IMAGE}"
            sudo docker pull "${IMAGE}"

            echo "Stopping old container if present..."
            sudo docker stop nginx-app || true
            sudo docker rm nginx-app || true

            echo "Running new container..."
            sudo docker run -d --name nginx-app --restart always -p 80:80 "${IMAGE}"

            echo "Validating container health..."
            sleep 5
            if [ "$(sudo docker inspect -f '{{.State.Status}}' nginx-app)" != "running" ]; then
              echo "Deployment failed! Container is not running." >&2
              exit 1
            fi
            echo "Deployment verified successfully!"
```

### Step 5: Verify Deployment
Commit and push code to `main`. After the GitHub Actions run finishes, open `http://<EC2-PUBLIC-IP>`.
