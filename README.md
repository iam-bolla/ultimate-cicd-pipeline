# CI/CD with GitOps – Full Hands-On Implementation 🚀

Welcome! This repository demonstrates a **complete CI/CD pipeline** with a GitOps approach, using Jenkins, Docker, SonarQube, and Kubernetes (ArgoCD). It’s designed to show **real-world DevOps automation** from code commit to deployment.  

Clone or fork the repo: [ultimate-cicd-pipeline](https://github.com/iam-bolla/ultimate-cicd-pipeline).

---

## Table of Contents

- [Overview](#overview)  
- [Prerequisites](#prerequisites)  
- [Step 1: Launch AWS EC2 Instance](#step-1-launch-aws-ec2-instance-☁️)  
- [Step 2: Jenkins Installation & Setup](#step-2-jenkins-installation--setup-🛠️)  
- [Step 3: Docker as Jenkins Agent](#step-3-docker-as-jenkins-agent-🐳)  
- [Step 4: SonarQube Setup](#step-4-sonarqube-setup-✅)  
- [Step 5: Configure Docker on EC2](#step-5-configure-docker-on-ec2-🐋)  
- [Step 6: Add Credentials in Jenkins](#step-6-add-credentials-in-jenkins-🔑)  
- [Step 7: Jenkins Pipeline Configuration](#step-7-jenkins-pipeline-configuration-📝)  
- [Step 8: GitOps CD with ArgoCD on Minikube](#step-8-gitops-cd-with-argocd-on-minikube-🌱)  
- [Results](#results-🎯)  
- [Skills Demonstrated](#skills-demonstrated-💡)  
- [Contributing](#contributing-🤝)  
- [License](#license-📄)  

---

## Overview

This project implements a **CI/CD pipeline with GitOps**:  

- **CI (Continuous Integration):**  
  - Jenkins automated builds  
  - Docker containerized agents  
  - SonarQube static code analysis  
  - Docker image build & push  

- **CD (Continuous Deployment / GitOps):**  
  - Minikube local Kubernetes cluster  
  - ArgoCD operator deployment  
  - Automated sync from Git repository → Kubernetes cluster  

---

## Prerequisites

- AWS account with EC2 access  
- Ubuntu 22.04 instance (t2.medium recommended)  
- GitHub account for repository management  
- DockerHub account for image repository  
- kubectl & minikube installed locally  

---

## Step 1: Launch AWS EC2 Instance ☁️

1. Go to AWS Console → EC2 → Instances → Launch instances  
2. Select **Ubuntu 22.04**  
3. Choose **t2.medium**  

💡 *Pro Tip:* t2.medium ensures smooth installation of Jenkins, Docker, SonarQube, and Kubernetes components.  


---

## Step 2: Jenkins Installation & Setup 🛠️

### Install Java
```bash
sudo apt update
sudo apt install openjdk-17-jre
java -version
```
Install Jenkins
```bash
curl -fsSL https://pkg.jenkins.io/debian/jenkins.io-2023.key | sudo tee /usr/share/keyrings/jenkins-keyring.asc > /dev/null
echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] https://pkg.jenkins.io/debian binary/ | sudo tee /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt-get update
sudo apt-get install jenkins
```
By default, Jenkins will not be accessible to the external world due to the inbound traffic restriction by AWS. Open port 8080 in the inbound traffic rules i allowed all traffic beacuse port 9000 should be also allowed 
as this is practice so iam allowing,in your case just allow 8080 and 9000.

EC2 > Instances > Click on
In the bottom tabs -> Click on Security
Security groups
Add inbound traffic rules (you can just allow TCP 8080 as well, in my case, I allowed All traffic).
Access Jenkins:
``` bash
URL: http://<EC2-PUBLIC-IP>:8080
```

Get initial admin password:
```bash
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```

<img width="987" height="453" alt="image" src="https://github.com/user-attachments/assets/7093130b-6d26-4530-b2c4-6ce5442c2c14" />

Install suggested plugins.

Wait for the Jenkins to Install suggested plugins.

Create First Admin User or Skip the step [If you want to use this Jenkins instance for future use-cases as well, better to create admin user
Jenkins Installation is Successful. You can now starting using the Jenkins.

<img width="1980" height="1232" alt="image" src="https://github.com/user-attachments/assets/37ce9bba-63b0-4ec3-84be-3d7876b6b582" />

---
Step 3: Docker as Jenkins Agent 

Why Docker?

Ephemeral containers free resources automatically

Parallel builds without conflicts

Pre-configured Maven eliminates manual setup

Plugins: Docker Pipeline Plugin, SonarQube Scanner Plugin
 
<img width="1027" height="367" alt="image" src="https://github.com/user-attachments/assets/4e8d9e2a-9857-431d-881d-500f9c74d230" />
 Pro Tip: Use Docker-outside-of-Docker (DooD) for speed, simplicity, and easier debugging

---
Step 4: SonarQube Setup ✅
```bash
adduser sonarqube
sudo apt install unzip
sudo su - sonarqube
wget https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-10.4.1.88267.zip
unzip *
chown -R sonarqube:sonarqube /opt/sonarqube
chmod -R 775 /opt/sonarqube
cd /opt/sonarqube/bin/linux-x86-64
./sonar.sh start
```

Access: http://<EC2-IP>:9000
Credentials: admin/admin

---
Step 5: Docker Setup on EC2
```bash
sudo apt install docker.io
sudo usermod -aG docker jenkins
sudo usermod -aG docker ubuntu
```
Quick Tip: We use Docker-outside-of-Docker (DooD) – faster, simpler, and avoids headaches of DinD.

---
Step 6: Add Credentials in Jenkins

We need tokens for:

SonarQube → ID: sonarqube

GitHub → ID: github

DockerHub → ID: docker-cred

Make sure the IDs match what’s in your Jenkinsfile.

---
Step 7: Pipeline Configuration

Modify your Jenkinsfile:
```bash

stage('Static Code Analysis') {
    environment { SONAR_URL = "http://<your-sonarqube-ip>:9000" }
}
stage('Update Deployment File') {
    environment { 
        GIT_REPO_NAME = "<your-repo-name>"
        GIT_USER_NAME = "<your-github-username>" 
    }
}
```

Restart Jenkins → New pipeline → SCM → Provide repo, branch, Jenkinsfile → Build.

✅ Watch as Docker builds images, SonarQube checks quality, and deployment files update automatically.
<img width="1865" height="913" alt="image" src="https://github.com/user-attachments/assets/f9af6b3d-aa59-48d1-bec0-9afb4e5b9321" />

<img width="1857" height="852" alt="image" src="https://github.com/user-attachments/assets/44a728d8-d10c-4fd3-8b65-e88aad75fee3" />

<img width="359" height="41" alt="image" src="https://github.com/user-attachments/assets/a762ea3e-1e85-4001-9b07-a90ce50156c4" />

<img width="1876" height="887" alt="image" src="https://github.com/user-attachments/assets/e10492fd-3e6d-45f2-8e85-14f30e02924a" />

---

Step 8: GitOps CD with ArgoCD on Minikube

Start Minikube
```bash
minikube start --driver=docker
```

Install Operator Lifecycle Manager (OLM)
```bash

curl -sL https://github.com/operator-framework/operator-lifecycle-manager/releases/download/v0.33.0/install.sh | bash -s v0.33.0
```

Install ArgoCD Operator
```bash
kubectl create -f https://operatorhub.io/install/argocd-operator.yaml
```

Check operator status: ```bash kubectl get svc -n operators ```

Create ArgoCD Controller YAML in repo (in repo there is argocd-basic.yaml) and apply it:
```bash
kubectl apply -f argocd.yaml
```

Controller is created as NodePort (as specified in YAML)

Access URL:
```bash

minikube service example-argocd-server
```

Username: admin

Password:
```bash
kubectl get secret argocd-cluster -o jsonpath="{.data.adminpassword}" | base64 --decode
```
<img width="1628" height="846" alt="image" src="https://github.com/user-attachments/assets/d59d424d-9eca-4b46-b4bf-85fc0ce2746d" />

Create an ArgoCD Application:

Provide repo details, path, cluster URL (local), and namespace

Click Sync


<img width="1628" height="846" alt="image" src="https://github.com/user-attachments/assets/b10cfdef-0457-43b4-b833-7e772f7f13eb" />

<img width="1628" height="846" alt="image" src="https://github.com/user-attachments/assets/12f2193b-9622-454e-b628-0ee42bc3b35d" />
Verify in cluster:
```bash
kubectl get deploy
kubectl get pods
```
You’ll see your deployment, replicas, and pods running successfully.
<img width="907" height="110" alt="image" src="https://github.com/user-attachments/assets/69c79cb2-8b33-4575-bc94-30a32105d0fa" />

✅ Congratulations!
Your CI/CD pipeline with GitOps approach is now fully functional—code builds, quality checks run, Docker images are pushed, and deployments are synced to Kubernetes via ArgoCD.

Happy CI/CD-fying! :)






