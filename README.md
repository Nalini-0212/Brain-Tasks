# Brain Cluster Setup

# 🚀 Brain Tasks Application – CI/CD Deployment on AWS EKS

---

## 📌 Project Overview

This project demonstrates an end-to-end CI/CD pipeline to deploy a containerized application into **Amazon EKS (Kubernetes)** using:

- AWS CodePipeline (CI/CD orchestration)
- AWS CodeBuild (build & Docker image creation)
- Docker (containerization)
- Amazon EKS (deployment)
- CloudWatch Logs & Container Insights (Monitoring)

---

## 🏗️ Architecture
GitHub → CodePipeline → CodeBuild (Build & Push)
→ CodeBuild (Deploy) → Amazon EKS → LoadBalancer → Application

## 📂 Project Structure


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

---

## ⚙️ AWS CI/CD Flow

1. Code pushed to GitHub
2. CodePipeline triggers automatically
3. CodeBuild:
   - Builds Docker image
   - Pushes to DockerHub 
4. Deploy to Kubernetes


---

## 🐳 Docker Setup

### Dockerfile

```dockerfile
FROM httpd:2.4-alpine
RUN rm -rf /usr/local/apache2/htdocs/*
COPY dist/ /usr/local/apache2/htdocs/
EXPOSE 80
CMD ["httpd-foreground"]

### Build Docker Image

docker build -t braintasks:v1 -f app/Dockerfile .

### Run Container

docker run -d --name  braintasks-container -p 3000:80 braintasks:v1

👉 Serves the application using Apache on port 80

☸️ Kubernetes


kubectl apply -f k8s/
kubectl get pods



⚙️ CI/CD Pipeline Explanation
✅ Stage 1: Source

Code is pulled from GitHub repository


✅ Stage 2: Build (CodeBuild)

Builds Docker image from app/ folder
Tags image dynamically:

V1.<CODEBUILD_BUILD_NUMBER>


Pushes image to DockerHub
Generates:

versions.txt
image.txt


✅ Stage 3: Deploy (CodeBuild)
kubectl apply -f k8s/deployment.yaml
kubectl apply -f k8s/service.yaml
kubectl set image deployment/brain-deployment \ brain-container=$IMAGE_URL
kubectl rollout status deployment/brain-deployment

👉 Ensures zero-downtime rolling updates

☸️ Kubernetes Deployment
✅ Deployment

3 replicas
Rolling updates enabled

✅ Service

Type: LoadBalancer
Port mapping:

3000 → 80

Application URL

http://a29be4fcfee3948b2a72c6225da8f3e9-811224438.us-east-1.elb.amazonaws.com:3000/

🔑 Kubernetes Cluster ARN
arn:aws:eks:us-east-1:767397831600:cluster/brain-cluster

📊 Monitoring (CloudWatch)
✅ Build Logs
/aws/codebuild/brain-tasks

✅ Deploy Logs
/aws/codebuild/eks-deploy-brain-tasks

✅ Application Logs
/aws/containerinsights/brain-cluster/application

👉 Logs collected using:

Fluent Bit
CloudWatch Agent


📊 Container Insights


aws eks create-addon \
  --cluster-name brain-cluster \
  --addon-name amazon-cloudwatch-observability \
  --region us-east-1

✅ Verification Commands


kubectl get pods
kubectl get svc
kubectl logs -l app=brain

# 📸 Screenshots

---

### ✅ 1. CodePipeline Success

 ![Pipeline Success](screenshots/pipeline.png)

👉 Shows all stages completed successfully (Source, Build, Deploy)

✅ 2. Kubernetes Pods Running
screenshots/pods.png

Output of:
kubectl get pods

✅ 3. CloudWatch Logs
screenshots/cloudwatch.png
👉 Shows application logs from:
/aws/containerinsights/brain-cluster/application

✅ 4. Application Output
screenshots/app-url.png
👉 Application accessed via LoadBalancer

✅ Key Features

✅ End-to-end CI/CD pipeline
✅ Dockerized application
✅ Kubernetes deployment on AWS EKS
✅ Dynamic image versioning
✅ CloudWatch monitoring enabled
✅ Container Insights enabled

🛠️ Author & Community
This project is maintained by Nalini Selvaraj 💡. Your feedback and contributions are welcome!

📧 Connect with me:

GitHub: @Nalini-0212















