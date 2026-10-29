# Practical 3: Deploy Static Website Manual on AWS EC2 using Ubuntu

## Objective
Provision an Ubuntu EC2 instance on AWS with security best practices, connect via SSH restricted to your IP, and manually install and configure Nginx with a dedicated site root.

## Prerequisites
- Active AWS Account.
- EC2 Key Pair (`.pem` file downloaded locally).
- Security Group inbound rules configured safely:
  - **SSH (Port 22)**: Restricted to **My IP** (e.g., `203.0.113.25/32`) instead of `0.0.0.0/0`.
  - **HTTP (Port 80)**: Allowed from Anywhere (`0.0.0.0/0`) for web browsing.

## What We Need To Do
1. Launch an Ubuntu EC2 instance (`t2.micro` or `t3.micro`).
2. Secure local key permissions and connect via SSH.
3. Update OS packages and install Nginx.
4. Deploy static web files to a dedicated site directory (`/var/www/ec2-site`).
5. Configure Nginx virtual host and test syntax.
6. Verify access using the EC2 Public IPv4 address.

---

## How To Do It (Step-by-Step Instructions)

### Step 1: Secure Key and Connect to EC2
In your local command terminal (navigate to where `my-key.pem` is saved):
```bash
chmod 400 my-key.pem
ssh -i "my-key.pem" ubuntu@<EC2-PUBLIC-IP>
```

### Step 2: Update System Packages and Install Nginx
```bash
sudo apt update -y && sudo apt upgrade -y
sudo apt install -y nginx curl
```

### Step 3: Create Dedicated Website Directory
```bash
sudo mkdir -p /var/www/ec2-site
sudo tee /var/www/ec2-site/index.html << 'EOF'
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>DevOps Practical 3 - AWS EC2 Manual</title>
    <style>
        body { font-family: Arial, sans-serif; text-align: center; margin-top: 10%; background: #eff6ff; color: #1e3a8a; }
        .card { background: white; padding: 2rem; border-radius: 8px; display: inline-block; box-shadow: 0 4px 6px rgba(0,0,0,0.1); }
    </style>
</head>
<body>
    <div class="card">
        <h1>Static Website on AWS EC2</h1>
        <p>Manually installed and configured with dedicated Nginx Virtual Host</p>
    </div>
</body>
</html>
EOF

sudo chown -R www-data:www-data /var/www/ec2-site
sudo chmod -R 755 /var/www/ec2-site
```

### Step 4: Configure Nginx Virtual Host
```bash
sudo tee /etc/nginx/sites-available/ec2-site << 'EOF'
server {
    listen 80;
    listen [::]:80;
    server_name _;

    root /var/www/ec2-site;
    index index.html;

    location / {
        try_files $uri $uri/ =404;
    }
}
EOF

sudo rm -f /etc/nginx/sites-enabled/default
sudo ln -sf /etc/nginx/sites-available/ec2-site /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx
sudo systemctl enable nginx
```

### Step 5: Verify Deployment
Run internal validation check:
```bash
curl -I http://localhost
```
Open a browser and navigate to the public IP:
```text
http://<EC2-PUBLIC-IP>
```
