# Practical 7: Hosting a Static Website on AWS EC2 using GitHub Actions (CI/CD) using Ubuntu Server + Nginx

## Objective
Build an automated CI/CD pipeline using GitHub Actions to continuously deploy static website updates to an AWS EC2 Ubuntu instance hosting Nginx.

## Prerequisites
- AWS EC2 instance running Ubuntu with Nginx installed.
- Security Group open on Port 22 (SSH) and Port 80 (HTTP).
- GitHub repository with website files (`index.html`, etc.).

## What We Need To Do
1. Configure Nginx web root directory permissions on EC2.
2. Add EC2 connection credentials to GitHub Secrets.
3. Create GitHub Actions workflow to transfer files via SCP/SSH and restart Nginx.
4. Push a code change to verify automatic deployment.

## How To Do It (Step-by-Step Instructions)

### Step 1: Prepare EC2 Server permissions
Connect to EC2 and set ownership of the web root to user `ubuntu`:
```bash
ssh -i "my-key.pem" ubuntu@<EC2-PUBLIC-IP>
sudo apt update -y && sudo apt install nginx -y
sudo chown -R ubuntu:ubuntu /var/www/html
```

### Step 2: Configure GitHub Secrets
In your GitHub repository, navigate to **Settings** -> **Secrets and variables** -> **Actions** -> **New repository secret**:
- `EC2_HOST`: `<EC2-PUBLIC-IP>`
- `EC2_USER`: `ubuntu`
- `EC2_SSH_KEY`: Paste the full content of your private `.pem` key file.

### Step 3: Create GitHub Actions Workflow
Create file `.github/workflows/deploy-nginx.yml`:
```yaml
name: Deploy Static Website to EC2 Nginx

on:
  push:
    branches: [ main ]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout repository code
        uses: actions/checkout@v4

      - name: Copy static files to EC2 Nginx web root
        uses: appleboy/scp-action@v0.1.7
        with:
          host: ${{ secrets.EC2_HOST }}
          username: ${{ secrets.EC2_USER }}
          key: ${{ secrets.EC2_SSH_KEY }}
          source: "index.html,css,js,images"
          target: "/var/www/html"

      - name: Restart Nginx on EC2
        uses: appleboy/ssh-action@v1.0.3
        with:
          host: ${{ secrets.EC2_HOST }}
          username: ${{ secrets.EC2_USER }}
          key: ${{ secrets.EC2_SSH_KEY }}
          script: |
            sudo systemctl restart nginx
```

### Step 4: Verify deployment
1. Commit any change to `index.html` and push to branch `main`.
2. Go to the **Actions** tab on GitHub to monitor pipeline execution.
3. Open `http://<EC2-PUBLIC-IP>` to view the deployed website.

