# ci-cd-devops-pipelines
# CI/CD DevOps Pipelines Jenkins CI/CD pipelines for StudentSphere project — Build, Test, Trivy Scan, Docker Push, EKS Deploy 
🏗️ CI/CD DevOps Pipelines

Production-grade Jenkins CI/CD pipeline for StudentSphere application.
Part of the multi-cloud-devops-studentsphere project.


📁 Repository Structure
ci-cd-devops-pipelines/
├── jenkins/
│   └── Jenkinsfile          # Production Jenkins pipeline
├── screenshots/             # Proof of working pipeline
└── README.md

🔄 Pipeline Architecture
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

📋 Pipeline Stages
StageToolDescriptionGit CheckoutGitPull latest code from GitHubBackend Maven BuildMaven + Java 17Build Spring Boot JARFrontend npm BuildNode.js 20Build React production bundleTrivy Security ScanTrivyScan for HIGH/CRITICAL vulnerabilitiesDocker BuildDockerBuild multi-stage imagesDocker PushDockerHubPush images with build number tagDeploy to EKSkubectlRolling update on AWS EKS

⚡ How to Setup Jenkins Pipeline
Prerequisites
- Jenkins 2.x installed
- Java 17 installed
- Docker installed
- kubectl configured for EKS
- AWS CLI configured
- Node.js 20 installed
- Maven installed
- Trivy installed
Step 1 — Install Required Plugins
Manage Jenkins → Plugins → Available Plugins

Install:
- Docker Pipeline
- Docker Commons
- Kubernetes CLI
- Git
- Pipeline
- GitHub Integration
- Credentials Binding
- Blue Ocean
Step 2 — Add Credentials
Manage Jenkins → Credentials → System → Global → Add Credentials

1. DockerHub Credentials:
   Kind:     Username with password
   Username: your-dockerhub-username
   Password: your-dockerhub-password
   ID:       dockerhub-credentials

2. AWS Credentials:
   Kind:   Secret text
   Secret: your-aws-secret-key
   ID:     aws-ecr-credentials
Step 3 — Configure Jenkins Java 17
bash# Set JAVA_HOME for Maven compatibility
sudo bash -c 'cat > /etc/profile.d/java.sh << EOF
export JAVA_HOME=/usr/lib/jvm/java-17-openjdk-amd64
export PATH=\$JAVA_HOME/bin:\$PATH
EOF'

source /etc/profile.d/java.sh
Step 4 — Give Jenkins Docker + kubectl Access
bash# Docker access
sudo usermod -aG docker jenkins

# kubectl access
sudo mkdir -p /var/lib/jenkins/.kube
sudo cp ~/.kube/config /var/lib/jenkins/.kube/config
sudo chown -R jenkins:jenkins /var/lib/jenkins/.kube

# AWS credentials
sudo mkdir -p /var/lib/jenkins/.aws
sudo cp ~/.aws/credentials /var/lib/jenkins/.aws/credentials
sudo cp ~/.aws/config /var/lib/jenkins/.aws/config
sudo chown -R jenkins:jenkins /var/lib/jenkins/.aws

# Restart Jenkins
sudo systemctl restart jenkins
Step 5 — Create Pipeline Job
New Item → studentsphere-pipeline → Pipeline → OK

Pipeline Definition:
  Definition:     Pipeline script from SCM
  SCM:            Git
  Repository URL: https://github.com/manesaurabh1704-devops/multi-cloud-devops-studentsphere.git
  Branch:         */main
  Script Path:    Jenkinsfile

Save → Build Now
Step 6 — Verify Build Success
bash# Check EKS pods updated
kubectl get pods -n studentsphere
kubectl get deployments -n studentsphere

# Check DockerHub for new image tags
# hub.docker.com → your-username → studentsphere-backend/frontend

📸 Output / Proof
Pipeline Success — All Stages Green
Show Image
Console Output
Show Image
DockerHub Images Pushed
Show Image
EKS Pods Updated
Show Image

🐛 Troubleshooting
Problem 1 — Maven Build Failed: release version not supported
Error: Fatal error compiling: error: release version 17/21 not supported

Root Cause: Maven using wrong Java version

Fix:
sudo bash -c 'cat > /etc/profile.d/java.sh << EOF
export JAVA_HOME=/usr/lib/jvm/java-17-openjdk-amd64
export PATH=\$JAVA_HOME/bin:\$PATH
EOF'
source /etc/profile.d/java.sh
sudo systemctl restart jenkins
Problem 2 — Jenkins GPG Key Error on Ubuntu 24.04
Error: NO_PUBKEY 7198F4B714ABFC68

Fix: Use official 2026 key
sudo wget -O /etc/apt/keyrings/jenkins-keyring.asc \
  https://pkg.jenkins.io/debian-stable/jenkins.io-2026.key
Problem 3 — Docker Permission Denied in Jenkins
Error: permission denied while trying to connect to Docker

Fix:
sudo usermod -aG docker jenkins
sudo systemctl restart jenkins
Problem 4 — kubectl Cannot Connect to EKS
Error: Unable to locate AWS credentials

Fix:
sudo mkdir -p /var/lib/jenkins/.aws
sudo cp ~/.aws/credentials /var/lib/jenkins/.aws/
sudo chown -R jenkins:jenkins /var/lib/jenkins/.aws
Problem 5 — Plugin List Empty in Jenkins
Error: Available plugins list is empty

Fix:
Manage Jenkins → Plugins → Advanced Settings
→ Update Site → Submit
→ Refresh page

🔗 Related Repositories
RepositoryPurposemulti-cloud-devops-studentsphereMain project — Full DevOps systemkubernetes-production-setupKubernetes manifeststerraform-multi-cloud-infraInfrastructure as Codemonitoring-observability-stackPrometheus + Grafanadevops-security-secretsRBAC + Security

👨‍💻 Author
Saurabh Mane — DevOps Engineer

GitHub: @manesaurabh1704-devops



⭐ Star this repo if you find it helpful!
