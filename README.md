
## Infracost as a FinOps Tool

This repository demonstrates the creation of an AWS EC2 instance using Terraform. It includes the use of Infracost to provide cost estimates for the resources created by Terraform scripts. The workflow is designed to trigger cost analysis comments on pull requests, facilitating better cost management in infrastructure development.

## Overview

This project integrates Terraform, Infracost, and GitHub Actions to deliver a comprehensive cost analysis of AWS EC2 instances directly within your GitHub workflow. By assessing potential cost impacts before changes are made, this setup helps manage cloud expenses more effectively, allowing you to make informed decisions about resource use.

## Prerequisites

Before you begin, ensure you have the following:
- AWS account with the necessary permissions to create EC2 instances.
- GitHub Account.
- Access to Infracost with a generated API key.

## Secrets

Ensure the following secrets are configured in your repository or CI/CD environment:

AWS_ACCESS_KEY_ID: Your AWS access key.

AWS_SECRET_ACCESS_KEY: Your AWS secret key.

INFRACOST_API_KEY: The API key for Infracost.

## Workflow Steps

Step 1: Clone the repository and switch to the directory:

$ git clone https://github.com/Arunkumar2255/Cost-EC2.git

$ cd Cost-EC2/

Step 2: Configure GitHub Actions

Create or modify the infracost.yml file in .github/workflows to set up the GitHub Actions workflow. The workflow triggers on pull requests and uses Infracost to analyze cost differences:

on: [pull_request]
jobs:
  infracost:
    name: Infracost
    runs-on: ubuntu-latest
    permissions:
      contents: read
      pull-requests: write

    env:
      TF_ROOT: ec2/  # Use TF_ROOT for clarity
    steps:
      - name: Setup Infracost
        uses: infracost/actions/setup@v2
        with:
          api-key: ${{ secrets.INFRACOST_API_KEY }}

      - name: Checkout base branch
        uses: actions/checkout@v3
        with:
          ref: '${{ github.event.pull_request.base.ref }}'

      # Generate Infracost JSON file as the baseline.
      - name: Generate Infracost cost estimate baseline
        run: |
          infracost breakdown --path=${TF_ROOT} \
                              --format=json \
                              --out-file=/tmp/infracost-base.json
      # Checkout the current PR branch so we can create a diff.
      - name: Checkout PR branch
        uses: actions/checkout@v3

      # Generate an Infracost diff and save it to a JSON file.
      - name: Generate Infracost diff
        run: |
          infracost diff --path=${TF_ROOT} \
                          --format=json \
                          --compare-to=/tmp/infracost-base.json \
                          --out-file=/tmp/infracost.json
                          
      - name: Post Infracost comment
        run: |
            infracost comment github --path=/tmp/infracost.json \
                                     --repo=$GITHUB_REPOSITORY \
                                     --github-token=${{github.token}} \
                                     --pull-request=${{github.event.pull_request.number}} \
                                     --behavior=update


Step 3: Create a Pull Request

Change the instance type in the main.tf file from t3.medium to a new type, like t3.large. Then, commit and push your changes to a new branch:
Next, create a pull request on GitHub against the main branch to initiate the cost analysis workflow.

Step 4: Actions Pipeline

Monitor the 'Actions' tab in your GitHub repository to view the workflow execution and the automated cost analysis comments on your pull requests.

Step 5: Review and Merge

Review the cost analysis provided by Infracost in the pull request comments before merging. This helps ensure that all changes are cost-effective and necessary.


## Troubleshooting

Ensure AWS credentials and InfraCost API keys are correctly configured in GitHub secrets.

Verify that the GitHub Actions workflow is correctly set up and triggered.
