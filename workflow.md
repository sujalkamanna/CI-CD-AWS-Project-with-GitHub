# 🚀 Complete CI/CD AWS DevOps Project - Beginner's Guide

> **Build a Production-Ready Automated Deployment Pipeline**

---

## 📑 Quick Navigation

- [Project Overview](#-project-overview)
- [Architecture Diagram](#-architecture-diagram)
- [Prerequisites](#-prerequisites)
- [Complete Step-by-Step Guide](#-complete-step-by-step-guide)
- [Monitoring & Troubleshooting](#-monitoring--troubleshooting)
- [Advanced Enhancements](#-advanced-enhancements)
- [Kubernetes Deployment](#-kubernetes-deployment-with-eks)
- [AWS Resources](#-aws-resources)
- [Best Practices](#-best-practices)

---

## 🎯 Project Overview

This guide walks you through building an **end-to-end CI/CD pipeline** that automatically:
- ✅ Detects code changes on GitHub
- ✅ Builds Docker images
- ✅ Pushes to AWS ECR
- ✅ Deploys to ECS Fargate
- ✅ Updates live website

### 💡 Difficulty Level: Beginner-Friendly
### 🎓 Skills Learned: DevOps, AWS, Docker, CI/CD

---

## 🏗️ Architecture Diagram

```
┌────────────────────────────────────────────────────────────────────────┐
│                        CI/CD PIPELINE ARCHITECTURE                     │
├────────────────────────────────────────────────────────────────────────┤
│                                                                        │
│   Developer              Source Control       Build & Push             │
│   ┌──────────┐           ┌──────────┐         ┌──────────┐             │
│   │  GitHub  │◄─────────►│  GitHub  │────────→│CodeBuild │             │
│   │ Repository│  Push    │   API    │ Trigger │ (Docker) │             │
│   └──────────┘  Changes  └──────────┘         └──────────┘             │
│       │                                             │                  │
│       │                                             ├─ Build Image     │
│       │                                             ├─ Run Tests       │
│       │                                             └─ Create Artifact │
│       │                                                   │            │
│       │                         ┌─────────────────────────┘            │
│       │                         │                                      │
│       │                   ┌─────▼───────┐                              │
│       │                   │   Amazon   │                               │
│       │                   │    ECR     │  (Docker Registry)            │
│       │                   └─────┬──────┘                               │
│       │                         │                                      │
│       │         ┌───────────────┴───────────────┐                      │
│       │         │                               │                      │
│   ┌───▼─────────▼──────┐              ┌─────────▼───────┐              │
│   │   CodePipeline     │              │  Amazon ECS     │              │
│   │ (Orchestration)    │─────────────→│ (Fargate)       │              │
│   │                    │   Deploy     │                 │              │
│   └────────────────────┘              └───────┬─────────┘              │
│                                               │                        │
│                                        ┌──────▼─────────┐              │
│                                        │  Load Balancer │              │
│                                        │      (ALB)     │              │
│                                        └──────┬─────────┘              │
│                                               │                        │
│                                        ┌──────▼─────────┐              │
│                                        │   Website      │              │
│                                        │   (Live)       │              │
│                                        └────────────────┘              │
│                                                                        │
└────────────────────────────────────────────────────────────────────────┘
```

### 📍 Pipeline Flow Breakdown:

```
CODE PUSH
   ↓
GITHUB WEBHOOK TRIGGERS
   ↓
CODEPIPELINE STARTS
   ↓
CODEBUILD:
  • Clone repo
  • Build Docker image
  • Run tests
  • Push to ECR
   ↓
ECS DEPLOYMENT:
  • Pull image from ECR
  • Stop old containers
  • Start new containers
  • Health checks
   ↓
LOAD BALANCER:
  • Route traffic
  • Health monitoring
   ↓
LIVE WEBSITE UPDATED ✅
```

---

## ✅ Prerequisites

### AWS Account Setup
- [ ] AWS Account created (free tier eligible)
- [ ] IAM user with programmatic access
- [ ] AWS CLI configured locally

### Local Machine Requirements
- [ ] Git installed
- [ ] Docker installed (for local testing)
- [ ] SSH client (for EC2 connection)
- [ ] Text editor

### GitHub Repository
- [ ] GitHub account
- [ ] Repository with Dockerfile
- [ ] buildspec.yml in repository

---

## 🎬 Complete Step-by-Step Guide

---

## 🟢 STEP 1: Launch EC2 Instance

### Why EC2?
EC2 serves as your development environment to test Docker locally before pushing to production.

### Detailed Instructions:

#### 1.1 Open AWS Console
```
https://console.aws.amazon.com
→ Services → EC2
```

#### 1.2 Launch New Instance
```
EC2 Dashboard → Instances → Launch Instance
```

#### 1.3 Select Amazon Linux 2
```
Select "Amazon Linux 2" AMI
→ Free tier eligible ✓
```

#### 1.4 Configure Instance
```
Instance Type: t2.micro (Free tier)
Network: Default VPC
Storage: 8 GB (default)
```

#### 1.5 Security Group Configuration
```
Create new security group:
┌─────────────────────────────────┐
│ Rule Type   │ Port  │ Source    │
├─────────────────────────────────┤
│ SSH         │ 22    │ Your IP   │
│ HTTP        │ 80    │ 0.0.0.0/0 │
│ HTTPS       │ 443   │ 0.0.0.0/0 │
└─────────────────────────────────┘
```

#### 1.6 Create Key Pair
```
Download: your-key.pem
Keep safe - never share!
Permissions: chmod 400 your-key.pem
```

#### 1.7 Launch & Wait
```
Status: "Running" ✓
Public IPv4: Copy this for SSH
```

### ✅ Verification:
```bash
# SSH into instance
ssh -i your-key.pem ec2-user@<PUBLIC-IP>

# Should see EC2 prompt
[ec2-user@ip-xxx-xxx-xxx-xxx ~]$
```

---

## 🟢 STEP 2: Install Docker & Git

### 2.1 Update System Packages
```bash
sudo yum update -y
```

### 2.2 Install Docker

```bash
# Install Docker
sudo yum install docker -y

# Start Docker service
sudo service docker start

# Enable auto-start on reboot
sudo systemctl enable docker

# Add user to docker group (run without sudo)
sudo usermod -aG docker ec2-user

# Apply group changes
newgrp docker
```

### Verify Docker Installation:
```bash
docker --version
# Output: Docker version 20.10.x, build xxxxx
```

### 2.3 Install Git

```bash
# Install Git
sudo yum install git -y

# Verify installation
git --version
# Output: git version 2.x.x
```

### Configure Git (Optional):
```bash
git config --global user.name "Your Name"
git config --global user.email "your-email@example.com"
```

### ✅ Verification:
```bash
# Test Docker
docker images
# Output: REPOSITORY   TAG   IMAGE ID   CREATED   SIZE
# (empty list is fine)

# Test Git
git --version
# Output: git version 2.x.x
```

---

## 🟢 STEP 3: Clone GitHub Repository

### 3.1 Clone Your Project
```bash
# Clone repository
git clone https://github.com/ReyazShaik/website.git

# Navigate into project
cd website

# View project structure
ls -la
```

### Expected Output:
```
drwxr-xr-x  Dockerfile
drwxr-xr-x  buildspec.yml
drwxr-xr-x  css/
drwxr-xr-x  js/
drwxr-xr-x  images/
drwxr-xr-x  index.html
drwxr-xr-x  about.html
drwxr-xr-x  service.html
drwxr-xr-x  contact.html
drwxr-xr-x  guard.html
```

### 3.2 View Dockerfile
```bash
cat Dockerfile
```

**Expected Content:**
```dockerfile
FROM nginx:latest
COPY . /usr/share/nginx/html/
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

### 📝 Understanding Dockerfile:
```
FROM nginx:latest          ← Base image (web server)
COPY . /usr/share/...     ← Copy website files into image
EXPOSE 80                 ← Listen on port 80
CMD ["nginx", "-g" ...]   ← Start web server
```

### ✅ Verification:
```bash
# Confirm you're in correct directory
pwd
# Output: /home/ec2-user/website

# Confirm Dockerfile exists
file Dockerfile
# Output: Dockerfile: ASCII text
```

---

## 🟢 STEP 4: Build & Test Docker Image Locally

### 4.1 Build Docker Image

```bash
# Build image with tag 'website:latest'
docker build -t website:latest .

# This may take 1-2 minutes
```

### Build Output Explanation:
```
Sending build context to Docker daemon...    ← Preparing files
Step 1/4 : FROM nginx:latest                  ← Pulling base image
Step 2/4 : COPY . /usr/share/nginx/html/    ← Copying files
Step 3/4 : EXPOSE 80                         ← Opening port
Step 4/4 : CMD ["nginx", "-g" ...]          ← Setting command
Successfully built abc123def456               ← Success ✅
Successfully tagged website:latest
```

### 4.2 Verify Image Created

```bash
docker images
```

### Expected Output:
```
REPOSITORY   TAG      IMAGE ID       CREATED         SIZE
website      latest   abc123def456   2 minutes ago   142MB
nginx        latest   def456ghi789   3 weeks ago     142MB
```

### 4.3 Run Container

```bash
# Run container
# -d = detached (background)
# -p = port mapping (host:container)
docker run -d -p 80:80 website:latest

# Get container ID (first 12 characters)
# Output: abc123def456
```

### 4.4 Verify Container Running

```bash
docker ps
```

### Expected Output:
```
CONTAINER ID    IMAGE            PORTS              STATUS
abc123def456    website:latest   0.0.0.0:80->80/tcp   Up 2 minutes
```

### 4.5 Access Website

```bash
# Get EC2 public IP
curl http://169.254.169.254/latest/meta-data/public-ipv4
# Output: 54.xxx.xxx.xxx

# Open in browser: http://54.xxx.xxx.xxx
# You should see your website! 🎉
```

### 4.6 Stop Container (for cleanup)

```bash
# Find container
docker ps

# Stop container
docker stop <CONTAINER-ID>

# Verify stopped
docker ps
# (should be empty)
```

### ✅ Verification:
```
✓ Docker image built successfully
✓ Container started without errors
✓ Website accessible via browser
✓ Port 80 responds correctly
```

---

## 🟢 STEP 5: Push Docker Image to AWS ECR

### What's ECR?
**Elastic Container Registry** - AWS's private Docker registry. Think of it as your personal Docker Hub.

### 5.1 Create ECR Repository

#### Via AWS Console:
```
AWS Console
→ Services → ECR
→ Repositories → Create Repository
```

#### Configuration:
```
Repository Name: website
Scan images: (optional - for security)
Encryption: (default - fine)
Click: Create repository
```

#### Via AWS CLI:
```bash
aws ecr create-repository \
  --repository-name website \
  --region ap-south-1
```

**Output:**
```json
{
    "repository": {
        "repositoryUri": "123456789.dkr.ecr.ap-south-1.amazonaws.com/website",
        "repositoryName": "website"
    }
}
```

**Save the URI!** You'll need it later.

### 5.2 Attach IAM Role to EC2

**Why?** EC2 needs permission to push images to ECR.

#### Step 1: Create IAM Role
```
AWS Console
→ IAM → Roles
→ Create Role
→ Select "EC2"
→ Next
```

#### Step 2: Attach Policy
```
Search for: AmazonEC2ContainerRegistryFullAccess
Click: Attach Policy
Role Name: EC2-ECR-Role
Click: Create
```

#### Step 3: Attach to EC2
```
EC2 Console → Instances
→ Select your instance
→ Instance State → Modify IAM Role
→ Select: EC2-ECR-Role
→ Update
```

#### Alternative (via CLI):
```bash
# Create role
aws iam create-role \
  --role-name EC2-ECR-Role \
  --assume-role-policy-document file://trust-policy.json

# Attach policy
aws iam attach-role-policy \
  --role-name EC2-ECR-Role \
  --policy-arn arn:aws:iam::aws:policy/AmazonEC2ContainerRegistryFullAccess

# Attach to EC2 instance
aws ec2 associate-iam-instance-profile \
  --instance-id i-xxxxxxxx \
  --iam-instance-profile Name=EC2-ECR-Role
```

### 5.3 Login to ECR

```bash
# Get login token and login
aws ecr get-login-password --region ap-south-1 | \
  docker login --username AWS --password-stdin \
  123456789.dkr.ecr.ap-south-1.amazonaws.com
```

**Expected Output:**
```
Login Succeeded
```

### 5.4 Tag Docker Image

```bash
# Tag with ECR repository URL
docker tag website:latest \
  123456789.dkr.ecr.ap-south-1.amazonaws.com/website:latest
```

### Verify Tag:
```bash
docker images
```

**Expected Output:**
```
REPOSITORY                                    TAG        IMAGE ID
website                                       latest     abc123def456
123456789.dkr.ecr.ap-south-1.amazonaws.com/website  latest  abc123def456
```

### 5.5 Push Image to ECR

```bash
# Push image
docker push 123456789.dkr.ecr.ap-south-1.amazonaws.com/website:latest
```

### Push Progress:
```
Pushing image to ECR...
Layer 1: ████████████████████████ 100%
Layer 2: ████████████████████████ 100%
...
Successfully pushed image
```

### ✅ Verification:

```bash
# Verify in ECR console
AWS Console → ECR → Repositories → website

# Or via CLI
aws ecr describe-images --repository-name website --region ap-south-1
```

**Expected Output:**
```json
{
    "imageDetails": [
        {
            "registryId": "123456789",
            "repositoryName": "website",
            "imageId": {
                "imageTag": "latest",
                "imageDigest": "sha256:abc123..."
            },
            "imageSizeInBytes": 142000000
        }
    ]
}
```

---

## 🟢 STEP 6: Create ECS Cluster (Fargate)

### What's ECS?
**Elastic Container Service** - AWS service that runs Docker containers at scale. **Fargate** means serverless (no servers to manage).

### 6.1 Create Cluster via Console

```
AWS Console → ECS
→ Clusters → Create Cluster
```

### Cluster Configuration:
```
┌─────────────────────────────────────────┐
│ Cluster Name: websitecluster            │
│ Infrastructure: AWS Fargate             │
│ VPC: Default VPC                        │
│ Subnets: All (default)                  │
│ Security Group: Default                 │
│ CloudWatch Logs: Enable (optional)      │
│ Click: Create                           │
└─────────────────────────────────────────┘
```

### Via CLI:
```bash
aws ecs create-cluster --cluster-name websitecluster
```

**Output:**
```json
{
    "cluster": {
        "clusterName": "websitecluster",
        "status": "ACTIVE"
    }
}
```

### ✅ Verification:
```
AWS Console → ECS → Clusters
→ Should see "websitecluster" with status "ACTIVE"
```

---

## 🟢 STEP 7: Create Task Definition

### What's a Task Definition?
A **Task Definition** is like a blueprint for your Docker container. It specifies:
- Which Docker image to use
- How much CPU/memory to allocate
- Which ports to expose
- Environment variables
- Logging configuration

### 7.1 Create via Console

```
AWS Console → ECS
→ Task Definitions → Create New Task Definition
```

### Configuration:

```
┌────────────────────────────────────────┐
│ Launch Type: AWS Fargate               │
│ Family: website-task                   │
│ Requires Compatibilities: FARGATE      │
│                                        │
│ CPU: 256 (.25 vCPU)                    │
│ Memory: 512 MB                         │
│                                        │
│ Container Details:                     │
│ ├─ Name: website                       │
│ ├─ Image: 123456...ecr.../website:latest
│ ├─ Port: 80                            │
│ ├─ Protocol: tcp                       │
│ └─ Essential: Yes                      │
│                                        │
│ Click: Create                          │
└────────────────────────────────────────┘
```

### Via CLI:
```bash
cat > task-definition.json << 'EOF'
{
  "family": "website-task",
  "networkMode": "awsvpc",
  "requiresCompatibilities": ["FARGATE"],
  "cpu": "256",
  "memory": "512",
  "containerDefinitions": [
    {
      "name": "website",
      "image": "123456789.dkr.ecr.ap-south-1.amazonaws.com/website:latest",
      "portMappings": [
        {
          "containerPort": 80,
          "hostPort": 80,
          "protocol": "tcp"
        }
      ],
      "logConfiguration": {
        "logDriver": "awslogs",
        "options": {
          "awslogs-group": "/ecs/website",
          "awslogs-region": "ap-south-1",
          "awslogs-stream-prefix": "ecs"
        }
      }
    }
  ]
}
EOF

aws ecs register-task-definition --cli-input-json file://task-definition.json
```

### ✅ Verification:
```bash
aws ecs describe-task-definition \
  --task-definition website-task \
  --region ap-south-1
```

---

## 🟢 STEP 8: Create Application Load Balancer

### Why ALB?
The load balancer distributes traffic across multiple container instances and provides a stable endpoint.

### 8.1 Create ALB via Console

```
AWS Console → EC2
→ Load Balancers → Create Load Balancer
→ Choose: Application Load Balancer
```

### Configuration:

```
┌────────────────────────────────────────┐
│ Name: website-alb                      │
│ Scheme: Internet-facing                │
│ IP Address Type: IPv4                  │
│                                        │
│ Network Configuration:                 │
│ ├─ VPC: Default VPC                    │
│ ├─ Subnets: All (2+)                   │
│                                        │
│ Security Groups:                       │
│ ├─ Allow HTTP (80)                     │
│ ├─ Allow HTTPS (443)                   │
│                                        │
│ Listeners and Rules:                   │
│ ├─ Protocol: HTTP                      │
│ ├─ Port: 80                            │
│ └─ Forward to: New Target Group        │
│                                        │
│ Target Group:                          │
│ ├─ Name: website-tg                    │
│ ├─ Type: IP                            │
│ ├─ Port: 80                            │
│ └─ Health checks: /                    │
│                                        │
│ Click: Create                          │
└────────────────────────────────────────┘
```

### ✅ Verification:
```
AWS Console → EC2 → Load Balancers
→ Find "website-alb"
→ Copy DNS Name
→ Open in browser (will be empty until service is created)
```

---

## 🟢 STEP 9: Create ECS Service

### What's a Service?
A **Service** keeps your containers running. It automatically:
- Restarts failed containers
- Balances load across containers
- Manages auto-scaling
- Integrates with load balancers

### 9.1 Create Service via Console

```
AWS Console → ECS
→ Clusters → websitecluster
→ Services → Create
```

### Configuration:

```
┌────────────────────────────────────────┐
│ Launch Type: FARGATE                   │
│ Task Definition: website-task:latest   │
│ Service Name: website-service          │
│ Desired Count: 2                       │
│                                        │
│ Network Configuration:                 │
│ ├─ VPC: Default VPC                    │
│ ├─ Subnets: All                        │
│ ├─ Security Groups: Create new         │
│ │  └─ Allow port 80 from ALB           │
│ └─ Public IP: ENABLED                  │
│                                        │
│ Load Balancer:                         │
│ ├─ Type: Application Load Balancer     │
│ ├─ Name: website-alb                   │
│ ├─ Container: website:80               │
│ └─ Service: website-tg                 │
│                                        │
│ Auto Scaling:                          │
│ ├─ Desired Count: 2                    │
│ └─ Min/Max: 1-4                        │
│                                        │
│ Click: Create Service                  │
└────────────────────────────────────────┘
```

### Via CLI:
```bash
aws ecs create-service \
  --cluster websitecluster \
  --service-name website-service \
  --task-definition website-task:1 \
  --desired-count 2 \
  --launch-type FARGATE \
  --network-configuration "awsvpcConfiguration={subnets=[subnet-xxx],assignPublicIp=ENABLED}" \
  --load-balancers "targetGroupArn=arn:aws:elasticloadbalancing:...,containerName=website,containerPort=80" \
  --region ap-south-1
```

### ⏳ Wait for Deployment:
```
The service takes 2-3 minutes to start all tasks.

Status progression:
PROVISIONING → PENDING → RUNNING ✅

You can see progress in:
AWS Console → ECS → Services → website-service
→ Tasks tab
```

### ✅ Verification:

```bash
# Check service status
aws ecs describe-services \
  --cluster websitecluster \
  --services website-service \
  --region ap-south-1
```

```bash
# Check running tasks
aws ecs list-tasks \
  --cluster websitecluster \
  --region ap-south-1
```

### 🌐 Access Website:

```
AWS Console → EC2 → Load Balancers
→ Select "website-alb"
→ Copy DNS Name
→ Open in browser: http://<dns-name>

You should see your website! 🎉
```

---

## 🟢 STEP 10: Setup AWS CodeBuild

### What's CodeBuild?
**CodeBuild** automatically compiles code, runs tests, and produces Docker images.

### 10.1 Create Build Project

```
AWS Console → CodeBuild
→ Create Build Project
```

### Configuration:

```
┌────────────────────────────────────────┐
│ Project Name: website-build            │
│                                        │
│ Source:                                │
│ ├─ Source Provider: GitHub             │
│ ├─ Repository: website                 │
│ ├─ Branch: main                        │
│ └─ Use Git Submodules: No              │
│                                        │
│ Environment:                           │
│ ├─ Managed Image: Yes                  │
│ ├─ Operating System: Amazon Linux 2    │
│ ├─ Runtime: Docker                     │
│ ├─ Image: Standard                     │
│ └─ Privileged Mode: Enable             │
│    (needed for Docker builds)          │
│                                        │
│ Buildspec:                             │
│ ├─ Use a buildspec file                │
│ └─ buildspec.yml location: root        │
│                                        │
│ Artifacts:                             │
│ └─ Type: No artifacts                  │
│                                        │
│ Logs:                                  │
│ └─ CloudWatch Logs: Enable             │
│    Group: /aws/codebuild/website-build │
│                                        │
│ Click: Create                          │
└────────────────────────────────────────┘
```

### 10.2 Create buildspec.yml

Add this file to your GitHub repository root:

```yaml
version: 0.2

# ============================================
# ENVIRONMENT VARIABLES
# ============================================
env:
  variables:
    AWS_DEFAULT_REGION: ap-south-1
    AWS_ACCOUNT_ID: 123456789          # Replace with your account ID
    IMAGE_REPO_NAME: website
    IMAGE_TAG: latest

# ============================================
# BUILD PHASES
# ============================================
phases:
  # Phase 1: Pre-build (Setup)
  pre_build:
    commands:
      - echo "========== PRE-BUILD PHASE =========="
      - echo "Logging into Amazon ECR..."
      - aws ecr get-login-password --region $AWS_DEFAULT_REGION | docker login --username AWS --password-stdin $AWS_ACCOUNT_ID.dkr.ecr.$AWS_DEFAULT_REGION.amazonaws.com
      - REPOSITORY_URI=$AWS_ACCOUNT_ID.dkr.ecr.$AWS_DEFAULT_REGION.amazonaws.com/$IMAGE_REPO_NAME
      - COMMIT_HASH=$(echo $CODEBUILD_RESOLVED_SOURCE_VERSION | cut -c 1-7)
      - IMAGE_TAG=${COMMIT_HASH:=latest}
      - echo "Repository URI: $REPOSITORY_URI"
      - echo "Image Tag: $IMAGE_TAG"

  # Phase 2: Build (Compile Docker image)
  build:
    commands:
      - echo "========== BUILD PHASE =========="
      - echo "Building the Docker image on `date`"
      - docker build -t $REPOSITORY_URI:$IMAGE_TAG .
      - docker tag $REPOSITORY_URI:$IMAGE_TAG $REPOSITORY_URI:latest
      - echo "Docker image built successfully"

  # Phase 3: Post-build (Push to ECR)
  post_build:
    commands:
      - echo "========== POST-BUILD PHASE =========="
      - echo "Pushing the Docker image to ECR on `date`"
      - docker push $REPOSITORY_URI:$IMAGE_TAG
      - docker push $REPOSITORY_URI:latest
      - echo "Image push successful"
      - echo "Writing image definitions file..."
      - printf '[{"name":"website","imageUri":"%s"}]' $REPOSITORY_URI:$IMAGE_TAG > imagedefinitions.json
      - cat imagedefinitions.json

# ============================================
# OUTPUT ARTIFACTS
# ============================================
artifacts:
  files:
    - imagedefinitions.json
  name: BuildArtifact

# ============================================
# LOGS
# ============================================
logs:
  cloudwatch:
    group-name: /aws/codebuild/website-build
    stream-name: website-build-logs
```

**Upload to GitHub:**
```bash
git add buildspec.yml
git commit -m "Add buildspec.yml for CI/CD"
git push origin main
```

### 10.3 Attach IAM Role to CodeBuild

**CodeBuild needs permission to push to ECR.**

```
AWS Console → CodeBuild → Select Project
→ Edit → Environment
→ Service Role: website-build-role

Edit the role to add policies:
1. AmazonEC2ContainerRegistryFullAccess
2. CloudWatchLogsFullAccess
```

### 10.4 Test Build Manually

```
AWS Console → CodeBuild
→ Select website-build project
→ Start Build
→ View logs in real-time
```

### Build Output:
```
[Container] 2024/01/15 10:30:45.123 Running command docker build -t...
Step 1/4 : FROM nginx:latest
Step 2/4 : COPY . /usr/share/nginx/html/
Step 3/4 : EXPOSE 80
Step 4/4 : CMD ["nginx", "-g" "daemon off;"]
Successfully built abc123
Successfully tagged 123456...ecr.../website:latest
[Container] 2024/01/15 10:32:15.456 Running command docker push...
Pushing image to ECR completed
```

### ✅ Verification:
```bash
# Check build succeeded
aws codebuild batch-get-builds \
  --ids <build-id> \
  --region ap-south-1

# Check image in ECR
aws ecr list-images --repository-name website --region ap-south-1
```

---

## 🟢 STEP 11: Setup AWS CodePipeline

### What's CodePipeline?
**CodePipeline** orchestrates your entire CI/CD workflow. It automatically:
- Detects code changes
- Triggers builds
- Deploys to ECS
- Provides visibility

### 11.1 Create Pipeline

```
AWS Console → CodePipeline
→ Create Pipeline
```

### Configuration:

```
┌─────────────────────────────────────────┐
│ Pipeline Settings:                      │
│ ├─ Name: website-cicd-pipeline          │
│ └─ Service Role: Create new             │
│                                         │
│ STAGE 1: Source                         │
│ ├─ Source Provider: GitHub              │
│ ├─ Repository: ReyazShaik/website       │
│ ├─ Branch: main                         │
│ └─ Detection Options: Webhook           │
│                                         │
│ STAGE 2: Build                          │
│ ├─ Build Provider: AWS CodeBuild        │
│ ├─ Project Name: website-build          │
│ └─ Single Build: (default)              │
│                                         │
│ STAGE 3: Deploy                         │
│ ├─ Deploy Provider: Amazon ECS          │
│ ├─ Cluster: websitecluster              │
│ ├─ Service Name: website-service        │
│ ├─ File Name: imagedefinitions.json     │
│                                         │
│ Click: Create Pipeline                  │
└─────────────────────────────────────────┘
```

### 11.2 Authorize GitHub

First time setup - CodePipeline needs access to your GitHub repo.

```
Click: Connect to GitHub
→ Authorize AWS Connector for GitHub
→ Select Repository
→ Click: Connect
```

### Pipeline Visualization:

```
┌──────────────┐      ┌──────────────┐      ┌──────────────┐
│   GitHub     │─────→│  CodeBuild   │─────→│  ECS Deploy  │
│  Repository  │      │  (Build)     │      │  (Fargate)   │
└──────────────┘      └──────────────┘      └──────────────┘
     Source              Build                 Deploy
```

### ✅ Verification:

```
AWS Console → CodePipeline
→ Select website-cicd-pipeline
→ View stages

Status should show:
✓ Source: Succeeded
✓ Build: Succeeded (or In Progress)
✓ Deploy: Succeeded (or In Progress)
```

---

## 🟢 STEP 12: Test Complete Pipeline

### Goal: Push code change and see automatic deployment

### 12.1 Make a Code Change

```bash
# Clone or navigate to your local repository
cd ~/website

# Edit index.html
nano index.html

# Add a test message in the page:
# <h2>Updated: CI/CD Pipeline Active!</h2>

# Save and commit
git add index.html
git commit -m "Test CI/CD pipeline"
git push origin main
```

### 12.2 Watch Pipeline Execute

```
AWS Console → CodePipeline
→ website-cicd-pipeline
→ Watch stages execute in real-time:

1. Source: Pull code ✅
2. Build: Docker build & push (2-3 min) 🔄
3. Deploy: Update ECS (1-2 min) 🔄
```

### 12.3 Verify Live Update

```
Wait 3-5 minutes for deployment
→ Go to Load Balancer DNS
→ Refresh browser
→ See your changes! 🎉
```

### ✅ Verification Timeline:
```
00:00 - Push to GitHub
00:05 - Pipeline triggered
00:08 - Build phase started
00:13 - Image pushed to ECR
00:15 - Deploy phase started
00:20 - ECS updated with new image
00:25 - Website updated ✅
```

---

## 📊 Monitoring & Troubleshooting

### 11.1 CloudWatch Logs

```
AWS Console → CloudWatch → Log Groups
→ /aws/codebuild/website-build
→ /ecs/website
```

### 11.2 CodeBuild Logs

```
AWS Console → CodeBuild
→ website-build
→ Build History → Select build
→ Logs tab
```

### Common Issues & Solutions:

| ❌ Issue | 🔧 Solution |
|---------|-----------|
| **Build fails - Permission denied** | Attach IAM role with ECR permissions |
| **Pipeline stuck at build** | Check CloudWatch logs for errors |
| **ECS tasks not starting** | Verify task definition image URI |
| **Website not updating** | Check imagedefinitions.json format |
| **Cannot connect to GitHub** | Authorize OAuth connection again |
| **Load balancer shows no targets** | Wait 2-3 minutes for health checks |

### Debugging Commands:

```bash
# Check CodeBuild logs
aws codebuild batch-get-builds \
  --ids <build-id> \
  --region ap-south-1

# Check ECS task status
aws ecs describe-tasks \
  --cluster websitecluster \
  --tasks <task-arn> \
  --region ap-south-1

# Check ECR images
aws ecr list-images \
  --repository-name website \
  --region ap-south-1

# Check service status
aws ecs describe-services \
  --cluster websitecluster \
  --services website-service \
  --region ap-south-1
```

---

## 🧹 CLEANUP (Important!)

### Avoid AWS Charges!

Delete resources in this order:

### 1️⃣ Delete ECS Service
```
AWS Console → ECS
→ Clusters → websitecluster
→ Services → website-service
→ Delete Service → Confirm
```

### 2️⃣ Delete ECS Cluster
```
AWS Console → ECS
→ Clusters → websitecluster
→ Delete Cluster → Confirm
```

### 3️⃣ Delete Load Balancer
```
AWS Console → EC2
→ Load Balancers
→ Select website-alb
→ Actions → Delete
→ Confirm
```

### 4️⃣ Delete Target Group
```
AWS Console → EC2
→ Target Groups
→ Select website-tg
→ Actions → Delete
→ Confirm
```

### 5️⃣ Delete ECR Repository
```
AWS Console → ECR
→ Repositories → website
→ Delete Repository
→ Confirm
```

### 6️⃣ Delete CodePipeline
```
AWS Console → CodePipeline
→ Select website-cicd-pipeline
→ Edit → Delete
→ Confirm
```

### 7️⃣ Delete CodeBuild Project
```
AWS Console → CodeBuild
→ Projects → website-build
→ Delete → Confirm
```

### 8️⃣ Terminate EC2 Instance
```
AWS Console → EC2
→ Instances → Select instance
→ Instance State → Terminate
→ Confirm
```

### 9️⃣ Delete Security Groups
```
AWS Console → EC2
→ Security Groups
→ Delete custom security groups
→ Confirm
```

### ✅ Verification:
```
All resources should show as deleted/terminated
Expected billing: $0 (if freed tier not exceeded)
```

---

# 🚀 ADVANCED ENHANCEMENTS

---

## 🟢 ENHANCEMENT 1: Add HTTPS/SSL Certificate

### Why HTTPS?
- Encrypts data in transit
- Builds trust with users
- Required for modern websites
- Improves SEO ranking

### Step 1: Request SSL Certificate from AWS Certificate Manager

```
AWS Console → Certificate Manager
→ Request Certificate
```

### Configuration:
```
┌────────────────────────────────────┐
│ Certificate Type:                  │
│ └─ Public Certificate              │
│                                    │
│ Domain Names:                      │
│ ├─ example.com                     │
│ └─ www.example.com                 │
│                                    │
│ Validation Method:                 │
│ └─ DNS Validation (recommended)    │
│                                    │
│ Click: Request Certificate         │
└────────────────────────────────────┘
```

### Step 2: Validate Certificate

```
AWS Console → Certificate Manager
→ Select your certificate
→ Add CNAME records to your DNS provider
→ Wait for validation (5-30 minutes)
```

### Step 3: Update Load Balancer

```
AWS Console → EC2 → Load Balancers
→ Select website-alb
→ Listeners → Add Listener
→ Protocol: HTTPS
│ Port: 443
│ Default Certificate: Your certificate
│ Default Action: Forward to website-tg
→ Save
```

### Step 4: Redirect HTTP to HTTPS (Optional)

```
AWS Console → EC2 → Load Balancers
→ Select website-alb
→ Listeners → Edit HTTP (80)
→ Default Action: Redirect to HTTPS
→ Save
```

### ✅ Verification:
```bash
# Test HTTPS
curl https://your-domain.com

# Check certificate
openssl s_client -connect your-domain.com:443
```

---

## 🟢 ENHANCEMENT 2: Add Custom Domain with Route 53

### Why Route 53?
- AWS's DNS service
- Integrates with other AWS services
- Low latency routing
- Health checks

### Step 1: Register Domain

Option A - Register with Route 53:
```
AWS Console → Route 53 → Registered Domains
→ Register Domain
→ Enter domain name
→ Complete registration (can take 24-48 hours)
```

Option B - Use existing domain registrar:
```
You can point your domain registrar's nameservers
to Route 53's nameservers
```

### Step 2: Create Hosted Zone

```
AWS Console → Route 53
→ Hosted Zones → Create Hosted Zone
→ Domain Name: example.com
→ Type: Public Hosted Zone
→ Create
```

### Step 3: Create Alias Record

```
AWS Console → Route 53
→ Hosted Zones → example.com
→ Create Record
```

### Configuration:
```
┌────────────────────────────────────┐
│ Record Name: (leave blank for root)│
│ Record Type: A                     │
│ Value: Alias to ALB                │
│ ├─ Choose Region: ap-south-1       │
│ ├─ Choose Load Balancer:           │
│ │  website-alb                     │
│ └─ Evaluate Target Health: Yes     │
│                                    │
│ Click: Create Record               │
└────────────────────────────────────┘
```

### Create WWW Record:
```
Repeat above for:
Record Name: www
Value: Alias to ALB
```

### Step 4: Update Security Group

Allow traffic on port 443:
```
AWS Console → EC2 → Security Groups
→ Select ALB security group
→ Edit Inbound Rules
→ Add Rule:
  Type: HTTPS
  Port: 443
  Source: 0.0.0.0/0
→ Save
```

### ✅ Verification:
```bash
# Test DNS resolution
nslookup example.com
# Should resolve to ALB IP

# Test HTTPS
curl https://example.com
# Should work
```

---

## 🟢 ENHANCEMENT 3: Add CloudFront CDN

### Why CloudFront?
- Caches content globally
- Reduces latency
- Lower bandwidth costs
- DDoS protection

### Step 1: Create CloudFront Distribution

```
AWS Console → CloudFront
→ Create Distribution
```

### Configuration:
```
┌──────────────────────────────────────┐
│ Origin Settings:                     │
│ ├─ Origin Domain: ALB DNS name       │
│ └─ Protocol: HTTPS                   │
│                                      │
│ Default Cache Behavior:              │
│ ├─ Viewer Protocol: HTTPS only       │
│ ├─ Cache Policy: CachingOptimized    │
│ └─ Compress: Yes                     │
│                                      │
│ CNAME (Optional):                    │
│ └─ Add: cdn.example.com              │
│                                      │
│ Default Root Object: index.html      │
│                                      │
│ Create Distribution                  │
└──────────────────────────────────────┘
```

### Step 2: Wait for Deployment

```
CloudFront deployment takes 15-30 minutes
Status will change from "In Progress" to "Deployed"
```

### Step 3: Update DNS (if using CNAME)

```
Route 53 → Create Record
Record Name: cdn
Type: CNAME
Value: <cloudfront-domain-name>.cloudfront.net
```

### ✅ Verification:
```bash
# Test CloudFront
curl https://cdn.example.com

# Check cache headers
curl -I https://cdn.example.com
# Look for X-Cache headers
```

---

## 🟢 ENHANCEMENT 4: Auto-Scaling Configuration

### Why Auto-Scaling?
- Automatically adjust capacity
- Handle traffic spikes
- Reduce costs during low usage
- Ensure reliability

### Step 1: Create Auto Scaling Group

```
AWS Console → EC2 → Auto Scaling Groups
→ Create Auto Scaling Group
```

### Configuration:
```
┌──────────────────────────────────────┐
│ Auto Scaling Group Name:             │
│ └─ website-asg                       │
│                                      │
│ ECS Service Integration:             │
│ ├─ Cluster: websitecluster           │
│ └─ Service: website-service          │
│                                      │
│ Desired Capacity: 2                  │
│ Min: 1                               │
│ Max: 4                               │
│                                      │
│ Scaling Policy: Target Tracking      │
│ ├─ Metric: CPU Utilization           │
│ └─ Target Value: 70%                 │
│                                      │
│ Create                               │
└──────────────────────────────────────┘
```

### Step 2: Monitor Scaling Activity

```
AWS Console → EC2 → Auto Scaling Groups
→ Select website-asg
→ Activity tab
→ Watch as tasks scale up/down
```

### Scaling Behaviors:
```
Low Traffic:
┌─────┐
│ 1   │ Task running
└─────┘

High Traffic:
┌─────┬─────┬─────┬─────┐
│ 2   │ 3   │ 4   │ 4   │ Tasks running
└─────┴─────┴─────┴─────┘

Back to Normal:
┌─────┐
│ 2   │ Tasks running
└─────┘
```

### ✅ Verification:
```bash
# Monitor scaling
aws ecs describe-services \
  --cluster websitecluster \
  --services website-service \
  --region ap-south-1 \
  | grep runningCount

# Check Auto Scaling history
aws autoscaling describe-scaling-activities \
  --auto-scaling-group-name website-asg \
  --region ap-south-1
```

---

## 🟢 ENHANCEMENT 5: Blue-Green Deployment

### What is Blue-Green?
Two identical environments where you switch traffic from one to another. Allows zero-downtime deployments and easy rollback.

### Architecture:
```
┌──────────────┐         ┌──────────────┐
│   BLUE       │         │   GREEN      │
│  (Current)   │         │   (New)      │
│               │        │              │
│  Healthy ✅  │         │  Building...│
└──────────────┘         └──────────────┘
       ↓                         ↓
    Route 53  →  ALB  ←  Route 53
    (Points to Blue)
    
    After validation:
    Route 53  →  ALB  ←  Route 53
    (Points to Green)
```

### Implementation:

#### Step 1: Create Second Service (Green)

```
AWS Console → ECS
→ Create Service (website-service-green)
→ Same configuration as website-service
→ Attach to separate target group
```

#### Step 2: Deploy New Version to Green

```
CodePipeline deploys to GREEN service
while BLUE service serves traffic
```

#### Step 3: Test Green Environment

```
Access Green service directly:
http://green-alb-dns

Verify functionality before switching
```

#### Step 4: Switch Traffic

Option A - Manual Switch:
```
Route 53 → Update record
Point to GREEN ALB
```

Option B - Automated Switch:
```
CodeDeploy → Traffic Routing Rules
→ Canary (10% → 90%)
→ Linear (10% every 5 minutes)
→ All-at-once
```

#### Step 5: Monitor & Rollback

```
If issues detected:
Route 53 → Switch back to BLUE
Investigate and fix
Deploy to GREEN again
```

### ✅ Verification:
```bash
# Health check both environments
curl http://blue-alb
curl http://green-alb

# Verify same content
diff <(curl http://blue-alb) <(curl http://green-alb)
```

---

# ⚙️ KUBERNETES DEPLOYMENT WITH EKS

---

## 🟢 KUBERNETES OPTION: Deploy to Amazon EKS

### What is EKS?
**Amazon Elastic Kubernetes Service** - Managed Kubernetes service on AWS. For advanced container orchestration beyond ECS.

### When to use EKS vs ECS?

| Feature | ECS | EKS |
|---------|-----|-----|
| **Learning Curve** | Easy | Steep |
| **Kubernetes Skills** | Not required | Required |
| **Multi-cloud | Limited | Yes |
| **Community** | AWS-focused | Large |
| **Costs** | Lower | Higher |
| **Flexibility** | Limited | High |

### Prerequisites:
- Kubernetes knowledge
- kubectl CLI tool
- eksctl CLI tool

---

## Step 1: Install Prerequisites

### Install kubectl:
```bash
# macOS
curl -o kubectl https://amazon-eks.s3.us-west-2.amazonaws.com/1.24.9/2023-01-11/bin/darwin/amd64/kubectl
chmod +x ./kubectl
sudo mv ./kubectl /usr/local/bin

# Linux
curl -o kubectl https://amazon-eks.s3.us-west-2.amazonaws.com/1.24.9/2023-01-11/bin/linux/x86_64/kubectl
chmod +x ./kubectl
sudo mv ./kubectl /usr/local/bin

# Verify
kubectl version --client
```

### Install eksctl:
```bash
# macOS
brew tap weaveworks/tap
brew install weaveworks/tap/eksctl

# Linux
curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
sudo mv /tmp/eksctl /usr/local/bin

# Verify
eksctl version
```

### Configure AWS Credentials:
```bash
aws configure
# Enter: Access Key ID
# Enter: Secret Access Key
# Enter: Default region (ap-south-1)
# Enter: Default format (json)
```

---

## Step 2: Create EKS Cluster

### Option 1: Using eksctl (Recommended)

```bash
# Create cluster
eksctl create cluster \
  --name website-cluster \
  --region ap-south-1 \
  --nodegroup-name website-nodes \
  --node-type t3.medium \
  --nodes 2 \
  --nodes-min 1 \
  --nodes-max 4

# This takes 15-20 minutes
```

### Option 2: Using AWS Console

```
AWS Console → EKS
→ Create Cluster

Configuration:
├─ Name: website-cluster
├─ Version: 1.27 (latest)
├─ Role: Create new EKS service role
├─ VPC: Default VPC
├─ Subnets: All
└─ Security Group: Default
```

### Step 3: Configure kubectl

```bash
# Update kubeconfig
aws eks update-kubeconfig \
  --name website-cluster \
  --region ap-south-1

# Verify connection
kubectl get nodes
```

### Expected Output:
```
NAME                                    STATUS   ROLES    AGE   VERSION
ip-xxx-xxx-xxx-xxx.ap-south-1...       Ready    <none>   2m    v1.27.0
ip-xxx-xxx-xxx-xxx.ap-south-1...       Ready    <none>   2m    v1.27.0
```

---

## Step 4: Create Kubernetes Manifests

### Create k8s.yaml

```yaml
# Namespace
apiVersion: v1
kind: Namespace
metadata:
  name: website

---
# Deployment
apiVersion: apps/v1
kind: Deployment
metadata:
  name: website
  namespace: website
spec:
  replicas: 3
  selector:
    matchLabels:
      app: website
  template:
    metadata:
      labels:
        app: website
    spec:
      containers:
      - name: website
        image: 123456789.dkr.ecr.ap-south-1.amazonaws.com/website:latest
        ports:
        - containerPort: 80
        resources:
          requests:
            cpu: 100m
            memory: 128Mi
          limits:
            cpu: 500m
            memory: 512Mi
        livenessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 5
          periodSeconds: 5

---
# Service
apiVersion: v1
kind: Service
metadata:
  name: website-service
  namespace: website
spec:
  type: LoadBalancer
  selector:
    app: website
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80

---
# Horizontal Pod Autoscaler
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: website-hpa
  namespace: website
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: website
  minReplicas: 2
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 80

---
# ConfigMap for environment variables
apiVersion: v1
kind: ConfigMap
metadata:
  name: website-config
  namespace: website
data:
  APP_ENV: production
  LOG_LEVEL: info

---
# Ingress (if using Ingress Controller)
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: website-ingress
  namespace: website
  annotations:
    kubernetes.io/ingress.class: alb
    alb.ingress.kubernetes.io/scheme: internet-facing
    alb.ingress.kubernetes.io/target-type: ip
spec:
  rules:
  - http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: website-service
            port:
              number: 80
```

---

## Step 5: Deploy to EKS

```bash
# Apply manifests
kubectl apply -f k8s.yaml

# Verify deployment
kubectl get pods -n website
kubectl get svc -n website

# Get Load Balancer DNS
kubectl get svc website-service -n website
# Copy EXTERNAL-IP and open in browser
```

### Expected Output:
```
NAME                       READY   STATUS    RESTARTS   AGE
website-5f8c7b4d5c-abc12   1/1     Running   0          30s
website-5f8c7b4d5c-def45   1/1     Running   0          30s
website-5f8c7b4d5c-ghi78   1/1     Running   0          30s

NAME               TYPE           CLUSTER-IP   EXTERNAL-IP
website-service    LoadBalancer   10.0.1.123   abc123.ap-south-1.elb.amazonaws.com
```

---

## Step 6: Monitor EKS Cluster

```bash
# View cluster info
kubectl cluster-info

# Get nodes
kubectl get nodes

# Get all resources
kubectl get all -n website

# View logs
kubectl logs -n website <pod-name>

# Describe pod (for troubleshooting)
kubectl describe pod -n website <pod-name>

# Watch deployment
kubectl rollout status deployment/website -n website

# View events
kubectl get events -n website
```

---

## Step 7: Update Deployment (Rolling Update)

### Update image:
```bash
# Update image version
kubectl set image deployment/website \
  website=123456789.dkr.ecr.ap-south-1.amazonaws.com/website:v2.0 \
  -n website

# Monitor rollout
kubectl rollout status deployment/website -n website

# View history
kubectl rollout history deployment/website -n website

# Rollback if needed
kubectl rollout undo deployment/website -n website
```

---

## Step 8: Enable CI/CD for EKS

### Integrate with CodePipeline:

1. **Create Deploy Stage** in CodePipeline:
```
Deploy Provider: Amazon ECS (with kubectl)
Or use: AWS Lambda + kubectl
```

2. **Create Lambda Function** to deploy:
```python
import boto3
import subprocess

def lambda_handler(event, context):
    # Get image from CodeBuild
    image_uri = event['image']
    
    # Update deployment
    subprocess.run([
        'kubectl',
        'set', 'image',
        'deployment/website',
        f'website={image_uri}',
        '-n', 'website'
    ])
    
    return {'statusCode': 200}
```

3. **Add to buildspec.yml**:
```yaml
post_build:
  commands:
    - echo "Image built and pushed to ECR"
    - aws lambda invoke --function-name update-k8s-deployment output.json
```

---

## Step 9: Cleanup EKS

### Delete resources in order:

```bash
# Delete namespace (deletes all resources)
kubectl delete namespace website

# Delete cluster (takes 10-15 minutes)
eksctl delete cluster --name website-cluster --region ap-south-1

# Or manually:
AWS Console → EKS → Clusters
→ Select website-cluster
→ Delete
```

---

## EKS vs ECS Decision Matrix

```
Choose ECS if:
✓ You're new to containerization
✓ You want simpler setup
✓ You're AWS-only
✓ You want lower costs

Choose EKS if:
✓ You know Kubernetes
✓ You need multi-cloud
✓ You want flexibility
✓ You have complex requirements
```

---

# 📚 AWS Resources & Documentation

---

## Core Services Documentation

| Service | Documentation | Learning Path |
|---------|---------------|--------------|
| **ECS** | [ECS Docs](https://docs.aws.amazon.com/ecs/) | Getting Started with ECS |
| **ECR** | [ECR Docs](https://docs.aws.amazon.com/ecr/) | Registry Management |
| **CodeBuild** | [CodeBuild Docs](https://docs.aws.amazon.com/codebuild/) | Build Configuration |
| **CodePipeline** | [CodePipeline Docs](https://docs.aws.amazon.com/codepipeline/) | Pipeline Basics |
| **EKS** | [EKS Docs](https://docs.aws.amazon.com/eks/) | Getting Started with EKS |
| **IAM** | [IAM Docs](https://docs.aws.amazon.com/iam/) | Access Management |
| **CloudWatch** | [CloudWatch Docs](https://docs.aws.amazon.com/cloudwatch/) | Monitoring & Logging |
| **Route 53** | [Route 53 Docs](https://docs.aws.amazon.com/route53/) | DNS & Traffic Routing |
| **CloudFront** | [CloudFront Docs](https://docs.aws.amazon.com/cloudfront/) | CDN & Edge Caching |

---

## Useful Learning Resources

### Official AWS Guides
- 🎥 [AWS ECS Tutorial](https://aws.amazon.com/ecs/getting-started/)
- 🎥 [AWS EKS Workshop](https://www.eksworkshop.com/)
- 📖 [AWS DevOps Whitepaper](https://aws.amazon.com/devops/)
- 🔧 [AWS CLI Reference](https://docs.aws.amazon.com/cli/latest/reference/)

### External Resources
- 🐳 [Docker Official Docs](https://docs.docker.com/)
- ☸️ [Kubernetes Official Docs](https://kubernetes.io/docs/)
- 📚 [Kubernetes Best Practices](https://kubernetes.io/docs/concepts/configuration/overview/)
- 💡 [AWS Architecture Center](https://aws.amazon.com/architecture/)

### Community & Support
- 💬 [AWS Forums](https://forums.aws.amazon.com/)
- 🆘 [Stack Overflow - AWS Tag](https://stackoverflow.com/questions/tagged/amazon-aws)
- 🆘 [Stack Overflow - ECS Tag](https://stackoverflow.com/questions/tagged/amazon-ecs)
- 🆘 [Stack Overflow - EKS Tag](https://stackoverflow.com/questions/tagged/amazon-eks)
- 👥 [AWS Communities](https://aws.amazon.com/developer/community/)

---

## AWS Certification Paths

### Associate Level (Start Here)
- ☁️ [AWS Solutions Architect Associate](https://aws.amazon.com/certification/certified-solutions-architect-associate/)
  - Topics: ECS, ECR, Lambda, RDS, VPC
  - Time: 2-3 months study
  - Cost: $150

- 🔨 [AWS Developer Associate](https://aws.amazon.com/certification/certified-developer-associate/)
  - Topics: CodeBuild, CodePipeline, Lambda
  - Time: 2-3 months study
  - Cost: $150

### Professional Level (Advanced)
- 🚀 [AWS DevOps Engineer Professional](https://aws.amazon.com/certification/certified-devops-engineer-professional/)
  - Topics: CI/CD, ECS, EKS, CodePipeline
  - Prerequisite: Associate certification
  - Cost: $300

- 💼 [AWS Solutions Architect Professional](https://aws.amazon.com/certification/certified-solutions-architect-professional/)
  - Topics: Complex architecture, scaling
  - Prerequisite: Associate certification
  - Cost: $300

---

## Practice Resources

- 📝 [A Cloud Guru](https://acloudguru.com/)
- 📝 [Linux Academy](https://linuxacademy.com/)
- 📝 [Coursera - AWS Courses](https://www.coursera.org/search?query=aws)
- 📝 [Pluralsight - AWS Path](https://www.pluralsight.com/paths/aws-architect)
- 🧪 [AWS Skill Builder](https://skillbuilder.aws/)

---

# ⚡ Best Practices

---

## 🔒 Security Best Practices

### 1. IAM Security
```yaml
✓ Use least privilege principle
  - Grant only needed permissions
  - Avoid using root account
  
✓ Use IAM roles instead of access keys
  - Attach roles to EC2/ECS
  - No credentials in code
  
✓ Enable MFA for sensitive operations
  - Multi-factor authentication
  - AWS Console access
  
✓ Rotate credentials regularly
  - Access keys every 90 days
  - Use temporary credentials
```

### 2. Container Security
```yaml
✓ Scan images for vulnerabilities
  - Enable ECR image scanning
  - Use AWS Inspector
  
✓ Use minimal base images
  - alpine:latest (5MB)
  - vs ubuntu:latest (77MB)
  
✓ Don't run as root
  - Create application user
  - Drop unnecessary capabilities
  
✓ Sign images
  - Content Trust
  - Image provenance
```

### 3. Network Security
```yaml
✓ Use VPC security groups
  - Restrict inbound/outbound
  - Least privilege rules
  
✓ Enable VPC Flow Logs
  - Monitor traffic
  - Troubleshooting
  
✓ Use AWS WAF
  - Protect from attacks
  - Rate limiting
  
✓ Enable VPC endpoints
  - Private connectivity
  - No internet exposure
```

### 4. Data Security
```yaml
✓ Encrypt in transit
  - Use HTTPS/TLS
  - AWS Certificate Manager
  
✓ Encrypt at rest
  - EBS encryption
  - RDS encryption
  - S3 encryption
  
✓ Manage secrets properly
  - AWS Secrets Manager
  - No hardcoded credentials
```

---

## 📊 Performance Best Practices

### 1. Container Optimization
```yaml
✓ Right-size resources
  - CPU: 256m-4096m
  - Memory: 512MB-30GB
  - Based on load testing
  
✓ Use health checks
  - Liveness probes
  - Readiness probes
  - Startup probes (K8s)
  
✓ Implement caching
  - CloudFront
  - ElastiCache
  - Application level
```

### 2. Load Distribution
```yaml
✓ Use load balancers
  - Application Load Balancer (ALB)
  - Network Load Balancer (NLB)
  
✓ Enable auto-scaling
  - Target tracking
  - Step scaling
  - Scheduled scaling
  
✓ Use CloudFront for static assets
  - Global edge locations
  - Compression
  - Caching
```

### 3. Database Optimization
```yaml
✓ Use read replicas
  - Distribute read load
  - Multi-region failover
  
✓ Implement connection pooling
  - RDS Proxy
  - Connection reuse
  
✓ Use appropriate instance types
  - db.t3.micro for dev
  - db.r5.large for prod
```

---

## 💰 Cost Optimization

### 1. Compute Costs
```yaml
✓ Use Fargate Spot
  - Up to 70% savings
  - For non-critical workloads
  
✓ Right-size instances
  - Monitor utilization
  - Scale down unused resources
  
✓ Use Reserved Instances
  - 30-40% savings
  - For predictable workloads
```

### 2. Storage Costs
```yaml
✓ Use S3 lifecycle policies
  - Move to Glacier after 30 days
  - Delete after 90 days
  
✓ Enable S3 Intelligent-Tiering
  - Automatic cost optimization
  
✓ Use EBS snapshots
  - Share across regions
  - Delete unused snapshots
```

### 3. Data Transfer Costs
```yaml
✓ Use CloudFront
  - Reduce data egress
  - Cheaper pricing
  
✓ Use VPC endpoints
  - Free data transfer
  - Same AZ
  
✓ Consolidate requests
  - Batch API calls
  - Reduce round-trips
```

### 4. Monitoring & Budgets
```yaml
✓ Set up AWS Budgets
  - Budget alerts
  - Anomaly detection
  
✓ Use Cost Explorer
  - Analyze spending
  - Identify savings
  
✓ Enable detailed billing
  - Per-service breakdown
  - Resource-level tagging
```

---

## 🛡️ Reliability Best Practices

### 1. High Availability
```yaml
✓ Multi-AZ deployment
  - Distribute across availability zones
  - Load balancer health checks
  
✓ Database replication
  - Read replicas
  - Automated backups
  
✓ Auto-scaling policies
  - Scale out under load
  - Scale in during low usage
```

### 2. Disaster Recovery
```yaml
✓ Regular backups
  - Daily RDS backups
  - Cross-region replication
  - Test restore procedures
  
✓ Chaos engineering
  - Inject failures
  - Test resilience
  - Improve reliability
  
✓ Disaster recovery plan (DRP)
  - RTO: Recovery Time Objective
  - RPO: Recovery Point Objective
  - Regular drills
```

### 3. Monitoring & Alerting
```yaml
✓ CloudWatch monitoring
  - CPU utilization
  - Memory usage
  - Error rates
  
✓ Application insights
  - AWS X-Ray tracing
  - Application Performance Monitoring
  
✓ Alert thresholds
  - CPU > 80% for 5 minutes
  - Error rate > 5%
  - Response time > 2 seconds
```

---

## 📈 Scalability Best Practices

### 1. Application Design
```yaml
✓ Stateless architecture
  - Load balance traffic
  - Easy scaling
  - Session in database
  
✓ Asynchronous processing
  - SQS for jobs
  - SNS for notifications
  - Lambda for events
  
✓ Microservices
  - Independent scaling
  - Failure isolation
  - Technology flexibility
```

### 2. Database Scaling
```yaml
✓ Read scaling
  - Read replicas
  - ElastiCache
  - Read-only endpoints
  
✓ Write scaling
  - Sharding
  - Partitioning
  - DynamoDB for scale
  
✓ Caching strategy
  - Application caching
  - Database query caching
  - CDN caching
```

---

## 🔄 Operational Excellence

### 1. Infrastructure as Code
```yaml
✓ Use CloudFormation/Terraform
  - Version control
  - Reproducible
  - Infrastructure documentation
  
✓ GitOps workflow
  - Git as source of truth
  - Automated deployments
  - Audit trail
```

### 2. Logging & Monitoring
```yaml
✓ Centralized logging
  - CloudWatch Logs
  - Log aggregation
  - Searchable logs
  
✓ Structured logging
  - JSON format
  - Consistent fields
  - Easy parsing
```

### 3. Change Management
```yaml
✓ Blue-Green deployments
  - Zero-downtime updates
  - Easy rollback
  
✓ Canary releases
  - Gradual rollout
  - Monitor metrics
  - Auto-rollback if errors
```

---

# 🧠 Quick Decision Trees

---

## Container Orchestration Decision

```
Do you know Kubernetes?
├─ YES → Use EKS
│   └─ Multi-cloud needed?
│       ├─ YES → EKS + Anywhere
│       └─ NO → EKS on AWS
│
└─ NO → Use ECS
    └─ Advanced features needed?
        ├─ YES → ECS + Advanced
        └─ NO → ECS + Simple
```

---

## Deployment Strategy Decision

```
Zero-downtime deployment needed?
├─ YES → Blue-Green or Canary
│   └─ Gradual rollout?
│       ├─ YES → Canary
│       └─ NO → Blue-Green
│
└─ NO → Rolling deployment
    └─ Simple one-at-a-time
```

---

## Scaling Strategy Decision

```
What needs scaling?
├─ Application → ECS/EKS Auto-Scaling
├─ Database → RDS Read Replicas
├─ Static Content → CloudFront CDN
├─ Load → Application Load Balancer
└─ Compute → Lambda (if applicable)
```

---

# 🎓 Learning Path Recommendation

```
Week 1: Basics
├─ Docker fundamentals
├─ AWS Account setup
└─ CLI basics

Week 2-3: Core Services
├─ ECR (image registry)
├─ ECS (simple container service)
└─ EC2 (compute instances)

Week 4: CI/CD
├─ CodeBuild (build automation)
├─ CodePipeline (orchestration)
└─ GitHub integration

Week 5-6: Advanced
├─ Load Balancers
├─ Auto-Scaling
├─ CloudWatch Monitoring
└─ Route 53 DNS

Week 7-8: Production
├─ Security best practices
├─ Cost optimization
├─ Disaster recovery
└─ Multi-region setup

Week 9-10: Expert
├─ Kubernetes (EKS)
├─ Infrastructure as Code
├─ Advanced monitoring
└─ Certification exam prep
```

---

# 🎉 Congratulations!

You've learned:
- ✅ Docker containerization
- ✅ AWS container services (ECS/EKS)
- ✅ CI/CD pipeline automation
- ✅ Infrastructure best practices
- ✅ Security, scalability, reliability
- ✅ Production deployment patterns

---

# 📞 Getting Help

### If you're stuck:
1. ✅ Check AWS documentation
2. ✅ Search Stack Overflow
3. ✅ Review CloudWatch logs
4. ✅ Check AWS forum
5. ✅ Contact AWS Support

---

# 📝 Final Checklist

Before deploying to production:

- [ ] Security scan images
- [ ] Enable CloudWatch monitoring
- [ ] Setup CloudTrail logging
- [ ] Implement backup strategy
- [ ] Document runbooks
- [ ] Setup alerting thresholds
- [ ] Test disaster recovery
- [ ] Plan scaling strategy
- [ ] Configure auto-scaling
- [ ] Setup CDN/caching
- [ ] Enable encryption
- [ ] Restrict IAM permissions
- [ ] Setup budget alerts
- [ ] Test failover
- [ ] Document architecture

---

**🚀 Happy DevOps Engineering!**

---

### 📌 Document Information
**Version:** 2.0 (Complete)  
**Last Updated:** January 2024  
**Status:** ✅ Production Ready  
**Content:** Complete Guide with ECS, EKS, and Best Practices

### Next Steps After This Guide:
1. ✅ Deploy project to production
2. 📚 Learn Kubernetes deeply (if using EKS)
3. 🏆 Pursue AWS certification
4. 🔐 Implement advanced security
5. 🌍 Setup multi-region deployment
6. 📊 Implement advanced monitoring
7. 💼 Build more complex applications

---

**Thank you for following this comprehensive guide!**