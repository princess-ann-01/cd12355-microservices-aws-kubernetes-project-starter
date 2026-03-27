# Coworking Analytics Microservice Deployment

## This project deploys a Python analytics application and a PostgreSQL database to Kubernetes on AWS.  

The application is containerized with Docker, built automatically with AWS CodeBuild, stored in Amazon ECR, and deployed to Kubernetes using manifest files in this repository.  
The deployment consists of two main services: a PostgreSQL database running inside the Kubernetes cluster and a Python Flask analytics application exposed through a Kubernetes `LoadBalancer` service.  
The application reads non-sensitive configuration from a Kubernetes ConfigMap and sensitive values from a Kubernetes Secret.  
This keeps the deployment flexible and avoids hardcoding credentials in the container image.  
The build pipeline uses AWS CodeBuild to build the Docker image from the source code in the `analytics/` directory.  
After a successful build, CodeBuild authenticates with Amazon ECR and pushes the image to the repository.  
Amazon ECR stores the versioned application images so Kubernetes releases are explicit and repeatable.  
To release a new version of the application, the source code is updated, committed, and pushed to GitHub.  
AWS CodeBuild is then used to build and push a new Docker image to Amazon ECR.  
If needed, the image tag is updated in the Kubernetes deployment manifest before redeployment.  
The Kubernetes manifests are then applied with `kubectl apply -f deployments/`.  
The application is packaged using the Dockerfile in `analytics/Dockerfile`.  
The image uses a Python base image, installs the required dependencies, and starts the Flask application on port `5153`.  
AWS CodeBuild automates the container build and push process using the `buildspec.yaml` file.  
The Kubernetes resources include the analytics deployment and service, ConfigMap, Secret, PostgreSQL deployment and service, and persistent volume resources stored in the `deployments/` directory.  
The analytics application is exposed externally on port `5153`, while PostgreSQL is exposed internally through `postgresql-service` on port `5432`.  
The deployment was validated by confirming that pods were running, services were created, and the analytics application responded correctly.  
The `/api/reports/daily_usage` and `/api/reports/user_visits` endpoints both returned valid JSON responses, and CloudWatch logs showed the application starting correctly without errors.  
The submission includes the application source code, `buildspec.yaml`, the `deployments/` directory with Kubernetes YAML files, `README.md`, and the `screenshots/` folder with project evidence.
