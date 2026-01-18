---
title : "Amazon ECS Network Mode"
date :  2025-01-01
weight : 1
chapter : false
pre : " <b> 7.1. </b> "  
---

Khi chạy containers, việc xem xét cấu hình mạng của các container đang chạy trên host là rất quan trọng. Hãy tham khảo [AWS Documentation](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/task-networking.html) để biết thêm thông tin về cách lựa chọn network mode phù hợp. 

Trong phần này, chúng ta sẽ có cái nhìn tổng quan về cấu hình mạng **AWSVPC Mode** cho Amazon ECS trên Fargate.

Trong **AWSVPC Mode**, Amazon ECS sẽ tạo và quản lý một Elastic Network Interface (ENI) cho mỗi task, và mỗi task sẽ nhận một private IP address riêng trong VPC. Cấu hình này mang lại sự linh hoạt cao, cho phép kiểm soát việc giao tiếp giữa các task và service ở mức độ chi tiết (granular level). 

**AWSVPC Network Mode** được hỗ trợ cho các Amazon ECS task chạy trên cả Amazon EC2 và Fargate. Tham khảo [AWS Documentation](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/task-networking-awsvpc.html) để biết thêm chi tiết.

{{% notice note %}}
Khi sử dụng Amazon ECS trên Fargate, AWSVPC Network Mode là bắt buộc.
{{% /notice %}}

