---
title : "Scheduled Scaling"
date :  2025-01-01
weight : 3
chapter : false
pre : " <b> 6.3. </b> "  
---

{{% notice warning %}}
You must complete the [Target Tracking Scaling](/6-auto-scaling/6.2-target_tracking_scaling/) section before proceeding with this section.
{{% /notice %}}

In this section, we will configure ECS Service Auto Scaling using Scheduled Scaling. This includes setting up scaling rules based on fixed schedules, suitable for services with predictable load patterns.

We will create a scheduled scaling for the UI service using cron expression (auto scaling schedule) to test scale-out and scale-in by adjusting the minimum value.

Run the following command to increase minimum capacity from 2 to 5 after 1 minute from the current time.

```bash
aws application-autoscaling put-scheduled-action \
 --service-namespace ecs \
 --scalable-dimension ecs:service:DesiredCount \
 --resource-id service/retail-store-ecs-cluster/ui \
 --scheduled-action-name "test-scale-up" \
 --schedule "at($(date -u -d "+1 minutes" "+%Y-%m-%dT%H:%M:%S"))" \
 --scalable-target-action MinCapacity=5,MaxCapacity=10
```

![Scheduled Scaling](/images/6-auto-scaling/6.3-scheduled_scaling/1.png)

{{% notice note %}}
You need to register a scalable target before creating scheduled scaling.
{{% /notice %}}

Check Event:

![Scheduled Scaling](/images/6-auto-scaling/6.3-scheduled_scaling/4.png)

Run the following command to observe the scheduled action. You will see a list of 5 ECS tasks.

```bash
aws ecs describe-tasks \
 --cluster retail-store-ecs-cluster \
 --tasks $(aws ecs list-tasks --cluster retail-store-ecs-cluster --service-name ui --query 'taskArns[]' --output text) \
 --query "tasks[*].[group, launchType, lastStatus, healthStatus, taskArn]" --output table
```

![Scheduled Scaling](/images/6-auto-scaling/6.3-scheduled_scaling/2.png)

Check on ECS Consle:

![Scheduled Scaling](/images/6-auto-scaling/6.3-scheduled_scaling/3.png)

Now run the following command to scale back. This command will set the desired capacity to 2, which is the initial task level.

```bash
aws ecs update-service \
    --cluster retail-store-ecs-cluster \
    --service ui \
    --task-definition retail-store-ecs-ui \
    --desired-count 2
```

![Scheduled Scaling](/images/6-auto-scaling/6.3-scheduled_scaling/5.png)

Check on ECS Consle:

![Scheduled Scaling](/images/6-auto-scaling/6.3-scheduled_scaling/6.png)

#### Deploying Scheduled Scaling in Practice

The commands can be adjusted to schedule scaling in the necessary time frames based on application requirements. For example, scheduled scaling can be configured to trigger at 9 AM UTC and 6 PM UTC daily, suitable for peak hours of a global application.

The following example can be applied in practice.

The following command will trigger scale-out, increasing the number of tasks to 5 every day at 9:00 AM UTC to handle expected high traffic during the day:

```bash
aws application-autoscaling put-scheduled-action \
    --service-namespace ecs \
    --scalable-dimension ecs:service:DesiredCount \
    --resource-id service/retail-store-ecs-cluster/ui \
    --scheduled-action-name "week-day-scale-out" \
    --schedule "cron(0 9 ? * MON-FRI *)" \
    --scalable-target-action MinCapacity=5,MaxCapacity=10
```

The following command will trigger scale-in, reducing the number of tasks during off-peak hours. For example, we will scale down the UI service at 6 PM every day.

```bash
aws application-autoscaling put-scheduled-action \
    --service-namespace ecs \
    --scalable-dimension ecs:service:DesiredCount \
    --resource-id service/retail-store-ecs-cluster/ui \
    --scheduled-action-name "week-day-scale-in" \
    --schedule "cron(0 18 ? * MON-FRI *)" \
    --scalable-target-action MinCapacity=2,MaxCapacity=10
```
