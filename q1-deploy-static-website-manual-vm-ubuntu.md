# Practical 1: Deploy Static Website Manual on VM Ubuntu OS

## Objective
Manually install a web server on a local Ubuntu Virtual Machine (e.g., VirtualBox, VMware) and host a static HTML website.

## Prerequisites
- Ubuntu VM installed and running.
- Sudo access and active internet connectivity on the VM.

## What We Need To Do
1. Update Ubuntu repository package lists.
2. Install Nginx web server.
3. Start and enable Nginx service to run on boot.
4. Replace default web assets with static website HTML files.
5. Set appropriate file ownership and permissions for the web server user.
6. Verify deployment in the browser using the VM IP address.

## How To Do It (Step-by-Step Instructions)

### Step 1: Update package lists
```bash
sudo apt update && sudo apt upgrade -y
```

### Step 2: Install Nginx
```bash
sudo apt install nginx -y
```

### Step 3: Verify and start Nginx service
```bash
sudo systemctl enable nginx
sudo systemctl start nginx
sudo systemctl status nginx
```

### Step 4: Create static website files
Remove default web files:
```bash
sudo rm -rf /var/www/html/*
```
Create `index.html`:
```bash
sudo tee /var/www/html/index.html << 'EOF'
<!DOCTYPE html>
<html>
<head>
    <title>Practical 1 - Static Site</title>
</head>
<body>
    <h1>Deployed Manually on Ubuntu VM</h1>
</body>
</html>
EOF
```

### Step 5: Configure ownership and permissions
```bash
sudo chown -R www-data:www-data /var/www/html
sudo chmod -R 755 /var/www/html
```

### Step 6: Verify deployment
Get the VM IP address:
```bash
ip a
```
Open a web browser on your host machine and navigate to:
```text
http://<VM-IP-ADDRESS>
```

