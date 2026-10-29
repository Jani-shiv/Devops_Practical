# Practical 4: Deploy Static Website Autoscript on AWS EC2 using Ubuntu

## Objective
Write and execute an automated shell script on an existing AWS EC2 Ubuntu instance that provisions Nginx, safely deploys website content into a dedicated root directory, validates configuration syntax, and verifies deployment health.

## Prerequisites
- AWS EC2 instance running Ubuntu.
- Security Group inbound rules:
  - SSH (Port 22) restricted to your IP.
  - HTTP (Port 80) open to `0.0.0.0/0`.
- SSH access via `.pem` key.

## What We Need To Do
1. Connect to EC2 instance via SSH.
2. Create an automated bash script (`setup_ec2_site.sh`) without unnecessary dependencies.
3. Deploy files to `/var/www/ec2-autoscript` without purging default files destructively.
4. Validate Nginx configuration with `nginx -t` and verify response using `curl`.
5. Run the script and test the live website.

---

## How To Do It (Step-by-Step Instructions)

### Step 1: Connect to AWS EC2 Instance
```bash
ssh -i "my-key.pem" ubuntu@<EC2-PUBLIC-IP>
```

### Step 2: Create the Automation Script
```bash
nano setup_ec2_site.sh
```

### Step 3: Add Script Content
```bash
#!/usr/bin/env bash
set -euo pipefail

SITE_DIR="/var/www/ec2-autoscript"
NGINX_CONF="/etc/nginx/sites-available/ec2-autoscript"

echo "[1/4] Installing Nginx and Curl..."
sudo apt update -y
sudo apt install -y nginx curl

echo "[2/4] Deploying static website content..."
sudo mkdir -p "${SITE_DIR}"
cat << 'EOF' | sudo tee "${SITE_DIR}/index.html" > /dev/null
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>DevOps Practical 4 - EC2 Autoscript</title>
    <style>
        body { font-family: Arial, sans-serif; text-align: center; margin-top: 10%; background: #faf5ff; color: #581c87; }
        .card { background: white; padding: 2rem; border-radius: 8px; display: inline-block; box-shadow: 0 4px 6px rgba(0,0,0,0.1); }
    </style>
</head>
<body>
    <div class="card">
        <h1>Deployed via Autoscript on AWS EC2</h1>
        <p>Configured using dedicated root directory with automated health validation</p>
    </div>
</body>
</html>
EOF

sudo chown -R www-data:www-data "${SITE_DIR}"
sudo chmod -R 755 "${SITE_DIR}"

echo "[3/4] Configuring Virtual Host..."
cat << EOF | sudo tee "${NGINX_CONF}" > /dev/null
server {
    listen 80;
    listen [::]:80;
    server_name _;

    root ${SITE_DIR};
    index index.html;

    location / {
        try_files \$uri \$uri/ =404;
    }
}
EOF

sudo rm -f /etc/nginx/sites-enabled/default
sudo ln -sf "${NGINX_CONF}" /etc/nginx/sites-enabled/

echo "[4/4] Validating Configuration and Reloading Nginx..."
sudo nginx -t
sudo systemctl restart nginx
sudo systemctl enable nginx

# Health check verification
HTTP_STATUS=$(curl -s -o /dev/null -w "%{http_code}" http://localhost)
if [ "${HTTP_STATUS}" -eq 200 ]; then
    PUBLIC_IP=$(curl -s http://checkip.amazonaws.com || hostname -I | awk '{print $1}')
    echo "=========================================================="
    echo " SUCCESS: Deployment verified with HTTP 200 OK!"
    echo " Access website: http://${PUBLIC_IP}"
    echo "=========================================================="
else
    echo " ERROR: Health check failed with HTTP status ${HTTP_STATUS}" >&2
    exit 1
fi
```

### Step 4: Execute Script
```bash
chmod +x setup_ec2_site.sh
./setup_ec2_site.sh
```

### Step 5: Verify Deployment
Open your browser and navigate to:
```text
http://<EC2-PUBLIC-IP>
```
