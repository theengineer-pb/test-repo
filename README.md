### Application Code

<img width="1280" height="720" alt="GCP-Project" src="https://github.com/user-attachments/assets/38fe01de-c6cc-4451-b068-c3d0c8d996a7" />

# DevSecOps Pipeline Automation on GCP

A complete DevSecOps implementation automating CI/CD with 
integrated security scanning, infrastructure provisioning, 
and monitoring on Google Cloud Platform.

## Architecture Overview

```
Developer pushes code to GitHub →
Jenkins CI Pipeline triggered (manual or via GitHub webhook) →
Security Scans (Trivy + SonarQube + OWASP) →
Docker Build & Push to GCP Artifact Registry →
Helm values.yaml updated →
ArgoCD detects change → Deploys to GKE →
App deployed and accessible via GKE LoadBalancer with SSL/TLS enabled through cert-manager and NGINX Ingress Controller
```

## Tech Stack

| Category | Tools |
|----------|-------|
| CI/CD | Jenkins |
| Infrastructure as Code | Terraform |
| Containerization | Docker |
| Container Registry | GCP Artifact Registry |
| Secret Management | Jenkins Credentials Store |
| Code Quality | SonarQube |
| Vulnerability Scanning | Trivy |
| Dependency Scanning | OWASP Dependency Check |
| Cloud Provider | Google Cloud Platform (GCP) |
| Kubernetes | Google Kubernetes Engine (GKE) |
| Monitoring | Prometheus + Grafana |
| GitOps | ArgoCD |

## Repository Structure

```
test-repo/
    ├── gke-terraform/
    │     ├── main.tf        → GKE cluster definition
    │     ├── provider.tf    → GCP provider configuration
    │     └── variable.tf    → Input variables
    │
    ├── Dockerfile           → Container build instructions
    ├── Jenkinsfile          → CI pipeline definition (Pipeline 1)
    |── Jenkinsfile-gke      → Infrastructure pipeline (Pipeline 2)
    └── index.html           → Application source code
```

## Pipelines

### Pipeline 1 — CI Pipeline (Jenkinsfile)
Manually triggered via Jenkins UI. (But, can be configured for automatic triggering via GitHub webhook on code push.)

```
1. Clean Workspace
2. Git Checkout
3. Authenticate with GCP (Service Account)
4. Trivy Filesystem Scan
5. SonarQube Code Analysis
6. Quality Gate Check
7. OWASP Dependency Check
8. Build Docker Image
9. Trivy Image Scan
10. Push to GCP Artifact Registry
11. Update helm/values.yaml in test-k8s repo
```

### Pipeline 2 — Infrastructure Pipeline (Jenkinsfile-gke)
Manually triggered to create or destroy GKE infrastructure.

```
1. Clean Workspace
2. Git Checkout
3. Terraform Init (connects to GCS remote state)
4. Terraform Plan (apply only)
5. Terraform Apply / Destroy
```

## Security Implementation

- **Trivy** scans both filesystem and Docker image at every build
- **SonarQube** analyzes code for bugs, vulnerabilities and code smells
- **OWASP** checks all third party dependencies for known CVEs
- **Quality Gate** blocks pipeline if code doesn't meet security standards
- **GCP IAM** service account with least privilege for all GCP operations
- **Jenkins Credentials Store** manages all secrets — nothing hardcoded

## Infrastructure

GKE cluster provisioned via Terraform with:
- Custom VPC network
- Dedicated node pools
- Remote state stored in GCS bucket
- Supports apply and destroy via parameterized pipeline

## Prerequisites

- GCP Project with billing enabled
- Jenkins VM with following tools installed:
  - Docker
  - Terraform
  - Trivy
  - gcloud CLI
  - SonarQube Scanner
- Jenkins Plugins:
  - SonarQube Scanner
  - OWASP Dependency Check
  - GCP credentials plugin
- Jenkins Credentials configured:
  - `gcp-service-account` — GCP Service Account JSON key
  - `gcp-project-id` — GCP Project ID
  - `github-token` — GitHub Personal Access Token
  - `Sonar-token` — SonarQube authentication token
  - `nvd-api-key` — NVD API key for OWASP scans

## Setup Instructions

### Step 1 — Clone Repository
```bash
git clone https://github.com/theengineer-pb/test-repo.git
```

### Step 2 — Create GCS Bucket for Terraform State
```bash
gsutil mb -p YOUR_PROJECT_ID gs://YOUR_BUCKET_NAME-tfstate
```

### Step 3 — Configure Jenkins Credentials
Add all credentials listed in Prerequisites section to Jenkins credentials store.

### Step 4 — Configure SonarQube
- Add Jenkins webhook in SonarQube: `http://JENKINS_IP:8080/sonarqube-webhook`
- Add SonarQube server in Jenkins system settings: `http://SONAR_IP:9000`

### Step 5 — Run Pipeline 2 (apply) to create GKE cluster

### Step 6 — Run Pipeline 1 to build and deploy application

## Monitoring

Prometheus and Grafana deployed on GKE cluster:
- **Prometheus** scrapes metrics every 15 seconds from nodes and pods
- **Grafana** visualizes metrics via pre-built dashboards:
  - Dashboard 315 — Kubernetes Cluster Overview
  - Dashboard 6417 — Kubernetes Detailed Monitoring
  - Dashboard 1860 — Node Exporter Full

## Related Repository

Kubernetes manifests and Helm charts:
**[test-k8s](https://github.com/theengineer-pb/test-k8s)**


