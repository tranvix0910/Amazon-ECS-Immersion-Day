---
title : "Workshop Amazon Elastic Container Service & AWS Fargate"
date :  2025-01-01
weight : 1
chapter : false
pre : " <b> 1 </b> "
---

# Workshop Amazon Elastic Container Service & AWS Fargate

Chào mừng bạn đến với **Amazon Elastic Container Service Immersion Day!**

![1.1](/images/ecs.png)

**Amazon Elastic Container Service (ECS)** là một dịch vụ **Container Orchestration Service** được quản lý hoàn toàn bởi AWS. Nó cho phép bạn Deploy, Quản lý và mở rộng quy mô ứng dụng Container một cách nhanh chống và hiệu quả.

ECS có thể tích hợp với hệ sinh thái AWS, cung cấp một giải pháp dễ sử dụng để chạy các workload container trên môi trường Cloud, đồng thời hỗ trợ các khả năng bảo mật nâng cao thông qua **Amazon ECS Anywhere**.

**AWS Fargate** là một dịch vụ **Serverless**, trả phí theo mức sử dụng ( pay-as-you-go ), cho phép xây dựng ứng dụng mà không cần quan tâm đến hạ tầng. Bằng cách giao việc quản lý máy chủ, phân bổ tài nguyên và scaling cho AWS, điều này giúp nâng cao năng lực vận hành, tăng tốc quá trình đưa ý tưởng vào production và tối ưu TCO (Total Cost of Ownership).

#### Bạn sẽ học được gì thông qua workshop này?

- Hiểu các khái niệm nền tảng của Amazon ECS và AWS Fargate: **cluster**, **task** và **service**.
- Triển khai autoscaling cho workload nhằm đáp ứng các mức tải thay đổi.
- Giám sát hành vi của workload thông qua log, metric và trace.
- Nắm bắt networking trong AWS Fargate và các khái niệm networking nâng cao.
- Khám phá các khía cạnh bảo mật, bao gồm quản lý credential và secret một cách an toàn.
- Tìm hiểu các chiến lược triển khai workload khác nhau trên Amazon ECS.

#### Đối tượng tham gia

- Workshop ở cấp độ **200+**, phù hợp với các bạn muốn Hands-on với Amazon ECS, làm quen với các thành phần cốt lõi của ECS, cũng như cách áp dụng thực tế trong việc triển khai và quản lý các workload container hóa.

#### Pre-requisites

- Đây không phải là workshop nhập môn về container. Người tham gia cần có kiến thức nền tảng về Container, cụ thể:

    - Hiểu về **Container**, bao gồm khả năng build container bằng **Dockerfile**.
    - Có kiến thức cơ bản và quen thuộc với **AWS Management Console**, **AWS APIs**.


#### Nội dung chính

1. [Giới thiệu](1-introduction/)
2. [Chuẩn bị](2-preparation/)
3. [Express Mode](3-express-mode/)
4. [Managed Instances](4-managed-instances/)
5. [Auto Scaling](5-auto-scaling/)
6. [Networking](6-networking/)
7. [Observability](7-observability/)
8. [Security](8-security/)
9. [Deployments](9-deployments/)
10. [Storage](10-storage/)
11. [Cost Optimizations](11-cost-optimizations/)
12. [Clean Up](12-clean-up/)