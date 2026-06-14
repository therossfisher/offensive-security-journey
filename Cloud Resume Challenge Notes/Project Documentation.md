# Cloud Resume Challenge — AWS Project Documentation

**Status:** Complete (manual build + Terraform + CI/CD)  
**Live site:** https://therossfisher.xyz  
**Frontend repo:** github.com/therossfisher/therossfisher-resume-frontend  
**Backend repo:** github.com/therossfisher/therossfisher-resume-backend  
**Completed:** June 2026

---

## What Was Built

A fully serverless resume website meeting all Cloud Resume Challenge requirements:

- Static resume hosted on AWS S3
- HTTPS via CloudFront CDN
- Custom domain via Route53
- JavaScript visitor counter on the page
- Counter backed by DynamoDB (NoSQL database)
- API Gateway + Lambda (Python) serving the counter endpoint
- Python unit tests with pytest
- All infrastructure defined in Terraform
- CI/CD via GitHub Actions — push to main triggers automatic deployment

---

## Architecture

```
Browser
  ↓
CloudFront (HTTPS, CDN, caching)
  ↓
S3 Bucket (static HTML/CSS/JS)
  
  + JavaScript on page calls:
  
API Gateway → Lambda (Python) → DynamoDB
(GET /visits)   (increment)     (store count)

DNS: Route53 → therossfisher.xyz → CloudFront
```

---

## AWS Resources Created

### Frontend Infrastructure

| Resource | Name/ID | Purpose |
|---|---|---|
| S3 Bucket | therossfisher-resume | Stores resume HTML/CSS/JS |
| CloudFront Distribution | E3B8E628HFB4EI | HTTPS CDN in front of S3 |
| ACM Certificate | us-east-1 region | SSL cert for therossfisher.xyz + *.therossfisher.xyz |
| Route53 Hosted Zone | therossfisher.xyz | DNS management |
| Route53 A Record | therossfisher.xyz → CloudFront | Routes domain to CDN |
| Route53 CNAME | dashboard.therossfisher.xyz → GitHub Pages | Preserves honeypot dashboard |

### Backend Infrastructure

| Resource | Name/ID | Purpose |
|---|---|---|
| DynamoDB Table | visitor-counter | Stores visitor count |
| Lambda Function | visitor-counter | Python function that increments counter |
| IAM Role | lambda-visitor-counter-role | Gives Lambda permission to access DynamoDB |
| API Gateway | wzwr121ije | HTTP API with GET /visits route |
| API Gateway Stage | production | Deployed stage |

---

## Key Configuration Details

**CloudFront domain:** d2t0kfvtum0px.cloudfront.net  
**API endpoint:** https://YOUR-API-ID.execute-api.us-east-1.amazonaws.com/production/visits
**Route53 zone:** therossfisher.xyz  
**AWS region:** us-east-1 (all resources)  
**ACM note:** Certificate MUST be in us-east-1 for CloudFront — no exceptions  
**CloudFront hosted zone ID for aliases:** Z2FDTNDATAQYW2 (this is fixed for all CloudFront distributions)

---

## Repository Structure

### Frontend (therossfisher-resume-frontend)

```
resume/
  index.html              Resume HTML with visitor counter JavaScript
terraform/
  provider.tf             AWS provider configuration
  variables.tf            Input variables (domain, bucket name, etc.)
  main.tf                 S3, CloudFront, Route53 resources
  outputs.tf              Output values
.github/workflows/
  deploy.yml              CI/CD pipeline
.gitignore
README.md
```

### Backend (therossfisher-resume-backend)

```
lambda/
  lambda_function.py      Visitor counter Lambda function
  test_lambda.py          pytest unit tests
  function.zip            Zipped Lambda deployment package
terraform/
  provider.tf             AWS provider configuration
  main.tf                 DynamoDB, Lambda, IAM, API Gateway resources
  outputs.tf              API endpoint output
.github/workflows/
  deploy.yml              CI/CD pipeline (tests run before deploy)
.gitignore
README.md
```

---

## The Lambda Function

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

**What each part does:**
- `boto3.resource('dynamodb')` — connects to DynamoDB using Lambda's IAM role
- `table.update_item(...)` — finds the item with id='visitors' and adds 1 to visits atomically
- `ReturnValues='UPDATED_NEW'` — returns the new value after the update
- The return dict — API Gateway requires exactly this format: statusCode + headers + body
- `Access-Control-Allow-Origin: *` — allows the resume page JavaScript to call this API from any domain (CORS)

---

## The Visitor Counter JavaScript

Added to resume/index.html before closing body tag:

```javascript
async function updateVisitorCount() {
    try {
        const response = await fetch('https://YOUR-API-ID.execute-api.us-east-1.amazonaws.com/production/visits');
        const data = await response.json();
        document.getElementById('visitor-count').textContent = data.visits;
    } catch (error) {
        console.error('Could not fetch visitor count:', error);
    }
}
updateVisitorCount();
```

HTML element in footer:
```html
<span class="footer-note">Visitors: <span id="visitor-count">...</span></span>
```

---

## The Python Tests

```python
import json
import pytest
from unittest.mock import patch, MagicMock
from lambda_function import lambda_handler

def test_lambda_returns_200():
    with patch('boto3.resource') as mock_boto:
        mock_table = MagicMock()
        mock_boto.return_value.Table.return_value = mock_table
        mock_table.update_item.return_value = {
            'Attributes': {'visits': 5}
        }
        result = lambda_handler({}, {})
        assert result['statusCode'] == 200

def test_lambda_returns_visits():
    with patch('boto3.resource') as mock_boto:
        mock_table = MagicMock()
        mock_boto.return_value.Table.return_value = mock_table
        mock_table.update_item.return_value = {
            'Attributes': {'visits': 5}
        }
        result = lambda_handler({}, {})
        body = json.loads(result['body'])
        assert 'visits' in body
        assert body['visits'] == 5

def test_lambda_returns_cors_header():
    with patch('boto3.resource') as mock_boto:
        mock_table = MagicMock()
        mock_boto.return_value.Table.return_value = mock_table
        mock_table.update_item.return_value = {
            'Attributes': {'visits': 1}
        }
        result = lambda_handler({}, {})
        assert result['headers']['Access-Control-Allow-Origin'] == '*'
```

Run tests:
```bash
cd lambda
pytest test_lambda.py -v
```

---

## GitHub Actions Pipelines

### Frontend (deploy on push to main)

```yaml
name: Deploy Frontend
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
        run: aws s3 sync ./resume s3://therossfisher-resume --delete
      - name: Invalidate CloudFront cache
        run: |
          aws cloudfront create-invalidation \
            --distribution-id E3B8E628HFB4EI \
            --paths "/*"
```

### Backend (test then deploy)

```yaml
name: Deploy Backend
on:
  push:
    branches: [main]
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-python@v4
        with:
          python-version: '3.12'
      - name: Install dependencies
        run: pip install pytest boto3
      - name: Run tests
        run: pytest lambda/test_lambda.py -v
  deploy:
    needs: test
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v2
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: us-east-1
      - name: Deploy Lambda
        run: |
          cd lambda
          zip function.zip lambda_function.py
          aws lambda update-function-code \
            --function-name visitor-counter \
            --zip-file fileb://function.zip \
            --region us-east-1
```

**GitHub Secrets required in both repos:**
- `AWS_ACCESS_KEY_ID`
- `AWS_SECRET_ACCESS_KEY`

---

## DNS Configuration

**Nameservers** (set in Hostinger, pointing to Route53):
```
ns-1625.awsdns-11.co.uk
ns-680.awsdns-21.net
ns-395.awsdns-49.com
ns-1105.awsdns-10.org
```

**Route53 records:**
- `therossfisher.xyz` — A record (Alias) → CloudFront distribution
- `dashboard.therossfisher.xyz` — CNAME → therossfisher.github.io
- `_6d5e00e7ad67f3f78408d1e9d91a0f7c.therossfisher.xyz` — CNAME → ACM validation

---

## Terraform Commands Reference

```bash
# Initialize (download providers)
terraform init

# Preview changes
terraform plan

# Apply changes
terraform apply

# Import existing resource into state
terraform import RESOURCE_TYPE.NAME RESOURCE_ID

# Destroy everything
terraform destroy

# Show current state
terraform show
```

**Import examples used in this project:**
```bash
terraform import aws_s3_bucket.resume therossfisher-resume
terraform import aws_route53_zone.main ZONE_ID
terraform import aws_lambda_function.visitor_counter visitor-counter
terraform import aws_apigatewayv2_api.visitor_api API_ID
```

---

## Key AWS CLI Commands Used

```bash
# S3
aws s3 mb s3://BUCKET-NAME --region us-east-1
aws s3 website s3://BUCKET --index-document index.html --error-document 404.html
aws s3 cp file.html s3://BUCKET/
aws s3 sync ./folder s3://BUCKET --delete
aws s3api put-public-access-block --bucket BUCKET --public-access-block-configuration "BlockPublicAcls=false,..."
aws s3api put-bucket-policy --bucket BUCKET --policy file://policy.json

# ACM
aws acm request-certificate --domain-name DOMAIN --subject-alternative-names "*.DOMAIN" --validation-method DNS --region us-east-1
aws acm describe-certificate --certificate-arn ARN --region us-east-1 --query 'Certificate.Status'

# CloudFront
aws cloudfront create-distribution --distribution-config '{...}'
aws cloudfront get-distribution --id DIST_ID --query 'Distribution.Status'
aws cloudfront create-invalidation --distribution-id DIST_ID --paths "/*"

# Route53
aws route53 create-hosted-zone --name DOMAIN --caller-reference $(date +%s)
aws route53 list-hosted-zones
aws route53 list-resource-record-sets --hosted-zone-id ZONE_ID
aws route53 change-resource-record-sets --hosted-zone-id ZONE_ID --change-batch '{...}'

# Lambda
aws lambda create-function --function-name NAME --runtime python3.12 --role ARN --handler FILE.FUNCTION --zip-file fileb://function.zip
aws lambda invoke --function-name NAME output.json && cat output.json
aws lambda update-function-configuration --function-name NAME --timeout 10

# DynamoDB
aws dynamodb create-table --table-name NAME --attribute-definitions AttributeName=id,AttributeType=S --key-schema AttributeName=id,KeyType=HASH --billing-mode PAY_PER_REQUEST
aws dynamodb put-item --table-name NAME --item '{"id": {"S": "visitors"}, "visits": {"N": "0"}}'
aws dynamodb get-item --table-name NAME --key '{"id": {"S": "visitors"}}'

# API Gateway
aws apigatewayv2 create-api --name NAME --protocol-type HTTP --cors-configuration AllowOrigins="*",AllowMethods="GET"
aws apigatewayv2 create-integration --api-id ID --integration-type AWS_PROXY --integration-uri LAMBDA_ARN --payload-format-version 2.0
aws apigatewayv2 create-route --api-id ID --route-key "GET /visits" --target integrations/INTEGRATION_ID
aws apigatewayv2 create-stage --api-id ID --stage-name production --auto-deploy

# IAM
aws iam create-role --role-name NAME --assume-role-policy-document file://trust-policy.json
aws iam put-role-policy --role-name NAME --policy-name POLICY_NAME --policy-document file://policy.json
aws iam attach-role-policy --role-name NAME --policy-arn arn:aws:iam::aws:policy/service-role/AWSLambdaBasicExecutionRole
```

---

## Concepts Learned

**S3 static hosting:** S3 can serve HTML files as a website. Requires public access block disabled and a bucket policy allowing GetObject for all principals.

**CloudFront:** CDN that caches your files at edge locations worldwide. Handles HTTPS termination. Cache invalidation required after updates so visitors see new content.

**ACM:** Free SSL certificates from AWS. Must be in us-east-1 for CloudFront. Validated via DNS CNAME record.

**Route53:** AWS DNS service. Alias records point to AWS resources like CloudFront — use these instead of CNAMEs for root domains. CloudFront's fixed hosted zone ID is Z2FDTNDATAQYW2.

**Lambda:** Serverless function — runs code without managing servers. Needs an IAM execution role to access other AWS services. Default timeout is 3 seconds — increase if function is slow.

**DynamoDB:** NoSQL key-value database. Items are like JSON objects. Partition key (id) is required on every item. PAY_PER_REQUEST billing mode — only pay for what you use.

**API Gateway HTTP API:** Exposes Lambda as an HTTP endpoint. Routes map URL paths to Lambda integrations. CORS must be configured so browser JavaScript can call the API from a different domain.

**IAM:** Every AWS service interaction requires permissions. Lambda needs a trust policy (allows Lambda service to assume the role) and a permissions policy (allows specific DynamoDB actions). Least privilege — only grant what's needed.

**Terraform import:** When resources already exist, import them into Terraform state before running plan/apply. Format: `terraform import RESOURCE_TYPE.NAME RESOURCE_ID`

**GitHub Actions:** YAML workflow files in `.github/workflows/`. Triggered by push to main. `needs: test` makes deploy job wait for test job to pass first. AWS credentials stored as GitHub Secrets — never in code.

**CORS:** Cross-Origin Resource Sharing. Browsers block JavaScript from calling APIs on different domains by default. The `Access-Control-Allow-Origin: *` header tells browsers to allow requests from any domain.

**CI/CD flow:**
```
Push code → GitHub Actions triggered → Tests run → If pass: deploy to AWS → CloudFront invalidated → Live site updated
```

---

## What To Study Next

- **AWS SAA-C03** — Andrew Brown free course on YouTube covers all services used here in depth
- **Python** — automatetheboringstuff.com chapters 1-6 to understand the Lambda function properly
- **Terraform** — developer.hashicorp.com/terraform/tutorials/aws-get-started
- **GitHub Actions** — docs.github.com/en/actions
- **The blog post** — Cloud Resume Challenge asks you to write about what you learned

---

## Estimated Monthly Cost

| Service | Cost |
|---|---|
| S3 | ~$0.00 |
| CloudFront | ~$0.00 |
| Route53 | $0.50/month |
| ACM | Free |
| Lambda | Free tier |
| DynamoDB | Free tier |
| API Gateway | Free tier |
| **Total** | **~$0.50/month** |
