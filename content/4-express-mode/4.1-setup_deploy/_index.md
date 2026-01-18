---
title : "Express Mode Setup & Deployment"
date :  2025-01-01
weight : 1
chapter : false
pre : " <b> 4.1. </b> "  
---

Before we can deploy applications using Express Mode, we need an Infrastructure Role named `ecsExpressInfrastructureRole`.

This IAM role allows Amazon ECS Express Mode to manage infrastructure components on your behalf, including:

- **Load balancers**

- **Auto-scaling**

- **Networking configuration**

#### Create Infrastructure Role

First we need to create a trust policy for this role with the json file content below:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Service": "ecs.amazonaws.com"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}
```

Proceed to run the following command to create Role:

```bash
aws iam create-role \
  --role-name ecsExpressInfrastructureRole \
  --assume-role-policy-document file://ecs-express-trust.json
```

![Create Infrastructure Role](/images/4-express-mode/4.1-setup_deploy/1.png)

Next proceed to Attach AWS-managed policy to the role just created:

```bash
aws iam attach-role-policy \
  --role-name ecsExpressInfrastructureRole \
  --policy-arn arn:aws:iam::aws:policy/service-role/AmazonECSInfrastructureRoleforExpressGatewayServices
```

![Attach AWS-managed policy to the role just created](/images/4-express-mode/4.1-setup_deploy/2.png)

Check Infrastructure Role Managed Policy

```bash
aws iam list-attached-role-policies \
  --role-name ecsExpressInfrastructureRole
```

![Check Infrastructure Role Managed Policy](/images/4-express-mode/4.1-setup_deploy/3.png)

We have successfully created Infrastructure Role and Attached AWS-managed policy to the role just created.

#### Express Mode Deployment

We will deploy UI service similar to the Fundamentals section, but using Amazon ECS Express Mode.

Express Mode Service only needs three components to start:

- **Container image**
- **Task execution role**
- **Infrastructure role**

To run the sample application, we will use some custom parameters described in the table below:

| Parameter                   | Required | Description                                                            |
| ------------------------- | -------- | ---------------------------------------------------------------- |
| `execution-role-arn`      | Yes       | IAM role used for task execution                                 |
| `infrastructure-role-arn` | Yes       | IAM role used to manage infrastructure                                 |
| `primary-container`       | Yes       | Identifies primary container and port                                 |
| `cluster`, `service-name` | No    | ECS cluster name and Express service name                           |
| `cpu`, `memory`           | No    | Resources allocated for service                                  |
| `health-check-path`       | No    | Endpoint used for health check                                   |
| `network-configuration`   | No    | Identifies VPC subnets and security groups for task                 |
| `scaling-target`          | No    | Identifies how service automatically scales number of tasks based on demand   |
| `monitor-resources`       | No    | Enable resource monitoring during create, update and delete |

Run the command below to deploy Service `ui-express-mode` on the existing ECS cluster `retail-store-ecs-cluster`.

```bash
aws ecs create-express-gateway-service \
    --execution-role-arn arn:aws:iam::${ACCOUNT_ID}:role/retailStoreEcsTaskExecutionRole \
    --infrastructure-role-arn arn:aws:iam::${ACCOUNT_ID}:role/ecsExpressInfrastructureRole \
    --service-name ui-express-mode \
    --cpu "1024" \
    --memory "2048" \
    --health-check-path "/actuator/health" \
    --cluster retail-store-ecs-cluster \
    --network-configuration '{
      "subnets": ["'$PUBLIC_SUBNET1'","'$PUBLIC_SUBNET2'"]
    }' \
    --scaling-target '{
        "minTaskCount": 2,
        "maxTaskCount": 5,
        "autoScalingMetric": "REQUEST_COUNT_PER_TARGET",
        "autoScalingTargetValue": 50
    }' \
    --primary-container '{
        "image": "public.ecr.aws/aws-containers/retail-store-sample-ui:1.2.3",
        "containerPort": 8080
    }' --monitor-resources
```

After running the above command, ECS Express Mode will automatically:

- Create task definition

- Configure load balancer

- Set up service

- Set up auto-scaling

- Configure necessary networking components

The `--monitor-resources` option will display an overview of all resources being created as shown in the image below:

![--monitor-resources option](/images/4-express-mode/4.1-setup_deploy/4.png)

To close monitoring, you can press `q`.

While ECS Express Mode is creating resources, access Amazon ECS Console to view:

- Active services

- Running tasks

![Express Mode Service ui-express-mode has been deployed successfully](/images/4-express-mode/4.1-setup_deploy/5.png)

![List of Tasks and resources after being created successfully](/images/4-express-mode/4.1-setup_deploy/6.png)

Tasks and resources after being created successfully.

![Resources after being created successfully](/images/4-express-mode/4.1-setup_deploy/8.png)

![Resources displayed on --monitor-resources](/images/4-express-mode/4.1-setup_deploy/7.png)


Proceed to get Application Load Balancer DNS and access the application.

```bash
ECS_ALB_DNS=$(aws ecs describe-express-gateway-service \
  --service-arn arn:aws:ecs:${AWS_REGION}:${ACCOUNT_ID}:service/retail-store-ecs-cluster/ui-express-mode \
  --query service.activeConfigurations[0].ingressPaths[0].endpoint \
  --output text)

echo "https://${ECS_ALB_DNS}"
```

![Monitor application in Observability and Logs tabs](/images/4-express-mode/4.1-setup_deploy/9.png)

Proceed to access the application using DNS:

![Resources tab provides overview of all AWS resources automatically provisioned by Express Mode](/images/4-express-mode/4.1-setup_deploy/11.png)

#### Express Mode Review

**Service Overview** displays service overview, you can:

- Track service status

- View running tasks

- Monitor deployment progress

- Note the Application URL displayed on this page — you can use that URL to access the deployed application.

![View application logs in Logs section](/images/4-express-mode/4.1-setup_deploy/10.png)

To monitor the application, you can view metrics of service `ui-express-mode` in the tabs:

- Observability

- Logs

In the Logs section, you can view application logs, filter specific messages, and track application behavior in real-time.

![Check Express Mode configuration for the latest revision of service ui-express-mode](/images/4-express-mode/4.1-setup_deploy/12.png)

Resources tab provides an overview of all AWS resources automatically provisioned by Express Mode.

This list includes all infrastructure components created and managed for you, such as:

- Security Groups

- Load Balancers

- Target Groups

![Check Express Mode configuration for the latest revision of service ui-express-mode](/images/4-express-mode/4.1-setup_deploy/13.png)

Check Express Mode configuration for the latest revision of service ui-express-mode with the following command:

```bash
SERVICE_REVISION_ARN=$(aws ecs list-service-deployments \
  --service arn:aws:ecs:${AWS_REGION}:${ACCOUNT_ID}:service/retail-store-ecs-cluster/ui-express-mode \
  --max-results 1 \
  --query serviceDeployments[0].targetServiceRevisionArn \
  --output text)
```

```bash
aws ecs describe-service-revisions \
  --service-revision-arn $SERVICE_REVISION_ARN
```

![Check Express Mode configuration for the latest revision of service ui-express-mode](/images/4-express-mode/4.1-setup_deploy/16.png)

During service creation, Express Mode automatically created Task Definition for the application. You can list Task Definition revisions with the following command:

```bash
aws ecs list-task-definitions --family-prefix retail-store-ecs-cluster-ui-express-mode --sort DESC --max-items 2
```

![List Task Definition revisions](/images/4-express-mode/4.1-setup_deploy/17.png)

Check Load Balancer by opening Amazon Load Balancer Console and selecting the load balancer named `ecs-express-gateway-alb-XXXXXXXX` to view configuration details.

![Check Load Balancer](/images/4-express-mode/4.1-setup_deploy/15.png)

You can see:
- HTTPS listener
- Valid certificate

Have been automatically configured by ECS Express Mode.

