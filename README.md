# CloudFormation Deploy Stack Action

![Built with Claude](https://img.shields.io/badge/Built%20with-Claude-ff9800??style=flat)&nbsp;![Release](https://github.com/subhamay-bhattacharyya-gha/cfn-deploy-action/actions/workflows/release.yaml/badge.svg)&nbsp;![Commit Activity](https://img.shields.io/github/commit-activity/t/subhamay-bhattacharyya-gha/cfn-deploy-action)&nbsp;![Last Commit](https://img.shields.io/github/last-commit/subhamay-bhattacharyya-gha/cfn-deploy-action)&nbsp;![Release Date](https://img.shields.io/github/release-date/subhamay-bhattacharyya-gha/cfn-deploy-action)&nbsp;![Repo Size](https://img.shields.io/github/repo-size/subhamay-bhattacharyya-gha/cfn-deploy-action)&nbsp;![File Count](https://img.shields.io/github/directory-file-count/subhamay-bhattacharyya-gha/cfn-deploy-action)&nbsp;![Issues](https://img.shields.io/github/issues/subhamay-bhattacharyya-gha/cfn-deploy-action)&nbsp;![CloudFormation](https://img.shields.io/badge/IaC-CloudFormation-orange?style=flat)&nbsp;![Top Language](https://img.shields.io/github/languages/top/subhamay-bhattacharyya-gha/cfn-deploy-action)&nbsp;![Custom Endpoint](https://img.shields.io/endpoint?url=https://gist.githubusercontent.com/bsubhamay/6e30aa9191fd99110ba56249fbec700f/raw/cfn-deploy-action.json?)

A GitHub Action for deploying and managing AWS CloudFormation stacks with OIDC authentication.

## CloudFormation Deploy Stack

### Description

This GitHub Action provides a reusable composite workflow for deploying AWS CloudFormation stacks. It supports both stack creation and updates, validates parameters, waits for deployment completion, and provides detailed deployment feedback. The action uses OIDC for secure AWS credential management.

---

## Inputs

| Name                | Description                                      | Required | Default                |
|---------------------|--------------------------------------------------|----------|------------------------|
| `stack-name`        | Name of the CloudFormation stack                 | Yes      | —                      |
| `region`            | AWS region                                       | Yes      | —                      |
| `template-url`      | URL of the CloudFormation template               | Yes      | —                      |
| `parameters-file`   | Path to parameters JSON file                     | No       | `infra/parameters.json`|
| `aws-account-id`    | AWS account ID for OIDC role assumption          | Yes      | —                      |
| `oidc-role-name`    | Name of the OIDC role to assume                  | Yes      | —                      |

---

## Outputs

| Name              | Description                                    |
|-------------------|------------------------------------------------|
| `operation`       | Type of operation performed (CREATE or UPDATE) |
| `stack-status`    | Current status of the CloudFormation stack     |
| `stack-outputs`   | JSON outputs from the CloudFormation stack     |

---

## Example Usage

### Basic Stack Deployment

```yaml
name: Deploy CloudFormation Stack

on:
  push:
    branches:
      - main

jobs:
  upload-to-s3:
    name: Upload Template to S3
    runs-on: ubuntu-latest
    outputs:
      template_url: ${{ steps.upload.outputs.template_url }}
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Upload template to S3
        id: upload
        shell: bash
        run: |
          # Your S3 upload logic here
          echo "template_url=https://s3.amazonaws.com/bucket/template.yaml" >> $GITHUB_OUTPUT

  deploy:
    name: Deploy Stack
    runs-on: ubuntu-latest
    environment: devl
    needs: upload-to-s3
    permissions:
      id-token: write
      contents: read
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Deploy CloudFormation Stack
        uses: subhamay-bhattacharyya-gha/cfn-deploy-action@v1
        with:
          stack-name: my-production-stack
          region: us-east-1
          template-url: ${{ needs.upload-to-s3.outputs.template_url }}
          parameters-file: infra/parameters.json
          aws-account-id: ${{ secrets.AWS_ACCOUNT_ID }}
          oidc-role-name: github-actions-role
```

### Accessing Action Outputs

```yaml
- name: Deploy CloudFormation Stack
  id: deploy
  uses: subhamay-bhattacharyya-gha/cfn-deploy-action@v1
  with:
    stack-name: my-stack
    region: us-east-1
    template-url: ${{ needs.upload-to-s3.outputs.template_url }}
    aws-account-id: ${{ secrets.AWS_ACCOUNT_ID }}
    oidc-role-name: github-actions-role

- name: Use deployment outputs
  shell: bash
  run: |
    echo "Operation: ${{ steps.deploy.outputs.operation }}"
    echo "Stack Status: ${{ steps.deploy.outputs.stack-status }}"
    echo "Stack Outputs: ${{ steps.deploy.outputs.stack-outputs }}"
```

---

## Features

- **OIDC Authentication**: Secure AWS credential management via GitHub OIDC provider
- **Automatic Stack Detection**: Creates new stacks or updates existing ones
- **Parameter Management**: Optional JSON parameters file support
- **Deployment Waiting**: Waits for stack operations to complete
- **Detailed Feedback**: Posts comprehensive deployment information to GitHub Actions step summary
- **Error Handling**: Graceful handling of stack update scenarios (no changes needed)
- **Output Extraction**: Captures and exposes CloudFormation stack outputs

---

## Prerequisites

1. AWS Account with CloudFormation permissions
2. GitHub repository with OIDC provider configured
3. IAM role with CloudFormation, S3, and other required permissions
4. CloudFormation template uploaded to S3 or accessible via URL

---

## IAM Policy Requirements

The OIDC role must have permissions for:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "cloudformation:CreateStack",
        "cloudformation:UpdateStack",
        "cloudformation:DescribeStacks",
        "cloudformation:GetTemplate"
      ],
      "Resource": "arn:aws:cloudformation:*:*:stack/*"
    },
    {
      "Effect": "Allow",
      "Action": [
        "iam:CreateRole",
        "iam:PutRolePolicy",
        "iam:PassRole"
      ],
      "Resource": "*"
    }
  ]
}
```

## License

MIT
