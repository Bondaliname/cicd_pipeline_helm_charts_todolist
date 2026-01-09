# Django ToDo list

**CI/CD pipeline** demonstrating automated deployment of a Django **TodoList** application to Kubernetes using **Helm charts** and **GitHub Actions**.

This project showcases **DevOps best practices** and is designed as a **portfolio project**.

---

## Features

* **Automated CI/CD Pipeline** – GitHub Actions workflow with testing and deployment
* **Kubernetes Deployment** – Uses **Kind (Kubernetes in Docker)** for local testing
* **Helm Charts** – Declarative infrastructure with **MySQL subchart dependency**
* **Multi-Environment Support** – Separate **development** and **staging** configurations
* **Infrastructure as Code** – All Kubernetes resources defined via Helm templates
* **Automated Testing** – Python unit tests, coverage, and linting
* **Secret Management** – Secure handling of sensitive data via GitHub Secrets

---

## Architecture

The pipeline consists of **four main stages**:

```
Push to main branch
        ↓
1. Python CI & Tests
   (Unit tests, coverage, linting)
        ↓
2. Helm Lint & Package
   (Chart validation, templating)
        ↓
3. Deploy to Development
   (Kind cluster, Helm install)
        ↓
4. Deploy to Staging
   (Separate Kind cluster)
```

### Components

* **Django Application** – TodoList REST API (Python 3.9)
* **MySQL Database** – StatefulSet with persistent storage
* **Helm Charts** – Main chart with MySQL subchart
* **GitHub Actions** – Automated CI/CD orchestration
* **Kind** – Kubernetes clusters for local testing

---

## Running Locally

### Prerequisites

Make sure you have installed:

* Docker Desktop or Docker Engine
* Kind (Kubernetes in Docker)
* Helm 3.x
* kubectl
* Python 3.9+

---

## Setup Instructions

### Clone the repository

```bash
git clone https://github.com/Bondaliname/cicd_pipeline_helm_charts_todolist.git
cd cicd_pipeline_helm_charts_todolist
```

---

### Create Kind cluster

```bash
kind create cluster --name todolist --config cluster.yml
kubectl cluster-info --context kind-todolist
```

---

### Update Helm dependencies

```bash
cd helm-charts/todoapp
helm dependency update
cd ../..
```

---

### Create secrets file

```bash
cat > helm-charts/secrets.yaml <<EOF
mysql:
  secrets:
    MYSQL_ROOT_PASSWORD: "your-root-password"
    MYSQL_USER: "app_user"
    MYSQL_PASSWORD: "your-password"

todoapp:
  secrets:
    SECRET_KEY: "django-secret-key-here"
    DB_NAME: "app_db"
    DB_USER: "app_user"
    DB_PASSWORD: "your-password"
    DB_HOST: "mysql.mysql.svc.cluster.local"
EOF
```

**Important:** Never commit `secrets.yaml` to version control!

---

### Deploy the application

```bash
helm install todoapp helm-charts/todoapp \
  -f helm-charts/todoapp/values.yaml \
  -f helm-charts/secrets.yaml \
  --create-namespace
```

---

## GitHub Actions Setup

### Required Secrets

Configure these secrets in your GitHub repository
**Settings → Secrets and variables → Actions**

#### Development Environment

* MYSQL_ROOT_PASSWORD
* MYSQL_USER
* MYSQL_PASSWORD
* SECRET_KEY
* DB_NAME
* DB_USER
* DB_PASSWORD
* DB_HOST

#### Staging Environment

* MYSQL_ROOT_PASSWORD_STAGING
* MYSQL_PASSWORD_STAGING
* SECRET_KEY_STAGING
* DB_PASSWORD_STAGING

---

### Pipeline Triggers

* **Push to `main`** – Runs full pipeline (tests + deployments)
* **Pull Request** – Runs CI tests only
* **Manual Trigger** – Via GitHub Actions UI

---

## Cleanup

### Remove Helm deployments

```bash
helm uninstall todoapp -n todoapp
helm uninstall todoapp-staging -n todoapp-staging
```

### Delete Kind cluster

```bash
kind delete cluster --name todolist
```

### Full cleanup

```bash
kubectl delete namespace todoapp
kubectl delete namespace mysql
kind delete cluster --name todolist
```

---
 **Portfolio Project** — Demonstrating DevOps expertise with **CI/CD, Kubernetes, and Helm**
 
