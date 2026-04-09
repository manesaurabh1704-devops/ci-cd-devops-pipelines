📁 ci-cd-devops-pipelines 
🏗️ Overview

Production-grade Jenkins CI/CD pipeline for StudentSphere application (Multi-cloud DevOps project)

📁 Repository Structure
ci-cd-devops-pipelines/
├── jenkins/
│   └── Jenkinsfile
├── screenshots/
└── README.md
🔄 Pipeline Flow
GitHub Push
   ↓
Jenkins Trigger
   ↓
Stage 1: Git Checkout
Stage 2: Backend Maven Build
Stage 3: Frontend npm Build
Stage 4: Trivy Scan
Stage 5: Docker Build
Stage 6: Docker Push
Stage 7: Deploy to EKS
   ↓
EKS Updated Deployment
📋 Pipeline Stages
Stage	Tool	Description
Git Checkout	Git	Latest code fetch
Backend Build	Maven + Java 17	Spring Boot JAR build
Frontend Build	Node.js 20	React build
Security Scan	Trivy	Vulnerability scan
Docker Build	Docker	Image build
Docker Push	DockerHub	Push with tag
Deploy	kubectl	EKS rolling update
⚡ Jenkins Setup
Prerequisites
Jenkins 2.x
Java 17
Docker
kubectl
AWS CLI
Node.js 20
Maven
Trivy
Step 1 — Plugins

Install:

Docker Pipeline
Docker Commons
Kubernetes CLI
Git
Pipeline
GitHub Integration
Credentials Binding
Blue Ocean
Step 2 — Credentials

Add:

DockerHub (username + password)
AWS Secret Key
Step 3 — Java Config
export JAVA_HOME=/usr/lib/jvm/java-17-openjdk-amd64
export PATH=$JAVA_HOME/bin:$PATH
Step 4 — Access Setup

Docker Access

sudo usermod -aG docker jenkins

kubectl

sudo mkdir -p /var/lib/jenkins/.kube
sudo cp ~/.kube/config /var/lib/jenkins/.kube/config
sudo chown -R jenkins:jenkins /var/lib/jenkins/.kube

AWS

sudo mkdir -p /var/lib/jenkins/.aws
sudo cp ~/.aws/* /var/lib/jenkins/.aws/
sudo chown -R jenkins:jenkins /var/lib/jenkins/.aws

Restart

sudo systemctl restart jenkins
Step 5 — Pipeline Job
Name: studentsphere-pipeline
Type: Pipeline from SCM
Repo: main branch
Script: Jenkinsfile
Step 6 — Verify
kubectl get pods -n studentsphere
kubectl get deployments -n studentsphere

Check DockerHub images

🐛 Troubleshooting
Maven Error
JAVA version issue → set JAVA_HOME to 17
Jenkins Key Error
wget https://pkg.jenkins.io/debian-stable/jenkins.io-2026.key
Docker Permission Error
sudo usermod -aG docker jenkins
sudo systemctl restart jenkins
AWS Credentials Error
copy ~/.aws to /var/lib/jenkins/.aws
Plugins Not Showing
Update site → Refresh
🔗 Related Repos
multi-cloud-devops-studentsphere
kubernetes-production-setup
terraform-multi-cloud-infra
monitoring-observability-stack
devops-security-secrets
👨‍💻 Author

Saurabh Mane — DevOps Engineer
