# Infrastructure Best Practices

## Overview

This document outlines best practices for infrastructure management, deployment, and maintenance in the PresalesWork Infrastructure repository.

## General Principles

### 1. Infrastructure as Code (IaC)

**Benefits:**
- Version controlled infrastructure
- Reproducible deployments
- Documentation through code
- Automated testing

**Best Practices:**
```hcl
# Good: Descriptive resource names
resource "aws_instance" "web_server" {
  ami           = var.ami_id
  instance_type = var.instance_type
  
  tags = {
    Name        = "${var.environment}-web-server"
    Environment = var.environment
    ManagedBy   = "Terraform"
  }
}

# Bad: Generic names, hardcoded values
resource "aws_instance" "instance1" {
  ami           = "ami-12345"
  instance_type = "t2.micro"
}
```

### 2. Separation of Concerns

**Environment Separation:**
```
environments/
├── dev/
│   ├── main.tf
│   └── variables.tfvars
├── staging/
│   ├── main.tf
│   └── variables.tfvars
└── production/
    ├── main.tf
    └── variables.tfvars
```

### 3. DRY (Don't Repeat Yourself)

**Use Modules:**
```hcl
# Module definition
module "vpc" {
  source = "./modules/vpc"
  
  vpc_cidr     = var.vpc_cidr
  environment  = var.environment
  region       = var.region
}

# Reuse across environments
module "prod_vpc" {
  source      = "./modules/vpc"
  environment = "production"
}

module "dev_vpc" {
  source      = "./modules/vpc"
  environment = "development"
}
```

## Security Best Practices

### 1. Secrets Management

**Never commit secrets:**
```bash
# Use environment variables
export DB_PASSWORD="secure_password"

# Use secret management services
aws secretsmanager get-secret-value --secret-id db-password

# Use parameter stores
aws ssm get-parameter --name /app/db/password --with-decryption
```

**Good Practice:**
```hcl
# Retrieve from secrets manager
data "aws_secretsmanager_secret_version" "db_password" {
  secret_id = "database/password"
}

resource "aws_db_instance" "main" {
  password = data.aws_secretsmanager_secret_version.db_password.secret_string
}
```

### 2. Least Privilege Access

**IAM Policy Example:**
```json
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Allow",
    "Action": [
      "s3:GetObject",
      "s3:ListBucket"
    ],
    "Resource": [
      "arn:aws:s3:::specific-bucket/*",
      "arn:aws:s3:::specific-bucket"
    ]
  }]
}
```

### 3. Encryption

**Enable encryption everywhere:**
```hcl
# S3 bucket encryption
resource "aws_s3_bucket_server_side_encryption_configuration" "bucket" {
  bucket = aws_s3_bucket.main.id

  rule {
    apply_server_side_encryption_by_default {
      sse_algorithm = "AES256"
    }
  }
}

# RDS encryption
resource "aws_db_instance" "main" {
  storage_encrypted = true
  kms_key_id        = aws_kms_key.db.arn
}
```

### 4. Network Security

**Security Groups:**
```hcl
# Restrictive ingress
resource "aws_security_group" "web" {
  name = "web-sg"

  ingress {
    from_port   = 443
    to_port     = 443
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  # Default deny - don't open all ports
  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
}
```

## Code Quality

### 1. Formatting

```bash
# Terraform
terraform fmt -recursive

# YAML
yamllint .

# JSON
jq . file.json
```

### 2. Validation

```bash
# Terraform validation
terraform validate

# Terraform security scan
tfsec .

# Cloud formation validation
aws cloudformation validate-template --template-body file://template.yaml
```

### 3. Linting

```bash
# Terraform
tflint

# Ansible
ansible-lint playbook.yml

# CloudFormation
cfn-lint template.yaml
```

## State Management

### 1. Remote State

**Configure S3 backend:**
```hcl
terraform {
  backend "s3" {
    bucket         = "terraform-state-prod"
    key            = "infrastructure/terraform.tfstate"
    region         = "us-east-1"
    encrypt        = true
    dynamodb_table = "terraform-locks"
  }
}
```

### 2. State Locking

**DynamoDB table for locking:**
```hcl
resource "aws_dynamodb_table" "terraform_locks" {
  name         = "terraform-locks"
  billing_mode = "PAY_PER_REQUEST"
  hash_key     = "LockID"

  attribute {
    name = "LockID"
    type = "S"
  }
}
```

### 3. State File Security

- Enable versioning on state bucket
- Enable encryption at rest
- Restrict access with IAM policies
- Regular backups

## Resource Naming

### Convention

```
{environment}-{application}-{resource}-{region}
```

### Examples

```hcl
# Good naming
resource "aws_instance" "prod_webapp_server_useast1" {}
resource "aws_s3_bucket" "dev_logs_bucket_uswest2" {}
resource "azure_resource_group" "staging_app_rg_eastus" {}

# Avoid generic names
resource "aws_instance" "server1" {}
resource "aws_s3_bucket" "bucket" {}
```

## Tagging Strategy

### Required Tags

```hcl
locals {
  common_tags = {
    Environment  = var.environment
    Project      = "PresalesWork"
    ManagedBy    = "Terraform"
    Owner        = var.owner
    CostCenter   = var.cost_center
    Compliance   = var.compliance_level
    CreatedDate  = formatdate("YYYY-MM-DD", timestamp())
  }
}

resource "aws_instance" "web" {
  tags = merge(
    local.common_tags,
    {
      Name = "${var.environment}-web-server"
      Role = "WebServer"
    }
  )
}
```

## High Availability

### Multi-AZ Deployment

```hcl
# Distribute across availability zones
resource "aws_instance" "web" {
  count             = 3
  availability_zone = element(data.aws_availability_zones.available.names, count.index)
  
  tags = {
    Name = "web-${count.index + 1}"
  }
}
```

### Auto Scaling

```hcl
resource "aws_autoscaling_group" "web" {
  min_size         = 2
  max_size         = 10
  desired_capacity = 3
  
  health_check_type         = "ELB"
  health_check_grace_period = 300
  
  vpc_zone_identifier = var.subnet_ids
}
```

## Monitoring and Logging

### CloudWatch Alarms

```hcl
resource "aws_cloudwatch_metric_alarm" "high_cpu" {
  alarm_name          = "${var.environment}-high-cpu"
  comparison_operator = "GreaterThanThreshold"
  evaluation_periods  = "2"
  metric_name         = "CPUUtilization"
  namespace           = "AWS/EC2"
  period              = "300"
  statistic           = "Average"
  threshold           = "80"
  
  alarm_actions = [aws_sns_topic.alerts.arn]
}
```

### Centralized Logging

```hcl
resource "aws_cloudwatch_log_group" "app" {
  name              = "/aws/app/${var.environment}"
  retention_in_days = 30
  
  tags = local.common_tags
}
```

## Cost Optimization

### 1. Right-Sizing

- Monitor resource utilization
- Use appropriate instance types
- Implement auto-scaling

### 2. Reserved Instances

```hcl
# For predictable workloads
# Purchase reserved instances for 1-3 years
```

### 3. Spot Instances

```hcl
resource "aws_spot_instance_request" "worker" {
  ami           = var.ami_id
  instance_type = var.instance_type
  spot_price    = "0.05"
  
  tags = {
    Name = "spot-worker"
  }
}
```

### 4. Resource Cleanup

```bash
# Schedule cleanup of unused resources
# Tag resources with expiration dates
# Implement automated cleanup scripts
```

## Documentation

### 1. Code Comments

```hcl
# Good: Explain why, not what
# Using t3.medium to handle expected traffic of 1000 req/sec
instance_type = "t3.medium"

# Bad: Obvious statement
# This sets the instance type
instance_type = "t3.medium"
```

### 2. README Files

Include in each module:
- Purpose and overview
- Prerequisites
- Usage examples
- Input variables
- Output values
- Dependencies

### 3. Diagrams

Use tools like:
- draw.io
- Lucidchart
- CloudCraft
- Architecture diagrams in documentation

## Testing

### 1. Pre-deployment

```bash
# Format check
terraform fmt -check

# Validation
terraform validate

# Plan review
terraform plan
```

### 2. Post-deployment

```bash
# Verify resources
terraform show

# Test connectivity
curl https://endpoint.example.com

# Check logs
aws logs tail /aws/app/production --follow
```

### 3. Automated Testing

```bash
# Terratest example
go test -v -timeout 30m
```

## Version Control

### 1. Git Workflow

```bash
# Feature branch
git checkout -b feature/new-vpc-module

# Make changes
git add .
git commit -m "feat: Add VPC module with public/private subnets"

# Push and create PR
git push origin feature/new-vpc-module
```

### 2. Commit Messages

Follow conventional commits:
```
feat: Add new feature
fix: Bug fix
docs: Documentation changes
refactor: Code refactoring
test: Add tests
chore: Maintenance
```

### 3. Versioning

```bash
# Tag releases
git tag -a v1.0.0 -m "Initial release"
git push origin v1.0.0
```

## Disaster Recovery

### 1. Backup Strategy

- Regular state file backups
- Database snapshots
- S3 versioning enabled
- Cross-region replication

### 2. Recovery Plan

- Document recovery procedures
- Test recovery regularly
- Maintain runbooks
- Define RTO/RPO

## Compliance

### 1. Audit Logging

Enable CloudTrail, Azure Monitor, or GCP Cloud Audit Logs

### 2. Compliance Checks

```bash
# AWS Config rules
# Azure Policy
# GCP Organization Policy
```

### 3. Regular Reviews

- Quarterly security audits
- Compliance verification
- Cost reviews
- Architecture reviews

## Performance

### 1. Resource Optimization

- Use CDN for static content
- Implement caching strategies
- Optimize database queries
- Use read replicas

### 2. Monitoring

- Set up performance dashboards
- Monitor key metrics
- Alert on anomalies
- Regular performance reviews

## Conclusion

Following these best practices ensures:
- Secure infrastructure
- Cost optimization
- High availability
- Easy maintenance
- Compliance adherence

## Additional Resources

- [AWS Well-Architected Framework](https://aws.amazon.com/architecture/well-architected/)
- [Terraform Best Practices](https://www.terraform-best-practices.com/)
- [Azure Cloud Adoption Framework](https://docs.microsoft.com/en-us/azure/cloud-adoption-framework/)
- [Google Cloud Best Practices](https://cloud.google.com/docs/enterprise/best-practices-for-enterprise-organizations)
