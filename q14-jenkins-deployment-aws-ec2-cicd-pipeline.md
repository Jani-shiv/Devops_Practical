# Practical 14: Jenkins Deployment on AWS EC2 including Jenkins CI/CD Pipeline (GitHub -> Jenkins -> Docker -> Nginx)

## Objective
Install Jenkins and Docker on an AWS EC2 Ubuntu instance, understand Docker daemon security implications for the `jenkins` user, configure GitHub webhook/polling triggers accurately, and execute a pipeline that containerizes and deploys an Nginx website.

## Prerequisites
- AWS EC2 Ubuntu instance (`t2.medium` recommended: 2 vCPUs, 4GB RAM).
- Security Group open on:
  - Port 22 (SSH) -> restricted to your IP.
  - Port 8080 (Jenkins UI) -> restricted to your IP (or open if testing external webhooks).
  - Port 80 (Nginx Application) -> open to anywhere (`0.0.0.0/0`).

---

> [!CAUTION]
> **Docker Daemon Security in Jenkins**:
> Adding `jenkins` to the `docker` group (`usermod -aG docker jenkins`) grants root-equivalent privileges on the host, as any process able to communicate with `/var/run/docker.sock` can launch privileged containers with host volume mounts. This is standard practice in isolated lab/sandbox instances, but should not be used on multi-tenant production servers without rootless Docker or isolated agent nodes.

---

## What We Need To Do
1. Install Java 17 LTS, Jenkins, and Docker on EC2.
2. Grant Docker daemon access to the `jenkins` user and restart the Jenkins service.
3. Install necessary Jenkins plugins (Git, Docker Pipeline).
4. Configure pipeline triggers (clear explanation of GitHub Webhook requirements vs SCM Polling).
5. Author a `Jenkinsfile` in GitHub with build and deployment stages.
6. Trigger the pipeline and verify deployment on Port 80.

---

## How To Do It (Step-by-Step Instructions)

### Step 1: Install Java, Jenkins, and Docker on EC2
Connect via SSH:
```bash
ssh -i "my-key.pem" ubuntu@<EC2-PUBLIC-IP>
```

Install OpenJDK 17 and official Jenkins repository:
```bash
sudo apt update -y
sudo apt install -y openjdk-17-jdk docker.io curl wget

sudo wget -O /usr/share/keyrings/jenkins-keyring.asc https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key
echo "deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] https://pkg.jenkins.io/debian-stable binary/" | sudo tee /etc/apt/sources.list.d/jenkins.list > /dev/null

sudo apt update -y
sudo apt install -y jenkins
```

Configure Docker permissions and restart services:
```bash
sudo usermod -aG docker jenkins
sudo usermod -aG docker ubuntu
sudo systemctl enable docker
sudo systemctl restart docker
sudo systemctl enable jenkins
sudo systemctl restart jenkins
```

Verify `jenkins` user has Docker socket access:
```bash
sudo -u jenkins docker ps
```
*(Should output the Docker process header without permission denied errors).*

### Step 2: Unlock Jenkins & Install Plugins
1. Fetch the initial admin secret:
```bash
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```
2. Open browser: `http://<EC2-PUBLIC-IP>:8080`.
3. Paste key, choose **Install suggested plugins**, and create your admin account.
4. Go to **Manage Jenkins** -> **Plugins** -> **Available plugins**.
5. Install:
   - **Docker Pipeline**

### Step 3: Trigger Configuration (Webhook vs Polling)
> **Note on Webhooks vs Polling**:
> - **GitHub Webhook Trigger**: Requires your EC2 Port `8080` to be reachable by GitHub's public webhook servers. In GitHub repo -> **Settings** -> **Webhooks** -> **Add webhook** -> Payload URL: `http://<EC2-PUBLIC-IP>:8080/github-webhook/`.
> - **Poll SCM Alternative**: If your EC2 instance is private or behind a firewall, use **Poll SCM** with schedule `H/5 * * * *` (checks every 5 minutes).

### Step 4: Add Files in GitHub Repository
**`Dockerfile`**:
```dockerfile
FROM nginx:alpine
COPY index.html /usr/share/nginx/html/index.html
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

**`index.html`**:
```html
<!DOCTYPE html>
<html>
<head><title>Jenkins CI/CD</title></head>
<body>
    <h1>Deployed via Jenkins CI/CD Pipeline on AWS EC2!</h1>
</body>
</html>
```

**`Jenkinsfile`**:
```groovy
pipeline {
    agent any

    stages {
        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t static-nginx-site:latest .'
            }
        }

        stage('Deploy Docker Container') {
            steps {
                sh '''
                    echo "Stopping running container if exists..."
                    docker stop static-web-app || true
                    docker rm static-web-app || true
                    echo "Launching updated container..."
                    docker run -d --name static-web-app --restart always -p 80:80 static-nginx-site:latest
                '''
            }
        }

        stage('Verify Deployment Health') {
            steps {
                sh '''
                    sleep 3
                    if [ "$(docker inspect -f '{{.State.Status}}' static-web-app)" != "running" ]; then
                        echo "Container failed to start!" >&2
                        exit 1
                    fi
                    curl -I http://localhost
                '''
            }
        }
    }

    post {
        success {
            echo "CI/CD Pipeline executed successfully! Web app is live."
        }
        failure {
            echo "Pipeline execution failed. Check console output for logs."
        }
    }
}
```

### Step 5: Configure Jenkins Pipeline Job
1. On Jenkins Dashboard, click **New Item**.
2. Name: `ec2-docker-nginx-pipeline` -> Select **Pipeline** -> Click **OK**.
3. Under **Build Triggers**:
   - Check **GitHub hook trigger for GITScm polling** (or **Poll SCM** if using periodic polling).
4. Under **Pipeline**:
   - **Definition**: `Pipeline script from SCM`
   - **SCM**: `Git`
   - **Repository URL**: `https://github.com/<YOUR_USER>/<YOUR_REPO>.git`
   - **Branch Specifier**: `*/main`
   - **Script Path**: `Jenkinsfile`
5. Click **Save**.

### Step 6: Execute and Verify
1. Click **Build Now** to execute the pipeline.
2. Inspect the Pipeline Stage View: all stages (**Checkout Code**, **Build Docker Image**, **Deploy Docker Container**, **Verify Deployment Health**) will show green.
3. Open `http://<EC2-PUBLIC-IP>` in your browser to verify the deployed application.
