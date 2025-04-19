# 🚀 Learner Report Application - Containerized Deployment

This project showcases how to deploy a full-stack MERN (MongoDB, Express.js, React.js, Node.js) application using Kubernetes and Helm for container orchestration, along with Jenkins for continuous integration and deployment.

---

## 📚 Contents

- [Overview](#overview)
- [Tech Stack](#tech-stack)
- [System Requirements](#system-requirements)
- [Architecture](#architecture)
- [Running the App Locally](#running-the-app-locally)
- [Dockerization Steps](#dockerization-steps)
- [Kubernetes with Helm](#kubernetes-with-helm)
- [CI/CD with Jenkins](#cicd-with-jenkins)
- [EKS Cluster Setup](#eks-cluster-setup)
- [Common Issues & Fixes](#common-issues--fixes)
- [Screenshots](#screenshots)

---

## 🌐 Overview

This repository contains everything needed to build, containerize, and deploy a learner-report web app using modern DevOps practices. The application is modularized as follows:

- 📦 Backend: Node.js + Express.js
- 💾 Database: MongoDB
- 🎨 Frontend: React.js (served via Nginx)

Deployment is managed using Kubernetes and Helm charts, with Jenkins handling automated builds and deployments.

---

## 🧰 Tech Stack

- Node.js / Express.js
- MongoDB
- React.js
- Docker
- Kubernetes + Helm
- Jenkins (CI/CD)
- AWS EKS (Cloud Deployment)

---

## 💻 System Requirements

Ensure the following tools are installed:

- Docker
- kubectl
- Helm
- AWS CLI + `eksctl`
- Jenkins
- Git

---

## 📐 Architecture Diagram

```
[Frontend - React.js]
        |
        ↓
[Backend - Express.js]
        |
        ↓
[Database - MongoDB]
```

- Frontend runs on port 80
- Backend serves on port 3001
- MongoDB listens on port 27017

---

## 🔧 Running the App Locally

1. **Clone the Repositories**

   ```bash
   git clone https://github.com/UnpredictablePrashant/learnerReportCS_backend.git
   git clone https://github.com/UnpredictablePrashant/learnerReportCS_frontend.git
   ```

2. **Start Backend**

   ```bash
   cd learnerReportCS_backend
   npm install
   node index.js
   ```

3. **Start Frontend**

   ```bash
   cd learnerReportCS_frontend
   npm install --legacy-peer-deps
   npm start
   ```

---

## 🐳 Dockerization Steps

- Each module (backend, frontend) is containerized using Dockerfiles.

### Sample Commands:

```bash
# Backend
docker build -t youruser/learner-backend .
docker push youruser/learner-backend

# Frontend (Production)
docker build -t youruser/learner-frontend -f Dockerfile.prod .
docker push youruser/learner-frontend
```

---

## ⛵ Kubernetes with Helm

Helm simplifies our deployment process with reusable and customizable charts.

### To Deploy:

```bash
cd helm-chart
helm install learner-report ./ -f values.yaml
```

### Check Status:

```bash
helm status learner-report
kubectl get all
```

> Your Helm chart will deploy the backend, frontend, and database services via Kubernetes manifests (Deployment, Services, PVC, Secrets, etc.).

---

## 🛠 CI/CD with Jenkins

The Jenkins pipeline automates:

- Cloning code
- Building Docker images
- Pushing to Docker Hub
- Deploying via `kubectl` or `helm`

### Steps:

1. Create a pipeline job.
2. Configure SCM as Git.
3. Set script path as `Jenkinsfile`.
4. Add Docker and AWS credentials in Jenkins.
5. Trigger the build.

---

## ☁️ EKS Cluster Setup (AWS)

### Create Cluster:

```bash
eksctl create cluster   --name learner-eks   --region us-west-1   --nodegroup-name learner-nodes   --node-type t3.medium   --nodes 2   --managed
```

### Connect to Cluster:

```bash
aws eks --region us-west-1 update-kubeconfig --name learner-eks
```

Then re-deploy Helm charts using the same process as above.

---

## 🐛 Common Issues & Fixes

| Issue                         | Solution                                                                 |
|------------------------------|--------------------------------------------------------------------------|
| Frontend can't reach backend | Ensure backend service URL is correct and exposed. Update frontend secret. |
| DB not connecting            | Verify MongoDB pod logs, PVC status, and connection string in backend.    |
| Jenkins build fails          | Double-check credentials and Dockerfile paths in Jenkins pipeline.        |

---

## 🖼 Screenshots

> Added screenshots of the local app, pods, services, Helm status, Jenkins pipeline, and EKS deployment here for better understanding.

>  Local Deployemnt
> ![Screenshot 2025-04-06 061216](https://github.com/user-attachments/assets/916abbb6-b4d9-41d1-96ff-ac1e992c4378)

> Pods
![image](https://github.com/user-attachments/assets/6f5f063d-63bb-47b3-b828-407a20103c23)

Docker images
> ![Screenshot 2025-04-06 070706](https://github.com/user-attachments/assets/a7f44702-f13b-4e7a-9559-13051a05cba7)

Helm Status
> ![Screenshot 2025-04-19 225231](https://github.com/user-attachments/assets/3b31b82c-35d6-412d-b42d-897c231ffbfd)

> Jnekin Pipeline
> ![Screenshot 2025-04-19 215138](https://github.com/user-attachments/assets/25c66c03-8d67-4057-bef3-b2a809debd50)

> EKS deployemnt
  ![Screenshot 2025-04-19 215723](https://github.com/user-attachments/assets/79ab0bde-36d3-4c61-aa3a-45eb5ecaa14e)
> ![Screenshot 2025-04-19 214106](https://github.com/user-attachments/assets/6dee3231-277b-403d-9566-8bc3d56cb8c8)



