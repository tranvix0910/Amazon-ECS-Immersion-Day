---
title : "Create Cluster & Task Definition"
date :  2025-01-01
weight : 2
chapter : false
pre : " <b> 3.2. </b> "  
---

#### Create Cluster

Proceed to create Amazon ECS Cluster named `retail-store-ecs-cluster` with [CloudWatch Container Insights](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/cloudwatch-container-insights.html).

Container Insight collects, aggregates and summarizes metrics and logs from Container applications.

```bash
aws ecs create-cluster --cluster-name retail-store-ecs-cluster --capacity-providers FARGATE FARGATE_SPOT --default-capacity-provider-strategy capacityProvider=FARGATE,weight=1 --settings name=containerInsights,value=enhanced
```

After running the command, you will get the following result and can see **CloudWatch Container Insights** has been configured successfully.

```json
{
    "cluster": {
        "clusterArn": "arn:aws:ecs:ap-northeast-2:111111111111:cluster/retail-store-ecs-cluster",
        "clusterName": "retail-store-ecs-cluster",
        "status": "ACTIVE",
        "registeredContainerInstancesCount": 0,
        "runningTasksCount": 0,
        "pendingTasksCount": 0,
        "activeServicesCount": 0,
        "statistics": [],
        "tags": [],
        "settings": [
            {
                "name": "containerInsights",
                "value": "enhanced"
            }
        ],
        "capacityProviders": [],
        "defaultCapacityProviderStrategy": []
    }
}
```

![Configure ECS with AWS CLI](/images/3-prepare/3.2-cluster-task_definitions/1.png)

Check on AWS Console to confirm cluster has been created successfully.

![Configure ECS with AWS CLI](/images/3-prepare/3.2-cluster-task_definitions/2.png)

#### Create Role

Create two IAM Roles needed for ECS Task:

- **retailStoreEcsTaskExecutionRole**: allows ECS to pull image, write logs, access secrets.

- **retailStoreEcsTaskRole**: grants permissions for application inside container.

```bash
aws iam create-role --role-name retailStoreEcsTaskExecutionRole --assume-role-policy-document file://ecs-task-trust.json
```
Content of `ecs-task-trust.json` file:
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": {
                "Service": "ecs-tasks.amazonaws.com"
            },
            "Action": "sts:AssumeRole"
        }
    ]
}
```

![Configure ECS with AWS CLI](/images/3-prepare/3.2-cluster-task_definitions/6.png)

Attach permissions to **retailStoreEcsTaskExecutionRole**:

```bash
aws iam attach-role-policy --role-name retailStoreEcsTaskExecutionRole --policy-arn arn:aws:iam::aws:policy/service-role/AmazonECSTaskExecutionRolePolicy
```

![Configure ECS with AWS CLI](/images/3-prepare/3.2-cluster-task_definitions/7.png)

Create **retailStoreEcsTaskRole**:

```bash
aws iam create-role --role-name retailStoreEcsTaskRole --assume-role-policy-document file://ecs-task-trust.json
```

![Configure ECS with AWS CLI](/images/3-prepare/3.2-cluster-task_definitions/8.png)

#### Create Task Definition

Create Task Definition named `retail-store-ecs-ui` used for **UI Service** with the following configuration:

```json
{
    "family": "retail-store-ecs-ui",
    "networkMode": "awsvpc",
    "requiresCompatibilities": [
        "FARGATE"
    ],
    "cpu": "1024",
    "memory": "2048",
    "runtimePlatform": {
        "cpuArchitecture": "X86_64",
        "operatingSystemFamily": "LINUX"
    },
    "containerDefinitions": [
        {
            "name": "application",
            "image": "public.ecr.aws/aws-containers/retail-store-sample-ui:1.2.3",
            "portMappings": [
                {
                    "name": "application",
                    "containerPort": 8080,
                    "hostPort": 8080,
                    "protocol": "tcp",
                    "appProtocol": "http"
                }
            ],
            "essential": true,
            "linuxParameters": {
                "initProcessEnabled": true
            },
            "healthCheck": {
                "command": [
                    "CMD-SHELL",
                    "curl -f http://localhost:8080/actuator/health || exit 1"
                ],
                "interval": 10,
                "timeout": 5,
                "retries": 3,
                "startPeriod": 60
            },
            "logConfiguration": {
                "logDriver": "awslogs",
                "options": {
                    "awslogs-group": "retail-store-ecs-tasks",
                    "awslogs-region": "$AWS_REGION",
                    "awslogs-stream-prefix": "ui-service"
                }
            }
        }
    ],
    "executionRoleArn": "arn:aws:iam::${ACCOUNT_ID}:role/retailStoreEcsTaskExecutionRole",
    "taskRoleArn": "arn:aws:iam::${ACCOUNT_ID}:role/retailStoreEcsTaskRole"
}
```

{{% notice note %}}
Note **ACCOUNT_ID** is your AWS account ID, **AWS_REGION** is your region.
{{% /notice %}}

Create a json file with the above content and name it `retail-store-ecs-ui-taskdef.json`.

Run the following command to create Task Definition.

```bash
aws ecs register-task-definition --cli-input-json file://retail-store-ecs-ui-taskdef.json
```

![Configure ECS with AWS CLI](/images/3-prepare/3.2-cluster-task_definitions/3.png)

Check on AWS Console to confirm Task Definition has been created successfully.

![Configure ECS with AWS CLI](/images/3-prepare/3.2-cluster-task_definitions/4.png)

You can retrieve the successfully created Task Definition by running the following command:

```bash
aws ecs describe-task-definition --task-definition retail-store-ecs-ui
```

![Configure ECS with AWS CLI](/images/3-prepare/3.2-cluster-task_definitions/5.png)

