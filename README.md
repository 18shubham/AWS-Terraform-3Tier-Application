# AWS Three-Tier Web Application Infrastructure (Terraform)

This repository contains a **complete, production-grade, highly-available three-tier web application** built 100 % with **Terraform IaC** in under 6 minutes.

Deployed and verified live on **04-Dec-2025** – from zero to fully working in **one command**.

## Resume Bullet (Copy-Paste This – 60–100 LPA Level)
> Designed and deployed a fully automated three-tier web application on AWS using Terraform IaC featuring custom VPC with 6 subnets across 3 AZs, internet-facing ALB, Auto Scaling Group with dynamic AMI lookup, Multi-AZ RDS MySQL in private subnets, security-group-only database access, S3 remote state with DynamoDB locking — entire stack provisioned in < 6 minutes with `terraform apply -auto-approve`.

## Architecture Overview (Exactly What Companies Run in 2025)

Internet
↓ (HTTP 80)
Application Load Balancer (Public Subnets – 3 AZs)
↓
Target Group → Health Checks
↓
Auto Scaling Group (Public Subnets)
├── Launch Template + User Data (Apache + PHP)
└── Dynamic latest Amazon Linux 2023 AMI
↓ (Port 3306 – SG only)
Multi-AZ RDS MySQL (Private Subnets)
├── Primary + Standby (automatic failover)
└── Secured with least-privilege SG


## Infrastructure Components (All Created by Terraform)

| Component                    | Resource Name                  | Location            | Key Feature                          |
|------------------------------|--------------------------------|---------------------|--------------------------------------|
| VPC + 6 Subnets + NAT        | `module.vpc`                   | 3 AZs               | Official AWS module                  |
| Web Security Group           | `aws_security_group.web`      | Public              | Port 80 open                         |
| RDS Security Group           | `aws_security_group.rds`       | Private             | 3306 only from web SG                |
| Application Load Balancer   | `aws_lb.main`                  | Public Subnets      | Internet-facing                      |
| Target Group                 | `aws_lb_target_group.web`      | –                   | Health check on /                    |
| Launch Template              | `aws_launch_template.web`      | –                   | Dynamic AMI + User Data              |
| Auto Scaling Group           | `aws_autoscaling_group.web`    | Public Subnets      | Min 2 → Max 4, auto-healing          |
| Multi-AZ RDS MySQL           | `aws_db_instance.main`         | Private Subnets     | Automatic failover + no public access|

## Terraform Files (Professional Structure)
aws-30days-terraform/
├── backend.tf          → S3 + DynamoDB remote state (production-ready)
├── provider.tf         → AWS provider + profile day1
├── variables.tf        → Sensitive variables
├── main.tf             → Entire 3-tier stack
├── outputs.tf          → ALB DNS & RDS endpoint
└── .gitignore          → Excludes .terraform & state files


## Live Proof (Working Right Now!)
- **ALB URL:** http://day5-alb-197928404.ap-south-1.elb.amazonaws.com
- **RDS Endpoint:** myappdb-tf.clyy6gyum9ld.ap-south-1.rds.amazonaws.com:3306

## One-Command Deployment
```bash
terraform init -upgrade -reconfigure
terraform apply -auto-approve     # ← 4–6 minutes → FULL STACK LIVE
***
terraform destroy -auto-approve
