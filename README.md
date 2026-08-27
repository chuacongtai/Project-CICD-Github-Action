# Blue-Green Deployment on AWS ECS with GitHub Actions & Terraform

This is my project about Blue-Green Deployment on AWS ECS Fargate.  
I use Terraform to create the infrastructure and GitHub Actions for CI/CD. There is also a manual approval step before switching traffic.

I did this project to practice Blue-Green deployment and understand how it works on AWS.

---
## Overview

The app is just a simple Node.js service. I deploy it with Blue-Green style:

- There are 2 environments: Blue and Green
- Traffic goes through ALB
- New version always deploy to the idle one first
- Need to approve manually before switch production traffic
- After switch, old environment will be scaled down to 0

---

## Architecture Diagrams

### 1. Network Architecture
![Network Architecture](./architecture/Network%20Architecture%20Diagram.png)

### 2. Security Architecture
![Security Architecture](./architecture/Security%20Architecture%20Diagram.png)

### 3. Application / Services Architecture
![Application Services Architecture](./architecture/Services%20and%20Resources%20Diagram.png)

### 4. CI/CD Architecture
![CI/CD Architecture](./architecture/CICD%20Pipeline%20Architecture.png)

---

## Key Features

- Blue-Green deployment, can switch traffic with no downtime
- Infra is written in Terraform (I split it into modules)
- GitHub Actions pipeline: build → push image to ECR → deploy → switch traffic → cleanup
- Manual approval with GitHub Environments
- Can deploy to Blue or Green depending on which one is idle
- ECS tasks run in private subnet, security group is limited

---

## Tech Stack

| Category                  | Technology                        |
|---------------------------|-----------------------------------|
| Cloud Provider            | AWS                               |
| Compute                   | ECS (Fargate)              |
| Container Registry        | ECR                        |
| Load Balancer             | ALB   |
| Infrastructure as Code    | Terraform                         |
| CI/CD                     | GitHub Actions                    |
| Application               | Node.js + Express                 |
| Region                    | ap-southeast-1        |

---

## Project Structure

```text
.
├── .github/workflows/
│   ├── blue-green-deployment.yml
│   ├── build-docker.yml
│   ├── deploy-ecs.yml
│   ├── switch-traffic.yml
│   ├── clear-resources.yml
│   └── test-on-develop.yml
├── architecture/                  
├── legacy/                         
├── terraform/
│   ├── modules/
│   │   ├── networking/
│   │   ├── security/
│   │   ├── load_balance/
│   │   └── ecs_cluster/
│   └── singapore-dev/
├── Dockerfile
├── index.js
├── package.json
└── README.md
```

## How It Works

1. Create a new tag for the version
2. GitHub Actions will build the image and push to ECR
3. Deploy new version to the idle environment (blue or green)
4. Wait for approval
5. Switch traffic on ALB (port 80 and 81)
6. Scale old environment to 0
---

## How to run

### Prerequisites

- AWS Account
- Terraform >= 1.5
- AWS CLI
- Docker
- GitHub Secrets:
  - `AWS_ACCESS_KEY_ID`
  - `AWS_SECRET_ACCESS_KEY`

### 1. Create infrastructure

```bash
cd terraform/singapore-dev
terraform init
terraform plan
terraform apply
```
Note: don’t commit tfstate file. For real project better use remote state (S3 + DynamoDB).

### 2. Setup GitHub

Secrets:
- `AWS_ACCESS_KEY_ID`
- `AWS_SECRET_ACCESS_KEY`

Variables
- `AWS_REGION`
- `AWS_ACCOUNT_ID`
- `ECR_REPOSITORY`
- `ALB_ARN`
- `BLUE_CLUSTER`
- `GREEN_CLUSTER`
- `TASK_FAMILY`
- `SERVICE_NAME`

### 3. Create GitHub Environments

Create environment name `production-approval` and enable Required reviewers.

### 4. Deploy

1. Create a tag (example: v1.0.1)
2. Go to Actions → Complete Blue-Green Deployment
3. Fill the inputs:
   - version: tag name
   - target_cluster: blue or green (the idle one)
   - switch_traffic: true/false
   - cleanup_old_cluster: true/false

---

## What I learned 

- How Blue-Green works on ECS
- Write Terraform with modules
- Make CI/CD pipeline with GitHub Actions
- Use GitHub Environments for approval
- Switch traffic between 2 target groups
- Basic setup for public/private subnet and security group
- Try to follow least privilege

---

## Notes

- Always deploy to idle environment first
- Don’t commit state file or secrets
- Should require approval before switch production traffic
- If work with team, better use remote state

---

## License

Apache-2.0