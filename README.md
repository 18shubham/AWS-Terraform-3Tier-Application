# AWS-Terraform-3Tier-Application
AWS Three-Tier Web Application Infrastructure (Terraform)

This project defines a highly available, secure, and scalable three-tier web application architecture on Amazon Web Services (AWS) using Terraform.

The infrastructure is designed according to AWS best practices, isolating sensitive components (like the database and application servers) in private subnets, while using an Application Load Balancer (ALB) to handle public traffic.

1. Architecture Overview

The system is logically divided into three primary tiers across multiple Availability Zones (AZs) for resilience.

Tier

Component

Location

Purpose

Presentation (Web)

Application Load Balancer (ALB)

Public Subnets

Receives internet traffic on Port 80 and forwards it.

Application (Logic)

EC2 Auto Scaling Group (ASG)

Private App Subnets

Runs the application code (e.g., Node.js, Python) on Port 3000.

Data (Persistence)

RDS Database (MySQL)

Private Data Subnets

Stores application data. Secured with least-privilege access.

Networking

VPC, NAT Gateway

All Subnets

Provides network isolation, routing, and outbound internet access for private resources.

2. Infrastructure Components (Terraform Files)

The architecture is built across five distinct Terraform files, ensuring modularity and clarity.

Phase 1: Networking & Security (network.tf & security.tf)

This phase lays the foundation for all other services.

network.tf Key Resources:

aws_vpc.main: The core isolated virtual network.

aws_subnet: Six subnets in total (2 Public Web, 2 Private App, 2 Private Data), spread across two Availability Zones.

aws_internet_gateway: Allows public subnets to reach the internet.

aws_nat_gateway: Allows private subnets to initiate outbound connections (e.g., software updates).

security.tf Key Resources:

aws_security_group.db_sg: Restricts MySQL (Port 3306) access only to the App Tier.

aws_security_group.app_sg: Allows incoming HTTP traffic from the ALB (Port 3000) and allows SSH/ICMP from trusted sources.

Phase 2: Data Tier (data_tier.tf)

The database layer, securely isolated in the private data subnets.

Key Resources:

aws_db_subnet_group: Defines which subnets the RDS database can use (the Private Data Subnets).

aws_db_instance.app_db: The RDS MySQL instance, automatically placed across multiple AZs for High Availability (HA) and protected by the db_sg.

Phase 3: Application Tier (app_tier.tf)

The middleware running the business logic, located in private application subnets.

Key Resources:

aws_launch_template.app_lt: The "cookie cutter" defining the configuration (OS, instance type, startup script) for app servers.

aws_autoscaling_group.app_asg: The "manager" that maintains a desired capacity of application servers (e.g., 2 instances) and performs automatic healing.

Startup Script: Installs basic tooling and simulates a backend application listening on Port 3000.

Phase 4: Presentation Tier & Load Balancing (web_tier.tf)

The public entry point, distributing traffic to the private application layer.

Key Resources:

aws_lb.web_alb: The Application Load Balancer (ALB), deployed in the Public Web Subnets.

aws_lb_target_group.app_tg: Defines the target for the ALB, configured to perform health checks and forward traffic to the App Tier's internal Port 3000.

aws_lb_listener.http_listener: Listens on public Port 80 and routes requests to the app_tg.

aws_autoscaling_attachment: Explicitly connects the app_asg instances to the app_tg, completing the request path.

(Optional) aws_autoscaling_group.web_asg: A separate ASG for static web content (e.g., Nginx) in the Public Subnets, providing redundancy.

3. Prerequisites

Before deployment, ensure you have the following installed and configured:

Terraform: Install Terraform (Version 1.0+ recommended).

AWS CLI: Configure your AWS credentials with sufficient permissions to create VPCs, EC2 instances, RDS, etc.

Variable Definitions: You must create a terraform.tfvars file to provide the necessary variables (which are defined in your original files, e.g., VPC CIDR, database credentials).

Example terraform.tfvars (Hypothetical)

# Define variables used throughout the Terraform files
db_username = "admin"
db_password = "SecurePassword123"
vpc_cidr = "10.0.0.0/16"


4. Deployment Instructions

Follow these steps to deploy the entire three-tier infrastructure:

Initialize Terraform:

terraform init


Validate Configuration:

terraform validate


Review the Plan: Review all resources that will be created before proceeding.

terraform plan


Apply the Configuration: This will provision all resources in your AWS account.

terraform apply


Access the Application: Once the apply is complete, Terraform will output the DNS name of the Application Load Balancer (ALB). Open this DNS name in your web browser.

The request flow will be:

Browser -> ALB (Public Port 80) -> App Tier Server (Private Port 3000)

The browser should display the message "Hello from the App Tier! I am connected to the DB." (This confirms the ALB, Target Group, and App ASG are all working together).

5. Cleanup

To avoid ongoing charges, destroy all resources when you are finished:

terraform destroy
