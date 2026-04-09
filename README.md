# 🏗️ CI/CD DevOps Pipelines

> Production-grade Jenkins CI/CD pipeline for StudentSphere application.
> Part of the [multi-cloud-devops-studentsphere](https://github.com/manesaurabh1704-devops/multi-cloud-devops-studentsphere) project.

---

## 📁 Repository Structure

```
ci-cd-devops-pipelines/
├── jenkins/
│   └── Jenkinsfile          # Production Jenkins pipeline
├── screenshots/             # Proof of working pipeline
└── README.md
```

---

## 🔄 Pipeline Architecture

```
GitHub Push
    ↓
Jenkins Pipeline (Triggered)
    ↓
┌─────────────────────────────────────────┐
│  Stage 1: Git Checkout                  │
│  Stage 2: Backend Maven Build           │
│  Stage 3: Frontend npm Build            │
│  Stage 4: Trivy Security Scan           │
│  Stage 5: Docker Build                  │
│  Stage 6: Docker Push to DockerHub      │
│  Stage 7: Deploy to AWS EKS             │
└─────────────────────────────────────────┘
    ↓
AWS EKS — Updated Deployment
```

---

## 📋 Pipeline Stages

| Stage | Tool | Description |
|---|---|---|
| Git Checkout | Git | Pull latest code from GitHub |
| Backend Maven Build | Maven + Java 17 | Build Spring Boot JAR |
| Frontend npm Build | Node.js 20 | Build React production bundle |
| Trivy Security Scan | Trivy | Scan for HIGH/CRITICAL vulnerabilities |
| Docker Build | Docker | Build multi-stage images |
| Docker Push | DockerHub | Push images with build number tag |
| Deploy to EKS | kubectl | Rolling update on AWS EKS |

---

## ⚡ Complete Jenkins Setup Guide

> All commands tested on Ubuntu 22.04 / 24.04

---

### Step 1 — Launch EC2 Instance

```
AMI:           Ubuntu 22.04 LTS
Instance Type: t2.large or higher (Jenkins needs minimum 2GB RAM)
Storage:       30 GB
Security Group:
  - SSH     Port 22   — Your IP
  - Jenkins Port 8080 — 0.0.0.0/0
  - HTTP    Port 80   — 0.0.0.0/0
```

---

### Step 2 — System Update

```bash
sudo apt update && sudo apt upgrade -y
```

Expected output:
```
Reading package lists... Done
Building dependency tree... Done
0 upgraded, 0 newly installed, 0 to remove and 0 not upgraded.
```

---

### Step 3 — Java 17 Install

```bash
sudo apt install -y fontconfig openjdk-17-jdk
```

```bash
# Verify
java -version
javac -version
```

Expected output:
```
openjdk version "17.0.18" 2026-01-20
OpenJDK Runtime Environment (build 17.0.18+8-Ubuntu-124.04.1)
OpenJDK 64-Bit Server VM (build 17.0.18+8-Ubuntu-124.04.1, mixed mode, sharing)
javac 17.0.18
```

```bash
# Set JAVA_HOME permanently
sudo bash -c 'cat > /etc/profile.d/java.sh << EOF
export JAVA_HOME=/usr/lib/jvm/java-17-openjdk-amd64
export PATH=\$JAVA_HOME/bin:\$PATH
EOF'

source /etc/profile.d/java.sh

# Verify JAVA_HOME
echo $JAVA_HOME
```

Expected output:
```
/usr/lib/jvm/java-17-openjdk-amd64
```

---

### Step 4 — Jenkins Install

```bash
# Add Jenkins GPG key
sudo wget -O /etc/apt/keyrings/jenkins-keyring.asc \
  https://pkg.jenkins.io/debian-stable/jenkins.io-2026.key
```

Expected output:
```
jenkins-keyring.asc saved [3175/3175]
```

```bash
# Add Jenkins repository
echo "deb [signed-by=/etc/apt/keyrings/jenkins-keyring.asc] \
  https://pkg.jenkins.io/debian-stable binary/" | \
  sudo tee /etc/apt/sources.list.d/jenkins.list > /dev/null

# Install Jenkins
sudo apt update
sudo apt install -y jenkins
```

Expected output:
```
Setting up jenkins (2.x.x) ...
```

```bash
# Start and enable Jenkins
sudo systemctl start jenkins
sudo systemctl enable jenkins

# Verify Jenkins running
sudo systemctl status jenkins --no-pager
```

Expected output:
```
● jenkins.service - Jenkins Continuous Integration Server
     Active: active (running) since Thu 2026-04-09 20:02:28 UTC
   Main PID: 174179 (java)
```

```bash
# Get initial admin password
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```

Expected output:
```
a1b2c3d4e5f6g7h8i9j0k1l2m3n4o5p6
```

#### Jenkins Web Setup
```
1. Open browser: http://<EC2-PUBLIC-IP>:8080
2. Paste initial admin password
3. Click "Install suggested plugins" — wait 2-3 minutes
4. Create admin user:
   Username: admin
   Password: your-password
   Full name: Your Name
   Email:     your@email.com
5. Click "Save and Finish" → "Start using Jenkins"
```

---

### Step 5 — Docker Install

```bash
sudo apt install -y docker.io
sudo systemctl start docker
sudo systemctl enable docker
```

```bash
# Verify
docker --version
```

Expected output:
```
Docker version 29.1.3, build 29.1.3-0ubuntu3~24.04.1
```

```bash
# Give Jenkins and current user Docker access
sudo usermod -aG docker jenkins
sudo usermod -aG docker $USER
newgrp docker

# Verify Docker works without sudo
docker ps
```

Expected output:
```
CONTAINER ID   IMAGE   COMMAND   CREATED   STATUS   PORTS   NAMES
```

```bash
# Restart Jenkins to apply Docker group
sudo systemctl restart jenkins
```

---

### Step 6 — AWS CLI Install

```bash
sudo snap install aws-cli --classic
```

```bash
# Verify
aws --version
```

Expected output:
```
aws-cli/2.34.26 Python/3.14.3 Linux/6.17.0-1007-aws exe/x86_64.ubuntu.24
```

```bash
# Configure AWS CLI
aws configure
```

Enter when prompted:
```
AWS Access Key ID:     AKIAIOSFODNN7EXAMPLE
AWS Secret Access Key: wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY
Default region name:   ap-south-1
Default output format: json
```

```bash
# Verify AWS credentials
aws sts get-caller-identity
```

Expected output:
```json
{
    "UserId": "AIDAIOSFODNN7EXAMPLE",
    "Account": "207457247776",
    "Arn": "arn:aws:iam::207457247776:user/your-user"
}
```

```bash
# Give Jenkins user AWS credentials
sudo mkdir -p /var/lib/jenkins/.aws
sudo cp ~/.aws/credentials /var/lib/jenkins/.aws/credentials
sudo cp ~/.aws/config /var/lib/jenkins/.aws/config
sudo chown -R jenkins:jenkins /var/lib/jenkins/.aws
sudo chmod 600 /var/lib/jenkins/.aws/credentials

# Verify Jenkins can access AWS
sudo -u jenkins aws sts get-caller-identity
```

Expected output:
```json
{
    "UserId": "AIDAIOSFODNN7EXAMPLE",
    "Account": "207457247776",
    "Arn": "arn:aws:iam::207457247776:user/your-user"
}
```

---

### Step 7 — kubectl Install

```bash
# Download kubectl
curl -LO "https://dl.k8s.io/release/$(curl -L -s \
  https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"

# Verify checksum
curl -LO "https://dl.k8s.io/release/$(curl -L -s \
  https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl.sha256"
echo "$(cat kubectl.sha256) kubectl" | sha256sum --check
```

Expected output:
```
kubectl: OK
```

```bash
# Install kubectl
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl

# Verify
kubectl version --client
```

Expected output:
```
Client Version: v1.35.3
Kustomize Version: v5.7.1
```

```bash
# Configure kubectl for EKS
aws eks update-kubeconfig --region ap-south-1 --name studentsphere-cluster
```

Expected output:
```
Added new context arn:aws:eks:ap-south-1:207457247776:cluster/studentsphere-cluster
```

```bash
# Verify kubectl can connect to EKS
kubectl get nodes
```

Expected output:
```
NAME                                            STATUS   ROLES    AGE
ip-192-168-62-10.ap-south-1.compute.internal    Ready    <none>   2d
ip-192-168-86-169.ap-south-1.compute.internal   Ready    <none>   2d
```

```bash
# Give Jenkins user kubectl access
sudo mkdir -p /var/lib/jenkins/.kube
sudo cp ~/.kube/config /var/lib/jenkins/.kube/config
sudo chown -R jenkins:jenkins /var/lib/jenkins/.kube
sudo chmod 600 /var/lib/jenkins/.kube/config

# Verify Jenkins can access EKS
sudo -u jenkins kubectl get nodes
```

Expected output:
```
NAME                                            STATUS   ROLES    AGE
ip-192-168-62-10.ap-south-1.compute.internal    Ready    <none>   2d
ip-192-168-86-169.ap-south-1.compute.internal   Ready    <none>   2d
```

---

### Step 8 — Node.js 20 Install

```bash
# Add Node.js 20 repository
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -

# Install Node.js
sudo apt install -y nodejs
```

```bash
# Verify
node --version
npm --version
```

Expected output:
```
v20.20.2
10.8.2
```

---

### Step 9 — Maven Install

```bash
sudo apt install -y maven
```

```bash
# Verify Maven with Java 17
mvn --version
```

Expected output:
```
Apache Maven 3.8.7
Maven home: /usr/share/maven
Java version: 17.0.18, vendor: Ubuntu
```

> **Important:** If Maven shows Java 21 instead of 17, run:
> ```bash
> source /etc/profile.d/java.sh
> sudo systemctl restart jenkins
> ```

---

### Step 10 — Trivy Install

```bash
# Add Trivy repository
sudo apt-get install -y wget apt-transport-https gnupg

wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | \
  sudo apt-key add -

echo "deb https://aquasecurity.github.io/trivy-repo/deb generic main" | \
  sudo tee /etc/apt/sources.list.d/trivy.list

# Install Trivy
sudo apt update
sudo apt install -y trivy
```

```bash
# Verify
trivy --version
```

Expected output:
```
Version: 0.69.3
```

---

### Step 11 — Verify All Tools

```bash
echo "=== JAVA ===" && java -version
echo "=== MAVEN ===" && mvn --version
echo "=== DOCKER ===" && docker --version
echo "=== KUBECTL ===" && kubectl version --client
echo "=== TRIVY ===" && trivy --version
echo "=== NODE ===" && node --version
echo "=== NPM ===" && npm --version
echo "=== AWS CLI ===" && aws --version
echo "=== JENKINS ===" && sudo systemctl status jenkins --no-pager | head -3
```

Expected output:
```
=== JAVA ===
openjdk version "17.0.18"

=== MAVEN ===
Apache Maven 3.8.7 — Java version: 17.0.18

=== DOCKER ===
Docker version 29.1.3

=== KUBECTL ===
Client Version: v1.35.3

=== TRIVY ===
Version: 0.69.3

=== NODE ===
v20.20.2

=== NPM ===
10.8.2

=== AWS CLI ===
aws-cli/2.34.26

=== JENKINS ===
Active: active (running)
```

---

### Step 12 — Install Jenkins Plugins

```
Manage Jenkins → Plugins → Available Plugins

Search and install:
1. Docker Pipeline
2. Docker Commons
3. Kubernetes CLI
4. Git
5. Pipeline
6. GitHub Integration
7. Credentials Binding
8. Blue Ocean

After installation:
✅ Check "Restart Jenkins when installation is complete"
```

---

### Step 13 — Add Credentials to Jenkins

```
Manage Jenkins → Credentials → System
→ Global credentials (unrestricted) → Add Credentials
```

DockerHub Credentials:
```
Kind:        Username with password
Scope:       Global
Username:    your-dockerhub-username
Password:    your-dockerhub-password
ID:          dockerhub-credentials
Description: Docker Hub Credentials for StudentSphere
```

---

### Step 14 — Create Pipeline Job

```
Jenkins Dashboard → New Item
Name: studentsphere-pipeline
Type: Pipeline → OK

Pipeline section:
  Definition:     Pipeline script from SCM
  SCM:            Git
  Repository URL: https://github.com/your-username/multi-cloud-devops-studentsphere.git
  Branch:         */main
  Script Path:    Jenkinsfile
→ Save
```

---

### Step 15 — Run Pipeline

```
studentsphere-pipeline → Build Now
```

Expected Stage View:
```
✅ Declarative: Checkout SCM   — 1s
✅ Git Checkout                — ~700ms
✅ Backend Maven Build         — ~11s
✅ Frontend npm Build          — ~10s
✅ Trivy Security Scan         — ~11s
✅ Docker Build                — ~29s
✅ Docker Push to DockerHub    — ~11s
✅ Deploy to EKS               — ~2min 18s
✅ Declarative: Post Actions   — ~200ms

Total time: ~3min 37sec
```

```bash
# Verify new pods deployed on EKS
kubectl get pods -n studentsphere
```

Expected output:
```
NAME                        READY   STATUS    RESTARTS   AGE
backend-xxxx                1/1     Running   0          2m
backend-xxxx                1/1     Running   0          2m
frontend-xxxx               1/1     Running   0          2m
frontend-xxxx               1/1     Running   0          2m
mariadb-0                   1/1     Running   0          2d
```

---

## 📸 Output / Proof

### Pipeline Success — All Stages Green
![Pipeline Success](screenshots/01-pipeline-success.png)

### Console Output
![Console Output](screenshots/02-console-output.png)

### DockerHub Images Pushed
![DockerHub](screenshots/03-dockerhub-images.png)

### EKS Pods Updated
![EKS Updated](screenshots/04-eks-pods-updated.png)

---

## 🐛 Troubleshooting

### Problem 1 — Maven Build Failed: release version not supported
```
Error: Fatal error compiling: error: release version 17/21 not supported

Root Cause: Maven using wrong Java version

Fix:
sudo bash -c 'cat > /etc/profile.d/java.sh << EOF
export JAVA_HOME=/usr/lib/jvm/java-17-openjdk-amd64
export PATH=\$JAVA_HOME/bin:\$PATH
EOF'
source /etc/profile.d/java.sh
sudo systemctl restart jenkins
```

### Problem 2 — Jenkins GPG Key Error on Ubuntu 24.04
```
Error: NO_PUBKEY 7198F4B714ABFC68

Fix:
sudo wget -O /etc/apt/keyrings/jenkins-keyring.asc \
  https://pkg.jenkins.io/debian-stable/jenkins.io-2026.key
```

### Problem 3 — Docker Permission Denied in Jenkins
```
Error: permission denied while trying to connect to Docker daemon

Fix:
sudo usermod -aG docker jenkins
sudo systemctl restart jenkins
```

### Problem 4 — kubectl Cannot Connect to EKS
```
Error: Unable to locate AWS credentials

Fix:
sudo mkdir -p /var/lib/jenkins/.aws
sudo cp ~/.aws/credentials /var/lib/jenkins/.aws/
sudo cp ~/.aws/config /var/lib/jenkins/.aws/
sudo chown -R jenkins:jenkins /var/lib/jenkins/.aws
```

### Problem 5 — Plugin List Empty in Jenkins
```
Error: Available plugins list is empty

Fix:
Manage Jenkins → Plugins → Advanced Settings
→ Update Site → Submit → Refresh page (F5)
```

### Problem 6 — Port 8080 Already in Use
```
Error: Jenkins cannot start — port 8080 in use

Fix:
sudo lsof -i :8080
# Stop the conflicting service or use different port
```

---

## 🔗 Related Repositories

| Repository | Purpose |
|---|---|
| [multi-cloud-devops-studentsphere](https://github.com/manesaurabh1704-devops/multi-cloud-devops-studentsphere) | Main project — Full DevOps system |
| [kubernetes-production-setup](https://github.com/manesaurabh1704-devops/kubernetes-production-setup) | Kubernetes manifests |
| [terraform-multi-cloud-infra](https://github.com/manesaurabh1704-devops/terraform-multi-cloud-infra) | Infrastructure as Code |
| [monitoring-observability-stack](https://github.com/manesaurabh1704-devops/monitoring-observability-stack) | Prometheus + Grafana |
| [devops-security-secrets](https://github.com/manesaurabh1704-devops/devops-security-secrets) | RBAC + Security |

---

## 👨‍💻 Author
**Saurabh Mane** — DevOps Engineer
- GitHub: [@manesaurabh1704-devops](https://github.com/manesaurabh1704-devops)

---

> ⭐ Star this repo if you find it helpful!
