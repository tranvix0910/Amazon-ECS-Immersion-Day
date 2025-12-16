---
title : "Express Mode"
date :  2025-01-01
weight : 4
chapter : false
pre : " <b> 4. </b> "  
---

{{% notice info %}}
Bạn phải hoàn thành các chương sau trước khi thực hiện lab này: [3. Chuẩn bị](/vi/3-prepare/)
{{% /notice %}}

Trong phần này, chúng ta sẽ tìm hiểu **Amazon ECS Express Mode** — một giải pháp triển khai container được đơn giản hóa, cho phép developer khởi chạy các ứng dụng container hóa có tính High Availability và Scalability chỉ với một lệnh duy nhất.

Bằng cách tự động hóa việc thiết lập hạ tầng và tuân thủ các best practices của AWS, ECS Express Mode loại bỏ sự phức tạp của container orchestration, trong khi vẫn giữ quyền truy cập đầy đủ vào các khả năng của Amazon ECS khi cần.

Điều này giúp các nhóm phát triển tập trung vào việc xây dựng ứng dụng, thay vì phải quản lý các thành phần hạ tầng như load balancing, auto-scaling và networking.

{{< youtube z9JUEQjpGgY >}}

#### Lợi ích của Amazon ECS Express Mode

- **Triển khai đơn giản**: Triển khai với các cấu hình mặc định sẵn sàng cho production chỉ bằng một lệnh.

- **Không giới hạn khả năng**: Vẫn có toàn quyền truy cập vào các tài nguyên AWS bên dưới khi cần.

- **Tối ưu chi phí**: Chia sẻ Application Load Balancer giữa các service để giảm chi phí.

- **Hạ tầng minh bạch**: Tất cả tài nguyên đều hiển thị rõ ràng và có thể truy cập trong tài khoản AWS của bạn.