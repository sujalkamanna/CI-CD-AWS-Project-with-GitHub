# CI/CD AWS DevOps Project with GitHub - Complete Guide

A complete automated deployment pipeline for a security website using Docker, AWS services, and GitHub.

---

## 📋 Table of Contents

1. [Project Overview](#project-overview)
2. [Project Structure](#project-structure)
3. [Prerequisites](#prerequisites)
4. [Step-by-Step Setup](#step-by-step-setup)
5. [Testing & Verification](#testing--verification)
6. [Cleanup](#cleanup)
- 📖 [**Full Workflow Guide & Instructions**](./WORKFLOW.md) - Complete step-by-step guide for CI/CD setup, Docker deployment, ECS/EKS options, and AWS best practices
---

## 🎯 Project Overview

This project demonstrates a **complete CI/CD pipeline** that automatically builds, pushes, and deploys a website whenever code is pushed to GitHub.

### Architecture Flow:
```
GitHub Repository
        ↓
   Code Change
        ↓
  CodePipeline Trigger
        ↓
  CodeBuild (Build & Test)
        ↓
  Push to ECR (Image Registry)
        ↓
  Deploy to ECS Fargate
        ↓
  Live Website
```

---

## 📁 Project Structure

```
CI-CD-AWS-Project-with-GitHub/
│
├── 📄 HTML Files (Website Content)
│   ├── index.html              # Home page
│   ├── about.html              # About page
│   ├── service.html            # Services page
│   ├── guard.html              # Guards/Team page
│   └── contact.html            # Contact page
│
├── 🐳 Docker Configuration
│   ├── Dockerfile              # Docker image definition
│
├── ☸️ Kubernetes Configuration
│   └── k8s.yaml               # Kubernetes deployment (optional)
│
├── 🔨 CI/CD Configuration
│   ├── buildspec.yml          # AWS CodeBuild configuration
│   └── cloudbuild.yaml        # Google Cloud Build (optional)
│
├── 📁 Static Assets
│   ├── css/                   # Stylesheets
│   ├── js/                    # JavaScript files
│   ├── images/                # Website images
│   └── fonts/                 # Custom fonts
│
└── 📝 Documentation
    └── README.md              # This file
```

---

## ✅ Prerequisites

Before starting, ensure you have:

- **AWS Account** (with appropriate permissions)
- **GitHub Account** (with repository access)
- **IAM User** (with programmatic access)
- **Basic Docker knowledge**
- **Command line/Terminal access**

### AWS Services Required:
- EC2 (for initial testing)
- ECR (Elastic Container Registry)
- ECS (Elastic Container Service)
- CodeBuild
- CodePipeline
- IAM Roles & Policies

---

## 🚀 Step-by-Step Setup

### **Step 1: Launch EC2 Instance**

1. Go to **AWS Console** → **EC2** → **Launch Instance**
2. Select **Amazon Linux 2**
3. Instance Type: **t2.micro** (free tier)
4. Configure security group to allow ports: 22, 80, 443

#### Connect to EC2 and Install Docker:

```bash
# Update system packages
sudo yum update -y

# Install Docker
sudo yum install docker -y

# Start Docker service
sudo service docker start

# Verify Docker installation
sudo service docker status

# View available Docker images
sudo docker images

# View running containers
sudo docker ps -a
```

✅ **Status Check:** Docker service should be running

---

### **Step 2: Clone GitHub Repository**

```bash
# Install Git
sudo yum install git -y

# Clone your repository
git clone https://github.com/sujalkamanna/CI-CD-AWS-Project-with-GitHub

# Navigate to project directory
cd website

# View project structure
ls -la
```

✅ **Status Check:** All project files should be downloaded

---

### **Step 3: Docker Setup & Local Testing**

#### 3.1 View Dockerfile:

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

**What it does:**
- Uses nginx as base image
- Copies website files to nginx directory
- Exposes port 80 for web traffic
- Starts nginx server

#### 3.2 Build Docker Image:

```bash
# Build image with tag 'website:latest'
sudo docker build -t website:latest .

# Verify image was created
sudo docker images
```

**Expected Output:**
```
REPOSITORY    TAG       IMAGE ID      CREATED       SIZE
website       latest    abc123def456  2 minutes ago  142MB
```

#### 3.3 Run Container Locally:

```bash
# Run container in background
# -d = detached mode
# -p = port mapping (host:container)
sudo docker run -d -p 80:80 website:latest

# Verify container is running
sudo docker ps -a
```

#### 3.4 Access Website:

```bash
# Get EC2 public IP
curl http://169.254.169.254/latest/meta-data/public-ipv4

# Open in browser: http://<your-ec2-public-ip>
```

✅ **Status Check:** Website should be accessible from browser

---

### **Step 4: Push Image to AWS ECR**

#### 4.1 Create ECR Repository:

1. Go to **AWS Console** → **ECR** → **Repositories**
2. Click **Create Repository**
3. Repository Name: `website`
4. Click **Create**

#### 4.2 Attach IAM Role to EC2:

The EC2 instance needs permission to push to ECR.

1. Go to **IAM** → **Roles**
2. Create new role → Select **EC2**
3. Attach policy: **AmazonEC2ContainerRegistryFullAccess**
4. Attach the role to your EC2 instance

#### 4.3 Login to ECR:

```bash
# Replace 'ap-south-1' with your region
# Replace '<account-id>' with your AWS account ID

aws ecr get-login-password --region ap-south-1 | \
  sudo docker login --username AWS --password-stdin \
  <account-id>.dkr.ecr.ap-south-1.amazonaws.com
```

**Expected Output:**
```
Login Succeeded
```

#### 4.4 Tag & Push Image:

```bash
# Tag image with ECR repository URL
sudo docker tag website:latest \
  <account-id>.dkr.ecr.ap-south-1.amazonaws.com/website:latest

# Push image to ECR
sudo docker push <account-id>.dkr.ecr.ap-south-1.amazonaws.com/website:latest
```

**Progress Output:**
```
Pushing image... 100%
Image pushed successfully
```

✅ **Status Check:** Image should appear in ECR console

---

### **Step 5: Create ECS Cluster & Services**

#### 5.1 Create Cluster:

1. Go to **AWS Console** → **ECS** → **Clusters**
2. Click **Create Cluster**
3. **Cluster Configuration:**
   - Name: `websitecluster`
   - Infrastructure: **AWS Fargate**
   - VPC: Default VPC
4. Click **Create**

#### 5.2 Create Task Definition:

1. Go to **ECS** → **Task Definitions**
2. Click **Create new task definition**
3. **Configuration:**
   - Family: `website-task`
   - Launch type: **Fargate**
   - CPU: 256
   - Memory: 512 MB
   - Container name: `website`
   - Image: `<account-id>.dkr.ecr.ap-south-1.amazonaws.com/website:latest`
   - Port mapping: 80:80
4. Click **Create**

#### 5.3 Create Service:

1. Go to **ECS** → **Clusters** → **websitecluster**
2. Click **Create Service**
3. **Configuration:**
   - Launch type: **Fargate**
   - Task Definition: `website-task`
   - Service name: `website-service`
   - Desired tasks: 2
   - Load balancer: **Create new** (Application Load Balancer)
4. Click **Create**

✅ **Status Check:** Service should show tasks as running

---

### **Step 6: Setup AWS CodeBuild**

CodeBuild automatically builds Docker images from your code.

#### 6.1 Create Build Project:

1. Go to **AWS Console** → **CodeBuild** → **Create Project**
2. **Configuration:**
   - Project name: `website-build`
   - Source: **GitHub**
   - Repository: Select your website repository
   - Buildspec: `buildspec.yml`
   - OS: Amazon Linux 2
   - Runtime: Docker
3. Click **Create**

#### 6.2 Create buildspec.yml:

Add this file to your GitHub repository root:

```yaml
version: 0.2

# Environment variables
env:
  variables:
    AWS_DEFAULT_REGION: eu-west-1a
    AWS_ACCOUNT_ID: <your-account-id>
    IMAGE_REPO_NAME: website
    IMAGE_TAG: latest

phases:
  # Phase 1: Pre-build (login to ECR)
  pre_build:
    commands:
      - echo "Logging in to Amazon ECR..."
      - aws ecr get-login-password --region $AWS_DEFAULT_REGION | docker login --username AWS --password-stdin $AWS_ACCOUNT_ID.dkr.ecr.$AWS_DEFAULT_REGION.amazonaws.com
      - REPOSITORY_URI=$AWS_ACCOUNT_ID.dkr.ecr.$AWS_DEFAULT_REGION.amazonaws.com/$IMAGE_REPO_NAME
      - COMMIT_HASH=$(echo $CODEBUILD_RESOLVED_SOURCE_VERSION | cut -c 1-7)
      - IMAGE_TAG=${COMMIT_HASH:=latest}

  # Phase 2: Build (build Docker image)
  build:
    commands:
      - echo "Building the Docker image on `date`"
      - docker build -t $REPOSITORY_URI:$IMAGE_TAG .
      - docker tag $REPOSITORY_URI:$IMAGE_TAG $REPOSITORY_URI:latest

  # Phase 3: Post-build (push to ECR)
  post_build:
    commands:
      - echo "Pushing the Docker image to ECR on `date`"
      - docker push $REPOSITORY_URI:$IMAGE_TAG
      - docker push $REPOSITORY_URI:latest
      - echo "Writing image definitions file..."
      - printf '[{"name":"website","imageUri":"%s"}]' $REPOSITORY_URI:$IMAGE_TAG > imagedefinitions.json

# Output artifacts
artifacts:
  files:
    - imagedefinitions.json
```

#### 6.3 Update IAM Role for CodeBuild:

1. Go to **IAM** → **Roles** → Find CodeBuild role
2. Attach policies:
   - `AmazonEC2ContainerRegistryFullAccess`
   - `AmazonEC2ContainerServiceFullAccess`
   - `CloudWatchLogsFullAccess`

#### 6.4 Test Build:

```
Go to CodeBuild → Select project → Start Build
```

**First run may fail due to permissions** - this is normal. Fix IAM role and run again.

✅ **Status Check:** Build should succeed and push image to ECR

---

### **Step 7: Create CI/CD Pipeline**

CodePipeline orchestrates the entire workflow automatically.

#### 7.1 Create Pipeline:

1. Go to **AWS Console** → **CodePipeline** → **Create Pipeline**
2. **Pipeline Configuration:**
   - Name: `website-pipeline`
   - Service role: Create new

#### 7.2 Configure Pipeline Stages:

**Stage 1: Source**
- Source provider: **GitHub**
- Repository: Your website repository
- Branch: `main`

**Stage 2: Build**
- Build provider: **CodeBuild**
- Project: `website-build`

**Stage 3: Deploy**
- Deploy provider: **Amazon ECS**
- Cluster: `websitecluster`
- Service: `website-service`
- Image definitions file: `imagedefinitions.json`

3. Click **Create Pipeline**

#### Pipeline Flow:
```
┌──────────────┐     ┌──────────────┐     ┌──────────────┐
│   GitHub     │────→│  CodeBuild   │────→│     ECS      │
│  Repository  │     │   (Docker)   │     │  (Fargate)   │
└──────────────┘     └──────────────┘     └──────────────┘
      Source              Build              Deploy
```

✅ **Status Check:** Pipeline should show "Succeeded"

---

## 🧪 Testing & Verification

### Test Automatic Deployment:

1. **Modify a file** in your GitHub repository (e.g., `index.html`)
2. **Commit and push** changes:
   ```bash
   git add .
   git commit -m "Update website"
   git push origin main
   ```

3. **Watch the pipeline:**
   - Go to **CodePipeline** → Your pipeline
   - See stages executing automatically
   - Image built → Pushed to ECR → Deployed to ECS

4. **Access updated website:**
   - Get Load Balancer DNS from **ECS** → **Services**
   - Open in browser: `http://<load-balancer-dns>`

✅ **Verification:** Website should reflect your changes within 2-3 minutes

---

## 🧹 Cleanup

When done, delete resources to avoid charges:

```bash
# 1. Delete ECS Service
AWS Console → ECS → Services → Select service → Delete

# 2. Delete ECS Cluster
AWS Console → ECS → Clusters → Select cluster → Delete

# 3. Delete Task Definition
AWS Console → ECS → Task Definitions → Deregister

# 4. Delete Load Balancer
AWS Console → EC2 → Load Balancers → Select → Delete

# 5. Delete Target Groups
AWS Console → EC2 → Target Groups → Select → Delete

# 6. Delete ECR Repository
AWS Console → ECR → Repositories → Select → Delete

# 7. Delete CodePipeline
AWS Console → CodePipeline → Select → Delete

# 8. Delete CodeBuild Project
AWS Console → CodeBuild → Select → Delete

# 9. Terminate EC2 Instance
AWS Console → EC2 → Instances → Select → Terminate
```

⚠️ **Important:** Always verify services are stopped before deleting!

---

## 📊 Architecture Summary

```
┌─────────────────────────────────────────────────────────────┐
│                      CI/CD Pipeline                         │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  GitHub              CodeBuild           ECS Fargate        │
│  ┌──────────┐       ┌──────────┐       ┌──────────┐         │
│  │ Source   │──────→│  Build   │──────→│ Deploy   │         │
│  │ Code     │       │ Docker   │       │ Website  │         │
│  └──────────┘       └──────────┘       └──────────┘         │
│       │                   │                   │             │
│       └───────────────────┴───────────────────┘             │
│                        ECR                                  │
│                  (Docker Registry)                          │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## 🎓 Key Concepts

### **Docker**
- Containerizes the application
- Ensures consistency across environments
- Reduces deployment issues

### **ECR (Elastic Container Registry)**
- AWS managed Docker registry
- Stores and versions your images
- Integrated with other AWS services

### **ECS (Elastic Container Service)**
- Runs Docker containers at scale
- Manages container orchestration
- Supports Fargate (serverless)

### **CodeBuild**
- Compiles code and creates Docker images
- Runs automated tests
- Pushes images to registries

### **CodePipeline**
- Orchestrates entire workflow
- Triggers automatically on code changes
- Manages workflow from source to production

---

## ⚡ Quick Troubleshooting

| Issue | Solution |
|-------|----------|
| Build fails | Check IAM permissions (ECR/ECS access) |
| Pipeline stuck | Check CodeBuild logs in CloudWatch |
| ECS tasks not running | Verify task definition image URI |
| Website not updating | Check imagedefinitions.json format |
| Cannot push to ECR | Verify AWS CLI credentials configured |

---

## 📚 Additional Resources

- [AWS CodePipeline Documentation](https://docs.aws.amazon.com/codepipeline/)
- [Docker Documentation](https://docs.docker.com/)
- [AWS ECS Fargate Guide](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/what-is-fargate.html)

---

## ✨ Benefits of This Setup

✅ **Fully Automated** - No manual deployments  
✅ **Scalable** - Handles traffic spikes automatically  
✅ **Reliable** - High availability with load balancing  
✅ **Cost-Effective** - Fargate pricing based on usage  
✅ **Secure** - Private ECR with IAM controls  
✅ **Version Control** - Track all changes in GitHub  

---

**Last Updated:** 2026 | **Status:** Production Ready ✅