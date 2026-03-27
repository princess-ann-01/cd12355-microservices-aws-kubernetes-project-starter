# Coworking Analytics Microservice Deployment

This project deploys a Python analytics application and a PostgreSQL database to Kubernetes on AWS. The application is containerized with Docker, built automatically with AWS CodeBuild, stored in Amazon ECR, and deployed to Kubernetes using manifest files in this repository.

## Project Overview

The deployment consists of two main services:

- A PostgreSQL database running inside the Kubernetes cluster
- A Python Flask analytics application exposed through a Kubernetes `LoadBalancer` service

The application reads non-sensitive configuration from a Kubernetes ConfigMap and sensitive values from a Kubernetes Secret. This keeps the deployment flexible and avoids hardcoding credentials in the container image.

## Build and Release Process

The build pipeline uses AWS CodeBuild to build the Docker image from the source code in the `analytics/` directory. After a successful build, CodeBuild authenticates with Amazon ECR and pushes the image to the repository.

Amazon ECR stores the versioned application images. The Kubernetes deployment references the image so releases are explicit and repeatable.

To release a new version of the application:

1. Update the application source code
2. Commit and push the changes to GitHub
3. Run AWS CodeBuild to build and push a new Docker image to Amazon ECR
4. Update the image tag in `deployment.yaml` if deploying a new version
5. Apply the Kubernetes manifests with `kubectl apply -f`
6. Verify the rollout with `kubectl get pods`, `kubectl get svc`, and endpoint checks

## Docker

The application is packaged using the Dockerfile in `analytics/Dockerfile`. The image uses a Python base image, installs the required dependencies, and starts the Flask application on port `5153`.

The image is versioned using semantic versioning and also tagged as `latest`.

## AWS CodeBuild

AWS CodeBuild automates the container build and push process. The `buildspec.yaml` file logs in to Amazon ECR in `us-east-1`, builds the image from `./analytics`, tags the image, and pushes it to the repository.

This approach separates build concerns from deployment concerns and makes it easier to produce consistent container images for Kubernetes.

## Kubernetes Deployment

The Kubernetes resources are defined as YAML manifests in the repository and include:

- `deployment.yaml` for the analytics application
- `service.yaml` for the analytics application
- `configmap.yaml` for non-sensitive environment variables
- `secret.yaml` for sensitive database credentials
- `postgresql-deployment.yaml` for PostgreSQL
- `postgresql-service.yaml` for PostgreSQL
- `pv.yaml` and `pvc.yaml` for persistent storage

The analytics application is exposed externally using a `LoadBalancer` service on port `5153`. PostgreSQL is exposed internally within the cluster using the `postgresql-service` service on port `5432`.

## Configuration Management

Application configuration is separated into two types:

- Non-sensitive configuration in `configmap.yaml`
  - `DB_HOST`
  - `DB_PORT`
  - `DB_NAME`
  - `DB_USERNAME`

- Sensitive configuration in `secret.yaml`
  - `DB_PASSWORD`

This keeps deployment configuration manageable and protects sensitive values.

## Validation

The deployment was validated by confirming:

- Kubernetes pods are running successfully
- Services are created successfully
- The analytics application responds correctly on port `5153`
- The `/api/reports/daily_usage` endpoint returns valid JSON
- The `/api/reports/user_visits` endpoint returns valid JSON
- CloudWatch logs show the application starting correctly and generating reports without errors

## Files Submitted

The submission includes:

- `analytics/`
- `buildspec.yaml`
- `configmap.yaml`
- `secret.yaml`
- `deployment.yaml`
- `service.yaml`
- `postgresql-deployment.yaml`
- `postgresql-service.yaml`
- `pv.yaml`
- `pvc.yaml`
- `README.md`
- `screenshots/`

## Standout Suggestions

### Resource allocation
Reasonable CPU and memory requests and limits should be defined in the Kubernetes deployment so pods can be scheduled efficiently and the application remains stable under load. This analytics service is lightweight, so modest resource values are appropriate.

### Best AWS instance type
A `t3.medium` instance type is a practical choice for this project because it provides a good balance of CPU and memory for a small Kubernetes learning workload. It is sufficient for running the analytics service and PostgreSQL without unnecessary cost.

### Cost savings
Costs can be reduced by deleting unused load balancers, persistent volumes, and container images after testing. Additional savings come from using small worker nodes, avoiding unnecessary rebuilds, and shutting down non-production resources when they are not needed.

## Screenshots

Project evidence screenshots are stored in the `screenshots/` folder, including:

- AWS CodeBuild successful build
- Amazon ECR repository with image tags
- `kubectl get pods`
- `kubectl get svc`
- `kubectl describe svc postgresql-service`
- `kubectl describe deployment analytics-deployment`
- AWS CloudWatch logs
