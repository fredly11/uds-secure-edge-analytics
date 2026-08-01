# Secure Edge Analytics Platform

A hands-on portfolio project demonstrating **Terraform + Amazon EKS + UDS Core** for secure, mission-oriented Kubernetes platforms.

This project was built to showcase skills relevant to Defense Unicorns and similar platform engineering roles focused on secure, airgap-capable, and observable Kubernetes environments.

---

## Project Goals

- Provision a production-like Kubernetes cluster on AWS using Infrastructure as Code (Terraform)
- Deploy and explore **UDS Core** (Unified Defense Stack) — the open-source secure runtime platform from Defense Unicorns
- Understand the difference between the local k3d demo experience and a real EKS-based deployment
- Practice GitOps-friendly patterns, observability, identity, and policy concepts used in mission environments

---

## Architecture Overview

**Infrastructure Layer (Terraform)**
- VPC with public/private subnets
- Amazon EKS cluster (Kubernetes 1.32)
- Managed node groups
- IAM roles & IRSA-ready configuration

**Platform Layer**
- UDS Core (Istio, Keycloak, Grafana, Loki, Pepr, Prometheus stack, etc.)

**Application Layer (planned / partial)**
- Sample telemetry/sensor ingestion services
- Visualization via Grafana

---

## Technologies Used

| Category              | Tools / Services                          |
|-----------------------|-------------------------------------------|
| Infrastructure as Code| Terraform, terraform-aws-modules          |
| Compute               | Amazon EKS, EC2 (dev workstation)         |
| Kubernetes            | EKS 1.32, kubectl, k3d                    |
| Secure Platform       | UDS Core, UDS CLI, Zarf                   |
| Identity & Mesh       | Keycloak, Istio                           |
| Observability         | Grafana, Prometheus, Loki                 |
| Policy / Automation   | Pepr                                      |
| CI / Repo             | GitHub                                    |

---

## What Was Implemented

### 1. Infrastructure (Terraform)
- Fully declarative EKS cluster + VPC
- Repeatable `terraform apply` / `terraform destroy` workflow
- Clean state management and modular structure

### 2. UDS Core Experience
- Successfully deployed the official **k3d-core-demo** on an EC2 instance
- Observed the full platform stack (Istio, Keycloak, Grafana, Pepr, monitoring, etc.)
- Explored the challenges of running the demo remotely (SSO redirects to `*.uds.dev` domains)
- Documented the production path requirements for deploying UDS Core on a real EKS cluster (external Postgres, S3 for Loki/Velero, DNS, TLS, registry authentication)

### 3. Operational Learning
- Difference between local demo bundles and production-style UDS bundles
- Importance of external services for a hardened platform
- Practical experience with `uds create`, `uds deploy`, and `uds zarf connect`

---

## Repository Structure
.
├── terraform/                 # EKS + VPC Infrastructure as Code
│   ├── main.tf
│   ├── providers.tf
│   ├── variables.tf
│   └── ...
├── uds-bundle.yaml            # Attempted production-style bundle for EKS
├── docs/                      # Architecture notes & lessons learned
└── README.md


---

## How to Recreate

### Prerequisites
- AWS account + credentials
- Terraform ≥ 1.5
- AWS CLI, kubectl, UDS CLI
- Sufficient EC2 resources if running the k3d demo (recommended ≥ 16 GB RAM)

### 1. Provision EKS
```bash
cd terraform
terraform init
terraform apply
aws eks update-kubeconfig --name uds-analytics-demo --region us-east-2
kubectl get nodes