# Deploy Applications on Amazon Elastic Container Service (ECS)

## Overview

This workshop provides hands-on experience deploying containerized applications using Amazon Elastic Container Service (ECS). Part of the comprehensive AWS First Cloud Journey learning path, this workshop guides you through the complete process of setting up, configuring, and managing containerized applications on AWS.

## Architecture

The workshop demonstrates deploying applications using ECS with the following components:
- Amazon ECS Cluster
- ECS Task Definitions and Services
- Application Load Balancer (ALB)
- Amazon ECR for container registry
- AWS CloudMap for service discovery
- VPC networking configuration

## Learning Objectives

By completing this workshop, you will:
- Understand core Amazon ECS concepts and architecture
- Create and manage ECS clusters using both EC2 and Fargate launch types
- Define ECS tasks and services for containerized applications
- Configure Application Load Balancer for traffic distribution
- Implement service discovery using AWS CloudMap
- Deploy and test real-world containerized applications
- Apply best practices for container orchestration on AWS

## Workshop Structure

1. **Introduction** - ECS fundamentals and architecture overview
2. **Preparation** - Environment setup and prerequisites
3. **Prepare for Deployment** - Container and infrastructure preparation
4. **Register Namespace in CloudMap** - Service discovery configuration
5. **Create ECS Cluster** - Cluster setup and configuration
6. **Create Task Definition** - Container task specifications
7. **Configure ALB** - Load balancer setup for traffic routing
8. **Create ECS Services** - Service deployment and management
9. **Test Result** - Validation and testing procedures
10. **Clean Up** - Resource cleanup and cost optimization

## Prerequisites

- AWS Account with appropriate permissions
- Basic understanding of containerization concepts
- Familiarity with Docker fundamentals
- Knowledge of AWS networking (VPC, subnets, security groups)

## Getting Started

### Local Development

This workshop uses Hugo static site generator for documentation:

```bash
# Install Hugo (if not already installed)
# On macOS
brew install hugo

# On Ubuntu/Debian
sudo apt-get install hugo

# Clone and navigate to workshop directory
cd /path/to/000016-AmazonECSECR

# Start local development server
hugo server -D

# Build static site
hugo
```

### Workshop Execution

1. Follow the sequential workshop modules (1-10)
2. Complete hands-on exercises in each section
3. Validate your progress using provided test procedures
4. Clean up resources to avoid unnecessary charges

## Key Technologies

- **Amazon ECS**: Container orchestration service
- **Amazon ECR**: Managed container registry
- **AWS Fargate**: Serverless compute for containers
- **Application Load Balancer**: Layer 7 load balancing
- **AWS CloudMap**: Service discovery for cloud resources
- **Amazon VPC**: Virtual private cloud networking

## Workshop Features

- **Bilingual Support**: Available in English and Vietnamese
- **Interactive Learning**: Step-by-step guided exercises
- **Real-world Scenarios**: Practical application deployment
- **Best Practices**: AWS-recommended configurations
- **Cost Optimization**: Resource cleanup procedures

## Project Structure

```
000016-AmazonECSECR/
├── content/           # Workshop content and modules
├── static/           # Static assets (images, CSS, fonts)
├── themes/           # Hugo theme configuration
├── public/           # Generated static site
├── config.toml       # Hugo configuration
└── resources/        # CloudFormation templates
```

## Support and Community

- **AWS Study Group Blog**: [AWS Blogs](https://aws.amazon.com/blogs)
- **Facebook Community**: [AWS Study Group](https://www.facebook.com/groups/awsstudygroupfcj)
- **Author**: journeyoftheaverageguy@gmail.com

## Related Workshops

This workshop is part of the AWS First Cloud Journey series featuring 190+ hands-on workshops covering:
- Compute Services (EC2, Lambda, ECS, EKS)
- Storage Solutions (S3, EBS, EFS)
- Database Services (RDS, DynamoDB, Aurora)
- Networking (VPC, CloudFront, Route 53)
- Security and Identity (IAM, Cognito, KMS)
- DevOps and CI/CD (CodePipeline, CodeBuild, CodeDeploy)

## License

This workshop is part of the AWS First Cloud Journey educational initiative. Please refer to the repository license for usage terms.

## Contributing

Contributions to improve workshop content are welcome. Please follow the established format and ensure all examples are tested before submission.
