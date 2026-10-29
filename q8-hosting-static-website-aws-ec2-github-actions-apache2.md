# Practical 8: Hosting a Static Website on AWS EC2 using GitHub Actions (CI/CD) using Ubuntu Server + Apache2

## Objective
Build an automated CI/CD pipeline using GitHub Actions to continuously deploy static website updates from a dedicated source folder in the repository to an AWS EC2 Ubuntu instance hosting Apache2.

## Prerequisites
- AWS EC2 instance running Ubuntu.
- Security Group open on Port 22 (SSH) and Port 80 (HTTP).
- GitHub repository with website source files located in `src/`.

## Repository Project Structure
```text
my-static-site/
├── src/
│   ├── index.html
│   ├── style.css
│   └── script.js
└── .github/
    └── workflows/
        └── deploy-apache.yml
```

## What We Need To Do
1. Install Apache2 on EC2 and configure a dedicated VirtualHost pointing to `/var/www/apache-site`.
2. Grant ownership of `/var/www/apache-site` to user `ubuntu`.
3. Configure GitHub Secrets (`EC2_HOST`, `EC2_USER`, `EC2_SSH_KEY`).
4. Write GitHub Actions workflow utilizing `appleboy/scp-action` targeting `src/*` with `strip_components: 1`.
5. Test configuration and reload Apache2 via `appleboy/ssh-action`.

---

## How To Do It (Step-by-Step Instructions)

### Step 1: Install and Configure Apache2 on EC2
Connect to EC2 via SSH:
```bash
ssh -i "my-key.pem" ubuntu@<EC2-PUBLIC-IP>
```
Install Apache2 and set up dedicated directory:
```bash
sudo apt update -y && sudo apt install apache2 -y

sudo mkdir -p /var/www/apache-site
sudo chown -R ubuntu:ubuntu /var/www/apache-site
sudo chmod -R 755 /var/www/apache-site
```

Configure Apache VirtualHost:
```bash
sudo tee /etc/apache2/sites-available/apache-site.conf << 'EOF'
<VirtualHost *:80>
    ServerAdmin webmaster@localhost
    DocumentRoot /var/www/apache-site

    <Directory /var/www/apache-site>
        Options -Indexes +FollowSymLinks
        AllowOverride None
        Require all granted
    </Directory>

    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
EOF

sudo a2dissite 000-default.conf
sudo a2ensite apache-site.conf
sudo apache2ctl configtest
sudo systemctl restart apache2
```

### Step 2: Configure GitHub Secrets
In your GitHub repository, navigate to **Settings** -> **Secrets and variables** -> **Actions** -> **New repository secret**:
- `EC2_HOST`: `<EC2-PUBLIC-IP>`
- `EC2_USER`: `ubuntu`
- `EC2_SSH_KEY`: Content of your `.pem` private key.

### Step 3: Create GitHub Actions Workflow
In your repository, create `.github/workflows/deploy-apache.yml`:
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

      - name: Transfer static assets to EC2 via SCP
        uses: appleboy/scp-action@v0.1.7
        with:
          host: ${{ secrets.EC2_HOST }}
          username: ${{ secrets.EC2_USER }}
          key: ${{ secrets.EC2_SSH_KEY }}
          source: "src/*"
          target: "/var/www/apache-site"
          strip_components: 1

      - name: Validate and Reload Apache2
        uses: appleboy/ssh-action@v1.0.3
        with:
          host: ${{ secrets.EC2_HOST }}
          username: ${{ secrets.EC2_USER }}
          key: ${{ secrets.EC2_SSH_KEY }}
          script: |
            sudo apache2ctl configtest
            sudo systemctl reload apache2
```

### Step 4: Verify Deployment
Push a change in `src/index.html` to branch `main`, inspect the Actions log, and check `http://<EC2-PUBLIC-IP>`.
