---
date: 2025-06-10T10:58:08-04:00
description: ""
featured_image: "/images/auto-build-push-docker-to-aws-ecr-github-actions.png"
tags: ["aws", "github-actions", "ecr", "docker"]
title: "I. Auto build & push Docker Image to AWS ECR with Github Actions"
---

## Introduction
Automating the build and deployment of Docker images using GitHub Actions and AWS Elastic Container Registry (ECR) significantly streamlines your software delivery process. 

By integrating these powerful tools, developers benefit from reduced manual intervention, minimized errors, and accelerated deployment cycles. 

GitHub Actions automates your build workflow whenever code changes, while AWS ECR securely stores and manages your container images, ensuring reliability and scalability. 

In this guide, you'll learn step-by-step how to effortlessly set up continuous integration and continuous deployment (CI/CD) to AWS ECR, maximizing your development efficiency.

## 1. Create ECR Repository
- Create simple ECR repository via AWS Console
    - Repository Name: banking-system
    - Image tag mutability: Immutable
    - Encryption configuration: AES-256
    - Scan on push: Enable

## 2. Setting up GitHub Actions OIDC authentication with AWS
- Add GitHub as an OIDC Identity Provider in AWS
    - Open the AWS IAM Console: AWS IAM Console => Identity providers => Add provider
    - Provider Type: OIDC
    - Provider URL: https://token.actions.githubusercontent.com
    - Audience: sts.amazonaws.com

- Create an IAM Role for GitHub Actions
    - AWS IAM Console => Create role
    - Trusted entity type: Web identity
    - Identity provider: token.actions.githubusercontent.com
    - Audience: sts.amazonaws.com
    - GitHub organization: {{Your Github username or organization}}
    - Add permissions:
        - AmazonEC2ContainerRegistryFullAccess
        - SecretsManagerReadWrite
    - Role name: github-actions-deploy-role

- After create the role successfull, please go inside the role and note the ARN of role.

- In your Github repository => Settings => Secrets and variables => Actions:
    - New repository secret
    - Name: STAGING_GITHUB_ACTION_DEPLOY_ROLE
    - Secret: ARN of AWS Role "github-actions-deploy-role" just created

## 3. GitHub Actions pipleline
.github/workflows/deploy-staging.yaml
```yaml
name: Deploy to staging

on:
  push:
    branches: [staging]

permissions:
  id-token: write     # <--- REQUIRED for OIDC to work
  contents: read

jobs:
  deploy:
    name: Build image
    runs-on: ubuntu-latest

    steps:
      - name: Checkout repo
        uses: actions/checkout@v4

      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: ${{ secrets.STAGING_GITHUB_ACTION_DEPLOY_ROLE }}
          aws-region: ap-southeast-1

      - name: Login to Amazon ECR
        id: login-ecr
        uses: aws-actions/amazon-ecr-login@v2

      - name: Build, tag, and push image to Amazon ECR
        env:
          ECR_REGISTRY: ${{ steps.login-ecr.outputs.registry }}
          ECR_REPOSITORY: banking-system
          IMAGE_TAG: ${{ github.sha }}
        run: |
          docker build \
            -f Dockerfile.deploy \
            --build-arg ENVIRONMENT=staging \
            -t $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG .
          docker push $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG
```

## Enjoy the results. ^^.