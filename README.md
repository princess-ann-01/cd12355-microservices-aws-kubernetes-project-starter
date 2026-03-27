# Coworking Analytics Microservice Deployment

This project deploys a Python analytics application and a PostgreSQL database to Kubernetes on AWS.  
The application is containerized with Docker, built automatically with AWS CodeBuild, stored in Amazon ECR, and deployed to Kubernetes using manifest files in this repository.  
The project ensures that deployment is secure, scalable, and follows best practices for containerized workloads.  

## Deployment Architecture

The deployment consists of two main services: PostgreSQL Database and Python Flask Analytics Application.  
PostgreSQL runs inside the Kubernetes cluster and is exposed internally through `postgresql-service` on port `5432`.  
The analytics application is exposed externally through a Kubernetes `LoadBalancer` service on port `5153`.  
Non-sensitive values are stored in a Kubernetes ConfigMap, while sensitive values like credentials are stored securely in a Kubernetes Secret.  

## Build and Release Process

AWS CodeBuild builds the Docker image from the `analytics/Dockerfile` and automates the container build and push process using `buildspec.yaml`.  
CodeBuild authenticates with Amazon ECR to push the image and stores versioned application images for explicit releases.  
To deploy, the image tag in Kubernetes manifests can be updated before applying resources with `kubectl`.  
Kubernetes resources include deployments for analytics and PostgreSQL, services, ConfigMap, Secret, and persistent volumes stored in the `deployments/` directory.  

## Application Endpoints

The analytics application exposes the following endpoints:
- `/api/reports/daily_usage`
- `/api/reports/user_visits`  

Both endpoints return valid JSON responses and read data from PostgreSQL.  

## Validation

Validation confirmed pods and services were running successfully, and CloudWatch logs verified the application startup.  
The deployment operates securely by preventing hardcoded values and validating the application behavior in multiple areas.  

## Cost-Saving Suggestions

Costs can be reduced by using smaller worker nodes for development environments and removing unused AWS resources after testing.  
Cleaning up old container images in ECR also reduces costs, similar to scaling clusters only as needed.  

## Contents

The README has been structured along with application code, build pipeline instructions, Kubernetes manifests, and screenshots of project validation.  
Submission includes required files for deployment, validation logs, and evidence stored in a specific repository folder.  