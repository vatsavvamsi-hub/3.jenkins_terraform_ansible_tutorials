# Terraform Tutorial: From Basics to AWS 3-Tier Architecture

## Table of Contents
1. [What is Terraform?](#what-is-terraform)
2. [Core Architecture](#core-architecture)
3. [Important Files and Configurations](#important-files-and-configurations)
4. [AWS Connectivity Setup](#aws-connectivity-setup)
5. [Use Case: 3-Tier Web Application on AWS](#use-case-3-tier-web-application-on-aws)

---

## What is Terraform?

Terraform is an Infrastructure as Code (IaC) tool that allows you to define and provision infrastructure using declarative configuration files. It's cloud-agnostic and supports multiple providers (AWS, Azure, GCP, etc.).

### Key Benefits
- **Version Control**: Infrastructure code can be versioned and tracked
- **Automation**: Reduces manual errors and deployment time
- **Consistency**: Same configuration produces identical infrastructure
- **Idempotent**: Apply configurations multiple times safely

---

## Core Architecture

```
┌─────────────────────────────────────────────────────────┐
│                    Terraform Core                        │
│  ┌───────────────────────────────────────────────────┐  │
│  │         Configuration Files (.tf)                  │  │
│  │  - Resources, Variables, Outputs, Providers       │  │
│  └───────────────────────────────────────────────────┘  │
│                          │                               │
│                          ▼                               │
│  ┌───────────────────────────────────────────────────┐  │
│  │              Terraform State                       │  │
│  │  (terraform.tfstate - tracks real resources)      │  │
│  └───────────────────────────────────────────────────┘  │
│                          │                               │
│                          ▼                               │
│  ┌───────────────────────────────────────────────────┐  │
│  │           Provider Plugins                         │  │
│  │  (AWS, Azure, GCP, etc.)                          │  │
│  └───────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────┘
                           │
                           ▼
              ┌────────────────────────┐
              │   Cloud Provider API   │
              │        (AWS)           │
              └────────────────────────┘
                           │
                           ▼
              ┌────────────────────────┐
              │   Actual Resources     │
              │  (EC2, VPC, RDS, etc.) │
              └────────────────────────┘
```

### Terraform Workflow

```
┌──────────┐      ┌──────────┐      ┌──────────┐      ┌──────────┐
│terraform │ ───> │terraform │ ───> │terraform │ ───> │terraform │
│   init   │      │   plan   │      │  apply   │      │ destroy  │
└──────────┘      └──────────┘      └──────────┘      └──────────┘
    │                  │                  │                  │
    │                  │                  │                  │
    ▼                  ▼                  ▼                  ▼
Download          Preview           Create/Update      Delete All
Providers         Changes           Resources          Resources
```

---

## Important Files and Configurations

### 1. **main.tf**
The primary configuration file containing resource definitions.

```hcl
# Example main.tf
terraform {
  required_version = ">= 1.0"
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}

provider "aws" {
  region = var.aws_region
}

resource "aws_instance" "example" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t2.micro"
  
  tags = {
    Name = "Example-Instance"
  }
}
```

### 2. **variables.tf**
Defines input variables for reusability and flexibility.

```hcl
variable "aws_region" {
  description = "AWS region for resources"
  type        = string
  default     = "us-east-1"
}

variable "environment" {
  description = "Environment name"
  type        = string
  validation {
    condition     = contains(["dev", "staging", "prod"], var.environment)
    error_message = "Environment must be dev, staging, or prod."
  }
}

variable "instance_count" {
  description = "Number of instances"
  type        = number
  default     = 1
}
```

### 3. **outputs.tf**
Defines output values to display after apply.

```hcl
output "instance_id" {
  description = "ID of the EC2 instance"
  value       = aws_instance.example.id
}

output "public_ip" {
  description = "Public IP address"
  value       = aws_instance.example.public_ip
}
```

### 4. **terraform.tfvars**
Contains actual values for variables (do not commit sensitive data).

```hcl
aws_region    = "us-west-2"
environment   = "dev"
instance_count = 2
```

### 5. **backend.tf** (Optional)
Configures remote state storage.

```hcl
terraform {
  backend "s3" {
    bucket         = "my-terraform-state-bucket"
    key            = "terraform.tfstate"
    region         = "us-east-1"
    encrypt        = true
    dynamodb_table = "terraform-lock"
  }
}
```

### 6. **terraform.tfstate**
Auto-generated state file tracking infrastructure (do not edit manually).

### 7. **.terraform/** directory
Contains downloaded provider plugins (auto-generated).

---

## AWS Connectivity Setup

### Prerequisites
1. AWS Account
2. AWS CLI installed
3. Terraform installed

### Step 1: Install AWS CLI
```bash
# Windows (PowerShell)
msiexec.exe /i https://awscli.amazonaws.com/AWSCLIV2.msi

# Verify installation
aws --version
```

### Step 2: Configure AWS Credentials

**Option A: AWS CLI Configuration**
```bash
aws configure
# Enter:
# - AWS Access Key ID
# - AWS Secret Access Key
# - Default region (e.g., us-east-1)
# - Default output format (json)
```

**Option B: Environment Variables**
```bash
# PowerShell
$env:AWS_ACCESS_KEY_ID="your-access-key"
$env:AWS_SECRET_ACCESS_KEY="your-secret-key"
$env:AWS_DEFAULT_REGION="us-east-1"
```

**Option C: Terraform Provider Configuration**
```hcl
provider "aws" {
  region     = "us-east-1"
  access_key = var.aws_access_key  # Not recommended for production
  secret_key = var.aws_secret_key  # Not recommended for production
}
```

**Best Practice**: Use AWS IAM roles or AWS SSO for production environments.

### Step 3: Initialize Terraform
```bash
terraform init
```

---

## Use Case: 3-Tier Web Application on AWS

### Architecture Overview

```
                        ┌─────────────────┐
                        │   Internet      │
                        │   Gateway       │
                        └────────┬────────┘
                                 │
                     ┌───────────▼──────────┐
                     │  Application Load    │
                     │     Balancer         │
                     └───────────┬──────────┘
                                 │
        ┌────────────────────────┼────────────────────────┐
        │                                                  │
┌───────▼────────┐  Public Subnet          ┌──────────────▼─────┐
│  Web Tier      │  (Availability Zone 1)  │    Web Tier        │
│  EC2 Instance  │                          │   EC2 Instance     │
│  (Nginx)       │                          │   (Nginx)          │
└───────┬────────┘                          └──────────┬─────────┘
        │                                              │
        │          Private Subnet                      │
        │          (Application Tier)                  │
        │                                              │
┌───────▼────────┐                          ┌──────────▼─────────┐
│  App Tier      │  (Availability Zone 1)   │   App Tier         │
│  EC2 Instance  │                          │   EC2 Instance     │
│  (Backend API) │                          │   (Backend API)    │
└───────┬────────┘                          └──────────┬─────────┘
        │                                              │
        │          Private Subnet                      │
        │          (Database Tier)                     │
        │                                              │
        └──────────────────┬───────────────────────────┘
                           │
                  ┌────────▼─────────┐
                  │   RDS PostgreSQL │
                  │   (Multi-AZ)     │
                  └──────────────────┘
```

### Complete Terraform Configuration

#### **Project Structure**
```
terraform-3tier-app/
├── main.tf
├── variables.tf
├── outputs.tf
├── terraform.tfvars
├── vpc.tf
├── security-groups.tf
├── alb.tf
├── web-tier.tf
├── app-tier.tf
└── database-tier.tf
```

#### **variables.tf**
```hcl
variable "aws_region" {
  description = "AWS region"
  type        = string
  default     = "us-east-1"
}

variable "project_name" {
  description = "Project name"
  type        = string
  default     = "3tier-webapp"
}

variable "environment" {
  description = "Environment"
  type        = string
  default     = "dev"
}

variable "vpc_cidr" {
  description = "VPC CIDR block"
  type        = string
  default     = "10.0.0.0/16"
}

variable "availability_zones" {
  description = "Availability zones"
  type        = list(string)
  default     = ["us-east-1a", "us-east-1b"]
}

variable "web_instance_type" {
  description = "Web tier instance type"
  type        = string
  default     = "t2.micro"
}

variable "app_instance_type" {
  description = "App tier instance type"
  type        = string
  default     = "t2.small"
}

variable "db_instance_class" {
  description = "RDS instance class"
  type        = string
  default     = "db.t3.micro"
}

variable "db_name" {
  description = "Database name"
  type        = string
  default     = "appdb"
}

variable "db_username" {
  description = "Database username"
  type        = string
  sensitive   = true
}

variable "db_password" {
  description = "Database password"
  type        = string
  sensitive   = true
}
```

#### **main.tf**
```hcl
terraform {
  required_version = ">= 1.0"
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}

provider "aws" {
  region = var.aws_region

  default_tags {
    tags = {
      Project     = var.project_name
      Environment = var.environment
      ManagedBy   = "Terraform"
    }
  }
}

# Data source for latest Amazon Linux 2 AMI
data "aws_ami" "amazon_linux_2" {
  most_recent = true
  owners      = ["amazon"]

  filter {
    name   = "name"
    values = ["amzn2-ami-hvm-*-x86_64-gp2"]
  }
}
```

#### **vpc.tf**
```hcl
# VPC
resource "aws_vpc" "main" {
  cidr_block           = var.vpc_cidr
  enable_dns_hostnames = true
  enable_dns_support   = true

  tags = {
    Name = "${var.project_name}-vpc"
  }
}

# Internet Gateway
resource "aws_internet_gateway" "main" {
  vpc_id = aws_vpc.main.id

  tags = {
    Name = "${var.project_name}-igw"
  }
}

# Public Subnets (Web Tier)
resource "aws_subnet" "public" {
  count                   = length(var.availability_zones)
  vpc_id                  = aws_vpc.main.id
  cidr_block              = cidrsubnet(var.vpc_cidr, 8, count.index)
  availability_zone       = var.availability_zones[count.index]
  map_public_ip_on_launch = true

  tags = {
    Name = "${var.project_name}-public-subnet-${count.index + 1}"
    Tier = "Web"
  }
}

# Private Subnets (App Tier)
resource "aws_subnet" "private_app" {
  count             = length(var.availability_zones)
  vpc_id            = aws_vpc.main.id
  cidr_block        = cidrsubnet(var.vpc_cidr, 8, count.index + 10)
  availability_zone = var.availability_zones[count.index]

  tags = {
    Name = "${var.project_name}-private-app-subnet-${count.index + 1}"
    Tier = "Application"
  }
}

# Private Subnets (Database Tier)
resource "aws_subnet" "private_db" {
  count             = length(var.availability_zones)
  vpc_id            = aws_vpc.main.id
  cidr_block        = cidrsubnet(var.vpc_cidr, 8, count.index + 20)
  availability_zone = var.availability_zones[count.index]

  tags = {
    Name = "${var.project_name}-private-db-subnet-${count.index + 1}"
    Tier = "Database"
  }
}

# NAT Gateway for private subnets
resource "aws_eip" "nat" {
  count  = length(var.availability_zones)
  domain = "vpc"

  tags = {
    Name = "${var.project_name}-nat-eip-${count.index + 1}"
  }
}

resource "aws_nat_gateway" "main" {
  count         = length(var.availability_zones)
  allocation_id = aws_eip.nat[count.index].id
  subnet_id     = aws_subnet.public[count.index].id

  tags = {
    Name = "${var.project_name}-nat-${count.index + 1}"
  }

  depends_on = [aws_internet_gateway.main]
}

# Route Tables
resource "aws_route_table" "public" {
  vpc_id = aws_vpc.main.id

  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.main.id
  }

  tags = {
    Name = "${var.project_name}-public-rt"
  }
}

resource "aws_route_table" "private" {
  count  = length(var.availability_zones)
  vpc_id = aws_vpc.main.id

  route {
    cidr_block     = "0.0.0.0/0"
    nat_gateway_id = aws_nat_gateway.main[count.index].id
  }

  tags = {
    Name = "${var.project_name}-private-rt-${count.index + 1}"
  }
}

# Route Table Associations
resource "aws_route_table_association" "public" {
  count          = length(var.availability_zones)
  subnet_id      = aws_subnet.public[count.index].id
  route_table_id = aws_route_table.public.id
}

resource "aws_route_table_association" "private_app" {
  count          = length(var.availability_zones)
  subnet_id      = aws_subnet.private_app[count.index].id
  route_table_id = aws_route_table.private[count.index].id
}

resource "aws_route_table_association" "private_db" {
  count          = length(var.availability_zones)
  subnet_id      = aws_subnet.private_db[count.index].id
  route_table_id = aws_route_table.private[count.index].id
}
```

#### **security-groups.tf**
```hcl
# ALB Security Group
resource "aws_security_group" "alb" {
  name        = "${var.project_name}-alb-sg"
  description = "Security group for Application Load Balancer"
  vpc_id      = aws_vpc.main.id

  ingress {
    description = "HTTP from internet"
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  ingress {
    description = "HTTPS from internet"
    from_port   = 443
    to_port     = 443
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = {
    Name = "${var.project_name}-alb-sg"
  }
}

# Web Tier Security Group
resource "aws_security_group" "web" {
  name        = "${var.project_name}-web-sg"
  description = "Security group for web tier"
  vpc_id      = aws_vpc.main.id

  ingress {
    description     = "HTTP from ALB"
    from_port       = 80
    to_port         = 80
    protocol        = "tcp"
    security_groups = [aws_security_group.alb.id]
  }

  ingress {
    description = "SSH from anywhere (restrict in production)"
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = {
    Name = "${var.project_name}-web-sg"
  }
}

# App Tier Security Group
resource "aws_security_group" "app" {
  name        = "${var.project_name}-app-sg"
  description = "Security group for application tier"
  vpc_id      = aws_vpc.main.id

  ingress {
    description     = "API traffic from web tier"
    from_port       = 8080
    to_port         = 8080
    protocol        = "tcp"
    security_groups = [aws_security_group.web.id]
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = {
    Name = "${var.project_name}-app-sg"
  }
}

# Database Security Group
resource "aws_security_group" "db" {
  name        = "${var.project_name}-db-sg"
  description = "Security group for database tier"
  vpc_id      = aws_vpc.main.id

  ingress {
    description     = "PostgreSQL from app tier"
    from_port       = 5432
    to_port         = 5432
    protocol        = "tcp"
    security_groups = [aws_security_group.app.id]
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = {
    Name = "${var.project_name}-db-sg"
  }
}
```

#### **alb.tf**
```hcl
# Application Load Balancer
resource "aws_lb" "main" {
  name               = "${var.project_name}-alb"
  internal           = false
  load_balancer_type = "application"
  security_groups    = [aws_security_group.alb.id]
  subnets            = aws_subnet.public[*].id

  enable_deletion_protection = false

  tags = {
    Name = "${var.project_name}-alb"
  }
}

# Target Group
resource "aws_lb_target_group" "web" {
  name     = "${var.project_name}-web-tg"
  port     = 80
  protocol = "HTTP"
  vpc_id   = aws_vpc.main.id

  health_check {
    enabled             = true
    healthy_threshold   = 2
    interval            = 30
    matcher             = "200"
    path                = "/"
    port                = "traffic-port"
    protocol            = "HTTP"
    timeout             = 5
    unhealthy_threshold = 2
  }

  tags = {
    Name = "${var.project_name}-web-tg"
  }
}

# Listener
resource "aws_lb_listener" "http" {
  load_balancer_arn = aws_lb.main.arn
  port              = "80"
  protocol          = "HTTP"

  default_action {
    type             = "forward"
    target_group_arn = aws_lb_target_group.web.arn
  }
}
```

#### **web-tier.tf**
```hcl
# Launch Template for Web Tier
resource "aws_launch_template" "web" {
  name_prefix   = "${var.project_name}-web-"
  image_id      = data.aws_ami.amazon_linux_2.id
  instance_type = var.web_instance_type

  vpc_security_group_ids = [aws_security_group.web.id]

  user_data = base64encode(<<-EOF
              #!/bin/bash
              yum update -y
              yum install -y nginx
              systemctl start nginx
              systemctl enable nginx
              echo "<h1>Web Tier - $(hostname)</h1>" > /usr/share/nginx/html/index.html
              EOF
  )

  tag_specifications {
    resource_type = "instance"
    tags = {
      Name = "${var.project_name}-web-instance"
      Tier = "Web"
    }
  }
}

# Auto Scaling Group for Web Tier
resource "aws_autoscaling_group" "web" {
  name                = "${var.project_name}-web-asg"
  vpc_zone_identifier = aws_subnet.public[*].id
  target_group_arns   = [aws_lb_target_group.web.arn]
  health_check_type   = "ELB"
  health_check_grace_period = 300
  min_size            = 2
  max_size            = 4
  desired_capacity    = 2

  launch_template {
    id      = aws_launch_template.web.id
    version = "$Latest"
  }

  tag {
    key                 = "Name"
    value               = "${var.project_name}-web-asg"
    propagate_at_launch = true
  }
}
```

#### **app-tier.tf**
```hcl
# Launch Template for App Tier
resource "aws_launch_template" "app" {
  name_prefix   = "${var.project_name}-app-"
  image_id      = data.aws_ami.amazon_linux_2.id
  instance_type = var.app_instance_type

  vpc_security_group_ids = [aws_security_group.app.id]

  user_data = base64encode(<<-EOF
              #!/bin/bash
              yum update -y
              yum install -y docker
              systemctl start docker
              systemctl enable docker
              # Run your application container here
              # docker run -d -p 8080:8080 your-app-image
              EOF
  )

  tag_specifications {
    resource_type = "instance"
    tags = {
      Name = "${var.project_name}-app-instance"
      Tier = "Application"
    }
  }
}

# Auto Scaling Group for App Tier
resource "aws_autoscaling_group" "app" {
  name                = "${var.project_name}-app-asg"
  vpc_zone_identifier = aws_subnet.private_app[*].id
  health_check_type   = "EC2"
  health_check_grace_period = 300
  min_size            = 2
  max_size            = 4
  desired_capacity    = 2

  launch_template {
    id      = aws_launch_template.app.id
    version = "$Latest"
  }

  tag {
    key                 = "Name"
    value               = "${var.project_name}-app-asg"
    propagate_at_launch = true
  }
}
```

#### **database-tier.tf**
```hcl
# DB Subnet Group
resource "aws_db_subnet_group" "main" {
  name       = "${var.project_name}-db-subnet-group"
  subnet_ids = aws_subnet.private_db[*].id

  tags = {
    Name = "${var.project_name}-db-subnet-group"
  }
}

# RDS Instance
resource "aws_db_instance" "main" {
  identifier             = "${var.project_name}-db"
  engine                 = "postgres"
  engine_version         = "15.3"
  instance_class         = var.db_instance_class
  allocated_storage      = 20
  storage_type           = "gp2"
  storage_encrypted      = true
  
  db_name  = var.db_name
  username = var.db_username
  password = var.db_password
  
  db_subnet_group_name   = aws_db_subnet_group.main.name
  vpc_security_group_ids = [aws_security_group.db.id]
  
  multi_az               = true
  backup_retention_period = 7
  skip_final_snapshot    = true
  
  tags = {
    Name = "${var.project_name}-db"
    Tier = "Database"
  }
}
```

#### **outputs.tf**
```hcl
output "vpc_id" {
  description = "VPC ID"
  value       = aws_vpc.main.id
}

output "alb_dns_name" {
  description = "DNS name of the load balancer"
  value       = aws_lb.main.dns_name
}

output "alb_url" {
  description = "URL of the application"
  value       = "http://${aws_lb.main.dns_name}"
}

output "db_endpoint" {
  description = "RDS instance endpoint"
  value       = aws_db_instance.main.endpoint
  sensitive   = true
}

output "web_asg_name" {
  description = "Web tier Auto Scaling Group name"
  value       = aws_autoscaling_group.web.name
}

output "app_asg_name" {
  description = "App tier Auto Scaling Group name"
  value       = aws_autoscaling_group.app.name
}
```

#### **terraform.tfvars** (Example - DO NOT COMMIT)
```hcl
aws_region         = "us-east-1"
project_name       = "myapp"
environment        = "dev"
vpc_cidr           = "10.0.0.0/16"
availability_zones = ["us-east-1a", "us-east-1b"]
web_instance_type  = "t2.micro"
app_instance_type  = "t2.small"
db_instance_class  = "db.t3.micro"
db_name            = "appdb"
db_username        = "admin"
db_password        = "YourSecurePassword123!"  # Use AWS Secrets Manager in production
```

---

## Deployment Steps

### 1. Initialize Terraform
```bash
cd terraform-3tier-app
terraform init
```

### 2. Validate Configuration
```bash
terraform validate
```

### 3. Format Code
```bash
terraform fmt -recursive
```

### 4. Preview Changes
```bash
terraform plan
```

### 5. Apply Configuration
```bash
terraform apply
# Review the plan and type 'yes' to confirm
```

### 6. View Outputs
```bash
terraform output
terraform output alb_url
```

### 7. Access Your Application
```bash
# Get the ALB URL
terraform output alb_url
# Open in browser: http://<alb-dns-name>
```

### 8. Destroy Infrastructure (when done)
```bash
terraform destroy
# Type 'yes' to confirm
```

---

## Best Practices

### 1. **State Management**
- Use remote state (S3 + DynamoDB for locking)
- Never commit `terraform.tfstate` to version control
- Enable state file encryption

### 2. **Security**
- Never hardcode credentials
- Use AWS Secrets Manager or Parameter Store
- Implement least privilege IAM policies
- Enable encryption for all resources

### 3. **Code Organization**
- Split large configurations into multiple files
- Use modules for reusable components
- Follow naming conventions

### 4. **Version Control**
- Use `.gitignore` to exclude sensitive files:
  ```
  .terraform/
  *.tfstate
  *.tfstate.backup
  *.tfvars
  .terraform.lock.hcl
  ```

### 5. **Testing**
- Use `terraform plan` before apply
- Implement automated testing with tools like Terratest
- Use workspaces for multiple environments

---

## Common Commands Reference

```bash
# Initialize working directory
terraform init

# Validate configuration
terraform validate

# Format code
terraform fmt

# Create execution plan
terraform plan

# Apply changes
terraform apply

# Apply without confirmation
terraform apply -auto-approve

# Destroy infrastructure
terraform destroy

# Show current state
terraform show

# List resources in state
terraform state list

# Get specific output
terraform output <output_name>

# Import existing resource
terraform import <resource_type>.<name> <resource_id>

# Refresh state
terraform refresh

# View dependency graph
terraform graph | dot -Tpng > graph.png
```

---

## Troubleshooting

### Issue: "Error acquiring the state lock"
**Solution**: Someone else is running Terraform or previous run didn't complete
```bash
terraform force-unlock <lock-id>
```

### Issue: "Provider configuration not present"
**Solution**: Run `terraform init` first

### Issue: Resource already exists
**Solution**: Import existing resource or use `terraform import`

### Issue: Authentication failures
**Solution**: Verify AWS credentials with `aws sts get-caller-identity`

---

## Additional Resources

- [Terraform Documentation](https://www.terraform.io/docs)
- [AWS Provider Documentation](https://registry.terraform.io/providers/hashicorp/aws/latest/docs)
- [Terraform Best Practices](https://www.terraform-best-practices.com/)
- [HashiCorp Learn](https://learn.hashicorp.com/terraform)

---

## Conclusion

This tutorial covered:
- ✅ Terraform architecture and workflow
- ✅ Essential file structure and configurations
- ✅ AWS connectivity setup
- ✅ Complete 3-tier web application deployment

You now have a production-ready template for deploying scalable, highly-available web applications on AWS using Terraform!
