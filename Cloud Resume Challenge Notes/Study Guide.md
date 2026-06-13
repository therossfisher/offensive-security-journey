# Cloud Resume Challenge — AWS Study Guide

**Project:** Cloud Resume Challenge (AWS)  
**Goal:** Full stack serverless resume site with CI/CD pipeline  
**Status:** GCP exploration complete — starting AWS build  
**Reference:** cloudresumechallenge.dev/docs/the-challenge/aws

---

## What You're Building

```
Browser → CloudFront (HTTPS/CDN) → S3 (static site)
                                         ↓
                              JavaScript visitor counter
                                         ↓
                         API Gateway → Lambda → DynamoDB
                                         ↑
                              GitHub Actions CI/CD
                                    deploys on push
```

**Every piece matters:**
- **S3** — stores your HTML/CSS/JS files
- **CloudFront** — CDN that serves them over HTTPS with your custom domain
- **Route53** — DNS routing yourname.com → CloudFront
- **API Gateway** — HTTP endpoint the counter JavaScript calls
- **Lambda** — serverless Python function that reads/writes the counter
- **DynamoDB** — NoSQL database storing the visitor count
- **Terraform** — provisions all of the above as code
- **GitHub Actions** — deploys automatically when you push to main

---

## Why AWS Instead of GCP

GCP requires a load balancer (~$18-20/month) to serve a custom domain over HTTPS from GCS. AWS S3 + CloudFront handles this natively with no load balancer — cost is roughly $0.50-1.00/month after free tier. The Cloud Resume Challenge was designed for AWS.

---

## What You Already Know

- Terraform concepts (provider, variables, plan, apply, destroy, outputs, tfvars, gitignore)
- DNS management (nameservers, CNAME, A records, TTL, /etc/hosts)
- Git and GitHub (clone, push, gitsave alias)
- Linux CLI
- Networking fundamentals at carrier level
- Python awareness (not fluent yet — see study path)

---

## The Full Requirements Checklist

- [ ] Resume written in HTML
- [ ] Styled with CSS
- [ ] Deployed as static website on S3
- [ ] HTTPS via CloudFront
- [ ] Custom DNS domain pointing to CloudFront
- [ ] JavaScript visitor counter on the page
- [ ] Counter data stored in DynamoDB
- [ ] API (API Gateway + Lambda) reads/writes the counter
- [ ] Lambda written in Python
- [ ] Tests written for the Python code
- [ ] Infrastructure defined in Terraform (no manual console clicks)
- [ ] CI/CD via GitHub Actions — push to main triggers deploy
- [ ] Backend and frontend in separate GitHub repos

---

## Study Path

### Week 1 — AWS Foundations

**Goal:** Understand every service you'll use before touching code.

**Resource:** AWS Cloud Practitioner Essentials  
**URL:** skillbuilder.aws (free, no account needed)  
**Time:** ~6 hours

Focus on these services specifically:
- S3 — object storage, static website hosting, bucket policies
- CloudFront — CDN, distributions, origins, HTTPS certificates
- Route53 — hosted zones, A records, alias records
- Lambda — serverless functions, triggers, execution role
- DynamoDB — tables, partition keys, read/write capacity
- API Gateway — REST APIs, endpoints, Lambda integration
- IAM — roles, policies, least privilege

**Don't skip IAM.** Every permission error you'll hit comes from misconfigured IAM. Understanding roles and policies upfront saves hours of debugging.

---

### Week 2 — Python Basics

**Goal:** Understand enough Python to write and modify the Lambda function.

**Resource:** Automate the Boring Stuff with Python  
**URL:** automatetheboringstuff.com (completely free)  
**Chapters needed:** 1-6 only

Chapter map:
- Ch 1 — Python basics, expressions, data types
- Ch 2 — Flow control (if/else, loops)
- Ch 3 — Functions
- Ch 4 — Lists
- Ch 5 — Dictionaries (critical — DynamoDB responses are dicts)
- Ch 6 — String manipulation

**The Lambda function you're building:**

```python
import boto3
import json

def lambda_handler(event, context):
    dynamodb = boto3.resource('dynamodb')
    table = dynamodb.Table('visitor-counter')
    
    response = table.update_item(
        Key={'id': 'visitors'},
        UpdateExpression='ADD visits :1',
        ExpressionAttributeValues={':1': 1},
        ReturnValues='UPDATED_NEW'
    )
    
    return {
        'statusCode': 200,
        'headers': {
            'Access-Control-Allow-Origin': '*',
            'Content-Type': 'application/json'
        },
        'body': json.dumps({'visits': int(response['Attributes']['visits'])})
    }
```

Once you finish chapters 1-6, read through this function line by line. You should be able to explain what every line does.

**boto3** is the AWS SDK for Python — it's how Python talks to AWS services. Read the boto3 DynamoDB quickstart at boto3.amazonaws.com/v1/documentation/api/latest/guide/dynamodb.html

---

### Week 3 — Terraform on AWS

**Goal:** Provision all AWS infrastructure as code.

**Resource:** Official Terraform AWS tutorials  
**URL:** developer.hashicorp.com/terraform/tutorials/aws-get-started  
**Also:** TechWorld with Nana — Terraform tutorial on YouTube

**AWS provider vs GCP provider — what changes:**

```hcl
# GCP provider (what you used)
provider "google" {
  project = var.project_id
  region  = var.region
}

# AWS provider (what you'll use)
provider "aws" {
  region = var.region
}
```

Resource names change but the pattern is identical:
- `google_storage_bucket` → `aws_s3_bucket`
- `google_dns_managed_zone` → `aws_route53_zone`
- `google_dns_record_set` → `aws_route53_record`

New resources you haven't used yet:
- `aws_cloudfront_distribution`
- `aws_lambda_function`
- `aws_dynamodb_table`
- `aws_apigatewayv2_api`
- `aws_iam_role`
- `aws_acm_certificate`

**Key difference from GCP:** AWS authentication uses `~/.aws/credentials` file instead of gcloud token. Install AWS CLI and run `aws configure`.

---

### Week 4 — GitHub Actions

**Goal:** Write a CI/CD pipeline that deploys on every push to main.

**Resource:** GitHub Actions documentation  
**URL:** docs.github.com/en/actions  
**Also:** TechWorld with Nana — CI/CD pipeline tutorial on YouTube

**What a basic deploy workflow looks like:**

```yaml
name: Deploy Resume

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v2
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: us-east-1
      
      - name: Deploy to S3
        run: aws s3 sync ./resume s3://your-bucket-name --delete
      
      - name: Invalidate CloudFront cache
        run: |
          aws cloudfront create-invalidation \
            --distribution-id ${{ secrets.CLOUDFRONT_DISTRIBUTION_ID }} \
            --paths "/*"
```

Key concepts:
- `on: push` — triggers the workflow
- `secrets` — GitHub stores your AWS keys, never in code
- `aws s3 sync` — uploads changed files only
- CloudFront invalidation — clears cached files so changes show immediately

---

### Week 5 — Build It

Work through the challenge in this order:

1. Set up AWS account and IAM user with appropriate permissions
2. Buy or transfer domain to Route53 (or keep at Hostinger and update nameservers)
3. Create S3 bucket, upload resume HTML, enable static hosting
4. Request SSL certificate in ACM (us-east-1 region required for CloudFront)
5. Create CloudFront distribution pointing to S3
6. Create Route53 records pointing domain to CloudFront
7. Verify resume loads at your domain over HTTPS
8. Create DynamoDB table with partition key `id`
9. Write Lambda function, test locally
10. Create API Gateway HTTP API, integrate with Lambda
11. Add visitor counter JavaScript to resume HTML
12. Write Python tests with pytest
13. Convert all of the above to Terraform
14. Set up GitHub Actions for frontend and backend separately
15. Push, verify pipeline runs, verify site deploys

---

## AWS Services — Quick Reference

| Service | What It Does | Monthly Cost |
|---|---|---|
| S3 | Stores HTML/CSS/JS files | ~$0.00 (tiny files) |
| CloudFront | CDN + HTTPS | Free tier generous |
| Route53 | DNS | $0.50/hosted zone |
| ACM | SSL certificate | Free |
| Lambda | Runs Python counter function | Free tier: 1M requests/month |
| DynamoDB | Stores visitor count | Free tier: 25GB |
| API Gateway | HTTP endpoint for counter | Free tier: 1M calls/month |

**Total realistic cost: $0.50-1.00/month** after free tier. Under $5/year.

---

## Repository Structure

Two separate repos as the challenge requires:

```
therossfisher-resume-frontend/
├── resume/
│   ├── index.html
│   └── style.css
├── terraform/
│   ├── main.tf
│   ├── variables.tf
│   ├── outputs.tf
│   └── provider.tf
└── .github/
    └── workflows/
        └── deploy.yml

therossfisher-resume-backend/
├── lambda/
│   ├── lambda_function.py
│   └── test_lambda.py
├── terraform/
│   ├── main.tf
│   ├── variables.tf
│   └── provider.tf
└── .github/
    └── workflows/
        └── deploy.yml
```

---

## GCP Work — What It Proved

The GCP exploration wasn't wasted. It demonstrated:
- Terraform provider configuration and authentication
- DNS zone management and nameserver delegation
- GCS bucket creation and IAM policies
- Real infrastructure debugging under pressure
- Understanding of domain verification and DNS propagation

Document this in your GitHub README as "GCP Infrastructure Exploration" — it's legitimate work that shows cloud breadth.

---

## Progress Tracker

### Study
- [ ] AWS Cloud Practitioner Essentials (skillbuilder.aws)
- [ ] Automate the Boring Stuff chapters 1-6
- [ ] Terraform AWS tutorials (official docs)
- [ ] GitHub Actions quickstart
- [ ] Read full AWS challenge requirements (cloudresumechallenge.dev)

### Build
- [ ] AWS account created
- [ ] AWS CLI installed and configured
- [ ] S3 bucket created, resume uploaded
- [ ] SSL certificate requested in ACM
- [ ] CloudFront distribution created
- [ ] Custom domain pointing to CloudFront over HTTPS
- [ ] DynamoDB table created
- [ ] Lambda function written and tested
- [ ] API Gateway configured
- [ ] Visitor counter JavaScript added to resume
- [ ] Python tests written with pytest
- [ ] All infrastructure in Terraform
- [ ] Frontend GitHub Actions pipeline working
- [ ] Backend GitHub Actions pipeline working
- [ ] Full end-to-end test — push triggers deploy, counter increments

---

## Key Resources

| Resource | URL | Cost |
|---|---|---|
| AWS Skill Builder | skillbuilder.aws | Free |
| Automate the Boring Stuff | automatetheboringstuff.com | Free |
| Terraform AWS tutorials | developer.hashicorp.com/terraform/tutorials/aws-get-started | Free |
| GitHub Actions docs | docs.github.com/en/actions | Free |
| TechWorld with Nana | youtube.com/@TechWorldwithNana | Free |
| boto3 DynamoDB docs | boto3.amazonaws.com | Free |
| Cloud Resume Challenge | cloudresumechallenge.dev/docs/the-challenge/aws | Free |
| Official challenge blog posts | Many on dev.to — search "cloud resume challenge AWS" | Free |
