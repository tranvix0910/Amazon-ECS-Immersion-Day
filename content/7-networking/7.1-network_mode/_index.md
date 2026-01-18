---
title : "Amazon ECS Network Mode"
date :  2025-01-01
weight : 1
chapter : false
pre : " <b> 7.1. </b> "  
---

When running containers, it is very important to consider the network configuration of containers running on the host. Please refer to [AWS Documentation](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/task-networking.html) for more information on how to choose the appropriate network mode. 

In this section, we will have an overview of **AWSVPC Mode** network configuration for Amazon ECS on Fargate.

In **AWSVPC Mode**, Amazon ECS will create and manage an Elastic Network Interface (ENI) for each task, and each task will receive a separate private IP address in the VPC. This configuration provides high flexibility, allowing detailed control (granular level) of communication between tasks and services. 

**AWSVPC Network Mode** is supported for Amazon ECS tasks running on both Amazon EC2 and Fargate. Refer to [AWS Documentation](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/task-networking-awsvpc.html) for more details.

{{% notice note %}}
When using Amazon ECS on Fargate, AWSVPC Network Mode is required.
{{% /notice %}}

![AWSVPC Network Mode](/images/7-networking/7.1-network_mode/1.png)

#### Review Network Mode

Open [Amazon ECS console](https://console.aws.amazon.com/ecs/home/clusters) to check services.

![Review Network Mode](/images/7-networking/7.1-network_mode/2.png)

Here, you can select the first running task. Scroll down to the Configuration section to review the following information:

- Network mode

- ENI ID

- Private IP attached to task

![Review Network Mode](/images/7-networking/7.1-network_mode/3.png)

You can get Task information using AWS CLI:

```bash
aws ecs describe-tasks \
 --cluster retail-store-ecs-cluster \
 --tasks $(aws ecs list-tasks --cluster retail-store-ecs-cluster --service ui --query 'taskArns[0]' --output text)
```

![Review Network Mode](/images/7-networking/7.1-network_mode/4.png)

Result:

```json
{
    "tasks": [
        {
            "attachments": [
                {
                    "id": "464044b3-626f-44da-86ec-fa20a064d408",
                    "type": "ElasticNetworkInterface",
                    "status": "ATTACHED",
                    "details": [
                        {
                            "name": "subnetId",
                            "value": "subnet-08c4050330714ed3d"
                        },
                        {
                            "name": "networkInterfaceId",
                            "value": "eni-0a6c55131166f85c8"
                        },
                        {
                            "name": "macAddress",
                            "value": "06:39:15:1c:ae:1f"
                        },
                        {
                            "name": "privateDnsName",
                            "value": "ip-10-0-4-128.us-west-2.compute.internal"
                        },
                        {
                            "name": "privateIPv4Address",
                            "value": "10.0.4.128"
                        }
                    ]
                }
            ],
            "attributes": [
                {
                    "name": "ecs.cpu-architecture",
                    "value": "x86_64"
                }
            ],
            "availabilityZone": "us-west-2b",
            "clusterArn": "arn:aws:ecs:us-west-2:XXXXXXXXXX:cluster/retail-store-ecs-cluster",
            "connectivity": "CONNECTED",
            "connectivityAt": "2024-04-10T08:09:33.968000+00:00",
            "containers": [
                {
                    "containerArn": "arn:aws:ecs:us-west-2:XXXXXXXXXX:container/retail-store-ecs-cluster/70137dd0c1d14cf982e5a6b7446c5f54/db0fa651-1727-4215-b5a5-8a5577120942",
                    "taskArn": "arn:aws:ecs:us-west-2:XXXXXXXXXX:task/retail-store-ecs-cluster/70137dd0c1d14cf982e5a6b7446c5f54",
                    "name": "application",
                    "image": "public.ecr.aws/aws-containers/retail-store-sample-ui:1.2.3",
                    "imageDigest": "sha256:6316a3c331c39c35798f3b1303f80494526c0a879fe5a3db3b0b9a85c22aab36",
                    "runtimeId": "70137dd0c1d14cf982e5a6b7446c5f54-524788293",
                    "lastStatus": "RUNNING",
                    "networkBindings": [],
                    "networkInterfaces": [
                        {
                            "attachmentId": "464044b3-626f-44da-86ec-fa20a064d408",
                            "privateIpv4Address": "10.0.4.128"
                        }
                    ],
                    "healthStatus": "HEALTHY",
                    "cpu": "0"
                }
            ],
            "cpu": "1024",
            "createdAt": "2024-04-10T08:09:29.943000+00:00",
            "desiredStatus": "RUNNING",
            "enableExecuteCommand": false,
            "group": "service:ui",
            "healthStatus": "HEALTHY",
            "lastStatus": "RUNNING",
            "launchType": "FARGATE",
            "memory": "2048",
            "overrides": {
                "containerOverrides": [
                    {
                        "name": "application"
                    }
                ],
                "inferenceAcceleratorOverrides": []
            },
            "platformVersion": "1.4.0",
            "platformFamily": "Linux",
            "pullStartedAt": "2024-04-10T08:09:44.535000+00:00",
            "pullStoppedAt": "2024-04-10T08:09:52.398000+00:00",
            "startedAt": "2024-04-10T08:10:44.148000+00:00",
            "startedBy": "ecs-svc/8962710467093341990",
            "tags": [],
            "taskArn": "arn:aws:ecs:us-west-2:XXXXXXXXXX:task/retail-store-ecs-cluster/70137dd0c1d14cf982e5a6b7446c5f54",
            "taskDefinitionArn": "arn:aws:ecs:us-west-2:XXXXXXXXXX:task-definition/retail-store-ecs-ui:8",
            "version": 4,
            "ephemeralStorage": {
                "sizeInGiB": 20
            }
        }
    ],
    "failures": []
}
```

