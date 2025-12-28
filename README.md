Caprivax Enterprise Platform Infrastructure
GCP Multi-Environment Orchestration with Terraform & Zero-Trust Security

📖 Overview
This project implements a production-grade, multi-tenant infrastructure on Google Cloud Platform (GCP). It utilizes a Decoupled Modular Architecture to manage Development, Staging, and Production environments with strict Zero-Trust security principles.
🛠️ Key Technical Features
1. Modular Design: Reusable standard-vm child module for consistent deployments.
2. Zero-Trust Connectivity: Complete removal of Public IPs in Staging/Prod; management via Identity-Aware Proxy (IAP).
3. Outbound Governance: Cloud NAT gateways for private instances to perform secure updates/patching.
4. State Management: Remote state storage in GCS with Terraform Workspaces for environment isolation.
5. FinOps Integration: Automated resource labeling for granular cost tracking.

📂 Project Structure

capx-platform-infrastructure/
├── modules/
│   └── standard-vm/          # The Blueprint (Compute, IAP Firewalls, Startup Logic)
└── root-module/              # The Orchestrator (Environment Composition)
    ├── main.tf               # Root composition
    ├── dev.auto.tfvars       # Cost-optimized (Spot instances, Public IP)
    ├── staging.auto.tfvars   # Pre-prod validation (Private IP, Cloud NAT)
    └── prod.auto.tfvars      # Zero-Trust hardened (Private IP, High-Availability)

🚀 Deployment Runbook
1. Initialize Project APIs
API enablement is project-scoped. Run this to ensure IAP and Compute services are ready across the promotion path:

PROJECTS=("caprivax-dev-platform-infra" "caprivax-stging-platform-infra" "caprivax-prod-platform-infra")

for PROJECT in "${PROJECTS[@]}"; do
  echo "Enabling APIs for $PROJECT..."
  gcloud services enable compute.googleapis.com iap.googleapis.com --project=$PROJECT
done

2. Environment Lifecycle (Using Workspaces)
cd root-module

# Initialize and select environment
terraform init
terraform workspace select staging || terraform workspace new staging

# Plan and Apply
terraform plan -var-file="staging.auto.tfvars" -out=staging.tfplan
terraform apply "staging.tfplan"

3. Secure Service Verification (IAP Tunneling)Since instances are private, use an IAP TCP tunnel to verify services like Nginx:
# Terminal 1: Open the Tunnel
gcloud compute start-iap-tunnel [VM_NAME] 80 --local-host-port=localhost:8080

# Terminal 2: Test the Service
curl -I http://localhost:8080

🛡️ Zero-Trust Security Matrix
Feature            Development        Staging     Production
Public IP          Enabled(Speed)     Disabled    Disabled
Inbound Access     Direct SSH /IAP    IAP Only    IAP Only
Outbound Access    DirectIGW          CloudNAT    CloudNAT
Attack Surface     Exposed            Hidden      Hidden

Decommissioning Protocol
To prevent orphaned resources and billing leaks, always destroy in Reverse Promotion Order:
1. Production: terraform workspace select prod && terraform destroy -var-file="prod.auto.tfvars"
2. Staging: terraform workspace select staging && terraform destroy -var-file="staging.auto.tfvars"
3. Development: terraform workspace select dev && terraform destroy -var-file="dev.auto.tfvars"