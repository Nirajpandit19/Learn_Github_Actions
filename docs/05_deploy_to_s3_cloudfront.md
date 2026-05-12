## Project — Deploy Static Site to S3 + CloudFront
 
> **What the lectures taught:** Deploy a static website to a public S3 bucket.
> **What we actually built:** Deploy a static website to a **private S3 bucket** served securely through **CloudFront with Origin Access Control (OAC)** — the production-grade approach.
 
---
 
### Prerequisites
 
Before running this workflow, complete the following one-time AWS setup:
 
---
 
#### Step 1 — Create a Private S3 Bucket
 
1. Go to **AWS Console → S3 → Create bucket**
2. Enter a unique bucket name (e.g. `my-static-site-bucket`)
3. Select your region (e.g. `ap-south-1`)
4. Under **Block Public Access** → keep all options **enabled** (bucket stays private)
5. Leave all other settings as default → click **Create bucket**
> 📝 Note down your bucket name — you'll add it as a GitHub secret later.
 
---
 
#### Step 2 — Create a CloudFront Distribution with OAC
 
1. Go to **AWS Console → CloudFront → Create distribution**
2. Under **Origin domain** → select your S3 bucket from the dropdown
3. Under **Origin Access** → select **Origin Access Control (OAC)**
4. Click **Create new OAC** → give it a short name (e.g. `static-site-oac`, max 64 chars) → click **Create**
5. Scroll down to **Default cache behavior** → leave defaults
6. Under **Settings → Default root object** → enter `index.html`
7. Click **Create distribution**
> ⚠️ After creating, CloudFront will show a **yellow banner** saying *"Copy policy"* — do not skip this step.
 
---
 
#### Step 3 — Apply the Bucket Policy to S3
 
CloudFront auto-generates a bucket policy that grants it read access to your private S3 bucket.
 
1. Click **Copy policy** from the CloudFront banner
2. Go to **S3 → your bucket → Permissions tab → Bucket Policy**
3. Click **Edit** → paste the copied policy → click **Save changes**
The policy looks like this:
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowCloudFrontRead",
      "Effect": "Allow",
      "Principal": {
        "Service": "cloudfront.amazonaws.com"
      },
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::YOUR_BUCKET_NAME/*",
      "Condition": {
        "StringEquals": {
          "AWS:SourceArn": "arn:aws:cloudfront::YOUR_ACCOUNT_ID:distribution/YOUR_DIST_ID"
        }
      }
    }
  ]
}
```
 
> This policy allows **only CloudFront** to read objects from your bucket — no direct public access.
 
---
 
#### Step 4 — Create an IAM User for GitHub Actions
 
1. Go to **AWS Console → IAM → Users → Create user**
2. Give it a name (e.g. `github-actions-deployer`)
3. Under **Permissions → Attach policies directly**, attach:
   - `AmazonS3FullAccess` (or a custom policy scoped to your bucket)
   - `CloudFrontFullAccess` (or a custom policy scoped to your distribution)
4. Click **Create user**
5. Go to the user → **Security credentials → Create access key**
6. Select **Application running outside AWS** → click **Create**
7. **Copy the Access Key ID and Secret Access Key** — you will not see the secret again
> 🔒 Best practice: Instead of full access policies, create a custom IAM policy scoped only to your specific bucket and distribution to follow least-privilege principles.
 
---
 
#### Step 5 — Add Secrets and Variables to GitHub
 
Go to your repository → **Settings → Secrets and variables → Actions**:
 
| Name | Type | Value |
|------|------|-------|
| `AWS_ACCESS_KEY_ID` | Secret | From Step 4 |
| `AWS_SECRET_ACCESS_KEY` | Secret | From Step 4 |
| `S3_BUCKET_NAME` | Secret | Your bucket name from Step 1 |
| `CLOUDFRONT_DISTRIBUTION_ID` | Secret | Your distribution ID from Step 2 |
| `AWS_REGION` | Variable | e.g. `ap-south-1` |
 
> ✅ You are now ready to run the workflow.
 
---
 
### Architecture
 
```
Developer pushes to main branch
          ↓
GitHub Actions triggers automatically
          ↓
Runner checks out code (actions/checkout)
          ↓
AWS credentials configured via secrets
          ↓
index.html synced to private S3 bucket (aws s3 sync)
          ↓
CloudFront cache invalidated (/index.html)
          ↓
Site live at CloudFront URL ✅
```
 
**Why CloudFront + OAC instead of a public S3 bucket:**
 
| | Public S3 Bucket | Private S3 + CloudFront + OAC |
|---|---|---|
| Bucket exposed publicly | ⚠️ Yes | ✅ No |
| Global CDN delivery | ❌ | ✅ |
| Cache control | ❌ | ✅ |
| Custom domain support | ❌ | ✅ |
| Best practice | ❌ | ✅ |
 
**Why not Presigned URLs:**
Presigned URLs expire after a set time — not suitable for a website meant to be permanently accessible. CloudFront + OAC gives permanent, secure access without expiry.
 
---
 
### Workflow File
 
```yaml
name: Deploy HTML to S3
 
on:
  push:
    branches:
      - main                        # Triggers only after PR is merged to main
 
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code from repo
        uses: actions/checkout@v4   # Clones repo onto the empty runner VM
 
      - name: Configure AWS Credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: ${{ vars.AWS_REGION }}    # Variable — not sensitive
 
      - name: Push static code to S3
        run: |
          aws s3 sync ./ s3://${{ secrets.S3_BUCKET_NAME }}/ \
          --exclude "*" \
          --include "index.html"    # Only sync index.html, ignore everything else
 
      - name: Invalidate CloudFront cache
        run: |
          aws cloudfront create-invalidation \
          --distribution-id ${{ secrets.CLOUDFRONT_DISTRIBUTION_ID }} \
          --paths "/index.html"     # Force CloudFront to fetch fresh file from S3
```
 
---
 
### Step-by-Step Explanation
 
**Step 1 — `actions/checkout@v4`**
 
The GitHub-hosted runner is a **brand new empty Ubuntu VM** every time. It has no knowledge of your repository. `actions/checkout` clones your repository onto that VM so subsequent steps have access to your files. Without this step, `aws s3 sync ./` would sync an empty directory.
 
**Step 2 — `aws-actions/configure-aws-credentials@v4`**
 
Sets up AWS CLI authentication on the runner using your IAM credentials stored as secrets. Uses the official action instead of manual `aws configure` commands — more secure, handles credential masking automatically.
 
**Step 3 — `aws s3 sync`**
 
Syncs files from the runner to the S3 bucket:
- `--exclude "*"` — exclude everything first
- `--include "index.html"` — then include only index.html
This ensures only your HTML file is uploaded, not the `.github/` folder, README, or any other repo files.
 
**Step 4 — CloudFront cache invalidation**
 
CloudFront caches your files at edge locations globally for fast delivery. When you update `index.html` in S3, CloudFront might still serve the old cached version. The invalidation tells CloudFront to clear the cache for `/index.html` and fetch the latest version from S3 on the next request.
 
---
 
### Key Learnings and Security Decisions
 
**`push` to main vs `pull_request`:**
- `pull_request` trigger fires when a PR is **opened** — before code is reviewed and merged
- `push` to main fires **after** the PR is merged — only reviewed, approved code gets deployed
- For deployment workflows always use `push` to `main`; use `pull_request` for CI (tests, scans)
**`vars.` vs `secrets.` for AWS region:**
- `AWS_REGION` is not sensitive — it's fine to use `vars.` (repository variable)
- Sensitive values like keys, bucket names, distribution IDs use `secrets.`
**Never echo a secret:**
- Even though GitHub masks secrets as `***` in logs, never use `echo ${{ secrets.X }}` in production
- Pass secrets directly to the commands that need them
**Docker login security note:**
When using `echo password | docker login`, GitHub-hosted runners are safe because the VM is destroyed after the job. On self-hosted runners this is a risk as `~/.docker/config.json` persists. Always prefer `docker/login-action@v3` which handles credentials securely.
 
---
 
*Notes by Pandit Niraj Raj | GitHub: [Nirajpandit19](https://github.com/Nirajpandit19)*