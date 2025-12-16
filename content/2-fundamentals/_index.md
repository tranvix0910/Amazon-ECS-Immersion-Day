---
title : "Fundamentals"
date :  2025-01-01
weight : 2
chapter : false
pre : " <b> 2. </b> "  
---

### Amazon ECS Core Components

![Amazon ECS Core Components](/images/2-fundamentals/1.png)

Amazon ECS has 4 main components:

- **Cluster**: The central management space in Amazon ECS, where all application services and tasks are organized and operated.

- **Service**: Represents a group of identical tasks, responsible for maintaining the desired number of tasks running, while supporting automatic restart or scaling when needed.

{{< youtube fXiUlXy5kRA >}}

- **Task**: The execution unit in ECS, consisting of one or more containers working together to perform a specific application function.

- **Task Definition**: A detailed description of how a task is run, including information such as CPU, memory, container image, network configuration, IAM role, and other necessary settings.

{{< youtube 5uJUmGWjRZY >}}

Understanding these components is very important for using Amazon ECS effectively in managing containerized applications. Each component plays a key role in the overall architecture and operation of ECS deployments.

### Layers

There are 3 Layers in ECS:

- **Capacity**: Infrastructure where containers run.

- **Controller**: Deploy and manage applications running on containers.

- **Provisioning**: Tools to interact with the scheduler to deploy and manage applications and containers.

![Layers](/images/2-fundamentals/2.png)

### Capacity

Capacity (compute capacity) is the infrastructure where containers run. Below is an overview of capacity options:

#### Amazon ECS Managed Instances

- Amazon ECS Managed Instances is a compute option for Amazon ECS, allowing you to run containerized workloads on various Amazon EC2 instance types, while transferring most infrastructure management to AWS.

- With Amazon ECS Managed Instances, you can:
    - GPU acceleration.
    - Specific CPU architecture.
    - High network performance.
    - Dedicated instance types.
- Meanwhile, AWS will be responsible for:
    - Infrastructure provisioning.
    - Operating system patching.
    - Scaling.
    - Maintenance and operation of underlying infrastructure.

#### Amazon EC2 Instances

- You can choose the instance type and number of instances yourself.
- Manage capacity yourself (Auto Scaling, patching, lifecycle, etc.).

#### Serverless

- AWS Fargate is a serverless compute engine, pay-as-you-go.
- With Fargate:
    - No need to manage servers.
    - No need to plan capacity.
    - No need to isolate container workloads yourself as AWS handles security.

#### Server/VM On-Premises

- Amazon ECS Anywhere registers external instances (on-premises servers or VMs). Connect them to an ECS cluster.

#### Amazon ECS Scheduler

- Amazon ECS Scheduler is software responsible for managing and orchestrating your applications, including:
    - Deciding where tasks run.
    - Managing task lifecycle.
    - Ensuring service desired state.
