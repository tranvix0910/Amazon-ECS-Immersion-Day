---
title : "Giới thiệu"
date :  2025-01-01
weight : 1
chapter : false
pre : " <b> 1. </b> "  
---

### Amazon ECS

![Amazon ECS](/images/1-introduction/1.png)

Amazon Elastic Container Service (ECS) là một dịch vụ container orchestration service được quản lý hoàn toàn bởi AWS, giúp đơn giản hóa việc triển khai, quản lý và mở rộng các ứng dụng được đóng gói bằng container. Dịch vụ này tích hợp chặt chẽ với các dịch vụ AWS khác, cung cấp một giải pháp an toàn, dễ sử dụng để chạy các workload container trên môi trường đám mây và cả hạ tầng on-premises thông qua Amazon ECS Anywhere.

{{< youtube o6M8GXwDr9w >}}

### AWS Fargate

![AWS Fargate](/images/1-introduction/2.png)

AWS Fargate là một compute engine dạng serverless, hoạt động theo mô hình trả tiền theo mức sử dụng (pay-as-you-go), cho phép bạn tập trung xây dựng ứng dụng mà không cần quản lý máy chủ. AWS Fargate tương thích với cả Amazon Elastic Container Service (Amazon ECS) và Amazon Elastic Kubernetes Service (Amazon EKS).

{{< youtube yi22xrvPnPk >}}

### Lợi ích của AWS Fargate

- **Tập trung vào ứng dụng, không phải hạ tầng**: Triển khai và quản lý ứng dụng mà không phải gánh các công việc vận hành như mở rộng quy mô (scaling), vá lỗi (patching), bảo mật và quản lý máy chủ.

- **Có cái nhìn toàn diện thông qua giám sát chi tiết**: Giám sát ứng dụng bằng các tích hợp sẵn với dịch vụ AWS như **Amazon CloudWatch Container Insights**, hoặc thu thập metrics và logs thông qua các công cụ của bên thứ ba.

- **Tăng cường bảo mật nhờ cách ly workload**: Nâng cao bảo mật ngay từ thiết kế với cơ chế cách ly workload. Mỗi workload chạy trên AWS Fargate đều sử dụng một compute instance riêng biệt, dùng một lần, dành riêng cho một tenant, và được cách ly bằng lớp ảo hóa. Mỗi Amazon ECS task hoặc Kubernetes pod sẽ chạy trên một instance mới được cấp phát. Để tìm hiểu chi tiết hơn về kiến trúc AWS Fargate, tham khảo [AWS Fargate Security Whitepaper](https://d1.awsstatic.com/whitepapers/AWS_Fargate_Security_Overview_Whitepaper.pdf).

- **Tối ưu chi phí hiệu quả**: Chỉ trả tiền cho tài nguyên compute thực sự sử dụng, không có chi phí trả trước. Có thể giảm chi phí hơn nữa bằng cách sử dụng Savings Plans, Fargate Spot hoặc bộ xử lý AWS Graviton.

![Lợi ích của AWS Fargate](/images/1-introduction/3.png)


### Tổng quan về ứng dụng Microservices được triển khai

Hầu hết các lab trong workshop này sử dụng một ứng dụng Microservices nhằm cung cấp các thành phần container cho các bài thực hành. Ứng dụng mẫu mô phỏng một cửa hàng web đơn giản, nơi khách hàng có thể duyệt danh mục sản phẩm, thêm sản phẩm vào giỏ hàng và hoàn tất đơn hàng thông qua quy trình thanh toán.

![Tổng quan về ứng dụng được triển khai](/images/1-introduction/4.png)

![Tổng quan về ứng dụng được triển khai](/images/1-introduction/5.png)

Ứng dụng bao gồm các thành phần và các dependency như sau:

| Thành phần   | Mô tả                                                                                      |
| ------------ | ------------------------------------------------------------------------------------------ |
| **UI**       | Cung cấp giao diện người dùng phía front-end và tổng hợp các API call tới các Services khác |
| **Catalog**  | API cung cấp danh sách và chi tiết sản phẩm                                                |
| **Cart**     | API quản lý giỏ hàng của khách hàng                                                        |
| **Checkout** | API điều phối toàn bộ quy trình thanh toán                                                 |
| **Orders**   | API tiếp nhận và xử lý đơn hàng của khách hàng                                             |

### Đóng gói ứng dụng thành Container

Trước khi triển khai một workload lên Amazon ECS, workload đó cần được đóng gói dưới dạng **container image** và đẩy (publish) lên một **container registry**.

Các kiến thức container cơ bản như vậy không nằm trong phạm vi của workshop này, và ứng dụng đã có sẵn các container image trong **Amazon Elastic Container Registry (Amazon ECR)** để sử dụng cho các bài lab.

Bảng dưới đây cung cấp đường dẫn tới ECR Public repository cho từng thành phần, cũng như Dockerfile được dùng để build mỗi component.

| Thành phần        | ECR Public repository | Dockerfile |
| ----------------- | --------------------- | ---------- |
| **UI**            | [Repository](https://gallery.ecr.aws/aws-containers/retail-store-sample-ui)            | [Dockerfile](https://github.com/aws-containers/retail-store-sample-app/blob/v1.2.3/src/ui/Dockerfile) |
| **Catalog**       | [Repository](https://gallery.ecr.aws/aws-containers/retail-store-sample-catalog)            | [Dockerfile](https://github.com/aws-containers/retail-store-sample-app/blob/v1.2.3/src/catalog/Dockerfile) |
| **Shopping cart** | [Repository](https://gallery.ecr.aws/aws-containers/retail-store-sample-cart)            | [Dockerfile](https://github.com/aws-containers/retail-store-sample-app/blob/v1.2.3/src/cart/Dockerfile) |
| **Checkout**      | [Repository](https://gallery.ecr.aws/aws-containers/retail-store-sample-checkout)            | [Dockerfile](https://github.com/aws-containers/retail-store-sample-app/blob/v1.2.3/src/checkout/Dockerfile) |
| **Orders**        | [Repository](https://gallery.ecr.aws/aws-containers/retail-store-sample-orders)            | [Dockerfile](https://github.com/aws-containers/retail-store-sample-app/blob/v1.2.3/src/orders/Dockerfile) |