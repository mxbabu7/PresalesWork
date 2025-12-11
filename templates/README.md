# Infrastructure Templates

This directory contains Infrastructure as Code (IaC) templates for various cloud providers and use cases.

## Directory Structure

```
templates/
├── aws/          # Amazon Web Services templates
├── azure/        # Microsoft Azure templates
├── gcp/          # Google Cloud Platform templates
└── README.md     # This file
```

## Available Templates

### AWS Templates

Coming soon:
- VPC with public/private subnets
- EC2 instances with auto-scaling
- EKS (Kubernetes) cluster
- RDS database instances
- S3 buckets with versioning
- CloudFront distributions
- Lambda functions

### Azure Templates

Coming soon:
- Virtual Networks
- Virtual Machines
- AKS (Kubernetes) cluster
- Azure SQL Database
- Storage Accounts
- Application Gateway
- Azure Functions

### GCP Templates

Coming soon:
- VPC Networks
- Compute Engine instances
- GKE (Kubernetes) cluster
- Cloud SQL
- Cloud Storage buckets
- Cloud Load Balancing
- Cloud Functions

## Usage

Each template directory contains:
- `main.tf` or equivalent - Main infrastructure definition
- `variables.tf` - Input variables
- `outputs.tf` - Output values
- `example.tfvars` - Example variable values
- `README.md` - Template-specific documentation

### Basic Usage

1. Navigate to the template directory:
   ```bash
   cd templates/aws/vpc
   ```

2. Copy example variables:
   ```bash
   cp example.tfvars terraform.tfvars
   ```

3. Edit variables:
   ```bash
   vim terraform.tfvars
   ```

4. Deploy:
   ```bash
   terraform init
   terraform plan
   terraform apply
   ```

## Contributing

To contribute a new template:

1. Create a new directory under the appropriate cloud provider
2. Include all required files (main, variables, outputs, README)
3. Add example configurations
4. Test thoroughly in an isolated environment
5. Document prerequisites and usage
6. Submit a pull request

## Best Practices

- Use descriptive resource names
- Parameterize all values
- Include tags/labels
- Document all variables
- Provide sensible defaults
- Include usage examples
- Test in multiple regions
- Follow cloud provider best practices

## Support

For questions or issues with templates:
- Check the template's README
- Review the main documentation in `docs/`
- Open an issue with detailed information
- Include error messages and environment details

## License

All templates are provided under the MIT License. See the root LICENSE file for details.
