# Practical 2: Deploy Static Website Autoscript on VM Ubuntu OS

## Objective
Automate the installation of Nginx and the deployment of a static website on an Ubuntu VM using a Bash shell script.

## Prerequisites
- Ubuntu VM with bash shell.
- Sudo/root access.

## What We Need To Do
1. Write a shell script (`deploy_static.sh`) that automates apt update, package installation, directory cleanup, and static asset creation.
2. Grant execution permissions to the script.
3. Run the script and verify website availability via the VM IP address.

## How To Do It (Step-by-Step Instructions)

### Step 1: Create the automation script
```bash
nano deploy_static.sh
```

### Step 2: Paste the following script content
```bash
#!/usr/bin/env bash
set -e

echo "[1/4] Updating packages..."
sudo apt update -y

echo "[2/4] Installing Nginx..."
sudo apt install nginx -y

echo "[3/4] Deploying static website files..."
sudo rm -rf /var/www/html/*
cat << 'EOF' | sudo tee /var/www/html/index.html
<!DOCTYPE html>
<html>
<head>
    <title>Practical 2 - Autoscript</title>
</head>
<body>
    <h1>Deployed via Automated Script on Ubuntu VM</h1>
</body>
</html>
EOF

sudo chown -R www-data:www-data /var/www/html
sudo chmod -R 755 /var/www/html

echo "[4/4] Restarting Nginx service..."
sudo systemctl restart nginx
sudo systemctl enable nginx

echo "Deployment complete! Visit: http://$(hostname -I | awk '{print $1}')"
```

### Step 3: Grant execution permissions
```bash
chmod +x deploy_static.sh
```

### Step 4: Run the script
```bash
./deploy_static.sh
```

### Step 5: Verify deployment
Open browser and access:
```text
http://<VM-IP-ADDRESS>
```

