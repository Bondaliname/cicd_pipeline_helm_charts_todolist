# Django ToDo list - CI/CD Pipeline with Helm Charts & Kubernetes

This project showcases DevOps best practices for deploying a Django TodoList application with MySQL database to Kubernetes using Helm charts. The pipeline automates testing, validation, and deployment across development and staging environments.
Live Demo: Docker Hub - todolist:2.0.0

## Pipeline Stages:

1. Python CI & Tests

Runs on Python 3.9
Installs dependencies from requirements.txt
Executes Django unit tests
Generates code coverage reports
Performs linting with flake8
Checks code complexity (max complexity: 6)

2. Helm Lint & Package

Updates Helm chart dependencies (MySQL subchart)
Validates Helm chart syntax and structure
Templates charts to verify rendering
Packages charts as .tgz artifacts
Uploads packaged charts for deployment

3. Deploy to Development

Creates Kind Kubernetes cluster
Deploys application using Helm
Configures MySQL database with persistent storage
Waits for all pods to be ready
Runs smoke tests and displays deployment status
Exposes application on NodePort 30007

4. Deploy to Staging

Creates separate Kind cluster for staging
Uses staging-specific configuration (stg.yaml)
Deploys with staging secrets and namespaces
Validates deployment health
Runs acceptance tests

## Pipeline Triggers:

Push to main: Full pipeline execution
Pull Request: CI tests only (no deployment)
Manual: Workflow dispatch option

## Local Setup

1. Clone Repository
bashgit clone https://github.com/Bondaliname/cicd_pipeline_helm_charts_todolist.git
cd cicd_pipeline_helm_charts_todolist

2. Create Kind Cluster
bashkind create cluster --name todolist --config cluster.yml
kubectl cluster-info --context kind-todolist

3. Update Helm Dependencies
bashcd helm-charts/todoapp
helm dependency update
cd ../..

4. Create Secrets File
bashcat > helm-charts/secrets.yaml <<EOF
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
Never commit secrets.yaml to version control!

5. Deploy to Development
bashhelm install todoapp helm-charts/todoapp \
  -f helm-charts/todoapp/values.yaml \
  -f helm-charts/secrets.yaml \
  --create-namespace
6. Deploy to Staging
bashhelm install todoapp-staging helm-charts/todoapp \
  -f helm-charts/todoapp/values.yaml \
  -f helm-charts/todoapp/values/stg.yaml \
  -f helm-charts/secrets.yaml \
  --create-namespace

## GitHub Secrets Configuration

Configure the following secrets in GitHub repository settings (Settings → Secrets and variables → Actions):
Development Environment:

MYSQL_ROOT_PASSWORD
MYSQL_USER
MYSQL_PASSWORD
SECRET_KEY
DB_NAME
DB_USER
DB_PASSWORD
DB_HOST

Staging Environment:

MYSQL_ROOT_PASSWORD_STAGING
MYSQL_PASSWORD_STAGING
SECRET_KEY_STAGING
DB_PASSWORD_STAGING