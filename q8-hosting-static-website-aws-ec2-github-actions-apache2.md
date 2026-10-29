# Practical 8: Hosting a Static Website on AWS EC2 using GitHub Actions (CI/CD) using Ubuntu Server + Apache2

## Objective
Build an automated CI/CD pipeline using GitHub Actions to continuously deploy static website updates to an AWS EC2 Ubuntu instance hosting Apache2.

## Prerequisites
- AWS EC2 instance running Ubuntu.
- Security Group open on Port 22 (SSH) and Port 80 (HTTP).
- GitHub repository with website files (`index.html`, etc.).

## What We Need To Do
1. Install Apache2 on the EC2 instance and set directory permissions.
2. Store EC2 SSH connection details in GitHub Secrets.
3. Create GitHub Actions workflow to transfer files via SCP and restart Apache2.
4. Verify continuous delivery upon code push.

## How To Do It (Step-by-Step Instructions)

### Step 1: Install Apache2 on EC2
```bash
ssh -i "my-key.pem" ubuntu@<EC2-PUBLIC-IP>
sudo apt update -y
sudo apt install apache2 -y
sudo systemctl enable apache2
sudo systemctl start apache2
sudo chown -R ubuntu:ubuntu /var/www/html
```

### Step 2: Configure GitHub Secrets
In your GitHub repository, go to **Settings** -> **Secrets and variables** -> **Actions** -> **New repository secret**:
- `EC2_HOST`: `<EC2-PUBLIC-IP>`
- `EC2_USER`: `ubuntu`
- `EC2_SSH_KEY`: Content of your `.pem` private key.

### Step 3: Create GitHub Actions Workflow
Create file `.github/workflows/deploy-apache.yml`:
```yaml
name: Deploy Static Website to EC2 Apache2

on:
  push:
    branches: [ main ]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout repository code
        uses: actions/checkout@v4

      - name: Copy static files to EC2 Apache2 web root
        uses: appleboy/scp-action@v0.1.7
        with:
          host: ${{ secrets.EC2_HOST }}
          username: ${{ secrets.EC2_USER }}
          key: ${{ secrets.EC2_SSH_KEY }}
          source: "index.html,css,js,images"
          target: "/var/www/html"

      - name: Restart Apache2 on EC2
        uses: appleboy/ssh-action@v1.0.3
        with:
          host: ${{ secrets.EC2_HOST }}
          username: ${{ secrets.EC2_USER }}
          key: ${{ secrets.EC2_SSH_KEY }}
          script: |
            sudo systemctl restart apache2
```

### Step 4: Verify deployment
1. Push an update to `index.html` on branch `main`.
2. Inspect the GitHub Actions pipeline.
3. Access `http://<EC2-PUBLIC-IP>` in your browser.

