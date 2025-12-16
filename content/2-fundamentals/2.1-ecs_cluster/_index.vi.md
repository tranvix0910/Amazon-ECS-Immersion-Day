---
title : "Amazon ECS Cluster"
date :  2025-01-01
weight : 2.1
chapter : false
pre : " <b> 2.1. </b> "  
---

### Amazon ECS Cluster

- ECS Cluster là một môi trường chạy Container, là **logical group** để quản lý tài nguyên compute.

- Bên trong ECS Cluster sẽ có **EC2 instances** hoặc **Fargate capacity** để chạy container.

- Tất cả **Tasks** và **Services** sẽ được quản lý bởi ECS Cluster.

![Amazon ECS Cluster](/images/2-fundamentals/2.1-core_components/1.png?width=300px)

{{% notice note %}}
Cluster KHÔNG phải là máy, nó chỉ là vùng quản lý.
{{% /notice %}}

- So sánh 2 kiểu **Launch Type** trong ECS: 

|                        | ECS on EC2            | ECS on Fargate        |
| ---------------------- | --------------------- | --------------------- |
| Bạn quản lý server     | **✔**                     | **✗**                     |
| Control instance type  | **✔**                     | **✗**                     |
| Trả tiền theo instance | **✔**                     | **✗**                     |
| Serverless             | **✗**                     | **✔**                     |
| Phù hợp                | workload lớn, ổn định | microservice, startup |