---
title : "Cập nhật Services"
date :  2025-01-01
weight : 4
chapter : false
pre : " <b> 3.4. </b> "  
---

Trong phần này chúng ta sẽ cập nhật một ECS Service. Quy trình này thường được sử dụng trong các tình huống như thay đổi container image hoặc điều chỉnh cấu hình ứng dụng.

**Environment variables** là một trong những cơ chế chính để cấu hình workload chạy trong container, bất kể dùng Orchestrator nào. Chúng ta sẽ tiến hành thay đổi cấu hình của UI Service để sử dụng các biến môi trường mới.

**Docker Image** không thể thay đổi sau khi được build, vì vậy biến môi trường là cách đơn giản và linh hoạt để cấu hình và điều chỉnh hành vi ứng dụng khi container đang chạy.

Trong trường hợp này, sẽ sử dụng biến **RETAIL_UI_THEME**, biến này sẽ thay đổi màu giao diện mặc định của ứng dụng.

#### Khai báo biến trong Task Definition

Biến môi trường trong ECS task definition được khai báo dưới dạng cặp name–value, ví dụ:

```json
"environment": [
    {
        "name": "RETAIL_UI_THEME",
        "value": "green"
    }
]
```

Tiếp theo sẽ tiến hành cập nhật file json `retail-store-ecs-ui-updated-taskdef.json` theo biến môi trường mới.

```json
{
    "family": "retail-store-ecs-ui",
    "executionRoleArn": "arn:aws:iam::${ACCOUNT_ID}:role/retailStoreEcsTaskExecutionRole",
    "taskRoleArn": "arn:aws:iam::${ACCOUNT_ID}:role/retailStoreEcsTaskRole",
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
            "environment": [
                {
                    "name": "RETAIL_UI_THEME",
                    "value": "green"
                }
            ],
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
    ]
}
```

Sau đó, sử dụng lệnh **register-task-definition** để đăng ký task definition mới:

```bash
aws ecs register-task-definition \
  --cli-input-json file://retail-store-ecs-ui-updated-taskdef.json
```

![Cập nhật Task Definition](/images/3-prepare/3.4-updated_services/1.png)


{{% notice note %}}
ECS task definition là bất biến (immutable), nghĩa là không thể chỉnh sửa sau khi đã tạo.
Thay vào đó, lệnh trên sẽ tạo một revision mới, là bản sao của task definition cũ nhưng đã được cập nhật các tham số mới.
{{% /notice %}}

#### Kiểm tra các revision của Task Definition

Có thể kiểm tra xem hiện tại có bao nhiêu revision bằng lệnh:

```bash
aws ecs list-task-definitions --family-prefix retail-store-ecs-ui --sort DESC --max-items 2
```

Kết quả sẽ hiển thị các revision của task definition UI:

```bash
{
    "taskDefinitionArns": [
        "arn:aws:ecs:us-west-2:XXXXXXXXXXXX:task-definition/retail-store-ecs-ui:2",
        "arn:aws:ecs:us-west-2:XXXXXXXXXXXX:task-definition/retail-store-ecs-ui:1"
    ]
}
```

![Kiểm tra các revision của Task Definition](/images/3-prepare/3.4-updated_services/3.png)

Hoặc có thể kiểm tra trực tiếp trên AWS Console:

![Cập nhật Task Definition](/images/3-prepare/3.4-updated_services/2.png)

#### Cập nhật ECS Service để sử dụng revision mới

Tiếp theo, cập nhật ECS service để dùng task definition mới:

```bash
aws ecs update-service --cluster retail-store-ecs-cluster --service ui --task-definition retail-store-ecs-ui
```

![Cập nhật ECS Service để sử dụng revision mới](/images/3-prepare/3.4-updated_services/4.png)

#### Kiểm tra kết quả

![Kiểm tra kết quả](/images/3-prepare/3.4-updated_services/5.png)

![Kiểm tra kết quả](/images/3-prepare/3.4-updated_services/6.png)