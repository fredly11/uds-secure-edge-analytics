# Deployment Guide

This document explains how to deploy and tear down the Secure Edge Analytics Platform components.

There are two main paths:

1. **Recommended starting point** – Official UDS Core k3d demo (full platform experience)
2. **Infrastructure path** – Terraform-managed Amazon EKS cluster

---

## Prerequisites

- AWS account with appropriate permissions
- AWS CLI configured (`aws configure`)
- Terraform ≥ 1.5
- `kubectl`
- UDS CLI (`uds`)
- Docker (required for the k3d demo)
- k3d (installed automatically or via the install script)
- An EC2 instance or local machine with at least **16 GB RAM** recommended for the full UDS Core demo

---

## 1. Deploy Infrastructure (Amazon EKS)

```bash
cd terraform

# Initialize
terraform init

# Review the plan
terraform plan

# Apply (takes 15–25 minutes)
terraform apply

```

After the apply completes:

```bash
# Update kubeconfig (adjust region if needed)
aws eks update-kubeconfig --name uds-analytics-demo --region us-east-2

# Verify nodes
kubectl get nodes
```

You should see two nodes in Ready status.

## 2. Deploy Official UDS Core Demo (Recommended)

This path gives you the complete UDS Core experience (Istio, Keycloak, Grafana, Pepr, monitoring stack, etc.).

```bash
# Make sure Docker is running
sudo systemctl start docker

# Install k3d if needed
curl -s https://raw.githubusercontent.com/k3d-io/k3d/main/install.sh | bash

# Deploy the official demo
uds deploy k3d-core-demo:latest --confirm
```

The first run typically takes 10–25 minutes while images are pulled.

### Accessing the Platform UIs

The demo uses special `*.uds.dev` domains and SSO redirects that work best when running locally.

On a remote EC2:

```bash
# Port-forward examples
kubectl port-forward -n grafana svc/grafana 3000:80
kubectl port-forward -n keycloak svc/keycloak-http 8080:8080
```

Then open from your local browser:

- Grafana → http://localhost:3000
- Keycloak → http://localhost:8080

Note: You will likely be redirected to `sso.uds.dev` / `grafana.admin.uds.dev`. These domains only resolve correctly in the local k3d environment. Full browser SSO experience is limited on a remote EC2.

You can also try:

```bash
uds zarf connect grafana
uds zarf connect keycloak
```

## 3. UDS Core on Existing EKS (Advanced / Partial)

Deploying the full production-style UDS Core onto a real EKS cluster requires additional external services:

- External PostgreSQL for Keycloak
- S3 (or S3-compatible) buckets for Loki and Velero
- Wildcard DNS + TLS certificates
- Access to the UDS / Defense Unicorns package registry

A starter `uds-bundle.yaml` is included in the repository. The typical flow is:

```bash
uds create .
uds deploy uds-bundle-*.tar.zst --confirm
```

This path was explored but not fully completed due to registry authentication and external dependency requirements. It remains a valid future improvement.

---

## Cleanup

### Destroy EKS Infrastructure

```bash
cd terraform
terraform destroy
```

### Remove k3d Demo Cluster

```bash
k3d cluster delete uds
# or whatever name the demo created
```

### Stop the EC2 Instance (cost control)

Stop (do not terminate) the EC2 instance when not in use so you only pay for EBS storage.

---

## Tips for Repeatability

- Keep the Terraform state and `uds-bundle.yaml` in Git.
- Consider creating a simple `deploy-all.sh` script that runs:
	- `terraform apply`
	- `aws eks update-kubeconfig ...`
	- `uds deploy ...`
- Always run `terraform destroy` when finished to avoid unexpected AWS charges.

---

## Known Limitations

- Full SSO-protected UIs (`*.uds.dev`) work best on a local machine.
- Production UDS Core on EKS requires external identity database, object storage, DNS, and TLS.
- Official Core packages may require registry authentication.