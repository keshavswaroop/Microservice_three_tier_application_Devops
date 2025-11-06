# 🧩 Project Overview

The project consists of three main components:
| Tier | Technology | Description |
| ------------ | --------------------- | ---------------------------------------- |
| **Frontend** | React.js | A user interface served via Nginx |
| **Backend** | Spring Boot (Java 17) | A REST API that connects to the database |
| **Database** | MySQL 8.0 | Stores user and application data |

These components are containerized using Docker, orchestrated using Kubernetes, and deployed to AWS EKS using a CI/CD pipeline built with GitHub Actions.

# 🏗️ Architecture

```bash
+--------------------------------------+
| GitHub Actions CI/CD |
| (Build → Push → Deploy to EKS) |
+--------------------------------------+
|
▼
+--------------------------------------+
| AWS EKS Cluster |
|--------------------------------------|
| Namespace: mtta |
| ├── Frontend Deployment + Service |
| ├── Backend Deployment + Service |
| ├── MySQL Deployment + Service |
+--------------------------------------+
|
▼
+--------------------------------------+
| AWS Infrastructure (VPC) |
| Created via Terraform (VPC + EKS) |
+--------------------------------------+
```

# 🧱 Tech Stack

```bash
| Category | Tool / Service |
| ---------------------- | --------------------- |
| **Infrastructure** | AWS VPC, EKS, EC2 |
| **IaC (Provisioning)** | Terraform |
| **Containers** | Docker |
| **Orchestration** | Kubernetes |
| **CI/CD** | GitHub Actions |
| **Backend** | Spring Boot (Java 17) |
| **Frontend** | React.js |
| **Database** | MySQL 8.0 |
```

# 📦 Folder Structure

```bash
Microservice_3_tier_application/
├── backend/ # Spring Boot REST API
│ ├── Dockerfile
│ ├── src/
│ └── target/backend.jar
│
├── frontend/ # React.js app
│ ├── Dockerfile
│ ├── src/
│ └── build/
│
├── k8s/ # Kubernetes manifests
│ ├── backend-deployment.yml
│ ├── backend-service.yml
│ ├── frontend-deployment.yml
│ ├── frontend-service.yml
│ ├── mysql-deployment.yml
│ ├── mysql-service.yml
│ ├── configmap.yml
│ ├── secrets.yml
│ └── namespace.yml
│
├── terraform/ # Terraform modules
│ ├── main.tf
│ ├── variables.tf
│ ├── outputs.tf
│ ├── vpc/
│ │ └── infra/
│ └── eks/
│ └── infra/
│
├── .github/
│ └── workflows/
│ └── cicd.yml # GitHub Actions CI/CD pipeline
│
└── README.md
```

# 🧰 Prerequisites

Before setting up the project, ensure you have:

- An AWS account
- Docker installed locally
- Terraform installed
- kubectl and AWS CLI installed
- GitHub repository with secrets configured:
  - DOCKERHUB_USERNAME
  - DOCKERHUB_TOKEN
  - AWS_ACCESS_KEY_ID
  - AWS_SECRET_ACCESS_KEY

# 🌍 Infrastructure Setup (Terraform)

Terraform is used to automate creation of:

- VPC
- Subnets
- Internet Gateway
- EKS Cluster + Node Group

Commands to Run:

```bash
cd terraform
terraform init
terraform plan
terraform apply -auto-approve
```

Once done, update your kubeconfig to connect to the new cluster:

```bash
aws eks update-kubeconfig --region <your-region> --name <your-cluster-name>
kubectl get nodes
```

# 🐳 Docker

Build and push:

```bash
docker build -t <username>/mtta_backend .
docker build -t <username>/mtta_frontend .
docker push <username>/mtta_backend
docker push <username>/mtta_frontend
```

# ☸️ Kubernetes Setup

Deploy the application:

```bash
kubectl apply -f k8s/namespace.yml
kubectl apply -f k8s/secrets.yml
kubectl apply -f k8s/configmap.yml
kubectl apply -f k8s/mysql-deployment.yml
kubectl apply -f k8s/mysql-service.yml
kubectl apply -f k8s/backend-deployment.yml
kubectl apply -f k8s/backend-service.yml
kubectl apply -f k8s/frontend-deployment.yml
kubectl apply -f k8s/frontend-service.yml
```

Verify deployments:

```bash
kubectl get all -n mtta
```

If using a LoadBalancer service (for frontend):

```bash
kubectl get svc -n mtta
```

# ⚙️ GitHub Actions CI/CD Workflow

Located in .github/workflows/cicd.yml

Workflow Summary:

1. Checkout code from GitHub
2. Build Docker images for backend & frontend
3. Push images to DockerHub
4. Configure AWS credentials
5. Update kubeconfig for EKS cluster
6. Deploy updated manifests to EKS
   The pipeline is triggered: manually via “Run workflow” in GitHub Actions

# ✅ Verification

Once workflow completes:

Check GitHub Actions logs → Deployment successful message

Run:

```bash
kubectl get svc -n mtta
```

Copy the LoadBalancer EXTERNAL-IP of frontend-service

Open it in your browser → App should be live 🎉
