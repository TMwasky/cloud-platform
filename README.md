# Cloud Platform - Infrastructure as Code (IaC)

This repository houses the automated deployment pipelines and AWS CloudFormation templates for the Cloud Platform project. It demonstrates a modular, enterprise-grade approach to Infrastructure as Code (IaC) by strictly decoupling identity management from application infrastructure to minimize blast radius and ensure secure deployments.

## Architecture & Structure

The repository is divided into two independent deployment domains:

1. **Identity Stack:** Manages AWS IAM resources. It deploys managed policies and exports their ARNs for dynamic consumption by IAM Roles using cross-stack references.
2. **Infrastructure Stack:** Provisions the core application infrastructure, including an Amazon S3 bucket and an AWS Lambda function with its execution role.

```text
cloud-platform/
├── .github/workflows/       # CI/CD Pipelines (Deployments & PR Validation)
├── identity/                # IAM Roles and Managed Policies
│   ├── policies/s3-readonly.yaml
│   └── roles/developer-role.yaml
└── infrastructure/          # Core App Resources
    └── app.yaml             # S3 Bucket and Lambda Function

```

## Prerequisites

To deploy this infrastructure, you need an AWS Account and a designated CI/CD IAM User with programmatic access (permissions to manage CloudFormation, IAM, S3, and Lambda).

Configure the following repository secrets in GitHub `(Settings > Secrets and variables > Actions)`:

```
AWS_ACCESS_KEY_ID
AWS_SECRET_ACCESS_KEY
AWS_REGION (e.g., us-east-1)
```

## Deployment

Deployments are orchestrated entirely via GitHub Actions.

**Manual Deployment (Initial Setup):** Navigate to the Actions tab, select a deployment workflow (`Deploy Identity` or `Deploy Infrastructure`), and execute it via the Run workflow dropdown.

_Note: Always deploy the Identity stack first, as the developer role relies on the exported policy ARN._

**Automated Deployment:** Merging a Pull Request into the `main` branch automatically triggers the respective deployment pipeline based on the modified file paths.

## Development Workflow

Direct pushes to the main branch are restricted. To contribute to this infrastructure, follow the standard operating procedure:

**Feature Branch:** Create a branch off `main` (e.g., `feature/update-lambda-runtime`).

**Pull Request:** Open a PR against `main`.

**Automated CI Checks:** Opening the PR automatically triggers the `PR Validation Checks` workflow, which runs `aws cloudformation validate-template` against your changes to catch syntax/schema errors.

**Review & Merge:** Once the CI checks pass (green checkmark) and peer approval is granted, merging the PR will automatically deploy the changes to AWS.

## Known Issues & Troubleshooting

**CloudFormation Early Validation Failure on IAM Policies:**
If the deployment of an `AWS::IAM::ManagedPolicy` fails during the `CREATE_CHANGESET` phase with `[AWS::EarlyValidation::PropertyValidation] Unsupported property [Tags]`, it is due to strict organizational compliance hooks (e.g., AWS CloudFormation Guard).

**Resolution:** Remove the `Tags:` YAML block from the Managed Policy resource. CloudFormation will automatically propagate tags via the `--tags` flag used in the GitHub Actions deployment command.

**Author**
- **Tonny Mwambingu** <[Mwasky](https://github.com/TMwasky)>
