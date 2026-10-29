# Practical 14: Jenkins Deployment on AWS EC2 including Jenkins CI/CD Pipeline (GitHub -> Jenkins -> Docker -> Nginx)

## Objective
Install and configure Jenkins and Docker on an AWS EC2 Ubuntu instance, and build a complete CI/CD pipeline triggered from GitHub that builds a Docker container and deploys an Nginx website.

## Prerequisites
- AWS EC2 instance (Ubuntu 22.04 LTS, `t2.medium` recommended).
- Security Group open on:
  - Port 22 (SSH)
  - Port 8080 (Jenkins UI)
  - Port 80 (Nginx Web Application)

## What We Need To Do
1. Install Java 17, Jenkins, and Docker on EC2.
2. Grant Docker permissions to the `jenkins` system user.
3. Complete initial Jenkins setup and install Git & Docker plugins.
4. Create a `Jenkinsfile` in your GitHub repository.
5. Create a Jenkins Pipeline job linked to your GitHub repo.
6. Trigger the pipeline and verify deployment on Port 80.

## How To Do It (Step-by-Step Instructions)

### Step 1: Install Java and Jenkins on EC2
Connect to EC2:
```bash
ssh -i "my-key.pem" ubuntu@<EC2-PUBLIC-IP>
```
Install OpenJDK 17 and Jenkins repository:
```bash
sudo apt update -y
sudo apt install openjdk-17-jdk -y

sudo wget -O /usr/share/keyrings/jenkins-keyring.asc https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key
echo "deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] https://pkg.jenkins.io/debian-stable binary/" | sudo tee /etc/apt/sources.list.d/jenkins.list > /dev/null

sudo apt update -y
sudo apt install jenkins -y
sudo systemctl enable jenkins
sudo systemctl start jenkins
```

### Step 2: Install Docker and Configure Jenkins Permissions
```bash
sudo apt install docker.io -y
sudo usermod -aG docker jenkins
sudo usermod -aG docker ubuntu
sudo systemctl restart docker
sudo systemctl restart jenkins
```

### Step 3: Unlock Jenkins and Install Plugins
1. Retrieve initial admin password:
```bash
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```
2. Navigate in browser to `http://<EC2-PUBLIC-IP>:8080`.
3. Paste password, select **Install suggested plugins**, and create the admin user.
4. Navigate to **Manage Jenkins** -> **Plugins** -> **Available plugins**.
5. Install:
   - **Docker Pipeline**
   - **Git Plugin** (installed by default)

### Step 4: Create Project Files in GitHub Repository
**`Dockerfile`**:
```dockerfile
FROM nginx:alpine
COPY . /usr/share/nginx/html
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

**`index.html`**:
```html
<!DOCTYPE html>
<html>
<head><title>Jenkins CI/CD</title></head>
<body>
    <h1>Deployed via Jenkins CI/CD Pipeline on EC2!</h1>
</body>
</html>
```

**`Jenkinsfile`**:
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
            echo "Deployment completed successfully!"
        }
        failure {
            echo "Deployment failed."
        }
    }
}
```

### Step 5: Configure Jenkins Pipeline Job
1. In Jenkins, click **New Item**.
2. Name: `ec2-docker-nginx-pipeline` -> Choose **Pipeline** -> Click **OK**.
3. Under **Build Triggers**, select **GitHub hook trigger for GITScm polling**.
4. Scroll to **Pipeline**:
   - **Definition**: `Pipeline script from SCM`
   - **SCM**: `Git`
   - **Repository URL**: `https://github.com/<YOUR_USER>/<YOUR_REPO>.git`
   - **Branch Specifier**: `*/main`
   - **Script Path**: `Jenkinsfile`
5. Click **Save**.

### Step 6: Trigger and Verify
1. Click **Build Now** in Jenkins.
2. Monitor all 3 stages: Clone -> Build -> Deploy.
3. Open browser:
```text
http://<EC2-PUBLIC-IP>
```
You will see the running static website served through Docker and Nginx.

