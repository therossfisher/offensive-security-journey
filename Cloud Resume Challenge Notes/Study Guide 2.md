# AWS SAA + Cloud Resume Challenge — Parallel Study Guide

**Approach:** Learn a service → immediately build that piece → move to next  
**Cert target:** AWS Solutions Architect Associate (SAA-C03)  
**Project target:** Cloud Resume Challenge (AWS)  
**Primary free resource:** Andrew Brown / ExamPro on YouTube

---

## The Core Principle

Don't study everything then build. Study one service, build that piece, repeat. By the time you finish the challenge you'll have hands-on experience with every service on the SAA exam. The exam will feel like a review.

---

## Before You Start

**Create AWS account:**
- aws.amazon.com → Create account
- Add credit card (required, won't be charged for this project)
- Activate free tier

**Read the challenge requirements once:**
- cloudresumechallenge.dev/docs/the-challenge/aws

**Install AWS CLI on Kali:**
```bash
sudo apt install awscli -y
aws configure
# Enter Access Key ID, Secret Access Key, region (us-east-1), output format (json)
```

**Create IAM user for CLI (never use root account for CLI):**
- AWS Console → IAM → Users → Create user
- Attach policy: AdministratorAccess (tighten later)
- Create access key → download credentials
- Run `aws configure` with those credentials

---

## Study + Build Sequence

### Phase 1 — IAM (Day 1-2)

**Why first:** Every AWS service uses IAM. Every permission error comes from here.

**Watch:**
- YouTube: search `AWS IAM ExamPro Andrew Brown`

**Key concepts:**
- Users, Groups, Roles, Policies
- Least privilege principle
- Trust policies (what allows Lambda to talk to DynamoDB)
- Access keys vs role-based access

No build step — you'll apply IAM as you build each service.

---

### Phase 2 — S3 (Day 2-3)

**Watch:**
- YouTube: search `AWS S3 ExamPro Andrew Brown`
- Focus: bucket creation, static website hosting, bucket policies, public access

**Build:**
1. Create S3 bucket in AWS Console (us-east-1)
2. Enable static website hosting
3. Upload resume index.html
4. Configure bucket policy for public read
5. Verify it loads at the S3 website endpoint URL

```bash
aws s3 cp ~/projects/resume-site/resume/index.html s3://YOUR-BUCKET-NAME/
```

**SAA exam topics:** Storage classes, lifecycle policies, versioning, cross-region replication

---

### Phase 3 — ACM + CloudFront (Day 3-5)

**Watch:**
- YouTube: search `AWS CloudFront ExamPro Andrew Brown`

**Critical gotcha:** ACM certificates must be requested in us-east-1 for CloudFront — no exceptions.

**Build:**
1. Request SSL certificate in ACM (us-east-1 — critical)
2. Validate via DNS (add CNAME record in Hostinger)
3. Create CloudFront distribution with S3 as origin
4. Configure OAC so S3 only serves through CloudFront
5. Verify resume loads over HTTPS via CloudFront URL

**SAA exam topics:** CDN concepts, edge locations, cache behaviors, signed URLs

---

### Phase 4 — Route53 (Day 5-6)

**Watch:**
- YouTube: search `AWS Route53 ExamPro Andrew Brown`

**Key concepts:**
- Alias records point to AWS resources (CloudFront, S3) — use these not CNAMEs
- Hosted zones

**Build:**
1. Create hosted zone for therossfisher.xyz in Route53
2. Update Hostinger nameservers to Route53 nameservers
3. Create Alias record: therossfisher.xyz → CloudFront distribution
4. Verify resume loads at therossfisher.xyz over HTTPS

**SAA exam topics:** All routing policies, health checks, private hosted zones

---

### Phase 5 — DynamoDB (Day 6-7)

**Watch:**
- YouTube: search `AWS DynamoDB ExamPro Andrew Brown`

**Build:**
1. Create DynamoDB table named `visitor-counter`
2. Partition key: `id` (String)
3. Create one item manually: `{"id": "visitors", "visits": 0}`
4. Verify in console

**SAA exam topics:** Consistency models, DAX, streams, global tables

---

### Phase 6 — Python + Lambda (Day 7-10)

**Study Python first (2-3 days):**
- automatetheboringstuff.com — chapters 1-6 only
- Chapter 5 (dictionaries) is critical — DynamoDB responses are dictionaries

**Watch:**
- YouTube: search `AWS Lambda ExamPro Andrew Brown`

**The Lambda function:**

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

**Line by line:**
- `boto3.resource('dynamodb')` — connects to DynamoDB using Lambda's IAM role
- `table.update_item(...)` — finds id='visitors' and adds 1 to visits
- `ReturnValues='UPDATED_NEW'` — returns the new value after update
- The return dict — API Gateway requires statusCode + headers + body exactly
- `Access-Control-Allow-Origin: *` — allows browser to call this API

**Write tests:**
```python
import json
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
        body = json.loads(result['body'])
        assert body['visits'] == 5
```

```bash
pip3 install pytest --break-system-packages
pytest test_lambda.py -v
```

**SAA exam topics:** Serverless concepts, execution roles, cold starts, concurrency

---

### Phase 7 — API Gateway (Day 10-11)

**Watch:**
- YouTube: search `AWS API Gateway ExamPro Andrew Brown`

**Build:**
1. Create HTTP API (not REST API — simpler and cheaper)
2. Create GET route: `/visits`
3. Integrate with Lambda function
4. Enable CORS (allow therossfisher.xyz as origin)
5. Deploy and test:

```bash
curl https://YOUR-API-ID.execute-api.us-east-1.amazonaws.com/visits
# Should return: {"visits": 1}
```

---

### Phase 8 — Visitor Counter JavaScript (Day 11-12)

**Add to resume HTML:**

```html
<p>Visitors: <span id="visitor-count">...</span></p>

<script>
async function updateVisitorCount() {
    try {
        const response = await fetch('https://YOUR-API-ENDPOINT/visits');
        const data = await response.json();
        document.getElementById('visitor-count').textContent = data.visits;
    } catch (error) {
        console.error('Could not fetch visitor count:', error);
    }
}
updateVisitorCount();
</script>
```

---

### Phase 9 — Terraform (Day 12-16)

**Watch:**
- YouTube: search `Terraform AWS TechWorld with Nana`
- Official: developer.hashicorp.com/terraform/tutorials/aws-get-started

**AWS authentication is cleaner than GCP:**
```bash
aws configure
# Terraform uses ~/.aws/credentials automatically — no token export needed
```

**Resources to define in Terraform:**
```
aws_s3_bucket
aws_s3_bucket_website_configuration
aws_s3_bucket_policy
aws_cloudfront_distribution
aws_cloudfront_origin_access_control
aws_acm_certificate
aws_acm_certificate_validation
aws_route53_zone
aws_route53_record
aws_dynamodb_table
aws_lambda_function
aws_iam_role
aws_iam_role_policy
aws_apigatewayv2_api
aws_apigatewayv2_integration
aws_apigatewayv2_route
aws_apigatewayv2_stage
aws_lambda_permission
```

Destroy manually created resources first, then rebuild with Terraform.

---

### Phase 10 — GitHub Actions CI/CD (Day 16-18)

**Two separate repos:**
- `therossfisher-resume-frontend` — HTML/CSS/JS + frontend Terraform
- `therossfisher-resume-backend` — Lambda + tests + backend Terraform

**Frontend workflow:**
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
        run: aws s3 sync ./resume s3://${{ secrets.S3_BUCKET }} --delete
      - name: Invalidate CloudFront cache
        run: |
          aws cloudfront create-invalidation \
            --distribution-id ${{ secrets.CLOUDFRONT_ID }} \
            --paths "/*"
```

**Backend workflow (tests run before deploy):**
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
      - run: pip install pytest boto3
      - run: pytest tests/ -v
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
          zip function.zip lambda_function.py
          aws lambda update-function-code \
            --function-name visitor-counter \
            --zip-file fileb://function.zip
```

**Add secrets in GitHub:**
Repository → Settings → Secrets → Actions → Add:
- AWS_ACCESS_KEY_ID
- AWS_SECRET_ACCESS_KEY
- S3_BUCKET
- CLOUDFRONT_ID

---

## SAA Exam Topics Beyond the Challenge

Study after the challenge is complete:
- EC2 instance types, auto scaling, load balancers
- VPC, subnets, security groups, NACLs
- RDS, Aurora, ElastiCache
- SQS, SNS, EventBridge
- ECS, EKS
- Disaster recovery strategies
- Well-Architected Framework

**Practice exams:** Tutorials Dojo — tutorialsdojo.com — $15. Do these until scoring 80%+ before booking the exam.

---

## Key Resources

| Resource | URL | Cost |
|---|---|---|
| Andrew Brown SAA Course | YouTube — search "AWS SAA ExamPro Andrew Brown" | Free |
| AWS Skill Builder | skillbuilder.aws | Free tier |
| Automate the Boring Stuff | automatetheboringstuff.com | Free |
| Terraform AWS tutorials | developer.hashicorp.com/terraform/tutorials/aws-get-started | Free |
| GitHub Actions docs | docs.github.com/en/actions | Free |
| Cloud Resume Challenge | cloudresumechallenge.dev/docs/the-challenge/aws | Free |
| Tutorials Dojo practice exams | tutorialsdojo.com | $15 |
| Adrian Cantrill SAA course | cantrill.io | $40 one-time |

---

## Estimated Cost After Free Tier

| Service | Monthly Cost |
|---|---|
| S3 | ~$0.00 |
| CloudFront | ~$0.00 |
| Route53 | $0.50/hosted zone |
| ACM | Free |
| Lambda | Free tier: 1M requests/month |
| DynamoDB | Free tier: 25GB |
| API Gateway | Free tier: 1M calls/month |
| **Total** | **~$0.50-1.00/month** |

---

## Progress Tracker

### Study
- [ ] AWS account created and CLI configured
- [ ] IAM section watched and understood
- [ ] S3 section watched
- [ ] CloudFront + ACM section watched
- [ ] Route53 section watched
- [ ] DynamoDB section watched
- [ ] Lambda section watched
- [ ] API Gateway section watched
- [ ] Python chapters 1-6 complete
- [ ] Terraform AWS tutorials complete

### Build
- [ ] S3 bucket created, resume accessible via S3 URL
- [ ] ACM certificate requested and validated
- [ ] CloudFront distribution created with OAC
- [ ] Resume loads over HTTPS via CloudFront URL
- [ ] Route53 hosted zone created
- [ ] Nameservers updated
- [ ] Resume loads at therossfisher.xyz over HTTPS
- [ ] DynamoDB table created with initial item
- [ ] Lambda function written and tested
- [ ] Python tests written and passing
- [ ] API Gateway created and tested with curl
- [ ] Visitor counter JavaScript added
- [ ] Counter increments on page load
- [ ] All infrastructure in Terraform
- [ ] Frontend GitHub Actions pipeline working
- [ ] Backend GitHub Actions pipeline working
- [ ] End-to-end: push triggers deploy, counter increments

### Exam Prep
- [ ] All SAA topics studied beyond challenge services
- [ ] Tutorials Dojo practice exams scoring 80%+
- [ ] Exam booked
- [ ] Exam passed
