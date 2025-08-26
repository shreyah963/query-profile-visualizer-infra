# Query Profiler Infrastructure

Infrastructure as Code for the Query Profiler application deployment on AWS EKS.

## Architecture

This repository contains all infrastructure and deployment configurations for the Query Profiler application:

- **EKS Cluster**: Kubernetes cluster configuration
- **IAM Roles**: Service roles for CodeBuild and EKS access
- **Kubernetes Manifests**: Deployment, service, and namespace configurations
- **CI/CD Pipeline**: Automated deployment via CodeBuild

## Repository Structure

```
query-profiler-infrastructure/
├── aws-infrastructure/          # IAM roles and policies
│   ├── codebuild-policy.json
│   ├── codebuild-role.json
│   └── kubectl-role.json
├── k8s/                        # Kubernetes manifests
│   ├── namespace.yaml
│   ├── deployment.yaml
│   └── service.yaml
├── cdk/                        # CDK configuration
│   └── bats-config.json
├── buildspec.yml               # CodeBuild deployment pipeline
└── README.md
```

## Deployment Process

1. **Triggered by**: Main application repository build completion
2. **Receives**: Docker image URI via environment variable
3. **Deploys**: Updated application to EKS cluster
4. **Monitors**: Rollout status and pod health

## Environment Variables

- `IMAGE_URI`: Docker image URI from ECR (passed from main app build)
- `EKS_CLUSTER_NAME`: Target EKS cluster name
- `AWS_DEFAULT_REGION`: AWS region for deployment

## Prerequisites

- EKS cluster: `query-profiler-dashboard`
- ECR repository: `query-profiler-frontend`
- CodeBuild service role with EKS and Parameter Store permissions
- kubectl role for cluster access

## Manual Deployment

To deploy manually:

```bash
# Set the image URI
export IMAGE_URI=869620365293.dkr.ecr.us-east-1.amazonaws.com/query-profiler-frontend:latest

# Update kubeconfig
aws eks update-kubeconfig --region us-east-1 --name query-profiler-dashboard

# Apply manifests
kubectl apply -f k8s/namespace.yaml
kubectl apply -f k8s/deployment.yaml
kubectl apply -f k8s/service.yaml

# Update deployment with new image
kubectl set image deployment/query-profiler-frontend query-profiler-frontend=$IMAGE_URI -n query-profiler

# Check rollout status
kubectl rollout status deployment/query-profiler-frontend -n query-profiler
```

## Monitoring

Check deployment status:
```bash
kubectl get pods -n query-profiler
kubectl get services -n query-profiler
kubectl logs -f deployment/query-profiler-frontend -n query-profiler
```