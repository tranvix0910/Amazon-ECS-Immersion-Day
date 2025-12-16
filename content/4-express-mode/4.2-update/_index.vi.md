---
title : "Express Mode Update"
date :  2025-12-15
weight : 2
chapter : false
pre : " <b> 4.2. </b> "  
---

#### Update Service

Trong phần này, chúng ta sẽ cập nhật service ui-express-mode bằng cách thay đổi giao diện mặc định của ứng dụng sang màu cam thông qua việc cập nhật environment variable.

```bash
"environment": [{
  "name": "RETAIL_UI_THEME",
  "value": "orange"
}]
```

Chạy lệnh sau để cập nhật Service `ui-express-mode`.

```bash
aws ecs update-express-gateway-service \
    --service-arn arn:aws:ecs:${AWS_REGION}:${ACCOUNT_ID}:service/retail-store-ecs-cluster/ui-express-mode \
    --primary-container '{
        "image": "public.ecr.aws/aws-containers/retail-store-sample-ui:1.2.3",
        "environment": [{
          "name": "RETAIL_UI_THEME",
          "value": "orange"
        }]
    }'
```

![Cập nhật Service ui-express-mode](/images/4-express-mode/4.2-update/1.png)

Mở Amazon ECS Console:

- Chọn tab Deployments.

- Chọn deployment đang diễn ra (in progress).

- Xem chi tiết quá trình Deployment.

![Cập nhật Service ui-express-mode](/images/4-express-mode/4.2-update/2.png)

![Cập nhật Service ui-express-mode](/images/4-express-mode/4.2-update/3.png)

Amazon ECS Express Mode sử dụng ECS **canary deployment strategy**.

Cơ chế này hoạt động như sau:

- Ban đầu, một tỷ lệ nhỏ traffic sẽ được chuyển đến revision mới để kiểm thử.

- Sau khi giai đoạn canary hoàn tất thành công, toàn bộ traffic còn lại sẽ được chuyển sang revision mới trong một lần.

![Cập nhật Service ui-express-mode](/images/4-express-mode/4.2-update/4.png)


#### Kiểm tra ứng dụng

Lấy DNS của Application Load Balancer:

```bash
ECS_ALB_DNS=$(aws ecs describe-express-gateway-service \
  --service-arn arn:aws:ecs:${AWS_REGION}:${ACCOUNT_ID}:service/retail-store-ecs-cluster/ui-express-mode \
  --query service.activeConfigurations[0].ingressPaths[0].endpoint \
  --output text)

echo "https://${ECS_ALB_DNS}"
```

![Lấy DNS của Application Load Balancer](/images/4-express-mode/4.2-update/5.png)

Tiến hành truy cập vào ứng dụng bằng DNS:

![Truy cập vào ứng dụng bằng DNS](/images/4-express-mode/4.2-update/6.png)
