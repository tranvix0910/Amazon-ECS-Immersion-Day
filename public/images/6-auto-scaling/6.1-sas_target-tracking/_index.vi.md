---
title : "Service Auto Scaling & Target Tracking Scaling"
date :  2025-01-01
weight : 1
chapter : false
pre : " <b> 6.1. </b> "  
---

#### Service Auto Scaling

Amazon ECS tích hợp với Amazon CloudWatch để cho phép scale ECS services một cách hiệu quả dựa trên các real-time metrics. Các metric này được truyền từ Amazon ECS sang CloudWatch theo chu kỳ mỗi một phút, cho phép giám sát chính xác và đưa ra các quyết định scale kịp thời. 

Khi các metric vượt quá thresholds được định nghĩa trong scaling policy, CloudWatch sẽ kích hoạt một alarm để điều chỉnh desired number of tasks trong service của bạn. Quá trình điều chỉnh động này sẽ tăng desired capacity trong các sự kiện scale-out và giảm trong các sự kiện scale-in, đảm bảo việc sử dụng tài nguyên tối ưu.

