---
title : "ECS Service Connect"
date :  2025-01-01
weight : 2
chapter : false
pre : " <b> 7.2. </b> "  
---

**ECS Service Connect** là phương pháp được khuyến nghị để xử lý việc giao tiếp giữa các service (service-to-service communication), cung cấp các tính năng như service discovery, connectivity và traffic monitoring. 

Với Service Connect, các ứng dụng của bạn có thể sử dụng short names và standard ports để kết nối tới các ECS services trong cùng một cluster, khác cluster, khác VPC, và thậm chí giữa các AWS Accounts trong cùng một AWS Region. [Để biết thêm thông tin chi tiết, hãy tham khảo AWS Documentation](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/networking-connecting-services.html#networking-connecting-services-serviceconnect).

{{< youtube 4xU7AvjVlIY >}}

Các lựa chọn thay thế để cấu hình inter-service communication trong Amazon ECS Services bao gồm:

- [Internal Load Balancer](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/networking-connecting-services.html#networking-connecting-services-elb)

- [Service Discovery](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/networking-connecting-services.html#networking-connecting-services-direct)

- [Amazon VPC Lattice](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/ecs-vpc-lattice.html)

