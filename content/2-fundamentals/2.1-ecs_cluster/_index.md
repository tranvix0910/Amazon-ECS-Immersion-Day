---
title : "Amazon ECS Cluster"
date :  2025-01-01
weight : 2.1
chapter : false
pre : " <b> 2.1. </b> "  
---

### Amazon ECS Cluster

- ECS Cluster is a container runtime environment, a **logical group** for managing compute resources.

- Inside ECS Cluster there will be **EC2 instances** or **Fargate capacity** to run containers.

- All **Tasks** and **Services** will be managed by ECS Cluster.

![Amazon ECS Cluster](/images/2-fundamentals/2.1-core_components/1.png?width=300px)

{{% notice note %}}
Cluster is NOT a machine, it is just a management area.
{{% /notice %}}

- Comparison of 2 **Launch Type** styles in ECS: 

|                        | ECS on EC2            | ECS on Fargate        |
| ---------------------- | --------------------- | --------------------- |
| You manage server     | **✔**                     | **✗**                     |
| Control instance type  | **✔**                     | **✗**                     |
| Pay per instance | **✔**                     | **✗**                     |
| Serverless             | **✗**                     | **✔**                     |
| Suitable                | large, stable workload | microservice, startup |
