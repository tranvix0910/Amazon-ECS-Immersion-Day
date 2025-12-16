---
title : "Tạo Cluster & Task Definition"
date :  2025-01-01
weight : 2
chapter : false
pre : " <b> 3.2. </b> "  
---

#### Tạo Cluster

Tiến hành tạo Amazon ECS Cluster có tên `retail-store-ecs-cluster` với [CloudWatch Container Insights](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/cloudwatch-container-insights.html).

Container Insight thu thập, tổng hợp và tóm tắt các chỉ số (metrics) và nhật ký (logs) từ các ứng dụng Container.

```bash
aws ecs create-cluster --cluster-name retail-store-ecs-cluster --capacity-providers FARGATE FARGATE_SPOT --default-capacity-provider-strategy capacityProvider=FARGATE,weight=1 --settings name=containerInsights,value=enhanced
```

Sau khi chạy lệnh sẽ được kết quả sau và có thể thấy **CloudWatch Container Insights** đã được cấu hình thành công.

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

![Cấu hình ECS với AWS CLI](/images/3-prepare/3.2-cluster-task_definitions/1.png)

Kiểm tra trên AWS Console để xác nhận cluster đã được tạo thành công.

![Cấu hình ECS với AWS CLI](/images/3-prepare/3.2-cluster-task_definitions/2.png)

#### Tạo Role

Tạo hai IAM Role cần thiết cho ECS Task:

- **retailStoreEcsTaskExecutionRole**: cho phép ECS pull image, ghi log, truy cập secrets.

- **retailStoreEcsTaskRole**: cấp quyền cho ứng dụng bên trong container.

```bash
aws iam create-role --role-name retailStoreEcsTaskExecutionRole --assume-role-policy-document file://ecs-task-trust.json
```
Nội dung file `ecs-task-trust.json`:
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

![Cấu hình ECS với AWS CLI](/images/3-prepare/3.2-cluster-task_definitions/6.png)

Gán quyền cho **retailStoreEcsTaskExecutionRole**:

```bash
aws iam attach-role-policy --role-name retailStoreEcsTaskExecutionRole --policy-arn arn:aws:iam::aws:policy/service-role/AmazonECSTaskExecutionRolePolicy
```

![Cấu hình ECS với AWS CLI](/images/3-prepare/3.2-cluster-task_definitions/7.png)

Tạo **retailStoreEcsTaskRole**:

```bash
aws iam create-role --role-name retailStoreEcsTaskRole --assume-role-policy-document file://ecs-task-trust.json
```

![Cấu hình ECS với AWS CLI](/images/3-prepare/3.2-cluster-task_definitions/8.png)

#### Tạo Task Definition

Tạo Task Definition với tên `retail-store-ecs-ui` được sử dụng cho **UI Service** và cấu hình như sau:

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
Chú ý **ACCOUNT_ID** là ID của tài khoản AWS, **AWS_REGION** là region của bạn.
{{% /notice %}}

Tạo file json với nội dung như trên và đặt tên file là `retail-store-ecs-ui-taskdef.json`.

Chạy lệnh sau để tạo Task Definition.

```bash
aws ecs register-task-definition --cli-input-json file://retail-store-ecs-ui-taskdef.json
```

![Cấu hình ECS với AWS CLI](/images/3-prepare/3.2-cluster-task_definitions/3.png)

Kiểm tra trên AWS Console để xác nhận Task Definition đã được tạo thành công.

![Cấu hình ECS với AWS CLI](/images/3-prepare/3.2-cluster-task_definitions/4.png)

Có thể truy xuất Task Definition đã được tạo thành công bằng cách chạy lệnh sau:

```bash
aws ecs describe-task-definition --task-definition retail-store-ecs-ui
```

![Cấu hình ECS với AWS CLI](/images/3-prepare/3.2-cluster-task_definitions/5.png)


