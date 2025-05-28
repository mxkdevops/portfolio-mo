
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
 Amazon S3-Buckets-portfolio.mkcloudai.com-Edit bucket policy
Get the policy from CloudFront-Distributions-E3PDGZ0JGLMUFB
-Edit origin

```bash
{
        "Version": "2008-10-17",
        "Id": "PolicyForCloudFrontPrivateContent",
        "Statement": [
            {
                "Sid": "AllowCloudFrontServicePrincipal",
                "Effect": "Allow",
                "Principal": {
                    "Service": "cloudfront.amazonaws.com"
                },
                "Action": "s3:GetObject",
                "Resource": "arn:aws:s3:::portfolio.mkcloudai.com/*",
                "Condition": {
                    "StringEquals": {
                      "AWS:SourceArn": "arn:aws:cloudfront::355804389681:distribution/E3PDGZ0JGLMUFB"
                    }
                }
            }
        ]
      }
```




### Request SSL Certificate for Domain
```bash
Go to AWS Certificate Manager (in us-east-1).
Click “Request a public certificate”.
Add the domain:
portfolio.mkcloudai.com

Choose DNS Validation ✅.
ACM will give you CNAME record(s) to add in Route 53.
Step 5: Add CNAME Records in Route 53
Go to Route 53 → Hosted Zones → mkcloudai.com
Click “Create Record”.
Type: CNAME
Name & value: Copy from ACM.
Save.

## 1. If your portfolio site is hosted on S3 + CloudFront
You need to add a CNAME or Alias record in Route 53 for portfolio.mkcloudai pointing to your CloudFront distribution domain (e.g., d1234abcd.cloudfront.net).

### Steps:

Open Route 53 Console
Go to your hosted zone for mkcloudai.com
Click Create Record
Set:
Record name: portfolio (so it’s portfolio.mkcloudai)
Record type: A – IPv4 address
Select Alias: Yes
Alias Target: Choose your CloudFront distribution domain from the dropdown or paste it manually

Save

📌 Important: Don’t modify the CNAME values. ACM checks this automatically.

Step 6: Wait for Validation (Usually 5–15 min)
Once validated:
Status will change to "Issued"
You can now use this certificate in CloudFront or ALB

```


✅ [Update DNS in Route 53 with CloudFront’s domain as alias](https://docs.google.com/document/d/1JITJ2r5EPSsZdbaxmWQMmBB_GvnjvAIqItDiTdyi5Go/edit?tab=t.5aprm9n42ssb)


✅ Test live website
```bash
Visit: https://portfolio.mkcloudai.com ✅
nslookup -type=ns portfolio.mkcloudai.com
whois portfolio.mkcloudai.com
ping portfolio.mkcloudai.com
tracert portfolio.mkcloudai.com
ipconfig /flushdns
```

✅  Add CloudFront cache invalidation to CI/CD

### If any change from github
```bash
git pull origin main
```

### Delete personal access token

```bash
https://github.com/settings/personal-access-tokens
```
The error identity_sign: private key ... contents do not match public means your private key and public key pair don’t match or your SSH keys are corrupted/misconfigured.

```bash
How to fix:
## 1. Backup and Remove Old SSH Keys
cd ~/.ssh
mkdir backup
move id_rsa* backup/
## 2. Generate a New SSH Key Pair

ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
Press Enter to accept defaults

(Optional) add a passphrase or leave empty

## 3. Start ssh-agent and Add New Key

## 4. Copy Your New Public Key

cat ~/.ssh/id_rsa.pub
Copy all the output.

## 5. Add the New Public Key to GitHub
Go to https://github.com/settings/keys

Click New SSH key

Paste your new public key

## 6. Test SSH Connection

ssh -T git@github.com
You should see:


Hi username! You've successfully authenticated...
Optional: Reset your Git remote URL to SSH if needed

git remote set-url origin git@github.com:mxkdevops/portfolio-mo.git

```