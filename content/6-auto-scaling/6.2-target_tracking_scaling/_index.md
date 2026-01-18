---
title : "Target Tracking Scaling"
date :  2025-01-01
weight : 2
chapter : false
pre : " <b> 6.2. </b> "  
---

#### Register Scalable Target

In this section, we will configure ECS Service Auto Scaling using Target Tracking Scaling. 

First, Register UI service as a scalable target with Application Auto Scaling. The following command sets scaling range for UI Service from minimum 2 to maximum 10 tasks.

The `register-scalable-target` command is used to allow ECS Service to be auto scaled and limit the scale range (min/max tasks); without this step, all scaling policies are invalid.

```bash
aws application-autoscaling register-scalable-target \
    --service-namespace ecs \
    --scalable-dimension ecs:service:DesiredCount \
    --resource-id service/retail-store-ecs-cluster/ui \
    --min-capacity 2 \
    --max-capacity 10
```

![Register Scalable Target](/images/6-auto-scaling/6.2-target_tracking_scaling/1.png)

![Register Scalable Target](/images/6-auto-scaling/6.2-target_tracking_scaling/2.png)

{{% notice note %}}
Scalable Target is an object that can be Auto Scaled. This declaration tells Application Auto Scaling: "This ECS Service IS ALLOWED to auto scale, and can only scale within a specified range."
{{% /notice %}}

#### Create Scaling Policy

Scaling policy is a set of rules that determine when and how Amazon ECS Service will increase or decrease the number of tasks, based on monitored metrics.

Scaling policy does not directly perform resource scaling, but:
- Monitors metrics from CloudWatch
- Compares metrics with configured conditions or targets
- Sends requests to Application Auto Scaling to adjust DesiredCount of ECS Service

In Amazon ECS:
- Scaling policy is managed by Application Auto Scaling
- The object being scaled is DesiredCount of ECS Service

Next, we will create a scaling policy for this scalable target.

First, create a JSON configuration file for scaling policy. This configuration uses predefined metric type of request count per target related to Application Load Balancer (ALB) routing requests to ECS service. In this case, our target is 1,500 requests for each ECS task (or target).

{{% notice note %}}
This scaling policy is for example purposes only. You need to understand the specific workload's scaling profile to determine appropriate scaling metrics and thresholds before enabling autoscaling.
{{% /notice %}}

JSON configuration file `ui-scaling-policy.json`:

```json
{
    "TargetValue": 1500,
    "PredefinedMetricSpecification": {
        "PredefinedMetricType": "ALBRequestCountPerTarget",
        "ResourceLabel": "$UI_ALB_PREFIX/$UI_TG_PREFIX"
    }
}
```

Note that we are using environment variables `$UI_ALB_PREFIX` and `$UI_TG_PREFIX` to specify the correct ALB and Target Group for UI service.

- `$UI_ALB_PREFIX`: ARN of ALB, however only the prefix part is needed. Example: `app/ecs-workshop-alb/07eeccd166f4ef38`

![UI ALB ARN](/images/6-auto-scaling/6.2-target_tracking_scaling/6.png)

- `$UI_TG_PREFIX`: Target Group prefix for UI service. Example: `targetgroup/ui-tg/2feb79f68add27c7`

![UI TG ARN](/images/6-auto-scaling/6.2-target_tracking_scaling/7.png)

Now, the `ResourceLabel` in the JSON configuration file is as follows: `app/ecs-workshop-alb/07eeccd166f4ef38/targetgroup/ui-tg/2feb79f68add27c7`

```bash
aws application-autoscaling put-scaling-policy \
    --service-namespace ecs \
    --scalable-dimension ecs:service:DesiredCount \
    --resource-id service/retail-store-ecs-cluster/ui \
    --policy-name ui-scaling-policy \
    --policy-type TargetTrackingScaling \
    --target-tracking-scaling-policy-configuration file://ui-scaling-policy.json
```

![Target Tracking Scaling Policy](/images/6-auto-scaling/6.2-target_tracking_scaling/3.png)

Check target tracking scaling policy in Amazon ECS console.

![Target Tracking Scaling Policy](/images/6-auto-scaling/6.2-target_tracking_scaling/5.png)

#### CloudWatch Alarm

Alarm state can change depending on the number of requests. For example, `UI-AlarmLow` will trigger when the number of requests drops below 1350.

![CloudWatch Alarm](/images/6-auto-scaling/6.2-target_tracking_scaling/4.png)

**High Alarm Configuration (Scale-out):**

- Metric: ALBRequestCountPerTarget > 1500

- Evaluation: 3 data points in 3 minutes

- Action: When in ALARM state, will add tasks to ECS service

- Behavior: Continue adding tasks in subsequent evaluation periods if alarm still exists, up to the limit specified in scaling policy

**Low Alarm Configuration (Scale-in):**

- Metric: ALBRequestCountPerTarget < 1350

- Evaluation: 15 data points in 15 minutes

- Action: When in ALARM state, will reduce number of tasks

- Behavior: Scale-in slower to prioritize high availability

#### Trigger Auto Scaling

Proceed to create synthetic load to trigger auto scaling.

First we need to get DNS Name of Application Load Balancer attached to UI service.

```bash
export RETAIL_ALB=$(aws elbv2 describe-load-balancers --name ecs-workshop-alb \
  --query 'LoadBalancers[0].DNSName' --output text)
echo "Retail ALB: ${RETAIL_ALB}"
```

![Trigger Auto Scaling](/images/6-auto-scaling/6.2-target_tracking_scaling/8.png)

Next, we will use [hey tool](https://github.com/rakyll/hey) to send requests to the /home path of UI service:

You can install hey tool with the following command:

```bash
sudo apt update
sudo apt install hey -y
```

```bash
hey -n 1000000 -c 5 -q 40 http://$RETAIL_ALB/home &
```

![Trigger Auto Scaling](/images/6-auto-scaling/6.2-target_tracking_scaling/9.png)

Scaling activity will be triggered when high alarm for scaling metric is breached in 3 consecutive cycles, each cycle 1 minute. If you want to automatically wait until the alarm is triggered, you can run the following command (approximately ~ 4 minutes):

```bash
sleep 90 && aws cloudwatch wait alarm-exists --alarm-name-prefix \
 TargetTracking-service/retail-store-ecs-cluster/ui-AlarmHigh --state-value ALARM
```
When the alarm is triggered, you will see task count of service scale out from 2 to a higher number:

```bash
aws ecs describe-tasks \
    --cluster retail-store-ecs-cluster \
    --tasks $(aws ecs list-tasks --cluster retail-store-ecs-cluster --query 'taskArns[]' --output text) \
    --query "tasks[*].[group, launchType, lastStatus, healthStatus, taskArn]" --output table
```

![Trigger Auto Scaling](/images/6-auto-scaling/6.2-target_tracking_scaling/11.png)

![Trigger Auto Scaling](/images/6-auto-scaling/6.2-target_tracking_scaling/12.png)

You can observe the high alarm related to scaling policy change to ALARM state in CloudWatch console, as shown below.

![Trigger Auto Scaling](/images/6-auto-scaling/6.2-target_tracking_scaling/10.png)

You can also check the Events tab in the UI Service page to see scaling activity, where desired count increases beyond the initial number of tasks.

![Trigger Auto Scaling](/images/6-auto-scaling/6.2-target_tracking_scaling/13.png)

Next we will proceed to reduce the number of tasks in ECS service.

Stop Hey process:

```bash
pkill -9 hey
```

Normally, after a few minutes, the number of tasks will scale back to the minimum of 2. However, to save time, we can force scale down the service by running the following commands:

```bash
aws ecs wait services-stable --cluster retail-store-ecs-cluster --services ui
```

```bash
aws ecs update-service \
    --cluster retail-store-ecs-cluster \
    --service ui \
    --task-definition retail-store-ecs-ui \
    --desired-count 2
```

![Force Scale Down](/images/6-auto-scaling/6.2-target_tracking_scaling/14.png)

![Force Scale Down](/images/6-auto-scaling/6.2-target_tracking_scaling/15.png)
