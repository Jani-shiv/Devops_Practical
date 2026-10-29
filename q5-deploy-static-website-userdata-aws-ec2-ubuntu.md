# Practical 5: Deploy Static Website Userdata on AWS EC2 using Ubuntu

## Objective
Bootstrap an AWS EC2 instance at launch time using EC2 User Data (cloud-init) to automatically deploy a static website with zero manual SSH configuration.

## Prerequisites
- AWS Account with permissions to launch EC2 instances.
- Security Group allowing inbound HTTP traffic on Port 80 (0.0.0.0/0).

## What We Need To Do
1. Open the AWS EC2 Launch Instance console.
2. Select Ubuntu Server AMI.
3. Configure networking to allow HTTP Port 80.
4. Paste cloud-init bootstrap script into the **User data** field in **Advanced details**.
5. Launch the instance and verify the site via the public IP address once initialized.

## How To Do It (Step-by-Step Instructions)

### Step 1: Launch EC2 Instance
- **Name**: `Static-Site-UserData`
- **AMI**: Ubuntu Server 22.04 LTS or 24.04 LTS
- **Instance type**: `t2.micro` or `t3.micro`
- **Security group**: Enable "Allow HTTP traffic from the internet (0.0.0.0/0)"

### Step 2: Configure EC2 User Data
Scroll down to **Advanced details**, find the **User data** box, and paste:

```bash
#!/usr/bin/env bash
apt update -y
apt install nginx -y
systemctl start nginx
systemctl enable nginx

rm -rf /var/www/html/*
cat << 'EOF' > /var/www/html/index.html
<!DOCTYPE html>
<html>
<head>
    <title>Practical 5 - EC2 User Data</title>
</head>
<body>
    <h1>Deployed Automatically via AWS EC2 User Data</h1>
</body>
</html>
EOF

chown -R www-data:www-data /var/www/html
chmod -R 755 /var/www/html
```

### Step 3: Launch Instance
Click **Launch instance**.

### Step 4: Verify deployment
1. Wait 2–3 minutes for the instance status checks to reach `2/2 checks passed`.
2. Copy the **Public IPv4 address** from the EC2 instance dashboard.
3. Open browser and visit:
```text
http://<EC2-PUBLIC-IP>
```

