# Terraform Basics Tutorial

## Table of Contents
1. [What is Terraform?](#what-is-terraform)
2. [Core Concepts](#core-concepts)
3. [Installation & Setup](#installation--setup)
4. [Basic Workflow](#basic-workflow)
5. [Configuration Fundamentals](#configuration-fundamentals)
6. [State Management](#state-management)
7. [Best Practices](#best-practices)

---

## What is Terraform?

Terraform is an Infrastructure as Code (IaC) tool that allows you to define, preview, and deploy cloud infrastructure using configuration files. It supports multiple cloud providers (AWS, Azure, GCP, etc.) and on-premises infrastructure.

### Key Features
- **Declarative**: Describe desired infrastructure state
- **Multi-cloud**: Work across different providers
- **Version controlled**: Track infrastructure changes
- **Reproducible**: Consistent deployments across environments

---

## Core Concepts

### 1. **Providers**
External APIs/services that Terraform manages

```
┌─────────────────────────────────────────┐
│         Terraform Configuration         │
├─────────────────────────────────────────┤
│                                         │
│  terraform {                            │
│    required_providers {                 │
│      aws = { ... }                      │
│      azure = { ... }                    │
│    }                                    │
│  }                                      │
│                                         │
└──────────┬──────────────────────┬───────┘
           │                      │
    ┌──────▼────┐         ┌──────▼─────┐
    │ AWS Cloud │         │Azure Cloud │
    └───────────┘         └────────────┘
```

### 2. **Resources**
Infrastructure components managed by Terraform (e.g., EC2 instances, databases)

### 3. **State**
Terraform's record of your infrastructure. Stored locally or remotely.

### 4. **Variables & Outputs**
- **Variables**: Input values for your configuration
- **Outputs**: Values returned after deployment

---

## Installation & Setup

### Windows Installation (PowerShell)

```powershell
# Using Chocolatey
choco install terraform

# Or download from https://www.terraform.io/downloads
# Then add to PATH
```

### Verify Installation
```powershell
terraform --version
terraform -help
```

---

## Basic Workflow

```
┌─────────────────────────────────────────────────────────┐
│                  Terraform Workflow                     │
└─────────────────────────────────────────────────────────┘

    ┌──────────┐
    │  Write   │  Create/edit .tf configuration files
    └────┬─────┘
         │
    ┌────▼───────┐
    │   Init     │  terraform init - Initialize working directory
    └────┬───────┘
         │
    ┌────▼──────────┐
    │   Validate    │  terraform validate - Check syntax
    └────┬──────────┘
         │
    ┌────▼──────────┐
    │    Plan       │  terraform plan - Preview changes
    └────┬──────────┘
         │
    ┌────▼──────────┐
    │    Apply      │  terraform apply - Deploy infrastructure
    └────┬──────────┘
         │
    ┌────▼──────────┐
    │    Manage     │  Monitor and update as needed
    └──────────────┘
```

---

## Configuration Fundamentals

### Project Structure

```
my-terraform-project/
├── main.tf           # Primary configuration
├── variables.tf      # Variable declarations
├── outputs.tf        # Output declarations
├── terraform.tfvars  # Variable values (local dev)
└── providers.tf      # Provider configuration
```

### 1. **Provider Configuration** (providers.tf)

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
  region = "us-east-1"
}
```

### 2. **Define Variables** (variables.tf)

```hcl
variable "instance_type" {
  description = "EC2 instance type"
  type        = string
  default     = "t2.micro"
}

variable "tags" {
  description = "Common tags"
  type        = map(string)
  default = {
    Environment = "dev"
    Project     = "demo"
  }
}
```

### 3. **Create Resources** (main.tf)

```hcl
resource "aws_instance" "web" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = var.instance_type

  tags = merge(
    var.tags,
    { Name = "web-server" }
  )
}

resource "aws_security_group" "web" {
  name = "web-sg"

  ingress {
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
}
```

### 4. **Output Values** (outputs.tf)

```hcl
output "instance_id" {
  description = "ID of the EC2 instance"
  value       = aws_instance.web.id
}

output "public_ip" {
  description = "Public IP of the instance"
  value       = aws_instance.web.public_ip
}
```

### 5. **Provide Variable Values** (terraform.tfvars)

```hcl
instance_type = "t2.small"
tags = {
  Environment = "production"
  Project     = "webapp"
}
```

---

## State Management

### Local State (Development)

```
┌──────────────────────┐
│  .terraform/         │
│  ├── terraform.tfstate     (Local state file)
│  ├── .terraform.lock.hcl   (Dependency lock)
│  └── providers/            (Downloaded providers)
└──────────────────────┘
```

### Remote State (Production - Best Practice)

```
┌──────────────────────────────────────────────────────────┐
│                      Local Machine                       │
│  ┌────────────────────────────────────────────────────┐  │
│  │  Terraform Configuration & State Reference        │  │
│  └────────────────────────────────────────────────────┘  │
└────────────────────┬─────────────────────────────────────┘
                     │
                     │ Remote Backend
                     │
        ┌────────────▼──────────────┐
        │   AWS S3 Bucket           │
        │ (Centralized State Store) │
        └───────────────────────────┘
```

### Configure Remote State (main.tf)

```hcl
terraform {
  backend "s3" {
    bucket         = "my-terraform-state"
    key            = "prod/terraform.tfstate"
    region         = "us-east-1"
    encrypt        = true
    dynamodb_table = "terraform-locks"
  }
}
```

---

## Common Commands

| Command | Purpose |
|---------|---------|
| `terraform init` | Initialize working directory |
| `terraform validate` | Validate configuration syntax |
| `terraform plan` | Preview changes |
| `terraform apply` | Apply changes to infrastructure |
| `terraform destroy` | Destroy infrastructure |
| `terraform state list` | List resources in state |
| `terraform state show <resource>` | Show resource details |
| `terraform fmt` | Format configuration files |
| `terraform taint <resource>` | Mark resource for recreation |

---

## Hands-On Example: Deploy an AWS EC2 Instance

### Step 1: Create Project Directory

```powershell
mkdir my-terraform-project
cd my-terraform-project
```

### Step 2: Create providers.tf

```hcl
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}

provider "aws" {
  region = "us-east-1"
}
```

### Step 3: Create variables.tf

```hcl
variable "instance_type" {
  description = "EC2 instance type"
  type        = string
  default     = "t2.micro"
}

variable "environment" {
  description = "Environment name"
  type        = string
  default     = "dev"
}
```

### Step 4: Create main.tf

```hcl
# Get the latest Ubuntu AMI
data "aws_ami" "ubuntu" {
  most_recent = true
  owners      = ["099720109477"] # Canonical

  filter {
    name   = "name"
    values = ["ubuntu/images/hvm-ssd/ubuntu-focal-20.04-amd64-server-*"]
  }
}

# Create EC2 Instance
resource "aws_instance" "web" {
  ami           = data.aws_ami.ubuntu.id
  instance_type = var.instance_type

  tags = {
    Name        = "terraform-demo"
    Environment = var.environment
  }
}
```

### Step 5: Create outputs.tf

```hcl
output "instance_id" {
  value       = aws_instance.web.id
  description = "EC2 Instance ID"
}

output "public_ip" {
  value       = aws_instance.web.public_ip
  description = "Public IP Address"
}
```

### Step 6: Deploy

```powershell
# Initialize Terraform
terraform init

# Validate configuration
terraform validate

# Preview changes
terraform plan

# Apply changes (type 'yes' when prompted)
terraform apply

# View outputs
terraform output
```

### Step 7: Cleanup

```powershell
# Destroy infrastructure
terraform destroy
```

---

## Best Practices

### 1. **Variable Naming & Organization**

✅ Good:
```hcl
variable "app_instance_count" { ... }
variable "app_environment" { ... }
```

❌ Avoid:
```hcl
variable "count" { ... }
variable "env" { ... }
```

### 2. **Use Modules for Reusability**

```
modules/
├── vpc/
│   ├── main.tf
│   ├── variables.tf
│   └── outputs.tf
├── security_group/
│   ├── main.tf
│   ├── variables.tf
│   └── outputs.tf
```

Usage:
```hcl
module "vpc" {
  source = "./modules/vpc"
  cidr   = "10.0.0.0/16"
}
```

### 3. **Environment Separation**

```
environments/
├── dev/
│   ├── terraform.tfvars
│   └── main.tf
├── staging/
│   ├── terraform.tfvars
│   └── main.tf
└── prod/
    ├── terraform.tfvars
    └── main.tf
```

### 4. **.gitignore for Terraform**

```
# Local .terraform directories
**/.terraform/*

# .tfstate files
*.tfstate*

# Crash log files
crash.log
crash.*.log

# Exclude all .tfvars files (except terraform.tfvars)
*.tfvars
!terraform.tfvars

# Ignore override files
override.tf
override.tf.json
*_override.tf
*_override.tf.json

# Ignore CLI configuration files
.terraformrc
terraform.rc

# Mac specific
.DS_Store
```

### 5. **Use State Locking**

```hcl
terraform {
  backend "s3" {
    bucket         = "my-state"
    dynamodb_table = "terraform-locks"
    encrypt        = true
  }
}
```

### 6. **Data Sources for Dynamic Values**

```hcl
# Get existing VPC
data "aws_vpc" "default" {
  default = true
}

# Use in resource
resource "aws_security_group" "example" {
  vpc_id = data.aws_vpc.default.id
}
```

### 7. **Code Formatting & Linting**

```powershell
# Format code
terraform fmt -recursive

# Validate syntax
terraform validate

# Lint with tflint (install separately)
tflint --init
tflint
```

---

## Resource Types by Provider

### AWS Common Resources

| Resource | Type |
|----------|------|
| Compute | `aws_instance`, `aws_launch_template`, `aws_autoscaling_group` |
| Networking | `aws_vpc`, `aws_subnet`, `aws_security_group`, `aws_route_table` |
| Storage | `aws_s3_bucket`, `aws_ebs_volume` |
| Database | `aws_db_instance`, `aws_dynamodb_table` |
| Load Balancing | `aws_lb`, `aws_lb_target_group` |

### Azure Common Resources

| Resource | Type |
|----------|------|
| Compute | `azurerm_virtual_machine`, `azurerm_windows_virtual_machine` |
| Networking | `azurerm_virtual_network`, `azurerm_subnet`, `azurerm_network_security_group` |
| Storage | `azurerm_storage_account` |
| Database | `azurerm_mssql_server`, `azurerm_cosmosdb_account` |

---

## Troubleshooting

### Issue: "Unauthorized: AWS credentials not found"

```powershell
# Configure AWS credentials
aws configure

# Or set environment variables
$env:AWS_ACCESS_KEY_ID = "your-key"
$env:AWS_SECRET_ACCESS_KEY = "your-secret"
```

### Issue: "Resource already exists"

```powershell
# Import existing resource into state
terraform import aws_instance.web i-1234567890abcdef0

# Or use taint to mark for recreation
terraform taint aws_instance.web
terraform apply
```

### Issue: "State lock timeout"

State is locked by another process. Check for running operations:
```powershell
terraform force-unlock <LOCK_ID>
```

---

## Next Steps

1. **Learn Modules**: Build reusable infrastructure components
2. **Explore Workspaces**: Manage multiple environments
3. **Master Providers**: Deep dive into your specific cloud provider
4. **CI/CD Integration**: Automate deployments with GitHub Actions, GitLab CI, etc.
5. **Policy as Code**: Use Sentinel or OPA for compliance
6. **Terraform Cloud**: Centralized state management and team collaboration

---

## Resources

- [Official Terraform Documentation](https://www.terraform.io/docs)
- [Terraform AWS Provider](https://registry.terraform.io/providers/hashicorp/aws/latest/docs)
- [Terraform Registry](https://registry.terraform.io/) - Modules and providers
- [HashiCorp Learn](https://learn.hashicorp.com/terraform) - Official tutorials

---

**Happy Terraforming! 🚀**
