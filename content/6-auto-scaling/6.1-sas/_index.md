---
title : "Service Auto Scaling"
date :  2025-01-01
weight : 1
chapter : false
pre : " <b> 6.1. </b> "  
---

#### Service Auto Scaling

Amazon ECS integrates with Amazon CloudWatch to allow scaling ECS services effectively based on real-time metrics. These metrics are transmitted from Amazon ECS to CloudWatch every minute, enabling accurate monitoring and timely scaling decisions. 

When metrics exceed thresholds defined in scaling policy, CloudWatch will trigger an alarm to adjust desired number of tasks in your service. This dynamic adjustment process will increase desired capacity in scale-out events and decrease in scale-in events, ensuring optimal resource usage.

![Service Auto Scaling](/images/6-auto-scaling/6.1-sas/1.png?width=80%)

Amazon ECS provides three advanced service scaling strategies:

**Target Tracking Scaling**:

- This method aims to maintain a scaling metric at a specific target value by automatically adjusting the number of tasks. Target tracking scaling is preferred due to its simplicity and low maintenance requirements, making it an ideal choice for businesses wanting to achieve operational efficiency without frequent manual intervention.

**Step Scaling**:

- This strategy provides more detailed control over scaling actions. Users can choose metrics, set threshold values, and define step adjustments to specify the amount of resources to add or remove. It also allows customizing breach evaluation periods for metric alarms, providing a flexible approach to efficiently handle volatile workloads.

**Scheduled Scaling**:

- This method is most suitable when scaling actions can be predicted in advance based on known demand patterns. It is ideal for applications with predictable traffic fluctuations, allowing proactive resource management to ensure service stability and performance during peak times.

#### Target Tracking Scaling

Automatically maintain metric around a target value. ECS doesn't care how many tasks need to be added, it only cares:

- "Is the current metric deviating from the target value?"

- If deviating → ECS automatically calculates the number of tasks to scale.

```
desiredTasks = currentTasks × (currentMetric / targetMetric)
```

Example: CPU-based scaling
- Metric: ECSServiceAverageCPUUtilization
- Target value: 50%
- Currently:
    - 2 tasks
    - CPU = 90%
```
desiredTasks = 2 × (90 / 50) = 3.6 ≈ 4 tasks
```
-> ECS scales 2 → 4 tasks.
- **Advantages**:
    - Easy to configure
    - No complex tuning needed
    - Low maintenance
    - Avoids over-scaling
- **Disadvantages**:
    - Less detailed control
    - Not suitable for workloads with very fast spikes

**When to use?**

- 80–90% of ECS services.
- Web services, APIs, microservices.
- Gradually increasing or relatively stable load.

#### Step Scaling

Scale by "steps" based on threshold.

Proactively define:
- Which metric
- When threshold is exceeded by how much
- Then scale how many tasks

ECS doesn't calculate automatically, but follows the established rules exactly.

Example: CPU-based Step Scaling

- Metric: CPUUtilization

| CPU   | Action |
| ----- | --------- |
| ≥ 60% | +1 task   |
| ≥ 75% | +2 tasks  |
| ≥ 90% | +4 tasks  |
| ≤ 30% | -1 task   |

- Service has 2 tasks and CPU increases to 85% -> ECS increases to 2 tasks.

Step scaling uses CloudWatch Alarm:

Example:
- CPU ≥ 75%
- Maintain for 2 consecutive minutes
→ Trigger scale action

Avoid scaling due to short-term spikes (increase briefly then decrease).

**Advantages**
- Very detailed control
- Suitable for highly volatile workloads
- Can use non-proportional metrics

**Disadvantages**
- Complex configuration
- Easy to make mistakes if tuning is poor
- Requires frequent monitoring & adjustment

**When to use?**
- Workloads with strong, non-linear spikes
- Batch processing
- Legacy systems
- When you need precise control over number of tasks to increase/decrease

#### Scheduled Scaling

Scale according to schedule.

Configurer knows in advance: When load is high, when load is low.

ECS scales according to preset time, no metric needed.

Example: Business hours website with operating hours from 8:00 to 18:00.

| Time     | Tasks |
| ------------- | ----- |
| 08:00 – 18:00 | 6     |
| 18:00 – 08:00 | 2     |

**Advantages**
- Very stable
- Not dependent on metrics
- Avoids cold start

**Disadvantages**
- Cannot react to unexpected traffic
- Must predict correctly

**When to use?**
- Traffic with clear patterns
- Business hours
- Fixed events (sales, livestreams, exam systems)
