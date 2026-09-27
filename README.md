# DevOps Starter Project: Auto-Deploy a Website to AWS with GitHub Actions

This project shows the core idea of DevOps: **when you push code, it deploys itself.**
No manual uploads, no clicking around a console every time you change something.

What it demonstrates (the words to use when you present it):
- **Version control** — Git/GitHub
- **CI/CD (Continuous Integration / Continuous Deployment)** — GitHub Actions pipeline
- **Infrastructure** — AWS S3 static website hosting
- **Least-privilege security** — a scoped-down IAM user/policy instead of using root credentials
- **Automation** — zero manual steps after the initial setup

Total cost: **$0** (everything here fits in the AWS Free Tier for a small demo site).
Total setup time: roughly 30–45 minutes the first time.

---

## Part 1 — Install tools locally

1. Install [VS Code](https://code.visualstudio.com/) (you likely have this already).
2. Install [Git](https://git-scm.com/downloads).
3. Install the [AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html) — optional but useful for testing locally.
4. In VS Code, install the extensions: **GitHub Actions** and **AWS Toolkit** (optional, nice-to-have, not required).

## Part 2 — Create your AWS account

1. Go to https://aws.amazon.com and click **Create an AWS Account**.
2. You'll need an email, a credit card (for identity verification — the Free Tier won't charge you if you stay within limits), and phone verification.
3. Once logged in, go to the top-right region selector and pick a region close to you (e.g. `us-east-1`). Remember this — you'll need it later.

⚠️ **Important safety step:** Never use your root account credentials for daily work or in code. Create a separate limited user (next step).

## Part 3 — Create an S3 bucket (this will host your website)

1. In the AWS Console, search for **S3** → **Create bucket**.
2. Give it a **globally unique name**, e.g. `yourname-devops-demo-2026`.
3. Uncheck "Block all public access" (a static site needs to be publicly readable) and confirm.
4. After creation, go to the bucket → **Properties** → **Static website hosting** → **Enable**. Set `index.html` as the index document.
5. Go to **Permissions** → **Bucket Policy** and add a policy allowing public `GetObject` (AWS shows you a template for this when you enable static hosting — follow its suggestion).
6. Note the **bucket website endpoint URL** shown on that page — that will be your live site URL.

## Part 4 — Create a limited IAM user for deployments

Never give GitHub your root AWS credentials. Instead:

1. Go to **IAM** → **Users** → **Create user**, e.g. `github-actions-deployer`.
2. Choose **Attach policies directly** → **Create policy** → paste the contents of `iam-policy.json` from this project (replace `REPLACE-WITH-YOUR-BUCKET-NAME` with your actual bucket name first).
3. Finish creating the user, then go to that user → **Security credentials** → **Create access key** → choose "Command Line Interface (CLI)".
4. Copy the **Access Key ID** and **Secret Access Key** — you'll only see the secret once.

## Part 5 — Push this project to GitHub

1. In VS Code, open this project folder.
2. Open the terminal (`` Ctrl+` ``) and run:
   ```bash
   git init
   git add .
   git commit -m "Initial DevOps demo project"
   ```
3. Create a new repository on GitHub (empty, no README), then:
   ```bash
   git remote add origin https://github.com/YOUR-USERNAME/YOUR-REPO.git
   git branch -M main
   git push -u origin main
   ```

## Part 6 — Add your AWS credentials as GitHub Secrets

In your GitHub repo: **Settings → Secrets and variables → Actions → New repository secret**. Add three secrets:

| Secret name | Value |
|---|---|
| `AWS_ACCESS_KEY_ID` | from Part 4 |
| `AWS_SECRET_ACCESS_KEY` | from Part 4 |
| `AWS_REGION` | e.g. `us-east-1` |
| `S3_BUCKET_NAME` | your bucket name from Part 3 |

These let GitHub Actions deploy on your behalf without the credentials ever appearing in your code.

## Part 7 — Watch it deploy

1. Make any small change to `site/index.html` (e.g. edit the text).
2. Commit and push:
   ```bash
   git add .
   git commit -m "Update homepage text"
   git push
   ```
3. Go to your GitHub repo → **Actions** tab. You'll see the workflow run automatically.
4. Once it finishes (green check ✅), visit your S3 website endpoint URL — your change is live.

That's the whole loop: **edit → push → auto-deploy**, with zero manual steps. This is CI/CD in miniature.

---

## How to talk about this project in an interview / presentation

- "I built a small CI/CD pipeline using GitHub Actions that automatically deploys a static website to AWS S3 every time I push to the main branch."
- "I followed least-privilege security practices by creating a scoped IAM policy instead of using root or admin credentials."
- "The pipeline is defined as code (`deploy.yml`), so the whole deployment process is version-controlled and repeatable."

## Natural next steps (mention these as "what I'd add next" — shows growth mindset)

- Add **CloudFront** in front of S3 for HTTPS and CDN caching.
- Add a **build/test step** to the pipeline before deploy (e.g. HTML linting).
- Manage the S3 bucket itself with **Terraform** instead of clicking in the console (Infrastructure as Code).
- Add a **staging vs production** environment split.
