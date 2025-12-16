---
title : "Services"
date :  2025-01-01
weight : 3
chapter : false
pre : " <b> 3.3. </b> "  
---

#### Create VPC

Access VPC on AWS Console:

![Create VPC](/images/3-prepare/3.3-services/1.png)

Select **Create VPC**.

![Create VPC](/images/3-prepare/3.3-services/2.png)

Select **VPC and more** and configure the following parameters:

- **VPC Name**: `ecs-workshop`
- **VPC CIDR Block**: `10.0.0.0/16`
- **Number of Availability Zones**: `2`
- **Number of Public Subnets**: `2`
- **Number of Private Subnets**: `2`
- **NAT Gateway**: `None` (NAT will be created manually later)
- **VPC Endpoint**: `None`

After configuration is complete, select **Create VPC**.

![Create VPC](/images/3-prepare/3.3-services/3.png)

![Create VPC](/images/3-prepare/3.3-services/4.png)

The VPC has been created successfully.

![Create VPC](/images/3-prepare/3.3-services/5.png)

![Create VPC](/images/3-prepare/3.3-services/6.png)

#### Create NAT Gateway

On the VPC Console interface, select **NAT Gateways** and select **Create NAT Gateway**.

![Create NAT Gateway](/images/3-prepare/3.3-services/7.png)

Configure the following parameters:

- **NAT Gateway Name**: `ecs-workshop-nat`
- **Availability mode**: `Regional`
- **VPC**: The VPC you just created.
- **Method of Elastic IP (EIP) allocation**: `Automatic`

After configuration is complete, select **Create NAT Gateway**

![Create NAT Gateway](/images/3-prepare/3.3-services/8.png)

Create an **Elastic IP** on the VPC Console interface.

- Select **Allocate Elastic IP address**

![Create NAT Gateway](/images/3-prepare/3.3-services/9.png)

Select **Allocate**

![Create NAT Gateway](/images/3-prepare/3.3-services/10.png)

The **Elastic IP** has been created successfully.

![Create NAT Gateway](/images/3-prepare/3.3-services/11.png)

The NAT Gateway will automatically use the created **Elastic IP**.

![Create NAT Gateway](/images/3-prepare/3.3-services/12.png)

After creating the NAT Gateway, configure the Route Table for the Private Subnet.

Access **Route Table**:

- Select **Private Subnet**.
- Select **Routes**
- Select **Edit Routes**

![Create Route Table](/images/3-prepare/3.3-services/37.png)

Configure the following parameters:

- **Destination**: `0.0.0.0/0`
- **Target**: `NAT Gateway` and select the ID of the NAT Gateway you just created.

![Create Route Table](/images/3-prepare/3.3-services/38.png)

The Route Table has been configured successfully.

![Create Route Table](/images/3-prepare/3.3-services/39.png)

Do the same for the second Private Subnet.

![Create Route Table](/images/3-prepare/3.3-services/40.png)

#### Create Security Group

Access the EC2 Console:

![Create Security Group](/images/3-prepare/3.3-services/13.png)

Select **Security Groups** and select **Create Security Group**.

![Create Security Group](/images/3-prepare/3.3-services/14.png)

We will create `alb-sg` to allow access to the ALB.

- **Security Group Name**: `alb-sg`
- **Description**: `Security Group for ALB`
- **VPC**: The VPC you just created.

Inbound rules:

- **Type**: `HTTP`
- **Port**: `80`
- **Source**: `0.0.0.0/0`
- **Type**: `HTTPS`
- **Port**: `443`
- **Source**: `0.0.0.0/0`

![Create Security Group](/images/3-prepare/3.3-services/15.png)

Outbound rules:

- **Type**: `All traffic`
- **Source**: `0.0.0.0/0`

After configuration is complete, select **Create Security Group**

![Create Security Group](/images/3-prepare/3.3-services/16.png)

After creating the Security Group for the ALB, create a Security Group for the application.

![Create Security Group](/images/3-prepare/3.3-services/17.png)

- **Security Group Name**: `ui-sg`
- **Description**: `Security Group for UI`
- **VPC**: The VPC you just created.

Inbound rules:

- **Type**: `Custom TCP Rule`
- **Port**: `8080`
- **Source**: `alb-sg`

![Create Security Group](/images/3-prepare/3.3-services/18.png)

After configuration is complete, select **Create Security Group**

![Create Security Group](/images/3-prepare/3.3-services/19.png)

The Security Group has been created successfully.

![Create Security Group](/images/3-prepare/3.3-services/20.png)

#### Create Target Group

Access the EC2 Console:

![Create Target Group](/images/3-prepare/3.3-services/21.png)

Select **Target Groups** and select **Create Target Group**.

![Create Target Group](/images/3-prepare/3.3-services/22.png)

Configure the following parameters:

- **Target Group Name**: `ui-tg`
- **Target Type**: `IP Addresses`
- **Protocol**: `HTTP`
- **Port**: `8080`
- **VPC**: The VPC you just created.
- **Health Check Protocol**: `HTTP`
- **Health Check Path**: `/actuator/health`

![Create Target Group](/images/3-prepare/3.3-services/23.png)

![Create Target Group](/images/3-prepare/3.3-services/24.png)

After configuration is complete, select **Next**.

![Create Target Group](/images/3-prepare/3.3-services/25.png)

![Create Target Group](/images/3-prepare/3.3-services/26.png)

Select **Next**.

![Create Target Group](/images/3-prepare/3.3-services/27.png)

Review the parameters and select **Create Target Group**.

![Create Target Group](/images/3-prepare/3.3-services/28.png)

The Target Group has been created successfully.

![Create Target Group](/images/3-prepare/3.3-services/29.png)

#### Create Application Load Balancer

On the EC2 Console interface, select **Load Balancers** and select **Create Load Balancer**.

![Create Application Load Balancer](/images/3-prepare/3.3-services/30.png)

Select Type as **Application Load Balancer**.

![Create Application Load Balancer](/images/3-prepare/3.3-services/31.png)

Configure the following parameters:

- **Load Balancer Name**: `ecs-workshop-alb`
- **Scheme**: `Internet-facing`

![Create Application Load Balancer](/images/3-prepare/3.3-services/32.png)

- **VPC**: The VPC you just created.
- Configure 2 Public Subnets for the ALB.

![Create Application Load Balancer](/images/3-prepare/3.3-services/33.png)

- Select the `alb-sg` you just created.

![Create Application Load Balancer](/images/3-prepare/3.3-services/34.png)

Review and select **Create Load Balancer**.

![Create Application Load Balancer](/images/3-prepare/3.3-services/35.png)

![Create Application Load Balancer](/images/3-prepare/3.3-services/36.png)

#### Create ECS Service

After creating the above components, create the ECS Service.

Use the following CLI to create the ECS Service.

Note the following parameters:

- **UI_TG_ARN**: ARN of the application's Target Group.
- **PRIVATE_SUBNET1**: ID of the first Private Subnet.
- **PRIVATE_SUBNET2**: ID of the second Private Subnet.
- **UI_SG_ID**: ID of the application's Security Group.


```bash
aws ecs create-service \
    --cluster retail-store-ecs-cluster \
    --service-name ui \
    --task-definition retail-store-ecs-ui \
    --desired-count 2 \
    --launch-type FARGATE \
    --load-balancers targetGroupArn=${UI_TG_ARN},containerName=application,containerPort=8080 \
    --network-configuration "awsvpcConfiguration={subnets=[${PRIVATE_SUBNET1},${PRIVATE_SUBNET2}],securityGroups=[${UI_SG_ID}],assignPublicIp=DISABLED}"
```

Access the Cluster to check and you can see the Service has been created successfully.

![Create ECS Service](/images/3-prepare/3.3-services/41.png)

Check Service information:

![Create ECS Service](/images/3-prepare/3.3-services/42.png)

Check Task information:

![Create ECS Service](/images/3-prepare/3.3-services/43.png)

We can see that 2 Tasks have been created successfully.

![Create ECS Service](/images/3-prepare/3.3-services/44.png)

Access the web using the ALB URL to check the website.

`http://ecs-workshop-alb-653740503.ap-northeast-2.elb.amazonaws.com`

![Create ECS Service](/images/3-prepare/3.3-services/45.png)

The application has been deployed successfully.

After creating the basic components, the current architecture of the configured Services is shown below.

![Create ECS Service](/images/3-prepare/3.3-services/46.png)

