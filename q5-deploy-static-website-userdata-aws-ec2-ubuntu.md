# Practical 5: Deploy Static Website Userdata on AWS EC2 using Ubuntu

## Objective
Bootstrap an AWS EC2 instance at initial boot using EC2 User Data (cloud-init) to automatically provision Nginx, configure a dedicated site root, and understand cloud-init logs for troubleshooting.

## Prerequisites
- AWS Account with EC2 launch permissions.
- Security Group allowing inbound HTTP (Port 80) from anywhere (`0.0.0.0/0`) and SSH (Port 22) from your IP for troubleshooting.

## What We Need To Do
1. Configure an EC2 launch template or wizard instance with Ubuntu Server AMI.
2. Embed the cloud-init bootstrap script in **User data** under **Advanced details**.
3. Deploy files to a dedicated web directory (`/var/www/userdata-site`).
4. Launch the instance and verify availability on the public IP.
5. Inspect `/var/log/cloud-init-output.log` if troubleshooting first-boot execution.

---

## How To Do It (Step-by-Step Instructions)

### Step 1: Launch EC2 Instance
- **Name**: `Static-Site-UserData`
- **AMI**: Ubuntu Server 22.04 LTS or 24.04 LTS
- **Instance Type**: `t2.micro` or `t3.micro`
- **Network Settings**:
  - Auto-assign public IP: `Enable`
  - Inbound Security Group Rules:
    - HTTP (Port 80) -> Source: `0.0.0.0/0`
    - SSH (Port 22) -> Source: `My IP`

### Step 2: Configure EC2 User Data
Under **Advanced details**, paste the following script into the **User data** text box:

```bash
#!/usr/bin/env bash
set -euo pipefail

# Log all output to cloud-init log
exec > >(tee /var/log/user-data.log|logger -t user-data -s 2>/dev/console) 2>&1

echo "Starting EC2 UserData provisioning..."

apt-get update -y
apt-get install -y nginx curl

SITE_DIR="/var/www/userdata-site"
mkdir -p "${SITE_DIR}"

cat << 'EOF' > "${SITE_DIR}/index.html"
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>DevOps Practical 5 - EC2 User Data</title>
    <style>
        body { font-family: Arial, sans-serif; text-align: center; margin-top: 10%; background: #fffbeb; color: #92400e; }
        .card { background: white; padding: 2rem; border-radius: 8px; display: inline-block; box-shadow: 0 4px 6px rgba(0,0,0,0.1); }
    </style>
</head>
<body>
    <div class="card">
        <h1>Static Website Deployed via EC2 User Data</h1>
        <p>Automated cloud-init bootstrap on instance launch</p>
    </div>
</body>
</html>
EOF

chown -R www-data:www-data "${SITE_DIR}"
chmod -R 755 "${SITE_DIR}"

# Create Nginx virtual host
cat << 'EOF' > /etc/nginx/sites-available/userdata-site
server {
    listen 80;
    listen [::]:80;
    server_name _;

    root /var/www/userdata-site;
    index index.html;

    location / {
        try_files $uri $uri/ =404;
    }
}
EOF

rm -f /etc/nginx/sites-enabled/default
ln -sf /etc/nginx/sites-available/userdata-site /etc/nginx/sites-enabled/

nginx -t
systemctl restart nginx
systemctl enable nginx

echo "UserData provisioning completed successfully!"
```

### Step 3: Launch Instance
Click **Launch instance**.

### Step 4: Verify Deployment
1. Wait 2–3 minutes for the instance status checks to reach `2/2 checks passed`.
2. Copy the **Public IPv4 address** from the AWS Console.
3. Open browser:
```text
http://<EC2-PUBLIC-IP>
```

### Step 5: Troubleshooting First-Boot Execution (If Site Fails to Load)
If the site does not load, connect via SSH and check the cloud-init execution logs:
```bash
ssh -i "my-key.pem" ubuntu@<EC2-PUBLIC-IP>
sudo tail -n 50 /var/log/cloud-init-output.log
sudo cat /var/log/user-data.log
```
Check Nginx status and error logs:
```bash
sudo systemctl status nginx
sudo cat /var/log/nginx/error.log
```
