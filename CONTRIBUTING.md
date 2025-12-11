# Contributing to PresalesWork Infrastructure

Thank you for your interest in contributing to the PresalesWork Infrastructure repository! This document provides guidelines and instructions for contributing.

## 🎯 Code of Conduct

By participating in this project, you agree to abide by our [Code of Conduct](CODE_OF_CONDUCT.md). Please read it before contributing.

## 🚀 Getting Started

1. **Fork the Repository**
   - Click the "Fork" button at the top right of the repository page
   - Clone your fork locally:
     ```bash
     git clone https://github.com/YOUR_USERNAME/PresalesWork.git
     cd PresalesWork
     ```

2. **Create a Branch**
   - Create a new branch for your feature or bug fix:
     ```bash
     git checkout -b feature/your-feature-name
     ```
   - Use descriptive branch names:
     - `feature/add-terraform-template`
     - `fix/correct-deployment-script`
     - `docs/update-architecture-guide`

3. **Make Your Changes**
   - Write clear, concise commit messages
   - Follow existing code style and conventions
   - Add or update documentation as needed
   - Test your changes thoroughly

## 📝 Contribution Types

### Infrastructure Templates
- Add new IaC templates (Terraform, CloudFormation, etc.)
- Improve existing templates
- Include examples and documentation

### Documentation
- Fix typos or clarify existing docs
- Add new guides or tutorials
- Update outdated information

### Scripts and Automation
- Add deployment or utility scripts
- Improve existing automation
- Ensure scripts are well-documented

### Bug Fixes
- Fix issues reported in GitHub Issues
- Include tests to prevent regression
- Reference the issue number in your commit

## 🔍 Pull Request Process

1. **Before Submitting**
   - Ensure your code follows project conventions
   - Update documentation if needed
   - Add tests for new features
   - Verify all existing tests pass
   - Update CHANGELOG.md if applicable

2. **Submitting the PR**
   - Push your changes to your fork
   - Open a Pull Request against the `main` branch
   - Fill out the PR template completely
   - Link related issues using keywords (e.g., "Fixes #123")

3. **PR Title Format**
   - Use clear, descriptive titles
   - Examples:
     - `feat: Add AWS EKS Terraform template`
     - `fix: Correct variable reference in deployment script`
     - `docs: Update getting started guide`

4. **After Submission**
   - Respond to review comments promptly
   - Make requested changes in new commits
   - Keep your PR up to date with the main branch

## ✅ Coding Standards

### General Guidelines
- Write clear, self-documenting code
- Add comments for complex logic
- Keep functions small and focused
- Follow DRY (Don't Repeat Yourself) principles

### Infrastructure as Code
- Use descriptive resource names
- Add tags/labels to all resources
- Include variable descriptions
- Provide default values where appropriate
- Document requirements and dependencies

### Documentation
- Use Markdown for all documentation
- Include code examples where relevant
- Keep language clear and concise
- Add diagrams for complex architectures

### Scripts
- Include usage instructions
- Add error handling
- Validate inputs
- Use shellcheck for bash scripts

## 🧪 Testing

- Test infrastructure templates in isolated environments
- Validate scripts before submitting
- Include test scenarios in documentation
- Report test results in PR description

## 📋 Commit Message Guidelines

Follow the [Conventional Commits](https://www.conventionalcommits.org/) specification:

```
<type>(<scope>): <description>

[optional body]

[optional footer]
```

**Types:**
- `feat`: New feature
- `fix`: Bug fix
- `docs`: Documentation changes
- `style`: Code style changes (formatting, etc.)
- `refactor`: Code refactoring
- `test`: Adding or updating tests
- `chore`: Maintenance tasks

**Examples:**
```
feat(terraform): Add Azure AKS cluster template

Added a new Terraform template for deploying Azure Kubernetes Service
clusters with configurable node pools and network settings.

Closes #45
```

## 🐛 Reporting Bugs

When reporting bugs, please include:
- Clear description of the issue
- Steps to reproduce
- Expected behavior
- Actual behavior
- Environment details (OS, versions, etc.)
- Screenshots or logs if applicable

## 💡 Suggesting Enhancements

Enhancement suggestions are welcome! Please:
- Check if the enhancement is already requested
- Clearly describe the proposed feature
- Explain the use case and benefits
- Provide examples if possible

## 📞 Questions?

If you have questions:
- Check existing documentation
- Search closed issues
- Open a new issue with the "question" label

## 🙏 Recognition

Contributors will be recognized in:
- Repository contributors page
- Release notes
- Project documentation

Thank you for contributing to PresalesWork Infrastructure! 🎉
