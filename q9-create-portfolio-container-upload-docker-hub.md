# Practical 9: Create a Portfolio Container and Upload into Docker Hub

## Objective
Package a static personal portfolio website into a Docker container image using Nginx, test it locally with health verification, and securely push it to Docker Hub using Access Tokens without committing credentials.

## Prerequisites
- Docker installed and running locally or on a Linux VM.
- Docker Hub account with a Personal Access Token (PAT) created from **Account Settings** -> **Security** -> **New Access Token**.

---

> [!SECURITY]
> Never hardcode passwords or personal access tokens in Dockerfiles or shell scripts. Always use `docker login` with environment variables or standard interactive prompts.

---

## What We Need To Do
1. Create a portfolio website structure (`index.html`, `style.css`).
2. Write a minimal, production-grade `Dockerfile` based on `nginx:alpine`.
3. Build and tag the container image locally with both version tag (`v1.0.0`) and `latest`.
4. Test the running container and verify using `curl http://localhost:8080`.
5. Authenticate securely to Docker Hub and push both image tags.

---

## How To Do It (Step-by-Step Instructions)

### Step 1: Create Project Structure
```bash
mkdir -p portfolio && cd portfolio
```

Create `index.html`:
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>DevOps Portfolio</title>
    <link rel="stylesheet" href="style.css">
</head>
<body>
    <header>
        <h1>John Doe | DevOps Engineer</h1>
        <p>CI/CD, Docker, Kubernetes, AWS & Infrastructure as Code</p>
    </header>
    <main>
        <div class="card">
            <h2>About Me</h2>
            <p>Passionate about cloud automation, container orchestration, and continuous delivery pipelines.</p>
        </div>
    </main>
</body>
</html>
```

Create `style.css`:
```css
body { font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif; background: #0f172a; color: #f8fafc; text-align: center; margin: 0; padding: 2rem; }
header { margin-bottom: 2rem; }
.card { background: #1e293b; padding: 1.5rem; border-radius: 8px; max-width: 500px; margin: 0 auto; box-shadow: 0 4px 6px -1px rgba(0,0,0,0.3); }
h1 { color: #38bdf8; }
```

### Step 2: Create Dockerfile
```dockerfile
FROM nginx:alpine
# Copy static web assets into standard Nginx web root
COPY . /usr/share/nginx/html
EXPOSE 80
STOPSIGNAL SIGQUIT
CMD ["nginx", "-g", "daemon off;"]
```

### Step 3: Build the Docker Image
Replace `<DOCKERHUB_USERNAME>` with your actual Docker Hub username:
```bash
docker build -t <DOCKERHUB_USERNAME>/portfolio:v1.0.0 -t <DOCKERHUB_USERNAME>/portfolio:latest .
```

### Step 4: Run and Test Locally
Launch test container on port `8080`:
```bash
docker run -d --name test-portfolio -p 8080:80 <DOCKERHUB_USERNAME>/portfolio:v1.0.0
```

Verify HTTP response:
```bash
curl -I http://localhost:8080
```
Open `http://localhost:8080` in your browser. Clean up the test container:
```bash
docker stop test-portfolio && docker rm test-portfolio
```

### Step 5: Authenticate and Push to Docker Hub
Authenticate using your Docker Hub username and Personal Access Token:
```bash
echo "<DOCKERHUB_PERSONAL_ACCESS_TOKEN>" | docker login -u <DOCKERHUB_USERNAME> --password-stdin
```

Push both tags:
```bash
docker push <DOCKERHUB_USERNAME>/portfolio:v1.0.0
docker push <DOCKERHUB_USERNAME>/portfolio:latest
```

### Step 6: Verify on Docker Hub
Log into [hub.docker.com](https://hub.docker.com), navigate to your repositories, and confirm both `v1.0.0` and `latest` tags are visible.
