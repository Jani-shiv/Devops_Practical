# Practical 6: GitHub Actions + Self-Hosted Runner Deployment on AWS EC2 (Ubuntu)

## Objective
Set up an AWS EC2 instance running Ubuntu as a GitHub Actions Self-Hosted Runner and execute CI/CD workflows directly on it.

## Prerequisites
- GitHub repository where you have Admin/Settings privileges.
- AWS EC2 Ubuntu instance with internet access.

## What We Need To Do
1. Retrieve Runner token from GitHub repository settings.
2. Download and configure the GitHub Actions Runner on the EC2 instance.
3. Install and start the Runner as a persistent `systemd` service.
4. Create a GitHub Actions workflow with `runs-on: self-hosted`.
5. Trigger the workflow and verify execution directly on EC2.

## How To Do It (Step-by-Step Instructions)

### Step 1: Connect to EC2 instance
```bash
ssh -i "my-key.pem" ubuntu@<EC2-PUBLIC-IP>
```

### Step 2: Get Runner Token from GitHub
1. In your GitHub repository, go to **Settings** -> **Actions** -> **Runners**.
2. Click **New self-hosted runner**.
3. Select **Linux** as the Runner image and **x64** architecture.

### Step 3: Install GitHub Actions Runner on EC2
Run the commands displayed in GitHub:
```bash
# Create directory and enter it
mkdir -p actions-runner && cd actions-runner

# Download runner package (adjust version matching GitHub's prompt)
curl -o actions-runner-linux-x64-2.321.0.tar.gz -L https://github.com/actions/runner/releases/download/v2.321.0/actions-runner-linux-x64-2.321.0.tar.gz
tar xzf ./actions-runner-linux-x64-2.321.0.tar.gz

# Configure the runner
./config.sh --url https://github.com/<YOUR_GITHUB_USER>/<YOUR_REPO> --token <YOUR_RUNNER_TOKEN>
```
Press `Enter` to accept default name and labels.

### Step 4: Run as a background service
```bash
sudo ./svc.sh install
sudo ./svc.sh start
sudo ./svc.sh status
```
*In GitHub Settings -> Actions -> Runners, the runner status should now display **Idle** (Green).*

### Step 5: Create workflow file
In your repository, create `.github/workflows/self-hosted-test.yml`:
```yaml
name: Self-Hosted Runner Deployment Test

on:
  push:
    branches: [ main ]

jobs:
  test-runner:
    runs-on: self-hosted
    steps:
      - name: Checkout Code
        uses: actions/checkout@v4

      - name: Verify Runner Host Details
        run: |
          echo "Workflow executed on Self-Hosted EC2 Runner!"
          hostname
          uname -a
```

### Step 6: Verify deployment
Commit and push changes to `main`. Open the **Actions** tab on GitHub to observe job logs running on your EC2 instance.

