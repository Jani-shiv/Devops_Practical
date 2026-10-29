# Practical 3: Deploy Static Website Manual on AWS EC2 using Ubuntu

## Objective
Launch an Ubuntu EC2 instance on AWS, connect via SSH, and manually install and configure Nginx to serve a static website.

## Prerequisites
- Active AWS Account.
- EC2 Key Pair (`.pem` file downloaded).
- Security Group inbound rules:
  - SSH: Port 22 (from My IP or 0.0.0.0/0)
  - HTTP: Port 80 (from 0.0.0.0/0)

## What We Need To Do
1. Launch an Ubuntu EC2 instance.
2. Connect to the EC2 instance using SSH.
3. Update OS packages and install Nginx.
4. Add static HTML files to `/var/www/html`.
5. Access the public IP to confirm website is live.

## How To Do It (Step-by-Step Instructions)

### Step 1: Connect to AWS EC2 instance via SSH
Open your local terminal (where `.pem` file is located):
```bash
chmod 400 my-key.pem
ssh -i "my-key.pem" ubuntu@<EC2-PUBLIC-IP>
```

### Step 2: Update system packages
```bash
sudo apt update -y && sudo apt upgrade -y
```

### Step 3: Install and start Nginx
```bash
sudo apt install nginx -y
sudo systemctl enable nginx
sudo systemctl start nginx
```

### Step 4: Deploy static website
```bash
sudo rm -rf /var/www/html/*
cat << 'EOF' | sudo tee /var/www/html/index.html
<!DOCTYPE html>
<html>
<head>
    <title>Practical 3 - AWS EC2 Manual</title>
</head>
<body>
    <h1>Deployed Manually on AWS EC2 Ubuntu</h1>
</body>
</html>
EOF
```

### Step 5: Verify deployment
Open browser and navigate to:
```text
http://<EC2-PUBLIC-IP>
```

