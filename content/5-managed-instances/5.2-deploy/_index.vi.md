---
title : "Triển khai với Managed Instances"
date :  2025-01-01
weight : 2
chapter : false
pre : " <b> 5.2. </b> "  
---

#### Register Task Definition

Chúng ta sẽ triển khai **UI service** (`ui-managed-instance`) bằng cách sử dụng **ECS Managed Instances** mà chúng ta đã tạo trong bước trước.

Tiến hành register một task definition mới với tên là `retail-store-ecs-ui-managed` bằng cách sử dụng file json `retail-store-ecs-ui-managed-taskdef.json` bên dưới:

```json
{
    "family": "retail-store-ecs-ui-managed",
    "networkMode": "awsvpc",
    "requiresCompatibilities": [
        "MANAGED_INSTANCES"
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
            "versionConsistency": "disabled",
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
                    "awslogs-stream-prefix": "ui-managed-service"
                }
            }
        }
    ],
    "executionRoleArn": "arn:aws:iam::${ACCOUNT_ID}:role/retailStoreEcsTaskExecutionRole",
    "taskRoleArn": "arn:aws:iam::${ACCOUNT_ID}:role/retailStoreEcsTaskRole"
}
```

Chạy lệnh sau để register task definition mới:

```bash
aws ecs register-task-definition --cli-input-json file://retail-store-ecs-ui-managed-taskdef.json
```

![Register Task Definition](/images/5-managed-instances/5.2-deploy/1.png)

#### Tạo Target Group & Application Load Balancer

Truy cập EC2 Console:

![Tạo Target Group](/images/3-prepare/3.3-services/21.png)

Chọn phần **Target Groups** và chọn **Create Target Group**.

![Tạo Target Group](/images/3-prepare/3.3-services/22.png)

Cấu hình các thông số sau:

- **Target Group Name**: `ui-managed-tg`
- **Target Type**: `IP Addresses`
- **Protocol**: `HTTP`
- **Port**: `8080`
- **VPC**: VPC chúng ta đã tạo trong bước trước.
- **Health Check Protocol**: `HTTP`
- **Health Check Path**: `/actuator/health`

![Tạo Target Group](/images/5-managed-instances/5.2-deploy/2.png)

![Tạo Target Group](/images/5-managed-instances/5.2-deploy/3.png)

![Tạo Target Group](/images/5-managed-instances/5.2-deploy/4.png)

![Tạo Target Group](/images/5-managed-instances/5.2-deploy/5.png)

Sau khi cấu hình xong chọn **Next**.

![Tạo Target Group](/images/5-managed-instances/5.2-deploy/6.png)

Kiểm tra lại các thông số và chọn **Create Target Group**.

![Tạo Target Group](/images/5-managed-instances/5.2-deploy/7.png)

Như vậy chúng ta đã tạo xong Target Group. Tiếp theo chúng ta sẽ tạo Application Load Balancer.

![Tạo Target Group](/images/5-managed-instances/5.2-deploy/8.png)

Tiếp tục truy cập vào EC2 Console và chọn **Load Balancers** và chọn **Create Load Balancer**.

![Tạo Application Load Balancer](/images/5-managed-instances/5.2-deploy/9.png)

Chọn Type là **Application Load Balancer**.

![Tạo Application Load Balancer](/images/5-managed-instances/5.2-deploy/10.png)

Cấu hình các thông số sau:

- **Load Balancer Name**: `ui-managed-alb`
- **Scheme**: `Internet-facing`

![Tạo Application Load Balancer](/images/5-managed-instances/5.2-deploy/11.png)

- **VPC**: VPC chúng ta đã tạo trong bước trước.
- Cấu hình 2 Publib Subnet cho ALB.

![Tạo Application Load Balancer](/images/5-managed-instances/5.2-deploy/12.png)

- Chọn `alb-sg` vừa tạo và chọn Target Group `ui-managed-tg`.

![Tạo Application Load Balancer](/images/5-managed-instances/5.2-deploy/13.png)

Review và chọn **Create Load Balancer**.

![Tạo Application Load Balancer](/images/5-managed-instances/5.2-deploy/14.png)

![Tạo Application Load Balancer](/images/5-managed-instances/5.2-deploy/15.png)

Như vậy chúng ta đã tạo xong Application Load Balancer.

#### Tạo Service ECS

Bây giờ, hãy tạo một UI service mới để sử dụng Amazon ECS Managed Instances capacity provider.

Chú ý các biến như:

- **UI_TG_ARN**: ARN của Target Group vừa tạo.

- **PRIVATE_SUBNET1**: ID của Private Subnet thứ nhất chúng ta đã tạo trong bước trước.

- **PRIVATE_SUBNET2**: ID của Private Subnet thứ hai chúng ta đã tạo trong bước trước.

- **UI_SG_ID**: ID của Security Group `ui-sg`.

```bash
aws ecs create-service \
    --cluster retail-store-ecs-cluster \
    --service-name ui-managed-instance \
    --desired-count 2 \
    --task-definition retail-store-ecs-ui-managed \
    --load-balancers targetGroupArn=arn:aws:elasticloadbalancing:ap-northeast-2:${ACCOUNT_ID}:targetgroup/ui-managed-tg/57dad0b464a5ef76,containerName=application,containerPort=8080 \
    --capacity-provider-strategy capacityProvider=retail-store-managed-instances-cp,weight=1 \
    --network-configuration "awsvpcConfiguration={subnets=[${PRIVATE_SUBNET1}, ${PRIVATE_SUBNET2}], securityGroups=[${UI_SG_ID}],assignPublicIp=DISABLED}"
```

![Tạo Service ECS](/images/5-managed-instances/5.2-deploy/16.png)

Kiểm tra thông tin Service trên ECS Console:

![Tạo Service ECS](/images/5-managed-instances/5.2-deploy/17.png)

Chạy lệnh sau để xem lại cấu hình của các task đang chạy. Sẽ thấy CapacityProviderName bây giờ là `retail-store-managed-instances-cp` thay vì FARGATE:

```bash
aws ecs describe-tasks \
    --cluster retail-store-ecs-cluster \
    --tasks $(aws ecs list-tasks --cluster retail-store-ecs-cluster --service  ui-managed-instance --query 'taskArns[]' --output text) \
    --query "tasks[*].{Group:group, CapacityProviderName:capacityProviderName, LastStatus:lastStatus, HealthStatus:healthStatus, TaskArn:taskArn}" --output table
```

![Tạo Service ECS](/images/5-managed-instances/5.2-deploy/18.png)

Có thể lấy load balancer URL trên AWS Console như sau:

![Tạo Service ECS](/images/5-managed-instances/5.2-deploy/19.png)

Truy cập vào load balancer URL để kiểm tra thì có thể thấy website đã được triển khai thành công.

![Tạo Service ECS](/images/5-managed-instances/5.2-deploy/20.png)

