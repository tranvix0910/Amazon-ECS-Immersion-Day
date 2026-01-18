---
title : "Amazon ECS Task"
date :  2025-01-01
weight : 2.3
chapter : false
pre : " <b> 2.3. </b> "  
---

### Amazon ECS Task

Task represents a running instance of a Task Definition in an ECS cluster. Task is the result when we launch a Task Definition at a specific time in a cluster. If we specify running 3 tasks from a Task Definition, AWS ECS will launch 3 separate instances of that Task Definition.

#### Task Lifecycle

When a task is launched (either as part of a service, or as a standalone task), it will go through many different states:

    PROVISIONING → PENDING → ACTIVATING → RUNNING → DEACTIVATING → STOPPING → DEPROVISIONING → STOPPED

Each state has meaning:

- **PROVISIONING**: ECS is allocating necessary resources (Fargate only).

- **PENDING**: Task is waiting for resources (CPU, memory) to be allocated from cluster.

- **ACTIVATING**: Container is starting and health checks are occurring.

- **RUNNING**: Container is running normally.

- **DEACTIVATING**: Task is preparing to stop.

- **STOPPING**: Task is being stopped.

- **DEPROVISIONING**: Resources are being released (Fargate only).

- **STOPPED**: Task has completely stopped.

ECS tracks two different states:

- **lastStatus**: Current actual state of the task.

- **desiredStatus**: Target state we want the task to achieve.

For example, if we request to stop a task, lastStatus might be RUNNING while desiredStatus will be STOPPED.

#### Task Execution Role vs Task Role

There are two types of IAM roles in ECS task:

- **Task Execution Role**: is the role used by ECS Container Agent (not container application). It grants permissions to ECS agent to perform actions such as:

    - Pull Docker images from Amazon ECR
    - Access Secrets Manager or Parameter Store to get configuration
    - Send logs to CloudWatch Logs
    - Pull files from S3 to launch

- **Task Role**: is the role used by the application inside the container. It allows the application to call other AWS services such as:

    - S3 (to read/write files)
    - DynamoDB (to access database)
    - SNS/SQS (to send messages)
    - RDS (to access database)

#### How ECS Places Tasks on Container Instances

When we run a task on an EC2-based cluster, ECS must decide which instance it will run on. This process is called task placement, and it can be controlled by **Placement Strategies** and **Placement Constraints**.

**Placement Strategies**:

- **spread**: Distribute tasks across multiple instances based on an attribute (e.g., ecs.availability-zone, ecs.instance-type). Good for high availability

- **binpack**: Pack tasks into as few instances as possible to save costs

- **random**: Place tasks randomly (rarely used)

**Placement Constraints**:

- **distinctInstance**: Ensure each task runs on a different instance

- **memberOf**: Only place tasks on instances with specific attributes/tags

#### Tasks - Standalone vs Service

**Standalone Task**: Used for short-term jobs (batch jobs, one-off tasks). Task will run until completion or error, then stop. No automatic restart mechanism.

**Service Task**: Used for long-term applications. Service scheduler automatically maintains the desired number of tasks by restarting any stopped task.
