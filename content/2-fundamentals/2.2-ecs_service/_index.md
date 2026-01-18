---
title : "Amazon ECS Service"    
date :  2025-01-01
weight : 2.2
chapter : false
pre : " <b> 2.2. </b> "  
---

### Amazon ECS Service

ECS Service is a management tool that allows you to run and maintain a certain number of tasks continuously in an ECS cluster. It is an essential part of ECS for long-term applications, unlike standalone tasks used for short-term batch jobs.

If one of the tasks fails or stops, the **ECS Service Scheduler** will launch another instance of the Task Definition to replace it.

This helps maintain the correct desired number of tasks for the service.

Additionally, only Service can be configured with Load Balancer. If you need to distribute traffic to containers, you must use Service, not standalone Task. LB helps:
- High availability
- Scale
- Health check

#### When to use Service Scheduler?

AWS recommends using Service Scheduler for:
- Services
- Stateless applications
- Long-running workloads

Service Scheduler ensures:
- Your configured scheduling strategy is followed
- Tasks are rescheduled when they fail

Example:
If the underlying infrastructure fails, Service Scheduler will restart the task.

#### ECS Service Configuration Components

- **Service Name**: Name of the service to identify it in the cluster

- **Task Definition**: Which Task Definition will be used to launch tasks. Service will reference this Task Definition.

- **Desired Count**: Number of tasks you want Service to keep running continuously. If you specify desired count as 3, Service will always try to maintain 3 running tasks. If a task stops, Service will launch a new task to replace it.​

- **Scheduling Strategy**: There are two options:

    - **REPLICA**: Service runs a desired number of tasks

    - **DAEMON**: Service runs one task on each container instance in the cluster (no desired count)​

- **Deployment Type**: How to deploy new versions:​

    - **ECS**: Use ECS Scheduler (default rolling update)

    - **CODE_DEPLOY**: Use AWS CodeDeploy (supports blue-green deployments)

    - **EXTERNAL**: Allows manual deployment through TaskSets (used with CloudFormation, Jenkins, etc.)

- **Load Balancer Configuration**: If you want to distribute traffic:

    - **Load Balancer Type**: Application Load Balancer (ALB) or Network Load Balancer (NLB)

    - **Target Group**: Identifies which containers will receive traffic

    - **Port Mapping**: Maps from load balancer port to container port

    - **Health Check**: Health check configuration to determine if task is functioning​

#### ECS Service Lifecycle

Service operates in the following cycle:

- **Initialization**: You create Service and specify desired count (e.g., 3 tasks)

- **Launch Tasks**: Service scheduler launches 3 instances of Task Definition

- **Register with Load Balancer**: If there is a load balancer, tasks are automatically registered with target group

- **Continuous Monitoring**: Service continuously checks the status of tasks

- **Replace Failed Tasks**: If a task encounters an issue, Service launches a new task to maintain desired count

- **Scale Up/Down**: If auto scaling is enabled, desired count will automatically adjust based on metrics

#### Deployment Configuration for Rolling Updates

When you deploy a new version of the application (update Task Definition), ECS uses deployment configuration to control the process:

- **MinimumHealthyPercent**: Minimum percentage of tasks that must remain in RUNNING state throughout the deployment process. For example, if desired count is 4 and MinimumHealthyPercent is 50%, ECS ensures at least 2 tasks (50% of 4) are still running when deploying new versions.​

- **MaximumPercent**: Maximum total number of tasks that can run simultaneously throughout the deployment process, expressed as a percentage of desired count. For example, if desired count is 4 and MaximumPercent is 200%, ECS allows up to 8 tasks to run at the same time - 4 old tasks + 4 new tasks. This allows faster deployment but requires more resources.​​

Example deployment process:

- You have 4 tasks running version v1.      

- You update Task Definition to version v2.

    - **MinimumHealthyPercent**: 100%

    - **MaximumPercent**: 200%

- ECS launches 4 new tasks (v2) before stopping 4 old tasks (v1).

- At this point you have 8 tasks running simultaneously.

- Then, ECS stops the 4 old v1 tasks.

- Result: 4 v2 tasks running.

#### Service Auto Scaling​

Service Auto Scaling allows ECS to automatically adjust the number of tasks based on metrics such as CPU, memory, or custom metrics. There are four types of scaling:

- **Target Tracking Scaling**: Maintains a specific metric at a target level. For example, you can say "maintain CPU at 70%". ECS will automatically add tasks when CPU exceeds 70% and remove tasks when CPU is below 70%.​

- **Step Scaling**: Uses CloudWatch alarms with steps. For example, when CPU > 80%, add 2 tasks; when CPU > 90%, add 4 tasks.​

- **Scheduled Scaling**: Adjusts desired count based on schedule. For example, scale up at 9 AM when company opens, scale down at 6 PM when closing.​

- **Predictive Scaling**: Uses machine learning to analyze historical patterns and predict future traffic.​

#### Mechanism for Maintaining Desired Count

It is important to understand that desired count and the actual number of running tasks may differ temporarily:
- **Desired Count**: Number of tasks you want Service to maintain. It is a target value that ECS tries to achieve.​
- **Running Tasks**: Actual number of tasks running at the current time. When you change desired count, ECS needs time to launch or stop tasks to match the new desired count.​

Example:
- You set desired count = 3
- Currently there is 1 task running
- ECS will launch 2 new tasks
- During launch, running tasks = 1 but desired count = 3
- After a few seconds, running tasks = 3 (matches desired count)

#### Deployment vs Auto Scaling​

Service Auto Scaling Min/Max is the allowed range that Service Auto Scaling can adjust desired count within. 

If you set Min = 2 and Max = 10:

- **Service Auto Scaling will never let desired count go below 2**

- **Service Auto Scaling will never let desired count exceed 10**

Deployment Configuration Min/Max Percent only applies during the deployment process (when updating Task Definition). It controls how many tasks can run during deployment, unrelated to normal auto scaling.
