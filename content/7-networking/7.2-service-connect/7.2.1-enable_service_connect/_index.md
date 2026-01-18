---
title : "Enable Service Connect"
date :  2025-01-01
weight : 1
chapter : false
pre : " <b> 7.2.1. </b> "  
---

In this section, we will enable ECS Service Connect in the cluster by deploying three additional microservices that UI service will communicate with:

![AWS ECS Service Connect Architecture](/images/7-networking/7.2-service-connect/7.2.1-enable_service_connect/1.png)

#### Create Security Groups for Services

Create Security Groups for services as follows:

- VPC ID is obtained from the VPC we created in the previous section.

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

Get the IDs above and configure Inbound rules for Security Groups as follows:

- Orders Service receives traffic from UI Service and Checkout Service.

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

- Checkout Service receives traffic from UI Service.

```bash
aws ec2 authorize-security-group-ingress \
  --group-id ${CHECKOUT_SG_ID} \
  --protocol tcp \
  --port 8080 \
  --source-group ${UI_SG_ID}
```

![Checkout Service Inbound Rules](/images/7-networking/7.2-service-connect/7.2.1-enable_service_connect/4.png)

- Catalog Service receives traffic from UI Service.

```bash
aws ec2 authorize-security-group-ingress \
  --group-id ${CATALOG_SG_ID} \
  --protocol tcp \
  --port 8080 \
  --source-group ${UI_SG_ID}
```

![Catalog Service Inbound Rules](/images/7-networking/7.2-service-connect/7.2.1-enable_service_connect/5.png)

#### Deploy Orders Service

Create ECS task definition for Orders service. We will do similarly to UI Service by using json file `retail-store-ecs-order-taskdef.json` with the following content:

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

Register task definition:

```bash
aws ecs register-task-definition \
    --cli-input-json file://retail-store-ecs-order-taskdef.json
```

![Order Task Definition](/images/7-networking/7.2-service-connect/7.2.1-enable_service_connect/6.png)

Use the following command to create Orders service:

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

Note: when creating service, we specify `--service-connect-configuration`, this will:

- Enable Service Connect.

- Specify shared namespace for all services.

- Configure Service Connect services provided by this ECS service, including alias and port.

[Refer to AWS Documentation for more information](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/networking-connecting-services.html#networking-connecting-services-serviceconnect).

#### Deploy Checkout Service

Create ECS task definition for Checkout service. Similarly as above we will create task definition with json file `retail-store-ecs-checkout-taskdef.json` with the following content:

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

Register task definition:

```bash
aws ecs register-task-definition \
    --cli-input-json file://retail-store-ecs-checkout-taskdef.json
```

Use the following command to create Checkout Service:

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

Create Task Definition for Catalog Service. Create task definition with json file `retail-store-ecs-catalog-taskdef.json` with the following content:

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

Register task definition:

```bash
aws ecs register-task-definition \
    --cli-input-json file://retail-store-ecs-catalog-taskdef.json
```

Use the following command to create Catalog Service:

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

Wait for Services to be created, then update UI Service configuration so it can interact with the microservices just deployed. The following environment variables need to be added to UI Task Definition:

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

Run the following script to print UI Task Definition:

```bash
cat retail-store-ecs-ui-connect-taskdef.json | jq
```

Register new Task Definition for UI:

```bash
aws ecs register-task-definition --cli-input-json file://retail-store-ecs-ui-connect-taskdef.json
```

Update UI Service with new Task Definition and Service Connect configuration. Also enable Service Connect access logs to collect detailed telemetry for each request.

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

