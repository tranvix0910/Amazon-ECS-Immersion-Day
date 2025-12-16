---
title : "Các khái niệm cơ bản"
date :  2025-01-01
weight : 2
chapter : false
pre : " <b> 2. </b> "  
---

### Amazon ECS Core Components

![Amazon ECS Core Components](/images/2-fundamentals/1.png)

Amazon ECS có 4 thành phần chính:

- **Cluster**: Là không gian quản lý trung tâm trong Amazon ECS, nơi tổ chức và vận hành toàn bộ service và task của ứng dụng.

- **Service**: Đại diện cho một nhóm các task giống hệt nhau, có nhiệm vụ duy trì số lượng task luôn chạy đúng như mong muốn, đồng thời hỗ trợ tự động khởi động lại hoặc mở rộng khi cần.

{{< youtube fXiUlXy5kRA >}}

- **Task**: Là đơn vị thực thi trong ECS, bao gồm một hoặc nhiều container phối hợp với nhau để thực hiện một chức năng cụ thể của ứng dụng.

- **Task Definition**: Là bản mô tả chi tiết cách một task được chạy, bao gồm các thông tin như CPU, bộ nhớ, container image, cấu hình mạng, IAM role, và các thiết lập cần thiết khác.

{{< youtube 5uJUmGWjRZY >}}

Việc hiểu rõ các thành phần này là rất quan trọng để sử dụng Amazon ECS một cách hiệu quả trong việc quản lý các ứng dụng container. Mỗi thành phần đều đóng vai trò then chốt trong kiến trúc tổng thể và cách vận hành của các ECS deployment.

### Layers

Có 3 Layers trong ECS:

- **Capacity**: Infrastructure nơi Container chạy.

- **Controller**: Deploy và quản lý ứng dụng chạy trên Container.

- **Provisioning**: Công cụ để tương tác với scheduler để deploy và quản lý ứng dụng và container.

![Layers](/images/2-fundamentals/2.png)

### Capacity

Capacity (năng lực tính toán) là hạ tầng nơi các container của chạy. Dưới đây là tổng quan về các tùy chọn capacity:

#### Amazon ECS Managed Instances

- Amazon ECS Managed Instances là một tùy chọn compute cho Amazon ECS, cho phép chạy các workload container hóa trên nhiều loại Amazon EC2 instance, đồng thời chuyển phần lớn việc quản lý hạ tầng cho AWS.

- Với Amazon ECS Managed Instances, có thể:
    - GPU acceleration.
    - Kiến trúc CPU cụ thể.
    - Hiệu năng mạng cao.
    - Các loại instance chuyên dụng.
- Trong khi đó, AWS sẽ chịu trách nhiệm:
    - Cấp phát hạ tầng.
    - Vá lỗi hệ điều hành.
    - Scale.
    - Bảo trì và vận hành hạ tầng bên dưới.

#### Amazon EC2 Instances

- Có thể tự chọn loại Instance, số lượng Instance.
- Tự quản lý Capacity (Auto Scaling, patching, lifecycle, etc.).

#### Serverless

- AWS Fargate là một compute engine serverless, trả tiền theo mức sử dụng.
- Với Fargate:
    - Không cần quản lý server.
    - Không cần lập kế hoạch capacity.
    - Không cần tự cách ly workload container vì AWS đã xử lý vấn đề bảo mật.

#### Server/VM On-Premises

- Amazon ECS Anywhere đăng ký các instance bên ngoài (on-premises server hoặc VM). Kết nối chúng vào ECS cluster.

#### Amazon ECS Scheduler

- Amazon ECS Scheduler là phần mềm chịu trách nhiệm quản lý và điều phối ứng dụng của bạn, bao gồm:
    - Quyết định task chạy ở đâu.
    - Quản lý vòng đời task.
    - Đảm bảo desired state của service.