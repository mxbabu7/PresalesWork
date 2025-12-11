# Deployment Guide

## Overview

This guide provides step-by-step instructions for deploying infrastructure using the templates and scripts in this repository.

## Prerequisites

### Required Tools

Install the following tools before deployment:

- **Git**: Version control
  ```bash
  # Verify installation
  git --version
  ```

- **Cloud Provider CLI**:
  - AWS CLI: `aws --version`
  - Azure CLI: `az --version`
  - gcloud CLI: `gcloud --version`

- **Terraform** (if using Terraform templates):
  ```bash
  # Install Terraform
  # Visit: https://www.terraform.io/downloads
  terraform --version
  ```

- **Docker** (for containerized deployments):
  ```bash
  docker --version
  ```

### Cloud Provider Setup

#### AWS Setup

1. Create an AWS account
2. Configure AWS CLI:
   ```bash
   aws configure
   ```
3. Set up credentials:
   - Access Key ID
   - Secret Access Key
   - Default region
   - Output format

#### Azure Setup

1. Create an Azure account
2. Login to Azure:
   ```bash
   az login
   ```
3. Set subscription:
   ```bash
   az account set --subscription "Your Subscription Name"
   ```

#### GCP Setup

1. Create a GCP project
2. Authenticate:
   ```bash
   gcloud auth login
   gcloud config set project PROJECT_ID
   ```

## Deployment Process

### Step 1: Clone Repository

```bash
git clone https://github.com/mxbabu7/PresalesWork.git
cd PresalesWork
```

### Step 2: Choose Template

Navigate to the appropriate template directory:

```bash
# For AWS
cd templates/aws

# For Azure
cd templates/azure

# For GCP
cd templates/gcp
```

### Step 3: Configure Variables

Create a variables file based on your requirements:

```bash
# Copy example variables
cp example.tfvars terraform.tfvars

# Edit with your values
vim terraform.tfvars
```

Example `terraform.tfvars`:
```hcl
region           = "us-east-1"
environment      = "production"
instance_type    = "t3.medium"
enable_monitoring = true
```

### Step 4: Terraform Deployment

#### Initialize Terraform

```bash
terraform init
```

This command:
- Downloads provider plugins
- Initializes backend
- Prepares working directory

#### Plan Deployment

```bash
terraform plan -out=tfplan
```

Review the plan to ensure:
- Resources to be created
- Resources to be modified
- Resources to be destroyed

#### Apply Changes

```bash
terraform apply tfplan
```

Confirm when prompted. This will:
- Create infrastructure resources
- Configure networking
- Set up security groups
- Deploy applications

#### Verify Deployment

```bash
terraform show
terraform output
```

### Step 5: Post-Deployment Tasks

#### Configure Monitoring

Set up monitoring and alerting:
```bash
./scripts/setup-monitoring.sh
```

#### Run Health Checks

Verify all services are running:
```bash
./scripts/health-check.sh
```

#### Document Outputs

Save important outputs:
```bash
terraform output > deployment-outputs.txt
```

## Deployment Strategies

### Development Environment

```bash
# Quick deployment for testing
terraform apply -var="environment=dev" -auto-approve
```

### Staging Environment

```bash
# Deploy to staging
terraform workspace new staging
terraform apply -var-file="staging.tfvars"
```

### Production Environment

```bash
# Production deployment (with approval)
terraform workspace new production
terraform plan -var-file="production.tfvars" -out=prod.tfplan

# Review plan carefully
terraform show prod.tfplan

# Apply after review
terraform apply prod.tfplan
```

## Common Deployment Scenarios

### Scenario 1: Web Application

```bash
cd templates/aws/web-app
terraform init
terraform plan -var-file="web-app.tfvars"
terraform apply
```

### Scenario 2: Kubernetes Cluster

```bash
cd templates/aws/eks-cluster
terraform init
terraform plan -var-file="cluster.tfvars"
terraform apply

# Configure kubectl
aws eks update-kubeconfig --name cluster-name
```

### Scenario 3: Database Infrastructure

```bash
cd templates/aws/rds
terraform init
terraform plan -var-file="database.tfvars"
terraform apply
```

## Rollback Procedures

### Using Terraform State

```bash
# View state history
terraform state list

# Restore previous state
terraform state pull > backup.tfstate

# Apply previous configuration
terraform apply -backup=backup.tfstate
```

### Manual Rollback

```bash
# Destroy specific resources
terraform destroy -target=resource.name

# Recreate with previous version
git checkout previous-commit
terraform apply
```

## Troubleshooting

### Common Issues

#### Issue: Provider Authentication Failed

**Solution:**
```bash
# Verify credentials
aws sts get-caller-identity  # AWS
az account show              # Azure
gcloud auth list             # GCP
```

#### Issue: Resource Already Exists

**Solution:**
```bash
# Import existing resource
terraform import resource.name resource-id
```

#### Issue: State Lock Error

**Solution:**
```bash
# Force unlock (use carefully)
terraform force-unlock LOCK_ID
```

### Debug Mode

Enable detailed logging:
```bash
# Terraform
export TF_LOG=DEBUG
terraform apply

# AWS CLI
aws --debug s3 ls

# Azure CLI
az --debug group list
```

## Best Practices

### 1. Use Remote State

Configure remote state backend:
```hcl
terraform {
  backend "s3" {
    bucket = "terraform-state-bucket"
    key    = "infrastructure/terraform.tfstate"
    region = "us-east-1"
  }
}
```

### 2. Implement State Locking

Prevent concurrent modifications:
```hcl
terraform {
  backend "s3" {
    # ...
    dynamodb_table = "terraform-locks"
  }
}
```

### 3. Version Control

Tag deployments:
```bash
git tag -a v1.0.0 -m "Production deployment"
git push origin v1.0.0
```

### 4. Validation Checks

Run validation before deployment:
```bash
terraform fmt -check
terraform validate
terraform plan
```

### 5. Documentation

Document each deployment:
```bash
# Create deployment log
echo "Deployed: $(date)" >> deployment.log
terraform output >> deployment.log
```

## CI/CD Integration

### GitHub Actions Example

```yaml
name: Deploy Infrastructure
on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Setup Terraform
        uses: hashicorp/setup-terraform@v2
      - name: Terraform Init
        run: terraform init
      - name: Terraform Apply
        run: terraform apply -auto-approve
```

## Security Considerations

- Never commit sensitive data
- Use environment variables for secrets
- Enable encryption for state files
- Implement least privilege access
- Regular security audits

## Maintenance

### Regular Tasks

- Update provider versions
- Review and optimize resources
- Monitor costs
- Check for security vulnerabilities
- Update documentation

### Cleanup

```bash
# Remove unused resources
terraform destroy -target=unused.resource

# Complete cleanup (careful!)
terraform destroy
```

## Additional Resources

- [Terraform Documentation](https://www.terraform.io/docs)
- [AWS CloudFormation](https://docs.aws.amazon.com/cloudformation/)
- [Azure Resource Manager](https://docs.microsoft.com/en-us/azure/azure-resource-manager/)
- [Google Cloud Deployment Manager](https://cloud.google.com/deployment-manager/docs)
