# Automated AWS Web Hosting with Terraform

> An Infrastructure-as-Code (IaC) project that provisions a production-grade static website architecture on AWS. Integrates S3 for secure storage, CloudFront for global content delivery, Route 53 with ACM for automated DNS and SSL management, and a fully automated CI/CD pipeline powered by CodePipeline and CodeBuild.

🌐 **Live at:** [juanito-ramos-dev.site](https://juanito-ramos-dev.site)

---

## 🏗️ Architecture

<p align="center">
  <img src="STATIC_WEBSITE_ARCHITECTURE.png" width="700"/>
</p>



**How it works:** Every push to the `main` branch triggers CodePipeline, which pulls the latest code from GitHub, syncs static files to S3 via CodeBuild, and automatically invalidates the CloudFront cache so visitors always see the latest version — all without any manual intervention.

---

## 📁 Project Structure

```
├── environments/
│   └── production/
│       ├── main.tf                   # Root module — wires all modules together
│       ├── variables.tf              # Input variable declarations
│       ├── outputs.tf                # Output values (URLs, IDs, nameservers)
│       ├── providers.tf              # AWS provider config (main + us-east-1 alias)
│       └── terraform.tfvars          # Your actual values (never commit this)
├── modules/
│   ├── acm/                          # SSL Certificate module (us-east-1)
│   │   ├── main.tf
│   │   ├── variables.tf
│   │   └── outputs.tf
│   ├── cloudfront/                   # CloudFront CDN module
│   │   ├── main.tf
│   │   ├── variables.tf
│   │   └── outputs.tf
│   ├── route53/                      # DNS hosted zone + A records
│   │   ├── main.tf
│   │   ├── variables.tf
│   │   └── outputs.tf
│   └── s3/                           # S3 bucket + CI/CD pipeline
│       ├── main.tf
│       ├── variables.tf
│       ├── outputs.tf
│       └── cicd.tf                   # CodePipeline + CodeBuild resources
├── index.html                        # Website entry point
├── error.html                        # 404 error page
└── .gitignore
```

---

## 🚀 Features

- ✅ S3 Static Website Hosting with versioning enabled
- ✅ CloudFront CDN with global distribution (`PriceClass_100`)
- ✅ Route 53 DNS management with A records for root and `www`
- ✅ Free SSL/TLS Certificates via ACM (auto-validated via DNS)
- ✅ Modular Terraform architecture for reusability
- ✅ Security best practices — OAC, HTTPS-only, public access blocked
- ✅ Fully automated CI/CD pipeline via CodePipeline + CodeBuild
- ✅ CloudFront cache invalidation on every deploy

---

## 🛠️ Prerequisites

Before getting started, make sure you have the following ready:

- AWS account with sufficient IAM permissions
- [Terraform >= 1.0](https://developer.hashicorp.com/terraform/install) installed locally
- [AWS CLI v2](https://aws.amazon.com/cli/) installed and configured
- A registered domain name (this project uses Hostinger)
- A GitHub repository for your website files
- A GitHub Personal Access Token with `repo` + `admin:repo_hook` scopes

---

## ⚙️ Quick Start

### 1. Clone the repository

```bash
git clone https://github.com/your-username/Automated-AWS-Web-Hosting-with-Terraform.git
cd Automated-AWS-Web-Hosting-with-Terraform
```

### 2. Configure AWS credentials

```bash
aws configure
# Enter your Access Key ID, Secret Access Key, region (ap-southeast-1), output (json)
```

Verify it works:
```bash
aws sts get-caller-identity
```

### 3. Set up your variables

```bash
cd environments/production
cp terraform.tfvars.example terraform.tfvars
# Edit terraform.tfvars with your actual values
```

### 4. Initialize and deploy

```bash
terraform init
terraform plan
terraform apply
```

### 5. Update your domain nameservers

After apply completes, copy the 4 Route 53 nameservers from the output and update them in your domain registrar's DNS settings:

```
ns-xxxx.awsdns-xx.com
ns-xxxx.awsdns-xx.net
ns-xxxx.awsdns-xx.org
ns-xxxx.awsdns-xx.co.uk
```

### 6. Authorize the GitHub connection

Go to **AWS Console → Developer Tools → Settings → Connections**, find your connection and click **"Update pending connection"** to authorize GitHub access.

---

## 📋 Configuration

### Required Variables (`terraform.tfvars`)

```hcl
# AWS Configuration
aws_region = "ap-southeast-1"

# Domain Configuration
domain_name = "yourdomain.com"

# Project Configuration
environment  = "production"
project_name = "your-project-name"
owner        = "YourName"

# GitHub Configuration
github_repo_owner = "your-github-username"
github_repo_name  = "your-repository-name"
branch_name       = "main"
```

> ⚠️ `terraform.tfvars` is listed in `.gitignore` — never commit this file as it may contain sensitive values.

---

## 🔄 CI/CD Pipeline

The pipeline is fully automated:

1. **Source stage** — CodePipeline detects a push to `main` via GitHub App connection
2. **Build/Deploy stage** — CodeBuild syncs all website files to S3, excluding Terraform files
3. **Cache invalidation** — CloudFront cache is cleared automatically so changes are live immediately

To test it, just push any change to `main`:

```bash
git add .
git commit -m "update: my change"
git push origin main
```

Watch it deploy at: **AWS Console → CodePipeline**

---

## 🌐 DNS Configuration

After deployment:

1. Copy the 4 Route 53 nameservers from `terraform output name_servers`
2. Update your domain registrar (Hostinger, Namecheap, etc.) with these values
3. Wait 15–30 minutes for DNS propagation

You can verify propagation with:
```bash
nslookup -type=NS yourdomain.com 8.8.8.8
```

When it returns `awsdns` nameservers instead of your registrar's defaults, propagation is complete.

---

## 💰 Cost Optimization

| Service | Approach |
|---|---|
| CloudFront | `PriceClass_100` — cheapest edge regions only |
| S3 | Standard storage with versioning for rollback capability |
| Route 53 | Pay-per-query — no flat monthly fee |
| ACM | SSL certificates are completely free |
| CodePipeline | Free tier covers 1 active pipeline |
| CodeBuild | Free tier covers 100 build minutes/month |

> 💡 **Estimated monthly cost:** $1–5 for small websites

---

## 🔒 Security Features

- **HTTPS Enforced** — all HTTP traffic is redirected to HTTPS
- **Origin Access Control (OAC)** — S3 bucket is private; only CloudFront can read from it
- **Public Access Blocked** — S3 bucket has all public access settings disabled
- **TLS 1.2+** — older protocol versions are rejected
- **Least Privilege IAM** — CodeBuild and CodePipeline roles have only the permissions they need

---

## 📊 Outputs

After `terraform apply`, the following values are available:

```bash
terraform output
```

```hcl
cloudfront_url             = "https://xxxx.cloudfront.net"
cloudfront_distribution_id = "EXXXXXXXXXXXX"
domain_url                 = "https://yourdomain.com"
www_url                    = "https://www.yourdomain.com"
certificate_arn            = "arn:aws:acm:us-east-1:..."
name_servers               = ["ns-xxx.awsdns-xx.com", ...]
```

---

## 🧹 Cleanup

To tear down all provisioned infrastructure:

```bash
cd environments/production
terraform destroy
```

> ⚠️ Empty the S3 bucket manually before destroying, otherwise Terraform will fail on bucket deletion.

```bash
aws s3 rm s3://yourdomain.com --recursive
terraform destroy
```

---

## 🐛 Troubleshooting

A full log of issues encountered during the build of this project and how each was resolved.

---

### 1. Terraform not recognized after adding to PATH

**Error:**
```
terraform : The term 'terraform' is not recognized...
```

**Cause:** The PATH entry pointed to the `.exe` file itself instead of the folder containing it.

**Fix:** In System Environment Variables, change the PATH entry from:
```
D:\path\to\terraform.exe     ❌
```
to just the folder:
```
D:\path\to\terraform\        ✅
```
Then open a **brand new** terminal window.

---

### 2. Invalid version constraint syntax

**Error:**
```
Error: Invalid version constraint — version = "-> 5.0"
```

**Cause:** The `~>` (tilde-greater-than) operator was corrupted to `->` when pasting into the file.

**Fix:** Manually type `~>` in `providers.tf` — tilde (`~`) is next to the `1` key on the keyboard. Alternatively, switch to `~> 6.0` since Terraform had already downloaded the v6 provider:

```hcl
version = "~> 6.0"
```

Then run `terraform init -upgrade` to update the lock file.

---

### 3. S3 bucket name included `https://`

**Error:**
```
+ bucket = "https://juanito-ramos-dev.site/"
```

**Cause:** The `domain_name` variable in `terraform.tfvars` was set to the full URL instead of just the domain.

**Fix:**
```hcl
domain_name = "juanito-ramos-dev.site"    ✅
# NOT
domain_name = "https://juanito-ramos-dev.site/"  ❌
```

---

### 4. ACM certificate stuck at "Pending validation" for 30+ minutes

**Symptom:**
```
module.acm.aws_acm_certificate_validation.website: Still creating... [30m00s elapsed]
```

**Cause:** The domain's nameservers in Hostinger were still pointing to Hostinger's parking servers, so AWS couldn't verify the DNS validation records it created in Route 53.

**Fix:**
1. Go to **Route 53 → Hosted Zones → your domain → NS record** and copy the 4 nameservers
2. Go to **Hostinger → DNS / Nameservers → Change Nameservers**
3. Replace the default parking nameservers with the 4 Route 53 nameservers
4. Wait 15–30 minutes for DNS propagation
5. Terraform will automatically continue once ACM validates

---

### 5. CloudFront distribution failed with invalid certificate error

**Error:**
```
InvalidViewerCertificate: The specified SSL certificate doesn't exist,
isn't in us-east-1 region, isn't valid, or doesn't include a valid certificate chain.
```

**Cause:** CloudFront was referencing the certificate ARN before it was fully validated. The `certificate_arn` output needed to come from `aws_acm_certificate_validation.website.certificate_arn` (the validated cert) rather than `aws_acm_certificate.website.arn` (the unvalidated cert).

**Fix:** Update `modules/acm/outputs.tf`:
```hcl
output "certificate_arn" {
  # Use the validation resource's ARN — guarantees cert is issued before CloudFront uses it
  value = aws_acm_certificate_validation.website.certificate_arn
}
```

---

### 6. CodeBuild buildspec used invalid phase name

**Error:**
```
YAML_FILE_ERROR: Invalid buildspec phase name: deploy
Expecting: build, install, post_build, pre_build
```

**Cause:** The buildspec inside `cicd.tf` used `deploy:` as a phase name, which doesn't exist in CodeBuild's phase model.

**Fix:** Rename the phase from `deploy` to `build`:
```yaml
phases:
  build:        # ✅ valid phase name
    commands:
      - aws s3 sync ...
```

---

### 7. CodeBuild AccessDenied on S3 sync

**Error:**
```
AccessDenied: not authorized to perform: s3:ListBucket on resource: "arn:aws:s3:::juanito-ramos-dev.site"
```

**Cause:** The CodeBuild IAM role policy was missing `s3:ListBucket` — required by `aws s3 sync` to compare local and remote files.

**Fix:** Add `s3:ListBucket` to the CodeBuild IAM role policy in `cicd.tf`:
```hcl
Action = [
  "s3:GetObject",
  "s3:GetObjectVersion",
  "s3:PutObject",
  "s3:DeleteObject",
  "s3:ListBucket"    # ← this was missing
]
```

---

### 8. Pipeline not auto-triggering on GitHub push

**Symptom:** GitHub connection showed as "Available" but pushing to `main` did not trigger the pipeline automatically. Pipeline only ran when manually triggered via "Release change."

**Cause:** The GitHub App was authorized at the account level but not **installed** on the specific repository. Without the app installation, CodePipeline has no webhook into the repository.

**Fix — Force a fresh GitHub App installation:**

1. Go to **AWS Console → CodePipeline → your pipeline → Edit**
2. Click **Edit stage** on the Source stage
3. Click the pencil icon on the Source action
4. In the side panel, under **Connection**, click **"Install a new app"**
5. In the GitHub popup:
   - Select your GitHub account
   - Choose **"Only select repositories"**
   - Find and check your repository
   - Click **"Install & Authorize"**
6. Back in AWS, confirm the GitHub Apps field now shows a numeric ID
7. Click **Done → Save → Save** (make sure to save the entire pipeline)

Verify it worked by checking **GitHub Settings → Applications → Installed GitHub Apps** — you should see **AWS Connector for GitHub** listed there.

Then test with an empty commit:
```bash
git commit --allow-empty -m "fix: verify pipeline auto-trigger"
git push origin main
```

The pipeline should start within seconds of the push.

---

## 📝 Module Documentation

### ACM Module (`modules/acm/`)
Requests an SSL certificate in `us-east-1` — a hard requirement for CloudFront, which only reads certificates from that region. Validation is done via DNS records automatically created in Route 53. Covers both the root domain and `www` subdomain via Subject Alternative Names (SAN). The `certificate_arn` output uses the **validated** ARN to prevent race conditions with CloudFront.

### CloudFront Module (`modules/cloudfront/`)
Creates a global CDN distribution with Origin Access Control (OAC) so S3 files are never publicly accessible directly. Configured with `PriceClass_100` for cost efficiency, HTTPS-only enforcement, TLS 1.2+ minimum, and custom error pages. Cache TTL is set to 1 hour default with a 24 hour maximum.

### Route 53 Module (`modules/route53/`)
Creates a public hosted zone for the domain and adds A records (as aliases) for both the root domain and `www` subdomain pointing to CloudFront. Outputs the 4 nameservers needed for your domain registrar configuration.

### S3 Module (`modules/s3/` + `cicd.tf`)
Provisions the website bucket with versioning, public access blocking, static website configuration, and a bucket policy restricting access to CloudFront only. The `cicd.tf` file adds the full CI/CD stack: a separate artifacts bucket, IAM roles for CodePipeline and CodeBuild with least-privilege policies, the CodeBuild project with a buildspec that syncs files and invalidates the CloudFront cache, a GitHub App connection via CodeStar Connections, and the CodePipeline pipeline with Source and Deploy stages.

---

## 🔮 Future Improvements

### Infrastructure
- **Remote state management** — HashiCorp now recommends S3 native locking using `use_lockfile` = true; DynamoDB locking is deprecated.
- **Terraform workspaces** — support multiple environments (staging, production) from the same codebase
- **WAF integration** — add AWS Web Application Firewall to CloudFront for DDoS protection and rate limiting
- **CloudFront access logging** — enable access logs to S3 for traffic analysis and debugging
- **S3 lifecycle policies** — automatically expire old object versions to reduce storage costs over time

### CI/CD Pipeline
- **Build notifications** — add SNS or Slack notifications for pipeline success/failure
- **Multi-environment pipeline** — add a staging stage before production with manual approval gate
- **Rollback mechanism** — use S3 versioning to automatically roll back on failed deployments
- **Pull request previews** — deploy preview environments on PR creation using separate CloudFront distributions

### Security
- **IAM least privilege audit** — tighten the `terraform-admin` IAM user permissions from `AdministratorAccess` to only the specific services used
- **Secrets Manager** — move sensitive values out of `terraform.tfvars` and into AWS Secrets Manager
- **CloudFront security headers** — add a Lambda@Edge or CloudFront Function to inject security headers (CSP, HSTS, X-Frame-Options)
- **MFA enforcement** — require MFA on the root account and IAM users

### Monitoring
- **CloudWatch alarms** — alert on 4xx/5xx error rate spikes from CloudFront
- **Uptime monitoring** — add Route 53 health checks with SNS alerting
- **Cost budgets** — set AWS Budget alerts to notify when monthly spend exceeds a threshold

---

## 🤝 Contributing

1. Fork the repository
2. Create a feature branch (`git checkout -b feat/your-feature`)
3. Make your changes
4. Validate with `terraform plan`
5. Open a pull request

---

## 📞 Support

- [Terraform AWS Provider Documentation](https://registry.terraform.io/providers/hashicorp/aws/latest/docs)
- [AWS CloudFront Developer Guide](https://docs.aws.amazon.com/cloudfront/)
- [AWS CodePipeline Documentation](https://docs.aws.amazon.com/codepipeline/)
- Open an issue in this repository

---

## 📄 License

This project is licensed under the MIT License — see the [LICENSE](LICENSE) file for details.
