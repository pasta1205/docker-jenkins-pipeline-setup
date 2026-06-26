# Docker + Jenkins Setup Guide
### Local Windows 11 — Complete End-to-End Setup

---

## Prerequisites Check

Open **PowerShell as Administrator** and run:

```powershell
# Check virtualization
systeminfo | findstr /i "virtualization"

# Check WSL2
wsl --status
```

---

## Phase 1 — Enable WSL2

```powershell
# Run as Administrator
wsl --install
wsl --set-default-version 2
```

> Restart your machine after this step.

---

## Phase 2 — Install Docker Desktop

1. Download from **https://www.docker.com/products/docker-desktop**
2. Click **Download for Windows – AMD64**
3. Run the installer → keep defaults → ensure **"Use WSL 2"** is checked
4. Restart when prompted
5. Verify Docker is working:

```powershell
docker --version
docker run hello-world
```

You should see `Hello from Docker!` ✅

---

## Phase 3 — Run Jenkins Container

```powershell
# Run Jenkins as root with Docker socket access
docker run -d `
  --name jenkins `
  -p 8080:8080 `
  -p 50000:50000 `
  -u root `
  -v jenkins_home:/var/jenkins_home `
  -v /var/run/docker.sock:/var/run/docker.sock `
  jenkins/jenkins:lts

# Install Docker CLI inside Jenkins
docker exec -u root jenkins bash -c "apt-get update && apt-get install -y docker.io"

# Get unlock password
docker exec jenkins cat /var/jenkins_home/secrets/initialAdminPassword
```

> **Note:** PowerShell uses backtick **`** for line continuation, not backslash like Linux.

---

## Phase 4 — Unlock & Setup Jenkins UI

1. Open **http://localhost:8080** in your browser
2. Paste the password from the command above
3. Click **Install suggested plugins** → wait for all plugins to install
4. Create your admin user → click **Save and Finish**
5. Click **Start using Jenkins**

---

## Phase 5 — Create Hello World Pipeline

1. Click **New Item** on the Jenkins dashboard
2. Enter name: `hello-world-pipeline`
3. Select **Pipeline** → click **OK**
4. Scroll down to the **Pipeline** section
5. Paste the script below
6. Click **Save** → **Build Now**

### Pipeline Script

```groovy
pipeline {
    agent any

    stages {
        stage('Hello World') {
            steps {
                script {
                    echo '===== Running Hello World Container ====='
                    sh 'docker run --name hello-world-container hello-world'
                    echo '===== Container Execution Complete ====='
                }
            }
        }

        stage('Fetch Logs') {
            steps {
                script {
                    echo '===== Fetching Container Logs Below ====='
                    sh 'docker logs hello-world-container'
                    echo '===== Log Fetch Complete ====='
                }
            }
        }
    }

    post {
        always {
            echo '===== Cleaning Up Container ====='
            sh 'docker rm -f hello-world-container || true'
            echo '===== Cleanup Done ====='
        }
    }
}
```

### Expected Console Output

```
===== Running Hello World Container =====
+ docker run --name hello-world-container hello-world
Hello from Docker!
This message shows that your installation appears to be working correctly.
===== Container Execution Complete =====

===== Fetching Container Logs Below =====
+ docker logs hello-world-container
Hello from Docker!
This message shows that your installation appears to be working correctly.
===== Log Fetch Complete =====

===== Cleaning Up Container =====
+ docker rm -f hello-world-container
hello-world-container
===== Cleanup Done =====

Finished: SUCCESS
```

---

## Useful Docker Commands

```powershell
# Check running containers
docker ps

# Check all containers (including stopped)
docker ps -a

# Stop Jenkins
docker stop jenkins

# Start Jenkins again
docker start jenkins

# View Jenkins logs
docker logs jenkins

# View any container logs
docker logs <container-name>

# Remove a container
docker rm -f <container-name>

# Restart Jenkins container
docker restart jenkins
```

---

## Troubleshooting

| Error | Fix |
|---|---|
| Permission denied on docker.sock | Use `-u root` in docker run command |
| Port 8080 already in use | `docker rm -f jenkins` and rerun |
| Jenkins password not found | Wait 30 seconds after container starts, then retry |
| Docker CLI not found in Jenkins | Re-run the `apt-get install docker.io` command |
| WSL2 not installed | Run `wsl --install` as Administrator and restart |

---

## Pipeline Flow

```
Jenkins (localhost:8080)
        │
        ├── Stage 1: docker run hello-world   → Prints Hello World
        ├── Stage 2: docker logs              → Fetches & replays logs
        └── Post:    docker rm -f             → Cleans up container
```

---

*Guide prepared for local Windows 11 setup with Docker Desktop (WSL2 backend)*
