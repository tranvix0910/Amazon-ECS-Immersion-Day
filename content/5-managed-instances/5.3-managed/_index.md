---
title : "Managed Instances Infrastructure"
date :  2025-01-01
weight : 3
chapter : false
pre : " <b> 5.3. </b> "  
---

#### Managed Instances Infrastructure

In this section, we will dive deeper into the EC2 instances that Amazon ECS Managed Instances has created to support the UI service we deployed in the previous section.

Run the following command to view EC2 instances:
```bash
aws ec2 describe-instances \
    --filters \
        "Name=tag:aws:ec2:managed-launch,Values=ecs-managed-instances" \
        "Name=instance-state-name,Values=running" \
    --query 'Reservations[*].Instances[*].{InstanceId:InstanceId,InstanceType:InstanceType,State:State.Name,PrivateIpAddress:PrivateIpAddress}' \
    --output table
```

Since UI service is deployed in high-availability (HA) model with 2 running tasks, and Amazon ECS Managed Instance capacity provider is attached to 2 subnets (in different AZs), you will see 2 different EC2 instances running.

You can see different EC2 instance types running in the cluster, because we did not specify any conditions in the capacity provider for Amazon ECS Managed Instances.

![EC2 instances](/images/5-managed-instances/5.3-managed/1.png)

```
------------------------------------------------------------------------
|                           DescribeInstances                          |
+----------------------+---------------+--------------------+----------+
|      InstanceId      | InstanceType  | PrivateIpAddress   |  State   |
+----------------------+---------------+--------------------+----------+
|  i-056556a991639db3c |  c5a.large    |  10.0.145.243      |  running |
|  i-08b1aa2b2ec697a12 |  c5a.large    |  10.0.135.156      |  running |
+----------------------+---------------+--------------------+----------+
```

You can view Instances on ECS Console:

![ECS Instances](/images/5-managed-instances/5.3-managed/2.png)

Now, let's check how Amazon ECS Managed Instance handles additional workload by increasing the number of running tasks of UI service from 2 to 6. In this case, you will notice that because current EC2 instances do not have enough resources to deploy the running tasks, additional EC2 instances will be deployed to support the expanded workload. See [AWS Documentation](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/managed-instances-instance-selection-best-practices.html) for more information on instance selection.

```bash
aws ecs update-service \
    --cluster retail-store-ecs-cluster \
    --service ui-managed-instance \
    --desired-count 6 \
    --placement-constraints \
    --task-definition retail-store-ecs-ui-managed
```

![ECS Tasks](/images/5-managed-instances/5.3-managed/3.png)

After the deployment process completes, let's see how tasks are distributed across EC2 instances.

![ECS Tasks](/images/5-managed-instances/5.3-managed/4.png)

![ECS Tasks](/images/5-managed-instances/5.3-managed/5.png)

You can see there are 2 tasks running on `m7i-flex.large` and 2 tasks running on `m5.large` with the remaining 2 tasks evenly distributed to `c5a.large`.

Check instance types by opening EC2 console to view new EC2 instances. Amazon ECS Managed Instance will initialize additional instances to meet new capacity requirements.

![EC2 instances](/images/5-managed-instances/5.3-managed/6.png)

Finally, scale down the number of tasks of UI service to 2.

```bash
aws ecs update-service \
    --cluster retail-store-ecs-cluster \
    --service ui-managed-instance \
    --desired-count 2 \
    --task-definition retail-store-ecs-ui-managed \
    --placement-constraints type="distinctInstance"
```

![ECS Tasks](/images/5-managed-instances/5.3-managed/7.png)

4 tasks have been removed and there are now 2 instances in idle state.

![EC2 instances](/images/5-managed-instances/5.3-managed/8.png)

After a few more minutes, idle instances will be deregistered and terminated by the capacity provider. Execute the following command, after a few minutes you will only see 2 EC2 instances running:

```bash
aws ec2 describe-instances \
    --filters \
        "Name=tag:aws:ec2:managed-launch,Values=ecs-managed-instances" \
        "Name=instance-state-name,Values=running" \
    --query 'Reservations[*].Instances[*].{InstanceId:InstanceId,InstanceType:InstanceType,State:State.Name,PrivateIpAddress:PrivateIpAddress}' \
    --output table
```

![EC2 instances](/images/5-managed-instances/5.3-managed/9.png)

![EC2 instances](/images/5-managed-instances/5.3-managed/10.png)

