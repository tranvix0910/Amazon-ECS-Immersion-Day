---
title : "Enable ECS Managed Instances"
date :  2025-01-01
weight : 1
chapter : false
pre : " <b> 5.1. </b> "  
---

In this section, you will perform setup to deploy workloads on Amazon ECS Managed Instances.

Specifically, we will set up default **Amazon ECS capacity providers** for **ECS Managed Instances**.

Amazon ECS capacity providers are responsible for managing infrastructure scaling for tasks in ECS cluster.

Amazon ECS currently provides three types of capacity providers for cluster:

- **Fargate capacity providers**: These are pre-defined capacity providers for AWS Fargate. This is also the capacity provider we have been using from the beginning of the workshop.

- **Managed Instances capacity providers**: Allows using Amazon ECS Managed Instances to run workloads.
    - 👉 This is the capacity provider that will be configured in this section.

- **Auto Scaling group capacity providers**: Allows using Amazon EC2 instances to run workloads through Auto Scaling Group.

![ECS Capacity Providers](/images/5-managed-instances/5.1-enable/1.png)

**Instance Profile** is an IAM object attached directly to EC2 instance, serving as a bridge between EC2 and IAM Role.

Through Instance Profile, EC2 (and processes/daemons running on EC2 such as ECS Agent) can assume the corresponding IAM Role and use granted AWS permissions without needing to configure access key/secret key.

#### Create ECS Managed Instances Capacity Provider

Before creating capacity provider, we need two IAM roles (refer to [AWS Documentation](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/ManagedInstances.html#managed-instances-iam-roles) for more details):

- **Infrastructure Role**: Allows Amazon ECS to manage ECS Managed Instances on your behalf.

- **Instance profile**: Provides permissions for Amazon ECS container agent running on managed instances.

We will create these two IAM roles using AWS CLI. For **Infrastructure Role** we will proceed to create role with name `ecsManagedInstanceInfrastructureRole`. First we need to create trust policy for this role with the json file content below:

{{% notice note %}}
ECS Agent is a daemon (background process) on EC2 instance.
This agent uses Instance Profile attached to EC2 to assume the corresponding IAM Role, thereby being able to execute granted AWS permissions (e.g., call ECS API, pull image from ECR, send logs to CloudWatch).
{{% /notice %}}


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
  --role-name ecsManagedInstanceInfrastructureRole \
  --assume-role-policy-document file://ecs-managed-instance-infrastructure-role-trust.json
```

![Create Infrastructure Role](/images/5-managed-instances/5.1-enable/2.png)

Next we need to Attach AWS-managed policy to the role just created:

- Policy 1 - Manage Managed Instances:

```bash
aws iam attach-role-policy \
  --role-name ecsManagedInstanceInfrastructureRole \
  --policy-arn arn:aws:iam::aws:policy/AmazonECSInfrastructureRolePolicyForManagedInstances
```

- Policy 2 - Manage Volume (EBS):

```bash
aws iam attach-role-policy \
  --role-name ecsManagedInstanceInfrastructureRole \
  --policy-arn arn:aws:iam::aws:policy/service-role/AmazonECSInfrastructureRolePolicyForVolumes
```

![Attach Policy](/images/5-managed-instances/5.1-enable/3.png)

Check if Role has been attached with policy using command:

```bash
aws iam list-attached-role-policies \
  --role-name ecsManagedInstanceInfrastructureRole
```

![List Attached Role Policies](/images/5-managed-instances/5.1-enable/4.png)

Infrastructure Role has been created successfully.

Next we will create Instance Role by creating similarly as above with json file `ecs-managed-instance-role-trust.json` with the json file content below:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Service": "ec2.amazonaws.com"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}
```

Run the following command to create Role:

```bash
aws iam create-role \
  --role-name ecsInstanceRoleManagedInstance \
  --assume-role-policy-document file://ecs-managed-instance-role-trust.json
```

![Create Instance Role](/images/5-managed-instances/5.1-enable/5.png)

Proceed to assign ECS Agent Policy:

```bash
aws iam attach-role-policy \
  --role-name ecsInstanceRoleManagedInstance \
  --policy-arn arn:aws:iam::aws:policy/AmazonECSInstanceRolePolicyForManagedInstances
```

![Attach ECS Agent Policy](/images/5-managed-instances/5.1-enable/6.png)

Check Role again:

```bash
aws iam list-attached-role-policies \
  --role-name ecsInstanceRoleManagedInstance
```

![List Attached Role Policies](/images/5-managed-instances/5.1-enable/7.png)

#### Create Instance Profile

Proceed to create Instance Profile with the Instance Role just created:

```bash
aws iam create-instance-profile --instance-profile-name ecsInstanceRoleManagedInstanceProfile
```

![Create Instance Profile](/images/5-managed-instances/5.1-enable/8.png)

Proceed to assign Role to Instance Profile:

```bash
aws iam add-role-to-instance-profile \
  --instance-profile-name ecsInstanceRoleManagedInstanceProfile \
  --role-name ecsInstanceRoleManagedInstance
```

![Assign Role to Instance Profile](/images/5-managed-instances/5.1-enable/9.png)

Check Instance Profile again:

```bash
aws iam list-instance-profiles
```

![List Instance Profiles](/images/5-managed-instances/5.1-enable/10.png)

Instance Profile has been created successfully.

Finally, create **ECS Managed Instances capacity provider**. See [AWS Documentation](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/managed-instances-capacity-providers-concept.html) for more information.

The capacity provider below will provide instances running in 2 private subnets, using **UI_SG_ID security group**, and have disk size 100GB. See [AWS Documentation](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/create-capacity-provider-managed-instances.html) for more examples on how to configure capacity provider for Amazon ECS Managed Instances.

We will create capacity provider with the json file content below named `retail-store-managed-instances-cp.json`:

```json
{
    "name": "retail-store-managed-instances-cp",
    "cluster": "retail-store-ecs-cluster",
    "managedInstancesProvider": {
        "infrastructureRoleArn": "arn:aws:iam::${ACCOUNT_ID}:role/ecsManagedInstanceInfrastructureRole",
        "instanceLaunchTemplate": {
            "ec2InstanceProfileArn": "arn:aws:iam::${ACCOUNT_ID}:instance-profile/ecsInstanceRoleManagedInstanceProfile",
            "networkConfiguration": {
                "subnets": [
                    "$PRIVATE_SUBNET1",
                    "$PRIVATE_SUBNET2"
                ],
                "securityGroups": [
                    "${UI_SG_ID}"
                ]
            },
            "storageConfiguration": {
                "storageSizeGiB": 100
            }
        }
    }
}
```

Use the following CLI to create capacity provider:

```bash
aws ecs create-capacity-provider --cli-input-json file://retail-store-managed-instances-cp.json
```

![Create Capacity Provider](/images/5-managed-instances/5.1-enable/11.png)

Check capacity provider on AWS Console:

![Check Capacity Provider](/images/5-managed-instances/5.1-enable/12.png)

**ECS Capacity Provider Details** is displayed as follows:

![Check Capacity Provider](/images/5-managed-instances/5.1-enable/13.png)

Your new ECS Managed Instances capacity provider with the following key characteristics:

- **Type**: Managed Instances.

- **Infrastructure Role**: Configured for automatic instance management.

- **Instance Profile**: Allows instances to join ECS cluster.

- **Network Configuration**: Uses your existing private subnets.

- **Storage**: 100 GiB EBS storage for each instance.

- **Monitoring**: Basic CloudWatch monitoring enabled.

After creating ECS Managed Instances capacity provider, proceed to the next step to deploy applications using this new compute option.
