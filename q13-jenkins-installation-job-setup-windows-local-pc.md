# Practical 13: Jenkins Installation and Job Setup (Windows Local PC)

## Objective
Install Jenkins on a local Windows PC, complete initial administrator setup, and configure and execute a build job.

## Prerequisites
- Java Development Kit (JDK 17 or JDK 21) installed.
- Administrator rights on Windows.

## What We Need To Do
1. Verify and set up Java JDK environment.
2. Download and run Jenkins Windows Installer (.msi).
3. Unlock Jenkins using initial admin password.
4. Install suggested plugins and create first admin account.
5. Create and execute a Freestyle build job.

## How To Do It (Step-by-Step Instructions)

### Step 1: Verify Java Installation
Open PowerShell and check Java version:
```powershell
java -version
```
*(If missing, download Eclipse Temurin JDK 17/21 from adoptium.net and ensure it is added to system PATH).*

### Step 2: Download and Install Jenkins
1. Download Jenkins Windows installer from [jenkins.io/download](https://www.jenkins.io/download/).
2. Run `jenkins.msi`.
3. Choose installation path (e.g., `C:\Program Files\Jenkins`).
4. Select service logon type (`Run service as LocalSystem` or local domain user).
5. Specify port (default `8080`, click **Test Port** to verify availability).
6. Select your JDK installation path and complete the installation wizard.

### Step 3: Unlock Jenkins
1. Open your browser and navigate to:
```text
http://localhost:8080
```
2. Retrieve the initial admin password from PowerShell:
```powershell
Get-Content "C:\Program Files\Jenkins\secrets\initialAdminPassword"
```
3. Copy and paste the key into the Jenkins setup wizard and click **Continue**.

### Step 4: Install Plugins and Create Admin User
1. Select **Install suggested plugins** and wait for completion.
2. Fill out the **Create First Admin User** form (Username, Password, Full Name, Email).
3. Confirm Instance Configuration (`http://localhost:8080/`) and click **Save and Finish**.

### Step 5: Create and Run a Build Job
1. On the Jenkins dashboard, click **New Item**.
2. Enter item name: `my-first-windows-job`.
3. Select **Freestyle project** and click **OK**.
4. Scroll to **Build Steps**, click **Add build step** -> **Execute Windows batch command**.
5. Add the following command:
```bat
echo "Jenkins build running successfully on Windows!"
hostname
date /t
time /t
```
6. Click **Save**.
7. Click **Build Now** on the left menu.
8. Click on build `#1` -> **Console Output** to verify the successful run.

