---
title : "Update Services"
date :  2025-01-01
weight : 4
chapter : false
pre : " <b> 3.4. </b> "  
---

In this section we will update an ECS Service. This process is commonly used in situations such as changing container image or adjusting application configuration.

**Environment variables** are one of the main mechanisms to configure workloads running in containers, regardless of which Orchestrator is used. We will proceed to change the configuration of UI Service to use new environment variables.

**Docker Image** cannot be changed after being built, so environment variables are a simple and flexible way to configure and adjust application behavior when the container is running.

In this case, we will use the **RETAIL_UI_THEME** variable, which will change the default interface color of the application.

#### Declare variables in Task Definition

Environment variables in ECS task definition are declared as name-value pairs, for example:

```json
"environment": [
    {
        "name": "RETAIL_UI_THEME",
        "value": "green"
    }
]
```

Next, proceed to update the json file `retail-store-ecs-ui-updated-taskdef.json` with the new environment variable.

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

Then, use the **register-task-definition** command to register the new task definition:

```bash
aws ecs register-task-definition \
  --cli-input-json file://retail-store-ecs-ui-updated-taskdef.json
```

![Update Task Definition](/images/3-prepare/3.4-updated_services/1.png)


{{% notice note %}}
ECS task definition is immutable, meaning it cannot be edited after creation.
Instead, the above command will create a new revision, which is a copy of the old task definition but with updated new parameters.
{{% /notice %}}

#### Check Task Definition revisions

You can check how many revisions currently exist with the command:

```bash
aws ecs list-task-definitions --family-prefix retail-store-ecs-ui --sort DESC --max-items 2
```

The result will display the revisions of the UI task definition:

```bash
{
    "taskDefinitionArns": [
        "arn:aws:ecs:us-west-2:XXXXXXXXXXXX:task-definition/retail-store-ecs-ui:2",
        "arn:aws:ecs:us-west-2:XXXXXXXXXXXX:task-definition/retail-store-ecs-ui:1"
    ]
}
```

![Check Task Definition revisions](/images/3-prepare/3.4-updated_services/3.png)

Or you can check directly on AWS Console:

![Update Task Definition](/images/3-prepare/3.4-updated_services/2.png)

#### Update ECS Service to use new revision

Next, update ECS service to use the new task definition:

```bash
aws ecs update-service --cluster retail-store-ecs-cluster --service ui --task-definition retail-store-ecs-ui
```

![Update ECS Service to use new revision](/images/3-prepare/3.4-updated_services/4.png)

#### Check results

![Check results](/images/3-prepare/3.4-updated_services/5.png)

![Check results](/images/3-prepare/3.4-updated_services/6.png)
