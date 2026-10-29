# Practical 7: Hosting a Static Website on AWS EC2 using GitHub Actions (CI/CD) using Ubuntu Server + Nginx

## Objective
Build an automated CI/CD pipeline using GitHub Actions to continuously deploy static website updates from a dedicated source folder in the repository to a dedicated site root on an AWS EC2 Ubuntu instance running Nginx.

## Prerequisites
- AWS EC2 instance running Ubuntu.
- Security Group open on:
  - SSH (Port 22) allowed from GitHub Actions runner IPs or specific bastion IP.
  - HTTP (Port 80) allowed from anywhere (`0.0.0.0/0`).
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
        └── deploy-nginx.yml
```

## What We Need To Do
1. Configure Nginx on EC2 with a dedicated web root (`/var/www/my-site`) with proper ownership for user `ubuntu`.
2. Add SSH secrets (`EC2_HOST`, `EC2_USER`, `EC2_SSH_KEY`) to GitHub Secrets.
3. Author a GitHub Actions workflow that transfers only the contents of `src/` to the destination directory, avoiding missing-path errors.
4. Test Nginx syntax and reload Nginx service.
5. Push a code change to verify automatic deployment.

---

## How To Do It (Step-by-Step Instructions)

### Step 1: Prepare EC2 Server & Dedicated Directory
Connect to your EC2 instance via SSH:
```bash
ssh -i "my-key.pem" ubuntu@<EC2-PUBLIC-IP>
```
Install Nginx and create dedicated web directory:
```bash
sudo apt update -y && sudo apt install nginx -y

# Create dedicated directory with ownership for deployment user
sudo mkdir -p /var/www/my-site
sudo chown -R ubuntu:ubuntu /var/www/my-site
sudo chmod -R 755 /var/www/my-site
```

Configure Nginx server block for `/var/www/my-site`:
```bash
sudo tee /etc/nginx/sites-available/my-site << 'EOF'
server {
    listen 80;
    listen [::]:80;
    server_name _;

    root /var/www/my-site;
    index index.html;

    location / {
        try_files $uri $uri/ =404;
    }
}
EOF

sudo rm -f /etc/nginx/sites-enabled/default
sudo ln -sf /etc/nginx/sites-available/my-site /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl restart nginx
```

### Step 2: Configure GitHub Secrets
In your GitHub repository, navigate to **Settings** -> **Secrets and variables** -> **Actions** -> **New repository secret**:
- `EC2_HOST`: `<EC2-PUBLIC-IP>`
- `EC2_USER`: `ubuntu`
- `EC2_SSH_KEY`: Entire contents of your private `.pem` key file (including `-----BEGIN RSA PRIVATE KEY-----` and `-----END RSA PRIVATE KEY-----`).

### Step 3: Create GitHub Actions Workflow
In your repository, create `.github/workflows/deploy-nginx.yml`:
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

      - name: Transfer static assets to EC2 via SCP
        uses: appleboy/scp-action@v0.1.7
        with:
          host: ${{ secrets.EC2_HOST }}
          username: ${{ secrets.EC2_USER }}
          key: ${{ secrets.EC2_SSH_KEY }}
          source: "src/*"
          target: "/var/www/my-site"
          strip_components: 1

      - name: Validate and Reload Nginx
        uses: appleboy/ssh-action@v1.0.3
        with:
          host: ${{ secrets.EC2_HOST }}
          username: ${{ secrets.EC2_USER }}
          key: ${{ secrets.EC2_SSH_KEY }}
          script: |
            sudo nginx -t
            sudo systemctl reload nginx
```

### Step 4: Verify Deployment
1. Commit changes to `src/index.html` and push to `main`.
2. Open the **Actions** tab on GitHub to monitor the pipeline.
3. Open `http://<EC2-PUBLIC-IP>` to view the updated site.
