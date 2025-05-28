
# 📦 Project Name: [portfolio.mkcloudai.com aws deployment]
## 📁 Project Description
Here's a full step-by-step guide to:
1.Fork a GitHub repo
2.Edit locally with VS Code
3.Push to your GitHub repo
4.Deploy to AWS (S3 + Route 53 + SSL + CloudFront)
5.Document the process

## 🚀 Architecture

- Language: Bash / VS code etc.
- AWS Services: S3 / IAM / Route 53/ CloudFront / SSL ACM etc.
- Tools: Git/GitHub Actions
- 
### Prerequisites

- AWS CLI configured
- IAM permissions for resource access
- Bash,Git,Vs Code Installed . (as applicable)


## 🛠️ Setup & Deployment

### Step 1: Fork and Clone the GitHub Repo
Go to the original GitHub repository.
Click Fork → it creates a copy under your GitHub account.
Clone it to your computer, open vs code and open terminal 
```bash
git clone https://github.com/mxkdevops/portfolio-mo.git
cd portfolio-mo
```
### Step 2: Edit Locally with VS Code
Customize the content (e.g., index.html, about.html, images, CSS).
Use Live Preview or simple local server:


### Step 3: Push Changes to Your GitHub Repo
```bash
git add .
git commit -m "Customize portfolio site"
git push origin main
```

✅ Create S3 bucket (portfolio.mkcloudai.com)

```bash
Go to AWS S3 > Create Bucket
Name: portfolio.mkcloudai.com
Block All Public Access ✅
Enable Versioning (optional)
Click Create Bucket
```

✅ Create IAM User with Programmatic Access
Go to IAM > Users > Add User

Name: github-portfolio-mo
Access type: ✅ Programmatic access

Attach Policy:
AmazonS3FullAccess
CloudFrontFullAccess

### 1. Go to IAM Console
Open the IAM Users page
Click on the user you created ( github-portfolio-mo)

### 2. Navigate to the "Security credentials" tab
Scroll down to Access keys
Click Create access key

### 3. Select the following option:
Use case: Choose
 ✅ "Command Line Interface (CLI)"
 This is the correct option for GitHub Actions and CI/CD.
Click Next.
Download credentials (Access Key ID & Secret)

## Add GitHub Repository Secrets
In your repo, go to Settings > Secrets and Variables > Actions and add:
Under the Secrets tab, click “New repository secret"
```bash
AWS_ACCESS_KEY_ID
AWS_SECRET_ACCESS_KEY
AWS_REGION = us-east-1 (or your region)
S3_BUCKET_NAME = portfolio.mkcloudai.com
CLOUDFRONT_DISTRIBUTION_ID = from CloudFront dashboard

```

### GitHub Actions CI/CD Workflow
Create .github/workflows/deploy.yml:
name: Deploy to S3 and Invalidate CloudFront
```bash
on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3

      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v3
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: ${{ secrets.AWS_REGION }}

      - name: Sync S3 (exclude README.md)
        run: |
          aws s3 sync . s3://${{ secrets.S3_BUCKET_NAME }} \
            --exclude ".git/*" --exclude "README.md" \
            --delete

      - name: Invalidate CloudFront Cache
        run: |
          aws cloudfront create-invalidation \
            --distribution-id ${{ secrets.CLOUDFRONT_DISTRIBUTION_ID }} \
            --paths "/*"



```


### Set Up CloudFront Origin Access Control (OAC)
To secure your S3 bucket and ensure only CloudFront can access it (not the public internet), set up OAC.
## Create Origin Access Control (OAC)
```bash
Go to AWS CloudFront Console

Click Security > Origin access
Click Create control setting
Use these values:
Name: OAC portfolio.mkcloudai.com
Signing behavior: Always
Signing protocol: SigV4
Origin type: S3
Click Create
```

### Create CloudFront Distribution with OAC (Origin Access Control)
```bash
Go to CloudFront > Create Distribution
Origin Domain: your S3 bucket endpoint
Access: ✅ Use Origin Access Control (OAC)
Create OAC and attach to the S3 bucket
Alternate Domain Name (CNAME): agilesynergyltd.com
SSL Certificate: select from ACM (see next step)
Save and copy the Distribution ID

```

### Apply Secure Bucket Policy to Allow CloudFront Access
Description:
 To keep your S3 bucket private while allowing CloudFront to fetch your files, apply the following policy. This ensures the public cannot directly access your bucket.
Then paste the corrected JSON from earlier.
```bash
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "AllowCloudFrontServicePrincipalReadOnly",
            "Effect": "Allow",
            "Principal": {
                "Service": "cloudfront.amazonaws.com"
            },
            "Action": "s3:GetObject",
            "Resource": "arn:aws:s3:::agilesynergyltd.com/*",
            "Condition": {
                "StringEquals": {
                    "AWS:SourceArn": "arn:aws:cloudfront::355804389681:distribution/E1W96NCO1QA5K6"
                }
            }
        }
    ]

```



✅[ Request SSL certificate in ACM (for agilesynergyltd.com)](https://docs.google.com/document/d/1JITJ2r5EPSsZdbaxmWQMmBB_GvnjvAIqItDiTdyi5Go/edit?tab=t.yqbrjyk4t996)
### Request SSL Certificate for Domain
```bash
Go to AWS Certificate Manager (in us-east-1).
Click “Request a public certificate”.
Add the domain:
agilesynergyltd.com
www.agilesynergyltd.com (optional)
Choose DNS Validation ✅.
ACM will give you CNAME record(s) to add in Route 53.
Step 5: Add CNAME Records in Route 53
Go to Route 53 → Hosted Zones → agilesynergyltd.com.
Click “Create Record”.
Type: CNAME
Name & value: Copy from ACM.
Save.

📌 Important: Don’t modify the CNAME values. ACM checks this automatically.

Step 6: Wait for Validation (Usually 5–15 min)
Once validated:
Status will change to "Issued"
You can now use this certificate in CloudFront or ALB

```


✅ [Update DNS in Route 53 with CloudFront’s domain as alias](https://docs.google.com/document/d/1JITJ2r5EPSsZdbaxmWQMmBB_GvnjvAIqItDiTdyi5Go/edit?tab=t.5aprm9n42ssb)


✅ Test live website
```bash
Visit: https://agilesynergyltd.com ✅
nslookup -type=ns agilesynergyltd.com
whois agilesynergyltd.com
ping agilesynergyltd.com
tracert agilesynergyltd.com
ipconfig /flushdns
```

✅  Add CloudFront cache invalidation to CI/CD

