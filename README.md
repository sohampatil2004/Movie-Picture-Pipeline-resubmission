# Movie Picture Pipeline


**Author:** Soham Patil 
**Live Frontend Link:** (http://a5bfcb27ca6c34e6292438ce420cf157-2004026337.us-east-1.elb.amazonaws.com)

A complete DevOps CI/CD pipeline for the Movie Picture application using GitHub Actions, Docker, Amazon ECR, Amazon EKS, Kubernetes, Kustomize, and Terraform.

## Project Overview

The Movie Picture Pipeline consists of two applications:

1. **Frontend** - React application providing the Movie List user interface.
2. **Backend** - Python Flask API providing movie data.

The project automates the complete development and deployment workflow using GitHub Actions.

The pipeline performs:

- Source code validation
- Linting
- Automated testing
- Docker image creation
- Amazon ECR image publishing
- Kubernetes deployment
- Amazon EKS deployment
- Git SHA-based Docker image tagging
- Frontend and backend continuous integration
- Frontend and backend continuous deployment

---

## Live Application

### Frontend

**Live Frontend URL:**

http://a5bfcb27ca6c34e6292438ce420cf157-2004026337.us-east-1.elb.amazonaws.com

The frontend is exposed through an AWS Elastic Load Balancer created by the Kubernetes `LoadBalancer` service.

### Backend API

**Backend Load Balancer:**

http://a1f85687526034cd0b74c05e03a80145-2028645763.us-east-1.elb.amazonaws.com

### Movies API

```text
http://a1f85687526034cd0b74c05e03a80145-2028645763.us-east-1.elb.amazonaws.com/movies
```

The `/movies` endpoint returns the movie catalog used by the frontend.

---

# Architecture

```text
                    GitHub Repository
                           |
                           |
                    GitHub Actions
                           |
              +------------+------------+
              |                         |
        Frontend CI                Backend CI
              |                         |
        Lint + Test                Lint + Test
              |                         |
              +------------+------------+
                           |
                    Build Docker Images
                           |
              +------------+------------+
              |                         |
          Frontend                    Backend
              |                         |
              +------------+------------+
                           |
                      Amazon ECR
                           |
              +------------+------------+
              |                         |
        Frontend CD                Backend CD
              |                         |
              +------------+------------+
                           |
                       Amazon EKS
                           |
              +------------+------------+
              |                         |
        Frontend Pod                Backend Pod
              |                         |
        LoadBalancer              LoadBalancer
              |                         |
        React Application          Flask API
```

---

# Technology Stack

| Technology | Purpose |
|---|---|
| React | Frontend application |
| Node.js 18.14.2 | Frontend runtime/build |
| Python 3.10 | Backend runtime |
| Flask | Backend API |
| Docker | Containerization |
| GitHub Actions | CI/CD automation |
| Amazon ECR | Docker image registry |
| Amazon EKS | Kubernetes cluster |
| Kubernetes | Container orchestration |
| Kustomize | Kubernetes manifest management |
| Terraform | AWS infrastructure provisioning |
| AWS IAM | Authentication and authorization |
| AWS Load Balancer | Public application access |

---

# Project Structure

```text
Movie-Picture-Pipeline-resubmission/
│
├── .github/
│   └── workflows/
│       ├── frontend-ci.yml
│       ├── backend-ci.yml
│       ├── frontend-cd.yml
│       └── backend-cd.yml
│
├── starter/
│   │
│   ├── frontend/
│   │   ├── src/
│   │   ├── k8s/
│   │   │   ├── deployment.yaml
│   │   │   ├── service.yaml
│   │   │   └── kustomization.yaml
│   │   ├── Dockerfile
│   │   ├── package.json
│   │   └── ...
│   │
│   └── backend/
│       ├── movies/
│       ├── k8s/
│       │   ├── deployment.yaml
│       │   ├── service.yaml
│       │   └── kustomization.yaml
│       ├── Dockerfile
│       ├── Pipfile
│       └── ...
│
└── setup/
    └── terraform/
        ├── main.tf
        ├── variables.tf
        ├── outputs.tf
        ├── versions.tf
        └── .terraform.lock.hcl
```

---

# Continuous Integration

Both applications have independent CI workflows.

## Frontend CI

Workflow:

```text
Frontend Continuous Integration
```

The workflow runs when:

- A pull request targets `main`
- Changes are made under:

```text
starter/frontend/**
```

It can also be manually triggered using `workflow_dispatch`.

### Frontend CI jobs

```text
Lint
  |
Test
  |
Build
```

The build job runs only after both lint and test jobs succeed.

### Frontend validation

```bash
cd starter/frontend

npm ci

npm run lint

CI=true npm test -- --watchAll=false

npm run build
```

The frontend tests successfully passed.

---

# Backend CI

The backend CI workflow validates the Flask application.

The workflow runs when backend application code changes and supports manual execution.

### Backend CI jobs

```text
Lint
  |
Test
  |
Build
```

The build job runs only after both lint and test jobs succeed.

### Backend validation

```bash
cd starter/backend

pipenv install --dev

pipenv run lint

pipenv run test
```

The backend tests successfully passed.

---

# Continuous Deployment

Both applications have independent CD workflows.

## Backend CD

The backend CD workflow:

1. Runs linting
2. Runs automated tests
3. Builds the Docker image
4. Logs into Amazon ECR
5. Pushes the Docker image to ECR
6. Configures Kubernetes access
7. Updates the Kubernetes image using Kustomize
8. Deploys the backend to Amazon EKS

The Docker image is tagged using the Git commit SHA.

Example:

```text
backend:<git-sha>
```

### Backend ECR Repository

```text
535749839925.dkr.ecr.us-east-1.amazonaws.com/backend
```

---

# Frontend CD

The frontend CD workflow:

1. Installs frontend dependencies
2. Runs linting
3. Runs frontend tests
4. Configures AWS credentials
5. Logs into Amazon ECR
6. Builds the frontend Docker image
7. Passes `REACT_APP_MOVIE_API_URL` during the Docker build
8. Pushes the image to ECR
9. Configures kubectl
10. Updates the Kubernetes image using Kustomize
11. Deploys the frontend to Amazon EKS

The Docker image is tagged with the Git commit SHA.

### Frontend ECR Repository

```text
535749839925.dkr.ecr.us-east-1.amazonaws.com/frontend
```

---

# Docker Configuration

## Frontend Dockerfile

The frontend image uses Node.js:

```dockerfile
FROM public.ecr.aws/docker/library/node:18.14.2-alpine3.17

ARG REACT_APP_MOVIE_API_URL
ENV REACT_APP_MOVIE_API_URL=${REACT_APP_MOVIE_API_URL}

WORKDIR /app

COPY package*.json ./

RUN npm ci

COPY . .

RUN npm run build

EXPOSE 3000

CMD ["npm", "run", "serve"]
```

The backend API URL is supplied at image build time using:

```text
REACT_APP_MOVIE_API_URL
```

---

## Backend Dockerfile

The backend uses Python 3.10 Alpine:

```dockerfile
FROM public.ecr.aws/docker/library/python:3.10-alpine3.17
```

The backend dependencies are installed using Pipenv.

---

# Kubernetes

The application is deployed to Amazon EKS using Kubernetes manifests managed with Kustomize.

## Frontend Kubernetes resources

Location:

```text
starter/frontend/k8s/
```

Resources:

```text
deployment.yaml
service.yaml
kustomization.yaml
```

Frontend service:

```yaml
type: LoadBalancer
```

The frontend receives a public AWS Load Balancer endpoint.

---

## Backend Kubernetes resources

Location:

```text
starter/backend/k8s/
```

Resources:

```text
deployment.yaml
service.yaml
kustomization.yaml
```

Backend service:

```yaml
type: LoadBalancer
```

The backend receives a public AWS Load Balancer endpoint.

---

# Kustomize

Kustomize is used to update Docker image tags without manually editing the Kubernetes deployment manifests.

## Backend

```bash
cd starter/backend/k8s

kustomize edit set image backend=<ECR_REPOSITORY>:<GIT_SHA>

kustomize build | kubectl apply -f -
```

## Frontend

```bash
cd starter/frontend/k8s

kustomize edit set image frontend=<ECR_REPOSITORY>:<GIT_SHA>

kustomize build | kubectl apply -f -
```

The CI/CD workflows perform these operations automatically.

---

# AWS Infrastructure

Terraform was used to provision the AWS infrastructure required by the application.

The infrastructure includes:

- Amazon EKS cluster
- EKS managed node group
- VPC
- Subnets
- Internet Gateway
- Route tables
- IAM roles
- IAM policies
- Amazon ECR repositories
- GitHub Actions IAM user
- Supporting AWS resources

## Terraform Version

```text
Terraform 1.3.9
```

## AWS Region

```text
us-east-1
```

## Kubernetes Version

```text
1.33
```

## EKS Cluster

```text
cluster
```

---

# Terraform Commands

Navigate to:

```bash
cd setup/terraform
```

Initialize Terraform:

```bash
terraform init
```

Validate the configuration:

```bash
terraform validate
```

Review infrastructure changes:

```bash
terraform plan
```

Apply infrastructure:

```bash
terraform apply
```

View Terraform outputs:

```bash
terraform output
```

---

# EKS Verification

Configure kubectl:

```bash
aws eks update-kubeconfig \
  --name cluster \
  --region us-east-1
```

Check nodes:

```bash
kubectl get nodes
```

Check deployments:

```bash
kubectl get deployments
```

Check pods:

```bash
kubectl get pods
```

Check services:

```bash
kubectl get svc
```

Expected application state:

```text
Backend Pod     1/1 Running
Frontend Pod    1/1 Running
```

---

# Backend API Verification

The backend service is exposed through an AWS Load Balancer.

```bash
curl http://a1f85687526034cd0b74c05e03a80145-2028645763.us-east-1.elb.amazonaws.com/movies
```

The API provides the movie catalog.

Example movies include:

- Top Gun: Maverick
- Sonic the Hedgehog
- A Quiet Place

---

# Frontend Verification

Open the live frontend URL:

```text
http://a5bfcb27ca6c34e6292438ce420cf157-2004026337.us-east-1.elb.amazonaws.com
```

The React application should display the Movie List and retrieve movie data from the backend API.

---

# GitHub Actions Secrets

The GitHub repository requires the following secrets for AWS deployment:

```text
AWS_ACCESS_KEY_ID
AWS_SECRET_ACCESS_KEY
REACT_APP_MOVIE_API_URL
```

`REACT_APP_MOVIE_API_URL` contains the public backend Load Balancer URL used by the React application.

Example:

```text
http://a1f85687526034cd0b74c05e03a80145-2028645763.us-east-1.elb.amazonaws.com
```

**Never commit AWS credentials, access keys, secret keys, session tokens, or Terraform state files to Git.**

---

# CI/CD Workflow Summary

## Frontend

```text
Code Change
    |
    v
Frontend CI
    |
    +--> Lint
    |
    +--> Test
    |
    v
Build
    |
    v
Docker Image
    |
    v
Amazon ECR
    |
    v
Frontend CD
    |
    v
Amazon EKS
    |
    v
Frontend LoadBalancer
```

## Backend

```text
Code Change
    |
    v
Backend CI
    |
    +--> Lint
    |
    +--> Test
    |
    v
Build
    |
    v
Docker Image
    |
    v
Amazon ECR
    |
    v
Backend CD
    |
    v
Amazon EKS
    |
    v
Backend LoadBalancer
```

---

# Verification Results

The completed project was verified successfully.

### Frontend CI

```text
PASSED
```

### Backend CI

```text
PASSED
```

### Backend CD

```text
PASSED
```

### Frontend CD

```text
PASSED
```

### Backend Pod

```text
1/1 Running
```

### Frontend Pod

```text
1/1 Running
```

### Backend Service

```text
LoadBalancer
```

### Frontend Service

```text
LoadBalancer
```

### Live Frontend

```text
http://a5bfcb27ca6c34e6292438ce420cf157-2004026337.us-east-1.elb.amazonaws.com
```

---

# Security and Repository Hygiene

The following files/directories should not be committed:

```text
setup/terraform/.terraform/
setup/terraform/terraform.tfstate
setup/terraform/terraform.tfstate.backup
setup/terraform/.terraform.tfstate.lock.info
```

AWS credentials must never be stored directly in source code.

GitHub Actions obtains AWS credentials from GitHub repository secrets.

---

# Cleanup

AWS resources incur costs while running.

After all testing and submission evidence has been collected, destroy the Terraform-managed infrastructure:

```bash
cd setup/terraform

terraform destroy
```

Review the resources that will be removed and confirm the operation when prompted.

**Do not run `terraform destroy` until all required project evidence and screenshots have been collected.**

---

# Useful Commands

### Check Git status

```bash
git status
```

### Commit changes

```bash
git add .
git commit -m "Update project"
```

### Push changes

```bash
git push origin main
```

### Check Kubernetes pods

```bash
kubectl get pods
```

### Check Kubernetes services

```bash
kubectl get svc
```

### Check deployments

```bash
kubectl get deployments
```

### Check backend logs

```bash
kubectl logs deployment/backend
```

### Check frontend logs

```bash
kubectl logs deployment/frontend
```

---

# Final Project Outcome

The Movie Picture Pipeline implements a complete automated CI/CD solution for a React frontend and Flask backend.

The solution uses GitHub Actions to validate, build, containerize, publish, and deploy both applications.

Docker images are stored in Amazon ECR, while the applications run on Amazon EKS and are exposed through AWS Load Balancers.

The final deployment was successfully verified through the live frontend URL and Kubernetes application status.

---

# License

See [LICENSE.md](LICENSE.md).