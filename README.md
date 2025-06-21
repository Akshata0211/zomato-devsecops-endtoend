# Zomato Clone: End-to-End DevSecOps Pipeline on AWS with Monitoring

## Overview

This project demonstrates a complete DevSecOps lifecycle by deploying a Zomato Clone using modern DevOps tools, infrastructure automation, security scanning, continuous deployment, and monitoring on AWS. The tech stack includes:

Terraform – Infrastructure as Code for provisioning AWS resources.

GitHub – Source code and pipeline scripts repository.

Jenkins – CI/CD automation.

SonarQube – Code quality analysis.

Trivy – Security vulnerability scanning.

NPM – Dependency management and build tool for Node.js.

Docker – Image containerization.

AWS ECR & EKS – Image storage and orchestration with Kubernetes.

ArgoCD – GitOps-based deployment.

Prometheus & Grafana – Cluster monitoring.

## Prerequisites

- AWS CLI configured with IAM user (access and secret keys).
- Terraform installed.

## Step 1: Infrastructure Setup with Terraform

### 1.1 Clone the Repository
```bash
git clone https://github.com/Akshata0211/zomato-devsecops-endtoend.git
cd zomato-devsecops-endtoend
code .
```

### 1.2 Launch EC2 for Jenkins
```bash
cd terraform_code/ec2_server
aws configure
terraform init
terraform plan
terraform apply
```

This step provisions an EC2 instance and installs Jenkins, Docker, SonarQube, and necessary tools.

After the Terraform apply completes, you will receive the Jenkins and SonarQube login URLs and credentials as output. Use these to access Jenkins and SonarQube for further configuration.

### 1.3 Launch EKS Cluster after Jenkins Build Pipeline

```bash
cd terraform_code/eks_code
aws configure
terraform init
terraform plan
terraform apply
```
Creates an EKS cluster for container orchestration.

### Step 2: SonarQube Configuration

- Access SonarQube on `http://<jenkins-ec2-ip>:9000`.
- Default credentials: `admin/admin`
- Create a token from **Administration → Security → Users → Tokens**.
- Add a webhook in SonarQube:
- URL: `http://<jenkins-url>:8080/sonarqube-webhook/`

### Step 3: Jenkins Configuration

3.1 Install Plugins

Go to `Manage Jenkins → Plugins` and install:
- SonarQube Scanner
- NodeJS
- Docker
- Prometheus Metrics
- Pipeline Stage View
- Eclipse Temurin (JDK)

### 3.2 Global Tool Configuration

`Manage Jenkins → Global Tool Configuration`

- JDK 17: Name: `jdk17`, Install from: `adoptium.net`
- SonarQube Scanner: Name: `sonar-scanner`, Latest version
- NodeJS: Name: `nodeJS`, Install from `nodejs.org`
- Docker: Name: `docker`

### 3.3 Add Credentials

`Manage Jenkins → Credentials → Global`

- **SonarQube token**: for code analysis.
- **AWS keys**: for pushing Docker images to ECR.

### 3.4 Configure SonarQube in Jenkins

`Manage Jenkins → Configure System`

- Add SonarQube Server URL and token.

## Step 4: Build Pipeline in Jenkins

### 4.1 Create Pipeline Job

- Go to Jenkins → New Item → Enter name: `build` → Select: `Pipeline`
- Check: `"GitHub hook trigger for GITScm polling"`

### 4.2 Set SCM

- SCM: Git
-Repo URL: `https://github.com/Akshata0211/zomato-devsecops-endtoend.git`
- Branch: `*/main`
- Script Path: `pipeline_script/build_pipeline`

### 4.3 GitHub Webhook Setup

- GitHub → Repo → Settings → Webhooks → Add webhook
- Payload URL: `http://<jenkins-url>/github-webhook/`
- Content Type: `application/json`
- Event: Push event only

### 4.4 Pipeline Stages

- Clone source code
- Run SonarQube code quality scan
- Install Node dependencies
- Perform Trivy vulnerability scan
- Build Docker image
- Push image to AWS ECR
- Clean up old local Docker images

## Step 5: Deployment Pipeline with ArgoCD

### 5.1 ArgoCD Setup

Access Jenkins EC2 instance and run access.sh from the repo root:
```bash
chmod a+x access.sh
./access.sh
```
Output will show URLs and credentials for:
- ArgoCD
- Prometheus
- Grafana

### 5.2 Update Image URL

In `k8s_files/deployment.yaml`, update the image path with your ECR URL and image tag.

### 5.3 Create ArgoCD Application

- Open ArgoCD UI and login.
- Create Application:

    - Source: Git repo
    - Path: `k8s_files/`
    - Destination: Kubernetes Cluster (namespace: default)
    - Sync Policy: Automatic + Self-Heal

### 5.4 Monitor Deployment

- Access Zomato app at `<service-url>:3000` (use kubectl get svc to fetch URL)
- Use ArgoCD’s graphical view to monitor app deployment

## Step 6: Monitoring with Prometheus & Grafana

### 6.1 Access Dashboards

- Grafana URL from `./access.sh` output
- Import dashboard ID `1860` in Grafana
- Set refresh interval to 5s and time range to last 5 min

## Step 7: Cleanup

### 7.1 Jenkins Cleanup Pipeline

- Jenkins → New Item → Name: `cleanup` → Type: Pipeline
- Script Path: `pipeline_script/cleanup_pipeline`

### 7.2 Terraform Destroy

Run `terraform destroy` inside both `ec2_server/` and `eks_code/` folders to remove all AWS resources.

### Useful Kubernetes Commands
```bash
kubectl get svc
kubectl get deploy
kubectl get pods
kubectl get svc -n argocd
kubectl get svc -n prometheus
```

### Summary

This project demonstrates a full CI/CD pipeline with security and monitoring:

- Code → Build → Quality Scan → Security Scan → Dockerize → Deploy → Monitor

- Managed through Jenkins, secured with SonarQube and Trivy, deployed via ArgoCD, and monitored with Prometheus & Grafana.
