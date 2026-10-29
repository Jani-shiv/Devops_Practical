# Practical 1: Deploy Static Website Manual on VM Ubuntu OS

## Objective
Manually install a web server on a local Ubuntu Virtual Machine (VirtualBox, VMware, Multipass) and host a static HTML website using dedicated site directories without destroying default host configurations.

## Prerequisites
- Ubuntu 22.04 / 24.04 LTS VM installed and running.
- Sudo administrative access and active internet connectivity on the VM.
- Hypervisor network configured as **Bridged Adapter** (to receive an IP on your local LAN) or **NAT with Port Forwarding** (forward Host Port `8080` to Guest Port `80`).

## What We Need To Do
1. Understand and verify VM network connectivity so the host machine can access the web server.
2. Update Ubuntu repository package lists.
3. Install Nginx web server and verify service daemon status.
4. Set up a dedicated website directory `/var/www/static-site` instead of destructively purging `/var/www/html`.
5. Configure an Nginx server block for the site.
6. Set correct permissions for the `www-data` user.
7. Verify deployment from both inside the VM and from the host machine browser.

---

## How To Do It (Step-by-Step Instructions)

### Step 1: Verify VM Network Access
Find the IP address of your Ubuntu VM:
```bash
ip -br a
```
> **Network Access Note:**
> - If using **Bridged Networking**: Your VM receives an IP on your home/office network (e.g. `192.168.1.50`). You can browse directly to this IP from your host machine.
> - If using **NAT Networking**: You must add a Port Forwarding rule in VirtualBox/VMware settings:
>   - Protocol: `TCP`
>   - Host Port: `8080`
>   - Guest Port: `80`
>   Then you access it via `http://localhost:8080` on your host machine.

### Step 2: Update Package Lists and Install Nginx
```bash
sudo apt update && sudo apt upgrade -y
sudo apt install nginx curl -y
```

### Step 3: Verify and Start Nginx Service
```bash
sudo systemctl enable nginx
sudo systemctl start nginx
sudo systemctl status nginx --no-pager
```

### Step 4: Create Dedicated Website Directory and Content
> [!NOTE]
> We use a dedicated directory `/var/www/static-site` to avoid destructive deletion of default web roots.

```bash
sudo mkdir -p /var/www/static-site
sudo tee /var/www/static-site/index.html << 'EOF'
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>DevOps Practical 1 - Ubuntu VM</title>
    <style>
        body { font-family: Arial, sans-serif; text-align: center; margin-top: 10%; background: #f4f6f8; color: #333; }
        .card { background: white; padding: 2rem; border-radius: 8px; display: inline-block; box-shadow: 0 4px 6px rgba(0,0,0,0.1); }
        h1 { color: #0066cc; }
    </style>
</head>
<body>
    <div class="card">
        <h1>Static Website Deployed Manually</h1>
        <p>Running on Ubuntu Linux VM with Nginx Web Server</p>
    </div>
</body>
</html>
EOF
```

### Step 5: Configure Nginx Virtual Host
Create a custom configuration file:
```bash
sudo tee /etc/nginx/sites-available/static-site << 'EOF'
server {
    listen 80;
    listen [::]:80;
    server_name _;

    root /var/www/static-site;
    index index.html;

    location / {
        try_files $uri $uri/ =404;
    }
}
EOF
```

Disable default site and enable custom site:
```bash
sudo rm -f /etc/nginx/sites-enabled/default
sudo ln -sf /etc/nginx/sites-available/static-site /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx
```

### Step 6: Configure Permissions
```bash
sudo chown -R www-data:www-data /var/www/static-site
sudo chmod -R 755 /var/www/static-site
```

### Step 7: Verify Deployment
Test locally on the VM:
```bash
curl -I http://localhost
```
Open a browser on your host machine and navigate to:
```text
http://<VM-IP-ADDRESS>
# or http://localhost:8080 (if using NAT Port Forwarding)
```
