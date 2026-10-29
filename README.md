# DevOps Practical Labs Guide (CI/CD, Docker, GitHub Actions, Jenkins, AWS)

Comprehensive, step-by-step documentation for all 14 DevOps practical exercises.

---

## Table of Contents
1. [Practical 1: Deploy Static Website Manually on VM (Ubuntu OS)](#practical-1-deploy-static-website-manually-on-vm-ubuntu-os)
2. [Practical 2: Deploy Static Website via Automated Script on VM (Ubuntu OS)](#practical-2-deploy-static-website-via-automated-script-on-vm-ubuntu-os)
3. [Practical 3: Deploy Static Website Manually on AWS EC2 (Ubuntu)](#practical-3-deploy-static-website-manually-on-aws-ec2-ubuntu)
4. [Practical 4: Deploy Static Website via Automated Script on AWS EC2 (Ubuntu)](#practical-4-deploy-static-website-via-automated-script-on-aws-ec2-ubuntu)
5. [Practical 5: Deploy Static Website via EC2 User Data on AWS (Ubuntu)](#practical-5-deploy-static-website-via-ec2-user-data-on-aws-ubuntu)
6. [Practical 6: GitHub Actions + Self-Hosted Runner Deployment on AWS EC2 (Ubuntu)](#practical-6-github-actions--self-hosted-runner-deployment-on-aws-ec2-ubuntu)
7. [Practical 7: Static Website on AWS EC2 using GitHub Actions + Nginx (CI/CD)](#practical-7-static-website-on-aws-ec2-using-github-actions--nginx-cicd)
8. [Practical 8: Static Website on AWS EC2 using GitHub Actions + Apache2 (CI/CD)](#practical-8-static-website-on-aws-ec2-using-github-actions--apache2-cicd)
9. [Practical 9: Create Portfolio Container and Push to Docker Hub](#practical-9-create-portfolio-container-and-push-to-docker-hub)
10. [Practical 10: CI/CD Pipeline using GitHub Actions, Docker, Docker Hub, and AWS EC2 (Nginx)](#practical-10-cicd-pipeline-using-github-actions-docker-docker-hub-and-aws-ec2-nginx)
11. [Practical 11: CI/CD Pipeline using GitHub Actions, Docker, Docker Hub, and AWS EC2 (Apache2)](#practical-11-cicd-pipeline-using-github-actions-docker-docker-hub-and-aws-ec2-apache2)
12. [Practical 12: Full Stack CI/CD (React + Node.js + MongoDB + Docker + GitHub Actions + AWS EC2)](#practical-12-full-stack-cicd-pipeline-using-react--nodejs--mongodb--docker--github-actions--aws-ec2)
13. [Practical 13: Jenkins Installation and Job Setup (Windows Local PC)](#practical-13-jenkins-installation-and-job-setup-windows-local-pc)
14. [Practical 14: Jenkins Deployment on AWS EC2 with CI/CD Pipeline (GitHub -> Jenkins -> Docker -> Nginx)](#practical-14-jenkins-deployment-on-aws-ec2-with-cicd-pipeline)

---

## Practical 1: Deploy Static Website Manually on VM (Ubuntu OS)

### Objective
Manually install a web server on a local Ubuntu Virtual Machine (e.g., VirtualBox, VMware) and host a static HTML website.

### Prerequisites
- Ubuntu VM installed and running.
- Sudo access and internet connectivity on the VM.

### Step-by-Step Instructions

1. **Update package lists:**
   ```bash
   sudo apt update && sudo apt upgrade -y
   ```

2. **Install Nginx:**
   ```bash
   sudo apt install nginx -y
   ```

3. **Verify and start Nginx service:**
   ```bash
   sudo systemctl status nginx
   sudo systemctl enable nginx
   sudo systemctl start nginx
   ```

4. **Create static HTML website files:**
   ```bash
   sudo rm -rf /var/www/html/*
   sudo nano /var/www/html/index.html
   ```
   Add sample content:
   ```html
   <!DOCTYPE html>
   <html>
   <head><title>Practical 1 - Static Site</title></head>
   <body>
       <h1>Deployed Manually on Ubuntu VM</h1>
   </body>
   </html>
   ```

5. **Set file permissions:**
   ```bash
   sudo chown -R www-data:www-data /var/www/html
   sudo chmod -R 755 /var/www/html
   ```

6. **Verify deployment:**
   - Find VM IP address:
     ```bash
     ip a
     ```
   - Open browser on host machine and browse to `http://<VM-IP-ADDRESS>`.

---

## Practical 2: Deploy Static Website via Automated Script on VM (Ubuntu OS)

### Objective
Automate the installation and deployment of a static website on an Ubuntu VM using a Bash shell script.

### Prerequisites
- Ubuntu VM with bash shell.

### Step-by-Step Instructions

1. **Create the deployment script:**
   ```bash
   nano deploy_static.sh
   ```

2. **Add the following script content:**
   ```bash
   #!/usr/bin/env bash
   set -e

   echo "[1/4] Updating packages..."
   sudo apt update -y

   echo "[2/4] Installing Nginx..."
   sudo apt install nginx -y

   echo "[3/4] Deploying website files..."
   sudo rm -rf /var/www/html/*
   cat << 'EOF' | sudo tee /var/www/html/index.html
   <!DOCTYPE html>
   <html>
   <head><title>Practical 2 - Autoscript</title></head>
   <body>
       <h1>Deployed via Automated Script on Ubuntu VM</h1>
   </body>
   </html>
   EOF

   sudo chown -R www-data:www-data /var/www/html
   sudo chmod -R 755 /var/www/html

   echo "[4/4] Restarting Nginx..."
   sudo systemctl restart nginx
   sudo systemctl enable nginx

   echo "Deployment complete! Visit: http://$(hostname -I | awk '{print $1}')"
   ```

3. **Grant execution permissions:**
   ```bash
   chmod +x deploy_static.sh
   ```

4. **Run the script:**
   ```bash
   ./deploy_static.sh
   ```

5. **Verify deployment:**
   - Access `http://<VM_IP>` from the browser.

---

## Practical 3: Deploy Static Website Manually on AWS EC2 (Ubuntu)

### Objective
Provision an Ubuntu EC2 instance on AWS and manually configure it to host a static website.

### Prerequisites
- AWS Account.
- Key pair (`.pem` file) created.
- Security Group allowing Port 22 (SSH) and Port 80 (HTTP).

### Step-by-Step Instructions

1. **Launch EC2 Instance:**
   - AMI: Ubuntu 22.04 LTS / 24.04 LTS.
   - Instance Type: `t2.micro` or `t3.micro`.
   - Security Group Rules:
     - SSH (Port 22) -> My IP (or 0.0.0.0/0)
     - HTTP (Port 80) -> Anywhere (0.0.0.0/0)

2. **Connect to EC2 Instance via SSH:**
   ```bash
   chmod 400 my-key.pem
   ssh -i "my-key.pem" ubuntu@<EC2-PUBLIC-IP>
   ```

3. **Update instance packages:**
   ```bash
   sudo apt update -y && sudo apt upgrade -y
   ```

4. **Install Nginx:**
   ```bash
   sudo apt install nginx -y
   sudo systemctl start nginx
   sudo systemctl enable nginx
   ```

5. **Deploy web assets:**
   ```bash
   sudo rm -rf /var/www/html/*
   cat << 'EOF' | sudo tee /var/www/html/index.html
   <!DOCTYPE html>
   <html>
   <head><title>Practical 3 - AWS EC2 Manual</title></head>
   <body>
       <h1>Deployed Manually on AWS EC2 Ubuntu</h1>
   </body>
   </html>
   EOF
   ```

6. **Verify deployment:**
   - Open browser and navigate to: `http://<EC2-PUBLIC-IP>`.

---

## Practical 4: Deploy Static Website via Automated Script on AWS EC2 (Ubuntu)

### Objective
Write and run a shell script on an existing AWS EC2 instance that installs dependencies and deploys a static website (cloning from a repository or creating assets).

### Prerequisites
- AWS EC2 Ubuntu instance running with Port 22 and Port 80 open.

### Step-by-Step Instructions

1. **SSH into the EC2 instance:**
   ```bash
   ssh -i "my-key.pem" ubuntu@<EC2-PUBLIC-IP>
   ```

2. **Create the automation script:**
   ```bash
   nano setup_ec2_site.sh
   ```

3. **Paste the script:**
   ```bash
   #!/usr/bin/env bash
   set -e

   echo "Updating system..."
   sudo apt update -y

   echo "Installing Nginx & Git..."
   sudo apt install nginx git -y

   echo "Deploying Static Web Pages..."
   sudo rm -rf /var/www/html/*
   cat << 'EOF' | sudo tee /var/www/html/index.html
   <!DOCTYPE html>
   <html>
   <head><title>Practical 4 - EC2 Autoscript</title></head>
   <body>
       <h1>Deployed using Autoscript on AWS EC2 Ubuntu</h1>
   </body>
   </html>
   EOF

   sudo chown -R www-data:www-data /var/www/html
   sudo systemctl restart nginx
   sudo systemctl enable nginx

   echo "Done. Server Public IP: $(curl -s http://checkip.amazonaws.com)"
   ```

4. **Execute the script:**
   ```bash
   chmod +x setup_ec2_site.sh
   ./setup_ec2_site.sh
   ```

5. **Verify deployment:**
   - Visit `http://<EC2-PUBLIC-IP>`.

---

## Practical 5: Deploy Static Website via EC2 User Data on AWS (Ubuntu)

### Objective
Use AWS EC2 User Data (cloud-init script) to deploy the static website automatically at instance launch time without manual SSH.

### Prerequisites
- AWS Account.
- Security Group allowing Port 80 (HTTP).

### Step-by-Step Instructions

1. **Open AWS EC2 Launch Instance Wizard:**
   - Name: `Static-Web-UserData`
   - AMI: Ubuntu Server 22.04 LTS / 24.04 LTS
   - Instance type: `t2.micro`
   - Key pair: Optional (not needed if only viewing via HTTP)
   - Network settings: Allow HTTP traffic from the internet (0.0.0.0/0).

2. **Expand "Advanced details" -> Find "User data" field:**
   Enter the following bootstrap script:
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
   <head><title>Practical 5 - EC2 User Data</title></head>
   <body>
       <h1>Deployed Automatically via AWS EC2 User Data</h1>
   </body>
   </html>
   EOF

   chown -R www-data:www-data /var/www/html
   chmod -R 755 /var/www/html
   ```

3. **Launch the instance.**

4. **Verify deployment:**
   - Wait 2–3 minutes for instance status checks to turn 2/2.
   - Copy Public IPv4 address.
   - Open browser: `http://<EC2-PUBLIC-IP>`.

---

## Practical 6: GitHub Actions + Self-Hosted Runner Deployment on AWS EC2 (Ubuntu)

### Objective
Configure an AWS EC2 instance as a GitHub Actions Self-Hosted Runner and execute a deployment workflow on it.

### Prerequisites
- GitHub repository.
- AWS EC2 instance (Ubuntu) with internet access.

### Step-by-Step Instructions

1. **SSH into AWS EC2:**
   ```bash
   ssh -i "my-key.pem" ubuntu@<EC2-PUBLIC-IP>
   ```

2. **Get Runner configuration token from GitHub:**
   - In GitHub: Go to Repository -> **Settings** -> **Actions** -> **Runners** -> **New self-hosted runner**.
   - Select OS: **Linux**, Architecture: **x64**.

3. **Install and configure Runner on EC2:**
   ```bash
   # Create a directory
   mkdir -p actions-runner && cd actions-runner

   # Download the latest runner package (adjust version if required by GitHub prompt)
   curl -o actions-runner-linux-x64-2.321.0.tar.gz -L https://github.com/actions/runner/releases/download/v2.321.0/actions-runner-linux-x64-2.321.0.tar.gz
   tar xzf ./actions-runner-linux-x64-2.321.0.tar.gz

   # Configure runner (use values provided on GitHub runner page)
   ./config.sh --url https://github.com/<YOUR_USER>/<YOUR_REPO> --token <RUNNER_TOKEN>
   ```

4. **Install and start the runner as a background systemd service:**
   ```bash
   sudo ./svc.sh install
   sudo ./svc.sh start
   sudo ./svc.sh status
   ```
   *(In GitHub Repository Settings -> Runners, runner state will show **Idle** (Green)).*

5. **Create GitHub Actions workflow:**
   In your repository, create `.github/workflows/self-hosted-test.yml`:
   ```yaml
   name: Self-Hosted Runner Test
   on:
     push:
       branches: [ main ]

   jobs:
     deploy:
       runs-on: self-hosted
       steps:
         - name: Checkout repository
           uses: actions/checkout@v4

         - name: Run command on EC2
           run: |
             echo "Runner is working on EC2!"
             hostname
             uname -a
   ```

6. **Trigger and verify:**
   - Push a commit to `main`.
   - Check GitHub Actions tab to observe the job executing on the self-hosted EC2 runner.

---

## Practical 7: Static Website on AWS EC2 using GitHub Actions (CI/CD) using Ubuntu Server + Nginx

### Objective
Create a CI/CD pipeline using GitHub Actions to continuously deploy static website files to an AWS EC2 instance running Nginx via SSH/SCP or self-hosted runner.

### Prerequisites
- AWS EC2 instance running Ubuntu with Nginx installed and Port 80 open.
- GitHub Repository with website files (`index.html`, etc.).

### Step-by-Step Instructions

1. **Prepare EC2 Server:**
   ```bash
   ssh -i "my-key.pem" ubuntu@<EC2-PUBLIC-IP>
   sudo apt update -y && sudo apt install nginx -y
   sudo chown -R ubuntu:ubuntu /var/www/html
   ```

2. **Add GitHub Secrets:**
   In GitHub Repo -> **Settings** -> **Secrets and variables** -> **Actions** -> **New repository secret**:
   - `EC2_HOST`: EC2 Public IP address.
   - `EC2_USER`: `ubuntu`
   - `EC2_SSH_KEY`: Entire contents of your `.pem` private key file.

3. **Create GitHub Actions Workflow:**
   Create `.github/workflows/deploy-nginx.yml`:
   ```yaml
   name: Deploy Static Website to EC2 Nginx

   on:
     push:
       branches: [ main ]

   jobs:
     deploy:
       runs-on: ubuntu-latest
       steps:
         - name: Checkout code
           uses: actions/checkout@v4

         - name: Deploy files to EC2 via SCP/SSH
           uses: appleboy/scp-action@v0.1.7
           with:
             host: ${{ secrets.EC2_HOST }}
             username: ${{ secrets.EC2_USER }}
             key: ${{ secrets.EC2_SSH_KEY }}
             source: "index.html,css,js,images"
             target: "/var/www/html"

         - name: Restart Nginx
           uses: appleboy/ssh-action@v1.0.3
           with:
             host: ${{ secrets.EC2_HOST }}
             username: ${{ secrets.EC2_USER }}
             key: ${{ secrets.EC2_SSH_KEY }}
             script: |
               sudo systemctl restart nginx
   ```

4. **Verify:**
   - Commit changes to `index.html` and push to `main`.
   - Pipeline triggers, uploads files, and restarts Nginx.
   - Check browser at `http://<EC2-PUBLIC-IP>`.

---

## Practical 8: Hosting a Static Website on AWS EC2 using GitHub Actions (CI/CD) using Ubuntu Server + Apache2

### Objective
Create a CI/CD pipeline using GitHub Actions to continuously deploy static website files to an AWS EC2 instance running Apache2.

### Prerequisites
- AWS EC2 instance with Ubuntu.
- Security Group Port 80 (HTTP) and Port 22 (SSH) open.

### Step-by-Step Instructions

1. **Configure EC2 Server for Apache2:**
   ```bash
   ssh -i "my-key.pem" ubuntu@<EC2-PUBLIC-IP>
   sudo apt update -y
   sudo apt install apache2 -y
   sudo systemctl enable apache2
   sudo systemctl start apache2
   sudo chown -R ubuntu:ubuntu /var/www/html
   ```

2. **Add GitHub Secrets:**
   - `EC2_HOST`: EC2 Public IPv4
   - `EC2_USER`: `ubuntu`
   - `EC2_SSH_KEY`: Content of `.pem` private key

3. **Create GitHub Actions Workflow:**
   Create `.github/workflows/deploy-apache.yml`:
   ```yaml
   name: Deploy Static Website to EC2 Apache2

   on:
     push:
       branches: [ main ]

   jobs:
     deploy:
       runs-on: ubuntu-latest
       steps:
         - name: Checkout repository
           uses: actions/checkout@v4

         - name: Copy files to EC2 Apache directory
           uses: appleboy/scp-action@v0.1.7
           with:
             host: ${{ secrets.EC2_HOST }}
             username: ${{ secrets.EC2_USER }}
             key: ${{ secrets.EC2_SSH_KEY }}
             source: "index.html,css,js,images"
             target: "/var/www/html"

         - name: Restart Apache2 service
           uses: appleboy/ssh-action@v1.0.3
           with:
             host: ${{ secrets.EC2_HOST }}
             username: ${{ secrets.EC2_USER }}
             key: ${{ secrets.EC2_SSH_KEY }}
             script: |
               sudo systemctl restart apache2
   ```

4. **Verify:**
   - Push code update to `main`.
   - Access `http://<EC2-PUBLIC-IP>` to view Apache2 deployed site.

---

## Practical 9: Create a Portfolio Container and Upload into Docker Hub

### Objective
Build a Docker image for a static portfolio website and push it to Docker Hub.

### Prerequisites
- Docker installed on local machine or EC2.
- Docker Hub account created.

### Step-by-Step Instructions

1. **Prepare portfolio files in a directory:**
   ```bash
   mkdir portfolio && cd portfolio
   ```

2. **Create `index.html`:**
   ```html
   <!DOCTYPE html>
   <html>
   <head><title>My Portfolio</title></head>
   <body>
       <h1>Welcome to My Portfolio</h1>
       <p>DevOps Engineer | Cloud Enthusiast</p>
   </body>
   </html>
   ```

3. **Create `Dockerfile`:**
   ```dockerfile
   FROM nginx:alpine
   COPY index.html /usr/share/nginx/html/index.html
   EXPOSE 80
   CMD ["nginx", "-g", "daemon off;"]
   ```

4. **Log in to Docker Hub:**
   ```bash
   docker login -u <DOCKERHUB_USERNAME>
   ```

5. **Build the Docker image:**
   ```bash
   docker build -t <DOCKERHUB_USERNAME>/portfolio:v1 .
   ```

6. **Test container locally:**
   ```bash
   docker run -d -p 8080:80 --name test-portfolio <DOCKERHUB_USERNAME>/portfolio:v1
   ```
   Open `http://localhost:8080` to verify. Clean up:
   ```bash
   docker stop test-portfolio && docker rm test-portfolio
   ```

7. **Push image to Docker Hub:**
   ```bash
   docker push <DOCKERHUB_USERNAME>/portfolio:v1
   ```

8. **Verify:**
   - Log into [hub.docker.com](https://hub.docker.com) and confirm the repository and tag exist.

---

## Practical 10: CI/CD Pipeline using GitHub Actions, Docker, Docker Hub, and AWS EC2 (Nginx)

### Objective
Automate building an Nginx-based Docker image on code push, pushing it to Docker Hub, and deploying it as a container on an AWS EC2 instance.

### Prerequisites
- Docker Hub account.
- AWS EC2 instance with Docker installed and port 80 opened.

### Step-by-Step Instructions

1. **Install Docker on AWS EC2:**
   ```bash
   ssh -i "my-key.pem" ubuntu@<EC2-PUBLIC-IP>
   sudo apt update -y
   sudo apt install docker.io -y
   sudo usermod -aG docker ubuntu
   newgrp docker
   sudo systemctl enable docker
   sudo systemctl start docker
   ```

2. **Configure GitHub Repository Secrets:**
   - `DOCKERHUB_USERNAME`: Your Docker Hub username.
   - `DOCKERHUB_TOKEN`: Docker Hub Personal Access Token.
   - `EC2_HOST`: EC2 Public IP.
   - `EC2_USER`: `ubuntu`
   - `EC2_SSH_KEY`: Content of `.pem` private key.

3. **Project Files:**
   - `Dockerfile`:
     ```dockerfile
     FROM nginx:alpine
     COPY . /usr/share/nginx/html
     EXPOSE 80
     CMD ["nginx", "-g", "daemon off;"]
     ```
   - `index.html`:
     ```html
     <h1>App running on Docker + Nginx via GitHub Actions</h1>
     ```

4. **Create GitHub Actions Workflow:**
   Create `.github/workflows/docker-nginx-deploy.yml`:
   ```yaml
   name: Docker Nginx CI/CD to EC2

   on:
     push:
       branches: [ main ]

   jobs:
     build-and-push:
       runs-on: ubuntu-latest
       steps:
         - name: Checkout code
           uses: actions/checkout@v4

         - name: Log in to Docker Hub
           uses: docker/login-action@v3
           with:
             username: ${{ secrets.DOCKERHUB_USERNAME }}
             password: ${{ secrets.DOCKERHUB_TOKEN }}

         - name: Build and Push Docker Image
           uses: docker/build-push-action@v5
           with:
             context: .
             push: true
             tags: ${{ secrets.DOCKERHUB_USERNAME }}/nginx-web:latest

     deploy-to-ec2:
       needs: build-and-push
       runs-on: ubuntu-latest
       steps:
         - name: SSH and deploy container
           uses: appleboy/ssh-action@v1.0.3
           with:
             host: ${{ secrets.EC2_HOST }}
             username: ${{ secrets.EC2_USER }}
             key: ${{ secrets.EC2_SSH_KEY }}
             script: |
               docker pull ${{ secrets.DOCKERHUB_USERNAME }}/nginx-web:latest
               docker stop nginx-app || true
               docker rm nginx-app || true
               docker run -d --name nginx-app -p 80:80 ${{ secrets.DOCKERHUB_USERNAME }}/nginx-web:latest
   ```

5. **Verify:**
   - Commit and push to `main`.
   - Inspect GitHub Actions workflow execution.
   - Check `http://<EC2-PUBLIC-IP>`.

---

## Practical 11: CI/CD Pipeline using GitHub Actions, Docker, Docker Hub, and AWS EC2 (Apache2)

### Objective
Automate building an Apache-based Docker container image, publishing to Docker Hub, and deploying to AWS EC2 via GitHub Actions.

### Prerequisites
- Docker Hub account and token.
- AWS EC2 instance with Docker installed and port 80 opened.

### Step-by-Step Instructions

1. **Project Files:**
   - `Dockerfile`:
     ```dockerfile
     FROM httpd:2.4-alpine
     COPY . /usr/local/apache2/htdocs/
     EXPOSE 80
     ```
   - `index.html`:
     ```html
     <h1>App running on Docker + Apache2 via GitHub Actions</h1>
     ```

2. **Configure GitHub Secrets:**
   - `DOCKERHUB_USERNAME`
   - `DOCKERHUB_TOKEN`
   - `EC2_HOST`
   - `EC2_USER` (`ubuntu`)
   - `EC2_SSH_KEY`

3. **Create GitHub Actions Workflow:**
   Create `.github/workflows/docker-apache-deploy.yml`:
   ```yaml
   name: Docker Apache CI/CD to EC2

   on:
     push:
       branches: [ main ]

   jobs:
     build-and-push:
       runs-on: ubuntu-latest
       steps:
         - name: Checkout code
           uses: actions/checkout@v4

         - name: Log in to Docker Hub
           uses: docker/login-action@v3
           with:
             username: ${{ secrets.DOCKERHUB_USERNAME }}
             password: ${{ secrets.DOCKERHUB_TOKEN }}

         - name: Build and Push Docker Image
           uses: docker/build-push-action@v5
           with:
             context: .
             push: true
             tags: ${{ secrets.DOCKERHUB_USERNAME }}/apache-web:latest

     deploy-to-ec2:
       needs: build-and-push
       runs-on: ubuntu-latest
       steps:
         - name: SSH and deploy container
           uses: appleboy/ssh-action@v1.0.3
           with:
             host: ${{ secrets.EC2_HOST }}
             username: ${{ secrets.EC2_USER }}
             key: ${{ secrets.EC2_SSH_KEY }}
             script: |
               docker pull ${{ secrets.DOCKERHUB_USERNAME }}/apache-web:latest
               docker stop apache-app || true
               docker rm apache-app || true
               docker run -d --name apache-app -p 80:80 ${{ secrets.DOCKERHUB_USERNAME }}/apache-web:latest
   ```

4. **Verify:**
   - Push to `main`.
   - Access `http://<EC2-PUBLIC-IP>`.

---

## Practical 12: Full Stack CI/CD Pipeline using React + Node.js + MongoDB + Docker + GitHub Actions + AWS EC2

### Objective
Create a multi-container full-stack application pipeline that automatically builds frontend and backend images, pushes them to Docker Hub, and orchestrates them on AWS EC2 using Docker Compose.

### Directory Structure
```text
fullstack-app/
├── client/
│   ├── Dockerfile
│   └── ... (React source files)
├── server/
│   ├── Dockerfile
│   └── ... (Node.js Express source files)
├── docker-compose.yml
└── .github/
    └── workflows/
        └── fullstack-deploy.yml
```

### Step-by-Step Instructions

1. **Client Dockerfile (`client/Dockerfile`):**
   ```dockerfile
   FROM node:18-alpine AS build
   WORKDIR /app
   COPY package*.json ./
   RUN npm install
   COPY . .
   RUN npm run build

   FROM nginx:alpine
   COPY --from=build /app/build /usr/share/nginx/html
   EXPOSE 80
   CMD ["nginx", "-g", "daemon off;"]
   ```

2. **Server Dockerfile (`server/Dockerfile`):**
   ```dockerfile
   FROM node:18-alpine
   WORKDIR /app
   COPY package*.json ./
   RUN npm install --production
   COPY . .
   EXPOSE 5000
   CMD ["node", "index.js"]
   ```

3. **Docker Compose Configuration (`docker-compose.yml`):**
   ```yaml
   version: '3.8'

   services:
     mongodb:
       image: mongo:6.0
       restart: always
       ports:
         - "27017:27017"
       volumes:
         - mongo-data:/data/db

     server:
       image: ${DOCKERHUB_USERNAME}/mern-server:latest
       restart: always
       ports:
         - "5000:5000"
       environment:
         - MONGO_URI=mongodb://mongodb:27017/appdb
       depends_on:
         - mongodb

     client:
       image: ${DOCKERHUB_USERNAME}/mern-client:latest
       restart: always
       ports:
         - "80:80"
       depends_on:
         - server

   volumes:
     mongo-data:
   ```

4. **EC2 Instance Setup:**
   ```bash
   ssh -i "my-key.pem" ubuntu@<EC2-PUBLIC-IP>
   sudo apt update -y
   sudo apt install docker.io docker-compose -y
   sudo usermod -aG docker ubuntu
   ```

5. **GitHub Secrets:**
   - `DOCKERHUB_USERNAME`
   - `DOCKERHUB_TOKEN`
   - `EC2_HOST`
   - `EC2_USER` (`ubuntu`)
   - `EC2_SSH_KEY`

6. **Create GitHub Actions Workflow (`.github/workflows/fullstack-deploy.yml`):**
   ```yaml
   name: Full Stack CI/CD Deployment

   on:
     push:
       branches: [ main ]

   jobs:
     build-and-push:
       runs-on: ubuntu-latest
       steps:
         - name: Checkout repository
           uses: actions/checkout@v4

         - name: Log in to Docker Hub
           uses: docker/login-action@v3
           with:
             username: ${{ secrets.DOCKERHUB_USERNAME }}
             password: ${{ secrets.DOCKERHUB_TOKEN }}

         - name: Build and push Client image
           uses: docker/build-push-action@v5
           with:
             context: ./client
             push: true
             tags: ${{ secrets.DOCKERHUB_USERNAME }}/mern-client:latest

         - name: Build and push Server image
           uses: docker/build-push-action@v5
           with:
             context: ./server
             push: true
             tags: ${{ secrets.DOCKERHUB_USERNAME }}/mern-server:latest

     deploy-ec2:
       needs: build-and-push
       runs-on: ubuntu-latest
       steps:
         - name: Checkout repository for compose file
           uses: actions/checkout@v4

         - name: Copy docker-compose.yml to EC2
           uses: appleboy/scp-action@v0.1.7
           with:
             host: ${{ secrets.EC2_HOST }}
             username: ${{ secrets.EC2_USER }}
             key: ${{ secrets.EC2_SSH_KEY }}
             source: "docker-compose.yml"
             target: "/home/ubuntu/app"

         - name: Pull images and launch stack
           uses: appleboy/ssh-action@v1.0.3
           with:
             host: ${{ secrets.EC2_HOST }}
             username: ${{ secrets.EC2_USER }}
             key: ${{ secrets.EC2_SSH_KEY }}
             script: |
               cd /home/ubuntu/app
               export DOCKERHUB_USERNAME=${{ secrets.DOCKERHUB_USERNAME }}
               docker-compose pull
               docker-compose down
               docker-compose up -d
   ```

7. **Verify:**
   - Push commit to `main`.
   - Monitor Actions tab.
   - Navigate to `http://<EC2-PUBLIC-IP>` (React UI) and `http://<EC2-PUBLIC-IP>:5000` (API status).

---

## Practical 13: Jenkins Installation and Job Setup (Windows Local PC)

### Objective
Install Jenkins on a Windows system, configure initial admin credentials, and set up a basic build job.

### Prerequisites
- Java JDK 17 or 21 installed (`java -version`).
- Windows administrator rights.

### Step-by-Step Instructions

1. **Install Java (if not already installed):**
   - Download Eclipse Temurin JDK 17 or 21 from [adoptium.net](https://adoptium.net/).
   - Add Java to system environment variable `PATH`.

2. **Download and Install Jenkins:**
   - Download Windows MSI installer from [jenkins.io/download](https://www.jenkins.io/download/).
   - Run the installer.
   - Select service logon credentials (Local System or specific user account).
   - Set Port (default `8080`, click "Test Port").
   - Complete installation.

3. **Unlock Jenkins:**
   - Open browser: `http://localhost:8080`.
   - Retrieve initial admin password from file displayed on screen:
     ```powershell
     Get-Content "C:\Program Files\Jenkins\secrets\initialAdminPassword"
     ```
   - Paste the password and click **Continue**.

4. **Plugin Installation & Admin Account:**
   - Click **Install suggested plugins**.
   - Create First Admin User (Username, Password, Full Name, Email).
   - Save and Finish.

5. **Create First Jenkins Freestyle Job:**
   - Click **New Item**.
   - Enter Name: `test-local-job`.
   - Select **Freestyle project** and click **OK**.
   - Under **Build Steps**, click **Add build step** -> **Execute Windows batch command**.
   - Command:
     ```bat
     echo "Jenkins is successfully configured on Windows!"
     hostname
     date /t
     ```
   - Click **Save**.

6. **Run and Verify:**
   - Click **Build Now**.
   - Click on Build `#1` -> **Console Output** to verify `SUCCESS`.

---

## Practical 14: Jenkins Deployment on AWS EC2 with CI/CD Pipeline

### Objective
Install Jenkins and Docker on an AWS EC2 Ubuntu instance, and configure a complete CI/CD pipeline (`GitHub -> Jenkins -> Docker -> Nginx`).

### Prerequisites
- AWS EC2 instance (Ubuntu 22.04+, minimum `t2.medium` recommended for Jenkins + Docker).
- Security Group inbound rules:
  - Port 22 (SSH)
  - Port 8080 (Jenkins UI)
  - Port 80 (Web application)

### Step-by-Step Instructions

1. **SSH into EC2:**
   ```bash
   ssh -i "my-key.pem" ubuntu@<EC2-PUBLIC-IP>
   ```

2. **Install Java and Jenkins:**
   ```bash
   sudo apt update -y
   sudo apt install openjdk-17-jdk -y

   # Add Jenkins repo keys
   sudo wget -O /usr/share/keyrings/jenkins-keyring.asc https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key
   echo "deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] https://pkg.jenkins.io/debian-stable binary/" | sudo tee /etc/apt/sources.list.d/jenkins.list > /dev/null

   sudo apt update -y
   sudo apt install jenkins -y
   sudo systemctl enable jenkins
   sudo systemctl start jenkins
   ```

3. **Install Docker and assign permissions to Jenkins user:**
   ```bash
   sudo apt install docker.io -y
   sudo usermod -aG docker jenkins
   sudo usermod -aG docker ubuntu
   sudo systemctl restart docker
   sudo systemctl restart jenkins
   ```

4. **Unlock and Setup Jenkins:**
   - Get unlock key:
     ```bash
     sudo cat /var/lib/jenkins/secrets/initialAdminPassword
     ```
   - Access `http://<EC2-PUBLIC-IP>:8080` in browser.
   - Complete plugin setup and admin account creation.

5. **Install Required Jenkins Plugins:**
   - Go to **Manage Jenkins** -> **Plugins** -> **Available plugins**.
   - Search & install:
     - Git Plugin
     - Docker Pipeline

6. **Create `Jenkinsfile` in GitHub Repository:**
   ```groovy
   pipeline {
       agent any

       stages {
           stage('Clone Repository') {
               steps {
                   checkout scm
               }
           }

           stage('Build Docker Image') {
               steps {
                   sh 'docker build -t static-nginx-site:latest .'
               }
           }

           stage('Deploy Container') {
               steps {
                   sh '''
                       docker stop static-web-app || true
                       docker rm static-web-app || true
                       docker run -d --name static-web-app -p 80:80 static-nginx-site:latest
                   '''
               }
           }
       }

       post {
           success {
               echo "Application successfully deployed to http://${env.BUILD_URL}"
           }
           failure {
               echo "Deployment failed."
           }
       }
   }
   ```

7. **Create Jenkins Pipeline Job:**
   - In Jenkins Dashboard -> **New Item**.
   - Name: `ec2-docker-nginx-pipeline` -> Select **Pipeline** -> Click **OK**.
   - Under **Build Triggers**, select **GitHub hook trigger for GITScm polling**.
   - Under **Pipeline**:
     - Definition: `Pipeline script from SCM`
     - SCM: `Git`
     - Repository URL: `https://github.com/<YOUR_USER>/<YOUR_REPO>.git`
     - Branch: `*/main`
     - Script Path: `Jenkinsfile`
   - Click **Save**.

8. **Trigger and Verify:**
   - Click **Build Now** in Jenkins (or configure GitHub Webhook to trigger on push).
   - Verify all stages pass in the Pipeline Stage View.
   - Open browser: `http://<EC2-PUBLIC-IP>` to view the deployed Nginx application.

