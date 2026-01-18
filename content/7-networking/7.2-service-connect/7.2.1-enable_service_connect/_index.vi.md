---
title : "Enable Service Connect"
date :  2025-01-01
weight : 1
chapter : false
pre : " <b> 7.2.1. </b> "  
---

Trong phần này, chúng ta sẽ kích hoạt ECS Service Connect trong cluster bằng cách triển khai ba microservices bổ sung mà UI service sẽ giao tiếp cùng:

![AWS ECS Service Connect Architecture](/images/7-networking/7.2-service-connect/7.2.1-enable_service_connect/1.png)

#### Tạo Security Group Cho Các Service

Tạo Security Group cho các service như sau:

- VPC ID được lấy từ VPC mà chúng ta đã tạo trong phần trước.

- **Orders Service**: `orders-sg`

```bash
aws ec2 create-security-group \
  --group-name orders-sg \
  --description "Orders ECS Service SG" \
  --vpc-id ${VPC_ID}
```

- **Checkout Service**: `checkout-sg`

```bash
aws ec2 create-security-group \
  --group-name checkout-sg \
  --description "Checkout ECS Service SG" \
  --vpc-id ${VPC_ID}
```

- **Catalog Service**: `catalog-sg`

```bash
aws ec2 create-security-group \
  --group-name catalog-sg \
  --description "Catalog ECS Service SG" \
  --vpc-id ${VPC_ID}
```

![Security Group Created](/images/7-networking/7.2-service-connect/7.2.1-enable_service_connect/2.png)

Lấy các ID trên và cấu hình Inbound rules cho các Security Group như sau:

- Orders Service nhận traffic từ UI Service và Checkout Service.

```bash
aws ec2 authorize-security-group-ingress \
  --group-id ${ORDERS_SG_ID} \
  --protocol tcp \
  --port 8080 \
  --source-group ${UI_SG_ID}

aws ec2 authorize-security-group-ingress \
  --group-id ${ORDERS_SG_ID} \
  --protocol tcp \
  --port 8080 \
  --source-group ${CHECKOUT_SG_ID}
```

![Orders Service Inbound Rules](/images/7-networking/7.2-service-connect/7.2.1-enable_service_connect/3.png)

- Checkout Service nhận traffic từ UI Service.

```bash
aws ec2 authorize-security-group-ingress \
  --group-id ${CHECKOUT_SG_ID} \
  --protocol tcp \
  --port 8080 \
  --source-group ${UI_SG_ID}
```

![Checkout Service Inbound Rules](/images/7-networking/7.2-service-connect/7.2.1-enable_service_connect/4.png)

- Catalog Service nhận traffic từ UI Service.

```bash
aws ec2 authorize-security-group-ingress \
  --group-id ${CATALOG_SG_ID} \
  --protocol tcp \
  --port 8080 \
  --source-group ${UI_SG_ID}
```

![Catalog Service Inbound Rules](/images/7-networking/7.2-service-connect/7.2.1-enable_service_connect/5.png)

#### Deploy Orders Service

Tạo ECS task definition cho Orders service. Chúng ta sẽ làm tương tự với UI Service bằng cách sử dụng file json `retail-store-ecs-order-taskdef.json` với nội dung như sau:

```json
{
    "family": "retail-store-ecs-orders",
    "executionRoleArn": "arn:aws:iam::${ACCOUNT_ID}:role/ordersEcsTaskExecutionRole",
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
            "image": "public.ecr.aws/aws-containers/retail-store-sample-orders:1.2.3",
            "cpu": 0,
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
            "environment": [
                {
                    "name": "RETAIL_ORDERS_PERSISTENCE_PROVIDER",
                    "value": "postgres"
                },
                {
                    "name": "RETAIL_ORDERS_MESSAGING_PROVIDER",
                    "value": "in-memory"
                }
            ],
            "secrets": [
                {
                    "name": "RETAIL_ORDERS_PERSISTENCE_ENDPOINT",
                    "valueFrom": "arn:aws:ssm:${AWS_REGION}:${ACCOUNT_ID}:parameter/retail-store-ecs/orders/db-endpoint-postgres"
                },
                {
                    "name": "RETAIL_ORDERS_PERSISTENCE_NAME",
                    "valueFrom": "arn:aws:secretsmanager:${AWS_REGION}:${ACCOUNT_ID}:secret:retail-store-ecs-orders-db:dbname::"
                },
                {
                    "name": "RETAIL_ORDERS_PERSISTENCE_USERNAME",
                    "valueFrom": "arn:aws:secretsmanager:${AWS_REGION}:${ACCOUNT_ID}:secret:retail-store-ecs-orders-db:username::"
                },
                {
                    "name": "RETAIL_ORDERS_PERSISTENCE_PASSWORD",
                    "valueFrom": "arn:aws:secretsmanager:${AWS_REGION}:${ACCOUNT_ID}:secret:retail-store-ecs-orders-db:password::"
                }
            ],
            "logConfiguration": {
                "logDriver": "awslogs",
                "options": {
                    "awslogs-group": "retail-store-ecs-tasks",
                    "awslogs-region": "$AWS_REGION",
                    "awslogs-stream-prefix": "orders-service"
                }
            }
        }
    ]
}
```

Đăng ký task definition:

```bash
aws ecs register-task-definition \
    --cli-input-json file://retail-store-ecs-order-taskdef.json
```

![Order Task Definition](/images/7-networking/7.2-service-connect/7.2.1-enable_service_connect/6.png)

Dùng lệnh sau để tạo Orders service:

```bash
aws ecs create-service \
    --cluster retail-store-ecs-cluster \
    --service-name orders \
    --task-definition retail-store-ecs-orders \
    --desired-count 1 \
    --launch-type FARGATE \
    --enable-execute-command \
    --network-configuration "awsvpcConfiguration={subnets=[subnet-0b742ee8bbf495219, subnet-0317d1d968b9963d0], securityGroups=[sg-069f1465934b99e2a],assignPublicIp=DISABLED}" \
    --service-connect-configuration '{
        "enabled": true,
        "namespace": "retailstore.local",
        "services": [
            {
                "portName": "application",
                "discoveryName": "orders",
                "clientAliases": [
                    {
                        "port": 80,
                        "dnsName": "orders"
                    }
                ]
            }
        ]
    }'
```

Lưu ý: khi tạo service, ta chỉ định `--service-connect-configuration`, thao tác này sẽ:

- Kích hoạt Service Connect.

- Chỉ định namespace dùng chung cho tất cả services.

- Cấu hình Service Connect services được cung cấp bởi ECS service này, bao gồm alias và port.

[Tham khảo thêm trong AWS Documentation](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/networking-connecting-services.html#networking-connecting-services-serviceconnect).

#### Deploy Checkout Service

Tạo ECS task definition cho Checkout service. Tương tự như trên chúng ta sẽ tạo task definition với file json `retail-store-ecs-checkout-taskdef.json` với nội dung như sau:

```json
{
    "family": "retail-store-ecs-checkout",
    "executionRoleArn": "arn:aws:iam::${ACCOUNT_ID}:role/checkoutEcsTaskExecutionRole",
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
            "image": "public.ecr.aws/aws-containers/retail-store-sample-checkout:1.2.3",
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
                    "name": "RETAIL_CHECKOUT_PERSISTENCE_PROVIDER",
                    "value": "redis"
                },
                 {
                    "name": "RETAIL_CHECKOUT_ENDPOINTS_ORDERS",
                    "value": "http://orders"
                }
            ],
            "secrets": [
                {
                    "name": "RETAIL_CHECKOUT_PERSISTENCE_REDIS_URL",
                    "valueFrom": "arn:aws:ssm:${AWS_REGION}:${ACCOUNT_ID}:parameter/retail-store-ecs/checkout/redis-endpoint"
                }
            ],
            "healthCheck": {
                "command": [
                    "CMD-SHELL",
                    "curl -f http://localhost:8080/health || exit 1"
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
                    "awslogs-stream-prefix": "checkout-service"
                }
            }
        }
    ]
}
```

Đăng ký task definition:

```bash
aws ecs register-task-definition \
    --cli-input-json file://retail-store-ecs-checkout-taskdef.json
```

Dùng lệnh sau để tạo Checkout Service:

```bash
aws ecs create-service \
    --cluster retail-store-ecs-cluster \
    --service-name checkout \
    --task-definition retail-store-ecs-checkout \
    --desired-count 1 \
    --launch-type FARGATE \
    --enable-execute-command \
    --network-configuration "awsvpcConfiguration={subnets=[subnet-0b742ee8bbf495219, subnet-0317d1d968b9963d0], securityGroups=[sg-037956d78913fda63],assignPublicIp=DISABLED}" \
    --service-connect-configuration '{
        "enabled": true,
        "namespace": "retailstore.local",
        "services": [
            {
                "portName": "application",
                "discoveryName": "checkout",
                "clientAliases": [
                    {
                        "port": 80,
                        "dnsName": "checkout"
                    }
                ]
            }
        ]
    }'
```

#### Deploy Catalog Service

Tạo Task Definition cho Catalog Service. Tạo task definition với file json `retail-store-ecs-catalog-taskdef.json` với nội dung như sau:

```json
{
    "family": "retail-store-ecs-catalog",
    "executionRoleArn": "arn:aws:iam::${ACCOUNT_ID}:role/catalogEcsTaskExecutionRole",
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
            "image": "public.ecr.aws/aws-containers/retail-store-sample-catalog:1.2.3",
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
                    "name": "RETAIL_CATALOG_PERSISTENCE_DB_NAME",
                    "value": "catalog"
                },
                {
                    "name": "RETAIL_CATALOG_PERSISTENCE_PROVIDER",
                    "value": "mysql"
                }
            ],
            "secrets": [
                {
                    "name": "RETAIL_CATALOG_PERSISTENCE_ENDPOINT",
                    "valueFrom": "arn:aws:ssm:${AWS_REGION}:${ACCOUNT_ID}:parameter/retail-store-ecs/catalog/db-endpoint-mysql"
                },
                {
                    "name": "RETAIL_CATALOG_PERSISTENCE_PASSWORD",
                    "valueFrom": "arn:aws:secretsmanager:${AWS_REGION}:${ACCOUNT_ID}:secret:retail-store-ecs-catalog-db:password::"
                },
                {
                    "name": "RETAIL_CATALOG_PERSISTENCE_USER",
                    "valueFrom": "arn:aws:secretsmanager:${AWS_REGION}:${ACCOUNT_ID}:secret:retail-store-ecs-catalog-db:username::"
                }
            ],
            "logConfiguration": {
                "logDriver": "awslogs",
                "options": {
                    "awslogs-group": "retail-store-ecs-tasks",
                    "awslogs-region": "${AWS_REGION}",
                    "awslogs-stream-prefix": "catalog-service"
                }
            },
            "healthCheck": {
                "command": [
                    "CMD-SHELL",
                    "curl -f http://localhost:8080/health || exit 1"
                ],
                "interval": 10,
                "timeout": 5,
                "retries": 3,
                "startPeriod": 60
            }
        }
    ]
}
```

Đăng ký task definition:

```bash
aws ecs register-task-definition \
    --cli-input-json file://retail-store-ecs-catalog-taskdef.json
```

Dùng lệnh sau để tạo Catalog Service:

```bash
aws ecs create-service \
    --cluster retail-store-ecs-cluster \
    --service-name catalog \
    --task-definition retail-store-ecs-catalog \
    --desired-count 2 \
    --launch-type FARGATE \
    --enable-execute-command \
    --network-configuration "awsvpcConfiguration={subnets=[${PRIVATE_SUBNET1}, ${PRIVATE_SUBNET2}], securityGroups=[$CATALOG_SG_ID],assignPublicIp=DISABLED}" \
    --service-connect-configuration '{
        "enabled": true,
        "namespace": "retailstore.local",
        "services": [
            {
                "portName": "application",
                "discoveryName": "catalog",
                "clientAliases": [
                    {
                        "port": 80,
                        "dnsName": "catalog"
                    }
                ]
            }
        ]
    }'
```

Chờ các Service được tạo xong, tiếp theo, cập nhật cấu hình UI Service để nó có thể tương tác với các microservices vừa triển khai. Các environment variables sau cần được thêm vào UI Task Definition:

```json
"environment": [
    {
        "name": "RETAIL_UI_ENDPOINTS_ORDERS",
        "value": "http://orders"
    },
    {
        "name": "RETAIL_UI_ENDPOINTS_CHECKOUT",
        "value": "http://checkout"
    },
    {
        "name": "RETAIL_UI_ENDPOINTS_CATALOG",
        "value": "http://catalog"
    }
]
```

Chạy script sau để in ra UI Task Definition:

```bash
cat retail-store-ecs-ui-connect-taskdef.json | jq
```

Đăng ký Task Definition mới cho UI:

```bash
aws ecs register-task-definition --cli-input-json file://retail-store-ecs-ui-connect-taskdef.json
```

Cập nhật UI Service với Task Definition mới và Service Connect configuration. Đồng thời bật Service Connect access logs để thu thập telemetry chi tiết cho từng request.

```bash
aws ecs update-service \
    --cluster retail-store-ecs-cluster \
    --service ui \
    --task-definition retail-store-ecs-ui \
    --force-new-deployment \
    --desired-count 2 \
    --service-connect-configuration '{
        "enabled": true,
        "namespace": "retailstore.local",
        "logConfiguration": {
            "logDriver": "awslogs",
            "options": {
                "awslogs-group": "'$SERVICE_CONNECT_LOG_GROUP_NAME'",
                "awslogs-region": "'$AWS_REGION'",
                "awslogs-stream-prefix": "ui-service-connect-logs"
            }
        },
        "accessLogConfiguration": {
            "format": "JSON",
            "includeQueryParameters": "ENABLED"
        },
        "services": [
            {
                "portName": "application",
                "discoveryName": "ui",
                "clientAliases": [
                    {
                        "port": 80,
                        "dnsName": "ui"
                    }
                ]
            }
        ]
    }'
```

