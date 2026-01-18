# Amazon ECS Immersion Day Workshop

## Overview

Welcome to the **Amazon Elastic Container Service Immersion Day!** This comprehensive workshop provides hands-on experience with Amazon ECS and AWS Fargate, guiding you through deploying, managing, and scaling containerized applications on AWS.

**Amazon Elastic Container Service (ECS)** is a fully managed **Container Orchestration Service** by AWS. It allows you to deploy, manage, and scale containerized applications quickly and efficiently.

**AWS Fargate** is a **Serverless** service, pay-as-you-go, that allows you to build applications without worrying about infrastructure. By delegating server management, resource allocation, and scaling to AWS, this helps enhance operational capabilities, accelerate the process of bringing ideas to production, and optimize TCO (Total Cost of Ownership).

## Architecture

The workshop demonstrates deploying a microservices retail store application using ECS with the following components:

- **Amazon ECS Cluster** - Logical grouping of tasks and services
- **ECS Task Definitions** - Blueprint for container configurations
- **ECS Services** - Maintain desired number of running tasks
- **Application Load Balancer (ALB)** - Traffic distribution and health checks
- **AWS Fargate** - Serverless compute for containers
- **ECS Managed Instances** - Fully managed EC2 instances for ECS
- **ECS Service Connect** - Service-to-service communication
- **VPC Networking** - Network configuration and security groups
- **Auto Scaling** - Automatic scaling based on metrics
- **CloudWatch Container Insights** - Monitoring and observability

## Learning Objectives

By completing this workshop, you will:

- Understand the fundamental concepts of Amazon ECS and AWS Fargate: **cluster**, **task**, and **service**
- Deploy autoscaling for workloads to meet changing load levels
- Monitor workload behavior through logs, metrics, and traces
- Grasp networking in AWS Fargate and advanced networking concepts
- Explore security aspects, including secure credential and secret management
- Learn different workload deployment strategies on Amazon ECS
- Understand ECS Express Mode for simplified deployments
- Work with ECS Managed Instances for infrastructure flexibility
- Implement Service Connect for service-to-service communication

## Workshop Structure

1. **Introduction** - Overview of Amazon ECS and AWS Fargate
2. **Fundamentals** - Core ECS components (Cluster, Service, Task, Task Definition)
3. **Preparation** - Environment setup, cluster creation, and service deployment
4. **Express Mode** - Simplified deployment with automatic infrastructure provisioning
5. **Managed Instances** - Using ECS Managed Instances for compute flexibility
6. **Auto Scaling** - Target tracking, step scaling, and scheduled scaling
7. **Networking** - Network modes and ECS Service Connect
8. **Observability** - Monitoring and logging
9. **Security** - Security best practices
10. **Deployments** - Deployment strategies
11. **Storage** - Persistent storage options
12. **Cost Optimizations** - Cost optimization techniques
13. **Clean Up** - Resource cleanup and cost optimization

## Prerequisites

- AWS Account with appropriate permissions (AdministratorAccess recommended for workshop)
- Basic understanding of containerization concepts
- Familiarity with Docker fundamentals
- Knowledge of AWS networking (VPC, subnets, security groups)
- AWS CLI installed and configured
- Basic knowledge of Linux command line

**Note**: This is not an introductory workshop on containers. Participants need foundational knowledge about Containers, specifically:
- Understanding of **Containers**, including the ability to build containers using **Dockerfile**
- Basic knowledge and familiarity with **AWS Management Console**, **AWS APIs**

## Getting Started

### Local Development

This workshop uses Hugo static site generator for documentation:

```bash
# Install Hugo (if not already installed)
# On macOS
brew install hugo

# On Ubuntu/Debian
sudo apt-get install hugo

# On Windows
# Download from https://gohugo.io/getting-started/installing/

# Clone and navigate to workshop directory
cd Amazon-ECS-Immersion-Day

# Start local development server
hugo server -D

# Build static site
hugo
```

The workshop will be available at `http://localhost:1313/`

### Workshop Execution

1. Follow the sequential workshop modules (1-13)
2. Complete hands-on exercises in each section
3. Validate your progress using provided test procedures
4. Clean up resources to avoid unnecessary charges

## Key Technologies

- **Amazon ECS**: Fully managed container orchestration service
- **AWS Fargate**: Serverless compute engine for containers
- **Amazon ECR**: Managed container registry
- **Application Load Balancer**: Layer 7 load balancing
- **ECS Service Connect**: Service discovery and connectivity
- **ECS Managed Instances**: Fully managed EC2 instances for ECS
- **ECS Express Mode**: Simplified deployment with automatic infrastructure
- **Amazon VPC**: Virtual private cloud networking
- **CloudWatch Container Insights**: Container monitoring and observability
- **Application Auto Scaling**: Automatic scaling based on metrics

## Workshop Features

- **Bilingual Support**: Available in English and Vietnamese
- **Interactive Learning**: Step-by-step guided exercises with AWS CLI commands
- **Real-world Scenarios**: Deploy a complete microservices retail store application
- **Best Practices**: AWS-recommended configurations and patterns
- **Cost Optimization**: Resource cleanup procedures and cost-saving tips
- **Comprehensive Coverage**: From fundamentals to advanced topics

## Application Overview

The workshop uses a microservices retail store application that simulates a simple web store where customers can:
- Browse product catalogs
- Add products to cart
- Complete orders through checkout process

**Application Components:**
- **UI** - Front-end user interface
- **Catalog** - Product listing and details API
- **Cart** - Shopping cart management API
- **Checkout** - Checkout process coordination API
- **Orders** - Order processing API

## Project Structure

```
Amazon-ECS-Immersion-Day/
├── content/           # Workshop content (English and Vietnamese)
│   ├── 1-introduction/
│   ├── 2-fundamentals/
│   ├── 3-prepare/
│   ├── 4-express-mode/
│   ├── 5-managed-instances/
│   ├── 6-auto-scaling/
│   ├── 7-networking/
│   ├── 8-observability/
│   ├── 9-security/
│   ├── 10-deployments/
│   ├── 11-storage/
│   ├── 12-cost-optimizations/
│   └── 13-clean-up/
├── static/            # Static assets (images, CSS, fonts)
├── themes/            # Hugo theme configuration
├── public/            # Generated static site
├── config.toml       # Hugo configuration
└── resources/        # Task definition templates and configuration files
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
