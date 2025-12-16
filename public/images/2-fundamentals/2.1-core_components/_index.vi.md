---
title : "ECS - Core Components"
date :  2025-01-01
weight : 2.1
chapter : false
pre : " <b> 2.1. </b> "  
---

### Amazon ECS Cluster

- ECS Cluster là một môi trường chạy Container, là **logical group** để quản lý tài nguyên compute.

- Bên trong ECS Cluster sẽ có **EC2 instances** hoặc **Fargate capacity** để chạy container.

- Tất cả **Tasks** và **Services** sẽ được quản lý bởi ECS Cluster.



{{% notice note %}}
Cluster KHÔNG phải là máy, nó chỉ là vùng quản lý.
{{% /notice %}}