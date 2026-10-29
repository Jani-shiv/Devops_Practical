# Practical 4: Deploy Static Website Autoscript on AWS EC2 using Ubuntu

## Objective
Write and execute an automated shell script on an existing AWS EC2 Ubuntu instance to install dependencies and deploy a static website.

## Prerequisites
- AWS EC2 instance running Ubuntu.
- Security Group open on Port 22 (SSH) and Port 80 (HTTP).
- SSH access via `.pem` key.

## What We Need To Do
1. SSH into the AWS EC2 instance.
2. Create an automated bash script (`setup_ec2_site.sh`).
3. Make the script executable and run it.
4. Verify the static website on the EC2 public IP.

## How To Do It (Step-by-Step Instructions)

### Step 1: Connect to the EC2 instance
```bash
ssh -i "my-key.pem" ubuntu@<EC2-PUBLIC-IP>
```

### Step 2: Create the automated setup script
```bash
nano setup_ec2_site.sh
```

### Step 3: Add the script content
```bash
#!/usr/bin/env bash
set -e

echo "[1/3] Updating system packages..."
sudo apt update -y

echo "[2/3] Installing Nginx..."
sudo apt install nginx -y

echo "[3/3] Deploying static website content..."
sudo rm -rf /var/www/html/*
cat << 'EOF' | sudo tee /var/www/html/index.html
<!DOCTYPE html>
<html>
<head>
    <title>Practical 4 - EC2 Autoscript</title>
</head>
<body>
    <h1>Deployed using Autoscript on AWS EC2 Ubuntu</h1>
</body>
</html>
EOF

sudo chown -R www-data:www-data /var/www/html
sudo chmod -R 755 /var/www/html
sudo systemctl restart nginx
sudo systemctl enable nginx

echo "Deployment complete! Visit: http://$(curl -s http://checkip.amazonaws.com)"
```

### Step 4: Make executable and run
```bash
chmod +x setup_ec2_site.sh
./setup_ec2_site.sh
```

### Step 5: Verify deployment
Open browser and browse to:
```text
http://<EC2-PUBLIC-IP>
```

