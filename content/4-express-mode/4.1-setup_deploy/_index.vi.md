---
title : "Express Mode Setup & Deployment"
date :  2025-01-01
weight : 1
chapter : false
pre : " <b> 4.1. </b> "  
---

Trước khi có thể triển khai ứng dụng bằng Express Mode, chúng ta cần một Infrastructure Role có tên `ecsExpressInfrastructureRole`.

IAM role này cho phép Amazon ECS Express Mode thay mặt bạn quản lý các thành phần hạ tầng, bao gồm:

- **Load balancers**

- **Auto-scaling**

- **Cấu hình networking**

#### Tạo Infrastructure Role

Đầu tiên chúng ta cần tạo trust policy cho role này với nội dung file json bên dưới:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Service": "ecs.amazonaws.com"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}
```

Tiến hành chạy lệnh sau để tạo Role:

```bash
aws iam create-role \
  --role-name ecsExpressInfrastructureRole \
  --assume-role-policy-document file://ecs-express-trust.json
```

![Tạo Infrastructure Role](/images/4-express-mode/4.1-setup_deploy/1.png)

Tiếp theo tiến hành Attach policy AWS-managed vào role vừa tạo:

```bash
aws iam attach-role-policy \
  --role-name ecsExpressInfrastructureRole \
  --policy-arn arn:aws:iam::aws:policy/service-role/AmazonECSInfrastructureRoleforExpressGatewayServices
```

![Attach policy AWS-managed vào role vừa tạo](/images/4-express-mode/4.1-setup_deploy/2.png)

Kiểm tra Managed Policy của Infrastructure Role

```bash
aws iam list-attached-role-policies \
  --role-name ecsExpressInfrastructureRole
```

![Kiểm tra Managed Policy của Infrastructure Role](/images/4-express-mode/4.1-setup_deploy/3.png)

Như vậy chúng ta đã tạo được Infrastructure Role và Attach policy AWS-managed vào role vừa tạo.

#### Express Mode Deployment

Chúng ta sẽ triển khai UI service tương tự như ở phần Fundamentals, nhưng sử dụng Amazon ECS Express Mode.

Service Express Mode chỉ cần ba thành phần để bắt đầu:

- **Container image**
- **Task execution role**
- **Infrastructure role**

Để chạy ứng dụng mẫu, chúng ta sẽ sử dụng một số tham số tùy chỉnh được mô tả trong bảng dưới đây:

| Tham số                   | Bắt buộc | Mô tả                                                            |
| ------------------------- | -------- | ---------------------------------------------------------------- |
| `execution-role-arn`      | Có       | IAM role dùng cho task execution                                 |
| `infrastructure-role-arn` | Có       | IAM role dùng để quản lý hạ tầng                                 |
| `primary-container`       | Có       | Xác định container chính và port                                 |
| `cluster`, `service-name` | Không    | Tên ECS cluster và tên Express service                           |
| `cpu`, `memory`           | Không    | Tài nguyên cấp phát cho service                                  |
| `health-check-path`       | Không    | Endpoint dùng cho health check                                   |
| `network-configuration`   | Không    | Xác định VPC subnets và security groups cho task                 |
| `scaling-target`          | Không    | Xác định cách service tự động scale số lượng task theo nhu cầu   |
| `monitor-resources`       | Không    | Bật giám sát tài nguyên trong quá trình create, update và delete |

Chạy câu lệnh bên dưới để triển khai Service `ui-express-mode` trên ECS cluster hiện có `retail-store-ecs-cluster`.

```bash
aws ecs create-express-gateway-service \
    --execution-role-arn arn:aws:iam::${ACCOUNT_ID}:role/retailStoreEcsTaskExecutionRole \
    --infrastructure-role-arn arn:aws:iam::${ACCOUNT_ID}:role/ecsExpressInfrastructureRole \
    --service-name ui-express-mode \
    --cpu "1024" \
    --memory "2048" \
    --health-check-path "/actuator/health" \
    --cluster retail-store-ecs-cluster \
    --network-configuration '{
      "subnets": ["'$PUBLIC_SUBNET1'","'$PUBLIC_SUBNET2'"]
    }' \
    --scaling-target '{
        "minTaskCount": 2,
        "maxTaskCount": 5,
        "autoScalingMetric": "REQUEST_COUNT_PER_TARGET",
        "autoScalingTargetValue": 50
    }' \
    --primary-container '{
        "image": "public.ecr.aws/aws-containers/retail-store-sample-ui:1.2.3",
        "containerPort": 8080
    }' --monitor-resources
```

Sau khi chạy lệnh trên, ECS Express Mode sẽ tự động:

- Tạo task definition

- Cấu hình load balancer

- Thiết lập service

- Thiết lập auto-scaling

- Cấu hình các thành phần networking cần thiết

Tùy chọn `--monitor-resources` sẽ hiển thị tổng quan tất cả các tài nguyên đang được tạo như ảnh bên dưới:

![Tùy chọn --monitor-resources](/images/4-express-mode/4.1-setup_deploy/4.png)

Để đóng độ giám sát, có thể ấn `q`.

Trong khi ECS Express Mode đang tạo tài nguyên, hãy truy cập Amazon ECS Console để xem:

- Các service đang hoạt động

- Các task đang chạy

![Express Mode Service ui-express-mode đã được triển khai thành công](/images/4-express-mode/4.1-setup_deploy/5.png)

![Danh sách Task và tài nguyên sau khi được tạo thành công](/images/4-express-mode/4.1-setup_deploy/6.png)

Các Task và tài nguyên sau khi được tạo thành công.

![Các tài nguyên sau khi được tạo thành công](/images/4-express-mode/4.1-setup_deploy/8.png)

![Resource được hiển thị trên --monitor-resources](/images/4-express-mode/4.1-setup_deploy/7.png)


Tiến hành lấy DNS của Application Load Balancer và truy cập vào ứng dụng.

```bash
ECS_ALB_DNS=$(aws ecs describe-express-gateway-service \
  --service-arn arn:aws:ecs:${AWS_REGION}:${ACCOUNT_ID}:service/retail-store-ecs-cluster/ui-express-mode \
  --query service.activeConfigurations[0].ingressPaths[0].endpoint \
  --output text)

echo "https://${ECS_ALB_DNS}"
```

![Giám sát ứng dụng trong các tab Observability và Logs](/images/4-express-mode/4.1-setup_deploy/9.png)

Tiến hành truy cập vào ứng dụng bằng DNS:

![Tab Resources cung cấp tổng quan tất cả tài nguyên AWS được Express Mode tự động provision](/images/4-express-mode/4.1-setup_deploy/11.png)

#### Express Mode Review

**Service Overview** hiển thị tổng quan service, có thể:

- Theo dõi trạng thái service

- Xem các task đang chạy

- Giám sát tiến trình deployment

- Lưu ý Application URL được hiển thị trên trang này — có thể sử dụng URL đó để truy cập ứng dụng đã triển khai.

![Xem log ứng dụng trong phần Logs](/images/4-express-mode/4.1-setup_deploy/10.png)

Để giám sát ứng dụng, có thể xem các metrics của service `ui-express-mode` trong các tab:

- Observability

- Logs

Trong phần Logs, có thể xem log ứng dụng, lọc các thông điệp cụ thể và theo dõi hành vi ứng dụng theo thời gian thực.

![Kiểm tra cấu hình Express Mode cho revision mới nhất của service ui-express-mode](/images/4-express-mode/4.1-setup_deploy/12.png)

Tab Resources cung cấp tổng quan tất cả tài nguyên AWS được Express Mode tự động provision.

Danh sách này bao gồm đầy đủ các thành phần hạ tầng được tạo và quản lý cho bạn, chẳng hạn như:

- Security Groups

- Load Balancers

- Target Groups

![Kiểm tra cấu hình Express Mode cho revision mới nhất của service ui-express-mode](/images/4-express-mode/4.1-setup_deploy/13.png)

Kiểm tra cấu hình Express Mode cho revision mới nhất của service ui-express-mode bằng lệnh sau:

```bash
SERVICE_REVISION_ARN=$(aws ecs list-service-deployments \
  --service arn:aws:ecs:${AWS_REGION}:${ACCOUNT_ID}:service/retail-store-ecs-cluster/ui-express-mode \
  --max-results 1 \
  --query serviceDeployments[0].targetServiceRevisionArn \
  --output text)
```

```bash
aws ecs describe-service-revisions \
  --service-revision-arn $SERVICE_REVISION_ARN
```

![Kiểm tra cấu hình Express Mode cho revision mới nhất của service ui-express-mode](/images/4-express-mode/4.1-setup_deploy/16.png)

Trong quá trình tạo service, Express Mode đã tự động tạo Task Definition cho ứng dụng. Có thể liệt kê các revision của Task Definition bằng lệnh sau:

```bash
aws ecs list-task-definitions --family-prefix retail-store-ecs-cluster-ui-express-mode --sort DESC --max-items 2
```

![Liệt kê các revision của Task Definition](/images/4-express-mode/4.1-setup_deploy/17.png)

Kiểm tra Load Balancer bằng cách mở Amazon Load Balancer Console và chọn load balancer có tên dạng `ecs-express-gateway-alb-XXXXXXXX` để xem chi tiết cấu hình.

![Kiểm tra Load Balancer](/images/4-express-mode/4.1-setup_deploy/15.png)

Có thể thấy:
- HTTPS listener
- Certificate hợp lệ

Đã được ECS Express Mode tự động cấu hình.


