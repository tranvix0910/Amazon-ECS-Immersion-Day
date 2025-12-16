---
title : "Amazon ECS Task Definition"
date :  2025-01-01
weight : 2.4
chapter : false
pre : " <b> 2.4. </b> "  
---

### Amazon ECS Task Definition

Task Definition là một bản thiết kế (blueprint) JSON mô tả cách một hoặc nhiều Docker containers sẽ chạy trong AWS ECS. Nó giống như một công thức hoặc template chứa tất cả các cấu hình cần thiết để khởi chạy containers, bao gồm Docker image, CPU, memory, port mappings, environment variables, logging, và nhiều hơn nữa.

#### Các Thành Phần Chính Của Task Definition

Task Definition chia thành hai mức độ cấu hình: **Task-level parameters (cấp độ task)** và **Container definitions (cấp độ container)**.

**Task-Level Parameters (Cấp Độ Task)**

- **Family**: Tên của Task Definition family. Đây là tên sẽ sử dụng để nhận diện task definition. Ví dụ: "my-web-app", "data-processor". Khi cập nhật task definition, một revision mới được tạo nhưng family name vẫn giữ nguyên.​

- **Task Size (CPU & Memory)**: Tổng lượng CPU và memory được cấp phát cho toàn bộ task. Điều này khác biệt với CPU/memory ở mức container. Với Fargate, các giá trị này là bắt buộc và phải là những giá trị hỗ trợ cụ thể (ví dụ: 0.25 vCPU với 0.5GB-1GB memory). Với EC2, các giá trị này là tùy chọn.​

- **Network Mode**: Cách containers sẽ kết nối với network:​

    - **awsvpc**: Mỗi task nhận một Elastic Network Interface (ENI) riêng với IP address riêng từ VPC. Điều này cho phép task hoạt động như một EC2 instance trong VPC. Đây là mode duy nhất được hỗ trợ bởi Fargate và khuyến nghị cho EC2.​

    - **bridge**: Containers kết nối qua Docker bridge network. Các containers trên cùng host có thể giao tiếp với nhau. Port mapping được sử dụng để expose ports.​

    - **host**: Containers chia sẻ network namespace với host. Container ports được map trực tiếp với host ports. Không khuyến nghị vì nó giảm isolation.

    - **none**: Containers không có network connectivity.​

- **Launch Type**: Nơi task sẽ chạy:

    - **EC2**: Task chạy trên EC2 instances được quản lý bởi bạn.

    - **Fargate**: AWS quản lý infrastructure, chỉ cần định nghĩa requirements

- **Execution Role (Task Execution Role ARN)**: IAM role cho ECS Container Agent. Role này cho phép agent kéo Docker images từ ECR, đẩy logs đến CloudWatch, lấy secrets từ Secrets Manager.​

- **Task Role**: IAM role cho ứng dụng bên trong container. Role này cấp quyền cho code truy cập các dịch vụ AWS như S3, DynamoDB, SNS, v.v.​

- **Platform**: Loại CPU architecture (x86_64 hoặc ARM64). Với ARM64, có thể sử dụng AWS Graviton processors tiết kiệm chi phí hơn.​

**Container Definitions (Cấp Độ Container)**

Task Definition có thể chứa một hoặc nhiều container definitions. Mỗi container definition mô tả một container riêng biệt sẽ chạy như một phần của task.

- **Name**: Tên của container (ví dụ: `web-server`, `database-client`). Bắt buộc phải có.​

- **Image**: Docker image URI để sử dụng. Có thể là từ Docker Hub (ví dụ: `nginx:latest`) hoặc từ Amazon ECR (ví dụ: `123456789012.dkr.ecr.us-east-1.amazonaws.com/my-app:1.0.0`). Bắt buộc phải có.​

- **CPU**: Số CPU units được cấp phát cho container này. 1 vCPU = 1024 CPU units. Nếu có task size là 1 vCPU và 2 containers, có thể cấp phát 512 units cho mỗi container. Nếu để 0, container sẽ sử dụng CPU units còn lại không được cấp phát cho containers khác.​

- **Memory**: Bộ nhớ tính bằng MB. Nó có thể là:

    - **hard limit (memory)**: Memory tối đa mà container có thể sử dụng. Nếu vượt quá, container bị dừng.

    - **soft limit (memoryReservation)**: Memory được đảm bảo sẵn sàng cho container.​

- **Ví dụ**: containerPort 8080 → hostPort 0 có nghĩa container chạy trên port 8080, nhưng expose qua dynamic port mapping trên host.​

    - **Essential**: Boolean flag. Nếu **true**, nếu container này gặp sự cố, cả task sẽ bị dừng. Ít nhất một container phải có **essential=true**. Nếu false, task sẽ tiếp tục chạy ngay cả khi container này dừng.​

    - **Environment Variables**: Các biến môi trường được truyền vào container. Ví dụ: `SERVER_PORT=8080`, `DATABASE_HOST=db.example.com`.​

- **Logging Configuration**: Cách logs sẽ được ghi. Các tùy chọn bao gồm:

    - **awslogs**: Logs đẩy đến AWS CloudWatch

    - **splunk**: Logs đẩy đến Splunk

    - **datadog**: Logs đẩy đến Datadog

    - **awsfirelens**: Sử dụng Fluent Bit hoặc Fluentd

Ví dụ cấu hình CloudWatch:

```json
"logConfiguration": {
  "logDriver": "awslogs",
  "options": {
    "awslogs-group": "/ecs/my-task",
    "awslogs-region": "us-east-1",
    "awslogs-stream-prefix": "ecs"
  }
}
```

- **Mount Points**: Ánh xạ volumes vào container. Gồm:

    - **sourceVolume**: Tên của volume được định nghĩa ở task-level

    - **containerPath**: Đường dẫn bên trong container nơi volume được mount (ví dụ: /data)

    - **readOnly**: Có là read-only không​

- **Health Check**: Cấu hình kiểm tra sức khỏe của container:

```json
"healthCheck": {
  "command": ["CMD-SHELL", "curl -f http://localhost:8080/health || exit 1"],
  "interval": 30,
  "timeout": 5,
  "retries": 3,
  "startPeriod": 0
}
```

**Volumes (Lưu Trữ Dữ Liệu)​**

- **Volumes**: Volumes được định nghĩa ở task-level nhưng được mount vào containers thông qua mount points. Các loại volumes bao gồm:

    - **EBS Volumes**: Amazon Elastic Block Store volumes cho persistent storage (chỉ EC2)

    - **EFS Volumes**: Amazon Elastic File System cho shared persistent storage giữa nhiều tasks/instances

    - **FSx Volumes**: FSx for Windows File Server volumes

    - **Bind Mounts**: Mount một đường dẫn từ host EC2 instance vào container

    - **Docker Volumes**: Docker-managed volumes

Ví dụ định nghĩa volume:

```json
"volumes": [
  {
    "name": "my-efs",
    "efsVolumeConfiguration": {
      "fileSystemId": "fs-1234567",
      "transitEncryption": "ENABLED"
    }
  },
  {
    "name": "my-bind-volume",
    "host": {
      "sourcePath": "/mnt/data"
    }
  }
]
```

**Versioning và Revisions​**

Task Definitions là bất biến (immutable). Điều này có nghĩa là bạn không thể chỉnh sửa một task definition hiện có. Thay vào đó, bạn phải tạo một revision mới.​

Revision là một bản sao của task definition hiện tại với những thay đổi được áp dụng. Mỗi task definition family có thể có nhiều revisions:

- **my-app:1 (revision 1)**.

- **my-app:2 (revision 2)**.

- **my-app:3 (revision 3)**.

Khi cập nhật một service hoặc chạy một task, chỉ định **family:revision** để sử dụng (ví dụ: "my-app:3"). Điều này cho phép dễ dàng rollback đến revision cũ nếu có vấn đề bằng cách chỉ cập nhật service để sử dụng revision trước đó.​

Ưu điểm của kiến trúc này:

- **Audit trail**: Có thể xem tất cả các thay đổi đã được thực hiện.

- **Easy rollback**: Quay lại revision cũ nếu cần.

- **Version control**: Giống như version control cho infrastructure.

**Task Definition JSON**

```json
{
  "family": "my-web-app",
  "cpu": "256",
  "memory": "512",
  "networkMode": "awsvpc",
  "requiresCompatibilities": ["FARGATE"],
  "executionRoleArn": "arn:aws:iam::123456789012:role/ecsTaskExecutionRole",
  "taskRoleArn": "arn:aws:iam::123456789012:role/ecsTaskRole",
  "containerDefinitions": [
    {
      "name": "web-server",
      "image": "123456789012.dkr.ecr.us-east-1.amazonaws.com/my-app:1.0.0",
      "cpu": 256,
      "memory": 512,
      "essential": true,
      "portMappings": [
        {
          "containerPort": 8080,
          "hostPort": 8080,
          "protocol": "tcp"
        }
      ],
      "logConfiguration": {
        "logDriver": "awslogs",
        "options": {
          "awslogs-group": "/ecs/my-web-app",
          "awslogs-region": "us-east-1",
          "awslogs-stream-prefix": "ecs"
        }
      },
      "environment": [
        {
          "name": "ENVIRONMENT",
          "value": "production"
        },
        {
          "name": "DATABASE_HOST",
          "value": "db.example.com"
        }
      ]
    }
  ]
}
```


