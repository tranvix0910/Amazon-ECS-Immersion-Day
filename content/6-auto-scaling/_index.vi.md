---
title : "Auto Scaling"
date :  2025-01-01
weight : 6
chapter : false
pre : " <b> 6. </b> "  
---

Vì một Fargate instance tương ứng với một ECS task, nên bạn cần chỉ định CPU và memory của task khi tạo task definition. Do đó, việc right-size các Fargate tasks là rất quan trọng để đảm bảo chúng có thể thực hiện nhiệm vụ với mức hiệu năng mong muốn. 

Nếu một task gặp khó khăn do CPU hoặc memory không đủ, điều này cho thấy task chưa được sizing đúng và có thể cần thêm tài nguyên. Có thể đánh giá chính xác nhu cầu của ứng dụng thông qua **performance measurement**, **comprehensive load testing**, hoặc theo dõi sát các **key metrics**.

Khi đã chắc chắn rằng các task được sizing phù hợp, có thể scale horizontally bằng cách triển khai thêm các task để xử lý nhiều request hơn. **Horizontal scaling** là phương pháp được ưu tiên để scale các workload cloud-native, containerized.

![Horizontal scaling](/images/6-auto-scaling/1.png?width=60%)

## Service Auto Scaling

Amazon ECS cung cấp khả năng tự động điều chỉnh **desired number of tasks** trong service, tính năng này được gọi là **Service Auto Scaling**. 

**ECS Service Auto Scaling** sử dụng Application Auto Scaling để cung cấp chức năng này. Để auto scaling hoạt động hiệu quả, metric được sử dụng phải là proportional metric. Nếu giữ nguyên số lượng task trong service và load tăng gấp đôi, thì giá trị metric cũng phải tăng gấp đôi.

{{% notice note %}}
Proportional metric là metric tăng/giảm tỷ lệ thuận với tải (load) của hệ thống.
{{% /notice %}}

Trong bối cảnh ECS khi sử dụng EC2 instances, cũng cần xem xét việc sử dụng capacity providers để quản lý capacity của EC2 instances bên cạnh ECS Service Auto Scaling. Tuy nhiên, vì phần lab này chủ yếu tập trung vào Fargate, nên chúng ta sẽ không đi sâu vào ECS capacity providers trong phần này.

{{< youtube YDbqnZ32NdM >}}

