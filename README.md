# 🚀 Brain Tasks Application – CI/CD Deployment on AWS EKS

---

## 📌 Project Overview

This project demonstrates the deployment of a production-ready React application using Docker and Kubernetes on Amazon EKS. The application is containerized with Docker, stored in Docker Hub, and automatically built and deployed through AWS CodePipeline and AWS CodeBuild. Monitoring is implemented using Amazon CloudWatch Logs and Container Insights.

The objective is to automate the complete CI/CD pipeline from source code to deployment on a Kubernetes cluster.

---

## Project Architecture

```

GitHub Repository
        │
        ▼
 AWS CodePipeline
        │
        ▼
 AWS CodeBuild
        │
        ├── Build Docker Image
        ├── Push Image to Docker Hub
        └── Deploy to Amazon EKS
                │
                ▼
        Kubernetes Deployment
                │
                ▼
        Kubernetes Service (LoadBalancer)
                │
                ▼
         React Application

```

---

## Prerequisites

Ensure the following tools are installed and configured before deploying the application:

- Git
- Docker
- Docker Hub account
- AWS CLI
- kubectl
- eksctl
- AWS Account with required IAM permissions

---

## Technologies Used

- React (Vite)
- Docker
- Docker Hub
- Kubernetes
- Amazon EKS
- AWS CodeBuild
- AWS CodePipeline
- AWS CloudWatch
- Git & GitHub

---

## 📂 Project Structure

```

Brain-Tasks/
│
├── app/                         # Application + Docker
│   ├── Dockerfile
│   ├── dist/                   # React build files
│
├── cicd/                       # CI/CD pipeline config
│   ├── buildspec.yml
│   ├── buildspec-deploy.yml
│
├── k8s/                        # Kubernetes manifests
│   ├── deployment.yaml
│   ├── service.yaml
│
├── screenshots/                # Proof (important for evaluation)
│   ├── pipeline.png
│   ├── pods.png
│   ├── cloudwatch.png
│   ├── app-url.png
│
├── docs/ (optional)            # Your Word/PDF explanation
│   └── Application-Deployment.docx
│
└── README.md

```

---

# Clone Repository

```bash
git clone https://github.com/Nalini-0212/Brain-Tasks.git

cd Brain-Tasks
```

---

# Docker

## Dockerfile

```dockerfile
FROM httpd:2.4-alpine
RUN rm -rf /usr/local/apache2/htdocs/*
COPY dist/ /usr/local/apache2/htdocs/
EXPOSE 80
CMD ["httpd-foreground"]
```

### Build Image

```bash
cd app
docker build -t <dockerhub-username>/braintask:latest .
```
Example

```bash
docker build -t naliniselv/braintask:latest .
```
OR

USE below command

```bash
docker build -t naliniselv/braintask:latest -f app/Dockerfile .
```
### Run Container

```bash
docker run -d --name  braintasks-container -p 3000:80 naliniselv/braintask:latest
```

👉 Serves the application using Apache on port 80

---

Open

```
http://localhost:3000
```

---

## Push Image to Docker Hub

```bash
docker login

docker push naliniselv/braintask:latest
```

---

## Docker Hub Repository

Docker Image:

```text
docker.io/naliniselv/braintask:latest
```

---

# Amazon EKS

## Create EKS cluster

---

```bash
eksctl create cluster --name brain-cluster --region us-east-1 --nodegroup-name brain-sg --node-type t2.medium --nodes 2
```
---

## Configure kubectl

```bash
aws eks update-kubeconfig \
--region us-east-1 \
--name brain-cluster
```

---

## Deploy Application

```bash
kubectl apply -f k8s/deployment.yaml

kubectl apply -f k8s/service.yaml
```

---

## Verify Deployment

```bash
kubectl get nodes

kubectl get deployment

kubectl get pods

kubectl get svc
```
## Verification Output

### Expected Output

```bash
kubectl get pods
```

```
NAME                                  READY   STATUS    RESTARTS
brain-deployment-xxxxx                1/1     Running   0
brain-deployment-yyyyy                1/1     Running   0
brain-deployment-zzzzz                1/1     Running   0
```

```bash
kubectl get svc
```

```
NAME            TYPE           EXTERNAL-IP
brain-service   LoadBalancer   a29be4fc...
```

---

# CI/CD Pipeline

The project uses AWS CodePipeline to automate deployments.

Pipeline Flow

```
GitHub
   │
   ▼
CodePipeline
   │
   ▼
CodeBuild
   │
   ├── Build Docker Image
   ├── Push Image to Docker Hub
   ├── Update Kubernetes Manifest
   └── Deploy to Amazon EKS
```

## CI/CD Pipeline Explanation

### Stage 1 – Source

- Source code is retrieved from the GitHub repository through AWS CodePipeline.

### Stage 2 – Build (AWS CodeBuild)

- Builds the Docker image from the `app/` directory.
- Tags the Docker image using the build number (`V1.<CODEBUILD_BUILD_NUMBER>`).
- Pushes the image to Docker Hub.
- Generates `versions.txt` and `image.txt`.

### Stage 3 – Deploy

- Updates the Kubernetes deployment.
- Applies the Kubernetes manifests.
- Performs a rolling update with zero downtime.

```bash
kubectl apply -f k8s/deployment.yaml

kubectl apply -f k8s/service.yaml

kubectl set image deployment/brain-deployment \
brain-container=$IMAGE_URL

kubectl rollout status deployment/brain-deployment
```

👉 Ensures zero-downtime rolling updates

## Kubernetes Resources

### Deployment

- Replicas: 3
- Container Image: Docker Hub
- Container Port: 80

### Service

- Type: LoadBalancer
- Service Port: 80
- Target Port: 80
---

# Application URL

http://a29be4fcfee3948b2a72c6225da8f3e9-811224438.us-east-1.elb.amazonaws.com/

---
# Amazon EKS Cluster

**Cluster Name**

brain-cluster

**Cluster ARN**

arn:aws:eks:us-east-1:767397831600:cluster/brain-cluster

---

# AWS LoadBalancer ARN

arn:aws:elasticloadbalancing:us-east-1:767397831600:loadbalancer/net/k8s-default-brain-service-xxxxxxxx

---

# AWS CodeBuild

CodeBuild performs the following tasks:

- Downloads source code
- Builds Docker image
- Logs in to Docker Hub
- Pushes Docker image
- Updates Kubernetes deployment manifest
- Deploys application to Amazon EKS using kubectl

Build specification file:

```
cicd/buildspec.yml
```

Deploy specification file:

```
cicd/buildspec-deploy.yml
```

---

# Monitoring

AWS CloudWatch Logs are used to monitor:

- CodeBuild execution
- Deployment logs
- Build errors
- Kubernetes deployment status

## CloudWatch Monitoring

CloudWatch is used to monitor the complete CI/CD workflow.

### Build Logs

```
/aws/codebuild/brain-tasks
```

### Deployment Logs

```
/aws/codebuild/eks-deploy-brain-tasks
```

### Application Logs

```
/aws/containerinsights/brain-cluster/application
```

Container Insights is enabled using the Amazon CloudWatch Observability add-on to collect Kubernetes metrics and application logs.

👉 Logs collected using:

Fluent Bit
CloudWatch Agent


📊 Container Insights

```bash
aws eks create-addon \
  --cluster-name brain-cluster \
  --addon-name amazon-cloudwatch-observability \
  --region us-east-1
```

✅ Verification Commands

```bash
kubectl get pods

kubectl get svc

kubectl logs -l app=brain
```

---

# Useful Commands

Docker

```bash
docker images

docker ps
```

Kubernetes

```bash
kubectl get nodes

kubectl get pods

kubectl get deployments

kubectl get svc

kubectl describe deployment brain-deployment

kubectl logs <pod-name>
```

---

# 📸 Screenshots
The following screenshots are included in the `screenshots/` folder:

| Screenshot | Description |
|------------|-------------|
| GitHub Repository | Source code repository |
| Docker Hub | Docker image repository |
| Amazon EKS | Running Kubernetes cluster |
| Kubernetes Pods | Running application pods |
| Kubernetes Service | LoadBalancer service |
| CodeBuild | Successful build execution |
| CodePipeline | Successful CI/CD pipeline |
| CloudWatch Logs | Build and deployment logs |
| Application | Running application in browser |

---

## Key Features

- End-to-end CI/CD pipeline using AWS CodePipeline
- Dockerized React application
- Docker Hub image repository
- Deployment on Amazon EKS
- Kubernetes LoadBalancer service
- Rolling updates with zero downtime
- CloudWatch Logs monitoring
- CloudWatch Container Insights

---

# Author

**Nalini Selvaraj**

- **GitHub Profile:** <https://github.com/Nalini-0212>
- **Repository:** <https://github.com/Nalini-0212/Brain-Tasks>
---

# Conclusion

This project demonstrates a complete DevOps CI/CD workflow for deploying a React application using Docker, Kubernetes, Amazon EKS, AWS CodeBuild, and AWS CodePipeline. The pipeline automates building, publishing, and deploying the application while CloudWatch provides centralized logging and monitoring.

---

## Project Status

**Status:** Completed

This project successfully demonstrates a complete CI/CD pipeline for deploying a Dockerized React application on Amazon EKS using AWS CodePipeline and CodeBuild. Monitoring is enabled through Amazon CloudWatch Logs and Container Insights.

---




















