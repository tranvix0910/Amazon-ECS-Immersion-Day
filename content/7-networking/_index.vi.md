--- 
title : "Networking"
date :  2025-01-01
weight : 7
chapter : false
pre : " <b> 7. </b> "  
---

Ứng dụng hiện tại được xây dựng từ nhiều thành phần phân tán (microservices) giao tiếp với nhau. Ví dụ, thành phần UI giao tiếp thông qua API với thành phần Checkout, thành phần này lại được liên kết với một lớp lưu trữ bền vững sử dụng Redis, như được minh họa trong sơ đồ kiến trúc bên dưới. Đồng thời, cả UI và Checkout đều giao tiếp thông qua API với dịch vụ Orders, dịch vụ này sử dụng PostgreSQL làm lớp lưu trữ dữ liệu.

![Architecture](/images/7-networking/1.png)

Trong phần này chúng ta sẽ tìm hiểu về **ECS Networking** và cách nó hoạt động liên quan đến Fargate.