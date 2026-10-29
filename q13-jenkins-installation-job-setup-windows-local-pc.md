# Practical 13: Jenkins Installation and Job Setup (Windows Local PC)

## Objective
Install Jenkins on a local Windows PC, verify Java LTS compatibility, configure the Windows service logon account and Git CLI availability, and set up a verified build job.

## Prerequisites
- Supported Java version: **OpenJDK 17 or 21 LTS** (Jenkins does not support Java 8, and Java 11 is deprecated in modern Jenkins versions).
- Git for Windows installed and added to system `PATH` (`git --version`).
- Windows administrator privileges.

---

> [!NOTE]
> **Windows Service Account Caveat**:
> By default, the Jenkins Windows installer sets up the service under the `LocalSystem` account or prompts for a local Windows user. If your Jenkins job needs to execute Git commands, Docker for Windows, or access network shares, ensure the service account has permission to execute those binaries or run as your standard administrative user.

---

## What We Need To Do
1. Verify Java 17/21 and Git for Windows from PowerShell.
2. Install Jenkins via Windows MSI installer and configure the HTTP port.
3. Unlock Jenkins using the generated admin secret.
4. Install recommended plugins and create the primary administrator user.
5. Configure system tools (Git path) and create a Freestyle test job.
6. Verify job execution and examine console logs.

---

## How To Do It (Step-by-Step Instructions)

### Step 1: Verify Prerequisites in PowerShell
Open PowerShell as Administrator:
```powershell
# Verify Java 17 or 21 is present
java -version

# Verify Git is installed and in PATH
git --version
```
*(If Java is missing, download Eclipse Temurin JDK 17/21 from [adoptium.net](https://adoptium.net/) and ensure `JAVA_HOME` and `PATH` are populated).*

### Step 2: Install Jenkins
1. Download Jenkins Windows installer from [jenkins.io/download](https://www.jenkins.io/download/).
2. Run `jenkins.msi`.
3. Choose installation destination (e.g., `C:\Program Files\Jenkins`).
4. **Service Logon Credentials**: Choose `Run service as LocalSystem` (or provide your local Windows credentials).
5. **Port Selection**: Default is `8080`. Click **Test Port** to ensure no conflicts.
6. **Select Java Home**: Browse and select the JDK 17/21 directory (e.g. `C:\Program Files\Eclipse Adoptium\jdk-17...`).
7. Complete the wizard.

### Step 3: Unlock Jenkins
1. Open your browser and navigate to:
```text
http://localhost:8080
```
2. Retrieve the initial admin password from PowerShell:
```powershell
Get-Content "C:\Program Files\Jenkins\secrets\initialAdminPassword"
```
3. Copy and paste the key into the unlock prompt and click **Continue**.

### Step 4: Install Suggested Plugins & Create Admin
1. Click **Install suggested plugins**. Wait for Git, Pipeline, and Workspace plugins to finish installing.
2. Fill in the **Create First Admin User** fields.
3. Confirm the Jenkins URL (`http://localhost:8080/`) and finish setup.

### Step 5: Configure Tool Paths & Create Freestyle Job
1. Check Git configuration:
   - Go to **Manage Jenkins** -> **Tools**.
   - Under **Git installations**, ensure `Path to Git executable` is set to `git.exe` or `C:\Program Files\Git\bin\git.exe`.
2. Create Job:
   - Click **New Item** -> Name: `local-windows-test-job`.
   - Select **Freestyle project** and click **OK**.
   - Under **Build Steps**, click **Add build step** -> **Execute Windows batch command**.
   - Add:
     ```bat
     echo "=== Jenkins Local Execution Verification ==="
     git --version
     hostname
     date /t
     time /t
     echo "Build successful!"
     ```
3. Click **Save**.

### Step 6: Execute and Verify
1. Click **Build Now** on the left menu.
2. Click on build `#1` -> **Console Output**.
3. Verify that the output shows `SUCCESS`.
