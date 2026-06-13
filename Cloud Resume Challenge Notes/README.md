# Cloud Resume Challenge — Portfolio Infrastructure

**Project:** therossfisher-portfolio  
**GitHub:** github.com/therossfisher/therossfisher-portfolio  
**Started:** June 2026  
**Status:** In Progress

---

## What This Is

Building a professional portfolio hosted on Google Cloud Platform using Infrastructure as Code. The goal is two things:

1. Have a live resume and blog at therossfisher.xyz to send to employers
2. Demonstrate cloud and DevOps skills that complement the security background

This is a recognized industry challenge called the Cloud Resume Challenge — using it as a framework to build something real rather than a toy project.

---

## Architecture

```
therossfisher.xyz (registered at Hostinger)
        ↓
GCP Cloud DNS (manages subdomains)
        ↓
GCS Buckets (static site hosting)
├── resume.therossfisher.xyz
├── blog.therossfisher.xyz
└── dashboard.therossfisher.xyz (existing honeypot)
```

**Tools:**
- **Terraform** — provisions GCP infrastructure (buckets, DNS, IAM)
- **Ansible** — deploys content to the buckets
- **gcloud CLI** — authentication and project management
- **GCP Cloud Storage** — static website hosting
- **GCP Cloud DNS** — DNS management

---

## Why These Tools

**Terraform** — Industry standard Infrastructure as Code tool. Defines infrastructure in code files that can be version controlled, reviewed, and reproduced. Running `terraform apply` creates or updates all resources. Running `terraform destroy` tears everything down. Every change is tracked in git.

**Ansible** — Configuration management and deployment automation. Where Terraform handles infrastructure (what exists), Ansible handles configuration and deployment (what's on it). Used here to upload HTML/CSS files to GCS buckets.

**Why not just use the GCP console?** — Clicking through a UI leaves no record, can't be reproduced, and doesn't demonstrate engineering skills. Code-based infrastructure is what employers want to see.

---

## Project Structure

```
~/projects/resume-site/
├── .gitignore
├── README.md
├── terraform/
│   ├── provider.tf      # GCP provider and auth config
│   ├── variables.tf     # Input variable definitions
│   ├── terraform.tfvars # Actual values (NOT in git)
│   ├── main.tf          # Resource definitions (buckets, DNS)
│   └── outputs.tf       # Values output after apply
├── ansible/
│   ├── deploy.yml       # Deployment playbook
│   └── inventory/
├── resume/
│   ├── index.html
│   └── style.css
└── blog/
    └── index.html
```

---

## Terraform Files Explained

### provider.tf
Tells Terraform to use the Google Cloud provider plugin. Like installing a driver — Terraform itself doesn't know about GCP, the provider adds that capability. Downloads the plugin on `terraform init`.

### variables.tf
Declares what inputs the configuration accepts — project ID, region, domain. Defines the shape of inputs without setting actual values. Safe to commit to git.

### terraform.tfvars
Sets the actual values for variables — real project ID, real domain. **Never commit this file.** Added to .gitignore. If it contained API keys or secrets, pushing it to GitHub would be a security incident.

### main.tf (coming)
The actual infrastructure — GCS buckets, DNS zone, DNS records, IAM permissions. This is where the real work happens.

---

## Terraform Commands Reference

```bash
terraform init      # Download providers, initialize backend
terraform plan      # Preview what will be created/changed/destroyed
terraform apply     # Actually create/update infrastructure
terraform destroy   # Tear everything down
terraform output    # Show output values after apply
terraform fmt       # Format code consistently
terraform validate  # Check for syntax errors
```

**Always run `plan` before `apply`** — review what's going to change before committing.

---

## GCP Setup

**Project ID:** project-472a5ff8-fda3-4adf-a97  
**Account:** fisher.ross1776@gmail.com  
**Region:** us-central1  
**Free credit:** $300 / 90 days activated

**APIs enabled:**
- storage.googleapis.com
- dns.googleapis.com
- compute.googleapis.com

**Estimated monthly cost after free credit:**
- Cloud DNS zone: $0.20/month
- GCS storage: ~$0.00 (site is tiny)
- Bandwidth: ~$0.00 (low traffic)
- Total: ~$0.20-0.50/month

---

## DNS Flow

1. Domain registered at Hostinger
2. Terraform creates a Cloud DNS managed zone for therossfisher.xyz
3. Terraform outputs the GCP nameservers
4. Update Hostinger to point to GCP nameservers
5. GCP Cloud DNS handles all subdomain routing

---

## .gitignore Entries

```
terraform/.terraform/
terraform/.terraform.lock.hcl
terraform/terraform.tfvars
terraform/*.tfstate
terraform/*.tfstate.backup
*.pem
*.key
secrets/
```

---

## Progress Log

- [x] GCP account created and free credit activated
- [x] gcloud CLI installed and authenticated
- [x] Terraform and Ansible installed
- [x] Project directory structure created
- [x] GitHub repo created (therossfisher-portfolio)
- [x] Terraform provider.tf, variables.tf, terraform.tfvars created
- [x] terraform init successful — Google provider v5.45.2 installed
- [x] .gitignore created
- [ ] main.tf — GCS buckets and DNS zone
- [ ] outputs.tf — nameservers and bucket URLs
- [ ] Update Hostinger nameservers
- [ ] Resume HTML/CSS built
- [ ] Ansible playbook for deployment
- [ ] Blog index page
- [ ] SSL/HTTPS configured
- [ ] Everything live and working

---

## Skills Demonstrated

This project shows:
- Infrastructure as Code (Terraform)
- Cloud platform experience (GCP)
- Configuration management (Ansible)
- DNS management
- Static site hosting
- Version controlled infrastructure
- Security-conscious practices (secrets out of git)

All relevant for Network Security Engineer and cloud security roles.
