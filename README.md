# 🚀 Jenkins on Amazon EKS – End-to-End CI/CD Pipeline

A production-like CI/CD platform built using Jenkins on Amazon EKS with dynamic Kubernetes agents, Amazon ECR integration, backup & recovery, multibranch pipelines, automatic rollback, and monitoring.

---

## 📂 Repository Structure

```text
.
├── app/
├── Dockerfile
├── Jenkinsfile
├── deployment.yaml
├── demo-app-ingress.yaml
├── service.yaml
└── README.md
```

---

## 🏗️ Architecture

```text
GitHub
   │
   ▼
Jenkins Pipeline
   │
   ├── Code Validation
   ├── Docker Build
   ├── Push Image to Amazon ECR
   └── Deploy to Amazon EKS
                     │
                     ▼
            Kubernetes Deployment
                     │
                     ▼
                Application
```

---

## ✨ Features

- Jenkins deployed on Amazon EKS
- Dynamic Kubernetes agents
- Shared Library support
- Docker image build
- Amazon ECR integration
- Automated Kubernetes deployment
- Google Chat notifications
- Backup and recovery using Amazon S3
- Disaster recovery validation
- Multibranch pipeline support
- Automatic rollback
- Build dashboard and statistics

---

## 📁 Components

### app/

Contains the application source code.

### Dockerfile

Builds the application image.

### Jenkinsfile

Defines the CI/CD pipeline stages:

- Set Environment
- Build Notification
- Checkout Code
- Code Validation
- Docker Build
- Push to ECR
- Deploy to EKS
- Verify Deployment
- Post-build Notifications

### deployment.yaml

Kubernetes deployment manifest.

### service.yaml

Exposes the application using Kubernetes Service.

### demo-app-ingress.yaml

Ingress configuration for external access.

---

## ⚙️ Pipeline Flow

```text
GitHub Commit
      │
      ▼
Jenkins Trigger
      │
      ▼
Checkout Source
      │
      ▼
Code Validation
      │
      ▼
Docker Build
      │
      ▼
Push to Amazon ECR
      │
      ▼
Deploy to Amazon EKS
      │
      ▼
Verify Deployment
      │
      ▼
Google Chat Notification
```

---

## 🔀 Multibranch Pipeline

Supported branches:

- `main`
- `dev`
- `release`

Environment mapping:

| Branch | Namespace | Replicas |
|----------|-----------|-----------|
| dev | dev | 1 |
| release | qa | 2 |
| main | uat | 3 |

---

## 🔄 Automatic Rollback

In case of deployment failure:

- Kubernetes rollout history is checked.
- Previous stable revision is restored automatically.

Example:

```bash
kubectl rollout undo deployment demo-app -n uat
```

---

## 📊 Build Dashboard

Provides:

- Success rate
- Failed builds
- Latest builds
- Build statistics
- Deployment frequency

---

## 💾 Backup and Recovery

Backups are stored in Amazon S3.

Recovery validates:

- Jobs
- Credentials
- Plugins
- Build History
- Configuration

---

## 🛠️ Technologies Used

- Jenkins
- Amazon EKS
- Kubernetes
- Docker
- Amazon ECR
- Amazon S3
- Helm
- GitHub
- Google Chat Webhooks
- Groovy Shared Libraries

---

## 🚀 Getting Started

### Clone Repository

```bash
git clone https://github.com/praveen-4942/praveen-jenkins.git
cd praveen-jenkins
```

### Build Image

```bash
docker build -t app .
```

### Deploy

```bash
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
kubectl apply -f demo-app-ingress.yaml
```

---

## 📸 Pipeline Stages

✅ Checkout Code

✅ Code Validation

✅ Docker Build

✅ Push to Amazon ECR

✅ Deploy to Amazon EKS

✅ Verify Deployment

✅ Notifications

---

## 🌟 Bonus Features

### Bonus 1 – Multibranch Pipeline

- main
- dev
- release

### Bonus 2 – Automatic Rollback

- Kubernetes rollout undo

### Bonus 3 – Build Dashboard

- Success %
- Failed builds
- Deployment frequency

---

## 👨‍💻 Author

**Praveen Kumar G**

DevSecOps Engineer

GitHub: https://github.com/praveen-4942

LinkedIn: https://linkedin.com/in/praveen4942

---

## 📜 License

MIT License

Copyright © 2026 Praveen Kumar G
