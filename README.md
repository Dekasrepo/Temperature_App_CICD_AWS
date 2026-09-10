# Temperature_App_CICD_AWS

# Secure CI/CD Pipeline with GitHub Actions, AWS OIDC & S3

A production-oriented CI/CD pipeline for deploying a Python application to Amazon S3 using GitHub Actions, with integrated code quality checks, automated testing, SAST, dependency vulnerability scanning, and passwordless AWS authentication through GitHub Actions OIDC.

---

## 🏗️ Architecture

```text
                    Developer
                        │
                        │ git push / pull request
                        ▼
                 ┌──────────────┐
                 │   GitHub      │
                 │ Repository    │
                 └──────┬───────┘
                        │
                        ▼
                ┌─────────────────┐
                │ GitHub Actions  │
                └────────┬────────┘
                         │
                 ┌───────▼────────┐
                 │       CI       │
                 │                │
                 │ • Flake8       │
                 │ • Pytest       │
                 │ • Bandit       │
                 │ • pip-audit    │
                 └───────┬────────┘
                         │
                    CI succeeds
                         │
                         ▼
                 ┌─────────────────┐
                 │   Deploy Job    │
                 │                 │
                 │ GitHub OIDC     │
                 └────────┬────────┘
                          │
                          ▼
                  GitHub OIDC Provider
                          │
                          ▼
                     AWS STS
                          │
                   Assume IAM Role
                          │
                          ▼
              ┌─────────────────────┐
              │ Dedicated IAM Role  │
              │                     │
              │ Least-privilege     │
              │ S3 permissions      │
              └──────────┬──────────┘
                         │
                         ▼
                 ┌───────────────┐
                 │   Amazon S3   │
                 │               │
                 │ Static Website │
                 └───────┬───────┘
                         │
                         ▼
                       Users

````

------
## Project Overview
This project demonstrates a secure CI/CD workflow that automatically validates, scans, and deploys a Python application to Amazon S3.

The pipeline is designed around two stages:

CI: Code quality, testing, and security validation
CD: Secure deployment to AWS after CI succeeds

Pull requests run the CI pipeline, while only successful pushes to the main branch trigger deployment.

----
## CI Pipeline
The CI job performs four checks:

### 1. Code Linting
Flake8 checks Python code quality and style.
```Bash
flake8 .
```
### 2. Unit Testing
Pytest runs the application's automated tests.
```Bash
bandit -r . -lll
````

### 3. Static Application Security Testing
Bandit scans the Python source code for common security issues.
```Bash
bandit -r . -lll
```
The pipeline is configured to fail on high-severity findings.

### 4. Dependency Vulnerability Scanning
pip-audit checks Python dependencies against known vulnerabilities.
```Bash
pip-audit -r requirements.txt
```
A vulnerable Flask dependency was identified during development and upgraded to a fixed version.

------
## AWS Deployment
Deployment uses GitHub Actions OIDC instead of long-lived AWS access keys.

The authentication flow is:
```
GitHub Actions
      │
      ▼
GitHub OIDC
      │
      ▼
AWS STS
      │
      ▼
Temporary AWS Credentials
      │
      ▼
IAM Role
      │
      ▼
Amazon S3
```
The deployment job requests an OIDC token and uses it to assume a dedicated IAM role through AWS STS.

No AWS access keys are stored in GitHub Secrets.

----
## IAM Security
The deployment IAM role follows the principle of least privilege.

Trust Policy

The role trust policy restricts access to:

The specific GitHub repository
The main branch
GitHub's OIDC provider
AWS STS as the intended audience

The repository uses GitHub's immutable OIDC subject format:
```
repo:OWNER@OWNER-ID/REPOSITORY@REPOSITORY-ID:ref:refs/heads/main
```

------
## Permissions Policy
The role is limited to the S3 bucket used for deployment.

Required permissions:
```
s3:ListBucket
s3:GetObject
s3:PutObject
s3:DeleteObject
```
No ```AdministratorAccess``` or broad S3 permissions are used.

-----
## Deployment Strategy
The workflow uses GitHub Actions job dependencies to create a deployment gate:
```
Pull Request
     │
     ▼
    CI
     │
     ├── Lint
     ├── Tests
     ├── SAST
     └── Dependency Scan
     
Push to main
     │
     ▼
    CI
     │
     ▼
CI Successful
     │
     ▼
  Deploy
     │
     ▼
    S3
```
The deployment job uses:
```
needs: CI
```
and only executes for pushes to main:
```
if: github.event_name == 'push' && github.ref == 'refs/heads/main'
```
This ensures pull requests can validate the application without deploying it.

----
## Key Security Decisions
| Decision                            | Reason                               |
| ----------------------------------- | ------------------------------------ |
| GitHub OIDC                         | Avoid long-lived AWS credentials     |
| Temporary AWS credentials           | Reduce credential exposure           |
| Dedicated IAM role                  | Isolate deployment permissions       |
| Least-privilege S3 permissions      | Minimize AWS blast radius            |
| Repository/branch trust restriction | Prevent unauthorized role assumption |
| CI security gates                   | Detect issues before deployment      |
| PR CI without deployment            | Validate changes before merging      |


----
## Troubleshooting & Lessons Learned
### GitHub OIDC Authentication
The initial ```AssumeRoleWithWebIdentity``` failure was caused by the IAM trust policy using the older GitHub OIDC subject format.

The repository required GitHub's newer immutable subject format:
```
repo:OWNER@OWNER-ID/REPOSITORY@REPOSITORY-ID:ref:refs/heads/main
```
Updating the trust policy resolved the authentication failure.

### Security Scanner Findings
Bandit identified security findings in the application and tests. These were reviewed in context rather than blindly suppressed.

### Dependency Vulnerability
``` pip-audit ``` identified a vulnerability in the Flask dependency. The dependency was upgraded before the pipeline was considered complete.

### GitHub Actions YAML
Several workflow configuration errors were encountered during development, including incorrect indentation, action names, runner names, dependency syntax, and environment-variable expressions.

These were resolved through debugging and validation of the workflow structure.

----
## Technologies
- Python
- Flask
- GitHub Actions
- GitHub OIDC
- Flake8
- Pytest
- Bandit
- pip-audit
- AWS IAM
- AWS STS
- Amazon S3

----

## Project Structure
```
Temperature_App_CICD_AWS/
│
├── .github/
│   └── workflows/
│       └── cicd.yaml
│
├── app.py
├── test_app.py
├── requirements.txt
└── README.md
```

----
## Future Improvements
This project is the foundation for a more advanced containerized deployment.

Planned extensions include:

- Docker containerization
- Amazon ECR
- Amazon EKS
- Kubernetes
- Terraform
- Container image security scanning
- Helm
- Prometheus & Grafana
- Production-grade AWS networking
- Private S3 + CloudFront

----

## Author
### Jane Obikwelu

### Cloud / DevOps Engineer

### AWS | Cloud Infrastructure | DevSecOps | Kubernetes | Terraform



