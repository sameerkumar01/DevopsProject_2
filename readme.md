DevSecOps Integration Project
This project implements a DevSecOps pipeline using GitHub Actions to automate CI/CD workflows, integrating security scanning with tfsec for Terraform code, Trivy for Docker image scanning, and Sealed Secrets for secure Kubernetes secret management. The pipeline triggers on each code push, performs security scans, manages encrypted secrets, and deploys workloads to a Kubernetes cluster or updates infrastructure via Terraform.
Table of Contents

Project Overview
Directory Structure
Prerequisites
Setup Instructions
Pipeline Workflow
Security Tools
Usage
Contributing
License

Project Overview
The goal of this project is to create a secure, automated CI/CD pipeline that:

Executes on every code push to the main branch.
Scans Terraform configurations with tfsec for misconfigurations.
Scans Docker images with Trivy for vulnerabilities.
Manages Kubernetes secrets securely using Sealed Secrets.
Deploys workloads to a Kubernetes cluster and updates infrastructure with Terraform.

The pipeline ensures a robust DevSecOps workflow, embedding security practices early in the development cycle (shift-left security) and maintaining compliance with cloud security best practices.
Directory Structure
devsecops-project/
├── .github/
│   └── workflows/
│       └── main.yml           # GitHub Actions workflow
├── terraform/
│   ├── main.tf               # Terraform configuration for S3 bucket
│   ├── variables.tf          # Terraform variables
│   └── terraform.tfvars      # Terraform variable values
├── k8s/
│   ├── deployment.yaml       # Kubernetes deployment manifest
│   ├── service.yaml          # Kubernetes service manifest
│   ├── secrets.yaml          # Sample Kubernetes secret
│   └── sealed-secrets.yaml   # Generated Sealed Secrets manifest
├── Dockerfile                # Sample Dockerfile for Node.js app
└── .trivyignore              # Trivy ignore file for specific CVEs

Prerequisites

GitHub Repository: A repository with write access to configure secrets and workflows.
Kubernetes Cluster: A running cluster with the Sealed Secrets controller installed.
AWS Account: For Terraform to manage AWS resources (e.g., S3 bucket).
Docker: For building and scanning container images.
GitHub Secrets:
KUBE_CONFIG: Kubernetes configuration for kubectl.
AWS_ACCESS_KEY_ID: AWS access key for Terraform.
AWS_SECRET_ACCESS_KEY: AWS secret key for Terraform.



Setup Instructions

Clone the Repository:
git clone <repository-url>
cd devsecops-project


Install Sealed Secrets Controller:

Follow the Sealed Secrets documentation to install the controller in your Kubernetes cluster.
Fetch the public certificate for sealing secrets:kubeseal --fetch-cert > pub-cert.pem




Configure GitHub Secrets:

Go to your GitHub repository → Settings → Secrets and variables → Actions.
Add the following secrets:
KUBE_CONFIG: Your Kubernetes cluster configuration.
AWS_ACCESS_KEY_ID: Your AWS access key.
AWS_SECRET_ACCESS_KEY: Your AWS secret key.




Customize Files:

Update terraform/terraform.tfvars with your desired AWS resource names (e.g., bucket_name).
Modify k8s/secrets.yaml with your application secrets (encode values in base64).
Adjust the Dockerfile to match your application’s requirements.


Push to Main Branch:
git add .
git commit -m "Initial setup"
git push origin main

This triggers the GitHub Actions pipeline.


Pipeline Workflow
The GitHub Actions workflow (.github/workflows/main.yml) performs the following steps:

Checkout Code: Clones the repository.
Setup Terraform: Installs Terraform (version 1.5.6).
Run tfsec: Scans Terraform code for misconfigurations and uploads results to GitHub Security.
Build Docker Image: Builds the application’s Docker image.
Run Trivy: Scans the Docker image for vulnerabilities (CRITICAL, HIGH severity) and uploads results.
Install kubeseal: Sets up the Sealed Secrets CLI.
Create Sealed Secrets: Encrypts secrets.yaml into sealed-secrets.yaml.
Setup kubectl: Configures kubectl for Kubernetes access.
Deploy to Kubernetes: Applies sealed-secrets.yaml, deployment.yaml, and service.yaml.
Apply Terraform: Initializes and applies Terraform configurations to update infrastructure.

The workflow is visualized in the following Mermaid diagram:
graph TD
    A[Code Push to Main Branch] -->|Trigger GitHub Actions| B[Checkout Code]
    B --> C[Setup Terraform]
    C --> D[Run tfsec Scan]
    D -->|Upload Results| E[tfsec Results]
    B --> F[Build Docker Image]
    F --> G[Run Trivy Scan]
    G -->|Upload Results| H[Trivy Results]
    B --> I[Install kubeseal]
    I --> J[Create Sealed Secrets]
    J --> K[Setup kubectl]
    K --> L[Deploy to Kubernetes]
    L -->|Apply Sealed Secrets| M[SealedSecrets]
    L -->|Apply Deployment & Service| N[Kubernetes Cluster]
    C --> O[Terraform Init]
    O --> P[Terraform Apply]
    P --> Q[Infrastructure Updated]

Security Tools

tfsec: Scans Terraform code for misconfigurations, ensuring compliance with cloud security best practices.
Trivy: Scans Docker images for vulnerabilities, focusing on CRITICAL and HIGH severity issues.
Sealed Secrets: Encrypts Kubernetes secrets using the cluster’s public key, allowing secure storage in the Git repository.

Usage

Monitor Pipeline:

Check the GitHub Actions tab in your repository to view workflow runs and logs.
Review security scan results in the GitHub Security tab (Code Scanning Alerts).


Verify Deployments:

Confirm Kubernetes deployments:kubectl get pods -n default
kubectl get svc -n default


Verify Terraform infrastructure:aws s3 ls s3://devsecops-example-bucket-2025




Update Secrets:

Modify k8s/secrets.yaml with new secrets (base64-encoded).
The pipeline automatically generates k8s/sealed-secrets.yaml using kubeseal.



Contributing
Contributions are welcome! To contribute:

Fork the repository.
Create a feature branch (git checkout -b feature/xyz).
Commit changes (git commit -m "Add feature xyz").
Push to the branch (git push origin feature/xyz).
Open a pull request.

Please ensure your changes pass tfsec and Trivy scans and include relevant tests.
License
This project is licensed under the MIT License. See the LICENSE file for details.