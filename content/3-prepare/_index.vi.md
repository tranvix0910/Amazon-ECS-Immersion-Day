---
title : "Chuẩn bị"
date :  2025-01-01
weight : 3
chapter : false
pre : " <b> 3. </b> "  
---

Trong phần này, bạn sẽ thực hiện các bước chuẩn bị cần thiết để sẵn sàng cho các phần thực hành tiếp theo. Các bước chuẩn bị này sẽ giúp bạn thiết lập môi trường và cấu hình cơ bản để triển khai và quản lý các ứng dụng containerized trên Amazon ECS.

Trong phần Chuẩn bị này, bạn sẽ:

[3.1 - Tạo User & Access Key](/vi/3-prepare/3.1-user_accesskey/)
- Cài đặt và cấu hình AWS CLI
- Tạo IAM User với quyền AdministratorAccess
- Tạo Access Key để xác thực với AWS services

[3.2 - Tạo Cluster & Task Definition](/vi/3-prepare/3.2-cluster-task_definitions/)
- Tạo Amazon ECS Cluster với CloudWatch Container Insights
- Định nghĩa Task Definitions với các thông số CPU, memory và container specifications

[3.3 - Services](/vi/3-prepare/3.3-services/)
- Tạo VPC và cấu hình networking (subnets, NAT Gateway)
- Triển khai và quản lý ECS Services để duy trì số lượng tasks mong muốn

[3.4 - Cập nhật Services](/vi/3-prepare/3.4-update_services/)
- Cập nhật services với cấu hình mới và deployments
- Sử dụng environment variables để cấu hình ứng dụng