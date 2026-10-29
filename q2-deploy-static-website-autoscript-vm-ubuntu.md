# Practical 2: Deploy Static Website Autoscript on VM Ubuntu OS

## Objective
Automate the installation of Nginx, configuration of a dedicated website directory, syntax validation, and deployment verification on an Ubuntu VM using an automated Bash script.

## Prerequisites
- Ubuntu 22.04 / 24.04 LTS VM with bash shell.
- User with sudo permissions.
- Network access configured (Bridged or NAT with port forwarding).

## What We Need To Do
1. Write a defensive Bash script (`deploy_static.sh`) that installs dependencies.
2. Deploy website files into a dedicated non-destructive directory (`/var/www/autoscript-site`).
3. Configure and validate Nginx configuration syntax (`nginx -t`).
4. Perform automated health check validation using `curl` inside the script before announcing success.
5. Grant execution permissions and run the script.

---

## How To Do It (Step-by-Step Instructions)

### Step 1: Create the Automated Deployment Script
Create the file:
```bash
nano deploy_static.sh
```

### Step 2: Add Script Content
```bash
#!/usr/bin/env bash
set -euo pipefail

SITE_DIR="/var/www/autoscript-site"
NGINX_CONF="/etc/nginx/sites-available/autoscript-site"

echo "=== [1/5] Updating Apt Packages and Installing Nginx ==="
sudo apt update -y
sudo apt install -y nginx curl

echo "=== [2/5] Creating Dedicated Site Directory ==="
sudo mkdir -p "${SITE_DIR}"
cat << 'EOF' | sudo tee "${SITE_DIR}/index.html" > /dev/null
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>DevOps Practical 2 - Autoscript VM</title>
    <style>
        body { font-family: Arial, sans-serif; text-align: center; margin-top: 10%; background: #f0fdf4; color: #166534; }
        .box { background: white; padding: 2rem; border-radius: 8px; display: inline-block; box-shadow: 0 4px 6px rgba(0,0,0,0.1); }
    </style>
</head>
<body>
    <div class="box">
        <h1>Static Website Deployed via Automated Script</h1>
        <p>Provisioned safely on Ubuntu VM with validated Nginx configuration</p>
    </div>
</body>
</html>
EOF

sudo chown -R www-data:www-data "${SITE_DIR}"
sudo chmod -R 755 "${SITE_DIR}"

echo "=== [3/5] Configuring Nginx Virtual Host ==="
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

echo "=== [4/5] Validating Nginx Configuration ==="
sudo nginx -t
sudo systemctl restart nginx
sudo systemctl enable nginx

echo "=== [5/5] Performing Health Check Verification ==="
HTTP_CODE=$(curl -s -o /dev/null -w "%{http_code}" http://localhost)
if [ "${HTTP_CODE}" -eq 200 ]; then
    PRIMARY_IP=$(hostname -I | awk '{print $1}')
    echo "=========================================================="
    echo " SUCCESS: Website is active and returned HTTP 200!"
    echo " Access from host browser at: http://${PRIMARY_IP}"
    echo "=========================================================="
else
    echo " ERROR: Health check failed with HTTP status code ${HTTP_CODE}" >&2
    exit 1
fi
```

### Step 3: Grant Execution Permissions and Run
```bash
chmod +x deploy_static.sh
./deploy_static.sh
```

### Step 4: Verify Deployment
Open your host browser and browse to the IP reported by the script output:
```text
http://<VM-IP-ADDRESS>
```
