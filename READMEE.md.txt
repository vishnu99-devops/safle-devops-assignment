
# Safle DevOps Assignment

##How to Deploy Infrastructure Using Terraform

1. Install Terraform  
   Download and install Terraform from [terraform.io](https://www.terraform.io/downloads).

2. Clone the Repository
   
   git clone https://github.com/<your-username>/safle-devops-assignment.git
   cd safle-devops-assignment/terraform
   

3. Set Up Your AWS Credentials  
   Either use AWS CLI (`aws configure`) or export environment variables:
   
   export AWS_ACCESS_KEY_ID=your_key
   export AWS_SECRET_ACCESS_KEY=your_secret
   export AWS_DEFAULT_REGION=ap-south-1
   

4. Initialize Terraform
   
   terraform init
   

5. Plan Infrastructure Changes
   
   terraform plan -out=tfplan
   

6. Apply Infrastructure
   
   terraform apply tfplan
   

7. Destroy Infrastructure (Optional)
   
   terraform destroy
   

> Note: The Terraform code is modularized into folders like `vpc/`, `alb/`, `asg/`, `rds/`, `ecr/`, `iam/`, etc.

--------------------------------------------------------------------------------------------------

##  How the CI/CD Pipeline Works

### CI/CD Pipeline Overview

We’re using GitHub Actions for CI/CD with the following stages:

- Test Stage: Runs Jest tests on each push or PR.
- Build Stage: Builds Docker image and pushes it to Amazon ECR.
- Deploy Stage: Deploys image to EC2 instances. Optionally runs `terraform apply` for infra changes.

### Triggering the Pipeline

- Commit and push to the `main` branch.
- Alternatively, open a pull request — the pipeline will automatically trigger.

yaml
on:
  push:
    branches:
      - main
  pull_request:
    branches:
      - main


### Failure Handling

- Rollback logic includes:
  - Retain older version tag in ECR
  - Revert EC2 instance user data
  - Manual `terraform apply` with last known good config
------------------------------------------------------------------------------------------------------------

## Steps to Set Up Monitoring & Alerts

### Metrics

- Amazon CloudWatch collects:
  - EC2 CPU and memory (via CloudWatch Agent)
  - ALB Request count and error rates
  - RDS performance metrics

### Dashboards

- Amazon Managed Grafana (or self-hosted Grafana) pulls metrics from:
  - CloudWatch
  - Prometheus (optional)

### Alerts

- CloudWatch alarms configured for:
  - High CPU (>70%)
  - HTTP 5xx errors (>10%)
  - Unhealthy ALB targets

- Alerts routed via SNS to:
  - Slack webhook
  - Email (subscribers)

## 🧠 Design Decisions, Architecture Trade-offs & Challenges

| Area | Decision | Trade-off / Notes |
|------|----------|--------------------|
| Modular Terraform | Used reusable modules | Easier maintenance, but requires clear variable mapping |
| Instance Type | `t3.micro` for testing | Not production-grade; upgrade required for real load |
| Self-hosted Node.js App on EC2 | Simplicity | In production, would prefer ECS/EKS |
| Secrets Management | AWS Secrets Manager | Secure, but slightly more cost |
| HTTPS | Free SSL via ACM + ALB | Eliminates manual cert handling |
| CI/CD | GitHub Actions | Simpler integration vs Jenkins, but less control in complex orgs |
| RDS | Encrypted + Private subnet | Secure setup, but restricts direct access for debugging |
| Monitoring | CloudWatch + Grafana | Full observability, needs IAM config tuning |



