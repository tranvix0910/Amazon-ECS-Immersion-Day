---
title : "Introduction"
date :  2025-01-01
weight : 1
chapter : false
pre : " <b> 1. </b> "  
---

### Amazon ECS

![Amazon ECS](/images/1-introduction/1.png)

Amazon Elastic Container Service (ECS) is a fully managed container orchestration service by AWS that simplifies deploying, managing, and scaling containerized applications. This service integrates closely with other AWS services, providing a secure, easy-to-use solution for running container workloads in the cloud environment and on-premises infrastructure through Amazon ECS Anywhere.

{{< youtube o6M8GXwDr9w >}}

### AWS Fargate

![AWS Fargate](/images/1-introduction/2.png)

AWS Fargate is a serverless compute engine, operating on a pay-as-you-go model, allowing you to focus on building applications without managing servers. AWS Fargate is compatible with both Amazon Elastic Container Service (Amazon ECS) and Amazon Elastic Kubernetes Service (Amazon EKS).

{{< youtube yi22xrvPnPk >}}

### Benefits of AWS Fargate

- **Focus on applications, not infrastructure**: Deploy and manage applications without bearing operational tasks such as scaling, patching, security, and server management.

- **Comprehensive visibility through detailed monitoring**: Monitor applications using built-in integrations with AWS services such as **Amazon CloudWatch Container Insights**, or collect metrics and logs through third-party tools.

- **Enhanced security through workload isolation**: Enhance security from design with workload isolation mechanism. Each workload running on AWS Fargate uses a separate, single-use compute instance, dedicated to one tenant, and isolated by a virtualization layer. Each Amazon ECS task or Kubernetes pod will run on a newly allocated instance. To learn more about AWS Fargate architecture, refer to [AWS Fargate Security Whitepaper](https://d1.awsstatic.com/whitepapers/AWS_Fargate_Security_Overview_Whitepaper.pdf).

- **Cost-effective optimization**: Only pay for compute resources actually used, with no upfront costs. Can further reduce costs by using Savings Plans, Fargate Spot, or AWS Graviton processors.

![Benefits of AWS Fargate](/images/1-introduction/3.png)


### Overview of Deployed Microservices Application

Most labs in this workshop use a Microservices application to provide container components for practice exercises. The sample application simulates a simple web store where customers can browse product catalogs, add products to cart, and complete orders through the checkout process.

![Overview of deployed application](/images/1-introduction/4.png)

![Overview of deployed application](/images/1-introduction/5.png)

The application includes the following components and dependencies:

| Component   | Description                                                                                      |
| ------------ | ------------------------------------------------------------------------------------------ |
| **UI**       | Provides front-end user interface and aggregates API calls to other Services |
| **Catalog**  | API providing product list and details                                                |
| **Cart**     | API managing customer shopping cart                                                        |
| **Checkout** | API coordinating the entire checkout process                                                 |
| **Orders**   | API receiving and processing customer orders                                             |

### Containerizing the Application

Before deploying a workload to Amazon ECS, that workload needs to be packaged as a **container image** and pushed (published) to a **container registry**.

Such basic container knowledge is not within the scope of this workshop, and the application already has container images available in **Amazon Elastic Container Registry (Amazon ECR)** for use in labs.

The table below provides paths to ECR Public repositories for each component, as well as Dockerfiles used to build each component.

| Component        | ECR Public repository | Dockerfile |
| ----------------- | --------------------- | ---------- |
| **UI**            | [Repository](https://gallery.ecr.aws/aws-containers/retail-store-sample-ui)            | [Dockerfile](https://github.com/aws-containers/retail-store-sample-app/blob/v1.2.3/src/ui/Dockerfile) |
| **Catalog**       | [Repository](https://gallery.ecr.aws/aws-containers/retail-store-sample-catalog)            | [Dockerfile](https://github.com/aws-containers/retail-store-sample-app/blob/v1.2.3/src/catalog/Dockerfile) |
| **Shopping cart** | [Repository](https://gallery.ecr.aws/aws-containers/retail-store-sample-cart)            | [Dockerfile](https://github.com/aws-containers/retail-store-sample-app/blob/v1.2.3/src/cart/Dockerfile) |
| **Checkout**      | [Repository](https://gallery.ecr.aws/aws-containers/retail-store-sample-checkout)            | [Dockerfile](https://github.com/aws-containers/retail-store-sample-app/blob/v1.2.3/src/checkout/Dockerfile) |
| **Orders**        | [Repository](https://gallery.ecr.aws/aws-containers/retail-store-sample-orders)            | [Dockerfile](https://github.com/aws-containers/retail-store-sample-app/blob/v1.2.3/src/orders/Dockerfile) |
