# Jenkins Basic CI/CD - Portfolio Site

## Project Overview

This project demonstrates a **basic CI/CD pipeline** using **Jenkins** and **Docker**. The application is a simple Flask-based portfolio website that is automatically built and deployed through Jenkins whenever code changes are pushed to GitHub.

## Project Structure

```
jenkins-basic-ci/
├── portfolio-site/
│   ├── app.py                 # Flask application
│   ├── requirements.txt        # Python dependencies
│   ├── Dockerfile             # Docker configuration
│   ├── templates/
│   │   └── index.html         # Portfolio website HTML
│   └── static/
│       └── style.css          # Styling
├── Jenkinsfile                # Jenkins pipeline configuration
├── README.md                  # This file
└── .gitignore                 # Git ignore rules
```

## What This Project Does

1. **Source Code Management**: The portfolio-site code is stored in a GitHub repository
2. **Continuous Integration**: Jenkins polls the GitHub repository for changes
3. **Automated Build**: When changes are detected, Jenkins automatically:
   - Clones the latest code
   - Builds a Docker image for the portfolio-site
   - Runs the Docker container on port 5002
4. **Continuous Deployment**: The application is immediately available at `http://localhost:5002`

## Prerequisites

- **Docker**: Installed and running on your machine
- **Docker Socket**: Available at `/var/run/docker.sock` (for Jenkins to access Docker)
- **Git**: For version control and cloning the repository
- **GitHub Account**: To host the repository
- **PowerShell**: For running Docker commands (Windows)

## Setup Instructions

### Step 1: Install and Run Jenkins with Docker

Open **PowerShell as Administrator** and run:

```powershell
docker run -d -p 8080:8080 -p 50000:50000 --name jenkins `
  -v jenkins_home:/var/jenkins_home `
  -v /var/run/docker.sock:/var/run/docker.sock `
  jenkins/jenkins:lts
```

### Step 2: Unlock Jenkins

Wait 2-3 minutes for Jenkins to start, then retrieve the initial admin password:

```powershell
docker exec -it jenkins cat /var/jenkins_home/secrets/initialAdminPassword
```

Copy the password and go to `http://localhost:8080` to unlock Jenkins.

### Step 3: Complete Jenkins Setup

- Install suggested plugins
- Create a new admin user (remember username and password)

### Step 4: Create a Jenkins Freestyle Job

1. Click **New Item**
2. Name it: `portfolio-site-build`
3. Select **Freestyle project** → Click **OK**

### Step 5: Configure Source Code Management

1. Under **Source Code Management**, select **Git**
2. Enter your GitHub repository URL:
   ```
   https://github.com/MRG-Hazmatz/jenkins-basic-ci.git
   ```
3. Set Branch to: `*/main`
4. Leave credentials blank (for public repos)
5. Click **Save**

### Step 6: Add Build Trigger

1. Go back to job configuration
2. Under **Build Triggers**, check **Poll SCM**
3. Enter schedule: `H/5 * * * *` (polls every 5 minutes)
4. Click **Save**

### Step 7: Add Build Steps

1. Under **Build**, click **Add build step** → Select **Execute shell**
2. Paste these commands:

```bash
cd portfolio-site
docker build -t portfolio-site .
docker run -d -p 5002:5000 portfolio-site
```

3. Click **Save**

### Step 8: Test the Build

1. Click **Build Now**
2. Check the build logs for success
3. Visit `http://localhost:5002` to see your portfolio site

## How the CI/CD Pipeline Works

```
Developer pushes code to GitHub
           ↓
Jenkins detects changes (via Poll SCM)
           ↓
Jenkins clones the repository
           ↓
Dockerfile builds the Docker image
           ↓
Docker container runs on port 5002
           ↓
Application is live at http://localhost:5002
```

## Dockerfile Explanation

```dockerfile
FROM python:3.11-slim          # Base image: Python 3.11
WORKDIR /app                   # Set working directory
COPY requirements.txt .        # Copy dependency file
RUN pip install -r requirements.txt  # Install dependencies
COPY app.py .                  # Copy Flask app
EXPOSE 5000                    # Expose port 5000
CMD ["python", "app.py"]       # Run the app on startup
```

## Flask App Structure

The `app.py` file is a simple Flask application that:
- Serves the portfolio website
- Listens on port 5000 inside the container
- Maps to port 5002 on the host machine

## Deployment Flow

1. **Manual trigger**: Click "Build Now" in Jenkins
2. **Automated trigger**: Jenkins automatically detects code changes every 5 minutes (Poll SCM)
3. **Build execution**: Docker builds and runs the container
4. **Access**: Visit `http://localhost:5002` to access the live application

---

## Screenshots & Working Demonstrations

Below are the Jenkins CI/CD execution screenshots for this project:

### ✅ Jenkins Dashboard
![Jenkins Dashboard](./screenshots/01-jenkins-dashboard.jpg)

### 🔧 Source Code Management (Git Config)
![SCM Config](./screenshots/04-jenkins-scm.jpg)

### ⏱️ Build Triggers (Poll SCM)
![Build Triggers](./screenshots/02-jenkins-triggers.jpg)

### 🧱 Build Steps (Execute Shell)
![Build Steps](./screenshots/03-jenkins-build-steps.jpg)

### 🚀 Portfolio App Running in Browser
![Portfolio Running](./screenshots/05-portfolio-running.jpg)
### 1. Jenkins Dashboard
![Jenkins Dashboard - Shows the home page with job pipeline](./screenshots/01-jenkins-dashboard.jpg)
*Jenkins main dashboard with portfolio-site-build job configured*

### 2. Jenkins Job Configuration
![Jenkins Job Configuration - Shows SCM and build trigger settings](./screenshots/02-jenkins-job-config.jpg)
*Jenkins Freestyle job configuration with Git SCM and Poll SCM trigger*

### 3. Successful Build
![Successful Build - Console output showing successful build steps](./screenshots/03-successful-build.jpg)
*Build console output showing successful Docker build and container deployment*

### 4. Build Console Output
![Build Console - Detailed build execution logs](./screenshots/04-build-console.jpg)
*Detailed console logs showing Git clone, Docker build, and container run commands*

### 5. Running Containers
![Docker Containers - Shows running Jenkins and portfolio-site containers](./screenshots/05-docker-containers.jpg)
*PowerShell output of `docker ps` showing both Jenkins and portfolio-site running*

### 6. Application Running
![Portfolio Site - Application accessible on localhost:5002](./screenshots/06-portfolio-site-running.jpg)
*Portfolio website running successfully at http://localhost:5002*

### 7. Build Triggered by Poll SCM
![Auto Build Triggered - Jenkins automatically triggered build on code change](./screenshots/07-auto-build-triggered.jpg)
*Jenkins build automatically triggered by Poll SCM after code push*

---

## Troubleshooting

### Jenkins container won't start
- Ensure Docker is running
- Check if port 8080 is available
- Run `docker logs jenkins` to see error messages

### Build fails with "docker: not found"
- Ensure Jenkins container has Docker socket mounted (`-v /var/run/docker.sock:/var/run/docker.sock`)
- Restart Jenkins container

### App not accessible on localhost:5002
- Check if container is running: `docker ps`
- Check logs: `docker logs <container_id>`
- Ensure Flask app is listening on port 5000 inside container

### Port already in use
- List containers: `docker ps -a`
- Stop conflicting container: `docker stop <container_id>`
- Or use a different port mapping: `-p 5003:5000`

---

## Key Concepts Demonstrated

| Concept | Implementation |
|---------|-----------------|
| **Version Control** | Git + GitHub |
| **Continuous Integration** | Jenkins polling SCM |
| **Automated Build** | Docker image creation |
| **Continuous Deployment** | Docker container orchestration |
| **Infrastructure as Code** | Dockerfile for reproducible builds |
| **CI/CD Pipeline** | Automated from code push to deployment |

---

## Next Steps

- Add more features to the Flask app
- Configure GitHub webhooks for instant builds (instead of polling)
- Add post-build actions (notifications, artifact archiving)
- Set up multiple environments (dev, staging, production)
- Implement automated testing in the pipeline
- Add health checks for the deployed application

---

## Author

Created as part of DevOps/CI-CD training for Jenkins and Docker integration.

## License

This project is provided as-is for educational purposes.
