# CI/CD Pipeline with Jenkins, SonarQube, Docker, and GitHub

This repository demonstrates how to set up a robust CI/CD pipeline using Jenkins, SonarQube, Docker, and GitHub. The pipeline automates the process of pulling code, performing a code quality scan, building a Docker image, and deploying a web application.

---

## Overview

The pipeline consists of the following steps:

1. **Code Push**: Code is pushed to the GitHub repository.
2. **Build Trigger**: A GitHub webhook triggers a Jenkins job.
3. **Code Quality Analysis**: SonarQube scans the code and provides quality feedback.
4. **Build Docker Image**: Jenkins builds a Docker image for the application.
5. **Deploy Application**: The application is deployed as a Docker container.

---

## Prerequisites

1. **Jenkins**: Installed and configured with the necessary plugins.
2. **SonarQube**: Installed on a server.
   - **Tip**: Use a t2.medium or t3.medium AWS instance for SonarQube. It may not work properly on t2.micro or t3.micro instances.
3. **Docker**: Installed on the host machine.
4. **GitHub Repository**: Contains the application code.
5. **GitHub Webhook**: Configured to trigger Jenkins builds on new commits.

---

## Pipeline Configuration

### Step 1: GitHub Webhook Setup
1. Navigate to **Settings > Webhooks** in your GitHub repository.
2. Add the Jenkins URL followed by `/github-webhook/` (e.g., `http://your-jenkins-server/github-webhook/`).
3. Configure the webhook to trigger on `push` events.

### Step 2: Jenkins Job Configuration
1. Create a new Freestyle Project in Jenkins.
2. Set up **Source Code Management** to pull the repository from GitHub.
3. Add the following build steps:

#### Code Quality Analysis with SonarQube
```bash
sonar-scanner \
  -Dsonar.projectKey=your_project_key \
  -Dsonar.sources=. \
  -Dsonar.host.url=http://your-sonarqube-server:9000 \
  -Dsonar.login=your_authentication_token
```

#### Docker Build and Deploy
```bash
#!/bin/bash

# Copy files from Jenkins workspace
if ! cp -r /var/lib/jenkins/workspace/demo/* /home/ubuntu/web; then
    echo "Error: Failed to copy files."
    exit 1
fi

# Navigate to the project directory
cd /home/ubuntu/web || { echo "Directory not found!"; exit 1; }

# Remove existing containers
if [ "$(docker ps -qa -f name=website)" ]; then
    docker rm -v -f $(docker ps -qa -f name=website)
else
    echo "No containers named 'website' to remove."
fi

# Build and run the Docker container
docker build -t web .
docker run -d -p 8081:80 --name website web
```

---

## Diagram

![CI/CD Pipeline Diagram](./pipeline_diagram.png)

---

## Tools Used

- **GitHub**: Source code repository and webhook integration.
- **Jenkins**: Orchestrates the CI/CD process.
- **SonarQube**: Performs static code analysis.
- **Docker**: Containerizes and deploys the application.

---

## Best Practices

1. **Secure Credentials**: Use Jenkins credentials or environment variables for sensitive data like SonarQube tokens.
2. **Optimize Docker Images**: Use multi-stage builds to keep images lightweight.
3. **Monitor Resources**: Ensure adequate resources for Jenkins and SonarQube servers.
4. **Clean Up**: Periodically prune unused Docker images and containers to save disk space.

---

## License

This project is licensed under the MIT License. See the LICENSE file for details.

---

## Feedback

Feel free to open an issue or submit a pull request if you have any suggestions or improvements.
