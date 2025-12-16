---
title : "Amazon ECS Task"
date :  2025-01-01
weight : 2.3
chapter : false
pre : " <b> 2.3. </b> "  
---

### Amazon ECS Task

Task đại diện cho một instance đang chạy của Task Definition trong một ECS cluster. Task là kết quả khi ta khởi chạy một Task Definition tại một thời điểm cụ thể trong một cluster. Nếu ta chỉ định chạy 3 task từ một Task Definition, AWS ECS sẽ khởi chạy 3 instance riêng biệt của Task Definition đó.

#### Vòng Đời Task (Task Lifecycle)

Khi một task được khởi chạy (hoặc như một phần của service, hoặc như một standalone task), nó sẽ trải qua nhiều trạng thái khác nhau:

    PROVISIONING → PENDING → ACTIVATING → RUNNING → DEACTIVATING → STOPPING → DEPROVISIONING → STOPPED

Mỗi trạng thái có ý nghĩa:

- **PROVISIONING**: ECS đang cấp phát các tài nguyên cần thiết (chỉ với Fargate).

- **PENDING**: Task đang chờ để các tài nguyên (CPU, bộ nhớ) được phân bổ từ cluster.

- **ACTIVATING**: Container đang khởi chạy và các health check đang diễn ra.

- **RUNNING**: Container đang chạy bình thường.

- **DEACTIVATING**: Task đang chuẩn bị dừng.

- **STOPPING**: Task đang được dừng.

- **DEPROVISIONING**: Tài nguyên đang được giải phóng (chỉ với Fargate).

- **STOPPED**: Task đã dừng hoàn toàn.

ECS theo dõi hai trạng thái khác nhau:

- **lastStatus**: Trạng thái thực tế hiện tại của task.

- **desiredStatus**: Trạng thái mục tiêu mà ta muốn task đạt được.

Ví dụ, nếu ta yêu cầu dừng một task, lastStatus có thể là RUNNING còn desiredStatus sẽ là STOPPED.

#### Task Execution Role vs Task Role

Có hai loại IAM roles trong ECS task:

- **Task Execution Role**: là vai trò được sử dụng bởi ECS Container Agent (không phải container application). Nó cấp quyền cho ECS agent thực hiện các hành động như:

    - Kéo (pull) Docker images từ Amazon ECR
    - Truy cập Secrets Manager hoặc Parameter Store để lấy cấu hình
    - Gửi logs đến CloudWatch Logs
    - Kéo tệp từ S3 để khởi chạy

- **Task Role**: là vai trò được sử dụng bởi ứng dụng bên trong container. Nó cho phép ứng dụng gọi các dịch vụ AWS khác như:

    - S3 (để đọc/ghi files)
    - DynamoDB (để truy cập cơ sở dữ liệu)
    - SNS/SQS (để gửi messages)
    - RDS (để truy cập database)

#### Cách ECS Đặt (Placement) Tasks Trên Container Instances

Khi ta chạy một task trên EC2-based cluster, ECS phải quyết định nó sẽ chạy trên instance nào. Quá trình này được gọi là task placement, và nó có thể được kiểm soát bằng **Placement Strategies** và **Placement Constraints**.

**Placement Strategies**:

- **spread**: Phân tán tasks trên nhiều instances dựa trên một thuộc tính (ví dụ: ecs.availability-zone, ecs.instance-type). Tốt cho high availability

- **binpack**: Đóng gói tasks vào ít instances nhất có thể để tiết kiệm chi phí

- **random**: Đặt tasks ngẫu nhiên (ít được sử dụng)

**Placement Constraints**:

- **distinctInstance**: Đảm bảo mỗi task chạy trên một instance khác nhau

- **memberOf**: Chỉ đặt tasks trên các instances có thuộc tính/tag cụ thể

#### Tasks - Standalone vs Service

**Standalone Task**: Dùng cho các job ngắn hạn (batch jobs, one-off tasks). Task sẽ chạy đến khi hoàn thành hoặc gặp lỗi, rồi dừng lại. Không có cơ chế tự động restart.

**Service Task**: Dùng cho các ứng dụng dài hạn. Service scheduler tự động duy trì số lượng desired tasks bằng cách khởi chạy lại bất kỳ task nào bị dừng.
