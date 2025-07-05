#  Jenkins CI/CD Pipeline for Node.js Express App

![jenkins](https://github.com/user-attachments/assets/88009d4a-b3f6-4a94-928a-87aa06ad7530)


This project demonstrates setting up a full **CI/CD pipeline** for a Node.js Express application using:
- Jenkins
- Docker & Docker Hub
- GitHub
- Kubernetes

The pipeline automatically builds, tests, packages into Docker, pushes the image to Docker Hub, and updates the Kubernetes deployment manifest.

---

##  How the Pipeline Works

1. Developer pushes code to GitHub
2. Jenkins (running on EC2) detects change via webhook
3. Jenkins:
   - Uses custom agent image: `<dockerhub_username>/node-docker-agent:latest`
   - Installs Node, Docker CLI, Git
   - Builds the Express app
   - Builds Docker image and tags it with build number (e.g., `:7`)
   - Pushes image to Docker Hub (`<dockerhub_username>/jenkins-cicd`)
   - Updates `k8s/deployment.yml` with new tag
   - Commits updated manifest back to GitHub
4. Kubernetes pulls the new image and deploys automatically

---


## Steps to Reproduce
1️. Launch Jenkins on EC2
Create EC2 instance (t2.large recommended)

Open port 8080 for Jenkins UI

2️. Install Jenkins & Plugins

Install Jenkins

Add plugins: Docker Pipeline.

3️. Build and Push Custom Agent
```
docker build -t <dockerhub_username>/node-docker-agent:latest .
docker push <dockerhub_username>/node-docker-agent:latest
```
4️. Setup Jenkins Pipeline
Point to your GitHub repo containing Jenkinsfile

Use credentials:

Docker Hub 

GitHub token 

## Jenkinsfile Highlights
Uses Docker agent:

```
agent {
  docker {
    image '<dockerhub_username>/node-docker-agent:latest'
    args '-v /var/run/docker.sock:/var/run/docker.sock'
  }
}
```

Builds Docker image:

```
docker build -t <dockerhub_username>/jenkins-cicd:${BUILD_NUMBER} .
```
Pushes image:

```
docker.withRegistry('https://index.docker.io/v1/', 'docker-cred') {
    dockerImage.push()
}
```

Updates k8s/deployment.yml and commits back to GitHub.

## Screenshots

![Screenshot 2025-07-05 170147](https://github.com/user-attachments/assets/80d7321a-4d43-4379-9983-d8237d98fdf0)


![Screenshot 2025-07-05 170052](https://github.com/user-attachments/assets/a5d3eadd-7bc0-4ab9-9375-619dc1570360)
