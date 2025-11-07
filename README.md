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

# 📊 Monitoring & Observability

## 🧠 Overview

To ensure system reliability and visibility, I implemented a monitoring stack using Prometheus and Grafana, deployed via Helm on the Kubernetes cluster.
This setup provides real-time insights into cluster performance, resource usage, and application-level metrics.

## ⚙️ Tools Used

Prometheus – For collecting and storing metrics from Kubernetes components, nodes, and application pods.

Grafana – For creating visual dashboards to analyze and track system health over time.

Helm – For easy and version-controlled installation of both Prometheus and Grafana.

## 🔧 Deployment Steps

Added the Helm repositories

```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo add grafana https://grafana.github.io/helm-charts
helm repo update
```

Deployed Prometheus

```bash
helm install prometheus prometheus-community/prometheus -n monitoring --create-namespace
```

Deployed Grafana

```bash
helm install grafana grafana/grafana -n monitoring
```

Retrieved the Grafana admin password

```bash
kubectl get secret grafana -n monitoring -o jsonpath="{.data.admin-password}" | base64 --decode
```

Port-forwarded Grafana for local access

```bash
kubectl port-forward svc/grafana -n monitoring 3000:80
```

Then accessed Grafana via 👉 http://localhost:3000

# 📈 Key Metrics Monitored

| Category               | Metric                                                                                    | Description                                      |
| ---------------------- | ----------------------------------------------------------------------------------------- | ------------------------------------------------ |
| **Kubernetes Nodes**   | `node_cpu_usage_seconds_total`, `node_memory_MemAvailable_bytes`                          | Tracks CPU and memory usage across nodes         |
| **Pods**               | `container_cpu_usage_seconds_total`, `container_memory_usage_bytes`                       | Measures per-container resource usage            |
| **Application**        | `http_server_requests_seconds_count`, `jvm_memory_used_bytes` (from Spring Boot Actuator) | Tracks API request latency and JVM memory        |
| **Cluster Health**     | `kube_pod_status_ready`, `kube_deployment_status_replicas_available`                      | Monitors pod and deployment status               |
| **MySQL** _(Optional)_ | `mysql_global_status_threads_connected`, `mysql_global_status_queries`                    | Tracks database connection and query performance |

# 📊 Grafana Dashboards

Cluster Overview: CPU, Memory, Node Utilization

Pod-Level Monitoring: Resource usage and restart counts

Application Performance: API latency and error rates

Database Metrics: Connection counts and query rates

Dashboards were built using the Prometheus data source and imported JSON templates from Grafana.com for faster setup.

# 🧩 Outcome

Achieved full observability of application and infrastructure.

Identified resource bottlenecks during testing and optimized resource requests/limits.

Demonstrated production-grade monitoring, alerting readiness, and visualization setup.
