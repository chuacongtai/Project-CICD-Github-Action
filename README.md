# Blue-Green Deployment on AWS ECS with GitHub Actions & Terraform

A hands-on DevOps project that implements **Blue-Green Deployment** on AWS ECS Fargate.  
Infrastructure is fully managed by **Terraform**, and the entire CI/CD pipeline is automated using **GitHub Actions** with manual approval gates.

This project is built as a portfolio piece to demonstrate practical knowledge of modern cloud-native deployment practices.

---

## Project Overview

The system deploys a simple Node.js application using the Blue-Green strategy:

- Two identical environments: **Blue** and **Green**
- Traffic is switched between environments via Application Load Balancer (ALB)
- New versions are always deployed to the idle environment first
- Manual approval is required before switching production traffic
- Old environment is cleaned up after a successful switch

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

- **Blue-Green Deployment** with zero-downtime traffic switching
- **Infrastructure as Code** using Terraform (modular structure)
- **CI/CD Pipeline** with GitHub Actions (Build → Push to ECR → Deploy → Switch Traffic → Cleanup)
- **Manual Approval** using GitHub Environments
- Alternating deployment between Blue and Green environments
- Least-privilege Security Groups and private subnets for ECS tasks

---

## Tech Stack

| Category                  | Technology                        |
|---------------------------|-----------------------------------|
| Cloud Provider            | AWS                               |
| Compute                   | Amazon ECS (Fargate)              |
| Container Registry        | Amazon ECR                        |
| Load Balancer             | Application Load Balancer (ALB)   |
| Infrastructure as Code    | Terraform                         |
| CI/CD                     | GitHub Actions                    |
| Application               | Node.js + Express                 |
| Region                    | ap-southeast-1 (Singapore)        |

---

## Project Structure

```text
.
├── .github/workflows/
│   ├── blue-green-deployment.yml   # Main orchestrator
│   ├── build-docker.yml            # Build & push image to ECR
│   ├── deploy-ecs.yml              # Deploy to Blue or Green cluster
│   ├── switch-traffic.yml          # Switch ALB traffic
│   ├── clear-resources.yml         # Scale down old environment
│   └── test-on-develop.yml         # Basic tests on develop branch
├── architecture/                   # Architecture diagrams
├── legacy/                         # Old files (Jenkins, appspec...)
├── terraform/
│   ├── modules/
│   │   ├── networking/             # VPC, Subnets, Route Tables
│   │   ├── security/               # Security Groups
│   │   ├── load_balance/           # ALB + Target Groups
│   │   └── ecs_cluster/            # ECS Clusters & Services
│   └── singapore-dev/              # Environment configuration
├── Dockerfile
├── index.js
├── package.json
└── README.md
```
## How It Works

1. Developer creates a new version tag
2. GitHub Actions builds the Docker image and pushes it to ECR
3. New version is deployed to the **idle** environment (Blue or Green)
4. Manual approval is required via GitHub Environment
5. ALB traffic is switched (Port 80 ↔ Port 81)
6. Old environment is scaled down to zero

---

## Getting Started

### Prerequisites

- AWS Account
- Terraform >= 1.5
- AWS CLI configured
- Docker
- GitHub Secrets:
  - `AWS_ACCESS_KEY_ID`
  - `AWS_SECRET_ACCESS_KEY`

### 1. Deploy Infrastructure

```bash
cd terraform/singapore-dev
terraform init
terraform plan
terraform apply
```
**Note:** Do not commit `.tfstate` files. Consider using remote state (S3 + DynamoDB) for production.

### 2. Configure GitHub

**Secrets:**
- `AWS_ACCESS_KEY_ID`
- `AWS_SECRET_ACCESS_KEY`

**Variables (recommended):**
- `AWS_REGION`
- `AWS_ACCOUNT_ID`
- `ECR_REPOSITORY`
- `ALB_ARN`
- `BLUE_CLUSTER`
- `GREEN_CLUSTER`
- `TASK_FAMILY`
- `SERVICE_NAME`

### 3. Create GitHub Environments

Create the following environment and enable **Required reviewers**:
- `production-approval`

### 4. Run Deployment

1. Create a version tag (e.g. `v1.0.1`)
2. Go to **Actions → Complete Blue-Green Deployment**
3. Fill in the inputs:
   - `version`: tag name
   - `target_cluster`: `blue` or `green` (idle environment)
   - `switch_traffic`: `true` / `false`
   - `cleanup_old_cluster`: `true` / `false`

---

## Skills Demonstrated

- Blue-Green deployment strategy on AWS ECS
- Infrastructure as Code with Terraform modules
- CI/CD pipeline design with GitHub Actions
- Manual approval gates using GitHub Environments
- ALB traffic switching between Target Groups
- Network design with Public/Private subnets and Security Groups
- Least privilege principle in security configuration

---

## Important Notes

- Always deploy to the idle environment first
- Never commit Terraform state files or secrets
- Use GitHub Environments to protect production traffic switching
- Remote state backend is recommended for real projects

---

## License

This project is licensed under the Apache-2.0 License.