# Cheat Sheet: CI/CD Pipeline (Commit → Build → Deploy → Pipeline)

Quick reference for AWS CI/CD services (CodeCommit, CodeBuild, CodeDeploy, CodePipeline).

---

## CI/CD Pipeline Overview

```
┌─────────────┐    ┌───────────┐    ┌────────────┐    ┌─────────────┐
│ CodeCommit  │ →  │ CodeBuild │ →  │ CodeDeploy │ →  │   EC2/ECS   │
│ (Git Repo)  │    │ (Build)   │    │ (Deploy)   │    │  /Lambda    │
└─────────────┘    └───────────┘    └────────────┘    └─────────────┘
       ↑                  ↑                 ↑                 ↑
       └──────────────────┴─────────────────┴─────────────────┘
                      CodePipeline (Orchestration)
```

---

## Service Overview Table

| Service | Purpose | Input | Output | Use Case |
|---------|---------|-------|--------|----------|
| **CodeCommit** | **Git repository** | Developer commits | Source code | Store and version control code |
| **CodeBuild** | **Build and test** | Source code | Build artifacts (JAR, ZIP, Docker image) | Compile, test, package code |
| **CodeDeploy** | **Deploy** | Build artifacts | Running application | Deploy to EC2, ECS, Lambda |
| **CodePipeline** | **Orchestrate** | All stages | Automated pipeline | Connect all steps (commit → build → deploy) |

---

## CodeCommit (Git Repository)

**What It Is:** Fully managed Git repository (like GitHub, GitLab, Bitbucket)

| Feature | Details |
|---------|---------|
| **Purpose** | Store source code with version control |
| **Protocol** | Git (SSH, HTTPS) |
| **Authentication** | IAM users, SSH keys, HTTPS credentials |
| **Encryption** | At-rest (KMS), in-transit (HTTPS, SSH) |
| **Integrations** | CodePipeline, CodeBuild, SNS notifications |
| **Pricing** | Free (up to 5 users), then $1/user/month |

**Key Features:**
- Pull requests and code reviews
- Branch protections
- SNS notifications (on commit, branch updates)
- Trigger Lambda on repository events

**Use Case:** Store application source code (alternative to GitHub)

---

## CodeBuild (Build and Test)

**What It Is:** Fully managed build service that compiles code, runs tests, produces artifacts

| Feature | Details |
|---------|---------|
| **Purpose** | Compile, test, package code |
| **Build Spec** | `buildspec.yml` (defines build steps) |
| **Environment** | Docker-based (AWS-managed or custom image) |
| **Artifacts** | Store in S3, push Docker images to ECR |
| **Pricing** | Pay per build minute (varies by instance type) |

**Buildspec Example (`buildspec.yml`):**
```yaml
version: 0.2
phases:
  install:
    commands:
      - npm install
  build:
    commands:
      - npm run build
      - npm test
artifacts:
  files:
    - '**/*'
  base-directory: 'dist'
```

**Key Features:**
- Continuous scaling (no build queue)
- Supports multiple languages (Java, Python, Node.js, etc.)
- Integrates with CodePipeline, GitHub, Bitbucket
- Store artifacts in S3 or push Docker images to ECR

**Use Case:** Build Java application (Maven), run unit tests, package as JAR

---

## CodeDeploy (Deploy)

**What It Is:** Automated deployment service to EC2, ECS, Lambda, on-premises servers

| Feature | Details |
|---------|---------|
| **Purpose** | Deploy applications to compute resources |
| **Targets** | EC2, ECS, Lambda, on-premises |
| **Deployment Types** | In-place, Blue/Green, Canary, Linear |
| **AppSpec** | `appspec.yml` (defines deployment steps) |
| **Rollback** | Automatic rollback on failure |
| **Pricing** | **Free** for EC2/Lambda, $0.02/ECS task update |

**Deployment Strategies:**

| Strategy | Description | Use Case |
|----------|-------------|----------|
| **In-Place** | Update instances in place (downtime possible) | Dev/test, cost-sensitive |
| **Blue/Green** | Deploy to new instances, switch traffic | Zero-downtime production deployments |
| **Canary** | Gradually shift traffic (e.g., 10% → 100%) | Test new version with small traffic first |
| **Linear** | Gradually shift traffic in equal increments (e.g., +10% every 10 min) | Controlled rollout |

**AppSpec Example (`appspec.yml` for EC2):**
```yaml
version: 0.0
os: linux
files:
  - source: /
    destination: /var/www/html
hooks:
  ApplicationStop:
    - location: scripts/stop_server.sh
  ApplicationStart:
    - location: scripts/start_server.sh
```

**Key Features:**
- Automatic rollback on deployment failure
- Deployment groups (deploy to specific environments)
- Lifecycle hooks (run scripts before/after deployment)

**Use Case:** Deploy web app to EC2 with blue/green strategy (zero downtime)

---

## CodePipeline (Orchestration)

**What It Is:** Fully managed CI/CD orchestration service (connects all stages)

| Feature | Details |
|---------|---------|
| **Purpose** | Automate entire CI/CD workflow |
| **Stages** | Source (CodeCommit, GitHub), Build (CodeBuild), Deploy (CodeDeploy, ECS, S3) |
| **Triggers** | Automatic (on code commit) or manual |
| **Approvals** | Manual approval gates (for production deployments) |
| **Pricing** | $1/active pipeline/month |

**Pipeline Structure:**
1. **Source Stage:** Detect code changes (CodeCommit, GitHub, S3)
2. **Build Stage:** Build and test code (CodeBuild, Jenkins)
3. **Test Stage:** Run integration tests (CodeBuild)
4. **Deploy Stage:** Deploy to environment (CodeDeploy, ECS, S3, CloudFormation)
5. **Approval Stage:** Manual approval before production (optional)

**Key Features:**
- Visual pipeline editor
- Parallel actions (build + test simultaneously)
- Manual approval gates
- SNS notifications (pipeline success/failure)
- Integration with third-party tools (GitHub, Jenkins, Bitbucket)

**Use Case:** Automate commit → build → test → deploy to production (triggered on every commit)

---

## Complete CI/CD Pipeline Example

**Scenario:** Deploy Node.js web app to EC2 on every commit

### **Pipeline Stages:**

1. **Source (CodeCommit)**
   - Developer pushes code to CodeCommit repository
   - Pipeline automatically triggers

2. **Build (CodeBuild)**
   - Run `npm install` (install dependencies)
   - Run `npm test` (unit tests)
   - Build artifacts (package as ZIP)
   - Store ZIP in S3

3. **Deploy (CodeDeploy)**
   - Deploy ZIP to EC2 instances
   - Blue/Green deployment (zero downtime)
   - Automatic rollback if deployment fails

4. **Approval (Manual)**
   - Wait for manual approval before production
   - Notify DevOps team via SNS

---

## Exam Scenarios

| Scenario | Service |
|----------|---------|
| Store source code with version control | **CodeCommit** |
| Compile Java code, run tests, package as JAR | **CodeBuild** |
| Deploy application to EC2 with zero downtime | **CodeDeploy (Blue/Green)** |
| Automate commit → build → deploy workflow | **CodePipeline** |
| Gradual traffic shift (10% → 50% → 100%) | **CodeDeploy (Canary/Linear)** |
| Manual approval before production deploy | **CodePipeline (Approval Stage)** |
| Build Docker image, push to ECR | **CodeBuild** |
| Deploy Lambda function | **CodeDeploy** |

---

## Key Exam Tips

1. **CodeCommit = Git repository** (like GitHub)
2. **CodeBuild = Build and test** (compile, package)
3. **CodeDeploy = Deploy** (to EC2, ECS, Lambda)
4. **CodePipeline = Orchestration** (automate entire workflow)
5. **Blue/Green = Zero downtime** (deploy to new instances, switch traffic)
6. **In-Place = Update existing instances** (possible downtime)
7. **Canary/Linear = Gradual rollout** (test with small traffic first)
8. **CodeDeploy is free for EC2/Lambda** (charged only for ECS)
9. **CodePipeline triggers automatically** on code commit
10. **Manual approval gates** available in CodePipeline

---

## Build Spec vs AppSpec

| File | Service | Purpose |
|------|---------|---------|
| **buildspec.yml** | CodeBuild | Define build commands (install, build, test) |
| **appspec.yml** | CodeDeploy | Define deployment steps (where to copy files, lifecycle hooks) |

---

## Integration with Other Services

### **CodePipeline Integrations:**

**Source:**
- CodeCommit, GitHub, Bitbucket, S3, ECR

**Build:**
- CodeBuild, Jenkins, CloudBees

**Deploy:**
- CodeDeploy, ECS, S3, CloudFormation, Elastic Beanstalk

**Test:**
- CodeBuild, third-party testing tools

---

## Summary

| Service | Purpose | Key Feature |
|---------|---------|-------------|
| **CodeCommit** | Git repository | Version control for source code |
| **CodeBuild** | Build and test | Compile code, run tests, produce artifacts |
| **CodeDeploy** | Deploy | Automated deployment to EC2, ECS, Lambda |
| **CodePipeline** | Orchestration | Automate commit → build → deploy workflow |

**CI/CD Flow:** CodeCommit (source) → CodeBuild (build) → CodeDeploy (deploy) → Orchestrated by CodePipeline
