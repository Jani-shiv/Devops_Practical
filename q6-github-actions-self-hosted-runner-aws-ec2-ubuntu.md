# Practical 6: GitHub Actions + Self-Hosted Runner Deployment on AWS EC2 (Ubuntu)

## Objective
Set up an AWS EC2 instance running Ubuntu as a GitHub Actions Self-Hosted Runner, dynamic download instructions, persistent service registration, and understanding self-hosted runner security boundaries.

## Prerequisites
- GitHub repository with administrative rights (access to **Settings** -> **Actions**).
- Dedicated AWS EC2 Ubuntu instance (do not use production infrastructure for untrusted workflow execution).
- Security Group allowing SSH (Port 22) from your IP.

---

> [!CAUTION]
> **Self-Hosted Runner Security Policy**:
> Never use self-hosted runners on **public repositories** without fork PR protections. Anyone opening a pull request could execute arbitrary code with the runner host's credentials and filesystem access. Use self-hosted runners only on **private repositories** or with trusted contributors.

---

## What We Need To Do
1. Retrieve dynamic Runner download commands and token directly from GitHub Settings.
2. Download, extract, and configure the Runner binary on the EC2 instance.
3. Install and manage the Runner as a persistent `systemd` daemon (`svc.sh`).
4. Write a GitHub Actions workflow with `runs-on: self-hosted`.
5. Trigger and verify workflow execution on the EC2 host.

---

## How To Do It (Step-by-Step Instructions)

### Step 1: Connect to AWS EC2 Instance
```bash
ssh -i "my-key.pem" ubuntu@<EC2-PUBLIC-IP>
```

### Step 2: Get Runner Setup Commands from GitHub
1. In your GitHub repository, navigate to **Settings** -> **Actions** -> **Runners**.
2. Click **New self-hosted runner**.
3. Select **Runner image**: `Linux` and **Architecture**: `x64`.
4. GitHub dynamically shows the exact download URL and runner token valid for your repository.

### Step 3: Install and Configure Runner on EC2
Execute the commands provided by your GitHub repository page:
```bash
# Create working directory
mkdir -p actions-runner && cd actions-runner

# Download the latest runner using GitHub's provided URL (example using GitHub releases API)
LATEST_VERSION=$(curl -s https://api.github.com/repos/actions/runner/releases/latest | grep '"tag_name":' | sed -E 's/.*"v([^"]+)".*/\1/')
curl -o actions-runner-linux-x64-${LATEST_VERSION}.tar.gz -L https://github.com/actions/runner/releases/download/v${LATEST_VERSION}/actions-runner-linux-x64-${LATEST_VERSION}.tar.gz

# Extract installer
tar xzf ./actions-runner-linux-x64-${LATEST_VERSION}.tar.gz

# Configure with your repository URL and Token from GitHub page
./config.sh --url https://github.com/<GITHUB_USERNAME>/<REPO_NAME> --token <RUNNER_REGISTRATION_TOKEN>
```
Press `Enter` to accept default runner name and labels.

### Step 4: Install and Start Runner as a Systemd Service
Do not run `./run.sh` interactively in a transient terminal. Install it as a persistent service:
```bash
sudo ./svc.sh install
sudo ./svc.sh start
sudo ./svc.sh status
```
*In GitHub under Settings -> Actions -> Runners, the runner status will now display a green **Idle** badge.*

### Step 5: Create Test Workflow File
In your repository, create `.github/workflows/self-hosted-test.yml`:
```yaml
name: Self-Hosted Runner Test

on:
  push:
    branches: [ main ]

jobs:
  run-on-ec2:
    runs-on: self-hosted
    steps:
      - name: Checkout Repository
        uses: actions/checkout@v4

      - name: Display Host Diagnostics
        run: |
          echo "Job running on Self-Hosted EC2 Runner"
          hostname
          whoami
          uptime
```

### Step 6: Verify Execution
1. Commit and push the workflow to the `main` branch.
2. In GitHub, go to the **Actions** tab.
3. Open the workflow run and expand the steps to verify execution on your EC2 host.
