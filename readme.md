# Django Application CI/CD Pipeline with GitHub Actions

This project demonstrates how to set up a CI/CD pipeline for a Django application using GitHub Actions, with automatic deployment to an EC2 instance.

## Overview

This pipeline is configured to automatically:

1. **Build and test the Django application**: Runs tests to ensure that the application works as expected.
2. **Deploy the application**: Once the tests pass, the pipeline automatically deploys the Django application to an EC2 instance.

## Features

- **Continuous Integration (CI)**: Runs tests on every push to the repository.
- **Continuous Deployment (CD)**: Deploys the application to an EC2 instance after successful tests.
- **GitHub Actions**: Automates the entire workflow.

## Prerequisites

Before setting up the pipeline, ensure that you have the following:

1. **Django Application**: The Django project should be ready and pushed to a GitHub repository.
2. **EC2 Instance**: Set up an EC2 instance running Ubuntu (or another compatible OS).
3. **GitHub Repository Secrets**: Add the following secrets in your GitHub repository under **Settings > Secrets**:
   - `AWS_ACCESS_KEY_ID`
   - `AWS_SECRET_ACCESS_KEY`
   - `EC2_HOST`: The public IP address or DNS of your EC2 instance.
   - `EC2_USER`: The user to connect to your EC2 instance (typically `ubuntu` for Ubuntu-based instances).
   - `SSH_PRIVATE_KEY`: The private SSH key for accessing the EC2 instance.

## GitHub Actions Workflow

The main configuration for the CI/CD pipeline is inside the `.github/workflows/ci-cd.yml` file. Below is an overview of the steps:

```yaml
name: Django CI

on:
  push:
    branches: [ "main" ]

jobs:
  build:
    runs-on: self-hosted
   
    steps:
      - name: Checkout code
        uses: actions/checkout@v2

        
      - name: Deploy application
        run: |
          app_dir="/var/www/project/PROJECT-K"
          cd "$app_dir"
          sudo su
          # Pull the latest changes from the Git repository
          sudo git checkout main
          sudo git pull -r
          # Check if the pull was successful
          if [ $? -eq 0 ]; then
            source  /var/www/project/venv/bin/activate
            python3 -m pip install -r requirements.txt
            python3 manage.py migrate
            sudo systemctl restart gunicorn
            sudo systemctl restart nginx
            echo "Git pull successful, and service restarted."
          else
            echo "Git pull failed. Please check your Git repository."
          fi
