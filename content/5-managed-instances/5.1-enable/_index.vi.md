---
title : "Enable ECS Managed Instances"
date :  2025-01-01
weight : 1
chapter : false
pre : " <b> 5.1. </b> "  
---

Trong phần này, bạn sẽ thực hiện thiết lập để triển khai workload trên Amazon ECS Managed Instances.

Cụ thể, chúng ta sẽ thiết lập **Amazon ECS capacity providers** mặc định cho **ECS Managed Instances**.

Amazon ECS capacity providers chịu trách nhiệm quản lý việc scale hạ tầng cho các task trong ECS cluster.

Amazon ECS hiện cung cấp ba loại capacity provider cho cluster:

- **Fargate capacity providers**: Đây là các capacity provider được định nghĩa sẵn cho AWS Fargate. Đây cũng là capacity provider mà chúng ta đã sử dụng từ đầu workshop.

- **Managed Instances capacity providers**: Cho phép sử dụng Amazon ECS Managed Instances để chạy workload.
    - 👉 Đây là capacity provider sẽ được cấu hình trong phần này.

- **Auto Scaling group capacity providers**: Cho phép sử dụng Amazon EC2 instances để chạy workload thông qua Auto Scaling Group.

![ECS Capacity Providers](/images/5-managed-instances/5.1-enable/1.png)

#### Tạo ECS Managed Instances Capacity Provider

Trước khi tạo capacity provider, chúng ta cần hai IAM role (tham khảo [AWS Documentation](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/ManagedInstances.html#managed-instances-iam-roles) để biết thêm chi tiết):

- **Infrastructure Role**: Cho phép Amazon ECS quản lý ECS Managed Instances thay mặt bạn.

- **Instance profile**: Cung cấp quyền cho Amazon ECS container agent chạy trên các managed instances.

