# Caprivax Enterprise Platform 🏢
### Multi-Tenant GCP Orchestration with Terraform & Zero-Trust Governance

![Terraform](https://img.shields.io/badge/terraform-%235835CC.svg?style=for-the-badge&logo=terraform&logoColor=white)
![GCP](https://img.shields.io/badge/GoogleCloud-%234285F4.svg?style=for-the-badge&logo=google-cloud&logoColor=white)
![Security](https://img.shields.io/badge/Security-Zero--Trust-green.svg?style=for-the-badge)
![FinOps](https://img.shields.io/badge/FinOps-Cost--Optimized-blue.svg?style=for-the-badge)

## 📖 Overview
This project implements a production-grade, multi-tenant infrastructure on Google Cloud Platform (GCP). It utilizes a **Decoupled Modular Architecture** to manage Development, Staging, and Production environments with strict adherence to **Zero-Trust** security principles.

---

## 🛠️ Key Technical Features
* **Modular Design:** Reusable `standard-vm` child module for consistent, rapid deployments.
* **Zero-Trust Connectivity:** Complete removal of Public IPs in Staging/Prod; management via **Identity-Aware Proxy (IAP)**.
* **Outbound Governance:** **Cloud NAT** gateways for private instances to perform secure updates and patching.
* **State Management:** Remote state storage in **GCS** with **Terraform Workspaces** for environment isolation.
* **FinOps Integration:** Automated resource labeling and Spot Instance utilization for granular cost tracking.

---

## 📂 Project Structure
```text
capx-platform-infrastructure/
├── Modules/
│   └── standard-vm/            # The Blueprint (Compute, IAP Firewalls, Startup Logic)
├── Root-module/                # The Orchestrator (Environment Composition)
│   ├── dev.auto.tfvars         # Cost-optimized (Spot instances, Public IP)
│   ├── staging.auto.tfvars     # Pre-prod validation (Private IP, Cloud NAT)
│   └── prod.auto.tfvars        # Zero-Trust hardened (Private IP, HA)
└── terraform-pipelines/        # Automated deployment manifests
🛡️ Zero-Trust Security Matrix
Feature           Development           Staging       Production
Public IP         Enabled (Speed)       Disabled      Disabled
Inbound Access    Direct SSH / IAP      IAP Only      IAP Only
Outbound Access   Direct IGW            Cloud NAT     Cloud NAT
Attack Surface    Exposed               Hidden        Hidden

🚀 Deployment Runbook
1. Initialize Project APIs
API enablement is project-scoped. Run this to ensure IAP and Compute services are ready across the promotion path:
Bash

PROJECTS=("caprivax-dev-infra" "caprivax-stg-infra" "caprivax-prd-infra")
for PROJECT in "${PROJECTS[@]}"; do
  echo "Enabling APIs for $PROJECT..."
  gcloud services enable compute.googleapis.com iap.googleapis.com --project=$PROJECT
done

2. Environment Lifecycle (Workspaces)
Bash

cd Root-module
terraform init
terraform workspace select staging || terraform workspace new staging

# Plan and Apply
terraform plan -var-file="staging.auto.tfvars" -out=staging.tfplan
terraform apply "staging.tfplan"

3. Secure Service Verification (IAP Tunneling)
Since instances are private, use an IAP TCP tunnel to verify services like Nginx:

Bash

# Terminal 1: Open the Tunnel
gcloud compute start-iap-tunnel [VM_NAME] 80 --local-host-port=localhost:8080

# Terminal 2: Test the Service
curl -I http://localhost:8080

♻️ Decommissioning Protocol
To prevent orphaned resources and billing leaks, always destroy in Reverse Promotion Order:

Production: terraform workspace select prod && terraform destroy -var-file="prod.auto.tfvars"

Staging: terraform workspace select staging && terraform destroy -var-file="staging.auto.tfvars"

Development: terraform workspace select dev && terraform destroy -var-file="dev.auto.tfvars"

👤 Author
Marcel Owhonda - Cloud & DevOps Engineer
- GitHub: [@Marcel2tight](https://github.com/Marcel2tight)
- LinkedIn: [Marcel Owhonda](https://www.linkedin.com/in/marcel-owhonda-devops)
This project was built as a capstone for demonstrating advanced expertise in GCP Cloud Engineering and DevOps Automation.