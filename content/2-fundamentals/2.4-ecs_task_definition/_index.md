---
title : "Amazon ECS Task Definition"
date :  2025-01-01
weight : 2.4
chapter : false
pre : " <b> 2.4. </b> "  
---

### Amazon ECS Task Definition

A Task Definition is a JSON blueprint describing how one or more Docker containers will run in AWS ECS. It is like a recipe or template containing all the necessary configurations to launch containers, including Docker image, CPU, memory, port mappings, environment variables, logging, and more.

#### Main Components of Task Definition

Task Definition is divided into two levels of configuration: **Task-level parameters** and **Container definitions**.

**Task-Level Parameters**

- **Family**: Name of the Task Definition family. This is the name used to identify the task definition. For example: "my-web-app", "data-processor". When updating a task definition, a new revision is created but the family name remains the same.​

- **Task Size (CPU & Memory)**: Total amount of CPU and memory allocated for the entire task. This is different from CPU/memory at the container level. With Fargate, these values are required and must be specific supported values (e.g., 0.25 vCPU with 0.5GB-1GB memory). With EC2, these values are optional.​

- **Network Mode**: How containers will connect to the network:​

    - **awsvpc**: Each task receives its own Elastic Network Interface (ENI) with a private IP address from the VPC. This allows the task to operate like an EC2 instance in the VPC. This is the only mode supported by Fargate and recommended for EC2.​

    - **bridge**: Containers connect through Docker bridge network. Containers on the same host can communicate with each other. Port mapping is used to expose ports.​

    - **host**: Containers share the network namespace with the host. Container ports are mapped directly to host ports. Not recommended as it reduces isolation.

    - **none**: Containers have no network connectivity.​

- **Launch Type**: Where the task will run:

    - **EC2**: Task runs on EC2 instances managed by you.

    - **Fargate**: AWS manages infrastructure, you only need to define requirements

- **Execution Role (Task Execution Role ARN)**: IAM role for the ECS Container Agent. This role allows the agent to pull Docker images from ECR, push logs to CloudWatch, get secrets from Secrets Manager.​

- **Task Role**: IAM role for the application inside the container. This role grants permissions for code to access AWS services such as S3, DynamoDB, SNS, etc.​

- **Platform**: CPU architecture type (x86_64 or ARM64). With ARM64, you can use AWS Graviton processors which are more cost-effective.​

**Container Definitions**

A Task Definition can contain one or more container definitions. Each container definition describes a separate container that will run as part of the task.

- **Name**: Name of the container (e.g., `web-server`, `database-client`). Required.​

- **Image**: Docker image URI to use. Can be from Docker Hub (e.g., `nginx:latest`) or from Amazon ECR (e.g., `123456789012.dkr.ecr.us-east-1.amazonaws.com/my-app:1.0.0`). Required.​

- **CPU**: Number of CPU units allocated to this container. 1 vCPU = 1024 CPU units. If task size is 1 vCPU and there are 2 containers, you can allocate 512 units to each container. If set to 0, the container will use remaining CPU units not allocated to other containers.​

- **Memory**: Memory in MB. It can be:

    - **hard limit (memory)**: Maximum memory the container can use. If exceeded, the container is stopped.

    - **soft limit (memoryReservation)**: Memory guaranteed to be available for the container.​

- **Port Mappings**: Maps container ports to host ports. For example: containerPort 8080 → hostPort 0 means the container runs on port 8080, but exposes through dynamic port mapping on the host.​

    - **Essential**: Boolean flag. If **true**, if this container fails, the entire task will be stopped. At least one container must have **essential=true**. If false, the task will continue running even if this container stops.​

    - **Environment Variables**: Environment variables passed to the container. For example: `SERVER_PORT=8080`, `DATABASE_HOST=db.example.com`.​

- **Logging Configuration**: How logs will be written. Options include:

    - **awslogs**: Logs pushed to AWS CloudWatch

    - **splunk**: Logs pushed to Splunk

    - **datadog**: Logs pushed to Datadog

    - **awsfirelens**: Use Fluent Bit or Fluentd

Example CloudWatch configuration:

```json
"logConfiguration": {
  "logDriver": "awslogs",
  "options": {
    "awslogs-group": "/ecs/my-task",
    "awslogs-region": "us-east-1",
    "awslogs-stream-prefix": "ecs"
  }
}
```

- **Mount Points**: Maps volumes into containers. Includes:

    - **sourceVolume**: Name of the volume defined at task-level

    - **containerPath**: Path inside the container where the volume is mounted (e.g., /data)

    - **readOnly**: Whether it is read-only​

- **Health Check**: Configuration to check container health:

```json
"healthCheck": {
  "command": ["CMD-SHELL", "curl -f http://localhost:8080/health || exit 1"],
  "interval": 30,
  "timeout": 5,
  "retries": 3,
  "startPeriod": 0
}
```

**Volumes (Data Storage)​**

- **Volumes**: Volumes are defined at task-level but are mounted into containers through mount points. Volume types include:

    - **EBS Volumes**: Amazon Elastic Block Store volumes for persistent storage (EC2 only)

    - **EFS Volumes**: Amazon Elastic File System for shared persistent storage between multiple tasks/instances

    - **FSx Volumes**: FSx for Windows File Server volumes

    - **Bind Mounts**: Mount a path from the host EC2 instance into the container

    - **Docker Volumes**: Docker-managed volumes

Example volume definition:

```json
"volumes": [
  {
    "name": "my-efs",
    "efsVolumeConfiguration": {
      "fileSystemId": "fs-1234567",
      "transitEncryption": "ENABLED"
    }
  },
  {
    "name": "my-bind-volume",
    "host": {
      "sourcePath": "/mnt/data"
    }
  }
]
```

**Versioning and Revisions​**

Task Definitions are immutable. This means you cannot edit an existing task definition. Instead, you must create a new revision.​

A revision is a copy of the current task definition with changes applied. Each task definition family can have multiple revisions:

- **my-app:1 (revision 1)**.

- **my-app:2 (revision 2)**.

- **my-app:3 (revision 3)**.

When updating a service or running a task, specify **family:revision** to use (e.g., "my-app:3"). This allows easy rollback to an old revision if there is an issue by simply updating the service to use the previous revision.​

Advantages of this architecture:

- **Audit trail**: Can view all changes that have been made.

- **Easy rollback**: Roll back to an old revision if needed.

- **Version control**: Like version control for infrastructure.

**Task Definition JSON**

```json
{
  "family": "my-web-app",
  "cpu": "256",
  "memory": "512",
  "networkMode": "awsvpc",
  "requiresCompatibilities": ["FARGATE"],
  "executionRoleArn": "arn:aws:iam::123456789012:role/ecsTaskExecutionRole",
  "taskRoleArn": "arn:aws:iam::123456789012:role/ecsTaskRole",
  "containerDefinitions": [
    {
      "name": "web-server",
      "image": "123456789012.dkr.ecr.us-east-1.amazonaws.com/my-app:1.0.0",
      "cpu": 256,
      "memory": 512,
      "essential": true,
      "portMappings": [
        {
          "containerPort": 8080,
          "hostPort": 8080,
          "protocol": "tcp"
        }
      ],
      "logConfiguration": {
        "logDriver": "awslogs",
        "options": {
          "awslogs-group": "/ecs/my-web-app",
          "awslogs-region": "us-east-1",
          "awslogs-stream-prefix": "ecs"
        }
      },
      "environment": [
        {
          "name": "ENVIRONMENT",
          "value": "production"
        },
        {
          "name": "DATABASE_HOST",
          "value": "db.example.com"
        }
      ]
    }
  ]
}
```

