# Waizly DevOps Documentation

## Tech Stack

### GitHub & GitHub Actions
- **Repository**: Used for storing code.
- **Pipeline**: Using GitHub Actions for CI/CD.
- **Variables and Secrets**: Stored in GitHub and consumed by GitHub Actions to create ConfigMap and Secret on EKS.

### Helm
- **Deployment Config Template**: Templates for deployment configurations.
- **Usage**: Specify necessary variables like resource limits, namespace, app name, environment, etc.

### Amazon ECR
- **Image Repository**: Stores Docker images after build.
- **Deployment**: Images are pulled from ECR during deployment.

### Amazon EKS
- **Orchestration**: Used for deploying microservices.
- **Nodegroup**: Using AMD64 architecture for cost efficiency.
- **Installation**: Currently using CloudFormation, with plans to switch to Terraform.
- **Setup**: Detailed setup instructions for EKS cluster can be found [here](./EKS%20Cluster%20Installation.md).

### Amazon S3
- **Storage**: Used for storing Helm templates.

### Nginx
- **Ingress**: Manages service ingress per namespace.

### Redis
- **Cache**: Deployed per service on EC2.
- **Worker**: Deployed per service on EC2.

### ArangoDB
- **Deployment**: Deployed on EC2.

### Load Balancer Nginx
- **Configuration**: Manages backend and frontend domain routing.

### Amazon RDS
- **Database**: Master-slave configuration.

### Environments
- **Staging and Production**: Each environment has dedicated resources.

### Logging
- **Fluentd, Elasticsearch, Kibana**: For logging.
- **Fluentd Operator**: Installed on EKS, sends logs to Elasticsearch on EC2, and uses Kibana for visualization.

## Implementation

### Pipeline
- **CI/CD**: Implemented using GitHub Actions.

### EKS Cluster
- **Setup**: Detailed setup instructions for EKS cluster can be found [here](./EKS%20Cluster%20Installation.md).

## Maintenance

### EKS Cluster Updates
- **Strategy**: Need to develop a strategy for updating the EKS cluster with no downtime.

## Improvement

### Terraform Pipeline
- **Resource Creation**: Plan to use Terraform for creating resources in AWS.

### Vault Integration
- **Current Status**: Integration with AWS is still in progress.
