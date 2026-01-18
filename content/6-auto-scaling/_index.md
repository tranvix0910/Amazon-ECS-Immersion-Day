---
title : "Auto Scaling"
date :  2025-01-01
weight : 6
chapter : false
pre : " <b> 6. </b> "  
---

Since a Fargate instance corresponds to one ECS task, you need to specify CPU and memory of the task when creating task definition. Therefore, right-sizing Fargate tasks is very important to ensure they can perform tasks with desired performance levels. 

If a task encounters difficulties due to insufficient CPU or memory, this indicates the task has not been sized correctly and may need additional resources. You can accurately assess application needs through **performance measurement**, **comprehensive load testing**, or closely monitoring **key metrics**.

Once you are certain that tasks are properly sized, you can scale horizontally by deploying additional tasks to handle more requests. **Horizontal scaling** is the preferred method to scale cloud-native, containerized workloads.

![Horizontal scaling](/images/6-auto-scaling/1.png?width=60%)

## Service Auto Scaling

Amazon ECS provides the ability to automatically adjust **desired number of tasks** in a service, this feature is called **Service Auto Scaling**. 

**ECS Service Auto Scaling** uses Application Auto Scaling to provide this functionality. For auto scaling to work effectively, the metric used must be a proportional metric. If you keep the number of tasks in the service constant and load doubles, then the metric value must also double.

{{% notice note %}}
Proportional metric is a metric that increases/decreases proportionally with system load.
{{% /notice %}}

In the context of ECS when using EC2 instances, you also need to consider using capacity providers to manage capacity of EC2 instances alongside ECS Service Auto Scaling. However, since this lab section primarily focuses on Fargate, we will not go deep into ECS capacity providers in this section.

{{< youtube YDbqnZ32NdM >}}

